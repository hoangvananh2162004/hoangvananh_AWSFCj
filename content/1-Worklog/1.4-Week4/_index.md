---
title: "Week 4 Worklog"
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

- Learn about virtual machines on AWS.
- What operating systems are available and which ones are the most commonly used on AWS.
- How EC2 is managed and why it is widely used.

### Tasks to be carried out this week:

| Day | Task                                                                  | Start Date | Completion Date | Reference Material                        |
| --- | --------------------------------------------------------------------- | ---------- | --------------- | ----------------------------------------- |
| 1   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance type AWS.           | 29/09/2025 | 29/09/2025      |
| 2   | - Amazon Elastic Compute Cloud ( EC2 ) - AMI / Backup / Key Pair EC2. | 30/09/2025 | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Amazon Elastic Compute Cloud ( EC2 ) - Elastic block store.         | 30/09/2025 | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance store               | 30/09/2025 | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Amazon Elastic Compute Cloud ( EC2 ) - User data                    | 30/09/2025 | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - Amazon Elastic Compute Cloud ( EC2 ) - Meta data                    | 30/09/2025 | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 7   | - Amazon Elastic Compute Cloud ( EC2 ) - EC2 auto scaling             | 1/10/2025  | 1/10/2025       | <https://cloudjourney.awsstudygroup.com/> |
| 8   | - EC2 Autoscaling - EFS/FSx - Lightsail - MGN                         | 1/10/2025  | 1/10/2025       | <https://cloudjourney.awsstudygroup.com/> |

### Week 4 Achievements:

- Concept of virtual machines and virtual machine services:

  - Amazon Elastic Compute Cloud (EC2)
  - Amazon Lightsail
  - Amazon EFS/FSx
  - AWS Application Migration Service (MGN)

- Amazon Elastic Compute Cloud (EC2):

  - EC2 is a virtual machine service, similar to physical servers and traditional virtual servers. However, EC2 can be launched faster and provides more flexible resource scalability.
  - EC2 can support workloads for hosting websites, applications, databases, and essentially anything that traditional servers can do, EC2 can also handle.
  - To be considered an Instance type, it must include the following factors.
    - CPU/GPU.
    - Memory.
    - Network.
    - Storage
  - AMI(Amazon Machine Image)/Backup/Keypair.
    - AMI (Amazon Machine Image) is used to provision one or multiple EC2 Instances at the same time.
    - AMIs are available from AWS, on the AWS Marketplace, or as custom AMIs created from EC2 Instances.
    - An AMI includes root OS volumes, usage permissions that define which AWS accounts can use the AMI, and EBS volume mappings that will be created and attached to EC2 Instances.
    - EC2 Instances can be backed up by creating snapshots.
    - Key pairs (public key and private key) are used to encrypt login information for EC2 Instances.
  - Elastic block store.

    - Amazon EBS provides block storage and is directly attached to an EC2 Instance. Although it is attached like a RAW device, EBS essentially operates independently from EC2 and is connected through its own private network.
    - EBS has two main types of disks: HDD and SSD, designed to achieve 99.999% availability by replicating data across 3 storage nodes within a single AZ.
    - Some specific EC2 Instances are optimized for EBS performance (Optimized EBS Instances).
    - By default, an EBS volume can only be attached to one EC2 Instance, but EC2 Instances running on the Nitro Hypervisor can use one EBS volume attached to multiple EC2 Instances (EBS Multi-Attach).
    - EBS can be backed up by creating snapshots stored in S3 (Simple Storage Service).
      - The first snapshot is a full backup, and all subsequent snapshots are incremental.

  - Instance Store is a high-speed NVMe storage located on the physical node that hosts the EC2 virtual machines..

    - Instance Store will lose all data when the EC2 instance is stopped.
    - Instance Store does not lose data when the instance is restarted, but data can be lost if the instance crashes.
    - Instance Store does not replicate data, so it is not recommended for storing important data.
    - It is typically used in environments that require very high IOPS performance.
    - When using Instance Store, data is often replicated to an EBS volume to ensure safety.

  - EC2 user data is a script that runs once when an EC2 instance is provisioned from an AMI.

    - Depending on the server's operating system, we will use Bash shell scripts (Linux) or PowerShell (Windows).

  - EC2 Metadata refers to information related to the EC2 instance itself, such as Private IP address, Public IP address, Hostname, Security Groups, and so on.

  - EC2 Auto Scaling is a feature that supports increasing or decreasing the number of EC2 instances based on specific conditions (scaling policies).
    - EC2 Auto Scaling can register EC2 instances with an Elastic Load Balancer.
    - EC2 Auto Scaling operates across multiple AWS Availability Zones.
    - EC2 Auto Scaling can support different pricing options.

- EC2 Autoscaling - EFS/FSx - Lightsail - MGN:
  - EC2 offers four pricing options:
    - On-Demand: Pay per hour/minute/second, only for what you use. Suitable for workloads that run occasionally, e.g., 6 hours a day.
    - Reserved Instance: Commit to using for a 1–3 year term to get a discount, but limited to specific EC2 instance types/families.
    - Savings Plans: Commit to usage for a 1–3 year term to get a discount, but may not be limited to specific EC2 instance type families.
    - Spot Instance: Utilize spare capacity at a low price; however, AWS can terminate the instance with a 2-minute notice.
  - Amazon Lightsail.
    - Amazon Lightsail is a low-cost computing service (billed monthly, starting from $3.5/month). Additionally, each Lightsail instance includes a data transfer allowance (which is much cheaper than data transfer from EC2).
    - Amazon Lightsail is suitable for light workloads, test/dev environments, and workloads that do not require high CPU usage continuously (less than 2 hours per day).
    - Amazon Lightsail also supports backups via snapshots, similar to EC2.
    - Amazon Lightsail runs in a separate VPC environment and can connect to other VPCs through VPC Peering.
  - Amazon EFS/FSX.
    - EFS
      - EFS (Elastic File System) allows you to create NFSv4 network volumes and attach them to multiple EC2 instances simultaneously, with storage capacity scaling up to petabytes. EFS is Linux-only.
      - Using EFS, you are billed only for the storage you use (whereas EBS charges based on allocated capacity).
      - EFS can be configured to mount to on-premises environments via Direct Connect (DX) or VPN.x
    - FSX
      - FSx allows you to create NTFS volumes and attach them to multiple EC2 instances simultaneously using the SMB (Server Message Block) protocol. FSx supports both Windows and Linux.
      - Using FSx, you are billed only for the storage you use (whereas EBS charges based on allocated capacity).
      - FSx supports deduplication, which can reduce costs by 30–50% for typical use cases.
- AWS Application Migration Serveice(MGN):
  - AWS Application Migration Service (MGN) is used to migrate and replicate physical servers into the AWS environment for the purpose of building a Disaster Recovery (DR) site.
  - Application Migration Service(MGN) continuously replicates source servers to EC2 instances in the AWS account (asynchronously/synchronously).
  - During replication, MGN uses staging machines that are much smaller in number and configuration compared to the source servers.
  - During cutover, MGN stops the source servers and launches the EC2 instances on AWS.
