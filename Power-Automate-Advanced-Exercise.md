---
  title: 'Power Automate Advanced: Hands-On Lab Exercises'
  module: 'Power Automate Advanced'
---

# Power Automate Advanced — Hands-On Lab Exercises

## Lab Introduction

These exercises accompany the Power Automate Advanced training session. Each lab maps to a segment of the session agenda and is designed to be completed independently or in sequence. Labs use Power Automate Desktop (PAD) and AI Builder capabilities that require a Power Automate Premium license.

This set of exercises covers the following labs:

- **Lab A**: RPA — Global Weather Report using Power Automate Desktop
- **Lab B**: Extract Invoice Data with AI Builder Document Processing
- **Lab C**: AI Builder Prompts — Summarise and Classify Text in a Flow
- **Lab D**: Resilient Flows — Try/Catch/Finally Error Handling

---

## Pre-requisites

- **Power Automate Premium license** assigned to your account
- **Power Automate Desktop (PAD)** installed on your Windows machine
  - Download: [aka.ms/powerautomatedesktop](https://aka.ms/powerautomatedesktop)
- **Microsoft Excel** installed on your machine (Microsoft 365 version recommended)
- **Edge or Chrome browser** installed (PAD web recorder supports both)
- Signed in to Power Automate Desktop with your work account
- Internet access to reach [weather.com](https://weather.com)

---

# Lab A: RPA — Global Weather Report

## Lab Introduction

In this lab, you will build a desktop flow in Power Automate Desktop that acts as a virtual assistant for your global office team. The flow visits weather.com, searches for each city by simulating a user typing into the search box, extracts the 5-day daily forecast, and writes the results into a structured Excel workbook — fully automatically.

This lab will guide you through the following tasks:

- Create a desktop flow, define three subflows (SearchCity, ExtractForecast, WriteRows), and set up an Excel workbook with headers
- Use the PAD Web Recorder to record the weather.com search interaction inside the SearchCity subflow
- Build the ExtractForecast subflow to wait, scroll, and extract the 5-day forecast as a DataTable
- Build the WriteRows subflow and assemble the Main flow with a city loop and Run subflow calls
- Save the Excel workbook and run the completed flow to verify the output

## Estimated Timing: 35 minutes

## Lab Scenario

Your organisation has offices in Paris, New York City, and Tokyo. The operations team manually checks the weather forecast for each city every morning before planning field activities and travel. This takes 10–15 minutes of daily effort.

Your task is to automate this process by building a desktop flow that:

- Opens a browser and navigates to weather.com
- Searches for each city using the site's search box — just as a person would
- Extracts the 5-day daily forecast (Day, Conditions, Low temperature, High temperature)
- Logs all results into an Excel workbook with one row per forecast day

## Architecture Overview

```
Power Automate Desktop — RPA Flow
===================================

Input
└── Cities list variable
    ├── Paris
    ├── New York City
    └── Tokyo

                    ↓ (For each city)

Main Flow (orchestrator)
├── Set Cities, RowIndex, launch Excel, write headers
└── For each CurrentItem in Cities:
      Run subflow → SearchCity
      Run subflow → ExtractForecast
      Run subflow → WriteRows
      Close web browser

Subflow: SearchCity
├── Launch browser → https://weather.com
├── Click city button (top navigation)
├── Type %CurrentItem% into search box     ← parameterised from recording
├── Click matching city result             
└── Click "10 Day" tab

Subflow: ExtractForecast
├── Wait for "Daily Forecast" heading
├── Scroll to "Daily Forecast" section     ← triggers lazy-load rendering
├── Extract data from web page
|   └── ForecastData (DataTable: Day | Conditions | Low | High)
└── Close web browser

Subflow: WriteRows
├── Set RowCounter = 0
└── For each ForecastRow in ForecastData (while RowCounter < 5):
      Write City | Day | Conditions | Low | High → Excel row %RowIndex%
      Increment RowIndex, RowCounter

                    ↓ (After all cities)

Excel Output
├── Row 1 (headers): City | Day | Conditions | Low | High
├── Rows 2–6:   Paris forecast (5 days)
├── Rows 7–11:  New York City forecast (5 days)
└── Rows 12–16: Tokyo forecast (5 days)
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following Power Automate Desktop skills:

| Task | Description | Duration |
|---|---|---|
| Task 1 | Set up the desktop flow, create the three subflows, set variables, and build the Excel workbook | 5 min |
| Task 2 | Record the weather.com search interaction inside the **SearchCity** subflow | 8 min |
| Task 3 | Build the **ExtractForecast** subflow with wait, scroll, and data extraction | 7 min |
| Task 4 | Build the **WriteRows** subflow and assemble the Main flow with a city loop | 8 min |
| Task 5 | Save the workbook and run the flow to verify the output | 5 min |

---

## Task 1: Set Up the Desktop Flow, Variables, and Excel Workbook

In this task, you will create your desktop flow, create the three subflows that will organise the work, define the city list variable, and open a new Excel workbook with column headers ready to receive forecast data.

> **What is a subflow?** A subflow is a named block of actions within the same desktop flow that can be called from the Main flow or from other subflows. Subflows share all variables with the Main flow — no parameters need to be passed explicitly. They improve readability, make each section independently runnable for testing, and keep the Main flow clean as a pure orchestrator.

### Create the Desktop Flow

1. Open **Power Automate Desktop** from the Start menu.

2. On the **Flows** screen, select **+ New** → **Flow**.

3. In the **Create a flow** dialog, enter the flow name:
   ```
   RPA - Global Weather Report
   ```

4. Select **Create**. The flow designer opens with an empty canvas.

   > **What is the flow designer?** The designer is where you build your flow by adding, arranging, and configuring actions. The left panel lists all available action categories. The centre panel is your flow canvas. The right panel shows properties for the selected action.

### Create the Three Subflows

Before adding any actions, create the three subflows that will hold the browser and Excel logic. The Main flow tab will only contain the orchestration (variables, loop, exception handling).

5. At the top of the flow canvas, select the **Subflows** tab (next to the **Main** tab).

6. Select **+ New**.

7. Enter the name:
   ```
   SearchCity
   ```
   Select **Save**. An empty subflow canvas opens.

8. Select the **Subflows** tab again and select **+ New**.

9. Enter the name:
   ```
   ExtractForecast
   ```
   Select **Save**.

10. Repeat to create a third subflow named:
    ```
    WriteRows
    ```
    Select **Save**.

11. Select the **Main** tab to return to the main flow canvas. You will now build the Main flow first, then populate each subflow in Tasks 2–4.

    > **Tip:** You can switch between Main and any subflow at any time using the tabs at the top of the canvas. All variables created in any tab are shared across all subflows and Main.

### Create the Cities List Variable

12. In the **Actions** panel on the left, expand **Variables**.

13. Double-click **Set variable** to add it to the canvas.

14. Configure the following:

   | Property | Value |
   |---|---|
   | Variable | `Cities` |
   | Value | `%["Paris", "New York City", "Tokyo"]%` |

   > **Note:** In PAD, list literals are written as JSON arrays inside `%` delimiters. This creates a list variable with three text items.

15. Select **Save** to confirm the action.

### Create the Row Index Variable

16. Add a second **Set variable** action below the first.

17. Configure it as follows:
    | Property | Value |
    |---|---|
    | Variable | `RowIndex` |
    | Value | `2` |

    > **Why start at 2?** Row 1 will be reserved for the column headers you are about to write. Data rows start at row 2.

18. Select **Save**.

### Launch Excel and Write Headers

19. In the **Actions** panel, expand **Excel**.

20. Double-click **Launch Excel** to add it to the canvas.

21. Configure the action:

    | Property | Value |
    |---|---|
    | Launch Excel | with a blank document |
    | Make instance visible | On |

22. Select **Save**. PAD stores the Excel instance in a variable called `ExcelInstance` by default.

23. Add a **Write to Excel worksheet** action.

24. Configure it to write the header row:

    | Property | Value |
    |---|---|
    | Excel instance | `%ExcelInstance%` |
    | Value to write | `City` |
    | Write mode | On specified cell |
    | Column | `A` |
    | Row | `1` |

25. Select **Save**.

26. Add four more **Write to Excel worksheet** actions for the remaining headers. Use the same settings but change the column and value each time:

    | Column | Value to write |
    |---|---|
    | `B` | `Day` |
    | `C` | `Conditions` |
    | `D` | `Low` |
    | `E` | `High` |

    > **Tip:** You can right-click any action on the canvas and select **Copy**, then paste and edit to save time repeating similar actions.

27. Press **Ctrl + S** to save your flow.

---

## Task 2: Record the SearchCity Subflow

In this task you will use the PAD **Web Recorder** to capture a single manual browser session on weather.com for the city of Paris. The generated actions will be placed inside the **SearchCity** subflow. You will parameterise them in Task 4.

You need to install the Power Automate browser extension for Microsoft Edge if you haven't already: https://learn.microsoft.com/en-us/power-automate/desktop-flows/install-browser-extensions

> **Why record for just one city first?** The recorder captures what you actually do step by step. It is much easier to record one accurate interaction and then replace the hardcoded city name with a variable, rather than trying to build browser actions manually from scratch.

### Switch to the SearchCity Subflow

1. Select the **SearchCity** tab at the top of the flow canvas.

   All actions you record or add in this task will live here.

### Start the Web Recorder

2. In the PAD designer toolbar, select **Recorder** (the icon shows a browser window with a record button).

3. In the **Recorder** dialog, select the ellipsis (...) and select Microsoft Edge.

4. Select **Record**.

### Record the Browser Interaction

5. In the browser address bar, navigate to:
   ```
   https://weather.com
   ```

6. Wait for the page to load fully.

   > **Note:** A notification popup ("We'd like to send you notifications") may appear. Click **Dismiss** to close it before continuing. The recorder will capture this click — you can delete it from the generated actions afterwards.

7. In the top navigation bar, click the SVG drop down button beside the current city name (e.g., drop down arrow beside **New York City**). This opens the location search panel.

8. The search box labelled **Search for a location** appears. Click inside it.

9. If the search box contains existing text, press **Ctrl + A** then **Delete** to clear it.

10. Type:
   ```
   Paris
   ```

11. Wait for the dropdown results to appear.

12. Click the object **Paris Paragraph** from the dropdown list.

13. Wait for the Paris weather page to load.

14. Click the **10 Day** tab in the top navigation.

15. Wait for the 10-day forecast page to load.

16. In the recorder toolbar, click **Done** to stop recording.

### Review the Generated Actions

17. PAD adds the recorded browser actions to the **SearchCity** subflow canvas. Delete unnecessary actions so that the flow contains only the following steps:

    ```
    Launch new Microsoft Edge, navigate to 'https://weather.com' and store the instance into 'Browser'
    Click on Svg of web page
    Populate text field Input text with 'Paris' using emulated typing
    Click on Paragraph 'Paris' of web page
    Click on Anchor '10 Day' of web page
    ```
    
18. Press **Ctrl + S** to save your flow.

---

## Task 3: Build the ExtractForecast Subflow

In this task you will add three actions to the **ExtractForecast** subflow to wait for the page to be ready, scroll to the forecast table, and extract its data into a DataTable variable.

### Switch to the ExtractForecast Subflow

1. Select the **ExtractForecast** tab at the top of the flow canvas.

### Add a Wait Action

2. Click on the empty canvas to position your cursor.

3. In the **Actions** panel, expand **Browser automation**.

4. Double-click **Wait for web page content** and add it to the flow.

5. Configure the action:

   | Property | Value |
   |---|---|
   | Web browser instance | `%Browser%` (the variable PAD created for your browser) |
   | Wait for page to | Contain element |
   | UI element | Click the drop down. Select **Add UI element**, switch to your browser, and Ctrl + Click the **Daily Forecast** heading text on the page |

   > **Why wait first?** Weather.com loads forecast data asynchronously. Attempting to extract data before the elements exist in the DOM will cause the flow to fail. Waiting for the heading confirms the content is ready.

6. Select **Save**.

### Add a Scroll Action

7. In the **Actions** search box, type `javascript` and double-click **Run JavaScript function on web page** to add it.

8. Configure it:

   | Property | Value |
   |---|---|
   | Web browser instance | `%Browser%` |
   | JavaScript function | `function ExecuteScript() { window.scrollBy(0, 800); }` |

   > **Why scroll?** Weather.com uses lazy loading — forecast rows only render fully in the DOM once they are scrolled into view. Scrolling down triggers this rendering before extraction begins. The **Scroll web page** action was removed in newer versions of PAD; this JavaScript equivalent achieves the same result.

9. Select **Save**.

### Add the Data Extraction Action

10. Add an **Extract data from web page** action from **Browser automation** → **Web data extraction**.

11. Configure the basic properties:

    | Property | Value |
    |---|---|
    | Web browser instance | `%Browser%` |
    | Variables produced | `ForecastData` |

12. To define what to extract, go to the browser that has the Weather.com page open. 

13. In the browser, right-click on the **first forecast row** in the Daily Forecast table (e.g., the row showing "Today or Tonight").

14. From the context menu, select **Extract element value** → **Text: [the full row text]**. PAD adds it in the Extraction preview.

15. Right-click on the **second forecast row** (e.g., "Thu 20"). Select **Extract element value** → **Text**.

    > **Pattern detection:** After selecting two rows, PAD recognises the repeating structure and automatically generates a pattern to extract **all matching rows** on the page as a DataTable. You will see a preview table appear in the Extract Helper panel showing all forecast rows.

16. Do the steps 13 to 15 for the Conditions, Low temperature, and High temperature columns. The Extract Helper should now show a table with four columns and many rows (one for each forecast day). Verify that the extracted columns correspond to: Day, Conditions, Low temperature, High temperature.

17. Select **Finish** to close the Extraction preview.

18. Select **Save**. `ForecastData` is now a DataTable variable containing all forecast rows for the current city.

19. Add a **Close web browser** action to close the browser after extraction:

    | Property | Value |
    |---|---|
    | Web browser instance | `%Browser%` |

20. Press **Ctrl + S** to save your flow.

---

## Task 4: Build the WriteRows Subflow and Assemble the Main Flow

In this task you will build the **WriteRows** subflow that writes up to 5 forecast rows to Excel, then return to the **Main** flow to add the city loop, the **Run subflow** calls, and a **Close web browser** action.

### Build the WriteRows Subflow

1. Select the **WriteRows** tab at the top of the flow canvas.

2. Add a **Set variable** action:

   | Property | Value |
   |---|---|
   | Variable | `RowCounter` |
   | Value | `0` |

3. Add a **For each** loop:

   | Property | Value |
   |---|---|
   | Value to iterate | `%ForecastData%` |
   | Store into | `ForecastRow` |

4. Inside the loop, add an **If** condition:

   | Property | Value |
   |---|---|
   | First operand | `%RowCounter%` |
   | Operator | Less than (`<`) |
   | Second operand | `5` |

5. Inside the **If** block, add five **Write to Excel worksheet** actions:

   | Column | Value to write |
   |---|---|
   | `A` | `%CurrentItem%` |
   | `B` | `%ForecastRow[0]%` |
   | `C` | `%ForecastRow[1]%` |
   | `D` | `%ForecastRow[2]%` |
   | `E` | `%ForecastRow[3]%` |

   Set the **Row** property of all five actions to `%RowIndex%`.

   > **Note:** `%CurrentItem%` is set by the **For each** loop in the Main flow. Ignore the error message that it doesn't exist for now. DataTable row values are accessed by zero-based column index — adjust the indices if the extraction captured columns in a different order.

6. After the five write actions (still inside the **If** block), add two **Increase variable** actions:
   - Increase `RowIndex` by `1`
   - Increase `RowCounter` by `1`

7. Press **Ctrl + S** to save.

### Assemble the Main Flow

Now return to the Main flow to add the city loop and the three subflow calls.

8. Select the **Main** tab.

9. Click after the last **Write to Excel worksheet** header action to position your cursor at the end of the existing Main actions.

10. Add a **For each** loop from the **Loops** category:

    | Property | Value |
    |---|---|
    | Iterate over | `%Cities%` |
    | Store current item into | `CurrentItem` |

11. Inside the loop body, add a **Run subflow** action from **Flow control**:

    | Property | Value |
    |---|---|
    | Subflow name | `SearchCity` |

12. Add a second **Run subflow** action:

    | Property | Value |
    |---|---|
    | Subflow name | `ExtractForecast` |

13. Add a third **Run subflow** action:

    | Property | Value |
    |---|---|
    | Subflow name | `WriteRows` |

14. Add a **Close web browser** action after the three subflow calls (still inside the loop):

    | Property | Value |
    |---|---|
    | Web browser instance | `%Browser%` |

15. Press **Ctrl + S** to save your flow.

### Parameterise the SearchCity Subflow

Now switch back to the SearchCity subflow to replace the hardcoded city name and update the UI element selector.

16. Select the **SearchCity** tab.

### Replace the Hardcoded City in the Type Action

17. Find the **Populate text field on web page** action that contains the text `Paris`.

18. Double-click the action to open its properties.

19. In the **Text** field, replace `Paris` with:
    ```
    %CurrentItem%
    ```

20. Select **Save**.

### Update the City Result Click

When the recorder captured the click on "Paris", it stored an exact text selector. Change this so it works for "New York City" and "Tokyo" as well.

21. Find the **Click link on web page** action that clicks the city search result (e.g., "Paris").

22. Double-click the action to open its properties.

23. Next to the **UI element** field, select the **Edit** (pencil) icon to open the UI Element editor.

24. In the UI Element editor, toggle Text editor on (top-right of the dialog).

25. Replace the entire selector with:
    ```
    [Text*="%CurrentItem%"]
    ```

26. Select **Save** to close the UI Element editor, then **Save** the action.

27. Press **Ctrl + S** to save your flow.

---

## Task 5: Save the Workbook and Run the Flow

In this task you will add the Excel save and close actions, then run the completed flow.

### Save and Close the Workbook

These actions go **outside and after** the For each loop, so they run once when all cities are complete.

1. Go back to the Main subflow. Add a **Save Excel** action from the **Excel** category. This must be placed after the For each loop.

    Configure it:

    | Property | Value |
    |---|---|
    | Excel instance | `%ExcelInstance%` |
    | Save mode | Save document as |
    | Document format | Excel workbook (.xlsx) |
    | Document path | Choose a folder on your desktop, e.g. `C:\Users\[YourName]\Desktop\WeatherReport.xlsx` |

2. Add a **Close Excel** action:

    | Property | Value |
    |---|---|
    | Excel instance | `%ExcelInstance%` |
    | Before closing Excel | Do not save document |

3. Press **Ctrl + S** to save your completed flow.

### Run the Flow

4. In the PAD designer, select **Run** (the play button in the toolbar).

5. Observe the flow executing:
   - A browser window opens and navigates to weather.com
   - The search box is populated and a city is selected automatically
   - The 10-day forecast page loads
   - Data extraction occurs (the page may highlight extracted elements briefly)
   - The browser closes and the sequence repeats for the next city

6. When the flow completes, the status bar shows **Flow run completed successfully**.

   > **If the flow stops with an error:** Check the action highlighted in red. Common issues include:
   > - **UI element not found**: The city dropdown result text did not match `%CurrentItem%`. Open the UI element and verify the Contains operator is set correctly.
   > - **Timeout on Wait action**: The page took longer than 30 seconds to load. Increase the timeout value in the Wait action.
   > - **Excel write error**: Verify `%RowIndex%` is incrementing correctly and is not zero.

### Verify the Excel Output

7. Navigate to the save location and open **WeatherReport.xlsx**.

8. Verify the workbook contains 15 data rows (5 days × 3 cities) plus the header row, structured as:

    | A (City) | B (Day) | C (Conditions) | D (Low) | E (High) |
    |---|---|---|---|---|
    | Paris | Today | PM Thunderstorms | 60° | 82° |
    | Paris | Thu 20 | Partly Cloudy | 59° | 78° |
    | … | … | … | … | … |
    | Tokyo | Today | … | … | … |

---

Your completed flow structure:

```
── Main ───────────────────────────────────────────────────────────────
Set variable: Cities = ["Paris", "New York City", "Tokyo"]
Set variable: RowIndex = 2
Launch Excel (blank workbook)
Write to Excel: A1="City" B1="Day" C1="Conditions" D1="Low" E1="High"
For each CurrentItem in Cities:
    Run subflow: SearchCity
    Run subflow: ExtractForecast
    Run subflow: WriteRows
    Close web browser
End For each
Save Excel → WeatherReport.xlsx
Close Excel

── Subflow: SearchCity ────────────────────────────────────────────────
Launch new Microsoft Edge and navigate to https://weather.com
Click city button (top nav)
Populate text field: %CurrentItem%
Click link containing: %CurrentItem%
Click "10 Day" tab

── Subflow: ExtractForecast ───────────────────────────────────────────
Wait for web page content: "Daily Forecast" heading
Scroll web page to: "Daily Forecast" heading
Extract data from web page → ForecastData
Close web browser

── Subflow: WriteRows ─────────────────────────────────────────────────
Set variable: RowCounter = 0
For each ForecastRow in ForecastData:
    If RowCounter < 5:
        Write City, Day, Conditions, Low, High → Excel row %RowIndex%
        Increase RowIndex by 1
        Increase RowCounter by 1
    End
End
```

---

## Lab A Summary

### What You Accomplished

✅ **Task 1**: Created a desktop flow with a cities list variable and an Excel workbook with headers

✅ **Task 2**: Used the PAD Web Recorder to capture a complete browser session on weather.com, simulating a user searching for a city by name

✅ **Task 3**: Added wait, scroll, and data extraction actions to reliably capture the 5-day forecast as a DataTable

✅ **Task 4**: Parameterised the recorded actions with `%CurrentItem%` and wrapped everything in a For each loop with a nested row-writing loop limited to 5 rows

✅ **Task 5**: Saved the workbook and ran the completed flow, verifying a 15-row weather report across three global cities

### Key Concepts You Practised

| Concept | Where it appeared | Learn path module |
|---|---|---|
| PAD flow designer and actions | All tasks | Build your first desktop flow |
| List variables and index variables | Task 1 | Handle variables |
| For each loop | Task 4 | Automate repetitive tasks using loops |
| Conditional actions (If / Less than) | Task 4 | Adjust process behavior using conditional actions |
| Subflows (SearchCity, ExtractForecast, WriteRows) | Task 1, 4 | Configure flow control |
| Web Recorder | Task 2 | Generate flows by recording |
| Web automation (navigate, click, populate, extract) | Tasks 2–3 | Web automation |

### Next Steps

- **Schedule this flow as unattended RPA**: Go to `make.powerautomate.com`, create a cloud flow with a **Recurrence** trigger (e.g., every weekday at 7:00 AM), and add a **Run a flow built with Power Automate for desktop** action pointing to this flow.
- **Add more cities**: Extend the `Cities` variable list — the loop handles any number of cities without further changes.
- **Explore deferred modules**: The following concepts from the [Desktop flows and RPA learning path](https://learn.microsoft.com/training/paths/desktop-flows-robotic-process-automation-power-automate/) were not covered in this lab and are recommended for further study: Input/Output parameters, Scripting (PowerShell/Python), and Flow control (Go-to, regions).

### Recommended References

- [Web automation in Power Automate for desktop](https://learn.microsoft.com/power-automate/desktop-flows/actions-reference/webautomation)
- [Handle variables in Power Automate desktop](https://learn.microsoft.com/power-automate/desktop-flows/variable-data-types)
- [Configure exception handling in Power Automate desktop](https://learn.microsoft.com/power-automate/desktop-flows/exception-handling)
- [Automate repetitive tasks using loops](https://learn.microsoft.com/training/modules/pad-loops/)


---

# Lab B: Extract Invoice Data with AI Builder Document Processing

## Lab Introduction

In this lab you will build an **automated cloud flow** in the Power Automate web portal that reads incoming invoice emails and extracts their key data automatically. Instead of a person opening each PDF and retyping the vendor, invoice number, dates, and totals into a tracking list, AI Builder's **prebuilt invoice processing model** reads the document for you and returns structured fields — each with a **confidence score** so you can flag anything that needs a human check.

This lab will guide you through the following tasks:

- Create an automated cloud flow triggered by incoming Outlook emails that have attachments
- Loop over each attachment and pass it to the AI Builder **Process invoices** action
- Branch on the model's **confidence score** to separate high-confidence results from those needing review
- Log every extracted invoice as an item in a Microsoft List, flagging low-confidence extractions for follow-up
- Test the flow end to end with a real invoice attachment

> **Pattern learned:** Automated trigger → loop over documents → **AI extraction** → branch on confidence → write a record. This is the standard shape for intelligent document automation.

## Estimated Timing: 30 minutes

## Pre-requisites

- **Power Automate Premium** license and **AI Builder credits** available in your environment.
- A **Microsoft 365** account with access to Outlook, SharePoint / Microsoft Lists.
- A sample **invoice** file (PDF or image). AI Builder's prebuilt invoice model supports PDF, JPG, PNG, and TIFF. A clear, English-language invoice works best for a first test.
- Confirm the **default environment** (or a Managed Environment) has AI Builder enabled. See [AI Builder licensing](https://learn.microsoft.com/ai-builder/administer-licensing).

## Lab Scenario

Your accounts-payable team receives dozens of supplier invoices by email every week. Today, someone opens each attachment, reads off the vendor name, invoice number, date, and total, and types them into a tracking list by hand — slow, error-prone, and impossible to audit. Your task is to replace that manual data entry with an AI-powered flow that reads each invoice the moment it arrives, records the extracted fields automatically, and flags any invoice the model was unsure about so a human can verify it.

## Architecture Overview

```
AI Builder Invoice Extraction Flow
==================================

Trigger
└── Automated: When a new email arrives (V3)  [Only with Attachments = Yes]

                    ↓

Flow
├── Control: Apply to each (Attachments)
│   ├── AI Builder: Process invoices
│   │   └── Input: Attachment content (contentBytes)
│   ├── Control: Condition — Confidence ≥ 0.80?
│   │   ├── True  → Create item (Review = No)
│   │   └── False → Create item (Review = Yes)  + Send review email
│   └── (fields: Vendor, Invoice ID, Dates, Total)
│
└── Output: Microsoft List "Invoice Register"
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description | Duration |
|---|---|---|
| Task 1 | Create the **Invoice Register** Microsoft List | 5 min |
| Task 2 | Build the automated flow and configure the email trigger | 5 min |
| Task 3 | Add the AI Builder **Process invoices** action | 8 min |
| Task 4 | Branch on the confidence score and write to the list | 8 min |
| Task 5 | Save, test, and verify the extraction | 4 min |

---

## Task 1: Create the Invoice Register Microsoft List

1. Go to [Microsoft Lists](https://www.microsoft.com/microsoft-365/microsoft-lists) and select **Create new** → **Create from blank** → **List**.

2. Name it **Invoice Register**. Select **Create**.

3. Add the following columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for **Vendor name** |
   | InvoiceID | Single line of text |
   | InvoiceDate | Date and time (Include Time = No) |
   | InvoiceTotal | Number |
   | ReviewNeeded | Choice (Yes, No) |
   | SourceEmail | Single line of text |

   > **Tip:** Note each column's **internal name** (open **Column settings → Edit**). You will map the flow's fields to these later; mismatched internal names are the most common cause of a failed **Create item** action.

---

## Task 2: Build the Automated Flow and Configure the Trigger

1. Sign in to [https://make.powerautomate.com](https://make.powerautomate.com) and confirm the correct environment is selected in the top-right picker.

2. Select **Create** → **Automated cloud flow**.

3. In **Flow name**, enter **Invoice Extraction to List**.

4. Search the trigger **When a new email arrives (V3)** and select **When a new email arrives (V3) - Office 365 Outlook**, then select **Create**.

   > If prompted, sign in to create the **Office 365 Outlook** connection using your work account.

5. Select the trigger, choose **Show all** under Advanced parameters, and set:

   | Parameter | Value |
   | --- | --- |
   | **Include Attachments** | Yes |
   | **Only with Attachments** | Yes |
   | **Subject Filter** | Invoice |
   | **Folder** | Inbox |

   > **Why these settings?** *Include Attachments = Yes* makes each file's bytes available to AI Builder. *Only with Attachments = Yes* stops the flow starting for mail that has nothing to read. *Subject Filter = Invoice* narrows processing to invoice emails.

---

## Task 3: Add the AI Builder Extract Information from Invoices Action

1. Below the trigger, select the plus sign (**+**) → **Add an action**.

2. Search for **Apply to each** and select the **Control** action **Apply to each**.

3. In **Select an output from previous steps**, insert the dynamic content **Attachments** (from the trigger).

   > The trigger returns a *collection* of attachments; `Apply to each` runs the inner actions once per attachment.

4. Inside the loop, select **Add an action**. Search **Process invoices** and select **Process invoices - AI Builder**.

   > If prompted, allow the flow to use AI Builder in this environment. This action uses the **prebuilt invoice processing model** — no training required.

5. Configure the action:

   | Parameter | Value |
   | --- | --- |
   | **Invoice file** | Switch the field to the **fx** editor and enter the expression below |

   > **Invoice file** — in the expression editor enter:
   >
   > ```
   > items('Apply_to_each')?['contentBytes']
   > ```
   >
   > Replace `Apply_to_each` with your loop's actual name if you renamed it (spaces become underscores). The V3 trigger exposes each attachment's binary content as `contentBytes`.

---

## Task 4: Branch on Confidence and Write to the List

1. Still inside **Apply to each**, below the AI Builder action, add a **Condition** control.

2. Configure the condition:
   - Set value to **And**
   - Left value: open the **lightning** icon and insert the **Confidence of vendor name** dynamic content from **Process invoices**.
   - Operator: **is greater than or equal to**
   - Right value: `0.8`
   - Add row and do the same steps above for the following:
      - **Confidence of invoice ID** 
      - **Confidence of invoice date**
      - **Confidence of invoice total**

   > A confidence of **0.8 (80%)** is a reasonable starting threshold. Tune it to your tolerance: raise it to catch more borderline reads, lower it to send fewer items to review.

3. In the **True** branch, add **Create item - SharePoint** and configure:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | Select the site hosting **Invoice Register** |
   | **List Name** | Invoice Register |
   | **Title** | Insert **Vendor name** (value) |
   | **InvoiceID** | Insert **Invoice ID** (value) |
   | **InvoiceDate** | Insert **Invoice date (date)** (value) |
   | **InvoiceTotal** | Insert **Invoice total (number)** (value) |
   | **ReviewNeeded** | `No` |
   | **SourceEmail** | Insert the trigger's **From** |

4. In the **False** branch, add another **Create item - SharePoint** with the same mapping, but set **ReviewNeeded** to `Yes`.

5. Still in the **False** branch, add **Send an email (V2) - Office 365 Outlook** to alert a reviewer:

   | Parameter | Value |
   | --- | --- |
   | **To** | Your accounts-payable reviewer (use your own address for testing) |
   | **Subject** | `Low-confidence invoice needs review` |
   | **Body** | Insert the formula below to display the fields |

   ```
   Vendor: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/vendorName/valueText']}
   Confidence of Vendor Name: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/vendorName/confidence']}
   Invoice ID: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/invoiceId/valueText']}
   Confidence of Invoice ID: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/invoiceId/confidence']}
   Total: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/invoiceTotal/valueNumber']}
   Confidence Invoice amount: @{outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/invoiceTotal/confidence']}
   ```

   > **Why branch on confidence?** Straight-through processing is only safe when the model is sure. Routing low-confidence reads to a human keeps the automation trustworthy without blocking the reliable majority.

---

## Task 5: Save, Test, and Verify

1. Select **Save**.

2. Select **Test** → **Manually** → **Test**.

3. Send yourself an email whose **subject contains "Invoice"** with an invoice **PDF or image attached**. Sample invoices can be downloaded from the Power-Automate-Advanced-Ref folder.

4. Return to Power Automate. Within a minute the run appears — select it and expand **Process invoices** to inspect the extracted values and confidence scores.

5. Open the **Invoice Register** list and confirm a new item was created with the vendor, invoice ID, date, and total.

6. Test the review path: send an invoice that is incorrect or incomplete, and confirm the item is created with **ReviewNeeded = Yes** and the review email is sent.

> **Troubleshooting:** If the AI action fails with an authorization error, confirm AI Builder is enabled and credits are available in the environment. If fields come back empty, the document may be low-quality or in an unsupported layout — try a clearer invoice. If **Create item** fails, verify the list column internal names match the fields you mapped.

> *References: [AI Builder prebuilt invoice processing model](https://learn.microsoft.com/ai-builder/prebuilt-invoice-processing) · [Use the invoice processing model in Power Automate](https://learn.microsoft.com/ai-builder/invoice-processing-in-flow)*

> **Congratulations!** You built an intelligent document-processing flow that reads invoices, records structured data, and escalates only what a human needs to check.

---

# Lab C: AI Builder Prompts — Summarise and Classify Text in a Flow

## Lab Introduction

In this lab you will build an **automated cloud flow** that turns free-text customer feedback into structured, actionable data using **AI Builder Prompts** (GPT). When a customer submits a feedback form, the flow uses a custom prompt to produce a short **summary** and assign a **category** (for example *Technical Support*, *Billing*, *Product Feedback*, or *Praise*). Results are stored in a Microsoft List, and any **Technical Support** item is automatically routed to the IT team by email.

This lab will guide you through the following tasks:

- Create a Microsoft Form to collect customer feedback
- Create a Microsoft List to store the AI-enriched results
- Build a custom **AI Builder Prompt** that returns a summary and a category as structured JSON
- Call the prompt from a flow, parse its output, and write the results to the list
- Conditionally route Technical Support feedback to the IT team

> **Pattern learned:** Automated trigger → **generative AI prompt** → parse structured output → store and route. This is the core pattern for infusing generative AI into a business process.

## Estimated Timing: 30 minutes

## Pre-requisites

- **Power Automate Premium** license and **AI Builder credits** in your environment.
- Access to **Microsoft Forms**, **SharePoint / Microsoft Lists**, and **Outlook**.
- AI Builder **Prompts** available in your region and environment. See [AI Builder prompts overview](https://learn.microsoft.com/ai-builder/prompts-overview).

## Lab Scenario

Your support team receives a steady stream of free-text feedback through an online form. Reading every message, boiling it down to a one-line summary, and sorting it into the right queue takes hours each week — and different people categorise inconsistently. Your task is to let a generative-AI prompt do the first pass: summarise each comment, classify it into a fixed set of categories, and route the technical issues straight to IT — so the team spends its time responding, not triaging.

## Architecture Overview

```
AI Builder Prompt — Summarise & Classify Flow
=============================================

Trigger
└── Automated: When a new response is submitted (Microsoft Forms)

                    ↓

Flow
├── Microsoft Forms: Get response details
├── AI Builder: Run a prompt  ("Feedback Triage")
│   ├── Input:  feedback text (from form)
│   └── Output: { "summary": "...", "category": "..." }  (JSON)
├── Data operation: Parse JSON  (summary, category)
├── SharePoint: Create item  (Summary, Category, Feedback)
└── Control: Condition — Category = "Technical Support"?
    └── True → Send an email to the IT team

                    ↓

Output: Microsoft List "Customer Feedback"
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description | Duration |
|---|---|---|
| Task 1 | Create the feedback **Form** and results **List** | 5 min |
| Task 2 | Build a custom **AI Builder Prompt** with structured output | 10 min |
| Task 3 | Build the flow and call the prompt | 6 min |
| Task 4 | Parse the output, write to the list, and route Technical Support | 6 min |
| Task 5 | Save, test, and verify | 3 min |

---

## Task 1: Create the Feedback Form and Results List

### Create the Microsoft Form

1. Go to [https://forms.office.com](https://forms.office.com) and select **New Form**.

2. Exit from the Draft with Copilot. Title the form **Customer Feedback**.

3. Add the following questions:

   | Question | Type |
   | --- | --- |
   | Your name | Text |
   | Your email | Text |
   | Your feedback | Text (long answer) |

4. Select **Collect responses** and set it so people **in your organization can respond**. Submit one test response so real data exists.

### Create the Microsoft List

1. Create a new **blank list** named **Customer Feedback** with these columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for the AI **Summary** |
   | Category | Choice (Technical Support, Billing, Product Feedback, Praise, Other) |
   | OriginalFeedback | Multiple lines of text |
   | SubmittedBy | Single line of text |
   | SubmittedEmail | Single line of text |

---

## Task 2: Build a Custom AI Builder Prompt

1. In [Power Automate](https://make.powerautomate.com), in the left navigation select **More → Prompts**, then select **Build your own prompt**.

2. Name the prompt **Feedback Triage**.

3. In the prompt editor, write instructions that ask for a summary and a category. Paste the following as the prompt text:

   ```
   You are a support triage assistant. Read the customer feedback provided and respond
   with ONLY a JSON object, no extra text, in exactly this shape:

   {
     "summary": "<one concise sentence summarising the feedback>",
     "category": "<one of: Technical Support, Billing, Product Feedback, Praise, Other>"
   }

   Customer feedback:
   ```

4. Select **Add content** → **Text**, name the input **FeedbackText**, and place its token at the end of the prompt (right after `Customer feedback:`).

5. Set the model to **GPT-5 chat**.

6. In the model response, select Output: **JSON**.

7. Click the FeedbackText input and paste a sample comment in the Sample data field (for example *"The app crashes every time I upload a file — please fix this urgently."*).

8. Select **Test** to confirm the model returns valid JSON with a sensible category.

9. Test the other categories to ensure the model correctly classifies different types of feedback.
   Category | Sample Comment
   --- | ---
   Technical Support | "The application freezes whenever I try to upload a file larger than 10 MB."
   Billing | "I need clarification on a charge that appeared on my latest invoice."
   Product Feedback | "A dark mode option would make the application more comfortable to use during long work sessions."
   Praise | "Your support team resolved my issue quickly and provided excellent service."
   Other | "I would like to learn more about upcoming training and certification opportunities."

10. Select **Save** to save your prompt. Exit the prompt editor.

   > **Why force JSON?** Generative models return free text by default. Instructing the model to emit a fixed JSON shape makes the output *parseable*, so downstream **Parse JSON** and **Condition** actions work every time.

---

## Task 3: Build the Flow and Call the Prompt

1. Select **Create** → **Automated cloud flow**. Name it **Feedback Triage with AI**.

2. Search the trigger **When a new response is submitted - Microsoft Forms** and select it, then **Create**.

3. In the trigger, set **Form Id** to **Customer Feedback**.

4. Add an action: search **Get response details** and select **Get response details - Microsoft Forms**. Set **Form Id** to **Customer Feedback** and **Response Id** to the trigger's **Response Id** dynamic content.

5. Add an action: search **Run a prompt** and select **Run a prompt - AI Builder**. Set **Prompt** to **Feedback Triage**. Set FeedbackText to the **Your feedback** dynamic content from **Get response details**.

---

## Task 4: Parse the Output, Write to the List, and Route

1. Add a **Parse JSON** action (**Data Operation**):

   | Parameter | Value |
   | --- | --- |
   | **Content** | Insert the prompt action's text output (**Text**) |
   | **Schema** | Paste the schema below |

   ```json
   {
       "type": "object",
       "properties": {
           "summary": { "type": "string" },
           "category": { "type": "string" }
       }
   }
   ```

   > **Tip:** If **Parse JSON** occasionally fails because the model wrapped the JSON in extra text, add a **Compose** step first that trims to the JSON, or tighten the prompt to *"respond with ONLY a JSON object"* (already included above).

2. Add **Create item - SharePoint** and configure:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | Enter custom value then paste the URL of the site hosting **Customer Feedback** |
   | **List Name** | Customer Feedback |
   | **Title** | Insert **summary** (from Parse JSON) |
   | **Category** | Enter custom value then insert **category** (from Parse JSON) |
   | **OriginalFeedback** | Insert **Your feedback** (from Get response details) |
   | **SubmittedBy** | Insert **Your name** (from Get response details) |
   | **SubmittedEmail** | Insert **Your email** (from Get response details) |

3. Add a **Condition** control below **Create item**:
   - Left value: insert **category** (from Parse JSON)
   - Operator: **is equal to**
   - Right value: `Technical Support`

4. In the **True** branch, add **Send an email (V2) - Office 365 Outlook** to route the issue to IT:

   | Parameter | Value |
   | --- | --- |
   | **To** | Your IT team distribution address (use your own for testing) |
   | **Subject** | `New Technical Support feedback` |
   | **Body** | Enter the **summary** from the Parse JSON action and **Your feedback** from Get response details dynamic content |

   > Leave the **False** branch empty — non-technical feedback is simply recorded in the list.

---

## Task 5: Save, Test, and Verify

1. Select **Save**.

2. Submit a new **Customer Feedback** form response describing a technical problem.

3. Return to Power Automate, open the run, and confirm the prompt returned valid JSON and **Parse JSON** succeeded.

4. Open the **Customer Feedback** list and confirm a new item with a concise **Summary** and the correct **Category**.

5. Confirm the IT team email was sent for the Technical Support response. Submit a non-technical comment (e.g. praise) and confirm it is logged but **no** email is sent.

> **Troubleshooting:** If **Parse JSON** fails, open the run and inspect the raw prompt output — tighten the prompt wording or add a **Compose** to strip surrounding text. If the category never matches, confirm the value casing exactly matches the list Choice options.

> *References: [AI Builder prompts overview](https://learn.microsoft.com/ai-builder/prompts-overview) · [Use prompts in Power Automate](https://learn.microsoft.com/ai-builder/use-a-custom-prompt-in-flow)*

> **Congratulations!** You infused generative AI into a business process — summarising and classifying free text, then routing it automatically.

---

# Lab D: Resilient Flows — Try/Catch/Finally Error Handling

## Lab Introduction

Automation is only valuable if it is trustworthy. A flow that silently fails — or, worse, half-completes — can be more damaging than no automation at all. In this lab you will harden the invoice-processing flow from **Lab B** with a **Try / Catch / Finally** pattern built from **Scope** actions and the **Configure run after** setting. When any step fails, the flow catches the error, records it in a SharePoint list, alerts the team, always runs its cleanup, and reports an accurate run status instead of a misleading "success".

This lab will guide you through the following tasks:

- Structure a flow into **Try**, **Catch**, and **Finally** `Scope` blocks
- Use **Configure run after** so the Catch block runs *only* when the Try block fails
- Capture the failing action and its error message with the `result()` expression
- Log every failure to a **Flow Error Log** list and notify the team by email
- Use **Terminate** and a **retry policy** so runs report correctly and recover from transient faults

> **Pattern learned:** **Try** (do the work) → **Catch** (handle failure) → **Finally** (always clean up). This is the backbone of production-grade, unattended automation.

## Estimated Timing: 35 minutes

## Pre-requisites

- Completion of **Lab B** is recommended — this lab hardens that invoice flow. (You can also apply the pattern to any existing flow of your own.)
- Standard connectors only: **SharePoint** and **Office 365 Outlook**.
- Permission to create a Microsoft list.

## Lab Scenario

Your invoice flow now runs unattended against a shared mailbox. Sooner or later something will break — a corrupt PDF, exhausted AI Builder credits, a renamed list column, or a transient network blip. Right now those failures are invisible until someone notices missing data. Your task is to make the flow fail *loudly and safely*: catch every error, record it where the team can see it, alert a human, always run cleanup, and mark the run as genuinely failed so it is easy to spot and re-run.

## Architecture Overview

```
Try / Catch / Finally Error-Handling Pattern
============================================

Trigger (existing Lab B trigger)
└── Initialize variable: HasError = false

                    ↓

Scope: TRY
└── (the working actions moved here)
    Apply to each → Extract invoice fields → Condition → Create item …

                    ↓  (run after: Try HAS FAILED / IS SKIPPED / HAS TIMED OUT)

Scope: CATCH
├── Filter array: result('Try') where status = 'Failed'
├── Compose: failed action name + error message
├── Set variable: HasError = true
├── SharePoint: Create item  → "Flow Error Log"
└── Office 365 Outlook: Send an email (alert)

                    ↓  (run after: Catch — ALL statuses → always runs)

Scope: FINALLY
└── Cleanup / completion marker

                    ↓

Condition: HasError = true?
└── True → Terminate (Status = Failed, with error message)
```

## Job Skills

By completing this lab, you will gain hands-on experience with the following skills:

| Task | Description | Duration |
|---|---|---|
| Task 1 | Create the **Flow Error Log** Microsoft List | 4 min |
| Task 2 | Add an error flag and wrap the work in a **Try** scope | 6 min |
| Task 3 | Add a **Catch** scope with **Configure run after** | 6 min |
| Task 4 | Capture and log the error inside **Catch** | 10 min |
| Task 5 | Add a **Finally** scope, then **Terminate** with an accurate status | 6 min |
| Task 6 | Test by forcing a failure | 3 min |

---

## Task 1: Create the Flow Error Log List

1. Go to [Microsoft Lists](https://www.microsoft.com/microsoft-365/microsoft-lists) and select **Create new** → **Create from blank** → **List**.

2. Name it **Flow Error Log** then select **Create**.

3. Add the following columns:

   | Column name | Type |
   | --- | --- |
   | Title (default) | Single line of text – reuse for the **flow name** |
   | FailedAction | Single line of text |
   | ErrorMessage | Multiple lines of text |
   | RunId | Single line of text |
   | RunLink | Hyperlink |
   | Timestamp | Date and time |
   | Status | Choice (Caught, Resolved) |

   > A shared error log turns invisible, one-off failures into a visible, auditable backlog the team can triage and mark **Resolved**.

---

## Task 2: Add an Error Flag and Wrap the Work in a Try Scope

1. In Power Automate go to My flows and select the  **Invoice Extraction to List** flow you built in Lab B. Select the ellipsis (**…**) and then **Save As**. Name the new flow **Invoice Extraction (resilient)** and select **Save**. Open the new flow and select **Edit**.

2. Directly **after the trigger** and before any other action, add **Initialize variable**:

   | Parameter | Value |
   | --- | --- |
   | **Name** | `HasError` |
   | **Type** | Boolean |
   | **Value** | `false` |

   > A single boolean flag lets the **Finally** stage decide, at the very end, whether the run should be reported as failed.

3. Below the variable, add a **Scope** control (search **Scope**) and rename it **Try**.

4. Move the existing working actions (the **Apply to each** loop with **Process invoices**, the **Condition**, and the **Create item** actions) **inside** the **Try** scope. In the designer, drag the Apply to each action into the Try scope, or cut and paste them.

   > **What is a Scope?** A `Scope` groups a set of actions so they can be treated as one unit — it succeeds only if every action inside it succeeds, which is exactly what makes Try/Catch possible.

---

## Task 3: Add a Catch Scope with Configure Run After

1. **After** the **Try** scope (not inside it), add another **Scope** control and rename it **Catch**.

2. On the **Catch** scope, select **Settings** and expand the Try scope section.

3. Uncheck **Is successful**, and select all three of:
   - **Has failed**
   - **Is skipped**
   - **Has timed out**

   > **Configure run after** is the heart of the pattern: by default an action runs only when the previous one *is successful*. Flipping it so **Catch** runs after the Try scope **has failed / timed out / is skipped** means Catch executes *only* when something went wrong.

---

## Task 4: Capture and Log the Error Inside Catch

All actions in this task go **inside** the **Catch** scope.

1. Add a **Filter array** action (**Data Operation**) and rename it **Filter failed actions**:

   | Parameter | Value |
   | --- | --- |
   | **From** | Switch to the **fx** editor and enter `union(result('Try'), result('Apply_to_each'))` |
   | **Filter Query** | Switch the Filter array to **advanced mode** (**Edit in advanced mode**) and paste the expression below |

   The **Filter Query** (advanced mode):

   ```
   @and(equals(item()?['status'], 'Failed'), not(equals(item()?['error']?['message'], 'An action failed. No dependent actions succeeded.')))
   ```

   > **Why `union(result('Try'), result('Apply_to_each'))`?** `result()` returns only the **first-level** actions of the named scope — *not* deeper nested actions. `result('Try')` therefore returns just the **Apply to each** loop wrapper, whose error is a generic *"An action failed. No dependent actions succeeded."* The real failure (e.g. an action inside the loop) is a first-level action of **Apply to each**, so you must also call `result('Apply_to_each')` and `union()` the two arrays. Use your loop's real name (spaces become underscores).

   > **Why the second filter condition?** The union now contains *both* the generic loop wrapper **and** the real failing action. The `not(equals(... , 'An action failed. No dependent actions succeeded.'))` clause drops the generic wrapper, so `first(...)` in the next step returns the action that actually broke.

2. Add a **Compose** action and rename it **Error details**. In its input, use the **fx** editor:

   ```
   concat('Action: ', coalesce(first(body('Filter_failed_actions'))?['name'], 'Unknown'),
   ' | Error: ', coalesce(
       first(body('Filter_failed_actions'))?['error']?['message'],
       first(first(body('Filter_failed_actions'))?['outputs'])?['error']?['message'],
       first(body('Filter_failed_actions'))?['outputs']?['body']?['error']?['message'],
       first(body('Filter_failed_actions'))?['code'],
       'No message'))
   ```

   > **Why `coalesce()`?** The exact location of an error message inside a `result()` item varies by connector and by depth. A top-level action exposes `error/message`; an action **inside a loop** nests it under `outputs[0]/error/message` (the second line drills into the first repetition); some connectors put it under `outputs/body/error/message`, and others only return a `code`. `coalesce()` returns the first non-null value, so you capture the *most specific* message available — for example a divide-by-zero returns *"Attempt to divide an integral or decimal value by zero in function 'div'."* instead of the generic loop error.

3. Add a **Set variable** action:

   | Parameter | Value |
   | --- | --- |
   | **Name** | `HasError` |
   | **Value** | `true` |

4. Add **Create item - SharePoint** to log the failure and rename it **Log Error**:

   | Parameter | Value |
   | --- | --- |
   | **Site Address** | Select Enter custom value then enter the URL of the site hosting **Flow Error Log** |
   | **List Name** | Flow Error Log |
   | **Title** | Use the **fx** editor: `workflow()?['tags']?['flowDisplayName']` (or type the flow name) |
   | **FailedAction** | Use the **fx** editor: `first(body('Filter_failed_actions'))?['name']` |
   | **ErrorMessage** | Insert the **Error details** Output (from Compose) |
   | **RunId** | Use the **fx** editor: `workflow()?['run']?['name']` |
   | **RunLink** | Use the **fx** editor and insert the expression below |
   | **Timestamp** | Use the **fx** editor: `utcNow()` |
   | **Status** | `Caught` |

   The **RunLink** expression:

   ```
   concat('https://make.powerautomate.com/environments/',
       workflow()?['tags']?['environmentName'],
       '/flows/', workflow()?['name'],
       '/runs/', workflow()?['run']?['name'])
   ```

   > **Tip:** With **RunLink** stored on every row, anyone triaging the **Flow Error Log** list can jump straight from a logged failure to the exact run in history — no need to open the alert email.

5. Add **Send an email (V2) - Office 365 Outlook** to alert the team and rename it **Email Error**:

   | Parameter | Value |
   | --- | --- |
   | **To** | Your support/ops address (use your own for testing) |
   | **Subject** | `Invoice flow failed — action needed` |
   | **Body** | Switch the **Body** to **</> (code view)** and paste the HTML below so the email shows the error details, the RunId, and a clickable link to the exact run |

   ```html
   <p>The <strong>Invoice Extraction (resilient)</strong> flow failed and was caught by the error handler.</p>
   <p><strong>Details:</strong> @{outputs('Error_details')}</p>
   <p><strong>Run ID:</strong> @{workflow()?['run']?['name']}</p>
   <p><a href="@{concat('https://make.powerautomate.com/environments/', workflow()?['tags']?['environmentName'], '/flows/', workflow()?['name'], '/runs/', workflow()?['run']?['name'])}">Open the failed run</a></p>
   ```

   > **How the link is built:** `workflow()` exposes everything needed to address a single run — `tags/environmentName` (which environment), `name` (the flow's ID), and `run/name` (this specific run's ID). Concatenated into the maker-portal URL, the recipient lands directly on the run that failed instead of hunting through run history. (This is the same link you stored in the **RunLink** column on the **Flow Error Log** list in the previous step.)

   > **Why log *and* email?** The list gives a durable, filterable record; the email gives an immediate nudge. Together they ensure no failure goes unnoticed.

---

## Task 5: Add a Finally Scope and Terminate with an Accurate Status

1. **After** the **Catch** scope, add a third **Scope** control and rename it **Finally**.

2. On the **Finally** scope, select **Settings**, expand the Catch scope, and select **all four** statuses (**Is successful**, **Has failed**, **Is skipped**, **Has timed out**) so Finally **always runs**.

3. Inside **Finally**, add a **Compose** action and rename it **Run complete**. This action marks that the flow reached the end regardless of outcome.

   - Switch **Inputs** to the **fx** editor and enter:

     ```
     concat('Flow finished at ', utcNow(), ' — HasError = ', variables('HasError'))
     ```

   > **Finally always runs** — whether the work succeeded or the Catch handled a failure. Use it for cleanup that must happen every time (releasing locks, closing records, final logging). Because a `Compose` requires an **Inputs** value, always put *something* in the box — even a static line — so the action is valid.

4. **After** the **Finally** scope, add a **Condition** control:
   - Left value: insert the **HasError** variable
   - Operator: **is equal to**
   - Right value: `true`

5. In the **True** branch, add a **Terminate** control (search **Terminate**):

   | Parameter | Value |
   | --- | --- |
   | **Status** | Failed |
   | **Code** | `InvoiceFlowError` |
   | **Message** | Insert the **Error details** Output |

   > **Why Terminate?** Because **Catch** handled the error, the run would otherwise be marked **Succeeded** — hiding the failure. **Terminate** with **Status = Failed** makes the run show as failed in run history (and triggers Power Automate's failure notifications), while your Catch block has already logged and alerted. Leave the **False** branch empty.

6. Select **Save** and **Turn on** the flow. If you renamed the flow, make sure you **Turn off** the old version.

---

## Task 6: Test by Forcing a Failure

1. First confirm the **happy path** still works: send a valid invoice email (subject contains "Invoice") and verify the flow completes and creates the list item as in Lab B.

2. Now **force a failure** to exercise the Catch path. Pick one:
   - Inside the Apply to each, add a Compose action with an invalid expression (for example, `div(1,0)`) to force a failure.
   - Send an invoice with no Vendor in the email. Sample invoice can be found in the Power-Automate-Advanced-Ref folder. 

3. **Save**, then send another invoice email.

4. Verify the resilience behaviour:
   - The run appears in history marked **Failed** (from **Terminate**).
   - A new row appears in the **Flow Error Log** list with the failed action name and error message.
   - The alert email arrives.
   - The **Finally** scope's **Run complete** action ran regardless.

5. **Undo** the deliberate break and **Save**.

> **Troubleshooting:** If **Catch** never runs, re-check its **Configure run after** (it must be set on the **Catch scope itself**, not an action inside it). If `result('Try')` errors, confirm the scope is named exactly `Try` (expressions use underscores for spaces, e.g. `result('Try')`). If the run shows **Succeeded** despite a failure, verify the final **Condition → Terminate** is *outside* the Finally scope.

> *References: [Handle errors and exceptions (coding guidelines)](https://learn.microsoft.com/power-automate/guidance/coding-guidelines/error-handling) · [Catch and handle errors with run after](https://learn.microsoft.com/azure/logic-apps/error-exception-handling) · [Fix flow failures](https://learn.microsoft.com/power-automate/fix-flow-failures)*

> **Congratulations!** You turned a fragile flow into a resilient one — it now catches failures, records them, alerts the team, always cleans up, and reports an honest run status.

---

## Labs B–D Summary

### What You Accomplished

✅ **Lab B**: Extracted structured invoice data with AI Builder's prebuilt invoice model and branched on confidence to keep the automation trustworthy.

✅ **Lab C**: Used an AI Builder Prompt to summarise and classify free-text feedback, then routed technical issues automatically.

✅ **Lab D**: Hardened the invoice flow with a Try/Catch/Finally pattern that catches failures, logs them, alerts the team, always cleans up, and reports an accurate run status.

### Key Concepts You Practised

| Concept | Where it appeared | Learn reference |
|---|---|---|
| Prebuilt document processing (invoices) | Lab B | [Prebuilt invoice processing](https://learn.microsoft.com/ai-builder/prebuilt-invoice-processing) |
| Confidence scores and human-in-the-loop review | Lab B | [AI Builder in flows](https://learn.microsoft.com/ai-builder/use-in-flow-overview) |
| AI Builder Prompts (GPT) with structured output | Lab C | [Prompts overview](https://learn.microsoft.com/ai-builder/prompts-overview) |
| Parse JSON of structured output | Lab C | [Data Operations](https://learn.microsoft.com/power-automate/data-operations) |
| Try/Catch/Finally with Scopes | Lab D | [Error handling guidance](https://learn.microsoft.com/power-automate/guidance/coding-guidelines/error-handling) |
| Configure run after, result(), and Terminate | Lab D | [Catch and handle errors](https://learn.microsoft.com/azure/logic-apps/error-exception-handling) |
| Logging failures and run-history debugging | Lab D | [Fix flow failures](https://learn.microsoft.com/power-automate/fix-flow-failures) |

### Next Steps

- **Add a shared mailbox trigger** so the flow processes a team inbox instead of a personal one.
- **Store the original file**: add **Create file - OneDrive/SharePoint** to archive each invoice alongside its list record.
- **Centralise error handling**: turn the Catch/logging block into a reusable **child flow** that any flow can call.
