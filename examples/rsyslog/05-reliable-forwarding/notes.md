# Example 05 - Reliable Forwarding

## Objective

Forwarding logs is easy.

Forwarding logs reliably is difficult.

This example demonstrates a production-oriented approach that reduces the risk of data loss during:

- Network interruptions
- Collector outages
- Traffic spikes
- Planned maintenance windows

---

## The Problem

A simple forwarding configuration looks like this:

```plaintext
Application
    ↓
rsyslog
    ↓
Collector
```

If the collector becomes unavailable:

```plaintext
Logs → Lost
```

Many organizations unknowingly operate this way.

---

## Reliable Architecture

This example introduces buffering between the sender and receiver.

```plaintext
Application
    ↓
rsyslog
    ↓
Action Queue
    ↓
TCP Forwarding
    ↓
Collector
```

Now temporary outages become survivable.

---

## Why TCP?

The forwarding action uses:

```plaintext
protocol="tcp"
```

Benefits:

- Connection-oriented
- Flow control
- Better delivery characteristics

Compared to UDP:

| Feature | UDP | TCP |
|----------|-----|-----|
| Connection | No | Yes |
| Retransmission | No | Yes |
| Delivery Visibility | No | Limited |
| Reliability | Low | Higher |

TCP improves delivery but does not eliminate failure.

---

## Action Queues

The forwarding action contains its own queue:

```plaintext
queue.type="LinkedList"
```

This queue buffers messages when:

- The collector is unavailable
- The network is congested
- Delivery slows

Without a queue:

```plaintext
Forwarding Failure
        ↓
Message Loss
```

With a queue:

```plaintext
Forwarding Failure
        ↓
Queue Growth
        ↓
Later Delivery
```

---

## Disk-Assisted Reliability

This example uses:

```plaintext
queue.maxDiskSpace="2g"
```

When memory limits are reached:

```plaintext
Memory Queue
        ↓
Disk Queue
```

Benefits:

- Improved durability
- Better outage tolerance

Tradeoffs:

- Increased latency
- Additional disk I/O

---

## Infinite Retry

```plaintext
action.resumeRetryCount="-1"
```

Behavior:

```plaintext
Send
 ↓
Fail
 ↓
Retry Forever
```

Advantages:

- Maximum persistence

Risks:

- Queue growth
- Disk consumption
- Delayed recovery after long outages

---

## Save On Shutdown

```plaintext
queue.saveOnShutdown="on"
```

Without this setting:

```plaintext
Restart
  ↓
Queue Lost
```

With this setting:

```plaintext
Restart
  ↓
Queue Recovered
```

This significantly improves resilience.

---

## Failure Scenario

Assume the collector becomes unavailable:

```plaintext
Collector Offline
```

What happens?

### Without Queue

```plaintext
Message
   ↓
Forward
   ↓
Drop
```

### With Queue

```plaintext
Message
   ↓
Queue
   ↓
Retry
   ↓
Collector Returns
   ↓
Deliver
```

The difference is operationally significant.

---

## Testing

Generate test traffic:

```bash
logger "reliable forwarding test"
```

Verify delivery on the collector.

---

### Simulate Collector Failure

Stop the remote rsyslog service.

Continue generating messages:

```bash
for i in {1..1000}; do
    logger "outage test $i"
done
```

Observe:

- Queue growth
- Retry behavior
- Disk queue creation

---

### Recovery Test

Restart the collector.

Observe:

```plaintext
Queued Logs
      ↓
Forwarded Logs
      ↓
Queue Drain
```

This validates durability.

---

## Testing with Repository Datasets

Recommended datasets:

### Burst Testing

```plaintext
datasets/sample_logs/burst_traffic.log
```

Observe queue behavior during spikes.

---

### Sustained Load Testing

```plaintext
datasets/sample_logs/high_volume.log
```

Observe:

- Queue depth
- Retry performance
- Throughput limits

---

### Duplicate Detection Testing

```plaintext
datasets/sample_logs/duplicate_messages.log
```

Observe how retries may create duplicate events in downstream systems.

---

## Common Misconceptions

### TCP Guarantees Delivery

False.

TCP improves reliability but cannot protect against:

- Queue exhaustion
- Process crashes
- Disk failures
- Misconfiguration

---

### Infinite Retries Solve Everything

False.

Retries consume:

- Disk
- CPU
- Operational attention

Every queue eventually has limits.

---

### Queues Eliminate Data Loss

False.

Queues delay loss.

They do not eliminate it.

---

## Key Lessons

- Reliability requires buffering.
- TCP is better than UDP for critical logs.
- Disk-assisted queues improve durability.
- Infinite retries must be monitored carefully.
- Queue monitoring is mandatory in production.

---

## Related Chapters

- Chapter 04 – Forwarding
- Chapter 05 – Buffering and Queues
- Chapter 07 – Scaling Syslog
- Chapter 08 – Failure Modes
