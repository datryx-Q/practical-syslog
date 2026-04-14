# 01 – Introduction

Logging systems are trusted the most when everything else is failing.

When an incident happens, engineers turn to logs to answer basic questions:

- What happened?
- When did it happen?
- What changed?
- What broke?

The expectation is simple: logs will be there, complete and accurate.

That expectation is often wrong.

---

## The Assumption Problem

Most systems treat logging as background infrastructure:
- Set it up once
- Assume it works
- Ignore it until something breaks

But logging pipelines are not passive systems.

They are:
- High-throughput data streams
- Distributed across multiple systems
- Dependent on networks, disks, and downstream services

Every one of those components can fail.

---

## What People Think Happens

A typical mental model looks like this:
