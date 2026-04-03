# 📉 Customer Churn Analysis

### End-to-end data analytics project using Python, SQL & Data Visualisation

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org)
[![Seaborn](https://img.shields.io/badge/Seaborn-Visualisation-4C8CBF?style=for-the-badge)](https://seaborn.pydata.org)
[![Status](https://img.shields.io/badge/Status-Completed-2ea44f?style=for-the-badge)](https://github.com)

> **Uncovering why telecom customers leave — and who's at risk next** — covering demographics, service usage, contract types, billing behaviour, and revenue at risk.

&nbsp;

| 📋 Records | 📊 Features | 📉 Churn Rate | 💰 Avg Monthly Charge |
|---|---|---|---|
| **7,043** | **20+** | **~26.5%** | **$64.76** |

---

## 📁 Project Structure

```
customer-churn-analysis/
│
├── 📓 Customer_Churn_Analysis.ipynb   ← Main Jupyter Notebook (EDA + SQL + Viz)
├── 📂 data/
│   └── Customer_Churn.csv            ← Raw dataset (7,043 rows × 21 cols)
└── 📂 visuals/
    └── *.png                         ← Exported chart images
```

---

## 🎯 Objective

Perform a **comprehensive analysis** of customer churn to answer:

- Who is churning — and why?
- Which contract types and services drive the most attrition?
- How much revenue has been lost, and how much more is at risk?
- Which customer segments should be targeted first for retention?
- Do bundled services reduce churn?

---

## 🔧 Tech Stack

| Layer | Tool |
|---|---|
| **Data Manipulation** | Python · Pandas · NumPy |
| **Visualisation** | Seaborn · Matplotlib |
| **Database** | PostgreSQL 15 via SQLAlchemy |
| **Environment** | Jupyter Notebook |

---

## 🧹 Data Cleaning & Feature Engineering

| Step | What Was Done |
|---|---|
| **Column Names** | Standardised all columns to `snake_case` using `.str.lower()` + rename dictionary |
| **Drop Column** | `customer_id` dropped — not needed for analysis |
| **Data Type Fix** | `total_charges` converted to numeric after stripping whitespace |
| **Simplify Categories** | `"No phone service"` / `"No internet service"` → `"No"` across 6 columns |
| **Label Encoding** | `senior_citizen` mapped from `0/1` → `"younger"/"senior"` |
| **New Feature** | `tenure_group` — 4 bins: `0–12 months / 13–24 months / 25–48 months / 48+ months` |

---

## 🗃️ SQL Analysis (PostgreSQL)

Data was loaded into PostgreSQL using `SQLAlchemy` and queried with `pd.read_sql()`.

**📌 Q1 — How Many Customers Have Churned?**

```sql
SELECT churn, COUNT(*)
FROM churn
GROUP BY churn;
```

| Churn | Count |
|---|---|
| No | 5,174 |
| Yes | 1,869 |

> **~26.5%** of customers have churned.

---

**📌 Q2 — What Type of Customers Are Churning?**

```sql
SELECT churn, COUNT(*) AS total_users,
    ROUND(AVG(CASE WHEN phone_service = 'Yes' THEN 1 ELSE 0 END) * 100, 1) AS pct_phone_service,
    ROUND(AVG(CASE WHEN online_security = 'Yes' THEN 1 ELSE 0 END) * 100, 1) AS pct_online_security,
    ROUND(AVG(CASE WHEN online_backup = 'Yes' THEN 1 ELSE 0 END) * 100, 1) AS pct_online_backup
FROM churn
GROUP BY churn;
```

> Churned customers have **significantly lower** adoption of online security and backup services.

---

**📌 Q3 — Which Contract Type Has the Most Churn?**

```sql
SELECT churn, COUNT(*) AS total_users,
    SUM(CASE WHEN contract = 'Month-to-month' THEN 1 ELSE 0 END) AS month_to_month,
    SUM(CASE WHEN contract = 'One year'       THEN 1 ELSE 0 END) AS one_year,
    SUM(CASE WHEN contract = 'Two year'       THEN 1 ELSE 0 END) AS two_year
FROM churn
GROUP BY churn;
```

| Contract | Churned | Retained |
|---|---|---|
| Month-to-month | 1,655 | 2,220 |
| One year | 166 | 1,307 |
| Two year | 48 | 1,647 |

> **Month-to-month** customers account for **~89%** of all churn.

---

**📌 Q4 — Are Senior Citizens More Likely to Churn?**

```sql
SELECT senior_citizen, COUNT(*) AS total_users,
    SUM(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) AS total_churned,
    ROUND(SUM(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS churn_rate_pct
FROM churn
GROUP BY senior_citizen;
```

| Segment | Total | Churned | Churn Rate |
|---|---|---|---|
| Senior | 1,142 | 476 | **41.7%** |
| Younger | 5,901 | 1,393 | **23.6%** |

> Senior citizens churn at nearly **2× the rate** of younger customers.

---

**📌 Q5 — How Much Revenue Was Lost to Churn?**

```sql
SELECT
    ROUND(SUM(monthly_charges::numeric), 2) AS total_revenue,
    ROUND(SUM(CASE WHEN churn = 'Yes' THEN monthly_charges::numeric ELSE 0 END), 2) AS revenue_lost,
    ROUND(SUM(CASE WHEN churn = 'Yes' THEN monthly_charges::numeric ELSE 0 END) * 100.0
        / SUM(monthly_charges::numeric), 1) AS pct_revenue_lost
FROM churn;
```

| Total Revenue | Revenue Lost | % Lost |
|---|---|---|
| $456,116 | $139,130 | **30.5%** |

---

**📌 Q6 — Revenue at Risk (Active Customers)**

```sql
SELECT COUNT(*) AS at_risk_users,
    ROUND(SUM(monthly_charges::numeric), 2) AS monthly_revenue_at_risk,
    ROUND(AVG(tenure), 1)                   AS avg_tenure,
    ROUND(AVG(monthly_charges::numeric), 2) AS avg_monthly_charge
FROM churn
WHERE churn = 'No'
  AND tenure <= 12
  AND monthly_charges::numeric >= 65
  AND contract = 'Month-to-month';
```

> High-value, short-tenure, month-to-month customers represent a **significant upcoming revenue risk**.

---

**📌 Q7 — Do More Services = Less Churn?**

```sql
WITH base AS (
    SELECT *,
        (CASE WHEN online_security  = 'Yes' THEN 1 ELSE 0 END +
         CASE WHEN online_backup    = 'Yes' THEN 1 ELSE 0 END +
         CASE WHEN tech_support     = 'Yes' THEN 1 ELSE 0 END +
         CASE WHEN streaming_tv     = 'Yes' THEN 1 ELSE 0 END +
         CASE WHEN streaming_movies = 'Yes' THEN 1 ELSE 0 END) AS service_count
    FROM churn
)
SELECT service_count, COUNT(*) AS total_customers,
    ROUND(AVG(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) * 100, 1) AS churn_rate_pct
FROM base
GROUP BY service_count
ORDER BY service_count;
```

> Customers with **4–5 services** churn at nearly **half the rate** of customers with 0 services.

---

**📌 Q8 — Payment Method vs Churn**

```sql
SELECT churn, paperless_billing,
    SUM(CASE WHEN payment_method = 'Electronic check'          THEN 1 ELSE 0 END) AS electronic_check,
    SUM(CASE WHEN payment_method = 'Mailed check'              THEN 1 ELSE 0 END) AS mailed_check,
    SUM(CASE WHEN payment_method = 'Bank transfer (automatic)' THEN 1 ELSE 0 END) AS bank_auto,
    SUM(CASE WHEN payment_method = 'Credit card (automatic)'   THEN 1 ELSE 0 END) AS credit_auto
FROM churn
GROUP BY churn, paperless_billing;
```

> **Electronic check** users churn at the highest rate — especially when combined with paperless billing.

---

**📌 Q9 — Which Segment to Target First for Retention?**

```sql
SELECT contract, internet_service,
    COUNT(*) AS total_users,
    SUM(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) AS churned,
    ROUND(AVG(monthly_charges::numeric), 2) AS avg_monthly,
    ROUND(SUM(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS churn_rate_pct,
    ROUND(SUM(CASE WHEN churn = 'Yes' THEN 1 ELSE 0 END) * AVG(monthly_charges::numeric), 2) AS revenue_at_risk
FROM churn
GROUP BY contract, internet_service
ORDER BY revenue_at_risk DESC;
```

> **Month-to-month + Fiber Optic** is the highest-risk segment — highest churn rate and highest revenue at risk combined.

---

## 📊 Python Visualisations

7 charts produced using **Seaborn** & **Matplotlib**:

| # | Chart | Type | Key Takeaway |
|---|---|---|---|
| 1 | Overall Churn Distribution | Count Plot | 26.5% churned |
| 2 | Contract Type vs Churn | Grouped Bar | Month-to-month = 3× higher churn |
| 3 | Tenure Distribution vs Churn | Histogram + KDE | New customers churn far more |
| 4 | Monthly Charges vs Churn | Box Plot | Churned customers pay ~$10 more/month |
| 5 | Churn % by Tenure Group | Bar (`Reds_r`) | 0–12 months = highest churn % |
| 6 | Internet Service vs Churn | Grouped Count | Fiber optic highest churn type |
| 7 | Correlation Heatmap | Heatmap (`coolwarm`) | Tenure negatively correlated with churn |

---

## 💡 Key Insights & Recommendations

| # | Insight | Recommendation |
|---|---|---|
| 1 | Month-to-month customers = ~89% of all churn | Offer incentives to upgrade to annual contracts |
| 2 | Senior citizens churn at 41.7% | Launch dedicated senior retention programme |
| 3 | New customers (0–12 months) churn the most | Improve onboarding experience in the first year |
| 4 | Fiber optic users churn more than DSL | Investigate service quality & pricing complaints |
| 5 | Customers with 4–5 services churn ~50% less | Push bundled service packages aggressively |
| 6 | Electronic check users churn most | Incentivise switching to auto-pay methods |
| 7 | 30.5% of monthly revenue was lost to churn | Prioritise M2M + Fiber segment for immediate intervention |

---

## 🚀 How to Run

### 1. Clone the repository

```bash
git clone https://github.com/your-username/customer-churn-analysis.git
cd customer-churn-analysis
```

### 2. Install dependencies

```bash
pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary jupyter
```

### 3. Set up PostgreSQL

```python
# Update credentials in the notebook (Cell 36)
username = 'your_username'
password = 'your_password'
host     = 'localhost'
port     = '5432'
database = 'Customer_Churn'
```

### 4. Launch Jupyter Notebook

```bash
jupyter notebook Customer_Churn_Analysis.ipynb
```

---

## 📦 Dependencies

```
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
sqlalchemy>=2.0
psycopg2-binary>=2.9
jupyter>=1.0
```

---

## 🗺️ Project Workflow

```
Raw CSV Data
     │
     ▼
Data Cleaning & Feature Engineering (Python / Pandas)
     │
     ├──► EDA & Key Metrics (Pandas GroupBy)
     │
     ├──► PostgreSQL Database (SQLAlchemy)
     │         │
     │         └──► SQL Business Queries (9 Questions)
     │
     └──► Visualisations (Seaborn / Matplotlib)
               │
               └──► Insights & Recommendations
```

---

## 🙋‍♂️ Author

Made with ❤️ by **[Your Name]**  
📧 your-email@example.com  
🔗 [LinkedIn](https://linkedin.com/in/your-profile) | [GitHub](https://github.com/your-username)

---

## ⭐ If you found this useful, drop a star!
