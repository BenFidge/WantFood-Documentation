# Performance Testing Guide

## Overview

This guide covers comprehensive performance testing strategies for WantFood's microservices architecture across all environments (development, staging, production). It includes load testing, stress testing, post-release monitoring, and device/browser compatibility testing using Azure services and third-party tools.

### Target Performance Metrics

Based on cost analysis projections:
- **100 Users/Day:** Baseline capacity testing
- **1,000 Users/Day:** Standard production load
- **10,000 Users/Day:** Peak capacity (holiday/promotion scenarios)

---

## Section 1: Azure Load Testing

### 1.1 Overview

Azure Load Testing provides native integration with Azure Pipelines and Application Insights for distributed, scalable load, stress, soak, and spike testing of HTTP/gRPC endpoints.

### 1.2 Setup and Configuration

#### Prerequisites
- Azure subscription with Load Testing resource provisioned
- JMeter test scripts (.jmx files)
- Service principal for CI/CD authentication

#### Bicep Template for Azure Load Testing

```bicep
param location string = resourceGroup().location
param loadTestName string = 'wantfood-load-test'
param environment string = 'production' // 'dev', 'staging', 'production'

resource loadTestResource 'Microsoft.LoadTestService/loadTests@2022-12-01' = {
  name: loadTestName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  tags: {
    environment: environment
    application: 'wantfood'
    team: 'platform'
  }
}

// RBAC role assignment for service principal
param servicePrincipalObjectId string = ''

resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = if (!empty(servicePrincipalObjectId)) {
  scope: loadTestResource
  name: guid(loadTestResource.id, servicePrincipalObjectId, 'Load Test Runner')
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'a92cd5a1-06ce-4b66-af07-12cb9b0d5cda') // Load Test Contributor
    principalId: servicePrincipalObjectId
    principalType: 'ServicePrincipal'
  }
}

output loadTestResourceId string = loadTestResource.id
output loadTestName string = loadTestResource.name
```

#### Azure Pipelines Integration

```yaml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  AZURE_LOAD_TEST_RESOURCE: 'wantfood-load-test'
  AZURE_LOAD_TEST_RG: 'rg-wantfood-prod'

stages:
  - stage: LoadTesting
    displayName: 'Performance Testing'
    jobs:
      - job: RunLoadTest
        displayName: 'Azure Load Testing'
        steps:
          - checkout: self
            fetchDepth: 0

          - task: AzureLoadTest@1
            inputs:
              azureSubscription: $(azureSubscriptionEndpoint)
              loadTestConfigFile: 'tests/performance/load-test-config.yaml'
              resourceGroup: $(AZURE_LOAD_TEST_RG)
              loadTestResource: $(AZURE_LOAD_TEST_RESOURCE)
              secrets: |
                [
                  {
                    "name": "admin_token",
                    "value": "$(ADMIN_PAT_TOKEN)"
                  },
                  {
                    "name": "api_endpoint",
                    "value": "$(API_ENDPOINT)"
                  }
                ]
              env: |
                [
                  {
                    "name": "target_endpoint",
                    "value": "$(TARGET_ENDPOINT)"
                  },
                  {
                    "name": "ramp_up_users",
                    "value": "50"
                  },
                  {
                    "name": "hold_duration_sec",
                    "value": "300"
                  }
                ]

          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '$(System.DefaultWorkingDirectory)/loadTestResults/*.xml'
              mergeTestResults: true
              failTaskOnFailedTests: false
              publishRunAttachments: true
```

### 1.3 Load Testing Scenarios

#### Scenario 1: Baseline Load (100 Users)
**Purpose:** Validate core endpoints under normal conditions
**Configuration:**
- Ramp-up: 5 minutes (1.67 users/sec)
- Hold: 10 minutes at 100 users
- Ramp-down: 5 minutes
- Endpoints tested: Order placement, menu retrieval, payment confirmation, driver assignment
- Expected metrics:
  - 95th percentile latency: < 500ms
  - 99th percentile latency: < 1000ms
  - Error rate: < 0.1%
  - Throughput: > 1000 req/sec

