---
title: "Week 10 Worklog"
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objective: Finalize the architecture

#### Work activities

- The team conducted an internal presentation session:

  - Presented the data flow from click to sessionization.
  - Demonstrated event flow, window time, session logic, and user detection methods.

- Consolidated feedback and agreed on the final architecture version:

  - Real-time data flow.
  - Session cut-off mechanism (30 minutes idle time).
  - Storage layers: raw, processed, sessionized.

- Verified feasibility for real-world deployment (scalability, latency, fault tolerance).

#### Results

- Final architecture completed.
- Clearly defined pipeline: ingestion → processing → session creation → storage → analytics.
- Created complete documentation to move to the proposal phase and cost estimation.

Link sheet report team https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing
