---
audience: Both
skillKeys: [systemadmin-commission-tiers, systemadmin-invoices]
title: Commission tiers and volume-based rules
---

# Commission tiers and volume-based rules

This document explains how commission tiers work, how volume-based rules affect the tier a vendor sits in, how System Admin manages tiers and assignments, how nightly recalculation works, and how commissions connect to invoicing.

## What is a commission tier?

A commission tier is a rule set that defines the commission rate a vendor pays on orders. Tiers are managed by System Admin and applied to vendors based on the commercial arrangement and the vendor's order volume.

Each tier contains:
- A commission rate (percentage of order value taken by the platform).
- Volume thresholds that determine which tier applies based on completed order counts over a rolling period.
- Any special rules or overrides that apply to specific vendor types or commercial agreements.

## How tiered and volume-based pricing works

Volume-based commission tiers mean that vendors who process higher order volumes can qualify for lower commission rates. The platform calculates a rolling completed-order count and compares it against the thresholds defined in each tier. When a vendor crosses a threshold, they move to the corresponding tier at the next recalculation.

For example: a vendor processing fewer than 100 orders per month might sit in the standard tier at 15%. A vendor processing more than 500 orders per month might qualify for a reduced tier at 12%.

The specific thresholds and rates are set by the platform and can be reviewed and adjusted in System Admin.

## Nightly recalculation

Commission tier assignments are recalculated on a nightly schedule (default 02:00 UTC unless a custom calculation time is configured). The recalculation service:
1. Counts completed orders for each vendor over the configured rolling period.
2. Compares the count against tier thresholds.
3. Updates the vendor's assigned tier if they have moved up or down.
4. Caches the result and publishes tier-change events to downstream services.

A manual recalculation trigger is also available from the System Admin tools dashboard when an immediate update is needed outside the normal schedule.

## How System Admin manages commission tiers

### Viewing the commissions dashboard

The commissions dashboard in System Admin gives a platform-wide view of the current commission state across all vendors. Use it to spot vendors with unusual assignments, review recent tier changes, and decide which records need attention.

### Creating a new tier

1. Navigate to Commission tiers from the commissions dashboard.
2. Click to create a new tier.
3. Set the rate, volume threshold, and any applicable rules.
4. Save the tier. It is not automatically assigned to vendors — you must configure assignments separately.

### Updating an existing tier

Open the tier from the Commission tiers list and edit the rate or threshold values. Changes apply at the next recalculation run. If you need the change to apply immediately, trigger a manual recalculation from the tools dashboard.

### Removing a tier

Before removing a tier, confirm that no vendors are currently assigned to it. Removing a tier that is in active use will cause those vendors to fall back to the platform default. Reassign affected vendors first.

### Assigning a tier to a vendor or group

Use the commission configuration and assignments workflow on the commissions dashboard. You can assign a tier to individual vendors or to a group. Once assigned, the tier applies at the next recalculation unless you also trigger an immediate recalculation.

## How commission connects to invoicing

When an order completes, the applicable commission is recorded against the order. Completed orders with unprocessed commission amounts appear in the Uninvoiced dashboard in System Admin. Finance operations use the uninvoiced dashboard to generate invoices for one vendor at a time or for a monthly run across all vendors.

The invoice total reflects the commission amounts from completed orders in the invoice period. If a tier assignment changed mid-period, orders may have different commission rates within the same invoice.

## What to check when a vendor's commission looks wrong

1. Open the vendor's record from the Vendors list and confirm the current tier assignment.
2. Check the commissions dashboard for any recent tier changes.
3. Confirm the last recalculation date and whether the vendor's order volume has changed significantly.
4. If the tier assignment looks stale, trigger a manual recalculation from the tools dashboard.
5. If the rate itself is wrong, check the tier definition in the Commission tiers list.
