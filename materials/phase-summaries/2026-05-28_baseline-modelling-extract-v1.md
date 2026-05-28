# 2026-05-28 Baseline Modelling Extract v1

## Context

本阶段位于 Project One 的 empirical modelling pipeline 中，承接已完成的 official price spine layer，并为下一步 `run_fuel_price_shock_scenarios.py` 建立固定、可追溯、shock-ready 的 property-level baseline input。该阶段不计算 fuel price shocks，也不声称已经得到 household fuel price risk；它只锁定当前价格和 required energy demand 下的 baseline energy burden 口径，并为后续价格冲击引擎提供连接字段。

## Research Log

- Cursor 首轮构建 `Baseline Modelling Extract v1`，输出 124,980 条 Westminster property records 和 7 组 baseline diagnostic figures。
- Codex review 后做了一轮小修：同步 `AGENTS.md` 状态，修正 notes 中对 official LILEE 的措辞，把 “validated” 改为更准确的 “calibrated and benchmarked with residual error documented”。
- 小修还新增了 scenario coverage 字段，防止 oil properties 被误接入 gas/electricity tariff-cap scenarios。
- 重跑脚本后，extract 从 73 列扩展到 77 列，QA 从 20/20 扩展为 22/22 PASS。

## Method Update

主 baseline cost 口径为：

```text
sap_inverse_total_cost = model_energy_cost_from_sap
```

该口径代表基于 SAP rating 与 floor area 反推的 required energy cost，作为 baseline energy burden 和后续 risk ranking 的主线成本指标。

价格可重估桥接口径为：

```text
epc_component_total_cost =
  EPC_HEATING_COST_CURRENT
  + EPC_HOT_WATER_COST_CURRENT
  + EPC_LIGHTING_COST_CURRENT
```

EPC component 仅包含 heating、hot water 和 lighting，不包含 cooking，因此不能作为完整 household energy bill。它的作用是为后续 fuel price shocks 提供可重估的 component-cost bridge。

Payment method 不做真实户级分配。`Direct Debit`、`Standard Credit`、`Prepayment` 作为 sensitivity scenarios 保留，匹配 price spine labels。

## Parameter Settings

- Input: `data/processed/all_splits_income_scenarios_epc_enriched.xlsx`
- Output: `outputs/baseline_modelling_extract/baseline_modelling_extract_v1.csv`
- Property count: 124,980
- LSOAs: 126
- MSOAs: 26
- Main cost metric: `sap_inverse_total_cost`
- Revaluable bridge metric: `epc_component_total_cost`
- Payment-method treatment: scenario sensitivity only, not observed household payment method
- Fuel mapping:
  - gas: full domestic cap spine, S0-S4 and S6, DD/SC/PPM
  - electricity: full domestic cap spine, S0-S4 and S6, DD/SC/PPM, off-peak not split
  - oil: partial coverage, S5 heating-oil only, no payment-method split

## Results

- Main extract: 124,980 rows x 77 columns.
- QA: 22 PASS, 0 WARN, 0 FAIL.
- SAP inverse median cost: about GBP 1,045/year.
- EPC component median cost: about GBP 651/year.
- Median EPC/SAP cost ratio: 0.566.
- Median `baseline_required_energy_burden`: 0.0239.
- Share above 10% under MSOA point-income burden: 0.6%.
- Median `burden_prob_gt_10`: 0.0525.
- HR_TOP10_BURDEN_PROB: 12,500 properties.
- Fuel composition:
  - gas: 102,958 properties, 82.4%
  - electricity: 21,520 properties, 17.2%
  - oil: 502 properties, 0.4%
- Scenario coverage:
  - full domestic cap spine: 124,478 properties
  - partial oil S5 only: 502 properties

## Interpretation and Discussion

The baseline evidence shows why the paper should keep point burden and distributional burden probability distinct. Under MSOA point income, only a small share of Westminster properties exceed a 10% energy burden, but the LSOA income-distribution metric shows a much wider probability tail. This supports the paper's conceptual separation between current energy burden and future fuel price risk.

Housing form also matters. Flats dominate the stock numerically, but houses and maisonettes are overrepresented in high-burden groups relative to their overall share. This suggests that Westminster's risk story is not simply "many flats are exposed"; it is a layered story in which stock composition, size, efficiency, income distribution, and fuel type interact.

Electricity exposure deserves close attention in the next shock engine. Electricity-heated properties are a minority of the stock but appear slightly overrepresented in the HR_TOP10 group. This is important for the paper's electrification argument: electrification is not automatically de-risking while GB electricity prices remain strongly linked to gas and while electricity/gas price ratios remain high.

Oil properties should be treated as a small, separate sensitivity group. They have a valid S5 heating-oil mapping, but they should not enter S0-S4 or S6 gas/electricity cap scenarios.

## Figures Synced

The core baseline diagnostic figures are synced for paper-writing reference:

- Fig 1: Baseline required energy burden distribution.
- Fig 2: `burden_prob_gt_10` distribution.
- Fig 3: SAP inverse vs EPC component cost comparison.
- Fig 4: High-burden group composition by property type.
- Fig 5: High-burden group composition by main fuel.
- Fig 6: LSOA official vs modelled fuel poverty rate.
- Fig 7: Fuel mapping and price-spine readiness.

Each figure is copied in both PNG and PDF form.

## Paper-writing Implications

- The paper should describe baseline energy burden as a present-tense required-cost metric, not as fuel price risk.
- `sap_inverse_total_cost` should be framed as the main required-cost baseline because it represents broader domestic energy services.
- `epc_component_total_cost` should be framed as a price-revaluable bridge, not a full bill.
- The methods section should explicitly state that payment methods are sensitivity scenarios rather than observed household attributes.
- The shock-engine section should include a scenario applicability matrix to separate gas/electricity cap scenarios from oil S5.

## Paper-ready English Notes

The baseline modelling extract establishes a fixed property-level input layer for subsequent fuel price shock simulations. The primary baseline metric is a SAP-inverse required energy cost, while EPC certificate component costs for heating, hot water and lighting are retained as a price-revaluable bridge. This distinction is important: the SAP-inverse metric provides the main burden baseline, whereas the EPC component metric supports sensitivity analysis under alternative fuel price scenarios and should not be interpreted as a complete household bill.

The baseline results indicate that Westminster's exposure cannot be inferred from point-income burden alone. Although only a small share of properties exceed a 10% energy burden under MSOA point-income assumptions, the LSOA income-distribution metric reveals a wider probability tail. This supports the paper's argument that household fuel price risk requires attention to distributional income uncertainty, property characteristics, and fuel-specific price exposure.

The extract also shows that fuel-price scenario applicability is fuel-specific. Gas and electricity properties can be linked to domestic tariff-cap scenarios, while oil properties form a small separate S5 heating-oil sensitivity group. Payment method is treated as a scenario dimension, not as an observed household attribute.

## Source Trace

- `scripts/build_baseline_modelling_extract.py`
- `outputs/baseline_modelling_extract/baseline_modelling_extract_v1.csv`
- `outputs/baseline_modelling_extract/baseline_modelling_extract_dictionary.csv`
- `outputs/baseline_modelling_extract/baseline_modelling_extract_qa_summary.csv`
- `outputs/baseline_modelling_extract/baseline_modelling_extract_notes.md`
- `outputs/baseline_modelling_extract/supporting/lsoa_summary_stats.csv`
- `TASKS.md`
- `AGENTS.md`

## Open Questions

- How should implied kWh be constructed from EPC component costs when standing charges are embedded in source EPC costs?
- Should the first shock engine report EPC-component shock burdens only, or also report a calibrated uplift applied to SAP inverse cost?
- How should Economy 7/off-peak electricity tariffs be represented in the first formal shock engine?
- Should oil S5 remain an appendix sensitivity or be included in headline household fuel price risk tables?

## Next Steps

Build `scripts/run_fuel_price_shock_scenarios.py`. The first version should construct an explicit implied-kWh and scenario applicability bridge, then combine `baseline_modelling_extract_v1.csv` with `outputs/price_spine/official_price_spine_event_london.csv` to estimate shocked component costs, burden uplift, threshold-crossing probability, and affordability gap metrics.
