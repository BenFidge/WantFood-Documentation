---
audience: SystemAdmin
skillKeys: [systemadmin-home]
title: System Admin — managing admin user accounts
---

# System Admin — managing admin user accounts

This document covers how to view, add, and remove System Admin users, and explains the role boundaries between System Admin, Vendor Admin, and other portal roles.

## Admin users list

The Users list in System Admin shows all current admin user accounts. Use it to:
- Confirm who currently has access to System Admin.
- Find a specific user to review or manage their account.
- Identify accounts that should be removed (former staff, role changes).

## How to add a new admin user

1. Navigate to **Users** in System Admin.
2. Click **Add User**.
3. Fill in the required fields:
   - **Email** — the email address the new user will sign in with
   - **First name** and **Last name**
   - **Display name** (optional — defaults to first + last name if left blank)
4. Click **Add User**.

After the form is submitted:
- A Microsoft Entra ID account is created immediately for the new user with a temporary password.
- You are redirected to the **User Created** confirmation screen.

### The User Created screen

This screen shows:
- The new user's name and email address
- Their **temporary password** in a read-only field with a Copy button

> ⚠️ **Copy the temporary password now.** This is the only time it will be shown. It is never stored or retrievable after you leave this screen. Share it with the new admin user using a secure channel (not email).

After copying the password, you can either:
- Click **Add Another User** to immediately add another admin user
- Click **View All Users** to return to the Users list

The new admin user will be prompted to set a new password on their first sign-in to System Admin.

Only existing System Admin users can add new admin users. You cannot self-add to System Admin — the action must be performed by an existing admin.

## How to remove an admin user

1. Navigate to Users List.
2. Find the user to remove.
3. Use the Delete or Remove action.
4. Confirm the removal.

Removing an admin user revokes their System Admin access immediately. It does not delete their Entra ID account — it only removes the association that grants System Admin access.

## Role boundaries — what a System Admin user can and cannot do

System Admin users can manage vendor applications, vendor records, vendor users, admin users, orders, commissions, invoices, promotions, reviews, content, cuisine types, bad words, payment settings, and platform tools.

System Admin users cannot:
- Access Vendor Admin management pages on behalf of a vendor.
- Accept, reject, or manage live orders directly (this is a Vendor Admin action).
- Access Driver Portal work or driver shift data.
- Access customer-facing order or account data beyond what System Admin order detail exposes for support purposes.

## What to check when an admin user cannot access a section

1. Confirm the user is listed in the Users list with an active account.
2. Check whether the section is gated by a role or permission that the user does not hold.
3. If the user was recently added and cannot sign in at all, confirm they are using the temporary password shared at the time of creation. Remind them it is case-sensitive and they will be prompted to change it on first sign-in.
4. If the temporary password was not copied before leaving the User Created screen, the user must be deleted and recreated — it cannot be retrieved.
5. If access was recently removed and the user can still sign in, their session may be cached. Ask them to sign out and sign back in.
