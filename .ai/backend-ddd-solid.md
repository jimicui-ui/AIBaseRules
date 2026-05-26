# Backend: DDD & SOLID

Apply when designing or changing backend services, use cases, domain logic, or infrastructure adapters.

## Layered architecture (DDD)

| Layer | Responsibility | May depend on |
|-------|----------------|---------------|
| **Domain** | Entities, value objects, aggregates, domain events, invariants, domain services | Nothing outside Domain |
| **Application** | Use cases, commands/queries, handlers, orchestration, ports (interfaces) | Domain only |
| **Infrastructure** | Persistence, messaging, HTTP clients, cloud SDKs — **adapters** | Application + Domain abstractions |
| **Presentation** | Minimal APIs, controllers, DTO mapping, auth middleware | Application (not Domain internals directly) |

### Rules

- **Ubiquitous language:** Names in code match domain terms (Order, Shipment, not `DataRow`, `ProcessItem`).
- **Aggregates:** One aggregate root per consistency boundary; external references by id only, not mutable entity graphs.
- **Domain purity:** No `DbContext`, `HttpClient`, `ILogger`, or framework attributes in Domain.
- **Use cases in Application:** One handler/class per command or query; no business rules in controllers or `Program.cs`.
- **Ports and adapters:** Application defines interfaces (repositories, gateways); Infrastructure implements them.
- **Thin edges:** Presentation validates transport shape only; invariants and business validation live in Domain/Application.

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

// ✅ Handler orchestrates ports
public class CreateOrderHandler(IOrderRepository orders, IPaymentGateway payments) {
    public async Task Handle(CreateOrderCommand cmd, ...) {
        var order = Order.Create(cmd.Items);
        await payments.ChargeAsync(...);
        await orders.AddAsync(order, ...);
    }
}
```

## Before finishing a change

1. Classify each touched file: Domain, Application, Infrastructure, or Presentation.
2. Confirm dependency direction (inward only; no Domain → Infrastructure).
3. If logic landed in the wrong layer, move it — do not duplicate rules across layers.
