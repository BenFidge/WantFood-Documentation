# WantFood UAT — Introduction and Testing Guide

## What is User Acceptance Testing (UAT)?

User Acceptance Testing (UAT) is the final validation phase before the WantFood platform goes live. Your role as a tester is to:

- **Verify** that every feature works as expected for real-world users
- **Discover** any issues, bugs, or unexpected behaviors
- **Validate** that the system meets business requirements
- **Report** any problems clearly so the development team can fix them

UAT is **not about finding every possible bug**—it's about ensuring the platform is **fit for purpose** and ready for launch.

---

## The WantFood Platform Overview

WantFood consists of four interconnected portals:

| Portal | Audience | Purpose |
|--------|----------|---------|
| **System Admin** | Platform administrators, operations staff | Manage vendors, applications, orders, commissions, invoices, content, and platform settings |
| **Vendor Admin** | Restaurant owners and managers | Manage menus, accept orders, assign drivers, handle promotions, and configure payment/delivery settings |
| **Driver Portal** | Delivery drivers | Accept deliveries, manage shifts, update delivery status, and optimize routes |
| **Customer Front-end** | Food ordering customers | Browse restaurants, place orders, track deliveries, leave reviews, and manage addresses |

Each portal will be tested separately, plus end-to-end **Order Saga** testing that spans all four systems.

---

## Test Environment Details

### Environment URLs

You will receive the following test environment URLs via email or Slack:

- **System Admin**: `https://system.wantfoodstaging.com` (example)
- **Vendor Admin**: `https://vendor.wantfoodstaging.com` (example)
- **Driver Portal**: `https://driver.wantfoodstaging.com` (example)
- **Customer Front-end**: `https://www.wantfoodstaging.com` (example)

> **Note**: These are example URLs. You will receive the actual UAT environment URLs before testing begins.

### Test Accounts by Role

You will be provided with pre-configured test accounts for each portal:

| Role | Username / Email | Password | What you can test |
|------|------------------|----------|-------------------|
| **System Admin** | `wantfooddevelopment+admin@gmail.com` | W@ntF00d2026 | Full system administration capabilities |
| **Vendor Manager** | `wantfooddevelopment+vendor@gmail.com` | W@ntF00d2026 | Vendor Admin portal for a test restaurant |
| **Driver** | `wantfooddevelopment+driver@gmail.com` | W@ntF00d2026 | Driver Portal for a test driver account |
| **Customer** | `wantfooddevelopment+customer@gmail.com` | W@ntF00d2026 | Customer Front-end for placing test orders |

> **Important**: Passwords will be shared securely via your project manager. Do not share credentials outside the UAT team.

### Test Payment Details

For payment testing, you will use **Stripe test cards**:

| Scenario | Card Number | Expiry | CVV | Expected Result |
|----------|-------------|--------|-----|-----------------|
| Successful payment | `4242 4242 4242 4242` | Any future date | Any 3 digits | Payment succeeds |
| Card declined | `4000 0000 0000 0002` | Any future date | Any 3 digits | Payment fails |
| Authentication required | `4000 0025 0000 3155` | Any future date | Any 3 digits | Triggers 3D Secure flow |

> **Note**: These are Stripe test mode cards. They will not charge real money.

---

## Supported Browsers and Devices

### Desktop Browsers

Test on the following browsers (latest stable versions):

- **Google Chrome** (Windows, macOS, Linux)
- **Microsoft Edge** (Windows, macOS)
- **Mozilla Firefox** (Windows, macOS, Linux)
- **Safari** (macOS only)

### Mobile Devices

Test on the following devices/browsers:

- **iOS**: Safari on iPhone (iOS 16+)
- **Android**: Chrome on Android (Android 12+)

### Screen Resolutions

Ensure pages render correctly at:

- **Desktop**: 1920×1080, 1366×768
- **Tablet**: 768×1024 (iPad)
- **Mobile**: 375×667 (iPhone SE), 414×896 (iPhone 11 Pro)

---

## How to Evaluate a Test Case

### Pass Criteria

A test case **passes** if:

1. All steps can be completed without errors
2. The actual result matches the expected result
3. No data is lost or corrupted
4. Performance is acceptable (pages load within 3 seconds)
5. Error messages (if expected) are clear and helpful

### Fail Criteria

A test case **fails** if:

1. Any step cannot be completed
2. The actual result differs from the expected result
3. An error occurs that prevents the workflow from continuing
4. Data is lost, corrupted, or displayed incorrectly
5. A critical security or privacy issue is observed

### Partial Pass (Log as a Bug)

