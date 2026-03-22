---
layout: project
title: Fintech Personal Loan Portfolio — EDA Project Summary
summary: Run a structured exploratory data analysis in Python to answer five strategic questions the CEO needs resolved before making underwriting, pricing, and marketing decisions for the next quarter. Surface the findings in a Power BI executive dashboard.
image: assets/images/loan data analysis_page-0001.jpg
tech: Python (Pandas, NumPy, Matplotlib, Seaborn) · Power BI
timeline: 1 week
dataset_size: 250,000 loans · 17 columns · March 2024 – February 2026
github: https://github.com/Justsharon/loan_analysis
status: complete
---

<img 
  src="{{ '/assets/images/loan data analysis_page-0001.jpg' | relative_url }}"
  alt="Loan Data Analytics Dashboard"
  class="dashboard-preview" />


## Business Problem

A 250,000-record personal loan portfolio spanning two years had never been systematically analysed. No documented profitability tracking existed by credit tier, no vintage cohort comparison had been run, and no quantified concentration risk had been established. Loan status distributions and borrower demographic performance were entirely unknown.

**Goal:** Run a structured exploratory data analysis in Python to answer five strategic questions the CEO needs resolved before making underwriting, pricing, and marketing decisions for the next quarter. Surface the findings in a Power BI executive dashboard.


## Dataset at a Glance

| Dimension | Detail |
|---|---|
| Total records | 250,000 loans |
| Total book value | $3.58B |
| Average loan amount | $14,312 |
| Average interest rate | 16.96% |
| Portfolio default rate | 10.14% |
| Net expected yield | 6.83% |
| Total expected loss | $362.70M |
| Portfolio at risk (PAR) | 0.11 |
| Date range | March 2024 – February 2026 (730 unique issue dates) |
| Loan purposes | 5 (Debt Consolidation, Home Improvement, Medical, Education, Venture) |
| Credit score range | 300 – 850 (539 unique values) |
| Employment length categories | 5 (< 1yr, 1–3yrs, 4–6yrs, 7–10yrs, 10+yrs) |
| States covered | 28 US states across 4 regions |

> **How the KPIs are calculated:** Default rate = proportion of loans with status "Defaulted". Net expected yield = average interest rate (16.96%) minus default rate expressed as a percentage (10.14%) = **6.83%**. Total expected loss = default rate × total book value (10.14% × $3.58B) = **$362.70M**. PAR is the raw default rate displayed as a decimal = **0.11**.

**Portfolio composition:**

| Status | Count | % of Portfolio |
|---|---|---|
| Fully Paid | 134,397 | 53.8% |
| Current | 74,949 | 30.0% |
| Defaulted | 25,342 | 10.1% |
| Charged Off | 15,312 | 6.1% |



## EDA Pipeline

```
fintech_loan_portfolio_250k.csv
  │
  ▼
EDA.ipynb
  │  Shape and dtype validation — 250,000 rows × 17 columns, zero nulls
  │  Age group binning: 21–30 · 31–40 · 41–50 · 51–60 · 61–75
  │  Loan amount grouping: Low · Medium · High · Very High
  │  Credit score banding: Poor (300–580) · Fair (580–670) · Good (670–740)
  │                         Very Good (740–800) · Exceptional (800–850)
  │  is_default flag engineered — Defaulted = 1, all other statuses = 0
  │  lti_ratio computed — loan_amount ÷ annual_income
  │  issue_date cast to datetime, issue_month period extracted
  │  expected_yield computed — interest_rate − (default_rate × 100)
  ▼
Analytical outputs:
  ├── Segment profitability table (Q1)
  ├── Monthly vintage default trend chart (Q2)
  ├── Loan purpose concentration bubble chart (Q3)
  ├── DTI bucket default rate scatter (Q4)
  └── Age × employment net yield matrix (Q5)
  │
  ▼
Power BI Executive Dashboard (6 pages)
  ├── Executive Summary — 4 KPI cards + default rate bar chart by credit tier
  ├── Risk vs Reward — grouped bar: avg interest rate vs net expected yield
  ├── Loan Quality Leak — dual-axis: monthly volume (bars) + default rate (line)
  ├── Concentration Risk & Debt Trap — bubble chart: volume vs default rate by purpose
  ├── Payment Shock Threshold — scatter: DTI bin vs default rate
  └── Efficiency & Borrower Demographics — matrix: employment length × age group
```



