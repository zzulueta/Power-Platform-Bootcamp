---
  title: 'Power Automate Advanced: Hands-On Lab Exercises'
  module: 'Power Automate Advanced'
---

# Power Automate Advanced — Hands-On Lab Exercises

## Lab Introduction

These exercises accompany the Power Automate Advanced training session. Each lab maps to a segment of the session agenda and is designed to be completed independently or in sequence. Labs use Power Automate Desktop (PAD) and AI Builder capabilities that require a Power Automate Premium license.

This set of exercises covers the following labs:

- **Lab A**: RPA — Global Weather Report using Power Automate Desktop *(this lab)*
- **Lab B**: Extract Invoice Data with AI Builder Document Processing *(coming soon)*
- **Lab C**: AI Builder Prompts — Summarise and Classify Text in a Flow *(coming soon)*
- **Lab D**: End-to-End Flow — Email Attachment → AI Extract → Microsoft List *(coming soon)*

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
└── Extract data from web page
    └── ForecastData (DataTable: Day | Conditions | Low | High)

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

5. In the **Actions** panel on the left, expand **Variables**.

6. Double-click **Set variable** to add it to the canvas.

7. Configure the following:

   | Property | Value |
   |---|---|
   | Variable | `Cities` |
   | Value | `%["Paris", "New York City", "Tokyo"]%` |

   > **Note:** In PAD, list literals are written as JSON arrays inside `%` delimiters. This creates a list variable with three text items.

8. Select **Save** to confirm the action.

### Create the Row Index Variable

9. Add a second **Set variable** action below the first.

10. Configure it as follows:

    | Property | Value |
    |---|---|
    | Variable | `RowIndex` |
    | Value | `2` |

    > **Why start at 2?** Row 1 will be reserved for the column headers you are about to write. Data rows start at row 2.

11. Select **Save**.

### Launch Excel and Write Headers

12. In the **Actions** panel, expand **Excel**.

13. Double-click **Launch Excel** to add it to the canvas.

14. Configure the action:

    | Property | Value |
    |---|---|
    | Launch Excel | with a blank document |
    | Make instance visible | On |

15. Select **Save**. PAD stores the Excel instance in a variable called `ExcelInstance` by default.

16. Add a **Write to Excel worksheet** action.

17. Configure it to write the header row:

    | Property | Value |
    |---|---|
    | Excel instance | `%ExcelInstance%` |
    | Value to write | `City` |
    | Write mode | On specified cell |
    | Column | `A` |
    | Row | `1` |

18. Select **Save**.

19. Add four more **Write to Excel worksheet** actions for the remaining headers. Use the same settings but change the column and value each time:

    | Column | Value to write |
    |---|---|
    | `B` | `Day` |
    | `C` | `Conditions` |
    | `D` | `Low` |
    | `E` | `High` |

    > **Tip:** You can right-click any action on the canvas and select **Copy**, then paste and edit to save time repeating similar actions.

20. Press **Ctrl + S** to save your flow.

---

## Task 2: Record the SearchCity Subflow

In this task you will use the PAD **Web Recorder** to capture a single manual browser session on weather.com for the city of Paris. The generated actions will be placed inside the **SearchCity** subflow. You will parameterise them in Task 4.

> **Why record for just one city first?** The recorder captures what you actually do step by step. It is much easier to record one accurate interaction and then replace the hardcoded city name with a variable, rather than trying to build browser actions manually from scratch.

### Switch to the SearchCity Subflow

1. Select the **SearchCity** tab at the top of the flow canvas.

   All actions you record or add in this task will live here.

### Start the Web Recorder

2. In the PAD designer toolbar, select **Recorder** (the icon shows a browser window with a record button).

3. In the **Recorder** dialog, select the ellipsis (...) and select Microsoft Edge.
    > Install the Power Automate extension for Edge if prompted.

4. Select **Record**.

### Record the Browser Interaction

5. In the browser address bar, navigate to:
   ```
   https://weather.com
   ```