Even if a test case **technically passes**, log a bug if you observe:

- Confusing UI/UX (unclear labels, poor layout)
- Spelling or grammar errors
- Slow performance (but not blocking)
- Missing helpful features or information

---

## Severity Definitions

When logging a bug, assign one of the following severity levels:

| Severity | Definition | Example |
|----------|------------|---------|
| **Critical** | System is unusable, no workaround exists, blocks testing | Login page crashes on load |
| **High** | Major feature broken, workaround exists but difficult | Cannot place an order (but can retry) |
| **Medium** | Feature partially broken or incorrect, but usable | Filter returns incorrect results |
| **Low** | Minor cosmetic issue, typo, or UI inconsistency | Button label misspelled |

### How to Choose Severity

Ask yourself:

- **Critical**: "Can I continue testing **at all**?" → If no, it's Critical.
- **High**: "Can I complete the task **without this working**?" → If no, it's High.
- **Medium**: "Is this annoying or incorrect but I can still complete the task?" → If yes, it's Medium.
- **Low**: "Is this just a cosmetic issue or typo?" → If yes, it's Low.

When in doubt, **choose the higher severity**—the development team can always downgrade it.

---

## Pass / Fail / Bug-Anyway — Recording Each Test Outcome

For every test case (TC) in every test script, record **one** of the three outcomes below. Do not leave an outcome blank — an unrecorded TC is as bad as a skipped one.

| Outcome | Symbol | When to use it | What to do next |
|---------|--------|----------------|-----------------|
| **Pass** | ✅ | Every expected result happened exactly as described | Move on to the next TC |
| **Fail** | ❌ | The test was blocked, or the actual result did not match the expected result | Log a bug immediately, then move on |
| **Bug-anyway** | 🐛 | The TC technically passed, but you spotted something off along the way | Log a Low or Medium bug and keep going |

### What counts as a "Bug-anyway"?

Anything that would frustrate or confuse a real user even though the core flow completed. Examples:

- Confusing or ambiguous wording on a label, button, or error message
- A toast notification that disappears too quickly to read
- A change taking noticeably longer to propagate than the 30-second expectation
- A missing screen-reader announcement (e.g. a confirmation that a screen-reader user would never hear)
- An image that loads but is blurry or incorrectly cropped
- A success message that does not clearly state what was saved

> **"Bug-anyway" is the most under-used and most valuable category in UAT.** Testers naturally focus on broken flows and skip the friction that real users encounter every day. A well-logged "Bug-anyway" finding often has more impact on launch quality than a duplicate report of an already-known failure. If you are in doubt whether something is worth logging — log it.

---

## How to Report a Bug

For detailed instructions on how to report bugs using Monday.com, see:

📄 **[Monday.com Setup and Bug Reporting Guide](01-monday-setup.md)**

### Quick Summary

When you find a bug:

1. **Stop and take a screenshot** (or screen recording if possible)
2. **Open the Monday.com UAT board**
3. **Create a new item** in the appropriate group (System Admin, Vendor Admin, etc.)
4. **Fill in the bug template**:
	- Title (short, clear)
	- Steps to reproduce (numbered list)
	- Expected result (what should happen)
	- Actual result (what actually happened)
	- Severity (Critical / High / Medium / Low)
	- Screenshot or recording (attach)
