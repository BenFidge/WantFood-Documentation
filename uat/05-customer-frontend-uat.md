# Customer Front-end UAT — Test Scripts

## Purpose

This document contains test scripts for the **Customer Front-end** (the public-facing food ordering website). This portal is used by customers to browse restaurants, place orders, track deliveries, leave reviews, and manage their accounts.

**Target users**: End customers placing food delivery orders

---

## Prerequisites

Before testing the Customer Front-end, ensure you have:

- ✅ Access to the Customer Front-end UAT environment URL
- ✅ Customer test account credentials (`customer.uat@wantfood.com`)
- ✅ At least one active vendor with a published menu available in the UAT environment
- ✅ Stripe test card details ready (see [00-introduction.md](00-introduction.md#test-payment-details))
- ✅ Browser and device ready, including mobile (see [00-introduction.md](00-introduction.md#supported-browsers-and-devices))
- ✅ Monday.com board set up (see [01-monday-setup.md](01-monday-setup.md))
- ✅ Screenshot/recording tool ready

---

## Test Data Required

| Item | Description |
|------|-------------|
| **Test Vendor (Card)** | "Test Restaurant 1" — active vendor accepting card payments |
| **Test Vendor (Cash)** | "Test Restaurant 2" — active vendor accepting cash payments |
| **Test Menu** | At least 2 categories and 5 dishes, all with prices set |
| **Test Address (In Zone)** | A delivery address within the test vendor's delivery zone |
| **Test Address (Out of Zone)** | A delivery address outside the test vendor's delivery zone |
| **Active Promo Code** | A valid promo code for testing discount at checkout |

---

> ⚠️ **Propagation reminder**: Changes made via System Admin or Vendor Admin (e.g. activating or deactivating a vendor, updating delivery cost tiers, toggling payment methods, publishing menus) may take up to **~30 seconds** to appear on the Customer Front-end due to cache invalidation. If you do not see an expected change immediately, wait 30 seconds and hard-refresh before raising a bug. See the **Propagation Cheat-Sheet** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) for the full list of propagated changes and their expected timings.

## Table of Contents

1. [Customer Sign-in and Account Setup](#customer-sign-in-and-account-setup)
2. [Home Page and Discovery](#home-page-and-discovery)
3. [Search and Cuisine Filtering](#search-and-cuisine-filtering)
4. [Vendor and Dish Detail Pages](#vendor-and-dish-detail-pages)
5. [Basket Management](#basket-management)
6. [Delivery Address and Quote Generation](#delivery-address-and-quote-generation)
7. [Checkout with Card Payment](#checkout-with-card-payment)
8. [Checkout with Cash Payment](#checkout-with-cash-payment)
9. [Post-Order Confirmation and Live Tracking](#post-order-confirmation-and-live-tracking)
10. [Cancel Order](#cancel-order)
11. [Add Tip Post-Delivery](#add-tip-post-delivery)
12. [Order Reviews](#order-reviews)
13. [Refund Visibility](#refund-visibility)
14. [Share Order](#share-order)
15. [Reorder Flow](#reorder-flow)
16. [Saved Address Management](#saved-address-management)
17. [Customer Account](#customer-account)
18. [Notifications, Privacy and Account Management](#notifications-privacy-and-account-management)

---

## Customer Sign-in and Account Setup

Authentication is handled by **Entra ID (Azure AD B2C)**. There are no local sign-up or password screens within the WantFood Customer Front-end itself. The tests in this section focus on the front-end redirect behaviour, return URL handling, and the customer profile that is created or retrieved after a successful sign-in.

> **Cross-reference**: For comprehensive sign-in round-trip and token-expiry tests across all four portals, see [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md).

---

### TC-FE-150: First-Time Customer Sign-in via Entra ID

**Given**: A new customer has never signed in to the WantFood Customer Front-end before

**Steps**:
1. Navigate to the Customer Front-end UAT URL whilst not signed in.
2. Click **Sign In** (or attempt to access a protected page such as `/Account`).
3. Observe the redirect to the Entra ID sign-in page.
4. Sign in with a new test customer credential (ask your PM for a disposable first-use test account).
5. Complete any Entra ID MFA or consent prompts.
6. Observe the return URL and the resulting page on the Customer Front-end.

**Expected Result**:
- Browser is redirected to the Entra ID sign-in page.
- After successful sign-in, the customer is returned to the Customer Front-end at the correct return URL (not the home page of a different portal).
- A customer profile record is created in the WantFood system (name and email are populated from the Entra ID claims).
- The account page (`/Account`) shows the customer's display name and email address.

**Pass Criteria**:
- ✅ Redirect to Entra ID sign-in page occurs.
- ✅ After sign-in, user lands on the correct Customer Front-end URL.
- ✅ Account page shows name and email from Entra ID.
- ✅ Customer profile is created in the system (verify via System Admin if needed).

**Edge Cases**:
- Customer uses a personal Microsoft account vs. a work/school account — confirm the Entra ID tenant policy accepts both.
- Sign-in prompt cancelled by customer → should return to the Customer Front-end home page (anonymous).

**Cross-portal verification**: See **TC-AC-001** (sign-in round-trip across all portals) and **TC-AC-010** (password reset returns user to correct portal).

---

### TC-FE-151: Sign-out and Return to Anonymous Browsing

**Given**: You are signed in as `customer.uat@wantfood.com` on the Customer Front-end

**Steps**:
1. Navigate to **Account**.
2. Click **Log Out**.
3. Observe the resulting page.
4. Attempt to browse the home page and a vendor page without signing in.

**Expected Result**:
- You are signed out and returned to the Customer Front-end home page (anonymous state).
- You can still browse the home page, search for restaurants, and view vendor pages without being signed in.
- Attempting to access `/Account` or proceed to checkout redirects you to sign in.

**Pass Criteria**:
- ✅ Sign-out completes successfully.
- ✅ Home page is accessible anonymously after sign-out.
- ✅ Protected pages redirect to sign-in when accessed anonymously.

**Edge Cases**:
- Pressing the browser back button after sign-out → should not restore the authenticated session (cached-page behaviour may vary by browser; the server session must be invalidated).

---

### TC-FE-152: Session Expiry Mid-Browse — Graceful Re-authentication

**Given**: You are signed in and the Entra ID token has expired, or your PM has revoked your session from the Entra ID admin portal

**Steps**:
1. Sign in as `customer.uat@wantfood.com`.
2. Wait until the Entra ID token expires (or ask your PM to revoke the session).
3. Without refreshing the page, add an item to your basket.
4. Attempt to proceed to checkout.

**Expected Result**:
- The application detects the expired or revoked token.
- You are redirected to Entra ID to re-authenticate gracefully (not presented with an unhandled error or blank page).
- After re-authenticating, you are returned to the basket or checkout page.
- Your basket contents are preserved (or a warning is shown if the basket was lost due to the session expiry).

**Pass Criteria**:
- ✅ Expired token is detected before or at checkout.
- ✅ Re-authentication flow is triggered cleanly (no unhandled error).
- ✅ After re-authentication, user is returned to a sensible page.

**Edge Cases**:
- Token expires on the order confirmation page → page may show stale data; re-auth must not re-submit the order.

**Cross-portal verification**: See **TC-AC-040** (leave tab open for more than one hour, then attempt an action).

---

## Home Page and Discovery

### TC-FE-001: View Home Page

**Given**: You are a visitor to the Customer Front-end (not logged in)

**Steps**:
1. Navigate to the Customer Front-end UAT URL
2. Observe the home page

**Expected Result**:
- Home page loads successfully
- Hero section (carousel/banner) displays with images and text
- "Support Local" section displays (if published)
- Navigation is accessible: Search, Cuisine Types, Account/Login
- A location or address input is visible for finding nearby restaurants
- A list of nearby or featured restaurants is visible (or a prompt to enter location)

**Pass Criteria**:
- ✅ Home page loads within 3 seconds
- ✅ Hero carousel displays
- ✅ Navigation is accessible

**Edge Cases**:
- No hero slides published → hero section should handle gracefully (hide or show placeholder)
- No vendors active → should show "No restaurants available" or empty state
- Home page on mobile → should display mobile-optimised layout

---

### TC-FE-002: Nearby Vendor Discovery (Location-Based)

**Given**: You are on the Home page

**Steps**:
1. Enter your delivery address or allow browser location access
2. Observe the list of nearby restaurants

**Expected Result**:
- Restaurants within the delivery zone for your address are displayed
- Each restaurant card shows: name, cuisine type, estimated delivery time, delivery fee, rating
- Clicking on a restaurant takes you to the vendor's page

**Pass Criteria**:
- ✅ Nearby restaurants are displayed for a valid address
- ✅ Restaurant cards show all key information

**Edge Cases**:
- Address outside all delivery zones → should show "No restaurants deliver to this address" or similar
- Location permission denied → should fall back to manual address entry
- No nearby restaurants → should show "No restaurants available in your area"
- Very rural address → should handle gracefully

---

## Search and Cuisine Filtering

### TC-FE-010: Search for a Restaurant

**Given**: You are on the Customer Front-end

**Steps**:
1. Click the **Search** icon or navigate to the Search page
2. Type a restaurant name (e.g., "Test Restaurant")
3. Observe the results

**Expected Result**:
- Search results display matching restaurants
- Results include: restaurant name, cuisine type, rating, estimated delivery time
- Clicking a result takes you to the restaurant's page

**Pass Criteria**:
- ✅ Search returns relevant results
- ✅ Results are displayed correctly
- ✅ Clicking a result navigates to the correct page

**Edge Cases**:
- Search with no results → should display "No restaurants found for '[search term]'"
- Search with a partial match → should show partial results (e.g., "Test" matches "Test Restaurant")
- Search with special characters → should handle gracefully

---

### TC-FE-011: Search for a Dish

**Given**: You are on the Search page

**Steps**:
1. Enter a dish name in the search box (e.g., "Burger" or "Pizza")
2. Observe the results

**Expected Result**:
- Search results display restaurants that offer the searched dish
- Alternatively, matching dishes are listed with their restaurant name

**Pass Criteria**:
- ✅ Dish search returns relevant results
- ✅ Results include restaurant context

**Edge Cases**:
- Searching for a dish not available → should display "No results"
- Searching for a dish available at multiple restaurants → all should appear

---

### TC-FE-012: Browse by Cuisine Type

**Given**: You are on the Customer Front-end

**Steps**:
1. Navigate to **Cuisine Types** or click a cuisine type filter/icon on the home page
2. Select a cuisine type (e.g., "Italian", "Chinese", "Indian")
3. Observe the filtered restaurant list

**Expected Result**:
- Only restaurants matching the selected cuisine type are displayed
- The selected cuisine type is highlighted or shown as an active filter
- You can clear the filter to return to all restaurants

**Pass Criteria**:
- ✅ Filter returns only matching restaurants
- ✅ Selected filter is clearly indicated
- ✅ Filter can be cleared

**Edge Cases**:
- Cuisine type with no restaurants → should display "No restaurants available for [Cuisine Type]"
- Selecting multiple cuisine types (if supported) → should show restaurants matching any selected type

---

## Vendor and Dish Detail Pages

### TC-FE-020: View Vendor Page

**Given**: You have found a restaurant you want to order from

**Steps**:
1. Click on the restaurant (from search, nearby list, or cuisine filter)
2. Observe the vendor page

**Expected Result**:
- Vendor page displays:
  - Restaurant name, logo/banner image, rating, number of reviews
  - Cuisine type(s)
  - Opening hours and current status (Open/Closed)
  - Delivery fee, estimated delivery time, minimum order value
  - Menu sections with categories and dishes
- You can click on a dish to view its details
- You can add dishes to your basket from this page

**Pass Criteria**:
- ✅ All vendor information is displayed
- ✅ Menu is displayed with categories and dishes
- ✅ Open/Closed status is correct

**Edge Cases**:
- Vendor is closed → should display "Currently closed" and prevent adding items
- Vendor has no reviews → should show "No reviews yet"
- Very long menu (50+ dishes) → should scroll or paginate correctly

---

### TC-FE-021: View Dish Detail

**Given**: You are on a vendor page with menu items

**Steps**:
1. Click on a dish from the menu
2. Observe the dish detail view (modal or page)

**Expected Result**:
- Dish details are displayed:
  - Dish name, description, price
  - Image (if available)
  - Dietary information (vegetarian, vegan, gluten-free, allergens)
  - Any option groups (e.g., size, extras, toppings) if applicable
- An "Add to Basket" button is visible

**Pass Criteria**:
- ✅ Dish details are displayed correctly
- ✅ Price is shown
- ✅ "Add to Basket" button is accessible

**Edge Cases**:
- Dish with no image → should show placeholder
- Dish with multiple option groups → all options are displayed and selectable
- Dish unavailable → should show "Currently unavailable" and disable "Add to Basket"

---

## Basket Management

### TC-FE-030: Add Item to Basket

**Given**: You are viewing a vendor's page and the restaurant is open

**Steps**:
1. Click on a dish
2. Select any required options (if applicable)
3. Click **"Add to Basket"**
4. Observe the basket

**Expected Result**:
- A success indicator appears: "Item added to basket" or the basket icon updates
- The dish appears in the basket with: name, quantity, price
- The basket total updates to include the new item

**Pass Criteria**:
- ✅ Item is added to the basket
- ✅ Basket total is updated correctly

**Edge Cases**:
- Adding the same item twice → quantity should increment (e.g., 2 × Burger)
- Adding item without selecting required options → should show validation error
- Adding item to basket from a different browser tab → basket should update

---

### TC-FE-031: Update Item Quantity in Basket

**Given**: You have at least one item in your basket

**Steps**:
1. Open the basket (click basket icon or expand basket panel)
2. Increase the quantity of an item (click "+" button)
3. Decrease the quantity of an item (click "–" button)
4. Observe the basket total after each change

**Expected Result**:
- Quantity updates when "+" or "–" is clicked
- Basket total updates to reflect the quantity change
- Decreasing quantity to 0 removes the item from the basket

**Pass Criteria**:
- ✅ Quantity updates correctly
- ✅ Basket total recalculates
- ✅ Item is removed when quantity reaches 0

**Edge Cases**:
- Setting quantity to a very large number (e.g., 100) → should allow or apply a reasonable maximum limit
- Rapidly clicking "+" multiple times → basket should handle debounced or sequential updates

---

### TC-FE-032: Remove Item from Basket

**Given**: You have at least one item in your basket

**Steps**:
1. Open the basket
2. Click the **"Remove"** button or trash icon next to an item
3. Confirm removal (if prompted)

**Expected Result**:
- The item is removed from the basket
- The basket total updates
- If the basket is now empty, an "empty basket" state is shown

**Pass Criteria**:
- ✅ Item is removed
- ✅ Basket total updates
- ✅ Empty basket state displays correctly

**Edge Cases**:
- Removing the last item from the basket → basket shows "Empty" state
- Accidental removal → there is no undo, but user can re-add the item

---

### TC-FE-033: Restaurant Conflict Handling in Basket

**Given**: You already have items from "Test Restaurant 1" in your basket

**Steps**:
1. Navigate to a **different** restaurant ("Test Restaurant 2")
2. Click **"Add to Basket"** for a dish from that restaurant
3. Observe the result

**Expected Result**:
- A warning or confirmation dialog appears:
  - "Your basket contains items from [Test Restaurant 1]. Adding items from [Test Restaurant 2] will clear your current basket."
  - Buttons: "Start New Basket" (or "Replace") and "Cancel"
- If you choose "Cancel": your original basket is unchanged
- If you choose "Start New Basket": your original basket is cleared and the new item is added

**Pass Criteria**:
- ✅ Warning dialog displays
- ✅ "Cancel" preserves original basket
- ✅ "Start New Basket" clears original basket and adds new item

**Edge Cases**:
- Adding a second item from the same restaurant → no conflict warning
- Refreshing the page during the conflict dialog → original basket should be preserved

---

## Delivery Address and Quote Generation

### TC-FE-040: Enter Delivery Address and Get Quote

**Given**: You have items in your basket and are ready to proceed to checkout

**Steps**:
1. Open the basket or navigate to checkout
2. Enter a delivery address in the address field
3. Wait for the delivery quote to generate (the price may update as you type — a 500ms debounce is used)
4. Observe the quote details

**Expected Result**:
- A delivery quote is generated:
  - Subtotal (items)
  - Delivery fee
  - Service fee (if applicable)
  - Total amount
- The quote updates if you change the address
- The quote is shown before you enter payment details

**Pass Criteria**:
- ✅ Delivery quote generates for a valid in-zone address
- ✅ Quote details are shown (subtotal, delivery fee, total)
- ✅ Quote updates when address changes

**Edge Cases**:
- Typing rapidly (multiple characters) → quote should not fire multiple concurrent requests (debounce works)
- Entering a partial address → quote should wait for a complete address or prompt for more detail
- Quote fails to load → should show error message

**Cross-portal verification**: Delivery fee shown by this quote is driven by the vendor's distance-based delivery cost tiers — see **TC-XP-022** (delivery cost update). If the vendor is currently closed, the quote endpoint will be blocked — see **TC-XP-023** (trading hours / closed vendor).

---

### TC-FE-041: Address Outside Delivery Zone

**Given**: You have items in your basket and are entering a delivery address

**Steps**:
1. Enter a delivery address that is **outside** the test vendor's delivery zone
2. Wait for the response

**Expected Result**:
- A clear error message appears: "Sorry, we don't deliver to this address" or "This address is outside our delivery zone"
- The checkout cannot proceed with this address
- You can change the address and try again

**Pass Criteria**:
- ✅ Error message is clear and helpful
- ✅ Checkout is blocked for out-of-zone addresses
- ✅ Changing to an in-zone address allows checkout to proceed

**Edge Cases**:
- Postcode exists but is on the boundary of the delivery zone → system should handle consistently (either in or out)
- Invalid postcode format → should show validation error before attempting a quote

**Cross-portal verification**: "Outside delivery zone" is implemented as an address that falls beyond the furthest distance-based delivery cost tier, not a polygon boundary — see **TC-XP-022** (delivery cost update and tier configuration) for the admin-side configuration that governs this behaviour.

---

### TC-FE-042: Quote Expiry

**Given**: You have generated a delivery quote but have been on the page for a while

**Steps**:
1. Generate a delivery quote (enter a valid address)
2. Leave the checkout page open for an extended period (e.g., 15–30 minutes, or check system timeout)
3. Attempt to submit the order

**Expected Result**:
- If the quote has expired:
  - The checkout page shows a message: "Your delivery quote has expired. Please refresh to get a new quote."
  - The system prevents submission with an expired quote
  - Refreshing the address generates a new valid quote

**Pass Criteria**:
- ✅ Expired quote is detected and blocked
- ✅ Clear error message about expiry
- ✅ Refreshing quote generates a new valid one

**Edge Cases**:
- Quote expires while the user is filling in payment details → should detect on submission
- Quote is refreshed just before expiry → new quote generates without losing basket

---

## Checkout with Card Payment

### TC-FE-050: Complete Checkout with Card (Successful Payment)

**Given**: You have items in your basket, a valid in-zone delivery address entered, and a Stripe test card ready

**Steps**:
1. Proceed to the checkout page
2. Confirm your delivery address is correct
3. Enter your contact details: email, phone number, delivery instructions (optional)
4. Select **"Pay by Card"** as the payment method
5. Wait for the Stripe Payment Element (card input) to load
6. Enter test card details:
	- Card Number: `4242 4242 4242 4242`
	- Expiry: Any future date (e.g., 12/29)
	- CVV: Any 3 digits (e.g., 123)
7. Click **"Place Order"** or **"Pay Now"**
8. Wait for the payment to process

**Expected Result**:
- The Stripe Payment Element loads correctly
- Payment is processed successfully
- You are redirected to the **Order Confirmation** page
- Confirmation page shows:
  - Order ID
  - Order items and totals
  - Estimated delivery time
  - Delivery address
- You receive an order confirmation email

**Pass Criteria**:
- ✅ Payment processes without errors
- ✅ Order Confirmation page displays
- ✅ Order ID is shown
- ✅ Order is visible in Vendor Admin as a "Pending" order (verify cross-portal)

**Edge Cases**:
- Stripe Payment Element takes too long to load → should show loading indicator
- Submitting without card details → should show validation error
- Browser autocomplete fills wrong card details → user should be able to clear and re-enter

**Cross-portal verification**: Card payment availability depends on platform Stripe settings and the vendor's Stripe Connect status — see **TC-XP-019** (platform payment settings), **TC-XP-020** (vendor Stripe Connect connect / disconnect), and **TC-XP-021** (vendor cash / card toggle).

---

### TC-FE-051: Card Payment Declined

**Given**: You are at the checkout with items in your basket

**Steps**:
1. Complete all checkout fields
2. Enter the declined test card:
	- Card Number: `4000 0000 0000 0002`
	- Expiry: Any future date
	- CVV: Any 3 digits
3. Click **"Place Order"** or **"Pay Now"**

**Expected Result**:
- Payment is declined
- An error message appears: "Your card was declined. Please try a different payment method."
- The checkout page remains with all fields intact (you do not lose your basket or address)
- You can try again with a different card

**Pass Criteria**:
- ✅ Payment is declined
- ✅ Error message is clear and helpful
- ✅ Checkout remains intact for retry

**Edge Cases**:
- Multiple declined attempts → each should show the error message without side effects
- Declined payment → order should NOT be created in Vendor Admin (verify cross-portal)

---

### TC-FE-052: Card Payment Requiring 3D Secure Authentication

**Given**: You are at the checkout with items in your basket

**Steps**:
1. Complete all checkout fields
2. Enter the 3DS test card:
	- Card Number: `4000 0025 0000 3155`
	- Expiry: Any future date
	- CVV: Any 3 digits
3. Click **"Place Order"** or **"Pay Now"**
4. A Stripe 3DS authentication popup/iframe should appear
5. Follow the authentication instructions (in test mode, click "Authenticate" or "Complete")

**Expected Result**:
- Stripe 3DS authentication popup appears
- After completing authentication, payment is processed
- You are redirected to the Order Confirmation page

**Pass Criteria**:
- ✅ 3DS popup appears
- ✅ Authentication can be completed
- ✅ Payment succeeds after 3DS
- ✅ Order Confirmation page displays

**Edge Cases**:
- Failing 3DS authentication (click "Fail") → payment should decline and show error message
- Closing 3DS popup without completing → checkout should remain intact for retry

---

### TC-FE-053: Payment Intent Polling Behaviour

**Given**: You have just placed an order and are on the order confirmation or loading page

**Steps**:
1. After clicking "Place Order", observe the loading state
2. Wait for the payment confirmation to resolve

**Expected Result**:
- A loading indicator is shown while payment is being confirmed
- Payment confirms within a reasonable time (under 10 seconds in test mode)
- On success: redirected to Order Confirmation page
- On failure: error is shown with instructions to retry

**Pass Criteria**:
- ✅ Loading state is shown during payment processing
- ✅ Confirmation resolves successfully
- ✅ No blank pages or unexpected errors during polling

**Edge Cases**:
- Slow network during polling → loading state should persist, not time out immediately
- Payment polling times out → should show clear error and option to retry or check order status

---

## Checkout with Cash Payment

### TC-FE-060: Complete Checkout with Cash Payment

**Given**: You have items in your basket from a vendor that accepts cash, and a valid delivery address entered

**Steps**:
1. Proceed to checkout
2. Select **"Pay with Cash"** as the payment method (should be available for this vendor)
3. Complete all contact detail fields
4. Click **"Place Order"**

**Expected Result**:
- No Stripe card fields are shown (cash payment skips Stripe)
- Order is placed immediately
- You are redirected to the **Order Confirmation** page
- Confirmation page shows the order with payment method: "Cash on Delivery"
- Order appears in Vendor Admin as "Pending" with payment method "Cash"

**Pass Criteria**:
- ✅ Cash payment option is available
- ✅ Order is placed without card entry
- ✅ Order Confirmation page displays
- ✅ Payment method shows "Cash on Delivery"
- ✅ Order is visible in Vendor Admin (verify cross-portal)

**Edge Cases**:
- Cash payment not available for vendor → "Pay with Cash" option should not appear
- Attempting to apply a promo code with cash payment → should work the same as card

**Cross-portal verification**: The "Pay with Cash" option only appears when the vendor has cash payments enabled — see **TC-XP-021** (enable / disable cash at vendor).

---

## Post-Order Confirmation and Live Tracking

### TC-FE-070: View Order Confirmation Page

**Given**: You have just placed an order successfully

**Steps**:
1. Observe the Order Confirmation page

**Expected Result**:
- Order confirmation page displays:
  - "Thank you for your order!" or similar message
  - Order ID (reference number)
  - Order items and prices
  - Delivery address
  - Estimated delivery time
  - Payment method
  - A link to view your order or track the delivery

**Pass Criteria**:
- ✅ Order confirmation page displays
- ✅ Order ID is shown
- ✅ All order details are correct

**Edge Cases**:
- Refreshing the confirmation page → should still show the confirmation (not re-submit the order)
- Navigating away and back → should show the order in Order History

---

### TC-FE-071: View Order History

**Given**: You are logged in as a customer

**Steps**:
1. Navigate to **Account** → **Orders** or **My Orders**
2. Observe the order history list

**Expected Result**:
- A list of your orders is displayed with columns:
  - Order ID, vendor name, order date, order status, total amount
- You can click on an order to view full details
- Your most recent order is visible in the list

**Pass Criteria**:
- ✅ Orders are listed
- ✅ Most recent order is visible
- ✅ Clicking an order shows full details

**Edge Cases**:
- No previous orders → should display "No orders yet"
- Many orders → pagination works correctly

---

### TC-FE-072: View Order Details

**Given**: You are viewing the Order History list

**Steps**:
1. Click on an order to view its details

**Expected Result**:
- Full order details are displayed:
  - Order ID, status, order date
  - Order items, quantities, prices
  - Subtotal, delivery fee, total
  - Payment method
  - Delivery address
  - Driver name (if assigned and dispatched)
  - Order timeline (status history)

**Pass Criteria**:
- ✅ All order data is displayed correctly
- ✅ Order status is accurate

**Edge Cases**:
- Order without driver assigned → should show "No driver assigned yet"
- Cancelled order → should show cancellation details

---

### TC-FE-073: Live Order Tracking

**Given**: You have a placed order that has been accepted by the vendor and is being prepared or delivered

**Steps**:
1. Navigate to the active order (from Order Confirmation or Order History)
2. Observe the live tracking view

**Expected Result**:
- Live tracking page shows:
  - Current order status (e.g., "Accepted", "Being Prepared", "Out for Delivery")
  - Estimated delivery time (if available)
  - Driver's name (if assigned and dispatched)
  - A map showing driver's approximate location (if available)
- Status updates without requiring a manual page refresh

**Pass Criteria**:
- ✅ Live tracking page loads
- ✅ Current status is shown
- ✅ Status updates in real-time or with a short delay

**Edge Cases**:
- Order still Pending (not yet accepted) → tracking should show "Waiting for restaurant to accept"
- Driver not yet assigned → should show "Preparing your order"
- Live tracking on mobile → should display correctly and update

---

## Cancel Order

### TC-FE-080: Cancel Order (Customer-Initiated, Before Acceptance)

**Given**: You have placed an order that is still in "Pending" status (not yet accepted by the vendor)

**Steps**:
1. Navigate to the active order (from Order Confirmation or Order History)
2. Click **"Cancel Order"**
3. Select a cancellation reason (if prompted)
4. Confirm the cancellation

**Expected Result**:
- A success message appears: "Order cancelled successfully"
- The order status changes to "Cancelled"
- A refund is initiated for card payments (and confirmed in the payment method)
- The cancellation is visible in Vendor Admin
- You receive a cancellation confirmation email

**Pass Criteria**:
- ✅ Order is cancelled
- ✅ Success message displayed
- ✅ Refund is initiated (for card payments)
- ✅ Vendor Admin shows order as cancelled (verify cross-portal)

**Edge Cases**:
- Cancelling after the vendor has accepted the order → may not be possible or may require vendor confirmation (see [06-order-saga-uat.md](06-order-saga-uat.md))
- Cancelling a cash order → no refund needed, but order should be cancelled
- Cancellation reason is required → should enforce validation

**Cross-portal verification**: If the vendor is deactivated mid-order (e.g. by System Admin whilst the order is in "Pending" status), the customer-side cancellation flow should still complete cleanly — see **TC-XP-001** (vendor deactivate — in-flight order edge case).

---

### TC-FE-081: Attempt to Cancel Order After Dispatch

**Given**: You have an order that has been dispatched (driver is on the way)

**Steps**:
1. Navigate to the active order
2. Attempt to click **"Cancel Order"**

**Expected Result**:
- The "Cancel Order" button is disabled or hidden
- A message appears: "This order cannot be cancelled as it has already been dispatched"
- You are directed to contact the restaurant or support

**Pass Criteria**:
- ✅ Cancel button is disabled or hidden after dispatch
- ✅ Clear message explains why cancellation is not possible

**Edge Cases**:
- Cancellation option removed entirely after certain status → confirm this is expected behavior

---

## Add Tip Post-Delivery

### TC-FE-090: Add Tip After Delivery (Card Payment)

**Given**: Your order has been marked as "Delivered" and was paid by card

**Steps**:
1. Navigate to the delivered order (from Order History)
2. Look for a **"Leave a Tip"** or **"Add Tip"** option
3. Click on it
4. Select or enter a tip amount (e.g., £2.00, £3.00, or a custom amount)
5. Confirm and process the tip payment

**Expected Result**:
- A success message appears: "Tip added successfully — thank you!"
- The tip amount is charged to the same payment method as the original order
- The driver receives the tip (visible in their Driver Portal earnings, if applicable)
- The order details page shows the tip amount added

**Pass Criteria**:
- ✅ Tip is added
- ✅ Success message displayed
- ✅ Tip amount is correct
- ✅ Order details show the tip

**Edge Cases**:
- Adding a tip of £0 → should be blocked or warned
- Adding a very large tip (e.g., £100) → should allow or apply a reasonable limit
- Adding a tip to a cash order → "Add Tip" should not be available (cash tips are direct)
- Attempting to add a tip twice → should be blocked after the first tip is added

---

## Order Reviews

### TC-FE-100: Review Prompt After Delivery

**Given**: Your order has been marked as "Delivered"

**Steps**:
1. Navigate to the completed order
2. Look for a review prompt, or navigate to the **Order Complete / Review Prompt Page**

**Expected Result**:
- A review prompt is displayed:
  - "How was your order from [Restaurant Name]?"
  - Star rating selector for the overall experience
  - Option to submit a written review
  - Option to review individual dishes (if applicable)

**Pass Criteria**:
- ✅ Review prompt is displayed for completed orders
- ✅ Rating selector is functional

**Edge Cases**:
- Review prompt not shown for a cancelled order → should not be displayed
- Review prompt shown multiple times → should only be shown once per order

---

### TC-FE-101: Submit Branch/Restaurant Review

**Given**: You are on the review prompt page for a completed order

**Steps**:
1. Click the star rating (e.g., 4 out of 5 stars)
2. Enter a written review (optional but test both with and without text)
3. Click **"Submit Review"**

**Expected Result**:
- A success message appears: "Thank you for your review!"
- The review is submitted and visible on the vendor's page on the front-end
- The vendor can see the review in their Vendor Admin reviews list
- The vendor's rating is updated to reflect the new review

**Pass Criteria**:
- ✅ Review is submitted
- ✅ Success message displayed
- ✅ Review is visible on vendor's front-end page
- ✅ Vendor's rating updates (may take a short delay if recalculation is asynchronous)

**Edge Cases**:
- Submitting a review without a rating → should enforce validation (rating is required)
- Submitting a review with bad words in the text → should be blocked or flagged
- Submitting the same review twice → should be blocked (one review per order per customer)

**Cross-portal verification**: Review text containing bad words is rejected at submission — see **TC-XP-013** (bad words filter). After submission, a review may be hidden or moderated by a System Admin — see **TC-XP-028** (review moderation and rating recalculation).

---

### TC-FE-102: Submit Dish Reviews

**Given**: You are on the review prompt page for a completed order and dish reviews are available

**Steps**:
1. Scroll to the dish review section (or click "Review Dishes")
2. For each dish, select a thumbs up (👍) or thumbs down (👎) (or a star rating, depending on implementation)
3. Add optional comments
4. Click **"Submit Dish Reviews"**

**Expected Result**:
- A success message appears: "Dish reviews submitted — thank you!"
- Individual dish ratings are updated

**Pass Criteria**:
- ✅ Dish reviews are submitted
- ✅ Success message displayed
- ✅ Dish ratings update

**Edge Cases**:
- Skipping some dish reviews → should be allowed (optional per dish)
- Reviewing dishes from an order with only one dish → should work the same way

**Cross-portal verification**: See **TC-XP-013** (bad words filter on review text) and **TC-XP-028** (review moderation and dish rating recalculation).

---

## Refund Visibility

> **Background**: There is no partial-refund UI in the Customer Front-end. When a vendor cancels an order after payment has been captured by Stripe, a **full refund** is initiated automatically. The customer's order detail page should reflect this status and the customer should receive a refund confirmation email.

---

### TC-FE-170: View Refund on a Cancelled Order

**Given**: A card-payment order has been placed successfully and the vendor subsequently cancels it via Vendor Admin (or the order reaches a vendor-cancelled state via the order saga), triggering an automatic Stripe refund

**Steps**:
1. Place a card-payment order as `customer.uat@wantfood.com` (use TC-FE-050 as the baseline).
2. In Vendor Admin, cancel the accepted order after payment has been captured (see TC-VA-056).
3. Navigate to **My Orders** on the Customer Front-end.
4. Open the cancelled order's detail page.
5. Observe the order status and any refund information shown.
6. Check the customer's inbox for a refund or cancellation confirmation email.
7. Verify the Stripe UAT dashboard reflects the refund against the original payment intent.

**Expected Result**:
- The order detail page shows the order status as **Cancelled**.
- A refund status message is visible on the order detail page (e.g. "A full refund of £XX.XX has been initiated" or similar).
- The customer receives a cancellation/refund confirmation email within a reasonable timeframe.
- The Stripe UAT dashboard shows a **Refund** entry against the original payment intent for the full order amount.

**Pass Criteria**:
- ✅ Order status shows as "Cancelled" on the Customer Front-end.
- ✅ Refund status is shown on the order detail page.
- ✅ Customer receives a refund/cancellation email.
- ✅ Stripe dashboard confirms the full refund.

**Edge Cases**:
- Cash-payment order cancelled by vendor → no Stripe refund is expected; order detail should show "Cancelled — Cash on Delivery (no refund applicable)".
- Stripe refund is delayed → order detail page should show "Refund pending" rather than a blank or error state.

**Cross-portal verification**: See vendor-cancel order scenario **S12** in [06-order-saga-uat.md](06-order-saga-uat.md) for the full cross-portal saga. The originating vendor action is covered by **TC-VA-056**.

---

## Share Order

### TC-FE-110: Share Order

**Given**: You have a completed or active order

**Steps**:
1. Navigate to the order details
2. Click **"Share Order"** or the share icon
3. Observe the share options

**Expected Result**:
- A share dialog or page appears with a shareable link or options to share via:
  - Copy link to clipboard
  - Native share (on mobile)
  - Social media (if implemented)
- The shareable link opens a view of the order (with limited details visible to non-account holders)

**Pass Criteria**:
- ✅ Share option is accessible
- ✅ Shareable link is generated
- ✅ Link opens the order view

**Edge Cases**:
- Sharing a cancelled order → shared link should show the order as cancelled
- Opening shared link without being logged in → should show limited order details

---

## Reorder Flow

### TC-FE-120: Reorder from Order History

**Given**: You have a previous completed order in your Order History

**Steps**:
1. Navigate to **My Orders** and find a completed order
2. Click **"Reorder"** or **"Order Again"**
3. Observe what happens

**Expected Result**:
- The items from the previous order are added to your basket
- You are redirected to the checkout or basket page
- If the restaurant is currently closed: a warning is shown ("Restaurant is currently closed — you can reorder when they open")
- If any item from the original order is no longer available: a warning is shown listing the unavailable items that were not added

**Pass Criteria**:
- ✅ Available items from the original order are added to basket
- ✅ Basket total reflects the reordered items
- ✅ Unavailable items are excluded with a warning

**Edge Cases**:
- All items from original order are unavailable → basket should remain empty, show a clear message
- Reordering from a vendor that is currently closed → should be blocked or warned
- Reordering from a vendor that no longer exists → should show error

**Cross-portal verification**: Reorder may fail if dishes have been removed from the menu since the original order was placed — see **TC-XP-005** (dish deleted — propagation to basket). Reorder from a vendor with an unpublished menu should also be blocked or show unavailable items — see **TC-XP-004** (unpublish menu — propagation).

---

## Saved Address Management

### TC-FE-130: View Saved Addresses

**Given**: You are logged in as a customer

**Steps**:
1. Navigate to **Account** → **Addresses**
2. Observe the list of saved addresses

**Expected Result**:
- A list displays all saved delivery addresses with:
  - Address details
  - "Default" label on the default address
- Actions: Add, Edit, Delete, Set as Default

**Pass Criteria**:
- ✅ Saved addresses are listed
- ✅ Default address is clearly indicated
- ✅ Actions are accessible

**Edge Cases**:
- No saved addresses → should display "No addresses saved" with an "Add Address" prompt

---

### TC-FE-131: Add New Address

**Given**: You are on the Addresses page

**Steps**:
1. Click **"Add Address"**
2. Enter a new delivery address (house number, street, city, postcode)
3. Add a label (optional, e.g., "Home", "Work")
4. Click **"Save"**

**Expected Result**:
- A success message appears: "Address saved successfully"
- The new address appears in the Addresses list
- If it's the first address, it is automatically set as the default

**Pass Criteria**:
- ✅ Address is saved
- ✅ Success message displayed
- ✅ Address appears in the list

**Edge Cases**:
- Invalid postcode format → should show validation error
- Missing required fields → should enforce validation
- Duplicate address → should allow (user may have two identical addresses with different labels)

---

### TC-FE-132: Edit Saved Address

**Given**: You have at least one saved address

**Steps**:
1. Click **"Edit"** next to a saved address
2. Modify one or more fields (e.g., add a delivery instruction)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Address updated successfully"
- The updated address is reflected in the Addresses list

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Editing to an invalid postcode → should show validation error

---

### TC-FE-133: Delete Saved Address

**Given**: You have at least two saved addresses (you need at least one remaining after deletion)

**Steps**:
1. Click **"Delete"** next to a saved address
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Address deleted successfully"
- The address is removed from the Addresses list

**Pass Criteria**:
- ✅ Address is deleted
- ✅ Success message displayed

**Edge Cases**:
- Deleting the default address → a new default should be set automatically (or the user should be prompted)
- Deleting the only address → should be allowed, but user will have no saved addresses

---

### TC-FE-134: Set Default Address

**Given**: You have at least two saved addresses

**Steps**:
1. Click **"Set as Default"** next to a non-default address
2. Observe the addresses list

**Expected Result**:
- A success message appears: "Default address updated"
- The selected address is now marked as "Default"
- The previous default address is no longer marked as default
- At checkout, the new default address is pre-filled

**Pass Criteria**:
- ✅ Default address updates
- ✅ Success message displayed
- ✅ Only one address is marked as default at a time

**Edge Cases**:
- Setting an already-default address as default → should be idempotent (no error)

---

## Customer Account

### TC-FE-140: View Account Overview

**Given**: You are logged in as a customer

**Steps**:
1. Navigate to **Account** or your profile
2. Observe the account overview page

**Expected Result**:
- Account overview displays:
  - Your name, email address
  - Quick links to: Orders, Addresses, Settings (if applicable)

**Pass Criteria**:
- ✅ Account overview loads
- ✅ Correct name and email shown

**Edge Cases**:
- Account with no orders → orders section should show "No orders"

---

### TC-FE-141: Privacy Page

**Given**: You are on the Customer Front-end

**Steps**:
1. Navigate to the Privacy Policy page (usually in the footer)
2. Observe the page

**Expected Result**:
- The Privacy Policy page loads and displays the policy text
- The page is readable and professionally formatted

**Pass Criteria**:
- ✅ Privacy page loads
- ✅ Policy text is displayed

**Edge Cases**:
- Privacy page linked from cookie banner → should open correctly

---

### TC-FE-142: Error Page

**Given**: You navigate to a URL that does not exist on the Customer Front-end

**Steps**:
1. Type a non-existent URL into the browser (e.g., `https://uat.wantfood.com/this-page-does-not-exist`)
2. Observe the result

**Expected Result**:
- A friendly 404 error page is displayed
- The page includes: error message, a link back to the home page, and does not expose technical details

**Pass Criteria**:
- ✅ 404 page displays
- ✅ Navigation back to home page works
- ✅ No technical error details are shown to the user

**Edge Cases**:
- Navigating to a vendor page that no longer exists → should show 404 page
- Navigating to an expired order link → should show 404 or "Order not found"

---

## Notifications, Privacy and Account Management

> **Scope note**: The account page (`/Account`) exposes several menu items that are **not yet implemented** in the current build. Confirmed from `AccountController.cs`, the controller contains only two working actions: `Index` (the account overview) and `Addresses`. The following menu items all link to `#` and do nothing when clicked: **Personal Details**, **Payment Methods**, **Promotions & Vouchers**, **Notification Preferences**, and **Help & Support**. Data export and account deletion endpoints do not exist. The test cases in this section record the current state and act as a sign-off gate: a tester should confirm whether each feature has been built before UAT closes.

---

### TC-FE-160: View Account Page

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Observe the page content and navigation menu.

**Expected Result**:
- Account page loads and displays:
  - The customer's display name and email address.
  - "Your Impact" statistics (businesses supported, orders placed).
  - Navigation menu items: **Personal Details**, **Saved Addresses**, **Payment Methods**, **Promotions & Vouchers**, **Notification Preferences**, **Help & Support**.
- **Saved Addresses** navigates to `/Account/Addresses` (working link).
- All other menu items currently navigate to `#` — confirm whether this is still the case or whether any now point to a real page.

**Pass Criteria**:
- ✅ Account page loads.
- ✅ Customer name and email are shown correctly.
- ✅ All menu items are present.
- ✅ "Saved Addresses" navigates to the Addresses page.

**Edge Cases**:
- Account with no orders → "Orders Placed" counter shows `–` (current placeholder behaviour is expected).

---

### TC-FE-161: Notification Preferences — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: The **Notification Preferences** menu item exists on the account page (`/Account`) and shows the label "Push, Email, and SMS", but the link currently points to `#` (i.e. it does nothing when clicked). No controller action or view exists for this route in the current build.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team whether notification preferences are required for launch. If yes, the page must be built (the route and view do not yet exist) and this test case should be updated. If notification preferences are out of scope for this release, the account page should either remove the menu item or display a "Coming soon" message — a dead link (`#`) is poor user experience.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Click **Notification Preferences**.
3. Confirm a preferences page loads (not a `#` anchor or a 404).
4. Toggle off **Email** notifications and save.
5. Refresh and confirm the setting persisted.

**Expected Result**:
- Notification Preferences page loads and shows current settings.
- Toggling a preference and saving persists the change on refresh.

**Pass Criteria** *(once feature is implemented)*:
- ✅ Notification preferences page is accessible from the account page.
- ✅ Each notification channel (Push, Email, SMS) can be toggled independently.
- ✅ Changes persist after page refresh.

**Edge Cases**:
- Turning off all notification types → user should still receive legally required communications (order confirmation, receipt).

---

### TC-FE-162: Personal Details Edit — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: The **Personal Details** menu item exists on the account page (`/Account`), but the link currently points to `#` (i.e. it does nothing when clicked). No controller action or view exists for this route in the current build. Personal data (name, email, phone number) is sourced from the Entra ID claims; any editable fields would need to determine whether changes propagate back to Entra ID or are stored locally in WantFood.
>
> **Action for PM**: Before UAT sign-off, please confirm whether personal details editing is required for launch, and if so, whether changes should update Entra ID directly or only the WantFood profile. The dead `#` link should be resolved before go-live.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Click **Personal Details**.
3. Confirm a page loads (not a `#` anchor or a 404).
4. Edit a field (e.g., phone number) and save.
5. Refresh and confirm the change persisted.

**Expected Result**:
- Personal Details page loads and shows current information.
- Editable fields accept input and save correctly.
- Changes are reflected on the account overview page.

**Pass Criteria** *(once feature is implemented)*:
- ✅ Personal Details page is accessible from the account page.
- ✅ Changes to editable fields persist.
- ✅ Updated data appears on the account overview.

**Edge Cases**:
- Entering an invalid phone number format → validation error shown.
- Name change: if sourced from Entra ID, the UI should make clear whether the change is local or Entra ID-wide.

---

### TC-FE-163: Saved Payment Methods — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: The **Payment Methods** menu item exists on the account page (`/Account`), but the link currently points to `#` (i.e. it does nothing when clicked). No controller action or view exists for this route in the current build. Stripe does not currently expose a saved-card management page within the customer account flow.
>
> **Action for PM**: Before UAT sign-off, please confirm whether saved payment methods are required for launch. If yes, a Stripe SetupIntent flow must be built to allow customers to save cards for future use. The dead `#` link should be resolved before go-live.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Click **Payment Methods**.
3. Confirm a page loads (not a `#` anchor or a 404).
4. Add a Stripe test card (e.g., `4242 4242 4242 4242`) as a saved payment method.
5. Confirm the card appears in the list.
6. Remove the saved card and confirm it is deleted.

**Expected Result**:
- Payment Methods page loads and shows any saved cards.
- A new card can be added via the Stripe Payment Element (SetupIntent flow).
- Cards can be removed from the saved list.

**Pass Criteria** *(once feature is implemented)*:
- ✅ Payment Methods page is accessible from the account page.
- ✅ A new card can be saved successfully.
- ✅ A saved card can be removed.

**Edge Cases**:
- Adding a declined card → should show validation error from Stripe.
- Removing the only saved card → should not error.

---

### TC-FE-164: Data Export (GDPR) — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: No data export endpoint or link exists in the current build of the Customer Front-end. The account page (`/Account`) does not contain a "Download my data" or "Request data export" option.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team and legal/compliance team whether a self-service data export feature is required for launch under **GDPR Article 20 (Right to Data Portability)**. If it is required, the feature must be built and this test case updated before UAT runs. If data export is handled by the customer support team via a manual process, document that decision here and note the agreed fulfilment process and timeline.

**If and when this feature is built, the test should verify**:

**Given**: You are signed in as `customer.uat@wantfood.com`

**Steps**:
1. Navigate to `/Account`.
2. Find the **"Download my data"** or **"Request data export"** option.
3. Click it and follow any confirmation steps.
4. Note the result — either a download begins immediately or a confirmation message states when the export will be emailed.

**Expected Result**:
- User receives a downloadable file or email confirmation within the stated timeframe.
- Export contains personal data held by WantFood (name, email, order history, addresses).
- Export does **not** contain other users' data.

**Pass Criteria** *(once feature is implemented)*:
- ✅ Feature is accessible from the account page.
- ✅ Export is delivered (file or email) within the stated timeframe.
- ✅ Export contains the correct user's data only.

---

### TC-FE-165: Account Deletion — FEATURE TO BE CONFIRMED

> ⚠️ **FEATURE TO BE CONFIRMED**: No account deletion endpoint or link exists in the current build of the Customer Front-end. The account page (`/Account`) does not contain a "Delete my account" or "Close my account" option.
>
> **Action for PM**: Before UAT sign-off, please confirm with the development team and legal/compliance team whether customer self-service account deletion is required under **GDPR Article 17 (Right to Erasure)**. If deletion is handled via customer support (manual request-and-fulfilment process), document the agreed process and the expected response time — GDPR requires erasure within **30 days**. Do **not** use the shared `customer.uat@wantfood.com` test account for deletion tests.

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
- An explicit confirmation step is shown (users must not be able to accidentally delete their account).
- After confirming, the account is deleted and the user is signed out.
- Attempting to sign back in either fails or results in an "account not found" message in WantFood.
- All personal data associated with the account is removed from the platform (or scheduled for removal within the GDPR retention period).

**Pass Criteria** *(once feature is implemented)*:
- ✅ Explicit confirmation step is required before deletion.
- ✅ User is signed out after successful deletion.
- ✅ Re-login is blocked or results in "no account found" message.
- ✅ Personal data is confirmed removed (verify with development team or run a data check).

---

## Summary and Next Steps

You have now tested all major features of the **Customer Front-end**.

### What to do next:

1. **Log all bugs** found during testing on your Monday.com board under the "Customer Front-end" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the Order Saga** — this is the most important test phase as it tests the full end-to-end order flow across all portals:
	- **[Order Saga UAT](06-order-saga-uat.md)** ← **Start here next**
	- **[Automation and Jobs UAT](07-automation-jobs-uat.md)**
	- **[Sign-off and Completion](08-signoff.md)**

---

**Great work! Keep testing! 🚀**
