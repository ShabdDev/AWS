# Module 7 - Database Services
# Assignment 3 – Amazon DynamoDB

## Problem Statement

You work for XYZ Corporation. Their application requires a database service that can store data which can be retrieved if required. Implement a suitable service for the same.

While migrating, you are asked to perform the following tasks:

1. Create a DynamoDB table with partition key as `ID`.
2. Add `5` items to the DynamoDB table.
3. Take backup and delete the table.

---

# Part 1: Free Tier / Cost Check and DynamoDB Table Creation

---

# 1. Free Tier / Cost Check

Before creating any AWS resources, it is important to understand whether this assignment can be completed within the AWS Free Tier and whether any resource may consume AWS promotional credits.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| Amazon DynamoDB Table | Yes, within Free Tier limits | Usually no within eligible limits | DynamoDB Free Tier includes limited storage and read/write capacity. This assignment uses only a very small table with `5` items. |
| DynamoDB On-Demand Capacity | Eligible within applicable Free Tier limits | May use credits if usage exceeds Free Tier limits | For this assignment, the number of requests is extremely small. |
| DynamoDB On-Demand Backup | Not generally included in the standard DynamoDB Free Tier allowance | Yes, potentially | Backup storage can incur a small charge depending on backup size and retention duration. |
| AWS Management Console | Yes | No | There is no charge for using the AWS Management Console itself. |

## Free Tier Conclusion

* The DynamoDB table creation and insertion of `5` items use extremely small amounts of storage and request capacity and should generally remain within applicable AWS Free Tier limits.

* The **on-demand backup may incur a small cost** because DynamoDB backup storage is separately priced.

* Since the table contains only `5` small items, the backup cost should be extremely small if the backup is deleted immediately after verification.

* If you have AWS promotional credits, any eligible charges may be deducted from those credits according to the terms of your AWS credits.

* It is reasonable to complete this assignment while promotional credits are available, particularly because the assignment explicitly requires creating a DynamoDB backup.

* To minimize cost, delete the DynamoDB table and its backup immediately after completing all verification steps.

---

# 2. Architecture and Dependency Order

The assignment will be completed in the following dependency order:

~~~text
AWS Account
    |
    v
Amazon DynamoDB
    |
    v
Create DynamoDB Table
Partition Key: ID
    |
    v
Wait Until Table Status = Active
    |
    v
Add 5 Items
    |
    v
Verify All 5 Items
    |
    v
Create On-Demand Backup
    |
    v
Wait Until Backup Status = Available
    |
    v
Delete DynamoDB Table
    |
    v
Verify Table Deletion
    |
    v
Cleanup Remaining Backup
~~~

The order is important because:

* Items cannot be added before the DynamoDB table exists.
* The table should reach the `Active` state before data operations are performed.
* A backup must be created before deleting the table because the assignment explicitly requires taking a backup and then deleting the table.
* The backup should reach the `Available` state before the source table is deleted.
* The remaining backup should be deleted during cleanup to minimize AWS costs.

---

# 3. Step-by-Step Solution

## Step 1: Sign In to the AWS Management Console

### Navigation

~~~text
AWS Management Console
→ Sign in to your AWS account
→ Open the AWS Console home page
~~~

### Configuration

Use the AWS account in which you want to perform the assignment.

No AWS resource is created in this step.

### Why is this step required?

The AWS Management Console provides the graphical interface required to access Amazon DynamoDB and create the database resources for this assignment.

### Dependency

This step requires:

* A valid AWS account.
* Permission to access Amazon DynamoDB.
* Permission to create tables, add items, create backups, and delete tables.

### What happens if this step is skipped?

You will not be able to access Amazon DynamoDB or perform any of the required assignment tasks through the AWS Management Console.

---

## Step 2: Open the Amazon DynamoDB Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "DynamoDB"
→ Select DynamoDB
~~~

### Configuration

No configuration is required in this step.

### Why is this step required?

Amazon DynamoDB is the AWS database service used in this assignment. It is a fully managed NoSQL database service that stores data in tables containing items and attributes.

