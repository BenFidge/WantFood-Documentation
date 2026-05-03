# Customer Front-End Guide

## 1. Overview
The Customer front-end is the public ordering experience for WantFood customers. It is used to discover nearby vendors, search for dishes and cuisines, browse vendor and dish details, build a basket, complete checkout, track orders, leave reviews, share orders, and maintain saved addresses.

### What this site is for
- discovering vendors and dishes near the customer
- browsing menus, dishes, and cuisine types
- building and maintaining a basket
- completing checkout and payment
- reviewing order history and tracking live orders
- adding tips, reviews, and order shares
- managing account details and saved addresses

### Access expectations
Most browsing is public. Checkout, order history, reviews, and account management generally require a signed-in customer account.

### How to use this guide
This guide focuses on customer-visible pages and the shared basket and checkout experience. Endpoint-driven behaviour is described only where it affects what the customer sees.

### Related documents
- Start with the [documentation index](index.md) for cross-site navigation.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when checkout, media updates, or later rating refreshes need background context.

### Typical prerequisites
- no sign-in is required for home, search, cuisine, vendor, or dish browsing
- a signed-in customer account is normally required for checkout, order history, reviews, tips, and saved addresses
- a non-empty basket is required before the checkout workflow can begin

### Navigation paths

| Workflow area | Typical entry path |
| --- | --- |
| Discovery | Home page -> Nearby vendors, Search page, Cuisine types page, Vendor page, or Dish page |
| Basket behaviour | Vendor page or Dish page -> shared basket experience |
| Checkout and payment | Shared basket experience -> Checkout page |
| Orders and post-order actions | Orders list or order history -> Order details, Order confirmation, Live order tracking, or Complete order or review prompt page |
| Account and saved addresses | Account overview page -> Addresses page |

---

## 2. Discovering vendors and dishes
These pages support location-aware browsing and search.

### Home page
**Purpose**  
Acts as the main landing page for discovery.

**Key actions**
- browse featured homepage content
- explore nearby vendors
- navigate into search, cuisine, vendor, and dish journeys

**Expected outcome**  
Customers can quickly begin finding somewhere to order from.

### Nearby vendors panel or flow
**Purpose**  
Shows nearby vendor choices from the home experience rather than as a separate standalone page.

**Expected outcome**  
Customers can move from the homepage into relevant local vendors.

### Search page
**Purpose**  
Lets customers search for vendors, dishes, or related discovery terms.

**Key actions**
- enter search terms
- review filtered results
- open a vendor or dish from the results

**Expected outcome**  
Customers can narrow a large catalogue down to relevant choices.

### Cuisine types page
**Purpose**  
Provides a cuisine-led browsing route.

**Expected outcome**  
Customers can explore vendors and dishes by cuisine preference.

### Vendor page
**Purpose**  
Shows the main storefront for a specific vendor.

**Key actions**
- review restaurant details
- browse the live menu
- open dish details
- start adding items to the basket

**Expected outcome**  
Customers can decide whether to order from the selected vendor.

### Dish page
**Purpose**  
Shows a deeper view of a specific dish.

**Expected outcome**  
Customers can confirm details before adding the dish to the basket.

### Privacy page
**Purpose**  
Displays privacy information for the customer site.

### Error page
**Purpose**  
Explains when a requested page or action cannot be completed.

### End-to-end workflow
1. Start on the home page.
2. Browse nearby vendors, search, or cuisine types.
3. Open a vendor page.
4. Review dish details and begin adding items to the basket.

---

## 3. Basket behaviour
The basket is best understood as a shared customer experience rather than a single routed page.

### Basket experience summary
**Purpose**  
Holds the items the customer plans to order and summarises their choices before checkout.

**Key actions**
- review basket contents
- update quantities
- remove items
- continue to checkout when ready

**Expected outcome**  
The customer can confirm that the basket matches their intended order.

### Vendor conflict handling in basket
**Purpose**  
Explains how the basket behaves when the customer attempts to add items from a different vendor.

**Expected outcome**  
The customer is prompted through the platform's single-vendor basket rule and can decide whether to replace the current basket context.

### Quantity update flow
**Purpose**  
Changes the quantity of items already in the basket.

**Expected outcome**  
Basket totals and contents reflect the new quantity.

### Remove item flow
**Purpose**  
Deletes an unwanted item from the basket.

**Expected outcome**  
The basket no longer contains the removed item.

### Menu context and scheduled order behaviour
**Purpose**  
Keeps the basket aligned to the active restaurant and any timing-related order conditions.

**Expected outcome**  
The customer sees a basket that remains valid for the current ordering context.

### End-to-end workflow
1. Add dishes from a vendor or dish page.
2. Review the shared basket summary.
3. Adjust quantities or remove items if needed.
4. Resolve any vendor conflict prompt if the customer changes restaurant.
5. Proceed to checkout.

---

## 4. Checkout and payment
These pages and behaviours turn a basket into an order.

