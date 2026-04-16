# The Impact of Internet Connectivity on Socioeconomic Outcomes 

**Authors**: Sara Murley, Samantha Rudravajhala, Mena Tetali    
**Course**: PPOL 5206 – Massive Data Fundamentals    
**Institution**: Georgetown University   
**Semester**: Spring 2026      

## Project Overview 

This project investigates how internet connectivity influences key socioeconomic outcomes across countries over time. Using data from the World Bank Open Data API and a cloud-based analytics pipeline built with Snowflake and Python, we construct a cross-country panel dataset to estimate relationships between internet access and indicators of education, income, and health.

The project demonstrates the use of scalable infrastructure and distributed SQL workflows to support policy-relevant analysis in a reproducible environment.

## Research Question 

How does internet access and connectivity impact socioeconomic outcomes such as education, income, and health?

### Policy Motivation 

Internet access is a critical resource in modern economies and may influence inequality across countries.

Understanding connectivity gaps helps policymakers evaluate:

- access to remote education opportunities
- economic development potential
- population-level health improvements
- digital inclusion strategies

This project contributes to policy conversations in:

*Education*: Internet access supports remote learning and digital literacy development.
*Economic Development*: Connectivity enables workforce participation, entrepreneurship, and innovation. 
*Public Health*: Internet infrastructure improves access to telehealth services and health information.

## Repository Structure

```{text}
project-root/
│
├── code/
│   ├── 01-get-data.md
│   └── 02-analysis.ipynb
│
├── deliverables/
│   ├── presentation.pdf
│   └── policy_paper.pdf
│
├── figures/
│
├── website/
│
└── README.md
```

### Folder Descriptions 

- `code`: Contains files and notebooks documenting the full data workflow. 
- `deliverables`: Contains final course outputs (presentation, research paper). 
- `figures`: Stores figures generated from notebooks and used in deliverables. 
- `website`: Contains source files for the public-facing GitHub Pages project website. 

## Data Sources

Data are retrieved from the World Bank Open Data API ([documentation here](https://data360.worldbank.org/en/api#/Search/post_data360_searchv2))

Indicators used in this project:

- Internet Users (% of Population)
- GDP per capita
- Life Expectancy
- Secondary School Enrollment (%)

The dataset is structured as a country–year panel dataset.