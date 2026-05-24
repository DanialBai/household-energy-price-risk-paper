# Household Fuel Price Risk and the Uneven Protective Value of Domestic Net Zero

This repository contains the manuscript project for an English-language journal article on household fuel price risk and the uneven protective value of domestic net zero interventions, using Westminster, London as the empirical case.

The project treats domestic net zero measures as conditional household de-risking tools. Retrofit, clean heat, electrification, solar PV, battery storage, smart tariffs, and flexibility are not assumed to reduce household exposure automatically. Their protective value depends on the alignment of housing fabric, heating systems, tariff design, market structure, tenure, leasehold governance, heritage constraints, and local implementation capacity.

## Research Question and Contribution

The central research question is:

> To what extent, and under what housing, market, and governance conditions, do domestic net zero interventions reduce unequal household exposure to fossil-fuel price volatility in Westminster?

The paper develops three linked questions:

1. Which household-dwelling types face the largest required energy burden increase under fuel price shocks?
2. How do income, housing fabric, heating system, tenure, and vulnerability jointly produce household fuel price risk?
3. How much risk reduction do retrofit, clean heat, PV, battery storage, smart tariffs, and integrated packages deliver, and for whom?

The intended contribution is to move beyond a present-tense account of fuel poverty and evaluate future-facing exposure to price shocks. The paper positions domestic net zero not as an automatic bill-reduction strategy, but as a set of conditional risk-protection mechanisms whose benefits are unevenly distributed across Westminster's housing and tenure landscape.

## Core Conceptual Distinctions

The manuscript keeps three concepts analytically separate:

| Concept | Meaning in this project |
| --- | --- |
| Fuel poverty | Current affordability hardship under present prices and required energy needs |
| Energy burden | Required or actual energy cost divided by residual household income |
| Household fuel price risk | Future exposure, sensitivity, and limited adaptive capacity under price-shock scenarios |

The guiding analytical sequence is:

```text
current hardship -> future exposure -> conditional protection -> uneven distribution
```

Required energy demand is preferred where underheating or self-rationing may make actual consumption misleading. Decarbonisation and de-risking are also kept distinct: electrification can reduce emissions before it reduces household price risk if electricity prices remain gas-linked or if tariff access is uneven.

## Repository Structure

The main LaTeX entry point is `main.tex`. Manuscript sections are maintained as separate files in `sections/`:

| Path | Purpose |
| --- | --- |
| `main.tex` | Main manuscript entry point |
| `sections/01_introduction.tex` | Motivation, research gap, research questions, and contribution |
| `sections/02_conceptual_framework.tex` | Conceptual framework distinguishing fuel poverty, energy burden, and fuel price risk |
| `sections/03_case_and_data.tex` | Westminster case logic, data narrative, unit of analysis, and empirical constraints |
| `sections/04_methods.tex` | Modelling strategy, price shocks, risk metrics, and intervention scenarios |
| `sections/05_expected_outputs.tex` | Planned tables, figures, maps, and distributional outputs |
| `sections/06_discussion.tex` | Interpretation, policy relevance, and limits of domestic net zero as risk protection |
| `sections/07_conclusion.tex` | Contribution, implications, and future research |
| `references.bib` | BibTeX bibliography for cited literature |
| `materials/phase-summaries/` | Imported stage summaries from related project work |
| `assets/images/source/` | Original or lightly renamed candidate images |
| `assets/images/selected/` | Selected candidate images for manuscript development |
| `paper/figures/` | Final or near-final figures referenced by the manuscript |
| `build/` | LaTeX build output, including useful PDF manuscript snapshots |
