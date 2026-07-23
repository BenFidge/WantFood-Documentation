---
audience: VendorAdmin
skillKeys: [vendor-home]
title: Vendor dashboard and context switching
---

# Vendor dashboard and context switching

This document covers the vendor dashboard, the no vendor context state, how to switch between vendor and branch contexts, and how context affects every page in Vendor Admin.

## Vendor dashboard

The vendor dashboard is the main working home for the currently selected vendor or branch. It shows today's order summary — incoming, in-progress, and completed orders for the current day — quick links to Orders, Menu, Drivers, Offers, and Settings, and any alerts relevant to the current context.

Start from the dashboard at the beginning of every session to understand the current state before moving to a specific task.

## Branch context card

The branch context card appears at the top of the dashboard (and also at the top of the Orders kanban page). It always shows the current branch name and two live toggles:

- **Taking Orders** — when on, the branch is accepting new customer orders. When off, no new orders can be placed for this branch on the customer front-end.
- **Scheduled Orders** — when on, customers can book future delivery slots during checkout. When off, only immediate (ASAP) orders are available.

Both toggles update instantly without a page reload. Changes apply only to the currently selected branch. The context card is always visible — even if you only manage one branch — so you can change branch status from the dashboard or kanban at any time without navigating to branch settings.

## No vendor context state

If you sign in and see the no vendor context page, you are signed in but no vendor or branch is currently active. This happens when you have just completed onboarding and have not yet selected a context, when you have access to multiple vendors and no context is active, or when your previous session context has been cleared. Use the context selector to choose the vendor or branch you want to work with.

## How to switch between vendor and branch contexts

If you manage multiple vendors or multiple branches under the same vendor, use the context selector in the top navigation to choose which restaurant you are working with. Every page in Vendor Admin — orders, menus, drivers, offers, delivery costs, payment methods — shows data and accepts changes only for the currently selected context.

To switch context:
1. Locate the context selector in the top navigation.
2. Click and select the vendor or branch you want.
3. The page reloads with the new context active.
4. The Taking Orders and Scheduled Orders toggles in the context card update to show the new branch's current state.

Changes you make in one context do not affect other contexts. Before making any change, confirm the correct context is active.

## How context affects every page

The selected vendor or branch context determines which orders appear in the kanban and all-orders views, which menus are shown in the menu builder, which drivers are available for assignment, which offers are listed, which delivery cost rules apply, and which payment methods are configured.

If data looks wrong — wrong orders, wrong menu, wrong drivers — confirm the active context before investigating further. Most data-looks-wrong issues in Vendor Admin are caused by the wrong context being selected.

## Common dashboard navigation patterns

- New order arrived: use the Orders quick link or kanban notification.
- Branch going offline (kitchen full, early close): use the Taking Orders toggle in the context card.
- Scheduled orders need enabling: use the Scheduled Orders toggle in the context card; configure timing parameters in Manage Restaurant → Branch tab.
- Menu change needed: use the Menu quick link from the dashboard.
- Promotion check: use the Offers quick link.
- Configuration work: use Settings (delivery costs, payment methods).
- Driver management: use the Drivers list.
