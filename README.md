# Global HIV Treatment Progress: the 95-95-95 Targets

An end-to-end analytics project that cleans UNAIDS estimates in Power Query and presents progress toward the 95-95-95 targets in a four-page Power BI dashboard, with measures written in DAX.

<img width="1920" height="1080" alt="Global progress" src="https://github.com/user-attachments/assets/eb9cda06-3208-44cb-b458-42ac0127ed4e" />

---

### Business Question
*How close is the world to the 95-95-95 targets, where are the gaps, and who is being left behind?*

The targets state that, by 2025:
1. **95%** of people living with HIV know their status (1st 95)
2. **95%** of those who know their status are on treatment (2nd 95)
3. **95%** of those on treatment have a suppressed viral load (3rd 95)

> Together, these mean about **86%** of all people living with HIV are virally suppressed ($95\% \times 95\% \times 95\%$).

---

### Key Findings
*Numbers are estimates from UNAIDS 2026 data (Global, all ages, unless stated).*

1. **Global Progress (2025):** An estimated **88%** know their status, **89%** of those are on treatment, and **95%** of those on treatment are virally suppressed. The 3rd 95 has reached its target; the 1st 95 is the weakest step, about 7 points short.
2. **Progress over time:** between 2015 and 2025 the 1st 95 rose from about 69% to 88%, the 2nd 95 from about 68% to 89%, and the 3rd 95 from about 83% to 95%. These series are only reported from 2015.
3. **Regional gaps:** only one region, Western and central Europe and North America, has reached 95 on any step (97% on the 3rd 95). The Middle East and North Africa has the lowest 1st 95 at about 57% (about 38 points short), Western and central Africa is at about 69%, and the Caribbean has the lowest 2nd 95 at about 72%.
4. **Who is Left Behind:** Children (ages 0–14) trail adults on treatment access by significant percentage points, highlighting key demographic disparities.
5. **Uncertainty matters:** the global 3rd 95 estimate is 95%, but its Low to High range is wide (about 72% to 98%), so reaching the target is less certain than the headline suggests.
6. **Pace of progress:** in the 2nd 95, year-on-year gains slowed from about 4 points a year in 2016 and 2017 to 2 points or less a year since 2021, with no change in 2024. 


---

### Recommendations
* **Prioritize testing and diagnosis** (1st 95) in the Middle East and North Africa and Western and central Africa, where fewer than 70% of people living with HIV know their status.
* **Strengthen linkage to treatment** in the Caribbean (72%) and Eastern Europe and central Asia (77%), the lowest 2nd 95 values.
* **Report estimates with their ranges**, so a result at target is not read as certain when the uncertainty is wide. 

---

## 📊 Dashboard Overview

| Page | Question It Answers | Main Visuals |
| :--- | :--- | :--- |
| **1. Global Progress** | How far has the world come on each of the three steps? | KPI cards (latest estimate & gap to 95), trend line with 95% target line, takeaway summary text box. |
| **2. Regional Gaps** | Which regions and countries are closest to or furthest from 95? | Regional bar chart, Red-Amber-Green heat table, country filled map. |
| **3. Who Is Left Behind** | Which groups and which steps lag, and how certain are the estimates? | Group comparison (children, women, men), bottleneck chart (gap to 95), range estimates (Low/High), YoY change. |
| **4. Data Notes** | How should the numbers be read? | Data source descriptions, definitions, cleaning rules, and analytical limitations. |


<img width="1920" height="1080" alt="Regional gaps" src="https://github.com/user-attachments/assets/12d18b83-f568-4dc4-82f3-aa6ff7a5d6b3" />

<img width="1920" height="1080" alt="Demographic gaps" src="https://github.com/user-attachments/assets/a4cfbca2-d00e-4da6-98d1-398d78d1a0db" />

---

## 🛠️ Data & Architecture

### Source Data
* **Source:** UNAIDS 2026 estimates, HIV estimates with uncertainty bounds (1990–2025).
* **Sheet Used:** `HIV-Test-&-Treat_ByArea` (Global, regions, and countries from 2010 to 2025).
* **Raw Shape:** Wide format, featuring 3 title rows, 3 header rows built from merged cells, ~100 columns, and ~2,944 data rows.
* **Clean Shape:** One normalized long table, `HIV_Data`, containing 141,439 rows and 12 columns.

### Tools & Skills
* **Excel:** Inspecting source structure and cross-checking numbers against dashboard outputs.
* **Power Query (M):** Rebuilding headers, unpivoting attribute columns, splitting text fields, type conversion, conditional flags.
* **Power BI:** Data modeling, dynamic visual design, conditional background formatting, filled maps.
* **DAX:** Measures for latest estimates, gap to target, range calculations, and year-over-year change.

---

## 🧹 Data Cleaning Process (Power Query)

The full step-by-step record is documented in `docs/data_cleaning_report.pdf`. Key steps included:

1. **Header Reconstruction:** Kept the raw sheet untouched as a reference query and built a small `Headers` query from the top three header rows (*Keep top rows $\rightarrow$ Transpose $\rightarrow$ Fill Down $\rightarrow$ Merge with `|` separator*).
2. **Bulk Renaming:** Renamed ~100 measurement columns in a single automated step using the constructed headers.
3. **Unpivoting & Splitting:** Unpivoted 97 measurement columns into long format (one row per measurement), then split names into `Indicator`, `Group`, and `Statistic`.
4. **Type Conversion & Value Cleaning:**
   * `...` converted to `null` (missing data is not zero).
   * `>98` converted to `98` with a `>` flag in `Qualifier`.
   * `1.3 m` converted to `1,300,000`.
   * `680 000` converted to `680000`.
5. **Helper Columns:** Added `Qualifier` (`Exact`, `>`, `No data`), `Unit` (`Percent` or `Count`), `Indicator_Short`, and `Area_Level` (`Global`, `Region`, `Country`).
6. **Standardization:** Standardized group and population labels (e.g., `(All ages)` $\rightarrow$ `All ages`).
7. **Validation:** Conducted row count checks, duplicate checks, and spot checks of Global 2025 values against source Excel files.

| Raw Value | Cleaned Value | Reason / Logic |
| :--- | :--- | :--- |
| `...` | `null` | No data is not the same as zero. |
| `>98` | `98` (flagged `>`) | It represents a lower bound, not an exact value. |
| `1.3 m` | `1,300,000` | `m` stands for millions (rounded/approximate). |
| `680 000` | `680000` | Space acts as a thousands separator. |
| Global, Regions, Countries | Separated by `Area_Level` | Adding them together directly causes double counting. |

---

## 📐 Key DAX Measures

```dax
-- Calculate percentage estimates specifically
Estimate % = 
CALCULATE(
    AVERAGE(HIV_Data[Value_Num]), 
    HIV_Data[Statistic] = "Estimate", 
    HIV_Data[Unit] = "Percent"
)

-- Dynamically pull the latest available estimate per series
Latest Estimate % = 
CALCULATE(
    [Estimate %], 
    HIV_Data[Year] = CALCULATE(MAX(HIV_Data[Year]), ALL(HIV_Data[Year]))
)

-- Distance from the 95% target
Gap to 95 = 95 - [Latest Estimate %]

-- Year-over-Year change calculation
YoY change = 
VAR CurrentYear = MAX(HIV_Data[Year]) 
VAR Prior = CALCULATE([Estimate %], HIV_Data[Year] = CurrentYear - 1) 
RETURN 
IF(ISBLANK(Prior), BLANK(), [Estimate %] - Prior)

Filtering Note: Every visual in the dashboard is filtered to one Area_Level, one Group (usually All ages), and a specific step to ensure values are never improperly aggregated across mismatched attributes.

💡 Key Learnings & Troubleshooting
Large Column Transposition: Unpivoting the full raw dataset generated over 2,900 temporary columns. I learned to isolate structural transformations on lightweight queries before merging.

Visual Context & Filter Scope: Discovered a mismatch where a card displayed 88% while a chart showed 78%. The discrepancy was traced to missing page-level context filters, causing the chart to unweightedly average all groups. I now cross-validate all visual totals against underlying tables.

DAX Syntax & Variable Scoping: Resolved DAX errors (e.g., 202.50K%) caused by incomplete multi-line variable pastes. Learned that DAX measure identity is defined strictly by the initial string preceding the first equals sign.

⚠️ Data Limitations
Position-Based Renaming: Power Query column renaming relies on ordinal positions. Reordered or new columns in future source refreshes require verification.

Capped Values: Figures capped at >98 mask exact upper-tier achievements.

Rounded Millions: Counts formatted in millions are approximate due to initial rounding.

Uncertainty Ranges: Wide confidence bounds mean single point estimates reaching target may still carry statistical uncertainty.

Historical Bounds: The 95-95-95 series tracking begins in 2015; earlier years were not reported under this framework.

📁 Repository Structure
Plaintext
hiv-treatment-progress/
├── README.md
├── hiv_95_dashboard.pbix          # Main Power BI report file
├── images/                        # Dashboard preview screenshots
│   ├── 01_global_progress.png
│   ├── 02_regional_gaps.png
│   ├── 03_who_is_left_behind.png
│   └── 04_data_notes.png
└── docs/
    └── data_cleaning_report.pdf   # Step-by-step M query documentation
🚀 How to Open the Report
Download hiv_95_dashboard.pbix and open it in Power BI Desktop.

If the data source path changes, go to Transform data > Data source settings and re-point the path to your local copy of the UNAIDS workbook.

Select Refresh and inspect Power Query Applied Steps if prompted.

🔗 Related Projects & Profile
GitHub Profile: github.com/Blessingjatto

LinkedIn: linkedin.com/in/blessing-jatto

Related Project: E-commerce Customer Churn and Retention Analysis

Data source: UNAIDS 2026 estimates. Created for analytical portfolio purposes.
