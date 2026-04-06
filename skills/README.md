# Custom Skills & Agents — Usage Guide

## Available Skills

| Command | Purpose | Tools | Edits Files? |
|---------|---------|-------|-------------|
| `/review <files>` | Code review: critique, write tests, validate | Read, Edit, Write, Bash, Grep, Glob | Yes (writes tests) |
| `/review-plan <plan.md>` | Plan review: correctness, completeness, risks, questions | Read, Grep, Glob, Bash | No |
| `/qa [test-paths]` | Run tests, analyze failures, assess quality | Read, Bash, Grep, Glob | No (soft restriction) |
| `/qa_subagent [test-paths]` | Same as `/qa` but hard-sandboxed via subagent | Agent → Read, Bash, Grep, Glob | No (hard restriction) |
| `/simplify-code <files>` | Simplify code one edit at a time with safety guards | Read, Edit, Bash, Grep, Glob | Yes |

### Direct Agent Invocation

You can also say **"use the qa-tester agent"** mid-conversation to invoke the QA agent directly without the slash command.

## Workflow: Author-Critic Separation

The most important rule: **if you wrote it, don't review it in the same session.**

Claude will rationalize and defend its own work if it reviews in the same session that authored it. Use separate sessions to ensure genuine independence.

### The Loop

```
Session A (author)  →  write code or plan
Session B (critic)  →  /review-plan, /qa_subagent, or /review
Session A (author)  →  paste feedback from Session B, fix issues
Session C (critic)  →  re-review if changes were significant
```

- **Session A** creates and fixes. It has full context of why things were built the way they were.
- **Session B** only reads and critiques. Never let it start editing — that breaks its independence.
- **Transfer feedback** by copy-pasting Session B's output into Session A.

### When to Use Which Critic Skill

- **Reviewing a plan before implementation** → `/review-plan docs/my_plan.md`
- **Reviewing code after implementation** → `/review src/module.py`
- **Running and evaluating tests** → `/qa` (convenient) or `/qa_subagent` (guaranteed read-only)

## Examples

### Review a plan
```
/review-plan docs/multimodal_plan.md
```
Reads the plan, explores the codebase it targets, evaluates against 9 dimensions (correctness, completeness, ordering & dependencies, edge cases & error handling, feasibility, scope & focus, consistency, testability & verification, risks), asks clarifying questions, and delivers a verdict: Ready / Ready with caveats / Needs revision / Needs rethink.

### Review code
```
/review dataset_generation/latent_generator.py
/review dataset_generation/constructive_mutual_information.py dataset_generation/mutual_info_calculations.py
```
4-phase process: learn the project → critique the code → write tests → run tests and validate.

### Run QA
```
/qa                                    # full test suite, inline
/qa tests/test_constructive_mi.py      # specific test file, inline
/qa_subagent                           # full suite, hard-sandboxed subagent
/qa_subagent tests/test_modality*.py   # specific tests, hard-sandboxed
```

### Simplify code
```
/simplify-code dataset_generation/constructive_mutual_information.py
```
Discovers project conventions, identifies simplification opportunities, checks each against safety guards (never removes precision code, safety checks, seeded RNGs, etc.), then applies one small edit at a time — running tests and showing you each change for approval before proceeding.

## Skill Locations

All skills are in `~/.claude/` (global, available across all projects):

```
~/.claude/skills/review/SKILL.md
~/.claude/skills/review-plan/SKILL.md
~/.claude/skills/qa/SKILL.md
~/.claude/skills/qa_subagent/SKILL.md
~/.claude/skills/simplify-code/SKILL.md
~/.claude/agents/qa-tester.md
```

## Notes

- Skills are discovered at **session startup**. If you create or edit a skill, start a new session for it to appear in `/` autocomplete.
- `/qa` uses `allowed-tools` to restrict editing (behavioral, soft). `/qa_subagent` delegates to an agent with a hard `tools:` restriction — the subagent physically cannot call Edit or Write.
- `/review` and `/simplify-code` run in the main context (no fork) so you see edits in real-time and can approve each one.
- All skills are project-agnostic — they discover conventions from CLAUDE.md, README, CI config, and existing tests rather than assuming any specific project structure.
