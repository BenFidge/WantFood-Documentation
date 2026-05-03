# WantFood Website Documentation Plan

## Purpose
Create a documentation set for the four primary websites in the WantFood platform:

1. System Admin
2. Vendor Admin
3. Driver Portal
4. Customer front-end

The documentation set should explain:
- what each website is for
- who uses it
- every user-facing page that exists today
- the major features available on each site
- the automated tasks, scheduled jobs, and background processes that support the platform

This document is a planning artifact for producing the full documentation set.

---

## Proposed Deliverables

### Core documents
- `docs/documentation/index.md` - documentation landing page and navigation
- `docs/documentation/system-admin-guide.md` - System Admin user guide
- `docs/documentation/vendor-admin-guide.md` - Vendor Admin user guide
- `docs/documentation/driver-portal-guide.md` - Driver Portal user guide
- `docs/documentation/customer-front-end-guide.md` - Customer-facing website guide
- `docs/documentation/automation-and-jobs.md` - shared appendix for scheduled jobs, background processes, and admin-triggered automation
- `docs/documentation/complex-processes.md` - shared appendix for cross-service workflows, sagas, and other complex platform processes

### Optional supporting documents
- `docs/documentation/glossary.md` - shared platform terms
- `docs/documentation/troubleshooting.md` - common operational issues and where to diagnose them
- `docs/documentation/release-checklist.md` - what to update when features or pages change

---

## Documentation Principles
- Prefer business-facing language over implementation detail.
- Keep each website guide task-oriented.
- Document user-visible behavior first, then note important constraints, permissions, and background automation.
- Treat AJAX and JSON endpoints as implementation details unless they directly support an operator workflow.
- Include screenshots later, but finish the written flow and page inventory first.

---

## Scope Summary by Website

## 1. System Admin

### Primary audience
- Platform administrators
- Operations staff
- Content and commercial administrators

### Pages to document
The following user-facing pages are currently in scope for the System Admin guide.

#### Entry and shell pages
- Home / dashboard
- Privacy
- Error pages

#### Application management
- Applications list
- Application details
- Approve application
- Reject application

#### Vendor management
- Vendors list
- Vendor details
- Edit vendor
- Activate vendor
- Deactivate vendor

#### Vendor user management
- Vendor users list
- Vendor-to-user lookup and transfer workflow
- Delete vendor user

#### Admin user management
- Users list
- Add admin user
- Delete admin user

#### Order and commission management
- Orders list
- Order details
- Commissions dashboard
- Commission tiers list
- Commission tier create
- Commission tier update
- Commission tier delete
- Commission configuration
- Commission assignments
- Commission tier recalculation trigger

#### Invoice management
- Uninvoiced dashboard
- Uninvoiced line items
- Invoice list
- Invoice detail
- Generate single vendor invoice
- Generate monthly invoices
- Mark invoice paid
- Cancel invoice

#### Offers and promotions
- Platform offers list
- Create platform offer
- Platform offer details
- Edit platform offer
- Pause platform offer
- Activate platform offer
- Delete platform offer
- Offer analytics

#### Review moderation
- Flagged reviews list
- Resolve flagged review
- Manual rating recalculation trigger

#### Content management
- Content asset library
- Upload asset
- Update asset metadata
- Delete asset
- Hero assets editor
- Hero slides list
- Create hero slide
- Edit hero slide
- Publish hero slide
- Unpublish hero slide
- Reorder hero slides
- Hero slide preview
- Regions list
- Create region
- Edit region
- Delete region
- Support Local slides list
- Create Support Local slide
- Edit Support Local slide
- Publish Support Local slide
- Unpublish Support Local slide
- Reorder Support Local slides
- Delete Support Local slide

#### Taxonomy and content safety
- Cuisine types list
- Cuisine type ordering for mobile
- Cuisine type ordering for web
- Bad words list
- Add bad word
- Edit bad word
- Delete bad word
- Bulk import bad words
- Settings page for bad words save flow

