# WantFood Microservices Architecture Guide

## 1. System Overview

WantFood is a distributed microservices food delivery platform built on .NET with Azure Container Apps, Azure Functions, and event-driven MassTransit orchestration. The architecture follows domain-driven design (DDD) principles with clear service boundaries and asynchronous communication patterns.

### Key Architectural Principles

- **BFF as Single Entry Point:** All external traffic routes through Backend-for-Frontend (BFF), the only public-facing service
- **Async-First Design:** Event-driven communication via MassTransit with RabbitMQ (dev/staging) and Azure Service Bus (production)
- **Domain-Driven Services:** Each microservice owns its domain model and database
- **Azure Functions for Specialized Tasks:** File processing, user provisioning, background jobs
- **Managed Identity Authentication:** All service-to-service calls use Azure Managed Identity (MSI), never API keys or connection strings
- **Search as Aggregate Engine:** Denormalized search service with intent-based querying via Azure AI Search

---

## 2. Microservices Catalog

### 2.1 Order Service (Critical Path)

**Responsibilities:** Order creation, state management, saga orchestration

**Technology Stack:**
- Container App or REST API in Container Apps
- PostgreSQL database (isolated)
- MassTransit Saga State Machine for order lifecycle
- Service Bus / RabbitMQ for event publishing

**Order Processing Saga:**

The order lifecycle is managed as a MassTransit state machine with the following states:

```
Created → Acknowledged → PaymentProcessing → Assigned → InDelivery → Completed
   ↓           ↓              ↓              ↓          ↓
[Events Publishing]
- OrderCreatedEvent (publish to Payment Service, Notification Service)
- OrderAcknowledgedEvent (vendor confirms)
- PaymentProcessedEvent (Payment Service publishes)
- OrderAssignedEvent (assign to delivery driver)
- OrderInDeliveryEvent (notify customer)
- OrderDeliveredEvent (mark complete, trigger analytics)
```

