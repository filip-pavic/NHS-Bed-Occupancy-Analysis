# NHS Bed Capacity Benchmarking Dashboard | GSTT vs London Peer Trusts

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Executive Summary](#1-executive-summary)
3. [Context & Objectives](#2-context--objectives)
4. [Dashboard](#6-dashboard)
5. [Methodology & Data Prep](#3-methodology--data-prep)
6. [DAX Logic & Measures](#4-dax-logic--measures)
7. [Tools & Skills Applied](#5-tools--skills-applied)
8. [Key Insights](#7-key-insights)
9. [Dataset Sources](#8-data-sources)

You may access interractive Dashboard here:
https://app.powerbi.com/view?r=eyJrIjoiZGRjN2YxMTItMThhZC00MzljLWI1ZTMtNWJmMjMzYTA2OTA3IiwidCI6ImQ5NzRiNGRmLWVlYzItNDMzZS1hOTE4LTNmZTE4NDEyNzM1ZiIsImMiOjh9
---

## 1. Project Overview

Managing hospital bed capacity is a key operational focus across the NHS. Guidelines from NHS England and clinical bodies recommend that General & Acute (G&A) bed occupancy should remain at or below **85%** to maintain safe patient flow, reduce emergency department delays, and minimize operational risk.

This Power BI dashboard analyzes quarterly bed availability and occupancy for **Guy’s and St Thomas’ NHS Foundation Trust (GSTT)** compared to key London peer trusts (*King's College Hospital*, *Imperial College Healthcare*, *UCLH*, *Royal Free London*, *St George's*, and *Lewisham & Greenwich*). 

The goal of this project was to calculate accurate full year occupancy rates, benchmark GSTT against regional averages, and track remaining headroom against the 85% safety threshold.
---
### Why this matters?
This analysis provides NHS operational teams with:
- visibility of capacity headroom
- regional benchmarking
- early warning indicators before occupancy pressure increases
---

## 2. Executive Summary

Across the evaluated reporting year, **Guy’s and St Thomas’ NHS Foundation Trust (GSTT)** maintained an average occupancy rate of **83.13%**, keeping operations within the national **85% safety threshold** with a **+1.87% (+1.9%) capacity buffer**.

By contrast, neighboring peer trusts averaged a full-year occupancy rate of **89.34%**, operating **4.34 percentage points above** the safety target. 

| Operational Indicator | GSTT | Peer Average | Target Limit | Status |
| :--- | :---: | :---: | :---: | :---: |
| **Occupancy Rate** | **83.13%** | **89.34%** | $\le 85.00\%$ | Within Target (GSTT) |
| **Headroom Buffer** | **+1.87%** | **-4.34%** | $\ge 0.00\%$ | Safe Buffer |
| **Regional Gap** | *-6.21% vs. Peers* | — | — | GSTT below peer average |

---

## 3. Context & Objectives

Evaluating occupancy across quarters requires careful aggregation. Taking a simple arithmetic average of quarterly percentages ($Average = \frac{Q1 + Q2 + Q3 + Q4}{4}$) introduces distortion because total available beds fluctuate seasonally when escalation beds open in winter.

To solve this, for all calculations in this project I used **weighted totals** ($\frac{\sum \text{Occupied Beds}}{\sum \text{Available Beds}}$) across the full dataset to ensure accurate representation of trust performance.

---
## 4. Dashboard

<img width="1363" height="765" alt="image" src="https://github.com/user-attachments/assets/61de08ba-83e3-4111-9034-27429e065dca" />

The dashboard is structured for quick readability:

* **Top Navigation:** A horizontal `Select Reporting Period` tile slicer (`[ All ] [ Q1 ] [ Q2 ] [ Q3 ] [ Q4 ]`) placed at the top right.
* **Executive Summary Cards:** 
  * **GSTT Occupancy:** `83.1%` (Green)
  * **Peer Average:** `89.3%` (Red)
  * **Headroom:** `+1.9%` (Green)
* **Visual Layer:**
  * **Line Chart:** *"How has bed occupancy fluctuated across quarters for GSTT vs. peers?"* (Uses clean short acronyms: `GSTT`, `KCH`, `Imperial`, `UCLH`, `L&G`, `St George's`).
  * **Bar Chart:** *"How does GSTT's full-year occupancy compare to neighboring trusts?"*
  * **Table:** Standard data table showing quarterly bed counts and occupancy rates per trust against the 85% threshold.
---

## 4. Methodology & Data Prep

1. **Data Cleaning (Power Query):**
   * Processed NHS KH03 quarterly returns for General & Acute beds.
   * Standardized trust naming conventions and handled missing/duplicate values.
   * Extracted data from four different Excel files, compiled data needed, and transformed the necessary data into new file.
2. **Data Modeling & DAX:**
   * Created calculated columns for shortened trust names to clean up visual legends.
   * Built dynamic DAX measures for weighted occupancy rates, peer benchmarks, and conditional status formatting.

---

## 5. DAX Logic & Measures

### GSTT Occupancy Rate (%)
Weighted total occupied beds divided by weighted available beds
```dax
Occupancy Rate % = 
DIVIDE(
    [Total Occupied Beds],
    [Total Available Beds],
    0
)
```

### Peer Average Occupancy (%)
Calculates weighted average across peer trusts, dynamically excluding GSTT
```dax
Peer Avg Occupancy Rate % = 
CALCULATE(
    [Occupancy Rate %],
    'GSTT_Bed_Occupancy_FullYear'[Org Code] <> "RJ1"
)
```

### Headroom Buffer (%)
Measures remaining margin before breaching the 85% cap
```dax
Headroom % = 
0.85 - [Occupancy Rate %]
```

### Dynamic Color Formatting
Drives card status colors based on headroom value.
```dax
Occupancy Status Color = 
VAR _Rate = [Occupancy Rate %]
VAR _NormalizedRate = IF(_Rate > 1, _Rate / 100, _Rate)
RETURN
IF(
    _NormalizedRate > 0.85,
    "#B05246", -- Soft Red / High Risk (Over 85%)
    "#689F38"  -- Soft Green / Safe Operational Level (Under 85%)
)
```

---

## 6. Tools & Skills Applied

* **Power BI Desktop:** Dashboard layout, interactive slicers, visual formatting
* **DAX:** Filter context (`CALCULATE`), weighted aggregations, dynamic visual measures
* **Power Query:** Data gathering, data transformation, column parsing, data hygiene
* **Healthcare Analytics:** Understanding NHS KH03 bed performance standards and operational targets

---

## 7. Key Insights

1. **GSTT Capacity:** GSTT operated within target limits across the year 2025/2026, maintaining a **1.87% headroom buffer**.
2. **Peer Pressures:** Most regional peer trusts operated above 85% occupancy, with peaks reaching $>90\%$ in winter quarters.
3. **Considerations:** High peer occupancy indicates wider system pressure in London, which can lead to emergency flow friction across neighboring trusts.
4. **Possible Future Problem:** Bed occupancy closely reaches 85% (84.88%) during Q4 making it a precaution for possible future complications and staff overwork.
---

## 8. Data Sources

* **NHS England:** I have used four datasets:  
NHS organisations in England, Quarter 4, 2025-26  
NHS organisations in England, Quarter 3, 2025-26  
NHS organisations in England, Quarter 2, 2025-26  
NHS organisations in England, Quarter 1, 2025-26 (revised 25.11.2025)  
found at:  
*Bed Availability and Occupancy Data (KH03 Returns)*: https://www.england.nhs.uk/statistics/statistical-work-areas/bed-availability-and-occupancy/bed-data-overnight/
* **Benchmark Standard:** NHS England & Royal College of Emergency Medicine operational guidelines (85% recommended occupancy limit).
