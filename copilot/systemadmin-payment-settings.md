---
audience: SystemAdmin
skillKeys: [systemadmin-home]
title: Payment provider settings and Stripe configuration
---

# Payment provider settings and Stripe configuration

This document covers how to configure platform-level payment provider settings in System Admin, including Stripe and WorldPay configuration, the relationship to vendor payment settings, and what to check when checkout behaviour looks wrong.

## Platform payment settings overview

The Payment Settings page in System Admin controls the platform-level payment provider configuration. These settings determine which payment providers the platform is registered with and how the platform's Stripe or WorldPay integration is connected.

Platform payment settings are not the same as vendor payment settings. Platform settings define the provider credentials and integration at the platform level. Vendor settings (managed in Vendor Admin) determine which payment methods each vendor makes available to their customers and how their vendor-level connection is configured.

Both layers must be correctly configured for checkout to work correctly.

## Configuring Stripe platform settings

1. Navigate to Payment Settings in System Admin.
2. Enter the Stripe platform credentials — API keys and any webhook secrets required for the integration.
3. Save.

Changes to Stripe platform settings take effect immediately for new checkout sessions. Existing sessions in progress may not see the change until they are completed or refreshed.

## Configuring WorldPay platform settings

Follow the same pattern as Stripe: navigate to Payment Settings, enter the WorldPay credentials, and save.

## Removing a payment configuration

Before removing a payment provider configuration:
1. Confirm that no vendors are currently using that provider.
2. Confirm that no checkout sessions are currently in progress using that provider.
3. Use the Remove Configuration action.

Removing an active payment configuration will immediately break checkout for any vendor or customer relying on it. Removing should only be done during planned maintenance windows or when the provider is no longer in use.

## Relationship to vendor payment settings

After platform-level Stripe configuration is in place, each vendor must still connect their own Stripe account through the payment methods configuration in Vendor Admin. The platform Stripe configuration enables card processing at the platform level; the vendor Stripe Connect setup enables the payout routing to the vendor's own bank account.

Both must be correctly set up for card payments to work end to end.

## Stripe Connect for vendor payouts

Stripe Connect allows the platform to collect card payment revenue and settle the vendor's share (order total minus commission) to the vendor's own Stripe account on the configured payout schedule.

The platform manages the Connect integration at the platform level. Individual vendors complete their Stripe Connect onboarding from within Vendor Admin. System Admin can see the vendor's payment connection state from the vendor detail page but cannot complete Stripe onboarding on behalf of the vendor.

## What to check when checkout behaviour looks wrong after a settings change

1. Confirm the platform payment settings are saved correctly (no missing fields, correct keys).
2. Check whether the vendor involved has their own payment methods configured correctly in Vendor Admin.
3. Confirm that the payment provider is not experiencing an outage (check the Stripe or WorldPay status page).
4. Check whether the issue is occurring for all vendors or just one — if just one, the problem is likely in the vendor's own payment configuration.
5. If the issue started immediately after a settings change, review what was changed and consider reverting to the previous configuration while investigating.
