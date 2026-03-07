# claude-personal-tools

A collection of custom Claude Code skills and agents for independent code review, QA testing, plan critique, and code simplification.

## Overview

These tools enforce an **author-critic separation** workflow: the session that writes code should never review it. A separate session runs these skills to provide genuinely independent feedback, avoiding the self-rationalization that happens when Claude reviews its own work in the same context.

```
Session A (author)  →  write code or plan
Session B (critic)  →  /review, /qa, /review-plan
Session A (author)  →  paste feedback, fix issues
Session C (critic)  →  re-review if changes were significant
```

## Skills

| Command | Purpose | Edits Files? |
|---------|---------|:------------:|
| `/review <files>` | 4-phase code review: learn project, critique, write tests, validate | Yes |
| `/review-plan <plan.md>` | Evaluate a plan for correctness, completeness, risks, and gaps | No |
| `/qa [test-paths]` | Run tests, analyze failures, assess test quality | No |
| `/qa_subagent [test-paths]` | Same as `/qa` but hard-sandboxed via subagent (cannot edit files) | No |
| `/simplify-code <files>` | Simplify code one edit at a time with safety guards and test validation | Yes |

### `/review`

Senior staff engineer code review. Phases:

1. **Learn** — Read project docs, test directory, tooling config, domain invariants
2. **Critique** — Classify issues by severity (critical/high/medium/low) across correctness, edge cases, error handling, security, API design, and more
3. **Write tests** — Tests that mirror the project's existing patterns, fixtures, and assertion style
4. **Validate** — Run the test suite, fix any test failures, report results

### `/review-plan`

Pre-implementation plan review. Evaluates against 9 dimensions:

- Correctness, completeness, ordering & dependencies, edge cases & error handling, feasibility, scope & focus, consistency, testability & verification, risks

Delivers a verdict: **Ready** / **Ready with caveats** / **Needs revision** / **Needs rethink**, plus clarifying questions for any ambiguities.

### `/qa` and `/qa_subagent`

Independent QA testing. Discovers the test environment, runs the suite, classifies each failure (test bug / code bug / environment issue), and evaluates overall test quality across coverage, assertion quality, determinism, and test smells.

- `/qa` restricts tools behaviorally (soft — allowed-tools list)
- `/qa_subagent` delegates to the `qa-tester` agent with a hard tool restriction — the subagent physically cannot call Edit or Write

### `/simplify-code`

Methodical code simplification with safety guards. Identifies opportunities (extract duplication, replace manual implementations, reduce function length, remove dead code, etc.) while enforcing hard rules against removing precision code, safety checks, seeded RNGs, numerical tolerances, or diagnostic logging.

Each simplification is applied one at a time: edit, run tests, show change, wait for approval.

## Agents

### `qa-tester`

A read-only agent with a hard tool restriction (`Read`, `Bash`, `Grep`, `Glob`). Used by `/qa_subagent` or invocable directly mid-conversation by saying "use the qa-tester agent."

## Installation

Copy or symlink the contents into your Claude Code config directory:

```sh
# Skills
cp -r skills/* ~/.claude/skills/

# Agents
cp -r agents/* ~/.claude/agents/
```

Skills are discovered at **session startup** — start a new Claude Code session after installing for them to appear in `/` autocomplete.

## Notes

- All skills are **project-agnostic** — they discover conventions from CLAUDE.md, README, CI config, and existing tests rather than assuming any specific project structure.
- `/review` and `/simplify-code` run in the main context so you see edits in real-time.
- The `instructions.md` file in `skills/` contains a detailed usage guide with examples and workflow recommendations.
