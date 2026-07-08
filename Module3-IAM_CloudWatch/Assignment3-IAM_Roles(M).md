# Module 3 - Assignment 3 - IAM Roles

## Problem Statement

You work for XYZ Corporation. To maintain the security of the AWS account and the AWS resources, you have been asked to implement a solution that helps easily recognize and monitor different users.

## Tasks To Be Performed

1. Create an IAM Role which only allows `user1` and `user2` (created in Assignment 1) to have complete access to Amazon VPC and Amazon DynamoDB.
2. Log in as `user1` and switch to the IAM Role to verify that the role works correctly.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|--------------------|--------------|------|
| IAM | Yes | No | IAM is provided at no additional cost. |
| Amazon VPC | Yes | No | Creating and managing VPCs does not incur charges. Charges apply only to billable resources inside the VPC such as NAT Gateway, EC2, etc. |
| Amazon DynamoDB | Yes | No (within Free Tier limits) | DynamoDB offers a Free Tier. Simply creating permissions does not incur charges. |
| AWS Management Console | Yes | No | Used only for configuration. |

### Free Tier Eligibility

This assignment is **completely Free Tier eligible**.

### AWS Services That Consume Credits

None.

This assignment only creates IAM permissions and tests role switching.

### Recommendation

This assignment can be completed safely even without AWS promotional credits because no billable resources are created.

---

# Architecture Diagram

```text
                    AWS Account
                         │
      ┌──────────────────┴──────────────────┐
      │                                     │
   IAM Users                           IAM Role
┌──────────────┐                  ┌─────────────────────┐
│ user1        │                  │ VPC-DynamoDB-Role   │
│ user2        │ ───────────────► │                     │
└──────────────┘  Assume Role     │ AmazonVPCFullAccess │
                                  │ AmazonDynamoDBFullAccess
                                  └─────────────────────┘
```

---

# Dependency Flow

```text
IAM Users (Assignment 1)

        │

        ▼

Create IAM Role

        │

        ▼

Attach Permission Policies

        │

        ▼

Configure Trusted Entity

        │

        ▼

Login as user1

        │

        ▼

Switch Role

        │

        ▼

Verify Access
```

---

# Prerequisites

Before beginning this assignment, ensure the following resources already exist.

| Resource | Required | Reason |
|----------|----------|--------|
| AWS Account | Yes | Required to access AWS services. |
| IAM User `user1` | Yes | Used to test role switching. |
| IAM User `user2` | Yes | Allowed to assume the role. |
| Administrator Account | Yes | Required to create IAM Roles and policies. |

> **Note:** This assignment assumes that `user1` and `user2` were created during **Module 3 - Assignment 1**.

---

# Step 1: Create an IAM Role

## Navigation

```text
AWS Console

→ IAM

→ Roles

→ Create Role
```

## Configuration

| Setting | Value |
|---------|-------|
| Trusted Entity Type | `AWS Account` |
| Account | `This AWS Account` |
| Role Name | `VPC-DynamoDB-Role` |
| Description | `Role for user1 and user2 to manage VPC and DynamoDB` |

## Why is this step required?

IAM Roles are used to grant temporary permissions.

Instead of assigning permissions directly to users, users temporarily assume a role whenever they require elevated access.

This improves security because:

- permissions are centralized
- permissions are easier to manage
- users normally operate with fewer privileges
- temporary credentials are issued instead of permanent elevated permissions

## Dependency

This step depends on:

- AWS Account
- IAM service
- Existing IAM users (`user1` and `user2`)

## What happens if this step is skipped?

Without creating the IAM Role:

- users cannot switch roles
- VPC permissions cannot be granted through a role
- DynamoDB permissions cannot be granted through a role
- Task 2 cannot be completed

---

# Step 2: Configure the Trusted Entity for the Role

## Navigation

```text
AWS Console

→ IAM

→ Roles

→ Create Role

→ Trusted Entity Type
```

## Configuration

Select the following options while creating the role:

| Setting | Value |
|---------|-------|
| Trusted Entity Type | `AWS account` |
| AWS Account | `This account` |
| Use Case | Not Applicable |

After selecting **This account**, proceed to the permissions page.

