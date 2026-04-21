---
layout: default
title: Methods
nav_order: 2
---

# Methods

## Data Source

Primary dataset:

World Bank Open Data API

Indicators analyzed:

- Internet users (% population)
- GDP per capita
- Secondary school enrollment
- Life expectancy

---

## Data Pipeline Architecture

Pipeline workflow:

API extraction → AWS storage → SQL cleaning → Python modeling

Cloud infrastructure enables scalable cross-country comparison across indicators.

---

## Analytical Strategy

We estimate relationships between connectivity and development outcomes using:

- linear regression
- correlation analysis
- cross-country trend comparisons

Models evaluate whether connectivity predicts improvements across education, income, and health domains.