#### Platform settings and tools
- Payment settings
- Save Stripe platform settings
- Save WorldPay platform settings
- Remove Stripe configuration
- Remove WorldPay configuration
- Tools dashboard
- Reindex vendors tool
- Rebuild caches tool
- Import ONS postcodes tool
- Postcode stats view
- Integrity check tool

### Major features to explain
- platform dashboard and operational overview
- vendor application approval workflow
- vendor lifecycle administration
- admin user management
- order investigation and support
- commission and invoice operations
- promotion management
- review moderation and rating maintenance
- homepage and campaign content administration
- payment provider configuration
- operational tools and recovery actions

---

## 2. Vendor Admin

### Primary audience
- restaurant owners
- vendor managers
- branch operators
- commercial teams managing menus, orders, and offers

### Pages to document

#### Public onboarding and application pages
- Apply page
- Application success page
- Onboarding landing page
- Onboarding registration page
- Onboarding welcome page
- Privacy
- Error page

#### Dashboard and context
- Vendor dashboard
- No vendor context state
- Vendor and branch context switching behavior

#### Restaurant management
- Manage restaurant page
- Branch update flow

#### Menu management
- Menu builder list
- Menu editor
- Menu creation flow
- Menu update flow
- Menu publish flow
- Menu category reorder flow
- Dish create flow
- Dish update flow
- Dish delete flow
- Category create flow
- Category update flow
- Category delete flow
- Dish move and reorder flows
- Menu availability/status management
- Image upload flow for menu content

#### Order operations
- Orders kanban dashboard
- Order details modal/view
- Accept order
- Reject order
- Mark ready for pickup
- Assign driver
- Unassign driver
- Cancel order
- Edit order
- All orders page
- Order history/detail JSON-backed views used by the admin screens
- Kanban refresh and stats refresh behaviors

#### Driver management
- Drivers list
- Invite driver
- Driver details
- Edit driver
- Resend invitation
- Remove driver
- Assign driver to branch
- Unassign driver from branch

#### Driver onboarding support
- Vendor-initiated invitation flow
- Expected driver onboarding handoff to Driver Portal

#### Promotions and reviews
- Vendor offers list
- Create offer
- Offer details
- Edit offer
- Pause offer
- Activate offer
- Clone offer
- Promo code availability checks
- Reviews list
- Flag review for system admin review

#### Delivery and payment settings
- Delivery costs page
- Payment methods page
- Stripe account/payment configuration
- Cash payment configuration
- Remove payment configuration

### Major features to explain
- vendor onboarding after application approval
- vendor and branch context switching
- restaurant profile and branch maintenance
- structured menu editing and publishing
- live order handling and kitchen workflows
- driver invitation and branch assignment
- vendor-level promotions
- review flagging and moderation escalation
- delivery pricing and payment method setup

---

## 3. Driver Portal

### Primary audience
- delivery drivers
- dispatch-linked delivery staff

### Pages to document

#### Public onboarding pages
- Onboarding landing page
- Onboarding registration page
- Onboarding welcome page
- Home landing page
- Error page

#### Driver work area
- Driver dashboard home
- Start shift
- End shift
- Shift status
- Live location update behavior
- Current location readback

#### Deliveries
- Active deliveries page
- Delivery dashboard map/route page
- Delivery detail page
- Delivery history page
- Update delivery status
- Mark delivered
- Mark failed / report issue
- Revert delivery status
- Decline delivery
- Dashboard partial refresh behaviors used by the live delivery screens

#### Route planning
- Route planner entry point
- Optimize route action
- Clear route action
- Optimized route retrieval used by the dashboard

#### Availability and account
- Availability schedule page
- Update weekly availability
- Apply availability preset
- Account/profile page
- Update profile

### Major features to explain
- accepting the invitation and completing onboarding
- starting and ending a shift
- keeping live location updated
- managing active deliveries
- viewing route optimization results
- reporting delivery issues
- managing weekly availability
- maintaining driver profile details

---

## 4. Customer Front-end

### Primary audience
- customers ordering food
- returning customers managing orders and addresses

### Pages to document

