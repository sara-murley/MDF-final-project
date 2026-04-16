# Data

## Source

Data are retrieved from the World Bank Open Data API.

## Indicators

- Internet Users (% of population)
- GDP per capita
- Life expectancy
- Secondary school enrollment

## Structure

We construct a **country-year panel dataset**.

## Pipeline

- API extraction (JSON format)
- Storage in AWS 
- Transformation and merging in SQL + Python