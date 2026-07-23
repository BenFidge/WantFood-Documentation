---
audience: VendorAdmin
skillKeys: [vendor-orders, vendor-order-detail]
title: Managing orders — live board and order history
---

# Managing orders — live board and order history

This document covers how to manage live and historical orders in Vendor Admin, including the kanban dashboard, all order actions available to vendors, and what to do when an order is stuck or a customer escalates.

## Orders kanban dashboard

The orders kanban dashboard is the kitchen-focused live view of all active orders for the current vendor or branch context. Orders flow left to right through columns representing their current state:

- Incoming: new orders awaiting vendor acceptance.
- Accepted / In Progress: orders the vendor has accepted and is preparing.
- Ready: orders marked as ready for pickup by a driver.
- Assigned: orders with a driver assigned and awaiting collection.
- Out for delivery: orders collected by the driver and in transit.

Completed and cancelled orders drop off the live board and appear in the All Orders history view.

## Branch context card on the kanban page

The same branch context card shown on the dashboard also appears at the top of the kanban page. It shows the current branch name and the **Taking Orders** and **Scheduled Orders** toggles. You can suspend or resume order acceptance directly from this page without leaving the live orders view. Changes made here are identical to changes made on the dashboard — the state is shared.

## How to accept an order

When a new order appears in the Incoming column, review the order details by clicking the card. Confirm you can fulfil the order and click Accept. The order moves to In Progress. There is a time window for acceptance — if no action is taken within the allowed period, the order may be automatically rejected.

## How to reject an order

If you cannot fulfil an order, click Reject on the order card. Enter a reason if prompted. The customer is notified and any card payment is reversed. Only reject orders you genuinely cannot prepare — repeated rejections affect your vendor rating and customer experience.

## How to edit an order

Some orders can be edited before being marked ready — for example to remove an unavailable item or adjust a quantity. The Edit action only appears on the order card when editing is permitted for that order. Make your changes and save. The customer may be notified of the change depending on the edit type.

## How to mark an order ready for pickup

When the food is prepared and ready for collection, click Mark Ready on the order card. The order moves to the Ready column. Any assigned driver is notified. The customer may also receive a notification.

## How to assign a driver

From the order card in the kanban, click Assign Driver and select a driver from the available pool. Only drivers linked to the current vendor or branch context appear. The driver is notified. See the drivers document for how to manage your driver pool.

## How to unassign a driver

If you need to change the driver before they collect the order, use Unassign Driver on the order card, then reassign a different driver. Unassignment is not available once the driver has marked the order as collected.

## How to cancel an order

Use the Cancel action on an active order card. Cancellation is available up to a certain state in the order lifecycle — it is not available once a driver has collected the order. Cancelled orders appear in the All Orders history view.

## All orders page

The All Orders page shows a broader history of orders beyond the live kanban view. Use it to:
- Review closed, completed, and cancelled orders.
- Search for a specific order by order number, date, or customer details.
- Check the outcome of a specific historical order when a customer or System Admin asks about it.

## What to do when an order is stuck

If an order is not progressing — for example it is sitting in Incoming for too long, or a driver assignment has stalled — check:
1. Is the order still within the acceptance time window? If not, it may need to be handled by System Admin.
2. Has the driver been notified? Check the driver assignment state on the order card.
3. Is there a payment issue? If the order payment has not confirmed, the order may be in a state the vendor cannot move forward from. Contact System Admin with the order number.

## What to do when a customer escalates an order complaint

For complaints about a specific order, open the order from the All Orders page to see its history — acceptance time, driver assignment, delivery outcome, and any cancellation details. Use this information to understand what happened before responding to the customer. If the complaint involves a payment dispute, refund request, or delivery failure that cannot be resolved from Vendor Admin, contact System Admin and provide the order number.
