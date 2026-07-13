---
audience: VendorAdmin
skillKeys: [vendor-promotions]
title: Vendor promotions and promo codes
---

# Vendor promotions and promo codes

This document covers how to create, edit, pause, activate, clone, and delete vendor offers in Vendor Admin, how to check promo code availability, and how vendor offers interact with platform offers.

## Vendor offers list

The Vendor Offers list shows all offers belonging to the current vendor context. Each row shows the offer name, type, discount value, status (draft, active, paused, expired), start and end dates, and available actions.

Use the list to review your active promotions, find an offer to edit, and check whether any offers are approaching their end date or budget cap.

## How to create a vendor offer

1. Confirm the correct vendor context is active.
2. Navigate to the Vendor Offers list.
3. Click to create a new offer.
4. Set the following:
   - Offer name (internal label for your reference).
   - Offer type: percentage discount, flat amount off, free delivery, or another available type.
   - Discount value.
   - Scope: whether the offer applies to all dishes or selected items.
   - Start date and end date.
   - Budget cap: the maximum total discount value before the offer stops applying.
   - Promo code (optional): a code customers enter at checkout to trigger the offer.
5. Save as draft.
6. Check the promo code availability if using a code (see below).
7. Activate the offer when it is ready to go live.

## How to edit an existing offer

Open the offer from the Vendor Offers list. Edit the fields you need to change. For significant changes to a live offer, pause it first, edit, then reactivate to avoid unexpected customer behaviour mid-transaction.

## How to pause an offer

Use the Pause action on an active offer. The offer stops applying at checkout immediately. Budget cap consumption pauses. The record is retained and can be reactivated.

## How to activate an offer

Use the Activate action on a draft or paused offer. The offer begins applying at checkout from that point forward, subject to its start date, end date, and budget cap.

## How to clone an offer

Use the Clone action on any offer to create a copy with the same settings. Cloning is useful when running a similar promotion to a previous one. Edit the cloned offer before activating it — at minimum, update the promo code (if used) to avoid a conflict with the original.

## How to delete an offer

Use the Delete action. Deleting an active offer stops it immediately. Best practice is to pause before deleting. Deleted offers are not recoverable.

## Promo code availability checks

Before saving an offer with a promo code, confirm the code is available. Promo codes must be unique across the platform — if the code you want is already used by another active offer (yours or a platform offer), the system will reject it.

To check availability:
1. In the offer creation or edit form, enter the promo code.
2. Use the availability check action (shown near the code field).
3. The system confirms whether the code is available or already in use.
4. If the code is taken, try a variation before saving.

## How vendor offers interact with platform offers

Platform offers are created by WantFood operations and can apply alongside vendor offers. When a customer qualifies for both a platform offer and your vendor offer at the same time, stacking rules determine which applies. By default, only the higher-value offer is applied. If you believe a stacking conflict is preventing your offer from applying, contact System Admin.

## What customers see when an offer applies

If an offer applies automatically at checkout (no promo code required), the customer sees the discount on their order summary without needing to do anything. If the offer requires a promo code, the customer must enter the code on the checkout page for the discount to apply.
