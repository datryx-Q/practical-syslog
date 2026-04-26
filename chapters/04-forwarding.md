# 04 – Forwarding

Once logs are ingested, they need to go somewhere.

This is where forwarding begins—and where many logging pipelines start to fail in visible ways.

Forwarding introduces a new dependency:

The network

And with it, a new class of problems:
- Packet loss
- Latency
- Connection instability
- Downstream outages

---

## The Basic Model

Forwarding is often described as:

Sender → Network → Receiver


Simple in theory.

In reality, each part can fail independently.

---

## Why Forward Logs

Logs are forwarded to:

- Centralize visibility
- Enable correlation across systems
- Feed downstream systems (SIEM, analytics platforms)
- Offload storage from local machines

Without forwarding, logs remain siloed and limited in usefulness.

---

## UDP Forwarding (Default Behavior)

The simplest form of syslog forwarding uses UDP.

Sender → UDP → Receiver

### Characteristics

- No connection required
- Low overhead
- High throughput

### Failure Modes

- Packets dropped under load
- No retransmission
- No visibility into loss

If the receiver is unavailable: Messages are silently lost

---

## TCP Forwarding (Improved, Not Perfect)

TCP introduces connection-based delivery.

Sender → TCP → Receiver

### Characteristics

- Connection-oriented
- Flow control (backpressure)
- Better delivery guarantees

### Failure Modes

- Connection resets
- Buffer exhaustion
- Sender blocking under load

TCP improves reliability, but introduces:
- Latency
- Complexity
- New failure modes

---

## TLS Forwarding (Secure Transport)

TLS adds encryption on top of TCP.
Sender → TLS (over TCP) → Receiver

### Benefits

- Encryption in transit
- Authentication of endpoints

### Tradeoffs

- Increased CPU usage
- Certificate management complexity
- Harder troubleshooting

Security improves, but operational burden increases.

---

## The Hidden Problem: Forwarding Without Buffering

A common configuration:

input → action(type="omfwd" target="remote")

This forwards logs immediately.

But if the remote system is unavailable:

- UDP: messages are dropped
- TCP: messages block or fail

Without buffering:
Forwarding is unreliable

---

## Network Reality

Forwarding assumes a stable network. Real networks are not stable.

Common issues:

- Temporary outages
- Packet loss during congestion
- Routing changes
- Firewall interruptions

Even short disruptions can cause:
- Large bursts of lost logs
- Backlog accumulation
- Cascading failures

---

## Backpressure and Its Effects

With TCP, backpressure can occur:

Receiver slows → Sender blocks → Ingestion backs up


This creates a chain reaction:

- Input queues fill
- New messages are dropped
- System performance degrades

Backpressure is necessary but dangerous if unmanaged.

---

## Forwarding at Scale

As volume increases:

- Network bandwidth becomes a constraint
- Connection limits are reached
- Remote systems become bottlenecks

Forwarding is no longer just transport; it becomes a **distributed systems problem**.

---

## Delivery Guarantees (Reality)

Forwarding introduces tradeoffs:

| Method | Behavior |
|-------|--------|
| UDP   | Fast, lossy |
| TCP   | Reliable-ish, can block |
| TLS   | Secure, higher overhead |

You must choose between:

- Performance
- Reliability
- Complexity

You cannot maximize all three.

---

## Common Misconception

“If I use TCP, my logs are safe.”

Reality:
- TCP reduces loss
- It does not eliminate it
- It introduces blocking and queue pressure

---

## Where Things Go Wrong

Most forwarding issues come from:

- No buffering
- Incorrect queue sizing
- Assuming network stability
- Not handling downstream outages
- Overloading receivers

---

## Connection to This Repository

This is where your examples begin to matter:

- `02-file-to-remote/` demonstrates basic forwarding
- `burst_traffic.log` exposes spike related loss
- `high_volume.log` tests sustained throughput
- `duplicate_messages.log` shows retry behavior

---

## Key Insight

Forwarding is the point where logging systems become distributed systems.

And distributed systems fail in ways that are:
- Non-obvious
- Intermittent
- Difficult to reproduce

---

## What Comes Next

Forwarding alone is not enough.

To handle failure, you need buffering and queues mechanisms that absorb disruption instead of dropping data.

---

## Final Thought

Forwarding moves logs.

It does not guarantee they will arrive.

