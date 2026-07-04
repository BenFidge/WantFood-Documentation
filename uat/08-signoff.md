# UAT Sign-off and Completion

## Purpose

This document defines the **criteria for passing UAT** and completing the formal sign-off process. It explains what counts as a blocker, how to handle accepted defects, and what the final state of your Monday.com board should look like before the platform is approved for go-live.

---

## UAT Sign-off Overview

UAT sign-off is the formal approval that the platform is ready for production release. It requires:

1. All **blocker** and **critical** bugs are resolved and verified
2. All test scenarios have a recorded outcome (pass, accepted defect, or waived)
3. A minimum pass rate is achieved in each portal
4. The UAT sign-off form is completed and signed by the project owner

---

## Minimum Pass Rates

The following pass rates must be achieved before sign-off can be granted. A test case is counted as "passed" if its **Pass Criteria** are all met with no active blocker bugs against it.

| Portal / Area | Minimum Pass Rate | Notes |
| --- | --- | --- |
| System Admin | 95% | Maximum 1 accepted minor defect per 20 test cases |
| Vendor Admin | 95% | Maximum 1 accepted minor defect per 20 test cases |
| Driver Portal | 90% | Some edge cases may be waived if not reproducible |
| Customer Front-end | 95% | Customer-facing experience must be near-perfect |
| Order Saga (cross-portal) | 100% of Scenarios 1 and 2 (Happy Paths) | Scenarios 3–15 require 90% pass rate |
| Order Saga additional (S16–S25) | 90% pass rate | S20 (Stripe webhook delay) and S25 (rate limiting) may require co-ordination with the dev team to simulate |
| Cross-Portal Impact | 100% of TC-XP-001–TC-XP-005 (highest blast-radius: vendor activation, vendor edit, application approve/reject, menu publish, dish CRUD) | 90% pass rate for TC-XP-006–TC-XP-030 |
| Permissions and Accounts | 100% of TC-AC-001–TC-AC-004 (sign-in round-trips) and TC-AC-030–TC-AC-039 (full RBAC matrix) | 90% on the rest. Note: TC-AC-061, TC-AC-062, TC-AC-063 are FEATURE TO BE CONFIRMED — PM decision required before sign-off |
| Resilience and Edge Cases | 100% of TC-RE-001–TC-RE-005 (network), TC-RE-020–TC-RE-025 (payment edge cases), and TC-RE-050–TC-RE-058 (accessibility) | 90% on performance budgets (TC-RE-070–TC-RE-072) |
| Automation & Jobs | 80% | Jobs that require special UAT configuration may be waived with evidence |

> **How to calculate pass rate**: (Number of passed test cases ÷ Total test cases run) × 100

---

## Bug Severity Reference

Use these definitions when classifying bugs found during UAT. They determine whether a bug blocks sign-off.

| Severity | Definition | Blocks Go-Live? |
| --- | --- | --- |
| **P1 — Blocker** | Platform cannot be used; critical feature completely broken; data loss or corruption; payment processing failure | **Yes — must be fixed** |
| **P2 — Critical** | Major feature broken but a workaround exists; significant data inaccuracy; customer cannot complete a purchase | **Yes — must be fixed** |
| **P3 — High** | Feature partially working; noticeable UX problem; edge case that affects a significant minority of users | **Recommended fix before go-live** |
| **P4 — Medium** | Minor UI/UX issue; cosmetic problem; non-critical feature behaving unexpectedly; edge case affecting few users | May be accepted and scheduled for post-launch |
| **P5 — Low** | Typo, minor visual inconsistency, non-functional enhancement request | Accepted and scheduled for post-launch |

---

## What Is a "Blocker"?

A blocker is any bug at severity **P1 or P2** that prevents sign-off. Examples:

**Automatic Blockers (P1)**:

