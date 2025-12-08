---
title: "Week 12 Worklog"
weight: 2
chapter: false
pre: " <b> 1.12. </b> "
---

### Week 12 Objective: Finalize Proposal, Test the Pipeline

#### Work activities

- Reviewed and finalized the entire project proposal:

  - Rechecked objectives – scope – architecture – cost.
  - Optimized the description of the data flow and sessionization logic.
  - Refined the proposal for official submission.

- Tested the pipeline at a logical level (no real deployment yet):

  - Tested session creation scenarios:
  - User interacts continuously.
  - User stays idle for 30 minutes.
  - User returns after timeout.
  - Multiple users with multiple concurrent sessions.

- Tested data flow through raw, processed, and sessionized layers.

- Analyzed risks & mitigation strategies:

  - Data stuck at the streaming layer.
  - Lambda timeout under high ingest volume.
  - Increased S3 cost when storing too much raw data.

- Prepared handover documentation and presentation slides:
  - Created architecture slides.
  - Created the final estimated cost table.
  - Wrote conclusion and proposed expansions.

#### Results

- Proposal is 100% completed and ready for official submission.
- Pipeline logic has been tested with multiple scenarios, ensuring its correctness.
- A full documentation package has been created for presentation:
  - Architecture slides
  - Sessionization logic description
  - Cost estimation table
  - Data layer model
  - Completed Risk Assessment and backup plan.

Link sheet report team(https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing)
