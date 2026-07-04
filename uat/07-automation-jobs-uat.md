# Automation and Background Jobs UAT — Test Scripts

## Purpose

This document covers testing for the **background/automation jobs** that run on the WantFood platform. These are scheduled or event-driven processes that run behind the scenes: they affect platform data, vendor commission, order archives, and review ratings without direct user interaction.

**Important**: You cannot directly trigger or view these jobs as an end user. Instead, this document describes:
1. What data to set up **before** a job runs
2. What to observe **after** the job has run
3. How to confirm the job had the expected effect

**Target audience**: Platform administrators and QA testers with access to System Admin

---

## Prerequisites

Before testing automation and jobs, ensure you have:

- ✅ Access to System Admin as `sysadmin.uat@wantfood.com`
- ✅ Completed at least 3–5 test orders (from the Order Saga UAT) including:
  - Completed delivered orders (card payment)
  - Completed delivered orders (cash payment)
  - Cancelled orders
- ✅ At least 3 customer reviews submitted on test vendors (from Customer Front-end UAT)
- ✅ Co-ordinator/developer access to trigger jobs manually (or confirm the job schedule with your project manager)

---

## How to Test Background Jobs

Since background jobs run on a schedule (e.g., daily, weekly), testing them in UAT typically requires one of:

1. **Waiting for the job to run** — the simplest approach if the schedule is short (e.g., every 15 minutes in UAT)
2. **Manually triggering the job** — requires a developer or admin to trigger it via an admin endpoint, command, or Hangfire dashboard
3. **Inspecting the effects** — checking the data has changed in the expected way after the job runs

> **Before starting each test**: confirm with your project manager or developer how to trigger or observe each job in the UAT environment.

---

## Table of Contents

