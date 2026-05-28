# 2026-05-28 Fuel Price Shock Results Figures

## Context

This sync records the first paper-facing result figure set from Fuel Price Shock Scenarios v1 for the Westminster household fuel price risk paper. The figures are designed to support a results narrative rather than only a modelling QA narrative:

```text
Exposure is structurally produced by fuel, fabric, dwelling form, and tenure
-> specific archetypes face the largest price-sensitive burden uplift
-> policy and tariff scenarios reshape but do not erase unequal exposure
-> the resulting risk is spatially uneven across Westminster
```

The figure set preserves the project distinction between fuel poverty, energy burden, and household fuel price risk. The results use required energy costs from the baseline/shock engine rather than actual observed energy spending.

## Research Log

- The user approved a four-figure main-results package after several rounds of design discussion.
- The selected narrative option was "Who is exposed, and why", with an added spatial Figure 4.
- The central payment-method track is Direct Debit. Standard Credit and Prepayment remain better suited to appendix/sensitivity figures because observed household payment method is not available.
- The main scenarios are S3, S4, and S6 Q3, with S0 and S2 included in scenario-profile Figure 3 as anchors.
- Existing project GIS was used for the spatial figure: Westminster borough boundary and Westminster LSOA boundary.
- The plotting script generated PNG and PDF versions of all four figures, figure source tables, a data summary, QA summary, and notes.

## Method Update

The figure workflow uses the v1 shock summary outputs rather than the very large property-event table:

- `archetype_event_summary_v1.csv` for mechanism heatmaps, archetype ranking, and group-level scenario profiles.
- `lsoa_event_summary_v1.csv` for spatial concentration and LSOA diagnostics.
- `GIS/Westminster_LOSA.shp` joined to result LSOAs through `lsoa21cd`.
- Weighted group means use `row_count` as the aggregation weight.

This is an interpretation layer over the completed price-shock scenario engine. It does not yet add formal Fuel Price-at-Risk, affordability gap-at-risk, or intervention/de-risking dividend calculations.

## Parameter Settings

- Main payment method: Direct Debit.
- Figure 1 metric: `mean_burden_uplift_vs_s0`.
- Figure 2 metrics: `mean_burden_uplift_vs_s0` and `mean_threshold_crossing_prob_gt_10_s`.
- Figure 3 metric: `mean_threshold_crossing_prob_gt_10_s`.
- Figure 4 metrics: `mean_threshold_crossing_prob_gt_10_s` and `mean_burden_uplift_vs_s0`.
- Figure 2 archetype inclusion threshold: `row_count >= 100`.
- Figure 4 spatial scope: Westminster LSOA boundaries available in the project GIS.

## Results

Figure 1 shows the structural mechanism of exposure. Under S3, gas-heated E-G dwellings have a much larger burden uplift than electricity-heated A-C dwellings. Key weighted mean burden-uplift values include:

- Gas E-G: about 10.0 percentage points.
- Electricity A-C: about 2.8 percentage points.
- House E-G: about 12.6 percentage points.
- Flat E-G: about 7.0 percentage points.

Figure 2 identifies the most exposed Westminster household-dwelling archetypes. The highest S3 burden-uplift archetypes are dominated by gas-heated houses or maisonettes with weak EPC performance. The top examples include:

- Owner occupied | House | E-G | Gas: burden uplift about 13.9 percentage points; threshold-crossing probability about 86.7%.
- Private rented | House | E-G | Gas: burden uplift about 11.8 percentage points; threshold-crossing probability about 81.4%.
- Owner occupied | Maisonette | E-G | Gas: burden uplift about 11.1 percentage points; threshold-crossing probability about 80.5%.
- Owner occupied | House | D | Gas: burden uplift about 10.6 percentage points; threshold-crossing probability about 76.7%.

Figure 3 shows that S3 creates a clear risk peak, S4 partially mitigates that risk, and S6 Q3 leaves a residual tariff-structure/fixed-cost risk. For EPC bands:

- A-C: S3 threshold-crossing probability about 38.6%; S4 about 26.7%; S6 Q3 about 19.6%.
- D: S3 about 54.3%; S4 about 40.1%; S6 Q3 about 30.0%.
- E-G: S3 about 66.0%; S4 about 51.4%; S6 Q3 about 40.2%.

Figure 4 shows spatial concentration inside Westminster. Top S3 LSOA threshold-crossing probabilities include:

- E01033602: about 75.7%.
- E01033605: about 71.3%.
- E01004719: about 70.6%.
- E01004702: about 70.4%.
- E01004757: about 69.7%.

## Interpretation and Discussion

The main result is not simply that prices rise or that inefficient homes have higher costs. The figure set shows that household fuel price risk is structurally produced by the interaction of fuel, fabric efficiency, dwelling form, tenure, and income-distribution vulnerability.

The S3 results expose a strong gas/fabric/dwelling-form mechanism: gas-heated E-G houses carry the largest price-sensitive burden uplift. However, the threshold-crossing results show that price sensitivity and affordability vulnerability are related but not identical. Some socially rented or lower-income groups can show high threshold risk even when they are not always the highest burden-uplift group.

S4 demonstrates that policy mitigation reduces exposure but does not erase inequality. S6 Q3 indicates that tariff structure and fixed-cost components still matter, especially for households with limited ability to reduce demand or absorb standing-charge-heavy bills.

Spatially, the maps make the Westminster case contribution visible: household fuel price risk is not evenly distributed across the borough. The spatial result should be interpreted as risk concentration under required-energy modelling, not as a direct map of current observed fuel poverty.

## Figures Synced

