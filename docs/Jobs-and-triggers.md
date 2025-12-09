# Jobs & Triggers

This document explains the automation jobs used in the MiniLeaf system and how they are typically triggered.

---

## Daily Jobs

### 1. Production Alerts

**Function:** `sendProductionAlerts()`  
**Trigger:** Time-based (daily, early morning)

**Purpose:**
- Sends a single daily email detailing:
  - Seeds to soak
  - Seeds to drain
  - Trays to sow
  - Trays to move to light
  - Trays to harvest
  - Seeds to weigh for tomorrow
- Includes weekly order summary on harvest days
- Minimizes mental overhead for daily operations

---

### 2. Tray Prep Forecast

**Function:** `sendTrayPrepForecast_()`  
**Trigger:** Called inside `sendProductionAlerts()`

**Purpose:**
- Looks 0–2 days ahead
- Calculates trays starting soon
- Outputs:
  - Total trays
  - Required hole vs no-hole trays
- Prevents last-minute prep issues

---

## Inventory Jobs

### 3. Resource Ledger Update & Alerts

**Function:** `updateResourceLedgerAndSendAlerts()`  
**Trigger:** Manual or scheduled (daily / weekly)

**Purpose:**
- Aggregates usage
- Predicts reorder timing
- Emails alerts only once per threshold breach
- Prevents stockouts without spam

---

### 4. Cycle Count Sync

**Function:** `pushLatestCycleCountsToLedger()`  
**Trigger:** Manual (menu action)

**Purpose:**
- Pulls latest physical cycle count
- Resets starting quantities in ledger
- Resolves drift between forecasted and real inventory

---

## Order & Forecast Jobs

### 5. Calendar Order Sync

**Function:** `syncCalendarOrders()`  
**Trigger:** Manual or time-based (daily)

**Purpose:**
- Converts calendar events into structured demand
- Eliminates manual order entry
- Maintains consistent naming and quantities

---

### 6. Seed, Soil, & Container Forecast Push

**Functions:**
- `pushSeedAndSoilForecastToMaster()`
- `pushContainerUsageToMaster()`

**Trigger:** Manual (menu action)

**Purpose:**
- Pushes forecasted usage into master inventory log
- Deduplicates entries
- Enables centralized inventory analysis

---

## Menu-Based Triggers

Each spreadsheet exposes a **MiniLeaf Tools** menu via `onOpen()`:

- Update resource ledger
- Push cycle counts
- Push forecasted usage

Menus are intentionally used for:
- Destructive or irreversible actions
- Cross-sheet operations
- Forecast snapshots