6. Wait for the page to load fully.

   > **Note:** A notification popup ("We'd like to send you notifications") may appear. Click **Dismiss** to close it before continuing. The recorder will capture this click — you can delete it from the generated actions afterwards.

7. In the top navigation bar, click the button showing the current city name (e.g., **New York City** with a dropdown arrow). This opens the location search panel.

8. The search box labelled **Search for a location** appears. Click inside it.

9. If the search box contains existing text, press **Ctrl + A** then **Delete** to clear it.

10. Type:
   ```
   Paris
   ```

11. Wait for the dropdown results to appear.

12. Click the result **Paris** from the dropdown list.

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
    Click on Paragraph 'ParisFrance' of web page
    Click on Anchor '10 Day' of web page
    ```

    > **Note:** The exact action names and count may vary depending on how many clicks the recorder captured. If a "Click" action was recorded for the notification dismiss, right-click it and select **Delete** — the flow should not depend on that popup appearing.

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
   | UI element | Select and create a new UI element by clicking **Add UI element**, switching to your browser, and Ctrl + Click the **Daily Forecast** heading text on the page |

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

19. Press **Ctrl + S** to save your flow.

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

   > **Note:** `%CurrentItem%` is set by the **For each** loop in the Main flow. All subflows share variables with Main, so it is available here without any extra steps. DataTable row values are accessed by zero-based column index — adjust the indices if the extraction captured columns in a different order.

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
    > Why remove the p tag prefix? The recorder locks the selector to a specific HTML element type (p), which can fail on different cities or page states. Using [Text*="%CurrentItem%"] matches any element whose text contains the city name, making it far more robust.

26. Select **Save** to close the UI Element editor, then **Save** the action.

27. Press **Ctrl + S** to save your flow.

---

## Task 5: Save the Workbook and Run the Flow

In this task you will add the Excel save and close actions, then run the completed flow.

### Save and Close the Workbook

These actions go **outside and after** the For each loop, so they run once when all cities are complete.

1. Add a **Save Excel** action from the **Excel** category:

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
Launch new Microsoft Edge
Navigate to https://weather.com
Click city button (top nav)
Populate text field: %CurrentItem%
Click link containing: %CurrentItem%
Click "10 Day" tab

── Subflow: ExtractForecast ───────────────────────────────────────────
Wait for web page content: "Daily Forecast" heading
Scroll web page to: "Daily Forecast" heading
Extract data from web page → ForecastData

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

---

# Lab B: Extract Invoice Data with AI Builder Document Processing

> **Coming soon.** This lab will be added in a future revision of this exercise file.
>
> **What you will build:** An automated cloud flow triggered by incoming emails with invoice attachments. AI Builder's pre-built invoice processing model extracts vendor name, invoice number, dates, and amounts. Results are logged to a Microsoft List with a conditional alert for low-confidence extractions.
>
> **Concepts covered:** AI Builder pre-built models, confidence scores, dynamic content from AI actions, Microsoft Lists.
>
> **License required:** AI Builder credits.

---

---

# Lab C: AI Builder Prompts — Summarise and Classify Text in a Flow

> **Coming soon.** This lab will be added in a future revision of this exercise file.
>
> **What you will build:** A flow triggered by Microsoft Forms responses that uses an AI Builder GPT-based prompt to summarise and categorise customer feedback, stores results in a Microsoft List, and routes Technical Support responses to the IT team via email.
>
> **Concepts covered:** AI Builder Prompts, structuring prompt inputs, parsing prompt output, conditional routing.
>
> **License required:** AI Builder credits.

---

---

# Lab D: End-to-End Flow — Email Attachment → AI Extract → Microsoft List

> **Coming soon.** This lab will be added in a future revision of this exercise file.
>
> **What you will build:** An end-to-end flow combining an email trigger, AI Builder document extraction, an AI Builder Prompt for summarisation, structured storage in a Microsoft List, and a Teams channel notification with a link to the new list item.
>
> **Concepts covered:** Multi-action AI flows, chaining AI extraction into a prompt, Teams integration, flow run history debugging.
>
> **License required:** Power Automate Premium + AI Builder credits.
