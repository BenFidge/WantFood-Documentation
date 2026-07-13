---
audience: SystemAdmin
skillKeys: [systemadmin-copilot-admin]
title: Copilot Admin — managing AI documentation
---

# Copilot Admin — managing AI documentation

This document covers the Copilot Admin area in System Admin: what it is for, how to upload and manage documentation, how to re-index, and how to author effective Copilot documents.

## What the Copilot Admin area is for

The Copilot Admin area is the System Admin interface for managing the knowledge base that powers the WantFood AI Copilot. Only System Admin users can access Copilot Admin — Vendor Admin users cannot upload, view, or manage Copilot documentation.

From Copilot Admin you can:
- Upload Markdown documentation files for the AI to use when answering questions.
- Set or change the audience for each document (SystemAdmin, VendorAdmin, or Both).
- Re-index the documentation so new and updated files become searchable by the AI.
- Delete files that are outdated or incorrect.

## Uploading documentation

1. Navigate to Admin Tools -> Copilot Admin in the System Admin sidebar.
2. In the Upload Documentation section, click to select one or more .md files. Only Markdown files are accepted.
3. Before saving, review the resolved audience shown for each file. The system reads YAML front-matter from the file automatically and shows you the result.
   - If front-matter is present, the front-matter audience value is used.
   - If no front-matter is present but the filename starts with systemadmin-, vendoradmin-, or both-, that prefix determines the audience.
   - If neither is present, use the form selector to choose the audience manually. The default is Both.
4. Click Upload to save the files.

## The uploaded documents table

The uploaded documents table shows all files currently stored in the Copilot knowledge base. Each row shows:
- File name.
- Resolved audience (editable after upload — use the audience selector on the row and save).
- File size.
- Last modified date.
- Skill keys extracted from front-matter (if present).
- Delete action.

To change the audience of an uploaded file, use the audience selector on the row, select the new value, and save. Remember to re-index after changing audience values so the index reflects the new filter.

## Re-indexing documentation

Re-indexing rebuilds the Azure AI Search vector index from the current contents of the document store. You must re-index after every upload, update, deletion, or audience change for the Copilot to use the new state.

1. Click Re-Index Documentation.
2. Wait for the job to complete. The panel shows a progress indicator.
3. When complete, the result shows: Indexed (chunks successfully stored), Skipped (files that had no changes or could not be parsed), Failed (files that caused errors).
4. Review any Failed entries and fix the underlying files if needed.

Re-indexing does not remove chunks for files that are still in the store — it upserts. To remove chunks for a deleted file, delete the file first and then re-index.

## Deleting a document

1. In the uploaded documents table, find the file to remove.
2. Click Delete on that row.
3. Confirm the deletion.
4. Click Re-Index Documentation to remove the file's chunks from the search index.

If you skip re-indexing after deletion, the AI may continue to answer from the deleted file's content until the next re-index run.

## Authoring guidance — how to write a good Copilot document

Good Copilot documents produce accurate AI answers. Follow these principles:

### Use ## and ### headings throughout

Vector retrieval is semantic. A file with clear headings retrieves more accurately than a wall of text. Break every topic into named sections.

### Write the phrases users actually type

Include natural-language questions and phrases in your headings and body content. For example: "how do I create a commission tier", "what happens when I reject a vendor application", "why is a vendor not appearing in search results". The AI uses these phrases to match user questions to your content.

### Use numbered steps for procedures

Numbered step lists are more useful than narrative paragraphs when explaining how to do something.

### One file per feature or area

Write small, focused files. Do not put everything into one document. A file about commission tiers should cover only commission tiers — not invoicing, not vendor management.

### Use the correct audience

Documents tagged SystemAdmin are only retrievable by System Admin users. Documents tagged VendorAdmin are only retrievable by Vendor Admin users. Documents tagged Both are retrievable by both. Never tag sensitive platform-admin content as Both.

## What to do when the AI gives a wrong answer

1. Identify which part of the answer is incorrect.
2. Find the Copilot document that covers the topic (check the sources footer at the bottom of the AI reply).
3. Edit the Markdown file to add or correct the relevant content.
4. Upload the updated file via Copilot Admin (same filename replaces the existing file).
5. Re-index.
6. Test the question again to confirm the answer has improved.
