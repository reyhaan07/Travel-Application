# Architecture & Workflow Reference

Technical reference for how the workflows fit together. For install/run see
[`SETUP.md`](SETUP.md); for the presentation script see
[`DEMO_GUIDE.md`](DEMO_GUIDE.md).

---

## Execution flow

```
Main.xaml  (Sequence → Try/Catch → Flowchart)
   │
   ├─ 1. GetTravelInputs.xaml ......... asks Source / Destination / Date / SearchMode / RunMode
   │
   ├─ 2. Acquire Data
   │      ├─ if Flights/Both → ExtractFlightData.xaml → dt_Flights
   │      └─ if Trains/Both  → ExtractTrainData.xaml  → dt_Trains
   │           (each: DEMO = Workbook Read Range on the .xlsx;
   │                  LIVE = Invoke Search*.xaml, else fall back to DEMO)
   │
   ├─ 3. Find Cheapest
   │      ├─ FindCheapestFlight.xaml → flightFound, flightRow, flightFare
   │      └─ FindCheapestTrain.xaml  → trainFound,  trainRow,  trainFare
   │
   ├─ 4. ComparePrices.xaml .......... cheaperMode/Service/Fare, priceDifference, message
   │
   ├─ 5. DisplayResults.xaml ......... 3 Message Boxes (the required outputs)
   │
   └─ 6. Save Report (Try/Catch) ..... Workbook Write Range → Output/*.xlsx + summary log
```

The whole process is wrapped in a top-level **Try/Catch** so any unexpected
error becomes one clear message box instead of a crash.

---

## Workflow arguments (contract)

### GetTravelInputs.xaml
| Arg | Dir | Type | Notes |
|-----|-----|------|-------|
| out_Source | Out | String | default "Chennai" if left blank |
| out_Destination | Out | String | default "New Delhi" |
| out_TravelDate | Out | String | default = today + 20 days |
| out_SearchMode | Out | String | normalised to `Flights` / `Trains` / `Both` |
| out_RunMode | Out | String | normalised to `Demo` / `Live` |

### ExtractFlightData.xaml / ExtractTrainData.xaml
| Arg | Dir | Type |
|-----|-----|------|
| in_RunMode | In | String |
| in_Source, in_Destination, in_TravelDate | In | String |
| in_DemoFilePath | In | String |
| in_LiveSiteUrl | In | String |
| out_FlightsTable / out_TrainsTable | Out | DataTable |

### SearchFlights.xaml / SearchTrains.xaml (LIVE)
| Arg | Dir | Type |
|-----|-----|------|
| in_Source, in_Destination, in_TravelDate, in_LiveSiteUrl | In | String |
| out_FlightsTable / out_TrainsTable | Out | DataTable |

### FindCheapestFlight.xaml / FindCheapestTrain.xaml
| Arg | Dir | Type |
|-----|-----|------|
| in_FlightsTable / in_TrainsTable | In | DataTable |
| out_Found | Out | Boolean |
| out_Row | Out | DataRow |
| out_Fare | Out | Double |

### ComparePrices.xaml
| Arg | Dir | Type |
|-----|-----|------|
| in_FlightFound, in_TrainFound | In | Boolean |
| in_FlightFare, in_TrainFare | In | Double |
| in_FlightService, in_TrainService | In | String |
| out_CheaperMode, out_CheaperService, out_Message | Out | String |
| out_CheaperFare, out_PriceDifference | Out | Double |

### DisplayResults.xaml
| Arg | Dir | Type |
|-----|-----|------|
| in_RunMode, in_Source, in_Destination, in_TravelDate | In | String |
| in_FlightFound, in_TrainFound | In | Boolean |
| in_FlightRow, in_TrainRow | In | DataRow |
| in_CheaperMode, in_CheaperService, in_ComparisonMessage | In | String |
| in_CheaperFare, in_PriceDifference | In | Double |

---

## Key expressions (VB, UiPath-compatible)

**Parse & validate a fare (skip blanks / non-numbers / non-positive):**
```vb
Double.TryParse(row("Fare").ToString.Trim, currentFare) AndAlso currentFare > 0
```
`Double.TryParse` returns True only for a valid number and writes it into
`currentFare`; the `> 0` guard drops zero/negative fares.

**Keep the smallest fare seen so far:**
```vb
(Not out_Found) OrElse (out_Fare > currentFare)
```
True for the first valid row, or whenever the current row is cheaper.

**Iterate DataTable rows in a typed For Each (`TypeArgument = System.Data.DataRow`):**
```vb
in_FlightsTable.Select()
```
`DataTable.Select()` returns a `DataRow()` array — a clean typed enumerable.

**Build a service label without a null-reference when nothing was found
(`If(...)` short-circuits):**
```vb
If(flightFound AndAlso flightRow IsNot Nothing,
   flightRow("Airline").ToString & " " & flightRow("FlightNumber").ToString,
   "N/A")
```

**Resolve file paths relative to the project folder:**
```vb
System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), "Data", "DemoFlights.xlsx")
```

---

## Data contract (Excel & demo site)

Both the `.xlsx` files (sheet **Sheet1**, first row = headers) and the demo site
tables use these exact column names:

- Flights: `Airline, FlightNumber, DepartureTime, ArrivalTime, Duration, Fare`
- Trains: `TrainName, TrainNumber, DepartureTime, ArrivalTime, Duration, Fare`

`Fare` is a plain number (e.g. `4499`). Keep these names/positions if you edit
the data, or the extraction and comparison will not find the columns.

---

## Design choices

- **Windows-Legacy / VB** target for the broadest UiPath Studio compatibility
  and to use the stable *Workbook* Excel activities (no MS Excel install needed).
- **Invoke Workflow File** for a clean, modular structure (one job per file) that
  matches the required project layout.
- **DataRow passing** between Find/Compare/Display keeps arguments few and avoids
  re-reading data.
- **Demo-first**: DEMO mode has no external dependencies so a demonstration never
  fails; LIVE mode is additive and falls back to DEMO.
