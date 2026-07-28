# AML Transaction Monitoring Project — Build Log

This is a step-by-step record of how this project was built. Kept simple and honest so anyone reviewing it (recruiter, hiring manager) can see exactly what was done, with what tool, and why.

**Tools used in this project:** SQL (MySQL), Excel, Power BI
**Dataset:** SAML-D — Anti Money Laundering Transaction Data (Kaggle, public synthetic dataset built for transaction monitoring research). ~9.5 million transactions, labeled as laundering/not laundering with typology categories (structuring, layering, cross-border patterns, etc.)

---

## Step 1: Understood the business problem before touching data

Before writing any code, reviewed the three red-flag types this project focuses on, since a transaction monitoring analyst needs to know *why* a pattern is suspicious, not just detect it mechanically:

- **Structuring** — breaking a large transaction into smaller ones to stay under a reporting threshold.
- **Layering** — moving money through multiple accounts quickly to hide where it came from.
- **Unusual cross-border flows** — transactions to/from countries that don't match a customer's normal activity or business profile.

## Step 2: Set up the environment

- Downloaded the SAML-D dataset from Kaggle (public, synthetic, no real customer data — safe to use and publish).
- Inspected the raw CSV file first (checked the column headers and a few sample rows) before building any table, to make sure the database structure would match the real data exactly.
- Created a new MySQL database (`aml_project`) to hold the transaction data.
- Chose `LOAD DATA INFILE` (a MySQL command for loading large files directly) instead of the usual point-and-click import tool, because the dataset has close to 9.5 million rows — this method loads it in under a minute instead of hours.

## Step 3: Loaded 9.5 million transactions into MySQL

- Inspected the raw CSV file in Notepad to confirm exact column names before building the table — this avoids import errors on large files.
- Created a table in MySQL (`transactions`) with columns matching the dataset exactly: Time, Date, Sender_account, Receiver_account, Amount, Payment_currency, Received_currency, Sender_bank_location, Receiver_bank_location, Payment_type, Is_laundering, Laundering_type.
- Used `LOAD DATA LOCAL INFILE` instead of the GUI import wizard — the dataset has 9,504,852 rows and the GUI cannot handle files this large reliably. This command loaded the full dataset in under 3 minutes.
- Confirmed successful load with `SELECT COUNT(*)` — result: **9,504,852 rows**, matching the dataset documentation exactly.

## Step 4: Explored the data before writing any detection queries

- Checked all unique values in key columns — laundering types, payment types, countries, and flagged vs normal transaction split
- Found 28 laundering typologies in the dataset including Structuring, Smurfing, Layering, Cycle, Behavioural Change, and Cross-border patterns
- Confirmed 2000 flagged transactions and 3000 normal transactions in our working sample
- This exploration step mirrors how a real AML analyst reviews data before building detection rules — understanding what is there before querying blindly

## Step 5: Built 5 SQL Detection Rules (Transaction Monitoring Rule Engine)

Each rule targets a specific AML red flag typology:

- **Rule 1 — Structuring:** Finds accounts sending 3 or more transactions below 10,000 on the same day with a total above 20,000. Detects deliberate threshold avoidance.
- **Rule 2 — Smurfing:** Finds receiver accounts collecting money from 3 or more different senders on the same day, each under 10,000. Detects coordinated structuring across multiple accounts.
- **Rule 3 — Layering:** Finds accounts that both sent and received large amounts on the same day with very little difference between the two — pass-through accounts used to hide the money trail.
- **Rule 4 — Cross Border Risk:** Flags transactions involving high risk countries (Nigeria, Pakistan, Morocco, Albania) and scores them CRITICAL, HIGH, or MEDIUM based on amount and destination.
- **Rule 5 — Behavioural Change:** Uses window functions (LAG) to compare each account's daily total against the previous day — detects sudden unexplained spikes in transaction activity.

