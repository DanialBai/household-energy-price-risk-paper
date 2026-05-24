# Project Mission

This repository is for writing a high-quality English-language journal article on **household fuel price risk** and the uneven protective value of domestic net zero interventions, using Westminster, London as the empirical case.

The project is currently a manuscript-writing repository, not a full data-production repository. Treat it as the place where the argument, literature positioning, conceptual framework, methods narrative, expected outputs, figures, and final paper are developed in a clean LaTeX workflow.

The working title is:

> Household Fuel Price Risk and the Uneven Protective Value of Domestic Net Zero: Evidence from Westminster, London

# Core Thesis

Fuel poverty identifies current affordability hardship. Household fuel price risk identifies future exposure to fuel price shocks. Domestic net zero determines who receives protection only when technology, housing fabric, tariff design, market structure, tenure, and governance align.

Do **not** write as if retrofit, heat pumps, electrification, PV, batteries, smart tariffs, or flexibility automatically reduce household risk. The paper's contribution is to evaluate domestic net zero interventions as **conditional household de-risking tools**, not as automatic bill-reduction or emissions-reduction devices.

# Central Research Question

To what extent, and under what housing, market, and governance conditions, do domestic net zero interventions reduce unequal household exposure to fossil-fuel price volatility in Westminster?

The three linked questions are:

1. Which household-dwelling types face the largest required energy burden increase under fuel price shocks?
2. How do income, housing fabric, heating system, tenure, and vulnerability jointly produce household fuel price risk?
3. How much risk reduction do retrofit, clean heat, PV, battery storage, smart tariffs, and integrated packages deliver, and for whom?

# Conceptual Guardrails

Never conflate these concepts:

| Concept | Meaning in this paper |
|---------|-----------------------|
| Fuel poverty | Current affordability hardship under present prices and required energy needs |
| Energy burden | Required or actual energy cost divided by residual household income |
| Household fuel price risk | Future exposure, sensitivity, and limited adaptive capacity under price-shock scenarios |

Use this conceptual formula as a guide, not as a finished econometric specification:

```text
Household fuel price risk = exposure + sensitivity - adaptive capacity
```

Keep these distinctions visible throughout the manuscript:

- Fuel poverty is a present-tense condition; fuel price risk is a future-facing exposure.
- Required energy demand is analytically preferable to actual consumption where underheating or self-rationing may exist.
- Decarbonisation is not the same as de-risking.
- Electrification can reduce emissions before it reduces household price risk if electricity prices remain gas-linked.
- The electricity/gas price ratio matters for heat pump risk outcomes.
- Westminster's private rented sector, leasehold flats, heritage constraints, high housing costs, and social housing pathways are not background details; they are causal mechanisms.

# Manuscript Structure

The main LaTeX entry point is `main.tex`. Section files live in `sections/`.

| File | Role |
|------|------|
| `sections/01_introduction.tex` | Motivation, research gap, research questions, contribution |
| `sections/02_conceptual_framework.tex` | Conceptual distinction between fuel poverty, energy burden, and household fuel price risk |
| `sections/03_case_and_data.tex` | Westminster case, data logic, unit of analysis, empirical constraints |
| `sections/04_methods.tex` | Modelling strategy, risk metrics, price shocks, intervention scenarios |
| `sections/05_expected_outputs.tex` | Tables, figures, maps, distributional outputs, expected empirical products |
| `sections/06_discussion.tex` | Interpretation, policy relevance, limits of domestic net zero as risk protection |
| `sections/07_conclusion.tex` | Contribution, implications, future research |

When editing one section, check adjacent sections for repetition and argumentative continuity.

# Writing Standard

Write for a serious international journal audience in energy policy, urban studies, housing studies, climate policy, or energy justice.

Use polished academic English. The tone should be precise, analytical, and publication-ready. Avoid generic policy-note phrasing, overclaiming, and slogans.

Preferred style:

