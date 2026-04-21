---
layout: default
title: Methods
nav_order: 2
---

# Methods
{: .fs-8 }

Data infrastructure, analytical pipeline, and modeling strategy
{: .fs-5 .fw-300 }

---

## 1. Data Sources

All data are drawn from the **World Bank World Development Indicators (WDI)**, accessed programmatically via the World Bank REST API. Four indicators were selected to capture the core dimensions of this analysis: digital connectivity, economic development, education, and health.

| Indicator | World Bank Code | Units | Role |
|---|---|---|---|
| Internet users | `IT.NET.USER.ZS` | % of population | Treatment variable |
| GDP per capita | `NY.GDP.PCAP.CD` | Current USD | Economic control |
| Life expectancy | `SP.DYN.LE00.IN` | Years at birth | Health outcome |
| Secondary enrollment | `SE.SEC.ENRR` | Gross % | Education outcome |

Coverage spans **2000–2023** across all World Bank reporting countries (180+), yielding a long, wide cross-country panel. The 2000 start date was chosen deliberately: it captures the full arc of the global internet expansion era while avoiding the sparse and unreliable early 1990s connectivity data. The WDI was selected over alternatives (e.g., ITU standalone database, Ookla Speedtest data) because it provides a harmonized, consistently updated source for all four variables under a single API, minimizing merge error and coverage inconsistency across datasets.

---

## 2. Data Infrastructure

### Why Cloud Storage?

Storing and accessing a multi-country, multi-decade panel in a local file introduces several problems: version drift, collaboration friction, and the inability to query subsets without loading the full dataset into memory. For a project of this scope — 180+ countries, 23 years, four indicators — a cloud-based architecture was the most practical and reproducible choice.

### Architecture

The project uses a two-tier AWS storage architecture:

```
World Bank API
      │
      ▼
 ┌─────────────────────────────────────────┐
 │          Amazon S3                      │
 │  raw/worldbank_raw.json   ← full API    │
 │  clean/worldbank_panel.csv ← processed  │
 └────────────────────┬────────────────────┘
                      │
                      ▼
 ┌─────────────────────────────────────────┐
 │       Amazon RDS (PostgreSQL)           │
 │    table: worldbank_panel               │
 │    queryable via psycopg2 / SQLAlchemy  │
 └─────────────────────────────────────────┘
```

**S3** serves as the durable, versioned data lake. The raw JSON response from each API call is preserved in full (`raw/worldbank_raw.json`), providing an audit trail back to the original source. The cleaned, merged CSV (`clean/worldbank_panel.csv`) is stored alongside it for lightweight access and sharing.

**RDS (PostgreSQL)** provides structured, queryable access to the panel. Rather than reading and filtering a full CSV into memory for every analysis script, any downstream query can be expressed in SQL and only the relevant rows and columns are transferred. This matters especially for iterative modeling work, where running a regression on a subset (e.g., post-2010 observations, or a single region) should not require loading 23 years of global data. PostgreSQL also enforces column types and supports joins, making it straightforward to extend the dataset later with additional indicators.

The combination — S3 for storage and provenance, RDS for analysis access — is a standard pattern in production data pipelines and provides a more rigorous foundation than a local flat-file workflow.

### Data Ingestion Code

The ingestion script loops over each indicator, calls the World Bank API with a single `per_page=20000` request to retrieve the full time series in one call, parses the JSON response into a tidy dataframe, and outer-merges all four indicators on `country_code` × `year`.

Key decisions in the ingestion step:

- **`per_page=20000`** — The default pagination limit is 50 records. Setting a high page size retrieves the full indicator series in a single request, avoiding the complexity of paginating through hundreds of pages per indicator.
- **Outer merge** — Using `how="outer"` on the indicator merge preserves all country-year observations even when one indicator has missing data for a given country-year. This defers the decision about how to handle missingness to the analysis stage, rather than silently dropping observations at merge time.
- **ISO3 filter** (`country_code.str.len() == 3`) — The World Bank API returns aggregate rows for regions and income groups (e.g., "East Asia & Pacific") alongside individual countries. Filtering to three-character ISO codes retains only sovereign country observations and drops regional aggregates.
- **`na_rep="NULL"` on CSV export** — Ensures that missing values are written as `NULL` rather than empty strings, which is essential for correct type inference when loading the CSV into PostgreSQL.

