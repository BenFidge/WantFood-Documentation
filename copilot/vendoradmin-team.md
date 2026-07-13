---
audience: VendorAdmin
skillKeys: [vendor-team]
title: Team Management — adding and removing vendor users
---

# Team Management — adding and removing vendor users

This document explains how to manage the users who have access to your vendor account in Vendor Admin. These are the staff members who can sign into Vendor Admin and manage your restaurant — owners, managers, and general staff.

This is separate from the Drivers section, which manages delivery drivers. Drivers use the Driver Portal; team members use Vendor Admin.

---

## Team list

Navigate to **Team** in the Vendor Admin sidebar to see all users currently associated with your vendor account.

Each row shows:
- The user's full name
- Their email address
- Their role (Owner, Manager, or Staff)
- The date their account was added
- A Delete action

---

## How to add a new team member

1. Navigate to **Team** in the sidebar.
2. Click **Add User**.
3. Fill in the required fields:
	- **Email** — the email address the new user will sign in with
	- **First name** and **Last name**
	- **Display name** (optional — defaults to first + last name if left blank)
	- **Role** — Owner, Manager, or Staff (defaults to Staff)
4. Click **Add User**.

After the form is submitted:
- A Microsoft Entra ID account is created for the new user with a temporary password.
- The account is assigned the Vendor role in the WantFood enterprise application.
- The user is linked to your vendor account in the database.
- You are taken to the **Account Created** confirmation screen.

### The Account Created screen

This screen shows:
- The new user's name and email address
- Their **temporary password**

> ⚠️ **Copy the temporary password now.** This is the only time it will be shown. It is never stored or retrievable later. Share it with the new team member using a secure channel (not email).

The new team member will be prompted to change their password on their first sign-in to Vendor Admin.

After copying the password, you can either:
- Click **Add Another User** to immediately add a second team member
- Click **Back to Team** to return to the user list

---

## How to delete a team member

1. Navigate to **Team** in the sidebar.
2. Find the user you want to remove.
3. Click **Delete** on their row.
4. Confirm the deletion when prompted.

Deleting a team member:
- Removes their association with your vendor account
- Deletes their Microsoft Entra ID account
- Revokes their access to Vendor Admin immediately

> This action cannot be undone. If a deleted user needs to regain access, they must be added again as a new user, and a new temporary password will be issued.

---

## Roles

| Role | Intended for |
|------|-------------|
| **Owner** | Restaurant owner or primary account holder |
| **Manager** | Branch manager or senior staff with full operational access |
| **Staff** | General team members — order processing, menu updates, day-to-day tasks |

Role labels are informational at this stage. All Vendor role users have the same access level within Vendor Admin.

---

## What to do when a new user cannot sign in

1. Confirm the user is using the correct Vendor Admin URL — not the Driver Portal or any other URL.
2. Confirm the user is entering the email address exactly as shown on the Account Created screen (no extra spaces, correct domain).
3. Confirm the user has the temporary password — remind them it is case-sensitive.
4. The user will be prompted to change their password on first sign-in. This is expected and normal.
5. If the user has already changed their password and is still blocked, ask them to use the **Forgot password** link on the Microsoft sign-in page. This goes through Microsoft's own password reset flow.
6. If none of the above resolves the issue, contact your System Admin with the user's email address.

---

## What to do when a user should no longer have access

Delete their account from the **Team** list. Access is revoked immediately. There is no suspend or deactivate option — deletion is the only way to remove access.

If the user is currently signed in when you delete them, their existing session will remain valid until it expires (typically 1 hour). After that, they will not be able to sign in again.

---

## How vendor user accounts differ from driver accounts

| | Team (Vendor Users) | Drivers |
|---|---|---|
| Portal | Vendor Admin | Driver Portal |
| Account creation | Immediate — account is created with a temporary password | Invite-based — driver receives an email and self-registers |
| Role | Vendor | Driver |
| Managed from | Vendor Admin → Team | Vendor Admin → Drivers |
| Password issued by | Vendor Admin user (temporary password, copy immediately) | Driver self-sets during onboarding |

---

## Frequently asked questions

**Can I have more than one Owner?**
Yes. The role label is informational — you can add as many users with the Owner role as needed.

**Can a user be a team member on multiple vendor accounts?**
Not currently. Each Entra account is linked to one vendor at a time.

**Can I reset a team member's password from within Vendor Admin?**
No. Password management is handled entirely by Microsoft Entra ID. If a user has forgotten their password, they should use the Forgot password link on the Microsoft sign-in page. You cannot retrieve or reset their password from within the portal.

**Is the temporary password sent by email?**
No. The temporary password is shown once on the Account Created screen immediately after you add the user. It is your responsibility to share it securely with the new team member. It is not emailed to the user or to you.

**What happens to a user's active orders or open tabs if I delete them?**
Deletion removes access — it does not affect any orders or data they have already created. Existing orders will continue to process normally.