**Saga State Machine Example (C#):**

```csharp
public class OrderStateMachine : MassTransitStateMachine<OrderState>
{
    public State Created { get; private set; }
    public State Acknowledged { get; private set; }
    public State PaymentProcessing { get; private set; }
    public State Assigned { get; private set; }
    public State InDelivery { get; private set; }
    public State Completed { get; private set; }

    public Event<OrderCreatedEvent> OrderCreated { get; private set; }
    public Event<PaymentCompletedEvent> PaymentCompleted { get; private set; }
    public Event<DriverAssignedEvent> DriverAssigned { get; private set; }
    public Event<DeliveryCompletedEvent> DeliveryCompleted { get; private set; }

    public override void Configure()
    {
        InstanceState(x => x.CurrentState);

        Event(() => OrderCreated, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => PaymentCompleted, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => DriverAssigned, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => DeliveryCompleted, x => x.CorrelateById(m => m.Message.OrderId));

        Initially(
            When(OrderCreated)
                .Then(context => context.Instance.OrderId = context.Data.OrderId)
                .Then(context => context.Instance.VendorId = context.Data.VendorId)
                .TransitionTo(Created)
                .Publish(context => new OrderAcknowledgedEvent(context.Instance.OrderId))
        );

        During(Created,
            When(PaymentCompleted)
                .TransitionTo(PaymentProcessing)
                .Publish(context => new PaymentProcessedEvent(context.Instance.OrderId))
        );

        During(PaymentProcessing,
            When(DriverAssigned)
                .TransitionTo(Assigned)
        );

        During(Assigned,
            When(DeliveryCompleted)
                .TransitionTo(InDelivery)
                .TransitionTo(Completed)
                .Publish(context => new OrderDeliveredEvent(context.Instance.OrderId))
                .Finalize()
        );
    }
}
```

**Key Consumers:**
- `OrderCreatedConsumer` - Publishes to Payment Service and Notification Service
- `PaymentCompletedConsumer` - Transitions to delivery phase, publishes driver assignment request
- `DriverAssignedConsumer` - Updates order with driver details
- `DeliveryCompletedConsumer` - Finalizes order, triggers analytics

**API Endpoints:**
- `POST /api/orders` - Create order (BFF calls this)
- `GET /api/orders/{orderId}` - Get order details
- `GET /api/orders?status=pending` - Query orders by status
- `PUT /api/orders/{orderId}/acknowledge` - Vendor acknowledges order

### 2.2 Payment Service

**Responsibilities:** Payment processing, refunds, payment reconciliation

**Technology Stack:**
- Container App REST API
- PostgreSQL database (isolated)
- Third-party payment gateway integration (Stripe/PayPal/Square)
- MassTransit event publishing

**Event Flow:**

```
BFF sends CreatePaymentCommand
         ↓
Payment Service validates payment
         ↓
Calls external gateway (Stripe API)
         ↓
On success: publishes PaymentCompletedEvent → Order Service saga picks it up
         ↓
On failure: publishes PaymentFailedEvent → Order Service rolls back
```

**Critical Patterns:**
- **Idempotency:** All payment operations include `IdempotencyKey` to prevent duplicate charges
- **Webhook Handling:** Stripe webhooks asynchronously confirm payment status
- **Retry Logic:** Exponential backoff for transient failures, Circuit Breaker pattern for persistent failures
- **PCI Compliance:** Never touches card details—all handled by payment gateway

**API Endpoints:**
- `POST /api/payments` - Initiate payment
- `GET /api/payments/{paymentId}` - Get payment status
- `POST /api/payments/{paymentId}/refund` - Process refund
- `POST /webhook/stripe` - Webhook receiver for Stripe events

### 2.3 Vendor Service

**Responsibilities:** Restaurant/vendor management, menu management, restaurant settings

**Technology Stack:**
- Container App REST API
- PostgreSQL database (isolated)
- Integration with File Service for menu images
- ElasticSearch/AI Search for vendor search queries

**Key Entities:**
- Vendor (restaurant profile, location, operating hours)
- Menu (catalog of food items with pricing, images, dietary tags)
- VendorSettings (commission rate, delivery zones, payment method)
- Reviews and Ratings

**API Endpoints:**
- `POST /api/vendors` - Create vendor account
- `GET /api/vendors/{vendorId}` - Get vendor profile
- `GET /api/vendors/{vendorId}/menu` - Get menu items
- `PUT /api/vendors/{vendorId}/menu/{itemId}` - Update menu item
- `POST /api/vendors/{vendorId}/images` - Upload menu images (calls File Service)

### 2.4 User Service

**Responsibilities:** User profiles, authentication metadata, user preferences

**Technology Stack:**
- Container App REST API
- PostgreSQL database (isolated)
- Azure Functions for async provisioning (sign-up workflows)
- Integration with Entra ID for identity

**Key Entities:**
- UserProfile (Entra ID Object ID, display name, avatar)
- UserPreferences (dietary restrictions, favorite restaurants, delivery address book)
- UserRoles (mapped from Entra ID group membership and database claims)

**API Endpoints:**
- `POST /api/users` - Create user profile (called by sign-up flow)
- `GET /api/users/{userId}` - Get user profile
- `PUT /api/users/{userId}` - Update profile
- `GET /api/users/{userId}/preferences` - Get user preferences
- `POST /api/users/{userId}/addresses` - Add delivery address

**Azure Function Integration (`UserService.Functions`):**
- `OnUserCreatedFunction` - Triggered when Entra ID user signs up
  - Creates database profile
  - Initializes preferences
  - Sends welcome email via Notification Service

### 2.5 Delivery Service

**Responsibilities:** Driver management, delivery assignment, route optimization

**Technology Stack:**
- Container App REST API
- PostgreSQL database (isolated)
- Azure Maps API for geolocation and routing
- Event publishing for delivery status updates

**Key Entities:**
- Driver (profile, vehicle, current location)
- DeliveryAssignment (order-to-driver mapping, delivery route)
- DeliveryMetrics (delivery time, distance, driver rating)

**Delivery Assignment Algorithm:**
- Filters drivers by availability and proximity to order location
- Considers current delivery queue (prevent overloading)
- Optimizes for delivery time (using Azure Maps routing)
- Publishes `DriverAssignedEvent` to Order Service saga

**API Endpoints:**
- `POST /api/drivers` - Register driver
- `GET /api/drivers/{driverId}/active-orders` - Current delivery queue
- `POST /api/deliveries` - Create delivery assignment
- `PUT /api/deliveries/{deliveryId}/status` - Update delivery status (in progress, completed, failed)
- `GET /api/deliveries/{deliveryId}/route` - Get optimized delivery route

### 2.6 Search Service (AI-Powered Intent-Based Search)

**Responsibilities:** Full-text search across restaurants/menus, intent-based natural language queries

**Technology Stack:**
- Container App REST API
- **Storage:** Azure AI Search (production) / Elasticsearch (dev/staging)
- Denormalized search index combining Vendor + Menu data
- Azure OpenAI for intent/semantic understanding

**Search Capabilities:**

#### 2.6.1 Full-Text Search
```
User Query: "pizza restaurants near downtown"
Search Service queries AI Search:
- Filter: cuisine contains "pizza"
- Filter: location within 5km of "downtown"
- Result: Ranked by distance and rating
```

#### 2.6.2 Intent-Based AI Search
```
User Query (natural language): "I'm craving spicy Thai food, budget-friendly, open now"

Azure OpenAI extracts intent:
{
  "cuisine": "Thai",
  "taste": "spicy",
  "priceRange": "budget",
  "availability": "open_now"
}

Search Service applies semantic search:
1. Matches restaurants tagged with Thai cuisine
2. Filters by operating hours (open_now = true)
3. Matches menu items with spice level tags
4. Filters by average item price
5. Re-ranks by semantic similarity using embedding models
```

**Index Structure (Denormalized):**

```json
{
  "vendorId": "v123",
  "vendorName": "Bangkok Express",
  "cuisineType": ["Thai", "Asian"],
  "cuisine_text": "Thai Asian fusion authentic",
  "location": { "latitude": 40.7128, "longitude": -74.0060 },
  "averageRating": 4.5,
  "minDeliveryTime": 25,
  "items": [
    {
      "itemId": "m456",
      "itemName": "Pad Thai",
      "spiceLevel": 3,
      "price": 12.99,
      "dietary": ["vegan", "gluten-free"],
      "itemDescription": "Traditional stir-fried noodles with tamarind, lime, and crushed peanuts"
    }
  ]
}
```

**Index Synchronization:**

```
Vendor Service → publishes VendorUpdatedEvent
Search Service consumer:
- Fetches updated vendor from Vendor Service
- Fetches menu items
- Rebuilds search index entry
- Publishes to AI Search

Menu Item updated:
- VendorService publishes MenuItemUpdatedEvent
- Search Service updates corresponding search entry
- AI Search index re-indexed in near-real-time
```

**API Endpoints:**
- `GET /api/search?q=pizza&location=downtown` - Full-text search
- `POST /api/search/intent` - Intent-based AI search
  ```
  Request: { "query": "spicy Thai food near me, budget-friendly" }
  Response: [
    { "vendorId": "v1", "vendorName": "Bangkok Express", "score": 0.95 },
    { "vendorId": "v2", "vendorName": "Pad Krapow House", "score": 0.87 }
  ]
  ```

### 2.7 Notification Service

**Responsibilities:** Multi-channel notifications (email, SMS, push)

**Technology Stack:**
- Container App REST API
- PostgreSQL database for notification templates and delivery logs
- Azure Communication Services (ACS) for email/SMS
- SignalR via NotificationHub for real-time push
- MassTransit event consumers for async notification handling

**Notification Templates:**

```
OrderCreatedEvent → "Order Confirmation" template
  - To: Customer email
  - Template: "Your order #{orderId} confirmed at {vendorName}"

DriverAssignedEvent → "Driver Assigned" template
  - To: Customer (SMS + push via SignalR)
  - Template: "Driver {driverName} is on the way, ETA {eta}"

DeliveryCompletedEvent → "Order Delivered" template
  - To: Customer (push + email)
  - Template: "Your order delivered by {driverName}, rate your experience"
```

**Consumer Pattern:**

```csharp
public class OrderCreatedConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        var notification = new Notification
        {
            UserId = context.Message.CustomerId,
            TemplateId = "order_created",
            Channel = NotificationChannel.Email,
            Data = new { OrderId = context.Message.OrderId, VendorName = context.Message.VendorName }
        };
        
        await _notificationService.SendAsync(notification);
    }
}
```

**SignalR Integration (Real-Time):**

Notification Hub maintains persistent WebSocket connections with clients:
- Receives events from Notification Service
- Broadcasts to connected clients in real-time
- Clients subscribe to specific order updates: `connection.on("OrderStatusChanged", (orderId, status) => updateUI())`

**API Endpoints:**
- `POST /api/notifications` - Send notification
- `GET /api/notifications/{userId}` - Get notification history
- `POST /api/notifications/{notificationId}/read` - Mark as read

### 2.8 File Service (with Azure Functions)

**Responsibilities:** Image upload/resize, file storage, CDN serving

**Technology Stack:**
- Container App REST API (`WantFood.Api.FileService`)
- Azure Blob Storage for file persistence
- Azure Functions (`WantFood.Api.FileService.Functions`) for async image processing
- Azure CDN for serving images

**Upload Workflow:**

```
1. Client uploads image to BFF
2. BFF calls FileService POST /api/files/upload
3. FileService generates presigned SAS URL pointing to Blob Storage
4. Client uploads directly to Blob Storage
5. Blob Storage triggers Azure Function on upload event
6. Function resizes image (thumbnail, medium, large variants)
7. Function stores resized versions in Blob Storage
8. Function publishes FileProcessedEvent
9. Vendor/User Service listens and updates database with CDN URLs
```

**Azure Function (`ImageProcessingFunction`):**

```csharp
[Function("ProcessUploadedImage")]
public async Task Run(
    [BlobTrigger("uploads/{name}", Connection = "AzureWebJobsStorage")] Stream image,
    [Blob("processed/{name}", FileAccess.Write)] CloudBlockBlob outputBlob,
    string name,
    FunctionContext context)
{
    var logger = context.GetLogger<ImageProcessingFunction>();

    using (var output = new MemoryStream())
    {
        // Resize to thumbnail (100x100)
        var thumb = await ResizeImageAsync(image, 100, 100);
        await _blobClient.UploadAsync($"thumbnails/{name}", thumb);

        // Resize to medium (400x400)
        var medium = await ResizeImageAsync(image, 400, 400);
        await _blobClient.UploadAsync($"medium/{name}", medium);

        // Publish event
        await _bus.Publish(new FileProcessedEvent { FileName = name });
    }
}
```

**API Endpoints:**
- `POST /api/files/upload` - Get upload URL (returns SAS presigned URL)
- `GET /api/files/{fileId}` - Get file metadata and CDN URL
- `DELETE /api/files/{fileId}` - Delete file

### 2.9 Promotion Service

**Responsibilities:** Coupons, discounts, promotional campaigns

**Technology Stack:**
- Container App REST API
- PostgreSQL database
- Time-based event publishing (e.g., daily promo refresh)

**API Endpoints:**
- `GET /api/promotions?vendorId={id}` - Get active promotions for vendor
- `POST /api/promotions/validate` - Validate coupon code at checkout
- `POST /api/promotions` - Create promotion campaign

### 2.10 Chat Service

**Responsibilities:** Customer-to-vendor/driver messaging

**Technology Stack:**
- Container App REST API
- PostgreSQL database for message history
- SignalR via NotificationHub for real-time messaging
- MassTransit for async message delivery

**Real-Time Communication:**
- Clients connect to NotificationHub WebSocket
- Subscribe to chat rooms: `connection.on("ChatRoomMessage:orderId123")`
- Messages published via MassTransit are delivered to NotificationHub
- NotificationHub broadcasts to all connected clients in chat room

### 2.11 Copilot Service

**Responsibilities:** AI-powered customer support, order recommendations

**Technology Stack:**
- Container App REST API
- Azure OpenAI for LLM integration
- Vector database for RAG (Retrieval-Augmented Generation) of restaurant/menu data

**Capabilities:**
- Answer customer questions about restaurants, menus, delivery times
- Provide personalized recommendations based on order history
- Handle support queries and escalations

### 2.12 Content Service

**Responsibilities:** Static content, FAQs, blog posts, terms of service

**Technology Stack:**
- Container App REST API
- PostgreSQL database
- File Service integration for content images

---

## 3. Azure Functions Architecture

Azure Functions provide serverless, event-driven compute for specialized workloads:

### 3.1 OrderService.Functions

**Trigger:** Service Bus Topic subscription (OrderStateMachine events)

**Responsibilities:**
- Long-running order processing workflows
- Order fulfillment tracking (e.g., if order not picked up within 30 minutes, auto-cancel)
- Inventory synchronization with Vendor Service

### 3.2 UserService.Functions

**Trigger:** Service Bus Topic subscription (user.provisioned events)

**Responsibilities:**
- New user onboarding (send welcome email, initialize preferences)
- Bulk user provisioning from Entra ID
- User profile cleanup on account deletion

### 3.3 FileService.Functions

**Trigger:** Azure Blob Storage event (image uploaded)

**Responsibilities:**
- Image resizing and optimization (thumbnail, medium, large variants)
- Virus scanning before storing in production
- CDN cache invalidation after processing

**Function Configuration (local.settings.json):**

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "ServiceBusConnection__fullyQualifiedNamespace": "wantfood-sb.servicebus.windows.net",
    "BlobStorage__blobServiceUri": "https://wantfoodblob.blob.core.windows.net"
  }
}
```

---

## 4. Event-Driven Communication (MassTransit)

### 4.1 Development/Staging (RabbitMQ)

```csharp
services.AddMassTransit(x =>
{
    x.AddConsumersFromNamespaceContaining<OrderCreatedConsumer>();
    
    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("rabbitmq://localhost/");
        cfg.ConfigureEndpoints(context);
    });
});
```

### 4.2 Production (Azure Service Bus)

```csharp
services.AddMassTransit(x =>
{
    x.AddConsumersFromNamespaceContaining<OrderCreatedConsumer>();
    
    x.UsingAzureServiceBus((context, cfg) =>
    {
        cfg.Host(new Uri("sb://wantfood-sb.servicebus.windows.net"));
        cfg.ConfigureEndpoints(context);
    });
});
```

### 4.3 Event Publishing Pattern

```csharp
// Order Service publishes when order is created
await _bus.Publish(new OrderCreatedEvent 
{ 
    OrderId = orderId,
    CustomerId = customerId,
    VendorId = vendorId,
    Items = items,
    TotalAmount = totalAmount,
    Timestamp = DateTimeOffset.UtcNow
});

