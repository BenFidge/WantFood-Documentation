# Automation and background jobs

You use this guide when the visible outcome of an action depends on scheduled work, queue-driven processing, or a tool that starts background activity.

## Scheduled jobs
- Daily stats reconciliation keeps reporting current.
- Order archiving moves older completed orders out of the primary active path.
- Commission tier recalculation refreshes vendor fee tiers on a schedule.
- Rating recalculation refreshes vendor, branch, and dish ratings after review changes.
- Chat cleanup closes or removes stale order-thread state.

## Queue-driven or asynchronous work
- Image processing completes after the initial upload step.
- Vendor reindex work refreshes search visibility in the background.
- Cache rebuild work refreshes cached read models in the background.
- Route optimisation returns a result after the dashboard asks for it.

## Admin-triggered operations
- System Admin tools can start reindex, cache rebuild, postcode import, integrity checks, manual rating refresh, manual commission refresh, and invoice generation work.

## What this means for documentation
- A success message does not always mean the downstream effect is already visible.
- You should describe both the trigger action and the delay before the final visible outcome appears.
- If a workflow looks stale, check whether it depends on background work before you treat it as broken.

## Related
- [Run platform tools](../system-admin/features/run-platform-tools.md)
- [Manage content assets](../system-admin/features/manage-content-assets.md)
- [Manage dish content and images](../vendor-admin/features/manage-dish-content-and-images.md)
- [Route optimisation](../driver-portal/features/route-optimisation.md)

