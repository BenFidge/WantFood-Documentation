# WantFood Complex Processes Appendix

This appendix explains the major cross-site workflows that span more than one website, service, or background process. It is intended for operators, support staff, and documentation authors who need a narrative view of how the platform behaves across handoffs, delayed work, and recovery scenarios.

## How to use this appendix

- Use this document when a workflow starts in one website and finishes in another.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for schedule, trigger, and ownership details behind the background work mentioned here.
- Use the site guides for page-level instructions and return here when a guide needs the larger end-to-end story.
- Treat this appendix as a business-facing description of the process, with only enough implementation detail to explain visible behaviour and operator actions.

## Guide cross-reference map

| Process area | Primary guide links |
| --- | --- |
| Order lifecycle and post-order behaviour | [Customer Front-End Guide](customer-front-end-guide.md), [Vendor Admin Guide](vendor-admin-guide.md), [Driver Portal Guide](driver-portal-guide.md) |
| Image uploads and delayed media availability | [System Admin Guide](system-admin-guide.md), [Vendor Admin Guide](vendor-admin-guide.md) |
| Delivery execution and route handling | [Vendor Admin Guide](vendor-admin-guide.md), [Driver Portal Guide](driver-portal-guide.md), [Customer Front-End Guide](customer-front-end-guide.md) |
| Search freshness and recovery actions | [Customer Front-End Guide](customer-front-end-guide.md), [System Admin Guide](system-admin-guide.md), [Vendor Admin Guide](vendor-admin-guide.md) |
| Onboarding invitations and handoffs | [System Admin Guide](system-admin-guide.md), [Vendor Admin Guide](vendor-admin-guide.md), [Driver Portal Guide](driver-portal-guide.md) |
| Payments, commissions, invoicing, and rating maintenance | [Customer Front-End Guide](customer-front-end-guide.md), [System Admin Guide](system-admin-guide.md) |

---

## 1. Order Process Saga

The order process saga is the main cross-platform workflow in WantFood. It begins when a customer commits a basket through checkout and continues through vendor handling, delivery execution, completion, cancellation, and post-order follow-up.

### Main participants

| Participant | Role in the process |
| --- | --- |
| Customer front-end | Creates the basket, completes checkout, views confirmation, tracks progress, and later leaves reviews or tips. |
| Vendor Admin | Accepts, rejects, edits, prepares, and dispatches the order. |
| Driver Portal | Handles the delivery run once a driver is assigned. |
| Order Service and related messaging | Coordinates state transitions, notifications, payment handoffs, and delivery requests. |
| System Admin | Investigates exceptions, supports finance, reviews, and operational recovery when the standard flow breaks down. |

### Trigger point

The saga starts when the customer places an order from the checkout page. From that point onward, the order moves through a controlled set of operational states rather than being treated as a single synchronous action.

### End-to-end lifecycle

1. **Basket and checkout preparation**  
   The customer builds a basket from a single vendor, reviews delivery details, receives a delivery quote, and submits the order.

2. **Order creation and payment branching**  
   The order is accepted into the platform. Cash orders can move directly into fulfilment. Card orders may continue through asynchronous payment setup before the customer reaches a stable confirmation outcome.

3. **Vendor decision**  
   Vendor Admin reviews the incoming order from the kanban workflow. The vendor can accept the order, reject it, edit it where the workflow permits, or cancel it later if fulfilment becomes impossible.

4. **Preparation**  
   Once accepted, the order progresses through preparation states until it is ready for pickup. The customer-facing order detail and tracking views reflect these changes as the order advances.

5. **Driver request and assignment**  
   When the order becomes ready for pickup, the delivery side is engaged. A driver request is created, the platform attempts assignment, and the result determines whether delivery continues normally or moves into an exception path.

6. **Pickup and live delivery**  
   After a driver is assigned, the run progresses through driver-en-route, pickup, in-transit, and arrival milestones. Driver Portal becomes the primary operational surface for the driver, while Vendor Admin and customer tracking continue to show the latest visible state.

