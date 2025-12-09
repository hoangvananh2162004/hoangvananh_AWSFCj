---
title: "Event 3"
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

# Bài thu hoạch “​DevOps on AWS”

### Mục Đích Của Sự Kiện

- Welcome & DevOps Mindset
- AWS DevOps Services – CI/CD Pipeline
- Infrastructure as Code (IaC)
- Container Services on AWS
- Monitoring & Observability
- DevOps Best Practices & Case Studies

### Danh Sách Diễn Giả

- **Bao Huynh** – AWS Community Builders
- **Thinh Nguyen** – AWS Community Builders
- **Vi Tran** – AWS Community Builders

### 1. Infrastructure as Code (IaC)

- IaC dùng **code thay vì click** để quản lý hạ tầng.
- Lợi ích: tự động hóa, ổn định, reproducible, dễ cộng tác.
- ClickOps → chậm, dễ lỗi, khó nhất quán.

### 2. AWS CloudFormation

- Viết template bằng **YAML/JSON**.
- Deploy thành **Stack** gồm nhiều resource.
- Cấu trúc template:
  - Parameters, Mappings, Conditions, Resources, Outputs.
- Có **Drift Detection** để phát hiện thay đổi thủ công.

### 3. AWS CDK (Cloud Development Kit)

- IaC bằng **ngôn ngữ lập trình thực** (TS, Python, Java…).
- Sinh ra CloudFormation template.
- 3 cấp Construct:
  - L1: Cfn\* (mapping 1:1 CFN)
  - L2: API thân thiện, có defaults
  - L3: Pattern kiến trúc hoàn chỉnh
- Lệnh CDK phổ biến: `init`, `bootstrap`, `synth`, `deploy`, `diff`, `destroy`.

### 4. Terraform vs Công cụ IaC khác

- Terraform: multi-cloud, dùng HCL.
- Pulumi: IaC bằng ngôn ngữ lập trình.
- OpenTofu: fork mã nguồn mở của Terraform.

### 5. Docker Fundamentals

- Container nhẹ, nhanh; VM nặng.
- Dockerfile → build image.
- Workflow: Build → Push → Pull → Run.

### 6. AWS Container Services

### ECR

- Private container registry của AWS.

### ECS

- Orchestration cho Docker.
- Chạy trên **EC2** hoặc **Fargate**.
- Thành phần: Cluster, Task Definition, Service, Task.

### EKS

- Managed Kubernetes.
- Linh hoạt hơn ECS nhưng phức tạp hơn.

### App Runner

- Deploy app/web/API nhanh, không cần quản lý server.
- Hỗ trợ deploy từ GitHub hoặc ECR.

#### Một số hình ảnh sự kiện

![anh](/images/su1.jpg)
![anh](/images/su2.jpg)
![anh](/images/su3.jpg)
![anh](/images/su4.jpg)
![anh](/images/su5.jpg)
![anh](/images/su6.jpg)
![anh](/images/su7.jpg)
![anh](/images/su8.jpg)
![anh](/images/fba8dbb5f6de798020cf2.jpg)
