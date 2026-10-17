---
  title: 'Power BI Advanced: Hands-On Lab Exercises'
  module: 'Power BI Advanced Lab'
---

# Power BI Advanced — Hands-On Lab Exercises

## Lab Introduction

These exercises accompany the Power BI Advanced training session. Each lab can be completed independently or in sequence and builds on a shared `AdvancedSalesReport.pbix` file covering performance tuning, security, governance, and AI-assisted analysis.

This set of exercises covers the following labs:

- **Lab A**: Performance Tuning & DAX Optimisation
- **Lab B**: Row-Level Security, Dynamic RLS & Object Level Security
- **Lab C**: Deployment Pipelines & Report Governance
- **Lab D**: AI Visuals & Copilot Capabilities

---

## Pre-requisites

- **Power BI Pro** license assigned to your account and the ability to sign in at [app.powerbi.com](https://app.powerbi.com)
- **Power BI Desktop** installed, November 2025 or newer ([aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore)) — UI elements may vary slightly depending on your version
- A shared workspace in the Power BI Service that you can publish to
- **Lab C** (Deployment Pipelines) requires workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a **PPU** workspace — trainer-led demo if neither is available
- **Lab E** (Copilot) requires **any Fabric F SKU (F2+)** or **PPU** — trainer-led demo if unavailable; built-in AI visuals (anomaly detection, forecasting, Smart Narrative, Q&A) work for all participants in Power BI Desktop

---

# Lab A: Performance Tuning & DAX Optimisation

## Lab Introduction

In this lab, you open a Power BI Desktop report built on AdventureWorks sales data that contains measures using inefficient DAX patterns. You use Performance Analyzer to capture timing data, identify the most expensive visual, analyze the DAX query, apply an optimization, and re-measure to confirm the improvement. You also explore cardinality by examining column statistics in the model.

This lab will guide you through the following tasks:

- Capture a performance baseline with Performance Analyzer
- Analyze the slow DAX query in DAX query view
- Optimize the DAX measure using a variable
- Examine column cardinality with `COLUMNSTATISTICS()`
- Verify the improvements against your baseline

## Lab Scenario

Your organisation's AdventureWorks sales report has a Sales Overview page with several visuals, including a table showing Year, Total Sales, and Sales YoY Growth. Some measures in the model use intentionally inefficient DAX patterns — such as repeated subexpressions — that slow down report rendering. Your task is to measure the current performance, diagnose the inefficient pattern, fix it, and verify the improvement.

## Architecture Overview

```
Power BI Performance Tuning Workflow
=====================================

AdvancedSalesReport.pbix (AdventureWorks model)
└── Sales Overview page → Table visual (Year, Total Sales, Sales YoY Growth)
        │
        ▼
Performance Analyzer (Optimize ribbon → Performance analyzer)
├── Start recording → Refresh visuals → Stop recording
└── Expand Table visual → record baseline DAX query time

        │
        ▼
DAX query view (Run in DAX query view → Run)
├── SUMMARIZECOLUMNS / ROLLUPADDISSUBTOTAL / TOPN generated query
└── Sales YoY Growth measure → repeated CALCULATE(SAMEPERIODLASTYEAR) subexpression

        │
        ▼
Optimized Measure
└── VAR SalesPriorYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
    RETURN DIVIDE([Total Sales] - SalesPriorYear, SalesPriorYear)

        │
        ▼
COLUMNSTATISTICS() → column cardinality (SalesOrderNumber, Date, Cost, Sales, ResellerKey...)

        │
        ▼
Performance Analyzer (re-run) → compare DAX query time to baseline
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Capture a performance baseline with Performance Analyzer |
| Task 2 | Analyze the slow DAX query in DAX query view |
| Task 3 | Optimize the DAX measure using a variable |
| Task 4 | Examine column cardinality |
| Task 5 | Verify the improvements |

---

## Task 1: Capture a Performance Baseline

1. Download the AdvancedSalesReport.pbix found in Power-BI-Advanced-Ref/PBIX folder and save it to your local machine.
2. Open the pbix file.
3. Navigate to the **Sales Overview** report page
4. On the **Optimize** ribbon, select **Performance analyzer** — the pane opens on the right side of the report canvas
5. In the Performance analyzer pane, select **Start recording**
6. Select **Refresh visuals** to reload all visuals on the current page
7. Wait for all visuals to finish loading, then select **Stop**
8. Expand the entry for the **Table** visual (showing Year, Total Sales, and Sales YoY Growth)
9. Note the **DAX query time** (in milliseconds) for this visual — this is your baseline
   > **Note:** With this dataset, query times may be small (under 500 ms) — that's expected since the dataset isn't large. The goal is to learn the diagnostic process. Even a change from 80 ms to 30 ms demonstrates the optimization worked. If all timings appear identical, select **Clear**, then select **Refresh visuals** again to get uncached measurements.

## Task 2: Analyze the Slow DAX Query

1. In the Performance analyzer pane, expand the entry for the **Table** visual if it isn't already expanded
2. Select **Run in DAX query view** — Power BI Desktop opens DAX query view with the visual's generated query
3. Select **Run** to execute the query — the result grid shows `Total_Sales` and `Sales_YoY_Growth` for each fiscal year
4. Review the generated query:
   ```
   DEFINE
       VAR __DS0Core = 
           SUMMARIZECOLUMNS(
               ROLLUPADDISSUBTOTAL('Date'[Year], "IsGrandTotalRowTotal"),
               "Total_Sales", 'Sales'[Total Sales],
               "Sales_YoY_Growth", 'Sales'[Sales YoY Growth]
           )
       VAR __DS0PrimaryWindowed = 
           TOPN(502, __DS0Core, [IsGrandTotalRowTotal], 0, 'Date'[Year], 1)

   EVALUATE
       __DS0PrimaryWindowed

   ORDER BY
       [IsGrandTotalRowTotal] DESC, 'Date'[Year]
   ```

   > **Note:** This query is not the measure definition itself — Power BI generates it to populate the visual. `SUMMARIZECOLUMNS` groups the data by year, `ROLLUPADDISSUBTOTAL` adds the grand total row, and `TOPN` caps the row count. The measures (`[Total Sales]` and `[Sales YoY Growth]`) are referenced by name, but their formulas live in the model — not in this generated query.
5. Switch back to **Report** view. In the **Data** pane, expand the **Sales** table and select the **Sales YoY Growth** measure found under the Time intelligence folder — the formula bar shows the measure definition:
   ```
   Sales YoY Growth =
   DIVIDE(
       [Total Sales] - CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])),
       CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
   )
   ```
6. Look at the formula. The `CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))` expression appears twice — in the numerator and the denominator — but calculates the same value both times. This means the engine evaluates the prior-year sales calculation twice per row in the query, which is wasteful.

   Common inefficient patterns include:
   - **Repeated subexpressions**: the same `CALCULATE` evaluated multiple times without storing it in a `VAR`
   - **FILTER on a full table**: `FILTER(Sales, ...)` iterating every row instead of using a column predicate in `CALCULATE`
   - **COUNTROWS(FILTER(…))**: counting rows by iterating a filtered table instead of using `CALCULATE(COUNTROWS(...), ...)`

   The **Sales YoY Growth** measure has a repeated subexpression — the prior-year calculation is evaluated twice. In the next task, you fix this by storing it in a variable.

## Task 3: Optimize the DAX Measure

1. In Report view, select the **Sales YoY Growth** measure in the Data pane so its formula appears in the formula bar
2. Select all the text in the formula bar and replace it with the optimized version:
   ```
   Sales YoY Growth =
   VAR SalesPriorYear =
       CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
   RETURN
       DIVIDE([Total Sales] - SalesPriorYear, SalesPriorYear)
   ```

   > The `VAR` stores the prior-year result once. The `RETURN` expression references `SalesPriorYear` twice without recalculating it.
3. Press **Enter** to confirm the formula change
4. To verify the measure still returns the correct values, switch to **DAX query view**, open a new query tab, and run:
   ```
   EVALUATE
   SUMMARIZECOLUMNS(
       'Date'[Year],
       "YoY Growth", [Sales YoY Growth]
   )
   ```
5. Compare the results to what you saw earlier — the values should be the same (for example, FY2019 should still show approximately 0.7 and FY2020 approximately 0.18). The optimization changes speed, not results.

## Task 4: Examine Column Cardinality

1. Switch to **DAX query view** in Power BI Desktop
2. In a new query tab, enter the following query and select **Run**:
   ```
   DEFINE
       VAR _stats = COLUMNSTATISTICS()
   EVALUATE
       FILTER(_stats, NOT CONTAINSSTRING([Column Name], "RowNumber-"))
   ORDER BY [Cardinality] DESC
   ```

   > The result grid returns one row per column in the model, sorted by the number of distinct values. The `FILTER` excludes internal system columns that aren't part of your model.
3. Review the results — the columns with the highest cardinality appear at the top:
   - **SalesOrderNumber** in the Sales table has the highest cardinality (3,616) — nearly one distinct value per row, typical for transaction identifiers in fact tables
   - **Date** in the Date table is second (1,826) — the calendar table covers a wider date range than the actual sales data, so it has more distinct values than `OrderDate` (990) in the Sales table
   - **Cost** (1,430) and **Sales** (1,411) have high cardinality for numeric columns — many distinct decimal values; rounding to fewer decimal places is one way to reduce cardinality
   - **ResellerKey** (701) and **Reseller** (699) are nearly 1:1, which is expected for a dimension key and its label
4. Notice that fact table columns (Sales) dominate the top of the list. In a star schema, dimension tables stay small while fact tables drive memory consumption — this is why fact table optimization has the biggest impact

## Task 5: Verify the Improvements

1. Switch to **Report** view and open the Performance analyzer pane if it isn't already open
2. Select **Clear**, then select **Start recording**
3. Select **Refresh visuals** and wait for all visuals to finish loading
4. Select **Stop**
5. Expand the **Table** visual entry and compare its **DAX query time** to the baseline you recorded in Task 1
   > **Note:** The absolute difference may be small with this dataset. The important takeaway is the process: measure → diagnose → fix → verify.

## Task 6: Create a Slow Measure and Observe Performance

1. In the **Data** pane, expand the **Sales** table and create a new measure with the definition:
   ```
   Slow Measure =
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])) +
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
   ```
Note: The repeated use of `CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))` demonstrates a scenario where a `VAR` can be used to optimize performance by avoiding repeated subexpression evaluation.
2. Create a new Card visual in your report and add the Slow Measure to it to observe the performance.
3. Open the Performance analyzer pane if it isn't already open
4. Select **Clear**, then select **Start recording**
5. Select **Refresh visuals** and wait for all visuals to finish loading
6. Select **Stop**
7. Expand the **Table** visual entry and records its **DAX query time**
8. In the **Data** pane, expand the **Sales** table and create a new measure with the definition:
```
Optimized Measure =
VAR PriorYearSales = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales +
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales +
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales +
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales +
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales +
    PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales + PriorYearSales
