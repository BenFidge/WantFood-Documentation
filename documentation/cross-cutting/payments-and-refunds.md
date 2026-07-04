# Payments and refunds

You use this guide when payment configuration, payment availability, or finance-side follow-up affects how the platform should behave.

## Where payment behaviour appears
- System Admin manages platform payment provider settings.
- Vendor Admin manages vendor payment-method availability and connection state.
- Finance-side invoice operations reflect downstream commercial activity.

## Core rules
- Platform payment settings should match the providers the platform intends to use.
- Vendor payment settings should match the payment methods the business intends to expose.
- Removing or changing payment configuration should be deliberate, because downstream checkout or settlement behaviour depends on it.

## What to watch
- Payment configuration changes may not appear instantly in every downstream experience.
- Invoice actions are finance and support workflows, not customer-facing payment actions.
- When payment behaviour looks wrong, confirm both platform settings and vendor-side settings before assuming the checkout flow is at fault.

## Related
- [Payment settings](../system-admin/pages/payment-settings.md)
- [Configure payment providers](../system-admin/features/configure-payment-providers.md)
- [Payment methods](../vendor-admin/pages/payment-methods.md)
- [Configure payment methods](../vendor-admin/features/configure-payment-methods.md)