> **Note:** The trust policy will be modified later so that only `user1` and `user2` can assume this role.

## Why is this step required?

Every IAM Role must define **who is allowed to assume the role**.

This is controlled by a **Trust Policy**.

Without a trusted entity, AWS does not know which users or services are allowed to use the role.

## Dependency

Depends on:

- Step 1 (IAM Role creation)

## What happens if this step is skipped?

The role cannot be assumed by anyone because no trusted principal has been defined.

---

# Step 3: Attach Permission Policies to the Role

## Navigation

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role

→ Add Permissions
```

## Configuration

Attach the following AWS Managed Policies:

| Policy | Purpose |
|---------|----------|
| `AmazonVPCFullAccess` | Allows complete management of Amazon VPC resources. |
| `AmazonDynamoDBFullAccess` | Allows complete management of Amazon DynamoDB resources. |

After selecting both policies, choose **Next** and create the role.

## Why is this step required?

A role without permission policies has **zero permissions**.

The attached policies determine **what actions** the role can perform after it has been assumed.

In this assignment, the role must provide full access to:

- Amazon VPC
- Amazon DynamoDB

## Dependency

Depends on:

- Step 2 (Trusted Entity)

## What happens if this step is skipped?

Users may successfully switch to the role, but they will not be able to perform any actions because the role has no permissions.

---

# Step 4: Modify the Trust Policy to Allow Only user1 and user2

## Navigation

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role

→ Trust Relationships

→ Edit Trust Policy
```

## Configuration

Replace the default trust policy with the following JSON.

> Replace `<AWS_ACCOUNT_ID>` with your own AWS Account ID.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::<AWS_ACCOUNT_ID>:user/user1",
          "arn:aws:iam::<AWS_ACCOUNT_ID>:user/user2"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Command Explanation

Although this is a JSON policy rather than a Linux command, every element has a specific purpose.

| Policy Element | Explanation |
|---------------|-------------|
| `"Version"` | Specifies the IAM policy language version. `2012-10-17` is the current version used by AWS. |
| `"Statement"` | Contains one or more permission statements. |
| `"Effect"` | `Allow` grants the specified permission. |
| `"Principal"` | Specifies who is allowed to assume the role. |
| `"AWS"` | Indicates AWS IAM identities. |
| `"arn:aws:iam::<AWS_ACCOUNT_ID>:user/user1"` | ARN of IAM user `user1`. Only this user is trusted. |
| `"arn:aws:iam::<AWS_ACCOUNT_ID>:user/user2"` | ARN of IAM user `user2`. Only this user is trusted. |
| `"Action"` | Specifies the operation being allowed. |
| `"sts:AssumeRole"` | Allows the trusted IAM users to assume this role using AWS Security Token Service (STS). |

## Why is this step required?

Permission policies define **what the role can do**.

The trust policy defines **who can use the role**.

Both are required.

This step ensures that only:

- `user1`
- `user2`

can assume the role.

No other IAM user will be able to switch to this role.

## Dependency

Depends on:

- Step 3 (Role with attached permission policies)

## What happens if this step is skipped?

If the trust policy is not updated:

- unauthorized users may be able to assume the role (depending on the default trust configuration), or
- the intended users may not be able to assume the role at all.

The assignment requirement that **only `user1` and `user2` have access** would not be satisfied.

---

# Step 5: Grant `user1` and `user2` Permission to Assume the Role

## Navigation

Perform the following steps for both `user1` and `user2`.

```text
AWS Console

→ IAM

→ Users

→ user1

→ Add Permissions

→ Create Inline Policy
```

Repeat the same process for `user2`.

## Configuration

Select the **JSON** editor and paste the following policy.

