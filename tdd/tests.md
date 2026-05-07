# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

### Go (Ginkgo/Gomega)

```go
// GOOD: Tests observable behavior
var _ = Describe("Checkout", func() {
    It("confirms order with valid cart", func() {
        cart := createCart()
        cart.Add(product)
        
        result, err := checkout(ctx, cart, paymentMethod)
        
        Expect(err).NotTo(HaveOccurred())
        Expect(result.Status).To(Equal("confirmed"))
    })
})
```

### TypeScript (Jest)

```typescript
// GOOD: Tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

### Characteristics

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One logical assertion per test

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

### Go (Ginkgo/Gomega)

```go
// BAD: Tests implementation details
var _ = Describe("Checkout", func() {
    It("calls payment service process", func() {
        fakePayment := &fakes.FakePaymentService{}
        
        checkout(ctx, cart, fakePayment)
        
        // Testing HOW, not WHAT
        Expect(fakePayment.ProcessCallCount()).To(Equal(1))
        Expect(fakePayment.ProcessArgsForCall(0)).To(Equal(cart.Total))
    })
})
```

### TypeScript (Jest)

```typescript
// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

### Red Flags

- Mocking internal collaborators
- Testing private methods
- Asserting on call counts/order
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

## Interface vs Database Verification

### Go (Ginkgo/Gomega)

```go
// BAD: Bypasses interface to verify
var _ = Describe("CreateUser", func() {
    It("saves to database", func() {
        createUser(ctx, &User{Name: "Alice"})
        
        // Querying DB directly — coupled to storage implementation
        var row User
        db.NewSelect().Model(&row).Where("name = ?", "Alice").Scan(ctx)
        Expect(row).NotTo(BeNil())
    })
})

// GOOD: Verifies through interface
var _ = Describe("CreateUser", func() {
    It("makes user retrievable", func() {
        user, err := createUser(ctx, &User{Name: "Alice"})
        Expect(err).NotTo(HaveOccurred())
        
        retrieved, err := getUser(ctx, user.ID)
        Expect(err).NotTo(HaveOccurred())
        Expect(retrieved.Name).To(Equal("Alice"))
    })
})
```

### TypeScript (Jest)

```typescript
// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```
