# High-Risk Figures Summary

**Generated:** 20260525_165148  
**Script:** `scripts/generate_high_risk_figures.py`  
**Project:** Westminster household fuel price risk

---

## Purpose

These six figures support the **high-risk household explanation** section of the paper. They describe
**current modelled high-risk energy burden groups** in Westminster.

### Conceptual caution

> **These figures do NOT show:**
> - Official fuel poverty (which uses the LILEE methodology and official thresholds)
> - Future fuel price shock risk (the price shock scenario engine is not yet built)
>
> **They DO show:**
> - HR10 and HR20: the top-decile and top-quintile of `burden_prob_gt_10`, a modelled probability that
>   a property's required energy cost exceeds 10% of its household's disposable income after housing costs.
> - BR10 and BR20: the top-decile and top-quintile of `baseline_required_energy_burden` (required energy
>   cost / disposable income AHC), a deterministic baseline burden metric.
> - Both are **current snapshot descriptive groups**, not causal attributions or future risk estimates.
> - Net zero interventions should NOT be interpreted as automatically de-risking from these figures alone.

---

## Input Files

| File | Role |
|------|------|
| `outputs/high_risk_explanation/high_risk_pathway_typology.xlsx` | Pathway overview, tenure/dwelling profiles |
| `outputs/high_risk_explanation/high_risk_mechanism_tables.xlsx` | Exposure/sensitivity cross-tabs, LSOA summary |
| `outputs/high_risk_explanation/high_risk_descriptive_tables.xlsx` | HR/BR overlap table |
| `data/processed/all_splits_income_scenarios_epc_enriched.xlsx` | Full property master (124,980 rows) |
| `GIS/Westminster_LOSA.shp` | LSOA polygons (123 LSOAs, EPSG:27700) |
| `GIS/westminster.shp` | Borough outline |

---

## Output Files

```
outputs/high_risk_explanation/figures/
  fig1_pathway_risk_contribution.png / .pdf
  fig2_epc_floorarea_heatmap.png / .pdf
  fig3_tenure_by_pathway.png / .pdf
  fig4_electric_exposure.png / .pdf
  fig5_spatial_maps.png / .pdf
  fig6_hr_br_overlap.png / .pdf
  figure_source_tables.xlsx
  high_risk_figures_summary.md
```

---

## Figure Descriptions

### Figure 1 — Pathway risk and contribution
**What it shows:** Two-panel horizontal bar chart. Panel A: share of each pathway's properties
that are HR10 (within-pathway risk rate). Panel B: each pathway's contribution to the total
HR10 count (scale of contribution).

**Data source:** `high_risk_pathway_typology.xlsx / pathway_overview` (columns
`HR10_share_within_pathway`, `contribution_to_all_HR10`)

**Key takeaway:** P1 (large owner-occupied gas houses) has the highest *within-pathway* HR10 rate
(~50%), meaning half of all P1 properties are HR10. P6 (Other high-risk) accounts for ~48% of
all HR10 properties due to its large catch-all size. P2/P3/P4 contribute meaningfully at scale
despite lower within-pathway rates.

**Caveat:** P6 is a residual category; its high contribution reflects heterogeneous HR10
properties not fitting P1–P5 criteria, not a single identifiable mechanism.

---

### Figure 2 — EPC band × floor area heatmap
**What it shows:** Heatmap of HR10 share within each EPC band × floor area group. Colour
intensity indicates the proportion of properties in each cell that are HR10.

**Data source:** `high_risk_mechanism_tables.xlsx / sensitivity_tables`
(cross_tab = `EPC_BAND_GROUP x FLOOR_AREA_BAND`)

**Key takeaway:** Low EPC efficiency (E/F/G) combined with large floor area (>120 m²) produces
the highest HR10 share (~70%). EPC band alone does not determine risk: a large D-rated property
carries higher HR10 share than a small E-rated property.

**Caveat:** Floor area is a proxy for required demand, not directly observed consumption.

---

### Figure 3 — Tenure structure by pathway
**What it shows:** 100% stacked horizontal bar chart showing the tenure composition of each
risk pathway (P1–P6), using EPC tenure proxy classification.

**Data source:** `high_risk_pathway_typology.xlsx / pathway_overview` (columns
`share_owner_occupied`, `share_private_rented`, `share_social_rented`)

**Key takeaway:** P1 is 100% owner-occupied (self-mobilisation route); P2 is 100% social rented
(housing association / local authority route); P3 is 100% private rented (MEES/EPC enforcement
route); P4 is predominantly private rented (~61%) with owner-occupied also present. This tenure
diversity implies different governance and intervention pathways for each group.

**Caveat:** EPC_TENURE_GROUP_FIXED is derived from the EPC tenure field at time of certificate
lodgement. It is a proxy, not verified current legal tenure.

