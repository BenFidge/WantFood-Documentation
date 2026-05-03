# System Admin Guide

## 1. Overview and permissions
The System Admin website is the operational control surface for the WantFood platform. It is used by platform administrators, operations staff, and commercial or content administrators who need to review applications, manage vendors, investigate orders, maintain commercial settings, moderate content, and run recovery tools.

### What this site is for
- reviewing and approving new vendor applications
- managing active and inactive vendors
- supporting order, commission, and invoice operations
- maintaining platform-wide offers and homepage content
- moderating reviews and content safety settings
- configuring payment providers and running operational tools

### Typical access expectations
Access is intended for trusted internal users with administrative responsibility. Different admin roles may be granted access to different areas, but this guide assumes the reader is using an account with permission to open the main administration workflows.

### How to use this guide
This guide focuses on user-visible pages and actions. Background handlers, JSON endpoints, and technical support calls are intentionally omitted unless they explain an operator-facing workflow.

### Related documents
- Start with the [documentation index](index.md) for cross-site navigation.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when a workflow depends on scheduled processing, background work, or delayed completion.

### Typical prerequisites
- an internal admin account with access to System Admin
- the required role or permission for the area being documented
- the relevant business record ready for lookup, such as an application, vendor, order, invoice, or offer

### Navigation paths

| Workflow area | Typical entry path |
| --- | --- |
| Application review | Home or dashboard -> Applications list -> Application details |
| Vendor and user administration | Home or dashboard -> Vendors list, Vendor users list, or Users list |
| Orders, commissions, and invoices | Home or dashboard -> Orders, Commissions, or Invoices |
| Promotions and reviews | Home or dashboard -> Platform offers or Flagged reviews |
| Content management | Home or dashboard -> Content asset library, Hero slides, Regions, or Support Local slides |
| Taxonomy, safety, settings, and payments | Home or dashboard -> Cuisine types, Bad words, Settings, or Payment settings |
| Operational tools | Home or dashboard -> Tools dashboard |

---

## 2. Dashboard

### Home / dashboard
**Purpose**  
Provides an operational overview of the platform and acts as the main entry point into day-to-day administration.

**Who uses it**  
Platform administrators and operations staff.

**Key actions**
- review headline operational indicators
- navigate into applications, vendors, orders, commissions, invoices, offers, reviews, content, and tools
- use the dashboard as the starting point for investigation or recovery work

**Expected outcome**  
Administrators can quickly identify where attention is needed and move into the relevant workflow.

### Privacy
**Purpose**  
Displays the platform privacy notice for the admin site.

**Expected outcome**  
Users can review privacy information without leaving the admin experience.

### Error pages
**Purpose**  
Communicate that the requested page or operation could not be completed.

**Expected outcome**  
The user understands that the current task failed and can retry, navigate elsewhere, or escalate the issue.

---

## 3. Application review
This workflow covers the intake and decision process for new vendor applications.

### Applications list
**Purpose**  
Shows submitted applications waiting for review or follow-up.

**Key actions**
- scan current applications
- open an individual application
- identify applications that need approval or rejection

**Expected outcome**  
Administrators can prioritise application handling and open the correct record.

### Application details
**Purpose**  
Displays the information submitted by a prospective vendor.

**Key actions**
- review submitted business details
- verify readiness for onboarding
- decide whether to approve or reject

**Expected outcome**  
The reviewer has enough information to make a decision.

### Approve application
**Purpose**  
Confirms that an application should progress into onboarding.

**Key actions**
- approve the application from the detail workflow
- confirm the approval action

**Expected outcome**  
The vendor moves into the next onboarding stage and the supporting invitation process can begin.

### Reject application
**Purpose**  
Stops an application that does not meet platform requirements.

**Key actions**
- reject the application from the detail workflow
- provide the rejection decision through the admin flow

**Expected outcome**  
The application is closed out and no onboarding access is issued.

### End-to-end workflow
1. Open the applications list.
2. Select an application and review the detail page.
3. Confirm the business is suitable for the platform.
4. Approve the application to begin onboarding, or reject it to end the process.

---

## 4. Vendor and user administration
This area covers vendor lifecycle management and the administration of both vendor-linked users and internal admin users.

### Vendors list
**Purpose**  
Provides a searchable operational list of registered vendors.

**Key actions**
- review vendor status
- open vendor details
- identify vendors needing maintenance, activation, or deactivation

**Expected outcome**  
Administrators can quickly locate the vendor they need to manage.

### Vendor details
**Purpose**  
Shows the current administrative view of a vendor.

**Key actions**
- review vendor profile data
- inspect current status and important fields
- move into edit, activate, or deactivate actions

**Expected outcome**  
The admin can confirm the vendor state before making changes.

### Edit vendor
**Purpose**  
Updates vendor information held by the platform.

**Key actions**
- change editable vendor details
- save the updated record

**Expected outcome**  
The vendor record reflects the latest approved information.

### Activate vendor
**Purpose**  
Makes a previously inactive vendor available for active platform use.

**Expected outcome**  
The vendor returns to an active operational state.

### Deactivate vendor
**Purpose**  
Prevents a vendor from participating in active platform workflows.

**Expected outcome**  
The vendor is placed into an inactive state until reactivated.

### Vendor users list
**Purpose**  
Shows vendor-linked user accounts managed through the admin site.

**Key actions**
- locate the user attached to a vendor
- start a vendor-to-user transfer workflow
- remove a vendor-linked user when required

**Expected outcome**  
User ownership and vendor access remain accurate.

### Vendor-to-user lookup and transfer workflow
**Purpose**  
Helps administrators move or correct the vendor relationship attached to a user.

**Expected outcome**  
The correct user is linked to the correct vendor record.

### Delete vendor user
**Purpose**  
Removes a vendor user from the administrative record.

**Expected outcome**  
Unwanted or invalid vendor-linked access is removed.

### Users list
**Purpose**  
Lists internal admin users who can access the System Admin site.

**Key actions**
- review current admin access
- add a new admin user
- delete an admin user who no longer needs access

**Expected outcome**  
Internal admin access remains current and controlled.

### Add admin user
**Purpose**  
Creates a new internal admin account or grants admin access to an existing user identity.

**Expected outcome**  
A new administrator can access the relevant admin workflows.

### Delete admin user
**Purpose**  
Removes admin access when it is no longer required.

**Expected outcome**  
The user can no longer use the System Admin site as an administrator.

### End-to-end workflow
1. Open the vendor or user list that matches the issue.
2. Review the current record and confirm the required change.
3. Apply the edit, activation, deactivation, transfer, add, or delete action.
4. Re-check the list or detail page to confirm the change is visible.

---

## 5. Orders, commissions, and invoices
These pages support order investigation, commercial configuration, and vendor invoicing.

### Orders list
**Purpose**  
Provides an administrative view of customer orders.

**Key actions**
- search for an order
- open order details for investigation
- support operations and dispute handling

**Expected outcome**  
The administrator can find and inspect the relevant order.

### Order details
**Purpose**  
Shows the administrative detail for a single order.

**Key actions**
- review order status, values, and lines
- inspect the order context for support work

**Expected outcome**  
Operations staff can understand what happened on the order and decide on any follow-up.

### Commissions dashboard
**Purpose**  
Summarises commission settings and assignments across the platform.

**Key actions**
- review current commission behaviour
- open related tier management pages
- inspect assignments and configuration

**Expected outcome**  
Commercial administrators can understand the current commission model.

### Commission tiers list
**Purpose**  
Lists the commission tiers available for platform use.

**Key actions**
- review existing tiers
- create a tier
- update a tier
- delete a tier when it is no longer needed

**Expected outcome**  
The commission tier catalogue stays aligned to commercial policy.

### Commission configuration
**Purpose**  
Controls broader commission settings that affect how tiers are applied.

**Expected outcome**  
Platform-level commission behaviour matches the intended commercial rules.

### Commission assignments
**Purpose**  
Shows or supports how tiers are applied to vendors.

**Expected outcome**  
Administrators can confirm that vendors are assigned to the correct commission arrangement.

### Commission tier recalculation trigger
**Purpose**  
Runs a manual recalculation when an operator needs to refresh commission outcomes outside the normal schedule.

**Expected outcome**  
Tier assignments and dependent results are recalculated.

### Uninvoiced dashboard
**Purpose**  
Highlights billable activity that has not yet been invoiced.

**Key actions**
- review pending invoice candidates
- decide whether to generate an invoice for a vendor or billing period

**Expected outcome**  
Finance or operations staff can prepare invoice generation.

### Uninvoiced line items
**Purpose**  
Provides the detailed billable lines behind the uninvoiced dashboard.

**Expected outcome**  
The admin can validate what will be included before generating invoices.

### Invoice list
**Purpose**  
Shows previously generated invoices.

**Key actions**
- review invoice history
- open invoice detail
- identify invoices that need payment updates or cancellation

**Expected outcome**  
Users can locate the invoice that needs action.

### Invoice detail
**Purpose**  
Displays invoice content and current status for one invoice.

**Key actions**
- review amounts and included lines
- mark an invoice as paid
- cancel an invoice when required

