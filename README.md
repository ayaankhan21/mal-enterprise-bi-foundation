# mal-enterprise-bi-foundation
# Mal Digital Bank — Enterprise BI Foundation

**Live dashboard:** https://app.powerbi.com/view?r=eyJrIjoiZDlmNTVlMGMtNmI3OC00OGNlLWIxYmUtYTM2NDQ2ODczZTQxIiwidCI6IjA1MjFmOTBhLWY0ZWMtNDFjMC1hZGM3LTU3ZDQ0OWRhYThmZiJ9&pageName=7958c1d318a592bfa538

## Overview

This is a working prototype of an enterprise BI foundation for Mal, a digital bank launching credit, payments, and savings products simultaneously. It addresses the core problem: three domain teams (Finance, Product, Credit) each measure the same KPIs differently, with no shared KPI dictionary or reporting layer.
Mal Enterprise BI Foundation
Background About the Company

Mal is a digital bank launching three products simultaneously credit payments and savings The bank operates as a startup with no existing BI platform and no shared definition of core business metrics across teams

From the perspective of the sole BI hire at Mal the primary responsibility is to build a working enterprise reporting foundation from scratch while resolving conflicting metric definitions across Finance Product and Credit before the multi product launch creates lasting reporting chaos

Stakeholder Pain Points

Before this project stakeholders faced several challenges that limited effective decision making

Finance Product and Credit each defined active customers differently leading to three different numbers reported for the same metric
Revenue was tracked inconsistently gross booked revenue by Finance net of credit losses by Credit and per user by Product with no single certified figure for leadership
Credit approval rate was calculated differently depending on which team reported it since each team used a different funnel stage as the denominator
No KPI dictionary existed so new hires and domain analysts had no single source of truth for metric definitions
No BI platform existed and data engineering was fully occupied with pipeline delivery leaving no support for reporting
Underperforming risk segments in the credit portfolio had no dedicated dashboard to surface delinquency trends early
Domain analysts had no clear process for requesting data access or building their own reports without creating further metric conflicts

## Stack

- **Power BI Desktop / Power BI Service** — report authoring and public hosting (Publish to Web)
- **Excel workbooks** — synthetic data source (5 operational tables + 1 KPI dictionary table)
- **Python (pandas)** — synthetic data generation script (`generate_data.py`)

Power BI was chosen for its native DAX modelling capabilities and because it's the tool the author has the deepest working fluency in, allowing more time to be spent on data modelling and governance design rather than tool-learning under a tight deadline.

DAX Used to create calculated measures including total revenue fee income ratio approval rate and portfolio delinquency rate

Data Modeling Star schema implemented with customer and accounts as dimension tables a DAX generated Date table and three fact tables transactions credit applications and credit performance

Python Used with pandas to generate a realistic synthetic banking dataset including a weighted risk trajectory per credit account so delinquency trends move over time rather than appearing as random noise

File Formats pbix file for Power BI development xlsx files for the synthetic source data and the KPI dictionary py file for the data generation script png image for the data model diagram

## Data model

A star schema with 2 dimension tables and 3 fact tables, plus a Date dimension:

- **Dimensions:** `customer` (1,500 rows), `accounts` (2,580 rows)
- **Facts:** `credit_applications` (950 rows), `transactions` (11,298 rows), `credit_performance` (4,776 rows — monthly delinquency snapshots)
- **Date table:** DAX-generated calendar spine, connected to both `transactions` and `credit_performance` for consistent time-based filtering across the report

All data is **synthetic**, generated with realistic distributions (e.g. funnel drop-off across credit application stages, a weighted "current vs. delinquent" risk trajectory per account rather than random noise) so KPIs reflect plausible bank behavior rather than flat/uniform test data. No real customer or bank data was used.

## Metric conflict resolution

The case study's central example — "active customers" — is defined three different ways depending on which domain is asking:

| Domain | Definition | Status |
|---|---|---|
| Product | Logged in / transacted in the last 30 days | **Certified** (enterprise default) |
| Finance | Holds an open account with non-zero balance at month-end | Exploratory |
| Credit | Holds an open credit line in good standing | Exploratory |

This isn't a data quality issue — each definition is correct for the question that domain is actually accountable for. The same three-way conflict pattern repeats for **Revenue** (gross booked vs. net-of-losses vs. per-user) and **Approval Rate** (underwriting-stage denominator vs. full-funnel vs. disbursement-gated).

