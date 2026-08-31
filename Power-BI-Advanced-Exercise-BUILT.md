---
  title: 'Lab Power BI Advanced: Modeling, DAX, Security, and AI'
  module: 'Power BI Advanced Lab'
---

# Lab Power BI Advanced – Advanced Modeling, DAX, Security, and AI

## Lab Introduction

In this advanced lab, you will extend the sales analytics solution you built in the *Power BI Intermediate* lab into an enterprise-grade Power BI solution. You will move beyond basic measures and visuals to master advanced DAX patterns, reusable calculation logic, dynamic report experiences, data security, AI-assisted analytics, and performance optimization.

This lab assumes you have already completed the *Power BI Intermediate* lab and have a working `SalesAnalysis.pbix` file containing the Northwind star schema (Customers, Orders, OrderDetails, Products, and a Date table) with the `Total Revenue`, `Total Orders`, and time-intelligence measures. If you do not have that file, you can rebuild the data model by following Tasks 2–3 and Task 7 of the intermediate lab before starting here.

This lab will guide you through the following tasks:

- Write advanced DAX measures using variables, iterators, `CALCULATE`, and `RANKX`
- Create reusable **calculation groups** to apply time-intelligence logic to any measure
- Build **field parameters** to let users dynamically swap measures and dimensions
- Implement **row-level security (RLS)** with static and dynamic roles, plus **object-level security (OLS)**
- Add **what-if parameters** for interactive scenario analysis
- Use **AI visuals** (Key influencers, Decomposition tree, Q&A, Smart narrative) and **Copilot**
- Configure **drillthrough** pages and **report page tooltips**
- Optimize report and model performance with **Performance analyzer**
- Prepare the solution for enterprise deployment (apps, deployment pipelines, incremental refresh)
- Govern content with **sensitivity labels** and **endorsement**
- Reuse data across teams with **shared semantic models**

## Pre-requisites

