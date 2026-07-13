---
audience: SystemAdmin
skillKeys: [systemadmin-invoices]
title: Invoicing, Stripe Connect payouts, and finance operations
---

# Invoicing, Stripe Connect payouts, and finance operations

This document covers how to generate invoices for vendors, how to review and manage invoices, how to mark invoices paid or cancel them, and how Stripe Connect payouts work for vendors who accept card payments.

## Uninvoiced dashboard

The uninvoiced dashboard shows activity that has not yet been turned into invoices. Use it to:
- See which vendors have completed orders that are ready to invoice.
- Review the activity counts and amounts before generating invoices.
- Choose whether to generate for one vendor or run the full monthly batch.

Open the uninvoiced detail for a vendor to see the individual order-level line items that will be included in their next invoice.

## Generating an invoice — single vendor

1. Open the uninvoiced dashboard.
2. Find the vendor whose activity you want to invoice.
3. Use the Generate Invoice action for that vendor.
4. The system creates an invoice record capturing the commission amounts from that vendor's uninvoiced completed orders.
5. The generated invoice appears in the Invoice list.

## Generating invoices — monthly run

For the full monthly invoicing batch:
1. Open the uninvoiced dashboard or the Invoice list.
2. Use the Generate Monthly Invoices action.
3. The system generates invoices for all vendors with uninvoiced activity in the billing period.
4. Monitor the invoice list to confirm all expected invoices were generated.

## Invoice list

The invoice list shows all generated invoices with their current state. Use it to:
- Find an invoice for a specific vendor and period.
- Check whether an invoice has been paid, cancelled, or is pending.
- Open an invoice for detailed review.

## Invoice detail — what it shows

The invoice detail page shows the full state of one invoice:
- Invoice number and period.
- Vendor details.
- Line items — the commission amounts from each order in the invoice period.
- Subtotals and total.
- Current payment state (pending, paid, cancelled).
- Any notes or history of state changes.

Review the line items to confirm the invoice is accurate before marking it paid.

## Marking an invoice paid

1. Open the invoice from the Invoice list.
2. Review the line items and total.
3. Use the Mark Paid action.
4. Record any relevant payment reference.
5. The invoice moves to the paid state.

## Cancelling an invoice

Use the Cancel action when an invoice was generated in error or when the underlying orders have changed. Cancelled invoices are retained in the system for audit purposes — they cannot be deleted.

1. Open the invoice and review the reason it needs to be cancelled.
2. Use the Cancel Invoice action.
3. Record a reason for the cancellation.
4. The invoice moves to the cancelled state and is no longer included in outstanding totals.

## Stripe Connect payouts — how vendor payouts work

Vendors who accept card payments through WantFood are connected to the platform via Stripe Connect. Card order revenue flows through the platform's Stripe account and is settled to the vendor's connected Stripe account according to the configured payout schedule.

The commission amounts invoiced to the vendor are separate from the Stripe payout amounts. The platform retains the commission from the order revenue before settling the remainder to the vendor.

For cash orders, there is no Stripe payout — the vendor collects cash directly. Cash orders still contribute to commission calculations and appear in invoices.

## What to check when an invoice total looks wrong

1. Open the invoice detail and review the line items individually.
2. Check whether any orders in the period had their commission rate change mid-period due to a tier recalculation.
3. Confirm whether all expected orders appear — if some are missing, check the uninvoiced dashboard to see if they were not yet processed.
4. If a specific order's commission rate looks wrong, open that order in the Orders list and check the commission state on the order detail page.
5. If the invoice was generated from a stale commission configuration, it may need to be cancelled and regenerated after correcting the tier assignment.
