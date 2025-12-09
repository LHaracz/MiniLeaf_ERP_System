# Pickeasy & Shopify Integration

This document explains how Shopify orders are routed into the MiniLeaf production system using Pickeasy and Google Calendar.

---

## Integration Overview

MiniLeaf uses **Pickeasy**, a Shopify app, to manage fulfillment scheduling.

Instead of pushing orders directly into Sheets, Pickeasy writes orders to a **Google Calendar**, which acts as the integration layer.

This approach provides:
- Visual scheduling
- Time windows
- A stable, event-based API

---

## Order Flow

Shopify Checkout
→
Pickeasy
→
Google Calendar Event
→
Calendar Sync Script
→
One-Time Orders Sheet

---

## Calendar Event Structure

Each event typically contains:

- Title:
  - Order number
  - Pickup or Delivery keyword
- Description (HTML):
  - Customer Name
  - Product list
    - Quantity
    - Size (oz)
    - Product name
  - Fulfillment notes

Example product lines:
- `2 x 6 oz Radish Rambo`
- `1x 4oz Pea Shoots`

---

## Parsing Strategy

The script:
- Strips HTML safely
- Normalizes whitespace and bullets
- Extracts product lines by section
- Parses:
  - Container count
  - Quantity in ounces
  - Raw product names
- Maps raw product names to **canonical microgreen names** used in production planning

---

## Fulfillment Matching

Orders are only imported if they fall within valid windows:

- Harvest date is read from `Production Summary!H1`
- Fulfillment windows are defined as offsets from harvest
- Prevents orders from being applied to the wrong week

---

## Why Google Calendar?

Google Calendar provides:
- Ordering guarantees
- Easy debugging
- Native Pickeasy integration
- Clear human visibility

It acts as a “message queue” between Shopify and production planning without custom webhooks or servers.

---

## Result

This integration:
- Eliminates manual order entry
- Prevents missed or duplicated orders
- Keeps production planning tightly aligned with real customer demand
