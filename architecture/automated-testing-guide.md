# Automated Testing Guide

## 1. Unit Testing with xUnit

### 1.1 xUnit Setup

Install NuGet packages:

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package NSubstitute
dotnet add package FluentAssertions
```

### 1.2 Naming Convention

- Test class: `{ClassName}Tests`
- Test method: `{MethodName}_Given{Condition}_Should{ExpectedBehavior}`

Example:

```csharp
public class OrderServiceTests
{
    private readonly IOrderRepository _orderRepository;
    private readonly OrderService _service;

    public OrderServiceTests()
    {
        _orderRepository = Substitute.For<IOrderRepository>();
        _service = new OrderService(_orderRepository);
    }

    [Fact]
    public async Task CreateOrder_GivenValidOrder_ShouldReturnOrderId()
    {
        // Arrange
        var order = new Order { RestaurantId = Guid.NewGuid(), Items = [new OrderItem { ProductId = Guid.NewGuid(), Quantity = 2 }] };
        _orderRepository.AddAsync(Arg.Any<Order>()).Returns(Task.CompletedTask);

        // Act
        var result = await _service.CreateOrderAsync(order);

        // Assert
        result.Should().NotBe(Guid.Empty);
        await _orderRepository.Received(1).AddAsync(Arg.Any<Order>());
    }

    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public async Task CreateOrder_GivenInvalidRestaurantId_ShouldThrowArgumentException(string? restaurantId)
    {
        // Arrange
        var order = new Order { RestaurantId = string.IsNullOrEmpty(restaurantId) ? Guid.Empty : Guid.Parse(restaurantId) };

        // Act & Assert
        await Assert.ThrowsAsync<ArgumentException>(() => _service.CreateOrderAsync(order));
    }
}
```

### 1.3 Substituting Best Practices

- **Substitute only external dependencies** (repositories, HTTP clients, message buses)
- **Don't substitute the class under test**
- **Use `Arg.Any<T>()`** for flexible argument matching
- **Use `Arg.Is<T>()`** for conditional matching with predicates
- **Verify critical interactions** using `Received()` and `DidNotReceive()`
- **Set up return values** before invocation

```csharp
// Good: Substitute the dependency
var paymentService = Substitute.For<IPaymentService>();
paymentService.ProcessPaymentAsync(Arg.Any<Payment>())
    .Returns(new PaymentResult { IsSuccessful = true });

// Verify was called
await paymentService.Received(1).ProcessPaymentAsync(Arg.Any<Payment>());

// Conditional matching
paymentService.GetOrderStatus(Arg.Is<Guid>(id => id != Guid.Empty))
    .Returns(OrderStatus.Pending);

// Bad: Don't substitute the class under test
var orderService = Substitute.For<OrderService>();
```

### 1.4 Async/Await Testing Patterns

```csharp
[Fact]
public async Task ProcessOrderAsync_GivenValidOrder_ShouldCompleteWithinTimeout()
{
    // Arrange
    var repository = Substitute.For<IOrderRepository>();
    var service = new OrderService(repository);
    var order = new Order { /* ... */ };
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));

    // Act
    var result = await service.ProcessOrderAsync(order, cts.Token);

    // Assert
    result.Should().NotBeNull();
}

[Fact]
public async Task ProcessOrderAsync_GivenCancellation_ShouldThrowOperationCanceledException()
{
    // Arrange
    var repository = Substitute.For<IOrderRepository>();
    var service = new OrderService(repository);
    repository.GetAsync(Arg.Any<Guid>())
        .Returns(async _ => 
        {
            await Task.Delay(5000);
            return new Order();
        });
    var cts = new CancellationTokenSource();
    cts.CancelAfter(100);

    // Act & Assert
    await Assert.ThrowsAsync<OperationCanceledException>(
        () => service.ProcessOrderAsync(new Order(), cts.Token)
    );
}
```

### 1.5 NSubstitute Advanced Patterns

#### Argument Matchers

```csharp
// Exact match
repository.GetAsync(expectedId).Returns(order);

