# Example 04 - Parsing and Routing

## Objective

Most logging systems begin by collecting logs.

Mature logging systems begin organizing them.

This example demonstrates how rsyslog can inspect messages and route them to different destinations based on their contents.

---

## Why Routing Matters

Without routing:

```plaintext
Everything → /var/log/messages
```

Over time this becomes:

- Difficult to search
- Difficult to troubleshoot
- Difficult to secure

As volume grows, separating data becomes essential.

---

## Architecture

```plaintext
Incoming Log
      ↓
Property Inspection
      ↓
Routing Rules
      ↓
Destination Files
```

---

## Program-Based Routing

Example:

```plaintext
if $programname == "sshd"
```

Sample message:

```plaintext
Jan 12 10:00:00 host01 sshd[1234]: Failed password for root
```

Extracted value:

```plaintext
programname = sshd
```

Result:

```plaintext
/var/log/ssh.log
```

---

## Routing by Severity

Syslog includes severity levels:

| Value | Severity |
|---------|----------|
| 0 | Emergency |
| 1 | Alert |
| 2 | Critical |
| 3 | Error |
| 4 | Warning |
| 5 | Notice |
| 6 | Informational |
| 7 | Debug |

---

Example:

```plaintext
if ($syslogseverity <= 3)
```

Captures:

- Errors
- Critical events
- Alerts
- Emergencies

Destination:

```plaintext
/var/log/errors.log
```

---

## Content-Based Routing

Example:

```plaintext
if ($msg contains "Failed password")
```

Sample message:

```plaintext
Failed password for root from 10.1.1.50
```

Destination:

```plaintext
/var/log/auth-failures.log
```

This is a simple example of detection-oriented logging.

---

## Why Parsing Is Difficult

Developers often assume logs are predictable.

Reality:

```plaintext
Failed password
```

Can become:

```plaintext
Authentication failed
```

Or:

```plaintext
Login rejected
```

Simple string matching breaks easily.

---

## The Role of Templates

Templates define output structure.

Example:

```plaintext
template(
    name="StandardFormat"
)
```

Output:

```plaintext
timestamp hostname program message
```

Templates help normalize data before storage.

---

## Understanding `stop`

Example:

```plaintext
if $programname == "sshd" then {
    ...
    stop
}
```

Without `stop`:

```plaintext
SSH Log
    ↓
ssh.log
    ↓
messages
```

With `stop`:

```plaintext
SSH Log
    ↓
ssh.log
```

This prevents duplicate processing.

---

## Testing

Generate sample messages:

```bash
logger -t sshd "Failed password for root"
```

```bash
logger -t sudo "User executed privileged command"
```

```bash
logger -p user.err "Application failure detected"
```

Verify logs appear in the expected destinations.

---

## Testing with Repository Datasets

Recommended:

```plaintext
datasets/sample_logs/mixed_syslog.log
```

Observe:

- Program extraction
- Severity routing
- Content matching

---

For parser resilience testing:

```plaintext
datasets/sample_logs/malformed.log
```

Observe:

- Rule failures
- Missing fields
- Unexpected formats

---

## Common Mistakes

### Over-Reliance on Message Text

Example:

```plaintext
contains "error"
```

May miss:

```plaintext
failure
critical
fault
```

---

### Excessive Rule Complexity

Large rule chains become:

- Difficult to maintain
- Slow to process
- Error-prone

---

### Ignoring Unknown Data

Always maintain a catch-all destination:

```plaintext
/var/log/messages
```

Unknown logs often become important later.

---

## Key Lessons

- Routing improves organization.
- Parsing introduces complexity.
- Content-based rules are fragile.
- Templates create consistency.
- Always keep a catch-all path.

---

## Related Chapters

- Chapter 06 – Parsing and Normalization
- Chapter 07 – Scaling Syslog
- Chapter 08 – Failure Modes
