# aongzera-skill

A collection of [Claude Code](https://claude.com/claude-code) skills for coding workflows — review, debugging discipline, and learning-oriented coaching.

## Skills

- **[review-and-polish](review-and-polish/SKILL.md)** — chained workflow: review code with `scrutinize`, fix findings with `karpathy-guidelines` (surgical, minimal fixes only), fall back to `debug-mantra` if a fix doesn't work as expected, then verify the original goal is actually accomplished before stopping.
- **[scrutinize](scrutinize/SKILL.md)** — outsider-perspective end-to-end review of a plan, PR, or code change. Questions intent first, then traces the actual code path (not just the diff) to verify the change does what it claims.
- **[karpathy-guidelines](karpathy-guidelines/SKILL.md)** — behavioral guidelines to reduce common LLM coding mistakes: avoid overcomplication, make surgical changes, surface assumptions, define verifiable success criteria.
- **[debug-mantra](debug-mantra/SKILL.md)** — four-mantra debugging discipline: reproduce, trace the fail path, falsify the hypothesis, cross-reference every breadcrumb, before proposing any fix.
- **[mentor-mode](mentor-mode/SKILL.md)** — Socratic, scaffold-don't-solve coaching style for walking a learner through writing their own code, question-by-question, instead of writing it for them.
- **[issue-review-loop](issue-review-loop/SKILL.md)** — works through GitHub issues or sub-issues one at a time: a builder subagent implements each one, a reviewer subagent checks it, and they repeat until the review is clean, then the orchestrator verifies, commits and moves to the next issue.
- **[handoff](handoff/SKILL.md)** — compacts the current conversation into a handoff document for another agent (or session) to pick up.

## Usage

Drop a skill folder into your `~/.claude/skills/` directory (or a project's `.claude/skills/`), and invoke it with `/<skill-name>` in Claude Code, or let it trigger automatically per its own description.
