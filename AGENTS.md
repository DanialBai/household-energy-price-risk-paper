# Project Mission

This project develops a high-quality journal paper on **UK household fuel price risk**, with Westminster, London as the empirical case. The paper reframes household energy inequality from a static problem of current `fuel poverty` into a dynamic problem of **uneven household exposure to fossil-fuel price volatility**.

The empirical ambition is property-level or property-archetype modelling to estimate how domestic interventions — fabric retrofit, systems upgrades, clean heat, solar PV, battery storage, smart tariffs, and flexibility — change household risk before and after implementation.

The intended contribution is not simply to show that retrofit or net zero reduces emissions. It is to evaluate whether, how, and for whom domestic low-carbon interventions provide credible **household risk protection** against future fuel price shocks.

# Central Thesis

> Fuel poverty identifies current hardship. Household fuel price risk identifies future exposure to price shocks. Net zero determines who receives protection.

> Domestic net zero interventions do not automatically de-risk households from fossil-fuel price volatility. They reduce unequal price exposure only when housing fabric improvements, low-carbon heat, distributed generation, flexibility, tariff design, market reform, and tenure-sensitive governance are aligned.

Treat net zero interventions as **conditional household de-risking tools**, not automatic bill-reduction devices.

# Research Questions

**Central RQ:** To what extent, and under what housing, market, and governance conditions, do domestic net zero interventions reduce unequal household exposure to fossil-fuel price volatility in Westminster?

1. **Who is most exposed?** — Which household-dwelling types face the largest required energy burden increase under fuel price shocks?
2. **Why are they exposed?** — How do income, fabric, heating system, tenure, and vulnerability jointly produce risk?
3. **Can interventions de-risk?** — How much risk reduction do retrofit, clean heat, PV, battery, smart tariffs, and integrated packages deliver, and for whom?

For full RQ framing and contribution language, read `materials/DP8.md` or `materials/Westminster_Fuel_Risk_RQ_Contribution_Methodology.md`.

# Core Concepts (Never Conflate These)

| Concept | Role in this project |
|---------|---------------------|
| **Fuel poverty** | Current affordability hardship under present prices and required energy needs |
| **Energy burden** | Required (or actual) energy cost / residual household income |
| **Household fuel price risk** | Future exposure, sensitivity, and limited adaptive capacity under price shock scenarios |

```text
Household fuel price risk = exposure + sensitivity - adaptive capacity
```

```text
Decarbonisation is not the same as de-risking.
```

Electricity decarbonisation may reduce system emissions before it reduces household price risk where GB wholesale electricity remains gas-linked. See `materials/DP2.md` and `materials/DP4.md` for conceptual and market detail.

**Modelling principle:** use **required energy demand**, not only actual consumption. Actual use may understate unmet need through underheating or self-rationing.

# Empirical Strategy

Unit of analysis: **property or household-dwelling pair**. Where household data are unavailable, use property records with synthetic household or archetype assignment. Aggregate to LSOA, MSOA, ward, tenure, or dwelling type for interpretation.

```text
Building stock layer
    -> Household or archetype assignment
    -> Required energy demand model
    -> Tariff and price engine
    -> Fuel price shock scenario engine
    -> Intervention engine
    -> Fuel poverty, burden, and risk metrics
    -> Spatial maps and distributional outputs
```

For intervention packages, metric formulas, shock scenarios, Westminster case detail, and expected outputs, read `materials/DP3.md`, `materials/DP5.md`, and `materials/DP6.md` before modelling or writing tasks.

# Current Modelling Status

- [x] Enriched property master (`data/all_splits_income_scenarios.xlsx`; 124,980 properties, 126 LSOAs, 26 MSOAs)
- [x] LSOA lognormal income distribution + SAP energy cost module (last verified run: 20260520_213323; 12/12 QA passed)
- [ ] Clean baseline modelling dataset extract
- [ ] Baseline required energy costs and baseline energy burden at property level
- [ ] Price shock scenario engine
- [ ] Risk metrics (Fuel Price-at-Risk, threshold-crossing, affordability gap-at-Risk)
- [ ] Intervention simulations and de-risking dividend maps
- [ ] Spatial and distributional outputs

For income-distribution methodology, run state, and QA gates, read `materials/LSOA_income_distribution_methodology.md`.

# Data Assets

The `data/` folder contains the working empirical datasets. Treat these as core research inputs and preserve raw or master files unless an overwrite has been explicitly approved.

