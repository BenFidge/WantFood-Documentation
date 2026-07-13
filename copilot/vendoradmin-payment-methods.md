---
audience: VendorAdmin
title: Payment methods and configuration
---

# Payment methods and configuration

This document covers how to configure which payment methods your restaurant accepts in Vendor Admin, how to set up Stripe for card payments, how to enable or disable cash payments, and what to check when customers report payment problems.

## Payment methods page

The Payment Methods page shows the current payment configuration for the active vendor or branch context. It shows which payment methods are enabled (card, cash, or both) and the connection state for any provider that requires a setup step.

Always confirm the correct vendor or branch context is active before making changes.

## How to configure Stripe for card payments

To accept card payments, you must connect a Stripe account to your vendor record.

1. Navigate to Payment Methods from the vendor dashboard.
2. If Stripe is not yet connected, you will see a Connect Stripe option.
3. Follow the Stripe Connect onboarding flow. Stripe will ask you to provide business details, banking information, and identity verification (KYC). This is handled directly by Stripe — WantFood does not see or store your banking credentials.
4. Once Stripe Connect onboarding is complete, card payments are enabled for your vendor.

If you have already connected Stripe and the connection state looks wrong (disconnected or error), use the reconnect or re-authorise option on the Payment Methods page.

## How to enable or disable cash payments

1. Navigate to Payment Methods.
2. Use the toggle or checkbox for cash payments to enable or disable it.
3. Save the change.

Enabling cash means customers can choose to pay in cash at delivery. Disabling cash means only card (or other enabled methods) are available at checkout.

## How to remove a payment configuration

To disconnect Stripe or remove another payment provider:
1. Open the Payment Methods page.
2. Use the Remove or Disconnect action for the payment method you want to remove.
3. Confirm the removal.

Removing a payment method takes effect immediately for new checkout sessions. Customers who are already mid-checkout may not see the change until they refresh or restart checkout.

Do not remove a payment method during busy trading periods without understanding the impact on active and upcoming orders.

## How vendor payment settings relate to platform payment settings

The platform has its own Stripe and payment provider configuration managed by System Admin. Your vendor-level Stripe Connect setup is layered on top of the platform configuration. Both must be correctly set up for card payments to work end to end:
- The platform Stripe configuration enables the platform to process card payments.
- Your Stripe Connect setup enables the platform to route your share of the revenue to your bank account.

If your Stripe connection looks correct but card payments are still failing, contact System Admin to check the platform-level payment configuration.

## What to check when customers report payment problems at checkout

1. Confirm card payments are enabled and Stripe is connected (not showing an error or disconnected state) on the Payment Methods page.
2. Ask the customer whether the problem is with a specific card or all cards. If only specific cards fail, the issue is likely with the customer''s card or bank, not your configuration.
3. Confirm cash is enabled if the customer is trying to pay with cash and cannot select it.
4. If card payments are failing for all customers, check the Stripe status page for any service disruptions.
5. If the issue is not customer-side or Stripe-side, contact System Admin with the vendor name and a description of the error the customer is seeing.
