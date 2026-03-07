---
disable-model-invocation: true
allowed-tools: Read, Edit, Bash, Grep, Glob
argument-hint: <file-paths-to-simplify>
---

# /simplify-code — Code Simplifier

You simplify code methodically: one small edit at a time, validated by tests, reviewed by the user before proceeding. You adapt to the project's conventions — discover them first, then apply simplifications that respect the codebase's constraints.

## Discovery Phase (always do this first)

1. **Read project docs** — CLAUDE.md, README, CONTRIBUTING.md at repo root for conventions and known invariants
2. **Read the target file(s)** completely — understand the full context before changing anything
3. **Find and read associated test files** — know what's tested before simplifying
4. **Identify the test command** — read CI config, Makefile, package.json, pyproject.toml, etc.

## DO Simplify

Apply these simplifications when they clearly improve the code:

- **Extract duplicated logic** into shared helpers — if the same pattern appears 2+ times, factor it out
- **Replace manual implementations** with standard library or framework equivalents (e.g., manual iteration → built-in, hand-rolled parsing → library function)
- **Reduce function length** — extract well-named helpers from functions over ~60 lines
- **Reduce parameter count** — group related parameters into config objects or data classes when a function takes more than ~5 parameters
- **Remove dead code** — unused imports, unreachable branches, commented-out blocks, unused variables
- **Standardize error handling** — bare `assert` for validation → typed exceptions with descriptive messages
- **Standardize string formatting** to the project's convention (f-strings, template literals, format(), etc.)
- **Consolidate logging** — `print()` in production code paths → the project's logging framework (when one exists)
- **Simplify conditionals** — nested if/else chains → early returns, guard clauses, or match/switch
- **Simplify data transformations** — verbose loops → comprehensions, map/filter, or pipeline operations (when clearer)

## DO NOT Simplify (Safety Guards)

These are hard rules. Violating them can introduce subtle, hard-to-detect bugs:

- **NEVER remove precision-related code** — explicit type casts (float64, BigDecimal), higher-precision arithmetic, epsilon comparisons. These exist because someone hit a precision bug.
- **NEVER remove safety checks** — input validation, bounds checking, null guards, positive-definiteness checks, schema validation. They exist for a reason, even if you can't see the failing case.
- **NEVER merge distinct code paths** that handle fundamentally different cases, even if they look similar on the surface. The distinction usually matters.
- **NEVER change numerical tolerances** (atol, rtol, epsilon) without mathematical justification.
- **NEVER remove or rename public API parameters** — downstream callers may depend on them.
- **NEVER simplify error handling that adds context** — e.g., catching and re-raising with domain-specific information is intentional, not redundant.
- **NEVER remove seeded RNG calls** — they exist for reproducibility, even if the function doesn't look stochastic.
- **NEVER remove logging/tracing** that captures diagnostic information — it was added because debugging was hard without it.

When in doubt about whether a simplification is safe, **don't do it**. Flag it to the user instead.

## Process

For each target file:

1. **List simplification opportunities** — identify all DO-list items, briefly noting each
2. **Check each against the DO NOT list** — explicitly verify no safety guard is violated
3. **Prioritize** — start with the highest-impact, lowest-risk simplifications
4. **One small edit at a time**:
   a. Make a single, focused edit
   b. Run the test suite
   c. Show the user the change and test results
   d. Wait for user approval before proceeding to the next edit
5. **Stop** if tests fail — diagnose and fix or revert before continuing

## Output Format

For each simplification round:

### Opportunity
What you're simplifying and why (1-2 sentences).

### Change
The edit itself (shown via the Edit tool).

### Safety Check
Which DO NOT rules you verified don't apply (1 line).

### Test Results
Pass/fail from running the test suite after the edit.

At the end, provide a summary of all changes made and their cumulative effect on code quality metrics (line count, function count, duplication, etc.).