7. **Delivery completion**  
   The driver marks the order delivered, the live operational flow closes, and the order enters its completed state.

8. **Cancellation and failure branches**  
   The order can leave the happy path because of vendor rejection, vendor cancellation, driver unavailability, payment problems, or delivery failure. Depending on timing and payment state, the process may void a payment, trigger a refund path, or end without capture.

9. **Post-order follow-up**  
   After completion, the customer can review the order, add tips where supported, view order history, share the order, or reorder later. Ratings and reporting updates may appear after supporting background work completes.

### Major state transitions and visible surfaces

| Broad state | Main operator surface | Customer-visible effect |
| --- | --- | --- |
| Basket | Customer front-end | Customer is still editing the intended order. |
| Submitted or awaiting payment completion | Customer front-end | Order has been placed, but card confirmation may still be completing. |
| Accepted or rejected | Vendor Admin | Customer sees whether the restaurant will fulfil the order. |
| Preparing | Vendor Admin | Customer sees active progress before pickup. |
| Ready for pickup | Vendor Admin | Delivery preparation starts and assignment pressure becomes visible operationally. |
| Driver assigned and en route | Driver Portal + Vendor Admin | Customer tracking moves from kitchen status to active delivery progress. |
| Picked up and out for delivery | Driver Portal | Customer sees the order on its way. |
| Delivered | Driver Portal | Customer sees final successful completion. |
| Cancelled, failed, or refunded | Varies by cause | Customer sees that the order did not complete normally and may later see payment correction outcomes. |

### Failure paths and recovery behaviour

- **Vendor rejects before fulfilment**: the order ends early and the customer is informed that fulfilment will not continue.
- **Vendor cancels after acceptance**: the order leaves the normal fulfilment path and payment recovery may be required.
- **No driver available**: a ready-for-pickup order can fail its delivery assignment window. If payment was already captured, the payment recovery branch becomes important.
- **Driver status correction**: Driver Portal supports status reversals when a driver applies the wrong state. This is an operational correction path, not a separate delivery type.
- **Card payment timing issues**: the customer may have placed the order successfully but still need to wait for the payment setup path to complete.
- **Support intervention**: System Admin does not normally drive the order saga, but it becomes relevant for invoice, commission, moderation, or recovery work after exceptions occur.

### Documentation guidance

- Use the [Customer Front-End Guide](customer-front-end-guide.md) for basket, checkout, confirmation, tracking, and post-order pages.
- Use the [Vendor Admin Guide](vendor-admin-guide.md) for acceptance, preparation, and dispatch actions.
- Use the [Driver Portal Guide](driver-portal-guide.md) for assigned delivery execution.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when documenting delayed payment setup, rating refreshes, or other background effects that appear after the main user action.

---

## 2. Image handling process

The image handling process explains how an uploaded image becomes a reusable platform asset. This matters because users often complete the visible upload step before the image is available everywhere it will eventually appear.

### Main participants

| Participant | Role in the process |
| --- | --- |
| System Admin | Uploads and maintains content assets, hero slides, support campaigns, and taxonomy-related media. |
| Vendor Admin | Uploads vendor and menu-related images, including dish content. |
| File Service | Accepts image work, processes variants, and publishes the final image output. |
| Downstream sites | Display the processed image after the background work finishes. |

### Main upload entry points

- content asset library in System Admin
- hero slide editing in System Admin
- support-local and similar campaign imagery in System Admin
- cuisine-related imagery where supported by admin flows
- vendor logos, vendor covers, and dish or menu imagery in Vendor Admin

### End-to-end lifecycle

1. **Upload initiation**  
   An operator chooses an image from an admin workflow and submits it.

2. **Initial acceptance**  
   The upload request is accepted quickly so the user can continue the current form or editing workflow.

3. **Queueing for processing**  
   The platform records what kind of image was uploaded and sends the work to the background image processing path.

4. **Background processing**  
   The image is downloaded from temporary storage, transformed into the required output sizes or variants, and written into its final storage layout.

