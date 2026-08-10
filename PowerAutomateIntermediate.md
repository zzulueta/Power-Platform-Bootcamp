# Power Automate Intermediate Training — 3-Hour Session

**Audience:** Users with basic Power Automate awareness  
**License:** Power Automate for Microsoft 365  
**Format:** Short presentation followed by hands-on labs  
**Duration:** 3 Hours  

> **Note:** All lab exercises use only standard M365 connectors available with the Power Automate for Microsoft 365 license (SharePoint, Outlook, Teams, OneDrive, Forms, Planner, Excel Online).

---

## Agenda Overview

| Segment | Topic | Duration |
|---|---|---|
| 1 | Welcome & Recap | 10 min |
| 2 | Presentation: Flow Design, Best Practices & Responsible Use | 35 min |
| 3 | Lab A: Save Outlook Email Attachments to OneDrive | 25 min |
| 4 | Lab B: Forms Submission → Approval + Microsoft List | 30 min |
| — | Break | 10 min |
| 5 | Lab C: Manual Trigger — Bulk Upload Excel Data to Microsoft List | 25 min |
| 6 | Lab D: New SharePoint File → Teams Notification | 20 min |
| 7 | Wrap-Up, Q&A & Next Steps | 5 min |

---

## Segment 1: Welcome & Recap (10 minutes)

- Welcome and housekeeping
- Quick recap of Power Automate awareness session concepts
- Session objectives and what participants will build today
- Overview of the Power Automate for M365 license scope
  - What's included: standard connectors, cloud flows
  - What's NOT included: premium connectors, desktop flows (RPA)

---

## Segment 2: Presentation — Flow Design, Best Practices & Responsible Use (35 minutes)

### Flow Design Concepts (10 minutes)
- Triggers vs. Actions vs. Conditions
- Understanding flow run history and monitoring
- Working with dynamic content and expressions
- Using variables and compose actions effectively
- Error handling: Configure run after, Try-Catch patterns

### Best Practices in Creating Flows (15 minutes)
- **Naming conventions** — use clear, descriptive names for flows, actions, and variables
- **Keep flows focused** — one flow per business process; avoid overloading a single flow
- **Use comments/notes** on actions to document logic for others
- **Avoid hardcoding values** — use variables or flow inputs for reusability
- **Test incrementally** — test each action before building the next step
- **Monitor run history** — regularly review failed runs and set up failure alerts
- **Scope and permissions** — understand who owns and can edit the flow (personal vs. shared)
- **Connection management** — use shared connections carefully; avoid embedding personal credentials
- **Avoid unnecessary triggers** — use specific conditions to prevent excessive flow runs
- **Version control mindset** — save a copy before major edits; use flow descriptions to track changes

### Responsible Use of Power Automate (10 minutes)
- **Data privacy & compliance** — only automate what you are authorized to access
- **Data Loss Prevention (DLP) policies** — understand your organization's DLP rules and which connectors are allowed
- **Principle of least privilege** — flows should only access data they need
- **Transparency** — inform colleagues when flows act on their behalf (e.g., sending emails, posting to Teams)
- **Human oversight** — keep humans in the loop for high-impact or irreversible actions (approvals, deletions)
- **Avoid automation bias** — review flow outputs regularly; don't assume automated results are always correct
- **Shared flow ownership** — document flows and ensure business continuity (avoid single owner dependencies)
- **AI features** — when using Copilot or AI Builder features, understand what data is processed and where

