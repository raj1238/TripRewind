# TripRewind — Product Requirements Document

## 1. Overview

### 1.1 Problem Statement
People have years of travel history buried across hundreds of emails — flight confirmations, hotel bookings, train tickets — scattered across dozens of providers. There's no easy way to see the full picture of your travel life. Existing tools are cloud-based, require manual input, or don't provide rich analytics. Travel is a huge part of people's identity, yet there's no personal "Wrapped" experience for it.

### 1.2 Product Vision
TripRewind is a personal, privacy-first travel analytics tool that builds rich, interactive dashboards from your email history. You grant Claude access to your Gmail, it extracts travel data and feeds it into TripRewind, and TripRewind turns it into a beautiful, nostalgic view of your travel life. Everything runs locally — no data ever leaves your machine.

### 1.3 Goals & Success Metrics
| Goal | Success Metric | Target |
|------|----------------|--------|
| Accept and store structured travel data | All 3 booking types (flights, hotels, trains) ingested and stored without validation errors | 100% of valid ingestion payloads stored and rendered on dashboard |
| Build a complete travel history | All 3 travel types rendered on dashboard with real data | Flights, hotels, and trains all represented with accurate stats |
| Deliver a fun, nostalgic experience | All 4 stat categories (geography, frequency, spending, trends) render with real data; Wrapped generates without errors for any year with 1+ bookings | Dashboard and Wrapped both functional end-to-end |
| Protect user privacy | All data stays local | Zero external API calls with personal data |

### 1.4 Scope
**In Scope:**
- Travel data ingestion from Gmail (via external Claude agent)
- Storage of structured travel data locally
- Interactive web dashboard with charts, maps, and filters
- Wrapped-style yearly travel summary (scroll-through narrative)
- Support for flights, hotels, and trains

**Out of Scope:**
- Non-Gmail email providers (Outlook, Yahoo, IMAP)
- Buses, cabs, and other transport modes (future phase)
- Cloud hosting or multi-user support
- Mobile app or mobile browser optimization
- Social sharing features
- Manual trip entry UI
- Real-time email monitoring / auto-sync

---

## 2. Users & Personas

### 2.1 Target Users
| Persona | Description | Primary Needs |
|---------|-------------|---------------|
| The Traveler (You) | Frequent traveler with years of booking emails in Gmail. Takes multiple trips per year across flights, trains, and hotels. | See a complete, visual picture of travel history for nostalgia and fun |

### 2.2 User Needs
- See all my trips in one place without manual data entry
- Explore my travel history through beautiful, interactive visuals
- Get fun, surprising stats about my travel life (distance, streaks, firsts)
- Generate a yearly "Wrapped" summary I can scroll through
- Know that my data stays on my machine
- Correct or remove bad data when the AI gets something wrong

### 2.3 Usage Context
- **After a new trip:** Re-run ingestion, check that the new trip appears, explore updated stats
- **End of year:** Generate a Wrapped summary for the year
- **Random nostalgia:** Browse the map, explore past trips, look at trends
- **Expected frequency:** A few times per month, with spikes around Wrapped generation and after trips

---

## 3. Functional Requirements

### 3.1 Feature Overview
| ID | Feature | Priority | Description |
|----|---------|----------|-------------|
| F-001 | User Configuration | P0 | Set home country, base currency, and user preferences |
| F-002 | Travel Data Ingestion | P0 | Capture and store structured travel data from Gmail via Claude |
| F-003 | Flight Tracking | P0 | View all flight journeys with origin, destination, airline, dates |
| F-004 | Hotel Tracking | P0 | View all hotel stays with location, check-in/out, hotel name |
| F-005 | Train Tracking | P0 | View all train journeys with stations, operator, dates |
| F-006 | Trip Grouping | P0 | Auto-group related bookings into logical trips |
| F-007 | Interactive Dashboard | P0 | Web dashboard with summary stats, charts, maps, and filters |
| F-008 | Map Visualization (Basic) | P0 | World map showing visited cities and travel routes |
| F-009 | Distance & Geography Stats | P0 | Total km, Earth circumferences, countries, cities, farthest trip |
| F-010 | Frequency & Streak Stats | P1 | Busiest month, longest gap, most-visited city, travel days/year |
| F-011 | Spending Insights | P1 | Total spend, avg cost/trip, most expensive booking, spend by category |
| F-012 | Time-Based Trends | P1 | Year-over-year comparisons, timeline, travel evolution |
| F-013 | Wrapped-Style Summary | P1 | Scroll-through animated yearly travel narrative |
| F-014 | Domestic vs International Split | P1 | Classify and visualize trips as domestic or international |
| F-015 | Deduplication | P1 | Detect and merge duplicate bookings from multiple emails |
| F-016 | Data Management | P1 | Edit, delete, and flag individual records |
| F-017 | Map Visualization (Enhanced) | P1 | Heat map, hover tooltips, color coding, hotel markers |
| F-018 | Data Export | P2 | Export travel data as CSV or JSON |

