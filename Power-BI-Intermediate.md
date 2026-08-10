# Power BI Intermediate Training — 3-Hour Session

**Audience:** Users with basic Power BI awareness  
**License:** Power BI Free (My Workspace) — Power BI Desktop is a free download  
**Format:** Short presentation followed by hands-on labs  
**Duration:** 3 Hours  

> **Note:** All lab exercises use Power BI Desktop (free download) and the Power BI free **My Workspace** in the Power BI Service. A Power BI Pro license is **not** required to complete any lab in this session. Publishing and sharing to shared workspaces requires Pro and is covered conceptually only.

---

## Agenda Overview

| Segment | Topic | Duration |
|---|---|---|
| 1 | Welcome & Recap | 10 min |
| 2 | Presentation: Data Modeling, Best Practices & Responsible Use | 35 min |
| 3 | Lab A: Connect to Data & Build Your First Report | 25 min |
| 4 | Lab B: Data Transformation with Power Query Editor | 30 min |
| — | Break | 15 min |
| 5 | Lab C: Data Modeling & DAX Basics | 25 min |
| 6 | Lab D: Publish to My Workspace & Explore the Power BI Service | 30 min |
| 7 | Wrap-Up, Q&A & Next Steps | 10 min |

---

## Segment 1: Welcome & Recap (10 minutes)

- Welcome and housekeeping
- Quick recap of Power BI awareness session concepts
- Session objectives and what participants will build today
- Overview of the Power BI free license scope
  - What's included: Power BI Desktop (full), My Workspace in the Power BI Service, view content in Premium capacity workspaces
  - What's NOT included: shared workspaces, sharing reports with colleagues, scheduled refresh on shared content (all require Pro)

---

## Segment 2: Presentation — Data Modeling, Best Practices & Responsible Use (35 minutes)

### The Power BI Workflow & Data Modeling Concepts (10 minutes)
- The end-to-end Power BI workflow: **Get Data → Transform → Model → Visualize → Publish**
- Understanding the data model: tables, columns, relationships
- Star schema basics — fact tables vs. dimension tables
- Calculated columns vs. measures — when to use each
- Introduction to DAX: `SUM`, `COUNT`, `CALCULATE`, `DIVIDE`
- Auto date/time vs. a dedicated date table

### Best Practices in Building Reports (15 minutes)
- **Naming conventions** — use clear, descriptive names for tables, columns, and measures
- **Use measures over calculated columns** for aggregations — measures are evaluated at query time and are more performant
- **Transform in Power Query, not DAX** — clean and shape data before it enters the model
- **Keep the model lean** — only import the columns and rows you actually need
- **Avoid relationships on text columns** — use integer keys for relationships where possible
- **One page, one story** — design each report page to answer a specific business question
- **Consistent visuals** — use a single color theme; limit the number of visual types per page
- **Slicers and filters** — use slicers for user-driven filtering; use report/page filters for fixed context
- **Label your measures** — group measures in a dedicated measures table for discoverability
- **Save incrementally** — save your `.pbix` file regularly; Power BI Desktop has no auto-save

### Responsible Use of Power BI (10 minutes)
- **Data sensitivity** — only connect to data you are authorized to access; apply sensitivity labels where available
- **Data accuracy** — validate report numbers against source systems before sharing; never publish unverified data
- **Row-level security (RLS)** — understand that reports shared without RLS expose all underlying data to viewers
- **Principle of least data** — only include data fields necessary for the business question being answered
- **Transparency in calculations** — document your DAX measures so colleagues understand what they represent
- **Avoid misleading visuals** — use appropriate chart types; do not truncate axes in ways that distort comparisons
- **Data refresh awareness** — communicate clearly when data was last refreshed; stale data can lead to poor decisions
- **Licensing and sharing** — understand what the free license allows; do not circumvent Pro requirements by exporting raw data for redistribution
- **AI and Copilot features** — when using Q&A or Copilot in Power BI, understand what data is sent for processing

> **Reference:** [Microsoft responsible AI principles for Power BI](https://learn.microsoft.com/power-bi/enterprise/service-security-sensitivity-label-overview)

---

## Segment 3: Lab A — Connect to Data & Build Your First Report (25 minutes)

**Business Context:** A sales team wants a quick visual summary of monthly sales performance by region and product. Rather than scrolling through raw Excel data, a Power BI report gives managers an interactive view that updates whenever the source file changes.

**Tool Used:** Power BI Desktop

**Steps:**
1. Download and open **Power BI Desktop** (if not already installed: [aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore))
2. Click **Get data → Excel workbook** and load the provided sample sales Excel file (columns: Date, Region, Product, Sales Amount, Units Sold)
3. In the **Navigator**, select the data table and click **Load**
4. In the **Report** view, add a **Clustered Bar Chart** — set **Y-axis** to Region, **X-axis** to Sales Amount
5. Add a **Card** visual — set the field to Sales Amount to display total sales
6. Add a **Slicer** — set the field to Region to enable interactive filtering
7. Format the visuals: apply a consistent color theme (**View → Themes**), add a report title text box
8. Save the file as `SalesReport.pbix`

**Key Learning Points:**
- Connecting Power BI Desktop to an Excel data source
- Understanding the Report, Data, and Model views in Power BI Desktop
- Building foundational visuals: bar chart, card, and slicer
- Applying a theme and formatting for a professional finish

> **Reference:** [Get started with Power BI Desktop](https://learn.microsoft.com/power-bi/fundamentals/desktop-getting-started)

---

## Segment 4: Lab B — Data Transformation with Power Query Editor (30 minutes)

**Business Context:** Real-world data is rarely clean. Column names are inconsistent, blank rows exist, data types are wrong, and lookup tables need to be joined to the main dataset. Power Query Editor lets you define these transformations once so they apply automatically every time the data refreshes — no manual cleanup required.

**Tool Used:** Power BI Desktop (Power Query Editor)

**Steps:**
1. Open `SalesReport.pbix` and click **Transform data** to open Power Query Editor
2. In the **Sales** table:
   - Rename columns for clarity (e.g., `SalesAmt` → `Sales Amount`)
   - Change the **Date** column data type to **Date**
   - Remove any blank rows: **Home → Remove Rows → Remove Blank Rows**
   - Filter the **Region** column to exclude any "Test" or placeholder entries
3. Load a second Excel sheet (e.g., a **Product** lookup table with columns: Product, Category, Unit Cost)
4. Merge the **Sales** table with the **Product** lookup using **Home → Merge Queries** on the Product column (Left Outer join)
5. Expand the merged column to bring in the **Category** and **Unit Cost** fields
6. Add a **Custom Column** (**Add Column → Custom Column**) to calculate `Profit = [Sales Amount] - ([Units Sold] * [Unit Cost])`
7. Click **Close & Apply** to load the transformed data back into the model
8. Verify the new columns appear in the Report view field list

**Key Learning Points:**
- Using Power Query Editor as the primary data cleaning layer
- Renaming columns, fixing data types, and removing bad rows
- Merging queries to enrich a fact table with a dimension/lookup table
- Adding custom calculated columns in Power Query (M language) vs. DAX

> **References:**
> - [Shape and combine data in Power BI Desktop](https://learn.microsoft.com/power-bi/connect-data/desktop-shape-and-combine-data)
> - [Use Power Query in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-query-overview)

---

## — Break (15 minutes) —

---

## Segment 5: Lab C — Data Modeling & DAX Basics (25 minutes)

**Business Context:** With clean data loaded, the next step is to build a meaningful data model and create reusable calculations. DAX measures allow the report to dynamically calculate totals, averages, and ratios in response to any filter or slicer the user applies — far more powerful and flexible than static calculated columns.

**Tool Used:** Power BI Desktop

**Steps:**
1. Switch to the **Model** view and confirm the relationship between the **Sales** table and the **Product** lookup table on the **Product** column (Power BI may have auto-detected this)
2. Create a dedicated **Measures** table: **Enter data** with a single blank row named "Measures", then delete the column — this acts as a container for your DAX measures
3. Add the following DAX measures to the Measures table (**Home → New Measure**):
   - `Total Sales = SUM('Sales'[Sales Amount])`
   - `Total Units = SUM('Sales'[Units Sold])`
   - `Average Sale = DIVIDE([Total Sales], [Total Units], 0)`
   - `Profit Margin % = DIVIDE(SUM('Sales'[Profit]), [Total Sales], 0)`
4. Switch to **Report** view and replace the raw field on the Card visual with the `Total Sales` measure
5. Add a **Line Chart** — set **X-axis** to Date (by Month), **Y-axis** to `Total Sales` to show sales trend over time
6. Add a **Matrix** visual — set **Rows** to Category, **Values** to `Total Sales` and `Profit Margin %`
7. Format the `Profit Margin %` measure as a percentage via **Modeling → Format**

**Key Learning Points:**
- Building and managing table relationships in Model view
- Creating a dedicated measures table to keep the model organized
- Writing foundational DAX measures: `SUM`, `DIVIDE`
- Understanding that measures respond dynamically to report filters and slicers

> **References:**
> - [Create DAX measures in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-measures)
> - [Model relationships in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-relationships-understand)

---

## Segment 6: Lab D — Publish to My Workspace & Explore the Power BI Service (30 minutes)

**Business Context:** Once a report is built in Power BI Desktop, publishing it to the Power BI Service makes it accessible from any browser without needing the Desktop app installed. Even with a free license, users can publish to their personal **My Workspace** to view and explore their reports online and pin visuals to a personal dashboard.

**Tool Used:** Power BI Desktop + Power BI Service (free My Workspace)

**Steps:**
1. In Power BI Desktop, click **Home → Publish** and sign in with your Microsoft account if prompted
2. Select **My Workspace** as the destination and click **Select**
3. Once published, click the link to open the report in **Power BI Service** ([app.powerbi.com](https://app.powerbi.com))
4. In the Power BI Service, navigate to **My Workspace** and open the published report
5. Interact with the report in the browser — test slicers and cross-filtering between visuals
6. Pin a visual to a dashboard: hover over the bar chart → click the **pin icon** → create a **New dashboard** named "Sales Overview"
7. Navigate to the dashboard and explore the pinned tile
8. Review the **Dataset** in My Workspace — click **Refresh now** to trigger a manual data refresh
9. Discuss: what changes with a Pro license (shared workspaces, scheduling refresh, sharing with colleagues)

**Key Learning Points:**
- Publishing a `.pbix` file from Power BI Desktop to My Workspace
- Navigating the Power BI Service: reports, datasets, and dashboards
- Pinning visuals to a personal dashboard
- Understanding the difference between a **report** and a **dashboard** in the Power BI Service
- Understanding free license boundaries and when Pro is required

> **References:**
> - [Publish from Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-upload-desktop-files)
> - [Power BI service basics](https://learn.microsoft.com/power-bi/fundamentals/service-basic-concepts)
> - [Power BI licensing](https://learn.microsoft.com/power-bi/fundamentals/service-features-license-type)

---

## Segment 7: Wrap-Up, Q&A & Next Steps (10 minutes)

- Recap of what was built today
- Discuss report management: renaming, deleting, and updating reports in My Workspace
- Encourage participants to identify one dataset in their own work they could visualize in Power BI
- Next steps and continued learning

### Recommended Learning Resources
- [Power BI learning paths on Microsoft Learn](https://learn.microsoft.com/training/powerplatform/power-bi)
- [Power BI documentation](https://learn.microsoft.com/power-bi/)
- [Power BI community](https://community.powerbi.com/)
- [Power BI sample datasets](https://learn.microsoft.com/power-bi/create-reports/sample-datasets)
- [DAX function reference](https://learn.microsoft.com/dax/dax-function-reference)
- [Power Query M function reference](https://learn.microsoft.com/powerquery-m/power-query-m-function-reference)

---

## Trainer Preparation Checklist

- [ ] All participants have **Power BI Desktop** installed ([aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore))
- [ ] All participants can sign in to [app.powerbi.com](https://app.powerbi.com) with their Microsoft account (free sign-up is sufficient)
- [ ] **Lab A & B:** A sample Excel workbook is prepared and distributed with two sheets:
  - Sheet 1 — **Sales** table (columns: Date, Region, Product, Sales Amount, Units Sold) with at least 50 rows of realistic sample data
  - Sheet 2 — **Product** lookup table (columns: Product, Category, Unit Cost)
- [ ] **Lab B:** Confirm the Excel file includes at least a few blank rows and one or more inconsistent column names to clean up
- [ ] **Lab C:** Confirm the Sales and Product tables have a matching **Product** column that Power BI can use for a relationship
- [ ] **Lab D:** Participants are signed in to Power BI Service before the session to avoid sign-in delays
- [ ] Trainer has tested the full publish flow from Desktop to My Workspace in the training tenant
- [ ] Participants understand that **My Workspace** is personal — they will not see each other's published reports
- [ ] Backup static screenshots prepared in case of connectivity issues during the Publish lab