```
9. Add the Optimized Measure to the Card visual and observe the improved performance compared to the Slow Measure.

---

## Lab A Summary

### What You Accomplished

✅ **Task 1**: Used Performance Analyzer to capture a baseline DAX query time for the Table visual
✅ **Task 2**: Exported and analyzed a slow DAX query, then located the repeated-subexpression pattern in the Sales YoY Growth measure
✅ **Task 3**: Optimized the measure using a `VAR` to evaluate the prior-year calculation only once
✅ **Task 4**: Used `COLUMNSTATISTICS()` to examine column cardinality and identify where memory consumption is highest
✅ **Task 5**: Re-ran Performance Analyzer to verify the DAX query time improved against the baseline
✅ **Task 6**: Created a Slow Measure and observed its performance compared to the Optimized Measure

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Performance Analyzer baselining | Task 1, 5 | [Use Performance Analyzer in Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-performance-analyzer) |
| DAX query view and generated queries | Task 2, 3 | [Optimize semantic model performance](https://learn.microsoft.com/training/modules/optimize-semantic-model-performance/) |
| Repeated subexpressions and `VAR` | Task 2, 3 | [DAX query performance best practices](https://learn.microsoft.com/power-bi/guidance/dax-variables) |
| Column cardinality with `COLUMNSTATISTICS()` | Task 4 | [Optimize semantic model performance](https://learn.microsoft.com/training/modules/optimize-semantic-model-performance/) |

### Recommended References

- [Optimize semantic model performance (Microsoft Learn module)](https://learn.microsoft.com/training/modules/optimize-semantic-model-performance/)
- [Use Performance Analyzer in Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-performance-analyzer)
- [DAX query performance best practices](https://learn.microsoft.com/power-bi/guidance/dax-variables)
- Source exercise: [Optimize semantic model performance — mslearn-fabric](https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/16-optimize-semantic-model-performance.html)

---

# Lab B: Row-Level Security, Dynamic RLS & Object Level Security

## Lab Introduction

In this lab, you will build static and dynamic Row-Level Security (RLS) roles, assign users to roles in the Power BI Service, and use Tabular Editor to apply Object Level Security (OLS).

This lab will guide you through the following tasks:

- Create static RLS roles filtered by region
- Create a dynamic RLS role using `USERPRINCIPALNAME()`
- Assign users to static roles in the Power BI Service
- Hide a column and a table from a role using Object Level Security

## Lab Scenario

In enterprise Power BI deployments, different users should only see data relevant to their role — a regional manager should see only their region's data, not the entire company's. Row-Level Security (RLS) enforces this at the data model level, regardless of which visual or page the user views.

## Architecture Overview

```
Row-Level Security & Object Level Security
===========================================

