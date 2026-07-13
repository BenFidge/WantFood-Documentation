---
audience: SystemAdmin
skillKeys: [systemadmin-commission-tiers]
title: Commission tiers, configuration, and assignments
---

# Commission tiers, configuration, and assignments

This document is the System Admin reference for managing commission tiers, configuring assignments, triggering recalculation, and diagnosing commission problems. For the cross-portal overview of how commission works, see both-commission-tiers-and-volume-rules.md.

## Commissions dashboard

The commissions dashboard in System Admin shows the platform-wide commission state. Use it to:
- See how many vendors are in each tier.
- Identify vendors with unusual or unexpected tier assignments.
- Navigate into commission tier management or assignment configuration.
- Trigger a manual recalculation when needed.

## Commission tiers list

The Commission tiers list shows all defined tier rules. Each row shows the tier name, the commission rate, the volume threshold, and the number of vendors currently assigned to it.

Open a tier to review its full rule set before making changes. Do not delete a tier while vendors are assigned to it — reassign affected vendors first.

## How to create a new commission tier

1. Navigate to Commission tiers from the commissions dashboard.
2. Click to add a new tier.
3. Set the tier name, commission rate (percentage), volume threshold (completed orders per rolling period), and any special rules.
4. Save the tier.
5. Newly created tiers are not automatically assigned to any vendor — configure assignments separately.

## How to update an existing tier

1. Open the tier from the Commission tiers list.
2. Edit the rate, threshold, or rules as needed.
3. Save the changes.
4. Changes apply at the next recalculation run. To apply immediately, trigger a manual recalculation from the commissions dashboard or tools dashboard.

## How to remove a commission tier

1. Open the tier and confirm how many vendors are currently assigned to it (shown on the tier detail).
2. Reassign all affected vendors to the correct replacement tier before deleting.
3. Once the assignment count is zero, use the delete action.
4. Deleting a tier with active assignments will cause those vendors to fall back to the platform default.

## Commission configuration and assignments

The commission configuration workflow connects tier rules to specific vendors. Use it to:
- Assign a tier to an individual vendor.
- Change a vendor's tier assignment when their commercial arrangement changes.
- Apply a bulk tier assignment across a group of vendors.
- Review which vendors are on a non-standard arrangement.

### How to assign or change a vendor's tier

1. From the commissions dashboard, open the commission configuration for the target vendor.
2. Review the current assignment.
3. Change the tier to the intended one.
4. Save.
5. The new assignment takes effect at the next recalculation. Trigger manual recalculation if it needs to apply immediately.

## Nightly recalculation

The commission tier recalculation service runs nightly (default 02:00 UTC). It calculates each vendor's rolling completed-order count, compares it to tier thresholds, and updates tier assignments accordingly. Results are cached and tier-change events are published to downstream services.

## How to trigger manual recalculation

From the commissions dashboard or the tools dashboard, use the Recalculate Commission Tiers action. Manual recalculation runs the same logic as the nightly job but on demand. Allow a few minutes for the job to complete before checking results.

## What to check when a vendor's commission looks wrong

1. Open the vendor's record from the Vendors list and check the current tier assignment.
2. Open the commission configuration for the vendor and confirm the assigned tier is correct.
3. Check the tier definition to confirm the rate and threshold values are what you expect.
4. Check the last recalculation date on the commissions dashboard.
5. If the assignment looks stale, trigger a manual recalculation.
6. If the tier assignment is correct but the invoice amount still looks wrong, review the uninvoiced line items to see which orders and rates were captured in the invoice period.
