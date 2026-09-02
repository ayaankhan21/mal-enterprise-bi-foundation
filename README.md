# mal-enterprise-bi-foundation
# Mal Digital Bank — Enterprise BI Foundation

**Live dashboard:** https://app.powerbi.com/view?r=eyJrIjoiZDlmNTVlMGMtNmI3OC00OGNlLWIxYmUtYTM2NDQ2ODczZTQxIiwidCI6IjA1MjFmOTBhLWY0ZWMtNDFjMC1hZGM3LTU3ZDQ0OWRhYThmZiJ9&pageName=7958c1d318a592bfa538

## Overview

This is a working prototype of an enterprise BI foundation for Mal, a digital bank launching credit, payments, and savings products simultaneously. It addresses the core problem: three domain teams (Finance, Product, Credit) each measure the same KPIs differently, with no shared KPI dictionary or reporting layer.

## Stack

- **Power BI Desktop / Power BI Service** — report authoring and public hosting (Publish to Web)
- **Excel workbooks** — synthetic data source (5 operational tables + 1 KPI dictionary table)
- **Python (pandas)** — synthetic data generation script (`generate_data.py`)

Power BI was chosen for its native DAX modelling capabilities and because it's the tool the author has the deepest working fluency in, allowing more time to be spent on data modelling and governance design rather than tool-learning under a tight deadline.

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
