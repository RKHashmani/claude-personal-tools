---
name: handoff
description: Write a repository-root HANDOFF.md that captures the current session so a fresh, zero-context Codex session can resume the work. Use when stopping for human input or review mid-task, or after a long autonomous work stretch. Do not use for short sessions, quick clarifying questions, or plan-approval pauses.
---

# Session Handoff

Write `HANDOFF.md` at the repository root as an ephemeral snapshot of the current task. Assume the next Codex session has no conversation context. Fold any extra context from the user's invocation into the appropriate section.

## Gather the State

1. Confirm that you are the main agent. If you are a subagent, report your state to the orchestrator instead of writing `HANDOFF.md`.
2. Locate the repository root with `git rev-parse --show-toplevel`. If the workspace is not a Git repository, use the current workspace root.
3. Inspect the current branch, `git status --short --branch`, relevant diffs, and the verification commands and results from this session.
4. Read any existing repository-root `HANDOFF.md`. In a Git repository, check whether Git tracks it; outside Git, treat it as untracked.
   - If Git tracks it, stop and ask before replacing it.
   - If it is untracked and describes this same task, overwrite it.
   - If it describes different work, stop and ask before replacing it.
5. Recover the goal, success criteria, decisions, open questions, and attempted approaches from the conversation. Distinguish observed facts from assumptions.

## Write the Snapshot

Start the file with this exact line:

> **Fresh session: read this file fully before doing anything. Verify the items under "Done but unverified" before building on them. Delete this file once the work is complete.**

Then use these sections, omitting only those that are genuinely empty:

1. **Goal** — State the original request and concrete success criteria.
2. **Current state** — Separate work into done and verified, done but unverified, in progress, and not started.
3. **Key files** — Give exact paths and one line explaining why each matters.
4. **Repo state** — Record the current branch. For every uncommitted or untracked file, explain what it represents and whether to keep, finish, or discard it.
5. **Decisions made** — Record each decision and its rationale so the next session does not re-litigate it.
6. **Open questions** — State what the user must decide before work can continue.
7. **Next steps** — Order the remaining work and pair each step with its verification method.
8. **Gotchas** — Capture non-obvious constraints, environment quirks, and failed approaches with their failure reasons.

Use exact paths, commands, and branch names. Do not repeat file contents or facts that a fresh session can cheaply rediscover from the repository or Git history. Most handoffs should fit in roughly 100–150 lines; exceed that only to preserve an important decision or gotcha.

Treat work as verified only when a command was run or output was directly observed. Never promote believed or partially checked work to verified.

Never stage or commit `HANDOFF.md`.

## Report Completion

After writing the file, reply with its path and one sentence naming the most important open question or next step.
