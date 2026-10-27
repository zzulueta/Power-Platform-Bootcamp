---
  title: 'Lab Power BI Intermediate (Online): Reports and Dashboards'
  module: 'Power BI Intermediate Lab - Browser Only'
---

# Lab Power BI Intermediate (Online) – Reports and Dashboards using only app.powerbi.com

## Lab Introduction

This lab is the browser-only version of [Power-BI-Intermediate-Exercise.md](./Power-BI-Intermediate-Exercise.md). You build the same end-to-end analytics solution, but **everything is done in the Power BI service at [https://app.powerbi.com](https://app.powerbi.com)**. You don't need to install Power BI Desktop (or anything else) on your machine.

This lab will guide you through the following tasks:

- Understand the Power BI service and how it replaces Power BI Desktop for authoring
- Connect to and import data from multiple sources (OData and CSV files) using Power Query Online
- Build a data model using star schema design with the web model editor
- Create interactive reports with visualizations, filters, and slicers using the browser report editor
- Save, explore, and refresh your content in a workspace
- Create a dashboard by pinning visuals from your report and share it
- Create a Date table and time intelligence measures with DAX
- Apply themes, conditional formatting, bookmarks, buttons, and page navigation

### What changes compared with the Desktop lab?

| Desktop lab | Online lab (this lab) |
| --- | --- |
| Get data → Power Query Editor in Desktop | **Create → Get data** (Power Query Online) in the service |
| Model view in Desktop | **Open semantic model** (web model editor) |
| Report view in Desktop | **Report editing view** in the browser |
| Save as `.pbix`, then Publish | Content is **saved directly to a workspace** – no `.pbix`, no publish step |
| Task 5 "Publish" | Task 5 "Save, explore and refresh" |

## Pre-requisites

- **A modern web browser** (Microsoft Edge, Chrome, Firefox or Safari). No Power BI Desktop required.
- **A Power BI account** that can sign in at [https://app.powerbi.com](https://app.powerbi.com).
- **A license that allows creating and sharing content**: Power BI Pro, Premium Per User, or a Microsoft Fabric trial/capacity workspace. Both you (the creator) and anyone you share with need a Pro (or equivalent) license unless the workspace is on a Fabric/Premium capacity.
- **Web data modeling enabled**: Editing semantic models in the service is on by default. If **Open semantic model** or **Edit** is disabled, ask your Power BI admin to check the tenant setting *Users can edit data models in the Power BI service*.
- **Sample data**: `products.csv` and `customers.csv` from [Power-BI-Intermediate-Ref/Data](./Power-BI-Intermediate-Ref/Data) downloaded to your computer. The browser uploads them for you.
- **Internet connection**: Required for the Northwind OData service (`https://services.odata.org/v4/northwind/northwind.svc/`).

## Estimated Timing: 150 minutes

## Lab Scenario

Your organization wants to analyze sales data to gain insights into product performance, customer behavior, and overall sales trends. The data exists in multiple locations:

- **Order transactions** and **order line items** are available through a Northwind OData service (a public demo service)
- **Product catalog** data is maintained in a CSV file
- **Customer information** is stored in a separate CSV file

Your task is to:

- Connect to these disparate sources and bring the data into the Power BI service
- Create a unified data model that relates orders to customers and products
- Build interactive reports that answer:
  - What are our total sales and how many orders have we processed?
  - Which customers are generating the most revenue?
  - Which products are our top sellers?
  - How are sales distributed geographically?
- Create a dashboard that highlights the most important KPIs
- Share the dashboard with your team

## Architecture Overview

```
Power BI Lab Architecture (100% online)
=======================================

Data Sources
├── Northwind OData Service (https://services.odata.org/v4/northwind/northwind.svc/)
│   ├── Orders table (order header data)
│   └── Order_Details table (order line item data)
│
└── CSV Files (uploaded from your computer in the browser)
    ├── products.csv (product catalog)
    └── customers.csv (customer information)

                    ↓ (Create → Get data → Power Query Online / Import Mode)

Power BI Service (https://app.powerbi.com)
├── Workspace (My workspace)
│   ├── Semantic model: SalesAnalysis  (built and edited in the web model editor)
│   │   ├── Fact tables: Orders, OrderDetails
│   │   ├── Dimension tables: Customers, Products, Date
│   │   ├── Relationships (many-to-one, single direction)
│   │   │   ├── Customers[CustomerID] → Orders[CustomerID]
│   │   │   ├── Orders[OrderID]       → OrderDetails[OrderID]
│   │   │   ├── Products[ProductID]   → OrderDetails[ProductID]
│   │   │   └── Date[Date]            → Orders[OrderDate]
│   │   └── DAX measures
│   │
│   ├── Report: SalesAnalysis (built in the browser report editor, 4 pages)
│   │   ├── Page 1: Sales Overview
│   │   ├── Page 2: Customer Analysis
│   │   ├── Page 3: Product Performance
│   │   └── Page 4: Sales Analysis (time intelligence)
│   │
│   └── Dashboard: Sales Performance Dashboard (pinned tiles)
│
└── Sharing & Collaboration
    └── Share dashboard → email notification with link

Data Flow Summary:
1. Connect: OData service + CSV files → Power Query Online
2. Model:   Relationships, measures and Date table in the web model editor
3. Visualize: Build report pages in the browser
4. Save & refresh: Content lives in the workspace
5. Dashboard: Pin key visuals to a new dashboard
6. Share: Distribute the dashboard to team members
```

## Job Skills

- Task 1: Introduction to the Power BI service and initial setup (10 minutes)
- Task 2: Connect to and import data from multiple sources with Power Query Online (20 minutes)
- Task 3: Build a data model with relationships and measures in the web model editor (15 minutes)
- Task 4: Create interactive reports in the browser (25 minutes)
- Task 5: Save, explore and refresh your content in the service (10 minutes)
- Task 6: Create and share a dashboard (10 minutes)
- Task 7: Create a Date table and time intelligence measures (20 minutes)
- Task 8: Conditional formatting and report themes (20 minutes)
- Task 9: Bookmarks, buttons, and report navigation (20 minutes)

---

## Task 1: Introduction to the Power BI Service and Initial Setup

> **What is Power BI?** Power BI is a business analytics service by Microsoft that provides interactive visualizations and business intelligence capabilities. The **Power BI service** (app.powerbi.com) is the online SaaS portal. In this lab you use it for both authoring and sharing, so Power BI Desktop isn't needed.

### Sign In and Explore the Portal

1. Open a browser and navigate to [https://app.powerbi.com](https://app.powerbi.com).

2. Sign in with your Power BI account.

3. Familiarize yourself with the left navigation pane:

   | Item | Purpose |
   | --- | --- |
   | **Home** | Recent and favorite content |
   | **Create** | Start a new report or semantic model (this is where you'll begin) |
   | **Browse / OneLake catalog** | Find content shared with you |
   | **Workspaces** | Your workspaces, including **My workspace** |

   > Menu names can differ slightly between tenants and releases (for example, some tenants show a **+ New item** button inside a workspace). The steps below name both entry points where they differ.

### Verify Workspace Access

1. In the left navigation pane, select **Workspaces**.

2. Select **My workspace**. This is your personal workspace where you'll save everything in this lab.

   > **What is a workspace?** A workspace is a collaborative environment where you and your team create and share dashboards, reports, and semantic models. "My workspace" is personal; team workspaces allow multiple users to collaborate.

3. (Optional) If your organization requires it, select **+ New workspace**, name it **Power BI Lab**, and use that workspace instead of My workspace throughout the lab.

> **Tip:** The service autosaves model changes, but reports must be saved explicitly. Select **File** → **Save** (or **Ctrl + S**) in report editing view regularly.

---

## Task 2: Connect to and Import Data from Multiple Sources

The Power BI service includes **Power Query Online**, the same Power Query experience as Desktop, so you can connect to, clean, and combine data without installing anything. In this task you create a new **import** semantic model from an OData service and two CSV files.

> **Import mode vs. DirectQuery:** Import mode copies the data into the semantic model for fast performance. This lab uses Import mode.

### Connect to the Northwind OData Service for Orders

1. In Power BI service, select **Create** in the left navigation pane.

2. Select **Get data**. The Power Query **Get data** dialog opens.

3. Search for **OData** and select **OData** (OData feed).

4. In **Connection settings**, enter the following in the **URL** box:
   ```
   https://services.odata.org/v4/northwind/northwind.svc/
   ```

5. In **Connection credentials**, leave **Connection** as *Create new connection* and set **Authentication kind** to **Anonymous** (this is a public service). Select **Next**.

6. The **Choose data** (Navigator) page lists the Northwind tables (Customers, Employees, Orders, Order_Details, Products, …).

7. Select the checkboxes next to **Orders** and **Order_Details** (note the underscore), then select **Transform data**.

   > **Why Transform data?** It opens Power Query Online so you can clean and shape the data before it's loaded into the model.

### Clean and Prepare Orders Data

1. In the **Queries** pane on the left, select **Orders**.

2. On the **Home** tab, select **Choose columns**.

3. In **Choose columns**, clear **(Select all)** and select only:
   - OrderID
   - CustomerID
   - OrderDate
   - ShippedDate
   - Freight
   - ShipCity
   - ShipCountry

4. Select **OK**.

5. Select the **ShipCountry** column, then on the **Transform** tab select **Format** → **UPPERCASE**.

6. Select the **OrderDate** column, then on the **Transform** tab select **Data type** → **Date**. If prompted, select **Add new step** (don't replace the existing step).

7. Select the **ShippedDate** column and set **Data type** → **Date** the same way.

### Clean and Prepare Order Details Data

1. In the **Queries** pane, select **Order_Details**.

2. The service adds navigation columns named **Order** and **Product**. Remove them:
   - Right-click the **Order** column header → **Remove columns**
   - Right-click the **Product** column header → **Remove columns**

   > **Why remove them?** They're nested tables generated by the OData service. You already have the keys (OrderID and ProductID) needed for relationships, and keeping them adds complexity and hurts performance.

3. Confirm the query contains only: OrderID, ProductID, UnitPrice, Quantity, Discount.

### Add the Products CSV File

1. On the **Home** tab of Power Query Online, select **Get data** → **Text/CSV**.

2. In **Connection settings**, select **Upload file** and browse to the **products.csv** file on your computer. (Alternatively, if you've stored the file in OneDrive for Business or SharePoint, paste its link in **File path or URL**.)

3. Note that the file is uploaded in OneDrive. Select **Next**.

4. Verify the preview shows headers: productID, productName, quantityPerUnit, unitPrice, discontinued, categoryID, unitCost. Select **Create**.

5. Select the new **products** query and rename the columns (right-click header → **Rename**):
   - productID → ProductID
   - productName → ProductName
   - quantityPerUnit → QuantityPerUnit
   - unitPrice → UnitPrice
   - discontinued → Discontinued
   - categoryID → CategoryID
   - unitCost → UnitCost

### Add the Customers CSV File

1. Select **Get data** → **Text/CSV** again and upload **customers.csv** the same way.

2. In the preview, check the **File origin**. Some company names (for example, *Antonio Moreno Taquería*) contain accented characters. If they look garbled, change **File origin** to **1252: Western European (Windows)**.

3. Select **Create**.

4. Confirm the headers are customerID, companyName, contactName, contactTitle, city, country. If the first row appears as data, on the **Transform** tab select **Use first row as headers**.

5. Select the **contactName** column. On the **Transform** tab select **Format** → **Trim**, then **Format** → **Clean**.

6. Rename the columns:
   - customerID → CustomerID
   - companyName → CompanyName
   - contactName → ContactName
   - contactTitle → ContactTitle
   - city → City
   - country → Country

### Rename Queries for Consistency

In the **Queries** pane, right-click each query, select **Rename**, and set:

| Current name | New name |
| --- | --- |
| Orders | Orders (no change) |
| Order_Details | **OrderDetails** |
| products | **Products** |
| customers | **Customers** |

### Create the Semantic Model

1. Confirm the Queries pane shows four queries: **Orders**, **OrderDetails**, **Products**, **Customers**.

2. Select **Create a semantic model only**.

   > Choosing **Create a report** would take you straight to report editing. We'll first finish the model, so the semantic-model-only option is better here.

3. Enter **SalesAnalysis**. Then select **Create**. 

4. The semantic model editor opens, showing the four queries as tables.

> **Congratulations!** You've imported data from multiple sources without Power BI Desktop. Your data is ready to be modeled.

---

## Task 3: Build a Data Model with Relationships and Measures

You use the **web model editor** (the Power BI service's equivalent of Model view) to build a star schema, where fact tables (Orders, OrderDetails) are the center and dimension tables (Customers, Products) provide context.

> **What is a star schema?** A star schema consists of **fact tables** (measurements and metrics), **dimension tables** (descriptive attributes), and **one-to-many relationships** from dimensions to facts.

> **Important:** Relationships from the data source are **not** imported when you use Power Query in the service. You create all relationships manually.

### Switch to Editing Mode

1. The model may open in **Viewing mode**. Select **Viewing** at the top right and switch to **Editing**.

2. You see four tables: Customers, OrderDetails, Orders, Products.

3. Drag tables so they're easy to read:
   - **Customers** on the left
   - **Orders** in the center
   - **OrderDetails** to the right of Orders
   - **Products** on the far right

### Create Relationship: Customers to Orders

1. Drag **CustomerID** from **Customers** onto **CustomerID** in **Orders** (or select **Manage relationships** → **New relationship**).

2. Configure the relationship:

   | Setting | Value |
   | --- | --- |
   | From table / column | Orders (CustomerID) |
   | To table / column | Customers (CustomerID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

3. Select **Save** / **OK**.

   > **Understanding cardinality:** *Many to one (\*:1)* means many orders belong to one customer. The "one" side is the dimension table.

### Create Relationship: Orders to OrderDetails

1. Drag **OrderID** from **Orders** onto **OrderID** in **OrderDetails**.

2. Verify:

   | Setting | Value |
   | --- | --- |
   | From table / column | OrderDetails (OrderID) |
   | To table / column | Orders (OrderID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Active | ✓ Checked |

### Create Relationship: Products to OrderDetails

1. Drag **ProductID** from **Products** onto **ProductID** in **OrderDetails**.

2. Verify:

   | Setting | Value |
   | --- | --- |
   | From table / column | OrderDetails (ProductID) |
   | To table / column | Products (ProductID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Active | ✓ Checked |

### Verify Your Data Model

```
Customers (1) ──────> (*) Orders (1) ──────> (*) OrderDetails (*) <────── (1) Products
  [Dimension]            [Fact]                  [Fact]                      [Dimension]
```

1. On the ribbon, select **Manage relationships**.
2. Confirm all three relationships are listed and **Active**.
3. Close the dialog.

Filters flow from dimensions to facts: selecting a customer filters their orders and order details; selecting a product filters its order details. This is called **filter propagation**.

### Create the Core DAX Measures

In the desktop lab you create measures while building the report. In the browser, the model editor is the most reliable place to author measures, so create them now.

1. In the **Data** pane, select the **OrderDetails** table, then select **New measure** on the ribbon.

2. In the formula bar, enter and press **Enter**:
   ```
   Total Revenue = SUMX(OrderDetails, OrderDetails[UnitPrice] * OrderDetails[Quantity] * (1 - OrderDetails[Discount]))
   ```

3. Select the **Orders** table → **New measure**:
   ```
   Total Orders = COUNTROWS(Orders)
   ```

4. Select the **Orders** table → **New measure**:
   ```
   Average Order Value = DIVIDE([Total Revenue], [Total Orders], 0)
   ```

5. Select each measure and set its format on the **Properties** pane:

   | Measure | Format | Decimal places |
   | --- | --- | --- |
   | Total Revenue | Currency | 0 |
   | Total Orders | Whole number | 0 |
   | Average Order Value | Currency | 2 |

> Changes in the web model editor are saved automatically.

### Test the Model

1. On the ribbon, select **File → Create new report**.

2. Add a **Table** visual and add **Country** from **Customers**, then **Quantity** from **OrderDetails**.

3. The table shows total quantity by country. This confirms the filter propagates Customers → Orders → OrderDetails.

4. Leave this report open. You continue to build it in Task 4.

> **Congratulations!** You built a star schema model with relationships and measures entirely in the browser.

---

## Task 4: Create Interactive Reports in the Browser

You now use the **report editing view** in the Power BI service, which has the same Visualizations, Format and Data panes as Desktop. You build three pages.

> **Best practice:** Organize your report into logical pages that focus on specific themes or audiences.

### Save the Report

1. In the report editing view, select **Save** (or **File** → **Save**).
2. Enter the name **SalesAnalysis**, choose **My workspace**, and select **Save**.

   > Because the report is browser-based, there's no `.pbix`. Remember to save again after each major change.

3. Go to back to the report **Editing** view.

4. Delete the test table from Task 3: select it and press **Delete**.

### Page 1: Sales Overview

#### Create KPI Cards

1. In the **Visualizations** pane, select Restore default visuals. Then select the **Card** icon.

2. Drag the **Total Revenue** measure from the **Data** pane (**OrderDetails** table) onto the card.

3. Format the card (select the card → **Format your visual** pane, the paint roller icon → **Visual** tab):
   - Expand **Callout value** → set **Display units** to **Thousands** and **Value decimal places** to **0**
   - Turn **Category label** (label) **Off** (optional)

4. Resize the card and position it in the upper-left corner.

5. On the **General** tab, expand **Title**, turn **Title** **On**, and set **Text** to `Total Revenue`.

6. Add a second **Card** with **Total Orders**. Display units: **None**, decimals **0**, title `Total Orders`. Position it to the right of Total Revenue.

7. Add a third **Card** with **Average Order Value**. Display units **None**, decimals **2**, title `Average Order Value`. Position it to the right of Total Orders.

#### Create Sales Trend Line Chart

1. Below the cards, add a **Line chart** visual.

2. Configure the visual:
   - **X-axis**: **OrderDate** from **Orders**
   - **Y-axis**: **Total Revenue**

   > If **OrderDate** shows as a hierarchy (Year, Quarter, Month, Day), select the dropdown on the field and choose **OrderDate** (not *Date Hierarchy*). In Task 7 you replace this with a proper Date table.

3. Format the chart (**Format your visual**):
   - **X-axis** → turn **Title** **On**, set **Title text** to `Order Date`
   - **Y-axis** → turn **Title** **On**, set **Title text** to `Revenue`

4. Resize the chart to about 40% of the page width.

#### Create Orders by Country Bar Chart

1. Add a **Clustered bar chart** visual.

2. Configure:
   - **Y-axis**: **ShipCountry** from **Orders**
   - **X-axis**: **Total Orders**

3. Sort the chart:
   - Select the visual → **More options (...)** → **Sort by** → **Total Orders**
   - **More options (...)** → **Sort by** → **Sort descending**

4. On the **General** tab → **Title**, turn it **On** and set **Text** to `Orders by Country`.

5. Position the chart to the right of the line chart.

#### Add a Date Slicer

1. Add a **Slicer** visual.

2. Drag **OrderDate** from **Orders** into the **Field** well.

3. Select the slicer → **Format your visual** → **Visual** → **Slicer settings** → **Options** and set **Style** to **Between**.

4. Position the slicer at the bottom of the page, spanning the full width.

5. On the **General** tab:
   - Turn **Title** **On** and set **Text** to `Select Date Range`
   - Expand **Effects** and turn **Background** **On**

#### Rename Page 1

1. Double-click the **Page 1** tab at the bottom (or right-click → **Rename page**).

2. Type `Sales Overview` and press **Enter**.

### Page 2: Customer Analysis

1. Select the **+** icon at the bottom to add a new page.

#### Create Top Customers Table

1. Add a **Table** visual.

2. Add these fields: **CompanyName**, **Country**, **City** (from **Customers**), then the **Total Revenue** and **Total Orders** measures.

3. Sort the table: **More options (...)** → **Sort by** → **Total Revenue** (or select the **Total Revenue** column header until the sort arrow points down).

4. Resize the table to the left half of the page.

5. **General** → **Title** → **On**, **Text**: `Top Customers by Revenue`.

#### Create Customer Treemap

1. Add a **Treemap** visual.

2. Configure:
   - **Category**: **Country** from **Customers**
   - **Values**: **Total Revenue**

3. Position it on the right side of the page.

4. **General** → **Title** → **On**, **Text**: `Revenue by Customer Location`.

#### Add Country Slicer

1. Add a **Slicer** visual and drag **Country** from **Customers** into **Field**.

2. **Format your visual** → **Visual** → **Slicer settings** → **Selection** → turn on **Show "Select all" option**.

3. Position the slicer at the bottom of the page.

4. **General** → **Title** → **On**, **Text**: `Filter by Country`. Under **Effects**, turn **Background** **On**.

#### Rename Page 2

Rename the page to `Customer Analysis`.

### Page 3: Product Performance

1. Add a third page with the **+** icon.

#### Create Product Sales Table

1. Add a **Table** visual with:
   - **ProductName** and **CategoryID** from **Products**
   - **Total Revenue** measure
   - **Quantity** from **OrderDetails** (summed automatically)

2. Sort by **Total Revenue** descending.

3. Resize it to the left half of the page.

4. **General** → **Title** → **On**, **Text**: `Product Sales Performance`.

#### Create Top 10 Products Column Chart

1. Add a **Clustered column chart**.

2. Configure:
   - **X-axis**: **ProductName** from **Products**
   - **Y-axis**: **Total Revenue**

3. Show only the top products:
   - Select the visual and open the **Filters** pane on the right (if it's hidden, select **View** → **Filters** or the **Filters** tab)
   - Expand the **ProductName** filter
   - Set **Filter type** to **Top N**
   - **Show items**: **Top 10**
   - **By value**: **Total Revenue** (drag the measure into the **By value** box if it's empty)
   - Select **Apply filter**

4. Format: **Visual** → **X-axis** → **Values** → set **Text size** to `8`.

5. Position it on the right side of the page.

6. **General** → **Title** → **On**, **Text**: `Top 10 Products by Revenue`.

#### Add Category Slicer

1. Add a **Slicer** with **CategoryID** from **Products**.

2. **Visual** → **Slicer settings** → **Options** → **Style**: **Vertical list**.

3. Position at the bottom of the page.

4. **General** → **Title** → **On**, **Text**: `Filter by Category`. Under **Effects**, turn **Background** **On**.

#### Rename Page 3

Rename the page to `Product Performance`.

### Test Interactivity

1. Switch to **Viewing** option at the top.

2. On **Sales Overview**, select a bar (for example USA) in **Orders by Country**. The other visuals filter to that country (**cross-filtering**). Select the bar again to clear it.

3. On **Customer Analysis**, pick a country in the slicer. The table and treemap update.

4. On **Product Performance**, pick a category. The table and column chart update.

> **Cross-filtering:** Selecting a visual element filters related visuals on the page. To change this, in editing view select a visual and use **Format** → **Edit interactions**.

> **Congratulations!** You created a three-page interactive report with KPIs, trends, tables, charts, a treemap, and slicers, all in the browser.

---

## Task 5: Save, Explore, and Refresh Your Content in the Service

In the Desktop lab you publish a `.pbix`. In the online lab your semantic model and report already live in the workspace, so this task focuses on exploring and maintaining them.

### Explore Your Report and Semantic Model

1. Open **My workspace** from the left navigation pane.

2. You should see:
   - **SalesAnalysis** (type **Semantic model**) – tables, relationships, measures
   - **SalesAnalysis** (type **Report**) – pages and visuals

   > **Report vs. semantic model:** The semantic model holds the data model; the report holds the visuals and pages. Many reports can reuse one semantic model.

3. Selecting the semantic model lets you view its tables, relationships, and measures. Selecting the report lets you view its pages and visuals.

### View Lineage

1. In **My workspace**, switch from **List** to **Lineage** view (top right).

2. You see the OData service and CSV sources flowing into the **SalesAnalysis** semantic model and then into the **SalesAnalysis** report. Lineage view helps you understand impact when something changes.

3. Head back to **List** view when done exploring lineage.

### Understanding Refresh

1. In the workspace, hover over the **SalesAnalysis** semantic model and select **Refresh now** (the circular arrow icon).

2. Open **Settings** (**More options (...)** → **Data access**) → **Data source credentials**. Verify if you are connected to the data sources. If you receive an error, follow the steps below to resolve it.
   - Select **Gateway and cloud connections**
   - Select the configuration for Products data source
   - Select Create a connection
   - Enter Connection Name: Products
   - Authentication method: OAuth 2.0
   - Select Edit credentials, login with your account
   - Select Create
   - Repeat the steps for the Customers data source and name the connection: Customers
   - Map the appropriate connection to each data source (Products and Customers)

3. Head back to the Data source credentials page and verify that there are no more errors.

4. Select **Refresh** to see the scheduling options.

   > **Note:** In this lab the data is static (CSV files and a demo OData service), so refresh won't change anything. In production you'd schedule a refresh to keep reports current.

5. Skip scheduling for now.

> **Congratulations!** Your report and semantic model are in the Power BI service. Next you create a dashboard.

---

## Task 6: Create and Share a Dashboard in the Power BI Service

Dashboards are single-page canvases of tiles pinned from one or more reports. They can only be created in the Power BI service.

> **Report vs. dashboard:**
> - **Report**: multi-page, interactive, built from a semantic model
> - **Dashboard**: single page of pinned tiles that link back to their source reports
> - **Use case**: Executives who want KPIs at a glance

### Create a New Dashboard

1. Open the **SalesAnalysis** report in **Viewing** and go to **Sales Overview**.

2. Hover over the **Total Revenue** card and select the **Pin visual** icon (pushpin).

3. In **Pin to dashboard**, select **New dashboard**.

4. Enter the name `Sales Performance Dashboard` and select **Pin**.

5. A confirmation says **Pinned to dashboard**.

### Pin Additional Visuals

1. On **Sales Overview**, pin each of the following to **Existing dashboard** → **Sales Performance Dashboard**:
   - **Total Orders** card
   - **Average Order Value** card
   - **Orders by Country** bar chart

2. On **Customer Analysis**, pin the **Top Customers by Revenue** table and the **Revenue by Customer Location** treemap.

3. On **Product Performance**, pin the **Top 10 Products by Revenue** chart.

### View and Arrange Your Dashboard

1. Open **My workspace** and select **Sales Performance Dashboard**.

2. Rearrange tiles by dragging and resizing. Suggested layout:
   - Row 1: Total Revenue, Total Orders, Average Order Value
   - Row 2: Orders by Country (left), Revenue by Customer Location (right)
   - Row 3: Top Customers by Revenue (left), Top 10 Products by Revenue (right)

### Interact with Dashboard Tiles

1. Select a tile (for example **Total Revenue**). Power BI opens the source report page.
2. Use the browser's back button to return to the dashboard.

### Share Your Dashboard

1. On the dashboard, select **Share**.

2. Enter the email address of a colleague.

   > **Important:** Recipients typically need a Power BI Pro (or equivalent) license and must be in your organization, unless the content is in a Fabric/Premium capacity workspace.

3. Review the options:
   - **Allow recipients to share this dashboard**
   - **Allow recipients to build content with the data associated with this dashboard**
   - **Send an email notification**

4. Select **Grant access**.

### Set Up Dashboard Alerts (Optional)

1. Hover over the **Total Revenue** tile → **More options (...)** → **Manage alerts**.
2. Select **+ Add alert rule**, set **Condition**: *Above* a threshold (for example 1,000,000) and **Maximum notification frequency**: *At most once every 24 hours*, and optionally **Send me email too**.
3. Select **Save and close**.

> **Note:** Alerts work only with cards, gauges, and KPIs and on data that changes, so they won't fire with this static data.

### Subscribe to the Dashboard (Optional)

1. On the dashboard, select **Subscribe**.
2. Select **Create a subscription** or **Add new subscription** and set a name (for example `Daily Sales Snapshot`), a repeat frequency, a time, a time zone, and an email subject and message.
3. Select **Save**.

> **Congratulations!** You created, arranged, and shared a dashboard without ever leaving the browser.

---

## Task 7: Create a Date Table and Time Intelligence Measures

You create a Date table and time intelligence measures in the web model editor.

> **Why a dedicated Date table?** Auto date/time works for basic charts, but accurate time intelligence (YTD, year-over-year) needs a dedicated Date table marked as a date table.
>
> *Reference: [Set and use date tables in Power BI](https://learn.microsoft.com/power-bi/transform-model/desktop-date-tables) and [Edit semantic models in the Power BI service](https://learn.microsoft.com/power-bi/transform-model/service-edit-data-models)*

### Open the Semantic Model

1. Go to **My workspace**, select the semantic model.

2. Switch to **Editing** mode if needed.

### Create the Date Table Using DAX

1. On the ribbon, select **New table**.

2. Enter this DAX and press **Enter**:
   ```
   Date =
   ADDCOLUMNS(
       CALENDAR(DATE(1996,1,1), DATE(1998,12,31)),
       "Year", YEAR([Date]),
       "Month Number", MONTH([Date]),
       "Month Name", FORMAT([Date], "MMMM"),
       "Quarter", "Q" & QUARTER([Date]),
       "Year-Month", FORMAT([Date], "YYYY-MM")
   )
   ```

   > `CALENDAR` generates one row per day across the Northwind data range, and `ADDCOLUMNS` adds Year, Month, Quarter, and a sortable Year-Month text value.

### Mark the Table as a Date Table

1. In the **Data** pane, right-click the **Date** table (or select **More options (...)**).

2. Select **Mark as date table**.

3. Turn on **Mark as a date table**.

4. In **Choose a date column**, select **Date**, and select **Save**.

### Create the Relationship to Orders

1. In the model diagram, drag **Date** from the **Date** table onto **OrderDate** in **Orders**.

2. Verify:

   | Setting | Value |
   | --- | --- |
   | Cardinality | One to many (1:\*) |
   | Cross filter direction | Single |
   | Active | ✓ Yes |

3. Select **Save**.

### Create Time Intelligence Measures

1. Select the **OrderDetails** table → **New measure** and add each measure below (one at a time):

   ```
   Revenue YTD = TOTALYTD([Total Revenue], 'Date'[Date])
   ```

   ```
   Revenue Last Year = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date]))
   ```

   ```
   YoY Growth % =
   DIVIDE(
       [Total Revenue] - [Revenue Last Year],
       [Revenue Last Year],
       0
   )
   ```

2. Select **YoY Growth %**, then on **Measure tools** set **Format** to **Percentage** with **2** decimal places.

   > *Reference: [Time intelligence functions (DAX)](https://learn.microsoft.com/dax/time-intelligence-functions-dax)*


### Add a Sales Analysis Page

1. Return to the report. In the report **Editing** view, add a new page with **+**.

2. Rename it to `Sales Analysis`.

3. Add a **Slicer** with **Year** from the **Date** table:
   - **Visual** → **Slicer settings** → **Options** → **Style**: **Vertical list**
   - **Slicer settings** → **Selection** → **Single select**: **On**
   - **General** → **Title** **On**, **Text**: `Select Year`
   - **Effects** → **Background** **On**

4. Add three **Card** visuals across the top:
   - **Revenue YTD**
   - **Revenue Last Year**
   - **YoY Growth %**

5. Add a **Table** visual below the cards with:
   - **CompanyName** from **Customers**
   - **Revenue YTD**
   - **Revenue Last Year**
   - **YoY Growth %**

   Sort by **Revenue YTD** descending.

6. Add a **Line and clustered column chart** to the right of the table:

   | Well | Field |
   | --- | --- |
   | X-axis | **Year-Month** from **Date** |
   | Column y-axis | **Total Revenue** |
   | Line y-axis | **Revenue Last Year** |

   Sort by **Year-Month** ascending.

7. **General** → **Title**: `Revenue: Current vs Prior Year`. Turn **Subtitle** **On** and set **Text** to `Comparison of current year revenue versus prior year`.

8. For the **YoY Growth %** card, **Visual** → **Callout value** → **Display units**: **Auto**, **Value decimal places**: `2`. Confirm the percentage format is preserved.

9. Arrange the page: year slicer on the left, customer table in the center, comparison chart on the right (see the screenshots in [Power-BI-Intermediate-Ref/Report Screenshots](<./Power-BI-Intermediate-Ref/Report Screenshots>)).

10. **Save** the report.

> **Congratulations!** Your report now has time intelligence KPIs backed by a proper Date table.

---

## Task 8: Conditional Formatting and Report Themes

### Apply a Report Theme

1. In the report **Editing** view, select the **Design** tab on the ribbon.

2. Select **Theme** and browse the built-in themes.

3. Select a theme. All visuals update immediately.

   > **Why themes matter:** Themes enforce consistent colors, fonts, and visual formatting across every page.
   >
   > *Reference: [Use report themes in Power BI](https://learn.microsoft.com/power-bi/create-reports/desktop-report-themes)*

4. Select the **Theme gallery** and download a theme `.json` file.

5. In the service, select **Import theme** and choose the downloaded `.json` file.

### Add Conditional Formatting — Data Bars

1. Go to the **Sales Analysis** page and select the customer table.

2. In **Format your visual** → **Visual** → **Cell elements**.

3. Set **Series** to **Revenue YTD** and turn **Data bars** **On**.

   > Data bars render an inline bar in each cell for an immediate sense of relative values.

### Add Conditional Formatting — Rules

1. Still in **Cell elements**, set **Series** to **YoY Growth %**.

2. Turn **Background color** **On**, then select **fx**.

3. Set **Format style** to **Rules** and add:
   - **Rule 1:** If value >= -100 Number and < 0 Number then colour `#FF0000`
   - **Rule 2:** If value >= 0 Number and < 0.25 Number then colour `#FFFF00`
   - **Rule 3:** If value >= 0.25 Number and < 1000 Number then colour `#00FF00`

4. Select **OK**.

### Add Conditional Formatting — Icons

1. With **Series** still set to **YoY Growth %**, turn **Icons** **On**, then select **fx**.

2. Set **Format style** to **Rules**, **Icon layout** to **Right of data**, and add:
   - **Rule 1:** If value >= -100 Number and < 0 Number then Icon: Gray downward-slanting arrow
   - **Rule 2:** If value >= 0 Number and < 0.25 Number then Icon: Gray side arrow
   - **Rule 3:** If value >= 0.25 Number and < 1000 Number then Icon: Green upward-slanting arrow

3. Select **OK**.

   > Icons give users an at-a-glance status indicator and reduce cognitive load.

4. Press **Ctrl + S**.

> **Congratulations!** Your report now has a consistent theme and tables enriched with conditional formatting.

---

## Task 9: Bookmarks, Buttons, and Report Navigation

### Add a Page Navigator

1. In **Editing** view, go to **Sales Overview**.

2. On the ribbon, select **Insert** → **Buttons** → **Navigator** → **Page navigator**.

   > The page navigator generates one button per report page and stays in sync when pages are added or renamed.
   >
   > *Reference: [Create and configure buttons in Power BI reports](https://learn.microsoft.com/power-bi/create-reports/desktop-buttons)*

3. Drag it to the top of the page and resize it to full width. Move other visuals down as needed.

4. In **Format** → **Visual** → **Shape**, set **Shape** to **Rounded rectangle (%)** and **Corner radius** to `25%`.

5. Copy the navigator (**Ctrl + C**), go to **Customer Analysis**, and paste (**Ctrl + V**). Position it identically.

6. Repeat for **Product Performance** and **Sales Analysis**.

### Open the Bookmarks and Selection Panes

1. Go to **Customer Analysis**.

2. On the **View** tab, enable **Filters**, **Bookmarks** and **Selection**.

### Create an "All Customers" Bookmark

1. Make sure no filters are active.

2. In the **Bookmarks** pane, select **Add**.

3. Rename the bookmark to `All Customers` (**More options (...)** → **Rename**).

4. **More options (...)** → **Update** and make sure **Data** is checked and **Display** is unchecked.

   > Capturing **Data** only saves the filter state, not the scroll position or visual selection.

### Create a "Top 15 Customers" Bookmark

1. Select the **Top Customers by Revenue** table.

2. In **Filters**, expand **CompanyName**.

3. Set **Filter type** to **Top N**, **Show items** to **Top 15**, **By value** to **Total Revenue**, and select **Apply filter**.

4. In the **Bookmarks** pane, select **Add** and rename it `Top 15 Customers`.

5. **More options (...)** → **Update**, with **Data** checked and **Display** unchecked.

   > *Reference: [Bookmarks capture the state of a report page](https://learn.microsoft.com/power-bi/create-reports/desktop-bookmarks#bookmarks-capture-the-state-of-a-report-page)*

### Wire Buttons to Bookmarks

1. On the ribbon, select **Insert** → **Buttons** → **Blank**.

2. In **Format button**:
   - **Button** tab → **Style** → turn **Text** **On** and set **Text** to `All Customers`
   - Turn **Action** **On**, **Type**: **Bookmark**, **Bookmark**: **All Customers**

3. Add a second blank button with **Text** `Top 15 Customers` and **Action** → **Bookmark** → **Top 15 Customers**.

4. Place both buttons side by side below the **Top Customers by Revenue** table.

5. Test: in **Editing** view, hold **Ctrl** and click each button.

6. Press **Ctrl + S**, then select **Reading** view. In **Reading** view, click the buttons without holding **Ctrl** and confirm the views toggle. Also test the page navigator.

> **Congratulations!** Your report has page navigation, conditional formatting, and bookmark-driven views — all built in the browser.

---

## Lab Summary

### What You Accomplished

✅ **Task 1**: Signed in to the Power BI service and verified your workspace

✅ **Task 2**: Connected to OData and CSV sources with Power Query Online and created an import semantic model

✅ **Task 3**: Built a star schema with relationships and DAX measures in the web model editor

✅ **Task 4**: Built a three-page interactive report in the browser:
   - Sales Overview: KPIs, trends, and geographic analysis
   - Customer Analysis: Top customers and location-based insights
   - Product Performance: Product sales and category filtering

✅ **Task 5**: Explored the workspace, lineage, and refresh options

✅ **Task 6**: Created and shared a dashboard highlighting key metrics

✅ **Task 7**: Created a Date table, marked it as a date table, and built time intelligence measures

✅ **Task 8**: Applied a theme and used conditional formatting (data bars, color rules, icons)

✅ **Task 9**: Added a page navigator, bookmarks, and buttons

### Key Concepts You Learned

- **Authoring entirely in the Power BI service**: no Desktop installation needed
- **Power Query Online**: connecting, cleaning, and combining OData and CSV data
- **Web model editor**: relationships, DAX measures, calculated tables, and date tables
- **Star schema design**: fact and dimension tables with one-to-many relationships
- **Semantic model vs. report vs. dashboard**
- **Cross-filtering, slicers, and interactive visuals**
- **Time intelligence**: YTD and year-over-year using DAX
- **Themes and conditional formatting**
- **Bookmarks, buttons, and page navigation**
- **Sharing**: Collaborating with colleagues (license requirements apply)

**Thank you for completing this lab!** You can now build and share complete Power BI solutions using nothing but a browser.
