# Data Flow Overview

This document explains how data moves through the MiniLeaf system, from customer order to harvest and inventory adjustment.

---

## High-Level Flow

Shopify
↓
Pickeasy App
↓
Google Calendar
↓
Calendar Order Sync Script
↓
One-Time Orders Sheet
↓
Production Schedule
↓
Seed / Soil / Container Usage
↓
Resource Usage Log
Resource Ledger & Alerts

---

## Step-by-Step Data Flow

### 1. Customer Orders (Shopify → Google Calendar)

- Customers place orders on the MiniLeaf Shopify store
- The Pickeasy app:
  - Assigns fulfillment date and type (Pickup / Delivery)
  - Creates an event on a designated Google Calendar
  - Embeds order details in the calendar event description

---

### 2. Calendar Events → Structured Orders

**Script:** `syncCalendarOrders()`

- Reads harvest date from `Production Summary!H1`
- Defines valid fulfillment windows (e.g. 1–5 days after harvest)
- Pulls calendar events in those windows
- Parses each event to extract:
  - Order number
  - Customer name
  - Product name(s)
  - Quantity in ounces
  - Fulfillment date & time
  - Pickup vs Delivery
- Normalizes product names (e.g. “Radish Rambo” → `Radish`)
- Writes clean, row-based data into `One-Time Orders`

---

### 3. Orders → Production Planning

- The `One-Time Orders` sheet provides real demand by:
  - Microgreen type
  - Quantity
  - Fulfillment date
- This demand informs:
  - Trays to sow
  - Expected harvest yield
  - Container needs

---

### 4. Production Schedule → Resource Usage

- Sowing schedules generate:
  - Seed usage (grams → pounds)
  - Soil usage (cubic inches → cubic feet)
- Harvest schedules generate:
  - Container usage
- These forecasts are written to local usage tabs:
  - `Seed Usage`
  - `Soil Usage`
  - `Container Usage`

---

### 5. Usage → Master Resource Log

**Scripts:**
- `pushSeedAndSoilForecastToMaster()`
- `pushContainerUsageToMaster()`

- Usage rows are pushed into a centralized `Resource Usage Log`
- Each entry includes:
  - Date
  - Resource
  - Quantity used
  - Source production file
- Deduplication is enforced using a composite key: yyyy-mm-dd + Resource + Source

---

### 6. Resource Log → Inventory Ledger

**Script:** `updateResourceLedgerAndSendAlerts()`

- Aggregates usage per resource
- Computes:
- Total used
- Remaining quantity
- Predicted reorder date
- Updates `Resource Ledger`
- Triggers email alerts when thresholds are crossed

---

### 7. Cycle Counts → Ledger Adjustment

**Script:** `pushLatestCycleCountsToLedger()`

- Reads the newest Google Form cycle count
- Overwrites ledger quantities with physically counted values
- Keeps the system grounded in real inventory levels

