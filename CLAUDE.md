# CLAUDE_gen.md

Global instructions for Claude Code across all projects.

## Corrections Log

When the user corrects a mistake, add the correction here so it is not repeated.

<!-- Add corrections below this line, one bullet per correction -->
- Do NOT add `Co-Authored-By` lines to commit messages

## Workflow

### Planning
- Always use plan mode before changes that touch critical or complex logic (e.g., core algorithms, data pipelines, model architectures, mathematical computations).
- When something goes sideways during implementation, stop and re-plan — don't keep pushing down a broken path.
- During planning, ask as many clarifying questions as needed to fully understand the relevant parts of the codebase, the current system, and how the proposed change fits in. Do not proceed with a plan until ambiguities are resolved.

### Review & Validation
- Act as a critical reviewer: challenge the user's reasoning, flag edge cases, and identify potential issues before implementation begins.
- After implementation, prove the changes work — diff behavior between the working branch and `main`, run tests, and demonstrate correctness rather than just asserting it.
- Do not create a PR or propose merging until you can show evidence that the changes are correct and complete.

### Testing
- Every new feature or behavior change must include corresponding tests that verify it works as expected. Write tests alongside (or before) the implementation, not as an afterthought.
- Run the project's test suite after making changes to verify nothing is broken.

## Scientific Programming Standards

### Reproducibility
- Always seed all RNG sources together so results are deterministic (e.g., `random.seed()`, `np.random.seed()`, `torch.manual_seed()`, `torch.cuda.manual_seed_all()` in Python ML projects).
- Disable non-deterministic backend behavior for reproducible runs (e.g., `torch.backends.cudnn.deterministic = True` and `benchmark = False`); only enable stochastic optimizations when explicitly trading reproducibility for speed.
- Save full configuration alongside every output so any result can be reproduced (e.g., save args/config to JSON next to output files).
- Default random seed to a deterministic value so runs are reproducible without explicit `--seed`.
- Never use truly random seeds unless explicitly requested by the user.

### Code Quality & Style
- Use the project's configured linter/formatter (e.g., ruff, black). Follow PEP 8 with 120-char line width for Python.
- Add type hints to all new/modified function signatures (parameters + return type).
- Use descriptive names; single-letter math variables (A, Sigma, z) are acceptable when they match paper notation — add a comment referencing the equation/section.
- Keep functions under 60 lines and ~5 parameters. Extract helpers for longer functions.
- Eliminate code duplication: if the same logic appears twice, factor it into a shared function.
- Never control behavior by commenting/uncommenting code — use flags or config parameters.
- Prefer f-strings for string formatting in Python.

### Testing
- Write unit tests for all mathematical/numerical functions using the project's test framework (e.g., pytest).
- Test mathematical invariants (e.g., non-negativity constraints, matrix properties like positive semi-definiteness, correct output dimensions).
- For numerical functions, test against known analytical solutions where available.
- Use regression tests: save known-good outputs and compare with explicit tolerances (e.g., `np.testing.assert_allclose`).
- Place tests in a top-level `tests/` directory mirroring the source structure.

### Documentation
- Add docstrings to all new/modified public functions using a consistent format (e.g., NumPy-style: Parameters, Returns, Notes).
- For new docstrings, use the project's preferred style; do not reformat existing docstrings in a different style unless modifying the function.
- Include mathematical context in docstrings: reference paper equations/sections where applicable.
- Every module should have a module-level docstring explaining its role in the system.
- Config/argument files should be self-documenting via comments.

### Logging & Error Handling
- Use the language's standard logging framework, not print statements, for all output in production code paths (e.g., Python's `logging` module with `logger = logging.getLogger(__name__)`).
- `print()` is acceptable only in debug blocks and CLI entry points (`if __name__ == "__main__":`).
- Use typed exceptions with descriptive messages for input validation — never bare `assert` for user-facing checks (e.g., `raise ValueError(...)` / `raise TypeError(...)`).
- Never silently swallow exceptions. Always log or re-raise caught exceptions.
- Validate data shapes/dimensions at function boundaries when they are non-obvious.

