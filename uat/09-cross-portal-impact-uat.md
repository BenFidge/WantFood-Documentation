# Cross-Portal Impact UAT — Test Scripts

## Purpose

This document tests the **consequences** of admin changes. When a platform administrator changes something in System Admin or Vendor Admin, that change must ripple out correctly to every other surface a customer, driver, or vendor user might see.

The per-portal documents (02-system-admin-uat.md and 03-vendor-admin-uat.md) cover the admin-side click-paths — how to make the change itself. **This document tests what happens next:** does the customer front-end update? Does the basket behave correctly? Does the right email arrive? Does the driver portal reflect the change?

These are **paired tests** — you will need the admin portal open in one window and the customer front-end open in another at the same time. For many test cases you will need three windows running simultaneously.

> **Why cross-portal testing matters**: A change that looks correct in Vendor Admin can still be broken if the customer never sees it, or if the basket silently serves stale data. Silent cross-portal failures are some of the hardest bugs to find — they are exactly what this document is designed to catch.

---

## Prerequisites

### Test Accounts

You need all four test accounts active and ready before starting.

| Account | Email | Where you log in |
|---------|-------|-----------------|
| **System Admin** | `admin.uat@wantfood.com` | System Admin portal |
| **Vendor Admin** | `vendor.uat@wantfood.com` | Vendor Admin portal |
| **Customer** | `customer.uat@wantfood.com` | Customer front-end (use incognito) |
| **Driver** | `driver.uat@wantfood.com` | Driver portal |

> All logins use **Microsoft Entra ID** (the same corporate sign-in screen). There is no local username/password reset flow — if your account is locked or your password has expired, contact your project manager.

### Browser Setup

**Recommended: three browser windows side-by-side**, not three tabs. Tabs make it too easy to miss the moment a change appears.

| Window | Contents |
|--------|----------|
| Window 1 | System Admin or Vendor Admin (whichever you are changing) |
| Window 2 | Customer front-end in **incognito / private browsing** |
| Window 3 | Driver portal (for order-handover tests) or a second admin portal |

### Monday Board

Bug reports for this document go into the **Cross-Portal Impact** group on the Monday.com UAT board. See [01-monday-setup.md](01-monday-setup.md) for how to log bugs.

### Test Data

Before starting, confirm the following exists in the UAT environment:

| Item | Example value | Where to check |
|------|--------------|----------------|
| Active vendor with published menu | "Pizza Roma" | Customer front-end home page |
| Second active vendor | "Sushi Spot" | Customer front-end search |
| Customer with items already in basket | 1× Margherita in basket | Customer front-end basket |
| Active platform offer | "SUMMER10" — 10% off | Customer front-end basket / checkout |
| Active vendor offer / promo code | "NEWCUSTOMER" | Customer front-end checkout |
| Published hero slide | Visible on home page | Customer front-end `/` |
| Published Support Local entry | Visible on home page | Customer front-end `/` |

---

## Propagation Cheat-Sheet

> **Link this section from every TC.** When a change does not appear immediately, work through these steps in order before logging a bug.

Most admin changes appear on the customer front-end within **approximately 30 seconds**. The system keeps a short-lived cache to keep the website fast, so there is always a small delay between pressing Save in the admin portal and the customer seeing the result.

**Propagation checklist — always try these before logging a bug:**

1. **Wait 30 seconds**, then hard-refresh the customer front-end page. On Windows press **Ctrl + F5**; on Mac press **Cmd + Shift + R**. A normal refresh (F5) may serve the browser's own cached copy and hide the change.
2. If still not visible, log in to System Admin and go to **Platform Tools** → click **Rebuild Caches** → wait 30 seconds → hard-refresh the customer front-end.
3. For changes that affect **search results or vendor discovery** (vendor name, cuisine, active status, dish keywords), also run **Platform Tools → Reindex Vendors** → wait 30 seconds → repeat your search.
4. If the change is still not visible after completing steps 1–3, **log a bug** with severity **Medium**, describe the propagation steps you tried, and attach screenshots of both the admin "saved" confirmation and the customer front-end showing the old value.

**What "Rebuild Caches" and "Reindex Vendors" do (plain English):** Rebuild Caches clears all the quick-lookup data the website stores in memory to serve pages fast. Reindex Vendors tells the search engine to re-read every vendor's details from scratch. Both tools are safe to run in UAT at any time. See TC-XP-027 for a dedicated test of the Tools page itself.

**If an email does not arrive within five minutes:** check the Junk/Spam folder first. All WantFood system emails come from `noreply@wantfood.com`. If you have checked Junk and the email is still missing after ten minutes, log a bug with severity **High** (missing transactional email is a serious failure).

**Back-end logs:** A small number of test cases in this document refer to "back-end logs" or "audit records". These are not visible to testers in the UAT environment — they exist in the engineering infrastructure only. If a test step says "back-end log only", you cannot verify it directly; instead, verify the observable behaviour (the change happened, the correct data is visible). If observable behaviour is wrong but there is no error on screen, note in your bug report that back-end logging may also be affected.

---

## Impact Matrix

The table below shows every admin change covered in this document, the customer-facing surfaces affected, the test case that covers it, and how serious a failure would be.

| # | Admin change (where) | Front-end / cross-portal surfaces affected | TC ID | Severity if broken |
|---|---------------------|--------------------------------------------|-------|-------------------|
| 1 | Vendor activate / deactivate (SA `/Admin/Vendors`) | Home, search, cuisine page, direct vendor URL, in-flight orders, orphaned basket items | TC-XP-001 | Critical |
| 2 | Edit vendor display details — name, description, logo, banner (VA `/Admin/VendorManage`) | Vendor page, search results, vendor cards on home | TC-XP-002 | High |
| 3 | Approve / reject vendor application (SA `/Admin/Applications`) | Applicant email, first vendor-admin login, vendor appearance on FE after onboarding | TC-XP-003 | Critical |
| 4 | Publish / unpublish menu (VA `/Admin/MenuBuilder`) | Vendor page menu list, dish search, deep-linked dish URLs, basket items | TC-XP-004 | Critical |
| 5 | Add / edit / delete dish (VA `/Admin/MenuBuilder`) | Vendor page, dish detail page, dish search, basket | TC-XP-005 | High |
| 6 | Dish availability toggle / variant availability (VA) | Real-time hide on FE, basket during mid-checkout | TC-XP-006 | High |
| 7 | Reorder dishes / categories (VA) | Order on customer vendor page | TC-XP-007 | Medium |
| 8 | Upload dish image (VA) | FE dish image, vendor card, thumbnail processing, alt text | TC-XP-008 | Medium |
| 9 | Hero slide publish / unpublish / reorder / delete (SA `/Admin/ContentHeroSlides`) | Homepage carousel order, mobile vs web, CTA link target, regional targeting | TC-XP-009 | High |
| 10 | Support Local slide create / edit / publish / delete (SA `/Admin/ContentSupportLocal`) | Homepage Support Local section visibility and order | TC-XP-010 | Medium |
| 11 | Cuisine taxonomy reorder mobile vs web (SA `/Admin/CuisineTypes`) | Cuisine ribbon order on mobile FE vs web FE separately | TC-XP-011 | Medium |
| 12 | Region create / edit / delete (SA `/Admin/ContentRegions`) | Hero slide regional targeting, location-based discovery | TC-XP-012 | High |
| 13 | Bad words add / edit / delete (SA `/Admin/Settings`) | Review submission rejection on FE; historical reviews unaffected | TC-XP-013 | Medium |
| 14 | Dish keywords add / delete (SA `/Admin/DishKeywords`) | Search results matching synonyms on FE | TC-XP-014 | Medium |
| 15 | Platform offer activate / pause / edit (SA `/Admin/Offers`) | Basket banner, checkout discount line, offer badges, eligibility rules | TC-XP-015 | High |
| 16 | Vendor offer create / pause / clone / delete (VA `/Admin/Offers`) | Promo code validation at checkout, basket display, stacking with platform offers | TC-XP-016 | High |
| 17 | Commission tier create / edit / delete (SA `/Admin/CommissionTiers`) | Vendor reassignment, invoicing maths, tier visibility | TC-XP-017 | High |
| 18 | Commission config change + manual recalculation (SA `/Admin/Commissions`) | Vendors reassigned, downstream invoice maths | TC-XP-018 | High |
| 19 | Platform Payment Settings — add / remove Stripe & WorldPay (SA `/Admin/PaymentSettings`) | Card payment at checkout, 3D Secure, error UX when removed mid-session | TC-XP-019 | Critical |
| 20 | Vendor Stripe Connect connect / disconnect (VA `/Admin/PaymentMethods`) | Customer checkout card option for that vendor; onboarding banner in VA | TC-XP-020 | Critical |
| 21 | Enable / disable cash at vendor (VA `/Admin/PaymentMethods`) | Cash option at customer checkout for that vendor only | TC-XP-021 | High |
| 22 | Delivery cost update — distance-based tiers (VA `/Admin/DeliveryCosts`) | Customer delivery quote, basket totals, order confirmation totals | TC-XP-022 | High |
| 23 | Vendor trading hours / opening times edit (VA `/Admin/VendorManage` → Branch tab) | "Closed now" indicator, basket/checkout block, scheduled order slots | TC-XP-023 | High |
| 24 | Vendor scheduled-orders settings (VA `/Admin/VendorManage` → AcceptsScheduledOrders) | Schedule picker enabled/disabled, furthest date, slot granularity | TC-XP-024 | High |
| 25 | Vendor user transfer / delete (SA `/Admin/VendorUsers`) | Login behaviour for affected user; vendor admin access removed | TC-XP-025 | High |
| 26 | Admin user add / delete (SA `/Admin/Users`) | New admin login works; deleted admin blocked; role scope | TC-XP-026 | High |
| 27 | Reindex Vendors / Rebuild Caches / Import ONS Postcodes / Integrity Check (SA `/Admin/Tools`) | Observable customer FE change after each tool | TC-XP-027 | High |
| 28 | Review moderation — flag / resolve + manual rating recalc (SA `/Admin/Reviews`) | Review hidden/restored on vendor page; vendor and dish ratings | TC-XP-028 | High |
| 29 | Content asset library upload / update / delete (SA `/Admin/ContentAssetLibrary`) | Assets in hero slides and Support Local; global hero carousel CSS/JS | TC-XP-029 | High |
| 30 | Driver invite / remove (VA `/Admin/Drivers`) | Driver portal access, order assignment availability, mid-flight handover | TC-XP-030 | High |

---

## Test Cases

---

### TC-XP-001: Vendor Activate / Deactivate — Customer Front-end Visibility and Basket Orphaning

**Goal**: Confirm that deactivating a vendor immediately removes it from all customer-facing discovery surfaces and that any basket items belonging to that vendor are handled gracefully.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-033 (Deactivate Vendor), TC-SA-034 (Reactivate Vendor) in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, open the home page (`/`). Locate the vendor card for **"Pizza Roma"** in the discovery grid.
2. **Take a screenshot** of the home page showing "Pizza Roma" in the results.
3. Note in your bug tracker: *"Before: Pizza Roma visible on home page, position [N] in grid."*
4. Add **1× Margherita** from "Pizza Roma" to the basket so you have an active basket item.
5. Navigate to the search page (`/search?q=pizza`) and confirm "Pizza Roma" appears in results. Screenshot.
6. Navigate to the "Italian" cuisine page (`/cuisine/italian`) and confirm "Pizza Roma" is listed. Screenshot.
7. Copy the direct vendor URL (e.g. `/vendor/pizza-roma`) and keep it in a browser tab.

#### Change — perform the admin action

1. Switch to **System Admin**.
2. Go to **Vendors** → find **"Pizza Roma"** → click **Edit** (or click the vendor name to open its detail page).
3. Locate the **Active** toggle (or **Status** field).
4. Switch the vendor to **Inactive** / deactivated.
5. Click **Save**. Confirm the success toast or confirmation message appears.
6. **Take a screenshot** of the admin vendor record showing "Inactive" status.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet) before logging any failures.

