# Manage admin users

You use this workflow to add or remove people who can access the System Admin site.

## When you use this feature
- a new internal user needs admin access
- an existing admin user should lose access
- you are reviewing whether internal access is still correct

## Where the workflow starts
Users list.

## How to add an admin user
1. Open the Users list.
2. Click **Add User**.
3. Enter the new user's email address, first name, and last name. Display name is optional and defaults to first and last name combined.
4. Click **Add User** to submit.
5. You are taken to the **User Created** confirmation screen.
6. Copy the temporary password shown. This is the only time it is displayed — it is never stored and cannot be retrieved after you leave this screen.
7. Share the temporary password with the new user through a secure channel.
8. The new user signs in with their email address and the temporary password. They are prompted to set a new password on their first sign-in.

## How to remove an admin user
1. Open the Users list.
2. Find the user to remove.
3. Click **Delete** and confirm.
4. The user's System Admin access is revoked immediately. Their sign-in session remains valid until it expires, after which they cannot sign in again.

> Deletion removes System Admin access only. It does not affect any other Entra ID accounts or services the person uses.

## What to do if the temporary password was not copied
There is no way to retrieve the temporary password after leaving the User Created screen. Delete the user from the Users list and add them again. A new temporary password is issued each time.

## Expected result
Internal admin access stays limited to the people who should have it, and new users receive a working set of credentials to sign in with.

## Related
- [Users list](../pages/users-list.md)
- [User Created](../pages/user-created.md)
- [Permissions and account boundaries](../../cross-cutting/permissions-and-account-boundaries.md)