### Checkout page
**Purpose**  
Collects the information required to place an order.

**Key actions**
- review basket contents and totals
- confirm delivery details
- review payment choice
- place the order

**Expected outcome**  
The customer has a final opportunity to validate the order before submission.

### Delivery quote creation flow
**Purpose**  
Calculates delivery pricing during checkout.

**Expected outcome**  
The customer sees the delivery cost that applies to the order.

### Place order flow
**Purpose**  
Submits the order for fulfilment.

**Expected outcome**  
The basket becomes an order and the customer is moved into the post-order experience.

### Payment intent polling flow for card payments
**Purpose**  
Supports card payment completion when checkout needs to wait for payment state updates.

**Expected outcome**  
The customer sees the order progress correctly into confirmation once payment completes.

### End-to-end workflow
1. Open Checkout from the basket.
2. Review delivery details, totals, and payment method.
3. Wait for the delivery quote if needed.
4. Place the order.
5. Let the payment flow finish and continue to order confirmation.

---

## 5. Orders and post-order experience
These pages cover confirmation, tracking, order history, and post-order actions.

### Orders list or order history
**Purpose**  
Shows past and current customer orders.

**Key actions**
- review previous orders
- open order details
- start reorder or share-related actions where available

**Expected outcome**  
Customers can revisit earlier activity and manage current orders.

### Order details
**Purpose**  
Displays the detail of one order.

**Key actions**
- review items, totals, and status
- cancel the order if allowed
- add a tip when supported

**Expected outcome**  
The customer can understand the full order record and available next actions.

### Order confirmation
**Purpose**  
Confirms that the order was placed successfully.

**Expected outcome**  
The customer has immediate reassurance that the checkout process succeeded.

### Live order tracking
**Purpose**  
Shows ongoing progress for an active order.

**Expected outcome**  
The customer can see where the order is in the fulfilment journey.

### Cancel order
**Purpose**  
Stops an order when platform rules still allow cancellation.

**Expected outcome**  
The customer sees the order move into a cancelled state.

### Add tip
**Purpose**  
Lets the customer add gratuity after order placement where supported.

**Expected outcome**  
The order total reflects the added tip.

### Complete order or review prompt page
**Purpose**  
Acts as the post-completion entry point for feedback collection.

**Expected outcome**  
The customer is prompted to leave a review once the order is finished.

### Submit branch review
**Purpose**  
Records feedback about the overall restaurant experience.

**Expected outcome**  
The vendor receives customer feedback and ratings can later be refreshed.

### Submit dish reviews
**Purpose**  
Records dish-level feedback from the completed order.

**Expected outcome**  
The platform stores more detailed customer sentiment about ordered items.

### Share order page
**Purpose**  
Lets the customer share order details or a related link from the post-order flow.

**Expected outcome**  
The order can be shared without the customer needing to recreate the context manually.

### Reorder flow
**Purpose**  
Starts a fresh basket using a previous order as the source.

**Expected outcome**  
The customer can quickly build a new order from a familiar purchase.

### End-to-end workflow
1. Finish checkout and review the confirmation page.
2. Track the order while it is active.
3. Open order details for further actions.
4. Add a tip, cancel, review, share, or reorder where supported.
5. Use the order history page for future reference.

---

## 6. Account and saved addresses
These pages help customers maintain account and delivery location details.

### Account overview page
**Purpose**  
Shows the main customer account summary.

**Expected outcome**  
The customer can navigate to address and account maintenance tasks.

### Addresses page
**Purpose**  
Lists saved delivery addresses.

**Key actions**
- add an address
- edit an address
- delete an address
- set a default address

**Expected outcome**  
The customer can maintain the delivery locations used during checkout.

### Add, edit, delete, and set default address flows
**Purpose**  
Support address maintenance directly from the addresses experience rather than separate standalone pages.

**Expected outcome**  
Saved addresses remain accurate and ready for future orders.

### End-to-end workflow
1. Open the account overview or addresses page.
2. Review saved addresses.
3. Add or update a location.
4. Remove obsolete addresses or set the preferred default.

---

## 7. Related automation and customer-visible async behaviour
Several visible customer flows rely on asynchronous processing.

### Important customer-visible async workflows
- homepage and campaign content may reflect centrally managed content publishing
- the basket experience maintains a single-vendor context when items are added across restaurants
- checkout can create delivery quotes asynchronously before final order placement
- card payments may use payment intent polling before confirmation is shown
- review submission can affect later vendor and dish ratings after scheduled or manual recalculation
- image-driven menu and content updates may appear after vendor or platform uploads finish processing

### Documentation note
Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the shared reference behind payment polling, asynchronous media updates, and later rating refreshes.

---

## 8. Screenshot candidates for first release

- home page with featured discovery content
- vendor page with live menu context
- dish page
- shared basket experience or basket canvas
- checkout page
- order confirmation or live tracking page
- complete order or review prompt page
- addresses page
