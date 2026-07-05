# Azure Monthly Running Cost Analysis — WantFood

> **Last updated:** July 2026
>
> All prices are approximate UK South region (GBP). Actual costs vary by region,
> reservation commitments, and consumption patterns.
>
> **Key Note:** This analysis splits **dev/staging** (RabbitMQ + Elasticsearch) from 
> **production** (Azure Service Bus + Azure AI Search). Migration to production equivalents
> occurs during Azure deployment phase.

---

## 1. Architecture Summary

| Component              | Dev/Staging (Local via Aspire)        | Production (Azure)                        |
|------------------------|---------------------------------------|-------------------------------------------|
| **Databases (6×)**     | SQL Server container                  | Azure SQL                                 |
| **Blob Storage**       | Azurite emulator                      | Azure Blob Storage                        |
| **Cache**              | Redis container                       | Azure Managed Redis (`redisEnterprise`)   |
| **Messaging**          | RabbitMQ (MassTransit)                | Azure Service Bus (MassTransit)           |
| **Search**             | Elasticsearch                         | Azure AI Search (Cognitive Search)        |
| **Email**              | SMTP4Dev                              | Azure Communication Services / SendGrid   |
| **Real-time**          | SignalR + Redis backplane             | Azure SignalR Service                     |
| **Functions**          | Local Functions host                  | Azure Functions (Consumption)             |
| **Compute**            | Local Aspire                          | Azure Container Apps                      |
| **Monitoring**         | Aspire Dashboard                      | Azure Monitor / Application Insights      |

### Services Inventory (15 container apps + 1 Azure Function)

| #  | App                                   | Type              |
|----|---------------------------------------|-------------------|
| 1  | `wantfood-api-orderservice`           | Backend API (saga)|
| 2  | `wantfood-api-paymentservice`         | Backend API       |
| 3  | `wantfood-api-deliveryservice`        | Backend API       |
| 4  | `wantfood-api-searchservice`          | Backend API       |
| 5  | `wantfood-api-notificationservice`    | Backend API       |
| 6  | `wantfood-api-userservice`            | Backend API       |
| 7  | `wantfood-api-vendorservice`          | Backend API       |
| 8  | `wantfood-api-fileservice`            | Backend API       |
| 9  | `wantfood-api-fileservice-functions`  | Azure Functions   |
| 10 | `wantfood-web-notificationhub`        | SignalR hub       |
| 11 | `wantfood-web-bff`                    | BFF gateway       |
| 12 | `wantfood-web-frontend`               | Customer portal   |
| 13 | `wantfood-web-vendoradmin`            | Vendor portal     |
| 14 | `wantfood-web-systemadmin`            | Admin portal      |
| 15 | `wantfood-web-driverportal`           | Driver portal     |

### Message Queues (MassTransit Endpoints)

All queues are prefixed with `queue-` per project conventions.

| Service              | Queues                                                                                                                                                  |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **OrderService**     | `queue-order-state` (saga)                                                                                                                              |
| **PaymentService**   | `queue-payment-authorize`, `queue-payment-capture`, `queue-payment-void`, `queue-payment-refund`, `queue-payment-tip`                                   |
| **DeliveryService**  | *(publishes to `queue-order-state`)*                                                                                                                    |
| **SearchService**    | `queue-search-index`, `queue-menu-status`                                                                                                               |
| **VendorService**    | `queue-restaurant-notify`, `queue-vendor-payment-credentials`, `queue-image-uploaded`                                                                   |
| **NotificationService** | `queue-driver-invitation-email`, `queue-vendor-approval-email`, `queue-vendor-rejection-email`, `queue-order-status-update`, `queue-vendor-new-order-notification`, `queue-vendor-driver-self-assigned`, `queue-driver-assignment-push` |
| **NotificationHub**  | `queue-deliver-web-notification`                                                                                                                        |
| **FileService**      | `queue-image-processing`                                                                                                                                |

**Total: ~20 queues.** MassTransit also creates topics/subscriptions for pub-sub routing.

---

## 1.5. Dev/Staging vs Production Infrastructure

### Messaging Bus Strategy

**Dev/Staging** uses **RabbitMQ** (via MassTransit) running on local Aspire or in a container. This is lightweight, free, and sufficient for development.

**Production** uses **Azure Service Bus** (Standard tier, required for MassTransit topics). See §7 for cost analysis.

| Phase          | Solution           | Cost/mo | Trade-off                              |
|----------------|--------------------|---------|----------------------------------------|
| **Local dev**  | RabbitMQ container | £0      | No SLA, local-only                     |
| **Staging**    | RabbitMQ ACA       | ~£3–5   | Low cost, acceptable SLA for staging   |
| **Production** | Service Bus Standard | £8     | Managed, HA, required for topics       |