- Customers cannot add items to basket
- Customers cannot complete checkout (payment always fails)
- Orders placed but not appearing in Vendor Admin
- Payments being charged but orders not created
- Personal data exposed to the wrong user (security issue)
- Platform is inaccessible (500 errors, blank pages on critical paths)

**Blockers (P2)**:

- Vendor cannot accept or reject orders
- Driver Portal deliveries not updating status across portals
- Promo codes applying incorrect discount amounts
- Stripe Connect not functioning for card payments
- Order confirmation emails not sending
- Customer cannot view their order history

---

## What Is an "Accepted Defect"?

An accepted defect is a known bug that is **not blocking go-live** and has been formally agreed to be fixed after launch. Accepted defects must:

1. Be **P4 or P5** severity only (P3 may be accepted with written justification from the project owner)
2. Be **documented** on the Monday.com board with status "Accepted Defect"
3. Have a **target fix date** agreed between the project owner and development team
4. Not affect core order flow, payment processing, or data integrity

### How to Log an Accepted Defect on Monday.com

1. Open the bug item on your Monday.com board
2. Set the **Status** column to `Accepted Defect`
3. In the **Notes** column, add: "Accepted defect — approved by [Project Owner Name] on [Date]. Target fix: [Sprint/Date]."
4. Tag the item with the label `Post-Launch`
5. Notify the project owner to confirm their acceptance in writing (e.g., via a Monday.com comment)

---

## Waived Test Cases

Some test cases may not be executable in the UAT environment due to configuration limitations (e.g., a job that requires a real billing period to elapse). These can be **waived** if:

1. The feature has been code-reviewed and the logic is sound
2. The test cannot be executed without special environment changes (e.g., simulating a 30-day period)
3. The project owner approves the waiver in writing

To log a waived test case:

1. Set the item status on Monday.com to `Waived`
2. Add a note: "Waived — [reason]. Approved by [Project Owner Name] on [Date]."

---

## Final Monday.com Board State

Before sign-off, your Monday.com UAT board should look like this:

| Status | Meaning | Required State at Sign-off |
| --- | --- | --- |
| `Done` | Test passed | All critical test cases must be `Done` |
| `Failed` | Bug found | **Must be zero** for P1/P2 bugs |
| `In Review` | Fix deployed, retest needed | Must be resolved (retested and `Done` or reclassified) |
| `Accepted Defect` | Known issue, accepted for post-launch | Only P4/P5 allowed |
| `Waived` | Test not executed, approved waiver | Only for config-limited tests |
| `Not Run` | Test not yet executed | **Must be zero** at sign-off |

### Before Granting Sign-off, Verify:

- [ ] Zero `Not Run` items remain
- [ ] Zero `Failed` items remain at P1 or P2 severity
- [ ] All `In Review` items have been re-tested and resolved
- [ ] All `Accepted Defect` items have project owner sign-off
- [ ] All `Waived` items have project owner approval

---

## UAT Sign-off Checklist

Complete this checklist in order. Each item must be confirmed before granting sign-off.

### Section 1: Test Execution Complete

- [ ] All System Admin test cases from [02-system-admin-uat.md](02-system-admin-uat.md) have been run
- [ ] All Vendor Admin test cases from [03-vendor-admin-uat.md](03-vendor-admin-uat.md) have been run
- [ ] All Driver Portal test cases from [04-driver-portal-uat.md](04-driver-portal-uat.md) have been run
- [ ] All Customer Front-end test cases from [05-customer-frontend-uat.md](05-customer-frontend-uat.md) have been run
- [ ] All Order Saga scenarios from [06-order-saga-uat.md](06-order-saga-uat.md) have been run
- [ ] All Automation & Jobs test cases from [07-automation-jobs-uat.md](07-automation-jobs-uat.md) have been run (or formally waived)
- [ ] All Cross-Portal Impact test cases from [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) have been run (TC-XP-001–TC-XP-030)
- [ ] All Permissions and Accounts test cases from [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) have been run (TC-AC-001–TC-AC-063, FEATURE TO BE CONFIRMED items waived or deferred by PM)
- [ ] All Resilience and Edge Cases test cases from [11-resilience-and-edge-cases-uat.md](11-resilience-and-edge-cases-uat.md) have been run (TC-RE-001–TC-RE-072)

