# Offers and Promotions Guide

This guide explains how to create, manage, and understand special offers and promotions on WantFood. It covers both the **Vendor Admin** perspective (vendors creating offers for their own restaurants) and the **System Admin** perspective (platform administrators creating platform-wide offers), as well as how customers interact with offers during checkout.

---

## Contents

1. [Overview — what offers do](#1-overview--what-offers-do)
2. [Offer types explained](#2-offer-types-explained)
3. [How customers see and use offers](#3-how-customers-see-and-use-offers)
4. [Free item offers — how they work end to end](#4-free-item-offers--how-they-work-end-to-end)
5. [Vendor Admin — managing your own offers](#5-vendor-admin--managing-your-own-offers)
6. [System Admin — managing platform offers](#6-system-admin--managing-platform-offers)
7. [Understanding stacking and priority](#7-understanding-stacking-and-priority)
8. [Offer status explained](#8-offer-status-explained)
9. [Offer analytics and usage tracking](#9-offer-analytics-and-usage-tracking)
10. [Common questions](#10-common-questions)

---

## 1. Overview — what offers do

Offers reduce what a customer pays. They can be applied automatically when conditions are met, or they can require a customer to enter a promo code at checkout. The discount is always calculated on the server at the point a quote is created — customers cannot enter their own amounts or bypass rules.

There are two sources of offers:

| Source | Who creates it | Who it applies to |
|--------|---------------|-------------------|
| **Vendor offer** | A restaurant owner or branch manager in Vendor Admin | Only orders from that vendor or branch |
| **Platform offer** | A WantFood administrator in System Admin | All vendors across the platform |

Offers are attached to a quote. When the customer submits their order, the platform re-validates that the offer is still active and eligible. If an offer expired between the quote and submission, the customer is told and the quote is refreshed before payment proceeds.

---

## 2. Offer types explained

### Percentage Discount
Reduces the order subtotal by a set percentage. An optional cap limits the maximum pounds saved (for example, "20% off, up to £10"). An optional minimum order value gates the offer. The percentage must be between 1 and 100.

**Example:** 20% off, minimum order £15, capped at £8.

### Fixed Amount Discount
Removes a flat pound amount from the subtotal. An optional minimum order value can be set. The discount can never exceed the subtotal.

**Example:** £5 off orders over £20.

### Free Delivery
Waives the delivery fee entirely. The subtotal is not affected. An optional minimum order value can require the customer to reach a spend threshold first.

**Example:** Free delivery on orders over £15.

### Promo Code
A code that a customer types at checkout. The code itself is a wrapper — behind it is one of the other discount types (percentage, fixed amount, or free delivery). Only one promo code can be applied per order by default.

**Example:** Enter SUMMER25 for 25% off.

### Buy One Get One Free (BOGOF)
The customer buys a qualifying item from a chosen category and receives a second item free (or at a reduced price). The "get" item is the cheapest eligible item in the get-category already in the basket. The vendor configures which categories qualify.

**Example:** Buy any pizza, get a garlic bread free.

### Spend and Save (Tiered Discount)
A set of spending tiers, each with its own percentage discount. The highest tier the customer's subtotal reaches is applied. Tiers must have strictly increasing minimum spend values.

**Example:** Spend £25 get 10% off; spend £40 get 20% off; spend £60 get 25% off.

### Free Item
Adds a specific dish to the customer's order at no cost when the order qualifies. The vendor picks the exact dish from their menu when creating the offer. See [section 4](#4-free-item-offers--how-they-work-end-to-end) for full detail on how this works at checkout.

**Example:** Free garlic bread with any order over £20.

### First Order Discount
A percentage discount applied automatically to a customer's very first order on the platform. The platform checks the customer's order history. Requires the customer to be signed in. Typically created as a platform offer.

**Example:** 30% off your first order (up to £10).

### Happy Hour
A time-based percentage discount active only on selected days and within a set time window. Days and times are configured when the offer is created.

**Example:** 15% off all orders Monday to Wednesday, 2 pm to 5 pm.

### Bundle / Meal Deal
A fixed price for a combination of items. The vendor defines groups (for example, Main, Side, Drink), the category each group draws from, and how many items a customer must pick from each group. The bundle price replaces the individual item prices.

**Example:** Any main + any side + any drink for £12.99.

---

## 3. How customers see and use offers

### On the vendor/restaurant page
Active offers are shown as small badges below the restaurant name and header. Badge text is generated automatically from the offer configuration (for example, "Free Delivery", "20% Off", "Happy Hour"). Customers do not need to do anything to see these — they are displayed whenever a qualifying offer is active.

### On the search and discovery page
Vendor cards show a small "OFFER" indicator when that restaurant has at least one active offer. Customers can filter search results to show only vendors with active offers using the Offers filter toggle.

### At checkout — automatic offers
Offers that do not require a code are evaluated automatically when the customer views their basket. The price breakdown shows each applied offer with a description and the saving amount. These apply without any action from the customer as long as the order meets the conditions (minimum spend, correct time of day, and so on).

### At checkout — promo codes
A "Have a promo code?" section appears in the basket. The customer:

1. Types their code into the input box.
2. Presses **Apply** (or hits Enter).
3. The platform validates the code immediately without a page reload.
4. If valid, the discount description and saving appear in the price breakdown, and the order total updates.
5. If invalid, an error message explains why (expired, wrong minimum, already used, and so on).

Only one promo code can be active at a time. Entering a new code replaces the previous one.

### Price breakdown
The basket always shows a full breakdown:

| Line | Notes |
|------|-------|
| Subtotal | Sum of item prices before any discount |
| Each applied offer | Description and saving shown in green (e.g., "-£4.00" or "FREE DELIVERY") |
| Delivery | Original delivery fee (shown struck-through if waived) |
| Delivery (free) | Shown in green when a free-delivery offer is active |
| Service fee | Calculated on the discounted subtotal |
| **Total** | Final amount charged |

---

## 4. Free item offers — how they work end to end

Free item offers are the offer type most likely to need explanation because the free item is not chosen by the customer — it is pre-configured by the vendor and added automatically. Here is exactly what happens.

### How the vendor sets it up

When creating a Free Item offer in Vendor Admin:

1. The vendor fills in the standard offer fields (name, dates, minimum order value, and so on).
2. In the **Free Item** section of the form, a dish picker lists all dishes on that vendor's menus.
3. The vendor selects exactly one dish. This is the item that will be added free of charge.
4. The vendor can also set a minimum order value — for example, the free item only applies when the subtotal reaches £20.

> **Important:** The free item is a single, fixed dish. There is no customer choice involved. If the vendor wants to offer a choice (for example, "any drink"), they should use a **Bundle / Meal Deal** offer instead, which lets customers pick from a category.

### What the customer sees

1. On the vendor page, the offer badge reads **"Free Item"** (or the vendor's custom offer name if set).
2. At checkout, when the minimum order value is met, the free item appears automatically in the **Applied Offers** section of the price breakdown, for example: *"Free Garlic Bread — Free Item offer applied"*.
3. The item itself is shown in the order summary as a line with £0.00 cost.
4. The customer does not need to add the dish to their basket themselves. It is injected into the order by the platform.

### If the free item is out of stock or unavailable

If the configured dish is marked as unavailable on the vendor's menu at the time of ordering, the free item offer will not apply and the customer will not see it. The vendor should ensure the free dish is always available during the offer period, or deactivate the offer when the dish is temporarily removed.

### No customer selection is required

Free item offers are fully automatic. The customer:

- does **not** choose which item they receive
- does **not** need to add anything to their basket
- does **not** enter a code (unless the vendor additionally gates it behind a promo code)

If the vendor wants to give the customer a choice of free item, the correct approach is to create a **Bundle / Meal Deal** offer where one of the bundle groups is the "free" selection, and set the bundle price to match the cost of the other (paid) items in the deal.

### Example walkthrough

A vendor creates: *"Free garlic bread with any order over £20, auto-applied."*

| Step | What happens |
|------|-------------|
| Customer browses the vendor page | Badge shows "Free Item" |
| Customer adds £18 of items to basket | Offer is not yet shown — minimum not met |
| Customer adds another item, subtotal reaches £22 | Offer appears in price breakdown: "Free Garlic Bread — Free Item offer applied" |
| Order summary | Garlic Bread line shows £0.00 |
| Customer removes an item, subtotal drops below £20 | Offer disappears from the breakdown |
| Customer submits order | Platform re-checks eligibility; garlic bread included in order at no cost |

---

## 5. Vendor Admin — managing your own offers

### Getting to the Offers area

From the Vendor Admin dashboard, select **Offers** from the sidebar. The list page shows all offers for your currently selected vendor or branch context.

If you manage multiple branches, use the branch switcher in the top navigation to change context:
- Select a specific branch to see only that branch's offers (and vendor-wide offers).
- Select **All branches** to see all offers across your restaurants.

### Understanding the list page

The list shows each offer with:

| Column | Meaning |
|--------|---------|
| Name | Your internal name for the offer |
| Type | The offer mechanism (Percentage Discount, Free Delivery, and so on) |
| Scope | Whether it applies to the whole vendor or a specific branch |
| Promo code | The code customers enter, if applicable |
| Dates | Start and expiry dates |
| Usage | How many times the offer has been used, and the cap if one is set (e.g., 12 / 100) |
| Status | Active, Scheduled, Expired, Disabled, or Usage Reached (see [section 8](#8-offer-status-explained)) |

You can filter the list by status, offer type, scope, branch, or by searching by name or promo code.

### Creating an offer

1. Press **Create offer**.
2. Select the **offer type** from the type picker at the top of the form. The form adapts to show only the fields relevant to that type.
3. Fill in the **common fields**:
   - **Name** — displayed to customers on badges and in the price breakdown.
   - **Description** — a sentence explaining the offer (optional but recommended).
   - **Internal notes** — private notes for your team; not shown to customers.
   - **Scope** — Vendor-wide (applies to all your branches) or a specific branch.
   - **Start date / time** — when the offer becomes active.
   - **Expiry date / time** — leave blank for a permanent offer.
   - **Minimum order value** — the subtotal the customer must reach.
   - **Maximum uses** — total uses across all customers, and/or per-customer limit.
   - **Auto-apply** — whether the offer applies automatically or requires a promo code.
   - **Stackable** — whether this offer can apply at the same time as other offers.
   - **Priority** — used when multiple non-stackable offers apply; higher number wins.
   - **Banner image** — optional promotional image shown on the vendor page.
4. Fill in the **type-specific fields** in the lower section (see offer type descriptions in [section 2](#2-offer-types-explained)).
5. Review the **live preview** in the right-hand sidebar to see how the badge and description will appear to customers.
6. Press **Save**.

> **Scope restriction:** Vendor Admin cannot create platform-wide offers. If you want a promotion to appear across all vendors, contact WantFood platform support.

### Editing an offer

1. Find the offer in the list and press **Edit**.
2. Make your changes. All fields are editable except the offer type (to change the type, clone the offer and create a new one).
3. Press **Save**. Changes take effect immediately for new quotes; in-flight quotes that were created before your change are not affected.

### Activating and deactivating an offer

- **Deactivate** stops the offer from being applied to new orders immediately. Use this to pause an offer without deleting it.
- **Activate** re-enables a deactivated offer.

Existing orders that already have the offer applied are not changed when you deactivate it.

### Cloning an offer

Press **Clone** on any existing offer to create a copy. The clone is created as inactive, with today as the start date, no expiry, and no promo code (codes must be unique). You are taken straight to the edit form to adjust the clone before activating it.

### Promo code availability check

When you type a promo code into the form, the page checks automatically whether that code is already in use by another of your offers or a platform offer. A tick confirms the code is available; a warning tells you it is already taken.

Promo codes must be uppercase letters, digits, dashes, or underscores (for example, `SUMMER25` or `FREE-DRINK`).

---

## 6. System Admin — managing platform offers

Platform offers apply to all vendors on the platform. They are created and managed in System Admin under **Platform Offers**.

### Getting to the Platform Offers area

From the System Admin dashboard, select **Platform Offers** from the navigation. The list and creation/editing workflow is the same as Vendor Admin offers, with the following differences:

- The **Scope** is always Platform; there is no vendor or branch selection.
- Platform offers can apply a discount at any vendor without the vendor needing to create anything.
- **Commission impact:** When a platform offer reduces the order subtotal, vendor commission is still calculated on the original subtotal (before the platform-funded discount). This protects vendors from losing commission on promotions they did not initiate. See [section 10](#10-common-questions) for more detail.

### Typical platform offer scenarios

| Scenario | Recommended type |
|----------|-----------------|
| Welcome discount for first-time customers | First Order Discount, auto-applied, platform scope |
| Seasonal campaign code | Promo Code wrapping a Percentage Discount, platform scope |
| Bank holiday free delivery | Free Delivery, auto-applied, specific date range |
| Partner promotion code | Promo Code wrapping a Fixed Amount Discount, per-customer limit of 1 |

### Managing analytics

Each platform offer has an analytics page showing total uses, unique customers, total discount given, average order value with the offer, and a daily usage chart. This is accessible via the **View** action on the offer list.

---

## 7. Understanding stacking and priority

By default, customers can have multiple automatic offers (for example, a vendor-set Happy Hour and a platform free-delivery offer) active on the same order. However, promo codes are in a separate stacking group and only one can be active at a time.

### The stacking group

Offers that share the same **stacking group** are mutually exclusive — only the best one (by priority, then by discount amount) applies. The default stacking group is `promo-codes`, which is why entering a second code replaces the first.

Automatic offers (free delivery, happy hour, percentage discounts) do not need to be in a stacking group unless you specifically want them to be mutually exclusive.

### Priority

When two non-stackable offers are in the same stacking group and both are eligible, the one with the higher **Priority** number wins. If priorities are equal, the larger discount wins. Set a higher priority number for offers you always want to take precedence.

### Total discount cap

The total discount applied to an order can never exceed the order subtotal. The delivery fee can be waived separately.

---

## 8. Offer status explained

| Status | Meaning |
|--------|---------|
| **Active** | The offer is live and being applied to eligible orders. |
| **Scheduled** | The offer has been created and will become active automatically when the start date/time is reached. |
| **Expired** | The expiry date has passed. The offer is no longer applied to new orders. Historical usage data is preserved. |
| **Disabled** | The offer has been manually deactivated. It can be reactivated at any time. |
| **Usage Reached** | The total usage cap has been hit. The offer will not apply to further orders even if it is still within its date range. |

---

## 9. Offer analytics and usage tracking

Every time an offer is applied to a completed order, a usage record is created. This record includes the customer, the order, the discount amount, and the timestamp. This data is visible on the **Details** page for each offer.

### Stats shown on the detail page

| Stat | Meaning |
|------|---------|
| Total uses | Number of times the offer has been applied to a completed order |
| Unique customers | Number of distinct customers who have used the offer |
| Total discount given | Sum of all discount amounts across all usages |
| Average order value with offer | Mean order subtotal on orders where this offer applied |
| Average discount per order | Mean discount amount across all usages |

The daily usage chart shows uses and total discount per day, useful for identifying peak periods and evaluating whether a time-limited campaign drove order volume.

The recent usages table lists the last 50 redemptions with the date, a masked customer reference, the order number, and the discount amount. Each order number links to the order detail page.

---

## 10. Common questions

### Can a customer apply a promo code and also get a free delivery offer?

Yes, if both offers are stackable and in different stacking groups. By default, a vendor's free delivery auto-applied offer and a platform promo code can apply together. The customer will see both in the price breakdown.

### What happens if a promo code expires while a customer is at checkout?

The quote contains the discount at the time it was generated. When the customer presses **Place Order**, the platform re-validates all applied offers. If the code has since expired, the quote is rejected and the customer is shown a message explaining the offer is no longer valid. The order is not placed until the customer confirms the updated (non-discounted) price.

### Can vendors see platform offers applied to their orders?

Yes. The order record includes a breakdown of all applied offers, including whether they are vendor-funded or platform-funded. This affects commission: vendor-funded discounts reduce the commission base; platform-funded discounts do not.

### How do I offer a customer a choice of free item?

The **Free Item** offer type delivers one pre-set dish with no customer choice. If you want to offer a choice, create a **Bundle / Meal Deal** instead. Add the paid items as one or more bundle groups, then add a "Free item" group with the category the customer can choose from, and set the bundle price to the cost of the paid items only (effectively making the chosen item free).

### Why is the free item not appearing in the customer's basket?

Check the following:

1. The offer's start date has passed and the expiry has not been reached.
2. The offer is set to **Active** (not Disabled or Scheduled).
3. The customer's subtotal meets the minimum order value, if one is set.
4. The configured free dish is currently marked as available on the menu.
5. If the offer requires a promo code, the customer has entered and applied it.

### Can I limit a promo code to first-time customers only?

Not directly via the promo code type. To achieve this, use a **First Order Discount** offer type, which automatically checks whether the customer has any previous orders. If you also want it gated behind a code, set **Auto-apply** to off and enter a promo code — the offer will only activate when the code is entered and the first-order check passes.

### What does "per-customer limit" mean?

Setting **Max uses per customer** to 1 means each individual customer account can only use that offer once. The global **Max uses total** is the overall cap across all customers. Both limits can be set independently.

### Can I run the same promo code across multiple branches?

A promo code must be unique across the platform. If you have multiple branches and want each to have its own offer, create separate offers with different codes. Alternatively, create a single vendor-wide offer (Scope = Vendor) and it will apply to orders from any of your branches.

### How is commission affected by discounts?

| Offer funded by | Commission is calculated on |
|-----------------|-----------------------------|
| Vendor (vendor or branch scope) | Discounted subtotal |
| Platform (platform scope) | Original subtotal (before discount) |

This means vendors are not penalised on their commission for platform-initiated promotions such as welcome offers or seasonal campaigns.
