# Security & Deployment Best Practices — WantFood

> **Last updated:** July 2026
>
> This document outlines Azure security services, networking architecture, secure coding
> practices, and deployment strategies for WantFood. All developers and DevOps engineers
> must follow these guidelines.

---

## 1. Network Architecture & Access Control

### 1.1 BFF as Single External Entry Point

**CRITICAL:** The **Web BFF** is the **only** service exposed to external clients. All client applications (Web FrontEnd, VendorAdmin, SystemAdmin, DriverPortal, future MobileApp) communicate exclusively via the BFF using Kiota-generated clients over HTTPS.

```
┌─────────────────────────────────────────────────────┐
│                   Internet                          │
└──────────────────────────┬──────────────────────────┘
                           │ HTTPS only
                    ┌──────▼──────┐
                    │  Web BFF    │ <- Single entry point
                    │ (Public IP) │   Audience: api://wantfood-bff
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
    ┌───▼────┐        ┌────▼────┐        ┌───▼────┐
    │ User   │        │ Vendor  │        │ Order  │
    │Service │        │Service  │        │Service │
    └────────┘        └─────────┘        └────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
              ┌────────────▼────────────┐
              │   Azure Service Bus     │
              │   (Pub/Sub messaging)   │
              └────────────────────────┘
```

### 1.2 Microservice Isolation (No Direct External Access)

**All microservices are internal-only.** They:
- ❌ Do NOT expose public endpoints
- ❌ Do NOT accept traffic from the internet
- ✅ Listen on internal ports (mapped to container networking only)
- ✅ Communicate via Azure Service Bus for async messaging
- ✅ Are called by the BFF using Kiota with Client Credentials (service-to-service)

### 1.3 Azure Container Apps Networking

**Production Deployment:**

1. **BFF Container App:**
   - External Ingress: **Enabled** (public IP / Azure Front Door)
   - Audience: `api://wantfood-bff`
   - Min replicas: **1** (persistent, cannot scale to zero due to WebSocket backplane and token cache)

2. **All Other Microservices & Notification Hub:**
   - External Ingress: **Disabled** (internal-only)
   - Communication: Via Kiota from BFF or via Azure Service Bus
   - Min replicas: **0** (scale to zero during idle)

3. **Web Frontend / Portals:**
   - Deployment: Azure Static Web Apps or Container Apps (external ingress)
   - API calls: Via BFF only (Kiota client)
   - Authentication: OIDC redirect to Entra ID CIAM

### 1.4 Azure Front Door / CDN (Edge Security)

**Azure Front Door** is the primary security boundary for all external traffic:

#### Configuration
```
┌─────────────────────────────────────────────────────┐
│           Internet (Untrusted)                      │
└──────────────────────────┬──────────────────────────┘
                           │
                    ┌──────▼──────────────┐
                    │  Azure Front Door   │
                    │  - DDoS Protection  │
                    │  - WAF Rules        │
                    │  - TLS Termination  │
                    │  - Rate Limiting    │
                    └──────┬──────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
      ┌─────▼────┐   ┌─────▼────┐   ┌───▼─────┐
      │ BFF      │   │ Static   │   │ Blob    │
      │ (CAE)    │   │ Web Apps │   │ Storage │
      │          │   │ (SWA)    │   │ (CDN)   │
      └──────────┘   └──────────┘   └─────────┘
```

#### Azure Front Door Features

| Feature | Configuration | Purpose |
|---------|---|---|
| **DDoS Protection** | Standard (included) | Mitigates L3/L4 attacks; upgrade to Premium for advanced protection |
| **Web Application Firewall (WAF)** | Enabled | Blocks OWASP Top 10 attacks (SQL injection, XSS, etc.) |
| **Rate Limiting** | 1000 req/5 min per IP | Prevents brute force / API abuse |
| **TLS/SSL** | Minimum TLS 1.2 | All traffic encrypted; auto-renewal via Azure-managed certs |
| **IP Allowlisting** | Stripe webhooks (whitelist) | Only Stripe IPs allowed to `/webhooks/stripe` |
| **Health Probes** | Every 30 sec | Auto-failover if origin unhealthy |
| **Session Affinity** | Enabled for BFF | Route same client to same backend (for token cache coherence) |
| **Request/Response Headers** | Remove server info headers | Hide Azure/ASP.NET version info |

#### WAF Rules (OWASP Managed Rule Set)

```bash
# In Bicep / Terraform
resource wafPolicy 'Microsoft.Network/frontDoorWebApplicationFirewallPolicies@2022-05-01' = {
  name: 'wantfood-waf'
  location: 'Global'
  properties: {
    policySettings: {
      enabledState: 'Enabled'
      mode: 'Prevention' // Block (not Detection)
      requestBodyCheckLimitInKB: 128
    }
    managedRules: {
      managedRuleSets: [
        {
          ruleSetType: 'Microsoft_DefaultRuleSet'
          ruleSetVersion: '2.1'
          ruleGroupOverrides: []
        }
        {
          ruleSetType: 'Microsoft_BotManagerRuleSet'
          ruleSetVersion: '1.0'
        }
      ]
    }
    customRules: {
      rules: [
        {
          name: 'BlockStripWebhooksExcept'
          priority: 1
          ruleType: 'MatchRule'
          action: 'Block'
          matchConditions: [
            {
              matchVariable: 'RequestPath'
              operator: 'BeginsWith'
              matchValue: ['/webhooks/stripe']
              negateCondition: true
            }
          ]
        }
        {
          name: 'RateLimitPerIp'
          priority: 2
          ruleType: 'RateLimitRule'
          action: 'Block'
          rateLimitDurationInMinutes: 5
          rateLimitThreshold: 1000
        }
      ]
    }
  }
}
```

#### IP Allowlisting for Webhooks

```csharp
// PaymentService: Validate incoming webhook is from Stripe
[HttpPost("/webhooks/stripe")]
public async Task<IResult> HandleStripeWebhook(HttpContext context)
{
    var clientIp = context.Connection.RemoteIpAddress?.ToString();
    
    // Stripe publishes docs on webhook IP ranges
    // https://stripe.com/docs/ip-addresses
    var stripeIpRanges = new[] {
        "3.18.0.0/16",
        "3.101.0.0/16",
        "52.84.0.0/16",
        // ... full list from Stripe docs
    };
    
    if (!IsIpInRanges(clientIp, stripeIpRanges))
        return Results.Unauthorized(); // Also logged as security event
    
    // Proceed to webhook signature validation
    ...
}
```

#### CDN Configuration (for static assets & images)

```bash
# Blob Storage → CDN profile
az cdn profile create --resource-group wantfood --name wantfood-cdn --sku Standard_Microsoft

# Origin: Blob Storage endpoint
az cdn endpoint create \
  --resource-group wantfood \
  --profile-name wantfood-cdn \
  --name wantfood-images \
  --origin blob.core.windows.net/images \
  --origin-host-header blob.core.windows.net

# Cache rules
az cdn endpoint rule action add \
  --resource-group wantfood \
  --profile-name wantfood-cdn \
  --endpoint-name wantfood-images \
  --order 1 \
  --action ModifyResponseHeader \
  --header-action Append \
  --header-name Cache-Control \
  --header-value "public, max-age=86400" # 24 hours for images
```

**CDN Security:**
- ✅ Only HTTPS traffic (`require-https`)
- ✅ HSTS header enforced (min 31536000 sec)
- ✅ Query string caching disabled (prevents cache bypass attacks)
- ✅ Blob Storage firewall: Allow CDN origin only

#### Blob Storage Behind CDN (Firewall Config)

```bash
# Only Front Door/CDN can access blob storage
az storage account update \
  --resource-group wantfood \
  --name wantfoodblob \
  --default-action Deny \
  --bypass AzureServices

# Allow only Front Door service tag
az storage account network-rule add \
  --resource-group wantfood \
  --account-name wantfoodblob \
  --service-endpoints AzureFrontDoor.Frontend

# Allow container apps on internal network (if direct access needed)
az storage account network-rule add \
  --resource-group wantfood \
  --account-name wantfoodblob \
  --vnet-name wantfood-vnet \
  --subnet container-apps
```

---

## 1.5 Azure Services Security Configuration

This section documents security settings for each Azure service in the production stack.

### Azure SQL Database

**Encryption & Compliance:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Transparent Data Encryption (TDE)** | Enabled | Encrypts data at rest (AES-256) |
| **Always Encrypted** | Optional (high security) | Column-level encryption for PII |
| **Firewall Rules** | Allow only Container Apps + Front Door | No public internet access |
| **Managed Identity Auth** | Enabled | Token-based auth, no passwords in code |
| **Auditing** | Enabled, 90-day retention | Logs all access, modifications, privilege changes |
| **Threat Detection** | Enabled | Alerts on suspicious patterns (SQL injection, brute force) |
| **Vulnerability Scans** | Weekly | Automatic scanning via Microsoft Defender for SQL |

**Firewall Configuration (Bicep):**

```bicep
resource sqlFirewallRule 'Microsoft.Sql/servers/firewallRules@2022-05-01-preview' = {
  parent: sqlServer
  name: 'AllowContainerAppsSubnet'
  properties: {
    startIpAddress: '10.0.0.0' // Container Apps internal range
    endIpAddress: '10.0.255.255'
  }
}

// NEVER create "0.0.0.0 - 255.255.255.255" rule (public internet)
```

**Connection String (Managed Identity):**

```csharp
// ✅ GOOD: No password in connection string
var connection = new SqlConnection(
    "Server=tcp:wantfood-sql.database.windows.net,1433;" +
    "Database=wantfood_order;" +
    "Authentication=Active Directory Default;");

// BFF Managed Identity automatically authenticates to SQL
```

### Azure Service Bus

**Messaging Security:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Managed Identity Auth** | Enabled | Services authenticate via MSI, no connection strings |
| **Encryption** | TLS 1.2+ for transit; at-rest optional | All messages encrypted in transit |
| **Premium Tier** | Standard for dev/staging; Premium for high SLA prod | Premium adds VNet integration, IP filtering |
| **IP Firewall** | Allow only Container Apps subnet | Block public internet access |
| **Private Endpoints** | Enabled (production) | Route through VNet, not internet |
| **Shared Access Policies** | Disabled (use Managed Identity instead) | No connection strings leaked |

**Bicep Configuration:**

```bicep
resource serviceBusNamespace 'Microsoft.ServiceBus/namespaces@2022-10-01-preview' = {
  name: 'wantfood-sb'
  location: location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
  properties: {
    disableLocalAuth: true // Force Managed Identity only
    publicNetworkAccess: 'Enabled' // But firewall restricts below
    networkRuleSets: {
      defaultAction: 'Deny'
      virtualNetworkRules: [
        {
          subnet: containerAppsSubnet
          ignoreMissingVnetServiceEndpoint: false
        }
      ]
      ipRules: [] // No IP allowlisting (use VNet instead)
    }
  }
}
```

**MassTransit Configuration (Managed Identity):**

```csharp
services.AddMassTransit(x =>
{
    x.UsingAzureServiceBus((context, cfg) =>
    {
        var credential = new DefaultAzureCredential();
        cfg.ConnectUsingTokenCredential(
            new Uri("sb://wantfood-sb.servicebus.windows.net"),
            credential);
        
        // Connection string NOT used; managed identity only
    });
});
```

### Azure Managed Redis

