# Baseline Modelling Extract v1 — Notes

**Build timestamp:** 2026-05-28 11:18:12  
**Script:** `scripts/build_baseline_modelling_extract.py`  
**Status:** Baseline extract for shock engine input. Not a full fuel price risk model.

---

## 1. Purpose and scope

This extract creates a fixed, traceable, shock-ready property-level baseline dataset
for Westminster, for use by `run_fuel_price_shock_scenarios.py`.

It captures:

- Property identifiers, spatial codes, and housing attributes.
- LSOA lognormal income distribution parameters and baseline energy burden metrics.
- EPC service-component costs (heating + hot water + lighting) as a price-revaluable bridge.
- Fuel-type mappings and price-spine join metadata.
- Payment-method scenario metadata (sensitivity, not observed household data).

**This file does not model fuel price shocks or household fuel price risk.** Shock
scenarios are applied downstream using `outputs/price_spine/official_price_spine_event_london.csv`.

---

## 2. Input files and row counts

| File | Rows | Notes |
|------|------|-------|
| `data/processed/all_splits_income_scenarios_epc_enriched.xlsx` | 124,980 | Primary input; read-only |

- LSOAs: 126 (expected: 126)
- MSOAs: 26 (expected: 26)

**Input files are never modified.** All outputs written to `outputs/baseline_modelling_extract/`.

---

## 3. Exact cost definitions

### 3.1 Main baseline metric: `sap_inverse_total_cost`

```
sap_inverse_total_cost = model_energy_cost_from_sap
```

Estimated from SAP Energy Cost Factor, inverted from SAP rating and `TOTAL_FLOOR_AREA`.
This is the **required** energy cost (all domestic energy services), not the actual bill.
It is the primary metric for:

- baseline energy burden (`baseline_required_energy_burden`)
- probability burden metric (`burden_prob_gt_10`)
- high-risk group flags (HR_TOP10, HR_TOP20, BR_TOP10, etc.)
- downstream shock metric ranking

### 3.2 EPC component cost: `epc_component_total_cost`

```
epc_component_total_cost = EPC_HEATING_COST_CURRENT
                         + EPC_HOT_WATER_COST_CURRENT
                         + EPC_LIGHTING_COST_CURRENT
```

**This is the price-revaluable bridge metric**, not a full household energy bill.
Cooking and other unspecified services are excluded (no EPC field available).
Median ratio EPC component / SAP inverse ≈ 0.566 (SAP inverse is systematically higher).

`epc_component_valid_flag = 0` if any component is negative or missing (1 row in this dataset);
in those cases `epc_component_total_cost = NaN`. Invalid costs are NOT silently coerced.

---

## 4. Why SAP inverse is the main baseline burden metric

SAP inverse cost represents total **required** energy demand across all services,
estimated from the physical energy efficiency rating of the property. It is:

- Consistent with the SAP methodology used in EPC certification.
- Higher than EPC component sum because it includes cooking and other services.
- Calibrated and benchmarked through the LSOA lognormal income model, with residual
  error against official LILEE fuel poverty rates documented rather than treated as validation.
- Appropriate as the primary metric for energy burden and high-risk group classification.

EPC component costs are lower and cover only heating, hot water, and lighting. Using EPC
component as the primary burden metric would **understate** the full energy service cost
and materially change which properties are classified as high risk
(Jaccard of SAP vs EPC top-10% burden groups ≈ 0.35; see energy-cost comparison round).

---

## 5. Why EPC component is the price-revaluable sensitivity bridge

The shock engine needs to rescale energy costs when fuel prices change. To do this it
requires an energy quantity (implied kWh) that can be repriced at a new unit rate.

EPC component costs (heating + hot water + lighting) are:
- Derived from the EPC certificate using SAP-standard fuel prices.
- Decomposable into implied kWh by fuel type, allowing rescaling.
- Separately available for heating (dominant) vs. hot water and lighting.

The rescaled EPC component cost under a price shock is then used as a **sensitivity
estimate of burden uplift**, while SAP inverse remains the primary ranking metric.
Results from EPC component rescaling are **not treated as a full household fuel bill**.

---

## 6. Payment method treatment

No real household-level payment method assignment is made.

- `payment_method_scenario_ready_flag = 1`: the shock engine can run three scenarios
  using Direct Debit, Standard Credit, and Prepayment rates from the price spine.
- `supported_payment_method_scenarios = "Direct Debit|Standard Credit|Prepayment"`: these
  labels match the `payment_method` column in `official_price_spine_event_london.csv` exactly.
- `observed_payment_method_available_flag = 0`: no observed payment method in the data.

