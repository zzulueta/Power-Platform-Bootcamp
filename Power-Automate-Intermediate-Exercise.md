---
  title: 'Lab Power Automate Intermediate: Building Cloud Flows'
  module: 'Power Automate Intermediate Lab'
---

# Lab Power Automate Intermediate – Building Cloud Flows

## Lab Introduction

In this hands-on lab, you will learn the fundamentals of Power Automate by building four practical cloud flows that automate everyday business processes across Microsoft 365. You will work in the Power Automate web portal to design automated, instant, and approval-based flows that connect Outlook, OneDrive, Microsoft Forms, Approvals, Excel, SharePoint, Microsoft Lists, and Microsoft Teams.

This document contains detailed step-by-step instructions for each of the four labs.

This lab will guide you through the following:

- Build **Lab A** – automatically save Outlook email attachments to OneDrive
- Build **Lab B** – route a Microsoft Forms submission through an approval and record the result in a Microsoft List
- Build **Lab C** – manually trigger a bulk upload of Excel data into a Microsoft List
- Build **Lab D** – notify a Teams channel when a new file is created in SharePoint

## Pre-requisites

- **Power Automate access**: A work or school account with a license that grants access to Power Automate and standard connectors (included with most Microsoft 365 / Office 365 plans). Sign in at [https://make.powerautomate.com](https://make.powerautomate.com).
- **Microsoft 365 apps**: Access to Outlook, OneDrive for Business, SharePoint Online, Microsoft Teams, Microsoft Forms, and Excel Online (Business).
- **A SharePoint site / Microsoft List**: Permission to create lists and libraries on a SharePoint site (or use a team site you own).
- **A Microsoft Team and channel**: Membership in at least one team where you can post messages.
- **Dataverse (for approvals)**: The Approvals action provisions a Dataverse database automatically in the default environment. If you use a non-default environment, an admin role may be required the first time an approval flow runs. See [Get started with approvals](https://learn.microsoft.com/power-automate/get-started-approvals).
- **Sample files**: A few small files (for example a `.txt`, `.docx`, or `.pdf`) to email as attachments, and a small Excel workbook formatted as a table for Lab C.

> **Tip:** All four labs use only **standard connectors**, so no premium license is required. If your tenant has strict Data Loss Prevention (DLP) policies, confirm that Outlook, OneDrive, SharePoint, Teams, Forms, Excel, and Approvals are all in the same connector group.

## Estimated Timing: 125 minutes

## Agenda

| Lab | Topic | Duration |
|---|---|---|
| A | Save Outlook Email Attachments to OneDrive | 25 min |
| B | Forms Submission → Approval + Microsoft List | 30 min |
| — | Break | 15 min |
| C | Manual Trigger — Bulk Upload Excel Data to Microsoft List | 25 min |
| D | New SharePoint File → Teams Notification | 30 min |

## Lab Scenario

Your organization is drowning in repetitive manual tasks: people forward email attachments into shared folders by hand, print and chase approval forms, copy spreadsheet rows into tracking lists one by one, and miss important document uploads because nobody was watching the library. Your task is to replace these manual chores with reliable, low-code automation using Power Automate cloud flows.

By the end of the day you will have built:

- An **automated flow** that files incoming Outlook attachments into OneDrive the moment they arrive.
- An **approval flow** that turns a Microsoft Forms request into a tracked, auditable decision stored in a Microsoft List.
- An **instant (manual) flow** that a person runs on demand to bulk-load an Excel table into a Microsoft List.
- An **event-driven flow** that keeps a Teams channel informed whenever new content lands in a SharePoint library.

## Architecture Overview

```
Power Automate Lab Architecture
===============================

Triggers (events that start a flow)
├── Automated:  When a new email arrives (V3)            → Lab A
├── Automated:  When a new response is submitted (Forms) → Lab B
├── Instant:    Manually trigger a flow                  → Lab C
└── Automated:  When a file is created (SharePoint)      → Lab D

                    ↓ (Power Automate cloud flow engine)

Flows & Connectors
├── Lab A — Save Outlook Attachments to OneDrive
│   ├── Office 365 Outlook  (trigger + Get attachments)
│   ├── Control: Apply to each (attachments)
│   └── OneDrive for Business (Create file)
│
├── Lab B — Forms → Approval → Microsoft List
│   ├── Microsoft Forms (trigger + Get response details)
│   ├── Approvals (Start and wait for an approval)
│   ├── Control: Condition (Outcome = Approve/Reject)
│   ├── SharePoint / Microsoft List (Create item)
│   └── Office 365 Outlook (Send an email notification)
│
├── Lab C — Bulk Upload Excel to Microsoft List
│   ├── Manual trigger (Instant cloud flow)
│   ├── Excel Online (Business) (List rows present in a table)
│   ├── Control: Apply to each (rows)
│   └── SharePoint / Microsoft List (Create item)
│
└── Lab D — New SharePoint File → Teams Notification
    ├── SharePoint (When a file is created - properties only)
    └── Microsoft Teams (Post message / adaptive card in a channel)

                    ↓ (Run history, monitoring, sharing)

Governance & Responsible Use (applies to every flow)
├── Naming conventions & descriptions
├── Error handling (Scope + Configure run after / Try-Catch)
├── Least-privilege connections & service accounts
├── DLP policies (connector data groups)
└── Run-history monitoring & analytics
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following Power Automate skills:

- Lab A – automated flow with triggers, loops, and file creation (25 minutes)
- Lab B – approvals, conditions, and writing to a Microsoft List (30 minutes)
- Lab C – instant flows, reading Excel tables, and bulk item creation (25 minutes)
- Lab D – event-driven flows and Teams notifications (30 minutes)

---

## Lab A — Save Outlook Email Attachments to OneDrive (25 minutes)

**Goal:** Build an automated cloud flow that watches your inbox and, whenever an email with one or more attachments arrives, saves each attachment to a folder in OneDrive for Business.

> **Pattern learned:** Automated trigger → loop over a collection → create a file. This is one of the most common automation shapes in Microsoft 365.

### Prepare a destination folder

1. Open [OneDrive for Business](https://www.office.com/launch/onedrive) in a browser.
2. Create a new folder named **Email Attachments**. This is where the flow will save files.

### Create the automated cloud flow

1. Sign in to [https://make.powerautomate.com](https://make.powerautomate.com).

2. Confirm the correct environment is selected in the top-right environment picker.

3. In the left navigation, select **Create**.

4. Select **Automated cloud flow**.

5. In the **Flow name** field, enter **Outlook Attachments to OneDrive**.

6. In the **Choose your flow's trigger** search box, type **new email**.

7. Select **When a new email arrives (V3) - Office 365 Outlook**, then select **Create**.

   > If prompted, sign in to create the **Office 365 Outlook** connection using your work account.

### Configure the trigger

1. Select the **When a new email arrives (V3)** trigger to open its settings.

2. Select **Show all** under Advanced parameters and set:

   | Parameter | Value |
   | --- | --- |
   | **Include Attachments** | Yes |
   | **Subject Filter** | Invoice |
   | **Only with Attachments** | Yes |
   | **Folder** | Inbox |

   > **Why these settings?** *Include Attachments = Yes* makes the attachment content available to later actions. *Subject Filter = Invoice* ensures only emails with "Invoice" in the subject are processed. *Only with Attachments = Yes* means the flow won't even start for emails that have nothing to save—an efficient trigger-level filter. *Folder = Inbox* ensures the flow watches your main inbox, not subfolders.

3. Ensure that your account has a connection. If not, create a new connection using your work account.

### Loop over the attachments and create files

1. Below the trigger, select the plus sign (**+**), then **Add an action**.

2. Search for **Apply to each** and select the **Control** action **Apply to each**.

3. In the **Select an output from previous steps** box, insert the dynamic content **Attachments** (from the trigger).

   > The trigger returns a *collection* of attachments. `Apply to each` runs the actions inside it once per attachment.

4. Inside the **Apply to each** loop, select **Add an action**.

5. Search for **Create file** and select **Create file - OneDrive for Business**.

   > If prompted, create the **OneDrive for Business** connection.

6. Configure the **Create file** action:

   | Parameter | Value |
   | --- | --- |
   | **Folder Path** | Select the folder icon and choose **/Email Attachments** |
   | **File Name** | Attachment name (see note below) |
   | **File Content** | Attachment content (see note below) |

   > - **File Name** — switch the field to the expression editor (the **fx** tab) and enter:
   >   ```
   >   items('Apply_to_each')?['name']
   >   ```
   > - **File Content** — in the expression editor, enter:
   >   ```
   >   items('Apply_to_each')?['contentBytes']
   >   ```
   >
   > Replace `Apply_to_each` with your loop's actual name if you renamed it (spaces become underscores). The V3 trigger exposes each attachment's properties as `name` and `contentBytes`.

7. Rename the action for clarity: select the action's name and name it **Save attachment to OneDrive**.

### Save and test the flow

1. Select **Save** in the top-right.

2. Select **Test** → **Manually** → **Test**.

3. From another mailbox (or the same account), send yourself an email **with files attached** to the mailbox the flow monitors. Ensure that the subject contains the word "Invoice" so it passes the filter.

4. Return to Power Automate. Within a minute the run should appear. Select the run to inspect each step's inputs and outputs.

5. Open OneDrive and confirm the attachment now appears in the **Email Attachments** folder.

6. Send yourself an email with a different subject (not containing "Invoice") and confirm the flow does not run.

> **Troubleshooting:** If the flow runs but creates no file, open the run and expand **Apply to each** to confirm attachments were returned. If the file name collides with an existing file, OneDrive overwrites the existing file; consider prefixing the name with a timestamp using an expression like `concat(formatDateTime(utcNow(),'yyyyMMdd-HHmmss'),'-',items('Apply_to_each')?['Name'])`.

> *References: [Office 365 Outlook connector](https://learn.microsoft.com/connectors/office365/) · [OneDrive for Business connector](https://learn.microsoft.com/connectors/onedriveforbusiness/)*

> **Congratulations!** You've built an event-driven flow that files attachments automatically. Next, add human decision-making with an approval.

---

## Segment 4: Lab B — Forms Submission → Approval + Microsoft List (30 minutes)

**Goal:** When someone submits a Microsoft Form (for example a purchase or time-off request), route it to an approver. Record every decision—approved or rejected—as an item in a Microsoft List, and email the requester the outcome.

> **Pattern learned:** Automated trigger → enrich data → **human-in-the-loop approval** → branch on the result → write to a list of record. This is the backbone of most business-process automation.

### Step 1: Create the Microsoft Form

1. Go to [https://forms.office.com](https://forms.office.com) and select **New Form**.

2. Title it **Purchase Request**.

3. Add the following questions:

   | Question | Type |
   | --- | --- |
   | Item requested | Text |
   | Estimated cost | Text |
   | Business justification | Text (long answer) |

   > **Note:** Microsoft Forms has no dedicated **Number** question type. For **Estimated cost**, add a **Text** question, then select **... (More settings) → Restrictions → Number** to force respondents to enter a numeric value. Even with this restriction, the response is returned to the flow as **text** (a string), which matters in Step 7.

4. Select **Collect responses** and ensure it is set so people **in your organization can respond**. 

5. Copy the link to a new browser. Submit one test response yourself so real data exists.

### Step 2: Create the Microsoft List

1. Go to your SharePoint site (or [https://lists.live.com](https://www.microsoft.com/microsoft-365/microsoft-lists) via **Microsoft Lists**) 

2. Select **Create new** and create a new **blank list** named **Purchase Requests**.

3. Add the following columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for **Item requested** |
   | EstimatedCost | Number |
   | Justification | Multiple lines of text |
   | Requester | Single line of text |
   | Decision | Choice (Approved, Rejected) |
   | DecisionComments | Multiple lines of text |

### Step 3: Build the approval flow

1. In [Power Automate](https://make.powerautomate.com), select **Create** → **Automated cloud flow**.

2. Name it **Purchase Request Approval**.

3. Search the trigger **When a new response is submitted - Microsoft Forms** and select it, then **Create**.

4. In the trigger, set **Form Id** to **Purchase Request**.

### Step 4: Get the response details

1. Add an action: search **Get response details** and select **Get response details - Microsoft Forms**.

2. Set **Form Id** to **Purchase Request**.

3. Set **Response Id** to the dynamic content **Response Id** from the trigger.

### Step 5: Start and wait for an approval

1. Add an action: search **Start and wait for an approval** and select it (from the **Standard approvals** connector).

   > If this is the first approval flow in your environment, Power Automate provisions a Dataverse database automatically. See [Get started with approvals](https://learn.microsoft.com/power-automate/get-started-approvals).

2. Configure the approval:

   | Parameter | Value |
   | --- | --- |
   | **Approval type** | Approve/Reject - First to respond |
   | **Title** | `Purchase approval: ` then insert **Item requested** dynamic content |
   | **Assigned to** | Enter your manager's (or a colleague's) email address; for testing you may use your own |
   | **Details** | Include cost and justification, e.g. `Cost: ` + **Estimated cost** + ` — ` + **Business justification** |

3. Rename the action **Approval - Purchase request**.

### Step 6: Branch on the approval outcome

1. Add a **Condition** control below the approval.

2. Configure the condition:
   - Left value: insert the approval's **Outcome** dynamic content.
   - Operator: **is equal to**
   - Right value: `Approve`

   > The **Outcome** value for the Approve/Reject types is the string `Approve` or `Reject`.

### Step 7: Record the decision in the Microsoft List

Do this in **both** branches so every request is logged.

1. In the **True** branch, add **Create item - SharePoint**.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | Select the site hosting **Purchase Requests** (see note below) |
   | **List Name** | Purchase Requests |
   | **Title** | Insert **Item requested** |
   | **EstimatedCost** | Convert the text response to a number (see note below) |
   | **Justification** | Insert **Business justification** |
   | **Requester** | Insert **Responders' Email** (from Get response details) |
   | **Decision** | `Approved` |
   | **DecisionComments** | First approver's comments (see note below) |

   > **Site Address:** select **Enter custom value** and paste the URL of the SharePoint site hosting your list, then select the list from the **List Name** dropdown. 
   >
   >**Number conversion:** Because **Estimated cost** arrives as text but **EstimatedCost** is a **Number** column, insert the expression in the **fx** editor:
   >
   > ```
   > float(outputs('Get_response_details')?['body/<Estimated cost field id>'])
   > ```
   >
   > Without the conversion the value may fail to save or be stored incorrectly.
   >
   > **DecisionComments:** Use the **fx** editor:
   >
   > ```
   > first(outputs('Start_and_wait_for_an_approval')?['body/responses'])?['comments']
   > ```
   >
   > Replace `Approval_-_Purchase_request` with your approval action's actual name if you renamed it differently (spaces become underscores).

3. In the **False** branch, add another **Create item - SharePoint** with the same mapping, but set **Decision** to `Rejected`.

### Step 8: Email the requester the outcome

1. In each branch, after **Create item**, add **Send an email (V2) - Office 365 Outlook**.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **To** | Enable Use dynamic content then insert **Responders' Email** |
   | **Subject** | `Your purchase request was ` + `Approved` (or `Rejected` in the No branch) |
   | **Body** | Include the item requested and the approver comment by adding the decision comments expression above |

### Step 9: Save and test

1. Select **Save**.

2. Submit a new **Purchase Request** form response.

3. Respond to the approval request from the Outlook email, the Teams adaptive card, or the Power Automate **Action center** (**Approvals** in the left nav).

4. Confirm a new row appears in the **Purchase Requests** list with the correct **Decision**, and that the requester receives the outcome email.

> **Troubleshooting:** If the approval never arrives, verify the **Assigned to** address is a valid user in your tenant. If list creation fails, check that column internal names match the fields you mapped (open the list column settings to confirm internal names).

> *Reference: [Get started with approvals](https://learn.microsoft.com/power-automate/get-started-approvals)*

> **Congratulations!** You built a human-in-the-loop approval that produces an auditable record.

---

## Break (15 minutes)

Stretch, refill coffee, and reset. When you return, we move from event-driven flows to a **manually triggered** flow and a **Teams notification** flow.

---

## Lab C — Manual Trigger: Bulk Upload Excel Data to Microsoft List (25 minutes)

**Goal:** Build an **instant cloud flow** that a person runs on demand. When triggered, it reads every row from an Excel table and creates a corresponding item in a Microsoft List—turning a spreadsheet into structured, shareable data in one click.

> **Pattern learned:** Manual trigger → read a data set → loop → bulk-create items. Ideal for one-off migrations and periodic data loads.

### Step 1: Prepare the Excel workbook

1. Create a new Excel workbook and save it to **OneDrive for Business** or a SharePoint document library (the file must be in the cloud, not local, for the connector to read it).

2. Add these column headers in row 1: **ProductName**, **Category**, **UnitPrice**, **InStock**.

3. Enter 5–10 rows of sample data.

4. Select the data range including headers, then on the **Insert** ribbon choose **Table**. Confirm **My table has headers**.

5. With the table selected, on the **Table Design** ribbon set the **Table Name** to **Products**.

   > **Why a named table?** The Excel Online (Business) connector reads **tables**, not raw sheets. Naming the table makes it selectable in the flow and guarantees a stable structure.

6. Save and close the workbook.

### Step 2: Create the destination Microsoft List

1. Create a new **blank list** named **Product Catalog** with these columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for **ProductName** |
   | Category | Single line of text |
   | UnitPrice | Number |
   | InStock | Number |

### Step 3: Build the instant cloud flow

1. In [Power Automate](https://make.powerautomate.com), select **Create** → **Instant cloud flow**.

2. Name it **Bulk Upload Products to List**.

3. Under **Choose how to trigger this flow**, select **Manually trigger a flow**, then **Create**.

### Step 4: Read the Excel table

1. Add an action: search **List rows present in a table** and select **List rows present in a table - Excel Online (Business)**.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **Location** | OneDrive for Business (or the SharePoint site) where you saved the workbook |
   | **Document Library** | OneDrive / the library |
   | **File** | Browse to your workbook |
   | **Table** | Products |

### Step 5: Create a list item per row

1. Add an action: search **Create item** and select **Create item - SharePoint**.

   > Power Automate automatically wraps this action in an **Apply to each** because you're inserting per-row dynamic content. That's expected.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | The site hosting **Product Catalog** |
   | **List Name** | Product Catalog |
   | **Title** | Insert **ProductName** |
   | **Category** | Insert **Category** |
   | **UnitPrice** | Insert **UnitPrice** |
   | **InStock** | Insert **InStock** |

3. Rename the action **Create product item**.

### Step 6: Run and verify

1. Select **Save**, then **Test** → **Manually** → **Test** → **Run flow**.

2. When the flow finishes, open the **Product Catalog** list and confirm one item exists per Excel row.

> **Performance note:** For large tables, this loop can be slow and may hit throttling. In production you would increase (carefully) or leave default the **Apply to each** concurrency, filter rows with the connector's **Filter Query**, or consider a data-flow/import approach. For this lab's small data set, defaults are fine.

> **Troubleshooting:** If **Table** is empty in the dropdown, re-open the workbook and confirm the range is formatted as a named table. If numbers import as text, check the Excel column data types.

> *Reference: [Excel Online (Business) connector](https://learn.microsoft.com/connectors/excelonlinebusiness/)*

> **Congratulations!** You turned a spreadsheet into governed list data with a one-click flow.

---

## Lab D — New SharePoint File → Teams Notification (30 minutes)

**Goal:** Keep a team informed automatically. When a new file is created in a SharePoint document library, post a message (and optionally a rich adaptive card) to a Microsoft Teams channel with the file's name and a link.

> **Pattern learned:** Event-driven trigger on a SharePoint library → post to Teams. A staple of collaboration automation.

### Step 1: Identify your SharePoint library and Teams channel

1. Choose (or create) a SharePoint document library, for example **Shared Documents** on a team site.

2. Choose a Microsoft Teams **team** and **channel** where notifications should appear (for example **General** in a "Project" team).

### Step 2: Create the automated cloud flow

1. In [Power Automate](https://make.powerautomate.com), select **Create** → **Automated cloud flow**.

2. Name it **New SharePoint File to Teams**.

3. Search the trigger **When a file is created (properties only) - SharePoint** and select it, then **Create**.

   > **Why "properties only"?** This lightweight trigger fires when a file is added and returns its metadata (name, link, author) without downloading the file content—perfect for a notification.

### Step 3: Configure the trigger

1. In the trigger, set:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | The site containing your library |
   | **Library Name** | Shared Documents (or your chosen library) |

### Step 4: Post a message to Teams

1. Add an action: search **Post message in a chat or channel** and select **Post message in a chat or channel - Microsoft Teams**.

   > If prompted, create the **Microsoft Teams** connection.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **Post as** | Flow bot (or User) |
   | **Post in** | Channel |
   | **Team** | Your team |
   | **Channel** | Your channel |
   | **Message** | Compose a message that includes dynamic content, for example: `New file added: ` + **File name with extension** + ` — ` + **Link to item** |

3. Rename the action **Notify Teams channel**.

### Step 5: (Optional) Post a richer adaptive card

For a more polished notification:

1. Instead of (or in addition to) the plain message, add **Post adaptive card in a chat or channel - Microsoft Teams**.

2. Paste a simple adaptive card JSON and bind fields to dynamic content, for example the file name and link. An adaptive card lets you present the file name, author, and a clickable **Open file** action button in a formatted card.

   > *Reference: adaptive card designer at [https://adaptivecards.io/designer](https://adaptivecards.io/designer).*

### Step 6: Save and test

1. Select **Save**.

2. Upload a new file to the monitored SharePoint library.

3. Within a minute, confirm the notification appears in the chosen Teams channel with the correct file name and a working link.

> **Troubleshooting:** The properties-only trigger can take a short time to fire and, by design, may not trigger for files created by the flow's own connection in some retention scenarios—upload manually to test. If the link doesn't open, verify recipients have permission to the library.

> *References: [SharePoint connector](https://learn.microsoft.com/connectors/sharepointonline/) · [Microsoft Teams connector](https://learn.microsoft.com/connectors/teams/)*

> **Congratulations!** You closed the loop between document management and team communication—no manual pinging required.


