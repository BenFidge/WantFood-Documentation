# WantFood System Documentation Index

This guide provides a comprehensive overview of the system architecture, security practices, testing strategies, and operational procedures for the distributed food delivery platform.

## 📑 Quick Navigation

### 🏗️ Architecture & Design

#### [Microservices Architecture](./microservices-architecture.md)
**Complete reference for all 12 microservices in the system.**
- Full microservices catalog with responsibilities and interfaces
- MassTransit Saga pattern for order processing (6-state lifecycle)
- Azure Functions architecture for async workloads
- AI-powered intent-based search using Azure OpenAI
- Real-time notifications via SignalR/NotificationHub
- Service-to-service communication with Managed Identity
- Database per service pattern
- End-to-end data flow diagrams

**For:** New developers understanding system design, architects designing new features, engineers implementing microservices

#### [Architecture Diagram](./architecture-diagram.md)
**Visual reference for system components and infrastructure.**
- High-level component relationships
- Environment-specific subgraphs (Dev/Staging vs Production)
- Infrastructure layers (Presentation, BFF, Services, Data)
- Infrastructure migration strategy and timeline
- Component legend with types and colors

**For:** Quick visual reference, infrastructure planning, stakeholder presentations

#### [Azure Cost Analysis](./azure-cost-analysis.md)
**Infrastructure cost breakdown and service tier justification.**
- Per-service monthly cost estimates
- Dev/Staging infrastructure (RabbitMQ, Elasticsearch)
- Production infrastructure (Service Bus, AI Search)
- **Critical constraint:** Service Bus Basic tier does NOT support topics (Standard tier required)
- Cost optimization recommendations
- Migration path with timeline

**For:** Infrastructure planning, budget forecasting, cost optimization decisions

---

### 🔐 Security & Deployment

#### [Security & Deployment Best Practices](./security-and-deployment-practices.md)
**Production security policies and deployment standards.**
- BFF as single security gatekeeper
- Azure Front Door & CDN with WAF
- 11 production-ready Bicep templates for infrastructure
- Managed Identity authentication (zero secrets in code)
- Claims enrichment and authorization patterns
- Service-to-service token caching
- Secure coding practices and checklists
- Deployment strategies (canary, blue-green)
- Incident response procedures and playbooks
- Monitoring and alerting setup
- Compliance and audit logging

**For:** DevOps engineers, security architects, deployment pipeline configuration, infrastructure provisioning

**Key Patterns:**
```bicep
// All 11 templates included:
- SQL Database with Managed Identity RBAC
- Service Bus with private endpoints
- Managed Redis for token caching
- Blob Storage with SAS URLs
- AI Search with RBAC
- Application Insights and Log Analytics
- Key Vault with access policies
- Container Apps Environment
- Front Door with WAF rules
- CDN with origin failover
- Event Grid for event routing
```

---

### 🧪 Testing & Quality Assurance

#### [Automated Testing Guide](./automated-testing-guide.md)
**Comprehensive testing strategy and implementation patterns.**
- Unit testing with xUnit + FluentAssertions + NSubstitute
  - NSubstitute advanced patterns (argument matchers, async returns, callbacks)
  - Test fixtures and builder patterns
  - Shared test context
- Integration testing with TestContainers
  - PostgreSQL containers
  - RabbitMQ/Service Bus containers
  - Integration test examples
- UI/E2E testing with Playwright
  - Cross-browser testing setup
  - BrowserStack integration
- Performance testing with Azure Load Testing and k6
- Test data management strategies
- CI/CD pipeline integration with SonarQube
- Code coverage targets and reporting
- 20-item testing best practices checklist

**Coverage:** 80% minimum code coverage, P95 latency < 800ms, zero high-severity SonarQube findings

**For:** Software engineers, QA specialists, CI/CD pipeline maintainers, architects defining quality gates

---

### 📊 Performance Testing & Monitoring

#### [Performance Testing Guide](./performance-testing-guide.md)
**Load testing, performance baselines, and post-release monitoring.**
- Azure Load Testing setup with Bicep template
- 6 load testing scenarios:
  - Baseline test (100 users)
  - Normal load test (1,000 users)
  - Peak load test (10,000 users)
  - Stress test (push to limits)
  - Soak test (sustained load 24h)
  - Spike test (sudden traffic spikes)
- Performance thresholds and SLI/SLO targets
- Application Insights performance monitoring
  - Custom KPI metrics
  - Dependency tracking
  - Exception analysis
- BrowserStack device/browser testing matrix
  - iOS/Android devices
  - Chrome/Safari/Firefox browsers
  - Tablet and desktop testing
- Real User Monitoring (RUM) setup
- Post-release SLI/SLO tracking
- Performance incident response procedures
- Troubleshooting common performance issues

**For:** Performance engineers, DevOps specialists, product managers tracking SLOs, incident responders

---

## 🚀 Getting Started

### New Developers
1. Start with [Microservices Architecture](./microservices-architecture.md) to understand service boundaries and event flows
2. Review [Architecture Diagram](./architecture-diagram.md) for visual reference
3. Read [Security & Deployment Best Practices](./security-and-deployment-practices.md) for production constraints
4. Check [Automated Testing Guide](./automated-testing-guide.md) for testing standards

### Infrastructure/DevOps Engineers
1. Review [Azure Cost Analysis](./azure-cost-analysis.md) for service tier requirements
2. Read [Security & Deployment Best Practices](./security-and-deployment-practices.md) for infrastructure provisioning
3. Check [Performance Testing Guide](./performance-testing-guide.md) for load testing and monitoring