**Expected outcome**  
The invoice lifecycle can be managed from creation through settlement or cancellation.

### Generate single vendor invoice
**Purpose**  
Creates an invoice for one vendor from the uninvoiced workflow.

**Expected outcome**  
The selected vendor receives a new invoice record for the chosen billable activity.

### Generate monthly invoices
**Purpose**  
Runs a broader monthly billing action across eligible vendors.

**Expected outcome**  
The platform creates invoices for the billing run without requiring each vendor to be processed individually.

### Mark invoice paid
**Purpose**  
Updates an invoice to reflect successful settlement.

**Expected outcome**  
The invoice status reflects payment completion.

### Cancel invoice
**Purpose**  
Stops an invoice from remaining active when it should no longer be collectible.

**Expected outcome**  
The invoice is clearly marked as cancelled.

### End-to-end workflow
1. Use Orders to investigate operational issues.
2. Use Commissions to maintain pricing and fee rules.
3. Review uninvoiced activity before generating invoices.
4. Use invoice detail to maintain paid or cancelled status after billing.

---

## 6. Promotions and reviews
This area supports platform-wide campaigns and moderation work.

### Platform offers list
**Purpose**  
Lists offers created at the platform level.

**Key actions**
- review active and inactive offers
- create a new offer
- open an offer for detail or editing

**Expected outcome**  
Commercial administrators can manage the current platform campaign set.

### Create platform offer
**Purpose**  
Creates a new centrally managed offer.

**Expected outcome**  
A new offer becomes available for later activation and monitoring.

### Platform offer details
**Purpose**  
Shows the current offer configuration and performance context.

**Expected outcome**  
The admin can confirm how the offer is configured before changing its state.

### Edit platform offer
**Purpose**  
Updates offer content or commercial settings.

**Expected outcome**  
The offer reflects the latest approved campaign setup.

### Pause platform offer
**Purpose**  
Temporarily stops an offer without deleting it.

**Expected outcome**  
The offer is no longer actively applied while remaining available for later reactivation.

### Activate platform offer
**Purpose**  
Makes a paused or newly created offer active.

**Expected outcome**  
The offer becomes live for eligible users.

### Delete platform offer
**Purpose**  
Removes an offer that should no longer exist in the catalogue.

**Expected outcome**  
The offer no longer appears as an available campaign item.

### Offer analytics
**Purpose**  
Provides the operator view of campaign performance.

**Expected outcome**  
Commercial teams can judge whether an offer is effective.

### Flagged reviews list
**Purpose**  
Collects reviews that need human moderation.

**Key actions**
- inspect flagged content
- decide whether to resolve the case
- trigger a rating refresh when moderation changes affect scores

**Expected outcome**  
Moderation work is managed from a central queue.

### Resolve flagged review
**Purpose**  
Records the moderation outcome for a flagged review.

**Expected outcome**  
The review leaves the active moderation queue with a defined resolution.

### Manual rating recalculation trigger
**Purpose**  
Refreshes platform ratings after review changes or moderation decisions.

**Expected outcome**  
Displayed ratings are recalculated instead of waiting for the scheduled background run.

### End-to-end workflow
1. Review current offers or flagged reviews.
2. Open the specific item that needs intervention.
3. Apply the commercial or moderation action.
4. Re-check status, analytics, or recalculated ratings where relevant.

---

## 7. Content management
This section manages the assets and homepage content shown across customer experiences.

### Content asset library
**Purpose**  
Stores and organises uploaded content assets.

**Key actions**
- browse existing assets
- upload a new asset
- update metadata
- delete obsolete content

**Expected outcome**  
Approved assets remain available for use in page and campaign content.

### Upload asset
**Purpose**  
Adds a new media asset to the shared content library.

**Expected outcome**  
The uploaded asset becomes available after the platform finishes processing it.

### Update asset metadata
**Purpose**  
Improves how assets are identified and reused.

**Expected outcome**  
The asset library stays organised and easier to maintain.

### Delete asset
**Purpose**  
Removes content that should no longer be available.

**Expected outcome**  
The deleted asset no longer appears in the library.

### Hero assets editor
**Purpose**  
Manages the assets used by hero carousel content.

**Expected outcome**  
Homepage hero content can be maintained with the correct media set.

### Hero slides list
**Purpose**  
Lists the slides available for the homepage hero carousel.

**Key actions**
- create a slide
- open an existing slide for editing
- publish, unpublish, reorder, or preview slides

**Expected outcome**  
Admins can maintain the live homepage hero sequence.

### Create hero slide
**Purpose**  
Builds a new hero slide.

**Expected outcome**  
A new draft slide is available for further editing or publishing.

