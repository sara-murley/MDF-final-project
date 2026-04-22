---
layout: default
title: Data & Code
nav_order: 4
---

# Data & Code
{: .fs-8 }

Infrastructure, notebooks, and instructions for reproducing this analysis
{: .fs-5 .fw-300 }

---

{: .highlight }
> All code is publicly available in the [GitHub repository](https://github.com/sara-murley/MDF-final-project). The analysis runs end-to-end in two Jupyter notebooks — one for data ingestion and one for analysis — backed by a cloud data infrastructure on AWS.

---

## Environment Setup

### Prerequisites

- Python 3.9+
- AWS credentials configured (for S3 and RDS access)
- PostgreSQL client libraries (`libpq`)

```bash
# Clone the repository
git clone https://github.com/sara-murley/MDF-final-project.git
cd MDF-final-project

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

| Package | Purpose |
|---|---|
| `boto3` | AWS S3 interaction |
| `pandas` | Data wrangling and panel construction |
| `sqlalchemy` / `psycopg2` | PostgreSQL connection and query |
| `numpy` | Numerical operations (log transform, quadratic term) |
| `statsmodels` | OLS regression with fixed effects |
| `scikit-learn` | K-means clustering, StandardScaler |
| `seaborn` / `matplotlib` | Static visualization |
| `requests` | World Bank API calls |

---

## Notebook 1: Data Ingestion (`01-get-data.ipynb`)

This notebook handles the full data pipeline from API to database. It is designed to be run once (or re-run when updated data is needed) and does not need to be re-executed for analysis.

### What it does

1. Calls the World Bank REST API for each of the four indicators, retrieving the full 2000–2023 time series in a single paginated request
2. Parses JSON responses into tidy dataframes and outer-merges them on `country_code` × `year`
3. Filters to sovereign countries (ISO3 codes only), sorts, and timestamps the dataset
4. Uploads the raw JSON and cleaned CSV to Amazon S3
5. Loads the cleaned panel into a PostgreSQL table on Amazon RDS

### Core ingestion loop

```python
INDICATORS = {
    "IT.NET.USER.ZS": "internet_users_pct",
    "NY.GDP.PCAP.CD": "gdp_per_capita",
    "SP.DYN.LE00.IN": "life_expectancy",
    "SE.SEC.ENRR":    "secondary_enrollment_pct",
}

merged = None

for indicator_code, column_name in INDICATORS.items():
    url = f"https://api.worldbank.org/v2/country/all/indicator/{indicator_code}"
    params = {"date": "2000:2023", "format": "json", "per_page": 20000}

    response = requests.get(url, params=params, timeout=60)
    records = response.json()[1]

    df_temp = pd.DataFrame([
        {
            "country_code": r["countryiso3code"],
            "country_name":  r["country"]["value"],
            "year":          int(r["date"]),
            column_name:     float(r["value"])
        }
        for r in records
        if r.get("value") is not None and r.get("countryiso3code")
    ])

    merged = df_temp if merged is None else merged.merge(
        df_temp, on=["country_code", "country_name", "year"], how="outer"
    )

# Drop regional aggregates (keep only ISO3 country codes)
merged = merged[merged["country_code"].str.len() == 3].copy()
```

{: .note }
**Why `per_page=20000`?** The World Bank API defaults to returning 50 records per page. Setting a large page size retrieves the full indicator time series in one request, eliminating pagination complexity and reducing API calls from hundreds to one per indicator.

### S3 upload

```python
# Raw JSON — full audit trail back to source
s3.put_object(
    Bucket="world-bank-project-data",
    Key="raw/worldbank_raw.json",
    Body=json.dumps(all_raw_data).encode("utf-8"),
    ContentType="application/json"
)

# Cleaned CSV — analysis-ready panel
csv_buffer = StringIO()
merged.to_csv(csv_buffer, index=False, na_rep="NULL")
s3.put_object(
    Bucket="world-bank-project-data",
    Key="clean/worldbank_panel.csv",
    Body=csv_buffer.getvalue().encode("utf-8"),
    ContentType="text/csv"
)
```

{: .note }
**Why `na_rep="NULL"`?** Writing missing values as `NULL` rather than empty strings ensures PostgreSQL correctly infers them as SQL `NULL` when the CSV is loaded, preventing type coercion errors on numeric columns.

### RDS load

```python
engine = create_engine(
    f"postgresql+psycopg2://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
)

merged.to_sql("worldbank_panel", engine, if_exists="replace", index=False)
```

---

## Notebook 2: Analysis (`02-analysis.ipynb`)

This notebook reads the panel from RDS, runs all five analytical stages, and generates the figures used throughout the site. Each section is self-contained and can be run independently once the database connection is established.

### Database connection

```python
import psycopg2, pandas as pd

conn = psycopg2.connect(
    host="worldbank-db.c8hyu8gkak7d.us-east-1.rds.amazonaws.com",
    port=5432, dbname="postgres",
    user="postgres", sslmode="require"
)

df = pd.read_sql("SELECT * FROM worldbank_panel", conn)
```

### Data preparation

```python
import numpy as np

# Drop rows with missing values in any key variable
df = df.dropna(subset=[
    'internet_users_pct', 'gdp_per_capita',
    'life_expectancy', 'secondary_enrollment_pct'
]).copy()

# Log-transform GDP per capita
df['log_gdp_pc'] = np.log(df['gdp_per_capita'])
```

### Correlation analysis

```python
import seaborn as sns, matplotlib.pyplot as plt, statsmodels.api as sm

corr = df[['internet_users_pct', 'log_gdp_pc',
           'life_expectancy', 'secondary_enrollment_pct']].corr()

plt.figure(figsize=(8, 6))
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, square=True, linewidths=0.5)
plt.title("Correlation Matrix: Internet Access and Socioeconomic Outcomes")
```

### Baseline OLS

```python
X = sm.add_constant(df[['internet_users_pct', 'log_gdp_pc']])
y = df['secondary_enrollment_pct']
model_baseline = sm.OLS(y, X).fit()
print(model_baseline.summary())
```

### Interaction model with year fixed effects

```python
df['internet_x_gdp'] = df['internet_users_pct'] * df['log_gdp_pc']
year_dummies = pd.get_dummies(df['year'], drop_first=True).astype(float)

