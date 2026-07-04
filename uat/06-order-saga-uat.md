# Order Saga UAT — Test Scripts

## Purpose

This document tests the **complete end-to-end order lifecycle** — the "Order Saga" — across all four portals simultaneously. Each test scenario walks through a real order from basket to delivery, verifying that every system involved handles its role correctly and that the correct state is visible in each portal at each step.

**This is the most important section of UAT.** A bug that only appears when multiple portals interact will only be found here.

**Target audience**: UAT co-ordinators and testers who are comfortable operating all four portals at once

---

## How These Tests Work

Each test scenario in this document is a **cross-portal workflow**. You will need to have all four portals open simultaneously (or co-ordinate with teammates assigned to each portal):

| Role | Portal | Test Account |
|------|--------|--------------|
| **Customer** | Customer Front-end | `customer.uat@wantfood.com` |
| **Vendor** | Vendor Admin | `vendor.uat@wantfood.com` |
| **Driver** | Driver Portal | `driver.uat@wantfood.com` |
| **Platform Admin** | System Admin | `sysadmin.uat@wantfood.com` |

### Test Co-ordination Tips

- Use four separate browser windows (or profiles) to keep all sessions open at once
- Note the **Order ID** as soon as it is created — use it to cross-reference across portals
- Agree on a communication method (e.g., team chat) so you can co-ordinate timing
- Take screenshots at each key status transition for your bug evidence

---

## Prerequisites

Before starting, verify:

- ✅ All four test accounts are working (can log in)
- ✅ Test vendor has at least one published menu with priced dishes
- ✅ Test driver has completed onboarding and is linked to the test vendor
- ✅ Stripe is connected to the test vendor (for card payment scenarios)
- ✅ Test vendor accepts both card and cash payments
- ✅ Test driver has started their shift before tests that require driver assignment
- ✅ All portals are open and sessions are active

---

## Order Status Reference

At each stage of a test, you should verify the order status is correct in each portal. Use this reference:

| Status | Customer Front-end | Vendor Admin Kanban | Driver Portal | System Admin |
|--------|--------------------|---------------------|---------------|--------------|
| Order placed (card) | "Awaiting Confirmation" | "Pending" column | — | Order visible |
| Order placed (cash) | "Awaiting Confirmation" | "Pending" column | — | Order visible |
| Accepted by vendor | "Being Prepared" | "Accepted" column | — | Status updated |
| Rejected by vendor | "Cancelled" | "Cancelled" column | — | Status updated |
| Ready for pickup | "Out for Delivery (soon)" | "Ready" column | Assigned delivery visible | Status updated |
| Driver assigned | "Driver on the way" | Order shows driver name | Active delivery in list | Status updated |
| Dispatched/Picked up | "On the way" | "Dispatched" column | Status options available | Status updated |
| Delivered | "Delivered" | "Completed" column | Delivery in history | Status updated |
| Cancelled | "Cancelled" | "Cancelled" column | — | Status updated |

> **Note**: Exact wording of statuses on the front-end may differ — what matters is that the customer-facing message matches the underlying state. Record any mismatch as a bug.

---

## Table of Contents

