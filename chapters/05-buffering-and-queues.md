# 05 – Buffering and Queues

Forwarding moves logs.

Buffering and queues determine whether those logs survive failure.

Without queues, logging systems are fragile.
With queues, they become resilient—but only if configured correctly.

---

## Why Queues Exist

Forwarding assumes the destination is always available.

It is not.

Queues exist to absorb the gap between:
- When logs are produced
- When they can be successfully delivered

Producer → Queue → Consumer


This decoupling is the foundation of reliable logging.

---

## What a Queue Does

A queue provides:

- Temporary storage
- Rate smoothing
- Backpressure handling
- Retry capability

It allows the system to continue operating when downstream systems fail.

---

## Types of Queues in rsyslog

### 1. Direct (No Queue)

Input → Output


- No buffering
- Immediate forwarding

**Result:**
- Fast
- Fragile
- Drops logs on failure

---

### 2. Memory Queues

Input → Memory Queue → Output


- Stored in RAM
- Very fast
- Limited capacity

**Failure mode:**
- Data lost on restart
- Overflow under sustained load

---

### 3. Disk Queues

Input → Disk Queue → Output


- Persisted to disk
- Survives restarts
- Slower than memory

**Failure mode:**
- Disk I/O bottlenecks
- Requires proper sizing

---

### 4. Linked (Hybrid) Queues

Input → Memory Queue → Disk Spillover → Output


- Fast under normal conditions
- Durable under pressure

This is the most practical configuration for production systems.

---

## The Hidden Complexity

Queues are not just storage.

They introduce:
- Ordering changes
- Latency
- Retry behavior
- Resource consumption

Improper configuration can:
- Hide problems
- Delay failures
- Create large backlogs

---

## Queue Growth and Backlog

When downstream systems slow or fail:

Output slows → Queue grows → Memory/Disk fills


Eventually:
- The queue reaches capacity
- Messages are dropped
- Or ingestion is blocked

---

## Retry Behavior

Queues enable retries:

Send → Fail → Retry → Success (or drop)


But retries introduce:

- Duplicate messages
- Delayed delivery
- Increased load after recovery

This directly leads to:
> At-least-once delivery semantics

---

## Disk Queue Tradeoffs

Disk queues improve durability, but:

- Increase latency
- Require tuning (size, file count, sync behavior)
- Can become bottlenecks under high throughput

Misconfigured disk queues can:
- Slow the entire pipeline
- Mask performance issues until failure

---

## Ordering Is No Longer Guaranteed

Queues break strict ordering.

Example:

Message A → queued
Message B → processed immediately


Result:
- B arrives before A

This becomes more pronounced with:
- Multiple queues
- Parallel processing

---

## Backpressure (Revisited)

Queues interact with backpressure:

Queue fills → Input slows or drops → System stabilizes (or fails)


Properly configured:
- Prevents total system collapse

Improperly configured:
- Causes silent data loss

---

## The Dangerous Illusion

Queues can make systems appear healthy.

- Logs are accepted
- No immediate errors
- System continues running

But in reality:
- Backlogs are growing
- Latency is increasing
- Failure is being delayed

---

## Monitoring Queues

If you do not monitor queues, you do not have a reliable system.

You must track:

- Queue size
- Queue depth over time
- Drop counts
- Processing latency

Without this:
> You are blind to failure until it is too late

---

## Connection to This Repository

This is where your datasets become powerful:

- `burst_traffic.log` → tests queue spike handling
- `high_volume.log` → tests sustained pressure
- `duplicate_messages.log` → shows retry effects
- `out_of_order.log` → shows ordering issues introduced by queues

---

## Common Misconfigurations

- No queue configured
- Queue too small
- No disk persistence
- No monitoring
- Infinite retry without limits

Each of these leads to different failure modes.

---

## Key Insight

Queues do not eliminate failure.

They shift failure from:
- Immediate loss

To:
- Delayed loss
- Duplication
- Reordering

---

## What Comes Next

Once logs are buffered, they are processed.

This is where structure is imposed on unstructured data—and where parsing failures introduce a new class of problems.

---

## Final Thought

Queues buy you time.

What you do with that time determines whether your logs survive.
