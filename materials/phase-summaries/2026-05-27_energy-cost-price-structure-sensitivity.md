# 2026-05-27 Energy Cost Price-Structure Sensitivity

## Context

This archive records the exploratory energy-cost price-structure sensitivity analysis completed in project one and prepared for use in the paper-writing workspace. The analysis responds to a methodological question: whether household exposure and energy-burden rankings differ when baseline required energy cost is represented by (i) SAP rating inverse total cost, (ii) EPC certificate service-component cost, and (iii) an exploratory unit-rate versus standing-charge decomposition.

This is a baseline-cost and price-structure sensitivity exercise. It is not official fuel poverty measurement, not a full future household fuel price risk model, and not a price shock engine.

## Research Log

The analysis was implemented as a non-destructive standalone workflow:

- Script: `scripts/compare_energy_cost_price_structures.py`
- Output directory: `outputs/energy_cost_price_structure_comparison/`
- Main report: `outputs/energy_cost_price_structure_comparison/energy_cost_price_structure_comparison_summary.md`
- Main workbook: `outputs/energy_cost_price_structure_comparison/energy_cost_price_structure_comparison.xlsx`

The script preserves the master row count and does not overwrite `data/all_splits_income_scenarios.xlsx`. It uses the processed EPC-enriched file for matched EPC cost components:

- `EPC_HEATING_COST_CURRENT`
- `EPC_HOT_WATER_COST_CURRENT`
- `EPC_LIGHTING_COST_CURRENT`

Cooking cost is excluded because the matched EPC certificate fields do not provide a separate cooking cost variable. This limitation is explicitly recorded in the output report.

## Method Update

Four baseline cost definitions were compared:

1. `sap_inverse_total_cost`: `model_energy_cost_from_sap`, derived from SAP rating and total floor area through the SAP Energy Cost Factor inverse.
2. `epc_component_total_cost`: EPC heating + hot water + lighting cost.
3. `unit_price_only_cost`: implied kWh reconstructed from EPC component cost and SAP 10.2 Table 12 unit price, excluding standing charge.
4. `unit_plus_standing_cost`: reconstructed unit-price cost plus SAP 10.2 Table 12 standing charge.

The price-structure decomposition is algebraic:

```text
approx_kwh = (epc_component_total_cost - standing_charge) / unit_price
unit_price_only_cost = approx_kwh * unit_price
unit_plus_standing_cost = unit_price_only_cost + standing_charge
```

Therefore, `unit_plus_standing_cost` equals `epc_component_total_cost` by construction. The ranking overlap between `epc_component` and `unit_plus_standing` should not be interpreted as independent validation. The meaningful standing-charge comparison in this run is `unit_price_only` versus `unit_plus_standing`.

Energy burden metrics in this run use the MSOA point estimate `Disposable annual income after housing costs`, not the LSOA distributional income engine or `burden_prob_gt_10`.

## Parameter Settings

SAP 10.2 Table 12 exploratory mapping from `MAIN_FUEL`:

| Main fuel | Unit price | Standing charge |
|---|---:|---:|
| gas | 3.64 p/kWh | GBP 92/year |
| electricity | 16.49 p/kWh | GBP 81/year |
| oil | 4.94 p/kWh | GBP 0/year |

Off-peak and dual-tariff structures are not modelled in this round. `ENERGY_TARIFF` is reported descriptively only.

## Results

QA:

- Input rows: 124,980
- Output rows: 124,980
- Master file overwritten: no
- Valid EPC component costs: 124,979
- EPC component rows nulled for negative values: 1
- Valid fuel mapping: 124,980
- Valid income: 124,980

Core result: baseline cost definition materially changes current high-burden ranking.

- `sap_inverse_total_cost` median: GBP 1,045/year
- `sap_inverse_total_cost` mean: GBP 1,256/year
- `epc_component_total_cost` median: GBP 651/year
- `epc_component_total_cost` mean: GBP 805/year
- Median EPC-component-to-SAP-inverse cost ratio: 0.566
- Share of properties where EPC component cost is below SAP inverse cost: 89.2%
- Pearson correlation between cost metrics: 0.713
- Spearman correlation between cost metrics: 0.763
- Top 10% SAP inverse vs EPC component Jaccard overlap: 0.351
- Top 20% SAP inverse vs EPC component Jaccard overlap: 0.449

Top-decile composition shifts by cost definition:

| Group | Flats | Houses | Maisonettes |
|---|---:|---:|---:|
| SAP inverse top 10% | 6,477 | 4,510 | 1,511 |
| EPC component top 10% | 7,400 | 3,794 | 1,307 |

Top-decile composition by main fuel:

| Group | Gas | Electricity | Oil |
|---|---:|---:|---:|
| SAP inverse top 10% | 9,763 | 2,674 | 61 |
| EPC component top 10% | 9,613 | 2,852 | 36 |

Standing-charge decomposition:

- Median burden uplift from standing charge: about 0.0021 burden units.
- Top 10% overlap between unit-price-only and unit-plus-standing burden: Jaccard 0.966.
- Under this approximation, standing charges raise burden levels modestly but do not materially reorder the highest-burden decile.
- Property-type differentiation in standing-charge uplift is limited in this run.

## Interpretation and Discussion

The strongest methodological implication is that baseline required-cost choice is not a neutral modelling detail. SAP inverse costs are systematically higher than EPC heating + hot water + lighting totals, and the resulting top-burden groups overlap only partially. This affects who is identified as currently exposed under an energy-burden framing.

For the Westminster paper, this supports a methodological caution: current `energy burden` and future `household fuel price risk` depend on the cost denominator selected before price shocks are introduced. The paper should avoid presenting any single baseline cost measure as self-evidently definitive without explaining its boundary.

Standing charges matter as a fixed-cost component, but in the current reconstruction they do not strongly change the high-burden ranking. This does not mean standing charges are unimportant for future risk modelling. A full shock engine should separately model unit-rate shocks, standing-charge shocks, electricity/gas price ratios, off-peak tariffs, and policy mitigation.

## Figures Synced

The following figures are synced for paper-writing reference:

1. `cost_distribution_comparison.png`
   - Description: compares annual cost distributions across SAP inverse, EPC component, unit-only, and unit-plus-standing definitions.
   - Suggested caption: Distribution of alternative baseline required energy cost definitions, Westminster property-level sample.
   - Caveat: exploratory sensitivity analysis; not official fuel poverty.

2. `burden_distribution_comparison.png`
   - Description: compares burden distributions under the four cost definitions.
   - Suggested caption: Energy burden distributions under alternative required-cost definitions.
   - Caveat: burden uses MSOA point income, not LSOA income distributions.

3. `sap_vs_epc_component_cost_scatter.png`
   - Description: shows the divergence between SAP inverse total cost and EPC service-component cost.
   - Suggested caption: SAP inverse total cost versus EPC heating, hot water, and lighting component cost.
   - Caveat: EPC component excludes cooking and other implicit SAP services.

4. `unit_only_vs_unit_standing_burden_scatter.png`
   - Description: compares reconstructed unit-price-only burden with unit-plus-standing burden.
   - Suggested caption: Standing-charge decomposition of EPC component burden.
   - Caveat: `unit_plus_standing_cost` equals EPC component cost by construction.

5. `standing_charge_share_by_main_fuel.png`
   - Description: fixed-charge share of reconstructed component cost by main fuel.
   - Suggested caption: Fixed-charge share of reconstructed component cost by main fuel.
   - Caveat: single-fuel mapping only; off-peak and dual-fuel structures are not modelled.

6. `burden_uplift_from_standing_by_property_type.png`
   - Description: burden uplift from adding standing charge by property type.
   - Suggested caption: Burden uplift associated with standing-charge component by property type.
   - Caveat: property-type differentiation is limited in this run.

7. `top10_overlap_heatmap.png`
   - Description: Jaccard overlap between top-decile burden groups across cost definitions.
   - Suggested caption: Top-decile high-burden group overlap under alternative cost definitions.
   - Caveat: `epc_component` and `unit_plus_standing` overlap by construction.

## Paper-writing Implications

- The paper should define the baseline required energy cost column before reporting exposure or risk rankings.
- The divergence between SAP inverse and EPC component cost can be used to justify a robustness section.
- The cost-definition sensitivity itself is publishable as a methodological transparency point: energy-burden and risk rankings are partly shaped by the translation from EPC/SAP data into required annual cost.
- The full household fuel price risk engine should not simply multiply a single total cost by a price-shock factor. It should separate demand/service components, fuel type, tariff structure, standing charges, and electricity/gas price pass-through.
- Westminster flats remain central, but this run does not support a strong claim that standing charges specifically rerank flats into the highest-burden decile.

## Paper-ready English Notes

Baseline cost definition is not a neutral modelling choice. In the Westminster property-level sample, SAP-inverse annual cost is substantially higher than the EPC heating, hot water and lighting component total, and the top-decile energy-burden groups overlap only partially. This indicates that household exposure rankings are sensitive to how EPC/SAP information is translated into required annual energy cost.

The standing-charge decomposition raises burden levels modestly but does not materially reorder the highest-burden decile under the present approximation. This result should not be interpreted as evidence that standing charges are irrelevant. Rather, it shows that a full fuel price risk engine must model standing charges, unit rates, tariff structures and fuel-specific pass-through separately rather than treating energy cost as a single scalar.

Fuel poverty identifies current hardship, energy burden measures required cost relative to income, and household fuel price risk concerns future exposure under price shocks. This sensitivity analysis informs the baseline burden model; it does not yet estimate future fuel price risk.

## Source Trace

- `scripts/compare_energy_cost_price_structures.py`
- `outputs/energy_cost_price_structure_comparison/energy_cost_price_structure_comparison_summary.md`
- `outputs/energy_cost_price_structure_comparison/energy_cost_price_structure_comparison.xlsx`
- `outputs/energy_cost_price_structure_comparison/cost_distribution_comparison.png`
- `outputs/energy_cost_price_structure_comparison/burden_distribution_comparison.png`
- `outputs/energy_cost_price_structure_comparison/sap_vs_epc_component_cost_scatter.png`
- `outputs/energy_cost_price_structure_comparison/unit_only_vs_unit_standing_burden_scatter.png`
- `outputs/energy_cost_price_structure_comparison/standing_charge_share_by_main_fuel.png`
- `outputs/energy_cost_price_structure_comparison/burden_uplift_from_standing_by_property_type.png`
- `outputs/energy_cost_price_structure_comparison/top10_overlap_heatmap.png`
- `materials/LSOA_income_distribution_methodology.md`
- `materials/Energy_Burden_Metrics_and_High_Risk_Groups.md`
- `materials/SAP 10.2 - 17-12-2021.pdf`

## Open Questions

- Which cost definition should become the canonical baseline for the clean baseline modelling dataset?
- Should SAP inverse total cost be retained as the main baseline, with EPC component cost used for robustness?
- How should cooking and other implicit SAP services be represented if they are not directly available in EPC certificate component fields?
- How should off-peak electricity tariffs, dual-fuel households, and heat-network cases be handled in the next model?

## Next Steps

1. Decide the canonical baseline cost column for the clean baseline extract.
2. Keep SAP inverse versus EPC component as a robustness comparison in the methods/results section.
3. Build explicit service/fuel demand vectors before implementing future price shocks.
4. Model standing-charge shocks separately from unit-rate shocks in the future price-shock engine.
5. Connect post-shock burdens to the LSOA income-distribution engine rather than relying only on MSOA point income.
