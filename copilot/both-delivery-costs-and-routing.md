---
audience: Both
skillKeys: [vendor-delivery-costs]
title: Delivery costs, pricing configuration, and routing
---

# Delivery costs, pricing configuration, and routing

This document covers how delivery pricing is configured in Vendor Admin, what each field does, how tier matching works, how pricing changes propagate to customer-facing quotes, and what to check when delivery quotes look wrong.

## What the delivery costs page does

The Delivery Costs page in Vendor Admin lets a vendor configure the delivery fees that customers are charged when placing an order. Configuration is **branch-scoped** — each branch has its own delivery cost settings. You must have the correct branch context active before making changes.

To open the page: sign into Vendor Admin → confirm the correct branch context is selected → Delivery Costs from the vendor dashboard.

## The configurable fields

### Distance-based tiers

Tiers let you charge different fees depending on how far a customer is from the branch. Each tier has two values:

- **Up to (miles)** — the maximum distance in miles that this tier covers. The lowest tier in the list applies first.
- **Fee (£)** — the delivery fee charged when the customer's distance falls within this tier.

Tiers are matched in ascending order. The first tier whose "Up to" distance is greater than or equal to the customer's actual distance applies. If the customer's address falls beyond all configured tiers but is within the max delivery distance, no tier match is found and the flat fallback fee is applied instead.

Tiers must be saved in ascending distance order. If you add tiers out of order, the save will be rejected with a validation error.

### Flat fallback fee

If no tiers are configured, the flat fallback fee from the branch settings is used for every order. This value is **shown on the Delivery Costs page as a reference only** — it is not editable here. To change the fallback fee, update the branch's base delivery settings.

When tiers are configured, the fallback fee is used when a customer's distance does not match any tier but is still within the max delivery distance.

### Max delivery distance (miles)

The maximum distance in miles from the branch that the vendor will accept orders from. Orders from customers beyond this distance are rejected before checkout completes. **This field is required when tiers are configured.** The validation will block saving if tiers are present but no max distance is set.

### Out of range message

An optional message shown to customers whose address is beyond the max delivery distance. If left blank, a generic rejection message is shown. Example: "Sorry, we don't deliver to your area."

## How a vendor configures delivery pricing

1. Sign into Vendor Admin and confirm the correct branch context is active.
2. Open Delivery Costs from the dashboard.
3. Review the current flat fallback fee shown as a reference.
4. Add, edit, or remove distance tiers as needed. Each tier needs an "Up to" distance (miles) and a fee (£).
5. Ensure tiers are in ascending distance order — the save will fail otherwise.
6. Set Max Delivery Distance if any tiers are configured — this field is required.
7. Optionally set an Out of Range Message for customers beyond your delivery area.
8. Click Save Changes.

A success or error message appears inline. If the save fails due to validation (e.g., tiers out of order, missing max distance), correct the issue and save again.

## How pricing changes propagate to customer quotes

After saving, the new pricing feeds into the delivery quoting logic used at customer checkout. WantFood uses caching to keep the customer experience responsive, so a saved change may take a short time to appear in customer-facing quotes.

This is expected behaviour. If pricing still looks wrong after allowing propagation time, a System Admin user can trigger a cache rebuild for the affected vendor using the Platform Tools in System Admin.

## Routing and driver availability

Delivery routing uses the available driver pool at the time of order. The factors that affect which driver receives an order include:

- **Shift status** — only drivers who have started a shift are in the active pool.
- **Driver location** — real-time location data identifies the closest available driver.
- **Branch assignment** — drivers assigned to a specific branch are prioritised for orders from that branch.
- **Current delivery load** — drivers already carrying active deliveries may be deprioritised.

When no drivers are available, an order remains in the ready state until a driver becomes available or the vendor manually assigns one from the order detail view.

## What to check when delivery quotes look wrong

1. Confirm the branch context in Vendor Admin is the branch you intended to configure.
2. Open Delivery Costs and check that the saved tiers and max distance reflect the intended pricing.
3. If no tiers are set, the flat fallback fee applies — check whether that value is correct.
4. Check the Max Delivery Distance — an address that exceeds this will be rejected, not quoted.
5. Allow normal propagation time and check the customer-facing quote again.
6. If the issue persists after propagation, ask a System Admin user to run Rebuild Caches for the affected vendor from the Platform Tools dashboard.
7. For platform-wide delivery configuration that may be overriding vendor settings, check System Admin payment and delivery settings.

## Common questions

**Why does the customer see a flat fee even though I configured tiers?**
The customer's address distance may have exceeded all tier upper bounds but still be within the max delivery distance. In that case, the fallback flat fee applies. Review your tier ranges to ensure they cover the expected delivery area.

**Why is Max Delivery Distance required when I add tiers?**
The platform needs to know the outer boundary of deliveries. Without a max distance, there is no way to reject addresses that are too far from the branch. Tiers alone do not define a hard boundary.

**Can I configure delivery costs for the whole vendor rather than per branch?**
No. Delivery costs are configured at the branch level. If you manage multiple branches, switch branch context and configure each one separately.

**Changes were saved but customers still see the old price — what do I do?**
Allow normal cache propagation time first. If the issue persists, ask a System Admin user to rebuild the cache for the affected vendor.
