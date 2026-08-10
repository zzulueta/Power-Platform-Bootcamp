# Power Automate Advanced Training — 3-Hour Session

**Audience:** Users who have completed the Intermediate session and are comfortable building cloud flows  
**License:** Power Automate Premium (or Power Automate Process) + AI Builder credits required  
**Format:** Short presentation followed by hands-on labs  
**Duration:** 3 Hours  

> **Note:** Desktop flows require the **Power Automate Premium** license and the **Power Automate for Desktop** app installed on the participant's machine. AI Builder features require **AI Builder credits** assigned to the environment. Confirm licensing with your admin before the session.

---

## Agenda Overview

| Segment | Topic | Duration |
|---|---|---|
| 1 | Welcome & Recap | 10 min |
| 2 | Presentation: Desktop Flows, AI Builder & Responsible AI | 35 min |
| 3 | Lab A: Automate a Desktop Application with Power Automate Desktop | 25 min |
| 4 | Lab B: Extract Invoice Data with AI Builder Document Processing | 30 min |
| — | Break | 15 min |
| 5 | Lab C: AI Builder Prompts — Summarise and Classify Text in a Flow | 25 min |
| 6 | Lab D: End-to-End Flow — Email Attachment → AI Extract → Microsoft List | 30 min |
| 7 | Wrap-Up, Q&A & Next Steps | 10 min |

---

## Segment 1: Welcome & Recap (10 minutes)

- Welcome and housekeeping
- Recap of Intermediate session concepts (cloud flows, connectors, approvals)
- Session objectives and what participants will build today
- Licensing overview for this session:
  - **Power Automate Premium** — required for Desktop flows and attended/unattended RPA
  - **AI Builder credits** — required for AI Builder models and prompts
  - Difference between Power Automate for M365 vs. Premium vs. Process

---

## Segment 2: Presentation — Desktop Flows, AI Builder & Responsible AI (35 minutes)

### Desktop Flows & RPA (12 minutes)
- What is Robotic Process Automation (RPA)?
- Power Automate Desktop — attended vs. unattended automation
- How desktop flows differ from cloud flows
- Recording actions vs. scripting actions in the designer
- Triggering desktop flows from cloud flows
- Common use cases: legacy applications, web browsers, desktop apps without APIs
- Limitations and when RPA is the right choice

### AI Builder Overview (13 minutes)
- What is AI Builder?
- Pre-built AI models vs. custom-trained models
  - **Pre-built:** sentiment analysis, language detection, business card reader, invoice processing, receipt processing, text recognition (OCR)
  - **Custom:** document processing, object detection, custom classification
- AI Builder Prompts — using GPT-based prompts directly in flows
  - What are prompts?
  - How to create and test a prompt in AI Builder
  - Passing dynamic inputs into prompts
  - Using prompt output in subsequent flow actions
- AI Builder credits and consumption

### Responsible AI in Automation (10 minutes)
- Microsoft's AI principles: fairness, reliability, privacy, inclusiveness, transparency, accountability
- Human-in-the-loop design — when AI output must be reviewed before acting
- Avoid over-reliance on AI output — validate results against business rules
- Data used to train custom models — know what data you are providing
- Understand AI confidence scores — set thresholds and handle low-confidence results
- Transparency to end users — disclose when AI is used in a process
- Governance — who approves the use of AI Builder models in your organization?