// Any argument
repository.GetAsync(Arg.Any<Guid>()).Returns(order);

// Conditional match
repository.GetAsync(Arg.Is<Guid>(id => id != Guid.Empty)).Returns(order);

// Matching by property
repository.GetByRestaurant(Arg.Is<Order>(o => o.RestaurantId == restaurantId))
    .Returns(orders);
```

#### Return Values for Async Methods

```csharp
// Task<T>
var repository = Substitute.For<IOrderRepository>();
repository.GetAsync(Arg.Any<Guid>())
    .Returns(Task.FromResult(new Order { Id = Guid.NewGuid() }));

// ConfigureAwait handling
repository.GetAsync(Arg.Any<Guid>())
    .Returns(x => Task.FromResult(new Order { Id = (Guid)x[0] }));
```

#### Callback and Side Effects

```csharp
var repository = Substitute.For<IOrderRepository>();
var savedOrder = default(Order);

repository.AddAsync(Arg.Do<Order>(o => savedOrder = o))
    .Returns(Task.CompletedTask);

await service.CreateOrderAsync(order);

savedOrder.Should().NotBeNull();
savedOrder.Id.Should().Be(order.Id);
```

#### Exceptions

```csharp
var paymentService = Substitute.For<IPaymentService>();
paymentService.ProcessPaymentAsync(Arg.Any<Payment>())
    .Returns(Task.FromException<PaymentResult>(new PaymentException("Service unavailable")));

await Assert.ThrowsAsync<PaymentException>(() => service.CompleteOrderAsync(order));
```

### 1.6 Unit Test Coverage Targets

- **Overall code coverage:** 80%+ for business logic
- **Critical paths:** 100% coverage (payment processing, order creation, vendor management)
- **Utility functions:** 70%+ coverage
- **Infrastructure code:** 60%+ coverage (repositories, mappers)

Run coverage analysis:

```bash
dotnet test /p:CollectCoverage=true /p:CoverageFormat=opencover /p:Exclude=[xunit.*]*
```

---

## 2. Integration Testing with TestContainers

### 2.1 TestContainers Setup

Install packages:

```bash
dotnet add package Testcontainers
dotnet add package Testcontainers.PostgreSql
dotnet add package Testcontainers.RabbitMq
dotnet add package MassTransit.TestFramework
```

### 2.2 Database Integration Tests

```csharp
[Collection("Database collection")]
public class OrderRepositoryIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgres;
    private readonly OrderDbContext _dbContext;

    public OrderRepositoryIntegrationTests()
    {
        _postgres = new PostgreSqlBuilder()
            .WithDatabase("testdb")
            .WithUsername("test")
            .WithPassword("test")
            .Build();
    }

    public async Task InitializeAsync()
    {
        await _postgres.StartAsync();
        var options = new DbContextOptionsBuilder<OrderDbContext>()
            .UseNpgsql(_postgres.GetConnectionString())
            .Options;
        
        _dbContext = new OrderDbContext(options);
        await _dbContext.Database.MigrateAsync();
    }

    public async Task DisposeAsync()
    {
        await _dbContext.DisposeAsync();
        await _postgres.StopAsync();
    }

    [Fact]
    public async Task GetOrderById_GivenValidOrderId_ShouldReturnOrder()
    {
        // Arrange
        var order = new Order 
        { 
            Id = Guid.NewGuid(),
            RestaurantId = Guid.NewGuid(),
            Status = OrderStatus.Pending,
            CreatedAt = DateTimeOffset.UtcNow
        };
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();

        var repository = new OrderRepository(_dbContext);

        // Act
        var result = await repository.GetByIdAsync(order.Id);

        // Assert
        result.Should().NotBeNull();
        result?.Id.Should().Be(order.Id);
        result?.Status.Should().Be(OrderStatus.Pending);
    }
}
```

### 2.3 Message Bus Integration Tests

```csharp
[Collection("MassTransit collection")]
public class OrderServiceMessageBusTests : IAsyncLifetime
{
    private readonly RabbitMqContainer _rabbit;
    private InMemoryTestHarness _harness;
    private OrderService _service;