**Cache Security:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Authentication** | Entra ID only (access keys disabled) | No fixed passwords; token-based |
| **Encryption** | TLS 1.2+ for transit; at-rest supported | All data encrypted |
| **Firewall** | Allow only Container Apps subnet | No public internet access |
| **Private Endpoints** | Enabled (production) | VNet-only access |
| **Cluster Mode** | Enabled (prod); disabled (staging) | High availability; partition tolerance |
| **Replication** | 2+ replicas (Premium tier) | Data durability |
| **Purging** | Eviction policy: `allkeys-lru` | Auto-evict oldest keys when full |

**Bicep Configuration:**

```bicep
resource redisenterprise 'Microsoft.Cache/redisEnterprise@2023-11-01' = {
  name: 'wantfood-redis'
  location: location
  sku: {
    name: 'Enterprise_E20'
    capacity: 2
  }
  properties: {}
}

resource redisDb 'Microsoft.Cache/redisEnterprise/databases@2023-11-01' = {
  parent: redisenterprise
  name: 'default'
  properties: {
    clientProtocol: 'Encrypted' // TLS only
    evictionPolicy: 'AllKeysLRU'
    persistence: {
      aofEnabled: false // RDB snapshots OK
      rdbEnabled: true
      rdbFrequency: 'OneHour'
    }
    accessKeysAuthentication: 'Disabled' // Entra ID only
  }
}

resource redisAccessPolicy 'Microsoft.Cache/redisEnterprise/accessPolicyAssignments@2023-11-01' = {
  parent: redisDb
  name: 'bff-access'
  properties: {
    objectId: bffManagedIdentity.properties.principalId
    accessPolicy: 'Data Owner' // Read/write cache
  }
}
```

**Client Configuration (Managed Identity):**

```csharp
var credential = new DefaultAzureCredential();
var redis = ConnectionMultiplexer.Connect(new ConfigurationOptions
{
    EndPoints = { "wantfood-redis.eastus.redisenterprise.cache.azure.com:10000" },
    Ssl = true,
    // No password; managed identity authenticates
    ConfigurationChannel = new RedisChannelOptions(
        credential: credential,
        connectTimeoutMs: 10000)
});
```

### Azure Blob Storage

**Storage Security:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Access Tier** | Hot (frequently accessed) | Optimizes cost for images |
| **Replication** | LRS (locally redundant) or GRS (geo-redundant) | GRS for production multi-region |
| **Firewall** | Allow Front Door + Container Apps | Block public internet access |
| **Private Endpoints** | Enabled (production) | VNet-only access for internal use |
| **Encryption** | TLS 1.2+ transit; AES-256 at rest (default) | All data encrypted |
| **Blob Public Access** | Disabled (all containers private) | No anonymous downloads |
| **Managed Identity** | Required for all access | No storage account keys in code |
| **SAS Tokens** | Time-limited (1 hour max) + read-only | For temporary client uploads |

**Bicep Configuration:**

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'wantfoodblob'
  location: location
  sku: {
    name: 'Standard_LRS' // Change to GRS for prod failover
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    publicNetworkAccess: 'Enabled' // But firewall below restricts
    networkAcls: {
      defaultAction: 'Deny'
      virtualNetworkRules: [
        {
          id: containerAppsSubnet.id
          action: 'Allow'
        }
      ]
      resourceAccessRules: [
        {
          tenantId: subscription().tenantId
          resourceId: frontDoor.id // Allow CDN origin
        }
      ]
    }
  }
}

resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2023-01-01' = {
  parent: storageAccount
  name: 'default'
  properties: {
    cors: [
      {
        allowedOrigins: ['https://wantfood.com'] // Strict CORS
        allowedMethods: ['GET', 'HEAD', 'OPTIONS']
        allowedHeaders: ['*']
        exposedHeaders: ['*']
        maxAgeInSeconds: 86400
      }
    ]
    deleteRetentionPolicy: {
      enabled: true
      days: 7 // Soft delete for 7 days (recovery window)
    }
  }
}

resource imageContainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2023-01-01' = {
  parent: blobService
  name: 'images'
  properties: {
    publicAccess: 'None' // Private; access via CDN / SAS only
  }
}
```

**Client Configuration (Managed Identity):**

```csharp
var credential = new DefaultAzureCredential();
var blobClient = new BlobClient(
    new Uri("https://wantfoodblob.blob.core.windows.net/images/..."),
    credential);

var upload = await blobClient.UploadAsync(stream, overwrite: true);

// Generate read-only SAS for client download (1 hour expiry)
var sasUri = blobClient.GenerateSasUri(
    BlobSasPermissions.Read,
    DateTimeOffset.UtcNow.AddHours(1));

// Client receives URI, downloads directly from CDN (not via BFF)
```

### Azure AI Search (Cognitive Search)

**Search Service Security:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Authentication** | Managed Identity + API keys (for indexing) | Services use Managed Identity; optional API keys for ops |
| **Firewall** | Allow Container Apps subnet only | No public internet search requests |
| **Encryption** | TLS 1.2+ transit; customer-managed keys at rest (optional) | Standard encryption sufficient for dev/staging |
| **RBAC** | Search Service Reader/Contributor per service | Principle of least privilege |
| **Audit Logs** | Enable via Azure Monitor | Track indexing operations |
| **Query Rate Limiting** | Enforce in SearchService middleware | 1000 queries/min per user |

**Bicep Configuration:**

```bicep
resource searchService 'Microsoft.Search/searchServices@2023-11-01' = {
  name: 'wantfood-search'
  location: location
  sku: {
    name: 'basic'
  }
  properties: {
    replicaCount: 3 // HA for prod
    partitionCount: 1
    hostingMode: 'default'
    publicNetworkAccess: 'Enabled' // But firewall below
    networkRuleBypassOptions: 'None'
    networkRuleSets: {
      ipRules: [] // Use VNet integration instead
      virtualNetworkRules: [
        {
          subnet: containerAppsSubnet
        }
      ]
    }
  }
}

