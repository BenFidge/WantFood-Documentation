---
audience: SystemAdmin
skillKeys: [systemadmin-home]
title: Content management — assets, hero slides, regions, and support-local content
---

# Content management — assets, hero slides, regions, and support-local content

This document covers how to manage the shared content asset library, hero slides, support-local slides, and regions in System Admin, including propagation behaviour for content changes.

## Content asset library

The content asset library is the shared repository of media assets that other admin-managed content relies on — including images used in hero slides and support-local slides.

### Uploading an asset

1. Navigate to the Content Asset Library in System Admin.
2. Click Upload.
3. Select the image file (JPEG or PNG, within the size limit).
4. Add metadata — name, description, and any relevant tags.
5. Save. Image processing is asynchronous — the asset is available after processing completes (typically within minutes).

### Updating asset metadata

Open the asset and edit the name, description, or tags. Metadata changes are immediate and do not require re-upload.

### Deleting an asset

Before deleting, confirm the asset is not referenced by any hero slide or support-local slide. Deleting an asset that is in use removes the image from any content that references it. Delete the referencing content first, then delete the asset.

## Hero slides

Hero slides are the banner carousel on the WantFood customer homepage. Managing them involves creating, editing, publishing, unpublishing, and reordering slides.

### How to create a hero slide

1. Navigate to Hero Slides List in System Admin.
2. Click to create a new slide.
3. Set the slide title, body text, call-to-action link, and select an image from the content asset library.
4. Set the display order.
5. Save as draft.

### How to edit a hero slide

Open the slide from the Hero Slides list and update the fields. Changes to a published slide are not live until you re-publish.

### How to publish a hero slide

Open the slide and use the Publish action. Published slides appear in the customer homepage hero carousel immediately after caches refresh.

### How to unpublish a hero slide

Open a published slide and use the Unpublish action. The slide is removed from the customer homepage after caches refresh but the record is retained.

### How to reorder hero slides

From the Hero Slides list, use the reorder interface to drag slides into the desired sequence. Save the new order. The updated order appears on the homepage after caches refresh.

## Support-local slides

Support-local slides are promotional cards on the customer homepage promoting local restaurants or campaigns. They follow the same create, edit, publish, unpublish, reorder, and delete pattern as hero slides.

Use Support Local slides for neighbourhood or regional campaigns where you want to feature specific vendors or local food stories.

## Regions

Regions are geographic or categorical groups that influence content targeting and discovery behaviour. A region can be associated with hero slides, support-local slides, or vendor discovery to control what customers in a given area see.

### How to create a region

1. Navigate to Regions List in System Admin.
2. Click to create a new region.
3. Set the region name and any targeting parameters.
4. Save.

### How to edit or remove a region

Open the region from the Regions list. Edit the settings and save. Before removing a region, confirm it is not actively used by any content targeting rules. Removing a region that is in use may affect the visibility of associated content.

## Trending and monthly giveaway

The platform supports editorial surfacing of trending vendors and monthly giveaway campaigns through the content management and settings surfaces in System Admin. Configuration for trending signals and giveaway parameters is available in the Settings area.

## Propagation delays for content changes

Content changes (publishing a slide, uploading an asset, changing region settings) may not appear immediately in customer-facing surfaces. WantFood uses caching to keep the homepage responsive. After making content changes, allow normal cache propagation time. If a change is not appearing after the expected delay, ask the operations team to run the rebuild caches tool for the affected surface.

Image changes within slides are subject to the additional async image processing delay on top of cache propagation.
