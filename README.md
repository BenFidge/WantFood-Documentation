# WantFood Website Documentation

This documentation set describes the four primary WantFood websites and the shared automation that supports them. It is intended for operators, administrators, support staff, and documentation authors who need a business-facing view of what each site does today.

## Documentation map

| Document | Purpose | Primary audience |
| --- | --- | --- |
| [System Admin Guide](system-admin-guide.md) | Operational guide for platform administration, finance, moderation, content, and recovery tools. | Platform administrators, operations staff, commercial administrators |
| [Vendor Admin Guide](vendor-admin-guide.md) | Working guide for approved vendors managing restaurants, menus, orders, drivers, offers, and payment settings. | Restaurant owners, vendor managers, branch operators |
| [Driver Portal Guide](driver-portal-guide.md) | Driver-facing guide for onboarding, shifts, deliveries, routing, and availability. | Drivers, dispatch-linked delivery staff |
| [Customer Front-End Guide](customer-front-end-guide.md) | Public ordering guide for discovery, basket, checkout, tracking, reviews, and account management. | Customers, support staff, documentation authors |
| [Automation and Jobs Appendix](automation-and-jobs.md) | Shared reference for scheduled jobs, asynchronous processing, and admin-triggered background work. | Operators, support staff, documentation authors |
| [Complex Processes Appendix](complex-processes.md) | Shared reference for cross-site workflows, onboarding handoffs, async payment behaviour, delivery execution, and operational recovery paths. | Operators, support staff, documentation authors |

## Recommended reading order

1. Start with the guide for the website you are documenting or supporting.
2. Use the automation appendix when a workflow includes background processing, overnight refreshes, or delayed completion.
3. Use the complex processes appendix when a workflow crosses more than one website or needs a single end-to-end narrative.
4. Return to this index when you need cross-site navigation or screenshot coverage.

## Shared documentation conventions

- Use business-facing language first and mention implementation details only when they explain visible behaviour.
- Treat modal flows, AJAX refreshes, and JSON endpoints as supporting details unless they materially change what a user sees.
- Describe navigation using the current page names and workflow entry points confirmed during inventory review.
- Link delayed or background behaviour back to the automation appendix instead of repeating technical explanations in every guide.

## Shared prerequisites by website

| Website | Typical prerequisites |
| --- | --- |
| System Admin | Internal admin account, appropriate admin role, and access to the operational area being documented. |
| Vendor Admin | Approved vendor application for onboarding, then an authenticated vendor-linked user with the correct vendor or branch context. |
| Driver Portal | Driver invitation from Vendor Admin, successful onboarding, and an authenticated driver account for the working area. |
| Customer front-end | Public browsing requires no sign-in, but checkout, order history, addresses, and reviews generally require a customer account. |

## Cross-site navigation summary

| Website | Main entry points | Key follow-on areas |
| --- | --- | --- |
| System Admin | Home or dashboard | Applications, vendors, users, orders, commissions, invoices, offers, reviews, content, tools |
| Vendor Admin | Apply, onboarding landing, vendor dashboard | Restaurant management, menus, orders, drivers, offers, reviews, delivery costs, payment methods |
| Driver Portal | Home landing, onboarding landing, driver dashboard | Active deliveries, delivery dashboard, availability, account |
| Customer front-end | Home page | Search, cuisine types, vendor page, shared basket, checkout, orders, account |

## Screenshot coverage for first release

Capture screenshots after UI review using the current production-like theme and realistic sample data.

| Guide | Recommended screenshots |
| --- | --- |
| System Admin | Dashboard, application detail, vendor detail, commissions dashboard, invoice detail, flagged reviews list, hero slides list, tools dashboard |
| Vendor Admin | Apply page, onboarding landing, dashboard, menu editor, orders kanban, drivers list, offer detail, payment methods page |
| Driver Portal | Onboarding landing, dashboard, active deliveries, delivery dashboard with route result, availability page, account page |
| Customer front-end | Home page, vendor page, dish page, basket experience, checkout, order tracking, order complete or review prompt, addresses page |

## Maintenance note

When routes, menu labels, or workflow entry points change, update the relevant guide first and then refresh this index if the cross-site summary or screenshot list also changes.
