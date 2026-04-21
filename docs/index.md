---
layout: home
title: Home
nav_order: 1
---

# Internet Connectivity & Development Outcomes
{: .fs-9 }

Policy Evidence from Global Broadband Expansion
{: .fs-5 .fw-300 }

---

{: .highlight }
> **Does expanding internet access independently improve human development — or does it simply reflect the wealth of countries that can already afford it?** This project uses two decades of World Bank data across 180+ countries to disentangle the causal story and identify where digital infrastructure investment generates the highest returns.

---

## The Policy Problem

Internet access has become a defining axis of global inequality. As of the early 2020s, roughly one-third of the world's population remains offline — a gap concentrated overwhelmingly in Sub-Saharan Africa, South Asia, and parts of Latin America and the Middle East. Governments, multilateral institutions, and development banks have committed hundreds of billions of dollars to connectivity expansion under the assumption that internet access drives human development.

But that assumption deserves scrutiny. Wealthy countries have both better internet infrastructure *and* better health and education systems. Richer governments can invest in all of these simultaneously. The risk is that much of the apparent "effect" of internet access on development is simply a reflection of underlying wealth — and that connectivity investments in low-income settings will underperform if the economic and institutional conditions for absorbing those investments are not in place.

This project takes that risk seriously. Rather than reporting raw correlations, we build a **progressive empirical framework** — controlling for income, introducing year fixed effects to absorb global time trends, modeling interaction and nonlinear effects, and ultimately clustering countries by their position on the returns curve — to arrive at actionable investment guidance.

---

## Research Questions

1. Is internet penetration independently associated with better education and health outcomes, after controlling for income and global time trends?
2. Does the effect of connectivity on development vary by a country's income level?
3. Are there diminishing returns to connectivity, and where on that curve are countries positioned?
4. Which countries represent the highest-return targets for digital infrastructure investment?

---

## Data at a Glance

Four World Bank indicators, pulled directly from the World Bank API, covering **2000–2023** across all reporting countries:

| Indicator | Code | Role |
|---|---|---|
| Internet users (% of population) | `IT.NET.USER.ZS` | Treatment variable |
| GDP per capita (current USD) | `NY.GDP.PCAP.CD` | Core control |
| Life expectancy at birth (years) | `SP.DYN.LE00.IN` | Health outcome |
| Secondary school enrollment (%) | `SE.SEC.ENRR` | Education outcome |

Data are stored in a cloud-based PostgreSQL database on AWS RDS and processed through a documented Python pipeline — fully reproducible from raw API call to final model output.

---

## Key Findings

{: .note }
**Finding 1 — Internet access has an independent positive effect on education, but income dominates.**
Controlling for GDP per capita, a 1 percentage point increase in internet penetration is associated with a **+0.17 pp increase in secondary enrollment**. A one-unit increase in log GDP per capita is associated with a **+12 pp increase** — confirming that economic development is the primary structural predictor, but connectivity has a distinct and measurable role.

{: .note }
**Finding 2 — Returns to connectivity are highest in lower-income countries.**
An interaction model shows a negative interaction between internet access and GDP (−0.130), meaning internet access has its **strongest marginal effect on education in poorer countries**. The benefit diminishes as countries grow wealthier, with implications for where investment is best directed.

{: .note }
**Finding 3 — Within-country increases in connectivity improve life expectancy.**
A first-difference model controlling for global year shocks finds that a 1 pp increase in internet penetration is associated with a **+0.027-year increase in life expectancy** — a small but statistically significant within-country effect that survives controls for global trends.

{: .note }
**Finding 4 — Diminishing returns set in around 60–70% penetration.**
A quadratic specification confirms a concave relationship between internet access and secondary enrollment. Marginal returns approach zero near the 60–70% penetration threshold, identifying a clear policy window for countries still on the steep part of the curve.

---

## Investment Targeting

K-means clustering on regression-derived features places countries into four profiles:

| Cluster | Profile | Recommendation |
|---|---|---|
| **0** | Low connectivity, moderate income, high absorptive capacity | ⭐ Primary investment target |
| **1** | Very low connectivity, very low income | Foundational development preconditions needed first |
| **2** | Mid-income, approaching saturation (~50% penetration) | Secondary target; efficiency-focused investments |
| **3** | High income, fully saturated (>80% penetration) | Shift to advanced digital transformation policy |

[Read the Methods →](methods){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[See the Results →](results){: .btn .fs-5 .mb-4 .mb-md-0 }