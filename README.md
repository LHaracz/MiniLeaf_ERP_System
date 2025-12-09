# MiniLeaf_ERP_System
Automated production scheduling, inventory forecasting, and order integration system for a microgreens farm using Google Sheets, Google Apps Script, Google Calendar, and Shopify (via Pickeasy).

---

## Tech Stack

- **Google Sheets** – data model + UI for production planning and inventory
- **Google Apps Script** – automation, email alerts, integrations
- **Google Calendar** – receives orders from Shopify (via Pickeasy app)
- **Shopify (Pickeasy)** – front-end for customers and order scheduling

---

## Main Components

### 1. Production Schedule

**Sheet:** `Production Summary`  
**Key scripts:**

- `productionAlerts.gs`
- `trayPrepForecast.gs`
- `yieldSyncFromHarvestSummary.gs`
- `calendarOrderSync.gs`

**What it does:**

- Sends **daily production emails** with:
  - Seeds to **soak**
  - Seeds to **drain**
  - Trays to **sow**
  - Trays to **move to light**
  - Trays to **harvest**
  - Seeds to **weigh today for tomorrow**
- Calculates per-crop **germination timelines** and milestone dates.
- Pulls **average yield per tray** from the *Harvest Yield Summary* file to estimate total yield.
- Pulls **orders from Google Calendar** (fed by Shopify/Pickeasy) and matches them to the correct **harvest week**, so I don’t have to manually re-enter demand.

---

### 2. Harvest Yield Summary

**File:** `Harvests Yield Summary`  
**Sheet:** `Yield Summary`  
**Key script:**

- `yieldSyncFromHarvestSummary.gs`

**What it does:**

- Harvest data is submitted via **Google Form**, one row per tray.
- The sheet computes:
  - Average yield per microgreen
  - Basic yield stats per variety
- `updateEstimatedYields()` builds a lookup of average yield per microgreen and writes the **Estimated Yield** column back to the Production Schedule.

This turns real harvest data into better future planning.

---

### 3. Resource Ledger & Usage Logging

**File:** `Resource Ledger`  
**Sheets:**

- `Resource Ledger`
- `Resource Usage Log`
- `Cycle Count (Responses)`

**Key scripts:**

- `resourceLedgerUpdateAndAlerts.gs`
- `resourceCycleCountSync.gs`

**What it does:**

- `updateResourceLedgerAndSendAlerts()`:
  - Aggregates usage from `Resource Usage Log` grouped by resource and date.
  - Updates:
    - **Used So Far**
    - **Predicted Reorder Date** (date when running balance crosses the reorder threshold)
  - Sends an **email alert** when remaining quantity falls below a threshold and flags that an alert has been sent.

- `pushLatestCycleCountsToLedger()`:
  - Takes the **latest Google Form cycle count** submitted to `Cycle Count (Responses)`.
  - Maps form columns to the matching `Resource` rows in the ledger.
  - Overwrites `Starting Quantity` with the most recent counted value.

- Custom menu via `onOpen()`:

  - `"MiniLeaf Tools" → "Update Resource Ledger/Send ReOrder Alerts"`
  - `"MiniLeaf Tools" → "Push Latest Cycle Count"`

This behaves like a lightweight **inventory & cycle-count system** on top of Google Sheets.

---

### 4. Master Resource Usage (Cross-Schedule Rollup)

For multiple production spreadsheets (e.g., different weeks/templates), usage is rolled into a single **master log**.

**Sheets (per production file):**

- `Container Usage`
- `Seed Usage`
- `Soil Usage`

**Master file:**

- `Resource Usage Log` (in a central “master” spreadsheet)

**Key scripts:**

- `containerUsageToMaster.gs`
- `seedAndSoilForecastToMaster.gs`

**What they do:**

- `pushContainerUsageToMaster()`:
  - Reads daily container usage and pushes new entries (date, "Container", quantity used, source schedule name) into the master `Resource Usage Log`.
  - Uses a **composite key** (`date_resource_source`) to avoid duplicate inserts.

- `pushSeedAndSoilForecastToMaster()`:
  - Reads seed usage by microgreen and soil usage (in³ → ft³) from the production schedule.
  - Converts units:
    - Seeds: grams → pounds
    - Soil: cubic inches → cubic feet
  - Pushes entries into the master `Resource Usage Log`, again deduping with a composite key.

- Custom menu:

  - `"MiniLeaf Tools" → "Push Container Usage Forecast"`
  - `"MiniLeaf Tools" → "Push Seed & Soil Forecast"`

This lets me run **global inventory analytics** based on all production templates.

---

### 5. Shopify / Pickeasy / Google Calendar Order Sync

Orders placed on my Shopify store are handled by the **Pickeasy** app, which creates events in a Google Calendar.

**Key script:**

- `calendarOrderSync.gs`

**What it does:**

- Reads harvest date from `Production Summary!H1`.
- Builds time windows (e.g., harvest date + 1 to +5 days) to match **Pickup/Delivery windows**.
- For each matching calendar event:
  - Parses the HTML description to clean text.
  - Extracts:
    - Order number
    - Customer name
    - Product lines (e.g. `2 x 6 oz Radish Rambo`)
    - Delivery vs Pickup
    - Fulfillment DOW & time window
  - Canonicalizes product names (e.g. “Radish Rambo” → `Radish`, “Pea Shoots” → `Pea`).
  - Writes structured rows into the `One-Time Orders` sheet with columns:
    - Customer Name
    - Order Number
    - Microgreen Type
    - Quantity (oz)
    - Num Containers (kept separate)
    - Potential Revenue (blank or formula)
    - Delivery Method
    - Fulfillment DOW
    - Fulfillment Time

This turns messy calendar events into a neat **order table** that the production schedule can use for demand planning.

---

## Email Alerts & Automations

The system currently sends:

- **Daily production reminder**
  - Soaks, drains, sows, light moves, harvests
  - Seed weighing prep for tomorrow
  - Orders due this week (from `One-Time Orders`)
- **Tray prep forecast (0–2 day window)**
  - Total trays and required hole/no-hole trays per day
- **Inventory reorder alerts**
  - When any resource drops below its configured threshold

All email addresses are configured at the top of the relevant scripts and should be updated per deployment.

---

## About MiniLeaf

MiniLeaf is the microgreens business I co-founded.  
This system replaced manual planning with an automated workflow that:

- Reduced manual scheduling and data entry
- Tightened inventory control for seeds, soil, containers, etc.
- Connected real customer orders to production constraints
