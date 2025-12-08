---
title: "Week 2 Worklog"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

- Study and understand what VPC and VPN are
- How to create, connect, and operate networking services
- Experimentation and results

### Tasks to be carried out this week:

| Day | Task                                          | Start Date | Completion Date | Reference Material                                                                            |
| --- | --------------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Learn the concepts of VPC and VPN           | 08/11/2025 | 08/11/2025      |
| 2   | - Practice: Use AWS services with VPC and VPN | 08/12/2025 | 08/12/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Create **Subnets**, **Internet Gateway**    | 13/08/2025 | 13/08/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Create Route **Table**, **security groups** | 13/08/2025 | 13/08/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Create **Route Table**, **security groups** | 13/08/2025 | 13/08/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 2 Achievements:

- Understanding VPC and VPN:

  - Amazon VPC (Virtual Private Cloud): Allows you to create a private virtual network to launch AWS resources such as virtual servers and virtual networks, with dedicated management.
  - A VPC can span multiple AZs (Availability Zones) but exists in only one Region.
  - When creating and configuring, you must specify an **IPv4** CIDR block (mandatory), while **IPv6** is optional.
  - Maximum: 5 VPCs per Region per Account.
  - Purpose: Used to separate environments (Production, Dev, Test, Staging).
  - You can divide instances into subnets, each subnet must belong to a specific AZ and be assigned a CIDR block that is a subset of the VPC’s CIDR.
  - Each subnet reserves 5 IP addresses:
    - **Network address** (e.g., 10.10.1.0)
    - **Broadcast addres** (e.g., 10.10.1.255)
    - **Router address** (10.10.1.1)
    - **DNS address** (10.10.1.2)
    - **Future use** (10.10.1.3)

- Route Table:

  - A set of routes that determine the path for network traffic.
  - Default Route Table: Cannot be deleted, contains a single route that allows subnets in the VPC to communicate with each other. Each subnet must be associated with a route table.
  - Custom Route Table: Can be created, but the default route cannot be removed.

- ENI (Elastic Network Interface): A virtual network card attached to EC2 instances.

  - Ensures network settings remain consistent even if the server is replaced.
  - Private IP.
  - Elastic IP.
  - MAC address.

- VPC Endpoint: Allows private connection to AWS services within the VPC without using the internet.
  - Two types: Interface Endpoint and Gateway Endpoint.
- VPC Security Group: A stateful virtual firewall that controls inbound and outbound traffic to AWS resources.

  - Security Group rules are limited to source IP, port, and other Security Groups.
  - Only allow rules are supported, applied to ENIs.

- NACL (Network Access Control List): A stateless virtual firewall that controls inbound and outbound traffic at the subnet level.

  - Rules limited by source IP and port.
  - Applied to VPC Subnets.
  - Default NACL allows all inbound and outbound traffic.

- VPC Flow Logs: Capture information about IP traffic going into and out of the VPC.

  - Logs are stored in Amazon CloudWatch Logs or S3.
  - Does not capture packet contents.

- VPC Peering: Enables direct connectivity between two VPCs, requires 1:1 peering connections, does not support overlapping IP address ranges.

- Transit Gateway: A hub for interconnecting multiple VPCs.

- VPN:

  - Site-to-Site VPN: For hybrid models, providing continuous connectivity between on-premises and VPC.
  - Client-to-Site VPN: Allows individual hosts to securely connect to VPC resources.

  - AWS Direct Connect: Provides dedicated private connection between on-premises and AWS with latency of 20–30ms.

  - Elastic Load Balancing (ELB): A managed load balancing service.

  - Supports **HTTP**, **HTTPS**, **TCP** (secure), **SSL**.
  - Can be public or private.

  - Four types: **Application Load Balancer (ALB)**, **Network Load Balancer (NLB)**, **Classic Load Balancer (CLB)**, **Gateway Load Balancer (GLB)**

- Practice:
  - Search for VPC → create VPC → give it a name
  - Choose IPv4 and enter the address **10.10.0.0/16**
  - Create Subnets
  - Create Internet gateways
  - Create Security groups

Create Security groups
![anh](/images/a1.png)
![anh](/images/a2.png)
![anh](/images/a3.png)
![anh](/images/a4.png)
![anh](/images/a5.png)
![anh](/images/a6.png)
![anh](/images/a7.png)