1. Wait 30 seconds.
2. In the customer window, hard-refresh the home page (Ctrl+F5 / Cmd+Shift+R).
3. If Pizza Roma is still visible, run **Platform Tools → Rebuild Caches**, then **Reindex Vendors**, and wait another 30 seconds.

#### Verify After

- ☐ **Home page** (`/`): "Pizza Roma" card is **no longer visible** in the discovery grid.
- ☐ **Search results** (`/search?q=pizza`): "Pizza Roma" does **not** appear in results.
- ☐ **Cuisine page** (`/cuisine/italian`): "Pizza Roma" is **not listed**.
- ☐ **Direct vendor URL** (`/vendor/pizza-roma`): Shows a "Restaurant not available" message or a 404-style page — **not** the vendor's menu. Screenshot this page.
- ☐ **Basket**: The Margherita item that was added earlier either shows a clear warning (e.g. "This restaurant is no longer available") or has been removed from the basket. It should **not** silently remain in the basket showing the old price without any indication. Screenshot the basket state.
- ☐ **Checkout** (if you attempt checkout with the orphaned basket item): The system should **block checkout** or re-validate the basket — you should not be able to complete a payment for a deactivated vendor's items.
- ☐ **In-flight orders** (any active order already placed with Pizza Roma before deactivation): These should **not** be cancelled automatically by the deactivation — the vendor admin should still be able to manage them. If you have an in-flight test order, check the Vendor Admin order kanban to confirm it is still visible.

#### Reactivation check

1. Switch back to System Admin and **reactivate** Pizza Roma.
2. Wait 30 seconds and hard-refresh the customer home page.
3. Confirm Pizza Roma is **visible again** in all surfaces listed above.
4. **Take a screenshot** confirming it reappears.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all "Verify After" boxes match expected outcomes AND the vendor reappears correctly after reactivation.
- ❌ **Fail** if Pizza Roma remains visible to customers after deactivation, or if checkout succeeds with an orphaned basket item.
- 🐛 **Bug-anyway** if the basket warning exists but is unclear or easy to miss. Log as **Medium** severity.

#### Edge cases to also try

- **Customer mid-checkout when vendor is deactivated** — start checkout on Pizza Roma, then deactivate the vendor in a second window. On the next checkout page advance, the system should detect the invalid basket state and block completion.
- **Customer with a saved "favourite" vendor that is then deactivated** — the favourites list (if it exists on the FE) should not crash; it should show the vendor as unavailable.
- **Vendor deactivated while a driver is mid-delivery** — the driver portal should still show the active delivery; this is a handover that must complete regardless of vendor status.

#### Common pitfalls

- *"Pizza Roma is still showing"* → almost always a cache issue. Follow the full propagation checklist before logging.
- *"The basket didn't warn me"* → this may be intentional (warning only shown at checkout) or a bug; check whether the checkout page shows a validation error. If there is no warning at all, log as **High**.
- *"The direct vendor URL showed a blank page"* → a blank page is a bug; the expected behaviour is a clear "not available" message.

---

### TC-XP-002: Edit Vendor Display Details — Propagation to Customer Front-end

**Goal**: Confirm that changes to a vendor's name, description, logo, and banner image appear correctly on the customer front-end vendor page, search results, and vendor cards.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-032 (Edit Vendor Details via System Admin), TC-VA-021 (Edit Vendor Profile) in 02/03-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, navigate to the **"Sushi Spot"** vendor page (e.g. `/vendor/sushi-spot`).
2. **Take a screenshot** of the full vendor page header: name, description text, logo, and banner image.
3. Navigate to `/search?q=sushi` and screenshot the vendor card showing the current name and logo.
4. Note in your bug tracker: *"Before: Sushi Spot — name, description, logo, and banner as shown in screenshot."*

#### Change — perform the admin action

1. Switch to **Vendor Admin**, logged in as the Sushi Spot vendor user.
2. Go to **Restaurant Management** → **Edit Restaurant** (or **VendorManage**).
3. Make all four of the following changes:
   - **Name**: change from "Sushi Spot" to **"Sushi Spot — City Kitchen"**
   - **Description**: append **" Now serving our summer menu!"** to the existing description
   - **Logo**: upload a new test logo image (any small square PNG will do)
   - **Banner**: upload a new test banner image (any wide PNG will do)
4. Click **Save** / **Update**. Confirm the success toast.
5. **Take a screenshot** of the Vendor Admin profile page showing the updated values.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

1. Wait 30 seconds → hard-refresh the customer vendor page.
2. If the name is unchanged, run **Rebuild Caches** then **Reindex Vendors**.

#### Verify After

- ☐ **Vendor page** (`/vendor/sushi-spot` or the updated slug): Name now reads **"Sushi Spot — City Kitchen"**. Description includes the new sentence. New logo is displayed (no broken image icon). New banner image is shown across the header. Screenshot.
- ☐ **Search results** (`/search?q=sushi`): The vendor card shows the **updated name** "Sushi Spot — City Kitchen" and the **new logo thumbnail**. Screenshot.
- ☐ **Home page vendor card** (if Sushi Spot appears in discovery): Updated name and logo visible. Screenshot.
- ☐ **Cuisine page** for Sushi Spot's cuisine: Updated name and logo visible in the list.
- ☐ **Basket** (if customer had Sushi Spot items): Basket shows updated vendor name — it should not retain the old name.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all display fields are updated on all four surfaces within the propagation window.
- ❌ **Fail** if the old name or logo appears on any surface after the full propagation checklist has been completed.
- 🐛 **Bug-anyway** if the logo or banner is blurry, incorrectly cropped, or takes more than 60 seconds to appear. Log as **Low** or **Medium** depending on severity.

#### Edge cases to also try

- **Very long vendor name** (over 50 characters) — does the vendor card truncate gracefully, or does text overflow the card boundary?
- **Logo with a transparent background** — does it display correctly on both light and dark card backgrounds?
- **Revert the name** back to "Sushi Spot" at the end of this TC so other test cases that rely on this vendor name continue to work.

#### Common pitfalls

- *"The logo updated but the name didn't"* → partial propagation. Run the full propagation checklist. If it persists after Rebuild Caches + Reindex, log as **High**.
- *"The banner is showing the old image"* → check that the new upload completed (look for a progress bar in Vendor Admin); some large images can fail silently if the upload timed out.

---

### TC-XP-003: Approve / Reject Vendor Application — Email and First Login

**Goal**: Confirm that approving a vendor application sends the correct onboarding email, that the first Vendor Admin login experience is correct, and that the vendor only appears publicly on the customer front-end once onboarding is fully complete.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ A separate email inbox for the applicant (use a test email address you can check)
- ☐ Vendor Admin (log in as the newly approved vendor after email arrives)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-022 (Approve Application), TC-SA-023 (Reject Application) in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. Ensure a **pending vendor application** exists for a test restaurant, e.g. **"The Burger Lab"**, submitted from a test email address you can access.
2. As **the customer**, do a search for "Burger Lab" (`/search?q=burger+lab`). Confirm it does **not** appear.
3. **Take a screenshot** of the empty search results.

#### Change (Approve path) — perform the admin action

1. Switch to **System Admin**.
2. Go to **Applications** → find **"The Burger Lab"** application.
3. Click to open it and review the details.
4. Click **Approve** and confirm when prompted.
5. Confirm the success message appears in System Admin.
6. **Take a screenshot** of the application showing "Approved" status.

#### Propagate

1. Wait up to **five minutes** for the onboarding email to arrive at the applicant's test email address. (Emails may take longer than the usual 30-second cache; this is expected.)
2. Check the Junk/Spam folder if the email does not arrive in the main inbox.

#### Verify After

- ☐ **Applicant email**: An onboarding invitation email has arrived. It contains the vendor name, a clear welcome message, and an **invitation link** (a URL). The sender is `noreply@wantfood.com`. Screenshot the email.
- ☐ **Invitation link works**: Click the link in the email. It opens the Vendor Admin **Onboarding Landing Page** — not a 404, not a generic login page.
- ☐ **First Vendor Admin login (Entra ID)**: Completing the Entra ID sign-in flow takes the new vendor user to the **Vendor Admin Onboarding Welcome page**, not a blank screen or error.
- ☐ **Customer front-end (during onboarding)**: While The Burger Lab is approved but onboarding is **not yet complete**, search for it on the customer front-end. It should **not** be discoverable yet. Screenshot the empty results.
- ☐ **Customer front-end (after onboarding complete)**: After the vendor completes onboarding and publishes their menu, they **should** appear in search and on the home page. (This full flow is covered in TC-VA-003; reference it here rather than repeating.)

#### Reject path (run as a separate sub-test)

1. Repeat the **Before** steps with a second test application, e.g. **"Bad Curry House"**.
2. In System Admin, click **Reject** (with a rejection reason if the field is available).
3. Verify: A **rejection email** arrives at the applicant's inbox explaining the decision. The application status shows "Rejected". The vendor does **not** appear anywhere on the customer front-end.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the onboarding email arrives within five minutes, the link works, and the vendor is not discoverable before onboarding is complete.
- ❌ **Fail** if the onboarding email never arrives, the invitation link is broken, or the vendor appears publicly before onboarding completes.
- 🐛 **Bug-anyway** if the email content is poorly worded, has a broken layout, or if the onboarding page is visually confusing. Log as **Low**.

#### Edge cases to also try

- **Clicking the invitation link a second time** (after onboarding is complete) — should redirect to the login page, not show an error.
- **Expired invitation link** — if the link is old (e.g. more than 7 days), it should show a clear "link expired, please contact support" message, not a crash.

#### Common pitfalls

- *"The email arrived but the link takes me to a blank page"* → log as **Critical** — the onboarding flow is broken.
- *"I can see the vendor on the customer FE before onboarding is finished"* → log as **High** — premature vendor visibility.

---

### TC-XP-004: Publish / Unpublish Menu — Customer Menu Visibility and Basket Handling

**Goal**: Confirm that unpublishing a menu removes it from the customer front-end and that basket items from that menu are handled gracefully.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-043 (Publish Menu), TC-VA-044 (Unpublish Menu) in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, navigate to the **"Pizza Roma"** vendor page.
2. Confirm the menu is visible: categories and dishes are listed.
3. Add **1× Margherita** and **1× Garlic Bread** to the basket.
4. **Take a screenshot** of the vendor page with the menu and a screenshot of the basket.
5. Note the URL of a specific dish (e.g. `/vendor/pizza-roma/dish/margherita`).

#### Change — perform the admin action

1. Switch to **Vendor Admin** (logged in as the Pizza Roma vendor).
2. Go to **Menu Builder** (`/Admin/MenuBuilder`).
3. Find the published menu (e.g. "Pizza Roma — Main Menu").
4. Click **Unpublish** (or toggle the menu status to "Draft" / "Unpublished").
5. Confirm when prompted. Confirm the success toast.
6. **Take a screenshot** of the Menu Builder showing the menu as "Draft" or "Unpublished".

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

1. Wait 30 seconds → hard-refresh the Pizza Roma vendor page.
2. If menu is still visible, run **Rebuild Caches** and retry.

#### Verify After

- ☐ **Vendor page** (`/vendor/pizza-roma`): The menu list is **empty** or replaced by a message such as "No menu available right now". The vendor page itself should still load — it should not show a 404. Screenshot.
- ☐ **Dish search** (`/search?q=margherita`): The Margherita dish from Pizza Roma no longer appears in search results. (Dishes from other vendors with the same name may still appear — that is correct.)
- ☐ **Direct dish URL** (`/vendor/pizza-roma/dish/margherita`): Should show a "dish not available" message or redirect to the vendor page — **not** a 404 crash page, and not the dish detail still showing as if it were available.
- ☐ **Basket**: The Margherita and Garlic Bread items in the basket either display a clear warning ("This item is no longer available") or are removed. They must **not** silently remain in the basket as if they are orderable. Screenshot the basket state.
- ☐ **Checkout** (attempt with the affected basket): Checkout should **block** or re-validate, preventing payment for items from an unpublished menu.