### Section 2: Bug Resolution

- [ ] Zero P1 (Blocker) bugs remain open
- [ ] Zero P2 (Critical) bugs remain open
- [ ] All P3 (High) bugs have either been fixed or formally accepted by the project owner
- [ ] All P4/P5 bugs have been logged as "Accepted Defect" with target fix dates
- [ ] All "In Review" items have been re-tested and closed

### Section 3: Pass Rate Confirmed

| Portal | Total Tests Run | Tests Passed | Pass Rate | Meets Minimum? |
| --- | --- | --- | --- | --- |
| System Admin |  |  | % | ☐ Yes ☐ No |
| Vendor Admin |  |  | % | ☐ Yes ☐ No |
| Driver Portal |  |  | % | ☐ Yes ☐ No |
| Customer Front-end |  |  | % | ☐ Yes ☐ No |
| Order Saga |  |  | % | ☐ Yes ☐ No |
| Automation & Jobs |  |  | % | ☐ Yes ☐ No |

> Fill in this table based on your Monday.com board totals before presenting for sign-off.

### Section 4: Cross-Portal Verification

- [ ] A complete end-to-end card payment order has been placed, accepted, delivered, and completed across all portals (Scenario 1)
- [ ] A complete end-to-end cash payment order has been placed, accepted, delivered, and completed across all portals (Scenario 2)
- [ ] A vendor rejection and payment void has been verified (Scenario 4)
- [ ] A customer cancellation has been verified (Scenario 6)
- [ ] Order statuses are consistent across Customer Front-end, Vendor Admin, Driver Portal, and System Admin

#### 4.1 Cross-Portal Impact Tests

These paired tests verify that admin changes propagate correctly to every affected surface. Run with an admin portal and the customer front-end open side-by-side (see [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md)).

- [ ] TC-XP-001 — Vendor activate / deactivate → customer front-end visibility and basket orphaning
- [ ] TC-XP-002 — Edit vendor display details (name, description, logo, banner) → vendor page and search results
- [ ] TC-XP-003 — Approve / reject vendor application → applicant email, first VA login, FE appearance
- [ ] TC-XP-004 — Publish / unpublish menu → vendor page, dish search, deep-linked dish URLs, basket
- [ ] TC-XP-005 — Add / edit / delete dish → vendor page, dish detail, dish search, basket
- [ ] TC-XP-006 — Dish availability toggle / variant availability → real-time hide on FE, basket during mid-checkout
- [ ] TC-XP-007 — Reorder dishes / categories → order on customer vendor page
- [ ] TC-XP-008 — Upload dish image → FE dish image, vendor card, thumbnail processing, alt text
- [ ] TC-XP-009 — Hero slide publish / unpublish / reorder / delete → homepage carousel, mobile vs web, CTA link
- [ ] TC-XP-010 — Support Local slide create / edit / publish / delete → homepage Support Local section
- [ ] TC-XP-011 — Cuisine taxonomy reorder mobile vs web → cuisine ribbon order on mobile and web FE
- [ ] TC-XP-012 — Region create / edit / delete → hero slide regional targeting, location-based discovery
- [ ] TC-XP-013 — Bad words add / edit / delete → review submission rejection on FE
- [ ] TC-XP-014 — Dish keywords add / delete → search results matching synonyms on FE
- [ ] TC-XP-015 — Platform offer activate / pause / edit → basket banner, checkout discount, offer badges
- [ ] TC-XP-016 — Vendor offer create / pause / clone / delete → promo code validation, basket display, stacking
- [ ] TC-XP-017 — Commission tier create / edit / delete → vendor reassignment, invoicing maths, tier visibility
- [ ] TC-XP-018 — Commission config change + manual recalculation → downstream invoice maths
- [ ] TC-XP-019 — Platform Payment Settings — add / remove Stripe & WorldPay → card payment, 3DS, error UX
- [ ] TC-XP-020 — Vendor Stripe Connect connect / disconnect → customer checkout card option for that vendor
- [ ] TC-XP-021 — Enable / disable cash at vendor → cash option at customer checkout for that vendor only
- [ ] TC-XP-022 — Delivery cost update — distance-based tiers → customer delivery quote, basket totals
- [ ] TC-XP-023 — Vendor trading hours / opening times edit → "Closed now" indicator, checkout block, scheduled slots
- [ ] TC-XP-024 — Vendor scheduled-orders settings → schedule picker enabled/disabled, furthest date, slot granularity
- [ ] TC-XP-025 — Vendor user transfer / delete → login behaviour for affected user; VA access removed
- [ ] TC-XP-026 — Admin user add / delete → new admin login works; deleted admin blocked; role scope
- [ ] TC-XP-027 — Reindex Vendors / Rebuild Caches / Import ONS Postcodes / Integrity Check → observable FE change
- [ ] TC-XP-028 — Review moderation — flag / resolve + manual rating recalc → review hidden/restored; vendor ratings
- [ ] TC-XP-029 — Content asset library upload / update / delete → assets in hero slides and Support Local
- [ ] TC-XP-030 — Driver invite / remove → Driver Portal access, order assignment availability, mid-flight handover