---

## 3. Variable Construction

### Treatment Variable

**Internet users (% of population)** is the primary treatment variable throughout all models. This measure — the share of a country's population that has used the internet in the past three months, as reported by national statistical agencies to the ITU and harmonized by the World Bank — is the broadest available measure of digital connectivity and has the most complete coverage across the panel.

{: .note }
A note on measurement: this variable captures *access and use*, not infrastructure capacity. A country with high fixed broadband infrastructure but low adoption will appear low on this measure. For the policy questions in this analysis — which are about population-level human development outcomes, not network capacity — usage penetration is the more relevant measure.

### Outcome Variables

**Secondary school enrollment (gross %)** is used as the education outcome. The gross enrollment ratio can exceed 100% when students outside the official age range are enrolled, which is why some values in the data exceed 100. This is a known feature of the measure, not a data error.

**Life expectancy at birth** is used as the health outcome in the within-country (first-difference) model. Life expectancy integrates health system quality, nutritional status, and disease burden into a single summary statistic, making it a reliable long-run health indicator across diverse country contexts.

### Transformations

**Log GDP per capita** — GDP per capita is log-transformed in all models. The income distribution across countries is highly right-skewed: a small number of very high-income countries have GDP per capita far above the median. Log transformation compresses this distribution, reduces the influence of outliers, and makes the coefficient on GDP interpretable as the effect of a proportional (rather than absolute) change in income — which is the theoretically appropriate scale for cross-country income comparisons.

**First differences (within-country model)** — For the life expectancy analysis, year-over-year changes in internet penetration and life expectancy within each country are computed. This transformation removes all time-invariant country characteristics (geography, colonial history, long-run institutional quality) that might confound the relationship.

**Interaction term** (`internet_users_pct × log_gdp_pc`) — Constructed to allow the marginal effect of internet access to vary by income level. A negative interaction coefficient indicates that the effect of connectivity on education is smaller in richer countries.

**Quadratic term** (`internet_users_pct²`) — Constructed to test for nonlinearity in the connectivity-education relationship. A negative quadratic coefficient indicates diminishing marginal returns: the relationship is concave, with large gains at low penetration levels and smaller gains as penetration increases.

---

## 4. Analytical Pipeline

The analysis proceeds through five stages, each building on the last. The progression from simple correlations to progressively controlled models is intentional: it makes the threat to causal inference explicit at each step and demonstrates the robustness of the core finding.

```
Stage 1: Exploratory Analysis
   Correlation matrix + heatmap
   → Establishes raw associations; motivates the OVB concern
         │
         ▼
Stage 2: Baseline OLS
   Secondary enrollment ~ internet + log(GDP) 
   → Isolates the independent effect of connectivity once income is controlled
         │
         ▼
Stage 3: Interaction Model with Year Fixed Effects
   Secondary enrollment ~ internet × log(GDP) + year FEs
   → Tests whether returns vary by income; controls for global time trends
         │
         ▼
Stage 4: Within-Country First-Difference Model
   ΔLife expectancy ~ ΔInternet + year FEs
   → Removes all time-invariant country heterogeneity
         │
         ▼
Stage 5: Nonlinear (Quadratic) Model + Clustering
   Secondary enrollment ~ internet + internet² + log(GDP) + year FEs
   → Identifies saturation point and groups countries by investment profile
```

---

## 5. Model Specifications

### Model 1 — Baseline OLS

$$Y_{i} = \alpha + \beta_1 \cdot \text{Internet}_i + \beta_2 \cdot \log(\text{GDP})_i + \varepsilon_i$$

A pooled cross-sectional regression establishing the baseline association between internet access and secondary enrollment, controlling for income. This model does not control for time trends or country fixed effects; its estimates reflect cross-country structural differences and are subject to omitted variable bias. It is included as a transparency step — to show what the naive estimate looks like before more rigorous controls are introduced.

### Model 2 — Interaction Model with Year Fixed Effects

$$Y_{it} = \alpha + \beta_1 \cdot \text{Internet}_{it} + \beta_2 \cdot \log(\text{GDP})_{it} + \beta_3 \cdot (\text{Internet} \times \log\text{GDP})_{it} + \lambda_t + \varepsilon_{it}$$

