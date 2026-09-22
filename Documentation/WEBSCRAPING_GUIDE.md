# Web Scraping Guide — REAL live data from Chrome (UiPath Data Scraping)

This shows how to pull **real, live data** from a website through Chrome using
UiPath's **Data Scraping** box (the "web scrape box"). We use
**https://books.toscrape.com** — a real live site made for scraping practice
(real titles + prices, no login, no CAPTCHA, fully permitted).

> ⚠️ Do NOT point this at MakeMyTrip / IRCTC / Goibibo etc. — those forbid
> scraping and use CAPTCHA/anti-bot. See `MAKEMYTRIP_NOTICE.md`. The steps below
> are identical for any site you are allowed to scrape.

---

## 0. One-time setup
1. Install **Google Chrome**.
2. Install the **UiPath Chrome extension**:
   Studio → **Home → Tools → UiPath Extensions → Chrome → Install**.
   Then in Chrome, turn the extension **On** if it asks.

---

## 1. Open the live website in Chrome
Go to **https://books.toscrape.com** in Chrome. You'll see a grid of books, each
with a **title** and a **price** (like £51.77). Keep this tab open.

---

## 2. Use the Data Scraping box (this is the web scraping)
1. In UiPath Studio, open the workflow you want (e.g. create a new one, or open
   `Workflows\SearchFlights.xaml`).
2. Top ribbon **Design → Data Scraping** (the table/spider icon).
3. A wizard opens: *"select the first element"* → switch to Chrome and **click
   the title of the first book** (e.g. *A Light in the Attic*).
4. *"select the second element"* → **click the title of the second book**
   (e.g. *Tipping the Velvet*). UiPath now knows it's a repeating list.
5. A preview grid appears with all the titles. In the little box, name the
   column **`Title`** → click **Next**.
6. It asks **"Extract Correlated Data?"** → click **Yes** (to add the price as a
   second column):
   - Click the **price of the first book** (£51.77), then the **price of the
     second book**. Name this column **`Price`** → **Finish**.
7. It asks about **multiple pages / max results**:
   - For a quick demo, set **Maximum number of results = 20** (one page), or
     leave 0 and click through the "next page" indication to scrape all pages.
8. Choose where to store the result → it creates a **DataTable** variable
   (e.g. `ExtractDataTable`). Studio drops a **Use Application/Browser** (or
   Attach Browser) scope with an **Extract Table Data** activity inside.

▶ **Run the workflow now** — Chrome opens the site and you get a DataTable of
**real, live titles and prices**. That is genuine web scraping from Chrome. 🎉

---

## 3. (Optional) Show the scraped data / find the cheapest
After the Extract Table Data activity, add these (all use activities your Studio
already runs):

**A. Just prove it worked** — add an **Input Dialog** (used here as a message
box) with **Label** set to:
```
"Scraped " + ExtractDataTable.Rows.Count.ToString + " books." + Environment.NewLine +
"First: " + ExtractDataTable.Rows(0)("Title").ToString + " - " + ExtractDataTable.Rows(0)("Price").ToString
```
(Set the Input Dialog's **Result** to any throwaway String variable.)

**B. Find the cheapest book** — add two **Assign** activities:
- `cheapestRow` (System.Data.DataRow) =
```
ExtractDataTable.Select().OrderBy(Function(r) CDbl(r("Price").ToString.Trim.Substring(r("Price").ToString.Trim.IndexOfAny("0123456789".ToCharArray())))).FirstOrDefault()
```
  *(That `Substring(...IndexOfAny(digits))` trick strips the "£" so the price
  becomes a number, e.g. £51.77 → 51.77.)*
- Then an **Input Dialog** with **Label** =
```
"Cheapest book: " + cheapestRow("Title").ToString + "  (" + cheapestRow("Price").ToString + ")"
```

---

## 4. (Optional) Feed it into this project's LIVE mode
The travel project's LIVE mode expects flight/train tables with a `Fare` column,
which books don't have — so keep the **travel comparison on DEMO data**. Use the
books scrape above as your **standalone "real web scraping" demonstration**.
(To scrape a real *travel* source later, it must be one that permits automation;
then point `in_LiveSiteUrl` in `Main.xaml` at it and redo this wizard on its
results table.)

---

## Troubleshooting
| Problem | Fix |
|---------|-----|
| "Data Scraping" does nothing / greyed out | Install the **UiPath Chrome extension** (Step 0) and make sure the Chrome tab is open and focused. |
| Wizard won't pick the element | Click precisely on the **text** (the title text / the price text), not the image or the card border. |
| Price won't convert to a number | Use the `Substring(...IndexOfAny("0123456789"...))` expression above — it removes the "£" symbol. |
| Only got a few rows | In the wizard's "maximum results" step set a bigger number, or enable next-page traversal. |
| Nothing/blocked on a real fare site | That site forbids scraping — use a permitted site (this guide) instead. |
