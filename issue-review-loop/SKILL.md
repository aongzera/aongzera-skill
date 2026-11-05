---
name: issue-review-loop
description: Use when the user asks to work through GitHub issues or sub-issues one at a time with one subagent building and another reviewing, repeating until each passes, e.g. "take each sub-issue, one agent implements, another runs /code-review, fix and repeat, then the next issue".
---

# Issue Review Loop

## Overview

You orchestrate; subagents write and review the code. Each issue goes builder → reviewer → builder fixes → reviewer re-checks, until the reviewer says `VERDICT: CLEAN`. Then you verify it yourself, commit it, and take the next issue. You never write the feature code yourself.

## Setup (once)

1. Read the repo's agent docs (AGENTS.md/CLAUDE.md, spec, glossary, ADRs) and `gh issue view` the parent issue and every sub-issue. Order them by their "blocked by" links.
2. Find the verify commands in the repo's CI workflow, Makefile or package scripts (tests, build, and how to clean). If the CI has none, ask the user.
3. If the working tree isn't clean, ask the user before anything else. Otherwise make a feature branch from the latest `origin/main`. Commit locally per issue. Push or open a PR only when the user asks.
4. Put one plan step per issue on the task list.

Use `general-purpose` agents, one builder and one reviewer, and continue each with `SendMessage` to its agent id. The issues run one at a time: two builders never share a working tree.

## Builder brief (first message per issue)

Give the builder:
- The issue number, the `gh issue view N --repo owner/repo` command, and the docs to read first. Name the later sub-issues it should skim so the design doesn't block them, but say to build only this issue's behaviour.
- The skill to use (the user's choice, e.g. a TDD skill), the branch to stay on, and the rules: **do not commit or push; `git add -N` new files; edit with the Edit tool, not scripts**.
- The exact verify commands, from a clean build (e.g. `rm -rf build && make test`, plus the firmware/app build). Also a deliberate-break check: break each behaviour on purpose and confirm a test catches it, then revert.
- A report contract: files changed, API added, test names, command output, and what couldn't be verified.

## Reviewer brief (first message per issue)

Give the reviewer:
- **Review only; no edits.** The uncommitted working-tree diff is the change, and named earlier commits are context.
- Run the review skill the user named (e.g. `code-review` with args `high`).
- Check the issue's acceptance criteria and the repo rules, and run the verify commands itself.
- **Scope note:** behaviour belonging to later sub-issues is not a finding; flag only design that would block them.
- The areas to think hardest about, taken from the issue's acceptance criteria and from the risky parts the builder reported (timing, concurrency, error paths).
- A report contract: the line `VERDICT: CLEAN` or `VERDICT: CHANGES NEEDED`; numbered findings with file:line, problem and fix, most severe first; then the commands it ran and any it could not run.

## The loop

| Reviewer says | You do |
|---|---|
| CHANGES NEEDED | Forward the findings to the **same builder** with `SendMessage`, then send the result to the **same reviewer** for a re-review. |
| A finding that is the user's decision (architecture, project type, posting to GitHub) | Ask the user. Send the other findings to the builder meanwhile. The issue may go CLEAN with the decision still open; when the user answers, send the change to the builder as its own round and review it again. |
| A finding outside the code (stale issue text, outward-facing changes) | Ask the user before acting. A note about an issue outside this run goes in your final report. |
| CLEAN, but it couldn't run some commands | Run those commands yourself. If they fail, send the output to the builder as a new round. |
| No `VERDICT` line, or an empty report | Resume that agent and ask for the report in the contract's shape. |
| The same finding comes back after 3 rounds | Stop and ask the user, showing both sides. |
| CLEAN | Run "Before you commit" below, commit with the issue number in the subject, mark the step done, and brief the next issue. |

When a later issue builds on the same code, keep the same pair. Start a fresh pair when the agent's completion notice shows more than about 200k tokens used, or when moving to a different area of the codebase. A fresh pair needs a full brief again.

## Before you commit (every issue)

1. `git add -N .` so new files show in the diff, then check `git status` for stray files (IDE folders, generated output).
2. Rerun the verify commands yourself from a clean build.
3. Read the diff yourself if the last round's change wasn't reviewed (for example, doc-only fixes) or if any agent's run was flagged as unverified.

## Interruptions

When a builder or reviewer stops on a rate limit or network error, its work is saved. Resume it with `SendMessage`: "you were cut off; check `git status`/`git diff` to see where you are, continue the same task, then send the full report." Don't start a new agent unless it's needed. Don't poll; wait for the notification.

## When every issue is done

Tell the user:
- the commits, one line per issue;
- what was verified (tests, builds) and what wasn't (hardware, CI, other services);
- the open decisions and the notes about issues outside this run;
- that nothing is pushed, unless the user asked you to push.

## Common mistakes

| Mistake | Fix |
|---|---|
| The reviewer's own suggested fix causes a regression | Every re-review checks the last round's fixes as fresh code, not as "resolved" |
| New files missing from the reviewer's diff | `git add -N` before each review |
| Scripted edits flip line endings or drop `\` continuations | Builder uses the Edit tool and reports file line endings |
| A stale incremental build hides a break | Always verify from a clean build |
| The reviewer flags later sub-issues' work | Put the scope note in every review brief |
| Notes for future issues get lost | Copy reviewer notes about later issues into those issues' builder briefs |
| Claiming it works when only builds and PC tests ran | Report plainly what ran (tests, builds) and what didn't (hardware, CI, other services) |