AdvancedSalesReport.pbix (Power BI Desktop)
├── Static Roles
│   ├── North Region  → Sales[Region] = "North"
│   └── South Region  → Sales[Region] = "South"
│
├── Dynamic Role
│   └── Dynamic RLS → UserMapping[UserEmail] = USERPRINCIPALNAME()
│         └── Relationship: UserMapping[Region] → Sales[Region]
│
                    ↓ (Publish)

Power BI Service — Dataset → Security
└── Assign users/test accounts to North Region / South Region roles

Tabular Editor 2 (connected via External Tools)
└── Roles → North Region
      ├── Sales[Unit Cost] → OLS Permission: None    (column hidden)
      └── UserMapping table → Permission: None        (table hidden)
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Create static RLS roles in Power BI Desktop |
| Task 2 | Create a dynamic RLS role with `USERPRINCIPALNAME()` |
| Task 3 | Assign users to roles in the Power BI Service |
| Task 4 | Apply Object Level Security with Tabular Editor |

---

## Task 1: Create Static RLS Roles (Power BI Desktop)

1. Open `AdvancedSalesReport.pbix` in Power BI Desktop
2. Go to **Modeling → Manage roles** and create a new role named `North Region`
3. In the role editor, select the **Sales** table and add the filter: `[Region] = "North"`
4. Create a second role `South Region` with filter `[Region] = "South"`
5. Test the roles: **Modeling → View as → North Region** — verify only North data is visible across all visuals
6. Exit the role view and return to normal mode

