# WantFood Automation and Jobs Appendix

This appendix summarises the major automated processes that support the WantFood websites. It is written for operators, administrators, and authors who need to understand what runs automatically, what can be triggered manually, and what user-facing behaviour depends on background processing.

## How to use this appendix

- Use the scheduled jobs section to understand routine overnight or daily maintenance.
- Use the queue-driven processing section to understand work that is accepted immediately but completed in the background.
- Use the admin-triggered operations section when documenting tools and maintenance actions available in System Admin.
- Use the user-visible async workflows section when explaining why some actions complete in stages from an end-user perspective.
- Use the [documentation index](index.md) to move back to the site guides when you need the workflow-level view.

## Guide cross-reference map

| Guide | Most relevant appendix sections |
| --- | --- |
| [System Admin Guide](system-admin-guide.md) | Scheduled jobs, admin-triggered background operations, startup automation, operational impact guidance |
| [Vendor Admin Guide](vendor-admin-guide.md) | Queue-driven processing, user-visible asynchronous workflows, operational impact guidance |
| [Driver Portal Guide](driver-portal-guide.md) | User-visible asynchronous workflows, queue-driven processing where route and refresh behaviour matter |
| [Customer Front-End Guide](customer-front-end-guide.md) | User-visible asynchronous workflows, scheduled jobs that affect rating or reporting freshness |

---

## 1. Scheduled jobs

These jobs run automatically on a timer or internal schedule without an operator having to trigger them from a website.

### Order Service Functions

| Automation | Trigger type | Schedule | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- | --- |
| `ReconcileDailyStats` | Azure Function timer | Daily at 00:05 UTC | Order Service Functions | Reconciles prior-day operational statistics. | Keeps daily reporting and aggregate figures aligned with the latest completed order data. |
| `ArchiveCompletedOrders` | Azure Function timer | Daily at 02:30 UTC | Order Service Functions | Archives older completed orders. | Keeps active order storage leaner while preserving historical records for reporting and lookup. |

### Vendor Service background services

| Automation | Trigger type | Schedule | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- | --- |
| `TierCalculationBackgroundService` | Hosted background service | Nightly at the configured commission calculation time, default 02:00 UTC if no active config exists | Vendor Service | Recalculates vendor commission tiers from rolling completed-order counts. | Updates vendor fee tiers, refreshes cached commission data, and publishes tier change events used by downstream processes. |
| `RatingCalculationBackgroundService` | Hosted background service | Nightly at the configured ratings calculation time, default 02:00 UTC | Vendor Service | Recalculates vendor, branch, and dish ratings. | Keeps displayed ratings current after new reviews, moderation decisions, or score-affecting changes. |

### Chat Service background services

| Automation | Trigger type | Schedule | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- | --- |
| `StaleThreadCleanupService` | Hosted background service | Daily at a configurable hour, default 02:00 UTC | Chat Service | Cleans up expired or orphaned order chat threads. | Removes expired driver participants, closes stale conversations, and prevents obsolete chat sessions from remaining open. |

### Scheduled job notes

- The two Order Service timers are explicit Azure Functions schedules and are suitable for operational runbooks.
- The Vendor Service and Chat Service jobs are service-hosted schedulers, so their exact run time depends on configuration and the service being healthy.
- When user guides mention delayed rating updates, commission refreshes, or chat closure after delivery, they should link back to this appendix rather than re-explaining the underlying technical implementation.

---

## 2. Queue-driven processing

These processes are message-driven. A user or operator action may complete quickly on the website, while the substantive work continues in the background.

### File Service Functions

| Automation | Trigger type | Invocation pattern | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- | --- |
| `ProcessImage` | Azure Function with Service Bus trigger | Runs when a message is received on `queue-image-processing` | File Service Functions | Processes uploaded images into the required output variants and publishes the resulting image information. | Supports content asset uploads, menu images, hero slides, cuisine type images, and similar media workflows where the upload step completes before processing finishes. |

### Vendor Service background queue processing

| Automation | Trigger type | Invocation pattern | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- | --- |
| `ReindexBackgroundService` | Hosted background service reading an internal channel | Runs when an admin reindex request is queued | Vendor Service | Republishes vendor data for search indexing in batches. | Refreshes vendor and dish visibility in search without blocking the admin tool request. |
| `CacheRebuildBackgroundService` | Hosted background service reading an internal channel | Runs when an admin cache rebuild request is queued | Vendor Service | Rebuilds vendor and menu JSON caches in the background. | Refreshes cached read models used by downstream website experiences and integration surfaces. |