#### 4.2 Permissions and Accounts

Run against [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md).

**Sign-in round-trips (Section A)**

- [ ] TC-AC-001 — System Admin sign-in, sign-out, and post-sign-out protection
- [ ] TC-AC-002 — Vendor Admin sign-in, sign-out, and post-sign-out protection
- [ ] TC-AC-003 — Driver Portal sign-in, sign-out, and post-sign-out protection
- [ ] TC-AC-004 — Customer Front-end sign-in, sign-out, and post-sign-out protection

**Password reset (Section B)**

- [ ] TC-AC-010 — Password reset returns user to correct portal

**MFA enrolment and second-factor login (Section C)**

- [ ] TC-AC-011 — First-time MFA enrolment
- [ ] TC-AC-012 — Subsequent login with second factor

**Invite acceptance edge cases (Section D)**

- [ ] TC-AC-020 — Invite link expiry — link has expired or already been used
- [ ] TC-AC-021 — Invite link used by a different Entra ID account

**RBAC matrix and deep-link probes (Section E)**

- [ ] TC-AC-030 — Customer accessing System Admin pages
- [ ] TC-AC-031 — Customer accessing Vendor Admin pages
- [ ] TC-AC-032 — Customer accessing Driver Portal pages
- [ ] TC-AC-033 — Vendor Manager accessing System Admin pages
- [ ] TC-AC-034 — Driver accessing Vendor Admin pages
- [ ] TC-AC-035 — Driver accessing System Admin pages
- [ ] TC-AC-036 — Vendor Manager accessing another vendor's data (same portal, different vendor)
- [ ] TC-AC-037 — Anonymous user accessing protected customer pages
- [ ] TC-AC-038 — Platform Admin cannot see other Vendor Admin users' data
- [ ] TC-AC-039 — Verify 403 / access-denied page is friendly

**Session timeout and token expiry (Section F)**

- [ ] TC-AC-040 — Leave tab open for more than one hour, then attempt an action
- [ ] TC-AC-041 — Deep-link to a protected page after token expiry
- [ ] TC-AC-042 — Sign out in one tab, attempt action in another
- [ ] TC-AC-043 — Sign in on two different browsers simultaneously

**User management consequences (Section G)**

