---
layout: project
title: Payment Transactions Analytics
summary: Built an end-to-end payment pipeline processing 49,943 transactions — identified a systemic 58% failure rate and 83% timestamp coverage gap across 2023–2024, surfaced through Python, MySQL, and Power BI.
tech: Python · SQL · MySQL · Power BI · Pandas · SQLAlchemy
timeline: 6 weeks
dataset_size: 49,943 transactions · 2023–2024
github: https://github.com/Justsharon/payment_transactions_analytics
status: complete
---

## Business Problem

A payment processing dataset spanning two years and 13 payment methods had never been systematically analysed. Raw data contained inconsistent country codes, mixed-case currencies, 16 variants of the same status values, and no documented data quality tracking.

**Goal:** Build a production-quality pipeline that cleans, stores, and analyses the data — and surface the findings in an executive dashboard that answers one question: *why is the payment system performing this badly, and how long has it been this way?*

## Data

- 49,943 transaction records across 2023–2024
- 14 raw columns: transaction_id, timestamp, customer_id, merchant_name, amount, currency, payment_method, status, card_last4, billing_country, processing_fee, ip_address, email, phone
- 6 currencies: USD, EUR, GBP, CAD, AUD, JPY
- 13 payment methods: PayPal, credit card, debit card, ACH, Apple Pay, Google Pay, crypto, Bitcoin, wire transfer, bank transfer, Cash App, Venmo, unknown

## Pipeline Architecture

```
Raw CSV
  │
  ▼
data_transformation.py
  │  Schema enforcement and type casting
  │  Fixed-precision Decimal for all monetary fields
  │  35-variant country normalisation dictionary
  │  Status consolidation (16 raw variants → 10 canonical)
  │  Duplicate detection and isolation
  │  7 boolean QA flags added
  │  Timestamp sentinel fill AFTER flagging (preserves source truth)
  ▼
transactions_fact.parquet  (staging layer — full enriched dataset)
  │
  ├──▶ load_data.py → MySQL 8
  │       Composite indexes on 6 dimensions
  │       vw_transactions_enriched analytical view
  │       Daily summary table and cohort retention logic
  │
  └──▶ run_export.py → export_func.py → transactions_bi.csv
          Strict 15-column contract
          Raises on any contract violation
          Amounts as fixed-precision strings, booleans as true/false
```

## Key Engineering Decisions

**Financial precision:** Every monetary value is handled as Python `Decimal` — never cast to float. Quantized to 2 decimal places with `ROUND_HALF_UP`. This ensures consistency across Python, MySQL `DECIMAL(18,2)`, Parquet, Excel, and Power BI.

**Normalisation at source:** All cleaning happens in `data_transformation.py` before the Parquet is written. A previous version of the pipeline applied country normalisation only during MySQL load — meaning the BI CSV never received those fixes. Moving it upstream ensures both export paths inherit clean data.

**Sentinel after flag:** The `is_missing_timestamp` flag is set on the raw null value, then the sentinel `1900-01-01` is filled. This preserves the accuracy of the flag — it reflects source truth, not the corrected value.

**Fail loudly:** The pipeline raises on null `transaction_id`, duplicate primary keys, and BI contract violations. Silent coercion hides data problems. Raising exposes them at the earliest possible point.

## Data Quality Findings

| Issue | Count | % | Resolution |
|---|---|---|---|
| Missing timestamps | 41,585 | 83.3% | Flagged, sentinel filled, excluded from time-series |
| Duplicate transaction_ids | 114 | 0.2% | Isolated to report, latest record retained |
| Non-standard country codes | ~15,000 | ~30% | Mapped via 35-variant ISO dictionary |
| Mixed-case currencies | ~18,000 | ~36% | Uppercased and validated to ISO 4217 |
| Inconsistent status values | 16 variants | — | Consolidated to 10 canonical values |

## Analytical Findings

**Finding 1 — Systemic failure, not isolated incidents**
58.3% of transactions failed. The failure rate is consistent across all 13 payment methods (56–59%), indicating a processor-level or classification problem rather than method-specific failures. Failures split into two distinct types: hard processing failures (32% of failures) and post-payment refunds/reversals (22%). A single fix will not resolve both.

**Finding 2 — Two years of flatline**
Monthly success rate has oscillated between 14–24% from January 2023 to December 2024 with no upward trajectory. Year-on-year improvement: 1.7 percentage points. At this rate, reaching the 95% target would take approximately 80 years. The problem is structural.

**Finding 3 — 22% permanently unresolved**
1,842 transactions remain in pending, processing, or unknown status — neither confirmed successful nor formally failed. If these resolve as failures, the true failure rate climbs from 58.3% to approximately 69%. The headline metric may be optimistic.

## What I Would Do With More Time

- Investigate the root cause of 83% missing timestamps — whether a single load event or a systematic ingestion failure
- Add exchange rate data to produce a single normalised TPV figure
- Build a fraud detection model using the failure pattern data by country and payment method
- Automate daily pipeline runs with failure alerting

## Tech Stack

Python · Pandas · SQLAlchemy · MySQL 8 · Power BI · Parquet · Git