The assignment requires:

* A database table.
* A partition key named `ID`.
* Five data items.
* A backup of the table.

DynamoDB directly supports all these requirements.

### Dependency

This step depends on:

* **Step 1:** Successful sign-in to the AWS Management Console.

### What happens if this step is skipped?

You cannot create or manage the DynamoDB table required for the assignment.

---

## Step 3: Understand the DynamoDB Data Model Used in This Assignment

Before creating the table, understand the DynamoDB structure that will be used.

~~~text
DynamoDB
    |
    v
Table: XYZ-Employee-Table
    |
    +-------------------------------+
    |                               |
    v                               v
Partition Key                  Other Attributes
ID                             Name
                               Department
                               Email
                               City
~~~

The table will contain the following sample data:

| ID | Name | Department | Email | City |
| -- | ---- | ---------- | ----- | ---- |
| `101` | `Amit Sharma` | `IT` | `amit@example.com` | `Pune` |
| `102` | `Priya Patil` | `HR` | `priya@example.com` | `Mumbai` |
| `103` | `Rahul Verma` | `Finance` | `rahul@example.com` | `Delhi` |
| `104` | `Sneha Joshi` | `Marketing` | `sneha@example.com` | `Bengaluru` |
| `105` | `Arjun Singh` | `Operations` | `arjun@example.com` | `Hyderabad` |

### Why is this step required?

DynamoDB is a NoSQL database, so its terminology differs from traditional relational databases.

| DynamoDB Term | Simple Meaning |
| ------------- | -------------- |
| Table | Collection of related data |
| Item | One complete record |
| Attribute | One individual data field |
| Partition Key | Primary attribute used to uniquely identify and distribute items |

For this assignment, each employee is represented as one item, and the `ID` attribute uniquely identifies each item.

### Dependency

This conceptual step depends only on understanding the assignment requirements.

### What happens if this step is skipped?

The assignment may still technically be completed, but it becomes easier to make configuration mistakes, particularly when defining the partition key or entering item attributes.

---

## Step 4: Start Creating the DynamoDB Table

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Create table
~~~

### Configuration

Configure the table as follows:

| Setting | Value |
| ------- | ----- |
| Table name | `XYZ-Employee-Table` |
| Partition key | `ID` |
| Partition key data type | `String` |
| Sort key | Not required |

### Important Note About the Partition Key

The partition key name must be exactly:

`ID`

The partition key data type for this assignment will be:

`String`

This allows values such as:

* `101`
* `102`
* `103`
* `104`
* `105`

Although these values contain digits, DynamoDB will store them as strings because the partition key data type is explicitly configured as `String`.

### Why is this step required?

Every DynamoDB table requires a primary key. For this assignment, the problem statement specifically requires the partition key to be named `ID`.

The partition key is used by DynamoDB to:

* Uniquely identify each item when no sort key is configured.
* Determine how data is distributed internally.
* Retrieve individual items efficiently when the key value is known.

### Dependency

This step depends on:

* **Step 2:** Opening the DynamoDB console.
* **Step 3:** Understanding the required table structure.

### What happens if this step is skipped?

No DynamoDB table will exist, so:

* Items cannot be inserted.
* Data cannot be retrieved.
* A backup cannot be created.
* The assignment cannot be completed.

---

## Step 5: Configure Table Settings

After entering the table name and partition key, configure the table settings.

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Create table
→ Table settings
~~~

### Configuration

For this beginner assignment, select:

* **Table settings:** `Default settings`

The default settings are suitable for this assignment because the workload consists of only:

* One small DynamoDB table.
* Five small items.
* A few console operations.
* One on-demand backup.

Depending on the current AWS Console workflow, default settings may use an appropriate capacity configuration managed by DynamoDB.

Do not add:

* A sort key.
* Secondary indexes.
* DynamoDB Streams.
* Additional advanced configurations unless automatically enabled by AWS defaults.

### Why is this step required?

DynamoDB needs operational settings that determine how the table handles requests and related features.