```yaml
# tests/performance/jmeter/baseline-100-users.jmx
# Include JMeter script with:
# - Thread group: 100 threads
# - Ramp-up period: 300 seconds
# - Hold-load duration: 600 seconds
# - HTTP samplers for:
#   - GET /api/restaurants (no auth)
#   - POST /api/orders (auth + payload)
#   - GET /api/orders/{id}/status (auth)
#   - POST /api/payments/confirm (auth + Stripe integration)
```

#### Scenario 2: Standard Production Load (1,000 Users)
**Purpose:** Validate sustained performance under typical daily load
**Configuration:**
- Ramp-up: 15 minutes (1.11 users/sec)
- Hold: 30 minutes at 1,000 users
- Ramp-down: 10 minutes
- Endpoints tested: Same as baseline + driver availability checks
- Expected metrics:
  - 95th percentile latency: < 800ms
  - 99th percentile latency: < 1500ms
  - Error rate: < 0.05%
  - Throughput: > 8,000 req/sec

#### Scenario 3: Peak Load (10,000 Users)
**Purpose:** Validate capacity under extreme conditions (promotions, holidays)
**Configuration:**
- Ramp-up: 30 minutes (5.56 users/sec)
- Hold: 20 minutes at 10,000 users
- Ramp-down: 15 minutes
- Endpoints tested: Same as above + surge pricing calculations
- Expected metrics:
  - 95th percentile latency: < 1200ms
  - 99th percentile latency: < 2000ms
  - Error rate: < 0.5%
  - Throughput: > 80,000 req/sec
  - Queue depths monitored (Service Bus)

#### Scenario 4: Stress Testing
**Purpose:** Identify breaking points and automatic scaling triggers
**Configuration:**
- Increment threads by 500 every 2 minutes until failure
- Failure criteria: Error rate > 5% or 99th percentile latency > 5000ms
- Maximum target: 50,000 concurrent users
- Monitored metrics: CPU utilization, memory, connection pool exhaustion, queue backlogs

#### Scenario 5: Soak Testing
**Purpose:** Detect memory leaks and connection pool exhaustion over time
**Configuration:**
- Constant load: 500 users
- Duration: 4 hours
- Monitored metrics: 
  - Memory usage (should remain stable)
  - Garbage collection pauses (< 100ms)
  - Connection pool utilization
  - Database connection leaks

#### Scenario 6: Spike Testing
**Purpose:** Validate system behavior when traffic suddenly spikes
**Configuration:**
- Baseline: 100 users for 5 minutes
- Spike: Jump to 5,000 users for 3 minutes
- Return to baseline: 100 users for 5 minutes
- Monitored metrics: Recovery time, queue draining, error recovery

### 1.4 Integrating JMeter Scripts

```jmx
<!-- Example JMeter HTTP Sampler for Order Placement -->
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="POST /api/orders" enabled="true">
  <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
    <collectionProp name="Arguments.arguments">
      <elementProp name="restaurantId" elementType="HTTPArgument">
        <boolProp name="HTTPArgument.always_encode">false</boolProp>
        <stringProp name="Argument.name">restaurantId</stringProp>
        <stringProp name="Argument.value">${__Random(1,10000)}</stringProp>
        <stringProp name="Argument.metadata">=</stringProp>
      </elementProp>
    </collectionProp>
  </elementProp>
  <stringProp name="HTTPSampler.domain">${target_endpoint}</stringProp>
  <stringProp name="HTTPSampler.port"></stringProp>
  <stringProp name="HTTPSampler.protocol">https</stringProp>
  <stringProp name="HTTPSampler.contentEncoding"></stringProp>
  <stringProp name="HTTPSampler.path">/api/orders</stringProp>
  <stringProp name="HTTPSampler.method">POST</stringProp>
  <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
  <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
  <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
  <boolProp name="HTTPSampler.DO_MULTIPART_POST">false</boolProp>
  <stringProp name="HTTPSampler.embedded_url_re"></stringProp>
  <stringProp name="HTTPSampler.connect_timeout"></stringProp>
  <stringProp name="HTTPSampler.response_timeout"></stringProp>
</HTTPSamplerProxy>
```