## CEO Question 1 — Risk vs. Reward: Are Poor and Fair Credit Segments Profitable?

> *"We are charging high-risk borrowers more interest, but is it enough? Are Poor and Fair credit segments actually profitable after accounting for their default rates?"*

**Method:** Credit scores binned into five tiers (notebook cell 16). For each tier, computed average interest rate, default rate, and expected yield using `expected_yield = interest_rate − (default_rate × 100)` (notebook cell 20). Visualised as a grouped bar chart in Power BI — gold bars for average interest rate, navy bars for net expected yield.

| Credit Tier | Score Range | Loans | Avg Interest Rate | Default Rate | Net Expected Yield |
|---|---|---|---|---|---|
| Poor | 300–580 | 42,675 | 24.69% | 19.30% | **5.39%** |
| Fair | 580–670 | 81,353 | 19.24% | 11.04% | **8.19%** |
| Good | 670–740 | 67,255 | 15.11% | 6.60% | **8.51%** |
| Very Good | 740–800 | 36,624 | 11.64% | 6.25% | **5.38%** |
| Exceptional | 800–850 | 22,084 | 8.14% | 6.32% | **1.82%** |

**Finding:** Poor credit is profitable at a net yield of 5.39% — but the margin is dangerously thin. A 5 percentage point deterioration in their default rate would make the segment net-negative. Fair and Good credit are the two most profitable tiers at 8.19% and 8.51% respectively and represent the primary growth opportunity.

The most counterintuitive result in the dataset: **Good credit (670–740) is more profitable than Exceptional credit (800–850)** — 8.51% vs 1.82%. The highest-rated borrowers are generating the lowest yield because their low risk profile does not compress the rate enough to produce meaningful margin. Very Good borrowers (740–800) produce almost identical yield to Poor borrowers (5.38% vs 5.39%) at a fraction of the default risk — a structural pricing inefficiency.

**Recommended action:** Raise the interest rate floor for Poor credit borrowers by 150–200bps to widen the safety buffer. Introduce a minimum net yield threshold of 8% as an approval gate for any application with a credit score below 620. Reprice Exceptional credit borrowers upward by 150bps — the current rate does not justify the capital deployed to this tier.


## CEO Question 2 — Vintage Quality Leak: Are Newer Loans Riskier Than Older Ones?

> *"Is the default rate of our new loans higher than loans issued a year ago at the same point in their lifecycle?"*

**Method:** Engineered `is_default` flag (Defaulted = 1). Cast `issue_date` to datetime and extracted `issue_month` as a period. Resampled by month to compute volume and default rate simultaneously. Plotted as a dual-axis chart in Power BI — navy bars for origination volume (left axis, $0.0bn–$0.2bn), gold line for monthly default rate (right axis, 0.10–0.12). Also plotted as a single-series line in the notebook (vintage_perf, dark red, marker='s').

**Monthly default rate — full 24-month record:**

| Period | Monthly Default Rate | Volume | Signal |
|---|---|---|---|
| March 2024 | 10.12% | $200.2M | Opening baseline |
| April 2024 | **9.45%** | $174.6M |  Lowest month in dataset |
| May–June 2024 | 9.70%–10.02% | — | Stable |
| July 2024 | 10.88% | $115.4M |  Deterioration begins |
| **August 2024** | **11.06%** | $119.1M |  **Peak — worst month in dataset** |
| September–December 2024 | 10.42%–10.70% | — | Elevated, not recovering |
| January–February 2025 | 10.41%–9.57% | — |  Recovery |
| March–May 2025 | 9.81%–9.87% | — |  Best sustained stretch |
| June–December 2025 | 10.03%–10.44% | — |  Stable, slight uptick |
| January 2026 | **9.51%** | $186.2M |  Second lowest |
| February 2026 | 10.44% | $157.2M |  Watch |

