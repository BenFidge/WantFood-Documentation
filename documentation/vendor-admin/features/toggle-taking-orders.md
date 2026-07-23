# Toggle taking orders

You use this workflow to suspend or resume order acceptance for the current branch without navigating to branch settings.

## When you use this feature
- you need to temporarily stop accepting new orders (kitchen is full, closing early, technical issue)
- you want to resume accepting orders after a pause
- you are checking whether the branch is currently live for new orders

## Where the workflow starts
Vendor dashboard (branch context card) or Orders kanban dashboard (branch context card).

## How it works
1. Locate the branch context card at the top of the dashboard or the kanban page.
2. Find the **Taking Orders** toggle — it shows the current live state of the branch.
3. Click the toggle to switch the state on or off.
4. The toggle updates immediately. A brief confirmation or error notification appears.
5. The change affects whether new orders can be placed by customers on the front-end for this branch.

## Scope
Taking Orders is a branch-level setting. It applies only to the branch shown in the context card. Other branches under the same vendor are unaffected. To change a different branch, switch context first.

## What customers see
When Taking Orders is off, the branch is no longer shown as available for immediate orders on the customer front-end. Existing in-progress orders continue normally — disabling Taking Orders does not cancel or affect orders already placed.

## Full branch settings
The Taking Orders state can also be viewed and changed in the full branch settings tab under Manage Restaurant. The quick-toggle on the dashboard and the setting on the Branch tab are the same field — changing one updates the other.

## Expected result
The branch accepts or rejects new orders as set.

## Related
- [Vendor dashboard](../pages/vendor-dashboard.md)
- [Orders kanban dashboard](../pages/orders-kanban-dashboard.md)
- [Toggle scheduled orders](toggle-scheduled-orders.md)
- [Update branch details](update-branch-details.md)