## Task 2: Create a Dynamic RLS Role (Power BI Desktop)

1. Add a **UserMapping** table to the model (via **Enter data**) with columns: `UserEmail` and `Region`, populated with sample user email addresses mapped to regions
2. Create a relationship between `UserMapping[Region]` and `Sales[Region]`
3. In **Manage roles**, create a new role named `Dynamic RLS` and apply the DAX filter to the **UserMapping** table:
   ```
   [UserEmail] = USERPRINCIPALNAME()
   ```
4. This ensures each user sees only rows where their email matches the UserMapping table — no static role per region needed
5. Test with **View as → Other user** and enter a sample email from the UserMapping table — verify the correct region data is returned

## Task 3: Assign Users to Roles in the Power BI Service

1. Publish the report to a shared workspace in the Power BI Service
2. Navigate to the **dataset → Security** in the Service
3. Assign colleagues or test accounts to the `North Region` and `South Region` static roles
4. Discuss: with dynamic RLS, no manual role assignment is needed per user — the DAX expression handles it automatically

## Task 4: Apply Object Level Security with Tabular Editor

1. Download and install **Tabular Editor 2** (free, open-source: [tabulareditor.com](https://tabulareditor.com)) if not already installed
2. In Power BI Desktop, go to **External Tools → Tabular Editor** to launch it connected to the open model
3. In Tabular Editor, expand the **Roles** node and select the `North Region` role
4. In the role properties, select the **Sales** table → find the `Unit Cost` column → set **OLS Permission** to **None** — this hides the column from users in this role
5. To hide an entire table: select the **UserMapping** table in the role and set its table permission to **None**
6. Save and close Tabular Editor — changes apply immediately to the open model in Desktop
7. Test with **Modeling → View as → North Region** — the `Unit Cost` column and **UserMapping** table should no longer appear in the field list
8. Discuss the key difference: **RLS hides rows** (data is filtered but columns remain visible); **OLS hides structure** (the field name itself is invisible regardless of filter context)

---

## Lab B Summary

### What You Accomplished

✅ **Task 1**: Created static RLS roles filtered by region and verified them with View as

✅ **Task 2**: Created a dynamic RLS role using `USERPRINCIPALNAME()` and a UserMapping table

✅ **Task 3**: Assigned users to static roles in the Power BI Service

✅ **Task 4**: Used Tabular Editor to hide a sensitive column and a table from a role with Object Level Security

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Static row-level security | Task 1 | [Row-level security in Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls) |
| Dynamic RLS with `USERPRINCIPALNAME()` | Task 2 | [Dynamic row-level security](https://learn.microsoft.com/power-bi/guidance/rls-guidance) |
| Role assignment in the Service | Task 3 | [Row-level security in Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls) |
| Object Level Security (OLS) | Task 4 | [Dynamic row-level security](https://learn.microsoft.com/power-bi/guidance/rls-guidance) |

RLS and OLS are complementary: use OLS to restrict sensitive columns and tables, RLS to filter sensitive rows. Always test both before publishing; untested security can expose or incorrectly restrict data.

### Recommended References

- [Row-level security in Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls)
- [Dynamic row-level security with USERPRINCIPALNAME](https://learn.microsoft.com/power-bi/guidance/rls-guidance)

---

# Lab C: Deployment Pipelines & Report Governance

> **License requirement:** Deployment Pipelines require workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a PPU workspace. If neither is available, this lab is trainer-led demo. Participants can replicate steps in their own environment afterward.

## Lab Introduction

In this lab, you will build a Dev → Test → Prod deployment pipeline and apply endorsement and governance settings to a published dataset.

This lab will guide you through the following tasks:

- Create a deployment pipeline with Development, Test, and Production stages
- Deploy content between stages and compare differences
- Apply endorsement, a sensitivity label, and review lineage for a published dataset

## Lab Scenario

Without a structured release process, Power BI developers often edit live production reports directly, risking breaking changes for end users. Deployment pipelines provide a formal Dev → Test → Prod promotion process, ensuring reports are validated before reaching consumers.

## Architecture Overview

```
Deployment Pipeline & Governance
===================================

Deployment Pipeline: Sales Report Pipeline
├── Development workspace   (active development)
├── Test workspace          ← Deploy to Test (Compare view)
└── Production workspace    ← Deploy to Production

                    ↓

Dataset in shared workspace
├── Endorsement: Promoted / Certified
├── Sensitivity label (e.g., Internal) → propagates to downstream reports
└── Lineage view → upstream sources / downstream reports
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Build a deployment pipeline (Dev → Test → Prod) |
| Task 2 | Apply report endorsement and governance settings |

---

## Task 1: Build a Deployment Pipeline (Trainer demo or hands-on with PPU)

1. In the Power BI Service, navigate to **Deployment pipelines** and click **Create a pipeline**
2. Name the pipeline (e.g., `Sales Report Pipeline`) and assign three stages: **Development**, **Test**, **Production**
3. Assign an existing workspace to the **Development** stage (this is where active development happens)
4. Create two additional workspaces for **Test** and **Production** and assign them to their respective stages
5. In the Development stage, click **Deploy to Test** — review the comparison view showing what has changed
6. Review the **Compare** button between stages — this shows differences in reports, datasets, and dashboards between stages
7. Deploy from Test to Production after review
8. Demonstrate how a dataset connection rule can be set per stage (e.g., point Production to a Production data source, not Development)

## Task 2: Apply Report Endorsement & Governance (hands-on with Pro)

1. In the Power BI Service, navigate to a published dataset in a shared workspace
2. Click **More options (…) → Settings → Endorsement**
3. Apply **Promoted** status to the dataset — discuss what this signals to report consumers
4. Discuss **Certified** status — this requires a tenant admin to enable and designate certifiers; it signals organizational trust and quality assurance
5. Apply a **sensitivity label** (e.g., Internal) to the dataset — observe that the label propagates downstream to reports built on it
6. Navigate to **Settings → Lineage view** for the workspace — explore upstream data sources and downstream reports built on shared datasets

---

## Lab C Summary

### What You Accomplished

✅ **Task 1**: Built a three-stage deployment pipeline and promoted content from Development to Production with a stage comparison

✅ **Task 2**: Applied endorsement, a sensitivity label, and reviewed lineage for a published dataset

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Deployment pipelines (Dev/Test/Prod) | Task 1 | [Get started with deployment pipelines](https://learn.microsoft.com/power-bi/deployment-pipelines/get-started-with-deployment-pipelines) |
| Stage-specific dataset connection rules | Task 1 | [Introduction to deployment pipelines](https://learn.microsoft.com/power-bi/developer/projects/projects-git) |
| Endorsement (Promoted/Certified) | Task 2 | [Endorse your content in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content) |
| Sensitivity labels and lineage | Task 2 | [Endorse your content in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content) |

### Recommended References

- [Introduction to deployment pipelines](https://learn.microsoft.com/power-bi/developer/projects/projects-git)
- [Get started with deployment pipelines](https://learn.microsoft.com/power-bi/deployment-pipelines/get-started-with-deployment-pipelines)
- [Endorse your content in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content)

---

# Lab D: AI Visuals & Copilot Capabilities

> **License note:** Built-in AI visuals (anomaly detection, forecasting, Smart Narrative, Q&A) are available in Power BI Desktop and Service to all participants. Copilot requires **any Fabric F SKU (F2+) or PPU** — trainer-led demo if unavailable.

## Lab Introduction

In this lab, you will use Power BI's built-in AI visuals to detect anomalies and forecast trends, then use Copilot to summarise a report and generate a new report page from natural language prompts.

This lab will guide you through the following tasks:

- Enable anomaly detection and forecasting on a line chart, and add Smart Narrative and Q&A visuals
- Use Copilot in the Power BI Service and Desktop to summarise and generate report content

## Lab Scenario

Power BI's built-in AI capabilities allow report consumers to discover insights they might otherwise miss — detecting unexpected spikes, forecasting future trends, and asking questions in plain English. Copilot further accelerates report creation and exploration by generating summaries and visuals from natural language prompts.

## Architecture Overview

```
AI Visuals & Copilot
=======================

AdvancedSalesReport.pbix — Line Chart (Sales Amount over time)
├── Analytics pane → Anomaly detection (sensitivity)
├── Analytics pane → Forecast (3-month horizon + confidence interval)
├── Smart Narrative visual → auto-generated page summary
└── Q&A visual → natural language question → auto-generated visual

                    ↓ (Publish to Fabric/Premium workspace)

Power BI Service — Copilot
├── "Summarise this report page"
├── "What are the top-performing regions?"
└── Power BI Desktop Copilot → "Create a report page showing sales trends by category"
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Use built-in AI visuals in Power BI Desktop |
| Task 2 | Use Copilot in the Power BI Service and Desktop |

---

## Task 1: Use Built-in AI Visuals in Power BI Desktop (hands-on)

1. Open `AdvancedSalesReport.pbix` and navigate to a page with a **Line Chart** showing Sales Amount over time
2. With the line chart selected, go to **Add further analysis to your visual** (the analytics pane, magnifying glass icon) and enable **Anomaly detection** — adjust sensitivity and observe flagged anomalies
3. In the same pane, enable **Forecast** — set a forecast length of 3 months and observe the projected trend and confidence interval
4. Add a **Smart Narrative** visual (from the Visualizations pane) — Power BI auto-generates a plain-English text summary of the report page; observe how it updates as slicers are applied
5. Add a **Q&A** visual — type natural language questions such as "total sales by region as a bar chart" and observe the auto-generated visual response

## Task 2: Use Copilot in the Power BI Service (trainer demo or hands-on with Fabric/Premium capacity)

1. Open the published report in a **Fabric or Premium capacity** workspace
2. Click the **Copilot** button in the report toolbar
3. Ask Copilot to: *"Summarise this report page"* — review the generated narrative summary
4. Ask Copilot: *"What are the top-performing regions?"* — observe the AI-driven insight extraction
5. In Power BI Desktop with Copilot enabled, ask Copilot to *"Create a report page showing sales trends by category"* — observe the auto-generated page layout
6. Discuss responsible use: Copilot insights should be validated before being acted on; understand what data is sent to the AI service

---

## Lab D Summary

### What You Accomplished

✅ **Task 1**: Enabled anomaly detection and forecasting on a line chart, and added Smart Narrative and Q&A visuals

✅ **Task 2**: Used Copilot in the Power BI Service and Desktop to summarise a report and generate a new report page

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Anomaly detection and forecasting | Task 1 | [Apply insights to explain fluctuations in visuals](https://learn.microsoft.com/power-bi/create-reports/desktop-insights) |
| Smart Narrative and Q&A | Task 1 | [AI visuals in Power BI](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-anomaly-detection) |
| Copilot in the Power BI Service and Desktop | Task 2 | [Overview of Copilot for Power BI](https://learn.microsoft.com/power-bi/create-reports/copilot-introduction) |

Copilot accelerates report creation but requires review — AI-generated insights can reflect biases or errors in the underlying data. Copilot in Power BI sends query results (not raw data) to Azure OpenAI; tenant admins control Copilot enablement.

### Recommended References

- [Apply insights to explain fluctuations in visuals](https://learn.microsoft.com/power-bi/create-reports/desktop-insights)
- [Overview of Copilot for Power BI](https://learn.microsoft.com/power-bi/create-reports/copilot-introduction)
- [AI visuals in Power BI](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-anomaly-detection)
