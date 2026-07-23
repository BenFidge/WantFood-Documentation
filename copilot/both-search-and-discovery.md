---
audience: Both
title: Search, discovery, and vendor placement
---

# Search, discovery, and vendor placement

This document covers how vendors and dishes become searchable, what controls vendor placement and ranking in search results, how the reindex and cache rebuild tools work, intent-based search behaviour, and what to check when a vendor is not appearing where expected.

## How vendors and dishes become searchable

WantFood uses Elasticsearch for vendor and dish search. Vendor and dish data is indexed during onboarding and updated whenever the vendor's profile, menu, or dishes change.

For menu content, a dish can carry multiple category assignments. Search indexing stores that multi-category metadata so one dish can be matched and displayed correctly across all of its assigned categories in downstream experiences.

Search indexing is not immediate — it happens through a background process. When vendor data changes (name, cuisine type, dish descriptions, availability), the change needs to be indexed before it appears in search results. This happens automatically as part of normal background processing, but it can be triggered manually if needed.

## What controls vendor placement and ranking in search results

Several factors affect where a vendor appears in search results and discovery surfaces:

- Vendor active status — inactive vendors do not appear in customer search.
- Proximity — the customer's location affects which vendors appear and in what order.
- Cuisine type assignment — vendors tagged with a cuisine type appear in cuisine-filtered searches.
- Ratings — higher-rated vendors generally rank better in discovery surfaces.
- Region assignment — regional targeting affects which vendors appear in location-specific discovery.
- Menu availability — vendors with no active published menu are typically excluded from search.
- Trending signals — recent order volume and engagement can boost placement.

## Reindex vendors tool

The reindex vendors tool in the System Admin tools dashboard manually republishes vendor data to the Elasticsearch search index. Use it when:
- A vendor's profile changes are not appearing in search.
- A newly published menu is not visible in discovery.
- Search results look stale after a maintenance operation.

Reindexing runs as a background job and takes time proportional to the number of vendors. It does not happen instantly — allow several minutes for the reindex to complete before checking results.

## Rebuild caches tool

The rebuild caches tool refreshes the cached read models that many downstream experiences depend on. Use it when:
- Delivery costs, opening hours, or branch details are not reflecting recent changes.
- Vendor data looks stale in Vendor Admin or in customer discovery after a profile update.

Cache rebuild and search reindex are separate operations. If a change is not appearing, it may need both a reindex and a cache rebuild to become fully visible.

## Intent-based search

WantFood supports intent-based or natural-language search queries from the Customer Front-end. When a customer types a phrase like "chicken burger near me" or "Italian restaurant open now", the platform interprets the intent and returns relevant vendors and dishes.

The quality of intent search results depends on how well vendor and dish data is structured and described. Clear dish names, accurate cuisine type tags, and complete vendor profiles all improve intent search accuracy.

## Trending, regional hero content, and cuisine ordering

Trending vendors and featured cuisine types affect what customers see in discovery carousels and browse flows. System Admin can manage cuisine type ordering, regional hero slides, and trending surfacing through the content management and taxonomy areas. Changes to these surfaces are subject to normal cache propagation delays.

## Customer search refinement

System Admin can adjust the ranking signals used for customer search through taxonomy and settings management. This includes cuisine type ordering, region-specific discovery rules, and featured content configuration. Changes to ranking signals affect all customers in the relevant scope.

## What to check when a vendor is not appearing where expected

1. Confirm the vendor is in the active state in the Vendors list in System Admin.
2. Confirm the vendor has an active published menu — inactive or unpublished menus may exclude the vendor from search.
3. Check that the vendor has the correct cuisine type assigned for the search filters the customer is using.
4. If a recent change was made, allow background indexing time to propagate.
5. If the delay has passed and the vendor is still missing, ask a System Admin user to run the reindex vendors and rebuild caches tools.
6. Check whether the vendor is in a region or location scope that matches the customer's search context.
