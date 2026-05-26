# 2026-05-26 baseline-burden-model

## Context

今日工作属于 Westminster household fuel price risk 项目的 baseline burden model 前置方法复核：在进入 `2.2 Clean baseline modelling dataset extract` 和 property-level baseline required energy burden 之前，先检查 LSOA 收入分布引擎、MSOA income anchor 场景敏感性、LSOA soft constraint 形式，以及最新可解释性图件是否足够支撑后续 burden/risk modelling。

本轮不是新的 fuel price shock engine，也不是 intervention simulation。它仍然处于 required-energy-based baseline burden model 的收入分布与 poverty-proxy 校准阶段。需要继续保留三组概念边界：`fuel poverty` 是当前官方/政策口径下的贫困状态，`energy burden` 是 required energy cost / residual income，`household fuel price risk` 是未来价格冲击下的 exposure + sensitivity - adaptive capacity。

## Research Log

今日共有三轮工作记录在 `TASKS.md`：

- `Round 2026-05-26-01`: 建立 MSOA income anchor 三场景敏感性分析，使用 `mean` / `ucl` / `lcl` 三个收入锚点，master read-only。
- `Round 2026-05-26-02`: 加入输入一致性 QA gates、LSOA colour key、outlier tables，并更新 CDF/PDF/summary 图的可解释性。
- `Round 2026-05-26-03`: 将 LSOA soft constraint 从 L2 squared loss 改为 pseudo-Huber L1，并改变 lambda tie-break 规则；完成 `20260526_144256` read-only rerun 与图件输出。

最新结果目录为 `outputs/income_scenario_plots_20260526_144256/`；最新场景 workbook 为 `outputs/lsoa_income_distribution_scenarios_20260526_144256.xlsx`。该 run 读取 `data/all_splits_income_scenarios.xlsx` 124,980 行，覆盖 126 LSOAs 和 26 MSOAs，未改写 master。

## Method Update

1. LSOA income distribution scenario extension remains a sensitivity branch, not a replacement baseline.

`mean`, `ucl`, and `lcl` use MSOA net income after housing costs point estimate and confidence limits as alternative income anchors. They bracket uncertainty in the MSOA anchor, but should not be described as household income forecasts. The canonical property-level master still carries the 20260520 baseline columns until a deliberate canonical rerun is approved.

2. LSOA soft constraint changed from L2 to pseudo-Huber L1.

The LSOA tail-probability constraint now uses:

```text
rho(d) = sqrt(d^2 + tau^2) - tau
d = P(Y_l < 19009) - income_score_l
tau = 1e-3
```

`tau` is only a numerical differentiability regulariser for L-BFGS-B, not a research parameter. This keeps IoD Income Score as a robust soft lower-income-tail constraint rather than an equality target.

3. Lambda selection tie-break changed.

The selection rule remains based on official LSOA fuel poverty rates as an ex-post fit diagnostic, but ties now prefer larger lambda:

```text
selected_lambda = argmin(weighted_rmse, weighted_mae, -lambda)
```

This change was applied in both the canonical script and the scenario script. Official fuel poverty rates are still not placed directly inside the LSOA-level objective.

4. Structural diagnosis.

The L1 change did not materially change the fitted LSOA distributions. The reason is structural: with the current parameterisation, the MSOA term anchors `mean_l`, while `sigma_l` is effectively identified by the LSOA tail constraint alone. Therefore any monotone loss in `|tail_l - income_score_l|` drives `tail_l` close to `income_score_l`. The residual modelled-vs-official gap should be written as a limitation of the proxy relationship between IoD lower-income tail and official fuel poverty, not as a tuning failure.

## Parameter Settings

- Unit basis: property-level master read in full, but scenario output is LSOA/MSOA-level; property-level burden scenario columns are deferred.
- Required energy basis: SAP-derived required energy cost logic remains unchanged.
- Poverty line used in income-tail fitting: GBP 19,009.
- Scenarios: `mean`, `ucl`, `lcl`.
- Sigma bounds: `(0.10, 1.4)`.
- Lambda grid: `np.geomspace(1e-4, 1e-1, 100)`.
- Optimiser: L-BFGS-B.
- LSOA loss: pseudo-Huber L1 with `tau = 1e-3`.
- ABC carve-out and `burden_prob_gt_10` definitions: unchanged.
- Master write status: read-only for this scenario run; canonical master not rewritten after the L1 change.

## Results

Latest run `20260526_144256`:

- Master rows read: 124,980.
- Master modified: no.
- Scenarios: 3.
- LSOAs: 126.
- MSOAs: 26.
- QA: 27 / 27 passed, including 6 input-consistency gates and 21 model gates.
- Selected lambda across all scenarios: min 0.000100, max 0.003765.
- Sigma range: 0.3411 to 1.4000.
- Mean income vs anchor ratio: 0.998959 to 1.002008.