5. **Result publication**  
   The platform publishes the processed image details so the relevant content record can point at the new image variant set.

6. **Eventual availability in websites**  
   The processed image becomes visible in the relevant admin preview or customer-facing experience after the background work and any dependent refresh behaviour complete.

### What users experience

| Stage | What the user typically sees |
| --- | --- |
| Immediately after upload | The form or edit action can complete successfully before the final image is visible. |
| During processing | The old image may remain visible temporarily, or the new image may not yet appear in previews and live pages. |
| After processing completes | The new image variants are available to the relevant page, tile, hero area, or menu item. |

### Replacement and exception behaviour

- A new upload usually replaces the prior visible image only after processing completes successfully.
- Different image contexts can produce different default output variants, so a hero image and a dish image may not appear in exactly the same way.
- If processing fails, the upload may have been accepted but the image will not become fully available until the issue is retried or corrected.
- Operators should expect a short delay rather than instant propagation, especially when testing just after an upload.

### Documentation guidance

- Use the [System Admin Guide](system-admin-guide.md) for operator-facing upload and content management steps.
- Use the [Vendor Admin Guide](vendor-admin-guide.md) for menu and vendor image workflows.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the background processing trigger and ownership details behind `ProcessImage`.

---

## 3. Delivery process

The delivery process covers the part of the order lifecycle that begins after fulfilment is accepted and continues through assignment, route handling, live execution, issue reporting, and delivery completion or failure.

### Main participants

| Participant | Role in the process |
| --- | --- |
| Vendor Admin | Marks the order ready, assigns or unassigns drivers where supported, and monitors dispatch progress. |
| Driver Portal | Manages shifts, active deliveries, route optimisation, live status changes, and issue reporting. |
| Customer front-end | Shows tracking and delivery status from the customer perspective. |
| Delivery Service | Stores assignment, route, and delivery status information. |

### Trigger point

The delivery process begins when an accepted order reaches the stage where pickup and dispatch work can start, usually when the vendor marks it ready for pickup.

### End-to-end lifecycle

1. **Dispatch readiness**  
   Vendor Admin moves the order from kitchen handling into pickup readiness.

2. **Driver assignment**  
   A driver is assigned or a request is made for assignment. This is the key handoff from restaurant fulfilment into delivery execution.

3. **Shift readiness and live location**  
   The driver must be in an active shift state for normal delivery handling. Live location updates help support dispatch visibility and customer tracking.

4. **Driver route setup**  
   The driver opens the delivery dashboard, reviews current stops, and can request route optimisation. Route planning is part of the dashboard workflow rather than a separate full page.

5. **Pickup execution**  
   The driver travels to the restaurant, confirms pickup-related milestones, and moves the delivery into the outbound stage.

6. **In-transit updates**  
   Delivery status changes continue through the route. Customer tracking and vendor-side monitoring reflect the same underlying delivery progress, but each site shows it in a role-appropriate way.

7. **Completion or issue handling**  
   The driver either marks the delivery as delivered or records a failed delivery or issue. When necessary, the driver can revert a mistaken status.

### Cross-site overlap and divergence

| Area | Vendor Admin view | Driver Portal view | Customer view |
| --- | --- | --- | --- |
| Before assignment | Sees order preparation and readiness | Usually not yet involved | Sees restaurant progress rather than driver detail |
| After assignment | Sees dispatch-linked order state | Sees delivery workload and route details | Sees active tracking begin to matter more |
| During the run | Monitors progress and exceptions | Primary working surface for status changes and route handling | Sees live order tracking and delivery progress |
| Delivery issue | Can respond operationally if the run goes off plan | Records the issue or failed state | Sees delayed, failed, or exceptional delivery outcomes |

### Route optimisation behaviour

- Route optimisation is an explicit action initiated by the driver from the delivery dashboard.
- The platform can clear any previously stored active route before writing a new route batch.
- Optimised route data is returned into the dashboard experience rather than opening a separate route-planning page.
- Clearing the route removes the current optimisation so the driver can work from the unoptimised delivery set again.

