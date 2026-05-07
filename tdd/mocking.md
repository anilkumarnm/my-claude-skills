# When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, GitLab, Jira)
- Databases (sometimes - prefer pgtest for real DB)
- Time/randomness
- File system (sometimes)
- HTTP clients

Don't mock:

- Your own packages/modules
- Internal collaborators
- Anything you control

## Designing for Mockability

At system boundaries, design interfaces that are easy to mock.

### 1. Use Dependency Injection

Pass external dependencies in rather than creating them internally.

#### Go

```go
// GOOD: Easy to mock — accepts interface
type PaymentProcessor struct {
    client PaymentClient  // interface, not concrete type
}

func (p *PaymentProcessor) Process(ctx context.Context, order *Order) error {
    return p.client.Charge(ctx, order.Total)
}

// BAD: Hard to mock — creates dependency internally
func ProcessPayment(ctx context.Context, order *Order) error {
    client := stripe.NewClient(os.Getenv("STRIPE_KEY"))
    return client.Charge(ctx, order.Total)
}
```

#### TypeScript

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

### 2. Define Interfaces for External Dependencies

#### Go (with counterfeiter)

```go
// Define interface for external dependency
//go:generate go run github.com/maxbrunsfeld/counterfeiter/v6 -generate

//counterfeiter:generate . PaymentClient
type PaymentClient interface {
    Charge(ctx context.Context, amount int64) (*ChargeResult, error)
}

//counterfeiter:generate . MetadataClient
type MetadataClient interface {
    Query(ctx context.Context, keys []string) (*QueryResult, error)
}
```

Then in tests:

```go
var _ = Describe("PaymentProcessor", func() {
    var (
        fakeClient *fakes.FakePaymentClient
        processor  *PaymentProcessor
    )
    
    BeforeEach(func() {
        fakeClient = &fakes.FakePaymentClient{}
        processor = &PaymentProcessor{client: fakeClient}
    })
    
    It("charges the order total", func() {
        fakeClient.ChargeReturns(&ChargeResult{Status: "success"}, nil)
        
        err := processor.Process(ctx, order)
        
        Expect(err).NotTo(HaveOccurred())
    })
})
```

### 3. Prefer Specific Methods over Generic Fetchers

#### Go

```go
// GOOD: Each method is independently mockable
type MissionMonitorClient interface {
    GetStage(ctx context.Context, id string) (*Stage, error)
    UpdateStageStatus(ctx context.Context, id string, status Status) error
    QueryMetadata(ctx context.Context, keys []string) (*MetadataResult, error)
}

// BAD: Generic method requires conditional fake logic
type APIClient interface {
    Do(ctx context.Context, method, path string, body any) (any, error)
}
```

#### TypeScript

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

### Benefits of Specific Interfaces

- Each mock returns one specific shape
- No conditional logic in test setup
- Easier to see which endpoints a test exercises
- Type safety per endpoint
- Counterfeiter generates type-safe fakes automatically

## Project Conventions

### Go (Mission Control)

- Use `counterfeiter` for generating fakes: `go generate ./...`
- Fakes go in `*fakes` package (auto-generated)
- Use `pgtest.SetupInMemoryRepo` for real DB tests
- Interface methods MUST accept `context.Context` as first argument

### TypeScript (React)

- Use Jest mocks for external APIs
- Use React Testing Library for component tests
- Mock at the API client level, not fetch level
