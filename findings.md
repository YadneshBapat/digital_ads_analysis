# 📋 Detailed Findings, Anomalies & Recommendations
## Digital Ad Platform Performance Analysis

**Prepared by:** Yadnesh Bapat  
**Dataset:** Social Media Advertisement Performance (400,000 events, 50 campaigns)  
**Platforms Analysed:** Facebook, Instagram  
**Period Covered:** April 2025 - September 2025  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Data Quality & Anomaly Report](#2-data-quality--anomaly-report)
3. [Platform Performance Analysis](#3-platform-performance-analysis)
4. [Campaign Performance Analysis](#4-campaign-performance-analysis)
5. [Audience & Demographic Analysis](#5-audience--demographic-analysis)
6. [Ad Format & Inventory Analysis](#6-ad-format--inventory-analysis)
7. [Temporal & Day-of-Week Analysis](#7-temporal--day-of-week-analysis)
8. [Revenue & Yield Analysis](#8-revenue--yield-analysis)
9. [Recommendations](#9-recommendations)
10. [Future Analytical Actions](#10-future-analytical-actions)
11. [Assumptions & Methodology](#11-assumptions--methodology)

---

## 1. Executive Summary

This analysis examined 400,000 ad events across 50 campaigns running on Facebook and Instagram, sourced from four separate vendor data feeds. The data was joined, cleaned, validated through a structured five-check QA framework, and enriched with seven engineered programmatic metrics before analysis.

**Three headline findings drive this report:**

**Finding 1 - Critical Data Integrity Issue:** 211,234 events (52.8% of all records) were recorded outside their respective campaign's active date window - either before the campaign launched or after it ended. This is the most significant finding in the dataset and has direct implications for how estimated revenue figures should be interpreted. No revenue or performance metric derived from this dataset should be treated as final until this anomaly is investigated and resolved with the vendor.

**Finding 2 - CPM Variance Signals Yield Inefficiency:** A 73.6x difference in CPM between the best and worst performing campaigns ($36,340 vs $494) indicates substantial yield inefficiency across the portfolio. Campaigns with high budgets but low impression volumes are artificially inflating CPM estimates, while high-volume, low-budget campaigns are significantly undermonetised.

**Finding 3 - Instagram Punches Above Its Weight:** Despite generating 57% fewer impressions than Facebook, Instagram delivered a higher engagement rate (4.19% vs 4.07%). This disproportionate engagement efficiency suggests Instagram inventory is underallocated relative to its audience responsiveness.

---

## 2. Data Quality & Anomaly Report

### 2.1 QA Check 1 - Missing and Unmatched Values

All seven critical fields were validated against the master table after joining:

| Field | Unmatched Records | Status |
|---|---|---|
| event_id | 0 | ✅ Clean |
| ad_id | 0 | ✅ Clean |
| user_id | 0 | ✅ Clean |
| ad_platform | 0 | ✅ Clean |
| campaign_name | 0 | ✅ Clean |
| user_gender | 0 | ✅ Clean |
| country | 0 | ✅ Clean |

**Interpretation:** The four source files join cleanly with zero orphaned records. Every event maps to a valid ad, campaign, and user - indicating well-structured vendor data exports with consistent ID formatting across systems. This is a positive baseline finding but does not rule out logical or temporal inconsistencies (see Section 2.3).

---

### 2.2 QA Check 2 - Duplicate Event IDs

A helper column (`is_duplicate`) was added to the master table using a cumulative COUNTIF approach to identify first-occurrence vs repeated event IDs across 400,000 rows without performance-intensive array formulas.

**Outcome:** No confirmed duplicate event IDs were found after formula correction. The initial formula error (SUMPRODUCT across full columns) produced a misleading result of 400,000 duplicates - this was a formula scoping error, not a data error, and was corrected by limiting the range to the actual data bounds (A2:A400001).

**Recommendation:** Always scope range-based formulas explicitly (e.g. `A2:A400001` rather than `A:A`) when working with large datasets. Unbounded column references on 400K+ rows produce both performance issues and logical errors.

---

### 2.3 QA Check 3 - Timing Anomalies (CRITICAL FINDING)

This is the most significant data quality finding in the entire analysis.

| Category | Event Count | % of Total |
|---|---|---|
| Events before campaign start | 63,197 | 15.8% |
| Events after campaign end | 148,037 | 37.0% |
| Events within campaign window | 188,766 | 47.2% |
| **Total** | **400,000** | **100%** |

**What this means:** Only 47.2% of all recorded events occurred during their campaign's active period. The remaining 52.8% were logged either before a campaign launched or after it officially ended.

**Likely Root Causes:**

*For BEFORE_START events (63,197):*
- Pixel pre-firing: tracking pixels placed on publisher pages may begin firing before campaign activation in the ad server
- Timezone misalignment: if the vendor's event logging system uses UTC while campaign dates are recorded in local time (e.g. AEST), a campaign starting at midnight AEST would appear to start 10-11 hours later in UTC logs, creating a false "before start" window
- Test or preview impressions: some DSPs serve a small number of test impressions before a campaign officially goes live to verify creative delivery

*For AFTER_END events (148,037):*
- Cookie/pixel persistence: tracking pixels placed during an active campaign can continue firing after the campaign ends if they are embedded in cached pages or user browsers
- Attribution windows: some platforms log engagement events (likes, shares) with a delay of hours or days after the interaction - if the underlying impression occurred during the campaign, the engagement event may be logged after the end date
- DSP flight date misalignment: the campaign end date recorded in the vendor's export may not match the actual deactivation date in the DSP, particularly if campaigns were extended or paused mid-flight and the metadata was not updated
- Bulk event log exports: some vendors export events in batches, and batch processing delays can cause events timestamped after campaign end to appear in the same export

**Why this matters for trading decisions:**
Revenue estimates derived from out-of-window events cannot be reliably attributed to the campaign they are tagged against. If a campaign's "estimated revenue" includes 37% of events that occurred after it ended - potentially from a different active campaign or from residual pixel fires - then CPM and yield figures are materially unreliable for billing reconciliation or future budget planning.

**Immediate action required:** Flag all AFTER_END and BEFORE_START records with the vendor and request a corrected export with verified event timestamps cross-referenced against ad server logs (not just pixel logs). Until this is resolved, revenue figures should be reported with a data quality caveat.

---

### 2.4 QA Check 4 - Platform Volume Sanity Check

| Platform | Event Count | % of Total |
|---|---|---|
| Facebook | 254,096 | 63.5% |
| Instagram | 145,904 | 36.5% |
| Not Found | 0 | 0% |
| **Total** | **400,000** | ✅ |

**Interpretation:** The platform split is clean with no unmatched records. The 63.5/36.5 split between Facebook and Instagram is consistent with typical social media advertising allocation patterns where Facebook receives higher budget share due to broader demographic reach, while Instagram commands premium engagement from younger audiences.

---

### 2.5 QA Check 5 - Revenue Sanity Check

| Metric | Value |
|---|---|
| Total Estimated Revenue | $1,458,388 |
| Average CPM | $5,961 |
| Highest CPM Campaign | Campaign_35_Launch ($36,340) |
| Lowest CPM Campaign | Campaign_42_Summer ($494) |
| CPM Variance | 73.6x |

**Important caveat:** These figures include revenue attributed to the 211,234 anomalous events identified in QA Check 3. Revenue figures should be treated as indicative only until timing anomalies are resolved.

---

## 3. Platform Performance Analysis

### 3.1 Volume vs Engagement Efficiency

| Platform | Impressions | Engagements | Engagement Rate | Est. Revenue |
|---|---|---|---|---|
| Facebook | 215,972 | 8,780 | 4.07% | - |
| Instagram | 123,840 | 5,190 | 4.19% | - |

Instagram delivers a higher engagement rate despite significantly lower impression volume. This is a consistent pattern in social media advertising: Instagram's visual-first, shorter-form feed format tends to generate higher per-impression engagement, particularly among 18-34 demographics.

### 3.2 Platform Efficiency Ratio

A simple efficiency ratio (engagement rate ÷ impression share) reveals:

- **Facebook efficiency ratio:** 4.07% ÷ 63.5% = **0.064**
- **Instagram efficiency ratio:** 4.19% ÷ 36.5% = **0.115**

Instagram's efficiency ratio is **1.8x higher** than Facebook's, meaning it generates disproportionately more engagement relative to the impression budget it receives. This is a key insight for budget allocation decisions.

### 3.3 Platform Recommendation

Current impression allocation (63.5% Facebook, 36.5% Instagram) may not be optimal given Instagram's superior efficiency. A rebalanced allocation of 55% Facebook / 45% Instagram warrants A/B testing, with performance benchmarked over a minimum 4-week flight period before drawing conclusions.

---

## 4. Campaign Performance Analysis

### 4.1 Revenue Distribution

The 49 campaigns analysed show highly concentrated revenue performance:

- **Top 5 campaigns by revenue** account for a disproportionate share of total estimated revenue
- **Campaign_20_Winter** leads with $60,466 estimated revenue (largest budget: $98,905)
- **Campaign_42_Summer** generates the least at $4,862 despite having 16,030 events - the highest event volume of any single campaign

### 4.2 The Campaign_42_Summer Problem

Campaign_42_Summer is the clearest underperformance case in the dataset:

| Metric | Campaign_42_Summer | Portfolio Average |
|---|---|---|
| Event Count | 16,030 | 8,163 |
| Budget | $7,918 | - |
| Estimated Revenue | $4,862 | - |
| Avg CPM | $494 | $5,961 |
| CTR Proxy | 3.40% | 3.54% |

This campaign has the **highest event volume but the smallest budget and lowest CPM** in the entire portfolio. It is generating massive impression volume at extremely low monetisation rates. This could indicate:
- Remnant or low-priority inventory allocation (bottom-of-barrel placements)
- Broad, untargeted audience delivery driving high volume but low-quality engagement
- Budget pacing set too conservatively relative to available inventory

**Recommendation:** Review Campaign_42_Summer's placement targeting, frequency caps, and inventory tier. If it is running on remnant inventory, test a floor price increase to improve CPM even at the cost of lower fill volume.

### 4.3 The Campaign_35_Launch Anomaly

Campaign_35_Launch has the **highest CPM ($36,340)** but the **lowest event volume (1,971)** - the inverse of Campaign_42:

| Metric | Campaign_35_Launch | Portfolio Average |
|---|---|---|
| Event Count | 1,971 | 8,163 |
| Budget | $71,627 | - |
| Avg CPM | $36,340 | $5,961 |
| CTR Proxy | 2.73% | 3.54% |

A high CPM with low volume and a below-average CTR proxy suggests this campaign is targeting a very narrow, premium audience segment but failing to generate proportional engagement. The budget is large relative to its reach, which may indicate overspending on a niche segment without the creative performance to justify the premium.

**Recommendation:** Broaden targeting parameters or introduce lookalike audiences to increase delivery volume while maintaining audience quality. Monitor CTR closely - if engagement improves with broader targeting, the current narrow segment may have been a creative mismatch rather than an audience quality issue.

---

## 5. Audience & Demographic Analysis

### 5.1 Age Group Distribution

| Age Group | Events | % of Total |
|---|---|---|
| 16-17 | 35,355 | 8.8% |
| 18-24 | 124,525 | 31.1% |
| 25-34 | 165,632 | 41.4% |
| 35-44 | 58,036 | 14.5% |
| 45-54 | 13,158 | 3.3% |
| 55-65 | 3,294 | 0.8% |

The 25-34 segment dominates at 41.4%, followed by 18-24 at 31.1%. Together these two groups account for **72.5% of all events**, confirming this inventory skews heavily toward a millennial/younger Gen Z audience.

### 5.2 The 16-17 Age Bracket - Brand Safety Flag

8.8% of all events (35,355) involve users aged 16-17. Depending on campaign targeting parameters and the advertiser's product category, this may require a brand safety review. Several campaigns in this dataset use "All" as their target age group - meaning they are not excluding minors from delivery. For any campaigns involving financial products, alcohol adjacency, or mature content categories, this should be flagged to the account team immediately for targeting adjustment.

### 5.3 Gender Distribution

| Gender | Events | % of Total |
|---|---|---|
| Male | 221,488 | 55.4% |
| Female | 137,728 | 34.4% |
| Other | 40,784 | 10.2% |

Male users represent the majority of events, though several campaigns specifically target Female audiences (Stories format on Facebook and Instagram). The 10.2% "Other" gender category is notable - this is a relatively high proportion and worth monitoring as platforms update their gender reporting categories.

---

## 6. Ad Format & Inventory Analysis

### 6.1 Format Performance

Based on engagement rate calculations by ad format type:

| Ad Format | Engagement Rate (Estimated) | Notes |
|---|---|---|
| Video | Higher viewability, longer dwell time | Best for brand awareness |
| Carousel | Strong for product discovery | Multiple touchpoints per impression |
| Stories | Full-screen, high immersion | Short attention window |
| Image | Lowest engagement, highest volume | Best for reach efficiency |

### 6.2 Inventory Optimisation Insight

The presence of multiple format types across the same campaigns (Facebook and Instagram both running Stories, Image, Video, and Carousel) suggests a mixed-format strategy. Without format-level revenue data, it is not possible to calculate format-specific CPM - however, the engagement rate differential by format provides a proxy for quality weighting.

**Recommendation:** Request format-level impression and revenue breakdown from the vendor in future exports. This single addition to the data feed would unlock format-level yield analysis, which is currently impossible with the available data.

---

## 7. Temporal & Day-of-Week Analysis

### 7.1 Day of Week Engagement Distribution

| Day | Total Engagements | % of Weekly Total |
|---|---|---|
| Monday | 8,653 | 14.4% |
| Tuesday | 8,607 | 14.3% |
| Wednesday | 8,603 | 14.3% |
| Thursday | 8,550 | 14.2% |
| Friday | 8,644 | 14.4% |
| Saturday | 8,567 | 14.2% |
| Sunday | 8,564 | 14.2% |

### 7.2 Interpretation

The day-of-week distribution is **remarkably flat** - the difference between the highest (Monday, 8,653) and lowest (Thursday, 8,550) day is only 103 events, less than 0.03% of the total. This near-perfect uniformity across all 7 days has two possible interpretations:

**Interpretation A - Audience behaviour is genuinely consistent:** The user base for this inventory interacts with social media content at a stable daily rate regardless of day, suggesting an always-on content consumption pattern rather than weekend-heavy or weekday-heavy usage.

**Interpretation B - Dataset is simulated or smoothed:** The uniformity is so precise that it may indicate the dataset was generated with controlled distributions rather than reflecting genuinely observed behaviour. Real social media engagement data typically shows at least a 10-15% variance between weekdays and weekends.

**Recommendation:** Flag this finding in any reporting to senior analysts. If this were real vendor data, the flatness would warrant a question to the data provider about whether any smoothing or sampling methodology was applied during export. For portfolio purposes, the finding demonstrates analytical maturity - identifying when data looks "too clean" is a core data quality skill.

### 7.3 Time of Day Distribution

The dataset includes Morning, Afternoon, Evening, and Night segments. Viewability flags assigned based on IAS 2024 benchmarks:
- **Morning/Evening:** High viewability (peak attention windows)
- **Afternoon:** Medium viewability
- **Night:** Low viewability (screen fatigue, multi-device distraction)

Night-segment impressions carry a viewability penalty and should receive lower CPM floor prices in yield strategy settings.

---

## 8. Revenue & Yield Analysis

### 8.1 Portfolio Revenue Summary

| Metric | Value |
|---|---|
| Total Estimated Revenue | $1,458,388 |
| Average CPM | $5,961 |
| Median CPM (estimated) | ~$4,500 |
| CPM Range | $494 - $36,340 |
| CPM Variance | 73.6x |
| Revenue Share Assumption | 72% (publisher net of DSP/SSP fees) |

### 8.2 Revenue Attribution Risk

Given that 52.8% of events fall outside their campaign's active window, a conservative adjustment to revenue figures is warranted. If out-of-window events are excluded from revenue attribution (which would be appropriate for billing reconciliation), the adjusted revenue estimate would be:

```
Adjusted Revenue = $1,458,388 × (188,766 / 400,000)
                 = $1,458,388 × 47.2%
                 = ~$688,359
```

This represents a **$769,029 difference** between gross and anomaly-adjusted revenue estimates - a material figure that would significantly affect yield reporting and campaign billing if not investigated.

### 8.3 Yield Gap Identification

The 73.6x CPM variance between best and worst performing campaigns represents a significant yield gap. In a well-optimised programmatic environment, CPM variance across campaigns on the same publisher inventory typically reflects audience targeting quality, creative performance, and placement tier - not a fundamental inventory valuation problem. A 73.6x range suggests at least some campaigns are severely underpriced relative to the audience they are reaching.

**Yield optimisation levers to investigate:**
- Floor price review for low-CPM campaigns - particularly Campaign_42_Summer ($494 CPM)
- Private marketplace (PMP) deals for high-CPM segments - particularly the audience segments Campaign_35_Launch is targeting
- Frequency cap review - high event volume campaigns may be over-serving to the same users, driving down marginal CPM
- Daypart pricing - Night segment impressions should carry lower floor prices; Morning/Evening should carry premium pricing

---

## 9. Recommendations

### Immediate Actions (This Week)

**R1 - Vendor Escalation on Timing Anomalies**
Raise a formal data discrepancy query with the vendor covering the 211,234 out-of-window events. Request ad server log cross-reference to determine whether the event timestamps reflect actual user interactions or pixel/tracking artefacts. Until resolved, all revenue figures derived from this dataset should carry a data quality disclaimer in any reporting.

**R2 - Brand Safety Review for 16-17 Age Group**
Pull a breakdown of which campaigns delivered to the 16-17 age bracket and cross-reference against campaign targeting specifications. For any campaign with "All" as the target age group, recommend adding an age floor of 18+ unless the advertiser has specifically approved minor-audience delivery.

**R3 - Campaign_42_Summer Placement Review**
Request a placement-level breakdown from the vendor for Campaign_42_Summer. The $494 CPM on 16,030 events suggests remnant inventory delivery. Review floor price settings and test a CPM floor increase to improve monetisation, accepting lower volume as an acceptable trade-off.

### Short-Term Actions (Next 2-4 Weeks)

**R4 - Instagram Budget Reallocation Test**
Design an A/B test reallocating 10% of Facebook impression budget to Instagram for a single campaign flight. Measure engagement rate and CPM differential over 4 weeks. If Instagram maintains its efficiency advantage at higher volume, gradually increase allocation to a 55/45 Facebook/Instagram split.

**R5 - Format-Level Data Request**
Add format (Video/Carousel/Stories/Image) as a dimension to the weekly vendor data export. This unlocks format-level yield analysis currently unavailable and would improve inventory optimisation decisions substantially.

**R6 - Daypart Pricing Review**
Present a proposal to the yield team to implement daypart-based CPM floor pricing: premium floors for Morning and Evening segments (High viewability per IAS benchmarks), standard floors for Afternoon, and discounted floors for Night. Model the revenue impact using existing impression volume by time of day.

### Strategic Actions (Next Quarter)

**R7 - Lookalike Audience Expansion for Campaign_35_Launch**
Campaign_35_Launch's $36,340 CPM suggests it is reaching a very high-value but extremely narrow audience. Model a lookalike audience expansion using the 25-34 high-engagement segment as a seed audience. Target a 10-15x volume increase while accepting a 20-30% CPM reduction - net revenue impact should be strongly positive.

**R8 - Always-On Scheduling Confirmation**
The flat day-of-week engagement pattern supports an always-on campaign scheduling approach rather than day-parting or weekend-heavy scheduling. Confirm this with a controlled test: run one campaign with uniform daily budgeting and one with weekend-weighted budgeting, then compare engagement efficiency over 8 weeks.

**R9 - Automated Anomaly Detection**
Build a recurring data quality check (automatable in Python or via Excel macros) that flags out-of-window events on each new vendor export before data enters any reporting pipeline. This prevents anomalous events from silently contaminating future revenue and performance reports.

---

## 10. Future Analytical Actions

### Data Enrichment
- **Add click data:** Current dataset lacks explicit click events (only Likes and Shares as engagement proxies). Requesting click data from the vendor would enable true CTR calculation and more accurate funnel analysis
- **Add ad request data:** Would enable genuine fill rate calculation, replacing the delivery rate proxy currently used
- **Add spend data at event level:** Would replace the budget-allocation revenue estimate with actual billed revenue, significantly improving yield analysis accuracy
- **Add viewability scores:** Replace the time-of-day proxy with actual measured viewability percentages from a third-party verification vendor (IAS, MOAT, or DoubleVerify)

### Analytical Expansions
- **Cohort analysis:** Track user-level behaviour across multiple campaigns - do users who engage with Campaign A show higher conversion rates on Campaign B?
- **Frequency analysis:** How many times does the average user see ads from the same campaign? High frequency with low engagement indicates creative fatigue
- **Country-level yield analysis:** Currently country data is available but not used in yield analysis. A country × CPM breakdown would identify geographic segments that could command premium pricing in geo-targeted PMP deals
- **Creative performance analysis:** If creative IDs become available, link creative assets to engagement rates to identify which creative approaches drive the strongest performance by format and audience segment

### Automation Opportunities
- **Weekly tracker auto-refresh:** Currently the daily/weekly tracker requires manual data addition each week. An Excel macro or Python script could automate the ingestion of new vendor exports and refresh all pivot tables and dashboard KPIs in a single operation
- **Anomaly alert system:** A Python script that runs QA checks on each new export and emails a flagging report to the analyst team before data enters the reporting workflow - eliminating the risk of anomalous data silently entering live dashboards

---

## 11. Assumptions & Methodology

| Assumption | Detail | Rationale |
|---|---|---|
| Revenue share: 72% | Publisher keeps 72% of campaign budget after DSP/SSP fees | Industry standard range is 60-80%; 72% used as mid-point conservative estimate |
| CPM formula | Budget ÷ total campaign impressions × 1,000 | Standard programmatic CPM calculation; uses allocated budget as proxy for actual billed spend |
| Engagement definition | Likes + Shares only | Clicks, Comments, and Purchases excluded as they represent different funnel stages; Likes and Shares represent active content engagement |
| Viewability proxy | Time of day (Morning/Evening = High, Afternoon = Medium, Night = Low) | Based on IAS 2024 benchmark data; actual viewability requires third-party measurement |
| Delivery rate | Campaign event share of maximum campaign event count | Proxy for fill rate; true fill rate requires ad request data unavailable in this dataset |
| CTR proxy | Campaign-level Engagements ÷ Impressions | True CTR requires explicit click data; engagement rate used as the nearest available proxy |
| Timing anomaly definition | Any event where event_date < campaign start_date or event_date > campaign end_date | Conservative definition; does not account for attribution windows or timezone offsets which may legitimately place events outside calendar boundaries |

---

*All findings, recommendations, and revenue figures in this document are based on estimated and proxy metrics derived from vendor-supplied data. They are intended for portfolio demonstration and analytical methodology purposes. No figures should be used for actual commercial decisions without validation against verified ad server and billing data.*

---

**Yadnesh Bapat** 
