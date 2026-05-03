# Vendor Admin Guide

## 1. Overview and access model
The Vendor Admin website is the operational workspace for restaurant owners, vendor managers, and branch operators. It is used to complete onboarding after approval, switch between vendor and branch contexts, manage restaurant information, maintain menus, handle live orders, manage drivers, run offers, review feedback, and maintain delivery or payment settings.

### What this site is for
- finishing onboarding after a vendor application is approved
- maintaining restaurant and branch information
- building menus and publishing menu changes
- managing incoming orders and dispatch-related actions
- inviting and maintaining drivers
- creating vendor offers and reviewing customer feedback
- configuring delivery pricing and payment methods

### Access expectations
This site is intended for authenticated vendor-side users. Some pages are public onboarding pages, while the main admin area requires an authenticated user with a valid vendor relationship.

### How to use this guide
This guide documents the user-facing pages and the main operational flows. Modal windows, AJAX refreshes, and backing endpoints are only mentioned where they affect the visible workflow.

### Related documents
- Start with the [documentation index](index.md) for cross-site navigation.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when onboarding, media processing, or background behaviour affects what the operator sees.

### Typical prerequisites
- a submitted application for the public apply flow, then an approved invitation for onboarding
- an authenticated vendor-linked user for the main admin area
- the correct vendor or branch context selected before editing operational data

### Navigation paths

| Workflow area | Typical entry path |
| --- | --- |
| Applying and onboarding | Apply page -> Application success page -> Onboarding landing -> Onboarding registration -> Onboarding welcome |
| Dashboard and context | Vendor dashboard -> vendor or branch context switching |
| Restaurant management | Vendor dashboard -> Manage restaurant |
| Menu management | Vendor dashboard -> Menu builder list -> Menu editor |
| Orders workflow | Vendor dashboard -> Orders kanban dashboard or All orders |
| Drivers and branch assignment | Vendor dashboard -> Drivers list |
| Offers and reviews | Vendor dashboard -> Vendor offers or Reviews |
| Delivery pricing and payment methods | Vendor dashboard -> Delivery costs or Payment methods |

---

## 2. Applying and onboarding
These pages support the journey from initial application through first-time onboarding.

### Apply page
**Purpose**  
Lets a prospective vendor submit an application to join the platform.

**Key actions**
- provide restaurant and contact details
- submit the application for review

**Expected outcome**  
The business enters the application review process.

### Application success page
**Purpose**  
Confirms that the vendor application was submitted.

**Expected outcome**  
The applicant knows the submission was accepted and can wait for review.

### Onboarding landing page
**Purpose**  
Acts as the entry point for vendor onboarding after approval.

**Expected outcome**  
The user can begin the registration and welcome flow linked to their invitation.

### Onboarding registration page
**Purpose**  
Collects the account information needed to complete onboarding.

**Expected outcome**  
The invited vendor user can register and continue into the admin site.

### Onboarding welcome page
**Purpose**  
Confirms successful onboarding and introduces the next steps.

**Expected outcome**  
The vendor is ready to begin using the dashboard and configuration workflows.

### Privacy
**Purpose**  
Displays privacy information for the Vendor Admin site.

### Error page
**Purpose**  
Explains that onboarding or a requested operation could not be completed.

### End-to-end workflow
1. Submit the initial vendor application.
2. Wait for approval from the platform team.
3. Use the onboarding invitation to open the landing page.
4. Register and finish the welcome flow.
5. Sign in to the main vendor admin area.

---

## 3. Dashboard and vendor context
The dashboard gives the vendor team a working view of the business and supports context switching.

### Vendor dashboard
**Purpose**  
Provides a central operational view for the current vendor or branch context.

**Key actions**
- review performance and operational signals
- navigate to menus, orders, drivers, offers, reviews, and settings

**Expected outcome**  
Users can move quickly into the task they need to complete.

### No vendor context state
**Purpose**  
Handles the case where the signed-in user does not currently have an active vendor context selected.

**Expected outcome**  
The user understands that they must choose or obtain the correct vendor context before proceeding.

### Vendor and branch context switching behaviour
**Purpose**  
Lets users change the active vendor or branch they are managing.

**Expected outcome**  
All admin pages reflect the currently selected operating context.

---

## 4. Restaurant and branch management
These pages maintain the business profile and branch-specific information.

### Manage restaurant page
**Purpose**  
Displays the main restaurant management surface for vendor profile maintenance.

**Key actions**
- review restaurant information
- update editable profile data
- maintain branch-related information

**Expected outcome**  
The restaurant profile shown to the platform remains accurate.

### Branch update flow
**Purpose**  
Allows branch-specific details to be updated from the restaurant management experience.

**Expected outcome**  
Branch data stays current for customers, operations, and reporting.

### End-to-end workflow
1. Select the correct vendor and branch context.
2. Open the restaurant management page.
3. Update vendor or branch details.
4. Save and verify the updated information.

---

## 5. Menu management
Menu management is one of the core day-to-day workflows in Vendor Admin.

