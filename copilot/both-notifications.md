---
audience: Both
title: Notifications — push, SMS, and ACS order chat
---

# Notifications — push, SMS, and ACS order chat

This document covers the three types of notification in WantFood — push notifications, SMS, and ACS order chat — explaining when each fires, who receives it, and how ACS order chat differs from the AI Copilot.

## Notification types

WantFood uses three communication mechanisms:

1. Push notifications — delivered to the customer mobile app or browser at key order state transitions.
2. SMS — sent to customers and vendors for critical alerts when push is not available or not opted into.
3. ACS order chat — a real-time three-way messaging channel between vendor, customer, and driver that is active for the duration of an order.

## Push notifications — when they fire

Push notifications are sent to customers at key order state transitions:
- Order confirmed — sent when the vendor accepts the order.
- Order rejected — sent when the vendor rejects the order.
- Order ready — sent when the vendor marks the order ready for pickup.
- Out for delivery — sent when the driver marks the order as collected and in transit.
- Delivered — sent when the driver marks the order as delivered.
- Order cancelled — sent when an order is cancelled by either party.

Vendors may also receive push notifications when new orders arrive, depending on their notification settings and device.

Drivers receive push or in-app notifications in Driver Portal when they are assigned a new delivery.

## SMS notifications

SMS is used as a fallback when push notifications cannot be delivered, and for invitation-related messages. SMS delivery depends on the notification service configuration and the phone number on record for the recipient.

## ACS order chat

ACS order chat is a Microsoft Azure Communication Services-powered messaging channel. For each active order, a chat thread is created that connects:
- The vendor (visible in Vendor Admin and in any linked mobile interface)
- The customer (visible in the Customer Front-end)
- The driver assigned to the order (visible in Driver Portal)

ACS order chat is used for order-specific communication — for example, the customer asking to substitute an ingredient, or the driver confirming they have arrived. It is not a general support channel.

### How ACS chat differs from the AI Copilot

The ACS order chat is a real-time three-way messaging channel between people involved in a specific order. It has nothing to do with the AI Copilot. The AI Copilot is a knowledge assistant that answers questions about how to use the platform — it does not participate in order conversations and has no visibility into customer or order-specific chat threads.

## What vendors see when a new order arrives

When a customer places an order with a vendor, the order appears in the incoming column of the Vendor Admin orders kanban dashboard. Depending on the vendor's notification settings, they may also receive a push notification or audio alert. The vendor must accept or reject the order within the allowed time window.

## What drivers see when assigned a delivery

When a vendor assigns a driver to an order, the driver receives a notification in Driver Portal and the delivery appears in their Active Deliveries list. The driver can see the pickup address, delivery address, and any relevant order notes.

## What customers see at key state transitions

Customers receive notifications (push or SMS) when:
- Their order is confirmed (vendor accepted)
- Their order is rejected
- Their order is ready for pickup
- Their driver is on the way
- Their order is delivered

After delivery, customers are prompted to submit a review through the Customer Front-end.

## Stale thread cleanup

ACS chat threads for completed or long-inactive orders are cleaned up by a background service that runs on a daily schedule. This removes expired driver participants and closes stale threads to keep the system tidy. This is an automated operation and does not require any action from portal users.