For this small assignment, default settings reduce unnecessary complexity while satisfying all assignment requirements.

### Dependency

This step depends on:

* **Step 4:** Entering the table name and defining the `ID` partition key.

### What happens if this step is skipped?

The table creation process cannot be completed because AWS requires valid table configuration before provisioning the resource.

---

## Step 6: Create the DynamoDB Table

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Create table
→ Review configuration
→ Create table
~~~

### Configuration

Before selecting **Create table**, verify:

| Setting | Expected Value |
| ------- | -------------- |
| Table name | `XYZ-Employee-Table` |
| Partition key | `ID` |
| Partition key type | `String` |
| Sort key | None |
| Table settings | `Default settings` |

Select:

`Create table`

### Expected Initial Status

Immediately after creation, the table may temporarily show:

~~~text
Status: Creating
~~~

Wait until it changes to:

~~~text
Status: Active
~~~

Do not add items until the table reaches the `Active` state.

### Why is this step required?

Selecting **Create table** sends the table definition to DynamoDB and provisions the actual database table.

The `Active` status confirms that the table is ready to accept read and write operations.

### Dependency

This step depends on:

* **Step 4:** Table name and partition key configuration.
* **Step 5:** Table settings configuration.

### What happens if this step is skipped?

The table remains only an unsubmitted configuration in the browser. No DynamoDB resource is actually created.

If you attempt data operations before the table becomes `Active`, the operation may be unavailable or fail because the table is not yet ready.

---

## Step 7: Verify the DynamoDB Table Is Active

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Overview
~~~

### Action

Confirm that the table status is:

~~~text
Active
~~~

Also verify:

| Property | Expected Value |
| -------- | -------------- |
| Table name | `XYZ-Employee-Table` |
| Partition key | `ID (String)` |
| Table status | `Active` |

### Expected Output

~~~text
Table name: XYZ-Employee-Table
Partition key: ID (String)
Status: Active
~~~

### Why does this confirm success?

The `Active` state means DynamoDB has successfully provisioned the table and it is ready for read and write operations.

The `ID (String)` value confirms that the required partition key was configured correctly.

### Why is this step required?

Verifying the table before inserting data prevents confusion caused by attempting operations while the table is still being created.

### Dependency

This step depends on:

* **Step 6:** Successful table creation.

### What happens if this step is skipped?

You may attempt to insert items before the table is fully available, or you may fail to notice an incorrect table name or partition key configuration.

---

## Step 8: Open the DynamoDB Item Explorer

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
~~~

Depending on the current AWS Console layout, you can also navigate through:

~~~text
AWS Management Console
→ DynamoDB
→ Explore items
→ Select XYZ-Employee-Table
~~~

### Configuration

No data is added yet in this step.

You should see the selected table:

`XYZ-Employee-Table`

### Why is this step required?

The **Explore table items** interface allows you to:

* Create new items.
* View existing items.
* Edit items.
* Delete items.
* Query or scan table data.

This is where the five required items will be inserted.

### Dependency

This step depends on:

* **Step 7:** The DynamoDB table must exist and have the `Active` status.

### What happens if this step is skipped?

You will not reach the console interface used to manually add the five required items.

---

## Step 9: Prepare to Create the First Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

The item creation screen should automatically include the required partition key:

* **Attribute name:** `ID`
* **Type:** `String`

For the first item, enter:

* **ID:** `101`

Additional attributes will be added in the next part of this document.

### Why is this step required?

Every DynamoDB item in this table must contain the `ID` partition key because it is part of the table's primary key schema.

Since no sort key exists, each `ID` value must uniquely identify an item.

### Dependency

This step depends on:

* **Step 8:** Opening the DynamoDB item explorer.

### What happens if this step is skipped?

The first item cannot be created, and the assignment requirement to add five items will remain incomplete.

---

# Part 2: Add 5 Items to the DynamoDB Table

---

## Step 10: Create the First Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

Enter the following attributes for the first item:

