# Propagation and cache behaviour

You use this guide when an admin change saved successfully but the downstream result is not visible yet.

## Why this happens
WantFood uses cached and precomputed data so the public and operational experiences can respond quickly. That means a saved change can take time to appear everywhere.

## Common examples
- Vendor activation or deactivation does not look immediate in every downstream surface.
- Menu publication and dish image changes can appear after image processing and cache refresh behaviour catch up.
- Dish category assignment changes can appear after menu cache and search index refresh behaviour catch up, especially when one dish is assigned to multiple categories.
- Region, cuisine, or delivery-cost changes can take time to affect discovery and quoting behaviour.
- Hero slides, support-local content, and other homepage content can appear after caches refresh.

## What to do
1. Confirm that the source record actually saved.
2. Allow the normal delay for caches and background work.
3. If the result still looks stale, review whether a reindex or cache rebuild is the right recovery action.
4. Use System Admin tools when the normal delay has clearly passed and the site still looks stale.

## Related
- [Run platform tools](../system-admin/features/run-platform-tools.md)
- [Manage hero slides](../system-admin/features/manage-hero-slides.md)
- [Publish a menu](../vendor-admin/features/publish-a-menu.md)
- [Configure delivery costs](../vendor-admin/features/configure-delivery-costs.md)