All 5 rules were saved as Views in MySQL and combined into a single Master Alerts view using UNION ALL — producing 1000 alerts ready for analysis.

SQL concepts used: GROUP BY, HAVING, WHERE, CASE WHEN, Subqueries, CTEs, INNER JOIN, Self JOIN, Window Functions (LAG, ROW_NUMBER, SUM OVER), Aggregate Functions (COUNT, SUM, AVG, MAX, MIN), ABS()

## Step 6: Exported Results to Google Sheets for Analysis

- Exported all 6 result sets from MySQL as CSV files — one per detection rule plus the master alerts combined view
- Imported all 6 files into Google Sheets as separate sheets in one workbook (AML_Analysis)

## Step 7: Built Excel Analysis on Master Alerts

**Summary Table:**
- Used COUNTIF to count total alerts per alert type
- Used COUNTIFS to count HIGH and CRITICAL alerts per alert type (two conditions at once)
- Used SUMIF to calculate total flagged amount per alert type
- Result: 1000 total alerts, 70 CRITICAL, 44 HIGH, total flagged amount of 16,169,750.84

**Key Insight Found:**
- 7.3 million (45% of total flagged amount) sits in LOW risk Cross Border alerts
- This indicates the cross border detection rule is generating too many low priority alerts — a false positive problem
- In real AML work this would trigger a threshold calibration review — exactly what the JD asks for

**Conditional Formatting:**
- Applied colour coding to Critical and High Risk count columns
- Red for high counts, yellow for medium, green for low — mirrors real monitoring dashboard priority display

**Pivot Tables:**
- Pivot 1 — Alert count broken down by alert type and risk level
- Pivot 2 — Total flagged amount broken down by alert type and risk level
- Pivot 3 — Flagged amount trends over time by date and alert type

**Charts:**
- Line chart showing flagged transaction amounts over time by alert type
- Bar chart showing total alerts by type

## Step 8: (in progress)

*Next: Connect data to Power BI and build interactive dashboard with slicers, KPI visuals, drill-through, and risk level filters.*

---

### Notes for later (README use)
- Every step above was chosen to reflect how a real transaction monitoring analyst would approach this: understand the typology first, then set up data properly, then investigate — not jump straight to querying blind.
- Dataset is fully synthetic — no real personal or customer data is used or exposed anywhere in this project.

## Step 8: Built Interactive Power BI Dashboard

Connected all 3 CSV files (master_alerts, rule1_structuring, rule4_crossborder) to Power BI Desktop and built a fully interactive AML monitoring dashboard.

**KPI Cards (3):**
- Total Alerts — 1,000
- Total Flagged Amount — $16.1M
- Critical Alerts — 70 (filtered using visual level filter)

**Charts (4):**
- Alerts by Type — horizontal bar chart showing alert volume per detection rule
- Flagged Amount by Risk Level — donut chart showing money distribution across CRITICAL, HIGH, MEDIUM, LOW
- Flagged Amount Over Time — line chart showing daily suspicious activity trends by alert type
- Flagged Amount by Destination Country — column chart using rule4_crossborder table showing which countries received the most flagged money and at what risk level

**Slicers (3) — makes entire dashboard interactive:**
- Filter by Risk Level — click any risk level to filter all visuals at once
- Filter by Alert Type — filter by structuring, smurfing, layering, cross border
- Filter by Date — filter by specific time period

**Key insight visible in dashboard:**
The donut chart and country chart together show that LOW risk Cross Border alerts account for 45% of total flagged amount — a clear false positive problem visible at a glance. This is the kind of insight a compliance manager would use to recalibrate detection thresholds.

## Project Complete

Two deliverables built:
1. AML Transaction Monitoring Rule Engine — SQL detection rules for structuring, smurfing, layering, cross border risk, and behavioural change across 9.5 million transactions
2. AML Analysis Dashboard — Google Sheets analysis + Power BI interactive dashboard
