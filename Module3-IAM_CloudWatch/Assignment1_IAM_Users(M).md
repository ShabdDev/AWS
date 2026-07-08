# Module 3 - Introduction to IAM and CloudWatch

# Assignment 1 - IAM Users

## Problem Statement

You work for XYZ Corporation. To maintain the security of the AWS account and the resources, you have been asked to implement a solution that can help easily recognize and monitor the different users.

## Tasks To Be Performed

1. Create four IAM users named `Dev1`, `Dev2`, `Test1`, and `Test2`.
2. Create two IAM groups named `Dev Team` and `Ops Team`.
3. Add `Dev1` and `Dev2` to the `Dev Team`.
4. Add `Dev1`, `Test1`, and `Test2` to the `Ops Team`.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|--------------------|--------------|-------|
| AWS Identity and Access Management (IAM) | Yes | No | IAM is a global AWS service and does not incur additional charges for creating users, groups, or assigning memberships. |

### Free Tier Eligibility

This assignment is **completely AWS Free Tier eligible.**

### AWS Promotional Credits

This assignment **does not consume AWS promotional credits** because AWS IAM is provided at no additional cost.

### Recommendation

This assignment can be completed safely even if you do not have AWS promotional credits because no billable AWS resources are created.

---

# Architecture Diagram

```text
                        AWS Account
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
        │                                           │
   IAM Group                                   IAM Group
   Dev Team                                     Ops Team
        │                                           │
   ┌────┴────┐                            ┌─────────┼─────────┐
   │         │                            │         │         │
 Dev1      Dev2                         Dev1      Test1     Test2
```

---

# Implementation Flow

```text
Start
   │
   ▼
Open AWS IAM Console
   │
   ▼
Create IAM Users
   │
   ▼
Create IAM Groups
   │
   ▼
Add Users to Groups
   │
   ▼
Verify Group Membership
   │
   ▼
Assignment Completed
```

---

# Step 1: Sign in to AWS Management Console

## Navigation

```text
AWS Console
→ Sign in
```

## Configuration

* Sign in using an AWS account that has administrative permissions to create IAM users and IAM groups.

## Why is this step required?

AWS resources can only be created after authenticating into your AWS account. Administrative permissions are required because creating IAM users and groups changes the account's identity management configuration.

## Dependency

None.

## What happens if this step is skipped?

You cannot access AWS services or create IAM resources.

---

# Step 2: Open the IAM Service

## Navigation

```text
AWS Console
→ Search Bar
→ IAM
```

## Configuration

* **Service:** `IAM (Identity and Access Management)`

## Why is this step required?

IAM is the AWS service responsible for managing users, groups, permissions, and authentication. All tasks in this assignment are performed inside IAM.

## Dependency

Depends on:

- Step 1

## What happens if this step is skipped?

You will not be able to create IAM users or IAM groups because those resources belong to the IAM service.

---

# Step 3: Create IAM User `Dev1`

## Navigation

```text
AWS Console
→ IAM
→ Users
→ Create User
```

## Configuration

| Property | Value |
|----------|-------|
| User name | `Dev1` |
| Provide user access to the AWS Management Console | Leave unchecked |
| Console Password | Not Required |
| Access Keys | Not Required |

> **Note:** This assignment only requires creating IAM users. It does not require AWS Management Console access or programmatic access.

## Why is this step required?

An IAM user represents an individual identity inside an AWS account. Creating separate users instead of sharing one account follows AWS security best practices by allowing each person or application to have its own identity.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

The required user `Dev1` will not exist and cannot be added to any IAM group.

---

# Step 4: Create IAM User `Dev2`

## Navigation

```text
AWS Console
→ IAM
→ Users
→ Create User
```

## Configuration

| Property | Value |
|----------|-------|
| User name | `Dev2` |
| Provide user access to the AWS Management Console | Leave unchecked |
| Console Password | Not Required |
| Access Keys | Not Required |