Scenario-level weighted RMSE:

- `mean`: mean 0.04944, median 0.04550, max 0.11959.
- `ucl`: mean 0.05022, median 0.04765, max 0.12635.
- `lcl`: mean 0.04821, median 0.04585, max 0.11101.

Scenario-level weighted MAE:

- `mean`: mean 0.04474, median 0.04107, max 0.11959.
- `ucl`: mean 0.04562, median 0.04111, max 0.12635.
- `lcl`: mean 0.04339, median 0.03756, max 0.11101.

Largest tail-probability scenario shifts:

- `E01033602` in `E02000968`: mean tail probability 0.66836; ucl 0.61543; lcl 0.69100; max absolute shift 0.05293.
- `E01004665` in `E02000983`: max absolute shift 0.04433.
- `E01004732` in `E02000979`: max absolute shift 0.04360.
- `E01032512` in `E02000969`: max absolute shift 0.04196.

Largest modelled-vs-official poverty-proxy residuals under the `mean` scenario:

- `E01033605` in `E02000968`: modelled 0.23349 vs official 0.10081; error +0.13268.
- `E01000586` in `E02000124`: modelled 0.22575 vs official 0.10616; error +0.11959.
- `E01035722` in `E02000983`: modelled 0.01535 vs official 0.12876; error -0.11341.
- `E01004700` in `E02000975`: modelled 0.03840 vs official 0.14507; error -0.10667.
- `E01004710` in `E02000961`: modelled 0.17353 vs official 0.07831; error +0.09523.

## Interpretation and Discussion

今天最重要的结论不是“L1 改进了拟合”，而是“L1 没有改变拟合，这揭示了当前收入分布模型的结构”。在现有 MSOA mean anchor + LSOA income-tail constraint 下，`mean_l` 被 MSOA income anchor 锚住，`sigma_l` 主要由 IoD Income Score 的 lower-tail target 决定。因此，L1/L2 的变化不会显著改变 `tail_l`、`sigma_l` 或 modelled poverty proxy。

这对论文是有价值的：它说明 residual gap 不是简单的 loss-function artefact，而是 IoD income deprivation 与 official fuel poverty rate 之间概念差异的经验体现。换言之，模型没有把 fuel poverty、energy burden 和 household fuel price risk 混成一个指标；它用 IoD 约束 income-tail exposure，用 SAP required cost 支撑 burden probability，再用 official fuel poverty rate 做外部 fit diagnostic。

默认建议是接受这一结构性发现作为方法限制，而不是立即引入新的 shrinkage prior 或 official-rate objective。这样更符合当前研究策略：尽量少引入研究参数，保留 required energy demand 的透明性，并把后续贡献放在 property-level energy burden、price shock exposure、intervention de-risking dividend 上。

需要特别注意：这些图和表现在仍然是 baseline burden model 的收入分布前置诊断，不应写成 price risk 结果。price risk 需要后续价格冲击引擎和 risk metrics 才能成立。

## Figures Synced

本次同步全部来自 `outputs/income_scenario_plots_20260526_144256/` 的 PNG 结果图：

- `lsoa_income_cdf_scenarios_fig1_of_3_20260526_144256.png`: LSOA income CDF scenario panels, figure 1 of 3.
- `lsoa_income_cdf_scenarios_fig2_of_3_20260526_144256.png`: LSOA income CDF scenario panels, figure 2 of 3.
- `lsoa_income_cdf_scenarios_fig3_of_3_20260526_144256.png`: LSOA income CDF scenario panels, figure 3 of 3.
- `lsoa_income_pdf_scenarios_fig1_of_3_20260526_144256.png`: LSOA income PDF scenario panels, figure 1 of 3.
- `lsoa_income_pdf_scenarios_fig2_of_3_20260526_144256.png`: LSOA income PDF scenario panels, figure 2 of 3.
- `lsoa_income_pdf_scenarios_fig3_of_3_20260526_144256.png`: LSOA income PDF scenario panels, figure 3 of 3.
- `modeled_vs_official_poverty_by_scenario_20260526_144256.png`: Modelled poverty proxy versus official LSOA fuel poverty rate by scenario.
- `msoa_rmse_mae_by_scenario_20260526_144256.png`: MSOA weighted RMSE / MAE comparison across scenarios.
- `tail_prob_scenario_shift_20260526_144256.png`: Mean-vs-ucl/lcl lower-tail probability shift by LSOA.

辅助表一并列入 Source Trace，但不作为 figures 复制：

- `lsoa_color_key_20260526_144256.csv`.
- `modeled_official_error_outliers_20260526_144256.csv`.
- `tail_prob_scenario_shift_outliers_20260526_144256.csv`.