- [ ] TC-AC-050 — Deleted vendor user can no longer sign in
- [ ] TC-AC-051 — Transferred vendor user sees new vendor after next login
- [ ] TC-AC-052 — Deleted admin user can no longer sign in
- [ ] TC-AC-053 — Vendor Admin user whose vendor is disabled sees correct message

**GDPR and data lifecycle (Section H)**

- [ ] TC-AC-060 — View account information page
- [ ] TC-AC-061 — Request data export *(FEATURE TO BE CONFIRMED — PM decision required)*
- [ ] TC-AC-062 — Delete account *(FEATURE TO BE CONFIRMED — PM decision required)*
- [ ] TC-AC-063 — Notification preferences *(FEATURE TO BE CONFIRMED — PM decision required)*

#### 4.3 Resilience and Edge Cases

Run against [11-resilience-and-edge-cases-uat.md](11-resilience-and-edge-cases-uat.md).

**Network resilience (Section 1)**

- [ ] TC-RE-001 — Slow connection during customer checkout
- [ ] TC-RE-002 — Brief offline / Wi-Fi drop during basket to checkout
- [ ] TC-RE-003 — Driver Portal offline during active delivery
- [ ] TC-RE-004 — Vendor Admin offline during order acceptance
- [ ] TC-RE-005 — Browser Back/Forward after a failed network call

**Service failures and error pages (Section 2)**

- [ ] TC-RE-010 — Customer visits a vendor URL that no longer exists (404)
- [ ] TC-RE-011 — Back-end service returns a 500 error
- [ ] TC-RE-012 — Customer session signed out by an admin — graceful re-authentication
- [ ] TC-RE-013 — Payment provider intermittently failing — retry UX

**Stripe and payment edge cases (Section 3)**

- [ ] TC-RE-020 — Webhook delayed more than 30 seconds after card capture
- [ ] TC-RE-021 — Payment Intent stuck in "Processing" state
- [ ] TC-RE-022 — 3D Secure — tester closes the challenge without completing
- [ ] TC-RE-023 — 3D Secure authentication failed
- [ ] TC-RE-024 — Insufficient funds card
- [ ] TC-RE-025 — Network drop during a 3DS challenge

**Concurrent edits (Section 4)**

- [ ] TC-RE-030 — Two Vendor Admin users editing the same dish simultaneously
- [ ] TC-RE-031 — Vendor Admin edits menu while a customer is in checkout against that menu
- [ ] TC-RE-032 — System Admin edits commission tier during invoice generation

**Locale, currency, and time zone (Section 5)**

- [ ] TC-RE-040 — Currency displays as GBP (£) everywhere
- [ ] TC-RE-041 — 24-hour vs 12-hour time format consistency
- [ ] TC-RE-042 — BST/GMT (daylight saving time) crossover
- [ ] TC-RE-043 — Customer browser locale set to non-English (French or Spanish)

**Accessibility — keyboard, screen reader, and contrast (Section 6)**

- [ ] TC-RE-050 — Keyboard-only navigation — System Admin main nav and vendor edit form
- [ ] TC-RE-051 — Keyboard-only navigation — customer checkout (basket to confirmation)
- [ ] TC-RE-052 — Screen reader landmarks — System Admin and customer front-end
- [ ] TC-RE-053 — Focus indicators visible on all interactive elements
- [ ] TC-RE-054 — Image alt text — hero slides, dish images, vendor logos
- [ ] TC-RE-055 — Form labels associated with inputs
- [ ] TC-RE-056 — Colour contrast spot-check — action buttons
- [ ] TC-RE-057 — Error message clarity and screen reader announcement
- [ ] TC-RE-058 — Skip-to-content link presence

**Admin portal mobile responsiveness (Section 7)**

- [ ] TC-RE-060 — System Admin on tablet — iPad portrait (768 × 1024)
- [ ] TC-RE-061 — System Admin on mobile — phone (414 × 896)
- [ ] TC-RE-062 — Vendor Admin on tablet (768 × 1024)
- [ ] TC-RE-063 — Vendor Admin Live Order Kanban and Driver Portal on mobile phone

