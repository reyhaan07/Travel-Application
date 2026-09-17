# MakeMyTrip Automation - Compliance Investigation

This document answers the project requirement to investigate whether automated
extraction from **MakeMyTrip** is permitted, and explains the design decision
that follows from it.

## Short answer

**This project does NOT scrape MakeMyTrip.** It uses clearly-labelled **DEMO
data** (local Excel files) and an included **local demo website**. The browser
and data-extraction architecture is built to be reusable, so an *authorized*
data source can be plugged in later.

## What was checked

1. **Terms of Use.** Like most online travel aggregators (OTAs), MakeMyTrip's
   Terms of Use reserve the content for personal, non-commercial use and
   **prohibit accessing the site with automated means** (bots, crawlers,
   scrapers) and prohibit copying/harvesting listings without written
   permission. Automated fare extraction therefore is generally **not
   permitted** under those terms.
2. **Technical anti-bot protection.** MakeMyTrip (again, like most OTAs) uses
   dynamic, script-generated pages, rotating element identifiers, rate limiting,
   bot-detection and **CAPTCHA**. This makes unattended scraping unreliable and
   fragile even where one might attempt it.
3. **`robots.txt`.** OTA `robots.txt` files typically disallow crawler access to
   search/listing paths.

> Transparency note: the environment used to build this project could not open
> `makemytrip.com` (outbound access to that domain was blocked), so the exact
> **current** wording of MakeMyTrip's Terms of Use and `robots.txt` could not be
> quoted here. The points above reflect well-established, widely-documented OTA
> policy. **Before pointing any automation at MakeMyTrip, read their current
> Terms of Use and `robots.txt` yourself**, and only proceed with explicit
> written authorization or an official API/partnership.

## Design decision (what this project does instead)

Because unauthorized scraping is off-limits, the project follows the safe path
requested in the brief:

- **DEMO MODE (default):** reads static, clearly-labelled sample fares from
  `Data/DemoFlights.xlsx` and `Data/DemoTrains.xlsx`. Always works offline.
- **LIVE MODE (optional):** drives a **local demo website**
  (`LiveDemoSite/index.html`) that you own and are allowed to automate. The
  website is prominently marked *"DEMO DATA - not live prices, not MakeMyTrip."*
- The browser-automation workflows (`SearchFlights.xaml`, `SearchTrains.xaml`)
  are kept generic so the target URL and the extraction step can later be
  repointed to **an authorized source** (for example, an official API, a data
  provider you have a licence for, or a site whose terms permit automation).

## What this project deliberately does NOT do

- ❌ No scraping of MakeMyTrip or any other live commercial travel site.
- ❌ No CAPTCHA solving or bypassing.
- ❌ No anti-bot / bot-detection evasion.
- ❌ No pretending that sample fares are live prices.

## If you want real, authorized data later

Good compliant options to investigate:
- An **official API** or affiliate/partner programme from a travel provider.
- A **licensed travel data provider** that supplies fares under contract.
- For Indian trains specifically, official/government data sources and their
  documented usage terms.

Swap the `in_LiveSiteUrl` argument and the extraction step in the two
`Search*.xaml` workflows to point at whatever source you are **authorized** to
use.
