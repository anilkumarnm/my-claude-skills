# Interface Design for Testability

Good interfaces make testing natural.

## 1. Accept Dependencies, Don't Create Them

### Go

```go
// GOOD: Testable — accepts interface dependency
type StageProcessor struct {
    dbServicer db.DatabaseServicer
    mmClient   MMClient
}

func NewStageProcessor(db db.DatabaseServicer, mm MMClient) *StageProcessor {
    return &StageProcessor{dbServicer: db, mmClient: mm}
}

// BAD: Hard to test — creates dependency internally
func ProcessStage(ctx context.Context, stageID string) error {
    db := db.NewPostgresClient(os.Getenv("DATABASE_URL"))
    // Can't inject a fake for testing
}
```

### TypeScript

```typescript
// Testable
function processOrder(order, paymentGateway) {}

// Hard to test
function processOrder(order) {
  const gateway = new StripeGateway();
}
```

## 2. Return Results, Don't Produce Side Effects

### Go

```go
// GOOD: Testable — returns result
func CalculateDiscount(cart *Cart) (*Discount, error) {
    // Pure calculation, easy to test
    return &Discount{Amount: cart.Total * 0.1}, nil
}

// BAD: Hard to test — mutates input
func ApplyDiscount(cart *Cart) {
    cart.Total -= cart.Total * 0.1  // Side effect, harder to verify
}
```

### TypeScript

```typescript
// Testable
function calculateDiscount(cart): Discount {}

// Hard to test
function applyDiscount(cart): void {
  cart.total -= discount;
}
```

## 3. Small Surface Area

- Fewer methods = fewer tests needed
- Fewer params = simpler test setup

### Go

```go
// GOOD: Deep module — simple interface, complex implementation
type MetadataValidator interface {
    Validate(ctx context.Context, input *ValidationInput) (*ValidationResult, error)
}

// BAD: Shallow module — many methods, each trivial
type MetadataValidator interface {
    CheckKeyExists(ctx context.Context, key string) (bool, error)
    CheckKeyFresh(ctx context.Context, key string, since time.Time) (bool, error)
    GetKeyValue(ctx context.Context, key string) (string, error)
    GetKeyTimestamp(ctx context.Context, key string) (time.Time, error)
    // ... 10 more methods
}
```

## 4. Context as First Parameter (Go)

All interface methods must accept `context.Context` as the first argument:

```go
// GOOD: Supports logging, tracing, cancellation
type Servicer interface {
    GetStage(ctx context.Context, id string) (*Stage, error)
    UpdateStatus(ctx context.Context, id string, status Status) error
}

// BAD: No context — can't propagate timeouts or tracing
type Servicer interface {
    GetStage(id string) (*Stage, error)
}
```

## 5. Use Pointer Receivers for Structs with State

```go
// GOOD: Pointer receiver for stateful struct
func (p *StageProcessor) Process(ctx context.Context, stage *Stage) error {
    return p.mmClient.UpdateStatus(ctx, stage.ID, StatusComplete)
}

// Input structs: use pointers to distinguish unset vs zero value
type CreateRequest struct {
    Name        *string `json:"name"`        // nil = not provided
    Description *string `json:"description"` // nil = not provided
}
```

## Project Patterns (Mission Control)

### BaseOp Pattern

```go
// All business logic operations implement BaseOp
type BaseOp interface {
    Name() string                              // For metrics/tracing
    Validate() error                           // Check required fields
    Execute(ctx context.Context) (any, error)  // Main logic
}

// Easy to test: inject fakes, call Execute, verify result
func TestRegisterStageOp(t *testing.T) {
    op := &RegisterStageOp{
        dbServicer: fakeDB,
        request:    validRequest,
    }
    
    result, err := op.Execute(ctx)
    
    // Assert on result
}
```

### ComponentContext for Dependency Injection

```go
// Dependencies injected via ComponentContext
type ComponentContext struct {
    DBServicer    db.DatabaseServicer
    MMClient      MMClient
    TemporalClient temporal.Client
}

// In tests: create ComponentContext with fakes
ctx := &ComponentContext{
    DBServicer: &fakes.FakeDatabaseServicer{},
    MMClient:   &fakes.FakeMMClient{},
}
```
