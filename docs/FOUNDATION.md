----

## 1. Purpose & Intent

### 1.1 Primary Intent

The purpose of this project is to design and implement a **banking-grade distributed backend system** that demonstrates how **financial transactions, orders, and state changes** are handled in a **highly regulated, failure-prone, enterprise environment**.

The system is not optimized for:

* Feature velocity
* UI richness
* Startup-style experimentation

Instead, it is optimized for:

* **Correctness**
* **Auditability**
* **Deterministic behavior**
* **Operational recoverability**

This mirrors the priorities of large financial institutions such as investment banks.

---

### 1.2 What This Project Proves

This project is intended to prove that the engineer understands:

* Why financial systems must behave deterministically
* How distributed systems fail in real life
* How banks prevent silent data corruption
* How retries, duplicates, crashes, and restarts are handled safely
* How engineering decisions are documented and justified

The goal is not to show *how much* technology is used, but **how responsibly it is used**.

---

## 2. Core Design Philosophy

### 2.1 Correctness Over Convenience

If there is a tradeoff between:

* Simpler implementation
* Financial correctness

**Financial correctness always wins**, even at the cost of additional complexity.

---

### 2.2 Failure Is the Default Assumption

The system is designed under the assumption that:

* Services will crash
* Messages will be duplicated
* Brokers may be temporarily unavailable
* Deployments may interrupt in-flight work

The system must **recover safely without manual intervention**.

---

### 2.3 Auditability Is a First-Class Concern

All financial actions must:

* Be traceable
* Be explainable after the fact
* Leave immutable evidence of what occurred

If an action cannot be audited later, it is considered **incorrect**, even if it “worked” in real time.

---

## 3. Non-Negotiable Technical Constraints

These constraints **must never be violated**, regardless of timeline pressure.

---

### 3.1 Financial Data Representation (“No Floats Rule”)

* **Floating-point types (`float`, `double`) are strictly forbidden for monetary values**
* All monetary values must be represented using:

  * `BigDecimal` with explicit `RoundingMode`, **or**
  * Fixed-point integers (`Long`) representing smallest currency units

**Rationale:**
Floating-point arithmetic introduces rounding errors that are unacceptable in financial systems and lead to reconciliation failures.

---

### 3.2 Exactly-Once *Logical* Financial Execution

* A financial operation (e.g., payment) must be **logically executed exactly once**
* The system must tolerate:

  * Duplicate API calls
  * Duplicate Kafka messages
  * Retries after partial failures

**Implementation expectation:**

* Idempotency keys
* Deduplication checks
* Transactional boundaries

---

### 3.3 No Dual Writes (Transactional Outbox Required)

* Writing to a database and publishing to a message broker **must never occur as separate, uncoordinated actions**
* All outbound events must be persisted first and published asynchronously

**Mandatory pattern:**

* Transactional Outbox

**Rationale:**
Prevents data loss and inconsistent state when the database succeeds but messaging fails.

---

### 3.4 Immutable Financial Ledger

* All financial movements must be recorded in an **append-only ledger**
* Ledger entries must:

  * Never be updated
  * Never be deleted
* Corrections are represented as **new compensating entries**

**Rationale:**
This mirrors real banking ledgers and enables audit, reconciliation, and regulatory review.

---

### 3.5 Deterministic Service Restart Behavior

* Any service may be restarted at any time
* On restart, the service must:

  * Resume safely
  * Not duplicate financial actions
  * Not lose pending work

If restarting a service can corrupt data, the design is invalid.

---

## 4. Architectural Constraints

### 4.1 Event-Driven Architecture

* Services communicate asynchronously using events
* Direct synchronous chaining across services is avoided for core workflows

**Rationale:**
Event-driven systems provide resilience, decoupling, and better failure isolation.

---

### 4.2 Database-Per-Service

* Each service owns its data
* No service reads another service’s database
* Cross-service consistency is handled via events and reconciliation

---

### 4.3 Explicit Distributed Transaction Management

* Two-Phase Commit (2PC) is **explicitly forbidden**
* Distributed workflows are handled using:

  * Saga Pattern
  * Orchestrated coordination

---

## 5. Security & Compliance Baselines

### 5.1 Sensitive Data Handling

* No sensitive information (PANs, secrets, credentials) is logged in plaintext
* Logs must mask sensitive fields by default

---

### 5.2 Secrets Management

* No credentials are hardcoded or committed to version control
* Secrets must be injected via:

  * Environment variables, or
  * Managed secret stores (e.g., AWS Secrets Manager)

---

### 5.3 Principle of Least Privilege

* Services only have access to the resources they require
* Elevated permissions must be justified and documented

---

## 6. Operational Safety Nets

### 6.1 Reconciliation Is Mandatory

* The system must include a reconciliation mechanism that:

  * Periodically verifies consistency across services
  * Detects mismatches between orders, payments, and ledger entries
  * Flags discrepancies for investigation

**Rationale:**
Banks assume real-time systems can fail silently. Reconciliation is the last line of defense.

---

## 7. Explicit Non-Goals

The following are intentionally out of scope:

* UI or frontend development
* Mobile applications
* Real card processing or PCI compliance
* Multi-region or active-active deployment
* Ultra-low latency optimization
* AI/ML components

These are excluded to preserve focus on **backend correctness and reliability**.

---

## 8. Documentation Discipline

* All major architectural decisions must be captured as **Architecture Decision Records (ADRs)**
* ADRs must include:

  * Context
  * Decision
  * Alternatives considered
  * Consequences

Code without documented reasoning is considered incomplete.

---

## 9. Success Criteria

This project is considered successful if:

* Financial actions are never duplicated
* Failures do not corrupt state
* All money movement can be reconstructed from the ledger
* The system can explain *why* something happened, not just *what* happened
* Design tradeoffs are clearly documented

---

## Closing Statement

This project intentionally prioritizes **banking realism over feature count**.
Any future enhancements must comply with the constraints defined here.

If a proposed change violates this document, it must be rejected or explicitly re-designed.

---
