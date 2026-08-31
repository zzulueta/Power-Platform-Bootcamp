# Power BI Advanced Training — 3-Hour Session

**Audience:** Power BI users with intermediate experience  
**License:** Power BI Pro (required); Deployment Pipelines require a Fabric capacity workspace (any F SKU, including F2) or PPU — trainer demo if unavailable; Copilot requires any Fabric F SKU (F2+) or PPU — trainer demo if unavailable  
**Format:** Short presentation followed by hands-on labs  
**Duration:** 3.5 Hours  

> **Note:** Labs A, B, and D are fully hands-on with a Power BI Pro license. Lab C (Deployment Pipelines) requires workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or PPU — trainer-led if neither is available. Lab E AI visuals are available to all participants in Power BI Desktop; Copilot features require **any Fabric F SKU (F2+) or PPU** and are trainer-led demo if unavailable.

---

## Agenda Overview

| Segment | Topic | Duration |
|---|---|---|
| 1 | Welcome & Recap | 10 min |
| 2 | Presentation: Architecture, Fabric Integration & Governance at Scale | 35 min |
| 3 | Lab A: Performance Tuning & DAX Optimisation | 25 min |
| 4 | Lab B: Row-Level Security, Dynamic RLS & Object Level Security | 40 min |
| — | Break | 10 min |
| 5 | Lab C: Deployment Pipelines & Report Governance | 25 min |
| 6 | Lab D: Sharing Semantic Models | 30 min |
| 7 | Lab E: AI Visuals & Copilot Capabilities | 25 min |
| 8 | Wrap-Up, Q&A & Next Steps | 10 min |

---

## Segment 1: Welcome & Recap (10 minutes)

- Welcome and housekeeping
- Quick recap of Power BI Intermediate session concepts
- Session objectives and what participants will explore today
- Current Power BI licensing — now administered under Microsoft Fabric
  - **Fabric (Free):** personal use only (My Workspace); cannot share or collaborate — previously called "Power BI Free"
  - **Power BI Pro:** shared workspaces, publishing, sharing, scheduled refresh, app creation, report endorsement
  - **Power BI PPU (Premium Per User):** all Pro features + deployment pipelines, paginated reports, XMLA endpoint, advanced AI; content shared only with other PPU users
  - **Fabric F SKUs (capacity):** organisation-wide capacity; replaces the old Power BI Premium P SKUs; unlocks OneLake, Copilot (any F SKU, F2+), Direct Lake, deployment pipelines (any F SKU), and sharing with free users at scale
  - **Key point:** deployment pipelines are available with **any Fabric F SKU** (including F2) — a full PPU or Premium P SKU is no longer required
- Note: Pro and PPU retain their Power BI names; only the free tier (now "Fabric (Free)") and capacity SKUs (now "Fabric F SKUs") have been rebranded under Fabric

