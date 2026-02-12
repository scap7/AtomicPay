# AtomicPay

**Concurrency-Safe, Failure-Resilient Payment Processing System**

AtomicPay is a backend system designed to ensure **financial transaction correctness under concurrency, retries, and system failures**.

It demonstrates how real-world systems achieve *exactly-once behavior* on top of infrastructure that only guarantees *at-least-once execution*.

---

## 🚩 Problem Statement

In distributed systems, the following failures are normal:

* Clients retry requests after timeouts
* Background jobs execute more than once
* Servers crash mid-transaction
* Duplicate messages are delivered
* Concurrent workers update the same data
* External gateways respond inconsistently

Without careful design, this leads to:

❌ Double charges
❌ Lost updates
❌ Inconsistent payment states
❌ Data corruption under race conditions

AtomicPay is built to **prevent these failures through engineering controls**.

---

## 🎯 Objectives

AtomicPay showcases:

* Idempotent API design
* Database-driven concurrency control
* Safe retry orchestration
* Exactly-once behavioral guarantees
* State-machine–enforced transitions
* Failure recovery through reconciliation
* Designing for correctness under race conditions

This project emphasizes **reliability over features**.

---

## 🏗 System Architecture

```
Client Request
     ↓
Rails API (Idempotency Enforcement)
     ↓
PostgreSQL (Transactional Guarantees)
     ↓
Sidekiq Worker (Async Processing)
     ↓
Gateway Simulator (External Dependency)
     ↓
State Transition Engine
     ↓
Reconciliation Worker (Consistency Repair)
```

---

## ⚙️ Tech Stack

| Layer               | Technology                               |
| ------------------- | ---------------------------------------- |
| Framework           | Ruby on Rails (API-only)                 |
| Database            | PostgreSQL (MVCC concurrency model)      |
| Queue               | Sidekiq + Redis                          |
| Concurrency Control | SQL constraints + transactional updates  |
| Architecture        | Service Objects + Explicit State Machine |
| Reliability Model   | Idempotency + Retry + Reconciliation     |

---

## 🔐 Core Engineering Concepts Demonstrated

### 1️⃣ Idempotency — Safe Handling of Duplicate Requests

AtomicPay ensures:

> The same request processed multiple times produces **one and only one effect**.

Implemented using:

* `Idempotency-Key` tracking
* Unique DB constraints
* Response caching for deterministic replay

---

### 2️⃣ Optimistic Concurrency Control

Prevents race conditions using atomic state transitions:

```sql
UPDATE payments
SET status = 'processing'
WHERE id = ? AND status = 'pending';
```

Only one concurrent worker can succeed.

---

### 3️⃣ Pessimistic Locking (When Required)

Critical paths use row-level locks:

```sql
SELECT * FROM payments WHERE id = ? FOR UPDATE;
```

Ensures serialized execution for financial safety.

---

### 4️⃣ At-Least-Once Job Execution Handling

Sidekiq guarantees jobs may run multiple times.

AtomicPay jobs are designed to be:

* Idempotent
* Re-entrant
* Crash-safe

---

### 5️⃣ State-Machine Driven Consistency

Payment lifecycle:

```
pending → processing → succeeded | failed
```

Invalid transitions are rejected, ensuring correctness even under concurrency.

---

### 6️⃣ Retry Engineering

Retries are treated as **first-class failure scenarios**, not afterthoughts:

* Exponential backoff
* Failure classification
* Safe re-execution
* No duplicate financial effects

---

### 7️⃣ Reconciliation (System Self-Healing)

A periodic worker audits system state to detect:

* Stuck payments
* Partial failures
* Inconsistent records

It safely repairs state using idempotent reprocessing.

---

## 📦 Project Structure

```
app/
 ├── controllers/
 │     payments_controller.rb
 │
 ├── services/
 │     create_payment.rb
 │     process_payment.rb
 │     idempotency_manager.rb
 │
 ├── workers/
 │     payment_worker.rb
 │     reconciliation_worker.rb
 │
 ├── models/
 │     payment.rb
 │     idempotency_key.rb
 │
 └── domain/
       payment_state_machine.rb
```

---

## 🔁 Example Flow

### Request

```
POST /payments
Idempotency-Key: 9c2f-1aa2
```

### Behavior

Even if this request is sent **5 times concurrently**:

✔ Only one payment is created
✔ Only one processing job executes
✔ All responses remain consistent

---

## ▶️ Running Locally

```bash
bundle install
rails db:create db:migrate

redis-server
bundle exec sidekiq

rails server
```

---

## 🧠 What This Project Teaches

AtomicPay demonstrates how to:

* Build systems that remain correct under concurrency
* Use the database as a consistency boundary
* Design idempotent distributed workflows
* Handle retries without duplicating effects
* Engineer exactly-once behavior in unreliable environments
* Think in terms of failure-oriented architecture

---

## 🚀 Future Enhancements

* Multi-node worker simulation
* Event-driven processing (Kafka model)
* Observability dashboards
* Chaos testing scenarios

---

## 📚 Key Insight

Reliable systems are not those that avoid failure —
they are those **designed to behave correctly despite failure**.

AtomicPay is an exploration of that principle.