---

## Section 2: Application Insights Performance Profiling

### 2.1 Baseline Establishment

Run performance tests against a stable baseline deployment and capture metrics:

```csharp
// In Startup.cs or Program.cs
services.AddApplicationInsightsTelemetry(options =>
{
    options.EnablePerformanceCounterCollectionModule = true;
    options.EnableEventCounterCollectionModule = true;
    options.EnableAdaptiveSampling = true;
    options.AdaptiveSamplingSettings.InitialSamplingPercentage = 10; // 10% of requests
    options.AdaptiveSamplingSettings.SamplingPercentageDecreaseTimeout = TimeSpan.FromMinutes(1);
});

services.AddApplicationInsightsSnapshotCollector();
```

### 2.2 Monitoring Dashboards

Create custom dashboards in Application Insights:

```kusto
// Latency distribution (P50, P95, P99)
let testStart = ago(24h);
requests
| where timestamp > testStart
| summarize
    P50_Latency = percentile(duration, 50),
    P95_Latency = percentile(duration, 95),
    P99_Latency = percentile(duration, 99),
    AvgLatency = avg(duration),
    ErrorCount = count(success == false),
    TotalCount = count()
    by tostring(name), bin(timestamp, 1m)
| render timechart
```

```kusto
// Dependency call performance
dependencies
| where timestamp > ago(24h)
| summarize
    AvgDuration = avg(duration),
    P99Duration = percentile(duration, 99),
    FailureCount = sum(iff(success == false, 1, 0))
    by tostring(target), tostring(type)
| render barchart
```

### 2.3 Anomaly Detection

Enable automatic anomaly detection in Application Insights:
- Detection method: Smart detection for abnormal failures and performance degradations
- Alert when: Failure rate deviates from baseline by > 2 standard deviations
- Alert when: Request latency deviates from baseline by > 2 standard deviations

---

## Section 3: Device and Browser Testing with BrowserStack

### 3.1 Setup

#### BrowserStack Integration

```yaml
# Tests run via Playwright against BrowserStack selenium grid
# capabilities.browserstack.yml
capabilities:
  browserstack.local: true
  browserstack.username: $(BROWSERSTACK_USERNAME)
  browserstack.accessKey: $(BROWSERSTACK_ACCESS_KEY)
  browserName: chrome
  browserVersion: latest
  platformName: Windows
  "bstack:options":
    os: Windows
    osVersion: '11'
    sessionName: 'WantFood Functional Tests'
    projectName: 'WantFood'
    buildName: 'Build #$(Build.BuildId)'
    debug: true
    networkLogs: true
    console: errors
```

#### Playwright BrowserStack Integration