#### Republish check

1. Switch back to Vendor Admin and **re-publish** the menu.
2. Wait 30 seconds and hard-refresh.
3. Confirm the menu reappears on the vendor page and dishes appear in search.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the menu disappears from all surfaces and basket is handled with a clear warning.
- ❌ **Fail** if the menu remains visible after full propagation, or if checkout succeeds with unpublished menu items.
- 🐛 **Bug-anyway** if the dish detail page shows a 404 error (rather than a graceful "not available" message). Log as **Medium**.

#### Edge cases to also try

- **Customer on the dish detail page at the moment of unpublish** — refresh the page; it should not show a JavaScript error.
- **Multiple menus** — if a vendor has one published and one draft menu, unpublishing the first should not affect the second.

#### Common pitfalls

- *"The basket item didn't clear"* → this may be by design (warn at checkout only). The key question is whether checkout can be completed — if it can, that is a **High** bug.

---

### TC-XP-005: Add / Edit / Delete Dish — Propagation to Vendor Page, Search, and Basket

**Goal**: Confirm that adding, editing, and deleting dishes in Vendor Admin correctly updates the vendor page, search results, dish detail pages, and any customer baskets holding those dishes.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-036 (Add Dish), TC-VA-037 (Edit Dish Name/Price), TC-VA-038 (Edit Dish Description/Image), TC-VA-039 (Delete Dish) in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, navigate to the **"Pizza Roma"** vendor page.
2. **Take a screenshot** showing the current dish list in the "Starters" category.
3. Note: there should be no dish called "Truffle Fries" yet.

#### Change A — add a new dish

1. In **Vendor Admin**, go to **Menu Builder** → **Pizza Roma — Main Menu** → **Starters** category.
2. Click **Add Dish** and fill in:
   - **Name**: `Truffle Fries`
   - **Description**: `Golden fries tossed in truffle oil and parmesan.`
   - **Price**: `£6.50`
   - **Calories**: `420`
   - **Available**: toggled ON
3. Click **Save**.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet). Wait 30 seconds then hard-refresh.

#### Verify After (Add)

- ☐ **Vendor page** → Starters category: "Truffle Fries" dish card is visible with the correct name, description, price (£6.50), and calorie count (420 kcal). Screenshot.
- ☐ **Search results** (`/search?q=truffle+fries`): The dish appears in results linked to Pizza Roma. Screenshot.
- ☐ **Dish detail page** (click the dish): Correct name, description, price, calories shown.

#### Change B — edit the dish

1. Back in **Vendor Admin**, edit the "Truffle Fries" dish:
   - Change price from £6.50 to **£7.00**
   - Change description to **"Crispy golden fries with truffle oil, parmesan, and fresh chives."**
2. Save.

#### Verify After (Edit)

- ☐ **Vendor page**: Truffle Fries now shows **£7.00** and the updated description.
- ☐ **Basket** (if a customer had added Truffle Fries at £6.50 before the price change): The basket **should update** the price to £7.00, or warn the customer that the price has changed. The customer should **not** be able to check out at the old price. Screenshot the basket showing the updated price.
- ☐ **Dish detail page**: Reflects new price and description.

#### Change C — delete the dish

1. In **Vendor Admin**, delete the "Truffle Fries" dish.
2. Confirm the deletion prompt.

#### Verify After (Delete)

- ☐ **Vendor page**: "Truffle Fries" is **no longer visible**.
- ☐ **Search results** (`/search?q=truffle+fries`): No result for Truffle Fries from Pizza Roma.
- ☐ **Direct dish URL** (if you saved it earlier): Shows a "dish not found" or "not available" message — not a crash.
- ☐ **Basket** (if a customer had the dish in their basket): Basket shows a warning or removes the dish.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all three operations propagate correctly to all surfaces.
- ❌ **Fail** if a deleted dish can still be added to a basket, or if a price change does not update in the basket.
- 🐛 **Bug-anyway** if the dish search takes more than 60 seconds to reflect a new dish. Log as **Low**.

#### Edge cases to also try

- **Dish name with special characters** (e.g. "Chef's Pâté") — confirm it displays correctly on the FE and is searchable.
- **MenuScan note**: If dishes were added via the Menu Scan tool (`/Admin/MenuScan`, Upload → Review → Confirm → Result flow), the resulting dishes are subject to the same propagation rules tested here. You do not need to re-test the MenuScan admin flow in this document, but if you notice a discrepancy in dishes created via scan, log it.

#### Common pitfalls

- *"The new dish appeared on the vendor page but not in search"* → run Reindex Vendors specifically. Search indexing can lag behind the main content cache.
- *"The price in the basket didn't update"* → this is a **High** severity bug if checkout can complete at the old price.

---

### TC-XP-006: Dish Availability Toggle and Variant Availability — Real-time FE Hide

**Goal**: Confirm that toggling a dish unavailable (or toggling a variant unavailable) hides it immediately from the customer front-end, and that any basket mid-checkout is handled correctly.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

---

#### Before — capture the starting state

1. As **the customer**, open the **"Sushi Spot"** vendor page.
2. Confirm the dish **"Vegan Pad Thai"** is visible and available to add to the basket.
3. Add it to the basket.
4. **Take a screenshot** of the vendor page showing the dish and the basket with the item.
5. Also note that the dish has at least one **variant** (e.g. "Regular" and "Large") — confirm both appear selectable.

#### Change — perform the admin action

1. In **Vendor Admin** (Sushi Spot), go to **Menu Builder**.
2. Find the **"Vegan Pad Thai"** dish.
3. Toggle **Available** to **OFF** (unavailable).
4. Also, if the dish has variants, toggle the **"Large"** variant's available flag to **OFF** while leaving the dish itself available. (Test both scenarios if possible — see edge cases.)
5. Save the changes.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet). This is a real-time availability change — expect it within 30 seconds.

#### Verify After

- ☐ **Vendor page** (dish toggle off): "Vegan Pad Thai" is **no longer visible** on the vendor page, OR is shown with a clear "Unavailable" label and cannot be added to the basket. Screenshot.
- ☐ **Vendor page** (variant toggle off): "Vegan Pad Thai" is still visible but the **"Large"** size option is greyed out or hidden — only "Regular" can be selected. Screenshot.
- ☐ **Basket** (dish toggle off, item was already in basket): Basket shows the item with a warning ("Sorry, this item is currently unavailable") or it has been removed. The customer cannot proceed to checkout with an unavailable item.
- ☐ **Mid-checkout** (customer is on the checkout page when the dish is toggled off): On the next page load or form submission, the system re-validates the basket and blocks completion, showing a clear error message. Screenshot.
- ☐ **Modifier availability**: If any modifiers (e.g. extra toppings, sauces) have their `IsAvailable` flag set to false, they should not appear as selectable options. This is a sub-check within the dish detail view.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the dish or variant disappears / becomes unselectable within 30 seconds and the basket is handled with a clear warning.
- ❌ **Fail** if the dish remains fully orderable after toggling off, or if checkout completes with an unavailable item.
- 🐛 **Bug-anyway** if the customer sees no warning and their basket is silently emptied. Log as **Medium** (warning is missing).

#### Edge cases to also try

- **Toggle the dish back to available** — it should reappear on the FE within 30 seconds.
- **Unavailable modifier group** — if the entire modifier group (e.g. "Choose your sauce") is toggled off, the dish should still be orderable without that group, unless the group is marked `IsRequired`.
- **IsRequired modifier group with all modifiers unavailable** — the dish itself should become unorderable in this scenario; confirm a clear message appears.

#### Common pitfalls

- *"The dish is greyed out but I can still add it to the basket"* → log as **High**.
- *"The basket didn't warn me the item became unavailable"* → this is acceptable if the checkout page catches it; if checkout also succeeds silently, that is **Critical**.

---

### TC-XP-007: Reorder Dishes and Categories — Customer Vendor Page Order

**Goal**: Confirm that reordering dishes within a category, and reordering categories, is reflected in the correct sequence on the customer vendor page.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-040 (Reorder Dishes within Category), TC-VA-041 (Reorder Categories), TC-VA-042 (Verify Order on FE) in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, open the **"Pizza Roma"** vendor page.
2. **Take a screenshot** of the full menu showing the category order and dish order within the first category ("Starters").
3. Note the exact current order: e.g. *"Starters → Garlic Bread, Bruschetta, Mozzarella Sticks"* and *"Starters, Mains, Desserts"* as the category order.

#### Change — perform the admin action

1. In **Vendor Admin**, go to **Menu Builder** for Pizza Roma.
2. **Reorder dishes**: Move **"Mozzarella Sticks"** to the **first position** in Starters (drag and drop or use the order arrows).
3. **Reorder categories**: Move **"Desserts"** to the **first position** (above Starters and Mains).
4. Save the changes.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After

- ☐ **Vendor page — category order**: **"Desserts"** is now the first category shown, followed by Starters and Mains. Screenshot.
- ☐ **Vendor page — dish order within Starters**: **"Mozzarella Sticks"** is now the first dish listed in Starters. Screenshot.
- ☐ **Basket**: No change expected — reordering does not affect basket contents or prices.

#### Revert

Reorder everything back to the original order to avoid breaking other test cases.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if both the category order and dish order match what was set in Vendor Admin.
- ❌ **Fail** if the order on the customer FE does not reflect the admin change.
- 🐛 **Bug-anyway** if the order is correct on desktop but wrong on mobile. Log as **Medium** and note the device/browser.

#### Edge cases to also try

- **Reorder on mobile** — visit the vendor page on a mobile browser after reordering; the order should be consistent.
- **Single-dish category** — reordering within a category with only one dish should have no visible effect and should not cause an error.

#### Common pitfalls

- *"The order looks right on the vendor page but categories are in the wrong order in the dish search"* — search results may order by relevance rather than display order; this is expected and not a bug.

---

### TC-XP-008: Upload Dish Image — Front-end Display and Thumbnail Processing

**Goal**: Confirm that a newly uploaded dish image appears on the customer vendor page and dish detail page, with correctly sized thumbnail and appropriate alt text.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

---

#### Before — capture the starting state

1. As **the customer**, open the **"Garlic Bread"** dish on the Pizza Roma vendor page.
2. **Take a screenshot** of the current state — either the dish has no image (placeholder icon) or an existing image.

#### Change — perform the admin action

1. In **Vendor Admin** (Pizza Roma), go to **Menu Builder** → find **"Garlic Bread"**.
2. Click **Edit** on the dish.
3. Upload a new image. Use a clearly distinctive test image (e.g. a bright red square saved as `garlic-bread-test.png`).
4. Save the dish.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet). Images may take slightly longer (up to 60 seconds) due to thumbnail processing.

#### Verify After

- ☐ **Vendor page — dish card**: The Garlic Bread card now shows the new image as a thumbnail. It is not broken (no broken image icon). It is not distorted or stretched beyond recognition. Screenshot.
- ☐ **Dish detail page** (click on Garlic Bread): The full image is shown at a larger size. It loads without error. Screenshot.
- ☐ **Alt text** (right-click the image → "Inspect" if you are comfortable doing so, or ask a tester with technical access): The `alt` attribute should contain the dish name ("Garlic Bread"), not be empty. If you cannot inspect, note this check as "Not verified (no technical access)" — that is acceptable.
- ☐ **Vendor card on home page / search**: The vendor card for Pizza Roma should not show the dish image (vendor cards show the vendor logo/banner, not dish images) — confirm no unexpected image appears there.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the image appears correctly on the dish card and dish detail page within 60 seconds.
- ❌ **Fail** if a broken image icon appears, or if the image does not appear at all after 90 seconds.
- 🐛 **Bug-anyway** if the image is visibly distorted or cropped badly. Log as **Low** with a screenshot.

#### Edge cases to also try

