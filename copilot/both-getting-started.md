---
audience: Both
title: WantFood platform overview — portals, roles, and access
---

# WantFood platform overview — portals, roles, and access

This document covers what WantFood is, the four portals that make up the platform, who uses each portal, what each role can and cannot do, and how accounts are created.

## What is WantFood?

WantFood is a food delivery platform connecting customers with local restaurants. It is built from four distinct portals each serving a separate audience.

## The four portals

### System Admin

System Admin is the platform administration portal used by WantFood operations staff. It covers vendor application review, vendor lifecycle management, commission and invoice operations, order support and investigation, promotions, content management, taxonomy, payment provider configuration, and platform recovery tools.

### Vendor Admin

Vendor Admin is the restaurant management portal used by restaurant owners, vendor managers, and branch operators. It covers applying to join WantFood, onboarding, managing menus and dishes, handling live orders, managing drivers, running promotions, responding to reviews, configuring delivery costs, and setting up payment methods.

### Driver Portal

Driver Portal is used by delivery drivers. It covers accepting the invitation to join, completing onboarding, starting and ending shifts, managing active deliveries, updating delivery status, reporting delivery issues, planning routes, and managing weekly availability.

### Customer Front-end

The Customer Front-end is the ordering experience used by customers. It covers discovering vendors and dishes, managing a basket, checkout and payment, tracking live orders, submitting reviews, and managing saved addresses.

## Who can access which portal?

Each portal is completely separated. A signed-in user in one portal cannot access another portal's data or actions.

- System Admin users are internal WantFood operations, commercial, and content staff. Access is granted by adding the person as an admin user through the Users section of System Admin.
- Vendor Admin users are restaurant owners and their staff. They gain access through the vendor onboarding journey, which is triggered by a System Admin approval of a vendor application.
- Driver Portal users are delivery drivers. They gain access through a driver invitation sent from Vendor Admin.
- Customer Front-end users are members of the public who register directly.

## Role boundaries — what each role can and cannot do

System Admin users can work across all platform-level operational areas: vendors, commissions, invoices, orders, content, promotions, reviews, taxonomy, payment settings, and tools. They cannot manage vendor menus, accept orders on behalf of vendors, or access driver portal work.

Vendor Admin users can work only within the vendor and branch contexts they hold. They can manage their menus, handle their orders, manage their drivers, run their promotions, and configure their delivery and payment settings. They cannot see other vendors' data or access any System Admin functionality.

Driver users can work only within their own driver account and the deliveries assigned to them. They cannot see vendor data, order details beyond their delivery, or customer account information.

If a Vendor Admin user sees the wrong restaurant data, confirm the active vendor or branch context. If a driver cannot see expected work, confirm onboarding completion, sign-in state, and branch assignment. If an admin user should no longer have access, remove the admin account through the Users section of System Admin.

## How accounts are created

All portals use Entra ID (Azure Active Directory) for authentication.

- System Admin accounts are created directly by an existing System Admin user from the Users list page.
- Vendor Admin accounts are created through the vendor onboarding journey. When System Admin approves an application, an invitation email is sent to the applicant. The applicant clicks the link, completes registration on the Vendor Admin onboarding pages, and their account is created.
- Driver accounts are created through the driver invitation flow. A Vendor Admin user invites the driver from the Drivers list. The driver receives an email, follows the link to Driver Portal, completes registration, and their account is created.
- Customer accounts are self-service via the Customer Front-end.

## How to reach the right portal

Each portal runs on its own URL. If you are signed in but cannot see content you expect, confirm you are in the correct portal. Context switching (for Vendor Admin users who manage multiple vendors or branches) happens through the context selector within Vendor Admin — not by opening a different URL.

## What the AI Copilot can and cannot do

The Copilot in System Admin can answer questions about platform operations, vendor management, commissions, invoicing, content management, promotions, reviews, tools, and cross-platform workflows.

The Copilot in Vendor Admin can answer questions about menus, orders, drivers, promotions, delivery costs, payment methods, reviews, and vendor-facing workflows.

The Copilot cannot make changes to data, place orders, approve applications, or perform any write operation. It is a read-only AI knowledge assistant.