> Replace `<AWS_ACCOUNT_ID>` with your AWS Account ID.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<AWS_ACCOUNT_ID>:role/VPC-DynamoDB-Role"
    }
  ]
}
```

Save the policy with the following name:

| Setting | Value |
|---------|-------|
| Policy Name | `Allow-Assume-VPC-DynamoDB-Role` |

Repeat the same steps for `user2`.

### Command Explanation

Although this is an IAM JSON policy rather than a Linux command, each element has a specific purpose.

| Policy Element | Explanation |
|----------------|-------------|
| `"Version"` | Specifies the IAM policy language version. |
| `"Statement"` | Contains the permission statement. |
| `"Effect"` | `Allow` grants the permission. |
| `"Action"` | Specifies the AWS action being permitted. |
| `"sts:AssumeRole"` | Allows the IAM user to switch into an IAM Role using AWS Security Token Service (STS). |
| `"Resource"` | Specifies the exact IAM Role that can be assumed. Restricting the ARN prevents the user from assuming other roles. |

## Why is this step required?

Even though the role trusts `user1` and `user2`, the users themselves also need permission to call the **AssumeRole** API.

Think of it as two-way permission:

- The **Role trusts the User**.
- The **User is allowed to assume the Role**.

Both conditions must be true.

## Dependency

Depends on:

- Step 4 (Trust Policy)

## What happens if this step is skipped?

When `user1` or `user2` tries to switch roles, AWS displays an **Access Denied** error because the user lacks permission to call `sts:AssumeRole`.

---

# Step 6: Log in as `user1`

## Navigation

```text
AWS Console

→ Sign Out

→ AWS Sign-In Page

→ IAM User

→ Enter Account ID or Alias

→ Enter Username

→ Enter Password
```

## Configuration

| Setting | Value |
|---------|-------|
| Username | `user1` |
| Password | Password created for `user1` in Assignment 1 |

## Why is this step required?

The assignment specifically requires testing the role using `user1`.

Logging in as `user1` confirms that a normal IAM user can temporarily obtain elevated permissions by assuming the IAM Role.

## Dependency

Depends on:

- Step 5

## What happens if this step is skipped?

The role cannot be tested using `user1`, so the assignment requirement remains incomplete.

---

# Step 7: Switch to the IAM Role

## Navigation

```text
AWS Console

→ Click Username (Top Right)

→ Switch Role

→ Switch Role
```

## Configuration

| Setting | Value |
|---------|-------|
| Account ID | `<AWS_ACCOUNT_ID>` |
| Role Name | `VPC-DynamoDB-Role` |
| Display Name | `VPCRole` *(Optional)* |
| Color | Any *(Optional)* |

Click **Switch Role**.

## Why is this step required?

Switching roles requests temporary AWS Security Token Service (STS) credentials.

These temporary credentials inherit all permissions attached to the IAM Role.

This is much more secure than assigning permanent administrator permissions directly to IAM users.

## Dependency

Depends on:

- Step 6

## What happens if this step is skipped?

`user1` will continue using only its own IAM permissions and will not receive the VPC and DynamoDB permissions granted by the role.

---

# Step 8: Verify the Active Role

## Navigation

```text
AWS Console

→ Observe the Account Menu (Top Right)
```

## Configuration

After switching roles, observe that:

- The display name changes (if provided).
- The active role name is shown.
- AWS Console indicates that you are operating using an assumed role.

## Why is this step required?

This confirms that AWS successfully issued temporary credentials and that the role assumption was successful.

Without verifying the active role, it is difficult to distinguish whether actions are being performed using the original IAM user or the assumed role.

## Dependency

Depends on:

- Step 7

## What happens if this step is skipped?

You may mistakenly believe that you are using the role while actually operating with the original IAM user permissions.

---

# Verification / Output Checking

## Verification Step 1

### Action

Verify that the IAM Role has been created successfully.

### AWS Console Action

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role
```

### Expected Output

The IAM Role should appear in the Roles list.

Example:

```text
Role Name
-----------------------
VPC-DynamoDB-Role
```

### Why does this confirm success?

If the role appears in the IAM Roles list, AWS has successfully created the IAM Role.

---

## Verification Step 2

### Action

Verify that the correct permission policies are attached to the IAM Role.

### AWS Console Action

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role

→ Permissions
```

### Expected Output

The following managed policies should be attached.

```text
AmazonVPCFullAccess

AmazonDynamoDBFullAccess
```

### Why does this confirm success?

These policies grant the permissions required by the assignment.

If either policy is missing, the role will not have complete access to both AWS services.

---

## Verification Step 3

### Action

Verify the Trust Relationship.

### AWS Console Action

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role

→ Trust Relationships
```