    public OrderServiceMessageBusTests()
    {
        _rabbit = new RabbitMqBuilder().Build();
    }

    public async Task InitializeAsync()
    {
        await _rabbit.StartAsync();

        _harness = new InMemoryTestHarness();
        var busControl = Bus.Factory.CreateUsingRabbitMq(cfg =>
        {
            cfg.Host(_rabbit.Hostname, _rabbit.Port, "/", h =>
            {
                h.Username("guest");
                h.Password("guest");
            });
            cfg.ConfigureEndpoints(_harness);
        });

        await _harness.Start();
        _service = new OrderService(_harness.Bus);
    }

    public async Task DisposeAsync()
    {
        await _harness.Stop();
        await _rabbit.StopAsync();
    }

    [Fact]
    public async Task PublishOrderCreated_GivenValidOrder_ShouldPublishEvent()
    {
        // Arrange
        var order = new Order { Id = Guid.NewGuid(), RestaurantId = Guid.NewGuid() };

        // Act
        await _service.CreateAndPublishOrderAsync(order);

        // Assert
        _harness.Published.Select<OrderCreatedEvent>().Should().HaveCount(1);
        var published = _harness.Published.Select<OrderCreatedEvent>().First();
        published.Message.OrderId.Should().Be(order.Id);
    }
}
```

### 2.4 Integration Test Best Practices

- **Use `IAsyncLifetime`** for container lifecycle management
- **Run tests in parallel** with separate container instances per test class
- **Seed test data** in `InitializeAsync()` before each test
- **Use test fixtures** for shared test infrastructure (collections in xUnit)
- **Clean up resources** in `DisposeAsync()` to prevent resource leaks

---

## 3. UI/E2E Testing with Playwright

### 3.1 Playwright Setup

Install packages:

```bash
dotnet add package Microsoft.Playwright
dotnet add package Playwright
pwsh bin/Debug/net8.0/playwright.ps1 install
```

### 3.2 Page Object Model Pattern

```csharp
public class OrderCheckoutPage
{
    private readonly IPage _page;

    public OrderCheckoutPage(IPage page)
    {
        _page = page;
    }

    public async Task FillCustomerDetailsAsync(string name, string email, string address)
    {
        await _page.FillAsync("[data-testid='customer-name']", name);
        await _page.FillAsync("[data-testid='customer-email']", email);
        await _page.FillAsync("[data-testid='customer-address']", address);
    }

    public async Task SelectPaymentMethodAsync(string method)
    {
        await _page.ClickAsync($"text={method}");
    }

    public async Task ClickSubmitOrderAsync()
    {
        await _page.ClickAsync("[data-testid='submit-order']");
    }

    public async Task<string?> GetErrorMessageAsync()
    {
        return await _page.TextContentAsync("[data-testid='error-message']");
    }

    public async Task WaitForOrderConfirmationAsync()
    {
        await _page.WaitForURLAsync(new Regex(".*/order-confirmation"));
    }
}