#### Browse and discovery
- Home page
- Nearby vendors panel/flow
- Search page
- Cuisine types page
- Vendor page
- Dish page
- Privacy page
- Error page

#### Basket and checkout
- Basket experience summary
- Vendor conflict handling in basket
- Quantity update flow
- Remove item flow
- Menu context and scheduled order behavior
- Checkout page
- Delivery quote creation flow
- Place order flow
- Payment intent polling flow for card payments

#### Orders and post-order experience
- Orders list / order history
- Order details
- Order confirmation
- Live order tracking
- Cancel order
- Add tip
- Complete order / review prompt page
- Submit branch review
- Submit dish reviews
- Share order page
- Reorder flow

#### Customer account
- Account overview page
- Addresses page
- Add address
- Edit address
- Delete address
- Set default address

### Major features to explain
- location-aware discovery
- refined search and cuisine browsing
- vendor and dish detail browsing
- basket management and restaurant conflict rules
- delivery quoting and checkout validation
- payment flow behavior
- order history, tracking, cancellation, and tipping
- review submission and order sharing
- saved address management

---

## Automation and Jobs Appendix Scope
This appendix should collect all major automated or background processes that support the websites.

### A. Scheduled jobs

#### Order Service Functions
- **ReconcileDailyStats**
  - schedule: daily at 00:05 UTC
  - purpose: reconcile prior-day operational statistics
  - impact: keeps reporting and daily aggregates accurate

- **ArchiveCompletedOrders**
  - schedule: daily at 02:30 UTC
  - purpose: archive older completed orders
  - impact: keeps primary order storage leaner and preserves history

#### Vendor Service background services
- **TierCalculationBackgroundService**
  - schedule: nightly at configured commission calculation time, defaulting to 02:00 UTC if no active config exists
  - purpose: recalculate vendor commission tiers using rolling completed-order counts
  - impact: updates vendor fee tiers, caches results, and publishes tier change events

- **RatingCalculationBackgroundService**
  - schedule: nightly at configured ratings calculation time
  - purpose: recalculate vendor, branch, and dish ratings
  - impact: keeps displayed ratings current after new reviews or moderation changes

#### Chat Service background services
- **StaleThreadCleanupService**
  - schedule: daily at configurable hour, default 02:00 UTC
  - purpose: clean up expired or orphaned order chat threads
  - impact: removes expired driver participants and closes stale threads

### B. Message-driven automation

#### File Service Functions
- **ProcessImage**
  - trigger: Service Bus message on `queue-image-processing`
  - purpose: process uploaded images asynchronously after upload
  - impact: supports content assets, menu images, and uploaded media workflows

#### Vendor Service background queue processing
- **ReindexBackgroundService**
  - trigger: admin reindex request queued through the vendor reindex channel
  - purpose: republish vendor data for search indexing in batches
  - impact: refreshes search visibility for vendors and dishes

- **CacheRebuildBackgroundService**
  - trigger: admin cache rebuild request queued through the cache rebuild channel
  - purpose: rebuild vendor/menu JSON caches in the background
  - impact: refreshes cached read models used by downstream experiences

### C. Admin-triggered operational automation
These are not always time-based, but they should be documented because administrators trigger them from the websites.

- reindex vendors from System Admin tools
- rebuild caches from System Admin tools
- import ONS postcode data from System Admin tools
- run vendor data integrity checks from System Admin tools
- trigger manual review rating recalculation from System Admin
- trigger manual commission tier recalculation from System Admin
- generate invoices for a single vendor or a monthly invoice run
- trigger manual daily stats backfill via admin HTTP function

### D. Startup automation
- **ElasticsearchInitializer**
  - trigger: Search Service startup
  - purpose: initialize search indexes with retry logic
  - impact: prepares search infrastructure before indexing and querying

### E. User-visible asynchronous workflows to mention in guides
These are not scheduled jobs, but they matter to end users and operators.

- vendor application approval sends invitation email for vendor onboarding
- driver invitation sends onboarding email for Driver Portal registration
- checkout can create payment intent details asynchronously after order submission
- image uploads complete processing asynchronously after the initial upload step
- route optimization runs as an explicit action from the Driver Portal dashboard