> **References:**
> - [Responsible AI FAQs for Power Platform](https://learn.microsoft.com/power-platform/responsible-ai-faqs)
> - [AI Builder overview](https://learn.microsoft.com/ai-builder/overview)

---

## Segment 3: Lab A — Automate a Desktop Application with Power Automate Desktop (25 minutes)

**Business Context:** Many organizations still rely on legacy desktop applications or repetitive manual tasks in Windows apps (e.g., copying data from a report into another system). Power Automate Desktop enables users to record and automate these interactions without any API, dramatically reducing manual effort and errors.

**License Required:** Power Automate Premium

**Steps:**
1. Open **Power Automate Desktop** on your machine (download from [aka.ms/powerautomatedesktop](https://aka.ms/powerautomatedesktop) if not installed)
2. Create a new desktop flow and name it clearly (e.g., "Copy Notepad Data to Excel")
3. Use the **Recorder** to capture the following actions:
   - Open Notepad
   - Type a sample line of text (simulating data entry)
   - Select all and copy
   - Open Excel and paste into a cell
   - Save the file
4. Stop the recording and review the generated actions in the designer
5. Edit variable names for clarity
6. Run the flow in **attended mode** and observe execution
7. (Optional) Return to **make.powerautomate.com** and create a cloud flow that triggers the desktop flow using "Run a flow built with Power Automate for desktop"

**Key Learning Points:**
- How the Power Automate Desktop recorder captures UI interactions
- Editing and cleaning up recorded actions
- Understanding attended vs. unattended execution mode
- Connecting a desktop flow to a cloud flow trigger

> **Reference:** [Introduction to Power Automate Desktop](https://learn.microsoft.com/power-automate/desktop-flows/introduction)

---

## Segment 4: Lab B — Extract Invoice Data with AI Builder Document Processing (30 minutes)

**Business Context:** Finance teams receive invoices as PDF or image attachments and manually key data (vendor name, invoice number, amounts, dates) into systems. AI Builder's pre-built invoice processing model extracts this data automatically, removing manual entry and accelerating payment processing.

**License Required:** AI Builder credits

**Connectors Used:** Outlook, AI Builder, SharePoint (Microsoft Lists)

**Steps:**
1. In **make.powerautomate.com**, create an automated cloud flow triggered by "When a new email arrives (V3)" with **Has Attachment: Yes** and a subject filter (e.g., subject contains "Invoice")
2. Add "Apply to each" over the email attachments
3. Add the **AI Builder** action: "Extract information from invoices" — pass the attachment content as the invoice file
4. After the AI action, add "Create item" in a Microsoft List with the following columns mapped from the AI output:
   - Vendor Name
   - Invoice ID
   - Invoice Date
   - Due Date
   - Amount Due
5. Add a condition: if the **confidence score** of the extraction is below 0.8, send an alert email to the finance team for manual review
6. Test by emailing a sample PDF invoice to yourself

**Key Learning Points:**
- Using the pre-built AI Builder invoice processing model — no training required
- Accessing AI model output fields as dynamic content in a flow
- Using confidence scores to determine when human review is needed
- Integrating AI extraction into an end-to-end business process

> **References:**
> - [Use the invoice processing prebuilt model](https://learn.microsoft.com/ai-builder/prebuilt-invoice-processing)
> - [AI Builder in Power Automate](https://learn.microsoft.com/ai-builder/use-in-flow-overview)

---

## — Break (15 minutes) —

---

## Segment 5: Lab C — AI Builder Prompts: Summarise and Classify Text in a Flow (25 minutes)

**Business Context:** Teams often receive long emails, feedback submissions, or support tickets and need to quickly understand the topic and sentiment without reading every word. AI Builder Prompts use a GPT-based model to summarise, classify, or rewrite text directly inside a flow — enabling intelligent triage and routing without custom model training.

**License Required:** AI Builder credits

**Connectors Used:** Microsoft Forms, AI Builder (Prompts), Outlook, SharePoint (Microsoft Lists)

**Steps:**
1. Create a Microsoft Form titled "Customer Feedback" with a single long-text field: "Please share your feedback"
2. Build an automated flow triggered by "When a new response is submitted" in Forms
3. Add "Get response details" to retrieve the feedback text
4. Add the **AI Builder** action: "Create text with GPT using a prompt"
5. Build two prompts (or one combined prompt with structured output):
   - **Prompt 1 – Summarise:** `"Summarise the following customer feedback in 2 sentences: [feedback text]"`
   - **Prompt 2 – Classify:** `"Classify the following feedback into one of these categories: Billing, Technical Support, General Enquiry, Compliment. Feedback: [feedback text]"`
6. Add "Create item" in a Microsoft List to store: original feedback, summary, and category
7. Add a condition: if the category is "Technical Support", send an alert email to the IT team
8. Test by submitting sample feedback through the Form

**Key Learning Points:**
- Creating and editing AI Builder Prompts in the Power Automate designer
- Passing dynamic content as input into a prompt
- Structuring prompts effectively for consistent, parseable output
- Using AI-classified output to drive routing logic in a flow

> **References:**
> - [AI Builder prompts overview](https://learn.microsoft.com/ai-builder/prompts-overview)
> - [Use AI Builder prompts in Power Automate](https://learn.microsoft.com/ai-builder/use-a-custom-prompt-in-flow)

---

## Segment 6: Lab D — End-to-End Flow: Email Attachment → AI Extract → Microsoft List (30 minutes)

**Business Context:** This lab consolidates the session's learning by combining an email trigger, AI Builder document extraction, and structured data storage into a single end-to-end process — simulating a real-world intelligent automation scenario that would typically require significant manual effort.

**License Required:** Power Automate Premium + AI Builder credits

**Connectors Used:** Outlook, AI Builder, SharePoint (Microsoft Lists), Microsoft Teams

**Steps:**
1. Build an automated flow triggered by "When a new email arrives (V3)" filtered to a specific sender or subject (e.g., "CV Received" for HR, or "Report Submitted" for Finance)
2. Loop over attachments with "Apply to each"
3. Add **AI Builder** "Extract information from documents" (use the pre-built receipt model or an existing trained custom model if available)
4. Add a **AI Builder Prompt** action to generate a one-line summary of the extracted content
5. Add "Create item" in a Microsoft List storing the extracted fields and the AI-generated summary
6. Post a Teams message to a designated channel: include the sender, attachment name, summary, and a link to the new list item
7. Test end-to-end with a sample email and attachment

**Key Learning Points:**
- Combining multiple AI Builder actions in one flow
- Chaining AI extraction output into a prompt for further processing
- End-to-end integration across Outlook, AI Builder, Lists, and Teams
- Reviewing the flow run history to debug multi-step AI flows

> **References:**
> - [AI Builder document processing](https://learn.microsoft.com/ai-builder/form-processing-model-overview)
> - [AI Builder prompts overview](https://learn.microsoft.com/ai-builder/prompts-overview)

---

## Segment 7: Wrap-Up, Q&A & Next Steps (10 minutes)

- Recap of what was built today: Desktop flows, AI document extraction, AI Prompts, end-to-end integration
- When to use Desktop flows vs. Cloud flows vs. AI Builder
- Governance considerations: managing AI Builder model usage and credit consumption
- Encourage participants to identify one process in their team that could benefit from AI or RPA
- Next steps and continued learning

### Recommended Learning Resources
- [Power Automate Desktop documentation](https://learn.microsoft.com/power-automate/desktop-flows/introduction)
- [AI Builder documentation](https://learn.microsoft.com/ai-builder/overview)
- [AI Builder prompts](https://learn.microsoft.com/ai-builder/prompts-overview)
- [Power Automate learning paths on Microsoft Learn](https://learn.microsoft.com/training/powerplatform/power-automate)
- [Responsible AI FAQs — Power Platform](https://learn.microsoft.com/power-platform/responsible-ai-faqs)
- [Power Automate community](https://powerusers.microsoft.com/t5/Microsoft-Power-Automate/ct-p/MPACommunity)

---

## Trainer Preparation Checklist

- [ ] All participants have a **Power Automate Premium** (or Process) license assigned
- [ ] All participants have **AI Builder credits** available in the environment
- [ ] **Lab A:** Power Automate Desktop is installed on each participant's machine ([aka.ms/powerautomatedesktop](https://aka.ms/powerautomatedesktop))
- [ ] **Lab A:** Participants have signed in to Power Automate Desktop with their work account
- [ ] **Lab B:** A Microsoft List is pre-created with columns: Vendor Name, Invoice ID, Invoice Date, Due Date, Amount Due, Confidence Score
- [ ] **Lab B:** A sample PDF invoice is prepared to test the email trigger
- [ ] **Lab B:** A finance team alert email address is identified for low-confidence routing
- [ ] **Lab C:** A Microsoft Form titled "Customer Feedback" is pre-built or participants create their own
- [ ] **Lab C:** A Microsoft List is pre-created with columns: Feedback, Summary, Category
- [ ] **Lab C:** An IT team alert email address is identified for Technical Support routing
- [ ] **Lab D:** A SharePoint document library or list is designated for end-to-end storage
- [ ] **Lab D:** A Teams channel is designated for the end-to-end notification
- [ ] AI Builder is enabled in the Power Platform environment (check with your admin)
- [ ] DLP policies reviewed — confirm AI Builder and premium connectors are permitted in your tenant
