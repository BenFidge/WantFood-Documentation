---
audience: SystemAdmin
skillKeys: [systemadmin-reviews-moderation]
title: Review moderation, ratings, and recalculation
---

# Review moderation, ratings, and recalculation

This document covers how flagged reviews are managed in System Admin, the resolution workflow, how ratings are calculated and recalculated, and how ratings affect vendor visibility.

## Flagged reviews list

The Flagged Reviews list shows customer reviews that have been escalated for platform moderation. A review is flagged when:
- A vendor flags the review from Vendor Admin as requiring platform moderation.
- The platform's automated content moderation detects potentially problematic content.

The list shows the vendor, the review text, the rating, when it was submitted, and when it was flagged. Use the list to work through reviews that need a moderation decision.

## Reviewing a flagged item

Open a flagged review to see:
- The full review text and rating submitted by the customer.
- The reason for flagging (vendor-reported or automated).
- Any vendor notes added when flagging.
- The vendor's response if they submitted one within the response window.
- The order context (which order the review relates to).

Read the review in the context of the order before making a moderation decision.

## How to resolve a flagged review

A flagged review can be resolved in three ways:

### Approve (keep the review)

Use Approve when the review is legitimate even if the vendor disagrees with it. The review remains visible to customers and the rating counts toward the vendor's score.

### Reject (remove the review)

Use Reject when the review violates platform content policies — for example, it contains offensive language, is clearly fabricated, or is about the wrong vendor. A rejected review is removed from public view. The rating from a rejected review is removed from the vendor's calculated score.

### Escalate

Use Escalate when the decision requires legal review, involves a fraud investigation, or is outside normal moderation scope. Add a note explaining the reason for escalation.

## Vendor response window

Vendors have a limited window after a review is submitted to respond or flag it for moderation. After the window closes, the vendor can no longer flag the review. System Admin can still moderate a review after the vendor response window closes if the review is referred through another channel.

## Manual rating recalculation

Ratings in WantFood are calculated on a nightly schedule. If a moderation decision (approving or rejecting a flagged review) changes the set of valid reviews for a vendor, the vendor's rating may not update until the next nightly run.

To apply the change immediately:
1. Go to the vendor's record or to the flagged reviews area.
2. Use the Recalculate Ratings action.
3. The recalculation refreshes vendor, branch, and dish ratings based on the current set of approved reviews.

## How ratings affect vendor visibility

A vendor's rating directly affects their position in customer search and discovery surfaces. Higher-rated vendors appear more prominently in nearby, search, and cuisine-filtered results. Vendors with very low ratings or many unresolved review flags may be deprioritised in discovery.

After a significant moderation decision that changes a vendor's rating materially, confirm the rating has updated (trigger manual recalculation if needed) and that the vendor's position in discovery is appropriate.

## Privacy considerations when resolving reviews

Reviews may contain customer details or order-specific information. Handle review moderation content with the same care as other customer data. Do not share the review text or customer details outside the moderation workflow. When a review needs to be retained for escalation or investigation, follow the platform's data handling policy.