### Numerical Best Practices
- Use library-provided linear algebra functions over manual implementations (e.g., `torch.linalg`, `numpy.linalg`, `scipy.linalg`).
- Validate mathematical properties of inputs before use (e.g., positive-definiteness of covariance matrices via Cholesky).
- Propagate device and dtype from input data — never hardcode device or precision (e.g., avoid `device='cuda'` or `dtype=torch.float32`).
- Use higher precision for computations where numerical accuracy matters (e.g., float64 for information-theoretic or covariance calculations).
- Guard against numerical edge cases: division by zero, log(0), overflow in exp(), and similar.

### Project Organization
- All packages must have proper init files (e.g., `__init__.py` for Python).
- Use proper package imports; avoid path-manipulation hacks (e.g., `sys.path.append()`).
- Scripts should be runnable from the project root, not require `cd` into a subdirectory.
- Keep distinct concerns (data processing, model architecture, training, visualization) as independent modules.

### Version Control
- Make small, focused commits with descriptive messages.
- Save argument files and config alongside results for traceability.
- Never commit large binary files (model checkpoints, datasets) or secrets. Use `.gitignore` appropriately.

### Data Management
- Save raw data separately from processed results.
- Document output formats: array shapes, value ranges, and units in docstrings or project docs.
- Use structured, self-describing formats with descriptive field names (e.g., `.npz` with named arrays, HDF5 with labeled datasets).

### Configuration Management
- All runtime behavior must be controllable via CLI arguments or config files — never require editing source code.
- Save complete configuration alongside outputs for every run.
- Use argfile or config-file syntax for reproducible argument sets (e.g., `@file` syntax with argparse).

## Software Engineering Standards

### Dependency Management
- Pin all dependencies with version bounds (e.g., `>=1.0,<2`); avoid unpinned or `*` versions.
- Commit lockfiles for reproducible installs (e.g., `pixi.lock`, `uv.lock`, `poetry.lock`).
- Keep dependency specs consistent across all environment/packaging files when adding or updating packages.
- Separate dev-only dependencies (testing, linting tools) from core dependencies using optional dependency groups.

### Security
- Never hardcode secrets, API keys, or credentials — use environment variables or config files.
- Never use `eval()`, `exec()`, or unsafe deserialization on untrusted data (e.g., `pickle.load()` on user-supplied files).
- Use safe loading modes when available (e.g., `torch.load(..., weights_only=True)`); document the reason when unsafe loading is necessary.
- Validate and sanitize all external inputs (CLI args, file paths, JSON) before use.
- Prefer safe subprocess invocation (e.g., `subprocess.run()` with list args); never use `shell=True` with user-provided input.

### CI/CD & Automation
- Configure CI to run linting, type checking, and tests on pull requests (e.g., GitHub Actions with ruff + pytest).
- Use pre-commit hooks for formatting and linting before commit.
- Automate repetitive workflows via scripts, not manual multi-step commands.

### Docker & Containerization
- Pin base image versions to specific tags, not `latest`.
- Run containers as a non-root user for security.
- Add `.dockerignore` to exclude unnecessary files (e.g., `data/`, `output_dir/`, `.git/`, `__pycache__/`).
- Keep Dockerfiles minimal — install only production dependencies.

### Performance & Memory Management
- Disable gradient computation for all inference and evaluation code paths (e.g., `torch.no_grad()`).
- Use mixed precision training when memory is a concern (e.g., AMP in PyTorch).
- Release memory between intensive phases (e.g., `gc.collect()` and `torch.cuda.empty_cache()` at epoch boundaries).
- Optimize data loading for the target hardware (e.g., `pin_memory=True` in PyTorch DataLoader for GPU training).
- Profile before optimizing — use profiling tools (e.g., `torch.profiler`, `cProfile`) to identify actual bottlenecks before making performance changes.