> **Reference:** [Microsoft responsible AI principles](https://learn.microsoft.com/power-platform/responsible-ai-faqs)

---

## Segment 3: Lab A — Save Outlook Email Attachments to OneDrive (25 minutes)

**Business Context:** Sales, Finance, or HR teams often receive critical documents such as purchase orders, invoices, or employee requirements via email. This flow automatically saves email attachments from specific senders or with specific subjects into a designated OneDrive folder, ensuring documents are securely stored, easy to find, and not missed or lost in inboxes.

**M365 Connectors Used:** Outlook, OneDrive for Business

**Steps:**
1. Build an automated flow triggered by "When a new email arrives (V3)" in Outlook
2. Configure the trigger filter: set **From** to a specific sender (e.g., a supplier or key contact) and/or **Has Attachment** to Yes
3. Add an "Apply to each" action looping over the **Attachments** output
4. Inside the loop, add "Create file" in OneDrive for Business, mapping the attachment name and content to a designated folder path
5. Add a condition to skip inline images (filter by file extension or attachment size if needed)
6. Test by sending a test email with an attachment from the configured sender

**Key Learning Points:**
- Filtering email triggers to target specific senders or subjects
- Iterating over email attachments with Apply to each
- Using dynamic content for file name and file content
- Organizing files into OneDrive folder paths with dynamic content

> **Reference:** [Save email attachments to OneDrive](https://learn.microsoft.com/power-automate/email-triggers)

---

## Segment 4: Lab B — Forms Submission → Approval + Microsoft List (30 minutes)

**Business Context:** Organizations commonly use Microsoft Forms to collect requests such as leave applications, purchase requests, or IT support needs. When a response is submitted, this flow saves the data into a Microsoft List, initiates an approval process, and updates the request status — giving managers clear visibility and an auditable approval trail.

**M365 Connectors Used:** Microsoft Forms, SharePoint (Microsoft Lists), Approvals, Outlook

**Steps:**
1. Create a Microsoft Form with fields: Requester Name, Email, Request Type, Description, and Date Needed
2. Create a Microsoft List with matching columns plus a **Status** column (default: Pending)
3. Build an automated flow triggered by "When a new response is submitted" in Forms
4. Add "Get response details" to retrieve the form answers
5. Add "Create item" in SharePoint to save the form data into the Microsoft List
6. Add "Start and wait for an approval" (Approve/Reject – First to respond), addressed to the relevant manager
7. Add a condition on the approval outcome:
   - **If approved:** Update the list item Status to "Approved" and send a confirmation email to the requester
   - **If rejected:** Update the list item Status to "Rejected" and email the requester with the rejection comments
8. Test by submitting a sample form response

**Key Learning Points:**
- Chaining Forms, Lists, and Approvals connectors in one flow
- Saving form responses to a structured Microsoft List
- Using approval outcomes to drive conditional updates
- Building an auditable end-to-end request process

> **References:**
> - [Use Microsoft Forms with Power Automate](https://learn.microsoft.com/connectors/microsoftforms/)
> - [Create an approval flow](https://learn.microsoft.com/power-automate/modern-approvals)

---

## — Break (10 minutes) —

---

## Segment 5: Lab C — Manual Trigger: Bulk Upload Excel Data to Microsoft List (25 minutes)

**Business Context:** Business teams often maintain data in Excel (such as employee records, inventory, or customer lists) and need to upload it into a structured system. This flow allows users to manually trigger a bulk upload of Excel rows into a Microsoft List, eliminating manual data entry and reducing errors while supporting one-time or recurring data migrations.

**M365 Connectors Used:** Manually trigger a flow, Excel Online (Business), SharePoint (Microsoft Lists)

**Steps:**
1. Prepare an Excel workbook in OneDrive or a SharePoint library with a formatted table (e.g., columns: Employee Name, Department, Role, Start Date)
2. Create a matching Microsoft List with the same columns
3. Build a flow triggered by "Manually trigger a flow" (instant cloud flow)
4. Add "List rows present in a table" from Excel Online (Business), pointing to the workbook and table
5. Add "Apply to each" looping over the Excel rows output
6. Inside the loop, add "Create item" in SharePoint (Microsoft Lists), mapping each Excel column to the corresponding list column
7. Add a "Get items" check or a counter variable to show a completion summary (optional)
8. Manually run the flow and verify the data appears in the Microsoft List

**Key Learning Points:**
- Using the manual trigger for on-demand business processes
- Reading structured data from Excel Online (Business) tables
- Bulk-creating list items with Apply to each and dynamic column mapping
- Understanding flow run history to confirm successful uploads

> **References:**
> - [Excel Online (Business) connector](https://learn.microsoft.com/connectors/excelonlinebusiness/)
> - [Trigger a flow manually](https://learn.microsoft.com/power-automate/trigger-flow-manually)

---

## Segment 6: Lab D — New SharePoint File → Teams Notification (20 minutes)

**Business Context:** Teams often collaborate using shared SharePoint folders as drop locations for deliverables, reports, or files. This flow sends an automatic Teams message with a clickable file link whenever a new file is added, instantly notifying the team and improving visibility and collaboration without manual follow-ups.

**M365 Connectors Used:** SharePoint, Microsoft Teams

**Steps:**
1. Identify or create a SharePoint document library to use as the monitored folder (e.g., a "Deliverables" or "Reports" library)
2. Build an automated flow triggered by "When a file is created (properties only)" in SharePoint, targeting that library
3. Add "Get file properties" to retrieve metadata including the file name, created by, and the direct link
4. Add "Post message in a chat or channel" in Microsoft Teams
5. Configure the message body to include: file name, who uploaded it, date/time, and the clickable file link using the **Link to item** dynamic value
6. Test by uploading a new file to the SharePoint library and confirming the Teams message appears

**Key Learning Points:**
- Triggering flows from SharePoint document libraries vs. lists
- Retrieving file metadata with "Get file properties"
- Composing a useful, informative Teams message with dynamic content
- Differentiating between file link types (view link vs. edit link)

> **Reference:** [Send a message in Teams from Power Automate](https://learn.microsoft.com/power-automate/teams/send-a-message-in-teams)

---

## Segment 7: Wrap-Up, Q&A & Next Steps (5 minutes)

- Recap of what was built today
- Discuss flow management: sharing, co-ownership, and turning flows on/off
- Encourage participants to identify one automation opportunity in their own work
- Next steps and continued learning

### Recommended Learning Resources
- [Power Automate learning paths on Microsoft Learn](https://learn.microsoft.com/training/powerplatform/power-automate)
- [Power Automate documentation](https://learn.microsoft.com/power-automate/)
- [Power Automate community](https://powerusers.microsoft.com/t5/Microsoft-Power-Automate/ct-p/MPACommunity)
- [Power Automate templates gallery](https://make.powerautomate.com/templates)
- [DLP policy overview](https://learn.microsoft.com/power-platform/admin/wp-data-loss-prevention)

---

## Trainer Preparation Checklist

- [ ] All participants have access to [make.powerautomate.com](https://make.powerautomate.com)
- [ ] **Lab A:** Identify a test sender email address; participants know the sender to filter on
- [ ] **Lab A:** A OneDrive folder path is agreed upon for saving attachments (e.g., `/Email Attachments/`)
- [ ] **Lab B:** A Microsoft Form is pre-built or participants create their own (leave/purchase/IT request)
- [ ] **Lab B:** A Microsoft List is pre-created with columns matching the Form fields plus a Status column
- [ ] **Lab B:** An approver email address is identified for the approval action
- [ ] **Lab C:** A sample Excel workbook with a formatted table is prepared and stored in OneDrive or SharePoint
- [ ] **Lab C:** A target Microsoft List is pre-created with columns matching the Excel table headers
- [ ] **Lab D:** A SharePoint document library is designated as the monitored folder
- [ ] **Lab D:** A Teams channel is designated to receive the file notifications
- [ ] Participants have been added to the SharePoint site and Teams channel
- [ ] DLP policies reviewed — confirm standard M365 connectors are allowed in your tenant