[PlaywrightTest]
public class CheckoutFlowTests
{
    [Test]
    public async Task CompleteCheckout_GivenValidOrder_ShouldDisplayConfirmation()
    {
        // Arrange
        await using var browser = await Playwright.CreateAsync().Chromium.LaunchAsync();
        var page = await browser.NewPageAsync();
        var checkoutPage = new OrderCheckoutPage(page);

        // Act
        await page.GotoAsync("https://localhost:7001/checkout");
        await checkoutPage.FillCustomerDetailsAsync("John Doe", "john@example.com", "123 Main St");
        await checkoutPage.SelectPaymentMethodAsync("Credit Card");
        await checkoutPage.ClickSubmitOrderAsync();

        // Assert
        await checkoutPage.WaitForOrderConfirmationAsync();
        var url = page.Url;
        url.Should().Contain("/order-confirmation");
    }
}
```

### 3.3 BrowserStack Integration

Configure in `playwright.config.ts`:

```json
{
  "use": {
    "connectOptions": {
      "browserName": "chromium",
      "wsEndpoint": "wss://cdp.browserstack.com/playwright?caps=%7B%22browserName%22%3A%22chrome%22%2C%22browserVersion%22%3A%22latest%22%2C%22os%22%3A%22Windows%22%2C%22osVersion%22%3A%2211%22%7D",
      "timeout": 30000
    }
  },
  "webServer": {
    "command": "dotnet run",
    "port": 7001,
    "reuseExistingServer": false
  }
}
```

Device testing matrix (BrowserStack):
- **iOS:** iPhone 15 Pro, iPhone 14, iPhone 13 (Safari, latest)
- **Android:** Galaxy S24, Pixel 8, OnePlus 12 (Chrome, latest)
- **Desktop:** Windows 11 (Chrome, Edge), macOS 15 (Chrome, Safari)

---

## 4. Performance Testing Integration

### 4.1 Azure Load Testing in CI/CD

Azure Pipelines integration:

```yaml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        steps:
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
              arguments: '--configuration Release'

          - task: DotNetCoreCLI@2
            inputs:
              command: 'test'
              arguments: '--configuration Release /p:CollectCoverage=true'

  - stage: PerformanceTesting
    condition: succeeded()
    jobs:
      - job: LoadTesting
        steps:
          - task: AzureLoadTest@1
            inputs:
              azureSubscription: 'Azure Subscription'
              loadTestConfigFile: 'load-test.yaml'
              resourceGroup: 'wantfood-rg'
              loadTestResource: 'wantfood-load-test'
              engineInstances: 1
```

### 4.2 k6 Performance Testing

Install: `npm install -g k6`

Example load test script (`load-test.js`):

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

export const errorRate = new Rate('errors');
export const responseTime = new Trend('http_request_duration');

export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 1000 },
    { duration: '10m', target: 1000 },
    { duration: '3m', target: 0 },
  ],
  thresholds: {
    'http_req_duration': ['p(95)<800'],
    'errors': ['rate<0.1'],
  },
};

export default function() {
  const response = http.post('https://api.wantfood.local/orders', {
    restaurantId: 'restaurant-id',
    items: [{ productId: 'product-id', quantity: 2 }],
  });

  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 800ms': (r) => r.timings.duration < 800,
  }) || errorRate.add(1);

  sleep(1);
}
```

Run: `k6 run load-test.js`

---

## 5. Test Data Management

### 5.1 Database Seeding

```csharp
public static class TestDataSeeder
{
    public static async Task SeedOrderTestDataAsync(OrderDbContext context)
    {
        var restaurants = Enumerable.Range(1, 5)
            .Select(i => new Restaurant
            {
                Id = Guid.NewGuid(),
                Name = $"Restaurant {i}",
                CreatedAt = DateTimeOffset.UtcNow
            })
            .ToList();

        var products = restaurants.SelectMany(r => Enumerable.Range(1, 10)
            .Select(j => new Product
            {
                Id = Guid.NewGuid(),
                RestaurantId = r.Id,
                Name = $"Product {j}",
                Price = (j * 10) + 5.99m
            }))
            .ToList();

        var orders = Enumerable.Range(1, 100)
            .Select(i => new Order
            {
                Id = Guid.NewGuid(),
                RestaurantId = restaurants[i % restaurants.Count].Id,
                CustomerId = Guid.NewGuid(),
                Status = OrderStatus.Delivered,
                CreatedAt = DateTimeOffset.UtcNow.AddDays(-(i / 10))
            })
            .ToList();

        context.Restaurants.AddRange(restaurants);
        context.Products.AddRange(products);
        context.Orders.AddRange(orders);

        await context.SaveChangesAsync();
    }
}
```

### 5.2 Builder Pattern for Test Objects

```csharp
public class OrderBuilder
{
    private Guid _id = Guid.NewGuid();
    private Guid _restaurantId = Guid.NewGuid();
    private Guid _customerId = Guid.NewGuid();
    private OrderStatus _status = OrderStatus.Pending;
    private readonly List<OrderItem> _items = [];

    public OrderBuilder WithId(Guid id)
    {
        _id = id;
        return this;
    }

    public OrderBuilder WithRestaurantId(Guid restaurantId)
    {
        _restaurantId = restaurantId;
        return this;
    }

    public OrderBuilder WithStatus(OrderStatus status)
    {
        _status = status;
        return this;
    }

    public OrderBuilder WithItem(Guid productId, int quantity)
    {
        _items.Add(new OrderItem { ProductId = productId, Quantity = quantity });
        return this;
    }

    public Order Build()
    {
        return new Order
        {
            Id = _id,
            RestaurantId = _restaurantId,
            CustomerId = _customerId,
            Status = _status,
            Items = _items,
            CreatedAt = DateTimeOffset.UtcNow
        };
    }
}

// Usage:
var order = new OrderBuilder()
    .WithRestaurantId(restaurantId)
    .WithStatus(OrderStatus.Processing)
    .WithItem(productId, 2)
    .Build();
```

---

## 6. CI/CD Pipeline Integration

### 6.1 SonarQube Code Coverage

Add to Azure Pipelines:

```yaml
- task: SonarQubePrepare@5
  inputs:
    SonarQube: 'SonarQube'
    scannerMode: 'MSBuild'
    projectKey: 'wantfood'
    projectVersion: '$(Build.BuildNumber)'
    extraProperties: |
      sonar.cs.coverage.reportPaths=$(Agent.TempDirectory)/coverage.opencover.xml

- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--configuration Release /p:CollectCoverage=true /p:CoverageFormat=opencover'

- task: SonarQubeAnalyze@5

- task: SonarQubePublish@5
  inputs:
    pollingTimeoutSec: '300'
```

### 6.2 Test Gate Policies

Enforce before merge:
- Unit test pass rate: 100%
- Code coverage: 80%+ minimum
- Performance test P95 latency: < 800ms
- No high-severity SonarQube findings

---

## 7. Testing Metrics & Reporting

### 7.1 Azure Pipelines Test Report

```yaml
- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'VSTest'
    testResultsFiles: '**/*test-results.trx'
    mergeTestResults: true
    failTaskOnFailedTests: true
    publishRunAttachments: true
```

### 7.2 Code Coverage Dashboard

```csharp
// Generate coverage badge
var coverage = GetCodeCoveragePercentage();
var badgeColor = coverage >= 80 ? "brightgreen" : "orange";
var badge = $"![Coverage](https://img.shields.io/badge/coverage-{coverage}%25-{badgeColor})";

File.WriteAllText("coverage-badge.md", badge);
```

---

## 8. Testing Best Practices Checklist

✓ Unit tests run in < 5 seconds total  
✓ Integration tests run in parallel with isolated containers  
✓ Each test has single responsibility  
✓ Test names are descriptive (Given/When/Then pattern)  
✓ Mock only external dependencies  
✓ Verify both positive and negative scenarios  
✓ Use FluentAssertions for readable assertions  
✓ No hardcoded test data (use builders/seeders)  
✓ Database tests use transactions for rollback  
✓ Async methods tested with proper timeout handling  
✓ Performance thresholds enforced in CI/CD  
✓ Test failures are deterministic (no flaky tests)  
✓ Code coverage tracked and trending upward  
✓ SonarQube gates prevent quality regression  
✓ Load testing scenarios reflect production usage  
✓ Device/browser matrix includes mobile first  
✓ Test documentation matches code changes  
✓ Deprecated tests removed promptly  
✓ Test failures root-caused before merge  
✓ Performance baselines reviewed quarterly  