5. **Assign** to your project manager
6. **Continue testing** (don't wait for a fix—log it and move on)

---

## Test Execution Workflow

### 1. Prepare Your Environment

- Confirm you have access to all UAT URLs
- Confirm you can log in with all test accounts
- Set up your Monday.com board (see [01-monday-setup.md](01-monday-setup.md))
- Clear your browser cache and cookies before starting each test session

### 2. Work Through Each Portal

Follow the test scripts in order:

1. **[System Admin UAT](02-system-admin-uat.md)**
2. **[Vendor Admin UAT](03-vendor-admin-uat.md)**
3. **[Driver Portal UAT](04-driver-portal-uat.md)**
4. **[Customer Front-end UAT](05-customer-frontend-uat.md)**
5. **[Order Saga UAT](06-order-saga-uat.md)** (end-to-end flows)
6. **[Automation and Jobs UAT](07-automation-jobs-uat.md)** (background processes)
7. **[Cross-Portal Impact UAT](09-cross-portal-impact-uat.md)** (admin changes rippling to other portals)
8. **[Permissions and Accounts UAT](10-permissions-and-account-uat.md)** (RBAC, role boundaries, account management)
9. **[Resilience and Edge Cases UAT](11-resilience-and-edge-cases-uat.md)** (error handling, connectivity loss, boundary inputs)

### 3. Log Bugs as You Go

- Don't batch your bugs at the end—log them immediately when found
- Don't spend more than 5 minutes trying to "fix" a bug yourself—log it and move on
- If a bug blocks you from continuing, mark it as **Critical** and notify your project manager

### 4. Retest Fixed Bugs

When a bug is marked as **"Fixed"** by the development team:

1. Re-run the exact steps you used to find it
2. Verify it is now working correctly
3. Update the Monday.com item status to **"Verified"** or **"Reopened"** (if still broken)

### 5. Sign Off

Once all test scripts are complete and all bugs are resolved or accepted:

- Review the **[Sign-off and Completion Criteria](08-signoff.md)**
- Confirm with your project manager that you are ready to sign off
- Your project manager will coordinate final approval with stakeholders

---

## Tips for Successful UAT

### Do:

✅ **Test like a real user**—don't just click through quickly  
✅ **Try edge cases**—leave fields blank, enter invalid data, test boundaries  
✅ **Test on multiple browsers and devices**  
✅ **Read error messages carefully** and include them in your bug reports  
✅ **Ask questions** if a test case is unclear  
✅ **Take your time**—quality over speed

### Don't:

❌ **Don't assume something works because it "looks right"**  
❌ **Don't batch test results**—log bugs immediately  
❌ **Don't test in production**—only use the UAT environment  
❌ **Don't share test account credentials** outside the UAT team  
❌ **Don't skip edge cases**—they often reveal critical bugs

---

## Propagation Cheat-Sheet — Why Admin Changes Don't Always Show Up Immediately

Most admin changes appear on the customer front-end within **approximately 30 seconds**. The platform keeps a short-lived cache to serve pages quickly, so there is always a small delay between pressing Save in an admin portal and a customer seeing the result. This is normal and expected — it is not a bug on its own.

**Work through these steps in order before logging a propagation bug:**

1. **Wait 30 seconds**, then hard-refresh the customer front-end page. On Windows press **Ctrl + F5**; on Mac press **Cmd + Shift + R**. A normal refresh (F5) may serve the browser's own cached copy and hide the change.
2. If the change is still not visible, log in to System Admin and go to **Platform Tools → Rebuild Caches** → wait 30 seconds → hard-refresh the customer front-end.
3. For changes that affect **search results or vendor discovery** (vendor name, cuisine, active status, dish keywords), also run **Platform Tools → Reindex Vendors** → wait 30 seconds → repeat your search.
4. If the change is still not visible after completing steps 1–3, **log a bug** with severity **Medium**. Describe the propagation steps you tried and attach screenshots of both the admin "saved" confirmation and the customer front-end showing the old value.

> **What these tools do (plain English):** Rebuild Caches clears the quick-lookup data the website holds in memory to serve pages fast. Reindex Vendors tells the search engine to re-read every vendor's details from scratch. Both are safe to run in the UAT environment at any time.

For the full propagation deep-dive, including email delivery timeouts and audit-log caveats, see **[09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md)**.

---

## Glossary

Plain-language definitions for terms used throughout the UAT test scripts.

| Term | Definition |
|------|------------|
| **Portal** | One of the four web applications that make up WantFood: System Admin, Vendor Admin, Driver Portal, and Customer Front-end. Each has its own URL and login. |
| **Branch** | A physical location of a vendor (e.g. a restaurant chain with multiple sites). Each branch can have its own menu, opening hours, and delivery zone. |
| **Vendor** | A restaurant or food business that sells through WantFood. Vendors are managed in the Vendor Admin portal and appear on the Customer Front-end. |
| **Cuisine** | A food category (e.g. Italian, Japanese, Vegan) used to group vendors. Cuisines appear as filter options on the Customer Front-end. |
| **Hero slide** | A large banner image displayed at the top of the Customer Front-end home page. Slides are published and reordered in System Admin and may target specific regions. |
| **Support Local slide** | A curated card on the home page that highlights an independent local vendor. Managed separately from hero slides in System Admin. |
| **Region** | A geographic area used to target hero slides and control vendor discovery. Regions are created and managed in System Admin. |
| **Basket** | The customer's in-progress list of items before they reach checkout. Items stay in the basket until the customer checks out or clears it. |
| **Quote** | A delivery cost and time estimate generated when the customer enters their address at checkout. The quote may expire if the customer takes too long. |
| **Checkout** | The multi-step process in which the customer confirms their address, reviews their order, applies a promo code, and pays. |
| **Payment intent** | A Stripe record that tracks the lifecycle of a payment attempt. Created when the customer reaches the payment step; completed when payment succeeds. |
| **3D Secure (3DS)** | An additional bank authentication step (e.g. a one-time code sent by SMS) that some cards require before authorising a payment. Triggered by the test card `4000 0025 0000 3155`. |
| **Order saga** | The end-to-end sequence of events from a customer placing an order through to delivery: payment → vendor notification → preparation → driver dispatch → handover → completion. All four portals are involved. |
| **Kanban** | The board view in Vendor Admin that shows incoming and in-progress orders as cards in columns (e.g. New → Preparing → Ready). |
| **Dispatch** | The act of assigning a driver to collect a prepared order. Can be automatic or manual depending on vendor settings. |
| **Active delivery** | An order that has been accepted by a driver and is currently in transit between the vendor and the customer. |
| **Commission** | A percentage fee that WantFood charges the vendor on each order. The rate is set per vendor or via a commission tier. |
| **Commission tier** | A named ruleset that defines a commission rate and any conditions attached to it (e.g. "Standard 12%", "Launch promo 8%"). Applied to vendors in System Admin. |
| **Invoice** | A monthly statement generated for a vendor showing orders, commission deducted, and net payout. Managed in System Admin and visible to vendors in Vendor Admin. |
| **Refund** | A full or partial reversal of a completed payment. Initiated in System Admin or Vendor Admin; processed via Stripe back to the customer's original payment method. |
| **Cache** | A temporary store of pre-computed data that helps the website respond quickly. Caches can become stale after an admin change — see the Propagation Cheat-Sheet above. |
| **Propagation** | The process by which an admin change travels from the admin portal through the back-end systems and eventually appears on the customer-facing front-end. |
| **Reindex** | Instructing the search engine to re-read vendor and dish data so that search results reflect the latest changes. Run via Platform Tools → Reindex Vendors in System Admin. |
| **Rebuild caches** | Clearing and regenerating all short-lived cached data so that the front-end serves the latest content. Run via Platform Tools → Rebuild Caches in System Admin. |
| **RBAC (role-based access control)** | The system that determines what each user is allowed to see and do based on their assigned role (e.g. System Admin, Vendor Manager, Driver). A user should never be able to access or change data outside their role. |
| **Entra ID** | Microsoft's cloud identity service (formerly Azure Active Directory). WantFood uses Entra ID for all user logins. If a login fails, it is usually an Entra ID account issue rather than a WantFood bug. |
| **Token expiry** | Authentication tokens issued by Entra ID have a limited lifespan. When a token expires the user is asked to sign in again. Long test sessions may encounter token expiry — simply log in again and continue. |
| **Cross-portal test** | A test case that requires actions in more than one portal to be completed (e.g. change a vendor's name in Vendor Admin, then verify the change on the Customer Front-end). |
| **Sunny path** | The expected, happy-path flow where everything works as intended and the user has valid data. Most test cases start with the sunny path before testing edge cases. |
| **Edge case** | An unusual or boundary input that real users will occasionally provide — blank fields, very long text, special characters, expired promo codes, cancelled orders mid-flow, and so on. Edge cases often reveal bugs that sunny-path testing misses. |

---

## Questions or Issues?

If you have questions about the UAT process, need access to the environment, or encounter a blocking issue:

**Contact your Project Manager:**

- Email: `pm@wantfood.com` (example)
- Slack: `#uat-wantfood` (example)
- Phone: `+44 20 1234 5678` (example)

> **Note**: Contact details will be provided by your project manager before UAT begins.

---

## Next Steps

📄 **[Set up your Monday.com board](01-monday-setup.md)** to start logging bugs and tracking progress.

Once your board is ready, begin testing:

1. **[System Admin UAT](02-system-admin-uat.md)**
2. **[Vendor Admin UAT](03-vendor-admin-uat.md)**
3. **[Driver Portal UAT](04-driver-portal-uat.md)**
4. **[Customer Front-end UAT](05-customer-frontend-uat.md)**
5. **[Order Saga UAT](06-order-saga-uat.md)**
6. **[Automation and Jobs UAT](07-automation-jobs-uat.md)**
7. **[Cross-Portal Impact UAT](09-cross-portal-impact-uat.md)**
8. **[Permissions and Accounts UAT](10-permissions-and-account-uat.md)**
9. **[Resilience and Edge Cases UAT](11-resilience-and-edge-cases-uat.md)**
10. **[Sign-off and Completion](08-signoff.md)**

---

**Happy Testing! 🚀**
