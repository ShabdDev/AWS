# Module 3: Assignment 3 - Custom IAM Policies and Group Mapping

## Problem Statement
XYZ Corporation requires custom permission boundaries to maintain secure access control. Implement two distinct customer-managed IAM policies to restrict or allow specific actions across Amazon S3, EC2, RDS, CloudWatch, and Billing services, and map them to the existing Dev and Ops teams.

---

## Tasks To Be Performed
1. Create **Policy 1** with the following permissions:
   * Full Access to Amazon S3.
   * Permission to *only* create Amazon EC2 instances (No delete/modify actions).
   * Full Access to Amazon RDS.
2. Create **Policy 2** with the following permissions:
   * Full Access to CloudWatch and AWS Billing dashboard.
   * Read-only capability to *only* list EC2 and S3 resources.
3. Attach **Policy 1** to the **Dev Team** group.
4. Attach **Policy 2** to the **Ops Team** group.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create Policy 1 (Dev Permissions)
1. Log in to the **AWS Management Console** and navigate to the **IAM Dashboard**.
2. Click on **Policies** in the left sidebar, then click **Create policy**.
3. Select the **JSON** tab and replace the default code with the following policy definition:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "FullS3andRDS",
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "rds:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "OnlyCreateEC2",
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances"
            ],
            "Resource": "*"
        }
    ]
}

```

4. Click **Next**. Name the policy `XYZ-Dev-Policy-1`.
5. Click **Create policy**.

---

### Step 2: Create Policy 2 (Ops Permissions)

1. In the IAM Policies dashboard, click **Create policy** again.
2. Select the **JSON** tab and paste the following policy configuration:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "FullCloudWatchAndBilling",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:*",
                "aws-portal:*",
                "billing:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ReadOnlyListEC2AndS3",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "*"
        }
    ]
}

```

3. Click **Next**. Name the policy `XYZ-Ops-Policy-2`.
4. Click **Create policy**.

---

### Step 3: Attach Policies to User Groups

1. Navigate to **User groups** from the left IAM sidebar.
2. **Attach Policy 1 to Dev Team:**
* Click on the **Dev Team** group name.
* Go to the **Permissions** tab, click **Add permissions** dropdown, and choose **Attach policies**.
* Search for `XYZ-Dev-Policy-1`, check the box next to it, and click **Add permissions**.

3. **Attach Policy 2 to Ops Team:**
* Go back to **User groups** and click on the **Ops Team** group name.
* Go to the **Permissions** tab, click **Add permissions** dropdown, and choose **Attach policies**.
* Search for `XYZ-Ops-Policy-2`, check the box next to it, and click **Add permissions**.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To detach and delete custom policies to restore your AWS account back to its initial environment state, use the following steps:

### 1. Detach Policies from Groups

1. Go to **User groups** -> Click `Dev Team` -> **Permissions** tab.
2. Select `XYZ-Dev-Policy-1` and click **Remove**.
3. Go to **User groups** -> Click `Ops Team` -> **Permissions** tab.
4. Select `XYZ-Ops-Policy-2` and click **Remove**.

### 2. Delete the Custom Policies

1. Navigate to the **Policies** section in the IAM Dashboard.
2. Search for `XYZ-Dev-Policy-1`, select it, click **Actions** > **Delete**. Confirm the prompt.
3. Search for `XYZ-Ops-Policy-2`, select it, click **Actions** > **Delete**. Confirm the prompt.