**Performance budgets (Section 8)**

- [ ] TC-RE-070 — Customer home page — first contentful paint under 2 seconds
- [ ] TC-RE-071 — Vendor Admin Live Order Kanban interactive under 3 seconds
- [ ] TC-RE-072 — Customer search results under 1.5 seconds

### Section 5: Environment Checks

- [ ] All test accounts used have been verified to function correctly
- [ ] No test data has been mixed with any production data (UAT and production environments are separate)
- [ ] Stripe test mode is confirmed (no real money was processed during UAT)
- [ ] Any UAT-specific configuration has been documented for the production deployment team

### Section 6: Formal Sign-off

- [ ] UAT Lead / Project Co-ordinator: all items above confirmed ✅
- [ ] **Project Owner sign-off**: I confirm that UAT has been completed to the required standard and the platform is approved for production release

| Role | Name | Signature | Date |
| --- | --- | --- | --- |
| UAT Lead / Co-ordinator |  |  |  |
| Project Owner |  |  |  |
| Development Lead (optional) |  |  |  |

---

## Post Sign-off Actions

Once sign-off is complete, the following should happen:

1. **Notify the development team**: "UAT sign-off granted — ready for production deployment"
2. **Archive the Monday.com board**: rename it to include the date (e.g., "WantFood UAT — SIGNED OFF [Date]") and set all items to read-only
3. **Export the bug list**: export the Monday.com board to a CSV or PDF for project records
4. **Schedule post-launch review**: agree a date (typically 1–2 weeks after go-live) to review any accepted defects and confirm they are on track to be fixed
5. **Retain test evidence**: keep screenshots and recordings from key test scenarios for at least 3 months

---

## Frequently Asked Questions

**Q: What if a P3 bug is found during a core user journey?**
A: Escalate to the project owner immediately. If the P3 bug consistently breaks a key flow (e.g., customers cannot track their orders), it should be elevated to P2 and treated as a blocker.

**Q: Can we go live with P3 bugs?**
A: Only with written approval from the project owner. Each accepted P3 must have a documented fix plan and timeline.

**Q: What if the UAT environment is not representative of production (e.g., different Stripe account)?**
A: Document the difference. If the difference could affect UAT results (e.g., different Stripe webhook configuration), the relevant test cases should be re-run on production before go-live or the risk should be formally accepted.

**Q: What happens if a new bug is found after sign-off?**
A: If it is a P1 or P2 bug, sign-off is revoked and the bug must be fixed and re-verified. If it is P3 or lower, it may be added to the post-launch defect list.

**Q: Who has the final authority to approve sign-off?**
A: The **Project Owner** (the person responsible for the product, typically your client or stakeholder). The UAT Lead facilitates the process, but the Project Owner makes the final call.

---

---

## Outstanding Feature Decisions

The items below were flagged as **FEATURE TO BE CONFIRMED** during UAT test-script authorship. Each one requires a decision from the **Product Manager** before the relevant test cases can be signed off. Suggested deadline: **at least one week before the planned sign-off date**, so that any scoping-in work can be scheduled.