| Attribute Name | Data Type | Value |
| -------------- | --------- | ----- |
| `ID` | String | `101` |
| `Name` | String | `Amit Sharma` |
| `Department` | String | `IT` |
| `Email` | String | `amit@example.com` |
| `City` | String | `Pune` |

The `ID` attribute is automatically displayed because it is the partition key of the table.

To add the remaining attributes, select:

`Add new attribute`

Then select:

`String`

Add each attribute one at a time.

After entering all attributes, select:

`Create item`

### Expected Item Structure

~~~json
{
  "ID": {
    "S": "101"
  },
  "Name": {
    "S": "Amit Sharma"
  },
  "Department": {
    "S": "IT"
  },
  "Email": {
    "S": "amit@example.com"
  },
  "City": {
    "S": "Pune"
  }
}
~~~

In the JSON representation:

| Symbol | Meaning |
| ------ | ------- |
| `S` | String data type |
| `ID` | Partition key attribute |
| `101` | Unique partition key value of the first item |

### Why is this step required?

The assignment requires five items to be stored in the DynamoDB table. This step creates the first item.

The `ID` value `101` uniquely identifies this item because the table has only a partition key and no sort key.

### Dependency

This step depends on:

* **Step 9:** The DynamoDB table must exist and the item creation page must be open.
* The table status must be `Active`.

### What happens if this step is skipped?

Only four or fewer items will exist in the table, so the assignment requirement to add five items will not be satisfied.

---

## Step 11: Create the Second Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

Enter the following attributes:

| Attribute Name | Data Type | Value |
| -------------- | --------- | ----- |
| `ID` | String | `102` |
| `Name` | String | `Priya Patil` |
| `Department` | String | `HR` |
| `Email` | String | `priya@example.com` |
| `City` | String | `Mumbai` |

After entering all attributes, select:

`Create item`

### Expected Item Structure

~~~json
{
  "ID": {
    "S": "102"
  },
  "Name": {
    "S": "Priya Patil"
  },
  "Department": {
    "S": "HR"
  },
  "Email": {
    "S": "priya@example.com"
  },
  "City": {
    "S": "Mumbai"
  }
}
~~~

### Why is this step required?

This creates the second required data item in the DynamoDB table.

The partition key value `102` is different from the first item's partition key value `101`, ensuring that both items can exist independently.

### Dependency

This step depends on:

* The `XYZ-Employee-Table` table being in the `Active` state.
* **Step 10:** The first item should already have been created.

### What happens if this step is skipped?

The table will contain fewer than the required five items.

---

## Step 12: Create the Third Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

Enter the following attributes:

| Attribute Name | Data Type | Value |
| -------------- | --------- | ----- |
| `ID` | String | `103` |
| `Name` | String | `Rahul Verma` |
| `Department` | String | `Finance` |
| `Email` | String | `rahul@example.com` |
| `City` | String | `Delhi` |

After entering all attributes, select:

`Create item`

### Expected Item Structure

~~~json
{
  "ID": {
    "S": "103"
  },
  "Name": {
    "S": "Rahul Verma"
  },
  "Department": {
    "S": "Finance"
  },
  "Email": {
    "S": "rahul@example.com"
  },
  "City": {
    "S": "Delhi"
  }
}
~~~

### Why is this step required?

This creates the third required data item and demonstrates how DynamoDB stores multiple independent items in the same table.

### Dependency

This step depends on:

* The DynamoDB table being `Active`.
* The partition key `ID` being correctly configured.

### What happens if this step is skipped?

The assignment requirement of five items will remain incomplete.

---

## Step 13: Create the Fourth Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

Enter the following attributes:

| Attribute Name | Data Type | Value |
| -------------- | --------- | ----- |
| `ID` | String | `104` |
| `Name` | String | `Sneha Joshi` |
| `Department` | String | `Marketing` |
| `Email` | String | `sneha@example.com` |
| `City` | String | `Bengaluru` |

After entering all attributes, select:

`Create item`

### Expected Item Structure

