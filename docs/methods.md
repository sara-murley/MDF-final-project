---
layout: default
title: Methods
nav_order: 2
---

# Methods
{: .fs-8 }

Data processing, analytical pipeline, and identification strategy
{: .fs-5 .fw-300 }

---

## Data Sources

<!--
FILL IN: Describe every dataset you used. For each one, include:
- Full name and source (with link)
- Unit of observation (country-year, household, etc.)
- Time period covered
- Key variables you used from it
- Any known limitations or coverage gaps

Example datasets you might have used:
- ITU World Telecommunication/ICT Indicators Database (broadband penetration)
- World Bank World Development Indicators (GDP, education, governance)
- Freedom House / Polity V (political institutions)
- UN Human Development Index
- Ookla Speedtest data
-->

| Dataset | Source | Coverage | Key Variables |
|---|---|---|---|
| [Dataset name] | [Source/link] | [Years, countries] | [Variables] |
| [Dataset name] | [Source/link] | [Years, countries] | [Variables] |
| [Dataset name] | [Source/link] | [Years, countries] | [Variables] |

### Data Assembly

<!--
FILL IN: Explain how you merged/joined datasets. What was the merge key (ISO country code + year)?
How did you handle missing data? What was the final panel dimensions (N countries × T years)?
-->

## Variable Construction

<!--
FILL IN: Define your key variables precisely.
- Dependent variable(s): how measured, units, source
- Main independent variable: broadband penetration (per 100 inhabitants? % of population? fixed vs. mobile?)
- Control variables: list and justify each one
- Any constructed indices or transformations (log, standardization, etc.)
-->

### Outcome Variables

<!--
FILL IN: Describe your dependent variable(s) in detail.
-->

### Treatment Variable

<!--
FILL IN: Describe broadband/connectivity measure. Note: it's worth flagging that "broadband" definitions
vary across ITU reporting — fixed broadband vs. mobile broadband vs. internet users are meaningfully different.
-->

### Controls

<!--
FILL IN: List control variables and briefly justify why each is included (omitted variable bias logic).
-->

## Identification Strategy

<!--
FILL IN: This is the most important methods section for a causal claim. Describe:
- What is the core endogeneity problem? (Infrastructure is placed in already-growing areas; reverse causality)
- What is your identification approach?
  Options: panel fixed effects, instrumental variables (IV), difference-in-differences, synthetic control, regression discontinuity
- If IV: what is your instrument and why is it valid (relevance + exclusion restriction)?
  Common instruments in this literature: submarine cable landings, terrain ruggedness interacted with time, colonial-era telephone infrastructure
- What assumptions does your approach require, and how do you defend them?
-->

{: .important }
**Endogeneity note:** Internet infrastructure is not randomly placed — providers and governments preferentially deploy broadband in areas with higher expected returns (denser, wealthier, more urban populations). Any naive OLS estimate of broadband on outcomes will be upward-biased. Our identification strategy addresses this by [describe your approach].

## Analytical Pipeline

<!--
FILL IN: Walk through the steps of your analysis in order. This should be reproducible — someone should be 
able to follow these steps and arrive at the same result.
-->

```
1. Raw data download & ingest
        ↓
2. Cleaning & standardization (country codes, year alignment)
        ↓
3. Panel construction & merge
        ↓
4. Exploratory data analysis (distributions, missing data, outliers)
        ↓
5. [Your model] estimation
        ↓
6. Robustness checks & sensitivity analysis
        ↓
7. Visualization & output
```

### Tools & Environment

<!--
FILL IN: List languages, packages, and compute environment.
-->

This analysis was conducted in **[Python / R]** using the following key libraries:

| Tool | Purpose |
|---|---|
| `pandas` / `tidyverse` | Data wrangling |
| `[statsmodels / linearmodels / fixest]` | Panel regression |
| `[geopandas / ggplot2]` | Geospatial visualization |
| `[plotly / bokeh / altair]` | Interactive charts |

## Model Specification

<!--
FILL IN: Write out your estimating equation in plain language and/or LaTeX if you can render it.
Describe each term. Example:

Y_it = α + β(Broadband_it) + γX_it + δ_i + λ_t + ε_it

Where:
- Y_it = outcome for country i in year t
- Broadband_it = fixed broadband subscriptions per 100 inhabitants
- X_it = vector of time-varying controls
- δ_i = country fixed effects (absorb time-invariant confounders)
- λ_t = year fixed effects (absorb global time trends)
-->

## Robustness Checks

<!--
FILL IN: Describe at least 2–3 robustness checks you ran. Examples:
- Alternative broadband measures (mobile vs. fixed)
- Dropping outliers or specific regions
- Different time windows
- Placebo tests
- Alternative control sets
-->

[View the Code →](data){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Explore Visualizations →](explorer){: .btn .fs-5 .mb-4 .mb-md-0 }