---
audience: Both
skillKeys: [vendor-orders, vendor-order-detail, vendor-drivers]
title: Driver assignment and delivery management
---

# Driver assignment and delivery management

This document covers how drivers are linked to vendors and branches, how to assign and unassign drivers from live orders, what the delivery dashboard shows from each perspective, driver availability and shift state, and what System Admin sees in order detail for driver history.

## How drivers are linked to a vendor or branch

Drivers are invited by Vendor Admin users from the Drivers list. Once a driver completes their Driver Portal onboarding, they are linked to the vendor or branch that invited them. A driver can be assigned to a specific branch or remain available at the vendor level.

If a driver's branch assignment needs to change, use the assign or unassign branch action on the driver record in the Drivers list. Only drivers linked to the current vendor or branch context appear in the driver selector for order assignment.

## Assigning a driver to a live order

1. Open the order in the Vendor Admin kanban dashboard.
2. Click Assign Driver on the order card.
3. Select an available driver from the list. Only drivers linked to the active context appear.
4. The driver is notified and the order moves to the assigned state.

## Unassigning a driver from an order

If you need to change the driver before collection, use the Unassign Driver action on the order. You can then reassign a different driver. Unassignment is not available once the driver has marked the order as collected.

## What the vendor sees in Vendor Admin

The orders kanban dashboard shows each order's assignment state — whether a driver is assigned, which driver, and whether the order is awaiting pickup or in transit. The vendor can monitor all active orders and their delivery progress from this view.

## What the driver sees in Driver Portal

The driver sees their active deliveries in the Active Deliveries page. Each assigned delivery shows the pickup address (the restaurant), the delivery address (the customer), and the current delivery status. The driver updates status through Driver Portal: collected, out for delivery, delivered, or failed.

## Driver availability and shift state

A driver must have started their shift in Driver Portal before they appear as available for assignment. Drivers who have not started a shift, who are at maximum delivery capacity, or who are marked unavailable will not appear in the driver selector.

Vendor Admin users can see driver availability state from the Drivers list. If an expected driver is not appearing as available, check their shift state in Driver Portal.

## Cross-tenant and multi-branch driver pools

Drivers assigned to a specific branch are prioritised for orders from that branch. If the vendor has multiple branches and a driver is not branch-assigned, they may be available across all of the vendor's locations. The platform prioritises nearby available drivers when routing.

## What happens when no driver is assigned

An order can stay in the ready state without a driver assigned. The vendor is responsible for managing this — they can manually assign a driver, wait for an automatic assignment if that routing mode is enabled, or contact a driver directly. Orders waiting in the ready state without a driver do not progress automatically.

## What System Admin sees for driver assignment

The System Admin order detail page shows the full driver assignment history for an order: which drivers were assigned, when each assignment was made, when each was unassigned, and the final delivery outcome. This is useful when investigating a disputed delivery or a failed delivery outcome.

System Admin cannot directly reassign drivers from the order detail page — that is a Vendor Admin action. System Admin can see the state and use it to guide recovery decisions.
