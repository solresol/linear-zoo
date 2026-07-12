# IMPROVEMENTS.md

*Analysis date: 2026-07-11*

linear-zoo is a small pedagogical repository whose entire payload is a single Jupyter
notebook, `linear-zoo.ipynb`, demonstrating "exotic and traditional" linear regressions
(OLS, Theil-Sen, RANSAC, Siegel, and two p-adic variants) on a six-point toy dataset,
plus a generated plot (`toydata-regression.png`) and LaTeX table (`toydata-data.tex`).
It is three commits old, has a clean working tree, and works — but it has no dependency
management, no tests, no way to run it non-interactively, and a few housekeeping warts.

## Bugs & Fixes

- **Empty trailing cells** — the last two code cells in `linear-zoo.ipynb` are empty.
  Delete them; they are noise in diffs and in rendered views.
- **Fragile p-adic claim** — the notebook itself says "I'm not sure if this is
  guaranteed to get the right answer" about `PadicLinearSumMinimizer`. Either prove/cite
  the property, bound the search, or add a brute-force cross-check cell so readers know
  the demonstrated answer is actually the optimum.
- **`type(v) == fractions.Fraction`** in `nice_format` — use
  `isinstance(v, fractions.Fraction)`; the equality check breaks on subclasses and is
  un-Pythonic.
- **Old-style super calls** — `LinearModel.__init__(self)` / `SklearnModel.__init__(self)`
  everywhere; several `__init__` overrides do nothing but chain. Use `super().__init__()`
  or drop the redundant overrides entirely.

## Improvements

- **Extract a module** — move the model classes (`LinearModel`, `SklearnModel`, `OLS`,
  `TheilSen`, `RANSAC`, `Siegel`, `PadicModel`, …) into `linear_zoo.py` and have the
  notebook import them. That makes the code testable, importable, and diffable, while
  the notebook stays the narrative.
- **Seed randomness** — RANSAC (and Theil-Sen subsampling) are stochastic; pass
  `random_state=` so the notebook, plot, and LaTeX table are reproducible run-to-run.
- **Regenerate outputs from a script** — `toydata-regression.png` and `toydata-data.tex`
  are committed build products. Add a small `make_outputs.py` (run via `uv run
  make_outputs.py` or `jupyter nbconvert --execute`) so they can be regenerated
  deterministically, and note in README that they are generated.
- **More zoo animals** — obvious candidates that fit the theme: LAD/quantile regression
  (L1), Huber, Deming/total least squares, Bayesian linear regression, and repeated
  median. Each is a few lines given the existing class scaffolding.

## Testing

- No tests exist. After extracting `linear_zoo.py`, add `tests/test_models.py` with:
  exact OLS slope/intercept on the toy dataset (closed form), Theil-Sen/Siegel median
  properties, and a brute-force verification of the p-adic minimizers over a small
  rational grid. Run with `uv run pytest`.
- Add a CI smoke test (GitHub Actions) that executes the notebook end-to-end with
  `nbconvert --execute` so dependency drift is caught.

## Documentation

- **README is four lines.** Add: what each model is (one line each), the rendered
  `toydata-regression.png` inline, how to set up and run (`uv sync`, `uv run jupyter
  lab`), and a pointer to the p-adic discussion — that is the genuinely novel content
  and deserves a paragraph or a link to a write-up.
- `CITATION.cff` exists (good) — reference it from the README ("how to cite").

## Security

- No secrets or credentials found. Nothing network-facing. No concerns beyond keeping
  notebook outputs free of local paths.

## Housekeeping / Modernization

- **Delete `CITATION.cff~`** — an editor backup file is committed at the repo root.
  Remove it and add `*~` (plus `.ipynb_checkpoints/`, `__pycache__/`) to a new
  `.gitignore`.
- **No dependency management at all** — the notebook imports numpy, pandas,
  scikit-learn, and matplotlib with nothing declaring them. Create a `pyproject.toml`
  with `uv init` / `uv add numpy pandas scikit-learn matplotlib jupyter` and commit
  `uv.lock`. Do **not** add a requirements.txt. Run everything via `uv run`.
- No remote is configured beyond the current origin state; verify `git remote -v` and
  push so this survives the laptop.

## Quick Wins

1. `git rm CITATION.cff~` and add `.gitignore` (2 minutes).
2. Delete the two empty trailing notebook cells.
3. `uv init` + `uv add` the four dependencies; update README run instructions.
4. Fix `nice_format` isinstance check and seed RANSAC.
