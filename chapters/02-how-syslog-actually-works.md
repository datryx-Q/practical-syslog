# 02 – How Syslog Actually Works

Most explanations of syslog focus on the protocol.

That is not where most problems come from.

To understand syslog in real environments, you need to understand how messages actually move through a system—not how the standard says they should.

---

## The Official Model (Simplified)

At a high level, syslog is described as:
Sender → Transport → Receiver


- A device generates a message  
- It is sent over the network  
- A system receives and stores it  

This model is correct—but incomplete.

---

## The Real Pipeline

In practice, a syslog message goes through multiple stages:
Application
↓
Local Logging Interface (/dev/log)
↓
Syslog Daemon (rsyslog, syslog-ng)
↓
Internal Queues
↓
Output Module (file, network, etc.)
↓
Network
↓
Remote Syslog Daemon
↓
Processing / Parsing
↓
Storage / SIEM


Each stage introduces:
- Latency  
- Failure modes  
- Data transformation  

---

## Message Structure (What You Expect)

A typical syslog message looks like:
In theory, this structure is consistent.

---

## Message Structure (Reality)

In practice, messages vary widely:

- Missing fields  
- Different timestamp formats  
- Inconsistent spacing  
- Embedded JSON  
- Vendor-specific formats  

Example:


2026-01-12T10:15:32Z host01 app: {"event":"login","status":"ok"}


Or worse:


host01 : message with no timestamp


Syslog is not a strict format. It is a loose convention.

---

## Transport: UDP vs TCP

### UDP (Default Behavior)

- Fast  
- No connection  
- No delivery guarantee  

If the receiver is unavailable:
- Messages are dropped

There is no retry.

---

### TCP (Improved, Not Perfect)

- Connection-based  
- Better delivery guarantees  
- Backpressure possible  

But:
- Connections can reset  
- Buffers can overflow  
- Messages can still be lost  

TCP reduces loss—it does not eliminate it.

---

## Internal Queues (Often Ignored)

Inside the syslog daemon, messages are queued before processing.

These queues:
- Absorb bursts  
- Decouple inputs from outputs  

But they are finite.

If a queue fills:
- Messages are dropped  
- Or inputs are blocked  

Queue configuration is one of the most important—and most overlooked—parts of syslog.

---

## Processing and Transformation

Before logs are stored or forwarded, they may be:

- Filtered  
- Reformatted  
- Parsed  
- Enriched  

Each step can:
- Fail  
- Introduce latency  
- Corrupt data  

---

## Forwarding

Forwarding is where many assumptions break.

A common belief:
> “If I forward logs, they will arrive.”

Reality:
- Network interruptions occur  
- Remote systems go down  
- Buffers fill  
- Messages are dropped  

Forwarding without buffering is unreliable.

---

## Delivery Semantics

Syslog does not guarantee:

- Delivery  
- Ordering  
- Uniqueness  

In practice, you get:

- **At-most-once delivery** (UDP): messages may be lost  
- **At-least-once delivery** (TCP + retries): messages may be duplicated  

Exactly-once delivery is not achievable in typical syslog pipelines.

---

## Ordering (The Hidden Problem)

Even if messages are delivered, they may not be in order.

Reasons include:
- Network latency differences  
- Queue delays  
- Clock drift between systems  

This breaks:
- Event timelines  
- Correlation logic  

---

## Time Is Not Reliable

Syslog timestamps depend on the sending system.

Problems:
- Incorrect system clocks  
- Missing time zones  
- Inconsistent formats  

You cannot assume timestamps are:
- Accurate  
- Comparable across systems  

---

## Where Things Go Wrong

Most syslog issues come from:

- Assuming delivery is guaranteed  
- Ignoring queue limits  
- Treating logs as structured when they are not  
- Trusting timestamps blindly  
- Underestimating network instability  

---

## The Key Insight

Syslog is not a reliable transport mechanism.

It is a best-effort messaging system built on:
- Weak guarantees  
- Flexible formats  
- Multiple failure points  

---

## Why This Matters

If you design your pipeline assuming:
- Perfect delivery  
- Clean data  
- Ordered events  

You will lose logs and not realize it.

---

## What Comes Next

Now that the mechanics are clear, the next step is understanding how logs enter the system.

That is where control begins—and where many problems start.

---

## Final Thought

Syslog does not fail because it is broken.

It fails because it behaves exactly as designed—and that design has limits.