### Search Engine Strategy

**Dev/Staging** uses **Elasticsearch** (via container/Aspire). This is free for development but requires significant compute/memory.

**Production** uses **Azure AI Search** (Basic tier minimum, required for production search at scale). See §8 for cost analysis.

| Phase          | Solution              | Cost/mo | Trade-off                              |
|----------------|-----------------------|---------|----------------------------------------|
| **Local dev**  | Elasticsearch container | £0      | No cost, local-only, high resource use |
| **Staging**    | Elasticsearch ACA     | ~£20–30 | Acceptable cost for staging validation |
| **Production** | Azure AI Search Basic | £57     | Managed, fully-featured search API     |
| **Future**     | Azure OpenAI Search   | ~£60–100| Semantic / hybrid search when mature   |

---

## 2. Assumptions

| Parameter                        | Value                                              |
|----------------------------------|----------------------------------------------------|
| Restaurants                      | 20,000                                             |
| Dishes per restaurant (max)      | 100                                                |
| Image size (max)                 | 1 MB each                                          |
| Traffic tiers                    | 100 / 1,000 / 10,000 users per day                |
| Page requests per user per day   | 10                                                 |
| Region                           | UK South                                           |
| Currency                         | GBP (£)                                            |

---

## 3. Blob Storage (Dish Images)

### Storage Volume

| Scenario                                | Images       | Total Size   |
|-----------------------------------------|--------------|--------------|
| Worst case (all dishes, 1 MB each)      | 2,000,000    | **~2 TB**    |
| Realistic (50% fill, avg 500 KB each)   | 1,000,000    | **~500 GB**  |

### Monthly Costs (LRS, Hot Tier)

| Cost Item                  | Rate              | 500 GB    | 2 TB       |
|----------------------------|-------------------|-----------|------------|
| Storage                    | £0.0156/GB/mo     | £7.80     | £31.20     |
| Read ops (100 users/day)   | £0.0033/10K ops   | ~£0.02    | ~£0.02     |
| Read ops (1,000 users/day) |                    | ~£0.17    | ~£0.17     |
| Read ops (10,000 users/day)|                    | ~£1.65    | ~£1.65     |
| Bandwidth (without CDN)    | £0.067/GB         | see below | see below  |

### ⚡ CDN Recommendation

At 10,000 users × 10 pages × 5 images × 200 KB average = **~100 GB/day egress**.

| Scenario         | Without CDN     | With CDN (Azure Front Door) |
|------------------|-----------------|-----------------------------|
| 100 users/day    | ~£4/mo          | ~£1/mo                      |
| 1,000 users/day  | ~£40/mo         | ~£3–£5/mo                   |
| 10,000 users/day | ~£200/mo        | ~£6–£13/mo                  |

> **Strongly recommended:** Place a CDN in front of blob storage for image delivery.

---

## 4. Compute — Azure Container Apps

ACA Consumption plan pricing: ~£0.000018/vCPU-s + ~£0.000002/GiB-s.

**Free grant:** 180,000 vCPU-s + 360,000 GiB-s/mo (~50 vCPU-hrs + 100 GiB-hrs).

| Traffic Tier     | Active vCPU-hrs/mo | Active GiB-hrs/mo | Monthly Cost     |
|------------------|--------------------|--------------------|------------------|
| 100 users/day    | ~100 hrs           | ~200 hrs           | **~£5–£10**      |
| 1,000 users/day  | ~400 hrs           | ~800 hrs           | **~£30–£50**     |
| 10,000 users/day | ~2,000 hrs         | ~4,000 hrs         | **~£140–£200**   |

### Azure Functions (FileService Functions — Consumption Plan)

| Item                   | Free Grant           | Estimated Cost |
|------------------------|----------------------|----------------|
| Executions             | 1M/month free        | **£0**         |
| Compute (GB-s)         | 400,000 GB-s free    | **£0**         |

Image processing triggers are low-volume; Consumption plan is more than sufficient.

---

## 5. Databases — Azure SQL (6 Databases)

Databases: `UserService`, `VendorService`, `OrderService`, `PaymentService`, `DeliveryService`, `NotificationService`.

### Pricing Options

