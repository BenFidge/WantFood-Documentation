# Resilience, Edge Cases, and Accessibility UAT — Test Scripts

## Purpose

This document contains test scripts for the **resilience, edge cases, and accessibility** aspects of the WantFood platform. Unlike the portal-specific scripts, these tests deliberately put the system under stress: slow connections, interrupted network calls, unusual payment flows, concurrent edits, and accessibility checks for users with disabilities.

These tests are important because real-world users do not always have perfect conditions — they lose their Wi-Fi, their bank challenges their card, or they use a screen reader. This document verifies that WantFood handles all of these situations gracefully.

**Target audience**: UAT testers who have already worked through the portal-specific scripts (02 through 07). You should be comfortable logging in to all four portals and using Chrome DevTools before starting this document.

---

## Prerequisites

Before starting, ensure you have:

- ✅ Completed or reviewed the portal-specific UAT scripts ([02](02-system-admin-uat.md) through [07](07-automation-jobs-uat.md))
- ✅ All four test accounts working (can log in to each portal)
- ✅ Google Chrome installed (required for network throttling tests)
- ✅ Stripe test card numbers ready (see [00-introduction.md](00-introduction.md#test-payment-details))
- ✅ Monday.com board set up and ready for logging bugs
- ✅ A second browser tab or window for cross-portal concurrent-edit tests
- ✅ NVDA (Windows) or VoiceOver (Mac) screen reader installed — see the [Quick Start guide](#quick-start-turning-on-a-screen-reader) in Section 6

---

## How to Open Chrome DevTools

Several tests in this document require Chrome DevTools. Here is how to open it:

1. Open Google Chrome
2. Press **F12** on your keyboard (or right-click anywhere on the page and choose **"Inspect"**)
3. The DevTools panel opens — usually at the bottom or right-hand side of the screen
4. Click the **"Network"** tab at the top of the DevTools panel

You will use the **Network** tab to simulate slow connections. Steps for each test will tell you exactly what to click.

---

## Table of Contents

1. [Network Resilience](#1-network-resilience)
2. [Service Failures and Error Pages](#2-service-failures-and-error-pages)
3. [Stripe and Payment Edge Cases](#3-stripe-and-payment-edge-cases)
4. [Concurrent Edits](#4-concurrent-edits)
5. [Locale, Currency, and Time Zone](#5-locale-currency-and-time-zone)
6. [Accessibility — Keyboard, Screen Reader, and Contrast](#6-accessibility--keyboard-screen-reader-and-contrast)
7. [Admin Portal Mobile Responsiveness](#7-admin-portal-mobile-responsiveness)
8. [Performance Budgets](#8-performance-budgets)
9. [Summary and Next Steps](#9-summary-and-next-steps)

---

## 1. Network Resilience

These tests simulate real-world network problems to check that the platform handles them gracefully rather than crashing or leaving the user stuck.

---

### TC-RE-001: Slow Connection During Customer Checkout

**Given**: You are logged in as a customer with items in your basket and are about to proceed to checkout

**Steps**:

1. Open the Customer Front-end in **Google Chrome**
2. Add at least one item to your basket
3. Press **F12** to open Chrome DevTools
4. Click the **"Network"** tab
5. Find the throttling dropdown — it usually says **"No throttling"** or shows a signal icon near the top of the Network tab. Click it.
6. Select **"Slow 3G"** from the dropdown list
7. Now navigate through checkout: enter your delivery address, wait for the quote, enter your payment details, and click **"Place Order"**
8. Observe how the page behaves whilst the connection is slow
9. When finished, open DevTools again and set throttling back to **"No throttling"**

**What "good" looks like**:
- Loading spinners or skeleton screens appear whilst the page waits for data — the user is never left staring at a blank or frozen page
- The Stripe card input loads eventually (it may take 5–10 seconds on Slow 3G — this is acceptable)
- The order can be placed successfully, even if it takes longer than usual
- No JavaScript errors appear in the Console tab of DevTools
- Progress is not lost — if the page takes a long time, the basket and address fields remain intact

**Pass Criteria**:
- ✅ Checkout completes successfully on Slow 3G (may take up to 30 seconds — this is acceptable)
- ✅ Loading indicators are shown whilst waiting for responses
- ✅ No data is lost from the basket or address fields
- ✅ No confusing error messages appear that could not be dismissed

**Edge Cases**:
- If the Stripe Payment Element shows a "Failed to load" message after 30+ seconds on Slow 3G, log this as a **Medium** bug — the payment element should retry or show a helpful retry button rather than a hard failure message

---

### TC-RE-002: Brief Offline / Wi-Fi Drop During Basket to Checkout

**Given**: You are logged in as a customer, have items in your basket, and are on the delivery address page

**Steps**:

1. Add items to your basket on the Customer Front-end in Chrome
2. Proceed to the checkout / address entry page
3. Enter your delivery address so a quote is generated
4. Press **F12** to open Chrome DevTools
5. Click the **"Network"** tab
6. Click the throttling dropdown and select **"Offline"** — this simulates your Wi-Fi dropping out
7. Wait 5 seconds (count to five)
8. Change the throttling back to **"No throttling"** (simulating the Wi-Fi reconnecting)
9. Try to proceed — click **"Continue to Payment"** or the equivalent next button

**What "good" looks like**:
- During the offline period, the page either shows an error banner ("No internet connection — please check your connection and try again") or simply waits
- When the connection is restored, the page recovers — either automatically reloading or showing a "Retry" button
- Your basket items and the delivery address are still present after reconnecting
- You can complete checkout without starting again from scratch

**Pass Criteria**:
- ✅ Basket items are preserved after the brief offline period
- ✅ The address entry is preserved after the brief offline period
- ✅ A clear offline or connection-error message is shown during the offline period
- ✅ Checkout can be completed after the connection is restored

**Edge Cases**:
- If the page shows a cryptic error code (e.g., "ERR_NETWORK_CHANGED" in the page itself rather than the browser bar), log this as a **Medium** bug — the error message should be friendly, not technical
- If the basket is cleared by the offline period, log this as a **High** bug

---

### TC-RE-003: Driver Portal Offline During Active Delivery

**Given**: You are logged in to the Driver Portal as a test driver with an active delivery in progress (an order has been assigned to you)

**Setup**: Complete enough of the Order Saga (see [06-order-saga-uat.md](06-order-saga-uat.md) Scenario 1 steps 1.1–1.4) so that a delivery is assigned to the test driver.

**Steps**:

1. Open the Driver Portal in Chrome and navigate to your active delivery
2. Confirm the **"Mark as Delivered"** button is visible
3. Press **F12** to open Chrome DevTools
4. Click the **"Network"** tab
5. Click the throttling dropdown and select **"Offline"**
6. Click **"Mark as Delivered"** whilst offline
7. Observe what happens
8. Switch throttling back to **"No throttling"**
9. Observe whether the action completes or whether you need to click again

**What "good" looks like**:
- Option A (preferred): The page queues the action and completes it once connectivity is restored — a brief "Reconnecting…" or "Saving…" indicator appears and then a success message follows
- Option B (acceptable): The page shows a clear error: "Unable to update — please check your connection and try again." The delivery status is **not** changed. Once back online, clicking the button again succeeds.
- Either way: the delivery is **not** marked as delivered without a successful server response

**Pass Criteria**:
- ✅ The action either queues for retry or shows a clear, friendly error message
- ✅ The delivery is only marked as delivered when the server has confirmed it
- ✅ The driver is not left in a broken state after reconnecting

**Edge Cases**:
- If the button is clicked offline and the delivery is silently marked as delivered without a server response, log this as a **High** bug — delivery state must be authoritative on the server

---

### TC-RE-004: Vendor Admin Offline During Order Acceptance

**Given**: You are logged in to the Vendor Admin portal and an incoming order appears in the "Pending" column of the Live Order Kanban

**Steps**:

1. Open Vendor Admin in Chrome
2. Navigate to the **Live Orders** kanban
3. Wait for or use a test order in the "Pending" column
4. Press **F12** to open Chrome DevTools → **Network** tab → set throttling to **"Offline"**
5. Click **"Accept"** on the pending order whilst offline
6. Observe the result
7. Set throttling back to **"No throttling"**
8. Observe whether the accept action retries or needs to be repeated

**What "good" looks like**:
- An error or warning appears: "Connection lost — your acceptance could not be saved. Please try again."
- The order does **not** move to "Accepted" until the server has confirmed it
- When the connection is restored, you can click "Accept" again and it succeeds

**Pass Criteria**:
- ✅ Order acceptance fails gracefully offline with a clear message
- ✅ Order state does not change until server confirmation is received
- ✅ Acceptance succeeds when connection is restored

**Edge Cases**:
- If the order appears to be accepted in the Vendor Admin UI but the Customer Front-end still shows "Awaiting Confirmation", log this as a **High** bug — a split-brain order state where one portal shows "Accepted" and another shows "Pending"

---

### TC-RE-005: Browser Back/Forward After a Failed Network Call

**Given**: You are using the Customer Front-end and you have just triggered a network error (use DevTools Offline throttle for a moment, then restore connection)

**Steps**:

1. On the Customer Front-end, navigate to a vendor page and begin adding items to the basket
2. Trigger a brief offline period (DevTools → Network → Offline for 3 seconds → restore)
3. If any error state appears on screen, use the browser's **Back** button (the left arrow at the top of Chrome)
4. Then use the **Forward** button to return
5. Attempt to continue with your original task (adding items, viewing the basket)

**What "good" looks like**:
- Pressing Back takes you to the previous page without a crash or blank screen
- Pressing Forward returns to the correct page — not a "page has expired" message
- The basket contents are still present
- No browser pop-ups asking "Do you want to resubmit the form?" appear when using Back/Forward through checkout

**Pass Criteria**:
- ✅ Back and Forward navigation works without crashes after a network error
- ✅ No "Confirm form resubmission" dialogs appear on normal Back/Forward navigation
- ✅ Basket is preserved through Back/Forward navigation

**Edge Cases**:
- If navigating Back after a failed payment attempt causes the payment to be retried automatically, log this as a **Critical** bug — duplicate charges must never occur

---

## 2. Service Failures and Error Pages

These tests check how the platform behaves when something goes wrong on the server side — a missing page, a back-end error, an expired session, or a payment provider outage.

---

### TC-RE-010: Customer Visits a Vendor URL That No Longer Exists (404)

**Given**: You are logged in (or not logged in) as a customer and you navigate to a URL for a vendor that does not exist

**Steps**:

1. On the Customer Front-end, navigate to a vendor page that exists (e.g., `/vendor/test-restaurant-1`)
2. Copy the URL but change the vendor slug to something that does not exist — for example, change `test-restaurant-1` to `this-vendor-does-not-exist-12345`
3. Press Enter to navigate to the non-existent URL
4. Observe the result

**What "good" looks like**:
- A friendly "Page Not Found" or "Restaurant Not Found" page appears — not a blank white page, not a raw error message, and not a page that shows the navigation broken
- The page includes a helpful message such as: "Sorry, we couldn't find that restaurant. It may have moved or been removed."
- The main navigation (home, search) is still accessible — you are not stuck
- A clear link or button to return to the home page is visible

**Pass Criteria**:
- ✅ A user-friendly 404-style page is displayed
- ✅ The error message is in plain English — no error codes, stack traces, or technical jargon visible to the customer
- ✅ Navigation back to the home page is available and works

**Edge Cases**:
- Visiting a 404 page whilst logged in — the navigation should still show your account/basket, not revert to a logged-out state
- Visiting the URL directly (pasting into the address bar) — result should be the same as navigating within the app

---

### TC-RE-011: Back-End Service Returns a 500 Error

**Given**: A back-end service is temporarily unavailable. You will simulate this by attempting an action that is known to fail in the UAT environment, or your test co-ordinator will arrange a deliberate service outage for this test.

> **Note for test co-ordinators**: This test requires a deliberate disruption. Ask the development team to temporarily stop one back-end service (e.g., the Vendor Service) in the UAT environment for 2–3 minutes, then restore it. Agree a time window for this test in advance.

**Steps**:

1. During the agreed service outage window, attempt to perform an action that uses the affected service:
   - If the Vendor Service is stopped: try to load a vendor's menu page on the Customer Front-end
   - If the Order Service is stopped: try to place an order
2. Observe the result
3. After the service is restored (your test co-ordinator will confirm), try the same action again

**What "good" looks like**:
- A friendly generic error page or banner appears: "Something went wrong — we're sorry for the inconvenience. Please try again in a moment."
- The message does **not** show technical details such as "500 Internal Server Error", exception stack traces, or database connection strings
- A "Try again" or "Go home" button is visible
- After the service is restored, the action succeeds on retry without needing to clear the browser cache

**Pass Criteria**:
- ✅ A user-friendly error page or banner is shown for a 5xx error
- ✅ No technical error details are exposed to the end user
- ✅ Navigation away from the error state is possible
- ✅ The action succeeds once the service is restored

**Edge Cases**:
- If the error page itself fails to load (a 500 on the error page), log this as a **Critical** bug
- Partial service failure (some sections of a page load, others do not) — each broken section should show an inline error rather than breaking the whole page layout

---

### TC-RE-012: Customer Session Signed Out by an Admin — Graceful Re-authentication

**Given**: You have the Customer Front-end open in one browser and the System Admin portal open in another. You are logged in as a customer.

**Steps**:

1. In the System Admin portal, navigate to **Vendor Management** → find the test vendor and open **"Vendor Users"** (or navigate to **Admin User Management** if testing with an admin account)
2. Find the customer test account (`customer.uat@wantfood.com`) — if it is listed as a vendor user, use that; otherwise, find the appropriate user account
3. Use the admin action to **"Revoke Session"**, **"Sign Out User"**, or **"Delete User"** (use the least destructive option available that will end their active session — check with your test co-ordinator for the correct action in your UAT environment)
4. Switch back to the browser where you are logged in as the customer
5. Attempt to perform an authenticated action: add an item to the basket, proceed to checkout, or view order history

**What "good" looks like**:
- The customer is redirected to the login page with a message such as: "Your session has expired. Please sign in again."
- The basket contents are preserved in local storage so the customer can recover their order after logging back in (this is the ideal behaviour — if basket is lost, log as a **Low** bug)
- After logging back in, the customer can complete their task normally
- They are **not** shown a confusing JavaScript error or a blank page

**Pass Criteria**:
- ✅ The customer is redirected to the login page after their session is terminated
- ✅ A clear "session expired" message is shown
- ✅ After re-authentication, the customer can continue using the platform
- ✅ No sensitive data from the previous session is accessible after sign-out

**Edge Cases**:
- If the customer's session cookie is forcibly expired but they have a checkout in progress, they should be redirected to login and then back to their checkout (or to the basket) after re-authentication — not dropped at the home page. If this does not work, log as a **Medium** bug.

---

### TC-RE-013: Payment Provider Intermittently Failing — Retry UX

**Given**: You are on the checkout page with a valid basket and delivery address, about to pay

> **Note**: For this test, use the Stripe test card `4000 0000 0000 0002` (generic decline) on the first attempt, then switch to `4242 4242 4242 4242` (success) on the second attempt. This simulates a failed first attempt followed by a successful retry — which is the most common real-world equivalent of a payment provider blip.

**Steps**:

1. Proceed to checkout on the Customer Front-end
2. Enter the **declined** test card: `4000 0000 0000 0002`, any future expiry, any CVV
3. Click **"Place Order"** and observe the error message
4. **Without** refreshing the page or navigating away, update the card number to the **success** card: `4242 4242 4242 4242`
5. Click **"Place Order"** again
6. Observe whether the order succeeds on the second attempt

**What "good" looks like**:
- After the first (declined) attempt, a clear error message appears: "Your payment was declined. Please check your card details and try again." or similar
- The checkout page remains intact — your basket, address, and other fields have not been cleared
- You can update the card number and try again without navigating away
- The second attempt (with the valid card) succeeds and takes you to the Order Confirmation page
- Only **one** order is created — the failed attempt does not generate a pending order in the Vendor Admin

**Pass Criteria**:
- ✅ First declined attempt shows a clear, friendly error message
- ✅ Checkout page remains intact after a decline so the customer can retry
- ✅ Second attempt with a valid card succeeds
- ✅ Only one order is created (no phantom orders from the failed attempt — verify in Vendor Admin)

**Edge Cases**:
- Three or more consecutive declines — the checkout should remain functional throughout without locking the user out. Log any lock-out as a **High** bug.
- If the loading spinner stays permanently after a decline without an error message appearing, log as a **High** bug — the customer is left with no indication of what happened

---

## 3. Stripe and Payment Edge Cases

These tests cover unusual but realistic Stripe payment scenarios. They use specific Stripe test cards — you do not need to remember these; they are listed in each test case.

---

### TC-RE-020: Webhook Delayed More Than 30 Seconds After Card Capture

**Given**: You are about to place an order using the standard success card

> **Background for testers**: When you pay by card, WantFood sends your card details to Stripe, which processes the payment. Stripe then sends a "webhook" (a background notification) to WantFood to confirm the payment went through. Usually this takes 1–2 seconds. This test simulates what happens if that notification is delayed.

> **Note for test co-ordinators**: To properly test this scenario, ask the development team to temporarily delay or block the Stripe webhook endpoint in the UAT environment. Alternatively, test the behaviour by observing what the confirmation page shows during the normal processing window — the test below describes the expected polling behaviour.

**Steps**:

1. On the Customer Front-end, proceed to checkout with items in your basket
2. Enter the success test card: `4242 4242 4242 4242`, any future expiry, any CVV
3. Click **"Place Order"**
4. Immediately after clicking, observe the page that appears before you reach the Order Confirmation page
5. Watch the page for at least 10 seconds without doing anything — observe whether it polls / updates automatically
6. If the co-ordinator has delayed the webhook: observe the page for up to 60 seconds

**What "good" looks like**:
- A loading or "processing" page appears immediately after clicking "Place Order" — something like "Processing your payment…" or "Confirming your order…"
- The page shows a spinner or progress indicator — the customer is never shown a blank page
- The page automatically updates (polls) every few seconds — you should see it "check" for confirmation without needing to refresh
- If the confirmation arrives within 30 seconds: the page transitions smoothly to the Order Confirmation page
- If the confirmation is delayed beyond 30 seconds: the page shows a holding message such as: "Your payment has been received — we're just confirming your order. This is taking a little longer than usual. You will receive a confirmation email shortly."
- The customer is **not** shown a false failure message if the webhook is simply delayed

**Pass Criteria**:
- ✅ A loading/processing page is shown after "Place Order" is clicked
- ✅ The page polls automatically — it does not require a manual refresh
- ✅ If delayed, a holding message reassures the customer rather than suggesting failure
- ✅ The order confirmation email is sent once the webhook eventually arrives

**Edge Cases**:
- If the webhook never arrives (permanent failure): the order should eventually time out and the customer should be directed to contact support. The payment should be refunded if the order was never created. Log any scenario where the customer's card is charged but no order confirmation is ever shown as **Critical**.

---

### TC-RE-021: Payment Intent Stuck in "Processing" State

**Given**: You are observing the post-payment confirmation page (the loading/polling page from TC-RE-020)

> **Background**: Occasionally a Stripe payment intent can get stuck in a "processing" state — this is different from a delayed webhook. The payment itself has not yet resolved to "succeeded" or "failed". This is rare but worth validating.

**Steps**:

1. After placing an order, observe the confirmation loading/polling page
2. Leave it open for **3 minutes** without interacting with it
3. Observe whether the page times out gracefully

**What "good" looks like**:
- After 2–3 minutes with no resolution, the page shows a clear message: "We're having trouble confirming your payment. Your card has not been charged — please try again, or contact support if this continues."
- A "Try again" button or link to contact support is visible
- The page does **not** spin indefinitely with no user-facing timeout

**Pass Criteria**:
- ✅ A timeout message appears after approximately 2–3 minutes
- ✅ The message clearly states whether the card has been charged
- ✅ A way to contact support or retry is shown

> **Declaring this test as a fail**: If after 3 minutes the page is still showing a loading spinner with no message, log this as a **High** bug — indefinite spinners are not acceptable as the customer has no way to know what has happened to their money.

**Edge Cases**:
- If the test environment does not easily produce a stuck intent, you can declare this test "Not testable in UAT" and note it for smoke testing in staging — but do log an observation if the timeout experience cannot be reached at all

---

### TC-RE-022: 3D Secure — Tester Closes the Challenge Without Completing

**Given**: You are at checkout with a valid basket and delivery address

**Steps**:

1. Enter the 3D Secure test card: `4000 0027 6000 3184` (this card always requires 3DS authentication)
   - Expiry: any future date (e.g., 12/29)
   - CVV: any 3 digits (e.g., 123)
2. Click **"Place Order"**
3. Wait for the **3D Secure challenge popup or iframe** to appear — this is a small Stripe window that asks you to authenticate with your bank
4. Instead of completing the challenge, **close the popup** by clicking the X, pressing Escape, or clicking outside it
5. Observe what happens to the checkout page

**What "good" looks like**:
- The checkout page remains intact — you are not stuck on a processing screen
- A message appears: "Authentication was cancelled. Your card has not been charged. Please try again."
- The customer can attempt payment again with the same card or a different one
- No order is created in the Vendor Admin (verify cross-portal)

**Pass Criteria**:
- ✅ Closing the 3DS popup without authenticating shows a clear, friendly cancellation message
- ✅ Checkout page remains usable for retry
- ✅ Card is **not** charged
- ✅ No order is created (verify in Vendor Admin)

**Edge Cases**:
- Closing the 3DS popup and then immediately trying again — the second attempt should open a fresh 3DS challenge without issues

---

### TC-RE-023: 3D Secure Authentication Failed

**Given**: You are at checkout with a valid basket and delivery address

**Steps**:

1. Enter the 3DS authentication **failure** test card: `4000 0027 6000 3184`
   - Expiry: any future date
   - CVV: any 3 digits
2. Click **"Place Order"**
3. When the **3D Secure challenge** appears, look for a **"Fail authentication"** option (in Stripe test mode, the 3DS popup offers both "Authenticate" and "Fail authentication" buttons — click **"Fail authentication"**)
4. Observe what happens

**What "good" looks like**:
- The checkout page shows a clear error: "Your card could not be authenticated. Please try a different payment method or contact your bank."
- The page remains intact for retry
- No order is created
- The error message does not mention technical terms like "3DS", "authentication error code", or "payment intent failed"

**Pass Criteria**:
- ✅ Failed 3DS authentication shows a friendly, clear error message
- ✅ Checkout page remains usable for retry
- ✅ Card is not charged
- ✅ No order created (verify in Vendor Admin)

**Edge Cases**:
- Failing authentication and then using the standard success card (`4242 4242 4242 4242`) on the retry — the order should succeed normally

---

### TC-RE-024: Insufficient Funds Card

**Given**: You are at checkout with a valid basket and delivery address

**Steps**:

1. Enter the **insufficient funds** test card: `4000 0000 0000 9995`
   - Expiry: any future date
   - CVV: any 3 digits
2. Click **"Place Order"**
3. Observe the error message

**What "good" looks like**:
- Payment is declined with a specific, helpful message such as: "Your card was declined due to insufficient funds. Please try a different card."
- The message mentions "insufficient funds" (or similar plain English) rather than a generic "card declined" message — customers benefit from knowing the specific reason if Stripe provides it
- Checkout page remains intact for retry
- No order is created

**Pass Criteria**:
- ✅ Payment is declined
- ✅ Error message is clear — ideally indicates the reason (insufficient funds)
- ✅ Checkout page remains usable for retry
- ✅ No order created (verify in Vendor Admin)

**Edge Cases**:
- Using an insufficient funds card on the retry after a previous decline — the system should handle multiple declines gracefully

---

### TC-RE-025: Network Drop During a 3DS Challenge

**Given**: You are mid-way through a 3D Secure authentication challenge

**Steps**:

1. Enter the 3DS test card: `4000 0027 6000 3184`, any future expiry, any CVV
2. Click **"Place Order"**
3. Wait for the **3D Secure challenge popup** to appear
4. Once the popup is open, press **F12** → **Network** tab → set throttling to **"Offline"**
5. Click **"Authenticate"** (the success option) within the Stripe 3DS popup whilst offline
6. Wait 5 seconds, then restore connectivity (set throttling back to "No throttling")
7. Observe what happens to the checkout page

**What "good" looks like**:
- Option A: The authentication queues and completes once connectivity is restored — the checkout proceeds normally to the Order Confirmation page
- Option B: The checkout page shows an error: "Authentication could not be completed — please check your connection and try again." The page remains usable, and a fresh attempt (clicking "Place Order" again with the same or different card) can succeed.
- Either way: the customer is **not** charged without the authentication completing successfully, and they are not left staring at a blank or broken page

**Pass Criteria**:
- ✅ A clear outcome occurs — either success after reconnection or a clear retry error
- ✅ Customer is not charged without a completed, confirmed authentication
- ✅ Checkout page remains usable after reconnecting

**Edge Cases**:
- If the popup closes on its own when going offline, ensure the checkout page correctly detects this and shows a retry option

---

## 4. Concurrent Edits

These tests check what happens when two people try to edit the same data at the same time. This is a realistic scenario in a busy restaurant: two managers might both be editing the menu simultaneously.

---

### TC-RE-030: Two Vendor Admin Users Editing the Same Dish Simultaneously

**Given**: Two browser sessions are open, both logged in to the Vendor Admin portal as vendor admin users for the same restaurant

> **Setup**: Use two different browser profiles in Chrome (Chrome Profile 1 and Chrome Profile 2), or use Chrome and an incognito window. Both sessions must be logged in to the same Vendor Admin account (or two different accounts with access to the same vendor).

**Steps**:

1. In **both** browser windows, navigate to **Menu Management** → open the same test dish (e.g., "Test Dish 1")
2. In **Browser 1**: change the dish name to "Test Dish 1 — Updated by User A" and change the price to £9.99. **Do not save yet.**
3. In **Browser 2**: change the dish name to "Test Dish 1 — Updated by User B" and change the price to £11.99. **Do not save yet.**
4. In **Browser 1**: click **"Save"**
5. In **Browser 2**: click **"Save"** (approximately 5–10 seconds after Browser 1)
6. Observe the result in both browsers and then navigate away and back to the dish to check the final saved state

**What "good" looks like**:
- **Ideal behaviour — conflict warning**: Browser 2 shows a warning such as: "This dish was updated by another user while you were editing. Your changes would overwrite theirs. Please review the latest version before saving." This gives the second editor a chance to reconcile.
- **Acceptable behaviour — last writer wins**: Browser 2's save overwrites Browser 1's changes without a warning. The final dish shows Browser 2's values. This is acceptable, but a conflict warning is preferable — note in your test log which behaviour you observed.
- **Unacceptable behaviour**: Data corruption (e.g., a price of £0, a blank name, or a server error on save). Log any data corruption as a **High** bug.

**Pass Criteria**:
- ✅ Both saves complete without server errors
- ✅ The final dish state is consistent — it reflects either Browser 1 or Browser 2's values, not a corrupted mix
- ✅ No data is lost or zeroed out

**Edge Cases**:
- If the edit form is opened by the second user whilst the first is saving, there should be no JavaScript crash in either session

---

### TC-RE-031: Vendor Admin Edits Menu While a Customer Is in Checkout Against That Menu

**Given**: A customer has a basket with items from a vendor's menu, and is mid-checkout (on the address or payment page). Simultaneously, the Vendor Admin edits one of those dishes — changing the price.

**Steps**:

1. As the **customer**: add dishes from "Test Restaurant 1" to your basket and proceed to the checkout address page. **Stay on this page — do not submit yet.**
2. As the **vendor** (in a second browser window): navigate to Menu Management → open one of the dishes currently in the customer's basket → change the price by £1–2 (e.g., from £8.00 to £9.00) → click **"Save"**
3. As the **customer**: complete the checkout — enter payment details and click **"Place Order"**
4. Observe the order confirmation — which price is shown? The old price (from when the basket was built) or the new price (after the vendor's update)?

**What "good" looks like**:
- The order is charged at the **price that was in the basket when the customer checked out** — not the new price the vendor just set. This protects the customer from being charged more than they agreed to.
- Alternatively, the system re-prices the basket when the order is submitted and shows the customer an updated total with a clear message before charging them — the customer must not be silently charged a different amount.
- The order confirmation clearly shows the per-item prices that were charged.

**Pass Criteria**:
- ✅ The customer is charged the amount they were shown when they agreed to pay
- ✅ No silent repricing occurs — any price change is shown to the customer before payment
- ✅ Order Confirmation page shows line-item prices that match the actual charges

**Edge Cases**:
- If the vendor deletes the dish entirely whilst the customer is checking out, the order submission should either succeed (using the last known price) or show a clear error explaining that an item is no longer available

---

### TC-RE-032: System Admin Edits Commission Tier During Invoice Generation

**Given**: The UAT environment has automated invoice generation running (or the test co-ordinator can manually trigger an invoice run). A commission tier is being edited in System Admin at the same time.

> **Note**: This scenario is difficult to force precisely in UAT. The goal is to understand the expected behaviour and verify the system's behaviour is consistent. Do not spend more than 15 minutes attempting to reproduce the exact race condition — instead, test the closest approximation described below and note your findings.

**Steps**:

1. In System Admin, navigate to **Commissions** and note the current commission rate for a tier (e.g., the standard tier is 15%)
2. Begin editing the commission tier — change the rate to a different value (e.g., 16%) but **do not save yet**
3. Ask your test co-ordinator to trigger an invoice generation run (or observe an automated run if one is scheduled)
4. After the invoice run starts (your co-ordinator will confirm), save your commission tier change
5. Once the invoice run completes, check the generated invoices — do they use the old (15%) or new (16%) rate?

**What "good" looks like**:
- The invoices generated during the run use the commission rate that was **active when the run started** (15% in this example) — not the new rate that was saved mid-run. Commission rates should be snapshotted at the start of an invoice run to ensure consistent results.
- Alternatively, if the system prevents saving a commission rate while an invoice run is in progress, this is equally acceptable — a message such as "Invoice generation is in progress. Please wait until it completes before changing commission rates." is good behaviour.
- There is **no scenario** in which some invoices in the same run use 15% and others use 16% — split-run commission rates are a billing error.

**Pass Criteria**:
- ✅ All invoices in a single run use the same commission rate
- ✅ Either the rate is snapshotted at run start, or edits are blocked during a run
- ✅ No invoices are generated with £0 commission or an obviously wrong rate

**Edge Cases**:
- If the commission change is applied immediately and retroactively to already-generated invoices in the current run, log this as a **High** bug

---

## 5. Locale, Currency, and Time Zone

These tests check that monetary values, times, and dates are displayed correctly and consistently across the entire WantFood platform.

---

### TC-RE-040: Currency Displays as GBP (£) Everywhere

**Given**: You have access to all four portals and there is at least one completed order with a known total in the test environment

**Steps**:

Check each of the following locations and confirm the currency symbol and formatting are correct:

1. **Customer Front-end — Basket**: Add items to your basket. Confirm each item price shows **£** (e.g., "£8.50") and the basket total shows **£** (e.g., "Total: £17.00")
2. **Customer Front-end — Order Confirmation**: Place a test order. Confirm the confirmation page shows the total in **£**
3. **Customer Front-end — Order History**: Navigate to your order history. Confirm past order totals show **£**
4. **Vendor Admin — Live Orders**: Open the Live Orders kanban. Confirm order totals on order cards show **£**
5. **Vendor Admin — Earnings / Reports** (if available): Confirm any totals or revenue figures show **£**
6. **System Admin — Orders**: Open the Orders list. Confirm order totals show **£**
7. **System Admin — Invoices**: Open a generated invoice. Confirm all line items and totals show **£**
8. **Driver Portal — Earnings** (if earnings are displayed): Confirm any driver earnings figures show **£**
9. **Confirmation emails**: If you receive an order confirmation email, confirm the amounts show **£**

**What "good" looks like**:
- Every monetary value across every portal shows the pound sign (**£**) followed by a number with two decimal places (e.g., £12.50, not $12.50, not 12.50, not £12.5)
- Large amounts use comma separators (e.g., £1,234.56, not £1234.56)

**Pass Criteria**:
- ✅ Every monetary value tested uses £ — no dollar signs ($), euro signs (€), or unformatted numbers
- ✅ All amounts show two decimal places
- ✅ Amounts over £999 use a comma separator

**Edge Cases**:
- If any value shows £0.00 where a non-zero amount is expected, log this as a **High** bug (possible currency conversion or missing data issue)
- If the currency symbol is missing entirely (e.g., showing "8.50" rather than "£8.50"), log as a **Medium** bug

---

### TC-RE-041: 24-Hour vs 12-Hour Time Format Consistency

**Given**: There are orders and scheduled times visible in the UAT environment

**Steps**:

Check the following locations and note the time format used:

1. **Customer Front-end — Order Tracking**: After placing an order, note the estimated delivery time shown (e.g., "Estimated delivery: 7:30 PM" or "19:30")
2. **Vendor Admin — Live Orders**: Note the order time stamps on order cards (e.g., "Placed at 14:32" or "2:32 PM")
3. **Vendor Admin — Order history / reports**: Note any time stamps in historical views
4. **System Admin — Orders list**: Note the order time stamps in the orders table
5. **Driver Portal — Delivery list**: Note the collection / delivery time shown

**What "good" looks like**:
- All portals use the **same** time format throughout — either all 24-hour (e.g., 14:32) or all 12-hour (e.g., 2:32 PM)
- Mixing formats (e.g., the confirmation page shows "2:32 PM" but the Vendor Admin shows "14:32") is acceptable only if each portal consistently uses one format — log any **within-portal** inconsistency
- "AM"/"PM" labels are present on all 12-hour times — a time showing "2:32" with no AM/PM indicator is ambiguous and should be logged as a **Low** bug

**Pass Criteria**:
- ✅ Time format is consistent within each portal
- ✅ No ambiguous times (12-hour format without AM/PM)

**Edge Cases**:
- Midnight and noon: confirm these display as "12:00 AM" / "12:00 PM" (or "00:00" / "12:00" in 24-hour) — not "0:00 AM" or "12:00 AM" and "12:00 AM" for both

---

### TC-RE-042: BST/GMT (Daylight Saving Time) Crossover

**Given**: The UAT environment uses UK time zones

> **Background**: The UK changes its clocks twice a year — forward one hour in spring (GMT → BST, British Summer Time) and back one hour in autumn (BST → GMT). Dates and times stored as UTC in the database should display correctly in the customer's local time.

> **Note**: This test only applies fully if your UAT session happens to be close to a clock-change date. If not, test using the closest approximation described below.

**Steps**:

**If testing near a clock-change date**:

1. Schedule or look for an order in the test environment that was placed or is expected to be delivered near the clock-change time (e.g., between 1:00 AM and 3:00 AM on the relevant Sunday)
2. Check the Order Confirmation page, Vendor Admin order card, and System Admin order details — confirm all portals show the same local time for the order

**If testing away from a clock-change date (approximation)**:

1. In System Admin, look for orders placed at times that would have a different UTC offset — for example, an order placed in winter (GMT, UTC+0) and an order placed in summer (BST, UTC+1)
2. Confirm that summer orders show times that are 1 hour ahead of their UTC equivalent, and winter orders show times equal to UTC

**What "good" looks like**:
- Order times are displayed in UK local time (adjusting for BST/GMT automatically) — not always in UTC
- A summer order placed at 13:00 UTC shows as "14:00" (BST) in the portals, not "13:00"
- A winter order placed at 13:00 UTC shows as "13:00" (GMT) — as expected
- No orders show an impossible time such as 1:30 AM on the night of the autumn clock change (when the clocks go back, 1:30 AM happens twice — the system should handle this without showing duplicated entries)

**Pass Criteria**:
- ✅ Order times are displayed in UK local time, adjusted for DST
- ✅ No impossible or duplicate times appear around the clock-change window

**Edge Cases**:
- If the UAT environment stores all times in UTC and displays them as UTC without conversion, log this as a **Medium** bug — customers will see confusing times during BST

---

### TC-RE-043: Customer Browser Locale Set to Non-English (French or Spanish)

**Given**: You can change your browser's language settings

> **Background**: WantFood is currently designed for UK customers and may not have full localisation (translation into other languages). However, the platform should not **break** when a customer's browser reports a non-English locale — pages should remain readable, and no JavaScript errors should occur.

**Steps**:

1. In Chrome, go to **Settings** → **Languages**
2. Add **French (France)** (or Spanish) as your primary language
3. Restart Chrome
4. Navigate to the Customer Front-end UAT URL
5. Browse through several pages: home page, a vendor's menu, the basket, the checkout address page
6. Note what language the platform displays — is it in English, French, or something in between?
7. Check for any broken layouts, missing text, or JavaScript errors (open DevTools → Console tab)
8. When finished, restore your browser language to **English (United Kingdom)** and restart Chrome

**What "good" looks like**:
- The platform continues to display in **English** (because it does not have French/Spanish localisation yet) — this is perfectly acceptable
- Pages do not break, go blank, or display garbled text simply because the browser locale is French or Spanish
- No JavaScript errors appear in the Console tab
- Form validation still works correctly (required field errors, email format errors, etc.)

**Pass Criteria**:
- ✅ All pages load and render correctly with a non-English browser locale
- ✅ No JavaScript errors in the browser console caused by the locale change
- ✅ Form validation continues to work

**Edge Cases**:
- If dates or numbers are formatted differently with a French locale (e.g., £8,50 instead of £8.50 — using a comma as the decimal separator), log this as a **Medium** bug — currency formatting must use UK conventions regardless of browser locale

---

## 6. Accessibility — Keyboard, Screen Reader, and Contrast

Accessibility testing ensures that WantFood can be used by people with disabilities — including those who cannot use a mouse, those who are blind or have low vision, and those with colour-perception differences.

> **You do not need to be an accessibility expert to complete these tests.** Follow the steps precisely and record what you observe. If something surprises you or feels wrong, log it — the development team will classify the severity.

---

### Quick Start: Turning On a Screen Reader

A screen reader is software that reads out everything on screen using a synthesised voice. It is the primary way blind and visually impaired people use computers.

**On Windows — NVDA (free)**:

1. Download NVDA from [https://www.nvaccess.org/download/](https://www.nvaccess.org/download/) if not already installed
2. Run the installer and follow the setup wizard
3. To **start** NVDA: find it in the Start menu or press the desktop shortcut. You will hear "NVDA started" and a voice will begin reading the screen.
4. To **stop** NVDA: press **Insert + Q**, then confirm
5. **Key controls during testing**: use **Tab** to move between interactive elements, **Enter** to activate a link or button, and **Arrow keys** to navigate text. NVDA will read each element aloud as you reach it.

**On macOS — VoiceOver (built-in)**:

1. Press **Command + F5** to toggle VoiceOver on or off. You will hear "VoiceOver on" or "VoiceOver off".
2. Alternatively, go to **Apple Menu → System Preferences → Accessibility → VoiceOver** and tick "Enable VoiceOver"
3. **Key controls during testing**: use **Tab** to move between interactive elements, **Control + Option + Right Arrow** to read the next item, and **Control + Option + Space** to activate a button or link

> **Tip**: Screen readers can be quite verbose — they read everything. Do not worry if the voice says more than you expect. Focus on whether the correct information is being read, not on the speed or amount of speech.

---

### TC-RE-050: Keyboard-Only Navigation — System Admin Main Nav and Vendor Edit Form

**Given**: You are logged in to the System Admin portal. You will use **only** the keyboard — do not touch the mouse for this entire test.

**Steps**:

1. After logging in, press **Tab** on your keyboard. Each press should move focus to the next interactive element (link, button, or input field)
2. Press **Tab** repeatedly to move through the main navigation menu items (Vendors, Applications, Orders, Commissions, etc.)
3. Confirm each navigation item receives a **visible focus indicator** — a visible border, outline, or highlight around the focused item. You should always be able to see where you are.
4. Use the **Arrow keys** (or Tab) to expand any dropdown menu items in the navigation
5. Press **Enter** to navigate to the **Vendors** section
6. Tab to a vendor in the list and press **Enter** to open the vendor edit form
7. Tab through all the fields in the vendor edit form (Name, Address, Phone Number, etc.)
8. Confirm each field can be reached and edited using only the keyboard
9. Tab to the **"Save"** button and press **Enter** to submit. Confirm the form saves.
10. Tab to the **"Cancel"** button or equivalent and press **Enter**. Confirm it navigates away without saving.

**What "good" looks like**:
- Every interactive element (nav item, button, form field, link) can be reached using Tab alone
- A clear, visible focus indicator (a coloured border or highlight) is always visible so you know where you are
- The Tab order is logical — it moves from left to right and top to bottom, not jumping randomly around the page
- Keyboard shortcuts work (e.g., Enter on a button triggers it)
- You are never "trapped" in a widget — you can always Tab out of any element

**Pass Criteria**:
- ✅ All navigation items are reachable by Tab
- ✅ All form fields are reachable and editable by keyboard alone
- ✅ Focus indicator is visible on every interactive element
- ✅ Form can be saved and cancelled using only the keyboard

**Edge Cases**:
- If focus disappears (you press Tab but cannot see where focus has gone), log as a **High** bug
- If a dropdown menu in the navigation closes when you Tab into it (meaning you cannot reach the items inside), log as a **High** bug

---

### TC-RE-051: Keyboard-Only Navigation — Customer Checkout (Basket to Confirmation)

**Given**: You are on the Customer Front-end with items in your basket. You will use **only** the keyboard for the entire checkout flow.

**Steps**:

1. Open the basket (Tab to the basket icon and press Enter, or navigate to the basket page)
2. Tab through the basket items — confirm you can reach the "Remove" and quantity buttons
3. Tab to the **"Proceed to Checkout"** button and press Enter
4. On the checkout / address page, Tab through all address fields and fill them in using the keyboard
5. Tab to the **"Continue"** or **"Get Quote"** button and press Enter
6. On the payment page, Tab to the Stripe card input fields and enter card details
   - Note: The Stripe card input is an iframe. You may need to Tab into the iframe and then Tab through the card number, expiry, and CVV fields within it
7. Tab to the **"Place Order"** button and press Enter
8. Confirm the Order Confirmation page is reached

**What "good" looks like**:
- The entire checkout flow from basket to confirmation can be completed using only the keyboard
- Focus is always visible throughout
- The Stripe payment element is reachable and its fields can be navigated using Tab within the iframe
- After placing the order, focus moves logically to the Order Confirmation page content

**Pass Criteria**:
- ✅ All checkout steps completable with keyboard only
- ✅ Focus indicator visible at every step
- ✅ Stripe payment fields reachable via Tab

**Edge Cases**:
- If focus is lost after clicking "Place Order" (e.g., it jumps to the top of the page or disappears entirely), log as a **Medium** bug

---

### TC-RE-052: Screen Reader Landmarks — System Admin and Customer Front-end

**Given**: You have NVDA or VoiceOver running (see the [Quick Start guide](#quick-start-turning-on-a-screen-reader) above)

**Steps — System Admin**:

1. With your screen reader running, navigate to the System Admin dashboard
2. Use NVDA's landmark navigation shortcut: press **D** (in NVDA's Browse mode) to jump between landmark regions, or use VoiceOver's **Control + Option + U** to open the Rotor and look for Landmarks
3. Confirm the following landmarks are announced by the screen reader:
   - A **banner** or **header** region (usually containing the logo and top navigation)
   - A **navigation** region (the main menu)
   - A **main** content region
   - A **footer** region (if a footer exists)

**Steps — Customer Front-end**:

1. Navigate to the Customer Front-end home page with the screen reader running
2. Use the same landmark navigation to confirm:
   - A **banner** / **header** region
   - A **navigation** region
   - A **main** content region
   - Any **complementary** regions (e.g., the basket panel, a sidebar)
   - A **footer** region

**What "good" looks like**:
- The screen reader announces meaningful landmark names when you navigate to them (e.g., "navigation landmark", "main content landmark")
- The page is not one giant undifferentiated block of text — landmarks help screen reader users jump quickly to the section they need
- The main navigation announces its items in a logical order

**Pass Criteria**:
- ✅ At least banner/header, navigation, and main content landmarks are present and announced on every portal
- ✅ Landmark names are meaningful (not "div", "section", or "unnamed region")

**Edge Cases**:
- If the screen reader only announces one or two regions (suggesting the rest of the page has no landmarks), log as a **Medium** bug — this significantly impairs blind users' ability to navigate

---

### TC-RE-053: Focus Indicators Visible on All Interactive Elements

**Given**: You are using the Customer Front-end (no screen reader needed for this test — just keyboard and eyes)

**Steps**:

1. Load the Customer Front-end home page
2. Press **Tab** to cycle through every interactive element on the page: nav links, search field, address input, vendor cards, buttons, etc.
3. For each element, look for a visible **focus ring** — a coloured border or outline that shows which element currently has keyboard focus
4. Repeat on the following pages:
   - A vendor's menu page
   - The basket
   - The checkout address page
   - The checkout payment page
5. Log each interactive element that has **no visible focus indicator** when it receives focus

**What "good" looks like**:
- Every button, link, and form field shows a clear, visible outline when focused — at a minimum, the browser's default blue focus ring
- The focus ring is not hidden or invisible (some designs accidentally suppress focus rings with `outline: none` in CSS)

**Pass Criteria**:
- ✅ All interactive elements on the tested pages have a visible focus indicator

> **Severity guidance**: Missing focus indicators are **Medium** severity — they do not break the site for mouse users but significantly impair keyboard-only and screen reader users.

---

### TC-RE-054: Image Alt Text — Hero Slides, Dish Images, Vendor Logos

**Given**: You are on the Customer Front-end. You do not need a screen reader for this test — you will use the browser's built-in inspector.

**Steps**:

1. On the Customer Front-end home page, right-click on the **hero carousel image** (the large banner image) and choose **"Inspect"** (or press F12 and click the Inspector cursor icon, then click the image)
2. In the DevTools panel, look at the HTML for the image element — it will look something like `<img src="..." alt="...">`
3. Check the value of the `alt` attribute:
   - ✅ Good: `alt="Freshly made pizza from Bella Italia restaurant"` — descriptive text
   - ✅ Acceptable for decorative images: `alt=""` — an empty alt tag tells screen readers to skip decorative images
   - ❌ Bad: `alt="image"`, `alt="banner"`, `alt="img_001.jpg"` — these are unhelpful
   - ❌ Bad: no `alt` attribute at all
4. Repeat this check for:
   - **Dish images** on a vendor's menu page (pick 3 dishes)
   - **Vendor logo / thumbnail** on the vendor list page and vendor detail page
   - Any **promotional banner** images

**What "good" looks like**:
- Hero slides have descriptive alt text describing what the image shows
- Dish images have alt text naming the dish (e.g., `alt="Margherita pizza"`)
- Vendor logos have alt text naming the vendor (e.g., `alt="Bella Italia restaurant logo"`)
- Purely decorative images (background patterns, dividers) use `alt=""`

**Pass Criteria**:
- ✅ All content images have meaningful alt text
- ✅ No content images are missing the alt attribute
- ✅ No alt text contains file names or placeholder text like "image" or "photo"

**Edge Cases**:
- If a dish has no image uploaded (blank or placeholder), the placeholder should have appropriate alt text like `alt="No image available"` or `alt=""`

---

### TC-RE-055: Form Labels Associated with Inputs

**Given**: You are on a form — use the customer checkout address form or the System Admin vendor edit form

**Steps**:

1. Navigate to the checkout address form (Customer Front-end) or the vendor edit form (System Admin)
2. For each visible form label (e.g., "First Name", "Delivery Address", "Phone Number"):
   - Click on the **label text itself** (not the input box — click the words of the label)
   - Observe whether the associated input field receives focus (the cursor should appear in the input)
3. Alternatively, use the browser inspector: right-click an input → Inspect → look for a `<label for="...">` element that references the input's `id` attribute

**What "good" looks like**:
- Clicking on a label text moves focus to its associated input field — this is the HTML `for` / `id` association working correctly
- Screen readers announce the label name when the input is focused (e.g., "First Name, edit field")
- Every visible label is associated with exactly one input — there are no "orphan" labels (labels with no associated input) or "orphan" inputs (inputs with no label)

**Pass Criteria**:
- ✅ Clicking each label focuses its associated input
- ✅ No form inputs are missing a label (every field has a visible, associated label)

> **Severity guidance**: Inputs without labels are a **Medium** accessibility bug — screen reader users cannot identify what to enter in an unlabelled field.

---

### TC-RE-056: Colour Contrast Spot-Check — Action Buttons

**Given**: You are on the System Admin portal and the Customer Front-end

> **Background**: WantFood's System Admin convention requires that all action buttons use **solid Bootstrap button variants** (e.g., `btn-primary` for main actions, `btn-danger` for destructive actions) with **both an icon and a text label**. Outline button variants (`btn-outline-primary`, `btn-outline-danger`, etc.) and icon-only buttons are not permitted.

**Steps — System Admin button audit**:

1. On the System Admin portal, visit the following pages and inspect the action buttons:
   - Vendor edit page → **"Save"** and **"Delete"** buttons
   - Applications list → **"Approve"** and **"Reject"** buttons
   - Orders list → any action buttons
   - Commissions or invoices pages → any action buttons
2. For each button, note:
   - Does it use a **solid colour** background (e.g., blue for primary, red for danger)? ✅
   - Or does it use an **outline** style (transparent background with a coloured border)? ❌ — log as **Medium** bug
   - Does it have **both** an icon and a text label? ✅
   - Or is it **icon only** (no text)? ❌ — log as **Medium** bug
   - Or is it **text only** (no icon)? This is acceptable but note it for consistency

**Steps — Contrast spot-check**:

3. On the Customer Front-end, visit the home page and checkout
4. Look at the main call-to-action buttons (e.g., "Order Now", "Add to Basket", "Place Order")
5. Visually assess whether the text on each button is clearly readable against its background — for example, white text on a dark blue button is easy to read; light grey text on a white background may not be

**What "good" looks like**:
- System Admin action buttons always use solid variants with both icon and text
- Customer Front-end buttons have strong contrast — dark text on a light background, or light text on a dark background
- No button uses a colour combination that is hard to read even for users with normal vision

**Pass Criteria**:
- ✅ All System Admin action buttons use solid Bootstrap variants (not outline)
- ✅ All System Admin action buttons include both an icon and a text label
- ✅ Customer Front-end primary action buttons have clearly readable text

> **Severity for violations**: Any System Admin button using an outline variant or missing a text label = **Medium** bug. Any button with genuinely unreadable text contrast = **High** bug.

---

### TC-RE-057: Error Message Clarity and Screen Reader Announcement

**Given**: You have NVDA or VoiceOver running (see the [Quick Start guide](#quick-start-turning-on-a-screen-reader) above). You are on a form.

**Steps**:

1. Navigate to the **checkout address form** on the Customer Front-end
2. Leave all required fields **blank**
3. Click **"Continue"** or **"Get Quote"** to submit the empty form
4. Listen to what the screen reader announces
5. Observe what appears on screen

Also test:
6. Navigate to the **System Admin — Vendor Edit form**
7. Clear the vendor's required "Name" field
8. Click **"Save"**
9. Listen to the screen reader announcement

**What "good" looks like**:
- When the form is submitted with errors:
  - The screen reader announces the error — either by reading out each error message, or by announcing "X errors found, please review" and then reading the errors when focus moves to them
  - Visible error messages appear next to (or below) each invalid field in red text
  - The error message text is specific: "Please enter your delivery address" rather than just "Error" or "Required"
- Focus moves to the first error field (or to a summary of errors at the top of the form) — the user is not left at the bottom of the page with an unexplained failure

**Pass Criteria**:
- ✅ Screen reader announces the error when the form is submitted with missing required fields
- ✅ Visible error messages are specific and helpful
- ✅ Focus moves to the first error or an error summary

**Edge Cases**:
- If the error messages appear visually but the screen reader does not announce them, log as a **Medium** bug — errors must be accessible programmatically, not just visually

---

### TC-RE-058: Skip-to-Content Link Presence

**Given**: You are on each portal's home or main page (Customer Front-end, System Admin, Vendor Admin, Driver Portal)

**Steps**:

1. Navigate to the Customer Front-end home page
2. Press **Tab** once — before any other navigation item receives focus
3. Look for a **"Skip to main content"** or **"Skip navigation"** link appearing at the top of the page. It may be invisible until focused — that is acceptable.
4. Press **Enter** on the skip link if it appears
5. Confirm that focus jumps directly to the main content area (past the navigation)
6. Repeat steps 1–5 on the System Admin portal, Vendor Admin portal, and Driver Portal

**What "good" looks like**:
- The first element focused on each portal (before the main nav) is a "Skip to main content" link
- The link may be visually hidden until focused — it becomes visible when keyboard focus reaches it
- Pressing Enter on the link jumps focus past the navigation to the main content, so keyboard users do not have to Tab through the entire navigation on every page

**Pass Criteria**:
- ✅ A skip-to-content link exists on every portal
- ✅ The link is the first focusable element on the page
- ✅ Activating the link moves focus to the main content area

> **Severity**: Missing skip-to-content links are a **Medium** bug — they significantly impair keyboard-only and screen reader users who must navigate the same menu on every page load.

---

## 7. Admin Portal Mobile Responsiveness

These tests check that the admin portals remain usable on smaller screens. Whilst admin portals are primarily desktop tools, a restaurant manager might check orders on a tablet, and a driver always uses a phone.

---

### TC-RE-060: System Admin on Tablet — iPad Portrait (768 × 1024)

**Given**: You can resize your browser window or use Chrome DevTools to simulate a tablet screen

**How to simulate an iPad in Chrome**:
1. Press **F12** to open DevTools
2. Click the **"Toggle device toolbar"** icon — it looks like a phone/tablet icon at the top left of the DevTools panel (or press **Ctrl + Shift + M** on Windows / **Cmd + Shift + M** on Mac)
3. In the device toolbar that appears, click the device dropdown (it may say "Responsive") and select **iPad** or type in **768** width and **1024** height
4. Reload the page

**Steps**:

1. Simulate iPad portrait (768 × 1024) as described above
2. Navigate to the System Admin portal
3. Check the **main navigation**: does it collapse into a hamburger menu (☰ icon) or remain expanded?
4. If it collapses: click the hamburger menu and confirm all navigation items are accessible
5. Navigate to the **Vendors list** and observe the data table: does it horizontally scroll if there are many columns, or do columns stack?
6. Navigate to the **Vendor Edit form** and confirm all form fields are visible and usable — nothing cut off or overlapping
7. Check the **Commissions** and **Invoices** sections for table readability

**What "good" looks like**:
- The navigation collapses neatly into a hamburger menu at tablet width, and all items are accessible from it
- Data tables scroll horizontally rather than columns being cut off or overlapping
- Form fields and their labels are stacked vertically at tablet width — not squished horizontally
- Buttons remain tappable (large enough to tap with a finger — at least 44 × 44 pixels)
- No content spills outside the viewport (no horizontal scroll on the page itself — only within tables where expected)

**Pass Criteria**:
- ✅ Navigation accessible at 768px width
- ✅ Data tables are usable (horizontal scroll within the table is acceptable)
- ✅ Forms are usable with no cut-off fields
- ✅ No unintended page-level horizontal scroll

---

### TC-RE-061: System Admin on Mobile — Phone (414 × 896)

**Given**: Using Chrome DevTools device toolbar (as described in TC-RE-060), set the viewport to **414 × 896** (iPhone 11 Pro size)

**Steps**:

1. Simulate 414 × 896 in Chrome DevTools
2. Navigate to the System Admin portal
3. Attempt to log in, view the dashboard, and navigate to the Vendors list

**What "good" looks like**:
- The portal is **viewable** — content is not completely broken or invisible
- Navigation is accessible via a collapsed hamburger menu
- The page is not completely unusable, but it is acceptable to note layout issues that would benefit from a proper mobile optimisation

> **Perspective for testers**: System Admin is primarily a desktop tool. A few layout issues at 414px are expected and may be accepted as known limitations. The goal is to check nothing is **blocking** — for example, a page that cannot be scrolled to at all, or a modal that appears off-screen entirely.

**Pass Criteria**:
- ✅ The portal loads and is viewable at 414px width
- ✅ Login works
- ✅ Navigation is accessible via hamburger menu

**Edge Cases**:
- Log any element that appears completely off-screen and cannot be scrolled to as a **Medium** bug
- Log any page that completely fails to render (blank page) as a **High** bug

---

### TC-RE-062: Vendor Admin on Tablet (768 × 1024)

**Given**: Using Chrome DevTools device toolbar, simulate an iPad portrait viewport (768 × 1024)

**Steps**:

1. Simulate 768 × 1024 in Chrome DevTools
2. Navigate to the Vendor Admin portal and log in
3. Check the following areas:
   - **Dashboard / home page**: metrics and navigation visible?
   - **Live Orders kanban**: are the Pending / Accepted / Ready / Dispatched columns visible and scrollable?
   - **Menu Management**: can you view the dish list and open a dish for editing?
   - **Settings / Profile**: form fields accessible?

**What "good" looks like**:
- The Live Orders kanban is usable on a tablet — a restaurant manager checking orders on an iPad in the kitchen should be able to use this comfortably
- If the kanban columns do not all fit horizontally, horizontal scrolling within the kanban is acceptable — the columns themselves must be readable and tappable
- Menu management table and forms are fully usable
- Buttons are large enough to tap comfortably

**Pass Criteria**:
- ✅ Vendor Admin is fully usable at 768px width
- ✅ Live Orders kanban is navigable and usable
- ✅ Menu management is usable
- ✅ All buttons are tappable at tablet scale

---

### TC-RE-063: Vendor Admin Live Order Kanban and Driver Portal on Mobile Phone

**Part A — Vendor Admin "Accept Order" on Mobile (414 × 896)**:

**Steps**:

1. Simulate 414 × 896 in Chrome DevTools
2. Navigate to the Vendor Admin **Live Orders** kanban
3. Locate an order in the "Pending" column
4. Check that the **"Accept"** button (or the order card itself, if it is tappable) is visible and reachable — you should not need to scroll excessively or zoom in to find it
5. Tap (click) the Accept button and confirm the order can be accepted from a phone screen

**What "good" looks like**:
- The "Accept" button is reachable and tappable on a 414px screen
- Order details (items, total, address) are readable
- Accepting an order works as expected

**Pass Criteria**:
- ✅ "Accept Order" button is reachable and functional at 414px width

---

**Part B — Driver Portal Portrait / Landscape Rotation (Mobile)**:

**Steps**:

1. Either use a real mobile phone (navigate to the Driver Portal UAT URL in the phone's browser) or simulate in Chrome DevTools by selecting an iPhone profile
2. In **portrait** orientation: confirm the active delivery list is visible, the delivery address is readable, and the "Mark as Delivered" button is visible
3. Rotate to **landscape** orientation (or in DevTools, swap width and height): confirm the same elements remain accessible and the layout has not broken

**What "good" looks like**:
- Portrait: the Driver Portal is clean and easy to use — a driver checking their deliveries in portrait mode should find everything they need without zooming
- Landscape: the layout adjusts — content is wider, no elements are cut off, the "Mark as Delivered" button remains visible
- No fixed-position elements (headers, floating buttons) obscure the main content in either orientation

**Pass Criteria**:
- ✅ Driver Portal is fully usable in portrait on a mobile screen
- ✅ Rotating to landscape does not break the layout
- ✅ "Mark as Delivered" button is accessible in both orientations

---

## 8. Performance Budgets

These tests measure how quickly key pages load. You do not need specialist tools — a stopwatch and Chrome DevTools are sufficient.

> **Background**: The WantFood introduction guide sets a general 3-second page-load budget. For the most critical customer-facing pages, we apply tighter budgets to ensure a good experience.

---

### How to Measure Page Load Time in Chrome

**Method 1 — Chrome DevTools (more accurate)**:

1. Press **F12** to open DevTools
2. Click the **"Network"** tab
3. Tick the **"Disable cache"** checkbox (this simulates a first visit — a returning visitor would see faster times)
4. Press **Ctrl + Shift + R** (Windows) or **Cmd + Shift + R** (Mac) to do a **hard refresh** — this loads the page fresh
5. Look at the bottom of the Network tab: it shows "X requests, X transferred, finished in X.XXs" — the **"DOMContentLoaded"** time (in blue) or the total **"Load"** time (in red) are the ones to note
6. The metric we care about for these tests is the time until the page is visually usable — this roughly corresponds to **"DOMContentLoaded"** for most pages

**Method 2 — Stopwatch (quick approximation)**:

1. Clear your browser cache: press **Ctrl + Shift + Delete** → select "Cached images and files" → clear
2. Start your stopwatch (your phone will do)
3. Navigate to the URL
4. Stop the stopwatch when the page appears usable — when you could start interacting with it (hero image visible, vendor cards loaded, etc.)

---

### TC-RE-070: Customer Home Page — First Contentful Paint Under 2 Seconds

**Given**: You are measuring from a cleared cache, on a broadband connection (no throttling applied)

**Steps**:

1. Clear your browser cache (Ctrl + Shift + Delete → Cached images and files)
2. Open Chrome DevTools → Network tab → **untick** "Disable cache" for the first run, then use Method 2 above for simplicity
3. Hard-refresh (`Ctrl + Shift + R`) the Customer Front-end home page
4. Start the stopwatch when you press the refresh / navigate key
5. Stop the stopwatch when the page is visually usable — hero image is visible, navigation is accessible, at least one vendor card or search field is showing

**Budget**: The home page should become visually usable within **2 seconds** on a broadband connection.

**What "good" looks like**:
- The page loads in under 2 seconds and feels snappy — images appear quickly, there is no extended blank-screen period
- If using DevTools: the DOMContentLoaded event fires under 2 seconds

**Pass Criteria**:
- ✅ Customer home page visually usable within 2 seconds on broadband

> **If the budget is missed**: Log a **Medium** performance bug with the exact measured time. Include the DevTools Network tab screenshot if possible. Do not log as High unless the page takes over 5 seconds.

**Edge Cases**:
- First load (cold cache) may be slower than subsequent loads — test both and note the difference

---

### TC-RE-071: Vendor Admin Live Order Kanban Interactive Under 3 Seconds

**Given**: You are logged in to the Vendor Admin portal on a broadband connection. The kanban has at least 2–3 active orders.

**Steps**:

1. Clear your browser cache
2. Navigate directly to the Live Orders kanban URL (or navigate via the menu)
3. Start your stopwatch when you navigate to the kanban
4. Stop when the kanban is visually usable — columns are rendered and order cards are visible

**Budget**: The kanban should be interactive within **3 seconds** on broadband.

**What "good" looks like**:
- The kanban columns appear and order cards load within 3 seconds — a restaurant manager accepting orders needs real-time responsiveness, not a 5-second wait each time they check in

**Pass Criteria**:
- ✅ Vendor Admin Live Order Kanban visually usable within 3 seconds on broadband

**Edge Cases**:
- Kanban with many orders (10+ across all columns) — loading time should still be under 3 seconds. If more orders significantly increase load time, this is a scalability concern worth noting.

---

### TC-RE-072: Customer Search Results Under 1.5 Seconds

**Given**: You are on the Customer Front-end, which has a populated vendor index in the UAT environment (at least 3–5 active vendors)

**Steps**:

1. Clear your browser cache
2. Navigate to the Customer Front-end home page
3. Enter a search term or location in the search/address field to trigger a vendor search
4. Start your stopwatch when you submit the search (press Enter or click the search button)
5. Stop when search results are visible — at least the first row of vendor cards is displayed

**Budget**: Search results should appear within **1.5 seconds** on broadband.

**What "good" looks like**:
- A customer typing their address and pressing search sees results in under 1.5 seconds — this is a critical path that directly affects conversion

**Pass Criteria**:
- ✅ First search results visible within 1.5 seconds on broadband

> **If the budget is missed**: Log as **Medium** (under 3 seconds) or **High** (over 3 seconds). Include timing evidence.

**Edge Cases**:
- Empty search (no results found) — the empty state message should also appear within 1.5 seconds, not spin for a long time before showing "No results"

---

## 9. Summary and Next Steps

### What This Document Covers

This document has tested nine categories of resilience, edge-case, and quality behaviour across all WantFood portals:

| Section | TC Range | What Was Tested |
|---------|----------|-----------------|
| Network Resilience | TC-RE-001–005 | Slow connections, offline periods, Back/Forward navigation under failure |
| Service Failures | TC-RE-010–013 | 404 pages, 5xx errors, expired sessions, payment retry UX |
| Payment Edge Cases | TC-RE-020–025 | Webhook delays, stuck intents, 3DS flows, declined cards, offline 3DS |
| Concurrent Edits | TC-RE-030–032 | Simultaneous dish edits, menu edits during checkout, commission conflicts |
| Locale and Currency | TC-RE-040–043 | GBP formatting, time format consistency, DST crossover, non-English locale |
| Accessibility | TC-RE-050–058 | Keyboard navigation, screen reader landmarks, focus indicators, alt text, form labels, colour contrast, error announcements, skip links |
| Mobile Responsiveness | TC-RE-060–063 | Admin portals on tablet and phone, Driver Portal portrait/landscape |
| Performance Budgets | TC-RE-070–072 | Home page < 2s, Kanban < 3s, Search < 1.5s |

---

### Severity Summary — What to Escalate Immediately

If you find any of the following during these tests, mark them **Critical** and contact your project manager immediately — do not wait until the end of the day:

- A customer is charged without receiving an order confirmation (TC-RE-020, TC-RE-021)
- Browser Back/Forward causes a duplicate payment charge (TC-RE-005)
- A service failure exposes a technical stack trace or database connection details to an end user (TC-RE-011)
- The Driver Portal silently marks a delivery as completed without a server confirmation (TC-RE-003)

---

### How to Log These Tests on Monday.com

When logging bugs from this document, use the group **"Resilience & Edge Cases"** on your UAT Monday.com board (create this group if it does not exist). Include:

- The **TC ID** (e.g., TC-RE-024) in the bug title — this makes cross-referencing easy
- The **exact steps** you took (which throttling setting, which test card number, which browser)
- The **expected result** from this document
- The **actual result** — what you saw, heard, or measured
- A **screenshot or screen recording** where possible — for network tests, a screenshot of the Chrome DevTools Network tab at the time of failure is very helpful

---

### Next Steps

Once all tests in this document are complete:

1. **Review your bug list** with your project manager — distinguish between blockers (Critical/High) and improvements (Medium/Low)
2. **Retest fixed bugs** as the development team resolves them — use the same TC steps to reproduce and verify
3. **Proceed to sign-off**: once all Critical and High bugs are resolved (or formally accepted), you are ready to sign off this section

📄 See **[08-signoff.md](08-signoff.md)** for the sign-off and completion criteria.

---

**Well done for reaching the end of the resilience and edge-case scripts — this is some of the most valuable testing in the entire UAT programme. 🎉**
