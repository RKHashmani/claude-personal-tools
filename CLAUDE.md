# CLAUDE.md

Global instructions for Claude Code across all projects.

## Corrections Log

When the user corrects a mistake, add the correction here so it is not repeated.

<!-- Add corrections below this line, one bullet per correction -->
- Do NOT add `Co-Authored-By` lines to commit messages
- Do NOT include "Generated with Claude Code" or any Claude attribution in pull request descriptions

## Code Quality
<!-- Set env CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1 w.r.t recent bug in Claude Code. From https://news.ycombinator.com/item?id=47660925. Possibly remove later. -->
- Use appropriate data structures and algorithms — don't brute-force what has a known better solution.
- When fixing a bug, fix the root cause, not the symptom.
- If something I asked for requires error handling or validation to work reliably, include it without asking — for realistic failure modes only, per Simplicity First.
- Never control behavior by commenting/uncommenting code — use flags or config parameters.

## Workflow

### Planning
- Always use plan mode before changes that touch critical or complex logic (e.g., core algorithms, data pipelines, model architectures, mathematical computations).
- When something goes sideways during implementation (e.g., an approach fails repeatedly, or the plan's assumptions turn out wrong), stop, ask the user, and re-plan — don't keep pushing down a broken path.

### Review & Validation
- Act as a critical reviewer: challenge the user's reasoning, flag edge cases, and identify potential issues before implementation begins.
- After implementation, prove the changes work — diff behavior between the working branch and `main`, run tests, and demonstrate correctness rather than just asserting it.
- Do not create a PR or propose merging until you can show evidence that the changes are correct and complete.

### Testing
- Every new feature or behavior change ships with tests, written alongside (or before) the implementation.
- Run the project's test suite after making changes to verify nothing is broken.

## Subagents & Orchestration

### Briefing subagents
- Write self-contained briefs: the goal, relevant file paths, constraints, explicit success criteria, and what you have already verified with evidence (e.g., tests run, outputs observed). The subagent starts with zero conversation context.
- Instruct every subagent to be thorough, to verify unverified claims against the actual code rather than trusting the brief, to surface anything questionable, and to push back if the brief appears wrong — reporting conflicts between the brief and reality instead of silently complying. Results the brief marks as verified can be trusted without re-running them; decisions (e.g., how to modify the code) are always open to challenge.
- Only delegate tasks whose output you can verify yourself (e.g., by reviewing the diff, running the tests, or re-running the check). Review subagent output critically before integrating it — treat it like a PR from a colleague, not a trusted oracle.
- When dispatching multiple subagents, send independent ones in a single message so they run in parallel.
- If you are a subagent: do not spawn further subagents or write handoff files.

### Plan classification
- In implementation plans, group related tasks into blocks and order the blocks by implementation dependency.
- Mark each block `[subagent]` if you can write a self-contained brief with verifiable success criteria for it; otherwise mark it `[main]`.

### Session handoffs
- When stopping mid-task because human input or review is needed, or when finishing a long autonomous stretch, invoke the `/handoff` skill to write `HANDOFF.md` — so the user can clear context and resume in a fresh session instead of replaying this one.
- Do not write handoffs for short sessions, quick clarifying questions, or plan-approval pauses.

## Scientific Programming (Python/ML projects)

### Reproducibility
- Seed all RNG sources together (e.g., `random.seed()`, `np.random.seed()`, `torch.manual_seed()`, `torch.cuda.manual_seed_all()`) with a deterministic default seed; never use truly random seeds unless explicitly requested.
- Disable non-deterministic backend behavior (e.g., `torch.backends.cudnn.deterministic = True`, `benchmark = False`); only enable stochastic optimizations when explicitly trading reproducibility for speed.
- Save the full configuration alongside every output so any result can be reproduced.

### Numerics
- Use library-provided linear algebra (`torch.linalg`, `numpy.linalg`, `scipy.linalg`) over manual implementations.
- Use higher precision where accuracy matters (e.g., float64 for information-theoretic or covariance calculations); validate mathematical properties of inputs before use (e.g., positive-definiteness via Cholesky).
- Guard against numerical edge cases: division by zero, log(0), overflow in exp().
- Propagate device and dtype from input data — never hardcode them.
- Disable gradient computation (`torch.no_grad()`) for inference and evaluation paths.

### Testing
- Test mathematical invariants (non-negativity, matrix properties, output dimensions) and, where available, known analytical solutions.
- Use regression tests with explicit tolerances (e.g., `np.testing.assert_allclose`); place tests in a top-level `tests/` directory.

### Style, Docs & Logging
- Follow PEP 8 with 120-char line width and the project's configured linter/formatter (e.g., ruff, black).
- Add type hints to new/modified function signatures.
- Use proper package imports; avoid `sys.path` manipulation.
- Use NumPy-style docstrings on public functions, referencing paper equations/sections; single-letter math variables are fine when they match paper notation.
- Use the `logging` module, not `print`, in production code paths; `print()` is acceptable in debug blocks and `if __name__ == "__main__":` entry points.

### Configuration & Data
- All runtime behavior must be controllable via CLI arguments or config files — never require editing source code.
- Use self-describing output formats (e.g., named `.npz` arrays, labeled HDF5) and document shapes, ranges, and units.

## Software Engineering Standards

- Pin dependencies with version bounds (e.g., `>=1.0,<2`) and commit lockfiles; keep specs consistent across environment/packaging files; separate dev-only dependencies from core ones.
- Never commit large binary files (model checkpoints, datasets) or secrets. Use `.gitignore` appropriately.
- Configure CI to run linting, type checking, and tests on pull requests; use pre-commit hooks for formatting and linting.


# Karpathy-style
<!-- Adapted from https://github.com/forrestchang/andrej-karpathy-skills -->

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
