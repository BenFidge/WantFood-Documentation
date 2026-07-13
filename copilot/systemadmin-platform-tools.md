---
audience: SystemAdmin
skillKeys: [systemadmin-home]
title: Platform tools, operational recovery, and background operations
---

# Platform tools, operational recovery, and background operations

This document covers the System Admin tools dashboard, each available tool, when to use them, and how to diagnose whether a problem is a state issue, a permission issue, or a propagation issue.

## Tools dashboard

The tools dashboard in System Admin is the starting point for operational recovery actions. It provides access to tools that refresh, repair, or backfill platform data. Use it when normal workflows are not enough and you need to manually trigger background operations.

## Reindex vendors

The reindex vendors tool republishes vendor and menu data to the Elasticsearch search index.

When to use it:
- A vendor's profile or menu changes are not appearing in customer search results.
- A newly published menu is not visible in discovery.
- Search results look stale after a maintenance operation or bulk data change.

How it works: The tool queues a background job that iterates all vendor records and republishes their data to the search index in batches. This is not instant — allow several minutes for the job to complete. Check search results after the job finishes.

## Rebuild caches

The rebuild caches tool refreshes the cached read models that Vendor Admin and customer-facing experiences depend on.

When to use it:
- Delivery costs, opening hours, branch details, or other vendor configuration looks stale despite being saved correctly.
- Vendor data appears outdated in customer discovery even after a reindex.

Rebuild caches and reindex vendors are separate operations. If data looks stale, running both is the most thorough recovery approach.

## Import ONS postcodes

The ONS postcodes import tool loads updated UK postcode data from the Office for National Statistics. Use it when:
- New postcode data has been released and delivery coverage or routing needs updating.
- Postcode-based delivery quoting is returning unexpected results.

After import, postcode stats are accessible from a JSON output view accessible from the tools dashboard.

## Integrity check

The integrity check tool validates platform data for consistency — checking for orphaned records, mismatched states, or data that should have been cleaned up by normal processing. Run it:
- After a maintenance operation or migration.
- When unusual behaviour suggests a data consistency problem.

Review the output carefully. Not every flag from an integrity check requires immediate action — some are expected states. Use the results to identify whether any issues require manual correction.

## Manual commission tier recalculation trigger

Triggers the same recalculation logic as the nightly commission tier job but on demand. Use it when:
- A tier assignment needs to apply before the next nightly run.
- A recent bulk change to tier configurations needs to be applied immediately.

## Manual rating recalculation trigger

Triggers a full recalculation of vendor, branch, and dish ratings based on the current set of approved reviews. Use it when:
- A moderation decision has changed the review set for one or more vendors and the rating needs to reflect the change before the nightly run.
- Ratings look incorrect after a bulk review operation.

## Invoice generation trigger

While invoice generation is typically done through the finance workflow (uninvoiced dashboard or invoice list), the tools dashboard may expose an additional trigger for special cases. Use the normal finance workflow for routine invoicing.

## Recovery pattern — diagnosing the class of problem

Before running any tool, diagnose which class of problem you are dealing with:

### State problem
The record is in a state it should not be in (wrong status, unexpected transition). State problems usually need a workflow action or an operations team fix rather than a tool run.

### Permission problem
A user cannot access a page or action they should be able to. Check the user's role assignment. Tool runs do not fix permission problems.

### Propagation problem
A saved change is not yet visible where expected. This is the most common class and is the right situation for cache rebuilds or reindex runs. Confirm the source record actually saved before running tools.

The typical diagnostic flow is: confirm the source record saved -> allow normal propagation time -> if still stale, run rebuild caches -> if search-related, also run reindex vendors -> if still wrong, investigate as a state or data integrity problem.
