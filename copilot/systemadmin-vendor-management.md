---
audience: SystemAdmin
skillKeys: [systemadmin-vendors, systemadmin-vendor-edit]
title: Vendor and vendor user management
---

# Vendor and vendor user management

This document covers how to find and review vendor records, activate and deactivate vendors, manage vendor-linked users, transfer or remove vendor user access, and understand propagation after vendor state changes.

## Vendors list

The Vendors list shows all vendor records on the platform. Use it to:
- Search for a vendor by name, email, or other identifiers.
- Filter by active or inactive state.
- Identify vendors that need state changes or user management.
- Open a vendor record for detailed review before making changes.

## Vendor details

The vendor detail page shows the platform view of one vendor: their current status, core profile fields, linked branches, commission assignment, and any recent state changes. Always review vendor details before running an activation, deactivation, or user management action to confirm the context is correct.

The page uses tabbed sections:
- **Vendor Profile** for core business and account data.
- **Branch Details** for branch-level support checks such as status, default branch, delivery fee, and estimated delivery time.

## Branch support workflow for System Admin

System Admin can support any vendor branch by opening the vendor record, reviewing **Branch Details**, and then opening menu operations in the same branch context used by Vendor Admin. This keeps branch-level troubleshooting and menu correction aligned with the same editor behavior vendors use.

Branch data can contain nullable values while onboarding is still in progress. Handle missing values as expected state instead of a failure condition.

## Payment configuration scope

Payment configuration is vendor-level, not branch-level. New vendors can exist without payment configuration during initial onboarding, so missing configuration should be treated as an expected onboarding state.

## Activating a vendor

Use the activation workflow when a vendor should return to live use on the platform. Activation makes the vendor visible in customer discovery and search.

1. Open the vendor from the Vendors list.
2. Review the current status and confirm activation is the right action.
3. Run the Activate action.
4. Allow normal propagation delay for downstream visibility — caches and search indexes refresh on their normal schedule.

## Deactivating a vendor

Use the deactivation workflow when a vendor should be taken out of live use. Deactivation removes the vendor from customer discovery and prevents new orders.

1. Open the vendor from the Vendors list.
2. Review the current status and confirm deactivation is appropriate.
3. Run the Deactivate action.
4. Monitor the vendor's state in the system if ongoing orders or commitments need to be resolved.

Deactivation does not delete the vendor record. All history, menus, orders, and commissions remain accessible for support and finance purposes.

## Propagation after a vendor state change

After activating or deactivating a vendor, downstream visibility catches up through normal caching and indexing cycles. Expect a short delay before the change is fully reflected in search, discovery surfaces, and cached reads. If the vendor state looks wrong after the normal delay, ask the System Admin tools team to run a reindex and cache rebuild for the vendor.

## Vendor users list

The Vendor Users list shows all users linked to one or more vendor records. Use it to:
- See who has access to a vendor's Vendor Admin account.
- Identify incorrect or outdated user-to-vendor associations.
- Start a transfer or removal workflow.

## Transferring a vendor user

Use the transfer workflow when a user's link to a vendor should change — for example when ownership of a restaurant changes hands or a manager moves to a different branch.

1. Open the Vendor Users list and find the user.
2. Review their current vendor and branch associations.
3. Use the Transfer action to move the association to the correct vendor or branch.
4. Confirm the user now has the right access.

## Removing a vendor user

Use the removal workflow when a user should no longer have access to a vendor at all — for example a former employee or owner.

1. Open the Vendor Users list and find the user.
2. Review the associations to confirm removal is the right action.
3. Use the Remove Vendor User action to remove the association.
4. The user loses access to the vendor's Vendor Admin context immediately.

Removing vendor access does not delete the user's Entra ID account. It removes only the association that grants Vendor Admin access to that vendor's data.

## What to check when a vendor user reports wrong access

1. Open the Vendor Users list for the affected vendor.
2. Confirm whether the user appears with the correct association.
3. If the user is associated with the wrong vendor or branch, use the transfer workflow.
4. If the user should not have access at all, use the removal workflow.
5. If the user cannot see a vendor they should have access to, confirm their association exists and the vendor is in the active state.
