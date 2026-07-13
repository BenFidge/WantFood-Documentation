---
audience: Both
title: How to manage WantFood AI Copilot documentation
---

# How to manage WantFood AI Copilot documentation

This guide explains how the WantFood AI Copilot works, how its knowledge base is maintained, and how to write and upload good documentation so the AI gives accurate answers to both System Admin and Vendor Admin users.

## What is the WantFood AI Copilot?

The AI Copilot is an AI assistant embedded in the SystemAdmin and VendorAdmin portals. It appears as a slide-out chat panel accessible from every page via the AI Assistant button in the top navigation bar.

The Copilot answers questions about platform operations, vendor management, menu management, orders, commissions, invoices, promotions, content, and other WantFood topics. It does this using Retrieval Augmented Generation (RAG): when a user asks a question, the system retrieves the most relevant passages from the uploaded Markdown documentation and includes them in the prompt sent to the AI model.

The Copilot is not the ACS order chat. The order chat is a three-way messaging channel between vendor, customer, and driver. The Copilot is an AI knowledge assistant — it does not send messages to vendors or customers.

## How RAG works

1. You write a Markdown file explaining a WantFood topic and upload it via the Copilot Admin area in System Admin.
2. When you click Re-Index Documentation, the system splits each file into overlapping chunks (~300 words each), creates a vector embedding for each chunk, and stores everything in the Azure AI Search index.
3. When a user asks the Copilot a question, the user's message is embedded and matched against the index. The top matching chunks are retrieved and sent to the AI model alongside the user's question.
4. The model uses only the retrieved content to answer. If the docs do not cover the question, the Copilot says so rather than guessing.

## How to upload and re-index documentation

1. Sign into System Admin.
2. Navigate to Admin Tools -> Copilot Admin in the sidebar.
3. Use the Upload Documentation panel to select one or more .md files. Only Markdown files are accepted.
4. For each file, confirm the resolved audience (the system reads YAML front-matter automatically and shows you the result before saving).
5. Click Upload.
6. After uploading, click Re-Index Documentation. The panel shows Indexed / Skipped / Failed counts and any errors when the run completes.
7. You do not need to re-index after every single upload — you can batch uploads and re-index once.

## Audience tagging — three ways

Every Copilot document is tagged with an audience that controls which portal can retrieve it. The audience is resolved in this priority order:

### 1. YAML front-matter (highest precedence — preferred)

Add a front-matter block at the top of the file:

```
---
audience: VendorAdmin
skillKeys: [vendor-menu-builder]
title: Managing your menu
---
```

Valid audience values: SystemAdmin, VendorAdmin, Both.

### 2. Filename prefix (belt-and-braces)

Name the file with the prefix systemadmin-, vendoradmin-, or both-. For example:
- vendoradmin-managing-menus.md resolves to audience VendorAdmin
- systemadmin-commission-tiers.md resolves to audience SystemAdmin
- both-order-lifecycle.md resolves to audience Both

### 3. Upload-form selector (fallback)

If a file has no front-matter and no recognisable filename prefix, the upload form lets you select the audience manually. The default is Both.

The Copilot Admin UI always shows the resolved audience before you confirm the upload, so you can catch mistakes before the file is stored.

## Audience filter in retrieval

- A SystemAdmin user's question only retrieves chunks tagged SystemAdmin or Both.
- A VendorAdmin user's question only retrieves chunks tagged VendorAdmin or Both.
- A vendor can never see content authored only for platform administrators.

## Skill keys

A skill key associates a document with a specific portal page. When a user is on a page that has a registered skill, the system boosts retrieval for documents whose skillKeys list includes that key.

Current System Admin skill keys: systemadmin-home, systemadmin-vendors, systemadmin-vendor-edit, systemadmin-commission-tiers, systemadmin-invoices, systemadmin-reviews-moderation, systemadmin-cuisine-types, systemadmin-promotions, systemadmin-copilot-admin.

Current Vendor Admin skill keys: vendor-home, vendor-menu-builder, vendor-orders, vendor-order-detail, vendor-drivers, vendor-team, vendor-promotions, vendor-profile, vendor-sales-reports, vendor-delivery-costs.

## How to write a good Copilot document

Good documents produce good AI answers. Poor documents produce vague or incorrect answers.

### Use headings generously

Break every file into ## and ### sections. Retrieval is semantic — short, well-headed files retrieve better than one long wall of text.

### Write the phrases users actually type

Include natural-language phrasing in your headings and body text. For example: "how do I add a new commission tier", "what happens when I publish a menu", "why is my vendor not appearing in search results", "what is the difference between a platform offer and a vendor offer".

### Use numbered steps for procedures

Step-by-step instructions are more useful to the AI (and to the reader) than narrative paragraphs.

### One file per area or feature

Do not write one giant document covering everything. Write one focused file per topic. For example: systemadmin-commission-tiers.md covers only commission tiers; vendoradmin-menu-management.md covers only menu management.

### Avoid duplication

Do not repeat the same content across files. If two topics share an explanation, write it once in a both- file.

## File naming conventions for this folder

Files prefixed both- have audience Both and cover cross-portal topics. Files prefixed systemadmin- have audience SystemAdmin and cover platform administrator topics. Files prefixed vendoradmin- have audience VendorAdmin and cover restaurant owner topics. The file copilot-guide.md is this guide and has audience Both.

## What to do when the AI gives a wrong answer

1. Identify which part of the answer is wrong.
2. Find the document (or write a new one) that should cover the topic correctly.
3. Update or create the Markdown file with accurate, well-headed content.
4. Upload the updated file via Copilot Admin (it replaces the existing file with the same name).
5. Click Re-Index Documentation.
6. Test the question again in the chat panel.

## Keeping the doc set up to date

When you ship a new feature: write the doc, upload, re-index. When a workflow changes: update the relevant doc, upload, re-index. When the AI gets something wrong: improve the doc, upload, re-index. You do not need a code deployment to update the Copilot's knowledge — the change takes effect as soon as you re-index.

## Deleting a document

1. In the Copilot Admin uploaded documents table, click Delete on the row for the file you want to remove.
2. Deleting a file does not automatically re-index. Click Re-Index Documentation after deleting to remove the chunks from the search index.
3. If you forget to re-index after deletion, the AI may still answer from the old content until the next re-index run.

## Developer notes — local provisioning

The Copilot uses Azure OpenAI (chat completions and embeddings) and Azure AI Search (vector index). There is no local emulator for either service. When running locally with Aspire, real Azure resources are provisioned automatically into a per-developer resource group (rg-WantFood-Dev-username) on the first aspire run. First run takes approximately two minutes. Subsequent runs reuse existing resources.

To tear down dev resources when not actively working on Copilot features, run: az group delete -n "rg-WantFood-Dev-$env:USERNAME" --yes --no-wait