~~~json
{
  "ID": {
    "S": "104"
  },
  "Name": {
    "S": "Sneha Joshi"
  },
  "Department": {
    "S": "Marketing"
  },
  "Email": {
    "S": "sneha@example.com"
  },
  "City": {
    "S": "Bengaluru"
  }
}
~~~

### Why is this step required?

This creates the fourth item required by the assignment.

Every item uses a unique `ID`, allowing DynamoDB to uniquely identify each record.

### Dependency

This step depends on:

* The `XYZ-Employee-Table` table existing.
* The table status being `Active`.

### What happens if this step is skipped?

Only four items or fewer will be available, and the assignment will not meet its stated requirement.

---

## Step 14: Create the Fifth Item

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Create item
~~~

### Configuration

Enter the following attributes:

| Attribute Name | Data Type | Value |
| -------------- | --------- | ----- |
| `ID` | String | `105` |
| `Name` | String | `Arjun Singh` |
| `Department` | String | `Operations` |
| `Email` | String | `arjun@example.com` |
| `City` | String | `Hyderabad` |

After entering all attributes, select:

`Create item`

### Expected Item Structure

~~~json
{
  "ID": {
    "S": "105"
  },
  "Name": {
    "S": "Arjun Singh"
  },
  "Department": {
    "S": "Operations"
  },
  "Email": {
    "S": "arjun@example.com"
  },
  "City": {
    "S": "Hyderabad"
  }
}
~~~

### Why is this step required?

This creates the fifth and final item required by the assignment.

After this step, the table should contain exactly five items with unique partition key values from `101` through `105`.

### Dependency

This step depends on:

* The DynamoDB table being available.
* The table status being `Active`.

### What happens if this step is skipped?

The table will contain only four items, and task `2` of the assignment will not be complete.

---

# 4. Verification / Output Checking

## Verification Step 1: Verify That All 5 Items Exist

### Action

Open the DynamoDB table item explorer and run a scan to display all stored items.

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Explore table items
→ Scan or Run
~~~

Depending on the current AWS Console interface, the button may be displayed as:

`Run`

### Expected Output

The table should display the following five items:

| ID | Name | Department | Email | City |
| -- | ---- | ---------- | ----- | ---- |
| `101` | `Amit Sharma` | `IT` | `amit@example.com` | `Pune` |
| `102` | `Priya Patil` | `HR` | `priya@example.com` | `Mumbai` |
| `103` | `Rahul Verma` | `Finance` | `rahul@example.com` | `Delhi` |
| `104` | `Sneha Joshi` | `Marketing` | `sneha@example.com` | `Bengaluru` |
| `105` | `Arjun Singh` | `Operations` | `arjun@example.com` | `Hyderabad` |

You should observe an item count of:

~~~text
5 items
~~~

### Why does this confirm success?

This confirms that:

* The DynamoDB table was created successfully.
* The partition key `ID` is functioning correctly.
* All five required items were successfully written to the table.
* Each item can be retrieved and displayed from DynamoDB.

---

## Verification Step 2: Verify the Partition Key

### Action

Open the table overview and inspect the primary key information.

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Overview
→ General information
~~~

### Expected Output

You should observe:

~~~text
Partition key: ID (String)
~~~

There should be no sort key configured.

### Why does this confirm success?

The assignment specifically requires the partition key to be named `ID`.

Seeing `ID (String)` in the table configuration proves that the table was created with the required key schema.

---

## Verification Step 3: Verify an Individual Item Can Be Retrieved

### Action

From the item explorer, locate and open the item whose partition key is:

`ID = 103`

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ XYZ-Employee-Table
→ Explore table items
→ Locate ID 103
→ Open the item
~~~

### Expected Output

The item should contain:

~~~json
{
  "ID": "103",
  "Name": "Rahul Verma",
  "Department": "Finance",
  "Email": "rahul@example.com",
  "City": "Delhi"
}
~~~

### Why does this confirm success?

The original problem statement requires a database service that can store data and retrieve it when required.

Successfully viewing the item with `ID = 103` demonstrates that DynamoDB can retrieve previously stored data.

