# 2026-05-28 Risk Metric Layer v1

## Context

This sync records the first formal household fuel price risk metric layer built from Fuel Price Shock Scenarios v1. The stage converts event-level price shock outputs into compact property-level and aggregate indicators that can be used in the paper as a named risk framework and later as the baseline comparator for intervention simulations.

The layer preserves the project's conceptual distinction:

- Fuel poverty: current affordability hardship under official or policy definitions.
- Energy burden: required energy cost relative to residual household income.
- Household fuel price risk: future exposure under price shock scenarios, expressed through tail burden, threshold-crossing, and affordability gap metrics.

## Research Log

- The user selected the "method indicator" route rather than immediately moving to intervention simulation.
- The approved design uses a compact property-level extract plus LSOA, MSOA, and archetype aggregates.
- The main risk scenario set is S2, S3, and S6 Q3.
- S0 is retained as baseline reference; S4 is retained as policy mitigation comparison; S1 is retained as sensitivity.
- Direct Debit is the central payment-method track. Payment method remains a sensitivity scenario, not observed household assignment.
- During implementation review, a key data-integrity issue was found: `UPRN` is not unique in the v1 event table. The final risk layer therefore uses a generated `risk_property_id` to preserve 124,477 valid repricing property rows rather than collapsing duplicate UPRNs.

## Method Update

Risk Metric Layer v1 is built from:

- `outputs/fuel_price_shock_scenarios_v1/property_event_results_v1.csv`
- `scripts/build_risk_metric_layer_v1.py`
- `scripts/plot_risk_metric_layer_figures.py`

The core output directory is:

```text
outputs/risk_metric_layer_v1/
```

Core metrics:

- `FPaR_P90` / `FPaR_P95`: tail quantiles of required energy burden across S2, S3, and S6 Q3.
- `threshold_crossing_risk_gt_10`: share of the main risk scenarios in which required energy burden exceeds 10%.
- `AGaR_P90` / `AGaR_P95`: tail quantiles of annual affordability gap in GBP/year across S2, S3, and S6 Q3.
- `FPaR_P90_with_S1` and `AGaR_P90_with_S1`: sensitivity metrics adding S1.
- `s4_burden_reduction_vs_s3` and `s4_burden_residual_vs_s3`: S4 policy mitigation comparison against S3.

Interpretation caveat: these are required-energy risk metrics. They are not observed bills, not official fuel poverty statistics, and not intervention results.

## Parameter Settings

- Main payment method: Direct Debit.
- Main risk scenario set: S2, S3, S6 Q3.
- Baseline reference: S0.
- Policy mitigation comparison: S4.
- Sensitivity scenario: S1.
- Main paper indicator: `FPaR_P90`.
- Sensitivity tail indicator: `FPaR_P95`.
- Policy interpretation indicator: `AGaR_P90`.
- Threshold: 10% required energy burden.

## Results

Risk Metric Layer v1 outputs:

- `property_risk_metrics_v1.csv`: 124,477 rows.
- `lsoa_risk_metrics_v1.csv`: 126 rows.
- `msoa_risk_metrics_v1.csv`: 26 rows.
- `archetype_risk_metrics_v1.csv`: 72 rows.
- `risk_metric_qa_summary_v1.csv`: 14/14 PASS.

Property-level summary:

- Mean `FPaR_P90`: 7.45% of required income.
- Median `FPaR_P90`: 6.34%.
- P90 of property-level `FPaR_P90`: 12.39%.
- Mean `threshold_crossing_risk_gt_10`: 10.34%.
- P90 of property-level `AGaR_P90`: about GBP 1,202/year.
- Mean S4/S3 burden residual: 74.11%, indicating that policy mitigation reduces but does not eliminate S3 risk.

Top LSOAs by mean `FPaR_P90` include:

- E01004646: 12.98%; threshold risk 32.81%; mean `AGaR_P90` about GBP 1,914/year.
- E01004720: 11.78%; threshold risk 32.02%; mean `AGaR_P90` about GBP 1,000/year.
- E01033605: 11.25%; threshold risk 30.99%; mean `AGaR_P90` about GBP 744/year.
- E01004719: 11.15%; threshold risk 28.04%; mean `AGaR_P90` about GBP 761/year.

Top archetypes by mean `FPaR_P90` are dominated by gas-heated houses and maisonettes with weaker EPC performance:

- Owner occupied | House | E-G | Gas: FPaR P90 19.3%; threshold risk 71.1%; AGaR P90 about GBP 4,340/year; S4/S3 residual 74.7%.
- Private rented | House | E-G | Gas: FPaR P90 16.5%; threshold risk 57.8%; AGaR P90 about GBP 3,112/year; S4/S3 residual 74.8%.
- Owner occupied | Maisonette | E-G | Gas: FPaR P90 15.0%; threshold risk 51.8%; AGaR P90 about GBP 2,561/year; S4/S3 residual 74.2%.
- Owner occupied | House | D | Gas: FPaR P90 14.4%; threshold risk 47.9%; AGaR P90 about GBP 2,370/year; S4/S3 residual 74.4%.

## Interpretation and Discussion

