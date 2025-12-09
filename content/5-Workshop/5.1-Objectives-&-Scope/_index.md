---
title: "Objectives & Scope"
weight: 51
chapter: false
pre: " <b> 5.1. </b> "
---

### Business Context

The target system is an e-commerce website selling laptops, monitors, and accessories.  
The business wants to:

- Understand **how users interact** with the website:
  - Which pages they visit
  - Which products they view
  - Add-to-cart and checkout events
- Measure the **conversion funnel** from product view → add to cart → checkout
- Identify:
  - Top products (best-selling / most viewed)
  - Peak activity windows (by hour, by day)
- Use these insights to design the most effective marketing strategies to:
  - Increase revenue
  - Optimize and reduce costs

To achieve this, the platform:

- Collects a dataset of user behavior via clickstream events, then produces statistics and analytics.

---

### Learning Objectives

#### Understand the Architecture

- Be able to explain the overall architecture of a **batch-based clickstream analytics platform** using:
  - Amplify, CloudFront, Cognito, EC2 OLTP in the **user-facing domain**
  - API Gateway, Lambda Ingest, S3 Raw bucket in the **ingestion & data lake domain**
  - ETL Lambda, PostgreSQL DW on EC2, R Shiny in the **analytics & DW domain**
- Be able to explain why OLTP and Analytics are separated:
  - **Logical**: different schemas, different types of workloads
  - **Physical**: public subnet vs private subnet, two different EC2 instances

#### Hands-on Skills

- Send clickstream events from the frontend to API Gateway → Lambda Ingest → S3 Raw (`clickstream-s3-ingest`).
- Configure a **Gateway VPC Endpoint for S3** and adjust the private route table so private components can access S3.
- Configure and test an **ETL Lambda** (`SBW_Lamda_ETL`) that can:
  - Read raw JSON files from `s3://clickstream-s3-ingest/events/YYYY/MM/DD/`
  - Transform events into rows for the table `clickstream_dw.public.clickstream_events`
- Connect to the DW (`SBW_EC2_ShinyDWH`) and run sample SQL queries:
  - Count the number of events
  - Top products
  - Basic funnel
- Access **R Shiny dashboards** via SSM port forwarding and interpret:
  - Funnel charts
  - Product engagement charts
  - Time-series charts of activity over time

#### Security & Cost Awareness

- Understand the importance of protecting user behavior data by placing `EC2_ShinyDWH` and `Lambda_ETL` in **private subnets**:
  - Only allow `Lambda_ETL` to access S3 through a **Gateway VPC Endpoint**
  - Only allow admins to access `EC2_ShinyDWH` through SSM using **Interface VPC Endpoints**
- Recognize the main security controls:
  - Separation of public and private subnets
  - Security groups between `sg_oltp_webDB`, `sg_Lambda_ETL`, `sg_analytics_ShinyDWH`
  - Least-privilege IAM permissions for each Lambda
  - Zero-SSH administration using AWS Systems Manager Session Manager (no bastion host, no open SSH port).

---

### Workshop Scope

The workshop focuses on three main capability areas:

1. **Implementing the clickstream ingestion layer**

   - Capture browser interactions in the Next.js frontend
   - Send JSON events to API Gateway (`clickstream-http-api`)
   - Store raw events over time in `clickstream-s3-ingest`

2. **Building the private analytics layer**

   - Create the VPC, subnets, route tables, and VPC endpoints
   - Run `SBW_Lamda_ETL` inside the VPC
   - Connect the ETL Lambda to the private EC2 Data Warehouse (`SBW_EC2_ShinyDWH`)

3. **Visualizing analytics with Shiny dashboards**
   - Query `clickstream_dw` from R Shiny
   - Display funnels, product performance, and time trends
   - Access Shiny through **SSM Session Manager port forwarding**

---

### Out of Scope

To keep the workshop focused and manageable, we **do not** go deep into:

- Real-time streaming (Kinesis, Kafka, MSK, …)
- Advanced DW services (Amazon Redshift / Redshift Serverless)
- Recommendation, segmentation, or anomaly detection using ML
- Production-grade CI/CD, blue/green deployments, multi-account setups
- Advanced SQL tuning or detailed index design

These are natural follow-up directions once the batch-based clickstream foundation in this workshop is in place.