```csharp
// tests/e2e/BrowserStackHelper.cs
public class BrowserStackHelper
{
    private readonly IBrowser _browser;
    private readonly IBrowserContext _context;

    public BrowserStackHelper(string username, string accessKey)
    {
        var endpoint = $"wss://{username}:{accessKey}@cdp.browserstack.com";
        _browser = Playwright.Chromium.ConnectAsync(endpoint).Result;
        _context = _browser.NewContextAsync().Result;
    }

    public async Task<IPage> CreatePageAsync()
    {
        return await _context.NewPageAsync();
    }

    public async Task TearDownAsync()
    {
        await _context.CloseAsync();
        await _browser.CloseAsync();
    }
}

// tests/e2e/OrderFlowTests.cs
[TestFixture]
public class OrderFlowTests
{
    private IBrowserContext _context;
    private IPage _page;

    [OneTimeSetUp]
    public async Task Setup()
    {
        var helper = new BrowserStackHelper(
            Environment.GetEnvironmentVariable("BROWSERSTACK_USERNAME"),
            Environment.GetEnvironmentVariable("BROWSERSTACK_ACCESS_KEY")
        );
        _page = await helper.CreatePageAsync();
    }

    [Test]
    public async Task Should_Complete_Order_On_Chrome_Latest()
    {
        await _page.GotoAsync("https://app.wantfood.io");
        await _page.ClickAsync("text=Browse Restaurants");
        await _page.SelectOptionAsync("select[name='cuisine']", new[] { "italian" });
        await _page.ClickAsync("button:text('Order')");
        
        // Assert order success
        await Expect(_page).ToHaveURLAsync(new Regex("orders/.*/confirmation"));
    }

    [Test]
    [TestCase("chrome", "latest")]
    [TestCase("firefox", "latest")]
    [TestCase("safari", "latest")]
    [TestCase("edge", "latest")]
    public async Task Should_Render_Correctly_On_Multiple_Browsers(string browser, string version)
    {
        // Cross-browser tests
        // Each runs on BrowserStack actual device
    }

    [OneTimeTearDown]
    public async Task TearDown()
    {
        await _page.CloseAsync();
    }
}
```

### 3.2 Device Coverage Matrix

| Device Category | Devices | Priority | Resolution | Network |
|---|---|---|---|---|
| **Mobile (iOS)** | iPhone 15, iPhone 14 Pro, iPhone 13 | High | 1125x2436 | 4G LTE |
| **Mobile (Android)** | Samsung Galaxy S24, Pixel 8, OnePlus 12 | High | 1440x3120 | 4G LTE |
| **Tablet (iPad)** | iPad Air (6th gen), iPad Pro 12.9 | Medium | 1024x1366 | Wi-Fi 6 |
| **Tablet (Android)** | Samsung Tab S10, Lenovo Tab P12 | Medium | 1536x2048 | Wi-Fi 6 |
| **Desktop (Windows)** | Windows 10/11 + Chrome/Edge | High | 1920x1080 | Fiber (100 Mbps) |
| **Desktop (Mac)** | macOS 14/15 + Safari/Chrome | Medium | 1920x1080 | Fiber (100 Mbps) |

### 3.3 Test Scenarios

#### Scenario: Order Placement Flow on Mobile
```csharp
[Test]
public async Task Should_Place_Order_On_iPhone_15_4G()
{
    // Network throttling: 4G (4 Mbps down, 3 Mbps up)
    // Latency: 20ms
    // Packet loss: 1%
    
    var stopwatch = Stopwatch.StartNew();
    
    await _page.GotoAsync("https://app.wantfood.io");
    Assert.That(_page.Url, Does.Contain("app.wantfood.io"));
    
    // Search for restaurant
    await _page.FillAsync("input[placeholder='Search restaurants']", "Pizza Palace");
    await _page.ClickAsync("text=Pizza Palace");
    
    // Add items to cart
    await _page.ClickAsync("text=Margherita Pizza");
    await _page.FillAsync("input[aria-label='Quantity']", "2");
    await _page.ClickAsync("button:text('Add to Cart')");
    
    // Checkout
    await _page.ClickAsync("button:text('Proceed to Payment')");
    await _page.FillAsync("input[name='cardNumber']", "4242424242424242");
    await _page.ClickAsync("button:text('Pay')");
    
    stopwatch.Stop();
    
    // Assertions
    await Expect(_page).ToHaveURLAsync(new Regex("orders/.*/confirmation"));
    Assert.That(stopwatch.ElapsedMilliseconds, Is.LessThan(8000), "Order flow should complete within 8 seconds on 4G");
}
```

---

## Section 4: Post-Release Monitoring

### 4.1 Performance Baseline Collection

Before production deployment, establish a baseline:

```csharp
// Capture baseline metrics from staging environment load test
public class PerformanceBaseline
{
    public double P50LatencyMs { get; set; } = 200;
    public double P95LatencyMs { get; set; } = 500;
    public double P99LatencyMs { get; set; } = 1000;
    public double ErrorRatePercent { get; set; } = 0.05;
    public double ThroughputReqSec { get; set; } = 1000;
}
```

### 4.2 Real User Monitoring (RUM)

Enable Application Insights JavaScript SDK in BFF frontend:

```html
<!-- In _Layout.cshtml -->
<script type="text/javascript">
!function(T,l,y){var S=T.location,k="script",D="instrumentationKey",C="ingestionendpoint",I="disableExceptionTracking",E="ai.device.",b="toLowerCase",w="crossOriginPolicy",N="POST",e="appInsightsSDK",t=y.name||"appInsights";(y.name||T[e])&&(T[e]=t);var n=T[t]||function(d){var g=!1;var f=!1;var m={initialize:!0,queue:[],sv:"7.0.0",version:2.0,config:d};
// Full minified SDK initialization code
</script>

<script type="text/javascript">
// Track custom performance metrics
window.appInsights?.trackPageView({
    name: document.title,
    url: window.location.href,
    duration: window.performance?.timing?.loadEventEnd - window.performance?.timing?.navigationStart,
    properties: {
        environmentId: '@ViewBag.EnvironmentId'
    }
});

// Track interactions
document.addEventListener('click', function(e) {
    if (e.target.tagName === 'BUTTON' || e.target.tagName === 'A') {
        window.appInsights?.trackEvent({
            name: 'UserAction',
            properties: {
                action: e.target.innerText,
                path: window.location.pathname
            }
        });
    }
});
</script>
```

### 4.3 Alert Configuration

Set up alert rules in Azure Monitor:

```bicep
// Alerts for performance degradation
resource latencyAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'wantfood-high-latency'
  location: 'global'
  properties: {
    enabled: true
    scopes: [
      applicationInsightsId
    ]
    evaluationFrequency: 'PT5M'
    windowSize: 'PT15M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'P95 Latency'
          metricName: 'performanceCounters/processorCpuPercentage'
          operator: 'GreaterThan'
          threshold: 800
          timeAggregation: 'Average'
          dimensions: []
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroupId
      }
    ]
  }
}

resource errorRateAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'wantfood-high-error-rate'
  location: 'global'
  properties: {
    enabled: true
    scopes: [
      applicationInsightsId
    ]
    evaluationFrequency: 'PT5M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria'
      allOf: [
        {
          name: 'Error Rate'
          metricName: 'requests/failed'
          operator: 'GreaterThan'
          threshold: 50 // 50 failed requests in 5 min window
          timeAggregation: 'Count'
        }
      ]
    }
    actions: [
      {
        actionGroupId: actionGroupId
      }
    ]
  }
}
```

### 4.4 SLI/SLO Tracking

```kusto
// Monthly SLI calculation
let windowStart = startofmonth(now());
let windowEnd = endofmonth(now());

union 
  (requests | where timestamp >= windowStart and timestamp < windowEnd),
  (dependencies | where timestamp >= windowStart and timestamp < windowEnd)
| summarize
    RequestCount = count(),
    SuccessfulRequests = sum(iff(success == true, 1, 0)),
    FailedRequests = sum(iff(success == false, 1, 0)),
    LatencyP99 = percentile(duration, 99),
    LatencyP95 = percentile(duration, 95)
    by tostring(name)
| extend
    Availability = (SuccessfulRequests * 100.0) / RequestCount,
    LatencySLI = iff(LatencyP99 < 1000, 100, 50)
| project
    Endpoint = name,
    Availability,
    LatencySLI,
    AvailabilitySLO = 99.5,
    LatencySLO = 1000,
    P95_Latency = LatencyP95,
    P99_Latency = LatencyP99
```

### 4.5 Anomaly Detection Alerting

