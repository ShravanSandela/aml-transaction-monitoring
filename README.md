# AML Transaction Monitoring Project

## Overview
This project simulates a real-world Anti-Money Laundering (AML) transaction monitoring system built using SQL, Excel, and Power BI. It detects financial crime red flag patterns across 9.5 million synthetic banking transactions using rule-based detection logic — the same approach used by financial institutions and fintechs.

**Dataset:** SAML-D Synthetic AML Transaction Dataset (Kaggle) — 9,504,852 transactions labeled with laundering typologies including structuring, smurfing, layering, cycle transactions, and behavioural change patterns.

---

## Business Problem
Financial institutions are required to monitor customer transactions for suspicious activity and report unusual behaviour to regulators. Manual review of millions of transactions is impossible — automated detection rules are needed to flag high risk activity for human investigation.

This project builds 5 automated detection rules that identify the most common financial crime typologies, score every alert by risk level, and present findings in an interactive dashboard for compliance teams.

---

## Tools Used
- **MySQL** — data loading, exploration, detection rule logic
- **Google Sheets** — alert analysis, COUNTIF/SUMIF formulas, pivot tables, charts
- **Power BI** — interactive compliance dashboard with slicers and KPI visuals

---

## Detection Rules Built

### Rule 1 — Structuring
Finds accounts sending 3 or more transactions below 10,000 on the same day with a combined total above 20,000. Detects deliberate threshold avoidance — one of the most common money laundering techniques.

### Rule 2 — Smurfing
Finds receiver accounts collecting money from 3 or more different senders on the same day, each transaction under 10,000. Detects coordinated structuring across multiple accounts working together.

### Rule 3 — Layering (Pass-Through Detection)
Finds accounts that both sent and received large amounts on the same day with very little difference between the two totals. These are pass-through accounts used to obscure the money trail — a key layering typology.

### Rule 4 — Cross Border Risk
Flags transactions sent to high risk jurisdictions (Nigeria, Pakistan, Morocco, Albania) and scores them CRITICAL, HIGH, or MEDIUM based on transaction amount and destination country risk.

### Rule 5 — Behavioural Change Detection
Uses SQL window functions (LAG) to compare each account's daily transaction total against the previous day. Flags accounts showing sudden unexplained spikes in activity — a key indicator of account takeover or new criminal use.

---

## Key Findings

| Alert Type | Total Alerts | Critical | High | Total Amount Flagged |
|---|---|---|---|---|
| Cross Border Risk | 871 | 5 | 25 | $9,075,528 |
| Structuring | 71 | 27 | 8 | $3,797,200 |
| Smurfing | 36 | 27 | 2 | $2,971,654 |
| Layering | 22 | 11 | 9 | $325,367 |
| **Total** | **1,000** | **70** | **44** | **$16,169,750** |

**Key Insight — False Positive Problem Identified:**
45% of total flagged amount ($7.3 million) sits in LOW risk Cross Border alerts. This indicates the cross border detection threshold is too broad and needs recalibration — a real finding that would trigger a rule tuning review in a live compliance environment.

---

## SQL Concepts Used
GROUP BY | HAVING | WHERE | CASE WHEN | Subqueries | CTEs | INNER JOIN | Self JOIN | Window Functions (LAG, ROW_NUMBER, SUM OVER) | Aggregate Functions (COUNT, SUM, AVG, MAX, MIN) | UNION ALL | Views | LOAD DATA INFILE

## Excel/Sheets Concepts Used
COUNTIF | COUNTIFS | SUMIF | Pivot Tables | Conditional Formatting | Charts

## Power BI Concepts Used
KPI Cards | Bar Chart | Donut Chart | Line Chart | Column Chart | Slicers | Visual Level Filters | Data Modelling

---

## Project Structure
aml-transaction-monitoring/
├── sql/
│ └── aml_detection_rules.sql
├── data/
│ ├── master_alerts.csv
│ ├── rule1_structuring.csv
│ ├── rule2_smurfing.csv
│ ├── rule3_layering.csv
│ ├── rule4_crossborder.csv
│ └── rule5_behavioural_change.csv
└── README.md

---

## About
Built as part of a portfolio project targeting Financial Crime Analyst and AML Investigation roles. All data is fully synthetic — no real customer or transaction data is used anywhere in this project.