### Menu builder list
**Purpose**  
Lists the menus available to the current vendor context.

**Key actions**
- open an existing menu
- create a new menu
- review current publication or status information

**Expected outcome**  
The user can select the menu that needs maintenance.

### Menu editor
**Purpose**  
Provides the main editing workspace for a menu.

**Key actions**
- update menu details
- create, edit, and delete categories
- create, edit, move, reorder, or delete dishes
- upload images for menu content
- manage availability and status
- publish the menu when ready

**Expected outcome**  
The vendor can make structured menu changes from one operational workspace.

### Menu creation flow
**Purpose**  
Creates a new menu for the current vendor or branch.

**Expected outcome**  
A new draft menu becomes available in the builder.

### Menu update flow
**Purpose**  
Saves changes to menu-level information.

**Expected outcome**  
The menu reflects the latest approved structure and content.

### Menu publish flow
**Purpose**  
Makes the latest menu version available for customer-facing use.

**Expected outcome**  
Published menu changes become visible in downstream experiences.

### Menu category reorder flow
**Purpose**  
Changes the order of categories within a menu.

**Expected outcome**  
Categories appear in the intended sequence.

### Dish create, update, delete, move, and reorder flows
**Purpose**  
Maintain the dish catalogue inside the menu editor.

**Expected outcome**  
Dishes remain accurate, well ordered, and attached to the correct category.

### Category create, update, and delete flows
**Purpose**  
Maintain the category structure for the menu.

**Expected outcome**  
The menu remains easy for customers to browse.

### Menu availability and status management
**Purpose**  
Controls whether a menu is active and how it can be used operationally.

**Expected outcome**  
Only the intended menu is available to customers and internal workflows.

### Image upload flow for menu content
**Purpose**  
Uploads menu-related media for dishes or other content.

**Expected outcome**  
The menu can display supporting imagery after processing completes.

### End-to-end workflow
1. Open the menu builder list.
2. Create a new menu or open an existing one.
3. Maintain categories and dishes.
4. Upload images and adjust availability where needed.
5. Publish the menu when it is ready for live use.

---

## 6. Orders workflow
These pages support live order handling and operational follow-up.

### Orders kanban dashboard
**Purpose**  
Provides the main live operations view for active orders.

**Key actions**
- review current order state by column or stage
- open order details
- accept, reject, edit, cancel, or advance an order
- assign or unassign a driver

**Expected outcome**  
Kitchen or operations staff can work through active orders efficiently.

### Order details modal or view
**Purpose**  
Shows the detailed information needed to act on a single order.

**Expected outcome**  
The operator can confirm items, timing, and context before changing status.

### Accept order
**Purpose**  
Confirms that the vendor will fulfil the order.

**Expected outcome**  
The order advances into active preparation.

### Reject order
**Purpose**  
Stops fulfilment when the order cannot be accepted.

**Expected outcome**  
The order leaves the normal fulfilment path with a rejected state.

### Mark ready for pickup
**Purpose**  
Signals that the order is prepared and ready for collection or next-stage delivery handling.

**Expected outcome**  
Downstream dispatch or driver workflows can continue.

### Assign driver
**Purpose**  
Links an available driver to an order.

**Expected outcome**  
The order has a named delivery resource.

### Unassign driver
**Purpose**  
Removes the current driver assignment when dispatch needs to change.

**Expected outcome**  
The order can be reassigned.

### Cancel order
**Purpose**  
Stops an order that can no longer continue.

**Expected outcome**  
The order is clearly marked as cancelled.

### Edit order
**Purpose**  
Makes operational adjustments through the order management flow.

**Expected outcome**  
The order record matches the agreed operational change.

### All orders page
**Purpose**  
Provides a broader order history and lookup view outside the live kanban board.

**Key actions**
- search across orders
- review historic order information

**Expected outcome**  
Users can investigate past orders without relying only on the live dashboard.

### Kanban refresh and stats refresh behaviour
**Purpose**  
Keeps the active order screen current while staff are working.

**Expected outcome**  
The live dashboard reflects recent order activity without requiring a full manual reload.

### End-to-end workflow
1. Use the kanban dashboard to monitor new and active orders.
2. Open order details when a decision is needed.
3. Accept, reject, edit, prepare, assign, unassign, or cancel as required.
4. Use All Orders for lookup and history after the live workflow has moved on.

---

## 7. Drivers and branch assignment
The driver area covers invitation, maintenance, and branch linking.

### Drivers list
**Purpose**  
Lists drivers associated with the current vendor context.

**Key actions**
- review drivers
- invite a new driver
- open driver detail or edit flows
- remove a driver or manage branch assignments

**Expected outcome**  
The vendor can maintain its driver pool.

### Invite driver
**Purpose**  
Starts the vendor-initiated invitation flow for a new driver.

**Expected outcome**  
The invited driver receives the onboarding handoff into the Driver Portal.

### Driver details
**Purpose**  
Shows the current data held for one driver.

