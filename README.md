# claude-personal-tools

A collection of custom Claude Code and Codex instructions, settings, skills, and agents for independent code review, QA testing, plan critique, code simplification, comment rewriting, subagent orchestration, and session handoffs.

Codex equivalents can be found under [`codex/`](codex/); see its [README](codex/README.md) for Codex-specific setup and installation.

## Repo contents

| Path               | What it is                                                                      |
| ------------------ | ------------------------------------------------------------------------------- |
| `CLAUDE.md`        | Global Claude Code instructions — see [Configuration](#configuration)           |
| `settings.json`    | Harness config: plugins, env, permissions — see [Configuration](#configuration) |
| `skills/`          | 7 slash-command skills — see [Skills](#skills)                                  |
| `agents/`          | 1 sandboxed subagent — see [Agents](#agents)                                    |
| `codex/`           | Codex equivalents — see the [Codex README](codex/README.md)                     |

## Configuration

Two harness-level files live at the repo root. Unlike the skills and agents, these affect every Claude Code session once installed, not individual commands. But they can be adapted for per-project requirements.

### CLAUDE.md

Global instructions Claude Code reads on every session startup. Drop-in replacement for (or content to merge into) `~/.claude/CLAUDE.md`. Sections:

- **Corrections Log** — running list of one-liners the user has had to correct so Claude stops repeating them
- **Code Quality** — use correct data structures, fix root causes, error handling for realistic failure modes, no commented-out behavior switches
- **Workflow** — planning-mode gating for complex changes, author-critic review discipline, tests required alongside implementation
- **Subagents & Orchestration** — self-contained subagent briefs with an explicit mandate to be thorough and push back (even against the orchestrator), delegate only what you can evaluate, plans classify blocks `[subagent]`/`[main]`, automatic `HANDOFF.md` via `/handoff` when stopping for human input
- **Scientific Programming (Python/ML projects)** — reproducibility (seeding, config-alongside-outputs), numerics (precision, guards, device/dtype propagation), invariant tests, logging
- **Software Engineering Standards** — dependency pinning + lockfiles, no binaries/secrets in git, CI + pre-commit
- **Karpathy-style** — think before coding, simplicity first, surgical changes, goal-driven execution

### settings.json

Claude Code harness configuration. Drop-in replacement for (or content to merge into) `~/.claude/settings.json`.

The `permissions` block is designed as a **safer substitute for `--dangerously-skip-permissions`**. This `settings.json` keeps the same ergonomic "don't ask me every time" feel by putting all the common tools (`Read`, `Edit`, `Write`, `Glob`, `Grep`, `Bash(*)`, `WebFetch`, `WebSearch`, `NotebookEdit`) in `allow`, but it still gates potentially important patterns behind `ask` so they require confirmation:

- `Bash(rm -rf *)`, `rm -fr *`, `rm -r *`
- `Bash(git commit *)`, `git push *`, `git reset --hard *`, `git rebase *`
- `Bash(git clean *)`, `git branch -D *`, `git stash drop *`, `git stash clear`
- `Bash(git checkout .)`, `git restore .`
- `Bash(killall *)`, `docker rm *`, `docker rmi *`

Other keys:

- **`enabledPlugins`** — `code-review@claude-plugins-official` (Anthropic's code review plugin) and `codex@openai-codex` (OpenAI Codex CLI integration, registered via `extraKnownMarketplaces` from `github:openai/codex-plugin-cc`)
- **`env`** — `CLAUDE_CODE_EFFORT_LEVEL=max` and `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`. The latter is a temporary workaround for a regression in adaptive thinking (see [this thread](https://news.ycombinator.com/item?id=47660925)); remove it once the upstream fix lands
- **`outputStyle`** — `explanatory`

## Skills/Agents Overview

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
| `/handoff` | Write `HANDOFF.md` capturing session state so a fresh session can resume | Yes |
| `/asd-ste100-rewrite [target]` | Rewrite comments and docstrings into plain English, batch-previewed | Yes (comment text only) |

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

### `/handoff`

Session handoff writer. Produces a `HANDOFF.md` at the repo root that a **fresh, zero-context session** can resume from: goal, current state (verified vs. unverified work), key files, decisions made, open questions, ordered next steps with verification checks, and gotchas.

`/handoff` and `/asd-ste100-rewrite` are the **model-invocable** skills here (no `disable-model-invocation`); the rest wait for you to type them. For `/handoff`, the CLAUDE.md orchestration rules tell Claude to invoke it on its own whenever it stops mid-task for human input or finishes a long autonomous stretch. Clearing context and resuming from `HANDOFF.md` in a new session is far cheaper than replaying a long conversation at uncached token prices. Never commit the generated file.

### `/asd-ste100-rewrite`

Plain-English comment rewriter, and the on-demand counterpart to the `orwell-explanatory` output style described above. The style shapes text Claude is about to write, so it never reaches comments already on disk. This skill applies the same two rule sets, Orwell's six rules and the ASD-STE100 baseline, to comments and docstrings that already exist.

With no argument it targets the uncommitted changed files, tracked and untracked, and drops every `.md` path. Markdown enters scope only when you name the path or ask for it, which lets it clean up a `pr_<branch>_draft.md` from `/draft-pr-issue` without touching `README.md` on an ordinary run.

Two carve-outs keep the rules from damaging code. Orwell's fifth rule (avoid the jargon word) yields to the STE rule demanding the exact technical term, so it applies to ordinary prose and never to the vocabulary of the domain. On top of the style's verbatim exemptions, the skill leaves NumPy docstring sections, single-letter math variables matching paper notation, PEP 257 summary lines, and short Latin tags such as `e.g.` alone.

On top of the two rule sets it strips em dashes, replacing each with a comma, a semicolon, parentheses, the `--` double hyphen, or two sentences. The double hyphen itself is never a target, so command flags, section dividers, and the house dash convention all survive. It holds itself to the rule too, in its own preview and report. Like `/rephrase-comments`, it never changes code and applies its batch only after one approval.

## Agents

### `qa-tester`

A read-only agent with a hard tool restriction (`Read`, `Bash`, `Grep`, `Glob`). Used by `/qa_subagent` or invocable directly mid-conversation by saying "use the qa-tester agent."

## Installation

### Tools (skills and agents)

These merge into `~/.claude/skills/` and `~/.claude/agents/` by directory name and are safe to copy blindly unless you already have a skill or agent of the same name.

```sh
# Skills
cp -r skills/* ~/.claude/skills/

# Agents
cp -r agents/* ~/.claude/agents/
```

Skills are discovered at **session startup** — start a new Claude Code session after installing for them to appear in `/` autocomplete.

### Configuration (CLAUDE.md and settings.json)

**Do not blind-copy these if you already have a `~/.claude/CLAUDE.md` or `~/.claude/settings.json`** — a plain `cp` will overwrite your existing config. Diff first and merge by hand:

```sh
diff ~/.claude/CLAUDE.md     CLAUDE.md
diff ~/.claude/settings.json settings.json
```

If you don't already have these files (fresh Claude Code install), pick one of the two install modes:

```sh
# Option A — copy (independent of the repo after install)
cp CLAUDE.md     ~/.claude/CLAUDE.md
cp settings.json ~/.claude/settings.json

# Option B — symlink (repo stays the source of truth; `git pull` updates your config)
ln -s "$PWD/CLAUDE.md"     ~/.claude/CLAUDE.md
ln -s "$PWD/settings.json" ~/.claude/settings.json
```

Merge notes:

- **`CLAUDE.md`** — most sections (Code Quality, Workflow, Scientific Programming, Software Engineering Standards, Karpathy-style) append cleanly; the Corrections Log is personal, merge entries individually. Subagents & Orchestration references `/handoff`, so install `skills/handoff/` alongside it.
- **`settings.json`** — `enabledPlugins`, `extraKnownMarketplaces`, `env`, and `permissions.ask` all merge additively. Be deliberate about `permissions.allow` (widening it affects every session) and `skipDangerousModePermissionPrompt` (opts you out of the dangerous-mode warning entirely — only safe in combination with the full `ask` list from this repo).

## Notes

- All skills are **project-agnostic** — they discover conventions from CLAUDE.md, README, CI config, and existing tests rather than assuming any specific project structure.
- `/review` and `/simplify-code` run in the main context so you see edits in real-time.
- `skills/README.md` has a longer usage guide with examples and workflow walkthroughs.