> **References:**
> - [Power BI service features by license type](https://learn.microsoft.com/power-bi/fundamentals/service-features-license-type)
> - [Power BI Premium Per User FAQ](https://learn.microsoft.com/power-bi/enterprise/service-premium-per-user-faq)
> - [Microsoft Fabric capacity licenses (F SKUs)](https://learn.microsoft.com/fabric/enterprise/licenses)

---

## Segment 2: Presentation — Architecture, Fabric Integration & Governance at Scale (35 minutes)

### Power BI Architecture & Microsoft Fabric (10 minutes)
- The Power BI end-to-end architecture: Desktop → Service → Consumer
- Where Power BI sits within **Microsoft Fabric** — the unified analytics platform
- **OneLake** — one logical data lake for all Fabric workloads; Power BI semantic models as a first-class citizen
- **Direct Lake mode** — query data in OneLake without import or caching overhead
- Semantic models as a shared, reusable enterprise layer — decouple the model from reports
- Import mode vs. DirectQuery vs. Direct Lake — when to use each

> **References:**
> - [What is Microsoft Fabric?](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview)
> - [OneLake overview](https://learn.microsoft.com/fabric/onelake/onelake-overview)
> - [Direct Lake overview](https://learn.microsoft.com/power-bi/enterprise/directlake-overview)
> - [Dataset modes in Power BI (Import, DirectQuery, Composite)](https://learn.microsoft.com/power-bi/connect-data/service-dataset-modes-understand)
> - [Power BI architecture and deployment patterns](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-introduction)

### Governance at Scale (15 minutes)
- **Workspace strategy** — workspace per team vs. per project vs. per domain; naming conventions and lifecycle management
- **Ownership models** — personal reports vs. team workspaces vs. certified enterprise datasets; avoiding orphaned content
- **Report endorsement** — Promoted vs. Certified; who can certify and what it signals to consumers
- **Sensitivity labels** — applying Microsoft Purview labels to datasets and reports; inheritance and downstream impact
- **Capacity planning** — understanding Premium/Fabric capacity units (CUs/v-cores), workspace assignment, and throttling
- **Power BI admin portal** — tenant settings, usage metrics, activity log, and governance guardrails

> **References:**
> - [Workspace planning overview](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-overview)
> - [Endorse content (Promoted and Certified)](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content)
> - [Sensitivity labels in Power BI](https://learn.microsoft.com/power-bi/enterprise/service-security-sensitivity-label-overview)
> - [Manage Premium capacity](https://learn.microsoft.com/power-bi/enterprise/service-admin-premium-manage)
> - [Power BI admin portal](https://learn.microsoft.com/power-bi/admin/service-admin-portal)

### Responsible Use at Enterprise Scale (10 minutes)
- **Data lineage** — use the lineage view in the Power BI Service to understand upstream/downstream dependencies
- **Impact analysis** — before changing a shared dataset, run impact analysis to see affected reports
- **Audit logs** — Power BI activity log and Microsoft 365 audit log for compliance and access reviews
- **DLP policies for Power BI** — Microsoft Purview DLP policies can restrict export of sensitive data from Power BI
- **Report accuracy accountability** — certified reports imply organizational trust; owners are accountable for data quality

> **Reference:** [Power BI governance, deployment, and adoption best practices](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-introduction)

---

## Segment 3: Lab A — Performance Tuning & DAX Optimisation (25 minutes)

**Business Context:** As Power BI reports grow in complexity and data volumes increase, slow-loading visuals and inefficient DAX become a real problem. Identifying and fixing performance bottlenecks makes reports faster for end users and reduces load on the underlying data source.

**Tool Used:** Power BI Desktop

**Steps:**
1. Open the provided `AdvancedSalesReport.pbix` file (pre-built with intentionally inefficient measures)
2. In Power BI Desktop, open **View → Performance Analyzer** and click **Start recording**
3. Refresh the report page and expand each visual in the Performance Analyzer pane — identify the visual with the highest **DAX query** time
4. Click **Copy query** on the slowest visual to extract the DAX query for review
5. Locate the underlying measure — examine its use of `FILTER` on a large table:
   ```
   // Inefficient: FILTER iterates the entire table
   Slow Measure = CALCULATE(SUM('Sales'[Amount]), FILTER('Sales', 'Sales'[Region] = "North"))
   ```
6. Rewrite the measure using a direct filter argument in `CALCULATE` instead:
   ```
   // Efficient: direct filter predicate — no full table scan
   Fast Measure = CALCULATE(SUM('Sales'[Amount]), 'Sales'[Region] = "North")
   ```
7. Replace the measure on the visual and re-run Performance Analyzer — compare the DAX query time before and after
8. Open **Power Query Editor → View → Query diagnostics** — click **Start diagnostics**, refresh a query, then **Stop diagnostics** to review query folding status
9. Identify a step that breaks query folding (indicated by a non-native step in diagnostics) and discuss the impact on refresh performance

**Key Learning Points:**
- Using Performance Analyzer to pinpoint slow visuals and extract DAX queries
- The performance cost of `FILTER` over a large table vs. direct filter predicates in `CALCULATE`
- Query folding in Power Query — what it is, why it matters, and how to spot when it breaks
- The principle: transform early and let the data source do as much work as possible

> **References:**
> - [Use Performance Analyzer in Power BI Desktop](https://learn.microsoft.com/power-bi/create-reports/desktop-performance-analyzer)
> - [DAX query performance best practices](https://learn.microsoft.com/power-bi/guidance/dax-variables)
> - [Power Query query folding](https://learn.microsoft.com/power-bi/guidance/power-query-folding)

---

## Segment 4: Lab B — Row-Level Security, Dynamic RLS & Object Level Security (40 minutes)

**Business Context:** In enterprise Power BI deployments, different users should only see data relevant to their role — a regional manager should see only their region's data, not the entire company's. Row-Level Security (RLS) enforces this at the data model level, regardless of which visual or page the user views.

**Tool Used:** Power BI Desktop + Power BI Service (Pro)

**Steps:**

**Part 1 — Static RLS (Power BI Desktop):**
1. Open `AdvancedSalesReport.pbix` in Power BI Desktop
2. Go to **Modeling → Manage roles** and create a new role named `North Region`
3. In the role editor, select the **Sales** table and add the filter: `[Region] = "North"`
4. Create a second role `South Region` with filter `[Region] = "South"`
5. Test the roles: **Modeling → View as → North Region** — verify only North data is visible across all visuals
6. Exit the role view and return to normal mode

**Part 2 — Dynamic RLS (Power BI Desktop):**
1. Add a **UserMapping** table to the model (via **Enter data**) with columns: `UserEmail` and `Region`, populated with sample user email addresses mapped to regions
2. Create a relationship between `UserMapping[Region]` and `Sales[Region]`
3. In **Manage roles**, create a new role named `Dynamic RLS` and apply the DAX filter to the **UserMapping** table:
   ```
   [UserEmail] = USERPRINCIPALNAME()
   ```
4. This ensures each user sees only rows where their email matches the UserMapping table — no static role per region needed
5. Test with **View as → Other user** and enter a sample email from the UserMapping table — verify the correct region data is returned

**Part 3 — Assign users to roles in the Power BI Service (Pro):**
1. Publish the report to a shared workspace in the Power BI Service
2. Navigate to the **dataset → Security** in the Service
3. Assign colleagues or test accounts to the `North Region` and `South Region` static roles
4. Discuss: with dynamic RLS, no manual role assignment is needed per user — the DAX expression handles it automatically

**Part 4 — Object Level Security (Power BI Desktop + Tabular Editor):**
1. Download and install **Tabular Editor 2** (free, open-source: [tabulareditor.com](https://tabulareditor.com)) if not already installed
2. In Power BI Desktop, go to **External Tools → Tabular Editor** to launch it connected to the open model
3. In Tabular Editor, expand the **Roles** node and select the `North Region` role
4. In the role properties, select the **Sales** table → find the `Unit Cost` column → set **OLS Permission** to **None** — this hides the column from users in this role
5. To hide an entire table: select the **UserMapping** table in the role and set its table permission to **None**
6. Save and close Tabular Editor — changes apply immediately to the open model in Desktop
7. Test with **Modeling → View as → North Region** — the `Unit Cost` column and **UserMapping** table should no longer appear in the field list
8. Discuss the key difference: **RLS hides rows** (data is filtered but columns remain visible); **OLS hides structure** (the field name itself is invisible regardless of filter context)

**Key Learning Points:**
- Static RLS: simple and explicit but requires manual role assignment per user in the Service
- Dynamic RLS with `USERPRINCIPALNAME()`: scales to any number of users with a single role
- RLS is enforced at the **dataset** level — it applies regardless of the report or visual used to access the data
- OLS hides tables or columns from a role entirely — even the field name is invisible in the field list
- RLS and OLS are complementary: use OLS to restrict sensitive columns, RLS to filter sensitive rows
- Always test both RLS and OLS before publishing; untested security can expose or incorrectly restrict data

> **References:**
> - [Row-level security in Power BI Desktop](https://learn.microsoft.com/power-bi/enterprise/service-admin-rls)
> - [Dynamic row-level security with USERPRINCIPALNAME](https://learn.microsoft.com/power-bi/guidance/rls-guidance)

---

## — Break (10 minutes) —

---

## Segment 5: Lab C — Deployment Pipelines & Report Governance (25 minutes)

> **License requirement:** Deployment Pipelines require workspaces assigned to a **Fabric capacity (any F SKU, including F2)** or a PPU workspace. If neither is available, this segment is trainer-led demo. Participants can replicate steps in their own environment after the session.

**Business Context:** Without a structured release process, Power BI developers often edit live production reports directly, risking breaking changes for end users. Deployment pipelines provide a formal Dev → Test → Prod promotion process, ensuring reports are validated before reaching consumers.

**Tool Used:** Power BI Service (PPU or Premium capacity required for pipelines; Pro required for endorsement steps)

**Part 1 — Deployment Pipelines (Trainer demo or hands-on with PPU):**
1. In the Power BI Service, navigate to **Deployment pipelines** and click **Create a pipeline**
2. Name the pipeline (e.g., `Sales Report Pipeline`) and assign three stages: **Development**, **Test**, **Production**
3. Assign an existing workspace to the **Development** stage (this is where active development happens)
4. Create two additional workspaces for **Test** and **Production** and assign them to their respective stages
5. In the Development stage, click **Deploy to Test** — review the comparison view showing what has changed
6. Review the **Compare** button between stages — this shows differences in reports, datasets, and dashboards between stages
7. Deploy from Test to Production after review
8. Demonstrate how a dataset connection rule can be set per stage (e.g., point Production to a Production data source, not Development)

**Part 2 — Report Endorsement & Governance (hands-on with Pro):**
1. In the Power BI Service, navigate to a published dataset in a shared workspace
2. Click **More options (…) → Settings → Endorsement**
3. Apply **Promoted** status to the dataset — discuss what this signals to report consumers
4. Discuss **Certified** status — this requires a tenant admin to enable and designate certifiers; it signals organizational trust and quality assurance
5. Apply a **sensitivity label** (e.g., Internal) to the dataset — observe that the label propagates downstream to reports built on it
6. Navigate to **Settings → Lineage view** for the workspace — explore upstream data sources and downstream reports built on shared datasets

**Key Learning Points:**
- Deployment pipelines enforce a structured release process and prevent direct edits to production
- Stage-specific connection rules decouple the dataset source per environment
- Endorsement (Promoted vs. Certified) is a governance signal — consumers can filter for certified content in the data hub
- Sensitivity labels inherit downstream — labelling a dataset propagates to all reports built on it

> **References:**
> - [Introduction to deployment pipelines](https://learn.microsoft.com/power-bi/developer/projects/projects-git)
> - [Get started with deployment pipelines](https://learn.microsoft.com/power-bi/deployment-pipelines/get-started-with-deployment-pipelines)
> - [Endorse your content in Power BI](https://learn.microsoft.com/power-bi/collaborate-share/service-endorse-content)

---

## Segment 7: Lab E — AI Visuals & Copilot Capabilities (25 minutes)

> **License note:** Built-in AI visuals (anomaly detection, forecasting, smart narrative, Q&A) are available in Power BI Desktop and Service to all participants. Copilot requires **any Fabric F SKU (F2+) or PPU** — trainer demo if unavailable.

**Business Context:** Power BI's built-in AI capabilities allow report consumers to discover insights they might otherwise miss — detecting unexpected spikes, forecasting future trends, and asking questions in plain English. Copilot further accelerates report creation and exploration by generating summaries and visuals from natural language prompts.

**Tool Used:** Power BI Desktop + Power BI Service

**Part 1 — AI Visuals in Power BI Desktop (hands-on):**
1. Open `AdvancedSalesReport.pbix` and navigate to a page with a **Line Chart** showing Sales Amount over time
2. With the line chart selected, go to **Add further analysis to your visual** (the analytics pane, magnifying glass icon) and enable **Anomaly detection** — adjust sensitivity and observe flagged anomalies
3. In the same pane, enable **Forecast** — set a forecast length of 3 months and observe the projected trend and confidence interval
4. Add a **Smart Narrative** visual (from the Visualizations pane) — Power BI auto-generates a plain-English text summary of the report page; observe how it updates as slicers are applied
5. Add a **Q&A** visual — type natural language questions such as "total sales by region as a bar chart" and observe the auto-generated visual response

**Part 2 — Copilot in Power BI Service (trainer demo or hands-on with Fabric/Premium capacity):**
1. Open the published report in a **Fabric or Premium capacity** workspace
2. Click the **Copilot** button in the report toolbar
3. Ask Copilot to: *"Summarise this report page"* — review the generated narrative summary
4. Ask Copilot: *"What are the top-performing regions?"* — observe the AI-driven insight extraction
5. In Power BI Desktop with Copilot enabled, ask Copilot to *"Create a report page showing sales trends by category"* — observe the auto-generated page layout
6. Discuss responsible use: Copilot insights should be validated before being acted on; understand what data is sent to the AI service

**Key Learning Points:**
- Anomaly detection and forecasting are built-in, no additional license required — they run on the existing data model
- Smart Narrative and Q&A lower the barrier for non-technical consumers to explore data independently
- Copilot accelerates report creation but requires review — AI-generated insights can reflect biases or errors in the underlying data
- Understand the data processing boundary: Copilot in Power BI sends query results (not raw data) to Azure OpenAI; tenant admins control Copilot enablement

> **References:**
> - [Apply insights to explain fluctuations in visuals](https://learn.microsoft.com/power-bi/create-reports/desktop-insights)
> - [Overview of Copilot for Power BI](https://learn.microsoft.com/power-bi/create-reports/copilot-introduction)
> - [AI visuals in Power BI](https://learn.microsoft.com/power-bi/visuals/power-bi-visualization-anomaly-detection)

---

## Segment 8: Wrap-Up, Q&A & Next Steps (10 minutes)

- Recap of what was covered: performance tuning, RLS, deployment pipelines, and AI capabilities
- Encourage participants to: audit one existing report for RLS coverage and one measure for performance issues
- Discuss the path to Power BI administration — tenant settings, capacity management, activity logs
- Next steps and continued learning

### Recommended Learning Resources
- [Power BI implementation planning guide](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-introduction)
- [Power BI guidance documentation](https://learn.microsoft.com/power-bi/guidance/)
- [DAX function reference](https://learn.microsoft.com/dax/dax-function-reference)
- [Power BI community](https://community.powerbi.com/)
- [Microsoft Fabric documentation](https://learn.microsoft.com/fabric/)
- [Power BI deployment pipelines best practices](https://learn.microsoft.com/power-bi/deployment-pipelines/deployment-pipelines-best-practices)

---

## Trainer Preparation Checklist

- [ ] All participants have a **Power BI Pro** license and can sign in to [app.powerbi.com](https://app.powerbi.com)
- [ ] All participants have **Power BI Desktop** installed ([aka.ms/pbidesktopstore](https://aka.ms/pbidesktopstore))
- [ ] **Lab A & B:** A pre-built `AdvancedSalesReport.pbix` file is prepared and distributed with:
  - A Sales fact table (columns: Date, Region, Product, Sales Amount, Units Sold) with at least 500 rows
  - An intentionally inefficient measure using `FILTER` on the full Sales table (for Lab A)
  - A UserMapping table with at least 4 user emails mapped to regions (for dynamic RLS in Lab B)
  - A line chart visual showing Sales Amount over time with sufficient data points for anomaly detection and forecasting (for Lab D)
- [ ] **Lab B:** Test user email addresses for the UserMapping table are confirmed; participants know which email maps to which region
- [ ] **Lab B (Service):** At least one shared workspace is available for participants to publish to
- [ ] **Lab C — Pipelines:** Confirm whether participants have PPU or Premium Per User; if not, trainer has a PPU account ready for demo
- [ ] **Lab C — Pipelines:** Three workspaces (Dev, Test, Prod) are pre-created in the trainer's PPU tenant if doing trainer demo
- [ ] **Lab C — Endorsement:** Tenant admin has enabled the Certified endorsement setting and designated certifiers
- [ ] **Lab D — Copilot:** Confirm whether a Fabric or Premium capacity workspace is available; if not, trainer has Copilot demo ready
- [ ] **Lab D — Copilot:** Copilot is enabled in the tenant admin settings (**Admin portal → Tenant settings → Copilot**)
- [ ] DLP and sensitivity label policies reviewed — confirm labels are available in the tenant for Lab C governance steps
- [ ] **Lab B (OLS):** **Tabular Editor 2** installed on all participant machines before the session ([tabulareditor.com](https://tabulareditor.com) — free download)
- [ ] **Lab D:** The `AdvancedSalesReport` semantic model from Lab B is already published to the shared workspace before Lab D begins
- [ ] **Lab D:** Participants have at least **Contributor** role in the shared workspace to use Get data → Power BI semantic models
- [ ] **Lab D:** Confirm the shared workspace is NOT a My Workspace — cross-workspace semantic model connections require a shared Pro workspace
- [ ] **Lab B (OLS):** **Tabular Editor 2** installed on all participant machines before the session ([tabulareditor.com](https://tabulareditor.com) — free download)
- [ ] **Lab D:** The `AdvancedSalesReport` semantic model from Lab B is already published to the shared workspace before Lab D begins
- [ ] **Lab D:** Participants have at least **Contributor** role in the shared workspace to use Get data → Power BI semantic models
- [ ] **Lab D:** Confirm the shared workspace is NOT a My Workspace — cross-workspace semantic model connections require a shared Pro workspace
