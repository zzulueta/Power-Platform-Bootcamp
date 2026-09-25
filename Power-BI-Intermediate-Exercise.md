---
  title: 'Lab Power BI Intermediate: Reports and Dashboards'
  module: 'Power BI Intermediate Lab'
---

# Lab Power BI Intermediate – Reports and Dashboards

## Lab Introduction

In this lab, you will learn the fundamentals of Power BI by building a complete end-to-end analytics solution. You will work with Power BI Desktop to connect to data, create a data model, build interactive reports, and then publish your work to the Power BI Service where you'll create and share a dashboard.

This lab will guide you through the following tasks:

- Understand the basics of Power BI Desktop and the Power BI Service
- Connect to and import data from multiple sources (OData and CSV files)
- Build a data model using star schema design with relationships
- Create interactive reports with various visualizations, filters, and slicers
- Publish your report to the Power BI Service
- Create a dashboard by pinning visuals from your report
- Share your dashboard with colleagues

## Pre-requisites

- **Power BI Desktop installed**: Download and install the free Power BI Desktop application from [Microsoft's website](https://powerbi.microsoft.com/desktop)
- **Power BI Pro license (Optional)**: Both you (the creator) and anyone you share content with must have a Power BI Pro license
- **Power BI Service account**: Sign up for a free account at [https://app.powerbi.com](https://app.powerbi.com)
- **Workspace access**: Access to a workspace in the Power BI Service (you'll create or use "My Workspace" in this lab)
- **Sample data access**: Download the CSV files (products.csv and customers.csv) from the Power-BI-Intermediate-Ref folder
- **Internet connection**: Required for accessing the Northwind OData service

## Estimated Timing: 150 minutes

## Lab Scenario

Your organization wants to analyze sales data to gain insights into product performance, customer behavior, and overall sales trends. The data currently exists in multiple locations:

- **Order transactions** and **order line items** are available through a Northwind OData service (a public demo service)
- **Product catalog** data is maintained in a CSV file
- **Customer information** is stored in a separate CSV file

Your task is to:

- Connect to these disparate data sources and bring the data into Power BI Desktop
- Create a unified data model that relates orders to customers and products
- Build interactive reports that help answer key business questions:
  - What are our total sales and how many orders have we processed?
  - Which customers are generating the most revenue?
  - Which products are our top sellers?
  - How are sales distributed geographically?
- Publish the report to the Power BI Service
- Create a dashboard that highlights the most important KPIs
- Share the dashboard with your team for collaborative decision-making

## Architecture Overview

```
Power BI Lab Architecture
========================

Data Sources
├── Northwind OData Service (https://services.odata.org/v4/northwind/northwind.svc/)
│   ├── Orders table (order header data)
│   └── Order_Details table (order line item data)
│
└── CSV Files (Power-BI-Intermediate-Ref folder)
    ├── products.csv (product catalog)
    └── customers.csv (customer information)

                    ↓ (Get Data / Import Mode)

Power BI Desktop
├── Data Model (Star Schema)
│   ├── Fact Tables
│   │   ├── Orders (center of star - many relationships)
│   │   └── Order_Details (transactional grain)
│   │
│   ├── Dimension Tables
│   │   ├── Customers (one-to-many → Orders)
│   │   ├── Products (one-to-many → Order_Details)
│   │   ├── Products (one-to-many → Orders via Order_Details)
│   │   └── Date (one-to-many → Orders via OrderDate)
│   │
│   └── Relationships (one-to-many, single direction filter)
│       ├── Customers[CustomerID] → Orders[CustomerID]
│       ├── Products[ProductID] → Order_Details[ProductID]
│       ├── Orders[OrderID] → Order_Details[OrderID]
│       └── Date[Date] → Orders[OrderDate]
│
├── Report Canvas (3 pages)
│   ├── Page 1: Sales Overview
│   │   ├── KPI Cards (Total Sales, Order Count, Avg Order Value)
│   │   ├── Sales Trend Line Chart
│   │   └── Orders by Country Bar Chart
│   │
│   ├── Page 2: Customer Analysis
│   │   ├── Top Customers Table
│   │   ├── Customer Map (by Country/City)
│   │   └── Orders by Customer Bar Chart
│   │
│   └── Page 3: Product Performance
│       ├── Product Sales Table
│       ├── Top 10 Products Column Chart
│       └── Category Slicer
│
└── Visualizations & Interactivity
    ├── Filters (report, page, visual level)
    ├── Slicers (date, category, country)
    └── Cross-filtering between visuals

                    ↓ (File → Publish to Power BI)

Power BI Service (https://app.powerbi.com)
├── Workspace
│   └── Published Report (interactive, shareable)
│
├── Dashboard
│   ├── Pinned Visuals from Report
│   │   ├── Total Sales Card
│   │   ├── Orders by Country Chart
│   │   ├── Top Customers Table
│   │   └── Top 10 Products by Revenue Column Chart
│   │
│   └── Real-time Interaction
│       └── Click tiles to navigate to source report
│
└── Sharing & Collaboration
    └── Share Dashboard with Pro users → Email notification with link

Data Flow Summary:
1. Connect: OData Service + CSV Files → Power BI Desktop
2. Model: Build star schema with relationships
3. Visualize: Create 3 report pages with interactive visuals
4. Publish: Upload report to Power BI Service workspace
5. Dashboard: Pin key visuals to a new dashboard
6. Share: Distribute dashboard to team members (requires Pro licenses)
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following Power BI skills:

- Task 1: Introduction to Power BI Desktop and initial setup (10 minutes)
- Task 2: Connect to and import data from multiple sources (20 minutes)
- Task 3: Build a data model with relationships using star schema design (15 minutes)
- Task 4: Create interactive reports with visualizations, filters, and slicers (25 minutes)
- Task 5: Publish your report to the Power BI Service (10 minutes)
- Task 6: Create and share a dashboard in the Power BI Service (10 minutes)
- Task 7: Create a Date Table and Time Intelligence Measures (20 minutes)
- Task 8: Conditional Formatting and Report Themes (20 minutes)
- Task 9: Bookmarks, Buttons, and Report Navigation (20 minutes)

---

## Task 1: Introduction to Power BI Desktop and Initial Setup

Power BI Desktop is a free Windows application that allows you to connect to data, transform it, and create rich interactive reports. The Power BI Service (the online component) is where you publish, share, and collaborate on your reports and dashboards.

> **What is Power BI?** Power BI is a business analytics service by Microsoft that provides interactive visualizations and business intelligence capabilities. It consists of:
> - **Power BI Desktop**: A free Windows application for creating reports (this is where you'll do most of your work)
> - **Power BI Service**: An online SaaS service for publishing, sharing, and collaborating on reports
> - **Power BI Mobile**: Mobile apps for iOS, Android, and Windows devices

### Launch Power BI Desktop

1. Open **Power BI Desktop** from your Start menu or desktop shortcut. Select **Blank report**

2. If you see a welcome screen or tutorial, you can close it or follow along briefly to get familiar with the interface.

3. You should now see a blank canvas with the following main components:

   | Component | Location | Purpose |
   | --- | --- | --- |
   | **Ribbon** | Top | Contains tabs (Home, Insert, Modeling, View) with tools and commands |
   | **Report canvas** | Center | The main workspace where you build and arrange visualizations |
   | **Visualizations pane** | Right side | Shows visualization types and formatting options |
   | **Fields pane** | Far right | Lists all tables and fields from your data model |
   | **Filters pane** | Right side (may be collapsed) | Apply filters at report, page, or visual level |
   | **Pages tabs** | Bottom | Navigate between report pages |

### Sign In to Power BI Service

Before you can publish reports, you need to sign in to the Power BI Service at least once.

1. In Power BI Desktop, select **File** → **Sign in**.

2. Enter your organizational account credentials (the account with your Power BI Pro license).

3. Complete the sign-in process. You should see your account name in the upper-right corner of Power BI Desktop.

> **Important:** Both the creator (you) and anyone you share content with must have a Power BI Pro license. The Power BI Service will prompt recipients to upgrade if they don't have the appropriate license.

### Verify Workspace Access

1. Open a web browser and navigate to [https://app.powerbi.com](https://app.powerbi.com).

2. Sign in with the same credentials you used in Power BI Desktop.

3. On the left navigation pane, you should see **Workspaces**.

4. Expand **Workspaces** and verify you can see **My workspace**. This is your personal workspace where you can publish content.

   > **What is a workspace?** A workspace is a collaborative environment where you and your team can create and share dashboards, reports, and datasets. "My workspace" is your personal workspace, while team workspaces allow multiple users to collaborate.

5. You can close the browser for now. We'll return to the Power BI Service in Task 5.

### Save Your Power BI Desktop File

1. In Power BI Desktop, select **File** → **Save As**.

2. Select **Browse this device** and choose a location on your computer and name the file **SalesAnalysis.pbix**.

3. Select **Save**.

> **Tip:** Save your work frequently as you progress through this lab using **Ctrl + S** or **File** → **Save**.

---

## Task 2: Connect to and Import Data from Multiple Sources

Power BI Desktop can connect to a wide variety of data sources including databases, cloud services, files, and web services. In this task, you'll connect to both an OData service and CSV files to bring sales data into Power BI.

> **Import Mode vs. DirectQuery:** When connecting to data, Power BI offers different connection modes:
> - **Import Mode** (default): Data is copied into Power BI and stored in the .pbix file. Best for smaller datasets and faster performance.
> - **DirectQuery**: Power BI queries the data source directly each time a visual is refreshed. Best for very large datasets or real-time data.
> 
> For this lab, we'll use **Import Mode** which is ideal for learning and for datasets of this size.

### Connect to the Northwind OData Service for Orders

The Northwind OData service is a publicly available demo service that provides sample order data.

1. In Power BI Desktop, on the **Home** ribbon, select **Get data**.

2. In the **Get Data** dialog, search for **OData** or scroll to find **OData Feed** in the list.

3. Select **OData Feed** and then select **Connect**.

4. In the **OData Feed** dialog, enter the following URL in the **URL** box:
   ```
   https://services.odata.org/v4/northwind/northwind.svc/
   ```

5. Select **OK**.

6. If prompted for authentication, select **Anonymous** and then **Connect** (this is a public service that doesn't require credentials).

7. The **Navigator** window appears showing all available tables from the Northwind service. You should see tables like Customers, Employees, Orders, Order_Details, Products, and more.

8. In the Navigator window, select the checkbox next to **Orders** to see a preview of the data.

   The Orders table contains order header information including:
   - OrderID (unique identifier for each order)
   - CustomerID (links to customer)
   - OrderDate (when the order was placed)
   - ShippedDate (when the order was shipped)
   - Freight (shipping cost)
   - ShipCountry, ShipCity (delivery location)

9. Also select the checkbox next to **Order_Details** (note the underscore).

   The Order_Details table contains order line items with:
   - OrderID (links back to Orders table)
   - ProductID (links to product)
   - UnitPrice (price per unit)
   - Quantity (number of units ordered)
   - Discount (discount percentage applied)

10. With both tables selected, select **Transform Data** (not Load).

    > **Why Transform Data?** The Transform Data button opens Power Query Editor where you can clean, shape, and prepare your data before loading it into the data model. This is a best practice for ensuring data quality.

### Clean and Prepare Orders Data in Power Query Editor

The Power Query Editor opens with your two queries listed on the left.

1. In the **Queries** pane on the left, select the **Orders** query.

2. Review the columns in the preview pane. Notice that some columns like RequiredDate and ShippedDate are visible.

3. Let's remove some unnecessary columns to simplify our model. Select the **Home** tab, then select **Choose Columns**.

4. In the **Choose Columns** dialog, **uncheck** the Select All Columns option, then check only the following columns to keep:
   - OrderID
   - CustomerID
   - OrderDate
   - ShippedDate
   - Freight
   - ShipCity
   - ShipCountry

5. Select **OK** to apply the column selection.

6. Select the ShipCountry column, then go to the **Transform** tab and select **Format** → **UPPERCASE** to convert all text to uppercase.

7. Select the ShippedDate column, then go to the **Transform** tab and select **Data Type** → **Date** to ensure the column is recognized as a date.

8. Select the OrderDate column, then go to the **Transform** tab and select **Data Type** → **Date** to ensure the column is recognized as a date.

### Clean and Prepare Order Details Data

1. In the **Queries** pane, select the **Order_Details** query.

2. Review the columns in the preview pane. You may notice columns named **Order** and **Product**. These are navigation properties from the OData service that contain nested table data.

   > **Why remove these columns?** The OData service automatically creates navigation columns when tables have relationships. However, you already have the foreign keys (OrderID and ProductID) needed to build proper relationships in Power BI's data model. Keeping these nested columns would make your data model unnecessarily complex and could cause performance issues.

3. Remove the **Order** column:
   - Right-click the **Order** column header
   - Select **Remove**

4. Remove the **Product** column:
   - Right-click the **Product** column header
   - Select **Remove**

5. Your **Order_Details** query should now contain only these five essential fields:
   - OrderID
   - ProductID
   - UnitPrice
   - Quantity
   - Discount

### Connect to CSV File for Products Data

Now let's add the product catalog data from a CSV file.

1. Download the **products.csv** file.

2. Head back to the Power BI Desktop. In Power Query Editor, select **Home** → **New Source** → **More**.

3. In the **Get Data** dialog, search for **CSV** or select **Text/CSV** from the list.

4. Select **Text/CSV** and then select **Connect**.

5. Select the **products.csv** file and then select **Open**.

6. Power BI automatically detects the file structure. You should see a preview showing:
   - productID
   - productName
   - quantityPerUnit
   - unitPrice
   - discontinued
   - categoryID
   - unitCost

7. Verify that the data looks correct (headers are properly detected, data types look appropriate).

8. Select **OK** to add this table to your Power Query Editor.

9. The **products** query now appears in your Queries pane on the left.

10. Let's rename the column names in the **products** query for consistency:
    - productID → ProductID
    - productName → ProductName
    - quantityPerUnit → QuantityPerUnit
    - unitPrice → UnitPrice
    - discontinued → Discontinued
    - categoryID → CategoryID
    - unitCost → UnitCost

    To do this, right-click each column header, select **Rename**, and type the new name as listed above.

### Connect to CSV File for Customers Data

1. Download the **customers.csv** file.

2. Head back to Power BI Desktop. In Power Query Editor, select **Home** → **New Source** → **Text/CSV**.

3. Select **customers.csv** and select **Open**.

4. Review the preview showing customer information:
   - customerID
   - companyName
   - contactName
   - contactTitle
   - city
   - country

5. Select **OK** to add this to Power Query Editor.

6. The **customers** query now appears in your Queries pane.

7. Verify if the headers of the table are correct. If not, select the **Use First Row as Headers** button in the ribbon found in the **Home** tab.

8. Let's clean up the customer names. Select the **contactName** column.

9. On the **Transform** tab, select **Format** → **Trim** to remove any leading or trailing spaces.

10. Also select **Format** → **Clean** to remove any non-printable characters.

11. Let's rename the column names in the **customers** query for consistency:
    - customerID → CustomerID
    - companyName → CompanyName
    - contactName → ContactName
    - contactTitle → ContactTitle
    - city → City
    - country → Country

    To do this, right-click each column header, select **Rename**, and type the new name as listed above.

### Rename Queries for Consistency

Let's rename our queries to follow a consistent naming convention (PascalCase without underscores).

1. In the **Queries** pane, right-click the **Orders** query and select **Rename**.

2. The name is already good (Orders), so just press **Enter**.

3. Right-click the **Order_Details** query and select **Rename**.

4. Change the name to **OrderDetails** (remove the underscore) and press **Enter**.

5. Right-click the **products** query and select **Rename**.

6. Change the name to **Products** (capitalize the P) and press **Enter**.

7. Right-click the **customers** query and select **Rename**.

8. Change the name to **Customers** (capitalize the C) and press **Enter**.

### Load Data into Power BI Desktop

1. Review your four queries in the Queries pane:
   - Orders
   - OrderDetails
   - Products
   - Customers

2. On the **Home** tab in Power Query Editor, select **Close & Apply**.

3. Power BI will now load all four tables into your data model. You'll see a progress dialog as the data is imported.

4. Once loading is complete, you're returned to the Power BI Desktop report canvas.

5. In the **Fields** pane on the right, you should now see all four tables listed with their columns:
   - Customers
   - OrderDetails
   - Orders
   - Products

> **Congratulations!** You've successfully connected to and imported data from multiple sources. Your data is now ready to be modeled and analyzed.

---

## Task 3: Build a Data Model with Relationships

A well-designed data model is the foundation of effective reporting in Power BI. In this task, you'll create relationships between your tables using star schema design principles, where fact tables (Orders, OrderDetails) sit at the center and dimension tables (Customers, Products) provide context.

> **What is a Star Schema?** A star schema is a database design pattern optimized for reporting and analytics. It consists of:
> - **Fact Tables**: Contain measurements and metrics (Orders, OrderDetails)
> - **Dimension Tables**: Contain descriptive attributes (Customers, Products)
> - **Relationships**: One-to-many relationships from dimensions to facts
> 
> This design makes queries fast and reports easy to build.

### Switch to Model View

1. On the left side of Power BI Desktop, select the **Model** view icon (it looks like three connected boxes).

2. You should now see a diagram showing your four tables as boxes with their columns listed inside.

   > **Auto-detected Relationships:** Power BI often automatically detects and creates relationships based on matching column names. You may already see lines connecting your tables. If so, that's great! The following steps will help you verify and understand these relationships. If you don't see any relationships yet, we'll create them manually.

3. Arrange the tables by dragging them around the canvas so you can see them clearly:
   - Place **Customers** on the left
   - Place **Orders** in the center
   - Place **OrderDetails** to the right of Orders
   - Place **Products** on the far right

### Create Relationship: Customers to Orders

Let's create the first relationship connecting customers to their orders.

> **Note:** Power BI may have already auto-detected and created these relationships for you based on matching column names (CustomerID, OrderID, ProductID). If you see relationship lines already connecting your tables, you can skip the manual creation steps and proceed directly to step 4 to verify the relationship properties. The following steps show you how to manually create relationships if needed.

1. In the **Customers** table, locate the **CustomerID** column.

2. **If a relationship line doesn't already exist**, click and drag the **CustomerID** column from the **Customers** table to the **CustomerID** column in the **Orders** table. If a line already exists, proceed to step 4.

3. A line appears between the two tables, indicating a relationship has been created.

4. Double-click the relationship line to view its properties.

5. The **New relationship** dialog appears. Verify the following settings:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | Orders (CustomerID) |
   | To: Table (Column) | Customers (CustomerID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

   > **Understanding Cardinality:** 
   > - **Many to one (\*:1)**: Many orders can belong to one customer
   > - The "many" side is indicated by an asterisk (\*)
   > - The "one" side is indicated by a 1
   > - This is the most common relationship type in Power BI

6. Select **OK** to close the dialog.

7. You should now see a relationship line with **1** on the Customers side and **\*** on the Orders side.

### Create Relationship: Orders to OrderDetails

Now connect the Orders table to the OrderDetails table.

1. **If a relationship line doesn't already exist**, drag the **OrderID** column from the **Orders** table to the **OrderID** column in the **OrderDetails** table. If a line already exists, proceed to step 3.

2. A relationship line appears.

3. Double-click the relationship line to verify the settings:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | OrderDetails (OrderID) |
   | To: Table (Column) | Orders (OrderID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

4. Select **OK**.

### Create Relationship: Products to OrderDetails

Finally, connect products to order details.

1. **If a relationship line doesn't already exist**, drag the **ProductID** column from the **Products** table to the **ProductID** column in the **OrderDetails** table. If a line already exists, proceed to step 3.

2. A relationship line appears.

3. Double-click the relationship line to verify:

   | Setting | Value |
   | --- | --- |
   | From: Table (Column) | OrderDetails (ProductID) |
   | To: Table (Column) | Products (ProductID) |
   | Cardinality | Many to one (\*:1) |
   | Cross filter direction | Single |
   | Make this relationship active | ✓ Checked |

4. Select **OK**.

### Verify Your Data Model

Your completed data model should now look like this:

```
Customers (1) ──────────> (*) Orders (1) ──────────> (*) OrderDetails (*) <────────── (1) Products
   [Dimension]              [Fact]                        [Fact]                       [Dimension]
```

The relationships allow filters to flow from dimension tables to fact tables:
- Selecting a customer will filter all their orders and order details
- Selecting a product will filter all order details for that product
- This is called **filter propagation** and is fundamental to how Power BI reports work

### Review the Relationships in the Relationships Pane

1. On the **Home** tab in Model view, select **Manage relationships**.

2. The **Manage relationships** dialog shows all three relationships you created:
   - Customers (CustomerID) to Orders (CustomerID) - Active
   - Orders (OrderID) to OrderDetails (OrderID) - Active
   - Products (ProductID) to OrderDetails (ProductID) - Active

3. All relationships should show **Active** status.

4. Select **Close**.

### Test the Model with a Quick Visual

Let's verify that the relationships work correctly by creating a quick test visual.

1. Select the **Report** view icon on the left (it looks like a bar chart).

2. In the **Fields** pane, expand the **Customers** table and check the box next to **Country**.

3. Expand the **OrderDetails** table and check the box next to **Quantity**.

4. Select the Table visual from the Visualizations pane. A table appears showing countries and the total quantity of products ordered.

5. This confirms that your relationships are working correctly - the filter is propagating from Customers → Orders → OrderDetails!

6. You can delete this test visual (select it and press **Delete**) or keep it for reference.

> **Congratulations!** You've successfully built a star schema data model with proper relationships. This model is now optimized for creating reports and analyzing your sales data.

---

## Task 4: Create Interactive Reports in Power BI Desktop

Now that your data model is ready, it's time to create interactive visualizations that answer business questions. You'll build three report pages, each focusing on different aspects of the sales data.

> **Best Practice:** Organize your reports into logical pages that focus on specific themes or audiences. This makes it easier for users to find the information they need.

### Page 1: Sales Overview

Let's create the first page showing high-level sales KPIs and trends.

#### Create KPI Cards

1. Ensure you're in **Report** view (select the bar chart icon on the left if needed).

2. You should see a blank report page labeled "Page 1" at the bottom.

3. Let's create a card showing total revenue. In the Visualizations pane, select the ellipsis (...) and then select **Restore default visuals**. After that, select the Card icon (without the lightning bolt) in the Visualizations pane.

4. A card visual appears on your canvas.

5. From the **Fields** pane, expand **OrderDetails** and drag **UnitPrice** to the card visual.

6. The card now shows a sum of all unit prices, but we need to calculate the actual revenue. Click on the card to select it.

7. In the **Visualizations** pane, under **Fields**, you'll see **Sum of UnitPrice** in the Values well.

8. Let's create a measure instead. Right-click on the **OrderDetails** table in the Fields pane and select **New measure**.

9. In the formula bar at the top, type the following DAX formula:
   ```
   Total Revenue = SUMX(OrderDetails, OrderDetails[UnitPrice] * OrderDetails[Quantity] * (1 - OrderDetails[Discount]))
   ```

10. Press **Enter** to create the measure.

11. Now select your card visual again, remove the **UnitPrice** field, and drag the new **Total Revenue** measure to the card.

12. The card displays the total revenue across all orders.

13. Format the card:
    - With the card selected, go to the **Format your visual** pane (paint roller icon)
    - Select the Visual tab
    - Expand **Callout value**
    - Set **Display units** to **Thousands**
    - Set **Value decimal places** to **0**
    - Turn **Category label** to **Off** (optional, hides the "Total Revenue" label on the card)

14. Resize the card to make it smaller and position it in the upper-left corner of the canvas.

15. Add a title to the card:
    - In the Format your visual pane, expand **General** → **Title**
    - Turn **Title** to **On**
    - Expand **Title**
    - Change **Text** to "Total Revenue"

#### Create Order Count Card

1. Add another Card visual to the canvas from the Visualizations pane.

2. Right-click the **Orders** table in the Fields pane and select **New measure**.

3. Type the following DAX formula:
   ```
   Total Orders = COUNTROWS(Orders)
   ```

4. Press **Enter**.

5. Drag the **Total Orders** measure to your new card visual.

6. Format the card similarly (display units: Auto, no decimal places, Category label: Off).

7. Add title: "Total Orders"

8. Position this card to the right of the Total Revenue card.

#### Create Average Order Value Card

1. Add a third card visual.

2. Right-click the **Orders** table and select **New measure**.

3. Type the following DAX formula:
   ```
   Average Order Value = DIVIDE([Total Revenue], [Total Orders], 0)
   ```

4. Press **Enter**.

5. Drag the **Average Order Value** measure to the card.

6. Format: Display units None, 2 decimal places, Category label: Off.

7. Add title: "Average Order Value"

8. Position this card to the right of Total Orders.

#### Create Sales Trend Line Chart

1. Below the KPI cards, add a **Line chart** visual from the Visualizations pane.

2. Configure the visual:
   - **X-axis**: Drag **OrderDate** from the Orders table 
   - **Y-axis**: Drag the **Total Revenue** measure
   - **Legend**: Leave empty for now

3. Format the line chart:
   - Select the visual
   - Go to Format your visual pane → **X-axis**
   - Turn **Title** to **On**
   - Change **Title text** to "Order Date"
   - Go to **Y-axis**
   - Turn **Title** to **On**
   - Change **Title text** to "Revenue"

4. Resize the chart to take up about 40% of the page width.

#### Create Orders by Country Bar Chart

1. Add a **Clustered bar chart** visual (horizontal bars).

2. Configure:
   - **Y-axis**: Drag **ShipCountry** from Orders table
   - **X-axis**: Drag **Total Orders** measure

3. Sort the chart by Total Orders descending:
   - Select the visual
   - Click the **More options** (...) at the top-right of the visual
   - Select **Sort by** → **Total Orders**
   - Select **Sort by** → **Sort descending**

4. Add title: "Orders by Country"
   - In Format your visual pane, go to the General tab
   - Turn **Title** to **On**
   - Change **Text** to "Orders by Country"

5. Position this chart to the right of the line chart.

#### Add a Date Slicer

1. Add a **Slicer** visual from the Visualizations pane.

2. Drag **OrderDate** from the Orders table to the **Field** well.

3. Change the slicer style:
   - Select the slicer
   - Go to Format your visual pane → Visual → Slicer settings
   - Expand **Options**
   - Select **Between** (shows a slider with two handles for date range selection)

4. Position the slicer at the bottom of the page, spanning the full width.

5. Add title: "Select Date Range"
   - Select the General tab in the Format your visual pane
   - Turn **Title** to **On**
   - Change **Text** to "Select Date Range"

6. Turn Background to **On**
   - In Format your visual pane, go to General tab
   - Expand Effects
   - Turn **Background** to **On**

#### Rename Page 1

1. At the bottom of the screen, right-click **Page 1** and select **Rename**.

2. Type **Sales Overview** and press **Enter**.

### Page 2: Customer Analysis

Let's create a second page focused on customer insights.

1. At the bottom of the screen, select the **+** icon to add a new page.

2. A new blank page (**Page 2**) is created.

#### Create Top Customers Table

1. Add a **Table** visual from the Visualizations pane.

2. Configure the table with these fields from the Customers table:
   - CompanyName
   - Country
   - City

3. Add the **Total Revenue** measure to the table.

4. Add the **Total Orders** measure to the table.

5. Sort the table by Total Revenue descending:
   - Click the **More options** (...) on the visual
   - Select **Sort by** → **Total Revenue**
   - Select **Sort descending**

6. Resize the table to take up the left half of the page.

7. Add title: "Top Customers by Revenue"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Top Customers by Revenue"

#### Create Customer Treemap

1. Add a **Treemap** visual from the Visualizations pane.

2. Configure:
   - **Category**: Drag **Country** from Customers table
   - **Values**: Drag **Total Revenue** measure

3. The treemap displays rectangles showing revenue by country, with larger rectangles representing higher revenue.

4. Position the treemap on the right side of the page, taking up the remaining space.

5. Add title: "Revenue by Customer Location"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Revenue by Customer Location"

#### Add Country Slicer

1. Add a **Slicer** visual.

2. Drag **Country** from Customers table to the Field well.

3. Format:
   - Go to Format pane → Visual → Slicer settings → Selection
   - Enable **Show "Select all" option**
   
4. Position the slicer at the bottom of the page.

5. Add title: "Filter by Country"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Filter by Country"

6. Turn Background to **On**
   - In Format your visual pane, go to General tab
   - Expand Effects
   - Turn **Background** to **On**

#### Rename Page 2

1. Right-click **Page 2** at the bottom and select **Rename**.

2. Type **Customer Analysis** and press **Enter**.

### Page 3: Product Performance

Create a third page focusing on product-level insights.

1. Add a new page by clicking the **+** icon at the bottom.

#### Create Product Sales Table

1. Add a **Table** visual.

2. Configure with these fields:
   - **ProductName** from Products table
   - **CategoryID** from Products table (shows product category)
   - **Total Revenue** measure
   - **Quantity** from OrderDetails table (Power BI will sum it automatically)

3. Sort by Total Revenue descending.
   - Click the **More options** (...) on the visual
   - Select **Sort by** → **Total Revenue**
   - Select **Sort descending**

4. Resize to take up about half of the page width on the left side.
   
5. Add title: "Product Sales Performance"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Product Sales Performance"

#### Create Top 10 Products Column Chart

1. Add a **Clustered column chart** visual (vertical bars).

2. Configure:
   - **X-axis**: Drag **ProductName** from Products table
   - **Y-axis**: Drag **Total Revenue** measure

3. Show only top products:
   - Select the visual
   - Go to **Filters** pane on the right
   - Expand the **ProductName** filter
   - Change **Filter type** to **Top N**
   - Set **Show items** to **Top 10**
   - Set **By value** to **Total Revenue**
   - Select **Apply filter**

4. Format the chart:
   - Resize X-axis labels for better readability
   - Go to Format your visual pane → **Visual** → **X-axis**
   - Set **Text size** to 8

5. Position on the right side of the page.

6. Add title: "Top 10 Products by Revenue"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Top 10 Products by Revenue"

#### Add Category Slicer

1. Add a **Slicer** visual.

2. Drag **CategoryID** from Products table to the Field well.

3. Format as a vertical list.
   - Go to Format your visual pane → Visual → Slicer settings → Options
   - Select **Vertical list** (shows all categories in a list)

4. Position at the bottom of the page.

5. Add title: "Filter by Category"
   - In Format your visual pane, go to General tab
   - Turn **Title** to **On**
   - Change **Text** to "Filter by Category"

6. Turn Background to **On**
   - In Format your visual pane, go to General tab
   - Expand Effects
   - Turn **Background** to **On**

#### Rename Page 3

1. Right-click **Page 3** and select **Rename**.

2. Type **Product Performance** and press **Enter**.

### Test Interactivity

Let's verify that your visualizations interact with each other correctly.

1. Go back to the **Sales Overview** page.

2. Click on one of the bars in the "Orders by Country" chart (e.g., USA).

3. Notice how all other visuals on the page automatically filter to show only data for that country. This is called **cross-filtering**.

4. Click the same bar again to deselect and reset the filters.

5. Go to the **Customer Analysis** page.

6. Select a country in the country slicer.

7. The table and treemap update to show only customers and data from that country.

8. Test the **Product Performance** page by selecting a category in the slicer.

9. The table and column chart update to show only products from that category.

> **Cross-filtering:** When you click on a visual element (bar, slice, data point), Power BI automatically filters related visuals on the same page. You can control this behavior using Format → Edit interactions.

### Save Your Work

1. Navigate to the Sales Overview Page. Press **Ctrl + S** to save your Power BI Desktop file.

> **Congratulations!** You've created a complete three-page interactive report with KPIs, trends, tables, charts, treemaps, and slicers. Your report is now ready to publish to the Power BI Service!

---

## Task 5: Publish Your Report to the Power BI Service

The Power BI Service is the online platform where you share, collaborate, and consume Power BI content. Publishing your report makes it accessible from anywhere via a web browser.

> **What Happens When You Publish?** Publishing uploads your .pbix file to the Power BI Service, including:
> - Your data model (tables, relationships, measures)
> - All report pages and visualizations
> - Any bookmarks or page-level filters
> 
> The published report will be refreshed with the data as it exists at the time of publishing.

### Publish the Report

1. In Power BI Desktop, ensure your file is saved (**Ctrl + S**).

2. On the **Home** ribbon, select **Publish**.

3. If prompted to save changes, select **Save**.

4. The **Publish to Power BI** dialog appears showing your available workspaces.

5. Select **My workspace** (or another workspace you have access to).

   > **Note:** "My workspace" is your personal workspace. In a team environment, you would typically publish to a shared workspace where colleagues can access the content.

6. Select **Select** to begin the publishing process.

7. Power BI uploads your report and data to the service. You'll see a progress dialog.

8. When publishing completes, you'll see a **Success!** message with a link to your report.

9. Select the **Open 'SalesAnalysis.pbix' in Power BI** link to open your report in the browser.

   > **Tip:** You can also navigate to [https://app.powerbi.com](https://app.powerbi.com) manually and find your report in "My workspace."

### Explore Your Report in the Power BI Service

Your report opens in the Power BI Service in your web browser.

1. You should see the same report you created in Power BI Desktop, starting with the **Sales Overview** page.

2. Test the interactivity:
   - Click on different elements in the visuals
   - Use the slicers to filter data
   - Navigate between pages using the page tabs at the bottom

3. Notice the toolbar at the top with options like:
   - **Edit**: Opens the report in editing mode (browser-based editing)
   - **Share**: Share the report with others
   - **Export**: Export the report to PowerPoint or PDF
   
### Explore the Workspace

1. In the left navigation pane, select **My workspace**.

2. You should see two items:
   - **SalesAnalysis** (Type Report) - The report you just published
   - **SalesAnalysis** (Type Semantic model) - The data model

   > **Report vs. Semantic model:** 
   > - The **Semantic model** contains your data model (tables, relationships, measures)
   > - The **Report** contains your visualizations and pages
   
3. Get Quick Insights:
   - Hover over the SalesAnalysis report
   - Click the **More options** (...) that appears
   - Select **Quick insights**
   - Power BI runs automated analysis on your data and generates insights like trends, outliers, and correlations.

### Understanding Refresh

1. Go back to your workspace and select the **SalesAnalysis** Semantic model.

2. Select **Refresh now** to manually refresh the data from your original sources.

   > **Note:** For this lab, the data sources are static (CSV files and a demo OData service), so refreshing won't change the data. In a production environment with live data sources, you would schedule automatic refreshes to keep your reports up to date.

3. You can also select **Schedule refresh** to set up automatic daily or weekly refreshes.

4. For now, we'll skip scheduling refresh since our data sources are for learning purposes.

> **Congratulations!** Your report is now published to the Power BI Service and accessible from any device with a web browser. Next, you'll create a dashboard to highlight the most important insights.

---

## Task 6: Create and Share a Dashboard in the Power BI Service

Dashboards in Power BI provide a single-page view of your most important metrics and insights. Unlike reports (which can have multiple pages), a dashboard is always a single page composed of tiles pinned from one or more reports.

> **Report vs. Dashboard:**
> - **Report**: Multi-page, interactive, created in Power BI Desktop
> - **Dashboard**: Single-page, tiles pinned from reports, created only in Power BI Service
> - **Use case**: Dashboards are great for executives and stakeholders who want quick access to KPIs without navigating through multiple report pages

### Create a New Dashboard

1. In the Power BI Service, open your **SalesAnalysis** report (if it's not already open).

2. Navigate to the **Sales Overview** page.

3. Let's pin the Total Revenue card to a new dashboard:
   - Hover over the **Total Revenue** card visual
   - Click the **Pin** icon (looks like a pushpin) that appears in the top-right or bottom-right corner of the visual

4. The **Pin to dashboard** dialog appears.

5. Select **New dashboard**.

6. Enter the name: **Sales Performance Dashboard**

7. Select **Pin**.

8. A confirmation message appears saying "Pinned to dashboard" with a link to view it.

### Pin Additional Visuals

Let's add more tiles to our dashboard from different report pages.

1. Remain on the **Sales Overview** page.

2. Pin the **Total Orders** card:
   - Hover over the card
   - Click the **Pin** icon
   - Select **Existing dashboard**
   - Choose **Sales Performance Dashboard**
   - Select **Pin**

3. Pin the **Average Order Value** card using the same process.

4. Pin the **Orders by Country** bar chart.

5. Navigate to the **Customer Analysis** page.

6. Pin the **Top Customers by Revenue** table.

7. Pin the **Revenue by Customer Location** treemap.

8. Navigate to the **Product Performance** page.

9. Pin the **Top 10 Products by Revenue** chart.

### View and Arrange Your Dashboard

1. In the left navigation pane, select **My workspace**.

2. You should now see your **Sales Performance Dashboard** in the list.

3. Click on the dashboard name to open it.

4. Your dashboard displays all the tiles you pinned from your report, arranged in a default layout.

5. Let's rearrange the tiles for a better layout:
   - Drag tiles to rearrange them
   - Resize tiles by dragging the corner handles
   - Suggested layout:
     - Row 1: Three KPI cards (Total Revenue, Total Orders, Avg Order Value) side by side
     - Row 2: Orders by Country chart (left half) and Revenue by Customer Location treemap (right half)
     - Row 3: Top Customers by Revenue table (left half) and Top 10 Products by Revenue chart (right half)


### Interact with Dashboard Tiles

Dashboard tiles are interactive:

1. Click on any tile on your dashboard (e.g., click the Total Revenue card).

2. Power BI navigates you back to the source report and the specific page where that visual exists.

3. This allows dashboard viewers to quickly drill into details when they see something interesting.

4. Use the browser's back button  to return to the dashboard.

### Share Your Dashboard

Now let's share the dashboard with colleagues.

1. On the dashboard, select **Share** at the top of the screen.

2. The **Share dashboard** dialog appears.

3. In the **Enter a name or email address** field, enter the email address of someone you want to share with.

   > **Important:** The recipient must have a Power BI Pro license to view shared content and must be in the same organization (same email domain). If they don't have access, they'll receive an error when trying to view the dashboard.

4. Review the sharing options:
   - **Allow recipients to share your dashboard** - Check this if you want recipients to be able to share with others
   - **Allow recipients to build content with the data associated with this dashboard** - Check this if you want recipients to create new reports using your dataset
   - **Send an email notification** - Check this to send an email with a link to the dashboard

5. Select **Grant access**.

6. The recipient receives an email notification with a link to access the dashboard.

### Set Up Dashboard Alerts (Optional)

You can configure alerts to notify you when data reaches certain thresholds.

1. Hover over the **Total Revenue** tile.

2. Click the **More options** (...) at the top-right of the tile.

3. Select **Manage alerts**.

4. Select **+ Add alert rule**.

5. Configure the alert:
   - **Condition**: Above
   - **Threshold**: Enter a value (e.g., 1,000,000)
   - **Maximum notification frequency**: Every 24 hours
   - **Send me email too**: Check this to receive email notifications

6. Select **Save and close**.

7. You'll receive an email notification when the total revenue exceeds your threshold.

> **Note:** Alerts only work with certain visual types (cards, gauges, KPIs) and only for data that changes (not static data like in this lab).

### Subscribe to the Dashboard (Optional)

Subscriptions send regular email snapshots of your dashboard.

1. On the dashboard, select **Subscribe to dashboard**.

2. Select **Create a subscription**.

3. Configure the subscription:
   - **Subscription name**: Enter a name (e.g., "Daily Sales Snapshot")
   - **Start date**: Select a specific date
   - **End date**: Select a specific date
   - **Repeat**: Hourly, Daily, Weekly, or Monthly
   - **Scheduled time**: Select a specific time
   - **Time zone**: Select your time zone
   - **Email subject**: Enter a subject for the email (e.g., "Daily Sales Performance Dashboard Snapshot")
   - **Message**: Enter a message for the email 

4. Select **Save**.

5. You will receive email snapshots of the dashboard at the specified frequency.

> **Congratulations!** You've successfully created a dashboard in the Power BI Service, arranged tiles for optimal viewing, and shared it with colleagues. Your team can now access real-time sales insights from anywhere!

---

## Task 7: Create a Date Table and Time Intelligence Measures

In this task, you will create a proper Date table and use it to build time intelligence measures that compare sales performance across different time periods.

> **Why a dedicated Date table?** Power BI's built-in auto date/time hierarchy works for basic charts, but for accurate time intelligence calculations (YTD, year-over-year, etc.) you need a dedicated Date table marked as a date table. This ensures DAX time intelligence functions work correctly in all scenarios.
>
> *Reference: [Use date tables in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-date-tables)*

### Create the Date Table Using DAX

1. In Power BI Desktop, switch to **Table** view.

2. On the **Home** ribbon, select **New table** from the **Calculations** group.

3. In the formula bar, enter the following DAX expression:
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

   > **What this does:** `CALENDAR` generates one row per day between the two dates (covering the Northwind data range). `ADDCOLUMNS` enriches each row with derived columns for Year, Month, Quarter, and a sortable Year-Month string.

4. Press **Enter** to create the table.

### Mark the Table as a Date Table

1. In the **Fields** pane, select the **Date** table.

2. On the **Table tools** ribbon, select **Mark as date table**.

3. In the dialog, set the following
   - Enable **Mark as date table** 
   -  **Choose a date column**: **Date**.

4. Select **Save**.

   > *Reference: [Set and use date tables in Power BI Desktop](https://learn.microsoft.com/power-bi/transform-model/desktop-date-tables#set-your-own-date-table)*

### Create the Relationship to Orders

1. Switch to **Model** view.

2. Drag the **Date** column from the **Date** table to the **OrderDate** column in the **Orders** table.

3. Verify the relationship properties:

   | Setting | Value |
   |---|---|
   | Cardinality | One to many (1:*) |
   | Cross filter direction | Single |
   | Active | ✓ Yes |

4. Select **Save**.

### Create Time Intelligence Measures

1. Switch to **Report** view.

2. Right-click the **OrderDetails** table in the Fields pane and select **New measure**.

3. Create a Year-to-Date revenue measure:
   ```
   Revenue YTD = TOTALYTD([Total Revenue], 'Date'[Date])
   ```

4. Add a same-period last year measure:
   ```
   Revenue Last Year = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date]))
   ```

5. Add a Year-over-Year growth percentage:
   ```
   YoY Growth % =
   DIVIDE(
       [Total Revenue] - [Revenue Last Year],
       [Revenue Last Year],
       0
   )
   ```
   - On the **Measure tools** ribbon, set **Format** to **Percentage**.
   - Set the decimal places to **2**.

   > *Reference: [Time intelligence functions (DAX)](https://learn.microsoft.com/dax/time-intelligence-functions-dax)*

### Add a Sales Analysis Page

1. Add a new report page by selecting the **+** icon at the bottom of the Power BI Desktop window.

2. Right-click the new page tab, select **Rename**, type **Sales Analysis**, and press **Enter**.

3. Add a **Slicer** visual and drag **Year** from the Date table to the Field well.
   - In the Format your visual pane, go to the Visual tab → **Slicer Settings** → **Options** and set Style to `Vertical list`
   - Expand **Slicer Settings** → **Selection** and set Single select to `On`.
   - Go to the General tab and turn the Title **On**  
   - Expand **Title** and set Text to `Select Year`
   - Expand **Effects** and set Background to **On**
   
4. Add three **Card** visuals across the top of the page:
   - **Revenue YTD** measure
   - **Revenue Last Year** measure
   - **YoY Growth %** measure

5. Add a **Table** visual below the cards. Configure it with:
   - **CompanyName** from the Customers table
   - **Revenue YTD** measure
   - **Revenue Last Year** measure
   - **YoY Growth %** measure

   Sort the table by **Revenue YTD** descending.

6. Add a **Line and clustered column chart** visual to the right of the table.

7. Configure the visual:

   | Well | Field |
   |---|---|
   | X-axis | **Year-Month** from the Date table |
   | Column y-axis | **Total Revenue** measure |
   | Line y-axis | **Revenue Last Year** measure |

   Sort by **Year-Month** ascending.

8. Add a title: **Revenue: Current vs Prior Year**
   - In the Format your visual pane, go to the General tab → **Title** → set Text to `Revenue: Current vs Prior Year`
   - Turn the Subtitle **On**
   - Expand **Subtitle** and set Text to `Comparison of current year revenue versus prior year`

9. For the **YoY Growth %** card, in the Format your visual pane → **Visual** → **Callout value**, set **Display units** to **Auto** and **Value decimal places** to **2**. Confirm that the percentage format from the measure is preserved.

10. Arrange the page so the year slicer is on the left, the customer table is in the centre, and the comparison chart is on the right, matching the lab screenshot.

11. Press **Ctrl + S** to save.

> **Congratulations!** Your Sales Overview page now includes time intelligence KPIs backed by a proper Date table — the foundation of accurate date-based analysis in Power BI.

---

## Task 8: Conditional Formatting and Report Themes

In this task, you will apply a report theme for consistent styling and use conditional formatting to make tables more visually informative at a glance.

### Apply a Report Theme

1. In Power BI Desktop, select the **View** ribbon.

2. Select the **Themes** dropdown.

3. Browse through the available built-in themes.

4. Select any built-in theme to apply it to your report.

   > **Why themes matter:** Themes enforce consistent colours, fonts, and visual formatting across every page of your report. When a theme is updated, all visuals update automatically.
   >
   > *Reference: [Report themes in Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-report-themes)*

5. Observe how all visuals update immediately to reflect the new colour palette.

6. Visit the [Power BI theme gallery](https://community.fabric.microsoft.com/category/pbi_comm_galleries/discussions/themesgallery) to explore and download additional themes.

7. Download and apply a theme from the gallery to see how it changes the look and feel of your report.
   - Download a theme file (usually a `.json` file) from the gallery.
   - In Power BI Desktop, go to the **View** ribbon → **Themes** dropdown → **Browse for themes** and select the downloaded `.json` file to apply it.

### Add Conditional Formatting — Data Bars

1. Navigate to the **Sales Analysis** page.

2. Select the Company Revenue table visual.

3. In the **Visualizations** pane, select the **Format your visual** tab (paint roller icon).

4. Expand **Visual** → **Cell elements**.

5. Set Series to **Revenue YTD** then toggle **Data bars** to **On**.

   > Data bars render an inline horizontal bar inside each cell, giving an immediate visual sense of relative values without requiring a separate chart visual.

### Add Conditional Formatting — Rules

1. Still in the **Cell elements** section of the same table, set Series to **YoY Growth %**.

2. Toggle **Background color** to **On**.

3. Select the **fx** button to configure:
   - **Format style**: Rules
   - **Rule 1:** If value >= -100 Number and < 0 Number then colour: #FF0000
   - **Rule 2:** If value >= 0 Number and < 0.25 Number then colour: #FFFF00
   - **Rule 3:** If value >= 0.25 Number and < 1000 Number then colour: #00FF00

4. Select **OK**.

### Add Conditional Formatting — Icons

1. Still in the **Cell elements** section of the same table, set Series to **YoY Growth %**.

2. Toggle **Icons** to **On**.

3. Select the **fx** button to configure:
   - **Format style**: Rules
   - **Icon layout**: **Right of data**
   - **Rule 1:** If value >= -100 Number and < 0 Number then Icon: Gray downward-slanting arrow
   - **Rule 2:** If value >= 0 Number and < 0.25 Number then Icon: Gray side arrow
   - **Rule 3:** If value >= 0.25 Number and < 1000 Number then Icon: Green upward-slanting arrow

4. Select **OK**.

   > Icons give users an at-a-glance status indicator, reducing cognitive load when scanning long product lists.

5. Press **Ctrl + S** to save.

> **Congratulations!** Your reports now have a consistent theme and tables enriched with conditional formatting that communicate performance levels visually.

---

## Task 9: Bookmarks, Buttons, and Report Navigation

In this task, you will add a navigation bar so users can move between report pages, and create bookmarks to toggle between two different customer views.

### Add a Page Navigator

1. Navigate to the **Sales Overview** page.

2. On the **Insert** ribbon, select **Buttons** → **Navigator** → **Page navigator**.

   > **Page navigator** automatically generates one button per report page and stays in sync when pages are added or renamed — no manual wiring required.
   >
   > *Reference: [Create a page navigation button in Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-buttons#create-a-page-navigation-button)*

3. A navigation bar appears on the canvas. Drag it to the top of the page and resize it to span the full width. Adjust the other visual elements on the page as needed to accommodate the navigation bar.

4. In the **Format** pane, customise the button appearance:
   - Go to **Visual** → **Shape**
   - Set **Shape** to **Rounded rectangle (%)**
   - Set **Corner radius** to **25%**

5. Copy the navigation bar (**Ctrl + C**), navigate to the **Customer Analysis** page, and paste (**Ctrl + V**). Position it identically.

6. Repeat for the **Product Performance** and **Sales Analysis** pages.

### Open the Bookmarks and Selection Panes

1. Navigate to the **Customer Analysis** page.

2. On the **View** ribbon, enable both **Bookmarks pane** and **Selection pane**.

### Create an "All Customers" Bookmark

1. Ensure no filters are active.

2. In the **Bookmarks** pane, select **Add**.

3. Double-click the new bookmark and rename it **All Customers**.

4. Right-click the bookmark and select **Update**. Ensure **Data** is checked and **Display** is unchecked.

   > Capturing **Data** only (not Display) means the bookmark saves the filter state, not the scroll position or visual selection.

### Create a "Top 15 Customers" Bookmark

1. Select the **Top Customers by Revenue** table visual.

2. In the **Filters** pane, expand the **CompanyName** filter well.

3. Set **Filter type** to **Top N**, **Show items** to **Top 15**, **By value** to **Total Revenue**.

4. Select **Apply filter**. The table now shows only the top 15 customers.

5. In the Bookmarks pane, select **Add**.

6. Rename the bookmark to **Top 15 Customers**.

7. Right-click → **Update** → ensure **Data** is checked, **Display** unchecked.

   > *Reference: [Bookmarks capture the state of a report page](https://learn.microsoft.com/power-bi/create-reports/desktop-bookmarks#bookmarks-capture-the-state-of-a-report-page)*

### Wire Buttons to Bookmarks

1. On the **Insert** ribbon, select **Buttons** → **Blank**.

2. In the **Format** pane:
   - Go to **Button** tab
   - Expand Style section
   - Turn **Text** on
   - Set **Text** to **All Customers**
   - Turn **Action** on
   - Set **Type** to **Bookmark**
   - Set **Bookmark** to **All Customers**

3. Add a second blank button. Set its Text to **Top 15 Customers** and Action → Bookmark → **Top 15 Customers**.

4. Position both buttons side by side in the bottom-left area of the Customer Analysis page, below the Top Customers by Revenue table.

5. To test: hold **Ctrl** and click each button to switch between the two views.

6. Go to the Sales Overview page. Press **Ctrl + S** to save.

7. Publish your report to the Power BI Service if you haven't already and verify that the bookmarks and buttons work as expected online.

> **Congratulations!** Your report now has professional page navigation, conditional formatting-enhanced tables, and bookmark-driven interactive views — completing a full 150-minute end-to-end Power BI solution.

---

## Lab Summary

In this lab, you've learned the fundamentals of Power BI by completing a comprehensive end-to-end workflow:

### What You Accomplished

✅ **Task 1**: Set up Power BI Desktop and signed into the Power BI Service

✅ **Task 2**: Connected to multiple data sources (OData service and CSV files) and imported data

✅ **Task 3**: Built a star schema data model with relationships between customers, orders, and products

✅ **Task 4**: Created three interactive report pages with various visualizations:
   - Sales Overview: KPIs, trends, and geographic analysis
   - Customer Analysis: Top customers and location-based insights
   - Product Performance: Product sales and category filtering

✅ **Task 5**: Published your report to the Power BI Service

✅ **Task 6**: Created and shared a dashboard highlighting key metrics

✅ **Task 7**: Created a Date table, marked it as a date table, and built time intelligence measures (YTD, year-over-year)

✅ **Task 8**: Applied a report theme and used conditional formatting (data bars, colour scales, icons) to improve data readability

✅ **Task 9**: Added a page navigation bar, created bookmarks, and wired buttons to create an interactive view toggle

### Key Concepts You Learned

- **Power BI Desktop vs. Service**: Desktop for creating, Service for sharing
- **Data connections**: OData feeds and CSV file imports
- **Import mode**: Data is copied into Power BI for fast performance
- **Star schema design**: Fact and dimension tables with one-to-many relationships
- **Data modeling**: Creating relationships with drag-and-drop
- **DAX measures**: Calculated values like Total Revenue and Total Orders
- **Interactive visualizations**: Cards, charts, tables, maps, and slicers
- **Cross-filtering**: How visuals interact and filter each other
- **Publishing workflow**: From Desktop to Service
- **Dashboards**: Single-page views with pinned tiles from reports
- **Sharing**: Collaborating with colleagues (requires Power BI Pro for all users)
- **Date tables**: Dedicated calendar tables marked for time intelligence
- **Time intelligence**: YTD, year-over-year, and period comparison using DAX
- **Report themes**: Consistent styling applied across all visuals
- **Conditional formatting**: Data bars, colour scales, and icons in tables
- **Bookmarks**: Saved report states wired to buttons for interactive navigation

### Next Steps

Now that you've completed the basics, consider exploring these advanced topics:

- **Advanced DAX**: Create more sophisticated calculations with time intelligence, ranking, and statistical functions
- **Custom visuals**: Download additional visualizations from the AppSource marketplace
- **Q&A**: Ask natural language questions about your data
- **Power BI Mobile**: Access your reports and dashboards on mobile devices
- **Dataflows**: Create reusable data preparation logic in the Power BI Service
- **Paginated reports**: Create pixel-perfect reports for printing and PDF export
- **Row-level security**: Restrict data access based on user roles

### Additional Resources

- [Microsoft Power BI Documentation](https://learn.microsoft.com/power-bi/)
- [Power BI Community Forums](https://community.powerbi.com/)
- [Power BI Guided Learning](https://learn.microsoft.com/training/powerplatform/power-bi)
- [DAX Reference Guide](https://dax.guide/)

**Thank you for completing this lab!** You now have practical experience with Power BI and are ready to create your own reports and dashboards using real-world data.