1. [Scenario 1: Happy Path — Card Payment, Delivered Successfully](#scenario-1-happy-path--card-payment-delivered-successfully)
2. [Scenario 2: Happy Path — Cash Payment, Delivered Successfully](#scenario-2-happy-path--cash-payment-delivered-successfully)
3. [Scenario 3: Card Payment Declined at Checkout](#scenario-3-card-payment-declined-at-checkout)
4. [Scenario 4: Vendor Rejects Order](#scenario-4-vendor-rejects-order)
5. [Scenario 5: Vendor Does Not Respond (Acceptance Timeout)](#scenario-5-vendor-does-not-respond-acceptance-timeout)
6. [Scenario 6: Customer Cancels Before Vendor Accepts](#scenario-6-customer-cancels-before-vendor-accepts)
7. [Scenario 7: Customer Cancels After Vendor Accepts](#scenario-7-customer-cancels-after-vendor-accepts)
8. [Scenario 8: Driver Cannot Be Found (No Available Drivers)](#scenario-8-driver-cannot-be-found-no-available-drivers)
9. [Scenario 9: Driver Assigned, Then Unassigned and Reassigned](#scenario-9-driver-assigned-then-unassigned-and-reassigned)
10. [Scenario 10: Delivery Marked as Failed](#scenario-10-delivery-marked-as-failed)
11. [Scenario 11: Tip Added After Delivery (Card Payment)](#scenario-11-tip-added-after-delivery-card-payment)
12. [Scenario 12: Full Refund After Vendor Cancellation](#scenario-12-full-refund-after-vendor-cancellation)
13. [Scenario 13: Promo Code Applied to Order](#scenario-13-promo-code-applied-to-order)
14. [Scenario 14: Quote Expires Before Payment](#scenario-14-quote-expires-before-payment)
15. [Scenario 15: Duplicate Checkout (Double-Submit Prevention)](#scenario-15-duplicate-checkout-double-submit-prevention)
16. [Scenario 16: Vendor Deactivated Mid-Flow](#scenario-16-vendor-deactivated-mid-flow)
17. [Scenario 17: Menu Republished Mid-Checkout](#scenario-17-menu-republished-mid-checkout)
18. [Scenario 18: Vendor Edits an In-Progress Order](#scenario-18-vendor-edits-an-in-progress-order)
19. [Scenario 19: Full Refund via Vendor Cancellation (Card Order, Post-Capture)](#scenario-19-full-refund-via-vendor-cancellation-card-order-post-capture)
20. [Scenario 20: Stripe Webhook Delayed](#scenario-20-stripe-webhook-delayed)
21. [Scenario 21: Order Placed Outside Vendor Trading Hours](#scenario-21-order-placed-outside-vendor-trading-hours)
22. [Scenario 22: Stacked Offers — Platform Promotion and Vendor Promo Code](#scenario-22-stacked-offers--platform-promotion-and-vendor-promo-code)
23. [Scenario 23: Driver Shift Ends Mid-Delivery](#scenario-23-driver-shift-ends-mid-delivery)
24. [Scenario 24: Customer Changes Default Address Mid-Checkout](#scenario-24-customer-changes-default-address-mid-checkout)
25. [Scenario 25: Rate Limiting and Duplicate Payment Submission](#scenario-25-rate-limiting-and-duplicate-payment-submission)

---

## Scenario 1: Happy Path — Card Payment, Delivered Successfully

**Summary**: A customer places an order and pays by card. The vendor accepts, the driver collects and delivers the order. All portals reflect the correct status at every step.

**Portals involved**: Customer Front-end, Vendor Admin, Driver Portal, System Admin

---

### Step 1.1 — Customer Places Order

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Navigate to "Test Restaurant 1" (the card-accepting vendor)
3. Add 2–3 dishes to the basket
4. Proceed to checkout
5. Enter a valid in-zone delivery address
6. Confirm the delivery quote (note the subtotal, delivery fee, and total)
7. Select **"Pay by Card"**
8. Enter Stripe test card: `4242 4242 4242 4242`, any future expiry, any CVV
9. Click **"Place Order"**

**Expected**:
- Order Confirmation page appears with an Order ID
- Order total matches the quote shown before payment
- Customer receives a confirmation email

**Verify in Vendor Admin**:
- Switch to the Live Order Kanban
- The new order should appear in the **"Pending"** column within 60 seconds
- Order card should show correct items and total

**Verify in System Admin**:
- The order should be visible in the Orders list with status "Pending"

**Pass Criteria**:
- ✅ Order Confirmation page shows correct details
- ✅ Order appears in Vendor Admin "Pending" column
- ✅ Order visible in System Admin

**Edge Cases**:
- Confirmation page takes too long to load → report as bug (performance issue)
- Order does not appear in Vendor Admin within 60 seconds → report as bug (real-time delivery failure)

---

### Step 1.2 — Vendor Accepts Order

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the order from the "Pending" column
2. Review the order details (items, delivery address, customer note)
3. Click **"Accept"**
4. Set an estimated preparation time (if prompted)

**Expected**:
- Success message: "Order accepted"
- Order moves to **"Accepted"** column in Vendor Admin
- Customer Front-end updates to show "Being Prepared" (or equivalent)

**Verify in Customer Front-end**:
- Order status should update without manual refresh

**Pass Criteria**:
- ✅ Order moves to "Accepted" column
- ✅ Customer-facing status updates

---

### Step 1.3 — Vendor Marks Order as Ready

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the accepted order
2. Click **"Mark as Ready"** (food is prepared and ready for collection)

**Expected**:
- Success message: "Order marked as ready"
- Order moves to **"Ready"** column in Vendor Admin

**Pass Criteria**:
- ✅ Order moves to "Ready" column

---

### Step 1.4 — Vendor Assigns Driver

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the "Ready" order
2. Click **"Assign Driver"**
3. Select the test driver (`driver.uat@wantfood.com`) from the available drivers list
4. Confirm the assignment

**Expected**:
- Success message: "Driver assigned"
- Driver's name appears on the order card
- Order remains in "Ready" column

**Verify in Driver Portal**:
- The assigned delivery should appear in the test driver's **Active Deliveries** list

**Pass Criteria**:
- ✅ Driver is assigned
- ✅ Delivery appears in Driver Portal

---

### Step 1.5 — Driver Marks Delivery as Dispatched / Picks Up Order

**Actor**: Driver

**Steps**:
1. In the Driver Portal, open the active delivery
2. Review the pickup address (restaurant) and delivery address (customer)
3. If available, click a "Picked Up" or "On My Way" status update

**Expected**:
- Delivery status updates in the Driver Portal
- Vendor Admin order card updates (e.g., moves to "Dispatched" column)
- Customer Front-end updates to show "Driver on the way" (or equivalent)

**Pass Criteria**:
- ✅ Status updates across all portals

---

### Step 1.6 — Driver Marks Order as Delivered

**Actor**: Driver

**Steps**:
1. In the Driver Portal, open the active delivery
2. Click **"Mark as Delivered"**
3. Confirm the action

**Expected**:
- Success message: "Delivery marked as completed"
- Delivery moves to Driver Portal delivery history
- Vendor Admin order moves to **"Completed"** column
- Customer Front-end updates to show "Delivered" (or equivalent)
- Customer receives a "Your order has been delivered" notification
- System Admin order status shows "Delivered/Completed"
- Customer is prompted to leave a review and/or add a tip

**Pass Criteria**:
- ✅ Order status is "Completed/Delivered" in all portals
- ✅ Customer notification sent
- ✅ Review/tip prompt appears on Customer Front-end
- ✅ Driver delivery history shows the completed delivery

---

## Scenario 2: Happy Path — Cash Payment, Delivered Successfully

**Summary**: Same as Scenario 1, but the customer selects "Pay with Cash". No Stripe payment is taken. The platform records the order for commission invoicing.

**Key differences from Scenario 1**:
- No Stripe card fields appear at checkout
- Payment method shows "Cash on Delivery" throughout
- Order is created immediately (no payment authorization step)
- Cash payment is collected by the driver at delivery
- No tip prompt appears after delivery (cash tips are direct)
- Cash commission should be tracked (verifiable in System Admin if invoice reporting exists)

**Steps**: Follow all steps from Scenario 1 (1.1 → 1.6), but:
- In Step 1.1: Select **"Pay with Cash"** instead of "Pay by Card"
- No card entry is needed at checkout
- In Step 1.6: Note that no tip prompt should appear for cash orders

**Additional Pass Criteria**:
- ✅ No Stripe card fields shown at checkout
- ✅ Order confirmation shows "Cash on Delivery" as payment method
- ✅ No tip prompt after delivery for cash orders
- ✅ Order is visible in System Admin with payment method "Cash"

**Edge Cases**:
- Cash payment option not available for the vendor → verify the vendor settings in Vendor Admin before starting

---

## Scenario 3: Card Payment Declined at Checkout

**Summary**: The customer attempts to pay with a declined test card. The order should NOT be created.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: add items to basket and proceed to checkout
2. Enter a valid in-zone delivery address
3. Select "Pay by Card"
4. Enter the **declined test card**: `4000 0000 0000 0002`, any future expiry, any CVV
5. Click "Place Order"

**Expected**:
- Payment is declined
- Error message appears on the checkout page: "Your card was declined. Please try a different payment method."
- The checkout page remains intact (basket and address are not cleared)
- **No order is created** in Vendor Admin
- **No order is created** in System Admin

**Verify in Vendor Admin**:
- No new order should appear in the "Pending" column

**Verify in System Admin**:
- No new order should appear in the Orders list for the customer

**Pass Criteria**:
- ✅ Payment declined message is shown
- ✅ Checkout remains intact for retry
- ✅ No order created in Vendor Admin
- ✅ No order created in System Admin

**Follow-up**:
After verifying the decline, re-attempt checkout with the success card `4242 4242 4242 4242` and confirm the order is placed successfully.

---

## Scenario 4: Vendor Rejects Order

**Summary**: The vendor rejects a placed order. The customer's payment is voided/refunded.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: place a card order (as per Scenario 1, Step 1.1)
2. Note the Order ID and confirm the order appears in Vendor Admin "Pending"
3. As Vendor: open the order in the kanban
4. Click **"Reject"**, select a reason (e.g., "Too busy")
5. Confirm the rejection

**Expected**:
- Success message in Vendor Admin: "Order rejected"
- Order moves to **"Cancelled"** column in Vendor Admin
- Customer Front-end shows order as "Cancelled" with a reason
- Customer receives a notification: "Your order has been rejected — [reason]"
- Payment authorization is **voided** (not charged) — verify this in Stripe test dashboard if accessible

**Verify in Customer Front-end**:
- Order History shows the order as "Cancelled"
- No charge appears on the card used

**Verify in System Admin**:
- Order status shows "Rejected/Cancelled"
- Payment status shows "Voided" or "Not charged"

**Pass Criteria**:
- ✅ Order is in "Cancelled" column in Vendor Admin
- ✅ Customer sees "Cancelled" status
- ✅ Customer is notified with rejection reason
- ✅ No payment is charged

---

## Scenario 5: Vendor Does Not Respond (Acceptance Timeout)

**Summary**: The vendor ignores the order and does not accept or reject within the configured timeout period. The system should auto-cancel and release the payment authorization.

**Note**: This test requires knowing the acceptance timeout window (typically 2–10 minutes in the UAT environment). Confirm the timeout duration with your project manager before starting.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: place a card order
2. Note the Order ID
3. As Vendor: do **not** accept or reject the order — leave it in "Pending"
4. Wait for the configured acceptance timeout period
5. After the timeout, observe all portals

**Expected**:
- The order is automatically cancelled after the timeout
- Vendor Admin shows the order in the "Cancelled" column with a reason: "Acceptance timed out" or similar
- Customer Front-end shows the order as "Cancelled"
- Customer receives a notification: "Your order was cancelled as the restaurant did not respond in time"
- Payment authorization is voided (not charged)

**Pass Criteria**:
- ✅ Order is auto-cancelled after timeout
- ✅ All portals reflect the cancellation
- ✅ Customer is notified
- ✅ No payment charged

**Edge Cases**:
- Timeout does not trigger → report as bug (critical, as customers would be waiting indefinitely)

---

## Scenario 6: Customer Cancels Before Vendor Accepts

**Summary**: The customer cancels their order before the vendor has accepted it. This is the cleanest cancellation path — no preparation has started.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: place a card order
2. Note the Order ID
3. As Vendor: **do not** accept or reject the order
4. As Customer: immediately navigate to the order in Order History
5. Click **"Cancel Order"**, select a reason, confirm

**Expected**:
- Success message: "Order cancelled successfully"
- Order status in Customer Front-end changes to "Cancelled"
- Vendor Admin order moves to "Cancelled" column
- Payment authorization is voided (card not charged)
- Customer receives cancellation confirmation

**Pass Criteria**:
- ✅ Order is cancelled
- ✅ All portals reflect cancellation
- ✅ Payment is voided

**Edge Cases**:
- Small timing window: vendor accepts the order at the same moment the customer cancels → system should handle this race condition gracefully (one action wins; the other is notified)

---

## Scenario 7: Customer Cancels After Vendor Accepts

**Summary**: The customer cancels after the vendor has accepted but before the driver has picked up. This is a more complex cancellation — the vendor may have already started preparation.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: place a card order
2. As Vendor: accept the order (Step 1.2)
3. As Customer: navigate to Order History and attempt to cancel the order

**Expected** (two possible outcomes — check which your platform supports):

**Option A** (soft cancel allowed post-acceptance):
- Customer can cancel, but may incur a cancellation fee
- Order is cancelled with a message: "Cancellation fee applied" (if applicable)
- Partial refund is issued (if cancellation fee policy exists)

**Option B** (cancel blocked post-acceptance):
- Cancel button is disabled or hidden after vendor acceptance
- Message displayed: "This order can no longer be cancelled as the restaurant has started preparing your order. Please contact support."

**Verify**:
- If Option A: order in Vendor Admin shows as cancelled; refund is initiated
- If Option B: order remains active; "Cancel" button is hidden

**Pass Criteria**:
- ✅ System behaves consistently with one of the two options above
- ✅ Customer receives clear communication about what happened
- ✅ Payment is handled correctly (voided, partial refund, or no refund as per policy)

---

## Scenario 8: Driver Cannot Be Found (No Available Drivers)

**Summary**: The vendor has no available drivers to assign. The order enters an "awaiting driver" state.

**Portals involved**: Vendor Admin, Customer Front-end

**Setup**: Before this test, ensure the test driver is **off shift** (ended their shift in the Driver Portal).

**Steps**:
1. As Customer: place an order
2. As Vendor: accept the order and mark it as ready
3. As Vendor: attempt to assign a driver — observe the driver list
4. If no drivers are available, observe the order state

**Expected**:
- Vendor Admin shows "No available drivers" in the driver assignment dialog
- Order remains in "Ready" column with a visual indicator (e.g., "Awaiting Driver")
- Customer Front-end may show "Preparing your order" with a longer estimated wait
- No automated cancellation occurs (Vendor is notified to resolve)

**Pass Criteria**:
- ✅ "No available drivers" message is clear
- ✅ Order remains in system (not auto-cancelled)
- ✅ Customer is not given a false "driver on the way" status

**Follow-up**:
Start the test driver's shift, then assign them to the order and verify the assignment succeeds.

---

## Scenario 9: Driver Assigned, Then Unassigned and Reassigned

**Summary**: The vendor assigns a driver, then removes them and assigns a different driver. Both driver changes should be reflected across portals.

**Portals involved**: Vendor Admin, Driver Portal, Customer Front-end

**Setup**: Have two driver accounts available, or simulate by assigning and unassigning the same test driver.

**Steps**:
1. As Customer: place an order
2. As Vendor: accept and mark the order as ready
3. As Vendor: assign the test driver to the order
4. Verify the assignment in the Driver Portal (driver sees the delivery)
5. As Vendor: unassign the driver ("Unassign Driver")
6. Verify in Driver Portal that the delivery has been removed
7. As Vendor: assign a new driver (or the same driver again)
8. Verify the reassignment in the Driver Portal

**Expected**:
- Each assignment change is reflected in the Driver Portal immediately
- Customer Front-end should not show incorrect driver information during the reassignment window

**Pass Criteria**:
- ✅ Unassignment removes delivery from original driver's portal
- ✅ Reassignment shows delivery in new driver's portal
- ✅ Customer Front-end shows current driver only

**Edge Cases**:
- Unassigning after driver has already picked up the order → should be blocked or warned

---

## Scenario 10: Delivery Marked as Failed

**Summary**: The driver cannot complete the delivery (e.g., customer not home). The delivery is marked as failed and the appropriate resolution is applied.

**Portals involved**: Driver Portal, Vendor Admin, Customer Front-end, System Admin

**Steps**:
1. Follow Scenario 1 Steps 1.1–1.4 (order placed, accepted, ready, driver assigned)
2. As Driver: in the Active Deliveries, open the delivery
3. Click **"Mark as Failed"** or **"Report Issue"**
4. Select a reason: "Customer not home"
5. Confirm the action

**Expected**:
- Success message in Driver Portal: "Delivery issue reported"
- Delivery moves to Driver Portal delivery history with status "Failed"
- Vendor Admin shows the order with a "Delivery Failed" or "Issue Reported" status
- Customer Front-end shows the order as "Delivery Failed" or similar
- Customer receives a notification: "We couldn't deliver your order — [reason]"
- For card payment: a refund process is initiated (verify in System Admin)
- System Admin shows the order with the failure reason and refund status

**Pass Criteria**:
- ✅ Failed delivery is recorded across all portals
- ✅ Customer is notified with reason
- ✅ Refund is initiated for card payments

**Edge Cases**:
- Failing a cash order → driver has not collected cash; no refund needed; order should just be cancelled

---

## Scenario 11: Tip Added After Delivery (Card Payment)

**Summary**: The customer adds a digital tip after the order is delivered. This is a separate Stripe payment and should not apply a platform commission.

**Portals involved**: Customer Front-end, System Admin

**Prerequisite**: Complete Scenario 1 in full (order delivered via card payment).

**Steps**:
1. As Customer: navigate to the completed order in Order History
2. Look for a **"Leave a Tip"** or **"Add Tip"** prompt (may also appear on the delivery confirmation screen)
3. Select a tip amount (e.g., £2.00)
4. Confirm and process the tip payment
5. Verify the tip payment goes through

**Expected**:
- Success message: "Tip added — thank you!"
- Order details page shows the tip amount
- A separate Stripe charge (tip amount only) is processed
- The charge goes to the vendor's connected account with **no platform application fee**

**Verify in System Admin**:
- The order shows the tip amount recorded
- The tip payment is separate from the main order charge

**Pass Criteria**:
- ✅ Tip is processed
- ✅ Order shows tip amount
- ✅ Tip is a separate transaction
- ✅ No commission on tip (verify if System Admin shows commission breakdown)

**Edge Cases**:
- Double-clicking the tip submit → only one charge should be processed (idempotency)
- Attempting to tip on a cash order → tip option should not be available

---

## Scenario 12: Full Refund After Vendor Cancellation

**Summary**: The vendor cancels an accepted order (e.g., key ingredient unavailable). The customer receives a full refund.

**Portals involved**: Vendor Admin, Customer Front-end, System Admin

**Steps**:
1. As Customer: place a card order and confirm it appears in Vendor Admin "Pending"
2. As Vendor: accept the order (order moves to "Accepted")
3. As Vendor: open the accepted order and click **"Cancel Order"**
4. Enter a cancellation reason: "Item unavailable"
5. Confirm the cancellation

**Expected**:
- Success message in Vendor Admin: "Order cancelled"
- Order moves to "Cancelled" column in Vendor Admin
- Customer Front-end shows order as "Cancelled — [reason]"
- Customer receives notification: "Your order has been cancelled by the restaurant — Item unavailable. A full refund has been issued."
- Full refund is initiated via the vendor's Stripe account
- Refund appears in System Admin under the order details

**Verify in System Admin**:
- Order status shows "Cancelled/Refunded"
- Refund amount matches the original order total

**Pass Criteria**:
- ✅ Order cancelled in all portals
- ✅ Customer notified with reason
- ✅ Full refund initiated
- ✅ Refund visible in System Admin

**Edge Cases**:
- Vendor cancels but refund fails → System Admin should show an error/warning; customer should be notified separately

---

## Scenario 13: Promo Code Applied to Order

**Summary**: The customer applies a vendor-specific promo code at checkout. The discount is applied, and the correct amount is charged.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Setup**: Ensure an active promo code exists for the test vendor (created in Vendor Admin under Offers — see [03-vendor-admin-uat.md](03-vendor-admin-uat.md#promotions-and-offers)).

**Steps**:
1. As Customer: add items to basket from the test vendor (ensure order total exceeds any minimum order value for the promo)
2. Proceed to checkout
3. Enter the promo code in the "Promo Code" field and click "Apply"
4. Verify the discount is applied in the order summary
5. Complete checkout with the Stripe test card

**Expected**:
- Promo code is accepted: "Promo applied — [discount] off your order!"
- Order total is reduced by the discount amount
- Order Confirmation page shows the discounted total and the discount line item
- Vendor Admin order shows the discounted total
- System Admin order shows both the full amount and the discount applied

**Pass Criteria**:
- ✅ Promo code is accepted
- ✅ Discount applied correctly
- ✅ Correct amount is charged (discounted total, not original total)
- ✅ Promo usage increments by 1 in Vendor Admin offers (verify offer usage count)

**Edge Cases**:
- Promo code typed with incorrect case (e.g., "save20" vs "SAVE20") → should be case-insensitive or show clear error
- Order total below minimum → clear error message with minimum order value shown
- Promo code usage limit reached → error: "This offer has reached its usage limit"

---

## Scenario 14: Quote Expires Before Payment

**Summary**: The customer generates a checkout quote but waits too long before paying. The quote expires and the checkout should be blocked until a new quote is generated.

**Portals involved**: Customer Front-end

**Steps**:
1. As Customer: add items to basket and proceed to checkout
2. Enter a valid delivery address and allow the quote to generate
3. Leave the checkout page open for the quote TTL period (typically 2–5 minutes — confirm with project manager)
4. After the TTL has elapsed, click "Place Order" (or attempt to submit payment)

**Expected**:
- The system detects the expired quote
- Error message: "Your delivery quote has expired. Please refresh to get a new quote."
- The order is not placed
- Refreshing the address (or clicking "Refresh Quote") generates a new valid quote

**Pass Criteria**:
- ✅ Expired quote is detected
- ✅ Clear error message is shown
- ✅ New quote can be generated without losing basket
- ✅ No order is created in Vendor Admin for the expired-quote attempt

**Edge Cases**:
- Quote expiry message does not appear (order is placed with stale data) → critical bug

---

## Scenario 15: Duplicate Checkout (Double-Submit Prevention)

**Summary**: The customer accidentally double-clicks "Place Order". Only one order should be created.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Steps**:
1. As Customer: proceed to checkout with items in basket and a valid quote
2. Enter Stripe test card details
3. Click "Place Order" rapidly twice in quick succession (or try using the browser back button after submission)

**Expected**:
- Only **one** order is created
- Only **one** payment is charged
- The "Place Order" button is disabled or shows a loading state after the first click
- If two requests did somehow reach the server, idempotency keys ensure only one order is created

**Verify in Vendor Admin**:
- Only one order appears in the "Pending" column for this customer

**Verify in System Admin**:
- Only one order and one payment transaction are recorded

**Pass Criteria**:
- ✅ Only one order created
- ✅ Only one payment processed
- ✅ No duplicate order in any portal

**Edge Cases**:
- Back button used after order submission → re-submission should be blocked (quote already consumed)
- Network retry from browser → server should reject the duplicate via idempotency key

---

## Scenario 16: Vendor Deactivated Mid-Flow

**Summary**: A customer places and pays for an order. Whilst the vendor is preparing it, a platform administrator deactivates the vendor. This scenario verifies that the existing in-flight order is honoured and not silently cancelled, whilst simultaneously confirming that no new orders can be placed through that vendor. It also confirms that any other active orders from that vendor are equally unaffected.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Paired test cases**: TC-SA-034 (Deactivate Vendor), TC-XP-001 (Vendor Activate/Deactivate — Cross-Portal Impact)

---

### Setup

Before beginning, ensure the following are in place:

- **Test Restaurant 1** is active and has at least one published menu
- System Admin is open and the test administrator has access to the Vendors section
- Optionally, a second browser session is ready to attempt a fresh basket from the same vendor after deactivation (simulates a second customer)
- Stripe test mode is active

---

### Step 16.1 — Customer Places and Pays for an Order

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Navigate to **Test Restaurant 1** and add 2–3 dishes to the basket
3. Proceed to checkout, enter a valid in-zone delivery address, and confirm the delivery quote
4. Note the **quoted subtotal, delivery fee, and total**
5. Select **Pay by Card** and enter the Stripe test card `4242 4242 4242 4242`, any future expiry, any CVV
6. Click **"Place Order"** and note the **Order ID** from the confirmation page

**Expected**:
- Order Confirmation page appears with the Order ID and correct total
- The order appears in Vendor Admin **"Pending"** column within 60 seconds

**Pass Criteria**:
- ✅ Order placed with a confirmed Order ID
- ✅ Order visible in Vendor Admin "Pending" column

---

### Step 16.2 — Vendor Accepts and Begins Preparation

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the new order from the **"Pending"** column
2. Click **"Accept"** — the order moves to **"Accepted"**
3. Leave the order in the **"Accepted"** state (do not mark as Ready — simulate preparation in progress)

**Expected**:
- Order is in **"Accepted"** column in Vendor Admin
- Customer Front-end shows **"Being Prepared"** (or equivalent)

**Pass Criteria**:
- ✅ Order accepted and in preparation state

---

### Step 16.3 — System Admin Deactivates the Vendor

**Actor**: System Admin

**Steps**:
1. In System Admin, navigate to **Vendors** and locate **Test Restaurant 1**
2. Open the vendor record and set its status to **Inactive** (TC-SA-034)
3. Click **Save** and confirm the success message or toast notification
4. Take a screenshot showing the vendor record with **"Inactive"** status

**Expected**:
- Vendor record shows Inactive status
- No automatic cancellation of in-flight orders is triggered

---

### Step 16.4 — Cross-Portal Verification

Wait **30 seconds** after deactivation and hard-refresh (Ctrl+F5 / Cmd+Shift+R) all customer-facing browser windows before verifying. If the vendor is still visible, run **Platform Tools → Rebuild Caches → Reindex Vendors** and wait a further 30 seconds.

**Customer Front-end — the existing in-flight order**:
- ☐ The in-flight order placed in Step 16.1 is **still visible** in Order History with status **"Being Prepared"** — it has **not** been cancelled or altered
- ☐ The customer has **not** received a cancellation or interruption notification relating to this order

**Customer Front-end — discovery surfaces**:
- ☐ **Home page** (`/`): Test Restaurant 1 is **no longer visible** in the discovery grid
- ☐ **Search** (`/search?q=test`): Test Restaurant 1 does **not** appear in results
- ☐ **Cuisine page** (relevant cuisine): Test Restaurant 1 is **not listed**
- ☐ **Direct vendor URL** (`/vendor/test-restaurant-1`): Shows a "Restaurant not available" message or 404-style page — **not** the live menu

**Customer Front-end — fresh basket attempt (second session)**:
- ☐ Attempting to add a dish from Test Restaurant 1 in a new session either shows a disabled "Add to Basket" button or is blocked at checkout with a clear error
- ☐ Any orphaned basket items (added before deactivation in a second session) display a clear warning such as **"This restaurant is no longer available"** — they do **not** silently remain in the basket at the old price

**Vendor Admin**:
- ☐ The in-flight order is **still visible** in the **"Accepted"** column
- ☐ The vendor can still advance the order (Mark as Ready → Assign Driver) — vendor deactivation does not block order progression
- ☐ No system-generated cancellation notification or status change has been applied to the order card

**System Admin**:
- ☐ The in-flight order still shows its current status (Accepted) — it has **not** been moved to Cancelled
- ☐ The vendor record shows **Inactive** status
- ☐ The order record still references the (now inactive) vendor correctly — no data corruption

**Pass Criteria**:
- ✅ Existing in-flight order is not auto-cancelled by vendor deactivation
- ✅ Vendor Admin can still manage and progress the in-flight order to delivery
- ✅ No new orders can be placed with the deactivated vendor (discovery and checkout both blocked)
- ✅ Discovery surfaces (home, search, direct URL) no longer show the vendor after propagation

---

### Step 16.5 — Continue Order to Delivery

**Actor**: Vendor Admin, Driver

**Steps**:
1. As Vendor: mark the in-flight order as **Ready**, then assign the test driver
2. As Driver: accept the delivery, pick up, and mark as **Delivered**
3. Confirm the full order lifecycle completes without error despite the vendor being inactive

**Expected**:
- The order completes through all remaining lifecycle steps without error
- System Admin shows the order as **Delivered/Completed**
- Customer receives the **"Your order has been delivered"** notification

**Pass Criteria**:
- ✅ In-flight order delivered successfully — vendor deactivation has no adverse effect on existing orders

---

### Step 16.6 — Reactivation Check

**Actor**: System Admin

**Steps**:
1. In System Admin, reactivate **Test Restaurant 1**
2. Wait 30 seconds and hard-refresh the customer home page
3. Confirm Test Restaurant 1 reappears in all discovery surfaces listed in Step 16.4

**Pass Criteria**:
- ✅ Vendor visible again on home page, search, cuisine page, and direct URL after reactivation

**Edge Cases**:
- Vendor deactivated whilst a second customer is mid-checkout on the vendor's page → checkout validation should detect the inactive vendor and block payment; no order should be created
- Two simultaneous in-flight orders from the same vendor at the moment of deactivation → both should be unaffected and completable independently
- Vendor deactivated whilst a driver is mid-delivery → the driver portal should still show the active delivery; the handover must complete regardless of vendor status

---

## Scenario 17: Menu Republished Mid-Checkout

**Summary**: A customer adds a dish to their basket and is sitting on the address and delivery-quote screen. Whilst the customer is on that screen, the vendor edits the published menu — removing the dish the customer has in their basket — and republishes. This scenario confirms that the customer's basket is handled gracefully: either with a clear warning or by removing the unavailable item. Critically, it verifies that no broken checkout state results if the customer attempts to pay before the change is detected on the client side, and that the server-side basket validation catches the stale state.

**Portals involved**: Customer Front-end, Vendor Admin

**Paired test cases**: TC-VA-038 (Edit Published Menu), TC-VA-043 (Republish Menu), TC-XP-004 (Publish/Unpublish Menu — Cross-Portal Impact)

---

### Setup

Before beginning:

- Identify a dish in the test vendor's published menu that you are willing to temporarily remove — call it **Dish X** throughout this scenario
- Confirm that removing Dish X will not reduce the basket below any minimum order value for the test
- Have both Vendor Admin and Customer Front-end open in separate windows, and co-ordinate closely — timing is important in this scenario

---

### Step 17.1 — Customer Adds Dish to Basket and Pauses on Quote Screen

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Navigate to **Test Restaurant 1** and add **Dish X** plus at least one other dish to the basket
3. Proceed to checkout and enter a valid in-zone delivery address
4. Allow the delivery quote to generate
5. **Pause here — remain on the address/quote screen, do not proceed to payment**
6. Note the basket contents (items, quantities, prices) and the quoted delivery fee and total

**Pass Criteria**:
- ✅ Customer is paused on the address/quote screen with Dish X in the basket and a valid quote displayed

---

### Step 17.2 — Vendor Removes Dish X and Republishes the Menu

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the **Menu Builder**
2. Locate **Dish X** and either delete it from the menu or mark it as unavailable
3. Save the change and **republish the menu** (TC-VA-038 + TC-VA-043)
4. Confirm the publish success message
5. Take a screenshot showing the updated, published menu with Dish X absent

**Pass Criteria**:
- ✅ Menu republished successfully without Dish X

---

### Step 17.3 — Customer Attempts to Advance from the Quote Screen

**Actor**: Customer

**Steps**:
1. Without refreshing the page, click **"Continue"** or **"Proceed to Payment"** from the quote screen
2. Observe the basket and checkout state carefully
3. If the client-side still shows Dish X and allows you to reach the payment screen, enter the Stripe test card `4242 4242 4242 4242` and attempt to submit payment
4. Record exactly what happens at each stage — the client-side response and (if payment is attempted) the server-side response

**Expected** — one of the following outcomes:

| Outcome | Description |
|---------|-------------|
| A — Client-side warning | The checkout page immediately shows a warning: **"One or more items in your basket are no longer available"** and prevents proceeding |
| B — Silent client removal | Dish X has been removed from the basket silently; the total has updated; the customer can proceed with the remaining items |
| C — Server-side block | The client allows progression to payment but the server rejects the submission with a basket validation error |

All three outcomes are potentially acceptable — document which occurred. **Outcome D (payment succeeds for a removed dish) is a critical bug.**

---

### Step 17.4 — Cross-Portal Verification

**Customer Front-end**:
- ☐ The customer **cannot** complete payment for Dish X if it has been removed from the menu — either the client-side or server-side must catch this
- ☐ If Outcome A: a clear warning message is shown at the checkout step; the basket or error clearly identifies which item is unavailable
- ☐ If Outcome B: the basket total has updated correctly to remove Dish X's price; the customer is not charged for it
- ☐ If Outcome C: the server returns a clear validation error identifying the unavailable item; the checkout page remains intact for correction
- ☐ Refreshing the checkout page causes the basket to reflect the updated menu (Dish X absent or flagged as unavailable)

**Vendor Admin**:
- ☐ No ghost or partial order appears in the **"Pending"** column from any failed checkout attempt
- ☐ The published menu no longer shows Dish X

**Pass Criteria**:
- ✅ Customer cannot successfully pay for a dish that no longer exists on the menu
- ✅ The basket response (warning, removal, or server error) is clear — no silent failure
- ✅ No orphaned order is created in Vendor Admin from a failed checkout attempt

**Edge Cases**:
- Customer submits payment at the exact moment the menu is republished → the payment should either succeed (old state captured before the republish) or fail with a clear error; it must **never** result in payment taken but no order created
- Dish X is the **only** item in the basket → the basket should become empty and checkout must be fully blocked, not allowed to proceed with a £0 order
- Customer refreshes the checkout page before the vendor republishes → baseline test; should work normally

**Note for testers**: After completing this scenario, restore Dish X to the menu and republish so subsequent scenarios are unaffected.

---

## Scenario 18: Vendor Edits an In-Progress Order

**Summary**: The customer's order has been accepted by the vendor. The vendor then uses the Edit Order function to substitute an out-of-stock item for an alternative dish and adjusts the quantity of a second item. This scenario verifies that the customer is notified of the change, that the Order Details page reflects the new items and totals, and that any payment delta is handled correctly — whether that means a partial refund, an additional charge request, or a simple total adjustment visible on the order record.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Paired test cases**: TC-VA-057 (Edit In-Progress Order)

---

### Setup

Before beginning:

- Confirm that the Edit Order functionality is available in Vendor Admin for the test vendor
- Plan the substitution in advance: identify the **dish to be removed** (call it **Dish A**), the **replacement dish** (call it **Dish B** — ideally at a different price), and the **item whose quantity will be changed** (call it **Dish C**)
- Record the **expected original total** and **expected new total** before the test begins — write these down so you can verify arithmetic later
- Bring the test order to the **"Accepted"** state (follow Scenario 1 Steps 1.1–1.2) before proceeding

---

### Step 18.1 — Customer Places Order and Vendor Accepts

**Actor**: Customer, then Vendor Admin

**Steps**:
1. As Customer: place a card order containing at least **three different dishes**, including Dish A and Dish C — note the exact items, quantities, and individual prices
2. As Vendor: accept the order — it moves to the **"Accepted"** column
3. Record the **original order total** from both the Customer Front-end Order Details page and the Vendor Admin order card — confirm they match

**Pass Criteria**:
- ✅ Order in "Accepted" state
- ✅ Original totals recorded and consistent across both portals

---

### Step 18.2 — Vendor Edits the Order

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the accepted order
2. Click **"Edit Order"** (TC-VA-057)
3. Remove **Dish A** from the order (simulating it being out of stock) and add **Dish B** in its place at the quantity originally ordered
4. Reduce the quantity of **Dish C** from 2× to 1×
5. Confirm and save the edits
6. Take a screenshot showing the **updated order summary in Vendor Admin** with Dish B present, Dish A absent, Dish C at reduced quantity, and the **new total**

**Pass Criteria**:
- ✅ Edit saved with a clear confirmation message
- ✅ Vendor Admin order card shows the updated items and the new calculated total

---

### Step 18.3 — Cross-Portal Verification

Wait approximately **30 seconds** for changes to propagate, then verify across all portals.

**Customer Front-end**:
- ☐ The customer has received a notification: **"Your order has been updated by the restaurant"** — record the exact wording
- ☐ The **Order Details page** shows the **new items** (Dish B present, Dish A absent, Dish C at the new quantity)
- ☐ The **new total** is displayed and matches the Vendor Admin total
- ☐ Both the **original total** and the **adjusted total** are visible on the Order Details page for customer reference
- ☐ If the new total is **lower** than the original: a partial refund line or credit note is visible; the difference is returned to the payment card
- ☐ If the new total is **higher** than the original: document the behaviour — is the customer prompted to authorise an additional charge, or is the difference handled outside the checkout flow? Record exactly what is shown

**Vendor Admin**:
- ☐ Order card reflects the updated items, quantities, and new total
- ☐ The vendor can continue progressing the order (Mark as Ready → Assign Driver) after editing — the edit does not lock the order

**System Admin**:
- ☐ The order record shows the **updated items and new total**
- ☐ If a payment delta was applied: the payment section shows the refund or supplementary charge amount and its status
- ☐ The audit trail (if accessible) records the edit event with the old and new values for comparison

**Pass Criteria**:
- ✅ Customer is notified of the order change
- ✅ Customer Front-end reflects the updated items and correct new total
- ✅ System Admin shows the correct financial record for the post-edit state
- ✅ Order can be progressed to delivery without errors after the edit

**Edge Cases**:
- Vendor attempts to increase the order total beyond the original authorised Stripe amount → document whether this is permitted, blocked, or requires a supplementary capture
- Vendor edits the order to remove all items (effectively an empty order) → the system should either block this as a logical error or treat it as a cancellation, not create a £0.00 order
- Customer is viewing the Order Details page at the exact moment the edit is saved → the page should either update live or display a **"This order has been updated"** banner prompting a refresh

---

## Scenario 19: Full Refund via Vendor Cancellation (Card Order, Post-Capture)

**Summary**: The vendor cancels a card-paid order after the Stripe payment has already been **captured** — i.e., the money has been taken from the customer's account, not merely authorised. WantFood does not offer a partial-refund UI; cancellation always triggers a full refund. This scenario verifies that the refund is processed end to end: Stripe receives the refund instruction, the customer's test inbox receives the refund email, the Stripe test dashboard shows the refund entry, and the commission line for the order is reversed so no net revenue is recorded against a fully refunded sale.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin, Stripe test dashboard

**Paired test cases**: TC-VA-056 (Vendor Cancels Order)

> **Note on capture timing**: In the UAT environment, Stripe test card payments are typically captured immediately upon placement. If the integration uses an "authorise first, capture later" flow, ensure the payment has been captured before performing this test — otherwise the scenario reduces to a void, which is already covered by Scenario 4.

---

### Setup

Before beginning:

- Confirm the test vendor's Stripe account is connected in Vendor Admin (**Payment Methods** section)
- Open the **Stripe test dashboard** at [dashboard.stripe.com](https://dashboard.stripe.com) and switch to **Test mode** — navigate to **Payments** so you can observe incoming and outgoing transactions in real time
- Have System Admin open in a second window

---

### Step 19.1 — Customer Places and Pays (Payment Captured)

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Add 2–3 dishes to the basket and proceed to checkout
3. Enter the Stripe test card `4242 4242 4242 4242`, any future expiry, any CVV
4. Click **"Place Order"** and note the **Order ID** and the exact **order total** from the confirmation page
5. In the Stripe test dashboard, locate the new payment — note the **Stripe Payment Intent ID** and confirm the payment status shows **"Succeeded"** (captured, not merely authorised)

**Pass Criteria**:
- ✅ Payment captured in Stripe with status "Succeeded"
- ✅ Order ID and exact total noted

---

### Step 19.2 — Vendor Accepts the Order

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, accept the order — it moves to the **"Accepted"** column
2. Do **not** mark it as Ready — leave it in "Accepted" to simulate cancellation occurring mid-preparation

**Pass Criteria**:
- ✅ Order in "Accepted" state

---

### Step 19.3 — Vendor Cancels the Order

**Actor**: Vendor Admin

**Steps**:
1. In Vendor Admin, open the accepted order
2. Click **"Cancel Order"** (TC-VA-056)
3. Select the cancellation reason: **"Item unavailable"**
4. Confirm the cancellation

**Expected**:
- Success message: **"Order cancelled"**
- Order moves to **"Cancelled"** column in Vendor Admin

**Pass Criteria**:
- ✅ Cancellation confirmed in Vendor Admin with a success message

---

### Step 19.4 — Cross-Portal Verification

**Customer Front-end**:
- ☐ Order History shows the order as **"Cancelled"** with the reason **"Item unavailable"**
- ☐ The customer has received a cancellation and refund notification email — check the test inbox (from `noreply@wantfood.com`); allow up to five minutes and check Junk before raising a bug
- ☐ The email states clearly that a **full refund** has been issued and provides an approximate timescale (typically 5–10 business days for real cards; instant in Stripe test mode)

**Stripe test dashboard**:
- ☐ The original payment (located by the Stripe Payment Intent ID noted in Step 19.1) now shows a **Refund** entry
- ☐ The refund amount **exactly matches** the original order total — not a lesser amount, and not zero
- ☐ The refund status is **"Pending"** or **"Succeeded"** — not "Failed"

**System Admin**:
- ☐ The order record shows status **"Cancelled/Refunded"**
- ☐ The financial summary for the order shows: original charge, refund amount, and a **net revenue of £0.00**
- ☐ The **commission line** for this order has been reversed or shows as £0.00 — there must be no outstanding commission charge against a cancelled, fully refunded order
- ☐ An audit trail is present: payment captured → order accepted → order cancelled → refund issued

**Pass Criteria**:
- ✅ Full refund processed in Stripe — refund amount equals original order total
- ✅ Customer email confirms the refund
- ✅ Commission reversed in System Admin — no net revenue against a refunded order
- ✅ Order status in all portals is Cancelled/Refunded

**Edge Cases**:
- Vendor attempts to cancel after the driver has already picked up the order → cancellation should be blocked or at minimum require an explicit warning, as the driver is already in transit with the food
- Stripe refund API call fails → System Admin should surface a clear error; customer should be notified separately and a manual refund flag should be raised for engineering
- Cancelling a cash-payment order via TC-VA-056 → confirm that no Stripe refund call is attempted for a cash order; the cancellation should simply update the order status

---

## Scenario 20: Stripe Webhook Delayed

**Summary**: The customer pays by card, but the Stripe webhook notification is delayed by more than 30 seconds before it reaches the WantFood backend. This scenario verifies that the customer confirmation page shows a graceful polling or loading state rather than a premature error or false success, and that the order is created correctly once the webhook eventually arrives. It also confirms that no duplicate orders are created during the polling window and that the customer can always retrieve the order from their history regardless of whether they remained on the confirmation page.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Paired test cases**: TC-FE-053 (Checkout Polling / Duplicate Submit Prevention)

> **Simulating a delayed webhook**: The most reliable approach in the UAT environment is to use the **Stripe CLI** (`stripe listen --forward-to <webhook-endpoint-url>`). Immediately after the customer clicks "Place Order", pause or kill the listener — then restart it 30–60 seconds later. If the Stripe CLI is not available, a second option is to use the **Delayed delivery** setting on the webhook endpoint in the Stripe Dashboard (if supported in your Stripe account's test mode). If neither is available, the test can be run by monitoring natural network conditions and documenting the state machine you observe; note in your test report that simulation was not possible.

---

### Setup

Before beginning:

- Obtain the Stripe webhook endpoint URL for the UAT environment from your project manager
- If using the Stripe CLI: install it, run `stripe login`, and confirm you can receive test events
- Have System Admin and Vendor Admin open in separate windows to observe order state in real time
- Have a timer or stopwatch available to measure the delay accurately

---

### Step 20.1 — Customer Proceeds to the Payment Screen

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Add dishes to the basket, enter a valid delivery address, and proceed to the payment screen
3. Enter the Stripe test card `4242 4242 4242 4242`, any future expiry, any CVV
4. **Immediately before** clicking "Place Order", take action to delay webhook delivery:
   - If using Stripe CLI: press Ctrl+C to pause the listener
   - If using Stripe Dashboard delay setting: apply a 60-second delay to the webhook endpoint
5. Start the timer, then click **"Place Order"**

---

### Step 20.2 — Observe the Polling State Within the First 30 Seconds

**Actor**: Customer (observe immediately after submission)

**Steps**:
1. Note the state of the Customer Front-end within the first 5–10 seconds after clicking "Place Order":
   - Does it show a **"Processing your order…"** spinner or loading indicator? (TC-FE-053)
   - Does it immediately display an order confirmation? (Unexpected — payment not confirmed yet)
   - Does it show an error?
2. Note the precise screen content, including any polling message or countdown
3. Check Vendor Admin and System Admin — **no order should appear** yet, as the webhook has not been delivered

**Expected**:
- Customer Front-end shows a **polling / "processing" state** — not an error and not a premature success confirmation
- Vendor Admin shows no new order in the "Pending" column
- System Admin shows no new order record

**Pass Criteria**:
- ✅ A clear polling/loading state is displayed on the Customer Front-end
- ✅ No order appears in Vendor Admin or System Admin before the webhook arrives

---

### Step 20.3 — Webhook Arrives (Resume)

**Actor**: Tester

**Steps**:
1. After **30–60 seconds** have elapsed, resume webhook delivery:
   - Stripe CLI: restart the listener with `stripe listen --forward-to <webhook-endpoint-url>`
   - Stripe Dashboard: remove the delay setting or allow the delayed window to expire
2. Start watching all portals — the webhook should be processed within a few seconds of the listener restarting
3. Note the exact time the webhook arrives (stop the timer)

---

### Step 20.4 — Cross-Portal Verification (Post-Webhook)

**Customer Front-end**:
- ☐ The polling state transitions to the **Order Confirmation page** showing the Order ID and the correct total — no error is displayed
- ☐ The total on the confirmation page matches the original quoted total — no discrepancy caused by the delay
- ☐ The customer receives a **confirmation email** within five minutes of the webhook arriving
- ☐ The order is visible in the customer's **Order History** (even if they navigated away during the polling window)

**Vendor Admin**:
- ☐ The order appears in the **"Pending"** column within 60 seconds of the webhook being processed
- ☐ The order details (items, total, delivery address) are correct — the delay has not corrupted any data

**System Admin**:
- ☐ The order record is created with the correct status and payment details
- ☐ The Stripe payment record on the order shows **"Succeeded"** with the actual webhook receipt timestamp

**Pass Criteria**:
- ✅ Order created correctly once the webhook arrives
- ✅ Customer receives the correct confirmation — not an error
- ✅ No duplicate orders are created during the polling window
- ✅ Order total is consistent across all portals

**Edge Cases**:
- Customer closes the browser tab during the polling state → the order should still be created when the webhook arrives; the customer should be able to find it in Order History without the confirmation page having been shown
- Webhook never arrives (Stripe delivery failure) → after a configurable timeout the system should either void the payment and notify the customer, or surface the issue in System Admin for manual resolution — do not leave the customer in a permanent "processing" state
- Webhook arrives twice (Stripe retry behaviour) → the second delivery must be rejected as a duplicate via idempotent processing; only one order should exist

---

## Scenario 21: Order Placed Outside Vendor Trading Hours

**Summary**: The test vendor's trading hours have been deliberately configured so that the current time falls outside the open window. A customer attempts to browse the vendor, add dishes to their basket, and proceed to checkout. This scenario verifies that immediate checkout is blocked with a clear "Closed now" message and that the next opening time is shown. If the vendor has scheduled ordering enabled, the scenario also verifies that the customer is offered the ability to schedule an order for the next available slot and that the scheduled order is handled correctly across portals.

**Portals involved**: Customer Front-end, Vendor Admin

**Paired test cases**: TC-VA-022 (Configure Trading Hours), TC-VA-023 (Configure Scheduled Orders), TC-XP-023 (Trading Hours — Cross-Portal Impact)

---

### Setup

Before beginning:

- In Vendor Admin, navigate to **Test Restaurant 1 → Branch settings** and configure trading hours (TC-VA-022) so that the **current time falls outside the open window** — for example, if running the test at 14:00, set trading hours to 18:00–22:00 only for today
- Note the **next opening time** precisely (e.g., 18:00 today) — you will need to verify this is shown to the customer
- **Path A — test scheduled orders**: ensure **AcceptsScheduledOrders** is **enabled** (TC-VA-023) with a schedule picker that includes the next opening slot
- **Path B — test blocked-only behaviour**: set **AcceptsScheduledOrders** to **disabled**
- Hard-refresh the Customer Front-end after saving the trading hours change and wait for propagation (30 seconds / Rebuild Caches if needed)

---

### Step 21.1 — Customer Navigates to the Closed Vendor

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Navigate directly to **Test Restaurant 1**'s vendor page (`/vendor/test-restaurant-1`)
3. Observe the vendor page — note any "Closed" indicator, banner, or badge
4. Attempt to **add a dish to the basket**

**Expected**:
- The vendor page displays a clear **"Closed now"** indicator with the next opening time (TC-XP-023)
- Either the **add-to-basket button is disabled**, or the dish is added but checkout is blocked at the next step

**Pass Criteria**:
- ✅ Vendor page clearly communicates the vendor is currently closed

---

### Step 21.2 — Customer Attempts to Proceed to Checkout

**Actor**: Customer

**Steps**:
1. If dishes were addable in Step 21.1, proceed to the checkout screen
2. Attempt to advance to the address and quote stage

**Expected**:
- Checkout is **blocked** with a message such as: **"Test Restaurant 1 is currently closed. They open at [next opening time]."**
- Payment is not possible for an immediate order

**Pass Criteria**:
- ✅ Immediate checkout blocked with a clear "Closed" message and the correct next opening time

---

### Step 21.3 — Scheduled Order Path (Path A — AcceptsScheduledOrders Enabled Only)

> **Only perform this step if AcceptsScheduledOrders is enabled for the test vendor.**

**Actor**: Customer

**Steps**:
1. At the checkout block screen, look for a **"Schedule for later"** option or a date/time picker
2. Select the **next available opening slot** (e.g., Today at 18:00) from the slot picker
3. Confirm the scheduled time, enter a valid delivery address, and confirm the delivery quote
4. Complete payment using the Stripe test card `4242 4242 4242 4242`
5. Note the Order ID and the scheduled delivery time on the confirmation page

**Expected**:
- The scheduled order is accepted and an Order ID is issued
- The Order Confirmation page shows the scheduled delivery slot prominently
- The customer receives a confirmation email that includes the scheduled delivery time

**Pass Criteria**:
- ✅ Scheduled order placed for the next opening slot
- ✅ Confirmation clearly shows the scheduled delivery time

---

### Step 21.4 — Cross-Portal Verification

**Customer Front-end**:
- ☐ Vendor page shows **"Closed now"** with the next opening time visible — not just a generic unavailability message (TC-XP-023)
- ☐ Immediate checkout for delivery **now** is blocked — no order can be placed outside trading hours for same-time delivery
- ☐ (Path A) A schedule picker is offered with at least the next opening slot available — slots outside the vendor's configured trading hours are **not** selectable
- ☐ (Path B) No schedule picker appears — only the closed message is shown; there is no way to place any order

**Vendor Admin**:
- ☐ The vendor's trading hours are correctly configured and visible under Branch settings (TC-VA-022)
- ☐ (Path A) The scheduled order appears in the order kanban with the scheduled delivery time displayed on the order card
- ☐ (Path A) The order is **not** automatically accepted — the vendor must manually accept it when they open

**System Admin**:
- ☐ (Path A) The order record shows the scheduled delivery time
- ☐ No order records exist for the blocked immediate-checkout attempts

**Pass Criteria**:
- ✅ Immediate checkout blocked and "Closed" message shown correctly
- ✅ (Path A) Scheduled order placed successfully for the next available opening slot
- ✅ (Path B) No schedule picker shown — only the closed message

**Edge Cases**:
- Vendor opens at exactly the minute of the test → the "Closed" state should transition to "Open" within the cache propagation window; if it does not, follow the Propagation Cheat-Sheet before logging a bug
- Customer places a scheduled order, then the vendor changes trading hours again before the scheduled slot → document what happens to the pending scheduled order (log as Medium bug if the order is silently dropped)
- AcceptsScheduledOrders is enabled but no future slots exist within the vendor's configured schedule → the scheduler should not offer any slots and the customer should see a clear message, not an empty or broken slot picker

---

## Scenario 22: Stacked Offers — Platform Promotion and Vendor Promo Code

**Summary**: A customer applies both an active platform-wide offer (TC-SA-091) and a vendor-specific promo code (TC-VA-081) at the same checkout. This scenario documents the actual stacking behaviour — whether both discounts apply cumulatively, whether one supersedes the other, or whether a stacking error is raised. The basket must display clear, itemised discount lines, and the final checkout total must be arithmetically correct regardless of the stacking policy. **This test is deliberately observational: whatever the system does is the documented outcome.** If the maths is wrong, that is always a bug; if stacking is silently blocked with no message, that is a UX bug.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Paired test cases**: TC-SA-091 (Activate Platform Offer), TC-VA-081 (Create/Activate Vendor Promo Code), TC-XP-015 (Platform Offer — Cross-Portal Impact), TC-XP-016 (Vendor Offer — Cross-Portal Impact)

---

### Setup

Before beginning:

- In System Admin, activate a platform offer using TC-SA-091 — record its **trigger** (auto-applied or code), **discount type** (percentage or fixed), and **discount value** (e.g., `SUMMER10` — 10% off)
- In Vendor Admin, create or activate a vendor promo code using TC-VA-081 — record its **code**, **discount type**, and **discount value** (e.g., `NEWCUSTOMER` — £3.00 off)
- Confirm both offers are active simultaneously before starting
- Calculate the **expected total under each of three possible stacking outcomes** and write them down:
  1. Both discounts applied: subtotal − platform discount − vendor discount + delivery fee
  2. Platform offer only: subtotal − platform discount + delivery fee
  3. Vendor promo only: subtotal − vendor discount + delivery fee
- These pre-calculated figures are your reference for verifying the maths in Step 22.4

---

### Step 22.1 — Customer Builds Basket and Applies Platform Offer

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Add dishes from the test vendor to the basket, ensuring the total meets the minimum order value for both offers
3. Proceed to checkout and enter a valid delivery address
4. Observe whether the platform offer (e.g., SUMMER10) is **auto-applied** or whether it requires a code entry
5. If a code is required, enter it in the promo code field and click **"Apply"**
6. Note the basket total and the discount line shown after the platform offer is applied

**Pass Criteria**:
- ✅ Platform offer is applied (auto or via code entry)
- ✅ Platform discount line is visible in the basket summary

---

### Step 22.2 — Customer Enters Vendor Promo Code

**Actor**: Customer

**Steps**:
1. In the promo code field (if separate or reusable), enter the vendor promo code (e.g., `NEWCUSTOMER`)
2. Click **"Apply"**
3. Observe the response carefully and record exactly what happens:
   - Does the vendor code apply **on top of** the platform offer, reducing the total further? (**Stacked**)
   - Does the system replace the platform offer with the vendor code? (**Vendor supersedes platform**)
   - Does the platform offer remain and the vendor code is rejected? (**Platform supersedes vendor**)
   - Is an error displayed? (e.g., **"You cannot combine discount codes"**)
4. Note the exact discount lines shown in the basket summary and the running total at this point

---

### Step 22.3 — Customer Completes Checkout

**Actor**: Customer

**Steps**:
1. Proceed to payment — confirm the total on the payment screen matches the basket summary (no discrepancy at the payment step)
2. Complete payment using the Stripe test card `4242 4242 4242 4242`
3. Note the **Order ID** and the **actual total charged** from the Order Confirmation page

---

### Step 22.4 — Cross-Portal Verification

**Customer Front-end**:
- ☐ The basket summary shows **clear, itemised discount lines** — minimum requirement: subtotal, each discount on its own labelled line, delivery fee, and final total
- ☐ The final total on the Order Confirmation page exactly matches the basket summary total — no surprise change at the payment step
- ☐ The confirmation email shows the discounted total

**Vendor Admin**:
- ☐ The order in Vendor Admin shows the correct **post-discount total** — not the undiscounted subtotal
- ☐ Any applied promo code(s) are visible on the order card or order detail view

**System Admin**:
- ☐ The order record shows the **full financial breakdown**: undiscounted subtotal, each discount applied (labelled), delivery fee, platform commission basis, and net revenue
- ☐ The platform offer's **usage count** has incremented by 1 (verify in System Admin → Offers)
- ☐ The vendor promo code's **usage count** has incremented by 1 (verify in Vendor Admin → Offers)
- ☐ The **arithmetic is correct** — cross-check the actual total charged against your pre-calculated expected figures from Setup

**Document the stacking behaviour**:

> Complete this section regardless of which outcome occurred — it is a required part of the test record:
>
> | Field | Value |
> |-------|-------|
> | Stacking outcome observed | *(both applied / platform only / vendor only / error shown)* |
> | Platform discount applied | *(£ amount or N/A)* |
> | Vendor discount applied | *(£ amount or N/A)* |
> | Total charged to card | *(exact £ amount)* |
> | Discount lines shown in basket | *(list each line label and amount)* |
> | Any error or warning shown | *(exact text or N/A)* |

**Pass Criteria**:
- ✅ The checkout total arithmetic is correct, regardless of which stacking policy was applied
- ✅ Basket discount lines are clear and itemised — no ambiguous or missing line items
- ✅ Confirmation total matches the checkout total
- ✅ Both offer usage counts increment correctly in their respective portals

**Edge Cases**:
- Customer enters an **expired** vendor promo code alongside a valid platform offer → expired code should be rejected with a clear error; the platform offer should remain applied unaffected
- Both offers are percentage-based → confirm whether the second percentage is applied to the post-first-discount subtotal (compound) or the original subtotal (additive) — document the behaviour; do not assume either is correct or incorrect
- The vendor promo has a minimum order value that is only met if the platform offer is **not** yet deducted → document whether the minimum check is evaluated against the pre-discount or post-discount subtotal

---

## Scenario 23: Driver Shift Ends Mid-Delivery

**Summary**: A driver has accepted a delivery, physically collected the order from the vendor, and is en route to the customer. The driver then attempts to end their shift through the Driver Portal. This scenario determines whether the system prevents the shift-end whilst a delivery is active, permits it with a warning, or silently orphans the delivery. The outcome — whatever it is — must be fully documented. If the delivery is lost silently, the customer is left without status updates, or no refund path exists, that is a **Critical** bug and must be escalated immediately.

**Portals involved**: Driver Portal, Vendor Admin, Customer Front-end, System Admin

**Paired test cases**: TC-DP-021 (End Driver Shift)

---

### Setup

Before beginning:

- Confirm that the test driver can both start and end their shift in the Driver Portal
- Have all four portals open simultaneously — real-time co-ordination across portals is essential in this scenario
- **Recommended**: run as a two-person test — one tester on the Driver Portal, a second tester on Vendor Admin and Customer Front-end — to avoid missing status transitions whilst switching windows

---

### Step 23.1 — Order Placed, Accepted, Ready, and Driver Assigned

**Actor**: Customer, Vendor Admin

**Steps**:
1. As Customer: place a card order (Scenario 1, Step 1.1) — note the Order ID
2. As Vendor: accept the order, mark it as **Ready**, and assign the test driver (`driver.uat@wantfood.com`)
3. As Driver: confirm the delivery is visible in the **Active Deliveries** list in the Driver Portal

**Pass Criteria**:
- ✅ Delivery is active and visible in the Driver Portal
- ✅ Customer Front-end shows **"Driver on the way"** (or equivalent)

---

### Step 23.2 — Driver Picks Up the Order

**Actor**: Driver

**Steps**:
1. In the Driver Portal, open the active delivery
2. Mark the order as **"Picked Up"** or **"On My Way"** — confirming the driver has the food and is en route
3. Verify the status propagates:
   - Vendor Admin: order moves to **"Dispatched"** column
   - Customer Front-end: shows **"On the way"** (or equivalent)

**Pass Criteria**:
- ✅ Driver's "picked up" status is visible and consistent across all portals

---

### Step 23.3 — Driver Attempts to End Shift Whilst En Route

**Actor**: Driver

**Steps**:
1. Whilst the delivery is in the "picked up / en route" state and has **not** been delivered, navigate to the Driver Portal's **shift management** section
2. Attempt to click **"End Shift"** (TC-DP-021)
3. Observe and record the **exact response** — button state, modal, warning message, or state change
4. Refer to the outcome table below and identify which outcome occurred:

| Outcome | Platform behaviour | Severity |
|---------|-------------------|----------|
| **A — Shift end blocked** | Warning modal: *"You have an active delivery. Please complete it before ending your shift."* Driver remains on shift | Acceptable |
| **B — Shift end permitted with warning** | Warning is shown; driver can acknowledge and end shift; delivery remains active in the system; driver can still update delivery status | Acceptable if audit trail maintained |
| **C — Shift end permitted silently** | No warning; delivery status becomes ambiguous, stuck, or lost | **Bug — High/Critical** |
| **D — Auto-reassign triggered** | System attempts to reassign the delivery to another available driver; original driver is removed from the delivery | Document the full reassignment flow |

---

### Step 23.4 — Cross-Portal Verification

**Driver Portal**:
- ☐ (Outcome A) Driver receives a clear blocking warning — the active delivery is still shown and the driver remains on shift
- ☐ (Outcome B) Driver can end shift; the active delivery remains visible in the portal; the driver can still mark it as delivered
- ☐ (Outcome C) **Log a bug immediately** — severity **Critical**; describe the exact delivery state in all portals after the shift end
- ☐ (Outcome D) The original driver's portal no longer shows the delivery; verify whether a second driver has been notified of a new assignment

**Customer Front-end**:
- ☐ The customer's order status does **not** regress (e.g., must not revert from "On the way" to "Being Prepared" or "Cancelled")
- ☐ The customer does not receive a false cancellation notification
- ☐ The customer's tracking screen continues to show a driver name (even if it changes under Outcome D)

**Vendor Admin**:
- ☐ The order does not disappear from the kanban
- ☐ (Outcome D) The order card shows the new driver's name if a reassignment occurred
- ☐ The vendor is notified (if applicable) of any driver change

**System Admin**:
- ☐ The order record is intact — no data loss from the shift-end attempt
- ☐ Driver event log (if accessible) shows the shift-end attempt and its outcome
- ☐ (Outcome C) System Admin order shows a stuck status — escalate to engineering immediately

**Pass Criteria**:
- ✅ The delivery is not silently lost or abandoned
- ✅ The customer is not left without status updates or a clear path to resolution
- ✅ All portals remain consistent after the shift-end attempt
- ✅ The observed behaviour matches one of the four defined outcomes — any other behaviour is a bug

**Note for testers**: After completing this scenario, bring the delivery to a resolved state: either complete it (if the driver portal still allows), or raise a support ticket in the UAT environment to arrange manual resolution. Do not leave an orphaned delivery in the system at the end of the test session.

---

## Scenario 24: Customer Changes Default Address Mid-Checkout

**Summary**: A customer has dishes in their basket and is on the checkout screen viewing their delivery address and quote. In a second browser window, the same customer opens their Account settings and changes their default delivery address. This scenario verifies that the delivery quote is correctly re-evaluated when the customer actively changes the address on the checkout screen, and that the delivery address recorded against the final order precisely matches the one selected at the point of payment — no silent mismatch between the quoted address and the address used for delivery.

**Portals involved**: Customer Front-end (two browser windows)

**Paired test cases**: TC-FE-134 (Change Default Delivery Address)

---

### Setup

Before beginning:

- The test customer (`customer.uat@wantfood.com`) must have **at least two saved delivery addresses**:
  - **Address 1** — in-zone, currently set as the default — with a known delivery fee (e.g., £1.99)
  - **Address 2** — either further away within the same zone (different delivery tier) or in a different delivery zone — with a **different delivery fee** (e.g., £2.99)
- Open the Customer Front-end in **two separate browser windows** (not two tabs) — both share the same logged-in session:
  - **Window A** — the checkout flow
  - **Window B** — Account / Address settings

---

### Step 24.1 — Customer Begins Checkout in Window A

**Actor**: Customer (Window A)

**Steps**:
1. In Window A, add 2–3 dishes from the test vendor to the basket
2. Proceed to checkout and allow the delivery quote to generate against **Address 1** (the current default)
3. Note the **Address 1 delivery fee** and the **Address 1 total** shown on the checkout screen
4. **Pause on the address/quote screen — do not advance to payment**

**Pass Criteria**:
- ✅ Checkout is showing Address 1 with its delivery fee and total

---

### Step 24.2 — Customer Changes Default Address in Window B

**Actor**: Customer (Window B)

**Steps**:
1. In Window B, navigate to **My Account → Delivery Addresses** (or equivalent account settings page)
2. Change the **default address** from Address 1 to **Address 2** (TC-FE-134)
3. Save the change — confirm the success message
4. Take a screenshot showing Address 2 set as the new default in the account settings

**Pass Criteria**:
- ✅ Default address changed to Address 2 in Window B account settings

---

### Step 24.3 — Customer Returns to Checkout in Window A

**Actor**: Customer (Window A)

**Steps**:
1. Return to Window A (the checkout screen) — **do not refresh the page yet**
2. Observe whether the checkout screen has automatically updated to show Address 2 (the new default)
3. Using the **address selector on the checkout screen**, manually change the delivery address to **Address 2**
4. Observe carefully whether a **new delivery quote is generated** when you change the address selector — note the new delivery fee and the new total
5. Confirm the updated total reflects the Address 2 delivery cost, not the original Address 1 cost

**Expected**:
- Changing the address selector to Address 2 triggers a **re-evaluation of the delivery quote**
- The delivery fee and total update before proceeding to payment
- There is **no state** in which the customer is quoted the Address 1 delivery fee but receives delivery to Address 2, or vice versa

---

### Step 24.4 — Customer Completes Checkout

**Actor**: Customer (Window A)

**Steps**:
1. With Address 2 selected and the re-evaluated quote displayed, proceed to payment
2. Enter the Stripe test card `4242 4242 4242 4242`, any future expiry, any CVV
3. Click **"Place Order"**
4. Note the **Order ID** and the **total charged** from the Order Confirmation page
5. Confirm the confirmation page shows **Address 2** as the delivery address

---

### Step 24.5 — Cross-Portal Verification

**Customer Front-end**:
- ☐ The Order Confirmation page shows **Address 2** as the delivery address — not Address 1
- ☐ The **delivery fee** on the confirmation page matches the re-evaluated Address 2 delivery cost
- ☐ The **total charged** (confirmed via Stripe or order details) matches the Address 2 total displayed at the checkout step — no discrepancy between the quoted amount and the amount captured
- ☐ The confirmation email shows **Address 2** as the delivery address

**Vendor Admin**:
- ☐ The order card shows **Address 2** as the delivery address
- ☐ The delivery fee on the order card matches the expected Address 2 delivery cost

**System Admin**:
- ☐ The order record shows **Address 2** as the delivery address
- ☐ The delivery fee charged is the correct fee for Address 2

**Pass Criteria**:
- ✅ Quote re-evaluated when the delivery address is changed on the checkout screen
- ✅ Order confirmation address matches what the customer selected at payment time — not the old default
- ✅ Amount charged is consistent with the re-evaluated Address 2 quote

**Edge Cases**:
- Customer changes the checkout address to one that is **outside the vendor's delivery zone** → checkout should be blocked with a clear message such as **"We don't deliver to this address"**; no quote should be generated and no order should be placed
- Customer changes default in Window B and then **refreshes** Window A → refresh should load Address 2 as the pre-selected address and generate a fresh quote for it automatically
- Customer reaches the payment confirmation step (after the address step) and the backend re-validates the address at capture time — if the address has since become unserviceable, the payment should be blocked with a clear error rather than capturing and delivering to the wrong address

---

## Scenario 25: Rate Limiting and Duplicate Payment Submission

**Summary**: This scenario combines two related resilience tests. Part one extends Scenario 15's single-click duplicate-submit test (TC-FE-053): the customer rapidly clicks **"Pay now"** five times in quick succession, confirming that only one order and one payment are created regardless of how aggressively the button is clicked. Part two tests platform-level rate limiting by having the same customer account place ten separate, complete orders within a 60-second window — documenting whether rate limiting engages and, if so, at which attempt and with what message. If no rate limiting is present, that must be documented too — it is not automatically a bug, but it must be an explicit, recorded decision.

**Portals involved**: Customer Front-end, Vendor Admin, System Admin

**Paired test cases**: TC-FE-053 (Checkout Polling / Duplicate Submit Prevention)

---

### Setup

Before beginning:

- Have Vendor Admin and System Admin open in separate windows to observe order creation in real time during both parts of this test
- Have the Stripe test dashboard (test mode) open to monitor payment captures
- If running Part 2 (the ten-order rate-limit test), confirm with your project manager whether browser automation tools are permitted in the UAT environment — manual rapid ordering is sufficient but a colleague's assistance will speed this up considerably
- Prepare a simple log table (on paper or in a text file) to record each order attempt: attempt number, time, Order ID (if created), and outcome (success / blocked / error message)

---

### Step 25.1 — Part 1: Rapid Five-Click "Pay Now" Test

**Actor**: Customer

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Add dishes to the basket, proceed to checkout, enter a valid delivery address, and enter the Stripe test card `4242 4242 4242 4242`
3. With the cursor positioned over the **"Place Order"** button, **click it five times in rapid succession** — within approximately one second, with no deliberate pause between clicks
4. Note the **immediate button state** after the first click (disabled? loading spinner? nothing visible?)
5. Observe the resulting page and note the Order ID

**Expected**:
- The **"Place Order"** button becomes visibly disabled or shows a loading state immediately after the **first click** — subsequent clicks have no effect
- Only **one** order is created in Vendor Admin
- Only **one** payment is captured in Stripe
- The Order Confirmation page appears once, showing a single Order ID

---

### Step 25.2 — Cross-Portal Verification: Five-Click Test

**Customer Front-end**:
- ☐ The **"Place Order"** button is visibly disabled or in a loading state after the first click — the UI provides clear feedback that submission is in progress
- ☐ Exactly **one** Order Confirmation page is shown — not multiple conflicting states or error pages
- ☐ No error message is displayed unless the single submission itself failed

**Vendor Admin**:
- ☐ Exactly **one** new order appears in the **"Pending"** column for this submission — not five

**System Admin**:
- ☐ Exactly **one** order record for this submission attempt
- ☐ Exactly **one** Stripe payment capture recorded

**Stripe test dashboard**:
- ☐ Exactly **one** payment capture for this test card at this time — no duplicate charges

**Pass Criteria**:
- ✅ Only one order created from five rapid clicks
- ✅ Only one payment captured in Stripe
- ✅ Button state provides clear visual feedback after the first click

---

### Step 25.3 — Part 2: Ten Orders in Sixty Seconds (Rate-Limit Test)

> **Note**: This step requires placing **ten separate, complete orders** from the same customer account within a 60-second window. Each order is a full cycle: basket → checkout → payment → confirmation. This is a higher-effort test — co-ordinate with a colleague or use a minimal basket to speed up each cycle. Record each attempt accurately.

**Actor**: Customer (assisted by test team)

**Steps**:
1. Clear the basket from Part 1, then start a timer
2. Place a complete order as quickly as possible — note Attempt #1, time, and Order ID
3. Immediately repeat the full checkout cycle nine more times, recording for each attempt:
   - **Attempt number** (1–10)
   - **Time of submission**
   - **Order ID** (if an order was created)
   - **Outcome**: success (confirmation shown) / blocked (error message shown) / silent failure (no confirmation, no error)
   - **Exact error message** (if any)
4. After the 60-second mark, note whether any rate-limit window appears to reset (attempt an 11th order and record the outcome)

---

### Step 25.4 — Cross-Portal Verification: Rate-Limit Test

**Customer Front-end**:
- ☐ For any submission that is **rate-limited**: a clear message is displayed — for example, **"You are placing orders too quickly. Please wait a moment before placing another order."** There must be no silent blocking
- ☐ For any submission that **succeeds**: the full order confirmation flow completes normally
- ☐ No attempt results in a **silent failure** — every attempt must either produce a confirmation or a clear rejection message

**Vendor Admin**:
- ☐ Orders that succeeded appear in **"Pending"** with correct details
- ☐ No ghost orders or partial records appear for blocked or rate-limited attempts

**System Admin**:
- ☐ The number of order records matches exactly the number of successful placements (not the total of 10 attempts)
- ☐ If rate limiting is active: rate-limit events are logged or visible in the audit trail or event log

**Document the rate-limit behaviour** — complete this table as part of the test record:

> | Field | Value |
> |-------|-------|
> | Were any requests rate-limited? | *(Yes / No)* |
> | First attempt number where rate limiting engaged | *(e.g., attempt 4 — or "N/A")* |
> | Message shown to customer on rate-limited attempt | *(exact text — or "No message, silent block")* |
> | Did the limit reset after 60 seconds? | *(Yes / No / Not tested)* |
> | Total payment captures in Stripe for this test run | *(number)* |
> | Total orders created in System Admin | *(number)* |

**Pass Criteria**:
- ✅ No single submission results in duplicate orders or duplicate payment captures
- ✅ Every attempt (whether successful or blocked) gives the customer a clear outcome — no silent failures
- ✅ If rate limiting exists: the message is clear and human-readable, and the limit engages consistently

**Edge Cases**:
- One of the ten orders fails mid-flow due to a Stripe timeout → the remaining attempts should not be affected; each order is independent
- Customer hits the rate limit but has a legitimate order already in progress (awaiting driver, etc.) → the rate limit must not affect orders already accepted and being fulfilled
- Rate limit engages on attempt 2 (very aggressive threshold) → log the threshold value; it may be too restrictive for legitimate high-intent customers placing repeat orders for a group

---

## Summary: Saga Permutation Coverage Matrix

Use this matrix to track which scenarios have been tested and passed:

| Scenario | Card | Cash | Vendor Accept | Vendor Reject | Customer Cancel | Driver Assigned | Delivered | Failed | Refunded |
|----------|------|------|---------------|---------------|-----------------|-----------------|-----------|--------|----------|
| 1 — Happy Path (Card) | ✅ | — | ✅ | — | — | ✅ | ✅ | — | — |
| 2 — Happy Path (Cash) | — | ✅ | ✅ | — | — | ✅ | ✅ | — | — |
| 3 — Payment Declined | ✅ | — | — | — | — | — | — | — | — |
| 4 — Vendor Rejects | ✅ | — | — | ✅ | — | — | — | — | ✅ |
| 5 — Acceptance Timeout | ✅ | — | — | — | — | — | — | — | ✅ |
| 6 — Cancel Pre-Accept | ✅ | — | — | — | ✅ | — | — | — | ✅ |
| 7 — Cancel Post-Accept | ✅ | — | ✅ | — | ✅ | — | — | — | ✅ |
| 8 — No Driver | ✅ | — | ✅ | — | — | — | — | — | — |
| 9 — Driver Reassign | ✅ | — | ✅ | — | — | ✅ | — | — | — |
| 10 — Delivery Failed | ✅ | ✅ | ✅ | — | — | ✅ | — | ✅ | ✅ |
| 11 — Tip After Delivery | ✅ | — | ✅ | — | — | ✅ | ✅ | — | — |
| 12 — Vendor Cancel Refund | ✅ | — | ✅ | — | — | — | — | — | ✅ |
| 13 — Promo Code | ✅ | ✅ | ✅ | — | — | ✅ | ✅ | — | — |
| 14 — Quote Expiry | ✅ | — | — | — | — | — | — | — | — |
| 15 — Duplicate Submit | ✅ | — | — | — | — | — | — | — | — |
| 16 — Vendor Deactivated Mid-Flow | ✅ | — | ✅ | — | — | ✅ | ✅ | — | — |
| 17 — Menu Republished Mid-Checkout | ✅ | — | — | — | — | — | — | — | — |
| 18 — Vendor Edits In-Progress Order | ✅ | — | ✅ | — | — | — | — | — | — |
| 19 — Vendor Cancel Refund (Post-Capture) | ✅ | — | ✅ | — | — | — | — | — | ✅ |
| 20 — Stripe Webhook Delayed | ✅ | — | — | — | — | — | — | — | — |
| 21 — Outside Trading Hours | ✅ | — | — | — | — | — | — | — | — |
| 22 — Stacked Offers | ✅ | ✅ | ✅ | — | — | ✅ | ✅ | — | — |
| 23 — Driver Shift Ends Mid-Delivery | ✅ | — | ✅ | — | — | ✅ | — | — | — |
| 24 — Default Address Mid-Checkout | ✅ | — | ✅ | — | — | ✅ | ✅ | — | — |
| 25 — Rate Limiting / Duplicate Submit | ✅ | — | — | — | — | — | — | — | — |

---

## What to do next:

1. **Log all bugs** on your Monday.com board under the "Order Saga" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the final testing documents**:
	- **[Automation and Jobs UAT](07-automation-jobs-uat.md)**
	- **[Sign-off and Completion](08-signoff.md)**

---

**Great work completing the most complex part of UAT! 🚀**