**Full range across 24 months: 9.45% (April 2024) to 11.06% (August 2024) — a 1.61 percentage point spread.** This is the 10%–12% oscillation band visible in the Power BI dual-axis chart.

**Year-on-year comparison:**

| Year | Avg Default Rate |
|---|---|
| 2024 | 10.29% |
| 2025 | 10.05% |
| Change | −0.24pp |

**Finding:** There is no confirmed vintage quality leak. The year-on-year improvement of 0.24 percentage points confirms underwriting standards held during growth — months with the highest origination volumes do not consistently produce the highest default rates. However, H2 2024 (July–December) produced the weakest cohorts in the portfolio, peaking at 11.06% in August. The portfolio recovered through H1 2025 but has not shown a sustained downward trend. A healthy, seasoning book should produce improving cohort performance over time — ours is oscillating within a narrow band rather than improving structurally.

**Recommended action:** Implement a formal vintage cohort tracker. Flag any cohort whose 6-month default rate exceeds the prior year's same-month cohort by more than 1 percentage point as an automatic escalation. Tag H2 2024 originations as a priority watch list as they approach full maturity.


## CEO Question 3 — Concentration Risk: Which Loan Purpose Will Sink Us in a Downturn?

> *"If the economy dips, which loan purpose will cause us to lose money? Show me where our money is bunched up and the risk level of those clusters."*

**Method:** Grouped by `loan_purpose`. Computed volume, default rate, and average loan size per category. Visualised as a bubble chart in Power BI — x-axis: total loan volume ($0.2bn–$1.6bn), y-axis: default rate (0.100–0.102), bubble size: loan volume, colour: loan purpose.

| Loan Purpose | Loans | Volume | % of Book | Default Rate | Avg Loan Size | Expected Loss |
|---|---|---|---|---|---|---|
| Debt Consolidation | 99,723 | $1,423.2M | **39.8%** | 10.11% | $14,272 | **$143.8M** |
| Home Improvement | 50,177 | $719.9M | 20.1% | 10.09% | $14,348 | $72.6M |
| Medical | 37,926 | $543.3M | 15.2% | 10.34% | $14,325 | $56.2M |
| Education | 37,453 | $538.9M | 15.1% | **9.96%** | $14,390 | $53.7M |
| Venture | 24,721 | $352.7M | 9.9% | 10.32% | $14,266 | $36.4M |

**What the bubble chart reveals:** All five loan purposes cluster between default rates of 0.100 and 0.102 on the y-axis — an exceptionally narrow band of 0.38 percentage points. The bubbles are not separated by risk behaviour; they are separated by volume. **Debt Consolidation's danger is not that its borrowers default more often — they default at almost exactly the same rate as every other purpose. The risk is purely concentration.** At 39.8% of the book ($1,423.2M), Debt Consolidation accounts for $143.8M of the $362.70M total expected loss — 39.7% of all losses from a single category.

In a recessionary scenario, Debt Consolidation borrowers are structurally the most fragile. They were already over-leveraged when they applied. A 2 percentage point deterioration in their default rate — not an extreme assumption under economic stress — would add approximately $28.5M to expected losses. Education loans are the most defensive segment: lowest default rate (9.96%), balanced volume (15.1%), highest average loan size ($14,390).

**Recommended action:** Impose a hard portfolio concentration cap — no single loan purpose should exceed 40% of total book value. Debt Consolidation is at 39.8% and approaching this threshold. Begin gradually diversifying origination mix toward Home Improvement and Education, which offer the best combination of default rate and volume balance.


## CEO Question 4 — Payment Shock Threshold: Where Is the DTI Hard Cap?

> *"At what DTI ratio do we see defaults spike? I need to set a hard cap for automated approvals."*

**Method:** Computed `lti_ratio = loan_amount / annual_income`. Bucketed `dti_ratio` into bands at 0.1, 0.2, 0.3, 0.4, 0.5, 0.6. Plotted as a bubble scatter chart in Power BI — x-axis: DTI bin (0.1–0.6), y-axis: default rate (0.05–0.12), bubble size: loan count. LTI ratio distribution also plotted by loan status using a KDE plot (notebook cell 22).