1. Figure 1: Structural mechanisms of household fuel price exposure
   - Source: `outputs/fuel_price_shock_scenarios_v1/figures/figure_01_mechanism_heatmaps.png`
   - Also synced as PDF for publication workflow.
   - Description: heatmaps of burden uplift by fuel/EPC, tenure/EPC, and dwelling type/EPC under S3.
   - Suggested caption: "Weighted mean required-energy burden uplift under the S3 unmitigated price shock, by Westminster dwelling and household-dwelling characteristics. Values are percentage-point changes relative to S0."

2. Figure 2: The most exposed Westminster household-dwelling archetypes
   - Source: `outputs/fuel_price_shock_scenarios_v1/figures/figure_02_top_archetype_ranking.png`
   - Also synced as PDF for publication workflow.
   - Description: top archetypes by S3 burden uplift, paired with threshold-crossing risk and S4 residual risk.
   - Suggested caption: "Top Westminster household-dwelling archetypes ranked by S3 required-energy burden uplift, with corresponding threshold-crossing risk and policy-mitigated residual exposure under S4."

3. Figure 3: How shock and policy scenarios reshape group-level risk
   - Source: `outputs/fuel_price_shock_scenarios_v1/figures/figure_03_scenario_response_profiles.png`
   - Also synced as PDF for publication workflow.
   - Description: scenario response profiles by EPC band, main fuel, and tenure.
   - Suggested caption: "Threshold-crossing probability across S0, S2, S3, S4, and S6 Q3, showing how unmitigated shocks, policy mitigation, and tariff-structure sensitivity reshape group-level household fuel price risk."

4. Figure 4: Spatial concentration of household fuel price risk within Westminster
   - Source: `outputs/fuel_price_shock_scenarios_v1/figures/figure_04_spatial_distribution.png`
   - Also synced as PDF for publication workflow.
   - Description: LSOA choropleths for S3 threshold-crossing risk and burden uplift, plus a baseline-risk versus S3-uplift diagnostic scatter.
   - Suggested caption: "Spatial concentration of S3 household fuel price risk across Westminster LSOAs. The maps show threshold-crossing probability and required-energy burden uplift; the scatter compares baseline threshold risk with S3 burden uplift."

Spatial caveat:

- Result LSOAs: 126.
- GIS LSOAs available: 123.
- Matched LSOAs: 123.
- Unmatched result LSOAs: `E01000586`, `E01002885`, `E01002886`.

## Paper-writing Implications

These figures can anchor the first empirical results subsection:

1. Start with mechanism: exposure is produced by housing and fuel-system structure.
2. Move to distribution: the most exposed groups are not generic "poor households" but specific household-dwelling archetypes.
3. Show policy relevance: bill support reduces risk but leaves an unequal residual pattern.
4. Close with spatial case contribution: Westminster's internal geography concentrates risk unevenly.

Suggested manuscript subsection sequence:

- "Structural mechanisms of price exposure"
- "High-risk household-dwelling archetypes"
- "Policy mitigation and residual risk"
- "Spatial concentration within Westminster"

## Paper-ready English Notes

The results show that household fuel price risk is not a uniform consequence of rising energy prices. It is produced through the interaction of heating fuel, fabric efficiency, dwelling form, tenure, and income vulnerability.

Under the unmitigated S3 shock, gas-heated and less efficient dwellings experience the largest required-energy burden uplifts. However, the highest burden uplifts do not perfectly coincide with the highest threshold-crossing probabilities, indicating that price sensitivity and income-mediated vulnerability operate through related but distinct mechanisms.

The S4 policy-mitigated scenario reduces risk but does not eliminate the unequal distribution of exposure. This supports the paper's central argument that policy support can dampen fuel price shocks while leaving structurally exposed households only partially protected.

The spatial results further show that household fuel price risk is geographically concentrated within Westminster. This makes the case-study contribution more than descriptive: it demonstrates how local housing stock, tenure composition, and income distributions translate national price shocks into uneven neighbourhood-level risk.

## Source Trace

- Design: `docs/superpowers/specs/2026-05-28-fuel-price-shock-results-figures-design.md`
- Implementation plan: `docs/superpowers/plans/2026-05-28-fuel-price-shock-results-figures.md`
- Plotting script: `scripts/plot_fuel_price_shock_results_figures.py`
- Tests: `tests/test_fuel_price_shock_results_figures.py`
- Figure notes: `outputs/fuel_price_shock_scenarios_v1/figures/fuel_price_shock_figure_notes.md`
- Figure QA: `outputs/fuel_price_shock_scenarios_v1/figures/fuel_price_shock_figure_qa_summary.csv`
- Figure data summary: `outputs/fuel_price_shock_scenarios_v1/figures/fuel_price_shock_figure_data_summary.csv`
- Figure source tables: `figure_01_source_table.csv`, `figure_02_source_table.csv`, `figure_03_source_table.csv`, `figure_04_source_table.csv`
- Round log: `TASKS.md`, Round 2026-05-28-04.

## Open Questions

- Should payment-method sensitivity be added as an appendix figure using Direct Debit, Standard Credit, and Prepayment tracks?
- Should the next risk-metric figure use Fuel Price-at-Risk, affordability gap-at-risk, or both?
- Should the three unmatched LSOAs be resolved by obtaining an updated Westminster LSOA boundary file, or retained as a transparent GIS coverage caveat?

## Next Steps

1. Convert the figure notes into polished manuscript captions.
2. Draft the first results subsection around Figures 1-4.
3. Decide whether appendix figures should cover payment-method sensitivity, FPaR, affordability gap-at-risk, or S5 oil.
