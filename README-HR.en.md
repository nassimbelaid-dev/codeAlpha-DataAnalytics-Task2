# 👥 HR Analytics Dashboard — Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Star Schema](https://img.shields.io/badge/Data%20Modeling-Role--Playing%20Dimension-blue?style=for-the-badge)
![HR Analytics](https://img.shields.io/badge/HR%20Analytics-Turnover%20%26%20Attrition-purple?style=for-the-badge)
![AI Visuals](https://img.shields.io/badge/Power%20BI-Key%20Influencers%20%2F%20Decomposition%20Tree-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

A Power BI dashboard for tracking recruitment, turnover, satisfaction/performance and attrition risk, built on the *Human Resources Data Set* (HRDataset_v14).

---

## 🎯 Project goal

Give an HR function a clear, actionable view of the workforce, organized around **three business angles**:

1. **Recruitment** — which channels bring the best hires (the ones who stay).
2. **Turnover / Attrition** — how many people leave, voluntarily or not, and at what pace.
3. **Satisfaction, performance & attrition risk** — who's engaged, who's performing, and who's at risk of leaving.

## 🗂️ Dataset

- **311 rows** × **36 columns** — one row per employee (fictitious company)
- Source: *Human Resources Data Set* (HRDataset_v14) by Rich Huebner & Dr. Carla Patalano
- Covers recruitment (`RecruitmentSource`), turnover (`DateofHire`, `DateofTermination`, `TermReason`, `EmploymentStatus`), performance/satisfaction (`PerformanceScore`, `EngagementSurvey`, `EmpSatisfaction`) and attendance (`Absences`, `DaysLateLast30`)

**Known limitation:** the dataset has no job-requisition/posting date, so a true *time-to-fill* metric cannot be computed — this limitation was identified and deliberately left out of scope rather than forced artificially.

## 🧹 Data cleaning (Power Query)

### Removed columns (19)
| Reason | Columns |
|---|---|
| Numeric ID duplicating a kept text column | `MarriedID`, `MaritalStatusID`, `GenderID`, `EmpStatusID`, `DeptID`, `PerfScoreID`, `PositionID`, `ManagerID` |
| Redundant with `RecruitmentSource` | `FromDiversityJobFairID` |
| Geo/PII, not needed | `State`, `Zip`, `DOB` |
| Sensitive personal attributes, no stated DEI requirement | `Sex`, `MaritalDesc`, `CitizenDesc`, `HispanicLatino`, `RaceDesc` |
| Redundant flag (already covered by `EmploymentStatus`) | `Termd` |

### Columns kept (17)
`Employee_Name`, `EmpID`, `Department`, `Position`, `ManagerName`, `Salary`, `DateofHire`, `DateofTermination`, `TermReason`, `EmploymentStatus`, `RecruitmentSource`, `PerformanceScore`, `EngagementSurvey`, `EmpSatisfaction`, `SpecialProjectsCount`, `LastPerformanceReview_Date`, `DaysLateLast30`, `Absences`.

### Type/formatting decisions
- Dates are US-format (`M/D/YYYY`) → locale forced to **English (United States)** in Power Query, not the default locale.
- `DateofTermination` is blank for every active employee — expected, don't fill it: it's the field that separates active vs. terminated.
- `Salary` → Fixed Decimal Number, Currency format.
- `EngagementSurvey` → Decimal Number, 2 decimals (already a correct 1–5 scale).
- `EmpSatisfaction`, `SpecialProjectsCount`, `DaysLateLast30`, `Absences` → Whole Number.
- `EmpID` → Whole Number, marked **Do not summarize**.

## 🧩 Data modeling — Star schema with a role-playing dimension

Unlike the financial model (one row per company-quarter), this dataset is **at employee grain**, with **two dates** attached to each row (hire + termination). This introduces a new technique: a **role-playing dimension** — a single calendar table used twice.

**Structure:** `FactEmployee` (1 row per employee) → `DimDate` (twice) + `DimDepartment`

- `DimDate[Date] → FactEmployee[DateofHire]` — **active** relationship by default (drives "New Hires").
- `DimDate[Date] → FactEmployee[DateofTermination]` — **inactive** relationship, activated on demand in specific measures via `USERELATIONSHIP` (drives "Terminations").
- `DimDepartment[DepartmentKey] (1) → FactEmployee[DepartmentKey] (many)`, with `Position` nested under `Department` as a hierarchy.

**Key steps:**
1. Load the CSV, remove the 19 columns, set types.
2. Add a calculated column `Tenure (Years)` = duration from `DateofHire` to (`DateofTermination` or today) ÷ 365.25.
3. Build `DimDepartment` via query reference: keep `Department` + `Position`, remove duplicates, add `DepartmentKey`.
4. Build `DimDate` **in DAX** (not Power Query, since it must span two source columns):
   ```dax
   DimDate = CALENDAR(
       MIN(FactEmployee[DateofHire]),
       MAX(TODAY(), MAX(FactEmployee[DateofHire]))
   )
   ```
   Mark this table as the Date Table in Model view.
5. In `FactEmployee`, drop `Department`/`Position` (replaced by `DepartmentKey`), rename the query.

## 🧮 Core DAX measures

```dax
Active Headcount = CALCULATE(COUNTROWS(FactEmployee), FactEmployee[EmploymentStatus] = "Active")

New Hires = CALCULATE(COUNTROWS(FactEmployee))   -- uses the active DateofHire relationship

Terminations =
CALCULATE(COUNTROWS(FactEmployee), USERELATIONSHIP(DimDate[Date], FactEmployee[DateofTermination]))

Headcount (As of Date) =
CALCULATE(
    COUNTROWS(FactEmployee),
    FILTER(
        ALL(FactEmployee),
        FactEmployee[DateofHire] <= MAX(DimDate[Date]) &&
        (FactEmployee[DateofTermination] > MAX(DimDate[Date]) || ISBLANK(FactEmployee[DateofTermination]))
    )
)

Turnover Rate % = DIVIDE([Terminations], [Headcount (As of Date)])

Voluntary Turnover Rate % =
VAR VolTerms = CALCULATE([Terminations], FactEmployee[EmploymentStatus] = "Voluntarily Terminated")
RETURN DIVIDE(VolTerms, [Headcount (As of Date)])

Avg Engagement Survey = AVERAGE(FactEmployee[EngagementSurvey])
Avg Employee Satisfaction = AVERAGE(FactEmployee[EmpSatisfaction])

% High Performers =
DIVIDE(CALCULATE(COUNTROWS(FactEmployee), FactEmployee[PerformanceScore] = "Exceeds"), [Active Headcount])

Forecasted Hiring Need (Next Qtr) =
VAR ThisQuarterKey = YEAR(TODAY()) * 4 + QUARTER(TODAY())
VAR LastCompleteQuarterKey = ThisQuarterKey - 1
VAR StartQuarterKey = LastCompleteQuarterKey - 2
VAR LastThreeQuarters =
    FILTER(
        VALUES(DimDate[QuarterKey]),
        DimDate[QuarterKey] >= StartQuarterKey && DimDate[QuarterKey] <= LastCompleteQuarterKey
    )
RETURN
AVERAGEX(
    LastThreeQuarters,
    CALCULATE([Terminations], USERELATIONSHIP(DimDate[Date], FactEmployee[DateofTermination]))
)
```
*(requires the column `QuarterKey = YEAR(DimDate[Date]) * 4 + QUARTER(DimDate[Date])` in `DimDate`)*

`Forecasted Hiring Need` mirrors the exact 3-quarter rolling-average logic used in the financial model (`Forecast Revenue (3Q Avg)`), applied to attrition instead of revenue, to answer: *how many replacement hires should we plan for next quarter?*

## 📑 Report layout — 2 tabs

### Tab 1 · Workforce & Recruitment Overview
Slicers (Department / Status / Hire Year / Source), 4 KPI cards (Active Headcount, New Hires, Terminations, Turnover Rate %), a "Headcount Bridge" waterfall chart (start → +hires → −terminations → end), a combo chart of hires vs. terminations with **Power BI's native forecast** on the net headcount line, a treemap of recruitment source effectiveness (size = hires, color = retention rate), and an exportable employee roster table with data bars.

### Tab 2 · Performance, Satisfaction & Attrition Risk
Slicers (Department / Manager / Performance), 4 KPI cards (Avg Engagement, Avg Satisfaction, % High Performers, Voluntary Turnover %), a Satisfaction vs. Engagement scatter/bubble chart (bubble = tenure, color = department), a Satisfaction-vs-benchmark gauge, a **Key Influencers** AI visual identifying which factors most increase attrition risk, and a **Decomposition Tree** AI visual to explore where attrition concentrates (Department → Source → Performance).

## 🛠️ Skills demonstrated

- Data cleaning with locale handling (US dates) and sensitive/PII column management
- Modeling a **role-playing dimension** (`USERELATIONSHIP`)
- Building a date table in DAX (`CALENDAR`) rather than in Power Query
- Advanced DAX measures: point-in-time headcount, turnover rate, rolling-average forecasting
- Using Power BI's **native AI visuals** (Key Influencers, Decomposition Tree)
- Designing a bilingual HR report (English data model, French interface)

## 🚀 Usage

1. Open the `.pbix` file in Power BI Desktop.
2. Refresh the data if needed (Home → Refresh).
3. Navigate between the two tabs using the report's bottom navigation bar.

## 📄 License

Project built for demonstration / portfolio purposes.
