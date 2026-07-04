# System Admin UAT — Test Scripts

## Purpose

This document contains test scripts for the **System Admin portal**. The System Admin portal is used by platform administrators to manage vendors, applications, orders, commissions, invoices, promotions, content, and platform settings.

**Target users**: Platform administrators, operations staff, content managers, commercial administrators

---

## Prerequisites

Before testing the System Admin portal, ensure you have:

- ✅ Access to the System Admin UAT environment URL
- ✅ System Admin test account credentials (`admin.uat@wantfood.com`)
- ✅ Browser and device ready (see [00-introduction.md](00-introduction.md#supported-browsers-and-devices))
- ✅ Monday.com board set up (see [01-monday-setup.md](01-monday-setup.md))
- ✅ Screenshot/recording tool ready

---

## Test Case Format

Each test case follows this format:

- **Given**: The starting state or prerequisites
- **Steps**: Numbered list of actions to perform
- **Expected Result**: What should happen if the feature works correctly
- **Pass Criteria**: Specific conditions that must be met for the test to pass
- **Edge Cases**: Additional scenarios to test for that feature

---

## Table of Contents

1. [Login and Authentication](#login-and-authentication)
2. [Dashboard](#dashboard)
3. [Vendor Applications](#vendor-applications)
4. [Vendor Management](#vendor-management)
5. [Vendor User Management](#vendor-user-management)
6. [Admin User Management](#admin-user-management)
7. [Orders](#orders)
8. [Commissions](#commissions)
9. [Invoices](#invoices)
10. [Platform Offers](#platform-offers)
11. [Review Moderation](#review-moderation)
12. [Content Management](#content-management)
    - [Hero Carousel Global Assets](#hero-carousel-global-assets) (TC-SA-128)
13. [Taxonomy Management](#taxonomy-management)
    - [Dish Keywords](#dish-keywords) (TC-SA-155 to TC-SA-157)
14. [Payment Settings](#payment-settings)
15. [Platform Settings](#platform-settings) (TC-SA-176 to TC-SA-177)
16. [Platform Tools](#platform-tools)

---

## Login and Authentication

### TC-SA-001: Successful Login

**Given**: You have valid System Admin credentials

**Steps**:
1. Navigate to the System Admin UAT environment URL
2. Click "Sign in" or enter the login page directly
3. Enter your System Admin credentials (`admin.uat@wantfood.com` and provided password)
4. Click "Sign in"

**Expected Result**:
- You are redirected to the System Admin dashboard
- Your username or email is displayed in the top navigation
- You see the main navigation menu with options for Vendors, Applications, Orders, Commissions, etc.

**Pass Criteria**:
- ✅ Login succeeds without errors
- ✅ Dashboard loads within 3 seconds
- ✅ No authentication error messages appear

**Edge Cases**:
- Invalid email format → should show validation error before submission
- Wrong password → should show "Invalid credentials" error
- Empty fields → should show "Required field" validation
- Account locked after multiple failed attempts → should show appropriate message

---

### TC-SA-002: Logout

**Given**: You are logged in as System Admin

**Steps**:
1. Click your profile/username in the top navigation
2. Click "Sign out" or "Logout"

**Expected Result**:
- You are redirected to the login page or public home page
- You cannot access any System Admin pages without logging in again
- Attempting to navigate to a System Admin URL redirects you to login

**Pass Criteria**:
- ✅ Session is terminated
- ✅ Direct URL access to admin pages requires re-authentication

---

## Dashboard

### TC-SA-010: Dashboard Loads and Displays Key Metrics

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to the System Admin dashboard (usually the home page after login)
2. Observe the displayed metrics and widgets

**Expected Result**:
- Dashboard displays key operational metrics (e.g., total vendors, active orders, pending applications)
- Widgets load without errors
- Recent activity or quick links are visible
- Navigation menu is accessible

**Pass Criteria**:
- ✅ All dashboard widgets load successfully
- ✅ Metrics display realistic numbers (not zeros if data exists)
- ✅ No error messages or "failed to load" indicators

**Edge Cases**:
- Empty state (no data) → should display "No data" messages gracefully
- Large numbers → should format correctly (e.g., 1,234,567 not 1234567)

---

## Vendor Applications

### TC-SA-020: View Pending Applications

**Given**: You are logged in as System Admin and pending vendor applications exist

**Steps**:
1. Navigate to **Applications** → **Applications List**
2. Observe the list of pending applications

**Expected Result**:
- A table or list displays all pending vendor applications
- Each application shows: vendor name, application date, contact email, status
- You can click on an application to view details

**Pass Criteria**:
- ✅ Pending applications are visible
- ✅ Application details load when clicked
- ✅ List is sorted (newest first by default)

**Edge Cases**:
- No pending applications → should display "No pending applications" message
- Pagination works if more than 10/20 applications exist

---

### TC-SA-021: View Application Details

**Given**: You are viewing a pending vendor application

**Steps**:
1. Click on a pending application from the list
2. Review the application details page

**Expected Result**:
- Full application details are displayed: business name, address, contact details, cuisine type, proposed delivery zones, supporting documents
- Action buttons are visible: "Approve" and "Reject"
- All submitted information is readable and correctly formatted

**Pass Criteria**:
- ✅ All application data is displayed
- ✅ No missing or garbled text
- ✅ "Approve" and "Reject" buttons are enabled

**Edge Cases**:
- Application with missing optional fields → should show "Not provided" or blank gracefully
- Long text fields → should wrap or truncate appropriately

---

### TC-SA-022: Approve Vendor Application

**Given**: You are viewing a pending vendor application

**Steps**:
1. Review the application details
2. Click the **"Approve"** button
3. Confirm the approval (if a confirmation dialog appears)

**Expected Result**:
- A success message appears: "Application approved successfully"
- The application status changes to "Approved"
- An onboarding invitation email is sent to the vendor (check email or email logs if accessible)
- The application moves out of the "Pending" list
- The vendor now appears in the Vendors list (inactive until onboarding completes)

**Pass Criteria**:
- ✅ Application status updates to "Approved"
- ✅ Success message displayed
- ✅ Vendor appears in Vendors list
- ✅ Email sent (if verifiable)

**Edge Cases**:
- Clicking "Approve" twice rapidly → should prevent duplicate approvals (idempotency)
- Network error during approval → should show error and not change status

**Cross-portal verification**: After completing this test, run **TC-XP-003** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-023: Reject Vendor Application

**Given**: You are viewing a pending vendor application

**Steps**:
1. Review the application details
2. Click the **"Reject"** button
3. Enter a rejection reason (if prompted)
4. Confirm the rejection

**Expected Result**:
- A success message appears: "Application rejected successfully"
- The application status changes to "Rejected"
- A rejection email is sent to the vendor (check email logs if accessible)
- The application moves out of the "Pending" list and may appear in a "Rejected" filter view

**Pass Criteria**:
- ✅ Application status updates to "Rejected"
- ✅ Success message displayed
- ✅ Rejection reason captured (if required)
- ✅ Email sent (if verifiable)

**Edge Cases**:
- Rejection reason is mandatory → should enforce validation if required
- Rejecting an already-rejected application → should show error or disable button

**Cross-portal verification**: After completing this test, run **TC-XP-003** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Vendor Management

### TC-SA-030: View Vendors List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Vendors** → **Vendors List**
2. Observe the list of vendors

**Expected Result**:
- A table displays all vendors with columns: vendor name, slug, city, region, status (Active/Inactive), actions
- You can search vendors by name
- You can filter by status (Active, Inactive, All)
- You can click on a vendor to view details

**Pass Criteria**:
- ✅ All vendors are listed
- ✅ Search and filter work correctly
- ✅ Actions (View, Edit, Activate, Deactivate) are visible

**Edge Cases**:
- Empty search result → should display "No vendors found"
- Pagination works if more than 10/20 vendors exist
- Sorting by column (name, city) works correctly

---

### TC-SA-031: View Vendor Details

**Given**: You are viewing the Vendors list

**Steps**:
1. Click on a vendor from the list
2. Review the vendor details page

**Expected Result**:
- Vendor details are displayed: business name, address, contact details, cuisine types, delivery zones, status, branches (if any)
- Action buttons are visible: "Edit", "Activate" (if inactive), "Deactivate" (if active)

**Pass Criteria**:
- ✅ All vendor data is displayed correctly
- ✅ Status is accurate (Active or Inactive)
- ✅ Branches are listed if they exist

**Edge Cases**:
- Vendor with no branches → should show "No branches" or similar
- Vendor with multiple branches → all branches are listed

---

### TC-SA-032: Edit Vendor Details

**Given**: You are viewing a vendor's details page

**Steps**:
1. Click the **"Edit"** button
2. Modify one or more fields (e.g., business name, contact email, phone number)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Vendor updated successfully"
- The updated information is reflected on the vendor details page
- No data is lost or corrupted

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Updated data is correct

**Edge Cases**:
- Invalid email format → should show validation error
- Required fields left blank → should enforce validation
- Concurrent edits by two admins → system should handle gracefully (last write wins or error)

**Cross-portal verification**: After completing this test, run **TC-XP-002** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-033: Activate Vendor

**Given**: You are viewing an inactive vendor's details page

**Steps**:
1. Click the **"Activate"** button
2. Confirm activation (if prompted)

**Expected Result**:
- A success message appears: "Vendor activated successfully"
- The vendor's status changes to "Active"
- The "Activate" button changes to "Deactivate"
- The vendor becomes visible on the customer front-end (if configured)

**Pass Criteria**:
- ✅ Vendor status changes to "Active"
- ✅ Success message displayed
- ✅ Vendor is searchable/visible on the front-end

**Edge Cases**:
- Activating an already-active vendor → button should be disabled or hidden
- Network error during activation → should show error and not change status

**Cross-portal verification**: After completing this test, run **TC-XP-001** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-034: Deactivate Vendor

**Given**: You are viewing an active vendor's details page

**Steps**:
1. Click the **"Deactivate"** button
2. Confirm deactivation (if prompted)

**Expected Result**:
- A success message appears: "Vendor deactivated successfully"
- The vendor's status changes to "Inactive"
- The "Deactivate" button changes to "Activate"
- The vendor is hidden from the customer front-end

**Pass Criteria**:
- ✅ Vendor status changes to "Inactive"
- ✅ Success message displayed
- ✅ Vendor is no longer visible on the front-end

**Edge Cases**:
- Deactivating a vendor with active orders → should either block deactivation or warn admin
- Deactivating an already-inactive vendor → button should be disabled or hidden

**Cross-portal verification**: After completing this test, run **TC-XP-001** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Vendor User Management

### TC-SA-040: View Vendor Users List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Vendors** → **Vendor Users**
2. Observe the list of vendor users

**Expected Result**:
- A table displays all vendor users with columns: name, email, associated vendor, status, actions
- You can search by name or email
- You can click on a user to view details

**Pass Criteria**:
- ✅ All vendor users are listed
- ✅ Search works correctly
- ✅ Actions (View, Transfer, Delete) are visible

**Edge Cases**:
- Empty search result → should display "No users found"
- Pagination works if more than 10/20 users exist

---

### TC-SA-041: Transfer Vendor User to Another Vendor

**Given**: You are viewing a vendor user's details

**Steps**:
1. Click the **"Transfer"** button or link
2. Select the target vendor from a dropdown or search field
3. Confirm the transfer

**Expected Result**:
- A success message appears: "User transferred successfully"
- The user is now associated with the new vendor
- The user can log in to Vendor Admin and see the new vendor context

**Pass Criteria**:
- ✅ User is transferred
- ✅ Success message displayed
- ✅ User's vendor association is updated

**Edge Cases**:
- Transferring to the same vendor → should show error or disable action
- Transferring a user who is the only admin for a vendor → should warn or block

**Cross-portal verification**: After completing this test, run **TC-AC-050** in [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-042: Delete Vendor User

**Given**: You are viewing a vendor user's details

**Steps**:
1. Click the **"Delete"** button
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "User deleted successfully"
- The user is removed from the Vendor Users list
- The user can no longer log in to Vendor Admin

**Pass Criteria**:
- ✅ User is deleted
- ✅ Success message displayed
- ✅ User's login is disabled

**Edge Cases**:
- Deleting the only admin user for a vendor → should warn or block
- Deleting a user who has active orders assigned → system should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-AC-051** in [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Admin User Management

### TC-SA-050: View Admin Users List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Admin** → **Users** or **Admin Users**
2. Observe the list of admin users

**Expected Result**:
- A table displays all system admin users with columns: name, email, role, status, actions
- You can search by name or email
- You can add a new admin user

**Pass Criteria**:
- ✅ All admin users are listed
- ✅ Search works correctly
- ✅ "Add User" button is visible

**Edge Cases**:
- Empty search result → should display "No users found"

---

### TC-SA-051: Add Admin User

**Given**: You are viewing the Admin Users list

**Steps**:
1. Click **"Add User"** or **"Create Admin User"**
2. Enter the new user's email address and role (if applicable)
3. Click **"Save"** or **"Create"**

**Expected Result**:
- A success message appears: "Admin user added successfully"
- The new user appears in the Admin Users list
- An invitation email is sent to the user (if applicable)

**Pass Criteria**:
- ✅ User is added
- ✅ Success message displayed
- ✅ User appears in the list

**Edge Cases**:
- Duplicate email → should show error: "User already exists"
- Invalid email format → should show validation error

**Cross-portal verification**: After completing this test, run **TC-AC-052** in [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-052: Delete Admin User

**Given**: You are viewing the Admin Users list

**Steps**:
1. Click the **"Delete"** button next to a user
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Admin user deleted successfully"
- The user is removed from the Admin Users list
- The user can no longer log in to System Admin

**Pass Criteria**:
- ✅ User is deleted
- ✅ Success message displayed
- ✅ User's login is disabled

**Edge Cases**:
- Deleting yourself → should be blocked or warned
- Deleting the only admin user → should be blocked

**Cross-portal verification**: After completing this test, run **TC-AC-053** in [10-permissions-and-account-uat.md](10-permissions-and-account-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Orders

### TC-SA-060: View Orders List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Orders** → **Orders List**
2. Observe the list of orders

**Expected Result**:
- A table displays all orders with columns: order ID, customer name, vendor name, order date, status, total amount, actions
- You can search orders by ID, customer name, or vendor name
- You can filter by status (Pending, Accepted, Completed, Cancelled, etc.)
- You can click on an order to view details

**Pass Criteria**:
- ✅ All orders are listed
- ✅ Search and filter work correctly
- ✅ Actions (View Details) are visible

**Edge Cases**:
- Empty search result → should display "No orders found"
- Pagination works if more than 10/20 orders exist
- Sorting by column (date, status, total) works correctly

---

### TC-SA-061: View Order Details

**Given**: You are viewing the Orders list

**Steps**:
1. Click on an order from the list
2. Review the order details page

**Expected Result**:
- Full order details are displayed:
  - Order ID, status, order date, customer details
  - Vendor and branch details
  - Order items (dishes, quantities, prices)
  - Subtotal, delivery fee, service fee, total amount
  - Payment method and status
  - Delivery address and delivery status (if applicable)
  - Driver assignment (if applicable)
  - Order timeline/history (if available)

**Pass Criteria**:
- ✅ All order data is displayed correctly
- ✅ Order status is accurate
- ✅ Payment and delivery information is visible

**Edge Cases**:
- Order with no driver assigned → should show "No driver assigned"
- Cancelled order → should show cancellation reason and timestamp
- Cash payment order → payment status should reflect "Cash"

---

### TC-SA-062: Order Support Investigation

**Given**: You are viewing an order with a reported issue (e.g., customer complaint, payment failure)

**Steps**:
1. Review the order timeline/history
2. Check payment status
3. Check delivery status
4. Note any errors or anomalies

**Expected Result**:
- You can trace the full order lifecycle (Pending → Accepted → Preparing → Dispatched → Delivered, or similar)
- Payment and delivery statuses are clearly visible
- Any errors or failures are logged and visible

**Pass Criteria**:
- ✅ Order timeline is complete and chronological
- ✅ Status transitions are logical
- ✅ Errors or issues are clearly indicated

**Edge Cases**:
- Order stuck in "Pending" → should be identifiable
- Payment authorized but not captured → should be visible
- Delivery marked "Failed" → reason should be displayed

---

## Commissions

### TC-SA-070: View Commission Tiers List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Commissions** → **Commission Tiers**
2. Observe the list of commission tiers

**Expected Result**:
- A table displays all commission tiers with columns: tier name, tier level, commission rate, order count threshold, actions
- You can add a new tier
- You can edit or delete existing tiers

**Pass Criteria**:
- ✅ All tiers are listed
- ✅ Tier details (rate, threshold) are correct
- ✅ Actions (Add, Edit, Delete) are visible

**Edge Cases**:
- No tiers defined → should display "No commission tiers" message
- Overlapping thresholds → system should validate or warn

---

### TC-SA-071: Create Commission Tier

**Given**: You are viewing the Commission Tiers list

**Steps**:
1. Click **"Add Tier"** or **"Create Tier"**
2. Enter tier details: tier name, tier level (e.g., Bronze, Silver, Gold), commission rate (%), order count threshold
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Commission tier created successfully"
- The new tier appears in the Commission Tiers list
- The tier is available for assignment to vendors

**Pass Criteria**:
- ✅ Tier is created
- ✅ Success message displayed
- ✅ Tier appears in the list

**Edge Cases**:
- Duplicate tier name → should show error or allow (depending on business rules)
- Invalid rate (e.g., negative or > 100%) → should show validation error
- Order threshold less than previous tier → should validate or warn

**Cross-portal verification**: After completing this test, run **TC-XP-017** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-072: Edit Commission Tier

**Given**: You are viewing a commission tier's details

**Steps**:
1. Click the **"Edit"** button
2. Modify one or more fields (e.g., commission rate, threshold)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Commission tier updated successfully"
- The updated tier details are reflected in the Commission Tiers list
- Vendors currently assigned to this tier will use the updated rate going forward

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Tier data is updated

**Edge Cases**:
- Updating a tier that vendors are currently using → should apply to future transactions only
- Invalid rate → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-017** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-073: Delete Commission Tier

**Given**: You are viewing the Commission Tiers list

**Steps**:
1. Click the **"Delete"** button next to a tier
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Commission tier deleted successfully"
- The tier is removed from the Commission Tiers list
- Vendors currently assigned to this tier are reassigned to a default tier (or flagged for reassignment)

**Pass Criteria**:
- ✅ Tier is deleted
- ✅ Success message displayed
- ✅ Vendors are handled gracefully (not left without a tier)

**Edge Cases**:
- Deleting a tier assigned to active vendors → should warn or prevent
- Deleting the only tier → should be blocked

**Cross-portal verification**: After completing this test, run **TC-XP-017** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-074: View Commission Configuration

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Commissions** → **Commission Configuration**
2. Observe the configuration page

**Expected Result**:
- Commission calculation settings are displayed (e.g., calculation frequency, default tier, tier upgrade rules)
- You can edit the configuration
- A "Save" button is visible

**Pass Criteria**:
- ✅ Configuration settings are displayed
- ✅ Edit controls are functional

**Edge Cases**:
- No configuration set → should show default values or prompt to configure

---

### TC-SA-075: Update Commission Configuration

**Given**: You are viewing the Commission Configuration page

**Steps**:
1. Modify one or more settings (e.g., change calculation time from 02:00 to 03:00)
2. Click **"Save"**

**Expected Result**:
- A success message appears: "Commission configuration updated successfully"
- The new settings are applied immediately or take effect at the next calculation run
- A confirmation message explains when changes will take effect

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Settings are applied correctly

**Edge Cases**:
- Invalid time format → should show validation error
- Changing calculation frequency mid-cycle → should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-018** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-076: View Commission Assignments

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Commissions** → **Commission Assignments**
2. Observe the list of vendors and their assigned tiers

**Expected Result**:
- A table displays vendors with columns: vendor name, current tier, commission rate, order count, last updated
- You can manually reassign a vendor to a different tier (if allowed)

**Pass Criteria**:
- ✅ All vendors are listed with their tier assignments
- ✅ Tier details are correct

**Edge Cases**:
- Vendor with no tier assigned → should show "Not assigned" or default tier
- Vendor recently upgraded by automated calculation → timestamp should be recent

---

### TC-SA-077: Trigger Manual Commission Tier Recalculation

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Commissions** → **Recalculate Tiers** (or similar)
2. Click **"Recalculate Now"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Commission tier recalculation triggered"
- The system recalculates vendor commission tiers based on order counts
- Vendors are upgraded or downgraded as appropriate
- A summary report is displayed (optional)

**Pass Criteria**:
- ✅ Recalculation completes successfully
- ✅ Vendor tiers are updated correctly
- ✅ Success message displayed

**Edge Cases**:
- Triggering recalculation while automated nightly job is running → should queue or block
- No orders exist → should complete with "No changes" message

**Cross-portal verification**: After completing this test, run **TC-XP-018** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Invoices

### TC-SA-080: View Uninvoiced Dashboard

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Invoices** → **Uninvoiced Dashboard**
2. Observe the uninvoiced order summary

**Expected Result**:
- A dashboard displays uninvoiced orders grouped by vendor
- Total uninvoiced amounts are shown per vendor
- You can drill down to see uninvoiced line items
- Actions to generate invoices are visible

**Pass Criteria**:
- ✅ Uninvoiced data is displayed correctly
- ✅ Vendor grouping is accurate
- ✅ Amounts are calculated correctly

**Edge Cases**:
- No uninvoiced orders → should display "No uninvoiced orders" message
- Vendor with zero commission (e.g., promo period) → should handle gracefully

---

### TC-SA-081: View Uninvoiced Line Items

**Given**: You are viewing the Uninvoiced Dashboard

**Steps**:
1. Click on a vendor to view uninvoiced line items
2. Observe the detailed list

**Expected Result**:
- A table displays all uninvoiced orders for that vendor with columns: order ID, order date, order total, commission rate, commission amount
- A total uninvoiced commission amount is shown at the bottom

**Pass Criteria**:
- ✅ All uninvoiced orders are listed
- ✅ Commission amounts are calculated correctly
- ✅ Total matches sum of line items

**Edge Cases**:
- Mixed commission rates (tier changes mid-month) → each order should reflect its correct rate
- Orders with refunds → commission should adjust accordingly

---

### TC-SA-082: Generate Single Vendor Invoice

**Given**: You are viewing uninvoiced line items for a vendor

**Steps**:
1. Click **"Generate Invoice"** or **"Create Invoice"**
2. Confirm the invoice details (date range, total amount)
3. Click **"Generate"**

**Expected Result**:
- A success message appears: "Invoice generated successfully"
- An invoice is created with a unique invoice number
- The invoice appears in the Invoices List
- The uninvoiced orders are now marked as "invoiced" and removed from the Uninvoiced Dashboard
- A PDF or downloadable invoice is available (optional)

**Pass Criteria**:
- ✅ Invoice is generated
- ✅ Success message displayed
- ✅ Uninvoiced orders are cleared
- ✅ Invoice appears in Invoices List

**Edge Cases**:
- Generating an invoice with zero amount → should block or warn
- Generating an invoice for a vendor with no uninvoiced orders → should block or warn

---

### TC-SA-083: Generate Monthly Invoices (Bulk)

**Given**: You are on the Uninvoiced Dashboard with multiple vendors having uninvoiced orders

**Steps**:
1. Click **"Generate Monthly Invoices"** or **"Bulk Generate"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Monthly invoices generated successfully"
- Invoices are created for all vendors with uninvoiced orders
- Each invoice has a unique invoice number
- All invoices appear in the Invoices List
- The Uninvoiced Dashboard is cleared (or shows only new uninvoiced orders after invoice date)

**Pass Criteria**:
- ✅ Invoices are generated for all eligible vendors
- ✅ Success message displayed
- ✅ Uninvoiced Dashboard is updated

**Edge Cases**:
- Some vendors have no uninvoiced orders → should skip them without error
- Invoices already generated for the current month → should prevent duplicate invoicing

---

### TC-SA-084: View Invoices List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Invoices** → **Invoices List**
2. Observe the list of invoices

**Expected Result**:
- A table displays all invoices with columns: invoice number, vendor name, invoice date, total amount, status (Paid/Unpaid/Cancelled), actions
- You can search by invoice number or vendor name
- You can filter by status

**Pass Criteria**:
- ✅ All invoices are listed
- ✅ Search and filter work correctly
- ✅ Actions (View, Mark Paid, Cancel) are visible

**Edge Cases**:
- No invoices exist → should display "No invoices" message
- Pagination works if more than 10/20 invoices exist

---

### TC-SA-085: View Invoice Details

**Given**: You are viewing the Invoices List

**Steps**:
1. Click on an invoice from the list
2. Review the invoice details page

**Expected Result**:
- Full invoice details are displayed:
  - Invoice number, vendor name, invoice date
  - Line items (orders included in this invoice)
  - Subtotal, commission rate, commission amount, total
  - Payment status (Paid/Unpaid/Cancelled)
  - Payment date (if paid)
- Actions: "Mark Paid", "Cancel Invoice", "Download PDF" (optional)

**Pass Criteria**:
- ✅ All invoice data is displayed correctly
- ✅ Line items match the uninvoiced orders that were invoiced
- ✅ Total amount is correct

**Edge Cases**:
- Invoice with many line items → should paginate or scroll
- Invoice with refunded orders → commission should reflect refunds

---

### TC-SA-086: Mark Invoice as Paid

**Given**: You are viewing an unpaid invoice's details

**Steps**:
1. Click **"Mark as Paid"**
2. Enter payment date (if prompted)
3. Confirm the action

**Expected Result**:
- A success message appears: "Invoice marked as paid"
- The invoice status changes to "Paid"
- The payment date is recorded
- The "Mark as Paid" button is disabled or hidden

**Pass Criteria**:
- ✅ Invoice status updates to "Paid"
- ✅ Success message displayed
- ✅ Payment date is recorded

**Edge Cases**:
- Marking an already-paid invoice → should be blocked or show error
- Invalid payment date (future date) → should show validation error

---

### TC-SA-087: Cancel Invoice

**Given**: You are viewing an unpaid invoice's details

**Steps**:
1. Click **"Cancel Invoice"**
2. Enter cancellation reason (if prompted)
3. Confirm the action

**Expected Result**:
- A success message appears: "Invoice cancelled successfully"
- The invoice status changes to "Cancelled"
- The invoiced orders are returned to the Uninvoiced Dashboard (or marked as "cancelled invoice" and excluded from future invoicing)
- The "Cancel Invoice" button is disabled or hidden

**Pass Criteria**:
- ✅ Invoice status updates to "Cancelled"
- ✅ Success message displayed
- ✅ Orders are handled appropriately

**Edge Cases**:
- Cancelling a paid invoice → should be blocked or require special permission
- Cancelling an already-cancelled invoice → should be blocked

---

## Platform Offers

### TC-SA-090: View Platform Offers List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Offers** → **Platform Offers**
2. Observe the list of platform offers

**Expected Result**:
- A table displays all platform offers with columns: offer name, discount type (%, £), discount value, promo code, status (Active/Paused), start date, end date, actions
- You can add a new offer
- You can edit or delete existing offers

**Pass Criteria**:
- ✅ All offers are listed
- ✅ Offer details are correct
- ✅ Actions (Add, Edit, Pause, Delete) are visible

**Edge Cases**:
- No offers exist → should display "No offers" message
- Expired offers → should be visible and marked as "Expired"

---

### TC-SA-091: Create Platform Offer

**Given**: You are viewing the Platform Offers list

**Steps**:
1. Click **"Add Offer"** or **"Create Offer"**
2. Enter offer details: offer name, discount type (percentage or fixed amount), discount value, promo code, start date, end date, usage limit (optional), terms
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Platform offer created successfully"
- The new offer appears in the Platform Offers list
- The offer is available for customers to use (if active and within date range)

**Pass Criteria**:
- ✅ Offer is created
- ✅ Success message displayed
- ✅ Offer appears in the list

**Edge Cases**:
- Duplicate promo code → should show error
- Invalid date range (end date before start date) → should show validation error
- Discount value > 100% → should show validation error (if percentage)

**Cross-portal verification**: After completing this test, run **TC-XP-015** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-092: View Platform Offer Details

**Given**: You are viewing the Platform Offers list

**Steps**:
1. Click on an offer from the list
2. Review the offer details page

**Expected Result**:
- Full offer details are displayed:
  - Offer name, discount type, discount value, promo code
  - Start date, end date, status
  - Usage limit, current usage count (if applicable)
  - Terms and conditions
- Actions: "Edit", "Pause" (if active), "Activate" (if paused), "Delete"

**Pass Criteria**:
- ✅ All offer data is displayed correctly
- ✅ Status is accurate
- ✅ Actions are appropriate for current status

**Edge Cases**:
- Offer with unlimited usage → usage limit should show "Unlimited"
- Offer with zero usage → usage count should show 0

---

### TC-SA-093: Edit Platform Offer

**Given**: You are viewing a platform offer's details

**Steps**:
1. Click **"Edit"**
2. Modify one or more fields (e.g., discount value, end date, terms)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Platform offer updated successfully"
- The updated offer details are reflected in the Platform Offers list
- Customers see the updated offer details immediately (if active)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Offer data is updated

**Edge Cases**:
- Editing an active offer → changes should apply immediately or warn admin
- Changing promo code to a duplicate → should show error

**Cross-portal verification**: After completing this test, run **TC-XP-015** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-094: Pause Platform Offer

**Given**: You are viewing an active platform offer's details

**Steps**:
1. Click **"Pause"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Platform offer paused successfully"
- The offer status changes to "Paused"
- The "Pause" button changes to "Activate"
- Customers can no longer use the promo code

**Pass Criteria**:
- ✅ Offer status updates to "Paused"
- ✅ Success message displayed
- ✅ Promo code is disabled on the front-end

**Edge Cases**:
- Pausing an already-paused offer → should be blocked or show error
- Pausing an expired offer → should be allowed or show warning

**Cross-portal verification**: After completing this test, run **TC-XP-015** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-095: Activate Platform Offer

**Given**: You are viewing a paused platform offer's details

**Steps**:
1. Click **"Activate"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Platform offer activated successfully"
- The offer status changes to "Active"
- The "Activate" button changes to "Pause"
- Customers can now use the promo code (if within date range)

**Pass Criteria**:
- ✅ Offer status updates to "Active"
- ✅ Success message displayed
- ✅ Promo code is enabled on the front-end

**Edge Cases**:
- Activating an already-active offer → should be blocked or show error
- Activating an expired offer → should warn or block

**Cross-portal verification**: After completing this test, run **TC-XP-015** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-096: Delete Platform Offer

**Given**: You are viewing the Platform Offers list

**Steps**:
1. Click **"Delete"** next to an offer
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Platform offer deleted successfully"
- The offer is removed from the Platform Offers list
- Customers can no longer use the promo code
- Historical usage data is preserved (optional)

**Pass Criteria**:
- ✅ Offer is deleted
- ✅ Success message displayed
- ✅ Promo code is disabled

**Edge Cases**:
- Deleting an offer with active usage → should warn or require confirmation
- Deleting an expired offer → should be allowed

**Cross-portal verification**: After completing this test, run **TC-XP-015** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-097: View Offer Analytics (Optional)

**Given**: You are viewing a platform offer's details

**Steps**:
1. Click **"View Analytics"** or **"Usage Report"**
2. Observe the analytics page

**Expected Result**:
- Analytics are displayed:
  - Total redemptions
  - Total discount amount given
  - Usage over time (chart or graph)
  - Top vendors where the offer was used (optional)

**Pass Criteria**:
- ✅ Analytics data is displayed
- ✅ Data is accurate and up-to-date

**Edge Cases**:
- Offer with zero usage → should show "No usage data"
- Offer with high usage → charts should render correctly

---

## Review Moderation

### TC-SA-100: View Flagged Reviews List

**Given**: You are logged in as System Admin and flagged reviews exist

**Steps**:
1. Navigate to **Reviews** → **Flagged Reviews**
2. Observe the list of flagged reviews

**Expected Result**:
- A table displays flagged reviews with columns: review ID, customer name, vendor/branch, rating, review text (truncated), flag reason, flag date, actions
- You can click on a review to view full details
- You can resolve or dismiss flags

**Pass Criteria**:
- ✅ All flagged reviews are listed
- ✅ Flag reasons are displayed
- ✅ Actions (View, Resolve, Dismiss) are visible

**Edge Cases**:
- No flagged reviews → should display "No flagged reviews" message
- Multiple flags on the same review → should aggregate or show count

**Cross-portal verification**: After completing this test, run **TC-XP-028** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-101: Resolve Flagged Review

**Given**: You are viewing a flagged review's details

**Steps**:
1. Review the review text and flag reason
2. Decide action: delete review, contact customer, or dismiss flag
3. Click **"Resolve"** or appropriate action button
4. Enter resolution notes (if prompted)
5. Confirm the action

**Expected Result**:
- A success message appears: "Flagged review resolved successfully"
- The review is removed from the Flagged Reviews list
- If the review was deleted, it is removed from the vendor's reviews
- If the flag was dismissed, the review remains published

**Pass Criteria**:
- ✅ Flagged review is resolved
- ✅ Success message displayed
- ✅ Review status is updated appropriately

**Edge Cases**:
- Deleting a review → vendor rating should recalculate
- Dismissing a flag → review should remain visible on front-end

**Cross-portal verification**: After completing this test, run **TC-XP-028** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-102: Trigger Manual Rating Recalculation

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Reviews** → **Recalculate Ratings** (or similar)
2. Click **"Recalculate Now"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Rating recalculation triggered"
- The system recalculates vendor, branch, and dish ratings
- Updated ratings are reflected on the front-end
- A summary report is displayed (optional)

**Pass Criteria**:
- ✅ Recalculation completes successfully
- ✅ Ratings are updated correctly
- ✅ Success message displayed

**Edge Cases**:
- Triggering recalculation while automated nightly job is running → should queue or block
- No reviews exist → should complete with "No changes" message

**Cross-portal verification**: After completing this test, run **TC-XP-028** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Content Management

### TC-SA-110: View Content Asset Library

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Content** → **Asset Library**
2. Observe the library of uploaded assets (images, videos, etc.)

**Expected Result**:
- A grid or list displays all content assets with thumbnails
- You can upload new assets
- You can search or filter assets by type, upload date, or name
- You can click on an asset to view details or edit metadata

**Pass Criteria**:
- ✅ All assets are displayed
- ✅ Upload button is visible
- ✅ Search and filter work correctly

**Edge Cases**:
- No assets uploaded → should display "No assets" message
- Large library → pagination or lazy loading works correctly

---

### TC-SA-111: Upload Content Asset

**Given**: You are viewing the Content Asset Library

**Steps**:
1. Click **"Upload Asset"**
2. Select an image or video file from your device
3. Enter metadata (optional): title, alt text, description
4. Click **"Upload"**

**Expected Result**:
- A success message appears: "Asset uploaded successfully"
- The new asset appears in the Content Asset Library
- The asset is available for use in hero slides, support local slides, or other content areas

**Pass Criteria**:
- ✅ Asset is uploaded
- ✅ Success message displayed
- ✅ Asset appears in the library

**Edge Cases**:
- Unsupported file type → should show error
- File size exceeds limit → should show error
- Duplicate filename → should handle gracefully (rename or overwrite with warning)

**Cross-portal verification**: After completing this test, run **TC-XP-029** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-112: Update Asset Metadata

**Given**: You are viewing an asset's details

**Steps**:
1. Click **"Edit"** or the asset to open details
2. Modify metadata fields (title, alt text, description)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Asset metadata updated successfully"
- The updated metadata is reflected in the Asset Library and wherever the asset is used

**Pass Criteria**:
- ✅ Metadata is updated
- ✅ Success message displayed

**Edge Cases**:
- Clearing required fields → should enforce validation if required

**Cross-portal verification**: After completing this test, run **TC-XP-029** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-113: Delete Content Asset

**Given**: You are viewing the Content Asset Library

**Steps**:
1. Click **"Delete"** next to an asset (or select asset and click delete)
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Asset deleted successfully"
- The asset is removed from the Content Asset Library
- If the asset is in use (e.g., in a hero slide), a warning is shown or the deletion is blocked

**Pass Criteria**:
- ✅ Asset is deleted
- ✅ Success message displayed

**Edge Cases**:
- Asset in use → should warn or block deletion
- Deleting an asset not in use → should succeed without issue

**Cross-portal verification**: After completing this test, run **TC-XP-029** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-120: View Hero Slides List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Content** → **Hero Slides**
2. Observe the list of hero slides (homepage carousel)

**Expected Result**:
- A list displays all hero slides with thumbnails, title, status (Published/Unpublished), display order, actions
- You can add a new slide
- You can reorder slides (drag-and-drop or up/down buttons)
- You can edit, publish, unpublish, or delete slides

**Pass Criteria**:
- ✅ All hero slides are listed
- ✅ Display order is correct
- ✅ Actions (Add, Edit, Reorder, Publish, Delete) are visible

**Edge Cases**:
- No slides exist → should display "No hero slides" message
- Slides out of order → reorder functionality works correctly

---

### TC-SA-121: Create Hero Slide

**Given**: You are viewing the Hero Slides list

**Steps**:
1. Click **"Add Slide"** or **"Create Slide"**
2. Enter slide details: title, description, link URL (optional), image (select from asset library or upload new)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Hero slide created successfully"
- The new slide appears in the Hero Slides list (unpublished by default)
- The slide is available for publishing

**Pass Criteria**:
- ✅ Slide is created
- ✅ Success message displayed
- ✅ Slide appears in the list

**Edge Cases**:
- No image selected → should enforce validation if required
- Invalid URL format → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-122: Edit Hero Slide

**Given**: You are viewing a hero slide's details

**Steps**:
1. Click **"Edit"**
2. Modify one or more fields (title, description, link URL, image)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Hero slide updated successfully"
- The updated slide details are reflected in the Hero Slides list and on the front-end (if published)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed
- ✅ Slide data is updated

**Edge Cases**:
- Editing a published slide → changes apply immediately on the front-end
- Changing image to an asset that doesn't exist → should show error

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-123: Publish Hero Slide

**Given**: You are viewing an unpublished hero slide's details

**Steps**:
1. Click **"Publish"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Hero slide published successfully"
- The slide status changes to "Published"
- The slide appears on the customer front-end homepage carousel

**Pass Criteria**:
- ✅ Slide status updates to "Published"
- ✅ Success message displayed
- ✅ Slide is visible on the front-end

**Edge Cases**:
- Publishing an already-published slide → should be blocked or show error
- Publishing a slide with missing required fields → should block or show error

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-124: Unpublish Hero Slide

**Given**: You are viewing a published hero slide's details

**Steps**:
1. Click **"Unpublish"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Hero slide unpublished successfully"
- The slide status changes to "Unpublished"
- The slide is removed from the customer front-end homepage carousel

**Pass Criteria**:
- ✅ Slide status updates to "Unpublished"
- ✅ Success message displayed
- ✅ Slide is hidden from the front-end

**Edge Cases**:
- Unpublishing the only published slide → carousel should handle gracefully (hide or show placeholder)

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-125: Reorder Hero Slides

**Given**: You are viewing the Hero Slides list

**Steps**:
1. Drag a slide to a new position (if drag-and-drop is supported)
	OR
	Click "Move Up" / "Move Down" buttons to reorder
2. Observe the new order
3. Save changes (if explicit save is required)

**Expected Result**:
- The hero slides are reordered in the list
- The new order is reflected on the customer front-end homepage carousel
- A success message appears (if applicable): "Hero slides reordered successfully"

**Pass Criteria**:
- ✅ Slides are reordered
- ✅ Front-end carousel reflects new order

**Edge Cases**:
- Moving a slide to the same position → should do nothing
- Concurrent reordering by two admins → system should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-126: Hero Slide Preview (Optional)

**Given**: You are viewing a hero slide's details

**Steps**:
1. Click **"Preview"**
2. Observe the preview

**Expected Result**:
- A preview of the hero slide is displayed as it would appear on the front-end
- The preview includes the image, title, description, and link (if applicable)

**Pass Criteria**:
- ✅ Preview renders correctly
- ✅ Preview matches front-end appearance

**Edge Cases**:
- Slide with no image → preview should show placeholder or error

---

### TC-SA-127: Delete Hero Slide

**Given**: You are viewing the Hero Slides list

**Steps**:
1. Click **"Delete"** next to a slide
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Hero slide deleted successfully"
- The slide is removed from the Hero Slides list
- The slide is removed from the customer front-end (if published)

**Pass Criteria**:
- ✅ Slide is deleted
- ✅ Success message displayed
- ✅ Front-end is updated

**Edge Cases**:
- Deleting a published slide → should remove it from the front-end immediately
- Deleting the only slide → carousel should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-009** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Hero Carousel Global Assets

### TC-SA-128: Edit Global Hero Carousel Assets

**Given**: You are logged in as System Admin and navigate to `/Admin/ContentHeroAssets/Edit`

**Steps**:
1. Navigate to **Content** → **Hero Carousel Assets** (or go directly to `/Admin/ContentHeroAssets/Edit`)
2. Observe the three editable fields:
   - **Custom CSS Content** — freeform CSS applied globally to the hero carousel
   - **Custom JS Content** — freeform JavaScript applied globally to the hero carousel
   - **Transition Type** — dropdown or text field controlling the slide transition animation (e.g., fade, slide, none)
3. Modify the **Transition Type** to a different value
4. Add a CSS comment to the **Custom CSS Content** field (e.g., `/* UAT test */`)
5. Click **"Save"**
6. Navigate to the customer front-end homepage and observe the carousel transition behaviour

**Expected Result**:
- A success message appears: "Hero carousel assets updated successfully"
- The updated transition type is reflected in the carousel on the customer front-end after a page refresh or cache rebuild
- The custom CSS is applied globally (inspect element to confirm the `<style>` tag includes the added comment)
- The global assets page is distinct from individual hero slide editing — changes here affect all slides simultaneously

**Pass Criteria**:
- ✅ Changes are saved without error
- ✅ Success message displayed
- ✅ Transition type change is observable on the customer front-end
- ✅ Custom CSS/JS changes do not break the carousel layout

**Edge Cases**:
- Invalid CSS syntax → should save (no server-side CSS validation expected) but may break carousel visually; tester should revert
- Invalid JavaScript syntax → should save but may cause a JS console error on the front-end; tester should revert
- Clearing all fields and saving → carousel should still render using default styles
- Transition Type set to an unrecognised value → carousel should fall back to a sensible default or display an error

**Cross-portal verification**: After completing this test, run **TC-XP-029** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-130: View Regions List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Content** → **Regions**
2. Observe the list of regions

**Expected Result**:
- A table displays all regions with columns: region name, slug, status, actions
- You can add a new region
- You can edit or delete existing regions

**Pass Criteria**:
- ✅ All regions are listed
- ✅ Actions (Add, Edit, Delete) are visible

**Edge Cases**:
- No regions exist → should display "No regions" message

---

### TC-SA-131: Create Region

**Given**: You are viewing the Regions list

**Steps**:
1. Click **"Add Region"** or **"Create Region"**
2. Enter region details: region name, slug (URL-friendly name)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Region created successfully"
- The new region appears in the Regions list
- The region is available for vendor assignment and front-end filtering

**Pass Criteria**:
- ✅ Region is created
- ✅ Success message displayed
- ✅ Region appears in the list

**Edge Cases**:
- Duplicate region name or slug → should show error
- Invalid slug format (spaces, special characters) → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-012** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-132: Edit Region

**Given**: You are viewing a region's details

**Steps**:
1. Click **"Edit"**
2. Modify region name or slug
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Region updated successfully"
- The updated region details are reflected in the Regions list and on the front-end

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Changing slug for a region with assigned vendors → should update URLs or warn

**Cross-portal verification**: After completing this test, run **TC-XP-012** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-133: Delete Region

**Given**: You are viewing the Regions list

**Steps**:
1. Click **"Delete"** next to a region
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Region deleted successfully"
- The region is removed from the Regions list
- Vendors assigned to this region are reassigned to a default region or flagged for reassignment

**Pass Criteria**:
- ✅ Region is deleted
- ✅ Success message displayed
- ✅ Vendors are handled gracefully

**Edge Cases**:
- Deleting a region with assigned vendors → should warn or prevent
- Deleting the only region → should be blocked

**Cross-portal verification**: After completing this test, run **TC-XP-012** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-140: View Support Local Slides List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Content** → **Support Local**
2. Observe the list of "Support Local" slides (promotional section on front-end)

**Expected Result**:
- A list displays all support local slides with thumbnails, title, status (Published/Unpublished), display order, actions
- You can add a new slide
- You can reorder slides
- You can edit, publish, unpublish, or delete slides

**Pass Criteria**:
- ✅ All support local slides are listed
- ✅ Display order is correct
- ✅ Actions (Add, Edit, Reorder, Publish, Delete) are visible

**Edge Cases**:
- No slides exist → should display "No support local slides" message

---

### TC-SA-141: Create Support Local Slide

**Given**: You are viewing the Support Local Slides list

**Steps**:
1. Click **"Add Slide"** or **"Create Slide"**
2. Enter slide details: title, description, link URL (optional), image (select from asset library or upload new)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Support local slide created successfully"
- The new slide appears in the Support Local Slides list (unpublished by default)

**Pass Criteria**:
- ✅ Slide is created
- ✅ Success message displayed
- ✅ Slide appears in the list

**Edge Cases**:
- No image selected → should enforce validation if required
- Invalid URL format → should show validation error

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-142: Edit Support Local Slide

**Given**: You are viewing a support local slide's details

**Steps**:
1. Click **"Edit"**
2. Modify one or more fields (title, description, link URL, image)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Support local slide updated successfully"
- The updated slide details are reflected in the Support Local Slides list and on the front-end (if published)

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Editing a published slide → changes apply immediately on the front-end

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-143: Publish Support Local Slide

**Given**: You are viewing an unpublished support local slide's details

**Steps**:
1. Click **"Publish"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Support local slide published successfully"
- The slide status changes to "Published"
- The slide appears on the customer front-end

**Pass Criteria**:
- ✅ Slide status updates to "Published"
- ✅ Success message displayed
- ✅ Slide is visible on the front-end

**Edge Cases**:
- Publishing an already-published slide → should be blocked or show error

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-144: Unpublish Support Local Slide

**Given**: You are viewing a published support local slide's details

**Steps**:
1. Click **"Unpublish"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Support local slide unpublished successfully"
- The slide status changes to "Unpublished"
- The slide is removed from the customer front-end

**Pass Criteria**:
- ✅ Slide status updates to "Unpublished"
- ✅ Success message displayed
- ✅ Slide is hidden from the front-end

**Edge Cases**:
- Unpublishing the only published slide → section should handle gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-145: Reorder Support Local Slides

**Given**: You are viewing the Support Local Slides list

**Steps**:
1. Drag a slide to a new position (if drag-and-drop is supported)
	OR
	Click "Move Up" / "Move Down" buttons to reorder
2. Observe the new order
3. Save changes (if explicit save is required)

**Expected Result**:
- The support local slides are reordered in the list
- The new order is reflected on the customer front-end
- A success message appears (if applicable): "Support local slides reordered successfully"

**Pass Criteria**:
- ✅ Slides are reordered
- ✅ Front-end reflects new order

**Edge Cases**:
- Moving a slide to the same position → should do nothing

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-146: Delete Support Local Slide

**Given**: You are viewing the Support Local Slides list

**Steps**:
1. Click **"Delete"** next to a slide
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Support local slide deleted successfully"
- The slide is removed from the Support Local Slides list
- The slide is removed from the customer front-end (if published)

**Pass Criteria**:
- ✅ Slide is deleted
- ✅ Success message displayed
- ✅ Front-end is updated

**Edge Cases**:
- Deleting a published slide → should remove it from the front-end immediately

**Cross-portal verification**: After completing this test, run **TC-XP-010** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Taxonomy Management

### TC-SA-150: View Cuisine Types List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Taxonomy** → **Cuisine Types**
2. Observe the list of cuisine types

**Expected Result**:
- A table displays all cuisine types with columns: cuisine name, slug, icon (optional), display order, actions
- You can add a new cuisine type
- You can reorder cuisine types for mobile or web

**Pass Criteria**:
- ✅ All cuisine types are listed
- ✅ Display order is correct
- ✅ Actions (Add, Edit, Reorder, Delete) are visible

**Edge Cases**:
- No cuisine types exist → should display "No cuisine types" message

---

### TC-SA-151: Reorder Cuisine Types for Mobile

**Given**: You are viewing the Cuisine Types list

**Steps**:
1. Navigate to **Cuisine Types** → **Mobile Ordering**
2. Drag cuisine types to reorder them (or use up/down buttons)
3. Save changes

**Expected Result**:
- Cuisine types are reordered in the list
- The new order is reflected on the mobile front-end
- A success message appears: "Cuisine type order updated successfully"

**Pass Criteria**:
- ✅ Cuisine types are reordered
- ✅ Mobile front-end reflects new order

**Edge Cases**:
- Moving a cuisine type to the same position → should do nothing

**Cross-portal verification**: After completing this test, run **TC-XP-011** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-152: Reorder Cuisine Types for Web

**Given**: You are viewing the Cuisine Types list

**Steps**:
1. Navigate to **Cuisine Types** → **Web Ordering**
2. Drag cuisine types to reorder them (or use up/down buttons)
3. Save changes

**Expected Result**:
- Cuisine types are reordered in the list
- The new order is reflected on the web front-end
- A success message appears: "Cuisine type order updated successfully"

**Pass Criteria**:
- ✅ Cuisine types are reordered
- ✅ Web front-end reflects new order

**Edge Cases**:
- Different ordering for mobile vs web → both should be respected

**Cross-portal verification**: After completing this test, run **TC-XP-011** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Dish Keywords

### TC-SA-155: View Dish Keywords List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Taxonomy** → **Dish Keywords** (or go directly to `/Admin/DishKeywords`)
2. Observe the list of existing dish keywords (synonyms used to match search queries to dishes)

**Expected Result**:
- A list or table displays all existing dish keywords
- Each keyword shows: the keyword text and an action to delete it
- A form or input field is visible to add a new keyword
- The maximum keyword length is 100 characters

**Pass Criteria**:
- ✅ All existing dish keywords are listed
- ✅ Add and delete controls are visible
- ✅ No error messages on page load

**Edge Cases**:
- No keywords exist → should display "No dish keywords" message or an empty list with the add form visible
- Very long list → pagination or scrolling works correctly

**Cross-portal verification**: After completing this test, run **TC-XP-014** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-156: Add Dish Keyword

**Given**: You are viewing the Dish Keywords page at `/Admin/DishKeywords`

**Steps**:
1. Locate the **Add Keyword** form (inline form or modal triggered by an "Add" button)
2. Enter a new keyword (e.g., `burger`)
3. Click **"Add"** or submit the form
4. Observe the page without a full reload

**Expected Result**:
- The keyword is added via AJAX — the page does **not** perform a full reload
- The new keyword appears in the list immediately after the AJAX response
- A success toast or inline confirmation is displayed

**Pass Criteria**:
- ✅ Keyword is added without a full page reload
- ✅ New keyword appears in the list
- ✅ Success feedback is displayed

**Edge Cases**:
- Keyword exceeds 100 characters → should show a client-side or server-side validation error
- Duplicate keyword → should show error: "Keyword already exists"
- Empty submission → should show validation error
- Special characters (e.g., `é`, `ñ`, `&`) → should be accepted and stored correctly

**Cross-portal verification**: After completing this test, run **TC-XP-014** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-157: Delete Dish Keyword

**Given**: You are viewing the Dish Keywords page at `/Admin/DishKeywords` with at least one keyword in the list

**Steps**:
1. Locate a keyword in the list and click its **"Delete"** button
2. Observe the confirmation dialog or prompt that appears
3. Confirm the deletion

**Expected Result**:
- A confirmation dialog appears before the deletion is performed
- After confirmation, the keyword is removed via AJAX — the page does **not** perform a full reload
- The keyword disappears from the list immediately after the AJAX response
- A success toast or inline confirmation is displayed

**Pass Criteria**:
- ✅ Confirmation dialog is shown before deletion
- ✅ Keyword is removed without a full page reload
- ✅ Keyword no longer appears in the list after deletion
- ✅ Success feedback is displayed

**Edge Cases**:
- Cancelling the confirmation dialog → keyword should remain in the list unchanged
- Attempting to delete the same keyword twice rapidly → should handle gracefully (idempotency)
- Deleting the last keyword → list should show empty state gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-014** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-160: View Bad Words List

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Taxonomy** → **Bad Words** or **Content Safety**
2. Observe the list of bad words (profanity filter)

**Expected Result**:
- A table displays all bad words with columns: word, severity (if applicable), actions
- You can add a new bad word
- You can edit or delete existing bad words
- You can bulk import bad words

**Pass Criteria**:
- ✅ All bad words are listed
- ✅ Actions (Add, Edit, Delete, Bulk Import) are visible

**Edge Cases**:
- No bad words exist → should display "No bad words" message
- Very long list → pagination works correctly

---

### TC-SA-161: Add Bad Word

**Given**: You are viewing the Bad Words list

**Steps**:
1. Click **"Add Bad Word"**
2. Enter the word to be filtered
3. Select severity (optional)
4. Click **"Save"**

**Expected Result**:
- A success message appears: "Bad word added successfully"
- The new word appears in the Bad Words list
- The word is now filtered from user-generated content (reviews, comments)

**Pass Criteria**:
- ✅ Bad word is added
- ✅ Success message displayed
- ✅ Word appears in the list

**Edge Cases**:
- Duplicate word → should show error or allow (depending on business rules)
- Empty field → should enforce validation

**Cross-portal verification**: After completing this test, run **TC-XP-013** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-162: Edit Bad Word

**Given**: You are viewing a bad word's details

**Steps**:
1. Click **"Edit"**
2. Modify the word or severity
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Bad word updated successfully"
- The updated word is reflected in the Bad Words list

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Editing to a duplicate word → should show error

**Cross-portal verification**: After completing this test, run **TC-XP-013** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-163: Delete Bad Word

**Given**: You are viewing the Bad Words list

**Steps**:
1. Click **"Delete"** next to a word
2. Confirm deletion (if prompted)

**Expected Result**:
- A success message appears: "Bad word deleted successfully"
- The word is removed from the Bad Words list
- The word is no longer filtered from user-generated content

**Pass Criteria**:
- ✅ Bad word is deleted
- ✅ Success message displayed

**Edge Cases**:
- Deleting a word → no historical impact on already-filtered content

**Cross-portal verification**: After completing this test, run **TC-XP-013** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-164: Bulk Import Bad Words

**Given**: You are viewing the Bad Words list

**Steps**:
1. Click **"Bulk Import"** or **"Import Bad Words"**
2. Upload a CSV or text file containing a list of bad words (one per line or comma-separated)
3. Click **"Import"**

**Expected Result**:
- A success message appears: "Bad words imported successfully" (or a summary: "X words added, Y duplicates skipped")
- All imported words appear in the Bad Words list
- Words are now filtered from user-generated content

**Pass Criteria**:
- ✅ Words are imported
- ✅ Success message displayed
- ✅ Words appear in the list

**Edge Cases**:
- File format incorrect → should show error and list expected format
- Duplicate words in import → should skip or warn
- Empty file → should show error or "0 words imported" message

**Cross-portal verification**: After completing this test, run **TC-XP-013** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-165: Bad Words Settings Page (Optional)

**Given**: You are viewing the Bad Words management area

**Steps**:
1. Navigate to **Settings** or **Bad Words Settings**
2. Configure settings (e.g., enable/disable filter, set action on detection: block or flag)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Bad words settings updated successfully"
- Settings are applied immediately

**Pass Criteria**:
- ✅ Settings are saved
- ✅ Success message displayed

**Edge Cases**:
- Disabling filter → user-generated content no longer filtered
- Setting action to "flag" → content is not blocked but flagged for review

---

## Payment Settings

### TC-SA-170: View Payment Settings

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Settings** → **Payment Settings**
2. Observe the payment configuration page

**Expected Result**:
- Payment provider settings are displayed (e.g., Stripe, WorldPay)
- You can configure or update payment provider credentials
- You can remove a payment provider configuration

**Pass Criteria**:
- ✅ Payment provider settings are displayed
- ✅ Edit/remove actions are visible

**Edge Cases**:
- No payment provider configured → should prompt to configure

---

### TC-SA-171: Save Stripe Platform Settings

**Given**: You are viewing the Payment Settings page

**Steps**:
1. Select **Stripe** as the payment provider
2. Enter Stripe API keys (publishable key, secret key)
3. Configure Stripe Connect settings (if applicable)
4. Click **"Save"**

**Expected Result**:
- A success message appears: "Stripe settings saved successfully"
- Stripe is now the active payment provider
- Vendors can connect their Stripe accounts via Stripe Connect

**Pass Criteria**:
- ✅ Stripe settings are saved
- ✅ Success message displayed
- ✅ Stripe is active

**Edge Cases**:
- Invalid API keys → should show error (e.g., "Invalid Stripe API key")
- Missing required fields → should enforce validation

**Cross-portal verification**: After completing this test, run **TC-XP-019** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-172: Save WorldPay Platform Settings

**Given**: You are viewing the Payment Settings page

**Steps**:
1. Select **WorldPay** as the payment provider
2. Enter WorldPay credentials (merchant code, installation ID, etc.)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "WorldPay settings saved successfully"
- WorldPay is now the active payment provider

**Pass Criteria**:
- ✅ WorldPay settings are saved
- ✅ Success message displayed
- ✅ WorldPay is active

**Edge Cases**:
- Invalid credentials → should show error
- Missing required fields → should enforce validation

**Cross-portal verification**: After completing this test, run **TC-XP-019** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-173: Remove Stripe Configuration

**Given**: Stripe is currently configured as the payment provider

**Steps**:
1. Navigate to **Payment Settings**
2. Click **"Remove Stripe Configuration"** or **"Disconnect Stripe"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Stripe configuration removed successfully"
- Stripe is no longer the active payment provider
- Vendors can no longer process payments via Stripe (warning should be shown)

**Pass Criteria**:
- ✅ Stripe configuration is removed
- ✅ Success message displayed

**Edge Cases**:
- Removing Stripe while vendors have active Stripe accounts → should warn or block
- No alternative payment provider configured → payments will fail (should warn)

**Cross-portal verification**: After completing this test, run **TC-XP-019** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-174: Remove WorldPay Configuration

**Given**: WorldPay is currently configured as the payment provider

**Steps**:
1. Navigate to **Payment Settings**
2. Click **"Remove WorldPay Configuration"** or **"Disconnect WorldPay"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "WorldPay configuration removed successfully"
- WorldPay is no longer the active payment provider

**Pass Criteria**:
- ✅ WorldPay configuration is removed
- ✅ Success message displayed

**Edge Cases**:
- Removing WorldPay while it's the only payment provider → should warn

**Cross-portal verification**: After completing this test, run **TC-XP-019** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Platform Settings

### TC-SA-176: View Platform Settings Page

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Settings** (or go directly to `/Admin/Settings`)
2. Observe the page layout and available tabs

**Expected Result**:
- The Settings page loads with a tabbed UI
- The default (first) tab displays the **Bad Words bulk editor** — a textarea containing all bad words, one per line
- Additional tabs may be present for other setting categories
- A **Save** button is visible within the active tab

**Pass Criteria**:
- ✅ Settings page loads without error
- ✅ Tabbed UI is rendered correctly
- ✅ Default tab shows the bad words bulk editor textarea
- ✅ The textarea is pre-populated with the current list of bad words

**Edge Cases**:
- No bad words configured → textarea should be empty (not an error state)
- Navigating directly to a non-default tab via URL hash → correct tab should be active

---

### TC-SA-177: Bulk Save Bad Words via Settings Tab

**Given**: You are viewing the Platform Settings page at `/Admin/Settings`, on the Bad Words tab

**Steps**:
1. Locate the bad words bulk textarea (newline-separated list of words)
2. Add two new words at the end of the list (one per line)
3. Remove one existing word by deleting its line
4. Click **"Save"**
5. Observe the feedback and the word count display (if present)
6. Navigate to **Taxonomy** → **Bad Words** (TC-SA-160) to verify the list reflects the changes

**Expected Result**:
- A success toast is displayed: "Bad words saved successfully" (or similar)
- The word count display updates to reflect the new total number of bad words
- The changes are persisted: the newly added words appear in the Bad Words list, and the removed word no longer appears

**Pass Criteria**:
- ✅ Save completes without error
- ✅ Success toast is displayed
- ✅ Word count display is updated
- ✅ Added words appear in the bad words list (verify via TC-SA-160 or TC-SA-164)
- ✅ Removed word is no longer in the list

**Edge Cases**:
- Submitting an empty textarea → should clear all bad words (or show a confirmation prompt)
- Leading/trailing whitespace on a line → should be trimmed before saving
- Duplicate words in the textarea → should be deduplicated on save (or show a warning)
- Very large list (thousands of words) → save should complete without timeout

**Cross-portal verification**: After completing this test, run **TC-XP-013** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals. See also **TC-SA-164** for individual bad-word import behaviour.

---

## Platform Tools

### TC-SA-180: Access Tools Dashboard

**Given**: You are logged in as System Admin

**Steps**:
1. Navigate to **Tools** or **Admin Tools**
2. Observe the tools dashboard

**Expected Result**:
- A dashboard displays available operational tools:
  - Reindex Vendors
  - Rebuild Caches
  - Import ONS Postcodes
  - View Postcode Stats
  - Run Integrity Check
- Each tool has a button or link to execute it

**Pass Criteria**:
- ✅ Tools dashboard loads
- ✅ All tools are listed and accessible

**Edge Cases**:
- No tools available → should display "No tools" message (unlikely)

---

### TC-SA-181: Reindex Vendors Tool

**Given**: You are viewing the Tools dashboard

**Steps**:
1. Click **"Reindex Vendors"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Vendor reindex triggered successfully"
- The system republishes vendor data for search indexing
- Vendors and dishes become searchable on the front-end (or updated search data is reflected)
- A summary report is displayed (optional)

**Pass Criteria**:
- ✅ Reindex completes successfully
- ✅ Success message displayed
- ✅ Search results on front-end are updated

**Edge Cases**:
- Triggering reindex while automated reindex is running → should queue or block
- No vendors exist → should complete with "0 vendors reindexed" message

**Cross-portal verification**: After completing this test, run **TC-XP-027** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-182: Rebuild Caches Tool

**Given**: You are viewing the Tools dashboard

**Steps**:
1. Click **"Rebuild Caches"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Cache rebuild triggered successfully"
- The system rebuilds vendor/menu JSON caches in the background
- Cached data on the front-end is refreshed
- A summary report is displayed (optional)

**Pass Criteria**:
- ✅ Cache rebuild completes successfully
- ✅ Success message displayed
- ✅ Front-end reflects updated cached data

**Edge Cases**:
- Triggering cache rebuild while automated rebuild is running → should queue or block
- No caches to rebuild → should complete with "0 caches rebuilt" message

**Cross-portal verification**: After completing this test, run **TC-XP-027** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-183: Import ONS Postcodes Tool

**Given**: You are viewing the Tools dashboard

**Steps**:
1. Click **"Import ONS Postcodes"**
2. Upload an ONS postcode data file (CSV or similar)
3. Click **"Import"**
4. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Postcode import triggered successfully" (or "X postcodes imported")
- Postcodes are imported into the database
- Delivery zone and address validation features use the updated postcode data
- A summary report is displayed (optional)

**Pass Criteria**:
- ✅ Postcodes are imported
- ✅ Success message displayed
- ✅ Postcode data is usable for address validation

**Edge Cases**:
- File format incorrect → should show error and list expected format
- Duplicate postcodes → should skip or update existing records
- Empty file → should show error or "0 postcodes imported" message
- File too large → should handle or warn about processing time

**Cross-portal verification**: After completing this test, run **TC-XP-027** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

### TC-SA-184: View Postcode Stats

**Given**: You are viewing the Tools dashboard

**Steps**:
1. Click **"View Postcode Stats"** or **"Postcode Stats"**
2. Observe the postcode statistics page

**Expected Result**:
- Postcode statistics are displayed:
  - Total postcodes in database
  - Date of last import
  - Coverage statistics (optional)
- A chart or summary is shown (optional)

**Pass Criteria**:
- ✅ Postcode stats are displayed
- ✅ Data is accurate and up-to-date

**Edge Cases**:
- No postcodes imported → should display "No postcode data" message

---

### TC-SA-185: Run Integrity Check Tool

**Given**: You are viewing the Tools dashboard

**Steps**:
1. Click **"Run Integrity Check"** or **"Data Integrity Check"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Integrity check completed successfully"
- A report is displayed showing:
  - Data consistency issues (if any)
  - Orphaned records (if any)
  - Recommendations for cleanup (if any)
- If no issues are found: "No issues detected"

**Pass Criteria**:
- ✅ Integrity check completes successfully
- ✅ Report is displayed
- ✅ Issues are clearly listed (if any)

**Edge Cases**:
- Issues detected → report should list specific records and issue types
- No issues detected → should display "All checks passed" message
- Large database → should handle processing time gracefully

**Cross-portal verification**: After completing this test, run **TC-XP-027** in [09-cross-portal-impact-uat.md](09-cross-portal-impact-uat.md) to confirm the change propagates correctly to the customer front-end / other portals.

---

## Summary and Next Steps

You have now tested all major features of the **System Admin portal**. 

### What to do next:

1. **Log all bugs** found during testing on your Monday.com board under the "System Admin" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the next portal**:
	- **[Vendor Admin UAT](03-vendor-admin-uat.md)**
	- **[Driver Portal UAT](04-driver-portal-uat.md)**
	- **[Customer Front-end UAT](05-customer-frontend-uat.md)**
	- **[Order Saga UAT](06-order-saga-uat.md)**
	- **[Automation and Jobs UAT](07-automation-jobs-uat.md)**

---

**Great work! Keep testing! 🚀**