## Why is this step required?

This creates the second developer identity required by the assignment.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

The `Dev Team` group cannot contain both required developer users.

---

# Step 5: Create IAM User `Test1`

## Navigation

```text
AWS Console
→ IAM
→ Users
→ Create User
```

## Configuration

| Property | Value |
|----------|-------|
| User name | `Test1` |
| Provide user access to the AWS Management Console | Leave unchecked |
| Console Password | Not Required |
| Access Keys | Not Required |

## Why is this step required?

This creates one of the testing team identities required for group membership.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

`Test1` cannot be added to the `Ops Team` group.

---

# Step 6: Create IAM User `Test2`

## Navigation

```text
AWS Console
→ IAM
→ Users
→ Create User
```

## Configuration

| Property | Value |
|----------|-------|
| User name | `Test2` |
| Provide user access to the AWS Management Console | Leave unchecked |
| Console Password | Not Required |
| Access Keys | Not Required |

> **Note:** This assignment only requires creating IAM users. AWS Management Console access and programmatic access are not required.

## Why is this step required?

This creates the fourth IAM user required by the assignment. Each IAM user represents a unique identity within the AWS account, making it easier to manage permissions and track user activities.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

`Test2` will not exist and therefore cannot be added to the `Ops Team` group, making the assignment incomplete.

---

# Step 7: Create IAM Group `Dev Team`

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Create Group
```

## Configuration

| Property | Value |
|----------|-------|
| User Group Name | `Dev Team` |
| Attach Permission Policies | None |

> **Note:** The assignment only asks to create the group. No IAM policies need to be attached.

## Why is this step required?

IAM Groups are used to organize users with similar responsibilities. Instead of assigning permissions individually to every developer, permissions can later be attached to the group, automatically applying them to all members.

Using groups makes administration easier, reduces configuration mistakes, and follows AWS best practices.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

There will be no `Dev Team` group available, so `Dev1` and `Dev2` cannot be grouped together.

---

# Step 8: Create IAM Group `Ops Team`

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Create Group
```

## Configuration

| Property | Value |
|----------|-------|
| User Group Name | `Ops Team` |
| Attach Permission Policies | None |

> **Note:** No permissions are attached because the assignment only requires creating the group.

## Why is this step required?

The `Ops Team` group provides a logical container for operational users. In real-world AWS environments, operations engineers often share common permissions such as monitoring infrastructure, managing backups, or viewing CloudWatch logs. Groups make assigning these permissions much easier.

## Dependency

Depends on:

- Step 2

## What happens if this step is skipped?

The required operations group will not exist, preventing the required users from being added to it.

---

# Step 9: Add `Dev1` and `Dev2` to the `Dev Team` Group

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev Team
→ Add Users
```

## Configuration

Select the following users:

| User | Add to Group |
|------|--------------|
| `Dev1` | Yes |
| `Dev2` | Yes |
| `Test1` | No |
| `Test2` | No |

Click **Add Users**.

## Why is this step required?

IAM Groups organize users based on their job roles. Adding `Dev1` and `Dev2` to the `Dev Team` group allows both users to be managed together. If permissions are attached to the group later, both users will automatically inherit them.

This approach is significantly easier than assigning permissions individually.

## Dependency

Depends on:

- Step 3
- Step 4
- Step 7

## What happens if this step is skipped?

Although the users and group exist, they will not be associated with each other. Future permissions assigned to the `Dev Team` group will not apply to `Dev1` and `Dev2`.

---

# Step 10: Add `Dev1`, `Test1`, and `Test2` to the `Ops Team` Group

## Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops Team
→ Add Users
```

## Configuration

Select the following users:

| User | Add to Group |
|------|--------------|
| `Dev1` | Yes |
| `Dev2` | No |
| `Test1` | Yes |
| `Test2` | Yes |

Click **Add Users**.

## Why is this step required?

This step completes the assignment requirements by placing the specified users into the `Ops Team` group.

Notice that `Dev1` belongs to both the `Dev Team` and `Ops Team` groups. AWS IAM allows a single user to be a member of multiple groups. This is commonly used in organizations where an employee has multiple responsibilities.

## Dependency

Depends on:

- Step 3
- Step 5
- Step 6
- Step 8

## What happens if this step is skipped?

The required group memberships will not exist, making the assignment incomplete. Any future permissions assigned to the `Ops Team` group would not be inherited by these users.

---

# Final IAM Structure

```text
AWS Account
│
├── IAM Users
│   ├── Dev1
│   ├── Dev2
│   ├── Test1
│   └── Test2
│
├── Dev Team
│   ├── Dev1
│   └── Dev2
│
└── Ops Team
    ├── Dev1
    ├── Test1
    └── Test2
```

---

# Verification

## Verification Step 1: Verify All IAM Users Have Been Created

### Action

Open the IAM Users page and verify that all four users are listed.

### Navigation

```text
AWS Console
→ IAM
→ Users
```

### Expected Output

The **Users** page should display the following users:

| User Name |
|-----------|
| `Dev1` |
| `Dev2` |
| `Test1` |
| `Test2` |

### Why does this confirm success?

This confirms that all four IAM users required by the assignment have been created successfully.

---

## Verification Step 2: Verify the `Dev Team` Group Exists

### Action

Open the IAM User Groups page and select the `Dev Team` group.

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev Team
```

### Expected Output

The group details should show:

| Property | Expected Value |
|----------|----------------|
| Group Name | `Dev Team` |
| Members | `Dev1`, `Dev2` |

### Why does this confirm success?

This verifies that the `Dev Team` group exists and contains the correct developer users as required by the assignment.

---

## Verification Step 3: Verify the `Ops Team` Group Exists

### Action

Open the IAM User Groups page and select the `Ops Team` group.

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops Team
```

### Expected Output

The group details should show:

| Property | Expected Value |
|----------|----------------|
| Group Name | `Ops Team` |
| Members | `Dev1`, `Test1`, `Test2` |

### Why does this confirm success?

This confirms that the `Ops Team` group has been created and contains the correct users specified in the assignment.

---

## Verification Step 4: Verify That `Dev1` Belongs to Both Groups

### Action

Open the details page for the `Dev1` IAM user.

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Dev1
→ Groups
```

### Expected Output

The **Groups** section should display:

| Group Membership |
|------------------|
| `Dev Team` |
| `Ops Team` |

### Why does this confirm success?

AWS IAM allows a user to belong to multiple groups. Seeing `Dev1` in both groups confirms that the user has been added correctly according to the assignment requirements.

---

## Verification Step 5: Verify Individual Group Memberships

### Action

Review each user's group membership.

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Select User
→ Groups
```

### Expected Output

| IAM User | Expected Group Membership |
|----------|----------------------------|
| `Dev1` | `Dev Team`, `Ops Team` |
| `Dev2` | `Dev Team` |
| `Test1` | `Ops Team` |
| `Test2` | `Ops Team` |

### Why does this confirm success?

This final verification ensures that every IAM user belongs to the correct group(s) exactly as specified in the assignment.

---

# Cleanup / Cost Optimization

> **Note:** AWS IAM users and groups do not incur charges. However, deleting unused IAM resources is considered a security best practice because it removes unnecessary identities that could accidentally receive permissions in the future.

---

## Cleanup Step 1: Remove Users from the `Dev Team` Group

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev Team
→ Remove Users
```

### Action

Remove:

- `Dev1`
- `Dev2`

### Why is this required?

Removing users from the group ensures that the group has no active members before it is deleted. This is a recommended administrative practice and makes cleanup easier.

### What happens if this resource is not deleted?

The users will continue to belong to the group. If permissions are later attached to the group, these users will automatically inherit them.

---

## Cleanup Step 2: Remove Users from the `Ops Team` Group

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops Team
→ Remove Users
```

