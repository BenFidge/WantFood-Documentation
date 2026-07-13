---
audience: VendorAdmin
skillKeys: [vendor-sales-reports]
title: Sales reports and vendor analytics
---

# Sales reports and vendor analytics

This document covers what sales and analytics data is available to vendors in Vendor Admin, how to read the dashboard summary numbers, how to use date range filters, what affects the figures, and what to do when numbers look incomplete.

## What sales data is available in Vendor Admin

Vendor Admin provides sales and performance data through the vendor dashboard and associated reporting views. Available data includes:

- Order counts by status and time period (today, this week, historical).
- Revenue figures from completed orders in a selected period.
- Top-performing dishes by order count.
- Order acceptance and rejection rates.
- Any promotional activity and its impact (visible via the Vendor Offers list and offer state).

There is no dedicated separate "reports" page in the current platform — sales data is surfaced through the vendor dashboard summary and the All Orders page.

## Dashboard summary numbers

The vendor dashboard shows a live summary for the current trading day:
- Incoming orders: orders received but not yet accepted.
- In progress: orders accepted and being prepared.
- Completed today: orders that reached the delivered or completed state today.

These numbers refresh as orders progress through the day. They reflect the current vendor or branch context — if the numbers look wrong, confirm the correct context is active.

## How to review a period''s orders

To review orders for a specific date range or historical period, navigate to the All Orders page. Use the date filter to select the start and end dates for the period you want to review. The list shows all orders (completed, cancelled, and other states) in that window.

Use the All Orders data to:
- Count completed orders in a period.
- Identify cancellations and their rates.
- Find specific orders for customer queries.
- Understand revenue trends by reviewing completed order totals.

## What affects the numbers

Completed orders are those that reached the delivered or completed state and had their payment confirmed. Orders that were cancelled, rejected, or had a failed delivery do not count as completed revenue.

Commission is deducted from the order revenue before the amount is settled to your Stripe account (for card orders). The figures visible in Vendor Admin reflect gross order value — commission deductions are handled at the invoice level and appear in the finance workflow managed by System Admin.

For cash orders, the customer pays you directly. Cash orders still count toward your order totals and commission calculations.

## Common questions about revenue figures

### Why is my revenue number different from what I expected?

Possible causes:
- Some orders in the period were cancelled or rejected — these are not counted as completed revenue.
- Commission has been deducted from Stripe payouts — the amount in your bank account is less than the gross order total.
- A date range filter is set to a different period than intended.
- You are viewing data for the wrong vendor or branch context.

### Why are some orders missing from a period?

Orders appear in the All Orders list based on when they were placed, not when they were delivered. If you use a date filter, check that the range covers the dates the orders were placed. Orders placed just before midnight may fall in the previous day''s range.

### Why do my dashboard numbers reset at midnight?

The dashboard summary shows today''s activity. At midnight, it resets to begin counting the new day. Historical data remains accessible through the All Orders page with date range filtering.

## What to do when a reporting period looks incomplete

1. Open the All Orders page and apply a date range filter for the period in question.
2. Confirm the correct vendor or branch context is active.
3. Check whether expected orders appear — look for them by approximate date or order value.
4. If orders from a specific period are missing entirely (not just filtered out), contact System Admin. Occasionally, a stats reconciliation job may need to run to correct daily aggregate data.