---

### Figure 4 — Electric exposure contrast
**What it shows:** Two-panel bar/dot chart comparing three groups: (1) electric-exposed
inefficient flats/maisonettes (EPC D/E–G, EPC_HAS_ELECTRICITY_EXPOSURE_FLAG=1); (2) non-electric
low-EPC flats/maisonettes (EPC E–G, not electric-exposed); (3) other stock. Metrics: HR10 share,
HR20 share, mean burden probability, mean modelled energy cost.

**Data source:** Computed from `all_splits_income_scenarios_epc_enriched.xlsx` using columns
`EPC_HAS_ELECTRICITY_EXPOSURE_FLAG`, `PROPERTY_TYPE`, `EPC_BAND_GROUP`, `HR_TOP10_BURDEN_PROB`,
`HR_TOP20_BURDEN_PROB`, `burden_prob_gt_10`, `model_energy_cost_from_sap`.

**Key takeaway:** Electric-exposed inefficient flats/maisonettes show elevated HR10 share and
higher mean energy costs compared to other stock, but their absolute burden risk is not the
highest because their non-electric low-EPC comparators also have concentrated inefficiency.
Electrification is not automatically de-risking under current electricity pricing.

**Caveat:** Groups reflect current EPC-recorded characteristics. EPC_HAS_ELECTRICITY_EXPOSURE_FLAG
captures exposure to electricity-priced heating, not heat pump adoption.

---

### Figure 5 — Spatial concentration maps (two panels)
**What it shows:** Panel A: LSOA-level choropleth of HR10 share (% of LSOA properties that
are HR10). Panel B: Dominant high-risk pathway among HR10 properties in each LSOA.

**Data source:**  
- Panel A: `high_risk_mechanism_tables.xlsx / LSOA_summary_HR10_HR20` joined to `GIS/Westminster_LOSA.shp`  
- Panel B: Pathway assignment re-applied to `all_splits_income_scenarios_epc_enriched.xlsx`; HR10 properties grouped by LSOA; dominant pathway = max HR10 count per pathway per LSOA.

**Key takeaway:** HR10 concentration is spatially uneven across Westminster LSOAs. Dominant
pathways differ by area, with P1 (large gas houses) concentrated in specific wards and P2/P3
(flats) more distributed. This spatial patterning matters for targeted intervention design.

**Caveat:** Panel B dominant pathway represents only which pathway is *most common* among HR10
properties; an LSOA may host multiple pathway types. Pathway rules are reproduced from
`scripts/high_risk_pathway_typology.py` — see that script for full rule logic.


### GIS QA

| Item | Value |
|------|-------|
| LSOA shapefile | `GIS/Westminster_LOSA.shp` |
| Borough outline | `GIS/westminster.shp` |
| CRS | EPSG:27700 (British National Grid) |
| GIS LSOA count | 123 |
| Data LSOA count | 126 |
| GIS LSOAs matched to data | 123 |
| Data LSOAs without GIS polygon | 3 |
| Properties omitted from map (no polygon) | 44 |
| Map base | GIS 123 LSOAs only |

> Maps use GIS 123 LSOAs as the base. The 44 properties in 3 LSOAs not present in the shapefile are omitted from maps but retained in all non-spatial analyses (Figures 1–4 and Figure 6).


---

### Figure 6 — HR vs BR robustness overlap
**What it shows:** Proportional stacked bar chart showing the overlap and divergence between
HR10/HR20 (burden probability groups) and BR10/BR20 (baseline burden groups). Panel A:
HR10 vs BR10; Panel B: HR20 vs BR20.

**Data source:** `high_risk_descriptive_tables.xlsx / overlap_table`

**Key takeaway:** HR10 and BR10 overlap by ~57% (7,128 properties in both). This confirms
reasonable consistency between the probabilistic and deterministic burden metrics, but also
shows that ~43% of each group is *not* captured by the other. The two metrics identify related
but distinct high-risk sets. Using HR10 (burden probability) captures properties with high
distributional risk even if their point-estimate burden is not in the top decile.

**Caveat:** The 10% threshold in `burden_prob_gt_10` (probability that burden exceeds 10% of
income) and the top-decile cutoff in `BR10` are different in concept: the former is a distributional
probability; the latter is a deterministic cross-sectional rank. Do not conflate.

---

## Script Notes

- Pathway assignment in Figure 5 Panel B replicates rules from `scripts/high_risk_pathway_typology.py`
  (function `assign_pathways`). Logic is duplicated in this script rather than imported, to avoid
  module-level side effects.
- All figures use 300 dpi PNG and vector PDF.
- Source data tables for each figure are saved in `figure_source_tables.xlsx`.

---

*End of summary.*
