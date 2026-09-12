---
name: review-and-polish
description: Chained workflow — review code with the scrutinize skill, fix findings using karpathy-guidelines (surgical, minimal fixes only), fall back to debug-mantra if a fix doesn't work as expected, then verify the original goal is actually accomplished before stopping. Use when asked to finish, polish, or ship a piece of work — review it, fix it cleanly, and confirm it's done.
---

# Review and polish

A three-skill chain for taking a piece of work from "written" to "verified done, cleanly."

## Workflow

1. **Scrutinize** — invoke the `scrutinize` skill against the change. Get concrete findings (blocker/major/nit), each with file:line, why it matters, and a suggested fix. Do not skip its "simpler alternative" pass.
2. **Karpathy-guidelines** — invoke the `karpathy-guidelines` skill, and apply ONLY the findings scrutinize actually surfaced. Fixes must be surgical: touch only what's needed, no speculative abstractions, no drive-by refactors of adjacent code. Note but don't fix findings outside the current task's scope — flag them for the user instead.
3. **Debug-mantra, if something breaks** — if applying a fix causes something unexpected (a type error, a failing test, behavior that doesn't match intent), invoke the `debug-mantra` skill before guessing at another fix.
4. **Verify against the actual goal.** Re-read the original ask. Re-run whatever check proves it (typecheck, a manual trace, a test). Don't declare done because "the diff looks right" — confirm the stated goal is actually met.
5. **Report, don't just fix silently.** State what scrutinize found, what you fixed and why, what you deliberately left alone (and why), and confirmation the goal is verified.

## Boundaries

- This chain reviews and fixes; it does not expand scope. If scrutinize or karpathy-guidelines surfaces something outside what was asked (a missing test suite, an unrelated bug), name it and ask rather than doing it unprompted.
- If the surrounding project uses `mentor-mode` (a "you review, I write" relationship with the user), this chain's fix step applies only to code already written by the assistant — e.g. under an explicit time-crunch bypass — not as a backdoor to rewrite the user's own learning-territory code unasked.