---

## Step 15: Open the DynamoDB Backup Section

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Backups
~~~

Depending on the current AWS Console interface, backup management may also be accessible through:

~~~text
AWS Management Console
→ DynamoDB
→ Backups
~~~

### Configuration

No backup is created yet in this step.

Locate the **On-demand backups** section.

### Why is this step required?

The assignment explicitly requires taking a backup before deleting the DynamoDB table.

An on-demand backup creates a backup of the table data that can later be used to restore the table.

### Dependency

This step depends on:

* The DynamoDB table existing.
* All five required items having been successfully added and verified.

### What happens if this step is skipped?

You will not complete the backup requirement of the assignment.

Deleting the table without first creating the required backup would also remove the active table and its data without satisfying task `3`.

---

## Step 16: Create an On-Demand Backup

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Backups
→ Create on-demand backup
~~~

### Configuration

Configure the backup as follows:

| Setting | Value |
| ------- | ----- |
| Source table | `XYZ-Employee-Table` |
| Backup name | `XYZ-Employee-Table-Backup` |

Select:

`Create backup`

### Expected Initial Status

The backup may initially show:

~~~text
Status: Creating
~~~

Wait until the status changes to:

~~~text
Status: Available
~~~

Do not delete the DynamoDB table until the backup reaches the `Available` state.

### Why is this step required?

The backup preserves a restorable copy of the DynamoDB table data.

This directly satisfies the assignment requirement to take a backup before deleting the table.

### Dependency

This step depends on:

* **Step 14:** All five items must be created.
* **Verification Step 1:** All five items should be verified.
* **Step 15:** The backup section must be accessible.

### What happens if this step is skipped?

The assignment requirement to take a backup will not be satisfied.

If the table is deleted without a completed backup, the active table and its items will be removed without having the assignment-required backup available.

---

## Step 17: Wait Until the Backup Becomes Available

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Backups
→ On-demand backups
~~~

### Action

Refresh the page if necessary and wait until the backup displays:

~~~text
Backup name: XYZ-Employee-Table-Backup
Status: Available
~~~

### Why is this step required?

A backup in the `Creating` state is still being generated.

The `Available` status confirms that the backup process has completed successfully and that the backup can be used for restoration if required.

### Dependency

This step depends on:

* **Step 16:** The on-demand backup must have been created.

### What happens if this step is skipped?

You may delete the source table before confirming that the backup was successfully completed.

This creates unnecessary risk and makes verification of the assignment requirement incomplete.

---

## Verification Step 4: Verify the DynamoDB Backup

### Action

Inspect the on-demand backup list.

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Backups
→ On-demand backups
~~~

### Expected Output

You should observe information similar to:

~~~text
Backup name: XYZ-Employee-Table-Backup
Source table: XYZ-Employee-Table
Status: Available
~~~

### Why does this confirm success?

The `Available` status confirms that DynamoDB successfully completed the on-demand backup.

At this point:

* The table exists.
* The five items exist.
* The backup exists.
* The backup is available.

The table can now be deleted as required by the assignment.

---

## Step 18: Delete the DynamoDB Table

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
→ Select XYZ-Employee-Table
→ Delete
~~~

Depending on the current AWS Console interface, the delete option may appear under:

~~~text
Actions
→ Delete table
~~~

### Action

Select:

`Delete`

AWS will display a confirmation dialog.

Confirm that the selected table is:

`XYZ-Employee-Table`

If AWS asks you to type the table name for confirmation, enter:

`XYZ-Employee-Table`

Then confirm the deletion.

### Expected Status

The table may temporarily display:

~~~text
Status: Deleting
~~~

After deletion completes, the table should disappear from the DynamoDB tables list.

### Why is this step required?

The assignment explicitly requires:

1. Taking a backup.
2. Deleting the table.

This step completes the table deletion requirement.

### Dependency

This step depends on:

* **Step 16:** The on-demand backup must have been created.
* **Step 17:** The backup status must be `Available`.
* **Verification Step 4:** The backup should be verified before deleting the source table.

