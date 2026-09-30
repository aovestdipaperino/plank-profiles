# Attribution

`SKILL.md` in this directory is an adaptation of the **adversarial-review**
Claude Code skill:

- Source: https://github.com/lemon03390/Claude-code-adversarial-review-skill
- Author: Mountain Fung
- License: MIT

The upstream skill itself credits `richiethomas/claude-devils-advocate` (debate
mechanic), `posit-dev/skills/critical-code-reviewer` (detection patterns,
severity tiers), `danielmiessler/Personal_AI_Infrastructure/RedTeam`
(multi-perspective attack), and `garrytan/gstack` (specialist dispatch,
confidence scoring, fix-first).

## MIT License (upstream)

```
MIT License

Copyright (c) 2026 Mountain Fung

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Changes made in this adaptation

- Retargeted `~/.claude/error-tracker.json` and `~/.claude/knowledge-base/`
  to plank's memory files (`~/.plank/MEMORY.md`, `<cwd>/.plank/MEMORY.md`) and
  `AGENTS.md`; the optional knowledge-flywheel steps were trimmed.
- Retargeted the "Agent tool" specialist dispatch to plank's `agent`/`fanout`
  tools.
- Rewrote the folded-scalar frontmatter into plank's single-line
  `name`/`description`/`argument-hint` form.
- Extended the detection table with boundary and Rust/`unsafe` patterns and
  scoped the skill to the user's own code and authorized targets.
