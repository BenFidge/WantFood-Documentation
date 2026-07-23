---
audience: VendorAdmin
skillKeys: [vendor-profile]
title: Managing your restaurant and branch details
---

# Managing your restaurant and branch details

This document covers how to update your restaurant profile and branch-specific information in Vendor Admin, and how changes propagate to customer-facing experiences.

## Manage restaurant page

The Manage Restaurant page is where you maintain the restaurant profile and branch-facing information for the current vendor or branch context. Fields you can update here include:

- Restaurant name and description.
- Branch address and contact details.
- Cuisine type and category tags.
- Opening hours and trading days.
- Any other branch-level configuration fields shown on the page.

Always confirm the correct vendor or branch context is active before making changes.

## Vendor-level vs branch-level information

Some information belongs to the vendor as a whole — for example the legal business name or the top-level brand identity. Other information is branch-specific — for example the trading address, phone number, opening hours, and order-acceptance settings.

The following fields are branch-level and are managed on the Branch tab of the Manage Restaurant page:

- Trading address and contact details.
- Opening hours and trading days.
- Cuisine types.
- **Taking Orders** (`IsAcceptingOrders`) — whether the branch is currently accepting new customer orders. This can also be toggled quickly from the dashboard and kanban context card without opening branch settings.
- **Scheduled Orders** (`AcceptsScheduledOrders`) — whether customers can book future delivery slots. The quick-toggle on the dashboard and kanban changes this field. The full scheduled-orders configuration (max schedule ahead hours, prep lead time, slot interval) is only editable here on the Branch tab.

When you update fields on the Manage Restaurant page, the changes apply to the currently active context (the vendor or branch shown in the context selector). If you manage multiple branches and need to update each one separately, switch context between each update.

## How to update branch details

1. Confirm the correct branch context is active.
2. Navigate to Manage Restaurant from the vendor dashboard.
3. Edit the fields you need to update — name, address, contact, opening hours, or other configuration.
4. Save the changes.

## How profile changes propagate to customer-facing experiences

After saving, profile changes are not immediately visible everywhere. WantFood uses caching so customer-facing experiences remain fast. A saved change typically propagates to customer discovery and the vendor detail page within the normal cache refresh cycle.

If a change is still not appearing after the expected delay, ask a System Admin user to run the rebuild caches tool for your vendor record.

## What to check when a profile change does not appear

1. Confirm the change was saved by reopening the Manage Restaurant page and checking the field values.
2. Allow normal propagation time (typically a few minutes to an hour depending on the cache cycle).
3. If still not appearing, contact System Admin to trigger a cache rebuild for your vendor.
4. If the address or opening hours are wrong in customer-facing results, check that the correct branch context was active when you saved.
