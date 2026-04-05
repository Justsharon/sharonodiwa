---
layout: project
title: Transaction Fraud Detection & Risk Intelligence & Analytics
summary: End-to-end fraud investigation for a fictional mid-sized commercial bank using SQL, Python, and Power BI — from raw transaction data to a board-ready executive risk report.
image: assets/images/operation_clearwater_page-0001.jpg
tech: Python · SQL · MySQL · Power BI · Pandas · SQLAlchemy
timeline: 3 days
dataset_size: January–September 2024 across 6 tables in a star schema data warehouse
github: https://github.com/Justsharon/Operation_Clear_Waters
status: complete
---

<img 
  src="{{ 'assets/images/operation_clearwater_page-0001.jpg' | relative_url }}"
  alt="North Axis Analytics Dashboard"
  class="dashboard-preview" />


# Operation Clearwater — Transaction Fraud Detection & Risk Intelligence

> End-to-end fraud investigation for a fictional mid-sized commercial bank using SQL, Python, and Power BI — from raw transaction data to a board-ready executive risk report.

## Project Overview

NorthAxis Bank plc experienced a **340% spike in fraud complaints** during Q3 2024. As the junior analyst on the Risk Intelligence team, I was tasked with investigating suspicious transaction patterns across 9 months of data and delivering a board-ready report within 72 hours.

This project covers the full analytical pipeline:
- Data exploration and baseline KPI analysis
- Multi-signal fraud flag creation using SQL window functions
- Customer risk profiling and behavioural deviation detection
- Merchant and channel risk scoring
- Composite fraud risk scoring model in Python
- Executive dashboard in Power BI

## Business Questions Answered

| Question | Answer |
|---|---|
| How much is at risk? | $24.35M across 728 flagged transactions |
| How many high-risk accounts? | 26 accounts meeting 3+ fraud signals |
| When does fraud occur? | Peak at 2AM–3AM via digital channels |
| Which channels are implicated? | Mobile Banking and Web Banking dominate |
| Which merchants are highest risk? | Retail, Travel, Food & Dining |
| Any regulatory concerns? | Yes — transactions from sanctioned jurisdictions detected |

## Repository Structure

```
operation-clearwater/
│
├── sql/
│   ├── 01_baseline_kpis.sql
│   ├── 02_flag_creation_view.sql
│   ├── 03_customer_risk_profiling.sql
│   ├── 04_merchant_channel_risk.sql
│   └── 05_top_risk_customers.sql
│
├── python/
│   └── fraud_risk_scoring.ipynb
│
├── powerbi/
│   └── operation_clearwater.pbix
│
├── data/
│   └── (CSV exports from SQL queries)
│
├── docs/
│   └── PROJECT_DOCUMENTATION.md
│
└── README.md
```

## Tools & Technologies

| Tool | Purpose |
|---|---|
| MySQL 8.0 | Data exploration, window functions, flag creation, aggregation |
| MySQL Workbench | SQL IDE and CSV export |
| Python (pandas) | Fraud risk scoring and tier assignment |
| Jupyter Notebook | Python development environment |
| Power BI Desktop | Executive dashboard and report |

##  Fraud Detection Methodology

A transaction was classified as suspicious only if it met **3 or more** of the following four conditions simultaneously. This strict threshold was chosen to minimise false positives.

| Flag | Condition | SQL Technique |
|---|---|---|
| Velocity Anomaly | Same account, 2+ transactions within 10 mins, amount > mean + 3σ | `LAG()`, `TIMESTAMPDIFF()` |
| Off-Hours Activity | Transaction between 01:00–03:59 AM | `HOUR()`, `CASE WHEN` |
| Geographic Mismatch | Transaction country ≠ customer's registered country | `JOIN`, `CASE WHEN` |
| Amount Outlier | Amount > customer's channel-adjusted mean + 3σ | `AVG() OVER()`, `STDDEV() OVER()` |

All flags were combined into a `total_flag_count` column and saved as a reusable view (`suspicious_transactions_view`).

## Key SQL Concepts Used

- **Window Functions** — `LAG()`, `AVG() OVER()`, `STDDEV() OVER()`, `RANK()`, `ROW_NUMBER()`, `PARTITION BY`
- **CTEs** — chained Common Table Expressions for readable, modular query logic
- **Views** — saving flagging logic as a reusable database object
- **Star Schema Joins** — joining fact and dimension tables across a data warehouse
- **Statistical Outlier Detection** — z-score approach using mean and standard deviation per channel

## Power BI Dashboard

The final report consists of four pages:

**Page 1 — Executive Summary**
KPI cards (total exposure, flagged transactions, high-risk accounts), top 10 watchlist bar chart, off-hours transaction volume chart

**Page 2 — Transaction Anomalies**
Monthly trend (value + volume), flagged transactions by channel × hour, geographic distribution of suspicious activity

**Page 3 — Customer Watchlist**
Risk-ranked table with conditional formatting by risk tier, average transaction value, most common suspicious hour

**Page 4 — Merchant Watchlist**
Flagged transaction value by channel, flagged transaction count by merchant category


## Key Findings

1. **$24.35M** in at-risk transaction value identified — exceeding the initial $2.3M audit estimate by 10x
2. Suspicious activity **peaks at 2AM–3AM** almost exclusively on digital channels
3. **Sanctioned jurisdictions** (North Korea, Iran, Syria, Cuba) detected in transaction origins — potential regulatory compliance breach
4. **Mobile Banking and Web Banking** are disproportionately implicated relative to their share of overall volume
5. **26 customers** met the strict 3-flag threshold for high-risk classification

## Assumptions & Caveats

- Off-hours window (01:00–03:59 AM) and velocity window (10 minutes) are analytical assumptions, not derived from historical fraud data
- Outlier threshold set at mean + 3 standard deviations — borderline cases between 2σ and 3σ are not flagged
- Risk tier boundaries based on analytical judgement — should be validated by the Risk team before operational use
- $24.35M represents total value of flagged transactions, not confirmed fraud losses — actual losses will be lower following manual review
- Data covers January–September 2024 only

## What I Learned

- How to structure a fraud investigation as a hypothesis-driven analytical process
- SQL window functions (`LAG`, `PARTITION BY`, `STDDEV OVER`) for temporal and statistical anomaly detection
- The difference between flagging at transaction level vs. profiling at customer level
- Why proportional risk scoring (flagged %) is more meaningful than raw counts for merchant/channel analysis
- How to build a star schema data model in Power BI and connect pre-aggregated SQL exports
- The importance of documenting analytical assumptions as caveats in executive reports


## Data Source

Fictional dataset created for the Operation Clearwater training brief by NorthAxis Bank plc (Risk Intelligence Division). Data covers January–September 2024 across 6 tables in a star schema data warehouse.


*Built as part of a structured data analytics training programme. All bank names, customer names, and transaction data are fictional.*