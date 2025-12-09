---
title: "Event 3"
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “​DevOps on AWS”

### Event Objectives

- Welcome & DevOps Mindset
- AWS DevOps Services – CI/CD Pipeline
- Infrastructure as Code (IaC)
- Container Services on AWS
- Monitoring & Observability
- DevOps Best Practices & Case Studies

### Speakers

- **Bao Huynh** – AWS Community Builders
- **Thinh Nguyen** – AWS Community Builders
- **Vi Tran** – AWS Community Builders

### 1. Infrastructure as Code (IaC)

- IaC uses **code instead of manual clicking** to manage infrastructure.
- Benefits: automation, consistency, reproducibility, easier collaboration.
- ClickOps → slow, error-prone, hard to maintain consistency.

### 2. AWS CloudFormation

- Write infrastructure templates using **YAML/JSON**.
- Deploy as a **Stack** containing multiple AWS resources.
- Template structure includes:
  - Parameters, Mappings, Conditions, Resources, Outputs.
- Supports **Drift Detection** to identify manual changes.

### 3. AWS CDK (Cloud Development Kit)

- IaC using **real programming languages** (TypeScript, Python, Java, etc.).
- Automatically synthesizes CloudFormation templates.
- Three Construct levels:
  - L1: Cfn\* constructs (1:1 mapping with CloudFormation)
  - L2: Higher-level API with sensible defaults
  - L3: Fully opinionated architecture patterns
- Common CDK commands: `init`, `bootstrap`, `synth`, `deploy`, `diff`, `destroy`.

### 4. Terraform vs Other IaC Tools

- **Terraform**: Multi-cloud, uses HCL, strong ecosystem.
- **Pulumi**: IaC using real programming languages.
- **OpenTofu**: Open-source fork of Terraform.

### 5. Docker Fundamentals

- Containers are lightweight and fast; VMs are heavier.
- Dockerfile defines how to build an image.
- Workflow: Build → Push → Pull → Run.

### 6. AWS Container Services

### ECR

- AWS private container registry.

### ECS

- Docker container orchestration service.
- Runs on **EC2** or **Fargate**.
- Components: Cluster, Task Definition, Service, Task.

### EKS

- Managed Kubernetes service.
- More flexible but more complex than ECS.

### App Runner

- Fast deployment for web apps and APIs without server management.
- Supports deploying from GitHub or ECR.

#### Some event photos

![anh](/images/su1.jpg)
![anh](/images/su2.jpg)
![anh](/images/su3.jpg)
![anh](/images/su4.jpg)
![anh](/images/su5.jpg)
![anh](/images/su6.jpg)
![anh](/images/su7.jpg)
![anh](/images/su8.jpg)
![anh](/images/fba8dbb5f6de798020cf2.jpg)
