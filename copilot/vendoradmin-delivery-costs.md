---
audience: VendorAdmin
skillKeys: [vendor-delivery-costs]
title: Delivery costs and pricing configuration
---

# Delivery costs and pricing configuration

This document covers how to view and configure delivery pricing for your restaurant in Vendor Admin, how changes propagate to customer quotes, and what to check when delivery quotes look wrong.

## Delivery costs page

The Delivery Costs page shows the current delivery pricing rules for the active vendor or branch context. Fields shown depend on your configuration and may include:
- Flat delivery fee applied to all orders.
- Distance-based pricing tiers (for example a lower fee for nearby customers and a higher fee for customers further away).
- Minimum order value threshold for free or reduced delivery.
- Any branch-level overrides that differ from the parent vendor defaults.

Always confirm the correct vendor or branch context is active before viewing or editing delivery costs. Each branch can have its own separate delivery pricing.

## How to configure delivery pricing

1. Confirm the correct vendor or branch context is active.
2. Navigate to Delivery Costs from the vendor dashboard.
3. Review the current pricing rules.
4. Update the values to reflect the intended pricing:
   - To change the flat fee, update the fee field.
   - To adjust distance tiers, update the tier boundaries and corresponding fees.
   - To set a free-delivery threshold, set the minimum order value field.
5. Save the changes.

## How changes propagate to customer delivery quotes

Saved delivery cost changes feed into the delivery quoting logic used when a customer reaches checkout. Because WantFood caches vendor data for performance, there is a normal delay between saving changes and the new pricing appearing in customer delivery quotes.

If pricing still looks wrong to customers after saving, allow normal cache propagation time. If the issue persists, contact System Admin and ask them to run the rebuild caches tool for your vendor record.

## Minimum order values

If you set a minimum order value for free delivery, customers whose basket is below the threshold see the standard delivery fee. Customers at or above the threshold see free delivery or the reduced rate. This is shown to customers on the vendor page and at checkout.

## What to check when delivery quotes look wrong for customers

1. Confirm the intended pricing is correctly saved in the Delivery Costs page (open it and read the current values).
2. Confirm the correct context was active when you saved — a change made in the wrong branch context will have updated the wrong branch.
3. Allow normal cache propagation time after saving.
4. If the issue persists after propagation time, contact System Admin to trigger a cache rebuild for your vendor.
5. If delivery quoting is wrong across all vendors (not just yours), it may be a platform-level delivery configuration issue — report it to System Admin.

## What to do when you cannot save delivery cost changes

1. Confirm all required fields are filled in.
2. Check that the values are within any permitted ranges shown on the page.
3. If you see a validation error, address the specific field highlighted.
4. If saving fails with a service error, try again after a short delay. If the problem persists, contact System Admin.