| # | Feature | What is missing | Who decides | Notes |
| --- | --- | --- | --- | --- |
| 1 | **GDPR data export** | No "Request my data" flow exists in the current build. TC-AC-061 (Permissions doc) and TC-FE-164 (Customer Front-end doc) cannot be executed. | PM | If in scope for launch, dev estimate is needed immediately. If out of scope, mark TC-AC-061 and TC-FE-164 as "Waived — post-launch" before sign-off. |
| 2 | **Self-service account deletion** | No "Delete my account" option exists in the current build. TC-AC-062 and TC-FE-165 cannot be executed. | PM | GDPR Article 17 ("right to erasure") applies. If deferred, a manual process (customer emails support) must be documented and communicated. |
| 3 | **Notification preferences** | No notification preferences page exists. TC-AC-063 and TC-FE-161 cannot be executed. | PM | If email/push notification preferences are in scope, confirm the UI location. If not, waive with a note that defaults will apply at launch. |
| 4 | **Personal Details edit page** | TC-FE-162 references an editable Personal Details page that is not yet confirmed in the build. | PM | Confirm whether customers can edit their display name / phone number in-app, or whether this is deferred to a post-launch iteration. |
| 5 | **Saved Payment Methods page** | TC-FE-163 references a Saved Payment Methods management page. This page was not confirmed as in-scope for launch. | PM | If Stripe SetupIntents / saved cards are in scope, the page must exist before TC-FE-163 can be executed. If not, waive and confirm that no card-saving UI is present. |
| 6 | **Driver document upload** | TC-DP-070 and TC-DP-071 cover driver licence and insurance document upload. Confirm whether document upload is in scope for the MVP or deferred. | PM | If deferred, the Driver Portal onboarding must clearly communicate what documents are required and how they should be submitted (e.g., via email to an ops team). |
| 7 | **Partial refund UI** | The current build supports full refund on cancellation only. No partial refund flow exists for scenarios such as a missing item or a substitution the customer rejected. Confirm whether partial refund is required for launch. | PM | If required, a dev estimate and new test cases are needed. If out of scope, customer-facing refund communication must be clarified (e.g., handled by the ops team via Stripe dashboard). |
| 8 | **Audit log UI** | No audit log UI exists in the current System Admin build. Internal admin actions (e.g., who changed a commission rate, who deactivated a vendor) are logged at the back end only. Confirm whether a UI for viewing audit logs is required for launch. | PM | If required, the scope and access controls for the audit log must be agreed. If deferred, confirm that the engineering team can provide ad-hoc audit log queries on request. |

> **Process**: For each item, the PM should reply in writing (Monday.com comment, email, or Slack with a date stamp) confirming the decision. The UAT Lead should log the decision against the relevant test cases on the Monday.com board before sign-off is granted.

---

## Congratulations! 🎉

You have completed the **WantFood UAT Programme**. Here is a summary of everything you tested:

| Document | Area Covered |
| --- | --- |
| [00-introduction.md](00-introduction.md) | UAT overview, test accounts, severity definitions, bug reporting |
| [01-monday-setup.md](01-monday-setup.md) | Monday.com board setup and bug tracking workflow |
| [02-system-admin-uat.md](02-system-admin-uat.md) | System Admin portal — full feature test scripts |
| [03-vendor-admin-uat.md](03-vendor-admin-uat.md) | Vendor Admin portal — full feature test scripts |
| [04-driver-portal-uat.md](04-driver-portal-uat.md) | Driver Portal — full feature test scripts |
| [05-customer-frontend-uat.md](05-customer-frontend-uat.md) | Customer Front-end — full feature test scripts |
| [06-order-saga-uat.md](06-order-saga-uat.md) | Order Saga — 15 cross-portal end-to-end scenarios |
| [07-automation-jobs-uat.md](07-automation-jobs-uat.md) | Background jobs and automation |
| [08-signoff.md](08-signoff.md) | Sign-off criteria, pass rates, and completion checklist |
| [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) | Cross-Portal Impact — 30 paired tests verifying that admin changes reach the customer front-end |
| [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) | Permissions and Accounts — sign-in round-trips, RBAC matrix, session management, GDPR data lifecycle |
| [11-resilience-and-edge-cases-uat.md](11-resilience-and-edge-cases-uat.md) | Resilience and Edge Cases — network failures, service errors, payment edge cases, accessibility, mobile responsiveness, performance budgets |

**Total test cases documented**: 200+

Thank you for your diligence and thoroughness. Your testing helps ensure a high-quality, reliable platform for every customer, vendor, and driver who uses WantFood. 🚀