# Toggle scheduled orders

You use this workflow to enable or disable the ability for customers to place pre-scheduled (future) orders against the current branch.

## When you use this feature
- you want to allow customers to book a delivery slot in advance
- you need to disable scheduled ordering while you configure slot settings
- you are confirming whether the branch currently accepts advance orders

## Where the workflow starts
Vendor dashboard (branch context card) or Orders kanban dashboard (branch context card).

## How it works
1. Locate the branch context card at the top of the dashboard or the kanban page.
2. Find the **Scheduled Orders** toggle — it shows the current state for the branch.
3. Click the toggle to switch the state on or off.
4. The toggle updates immediately. A brief confirmation or error notification appears.
5. When on, customers can choose a future delivery slot during checkout. When off, only immediate (ASAP) orders are available.

## Scope
Scheduled Orders is a branch-level setting. It applies only to the branch shown in the context card. Other branches under the same vendor are unaffected.

## Configuring scheduled order parameters
The quick-toggle enables or disables scheduled ordering. The detailed timing parameters — how far ahead customers can schedule, preparation lead time, and slot interval — are set in the full Branch Settings tab under Manage Restaurant. You must configure those values before enabling the toggle for a good customer experience.

| Setting | What it controls |
|---------|-----------------|
| Max Schedule Ahead Hours | How many hours in advance a customer can book a slot (1–168) |
| Prep Lead Time (Minutes) | Minimum notice required before a slot can be booked |
| Slot Interval (Minutes) | Granularity of available slots (15, 30, or 60 minutes) |

## What customers see
When Scheduled Orders is on and parameters are configured, customers see a slot picker during checkout that shows available time windows based on the branch's opening hours and slot settings. When off, the slot picker is hidden.

## Expected result
The branch shows or hides the scheduled-orders slot picker on the customer front-end as set.

## Related
- [Vendor dashboard](../pages/vendor-dashboard.md)
- [Orders kanban dashboard](../pages/orders-kanban-dashboard.md)
- [Toggle taking orders](toggle-taking-orders.md)
- [Update branch details](update-branch-details.md)
