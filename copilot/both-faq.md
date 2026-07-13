---
audience: Both
title: Frequently asked questions — WantFood portals
---

# Frequently asked questions — WantFood portals

## How do I reset or recover vendor access?

If a vendor cannot sign in or has lost access to Vendor Admin, the problem is usually one of: the account was not fully created during onboarding, the account was removed from System Admin, or the email address is wrong. A System Admin user can check the vendor's record and vendor-users list to identify which applies. If the account needs to be recreated, the System Admin user can trigger a new invitation from the application or vendor management area.

## Why is a change I made not appearing on the live site yet?

WantFood uses caching and background indexing to keep the platform fast. After saving a change, there is a normal propagation delay before the change is visible everywhere. This applies to vendor profile updates, menu changes, delivery cost updates, hero slide content, and cuisine ordering. If the delay has passed and the change is still not visible, a System Admin user can run the rebuild caches and reindex vendors tools from the tools dashboard.

## Why is a vendor not appearing in search results?

The most common reasons a vendor is missing from search are: the vendor is inactive, the vendor has no active published menu, the vendor's cuisine type is not set correctly, or the search index has not yet picked up a recent change. A System Admin user should check the vendor's active status, menu state, and cuisine assignment, then run the reindex vendors tool if needed.

## What does commission tier recalculation do?

Commission tier recalculation looks at each vendor's completed order count over a rolling period and compares it against the volume thresholds in the tier configuration. Vendors who have crossed a threshold move to the matching tier. Recalculation normally happens on a nightly schedule. A System Admin user can also trigger it manually from the tools dashboard.

## What is the difference between a platform offer and a vendor offer?

Platform offers are created by System Admin and can apply across multiple vendors or be platform-wide. Vendor offers are created by individual vendors in Vendor Admin and apply only to their own orders. Both types can use promo codes. When both types are active for the same order, stacking rules determine which applies.

## How do I change delivery pricing for my restaurant?

Sign into Vendor Admin, confirm the correct vendor or branch context is active, and navigate to Delivery Costs from the dashboard. Update the pricing settings and save. The change propagates to customer delivery quotes after a normal caching delay.

## What do I do when an order is stuck in a bad state?

Open the order in System Admin orders detail to see the full state history. Identify whether the issue is a state problem (wrong transition), a payment problem (intent not confirmed), a delivery problem (driver not updating status), or a data issue. Use the appropriate recovery tool or escalation path based on what the state history shows.

## How do I add a new admin user to System Admin?

Navigate to the Users section in System Admin. Click to add a new admin user, enter their name and email address, assign the appropriate role, and save. They will receive an invitation to sign in. Only existing System Admin users can add new admin users.

## How do I invite a new driver for my restaurant?

In Vendor Admin, confirm the correct vendor or branch context and navigate to the Drivers list. Click Invite Driver, enter the driver's name and email address, and save. An invitation email is sent to the driver. Once they complete Driver Portal onboarding, they appear as available for delivery assignment.

## Who can see which Copilot documents?

Documents tagged audience SystemAdmin are only visible to questions asked in the System Admin portal. Documents tagged audience VendorAdmin are only visible to questions asked in the Vendor Admin portal. Documents tagged audience Both are visible to questions from either portal. A vendor user can never retrieve content intended only for platform administrators.

## Why is a dish image I uploaded not showing yet?

Image uploads are processed asynchronously. There is a normal delay of seconds to a few minutes between upload and the image appearing in its destination. If the image has not appeared after 10 minutes, check whether an error was recorded against the upload. If no error and the image is still missing, ask a System Admin user to check the image processing logs and whether a cache rebuild is needed.

## How do I change the audience tag on an uploaded Copilot document?

In System Admin, navigate to Admin Tools -> Copilot Admin. In the uploaded documents table, find the file and use the audience selector on that row to change the tag. After changing it, click Re-Index Documentation so the new audience filter takes effect in the search index.

## What is the difference between reindex vendors and rebuild caches?

Reindex vendors refreshes the Elasticsearch search index with the latest vendor and dish data. It affects what appears in customer search results and discovery. Rebuild caches refreshes the cached read models used by the vendor management and customer experience surfaces. They are separate operations. When vendor data looks stale, running both is the safest recovery approach.