### Queue-driven processing notes

- Queue-driven or channel-driven processing should be documented as asynchronous workflows, not as standalone pages.
- Operators should expect these actions to begin immediately but complete after background workers consume the request.
- If an admin screen reports that a reindex or cache rebuild was triggered successfully, that message confirms acceptance of the request rather than instant completion of all downstream work.

---

## 3. Admin-triggered background operations

These actions are initiated from System Admin or other admin-only surfaces. Some complete in-process, while others hand work off to background services or functions.

| Operation | Primary entry point | Trigger type | Owning service or surface | Purpose | Operator impact |
| --- | --- | --- | --- | --- | --- |
| Reindex vendors | System Admin tools | Admin-triggered background request | System Admin + Vendor Service | Queues a reindex of vendor data for search. | Used after large data updates, search issues, or recovery activity affecting vendor discoverability. |
| Rebuild caches | System Admin tools | Admin-triggered background request | System Admin + Vendor Service | Queues a rebuild of cached vendor and menu read models. | Used when cached menu or vendor data needs to be refreshed outside the normal request flow. |
| Import ONS postcode data | System Admin tools | Admin-triggered tool action | System Admin | Imports or updates postcode reference data. | Supports postcode-dependent operational tools and geographic lookups used across the platform. |
| Run vendor data integrity checks | System Admin tools | Admin-triggered tool action | System Admin | Runs integrity checks and may apply fixes. | Helps operations diagnose and remediate vendor data problems without manual database work. |
| Trigger review rating recalculation | System Admin reviews | Admin-triggered request | System Admin + review/rating pipeline | Forces rating recalculation after moderation or support work. | Used when operators need ratings refreshed immediately instead of waiting for the next scheduled run. |
| Trigger commission tier recalculation | System Admin commission assignments | Admin-triggered background request | System Admin + Vendor Service | Forces commission tier recalculation outside the nightly schedule. | Used after tier configuration changes or when support staff need updated commission assignments quickly. |
| Generate single-vendor invoice | System Admin invoices | Admin-triggered workflow | System Admin + Order Service invoicing | Generates an invoice for one vendor. | Supports exception handling and targeted billing activity. |
| Generate monthly invoices | System Admin invoices | Admin-triggered workflow | System Admin + Order Service invoicing | Generates a monthly invoice batch. | Supports the regular finance process for vendor billing cycles. |
| Mark invoice paid | System Admin invoices | Admin-triggered workflow | System Admin + Order Service invoicing | Updates an invoice to paid status. | Keeps finance-facing invoice status aligned with real payment receipt. |
| Cancel invoice | System Admin invoices | Admin-triggered workflow | System Admin + Order Service invoicing | Cancels an invoice. | Supports billing correction and exception handling workflows. |
| Backfill daily stats | Admin HTTP function | Admin-triggered HTTP function | Order Service Functions | Reconciles a specified historical date range. | Supports recovery when daily statistics need to be rebuilt for one or more past dates. |

### Admin-triggered operation notes

- The System Admin UI should describe these as maintenance or recovery actions, not as ordinary end-user pages.
- Where a tool returns a success message immediately, documentation should clarify whether that means the work finished or was merely accepted for background processing.
- The manual rating recalculation and commission recalculation actions are particularly important when operators need urgent data correction outside the normal overnight schedule.

### Manual nightly job triggers (Phase 3)

System Admin -> Tools includes a **Nightly Jobs** section that exposes a "Run now" button for every nightly job. Each button calls `POST /api/admin/jobs/{job}` on the BFF, which forwards to the owning service and publishes the corresponding MassTransit command. The service responds with `202 Accepted` and a JSON payload containing a `correlationId`; the UI surfaces this in the success banner ("Job queued (correlation id: ...)").

