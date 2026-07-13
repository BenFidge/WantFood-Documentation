---
audience: Both
title: Image upload and processing
---

# Image upload and processing

This document covers where images are uploaded across the WantFood platform, allowed formats and sizes, how async image processing works, how to replace or remove an image, and what to do when an image does not appear after upload.

## Where images are uploaded

Images are used in three main areas across the platform:

- Content assets in System Admin — shared media used for homepage, hero slides, and campaign content.
- Hero slides in System Admin — the images that appear in the homepage hero carousel.
- Menu dishes in Vendor Admin — images attached to individual dishes in the menu editor.

## Allowed formats and sizes

Accepted image formats are JPEG and PNG. The platform enforces a maximum file size at upload. Images that exceed the size limit or use an unsupported format are rejected with an error.

Specific size limits and recommended dimensions are shown in the upload UI at the point of upload. Dish images should be square or near-square for best appearance in the customer menu view. Hero slide images should match the recommended banner dimensions shown in the hero slide editor.

## Async image processing — why images may not appear immediately

Image uploads in WantFood are processed asynchronously. When you upload an image, the file is accepted and queued for processing. A background service then processes the image — resizing, transcoding, and making it available in the platform's CDN-accessible storage.

This means there is a delay between when you upload an image and when it appears in its destination. This is normal behaviour and not an error. The delay is typically seconds to a few minutes depending on image size and processing load.

## How to replace an image

To replace an existing image with a new one, upload the new image through the same upload surface where the original was uploaded. The new file replaces the old one after processing completes.

- For dish images: open the dish in the menu editor and upload a new image.
- For content assets: open the asset in the content asset library and use the replace action.
- For hero slides: open the slide in the hero slide editor and upload a new image.

## How to remove an image

Images can be removed from the same management surface where they were uploaded. Removing a dish image leaves the dish without an image — the dish still appears in the menu but without a photo. Removing a content asset removes it from the shared library; any hero slide or content reference that used the asset will need updating.

## Content moderation

Uploaded images go through automated content moderation as part of the processing pipeline. Images that fail moderation are rejected and do not appear in the platform. If an uploaded image never appears and no error was shown, contact the platform operations team to check whether moderation rejected the file.

## What to do when an image does not appear after upload

1. Wait a few minutes for async processing to complete — this is normal.
2. Refresh the page where you expect to see the image.
3. If the image still does not appear after 10 minutes, check whether an error was recorded against the upload in the management surface.
4. Confirm the file format and size were within the accepted limits.
5. If a hero slide or content asset image is missing from the customer-facing site, ask a System Admin user to check whether a cache rebuild is needed.
6. If none of the above resolves it, the image may have failed content moderation — contact the platform operations team.