- **Large image file** (e.g. a 5MB JPEG) — the upload should either accept it (and resize automatically) or show a clear file-size limit error.
- **Non-square image** — does the thumbnail crop correctly to a square, or is the dish card broken?
- **Delete the image** (after testing) — the dish should fall back to the placeholder icon gracefully.

---

### TC-XP-009: Hero Slide Publish / Unpublish / Reorder / Delete — Homepage Carousel

**Goal**: Confirm that hero slide changes in System Admin propagate correctly to the customer front-end homepage carousel, including mobile vs desktop display, CTA link targets, and regional targeting via RegionIds.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window (test on both desktop and mobile viewport)

**See also**: TC-SA-120 through TC-SA-127 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, open the home page (`/`).
2. **Take a screenshot** of the hero carousel in its current state — note the slide order, the text on each slide, and any CTA buttons.
3. Note the current first slide. Write in your bug tracker: *"Before: First slide is '[title]' with CTA '[label]' pointing to '[URL]'."*
4. Resize your browser to a **mobile viewport** (375px wide) and take a second screenshot — note whether the hero images are different (mobile-specific assets may be configured separately).

#### Change A — publish a new hero slide

1. In **System Admin**, go to **Content → Hero Slides** (`/Admin/ContentHeroSlides`).
2. Click **Add New Slide** (or equivalent).
3. Fill in:
   - **Title**: `Summer Festival Special`
   - **Body text**: `Order now and get a free dessert.`
   - **CTA label**: `Order Now`
   - **CTA URL**: `/search?q=dessert`
   - **Published**: toggled ON
   - **Display order**: set to **1** (first position)
   - **RegionIds**: leave blank (or select "All regions") — this slide should be visible everywhere
4. Upload an appropriately sized desktop hero image and a mobile hero image.
5. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Publish)

- ☐ **Home page — desktop**: The "Summer Festival Special" slide now **appears in the carousel**. It is in the **first position**. The title, body text, and CTA label ("Order Now") are correct. Screenshot.
- ☐ **CTA link**: Click the "Order Now" button. It navigates to `/search?q=dessert`. It does **not** open a new tab unexpectedly (unless designed to). Screenshot the resulting page.
- ☐ **Home page — mobile viewport**: The slide is also visible on mobile. The mobile image is used (if a separate mobile image was uploaded). Text is readable at mobile sizes. Screenshot.
- ☐ **Regional targeting**: If you set a RegionId on the slide, a customer in a **different region** should NOT see this slide. (Test by navigating from an address outside the target region — or ask your project manager to confirm regional targeting is configured correctly in the UAT environment.)

#### Change B — reorder slides

1. In System Admin, change the "Summer Festival Special" slide's **display order** to **2** (move it to second).
2. Save.

#### Verify After (Reorder)

- ☐ **Home page carousel**: The "Summer Festival Special" slide now appears **second** in the carousel, not first. Screenshot.

#### Change C — unpublish the slide

1. In System Admin, find the "Summer Festival Special" slide and toggle **Published** to **OFF**.
2. Save.

#### Verify After (Unpublish)

- ☐ **Home page**: The "Summer Festival Special" slide **no longer appears** in the carousel. Screenshot.

#### Change D — delete the slide

1. In System Admin, **delete** the "Summer Festival Special" slide entirely.
2. Confirm the deletion prompt.

#### Verify After (Delete)

- ☐ **Home page**: Carousel is unchanged from before the test (same slides as the original "Before" screenshot). No broken slide or blank space.

#### Global Hero Assets (TC-XP-029 sub-check)

> Note: The **global carousel CSS and JavaScript** (transition animations, timing, etc.) are managed separately in `/Admin/ContentHeroAssets`. If you notice the carousel animation is broken or the transition style has changed unexpectedly, cross-reference with TC-XP-029.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all four operations propagate correctly on both desktop and mobile.
- ❌ **Fail** if an unpublished or deleted slide remains visible on the homepage.
- 🐛 **Bug-anyway** if the carousel layout breaks on mobile after adding the new slide. Log as **Medium**.

#### Edge cases to also try

- **Slide with a very long title** (over 80 characters) — does text overflow the carousel container?
- **Carousel with only one slide** — it should still render without navigation arrows causing a loop back to itself confusingly.

#### Common pitfalls

- *"The slide appears on desktop but not mobile"* → check whether separate mobile assets were required and not uploaded.
- *"CTA link opens wrong page"* → verify the URL entered in admin exactly — trailing slashes and query strings matter.

---

### TC-XP-010: Support Local Slide Create / Edit / Publish / Delete — Homepage Section

**Goal**: Confirm that Support Local entries managed in System Admin appear, update, and disappear correctly in the "Support Local" section on the customer home page.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-140 through TC-SA-146 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, open the home page (`/`) and scroll to the **"Support Local"** section.
2. **Take a screenshot** of the current Support Local entries.
3. Note how many entries are visible and their order.

#### Change — perform the admin action

1. In **System Admin**, go to **Content → Support Local** (`/Admin/ContentSupportLocal`).
2. Click **Add New** and fill in:
   - **Title**: `Family-Run Gems`
   - **Subtitle**: `Discover independently owned restaurants in your area.`
   - **Image**: upload a test image
   - **Link URL**: `/search?q=independent`
   - **Published**: ON
   - **Display order**: 1 (first)
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After

- ☐ **Home page — Support Local section**: The **"Family-Run Gems"** entry appears at the top of the section. Title, subtitle, and image are correct. Screenshot.
- ☐ **Link**: Clicking the entry navigates to `/search?q=independent`.
- ☐ **Unpublish**: Toggle the entry to unpublished in System Admin → the entry should **disappear** from the home page within 30 seconds.
- ☐ **Delete**: Delete the entry → confirm the home page returns to the state captured in the "Before" screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the entry appears, links correctly, disappears when unpublished, and the section is stable after deletion.
- ❌ **Fail** if a deleted or unpublished entry remains visible.
- 🐛 **Bug-anyway** if the section disappears entirely when all entries are unpublished (the section container should handle an empty state gracefully, not cause a layout break). Log as **Low**.

#### Edge cases to also try

- **Reorder two existing Support Local entries** — confirm the customer home page reflects the new order.
- **Entry with no image** — if the image field is optional, confirm the section handles it without a broken layout.

---

### TC-XP-011: Cuisine Taxonomy Reorder — Mobile vs Web Separate Ordering

**Goal**: Confirm that reordering cuisine types in System Admin with different display orders for mobile and web is correctly reflected in the cuisine ribbon on the customer front-end for each context.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end — desktop window
- ☐ Customer Front-end — mobile viewport (or a second device/browser resized to 375px)

**See also**: TC-SA-150 (Reorder Cuisines — Web), TC-SA-151 (Reorder Cuisines — Mobile), TC-SA-152 (Cuisine Ribbon on FE) in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. On the **desktop** customer front-end, screenshot the cuisine ribbon (the row of cuisine icons/buttons near the top of the home page or search page).
2. On a **mobile viewport**, screenshot the same cuisine ribbon.
3. Note the order of the first three cuisines on each. Write: *"Before — Desktop: [C1, C2, C3…]. Mobile: [C1, C2, C3…]".*

#### Change — perform the admin action

1. In **System Admin**, go to **Taxonomy → Cuisine Types** (`/Admin/CuisineTypes`).
2. Find **"Pizza"** and set:
   - **Web display order**: `1` (first on web)
   - **Mobile display order**: `3` (third on mobile)
3. Find **"Japanese"** and set:
   - **Web display order**: `2`
   - **Mobile display order**: `1` (first on mobile)
4. Save all changes.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet) including Rebuild Caches.

#### Verify After

- ☐ **Customer front-end — desktop cuisine ribbon**: **"Pizza"** appears first, **"Japanese"** appears second. Screenshot.
- ☐ **Customer front-end — mobile cuisine ribbon** (resize to 375px width): **"Japanese"** appears first, **"Pizza"** appears third. Screenshot.
- ☐ **Cuisine page links**: Clicking "Pizza" in the ribbon navigates to `/cuisine/pizza` (or equivalent). Clicking "Japanese" navigates to `/cuisine/japanese`. Both pages load correctly.

#### Revert

Return all cuisine display orders to their original values.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the desktop and mobile orders are independent and correct on both viewport sizes.
- ❌ **Fail** if the mobile and web orders are the same despite being configured differently, or if the cuisine ribbon does not update at all.
- 🐛 **Bug-anyway** if the cuisine ribbon breaks its layout when two cuisines have the same display order value. Log as **Medium**.

#### Edge cases to also try

- **Two cuisines with the same display order** — the tie-breaking rule (e.g. alphabetical) should apply consistently.

---

### TC-XP-012: Region Create / Edit / Delete — Hero Slide Targeting and Discovery

**Goal**: Confirm that creating and editing regions in System Admin correctly scopes hero slide visibility and location-based vendor discovery on the customer front-end.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-130 through TC-SA-133 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, enter delivery address **"SW1A 1AA"** (Westminster, London).
2. Screenshot the home page hero carousel as seen from this location.
3. Note which hero slides are visible.

#### Change — perform the admin action

1. In **System Admin**, go to **Content → Regions** (`/Admin/ContentRegions`).
2. Create a new region: **Name** = `London Test Region`, and configure it to include postcode area **SW1** (or equivalent region geometry available in UAT).
3. Go to **Content → Hero Slides** and **edit** an existing hero slide (or the one created in TC-XP-009).
4. Set the slide's **RegionIds** to target **"London Test Region"** only.
5. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After

- ☐ **Customer FE — address SW1A 1AA** (in region): The targeted hero slide **is visible** in the carousel. Screenshot.
- ☐ **Customer FE — address M1 1AE** (Manchester, not in the London region): The targeted hero slide **is NOT visible** in the carousel. Screenshot.
- ☐ **Vendor discovery**: Vendors in the London Test Region area are correctly surfaced when a London address is entered. This is not directly affected by region creation, but confirm no discovery regression occurred. Screenshot.

#### Edit region

1. Edit the "London Test Region" name to **"Greater London"**.
2. Verify the renamed region appears in the hero slide's region targeting dropdown and the slide's targeting still works.

#### Delete region

1. Delete the "Greater London" region.
2. Verify the hero slide previously targeting that region either becomes **globally visible** (if that is the system's default for no-region slides) or is **automatically unpublished** — whichever behaviour is correct per spec. If neither behaviour occurs (e.g. the slide simply disappears silently), log a bug.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if regional targeting correctly shows/hides slides by location.
- ❌ **Fail** if a region-targeted slide is visible to all users regardless of location.
- 🐛 **Bug-anyway** if deleting a region causes an unrelated hero slide to disappear unexpectedly. Log as **High**.

#### Common pitfalls

- *"I can't tell which region my test address falls in"* → ask your project manager to confirm which postcodes map to which regions in the UAT environment.

---

### TC-XP-013: Bad Words — Review Submission Rejection on Customer Front-end

**Goal**: Confirm that adding a bad word in System Admin causes new review submissions containing that word to be rejected on the customer front-end, and that existing approved reviews containing the word are not retroactively removed.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-160 through TC-SA-165 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. Check that the word **"uglytest"** does not currently exist in the bad words list (System Admin → **Settings → Bad Words** tab).
2. As **the customer**, navigate to a vendor you have previously ordered from (e.g. "Pizza Roma").
3. Confirm you can see the **"Leave a Review"** button or equivalent.
4. Confirm there is at least one approved review currently visible. **Take a screenshot** — this is the "before" state for existing reviews.

#### Change — add the bad word

1. In **System Admin**, go to **Settings → Bad Words** tab (or `/Admin/Settings?tab=badwords`).
2. Add the word: **`uglytest`** (all lowercase).
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet). Wait 30 seconds before attempting the review submission.

#### Verify After (new review rejected)

