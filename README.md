# Hospital Quality Analysis

An end-to-end analytics project on the CMS Hospital General Information dataset (4,812 hospitals). It uses **Python, SQLite, SQL, Excel and Power BI** to study hospital ratings, rating coverage and CMS quality-domain performance across states, ownership types and hospital types.

![Hospital Quality Analytics dashboard]<img width="1582" height="803" alt="Final Dashboard" src="https://github.com/user-attachments/assets/1ff14970-c4b3-4dba-ba99-485a0fdf530a" />



---

## Key findings

| Metric | Value |
|---|---|
| Total hospitals | 4,812 |
| Rating coverage | 74.13% |
| Missing overall ratings | 25.87% (about 1 in 4 hospitals) |
| Hospitals offering emergency services | 93.45% |
| Below-average quality (national comparison) | 19.19% |
| Average hospital rating | 3.06 out of 5 |

- **Rating data is incomplete.** About a quarter of hospitals have no overall rating, so any ranking should be read with coverage in mind.
- **Ownership mix:** Voluntary non-profit (Private) hospitals are the largest group (2,052, 43%), followed by Proprietary (800, 17%).
- **Small groups can look misleading.** Physician-owned hospitals have the highest raw average rating (3.75), but there are only 68 of them. This is why the project uses a sample-adjusted rating score (see below).
- **Hospital type:** Critical Access hospitals average 3.32 and Acute Care hospitals 3.01. These types differ in size and services, so this is not a like-for-like comparison.
- **Largest states by hospital count:** Texas (411) and California (343).

*Figures are from the dashboard report dated November 18, 2026.*

---

## Project workflow

```
Cleaned hospital CSV
→ SQLite database
→ SQL analysis
→ SQL summary views
→ Python export script
→ Excel report workbook
→ Power BI dashboard
```

## Tools used

| Stage | Tools |
|---|---|
| Data loading | Python, pandas, SQLite |
| Analysis | SQL (aggregations, `UNION ALL`, views) |
| Reporting | Excel (PivotTables, slicers, PivotCharts, XLOOKUP) |
| Dashboard | Power BI, DAX |

## Repository structure

```
Hospital-Quality-Analysis/
├── Data/                   # Cleaned dataset and SQL output CSV files
├── SQL/                    # SQL analysis scripts
├── Excel/                  # Excel report workbook
├── Power BI/               # Power BI dashboard (.pbix) and screenshots
├── load_to_sqlite.py       # Loads the cleaned CSV into SQLite
├── export_sql_outputs.py   # Exports SQL query results to CSV files
└── README.md
```

## How to run

1. Install Python 3 and pandas (`pip install pandas`).
2. Run `python load_to_sqlite.py` to load the cleaned CSV into a SQLite database.
3. Run the SQL scripts in the `SQL/` folder against the database.
4. Run `python export_sql_outputs.py` to export the SQL results to CSV.
5. Open the Excel workbook in `Excel/` and the dashboard in `Power BI/` (Power BI Desktop is required for the `.pbix` file).

---

## Data source

The dataset is the cleaned version of the **CMS Hospital General Information** dataset. The cleaning was done in a separate Python/pandas project:

- Standardized column names
- Cleaned key fields such as state, ZIP code, phone number and provider ID
- Removed unnecessary footnote columns
- Validated the cleaned output

### Missing values

The cleaning pipeline converted placeholder labels such as `"Not Available"`, `"Not Applicable"`, `"N/A"`, `"NA"` and blank cells into true missing values. When the cleaned CSV is loaded into SQLite through pandas, these are preserved as SQL `NULL`, so missing hospital ratings are queried with `IS NULL` instead of searching for text labels.

---

## SQL analysis

The SQL analysis includes:

- Database preview and schema checks
- Data quality checks for key fields (provider ID, ZIP code, phone number, state code, hospital rating)
- Quality-domain analysis using the CMS national comparison fields
- Summaries and rating rankings by **state**, **ownership type** and **hospital type**

Each analysis level produces three types of output:

- **Summary tables:** total hospitals, rated hospitals, emergency service coverage, missing ratings and average rating.
- **Sample-adjusted rating ranking tables:** compare average ratings while reducing the influence of groups with very few rated hospitals.
- **Quality-domain comparison tables:** show how below-average or missing quality-domain results vary by state, ownership type and hospital type.

