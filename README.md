# Employee Attendance Insights — Power BI Dashboard

## An HR analytics dashboard built in Power BI to track employee presence, work-from-home, and sick leave patterns across a rolling date range. The project focuses on accurate percentage calculations, clean day-of-week trend analysis, and a readable, filterable layout.

**Note:** This project does not use any real employee IDs, names, or personal data.

## 📊 Overview

The dashboard answers three core questions for HR stakeholders:

1. What percentage of scheduled working days were employees actually present?
2. How much of that time was spent working from home vs. in-office?
3. What's the sick leave trend, and does it vary by day of the week?

It supports filtering by month and by a custom date range, and breaks down each metric both by individual employee and by day of the week.

🖼️ Screenshots

<img width="1760" height="797" alt="image" src="https://github.com/user-attachments/assets/ed9d6c51-89a6-465b-b7aa-ee60f82e18e5" />
<img width="259" height="689" alt="day-of-week-tables png" src="https://github.com/user-attachments/assets/4a6dcd17-a616-42b2-b988-eac31fe57048" />

## ⚙️ Features
KPI cards — Presence %, WFH %, and SL % at a glance
Trend charts — daily line charts for each metric across the selected date range
Employee-level table — sortable breakdown by individual, with daily attendance codes (P, WFH, SL, WO, etc.)
Day-of-week analysis — Mon–Sun breakdown for each metric, including days with zero activity (e.g., week-off days) shown explicitly as 0% rather than omitted
Month buttons + custom date range slicer — quick filtering by month or precise start/end date
Consistent color coding — each metric (Presence, WFH, SL) has a dedicated color used across cards, charts, and tables
## 🧠 Technical Highlights
### 1. Fixed a denominator bug causing percentages above 100%

The initial Total Working Days measure recalculated its denominator based on whatever filter context was active on the visual — including filters applied to the Value column (e.g., excluding week-offs). This caused the denominator to shrink whenever a Value-based filter was applied, inflating results past 100%.

Before (buggy):

dax
Total Working Days = 
VAR totaldays = COUNT('Final Data'[Value])
VAR nonworkdays = 
    CALCULATE(COUNT('Final Data'[Value]), 'Final Data'[Value] IN {"WO", "HO"})
RETURN
totaldays - nonworkdays

After (fixed):

dax
Total Working Days = 
VAR totaldays = 
    CALCULATE(
        COUNT('Final Data'[Value]),
        REMOVEFILTERS('Final Data'[Value])
    )
VAR nonworkdays = 
    CALCULATE(
        COUNT('Final Data'[Value]),
        REMOVEFILTERS('Final Data'[Value]),
        'Final Data'[Value] IN {"WO", "HO"}
    )
RETURN
totaldays - nonworkdays

REMOVEFILTERS locks the denominator so it only respects Date and Employee filters — not whatever slice of the Value column a visual happens to be filtering on. This keeps Presence %, WFH %, and SL % mathematically valid regardless of which categories are shown or hidden.

### 2. Fixed blank values on days with no matching data

Days with zero matching rows (e.g., a fixed week-off day) returned blank instead of 0%, since a blank numerator in DIVIDE stays blank even with a specified default value. Fixed by forcing blank-to-zero conversion:

dax
Presence % = DIVIDE([Present Days], [Total Working Days], 0) + 0

BLANK() + 0 evaluates to 0 in DAX, ensuring every day in the week displays a real value instead of disappearing from the visual.

### 3. Built a dedicated Day-of-Week dimension table

Sorting "Day of Week" correctly (Mon → Sun instead of alphabetical) required a Sort by Column relationship. To also ensure days with no data (e.g., week-offs) still appear in tables rather than vanishing entirely, a small standalone Days table (Mon–Sun + numeric sort key) was created and related to the fact table, with "Show items with no data" enabled on the visual-level field.

## 🛠️ Tech Stack
Power BI Desktop — data modeling, DAX measures, report design
DAX — custom measures for Presence %, WFH %, SL %, Total Working Days
Data modeling — star-schema-style relationship between a fact table (Final Data) and a small dimension table (Days)

## 📁 Files in this repo
   ```
├── README.md
├── Attendance-Sheet-2025-2026.xlsx
├── HR-Analytics.pbix
└── screenshots/
    └── (dashboard images)
```
      
## 🚀 How to Use
Clone or download this repo
Open HR-Analytics.pbix in Power BI Desktop
Data is embedded/sample — no external connection required
Use the Month buttons or Date range slicer to filter results
