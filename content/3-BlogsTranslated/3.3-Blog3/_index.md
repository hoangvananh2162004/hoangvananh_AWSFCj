---
title: "Blog 3"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Build centralized cross-Region backup architecture with AWS Control Tower

by Chris Falk, Lei Shi, và Pujah Goviel | on 11 SEP 2025 | in Advanced (300), AWS Backup, AWS Control Tower, AWS Key Management Service (KMS), AWS Organizations, Best Practices, Enterprise governance and control, Resilience, Storage | Permalink | Comments

Managing data protection at scale is a critical challenge for the modern enterprise. As organizations grow, their data becomes increasingly distributed, making it difficult to implement consistent backup policies that ensure comprehensive coverage. IT teams must balance competing needs of compliance requirements, resource protection, and operational efficiency – all while struggling to validate and orchestrate backup procedures across an expanding digital footprint.

AWS Backup offers a powerful solution to these challenges with a centralized, fully-managed service that streamlines data protection at scale. Enterprises can leverage AWS Organizations alongside AWS Backup to implement automated, consistent backup policies across their entire cloud environment. The integration with AWS Control Tower further simplifies this process by enabling organizations to incorporate enterprise-wide backup management directly into their well-architected multi-account landing zone.

In this post, we’ll demonstrate how to implement AWS Backup using AWS Control Tower integration. We’ll explore the architecture, prerequisites, and step-by-step implementation process. By following this guide, you’ll learn how to automatically deploy and manage backup policies across your organization, helping to meet compliance requirements, protect critical resources, and reduce administrative overhead. This solution is particularly valuable to enterprises looking to standardize backup operations while scaling their cloud environment.

## Solution overview

AWS Control Tower now offers built-in capabilities to streamline your backup management at scale with direct AWS Backup integration. This capability automatically provisions a central backup vault in each AWS Region within a dedicated central backup account. As your organization scales to more accounts and regions, AWS Control Tower automatically creates local backup vaults in each workload account in AWS Backup-enabled organizational units (OUs) across all governed AWS Regions. You can customize your backup strategy by configuring backup policies that copy your backups from local vaults to central vaults, either within the same Region or cross-Region. Moreover, you don’t need custom deployment pipelines or complex automation. You can use AWS Backup policies to implement a comprehensive backup strategy that meets your organization’s specific requirements while maintaining consistent governance across your AWS environment.

![anh](/images)

## Prerequisites

Before implementing the AWS Control Tower and AWS Backup integration, make sure that you have the following foundational elements in place:

- An existing AWS Organization governed by AWS Control Tower
- Administrative access to the AWS Organizations management account

## Walkthrough

This section walks you through the solution steps.

### Step 1: Provision dedicated backup accounts

First, you must create two specialized accounts that form the backbone of your backup infrastructure:

- **Backup administrator account**: This account manages backup policies and configurations for your entire organization.
- **Central backup account**: This account stores centralized backup vaults and cross-account backup copies.

Create these two accounts through the Organizations console or AWS Command Line Interface (AWS CLI). Do not use AWS Control Tower Account Factory or any other AWS account vending process that enrolls new accounts in AWS Control Tower governance. In step three, when you enable AWS Backup in AWS Control Tower, the accounts are enrolled and placed into the Security OU automatically.

### Step 2: Create a multi-Region AWS KMS Key

Backup encryption is a crucial security requirement in enterprise organizations. Create a multi-Region AWS Key Management Service (AWS KMS) key in your management account to enable encryption of backup data at rest and secure cross-account backup capabilities. The key should be created in your AWS Control Tower Home Region and replicated to all governed AWS Regions. This multi-Region AWS KMS key is used to encrypt backups in the central backup account vaults, and all local vaults in the workload accounts in all Regions. Although you can create the key manually through the AWS Management Console, we recommend infrastructure as code (IaC) (AWS CloudFormation, AWS Cloud Development Kit (AWS CDK), or Terraform).

Refer to the AWS Control Tower documentation for more details on how to create the relevant AWS KMS key policy.

### Step 3: Enable AWS Backup in AWS Control Tower

You can now to integrate AWS Backup with your AWS Control Tower landing zone. Navigate to **AWS Control Tower console**, choose **Landing zone settings**, and choose **Update landing zone**. In the backup configuration section, type or choose the following details:

- **Central backup account ID**: The 12-digit account ID of the Central Backup account that you created in Step 1.
- **Backup administrator account ID**: The 12-digit account ID of the backup administrator account that you created in Step 1.
- **KMS Key ARN**: The Amazon Resource Name (ARN) of the AWS KMS key that you created in Step 2.

You can choose the accounts created in Step 1 and the AWS KMS key in Step 2 from the drop down list in each text box, as shown in the following figure.

![anh](/images)

