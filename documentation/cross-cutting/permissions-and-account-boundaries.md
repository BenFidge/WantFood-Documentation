# Permissions and account boundaries

You use this guide when you need to explain who should be able to see or change something in WantFood.

## Core rule
Each portal exposes only the records and actions that belong to that role and context.

## What this means in practice
- System Admin users can work across platform-wide operational areas.
- Vendor Admin users can work only within the vendor and branch contexts they hold.
- Driver users can work only within their own driver account and assigned delivery context.
- A user should never see or change data outside the role and business relationship they hold.

## Common boundary checks
- If a Vendor Admin user sees the wrong restaurant data, confirm the active vendor or branch context first.
- If a driver cannot see expected work, confirm onboarding, sign-in, and assignment state.
- If an internal user should no longer have access, remove the admin account or vendor-linked access through the right workflow.

## Related
- [Manage admin users](../system-admin/features/manage-admin-users.md)
- [Transfer or remove vendor user](../system-admin/features/transfer-or-remove-vendor-user.md)
- [Switch vendor or branch context](../vendor-admin/features/switch-vendor-or-branch-context.md)
- [Update profile](../driver-portal/features/update-profile.md)