1. [ReconcileDailyStats Job](#reconciledailystats-job)
2. [ArchiveCompletedOrders Job](#archivecompletedorders-job)
3. [Tier Calculation Job](#tier-calculation-job)
4. [Rating Calculation Job](#rating-calculation-job)
5. [Stale Thread Cleanup Job](#stale-thread-cleanup-job)
6. [Cash Order Commission Invoicing](#cash-order-commission-invoicing)

---

## ReconcileDailyStats Job

### TC-AUTO-001: Daily Stats Reconciliation

**Purpose**: This job recalculates vendor statistics at the end of each day, including total orders, total revenue, and platform commission for the day.

**What it affects**:
- Vendor-level daily summary data in System Admin
- Commission calculations for the daily period
- Dashboard metrics in System Admin and Vendor Admin

**Setup (before the job runs)**:
1. Ensure at least 3 completed orders exist for the test vendor today
2. Note the current order counts and revenue totals for the test vendor in System Admin

**Steps**:
1. Ask your developer/co-ordinator to trigger the `ReconcileDailyStats` job (or wait for it to run on schedule)
2. After the job completes, navigate to System Admin → Vendors → [Test Vendor 1]
3. Observe the vendor's daily stats or dashboard metrics

**Expected**:
- The vendor's daily order count reflects the completed orders from today
- The total revenue matches the sum of completed order totals
- Platform commission figures are updated to match the expected commission rate

**Pass Criteria**:
- ✅ Order count matches expected number of completed orders
- ✅ Revenue total matches expected sum
- ✅ Commission figures are calculated correctly

**Edge Cases**:
- Orders created after the reconciliation runs → will be included in the next reconciliation
- Refunded orders → commission should be recalculated to exclude the refunded amount
- Orders with promo codes → revenue should reflect the discounted amount actually charged

**Front-end / cross-portal verification**:
After the job completes, open the SystemAdmin dashboard (see TC-SA-010) and confirm the metric cards have refreshed to include the previous day's orders and revenue.
The daily-revenue chart should contain a data point for yesterday; no bar should be missing or show £0.00 when orders were placed.
No customer-facing change is expected — spot-check the customer vendor page to confirm no artefact or error is visible.

---

### TC-AUTO-002: Daily Stats — Zero Order Day

**Setup**: Do not place any orders on the test vendor for the current day. Then trigger the `ReconcileDailyStats` job.

**Expected**:
- The vendor's daily stats show zero orders and zero revenue for today
- No division-by-zero or null errors occur
- Yesterday's stats are unaffected

**Pass Criteria**:
- ✅ Zero order day handled gracefully (no errors)
- ✅ Previous days' stats unchanged

**Front-end / cross-portal verification**:
Open the SystemAdmin dashboard (TC-SA-010) and confirm no chart is broken, no metric card shows NaN or an error state, and the daily-revenue chart renders a valid £0.00 bar (or omits the day gracefully — document whichever behaviour is observed).
Previous days' bars and totals must be entirely unchanged.

---

## ArchiveCompletedOrders Job

### TC-AUTO-010: Archive Completed Orders

**Purpose**: This job moves older completed and cancelled orders to an archived state, reducing the size of the active orders dataset. Archived orders are still visible in Order History but are not included in live dashboards.

**What it affects**:
- Orders older than the configured archive threshold (confirm threshold with your project manager — e.g., 30 days or 7 days in UAT)
- Archived orders are still visible in System Admin Order History and Customer Order History
- Archived orders are excluded from Vendor Admin live kanban and active reports

**Setup**:
- If the UAT environment supports date simulation, use it to create orders dated older than the archive threshold
- If not, check if any real orders in the UAT environment are old enough to be archived

**Steps**:
1. In System Admin, navigate to Orders and note how many orders exist for a test vendor
2. Ask your developer/co-ordinator to trigger the `ArchiveCompletedOrders` job
3. After the job completes, refresh the Orders view
4. Check the Vendor Admin Live Order Kanban

**Expected**:
- Orders older than the threshold are no longer in the active order views
- Archived orders remain visible in System Admin's order history (with an "Archived" label or status)
- Archived orders remain visible in the Customer's Order History in the Customer Front-end
- Live kanban in Vendor Admin does not show archived orders
- No data is deleted — only the visibility/status is changed

**Pass Criteria**:
- ✅ Old orders are marked as archived
- ✅ Archived orders are still accessible in history views
- ✅ Archived orders do not appear in live/active order views
- ✅ No orders are permanently deleted

**Edge Cases**:
- Orders at exactly the archive threshold age → confirm whether they are archived or not (boundary condition)
- Partially completed orders (e.g., status "Dispatched" for a long time) → should not be archived until completed

**Front-end / cross-portal verification**:
Navigate to the customer Order History (TC-FE-071) and confirm that archived orders are still listed — the system reads from both active and archive stores, so no order should "vanish" from the customer's view after archival.
In SystemAdmin, confirm the order search falls back to the archive and returns the archived orders when searched by ID or reference.
Spot-check the VendorAdmin live kanban to confirm archived orders do not appear there.

---

## Tier Calculation Job

### TC-AUTO-020: Vendor Tier Calculation

**Purpose**: This job recalculates vendor tiers (e.g., Bronze, Silver, Gold, Platinum) based on their order volume, revenue, ratings, or other platform criteria. Tier affects the vendor's visibility, commission rate, or other platform benefits.

**What it affects**:
- Vendor tier in System Admin
- Vendor display tier in Vendor Admin (if shown)
- Any tier-based commission rates or features

**Setup**:
1. In System Admin, note the current tier of the test vendor (e.g., Bronze)
2. Ensure the test vendor has sufficient completed orders to potentially qualify for a higher tier
3. Confirm with your project manager what the tier thresholds are in UAT

**Steps**:
1. Ask your developer/co-ordinator to trigger the `TierCalculationBackgroundService` job
2. After the job completes, navigate to System Admin → Vendors → [Test Vendor 1]
3. Observe the vendor's tier

**Expected**:
- If the vendor's order history meets the criteria for a higher tier: the tier is upgraded
- If the vendor does not meet the criteria: the tier remains unchanged
- The tier update is reflected in System Admin

**Pass Criteria**:
- ✅ Tier is updated based on the correct criteria
- ✅ Tier change (if any) is visible in System Admin
- ✅ Tier does not change incorrectly (i.e., a vendor should not be downgraded without meeting the downgrade criteria)

**Edge Cases**:
- Vendor on the boundary of a tier threshold → test both just above and just below
- Newly created vendor with no orders → should remain at the lowest tier

**Front-end / cross-portal verification**:
After the tier change, place a small test order for the vendor and check the next invoice run (TC-SA-083) to confirm the commission line reflects the new tier's rate — cross-reference TC-XP-017.
Confirm in SystemAdmin that the vendor's tier badge is updated; the new commission rate should appear on subsequent invoices generated by TC-AUTO-050.

---

### TC-AUTO-021: Tier Calculation — Downgrade Scenario

**Setup**: Ensure the test vendor has orders that would normally qualify for a lower tier (or use a test vendor with minimal recent orders if the tier calculation considers recent activity only).

**Steps**:
1. Note the current tier of the test vendor
2. Trigger the `TierCalculationBackgroundService` job
3. Observe the result

**Expected**:
- If the vendor no longer meets the criteria for their current tier: the tier is downgraded
- Vendor Admin and System Admin reflect the new (lower) tier

**Pass Criteria**:
- ✅ Downgrade occurs when criteria are not met
- ✅ Downgrade is reflected in both portals

**Front-end / cross-portal verification**:
As with TC-AUTO-020, confirm the next invoice run (TC-SA-083) uses the downgraded commission rate — cross-reference TC-XP-017.
Confirm there is no customer-visible artefact: commission tier is a back-office concept only and must not appear on the customer vendor page, checkout, or order confirmation.

---

## Rating Calculation Job

### TC-AUTO-030: Vendor Rating Recalculation

**Purpose**: This job recalculates the average star rating for each vendor based on all approved customer reviews. The calculated rating is displayed on the vendor's page on the Customer Front-end.

**What it affects**:
- Vendor's displayed star rating on the Customer Front-end
- Vendor's rating in System Admin and Vendor Admin

**Setup**:
1. Submit 3 customer reviews for the test vendor (from Customer Front-end UAT):
	- 2× five-star reviews
	- 1× two-star review
	- Expected average: (5 + 5 + 2) ÷ 3 = **4.0 stars**
2. Note the current displayed rating of the test vendor on the Customer Front-end

**Steps**:
1. Ask your developer/co-ordinator to trigger the `RatingCalculationBackgroundService` job
2. After the job completes, navigate to the test vendor's page on the Customer Front-end
3. Observe the displayed star rating

**Expected**:
- The vendor's displayed rating is updated to 4.0 (or the expected average based on your test data)
- The rating is displayed correctly in Customer Front-end, Vendor Admin, and System Admin

**Pass Criteria**:
- ✅ Rating is recalculated correctly
- ✅ Rating matches the expected average based on submitted reviews
- ✅ Rating is updated on the Customer Front-end vendor page

**Edge Cases**:
- Vendor with no reviews → should display "No reviews" or a neutral state (not 0 stars)
- Vendor with 1 review → rating should equal that one review's score exactly
- Vendor with a flagged (hidden) review → flagged review should not be included in the average

**Front-end / cross-portal verification**:
After the job completes, open the customer-facing vendor page (TC-FE-020) and confirm the displayed star rating and `TotalReviews` count have updated within a few minutes.
The updated values should also be reflected in SystemAdmin and VendorAdmin vendor detail views — cross-reference TC-XP-028.

---

### TC-AUTO-031: Rating — Flagged Review Excluded

**Setup**:
1. Submit 3 reviews: 5-star, 5-star, and 1-star
2. As Vendor Admin, **flag** the 1-star review for moderation
3. As System Admin, process the flag and **remove** the 1-star review

**Steps**:
1. Trigger the `RatingCalculationBackgroundService` job after the review is removed
2. Check the vendor's rating

**Expected**:
- Rating recalculates without the removed 1-star review
- New rating = (5 + 5) ÷ 2 = 5.0 stars

**Pass Criteria**:
- ✅ Removed review is excluded from rating calculation
- ✅ Rating updates correctly

**Front-end / cross-portal verification**:
On the customer-facing vendor page, confirm the flagged 1-star review is not listed and the displayed `AverageRating` is 5.0 (not 3.67).
The `TotalReviews` count should have decreased by one — cross-reference TC-XP-028 for the moderation flow that precedes this job.

---

## Stale Thread Cleanup Job

### TC-AUTO-040: Stale Thread Cleanup

**Purpose**: This job cleans up stale or incomplete data threads — such as abandoned baskets, expired quote records, incomplete checkout sessions, or orphaned payment intent records that were never completed.

**What it affects**:
- Abandoned quote records (quotes that expired without an order being placed)
- Stale basket/session data (if persisted server-side)
- Orphaned payment intent records (payment intents created but never confirmed)

**Setup**:
1. In a browser, start a checkout as the Customer test account but **do not complete the purchase** (leave the checkout page after generating a quote)
2. Wait for the quote to expire (confirm TTL with your project manager)

**Steps**:
1. Ask your developer/co-ordinator to trigger the `StaleThreadCleanupService` job
2. After the job runs, ask the developer to confirm that expired quote records no longer exist in the database (or check a System Admin diagnostic view if available)

**Expected**:
- Expired quote records are cleaned up
- Orphaned payment intent records are cancelled (if the job handles this)
- No customer-facing disruption occurs
- Active quotes and in-progress orders are NOT affected

**Pass Criteria**:
- ✅ Stale records are cleaned up without errors
- ✅ Active orders and valid sessions are unaffected
- ✅ No errors appear in System Admin after the job runs

**Edge Cases**:
- Cleanup job runs while a customer is actively checking out → active sessions should not be affected (the job should only affect expired/abandoned records)

**Front-end / cross-portal verification**:
Simulate a customer who abandoned checkout at the payment step, wait for the cleanup window to elapse, then trigger the job.
After the job runs, log in as that customer and attempt to place a new order — the customer should be able to start fresh with no ghost basket, stale quote reference, or pre-filled payment state carried over from the abandoned session.
Confirm no error banners appear on the basket or checkout pages.

---

## Cash Order Commission Invoicing

### TC-AUTO-050: Cash Commission Invoice Generation

**Purpose**: The platform must generate periodic invoices to vendors for the commission owed on cash orders (since cash payments bypass Stripe). This job creates and sends invoices for the current billing period.

**What it affects**:
- Commission invoices in System Admin
- Vendor's invoice history in Vendor Admin (if visible)
- Vendor's total outstanding balance

**Setup**:
1. Ensure at least 3 completed cash orders exist for the test vendor (from Order Saga UAT Scenario 2)
2. Note the expected commission amount: (number of cash orders × average order value) × commission rate %
3. Confirm with your project manager: what is the commission rate and billing period in UAT?

**Steps**:
1. Ask your developer/co-ordinator to trigger the commission invoicing job (or wait for the billing period to end)
2. After the job runs, navigate to System Admin → Vendors → [Test Vendor 1] → Invoices (or similar)
3. Observe the generated invoice

**Expected**:
- A new invoice is created for the billing period
- Invoice shows: vendor name, billing period, list of cash orders included, subtotal, commission rate, commission amount due
- Invoice status is "Pending" or "Awaiting Payment"
- The vendor can view the invoice in Vendor Admin

**Pass Criteria**:
- ✅ Invoice is generated
- ✅ Invoice amount is correct (matches expected calculation)
- ✅ Invoice is visible in System Admin
- ✅ Invoice is visible in Vendor Admin (if applicable)

**Edge Cases**:
- Vendor with zero cash orders for the billing period → no invoice should be generated (or a £0.00 invoice, depending on business rules)
- Cancelled cash orders → should be excluded from commission (or adjusted)
- Partially completed cash orders → confirm business rules on how these are treated

**Front-end / cross-portal verification**:
After the job runs, navigate to SystemAdmin → Invoices (TC-SA-084) and confirm the new invoice appears in the list with correct vendor, billing period, and amount.
If VendorAdmin surfaces invoice history, log in as the vendor and confirm the invoice is visible there too — cross-reference TC-XP-017 for tier-based commission rate validation.

---

### TC-AUTO-051: Mark Invoice as Paid

**Given**: A cash commission invoice exists in System Admin with status "Pending"

**Steps**:
1. In System Admin, navigate to the invoice
2. Click **"Mark as Paid"**
3. Enter payment reference (if required)
4. Confirm

**Expected**:
- Invoice status changes to "Paid"
- Payment date is recorded
- Vendor Admin (if it shows invoices) reflects the "Paid" status

**Pass Criteria**:
- ✅ Invoice marked as paid
- ✅ Status updates correctly

**Edge Cases**:
- Marking the same invoice as paid twice → should be blocked or idempotent

**Front-end / cross-portal verification**:
Open the SystemAdmin invoice detail (TC-SA-085) and confirm status shows "Paid" with a payment date recorded.
If VendorAdmin surfaces invoice history, log in as the vendor and confirm the invoice also reflects "Paid" status there.
Confirm the "Mark as Paid" button or action is no longer available (or is disabled) on an already-paid invoice.

---

### TC-AUTO-052: Overdue Invoice Handling

**Setup**: If possible in the UAT environment, simulate an invoice that has passed its due date without being marked as paid.

**Steps**:
1. Trigger the overdue invoice check (or wait for the scheduled job to run)
2. Observe the invoice status in System Admin

**Expected**:
- Invoice status changes to "Overdue"
- System Admin shows a warning or alert for the overdue vendor
- An automated reminder email is sent to the vendor (if applicable)

**Pass Criteria**:
- ✅ Overdue invoice status is updated
- ✅ Overdue vendor is flagged in System Admin

**Front-end / cross-portal verification**:
In SystemAdmin, confirm the invoice list (TC-SA-084) shows the "Overdue" status and a visual warning against the vendor.
Check whether the vendor receives an automated reminder email — if so, record the email received and note the subject and sender; if no email is configured, document this as the observed behaviour.
Confirm there is no customer-facing impact: the overdue status is a back-office financial matter only and must not affect the vendor's visibility or checkout availability for customers.

---

## Summary and Next Steps

You have now tested all automation and background job functionality.

### What to do next:

1. **Log all bugs** on your Monday.com board under the "Automation & Jobs" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the final step — Sign-off**:
	- **[Sign-off and Completion](08-signoff.md)** ← **Final step**

---

**Great work! You're almost done! 🚀**
