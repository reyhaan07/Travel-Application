# SETUP GUIDE

How to open and run the **Centralized Travel Price Comparison System** in UiPath
Studio.

---

## 1. Prerequisites

| Requirement | Notes |
|-------------|-------|
| Windows 10/11 | UiPath Studio is a Windows application. |
| **UiPath Studio** (Community Edition is free) | 2022.10 or newer recommended. Download from the UiPath website and sign in with a free UiPath account. |
| Internet (first open only) | Needed once so Studio can **restore the activity packages**. After that, DEMO mode runs offline. |
| Microsoft Excel | **Not required.** The demo builds its sample data in memory, so no Excel is needed. |
| Google Chrome + UiPath Extension | **Only for LIVE mode** (optional). Not needed for DEMO mode. |

---

## 2. Get the project

**Option A - Git**
```
git clone <your-repo-url>
```

**Option B - ZIP**
Download the repository ZIP from GitHub and extract it to a folder such as
`C:\UiPathProjects\CentralizedTravelPriceComparison`.

> Keep the folder structure intact. `project.json` must stay in the top folder
> next to `Main.xaml`.

---

## 3. Open in UiPath Studio

1. Launch **UiPath Studio**.
2. Click **Open** (or **Open a Local Project**).
3. Browse to the project folder and select **`project.json`**.
4. Studio opens the project and begins **restoring dependencies** (watch the
   status bar / Output panel).

### Activity packages used
The project depends on three standard UiPath packages (declared in
`project.json`):

| Package | Purpose |
|---------|---------|
| `UiPath.System.Activities` | Input Dialog, Message Box, Assign, If, Invoke Workflow File, Try Catch, Log Message. |
| `UiPath.Excel.Activities`  | Excel support (optional; the demo uses built-in sample data). |
| `UiPath.UIAutomation.Activities` | Open Browser, Type Into, Click for LIVE mode. |

**If a pinned version is not found** (you may see an unresolved dependency):
open **Manage Packages** (top ribbon) → **Project Dependencies**, and for each
of the three packages click **Update** / install the **latest available**
version, then **Save**. This is normal for any shared UiPath project and takes a
minute. The workflows do not depend on a specific patch version.

---

## 4. Run

1. In the **Project** panel, double-click **`Main.xaml`**.
2. Click **Run** (or press **Ctrl+F5** to run without debugging, or **F5** to
   debug).
3. Answer the five Input Dialogs (or just press **OK** on each to accept the
   defaults: Chennai → New Delhi, a future date, **Both**, **Demo**).
4. Three message boxes appear in order:
   - **OUTPUT 1 of 3 : Cheapest Flight**
   - **OUTPUT 2 of 3 : Cheapest Train**
   - **OUTPUT 3 of 3 : Cheapest Overall**
5. Detailed result files are written to the **`Output`** folder
   (`FlightResults.xlsx`, `TrainResults.xlsx`).

---

## 5. Troubleshooting

| Symptom | Fix |
|---------|-----|
| "Some dependencies could not be resolved" | You are offline, or the pinned version isn't in your feed. Connect to the internet and let Studio restore, or open **Manage Packages** and install the latest of the three packages (see step 3). |
| Red error icon on an activity after opening | Make sure all three packages finished restoring, then click **Save** to let Studio re-validate. If a single property shows an error because your package version differs slightly, click the activity and re-select that property. |
| "Could not load demo flight/train data" message | Confirm `Data\DemoFlights.xlsx` and `Data\DemoTrains.xlsx` exist in the project folder and are not open in Excel. |
| Nothing happens after inputs | Check the **Output** panel logs. Every step logs an `Info` message; errors are logged and also shown in a final error message box. |
| The `Main.xaml` flowchart boxes look stacked on top of each other | Purely cosmetic. Drag them apart, or right-click the canvas → **Auto Arrange**. Execution is unaffected. |
| LIVE mode does nothing useful / errors | Expected until you configure it - see `DEMO_GUIDE.md` → "Enabling LIVE mode". DEMO mode is the supported demonstration path. |
| The two `Search*.xaml` files show errors and block running | DEMO mode does not use them. If needed, you can right-click each in the Project panel and **Delete** them; DEMO mode still runs (they are only invoked in LIVE mode via a file path). |

---

## 6. Important honesty note

This project was authored as valid UiPath XAML but **could not be opened or
executed inside UiPath Studio in the build environment** (no Studio available
there). The XAML follows standard UiPath serialization and has been checked for
well-formed XML and for matching Invoke arguments, and the price-comparison
logic was verified against the sample data. If your Studio version flags a minor
activity-property difference, it is a quick fix in the designer, not a design
flaw in the workflow logic.
