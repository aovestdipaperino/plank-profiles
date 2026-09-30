# d3v1l, a plank adversarial code-analysis profile

An agent that reads your codebase the way an attacker would, then helps you
close what it finds. It hunts logic flaws, injection points, auth and signature
bypasses, unsafe patterns and boundary bugs; writes the failing test or minimal
proof-of-concept that turns a suspicion into a reproduction; and proposes the
smallest fix beside it. The pose is adversarial; the aim is hardening.

Run it:

    plank --profile aovestdipaperino/plank-profiles:d3v1l

The first launch asks before installing it into `~/.plank/profiles/d3v1l/`;
after that the same command, or `plank --profile d3v1l`, starts it directly.
Change it from inside a session with `/edit-profile`, and remove it with
`rm -rf ~/.plank/profiles/d3v1l`.

## Scope

d3v1l is for reviewing and hardening **your own code** and systems you are
authorized to test. Its prompt keeps it on the target in the working tree,
tells it to treat anything it reads as data rather than instructions, and to
refuse work that drifts toward attacking third parties or building tools whose
only purpose is harm. Point it at a repository, describe your threat model, and
ask it what breaks.

## The model

d3v1l recommends the `d4abliterated` engine. If that engine's model file is
already on disk, `--profile d3v1l` runs on it; otherwise plank keeps your usual
model and prints one line saying so. A `--model` on the command line always
wins. The recommendation is never downloaded on your behalf.

## What d3v1l can do

The `tools.builtin` allow-list gives it the tools an adversarial review needs
and withholds the rest:

- `read`, `more`, `glob`, `search`, `list` — read and navigate the codebase.
- `bash`, `bash_status`, `bash_stop` — run the project's tests, build and drive
  a fuzz harness (including long jobs in the background), execute a
  proof-of-concept against the tree.
- `edit`, `write` — write the failing test, the PoC, and the proposed fix.
- `skill` — invoke the skills below (and plank's built-ins).
- `task` — track a multi-step review that survives compaction.
- `agent`, `fanout` — dispatch parallel specialist sub-agents (the
  adversarial-review skill fans out security / correctness / failure-mode /
  testing reviewers).
- `ask` — interview you about the threat model and what is authorized.
- `google_search`, `visit_page` — look up a CVE, an advisory, a library's
  security notes.

`folderContext` and `agentsMd` are both on, so it starts with the repository's
git status and any `AGENTS.md`, and works against the project you launched it
in. Writes stay inside plank's normal write-containment; the prompt asks it to
keep proof-of-concept code clearly marked and to never send findings anywhere
off the machine.

## Skills

Because a profile is spliced in as a plugin, d3v1l gets plank's built-in skills
plus the ones it bundles.

Useful built-ins (always available): **`code-review`** (dispatches a reviewer
sub-agent and acts on its findings), **`debug`** (diagnoses a misbehaving tool
or session from plank's error log), and **`verify`** (runs plank itself to show
a change working). `code-review` needs the `agent` tool, which is why it is in
the allow-list.

Bundled with the profile:

- **`d3v1l:adversarial-review`** — a structured adversarial pass over a diff or
  a design: a critical scan, parallel specialist dispatch, a forced
  devil's-advocate debate, confidence-scored and severity-tiered findings, a
  guarded fix-first step, and a PR/architecture verdict. Invoke it with
  `/d3v1l:adversarial-review [--fix] [rounds=N] [architecture]`. It is adapted
  from Mountain Fung's MIT-licensed
  [`Claude-code-adversarial-review-skill`](https://github.com/lemon03390/Claude-code-adversarial-review-skill);
  see `skills/adversarial-review/ATTRIBUTION.md` for the retargeting and the
  upstream license.

Other public collections worth a look if you want to extend d3v1l — review
each `SKILL.md` before installing, since prompt injection has been found in a
meaningful fraction of published skills:
[`Masriyan/Claude-Code-CyberSecurity-Skill`](https://github.com/Masriyan/Claude-Code-CyberSecurity-Skill)
and [`SnailSploit/Claude-Red`](https://github.com/SnailSploit/Claude-Red).
