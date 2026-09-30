---
name: adversarial-review
description: Adversarial code and architecture review with forced-disagreement debate and confidence scoring. Use for "red team this", "find holes", "break this", "stress test", "poke holes", "what's wrong with this", "find weaknesses", "challenge this".
argument-hint: [--fix] [rounds=N] [architecture] [scope: diff|path]
---

# Adversarial Review

An adapted, plank-native version of Mountain Fung's `adversarial-review`
Claude Code skill (see the attribution at the end). It runs a structured
adversarial pass over a diff or a design and scores every finding.

**Core principles:**
1. Guilty until proven exceptional — assume every line is broken.
2. Evaluate artifacts, not intent — ignore "TODO" and "will fix later".
3. Forced disagreement — MUST challenge before conceding.
4. Concrete over abstract — every finding needs a code snippet and a trigger.
5. Stay on the target — this reviews the user's own code and authorized
   systems, never someone else's.

**Anti-sycophancy test:** if the review would make the author say "yeah, I knew
all that", dig deeper.

## Mode selection

| Input | Mode |
|-------|------|
| Git diff / PR / changed files / a path | **Code Review** (Phase A) |
| Architecture doc / design / plan | **Architecture Attack** (Phase B) |
| Both or unclear | Ask the user to clarify scope. |

`$ARGUMENTS` may set the scope, the debate round count (`rounds=N`, default 5),
`architecture` to force Phase B, and `--fix` to allow auto-fixing mechanical
issues (default is read-only).

## Confidence scoring (every finding, both phases)

| Score | Action |
|-------|--------|
| 9-10 | Surface — high-confidence, likely demonstrable. |
| 7-8 | Surface with evidence — probable issue. |
| 4-6 | Suppress to a "Low-confidence appendix" only. |
| 1-3 | Discard. |

Confidence is a heuristic ranking that forces a priority order, not a
probability. Format: `[CONFIDENCE: 8/10] Missing await on delete_user() —
fire-and-forget silently fails`.

## Severity tiers (every finding)

| Tier | Definition | Score weight |
|------|-----------|--------------|
| **Blocking** | Security holes, data corruption, logic errors, races, silent failures | -15 |
| **Required** | Lazy patterns, unhandled edge cases, poor naming, type-safety holes | -5 |
| **Suggestion** | Suboptimal approaches, missing tests, performance concerns | -1 |

## Detection patterns

| Category | Patterns |
|----------|----------|
| **Injection / input** | Raw string interpolation into SQL/shell/paths, missing parameterization, unvalidated deserialization, template injection, unbounded queries |
| **Auth / secrets** | Missing permission checks, auth bypass, privilege escalation, secrets in code, fail-open error paths, signature checks that don't gate |
| **Race / async** | Missing `await`/`.await`, unguarded shared state, fire-and-forget, TOCTOU, retries that replay non-idempotent effects |
| **Silent failures** | Empty `catch {}` / `let _ =`, bare `except:`, log-only handling, swallowed errors, missing rollback |
| **Data integrity** | Partial writes without transactions, NULL/None mishandling, missing uniqueness/`null: false` constraints |
| **Boundaries** | Integer width/sign, off-by-one, encoding/decoding, buffer/allocation sizing, untrusted length fields |
| **Rust** | `unwrap`/`expect` on attacker-reachable paths, `unsafe` invariants unproven, panics across FFI, `as` casts that truncate, unchecked lock poisoning, blocking in async |

## Phase A: adversarial code review

### A.0 — Consult prior learnings (skip silently if absent)

Read what plank already knows about this project, to compound learnings:

1. `~/.plank/MEMORY.md` and `<cwd>/.plank/MEMORY.md` — extract `[feedback]`
   and `[project]` entries that name files or invariants in the diff.
2. `AGENTS.md` / `CLAUDE.md` in context — project rules already loaded.
3. Any `FINDINGS.md`, `docs/`, or security notes the repo ships.

Emit a short "Prior learnings applied" preamble before A.1, even if nothing
was found. A finding that matches a loaded rule gets **+1 confidence** and
auto-promotes to **Blocking**; cite the source.

### A.1 — Scope

Default to `git diff main...HEAD`; if empty, `git diff --staged`; if a path was
given in `$ARGUMENTS`, review that. If there is nothing to review, say so and
stop.