| Option                    | Min Cost/DB/mo | 6 DBs/mo     | Notes                                                        |
|---------------------------|----------------|---------------|--------------------------------------------------------------|
| **Free offer**            | **£0**         | N/A           | 1 per subscription only. 32 GB, 100 DTU burst.              |
| **Basic DTU (5 DTU)**     | ~£3.77         | ~£22.60       | 2 GB max size. Dev/test only.                                |
| **S0 Standard (10 DTU)**  | ~£11.58        | ~£69.50       | 250 GB. Minimum viable for low-traffic production.           |
| **Serverless GP vCore**   | ~£4.20         | ~£25          | Auto-pauses after idle → £0 compute when paused.             |
| **Basic Elastic Pool**    | ~£11.60 total  | **~£11.60**   | 50 eDTU shared across all 6 databases.                       |
| **Standard Elastic Pool** | ~£56 total     | **~£56**      | 50 eDTU, suitable for 10K users.                             |

### Recommended Strategy

| Environment               | Configuration                                  | Monthly Cost  |
|---------------------------|------------------------------------------------|---------------|
| **Dev/Test**              | 1 × Free DB + 5 × Basic DTU                   | **~£19**      |
| **Dev/Test (auto-pause)** | 6 × Serverless vCore with auto-pause           | **~£0 idle**  |
| **Prod (100–1K users)**   | Basic Elastic Pool (all 6 DBs)                 | **~£12**      |
| **Prod (10K users)**      | Standard Elastic Pool (50 eDTU)                | **~£56**      |

---

## 6. Azure Managed Redis

> **Replaces** the retiring **Azure Cache for Redis** (`Microsoft.Cache/redis`).
> All new deployments must use **Azure Managed Redis** (`Microsoft.Cache/redisEnterprise`).
> See: <https://aka.ms/AzureCacheForRedisRetirement>.

| Tier              | Monthly  | Cache   | Notes                                                        |
|-------------------|---------|---------|--------------------------------------------------------------|
| **Balanced_B0**   | ~£42–£55 | 0.5 GB  | Cheapest Managed Redis tier. Single node, no HA.             |
| **Balanced_B1**   | ~£85    | 1 GB    | Adds replication / HA.                                       |
| **MemoryOptimized_M10** | ~£170 | 12 GB | Production at scale.                                          |

There is no Free or Basic tier for Managed Redis. Entra-ID-only auth (`accessKeysAuthentication: Disabled`) is enforced via the access-policy model on each database.

### Cheaper alternatives for dev/staging

| Option                                   | Monthly  | Trade-off                                                          |
|------------------------------------------|---------|---------------------------------------------------------------------|
| **Redis on Azure Container Apps**        | ~£0–£3  | Public Redis image, scale-to-zero, no SLA. Acceptable for staging.  |
| **In-memory caching only (no Redis)**    | £0      | Acceptable only if all services run with `replicas: 1` and you tolerate cache-miss bursts after revision rollouts. Breaks the SignalR Redis backplane (use Azure SignalR Service instead). |
| **Azure Managed Redis Balanced_B0**      | ~£42–£55 | Recommended for prod; only option for an HA-ish managed cache.       |

**Recommendation:** Run Redis on ACA for dev/staging (saves ~£50/mo); switch to Managed Redis `Balanced_B0` (or `Balanced_B1` once HA matters) for production.

---

## 7. Azure Service Bus (Replacing RabbitMQ)

> **⚠️ Important:** MassTransit uses **Topics + Subscriptions** for pub-sub routing.
> The **Basic tier does NOT support topics**. You **must use Standard tier** for MassTransit.

### Pricing

| Tier         | Monthly Base | Included Ops | Per-message overage |
|--------------|-------------|--------------|---------------------|
| **Basic**    | ~£0.038     | 1M ops       | ❌ No topics — **will not work with MassTransit** |
| **Standard** | ~£7.60      | 12.5M ops    | £0.038/1M           |

### Estimated Monthly Cost (Standard Tier)

| Traffic Tier      | Est. Messages/mo | Monthly Cost |
|-------------------|------------------|--------------|
| 100 users/day     | ~50K             | **~£8**      |
| 1,000 users/day   | ~500K            | **~£8**      |
| 10,000 users/day  | ~5M              | **~£8**      |

All within the included 12.5M operations per month at Standard tier.

---

## 8. Azure AI Search (Replacing Elasticsearch)

20,000 restaurants with menu data to index.

| Tier             | Monthly      | Storage  | Indexes | Notes                                         |
|------------------|-------------|----------|---------|-----------------------------------------------|
| **Free**         | **£0**      | 50 MB    | 3       | ❌ Insufficient for 20K restaurants.           |
| **Basic**        | ~£57        | 2 GB     | 15      | ✅ Sufficient for 20K restaurants + menus.     |
| **Standard S1**  | ~£190       | 25 GB    | 50      | Needed for semantic search / complex facets.   |

> **💡 Budget alternative:** Use **Azure SQL Full-Text Search** for vendor/menu search at
> low scale and defer AI Search until revenue justifies £57–£190/mo.

---

## 9. Azure Monitor & Application Insights

| Component                      | Free Tier            | 100 users/day | 1K users/day | 10K users/day |
|--------------------------------|----------------------|---------------|--------------|---------------|
| **App Insights (data)**        | First 5 GB/mo free   | **£0**        | ~£5–£15      | ~£30–£80      |
| **Ingestion (after 5 GB)**     | £2.14/GB             | —             | —            | —             |
| **Standard Metrics**           | Free                 | **£0**        | **£0**       | **£0**        |
| **Alert Rules**                | 10 rules free        | **£0**        | **£0–£1**    | **£0–£1**     |
| **Log Analytics Workspace**    | 5 GB/mo free         | included      | included     | included      |
| **Container Apps built-in**    | Routes to Log Analytics | included   | included     | included      |

> **💡 Tip:** Set sampling rate in Application Insights to 20–50% in production to control
> log ingestion costs. Aspire Service Defaults already wire up OpenTelemetry — this routes
> to Azure Monitor automatically when deployed to ACA.

---

## 10. Security Services (Not Yet Configured with Aspire)

| Service                          | Purpose                              | Monthly Cost                               |
|----------------------------------|--------------------------------------|--------------------------------------------|
| **Entra ID (Free)**              | Auth for drivers, vendors, admins    | **£0** (Free tier: 50K MAU)                |
| **Entra ID B2C**                 | Customer sign-up/sign-in             | **£0** first 50K auths/mo, £0.0022 after   |
| **Azure Key Vault**              | Stripe keys, DB passwords, API keys  | £0.024/10K ops → **~£0–£1/mo**             |
| **Azure DDoS Protection**        | DDoS mitigation                      | **⚠️ ~£2,340/mo** — skip, use free Basic   |
| **Azure WAF (on Front Door)**    | Web Application Firewall             | ~£7.60/mo + £0.57/M requests               |
| **Defender for Cloud**           | Security posture management          | **Free CSPM tier**; paid ~£11/server/mo     |
| **Managed Identity**             | Service-to-service auth (no secrets) | **£0**                                     |

**Recommendation:** Start with Entra ID Free + Key Vault + Managed Identities + Defender Free
tier. Skip DDoS Standard and WAF until traffic and revenue justify it.

---

## 11. Azure SignalR Service

Used by `wantfood-web-notificationhub` with Redis backplane.

| Tier                   | Monthly     | Connections       | Messages        |
|------------------------|-------------|-------------------|-----------------|
| **Free**               | **£0**      | 20 concurrent     | 20K msgs/day    |
| **Standard (1 unit)**  | ~£37.45     | 1,000 concurrent  | 1M msgs/day     |

---

## 12. Azure Communication Services (Replacing SMTP4Dev)

Email notifications: order status, vendor approvals, driver invitations.

| Traffic Tier      | Est. Emails/mo | Rate              | Monthly Cost |
|-------------------|----------------|-------------------|--------------|
| 100 users/day     | ~1,000         | £0.00020/email    | **~£0.20**   |
| 1,000 users/day   | ~10,000        | £0.00020/email    | **~£2**      |
| 10,000 users/day  | ~100,000       | £0.00020/email    | **~£20**     |

---

## 13. Azure Container Registry

Required to store container images for ACA deployments.

| Tier         | Monthly | Storage  | Notes              |
|--------------|---------|----------|--------------------|
| **Basic**    | ~£4     | 10 GB    | Sufficient for dev |
| **Standard** | ~£16    | 100 GB   | Production         |

---

## 14. Complete Monthly Cost Summary

| Component                     | 100 users/day | 1,000 users/day | 10,000 users/day |
|-------------------------------|---------------|------------------|-------------------|
| **ACA Compute** (14 apps)     | £5–£10        | £30–£50          | £140–£200         |
| **Azure Functions**           | £0            | £0               | £0–£1             |
| **Azure SQL** (6 DBs)         | £0–£19 ¹      | £12–£25 ²        | £56–£70 ³         |
| **Azure Managed Redis**       | £2 ⁴ / £42    | £42–£55          | £85               |
| **Azure Service Bus**         | £8            | £8               | £8                |
| **Azure AI Search**           | £0 ¹ / £57    | £57              | £57–£190          |
| **Blob Storage** (500 GB)     | £8            | £8               | £10               |
| **CDN / Front Door**          | £5            | £7               | £13–£25           |
| **Azure Monitor / App Insights** | £0         | £5–£15           | £30–£80           |
| **Azure SignalR**             | £0 ¹          | £37              | £37               |
| **Key Vault**                 | £0            | £0               | £1                |
| **Entra ID**                  | £0            | £0               | £0                |
| **Communication Services**    | £0            | £2               | £20               |
| **Container Registry**        | £4            | £4               | £16               |
| | | | |
| **TOTAL (prod-ready)**        | **~£70–£155** | **~£210–£270**   | **~£440–£650**    |
| **TOTAL (staging, max savings)** ⁴ | **~£15–£30** | —                | —                 |

