# Vendor Admin UAT — Test Scripts

## Purpose

This document contains test scripts for the **Vendor Admin portal**. The Vendor Admin portal is used by restaurant owners, managers, and branch operators to manage their restaurant profile, menus, orders, drivers, promotions, and delivery/payment settings.

**Target users**: Restaurant owners, vendor managers, branch operators

---

## Prerequisites

Before testing the Vendor Admin portal, ensure you have:

- ✅ Access to the Vendor Admin UAT environment URL
- ✅ Vendor Admin test account credentials (`vendor.uat@wantfood.com`)
- ✅ A test vendor application that has been approved (see [02-system-admin-uat.md](02-system-admin-uat.md#vendor-applications))
- ✅ Browser and device ready (see [00-introduction.md](00-introduction.md#supported-browsers-and-devices))
- ✅ Monday.com board set up (see [01-monday-setup.md](01-monday-setup.md))
- ✅ Screenshot/recording tool ready

---

## Test Setup: Pre-existing Test Data

The following test data should be available in the UAT environment before starting:

| Item | Description |
|------|-------------|
| **Test Vendor** | "Test Restaurant 1" — an approved and onboarded vendor |
| **Test Branch** | "Test Restaurant 1 — City Centre" — a branch linked to the test vendor |
| **Test Menu** | A draft menu with at least 2 categories and 5 dishes |
| **Test Driver** | A test driver account (`driver.uat@wantfood.com`) invited to the test branch |
| **Test Customer Order** | A pending order placed by `customer.uat@wantfood.com` |

If any of this data is missing, contact your project manager before starting.

---

## Table of Contents

1. [Vendor Application and Onboarding](#vendor-application-and-onboarding)
2. [Vendor Context Switching](#vendor-context-switching)
   - [TC-VA-012: Switch Between Branches](#tc-va-012-switch-between-branches)
   - [TC-VA-013: Taking Orders Toggle — Dashboard](#tc-va-013-taking-orders-toggle--dashboard)
   - [TC-VA-014: Scheduled Orders Quick-Toggle — Dashboard](#tc-va-014-scheduled-orders-quick-toggle--dashboard)
   - [TC-VA-015: Branch Context Card — Live Order Kanban](#tc-va-015-branch-context-card--live-order-kanban)
3. [Restaurant Management](#restaurant-management)
4. [Trading Hours and Scheduled Orders](#trading-hours-and-scheduled-orders)
5. [Menu Management](#menu-management)
   - [TC-VA-037A: Manage dish upsell links](#tc-va-037a-manage-dish-upsell-links)
   - [Dish Variants and Modifiers](#dish-variants-and-modifiers)
   - [Menu Scanning (AI Import)](#menu-scanning-ai-import)
6. [Live Order Kanban](#live-order-kanban)
7. [All Orders View](#all-orders-view)
8. [Driver Management](#driver-management)
9. [Team Management](#team-management)
10. [Promotions and Offers](#promotions-and-offers)
11. [Reviews](#reviews)
12. [Delivery Cost Configuration](#delivery-cost-configuration)
13. [Payment Settings](#payment-settings)

---

## Vendor Application and Onboarding

### TC-VA-001: Submit Vendor Application

**Given**: You are a new vendor who does not yet have an account

**Steps**:
1. Navigate to the Vendor Admin UAT URL
2. Click **"Apply to Join"** or navigate to the application page
3. Fill in all required fields:
	- Business name
	- Business address
	- Contact name, email, phone number
	- Cuisine type (select from list)
	- Brief description of the restaurant
4. Submit the application

**Expected Result**:
- An "Application Received" confirmation page or message is shown
- You receive a confirmation email to the email address provided
- The application is now visible to System Admin for review

**Pass Criteria**:
- ✅ Application submits without errors
- ✅ Confirmation page displays
- ✅ Confirmation email received (if verifiable)

**Edge Cases**:
- Missing required fields → should show validation errors on each field
- Invalid email format → should show email validation error
- Duplicate business name → may be allowed (check business rules)
- Very long text in description → should truncate or limit with validation

---

### TC-VA-002: Application Success Page

**Given**: You have just submitted a vendor application

**Steps**:
1. Observe the application success/confirmation page

**Expected Result**:
- A clear success message is displayed: "Application received" or similar
- Instructions for next steps are shown (e.g., "We will review your application within X days")
- Contact details for queries are provided
- You cannot re-submit the same application from this page

**Pass Criteria**:
- ✅ Success page displays correct message
- ✅ Next steps information is present

**Edge Cases**:
- Navigating back to the application form → should not allow resubmission without refreshing

---

### TC-VA-003: Complete Vendor Onboarding (After Application Approval)

**Given**: Your vendor application has been approved (by System Admin) and you have received an onboarding invitation email

**Steps**:
1. Open the invitation email
2. Click the invitation link
3. You are taken to the **Onboarding Landing Page**
4. Click through to the **Onboarding Registration Page**
5. Set up your account (password, profile details)
6. Complete the registration
7. You are taken to the **Onboarding Welcome Page**

**Expected Result**:
- The invitation link works and opens the correct page
- Registration form accepts your details
- After completing registration, you are taken to the welcome/confirmation page
- You can now log in to Vendor Admin with your new credentials
- Your vendor context is pre-loaded (your restaurant is linked to your account)

**Pass Criteria**:
- ✅ Invitation link works
- ✅ Registration completes successfully
- ✅ Welcome page displays
- ✅ Login works after registration

**Edge Cases**:
- Invitation link expired → should show "Link expired, contact support" message
- Invitation link used twice → should show "Already registered" or redirect to login
- Password does not meet requirements → should show password policy validation

---

### TC-VA-004: Access Vendor Admin Without Onboarding Complete

**Given**: You are a vendor user who has registered but not completed onboarding

**Steps**:
1. Log in to Vendor Admin
2. Observe the state of the dashboard

**Expected Result**:
- You are shown the **No Vendor Context** page or an "onboarding not complete" message
- You cannot access full admin features until onboarding is complete
- Guidance is provided on how to complete onboarding

**Pass Criteria**:
- ✅ No Vendor Context page displays
- ✅ Full admin features are blocked appropriately

**Edge Cases**:
- Accessing admin URLs directly → should redirect to No Vendor Context or login

---

## Vendor Context Switching

### TC-VA-010: View Vendor Dashboard After Login

**Given**: You are logged in as a vendor admin user with a single vendor context

**Steps**:
1. Log in to Vendor Admin
2. Observe the dashboard

**Expected Result**:
- The dashboard displays your vendor's name prominently
- Key metrics are shown: today's orders, pending orders, total revenue (if available)
- Navigation menu is accessible: Menu, Orders, Drivers, Offers, Reviews, Settings
- A **Branch Context card** is visible in the top area showing:
  - Current branch name
  - **Taking Orders** toggle (reflects `IsAcceptingOrders` for the current branch)
  - **Scheduled Orders** toggle (reflects `AcceptsScheduledOrders` for the current branch)
- The branch switcher dropdown is always visible — even when the vendor has only one branch

**Pass Criteria**:
- ✅ Vendor name is correctly shown
- ✅ Dashboard widgets load
- ✅ Navigation is accessible
- ✅ Branch Context card visible with both toggles
- ✅ Branch switcher visible for single-branch vendors

**Edge Cases**:
- Dashboard with no orders → should show "No orders today" gracefully
- Dashboard with many orders → metrics should display correctly

---

### TC-VA-011: Switch Between Vendors (If Multiple Vendors)

**Given**: You are logged in as a vendor admin user associated with more than one vendor

**Steps**:
1. Locate the vendor/context switcher in the navigation (top bar or sidebar)
2. Click to switch to a different vendor
3. Observe the dashboard and navigation after switching

**Expected Result**:
- The dashboard updates to show the newly selected vendor's data
- All menu items, orders, and settings reflect the new vendor context
- The vendor name in the navigation updates

**Pass Criteria**:
- ✅ Context switches correctly
- ✅ Dashboard data updates
- ✅ No data from the previous vendor is visible

**Edge Cases**:
- Switching while on the order kanban → should refresh kanban with new vendor's orders
- Switching to a vendor with no branches → should show appropriate empty state

---

### TC-VA-012: Switch Between Branches

**Given**: You are logged in as a vendor admin user and your vendor has multiple branches

**Steps**:
1. Locate the branch switcher in the navigation
2. Switch to a different branch
3. Observe the dashboard after switching

**Expected Result**:
- The dashboard updates to show the selected branch's data
- Orders, drivers, and settings reflect the selected branch
- The branch name in the navigation updates
- The **Taking Orders** and **Scheduled Orders** toggles in the Branch Context card update to reflect the newly selected branch's state

**Pass Criteria**:
- ✅ Branch context switches correctly
- ✅ Branch-specific data is displayed
- ✅ Taking Orders toggle reflects new branch's `IsAcceptingOrders`
- ✅ Scheduled Orders toggle reflects new branch's `AcceptsScheduledOrders`

**Edge Cases**:
- Switching to a branch with no orders → should show "No orders" gracefully
- Switching to a branch with no assigned drivers → should show "No drivers" gracefully
- **Single-branch vendor**: The branch switcher dropdown must still be visible (even though only one branch can be selected), showing the current branch name

---

### TC-VA-013: Taking Orders Toggle — Dashboard

**Given**: You are on the Vendor Admin Dashboard

**Steps**:
1. Locate the **Branch Context card** (showing current branch name)
2. Note the current state of the **Taking Orders** toggle
3. Click the toggle to turn **Taking Orders OFF** (if currently on)
4. Observe the response — the toggle should update immediately
5. Open the customer-facing front-end for the same branch and search for the restaurant
6. Confirm the branch is shown as **closed / not accepting orders**
7. Return to VendorAdmin Dashboard and toggle **Taking Orders back ON**
8. Verify the customer front-end shows the branch as **open / accepting orders** again

**Expected Result**:
- Toggling off disables order acceptance for the current branch only (other branches unaffected)
- Customer front-end reflects the change (branch hidden from search or shown as unavailable)
- Toggling back on restores availability
- No page reload required; the toggle is async

**Pass Criteria**:
- ✅ Toggle responds immediately with visual feedback
- ✅ Branch `IsAcceptingOrders` state persists after page reload
- ✅ Customer front-end reflects the change
- ✅ Other branches (if any) are unaffected

**Edge Cases**:
- Network error during toggle → toggle should revert to previous state and show an error
- Toggling off while an order is in-flight → existing orders continue; new orders are blocked

**Cross-portal verification**: Run **TC-XP-002** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md).

---

### TC-VA-014: Scheduled Orders Quick-Toggle — Dashboard

**Given**: You are on the Vendor Admin Dashboard with `AcceptsScheduledOrders` already **enabled** in the Branch Settings

**Steps**:
1. Locate the **Branch Context card**
2. Note the **Scheduled Orders** toggle is ON
3. Click the toggle to turn **Scheduled Orders OFF**
4. Observe: the toggle updates; a success/error notification appears
5. Navigate to **Restaurant** → **Manage Restaurant** → **Branch** tab
6. Confirm `AcceptsScheduledOrders` shows as disabled there too
7. Return to the Dashboard and re-enable the toggle
8. Confirm the Branch tab shows `AcceptsScheduledOrders` as enabled again

**Expected Result**:
- Dashboard quick-toggle and Branch Settings tab stay in sync
- Toggling off removes the scheduled-order slot picker from the customer front-end
- Toggling on restores it

**Pass Criteria**:
- ✅ Dashboard toggle and Branch Settings are in sync
- ✅ Change persists after page reload
- ✅ Customer slot picker appears/disappears as expected

**Edge Cases**:
- Toggling while the Branch Settings tab is open in another tab → refreshing that tab should show the updated value
- Disabling scheduled orders while a future scheduled order is pending → existing order is unaffected

**Cross-portal verification**: Run **TC-XP-024** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md).

---

### TC-VA-015: Branch Context Card — Live Order Kanban

**Given**: You are on the **Live Orders / Kanban** page

**Steps**:
1. Navigate to **Orders** → **Live Orders**
2. Locate the **Branch Context card** (same as on the Dashboard)
3. Verify both **Taking Orders** and **Scheduled Orders** toggles are present
4. Toggle **Taking Orders OFF** from the Kanban page
5. Confirm the toggle updates and a notification appears
6. Navigate back to the Dashboard — confirm the Dashboard toggle reflects the same OFF state

**Expected Result**:
- The Branch Context card with both toggles is present on the Kanban page as well as the Dashboard
- Toggling on the Kanban page has the same effect as toggling on the Dashboard
- State is shared — changing it on one page is reflected on all others

**Pass Criteria**:
- ✅ Branch Context card visible on Kanban page
- ✅ Both toggles present and functional
- ✅ Toggle state is consistent between Dashboard and Kanban

**Edge Cases**:
- Rapid toggle (ON/OFF/ON in quick succession) → final state should be persisted correctly
- Toggling while orders are actively being managed → toggle should not interfere with order actions

---

## Restaurant Management

### TC-VA-020: View Manage Restaurant Page

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Restaurant** → **Manage Restaurant** or similar
2. Observe the restaurant management page

**Expected Result**:
- Restaurant details are displayed:
  - Business name, address, phone number, email
  - Opening hours
  - Cuisine types
  - Delivery zone configuration
  - Description / about text
- An "Edit" or "Update" button is visible

**Pass Criteria**:
- ✅ All restaurant details are displayed
- ✅ Edit functionality is accessible

**Edge Cases**:
- No opening hours set → should show "Not configured" or prompt to set
- No delivery zone set → should show "Not configured" or prompt to set

---

### TC-VA-021: Update Branch Details

**Given**: You are viewing the Manage Restaurant page

**Steps**:
1. Click **"Edit"** or **"Update Branch"**
2. Modify one or more fields (e.g., phone number, opening hours, description)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Restaurant updated successfully"
- The updated information is reflected on the Manage Restaurant page
- Updated information is visible on the customer front-end (if applicable)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Front-end reflects updates

**Edge Cases**:
- Invalid phone format → should show validation error
- Required fields left blank → should enforce validation
- Setting opening hours that overlap (e.g., 09:00–22:00 and 10:00–15:00) → should validate or warn

**Cross-portal verification**: After completing this test, run **TC-XP-002** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Trading Hours and Scheduled Orders

The branch model stores `OpeningHoursJson` (per-day open/close times), `AcceptsScheduledOrders`, `MaxScheduleAheadHours` (1–168 hours), `PrepLeadTimeMinutes`, and `ScheduledOrderSlotIntervalMinutes` (15, 30, or 60 minutes). All settings are editable on the Branch tab of `/Admin/VendorManage`.

### TC-VA-022: Configure Trading Hours (Opening Hours)

**Given**: You are logged in as a Vendor Admin user and are viewing the Branch tab of the Manage Restaurant page

**Steps**:
1. Navigate to **Restaurant** → **Manage Restaurant** → **Branch** tab
2. Locate the **Opening Hours** section
3. Enable Monday–Friday and set open: **09:00**, close: **22:00**
4. Enable Saturday and set open: **10:00**, close: **23:00**
5. Close Sunday entirely (toggle off)
6. Set Friday with a midnight rollover: open **22:00**, close **02:00** (next day)
7. Click **"Save"**

**Expected Result**:
- A success message appears: "Branch updated successfully"
- Opening hours are saved for all enabled days
- Sunday displays as "Closed"
- The Friday midnight rollover (22:00–02:00) is accepted without a validation error
- On the customer front-end the branch shows as open or closed at times consistent with the saved hours

**Pass Criteria**:
- ✅ Monday–Saturday opening hours saved correctly
- ✅ Sunday shows as "Closed"
- ✅ Midnight rollover accepted and stored correctly
- ✅ Success message displayed

**Edge Cases**:
- Setting close time equal to open time (e.g., 12:00–12:00) → should show validation error
- Setting close time before open time without midnight rollover (e.g., 09:00–08:00) → should show validation error or require midnight-rollover confirmation
- Leaving open/close fields blank on an enabled day → should enforce required field validation
- All days closed → branch should appear as permanently closed on the customer front-end

**Cross-portal verification**: After completing this test, run **TC-XP-023** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-023: Configure Scheduled Orders

**Given**: You are logged in as a Vendor Admin user and are viewing the Branch tab of the Manage Restaurant page

> **Note**: `AcceptsScheduledOrders` can also be toggled quickly from the **Dashboard** (TC-VA-014) and the **Kanban page** (TC-VA-015) without navigating to the Branch settings tab. The Branch settings tab is the only place to configure `MaxScheduleAheadHours`, `PrepLeadTimeMinutes`, and `ScheduledOrderSlotIntervalMinutes`.

**Steps**:
1. Navigate to **Restaurant** → **Manage Restaurant** → **Branch** tab
2. Locate the **Scheduled Orders** settings
3. Toggle **"Accept Scheduled Orders"** on
4. Set **Max Schedule Ahead Hours** to **24**
5. Set **Prep Lead Time (Minutes)** to **30**
6. Set **Slot Interval (Minutes)** to **30**
7. Click **"Save"** and confirm success
8. Repeat, setting Max Schedule Ahead to **1** (minimum) — confirm accepted
9. Repeat, setting Max Schedule Ahead to **168** (maximum, 7 days) — confirm accepted
10. Attempt to set Max Schedule Ahead to **0** — confirm validation error
11. Attempt to set Max Schedule Ahead to **169** — confirm validation error

**Expected Result**:
- AcceptsScheduledOrders toggle is saved as enabled
- Values 24, 1, and 168 are all accepted without error
- Values 0 and 169 are rejected with a clear validation message
- PrepLeadTimeMinutes and ScheduledOrderSlotIntervalMinutes are stored correctly

**Pass Criteria**:
- ✅ AcceptsScheduledOrders toggle saved as enabled
- ✅ MaxScheduleAheadHours boundary values (1 and 168) accepted
- ✅ Out-of-range values (0 and 169) rejected with a validation error
- ✅ PrepLeadTimeMinutes and ScheduledOrderSlotIntervalMinutes saved correctly
- ✅ Success message displayed

**Edge Cases**:
- Toggling AcceptsScheduledOrders off → scheduled-order slot picker should disappear on the customer front-end
- Slot interval set to 15 minutes → customer sees a slot every 15 minutes in the schedule picker
- Slot interval set to 60 minutes → customer sees hourly slots only
- PrepLeadTimeMinutes set to 0 → should be allowed (no lead-time buffer)

**Cross-portal verification**: After completing this test, run **TC-XP-024** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-024: Trading Hours — Validation Edge Cases

**Given**: You are logged in as a Vendor Admin user and are viewing the Opening Hours configuration

**Steps**:
1. Navigate to **Restaurant** → **Manage Restaurant** → **Branch** tab
2. Enable a day, then delete the close-time value so it is blank → attempt to save
3. Type letters into a time field (if the browser allows it) → attempt to save
4. Set close time to the same minute as open time (e.g., both set to 10:00) → attempt to save
5. Observe the save-failure UX: does the form highlight the failing field? Does it scroll to it?
6. Fix each error in turn and confirm the form saves successfully once all errors are resolved

**Expected Result**:
- Each invalid state produces a clear, in-line validation error on the affected field
- The form does not submit until all validation errors are resolved
- After fixing all errors and resubmitting, the save succeeds with a success message

**Pass Criteria**:
- ✅ Blank required time field blocks save with a clear error
- ✅ Invalid time format is rejected
- ✅ Close time equal to open time is rejected
- ✅ Error messages are visible and specific to the failing field
- ✅ Form highlights or scrolls to the first error

**Edge Cases**:
- All fields valid except one → only the failing field shows an error; valid fields are unaffected
- Submitting via keyboard (Tab to the Save button, then Enter) → validation fires the same as a mouse click

---

## Menu Management

### TC-VA-030: View Menu Builder List

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Menu** → **Menu Builder** or **Menus**
2. Observe the list of menus

**Expected Result**:
- A list displays all menus with columns: menu name, status (Active/Draft/Archived), last updated, actions
- You can create a new menu
- You can edit or delete existing menus

**Pass Criteria**:
- ✅ All menus are listed
- ✅ Menu status is accurate
- ✅ Actions (Create, Edit, Publish, Delete) are visible

**Edge Cases**:
- No menus exist → should display "No menus" message with a prompt to create one
- Archived menus → should be distinguishable from active menus

---

### TC-VA-031: Create Menu

**Given**: You are viewing the Menu Builder list

**Steps**:
1. Click **"Create Menu"** or **"New Menu"**
2. Enter a menu name (e.g., "Lunch Menu")
3. Select availability (e.g., all-day, lunch only, dinner only)
4. Click **"Save"** or **"Create"**

**Expected Result**:
- A success message appears: "Menu created successfully"
- The new menu appears in the Menu Builder list as a draft
- You are redirected to the menu editor where you can add categories and dishes

**Pass Criteria**:
- ✅ Menu is created
- ✅ Success message displayed
- ✅ Menu appears in the list

**Edge Cases**:
- Duplicate menu name → may be allowed (check business rules)
- Missing required fields → should enforce validation

---

### TC-VA-032: Open Menu Editor

**Given**: You are viewing the Menu Builder list

**Steps**:
1. Click on a menu to open it in the editor
2. Observe the menu editor interface

**Expected Result**:
- The menu editor opens showing:
  - Menu name and availability settings
  - List of categories (if any exist)
  - List of dishes within each category
  - Buttons to add categories and dishes
- The menu can be published from this page

**Pass Criteria**:
- ✅ Menu editor loads correctly
- ✅ Existing categories and dishes are displayed
- ✅ Add/Edit/Delete controls are visible

**Edge Cases**:
- Empty menu → should prompt to add categories or dishes
- Menu with many categories → should scroll or paginate correctly

---

### TC-VA-033: Create Menu Category

**Given**: You are in the menu editor for an existing menu

**Steps**:
1. Click **"Add Category"** or **"New Category"**
2. Enter a category name (e.g., "Starters")
3. Enter an optional description
4. Click **"Save"**

**Expected Result**:
- A success message appears: "Category created successfully"
- The new category appears in the menu editor
- The category is visible on the customer front-end when the menu is published

**Pass Criteria**:
- ✅ Category is created
- ✅ Success message displayed
- ✅ Category appears in the editor

**Edge Cases**:
- Duplicate category name in same menu → should show error or allow (check business rules)
- Missing category name → should enforce validation

---

### TC-VA-034: Update Menu Category

**Given**: You are in the menu editor and have at least one category

**Steps**:
1. Click **"Edit"** next to a category
2. Modify the category name or description
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Category updated successfully"
- The updated category name/description is reflected in the editor
- Updated information is visible on the customer front-end (if menu is published)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Clearing required name field → should enforce validation

---

### TC-VA-035: Delete Menu Category

**Given**: You are in the menu editor and have at least one category

**Steps**:
1. Click **"Delete"** next to a category
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Category deleted successfully"
- The category and all its dishes are removed from the editor
- The category is removed from the customer front-end (if menu is published)

**Pass Criteria**:
- ✅ Category is deleted
- ✅ Success message displayed
- ✅ Dishes within the category are also removed or handled

**Edge Cases**:
- Deleting a category with dishes → should warn that all dishes will also be deleted
- Deleting the only category → menu will be empty

---

### TC-VA-036: Create Dish

**Given**: You are in the menu editor and have at least one category

**Steps**:
1. Click **"Add Dish"** within a category
2. Enter dish details:
	- Dish name (required)
	- Description (optional)
	- Price (required)
	- Category assignment (required): select one or more categories from the Select2 multi-select
	- Dietary information (optional): vegetarian, vegan, gluten-free, allergens
	- Image (optional)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Dish created successfully"
- The new dish appears in every selected category in the editor
- The dish is visible on the customer front-end when the menu is published

**Pass Criteria**:
- ✅ Dish is created
- ✅ Success message displayed
- ✅ Dish appears in all selected categories
- ✅ Category selector renders as a Select2 control (not a native browser multi-select)

**Edge Cases**:
- Missing dish name or price → should enforce validation
- No category selected → should block save and show validation because at least one category is required
- Negative price → should show validation error
- Price of £0.00 → should warn or allow (check business rules)
- Dish name exceeds character limit → should show validation error
- Image upload fails → should show error and allow retry

**Cross-portal verification**: After completing this test, run **TC-XP-005** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-037: Update Dish

**Given**: You are in the menu editor and have at least one dish

**Steps**:
1. Click **"Edit"** next to a dish
2. Modify one or more fields (e.g., price, description, image, category assignments)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Dish updated successfully"
- The updated dish details are reflected in the editor
- If category assignments were changed, the dish appears in added categories and is removed from unselected categories
- Updated information is visible on the customer front-end (if menu is published)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Updating price for a dish that's in an active order → system should handle gracefully (order uses price snapshot)
- Invalid price format → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-005** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-037A: Manage dish upsell links

**Given**: You are in the menu editor and the menu has at least two dishes

**Steps**:
1. In a category row, click the **upsell link icon** next to the dish settings icon.
2. Observe the **Upsell dishes** off-canvas.
3. Confirm the table is grouped by category and existing upsell links for this dish are already ticked.
4. Tick or untick dishes in the grouped table, then click **Auto-suggest** and review the selected set.
5. In **Also apply to category**, leave **This dish only** selected and click **Save**.
6. Reopen the same dish upsell panel and confirm your selection persisted.
7. Select a category in **Also apply to category**, click **Save**, and confirm the bulk-apply prompt.

**Expected Result**:
- The upsell panel opens without forcing dish-edit mode.
- Category groups are clear and each category appears once in the grouped list.
- The current dish save applies immediately.
- If you choose a category in **Also apply to category**, the same upsell set is applied to the other dishes in that category after confirmation.

**Pass Criteria**:
- ✅ Existing upsell links are preselected when reopening the panel
- ✅ Category grouping is stable (no duplicated group headings for the same category)
- ✅ Saving with **This dish only** updates only the current dish
- ✅ Saving with a selected category updates additional dishes in that category

**Edge Cases**:
- Selecting the same dish as its own upsell should be blocked.
- Choosing a category with no other dishes should save the current dish and show a non-error informational result.
- Cancelling the bulk-apply confirmation should still keep the current dish save.

**Cross-portal verification**: After completing this test, run **TC-FE-022** in [05-customer-frontend-uat.md](05-customer-frontend-uat.md#tc-fe-022-dish-upsell-ordering-and-visibility) to confirm upsell cards render correctly for customers.

---

### TC-VA-038: Delete Dish

**Given**: You are in the menu editor and have at least one dish

**Steps**:
1. Click **"Delete"** next to a dish
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Dish deleted successfully"
- The dish is removed from the category in the editor
- The dish is removed from the customer front-end (if menu is published)

**Pass Criteria**:
- ✅ Dish is deleted
- ✅ Success message displayed

**Edge Cases**:
- Deleting a dish that's in an active order → system should handle gracefully (order retains dish snapshot)
- Deleting the only dish in a category → category remains but is empty

**Cross-portal verification**: After completing this test, run **TC-XP-005** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Dish Variants and Modifiers

The dish create/update form in Menu Builder supports **Variants** (Name, Price, IsDefault, DisplayOrder, Calories) and **Modifier Groups** (Name, SelectionType: 1 = Single / 2 = Multiple, MinSelections, MaxSelections, IsRequired) each containing **Modifiers** (Name, PriceDelta, CaloriesDelta, IsAvailable, IsDefault, DisplayOrder).

### TC-VA-095: Add Variants to a Dish

**Given**: You are creating or editing a dish in Menu Builder — use the "Margherita" pizza as the test dish

**Steps**:
1. Open the dish form for "Margherita" in the Menu Builder
2. Locate the **Variants** section
3. Add three variants in this order:
   - **Regular** — Price: £9.00, Calories: 800, IsDefault: **yes**, DisplayOrder: 1
   - **Large** — Price: £12.00, Calories: 1100, IsDefault: no, DisplayOrder: 2
   - **Family** — Price: £16.00, Calories: 1600, IsDefault: no, DisplayOrder: 3
4. Confirm only "Regular" is marked as default
5. Click **"Save"**

**Expected Result**:
- A success message appears
- All three variants (Regular £9.00, Large £12.00, Family £16.00) are saved and visible in the dish editor
- "Regular" is marked as the default variant
- Display order is respected (Regular → Large → Family)

**Pass Criteria**:
- ✅ All three variants saved
- ✅ Default variant is "Regular"
- ✅ Prices are £9.00, £12.00, and £16.00
- ✅ Display order is correct

**Edge Cases**:
- Marking two variants as default simultaneously → should allow only one default; selecting a second default should clear the first
- Saving a variant with a blank name → should enforce required field validation
- Saving a variant with a negative price → should show validation error
- Removing the only default variant → should require a new default to be selected before save

---

### TC-VA-096: Add Modifier Group to a Dish

**Given**: You are creating or editing the "Margherita" dish in Menu Builder

**Steps**:
1. Open the dish form for "Margherita" in the Menu Builder
2. Locate the **Modifier Groups** section and click **"Add Modifier Group"**
3. Configure the group as follows:
   - **Name**: "Toppings"
   - **SelectionType**: Multiple
   - **MinSelections**: 0
   - **MaxSelections**: 3
   - **IsRequired**: no (optional group)
4. Add three modifiers to the "Toppings" group:
   - **Extra Cheese** — PriceDelta: +£1.00, CaloriesDelta: +80, IsAvailable: yes, IsDefault: no, DisplayOrder: 1
   - **Olives** — PriceDelta: +£0.50, CaloriesDelta: +20, IsAvailable: yes, IsDefault: no, DisplayOrder: 2
   - **Jalapeños** — PriceDelta: +£0.75, CaloriesDelta: +15, IsAvailable: yes, IsDefault: no, DisplayOrder: 3
5. Click **"Save"**

**Expected Result**:
- A success message appears
- The "Toppings" modifier group is saved with all three modifiers
- PriceDeltas are stored correctly (Extra Cheese +£1.00, Olives +£0.50, Jalapeños +£0.75)
- MinSelections = 0 and MaxSelections = 3 are stored correctly

**Pass Criteria**:
- ✅ "Toppings" modifier group saved
- ✅ All three modifiers saved with correct PriceDeltas
- ✅ MinSelections and MaxSelections stored correctly
- ✅ Success message displayed

**Edge Cases**:
- Setting MaxSelections less than MinSelections → should show validation error
- Adding a modifier with a blank name → should enforce required field validation
- Setting MaxSelections = MinSelections = 1 → effectively a single-required choice; verify customer UX matches expected behaviour
- Negative PriceDelta (e.g. −£0.50) → should be allowed if business rules permit discounted modifiers

---

### TC-VA-097: Toggle Modifier Availability

**Given**: You are editing the "Margherita" dish, which has the "Toppings" modifier group created in TC-VA-096

**Steps**:
1. Open the "Margherita" dish in Menu Builder
2. Locate the "Toppings" modifier group and find the **Extra Cheese** modifier
3. Toggle **IsAvailable** to **false** (unavailable)
4. Click **"Save"**
5. On the customer front-end, navigate to the Margherita dish detail page

**Expected Result**:
- A success message appears
- Extra Cheese is saved as unavailable
- On the customer front-end, Extra Cheese no longer appears as a selectable topping for Margherita

**Pass Criteria**:
- ✅ IsAvailable = false saved correctly
- ✅ Success message displayed
- ✅ Extra Cheese is hidden on the customer front-end

**Edge Cases**:
- Toggling the only available modifier in a required group to unavailable → should warn that customers will be unable to satisfy the required selection
- Re-enabling an unavailable modifier → restores it on the customer front-end within the propagation window

**Cross-portal verification**: After completing this test, run **TC-XP-006** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-098: Required Modifier Group

**Given**: You are creating or editing the "Margherita" dish in Menu Builder

**Steps**:
1. Open the "Margherita" dish form in Menu Builder
2. Add a modifier group:
   - **Name**: "Choose Your Base"
   - **SelectionType**: Single
   - **MinSelections**: 1
   - **MaxSelections**: 1
   - **IsRequired**: yes
3. Add two modifiers:
   - **Thin Crust** — PriceDelta: £0.00, IsDefault: yes, IsAvailable: yes
   - **Deep Pan** — PriceDelta: +£1.50, IsDefault: no, IsAvailable: yes
4. Save the dish
5. Publish the menu (if not already published)
6. On the customer front-end, navigate to the Margherita dish detail page
7. Attempt to add the dish to the basket **without** making a base selection

**Expected Result**:
- The dish is saved successfully in Menu Builder
- On the customer front-end the "Choose Your Base" group is shown as required
- Attempting to add the dish without a selection shows a validation message: "Please make a selection for Choose Your Base"
- Selecting "Thin Crust" or "Deep Pan" allows the dish to be added to the basket normally

**Pass Criteria**:
- ✅ Required modifier group saved correctly
- ✅ Customer cannot add the dish to the basket without selecting a base
- ✅ Validation message is clear and specific
- ✅ Correct selection allows the dish to be added to the basket

**Edge Cases**:
- Setting IsRequired = yes with MinSelections = 0 → contradictory settings; should validate or auto-correct MinSelections to 1
- Deselecting in a Single/Required group → customer should not be able to deselect a required single-selection once made
- Dish added to basket with "Thin Crust" selected, then price edited in Admin → basket item should retain the original price snapshot

**Cross-portal verification**: After completing this test, run **TC-XP-005** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-039: Upload Dish Image

**Given**: You are creating or editing a dish

**Steps**:
1. In the dish form, click **"Upload Image"** or the image placeholder
2. Select an image file from your device (JPG, PNG, or WEBP recommended)
3. Observe the upload progress
4. Save the dish

**Expected Result**:
- The image uploads successfully
- A thumbnail preview of the image is shown in the dish form
- After saving, the image appears on the dish on the customer front-end (processing may take a few seconds)

**Pass Criteria**:
- ✅ Image uploads without errors
- ✅ Thumbnail preview is shown
- ✅ Image appears on the front-end after save

**Edge Cases**:
- Unsupported file type (e.g., PDF, GIF) → should show error
- File too large → should show error with size limit
- Very slow upload (large image file) → progress indicator should show
- Network failure during upload → should show error and allow retry

**Cross-portal verification**: After completing this test, run **TC-XP-008** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-040: Reorder Dishes Within a Category

**Given**: You are in the menu editor and a category has at least 2 dishes

**Steps**:
1. Drag a dish to a different position within the category (if drag-and-drop supported)
	OR
	Click "Move Up" / "Move Down" buttons to reorder
2. Save changes (if explicit save is required)

**Expected Result**:
- Dishes are reordered within the category
- The new order is reflected on the customer front-end when the menu is published
- A success message appears (if applicable)

**Pass Criteria**:
- ✅ Dishes are reordered
- ✅ Front-end reflects new order

**Edge Cases**:
- Moving a dish to the same position → should do nothing
- Concurrent editing by two users → system should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-007** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-041: Move Dish to Different Category

**Given**: You are in the menu editor and have at least 2 categories with dishes

**Steps**:
1. Select a dish and choose **"Move to Category"** (if available)
	OR
	Drag the dish to a different category (if drag-and-drop supported)
2. Confirm the move

**Expected Result**:
- A success message appears: "Dish moved successfully"
- The dish appears in the new category and is removed from the original category

**Pass Criteria**:
- ✅ Dish is moved
- ✅ Success message displayed

**Edge Cases**:
- Moving a dish to the same category → should do nothing
- Moving the only dish in a category → category remains but is empty

**Cross-portal verification**: After completing this test, run **TC-XP-007** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-042: Reorder Categories

**Given**: You are in the menu editor and have at least 2 categories

**Steps**:
1. Drag a category to a different position (if drag-and-drop supported)
	OR
	Click "Move Up" / "Move Down" buttons to reorder
2. Save changes (if explicit save is required)

**Expected Result**:
- Categories are reordered in the editor
- The new order is reflected on the customer front-end when the menu is published

**Pass Criteria**:
- ✅ Categories are reordered
- ✅ Front-end reflects new order

**Edge Cases**:
- Moving a category to the same position → should do nothing

**Cross-portal verification**: After completing this test, run **TC-XP-007** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-043: Publish Menu

**Given**: You are viewing a draft menu with at least one category and one dish

**Steps**:
1. Click **"Publish Menu"** in the menu editor or menu builder list
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Menu published successfully"
- The menu status changes to "Active" or "Published"
- The menu and its categories/dishes are visible on the customer front-end
- Customers can now add dishes from this menu to their basket

**Pass Criteria**:
- ✅ Menu status changes to "Published/Active"
- ✅ Success message displayed
- ✅ Menu is visible on the customer front-end

**Edge Cases**:
- Publishing an empty menu (no dishes) → should block or warn
- Publishing a menu with no price on a dish → should block or warn
- Publishing a second menu while one is already active → should handle gracefully (may deactivate the previous one)

**Cross-portal verification**: After completing this test, run **TC-XP-004** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-044: Manage Menu Availability/Status

**Given**: You have a published menu

**Steps**:
1. In the menu builder or editor, find the availability/status controls
2. Toggle the menu status (e.g., mark as "Temporarily Unavailable" or set active hours)
3. Save changes

**Expected Result**:
- Menu availability is updated
- Customers cannot order from a temporarily unavailable menu
- The front-end shows the menu as unavailable (or hides it)

**Pass Criteria**:
- ✅ Availability setting is saved
- ✅ Front-end respects availability setting

**Edge Cases**:
- Setting a menu as unavailable while orders are in progress → in-progress orders are not affected
- All menus set as unavailable → vendor shows as "not accepting orders" on front-end

**Cross-portal verification**: After completing this test, run **TC-XP-006** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-045: Update Menu (Edit Published Menu)

**Given**: You have an active/published menu

**Steps**:
1. Open the published menu in the menu editor
2. Make changes (add a dish, change a price, update a category name)
3. Save changes

**Expected Result**:
- Changes are saved
- A success message appears
- Updated menu is visible on the customer front-end immediately (or after a short delay)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Front-end is updated

**Edge Cases**:
- Saving changes while a customer is viewing the menu → customer sees the update on their next page load
- Removing a dish that's in an active basket → system should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-004** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Menu Scanning (AI Import)

The Menu Scan flow at `/Admin/MenuScan` lets vendors import a menu from uploaded photos or a PDF. The flow uses a wizard layout with three steps: **Upload pages** → **Review and delta** → **Import complete**.

### TC-VA-046: Upload Menu Photos for Scanning

**Given**: You are logged in as a Vendor Admin user and have a printed menu photo (or PDF) ready to upload

**Steps**:
1. Navigate to **Menu** → **Scan Menu** (or **Menu Scan**)
2. Select the target menu from the **MenuId** dropdown (or enter a new **MenuName**)
3. Verify the upload area is shown as a drag-and-drop dropzone
4. Upload one JPEG photo of a printed menu — confirm it is accepted
5. Upload one PNG photo — confirm it is accepted
6. Upload one PDF — confirm it is accepted
7. Remove one selected file from the list and re-add it
8. Verify the selected menu context is still correct
9. Click **Scan menu**
10. Attempt to upload a `.txt` file — confirm it is rejected
11. Attempt to upload a file that exceeds the maximum allowed size — confirm it is rejected with a size-limit error

**Expected Result**:
- JPEG, PNG, and PDF files are accepted
- Unsupported file types are rejected with a clear error message
- Oversized files are rejected with a clear error message referencing the size limit
- Dropzone accepts click-to-browse and drag-and-drop input
- Upload progress is shown during upload
- On success, the user is redirected to the Review step

**Pass Criteria**:
- ✅ JPEG accepted
- ✅ PNG accepted
- ✅ PDF accepted
- ✅ Unsupported file type rejected with error
- ✅ Oversized file rejected with size-limit error
- ✅ Dropzone interaction works for drag-and-drop and click upload
- ✅ Upload progress indicator visible
- ✅ Redirected to Review step on success

**Edge Cases**:
- Uploading a blank or unreadable image → AI may return zero dishes; proceed to TC-VA-047 to verify the empty-review state
- Very large PDF (10+ pages) → system should either process all pages or document and enforce a page limit
- Network failure mid-upload → error is shown; user can retry without losing the selected MenuId

---

### TC-VA-047: Review Scanned Menu

**Given**: You have completed a successful upload in TC-VA-046 and are on the Review step

**Steps**:
1. Observe the list of dishes extracted by the AI scanner
2. Verify that dish names and prices appear to match the uploaded menu
3. If scanning into an existing menu, inspect the delta sections for **Added**, **Removed**, and **Changed** items
4. Verify at least one known changed dish (name or price) appears in delta
5. Trigger **Re-scan** with updated source images or PDF
6. Wait for re-scan completion and confirm menu context remains unchanged
7. Click **Import menu**

**Expected Result**:
- Existing menu imports show a visual delta (added, removed, changed)
- Re-scan updates extraction output without losing selected menu context
- Import action runs directly from Review and redirects to Result

**Pass Criteria**:
- ✅ Extracted dishes are displayed
- ✅ Delta view is visible for existing menu imports
- ✅ Re-scan action succeeds and retains menu context
- ✅ Import action is available from Review

**Edge Cases**:
- AI extracts zero dishes → should show "No dishes detected" with an option to go back to Upload
- AI extracts 50+ dishes → list should scroll or paginate correctly
- Dish name containing special characters (for example, é or ñ) should be preserved

---

### TC-VA-048: Import Scanned Menu

**Given**: You are on the Review step after scanning in TC-VA-046 and validating output in TC-VA-047

**Steps**:
1. Review the extracted output and, if applicable, the delta summary
2. Click **Import menu**
3. Wait for the import to complete
4. Navigate to **Menu Builder** and open the target menu

**Expected Result**:
- A success message appears: "Menu import completed" (or similar)
- All confirmed dishes are present in the target menu in Menu Builder
- For existing menus, final state reflects the accepted delta
- The Result page is displayed (proceed to TC-VA-049)

**Pass Criteria**:
- ✅ Import completes without errors
- ✅ All confirmed dishes are present in Menu Builder
- ✅ Existing menu changes match delta expectations
- ✅ Success message displayed

**Edge Cases**:
- Confirming with zero dishes (all removed during Review) → should warn or block, or complete with a "No dishes imported" message
- Duplicate dish names in the same category → should be allowed or deduplicated with a warning (check business rules)
- Import fails mid-way → should show an error; the menu should not be left in a partial state

**Cross-portal verification**: After completing this test, run **TC-XP-005** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-049: Menu Scan Result View

**Given**: You have completed a successful import in TC-VA-048 and are on the Result page

**Steps**:
1. Observe the **Result** page
2. Verify the summary information: category count, dish count, and optional variant count
3. Click **Open menu builder** to navigate to Menu Builder
4. Return to the Result page (or use the browser back button)
5. Click **Scan again** to begin another scan in the same menu context

**Expected Result**:
- The Result page shows a clear summary including category and dish counts
- **Open menu builder** navigates correctly to the target menu in Menu Builder
- **Scan again** returns to the Upload step with the selected menu context preserved

**Pass Criteria**:
- ✅ Result page displays correct import summary
- ✅ **Open menu builder** navigates to Menu Builder correctly
- ✅ **Scan again** starts a new scan cycle correctly

**Edge Cases**:
- Navigating directly to the Result page URL without completing a scan → should redirect to the Upload step or show an appropriate error
- Using the browser back button from the Result page → should not re-trigger the import

---

### TC-VA-049A: Existing Menu Import Shows Delta Before Commit

**Given**: You start Menu Scan against an existing menu that already has categories and dishes

**Steps**:
1. Run upload and review steps for an existing menu
2. Inspect the delta sections for **Added**, **Removed**, and **Changed** items on Review
3. Verify at least one known changed dish (name or price) appears in delta
4. Import from the Review step
5. Open Menu Builder and compare final menu state

**Expected Result**:
- Review step displays a clear visual delta between existing menu and scanned result
- Added, removed, and changed items are classified correctly
- Final menu state matches accepted delta after commit

**Pass Criteria**:
- ✅ Delta view is visible before commit for existing menu imports
- ✅ Delta classification is accurate for sampled items
- ✅ Post-import menu state matches accepted changes

**Edge Cases**:
- No differences found → delta indicates no changes and import can be skipped or completed safely
- Large menu changes (100+ items) → delta remains readable and performant

---

### TC-VA-049B: Re-Scan Existing Menu Before Commit

**Given**: You are in the Menu Scan flow for an existing menu and have reached Review

**Steps**:
1. Trigger **Re-Scan** with updated source images/PDF
2. Wait for re-scan completion
3. Confirm the latest extracted items replace prior scan results
4. Verify menu context (selected existing menu) remains unchanged
5. Complete import

**Expected Result**:
- Re-scan updates extraction output in-place without losing selected menu context
- Latest extraction is what gets committed
- Import completes without creating duplicate menu records

**Pass Criteria**:
- ✅ Re-scan action succeeds
- ✅ Existing menu context is preserved
- ✅ Final import reflects latest re-scan output

**Edge Cases**:
- Re-scan returns fewer items than first scan → removed items appear in delta
- Re-scan fails → user sees error and can retry without losing current session

---

### TC-VA-049C: New Menu Import Saves as Unpublished

**Given**: You start Menu Scan with a new menu name (not an existing menu)

**Steps**:
1. Complete scan flow and confirm import
2. Open Menu Builder list
3. Locate the imported menu

**Expected Result**:
- Imported menu is created successfully
- Menu status is **Draft/Unpublished** until vendor explicitly publishes

**Pass Criteria**:
- ✅ New imported menu exists
- ✅ Menu is not auto-published

**Edge Cases**:
- Existing published menu remains active while new scanned menu stays unpublished

---

## Live Order Kanban

### TC-VA-050: View Live Order Kanban Dashboard

**Given**: You are logged in as a Vendor Admin user and have active orders

**Steps**:
1. Navigate to **Orders** → **Live Orders** or the **Kanban Dashboard**
2. Observe the kanban board
3. Verify the **Branch Context card** is visible with **Taking Orders** and **Scheduled Orders** toggles

**Expected Result**:
- A kanban board displays orders in columns:
  - **Pending** (new orders awaiting acceptance)
  - **Accepted** (accepted orders being prepared)
  - **Ready** (ready for driver pickup)
  - **Dispatched** (out for delivery)
  - **Completed** (delivered)
  - **Cancelled** (cancelled orders)
- Each order card shows: order ID, customer name, items ordered, total amount, order time
- The board auto-refreshes (or a refresh button is available)
- The **Branch Context card** is shown with both toggles accurately reflecting the current branch state

**Pass Criteria**:
- ✅ Kanban board loads correctly
- ✅ Orders are in the correct columns
- ✅ Order cards show key information
- ✅ Branch Context card visible with Taking Orders and Scheduled Orders toggles

**Edge Cases**:
- No active orders → "No orders" state should display for each column
- Many orders in one column → should scroll or paginate
- Auto-refresh frequency → new orders appear without manual refresh (within 60 seconds or as configured)

---

### TC-VA-051: Accept Order

**Given**: You are viewing the Live Order Kanban and a new order appears in the "Pending" column

**Steps**:
1. Click on the order card or open the order details
2. Review the order items, delivery address, and special instructions
3. Click **"Accept"**
4. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Order accepted"
- The order moves from the "Pending" column to the "Accepted" column
- The customer receives a notification: "Your order has been accepted"
- An estimated preparation time is set (if applicable)

**Pass Criteria**:
- ✅ Order status changes to "Accepted"
- ✅ Order moves to correct kanban column
- ✅ Success message displayed

**Edge Cases**:
- Accepting an order that was already accepted (by another user) → should show error
- Accepting an order after the acceptance timeout → should warn or show as expired
- Order with items that are out of stock → should allow acceptance or prompt to edit order

---

### TC-VA-052: Reject Order

**Given**: You are viewing a "Pending" order in the Live Order Kanban

**Steps**:
1. Click on the order card or open the order details
2. Click **"Reject"**
3. Select a rejection reason (e.g., "Too busy", "Item unavailable", "Closing soon")
4. Confirm the action

**Expected Result**:
- A success message appears: "Order rejected"
- The order moves to the "Cancelled" column
- The customer receives a notification: "Your order has been rejected" with the reason
- The customer's payment is refunded (if card payment)

**Pass Criteria**:
- ✅ Order status changes to "Cancelled/Rejected"
- ✅ Order moves to correct kanban column
- ✅ Success message displayed
- ✅ Rejection reason is recorded

**Edge Cases**:
- Rejecting without a reason → should enforce validation if reason is required
- Rejecting an order that was already accepted → should block or warn
- Rejecting a cash order → no refund needed, but customer should still be notified

---

### TC-VA-053: Mark Order as Ready for Pickup

**Given**: You are viewing an "Accepted" order in the Live Order Kanban

**Steps**:
1. Click on the order card or open the order details
2. Click **"Mark Ready"** or **"Ready for Pickup"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Order marked as ready"
- The order moves from "Accepted" to "Ready" column
- The assigned driver (if any) receives a notification: "Order is ready for pickup"
- The customer may receive a notification: "Your order is ready"

**Pass Criteria**:
- ✅ Order status changes to "Ready"
- ✅ Order moves to correct kanban column
- ✅ Success message displayed

**Edge Cases**:
- Marking as ready without a driver assigned → should proceed but prompt to assign a driver
- Marking as ready too quickly (e.g., 30 seconds after acceptance) → system should allow

---

### TC-VA-054: Assign Driver to Order

**Given**: You are viewing an order in the "Accepted" or "Ready" column

**Steps**:
1. Click on the order card or open the order details
2. Click **"Assign Driver"**
3. Select a driver from the available drivers list
4. Confirm the assignment

**Expected Result**:
- A success message appears: "Driver assigned"
- The driver's name appears on the order card
- The driver receives a notification of the assignment
- The order is visible in the Driver Portal for the assigned driver

**Pass Criteria**:
- ✅ Driver is assigned
- ✅ Success message displayed
- ✅ Driver name appears on order

**Edge Cases**:
- No drivers available → should show "No available drivers" message
- Driver already assigned to another active delivery → should warn but allow (depending on business rules)
- Assigning a driver to a cash order → should work the same as card

---

### TC-VA-055: Unassign Driver from Order

**Given**: You are viewing an order with a driver assigned

**Steps**:
1. Click on the order card or open the order details
2. Click **"Unassign Driver"** or change the driver assignment
3. Confirm the action

**Expected Result**:
- A success message appears: "Driver unassigned"
- The driver's name is removed from the order card
- The driver receives a notification that they are unassigned

**Pass Criteria**:
- ✅ Driver is unassigned
- ✅ Success message displayed
- ✅ Order no longer shows driver name

**Edge Cases**:
- Unassigning a driver who has already picked up the order → should warn or block
- Unassigning and immediately reassigning → both actions should complete cleanly

---

### TC-VA-056: Cancel Order (Vendor-Initiated)

**Given**: You are viewing an accepted order in the Live Order Kanban

**Steps**:
1. Click on the order card or open the order details
2. Click **"Cancel Order"**
3. Enter a cancellation reason
4. Confirm the action

**Expected Result**:
- A success message appears: "Order cancelled"
- The order moves to the "Cancelled" column
- The customer receives a notification with the cancellation reason
- The customer's payment is refunded (if card payment)

**Pass Criteria**:
- ✅ Order status changes to "Cancelled"
- ✅ Order moves to correct kanban column
- ✅ Cancellation reason is recorded
- ✅ Refund is initiated (for card payments)

**Edge Cases**:
- Cancelling an order that is already dispatched → should block or require special permission
- Cancelling a cash order → no refund needed, but customer should be notified
- Cancellation reason is required → should enforce validation

---

### TC-VA-057: Edit Order

**Given**: You are viewing a "Pending" or "Accepted" order

**Steps**:
1. Click on the order card or open the order details
2. Click **"Edit Order"** (if available)
3. Remove or modify items (e.g., remove a dish that is unavailable)
4. Save changes

**Expected Result**:
- A success message appears: "Order updated"
- The order is updated with the new items/pricing
- The customer is notified of the change (if applicable)
- The order total is recalculated

**Pass Criteria**:
- ✅ Order is updated
- ✅ Success message displayed
- ✅ Order total is recalculated correctly

**Edge Cases**:
- Editing order to have zero items → should block or prompt to cancel instead
- Editing a dispatched order → should block
- Price change after edit → system should handle refund/additional charge appropriately

---

### TC-VA-058: Kanban Auto-Refresh

**Given**: You are viewing the Live Order Kanban

**Steps**:
1. Keep the kanban board open
2. In a separate browser tab, place a new test order using the customer test account
3. Return to the kanban board and wait up to 60 seconds (or manually refresh)

**Expected Result**:
- The new order appears in the "Pending" column without requiring a manual page reload
- The order count updates

**Pass Criteria**:
- ✅ New order appears within expected refresh interval
- ✅ No manual refresh required (if auto-refresh is implemented)

**Edge Cases**:
- Kanban open for a long time (30+ minutes) → should still receive new orders
- Multiple new orders arrive simultaneously → all should appear correctly

---

## All Orders View

### TC-VA-060: View All Orders Page

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Orders** → **All Orders** or **Order History**
2. Observe the complete orders list

**Expected Result**:
- A table displays all orders (historical and current) with columns:
  - Order ID, customer name, order date, status, total amount, payment method, actions
- You can search by order ID or customer name
- You can filter by status, date range, and payment method
- You can click on an order to view full details

**Pass Criteria**:
- ✅ All orders are listed
- ✅ Search and filter work correctly
- ✅ Pagination works

**Edge Cases**:
- No orders → should display "No orders" message
- Very old orders → should still be retrievable
- Large order history → pagination is efficient and functional

---

### TC-VA-061: View Order Details from All Orders

**Given**: You are viewing the All Orders page

**Steps**:
1. Click on an order from the list
2. Review the order details

**Expected Result**:
- Full order details are displayed:
  - Order ID, status, customer details, delivery address
  - Order items, prices, subtotal, delivery fee, total
  - Payment method and status
  - Driver assignment (if applicable)
  - Order timeline/history

**Pass Criteria**:
- ✅ All order data is displayed correctly
- ✅ Timeline/history shows all status changes

**Edge Cases**:
- Viewing a cancelled order → cancellation reason should be visible
- Viewing a refunded order → refund details should be visible

---

## Driver Management

### TC-VA-070: View Drivers List

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Drivers** or **Driver Management**
2. Observe the drivers list

**Expected Result**:
- A table displays all drivers with columns: driver name, email, status (Active/Invited/Removed), branch assignment, actions
- You can invite a new driver
- You can view or edit driver details
- You can remove a driver

**Pass Criteria**:
- ✅ All drivers are listed
- ✅ Driver status is accurate
- ✅ Actions (Invite, View, Edit, Remove) are visible

**Edge Cases**:
- No drivers → should display "No drivers" message with a prompt to invite
- Driver with pending invitation → should show "Pending" status

---

### TC-VA-071: Invite Driver

**Given**: You are viewing the Drivers list

**Steps**:
1. Click **"Invite Driver"** or **"Add Driver"**
2. Enter the driver's name and email address
3. Select the branch(es) to assign them to (if applicable)
4. Click **"Send Invitation"**

**Expected Result**:
- A success message appears: "Driver invitation sent"
- The driver appears in the Drivers list with status "Invited"
- The driver receives an invitation email with a link to register on the Driver Portal

**Pass Criteria**:
- ✅ Invitation is sent
- ✅ Success message displayed
- ✅ Driver appears in the list with "Invited" status
- ✅ Email received (if verifiable)

**Edge Cases**:
- Duplicate email (driver already invited) → should show error or allow resend
- Invalid email format → should show validation error
- Missing required fields → should enforce validation

**Cross-portal verification**: After completing this test, run **TC-XP-030** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-072: Resend Driver Invitation

**Given**: A driver has been invited but has not yet registered

**Steps**:
1. In the Drivers list, find the driver with "Invited" status
2. Click **"Resend Invitation"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Invitation resent"
- The driver receives another invitation email
- The driver's status remains "Invited"

**Pass Criteria**:
- ✅ Invitation is resent
- ✅ Success message displayed

**Edge Cases**:
- Resending to a driver who has already registered → should show error or update status

---

### TC-VA-073: View Driver Details

**Given**: You are viewing the Drivers list

**Steps**:
1. Click on a driver to view their details

**Expected Result**:
- Driver details are displayed: name, email, phone number (if set), status, branch assignments
- Actions are visible: Edit, Remove, Resend Invitation (if pending)

**Pass Criteria**:
- ✅ All driver data is displayed
- ✅ Actions are appropriate for driver status

**Edge Cases**:
- Driver with no branch assignment → should show "No branch assigned"

---

### TC-VA-074: Edit Driver Details

**Given**: You are viewing a driver's details

**Steps**:
1. Click **"Edit"**
2. Modify driver details (e.g., name)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Driver updated successfully"
- Updated details are reflected in the Drivers list and details page

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Invalid phone format → should show validation error

---

### TC-VA-075: Assign Driver to Branch

**Given**: You are viewing a driver's details

**Steps**:
1. Click **"Assign to Branch"** or edit branch assignment
2. Select a branch from the available list
3. Save changes

**Expected Result**:
- A success message appears: "Driver assigned to branch"
- The driver is now visible as an available driver for that branch's orders

**Pass Criteria**:
- ✅ Driver is assigned
- ✅ Success message displayed
- ✅ Driver appears in branch's available driver list

**Edge Cases**:
- Assigning to a branch the driver is already assigned to → should show error or be idempotent
- Assigning to a branch that doesn't exist → should not be possible via UI

---

### TC-VA-076: Unassign Driver from Branch

**Given**: A driver is assigned to a branch

**Steps**:
1. In the driver's details, click **"Unassign from Branch"** or edit branch assignment
2. Remove the branch assignment
3. Save changes

**Expected Result**:
- A success message appears: "Driver unassigned from branch"
- The driver is no longer available for that branch's orders

**Pass Criteria**:
- ✅ Driver is unassigned
- ✅ Success message displayed

**Edge Cases**:
- Unassigning a driver who is currently delivering an order → should warn or block

---

### TC-VA-077: Remove Driver

**Given**: You are viewing a driver's details or the Drivers list

**Steps**:
1. Click **"Remove Driver"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Driver removed"
- The driver is removed from the Drivers list
- The driver can no longer log in to the Driver Portal for this vendor
- Any active deliveries by this driver are flagged (if applicable)

**Pass Criteria**:
- ✅ Driver is removed
- ✅ Success message displayed
- ✅ Driver's access is revoked

**Edge Cases**:
- Removing a driver with active deliveries → should warn or block
- Removing a driver who has not yet accepted their invitation → should be allowed

**Cross-portal verification**: After completing this test, run **TC-XP-030** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Team Management

Team Management allows vendor owners and managers to add and remove the staff who have access to Vendor Admin. Unlike drivers (who self-register via an invitation email), team members are created directly — a Microsoft Entra ID account is provisioned immediately and a temporary password is shown once.

### TC-VA-076: View Team Members List

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Team** in the sidebar
2. Observe the team members list

**Expected Result**:
- A table displays all current team members with columns: name, email, roles, date added, and actions
- An **Add User** button is visible
- A **Delete** button is available on each row

**Pass Criteria**:
- ✅ All team members are listed
- ✅ Role badges are displayed for each user
- ✅ Date added is visible
- ✅ Add User button is present

**Edge Cases**:
- No team members (brand-new vendor) → table should be empty but the Add User button should still be shown

---

### TC-VA-077: Add a New Team Member

**Given**: You are viewing the Team list

**Steps**:
1. Click **Add User**
2. Enter a valid email address (use a test address that does not already have a WantFood account)
3. Enter first name and last name
4. Leave display name blank
5. Select role **Staff** (default)
6. Click **Add User**

**Expected Result**:
- The form submits successfully
- You are redirected to the **Account Created** screen
- The Account Created screen shows:
  - The new user's name and email address
  - A **temporary password** in a read-only field with a Copy button
  - A warning that the password is shown **only once** and is never stored
  - An **Add Another User** button and a **Back to Team** button
- After clicking **Back to Team**, the new user appears in the team list

**Pass Criteria**:
- ✅ User is created successfully
- ✅ Temporary password is displayed on the Account Created screen
- ✅ Copy button is present and functional
- ✅ New user appears in the team list after returning
- ✅ Warning about copying the password is clearly visible

**Edge Cases**:
- Missing required field (email, first name, last name) → validation error shown, form not submitted
- Invalid email format → validation error shown
- Email already exists as a WantFood user → should return an error (duplicate account prevention)
- Navigating away from Account Created without copying → password cannot be retrieved; user must be deleted and recreated

---

### TC-VA-078: Copy Temporary Password

**Given**: You are on the Account Created screen after adding a team member

**Steps**:
1. Click the **Copy** button next to the temporary password field
2. Open a text editor or browser address bar
3. Paste (Ctrl+V / Cmd+V)

**Expected Result**:
- The temporary password is copied to the clipboard
- The text pasted matches the password shown in the read-only field exactly

**Pass Criteria**:
- ✅ Copy button is present and functional
- ✅ Clipboard content matches the displayed password exactly

**Edge Cases**:
- Test in both Chrome and Safari (clipboard API behaviour differs by browser and requires HTTPS)

---

### TC-VA-079: Add Team Member With Role Owner

**Given**: You are on the Add User form

**Steps**:
1. Fill in all required fields with valid test data
2. Select role **Owner** from the Role dropdown
3. Click **Add User**

**Expected Result**:
- User is created with Owner role
- Account Created screen is shown
- After returning to the Team list, the user's role badge shows **Owner**

**Pass Criteria**:
- ✅ Role dropdown contains: Owner, Manager, Staff
- ✅ Owner role is saved and shown in the list

---

### TC-VA-079a: Delete a Team Member

**Given**: You have at least one team member in the list (ideally a test user created for this purpose — do **not** delete your own account)

**Steps**:
1. Navigate to **Team**
2. Find the test user
3. Click **Delete**
4. Confirm the deletion in the confirmation prompt

**Expected Result**:
- A success message appears
- The deleted user is removed from the team list
- The deleted user's Microsoft Entra account is also removed (they can no longer sign in to Vendor Admin)

**Pass Criteria**:
- ✅ User is removed from the list
- ✅ Success message is shown
- ✅ Attempting to sign in as the deleted user fails (run TC-AC-054 to verify)

**Edge Cases**:
- Clicking Delete and then cancelling the confirmation → user should NOT be deleted
- Attempting to delete the only remaining user (yourself) → should be blocked or warned

**Cross-portal verification**: After completing TC-VA-079a, run **TC-AC-054** in [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) to confirm the deleted user can no longer sign in.

---

## Promotions and Offers

### TC-VA-080: View Vendor Offers List

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Offers** or **Promotions**
2. Observe the list of vendor-specific offers

**Expected Result**:
- A table displays all vendor offers with columns: offer name, discount type, discount value, promo code, status, start date, end date, actions
- You can create a new offer
- You can edit, pause, activate, clone, or delete existing offers

**Pass Criteria**:
- ✅ All offers are listed
- ✅ Offer details are correct
- ✅ Actions (Create, Edit, Pause, Activate, Clone, Delete) are visible

**Edge Cases**:
- No offers → should display "No offers" message
- Expired offers → should be visible and marked as "Expired"

---

### TC-VA-081: Create Vendor Offer

**Given**: You are viewing the Vendor Offers list

**Steps**:
1. Click **"Create Offer"** or **"New Offer"**
2. Enter offer details: offer name, discount type (% off or £ off), discount value, promo code, start date, end date, minimum order value (optional), usage limit (optional), terms
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Offer created successfully"
- The new offer appears in the Vendor Offers list
- Customers can use the promo code at checkout (if active and within date range)

**Pass Criteria**:
- ✅ Offer is created
- ✅ Success message displayed
- ✅ Offer appears in the list

**Edge Cases**:
- Duplicate promo code → should show error
- Invalid date range → should show validation error
- Discount value > 100% → should show validation error
- Discount value greater than average order value → should warn

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-082: View Offer Details

**Given**: You are viewing the Vendor Offers list

**Steps**:
1. Click on an offer
2. Review the offer details page

**Expected Result**:
- Full offer details are displayed: name, discount, promo code, dates, usage limit, current usage, status, terms
- Actions are visible: Edit, Pause/Activate, Clone, Delete

**Pass Criteria**:
- ✅ All offer data is displayed correctly
- ✅ Status is accurate

**Edge Cases**:
- Offer with zero usage → should show "0 redemptions"
- Offer with unlimited usage → should show "Unlimited"

---

### TC-VA-083: Edit Vendor Offer

**Given**: You are viewing a vendor offer's details

**Steps**:
1. Click **"Edit"**
2. Modify one or more fields (e.g., discount value, end date)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Offer updated successfully"
- Updated details are reflected in the Vendor Offers list

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Editing an active offer → changes apply immediately
- Changing promo code to a duplicate → should show error

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-084: Pause Vendor Offer

**Given**: You are viewing an active vendor offer

**Steps**:
1. Click **"Pause"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Offer paused"
- The offer status changes to "Paused"
- Customers can no longer use the promo code

**Pass Criteria**:
- ✅ Offer status changes to "Paused"
- ✅ Success message displayed
- ✅ Promo code is disabled

**Edge Cases**:
- Pausing an already-paused offer → should be blocked

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-085: Activate Vendor Offer

**Given**: You are viewing a paused vendor offer

**Steps**:
1. Click **"Activate"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Offer activated"
- The offer status changes to "Active"
- Customers can now use the promo code (within date range)

**Pass Criteria**:
- ✅ Offer status changes to "Active"
- ✅ Success message displayed

**Edge Cases**:
- Activating an expired offer → should warn or block

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-086: Clone Vendor Offer

**Given**: You are viewing a vendor offer's details

**Steps**:
1. Click **"Clone"** or **"Duplicate Offer"**
2. Review the cloned offer details (should be pre-filled with original values)
3. Modify the promo code (must be unique) and date range
4. Click **"Save"**

**Expected Result**:
- A success message appears: "Offer cloned successfully"
- The new offer appears in the Vendor Offers list
- The original offer is unchanged

**Pass Criteria**:
- ✅ Offer is cloned
- ✅ Success message displayed
- ✅ Cloned offer has a unique promo code

**Edge Cases**:
- Cloning with the same promo code → should show error (must be unique)
- Cloning an expired offer → should allow but prompt to update dates

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-087: Delete Vendor Offer

**Given**: You are viewing the Vendor Offers list

**Steps**:
1. Click **"Delete"** next to an offer
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Offer deleted"
- The offer is removed from the Vendor Offers list
- Customers can no longer use the promo code

**Pass Criteria**:
- ✅ Offer is deleted
- ✅ Success message displayed

**Edge Cases**:
- Deleting an offer with active usage → should warn
- Deleting an expired offer → should be allowed

**Cross-portal verification**: After completing this test, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-088: Verify Promo Code Validation on Checkout (Cross-Portal)

**Given**: A vendor offer with a promo code is active

**Steps**:
1. Log in to the Customer Front-end as `customer.uat@wantfood.com`
2. Add items from the vendor with the active offer to your basket
3. Proceed to checkout
4. Enter the promo code in the promo code field
5. Click **"Apply"**

**Expected Result**:
- The promo code is accepted and the discount is applied
- The order total is reduced by the discount amount
- The discount is visible on the order summary

**Pass Criteria**:
- ✅ Promo code is accepted
- ✅ Discount is applied correctly
- ✅ Order total reflects discount

**Edge Cases**:
- Incorrect promo code → should show error: "Invalid promo code"
- Expired promo code → should show error: "This offer has expired"
- Paused offer promo code → should show error: "This offer is not currently active"
- Order below minimum order value → should show error: "Minimum order value for this offer is £X"

**Cross-portal verification**: This test case already exercises the customer front-end. For deeper cross-portal verification including basket-level display, offer stacking with platform offers, and usage-limit exhaustion, run **TC-XP-016** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md).

---

## Reviews

### TC-VA-090: View Vendor Reviews List

**Given**: You are logged in as a Vendor Admin user and your vendor has received reviews

**Steps**:
1. Navigate to **Reviews**
2. Observe the list of reviews

**Expected Result**:
- A table displays all reviews with columns: customer name, rating (star), review text, date, status, actions
- You can filter reviews by rating or status

**Pass Criteria**:
- ✅ All reviews are listed
- ✅ Review details are correct

**Edge Cases**:
- No reviews → should display "No reviews yet"
- Very long review text → should truncate with a "Read more" option

---

### TC-VA-091: Flag Review for System Admin

**Given**: You are viewing a review you believe is inappropriate or violates guidelines

**Steps**:
1. Click on the review or the **"Flag"** button next to it
2. Select a flag reason (e.g., "Inappropriate content", "Spam", "Fake review")
3. Confirm the action

**Expected Result**:
- A success message appears: "Review flagged for moderation"
- The review is submitted to System Admin for review
- The review remains visible on the front-end until System Admin resolves it
- The review's status changes to "Flagged" in the Vendor Admin reviews list

**Pass Criteria**:
- ✅ Review is flagged
- ✅ Success message displayed
- ✅ Flagged review appears in System Admin's flagged reviews queue (verify with System Admin)

**Edge Cases**:
- Flagging a review that is already flagged → should show error or be idempotent
- Flagging without a reason → should enforce validation if reason is required

---

## Delivery Cost Configuration

### TC-VA-100: View Delivery Costs Page

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Settings** → **Delivery Costs** or **Delivery**
2. Observe the delivery cost configuration page

**Expected Result**:
- Delivery cost settings are displayed:
  - Delivery zones (if configured)
  - Delivery fee per zone or flat fee
  - Minimum order value for delivery
  - Free delivery threshold (if applicable)
- An "Edit" or "Save" button is visible

**Pass Criteria**:
- ✅ Delivery settings are displayed
- ✅ Edit functionality is accessible

**Edge Cases**:
- No delivery zones configured → should prompt to configure
- Single flat delivery fee → should display correctly

---

### TC-VA-101: Update Delivery Costs

**Given**: You are viewing the Delivery Costs page

**Steps**:
1. Click **"Edit"** or modify the delivery fee fields directly
2. Update delivery fee, minimum order value, or free delivery threshold
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Delivery costs updated successfully"
- Updated costs are applied immediately to new orders
- Customers at checkout see the new delivery fee

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ New delivery fee appears at checkout on front-end

**Edge Cases**:
- Setting delivery fee to £0 → should be allowed (free delivery)
- Setting minimum order value to £0 → should be allowed
- Setting free delivery threshold below minimum order → should validate or warn
- Invalid number format (e.g., letters in price field) → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-022** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Payment Settings

### TC-VA-110: View Payment Methods Page

**Given**: You are logged in as a Vendor Admin user

**Steps**:
1. Navigate to **Settings** → **Payment** or **Payment Methods**
2. Observe the payment settings page

**Expected Result**:
- Payment method settings are displayed:
  - Stripe Connect status (connected/not connected)
  - Cash payment option (enabled/disabled)
- Actions: Connect Stripe, Remove Stripe, Enable/Disable Cash

**Pass Criteria**:
- ✅ Payment settings are displayed
- ✅ Stripe connection status is accurate

**Edge Cases**:
- No payment methods configured → should prompt to configure (vendor cannot receive orders)

---

### TC-VA-111: Connect Stripe Account (Stripe Connect Setup)

**Given**: Your vendor does not yet have a Stripe account connected

**Steps**:
1. Click **"Connect Stripe"** or **"Set Up Stripe Payments"**
2. You are redirected to Stripe's Connect onboarding flow
3. Complete the Stripe Connect onboarding (use Stripe test mode credentials)
4. You are redirected back to Vendor Admin

**Expected Result**:
- You are returned to the payment settings page
- A success message appears: "Stripe connected successfully"
- The Stripe Connect status shows as "Connected"
- Customers can now pay by card for orders from your restaurant

**Pass Criteria**:
- ✅ Stripe account is connected
- ✅ Success message displayed
- ✅ Stripe status shows "Connected"

**Edge Cases**:
- Abandoning Stripe Connect flow → should return to Vendor Admin and show "Not connected"
- Stripe Connect flow error → should show error and provide retry option

**Cross-portal verification**: After completing this test, run **TC-XP-020** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-112: Enable Cash Payments

**Given**: You are viewing the Payment Methods page and cash payments are disabled

**Steps**:
1. Toggle or click **"Enable Cash Payments"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Cash payments enabled"
- Customers at checkout can now select "Pay with Cash" for your restaurant
- Cash orders follow the same flow but skip Stripe payment processing

**Pass Criteria**:
- ✅ Cash payments are enabled
- ✅ Success message displayed
- ✅ Cash payment option appears at checkout for this vendor

**Edge Cases**:
- Enabling cash for a vendor with no Stripe → should be allowed (cash only vendor)
- Enabling cash while Stripe is connected → both options should appear at checkout

**Cross-portal verification**: After completing this test, run **TC-XP-021** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-113: Disable Cash Payments

**Given**: Cash payments are currently enabled

**Steps**:
1. Toggle or click **"Disable Cash Payments"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Cash payments disabled"
- Cash payment option is no longer available at checkout for this vendor

**Pass Criteria**:
- ✅ Cash payments are disabled
- ✅ Success message displayed
- ✅ Cash payment option is removed from checkout

**Edge Cases**:
- Disabling cash while a cash order is in progress → in-progress orders should not be affected

**Cross-portal verification**: After completing this test, run **TC-XP-021** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

### TC-VA-114: Remove Stripe Configuration

**Given**: Your vendor has a Stripe account connected

**Steps**:
1. Navigate to **Payment Settings**
2. Click **"Remove Stripe"** or **"Disconnect Stripe"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Stripe configuration removed"
- The Stripe Connect status shows as "Not connected"
- Customers can no longer pay by card for orders from your restaurant (warning should be shown)

**Pass Criteria**:
- ✅ Stripe is disconnected
- ✅ Success message displayed
- ✅ Card payments are disabled

**Edge Cases**:
- Removing Stripe while cash payments are also disabled → vendor cannot accept any orders (should warn)
- Removing Stripe while there are pending card payments → should warn

**Cross-portal verification**: After completing this test, run **TC-XP-020** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change reaches the customer front-end / driver portal.

---

## Summary and Next Steps

You have now tested all major features of the **Vendor Admin portal**.

### What to do next:

1. **Log all bugs** found during testing on your Monday.com board under the "Vendor Admin" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the next portal**:
	- **[Driver Portal UAT](04-driver-portal-uat.md)**
	- **[Customer Front-end UAT](05-customer-frontend-uat.md)**
	- **[Order Saga UAT](06-order-saga-uat.md)** (critical — tests the full order flow across all portals)

---

**Great work! Keep testing! 🚀**