| Job | BFF route | MassTransit command | Notes |
| --- | --- | --- | --- |
| Reconcile daily stats | `POST /api/admin/jobs/reconcile-daily-stats?date=yyyy-MM-dd` | `ReconcileDailyStatsCommand` | Date is optional and defaults to yesterday (UTC). Idempotent on `(date, correlationId)`. |
| Archive completed orders | `POST /api/admin/jobs/archive-orders` | `ArchiveCompletedOrdersCommand` | Skips orders already archived. |
| Recalculate vendor tiers | `POST /api/admin/jobs/recalculate-tiers` | `RecalculateVendorTiersCommand` | Suppresses duplicate `VendorTierChanged` events. |
| Recalculate ratings | `POST /api/admin/jobs/recalculate-ratings` | `RecalculateRatingsCommand` | Recomputes only deltas. |
| Cleanup stale chat threads | `POST /api/admin/jobs/cleanup-stale-chat-threads` | `CleanupStaleChatThreadsCommand` | One-shot per thread; safe to re-run. |
| Reindex vendors (job) | `POST /api/admin/jobs/reindex-vendors` | `ReindexVendorsCommand` | Same effect as the legacy "Reindex vendors" tool button but via the job pipeline. |
| Rebuild vendor caches (job) | `POST /api/admin/jobs/rebuild-vendor-caches` | `RebuildVendorCachesCommand` | Same effect as the legacy "Rebuild caches" tool button but via the job pipeline. |

All endpoints require the `PlatformAdmin` policy. Manual and scheduled invocations publish the same command, so consumer-side dedupe (via `JobRun` and the MassTransit outbox) prevents overlapping or duplicated work. Manual runs do not shift the next scheduled time.

---

## 4. Startup automation

Some automation runs when a service starts rather than on a time-based or user-initiated trigger.

| Automation | Trigger type | Owning service | Purpose | User or operator impact |
| --- | --- | --- | --- | --- |
| `ElasticsearchInitializer` | Search Service startup | Search Service | Initializes Elasticsearch indexes with retry logic during service startup. | Prepares search infrastructure before indexing and query traffic depend on it. A failed initialization can delay or degrade search-related experiences until the service recovers. |

### Startup automation notes

- This process is operationally important even though it is not user-invoked.
- Search-related troubleshooting should consider startup initialization health if indexing or query behaviour looks incomplete after deployment or restart.

---

## 5. User-visible asynchronous workflows

These are not all scheduled jobs, but they directly affect what users see on the websites and therefore should be referenced from the site guides.

| Workflow | Website or audience | What starts it | Background or delayed behaviour | Documentation note |
| --- | --- | --- | --- | --- |
| Vendor onboarding invitation | Vendor Admin applicants | System Admin approves an application | Approval creates an invitation token and sends a vendor approval email with the onboarding link. | Explain that approval and email delivery are linked, but resend or recovery may be required if email delivery fails. |
| Driver onboarding invitation | Drivers invited by vendors | Vendor Admin invites a driver | Invitation handling sends a driver onboarding email containing the Driver Portal onboarding link. | Explain that invitation acceptance happens later in Driver Portal rather than in Vendor Admin. |
| Card payment setup after checkout | Customer front-end customers | Customer places a card order | Payment intent details may not be available immediately and the checkout experience polls until the client secret is ready or times out. | Explain that order submission and payment confirmation can complete in stages. |
| Image upload processing | Admins and vendors managing content | User uploads an image | Upload acceptance is followed by asynchronous image processing and derivative generation. | Explain that uploaded media may appear after a short processing delay. |
| Route optimisation | Driver Portal drivers | Driver triggers optimise route from the delivery dashboard | Optimisation runs as an explicit action, with results retrieved back into the dashboard rather than on a separate page. | Document this inside the dashboard workflow, not as a standalone route-planning page. |

### User-visible async workflow notes

- These items should be cross-referenced from the website guides wherever the delay is visible to the end user.
- The goal is not to expose internal implementation details, but to set the right expectation that some actions are accepted first and completed shortly afterward.

---

## 6. Operational ownership and impact notes

### Ownership summary

| Area | Primary owner |
| --- | --- |
| Daily stats reconciliation and archival | Order Service Functions |
| Commission tier recalculation and cache or search maintenance | Vendor Service |
| Ratings recalculation | Vendor Service |
| Image processing | File Service Functions |
| Chat cleanup | Chat Service |
| Search startup initialization | Search Service |
| Admin tool entry points | System Admin |

### Documentation guidance

- Prefer business-facing wording in the website guides and use this appendix for the implementation-aware summary.
- Mention trigger timing only when it affects operator expectations, reporting freshness, or customer-visible delays.
- If a workflow begins in one website and completes through another surface, document the handoff in both places.

### Operational impact guidance

- Reporting, invoice, and commission documentation should call out the overnight jobs that can affect when data appears current.
- Content and menu documentation should note that image-driven changes may appear after asynchronous processing completes.
- Search and discovery documentation should mention that large content or vendor updates may depend on a follow-up reindex or cache rebuild during recovery scenarios.
