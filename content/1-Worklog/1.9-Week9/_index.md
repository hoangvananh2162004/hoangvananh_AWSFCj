---
title: "Week 9 Worklog"
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objective: Build the system architecture

#### Work activities

- Studied AWS services in detail for building a real-time pipeline:

  - Kinesis Stream / Firehose (streaming ingest)
  - AWS Lambda (real-time processing)
  - S3 (storage)
  - Glue / EMR (ETL & analytics)
  - Athena / QuickSight (reporting)

- Each member proposed an architecture design: streaming vs batch, session processing steps, data schema.
- Discussed end-to-end MLOps solutions for processing and automation.

#### Results

- Identified the overall architecture that fits the requirements: ingest → transform → sessionization → storage → analysis.
- The team agreed to choose a streaming-first architecture model.

Link sheet báo cáo team https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing
