# HAL, a plank mail and calendar profile

Run it:

    plank --profile aovestdipaperino/plank-profiles:HAL

The first launch asks before installing it into `~/.plank/profiles/hal/`;
after that the same command, or `plank --profile hal`, starts it directly.
Change it from inside a session with `/edit-profile`, and remove it with
`rm -rf ~/.plank/profiles/hal`.

## The mail server

HAL's mail tools come from
[`plank-mail-mcp`](https://github.com/aovestdipaperino/plank-mail-mcp), an MCP
server over one Outlook mailbox through Microsoft Graph. Install it and sign in
once before launching HAL:

    cargo install --git https://github.com/aovestdipaperino/plank-mail-mcp
    plank-mail-mcp login

`.mcp.json` starts it with `--config ~/.plank/hal/mail.toml`; the file is
optional (see that repository's README). Outlook is the only provider for now;
Gmail is planned. Without a sign-in HAL still starts, and its mail tools say
the mailbox is not signed in.

## What HAL can and cannot do

The `tools.builtin` allow-list gives HAL five builtins — `read`, `more`,
`glob`, `search` and `ask` — and withholds every other one, `bash`, `edit` and
`write` included. It reads files it is pointed at and cannot run a shell or
change your working tree.

The allow-list governs **builtins only**. Every tool the mail MCP server
exposes is available to HAL regardless of what is listed here, so the mail
server sets its own limits: HAL can list, search and read mail, mark messages
read or unread, save drafts, and move messages to a `HAL-processed` folder. It
cannot send mail (the sign-in never grants that permission) and cannot delete
it (the server has no such operation, and `HAL-processed` is the only place it
can move a message to).
