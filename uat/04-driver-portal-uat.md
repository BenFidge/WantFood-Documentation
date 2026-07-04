# Driver Portal UAT — Test Scripts

## Purpose

This document contains test scripts for the **Driver Portal**. The Driver Portal is used by delivery drivers to accept delivery assignments, manage their shifts, update delivery statuses, plan routes, manage their weekly availability, and maintain their profile.

**Target users**: Delivery drivers employed or contracted by vendors

---

## Prerequisites

Before testing the Driver Portal, ensure you have:

- ✅ Access to the Driver Portal UAT environment URL
- ✅ Driver test account credentials (`driver.uat@wantfood.com`)
- ✅ A test driver invitation sent by the Vendor Admin test account (see [03-vendor-admin-uat.md](03-vendor-admin-uat.md#driver-management))
- ✅ An active test branch with orders awaiting delivery (co-ordinate with Vendor Admin testing)
- ✅ Browser and device ready — **test on mobile where possible** (see [00-introduction.md](00-introduction.md#supported-browsers-and-devices))
- ✅ Location permissions allowed in your browser/device
- ✅ Monday.com board set up (see [01-monday-setup.md](01-monday-setup.md))
- ✅ Screenshot/recording tool ready

---

## Important Notes for Driver Portal Testing

### Location Permissions

The Driver Portal uses your device's location for live tracking and route planning. Before testing:

1. **Desktop browser**: When prompted, click "Allow" for location access
2. **Mobile**: Ensure location is enabled in device settings → Browser permissions
3. If location is **not available** (e.g., desktop without GPS), use the browser's developer tools to simulate a location, or test on a real mobile device

### Test Co-ordination

Several Driver Portal tests require a co-ordinated test order. You will need:

- A **Customer** to place an order via the Customer Front-end
- A **Vendor Admin** to accept the order and assign it to the test driver
- The **Driver** to then see and act on that delivery

Co-ordinate these steps with other members of your UAT team, or use all three test accounts yourself.

---

## Table of Contents

1. [Driver Onboarding](#driver-onboarding)
2. [Login and Dashboard](#login-and-dashboard)
3. [Shift Management](#shift-management)
4. [Active Deliveries](#active-deliveries)
5. [Route Planning](#route-planning)
6. [Availability Management](#availability-management)
7. [Driver Documents](#driver-documents)
8. [Driver Profile](#driver-profile)

---

## Driver Onboarding

### TC-DP-001: Receive and Open Driver Invitation

**Given**: A Vendor Admin has sent a driver invitation to `driver.uat@wantfood.com`

**Steps**:
1. Open the invitation email sent to `driver.uat@wantfood.com`
2. Review the email content
3. Click the invitation link in the email

**Expected Result**:
- The email is received promptly (within a few minutes of being sent)
- Email content includes:
  - Vendor/restaurant name
  - Clear call-to-action: "Click here to register"
  - Invitation link is clickable
- Clicking the link opens the Driver Portal Onboarding Landing Page

**Pass Criteria**:
- ✅ Invitation email received
- ✅ Email content is clear and professional
- ✅ Invitation link works and opens the correct page

**Edge Cases**:
- Email lands in spam/junk folder → testers should check spam if not received within 5 minutes
- Invitation link broken → should show an error page rather than a blank page

---

### TC-DP-002: Complete Driver Onboarding Registration

**Given**: You have clicked the invitation link and are on the Onboarding Landing Page

**Steps**:
1. Review the Onboarding Landing Page — it should introduce the Driver Portal and explain next steps
2. Click **"Get Started"** or **"Register"**
3. You are taken to the **Onboarding Registration Page**
4. Fill in all required fields:
	- First name, last name
	- Password (and confirm password)
	- Phone number
	- Any other required fields (e.g., vehicle type)
5. Click **"Register"** or **"Create Account"**

**Expected Result**:
- Registration form accepts your details
- You are taken to the **Onboarding Welcome Page**
- A confirmation message is shown: "Welcome to [Vendor name]! Your account is ready."

**Pass Criteria**:
- ✅ Registration form accepts valid data
- ✅ Account is created
- ✅ Welcome page displays

**Edge Cases**:
- Password too short or does not meet requirements → should show password policy error
- Password and confirm password do not match → should show validation error
- Missing required fields → should show validation errors
- Invalid phone format → should show validation error

---

### TC-DP-003: Onboarding Welcome Page

**Given**: You have just completed Driver Portal registration

**Steps**:
1. Observe the Onboarding Welcome Page

**Expected Result**:
- A clear welcome message is shown
- Instructions for next steps are displayed (e.g., "You're all set! You can now start accepting deliveries.")
- A link or button to access the Driver Dashboard is visible

**Pass Criteria**:
- ✅ Welcome page displays
- ✅ Clear next-steps message
- ✅ Navigation to dashboard works

**Edge Cases**:
- Navigating back to the registration page → should redirect to dashboard (not allow re-registration)

---

### TC-DP-004: Log in After Onboarding

**Given**: You have completed driver onboarding

**Steps**:
1. Navigate to the Driver Portal URL
2. Enter your registered email and password
3. Click **"Sign in"**

**Expected Result**:
- You are logged in and redirected to the Driver Dashboard
- Your name is shown in the navigation
- You can see the dashboard with delivery and shift controls

**Pass Criteria**:
- ✅ Login succeeds
- ✅ Dashboard loads
- ✅ Your identity (name) is shown

**Edge Cases**:
- Wrong password → should show "Invalid credentials" error
- Using invitation link again after registration → should redirect to login

**Cross-portal verification**: Run **[TC-XP-030](09-cross-portal-impact-uat.md#tc-xp-030-driver-invite--remove--driver-portal-access-and-order-assignment)** immediately after completing TC-DP-001 – TC-DP-004 to confirm the driver is visible and assignable in Vendor Admin. For invite link expiry and wrong-account sign-in edge cases, see **[TC-AC-020 and TC-AC-021](10-permissions-and-account-uat.md#tc-ac-020-invite-link-expiry--link-has-expired-or-already-been-used)** in `10-permissions-and-account-uat.md`.

---

## Login and Dashboard

### TC-DP-010: Driver Dashboard Home

**Given**: You are logged in as a driver

**Steps**:
1. Navigate to the Driver Dashboard (home page after login)
2. Observe the dashboard

**Expected Result**:
- Dashboard displays:
  - Current shift status (On Shift / Off Shift)
  - Start/End shift button(s)
  - Active deliveries section (or "No active deliveries")
  - Quick links to: Active Deliveries, Delivery History, Route Planner, Availability, Profile

**Pass Criteria**:
- ✅ Dashboard loads without errors
- ✅ Shift status is displayed
- ✅ Navigation is accessible

**Edge Cases**:
- Dashboard with no active deliveries → should show "No deliveries assigned" gracefully
- Dashboard with location permission denied → should show warning or prompt to enable

---

## Shift Management

### TC-DP-020: Start Shift

**Given**: You are logged in as a driver and are currently "Off Shift"

**Steps**:
1. On the Driver Dashboard, click **"Start Shift"**
2. If prompted, confirm your location is enabled
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Shift started"
- Shift status changes to "On Shift" or "Active"
- The "Start Shift" button changes to "End Shift"
- Your live location begins updating (if location permission is granted)
- You are now visible to the Vendor Admin as an available driver

**Pass Criteria**:
- ✅ Shift status changes to "On Shift"
- ✅ Success message displayed
- ✅ "End Shift" button is visible
- ✅ Location updates begin (if location permission granted)

**Edge Cases**:
- Starting a shift when already on shift → should show error or disable button
- Location permission denied → should allow shift start but warn that location tracking is unavailable
- Starting a shift outside of scheduled availability hours → should allow but may warn

---

### TC-DP-021: End Shift

**Given**: You are logged in as a driver and are currently "On Shift"

**Steps**:
1. On the Driver Dashboard, click **"End Shift"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Shift ended"
- Shift status changes to "Off Shift"
- The "End Shift" button changes to "Start Shift"
- Live location updates stop
- You are no longer visible as an available driver for new assignments

**Pass Criteria**:
- ✅ Shift status changes to "Off Shift"
- ✅ Success message displayed
- ✅ "Start Shift" button is visible

**Edge Cases**:
- Ending shift while an active delivery is in progress → should warn or block (delivery must be completed first)
- Ending a shift when not on shift → should show error or disable button
- Network failure during end shift → should show error and allow retry

**Mobile responsiveness**: Drivers typically end shifts on a phone. Run **[TC-RE-061](11-resilience-and-edge-cases-uat.md#tc-re-061-system-admin-on-mobile--phone-414--896)** (mobile viewport check) alongside this test case to confirm the End Shift button and confirmation dialogue are fully usable on a small screen.

---

### TC-DP-022: Shift Status Display

**Given**: You are logged in as a driver

**Steps**:
1. Note your current shift status (On Shift or Off Shift)
2. Start a shift (if off shift)
3. Open a second browser tab/window at the Driver Portal
4. Observe the shift status in the second tab

**Expected Result**:
- Shift status is consistent across tabs/windows
- Status updates in real-time or after a short refresh

**Pass Criteria**:
- ✅ Shift status is consistent
- ✅ Status updates correctly

**Edge Cases**:
- Long-running shift (8+ hours) → session should remain active or auto-refresh

---

### TC-DP-023: Live Location Update

**Given**: You are on shift with location permission granted

**Steps**:
1. Start a shift (if not already on shift)
2. Change your physical location (if testing on mobile) OR use browser developer tools to simulate a location change (desktop)
3. In System Admin or Vendor Admin (if a live map is available), check if your location updates
4. Alternatively, proceed with delivery actions and check that location-based features work

**Expected Result**:
- Your location is tracked and updates periodically while on shift
- Location data is used for route planning and delivery tracking features

**Pass Criteria**:
- ✅ Location updates are recorded (visible in admin or used by route planner)
- ✅ No errors related to location tracking

**Edge Cases**:
- Location updates stopped (signal lost) → system should handle gracefully (last known location retained)
- Location updates resumed after signal loss → should continue tracking normally

---

## Active Deliveries

### TC-DP-030: View Active Deliveries Page

**Given**: You are on shift and have been assigned a delivery by a Vendor Admin

**Steps**:
1. Navigate to **Active Deliveries** from the Driver Dashboard
2. Observe the active deliveries page

**Expected Result**:
- The assigned delivery is displayed with:
  - Order ID
  - Restaurant name and pickup address
  - Customer name and delivery address
  - Order items summary
  - Estimated pickup time
  - Delivery status (e.g., "Ready for Pickup")
- Action buttons are visible (e.g., "View Details", "Navigate", "Update Status")

**Pass Criteria**:
- ✅ Active delivery is visible
- ✅ Pickup and delivery address are shown correctly
- ✅ Action buttons are accessible

**Edge Cases**:
- No deliveries assigned → should show "No active deliveries" message
- Multiple deliveries assigned → all should be listed

---

### TC-DP-031: View Delivery Detail

**Given**: You are viewing the Active Deliveries page

**Steps**:
1. Click on an active delivery to view its details
2. Observe the delivery detail page

**Expected Result**:
- Full delivery details are displayed:
  - Order items (what was ordered)
  - Pickup address with map link or navigation button
  - Delivery address with map link or navigation button
  - Customer phone number (if provided)
  - Special delivery instructions
  - Order status and timeline

**Pass Criteria**:
- ✅ All delivery details are displayed
- ✅ Navigation buttons work (open maps app or browser maps)
- ✅ Customer contact details are visible (if applicable)

**Edge Cases**:
- Delivery with special instructions → instructions should be prominently displayed
- Delivery with no special instructions → should show "No special instructions"

---

### TC-DP-032: View Delivery Dashboard Map/Route

**Given**: You are viewing an active delivery

**Steps**:
1. Click **"Navigate"** or **"View Route"** (if available)
2. Observe the delivery dashboard map view

**Expected Result**:
- A map view is displayed showing:
  - Your current location (if location permission granted)
  - Pickup location (restaurant)
  - Delivery location (customer address)
  - Estimated route (if available)

**Pass Criteria**:
- ✅ Map loads correctly
- ✅ Pickup and delivery locations are plotted
- ✅ Your current location is shown (if location enabled)

**Edge Cases**:
- Location permission denied → map should show pickup/delivery pins without current location
- Invalid address → map should handle gracefully

---

### TC-DP-033: Mark Order Delivered

**Given**: You have collected the order from the restaurant and delivered it to the customer

**Steps**:
1. Navigate to the active delivery
2. Click **"Mark as Delivered"** or **"Delivery Complete"**
3. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Delivery marked as completed"
- The delivery status changes to "Delivered"
- The delivery moves out of "Active Deliveries"
- The customer receives a notification: "Your order has been delivered"
- The order status updates in Vendor Admin and System Admin
- You may be prompted to collect a tip (if the order allows digital tips)

**Pass Criteria**:
- ✅ Delivery status changes to "Delivered"
- ✅ Success message displayed
- ✅ Delivery removed from active list
- ✅ Customer notification sent (if verifiable)
- ✅ Order status visible in Vendor Admin (verify cross-portal)

**Edge Cases**:
- Marking as delivered before picking up → system should allow (no technical enforcement of pickup first) but confirm this is expected behavior
- Marking as delivered when the customer is not home → use "Mark as Failed" instead (see TC-DP-034)
- Network failure during status update → should show error and allow retry

**Cross-portal consequences**: When this step passes, verify **all** of the following surfaces reflect the delivery:
- **Vendor Admin** live order kanban: card moves to "Completed" column
- **Customer order tracking** page: status shows "Delivered"
- **Customer** receives a "Your order has been delivered" email
- **Commission line item** is generated in the order financials (check System Admin or Vendor Admin billing view)
- **Driver earnings** (if surfaced in the portal) update to include this delivery

See **[TC-XP-030](09-cross-portal-impact-uat.md#tc-xp-030-driver-invite--remove--driver-portal-access-and-order-assignment)** and the Order Saga UAT document (`06-order-saga-uat.md`) for the full order saga cross-portal verification steps.

---

### TC-DP-034: Mark Delivery as Failed / Report Issue

**Given**: You have attempted delivery but cannot complete it (e.g., customer not home, wrong address)

**Steps**:
1. Navigate to the active delivery
2. Click **"Mark as Failed"** or **"Report Issue"**
3. Select a reason (e.g., "Customer not home", "Wrong address", "Access denied")
4. Add any additional notes (optional)
5. Confirm the action

**Expected Result**:
- A success message appears: "Delivery issue reported"
- The delivery status changes to "Failed" or "Issue Reported"
- The delivery moves out of "Active Deliveries" or is flagged
- Vendor Admin and System Admin are notified of the failed delivery
- The customer may receive a notification

**Pass Criteria**:
- ✅ Failure/issue is recorded
- ✅ Success message displayed
- ✅ Delivery status reflects failure
- ✅ Visible in Vendor Admin (verify cross-portal)

**Edge Cases**:
- Failing a delivery without selecting a reason → should enforce validation if reason is required
- Failing a delivery multiple times → should handle gracefully
- Cash payment delivery failed → cash has not exchanged hands, should be noted

---

### TC-DP-035: Revert Delivery Status

**Given**: You have accidentally updated a delivery status (e.g., marked as delivered too early)

**Steps**:
1. Navigate to the delivery (may be in delivery history or a "revert" option in the active delivery view)
2. Click **"Revert Status"** or **"Undo"** (if available)
3. Confirm the action

**Expected Result**:
- A success message appears: "Delivery status reverted"
- The delivery status is reverted to the previous state
- The delivery reappears in "Active Deliveries" if it was marked as delivered

**Pass Criteria**:
- ✅ Delivery status is reverted
- ✅ Success message displayed

**Edge Cases**:
- Reverting a status more than 30 minutes after the initial update → system may block or warn
- Reverting after the customer has left a review → system should handle gracefully

---

### TC-DP-036: Decline Delivery Assignment

**Given**: You have been assigned a delivery but are unable to accept it (e.g., vehicle breakdown)

**Steps**:
1. Navigate to the assigned delivery notification or active deliveries page
2. Click **"Decline"** or **"Cannot Accept"** (if available)
3. Enter a reason (if prompted)
4. Confirm the action

**Expected Result**:
- A success message appears: "Delivery declined"
- The delivery is removed from your active deliveries
- Vendor Admin is notified and can reassign to another driver

**Pass Criteria**:
- ✅ Delivery is declined
- ✅ Success message displayed
- ✅ Delivery is removed from your active list

**Edge Cases**:
- Declining after already starting the delivery → should warn or block
- Declining when you are the only available driver → Vendor Admin should still be notified

**Cross-portal verification**: After declining, switch to Vendor Admin and confirm the order has returned to the "Unassigned" column (or equivalent) so a new driver can be assigned. See **[TC-XP-030](09-cross-portal-impact-uat.md#tc-xp-030-driver-invite--remove--driver-portal-access-and-order-assignment)** for the full driver-action cross-portal checklist.

---

### TC-DP-037: View Delivery History

**Given**: You are logged in as a driver

**Steps**:
1. Navigate to **Delivery History**
2. Observe the list of completed/historical deliveries

**Expected Result**:
- A table displays all completed deliveries with columns:
  - Order ID, date, vendor/restaurant, delivery address, status (Delivered/Failed), order total
- You can click on a delivery to view its details

**Pass Criteria**:
- ✅ Completed deliveries are listed
- ✅ Delivery status is accurate

**Edge Cases**:
- No delivery history → should display "No delivery history" message
- Very old deliveries → should still be retrievable

---

### TC-DP-038: Dashboard Partial Refresh (Live Delivery Screens)

**Given**: You are on the Delivery Dashboard map/route view during an active delivery

**Steps**:
1. Keep the delivery dashboard open for 2–3 minutes
2. Ask a Vendor Admin to update the order status (e.g., mark order as ready)
3. Observe if the dashboard updates without a full page reload

**Expected Result**:
- The dashboard updates to reflect the new order status without a full page reload
- Your route or delivery assignment information refreshes as needed

**Pass Criteria**:
- ✅ Dashboard refreshes without manual reload
- ✅ Status updates are reflected

**Edge Cases**:
- Delivery dashboard open for a long time → should continue refreshing
- Poor network connection → should retry or show stale data warning

---

## Route Planning

### TC-DP-040: Optimise Route (From Dashboard)

**Given**: You have at least one active delivery with a pickup and delivery address

**Steps**:
1. Navigate to the Delivery Dashboard or Route Planner entry point
2. Click **"Optimise Route"** or **"Plan Route"**
3. Wait for the route optimisation to complete

**Expected Result**:
- An optimised route is displayed showing:
  - Stops in the most efficient order (pickup first, then delivery)
  - Estimated distances and travel times
  - Map with route plotted (if map is displayed)
- The route can be used for navigation

**Pass Criteria**:
- ✅ Route optimisation completes successfully
- ✅ Stops are in a logical order (restaurant before customer)
- ✅ Map/route is displayed

**Edge Cases**:
- Single delivery (no optimisation needed) → should show a direct route
- Multiple deliveries → should order stops optimally
- Address not found → should show error for that stop

---

### TC-DP-041: Clear Optimised Route

**Given**: You have an optimised route displayed

**Steps**:
1. Click **"Clear Route"** or **"Reset Route"**
2. Confirm the action (if prompted)

**Expected Result**:
- A success message appears: "Route cleared"
- The optimised route is removed
- You can re-optimise if needed

**Pass Criteria**:
- ✅ Route is cleared
- ✅ Success message displayed

**Edge Cases**:
- Clearing route while actively navigating → should warn

---

## Availability Management

### TC-DP-050: View Weekly Availability Schedule

**Given**: You are logged in as a driver

**Steps**:
1. Navigate to **Availability** or **My Availability**
2. Observe the weekly availability schedule

**Expected Result**:
- A weekly schedule is displayed showing 7 days (Monday to Sunday)
- For each day, availability time slots are shown (or "Not available")
- You can edit your availability for each day

**Pass Criteria**:
- ✅ Weekly schedule loads correctly
- ✅ Current availability is displayed accurately

**Edge Cases**:
- No availability set → should show all days as "Not available" or "Unset"

---

### TC-DP-051: Update Weekly Availability

**Given**: You are viewing the weekly availability schedule

**Steps**:
1. Click **"Edit"** or click on a day to set availability
2. For one or more days, set your available time slots (e.g., Monday: 11:00 – 22:00)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Availability updated successfully"
- The updated availability is reflected in the schedule
- Vendor Admin can see your availability when scheduling

**Pass Criteria**:
- ✅ Availability is saved
- ✅ Success message displayed
- ✅ Schedule reflects new availability

**Edge Cases**:
- Setting availability for all 7 days → should save correctly
- Setting overlapping time slots (e.g., 10:00–14:00 and 12:00–18:00 on the same day) → should validate or merge
- Setting end time before start time (e.g., 22:00–10:00) → should show validation error
- Removing availability for a day → should save "Not available" for that day

**Mobile responsiveness**: Drivers are very likely to manage their availability on a phone. Run **[TC-RE-061](11-resilience-and-edge-cases-uat.md#tc-re-061-system-admin-on-mobile--phone-414--896)** (mobile viewport check) alongside this test case to confirm the availability grid and time-slot pickers are fully usable on a small screen.

---

### TC-DP-052: Apply Availability Preset

**Given**: You are viewing the weekly availability schedule and presets are available

**Steps**:
1. Look for a **"Apply Preset"** or **"Load Preset"** option
2. Select a preset (e.g., "Weekdays Only", "Weekends Only", "Full Week")
3. Click **"Apply"**

**Expected Result**:
- A success message appears: "Preset applied"
- The weekly schedule updates to reflect the preset
- You can further edit the schedule after applying the preset

**Pass Criteria**:
- ✅ Preset is applied
- ✅ Success message displayed
- ✅ Schedule reflects preset values

**Edge Cases**:
- Applying a preset over existing availability → should overwrite existing availability (with warning if applicable)
- No presets available → preset option should be hidden or disabled

---

## Driver Documents

> ⚠️ **FEATURE TO BE CONFIRMED** — The test cases in this section are placeholder tests. A code search of the Driver Portal (controllers, views, and models) found no document-upload UI, file-input fields, or insurance/licence-document endpoints in the current codebase. The data model captures a **licence plate number** as a free-text string but does not store scanned documents.
>
> **Action required (Project Manager)**: Please confirm whether driver document upload (driving licence scan, vehicle insurance certificate, etc.) is in scope for go-live. If yes, assign a developer to build the feature and re-run TC-DP-070 and TC-DP-071 once the UI exists. If no, mark both test cases as "Out of Scope – Not Required for Go-Live" on the Monday.com UAT board.

### TC-DP-070: Upload Driver Documents

**Status**: ⚠️ FEATURE TO BE CONFIRMED — no upload UI found in current codebase

**Given**: (Assumed) A document-upload section exists in driver onboarding or the driver profile page

**Steps**:
1. Navigate to the document upload section (location TBC by developer)
2. Click **"Upload Document"** or **"Add Document"**
3. Select a valid document file (e.g., PDF or image of driving licence)
4. Click **"Submit"** or **"Save"**

**Expected Result**:
- Document is uploaded successfully
- A confirmation message is shown: "Document uploaded"
- The uploaded document appears in the driver's document list with status "Pending Review" (or similar)
- Vendor Admin / System Admin can view the submitted document

**Pass Criteria**:
- ✅ Document uploads without error
- ✅ Confirmation message displayed
- ✅ Document visible in admin view

**Edge Cases**:
- Unsupported file type (e.g., `.exe`) → should show validation error
- File too large → should show file-size validation error
- Uploading duplicate document → should warn or replace existing

---

### TC-DP-071: Document Expiry Handling

**Status**: ⚠️ FEATURE TO BE CONFIRMED — no expiry UI found in current codebase

**Given**: (Assumed) A driver has an uploaded document with a recorded expiry date

**Steps**:
1. Set the expiry date of an uploaded document to a past date (test data manipulation required)
2. Log in as the driver and navigate to the documents section
3. Observe whether an expiry warning is shown
4. Attempt to start a shift with an expired document (if enforced)

**Expected Result**:
- Expired document is flagged with a visible warning (e.g., red "Expired" badge)
- The driver is prompted to re-upload a valid document
- Depending on business rules: driver may be prevented from starting a shift until the document is renewed

**Pass Criteria**:
- ✅ Expired document is clearly flagged
- ✅ Driver is prompted to upload a new document
- ✅ Shift-start enforcement behaviour matches agreed business rule

**Edge Cases**:
- Document expiring within 30 days → should show "Expiring soon" warning
- No expiry date stored → should handle gracefully (no false-positive warning)

---

## Driver Profile

### TC-DP-060: View Driver Profile / Account Page

**Given**: You are logged in as a driver

**Steps**:
1. Navigate to **Account** or **Profile** (typically in the top navigation)
2. Observe the profile page

**Expected Result**:
- Your profile details are displayed:
  - Name, email, phone number
  - Profile photo (if set)
  - Vehicle type (if applicable)
  - Branch assignment(s) (view only)
- An "Edit" button is visible

**Pass Criteria**:
- ✅ Profile page loads
- ✅ Your details are displayed correctly
- ✅ Edit functionality is accessible

**Edge Cases**:
- Profile with no photo → should show a placeholder icon
- Profile with no phone number → should show "Not set"

---

### TC-DP-061: Update Driver Profile

**Given**: You are viewing your driver profile page

**Steps**:
1. Click **"Edit"** or **"Update Profile"**
2. Modify one or more fields (e.g., phone number, vehicle type)
3. Click **"Save"**

**Expected Result**:
- A success message appears: "Profile updated successfully"
- Updated details are reflected on the profile page

**Pass Criteria**:
- ✅ Changes are saved
- ✅ Success message displayed

**Edge Cases**:
- Invalid phone format → should show validation error
- Clearing required fields → should enforce validation
- Updating email address → should require re-authentication or send a verification email

---

### TC-DP-072: Driver Removed from Vendor Mid-Active Delivery (Mid-Shift Handover)

**Given**: A driver is currently on shift and has a **live delivery in progress** (order collected from restaurant, en route to customer). A Vendor Admin removes or unassigns the driver from the vendor (see **[TC-VA-077](03-vendor-admin-uat.md)** for the VA-side step).

**Steps**:
1. In Vendor Admin, while the driver has an active delivery, remove or unassign the driver from the vendor (or revoke their access)
2. On the Driver Portal (keep open in a separate window), wait for the next page refresh or polling cycle (typically 30–60 seconds)
3. Observe the Driver Portal behaviour: does an error toast appear? Is the active delivery removed? Is any warning shown?
4. Attempt to mark the delivery as complete from the driver's current delivery view

**Expected Result** (graceful behaviour — exact behaviour to be confirmed):
- The driver should receive a clear, actionable notification (e.g., toast: "You have been removed from this vendor. Please contact your manager.")
- The active delivery should **not** silently disappear without warning — if removed, the driver must be told
- The order should be returned to the Vendor Admin kanban so it can be reassigned or resolved
- No order data should be silently lost; all state changes should be logged

**Pass Criteria**:
- ✅ Driver sees a clear notification on next refresh
- ✅ Active delivery state is handled gracefully (no silent data loss)
- ✅ Order reappears in Vendor Admin kanban for reassignment
- ✅ No unhandled exceptions or blank error pages

**Edge Cases**:
- Driver completes the delivery after being removed → system should accept the status update or show a clear "You are no longer assigned to this delivery" message
- Network delay means the driver's app is stale for several minutes → system should handle eventual consistency gracefully

> ℹ️ **Exploratory guidance**: If the behaviour is unclear from observation, treat this as an **exploratory test**. Record exactly what happens, verify no data is silently lost, and **log a bug** if the driver sees a blank page, an unhandled exception, or if the order disappears from the VA kanban without trace. Cross-reference **[TC-XP-030](09-cross-portal-impact-uat.md#tc-xp-030-driver-invite--remove--driver-portal-access-and-order-assignment)** for the full cross-portal driver removal checklist.

---

## Summary and Next Steps

You have now tested all major features of the **Driver Portal**.

### What to do next:

1. **Log all bugs** found during testing on your Monday.com board under the "Driver Portal" group
2. **Verify fixed bugs** when developers mark them as "Fixed"
3. **Move on to the next portal**:
	- **[Customer Front-end UAT](05-customer-frontend-uat.md)**
	- **[Order Saga UAT](06-order-saga-uat.md)** (critical — tests the full order flow across all portals, including Driver Portal delivery actions)

---

**Great work! Keep testing! 🚀**