### Action

Remove:

- `Dev1`
- `Test1`
- `Test2`

### Why is this required?

This removes all user memberships before deleting the group, ensuring that no users retain unnecessary group associations.

### What happens if this resource is not deleted?

The users will remain members of the group and could inherit permissions if policies are attached later.

---

## Cleanup Step 3: Delete the `Dev Team` Group

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Dev Team
→ Delete
```

### Action

Delete the `Dev Team` group.

### Why is this required?

The assignment is complete, so the group is no longer needed. Removing unused IAM groups helps maintain a clean and organized AWS account.

### What happens if this resource is not deleted?

The unused group will remain in the AWS account. Although it does not incur charges, it can create unnecessary administrative overhead and confusion.

---

## Cleanup Step 4: Delete the `Ops Team` Group

### Navigation

```text
AWS Console
→ IAM
→ User Groups
→ Ops Team
→ Delete
```

### Action

Delete the `Ops Team` group.

### Why is this required?

Deleting the unused group removes unnecessary IAM resources and follows the principle of least privilege by eliminating identities that are no longer required.

### What happens if this resource is not deleted?

The unused group will remain available and could accidentally receive permissions in the future.

---

## Cleanup Step 5: Delete IAM User `Dev1`

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Dev1
→ Delete
```

### Action

Delete the IAM user `Dev1`.

> **Note:** If the user has any login credentials (password, access keys, MFA devices, signing certificates, or SSH public keys), AWS will require you to remove those credentials before the user can be deleted. In this assignment, no credentials were created, so the user can be deleted directly.

### Why is this required?

The IAM user was created only for this assignment. Deleting unused IAM users is a security best practice because it reduces the number of identities that could potentially be misused.

### What happens if this resource is not deleted?

Although there is no additional AWS cost, the unused IAM user will remain in your AWS account and increase administrative overhead.

---

## Cleanup Step 6: Delete IAM User `Dev2`

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Dev2
→ Delete
```

### Action

Delete the IAM user `Dev2`.

### Why is this required?

The user is no longer required after completing the assignment. Removing unnecessary IAM users helps keep the AWS account organized and secure.

### What happens if this resource is not deleted?

The unused IAM user will remain in the AWS account. While there is no cost, it increases the number of identities that administrators must manage.

---

## Cleanup Step 7: Delete IAM User `Test1`

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Test1
→ Delete
```

### Action

Delete the IAM user `Test1`.

### Why is this required?

Deleting temporary IAM users ensures that only active and required identities remain in the AWS account.

### What happens if this resource is not deleted?

The user will continue to exist and could accidentally be granted permissions in the future.

---

## Cleanup Step 8: Delete IAM User `Test2`

### Navigation

```text
AWS Console
→ IAM
→ Users
→ Test2
→ Delete
```

### Action

Delete the IAM user `Test2`.

### Why is this required?

This removes the final IAM user created during the assignment and restores the AWS account to its original state.

### What happens if this resource is not deleted?

The unused IAM user will remain in the account. Although there is no billing impact, leaving unused identities behind is not recommended from a security perspective.

---

# Cleanup Dependency Order

The cleanup must be performed in the following order:

```text
Remove Users from Groups
          │
          ▼
Delete IAM Groups
          │
          ▼
Delete IAM Users
```

## Why is the deletion order important?

IAM groups contain references to IAM users. Removing users from groups first ensures that group memberships are cleaned up before the groups are deleted. Once the groups are removed, the users can be deleted safely.

Following this dependency order makes the cleanup process organized and prevents leaving unused IAM resources in the AWS account.

## What happens if the cleanup is not performed?

- IAM users will continue to exist in the AWS account.
- IAM groups will remain available.
- Although IAM resources do **not** generate AWS charges, they increase administrative overhead.
- Unused identities may accidentally receive permissions in the future, creating unnecessary security risks.

---