**DTI default rate by bucket — matching Power BI scatter points:**

| DTI Bin (midpoint) | Loans | Default Rate | Volume | Signal |
|---|---|---|---|---|
| 0.10–0.20 (0.15) | 4,098 | **5.98%** | $17.9M |  Lowest risk |
| 0.20–0.30 (0.25) | 15,369 | 6.58% | $84.6M |  Stable |
| 0.30–0.40 (0.35) | 26,356 | 6.46% | $184.1M |  Stable |
| 0.40–0.50 (0.45) | 29,782 | **7.85%** | $261.7M |  Rising |
| 0.50–0.60 (0.55) | 174,053 | **11.50%** | $3,028.6M |  Cliff |

**The cliff is visible in the Power BI scatter between x=0.4 and x=0.5** — the default rate makes a non-linear jump, rising steeply before plateauing above 0.5 at approximately 11.5%. From DTI 0.10 through 0.40, the default rate moves in a narrow, predictable range of 5.98%–6.58% — a manageable, priceable risk curve. The jump at 0.45 is structural, not noise.

**Borrower profile — defaulted vs. non-defaulted:**

| Metric | Defaulted Borrowers | Non-Defaulted Borrowers | Gap |
|---|---|---|---|
| Avg annual income | $74,463 | $78,841 | −$4,378 |
| Avg loan amount | $15,219 | $14,210 | +$1,009 |
| Avg DTI ratio | 0.545 | 0.519 | +0.026 |
| Avg LTI ratio | 0.253 | 0.228 | +0.025 |

Defaulted borrowers earn less, borrow more, and carry higher DTI and LTI ratios — a consistent profile of over-extension relative to income.

**The scale problem the scatter does not show:** 174,053 loans — **69.6% of the entire portfolio** — sit in the 0.50–0.60 DTI bucket. The hard cap recommendation is not a minor policy adjustment. It would technically disqualify the majority of the current loan profile. This context is critical for how the CEO frames the decision.

**Recommended action:** Set DTI 0.45 as the maximum threshold for automated loan approvals. Applications above this level must be reviewed manually and must demonstrate compensating factors — higher income stability, lower loan-to-value ratio, or strong employment tenure — to proceed to approval. For the existing book above 0.50, initiate a proactive borrower health programme including income re-verification and early restructuring offers for accounts showing payment stress signals.


## CEO Question 5 — Efficiency & Borrower Demographics: Who Is the Golden Goose?

> *"Which segment has the highest loan amounts with the lowest default rates? This will dictate where we spend our marketing budget next quarter."*

**Method:** Binned `age` into five groups (notebook cell 8). Cross-tabulated `employment_length` against `age_group`. Computed net expected yield per cell. Displayed as a matrix visual in Power BI — rows: employment length, columns: age group, values: net expected yield (%).

**Net yield matrix — exact values matching Power BI dashboard:**

| Employment Length | 21–30 | 31–40 | 41–50 | 51–60 | 61–75 | Row Total |
|---|---|---|---|---|---|---|
| < 1 yr | 6.24 | 6.14 | 5.95 | 5.92 | 4.43 | **6.06** |
| 1–3 yrs | 6.86 | **7.26** | 6.85 | 6.95 | 6.25 | **6.99** |
| 4–6 yrs | 6.92 | 7.14 | 6.64 | 6.93 | 6.44 | **6.91** |
| 7–10 yrs | 6.47 | 7.14 | 6.75 | 6.56 | 7.11 | **6.80** |
| 10+ yrs | 7.10 | 7.07 | 6.70 | 6.75 | 6.46 | **6.93** |
| **Col Total** | **6.79** | **7.05** | **6.65** | **6.71** | **6.35** | **6.83** |

**The Golden Goose:** Borrowers aged **31–40 with 1–3 years of employment** produce the highest net yield in the portfolio at **7.26%** — 43 basis points above the portfolio average of 6.83%. The 31–40 age column is consistently the strongest performing age cohort regardless of employment length, with a column total of 7.05%.

