# MiniLeaf Production & Inventory System – Overview

This project is an automated production planning and inventory management system built for **MiniLeaf**, a microgreens farm.

It replaces manual scheduling, demand entry, and inventory tracking with a tightly integrated workflow using **Google Sheets, Google Apps Script, Google Calendar, and Shopify (via Pickeasy)**.

The system is designed around three core objectives:

1. **Minimize manual planning effort**
2. **Prevent inventory stockouts**
3. **Tie real customer demand directly into production decisions**

---

## System Scope

The system manages the full lifecycle of microgreen production:

- Customer orders placed on Shopify
- Orders routed through Google Calendar via Pickeasy
- Automated order parsing and scheduling
- Crop-specific germination and harvest planning
- Inventory usage logging and forecasting
- Cycle counting and reorder alerts
- Yield learning based on actual harvest data

---

## Core Components

### 1. Production Schedule
- Central planning sheet that determines:
  - When trays are soaked, sown, drained, uncovered, moved to light, and harvested
  - What needs to be done **today** and what needs prep for **tomorrow**
- Sends daily production emails with clear task breakdowns
- Pulls estimated yields from real historical harvest data

### 2. Order Integration (Shopify → Calendar → Sheet)
- Shopify orders are sent to Google Calendar via Pickeasy
- Calendar events are parsed automatically into structured order rows
- Orders are matched to the correct harvest week to enforce demand constraints

### 3. Inventory & Resource Ledger
- Tracks starting quantities, usage, and remaining balance for:
  - Seeds
  - Soil
  - Containers
  - Other production inputs
- Predicts reorder dates based on actual usage rates
- Sends automated email alerts when thresholds are crossed
- Supports cycle counting via Google Forms

### 4. Yield Learning Loop
- Harvest data is collected per tray using Google Forms
- Average yield per microgreen is continuously recalculated
- Estimated yields in the production schedule improve over time

---

## Design Philosophy

- **Lightweight ERP mindset**: use flexible tools (Sheets + Apps Script) instead of heavy software
- **Automation first**: alerts, parsing, forecasting, and syncing are automatic
- **Human-readable outputs**: emails and sheets are optimized for daily farm operations
- **Modular scripts**: each automation task has a single responsibility

---

## Intended Use

This repository serves as:
- A real-world example of operations automation
- A Google Apps Script system design reference
- A portfolio project demonstrating applied analytics, systems thinking, and automation

Some IDs and configurations are specific to MiniLeaf and will need modification for reuse.
