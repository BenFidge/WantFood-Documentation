---
audience: SystemAdmin
skillKeys: [systemadmin-cuisine-types]
title: Taxonomy, content safety, settings, and GDPR
---

# Taxonomy, content safety, settings, and GDPR

This document covers how cuisine types are managed and ordered, the bad words list and content safety controls, the settings page, GDPR and account deletion workflows, and customer search refinement.

## Cuisine types

Cuisine types are the taxonomy used to categorise vendors and dishes. They appear in customer discovery and search filters. The quality of cuisine type assignment and ordering directly affects how well customers can find the food they are looking for.

### Managing cuisine types

The Cuisine types list in System Admin shows all defined cuisine types. Use it to review the taxonomy and identify types that need reordering, renaming, or removal.

### How to manage cuisine ordering for mobile vs web

Cuisine ordering controls the sequence in which cuisine types appear in the mobile app and in the web discovery experience. Mobile and web can have different orderings to suit each surface's layout.

1. Open the Cuisine types list.
2. Use the ordering interface to set the sequence for the mobile display.
3. Set the sequence for the web display separately.
4. Save the ordering.

Changes to cuisine ordering are subject to normal cache propagation — they appear in the customer surfaces after caches refresh.

### How cuisine types affect vendor placement

Vendors tagged with a cuisine type appear in that cuisine's filtered discovery results. Vendors with no cuisine type assigned may not appear in filtered searches. When a vendor is missing from a specific cuisine filter, confirm their cuisine type assignment in the vendor record.

## Bad words list

The bad words list is a set of blocked words and phrases that the platform checks against user-generated content (dish names, descriptions, review text) to prevent abusive or inappropriate content from appearing.

### How to add a bad word

1. Navigate to Bad Words List in System Admin.
2. Click Add.
3. Enter the word or phrase.
4. Save. The word is immediately active in content safety checks.

### How to edit a bad word

Open the entry and update the text. Changes take effect immediately.

### How to delete a bad word

Use the Delete action on the entry. Deleted words are removed from content safety checks immediately.

### How to bulk-import bad words

The Bad Words list supports bulk import via a file upload. Upload a newline-separated text file of words. Imported words are added to the existing list without overwriting it.

## Settings page

The Settings page in System Admin contains shared configuration values that affect admin-managed behaviour across the platform. This includes parameters for scheduled jobs (calculation times), content safety thresholds, and other platform-level settings.

Review the settings page before making significant operational changes. Changing a setting here affects all users and all workflows that depend on it.

## GDPR and account deletion

When a customer submits a data subject access request or deletion request under GDPR:

1. The request should come through the platform's designated privacy contact.
2. A System Admin user processes the deletion by removing the customer's account and associated personal data through the appropriate system administration process.
3. Orders, reviews, and other records that are needed for legal, finance, or audit purposes may be retained in anonymised form as permitted by the platform's data retention policy.
4. Confirm the deletion has been completed and record the outcome as required by the platform's GDPR handling procedure.

Do not use the bad words list or content management tools to respond to GDPR requests — use the dedicated account management path.

## Customer search refinement

System Admin can influence how customers find vendors and dishes through taxonomy and settings management. This includes:
- Cuisine type ordering — controlling which categories appear prominently.
- Region targeting — controlling which vendors appear in location-based discovery.
- Trending and featured content configuration — editorial surfacing in discovery carousels.

Changes to ranking signals and discovery configuration affect all customers in the relevant scope and should be deliberate.