### Exception and recovery paths

- **No driver available**: the order can leave the normal delivery path and enter a delivery failure branch.
- **Driver cancels or is unassigned**: the order can return to a state where another assignment attempt is needed.
- **Status applied incorrectly**: Driver Portal supports status reversal to correct operational mistakes.
- **Route no longer matches reality**: the driver can clear and regenerate an optimised route when active deliveries change.
- **Location problems**: live tracking quality depends on current location updates reaching the platform.

### Documentation guidance

- Use the [Vendor Admin Guide](vendor-admin-guide.md) for order readiness, driver assignment, and vendor-side monitoring.
- Use the [Driver Portal Guide](driver-portal-guide.md) for shift, route, and delivery execution steps.
- Use the [Customer Front-End Guide](customer-front-end-guide.md) for tracking and delivery completion visibility.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) when explaining route optimisation as a user-visible asynchronous action.

---

## 4. Search and indexing mechanism

The search mechanism explains how vendor and dish information becomes searchable and why some changes may not appear instantly in customer discovery experiences.

### Main participants

| Participant | Role in the process |
| --- | --- |
| Vendor Admin | Creates and publishes menu and profile changes that eventually affect search visibility. |
| Customer front-end | Uses search, cuisine browsing, and vendor discovery experiences that depend on current search data. |
| Search Service | Owns index initialization and query behaviour. |
| Vendor Service | Rebuilds caches and republishes vendor data for reindexing. |
| System Admin | Triggers recovery actions when search freshness needs intervention. |

### Main process

1. **Source data changes**  
   Vendor or content changes occur through menu management, restaurant updates, offer changes, or related data maintenance.

2. **Read-model and cache dependency**  
   Customer-facing experiences depend on prepared read models and caches as well as the search index itself. This means search freshness is not only about the index; it also depends on upstream data being in a healthy state.

3. **Search startup preparation**  
   When the Search Service starts, index initialization runs with retry logic. Search behaviour can remain incomplete until this startup preparation succeeds.

4. **Reindex publication**  
   When a full recovery is needed, System Admin can trigger a vendor reindex. The request is accepted immediately, then processed in the background in batches.

5. **Cache rebuild support**  
   System Admin can also trigger a cache rebuild. This refreshes the cached vendor and menu data used by downstream experiences.

6. **Customer discovery**  
   Once the underlying data, caches, and search indexes are aligned, the customer-facing search and discovery pages reflect the latest changes.

### Operator-visible symptoms of stale search data

- a vendor or dish was updated but does not appear in search results yet
- menu or vendor detail seems current in one place but old in another
- a deployment or restart left search behaving as though indexes are incomplete
- content changes were accepted, but customer-facing discovery still reflects older cached data

### Recovery actions and when to use them

| Recovery action | When it helps | Expected result |
| --- | --- | --- |
| Reindex vendors | Search visibility is stale after major vendor or menu changes, or after index reset activity | Vendor data is republished to search in the background. |
| Rebuild caches | Menu or vendor read models appear stale or inconsistent | Cached menu and vendor data is refreshed in the background. |
| Search startup review | Search issues appear after deployment, restart, or environment recovery | Confirms whether startup index initialization completed successfully. |

### Documentation guidance

- Use the [Customer Front-End Guide](customer-front-end-guide.md) for the customer-facing search and discovery pages.
- Use the [Vendor Admin Guide](vendor-admin-guide.md) for the source workflows that ultimately feed discovery.
- Use the [System Admin Guide](system-admin-guide.md) for the reindex and cache rebuild tool surfaces.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for queue-driven reindexing, cache rebuild behaviour, and search startup automation details.

---

## 5. Payments and checkout async flow

The payment and checkout async flow explains why card orders can move through acceptance and confirmation in stages rather than as one immediate response.

### Main participants

| Participant | Role in the process |
| --- | --- |
| Customer front-end | Collects the checkout data, submits the order, polls for payment readiness, and completes confirmation. |
| Order and payment services | Create the order, start payment work, and publish the payment state the website needs. |
| Vendor Admin | Receives the order once it is accepted into the fulfilment pipeline. |