resource searchRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  scope: searchService
  name: guid(searchService.id, searchServiceManagedIdentity.id, 'Search Contributor')
  properties: {
    principalId: searchServiceManagedIdentity.properties.principalId
    roleDefinitionId: subscriptionResourceId(
      'Microsoft.Authorization/roleDefinitions',
      'a8b88a84-a47a-4a2a-84cb-23fa59cc446f') // Search Contributor
    principalType: 'ServicePrincipal'
  }
}
```

### Application Insights & Log Analytics

**Monitoring Security:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Data Retention** | 90 days | Sufficient for audit/compliance |
| **Sampling** | 20-50% in production | Reduce costs while maintaining diagnostics |
| **PII Filtering** | Enabled | Automatically redact emails, phone, IPs |
| **Access Control** | RBAC (Reader, Contributor, etc.) | Limit who can view logs |
| **Export to Storage** | Archive older logs (30+ days) | Long-term retention, compliance |
| **Alerts** | Configured as per §7.2 | Critical events trigger on-call |

**Configure PII Filtering:**

```json
{
  "ApplicationInsights": {
    "LoggingOptions": {
      "IncludePII": false,
      "SanitizationPatterns": [
        "email",
        "phone",
        "ipaddress",
        "credit_card"
      ]
    }
  }
}
```

### Key Vault Security

**Secret Management:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Network Access** | Private Endpoint (prod); Allow Container Apps subnet | No public internet access |
| **Purge Protection** | Enabled | Prevent accidental secret deletion |
| **Soft Delete** | 90 days retention | Recover deleted secrets within window |
| **Access Policies** | MSI only (no user access) | Principle of least privilege |
| **Audit Logging** | Enabled, 365-day retention | Track all secret access |
| **Key Rotation** | Quarterly or on breach | Manual or automated via Event Grid |
| **TLS** | 1.2+ only | All communications encrypted |

**Bicep Configuration:**

```bicep
resource keyVault 'Microsoft.KeyVault/vaults@2023-07-01' = {
  name: 'wantfood-kv'
  location: location
  properties: {
    sku: {
      family: 'A'
      name: 'standard'
    }
    tenantId: subscription().tenantId
    enableRbacAuthorization: true // Use RBAC, not access policies
    enablePurgeProtection: true
    softDeleteRetentionInDays: 90
    publicNetworkAccess: 'Enabled' // But firewall below
    networkAcls: {
      defaultAction: 'Deny'
      bypass: 'AzureServices'
      virtualNetworkRules: [
        {
          id: containerAppsSubnet.id
        }
      ]
    }
  }
}

resource privateEndpoint 'Microsoft.Network/privateEndpoints@2023-06-01' = {
  name: 'kv-private-endpoint'
  location: location
  properties: {
    subnet: {
      id: containerAppsSubnet.id
    }
    privateLinkServiceConnections: [
      {
        name: 'kv-connection'
        properties: {
          privateLinkServiceId: keyVault.id
          groupIds: ['vault']
        }
      }
    ]
  }
}

resource bffKeyVaultAccess 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  scope: keyVault
  name: guid(keyVault.id, bffManagedIdentity.id, 'Key Vault Secrets Officer')
  properties: {
    principalId: bffManagedIdentity.properties.principalId
    roleDefinitionId: subscriptionResourceId(
      'Microsoft.Authorization/roleDefinitions',
      'b86a8fe4-44ce-4948-aee5-eccb2c155bad') // Key Vault Secrets Officer
    principalType: 'ServicePrincipal'
  }
}
```

### Container Apps Environment (Network Isolation)

**VNet Integration:**

| Component | Network | Access |
|---|---|---|
| **BFF Container App** | Container Apps subnet | External ingress via Front Door |
| **Microservices** | Container Apps subnet | Internal only; no external ingress |
| **Functions** | Container Apps subnet | Service Bus + Storage only |
| **Azure Bastion** | Bastion subnet (admin access only) | For emergency debugging (rarely used) |

**Bicep:**

```bicep
resource containerEnv 'Microsoft.App/managedEnvironments@2023-11-02-preview' = {
  name: 'wantfood-env'
  location: location
  properties: {
    vnetConfiguration: {
      infrastructureSubnetId: containerAppsSubnet.id
      runtimeSubnetId: containerAppsSubnet.id
      internal: false // BFF needs external ingress
    }
    workloadProfiles: [
      {
        name: 'Consumption'
        workloadProfileType: 'Consumption'
      }
    ]
  }
}

