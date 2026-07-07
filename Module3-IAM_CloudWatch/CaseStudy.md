# Module 3: Case Study 1 - Infrastructure Migration & Secure Access Delegation

## Problem Statement
XYZ Corporation is migrating from an expensive on-premises infrastructure to AWS to handle fluctuating application loads efficiently. To ensure operational security during and after this migration, establish a robust Identity and Access Management (IAM) framework that delegates controlled resource provisioning (EC2, VPC, Networking, and RDS) to specific user identities while enforcing strict security controls.

---

## Tasks To Be Performed
1. IAM User & Group Setup:
   * Create a user account with AWS Management Console access.
   * Create an IAM Group restricted to *only* launching and stopping EC2 instances using the created user account.
2. Advanced Network & Database Permissions:
   * Provide permissions allowing the user to manage Amazon VPCs, Subnets, NACLs, and Security Groups.
   * Append permissions allowing the user to provision Amazon RDS databases.
   * Explore and implement security best practices to protect resources and secure group privileges.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the IAM User with Console Access
1. Log in to the **AWS Management Console** as an Administrator and search for **IAM**.
2. Click **Users** in the left sidebar, then click **Create user**.
3. Configure the user settings:
   * **User name:** `XYZ-Migration-Admin`
   * Check **Provide user access to the AWS Management Console - Optional**.
   * **Identity Type:** Select *I want to create an IAM user*.
   * **Console password:** Select *Custom password* and enter a secure password.
   * Uncheck *Users must create a new password at next sign-in* for easy testing.
4. Click **Next** (Keep permissions unassigned for now) -> **Next** -> **Create user**.
5. Save the **Console sign-in URL** and credentials.

### Step 2: Create Custom Managed Policies for Delegated Access

#### 1. EC2 Restrictive Policy (Launch & Stop Only)
1. Go to **Policies** > **Create policy** > Select the **JSON** tab.
2. Overwrite the template with the following policy restricting operations to start, stop, and describe EC2:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EC2LaunchAndStopOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:Describe*"
            ],
            "Resource": "*"
        }
    ]
}

```

3. Click **Next**. Name it `XYZ-EC2-Core-Policy`. Click **Create policy**.

#### 2. Network & RDS Provisioning Policy

1. Click **Create policy** again -> **JSON** tab.
2. Paste the following configuration allowing full VPC network topology creation alongside RDS management:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "NetworkProvisioning",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpc",
                "ec2:CreateSubnet",
                "ec2:CreateNetworkAcl",
                "ec2:CreateNetworkAclEntry",
                "ec2:CreateSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress"
            ],
            "Resource": "*"
        },
        {
            "Sid": "RDSManagement",
            "Effect": "Allow",
            "Action": [
                "rds:CreateDBInstance",
                "rds:DeleteDBInstance",
                "rds:DescribeDBInstances"
            ],
            "Resource": "*"
        }
    ]
}

```

3. Click **Next**. Name it `XYZ-Network-RDS-Policy`. Click **Create policy**.

### Step 3: Configure the IAM Group & Attach Policies

1. Click **User groups** in the left sidebar -> **Create group**.
2. **Group Name:** `XYZ-Migration-Group`
3. Under **Users**, check the box next to `XYZ-Migration-Admin`.
4. Under **Attach permissions policies**, search for and select:
* `XYZ-EC2-Core-Policy`
* `XYZ-Network-RDS-Policy`


5. Click **Create group**.

### Step 4: Explore & Implement Security Best Practices

To meet Task 2(c) requirements for safeguarding corporate cloud resources, deploy the following layers:

1. **Enforce Multi-Factor Authentication (MFA):** Go to **Users** > `XYZ-Migration-Admin` > **Security credentials** tab. Click **Assign MFA device** to set up virtual authenticator apps.
2. **Implement Permissions Boundaries:** Apply a boundary template to the user identity ensuring that even if policies are modified maliciously, the execution limits cannot exceed basic EC2/VPC boundaries.
3. **Password Policy Enforcement:** Configure Account Settings to mandate complex corporate passwords (minimum 12 characters, symbols, and routine rotations).

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To tear down the test security infrastructure safely, follow this specific workflow:

### 1. Remove User from Group and Delete User

1. Go to **IAM** > **Users**.
2. Click on `XYZ-Migration-Admin`.
3. Go to the **Groups** tab, select `XYZ-Migration-Group`, and click **Remove**.
4. Go back to the **Users** menu, select `XYZ-Migration-Admin`, and click **Delete**. Confirm by typing the user name.

### 2. Delete the IAM Group

1. Go to **IAM** > **User groups**.
2. Select `XYZ-Migration-Group`.
3. Click **Delete** and confirm. (This detaches attached policies automatically).

### 3. Delete Custom Policies

1. Go to **IAM** > **Policies**.
2. Search for `XYZ-EC2-Core-Policy`, select it, click **Actions** > **Delete**.
3. Search for `XYZ-Network-RDS-Policy`, select it, click **Actions** > **Delete**.
