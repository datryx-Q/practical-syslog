# practical-syslog
A practical guide to syslog architecture, log ingestion, and pipeline reliability, with a focus on large-scale environments and real-world failure scenarios.

---

## Overview

Logging systems are trusted the most when everything else is failing.

During incidents, engineers depend on logs to reconstruct events, understand failures, and make decisions under pressure. The assumption is simple: logs will be there when you need them.

That assumption is often wrong.

This project accompanies the *Practical Syslog* book and focuses on what actually happens in real environments:
- Log loss under pressure  
- Backpressure and buffering failures  
- Parsing inconsistencies  
- Pipeline bottlenecks  
- Cost vs reliability tradeoffs  

This repository is not just theory. It is designed to be **used, tested, and broken**.

---

## What This Repository Is

This is the **hands-on companion** to the book.

It contains:
- Real syslog configurations (rsyslog, syslog-ng)  
- Parsing and normalization examples  
- Reproducible labs  
- Sample datasets  
- Benchmarking tools  
- A full cloud pipeline implementation  

Everything here is designed to reflect **production realities**, not ideal scenarios.

---

## Who This Is For

This project is for:
- Engineers building or maintaining logging pipelines  
- Security professionals working with SIEM platforms  
- SREs dealing with high-throughput systems  
- Anyone who has ever said: *“Why are the logs missing?”*

---

## How to Use This Repo

You can approach this repository in three ways:

### 1. Follow Along With the Book
Each lab and example aligns with concepts covered in the book.

### 2. Jump Straight to Labs
If you prefer hands-on learning:
- Start with `labs/lab-1-ingestion`
- Progress through parsing and scaling
- Finish with the full cloud pipeline

### 3. Use It As a Reference
Copy, adapt, and test configurations in your own environment.

---

## Repository Structure

syslog-book/
│
├── README.md
├── chapters/
├── examples/
├── labs/
├── datasets/
├── benchmarks/
└── diagrams/

### Key Areas

- **examples/**  
  Practical configurations and parsing techniques.

- **labs/**  
  Step-by-step environments designed to simulate real-world scenarios.

- **datasets/**  
  Sample logs for testing and experimentation.

- **benchmarks/**  
  Tools and scripts for measuring performance (EPS, latency, loss).

- **diagrams/**  
  Visual representations of architectures and data flow.

---

## Labs Overview

The labs are designed to build on each other:

### Lab 1 – Ingestion
Set up a basic syslog pipeline and understand how logs enter the system.

### Lab 2 – Parsing
Normalize and structure logs for downstream use.

### Lab 3 – Scaling
Introduce load, identify bottlenecks, and implement buffering strategies.

### Lab 4 – Cloud Pipeline
Build a production-style pipeline:
- On-prem syslog → cloud ingestion  
- Buffering and retry strategies  
- Transformation and routing  
- Integration with a SIEM  
- Cost and reliability tradeoffs  

This lab represents a **real-world architecture**, not a simplified example.

---

## Design Philosophy

This project is built around a few core principles:

### 1. Assume Failure
Every component in a logging pipeline will fail under the right conditions.

### 2. Logs Are a Data Stream, Not a File
Treat logs as continuous, high-volume data with flow control challenges.

### 3. Backpressure Is Inevitable
If you don’t design for it, you will lose data.

### 4. Perfect Parsing Is a Myth
Focus on consistency and usefulness over perfection.

### 5. Cost Is Part of Architecture
Especially in cloud environments, design decisions directly impact cost.

---

## What This Project Is Not

- A beginner-only introduction to syslog  
- A vendor marketing guide  
- A “perfect architecture” blueprint  

There is no perfect pipeline—only tradeoffs.

---

## Contributing

Contributions are welcome, especially:
- Improvements to configurations  
- Additional lab scenarios  
- Bug fixes and clarifications  
- Real-world failure cases  

If something breaks in your environment, that’s valuable—open an issue.

---

## License

This project is licensed under the Apache 2.0 License.

---

## Final Note

If you take one thing from this project, let it be this:

**Logging systems don’t fail gracefully. They fail silently.**

Design accordingly.