// NSG: Restrict ingress to Front Door + health checks only
resource nsg 'Microsoft.Network/networkSecurityGroups@2023-06-01' = {
  name: 'wantfood-nsg'
  location: location
  properties: {
    securityRules: [
      {
        name: 'AllowFrontDoorIngress'
        properties: {
          protocol: 'Tcp'
          sourcePortRange: '*'
          destinationPortRange: '443'
          sourceAddressPrefix: 'AzureFrontDoor.Backend'
          destinationAddressPrefix: '*'
          access: 'Allow'
          priority: 100
          direction: 'Inbound'
        }
      }
      {
        name: 'AllowHealthChecks'
        properties: {
          protocol: 'Tcp'
          sourcePortRange: '*'
          destinationPortRange: '80'
          sourceAddressPrefix: 'AzureCloud'
          destinationAddressPrefix: '*'
          access: 'Allow'
          priority: 110
          direction: 'Inbound'
        }
      }
      {
        name: 'DenyAllOtherIngress'
        properties: {
          protocol: '*'
          sourcePortRange: '*'
          destinationPortRange: '*'
          sourceAddressPrefix: '*'
          destinationAddressPrefix: '*'
          access: 'Deny'
          priority: 4096
          direction: 'Inbound'
        }
      }
    ]
  }
}
```

---

## 2. Authentication & Authorization

### 2.1 Multi-Layer Auth Strategy

#### Layer 1: End-User Authentication (OIDC)
- **Provider:** Microsoft Entra ID (Azure AD)
- **Clients:** All web apps (FrontEnd, VendorAdmin, SystemAdmin, DriverPortal)
- **Flow:** Authorization Code + PKCE
- **Token:**
  - **ID Token** (OIDC): User identity, stored in session cookie
  - **Access Token** (OAuth 2.0): Bearer token sent to BFF
  - **Refresh Token** (OAuth 2.0): Stored securely, used to refresh access tokens

#### Layer 2: BFF Claims Enrichment
- **JWT validation** via Microsoft.Identity.Web middleware
- **Database role lookup** (DatabaseRolesClaimsTransformation)
  - Query UserService for Entra ID OID → DB roles mapping
  - Enrich ClaimsPrincipal with roles: `Admin`, `Vendor`, `Driver`
- **Authorization policies:**
  - `PlatformAdmin` → SystemAdmin access
  - `VendorAdminAccess` → VendorAdmin access (Vendor or Admin)
  - `DriverAccess` → Driver Portal access (Driver or Admin)
  - Any authenticated user → Web FrontEnd

#### Layer 3: Service-to-Service Auth (Client Credentials)
- **BFF → Microservices:** Client Credentials grant (not OBO)
  - BFF ClientId + ClientSecret
  - Audience: each microservice's app registration
  - Scoped to `api://wantfood-{service}/API.Access`
- **Result:** Access token cached in Redis (TTL = actual expiry - 5 min)

#### Layer 4: Azure Service Bus Authentication
- **Managed Identity** (no secrets needed)
- BFF, all microservices, and Functions use Managed Identities to authenticate to Service Bus
- No connection strings or keys in code

### 2.2 Token Lifecycle & Security

| Component | Storage | TTL | Refresh | Notes |
|-----------|---------|-----|---------|-------|
| **ID Token** (OIDC) | Session cookie (secure, httpOnly) | ~3600s | Automatic via refresh token | User identity |
| **Access Token** (OAuth) | Session cookie (secure, httpOnly) | ~3600s | Automatic via refresh token | BFF bearer token |
| **Service-to-service token** (Client Creds) | Redis cache | ~3599s (expiry - 5 min) | Automatic before expiry | BFF → microservices |
| **Refresh Token** (OAuth) | Session cookie (secure, httpOnly) | ~86400s (24 hrs) | N/A | Stored server-side, never sent to client |

**Security hardening:**
- ✅ Tokens in secure, httpOnly cookies (prevent XSS theft)
- ✅ CSRF protection on all state-changing operations (anti-forgery tokens)
- ✅ SameSite=Strict on cookies to prevent CSRF
- ✅ Cache service tokens to avoid excessive Entra ID calls
- ✅ Refresh tokens rotated on each use (automatic via Microsoft.Identity.Web)

---

## 3. Data Protection & Secrets Management

### 3.1 Azure Key Vault (Mandatory for Production)

**All secrets stored in Key Vault, never in code or config files:**

| Secret Type | Key Vault Secret | Used By | Rotation Policy |
|---|---|---|---|
| **Stripe API Key** | `stripe-api-key` | PaymentService | Quarterly or on breach |
| **Stripe Webhook Secret** | `stripe-webhook-secret` | PaymentService | Quarterly or on breach |
| **OpenAI API Key** | `openai-api-key` | SearchService | Quarterly |
| **BFF Client Secret** | `bff-client-secret` | BFF (OAuth OIDC) | Quarterly |
| **Microservice Client Secrets** | `{service}-client-secret` | BFF (Client Creds) | Quarterly |
| **DB Connection Strings** | `sql-connection-string` | All services via App Config | Quarterly |

**Access Control:**
- Each Container App has a Managed Identity
- Key Vault Access Policy grants `Get` permission only to required identities
- No other team members can view secret values via the portal
- Rotation audited via Azure Monitor / Log Analytics

### 3.2 Managed Identities (Zero Secrets in Code)

**Every Container App and Function has a Managed Identity:**

```
BFF Managed Identity →
  ├─ Key Vault: Get secrets
  ├─ Blob Storage: Read/Write
  ├─ Azure Service Bus: Send/Listen/Manage
  ├─ Azure SQL: Token-based auth
  └─ App Configuration: Read config values

PaymentService Managed Identity →
  ├─ Key Vault: Get secrets (Stripe keys)
  ├─ Blob Storage: Write
  ├─ Azure Service Bus: Send/Listen
  └─ Azure SQL: Token-based auth

[... similar for all services ...]
```

**Implementation:**
```csharp
// BFF startup
var credential = new DefaultAzureCredential();
builder.Services.AddAzureAppConfiguration();
builder.Services.AddKeyVaultSecrets(credential);
builder.Services.AddServiceBusMessaging(credential); // no connection string
```

### 3.3 Encryption in Transit & at Rest

| Layer | Technology | Details |
|-------|---|---|
| **HTTPS/TLS** | TLS 1.2+ (Azure Front Door) | All external traffic encrypted |
| **Service-to-Service** | TLS 1.2+ (Container network) | All internal APIs encrypted |
| **Database** | TLS 1.2+ (Azure SQL) | Transparent Data Encryption (TDE) enabled |
| **Blob Storage** | TLS 1.2+ (Azure Storage) | Server-side encryption at rest (AES-256) |
| **Service Bus** | TLS 1.2+ (Azure Service Bus) | At-rest encryption with customer keys (optional) |
| **Redis** | TLS 1.2+ (Azure Managed Redis) | All connections encrypted |

---

## 4. Authorization Patterns & Policy Enforcement

### 4.1 BFF Authorization Middleware

All BFF endpoints enforce authorization policies **before** calling microservices:

```csharp
// Example endpoint in BFF
[Authorize(Policy = "VendorAdminAccess")]
[HttpGet("/admin/vendors/{vendorId}/menu")]
public async Task<IResult> GetVendorMenu(Guid vendorId, IVendorServiceClient vendorClient)
{
    // Authorization check passed
    // Call microservice via Kiota (automatic Client Credentials flow)
    return Results.Ok(await vendorClient.Api.Vendors[vendorId].Menu.GetAsync());
}
```

### 4.2 Role-Based Access Control (RBAC) — Entra ID + Database

**Entra ID RBAC (static roles):**
- `Admin` → System Admin portal access
- `Vendor` → Vendor Admin portal access
- `Driver` → Driver Portal access
- `Customer` → Web FrontEnd (any authenticated user)

**Database RBAC (dynamic roles, per-vendor/org):**
- Vendor users assigned to specific vendors (M:M relationship)
- Vendor admins can only view/edit their own vendors
- System admins can view all vendors
- Drivers assigned to vendors (team affiliation)

**Enforcement in BFF claims enrichment:**
```csharp
// BFF fetches user roles from UserService
var roles = await userService.GetUserRolesAsync(userOid);
// Adds to ClaimsPrincipal: claim type "role", values: ["Admin", "Vendor"]
// Policies use these claims to authorize endpoints
```

### 4.3 Data-Level Authorization (Microservice Responsibility)

Each microservice **must validate** that the caller has permission to access the requested resource:

```csharp
// Example in VendorService
[HttpGet("/vendors/{vendorId}/menu")]
public async Task<IResult> GetVendorMenu(Guid vendorId, ClaimsPrincipal user)
{
    var userOid = user.GetOid();
    var userRoles = user.GetRoles();
    
    // Check: Is user an admin OR a vendor admin for this vendor?
    var isAuthorized = userRoles.Contains("Admin") ||
        (await _vendorService.IsUserAdminForVendor(userOid, vendorId));
    
    if (!isAuthorized)
        return Results.Forbid();
    
    // Proceed
    return Results.Ok(await _vendorService.GetMenuAsync(vendorId));
}
```

---

## 5. Secure Coding Practices

### 5.1 Input Validation & Sanitization

**All endpoints must validate input:**

```csharp
// Use FluentValidation for complex objects
public class CreateMenuItemRequest
{
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
}

public class CreateMenuItemRequestValidator : AbstractValidator<CreateMenuItemRequest>
{
    public CreateMenuItemRequestValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .MaximumLength(100).WithMessage("Name max 100 chars")
            .Matches(@"^[a-zA-Z0-9\s\-&'(),]+$").WithMessage("Name contains invalid characters");
        
        RuleFor(x => x.Description)
            .MaximumLength(500).WithMessage("Description max 500 chars");
        
        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive")
            .LessThan(9999.99m).WithMessage("Price max £9,999.99");
    }
}

// In endpoint
app.MapPost("/vendors/{vendorId}/menu", async (Guid vendorId, CreateMenuItemRequest req) =>
{
    // Validation runs automatically via MiniValidation or Fluent Validation middleware
    // If invalid, returns 400 Bad Request before handler runs
});
```

### 5.2 SQL Injection Prevention

**Always use parameterized queries (Entity Framework Core):**

```csharp
// ✅ GOOD: EF Core with parameterized queries
var vendor = await dbContext.Vendors
    .Where(v => v.Id == vendorId)
    .FirstOrDefaultAsync();

// ✅ GOOD: Using Dapper with parameters
var items = await connection.QueryAsync<MenuItem>(
    "SELECT * FROM MenuItems WHERE VendorId = @vendorId",
    new { vendorId }
);

// ❌ BAD: String concatenation (forbidden)
// var vendor = dbContext.Vendors.FromSqlRaw($"SELECT * FROM Vendors WHERE Id = {vendorId}");
```

### 5.3 XSS & CSRF Prevention

**Web frontends (Razor Pages):**

```html
<!-- ✅ GOOD: HTML encode user input -->
<h1>@Html.Encode(Model.VendorName)</h1>

<!-- ✅ GOOD: Use Tag Helpers (automatic encoding) -->
<h1 asp-for="VendorName"></h1>

<!-- ✅ GOOD: Anti-forgery token on forms -->
<form asp-action="UpdateMenu" method="post">
    @Html.AntiForgeryToken()
    <input type="text" name="menuName" />
</form>

<!-- ❌ BAD: Direct HTML (allows XSS) -->
<!-- <h1>@Model.VendorName</h1> -->

<!-- ❌ BAD: Missing anti-forgery token -->
<!-- <form action="/update" method="post"> ... </form> -->
```

### 5.4 Secure Deserialization

**Never deserialize untrusted JSON with `JsonSerializerOptions.PropertyNameCaseInsensitive = true` + type mapping:**

```csharp
// ✅ GOOD: Use specific DTOs, strict type mapping
var options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = false, // Strict matching
    TypeInfoResolver = new DefaultJsonTypeInfoResolver()
};
var request = JsonSerializer.Deserialize<CreateMenuItemRequest>(json, options);

// ❌ BAD: Using dynamic objects or loose type mapping
// var data = JsonSerializer.Deserialize<dynamic>(json); // Allows arbitrary properties
```

### 5.5 Dependency Injection (No Service Locator Antipattern)

**Always inject dependencies via constructor; never use `IServiceProvider.GetService()`:**

```csharp
// ✅ GOOD: Constructor injection
public class VendorService(IVendorRepository repo, ILogger<VendorService> logger)
{
    public async Task UpdateVendorAsync(Guid id, string name) => 
        await repo.UpdateAsync(id, name);
}

// ❌ BAD: Service locator (hard to test, hidden dependencies)
// public class VendorService 
// {
//     private readonly IServiceProvider _serviceProvider;
//     public VendorService(IServiceProvider sp) => _serviceProvider = sp;
//     public async Task UpdateVendorAsync(Guid id, string name)
//     {
//         var repo = _serviceProvider.GetService<IVendorRepository>();
//         await repo.UpdateAsync(id, name);
//     }
// }
```

