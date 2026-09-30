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

- `read`, `more`, `glob`, `search` — read and navigate the codebase.
- `bash` — run the project's tests, build a fuzz harness, execute a
  proof-of-concept against the tree.
- `edit`, `write` — write the failing test, the PoC, and the proposed fix.
- `ask` — interview you about the threat model and what is authorized.
- `google_search`, `visit_page` — look up a CVE, an advisory, a library's
  security notes.

`folderContext` and `agentsMd` are both on, so it starts with the repository's
git status and any `AGENTS.md`, and works against the project you launched it
in. Writes stay inside plank's normal write-containment; the prompt asks it to
keep proof-of-concept code clearly marked and to never send findings anywhere
off the machine.