> ¹ Using free tiers where available (1 free SQL DB, free AI Search subset, free SignalR). Note: Managed Redis has no free tier — use Redis on ACA for dev/staging if you need £0.
>
> ² Basic Elastic Pool
>
> ³ Standard Elastic Pool or individual S0 databases
>
> ⁴ Redis-on-ACA in place of Managed Redis (~£2/mo vs £42–£55/mo). No SLA — acceptable for staging only.

---

## 15. ⚠️ Scale-Down Blockers

| Component               | Scales to Zero? | Min Always-On Cost | Mitigation                                    |
|-------------------------|-----------------|---------------------|-----------------------------------------------|
| **Azure AI Search**     | ❌ **No**       | £57/mo (Basic)      | Defer to SQL Full-Text Search at low scale    |
| **Azure SQL (DTU)**     | ❌ No           | £3.77/db/mo (Basic) | Use Serverless vCore with auto-pause          |
| **Azure SQL (Serverless)** | ✅ Auto-pause | £0 when paused      | ⚠️ 60s cold-start on resume                  |
| **Azure Managed Redis** | ❌ **No**       | £42–£55/mo (Balanced_B0) | Use Redis on ACA for dev/staging (no SLA)     |
| **Azure SignalR**       | ❌ No (Standard)| £37/mo              | Free tier (20 connections) for dev            |
| **Azure Service Bus**   | ✅ Pay-per-msg  | £7.60/mo base       | Unavoidable for MassTransit topic support     |
| **ACA apps**            | ✅ Scale to zero| £0 when idle        | ✅ No blocker                                 |
| **Azure Functions**     | ✅ Scale to zero| £0 (Consumption)    | ✅ No blocker                                 |
| **Container Registry**  | ❌ Always on    | £4/mo (Basic)       | Required for ACA deployments                  |

> **Minimum cost floor** even with zero users (production-ready stack):
> AI Search (£57) + Managed Redis Balanced_B0 (£42) + Service Bus (£8) + SQL Elastic Pool (£12) + ACR (£4) = **~£123/mo**
>
> **Staging-floor variant** (Redis-on-ACA, no AI Search, Serverless SQL paused):
> Service Bus (£8) + ACR (£4) + ACA Redis (£2) = **~£14/mo**

---

## 16. Dev/Test Environment Recommendations

| Strategy                                       | Monthly Saving |
|------------------------------------------------|----------------|
| Azure SQL Free offer (1 DB) + Basic DTU (5)    | ~£40           |
| Serverless auto-pause for all databases        | ~£20 (but cold starts) |
| Redis on ACA (replaces retired Free tier)      | ~£42 vs Managed Redis Balanced_B0 |
| AI Search Free (test with small data subset)   | ~£57           |
| SignalR Free                                   | ~£37           |
| ACA naturally scales to zero                   | Already free   |
| Consumption-plan Functions                     | Already free   |
| **Dev/test total**                             | **~£10–£20/mo** |

Primary costs in dev/test are Service Bus Standard (£8) + blob storage + any non-free DB tiers.

---

## 17. Notes & Recommendations

1. **CDN is critical** — without it, image bandwidth at scale will dominate costs.
2. **Service Bus Standard is mandatory** — MassTransit requires topics; Basic tier will not work.
3. **AI Search is the most expensive always-on service** — consider deferring until needed.
4. **Managed Identities** should replace connection strings wherever possible to avoid
   secret management overhead and improve security posture.
5. **Application Insights sampling** should be configured to 20–50% in production to
   control log ingestion costs while retaining diagnostic capability.
6. **BFF cold-start risk** — the BFF is called by all frontends. If ACA scales it to zero,
   the first request after idle will have ~2–5s latency. Consider `minReplicas: 1` for BFF.
7. **NotificationHub cold-start risk** — requires persistent WebSocket connections.
   Cannot effectively scale to zero if users have the app open. Keep `minReplicas: 1`.
