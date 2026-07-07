---
description: Write HANDOFF.md capturing the current session's state so a fresh, zero-context session can resume the work. Use when stopping for human input or review mid-task, or when finishing a long autonomous work stretch — not for short sessions, quick clarifying questions, or plan-approval pauses.
allowed-tools: Read, Write, Bash, Grep, Glob
argument-hint: [optional extra context to include]
---

# /handoff — Session Handoff Writer

You are pausing or ending a work session. Write a `HANDOFF.md` file at the repository root that lets a **fresh session with zero conversation context** pick up exactly where this one left off. The reader has not seen this conversation — write accordingly.

If arguments were provided, treat them as extra context the user wants captured — fold them into the relevant sections.

## Rules

- If you are running as a subagent, do not write `HANDOFF.md` — report your state back to the orchestrator instead.
- `HANDOFF.md` is an ephemeral snapshot, not a log: overwrite an existing one if it describes this same task. If it is tracked by git or describes different work, stop and ask before replacing it.
- **Never commit or stage `HANDOFF.md`.**
- Use **exact paths, commands, and branch names** — never "the file we discussed" or "the fix from earlier."
- Cut anything a fresh session can rediscover from the code or git history — the handoff carries intent, decisions, and verification state, not file contents. Most handoffs fit in ~100–150 lines; going over is fine, losing a decision or gotcha is not.
- Distinguish clearly between what is **verified** (tests run, output observed) and what is merely **believed done**. Never promote unverified work to done.

## Template

Start the file with this line:

> **Fresh session: read this file fully before doing anything. Verify the items under "Done but unverified" before building on them. Delete this file once the work is complete.**

Then these sections (omit any that are genuinely empty):

1. **Goal** — the original request and its success criteria.
2. **Current state** — four lists: done & verified / done but unverified / in progress / not started.
3. **Key files** — exact paths, one line each on why they matter.
4. **Repo state** — current branch; for each uncommitted or untracked file: what it corresponds to and whether to keep, finish, or discard it.
5. **Decisions made** — each with its rationale, so the next session doesn't re-litigate them.
6. **Open questions** — what the human needs to decide before work can continue.
7. **Next steps** — ordered; each step paired with how to verify it.
8. **Gotchas** — non-obvious constraints, environment quirks, and approaches already tried that failed (and why).

## After writing

Reply with the file path and a one-line summary of the most important open question or next step, so the user knows what they are coming back to.
