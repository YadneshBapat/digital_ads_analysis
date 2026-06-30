# 📊 Digital Ad Platform Performance Analysis

> A programmatic advertising analytics project simulating the daily data operations of a digital ad trading team - built to demonstrate data quality assurance, performance reporting, KPI dashboard development, and yield analysis using real social media ad event data.

---

## 📌 Project Overview

This project analyses **400,000 ad events** across Facebook and Instagram campaigns sourced from a real multi-table social media advertising dataset. The work mirrors the core responsibilities of a Junior Data Analyst in a programmatic trading team: joining raw vendor data, validating data integrity, engineering missing metrics, building performance trackers, and surfacing actionable insights through a structured Excel dashboard.

The dataset was sourced from Kaggle: [Social Media Advertisement Performance](https://www.kaggle.com/datasets/alperenmyung/social-media-advertisement-performance)

---

## 🎯 Key Objectives

- Join and clean 4 raw vendor datasets into a single analysis-ready master table
- Conduct structured data quality assurance across all fields
- Engineer programmatic metrics absent from raw data (CPM, revenue, CTR, viewability)
- Identify and quantify data anomalies including out-of-window ad events
- Build daily and weekly performance trackers for trend monitoring
- Develop an interactive KPI dashboard consolidating platform, audience, geographic, and format-level performance
- Produce a structured analyst insights report with methodology documentation

---

## 🗂️ Dataset

| File | Description | Rows |
|---|---|---|
| `adevent.csv` | Ad events: impressions, clicks, likes, shares, purchases | 400,000 events |
| `ads.csv` | Ad metadata: platform, format, targeting parameters | 200 ads |
| `campaigns.csv` | Campaign details: dates, budget, duration | 50 campaigns |
| `users.csv` | User demographics: age, gender, country, interests | 1000 users |

**Source:** [Kaggle - Social Media Advertisement Performance](https://www.kaggle.com/datasets/alperenmyung/social-media-advertisement-performance)

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Microsoft Excel | Data cleaning, joining, pivot tables, dashboard, trackers |
| Excel Formulas | XLOOKUP, COUNTIFS, SUMIFS, AVERAGEIF, SUMPRODUCT, INDEX/MATCH |
| Pivot Tables | Campaign, platform, demographic, and time-based segmentation |
| Conditional Formatting | Anomaly flagging, heatmaps, QA status indicators |

---

## 📁 Workbook Structure

The Excel workbook (`Nine_AdPerformance_Portfolio.xlsx`) contains 9 tabs:

```
├── raw_events          # Raw event data - untouched source
├── raw_ads             # Raw ad metadata - untouched source
├── raw_campaigns       # Raw campaign data - untouched source
├── raw_users           # Raw user data - untouched source
├── master_table        # Joined dataset with 28 columns including engineered metrics
├── data_qa             # 5-check data quality assurance report
├── pivot_analysis      # 4 pivot tables: platform, campaign, day, demographics
├── kpi_dashboard       # Interactive dashboard: 12 KPI cards, alert banner, 7 charts
└── insights            # Analyst commentary, findings, and methodology notes
```

---

## ⚙️ Methodology

### 1. Data Joining
All 4 source files were joined into a single `master_table` using **XLOOKUP** (and INDEX/MATCH for older Excel versions), with `raw_events` as the base table enriched by ad, campaign, and user attributes - simulating a multi-platform vendor data reconciliation workflow.

### 2. Data Quality Assurance (5-Check Framework)

| Check | Finding |
|---|---|
| Missing / unmatched values | 0 unmatched records across all 7 key fields ✓ |
| Duplicate event IDs | Validated via helper column deduplication |
| Events outside campaign date range | **211,234 events (52.8%) outside active campaign window** ⚠️ |
| Platform volume sanity check | Facebook: 254,096 \| Instagram: 145,904 \| No unmatched ✓ |
| Revenue sanity check | $1.46M estimated \| CPM range $494–$36,340 ✓ |

> **Critical Finding:** 63,197 events occurred before campaign start dates and 148,037 after campaign end dates - flagged as a vendor tracking anomaly requiring investigation before revenue figures can be trusted for yield decisions.

### 3. Engineered Metrics
The following metrics were absent from the raw dataset and derived using industry-standard programmatic assumptions:

| Metric | Formula Logic | Assumption |
|---|---|---|
| `revenue_estimate` | Budget × 72% ÷ campaign impressions | 72% publisher revenue share after DSP/SSP fees |
| `cpm_estimate` | Budget ÷ campaign impressions × 1,000 | Standard CPM calculation |
| `delivery_rate` | Campaign event share of total events | Proxy for fill rate (ad request data unavailable) |
| `viewability_flag` | High/Medium/Low by time of day | IAS 2024 viewability benchmarks |
| `is_engagement` | 1 if Like or Share, else 0 | Active intent proxy |
| `ctr_proxy` | Engagements ÷ impressions per campaign | Industry-standard engagement rate definition |
| `timing_anomaly` | BEFORE_START / AFTER_END / OK | Compares event_date to campaign start/end dates |

All assumptions are documented in the Insights tab methodology section.

### 4. Performance Trackers
Daily and weekly trackers were built in a dedicated tab using **COUNTIFS** and **SUMIFS** referencing `week_starting` and `event_date` columns, tracking impressions, engagements, engagement rate, estimated revenue, and average CPM over time - with conditional formatting (green-yellow-red colour scale) for at-a-glance trend scanning.

---

## 📈 Key Findings

### Platform Performance
- Instagram delivered a **4.19% engagement rate** vs Facebook's **4.07%** despite generating **57% fewer impressions** - suggesting stronger creative resonance relative to volume
- Facebook dominates raw volume: **254,096 events (63.5%)** vs Instagram's **145,904 (36.5%)**

### Campaign Performance
- **49 campaigns** analysed with estimated revenue ranging from **$4,862 (Campaign_42_Summer)** to **$60,466 (Campaign_20_Winter)**
- **73.6x CPM variance** between highest ($36,340 - Campaign_35_Launch) and lowest ($494 - Campaign_42_Summer) performing campaigns - significant yield optimisation opportunity
- Overall average CPM: **$5,961**

### Audience Insights
- **25–34 age group** represents the largest audience segment at **41.4% (165,632 events)**
- Male users account for **55.4%** of all events; Female **34.4%**; Other **10.2%**
- 16–17 age bracket represents **8.8%** of events - worth flagging for brand safety review on targeted campaigns

### Timing Anomalies
- **52.8% of all events (211,234)** recorded outside their campaign's active window
- 63,197 events before campaign start; 148,037 after campaign end
- Revenue attributed to anomalous events flagged for re-validation before use in yield reporting

### Day of Week
- Engagement is **consistent across all 7 days** (8,550–8,653 engagement events per day) - no significant weekend uplift, suggesting always-on scheduling is appropriate for this inventory

---

## 📊 Dashboard Preview

The `kpi_dashboard` tab contains:

**12 KPI Cards across 3 rows:**
- Row 1: Total Events · Total Impressions · Engagement Rate · Estimated Revenue
- Row 2: Avg CPM · CPM Variance · Top Platform · Largest Audience Segment
- Row 3: Top Country · Top Ad Format · Revenue from Anomalous Events · Peak Time of Day

**1 Critical Alert Banner** (full-width red) surfacing the timing anomaly finding

**7 Charts:**
1. Event Volume by Platform & Type (Clustered Bar)
2. Weekly Engagement Pattern by Day (Line)
3. Top 10 Campaigns by Estimated Revenue (Horizontal Bar)
4. Audience Distribution by Age & Gender (Stacked Column)
5. Timing Anomalies by Week - Trend Check (Line)
6. Event Volume by Country - Top 8 (Bar)
7. Engagement Rate by Ad Format (Column)

---

## 📋 Assumptions & Limitations

```
- Revenue estimated from campaign budget allocation across impression volume
- CPM derived from budget ÷ impressions; actual billed CPM will differ by DSP/SSP
- Fill rate not calculable without ad request data; delivery rate used as proxy
- Viewability estimated from time-of-day benchmarks (IAS 2024 industry standards)
- Click events not present in dataset; Likes and Shares treated as engagement proxies
- Timing anomalies may reflect timezone discrepancies or vendor pixel firing delays
  rather than confirmed tracking errors - vendor cross-check recommended
```

---

## 💼 Relevance to Programmatic Trading Role

This project was designed to reflect the core responsibilities of a Junior Data Analyst in a digital ad trading environment:

| Requirement | Project Demonstration |
|---|---|
| Data quality assurance using Excel and SQL | 5-check QA framework with structured anomaly flagging |
| Daily and weekly performance trackers | Dedicated tracker tab with time-series KPIs |
| Pivot tables to organise raw vendor datasets | 4 pivot analyses across platform, campaign, audience, time |
| Flag data anomalies and missing fields | 211,234 out-of-window events identified and quantified |
| Inventory optimisation and yield analysis | CPM variance analysis across 49 campaigns |
| AI-assisted tools for workflow efficiency | Excel formula automation reducing manual processing |
| Translate data into actionable insights | Structured insights memo with methodology documentation |

---

## 👤 Author

**Yadnesh Bapat**
Data Analyst | Sydney, NSW

---

## 📄 License

This project uses a publicly available dataset from Kaggle for portfolio and educational purposes only. No proprietary or confidential data is included.
