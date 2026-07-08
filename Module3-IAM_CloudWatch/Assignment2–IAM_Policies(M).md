# Module 3 - Assignment 2 - IAM Policies

## Problem Statement

XYZ Corporation wants to improve AWS account security by implementing Identity and Access Management (IAM) policies for different teams. The goal is to provide only the permissions required by each team while restricting unnecessary access.

## Objective

Create and attach IAM policies according to the following requirements:

1. Create **Policy 1** that allows users to:
   - Full access to Amazon S3
   - Only create Amazon EC2 instances
   - Full access to Amazon RDS

2. Create **Policy 2** that allows users to:
   - Full access to Amazon CloudWatch
   - Full access to Billing
   - Only list Amazon EC2 resources
   - Only list Amazon S3 resources

3. Attach **Policy 1** to the **Dev Team** created in Assignment 1.

4. Attach **Policy 2** to the **Ops Team** created in Assignment 1.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|------|
| IAM | Yes | No | IAM is a global service and has no additional charges. |
| IAM Policies | Yes | No | Creating customer-managed IAM policies is free. |
| IAM Groups | Yes | No | Creating IAM groups is free. |
| IAM Policy Attachments | Yes | No | Attaching policies to users or groups is free. |
| Amazon S3 (Permissions only) | Yes | No | This assignment only creates permissions, not S3 buckets. |
| Amazon EC2 (Permissions only) | Yes | No | Only IAM permissions are configured. No EC2 instances are launched. |
| Amazon RDS (Permissions only) | Yes | No | Only IAM permissions are created. No database is created. |
| Amazon CloudWatch (Permissions only) | Yes | No | Only IAM permissions are configured. |
| AWS Billing Console Access | Yes | No | Granting billing permissions does not incur charges. |

**Free Tier Status:** ✅ This assignment is completely Free Tier eligible.

**AWS Promotional Credits Required:** **No**

Since this assignment only creates IAM policies and attaches them to IAM groups, no billable AWS resources are created.

---

# Architecture Diagram

```text
                     AWS Account
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
   IAM Group                           IAM Group
   Dev-Team                            Ops-Team
        │                                   │
        │                                   │
        ▼                                   ▼
   Policy-1                           Policy-2
        │                                   │
        │                                   │
        ▼                                   ▼

Full S3 Access                    Full CloudWatch Access

Create EC2 Only                   Full Billing Access

Full RDS Access                   List EC2 Resources

                                  List S3 Resources
```

---

# Permission Overview

| Team | Policy | Permissions |
|------|---------|-------------|
| Dev Team | Policy 1 | Full Amazon S3, Create EC2 Instances Only, Full Amazon RDS |
| Ops Team | Policy 2 | Full CloudWatch, Full Billing, List EC2 Resources, List S3 Resources |

---

# Dependency Flow

```text
Assignment 1
│
├── Dev Team IAM Group
│
└── Ops Team IAM Group
        │
        ▼
Create Policy 1
        │
        ▼
Create Policy 2
        │
        ▼
Attach Policy 1 to Dev Team
        │
        ▼
Attach Policy 2 to Ops Team
```

---

# Step 1: Verify IAM Groups Exist

## Navigation

```text
AWS Console
→ IAM
→ User Groups
```

## Configuration

Verify that the following IAM Groups already exist:

| Group Name |
|------------|
| `Dev-Team` |
| `Ops-Team` |

These groups should have been created in **Module 3 Assignment 1**.

## Why is this step required?

IAM policies cannot be attached to a group that does not exist.

The assignment requires attaching different policies to two different teams. Therefore, these IAM groups must already be available before creating and attaching the policies.

## Dependency

Depends on:

- Module 3 Assignment 1
- IAM Groups successfully created

## What happens if this step is skipped?

If the IAM groups do not exist:

- Policy attachment will fail.
- AWS will not display the required group during policy attachment.
- The assignment cannot be completed until the groups are created.

---

