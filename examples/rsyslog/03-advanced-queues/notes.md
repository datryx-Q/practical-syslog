# Example 03 - Advanced Queue Configuration

## Objective

This example introduces one of the most important concepts in logging systems:

**Queues are what separate reliable systems from fragile systems.**

Without queues, log delivery depends entirely on the availability of downstream systems.

With queues, temporary failures become survivable events.

---

## Architecture

```plaintext
Application
    ↓
Local Input
    ↓
Main Queue
    ↓
Forwarding Queue
    ↓
Remote Collector
```

---

## Why Queues Matter

Imagine a collector outage:

```plaintext
Collector Offline
```

Without a queue:

```plaintext
Logs → Dropped
```

With a queue:

```plaintext
Logs → Buffered → Delivered Later
```

Queues buy time.

---

## Main Queue

```plaintext
main_queue(
    queue.type="LinkedList"
)
```

The main queue sits between ingestion and processing.

Responsibilities:

- Absorb bursts
- Decouple inputs from outputs
- Prevent immediate data loss

---

## Queue Size

```plaintext
queue.size="100000"
```

Defines the maximum number of messages that can be buffered.

Larger queues:

Benefits:
- Better outage tolerance

Risks:
- Increased memory consumption
- Longer recovery times

---

## Watermarks

```plaintext
queue.highWatermark="80000"
queue.lowWatermark="20000"
```

Watermarks determine when rsyslog begins writing queue data to disk.

### High Watermark

When reached:

```plaintext
Queue Usage > 80%
```

rsyslog begins aggressively protecting queue capacity.

### Low Watermark

Normal operation resumes once usage falls below the configured threshold.

---

## Disk-Assisted Queues

```plaintext
queue.maxDiskSpace="5g"
```

When memory queues become full:

```plaintext
Memory Queue
        ↓
Disk Queue
```

This significantly improves durability.

Benefits:

- Survives temporary spikes
- Handles longer outages
- Prevents memory exhaustion

Tradeoffs:

- Increased latency
- Disk I/O overhead

---

## Save On Shutdown

```plaintext
queue.saveOnShutdown="on"
```

Without this setting:

```plaintext
Restart = Lost Queue
```

With this setting:

```plaintext
Restart = Queue Recovery
```

This is critical for production environments.

---

## Worker Threads

```plaintext
queue.workerThreads="4"
```

Allows multiple queue workers.

Benefits:

- Increased throughput
- Better CPU utilization

Tradeoffs:

- Increased complexity
- Possible message reordering

---

## Action Queues

Notice the forwarding action has its own queue:

```plaintext
action(
    type="omfwd"

    queue.type="LinkedList"
)
```

This creates a dedicated forwarding buffer.

Benefits:

- Isolates network failures
- Prevents forwarding problems from affecting ingestion

---

## Infinite Retries

```plaintext
action.resumeRetryCount="-1"
```

Behavior:

```plaintext
Send
  ↓
Fail
  ↓
Retry
  ↓
Retry
  ↓
Retry
```

Until delivery succeeds.

This improves reliability but may create large backlogs.

---

## Testing Scenario 1 - Remote Outage

1. Start the collector.
2. Generate logs.

```bash
logger "normal operation"
```

3. Stop the collector.
4. Continue generating logs.

```bash
for i in {1..1000}; do
    logger "outage test $i"
done
```

Observe:

- Queue growth
- Disk queue creation
- Retry behavior

---

## Testing Scenario 2 - Burst Traffic

Use:

```plaintext
datasets/sample_logs/burst_traffic.log
```

Observe:

- Queue absorption
- Recovery behavior
- Latency impact

---

## Testing Scenario 3 - Sustained Load

Use:

```plaintext
datasets/sample_logs/high_volume.log
```

Observe:

- Queue depth trends
- Disk utilization
- Throughput limits

---

## Common Mistakes

### No Queue

```
Input → Output
```

Most fragile configuration possible.

---

### Queue Too Small

Results:

- Drops under load
- Minimal outage tolerance

---

### No Disk Assistance

Results:

- Memory exhaustion
- Data loss during long outages

---

### No Monitoring

Results:

- Silent queue growth
- Delayed discovery of failures

---

## Key Lessons

- Queues absorb failure.
- Disk-assisted queues improve durability.
- Reliability always comes with resource costs.
- Queue monitoring is mandatory.
- Buffering delays failure—it does not eliminate it.

---

## Related Chapters

- Chapter 04 – Forwarding
- Chapter 05 – Buffering and Queues
- Chapter 07 – Scaling Syslog
