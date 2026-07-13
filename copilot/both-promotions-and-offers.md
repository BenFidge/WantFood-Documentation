---
audience: Both
skillKeys: [vendor-promotions, systemadmin-promotions]
title: Promotions and offers — platform and vendor
---

# Promotions and offers — platform and vendor

This document covers the two types of promotion in WantFood — platform offers managed by System Admin and vendor offers managed by individual vendors — including how to create, edit, pause, activate, and remove each type, promo code rules, stacking behaviour, and analytics.

## Platform offers vs vendor offers

Platform offers are created and managed by System Admin users. They are platform-wide promotions that can affect all or selected vendors and appear in customer-facing discovery and checkout experiences. Examples include percentage-off promotions, free delivery periods, and welcome-back campaigns.

Vendor offers are created and managed by individual vendors through Vendor Admin. They are scoped to that vendor's customers and menu. Examples include percentage-off specific dishes, promo codes for repeat customers, and time-limited discount events.

Both types of offer can be active at the same time. Stacking rules determine what happens when a customer qualifies for multiple offers at checkout.

## Creating a platform offer (System Admin)

1. Navigate to Platform Offers List in System Admin.
2. Click to create a new offer.
3. Set the offer type, discount value, eligible vendors or scope, start and end dates, budget cap, and any promo code.
4. Save as draft. The offer is not visible to customers until it is activated.
5. Activate the offer when it is ready to go live.

## Creating a vendor offer (Vendor Admin)

1. Navigate to Vendor Offers List in Vendor Admin.
2. Confirm the correct vendor context is active.
3. Click to create a new offer.
4. Set the offer type, discount value, eligible dishes or scope, start and end dates, budget cap, and promo code if applicable.
5. Save the offer. Activate it when it is ready.

## Editing an existing offer

Editing a live offer is possible but changes may affect in-progress customer journeys. For significant changes, pause the offer, make the edit, then reactivate.

## Pausing and activating an offer

Use the Pause action to temporarily stop an offer from being applied at checkout. The offer record is retained and can be reactivated later. Use Activate to make a paused or draft offer live.

## Cloning a vendor offer

Use the Clone action on a vendor offer to create a copy with the same settings. Cloning is useful when running a similar promotion to a previous one with minor adjustments.

## Deleting an offer

Platform offers can be deleted from the Platform Offers List. Vendor offers can be deleted from the Vendor Offers List. Deleting an active offer is not recommended — pause it first, then delete once you are sure it is no longer needed.

## Promo codes

Promo codes are optional codes that customers enter at checkout to trigger an offer. Rules:
- A promo code must be unique across the platform. If you try to save a code that already exists in another active offer, the system will flag the conflict.
- Use the promo code availability check in Vendor Admin before finalising a new code to confirm it is not already in use.
- Codes are case-insensitive at checkout.
- If a code is tied to a paused or deleted offer, it will not work at checkout.

## Stacking rules — when multiple offers apply

When a customer qualifies for both a platform offer and a vendor offer at the same time, the platform applies stacking rules to determine the outcome. By default, stacking is limited — only the highest-value single offer applies unless the platform has explicitly enabled stacking for a specific promotion. System Admin can see stacking configuration in the offer settings.

## Budget caps and schedule controls

Both offer types support a maximum budget cap that stops the offer from applying once a spending threshold is reached. Both types also support defined start and end dates. An offer outside its active date window will not apply at checkout even if it is in the activated state.

## Offer analytics (System Admin)

The Offer Analytics page in System Admin shows performance signals for a platform offer: number of times applied, total discount value given, number of orders with the offer, and any budget cap consumption. Use this to evaluate whether an offer is performing as intended and whether to extend, adjust, or close it.

## What to check when an offer is not applying at checkout

1. Confirm the offer is in the activated state (not paused, draft, or deleted).
2. Check the start and end dates — the offer may have expired or not yet started.
3. Check the budget cap — the offer may have been fully consumed.
4. If a promo code is involved, confirm the customer is entering the code correctly.
5. If there is a stacking conflict, confirm which offer rule takes precedence.