Year fixed effects (λ_t) absorb any global shocks common to all countries in a given year — the 2008 financial crisis, the COVID-19 pandemic, the global expansion of mobile internet — ensuring that the estimated coefficients reflect within-year variation across countries rather than global time trends. The interaction term tests the hypothesis that the marginal effect of connectivity varies by income level.

### Model 3 — First-Difference Model (Health Outcome)

$$\Delta \text{LifeExp}_{it} = \alpha + \beta \cdot \Delta \text{Internet}_{it} + \lambda_t + \varepsilon_{it}$$

By regressing year-over-year *changes* in life expectancy on year-over-year *changes* in internet penetration within the same country, this specification removes all time-invariant country heterogeneity — geography, historical institutions, cultural factors — that could confound the cross-sectional relationship. Year fixed effects additionally control for common global health shocks (e.g., pandemic years). This is the most credibly causal specification in the analysis.

### Model 4 — Quadratic Model

$$Y_{it} = \alpha + \beta_1 \cdot \text{Internet}_{it} + \beta_2 \cdot \text{Internet}_{it}^2 + \beta_3 \cdot \log(\text{GDP})_{it} + \lambda_t + \varepsilon_{it}$$

Introduces a quadratic internet term to test for nonlinearity. A significant negative β_2 confirms diminishing marginal returns. The turning point (penetration level at which marginal returns approach zero) is derived from the estimated coefficients: $\text{turning point} = -\hat{\beta}_1 / (2\hat{\beta}_2)$.

---

## 6. Clustering Methodology

The final analytical step uses **K-means clustering** to group countries by their investment profile. Rather than clustering on raw indicators alone, the feature set is constructed from regression-derived quantities that directly encode each country's position on the returns curve:

| Feature | Rationale |
|---|---|
| `internet_users_pct` | Raw connectivity level |
| `log_gdp_pc` | Economic development level |
| `internet_sq` | Nonlinear position on returns curve |
| `internet_gdp` | Interaction between connectivity and income |
| `marginal_return_proxy` | Estimated marginal return at observed penetration level |
| `internet_midrange_distance` | Distance from the 55% penetration midpoint |

Clustering is performed on the **earliest available observation per country** (`groupby('country_code').first()`) rather than the full panel. This is an important methodological choice: clustering on time-averaged or pooled data would mix countries' starting positions with their trajectories, potentially grouping a rapidly developing economy with a stagnant one simply because they share a similar average. Using the baseline observation captures where each country *entered* the internet expansion era.

All features are standardized using `StandardScaler` before clustering. K-means is sensitive to feature scale, so without standardization, features with larger numeric ranges (e.g., GDP) would dominate the distance calculations and swamp the regression-derived features.

**K = 4** clusters were chosen based on interpretability and alignment with the theoretical framework: the four groups correspond naturally to the four stages of the digital development curve identified in the regression analysis (pre-transition, early transition, mid-transition, saturation).

---

## 7. Limitations

{: .warning }
**Endogeneity.** The most important limitation of this analysis is that internet infrastructure is not randomly deployed. Providers and governments preferentially invest in areas with higher expected returns — denser, wealthier, better-governed populations. This means OLS estimates of the effect of connectivity on outcomes are likely upward-biased. The first-difference model partially addresses this by removing time-invariant confounders, but a fully credible causal estimate would require an instrument (e.g., submarine cable landings, terrain-based cost variation) that this project does not pursue. Results should be interpreted as **associations with causal language used cautiously** to indicate direction, not magnitude.

**Outcome measurement.** Secondary enrollment gross ratios can exceed 100%, secondary school quality varies enormously across countries, and life expectancy is influenced by many factors outside the reach of digital policy. These measures are the best available at global scale but are imperfect proxies for the underlying human development concepts.

**Temporal dynamics.** All models treat the relationship between connectivity and outcomes as contemporaneous. In reality, effects on education and health likely materialize with lags of several years. A distributed lag model would be a natural extension.

[See the Results →](results){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View the Code →](data){: .btn .fs-5 .mb-4 .mb-md-0 }