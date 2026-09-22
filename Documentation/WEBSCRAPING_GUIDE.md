# Web Scraping Guide (UiPath Data Scraping)

This guide shows how to add **real web scraping** to the project using UiPath's
built‑in **Data Scraping** wizard, scraping the included **local demo website**
(`LiveDemoSite/index.html`). This is the standard, correct way to scrape in
UiPath — the selectors are captured by the wizard, so it cannot be pre‑written
in the file.

> You will scrape the flights table (and trains table) from the local website
> into a DataTable. It runs offline and does not touch MakeMyTrip or any live
> site (see `MAKEMYTRIP_NOTICE.md`).

---

## 0. One‑time prerequisites

1. **Google Chrome** installed.
2. **UiPath Chrome extension** installed:
   Studio → **Home → Tools → UiPath Extensions → Chrome** → click **Install**.
   (Then, in Chrome, enable the "UiPath Web Automation" extension if prompted.)
3. The **UiPath.UIAutomation.Activities** package restored (it already is — you
   can see it under *Dependencies* in the project).

---

## 1. Open the local demo website

The file to scrape is in your project at:
```
LiveDemoSite\index.html
```
Right‑click it in Windows → **Open with → Chrome** to see it. It has two clean
tables: **Available Flights** (`id="flightsTable"`) and **Available Trains**
(`id="trainsTable"`). Keep this Chrome tab open.

---

## 2. Scrape the flights table with the wizard

1. In UiPath Studio, open **`Workflows\SearchFlights.xaml`**.
2. On the top ribbon (**Design** tab), click **Data Scraping** (the "Table
   Extraction" / spider‑web icon).
3. The wizard says "select the first element" — go to the Chrome tab and
   **click the first cell** of the flights table (e.g. the word **IndiGo**).
4. It asks for the second element — **click the second row's first cell**
   (e.g. **Air India**). This tells UiPath it's a repeating table.
5. A preview appears showing the columns. Click **Next / Finish** →
   "Extract Correlated Data?" → for a single table click **No**.
6. Studio drops an **Extract Table Data** (a.k.a. Extract Structured Data)
   activity, usually inside a **Use Application/Browser** or **Attach Browser**
   scope, with the output going to a `DataTable` variable (e.g. `ExtractDataTable`).

---

## 3. Send the scraped data to the project

The workflow `SearchFlights.xaml` has an output argument **`out_FlightsTable`**
(type `System.Data.DataTable`). Point the scraping output to it:

- Click the **Extract Table Data** activity → in **Properties**, set
  **DataTable / Output** to **`out_FlightsTable`**
  (or add an **Assign**: `out_FlightsTable = ExtractDataTable`).
- Make sure the scraped column **headers match**: `Airline, FlightNumber,
  DepartureTime, ArrivalTime, Duration, Fare`. In the wizard's preview you can
  rename each column to these exact names (this matters so the "find cheapest"
  step can read the `Fare` column).

Repeat **Steps 2–3** in **`Workflows\SearchTrains.xaml`** for the **trains**
table (`id="trainsTable"`) → output **`out_TrainsTable`**, columns
`TrainName, TrainNumber, DepartureTime, ArrivalTime, Duration, Fare`.

---

## 4. Run it in LIVE mode

1. Run **`Main.xaml`**.
2. On the **Run mode** prompt, type **`Live`** (instead of Demo).
3. The project now calls your scraping workflows, extracts the tables from the
   website, finds the cheapest flight/train, and shows the three results.

> If the scrape returns nothing (site closed, extension missing, etc.), the
> project automatically **falls back to the built‑in demo data**, so the three
> outputs always appear.

---

## 5. (Optional) Point it at a real, permitted site later

To scrape a real website instead of the local demo:

1. Change the **`in_LiveSiteUrl`** value in `Main.xaml` (the "Set Live Site Url"
   Assign) to the site you are **authorised** to automate.
2. Redo the **Data Scraping wizard** on that site's results table (selectors are
   site‑specific).

⚠️ Do **not** scrape MakeMyTrip or other sites whose terms forbid automated
access, or that use CAPTCHA/anti‑bot protection. Use a site you own, a permitted
test site, or an official API/data provider. See `MAKEMYTRIP_NOTICE.md`.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Data Scraping" button greyed out / nothing happens | Install the **UiPath Chrome extension** (Step 0) and make sure a browser page is open and focused. |
| Wizard can't select the element | Click precisely on the **text inside a table cell**, not the border. |
| Columns have wrong names | In the wizard preview (or the Extract activity's *Configure Columns*), rename them to the exact names in Step 3. |
| `Fare` shows text like "Rs. 4499" | Keep it as the plain number (`4899`, `1985`, …) as in the demo site, so the fare comparison can parse it. |
| Live returns 0 rows | The project falls back to demo data automatically — check the browser opened and the extension is enabled. |
