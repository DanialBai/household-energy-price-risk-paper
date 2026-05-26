# L2 → Pseudo-Huber L1 LSOA Soft Constraint: Implementation, Result, and Honest Diagnosis

> Round 2026-05-26-02 update. Read this together with
> `LSOA_income_distribution_methodology.md` and
> `LSOA_income_distribution_scenario_extension.md`.

## 1. Motivation

The earlier pipeline minimised an L2 squared error between the LSOA
lower-tail probability `P(Y_l < 19009)` and the IoD Income Score
`income_score_l`. Diagnostic review found that:

- Under L2, the optimiser drove `tail_l ≈ income_score_l` to within 0.001
  on average regardless of `lambda` on the `geomspace(1e-4, 1e-1, 100)`
  grid; `lambda` therefore had little leverage on the location of the
  optimum and only scaled the objective value.
- Selected `lambda` clustered at the grid lower edge for several MSOAs.
- In LSOAs where IoD diverges sharply from the official fuel poverty
  rate (e.g. `E01033605`, `income_score = 0.57`, `official = 0.10`),
  the squared error pulled `sigma_l` to extreme values to satisfy the
  IoD target, producing modelled fuel poverty rates far from the
  official series.

The user's intent — *"IoD is a soft low-income tail constraint, not an
equality target; use official rates only to select lambda; do not put
official rates into the objective; minimise new parameters; do not make
big methodological changes"* — argued for replacing the squared error
with a robust L1 loss that bounds the per-LSOA gradient.

## 2. Change implemented

In `scripts/income_distribution_model.py`:

```text
old LSOA-term loss:    (tail_l - income_score_l)^2
new LSOA-term loss:    rho(d) = sqrt(d^2 + tau^2) - tau,   d = tail_l - income_score_l,   tau = 1e-3
```

`tau` is a numerical regulariser to keep the loss differentiable at zero
for L-BFGS-B; it is fixed once at 1e-3 (two orders of magnitude below
the typical |d| in data) and is **not** exposed as a research parameter.

Also: tie-break in `argmin_lambda` over the grid changed from "smaller
lambda" to "larger lambda" (i.e. `key=(rmse, mae, -lam)`), applied in
both `scripts/build_lsoa_income_distributions.py` and
`scripts/build_lsoa_income_distribution_scenarios.py`.

No other constants, bounds, grids, seeds, or QA gates changed.
**Property-level definitions (SAP cost, ABC carve-out, burden_prob_gt_10)
are untouched**, and official LSOA fuel poverty rate remains outside the
objective function.

## 3. Result on the scenario sensitivity rerun (`20260526_144256`)

Read-only pipeline; master unchanged.

| Statistic (mean scenario) | OLD (L2)  | NEW (pseudo-Huber L1) |
|--------------------------|-----------|-----------------------|
| `corr(modeled, official)` LSOA | 0.173 | **0.173** |
| `corr(tail, income_score)` LSOA | 0.999 | **0.999** |
| Mean &#124;LSOA modeled − official&#124; | 0.0414 | **0.0414** |
| Max &#124;LSOA modeled − official&#124;  | 0.1327 | **0.1327** |
| Mean bias (modeled − official) | −0.0174 | **−0.0174** |
| Mean &#124;tail − income_score&#124;     | 0.0010 | **0.0010** |
| `sigma_l`: mean / median / max | 0.771 / 0.725 / 1.400 | **0.771 / 0.725 / 1.400** |
| MSOA weighted RMSE: mean / max | 0.0494 / 0.1196 | **0.0494 / 0.1196** |
| Selected lambda: min / median / max | 1.0e-4 / 1.4e-4 / 7.05e-2 | **1.0e-4 / 1.8e-4 / 3.8e-3** |
| MSOAs at grid lower edge (≤1.05e-4) | 5 / 26 | **2 / 26** |
| MSOAs ≥ 5e-2 (high λ) | 1 / 26 | **0 / 26** |

The five worst-case LSOAs are identical to three decimals in `modeled`,
`tail`, and `sigma_l` between OLD and NEW.

**Plain reading:** the fit itself did not change. Only the selected
`lambda` distribution moved — it became tighter and shifted away from
the extreme upper edge of the grid.

QA: 27 / 27 passed (input consistency + standard gates).

## 4. Honest diagnosis of why the fit did not change

The L1 change was insufficient because of a deeper structural property
that the earlier analysis underweighted:

**`sigma_l` is over-determined by the LSOA term alone.** The MSOA-mean
log-anchor term depends only on `mean_l`, not on `sigma_l`. Once `mean_l`
collapses to the MSOA anchor (which it does at any positive `lambda`),
nothing in the objective penalises `sigma_l` except the IoD soft
constraint. The optimum of *any* monotone loss in `|tail_l −
income_score_l|` is then at `tail_l ≈ income_score_l`, regardless of
whether the loss is L2, L1, pseudo-Huber, or any other monotone
transformation.