- Lead with the argument, not with broad background.
- Make causal mechanisms explicit.
- Use topic sentences that advance the paper's thesis.
- Distinguish what the paper demonstrates, what it proposes to estimate, and what remains a limitation.
- Treat Westminster as an analytically revealing case, not just a convenient location.
- Avoid unsupported claims about policy effectiveness.
- Avoid saying an intervention "reduces bills" or "protects households" unless the condition, mechanism, and affected group are specified.

When improving prose, preserve the paper's intellectual spine:

```text
current hardship -> future exposure -> conditional protection -> uneven distribution
```

# Evidence and Citation Rules

Do not invent citations, statistics, policy details, or empirical results.

Use `references.bib` for cited literature. If a claim requires support and the reference is not yet available, either:

- add a clear placeholder comment in LaTeX, or
- flag the missing source in your response.

For current UK policy, tariff, market-design, or fuel-price claims, verify the information before writing definitive text. Include concrete dates where timing matters.

When importing material from another project, keep provenance:

- Markdown stage summaries go in `materials/phase-summaries/`.
- Original or lightly renamed candidate images go in `assets/images/source/`.
- Selected candidate images go in `assets/images/selected/`.
- Final manuscript figures referenced by LaTeX go in `paper/figures/`.

# Figure and Table Discipline

Final figures should support the manuscript's argument, not merely decorate it.

Good figure candidates include:

- conceptual diagram distinguishing fuel poverty, energy burden, and fuel price risk;
- Westminster case map or spatial context figure;
- distribution of baseline required energy burden;
- price-shock burden uplift by dwelling or household archetype;
- intervention de-risking comparison by tenure, dwelling type, or income group;
- map of de-risking dividends and residual exposure.

Do not place exploratory image exports directly in `paper/figures/`. Promote only figures that are likely to be cited in the manuscript.

# LaTeX Workflow

Keep LaTeX edits clean and minimal.

- Edit section files in `sections/` rather than expanding `main.tex` unnecessarily.
- Keep labels, captions, and references stable once introduced.
- Avoid hard-coded formatting hacks unless needed for journal-style readability.
- Build artifacts stay in `build/`.
- The committed PDF `build/main.pdf` may be updated when it represents a useful manuscript snapshot.

Ignored LaTeX intermediates include `.aux`, `.bbl`, `.blg`, `.fdb_latexmk`, `.fls`, `.log`, `.out`, `.synctex.gz`, and related files.

# Repository Structure

Current core folders:

| Path | Purpose |
|------|---------|
| `sections/` | LaTeX manuscript sections |
| `materials/phase-summaries/` | Imported Markdown stage summaries from related projects |
| `assets/images/source/` | Source images imported from related projects |
| `assets/images/selected/` | Selected candidate images for paper development |
| `paper/figures/` | Final or near-final figures referenced in the manuscript |
| `build/` | LaTeX build output, including manuscript PDF snapshots |

Do not use this repository as a dump for raw data, repeated outputs, temporary screenshots, or debug artifacts.

# Git and File Hygiene

Preserve a clean research history.

- Commit manuscript sources, curated materials, final or near-final figures, and reproducible project configuration.
- Do not commit temporary files, exploratory outputs, scratch work, or build intermediates.
- Review image and data file sizes before committing. Files over 50 MB need explicit review; core files over 100 MB should use Git LFS if they must be versioned.
- Use explicit `git add <path>` commands rather than broad staging.
- Do not rewrite or delete user work unless explicitly asked.

# Agent Instructions

When helping in this repository:

- Prioritise publication-quality English argumentation.
- Preserve the distinction between fuel poverty, energy burden, and household fuel price risk.
- Ask "who receives protection?" and "who remains exposed?" whenever interventions are discussed.
- Keep electricity market design, tariff access, and gas-to-electricity price pass-through in view.
- Treat tenure, leasehold governance, heritage constraints, housing costs, and dwelling form as central mechanisms.
- Read the relevant section or material file before making detailed edits.
- Make narrowly scoped edits that improve the manuscript without flattening its thesis.
- Flag missing evidence rather than filling gaps with invented certainty.

For git commits, follow the research-git-curator workflow: show the proposed scope, stage explicit paths, use concise research-oriented commit messages, and push only after confirmation.
