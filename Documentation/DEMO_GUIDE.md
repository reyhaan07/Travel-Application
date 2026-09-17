# DEMO GUIDE (for your college presentation)

A simple script you can follow live, plus the exact expected outputs.

---

## 1. One-line description to say out loud

> "This is a UiPath Studio automation that compares flight and train fares for a
> journey and shows the cheapest flight, the cheapest train, and the cheapest
> option overall. It runs in DEMO mode using clearly-labelled sample data, so it
> works instantly and offline."

---

## 2. Run the demo (2 minutes)

1. Open `Main.xaml` in UiPath Studio and click **Run**.
2. Five small dialogs appear. **Just press OK on each** to use the defaults:
   - Source: **Chennai**
   - Destination: **New Delhi**
   - Travel date: a future date (auto-filled)
   - What to search: **Both**
   - Run mode: **Demo**
3. Read out the three result pop-ups as they appear.

> Tip: To show it is interactive, on one dialog type a different city (e.g.
> `Mumbai`) instead of pressing OK. The journey label in the outputs will change.
> (The fares stay the same sample set - they are demo data, not live.)

---

## 3. Expected outputs (with the shipped sample data)

**OUTPUT 1 of 3 : Cheapest Flight**
```
Airline         : SpiceJet
Flight Number   : SG-401
Departure Time  : 18:40
Arrival Time    : 21:35
Duration        : 2h 55m
Fare            : Rs. 4499
```

**OUTPUT 2 of 3 : Cheapest Train**
```
Train Name      : Grand Trunk Express
Train Number    : 12615
Departure Time  : 19:15
Arrival Time    : 06:50
Duration        : 35h 35m
Fare            : Rs. 1985
```

**OUTPUT 3 of 3 : Cheapest Overall**
```
Cheaper Transport Mode : Train
Service Name           : Grand Trunk Express 12615
Fare                   : Rs. 1985
Price Difference       : Rs. 2514 (versus the other option)

The TRAIN is the cheaper option, saving Rs. 2514 compared with the cheapest flight.
```

After the run, open the **Output** folder to show `FlightResults.xlsx` and
`TrainResults.xlsx` (all extracted rows saved with headers).

---

## 4. Change the sample data (optional, impressive)

Open `Data/DemoFlights.xlsx` or `Data/DemoTrains.xlsx` and edit the **Fare**
column. For example, set a flight fare to `1500`. Re-run - the cheapest flight
and the overall winner update automatically. This proves the comparison is real
logic, not hard-coded text.

Columns must stay named exactly:
`Airline, FlightNumber, DepartureTime, ArrivalTime, Duration, Fare`
(trains use `TrainName, TrainNumber, ...`), on a sheet named **Sheet1**.

---

## 5. What DEMO mode vs LIVE mode means

- **DEMO mode (default):** reads the sample Excel files. Deterministic, offline,
  perfect for a classroom. Fares are **sample values, not live prices.**
- **LIVE mode (optional):** opens a browser and drives the included local demo
  website (`LiveDemoSite/index.html`). It demonstrates the browser-automation
  architecture (open site, type source/destination/date, click Search). The
  actual table-scraping step is intentionally left for you to add in Studio (see
  below), because reliable data-scraping selectors must be captured with the
  Studio wizard. Read `MAKEMYTRIP_NOTICE.md` for why we do not scrape MakeMyTrip.

---

## 6. Enabling LIVE mode (advanced / optional)

> DEMO mode is the supported demonstration. Do LIVE only if you want the extra
> browser step. If LIVE fails for any reason, the project automatically falls
> back to DEMO data, so the three outputs still appear.

1. Install **Google Chrome** and the **UiPath Browser Extension for Chrome**
   (Studio → Home → Tools → UiPath Extensions → Chrome).
2. Run `Main.xaml` and on the last dialog type **Live** instead of Demo.
3. Chrome opens the local demo site and the automation fills the form and
   clicks **Search Fares**.
4. To actually scrape the results table, open `Workflows/SearchFlights.xaml`:
   - Delete the yellow "Log Extraction Note" activity.
   - Add an **Extract Table Data** (a.k.a. Data Scraping / Extract Structured
     Data) activity in its place.
   - Use the wizard to indicate the first cell of the **flights** table
     (HTML `id='flightsTable'`) and choose "extract entire table".
   - Set the activity's output DataTable to the argument **`out_FlightsTable`**.
   - Repeat in `Workflows/SearchTrains.xaml` for the **trains** table
     (`id='trainsTable'`) into **`out_TrainsTable`**.
5. Re-run in LIVE mode. The same three outputs now come from the scraped table.

To later point LIVE mode at an **authorized** real source, change the
`in_LiveSiteUrl` value (set in `Main.xaml`) and redo the Extract Table Data
wizard against that site's results table.

---

## 7. Error handling shown in the demo (talking points)

The project handles, without crashing:
- Empty inputs → sensible defaults are substituted.
- Missing/invalid fares → those rows are skipped when finding the cheapest.
- No flights or no trains found → the relevant output says so, and the overall
  comparison still works with whatever is available.
- Missing demo Excel file → a clear logged error; the run continues.
- LIVE/website/browser failure → automatic fallback to DEMO data.
- Any unexpected error → caught by a top-level Try/Catch that shows one clear
  error message instead of a crash.
