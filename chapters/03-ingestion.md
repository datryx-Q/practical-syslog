# 03 – Ingestion

Ingestion is where logs first enter the system.

It is also where a surprising amount of data is lost, silently and permanently.

Most people assume ingestion is simple:
> An application writes a log, and it is captured.

In reality, ingestion is a collection of fragile entry points, each with its own failure modes.

---

## What “Ingestion” Actually Means

In syslog systems, ingestion refers to:

- Capturing logs from applications
- Receiving logs from the kernel
- Accepting network-based messages
- Feeding those messages into a processing pipeline

It is the **boundary between the system generating logs and the system managing them**.

---

## Primary Ingestion Sources

### 1. Local Socket (`/dev/log`)

Used by most applications on Unix-like systems.

Application → /dev/log → syslog daemon


This is handled by modules such as:

- `imuxsock` (rsyslog)

**Key characteristics:**
- Fast
- Local only
- No persistence

**Failure modes:**
- Socket backlog overflow
- Daemon not running
- Application continues writing into void

---

### 2. Kernel Logging

Kernel messages are captured separately:

Kernel → syslog daemon


Handled by:

- `imklog` (rsyslog)

**Key characteristics:**
- Low-level system visibility
- Critical for debugging system failures

**Failure modes:**
- Buffer overflow under high kernel event rate
- Loss during boot/shutdown windows

---

### 3. Network Ingestion

Logs from other systems:

Remote Host → Network → Syslog Receiver


Protocols:
- UDP (unreliable)
- TCP (reliable-ish)
- TLS (secure transport)

**Failure modes:**
- Packet loss (UDP)
- Connection resets (TCP)
- Queue saturation
- Firewall drops

---

## The Hidden Problem: Ingestion Is Not Durable

A common assumption:

> “If an application writes a log, it is stored somewhere.”

This is false.

At ingestion time:
- There is often no disk write
- No guarantee of persistence
- No retry mechanism

If ingestion fails, the log is gone.

---

## Backpressure at Ingestion

When ingestion is overwhelmed:

- Input buffers fill
- Writes block or fail
- Messages are dropped silently

Example scenario:

High log rate → socket buffer fills → kernel drops messages


This is one of the most common forms of **invisible data loss**.

---

## Ingestion vs Processing

It is important to separate two concepts:

| Stage      | Responsibility |
|------------|---------------|
| Ingestion  | Accept logs    |
| Processing | Transform logs  |

Ingestion should be fast and lightweight.

If it becomes heavy:
- You increase the chance of loss before processing even begins

---

## Why imuxsock Matters

In rsyslog:

module(load="imuxsock")


This module:
- Listens on `/dev/log`
- Receives local application logs
- Feeds them into rulesets

But it does NOT:
- Persist logs
- Retry failed deliveries
- Guarantee delivery

---

## Why imklog Matters

module(load="imklog")


This module:
- Captures kernel messages
- Provides system-level visibility

But it is subject to:
- Kernel buffer limits
- High-frequency event loss

---

## Ingestion Is a Bottleneck

Under high load:

- CPU is not usually the first limit
- Disk is not always the first limit
- **Ingestion buffers are often the first to fail**

Once ingestion fails:
> Everything downstream becomes irrelevant

---

## Common Misconception

> “If logs exist on the machine, they are safe.”

Reality:
- Logs exist temporarily in memory buffers
- They are flushed asynchronously
- They can be dropped before persistence

---

## Why This Matters

If ingestion is unreliable:

- No amount of downstream processing matters
- No SIEM can recover lost logs
- No analysis can reconstruct missing events

You cannot analyze data that never arrived.

---

## Connection to This Repository

This is where your examples begin:

- `01-basic/` shows local ingestion via `imuxsock`
- `multi_host.log` simulates multiple ingestion sources
- `high_volume.log` tests ingestion saturation
- `malformed.log` tests ingestion resilience

---

## Key Insight

Ingestion is not the beginning of reliability.

It is the **first failure boundary** in the system.

---

## What Comes Next

Once logs are ingested, they must move.

That introduces transport, buffering, and forwarding—where most real-world log loss actually occurs.

---

## Final Thought

If ingestion drops a log, nothing downstream will ever know it existed.