### Edit hero slide
**Purpose**  
Updates slide content and presentation settings.

**Expected outcome**  
The slide reflects the latest approved messaging.

### Publish hero slide
**Purpose**  
Makes a slide available for the live carousel.

**Expected outcome**  
The slide can appear on the customer-facing homepage.

### Unpublish hero slide
**Purpose**  
Removes a slide from the live carousel without deleting it.

**Expected outcome**  
The slide is retained but no longer displayed.

### Reorder hero slides
**Purpose**  
Changes the display sequence of hero slides.

**Expected outcome**  
Slides appear in the intended order.

### Hero slide preview
**Purpose**  
Lets an administrator review how a slide will appear before or after publishing.

**Expected outcome**  
The admin can confirm content quality before it goes live.

### Regions list
**Purpose**  
Manages the regional entities used in content targeting and related administration.

**Key actions**
- create a region
- edit a region
- delete a region

**Expected outcome**  
Regional content structures stay accurate.

### Support Local slides list
**Purpose**  
Lists slides used for the Support Local content area.

**Key actions**
- create, edit, publish, unpublish, reorder, or delete slides

**Expected outcome**  
The Support Local area reflects current campaign content.

### End-to-end workflow
1. Upload or select the required content asset.
2. Create or edit the hero or Support Local slide.
3. Preview and confirm the content.
4. Publish and reorder the item as needed.

---

## 8. Taxonomy, safety, settings, and payments
These pages maintain platform-wide classification, content safety, and payment configuration.

### Cuisine types list
**Purpose**  
Maintains the cuisine types shown across customer discovery experiences.

**Key actions**
- review cuisine entries
- update display order for mobile and web experiences

**Expected outcome**  
Cuisine browsing stays relevant and correctly ordered.

### Bad words list
**Purpose**  
Maintains the blocked or flagged terms used in moderation and content safety workflows.

**Key actions**
- add a bad word
- edit an existing bad word
- delete an entry
- import words in bulk

**Expected outcome**  
Content safety rules remain current and enforceable.

### Settings page for bad words save flow
**Purpose**  
Provides the supporting settings workflow used when saving moderation-related changes.

**Expected outcome**  
Bad word administration changes are persisted correctly.

### Payment settings
**Purpose**  
Configures platform-level payment provider settings.

**Key actions**
- save Stripe platform settings
- save WorldPay platform settings
- remove Stripe configuration
- remove WorldPay configuration

**Expected outcome**  
The platform payment configuration matches the intended live provider setup.

---

## 9. Tools and operational recovery actions
The tools area is used for controlled operational maintenance.

### Tools dashboard
**Purpose**  
Central entry point for admin-triggered operational actions.

**Key actions**
- reindex vendors
- rebuild caches
- import ONS postcode data
- review postcode output where exposed in tool results
- run integrity checks

**Expected outcome**  
Administrators can launch maintenance operations without leaving the site.

### Reindex vendors tool
**Purpose**  
Refreshes search visibility data for vendors and dishes.

**Expected outcome**  
Search results and related read models are republished in the background.

### Rebuild caches tool
**Purpose**  
Refreshes cached vendor and menu data.

**Expected outcome**  
Downstream experiences read updated cached content.

### Import ONS postcodes tool
**Purpose**  
Loads postcode reference data required for platform operations.

**Expected outcome**  
Postcode-backed workflows use refreshed reference data.

### Integrity check tool
**Purpose**  
Runs vendor or platform integrity checks to identify data issues.

**Expected outcome**  
Operations staff receive a current view of detected issues and can plan remediation.

### End-to-end workflow
1. Open the Tools dashboard.
2. Select the maintenance action that matches the operational issue.
3. Trigger the action.
4. Confirm the resulting status or follow-up output.

---

## 10. Related automation and operator notes
Several admin workflows rely on asynchronous or scheduled processing.

### Important operator-visible automation
- approving vendor applications supports the downstream onboarding invitation workflow
- manual commission recalculation refreshes vendor fee assignments outside the nightly schedule
- manual rating recalculation refreshes displayed scores after moderation work
- invoice generation actions create billing records for later payment handling
- content asset uploads rely on asynchronous image processing after the initial upload step
- reindex and cache rebuild tools launch background processing instead of completing all work inline

### Documentation note
Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the shared background-processing reference behind these admin workflows.

---

## 11. Screenshot candidates for first release

- dashboard home with the main operational navigation visible
- application details showing approve or reject decision context
- vendor details or edit vendor workflow
- commissions dashboard or assignments view
- invoice detail showing lifecycle actions
- flagged reviews list with moderation queue context
- hero slides list or preview workflow
- tools dashboard with the main maintenance actions visible
