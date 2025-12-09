---
title: "Architecture Walkthrough"
weight: 52
chapter: false
pre: " <b> 5.2. </b> "
---

![Architecture](/images/architecture.png)
<p align="center"><em>Figure: Architecture Batch-base Clickstream Analytics Platform.</em></p>

---

## 5.2.1 User & Frontend Domain

### (1) User Browser – End User

**Responsibilities**

- Access the website: browse the catalogue, view product detail pages, cart, checkout, etc.
- Log in / register an account.
- Generate **clickstream events** such as:
  - `page_view`, `product_view`, `add_to_cart`, `purchase`, …

**Role in the pipeline**

- Is the **source of all behaviors** that the analytics system analyzes.
- JavaScript on the frontend will collect these events and send them to the backend through the API (6).

---

### (2) Amazon CloudFront (CDN / Edge)

**Responsibilities**

- Acts as the **CDN layer** that delivers website content:
  - Caches HTML/CSS/JS/images to reduce latency.
  - Offloads traffic from the origin (Amplify, S3 assets).
- Can serve as a **single entry point** for web traffic:
  - Route/forward requests:
    - Dynamic web content → Amplify.
    - Static images/media → S3 (3).

**Why it is important**

- An e-commerce website needs **good page load speed**.  
- CloudFront helps improve user experience across different geographic regions.

---

### (3) Amazon S3 – Media Assets

**Responsibilities**

- Store **product images** and other **static assets** for the website.
- Act as the **origin** when CloudFront (2) serves images/assets.

**Notes**

- This bucket (for example, `clickstream-s3-sbw`) is **separate** from the RAW clickstream bucket (8) in order to:
  - Clearly separate permissions.
  - Make management and cleanup easier.

---

### (4) Amazon Amplify – Front-end Hosting

**Responsibilities**

- Build & deploy the **Next.js** application (for example, `ClickSteam.NextJS`).
- Host the frontend (SSR/ISR, API routes).
- Manage the deployment pipeline: build, artifact, domain mapping.

**Integration with other services**

- Connects to **Cognito (5)** for handling authentication.
- Sends **clickstream events** from the frontend to **API Gateway (6)**.
- Connects to **EC2 OLTP (20)** via Prisma to read/write transactional data.

---

### (5) Amazon Cognito – Authentication / Authorization

**Responsibilities**

- Manage **user identity**:
  - Sign up / sign in / reset password.
  - Store the user pool and issue JWT tokens (ID token, access token).
- Issue tokens for the frontend so that it can call backend APIs with the correct permissions.

**Role in clickstream**

- Allows determining:
  - `user_login_state` (logged-in / guest).
  - `identity_source` (Cognito, social login, etc.).
- This information is attached to events and stored in S3/DWH to analyze user behavior.

---

## 5.2.2 Ingestion & Data Lake Domain

### (6) Amazon API Gateway – Clickstream HTTP API

**Responsibilities**

- Provide a public HTTP endpoint for the frontend:
  - `POST /clickstream`.
- Perform basic validation:
  - Method/path, throttling.
  - Optional: integrate a Cognito authorizer.

**Processing flow**

- Receive JSON requests from the browser.
- Forward the payload to **Lambda Clickstream Ingest (7)**.

---

### (7) AWS Lambda – Clickstream Ingest

**Main responsibilities**

- Receive events from API Gateway (6).
- Act as the **ingestion layer**:

  - Validate schema / event type.
  - Perform minimal enrichment:
    - Generate `event_id`.
    - Normalize `event_timestamp`.
    - Attach `client_id`, `session_id`, `user_login_state`, `identity_source` if needed.

- Write raw events (`raw JSON`) to the **S3 Clickstream Raw bucket (8)** using time-based prefixes/partitions.

- Write logs to **CloudWatch (17)** for debugging.

---

### (8) Amazon S3 – Clickstream Raw Data

**Responsibilities**

- Store all **raw clickstream data** written by Lambda Ingest (7).
- Act as a **“mini data lake”** for batch ETL:

  - Data is organized by **time-based folders/prefixes**:
    - `events/YYYY/MM/DD/`.
  - ETL (11) reads the data in batches (hourly, daily, etc.).

**Architectural role**

- Serves as the **source of truth** for clickstream data:
  - You can reprocess or rebuild the Data Warehouse if needed.

---

### (9) Amazon EventBridge – ETL Trigger / Cron Job

**Responsibilities**

- Run batch schedules, for example: `rate(1 hour)`.
- On each schedule:
  - Trigger **Lambda ETL (11)**.

**Why separating it out**

- The ETL “clock” lives in EventBridge:
  - Easy to adjust frequency (1h, 30’, 5’, etc.).
  - Easy to disable/enable.
  - Separates “when to run” from the ETL logic itself.

---

## 5.2.3 VPC & Private Analytics Domain

### (19) VPC + Subnets + Internet Gateway

**Amazon VPC**

- Isolate the network and control routing / security for the entire platform.

