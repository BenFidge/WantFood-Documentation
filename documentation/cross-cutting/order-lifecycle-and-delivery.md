# Order lifecycle and delivery

You use this guide when you need the end-to-end operational story for an order from vendor acceptance through final delivery outcome.

## Participants
- Vendor Admin handles kitchen and dispatch-side order work.
- Driver Portal handles delivery execution and route work.
- System Admin provides support, investigation, and recovery context.

## Typical flow
1. Vendor Admin receives the order and reviews it in the live board.
2. The vendor accepts, rejects, edits, or prepares the order.
3. A driver is assigned or unassigned as the dispatch model requires.
4. The driver starts shift work, opens active deliveries, and uses the delivery dashboard.
5. The driver updates delivery state until the order is delivered or an issue path is recorded.
6. System Admin can later inspect the order record, delivery state, finance signals, and recovery history.

## Important operating notes
- The vendor board and driver delivery pages are the main live operational surfaces.
- Route work is part of the driver delivery dashboard rather than a separate working page.
- Failed, reverted, declined, or support-heavy states should be treated as recovery paths, not as normal completion.

## Related
- [Manage live orders](../vendor-admin/features/manage-live-orders.md)
- [Update delivery status](../driver-portal/features/update-delivery-status.md)
- [Report a delivery issue](../driver-portal/features/report-a-delivery-issue.md)
- [Order details](../system-admin/pages/order-details.md)

