# plank profiles

Profiles for [plank](https://github.com/aovestdipaperino/plank), each in its own
folder. A profile launches plank as a different agent: its own system prompt,
its own set of builtin tools, its own settings, logo and accent colour.

Profiles need plank 6.0.0 or later (`brew install aovestdipaperino/tap/plank-agent`).

Launch one straight from this repository:

    plank --profile aovestdipaperino/plank-profiles:HAL

plank asks before installing it into `~/.plank/profiles/`, then starts it.
Later launches with the same argument use the installed copy without asking.

| Folder | Profile |
|---|---|
| [`HAL`](HAL) | Mail assistant over one Outlook mailbox, through Softeria's [`ms-365-mcp-server`](https://github.com/Softeria/ms-365-mcp-server) limited to mail tools. Reads and tidies mail; cannot send or delete. |

The manifest format is documented in plank's
[`docs/PROFILES.md`](https://github.com/aovestdipaperino/plank/blob/main/docs/PROFILES.md), and the [Profiles chapter](https://plank-agent.dev/guide/14-profiles) of the user guide walks through installing, updating and writing one.