---

## Complex Processes Appendix Scope
This appendix should collect the major cross-service or multi-step platform processes that are too broad to fit cleanly inside a single website guide.

### A. Order Process Saga
- document the end-to-end order lifecycle from basket and checkout through vendor acceptance, preparation, dispatch, delivery, completion, cancellation, and post-order review prompts
- explain the systems and roles involved at each step, including Customer front-end, Vendor Admin, Driver Portal, background messaging, and admin/support intervention points
- note important state transitions, failure paths, retries, and operator-visible recovery behaviour

### B. Image Handling Process
- document how images move from upload initiation through validation, storage, queueing, background processing, resizing or transformation, and eventual availability in site experiences
- cover the main user-visible upload points, including content assets, hero images, support local slides, and menu content
- explain delayed availability, replacement behaviour, and likely failure or retry scenarios that operators need to understand

### C. Delivery Process
- document the delivery lifecycle from order acceptance through driver assignment, pickup readiness, route planning, live delivery execution, issue reporting, delivery completion, and failed delivery handling
- explain where Vendor Admin, Driver Portal, and Customer tracking views overlap or diverge
- include shift status, live location updates, delivery status transitions, and route optimisation as part of the operational flow

### D. Search Mechanism
- document how vendors and dishes become searchable, including indexing, cache rebuild dependencies, reindex triggers, and search startup preparation
- explain the relationship between the Search Service, reindex tooling, cache rebuild tooling, and customer-facing search or discovery experiences
- note what delays or stale-data symptoms operators may see after changes to vendors, menus, or content

### E. Other complex processes to capture
- vendor application approval and onboarding invitation flow
- driver invitation and onboarding handoff flow
- payment intent and checkout confirmation flow for card payments
- commission tier recalculation and invoice generation flow
- rating recalculation and flagged review resolution flow
- cache rebuild and vendor reindex operational recovery flow

---

## Proposed Documentation Structure

### Document 1: System Admin guide
Recommended sections:
1. Overview and permissions
2. Dashboard
3. Application review
4. Vendor and vendor user management
5. Orders, commissions, and invoices
6. Promotions and reviews
7. Content management
8. Taxonomy, safety, and settings
9. Payment setup
10. Tools and operational recovery actions

### Document 2: Vendor Admin guide
Recommended sections:
1. Overview and access model
2. Applying and onboarding
3. Dashboard and vendor context
4. Restaurant and branch management
5. Menu management
6. Orders workflow
7. Drivers and branch assignment
8. Offers and reviews
9. Delivery pricing and payment methods

### Document 3: Driver Portal guide
Recommended sections:
1. Overview and onboarding
2. Dashboard and shift controls
3. Deliveries workflow
4. Route planning
5. Availability management
6. Account management

### Document 4: Customer front-end guide
Recommended sections:
1. Discovering vendors and dishes
2. Basket behavior
3. Checkout and payment
4. Orders and tracking
5. Reviews, tips, and sharing
6. Account and addresses

### Document 5: Automation and jobs appendix
Recommended sections:
1. Scheduled jobs
2. Queue-driven processing
3. Admin-triggered background operations
4. Startup automation
5. User-visible async workflows
6. Operational ownership and impact notes

### Document 6: Complex processes appendix
Recommended sections:
1. Order Process Saga
2. Image handling process
3. Delivery process
4. Search and indexing mechanism
5. Payments and checkout async flow
6. Onboarding and invitation flows
7. Operational recovery and exception paths

---

## Execution Plan

### Phase 1 - Confirm inventory
- validate page names against navigation and view titles
- confirm whether any controller actions are utility-only and should be excluded from page-level docs
- confirm if any Razor Pages exist outside the controller-based flows already identified

#### Phase 1 findings

##### Cross-site findings
- All four websites are implemented primarily with MVC controllers and views.
- Razor Pages are enabled in all four web applications, but the only Razor Pages currently confirmed in scope are the Microsoft Identity account pages under `Areas/MicrosoftIdentity/Pages`.
- JSON endpoints, AJAX handlers, and partial views exist across every site. These support the documented user workflows, but they should remain implementation details unless a guide needs them to explain operator-visible behaviour.

##### System Admin validation
- Confirmed the main user-facing pages for dashboard, applications, vendors, vendor users, admin users, orders, commissions, invoices, offers, reviews, content assets, hero slides, regions, support local slides, cuisine types, bad words, settings, payment settings, and tools.
- Confirmed that several planned actions are implemented as in-page posts or JSON handlers rather than separate pages, including vendor activation/deactivation, vendor-user transfer, vendor-user deletion, commission tier create/update/delete, commission configuration save, commission recalculation trigger, invoice generation, invoice payment/cancel actions, offer activation/pause/delete, flagged review resolution, bad word add/edit/delete/import, payment provider save/remove actions, and tool execution actions.
- Confirmed `Invoices/List` and `Invoices/Detail` exist as separate views in addition to the uninvoiced dashboard and uninvoiced line items pages.
- Confirmed the hero slide preview page exists.
- Did not find a separate dedicated page for hero slide details or support local slide details; the edit views appear to be the primary detail surfaces.
- Confirmed postcode stats is exposed as a JSON/tool output rather than a standalone page view.

##### Vendor Admin validation
- Confirmed public onboarding and application pages for apply, apply success, onboarding landing, onboarding register, onboarding welcome, privacy, and home/error surfaces.
- Confirmed authenticated admin pages for dashboard, no-vendor-context state, restaurant management, menu builder list, menu editor, orders kanban, all orders, drivers, offers, reviews, delivery costs, and payment methods.
- Confirmed order details are primarily modal/partial and JSON-backed rather than separate full pages.
- Confirmed menu create/update/publish/status actions are workflow actions within the menu builder experience, supported by API endpoints rather than separate page views.
- Confirmed driver invitation resend, delete, branch assignment, and branch unassignment are action-based flows rather than dedicated pages.
- Confirmed promo code checks and several order refresh/stat endpoints are implementation details for the admin screens.

##### Driver Portal validation
- Confirmed public onboarding pages for landing, register, welcome, and onboarding error, plus a separate public home landing page.
- Confirmed authenticated driver pages for dashboard home, active deliveries, delivery dashboard, delivery detail, delivery history, availability, and account/profile.
- Confirmed shift start/end, shift status, live location update, delivery status changes, failed/delivered flows, revert/decline actions, and route optimisation actions are endpoint-driven behaviours rather than standalone pages.
- Confirmed the route planner controller entry point redirects back to the delivery dashboard, so route planning should be documented as part of the dashboard workflow rather than as a separate page.
- Confirmed dashboard card refresh, route stop refresh, and dashboard JSON responses are implementation details used by the live screens.

##### Customer front-end validation
- Confirmed user-facing pages for home, search, cuisine types, vendor, dish, checkout, orders history, order details, order confirmation, live tracking, order complete/review prompt, share order, account overview, addresses, and privacy.
- Confirmed nearby vendors is implemented as a partial flow from the home page rather than a separate full page.
- Confirmed delivery quote creation, payment intent polling, and reorder are endpoint-driven flows rather than dedicated pages.
- Confirmed address add, edit, delete, and set default are form actions within the addresses page rather than separate pages.
- Found a mismatch for basket documentation: a basket view exists, but `BasketController.Index()` currently redirects to the home page and the active basket experience is primarily implemented through the basket canvas/API flow. Document the basket as a shared basket experience unless a routed basket page is restored.

##### Phase 1 documentation decisions
- Treat modal views, partial views, JSON endpoints, and AJAX handlers as supporting implementation details unless they directly explain a user-visible workflow.
- Document route planning inside the Driver Portal delivery dashboard section, not as a standalone page.
- Document basket behaviour for the customer site as a shared basket flow/panel first, with a note that a standalone basket view exists in the codebase but is not currently the primary routed experience.

### Phase 2 - Draft user guides
- create one guide per website
- document each page with purpose, access requirements, key actions, and expected outcomes
- group related pages into end-to-end workflows

### Phase 3 - Draft automation appendix
- convert the automation inventory into an operator-friendly summary
- record trigger type, schedule, owning service, and user impact
- distinguish scheduled jobs from on-demand admin operations

#### Phase 3 outcomes
- Created `docs/documentation/automation-and-jobs.md` as the shared appendix for scheduled jobs, queue-driven processing, startup automation, admin-triggered operations, and user-visible asynchronous workflows.
- Captured each major automation item with trigger type, schedule or invocation pattern, owning service, operator-visible impact, and where relevant the primary website or admin surface that initiates or depends on it.
- Documented that several operational actions exposed in System Admin are synchronous trigger points for longer-running background work, and should be described as operator workflows rather than standalone pages.
- Confirmed that checkout payment intent retrieval, image processing, vendor onboarding invitations, driver invitations, and route optimisation should be referenced in the website guides as user-visible asynchronous behaviour rather than as separate technical pages.

### Phase 4 - Draft complex processes appendix
- create `docs/documentation/complex-processes.md`
- document the Order Process Saga, Image Handling process, Delivery process, and Search mechanism as cross-service workflows
- capture other complex operational flows that span multiple websites or background components
- describe participants, trigger points, major state transitions, exception paths, and operator-visible outcomes
- link each complex process back to the relevant website guides and to `automation-and-jobs.md` where background work is involved

#### Phase 4 outcomes
- Created `docs/documentation/complex-processes.md` as the shared appendix for cross-site workflows, sagas, onboarding handoffs, payment timing, and operational recovery paths.
- Documented the Order Process Saga, Image handling process, Delivery process, and Search and indexing mechanism using business-facing workflow narratives that explain participants, trigger points, major states, and exception behaviour.
- Added dedicated sections for the card payment and checkout async flow, the vendor approval and onboarding invitation flow, the driver invitation and Driver Portal onboarding handoff, and the operational recovery flows for commissions, invoicing, ratings, caches, and search reindexing.
- Cross-linked the appendix back to the four website guides and to `automation-and-jobs.md` so page-level documentation can reference the broader process without repeating implementation-heavy detail.

#### Phase 4 target topics
- Order Process Saga
- Image Handling process
- Delivery process
- Search and indexing mechanism
- Vendor application approval and onboarding flow
- Driver invitation and onboarding flow
- Payment intent and checkout confirmation flow
- Commission, invoicing, and rating maintenance flows

### Phase 5 - Review and harden
- review naming consistency across all guides
- add screenshots, navigation paths, and prerequisite notes
- verify documentation against latest routes and menu structure

#### Phase 5 outcomes
- Populated `docs/documentation/index.md` as the landing page for the documentation set, including cross-site navigation, shared prerequisites, and a first-release screenshot checklist.
- Hardened each website guide with shared prerequisite notes, route-safe navigation paths, and direct links back to `automation-and-jobs.md` for delayed or background behaviour.
- Replaced outdated wording that said the automation appendix was still planned, so the guides now reference the completed appendix consistently.
- Added screenshot candidate lists to each guide so the first documentation release has a defined capture scope without blocking the written guidance.
- Rechecked the public route entry points and workflow starts against the current controller structure, including the shared basket redirect behaviour and the onboarding flows for Vendor Admin and Driver Portal.

---

## Open Questions to Resolve During Authoring
- Which pages should include screenshots in the first release?
- Should admin-only AJAX-backed flows be documented inline or in a technical appendix?
- Should the complex processes appendix include sequence diagrams or stay narrative-only in the first release?
- Should payment provider setup steps include environment-specific guidance?
- Is there a preferred owner for keeping the automation appendix up to date?
- Is there a preferred owner for keeping the complex processes appendix up to date?
- Do we want a single role/permissions matrix shared across all guides?

---

## Recommended Next Step
Start by creating `index.md` and `system-admin-guide.md`, because the System Admin area contains the broadest operational surface area and references several automation features that the other guides will link back to.