The **Metric Conflict Resolution** page in the dashboard shows all three definitions for each of these metrics side by side, with the certified version clearly marked, and a one-line rationale for why that definition was chosen as the enterprise default. The other two remain visible and usable for domain-specific analysis — they are not deleted, just not used in enterprise-wide reporting.

Key business metrics tracked include

Active customers certified and domain specific variants
Total revenue and fee income ratio
Credit approval rate and application to disbursement conversion
Portfolio delinquency rate and non performing loan ratio
Product cross sell rate and customer signups

## Dashboard pages

1. **Executive Dashboard** — 8 certified KPIs spanning Product, Finance, and Credit, plus revenue and active-customer trend lines
2. **Credit & Lending Domain Dashboard** — approval funnel, portfolio quality, delinquency and NPL risk indicators
3. **Metric Conflict Resolution** — the three-way definition conflicts described above
4. **KPI Dictionary** — 18 metrics with name, domain owner, business definition, formula, data source, refresh cadence, and certified/exploratory status
5. **Self-Service Landing Page** — dataset inventory, access request process, and SLA for domain analysts

## Files in this repo

- `generate_data.py` — synthetic data generation script (Python/pandas)
- `mal_synthetic_data.xlsx` — the 5 operational data tables
- `mal_kpi_dictionary_clean.xlsx` — the 18-metric KPI dictionary

## Note on the Date table grain

`transactions.date` is daily grain; `credit_performance.month` is monthly grain (one snapshot per account per month). Both relate to the same Date dimension table, but this is a deliberate mixed-grain design — consistent with how real banks typically log daily transactions but monthly risk snapshots — not an oversight.

## Executive Summary
Overview of Findings

From a stakeholder perspective the three most important takeaways from this project are

The same metric produced three different legitimate numbers depending on which domain reported it not because of data errors but because each domain optimizes for a different business decision
A single certified definition per conflicted metric restores enterprise wide trust in reporting while preserving each domain's own exploratory view for their internal use
Credit portfolio risk becomes visible over time rather than as a static snapshot once delinquency is tracked monthly per account rather than only at the point in time of reporting

## Insights Deep Dive
Category 1 Metric Governance and Conflict Resolution

Insight 1 Active customers reported between a low of roughly 15 to 20 percent of the base under Credit's definition and the full engagement based figure under Product's definition depending on which domain's view was used

Insight 2 Revenue diverges meaningfully between Finance's gross booked figure and Credit's net of expected losses figure once delinquency in the credit book is factored in

Insight 3 Credit approval rate varies depending on whether incomplete applications are included in the denominator with Product's full funnel view producing a materially lower rate than Credit's underwriting stage only view
Category 2 Credit and Lending Portfolio Performance

Insight 1 The credit approval funnel shows meaningful drop off between submitted and scored applications reflecting incomplete or abandoned applications rather than underwriting rejection

Insight 2 Portfolio delinquency skews heavily toward current status in any single month consistent with a healthy launching credit book with a realistic long tail into later delinquency buckets

Insight 3 The non performing loan ratio at the 90 plus days past due threshold remains within a plausible range for a newly launched credit portfolio

Category 3 Revenue and Financial Health

Insight 1 Fee income makes up a meaningful share of total certified revenue alongside interest income across the credit and payments products

Insight 2 Revenue shows real month over month movement across the six month synthetic history rather than a flat trend supporting realistic trend analysis on the executive dashboard

Insight 3 Average revenue per user is tracked against total open accounts to support unit economics discussions separate from the top line revenue figure

Category 4 Product Engagement and Growth

Insight 1 Certified active customers based on the thirty day engagement definition covers the full customer base regardless of which specific product a customer holds

Insight 2 Product cross sell rate highlights the share of customers holding two or more products indicating early cross product adoption during the launch period

Insight 3 New customer signups tracked over the six month period provide a growth trend independent of the active customer engagement metric

## Recommendations
Based on the analysis the following actions are recommended for Mal's Head of Data and domain leadership teams

Ratify the certified definitions for active customers revenue and credit approval rate at the enterprise governance sync before any further domain dashboards are built on top of them
Prioritize the Credit and Lending domain dashboard first given the highest regulatory visibility and the cleanest underlying funnel data structure
Flag the decision date dependency to data engineering early so the time to decision KPI can be fully certified rather than remaining a documented gap

## Assumptions and Caveats
All data used in this project is fully synthetic and was generated specifically for this case study no real Mal customer or transaction data exists or was used
Synthetic data was generated with realistic distributions and a weighted risk trajectory per account rather than uniform randomness so that trends and delinquency patterns resemble a real banking book