- L2 and L1 share the same minimiser when only one direction of the
  parameter space carries the loss gradient.
- Lambda still has formal leverage in the sense that the objective value
  varies with lambda, but the *location* of the optimum (`sigma_l`,
  hence `tail_l`, `modeled_lsoa_rate_l`) is essentially insensitive to
  lambda's level once lambda is positive.
- This is why `corr(tail, income_score) = 0.999` survives the L1
  change.

The L1 change is therefore **not a useful empirical improvement to the
fit** under the current MSOA mean anchoring scheme. It remains
**methodologically defensible** as a robust soft constraint, but the
substantive identification of LSOA dispersion is essentially the same.

## 5. What did change, and why it still matters

1. **Selected lambda distribution tightened**: max from 7.05e-2 to
   3.8e-3, with no MSOAs left in the high-lambda regime. Under the L1
   loss, the LSOA-term value at the optimum is bounded; the official
   RMSE curve no longer prefers very large lambda for any MSOA. The
   selected lambda is a cleaner, more informative diagnostic than under
   L2.
2. **Methodological framing is honest**: the soft constraint is L1, the
   selection rule prefers larger lambda when ties occur, and the
   property-level definitions are unchanged. The paper can claim "IoD
   enters as a robust soft constraint on the lower-income tail, with
   regularisation strength identified ex post by official fuel poverty
   rates" without overstating empirical leverage.
3. **Structural limitation surfaced**: the `tail = income_score`
   equivalence under any monotone loss is *information about the
   parameterisation*. It tells us that the residual modelled-vs-official
   gap is genuinely about *IoD vs official fuel poverty being different
   constructs*, not about loss specification. This is a defensible
   limitation to write into the methods section, not a tuning failure.

## 6. Options for next steps (not implemented in this round)

If we later want the modelled-vs-official scatter to move toward the 45°
line, the only directions that would help are **structural changes to
how `sigma_l` is identified**, not further tweaks to the LSOA loss form.
All of the below introduce at least one new parameter or assumption and
should only be considered after discussion:

1. **Couple `sigma_l` to a borough-wide prior or shrinkage target**, so
   `sigma_l` is not solely determined by IoD. One new parameter
   (shrinkage strength, or prior mean of `sigma`).
2. **Move some IoD-vs-official discipline into the MSOA term** — e.g.
   penalise the modelled MSOA-aggregate fuel poverty rate against the
   MSOA-aggregate official rate. This stays out of the LSOA-level
   objective but uses official rates at MSOA level. One new parameter
   (relative weight).
3. **Replace `Income Score` with a tail-relevant covariate that
   correlates with official fuel poverty** at LSOA level (e.g. a fuel
   poverty proxy from local benefits uptake). Methodological change.
4. **Accept the structural finding as a limitation** and write the paper
   accordingly. The figures and tables are diagnostic, not a calibration
   pass / fail.

The user's stated preferences (minimise parameters, no big method
change) point toward option 4 as the default. Options 1–3 should only
be revisited if a reviewer pushes back on the residual gap.

## 7. Outputs from this round

- Code: `scripts/income_distribution_model.py` (loss change);
  `scripts/build_lsoa_income_distributions.py` (tie-break);
  `scripts/build_lsoa_income_distribution_scenarios.py` (tie-break).
- Scenario workbook:
  `outputs/lsoa_income_distribution_scenarios_20260526_144256.xlsx`
  (QA 27/27 passed; 124,980 master rows read-only).
- Plot directory:
  `outputs/income_scenario_plots_20260526_144256/` (3 CDF + 3 PDF + 3
  summary PNGs, 2 outlier CSVs, 1 LSOA colour key).
- Documentation updates:
  - `materials/LSOA_income_distribution_methodology.md` § 4.2 / § 4.3.
  - `materials/LSOA_income_distribution_scenario_extension.md` § 2 / § 3.
  - This file.

## 8. Pending: canonical pipeline rerun on master

The canonical pipeline `scripts/build_lsoa_income_distributions.py` has
the same code path change, but **has not been rerun**; the master
workbook `data/all_splits_income_scenarios.xlsx` still carries the
fourteen L2-derived columns from run `20260520_213323`. Given the
finding that L1 and L2 produce numerically near-identical fits under
the current parameterisation, rerunning the canonical pipeline would
produce essentially the same master columns and is therefore deferred
until either a downstream user explicitly requires bit-for-bit
consistency with the new loss expression, or a future methodological
change (per § 6) actually changes the fit.

If a rerun is requested, the steps are:

```powershell
cd "D:\Documents\2025\Research\Journal\third paper\UK household risk"
python -W ignore::DeprecationWarning scripts/build_lsoa_income_distributions.py
```

A timestamped backup of the master is written automatically before
overwrite, per the canonical pipeline.