This stage moves the empirical analysis from scenario-specific burden uplift into a formal household fuel price risk layer. The main contribution is not only that some households face higher costs under S3; it is that risk can now be expressed as a tail burden metric (`FPaR_P90`), an intuitive threshold-crossing probability, and a GBP-denominated affordability gap-at-risk.

The results reinforce the paper's central thesis. Policy support can reduce the severity of price shock exposure, but S4 does not erase the structural concentration of risk. The highest-risk archetypes remain shaped by fuel, fabric efficiency, dwelling form, tenure, and income vulnerability.

The S1 sensitivity fields also help separate the main shock-risk narrative from milder price-change sensitivity. This is useful for defending the paper's scenario design: the main text can focus on S2/S3/S6 Q3 while retaining S1 as a transparent robustness check.

## Figures Synced

1. Figure 1: FPaR P90 spatial distribution
   - Source: `outputs/risk_metric_layer_v1/figures/figure_01_fpar_p90_spatial_distribution.png`
   - Also synced as PDF for publication workflow.
   - Description: LSOA-level mean FPaR P90 across Westminster.
   - Suggested caption: "Spatial distribution of mean household Fuel Price-at-Risk P90 across Westminster LSOAs, expressed as required energy burden under the main risk scenario set."
   - Caveat: 123 of 126 result LSOAs matched the project GIS file. Unmatched result LSOAs are `E01000586`, `E01002885`, and `E01002886`.

2. Figure 2: Top archetype risk metrics
   - Source: `outputs/risk_metric_layer_v1/figures/figure_02_top_archetype_risk_metrics.png`
   - Also synced as PDF for publication workflow.
   - Description: top archetypes ranked by FPaR P90, with threshold risk and AGaR P90.
   - Suggested caption: "Top Westminster household-dwelling archetypes ranked by Fuel Price-at-Risk P90, with corresponding threshold-crossing risk and affordability gap-at-risk."

3. Figure 3: Policy mitigation residual
   - Source: `outputs/risk_metric_layer_v1/figures/figure_03_policy_mitigation_residual.png`
   - Also synced as PDF for publication workflow.
   - Description: S4 residual burden relative to S3 for the same high-risk archetypes.
   - Suggested caption: "Residual S4 burden relative to S3 among high-risk archetypes, showing that policy mitigation reduces but does not eliminate structurally concentrated household fuel price risk."

## Paper-writing Implications

This stage can support a dedicated methodology/results bridge subsection:

- "Operationalising Household Fuel Price-at-Risk"
- "Tail burden, threshold risk, and affordability gap-at-risk"
- "Policy mitigation and residual risk"

The paper can now state that the Westminster model does not simply report scenario bills. It constructs a compact property-level risk layer that translates price shock scenarios into tail burden, exceedance, and gap metrics.

The next modelling stage can use these same metrics before and after intervention packages to calculate de-risking dividends.

## Paper-ready English Notes

Risk Metric Layer v1 operationalises household fuel price risk as a property-level tail-risk framework. Rather than treating price shocks as isolated bill changes, it converts the S2, S3 and S6 Q3 scenarios into Fuel Price-at-Risk, threshold-crossing risk and affordability gap-at-risk metrics.

FPaR P90 is the primary manuscript indicator because it captures the upper tail of required energy burden across the main risk scenario set. Threshold-crossing risk translates this into an intuitive 10% affordability benchmark, while AGaR P90 expresses the same risk in annual GBP shortfall terms.

The results show that risk remains structurally concentrated. Gas-heated, less efficient houses and maisonettes face the highest FPaR values, while S4 policy mitigation reduces but does not eliminate residual exposure. This supports the paper's argument that household de-risking is conditional: policy support and net zero interventions must be assessed by who receives protection and who remains exposed.

## Source Trace

- Design: `docs/superpowers/specs/2026-05-28-risk-metric-layer-v1-design.md`
- Implementation plan: `docs/superpowers/plans/2026-05-28-risk-metric-layer-v1.md`
- Build script: `scripts/build_risk_metric_layer_v1.py`
- Plot script: `scripts/plot_risk_metric_layer_figures.py`
- Tests: `tests/test_risk_metric_layer.py`; `tests/test_risk_metric_layer_figures.py`
- Metric notes: `outputs/risk_metric_layer_v1/risk_metric_notes_v1.md`
- Figure notes: `outputs/risk_metric_layer_v1/figures/risk_metric_figure_notes.md`
- QA: `outputs/risk_metric_layer_v1/risk_metric_qa_summary_v1.csv`
- Core outputs: `property_risk_metrics_v1.csv`, `lsoa_risk_metrics_v1.csv`, `msoa_risk_metrics_v1.csv`, `archetype_risk_metrics_v1.csv`
- Round log: `TASKS.md`, Round 2026-05-28-06.

## Open Questions

- Should this stage be written up before or after the first intervention simulation?
- Should S1 sensitivity be presented in an appendix table or only mentioned in methods?
- Should payment-method sensitivity become a separate appendix risk-metric layer?
- Should the three unmatched LSOAs be resolved with an updated Westminster LSOA boundary file?

## Next Steps

1. Review the three risk metric figures for manuscript use.
2. Decide whether to draft the FPaR methodology/results subsection now.
3. Use Risk Metric Layer v1 as the baseline comparator for the first intervention simulation and de-risking dividend calculation.
