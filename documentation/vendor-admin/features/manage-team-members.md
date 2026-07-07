# Manage team members

You use this workflow to add or remove the staff members who have access to Vendor Admin for the current vendor account.

## When you use this feature
- a new staff member needs access to Vendor Admin
- a staff member's role label should change
- a staff member should no longer have access
- you are reviewing whether current access is still correct

## Where the workflow starts
Team list.

## How to add a team member
1. Navigate to **Team** in the Vendor Admin sidebar.
2. Click **Add User**.
3. Enter the new member's email address, first name, and last name. Display name is optional and defaults to first and last name combined.
4. Select a role: Owner, Manager, or Staff. Staff is the default.
5. Click **Add User** to submit.
6. You are taken to the **Account Created** confirmation screen.
7. Copy the temporary password shown. This is the only time it is displayed — it is never stored and cannot be retrieved after you leave this screen.
8. Share the temporary password with the new team member through a secure channel.
9. The new member signs in with their email address and temporary password. They are prompted to set their own password on their first sign-in.

## How to remove a team member
1. Navigate to **Team**.
2. Find the member to remove.
3. Click **Delete** and confirm.
4. The member's Vendor Admin access is revoked immediately. Their active session remains valid until it expires, after which they cannot sign in again.

> Deletion removes Vendor Admin access and the associated Entra ID account. This action cannot be undone. If a deleted user needs access again, add them as a new team member.

## What to do if the temporary password was not copied
There is no way to retrieve the temporary password after leaving the Account Created screen. Delete the user from the Team list and add them again. A new temporary password is issued each time.

## Roles
| Role | Intended for |
|------|-------------|
| **Owner** | Restaurant owner or primary account holder |
| **Manager** | Branch manager or senior staff with full operational access |
| **Staff** | General team members for day-to-day operations |

Role labels are informational. All Vendor role users have the same access level within Vendor Admin.

## Expected result
Access to Vendor Admin matches the real team composition, and new members receive a working set of credentials to sign in with.

## Related
- [Team list](../pages/team-list.md)
- [Account Created](../pages/account-created.md)
- [Permissions and account boundaries](../../cross-cutting/permissions-and-account-boundaries.md)
