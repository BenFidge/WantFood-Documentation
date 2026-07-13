---
audience: SystemAdmin
skillKeys: [systemadmin-promotions]
title: Platform offers and promotions management
---

# Platform offers and promotions management

This document covers how to create, edit, pause, activate, and delete platform-level offers, how to read offer analytics, and how platform offers interact with vendor-level offers.

## Platform offers list

The Platform Offers list shows all platform-level offers with their current state (draft, active, paused, expired, deleted). Use it to:
- Review all active and scheduled promotions.
- Find a specific offer to edit, pause, or activate.
- Identify offers approaching their end date or budget cap.

## Creating a platform offer

1. Navigate to Platform Offers List in System Admin.
2. Click to create a new offer.
3. Set the following:
   - Offer name and internal description.
   - Offer type (percentage discount, free delivery, flat amount off, or other configured types).
   - Discount value.
   - Scope — whether the offer applies to all vendors, specific vendors, or a defined group.
   - Start date and end date.
   - Budget cap — the maximum total discount value the offer will pay out before it stops applying.
   - Promo code (optional) — if customers need to enter a code to trigger the offer.
4. Save as draft.
5. Review and activate when ready to go live.

## Editing an existing offer

Open the offer from the Platform Offers list and edit the fields you need to change. Editing a live (active) offer is possible but changes take effect immediately. For significant changes to an active offer, pause it first, edit, then reactivate.

## Pausing a platform offer

Use Pause to temporarily stop the offer from applying at checkout without deleting the record. Paused offers retain all their settings and can be reactivated later. A paused offer does not count down its budget cap while paused.

## Activating a platform offer

Use Activate to make a draft or paused offer live. The offer begins applying at checkout from the activation point forward, subject to its start date, end date, and budget cap.

## Deleting a platform offer

Use Delete to permanently remove an offer record. Deleting an active offer stops it immediately. Best practice is to pause an offer before deleting it. Deleted offers are not recoverable.

## Offer analytics

The Offer Analytics page shows performance signals for a platform offer:
- Number of times the offer was applied at checkout.
- Total discount value paid out.
- Number of orders that included the offer.
- Budget cap consumption — how much of the cap has been used.
- Offer code redemption count (if a promo code is used).

Use analytics to evaluate whether an offer is performing as intended. If budget consumption is faster than expected, consider pausing the offer and reviewing the scope.

## How platform offers interact with vendor offers

Platform offers and vendor offers can both be active at the same time. Stacking rules determine what happens when a customer qualifies for both. By default, only the higher-value offer applies. If you need to understand the stacking behaviour for a specific offer combination, check the offer configuration for any stacking override settings.

## What to check when a platform offer is not applying

1. Confirm the offer is in the active state (not paused, draft, or deleted).
2. Check the start and end dates — it may have expired or not yet started.
3. Check the budget cap — the offer may be fully consumed.
4. Confirm the scope — the vendor the customer is ordering from must be in the offer's scope.
5. If a promo code is required, confirm the customer is entering it correctly.
