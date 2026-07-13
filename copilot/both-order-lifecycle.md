---
audience: Both
skillKeys: [vendor-orders, vendor-order-detail, systemadmin-invoices]
title: Order lifecycle and delivery
---

# Order lifecycle and delivery

This document covers the full order lifecycle from the moment a customer places an order through vendor acceptance, kitchen preparation, driver dispatch, delivery, and completion. It explains who acts at each stage, what the available actions are, and how System Admin investigates an order.

## Order states

An order moves through the following states:

1. Placed — the customer has submitted the order and payment is in progress.
2. Pending — the order is confirmed and waiting for vendor acceptance.
3. Accepted — the vendor has accepted the order and is preparing it.
4. Rejected — the vendor has declined the order. The customer is notified and any charge is reversed.
5. Ready — the vendor has marked the food ready for pickup by a driver.
6. Assigned — a driver has been assigned to collect the order.
7. Collected — the driver has collected the order from the restaurant.
8. Out for delivery — the driver is en route to the customer.
9. Delivered — the driver has marked the order as delivered.
10. Completed — the order is fully closed after delivery confirmation.
11. Cancelled — the order was cancelled before completion by the vendor, customer, or platform.
12. Failed — the delivery encountered a problem and could not complete normally.

## Who acts at each stage

Vendor Admin is the primary surface for kitchen and dispatch-side order work. The vendor accepts, prepares, marks ready, assigns or unassigns drivers, and can cancel orders while they are in the active phases.

Driver Portal is used for delivery execution. The driver picks up the order, updates delivery status at each step, and marks the order delivered or reports a delivery issue.

System Admin provides support, investigation, and recovery context. System Admin users can see the full order record including state history, finance signals, and delivery information, but the normal live workflow is handled by Vendor Admin and Driver Portal.

## Vendor-side order actions

### How to accept an order

When a new order arrives in the Vendor Admin orders kanban dashboard, it appears in the incoming column. Review the order details, confirm you can fulfil it, and click Accept. Accepted orders move to the in-progress column. There is a time window for accepting — if no action is taken within the allowed time the order may be automatically rejected.

### How to reject an order

If you cannot fulfil an order, click Reject. The customer is notified and any payment is reversed. Provide a reason where prompted. Only reject orders you genuinely cannot prepare — repeated rejections affect your vendor rating.

### How to edit an order

In some cases you can edit an order before marking it ready — for example to remove an unavailable item. Use the edit action on the order in the kanban dashboard. Not all order types allow editing; the action only appears when it is permitted.

### How to mark an order ready for pickup

When the food is prepared and ready for collection, click Mark Ready on the order. This signals to any assigned driver and to the customer that the order is awaiting pickup.

### How to assign a driver to an order

From the order in the kanban dashboard, use Assign Driver to select a driver from the available pool. The driver is notified. You can unassign a driver and reassign to a different one if the situation changes before collection.

### How to cancel an order

Use the Cancel action on an active order when the order cannot proceed. Cancellation is available up to a certain state transition; it is not available once the driver has collected. Cancelled orders appear in the All Orders history view.

## Failed, reverted, and declined delivery paths

When a delivery cannot complete normally, the driver marks it failed or reports an issue from Driver Portal. The order moves to a problem state visible in Vendor Admin and System Admin. System Admin can review the delivery detail, contact history, and decide on the appropriate recovery action such as a refund, reorder, or manual closure.

## System Admin order investigation

System Admin users can open any order from the Orders list. The order detail page shows the full state history, all state transitions with timestamps, the vendor's acceptance or rejection reason, driver assignment history, delivery status, and the finance signals associated with the order (payment intent state, refund state, commission applicability).

When a customer or vendor escalates an order issue, use System Admin order detail to understand the current state before taking any recovery action. Check the state history to confirm what actually happened versus what the customer or vendor reports.

## Commission and invoice activity after an order

When an order completes, it becomes uninvoiced activity that feeds into the commission and invoice workflow. System Admin can see this activity in the Uninvoiced dashboard. Completed orders contribute to the vendor's volume for the current commission tier calculation period. Commission tier recalculation happens nightly and updates the vendor's applicable commission rate.

## What to do when an order is stuck in a bad state

1. Open the order in System Admin and read the full state history.
2. Confirm whether the problem is a state issue (wrong transition), a payment issue (intent not confirmed), a delivery issue (driver not updating status), or a data issue.
3. Use the appropriate recovery tool or escalation path based on what you find.
4. If the issue is a stale cache or propagation delay, allow normal background processing time before taking action.
