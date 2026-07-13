---
audience: SystemAdmin
skillKeys: [systemadmin-invoices]
title: System Admin — orders, support, and investigation
---

# System Admin — orders, support, and investigation

This document covers how to find and investigate orders in System Admin, what the order detail support view exposes, when to open an order for support, available recovery actions, and when to escalate beyond what System Admin can do.

## Orders list

The Orders list shows orders across the platform that may need support, investigation, or financial follow-up. Use it to:
- Search for an order by order number, vendor name, customer details, or date range.
- Filter by status to find orders in problematic states (failed, cancelled, pending too long).
- Identify orders that have been flagged for support review.

Use the filters to narrow results before opening individual records. The list is not intended for routine browsing of all orders — filter first.

## Order detail — what the support view exposes

The order detail page in System Admin gives the full operational and support context for one order. It shows:

- Current order state and the full state transition history with timestamps.
- Vendor acceptance or rejection reason if recorded.
- Driver assignment history — who was assigned, when, and what happened.
- Delivery status and delivery outcome (delivered, failed, issue reported).
- Payment intent state — whether card payment was confirmed, failed, or refunded.
- Commission applicability — whether this order's commission has been recorded.
- Any cancellation or refund state.
- ACS chat thread activity indicator.

Use the state history to understand what actually happened to an order before making any decisions or communicating with a vendor or customer.

## When to open an order for support

Open an order in System Admin when:
- A customer or vendor escalates a complaint about a specific order.
- An order is in an unexpected state (stuck in pending, failed delivery not resolved, payment not confirmed).
- Finance operations need to check whether an order's commission or payment state is correct before generating an invoice.
- An order needs to be reviewed as part of a fraud or abuse investigation.

## Recovery actions available from order detail

System Admin order detail is a read and investigate surface. Direct recovery actions are limited. System Admin can:
- View all state and history without being able to alter the order state directly.
- Use the information to guide a Vendor Admin user to take the correct action.
- Identify whether a refund or payment correction is needed and raise it through the finance workflow.
- Escalate to the platform operations team when no self-service resolution is available.

## Using order detail to explain a situation to a vendor

When a vendor asks "why was my order cancelled?" or "why was the commission different on this order?", open the order detail in System Admin and read the state history. The history contains the timestamps and reasons recorded at each transition. Use this information to provide the vendor with an accurate explanation.

## When to escalate beyond System Admin

Escalate when:
- The state history shows a platform error rather than a user or vendor action.
- A payment intent is stuck and cannot be resolved through normal finance workflows.
- An order is in a state that should be impossible based on the normal lifecycle.
- A data integrity problem is suspected.

In these cases, raise the issue with the platform engineering or operations team and provide the order ID and a summary of what the state history shows.

## Diagnosing common order problems

### Order stuck in pending

The vendor has not accepted or rejected within the expected time. Check whether the vendor is active and online. If the vendor cannot be reached, the order may need to be cancelled through the platform operations team.

### Delivery marked failed but order is disputed

Open the order detail and read the driver assignment and delivery status history. Check whether the driver updated the status correctly. If there is a dispute about whether delivery actually occurred, the delivery status timestamp and any driver notes are the primary evidence.

### Commission not appearing on an order

Check the payment intent state — commission is typically only recorded on orders where payment was confirmed. If payment failed or was refunded, commission may not apply.
