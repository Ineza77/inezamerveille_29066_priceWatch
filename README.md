# inezamerveille_29066_priceWatch
# PriceWatch Rwanda: Market Food Price Monitoring and Price-Spike Alert System

**Student:** INEZA RUTAYISIRE Merveille  
**Student ID:** 29066  
**Course:** INSY 8311, Database Development with PL/SQL  
**Instructor:** Eric Maniraguha  
**Institution:** Adventist University of Central Africa (AUCA), Faculty of Information Technology  
**Academic year:** 2026-2027 

---

## 1. Problem Statement

Food prices in Rwandan markets can change quickly, and households, traders and planners are often affected before anyone notices. Price records come from different reports and files, so there is no single place where prices by market, district and commodity can be compared over time. As a result, sudden rises in staples such as maize, beans and potatoes are found late, and nobody records who was alerted or what was done.

## 2. Proposed Solution

PriceWatch is an Oracle PL/SQL database that stores real market price records by district, market and commodity. PL/SQL code compares each new price with the commodity's recent average and opens an alert when the rise passes a set percentage. Triggers reject invalid prices and keep an audit log of every change, and a scheduled job scans for spikes automatically every night.

## 3. Objectives

**Main objective:** Build an Oracle PL/SQL database that monitors Rwandan market food prices and raises alerts when prices rise abnormally.

**Specific objectives:**
1. Store real WFP market price data in a normalized (3NF) Oracle database.
2. Calculate average prices and percentage changes per commodity, market and period using PL/SQL functions.
3. Detect price spikes automatically and record each alert from opened to closed.
4. Enforce data quality and security with triggers, a restriction rule and an audit log.
5. Run a nightly scheduled job that scans new prices and creates alerts without manual work.

## 4. Data Source

| Item | Detail |
|---|---|
| Dataset | Rwanda - Food Prices |
| Publisher | World Food Programme (WFP), via the Humanitarian Data Exchange (HDX) |
| Link | https://data.humdata.org/dataset/wfp-food-prices-for-rwanda |
| Content | Market-level food prices for Rwanda (market, commodity, date, price) |
| Size | THOUSANDS OF RECORDS: confirm the exact row count after download |
| Licence | CONFIRM ON THE HDX PAGE before submitting (must allow educational use) |
| Personal data | None: records are market-level prices, not individuals |

The raw CSV is stored in the `data/` folder.

## 5. Innovation

**Option:** Smart automation (the database acts on its own on a schedule)  
**Tool:** `DBMS_SCHEDULER`

A nightly scheduled job runs the spike-detection procedure on newly loaded prices, opens alerts for commodities whose price has risen sharply, and lists open alerts for managers. All business rules stay in PL/SQL. The job is demonstrated live in the final presentation and documented in the `innovation/` folder.

## 6. Planned Tables (3NF)

| Table | Purpose | Key columns |
|---|---|---|
| `DISTRICT` | Districts and provinces | district_id (PK), district_name, province |
| `MARKET` | Markets in each district | market_id (PK), market_name, district_id (FK) |
| `COMMODITY_CATEGORY` | Food groups (cereals, tubers, pulses...) | category_id (PK), category_name |
| `COMMODITY` | Food items and units | commodity_id (PK), commodity_name, unit, category_id (FK) |
| `PRICE_RECORD` | Each observed price | price_id (PK), market_id (FK), commodity_id (FK), price_date, price_type, price_rwf |
| `ALERT_RULE` | Spike thresholds per commodity | rule_id (PK), commodity_id (FK), threshold_pct, window_months |
| `PRICE_ALERT` | Spike alerts and their status | alert_id (PK), price_id (FK), rule_id (FK), avg_price, pct_change, status, created_at, closed_at |
| `AUDIT_LOG` | Record of every data change | log_id (PK), table_name, action, old_value, new_value, changed_by, changed_at |

**Relationships:** a district has many markets; a market has many price records; a commodity belongs to one category and has many price records; a price record may trigger one alert under one alert rule.

## 7. Planned PL/SQL Features

| Feature | Where it is used |
|---|---|
| **Function** `get_avg_price` | Returns a commodity's average price in a market over the last N months |
| **Function** `calc_pct_change` | Returns how far a new price is above or below that average |
| **Procedure** `record_price` | Validates and inserts a new price record |
| **Procedure** `detect_spikes` | Loops through new prices and opens alerts when the change passes the rule threshold |
| **Procedure** `close_alert` | Closes an alert with a resolution note |
| **Cursor** | Used inside `detect_spikes` to scan new prices commodity by commodity |
| **Package** `pkg_price_watch` | Groups the functions and procedures above |
| **Trigger** (validation) | Rejects zero or negative prices and duplicate market/commodity/date records |
| **Trigger** (audit) | Writes every insert, update and delete on price data to `AUDIT_LOG` |
| **Restriction rule** | Blocks changes to price data on weekends and public holidays |
| **Exceptions** | `e_invalid_price`, `e_duplicate_record`, `e_alert_already_closed` give clear error messages |
| **Scheduler job** | `nightly_spike_scan` runs `detect_spikes` every night |

## 8. Repository Structure

```
docs/
  research/       research_log.md
  proposal/       2-page proposal
  presentation/   Phase 0 slides
data/             raw dataset
innovation/       added in the final phase
```

Later phases add business process diagrams, ERD, data dictionary, SQL scripts and screenshots.

## 9. References

World Food Programme. (n.d.). *Rwanda - Food prices* [Data set]. Humanitarian Data Exchange. https://data.humdata.org/dataset/wfp-food-prices-for-rwanda

