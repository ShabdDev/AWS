# Module 3: Assignment 4 - IAM Roles and Switch Role Configuration

## Problem Statement
XYZ Corporation requires a security framework where specific users can elevate their permissions temporarily to manage network infrastructure and databases. Implement an IAM Role that provides complete access to Amazon VPC and Amazon DynamoDB, restricted to designated users, and test the cross-account/identity switching capability.

---

## Tasks To Be Performed
1. Create an IAM Role which allows specific users (`Dev1` and `Dev2` from previous tasks) to have complete access to VPCs and DynamoDB.
2. Log in as `Dev1` and switch to the newly created role to verify the configuration.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the Custom Trust Policy IAM Role
1. Log in to the **AWS Management Console** as an Administrator and navigate to the **IAM Dashboard**.
2. Click on **Roles** in the left sidebar, then click **Create role**.
3. Select **Custom trust policy** under Trusted entity type. 
4. Paste the following JSON script to specify that only `Dev1` and `Dev2` can assume this role (Replace `YOUR_ACCOUNT_ID` with your actual 12-digit AWS account ID):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::YOUR_ACCOUNT_ID:user/Dev1",
          "arn:aws:iam::YOUR_ACCOUNT_ID:user/Dev2"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

5. Click **Next**.
6. On the **Add permissions** screen, search and attach the following AWS-managed policies:
* Check **AmazonVPCFullAccess**
* Check **AmazonDynamoDBFullAccess**


7. Click **Next**. Name the role `XYZ-VPC-Dynamo-Role`.
8. Click **Create role**. Open the newly created role and copy its **ARN** (e.g., `arn:aws:iam::YOUR_ACCOUNT_ID:role/XYZ-VPC-Dynamo-Role`).

---

### Step 2: Grant Permission to Users to Assume the Role

1. Click on **Policies** -> **Create policy**.
2. Select **JSON** tab and paste the following permission rule allowing users to invoke the `sts:AssumeRole` action (Replace with your actual Role ARN):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::YOUR_ACCOUNT_ID:role/XYZ-VPC-Dynamo-Role"
        }
    ]
}

```

3. Click **Next**. Name the policy `XYZ-Allow-AssumeRole-Policy` and click **Create policy**.
4. Go to **User groups** -> Select **Dev Team** (containing Dev1 and Dev2) -> **Permissions** tab -> **Add permissions** -> **Attach policies**.
5. Search for `XYZ-Allow-AssumeRole-Policy`, select it, and click **Add permissions**.

---

### Step 3: Test the Feature (Switch Role as Dev1)

1. Give `Dev1` a console password via the IAM console if not done already (**Users** -> `Dev1` -> **Security credentials** -> **Enable console access**).
2. Log out from your Admin account and sign back in using the **IAM User sign-in link** as `Dev1`.
3. Once logged in as `Dev1`, click on the **username/account identity menu** in the top-right corner of the AWS banner.
4. Click on **Switch Role**.
5. Enter the target credentials:
* **Account:** Your 12-digit AWS Account ID.
* **Role:** `XYZ-VPC-Dynamo-Role`
* **Display Name:** `Dev-Elevated-Access` (Choose any display tag/color).


6. Click **Switch Role**.
7. Your top-right badge will update to show you are acting under the assumed role identity. Navigate to **VPC Dashboard** and **DynamoDB** to verify you have unrestricted creation privileges.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To avoid dangling security credentials and remove unused permission configurations, follow these clean-up steps:

### 1. Delete the IAM Role

1. Sign back into the AWS Console using your main Administrator identity.
2. Go to **IAM** > **Roles**.
3. Search for `XYZ-VPC-Dynamo-Role`, select it, and click **Delete**. Confirm by typing the role name.

### 2. Remove Custom Policy from Dev Group

1. Go to **IAM** > **User groups** > Select **Dev Team** -> **Permissions** tab.
2. Find `XYZ-Allow-AssumeRole-Policy` and click **Remove**.
3. Navigate to **Policies** in the sidebar, search for `XYZ-Allow-AssumeRole-Policy`, click **Actions** > **Delete**, and confirm the prompt.