1. As **the customer**, attempt to submit a new review for Pizza Roma containing the word **"uglytest"** (e.g. "Worst uglytest pizza ever.").
2. ☐ The review submission is **rejected** with a clear error message (e.g. "Your review contains language that is not permitted. Please edit your review."). Screenshot.
3. ☐ The review does **not** appear on the vendor page.
4. Now submit a review **without** "uglytest" (e.g. "Great pizza, would order again.").
5. ☐ The clean review is **accepted** and either appears immediately or is queued for moderation. Screenshot.

#### Verify After (existing reviews unaffected)

- ☐ **Existing approved reviews**: The reviews that were visible before you added "uglytest" to the bad words list are **still visible** — the system should not retroactively remove or hide them. Screenshot and compare to the "Before" screenshot.

#### Bulk upload note

If the System Admin Settings tab includes a **bulk upload** feature for bad words (e.g. a CSV upload), also test: upload a file containing the word **"bulktest"**, confirm it is added, then attempt a review submission containing "bulktest" and verify it is rejected.

#### Cleanup

After this TC, **remove "uglytest"** (and "bulktest" if added) from the bad words list so future review tests are not affected.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the new review is rejected with a clear message and existing reviews remain untouched.
- ❌ **Fail** if a review containing the bad word is accepted and published.
- ❌ **Fail** if existing reviews disappear after adding the word to the bad words list.
- 🐛 **Bug-anyway** if the rejection message does not tell the customer what to do next. Log as **Low**.

#### Edge cases to also try

- **Case sensitivity**: try "UGLYTEST" and "UglyTest" — the bad word filter should catch all case variations.
- **Word inside another word**: try "uglytestation" — decide whether the system should reject this (whole-word match vs substring match) and check actual behaviour against expected.

---

### TC-XP-014: Dish Keywords — Search Synonym Matching on Customer Front-end

**Goal**: Confirm that adding a dish keyword (synonym or alternative name) in System Admin causes customer searches for that keyword to return the associated dish.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-160 through TC-SA-165 cross-reference (bad words/keywords share the Settings area); dish keywords have their own TC group in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, search for **"vegan noodles"** (`/search?q=vegan+noodles`).
2. **Take a screenshot**. "Vegan Pad Thai" from Sushi Spot should **not** appear in results at this point (it is not labelled "noodles" by default).

#### Change — perform the admin action

1. In **System Admin**, go to **Dish Keywords** (`/Admin/DishKeywords`).
2. Find the dish **"Vegan Pad Thai"** (from Sushi Spot) or add a keyword entry for it.
3. Add keyword: **`vegan noodles`**.
4. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet) — **you must also run Reindex Vendors** for keyword changes to appear in search.

#### Verify After

- ☐ **Search results** (`/search?q=vegan+noodles`): **"Vegan Pad Thai"** from Sushi Spot now appears in the results. Screenshot.
- ☐ **Search results** (`/search?q=vegan+pad+thai`): "Vegan Pad Thai" also still appears via its original name (the keyword did not replace the name — it is additive).
- ☐ **No unintended matches**: The search for "vegan noodles" should not return completely unrelated dishes from other vendors that do not have this keyword.

#### Delete the keyword

1. In **System Admin**, remove the **"vegan noodles"** keyword from Vegan Pad Thai.
2. Run **Reindex Vendors** and wait 30 seconds.
3. Search for "vegan noodles" again.
4. ☐ **"Vegan Pad Thai"** no longer appears in results for "vegan noodles". It still appears for "vegan pad thai". Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the keyword addition causes the dish to appear in synonym search, and removal causes it to disappear.
- ❌ **Fail** if the keyword addition has no effect on search results even after Reindex Vendors.
- 🐛 **Bug-anyway** if adding a keyword causes wildly unrelated results to appear. Log as **Medium**.

#### Edge cases to also try

- **Multi-word keyword with different word order** — e.g. add "noodles vegan" and test whether "vegan noodles" still matches.
- **Keyword on a dish that is currently unavailable** — the keyword match should not surface an unavailable dish.

---

### TC-XP-015: Platform Offer Activate / Pause / Edit — Basket, Checkout, and Eligibility

**Goal**: Confirm that activating, pausing, and editing a platform-wide discount offer is correctly reflected in the customer basket and checkout, including eligibility checks (minimum spend, vendor scope, cuisine scope).

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-090 through TC-SA-097 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, add items from **"Pizza Roma"** worth at least **£20** to the basket.
2. Navigate to the basket. **Take a screenshot** showing the subtotal with no offer applied.
3. Navigate to checkout and confirm no discount line appears.

#### Change — activate a new offer

1. In **System Admin**, go to **Offers** (`/Admin/Offers`).
2. Create a new offer (or activate an existing one) with these settings:
   - **Offer code**: `SUMMER10`
   - **Type**: Percentage discount — **10%**
   - **Minimum spend**: **£15.00**
   - **Applicable to**: All vendors (platform-wide)
   - **Start date**: today / **End date**: tomorrow
   - **Status**: Active
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Activate)

- ☐ **Home page**: An offer badge or banner for "SUMMER10" / "10% off" appears on the homepage or near vendor cards (if the platform surfaces this). Screenshot.
- ☐ **Basket**: The basket shows an option to enter a promo code. Enter **`SUMMER10`** and apply it. The discount line **"-£2.00"** (10% of £20) appears in the basket summary. Screenshot.
- ☐ **Checkout**: The discount line is carried through to the checkout summary. The total is reduced correctly.
- ☐ **Minimum spend eligibility**: Remove one item so the basket total drops below **£15**. Re-apply `SUMMER10`. The system should **reject** the code with a message like "Minimum order amount is £15". Screenshot.

#### Change — pause the offer

1. In **System Admin**, set the `SUMMER10` offer status to **Paused**.
2. Save.

#### Verify After (Pause)

- ☐ **Basket**: Apply `SUMMER10` again. The code is **rejected** with a message such as "This offer is not currently active." Screenshot.
- ☐ **Home page offer badge** (if it appeared earlier): The badge is **gone** or shows as expired/unavailable.

#### Change — edit the offer

1. **Reactivate** the offer and change the discount to **15%**.
2. Save.

#### Verify After (Edit)

- ☐ **Basket**: The discount applied is now **15%** of the basket total (not 10%). Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all activation, pause, and edit changes propagate correctly including eligibility rejections.
- ❌ **Fail** if a paused offer code is still accepted at checkout.
- ❌ **Fail** if the discount percentage in the basket does not update after editing the offer.
- 🐛 **Bug-anyway** if the error message for a paused code is confusing or unhelpful. Log as **Low**.

#### Edge cases to also try

- **Vendor-scoped offer**: Create an offer applicable to Pizza Roma only. Confirm it does **not** apply to a basket containing only Sushi Spot items.
- **Cuisine-scoped offer**: If configurable, create an offer for "Italian" cuisine only. Test with a pizza order (should apply) and a sushi order (should not apply).
- **Stacking**: Attempt to apply both a platform offer and a vendor offer simultaneously — see TC-XP-016 for the specific stacking test.

---

### TC-XP-016: Vendor Offer Create / Pause / Clone / Delete — Promo Code and Basket

**Goal**: Confirm that vendor-specific offers created in Vendor Admin are correctly validated at checkout, appear in the basket, and that pausing or deleting the offer prevents its use.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-080 through TC-VA-088 in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, add items from **"Sushi Spot"** to the basket (minimum £20 worth).
2. Proceed to checkout and **take a screenshot** of the checkout page showing no discount.

#### Change — create a vendor offer

1. In **Vendor Admin** (Sushi Spot), go to **Offers** (`/Admin/Offers`).
2. Create a new offer:
   - **Code**: `SUSHILOVER`
   - **Type**: Fixed amount — **£5 off**
   - **Minimum spend**: **£20**
   - **Status**: Active
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Create)

- ☐ **Checkout — promo code field**: Enter **`SUSHILOVER`**. The basket summary shows **"−£5.00"** discount. Total is correct. Screenshot.
- ☐ **Wrong vendor**: Clear the Sushi Spot basket and add items from **Pizza Roma** instead. Apply `SUSHILOVER`. The code should be **rejected** with a message such as "This offer is not valid for this restaurant." Screenshot.

#### Stacking check

- ☐ Add Sushi Spot items (£20+) and apply **`SUMMER10`** (platform offer from TC-XP-015) AND **`SUSHILOVER`** simultaneously. Note: the system may or may not allow stacking — whatever it does, the behaviour should be consistent and clearly communicated. If stacking is blocked, the error message should explain why. Screenshot.

#### Change — pause the offer

1. In **Vendor Admin**, set `SUSHILOVER` to **Paused**.
2. Test: Apply `SUSHILOVER` at checkout. ☐ Rejected with a clear message.

#### Change — clone the offer

1. In **Vendor Admin**, clone `SUSHILOVER` to create `SUSHILOVER2` (same settings).
2. Activate `SUSHILOVER2`.
3. Test: Apply `SUSHILOVER2` at checkout. ☐ Accepted with the same £5 discount.

#### Change — delete the offer

1. In **Vendor Admin**, delete `SUSHILOVER`.
2. Test: Apply `SUSHILOVER` at checkout. ☐ Rejected ("offer not found" or similar message).

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all four operations (create, pause, clone, delete) are correctly reflected at checkout.
- ❌ **Fail** if a deleted offer code still applies a discount.
- 🐛 **Bug-anyway** if a cloned offer inherits the wrong settings. Log as **Medium**.

---

### TC-XP-017: Commission Tier Create / Edit / Delete — Vendor Reassignment and Invoice Maths

**Goal**: Confirm that creating, editing, and deleting commission tiers in System Admin correctly reassigns vendors and that the correct commission percentage is used in the next invoicing calculation.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)

> **Note**: Commission changes are not directly visible to customers — this TC focuses on System Admin and the downstream invoicing calculation, not the customer front-end.

**See also**: TC-SA-070 through TC-SA-077 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. In **System Admin**, go to **Commissions → Commission Tiers** (`/Admin/CommissionTiers`).
2. **Take a screenshot** of the current list of tiers, their names, and their commission rates.
3. Note which tier **"Pizza Roma"** is currently assigned to.

#### Change — create a new tier

1. Create a new tier:
   - **Name**: `UAT Test Tier`
   - **Rate**: `12.5%`
2. Save.

#### Change — reassign a vendor to the new tier

1. Go to **Commissions** (`/Admin/Commissions`).
2. Find **"Pizza Roma"** and change its assigned tier to **"UAT Test Tier"**.
3. Save.

#### Change — run manual recalculation

1. In **System Admin**, go to **Commissions** and use the **"Recalculate"** or **"Manual Recalc"** function (if available).
2. Confirm the calculation completes successfully.

#### Verify After

- ☐ **Pizza Roma commission tier**: Shows **"UAT Test Tier"** at **12.5%** in the commissions list. Screenshot.
- ☐ **Invoice maths**: If a test invoice is generated (or an existing one can be previewed), confirm the commission line uses 12.5% not the previous rate.
- ☐ **Audit log** (back-end log only — tester cannot verify directly): Note in your bug report that you changed the tier; if the downstream invoice calculation looks wrong, log as a potential audit/logging failure.

> **Important note**: There is **no partial refund UI** in the WantFood platform. If a vendor cancels a captured order, any refund is handled implicitly by the back-end — it does not need to be triggered manually. This is relevant if you are checking commission calculations on cancelled orders.

#### Change — delete the test tier

1. Delete **"UAT Test Tier"** from the tier list.
2. ☐ If Pizza Roma was assigned to this tier, confirm either: the vendor is automatically moved to a default tier, or the deletion is blocked with a message like "This tier has vendors assigned; please reassign them first." Screenshot either outcome.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if vendor reassignment and invoice maths are correct.
- ❌ **Fail** if a vendor can be left with no tier assigned, causing errors in invoice generation.
- 🐛 **Bug-anyway** if the tier name appears correctly but the rate is wrong. Log as **High**.

#### Edge cases to also try

- **Two vendors on the same tier** — deleting the tier should require reassigning both, not just one.

---

### TC-XP-018: Commission Config Change + Manual Recalculation