### 5.6 Logging Best Practices

**Log security-relevant events; never log secrets:**

```csharp
// ✅ GOOD: Log auth success/failure, data access
logger.LogInformation("User {UserId} logged in successfully", userId);
logger.LogWarning("Authorization failed for user {UserId} on resource {ResourceId}", userId, resourceId);

// ✅ GOOD: Redact PII in logs
logger.LogInformation("Payment processed for order {OrderId}", orderId);
// Don't log: credit card numbers, SSNs, API keys

// ❌ BAD: Logging secrets
// logger.LogInformation("Using API key: {ApiKey}", apiKey);

// ❌ BAD: Logging PII without redaction
// logger.LogInformation("Customer email: {Email}", email);
```

---

## 6. Deployment Best Practices

### 6.1 Azure Container Registry (ACR) Security

**All container images stored in Azure Container Registry:**

| Setting | Configuration | Rationale |
|---------|---|---|
| **Public access** | Disabled | Images pulled only via Managed Identity |
| **Admin account** | Disabled | Use Managed Identities instead |
| **Image scanning** | Enabled (Microsoft Defender) | Detects vulnerabilities |
| **Retention policy** | 30 days for dev, 90 days for prod | Saves storage costs |
| **Webhook signing** | Enabled | Validate image push/pull events |

### 6.2 Image Signing & Verification

**All production images signed with Notation (optional but recommended):**

```bash
# Sign image during CI/CD
notation sign --key wantfood-cosign-key \
  acrwantfood.azurecr.io/wantfood-api-orderservice:v1.2.3

# Container App deployment validates signature before pulling
```

### 6.3 Secrets in CI/CD (GitHub Actions)

**GitHub Secrets (short-lived tokens for deployment):**

```yaml
name: Deploy to Azure

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Login to Azure using OIDC (no secret storage)
      - uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      # Build & push image (uses logged-in identity)
      - run: |
          az acr build --registry acrwantfood \
            --image wantfood-api-orderservice:${{ github.sha }} \
            --file Dockerfile .
      
      # Deploy (uses logged-in identity)
      - run: |
          az containerapp up \
            --name wantfood-api-orderservice \
            --image acrwantfood.azurecr.io/wantfood-api-orderservice:${{ github.sha }}
```

**Never store:**
- ❌ Container Registry passwords
- ❌ Database passwords
- ❌ API keys (Stripe, OpenAI)
- ❌ Entra ID client secrets

These are fetched from Key Vault at runtime by the container's Managed Identity.

### 6.4 Infrastructure as Code (Bicep) Security

**Bicep parameters marked as `@secure()`:**

```bicep
@secure()
param sqlAdminPassword string

@secure()
param stripeApiKey string

resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
  name: 'wantfood-sql'
  location: location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: sqlAdminPassword
  }
}

// KeyVault secret (not passed as parameter)
resource vaultSecret 'Microsoft.KeyVault/vaults/secrets@2021-11-01-preview' = {
  parent: vault
  name: 'stripe-api-key'
  properties: {
    value: stripeApiKey
  }
}
```

**Best practice:** Store sensitive Bicep parameters in Azure Key Vault, reference via Key Vault URI:
```bash
az containerapp create \
  --resource-group wantfood \
  --name wantfood-api-orderservice \
  --environment $CONTAINER_APP_ENV \
  --image acrwantfood.azurecr.io/wantfood-api-orderservice:latest \
  --secrets "stripe-api-key=$KEYVAULT_SECRET_ID" \
  --env-vars "STRIPE_KEY_NAME=stripe-api-key"
```

### 6.5 Blue-Green Deployments

**Container Apps Traffic Splits for safe rollouts:**

```bash
# Deploy new revision (weight 0)
az containerapp update \
  --name wantfood-api-orderservice \
  --traffic "current-revision=100" "new-revision=0"

# Test new revision in production
# (Internal traffic only, not exposed to users)

# Gradually shift traffic (5% to new)
az containerapp traffic-weight \
  --name wantfood-api-orderservice \
  --revision-weights current=95 new=5 --wait

# Monitor error rates, then shift 100% or rollback
az containerapp traffic-weight \
  --name wantfood-api-orderservice \
  --revision-weights current=0 new=100 --wait
```

### 6.6 Network Security Groups (NSGs) & Firewall Rules

**Azure SQL Firewall:**
```bash
# Only allow Container Apps
az sql server firewall-rule create \
  --server wantfood-sql \
  --name "allow-container-apps" \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255 \
  # Use Managed Identity instead (recommended)
```

**Recommended: Managed Identity + RBAC (no firewall rule needed)**
```csharp
// In-code: Connection string fetched from Key Vault
// Database auth via Managed Identity token (no password)
var connection = new SqlConnection($"Server=wantfood-sql.database.windows.net; Database=wantfood_order; Authentication=Active Directory Default;");
```

---

## 7. Monitoring, Auditing & Compliance

### 7.1 Azure Monitor & Log Analytics

**All activity logged and retained for 90+ days:**

| Log Type | Source | Retention | Alert Threshold |
|---|---|---|---|
| **Application Insights** | App code via OpenTelemetry | 90 days | Custom (5 GB/mo free) |
| **Container App Logs** | Container output | 30 days | Warnings & errors |
| **Activity Log** | Azure resource changes | 90 days | Any data plane modifications |
| **Azure SQL Audit Logs** | Database access & queries | 90 days | Failed logins, privilege changes |
| **Key Vault Audit Logs** | Secret access | 90 days | All Get/Set/Delete operations |