### 3.2 Feature Dependencies

| Feature | Depends On | Notes |
|---------|-----------|-------|
| F-002 (Ingestion) | F-001 (Config) | Needs base currency for storage |
| F-006 (Trip Grouping) | F-003, F-004, F-005 | Groups bookings from all 3 tracking features |
| F-007 (Dashboard) | F-001 through F-006 | Renders data from all upstream features |
| F-008 (Basic Map) | F-003, F-004, F-005 | Plots cities and routes from all booking types |
| F-009 (Geography Stats) | F-001 (home city), F-008 (geocoding) | Farthest-from-home needs home city config |
| F-010 (Frequency Stats) | F-006 (Trip Grouping) | "Trips per month" needs trip definitions |
| F-011 (Spending) | F-001 (base currency) | Currency conversion needs config |
| F-012 (Trends) | F-006 (Trip Grouping) | Year-over-year trip counts need grouping |
| F-013 (Wrapped) | F-006, F-009 | Uses trip data and distance stats |
| F-014 (Domestic/Intl) | F-001 (home country) | Classification requires home country |
| F-015 (Dedup) | F-002 (Ingestion) | Runs during ingestion |
| F-016 (Data Mgmt) | F-003, F-004, F-005 | Edits records from tracking features |
| F-017 (Enhanced Map) | F-008 (Basic Map) | Extends basic map with advanced features |
| F-018 (Export) | F-003, F-004, F-005, F-006 | Exports all stored and derived data |

### 3.3 Detailed Requirements

---

#### F-001: User Configuration

**Description:**
First-run configuration that sets required preferences used across features. These values are prerequisites for distance calculations, spending aggregation, and (when F-014 is implemented) domestic/international classification.

**User Stories:**
- As a user, I want to set my home country so TripRewind can calculate distance-from-home stats
- As a user, I want to set my base currency so spending totals make sense

**Configuration Fields:**
| Setting | Required? | Default | Notes |
|---------|-----------|---------|-------|
| Home country | Yes | None — must be set | Used for farthest-from-home; also used by F-014 when implemented |
| Home city | No | Capital of home country | Used for distance-from-home calculations |
| Base currency | Yes | INR | Used for aggregating spending across currencies |
| Currency exchange rates | No | Built-in approximate rates | User can override per currency pair with a fixed rate |

**Acceptance Criteria:**
- [ ] User can set home country and base currency before first dashboard use
- [ ] Settings are persisted locally and editable at any time
- [ ] Changing home city recalculates farthest-from-home stat in F-009
- [ ] Changing base currency recalculates spending aggregations in F-011

---

#### F-002: Travel Data Ingestion

**Description:**
TripRewind captures structured travel data extracted from Gmail by a Claude agent. The system accepts flight, hotel, and train booking data and stores it locally for analysis. After ingestion, the user sees a clear summary of what was processed.

**User Stories:**
- As a user, I want my travel history automatically extracted from Gmail so I don't have to enter anything manually
- As a user, I want to re-run ingestion later to capture new trips without losing existing data
- As a user, I want to see a summary of what was ingested so I know if anything was missed

**What Data to Capture:**

**Flights:**
| Field | Required? | Notes |
|-------|-----------|-------|
| Booking reference / PNR | Yes | Primary identifier |
| Airline name | Yes | |
| Flight number | No | |
| Origin city + airport code | Yes | |
| Destination city + airport code | Yes | |
| Departure date & time | Yes (date required, time optional) | |
| Arrival date & time | No | |
| Passenger name | No | |
| Fare / price | No | May not be in email |
| Currency | No | Required if price present |
| Booking source | No | Airline website, OTA name, etc. |
| Cabin class | No | Economy, business, etc. |