```yaml
# Enable in Application Insights Smart Detection
name: "Performance Degradation"
enabled: true
rule_type: "AnomalousFailureRateRule"
scope: "requests | dependencies"
condition: "failure_rate > baseline * 1.5 OR latency_p99 > baseline * 2"
notification_channels:
  - "email:team@wantfood.io"
  - "slack:#incidents"
  - "pagerduty:wantfood-sre"
```

---

## Section 5: Performance Incident Response

### 5.1 Performance Degradation Investigation

**Trigger:** Latency increases by 50% or error rate exceeds 0.5%

**Response Steps:**

1. **Immediate triage (5 min)**
   - Check Application Insights live metrics
   - Identify affected endpoints
   - Check Azure health dashboard for service incidents

2. **Gather diagnostics (10 min)**
   ```kusto
   // Identify slow requests
   requests
   | where timestamp > ago(30m)
   | where duration > 2000
   | summarize Count = count(), AvgDuration = avg(duration)
   | by tostring(name), bin(timestamp, 1m)
   | render timechart
   ```

3. **Check dependencies (10 min)**
   ```kusto
   // Database query performance
   dependencies
   | where name contains "Sql" and timestamp > ago(30m)
   | summarize AvgDuration = avg(duration), P99 = percentile(duration, 99)
   | by target
   ```

4. **Scale up if needed (5 min)**
   - Check Container Apps CPU/memory utilization
   - Increase replica count if CPU > 70%

5. **Remediation**
   - Identify root cause (code, database query, external dependency)
   - Apply fix or rollback recent deployment
   - Monitor for 30 minutes to confirm resolution

### 5.2 Capacity Planning

Use metrics from load testing to plan infrastructure:

```csharp
// Projected costs for different user loads
var capacityPlanning = new[]
{
    new { UsersPerDay = 100, ContainerAppReplicas = 2, CostPerMonth = "$450" },
    new { UsersPerDay = 1000, ContainerAppReplicas = 5, CostPerMonth = "$1200" },
    new { UsersPerDay = 10000, ContainerAppReplicas = 15, CostPerMonth = "$3200" },
};
```

---

## Section 6: Performance Testing Checklist

- [ ] Load test baseline (100 users) passes with latency < 500ms P95
- [ ] Load test standard (1K users) passes with latency < 800ms P95
- [ ] Load test peak (10K users) passes with latency < 1200ms P95
- [ ] Stress test identifies breaking point > 5K concurrent users
- [ ] Soak test (4 hours @ 500 users) shows no memory leaks
- [ ] Spike test recovery time < 30 seconds
- [ ] All endpoints tested on mobile (iOS + Android)
- [ ] All endpoints tested on tablet (iPad + Android tablet)
- [ ] Desktop rendering verified (Windows 10/11, macOS)
- [ ] 4G latency scenarios validated (mobile)
- [ ] Application Insights dashboards operational
- [ ] Anomaly detection alerts configured
- [ ] Post-release on-call procedures documented
- [ ] SLI/SLO targets defined and tracked
- [ ] Performance regression tests integrated in CI/CD

---

## Appendix: Azure Load Testing Resource Limits

| Metric | Limit |
|---|---|
| Max engine instances per test | 45 |
| Max threads per engine | 1000 |
| Max virtual users per test | 45,000 |
| Max test duration | 24 hours |
| Storage quota per resource | 1 TB |
| Tests per resource | Unlimited |

---

## References

- [Azure Load Testing Documentation](https://learn.microsoft.com/azure/load-testing/)
- [JMeter User Manual](https://jmeter.apache.org/usermanual/)
- [BrowserStack Documentation](https://www.browserstack.com/docs)
- [Application Insights Best Practices](https://learn.microsoft.com/azure/azure-monitor/app/tutorial-performance)
- [Azure Monitor Alerts](https://learn.microsoft.com/azure/azure-monitor/alerts/alerts-overview)