**Public subnet – OLTP**

- Contains **EC2 OLTP (20)**.
- Has a route `0.0.0.0/0 → Internet Gateway`.
- Allows:
  - Amplify/Internet to access PostgreSQL OLTP based on security groups.

**Private subnet – Analytics**

- Contains:
  - **EC2 Shiny + DWH (12)**.
  - **Lambda ETL (11)**.
- Has no public IP.
- Can only go out through **VPC Endpoints (10, 15)**.

**Internet Gateway**

- Provides Internet access for resources in the public subnet.
- The private subnet does not route to the IGW (unless you configure it differently).

---

### (10) Gateway Endpoint – S3 Gateway Endpoint

**Responsibilities**

- Allow resources inside the VPC (Lambda ETL, private EC2) to access S3 (8) **without needing NAT/Internet**.
- Traffic to S3 goes over the **AWS private network**, which is more secure and cheaper than a NAT Gateway.

**Role**

- A key piece to:
  - Keep the decision of “**no NAT Gateway**”.
  - Still allow ETL / DW to read and write S3.

---

### (11) AWS Lambda – ETL

**Responsibilities**

- Triggered by **EventBridge (9)** on a schedule.
- Runs inside the **private subnet** of the VPC.
- Performs batch ETL:

  - Read raw events from **S3 (8)** via the Gateway Endpoint (10).
  - Transform / clean / flatten data according to the Data Warehouse schema (for example, the 15 fields you finalized).
  - Load into **PostgreSQL DWH on private EC2 (12)**:
    - Insert / upsert, possibly partitioned by day/hour.

- Write logs and metrics to **CloudWatch (17)**; on errors it can send notifications via **SNS (18)**.

---

### (12) Amazon EC2 – Private (Shiny + DWH)

**Responsibilities**

- Server for **Data Warehouse (PostgreSQL)** + **Shiny Server**.
- Receive cleaned data from Lambda ETL (11) and store it in DWH tables.
- Provide data to Shiny dashboards:
  - Shiny queries the local/private Postgres instance.

**Characteristics**

- Runs in a **private subnet**.
- Has no public IP.
- Administration access is via **SSM Session Manager (14 + 15)**.

---

### (13) R Shiny Server (on Private EC2)

**Responsibilities**

- Host analytics dashboards:

  - Overall KPIs.
  - Conversion funnel.
  - Top products.
  - User behavior, session analysis, etc.

- Connect directly to **PostgreSQL DWH** on the same EC2 instance.

**Access**

- Listens on port `3838`.
- Typically accessed via:

  - Internal VPN, or
  - **SSM port forwarding** (14).

---

### (14) AWS Systems Manager – Session Manager

**Responsibilities**

- Allow admins to access private EC2 instances **without SSH, without public IPs**.
- Support:

  - Opening a terminal into EC2 (run command).
  - **Port forwarding** (for example, forward `localhost:3838` → Shiny on EC2).
  - Better auditing of access sessions.

**Example**

- You can access Shiny from your local machine at:  
  `http://localhost:3838/sbw_dashboard`  
  after successfully configuring the session port forwarding.

---

### (15) Interface Endpoint – SSM VPC Endpoints

**Responsibilities**

- Create **private** connections from EC2/Lambda in the VPC to SSM/Session Manager services:

  - `ssm`, `ssmmessages`, `ec2messages`, …

- Ensure that:

  - Private EC2 instances can still use SSM.
  - Traffic does not have to go through the Internet/NAT.

---

## 5.2.4 Cross-cutting Services

### (16) Amazon IAM

**Responsibilities**

- Manage **roles/policies** for the whole system:

  - Permissions for API Gateway, Lambda.
  - Lambda Ingest writes to S3 (8).
  - Lambda ETL reads S3 (8) + connects to DB (12).
  - EC2 role for SSM + CloudWatch agent (if any).

**Principles**

- **Least privilege**.
- Clearly separate permissions by function and by service.

---

### (17) Amazon CloudWatch

**Responsibilities**

- Collect **logs & metrics** from:

  - Lambda Ingest (7).
  - Lambda ETL (11).
  - EventBridge rule (9).
  - EC2 if the agent is installed.

- Define **alarms**: ETL failures, ingest failures, number of retries, duration, etc.

**Role**

- The **main place for debugging** when the pipeline has problems.

---

### (18) Amazon SNS

**Responsibilities**

- Acts as the **notification channel** when there are incidents:

  - ETL failure.
  - Ingest failure.
  - CloudWatch alarm triggered.

- Send email / SMS / webhook depending on configuration.

---

### (20) Amazon EC2 – Public (OLTP)

**Responsibilities**

- Run **PostgreSQL OLTP** for the e-commerce web system:

  - Order transactions.
  - Product information, inventory.
  - User information.

**Relation to DWH**

- **Separated** from the Data Warehouse (12):

  - OLTP is optimized for **transactions**.
  - DWH is optimized for **analytics and batch loads**.