### Expected Output

The trust policy should contain only the following principals.

```text
user1

user2
```

or their corresponding IAM User ARNs.

### Why does this confirm success?

This confirms that only the specified IAM users are trusted to assume the role.

No other IAM user should appear in the trust policy.

---

## Verification Step 4

### Action

Log in as `user1`.

### AWS Console Action

```text
AWS Sign-In Page

→ IAM User Login

→ Username: user1

→ Password
```

### Expected Output

`user1` should successfully sign in to the AWS Management Console.

### Why does this confirm success?

The assignment requires testing the IAM Role using `user1`.

Successful login confirms that the IAM user is functioning correctly.

---

## Verification Step 5

### Action

Switch from `user1` to the IAM Role.

### AWS Console Action

```text
AWS Console

→ Username

→ Switch Role

→ Account ID

→ Role Name

→ Switch Role
```

### Expected Output

The AWS Console should display the role name or the chosen display name.

Example:

```text
Currently using role:

VPC-DynamoDB-Role
```

### Why does this confirm success?

This confirms that AWS Security Token Service (STS) successfully issued temporary credentials for the IAM Role.

---

## Verification Step 6

### Action

Open the Amazon VPC service.

### AWS Console Action

```text
AWS Console

→ VPC
```

### Expected Output

The VPC Dashboard should open without displaying an **Access Denied** message.

### Why does this confirm success?

This verifies that the assumed IAM Role provides the required permissions for Amazon VPC.

---

## Verification Step 7

### Action

Open the Amazon DynamoDB service.

### AWS Console Action

```text
AWS Console

→ DynamoDB
```

### Expected Output

The DynamoDB Dashboard should open successfully.

You should be able to:

- View existing tables.
- Create a new table (if desired).
- Delete a table (if permitted).

No permission-related errors should appear.

### Why does this confirm success?

This confirms that the IAM Role has the `AmazonDynamoDBFullAccess` policy attached and that the permissions are working as expected.

---

# Cleanup / Cost Optimization

> **Note:** IAM resources do not incur usage charges. However, deleting unused IAM resources is considered an AWS security best practice because it follows the principle of least privilege and reduces the attack surface.

---

## Cleanup Step 1

### Navigation

```text
AWS Console

→ Sign Out

or

→ Switch Back
```

### Action

Exit the assumed IAM Role session and return to your original IAM user or sign out of the AWS Console.

### Why is this required?

This ends the temporary elevated access provided by the IAM Role.

### What happens if this resource is not deleted?

The temporary session will eventually expire automatically, but it is a good security practice to stop using elevated privileges when they are no longer required.

---

## Cleanup Step 2

### Navigation

```text
AWS Console

→ IAM

→ Users

→ user1

→ Permissions
```

Repeat the same process for `user2`.

### Action

Delete the inline policy:

```text
Allow-Assume-VPC-DynamoDB-Role
```

### Why is this required?

This removes the users' ability to assume the IAM Role.

### What happens if this resource is not deleted?

The users will continue to have permission to assume the role in the future, which may violate the principle of least privilege.

---

## Cleanup Step 3

### Navigation

```text
AWS Console

→ IAM

→ Roles

→ VPC-DynamoDB-Role

→ Delete
```

### Action

Delete the IAM Role `VPC-DynamoDB-Role`.

### Why is this required?

The role is no longer needed after completing the assignment.

Removing unused IAM Roles helps maintain a clean and secure AWS environment.

### What happens if this resource is not deleted?

Although IAM Roles do not incur charges, unused roles remain available for future use. Keeping unnecessary roles increases administrative overhead and can become a security risk if they are accidentally granted additional permissions later.

---

## Cleanup Dependency Order

```text
Exit Role Session
        │
        ▼
Remove AssumeRole Permission from Users
        │
        ▼
Delete IAM Role
```

### Why is the deletion order important?

The deletion order ensures that:

1. Active role sessions are ended.
2. IAM users lose permission to assume the role.
3. The IAM Role can then be safely deleted.

Deleting resources in this order avoids permission inconsistencies and follows AWS IAM security best practices.

---

