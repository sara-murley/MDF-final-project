---
layout: default
title: About
nav_order: 5
---

# About This Project
{: .fs-8 }

---

## Overview

This project was completed as part of **PPOL 5206: Massive Data Fundamentals** at the Georgetown University McCourt School of Public Policy. It examines the relationship between internet connectivity and human development outcomes — specifically secondary education enrollment and life expectancy. 

The analysis is built on a cloud-based data infrastructure (AWS S3 + RDS), a progressive sequence of OLS regression models, and a K-means clustering framework designed to translate statistical findings into policy-relevant investment guidance.

## Authors

**Sara Murley, Mena Tetali, Samantha Rudra**    
Master of Science, Data Science for Public Policy
Georgetown University McCourt School of Public Policy  

## Tools & Infrastructure

This project was built entirely in Python and deployed on AWS and GitHub Pages.

| Layer | Tool |
|---|---|
| Data ingestion | World Bank REST API via `requests` |
| Cloud storage | Amazon S3 |
| Database | Amazon RDS (PostgreSQL) |
| Analysis | `pandas`, `numpy`, `statsmodels`, `scikit-learn` |
| Visualization | `seaborn`, `matplotlib` |
| Website | Jekyll + Just the Docs, hosted on GitHub Pages |


## Acknowledgments

Thank you to the PPOL 5206 teaching team at Georgetown University. Data sourced from the World Bank World Development Indicators database.