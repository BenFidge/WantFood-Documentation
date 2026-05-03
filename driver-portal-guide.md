# Driver Portal Guide

## 1. Overview and onboarding
The Driver Portal is the workspace for delivery drivers. It is used to complete onboarding after invitation, start and end shifts, keep live location current, manage active deliveries, review route planning results, maintain weekly availability, and update profile information.

### What this site is for
- completing driver onboarding after invitation
- starting and ending shifts
- keeping delivery location data current
- reviewing active deliveries and updating delivery status
- using route optimisation during a working shift
- maintaining availability and personal profile details

### Access expectations
Some onboarding pages are public and invitation-driven. The main working area requires an authenticated driver account with valid driver access.

### How to use this guide
This guide focuses on the visible driver workflows. Supporting refresh endpoints and technical handlers are described only when they explain what the driver sees on screen.

### Related documents
- Start with the [documentation index](index.md) for cross-site navigation.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when onboarding, live updates, or route optimisation behaviour needs additional background.

### Typical prerequisites
- a valid driver invitation for the onboarding journey
- a completed driver onboarding registration before entering the working area
- an authenticated driver account to access shift, delivery, availability, and account pages

### Navigation paths

| Workflow area | Typical entry path |
| --- | --- |
| Public onboarding | Home landing page or invitation link -> Onboarding landing -> Onboarding registration -> Onboarding welcome |
| Dashboard and shift controls | Driver dashboard home -> Start shift or End shift |
| Deliveries workflow | Driver dashboard home -> Active deliveries -> Delivery dashboard or Delivery detail |
| Route planning | Delivery dashboard map or route page -> Optimize route or Clear route |
| Availability management | Driver dashboard home -> Availability schedule page |
| Account management | Driver dashboard home -> Account or profile page |

---

## 2. Public onboarding pages
These pages take a driver from invitation into an active portal account.

### Onboarding landing page
**Purpose**  
Acts as the first page opened from a driver invitation.

**Expected outcome**  
The invited driver can begin registration.

### Onboarding registration page
**Purpose**  
Collects the account information required to create the driver's portal access.

**Expected outcome**  
The driver completes registration and can continue onboarding.

### Onboarding welcome page
**Purpose**  
Confirms that onboarding is complete and introduces the next step.

**Expected outcome**  
The driver is ready to sign in and use the portal.

### Home landing page
**Purpose**  
Provides the public entry page for the Driver Portal.

**Expected outcome**  
Drivers and invited users have a clear starting point for access.

### Error page
**Purpose**  
Explains when onboarding or another public action could not be completed.

**Expected outcome**  
The user understands that the current operation failed and may need to retry or request support.

### End-to-end workflow
1. Receive a driver invitation from the vendor team.
2. Open the onboarding landing page.
3. Register the account.
4. Complete the welcome flow.
5. Sign in to the main driver work area.

---

## 3. Dashboard and shift controls
The dashboard is the central work area during an active shift.

### Driver dashboard home
**Purpose**  
Provides the main signed-in landing page for drivers.

**Key actions**
- review current shift state
- open active deliveries and route information
- access availability and account pages

**Expected outcome**  
The driver can begin work quickly and see what needs attention.

### Start shift
**Purpose**  
Moves the driver into an active working state.

**Expected outcome**  
The platform records that the driver is available for operational work.

### End shift
**Purpose**  
Closes the driver's active working period.

**Expected outcome**  
The platform records that the driver is no longer available for delivery assignment.

### Shift status
**Purpose**  
Shows whether the driver is currently on or off shift.

**Expected outcome**  
The driver has a clear readback of current availability.

### Live location update behaviour
**Purpose**  
Keeps current location information flowing while the driver is working.

**Expected outcome**  
Dispatch and live delivery screens can use up-to-date location data.

### Current location readback
**Purpose**  
Lets the driver confirm that the portal is using the expected location.

**Expected outcome**  
The driver can spot if location sharing is not behaving as expected.

### End-to-end workflow
1. Sign in to the dashboard.
2. Start the shift.
3. Confirm the shift status and location readback.
4. Continue into active deliveries.
5. End the shift when work is complete.

---

## 4. Deliveries workflow
These pages support the core delivery lifecycle from active work through history.

### Active deliveries page
**Purpose**  
Lists the deliveries currently assigned to the driver.

**Key actions**
- review active jobs
- open the delivery dashboard or delivery detail page

**Expected outcome**  
The driver can select the next job that needs action.

### Delivery dashboard map or route page
**Purpose**  
Combines live delivery information with routing support.

**Key actions**
- review the route and assigned stops
- update delivery status
- use route optimisation actions

