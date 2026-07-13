---
audience: SystemAdmin
skillKeys: [systemadmin-home]
title: System Admin — dashboard and navigation
---

# System Admin — dashboard and navigation

This document covers how to use the System Admin dashboard and navigate the portal efficiently, including what each section of the dashboard shows, common operating patterns, the privacy page, and error pages.

## What the System Admin dashboard shows

The dashboard gives you an operational overview of the platform and is the fastest route into the area that needs attention next. It shows:

- Summary counts of pending vendor applications awaiting review.
- Active vendors and any that have recently changed state.
- Recent orders that may need support attention.
- Uninvoiced activity indicators for the finance workflow.
- Quick links to the most commonly used sections: Applications, Vendors, Orders, Commissions, Invoices, Promotions, and Tools.

Use the dashboard at the start of every session to understand the current platform state before opening a specific record.

## Quick-navigation patterns

### When you need to review vendor applications

Start from the dashboard Applications tile or go directly to Applications List in the Applications section of the sidebar.

### When you need to change vendor state or investigate a vendor

Go to Vendors List from the dashboard or the Vendors section. Search or filter to find the vendor, then open their record from the list.

### When you need to support an order

Go to Orders List from the dashboard or the Orders section. Use the search and filter options to locate the order by vendor name, order number, date range, or status.

### When you need to do finance work

Use the Commissions section for commission dashboard and tier management. Use the Invoices section for uninvoiced dashboard and invoice list.

### When you need to do content or campaign work

Use the Content section for the asset library, hero slides, support-local slides, and regions.

### When you need to run recovery tools

Use Admin Tools -> Tools Dashboard.

## Common operating patterns

1. Always start from the dashboard to get the current platform state.
2. Open the page that owns the record you need to change.
3. Use the matching feature workflow when the action changes state, visibility, or commercial behaviour.
4. If a change depends on background work (cache rebuild, reindex, nightly recalculation), check the relevant cross-cutting note before treating the outcome as complete.

## Privacy page

The privacy page in System Admin displays the privacy notice applicable to the System Admin site. It describes how operator data is used within the context of the platform administration role. Refer to this page when a user has a question about what data the admin site collects or how it is handled.

## Error pages

Error pages appear in System Admin when the site cannot complete the current request or a route fails. Common causes include:

- An expired session — sign in again.
- A record that no longer exists — it may have been deleted or its URL has changed.
- A permissions issue — the current admin user does not have access to the requested area.
- A downstream service being temporarily unavailable — wait a moment and retry.

If an error page appears repeatedly for the same action, it is likely not a transient issue. Check the Operations team or raise a support investigation through the normal internal channel.