### What happens if this step is skipped?

Task `3` of the assignment remains incomplete because the table has not been deleted.

The table may also continue consuming billable resources if usage exceeds applicable Free Tier limits.

---

## Verification Step 5: Verify That the DynamoDB Table Has Been Deleted

### Action

Return to the DynamoDB tables list.

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Tables
~~~

### Expected Output

The following table should no longer appear:

~~~text
XYZ-Employee-Table
~~~

You may see:

~~~text
No tables found
~~~

if no other DynamoDB tables exist in the selected AWS Region.

### Why does this confirm success?

The absence of `XYZ-Employee-Table` confirms that the source DynamoDB table was successfully deleted.

This verifies the final implementation requirement of the assignment.

---

# 5. Cleanup / Cost Optimization

At this stage, the DynamoDB table has already been deleted as part of the assignment.

However, the on-demand backup may still remain in the AWS account and can incur backup storage charges.

The remaining backup should therefore be deleted after all required verification and evidence collection are complete.

The cleanup order is:

~~~text
Verify Backup Exists
    |
    v
Verify Source Table Is Deleted
    |
    v
Delete On-Demand Backup
    |
    v
Verify Backup Deletion
~~~

The order is important because the backup should not be deleted until you have completed the assignment and captured any screenshots or evidence required for submission.

---

## Cleanup Step 1: Open the DynamoDB Backups Page

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Backups
~~~

### Action

Locate the backup:

`XYZ-Employee-Table-Backup`

### Why is this required?

The source DynamoDB table has already been deleted, but an on-demand backup can continue to exist independently.

You must locate the remaining backup before deleting it.

### What happens if this resource is not deleted?

The backup remains stored in AWS and may continue to incur DynamoDB backup storage charges.

---

## Cleanup Step 2: Delete the On-Demand Backup

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Backups
→ Select XYZ-Employee-Table-Backup
→ Delete
~~~

### Action

Select the backup:

`XYZ-Employee-Table-Backup`

Then select:

`Delete`

Confirm the deletion when prompted.

### Why is this required?

DynamoDB on-demand backups are stored independently from the source table.

Deleting the original table does not necessarily remove separately created on-demand backups.

Removing the backup after assignment verification helps minimize AWS charges.

### What happens if this resource is not deleted?

The backup may remain in the AWS account and continue consuming billable backup storage.

Even though this assignment contains only five small items and the resulting cost is likely extremely small, unused resources should still be removed as an AWS cost-optimization best practice.

---

## Cleanup Step 3: Verify the Backup Has Been Deleted

### Navigation

~~~text
AWS Management Console
→ DynamoDB
→ Backups
~~~

### Action

Confirm that the following backup no longer appears:

`XYZ-Employee-Table-Backup`

### Expected Output

The backup list should not contain:

~~~text
XYZ-Employee-Table-Backup
~~~

If no other backups exist, the console may display an empty backup list.

### Why is this required?

This confirms that all resources created specifically for this assignment have been removed.

### What happens if this resource is not deleted?

The backup may continue to exist and potentially generate storage charges.

---

## Cleanup Step 4: Final Resource Verification

### Navigation

Verify both locations:

~~~text
AWS Management Console
→ DynamoDB
→ Tables
~~~

and:

~~~text
AWS Management Console
→ DynamoDB
→ Backups
~~~

### Action

Confirm that:

| Resource | Expected Final State |
| -------- | -------------------- |
| `XYZ-Employee-Table` | Deleted |
| `XYZ-Employee-Table-Backup` | Deleted |

### Expected Output

~~~text
DynamoDB Table: Deleted
On-Demand Backup: Deleted
Assignment Resources Remaining: None
~~~

### Why is this required?

This final verification confirms that every AWS resource created specifically for this assignment has been removed after successful implementation and verification.

### What happens if this step is skipped?

You may accidentally leave resources running or stored in your AWS account, which can lead to unnecessary AWS credit consumption or charges.

---


