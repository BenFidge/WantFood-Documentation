---
audience: SystemAdmin
skillKeys: [systemadmin-vendors]
title: Vendor application review and onboarding
---

# Vendor application review and onboarding

This document covers the full vendor application lifecycle from submission through review, approval or rejection, and the onboarding steps that follow approval. It also covers common problems with invitation delivery.

## Application lifecycle

A vendor application moves through the following states:

1. Submitted — the restaurant owner has submitted the application form on the Vendor Admin public apply page.
2. Under review — a System Admin user is reviewing the application.
3. Approved — the application meets platform requirements. An invitation email is sent and the onboarding journey begins.
4. Rejected — the application does not meet platform requirements. The applicant is notified.

## Applications list

The Applications list shows all submitted vendor applications. Use it to:
- See pending applications that have not yet been reviewed.
- Filter by status to find applications at a specific stage.
- Open an application to begin review.

Sort by submission date to work through applications in order of arrival. Older pending applications are higher priority.

## Application details — what to review

The application detail page shows the submitted business information for one vendor. Review:
- Business name, type, and registered address.
- Contact details and the email address that will receive the invitation.
- Any supporting information the applicant has provided.
- The current application state and any previous review notes.

Confirm the information looks complete and legitimate before approving. If anything requires clarification, note it before making a decision.

## Approving a vendor application

1. Open the application from the Applications list.
2. Review the details and confirm the application meets platform requirements.
3. Click Approve.
4. An invitation email is sent to the email address on the application.
5. The application moves to the approved state.

After approval, the applicant follows the invitation link to the Vendor Admin onboarding landing page, completes registration on the onboarding registration page, and reaches the onboarding welcome page. Their Vendor Admin account is created at the end of registration.

## AI menu scan onboarding wizard

System Admin can also onboard a vendor directly from brochure scans using a wizard that combines vendor creation and menu seeding:

1. **Create vendor** from extracted business details and uploaded images.
2. **Create menu** from extracted menu items in the same scan session.

The wizard supports:
- editable business details before save
- preview of extracted content and source images
- re-scan within the same flow
- scan-session caching between steps to reduce repeated AI token usage

On save, the flow creates the vendor, ensures a default branch exists, creates an unpublished menu, and sends the same invitation email pattern used by approved vendor applications.

### Stripe Connect and KYC after approval

Once the vendor has completed onboarding, they need to set up Stripe Connect to accept card payments. This happens through the payment methods configuration in Vendor Admin. The Stripe Connect onboarding flow includes KYC and identity verification steps managed by Stripe directly. Platform admins are not involved in the Stripe Connect steps — the vendor completes them independently.

## Rejecting a vendor application

1. Open the application from the Applications list.
2. Review the details and confirm the reason for rejection.
3. Click Reject.
4. Record a reason for the rejection — this is for internal reference.
5. The application moves to the rejected state.

The applicant is notified of the rejection. Rejected applications are retained in the system for reference but do not block the same applicant from submitting a new application through the normal channel.

## Common problems

### Vendor did not receive the invitation email

1. Open the application in System Admin and confirm the email address is correct.
2. Ask the vendor to check their spam or junk folder.
3. If the email address is wrong, review whether the application can be corrected and the invitation reissued.
4. If the invitation link has expired, contact the platform operations team to arrange a new one.

### Application is stuck in under review state

If an application has been in the under review state for longer than expected without a decision, it may have been overlooked. Open it from the Applications list and either approve, reject, or add a review note recording the reason for the delay.

### Stale application state after a system issue

If an application appears stuck in an unexpected state after a system issue, contact the platform operations team. Do not approve or reject an application if the state appears corrupted — get the state confirmed first.