**Hotels:**
| Field | Required? | Notes |
|-------|-----------|-------|
| Booking reference | Yes | Primary identifier |
| Hotel name | Yes | |
| City | Yes | |
| Country | Yes | |
| Check-in date | Yes | |
| Check-out date | Yes | |
| Number of nights | Derived | Calculated from dates |
| Guest name | No | |
| Price (per night or total) | No | May not be in email |
| Currency | No | Required if price present |
| Booking source | No | Hotel website, OTA name, etc. |

**Trains:**
| Field | Required? | Notes |
|-------|-----------|-------|
| Booking reference / PNR | Yes | Primary identifier |
| Train operator | Yes | |
| Train number / name | No | |
| Origin station / city | Yes | |
| Destination station / city | Yes | |
| Departure date & time | Yes (date required, time optional) | |
| Arrival date & time | No | |
| Passenger name | No | |
| Class | No | Sleeper, AC, etc. |
| Fare / price | No | |
| Currency | No | Required if price present |
| Booking source | No | |

**Ingestion Summary:**
The ingestion summary has two parts:

1. **Agent-provided metadata** (passed by the Claude agent alongside booking data):
   - Total emails scanned
   - Emails skipped (with reasons — unrecognized format, not travel-related)
   - Extraction errors (emails that looked like bookings but couldn't be parsed)

2. **TripRewind-computed metrics** (calculated by TripRewind during ingestion):
   - Bookings received (broken down by flights, hotels, trains)
   - Bookings stored successfully
   - Bookings rejected (validation failures — missing required fields)
   - Duplicates detected and merged (exact PNR matches)
   - Probable duplicates flagged for review (fuzzy matches)

The Claude agent must include a summary metadata object in its ingestion payload for part 1 to be available.

**Acceptance Criteria:**
- [ ] System accepts and stores flight, hotel, and train booking data
- [ ] System accepts an optional agent summary metadata object alongside booking data
- [ ] Required fields are validated; bookings with missing required fields are rejected with clear feedback
- [ ] Optional fields gracefully default to empty/null when unavailable
- [ ] Multiple bookings can be ingested in one session
- [ ] Re-running ingestion doesn't create duplicates (see F-015)
- [ ] All stored data is queryable by type, date range, city, and other relevant fields
- [ ] After ingestion, the full summary (agent metadata + TripRewind metrics) is viewable on the dashboard
- [ ] If agent metadata is not provided, the summary shows only TripRewind-computed metrics

**Edge Cases:**
- Missing price data: Accept the booking; price-dependent stats simply exclude it
- Missing times (only dates available): Accept; time-dependent features degrade gracefully
- Multi-leg flights (e.g., DEL→DXB→LHR): Each leg is a separate flight record, linked by booking reference
- Unrecognized email format: Handled by Claude agent; reflected in agent-provided metadata
- Agent doesn't send metadata: TripRewind-side summary still works; agent-side fields show "Not available"

---

#### F-003: Flight Tracking

**Description:**
View all flight journeys. Each flight is a single leg (origin → destination). All map visualization is handled by F-008.

**User Stories:**
- As a user, I want to see every flight I've ever taken, with airline, route, and date

**Acceptance Criteria:**
- [ ] All flights displayed in a searchable, sortable list
- [ ] Each flight shows: date, airline, flight number, origin → destination, duration (if times available)
- [ ] Flights can be filtered by year, airline, origin, destination
- [ ] Total flight count displayed prominently

**Edge Cases:**
- One-way vs round-trip: Each leg is separate; UI can group by booking reference
- Connecting flights: Each leg is a separate record
- Cancelled flights: If ingested with a cancelled flag, exclude from stats but show in list with visual indicator

---

#### F-004: Hotel Tracking

**Description:**
View all hotel stays. All map visualization is handled by F-008.

**User Stories:**
- As a user, I want to see every hotel I've stayed at, including where and for how long

**Acceptance Criteria:**
- [ ] All hotel stays displayed in a searchable, sortable list
- [ ] Each stay shows: hotel name, city, check-in/out dates, number of nights
- [ ] Hotels can be filtered by year, city, country
- [ ] Total nights stayed displayed prominently

**Edge Cases:**
- Same hotel booked multiple times: Each stay is a separate record
- Extended stays (30+ nights): Display correctly, don't distort averages

---

#### F-005: Train Tracking

**Description:**
View all train journeys. All map visualization is handled by F-008.

**User Stories:**
- As a user, I want to see every train journey I've taken, with route and operator

**Acceptance Criteria:**
- [ ] All train journeys displayed in a searchable, sortable list
- [ ] Each journey shows: date, operator, train name/number, origin → destination
- [ ] Trains can be filtered by year, operator, origin, destination
- [ ] Total train journey count displayed prominently

**Edge Cases:**
- Multi-leg train journeys (connection at a station): Each leg is a separate record, linked by booking reference
- Waitlisted / RAC tickets (common in India): Ingest as normal; if status is available, display it but include in stats (user traveled regardless)
- Same-day round trips: Two separate records with same date, different directions

---

#### F-006: Trip Grouping

**Description:**
Automatically group related bookings into logical "trips." E.g., a flight to Paris + hotel in Paris on the same dates = one Paris trip. A "trip" is the core unit of measurement across the dashboard — total trips, trips per year, etc. all depend on this grouping.

**User Stories:**
- As a user, I want TripRewind to understand that my flight + hotel in the same city are part of one trip
- As a user, I want "total trips" on my dashboard to reflect actual trips, not individual bookings

**Grouping Rules:**
1. **City matching:** A booking "touches" a city if that city appears as either its origin OR destination (for flights/trains) or its location (for hotels). Two bookings that touch the same city are candidates for grouping.
2. **Date proximity:** Bookings with overlapping or adjacent dates (within 1 calendar day) that touch at least one common city = same trip.
3. **Chain linking:** If booking A groups with booking B, and booking B groups with booking C (even if A and C don't directly match), all three are in the same trip. This handles multi-city itineraries and return flights.
4. **Trip dates:** A trip's start date = earliest departure/check-in across all its bookings; end date = latest arrival/check-out.
5. **Standalone trips:** Bookings that don't match any other booking = standalone single-booking trips.

**Acceptance Criteria:**
- [ ] Auto-group bookings by date proximity and shared cities (origin, destination, or hotel location)
- [ ] Each trip has: destination city or list of cities (for multi-city), start date, end date, list of associated bookings
- [ ] Trip list view showing all grouped trips
- [ ] Individual trip detail view showing all bookings within that trip
- [ ] Handle multi-city trips (flight A→B, B→C, with hotels in B and C = one multi-city trip)
- [ ] Standalone bookings that don't match a group are still shown as single-booking trips

**Edge Cases:**
- **Round trip:** DEL→BOM flight + hotel in BOM + BOM→DEL flight. The return flight's origin (BOM) matches the hotel city, so all three group correctly into one trip.
- **Open-jaw itinerary:** Fly into city A, fly out of city B (e.g., fly DEL→BOM, train BOM→GOA, fly GOA→DEL). Chain linking connects all bookings through shared cities.
- **Back-to-back trips with no gap:** Trip A ends Monday in BOM, trip B starts Tuesday in GOA. These are separate trips — different cities with no chain link (unless a BOM→GOA booking connects them).
- **Hotel city doesn't match airport city exactly:** Fly into EWR (Newark) but hotel in "New York City." City normalization (see F-008 geocoding) maps both to the same canonical city.
- **Overlapping bookings to different cities:** Two hotel bookings in different cities with overlapping dates (e.g., work trip extended with personal trip). If no shared city, these are separate trips.
- **One-way flight with no hotel:** Standalone single-booking trip.

---

#### F-007: Interactive Dashboard

**Description:**
The main interface. A persistent dashboard for exploring your full travel history with charts, maps, and filters. Desktop-first design; mobile browser support is out of scope for v1.

**User Stories:**
- As a user, I want a beautiful home screen that gives me an at-a-glance view of my entire travel life
- As a user, I want to drill down into specific years, cities, or trip types

**Navigation Structure:**
The dashboard uses a sidebar or top-nav with these primary sections:

**P0 sections (available at launch):**
- **Overview** — Summary cards, key stats, geography stats (F-009), at-a-glance view (homepage)
- **Trips** — Grouped trip list with detail views (F-006)
- **Flights** — Flight list and flight-specific stats (F-003)
- **Hotels** — Hotel list and stay stats (F-004)
- **Trains** — Train list and journey stats (F-005)
- **Map** — Interactive map visualization (F-008)
- **Settings** — User configuration (F-001)

**P1 sections (added when implemented):**
- **Stats** — Frequency, spending, trends (F-010, F-011, F-012). Introduced as a nav section when any P1 stat feature ships.
- **Wrapped** — Yearly travel summary generator (F-013)

**Acceptance Criteria:**
- [ ] Dashboard homepage with key summary cards: total trips, total flights, total train journeys, total cities, total countries, total distance, total hotel nights
- [ ] Geography stats (F-009) displayed on the Overview page
- [ ] Global filter by year / date range that updates all dashboard components
- [ ] Filter by travel type (flights, hotels, trains)
- [ ] All charts are interactive (hover for details, click to drill down)
- [ ] Desktop-first responsive layout
- [ ] Empty states with friendly messages when no data matches current filters
- [ ] Clear navigation between all sections (sidebar or top-nav)
- [ ] Last ingestion summary visible on Overview (date, counts) — links to full ingestion report

---

#### F-008: Map Visualization (Basic)

**Description:**
An interactive world map showing all places visited, with flight/train routes drawn as arcs/lines. This is the single consolidated map for the entire application — flight routes, train routes, and hotel locations all render here.

**User Stories:**
- As a user, I want to see a map of everywhere I've been, with lines showing my routes

**City Geocoding:**
- Flights and trains: city coordinates derived from airport/station codes via a static reference dataset
- Hotels: city coordinates derived from city + country name via a static city-to-coordinates lookup
- **City name normalization:** All city names (from flights, hotels, trains) are normalized to a canonical form. E.g., "NYC", "New York", "New York City" all map to the same location. "Newark" (EWR) and "New York" (JFK/LGA) are treated as the same metro area for trip grouping but may have distinct map pins.

**Acceptance Criteria (P0 — Basic Map):**
- [ ] World map with markers for every city visited (from flights, trains, and hotels)
- [ ] Arc lines for flight routes (origin → destination)
- [ ] Lines for train routes
- [ ] Filter map by year
- [ ] Zoom into regions/countries
- [ ] Click on a city marker to see city name and visit count

---

#### F-009: Distance & Geography Stats

**Description:**
Headline stats about distance traveled and geographic reach. Displayed on the Overview page (F-007). Requires home city from F-001 for "farthest from home" calculation.

**User Stories:**
- As a user, I want to know how far I've traveled in total and how many places I've been

**Acceptance Criteria:**
- [ ] Total distance traveled (km and miles)
- [ ] Number of times around the Earth (distance ÷ 40,075 km)
- [ ] Total countries visited (unique count)
- [ ] Total cities visited (unique count)
- [ ] Farthest single journey from home city (requires F-001 home city config)
- [ ] Northernmost, southernmost, easternmost, westernmost points visited

---

#### F-010: Frequency & Streak Stats

**Description:**
Stats about travel frequency, patterns, and streaks.

**Definition — Travel Day:** A "travel day" is any calendar day where the user has an active booking: in transit on a flight or train, OR checked into a hotel (all days from check-in to check-out inclusive). A same-day round trip counts as 1 travel day.

**User Stories:**
- As a user, I want to know my busiest travel periods and interesting patterns

**Acceptance Criteria:**
- [ ] Busiest travel month — the calendar month with the most departures across all years
- [ ] Busiest travel year — the year with the most trips
- [ ] Longest streak of consecutive travel days
- [ ] Longest gap between trips (days between end of one trip and start of the next)
- [ ] Most visited city (with visit count)
- [ ] Most used airline (by number of flights)
- [ ] Most stayed hotel chain (if data allows grouping by chain name)
- [ ] Total travel days per year (bar chart)
- [ ] Average trips per month

---

#### F-011: Spending Insights

**Description:**
Analytics on travel spending patterns. Only shown when price data is available.

**User Stories:**
- As a user, I want to understand how much I've spent on travel and where the money went

**Currency Handling:**
- Each booking stores its original currency and amount
- For aggregated totals (total spend, averages, comparisons), amounts are converted to the base currency configured in F-001
- Conversion uses fixed exchange rates — either built-in approximate rates or user-configured overrides in F-001
- Individual bookings always display their original currency

**Acceptance Criteria:**
- [ ] Total amount spent on travel (in base currency)
- [ ] Spending breakdown by category (flights vs hotels vs trains) — pie/donut chart
- [ ] Average cost per trip (in base currency)
- [ ] Most expensive single booking (shown in original currency)
- [ ] Spending trend over years (line chart, in base currency)
- [ ] Spending by city/destination
- [ ] Clear indication when price data is missing for certain bookings — show "N/A" and exclude from calculations
- [ ] Label on aggregated stats: "Converted to [base currency] using fixed rates"

**Edge Cases:**
- Missing prices: Stats only use bookings with available price data; clearly label "based on X of Y bookings with price data"
- All prices missing: Hide the spending section entirely with a message: "No price data available"

---

#### F-012: Time-Based Trends

**Description:**
Visualize how travel patterns have evolved over time.

**User Stories:**
- As a user, I want to see how my travel has changed year over year

**Acceptance Criteria:**
- [ ] Timeline view: chronological visualization of all trips
- [ ] Year-over-year comparison charts (trips, distance, cities, spending)
- [ ] First-ever trip highlighted
- [ ] Most recent trip highlighted
- [ ] Monthly heatmap (which months you travel most across all years)
- [ ] Day-of-week distribution (which days you typically depart)
- [ ] Travel frequency trend line (increasing, decreasing, steady)

---

#### F-013: Wrapped-Style Summary

**Description:**
A Spotify Wrapped-inspired, scroll-through narrative summarizing a travel year (or all-time). Card-by-card reveals with animations.

**User Stories:**
- As a user, I want a fun, story-like recap of my travel year that I can scroll through
- As a user, I want to generate a Wrapped for any specific year or for all-time

**Acceptance Criteria:**
- [ ] Select a year (or "all-time") to generate a Wrapped summary
- [ ] Card-by-card scroll-through format with smooth transitions/animations
- [ ] Cards include:
  - Total trips taken
  - Total flights + total train journeys
  - New cities discovered (visited for the first time this year)
  - Top destination (city with the most visits this year)
  - Total distance with fun comparisons (see below)
  - First trip of the year
  - Last trip of the year
  - Spending highlights (if price data available)
- [ ] Summary card at the end with all key stats
- [ ] Works on desktop browsers

**Fun Comparisons (used in Wrapped cards and optionally on dashboard):**
- "You traveled X km — that's Y times around the Earth!"
- "You flew the distance to the Moon!" (if total distance > 384,400 km)
- "You visited X countries — that's Y% of all 193 UN-recognized nations!"
- "You spent Z days traveling — that's like binge-watching all of Breaking Bad N times!"
- "You took X flights — that's one every Y days!"
- "The average person visits ~20 cities in their lifetime — you've been to X!"
- "Your longest trip was X days — basically a mini-sabbatical!"

**Edge Cases:**
- Year with no travel: Show a "You stayed home this year" card
- Year with only 1 trip: Still generate a meaningful Wrapped with available data
- Comparison doesn't apply (e.g., distance too short for Moon): Skip that comparison, use the next applicable one

---

#### F-014: Domestic vs International Classification

**Description:**
Classify each trip as domestic or international based on the home country configured in F-001.

**User Stories:**
- As a user, I want to see the split between my domestic and international travel

**Classification Rule:**
- A flight or train is **domestic** if both origin and destination are in the home country
- A flight or train is **international** if either origin or destination is outside the home country
- A trip is **domestic** if all its bookings are domestic; otherwise it's **international**

**Acceptance Criteria:**
- [ ] Uses home country from F-001 configuration
- [ ] Each flight/train auto-classified as domestic or international per the rule above
- [ ] Each trip (F-006) classified based on its constituent bookings
- [ ] Dashboard card showing domestic vs international ratio (trip count and percentage)
- [ ] Filter dashboard by domestic/international
- [ ] Visual breakdown (pie chart or bar chart)
- [ ] Changing home country in F-001 recalculates all classifications

---

#### F-015: Deduplication

**Description:**
Detect and handle duplicate bookings that may arise from multiple confirmation emails for the same trip (booking confirmation + e-ticket + check-in reminder, etc.).

**User Stories:**
- As a user, I want each trip to appear only once even if I received multiple emails about it

**Deduplication Strategy:**
1. **Exact match:** Same booking reference / PNR + same transport type → definite duplicate. Merge automatically.
2. **Probable match (no booking reference):** Same date + same origin + same destination + same transport type, but no matching PNR → flag as probable duplicate for user review. Do NOT auto-merge.
3. **Conflict resolution (for exact matches):** When two records for the same booking have different field values, keep the record with the most populated fields. If equal, keep the most recently ingested version.

**Acceptance Criteria:**
- [ ] Detect and auto-merge duplicates by exact booking reference match
- [ ] Detect probable duplicates by matching date + route + type when no booking reference matches; flag for user review (visible in F-016 Data Management)
- [ ] Re-ingesting the same data doesn't create duplicates
- [ ] When conflicting data exists for the same booking (exact match), keep the most complete version (most populated fields), then most recent
- [ ] Surface counts in the ingestion summary (F-002): exact duplicates merged, probable duplicates flagged

---

#### F-016: Data Management

**Description:**
Allow the user to view, edit, delete, and flag individual booking records. Since the Claude agent's extraction won't be perfect, users need a way to correct bad data. Also surfaces probable duplicates flagged by F-015 for user review.

**User Stories:**
- As a user, I want to fix incorrect data when the AI gets a booking wrong (wrong city, wrong date, wrong price)
- As a user, I want to delete a record that doesn't belong (e.g., someone else's booking forwarded to me)
- As a user, I want to flag a record as "needs review" if I'm not sure it's correct
- As a user, I want to review probable duplicates and decide whether to merge or keep them separate

**Acceptance Criteria:**
- [ ] View full details of any individual booking record
- [ ] Edit any field of a booking record (inline or via edit form)
- [ ] Delete a booking record (with confirmation prompt)
- [ ] Flag a record as "needs review" (visual indicator on the record)
- [ ] Filter views to show only flagged records
- [ ] View probable duplicates flagged by F-015; merge or dismiss each pair
- [ ] Edits and deletions are reflected across all dashboard stats and visualizations on the next page load (no manual cache-clearing or re-ingestion required)

---

#### F-017: Map Visualization (Enhanced)

**Description:**
Extends the basic map (F-008) with richer interactivity and visual modes.

**User Stories:**
- As a user, I want to see detailed information when I hover over cities and routes on the map
- As a user, I want to see which cities I visit most through a heat map

**Acceptance Criteria:**
- [ ] Color coding routes by transport type (flights vs trains — different colors)
- [ ] Hotel locations shown as distinct markers (city-level placement, different icon from flight/train cities)
- [ ] Hover on a city to see: visit count, first visit date, last visit date
- [ ] Hover on a route to see: frequency, dates traveled
- [ ] Filter map by transport type (flights only, trains only, all)
- [ ] Heat map mode — cities colored by visit frequency

---

#### F-018: Data Export

**Description:**
Export stored travel data for backup or external use.

**User Stories:**
- As a user, I want to export my travel data so I have a backup or can use it elsewhere

**Acceptance Criteria:**
- [ ] Export all travel data as CSV (one file per type: flights, hotels, trains)
- [ ] Export all travel data as a single JSON file
- [ ] Export includes all fields, including derived fields (trip grouping, and domestic/intl classification if F-014 is implemented)
- [ ] Export is downloadable from the dashboard settings/data section

---

## 4. User Flows

### 4.1 First-Time Setup
1. User starts TripRewind locally
2. User opens the dashboard in their browser
3. Dashboard prompts for initial configuration: home country, home city, base currency
4. User saves configuration
5. Dashboard shows empty state with instructions to run the Claude agent for ingestion

### 4.2 Data Ingestion
1. User runs the Claude agent (via Claude Code / Cowork) with Gmail access
2. Claude scans Gmail for travel-related emails (airline confirmations, hotel bookings, train tickets)
3. Claude extracts structured data from each email
4. Claude sends extracted booking data + agent summary metadata to TripRewind
5. TripRewind validates, deduplicates (exact matches), flags probable duplicates, and stores the data
6. Ingestion summary is generated (agent metadata + TripRewind metrics)
7. User opens (or refreshes) the TripRewind dashboard
8. Dashboard displays the full travel history; ingestion summary is visible on Overview

### 4.3 Exploring the Dashboard
1. User opens the dashboard
2. Overview page shows summary cards: total trips, cities, countries, distance, flights, trains, hotel nights, plus geography stats
3. User navigates to Map to explore routes and cities
4. User uses global year filter to narrow to a specific time period
5. User navigates to Trips, clicks into a specific trip to see all associated bookings
6. User explores Stats sections (when P1 features are available): frequency, spending, trends

### 4.4 Generating a Wrapped Summary
1. User navigates to the "Wrapped" section via sidebar/nav
2. User selects a year (or "all-time")
3. TripRewind generates personalized Wrapped cards from stored data
4. User scrolls through the card-by-card narrative
5. Final summary card displays all key stats

### 4.5 Adding New Travel Data (Incremental)
1. User takes a new trip
2. User re-runs the Claude agent (or runs it for a specific date range)
3. Claude finds new booking emails and sends them to TripRewind
4. TripRewind deduplicates and adds only new bookings
5. Dashboard reflects updated data on next visit; ingestion summary shows what's new

### 4.6 Correcting Bad Data
1. User notices an incorrect record on the dashboard (wrong city, wrong date, etc.)
2. User navigates to the record detail view
3. User edits the incorrect fields, or deletes the record entirely
4. Dashboard stats and visualizations update on next page load

### 4.7 Reviewing Probable Duplicates
1. After ingestion, user sees "X probable duplicates flagged" in the ingestion summary
2. User navigates to Data Management, filters to flagged probable duplicates
3. For each pair, user reviews the records side by side
4. User chooses to merge (keep one) or dismiss (keep both as separate bookings)
5. Merged or dismissed records are reflected on next page load

---

## 5. Edge Cases & Error Handling

| Scenario | Expected Behavior |
|----------|-------------------|
| Email with incomplete data (no price, no time) | Accept what's available, leave missing fields empty |
| Multi-leg flight (e.g., DEL→DXB→LHR) | Each leg is a separate flight, linked by booking reference |
| Multi-leg train journey (connection at a station) | Each leg is a separate record, linked by booking reference |
| Booking cancellation email | Mark as cancelled; exclude from stats but visible in list |
| Same city, different airports (e.g., LHR vs LGW) | Treat as same city for city-count and map stats |
| Same city, different name spellings (NYC vs New York) | Normalize to canonical city name via reference dataset |
| Airport city ≠ hotel city name (EWR vs "New York") | Normalize to same metro area for trip grouping |
| Multiple currencies across bookings | Store original currency; convert to base currency (F-001) using fixed rates for aggregation |
| No travel data for a selected year | Show empty state with friendly message |
| Very old emails (10+ years) | Handle gracefully; no date range limitation |
| Exact duplicate bookings from multiple emails | Auto-merge by PNR match (F-015) |
| Probable duplicate bookings (no PNR, same route+date) | Flag for user review, do not auto-merge (F-015) |
| Unrecognized email format | Handled by Claude agent; reflected in agent-provided metadata if available |
| Waitlisted / RAC train tickets | Ingest as normal; display status if available |
| Home country not configured | Block farthest-from-home stat; prompt user to configure |
| All price data missing | Hide spending section with message "No price data available" |
| AI extracts wrong data | User corrects via F-016 Data Management |
| Two real trips on same day to same city | Probable duplicate flag lets user keep both |
| Agent doesn't send summary metadata | TripRewind-side metrics still display; agent-side fields show "Not available" |

---

## 6. Open Questions & Assumptions

### 6.1 Open Questions
- [ ] Should the Claude agent prompt/instructions be documented as part of the TripRewind repo?
- [ ] Should Trip Grouping (F-006) support manual override — letting the user split or merge trips?

### 6.2 Assumptions
- User has a Gmail account with travel booking emails
- Claude (via Claude Code or Cowork) has MCP-based access to Gmail
- The Claude agent handles all email parsing and extraction; TripRewind only receives clean, structured data
- The Claude agent is expected to include a summary metadata object in its ingestion payload (emails scanned, skipped, errors)
- Everything runs locally — the user opens the dashboard in a local browser
- Airport/station-to-city mapping and distance calculations use a static reference dataset (airport codes → coordinates)
- Hotel city geocoding uses a static city-to-coordinates lookup with name normalization
- All data stays local — no external API calls with personal data
- User manages their own local backups; data export (F-018) provides the mechanism

---

### Revision History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-22 | Product Owner | Initial draft |
| 1.1 | 2026-02-22 | Product Owner | Applied review #1 fixes: added User Configuration (F-001), promoted Trip Grouping to P0, added ingestion summary, added Data Management (F-016) and Data Export (F-017), resolved home location dependency, fixed success metrics, consolidated map into F-008, clarified dedup logic, added navigation model, expanded fun comparisons, added train edge cases, clarified currency handling and mobile scope |
| 1.2 | 2026-02-22 | Product Owner | Applied review #2 fixes: rewrote trip grouping rules to handle return flights (origin OR destination matching), added trip grouping edge cases, split map into Basic (P0) and Enhanced (P1), removed P0/P1 cross-dependency for domestic/intl stats, split ingestion summary into agent-provided vs TripRewind-computed, changed fuzzy dedup from auto-merge to flag-for-review, reframed success metrics to TripRewind's scope, added feature dependency table, defined "travel day", added duplicate review user flow, fixed fun comparisons, clarified F-016 update timing, restructured nav for P0 reality |
