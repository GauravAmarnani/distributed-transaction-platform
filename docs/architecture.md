# Architecture Overview
## Distributed Transaction Platform

---

## 1. Architectural Goals

The architecture of this platform is designed to satisfy the following goals:

- Financial correctness under failure
- Deterministic transaction processing
- Auditability and traceability
- Loose coupling between services
- Operational recoverability

These goals take precedence over simplicity or development speed.

---

## 2. High-Level Architecture Diagram

![Distributed Transaction Platform Architecture](../assets/architecture.png)

---

## 3. Component Responsibilities

### 3.1 Client

- External consumer of the system
- Interacts via HTTP/REST only
- Never communicates directly with Kafka or internal databases

---

### 3.2 API Gateway (Optional)

- Authentication and authorization
- Rate limiting
- Request validation
- Acts as the external boundary of the system

The gateway is optional and may be bypassed in local or simplified deployments.

---

### 3.3 Order Service (Saga Orchestrator)

The Order Service is the **central coordinator** of distributed workflows.

Responsibilities:
- Accept order creation requests
- Persist orders and outbox events atomically
- Publish domain events asynchronously
- Track order state transitions
- React to downstream success/failure events

The Order Service owns:
- Orders table
- Outbox events table

---

### 3.4 Transactional Outbox Publisher

- Reads pending events from the outbox table
- Publishes events to Kafka
- Marks events as published

This component eliminates dual-write inconsistencies between the database and Kafka.

---

### 3.5 Apache Kafka

Kafka serves as the **internal event backbone**.

Key characteristics:
- Asynchronous communication
- At-least-once delivery
- Decoupling between producers and consumers

Kafka is not exposed to external clients.

---

### 3.6 Payment Service

The Payment Service is responsible for **financial execution**.

Responsibilities:
- Consume order events idempotently
- Execute payment logic exactly once
- Record immutable ledger entries
- Emit payment success or failure events

The Payment Service owns:
- Payments table
- Idempotency keys table
- Ledger table (append-only)

---

### 3.7 Inventory Service (Saga Participant)

The Inventory Service manages stock reservations.

Responsibilities:
- Reserve inventory on order creation
- Release inventory on compensation
- Enforce concurrency safety
- Participate in saga workflows

The Inventory Service owns:
- Inventory/stock table
- Idempotency keys table

---

### 3.8 Reconciliation Service

The Reconciliation Service acts as an **operational safety net**.

Responsibilities:
- Periodically verify consistency between:
  - Orders
  - Payments
  - Ledger entries
- Detect silent failures
- Produce alerts or reports

This service acknowledges that real-time systems are eventually consistent and may fail silently.

---

## 4. Communication Patterns

| Interaction | Type |
|----|----|
| Client → Order Service | Synchronous HTTP |
| Order Service → Kafka | Asynchronous (Outbox) |
| Kafka → Payment / Inventory | Asynchronous |
| Payment → Order Service | Asynchronous events |
| Reconciliation → Databases | Read-only |

---

## 5. Failure Handling Philosophy

The architecture assumes:
- Duplicate events
- Partial failures
- Service restarts

Correctness is achieved through:
- Idempotent processing
- Immutable data structures
- Explicit compensation
- Reconciliation

Failures are expected, not exceptional.

---

## 6. Architectural Tradeoffs

| Decision | Tradeoff |
|----|----|
| Saga over 2PC | Complexity over blocking |
| Event-driven | Eventual consistency |
| Immutable ledger | Higher storage usage |
| Outbox pattern | Extra tables & logic |

All tradeoffs are intentional and documented via ADRs.

---

## 7. Evolution Strategy

The architecture is designed to evolve toward:
- Managed Kafka
- Kubernetes orchestration
- Service mesh with mTLS
- Advanced observability

Core correctness guarantees will remain unchanged.