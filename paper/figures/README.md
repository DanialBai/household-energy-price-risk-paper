# Manuscript Figures

Use this folder only for figures that are confirmed for the LaTeX manuscript.

## What Belongs Here

- Figures referenced from `main.tex` or files in `sections/`.
- Final or near-final images with stable names.
- Publication-facing diagrams, maps, and charts.

## Naming

Use manuscript-style names:

```text
fig_01_conceptual_framework.png
fig_02_westminster_case_map.png
fig_03_price_shock_exposure_distribution.png
```

## Figure Notes

For each figure, keep enough information to reproduce or audit it:

```text
Source data:
Script or source project:
Generated/imported on:
Purpose in manuscript:
```

Exploratory or repeated debug outputs should stay in `outputs/` and should not be committed by default.

## Imported Figure Notes

### Baseline income PDF scenario panels

Source data: `outputs/lsoa_income_distribution_scenarios_20260526_144256.xlsx` from the related Westminster household risk project.

Script or source project: `plot_lsoa_income_distribution_scenarios.py` in project one.

Generated/imported on: 2026-05-26.

Purpose in manuscript: Diagnostic LSOA income-distribution sensitivity panels for the baseline burden model; referenced in `sections/05_expected_outputs.tex`.