**Expected outcome**  
The driver can manage the current run from a single working page.

### Delivery detail page
**Purpose**  
Shows the detail for one delivery.

**Key actions**
- review order and address context
- confirm the next status action

**Expected outcome**  
The driver has enough information to continue the delivery correctly.

### Delivery history page
**Purpose**  
Provides a record of past deliveries.

**Expected outcome**  
The driver can review recently completed or older work.

### Update delivery status
**Purpose**  
Moves the delivery through its working states.

**Expected outcome**  
The delivery record reflects the real-world progress of the run.

### Mark delivered
**Purpose**  
Confirms successful handoff of the order.

**Expected outcome**  
The delivery closes in a delivered state.

### Mark failed or report issue
**Purpose**  
Records that a delivery could not be completed normally.

**Expected outcome**  
The delivery captures the issue and leaves the standard success path.

### Revert delivery status
**Purpose**  
Corrects an earlier status change when the wrong state was applied.

**Expected outcome**  
The delivery returns to the correct working state.

### Decline delivery
**Purpose**  
Lets the driver refuse a delivery they cannot take.

**Expected outcome**  
The delivery can be reassigned or otherwise handled by the platform.

### Dashboard partial refresh behaviour
**Purpose**  
Keeps live screens current while the driver is working.

**Expected outcome**  
The portal updates active information without forcing the driver through a full page reload.

### End-to-end workflow
1. Open Active Deliveries.
2. Select the required delivery and review its detail or dashboard state.
3. Follow the route and update statuses as work progresses.
4. Mark the delivery delivered, failed, declined, or corrected as needed.
5. Use Delivery History to review completed work later.

---

## 5. Route planning
Route planning is part of the main delivery dashboard workflow rather than a separate standalone page.

### Route planner entry point
**Purpose**  
Gives the driver access to route planning actions from the delivery workflow.

**Expected outcome**  
The driver can run route planning without leaving the delivery context.

### Optimize route action
**Purpose**  
Calculates an improved route order for the active delivery set.

**Expected outcome**  
The dashboard shows an optimised sequence for current stops.

### Clear route action
**Purpose**  
Removes the current optimisation result.

**Expected outcome**  
The dashboard no longer uses the optimised route arrangement.

### Optimized route retrieval in the dashboard
**Purpose**  
Displays the latest route planning result inside the live delivery experience.

**Expected outcome**  
The driver sees current route guidance while staying in the main dashboard flow.

### End-to-end workflow
1. Open the delivery dashboard.
2. Trigger route optimisation when multiple stops need planning.
3. Review the returned route order.
4. Clear or rerun the route if the working situation changes.

---

## 6. Availability management
These pages help drivers communicate when they can work.

### Availability schedule page
**Purpose**  
Displays the driver's weekly availability settings.

**Key actions**
- review current availability
- update the schedule
- apply a preset if one is available

**Expected outcome**  
The platform holds the driver's latest availability pattern.

### Update weekly availability
**Purpose**  
Saves changes to the normal working schedule.

**Expected outcome**  
Future scheduling and dispatch processes can use the updated availability.

### Apply availability preset
**Purpose**  
Speeds up schedule changes using a predefined pattern.

**Expected outcome**  
The driver can update availability faster than editing every time slot manually.

### End-to-end workflow
1. Open the Availability page.
2. Review the current weekly schedule.
3. Edit the required days or apply a preset.
4. Save and confirm the updated schedule.

---

## 7. Account management
The account area holds the driver's personal profile information.

### Account or profile page
**Purpose**  
Displays the main personal profile page.

**Key actions**
- review current profile details
- update account information

**Expected outcome**  
The driver keeps their profile information current.

### Update profile
**Purpose**  
Saves changes made to personal details.

**Expected outcome**  
The latest profile information is stored for operational and contact use.

---

## 8. Related automation and operator notes
Several visible driver workflows depend on background or asynchronous behaviour.

### Important operator-visible automation
- vendor-side driver invitations hand off into the onboarding flow documented in this guide
- shift status and live delivery screens rely on refresh behaviour to keep the current state visible
- live location updates support dispatch and delivery tracking while the shift is active
- route optimisation runs when the driver explicitly requests it from the dashboard rather than on a fixed schedule

### Documentation note
Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the shared reference behind invitation handoff, live refresh behaviour, and route optimisation.

---

## 9. Screenshot candidates for first release

- onboarding landing page from a driver invitation
- driver dashboard home with shift status
- active deliveries page
- delivery dashboard with route or map context
- delivery detail page
- availability schedule page
- account or profile page