- **Power BI Desktop installed** (latest version): Download from [Microsoft's website](https://powerbi.microsoft.com/desktop). Advanced authoring features such as native calculation groups and field parameters require a recent build.
- **Completion of the Power BI Intermediate lab** (or an equivalent `SalesAnalysis.pbix` with the Northwind star schema and Date table).
- **Power BI Pro license**: Required to publish, share, and collaborate.
- **Power BI Service account**: Sign up at [https://app.powerbi.com](https://app.powerbi.com).
- **Premium Per User (PPU) or Premium capacity** *(optional)*: Required only for the enterprise features in Task 9 (deployment pipelines and incremental refresh). These steps are marked as optional where a Premium license is needed.
- **Internet connection**: Required for the Northwind OData service and the Power BI Service.

## Estimated Timing: 215 minutes

## Lab Scenario

Your sales analytics solution has proven valuable, and stakeholders now want more. Regional sales managers should only see the data for their own territory. Analysts want to compare measures dynamically without you building dozens of visuals. Executives want AI to surface *why* revenue changed, not just *what* changed. And the IT team needs the solution to perform well and follow a proper development-to-production release process.

Your task is to evolve the report into a secure, high-performing, self-service analytics solution by:

- Building advanced DAX to rank customers/products and calculate contribution
- Centralizing time-intelligence logic with calculation groups
- Letting users choose their own metrics and dimensions with field parameters
- Restricting data by region using row-level security
- Adding scenario modeling with what-if parameters
- Embedding AI-driven insights
- Improving usability with drillthrough and tooltip pages
- Tuning performance and preparing an enterprise release

## Architecture Overview

```
Power BI Advanced Lab Architecture
==================================

Existing Data Model (from Intermediate lab)
├── Fact Tables:  Orders, OrderDetails
├── Dimension Tables:  Customers, Products, Date (marked as date table)
└── Base Measures:  Total Revenue, Total Orders, Average Order Value,
                    Revenue YTD, Revenue Last Year, YoY Growth %

                    ↓ (Advanced modeling layer)

Advanced Model Enhancements
├── Advanced DAX Measures
│   ├── Customer Rank (RANKX)
│   ├── Product Rank (RANKX)
│   ├── Revenue % of Total (ALLSELECTED)
│   └── Running Total Revenue (variables + iterators)
│
├── Calculation Group:  "Time Intelligence"
│   ├── Current
│   ├── YTD
│   ├── Prior Year
│   └── YoY %
│
├── Field Parameters
│   ├── "Metric Selector"  (Total Revenue | Total Orders | Avg Order Value)
│   └── "Dimension Selector"  (Country | Category | Customer)
│
├── Security
│   ├── Row-Level Security roles
│   │   ├── Static role:  "USA Only"
│   │   └── Dynamic role:  "Region Manager" (USERPRINCIPALNAME)
│   └── Object-Level Security (hide column/table via Tabular Editor)
│
└── What-if Parameter:  "Discount Scenario %" (0%–30%)

                    ↓ (Report experience layer)

Report Enhancements
├── AI Visuals
│   ├── Key influencers (what drives revenue)
│   ├── Decomposition tree (revenue breakdown)
│   ├── Q&A (natural language)
│   ├── Smart narrative (auto-generated summary)
│   └── Copilot (create pages, summaries, DAX) *Fabric capacity*
│
├── Navigation & Detail
│   ├── Drillthrough page:  Customer Detail
│   └── Report page tooltip:  Revenue mini-summary
│
└── Optimization
    └── Performance analyzer → tune slow visuals / DAX

                    ↓ (Enterprise deployment layer)

Power BI Service
├── Workspace (Dev → Test → Prod via Deployment Pipeline) *Premium*
├── Published Power BI App
├── Governance:  Sensitivity labels + Endorsement (Promoted/Certified)
├── Shared Semantic Model (Build permission → live connections)
├── Copilot experiences *Fabric capacity*
└── Incremental refresh policy *Premium*
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following advanced Power BI skills:

- Task 1: Advanced DAX with variables, iterators, and RANKX (25 minutes)
- Task 2: Create and use calculation groups (25 minutes)
- Task 3: Build field parameters for dynamic reports (20 minutes)
- Task 4: Row-level security, dynamic RLS, and object-level security (30 minutes)
- Task 5: What-if parameters for scenario analysis (15 minutes)
- Task 6: AI visuals and Copilot capabilities (25 minutes)
- Task 7: Drillthrough pages and report page tooltips (20 minutes)
- Task 8: Optimize performance with Performance analyzer (15 minutes)
- Task 9: Deployment pipelines and report governance (25 minutes)
- Task 10: Share semantic models for reuse (20 minutes)

---

## Task 1: Advanced DAX with Variables, Iterators, and RANKX

Advanced DAX unlocks calculations that go far beyond simple sums. In this task you will use variables (`VAR`/`RETURN`) for readable, efficient measures, iterator functions (`SUMX`, `RANKX`), and context-modifying functions (`CALCULATE`, `ALL`, `ALLSELECTED`) to rank entities and calculate contribution.

> **Why variables?** Variables are evaluated once and reused, which improves readability and performance. They also lock in a value at a specific point in the filter context, avoiding subtle context-transition bugs.
>
> *Reference: [VAR keyword (DAX)](https://learn.microsoft.com/dax/var-dax)* and *[Use variables to improve your DAX formulas](https://learn.microsoft.com/dax/best-practices/dax-variables)*

### Open Your Solution

1. Open your existing **SalesAnalysis.pbix** file in Power BI Desktop.

2. Switch to **Report** view and confirm your base measures (`Total Revenue`, `Total Orders`, `Average Order Value`) exist in the Fields pane.

3. Save a copy for this lab: select **File** → **Save As** and name it **SalesAnalysis-Advanced.pbix**.

### Create a Ranking Measure with RANKX

You will rank customers by revenue so reports can highlight the highest-value accounts.

1. Right-click the **OrderDetails** table in the Fields pane and select **New measure**.

2. Enter the following DAX:
   ```
   Customer Rank =
   IF(
       HASONEVALUE(Customers[companyName]),
       RANKX(
           ALLSELECTED(Customers[companyName]),
           [Total Revenue],
           ,
           DESC,
           Dense
       )
   )
   ```

   > **What this does:** `RANKX` iterates over all customers currently selected in the report (`ALLSELECTED` respects slicers but ignores the row's own filter), evaluates `Total Revenue` for each, and returns the rank in descending order. `HASONEVALUE` ensures a rank is only returned when a single customer is in context, so totals stay blank.
   >
   > *Reference: [RANKX function (DAX)](https://learn.microsoft.com/dax/rankx-function-dax)*

3. Press **Enter**.

### Create a Product Ranking Measure

1. Right-click the **OrderDetails** table and select **New measure**.

2. Enter:
   ```
   Product Rank =
   IF(
       HASONEVALUE(Products[productName]),
       RANKX(
           ALLSELECTED(Products[productName]),
           [Total Revenue],
           ,
           DESC,
           Dense
       )
   )
   ```

3. Press **Enter**.

### Create a "Percent of Total" Measure

This measure shows each item's contribution to the overall revenue, which is essential for Pareto/ABC analysis.

1. Right-click the **OrderDetails** table and select **New measure**.

2. Enter the following DAX using variables:
   ```
   Revenue % of Total =
   VAR CurrentRevenue = [Total Revenue]
   VAR TotalAllRevenue =
       CALCULATE([Total Revenue], ALLSELECTED())
   RETURN
       DIVIDE(CurrentRevenue, TotalAllRevenue, 0)
   ```

   > **What this does:** `ALLSELECTED()` with no arguments removes filters coming from the visual's own rows (for example each row of a table) while keeping outer filters such as slicers, giving you the "visible total" as the denominator.
   >
   > *Reference: [ALLSELECTED function (DAX)](https://learn.microsoft.com/dax/allselected-function-dax)* and *[CALCULATE function (DAX)](https://learn.microsoft.com/dax/calculate-function-dax)*

3. Press **Enter**.

4. Format the measure as a percentage: with the measure selected, on the **Measure tools** ribbon set **Format** to **Percentage** with **2** decimal places.

### Create a Running Total with an Iterator

1. Right-click the **OrderDetails** table and select **New measure**.

2. Enter:
   ```
   Running Total Revenue =
   VAR MaxDate = MAX('Date'[Date])
   RETURN
       CALCULATE(
           [Total Revenue],
           FILTER(
               ALLSELECTED('Date'[Date]),
               'Date'[Date] <= MaxDate
           )
       )
   ```

   > **What this does:** `MAX('Date'[Date])` captures the last date in the current context (for example, the current point on a line chart). The `FILTER` then re-includes every date up to and including that point, producing a cumulative total.
   >
   > *Reference: [FILTER function (DAX)](https://learn.microsoft.com/dax/filter-function-dax)*

3. Press **Enter**.

### Visualize the Advanced Measures

1. Navigate to the **Customer Analysis** page.

2. Select the **Top Customers by Revenue** table and add the **Customer Rank** and **Revenue % of Total** measures to it.

3. Navigate to the **Product Performance** page and add **Product Rank** and **Revenue % of Total** to the **Product Sales Performance** table.

4. Navigate to the **Sales Overview** page. Add an **Area chart**:
   - **X-axis**: **Date** (from the Date table)
   - **Y-axis**: **Running Total Revenue**
   - Add title: **Cumulative Revenue**

5. Press **Ctrl + S** to save.

> **Congratulations!** You've written advanced DAX using variables, iterators, and ranking functions that power richer, context-aware analytics.

---

## Task 2: Create and Use Calculation Groups

Calculation groups let you define a calculation (like YTD or Prior Year) **once** and apply it to **any** measure, instead of authoring near-identical time-intelligence measures for every metric. They are ideal when you have many measures that each need the same set of time calculations.

> **What is a calculation group?** A calculation group is a special table containing **calculation items** — reusable expressions that use the `SELECTEDMEASURE()` placeholder to transform whatever measure the user has placed in the visual.
>
> *Reference: [Calculation groups](https://learn.microsoft.com/analysis-services/tabular-models/calculation-groups)* and *[Create calculation groups in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-calculation-groups)*

### Create the Calculation Group

1. Switch to **Model** view.

2. Open the **Model explorer** in the **Data** pane (select the model/semantic model node at the top of the pane).

3. Right-click **Calculation groups** and select **New calculation group**.

   > **Discourage implicit measures prompt:** When you create your first calculation group, Power BI recommends turning off implicit measures. Select **Yes** — calculation groups can produce unexpected results when combined with implicit (drag-and-drop aggregation) measures.

4. Rename the calculation group to **Time Intelligence** and rename its column to **Time Calc**.

### Add Calculation Items

You will add four calculation items. Each uses `SELECTEDMEASURE()` so it works with any base measure.

1. Select the calculation group and add a calculation item named **Current** with this expression:
   ```
   SELECTEDMEASURE()
   ```

2. Add a calculation item named **YTD**:
   ```
   TOTALYTD(SELECTEDMEASURE(), 'Date'[Date])
   ```

3. Add a calculation item named **Prior Year**:
   ```
   CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))
   ```

4. Add a calculation item named **YoY %**:
   ```
   VAR CurrentValue = SELECTEDMEASURE()
   VAR PriorValue =
       CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))
   RETURN
       DIVIDE(CurrentValue - PriorValue, PriorValue)
   ```

   > *Reference: [SELECTEDMEASURE function (DAX)](https://learn.microsoft.com/dax/selectedmeasure-function-dax)*

### Set the Ordinal (Sort Order)

By default calculation items sort alphabetically. Set a logical order so they appear as Current → YTD → Prior Year → YoY %.

1. Select the **Current** calculation item. In the **Properties** pane, set **Ordinal** to **0**.

2. Set **YTD** to **1**, **Prior Year** to **2**, and **YoY %** to **3**.

### Apply Dynamic Formatting to the YoY % Item (Optional)

The `YoY %` item returns a ratio but inherits the base measure's format. Add a dynamic format string so it always displays as a percentage.

1. Select the **YoY %** calculation item.

2. In the **Properties** pane, turn on **Dynamic format string** and enter:
   ```
   "0.0%"
   ```

   > *Reference: [Dynamic format strings for measures](https://learn.microsoft.com/power-bi/create-reports/desktop-dynamic-format-strings)*

### Use the Calculation Group in a Report

1. Switch to **Report** view and navigate to the **Sales Overview** page.

2. Add a **Matrix** visual:
   - **Rows**: **Year-Month** (from the Date table)
   - **Columns**: drag the **Time Calc** column from the **Time Intelligence** calculation group
   - **Values**: **Total Revenue**

3. The matrix now shows **Current, YTD, Prior Year, and YoY %** columns — all generated from a single measure. Try changing **Total Revenue** to **Total Orders** in the Values well and notice the same four calculations apply automatically.

4. Add title: **Revenue Time Intelligence Matrix** and press **Ctrl + S**.

> **Congratulations!** You built a calculation group that applies reusable time-intelligence logic to any measure — a key enterprise modeling pattern that dramatically reduces measure sprawl.

---

## Task 3: Build Field Parameters for Dynamic Reports

Field parameters let report consumers **choose which measure or dimension** a visual displays, using a slicer — without you building a separate visual for each option.

> **What are field parameters?** A field parameter is a special table listing fields (measures or columns). Placing it on a slicer lets users switch what a visual shows at runtime.
>
> *Reference: [Create and use field parameters](https://learn.microsoft.com/power-bi/create-reports/power-bi-field-parameters)*

### Enable the Feature (if required)

Field parameters are generally available in current Power BI Desktop. If you do not see the option, enable it under **File** → **Options and settings** → **Options** → **Preview features** → **Field parameters**, then restart Power BI Desktop.

### Create a Metric Field Parameter

1. Switch to **Report** view.

2. On the **Modeling** ribbon, select **New parameter** → **Fields**.

3. In the dialog:
   - **Name**: `Metric Selector`
   - Ensure **Add slicer to this page** is checked.
   - From the Fields list, add these measures in order: **Total Revenue**, **Total Orders**, **Average Order Value**.

4. Select **Create**. Power BI generates a `Metric Selector` table and adds a slicer to the page.

   > **Tip:** You can reorder or rename the entries by editing the generated DAX table definition in the formula bar — each row is `("Display Name", NAMEOF('Table'[Measure]), ordinal)`.

### Create a Dimension Field Parameter

1. On the **Modeling** ribbon, select **New parameter** → **Fields** again.

2. In the dialog:
   - **Name**: `Dimension Selector`
   - Check **Add slicer to this page**.
   - Add these columns: **ShipCountry** (Orders), **categoryID** (Products), and **companyName** (Customers).

3. Select **Create**.

### Build a Fully Dynamic Chart

1. Add a **Clustered column chart** to the page.

2. Configure it to use both parameters:
   - **X-axis**: drag the **Dimension Selector** field parameter
   - **Y-axis**: drag the **Metric Selector** field parameter

3. Add title: **Dynamic Analysis** (or leave the default, which reflects the selection).

4. Test it: use the **Metric Selector** slicer to switch between Revenue, Orders, and Avg Order Value, and use the **Dimension Selector** slicer to switch the breakdown between Country, Category, and Customer. A single visual now answers many questions.

5. Press **Ctrl + S** to save.

> **Congratulations!** You created field parameters that turn one chart into a flexible, self-service analysis tool.

---

## Task 4: Row-Level Security, Dynamic RLS, and Object-Level Security (OLS)

Row-level security restricts the data a given user can see by applying DAX filters to roles. You will create a **static** role (fixed filter) and a **dynamic** role (filter based on the signed-in user), then add **object-level security (OLS)** to hide an entire column or table from a role.

> **How RLS works:** You define roles with DAX filter expressions in Power BI Desktop, then assign users or security groups to those roles in the Power BI Service after publishing. Filters propagate through relationships just like slicers.
>
> *Reference: [Row-level security (RLS) with Power BI](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls)*

### Create a Static Role

1. Switch to **Report** or **Model** view.

2. On the **Modeling** ribbon, select **Manage roles**.

3. In the **Manage roles** dialog, select **Create** and name the role **USA Only**.

4. Under **Select tables**, choose the **Orders** table and add the following filter expression for **ShipCountry**:
   ```
   [ShipCountry] = "USA"
   ```

   > **Note:** Because the base measures aggregate through relationships from Orders → OrderDetails, filtering Orders to USA also limits the revenue and product data shown to that role.

5. Select **Save**.

### Test the Static Role

1. On the **Modeling** ribbon, select **View as**.

2. In the **View as roles** dialog, check **USA Only** and select **OK**.

3. A banner appears at the top: *"Now viewing report as: USA Only."* Navigate the report pages and confirm only USA data is displayed.

4. Select **View as** again and clear the selection (or select **None**) to stop testing.

   > *Reference: [Validate the roles within Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls#validate-the-roles-within-power-bi-desktop)*

### Create a Dynamic Role Using USERPRINCIPALNAME

Dynamic RLS filters data based on the identity of the signed-in user, so a single role serves many users. This requires a mapping between users and the territory they manage. For this lab you will add a small mapping table.

1. On the **Home** ribbon, select **Enter data**.

2. Create a table named **UserRegion** with two columns, **UserEmail** and **Country**, and add a few rows mapping example users to countries. For testing, include your **own** Power BI account email mapped to a country that has data (for example `USA`):

   | UserEmail | Country |
   |---|---|
   | your.name@yourcompany.com | USA |
   | manager.eu@yourcompany.com | GERMANY |
   | manager.latam@yourcompany.com | BRAZIL |

3. Select **Load**.

4. On the **Modeling** ribbon, select **Manage roles** → **Create**, and name the role **Region Manager**.

5. Select the **Orders** table and enter this filter expression on **ShipCountry**:
   ```
   [ShipCountry] =
   LOOKUPVALUE(
       UserRegion[Country],
       UserRegion[UserEmail], USERPRINCIPALNAME()
   )
   ```

   > **What this does:** `USERPRINCIPALNAME()` returns the signed-in user's UPN (email). `LOOKUPVALUE` finds that user's assigned country in the mapping table and filters Orders to it. Add more rows to `UserRegion` to onboard new managers without changing the model.
   >
   > *Reference: [USERPRINCIPALNAME function (DAX)](https://learn.microsoft.com/dax/userprincipalname-function-dax)*

6. Select **Save**.

### Test the Dynamic Role

1. Select **View as** on the Modeling ribbon.

2. Check both **Region Manager** and **Other user**, and in the **Other user** box type one of the emails from your mapping table (for example `manager.eu@yourcompany.com`).

3. Select **OK**. Confirm the report now shows only that manager's country (Germany).

4. Turn off **View as** when finished.

### Publish and Assign Members (in the Service)

1. Publish the report to a workspace using **Publish**.

2. In the Power BI Service, open the workspace, hover over the **SalesAnalysis-Advanced** semantic model, select **More options (...)** → **Security**.

3. Select a role (for example **Region Manager**), add the users or security groups that belong to it, and select **Save**.

   > **Important:** RLS applies to users with the **Viewer** role in a workspace, or those consuming a published **app**. Workspace **Admins, Members, and Contributors** are not restricted by RLS. Test with a Viewer account.
   >
   > *Reference: [Managing security on your model / assign members](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls#managing-security-on-your-model)*

### Add Object-Level Security (OLS) with Tabular Editor

While RLS filters **rows**, object-level security hides an entire **table or column** so that, for a restricted role, the object appears not to exist. Power BI Desktop has no built-in UI for OLS, so you configure it with the free **Tabular Editor** external tool.

> **What is OLS?** OLS secures model **metadata**: a secured column or table is removed from the model for members of the role, and any visual that references it shows an error for those users. OLS is often combined with RLS for defense in depth.
>
> *Reference: [Object-level security (OLS)](https://learn.microsoft.com/power-bi/enterprise/service-admin-ols-object-level-security)*

1. Install **Tabular Editor** (the free version 2 is sufficient) from [tabulareditor.com](https://tabulareditor.com). It registers under the **External Tools** ribbon in Power BI Desktop.

2. In Power BI Desktop, on the **External Tools** ribbon, select **Tabular Editor**. It opens connected to your model.

3. In Tabular Editor, create a role for OLS testing: right-click **Roles** → **New Role**, and name it **No Freight Access**. Set its **Row Level Security** to leave all tables unfiltered (this role only restricts an object).

4. Expand **Tables** → **Orders** → **Columns** and select the **Freight** column.

5. In the **Properties** grid, find **Object Level Security**, expand it, and set the **No Freight Access** role to **None** (the other value, **Default**, leaves the column visible).

6. Save the changes back to the model: in Tabular Editor select **File** → **Save** (or press **Ctrl + S**). Return to Power BI Desktop.

7. Test with **View as** → **No Freight Access**. Any visual that uses **Freight** now returns an error for that role, and the field is hidden in the field list — confirming the column is secured.

   > **Note:** Because a secured object breaks visuals that reference it, plan reports so restricted roles do not depend on secured fields. Assign members to OLS roles in the Service the same way as RLS (semantic model → **Security**).

> **Congratulations!** You implemented static and dynamic row-level security **and** object-level security — layered protection for rows *and* model objects.

---

## Task 5: What-If Parameters for Scenario Analysis

What-if parameters create a numeric slicer that feeds a value into your DAX, letting users model scenarios interactively (for example, "what if we increased the discount by 10%?").

> **What is a what-if parameter?** It generates a single-column table of numbers plus a measure that returns the user's selected value (`SELECTEDVALUE`). You reference that measure in your own calculations.
>
> *Reference: [Create and use a what-if parameter to visualize variables](https://learn.microsoft.com/power-bi/transform-model/desktop-what-if)*

### Create the Parameter

1. Switch to **Report** view and navigate to the **Sales Overview** page (or add a new **Scenario Analysis** page).

2. On the **Modeling** ribbon, select **New parameter** → **Numeric range**.

3. Configure:
   - **Name**: `Discount Scenario %`
   - **Data type**: **Decimal number**
   - **Minimum**: `0`
   - **Maximum**: `0.30`
   - **Increment**: `0.05`
   - **Default**: `0`
   - Check **Add slicer to this page**.

4. Select **Create**. Power BI creates a `Discount Scenario %` table, a `Discount Scenario % Value` measure, and a slicer.

### Create a Scenario Measure

1. Right-click the **OrderDetails** table and select **New measure**.

2. Enter:
   ```
   Revenue What-If =
   SUMX(
       OrderDetails,
       OrderDetails[UnitPrice]
           * OrderDetails[Quantity]
           * (1 - OrderDetails[Discount] - 'Discount Scenario %'[Discount Scenario % Value])
   )
   ```

   > **What this does:** It recalculates revenue applying an **additional** discount chosen on the slicer, on top of each line's existing discount, so users can see the revenue impact of a promotional scenario in real time.

3. Press **Enter**.

### Visualize the Scenario

1. Add a **Card** visual and drag **Revenue What-If** onto it. Add title: **Projected Revenue (Scenario)**.

2. Place it next to the existing **Total Revenue** card so users can compare the baseline with the scenario.

3. Add a **Clustered column chart** comparing **Total Revenue** and **Revenue What-If** by **Year-Month** to visualize the gap.

4. Move the **Discount Scenario %** slicer to a discount value and watch both the card and chart update.

5. Press **Ctrl + S** to save.

> **Congratulations!** You added interactive scenario modeling that lets stakeholders explore "what-if" outcomes without touching the data.

---

## Task 6: AI Visuals and Copilot Capabilities

Power BI's AI visuals help users understand *why* something happened and explore data with natural language, while **Copilot** uses generative AI to help you build and summarize content.

> *Reference: [Create Key influencers visualizations](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-influencers)*, *[Create and view a decomposition tree](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-decomposition-tree)*, *[Intro to Q&A](https://learn.microsoft.com/power-bi/natural-language/q-and-a-intro)*, and *[Create smart narrative summaries](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-smart-narrative)*

### Add a New AI Insights Page

1. At the bottom of the report, select **+** to add a page and rename it **AI Insights**.

### Key Influencers

1. Add a **Key influencers** visual from the Visualizations pane.

2. Configure:
   - **Analyze**: drag **Freight** from the Orders table (a numeric value to explain).
   - **Explain by**: drag **ShipCountry** (Orders) and **categoryID** (Products).

3. Review the **Key influencers** and **Top segments** tabs. The visual ranks which factors most influence the analyzed value.

   > **Tip:** For best results, "Analyze" a meaningful metric and provide several categorical "Explain by" fields. Key influencers respects slicers and filters on the page.

### Decomposition Tree

1. Add a **Decomposition tree** visual.

2. Configure:
   - **Analyze**: **Total Revenue** measure
   - **Explain by**: **ShipCountry** (Orders), **categoryID** (Products), and **companyName** (Customers)

3. Select the **+** on the root node and choose a field (or **High value** / **Low value** for AI splits) to drill into the drivers of revenue interactively.

   > **AI splits:** Choosing **High value** or **Low value** lets Power BI pick the dimension that most increases or decreases the measure at each level.
   >
   > *Reference: [Use AI splits in the decomposition tree](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-decomposition-tree#use-the-ai-splits)*

### Q&A Visual

1. Add a **Q&A** visual.

2. In the question box, type a natural-language question such as `total revenue by country` or `top 5 customers by total revenue` and press **Enter**.

3. Power BI interprets the question and renders an appropriate visual. Use the suggested questions to explore.

   > **Improve Q&A:** As the model author you can add synonyms and review questions under **Modeling** → **Q&A setup** to make natural-language queries more accurate.
   >
   > *Reference: [Introduction to Q&A tooling to train Power BI Q&A](https://learn.microsoft.com/power-bi/natural-language/q-and-a-tooling-intro)*

### Smart Narrative

1. Add a **Smart narrative** visual.

2. Power BI auto-generates a text summary of the page's data (trends, highs/lows, totals).

3. While editing the narrative, select **+ Value** to insert a dynamic value tied to a measure (for example, insert `Total Revenue`), so the paragraph updates as filters change.

4. Press **Ctrl + S** to save.

### Explore Copilot in Power BI

Copilot brings generative AI into report authoring — it can create report pages, summarize data, and help you write and explain DAX.

> **Prerequisite:** Copilot requires a workspace backed by **Fabric or Power BI Premium capacity** (F2 or higher, or a P SKU) with **Copilot enabled** in the tenant admin settings. If your environment does not have capacity, review this section for awareness and skip the hands-on steps.
>
> *Reference: [Copilot in Power BI (overview)](https://learn.microsoft.com/power-bi/create-reports/copilot-introduction)*

1. **Create a report page with Copilot.** Open the report in the **Power BI Service** (or Desktop) and select **Copilot** on the ribbon/toolbar. In the Copilot pane, choose **Create a page** (or type a prompt) such as: `Create a page analyzing revenue by country and product category`. Copilot proposes a page you can keep and edit.

2. **Summarize a page.** In the Copilot pane, ask `Summarize the key insights on this page`. Copilot generates a narrative highlighting trends, top contributors, and outliers.

3. **Write DAX with Copilot.** Open **DAX query view**, select **Copilot**, and prompt: `Write a measure for revenue year-over-year growth using the Date table`. Review the suggested DAX, then **Keep** it to add the measure. You can also ask Copilot to **explain** an existing measure.

   > *Reference: [Create reports with Copilot in Power BI](https://learn.microsoft.com/power-bi/create-reports/copilot-create-report-service)* and *[Write DAX queries with Copilot](https://learn.microsoft.com/power-bi/transform-model/dax-query-view#write-dax-queries-with-copilot)*

4. Press **Ctrl + S** to save.

> **Congratulations!** Your report now surfaces AI-driven insights, supports natural-language exploration, and leverages Copilot to accelerate authoring.

---

## Task 7: Drillthrough Pages and Report Page Tooltips

Drillthrough lets users right-click a data point and jump to a detail page filtered to that item. Report page tooltips let you show a rich, custom visual as a hover tooltip.

> *Reference: [Set up drillthrough in Power BI reports](https://learn.microsoft.com/power-bi/create-reports/desktop-drillthrough)* and *[Create tooltips based on report pages](https://learn.microsoft.com/power-bi/create-reports/desktop-tooltips)*

### Create a Customer Detail Drillthrough Page

1. Add a new page and rename it **Customer Detail**.

2. Add visuals that describe a single customer, for example:
   - A **Card** with **companyName**
   - **Cards** with **Total Revenue** and **Total Orders**
   - A **Table** of **productName** and **Total Revenue**
   - A **Line chart** of **Total Revenue** by **Date**

3. In the **Visualizations** pane (with no visual selected), locate the **Drill through** section and drag **companyName** (Customers) into the **Add drill-through fields here** well.

4. A **Back** button is added automatically to return to the source page.

   > **Tip:** Turn on **Keep all filters** in the drillthrough setup so filters from the source page carry over.

### Use the Drillthrough

1. Navigate to the **Customer Analysis** page.

2. Right-click a customer in the **Top Customers by Revenue** table.

3. Select **Drill through** → **Customer Detail**. The detail page opens filtered to that customer.

4. Select the **Back** button (hold **Ctrl** and click) to return.

### Create a Report Page Tooltip

1. Add a new page and rename it **Revenue Tooltip**.

2. Select the page background, then in the page's **Format** pane → **Page information**, turn **Allow use as tooltip** to **On**.

3. In **Format** → **Canvas settings**, set **Type** to **Tooltip** (this sizes the page as a small tooltip canvas).

4. On this small page, add a compact visual, for example a **Card** with **Total Revenue** and a small **line chart** of **Total Revenue** by **Date**.

5. Assign the tooltip to a visual: navigate to the **Sales Overview** page, select the **Orders by Country** chart, open its **Format** pane → **General** → **Tooltips**, set **Type** to **Report page**, and set **Page** to **Revenue Tooltip**.

6. Hover over a bar in the chart — your custom tooltip page appears, filtered to that bar's context.

7. Press **Ctrl + S** to save.

> **Congratulations!** You added drillthrough detail navigation and custom report-page tooltips for a richer, more guided user experience.

---

## Task 8: Optimize Performance with Performance Analyzer

As reports grow, performance matters. **Performance analyzer** records how long each visual takes to render and how much of that time is spent in the DAX query, so you can find and fix bottlenecks.

> *Reference: [Use Performance analyzer to examine report element performance](https://learn.microsoft.com/power-bi/create-reports/desktop-performance-analyzer)*

### Record a Performance Trace

1. On the **View** ribbon, select **Performance analyzer**.

2. In the **Performance analyzer** pane, select **Start recording**.

3. Select **Refresh visuals** to reload every visual on the current page (or interact with slicers to capture specific actions).

4. Expand each visual in the pane to see the breakdown:
   - **DAX query** — time to run the query in the engine
   - **Visual display** — time to render
   - **Other** — background/overhead

   > **Interpreting results:** A high **DAX query** time points to a measure or model issue; a high **Visual display** time points to a visual with too much data or too many elements.

### Investigate a Slow Query

1. Find the visual with the longest **DAX query** time and select **Copy query** next to it.

2. Paste this query into **DAX query view** in Power BI Desktop (or an external tool such as **DAX Studio**) to analyze it in depth.

   > *Reference: [DAX query view in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/dax-query-view)*

### Apply Optimization Best Practices

Review your model and report against these common optimizations:

- **Reduce columns/rows**: remove unused columns (you already trimmed several in the intermediate lab). A smaller model is faster.
- **Prefer measures over calculated columns** where possible; measures are evaluated only for what's on screen.
- **Avoid bidirectional relationships** unless required — they add query overhead.
- **Limit visuals per page**: each visual issues at least one query. Consolidate where sensible.
- **Use variables** in DAX to avoid recomputing the same expression (as you did in Task 1).
- **Turn off Auto date/time** for the model since you have a dedicated Date table.

   > *Reference: [Optimization guide for Power BI](https://learn.microsoft.com/power-bi/guidance/power-bi-optimization)* and *[Auto date/time guidance in Power BI Desktop](https://learn.microsoft.com/power-bi/guidance/auto-date-time)*

1. Turn off **Auto date/time**: select **File** → **Options and settings** → **Options** → **Data Load** (Current File), and uncheck **Auto date/time**.

2. Select **Close** in the Performance analyzer pane when done, and press **Ctrl + S** to save.

> **Congratulations!** You measured report performance and applied model optimizations that keep your solution fast as it scales.

---

## Task 9: Deployment Pipelines and Report Governance

Enterprise solutions need a controlled path from development to production, a strategy for refreshing large datasets efficiently, and governance controls that classify and endorse trusted content. This task introduces publishing an **app**, using **deployment pipelines**, configuring **incremental refresh**, and applying **governance** with sensitivity labels and endorsement.

> Some features in this task require **Premium Per User (PPU)** or **Premium/Fabric capacity** and are marked accordingly.

### Publish and Create a Power BI App

1. Publish your **SalesAnalysis-Advanced.pbix** to a standard **workspace** (apps require a workspace, not "My workspace"). Create one via **Workspaces** → **New workspace** if needed.

2. In the Power BI Service, open the workspace and select **Create app** (or **Update app**).

3. Configure the app's **Setup**, **Content**, and **Audience**:
   - Give the app a name and description.
   - Choose which reports/dashboards to include.
   - Define an **Audience** and add the users or groups who should consume it.

4. Select **Publish app**. Consumers get a clean, read-only experience, and **RLS from Task 4 is enforced** for app viewers.

   > *Reference: [Publish an app in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-create-distribute-apps)*

### Use a Deployment Pipeline *(Premium / PPU)*

Deployment pipelines provide **Development → Test → Production** stages so you can promote content safely.

1. In the Service, select **Deployment pipelines** → **Create pipeline**.

2. Assign your workspace to the **Development** stage.

3. Select **Deploy to Test**, review the comparison, and deploy. Repeat **Test → Production** when validated.

4. Configure **deployment rules** (for example, point the Production dataset at a production data source) as needed.

   > *Reference: [Introduction to deployment pipelines](https://learn.microsoft.com/power-bi/create-reports/deployment-pipelines-overview)* and *[Get started with deployment pipelines](https://learn.microsoft.com/power-bi/create-reports/deployment-pipelines-get-started)*

### Configure Incremental Refresh *(Premium / PPU)*

Incremental refresh refreshes only recently changed data instead of the entire dataset, making refreshes faster and more reliable for large tables.

1. In **Power BI Desktop**, on the **Home** ribbon open **Transform data** → **Manage parameters**, and create two **Date/Time** parameters named exactly **RangeStart** and **RangeEnd** (these reserved names are required).

2. In **Power Query Editor**, filter the **Orders** table's **OrderDate** using these parameters (for example, `OrderDate >= RangeStart` and `OrderDate < RangeEnd`), then **Close & Apply**.

3. In the Fields pane, right-click the **Orders** table and select **Incremental refresh**.

4. Turn on **Incrementally refresh this table**, set the archive window (for example, store rows for the last **5 years**) and the refresh window (for example, refresh rows in the last **10 days**), then **Apply**.

5. Publish to a **Premium/PPU** workspace and run a refresh — only the recent partition is refreshed.

   > *Reference: [Incremental refresh and real-time data for semantic models](https://learn.microsoft.com/power-bi/connect-data/incremental-refresh-overview)* and *[Configure incremental refresh](https://learn.microsoft.com/power-bi/connect-data/incremental-refresh-configure)*

### Apply Sensitivity Labels (Report Governance)

Sensitivity labels from **Microsoft Purview Information Protection** classify and protect content (for example, *Confidential*). Labels persist even when data is exported to Excel, PDF, or PowerPoint, enforcing protection downstream.

> **Prerequisite:** Sensitivity labels must be enabled and published by your Microsoft Purview / Microsoft 365 administrators, and enabled for Power BI in the tenant settings.
>
> *Reference: [Apply sensitivity labels in Power BI](https://learn.microsoft.com/power-bi/enterprise/service-security-apply-data-sensitivity-labels)*

1. In the **Power BI Service**, open your **SalesAnalysis-Advanced** report.

2. On the report toolbar, select **Sensitivity** and choose a label such as **Confidential**. (In Power BI Desktop, the **Sensitivity** button is on the **Home** ribbon.)

3. Apply the same label to the **semantic model** from the workspace item's **More options (...)** menu, so classification follows the data.

### Endorse Your Content (Promote and Certify)

Endorsement signals which content is trustworthy. **Promoted** highlights content you recommend; **Certified** marks an authoritative, reviewed "gold" source and can only be applied by users your admin authorizes.

> *Reference: [Endorsement: Promote or certify Power BI content](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content)*

1. In the workspace, hover over the **SalesAnalysis-Advanced** semantic model, select **More options (...)** → **Settings**.

2. Expand **Endorsement and discovery** and select **Promoted**. If you are an authorized certifier, you can instead select **Certified**.

3. Optionally turn on **Make discoverable** so users who lack access can find and request the model.

4. Select **Apply**. Endorsed content now displays a badge across the Service, helping consumers identify trusted data.

> **Congratulations!** You prepared the solution for enterprise release with app distribution, staged deployment, efficient incremental refresh, and governance through sensitivity labels and endorsement.

---

## Task 10: Share Semantic Models for Reuse

A **shared semantic model** is a published dataset that many reports connect to **live**, giving your organization one governed source of truth instead of duplicated models. You grant **Build** permission, and report authors connect without re-importing data. Security you defined in Task 4 (RLS and OLS) is automatically enforced on every downstream report.

> **Why share semantic models?** Centralizing the model reduces duplication, guarantees consistent measures and security, and lets analysts focus on building reports rather than re-modeling data.
>
> *Reference: [Use shared semantic models](https://learn.microsoft.com/power-bi/connect-data/service-datasets-share)*

### Grant Build Permission

1. In the **Power BI Service**, open the workspace containing the published **SalesAnalysis-Advanced** semantic model.

2. Hover over the semantic model, select **More options (...)** → **Manage permissions**.

3. Select **Add user**, enter a colleague's name or a security group, and grant the **Build** permission (in addition to Read). Build allows them to create new content on top of the model.

4. Select **Add**.

   > *Reference: [Build permission for shared semantic models](https://learn.microsoft.com/power-bi/connect-data/service-datasets-build-permissions)*

### Endorse and Make the Model Discoverable

1. If you have not already, endorse the semantic model as **Promoted** or **Certified** (see Task 9) and enable **Make discoverable** so authors across workspaces can find it.

   > *Reference: [Discover and connect to semantic models across workspaces](https://learn.microsoft.com/power-bi/connect-data/service-datasets-discover-across-workspaces)*

### Connect to the Shared Semantic Model (Live Connection)

1. Open a new **Power BI Desktop** file.

2. On the **Home** ribbon, select **Get data** → **Power BI semantic models**.

3. In the data hub, select the **SalesAnalysis-Advanced** model (endorsement badges help you identify it) and select **Connect**.

4. Power BI creates a **live connection**: the Fields pane shows the shared model's tables and measures, but no data is imported and you cannot change the model. Build a new report page (for example, a **Total Revenue** card and a **country** bar chart) directly on the shared model.

5. Publish this thin report. It reuses the central model — and any **RLS/OLS** roles and **measures** defined once are enforced everywhere.

   > *Reference: [Connect to semantic models in the Power BI Service from Power BI Desktop](https://learn.microsoft.com/power-bi/connect-data/desktop-report-lifecycle-datasets)*

> **Congratulations!** You published a governed, reusable semantic model and built a report on it via a live connection — the backbone of scalable, consistent self-service BI.

---

## Lab Summary

In this advanced lab, you transformed a basic sales report into a secure, high-performing, enterprise-ready Power BI solution.

### What You Accomplished

✅ **Task 1**: Wrote advanced DAX using variables, iterators (`SUMX`, `FILTER`), `RANKX`, and `ALLSELECTED` for ranking and contribution analysis

✅ **Task 2**: Built a **calculation group** to apply reusable time-intelligence logic to any measure

✅ **Task 3**: Created **field parameters** for user-driven metric and dimension selection

✅ **Task 4**: Implemented **row-level security** (static and dynamic with `USERPRINCIPALNAME`) and **object-level security** with Tabular Editor

✅ **Task 5**: Added a **what-if parameter** for interactive scenario modeling

✅ **Task 6**: Embedded **AI visuals** — Key influencers, Decomposition tree, Q&A, Smart narrative — and used **Copilot** to create pages, summaries, and DAX

✅ **Task 7**: Configured **drillthrough** pages and custom **report-page tooltips**

✅ **Task 8**: Used **Performance analyzer** and applied model/report optimizations

✅ **Task 9**: Prepared for **enterprise deployment** with apps, deployment pipelines, and incremental refresh, and applied **governance** via sensitivity labels and endorsement

✅ **Task 10**: Shared a **semantic model** with Build permission and built a report on it via a **live connection**

### Key Concepts You Learned

- **Advanced DAX**: variables for readability/performance, iterators, ranking, and context transition
- **Calculation groups**: `SELECTEDMEASURE()`, calculation items, ordinals, and dynamic format strings
- **Field parameters**: dynamic measures and dimensions driven by slicers
- **Row-level security**: static vs. dynamic RLS, testing with "View as", and assigning members in the Service
- **Object-level security**: hiding tables/columns per role with Tabular Editor
- **What-if parameters**: numeric-range tables feeding scenario measures
- **AI visuals & Copilot**: explaining drivers, decomposing metrics, natural-language Q&A, auto-generated narratives, and generative AI for pages, summaries, and DAX
- **Drillthrough and tooltips**: guided navigation and rich hover context
- **Performance**: measuring with Performance analyzer and applying optimization guidance
- **Enterprise deployment & governance**: apps, Dev/Test/Prod pipelines, incremental refresh, sensitivity labels, and endorsement
- **Shared semantic models**: Build permission, discovery, and live connections for reuse

### Next Steps

- **Composite models & aggregations**: combine Import and DirectQuery, and speed up DirectQuery with aggregations
- **Object-level security (OLS)** alongside RLS, using security groups
- **Dataflows and Datamarts** for reusable, centralized data preparation
- **XMLA endpoint** tooling (Tabular Editor, DAX Studio) for pro model development
- **Power BI REST APIs** and deployment automation for CI/CD
- **Fabric** integration: OneLake, Direct Lake mode, and semantic model best practices

### Additional Resources

- [Microsoft Power BI Documentation](https://learn.microsoft.com/power-bi/)
- [DAX reference](https://learn.microsoft.com/dax/)
- [Power BI guidance and best practices](https://learn.microsoft.com/power-bi/guidance/)
- [Row-level security (RLS) with Power BI](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls)
- [Calculation groups](https://learn.microsoft.com/analysis-services/tabular-models/calculation-groups)
- [Field parameters](https://learn.microsoft.com/power-bi/create-reports/power-bi-field-parameters)
- [DAX Guide (community)](https://dax.guide/)

**Thank you for completing this advanced lab!** You now have practical experience building secure, dynamic, AI-enhanced, and enterprise-ready Power BI solutions.