# Step 2: Create Policy Number 1

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Create Policy
```

## Configuration

Choose:

- **Policy Editor:** `JSON`

Replace the default JSON with the following policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3FullAccess",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    },
    {
      "Sid": "EC2RunInstancesOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": "*"
    },
    {
      "Sid": "RDSFullAccess",
      "Effect": "Allow",
      "Action": "rds:*",
      "Resource": "*"
    }
  ]
}
```

Click **Next**.

Provide the following details.

- **Policy Name:** `Dev-Team-Policy`
- **Description:** `Provides Full S3 access, EC2 instance creation permission and Full RDS access`

Click **Create Policy**.

## Why is this step required?

IAM policies define what actions users are allowed or denied inside an AWS account.

Instead of assigning permissions directly to each user, AWS best practice is to create reusable policies and attach them to groups or roles.

This policy grants the Dev Team only the permissions specified in the assignment.

## Dependency

Depends on:

- IAM service
- Administrator permissions
- IAM console access

## What happens if this step is skipped?

Without Policy 1:

- Dev Team users will not receive the required permissions.
- They will be unable to access S3.
- They will be unable to launch EC2 instances.
- They will not have RDS access.

---

---

# Step 3: Create Policy Number 2

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Create Policy
```

## Configuration

Choose:

- **Policy Editor:** `JSON`

Replace the default JSON with the following policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchFullAccess",
      "Effect": "Allow",
      "Action": "cloudwatch:*",
      "Resource": "*"
    },
    {
      "Sid": "BillingFullAccess",
      "Effect": "Allow",
      "Action": [
        "aws-portal:*",
        "billing:*",
        "budgets:*",
        "ce:*",
        "cur:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EC2ListOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3ListOnly",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    }
  ]
}
```

Click **Next**.

Provide the following details.

- **Policy Name:** `Ops-Team-Policy`
- **Description:** `Provides full CloudWatch and Billing access with read-only access to EC2 and Amazon S3`

Click **Create Policy**.

## Why is this step required?

The Operations Team is responsible for monitoring AWS resources and reviewing account usage rather than creating or modifying infrastructure.

This policy grants:

- Full access to Amazon CloudWatch for monitoring.
- Full access to the Billing Console for cost management.
- Read-only access to EC2 resources.
- Read-only access to Amazon S3 bucket information.

This follows the Principle of Least Privilege by providing only the permissions required for operational tasks.

## Dependency

Depends on:

- IAM service
- Administrator permissions
- Access to the IAM Policy editor

## What happens if this step is skipped?

Without Policy 2:

- The Ops Team cannot receive the required permissions.
- Users cannot monitor CloudWatch resources.
- Billing information cannot be accessed.
- EC2 and S3 resources cannot be viewed.

---

# Step 4: Attach Policy Number 1 to Dev Team

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev-Team
→ Permissions
→ Add Permissions
→ Attach Policies
```

## Configuration

Search for:

`Dev-Team-Policy`

Select the policy.

Click **Add Permissions**.

## Why is this step required?

Creating a policy alone does not grant permissions to anyone.

The policy must be attached to an IAM Group so that every member of the group automatically inherits those permissions.

By attaching the policy to the Dev Team group, every developer receives identical permissions without configuring each user individually.

## Dependency

Depends on:

- Step 2 (Policy 1 created)
- Step 1 (Dev-Team group exists)

## What happens if this step is skipped?

If the policy is not attached:

- Dev Team users will continue to have only their existing permissions.
- They will not receive access to Amazon S3.
- They will not be able to launch EC2 instances.
- They will not receive Amazon RDS permissions.

---

# Step 5: Attach Policy Number 2 to Ops Team

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops-Team
→ Permissions
→ Add Permissions
→ Attach Policies
```

## Configuration

Search for:

`Ops-Team-Policy`

Select the policy.

Click **Add Permissions**.

## Why is this step required?

Attaching the policy makes the permissions effective for every member of the Operations Team.

Using IAM Groups simplifies permission management because administrators only manage permissions once at the group level rather than individually for each user.

## Dependency

Depends on:

- Step 3 (Policy 2 created)
- Step 1 (Ops-Team group exists)

## What happens if this step is skipped?

If this policy is not attached:

- Operations Team users will not receive CloudWatch access.
- Billing permissions will not be granted.
- EC2 resources cannot be viewed.
- Amazon S3 bucket information cannot be listed.

---

# Permission Summary

| IAM Group | Attached Policy | Effective Permissions |
|-----------|-----------------|-----------------------|
| `Dev-Team` | `Dev-Team-Policy` | Full Amazon S3, Create EC2 Instances, Full Amazon RDS |
| `Ops-Team` | `Ops-Team-Policy` | Full CloudWatch, Full Billing, Read-only EC2, Read-only Amazon S3 |

---

# Policy Relationship Diagram

```text
                 IAM Policies

        +-------------------------+
        |   Dev-Team-Policy       |
        +-------------------------+
               │
               │ Attached To
               ▼
          +-------------+
          | Dev-Team    |
          +-------------+

      Full Amazon S3 Access
      Create EC2 Instances
      Full Amazon RDS Access



        +-------------------------+
        |   Ops-Team-Policy       |
        +-------------------------+
               │
               │ Attached To
               ▼
          +-------------+
          | Ops-Team    |
          +-------------+

      Full CloudWatch Access
      Full Billing Access
      List EC2 Resources
      List S3 Resources
```

---

---

# Verification Step 1: Verify Policy Number 1 Exists

## Navigation

```text
AWS Console
→ IAM
→ Policies
```

## Action

Search for:

`Dev-Team-Policy`

## Expected Output

The policy should appear in the policy list.

Example:

| Policy Name | Type |
|-------------|------|
| `Dev-Team-Policy` | Customer Managed |

## Why does this confirm success?

If the policy appears in the IAM Policies list, AWS has successfully created and stored the customer-managed IAM policy.

---

# Verification Step 2: Verify Policy Number 2 Exists

## Navigation

```text
AWS Console
→ IAM
→ Policies
```

## Action

Search for:

`Ops-Team-Policy`

## Expected Output

The policy should appear in the policy list.

Example:

| Policy Name | Type |
|-------------|------|
| `Ops-Team-Policy` | Customer Managed |

## Why does this confirm success?

This confirms that the second customer-managed IAM policy has been successfully created and is available for attachment to IAM users, groups, or roles.

---

# Verification Step 3: Verify Dev-Team Policy Attachment

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev-Team
→ Permissions
```

## Action

Open the **Permissions** tab.

## Expected Output

The attached policies section should contain:

| Attached Policy |
|-----------------|
| `Dev-Team-Policy` |

## Why does this confirm success?

Seeing the policy attached to the Dev-Team group confirms that every user belonging to this IAM group will automatically inherit the permissions defined in the policy.

---

# Verification Step 4: Verify Ops-Team Policy Attachment

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops-Team
→ Permissions
```

## Action

Open the **Permissions** tab.

## Expected Output

The attached policies section should contain:

| Attached Policy |
|-----------------|
| `Ops-Team-Policy` |

## Why does this confirm success?

This confirms that all users who belong to the Ops-Team IAM group will automatically receive the permissions granted by the attached policy.

---

# Verification Step 5: Verify Dev-Team-Policy Permissions

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Dev-Team-Policy
→ Permissions
```

## Action

Open the policy and review the JSON document.

## Expected Output

The JSON policy should contain permissions similar to:

| AWS Service | Permission |
|-------------|------------|
| Amazon S3 | `s3:*` |
| Amazon EC2 | `ec2:RunInstances` |
| Amazon RDS | `rds:*` |

## Why does this confirm success?

These actions match the assignment requirements:

- Full Amazon S3 access
- Only permission to create EC2 instances
- Full Amazon RDS access

This verifies that the policy grants the intended permissions.

---

# Verification Step 6: Verify Ops-Team-Policy Permissions

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Ops-Team-Policy
→ Permissions
```

## Action

Open the policy and review the JSON document.

## Expected Output

The policy should include permissions similar to:

| AWS Service | Permission |
|-------------|------------|
| Amazon CloudWatch | `cloudwatch:*` |
| Billing | `billing:*`, `aws-portal:*`, `budgets:*`, `ce:*`, `cur:*` |
| Amazon EC2 | `ec2:Describe*` |
| Amazon S3 | `s3:ListAllMyBuckets`, `s3:ListBucket`, `s3:GetBucketLocation` |

