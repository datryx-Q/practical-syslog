# Example 02 - Forwarding Logs to a Remote System

## Objective

This example demonstrates how to forward logs from one rsyslog instance to another.

Forwarding is the foundation of centralized logging. Rather than storing logs only on the local machine, events are transmitted to a dedicated collector or aggregation server.

---

## Architecture

```
Application
    ↓
Local rsyslog
    ↓
TCP Forwarding
    ↓
Remote rsyslog
    ↓
Storage / SIEM
```

---

## Modules Used

### imuxsock

```
module(load="imuxsock")
```

Captures logs written to:

```
/dev/log
```

Most Linux applications send logs through this interface.

---

### imklog

```
module(load="imklog")
```

Captures kernel messages.

---

## Forwarding Action

```
action(
    type="omfwd"
    target="192.168.1.100"
    port="514"
    protocol="tcp"
)
```

This sends logs to a remote syslog server.

Parameters:

| Setting | Description |
|----------|-------------|
| target | Destination hostname or IP |
| port | Remote listening port |
| protocol | UDP or TCP |

---

## Why TCP?

Traditional syslog used UDP.

UDP provides:

- Low overhead
- High throughput

But also:

- No delivery confirmation
- No retransmission
- Silent packet loss

TCP provides:

- Connection-based transport
- Flow control
- Better reliability

TCP does not guarantee logs cannot be lost, but it significantly improves delivery success.

---

## Queue Configuration

```
queue.type="linkedList"
queue.filename="forwarding"
queue.saveonshutdown="on"
```

This enables buffering when the remote system is unavailable.

Without a queue:

```
Sender → Receiver
```

With a queue:

```
Sender → Queue → Receiver
```

Queues allow logs to survive:

- Network interruptions
- Remote outages
- Temporary congestion

---

## Infinite Retry

```
action.resumeRetryCount="-1"
```

This instructs rsyslog to continue retrying indefinitely.

Benefits:

- Improved durability

Risks:

- Large queue growth
- Disk consumption
- Delayed delivery

---

## Testing

Generate a test log:

```
logger "forwarding test message"
```

Verify transmission on the remote server.

---

## Failure Testing

Stop the remote rsyslog service and continue generating logs:

```
logger "test during outage"
```

Observe:

- Queue growth
- Retry behavior
- Recovery after the receiver returns

---

## Key Lessons

- Forwarding introduces network dependencies.
- TCP improves reliability but does not eliminate failure.
- Queues are essential for production deployments.
- Logging systems become distributed systems once forwarding is introduced.

---

## Related Chapters

- Chapter 03 – Ingestion
- Chapter 04 – Forwarding
- Chapter 05 – Buffering and Queues