Suggested caption:

> LSOA income distribution scenario sensitivity under alternative MSOA income anchors. The figures compare central, upper-confidence-limit, and lower-confidence-limit MSOA income anchors while holding the IoD lower-tail constraint, SAP required-energy cost logic, and official fuel-poverty diagnostic unchanged. The results show that income-anchor uncertainty shifts lower-tail probabilities in selected LSOAs, but the modelled-vs-official poverty-proxy residual remains structurally persistent.

Caveat:

> The `ucl` and `lcl` anchors are calibration sensitivity checks, not forecasts. These figures diagnose income-distribution assumptions for the baseline burden model; they do not yet measure household fuel price risk under future price shocks.

## Paper-writing Implications

1. 方法部分可以写成：LSOA income distributions are calibrated to MSOA after-housing-cost income anchors and constrained by IoD lower-tail evidence, with official LSOA fuel poverty rates used as an external diagnostic for lambda selection.

2. robustness subsection 可以说明：MSOA income uncertainty shifts lower-tail probabilities for some LSOAs but leaves borough-level fit patterns broadly stable.

3. limitations subsection 应明确：official fuel poverty and IoD income deprivation are not the same construct; residual modelled-vs-official gaps are expected, especially where housing cost, dwelling efficiency, tenure, and fuel system conditions diverge from income deprivation.

4. 后续 baseline burden model 应继续使用 required energy demand，而不是 actual consumption，因为 actual consumption 会低估 underheating/self-rationing households 的 need。

5. 在进入 heat pump / electrification / PV / tariff intervention 之前，必须先完成 property-level baseline required energy burden 和 shock scenario engine。当前结果不能直接支持“net zero 自动 de-risk”的结论。

## Paper-ready English Notes

The income distribution module treats small-area income deprivation as a soft constraint on the lower tail of the LSOA income distribution, rather than as an equality target. Official LSOA fuel poverty rates are retained as an external diagnostic for selecting regularisation strength and evaluating residual fit.

The May 2026 robustness run replaced the L2 lower-tail loss with a pseudo-Huber L1 loss. This change did not materially alter the fitted LSOA distributions, which indicates that the remaining modelled-vs-official gap is structural rather than a simple consequence of the loss function.

This finding supports a transparent limitation: income deprivation, modelled required-energy burden, and official fuel poverty are related but non-equivalent constructs. The baseline burden model therefore uses the income engine to represent exposure and sensitivity, while reserving household fuel price risk for subsequent price-shock scenarios and intervention simulations.

Net zero interventions should not be interpreted as automatic household protection. Their de-risking value can only be assessed after the baseline required-energy burden, tariff/price-shock engine, and intervention packages are modelled at property or household-dwelling level.

## Source Trace

- `TASKS.md`
- `materials/LSOA_income_distribution_methodology.md`
- `materials/LSOA_income_distribution_scenario_extension.md`
- `materials/LSOA_income_robust_soft_constraint_change.md`
- `scripts/income_distribution_model.py`
- `scripts/build_lsoa_income_distributions.py`
- `scripts/build_lsoa_income_distribution_scenarios.py`
- `scripts/plot_lsoa_income_distribution_scenarios.py`
- `outputs/lsoa_income_distribution_scenarios_20260526_144256.xlsx`
- `outputs/income_scenario_plots_20260526_144256/lsoa_color_key_20260526_144256.csv`
- `outputs/income_scenario_plots_20260526_144256/modeled_official_error_outliers_20260526_144256.csv`
- `outputs/income_scenario_plots_20260526_144256/tail_prob_scenario_shift_outliers_20260526_144256.csv`

## Open Questions

- 是否接受当前 residual modelled-vs-official gap 作为方法限制，并进入 `2.2 Clean baseline modelling dataset extract`？
- 是否需要 canonical pipeline rerun，让 master workbook 的 columns 在数值上也对应 pseudo-Huber L1 版本？当前诊断认为 L1 与 L2 结果近似一致，因此默认可暂缓。
- property-level scenario `burden_prob_gt_10` 是否要在三种 income anchors 下做 read-only extract，用于后续 HR/BR high-risk flag 稳健性比较？

## Next Steps

1. 若接受结构性限制，进入 `2.2 Clean baseline modelling dataset extract`。
2. 生成 property-level baseline required energy burden，明确 required energy cost / residual income 的 baseline distribution。
3. 建立 gas/electricity/combined price shock scenario engine。
4. 在价格冲击后计算 Fuel Price-at-Risk、threshold-crossing、affordability gap-at-risk。
5. 之后再评估 retrofit、clean heat、PV、battery、smart tariffs 和 integrated packages 的 conditional de-risking dividend。
