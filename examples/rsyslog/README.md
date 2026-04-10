# Rsyslog Examples

This directory contains practical rsyslog configurations designed to demonstrate real-world logging scenarios.

These are not theoretical examples—they are built to reflect how rsyslog behaves under actual conditions, including failure, load, and misconfiguration.

---

## Purpose

The goal of these examples is to move beyond simple configurations and show how rsyslog behaves in production-like environments.

Each example focuses on a specific concept:
- Forwarding logs
- Handling failures
- Scaling under load
- Structuring data
- Securing transport

---

## How to Use These Examples

Each folder represents a stage in building a logging pipeline.

You can:
- Follow them sequentially to build understanding
- Jump to a specific use case
- Copy and adapt configurations into your own environment

Each example typically includes:
- `rsyslog.conf` – the configuration
- `notes.md` – explanation, tradeoffs, and failure scenarios

---

## Example Progression

### 01 – Basic
Minimal rsyslog configuration.

- Local logging only
- Establishes a working baseline

---

### 02 – File to Remote
Forward logs to a remote system using UDP.

- Simple setup
- No delivery guarantees

**Key takeaway:** UDP is fast but unreliable.

---

### 03 – TCP Reliable Forwarding
Switch to TCP transport.

- Connection-based delivery
- Improved reliability over UDP

**Key takeaway:** TCP improves delivery, but does not eliminate loss.

---

### 04 – Disk Buffering
Introduce persistent queues.

- Handles outages
- Prevents immediate data loss

**Key takeaway:** Buffering is critical for reliability.

---

### 05 – High Throughput
Tune rsyslog for high event rates.

- Queue tuning
- Worker threads
- Batch processing

**Key takeaway:** Default configurations will fail under load.

---

### 06 – Filtering and Routing
Route logs based on rules.

- Facility and severity
- Host-based routing
- Content filtering

**Key takeaway:** Pipelines are not linear—they branch.

---

### 07 – JSON Formatting
Convert logs into structured format.

- Template-based JSON output

**Key takeaway:** Structure enables downstream processing.

---

### 08 – Secure Forwarding (TLS)
Encrypt log transport.

- TLS configuration
- Certificate handling

**Key takeaway:** Security adds complexity and failure modes.

---

### 09 – Production Example
A modular, production-style configuration.

- Inputs, rulesets, and outputs separated
- Scalable and maintainable design

**Key takeaway:** Real systems are modular, not monolithic.

---

## Important Notes

- These examples prioritize clarity over completeness.
- Many defaults are not safe for production use.
- Always test configurations under:
  - High throughput
  - Network instability
  - Disk pressure

---

## Suggested Learning Path

1. Start with `01-basic`
2. Move to `03-tcp-reliable-forwarding`
3. Study `04-disk-buffering` carefully
4. Explore `05-high-throughput`
5. Review `09-production-example`

---

## Final Thought

If your rsyslog configuration has never been tested during failure,  
it has not been tested at all.