The landing zone update process may take 30-45 minutes to complete. When it is finished, you should see the AWS Backup status as **Enabled**, and your specified accounts and AWS KMS keys should appear in the AWS Control Tower console, as shown in the following figure.

![anh](/images)

### Step 4: Configure service opt-in and enable delegated administrator

After enabling AWS Backup in AWS Control Tower, configure service opt-in in the AWS Backup console.

Navigate to AWS Backup console, choose Settings, and in the Service opt-in list:

- Enable opt-in for services needed for your workloads that you would like to cover with AWS Backup, such as Amazon Relational Database Service (Amazon RDS), Amazon Elastic Compute Cloud (Amazon EC2), Amazon DynamoDB, Amazon Elastic Block Store (Amazon EBS), Amazon Elastic File System (Amazon EFS).
- This must be done manually in the AWS Backup console.
- Backup service opt-in is an AWS Region-based setting, and you must repeat the process for all AWS Control Tower governed AWS Regions.

![anh](/images)

Finally, set the backup administrator account as a delegated administrator to enable organization-wide backup task monitoring and backup policy management from this account, as shown in the following figure.

![anh](/images)

### Step 5: Enable AWS Backup baseline on specific OUs

After configuring the preceding core backup infrastructure, you must enable the AWS Backup baseline on your desired OUs. Navigate to the AWS Control Tower console, choose Organization from the left navigation pane, choose the OU where you want to enable backup, then find and enable the option AWS Backup on this OU, as shown in the following figure

![anh](/images)

The order of enablement is critical, so when enabling the AWS Backup baseline across your organization, follow a hierarchical approach. Start with the top-level OUs before proceeding to child OUs. In the example shown in the following figure, you would first enable the AWS Backup baseline on the Workloads top-level OU, then proceed to child OUs such as Workloads X, Workloads Y, etc.

![anh](/images)

### Step 6: Tag resources for backup

The final step in your implementation is to apply tags to the resources to be backed up.

When you enable AWS Backup through AWS Control Tower, a set of default backup plans with pre-defined resource tags are automatically created for hourly, daily, weekly, and monthly backups. These plans provide reasonable protection for common scenarios. However, most enterprises will want to implement custom backup plans based on workload criticality and compliance requirements by using AWS Organizations backup policies.

## Cleaning up

The following steps walk you through cleaning up after completion of this solution.

### Step 1: Disable backup baseline on specific OUs

Before removing the integration at the landing zone level, you should first disable the backup controls at each OU.

1. Navigate to the **AWS Control Tower console**
2. Choose **Organization** from the left navigation pane
3. Following the reverse order of your enablement process, starting with child OUs before proceeding to parent OUs::
   a. Choose the OU
   b. Choose the **Controls** tab
   c. Find and **disable** the **AWS Backup on this OU** option

### Step 2: Remove the AWS Backup integration

Now you are ready to remove the AWS Backup integration from your AWS Control Tower landing zone.

1. Navigate to the **AWS Control Tower console**
2. Choose **Landing zone settings**
3. Choose **Update landing zone**
4. In the backup configuration section, choose **AWS Backup is not enabled**
5. Choose **Next**, review the changes, then choose **Update landing zone** to apply these changes

AWS Control Tower removes the integration while preserving your existing backup data, as shown in the following figure.

![anh](/images)

### Step 3: Clean up additional resources

After the integration is removed, you may want to clean up more resources:

- Schedule deletion for the AWS KMS key that was created
- Remove any backup-specific AWS Identity and Access Management (IAM) roles that are no longer needed
- Decide whether to repurpose or close the dedicated backup accounts
- Delete any recovery points and vaults that are no longer needed

By default, removing the AWS Backup integration does not delete your existing backup vaults or recovery points. This is by design to prevent accidental data loss during configuration changes. However, this means that you must manage these resources explicitly if you want to remove them. To manage your backup vaults and recovery points after removing the integration, navigate to the **AWS Backup console** in your central backup account, and choose **Backup vaults** from the left navigation pane. To delete the entire vault and all recovery points, first delete all recovery points within the vault, then choose the vault and choose .

## Conclusion

By integrating AWS Backup with AWS Control Tower, you can automate and standardize data protection across your entire enterprise without custom development. Organizations using AWS Control Tower can enable AWS Backup as a baseline landing zone service to automate backup vault creation and policy deployment across multiple accounts and AWS Regions, ensuring consistent data protection while reducing administrative overhead. The centralized management approach streamlines operations, while flexible policy options allow you to tailor protection strategies to your specific business needs. Most importantly, this solution enhances overall data resilience through built-in best practices. As your cloud environment scales, you can confidently maintain robust, consistent backup practices that align with well-architected principles.

#### TAGS: AWS Backup, AWS Cloud Storage, AWS Control Tower, AWS Key Management Service (AWS KMS), data resiliency
