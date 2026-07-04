# Resilience and edge cases

You use this guide when a workflow leaves the sunny path and you need to explain what safe behaviour should look like.

## Common edge cases
- An onboarding link is invalid or expired.
- A user opens a page without the right vendor or branch context.
- A delivery cannot complete normally.
- A moderation or finance action starts but the visible result lags.
- A saved change does not appear where the user expects it.

## What good resilience looks like
- The site makes it clear that the current action failed or needs a different path.
- Users can retry, step back, or escalate without losing their place in the broader workflow.
- Support and admin tools can explain or repair the visible problem.

## Related
- [Error pages](../system-admin/pages/error-pages.md)
- [Error page](../vendor-admin/pages/error-page.md)
- [Error page](../driver-portal/pages/error-page.md)
- [Support and recovery actions](support-and-recovery-actions.md)