### QA/Testing Specialists
1. Start with [Automated Testing Guide](./automated-testing-guide.md) for testing standards
2. Review [Performance Testing Guide](./performance-testing-guide.md) for load and performance testing
3. Refer to individual microservice documentation for service-specific test scenarios

### Security/Compliance Officers
1. Read [Security & Deployment Best Practices](./security-and-deployment-practices.md) for full security posture
2. Review deployment and incident response procedures
3. Check compliance and audit logging sections

---

## 📚 Documentation Organization

```
src/docs/
├── INDEX.md (this file)
├── architecture-diagram.md
├── Architecture/
│   ├── microservices-architecture.md
│   ├── security-and-deployment-practices.md
│   └── azure-cost-analysis.md
├── Performance/
│   └── performance-testing-guide.md
├── Testing/
│   └── automated-testing-guide.md
├── BFF/
├── Deployment/
├── Entra/
├── Front End PoC/
├── Manifests/
├── Order Processing and Payments/
├── Plans/
├── Registration/
└── [other domain-specific docs]
```

---

## 🔑 Key Technologies & Patterns

### Messaging & Events
- **Dev/Staging:** RabbitMQ (zero cost, quick iteration)
- **Production:** Azure Service Bus Standard tier (enterprise reliability)
- **Abstraction:** MassTransit (transparent transport switching)

### Data Access
- **Microservices:** PostgreSQL (isolated per service)
- **Search:** Azure AI Search (semantic + full-text)
- **Cache:** Azure Managed Redis (Managed Identity auth)

### Compute
- **APIs & Services:** Azure Container Apps (managed Kubernetes)
- **Async Workloads:** Azure Functions (image processing, provisioning, background jobs)
- **Frontend:** Blazor WebAssembly SPA + Container App

### Authentication & Authorization
- **Identity:** Entra ID (formerly Azure AD)
- **Service Auth:** Managed Identity (RBAC, zero secrets)
- **API Gateway:** BFF (single entry point)
- **Frontend Auth:** MSAL.NET + Entra ID

### Monitoring & Observability
- **Application Insights:** Structured logs, traces, custom metrics
- **Real User Monitoring:** Client-side performance tracking
- **Log Analytics:** KQL queries for operational insights
- **Alerts:** Threshold-based incidents

---

## 📋 Microservices at a Glance

| Service | Responsibility | Key Technology | Communication |
|---------|---|---|---|
| **Order Service** | Order lifecycle management | MassTransit Saga (6-state) | Events, HTTP |
| **Payment Service** | Payment processing | Stripe/PayPal/Square | Events, Webhooks |
| **Vendor Service** | Restaurant profiles & menus | PostgreSQL | HTTP, Events |
| **User Service** | User profiles & preferences | PostgreSQL, Functions | Events, HTTP |
| **Delivery Service** | Driver assignment & routing | Azure Maps | HTTP, Events |
| **Search Service** | Intent-based AI search | Azure OpenAI, AI Search | HTTP, Events |
| **File Service** | Image management & processing | Functions, Blob Storage, CDN | HTTP |
| **Notification Service** | Multi-channel alerts | SignalR, SMS/Email/Push | Events |
| **Chat Service** | Real-time messaging | SignalR, WebSocket | HTTP, Events |
| **Promotion Service** | Coupons & campaigns | PostgreSQL | Events, HTTP |
| **Copilot Service** | AI support assistant | Azure OpenAI, Vector DB | HTTP |
| **Content Service** | Static content & FAQs | PostgreSQL | HTTP |

---

## 🎯 Testing Matrix

| Test Type | Technology | Coverage Target | Frequency |
|---|---|---|---|
| **Unit Tests** | xUnit + NSubstitute + FluentAssertions | 80% | Every commit |
| **Integration Tests** | TestContainers + PostgreSQL | Service boundaries | Every commit |
| **E2E Tests** | Playwright + BrowserStack | Critical user paths | Nightly |
| **Performance Tests** | Azure Load Testing + k6 | P95 < 800ms | Pre-release |
| **Load Tests** | Azure Load Testing (6 scenarios) | Spike resilience | Weekly |
| **Device Tests** | BrowserStack | All target devices | Nightly |

---

## 🚨 Critical Constraints & Decisions

### Infrastructure
- **Service Bus Tier:** Basic tier does NOT support topics (required by MassTransit). Production MUST use Standard tier (~£8/mo base)
- **Database Per Service:** Each microservice has isolated PostgreSQL. Synchronization via events only.
- **Managed Identity Only:** No connection strings or API keys in code. All service authentication via Entra ID Managed Identity.

### Security
- **BFF Gateway:** Only public endpoint. All microservices are internal-only.
- **Azure Front Door:** All traffic routes through Front Door + WAF before reaching BFF.
- **CDN:** Images served exclusively through CDN with SAS URL pre-signing.

### Performance
- **P95 Latency Target:** < 800ms for API endpoints
- **Error Rate:** < 0.1% (SLO 99.9%)
- **Availability:** > 99.5% uptime

### AI & Search
- **Intent-Based Search:** Azure OpenAI extracts structured intent from natural language
- **Semantic Matching:** AI Search embedding models rank results by relevance
- **Denormalized Index:** Vendor + Menu data combined for performance

---

## 📞 Support & Contributions

For questions about specific services, refer to the [Microservices Architecture](./microservices-architecture.md) guide.

For infrastructure or deployment questions, see [Security & Deployment Best Practices](./security-and-deployment-practices.md).

For testing or quality questions, consult the [Automated Testing Guide](./automated-testing-guide.md).

---

**Last Updated:** July 2026  
**Version:** 1.0  
**Status:** Production Ready