### 7.2 Security Events to Alert On

**Critical alerts (page on-call):**
- Authorization failures (5+ in 1 min)
- SQL injection attempt detected
- Unauthenticated API requests to BFF (10+ in 1 min)
- Key Vault secret access denied

**Warning alerts (Slack notification):**
- Deployment failures
- Container App restart loop
- High error rate (>5%) in Application Insights
- Service Bus message backlog (>1000 messages)

### 7.3 Compliance & Auditing

**Maintain audit trail for:**
- Who accessed what data (via ClaimsPrincipal + logging)
- When configuration changes occurred (Azure Activity Log)
- Which secrets were accessed (Key Vault audit)
- Payment transactions (PCI-DSS requirement)

**Example audit query:**
```kusto
// App Insights: Who fetched sensitive data?
customEvents
| where name == "VendorPaymentDetails.Fetched"
| extend UserId = tostring(customDimensions.user_id)
| extend VendorId = tostring(customDimensions.vendor_id)
| summarize by UserId, VendorId, timestamp
```

---

## 8. External Integrations (Stripe, OpenAI)

### 8.1 Stripe Webhook Security

**Webhook signature validation (mandatory):**

```csharp
public class StripeWebhookHandler
{
    private readonly string _endpointSecret; // From Key Vault
    
    public async Task<IResult> HandleWebhook(HttpContext context)
    {
        var json = await new StreamReader(context.Request.Body).ReadToEndAsync();
        var stripeSignature = context.Request.Headers["Stripe-Signature"];
        
        try
        {
            // ✅ GOOD: Verify signature
            var stripeEvent = EventUtility.ConstructEvent(
                json, stripeSignature, _endpointSecret);
            
            // Process event
            return Results.Ok();
        }
        catch (StripeException ex)
        {
            // ❌ Signature invalid
            return Results.BadRequest();
        }
    }
}
```

### 8.2 OpenAI API Security

**Store API key in Key Vault; use least-privilege scopes:**

```csharp
var openAiClient = new OpenAIClient(new Uri("https://api.openai.com/v1"), 
    new AzureKeyCredential(keyVaultClient.GetSecret("openai-api-key")));

// Call only whitelisted operations (semantic search)
var embedding = await openAiClient.GetEmbeddingsAsync(...)
```

### 8.3 Rate Limiting on External APIs

**Implement exponential backoff + circuit breaker:**

```csharp
services.AddHttpClient<IOpenAiClient, OpenAiClient>()
    .AddTransientHttpErrorPolicy(p =>
        p.Or<HttpRequestException>()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: retryAttempt =>
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))))
    .AddPolicyHandler(GetCircuitBreakerPolicy());

static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy() =>
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 5,
            durationOfBreak: TimeSpan.FromSeconds(30));
```

---

## 9. Incident Response & Security Contacts

### 9.1 Security Incident Checklist

| Step | Owner | Action | Timeframe |
|---|---|---|---|
| 1. **Detect** | Monitoring/Alerts | Anomaly detected | Immediate |
| 2. **Isolate** | DevOps | Scale affected service to 0; block IPs if DDoS | 5 min |
| 3. **Investigate** | Security team | Review logs, Application Insights, audit trail | 30 min |
| 4. **Contain** | DevOps | Rotate secrets if compromised; patch if needed | 1 hour |
| 5. **Eradicate** | DevOps + Dev | Fix root cause; redeploy if necessary | 4 hours |
| 6. **Recover** | DevOps | Restore service; validate functionality | 2 hours |
| 7. **Document** | Team | Post-incident review; update processes | 24 hours |

### 9.2 Security Contacts

**On-Call Escalation:**
1. **L1 (Dev):** First responder (DevOps)
2. **L2 (Security):** Security engineer (if data breach suspected)
3. **L3 (Executive):** VP Engineering (if public-facing impact)

---

## 10. Checklists & Quick Reference

### 10.1 Pre-Deployment Security Checklist

- [ ] All secrets moved to Key Vault (no hardcoded values)
- [ ] Managed Identities assigned to all services
- [ ] Authorization policies in place on all BFF endpoints
- [ ] Input validation on all endpoints
- [ ] SQL queries use parameterized queries
- [ ] HTTPS/TLS enforced (no HTTP)
- [ ] CSRF tokens present on all state-changing forms
- [ ] Logging does not include PII or secrets
- [ ] Container image scanned for vulnerabilities
- [ ] Blue-green deployment tested
- [ ] Rollback procedure documented
- [ ] Monitoring & alerts configured

### 10.2 Code Review Security Questions

**Reviewer must verify:**

1. **No secrets in code?** (Check for: API keys, connection strings, passwords)
2. **Input validated?** (Check for: length, type, allowed characters)
3. **SQL parameterized?** (Check for: `.FromSqlRaw()` or string concatenation)
4. **Authorization checked?** (Check for: `[Authorize(...)]` or explicit permission checks)
5. **No service locator?** (Check for: `IServiceProvider.GetService()`)
6. **Logging sanitized?** (Check for: no PII, no secrets)
7. **Dependencies injected?** (Check for: constructor injection)
8. **Error handling present?** (Check for: try-catch, specific exceptions)
9. **Async for I/O?** (Check for: `await`, `async Task`)
10. **Rate limiting?** (For external APIs: circuit breaker, retries)

---

## Appendix: References

- **Azure Security Best Practices:** https://docs.microsoft.com/en-us/security/benchmark/azure/overview
- **OWASP Top 10:** https://owasp.org/Top10/
- **PCI-DSS Compliance:** https://www.pcisecuritystandards.org/ (for Stripe integration)
- **Microsoft Identity Web:** https://github.com/AzureAD/microsoft-identity-web
- **MassTransit Security:** https://masstransit.io/documentation/configuration/security