// Payment Service subscribes
public class OrderCreatedConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        var paymentCommand = new CreatePaymentCommand(context.Message.OrderId, context.Message.TotalAmount);
        await _paymentService.ProcessAsync(paymentCommand);
    }
}
```

---

## 5. Data Flow Diagrams

### 5.1 Order Creation End-to-End

```
Customer (FrontEnd) 
    ↓ POST /api/orders (via BFF)
BFF (Backend for Frontend)
    ↓ Kiota client call
Order Service
    ↓ CreateOrderCommand
    ↓ Saga State Machine: Created
    ↓ Publish OrderCreatedEvent
    ├→ Payment Service (Consumer) → validate payment
    ├→ Notification Service (Consumer) → send confirmation email
    ├→ Search Service (Consumer) → update vendor availability index
    └→ Analytics Service (Consumer) → log event

Payment Service
    ↓ Call Stripe API
    ↓ On success: Publish PaymentCompletedEvent
    └→ Order Service Saga (Receives PaymentCompletedEvent)
       ↓ Transition to PaymentProcessing
       ↓ Publish PaymentProcessedEvent

Order Service Saga (PaymentProcessed)
    ↓ Transition to Assigned
    ↓ Call Delivery Service API to assign driver
    ↓ Publish DriverAssignedEvent
    └→ Notification Service → notify customer of driver assignment
