---
audience: VendorAdmin
skillKeys: [vendor-menu-builder]
title: Menu management — build, edit, and publish menus
---

# Menu management — build, edit, and publish menus

This document covers how to create, edit, and publish menus in Vendor Admin, manage categories and dishes, reorder content, handle multi-menu setups, and understand how scheduled orders interact with menu availability.

## Menu builder list

The menu builder list shows all menus belonging to the current vendor context. Each row shows the menu name, its current status (draft, published, or inactive), and the last modified date. Select a menu to open it in the menu editor.

## How to create a new menu

1. From the menu builder list, click to create a new menu.
2. Set the menu name and availability — the days and time windows when this menu is active.
3. Save. The menu is created in draft status and is not visible to customers until it is published.

## Menu editor

The menu editor is the main workspace for structured menu editing. It displays the menu as a hierarchy of categories and dishes. From here you can:
- Add, edit, and remove categories.
- Add, edit, move, and remove dishes within categories.
- Set dish prices, descriptions, and availability.
- Upload dish images.
- Reorder categories and dishes.

## How to add a category

1. In the menu editor, use the Add Category action.
2. Set the category name and optionally a description.
3. Save. The new category appears at the bottom of the menu. Reorder it if needed.

## How to add a dish

1. Select the category you want the dish to belong to.
2. Use the Add Dish action.
3. Fill in the dish name, description, price, and any option groups or modifiers.
4. Upload a dish image if available (see image upload notes below).
5. Set the dish availability — whether it is always available or restricted to specific times.
6. Save.

## How to edit an existing dish

Open the dish from the menu editor. Update any fields — name, description, price, options, image, or availability. Save when done. Changes to a published menu are visible to customers after the menu is re-published and caches refresh.

## How to move a dish to a different category

In the menu editor, use the move action on the dish to select a different category. The dish moves to the target category. Reorder it within the category if needed.

## How to reorder categories and dishes

Use the drag-and-drop or ordering interface in the menu editor to sequence categories and dishes. Save the new order. The sequence affects how customers see the menu — the top category and first dishes in each category receive the most attention.

## How to delete a category or dish

Use the delete action on the category or dish. Deleting a category removes all dishes within it. Confirm before deleting to avoid losing content. Deleted categories and dishes cannot be recovered.

## How to publish a menu

Publishing makes the menu visible to customers in the WantFood ordering experience.

1. In the menu builder list or menu editor, use the Publish action.
2. Confirm the publish.
3. The menu enters the published state. Customers can see it after caches refresh (typically within minutes).

Only one menu can be the active published menu at a time for a given availability window. If you publish a new menu that overlaps with an existing published menu, the system may prompt you to resolve the conflict.

## How to unpublish or deactivate a menu

Use the Unpublish or Deactivate action on the menu. Customers can no longer order from a deactivated menu. The menu record is retained and can be republished later.

## Multi-menu handling and time windows

WantFood supports multiple menus per vendor, each with its own availability window. This lets you serve a breakfast menu in the morning and a different lunch or dinner menu later in the day. Set the availability window (days and times) for each menu when creating or editing it.

When a customer browses your restaurant, the menu that is active at that time (based on the current time and your availability window settings) is the one they see.

## Scheduled orders and menu availability

Customers can place scheduled orders for a future time slot. A scheduled order uses the menu that will be active at the chosen delivery time, not the menu active when the order is placed. Make sure your menu availability windows are set correctly so scheduled orders always fall within a valid menu window.

## Dish image upload — async processing

When you upload a dish image, the image is queued for processing asynchronously. There is a normal delay of seconds to a few minutes before the image appears on the dish. If a dish image has not appeared after 10 minutes, check that the file format (JPEG or PNG) and size were within the accepted limits.

## AI menu scan and re-scan behavior

Vendor Admin can import menus from brochure images and PDFs through the AI menu scan flow.

- The flow uses a three-step wizard: **Upload pages** → **Review and delta** → **Import complete**.
- Upload uses a drag-and-drop dropzone and keeps the selected menu context through review and re-scan.
- For an existing menu import, review a visual delta before import so you can verify added, removed, and changed items.
- For a new menu import, the result is saved as unpublished by default.
- You can re-scan in the same menu context and compare extraction output before applying changes.
- This flow only imports menu data. It does not create vendor accounts, vendor records, branches, or invitations.

## What to do when menu changes are not appearing for customers

1. Confirm the menu is in the published state.
2. Allow normal cache propagation time after publishing.
3. If still not visible, ask a System Admin user to run the rebuild caches tool for your vendor.
4. For dish images specifically, allow extra time for async image processing before triggering a cache rebuild.
