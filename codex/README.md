# Codex Personal Tools

Codex equivalents for the global instructions, permissions, and session handoff workflow in this repository. See the [root README](../README.md) for the shared purpose and behavior; this file covers only Codex-specific structure and installation.

## Contents

| Path | Codex-specific role |
| --- | --- |
| `AGENTS.md` | Global guidance template for Codex |
| `config.toml` | Default approval and permission profile |
| `rules/default.rules` | Interactive safeguards for destructive commands |
| `skills/handoff/SKILL.md` | Codex-native `$handoff` workflow |
| `skills/handoff/agents/openai.yaml` | Skill UI metadata and implicit-invocation policy |

## Codex differences

- Codex reads global personal guidance from `$CODEX_HOME/AGENTS.md` when `CODEX_HOME` is set, or `~/.codex/AGENTS.md` by default.
- User-level skills live under `~/.agents/skills/`, so install this handoff skill as `~/.agents/skills/handoff/`.
- Invoke the skill explicitly as `$handoff`. `AGENTS.md` can also trigger it implicitly when the session-handoff conditions apply.
- The skill uses Codex-native `name` and `description` frontmatter. `agents/openai.yaml` supplies display metadata and keeps implicit invocation enabled.
- Permission profiles require Codex 0.138.0 or later and do not compose with legacy `sandbox_mode` or `sandbox_workspace_write` settings.
- No custom agent, plugin, or external tool dependency is required.

## Permissions

`config.toml` provides a safer analogue of the root `settings.json` permission posture without disabling Codex's sandbox:

- Read access across the filesystem, matching Claude's `Read` allowance.
- Write access limited to the active workspace and the standard temporary directories inherited from Codex's `:workspace` profile.
- Public network access for shell commands and live native web search without per-call approval.
- `approval_policy = "on-request"` for actions that need to cross the profile boundary.

`rules/default.rules` keeps the commands listed under `permissions.ask` in the root `settings.json` interactive: recursive deletion, Git commits and publishing/history rewrites, stash deletion, discarding working-tree changes, `killall`, and Docker container/image removal.

This setup intentionally avoids `approval_policy = "never"` and `:danger-full-access`. Codex may still ask when a command writes outside the workspace, reaches a non-public network destination, or otherwise exceeds the profile.

## Installation

Run these commands from the repository root. If either destination already exists, inspect and merge it instead of overwriting it:

```sh
diff ~/.codex/AGENTS.md codex/AGENTS.md
diff ~/.codex/config.toml codex/config.toml
diff ~/.codex/rules/default.rules codex/rules/default.rules
diff -ru ~/.agents/skills/handoff codex/skills/handoff
```

When merging `config.toml`, preserve unrelated model, UI, MCP, and project settings. Merge the three top-level permission/search keys and the `[permissions.claude-auto...]` tables; do not retain legacy `sandbox_mode` settings alongside them.

For a fresh installation, choose either copy or symlink mode:

```sh
mkdir -p ~/.codex/rules ~/.agents/skills

# Option A — copy
cp codex/AGENTS.md ~/.codex/AGENTS.md
cp codex/config.toml ~/.codex/config.toml
cp codex/rules/default.rules ~/.codex/rules/default.rules
cp -R codex/skills/handoff ~/.agents/skills/

# Option B — symlink instead
ln -s "$PWD/codex/AGENTS.md" ~/.codex/AGENTS.md
ln -s "$PWD/codex/config.toml" ~/.codex/config.toml
ln -s "$PWD/codex/rules/default.rules" ~/.codex/rules/default.rules
ln -s "$PWD/codex/skills/handoff" ~/.agents/skills/handoff
```

Use only one option for each destination. Prefer copying `default.rules` if you want Codex to append personal allow rules without modifying this repository through a symlink.

Restart Codex after installing or changing `AGENTS.md`, `config.toml`, or `default.rules`. Codex normally detects skill changes automatically; restart it if `$handoff` does not appear.