**Goal**: Confirm that changing the commission configuration and running a manual recalculation in System Admin results in vendors being reassigned correctly and the downstream invoice calculation uses updated rates.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)

**See also**: TC-SA-070 through TC-SA-077 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. In **System Admin**, go to **Commissions** (`/Admin/Commissions`).
2. **Take a screenshot** of the current commission configuration — the threshold values, tier mapping rules, and any auto-assignment rules.
3. Note the current commission tier assigned to both **"Pizza Roma"** and **"Sushi Spot"**.

#### Change — update commission config

1. If there is an auto-assignment rule (e.g. "vendors with revenue over £10,000/month move to Tier 2"), adjust the threshold so that Pizza Roma would qualify for a different tier.
2. Save the configuration.

#### Change — run manual recalculation

1. Click **Recalculate** / **Run Manual Recalc** (or the equivalent tool in System Admin → Commissions).
2. Wait for the process to complete and confirm success.

#### Verify After

- ☐ **Pizza Roma tier**: After recalculation, Pizza Roma's tier reflects the new assignment rule. Screenshot.
- ☐ **Sushi Spot tier**: If Sushi Spot did not meet the threshold, it should remain on its original tier. Screenshot.
- ☐ **Downstream invoice maths**: The next invoice run (or a preview) uses the updated tier rates. Screenshot if a preview is available.
- ☐ **Back-end log** (tester cannot verify directly): Note in your bug tracker that a manual recalc was run. If the tier assignments look wrong, flag a potential log/audit miss.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if vendor tiers update correctly after recalculation and invoice maths are consistent.
- ❌ **Fail** if recalculation completes but vendor tier assignments do not change.

---

### TC-XP-019: Platform Payment Settings — Add / Remove Stripe and WorldPay

**Goal**: Confirm that adding or removing a payment provider in System Admin correctly enables or disables that payment method for customers at checkout, including graceful error handling when a provider is removed mid-session.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-170 through TC-SA-174 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, add items from **"Pizza Roma"** to the basket and proceed to **checkout**.
2. **Take a screenshot** of the payment methods shown at checkout — confirm that **card payment (Stripe)** is available.

#### Change — disable Stripe at platform level

1. In **System Admin**, go to **Payment Settings** (`/Admin/PaymentSettings`).
2. Locate the **Stripe** integration and **disable** it (toggle off or set to inactive).
3. Save. Confirm the success toast.
4. **Take a screenshot** of the Payment Settings page showing Stripe as disabled.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Stripe disabled)

- ☐ **Checkout — payment methods**: Card payment (Stripe) is **no longer available** as an option. Screenshot.
- ☐ **Cash option**: If cash is enabled for the vendor, cash payment should still be available (Stripe being disabled should not remove cash).
- ☐ **3D Secure check** (if Stripe is re-enabled later): When you re-enable Stripe and process a payment with the 3DS test card (`4000 0025 0000 3155`), the 3D Secure prompt still appears and the flow completes correctly.

#### Mid-session scenario

1. Re-enable Stripe.
2. As **the customer**, start checkout and reach the **payment page** — do not submit yet.
3. Switch to **System Admin** and disable Stripe again.
4. Return to the customer checkout and attempt to **submit the payment**.
5. ☐ The system should return a clear error message (e.g. "Payment method unavailable, please choose another option") — **not** a blank page or unhandled crash. Screenshot.

#### Re-enable Stripe

1. Re-enable Stripe in System Admin.
2. ☐ Card payment reappears at checkout. Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if disabling a payment provider removes it from checkout and the mid-session error is handled gracefully.
- ❌ **Fail** if a disabled payment method can still be selected and a payment completes.
- ❌ **Fail** if disabling Stripe causes a crash or unhandled error at checkout.
- 🐛 **Bug-anyway** if the mid-session error message is cryptic. Log as **Medium**.

#### Common pitfalls

- *"I disabled Stripe but card payment still shows"* → follow the full propagation checklist; payment settings may have a separate cache layer.

---

### TC-XP-020: Vendor Stripe Connect — Connect / Disconnect Effect on Customer Checkout

**Goal**: Confirm that a vendor connecting their Stripe account enables card payment for customers at that vendor's checkout, and that disconnecting removes it.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-110 through TC-VA-114 in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, add items from **"Sushi Spot"** to the basket and proceed to checkout.
2. **Take a screenshot** of the payment methods shown. Confirm whether card payment is currently available or not.

#### Change — disconnect Stripe Connect

1. In **Vendor Admin** (Sushi Spot), go to **Payment Methods** (`/Admin/PaymentMethods`).
2. Find the **Stripe Connect** section and click **Disconnect** (or **Remove**).
3. Confirm the disconnection. **Take a screenshot** of the Payment Methods page showing Stripe as not connected.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Disconnect)

- ☐ **Customer checkout** (Sushi Spot items): Card payment option **disappears**. Screenshot.
- ☐ **Vendor Admin onboarding banner**: A banner or prompt should appear in Vendor Admin reminding the vendor to connect Stripe to accept card payments (if such a banner exists in the design).
- ☐ **Cash payment**: Cash payment (if configured for Sushi Spot) should remain unaffected. Screenshot.

#### Change — reconnect Stripe Connect

1. In **Vendor Admin**, go through the Stripe Connect flow to re-connect the account (in UAT, this may be a test Stripe account).
2. Confirm connection success.

#### Verify After (Reconnect)

- ☐ **Customer checkout**: Card payment option **reappears** for Sushi Spot. Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if disconnecting Stripe removes card payment and reconnecting restores it.
- ❌ **Fail** if card payment remains available after disconnecting Stripe.
- 🐛 **Bug-anyway** if there is no onboarding banner in Vendor Admin after disconnecting. Log as **Low** (important for vendor usability).

---

### TC-XP-021: Enable / Disable Cash Payment at Vendor — Checkout Option

**Goal**: Confirm that enabling or disabling cash payment for a specific vendor in Vendor Admin correctly shows or hides the cash option at checkout for that vendor only, without affecting other vendors.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-110 through TC-VA-114 in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, add items from **"Pizza Roma"** to the basket and proceed to checkout.
2. **Take a screenshot** of the payment methods. Note whether **"Cash on delivery"** is currently shown.
3. Repeat for **"Sushi Spot"** — take a second screenshot for comparison. We need to confirm changes to Pizza Roma do not affect Sushi Spot.

#### Change — disable cash at Pizza Roma

1. In **Vendor Admin** (Pizza Roma), go to **Payment Methods** (`/Admin/PaymentMethods`).
2. Toggle **"Accept cash on delivery"** to **OFF**.
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After

- ☐ **Pizza Roma checkout**: The **cash on delivery option is gone**. Only card payment (if Stripe is connected) remains. Screenshot.
- ☐ **Sushi Spot checkout**: Cash option is **unchanged** — if it was available before, it is still available. Screenshot.

#### Change — re-enable cash at Pizza Roma

1. In **Vendor Admin**, toggle **"Accept cash on delivery"** back to **ON**.
2. ☐ Cash option reappears at Pizza Roma checkout. Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the cash toggle affects only Pizza Roma and does not touch Sushi Spot.
- ❌ **Fail** if disabling cash at one vendor also removes it for other vendors.
- ❌ **Fail** if the cash option remains after being disabled.

#### Edge cases to also try

- **Both Stripe and cash disabled**: If a vendor has no payment method enabled, the checkout page should block order completion with a clear message ("No payment methods available") rather than silently failing.

---

### TC-XP-022: Delivery Cost Update — Customer Quote and Basket Totals

**Goal**: Confirm that updating a vendor's distance-based delivery cost tiers in Vendor Admin causes the correct delivery fee to be quoted on the customer front-end for addresses in each tier.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-100 (Delivery Cost Configuration), TC-VA-101 (Test Delivery Quote) in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, enter delivery address **"SW1A 1AA"** and navigate to **"Pizza Roma"**.
2. Add items and proceed to the delivery address / quote step.
3. **Take a screenshot** of the delivery fee quoted (e.g. "Delivery: £2.50").
4. Note the current delivery cost tier settings in Vendor Admin (take a screenshot of the Delivery Costs page for Pizza Roma).

#### Change — update a delivery cost tier

1. In **Vendor Admin** (Pizza Roma), go to **Delivery Costs** (`/Admin/DeliveryCosts`).
2. Find the tier that covers **0–2 km** (the tier your test address falls into).
3. Change the delivery fee from its current value to **£3.99**.
4. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After

- ☐ **Customer delivery quote**: Re-enter the same address and start a new order from Pizza Roma. The delivery fee now shows **£3.99**. Screenshot.
- ☐ **Basket total**: Add items and confirm the basket total includes the **£3.99** delivery fee correctly. Screenshot.
- ☐ **Order confirmation total** (if you complete a test order): The order confirmation email and the confirmation page both show **£3.99** delivery fee — not the old rate. Screenshot.
- ☐ **Address outside all tiers**: Enter an address that is beyond the maximum delivery distance (e.g. a postcode more than 10 km away). The system should decline to quote a delivery fee and show a "Sorry, we can't deliver to this address" message — not £0 or a negative fee. Screenshot.

> **Important note**: There is **no partial refund UI** in WantFood. If the delivery fee changes after an order has been placed, the original order total is honoured — refunds are handled implicitly by the back-end if the vendor cancels.

#### Revert

Return the 0–2 km delivery cost to its original value.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the updated delivery fee is quoted correctly and appears in basket and order confirmation.
- ❌ **Fail** if the old delivery fee is still quoted after the full propagation checklist.
- 🐛 **Bug-anyway** if the delivery fee in the basket does not match the delivery fee on the confirmation page. Log as **High**.

#### Edge cases to also try

- **Changing the delivery distance threshold** (e.g. extending the 0–2 km tier to 0–3 km) — addresses at 2.5 km should now qualify for the tier fee rather than the next tier's higher fee. Test with a specific address at that distance if your project manager can provide one.

---

### TC-XP-023: Vendor Trading Hours Edit — "Closed Now" Indicator and Checkout Block

**Goal**: Confirm that editing a vendor's trading hours in Vendor Admin causes the customer front-end to correctly show "Closed now" when outside hours, and that the basket / checkout is blocked during closed periods. Also confirm that scheduled-order slot configuration propagates correctly.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-VA-021 and related vendor management TCs in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. In **Vendor Admin** (Pizza Roma), go to **Restaurant Management** → **Branch tab** and note the current opening hours (the `OpeningHoursJson` configuration).
2. Check today's trading hours. **Take a screenshot**.
3. As **the customer**, open the Pizza Roma vendor page. Note whether it shows "Open now" or "Closed now" — **take a screenshot**.

#### Change A — set the vendor as currently closed

1. In **Vendor Admin**, edit the `OpeningHoursJson` for today's day-of-week to a closed time slot (e.g. if today is Wednesday, set Wednesday hours to "Closed" or to a time window in the past, such as 09:00–10:00 when it is currently 14:00).
2. Also set the relevant branch fields:
   - **`IsActive`**: remains `true` (the branch is active, just closed right now)
   - **`PrepLeadTimeMinutes`**: leave at current value (e.g. 20 minutes)
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Closed now)

- ☐ **Vendor page**: The vendor shows a clear **"Closed now"** indicator (banner, label, or greyed-out state). The opening times for today are visible (e.g. "Opens at 17:00"). Screenshot.
- ☐ **Add to basket**: The customer should be **unable to add items** to the basket (the "Add" button is disabled or replaced with a "Closed" message). Screenshot.
- ☐ **Checkout block**: If the customer already has items in the basket from before the closure, the checkout should be blocked with a message such as "This restaurant is currently closed. Your order cannot be placed." Screenshot.
- ☐ **Vendor card on home page**: The vendor card should reflect the "Closed" state (e.g. a "Closed" badge). Screenshot.

#### Change B — configure scheduled-order slots

