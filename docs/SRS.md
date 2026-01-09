# Software Requirements Specification (SRS)
## Distributed Transaction Platform
**Version:** 0.1  
**Status:** Living Document  
**Last Updated:** Initial Design Phase

---

## 1. Introduction

### 1.1 Purpose

This document defines the functional and non-functional requirements for the **Distributed Transaction Platform**, a banking-grade backend system designed to process financial orders and payments reliably in a distributed environment.

The primary purpose of this SRS is to:
- Establish a shared understanding of system behavior
- Capture constraints critical to financial correctness
- Serve as an authoritative reference throughout development

This document intentionally prioritizes **correctness, auditability, and recoverability** over feature completeness.

---

### 1.2 Scope

The system supports:
- Order creation and lifecycle management
- Payment execution with exactly-once logical semantics
- Inventory reservation as part of a distributed transaction
- Event-driven coordination using Kafka
- Immutable financial ledger recording
- Periodic reconciliation for operational safety

The system does **not** include UI, real payment gateways, or customer-facing applications.

---

### 1.3 Definitions & Acronyms

| Term | Meaning |
|----|--------|
| Saga | Pattern for managing distributed transactions |
| Outbox | Table used to safely publish events |
| Ledger | Append-only record of financial movements |
| Idempotency | Ability to safely retry operations |
| EOD | End of Day reconciliation |

---

## 2. System Overview

### 2.1 Architectural Style

- Microservices architecture
- Event-driven communication
- Database-per-service
- Saga orchestration (central coordinator)
- No two-phase commit (2PC)

---

### 2.2 High-Level Components

- API Gateway (optional deployment-time component)
- Order Service (Saga Orchestrator)
- Payment Service (Financial execution)
- Inventory Service (Resource reservation)
- Apache Kafka (Event backbone)
- Reconciliation Service (Audit & verification)

---

## 3. Functional Requirements

### 3.1 Order Management

**FR-1:** The system shall allow clients to create orders via a REST API.  
**FR-2:** Order creation shall be idempotent.  
**FR-3:** Orders shall progress through deterministic states (e.g., CREATED, CONFIRMED, CANCELLED).  
**FR-4:** Orders and corresponding events shall be persisted atomically using the Transactional Outbox pattern.

---

### 3.2 Payment Processing

**FR-5:** The system shall process payments exactly once at the logical level.  
**FR-6:** Duplicate payment requests or events shall not result in duplicate financial charges.  
**FR-7:** All monetary values shall be represented using `BigDecimal` or fixed-point integers.  
**FR-8:** Every financial operation shall result in an immutable ledger entry.

---

### 3.3 Inventory Management

**FR-9:** Inventory shall be reserved as part of the order saga.  
**FR-10:** Inventory reservations shall be released on saga compensation.  
**FR-11:** Inventory updates shall be concurrency-safe.

---

### 3.4 Event Processing

**FR-12:** Services shall communicate asynchronously using Kafka.  
**FR-13:** All event consumers shall be idempotent.  
**FR-14:** Events shall be versioned to support evolution.

---

### 3.5 Reconciliation

**FR-15:** The system shall periodically reconcile orders, payments, and ledger entries.  
**FR-16:** Discrepancies shall be logged and reported for investigation.

---

## 4. Non-Functional Requirements

### 4.1 Reliability & Correctness

- No financial operation may be executed more than once.
- The system must tolerate duplicate messages and retries.
- Services must be restart-safe.

---

### 4.2 Performance

- Order creation latency should be under 200 ms under normal load.
- Asynchronous processing may complete eventually.

---

### 4.3 Availability

- The system shall be designed for ≥99.9% availability.
- Partial failures shall not corrupt data.

---

### 4.4 Security & Compliance

- No sensitive data shall be logged in plaintext.
- Secrets shall not be committed to version control.
- Principle of least privilege shall be enforced.

---

### 4.5 Auditability

- All financial actions must be traceable.
- Ledger data must be immutable.
- System behavior must be explainable post-facto.

---

## 5. Constraints

- Floating-point types are forbidden for monetary values.
- Two-phase commit (2PC) is forbidden.
- Kafka is internal-only and not client-facing.
- Each service owns its data exclusively.

---

## 6. Assumptions

- Moderate traffic volume
- Single-region deployment
- Trusted internal network between services
- Infrastructure failures are expected

---

## 7. Out of Scope

- Frontend/UI
- Real payment gateways
- PCI compliance
- Multi-region replication
- User management

---

## 8. Future Enhancements

- Event sourcing
- Multi-region failover
- Service mesh with mTLS
- Automated reconciliation dashboards

---

## 9. Document Evolution

This SRS is a **living document** and will be updated as:
- Failure modes are discovered
- Architectural decisions are finalized
- Operational learnings emerge