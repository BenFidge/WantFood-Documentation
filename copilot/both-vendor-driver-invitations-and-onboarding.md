---
audience: Both
title: Vendor and driver invitations and onboarding
---

# Vendor and driver invitations and onboarding

This document covers the full invitation and onboarding chain for both vendors and drivers — from how an approval or invitation is triggered through to a working account in the correct portal.

## The invitation chain

Onboarding for both vendors and drivers starts with an invitation that is triggered by a platform action:

- Vendor onboarding is triggered when a System Admin user approves a vendor application.
- Driver onboarding is triggered when a Vendor Admin user invites a driver from the Drivers list.

In both cases, the invited person receives an email containing a link to the onboarding landing page for their portal. They follow the link, complete registration, and gain access to their portal.

## Vendor onboarding after application approval

### What happens when System Admin approves an application

1. The System Admin user reviews the application details and clicks Approve.
2. An invitation email is sent to the email address on the application.
3. The applicant follows the link to the Vendor Admin onboarding landing page.
4. On the onboarding registration page, the applicant creates their account credentials (linked to their Entra ID identity).
5. The onboarding welcome page confirms completion and explains what to do next.
6. The vendor can now sign into Vendor Admin and access their restaurant management pages.

### Stripe Connect and KYC steps that follow approval

After onboarding completes, the vendor typically needs to complete Stripe Connect account setup to enable card payment acceptance. This is a separate step handled through the payment methods configuration in Vendor Admin. KYC and identity verification requirements are part of the Stripe Connect flow and are handled directly by Stripe.

### What to do if the vendor does not receive the invitation email

1. Check the email address on the application in System Admin to confirm it is correct.
2. Ask the vendor to check their spam folder.
3. If the email address is wrong, a System Admin user may need to correct the application record before reissuing the invitation.
4. If the invitation link has expired, contact the platform operations team to arrange a new invitation.

## Driver invitation from Vendor Admin

### How to invite a driver

1. Sign into Vendor Admin and confirm the correct vendor or branch context.
2. Navigate to Drivers List.
3. Click Invite Driver.
4. Enter the driver's name and email address.
5. Save the invitation. An invitation email is sent to the driver.

### What the driver receives

The driver receives an email with a link to the Driver Portal onboarding landing page. The link is specific to their invitation and persona — it cannot be used to access Vendor Admin or System Admin.

### Driver Portal onboarding handoff

1. The driver follows the invitation link to the Driver Portal onboarding landing page.
2. On the onboarding registration page, the driver creates their account credentials.
3. The onboarding welcome page confirms completion.
4. The driver can now sign into Driver Portal, see their assigned deliveries, and start shift work.

## Common invitation problems

### Expired invitation link

Invitation links expire after a set period. If a driver or vendor reports that their link does not work, resend the invitation from the relevant management surface (Drivers list for driver invitations, or contact the operations team for vendor invitations).

### Wrong portal link

If a person follows a link intended for a different portal, sign-in will fail or they will be shown an access error. Confirm which portal the invitee should use and provide the correct link.

### Account state problems

If an invitee has previously had an account that was removed, there may be a conflict in the identity provider. Contact the platform operations team to clear the prior state before reissuing the invitation.

### Resending a driver invitation

From the Drivers list in Vendor Admin, find the driver record and use the Resend Invitation action. A new invitation email is sent to the email address on the driver record.