1. In **Vendor Admin**, edit the branch with:
   - **`AcceptsScheduledOrders`**: `true`
   - **`MaxScheduleAheadHours`**: `48` (customer can schedule up to 48 hours ahead)
   - **`ScheduledOrderSlotIntervalMinutes`**: `30` (30-minute slot intervals)
2. Save.

#### Verify After (Scheduled orders)

- ☐ **Customer checkout** (when the vendor is open): A **"Schedule for later"** option appears in the checkout flow. Screenshot.
- ☐ **Date range**: The furthest available date is approximately **48 hours from now** (not tomorrow only, not unlimited). Screenshot the date picker.
- ☐ **Slot interval**: Time slots are offered in **30-minute intervals** (e.g. 12:00, 12:30, 13:00). Screenshot the time picker.

#### Change — disable scheduled orders

1. Set **`AcceptsScheduledOrders`** back to `false`.
2. ☐ The "Schedule for later" option **disappears** from checkout. Screenshot.

#### Revert

Return the opening hours to their original values.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the "Closed now" state and scheduled-order slot configuration both propagate correctly.
- ❌ **Fail** if the customer can add to the basket or complete checkout during a closed trading period.
- ❌ **Fail** if the scheduled-order picker shows the wrong date range or wrong slot intervals.
- 🐛 **Bug-anyway** if "Closed now" is shown but the basket can still be modified. Log as **High**.

#### Edge cases to also try

- **Vendor with `PrepLeadTimeMinutes` = 45**: The earliest available scheduled slot should be at least 45 minutes from the current time, not immediately.
- **`MaxScheduleAheadHours` = 168** (the maximum — 7 days): Confirm the date picker allows selection up to 7 days ahead but not 8.

---

### TC-XP-024: Vendor Scheduled-Orders Settings — Schedule Picker Behaviour

**Goal**: Confirm that toggling `AcceptsScheduledOrders` and adjusting the slot interval and advance-booking window in Vendor Admin is immediately reflected in the schedule picker shown to customers at checkout.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

---

#### Before — capture the starting state

1. Confirm that **"Sushi Spot"** currently has `AcceptsScheduledOrders` = `false` (or note its current value).
2. As **the customer**, add Sushi Spot items to the basket and proceed to checkout.
3. **Take a screenshot** — confirm no "Schedule for later" option is shown.

#### Change — enable scheduled orders with 60-minute slots

1. In **Vendor Admin** (Sushi Spot), go to **Restaurant Management → Branch tab**.
2. Set:
   - **`AcceptsScheduledOrders`**: `true`
   - **`MaxScheduleAheadHours`**: `72`
   - **`PrepLeadTimeMinutes`**: `30`
   - **`ScheduledOrderSlotIntervalMinutes`**: `60`
3. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (60-minute slots)

- ☐ **Customer checkout**: A "Schedule for later" toggle or link is now available. Screenshot.
- ☐ **Date range**: Customer can select dates from now up to **72 hours** ahead. Screenshot.
- ☐ **Slot intervals**: Time slots are in **60-minute increments** (e.g. 13:00, 14:00, 15:00 — not 13:30). Screenshot.
- ☐ **Earliest slot**: The first available slot is at least **30 minutes from now** (due to `PrepLeadTimeMinutes`). Screenshot.

#### Change — switch to 15-minute slots

1. In **Vendor Admin**, change `ScheduledOrderSlotIntervalMinutes` to **15**.
2. Save.

#### Verify After (15-minute slots)

- ☐ **Customer checkout**: Time slots are now in **15-minute increments** (e.g. 13:00, 13:15, 13:30). Screenshot.

#### Change — disable scheduled orders

1. Set `AcceptsScheduledOrders` to **false**.
2. ☐ The "Schedule for later" option is gone. Screenshot.

#### Revert

Return settings to original values.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all slot configuration changes are reflected correctly in the checkout picker.
- ❌ **Fail** if the slot interval on the FE does not match the configured value.
- 🐛 **Bug-anyway** if the "earliest slot" does not respect `PrepLeadTimeMinutes`. Log as **High** (customer could place an order the kitchen cannot prepare in time).

---

### TC-XP-025: Vendor User Transfer / Delete — Login and Portal Access

**Goal**: Confirm that transferring or deleting a vendor user in System Admin immediately blocks that user's access to Vendor Admin, and that a new owner can log in with appropriate vendor context.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Vendor Admin (logged in as the user about to be transferred/deleted)

**See also**: TC-SA-040 through TC-SA-052 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. In **System Admin**, go to **Vendor Users** (`/Admin/VendorUsers`).
2. Find the test vendor user (e.g. `testvendor2.uat@wantfood.com`).
3. **Take a screenshot** of the user record showing their current vendor association.
4. Confirm the test vendor user can log in to **Vendor Admin** and see their restaurant's dashboard.

#### Change — transfer the user to a different vendor

1. In **System Admin**, find `testvendor2.uat@wantfood.com` and **transfer** them to a different vendor context (e.g. from "Pizza Roma" to "Sushi Spot").
2. Save.

#### Verify After (Transfer)

- ☐ **Vendor Admin login** as `testvendor2.uat@wantfood.com`: After re-logging in (or refreshing), the vendor context is **"Sushi Spot"** — not "Pizza Roma". Screenshot.
- ☐ **"Pizza Roma" vendor data**: The transferred user cannot see Pizza Roma's orders, menu, or settings. Screenshot.

#### Change — delete the vendor user

1. In **System Admin**, find `testvendor2.uat@wantfood.com` and click **Delete** / **Remove**.
2. Confirm the deletion.

#### Verify After (Delete)

- ☐ **Login attempt** as `testvendor2.uat@wantfood.com`: The user should be **blocked from accessing Vendor Admin**. They may see an access-denied page or be redirected to login. Screenshot.

> **Audit note** (back-end log only): The transfer and deletion should be logged by the back-end. Testers cannot verify this directly — if observable behaviour looks correct but something downstream seems wrong, flag a possible audit miss.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the transferred user sees the new vendor context and the deleted user is blocked.
- ❌ **Fail** if the deleted user can still log in and access any vendor data.
- ❌ **Fail** if the transferred user can still see the old vendor's data.
- 🐛 **Bug-anyway** if the error page for the blocked user is confusing or shows raw code. Log as **Low**.

#### Edge cases to also try

- **User deletes themselves** (if the admin UI allows a user to delete their own account) — should show an appropriate error.

---

### TC-XP-026: Admin User Add / Delete — New Admin Login and RBAC Scope

**Goal**: Confirm that adding a new admin user in System Admin allows that person to log in with the correct role-based permissions, and that deleting the user immediately blocks their access.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ A second browser window for the new admin user's login

**See also**: TC-SA-040 through TC-SA-052 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. In **System Admin**, go to **Users** (`/Admin/Users`).
2. **Take a screenshot** of the current admin user list.
3. Confirm there is no user with the email `newadmin.uat@wantfood.com`.

#### Change — add a new admin user

1. Click **Add User**.
2. Fill in:
   - **Email**: `newadmin.uat@wantfood.com`
   - **Role**: `Content Manager` (or the lowest-privilege role available, so testing is safe)
3. Save.

#### Verify After (Add)

- ☐ **New admin login**: Log in as `newadmin.uat@wantfood.com` via Entra ID. The System Admin dashboard loads. Screenshot.
- ☐ **RBAC scope**: As a Content Manager, confirm the user can access content management sections but **cannot** access high-privilege sections (e.g. Commission Tiers, Payment Settings). If accessing a restricted section shows a "403 Forbidden" or "Access Denied" page, that is correct. Screenshot.

#### Change — delete the new admin user

1. Switch back to the primary admin account.
2. In System Admin, find `newadmin.uat@wantfood.com` and **delete** the user.
3. Confirm deletion.

#### Verify After (Delete)

- ☐ **Login attempt** as `newadmin.uat@wantfood.com`: Access is **blocked**. The user sees an appropriate error. Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the new admin can log in with correct scoped access, and the deleted admin is immediately blocked.
- ❌ **Fail** if a deleted admin can still access the System Admin portal.
- 🐛 **Bug-anyway** if the "Access Denied" page for out-of-scope sections has no helpful message. Log as **Low**.

---

### TC-XP-027: Reindex Vendors / Rebuild Caches / Import ONS Postcodes / Integrity Check — Observable Customer FE Changes

**Goal**: Confirm that each Platform Tool in System Admin produces a verifiable, observable change on the customer front-end, and that the tools complete without errors.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-180 through TC-SA-185 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, search for **"margherita"** on the customer front-end. Screenshot the results.
2. Enter postcode **"SW1A 1AA"** in the address lookup. Confirm it resolves to a readable address (e.g. "Downing Street, Westminster"). Screenshot.

#### Tool 1 — Rebuild Caches

1. In **System Admin**, go to **Platform Tools** (`/Admin/Tools`).
2. Click **Rebuild Caches**.
3. Wait for the completion message. Screenshot.

#### Verify After (Rebuild Caches)

- ☐ **Tool completes**: A success message appears — no error, no timeout. Screenshot.
- ☐ **Customer front-end**: Hard-refresh the home page. The page loads correctly — no blank sections, no error messages, no missing vendor cards. Screenshot.
- ☐ **Speed**: The home page loads within **5 seconds** after a cache rebuild (slightly slower than normal is expected as the cache warms up). Note the approximate load time.

#### Tool 2 — Reindex Vendors

1. Click **Reindex Vendors**.
2. Wait for completion. Screenshot.

#### Verify After (Reindex Vendors)

- ☐ **Tool completes**: A success message. No error. Screenshot.
- ☐ **Customer search** (`/search?q=margherita`): Search results still return the same dishes as the "Before" screenshot (the reindex should not change results — only refresh them). Screenshot.

#### Tool 3 — Import ONS Postcodes

1. Click **Import ONS Postcodes** (if available and safe to run in UAT — check with your project manager first; this may import a large dataset).
2. Wait for completion.

#### Verify After (Import ONS Postcodes)

- ☐ **Tool completes** (or shows a progress indicator for a long-running import). Screenshot.
- ☐ **Postcode lookup**: Re-enter **"SW1A 1AA"** in the customer address lookup. It still resolves correctly. Screenshot.
- ☐ **New postcode** (if your project manager provides one that was previously unrecognised): Enter it; it should now resolve to an address.

#### Tool 4 — Integrity Check

1. Click **Integrity Check** (or equivalent).
2. Wait for completion.

#### Verify After (Integrity Check)

- ☐ **Tool completes**: The results page shows either "All checks passed" or a list of specific issues to review. Screenshot the results page.
- ☐ **No cascading failures**: The customer front-end continues to load correctly after the integrity check completes.
- ☐ **Any flagged issues**: If the integrity check flags issues (e.g. "3 dishes have no category"), log each one as a separate bug with severity **Medium** — the issues themselves are not part of this TC, but the integrity check tool should be surfacing them.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if all four tools complete without error and produce the expected observable changes.
- ❌ **Fail** if a tool times out, crashes, or produces an error message.
- 🐛 **Bug-anyway** if a tool shows a confusing or empty results page. Log as **Low**.

#### Edge cases to also try

- **Run two tools in quick succession** — confirm the second tool queues correctly and does not conflict with the first.

---

### TC-XP-028: Review Moderation — Flag / Resolve + Manual Rating Recalc

**Goal**: Confirm that flagging a review in System Admin hides it from the customer front-end, that resolving the flag restores it, and that the vendor and dish ratings are updated after a manual recalculation.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-100 (Flag Review), TC-SA-101 (Resolve Flagged Review), TC-SA-102 (Manual Rating Recalc) in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, navigate to the **"Pizza Roma"** vendor page and scroll to the **Reviews** section.
2. **Take a screenshot** of the current reviews — note the review content, the number of reviews, and the overall star rating (e.g. "4.3 ★ — 12 reviews").
3. Note the text of one specific review (e.g. a 5-star review saying "Best pizza in London!"). This is the review you will flag.

#### Change — flag the review

