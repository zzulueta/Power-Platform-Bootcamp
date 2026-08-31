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
- Build **Lab D** – notify a Teams channel when a new file is created in OneDrive

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
| D | New OneDrive File → Teams Notification | 30 min |

## Lab Scenario

Your organization is drowning in repetitive manual tasks: people forward email attachments into shared folders by hand, print and chase approval forms, copy spreadsheet rows into tracking lists one by one, and miss important document uploads because nobody was watching the library. Your task is to replace these manual chores with reliable, low-code automation using Power Automate cloud flows.

By the end of the day you will have built:

- An **automated flow** that files incoming Outlook attachments into OneDrive the moment they arrive.
- An **approval flow** that turns a Microsoft Forms request into a tracked, auditable decision stored in a Microsoft List.
- An **instant (manual) flow** that a person runs on demand to bulk-load an Excel table into a Microsoft List.
- An **event-driven flow** that keeps a Teams channel informed whenever new content lands in a OneDrive folder.

## Architecture Overview

```
Power Automate Lab Architecture
===============================

Triggers (events that start a flow)
├── Automated:  When a new email arrives (V3)            → Lab A
├── Automated:  When a new response is submitted (Forms) → Lab B
├── Instant:    Manually trigger a flow                  → Lab C
└── Automated:  When a file is created (OneDrive)        → Lab D

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
│   ├── Control: For each (rows)
│   └── SharePoint / Microsoft List (Create item)
│
└── Lab D — New OneDrive File → Teams Notification
    ├── OneDrive for Business (When a file is created - properties only)
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

2. Confirm the correct environment is selected in the top-right environment picker and that you are in the New design experience.

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
   | Estimated cost | Text (see note below) |
   | Business justification | Text (long answer) |

   > **Note:** Microsoft Forms has no dedicated **Number** question type. For **Estimated cost**, add a **Text** question, then select **... (More settings) → Restrictions → Number** to force respondents to enter a numeric value.

4. Select **Collect responses** and ensure it is set so people **in your organization can respond**. 

5. Copy the link to a new browser. Submit one test response yourself so real data exists.

### Step 2: Create the Microsoft List

1. Go to your SharePoint site (or [https://lists.live.com](https://www.microsoft.com/microsoft-365/microsoft-lists) via **Microsoft Lists**) 

2. Select **Create new** and create a new **blank list** named **Purchase Requests**.

3. Add the following columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for **Item requested** |
   | EstimatedCost | Single line of text |
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
   | **EstimatedCost** | Insert **Estimated cost** |
   | **Justification** | Insert **Business justification** |
   | **Requester** | Insert **Responders' Email** (from Get response details) |
   | **Decision** | `Approved` |
   | **DecisionComments** | First approver's comments (see note below) |

   > **Site Address:** select **Enter custom value** and paste the URL of the SharePoint site hosting your list, then select the list from the **List Name** dropdown. 
   >
   > **DecisionComments:** Use the **fx** editor:
   >
   > ```
   > first(outputs('Start_and_wait_for_an_approval')?['body/responses'])?['comments']
   > ```

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

### Step 1: Upload the provided Excel workbook

1. Download the provided **Products.xlsx** workbook from the lab materials.

2. Upload it to **OneDrive for Business** (or a SharePoint document library). The file must live in the cloud — the Excel Online (Business) connector cannot read files stored only on your local machine.

3. Open the file and confirm the data is already formatted as a named table called **Products** with the following columns:

   | Column | Purpose |
   | --- | --- |
   | **ProductID** | Unique key for each row (used later to update the row) |
   | **ProductName** | Product name |
   | **Category** | Product category |
   | **UnitPrice** | Price per unit |
   | **InStock** | Units in stock |
   | **Uploaded** | Tracks whether the row has already been added to the list. Every row starts as **No** |

   > **Why the Uploaded column?** It lets the flow process only new rows and mark each one **Yes** after it's added, so re-running the flow never creates duplicates. This is the *idempotent* pattern used for reliable data loads.

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

3. Expand **Advanced parameters** and set **Filter Query** to:

   ```
   Uploaded eq 'No'
   ```

   > This returns only the rows that haven't been uploaded yet, so the flow skips any row already marked **Yes** — the key to making re-runs safe.

### Step 5: Create a list item per row

1. Add an action: search **Create item** and select **Create item - SharePoint**.

2. Configure:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | The site hosting **Product Catalog** |
   | **List Name** | Product Catalog |
   | **Title** | Insert **ProductName** |
   | **Category** | Insert **Category** |
   | **UnitPrice** | Convert to a number (see note below) |
   | **InStock** | Convert to a number (see note below) |

   > **Number conversion:** **UnitPrice** and **InStock** are **Number** columns, but the Excel connector returns cell values as text, so the plain dynamic content can't be dropped into a number field. In each field, open the **fx** editor and wrap the value in a conversion function — `float()` for the decimal price and `int()` for the whole-number stock:
   >
   > ```
   > float(items('For_each')?['UnitPrice'])
   > ```
   >
   > ```
   > int(items('For_each')?['InStock'])
   > ```
   >
   > Power Automate automatically wraps this action in a **For each** because you're inserting per-row dynamic content. That's expected.

3. Rename the action **Create product item**.

### Step 6: Mark the row as uploaded

1. Inside the same **For each**, directly after **Create item**, add an action: search **Update a row** and select **Update a row - Excel Online (Business)**.

2. Point it at the same workbook and table, then set the key and flag:

   | Parameter | Value |
   | --- | --- |
   | **Location** | Same as Step 4 |
   | **Document Library** | Same as Step 4 |
   | **File** | Same as Step 4 (Products.xlsx) |
   | **Table** | Products |
   | **Key Column** | ProductID |
   | **Key Value** | Insert **ProductID** (current row) |
   | **Uploaded** | Yes |

   > Marking the flag *inside* the loop, immediately after the item is created, makes the flow **idempotent**: if it runs again, the already-processed rows are filtered out in Step 4 and never duplicated.

3. Rename the action **Mark row as uploaded**.

### Step 7: Run and verify

1. Select **Save**, then **Test** → **Manually** → **Test** → **Run flow**.

2. When the flow finishes, open the **Product Catalog** list and confirm one item exists per Excel row.

3. Open the **Products.xlsx** workbook and confirm every processed row's **Uploaded** column is now **Yes**.

4. Run the flow a second time. Because all rows are already marked **Yes**, the **Filter Query** returns nothing and no duplicate items are created — proving the flow is safe to re-run.

> **Performance note:** For large tables, this loop can be slow and may hit throttling. In production you would increase (carefully) or leave default the **For each** concurrency, filter rows with the connector's **Filter Query**, or consider a data-flow/import approach. For this lab's small data set, defaults are fine.

> **Troubleshooting:** If **Table** is empty in the dropdown, re-open the workbook and confirm the range is formatted as a named table. If numbers import as text, check the Excel column data types.

> *Reference: [Excel Online (Business) connector](https://learn.microsoft.com/connectors/excelonlinebusiness/)*

> **Congratulations!** You turned a spreadsheet into governed list data with a one-click flow.

---

## Lab D — New OneDrive File → Teams Notification (30 minutes)

**Goal:** Keep a team informed automatically. When a new file is added to a OneDrive for Business folder, post a message (and optionally a rich adaptive card) to a Microsoft Teams channel with the file's name and location.

> **Pattern learned:** Event-driven trigger on a OneDrive folder → post to Teams. A staple of collaboration automation.

### Step 1: Prepare your OneDrive folder and Teams channel

1. In [OneDrive for Business](https://www.office.com/launch/onedrive), create a folder named **Uploads** (or reuse an existing one). The flow will watch this folder for new files.

2. Choose a Microsoft Teams **team** and **channel** where notifications should appear (for example **General** in a "Project" team).

### Step 2: Create the automated cloud flow

1. In [Power Automate](https://make.powerautomate.com), select **Create** → **Automated cloud flow**.

2. Name it **New OneDrive File to Teams**.

3. Search the trigger **When a file is created (properties only) - OneDrive for Business** and select it, then **Create**.

   > **Why "properties only"?** This lightweight trigger fires when a file is added and returns its metadata (name, path, size) without downloading the file content—perfect for a notification.

### Step 3: Configure the trigger

1. In the trigger, set:

   | Parameter | Value |
   | --- | --- |
   | **Folder** | Select the folder icon and choose **/Uploads** |

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
   | **Message** | Compose a message with dynamic content, for example: `New file added: ` + **Name** + ` (` + **Path** + `)` |

3. Rename the action **Notify Teams channel**.

### Step 5: (Optional) Post a richer adaptive card

For a more polished notification:

1. Instead of (or in addition to) the plain message, add **Post adaptive card in a chat or channel - Microsoft Teams** and set **Post in**, **Team**, and **Channel** the same way as in Step 4.

2. In the **Adaptive Card** field, paste the JSON below. The `@{...}` expressions pull the file name and path straight from the trigger, so you don't need to insert dynamic content manually:

   ```json
   {
       "type": "AdaptiveCard",
       "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
       "version": "1.4",
       "body": [
           {
               "type": "TextBlock",
               "size": "Medium",
               "weight": "Bolder",
               "text": "New file uploaded to OneDrive",
               "wrap": true
           },
           {
               "type": "FactSet",
               "facts": [
                   {
                       "title": "File name",
                       "value": "@{triggerOutputs()?['body/Name']}"
                   },
                   {
                       "title": "Location",
                       "value": "@{triggerOutputs()?['body/Path']}"
                   },
                   {
                       "title": "Detected (UTC)",
                       "value": "@{formatDateTime(utcNow(),'yyyy-MM-dd HH:mm')}"
                   }
               ]
           }
       ]
   }
   ```

   > **Tip:** Preview or tweak the layout in the [Adaptive Card Designer](https://adaptivecards.io/designer) (choose **Microsoft Teams** as the host app). Teams supports Adaptive Cards up to version 1.5. If your trigger exposes different property names, adjust `body/Name` and `body/Path` to match.

### Step 6: Save and test

1. Select **Save**.

2. Upload a new file to the monitored OneDrive **Uploads** folder.

3. Within a minute, confirm the notification appears in the chosen Teams channel with the correct file name and a working link.

> **Troubleshooting:** The properties-only trigger can take a short time to fire and, by design, may not trigger for files created by the flow's own connection in some scenarios—upload manually to test. If recipients can't open the file, share it with them from OneDrive first.

> *References: [OneDrive for Business connector](https://learn.microsoft.com/connectors/onedriveforbusiness/) · [Microsoft Teams connector](https://learn.microsoft.com/connectors/teams/)*

> **Congratulations!** You closed the loop between document management and team communication—no manual pinging required.

> **Note — making the file a clickable link:** The **properties only** trigger returns the file's **Path** (plain text), not a web URL, so it isn't clickable on its own. To post a working link:
>
> 1. Between the trigger and the Teams action, add **Create share link - OneDrive for Business** and set:
>    - **File**: insert the trigger's **Id** dynamic content
>    - **Link type**: `View` (or `Edit`)
>    - **Link scope**: `Organization` (or `Anonymous`, if your tenant allows it)
> 2. This action outputs a **webUrl**. Use it in the **Post message** action's **Message** field with Markdown link syntax so the file name becomes a hyperlink:
>
>    ```
>    New file added: [Name](webUrl)
>    ```
>
>    i.e. insert the **Name** dynamic content as the link text and the **webUrl** (from Create share link) as the target.
> 3. For the adaptive card, add an **Action.OpenUrl** button that points to the same `webUrl`:
>
>    ```json
>    {
>        "type": "ActionSet",
>        "actions": [
>            {
>                "type": "Action.OpenUrl",
>                "title": "Open file",
>                "url": "@{outputs('Create_share_link')?['body/link/webUrl']}"
>            }
>        ]
>    }
>    ```
>
> **Create share link** respects your tenant's sharing policies — if `Anonymous` is blocked, use `Organization` and ensure recipients have access. Avoid hand-building a URL from the path, as it breaks with special characters and tenant differences.


