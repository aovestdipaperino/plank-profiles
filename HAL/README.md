# HAL, a plank mail and calendar profile

Run it:

    plank --profile aovestdipaperino/plank-profiles:HAL

The first launch asks before installing it into `~/.plank/profiles/hal/`;
after that the same command, or `plank --profile hal`, starts it directly.
Change it from inside a session with `/edit-profile`, and remove it with
`rm -rf ~/.plank/profiles/hal`.

## The mail server

HAL's mail tools come from Softeria's
[`ms-365-mcp-server`](https://github.com/Softeria/ms-365-mcp-server), pinned to
0.156.2 and started through `npx`, so it needs Node.js but no install step and
no Azure app registration: it signs in with Softeria's own. Its tools are
limited by `--enabled-tools` to eleven mail tools, and because it requests only
the permissions its enabled tools need, the token covers `Mail.ReadWrite` and
never `Mail.Send`.

Sign in once, in a terminal, **with the same filter** (without it the sign-in
would ask for every permission the server knows, sending included):

    MS365_MCP_TENANT_ID=consumers npx -y @softeria/ms-365-mcp-server@0.156.2 \
      --enabled-tools '^(list-mail-messages|list-mail-folders|list-mail-child-folders|list-mail-folder-messages|get-mail-message|list-mail-attachments|update-mail-message|create-draft-email|create-reply-draft|move-mail-message|create-mail-folder)$' \
      --login

It prints a code and a Microsoft address; open the address, enter the code,
and approve. `MS365_MCP_TENANT_ID=consumers` is for a personal Outlook.com or
Hotmail account; for a work or school account, drop it and add `--org-mode` in
both this command and `.mcp.json`.

## What HAL can and cannot do

The `tools.builtin` allow-list gives HAL five builtins — `read`, `more`,
`glob`, `search` and `ask` — and withholds every other one, `bash`, `edit` and
`write` included. It reads files it is pointed at and cannot run a shell or
change your working tree.

The allow-list governs **builtins only**. Every tool the mail server exposes is
available to HAL regardless of what is listed here, so the mail limits come
from the server and the prompt:

- **Enforced by the server:** no sending (no send, reply or forward tool, and
  the token lacks `Mail.Send`) and no deleting (no delete tool).
- **Asked of HAL by its prompt, not enforced:** moving only to
  `HAL-processed`, and changing only the read flag when updating a message.
  `move-mail-message` accepts any folder, so a move to Deleted Items is
  possible if HAL disobeys its prompt.