- `all_splits_income_scenarios.xlsx`  
  Main Westminster property-level master file: EPC attributes (address, postcode, UPRN, age band, property type, built form, tariff, main fuel, fabric, floor area, SAP/EI, heating cost, coordinates, MSOA code), enriched with LSOA code, MSOA after-housing-cost income, LSOA income deprivation score, official LSOA fuel poverty rate, and LSOA income-distribution columns from the latest pipeline run.

- `PCD_OA21_LSOA21_MSOA21_LAD_FEB25_UK_LU.csv`  
  UK postcode lookup for OA 2021, LSOA 2021, MSOA 2021, and LAD. Use to match postcode to `LSOA code` and validate `MSOA code`.

- `datasetfinal.xlsx`  
  ONS small-area household income. Use sheet `Net income after housing costs` at MSOA level.

- `File_7_IoD2025_All_Ranks_Scores_Deciles_Population_Denominators.csv`  
  IoD 2025 at LSOA level. Use `Income Score (rate)`.

- `File_6_IoD2025_Population_Denominators.xlsx`  
  IoD 2025 population denominators for LSOA-level rates and aggregation checks.

- `fuel-poverty-sub-regional-2026-2024-data-tables.xlsx`  
  Official sub-regional fuel poverty data. Use `Table 4` for LSOA `Proportion of households fuel poor (%)`.

- `data/backups/`  
  Timestamped backups before overwriting master or processed files. Do not use as analytical inputs unless restoring or auditing.

# Source Materials — When to Read What

Read the relevant file before detailed tasks. Do not guess from memory when precision matters.

| File | Read for |
|------|----------|
| `materials/项目核心研究总结.md` | Chinese overview of framework, concepts, data pipeline, and current LSOA income module |
| `materials/UK_Household_Fuel_Risk_Research_Mainline.md` | UK-level mainline, thesis, RQs, archetypes, shock and intervention scenarios |
| `materials/Westminster_Fuel_Risk_RQ_Contribution_Methodology.md` | Westminster RQ, contribution, methodology, spatial outputs, paper structure |
| `materials/DP1.md` | Systematic literature review: fuel poverty, energy vulnerability, net zero protection, London/Westminster gaps |
| `materials/DP2.md` | Conceptual precision: fuel poverty vs price risk, required vs actual expenditure, hidden energy poverty |
| `materials/DP3.md` | Net zero interventions as de-risking tools: retrofit, heat pumps, PV, batteries, smart tariffs, adoption inequality |
| `materials/DP4.md` | GB marginal pricing, gas-to-electricity pass-through, electrification risk, missing insurance dividend |
| `materials/DP5.md` | Westminster case detail: PRS, social housing, flats, heritage stock, LAEP, local policy |
| `materials/DP6.md` | Method and modelling design: metrics, shock scenarios, interventions, maps, outputs, limitations |
| `materials/DP7.md` | Policy context: Fuel Poverty Strategy, ECO4, GBIS, SHDF, BUS, MEES, social tariff |
| `materials/DP8.md` | Research gap, refined RQs, contribution statement, introduction-ready gap language |
| `materials/LSOA_income_distribution_methodology.md` | LSOA lognormal income distribution, MSOA lambda calibration, SAP cost model, run procedure, QA gates |

# Agent Instructions

Treat this as a serious high-impact journal paper, not a generic policy note.

- Preserve the distinction between `fuel poverty`, `energy burden`, and `household fuel price risk`.
- Do not treat retrofit, heat pumps, electrification, PV, batteries, or smart tariffs as automatically beneficial.
- Always ask: **who receives protection?** and **who remains exposed?**
- Use property-level or household-dwelling evidence as the empirical backbone.
- When a task needs detail, consult the document map above and read the relevant source file first.
- Use `required energy demand`; do not rely only on actual energy spending.
- Keep electricity market design in view: gas-to-electricity pass-through and the electricity/gas price ratio matter for heat pumps and electrification.
- Treat Westminster's PRS, flats, leasehold blocks, heritage constraints, and social housing pathways as core causal mechanisms.
- Prioritise publication-quality argumentation for energy, urban studies, housing, climate policy, or energy justice journals.

For coding, data processing, and simulation tasks, follow `.cursor/rules/research-workflow.mdc`.

For git commits, follow `.cursor/rules/git-workflow.mdc` (or `@git-workflow` when committing).
