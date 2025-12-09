---
title: "Summary & Clean up"
weight: 56
chapter: false
pre: " <b> 5.6. </b> "
---

## 5.6.1 Summary

After completing the lab, we have built a fully working **Clickstream Analytics Platform**:

1. **User-Facing Layer**
   - Next.js application (`ClickSteam.NextJS`) on Amplify + CloudFront  
   - User authentication with Cognito  
   - PostgreSQL OLTP database (`clickstream_web`) on `SBW_EC2_WebDB` (public subnet)  

2. **Ingestion & Raw Data Layer**
   - API Gateway HTTP API: `clickstream-http-api` (route `POST /clickstream`)  
   - Lambda Ingest: `clickstream-lambda-ingest`  
   - S3 Raw bucket: `clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json`  

3. **Private Analytics Layer**
   - VPC with public & private subnets (`SBW_Project_VPC`)  
   - S3 Gateway Endpoint, SSM Interface Endpoints  
   - Data Warehouse on EC2: `SBW_EC2_ShinyDWH`, database `clickstream_dw`  
   - ETL Lambda inside the VPC: `SBW_Lamda_ETL`, triggered by `SBW_ETL_HOURLY_RULE`  
   - R Shiny dashboards (`sbw_dashboard`) accessible only via SSM port forwarding  

Overall, this architecture demonstrates how to design a **batch-based analytics platform** that is secure, cost-optimized, and primarily built on serverless components plus two EC2 instances.

---

## 5.6.2 Key Takeaways

- **Separation of concerns**:
  - OLTP and Analytics are separated on 2 different EC2 instances, belonging to different logical domains.  

- **Security**:
  - The DW and Shiny run in a private subnet with no public IP.  
  - SSM Session Manager replaces traditional SSH.  
  - The S3 Gateway Endpoint keeps S3 traffic inside the AWS private network.  

- **Cost optimization**:
  - No NAT Gateway is used.  
  - ETL relies on serverless (Lambda + EventBridge).  
  - S3 is used as low-cost storage for raw data.  

- **Easy to extend**:
  - The current design is batch-based, but can be extended to real-time, more complex analytics, or migrated to other DW technologies.

---

## 5.6.3 Resource Clean-up

1. **Amplify & CloudFront**
   - Delete the Amplify app (`ClickSteam.NextJS`).  
   - This action also deletes the associated CloudFront distribution.

2. **API Gateway & Lambda**
   - Delete `clickstream-http-api`.  
   - Delete the Lambda functions:
     - `clickstream-lambda-ingest`  
     - `SBW_Lamda_ETL`  

3. **EventBridge**
   - Delete the rule `SBW_ETL_HOURLY_RULE`.  

4. **S3 Buckets**
   - Empty and then delete:
     - `clickstream-s3-ingest` (RAW clickstream)  
     - `clickstream-s3-sbw` (assets), if it is not used for other projects  

5. **EC2 Instances**
   - Stop or terminate:
     - `SBW_EC2_WebDB`  
     - `SBW_EC2_ShinyDWH`  
   - Release any associated Elastic IP (if attached).

6. **VPC & Networking**
   - Delete VPC endpoints (S3 Gateway, SSM Interface Endpoints).  
   - Delete route tables, subnets, and the Internet Gateway.  
   - Finally, delete `SBW_Project_VPC` if it is no longer needed.