### Main process

1. **Customer submits checkout**  
   The customer confirms delivery details, totals, and payment choice from the checkout page.

2. **Order accepted into the platform**  
   The order is created and given an order identifier.

3. **Payment branch by method**  
   Cash orders can continue without card setup. Card orders require payment intent information before the payment element can finish confirmation.

4. **Asynchronous payment intent availability**  
   The checkout page may not receive the client secret immediately. In that case, the front-end polls the payment-intent endpoint until the data becomes available or the wait times out.

5. **Customer confirmation step**  
   Once the payment intent is ready, the customer-side payment element completes the card confirmation flow.

6. **Order confirmation**  
   The customer is redirected into the order confirmation experience when the platform has enough information to continue.

### What the customer sees

| Situation | Customer-facing behaviour |
| --- | --- |
| Cash order | The order can progress straight into confirmation without card setup. |
| Card order with prompt payment readiness | Checkout transitions smoothly into confirmation with little visible delay. |
| Card order with delayed payment readiness | The customer waits while the page polls for payment setup. |
| Payment setup timeout or failure | The customer sees an error and may need to retry the payment attempt or restart checkout. |

### Exception and recovery notes

- A placed order is not the same as a fully confirmed card payment at the exact same moment.
- The payment setup delay is expected platform behaviour, not automatically a fault condition.
- If a later fulfilment failure occurs after capture, refund or void handling becomes part of the downstream order exception path.
- Documentation should explain staged confirmation without exposing unnecessary gateway implementation detail.

### Documentation guidance

- Use the [Customer Front-End Guide](customer-front-end-guide.md) for checkout, payment, order confirmation, and tracking pages.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the user-visible asynchronous payment note.
- Refer back to the order saga section in this appendix when payment outcomes need to be understood in the wider fulfilment lifecycle.

---

## 6. Onboarding and invitation flows

WantFood has two major invitation-driven onboarding flows: vendor onboarding after application approval and driver onboarding after invitation by a vendor.

### 6.1 Vendor application approval and onboarding flow

#### Main participants

| Participant | Role in the process |
| --- | --- |
| System Admin | Reviews the application and decides whether to approve or reject it. |
| Vendor Admin public onboarding pages | Accept the approved user into the onboarding and registration flow. |
| Notification path | Sends the approval email containing the onboarding link. |
| User and vendor account services | Persist invitation state, completion state, and user role assignment. |

#### Main process

1. A prospective vendor submits an application from the public apply page.
2. System Admin reviews the application from the application list and detail workflow.
3. If approved, the application is marked approved, an invitation token is created, and an expiry window is set.
4. An approval email is sent with the Vendor Admin onboarding link.
5. The invited user opens the onboarding landing page and completes registration.
6. Invitation completion records that onboarding was accepted and can also attach the created vendor record and vendor role.
7. The user reaches the onboarding welcome page and then moves into the normal Vendor Admin dashboard flow.

#### Important exception paths

- If email delivery fails, approval can still succeed because the invitation token is already stored.
- If onboarding is repeated or retried, invitation completion is designed to be tolerant of idempotent re-entry.
- If an application is rejected, the process ends without issuing onboarding access.

### 6.2 Driver invitation and onboarding handoff flow

#### Main participants

| Participant | Role in the process |
| --- | --- |
| Vendor Admin | Creates the driver record and sends the invitation. |
| Notification path | Sends the driver onboarding email. |
| Driver Portal public onboarding pages | Complete the registration and welcome flow. |
| Delivery-side account record | Stores the invited driver and later supports active working access. |

#### Main process

1. A vendor user opens the driver management flow in Vendor Admin.
2. The vendor invites a driver by providing the required identity and contact details.
3. The platform creates the driver record in a pending invitation state.
4. An invitation email is queued and sent with the Driver Portal onboarding link.
5. The invited driver opens the Driver Portal onboarding flow, registers, completes the welcome page, and signs in.
6. The working relationship then continues in the authenticated Driver Portal experience.

