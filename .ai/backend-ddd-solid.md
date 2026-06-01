# Backend: DDD & SOLID

Apply when designing or changing backend services, use cases, domain logic, or infrastructure adapters.

## Layered architecture (DDD)

| Layer | Responsibility | May depend on |
|-------|----------------|---------------|
| **Domain** | Entities, value objects, aggregates, domain events, invariants, domain services | Nothing outside Domain |
| **Application** | Use cases, commands/queries, handlers — **orchestration only**, ports (interfaces) | Domain only |
| **Infrastructure** | Persistence, messaging, HTTP clients, cloud SDKs — **adapters** | Application + Domain abstractions |
| **Presentation** | Minimal APIs, controllers, DTO mapping, auth middleware | Application (not Domain internals directly) |

### Rules

- **Ubiquitous language:** Names in code match domain terms (Order, Shipment, not `DataRow`, `ProcessItem`).
- **Aggregates:** One aggregate root per consistency boundary; external references by id only, not mutable entity graphs.
- **Domain purity:** No `DbContext`, `HttpClient`, `ILogger`, or framework attributes in Domain.
- **Use cases in Application:** One handler/class per command or query; no business rules in controllers or `Program.cs`.
- **Application orchestrates only:** Handlers load aggregates, call domain methods, and invoke ports — no core business rules, calculations, or domain conditionals in handlers. If any are found, **refactor into Domain entities** (or domain services) before adding or extending tests.
- **Ports and adapters:** Application defines interfaces (repositories, gateways); Infrastructure implements them.
- **Thin edges:** Presentation validates transport shape only; invariants and business validation live in **Domain**.

## Event-driven architecture (EDA)

Design EDA from **business and system requirements**, not from messaging technology. For each capability, decide what must be **consistent and immediate** for the user or business vs what can be **eventually consistent** side work.

| Concern | Approach |
|---------|----------|
| **Requirement analysis** | Map user journeys and business rules first; label each step as commit-critical (must succeed/fail in the request) or side effect (can follow asynchronously). |
| **Payment and money movement** | **Synchronous** call on the critical path (e.g. `IPaymentGateway.ChargeAsync` inside the use case). The caller gets a definitive outcome before the operation is considered successful. |
| **Non-critical side effects** | **EDA** — notifications, search indexes, analytics, downstream integrations, read-model updates. Publish **after** the aggregate/state change is committed. |
| **Reliable publish** | **Transactional outbox** — persist domain changes and outbox rows in the **same database transaction**; a separate process publishes to the bus and marks rows sent. |
| **Delivery and retry** | Assume **at-least-once** delivery; consumers must be **idempotent**. Retries use backoff; after max attempts, route to **dead-letter** for manual replay — never lose messages silently. |

### Rules

- Do not put **payment authorization/capture** on a fire-and-forget event; payment failure must surface in the same request/transaction boundary as the business operation (or use an explicit saga with compensations — still modeled, not “hope the consumer charges”).
- Raise **domain events** for facts that already happened inside the aggregate; Application commits, then outbox (or equivalent) carries integration events to the outside world.
- **Outbox before bus:** never publish to a message broker inside the same handler without outbox (or inbox+outbox) if you cannot afford lost messages on process crash between DB commit and publish.
- **Safe retry:** retry only on transient failures; use deterministic idempotency keys (payment id, order id, event id) so duplicate deliveries do not double-charge or double-ship.
- Side-effect handlers live in **Infrastructure** (or dedicated workers); **Domain** stays unaware of brokers and outbox tables.

### Example (.NET)

```csharp
// ✅ Payment on sync path; side effects via outbox after commit
public class PlaceOrderHandler(
    IOrderRepository orders,
    IPaymentGateway payments,
    IUnitOfWork uow) {
    public async Task Handle(PlaceOrderCommand cmd, CancellationToken ct) {
        var order = Order.Place(cmd.Items);
        var paymentResult = await payments.ChargeAsync(order.Total, order.Id, ct);
        if (!paymentResult.Succeeded)
            throw new PaymentFailedException(paymentResult.Reason);

        order.MarkPaid(paymentResult.TransactionId);
        await orders.AddAsync(order, ct);
        uow.EnqueueOutbox(new OrderPlacedIntegrationEvent(order.Id, order.CustomerId));
        await uow.CommitAsync(ct); // order + outbox in one transaction
    }
}
// Background worker: read outbox → publish → mark sent; DLQ on permanent failure
```

## SOLID (implementation)

| Principle | Practice |
|-----------|----------|
| **S** — Single responsibility | One reason to change per class (e.g. `CreateOrderHandler` vs `OrderRepository`). |
| **O** — Open/closed | Extend behavior via new types/strategies; avoid editing stable Domain code for every variant. |
| **L** — Liskov substitution | Implementations honor contract pre/post-conditions; no surprising throws or weakened guarantees. |
| **I** — Interface segregation | Small ports (`IOrderRepository`, `IEmailSender`) — not a single `IService` god-interface. |
| **D** — Dependency inversion | High-level modules depend on abstractions; wire concretions only in composition root (`Program.cs`). |

## Examples (.NET)

```csharp
// ❌ Domain coupled to EF
public class Order {
    public void Save(DbContext db) => db.SaveChanges();
}

// ✅ Domain expresses behavior; persistence is outside
public class Order {
    public void Confirm() { /* invariant checks */ }
}
// IOrderRepository in Application; implementation in Infrastructure
```

```csharp
// ❌ Application handler does SQL and HTTP
public class CreateOrderHandler {
    public async Task Handle(...) {
        using var conn = new SqlConnection(...);
        await http.PostAsync(...);
    }
}

// ❌ Business rules in handler
public class CreateOrderHandler {
    public async Task Handle(CreateOrderCommand cmd, ...) {
        if (cmd.Items.Sum(i => i.Price) < 10) throw new InvalidOperationException("Minimum order");
        // discount logic, status transitions, etc. in handler
    }
}

// ✅ Handler orchestrates; rules live on Order
public class CreateOrderHandler(IOrderRepository orders, IPaymentGateway payments) {
    public async Task Handle(CreateOrderCommand cmd, ...) {
        var order = Order.Create(cmd.Items); // minimum, discounts, invariants inside Domain
        await payments.ChargeAsync(order.Total, ...);
        await orders.AddAsync(order, ...);
    }
}
```

## Before finishing a change

1. Classify each touched file: Domain, Application, Infrastructure, or Presentation.
2. Confirm dependency direction (inward only; no Domain → Infrastructure).
3. If logic landed in the wrong layer, move it — do not duplicate rules across layers.
4. If the flow touches **payment** or **async side effects**: payment stays **sync** on the critical path; side effects use **outbox + idempotent consumers** with safe retry and dead-letter — not naked publish-after-commit.
