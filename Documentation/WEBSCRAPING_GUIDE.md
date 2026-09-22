# Web Scraping Guide — get the FLIGHT & TRAIN info from Chrome (UiPath Data Scraping)

This shows how to pull the **flight and train information** off a webpage in
Chrome using UiPath's **Data Scraping** box (the "web scrape box"), and feed it
straight into the price comparison.

**The website you scrape:** the travel site included in this project —
`LiveDemoSite/index.html`. It has an **Available Flights** table and an
**Available Trains** table. You open it in Chrome and scrape both tables.

> ℹ️ **Why this site and not MakeMyTrip?** Real fare sites (MakeMyTrip, IRCTC,
> Goibibo, Yatra) **forbid scraping** and block bots with CAPTCHA — see
> `MAKEMYTRIP_NOTICE.md`. So we scrape our own travel page instead. The fares on
> it are clearly-labelled **DEMO DATA**, but the **web scraping is 100% real** —
> Chrome really opens the page and UiPath really extracts the tables. The exact
> same wizard steps work on any travel site you are *authorised* to automate.

---

## 0. One-time setup
1. Install **Google Chrome**.
2. Install the **UiPath Chrome extension**:
   Studio → **Home → Tools → UiPath Extensions → Chrome → Install**,
   then turn it **On** in Chrome if it asks.
3. In Studio, open **Manage Packages** and make sure
   **UiPath.UIAutomation.Activities** is installed (it is already listed in
   `project.json`). This is the package that provides Data Scraping.

---

## 1. Open the travel site in Chrome
In your extracted project folder, open the folder `LiveDemoSite` and
**double-click `index.html`** — it opens in Chrome and shows:
- a blue **Available Flights** table (Airline, FlightNumber, times, Duration, Fare)
- an **Available Trains** table (TrainName, TrainNumber, times, Duration, Fare)

Keep this tab open.

---

## 2. Scrape the FLIGHTS table
1. In Studio, open **`Workflows\SearchFlights.xaml`** (the box that says
   *"SCRAPE THE FLIGHTS TABLE HERE"*).
2. Top ribbon: **Design → Data Scraping** (the table/spider icon).
3. The wizard says *"select the first element"* → switch to Chrome and **click a
   cell in the Flights table** (e.g. the word **IndiGo**).
4. *"select the second element"* → **click the next row's cell** (e.g.
   **Air India**). UiPath recognises it as an HTML table.
5. A **preview grid** appears with **all 6 columns already filled in**
   (Airline, FlightNumber, DepartureTime, ArrivalTime, Duration, Fare).
   Leave the column names as they are → **Next**.
   *(If it asks "Extract Correlated Data?" you can click No — a real `<table>`
   is captured in one shot. Only div-based lists need the extra clicks.)*
6. **Maximum number of results = 0** (means "all rows") → **Finish**.
7. Studio drops a **Use Application/Browser** (or Attach Browser) scope with an
   **Extract Table Data** activity inside, and makes a DataTable variable
   (e.g. `ExtractDataTable`).
8. **Feed it into the project:** click the **Extract Table Data** activity, find
   its **output** property (the DataTable it produces), and set it to
   **`out_FlightsTable`**. *(Or keep `ExtractDataTable` and add one Assign:
   `out_FlightsTable = ExtractDataTable`.)*

---

## 3. Scrape the TRAINS table (same steps)
1. Open **`Workflows\SearchTrains.xaml`** (*"SCRAPE THE TRAINS TABLE HERE"*).
2. **Design → Data Scraping** → in Chrome scroll to the **Available Trains**
   table.
3. Click a cell (e.g. **Tamil Nadu SF Express**), then the next row
   (**Grand Trunk Express**).
4. Preview shows: TrainName, TrainNumber, DepartureTime, ArrivalTime, Duration,
   Fare → **Next** → Max results **0** → **Finish**.
5. Set the **Extract Table Data** output to **`out_TrainsTable`**.

---

## 4. Run it — the scrape now drives the comparison
1. Open **`Main.xaml`** and click **Run**.
2. In the **run-mode** dialog, type **`Live`** (instead of `Demo`).
   *(`Live` is what tells the project to call SearchFlights / SearchTrains and
   use the scraped tables. `Demo` uses the built-in sample instead.)*
3. Chrome opens the travel site, UiPath scrapes both tables, and the **three
   result pop-ups** (cheapest flight, cheapest train, cheapest overall) are now
   computed from the **scraped data**. 🎉

If a scrape ever returns nothing, the project **automatically falls back to the
built-in sample data**, so the three outputs always appear.

---

## 5. (Optional) Prove the scrape worked on its own
Inside `SearchFlights.xaml`, after the Extract Table Data activity, add an
**Input Dialog** (used as a message box) with **Label** =
```
"Scraped " + out_FlightsTable.Rows.Count.ToString + " flights. Cheapest first row: " + out_FlightsTable.Rows(0)("Airline").ToString + " Rs." + out_FlightsTable.Rows(0)("Fare").ToString
```
(Set its **Result** to any throwaway String variable.) Run just this file with
the **Run File** button to see the scraped count.

---

## Troubleshooting
| Problem | Fix |
|---------|-----|
| "Data Scraping" greyed out / does nothing | Install the **UiPath Chrome extension** (Step 0) and keep the Chrome tab open and focused. |
| Wizard grabbed only one column | Click a **table cell** (the text), not the heading; on a real `<table>` all columns come in together. |
| Only 1–2 rows captured | In the wizard set **Maximum results = 0** (all rows). |
| Columns have different names | Rename them in the preview to exactly **Airline/FlightNumber/DepartureTime/ArrivalTime/Duration/Fare** (flights) and **TrainName/TrainNumber/…/Fare** (trains) so the comparison finds the `Fare` column. |
| Nothing happens on a real fare site | That site forbids scraping/*has CAPTCHA* — use this permitted travel page (or another site you are authorised to automate). |
| `Extract Table Data` won't load | Reinstall **UiPath.UIAutomation.Activities** via Manage Packages, then reopen the file. |
