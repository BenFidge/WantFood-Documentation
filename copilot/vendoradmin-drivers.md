---
audience: VendorAdmin
skillKeys: [vendor-drivers]
title: Drivers — invitation, management, and onboarding handoff
---

# Drivers — invitation, management, and onboarding handoff

This document covers how to invite and manage delivery drivers in Vendor Admin, what the driver experiences after accepting an invitation, and how to troubleshoot driver access problems.

## Drivers list

The Drivers list shows all drivers currently linked to the active vendor or branch context. Each row shows the driver name, email address, branch assignment, invitation status, and available actions.

Use the Drivers list to:
- See who is currently set up as a driver for your restaurant.
- Invite new drivers.
- Edit driver details.
- Assign or unassign drivers from branches.
- Resend invitations.
- Remove drivers.

## How to invite a new driver

1. Confirm the correct vendor or branch context is active.
2. Navigate to the Drivers list from the vendor dashboard.
3. Click Invite Driver.
4. Enter the driver''s name and email address.
5. Save. An invitation email is sent to the driver.

The invitation email contains a link to the Driver Portal onboarding landing page. The driver must follow this link to create their account.

## What the driver experiences after accepting

1. The driver follows the invitation link to the Driver Portal onboarding landing page.
2. On the registration page, the driver creates their account credentials using their email address and Entra ID.
3. The onboarding welcome page confirms their account is set up.
4. The driver can now sign into Driver Portal, see their assigned deliveries, and start shift work.

After the driver completes onboarding, they appear as available in the driver selector when assigning orders from the Vendor Admin kanban dashboard.

## How to edit a driver record

Open the driver from the Drivers list and update the details — name, email address, or any other editable fields. Save the changes.

## How to resend a driver invitation

If a driver has not received their invitation email or the link has expired:
1. Open the driver record from the Drivers list.
2. Use the Resend Invitation action.
3. A new invitation email is sent to the email address on the driver record.

## How to assign a driver to a branch

1. Open the driver record from the Drivers list.
2. Use the Assign to Branch action.
3. Select the branch from the available list.
4. Save. The driver is now associated with that branch and will appear in order assignment for orders from that branch.

## How to unassign a driver from a branch

Open the driver record and use the Unassign from Branch action. The driver is removed from the branch association. They may still be available at the vendor level depending on your configuration.

## How to remove a driver

Use the Remove action on the driver record. Removing a driver revokes their association with your vendor — they can no longer be assigned to your orders. Removing a driver does not delete their Driver Portal account; it only ends the association with your vendor.

Before removing a driver, confirm they do not have any active deliveries in progress.

## What to check when a driver cannot see their work

1. Confirm the driver has completed onboarding — they must have finished the Driver Portal registration before they can see deliveries.
2. Confirm the driver is signed into the correct portal (Driver Portal, not Vendor Admin).
3. Confirm they have started their shift in Driver Portal — drivers must begin a shift before deliveries appear in their active list.
4. Check the driver''s branch assignment — if the driver is assigned to a different branch than the order''s branch, they may not see those orders.
5. If all of the above look correct and the driver still cannot see their work, contact System Admin with the driver''s name and email address.