### A.2 — Critical scan

A silent first pass with the Detection Patterns table. Any finding at
confidence >= 7 is an automatic blocker — list these before the debate.

### A.3 — Parallel specialist dispatch (diffs >= 100 lines)

For diffs under 100 lines, skip to A.4. Otherwise dispatch specialists in
parallel with the `agent`/`fanout` tools, one domain each:

| Specialist | Focus | Key question |
|------------|-------|--------------|
| **Security** | Auth bypass, injection, privilege escalation | "How would I exploit this?" |
| **Correctness** | Logic errors, edge cases, races, boundary bugs | "What input breaks the invariant?" |
| **Failure modes** | Silent failures, missing rollback, retry/idempotency | "What happens when this fails?" |
| **Testing** | Missing negative-path tests, untested branches | "What test catches the regression?" |

Each returns findings with confidence + severity. Then deduplicate, drop
confidence < 7, rank by severity then confidence, and feed the survivors into
the debate.

### A.4 — Devil's advocate debate

Simulate Author vs Reviewer (a senior engineer):

- Up to N rounds (default 5, override via `rounds=N` in `$ARGUMENTS`).
- Label each: `### Round N — [Topic]`.
- The Reviewer must raise >= 1 substantive concern per round (no style nits
  while real issues remain); the Author must push back on >= 1 point before
  conceding.
- Every suggestion carries a concrete code snippet.
- If A.3 ran, the Reviewer uses specialist findings as ammunition — debated,
  not blindly accepted.
- Terminate early on "Consensus reached after N rounds."

**Topic priority:** correctness → error handling → security → performance →
maintainability → testing gaps. Reserve >= 1 round for a lower-priority topic
and flag anything skipped.

### A.5 — Fix-first (guarded)

Default is read-only. Auto-fix only when `$ARGUMENTS` contains `--fix`, or the
request plainly implies it ("review and fix", "clean this up"). When fixing is
allowed, prove the flaw first — write or run the failing test — then apply the
**smallest** change and keep the proof as a regression test.

- **Auto-fixable** (mechanical): missing `await`, empty `catch {}`, `==` vs
  `===`, unused imports, obvious typos.
- **Needs a human decision**: architecture changes, business logic, error
  strategy, performance tradeoffs.

### A.6 — Score & summary

`score = max(0, 100 - Blocking*15 - Required*5 - Suggestion*1 + AutoFixed*2)`

| Score | Grade |
|-------|-------|
| 90-100 | A — ship it |
| 75-89 | B — a few required changes |
| 60-74 | C — needs work |
| <60 | D/F — significant issues |

Summarise: score & grade, auto-fixed items, blockers (confidence >= 7), round
breakdown, agreed changes (with snippets), open disagreements, priority-ranked
action items, low-confidence appendix, and a verdict of **Request Changes /
Needs Discussion / Approve**.

## Phase B: architecture attack

### B.1 — Decomposition

Break the proposal into 10-20 atomic claims.

### B.2 — Multi-perspective attack

Attack each claim from six angles: **Skeptical Engineer** ("simplest thing that
could go wrong?"), **Incident Responder** ("how do we debug this at 3 AM?"),
**Security Researcher** ("how would I exploit this?"), **Maintainer** ("can I
own this in six months?"), **Cost Analyst** ("hidden operational cost?"),
**Devil's Advocate** ("what if the opposite approach is better?"). Attach
confidence + severity to each finding.

### B.3 — Steelman & counter-argument

Five strongest points in favour, then the five most devastating weaknesses.

### B.4 — Verdict

Report the atomic claims, attack findings by perspective, the steelman, the
counter-argument, any fatal flaws (or "none found"), a verdict of **Rethink /
Revise / Proceed with noted risks**, and recommended mitigations.

---

*Adapted for plank from [`lemon03390/Claude-code-adversarial-review-skill`](https://github.com/lemon03390/Claude-code-adversarial-review-skill)
by Mountain Fung, MIT-licensed. Changes: Claude Code paths retargeted to
plank's memory files, the specialist dispatch retargeted to plank's
`agent`/`fanout` tools, the detection table extended with boundary and Rust
patterns, and the knowledge-flywheel steps trimmed. See `ATTRIBUTION.md`.*
