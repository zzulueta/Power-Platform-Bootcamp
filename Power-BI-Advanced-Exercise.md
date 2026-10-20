---
  title: 'Power BI Advanced: Hands-On Lab Exercises'
  module: 'Power BI Advanced Lab'
---

# Power BI Advanced — Hands-On Lab Exercises

## Lab Introduction

These exercises accompany the Power BI Advanced training session. Each lab can be completed independently or in sequence and builds on a shared file covering performance tuning, security, version control, deployment, governance, and AI-assisted analysis.

This set of exercises covers the following labs:

- **Lab A**: Performance Tuning & DAX Optimisation
- **Lab B**: Row-Level Security, Dynamic RLS & Object Level Security
- **Lab C**: Version Control with PBIP & Git Integration
- **Lab D**: Deployment Pipelines & Report Governance
- **Lab E**: AI Visuals & Copilot Capabilities

---

## Pre-requisites

- **Power BI Pro** license assigned to your account and the ability to sign in at [app.powerbi.com](https://app.powerbi.com)
- **Power BI Desktop** installed, November 2025 or newer ([aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore)) — UI elements may vary slightly depending on your version
- A shared workspace in the Power BI Service that you can publish to
- **Lab C** (Version Control) requires **Git** and **Visual Studio Code** for the local exercises; no Fabric capacity is needed. The pull-request exercise uses an organisation-approved private **GitHub** repository. Workspace Git integration additionally requires a **Fabric capacity** (or an existing supported Power BI Premium capacity), workspace **Admin** access, repository access, and the relevant tenant settings enabled — **PPU alone is not sufficient**; use a trainer-led demo for this task if unavailable
- **Lab D** (Deployment Pipelines) requires workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a **PPU** workspace — trainer-led demo if neither is available
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
| Task 6 | Create a Slow Measure and observe its performance compared to the Optimized Measure |


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
> Note: The repeated use of `CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))` demonstrates a scenario where a `VAR` can be used to optimize performance by avoiding repeated subexpression evaluation.

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

10. Other DAX queries to compare:

   Table Filter vs. KEEPFILTERS example
   ```
   Red Sales = 
   CALCULATE(Sales[Total Sales],FILTER('Product', 'Product'[Color] = "Red"))
   ```
   vs.
   ```
   Red Sales =
   CALCULATE(Sales[Total Sales],KEEPFILTERS('Product'[Color] = "Red"))
   ```
   > Note: 
   > First Query: FILTER scans every row of Product and builds an entire filtered table before CALCULATE can use it.
   > Second Query: The Boolean expression is passed directly as a filter argument. VertiPaq's in-memory column store is optimized to filter this way — no intermediate table is materialized. (KEEPFILTERS preserves any existing filter on Color rather than overwriting it.)

   Count with FILTER example
   ```
   High Value Orders =
   COUNTROWS(
       FILTER(Sales, Sales[Total Sales] > 1000)
   )
   ```
   vs.
   ```
   High Value Orders =
   CALCULATE(
       COUNTROWS(Sales),
       Sales[Total Sales] > 1000
   )
   ```
   > Note: 
   > First Query: FILTER builds a filtered copy of Sales row by row before COUNTROWS can count it.
   > Second Query: COUNTROWS counts the base table directly, while the condition is applied as a native CALCULATE filter argument — no intermediate FILTER table required.
   
  
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

---

# Lab B: Row-Level Security, Dynamic RLS & Object Level Security

## Lab Introduction

In this lab, you will build static and dynamic Row-Level Security (RLS) roles, assign users to roles in the Power BI Service, and use Tabular Editor to apply Object Level Security (OLS).

This lab will guide you through the following tasks:

- Create static RLS roles filtered by region
- Create a dynamic RLS role using `USERPRINCIPALNAME()`
- Create a role that applies dynamic RLS role using `USERPRINCIPALNAME()` and hides a column using Object Level Security
- Assign users to different roles in the Power BI Service

## Lab Scenario

In enterprise Power BI deployments, different users should only see data relevant to their role — a regional manager should see only their region's data, not the entire company's. Row-Level Security (RLS) enforces this at the data model level, regardless of which visual or page the user views. In addition, Object Level Security (OLS) allows you to hide specific columns or tables from certain roles, providing an extra layer of data protection.

## Architecture Overview

```
Row-Level Security & Object Level Security
===========================================

AdvancedSalesReport.pbix (Power BI Desktop)
├── Static Roles
│   ├── NW Region  → Region[Region] = "Northwest"
│   └── SW Region  → Region[Region] = "Southwest"
│
├── Dynamic RLS Role
│   └── Dynamic RLS
│         └── Performance[UPN] = USERPRINCIPALNAME()
│         └── Salesperson[UPN] = USERPRINCIPALNAME()
│
├── UPN Role
│   └── Dynamic RLS
│         └── Performance[UPN] = USERPRINCIPALNAME()
│         └── Salesperson[UPN] = USERPRINCIPALNAME()
│   └── No access to Sales[Cost] and Sales[Cost Price] (column hidden)
│

                    ↓ (Publish)

Power BI Service — Semantic Model → Security
└── Assign users/test accounts to roles
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Create static RLS roles in Power BI Desktop |
| Task 2 | Create a dynamic RLS role with `USERPRINCIPALNAME()` |
| Task 3 | Apply Object Level Security with Tabular Editor |
| Task 4 | Assign users to roles in the Power BI Service |


---

## Task 1: Create Static RLS Roles (Power BI Desktop)

1. Open `AdvancedSalesReport.pbix` in Power BI Desktop and go to the **Model** view.

2. Select the Region to SalespersonRegion relationship and disable Apply security filter in both directions.

3. Click Save.

4. Go to **Manage roles** and create a new role named `NW Region`

5. In the role editor, select the **Region** table and add the filter: `Region = "Northwest"`

6. Create a second role `SW Region` with filter `Region = "Southwest"`

7. Click **Save** to save the roles. Then Close the role editor.

8. Test the roles by selecting **View as → NW Region**.

9. Go to the **Report** view, select Page 3, and verify only Northwest data is visible across all visuals

10. Exit the role view by selecting **Stop viewing** to return to normal mode

11. Verify as well that the SW Region role correctly filters the data to only show Southwest region information.

## Task 2: Create a Dynamic RLS Role (Power BI Desktop)

1. Go back to the **Model** view

2. In **Manage roles**, create a new role named `Dynamic RLS`.

3. In the role editor, select the **Salesperson** table and select **Switch to DAX editor**

4. Apply the DAX filter:
   ```
   [UPN] = USERPRINCIPALNAME()
   ```

5. Select the **Salesperson (Performance)**, select **Switch to DAX editor**, and apply the same DAX filter:
   ```
   [UPN] = USERPRINCIPALNAME()
   ```

6. Select Save to save the role and close the role editor.

7. To test the Dynamic RLS role, select **View as**. 
    - Select Other user and enter a sample email: `michael-blythe@adventureworks.com`
    - Select Dynamic RLS
    - Click **OK** to apply the role

8. Go to the **Report** view, select Page 1, and verify only Michael Blythe's data is visible across all visuals for the Dynamic RLS role

9. Exit the role view by selecting **Stop viewing** to return to normal mode

## Task 3: Apply Object Level Security with Tabular Editor

1. In Power BI Desktop, select **Transform data** to open the Power Query Editor.

2. Rename the query `Salesperson (Performance)` to `Performance`

3. Select **Close & Apply** to close the Power Query Editor and apply the changes.

4. In Power BI Desktop, go to **TMDL** in the left pane to go to the Tabular Editor.

5. In Tabular Editor, enter the following formula:
    ```
    createOrReplace  
        role UPN
            modelPermission: read

            tablePermission Salesperson = Salesperson[UPN] = USERPRINCIPALNAME()
            tablePermission Performance = Performance[UPN] = USERPRINCIPALNAME()

            tablePermission Sales
                columnPermission Cost
                    metadataPermission: none
                columnPermission 'Cost Price'
                    metadataPermission: none
    ```

6. Select **Apply** to apply the changes in Tabular Editor.

7. Go to the Model view and select **View as** to test the UPN role with Object Level Security applied.
    - Select Other user and enter a sample email: `michael-blythe@adventureworks.com`
    - Select UPN
    - Click **OK** to apply the role

8. Go to the **Report** view, select Page 1, and verify that the first table shows an issue. Verify that the second table works correctly and shows only the user's data.

9. Select **See details** to view the specific issue caused by Object Level Security in the first table. The error indicates that the user does not have access to the (Sales) Profit, and (Sales) Profit Margin. 

10. Go to the Data section of the Report view and verify that the sensitive columns are hidden for the UPN role.

11. Exit the role view by selecting **Stop viewing** to return to normal mode.

12. Verify that the Table now shows the data correctly for the user after exiting the role view and that the Profit and Profit Margin columns are visible again.

## Task 4: Assign Users to Roles in the Power BI Service

1. Go to the Power Query Editor in Power BI Desktop.

2. Select the Performance query.

3. Right-click the UPN column header to edit it.

3. Select **Replace Values**.

4. Value to Find: rachel-valdez@adventureworks.com

5. Replace With: the new email you want (select a colleague or test account).

6. Click OK then select Close & Apply.

7. Create a workspace in Power BI and provide the colleague or the test account Viewer access.

8. Publish the report to the shared workspace in the Power BI Service

9. Navigate to the **Security** settings of the semantic model in the Power BI Service

10. Assign colleague or the test account to the different roles:
    - No role assigned
    - `NW Region` and `SW Region`
    - Dynamic RLS
    - UPN 

11. Verify that the user has the correct access according to the roles assigned.

---

## Lab B Summary

### What You Accomplished

✅ **Task 1**: Created static RLS roles filtered by region and verified them with View as

✅ **Task 2**: Created a dynamic RLS role using `USERPRINCIPALNAME()` and a UserMapping table

✅ **Task 3**: Used Tabular Editor to hide a sensitive column and a table from a role with Object Level Security

✅ **Task 4**: Assigned users to roles in the Power BI Service



### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Static row-level security | Task 1 | [Secure data access in Power BI](https://learn.microsoft.com/en-us/training/modules/row-level-security-power-bi/) |
| Dynamic RLS with `USERPRINCIPALNAME()` | Task 2 | [Dynamic row-level security](https://learn.microsoft.com/power-bi/guidance/rls-guidance) |
| Object Level Security (OLS) | Task 3 | [Object-level security in Power BI](https://learn.microsoft.com/power-bi/enterprise/service-admin-ols) |
| Role assignment in the Service | Task 4 | [Row-level security in Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls) |

RLS and OLS are complementary: use OLS to restrict sensitive columns and tables, RLS to filter sensitive rows. Always test both before publishing; untested security can expose or incorrectly restrict data.


---

# Lab C: Version Control with PBIP & Git Integration

## Lab Introduction

In this lab, you will save a Power BI report as a Power BI Project (PBIP), track its report and semantic model definitions in Git, and review changes before merging them. You will also practise reverting a change without deleting history. Where capacity is available, you will connect a development workspace to the repository, commit a workspace change to Git, and update the workspace and Desktop from `main`.

This lab will guide you through the following tasks:

- Save a PBIX as a PBIP and inspect its text-based project structure
- Create a local Git repository and commit a clean baseline
- Make an isolated change on a feature branch and inspect the diff
- Review and merge changes through a pull request on GitHub
- Revert a committed change while preserving history
- Connect a development workspace to Git, commit a workspace change, and update from `main`

## Lab Scenario

Your team currently shares files named `AdvancedSalesReport_v2.pbix` and `AdvancedSalesReport_FINAL.pbix`. It is difficult to tell which version is current or whether a change affects a visual, a measure, or security. You will replace filename-based versioning with a Git history of readable project definitions. Lab D then builds on this foundation by promoting approved content through deployment stages.

## Architecture Overview

```text
AdvancedSalesReport.pbix
    |
    | Save as Power BI Project
    v
SalesVersionControl\
|-- AdvancedSalesReport.pbip          (project entry point)
|-- AdvancedSalesReport.Report\       (report definition)
|-- AdvancedSalesReport.SemanticModel\ (model definition)
`-- .gitignore                       (exclude local settings and data cache)
    |
    | VS Code + Git: baseline -> feature branch -> diff -> review -> merge
    v
Approved private repository: main
    |
    | Fabric Git integration: Update from Git / Commit to Git
    v
Sales Dev workspace
    |
    | Lab D: deployment pipeline
    v
Sales Test workspace -> Sales Production workspace
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Save and inspect a Power BI Project |
| Task 2 | Initialise Git and commit a baseline without local data caches |
| Task 3 | Use a feature branch and inspect report and model diffs |
| Task 4 | Review and merge a change |
| Task 5 | Revert a change with an auditable commit |
| Task 6 | Synchronise a development workspace with Git |

---

## Task 1: Save the Report as a Power BI Project

1. Open [AdvancedSalesReport.pbix](./Power-BI-Advanced-Ref/PBIX/AdvancedSalesReport.pbix), or your copy from the previous labs, in Power BI Desktop.

2. Note the current Sales Overview page title, the **Total Sales** measure's description, and a displayed sales total. These are your baseline checks.

3. Select **File → Save as → Browse this device** and choose **Power BI project files (.pbip)**. Save the file in your local drive under a new folder called `SalesVersionControl`. 

4. In File Explorer, verify that the folder contains the `.pbip` file, `.Report` and `.SemanticModel` folders, and a `.gitignore`. Keep the original PBIX as a backup outside the new project folder.

5. Close Power BIDesktop, reopen `AdvancedSalesReport.pbip`, and verify that the report, sales total, and any existing security role definitions are still present. Do not rename or delete anything.

> **Checkpoint:** PBIP is a project structure, not just a renamed PBIX. Commit the report and semantic model definitions together; the `.pbip` pointer alone cannot reproduce the report. A fresh clone without the ignored data cache will need a data refresh.

## Task 2: Initialise Git and Commit a Baseline

1. Install [Visual Studio Code](https://code.visualstudio.com/) if needed, then open the **SalesVersionControl** folder in VS Code.

2. Open **Source Control → Initialise Repository**. This must be a separate lab repository.

3. Verify that the initial branch is named `main`; use **Git: Rename Branch** from the Command Palette if necessary.

4. Open `.gitignore` and confirm it includes the default PBIP exclusions below. Add them if missing, before staging any files:

   ```gitignore
   **/.pbi/localSettings.json
   **/.pbi/cache.abf
   ```

5. Review the untracked files. Confirm that `cache.abf`, `localSettings.json`, and the backup PBIX are not being staged. 

6. Open **Terminal → New Terminal** in the **SalesVersionControl** repository root and check your Git author identity:

   ```shell
   git config --get user.name
   git config --get user.email
   ```

   If either value is missing or incorrect, run the following with your approved name and email. These settings apply only to this repository; skip them if the existing values are correct.

   ```shell
   git config --local user.name "Your Name"
   git config --local user.email "your.name@example.com"
   ```

7. In the same terminal, run the following to stage all project files at once, respecting the `.gitignore` rules:

   ```shell
   git status --short
   git add .
   git diff --cached --stat
   ```

   Press **q** if the diff viewer opens—no Enter needed. Review the staged file list: include the `.pbip`, report and semantic model definitions, and `.gitignore`. Then create the local baseline commit (this saves locally; it does not push to GitHub):

   ```shell
   git commit -m "Add AdventureWorks PBIP baseline"
   ```
8. Verify that the commit appears in VS Code's **Source Control Graph** and the working tree is clean.

> **Checkpoint:** You have one baseline commit containing the project definitions, with no cached model data or machine-specific local settings. Do not ignore the entire `.pbi` folder: some files in it are shared project metadata.

## Task 3: Make a Change on a Feature Branch and Review the Diff

1. With Power BI Desktop closed, create and switch to a feature branch in the terminal so your changes stay off `main`:

   ```shell
   git switch -c feature/sales-title
   git branch --show-current
   ```

   The second command should return `feature/sales-title`.

2. Open the PBIP in Power BI Desktop and go to **Sales Overview**.

3. Add a text box and set its text to `Sales Overview - Version Control Lab`. Place it prominently on the top of the report page with font size of 24.

4. In **Model** view, select the **Total Sales** measure and set its **Description** property to `AdventureWorks sales total reviewed in the version control lab`. Do not change its DAX expression.

5. Save the file and close Power BI Desktop.

6. In VS Code **Source Control**, open the changed files to view the before-and-after diff:
   - Locate the title change in the report definition.
   - Locate the description change in the semantic model's TMDL files.
   - Review any additional Desktop-generated changes

7. Stage the reviewed changes and commit them in the terminal:

   ```shell
   git add .
   git commit -m "Created Sales Overview Page title and measure description"
   ```

> **Checkpoint:** The feature branch contains a reviewable report change and a model metadata change. The sales calculation still produces the baseline result.

## Task 4: Review and Merge the Change

### GitHub pull-request path

1. Create an empty organisation-approved **private** repository on GitHub (no README, `.gitignore`, or license), then copy its URL. In the terminal, switch to `main`, connect the remote, and push the baseline:

   ```shell
   git switch main
   git remote add origin <repository-url>
   git push -u origin main
   ```

2. Push the feature branch to the same repository:

   ```shell
   git push -u origin feature/sales-title
   ```

3. In GitHub, open the repository's **Pull requests** tab and select **Compare & pull request** in the banner for `feature/sales-title`. If the banner is not shown, select **New pull request** and choose the same branches.
   - Confirm **base** is `main` and **compare** is `feature/sales-title`.
   - Enter the title `Create Sales Overview title and measure description` and select **Create pull request**.

4. Open the **Files changed** tab. Confirm the diff contains the intended title and measure description changes and no secrets, local settings, or data cache.

5. Merge the pull request on GitHub: Go to the Conversation tab, select **Merge pull request**, then **Confirm merge**. 

6. Update your local files in VS Code. A merge on GitHub does not automatically update your local files. In the terminal, run:

   ```shell
   git switch main
   git pull
   git status
   ```

   `git status` should report a clean working tree.

7. Reopen the PBIP and verify the new title and measure description, with the sales total unchanged. Close Desktop again before Task 5.

> **Checkpoint:** `main` contains the approved change, and you have verified the merged project in Power BI Desktop. Fabric also supports Azure DevOps repositories; the hosted steps here use GitHub consistently to keep the exercise focused.

## Task 5: Revert a Change Without Deleting History

1. With Desktop closed and a clean working tree, create a branch named `revert/sales-title` from the updated `main`:

   ```shell
   git switch main
   git pull
   git switch -c revert/sales-title
   ```

2. List recent commits:

   ```shell
   git --no-pager log --oneline -5
   ```

3. In the `git log` output from step 2, find the commit that made your title and description changes. Each line starts with a short commit hash followed by its message. Skip the line that starts with **Merge pull request**, which only records the pull request merge, and choose the line with the message you committed in Task 3. Copy that line's hash (the first 7 characters).

   For example, in this output the commit to revert is `6e4e743`:

   ```text
   63a27a9 (HEAD -> revert/sales-title, origin/main, main) Merge pull request #1 from <user>/feature/sales-title
   6e4e743 (origin/feature/sales-title, feature/sales-title) Update Sales Overview title and measure description
   94bc170 Add AdventureWorks PBIP baseline
   ```

   Your hashes will differ. If you used **Squash and merge**, there is no separate merge line: choose the top commit that has your change.

4. Run `git revert`, replacing `<change-commit-sha>` with the hash you copied in step 3. For the example above, the command is `git revert --no-edit 6e4e743`:

   ```shell
   git revert --no-edit <change-commit-sha>
   ```

   Git creates a new commit that undoes that change. If Git reports a conflict, stop and review it with the trainer; do not discard files or use a hard reset to bypass the conflict.

5. Inspect the new revert commit, then reopen the PBIP. Verify the title and measure description match the baseline from Task 1 (or the added title text box is gone), and the sales total remains unchanged.

   ```shell
   git --no-pager show --stat HEAD
   ```

6. Close Power BI Desktop, then push the revert branch to GitHub. The revert commit exists only on your computer until you push it, and `main` is unchanged:

   ```shell
   git push -u origin revert/sales-title
   ```

7. In GitHub, open the repository's **Pull requests** tab and select **Compare & pull request** in the banner for `revert/sales-title`. If the banner is not shown, select **New pull request** and choose the same branches.
   - Confirm **base** is `main` and **compare** is `revert/sales-title`.
   - Enter the title `Revert Sales Overview title and measure description` and select **Create pull request**.
   - Open the **Files changed** tab and confirm the diff reverses your Task 3 changes and contains nothing else.

8. Merge the pull request on GitHub: go to the **Conversation** tab, select **Merge pull request**, then **Confirm merge**. Follow any repository review requirements before merging. This is the step that puts the revert on `main`.

9. Update your local files. A merge on GitHub does not automatically update your computer. In the terminal, run:

   ```shell
   git switch main
   git pull
   ```

10. Verify that history contains both the original change and its revert, and that the working tree is clean:

    ```shell
    git --no-pager log --oneline -5
    git status
    ```

> **Checkpoint:** Recovery created a new commit rather than erasing history. Reverting definitions does not roll back source data, refreshes, permissions, or other service-side configuration.

## Task 6: Connect a Development Workspace to Git (Capacity Required)

1. Create a new development workspace named **Sales Dev**. The workspace must have a Fabric capacity or an existing supported Power BI Premium capacity assigned, and you must be its **Admin**.

2. Create a Personal Access Token (PAT) in GitHub.
    - Go to the [GitHub Personal Access Tokens page](https://github.com/settings/personal-access-tokens)
    - Select **Fine-grained tokens** in the left-hand menu.
    - Select **Generate new token**, provide the following:
        - Token name: Sales Dev PAT
        - Expiration: Select an appropriate expiration
        - Repository access: Select the repository containing the project definitions
        - Permissions: Contents: Read and write
    - Select **Generate token** to create the PAT.
    - Select **Generate token** again.
    - Copy the generated token and store it securely; you will need it to connect the workspace to Git.

3. Head back to the Power BI Service. In **Sales Dev → Workspace settings → Git integration**, select **GitHub**.

4. Select **+ Add account**. Enter an appropriate display name, the PAT you generated, and the repository URL containing the project definitions, then select **Add**.

5. Select the created account and select **Connect**.

6. Select the main branch then select **Connect and sync**. 

7. Verify that the report and semantic model appear with **Synced** status. 

8. Select your Semantic Model and go to **Settings → Data access → Data source credentials**. Configure the necessary credentials for the data sources used in the semantic model. Refresh the semantic model to ensure the credentials are correctly applied.

9. Open the report in the Service, go to the Sales Overview page, enter Editing mode, add a text box containing `Git Integration Check`, and select Save. 

10. Go back to the Workspace and select **Source control → Changes** pane. 
    - Select the changed report
    - Enter `Add workspace Git verification text` in the Commit message
    - Select **Commit to new branch**. 
    - Enter a branch name, such as `feature/git-verification`, and select **Create**.

11. In GitHub, open the repository's **Pull requests** tab and select **Compare & pull request** in the banner for `feature/git-verification`. If the banner is not shown, select **New pull request** and choose the same branches.
   - Confirm **base** is `main` and **compare** is `feature/git-verification`.
   - Enter the title `Add workspace Git verification text` and select **Create pull request**.
   - Open the **Files changed** tab and confirm the diff contains your changes and nothing else.

12. Merge the pull request on GitHub: go to the **Conversation** tab, select **Merge pull request**, then **Confirm merge**. Follow any repository review requirements before merging. This is the step that puts the change on `main`.

13. Update your local files. A merge on GitHub does not automatically update your computer. Go back to your VS Code. In the terminal, run:

   ```shell
   git switch main
   git pull
   ```

14. Open the report in Desktop and verify that the `Git Integration Check` text box is present.

15. Go back to your Power BI workspace and update it from `main`. A Git merge does not automatically update workspace items.
    - Select **Source control** (it shows a pending update count) and open the **Updates** tab.
    - Confirm the changed report is listed, then select **Update all** and confirm the update.
    - Wait for the update to complete, then confirm the report's **Git status** shows **Synced**.
    - Open the report, go to the Sales Overview page, and verify the `Git Integration Check` text box is present.

> **Checkpoint:** You connected the **Sales Dev** workspace to `main` and made a change in the workspace, committed it to a new branch, and merged it through a pull request. You then pulled it to Desktop (**workspace → Git → Desktop**) and updated the workspace from `main` (**Git → workspace**). Git changes only move into the workspace or your computer when you update or pull them. If the workspace has uncommitted edits and Git has incoming changes, resolve them before updating; do not blindly overwrite either side. 

---

## Lab C Summary

### What You Accomplished

✅ **Task 1**: Saved the sample report as a PBIP and inspected its report and semantic model definitions

✅ **Task 2**: Committed a clean baseline while excluding local settings and the data cache

✅ **Task 3**: Used a feature branch and inspected readable report and model diffs

✅ **Task 4**: Pushed the baseline and feature branch to GitHub, reviewed the change in a pull request, merged it, and pulled the result into your local `main`

✅ **Task 5**: Reverted the committed change with a new revert commit, merged it through a pull request, and verified that history keeps both commits

✅ **Task 6**: Connected a development workspace to Git, committed a workspace change to a branch, merged it through a pull request, and updated both Desktop and the workspace from `main`

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| PBIP project structure and cache exclusions | Task 1, 2 | [Power BI Desktop projects](https://learn.microsoft.com/power-bi/developer/projects/projects-overview) |
| TMDL and semantic model definitions | Task 1, 3 | [Power BI Desktop project semantic model folder](https://learn.microsoft.com/power-bi/developer/projects/projects-dataset) |
| Git commits, branches, pull requests, and reverts | Task 2–5 | [Power BI Desktop projects Git integration](https://learn.microsoft.com/power-bi/developer/projects/projects-git) |
| Workspace connection, commits, and updates | Task 6 | [Get started with Git integration](https://learn.microsoft.com/fabric/cicd/git-integration/git-get-started) |


---

# Lab D: Deployment Pipelines & Report Governance

> **License requirement:** Deployment Pipelines require workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a PPU workspace. If neither is available, this lab is trainer-led demo. Participants can replicate steps in their own environment afterward.

## Lab Introduction

In this lab, you will build a Dev → Test → Prod deployment pipeline and apply endorsement and governance settings to a published dataset.

This lab will guide you through the following tasks:

- Create a deployment pipeline with Development, Test, and Production stages
- Deploy content between stages and compare differences
- Apply endorsement, a sensitivity label, and review lineage for a published dataset

## Lab Scenario

Without a structured release process, Power BI developers often edit live production reports directly, risking breaking changes for end users. Deployment pipelines provide a formal Dev → Test → Prod promotion process, ensuring reports are validated before reaching consumers. Lab C established version history and review; this lab uses the approved content as the starting point for release management.

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
3. Assign **Sales Dev** from Lab C to the **Development** stage. If you completed only the local Git path or skipped Lab C, first publish the approved project or sample PBIX to a development workspace that meets this lab's capacity requirements.
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

## Lab D Summary

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

# Lab E: AI Visuals & Copilot Capabilities

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

## Lab E Summary

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
