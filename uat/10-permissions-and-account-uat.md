# Permissions, Accounts & Authentication UAT — Test Scripts

## Purpose

This document contains test scripts for all **authentication, permissions, invite acceptance, session management,
and account/data lifecycle** behaviours across the WantFood platform.

**Target users**: UAT testers verifying that users can sign in and out correctly, that each persona is restricted to
the right areas, and that the customer account page behaves as expected.

---

## Prerequisites

Before running these tests, ensure you have:

- ✅ Access to **all four** UAT environment URLs (see [00-introduction.md](00-introduction.md#environment-urls))
- ✅ All four test account credentials (see table below)
- ✅ A spare email address that is **not** one of the four test accounts (for invite-mismatch tests)
- ✅ Monday.com board set up (see [01-monday-setup.md](01-monday-setup.md))
- ✅ Browser and device ready — most of these tests should be run on **Chrome and one mobile browser**
- ✅ Screenshot/recording tool ready
- ✅ Confirmed with your project manager that the UAT Entra ID tenant has MFA **not enforced by policy** for the
  standard test accounts (so you can test MFA enrolment deliberately rather than being forced through it on every login)

### Test accounts used in this document

| Role | Email | Portal(s) |
|------|-------|-----------|
| Platform Administrator | `admin.uat@wantfood.com` | System Admin |
| Vendor Manager | `vendor.uat@wantfood.com` | Vendor Admin |
| Delivery Driver | `driver.uat@wantfood.com` | Driver Portal |
| Customer | `customer.uat@wantfood.com` | Customer Front-end |

> **Passwords** will be shared by your project manager via a secure channel before testing begins. Do not share them
> outside the UAT team.

---

## Important: Why This Document Looks Different From the Per-Portal Docs

All four WantFood portals use **Microsoft Entra ID** (formerly Azure Active Directory) for authentication. This means:

- There are **no** local "Sign up", "Forgot password", or "Change password" pages built into the WantFood codebase.
- Password reset, MFA enrolment, and account recovery are all handled on Microsoft's own hosted sign-in pages
  (the ones with the Microsoft logo and the `login.microsoftonline.com` web address).
- WantFood simply redirects you there and brings you back once you're done.

**What does this mean for you as a tester?**

You are not testing Microsoft's sign-in pages — those are Microsoft's responsibility. You *are* testing that:

1. WantFood correctly redirects you *to* Entra ID when you need to sign in.
2. After completing sign-in, password reset, or MFA on the Microsoft pages, WantFood correctly brings you
   *back* to the right place.
3. WantFood enforces the correct permissions: the right people can see the right pages, and the wrong people cannot.

When a test case asks you to click through the Microsoft sign-in screens, step-by-step instructions are provided
below each step so you know what to expect.

> **Tip for first-time testers**: The Microsoft sign-in page may look slightly different depending on your browser and
> whether your organisation uses a custom theme. The button labels and field names described below are the defaults.
> If something looks different, look for the same information under a slightly different label and carry on — it is
> still the same screen.

---

## Table of Contents

1. [Section A — Sign-in Round-trip Per Portal](#section-a--sign-in-round-trip-per-portal)
   - [TC-AC-001: System Admin sign-in and sign-out](#tc-ac-001-system-admin--sign-in-sign-out-and-post-sign-out-protection)
   - [TC-AC-002: Vendor Admin sign-in and sign-out](#tc-ac-002-vendor-admin--sign-in-sign-out-and-post-sign-out-protection)
   - [TC-AC-003: Driver Portal sign-in and sign-out](#tc-ac-003-driver-portal--sign-in-sign-out-and-post-sign-out-protection)
   - [TC-AC-004: Customer Front-end sign-in and sign-out](#tc-ac-004-customer-front-end--sign-in-sign-out-and-post-sign-out-protection)
2. [Section B — Entra ID Password Reset Round-trip](#section-b--entra-id-password-reset-round-trip)
   - [TC-AC-010: Password reset returns user to correct portal](#tc-ac-010-password-reset-returns-user-to-correct-portal)
3. [Section C — Entra ID MFA Enrolment & Second-factor Login](#section-c--entra-id-mfa-enrolment--second-factor-login)
   - [TC-AC-011: First-time MFA enrolment](#tc-ac-011-first-time-mfa-enrolment)
   - [TC-AC-012: Subsequent login with second factor](#tc-ac-012-subsequent-login-with-second-factor)
4. [Section D — Invite Acceptance Flows](#section-d--invite-acceptance-flows)
   - [Cross-references: vendor and driver invite (covered elsewhere)](#cross-references)
   - [TC-AC-020: Invite link expiry](#tc-ac-020-invite-link-expiry--link-has-expired-or-already-been-used)
   - [TC-AC-021: Invite link used by wrong account](#tc-ac-021-invite-link-used-by-a-different-entra-id-account)
5. [Section E — RBAC Matrix and Deep-link Probes](#section-e--rbac-matrix-and-deep-link-probes)
   - [RBAC permissions matrix](#rbac-permissions-matrix)
   - [TC-AC-030 to TC-AC-039: Deep-link probe tests](#tc-ac-030-customer-accessing-system-admin-pages)
6. [Section F — Session Timeout and Token Expiry](#section-f--session-timeout-and-token-expiry)
   - [TC-AC-040: Leave a tab open, attempt action](#tc-ac-040-leave-tab-open-for-more-than-one-hour-then-attempt-an-action)
   - [TC-AC-041: Deep-link to protected page after token expiry](#tc-ac-041-deep-link-to-a-protected-page-after-token-expiry)
   - [TC-AC-042: Sign out in one tab, attempt action in another](#tc-ac-042-sign-out-in-one-tab--attempt-action-in-another)
   - [TC-AC-043: Sign in on two different browsers simultaneously](#tc-ac-043-sign-in-on-two-different-browsers-simultaneously)
7. [Section G — User Management Consequences](#section-g--user-management-consequences)
   - [TC-AC-050: Deleted vendor user can no longer sign in](#tc-ac-050-deleted-vendor-user-can-no-longer-sign-in)
   - [TC-AC-051: Transferred vendor user sees new vendor](#tc-ac-051-transferred-vendor-user-sees-new-vendor-after-next-login)
   - [TC-AC-052: Deleted admin user can no longer sign in](#tc-ac-052-deleted-admin-user-can-no-longer-sign-in)
   - [TC-AC-053: Disabled vendor sees correct message](#tc-ac-053-vendor-admin-user-whose-vendor-is-disabled-sees-correct-message)
   - [TC-AC-054: Vendor-created team member first sign-in](#tc-ac-054-vendor-created-team-member-signs-in-for-the-first-time)
   - [TC-AC-055: Deleted vendor team member cannot sign in](#tc-ac-055-deleted-vendor-team-member-can-no-longer-sign-in)
8. [Section H — GDPR and Data Lifecycle](#section-h--gdpr-and-data-lifecycle)
   - [TC-AC-060: View account information page](#tc-ac-060-view-account-information-page)
   - [TC-AC-061: Request data export](#tc-ac-061-request-data-export--feature-to-be-confirmed)
   - [TC-AC-062: Delete account](#tc-ac-062-delete-account--feature-to-be-confirmed)
   - [TC-AC-063: Notification preferences](#tc-ac-063-notification-preferences--feature-to-be-confirmed)
9. [Section I — Summary and Next Steps](#section-i--summary-and-next-steps)

---

## Section A — Sign-in Round-trip Per Portal

These four tests verify the complete sign-in and sign-out cycle for each portal. Run them before any other testing —
if a sign-in does not work, all other tests for that portal will fail.

---

### TC-AC-001: System Admin — Sign-in, Sign-out, and Post-sign-out Protection

**Given**: You have System Admin credentials (`admin.uat@wantfood.com`) and you are **not** currently signed in to any portal

**Steps**:
1. Open a **new browser window** (not incognito — you want cookies to be retained for the sign-out check).
2. Navigate to the **System Admin UAT URL** (e.g. `https://systemadmin-uat.wantfood.com`).
3. You should be immediately redirected to a Microsoft sign-in page at `login.microsoftonline.com`. If you see the
   WantFood System Admin dashboard directly, you are already signed in — sign out first (see Step 8 below) and
   start again.
4. On the Microsoft sign-in page, type `admin.uat@wantfood.com` into the **Email, phone, or Skype** field and
   click **Next**.
5. On the next screen, enter the password provided by your project manager and click **Sign in**.
6. If you see a screen asking **"Stay signed in?"**, click **No** (this makes the session behave in a predictable
   way for the later session-timeout tests).
7. You should be redirected back to the System Admin dashboard. Confirm:
   - The page title or heading says something like "System Admin" or "Dashboard"
   - The navigation menu includes items such as Vendors, Applications, Orders
   - Your email or name appears somewhere in the top navigation bar
8. **Now test sign-out**: click your profile icon or name in the top-right corner, then click **Sign out** or
   **Log out**.
9. You should be returned to a sign-in page or a public "You have been signed out" screen.
10. Without signing in again, paste the System Admin dashboard URL directly into the browser address bar and press
    Enter.

**Expected Result**:
- Step 3: Microsoft sign-in page loads (URL begins `login.microsoftonline.com`)
- Step 7: System Admin dashboard loads with correct navigation
- Step 9: Session ends cleanly; you see a sign-in page or sign-out confirmation
- Step 10: You are redirected to sign-in again — you cannot see the dashboard while signed out

**Pass Criteria**:
- ✅ Redirect to Entra ID sign-in page happens automatically on first visit
- ✅ Dashboard loads correctly within 3 seconds of completing sign-in
- ✅ Username or email is visible in the top navigation after sign-in
- ✅ Sign-out terminates the session
- ✅ Attempting to re-visit a protected page after sign-out redirects to sign-in

**Edge Cases**:
- Entering the wrong password → should show "Incorrect username or password" on the Microsoft sign-in page (not a
  WantFood error page)
- Pressing the browser Back button immediately after sign-out → the dashboard should not reappear; if it does, log
  as a **High** severity bug
- Opening the System Admin URL in an incognito window whilst already signed in on a normal window → incognito
  should require a fresh sign-in

---

### TC-AC-002: Vendor Admin — Sign-in, Sign-out, and Post-sign-out Protection

**Given**: You have Vendor Admin credentials (`vendor.uat@wantfood.com`) and you are **not** currently signed in

**Steps**:
1. Open a **new browser window**.
2. Navigate to the **Vendor Admin UAT URL** (e.g. `https://vendoradmin-uat.wantfood.com`).
3. You should be redirected to the Microsoft sign-in page at `login.microsoftonline.com`.
4. Enter `vendor.uat@wantfood.com` and click **Next**.
5. Enter the password and click **Sign in**.
6. If prompted **"Stay signed in?"**, click **No**.
7. You should arrive at the Vendor Admin portal. Confirm:
   - The page shows the vendor/restaurant name for the test account
   - Navigation includes items such as Menu, Orders, Drivers
8. **Test sign-out**: click the profile icon or name → **Sign out**.
9. Confirm you are returned to a sign-in or signed-out page.
10. Paste the Vendor Admin dashboard URL into the address bar and press Enter.

**Expected Result**: Same pattern as TC-AC-001 — redirect to Entra ID, dashboard loads on success, session
terminated on sign-out, direct URL access after sign-out requires re-authentication.

**Pass Criteria**:
- ✅ Vendor Admin dashboard loads correctly after sign-in
- ✅ Vendor name shown on dashboard matches the test vendor
- ✅ Sign-out terminates session; direct URL access triggers re-authentication

**Edge Cases**:
- Attempting to sign in with `admin.uat@wantfood.com` (the System Admin account) on the Vendor Admin portal → the
  user should either be blocked with a "Not authorised" message or redirected to an error page — **not** allowed
  in. Log the exact behaviour.
- Attempting to sign in with `driver.uat@wantfood.com` on the Vendor Admin portal → same expectation

---

### TC-AC-003: Driver Portal — Sign-in, Sign-out, and Post-sign-out Protection

**Given**: You have Driver credentials (`driver.uat@wantfood.com`) and you are **not** currently signed in

> **Note**: This test assumes the driver invitation and onboarding have already been completed. If the driver
> account has not been set up yet, run the driver onboarding tests in [04-driver-portal-uat.md](04-driver-portal-uat.md)
> first (TC-DP-001 to TC-DP-004), then return here.

**Steps**:
1. Open a **new browser window**.
2. Navigate to the **Driver Portal UAT URL** (e.g. `https://driver-uat.wantfood.com`).
3. You should be redirected to the Microsoft sign-in page.
4. Enter `driver.uat@wantfood.com` and click **Next**.
5. Enter the password and click **Sign in**.
6. If prompted **"Stay signed in?"**, click **No**.
7. You should arrive at the Driver Portal dashboard. Confirm:
   - The page shows the driver's name or a welcome message
   - Navigation includes items such as Active Deliveries, Availability, or Shifts
8. **Test sign-out**: click the profile icon or name → **Sign out**.
9. Confirm you are returned to a sign-in or signed-out page.
10. Paste the Driver Portal dashboard URL into the address bar.

**Expected Result**: Same sign-in/sign-out pattern. Direct URL access after sign-out requires re-authentication.

**Pass Criteria**:
- ✅ Driver Portal dashboard loads correctly after sign-in
- ✅ Sign-out terminates session
- ✅ Direct URL access after sign-out triggers re-authentication

**Edge Cases**:
- Attempting to sign in with `customer.uat@wantfood.com` on the Driver Portal → should be blocked with a "Not
  authorised" or access denied message, not a crash
- If the driver account has no active assignments, confirm the portal shows an appropriate empty state rather than
  an error

---

### TC-AC-004: Customer Front-end — Sign-in, Sign-out, and Post-sign-out Protection

**Given**: You have Customer credentials (`customer.uat@wantfood.com`) and you are **not** currently signed in

**Steps**:
1. Open a **new browser window**.
2. Navigate to the **Customer Front-end UAT URL** (e.g. `https://uat.wantfood.com`).
3. The home page should load **without requiring sign-in** — browsing is available to anyone. Confirm you can see
   the restaurant listing or home page.
4. Click the **Sign in** link (usually top-right of the navigation bar or on the account icon).
5. You should be redirected to the Microsoft sign-in page.
6. Enter `customer.uat@wantfood.com` and click **Next**.
7. Enter the password and click **Sign in**.
8. If prompted **"Stay signed in?"**, click **No**.
9. You should be returned to the Customer Front-end, now signed in. Confirm:
   - Your name or a profile icon with your initial appears in the navigation
   - You can navigate to **Account** (`/Account`) and see your profile details
10. **Test sign-out**: navigate to Account → click **Log Out** (the orange button at the bottom of the account page).
11. Confirm you are returned to a signed-out state (account icon changes to a sign-in prompt, or you are redirected
    to the home page).
12. Navigate directly to `https://uat.wantfood.com/Account` (your account page).

**Expected Result**:
- Step 3: Home page loads without authentication
- Step 9: Signed-in state visible; account page shows display name and email
- Step 11: Sign-out completes cleanly
- Step 12: Attempting to access the account page while signed out redirects to sign-in

**Pass Criteria**:
- ✅ Customer can browse the home page without signing in
- ✅ After sign-in, account page shows correct name and email
- ✅ Sign-out works and session is terminated
- ✅ Direct URL to account page after sign-out triggers re-authentication

**Edge Cases**:
- Sign out, then press browser Back button → page should not show account data; if it does, log a **High** severity
  bug
- Sign in as customer, then try navigating to `/Admin` or any admin-looking path → should receive a 404 or 403
  page, not an admin screen (covered in more detail in Section E)
- Customer signs in on mobile browser → confirm sign-in flow works on iOS Safari and Android Chrome

---

## Section B — Entra ID Password Reset Round-trip

These tests verify that a user can use the "Forgot password" link on the Microsoft sign-in page and be returned
correctly to WantFood after completing the reset.

> **Before running TC-AC-010**: Confirm with your project manager that the Entra ID tenant has **Self-service
> password reset (SSPR)** enabled. If it is not enabled, this test cannot be run and should be marked as
> **"Skipped — SSPR not configured in this environment"** in your Monday.com board.

---

### TC-AC-010: Password Reset Returns User to Correct Portal

**Given**: You are signed out of all portals. You have access to the inbox for `customer.uat@wantfood.com` (or a
test account confirmed by your PM to support SSPR).

**Steps**:
1. Navigate to the **Customer Front-end UAT URL**.
2. Click **Sign in**.
3. On the Microsoft sign-in page, enter the email address and click **Next**.
4. On the password screen, look for a link that says **"Forgot my password"** or **"I forgot my password"** and
   click it.
   > The link is usually below the password field or on the sign-in options page. If you cannot find it, look for
   > **"Sign-in options"** and then **"Forgot my password"**.
5. Microsoft will ask you to verify your identity. Follow the on-screen prompts:
   - If asked for a verification code by email, check the inbox for the email address you used and enter the code
     shown in the email.
   - If asked for a verification code by phone/SMS, enter the number registered to the account and enter the code
     you receive.
6. When prompted, enter a **new password** (use a memorable test password agreed with your PM; note it down).
7. Click **Finish** or **Reset password**.
8. Microsoft should display a confirmation message ("Your password has been changed"). Look for a button that says
   **"Click here to sign in"** or a countdown that takes you back automatically.
9. Click through to sign in. You should be redirected back to WantFood's sign-in flow and then land on the **Customer
   Front-end** — not a blank page, not `login.microsoftonline.com`, and not the wrong portal.
10. Confirm the Customer Front-end home page loads and your account icon is visible.

**Expected Result**:
- The "Forgot password" link is present on the Microsoft sign-in page
- The SSPR flow completes without errors
- After the password reset, the user is returned to the WantFood Customer Front-end (not stranded on a Microsoft page)
- The user is signed in (or asked to sign in once more using the new password) on the correct portal

**Pass Criteria**:
- ✅ "Forgot my password" link is visible on the Microsoft sign-in page
- ✅ Identity verification step is presented and accepts the correct code
- ✅ New password is accepted
- ✅ After reset, user arrives back on the Customer Front-end
- ✅ User can sign in with the new password
- ✅ No WantFood error page appears at any point in the flow

**Edge Cases**:
- Entering an invalid verification code → Microsoft should show an error and allow retry; WantFood should not
  be involved at this point
- Cancelling out of the SSPR flow mid-way → clicking the browser Back button or "Cancel" on Microsoft's page
  should return the user to the WantFood sign-in prompt, not a blank page
- Repeat this test on **one other portal** (e.g. System Admin) to confirm the redirect-back-to-portal behaviour
  is consistent regardless of which portal initiated the sign-in

---

## Section C — Entra ID MFA Enrolment & Second-factor Login

These tests verify that the multi-factor authentication (MFA) enrolment and second-factor login flows work
correctly for WantFood users.

> **Before running TC-AC-011**: Confirm with your project manager that:
> 1. A dedicated **MFA test account** has been provided (separate from the standard test accounts above, to avoid
>    disrupting other test sessions).
> 2. You have access to the mobile phone number or authenticator app registered to that account.
> 3. MFA registration is not yet completed for that account — or it has been deliberately reset by the PM.

---

### TC-AC-011: First-time MFA Enrolment

**Given**: You have a test account that has **not yet** enrolled in MFA, and MFA is configured as required or
prompted in the Entra ID tenant. Your PM has provided the account details and a way to receive the verification
(e.g. a phone number you can receive SMS on, or the Microsoft Authenticator app is installed on your device).

**Steps**:
1. Open a **new browser window**.
2. Navigate to the **System Admin UAT URL**.
3. On the Microsoft sign-in page, enter the MFA test account email and click **Next**.
4. Enter the password and click **Sign in**.
5. Microsoft will display a screen that says **"More information required"** or **"Keep your account secure"** with
   a **Next** button. Click **Next**.
6. You will be offered one or more methods for setting up a second factor. Choose the method your PM has set up for
   the test account:
   - **Authenticator app**: Follow the on-screen instructions to scan the QR code with the Microsoft Authenticator
     app on your phone. Once scanned, the app will display a code — enter it when prompted.
   - **Phone (SMS)**: Enter the phone number you have access to, click **Next**, and enter the 6-digit code sent by
     SMS.
7. Click **Next** or **Done** on the confirmation screen.
8. If prompted **"Stay signed in?"**, click **No**.
9. You should now arrive at the System Admin dashboard.

**Expected Result**:
- The MFA setup screens appear after entering valid credentials
- The chosen verification method works (app code or SMS code is accepted)
- After enrolment, WantFood loads the correct portal dashboard — not a blank or error page

**Pass Criteria**:
- ✅ MFA enrolment screens appear correctly in the sign-in flow
- ✅ Verification (QR scan or SMS) completes without error
- ✅ User arrives at the portal dashboard after completing enrolment
- ✅ No WantFood-branded error page appears during the flow

**Edge Cases**:
- Entering an incorrect or expired SMS code → Microsoft should show an error and offer a retry or re-send option
- Closing the browser mid-enrolment and reopening → Microsoft may re-present the enrolment prompt on next sign-in;
  verify this does not loop infinitely

---

### TC-AC-012: Subsequent Login With Second Factor

**Given**: The MFA test account from TC-AC-011 has completed enrolment and is currently signed out

**Steps**:
1. Open a **new browser window**.
2. Navigate to the **System Admin UAT URL**.
3. On the Microsoft sign-in page, enter the MFA test account email and click **Next**.
4. Enter the password and click **Sign in**.
5. Microsoft will present a second-factor challenge — either:
   - **Authenticator app**: A prompt appears in the Microsoft Authenticator app on your phone. Approve it.
   - **SMS**: A 6-digit code is sent to the registered phone number. Enter it in the field shown.
6. Complete the challenge and click **Verify** or **Sign in**.
7. You should arrive at the System Admin dashboard.

**Expected Result**:
- The second-factor challenge is presented after the correct password is entered
- The challenge can be completed using the enrolled method
- After passing the second factor, WantFood loads the correct portal dashboard

**Pass Criteria**:
- ✅ Second-factor challenge appears for the enrolled account
- ✅ Correct approval/code passes the challenge
- ✅ Portal dashboard loads after MFA challenge is completed
- ✅ User is not asked to set up MFA again (enrolment is retained)

**Edge Cases**:
- Approving the authenticator prompt on the wrong device or from a stale notification → Microsoft should reject
  the approval; verify WantFood does not get into a broken state
- Entering a valid code but waiting longer than the code's validity window (usually 30 seconds) → code should be
  rejected; verify user can request a new code

---

## Section D — Invite Acceptance Flows

### Cross-references

The standard invite acceptance flows are covered in other UAT documents. Before testing the new edge cases
below, confirm the happy-path flows have passed:

| Flow | Where to find it |
|------|-----------------|
| Vendor receives and accepts onboarding invite from System Admin | [02-system-admin-uat.md](02-system-admin-uat.md) — TC-SA-022; [03-vendor-admin-uat.md](03-vendor-admin-uat.md) — TC-VA-003 |
| Driver receives and accepts invitation from Vendor Admin | [04-driver-portal-uat.md](04-driver-portal-uat.md) — TC-DP-001, TC-DP-002 |

The two test cases below test **what happens when invite links are not used correctly**. They are independent of
the happy-path tests and can be run in any order after the happy-path cases have passed.

---

### TC-AC-020: Invite Link Expiry — Link Has Expired or Already Been Used

**Given**: A vendor invite or driver invite link exists that is **either**:
- More than the permitted number of hours/days old (ask your PM what the invite expiry period is), **or**
- Has already been used to complete onboarding once

> **How to get an expired or used link**: Either wait for a previous invite to expire, or use the driver invite
> link that was already accepted during TC-DP-002. Copy that link before completing TC-DP-002 and save it for use
> here.

**Steps**:
1. Open a **new browser window** (or incognito window to ensure you are not already signed in as the invitee).
2. Paste the expired or already-used invite link into the browser address bar and press **Enter**.
3. Note what happens:
   - You may be asked to sign in to Entra ID first — if so, complete the sign-in using the invited email address.
   - After sign-in (or immediately if you were already signed in), note the page or message displayed.

**Expected Result**:
- The user is **not** allowed to complete onboarding again using an expired or already-used link
- A clear, friendly error message is displayed explaining that the invitation link is no longer valid (e.g.
  "This invitation link has expired" or "This invitation has already been accepted")
- The user is offered a sensible next step (e.g. a link to sign in normally, or a prompt to contact their manager
  to request a new invite)
- No sensitive data (e.g. the invited email address or internal IDs) is exposed in the error message

**Pass Criteria**:
- ✅ Expired/used link does not allow onboarding to be completed
- ✅ Error message is displayed in plain English
- ✅ Error message does not contain technical error codes or stack traces
- ✅ User is given a clear next step

**Edge Cases**:
- Try the expired link whilst signed in as a **different** account (e.g. `admin.uat@wantfood.com`) → confirm it
  is still rejected and does not inadvertently associate the wrong user with the invite
- Try the link in an incognito window with no cached session → should prompt sign-in then show the expiry error

---

### TC-AC-021: Invite Link Used by a Different Entra ID Account

**Given**: A valid, unexpired invite link has been sent to `driver.uat@wantfood.com` (or a freshly generated test
invite — co-ordinate with your PM). You have a **different** test email address that is registered in Entra ID
but was **not** the invited address.

**Steps**:
1. Open an **incognito browser window** to ensure you start signed out.
2. Paste the valid invite link into the address bar and press **Enter**.
3. The system will redirect you to the Microsoft sign-in page. Sign in using the **wrong** account — i.e., a
   different email address from the one that was invited. Use the spare test email address agreed with your PM.
4. After completing sign-in with the wrong account, note what happens.

**Expected Result**:
- The system recognises that the signed-in Entra ID account does not match the invited email address
- The user is shown a clear, friendly error message explaining the mismatch (e.g. "This invitation was sent to a
  different email address. Please sign in with the account that received the invitation.")
- The onboarding is **not** completed for the wrong account
- The invite link remains valid so the correct recipient can still use it

**Pass Criteria**:
- ✅ Signing in with the wrong Entra ID account is detected
- ✅ Clear error message is shown — no silent acceptance of the wrong account
- ✅ Onboarding is not associated with the wrong account
- ✅ The invite link is not consumed/invalidated by the failed attempt

**Edge Cases**:
- Retry the same link with the **correct** account after the failed attempt — confirm the invite still works
  normally and the onboarding completes successfully

---

## Section E — RBAC Matrix and Deep-link Probes

### RBAC Permissions Matrix

The table below summarises which personas should be able to access which portals. Use this as a reference
when running TC-AC-030 to TC-AC-039.

| Persona | System Admin portal | Vendor Admin portal | Driver Portal | Customer Front-end |
|---------|:-------------------:|:-------------------:|:-------------:|:------------------:|
| **Platform Administrator** (`admin.uat@wantfood.com`) | ✅ Full access | ❌ Blocked | ❌ Blocked | ✅ Browse as customer |
| **Vendor Manager** (`vendor.uat@wantfood.com`) | ❌ Blocked | ✅ Own vendor only | ❌ Blocked | ✅ Browse as customer |
| **Driver** (`driver.uat@wantfood.com`) | ❌ Blocked | ❌ Blocked | ✅ Own assignments | ✅ Browse as customer |
| **Customer** (`customer.uat@wantfood.com`) | ❌ Blocked | ❌ Blocked | ❌ Blocked | ✅ Full customer access |
| **Not signed in (anonymous)** | ❌ Blocked | ❌ Blocked | ❌ Blocked | ✅ Browse (public pages only) |

> **"Blocked"** means the user should be either:
> - Redirected to a sign-in page (if they are not signed in on that portal), **or**
> - Shown an "Access denied" / 403 / "Not authorised" page (if they are signed in but with the wrong role)
>
> In both cases the user must **not** see any admin content or be able to take any action.

---

### TC-AC-030: Customer Accessing System Admin Pages

**Given**: You are signed in to the Customer Front-end as `customer.uat@wantfood.com`

**Steps**:
1. Whilst signed in as the customer, open a **new tab** in the same browser.
2. Navigate to the System Admin UAT URL (e.g. `https://systemadmin-uat.wantfood.com`).
3. Note the result (redirect to Entra ID? Access denied page? 403 error?).
4. If you are taken to a Microsoft sign-in page, do **not** sign in — close the tab. The test passes at step 3
   as long as the System Admin content is not shown.

**Expected Result**:
- The customer is redirected to sign-in or shown an access-denied page — they cannot see any System Admin content

**Pass Criteria**:
- ✅ System Admin dashboard is not shown to a customer
- ✅ No admin data (vendor list, orders, commissions, etc.) is visible to the customer

**Edge Cases**:
- Try a specific deep link, such as `https://systemadmin-uat.wantfood.com/Admin/Vendors` → same expectation;
  must not show vendor data

---

### TC-AC-031: Customer Accessing Vendor Admin Pages

**Given**: You are signed in to the Customer Front-end as `customer.uat@wantfood.com`

**Steps**:
1. Open a new tab.
2. Navigate to the Vendor Admin UAT URL (e.g. `https://vendoradmin-uat.wantfood.com`).
3. Note the result.

**Expected Result**: Customer is redirected to sign-in or shown an access-denied page — Vendor Admin content is not visible.

**Pass Criteria**:
- ✅ Vendor Admin dashboard is not shown to a customer

---

### TC-AC-032: Customer Accessing Driver Portal Pages

**Given**: You are signed in to the Customer Front-end as `customer.uat@wantfood.com`

**Steps**:
1. Open a new tab.
2. Navigate to the Driver Portal UAT URL (e.g. `https://driver-uat.wantfood.com`).
3. Note the result.

**Expected Result**: Customer is redirected to sign-in or shown an access-denied page — Driver Portal content is not visible.

**Pass Criteria**:
- ✅ Driver Portal dashboard is not shown to a customer

---

### TC-AC-033: Vendor Manager Accessing System Admin Pages

**Given**: You are signed in to the Vendor Admin portal as `vendor.uat@wantfood.com`

**Steps**:
1. Open a new tab.
2. Navigate to the System Admin UAT URL.
3. Note the result.
4. If you are redirected to a Microsoft sign-in page, **do not sign in**. The portal access is correctly blocked.
5. Additionally, try a specific admin deep link: `https://systemadmin-uat.wantfood.com/Admin/Vendors`.

**Expected Result**: Vendor Manager cannot access any System Admin page. Either redirected to Entra ID sign-in
(if the System Admin portal treats this as a different session) or shown a 403/access-denied page.

**Pass Criteria**:
- ✅ System Admin content is not shown to a Vendor Manager
- ✅ The vendor data management pages are not accessible

---

### TC-AC-034: Driver Accessing Vendor Admin Pages

**Given**: You are signed in to the Driver Portal as `driver.uat@wantfood.com`

**Steps**:
1. Open a new tab.
2. Navigate to the Vendor Admin UAT URL.
3. Note the result.
4. Also try: `https://vendoradmin-uat.wantfood.com/Admin/Menu`.

**Expected Result**: Driver is blocked from accessing Vendor Admin portal content.

**Pass Criteria**:
- ✅ Vendor Admin dashboard is not shown to a driver
- ✅ Menu management pages are not accessible to the driver

---

### TC-AC-035: Driver Accessing System Admin Pages

**Given**: You are signed in to the Driver Portal as `driver.uat@wantfood.com`

**Steps**:
1. Open a new tab.
2. Navigate to the System Admin UAT URL.
3. Also try: `https://systemadmin-uat.wantfood.com/Admin/Orders`.

**Expected Result**: Driver is blocked from accessing System Admin content.

**Pass Criteria**:
- ✅ System Admin dashboard and orders pages are not shown to a driver

---

### TC-AC-036: Vendor Manager Accessing Another Vendor's Data (Same Portal, Different Vendor)

**Given**: You are signed in to the Vendor Admin portal as `vendor.uat@wantfood.com`. You know the ID of a
**second test vendor** (ask your PM to provide a second vendor ID for this test).

**Steps**:
1. In the Vendor Admin portal, note the URL whilst on the dashboard — it will contain a vendor ID or slug.
2. Manually edit the URL to replace the current vendor's ID with the second vendor's ID (e.g. change
   `vendoradmin-uat.wantfood.com/Admin/Dashboard?vendorId=AAA` to use `vendorId=BBB`).
3. Press **Enter** and note the result.
4. Also try navigating to the second vendor's menu: e.g. change the vendor ID in a menu management URL.

**Expected Result**: The Vendor Manager can only see their own vendor's data. Attempting to access another
vendor's pages should result in an access-denied page, a 403, or a redirect to their own vendor's dashboard.
The second vendor's data must not be visible.

**Pass Criteria**:
- ✅ A vendor manager cannot view or modify another vendor's data by manipulating the URL
- ✅ Either an access-denied page is shown, or the user is silently redirected to their own vendor

**Edge Cases**:
- Try modifying the vendor ID in a **POST** action (e.g. update a menu item using another vendor's item ID) →
  this requires slightly more technical effort; ask your PM whether this level of testing is in scope for UAT

---

### TC-AC-037: Anonymous User Accessing Protected Customer Pages

**Given**: You are **not signed in** to any portal

**Steps**:
1. Open an incognito browser window.
2. Navigate directly to `https://uat.wantfood.com/Account` (the customer account page).
3. Note the result.
4. Also try: `https://uat.wantfood.com/Checkout`.
5. Also try: `https://uat.wantfood.com/Orders`.

**Expected Result**:
- The account page (`/Account`) requires sign-in — anonymous users are redirected to Entra ID
- The checkout page (`/Checkout`) may redirect to sign-in, or may allow browsing up to a point before requiring sign-in
- The orders page (`/Orders`) requires sign-in
- Public pages (home, restaurant listings, search) should load without sign-in

**Pass Criteria**:
- ✅ `/Account` redirects to sign-in when accessed anonymously
- ✅ `/Orders` redirects to sign-in when accessed anonymously
- ✅ Home page and restaurant listings are accessible without sign-in

---

### TC-AC-038: Platform Admin Cannot See Other Vendor Admin Users' Data

> This is a server-side access check. If your PM has provided two vendor accounts linked to different vendors,
> this test verifies the System Admin can see both but a vendor can only see their own.

**Given**: You are signed in to the System Admin portal as `admin.uat@wantfood.com`. Two test vendors exist.

**Steps**:
1. In System Admin, navigate to **Vendors** and note both test vendors are listed.
2. Click into Vendor A, then click into Vendor B — confirm the admin can access both.
3. Sign out of System Admin.
4. Sign in to Vendor Admin as `vendor.uat@wantfood.com` (linked to Vendor A only).
5. Note: Vendor A's data is visible. Vendor B's data should not be accessible.
6. Attempt to navigate to a Vendor B menu or order page by modifying the URL (as in TC-AC-036).

**Expected Result**: Platform Admin can view all vendors. Vendor Manager can only view their own vendor.

**Pass Criteria**:
- ✅ System Admin can navigate between Vendor A and Vendor B
- ✅ Vendor Manager cannot access Vendor B's data

---

### TC-AC-039: Verify 403 / Access-denied Page Is Friendly

**Given**: You have induced an access-denied response (by running any of TC-AC-030 to TC-AC-038 where a block
was triggered)

**Steps**:
1. Note the page that was displayed when access was blocked.
2. Check the following:
   - Does the page show a friendly message in plain English (e.g. "You don't have permission to view this page")?
   - Does the page show a **Back** button, a **Sign in** link, or another helpful option?
   - Does the page show a stack trace, technical error message, or internal IDs?

**Expected Result**: All access-denied responses should be user-friendly. No stack traces or internal error
details should be visible.

**Pass Criteria**:
- ✅ Access-denied page uses plain English
- ✅ No stack trace or technical error text is shown to the user
- ✅ At least one helpful action is available (sign in link, return to home, or contact support)

---

## Section F — Session Timeout and Token Expiry

These tests verify that when a user's Entra ID token expires, WantFood handles the situation gracefully — either
re-prompting for sign-in or showing a friendly message — rather than showing a cryptic error or silently failing.

> **Token expiry timing**: Entra ID tokens typically expire after **1 hour** by default. Some UAT environments
> may be configured with shorter expiry periods for easier testing. Ask your PM what the token lifetime is in
> your UAT tenant before running these tests. If the lifetime is 1 hour, you will need to leave the browser idle
> for the appropriate time.

---

### TC-AC-040: Leave Tab Open for More Than One Hour, Then Attempt an Action

**Given**: You are signed in to the **System Admin** portal as `admin.uat@wantfood.com`

**Steps**:
1. Sign in to the System Admin portal.
2. Leave the browser tab open and **do not interact with it** for the duration of the configured token lifetime
   (e.g. 1 hour). You can work on other tests in separate windows during this time.
3. After the token should have expired, return to the System Admin tab.
4. Try to perform an action that requires server communication, such as navigating to **Vendors** or clicking
   **Refresh** on the dashboard.
5. Note what happens.

**Expected Result**:
- The user is either automatically redirected to the Microsoft sign-in page (and then returned to the System Admin
  page they were on after signing in), **or**
- A friendly message is displayed explaining that the session has expired, with a button to sign in again
- The user is **not** shown a blank page, a cryptic error, or a JSON error response

**Pass Criteria**:
- ✅ Expired session is handled gracefully — no unhandled error shown to the user
- ✅ User can sign in again and return to the page they were trying to access (or at least to the dashboard)
- ✅ No data is lost (e.g. a form the user was completing should ideally warn them before the session expires,
  though displaying a sign-in prompt after submission failure is acceptable as a minimum)

**Edge Cases**:
- If your PM confirms the token lifetime is too long to test manually, ask whether the tenant can be reconfigured
  to use a 5-minute lifetime for UAT — this will make the test much faster to run

---

### TC-AC-041: Deep-link to a Protected Page After Token Expiry

**Given**: Your Entra ID token has expired (wait as per TC-AC-040, or ask your PM to revoke the session from the
Entra ID admin panel)

**Steps**:
1. Open a **new tab** (do not click anything in the signed-in System Admin tab).
2. Paste a direct URL to a protected page into the address bar, such as
   `https://systemadmin-uat.wantfood.com/Admin/Vendors`.
3. Press **Enter**.
4. Complete sign-in if redirected to Microsoft.
5. Note where you land after signing in.

**Expected Result**:
- You are redirected to Microsoft sign-in
- After signing in, you are returned to the page you tried to visit (`/Admin/Vendors`), **not** to the dashboard
  root — this is called **"post-login redirect"** and is an important usability feature

**Pass Criteria**:
- ✅ Deep link to a protected page triggers sign-in when token has expired
- ✅ After sign-in, user lands on the originally requested page (or, at minimum, the dashboard — not a 404 or error)

---

### TC-AC-042: Sign Out in One Tab, Attempt Action in Another

**Given**: You are signed in to the **Vendor Admin** portal in two browser tabs side by side

**Steps**:
1. Open the Vendor Admin portal in **Tab 1**.
2. Open the Vendor Admin portal in **Tab 2** (same browser, not incognito).
3. In **Tab 1**, sign out using the sign-out link.
4. Confirm Tab 1 shows a sign-out confirmation or the sign-in page.
5. Switch to **Tab 2** — the page should still look like you are signed in (the tab hasn't refreshed yet).
6. In **Tab 2**, try to perform an action — for example, navigate to **Orders** or click **Refresh**.
7. Note what happens.

**Expected Result**:
- After signing out in Tab 1, Tab 2 should eventually detect the sign-out and either:
  - Redirect to sign-in on the next navigation or page action, **or**
  - Show an "Your session has ended" message with a sign-in link
- Tab 2 should **not** successfully load any new vendor data after sign-out in Tab 1

**Pass Criteria**:
- ✅ Signed-out session prevents fresh data from loading in the other tab
- ✅ No vendor data is returned after the session has been terminated
- ✅ User is not stuck on a broken page — they can sign in again from Tab 2

---

### TC-AC-043: Sign In on Two Different Browsers Simultaneously

**Given**: You have access to two different browsers (e.g. Chrome and Edge) on the same machine

**Steps**:
1. In **Chrome**, sign in to the Customer Front-end as `customer.uat@wantfood.com`.
2. In **Edge**, sign in to the Customer Front-end as `customer.uat@wantfood.com` using the same account.
3. Confirm both browsers show you as signed in and can access the account page.
4. In **Chrome**, sign out.
5. In **Edge**, attempt to navigate to `/Account`.

**Expected Result**:
- Both browsers can be signed in simultaneously without conflict
- Signing out in one browser does not immediately invalidate the session in the other (Entra ID supports
  multiple concurrent sessions by default)
- If the Edge session is still valid, the account page loads normally

**Pass Criteria**:
- ✅ Concurrent sessions in two browsers work without errors
- ✅ No data corruption or mixing of sessions

---

## Section G — User Management Consequences

These tests verify that when a System Admin makes a user management change (deleting a user, transferring a
vendor user, or disabling a vendor), the affected user's next sign-in reflects the change. These tests are
paired with System Admin test cases — run those **first**, then run the corresponding test here.

---

### TC-AC-050: Deleted Vendor User Can No Longer Sign In

**Paired with**: TC-SA-042 (System Admin → Vendor Management → Delete Vendor User)

**Given**: TC-SA-042 has been completed and a vendor user (`vendor-deleted.uat@wantfood.com` or equivalent — your
PM will provide a deletable test account) has been deleted in the System Admin portal

**Steps**:
1. Sign out of all portals.
2. Open an incognito browser window.
3. Navigate to the **Vendor Admin UAT URL**.
4. On the Microsoft sign-in page, enter the deleted user's email and password.
5. Complete the Entra ID sign-in.
6. Note what happens after sign-in completes.

**Expected Result**:
- After successfully authenticating with Entra ID, WantFood should **not** grant access to the Vendor Admin portal
  for the deleted user
- The user should see an access-denied message or be redirected to a "Your account is no longer active" page

> **Note**: Microsoft Entra ID holds the identity, but WantFood holds the authorisation (the user's role and vendor
> association). Deleting the user in WantFood removes their authorisation. The Entra ID account itself may still
> exist, which is why the user may pass the Entra ID sign-in step but then be blocked by WantFood.

**Pass Criteria**:
- ✅ Deleted vendor user cannot access the Vendor Admin dashboard after deletion
- ✅ A clear, friendly message is shown (not a crash or blank screen)

**Edge Cases**:
- Deleted user tries to access the **Customer Front-end** — if their Entra ID account still exists, they may be
  able to browse as a customer. Confirm whether this is the expected behaviour with your PM.

---

### TC-AC-051: Transferred Vendor User Sees New Vendor After Next Login

**Paired with**: TC-SA-041 (System Admin → Vendor Management → Transfer Vendor User to New Vendor)

**Given**: TC-SA-041 has been completed and a vendor user has been transferred from Vendor A to Vendor B

**Steps**:
1. Sign out of the Vendor Admin portal as the transferred user.
2. Sign back in using the same credentials.
3. Observe the vendor name and data shown on the Vendor Admin dashboard.

**Expected Result**:
- After signing back in, the user now sees **Vendor B's** data (the new vendor they were transferred to) rather
  than Vendor A's
- Menu items, orders, and settings shown belong to Vendor B

**Pass Criteria**:
- ✅ After re-login, the correct (new) vendor's data is shown
- ✅ No data from the old vendor (Vendor A) is visible in the main dashboard

**Edge Cases**:
- If the user had the Vendor Admin portal open in a tab before the transfer, confirm that navigating to a new page
  after the transfer shows the correct vendor (not stale data from Vendor A)

---

### TC-AC-052: Deleted Admin User Can No Longer Sign In

**Paired with**: TC-SA-052 (System Admin → Admin User Management → Delete Admin User)

**Given**: TC-SA-052 has been completed and an admin user (`admin-deleted.uat@wantfood.com` or equivalent — your
PM will provide a deletable admin test account) has been deleted in the System Admin portal

**Steps**:
1. Sign out of all portals.
2. Open an incognito browser window.
3. Navigate to the **System Admin UAT URL**.
4. Enter the deleted admin user's email and password on the Microsoft sign-in page.
5. Complete the Entra ID sign-in.
6. Note what happens.

**Expected Result**:
- The deleted admin user cannot access the System Admin portal
- A friendly access-denied message is displayed

**Pass Criteria**:
- ✅ Deleted admin user is blocked from accessing System Admin
- ✅ Friendly error message is shown

---

### TC-AC-053: Vendor Admin User Whose Vendor Is Disabled Sees Correct Message

**Given**: A test vendor (`vendor-disabled.uat@wantfood.com` or equivalent) has been disabled in the System Admin
portal (ask your PM to disable a test vendor before running this test)

**Steps**:
1. Sign out of all portals.
2. Open an incognito window.
3. Navigate to the **Vendor Admin UAT URL**.
4. Sign in using the disabled vendor's credentials.
5. Note the page or message shown after sign-in.

**Expected Result**:
- The vendor user can authenticate with Entra ID
- But WantFood shows a message indicating their vendor account has been suspended, disabled, or is no longer active
- They cannot access the Vendor Admin dashboard or take any actions

**Pass Criteria**:
- ✅ Disabled vendor's user cannot access the Vendor Admin portal
- ✅ The message is clear and friendly — it explains the account is disabled (not a generic error)
- ✅ User is given a contact point (e.g. "Please contact your WantFood account manager")

---

### TC-AC-054: Vendor-created Team Member Signs In for the First Time

**Paired with**: TC-VA-077 (Vendor Admin → Team → Add User)

**Given**: TC-VA-077 has been completed and a new team member account was created. You have the temporary
password that was shown on the Account Created screen.

**Steps**:
1. Open an incognito browser window.
2. Navigate to the **Vendor Admin UAT URL**.
3. On the Microsoft sign-in page, enter the new user's email address.
4. Enter the **temporary password** copied from the Account Created screen.
5. Microsoft Entra will prompt you to **set a new password** — enter a new secure password and confirm it.
6. Complete the sign-in flow.
7. Confirm the Vendor Admin dashboard loads.

**Expected Result**:
- The temporary password is accepted on first sign-in.
- Microsoft Entra prompts for a new password (forced change on first sign-in).
- After setting a new password, the Vendor Admin dashboard loads correctly.
- The new user can navigate to core pages (Team, Orders, Menu).

**Pass Criteria**:
- ✅ Temporary password accepted by Microsoft Entra
- ✅ New password prompt shown on first sign-in
- ✅ Vendor Admin dashboard loads after password change
- ✅ New user can access Vendor Admin features

**Edge Cases**:
- Entering the wrong temporary password → Entra should show an "incorrect credentials" error (not a WantFood error)
- Entering a new password that does not meet complexity requirements → Entra shows its own password-policy error

---

### TC-AC-055: Deleted Vendor Team Member Can No Longer Sign In

**Paired with**: TC-VA-079a (Vendor Admin → Team → Delete)

**Given**: TC-VA-079a has been completed and a test team member has been deleted via the Vendor Admin Team list.

**Steps**:
1. Sign out of all portals.
2. Open an incognito browser window.
3. Navigate to the **Vendor Admin UAT URL**.
4. On the Microsoft sign-in page, enter the deleted team member's email address.
5. Attempt to sign in using the deleted user's known password.
6. Note the result.

**Expected Result**:
- The deleted team member's Entra ID account no longer exists.
- Microsoft Entra shows a sign-in error (account not found / incorrect credentials).
- The user cannot reach the Vendor Admin portal.

> **Note**: Unlike vendor user deletion by System Admin (TC-AC-050), where the Entra account may persist and
> only WantFood authorisation is removed, vendor team member deletion also removes the underlying Entra account.
> The sign-in failure therefore happens at the Entra level, before WantFood is even reached.

**Pass Criteria**:
- ✅ Sign-in attempt fails at the Microsoft Entra sign-in page
- ✅ Deleted user cannot reach the Vendor Admin dashboard

**Edge Cases**:
- If the deleted user is currently signed in (active session) when they are deleted → their session remains valid
  until it expires (up to 1 hour). Confirm that after the session expires they cannot sign in again.

---

## Section H — GDPR and Data Lifecycle

This section covers the customer account page and data self-service features. Before running these tests,
please read the notes below about which features are present in the current build.

### Feature Presence Summary

Based on a review of the codebase at the time this document was written, the following features were
identified:

| Feature | Status in current build |
|---------|------------------------|
| View account information (name, email, phone) | ✅ **Implemented** — `/Account` page |
| Manage saved addresses | ✅ **Implemented** — `/Account/Addresses` page |
| Notification preferences | ⚠️ **UI link exists but is not yet connected** — the "Notification Preferences" menu item on the account page currently goes nowhere. See TC-AC-063. |
| Request data export | ❌ **Not present in current build** — see TC-AC-061 |
| Delete account (self-service) | ❌ **Not present in current build** — see TC-AC-062 |

> **If features have been added since this document was written**, update the table above and remove the
> "FEATURE TO BE CONFIRMED" heading from the relevant test case.

---

### TC-AC-060: View Account Information Page

**Given**: You are signed in to the Customer Front-end as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `https://uat.wantfood.com/Account` or click the profile icon in the navigation bar and select
   **Account**.
2. Review the page that loads.
3. Check the **Personal Details** section — note whether clicking it opens a dedicated edit page or stays on the
   same page.
4. Check the **Saved Addresses** link — click it.
5. On the Saved Addresses page, add a new address:
   - Click **Add Address** (or equivalent button)
   - Fill in a test address (e.g. "123 Test Street, London, EC1A 1BB")
   - Save the address
6. Return to the account page. Confirm the address is now listed.
7. Edit the address — change the label or a field — and save.
8. Set the address as **Default** using the corresponding button.
9. Delete the address using the delete button and confirm the deletion prompt.

**Expected Result**:
- Account page loads and displays: display name, email address, and phone number (if set)
- Saved Addresses page loads and shows existing addresses (or an empty state message if none exist)
- Adding an address works: new address appears in the list after saving
- Editing an address works: changes are reflected after saving
- Setting default address works: address is marked as "Default" visually
- Deleting an address works: address is removed from the list after deletion

**Pass Criteria**:
- ✅ Account page loads with correct name and email for `customer.uat@wantfood.com`
- ✅ Saved Addresses page loads without errors
- ✅ Add address flow completes and new address appears
- ✅ Edit address flow saves changes correctly
- ✅ Default address is visually distinguished from other addresses
- ✅ Delete address removes the address and a confirmation message or prompt is shown

**Edge Cases**:
- Add an address with **only required fields** filled in → should save successfully
- Add an address with a **very long delivery instruction** (200+ characters) → should either save or show a
  character limit warning, not a crash
- Add more than **5 addresses** → confirm whether there is a limit; if so, a clear message should appear
- Try deleting the **default address** → confirm whether the system prevents this or automatically assigns a new
  default

---

### TC-AC-061: Request Data Export — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: A data export (Subject Access Request / SAR) self-service feature was **not
> found** in the current build of the Customer Front-end at the time this document was written. The account page
> (`/Account`) does not contain a "Download my data" or "Request data export" link or button.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team and legal/compliance team
> whether a self-service data export feature is required for launch under GDPR Article 20 (Right to Data
> Portability). If it is required, the feature should be built and this test case updated before UAT runs.
> If it is out of scope for this release (e.g. handled by the customer support team via a manual process),
> document that decision here and note the agreed support process.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Find the **"Download my data"** or **"Request data export"** option.
3. Click it and follow any confirmation steps.
4. Note the result — either a download begins immediately, or a confirmation message states when the export will
   be emailed.

**Expected Result**:
- User receives a downloadable file or email confirmation within the stated time
- Export contains personal data held by WantFood (name, email, order history, addresses)
- Export does **not** contain other users' data

**Pass Criteria** *(once feature is implemented)*:
- ✅ Feature is accessible from the account page
- ✅ Export is delivered (file or email) within the stated timeframe
- ✅ Export contains the correct user's data only

---

### TC-AC-062: Delete Account — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: A self-service account deletion feature was **not found** in the current build
> of the Customer Front-end at the time this document was written. The account page does not contain a
> "Delete my account" or "Close my account" option.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team and legal/compliance team
> whether customer self-service account deletion is required under GDPR Article 17 (Right to Erasure). If it is
> required, the feature should be built and this test case updated. If deletion is handled via customer support
> (with a manual request-and-fulfilment process), please document the agreed process here and include the
> expected response time (GDPR requires erasure within 30 days).
>
> **Note for testers**: Do **not** attempt to delete the `customer.uat@wantfood.com` test account — this account
> is shared and needed for other tests.

**If and when this feature is built, the test should verify**:

**Given**: A dedicated **disposable** test account has been created for this test (ask your PM)

**Steps**:
1. Sign in as the disposable test account on the Customer Front-end.
2. Navigate to `/Account`.
3. Find the **"Delete my account"** or **"Close my account"** option.
4. Click it. A confirmation step should be presented (e.g. "Are you sure? This cannot be undone.").
5. Confirm the deletion.
6. Attempt to sign in again with the same credentials.

**Expected Result**:
- Deletion confirmation step is shown (users must not be able to accidentally delete their account)
- After confirming, the account is deleted and the user is signed out
- Attempting to sign back in either fails (if the Entra ID account is also removed) or results in an "account
  not found" message in WantFood
- All personal data associated with the account is removed from the platform (or scheduled for removal within
  the GDPR retention period)

**Pass Criteria** *(once feature is implemented)*:
- ✅ Explicit confirmation step is required before deletion
- ✅ User is signed out after successful deletion
- ✅ Re-login is blocked or results in "no account found" message
- ✅ Personal data is confirmed removed (verify with development team or run a data check query)

---

### TC-AC-063: Notification Preferences — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: The **Notification Preferences** menu item exists on the customer account page
> (`/Account`) and shows the label "Push, Email, and SMS", but the link currently points to `#` (i.e. it does
> nothing when clicked). The underlying notification service also returns an empty list in the current build,
> indicating this feature is a placeholder.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team whether notification
> preferences are required for launch. If yes, the page should be built (the route and view are not yet
> implemented), and this test case should be updated. If notification preferences are out of scope for this
> release, the account page should be updated to either remove the menu item or show a "Coming soon" message —
> a dead link (`#`) is poor user experience.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Click **Notification Preferences**.
3. Confirm a preferences page loads (not a `#` anchor or a 404).
4. Toggle off **Email** notifications.
5. Save the preference.
6. Refresh the page and confirm the setting persisted.
7. Toggle off **Push** notifications (if applicable — may require a device with push enabled).
8. Toggle off **SMS** notifications (if applicable).

**Expected Result**:
- Notification Preferences page loads and shows current settings
- Toggling a preference and saving persists the change
- After a page refresh, the changed setting is still shown as off/on

**Pass Criteria** *(once feature is implemented)*:
- ✅ Notification preferences page is accessible from the account page
- ✅ Each preference can be toggled independently
- ✅ Changes persist after page refresh
- ✅ Preferences actually affect notification delivery (co-ordinate with development team to verify)

**Edge Cases**:
- Turn off all notification types → user should still receive legally required communications (e.g. order
  confirmation and receipt), but marketing or promotional notifications should stop

---

## Section I — Summary and Next Steps

### What This Document Covers

| Section | Test Cases | Theme |
|---------|-----------|-------|
| A — Sign-in round-trip | TC-AC-001 to TC-AC-004 | Baseline sign-in/sign-out for all four portals |
| B — Password reset | TC-AC-010 | Entra ID SSPR round-trip |
| C — MFA | TC-AC-011, TC-AC-012 | MFA enrolment and second-factor login |
| D — Invite acceptance | TC-AC-020, TC-AC-021 | Expired links; wrong-account edge cases |
| E — RBAC | TC-AC-030 to TC-AC-039 | Cross-portal access control probes |
| F — Session / token | TC-AC-040 to TC-AC-043 | Expiry, multi-tab, multi-browser |
| G — User management | TC-AC-050 to TC-AC-053 | Consequences of admin changes |
| H — GDPR / account | TC-AC-060 to TC-AC-063 | Customer account page; feature-gap placeholders |

### Features to Be Confirmed by PM Before UAT Sign-off

The following items were found to be missing or incomplete in the current build and require PM sign-off before
this test script can be fully executed:

| Ref | Feature | Action Required |
|-----|---------|----------------|
| TC-AC-010 | Password reset (SSPR) | Confirm Entra ID tenant has SSPR enabled in UAT |
| TC-AC-011/012 | MFA enrolment | Confirm MFA policy configuration and provide dedicated MFA test account |
| TC-AC-061 | Customer data export | Confirm whether self-service data export is in scope for launch; if yes, feature must be built |
| TC-AC-062 | Account self-deletion | Confirm whether self-service account deletion is in scope; if not, document the manual GDPR erasure process |
| TC-AC-063 | Notification preferences | Confirm whether the notification preferences page will be built before launch; remove dead `#` link if not |

### Known Limitations of This Test Script

- **MFA tests (TC-AC-011, TC-AC-012)** require a separate MFA test account to avoid disrupting other testers.
  Do not use the standard `admin.uat@wantfood.com` account for MFA enrolment.
- **Session timeout tests (TC-AC-040, TC-AC-041)** require either a long idle period or a UAT tenant configured
  with a short token lifetime. Co-ordinate with your PM to set the token lifetime to 5–10 minutes for UAT.
- **RBAC URL manipulation tests (TC-AC-036)** require your PM to provide a second vendor ID for testing.

### Next Steps

Once all test cases in this document have been executed:

1. Log any failures or bugs in Monday.com following the process in [01-monday-setup.md](01-monday-setup.md).
2. Raise any "Feature to be Confirmed" items as questions with your PM — do not skip them or mark them as passed.
3. When all bugs from this document are resolved or accepted, move on to the end-to-end tests:

📄 **[Order Saga UAT (cross-portal end-to-end flows)](06-order-saga-uat.md)**

Once all portal and authentication UAT is complete, review the sign-off criteria:

📄 **[Sign-off and Completion Criteria](08-signoff.md)**

---

*Document version: 1.0 — written for WantFood UAT Sprint 1.*  
*Last updated: June 2026.*  
*Queries: contact your Project Manager at `pm@wantfood.com` (example).*
