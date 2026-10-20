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
- **Lab E**: Sharing Semantic Models
- **Lab F**: AI Visuals & Preparing a Semantic Model for AI

---

## Pre-requisites

- **Power BI Pro** license assigned to your account and the ability to sign in at [app.powerbi.com](https://app.powerbi.com)
- **Power BI Desktop** installed, November 2025 or newer ([aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore)) — UI elements may vary slightly depending on your version
- A shared workspace in the Power BI Service that you can publish to
- **Lab C** (Version Control) requires **Git** and **Visual Studio Code** for the local exercises; no Fabric capacity is needed. The pull-request exercise uses an organisation-approved private **GitHub** repository. Workspace Git integration additionally requires a **Fabric capacity** (or an existing supported Power BI Premium capacity), workspace **Admin** access, repository access, and the relevant tenant settings enabled — **PPU alone is not sufficient**; use a trainer-led demo for this task if unavailable
- **Lab D** (Deployment Pipelines) requires workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a **PPU** workspace — trainer-led demo if neither is available
- **Lab E** (Sharing Semantic Models) requires the **Sales Production** workspace from Lab D (or any workspace containing the published semantic model), **Admin or Member** access to that workspace, and a **second user or test account** with a Power BI Pro or PPU license whose email matches the UPN value you set in Lab B, Task 4 (the `UPN` role is used in Lab E). Free-license users can only build on a shared model hosted on **Fabric F64 or higher** (or Premium) capacity
- **Lab F** (AI Visuals & Preparing a Semantic Model for AI) uses the `AdvancedSalesReport.pbix` file in Power BI Desktop. The decomposition tree, key influencers, and custom smart narrative (Tasks 1–3) work for all participants. Testing the **Prep data for AI** changes with **Copilot** (Tasks 4–6) requires a Copilot-enabled workspace on **any Fabric F SKU (F2+)** or **PPU** — trainer-led demo if unavailable. Marking the model **Approved for Copilot** (Task 7) requires publishing to the Power BI service

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

## Task 1: Build a Deployment Pipeline

1. In the Power BI Service, create two additional workspaces for **Sales Test** and **Sales Production** and assign them to their respective stages

2. Navigate to **Deployment pipelines** and click **+ New pipeline**

3. Name the pipeline (e.g., `Sales Report Pipeline`), click Next, and assign three stages: **Development**, **Test**, **Production**, then click **Create**.

4. Select **Sales Dev** from Lab C to the **Development** stage then click **Assign a workspace**.

5. Do the same for the **Test** and **Production** stages by assigning **Sales Test** and **Sales Production** workspaces respectively.

6. Select the **Compare** button between the Development and Test stages — this shows differences in reports, datasets, and dashboards between stages

7. Click **Deploy** to move content from Development to Test.

8. Demonstrate how a dataset connection rule can be set per stage (e.g., point Production to a Production data source, not Development)

9. Repeat the comparison and deployment process to move content from Test to Production.

10. Provide Viewer access to the deployed content in the Test and Production workspaces to demonstrate role-based access control.

11. View Deployment history to verify that the content has been successfully deployed to the Test and Production stages.

## Task 2: Apply Report Endorsement & Governance

1. In the Power BI Service, navigate to the Sales Production workspace and select the Semantic Model.

2. Click **More options (…) → Settings → Endorsement**

3. Apply **Promoted** status to the dataset — discuss what this signals to report consumers

4. Discuss **Certified** status — this requires a tenant admin to enable and designate certifiers (Admin portal → Tenant settings → Certification); it signals organizational trust and quality assurance

5. Go to **Sensitivity label** to apply a Sensitivity label (e.g., Internal) to the dataset — observe that the label propagates downstream to reports built on it

6. Navigate to **Lineage view** for the workspace to explore upstream data sources and downstream reports built on shared datasets

7. Go to **OneLake catalog** and navigate to Endorsed items to see the Promoted semantic model.

---

## Lab D Summary

### What You Accomplished

✅ **Task 1**: Built a three-stage deployment pipeline and promoted content from Development to Production with a stage comparison

✅ **Task 2**: Applied endorsement, a sensitivity label, and reviewed lineage for a published dataset

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Deployment pipelines (Dev/Test/Prod) | Task 1 | [Get started with deployment pipelines](https://learn.microsoft.com/en-us/fabric/cicd/deployment-pipelines/get-started-with-deployment-pipelines) |
| Create deployment rules | Task 1 | [Introduction to deployment pipelines](https://learn.microsoft.com/en-us/fabric/cicd/deployment-pipelines/create-rules) |
| Endorsement (Promoted/Certified) | Task 2 | [Endorse your content in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content) |
| Sensitivity labels and lineage | Task 2 | [Apply sensitivity labels in Power BI](https://learn.microsoft.com/en-us/fabric/enterprise/powerbi/service-security-sensitivity-label-overview) |


---

# Lab E: Sharing Semantic Models

> **Access requirements:** You need **Admin or Member** access to the workspace that holds the semantic model, and a **second user or test account** (the "colleague") with a Power BI Pro or PPU license. Pair up with another participant if you have no second account; each of you plays the owner in one half and the colleague in the other. Free-license users can only build on a shared model hosted on Fabric F64 or higher (or Premium) capacity, and can only save reports to **My workspace**.
>
> **Lab B dependency:** This lab uses the `UPN` role from Lab B. The colleague's email address must match the UPN value you set in Lab B, Task 4 (the **Replace Values** step in the `Performance` query), and the published model must include that change. Otherwise the role returns no rows for the colleague.

## Lab Introduction

In this lab, you will share one governed semantic model so other people can build their own reports on it instead of keeping separate copies. You grant Build permission, build a thin report in Power BI Desktop through a live connection, see how the **UPN** role's dynamic row-level and object-level security still applies to shared models, and then remove access.

This lab will guide you through the following tasks:

- Prepare the semantic model and configure how people request access
- Grant a colleague Build permission
- Build a thin report from the shared semantic model, and assign the `UPN` role so the colleague can read it
- Observe the `UPN` role's dynamic RLS and OLS in the thin report
- Remove Build permission and review governance controls

## Lab Scenario

Several analysts each keep their own copy of the AdventureWorks model, so their reports show different numbers. You will publish one promoted semantic model, share it with a colleague, and let them build a report that contains no model of its own. Every report then uses the same measures, relationships, and security.

## Architecture Overview

```text
Sales Production workspace (owner)
`-- AdvancedSalesReport semantic model   (Promoted, `UPN` role from Lab B)
        |
        | Share: Read + Build
        v
Colleague
|-- Power BI Desktop: Get data -> Power BI semantic models (live connection)
|-- Thin report: no model, uses shared measures and tables
`-- Publish -> colleague's own workspace
        |
        | Role membership: UPN
        |   RLS: Performance[UPN] / Salesperson[UPN] = USERPRINCIPALNAME()
        |   OLS: Sales[Cost] and Sales[Cost Price] hidden
        v
Colleague sees only their own rows; Profit and Profit Margin fail
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Prepare the semantic model and configure access requests |
| Task 2 | Grant Build permission |
| Task 3 | Connect to the shared model, assign the `UPN` role, and build a thin report |
| Task 4 | Observe dynamic RLS and OLS in the thin report |
| Task 5 | Remove Build permission and review governance controls |

---

## Task 1: Prepare the Semantic Model and Configure Access Requests

*Performed by the owner.*

1. In the Power BI service, open the **Sales Production** workspace from Lab D. If you skipped Lab D, use any workspace where you are Admin or Member and publish `AdvancedSalesReport.pbix` to it from Desktop.

2. Confirm the **AdvancedSalesReport** semantic model is **Promoted** (the badge from Lab D). Promoted models are easier to find.

3. Select the semantic model, then **More options (…) → Settings → Data access → Data source credentials**. Confirm the credentials are configured, then select **Refresh now** and wait for it to succeed. A thin report can't show data until the shared model has been refreshed.

4. In the same settings, find **Sharing and access → Request access**. 
    - Select the option: User requesting access will get the following instructions
    - Enter `Contact the Sales BI team to request Build permission.` 
    - Select Apply. People without Build permission now see this message instead of sending you an email.

   > **Note:** When you provide instructions, your email address is visible to people who request access.

> **Checkpoint:** The semantic model is promoted, refreshed, and has request-access instructions configured.

## Task 2: Grant Build Permission

*Performed by the owner.*

1. In the **Sales Production** workspace, select **More options (…) → Manage permissions** next to the semantic model, then **Add user**.

2. Enter the colleague's name or email address.

3. Select only **Allow recipients to build content with the data associated with this semantic model**. Leave **modify** and **share** cleared. Select **Grant access**.

   > **Note:** Build includes Read. If you cleared all three checkboxes, the colleague would get Read only.

4. Select **Direct access** tab and confirm the colleague is listed with **Read** and **Build** permission.

> **Checkpoint:** The colleague has Read and Build permission, and no Write or Reshare permission.

## Task 3: Build a Thin Report on the Shared Semantic Model

*Performed by the colleague.*

1. Open Power BI Desktop and select **Get data → Power BI semantic models**.

2. In the list, find **AdvancedSalesReport**. Look for the **Promoted** badge or filter to **Endorsed in your org**. Select it and select **Connect**.

   > **Note:** This creates a live connection. You haven't copied the model; it stays in the owner's workspace.

3. **Expected result:** Desktop shows **Cannot load model**, with a message that the database doesn't exist or you don't have permission to access it. This is expected. The semantic model has row-level security roles, and a user who has Read and Build permission but belongs to **no role** can discover the model but can't read it. Close the message and tell the owner to provide you the proper Security role.

4. **Owner:** In the **Sales Production** workspace, select **More options (…)** next to the semantic model, then **Security**. Select the **UPN** role, add the colleague's account, and select **Add**, then **Save**.

   > **Note:** If the `UPN` role is missing because you skipped Lab B, or the model wasn't redeployed after you created it, ask the trainer to demonstrate this lab. Role definitions are deployed with the model, but role membership is configured separately in each workspace.

5. **Colleague:** Close the Desktop error, open a new Desktop file, and repeat steps 1 and 2. The connection now succeeds. Allow a few minutes for the role assignment to take effect if it still fails.

6. Review the **Data** pane. The tables and measures, including **Total Sales**, come from the shared model. There is no Power Query to load.

7. Add a **Clustered bar chart** with **Region[Region]** on the axis and **Total Sales** as the value.

8. Add a **Table** visual with **Salesperson[Salesperson]** and **Sales[Sales Amount]**. Notice that the Cost, Cost Price, Profit, and Profit Margin columns from the Sales Table are not visible as they are affected by object-level security.

9. Save the report as `SalesThinReport.pbix`, then select **Publish** and choose a workspace you can publish to, such as a new workspace named **Sales Reports**. Ensure that the Owner of the shared semantic model has permissions in the same workspace to access the report.

10. In the service, open the published report. Confirm that the report opens and shows data. Notice as well that the published report does not have a semantic model of its own; it relies entirely on the shared model.

> **Checkpoint:** The colleague has a published report built entirely on the shared semantic model. Build permission let the colleague discover the model; the `UPN` role let them read it. Owners can change a measure in the shared model and every report built on it would reflect that change.

## Task 4: Observe Dynamic RLS and OLS in the Thin Report

*Performed by the colleague, then the owner.*

Security defined on the shared model applies to everyone who builds on it. 

1. **Colleague:** Open the thin report in the service and observe the two controls working together:
   - **Row-level security:** The bar chart and table show only the colleague's own sales rows, matched on their email address, not the whole company.
   - **Object-level security:** The table cannot display the Cost, Cost Price, Profit, and Profit Margin columns from the Sales Table.

2. **Colleague:** Open the report in Editing mode. In the **Data** pane of the report, confirm that Cost, Cost Price, Profit, and Profit Margin columns from the Sales Table aren't listed.

3. **Owner:** Open the same thin report. The owner isn't restricted by the role: the report shows **all** sales rows and all salespersons, and the Cost, Cost Price, Profit, and Profit Margin columns from the Sales Table are visible. Users with Write permission on the semantic model aren't limited by RLS.

> **Checkpoint:** Security follows the shared model. The colleague can only see rows allowed by their UPN, and can't see hidden columns, regardless of which report they use. Don't rely on withholding Build or Read permission to protect sensitive data; use RLS and OLS.

## Task 5: Remove Build Permission and Review Governance Controls

*Performed by the owner, with a trainer discussion.*

1. Open **Manage permissions → Direct access** for the semantic model.

2. Find the colleague, select **More options (…)**, and **Remove Build** permission. The colleague must only have Read permission.

3. **Colleague:** Open the thin report in the service. The report is still visible, but the colleague can no longer edit it or export the underlying data. 

4. **Colleague:** Close the Power BI Desktop and reopen again the SalesThinReport.pbix file. Observe that the colleague can no longer access the semantic model.

4. **Trainer discussion:** Review these governance points:
   - The Power BI admin tenant setting **Use semantic models across workspaces** can restrict or turn off semantic model reuse for the whole tenant or specific security groups. When it's off, report copying is unavailable and discovery only shows models in the current workspace.
   - If you publish an app, the audience also needs access to the underlying semantic model. Otherwise reports in the app won't open.
   - *Publish to web* doesn't work for reports based on a shared semantic model.

---

## Lab E Summary

### What You Accomplished

✅ **Task 1**: Prepared a promoted, refreshed semantic model and configured how people request access

✅ **Task 2**: Granted a colleague Read and Build permission without Write or Reshare

✅ **Task 3**: Saw why Build permission alone isn't enough on a model with RLS, assigned the `UPN` role, and built and published a thin report using a live connection to the shared semantic model

✅ **Task 4**: Observed dynamic row-level security and object-level security on the shared model, compared with the owner's view

✅ **Task 5**: Removed Build permission and reviewed tenant-level governance controls

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Sharing semantic models across workspaces | Task 1, 3 | [Introduction to semantic models across workspaces](https://learn.microsoft.com/power-bi/connect-data/service-datasets-across-workspaces) |
| Share dialog, Read, Build, Reshare and Write | Task 2 | [Share access to a semantic model](https://learn.microsoft.com/power-bi/connect-data/service-datasets-share) |
| Build permission and access requests | Task 1, 2, 5 | [Build permission for shared semantic models](https://learn.microsoft.com/power-bi/connect-data/service-datasets-build-permissions) |
| Live connection from Power BI Desktop | Task 3 | [Connect to semantic models from Power BI Desktop](https://learn.microsoft.com/power-bi/connect-data/desktop-report-lifecycle-datasets) |
| Dynamic RLS and OLS on a shared model | Task 4 | [Semantic model permissions](https://learn.microsoft.com/power-bi/connect-data/service-datasets-permissions) and [Object-level security in Power BI](https://learn.microsoft.com/power-bi/enterprise/service-admin-ols) |
| Manage direct access | Task 2, 5 | [Manage semantic model access permissions](https://learn.microsoft.com/power-bi/connect-data/service-datasets-manage-access-permissions) |
| Tenant control of model reuse | Task 5 | [Control the use of semantic models across workspaces](https://learn.microsoft.com/power-bi/connect-data/service-datasets-admin-across-workspaces) |

---

# Lab F: AI Visuals & Preparing a Semantic Model for AI

> **License note:** The decomposition tree, key influencers, and custom-mode smart narrative are available in Power BI Desktop and the Power BI service to all participants. The Copilot mode of the Narrative visual, and testing **Prep data for AI** with Copilot, require a Copilot-enabled workspace on **any Fabric F SKU (F2+) or PPU** — trainer-led demo if unavailable.
>
> **Lab A dependency (optional):** If you completed Lab A, Task 6, your model contains the `Slow Measure` and `Optimized Measure` measures. Task 5 uses them as examples of fields to keep away from Copilot. If you skipped it, ignore those two names.

## Lab Introduction

In this lab, you use Power BI's AI visuals to explore the AdventureWorks sales model: a **decomposition tree** to drill into sales and find root causes, **key influencers** to find what drives sales, and a **smart narrative** to summarize the page in plain language. You then prepare the same semantic model for AI by using **Prep data for AI** (a simplified AI data schema, AI instructions, and verified answers), test the result with Copilot against a baseline, and mark the model **Approved for Copilot**.

This lab will guide you through the following tasks:

- Break down Total Sales across several dimensions with a decomposition tree and AI splits
- Identify the factors that drive sales with the key influencers visual
- Summarize the page and individual visuals with a custom smart narrative
- Capture a Copilot baseline on the unprepared model
- Prepare the semantic model for AI: simplify the data schema, add AI instructions, and set verified answers
- Test the prepared model with Copilot and compare it to the baseline
- Publish the model and mark it as Approved for Copilot

## Lab Scenario

Your sales leaders keep asking the same "why" questions: *Where do our sales come from? What drives a large sale? Can the report just tell me what changed?* You will add AI visuals to the AdventureWorks report so consumers can answer these questions themselves. Because the same semantic model will also back Copilot, you then clean it up so that Copilot uses the right measures, ignores technical fields, and gives consistent answers to the questions people ask most.

## Architecture Overview

```text
AdvancedSalesReport.pbix  -> new page: AI Insights
|-- Decomposition tree   Analyze: Total Sales   Explain by: Category, Business Type, Region, Year
|-- Key influencers      Analyze: Sales Amount  Explain by: Category, Business Type, Country-Region, Channel
|-- Narrative (Custom)   Page summary + dynamic value + visual summary
`-- Supporting visuals   Line chart (Total Sales by Year) and bar chart (Total Sales by Region)

        |
        v
Prep data for AI (saved on the semantic model, not the report)
|-- Simplify data schema   keep business-friendly fields, remove keys / UPN / cost / test measures
|-- Add AI instructions    metrics, dates, terminology, ambiguity rules
`-- Verified answers       trigger phrases mapped to approved visuals

        |
        v
Copilot pane in Desktop (skill picker: Answer questions about the data)
Baseline prompts (before)  ->  same prompts (after)  ->  compare

        |
        v  Publish
Power BI service -> semantic model Settings -> Approved for Copilot
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description |
|---|---|
| Task 1 | Explore Total Sales with a decomposition tree and AI splits |
| Task 2 | Find what drives sales with key influencers |
| Task 3 | Summarize the page and visuals with a smart narrative |
| Task 4 | Capture a Copilot baseline on the unprepared model |
| Task 5 | Prepare the semantic model for AI |
| Task 6 | Test the prepared model and compare with the baseline |
| Task 7 | Publish and mark the model as Approved for Copilot |

> **Field names:** The steps use the AdventureWorks tables in `AdvancedSalesReport.pbix` (`Sales`, `Date`, `Product`, `Reseller`, `Region`, `Salesperson`). If a column in your copy has a slightly different name, use the closest equivalent. Absolute numbers will differ from any examples you see on screen.

---

## Task 1: Explore Total Sales with a Decomposition Tree

The decomposition tree aggregates a measure and lets you drill into any dimension in any order. It is useful for ad hoc exploration and root-cause analysis.

1. Open `AdvancedSalesReport.pbix` in Power BI Desktop. Add a new report page and rename it **AI Insights**.

2. Add a **Line chart** with `Date[Year]` on the X-axis and `Sales[Total Sales]` on the Y-axis. Sort the line chart by Ascending Year. Add a **Clustered bar chart** with `Region[Region]` on the Y-axis and `Sales[Total Sales]` on the X-axis. Resize them to the left of the page.  These give the other AI visuals something to cross-filter with and summarize.

3. Select an empty area of the canvas and, from the **Visualizations** pane, select **Decomposition tree**.

4. Set the two inputs:
   - **Analyze**: `Sales[Total Sales]`. This must be a measure or an aggregate.
   - **Explain by**: `Product[Category]`, `Reseller[Business Type]`, `Region[Region]`, `Date[Year]`, and `Salesperson[Salesperson]`.

   The visual shows a single root node with the aggregated Total Sales.

5. Select the **+** next to the root node and choose **Category**. The tree expands to show Total Sales by category. Select the **+** next to a category node and choose **Region** to drill further. You can drill into the fields in any order.

6. Select a node from an **earlier level**. Notice that the path changes. Selecting a node on the **last level** cross-filters the other visuals on the page instead.

7. Now use **AI splits**. Select the **+** next to a node and choose **High value**. The tree adds a level marked with a **light bulb**, with a dotted line pointing to the node that has the highest Total Sales. Hover over the light bulb to read the explanation, for example "Total Sales is highest when Region is …".

8. Repeat with **Low value** on another node and compare the field that Power BI chooses.

9. Select the decomposition tree, then open **Format visual** and find the Analysis type setting for AI splits. Switch it from **Absolute** to **Relative**, then run **High value** again. Relative mode looks for a value that stands out compared with the rest of the values in the same column, which is often more interesting than the largest absolute value. To stop using AI splits entirely, turn them off in the same formatting section.

10. Select one region in the bar chart. The decomposition tree is cross-filtered and the AI splits are recalculated, so Power BI might now recommend a different field or node. Select the bar again to clear the selection.

11. Hover over the header of the first level of the tree and select the **lock** icon. A locked level can't be removed or changed by report consumers, who can still explore the nodes beneath it.

> **Note:** The tree supports up to 50 levels, shows the top 10 values per level, and can visualize up to 5,000 data points at a time. AI splits aren't supported in Publish to web or Power BI Report Server.

> **Checkpoint:** You can drill into Total Sales by any dimension, use High value and Low value AI splits, and lock the first level of the tree.

## Task 2: Find What Drives Sales with Key Influencers

The key influencers visual analyzes your data, ranks the factors that matter, and displays them as key influencers. Here you ask: *what influences the value of a sales order line to be higher or lower?*

1. On the **AI Insights** page, select an empty area and, from **Visualizations**, select **Key influencers**. Place it so it doesn't cover the other visuals; you can resize the decomposition tree to make room.

2. Drag `Sales[Sales]` into **Analyze**. Select the drop-down arrow on the field in the well and choose **Don't summarize** so the analysis runs on each row of the Sales table rather than on a total.

3. Drag these fields into **Explain by**: `Product[Category]`, `Reseller[Business Type]`, `Region[Country]`, `Salesperson[Salesperson]`, and `Date[Year]`. Leave **Expand by** empty for now.

   > **Note:** Every explanatory field must be at the same level as the metric you analyze. The fields above are attributes of a sales row (many-to-one from `Sales`), so Power BI can use them directly. Fields on the "many" side, such as a count of orders, must first be aggregated to that level.

4. Because Sales Amount has many unique values, it isn't meaningful to ask "what influences Sales Amount to be 1,234.56?". Open **Format visual**, find the **Analysis → Analysis type** setting, and select **Continuous**. 

5. In the drop-down at the top of the visual, select **Increase**, so the question reads *What influences Sales to Increase?*

6. On the **Key influencers** tab, select the top influencer in the left pane. Use the sentence above each pane to read it. The bubble shows how much the average Sales increases by (for example "$2.29K") when that condition is true. In the right pane, the blue bar is the average when the condition is true and the dotted line is the average of all the other values.

7. Select the second influencer and notice that the dotted line (average) changes. 

8. Select the **Only show values that are influencers** checkbox under the right-hand chart to remove values that have no effect.

9. Change the question to **Decrease** and compare which fields drive lower sales.

10. Select the **Top segments** tab and select **High**. Each bubble is a combination of conditions. 
   - The higher the bubble, the higher the average Sales in the segment. 
   - The bigger the bubble, the more rows it holds. 
   
   Select the first segment and read the description of which values make it up and how its average compares with the overall average.

11. Return to the **Key influencers** tab. In **Format visual**, open the **Analysis → Analysis type** card and turn on **Counts**. A ring now shows how much of the data each influencer covers. A big effect that covers very little data may not be worth acting on.

12. In the bar chart, select one region. Every selection reruns the analysis on that slice of data. Compare the top influencers for one region against the whole company, then clear the selection.

> **Checkpoint:** You can explain what the top influencer is, how it is calculated, and the difference between the **Key influencers** and **Top segments** tabs.

## Task 3: Summarize the Page and Visuals with a Smart Narrative

The **Narrative** visual has two modes: **Copilot** (AI-generated text, needs a Copilot license) and **Custom** (a dynamic, editable summary). This task uses **Custom** mode, which works for everyone.

1. On the **AI Insights** page, select an empty area of the canvas and, from **Visualizations**, select **Narrative**.

2. If the visual opens in Copilot mode, select the **Copilot** drop-down and choose **Custom** to generate an editable summary of the visuals on the page. The text covers the trends and highest and lowest values in the line chart and the bar chart, and it can also include a sentence about the key influencers visual, such as which field contributed most to the increase in sales.

   > **Note:** Custom mode doesn't summarize every visual type. Tables, matrices, and custom visuals are not summarized, and it generates up to four summaries per visual and up to 16 per page. Which visuals are summarized, and how many summaries each gets, depends on their size and position on the page, so your text might differ from a classmate's. Key influencers summaries are only available for some analyses; if you don't see one, the other summaries are unaffected.

3. Add your own text to the summary and format it with the text box toolbar (font, **B**, *I*, alignment, bullets). The toolbar appears when you select the narrative visual and click inside the text.
   - Click at the very beginning of the first sentence, press **Enter** to create two blank lines above it, then click in the top blank line and type `Sales Insights`.
   - Select the text `Sales Insights` and select **B** (Bold) on the toolbar. Increase the font size to 14.

   > **Note:** The underlined parts of the generated text are dynamic values that update with your data and filters. Type around them rather than inside them. To remove a generated sentence you don't need, select the whole sentence and delete it.

4. Add a new blank line at the very end.  Select **+ Value**. Under **Data**, select `Sales[SalesOrderNumber]`, set **Summarization** to **Count (Distinct)**, name the value `OrderCount`, and select **Save**. Type `distinct orders were placed.` after it.

5. Add two new blank lines at the very end. Add a second dynamic value that uses the `Total Sales` measure, and format it as Currency, name it as `TotalSales`, and select **Save**. Type `Total Sales:` before it.

7. Select a region in the bar chart. The summary and your dynamic values change to reflect the selection. Clear the selection.

8. Right-click the **line chart** and select **Summarize**. Notice that summarizing a single visual can produce more insights than the page summary does.

9. Select the **bar chart**, open **Format visual → General**, expand **Header icons → Icons**, and set **Smart narrative** to **On**. Hover over the visual and select the **Smart narrative** icon to see an on-demand description. Unlike the narrative visual, readers can't add this text to the page.

10. **Optional (Copilot required):** Add another Narrative visual, keep it in **Copilot** mode, and use the suggested prompt to generate a summary. Compare it with the custom narrative: custom mode is dynamic and repeatable, while Copilot output varies with each prompt.

> Note: To enable Copilot, select the Settings gear in the top-right corner of Power BI Desktop, then choose **Options → Global → Copilot (preview)** and select a workspace with a Fabric capacity.

> **Checkpoint:** The page has a custom narrative with two dynamic values that update when you select a region, and the bar chart has a smart narrative icon.

## Task 4: Capture a Copilot Baseline on the Unprepared Model

Before you prepare the model, record how Copilot responds to the same questions you will test again later.

> **Copilot is required for Tasks 4 and 6.** To enable Copilot, select the Settings gear in the top-right corner of Power BI Desktop, then choose **Options → Global → Copilot (preview)** and select a workspace with a Fabric capacity. If Copilot isn't available, the trainer demonstrates these tasks. You can still complete Task 5.

1. In Power BI Desktop, select **Copilot** on the **Home** ribbon to open the Copilot pane.

2. Ask each of the following prompts, and note the answer and which fields or measures Copilot used. Use **How Copilot arrived at this** under each answer to see the details.
   1. `What were total sales for FY2020?`
   2. `Which region performs best?`
   3. `Show sales by product category.`
   4. `What is the margin by reseller business type?`
   5. `Show sales by salesperson.`

3. Record your baseline in a table like this one:

   | Prompt | Measure or column Copilot used | Was it the right one? |
   |---|---|---|
   | 1 |  |  |
   | 2 |  |  |
   | 3 |  |  |
   | 4 |  |  |
   | 5 |  |  |

> **Checkpoint:** You have a record of how Copilot answers five prompts before any preparation.

## Task 5: Prepare the Semantic Model for AI

**Prep data for AI** has three features that give Copilot the context it needs to reduce ambiguity: an **AI data schema**, **AI instructions**, and **verified answers**. All three are saved on the **semantic model**, not the report, so every report built on the model benefits.

### Part A: Turn on Prep data for AI

1. On the **Home** ribbon, select **Prep data for AI**. If the button isn't visible, go to **File → Options and settings → Options → Preview features**, select **Prep data for AI**, and restart Power BI Desktop.

2. If the tabs in the dialog are disabled, select **Turn on Q&A** (Q&A must be enabled on the model for these features to work), then continue.

   > **Note:** The Q&A *visual* is being retired in February 2027. The model-level Q&A setting is still required by **Prep data for AI**, and Copilot is the supported way to ask questions in natural language.

### Part B: Simplify the data schema

An AI data schema is a focused subset of the model that Copilot prioritizes when it answers data questions. Keep clean, business-friendly fields and remove anything that could confuse Copilot.

3. In the dialog, open the **Simplify data schema** tab.

4. Keep these fields selected:

   | Table | Keep |
   |---|---|
   | `Sales` | `Total Sales`, `Sales Amount`, `Profit`, `Profit Margin`, `Sales YoY Growth`, `Order Quantity` |
   | `Date` | `Date`, `Year`, `Month`, and any Quarter field |
   | `Product` | `Category`, `Subcategory`, `Product` |
   | `Reseller` | `Reseller`, `Business Type` |
   | `Region` | `Region`, `Country-Region`, `Group` |
   | `Salesperson` | `Salesperson` |

5. Clear everything else, including:
   - Key and ID columns such as `ResellerKey`, `ProductKey`, and `SalesOrderNumber`
   - The `UPN` columns in `Salesperson` and `Performance`, and the whole `Performance` table
   - `Sales[Cost]` and `Sales[Cost Price]`, which Lab B restricts with object-level security
   - The `Slow Measure` and `Optimized Measure` test measures from Lab A, if present

   > **Note:** Fields hidden in the model start unselected the first time you set a schema, but all fields appear in this pane regardless of visibility. Copilot still respects relationships, so a field on a kept table can still be filtered by a related table you removed.

6. Select **Apply**.

### Part C: Add AI instructions

AI instructions give Copilot business context, terminology, and rules for analysis. They are saved at the model level and consumers can't see them.

7. Open the **Add AI instructions** tab and enter the following text. Adjust names that don't match your model:

   ```text
   Metrics
   - When a user asks about "sales" or "revenue" without another qualifier,
     use [Total Sales].
   - For margin, use [Profit Margin]. Don't average transaction-level margins.
   - For growth or "year over year", use [Sales YoY Growth].

   Dates
   - Use 'Date'[Date], related to the order date, for all sales analysis.
   - 'Date'[Year] holds fiscal years such as FY2019. When a user says "last year",
     use the most recent complete fiscal year in the data, not today's date.

   Terminology
   - "Best performing" region means the region with the highest [Total Sales].
   - "Top sellers" means the three salespeople with the highest [Total Sales].
   - "Partner" and "dealer" mean a reseller in the 'Reseller' table.

   Ambiguity
   - If a user asks about "performance" and doesn't say which metric,
     ask whether they mean sales, margin, or growth.
   - Don't show UPN, email addresses, or key columns.
   ```

8. Select **Apply**.

   > **Note:** Keep instructions explicit, grouped by theme, and short. Fewer focused instructions usually beat many broad ones. Instructions guide Copilot but can't guarantee a specific output, and they are limited to 10,000 characters.

### Part D: Set verified answers

A verified answer is a human-approved visual that predefined trigger phrases return. Copilot checks for an exact or semantically similar match to a trigger phrase before it generates its own answer.

9. On the **AI Insights** page, add a **Clustered bar chart** with `Region[Region]` on the Y-axis and `Sales[Total Sales]` on the X-axis. Name it clearly with a title such as `Total Sales by Region`.

10. Add `Date[Year]` to the **Filters** pane of this visual and leave it with no value selected. A filter must already exist in the report and have no specific value to be offered to users as an adjustable filter.

11. Select the visual, then select **More options (…) → Set up a verified answer**.

12. Add the trigger phrases `Sales by region`, `Which region sells the most?`, and `Show me total sales by region`. You can also select suggestions under **Copilot suggestions**.

13. Select the **filter** icon and, under **Available to users**, select `Date[Year]`. Users can now ask for a specific year, such as *Sales by region for FY2020*, and Copilot returns this visual filtered to that year.

14. Select **Apply**.

15. Add a **Line chart** with `Date[Year]` on the X-axis and `Sales[Total Sales]` on the Y-axis, set up a verified answer in the same way with trigger phrases `Sales trend` and `How are sales changing over time?`, and select **Apply**.

   > **Note:** Filters applied to the visual when you set up the answer are saved with it, and slicers aren't carried over. Verified answers support up to three adjustable filters.

> **Checkpoint:** The model has a simplified AI data schema, a set of AI instructions, and two verified answers.

## Task 6: Test the Prepared Model and Compare with the Baseline

1. Close and reopen the **Copilot** pane. You must do this every time you change **Prep data for AI**.

2. Make sure the skill picker still has only **Answer questions about the data** selected.

3. Ask the same five prompts from Task 4 and compare the answers to your baseline table:
   1. `What were total sales last year?` – should use `[Total Sales]` and the fiscal year logic from your instructions.
   2. `Which region performs best?` – should use the highest `[Total Sales]` instead of asking what "best" means.
   3. `Show sales by product category.` – should use `[Total Sales]`.
   4. `What is the margin by reseller business type?` – should use `[Profit Margin]`.
   5. `Show sales by salesperson email.` – Copilot shouldn't use the `UPN` field because it is no longer in the schema.

4. Test the verified answers:
   - Ask `Sales by region` and confirm Copilot returns the visual you approved and marks it as a verified answer.
   - Ask `Sales by region for FY2020` and confirm the filter is applied.
   - Ask `How are sales changing over time?` and confirm the line chart is returned, even though the wording differs from your trigger phrases.

5. Test the schema: ask for a field you removed, such as `Show sales by ResellerKey`. Copilot shouldn't return an answer based on that field.

6. Review **How Copilot arrived at this** under an answer, and use **… → Download diagnostics** on the Copilot pane to confirm the AI data schema and instructions were applied.

7. Switch the skill picker to **Create new report pages** only, and ask `Create a page showing sales trends by product category`. Observe that creating a page uses the entire semantic model, so the AI data schema doesn't restrict it.

8. If an answer isn't what you expected, change the instructions or schema in **Prep data for AI**, select **Apply**, close and reopen the Copilot pane, and test again. Preparing a model is iterative, and AI responses vary from run to run.

> **Checkpoint:** Copilot answers are more consistent than the baseline, and the verified answers are returned for the trigger phrases.

## Task 7: Publish and Mark the Model as Approved for Copilot

1. Save the file and select **Publish** on the **Home** ribbon. Publish to the **Sales Production** workspace from Lab D, or to another Copilot-enabled workspace you can publish to. Replace the existing report and semantic model if prompted.

2. In the Power BI service, open the workspace, select the **AdvancedSalesReport** semantic model, and confirm the **Prep data for AI** button is available on the ribbon. You can edit the AI data schema and instructions here too, and manage verified answers from this button.

3. Open the semantic model's **Settings**, expand **Approved for Copilot**, select the **Approved for Copilot** box, and select **Apply**.

4. Open the report and test one of the verified answers in the Copilot pane. Changes can take a few minutes to take effect and may take up to an hour (or up to 24 hours for models with many reports) to show everywhere. Saving a small change to a report can speed up the refresh for that report.

5. **Trainer discussion:** Review these points:
   - Approving a model removes the friction treatment that standalone Copilot adds to answers from unapproved models, and an admin can restrict standalone Copilot to approved items only.
   - Only the semantic model can be marked **Approved for Copilot**. You can't mark a report, dashboard, or app.
   - **Prep data for AI is not a security feature.** Removing a field from the AI data schema keeps Copilot from using it, but it doesn't stop anyone with Build permission from querying it. Keep using RLS and OLS (Lab B) to protect data.
   - Validate Copilot answers before acting on them. Copilot in Power BI sends query results and metadata to Azure OpenAI, and tenant admins control whether Copilot is enabled.
   - If you move the model through Git or deployment pipelines (Labs C and D), refresh the model in the service after deployment so the Prep data for AI changes sync.

> **Checkpoint:** The model is published and approved for Copilot, and its AI data schema, instructions, and verified answers apply to every report that uses it.

---

## Lab F Summary

### What You Accomplished

✅ **Task 1**: Broke down Total Sales with a decomposition tree, used High value and Low value AI splits, and locked a level

✅ **Task 2**: Used key influencers to identify what drives sales up or down and reviewed the top segments

✅ **Task 3**: Created a custom smart narrative with dynamic values, summarized a single visual, and enabled the smart narrative icon

✅ **Task 4**: Recorded a Copilot baseline on the unprepared model

✅ **Task 5**: Simplified the AI data schema, wrote AI instructions, and set verified answers

✅ **Task 6**: Tested the prepared model with Copilot and compared the results with the baseline

✅ **Task 7**: Published the model and marked it as Approved for Copilot

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Decomposition tree and AI splits | Task 1 | [Create and view decomposition tree visuals in Power BI](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-decomposition-tree) |
| Key influencers and top segments | Task 2 | [Create key influencers visualizations](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-influencers?tabs=powerbi-desktop) |
| Custom smart narrative and dynamic values | Task 3 | [Create smart narrative summaries](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-smart-narrative) |
| Prep data for AI, skill picker, and Approved for Copilot | Tasks 4–7 | [Prepare your data for AI to improve Copilot results](https://learn.microsoft.com/power-bi/create-reports/copilot-prepare-data-ai) |
| AI data schema | Task 5 | [Prepare your data for AI: AI data schemas](https://learn.microsoft.com/power-bi/create-reports/copilot-prepare-data-ai-data-schema) |
| AI instructions | Task 5 | [Prepare your data for AI: AI instructions](https://learn.microsoft.com/power-bi/create-reports/copilot-prepare-data-ai-instructions) |
| Verified answers | Task 5, 6 | [Prepare your data for AI: Verified answers](https://learn.microsoft.com/power-bi/create-reports/copilot-prepare-data-ai-verified-answers) |
| Lab template | Tasks 4–7 | [Prepare a semantic model for AI (mslearn-fabric)](https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/30-prepare-model-ai.html) |

AI visuals show patterns; they don't prove causes. Treat key influencers and AI splits as leads to investigate. Copilot accelerates analysis but requires review, because AI-generated answers can reflect errors or bias in the underlying data and aren't identical on every run.

### Recommended References

- [Create and view decomposition tree visuals in Power BI](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-decomposition-tree)
- [Create key influencers visualizations](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-influencers?tabs=powerbi-desktop)
- [Create smart narrative summaries](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-smart-narrative)
- [Prepare your data for AI to improve Copilot results](https://learn.microsoft.com/power-bi/create-reports/copilot-prepare-data-ai)
- [Prepare a semantic model for AI (mslearn-fabric)](https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/30-prepare-model-ai.html)
- [Overview of Copilot for Power BI](https://learn.microsoft.com/power-bi/create-reports/copilot-introduction)