X = sm.add_constant(pd.concat([
    df[['internet_users_pct', 'log_gdp_pc', 'internet_x_gdp']].astype(float),
    year_dummies
], axis=1))

model_interaction = sm.OLS(df['secondary_enrollment_pct'].astype(float), X).fit()
print(model_interaction.summary())
```

### Within-country first-difference model

```python
df = df.sort_values(['country_code', 'year'])
df['internet_change']  = df.groupby('country_code')['internet_users_pct'].diff()
df['life_exp_change']  = df.groupby('country_code')['life_expectancy'].diff()

df_diff = df.dropna(subset=['internet_change', 'life_exp_change']).copy()
year_dummies_diff = pd.get_dummies(df_diff['year'], drop_first=True).astype(float)

X = sm.add_constant(pd.concat([
    df_diff[['internet_change']].astype(float),
    year_dummies_diff
], axis=1))

model_fd = sm.OLS(df_diff['life_exp_change'].astype(float), X).fit()
print(model_fd.summary()) 
```

### Quadratic model

```python
df['internet_sq'] = df['internet_users_pct'] ** 2
year_dummies = pd.get_dummies(df['year'], drop_first=True).astype(float)

X = sm.add_constant(pd.concat([
    df[['internet_users_pct', 'internet_sq', 'log_gdp_pc']].astype(float),
    year_dummies
], axis=1))

model_quad = sm.OLS(df['secondary_enrollment_pct'].astype(float), X).fit()
print(model_quad.summary())
```

### K-means clustering

```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

# Baseline snapshot: first observation per country
df_base = df.sort_values('year').groupby('country_code').first().reset_index()

# Construct regression-derived features
df_base['internet_sq']               = df_base['internet_users_pct'] ** 2
df_base['internet_gdp']              = df_base['internet_users_pct'] * df_base['log_gdp_pc']
df_base['marginal_return_proxy']     = 0.9021 + 2 * (-0.0057) * df_base['internet_users_pct']
df_base['internet_midrange_distance']= (df_base['internet_users_pct'] - 55).abs()

features = df_base[[
    'internet_users_pct', 'log_gdp_pc', 'internet_sq',
    'internet_gdp', 'marginal_return_proxy', 'internet_midrange_distance'
]].dropna()

X_scaled = StandardScaler().fit_transform(features)

kmeans = KMeans(n_clusters=4, random_state=42, n_init=20)
df_base.loc[features.index, 'investment_cluster'] = kmeans.fit_predict(X_scaled)

print(df_base.groupby('investment_cluster').agg(
    n=('country_code', 'count'),
    internet=('internet_users_pct', 'mean'),
    gdp=('log_gdp_pc', 'mean'),
    enrollment=('secondary_enrollment_pct', 'mean'),
    life_exp=('life_expectancy', 'mean')
))
```

---


[View the Results →](results){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[GitHub Repository →](https://github.com/sara-murley/MDF-final-project){: .btn .fs-5 .mb-4 .mb-md-0 }