# 01 – Basic Configuration Notes

This example introduces rsyslog using an explicit pipeline model:

**Input → Ruleset → Action**

This is how rsyslog actually processes messages internally. Understanding this early will make every advanced configuration easier to reason about.

---

## What This Configuration Does

- Receives logs from:
  - The local system (`imuxsock`)
  - The kernel (`imklog`)
- Routes all messages into a single ruleset: `local_logs`
- Writes logs to files in `/var/log/` based on facility
- Sends emergency messages to all logged-in users

---

## The Processing Model

### 1. Inputs (Where Logs Enter)

```bash
input(type="imuxsock" ruleset="local_logs")
input(type="imklog" ruleset="local_logs")
```


Inputs define:
- **Where logs come from**
- **Which ruleset processes them**

At this stage, all logs are local.

---

### 2. Ruleset (How Logs Are Processed)

```bash
ruleset(name="local_logs") { ... }
```


The ruleset acts as a **pipeline**:
- Messages are evaluated as they pass through
- Conditions determine what happens next
- Multiple actions can be triggered per message

This replaces the older “top-to-bottom rules” mental model with something more accurate:
> Logs flow through a processing pipeline.

---

### 3. Actions (What Happens to Logs)

Example:

```bash
action(
type="omfile"
file="/var/log/syslog"
)
```


Actions define:
- Where logs go (files, remote systems, users)
- How they are written (via templates)

Multiple actions can run for the same message.

---

## Facilities and Filtering

This configuration uses `$syslogfacility-text` to route logs:

- `auth`, `authpriv` → `/var/log/auth.log`
- `kern` → `/var/log/kern.log`
- `mail` → `/var/log/mail.log`
- `cron` → `/var/log/cron.log`
- everything else → `/var/log/syslog`

Example condition:

```bash
if ($syslogfacility-text == "kern") then { ... }
```

This is more explicit than traditional syntax and easier to extend later.

---

## Templates

A basic template is defined:

```bash
template(name="TraditionalFormat" type="string"
string="%TIMESTAMP% %HOSTNAME% %syslogtag%%msg%\n")
```


Templates control:
- Output format
- Structure of log messages

This becomes critical when moving to JSON and structured logging.

---

## What This Configuration Does NOT Do

This setup is intentionally minimal.

It does NOT include:
- Remote forwarding
- Buffering or queues
- Retry logic
- Flow control
- Encryption

---

## Failure Characteristics

This configuration has **no durability guarantees**.

### If rsyslog stops:
Logs are lost.

### If disk fills:
Writes fail silently.

### If system is under heavy load:
Messages may be dropped before processing.

---

## What to Observe

Generate a test message:

```bash
logger "test message"
```


Templates control:
- Output format
- Structure of log messages

This becomes critical when moving to JSON and structured logging.

---

## What This Configuration Does NOT Do

This setup is intentionally minimal.

It does NOT include:
- Remote forwarding
- Buffering or queues
- Retry logic
- Flow control
- Encryption

---

## Failure Characteristics

This configuration has **no durability guarantees**.

### If rsyslog stops:
Logs are lost.

### If disk fills:
Writes fail silently.

### If system is under heavy load:
Messages may be dropped before processing.

---

## What to Observe

Generate a test message:

```bash
logger "test message"
```

Check output:

```bash
tail -f /var/log/syslog
```

Verify routing:

Auth logs → `/var/log/auth.log`
Kernel logs → `/var/log/kern.log`

Failure Exercise (Recommended)

1. Stop rsyslog:

```bash
systemctl stop rsyslog
```

2. Generate logs:

```bash
logger "this will be lost"
```

3. Start rsyslog:

```bash
systemctl start rsyslog
```

4. Check logs:

The messages are gone