#### Important exception paths

- The driver record can be created even if the invitation email fails, so resend and support recovery remain important operator actions.
- Vendor Admin is the start of the invitation flow, but Driver Portal is where onboarding is actually completed.
- Documentation should make the handoff explicit so operators understand why the process starts in one site and finishes in another.

### Documentation guidance

- Use the [System Admin Guide](system-admin-guide.md) for application review and approval.
- Use the [Vendor Admin Guide](vendor-admin-guide.md) for vendor onboarding entry points and vendor-side driver invitation flows.
- Use the [Driver Portal Guide](driver-portal-guide.md) for driver onboarding completion.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for the user-visible invitation email behaviour and related asynchronous notes.

---

## 7. Operational recovery and exception paths

This section groups the operational processes that are broader than one page and often combine manual action with scheduled or background work.

### 7.1 Commission tier recalculation and invoice generation flow

Commission and invoicing are linked but not identical. Commission tiers affect vendor charging logic, while invoicing turns invoiceable commission line items into finance-facing records.

#### Main process

1. Vendor activity generates completed-order history and commission line items.
2. Commission tier calculations run on schedule to refresh vendor fee tier assignments based on rolling order counts.
3. System Admin can trigger a manual recalculation when configuration changes or urgent correction work is needed.
4. Uninvoiced commission items accumulate until finance staff generate a single-vendor invoice or a monthly invoice batch.
5. Generated invoices move commission line items from pending to invoiced.
6. Finance operations can later mark invoices paid or cancel them, with cancellation returning the line items to a pending state.

#### Operator-visible outcomes

- tier assignments can update overnight without a manual action
- manual recalculation is the recovery path when operators need refreshed tier data immediately
- invoice generation creates a finance record from existing commission items rather than recalculating the original order
- invoice cancellation restores the affected commission items so they can be invoiced again correctly

### 7.2 Rating recalculation and flagged review resolution flow

Ratings are visible to users, but the maintenance path spans review moderation and background recalculation.

#### Main process

1. Reviews are submitted through customer-facing flows.
2. Vendor-side users can flag problematic reviews for escalation.
3. System Admin reviews flagged items and either upholds the flag or dismisses it.
4. Rating recalculation runs on schedule to refresh vendor, branch, and dish ratings.
5. System Admin can also trigger immediate recalculation after moderation or support actions when an overnight wait is not acceptable.

#### Operator-visible outcomes

- resolving a flagged review does not always mean the displayed rating changes immediately
- manual recalculation is the faster correction path after moderation work
- overnight recalculation remains the default maintenance path for steady-state rating freshness

### 7.3 Cache rebuild and vendor reindex recovery flow

These recovery tools are used when search or read-model freshness has drifted from the current source data.

#### Main process

1. An operator identifies stale or inconsistent vendor, menu, or search behaviour.
2. System Admin triggers a cache rebuild, a vendor reindex, or both, depending on the symptom.
3. The platform accepts the request immediately and queues the substantive work.
4. Vendor Service background workers process the request and refresh the affected downstream data.
5. Customer-facing discovery and admin-side read models gradually return to a current state.

#### Operator-visible outcomes

- a successful trigger message means the recovery job was accepted, not that all data is already fresh
- cache rebuild and reindex actions may be paired during larger recovery scenarios
- search recovery should also consider whether startup index initialization completed successfully after restart or deployment

### Documentation guidance

- Use the [System Admin Guide](system-admin-guide.md) for the operator-facing tools, reviews, commissions, and invoicing surfaces.
- Use the [Automation and Jobs Appendix](automation-and-jobs.md) for exact schedules, trigger types, and ownership information.
- Link back to the earlier sections of this appendix when the recovery action relates to search freshness, ratings, or order exceptions.

---

## Maintenance note

When a workflow changes across more than one site, update this appendix alongside the relevant page-level guides. This appendix should stay focused on cross-site behaviour, handoffs, exception paths, and operator-visible outcomes rather than on controller or endpoint inventories.