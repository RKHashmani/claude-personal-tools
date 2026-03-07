---
disable-model-invocation: true
allowed-tools: Agent
argument-hint: [test-file-paths-or-patterns]
---

# /qa — Spawn subagent for Independent QA Testing (sandboxed)

You MUST use the Agent tool to spawn a subagent for this task. Do NOT perform the QA work yourself. Delegate entirely to the `qa-tester` agent.

Use the Agent tool with these parameters:
- `agent`: `qa-tester`
- `prompt`: Pass along any test file paths or patterns the user provided as arguments. If no arguments were given, instruct the agent to run the full test suite.

The qa-tester agent has a hard tool restriction (Read, Bash, Grep, Glob only) — it physically cannot edit files. This guarantees an independent, read-only assessment.

Wait for the agent to complete and present its findings to the user.