### Quality domains

The project analyzes seven CMS national comparison domains: mortality, safety of care, readmission, patient experience, effectiveness of care, timeliness of care and efficient use of medical imaging.

These fields are **categorical**, not numeric. Instead of averaging them, the SQL counts how many hospitals are above, the same as, below, or missing compared to the national average. The columns are reshaped from wide to long format using `UNION ALL`, which makes it easier to compare domains and connect them back to each analysis level.

### Sample-adjusted rating score

A raw average rating can mislead when a group has very few rated hospitals. To reduce this, the project uses:

```
sample_adjusted_rating_score = avg_rating * rated_hospitals / (rated_hospitals + 20)
```

This is **not an official CMS metric**. It is an internal ranking aid for this analysis. The summary tables still include the raw average rating, and the ranking tables use the adjusted score.

### Important analysis notes

- Missing ratings are treated as missing values, **not zeroes**. Averages use only hospitals with an overall rating.
- Some groups have limited rating coverage. For example, Children's hospitals have no overall ratings in this dataset, so they are included in hospital counts, emergency service and missingness analysis but excluded from rating-ranking comparisons.
- National comparison fields also contain missing values. Missingness is analyzed directly so the quality-domain results are not read without context.

---

## Excel report

Location: `Excel/hospital_quality_report.xlsx` <img width="2306" height="1037" alt="pivot_analysis" src="https://github.com/user-attachments/assets/972dbb56-5d5e-4140-bf79-b5d5f9dbb948" />


The workbook uses the exported SQL output CSV files from `Data/output/` and includes:

- Imported SQL output tables (state, ownership, hospital type and quality-domain analyses)
- A front-page report with four sections: **Dataset Overview**, **Rating Coverage**, **Rating Rankings** and **Quality Domain Findings**
- A `Chart Sources` sheet with the mini-tables behind the report visuals
- A `Pivot Analysis` sheet with a slicer-based state quality-domain explorer, a PivotChart, and an ownership-by-domain PivotTable heat map of below-national-average rates
- XLOOKUP to bring sample-adjusted scores into the state pivot analysis

---

## Power BI dashboard

Location: `Power BI/hospital_quality_dashboard.pbix`

The dashboard is the interactive layer of the project. It uses the SQL and Excel outputs to summarize hospital distribution, rating coverage, adjusted rating rankings and quality-domain patterns.

**Main dashboard:**

- KPI cards: Total Hospitals, Rating Coverage %, Emergency Services %, Missing Ratings %, Below Average Quality % and Average Hospital Rating
- Hospitals by ownership (donut chart) and average rating by ownership
- Average rating and emergency hospitals by hospital type
- State-level charts: total hospitals, rating coverage, missing ratings, emergency hospitals and average rating
- Slicers for State, Hospital Type and Hospital Ownership

**Additional analysis pages:**

1. **Overview:** KPI cards, a slicer-controlled chart of hospital counts by state, ownership or type, a donut chart of rating availability, and a 0–5 gauge for the average rating.
2. **Rating Rankings:** a scatter plot of raw average rating against the sample-adjusted score (bubble size = rated hospitals), an adjustment-gap chart and a ranking chart.
3. **Quality Domain Analysis:** slicers for analysis level and quality domain, below-national-average percentage by group, and matrices of below-average and missing-comparison rates.

### Power BI modeling notes

Helper tables were created with DAX to support the interactive visuals:

- `Overview_Group_Counts`: hospital counts by state, ownership type and hospital type
- `Combined_Rating_Rankings`: state, ownership and hospital type rankings combined
- `Combined_Quality_Domains`: quality-domain outputs across all analysis levels
- `Rating_Availability`: rated versus missing ratings

DAX measures were used for metrics such as rating coverage, average raw rating, average adjusted score and average below-average percentage.

---

## Skills demonstrated

SQL data analysis and quality checks · wide-to-long reshaping with `UNION ALL` · handling missing data · small-sample bias adjustment · Excel PivotTables, slicers and XLOOKUP · Power BI dashboard design and DAX · healthcare analytics

## Author

**Mekala Sanjana** · 
Data Analytics Portfolio Project
LinkedIn: [linkedin.com/in/mekala-sanjana](https://linkedin.com/in/mekala-sanjana)
