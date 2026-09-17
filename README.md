# Centralized Travel Price Comparison System (UiPath)

A desktop automation built **entirely in UiPath Studio** that compares flight and
train fares for a journey and shows exactly three results:

1. **Cheapest flight**
2. **Cheapest train**
3. **Cheapest overall** (flight vs train, with the price difference)

No Python, Java, Node.js, React, backend, or external API server. Just UiPath
workflows, activities, variables/arguments, Excel activities, and browser
automation.

> **DEMO DATA:** The sample fares in this project are **static demo values for
> learning only — they are NOT live prices and are NOT from MakeMyTrip** or any
> real travel provider. See [`Documentation/MAKEMYTRIP_NOTICE.md`](Documentation/MAKEMYTRIP_NOTICE.md).

---

## Quick start

1. Install **UiPath Studio** (free Community Edition), Windows.
2. Download/clone this repo and open **`project.json`** in Studio.
3. Let Studio **restore the packages** (first open, needs internet once).
4. Open **`Main.xaml`** and click **Run**.
5. Press **OK** through the five dialogs to accept the defaults.
6. Read the three result message boxes. 🎉

Full details: [`Documentation/SETUP.md`](Documentation/SETUP.md) ·
Presentation script: [`Documentation/DEMO_GUIDE.md`](Documentation/DEMO_GUIDE.md)

---

## What you will see (with the shipped sample data)

| Output | Result |
|--------|--------|
| Cheapest Flight | **SpiceJet SG-401** — Rs. 4499 (18:40 → 21:35, 2h 55m) |
| Cheapest Train  | **Grand Trunk Express 12615** — Rs. 1985 (19:15 → 06:50, 35h 35m) |
| Cheapest Overall | **Train** — Grand Trunk Express, Rs. 1985, saving **Rs. 2514** vs the cheapest flight |

The comparison is real logic over the Excel data — edit a fare in
`Data/DemoFlights.xlsx` and the winner changes.

---

## DEMO mode vs LIVE mode

| | DEMO mode (default) | LIVE mode (optional) |
|---|---|---|
| Data source | `Data/DemoFlights.xlsx`, `Data/DemoTrains.xlsx` | Local demo website `LiveDemoSite/index.html` |
| Needs internet/browser | No | Chrome + UiPath extension |
| Reliability | Always works, offline | Demonstrates browser automation; extraction step is configured in Studio |
| Purpose | The classroom demonstration | Shows the reusable scraping architecture |

LIVE mode **automatically falls back to DEMO data** if anything fails, so the
three outputs always appear. We deliberately do **not** scrape MakeMyTrip —
[read why](Documentation/MAKEMYTRIP_NOTICE.md).

---

## Project structure

```
CentralizedTravelPriceComparison/
├── Main.xaml                       # Orchestrator (Flowchart + Try/Catch)
├── project.json                    # UiPath project config (Windows-Legacy, VB)
├── Workflows/
│   ├── GetTravelInputs.xaml        # Ask source / destination / date / mode / run-mode
│   ├── ExtractFlightData.xaml      # Demo: read Excel  | Live: invoke SearchFlights
│   ├── ExtractTrainData.xaml       # Demo: read Excel  | Live: invoke SearchTrains
│   ├── SearchFlights.xaml          # LIVE browser automation (open/type/click)
│   ├── SearchTrains.xaml           # LIVE browser automation (open/type/click)
│   ├── FindCheapestFlight.xaml     # Min valid fare over the flights DataTable
│   ├── FindCheapestTrain.xaml      # Min valid fare over the trains DataTable
│   ├── ComparePrices.xaml          # Flight vs train, price difference, edge cases
│   └── DisplayResults.xaml         # The three Message Box outputs
├── Data/
│   ├── DemoFlights.xlsx            # Sample flights (Sheet1, headers)
│   └── DemoTrains.xlsx             # Sample trains  (Sheet1, headers)
├── LiveDemoSite/
│   └── index.html                  # Local demo website for LIVE mode
├── Output/                         # Result files written at run time
├── Documentation/
│   ├── README.md                   # Architecture & workflow reference
│   ├── SETUP.md                    # Install / open / run / troubleshoot
│   ├── DEMO_GUIDE.md               # Presentation script + expected outputs
│   └── MAKEMYTRIP_NOTICE.md        # Compliance investigation
└── Screenshots/                    # Put presentation screenshots here
```

---

## Required UiPath packages

| Package | Used for |
|---------|----------|
| `UiPath.System.Activities` | Input Dialog, Message Box, Assign, If, For Each, Invoke Workflow File, Try Catch, Log Message |
| `UiPath.Excel.Activities` | Workbook Read Range / Write Range |
| `UiPath.UIAutomation.Activities` | Open Browser, Type Into, Click (LIVE mode) |

`project.json` pins reasonable versions; if a version is unavailable in your
feed, install the latest of each via **Manage Packages** (one click). Target
framework is **Windows-Legacy (.NET Framework, VB)** for the widest Studio
compatibility.

---

## How the three outputs are computed

- **Cheapest flight / train:** `FindCheapest*.xaml` loops the DataTable, parses
  the `Fare` column with `Double.TryParse` (skipping blank/invalid fares), and
  keeps the row with the smallest positive fare.
- **Cheapest overall:** `ComparePrices.xaml` compares the two cheapest fares,
  reports the cheaper mode, the service name, the fare, and the **price
  difference**. It handles: both found, only one found, none found, and ties
  (fare-based only — never confuses cheapest with fastest).

---

## Honesty / limitations

- Built as valid UiPath XAML, but **not executed in UiPath Studio in the build
  environment** (no Studio there). XML well-formedness, matching Invoke
  arguments, and the comparison logic were all verified programmatically; a
  minor activity-property tweak may be needed on first open depending on your
  package versions.
- LIVE mode's table-extraction step is intentionally completed in Studio with the
  Extract Table Data wizard (selectors must be captured in Studio).
- Sample fares are illustrative demo data, not real prices.

## License / use

Educational college project. Uses only sample/demo data. Do not point the
automation at any website you are not authorized to automate.