1. In **System Admin**, go to **Reviews** (`/Admin/Reviews`).
2. Find the "Best pizza in London!" review for Pizza Roma.
3. Click **Flag** (or **Moderate** → **Flag for removal**).
4. Confirm. **Take a screenshot** of the review in System Admin showing "Flagged" status.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Flag)

- ☐ **Customer vendor page — Reviews section**: The "Best pizza in London!" review **no longer appears**. Screenshot.
- ☐ **Review count**: The number of reviews shown has decreased by 1 (e.g. "11 reviews" if it was "12 reviews"). Screenshot.
- ☐ **Overall star rating**: The rating may change slightly if the flagged review was 5 stars. Note the new rating. Screenshot.

#### Change — run manual rating recalc

1. In **System Admin**, find the manual rating recalculation function (in Reviews section or Platform Tools).
2. Run the recalculation for **Pizza Roma**.

#### Verify After (Rating Recalc)

- ☐ **Customer vendor page rating**: The rating reflects the recalculation — it should be consistent with the remaining reviews. Screenshot.
- ☐ **Dish ratings** (if individual dish ratings are surfaced on the FE): Dishes that were rated in the flagged review may have updated ratings. Screenshot if dish-level ratings are visible.

#### Change — resolve the flagged review

1. In **System Admin**, find the flagged review and click **Resolve** (restore it as approved).
2. Run the rating recalculation again.

#### Verify After (Resolve)

- ☐ **Customer vendor page**: The "Best pizza in London!" review **reappears**.
- ☐ **Review count**: Returns to the original number (e.g. "12 reviews").
- ☐ **Rating**: Returns to the original rating (e.g. 4.3 ★). Screenshot.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if flagging hides, resolving restores, and ratings are correct after recalculation.
- ❌ **Fail** if a flagged review remains visible to customers.
- ❌ **Fail** if the rating does not update after a manual recalculation.
- 🐛 **Bug-anyway** if the review count changes but the star rating does not update. Log as **High**.

#### Edge cases to also try

- **Flag all reviews for a vendor** — the reviews section should show an empty state gracefully, not a zero-review page that looks like a new vendor.
- **Flag a review containing a bad word** (added in TC-XP-013) — both mechanisms should be independent. Flagging removes it; resolving it would restore it even if it contains the bad word (the moderation override).

---

### TC-XP-029: Content Asset Library Upload / Update / Delete — Hero Slides and Global Carousel Assets

**Goal**: Confirm that the content asset library correctly serves images to hero slides and Support Local entries, and that deleting an asset used by a live slide produces a graceful fallback (not a broken image). Also confirm that global hero carousel CSS/JS changes take effect on the customer front-end.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window

**See also**: TC-SA-110 through TC-SA-113 in 02-system-admin-uat.md.

---

#### Before — capture the starting state

1. As **the customer**, open the home page (`/`) and **take a screenshot** of the hero carousel. Note the image used on the first slide.
2. In **System Admin**, note which asset from the **Content Asset Library** (`/Admin/ContentAssetLibrary`) is linked to that hero slide.

#### Change — upload a new asset and update the hero slide

1. In **System Admin**, go to **Content → Asset Library** (`/Admin/ContentAssetLibrary`).
2. Upload a new clearly-distinctive test image (e.g. a bright yellow rectangle).
3. Go to **Content → Hero Slides** and edit the first hero slide.
4. Replace its background image with the newly uploaded asset.
5. Save.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Asset Update)

- ☐ **Home page carousel**: The first hero slide now shows the **bright yellow test image**. No broken image icon. Screenshot.

#### Change — delete the asset while it is in use

1. Go back to the **Content Asset Library** and **delete** the bright yellow test image (without updating the hero slide to use a different image first).
2. Confirm deletion.

#### Propagate

> Follow the [Propagation Cheat-Sheet](#propagation-cheat-sheet).

#### Verify After (Asset Deleted)

- ☐ **Home page carousel**: The hero slide either shows a **fallback placeholder image** or the slide is hidden — it should **not** show a broken image icon or cause a layout break. Screenshot.
- ☐ **No JavaScript errors** on the home page (if you are able to open the browser developer console and check — ask a technically confident colleague if needed). A broken asset should not cause the entire carousel to crash.

#### Global Hero Assets (CSS/JS/transitions)

1. In **System Admin**, go to **Content → Hero Assets** (`/Admin/ContentHeroAssets`).
2. Note the current **transition setting** (e.g. "Fade", "Slide left", animation duration).
3. Change the transition to a visibly different option (e.g. change duration from 500ms to 2000ms if configurable).
4. Save.

#### Verify After (Global Hero Assets)

- ☐ **Home page carousel**: The transition animation is visibly different (slower if you increased the duration). Screenshot or record a short screen video to capture the animation.

#### Cleanup

1. Revert the hero slide to its original image.
2. Return the carousel transition to its original setting.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the asset update is reflected correctly and a deleted-asset slide falls back gracefully.
- ❌ **Fail** if a deleted asset causes a broken image icon or carousel JavaScript crash.
- 🐛 **Bug-anyway** if there is no warning in System Admin when you attempt to delete an asset that is in use by a live slide. Log as **Medium** (this is a UX safety net, not a data corruption bug).

---

### TC-XP-030: Driver Invite / Remove — Driver Portal Access and Order Assignment

**Goal**: Confirm that inviting a driver in Vendor Admin grants them access to the Driver Portal, that removing a driver immediately revokes that access, and that mid-flight delivery handovers are handled appropriately.

**Personas / portals you need open at once**:

- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Driver Portal (`driver.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window (optional, for tracking)

**See also**: TC-VA-070 through TC-VA-077 in 03-vendor-admin-uat.md.

---

#### Before — capture the starting state

1. In **Vendor Admin** (Pizza Roma), go to **Drivers** (`/Admin/Drivers`).
2. **Take a screenshot** of the current driver list for Pizza Roma.
3. Confirm the test driver account (`driver.uat@wantfood.com`) is either currently invited or not invited. Note the status.

#### Change — invite the driver

1. In **Vendor Admin**, click **Invite Driver**.
2. Enter email: `driver.uat@wantfood.com` (or a test driver email provided by your project manager).
3. Send the invitation.
4. **Take a screenshot** of the confirmation.

#### Verify After (Invite)

- ☐ **Driver invitation email**: An invitation email arrives at `driver.uat@wantfood.com`. It contains a clear call-to-action link or instructions to accept the invite. Screenshot the email.
- ☐ **Driver Portal login**: The driver logs in to the Driver Portal via Entra ID. They can see the **Pizza Roma** delivery assignment queue. Screenshot.
- ☐ **Order assignment availability**: An active test order for Pizza Roma should appear as available for the driver to accept. (If no active orders are in progress, confirm the order queue is visible and not blocked.) Screenshot.

#### Mid-flight handover check

1. Ensure there is an **active delivery in progress** by the test driver for Pizza Roma (accept an order in the driver portal).
2. While the delivery is active, switch to **Vendor Admin** and attempt to **remove** the driver.

#### Verify After (Mid-flight)

- ☐ **Remove during active delivery**: The system should either:
  - **Block the removal** with a message ("This driver has an active delivery in progress — removal is not permitted until it completes"), or
  - **Allow the removal** but the active delivery continues to completion (the driver does not lose mid-flight visibility).
- ☐ Whichever behaviour occurs, it should be **clearly communicated** in the UI and should not cause a blank error screen. Screenshot.

#### Change — remove the driver (after delivery completes)

1. Once the delivery is complete, in **Vendor Admin**, **remove** `driver.uat@wantfood.com` from Pizza Roma.
2. Confirm the removal.

#### Verify After (Remove)

- ☐ **Driver Portal login** as `driver.uat@wantfood.com`: The driver sees **no deliveries available for Pizza Roma**, or is shown a message that they are not assigned to this restaurant. They should **not** be able to accept new Pizza Roma orders. Screenshot.
- ☐ **Customer tracking page**: If the customer front-end has a live tracking page, it should not show the removed driver as still assigned.

#### Pass / Fail / Bug-anyway

- ✅ **Pass** if the invite grants access, the removal blocks new order acceptance, and mid-flight deliveries complete without data loss.
- ❌ **Fail** if the invitation email does not arrive, or if the driver cannot log in after being invited.
- ❌ **Fail** if a removed driver can still accept new orders for the vendor.
- 🐛 **Bug-anyway** if removing a driver mid-flight causes the customer tracking page to crash or show an error. Log as **High**.

#### Edge cases to also try

- **Driver invited to two vendors**: Remove them from one vendor but not the other. They should still have access to the second vendor's deliveries.
- **Invite a driver with an invalid email format**: The Vendor Admin form should show a validation error before sending.

#### Common pitfalls

- *"The driver invitation email went to spam"* → check the Junk folder. All WantFood emails come from `noreply@wantfood.com`.
- *"The driver logged in but sees no orders"* → check that there is an active order in a state the driver can accept. Orders in "New" or "Confirmed" state should appear; orders in "Delivered" state will not.

---

## Summary and Next Steps

### What this document covers

This document provides **30 paired test cases** (TC-XP-001 to TC-XP-030) that verify the cross-portal consequences of admin changes on the WantFood platform. Each test case follows the Before → Change → Propagate → Verify After pattern to ensure that:

- **Customer-facing surfaces** (home page, search, cuisine pages, vendor pages, basket, and checkout) reflect admin changes correctly and within the expected 30-second propagation window.
- **Basket and checkout integrity** is maintained — orphaned items are warned about or removed, and customers cannot checkout with items that are unavailable, from deactivated vendors, or at outdated prices.
- **Transactional emails** (onboarding invitations, offer notifications) are sent correctly.
- **Driver portal access** is gated correctly by vendor-driver invite/remove actions.
- **Platform tools** (Rebuild Caches, Reindex Vendors, Import ONS Postcodes, Integrity Check) complete without error and produce verifiable observable changes.

### Areas not covered by this document

The following areas have dedicated UAT documents and are **not retested here**:

| Area | Where to test it |
|------|-----------------|
| Admin-side click-paths for each change | 02-system-admin-uat.md, 03-vendor-admin-uat.md |
| Driver portal own workflow (shifts, route, delivery status) | 04-driver-portal-uat.md |
| Customer-side order placing, tracking, and reviews | 05-customer-frontend-uat.md |
| End-to-end order saga (order through to delivery) | 06-order-saga-uat.md |
| Automation jobs and scheduled processes | 07-automation-jobs-uat.md |

### When you find a bug

1. **Stop and take a screenshot** immediately.
2. Open the **Monday.com UAT board** → **Cross-Portal Impact** group.
3. Log the bug with:
   - TC reference (e.g. "TC-XP-007")
   - The admin change you made (portal, page, field, value)
   - The surface where the problem appeared (customer FE, basket, email, etc.)
   - The propagation steps you completed before logging
   - Screenshots: one of the admin "saved" confirmation, one of the incorrect customer FE state
4. Assign severity using the guide in [00-introduction.md](00-introduction.md#severity-definitions). When in doubt, **choose the higher severity**.

### Recommended test order

For maximum efficiency, run the test cases in the following groupings:

| Session | TCs | Focus |
|---------|-----|-------|
| Session 1 (morning) | TC-XP-001 to TC-XP-008 | Vendor and menu changes |
| Session 2 (afternoon) | TC-XP-009 to TC-XP-014 | Content, taxonomy, and discovery |
| Session 3 (morning) | TC-XP-015 to TC-XP-022 | Offers, commissions, payments, and delivery |
| Session 4 (afternoon) | TC-XP-023 to TC-XP-030 | Trading hours, users, tools, reviews, and drivers |

Always clear your browser cache between sessions.

### Sign-off

Once all 30 test cases are complete and all **Critical** and **High** severity bugs are resolved:

- Review the summary in **[08-signoff.md](08-signoff.md)**
- Confirm with your project manager that cross-portal impact testing is signed off
- Update the Monday.com board status for the **Cross-Portal Impact** group

---

*End of Cross-Portal Impact UAT — Test Scripts*
