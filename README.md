# Contribution #2: Normalize Windows paths in export()

**Contribution Number:** 2
**Student:** Jenisha Subedi
**Issue:** https://github.com/venomez-viper/breezeml/issues/10
**Status:** Phase I COMPLETE

---

## Why I Chose This Issue

I chose this issue because it sits at the intersection of Python tooling and cross-platform compatibility, which matches my strongest skill area. The problem is clearly scoped file paths written into exported scripts break on Windows because backslashes aren't valid in that context making it a great next contribution to a real, actively maintained machine learning library.

I'm also excited to learn more about how BreezeML's `export()` feature reproduces a full training pipeline as a standalone script. Contributing to BreezeML will give me hands-on experience with a production-aware ML tool, and since I'm on Windows myself, I can directly reproduce and verify the bug rather than working from a report alone which is exactly the kind of concrete, verifiable contribution I want on my resume.

---

## Understanding the Issue

### Problem Description

`Pipeline.export()` writes a standalone Python script that reproduces a trained ML pipeline, including the file path to the training data. On Windows, that path uses backslashes (e.g. `C:\Users\name\data.csv`). When written directly into the generated Python source as a plain string, backslashes are interpreted as escape characters, which can corrupt the path or raise errors when the exported script is run.

### Expected Behavior

The exported script should contain a properly escaped or normalized path (e.g. using forward slashes or a raw string) so that it runs correctly regardless of the operating system it was exported on.

### Current Behavior

The path is written into the generated script as-is, with raw backslashes, which is not guaranteed to produce valid, runnable Python source on Windows.

### Affected Components

- `Pipeline.export()` (path-writing logic)
- Generated training script output
- Existing test coverage: `tests/test_v040_features.py::test_export_writes_and_script_is_valid_python`

---

## Reproduction Process

### Environment Setup

*Not yet started planned for Phase II.* I will clone my fork (`jenishasubedi05/breezeml`), install BreezeML locally per `CONTRIBUTING.md` (`pip install -e ".[dev]"`), and confirm the existing test suite passes with `pytest tests/ -v` before making changes.

### Steps to Reproduce (planned)

1. Train a simple model using `breezeml.fit()` on a dataset loaded from a path containing a space or Windows-style separators
2. Call `model.export("train.py", data_path=...)` with a Windows-style path
3. Inspect the generated `train.py` to confirm the path is written with unescaped backslashes
4. Attempt to run the generated script and confirm it fails or produces an incorrect path

### Reproduction Evidence

*To be completed in Phase II, once the environment is set up and the bug is reproduced locally.*

---

## Solution Approach

### Analysis

*To be completed in Phase II, after reproducing the bug and reading the relevant `export()` implementation.*

### Proposed Solution (initial plan, subject to revision after reproduction)

1. Locate the path-writing logic inside `Pipeline.export()`
2. Normalize the path before it is embedded in the generated script  e.g. using `pathlib.Path(...).as_posix()` or converting to a raw string
3. Extend `tests/test_v040_features.py` to add a case that verifies exported scripts are valid Python on Windows-style paths

### Implementation Plan

**Understand:** An exported training script can contain an improperly escaped Windows path, risking a broken or invalid script.

**Match:** Follow the existing pattern in `export()` for how other pipeline attributes (imputers, scaler, encoder, seed) are serialized into the generated script.

**Plan:**
1. Reproduce the issue locally with a Windows-style path
2. Identify the exact line(s) in `export()` responsible for writing the path
3. Normalize the path before writing
4. Add/extend a test confirming the exported script is valid regardless of path format
5. Run the full test suite and `ruff check .` before submitting

**Implement:** *Not yet started Phase III.*

**Review:** Will follow `CONTRIBUTING.md`: pass CI, include tests, follow existing docstring style.

**Evaluate:** Will manually run the exported script after the fix to confirm it executes correctly, in addition to the automated test.

---

## Testing Strategy

*To be completed in Phase III, once the environment is set up and the fix is implemented.* Planned: extend `tests/test_v040_features.py` to cover path normalization for Windows-style and space-containing paths, and confirm `pytest tests/ -v` and `ruff check .` both pass.

---

## Implementation Notes

*To be filled in during Phase III.*

---

## Pull Request

**PR Link:** *Not yet submitted planned for Phase IV.*

**PR Description:** *To be written once implementation is complete.*

**Status:** Not started

---

## Learnings & Reflections

[To be filled upon completion]

---

## Resources Used

- https://github.com/venomez-viper/breezeml/issues/10
- https://github.com/venomez-viper/breezeml/blob/main/CONTRIBUTING.md
- https://pypi.org/project/breezeml/