```

### 5.2 Search Execution

```
Customer searches: "spicy Thai food near me"
    ↓
BFF calls Search Service POST /api/search/intent
    ↓
Search Service receives request
    ├→ Extracts intent using Azure OpenAI
    │   (cuisine: Thai, taste: spicy, location: nearby)
    ├→ Queries AI Search index with semantic filters
    └→ Re-ranks results using embedding similarity
    ↓
Returns top 10 matching restaurants
    ↓
FrontEnd displays results with images from CDN
```

### 5.3 Real-Time Notifications

```
Order Service publishes DriverAssignedEvent
    ↓
Notification Service Consumer (via MassTransit)
    ├→ Creates Notification in database
    ├→ Calls Azure Communication Services (SMS to customer)
    └→ Publishes to NotificationHub
    ↓
NotificationHub broadcasts to WebSocket clients
    (All customers subscribed to that order)
    ↓
FrontEnd receives real-time update
    ↓
UI updates with driver info, ETA, live tracking
```

---

## 6. Service-to-Service Communication

All internal service calls use **Managed Identity** for authentication:

```csharp
// Vendor Service calling File Service
var httpClient = new HttpClient();
var token = await _tokenProvider.GetTokenAsync("https://wantfood-file-service.azurecontainerapps.io");
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

var response = await httpClient.PostAsync(
    "https://wantfood-file-service.azurecontainerapps.io/api/files/upload",
    request
);
```

Token is cached in Azure Managed Redis with TTL = token expiry - 5 minutes.

---

## 7. Database Strategy

Each microservice owns its database (no shared DB):

| Service | Database | Size (Est.) | Isolation |
|---------|----------|------------|-----------|
| Order Service | PostgreSQL | Large (order history) | ✓ Isolated DB |
| Payment Service | PostgreSQL | Medium (transactions) | ✓ Isolated DB |
| Vendor Service | PostgreSQL | Medium (menus, vendors) | ✓ Isolated DB |
| User Service | PostgreSQL | Large (user profiles) | ✓ Isolated DB |
| Delivery Service | PostgreSQL | Medium (routes, drivers) | ✓ Isolated DB |
| Search Service | AI Search Index | N/A (denormalized) | ✓ Dedicated index |
| Notification Service | PostgreSQL | Small (templates, logs) | ✓ Isolated DB |

**Sync Strategy:** Events published via MassTransit are consumed by other services to keep their denormalized caches/indices in sync.

---

## 8. Deployment Architecture

### 8.1 Azure Container Apps

Most microservices run as Container Apps with:
- Managed identity for service-to-service auth
- Auto-scaling based on CPU/memory/HTTP request count
- Built-in logging to Application Insights
- Zero-trust networking (no direct internet access except via Front Door)

### 8.2 Azure Functions

Specialized workloads run as Functions with:
- Event-driven triggers (Service Bus, Blob Storage, Timer)
- Consumption plan pricing (pay per execution)
- Seamless integration with MassTransit

### 8.3 Presentation Tier (Separate Container Apps)

- **BFF:** Routes all traffic, enforces policies, aggregates downstream services
- **FrontEnd:** Blazor WebAssembly SPA (runs in browser)
- **VendorAdmin:** Vendor management portal
- **SystemAdmin:** System administration panel
- **DriverPortal:** Driver app/web portal

---

## 9. Security & Isolation

- **BFF as Gatekeeper:** Only public endpoint behind Azure Front Door
- **Managed Identity:** All service-to-service calls via MSI, no API keys in code
- **Network Isolation:** Services in Container Apps Environment, private subnets
- **Secrets:** Azure Key Vault for runtime secrets (never in connection strings)
- **RBAC:** Fine-grained role assignments per service per resource

---

## 10. Monitoring & Observability

### 10.1 Structured Logging

All services use structured logging to Application Insights:

```csharp
_logger.LogInformation(
    "OrderCreated: OrderId={orderId}, VendorId={vendorId}, Amount={amount}",
    orderId, vendorId, amount
);
```

### 10.2 Distributed Tracing

MassTransit automatically correlates events across services:

```
Trace ID: abc123
├─ Order Service (CreateOrder) [0.1s]
├─ Payment Service (ProcessPayment) [1.2s]
├─ Notification Service (SendEmail) [0.5s]
└─ Search Service (UpdateIndex) [0.8s]
Total: 2.6s
```

### 10.3 Custom Metrics

KPI tracking via Application Insights:

```csharp
_telemetryClient.TrackEvent("OrderCompleted", new Dictionary<string, string>
{
    { "VendorId", vendorId.ToString() },
    { "DeliveryTime", deliveryTime.TotalMinutes.ToString() }
});
```

---

## 11. Resilience Patterns

### 11.1 Circuit Breaker (for external APIs)

```csharp
services.AddHttpClient<StripeClient>()
    .AddPolicyHandler(Policy.Handle<HttpRequestException>()
        .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
        .CircuitBreakerAsync(handledEventsAllowedBeforeBreaking: 5, durationOfBreak: TimeSpan.FromSeconds(30))
    );
```

### 11.2 Retry with Exponential Backoff (for MassTransit)

```csharp
cfg.Receive(x =>
{
    x.UseMessageRetry(m => m.Exponential(
        retryLimit: 3,
        initialInterval: TimeSpan.FromSeconds(1),
        intervalIncrement: TimeSpan.FromSeconds(1),
        intervalMultiplier: 2.0
    ));
});
```

### 11.3 Bulkhead Isolation

```csharp
// Limit concurrent calls to external service
services.AddHttpClient<PaymentGatewayClient>()
    .AddPolicyHandler(Policy.BulkheadAsync(maxParallelization: 10, maxQueuingActions: 20));
```

---

## 12. Future Enhancements

1. **GraphQL Gateway:** Consolidate BFF queries into single flexible API layer
2. **API Versioning:** Implement API versioning for breaking changes (semantic versioning)
3. **Event Sourcing:** Consider event sourcing for Order Service to maintain complete audit trail
4. **CQRS:** Separate command (create/update) and query (search) models for complex domains
5. **Distributed Caching:** Redis cache for frequently accessed data (vendor profiles, menus)
6. **API Rate Limiting:** Per-user/per-IP rate limits at Front Door level
7. **Multi-Tenancy:** Support white-label deployments for different markets