**Expected outcome**  
The vendor can review status and assignment information before making changes.

### Edit driver
**Purpose**  
Updates driver details.

**Expected outcome**  
The driver record remains accurate.

### Resend invitation
**Purpose**  
Sends a fresh onboarding invitation when the original invite was missed or expired.

**Expected outcome**  
The driver gets another chance to complete registration.

### Remove driver
**Purpose**  
Deletes or detaches a driver who should no longer be managed by the vendor.

**Expected outcome**  
The driver no longer appears in the active vendor driver list.

### Assign driver to branch
**Purpose**  
Links a driver to a branch for dispatch use.

**Expected outcome**  
The driver is available in the correct branch context.

### Unassign driver from branch
**Purpose**  
Removes a branch relationship when it is no longer valid.

**Expected outcome**  
The driver is no longer presented as assigned to that branch.

### Driver onboarding support
**Purpose**  
Explains the handoff from Vendor Admin into the separate Driver Portal onboarding experience.

**Expected outcome**  
Vendor teams understand that inviting a driver starts a cross-site onboarding process rather than completing all setup inside Vendor Admin.

### End-to-end workflow
1. Open the Drivers list.
2. Invite a new driver or open an existing driver.
3. Update details and branch assignments as required.
4. Resend or remove access when operationally necessary.

---

## 8. Offers and reviews
These pages support vendor-level promotion management and customer feedback handling.

### Vendor offers list
**Purpose**  
Lists the offers belonging to the current vendor.

**Key actions**
- create an offer
- open offer detail
- edit, pause, activate, or clone an offer

**Expected outcome**  
The vendor can manage its own offer catalogue.

### Create offer
**Purpose**  
Creates a new vendor-owned promotional offer.

**Expected outcome**  
A draft or inactive offer is created for later activation.

### Offer details
**Purpose**  
Shows the configuration and current state of one offer.

**Expected outcome**  
The user can confirm the setup before changing it.

### Edit offer
**Purpose**  
Updates an existing offer.

**Expected outcome**  
The offer reflects the latest commercial intent.

### Pause offer
**Purpose**  
Temporarily stops an offer from being applied.

**Expected outcome**  
The offer remains stored but inactive.

### Activate offer
**Purpose**  
Makes an available offer live.

**Expected outcome**  
Eligible customers can receive the offer.

### Clone offer
**Purpose**  
Creates a copy of an existing offer to speed up similar campaign creation.

**Expected outcome**  
The vendor can reuse a successful offer structure without starting from scratch.

### Promo code availability checks
**Purpose**  
Supports offer creation by confirming whether a proposed code can be used.

**Expected outcome**  
The vendor avoids publishing a conflicting promo code.

### Reviews list
**Purpose**  
Shows customer reviews associated with the vendor.

**Key actions**
- review customer feedback
- identify reviews that should be escalated
- flag a review for system admin moderation

**Expected outcome**  
The vendor can monitor feedback and escalate problematic content.

### Flag review for system admin review
**Purpose**  
Escalates a review that needs central moderation.

**Expected outcome**  
The review is passed into the System Admin moderation queue.

### End-to-end workflow
1. Review current offers or customer reviews.
2. Open the relevant offer or feedback item.
3. Apply the commercial change or moderation escalation.
4. Re-check the list to confirm the new state.

---

## 9. Delivery pricing and payment methods
These pages handle local fulfilment pricing and payment configuration.

### Delivery costs page
**Purpose**  
Maintains delivery pricing rules for the current vendor or branch.

**Key actions**
- review existing delivery cost tiers
- update delivery pricing values

**Expected outcome**  
Delivery quotes use the intended pricing structure.

### Payment methods page
**Purpose**  
Maintains the payment options available to the vendor.

**Key actions**
- configure Stripe account or payment settings
- configure cash payment support
- remove payment configuration when needed

**Expected outcome**  
Customers are offered the intended payment methods during checkout.

### End-to-end workflow
1. Open Delivery Costs or Payment Methods.
2. Review the current configuration.
3. Save the required changes.
4. Confirm the new settings are shown in the page state.

---

## 10. Related automation and operator notes
Several visible vendor workflows depend on background processing or actions outside the current page.

### Important operator-visible automation
- approval in the System Admin site precedes the vendor onboarding invitation flow used here
- menu and content image uploads are processed asynchronously after the initial upload step
- the live orders board uses refresh behaviour to keep operational information current
- inviting a driver hands the user off into Driver Portal onboarding by email or invitation link
- payment and offer-related checks may validate data before the user can complete a workflow

### Documentation note
Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the shared reference behind onboarding emails, image processing, and other delayed or background work.

---

## 11. Screenshot candidates for first release

- apply page with the main vendor application fields
- onboarding landing or registration page from an approved invitation
- vendor dashboard with context switching visible
- manage restaurant page
- menu editor with categories and dishes visible
- orders kanban dashboard
- drivers list or driver detail
- offer detail or list
- payment methods page
