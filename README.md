# Contribution #2: Normalize Windows paths in export()

**Contribution Number:** 2
**Student:** Jenisha Subedi
**Issue:** https://github.com/venomez-viper/breezeml/issues/10
**Status:** Phase II COMPLETED

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
cloned my fork (`jenishasubedi05/breezeml`), ran `pip install -e ".[dev]"` to install BreezeML and dev dependencies (pytest, ruff, etc.) locally on Windows. Confirmed the existing test suite passes before making any changes:
python -m pytest tests/ -v
Result: 51 passed, 4 skipped (skips are optional XGBoost/LightGBM tests, not installed).

One setup note: `pytest` wasn't recognized directly from PowerShell since its install location wasn't on PATH. Worked around it by running `python -m pytest` instead of `pytest` directly.

Working branch: https://github.com/jenishasubedi05/breezeml/tree/fix-issue-10

### Steps to Reproduce

1. Train a model: `model = fit(datasets.iris(), "species")`
2. Export it with a Windows-style data path: `export(model, "train.py", data_path=r"C:\Users\College\data.csv")`
3. Open the generated `train.py` and find the line: `DATA_PATH = "C:\Users\College\data.csv"`
4. Run `python train.py`
5. **Expected:** Script loads the CSV and trains normally
6. **Actual:** `SyntaxError: (unicode error) 'unicodeescape' codec can't decode bytes in position 2-3: truncated \UXXXXXXXX escape` — the generated script fails to even parse

Reproduced consistently (twice) with the same error.

### Reproduction Evidence

Branch with reproduction script (`repro.py`): https://github.com/jenishasubedi05/breezeml/tree/fix-issue-10

---

## Solution Approach

### Analysis

Root cause is in `breezeml/export.py`, in `export_code()`:

```python
lines.append(f'DATA_PATH = "{data_path}"')
```

This drops the raw `data_path` string directly into the generated Python source wrapped in double quotes. Windows paths contain backslashes, which Python interprets as escape characters inside string literals. Sequences like `\U` are parsed as the start of an (invalid) 8-digit unicode escape, causing a `SyntaxError` before the exported script can even run.

### Proposed Solution

1. In `export_code()`, stop manually wrapping `data_path` in quotes with an f-string
2. Use `repr(data_path)` instead of `f'"{data_path}"'` — `repr()` correctly escapes backslashes so the resulting string literal is always valid Python, regardless of OS
3. Add a test in `tests/test_v040_features.py` that exports with a Windows-style path (`r"C:\Users\name\data.csv"`) and confirms the generated script parses and runs without error

### Implementation Plan

**Understand:** An exported training script can contain an improperly escaped Windows path, risking a broken or invalid script.

**Match:** Follow the existing pattern in `export()` for how other pipeline attributes (imputers, scaler, encoder, seed) are serialized into the generated script.

**Plan:**
1. Reproduce the issue locally with a Windows-style path (done — confirmed above)
2. Identify the exact line in `export()` responsible (done — `lines.append(f'DATA_PATH = "{data_path}"')`)
3. Normalize the path before writing, using `repr(data_path)` instead of manual quote-wrapping
4. Add/extend a test confirming exported scripts are valid Python for Windows-style paths
5. Run the full test suite and `ruff check .` before submitting

**Implement:** *Not yet started planned for Phase III.*

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