**The single most important row in the matrix:** The `< 1 yr` employment row is the only consistent underperformer across all five age groups — every cell falls below the portfolio average of 6.83%. The oldest borrowers with short tenure (61–75, < 1 yr) are the weakest segment in the book at **4.43%** — 240 basis points below average. This weakness is not age-driven; it is entirely driven by employment instability.

**What this means for marketing:** The 10+ years employment group (row total 6.93%) and the 1–3 years group (row total 6.99%) are the two strongest employment cohorts. Age 31–40 is the strongest age cohort across every employment length. The intersection of these two dimensions — **31–40, 1–3 years employment** — is the highest-value acquisition target in the portfolio.

**Recommended action:** Concentrate next quarter's marketing spend on the 28–42 age demographic with a minimum of 12 months of employment history. This segment produces the best yield, the most predictable repayment behaviour, and the highest average loan amounts. Introduce a minimum 12-month employment requirement as a soft gate in the automated approval scoring model — the under-1-year employment segment underperforms at every age and should carry a rate premium of at least 150bps to compensate.


## Summary Decision Matrix

| Finding | Risk Level | Immediate Action Required |
|---|---|---|
| Poor/Fair credit margin compression |  Medium | Raise floor rates by 150–200bps; set 8% minimum net yield gate |
| Vintage quality volatility — no structural leak |  Medium | Implement cohort tracking dashboard; watch H2 2024 loans |
| Debt Consolidation concentration at 39.8% |  High | Hard cap at 40% of book; diversify toward Education |
| DTI cliff — 69.6% of book above 0.50 |  High | Hard cap at 0.45 in approval engine; proactive borrower outreach |
| Golden Goose identified — age 31–40, 1–3yr employment |  Opportunity | Redirect Q2 marketing; introduce 12-month employment soft gate |


## What the Data Confirmed vs. What Was Expected

| Assumption Going In | What the Data Confirmed |
|---|---|
| Poor credit is the most profitable high-risk tier | True — but only at 5.39%, with no buffer for further deterioration |
| Exceptional credit is safe and generates strong yield | False — Exceptional yields only 1.82%, the lowest in the portfolio |
| Default rates vary significantly by loan purpose | False — all five purposes cluster within 0.38pp of each other (9.96%–10.34%) |
| Debt Consolidation is riskier per loan | False — the risk is concentration at 39.8% of book, not per-loan behaviour |
| Older borrowers with long tenure are the best customers | Partially true — employment stability matters more than age alone |
| The DTI cliff is a gradual slope | False — it is a non-linear jump at DTI 0.45, confirmed in both Python and Power BI |


## What I Would Do With More Time

- **LTI threshold modelling:** The LTI ratio was computed and visualised as a KDE distribution by loan status but not bucketed for a cliff analysis equivalent to the DTI work. The gap between defaulted LTI (0.253) and non-defaulted LTI (0.228) suggests a threshold exists around LTI 0.24 — identifying it precisely would add a second hard cap variable to the approval engine.
- **Charged Off cohort analysis:** 15,312 charged-off loans ($233.4M) are excluded from the default flag used throughout this analysis. A separate charge-off vintage analysis would determine whether charged-off loans originated from the same H2 2024 cohorts that produced the highest Defaulted rates, or whether they represent a different risk profile entirely.
- **Regional performance:** The dataset covers 28 states across 4 regions. A state-level default rate breakdown would identify geographic concentrations and determine whether the South's higher default rates are being adequately compensated by the interest rate premium already applied in the pricing model.
- **Full correlation matrix:** A numerical heatmap across `credit_score`, `dti_ratio`, `lti_ratio`, `interest_rate`, `annual_income`, and `is_default` would quantify which variables are the strongest predictors of default — a prerequisite for building a logistic regression or gradient boosting credit scoring model.
- **Machine learning model:** With 250,000 cleanly labelled records and well-engineered features, this dataset is ML-ready. A gradient boosting classifier (XGBoost or LightGBM) on `is_default` would produce feature importances that either validate or challenge the EDA findings above with statistical rigour.


## Tech Stack

Python · Pandas · NumPy · Matplotlib · Seaborn · Jupyter Notebook · Power BI