## Why does this confirm success?

This verifies that:

- CloudWatch has full administrative permissions.
- Billing services are fully accessible.
- EC2 permissions are limited to viewing resources.
- Amazon S3 permissions are limited to listing bucket information.

This matches the assignment requirements exactly.

---

# Verification Flow

```text
Policies Created
        │
        ▼
Policy 1 Visible
        │
        ▼
Policy 2 Visible
        │
        ▼
Attached to Dev-Team
        │
        ▼
Attached to Ops-Team
        │
        ▼
Permissions Match Assignment
        │
        ▼
Assignment Verified Successfully
```

---

# Cleanup Step 1: Detach Policy from Dev-Team

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev-Team
→ Permissions
```

## Action

1. Select `Dev-Team-Policy`.
2. Click **Remove** or **Detach**.
3. Confirm the action.

## Why is this required?

AWS does not allow deletion of a customer-managed IAM policy while it is attached to a user, group, or role.

The policy must first be detached from every IAM identity.

## What happens if this resource is not deleted?

- The Dev-Team group will continue to receive the assigned permissions.
- The policy cannot be deleted until it is detached.
- Although IAM policies are free, leaving unused permissions behind increases administrative complexity and can violate the principle of least privilege.

---

# Cleanup Step 2: Detach Policy from Ops-Team

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops-Team
→ Permissions
```

## Action

1. Select `Ops-Team-Policy`.
2. Click **Remove** or **Detach**.
3. Confirm the action.

## Why is this required?

This removes the permissions from the Ops-Team group and prepares the policy for deletion.

## What happens if this resource is not deleted?

- The policy remains attached.
- The policy cannot be deleted.
- Users in the Ops-Team group continue to inherit the assigned permissions.

---

---

# Cleanup Step 3: Delete Policy Number 1

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Dev-Team-Policy
→ Delete
```

## Action

1. Search for `Dev-Team-Policy`.
2. Select the policy.
3. Click **Delete**.
4. Confirm the deletion.

## Why is this required?

Deleting the customer-managed IAM policy removes the custom permission set that was created specifically for this assignment.

Since the policy is no longer attached to any IAM group, it can now be safely deleted.

## Dependency

Depends on:

- Cleanup Step 1 (Policy detached from `Dev-Team`)

## What happens if this resource is not deleted?

- The unused policy remains in your AWS account.
- Although IAM policies do not incur charges, unused policies clutter the IAM console.
- Over time, unused policies make permission management more difficult and increase the risk of accidentally assigning outdated permissions.

---

# Cleanup Step 4: Delete Policy Number 2

## Navigation

```text
AWS Console
→ IAM
→ Policies
→ Ops-Team-Policy
→ Delete
```

## Action

1. Search for `Ops-Team-Policy`.
2. Select the policy.
3. Click **Delete**.
4. Confirm the deletion.

## Why is this required?

Deleting the policy removes the custom permissions created for the Operations Team as part of this assignment.

After it has been detached from all IAM identities, it is no longer required.

## Dependency

Depends on:

- Cleanup Step 2 (Policy detached from `Ops-Team`)

## What happens if this resource is not deleted?

- The unused policy remains available in the AWS account.
- While there is no additional AWS cost, unnecessary IAM policies can complicate permission management and make future audits more difficult.

---

# Cleanup Dependency Flow

```text
Detach Dev-Team-Policy
          │
          ▼
Detach Ops-Team-Policy
          │
          ▼
Delete Dev-Team-Policy
          │
          ▼
Delete Ops-Team-Policy
```

---

# Why is the Cleanup Order Important?

IAM policies cannot be deleted while they are attached to IAM users, groups, or roles.

The correct order is:

1. Detach the policy from all IAM identities.
2. Delete the policy.

Attempting to delete an attached policy will result in an AWS error indicating that the policy is currently in use.

Following the correct dependency order ensures that all resources are removed successfully without errors.

---