Payment-method sensitivity captures the structural fact that Standard Credit and
Prepayment customers pay higher unit rates and standing charges. This is relevant
for distributional risk analysis even without household-level assignment.

---

## 7. Fuel mapping treatment

| MAIN_FUEL | price_spine_fuel | Confidence | Notes |
|-----------|-----------------|------------|-------|
| gas (102,958 properties, 82.4%) | gas | high | S0–S4, S6; DD/SC/PPM |
| electricity (21,520 properties, 17.2%) | electricity | high | S0–S4, S6; DD/SC/PPM; off-peak not split |
| oil (502 properties, 0.4%) | oil | medium | S5 only; no payment-method split; p/litre |

All 124,980 properties have a valid fuel mapping. No unmapped MAIN_FUEL values.

Scenario applicability is intentionally narrower than fuel mapping:

- Domestic gas/electricity tariff-cap spine ready (S0-S4, S6): 124,478 properties.
- Heating-oil S5 ready only: 502 properties.
- Oil properties are valid mapped records, but they are **partial coverage** and must not be
  joined to S0-S4 or S6 gas/electricity tariff-cap scenarios.

---

## 8. QA summary

| Status | Count |
|--------|------:|
| PASS | 22 |
| WARN | 0 |
| FAIL | 0 |

Full QA details: `baseline_modelling_extract_qa_summary.csv`.

---

## 9. Key empirical baseline findings

- Properties: 124,980
- SAP inverse cost: median GBP 1045/year
- EPC component cost: median GBP 651/year (ratio to SAP inverse: 0.566)
- Baseline energy burden (MSOA point income): median 0.0239 (0.6% > 10% threshold)
- burden_prob_gt_10 (distributional): median 0.0525
- HR_TOP10_BURDEN_PROB: 12,500 properties (10.0%)
- MAIN_FUEL: gas 102,958 (82.4%), electricity 21,520 (17.2%), oil 502 (0.4%)

---

## 10. Limitations

1. **No cooking cost**: EPC component excludes cooking. SAP inverse includes it implicitly.
2. **Off-peak tariffs not split**: ENERGY_TARIFF distinguishes off-peak but unit rates not split here.
3. **MSOA point income**: `baseline_required_energy_burden` uses MSOA-level median income; within-MSOA income inequality is captured only by `burden_prob_gt_10`.
4. **Oil scenarios limited**: Only S5 (heating oil p/litre) available; no payment-method split for oil.
5. **EPC certificate vintage**: EPC costs reflect the inspection date, not current tariffs. Prices indexed to SAP 10.2 (2021). Use EPC component as a structure (relative ranking) proxy, not an accurate current bill.
6. **Payment method not observed**: DD/SC/PPM is a sensitivity scenario, not household-reported data.
7. **Demand bridge not fully built yet**: this extract identifies the revaluable EPC component cost, but does not yet contain implied kWh, baseline SAP unit prices, standing-charge allocation, or service-level demand vectors.
8. **No intervention scenario in this extract**: Fabric retrofit, heat pump, PV, battery, and other interventions are not included here. This is the baseline-only extract.

---

## 11. Next step: fuel price shock scenario engine

Inputs needed:

1. **This file**: `outputs/baseline_modelling_extract/baseline_modelling_extract_v1.csv`
2. **Price spine**: `outputs/price_spine/official_price_spine_event_london.csv`

Script to build: `scripts/run_fuel_price_shock_scenarios.py`

Logic:

```text
For each scenario S in [S0, S1, S2, S3, S4, S5, S6]:
  For each payment method PM in [DD, SC, PPM] (where applicable):
    1. Apply the scenario applicability matrix:
       gas/electricity -> S0-S4 and S6; oil -> S5 only.
    2. Load unit rate and standing charge from price spine for S x PM x fuel.
    3. Build explicit implied-kWh bridge fields from epc_component_total_cost,
       baseline SAP Table 12 prices, and fuel-specific standing-charge assumptions.
    4. Re-price implied kWh at scenario unit rate.
    5. Add scenario standing charge where applicable.
    6. Compute shocked_epc_component_cost = repriced cost.
    7. Compute shocked burden and burden uplift using clearly named baseline comparators.
    8. Compute threshold-crossing probability and affordability gap.
```

The SAP inverse cost is held fixed as the baseline comparator.
The shocked EPC component cost is the sensitivity estimate.
Burden uplift = shocked_burden − baseline_burden (per property or per archetype).

---

*Generated by `scripts/build_baseline_modelling_extract.py` at 2026-05-28 11:18:12.*
