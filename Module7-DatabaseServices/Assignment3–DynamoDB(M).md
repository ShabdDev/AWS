# Module 7: Assignment 3 - Amazon DynamoDB Table Management and On-Demand Backup

## Problem Statement
XYZ Corporation requires a high-performance, fully managed NoSQL database service to store and retrieve non-relational application datasets with single-digit millisecond latency. Implement an Amazon DynamoDB table utilizing a unique Partition Key, ingest five sample document data items, execute an on-demand infrastructure backup for compliance, and perform a clean teardown of the live table asset.

---

## Tasks To Be Performed
1. Create a DynamoDB table with a partition key designated as `ID`.
2. Add 5 items with distinct attribute profiles to the DynamoDB table.
3. Take an on-demand database backup and safely delete the active table.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision the DynamoDB Table
1. Log in to the **AWS Management Console** and search for **DynamoDB** to open the Amazon DynamoDB dashboard.
2. In the left navigation sidebar or main panel, click on **Tables** > **Create table**.
3. Configure the table blueprint settings:
   * **Table name:** `XYZ-Corporate-NoSQL-Table`
   * **Partition key:** Type `ID` and set its data type dropdown to **String** (or Number).
   * **Sort key:** Leave blank (Optional).
4. **Table settings:** Keep **Default settings** selected *(Note: This automatically enforces the cost-effective Provisioned capacity mode with Free Tier thresholds).*
5. Click **Create table**. *(The infrastructure deploys instantly in less than 30 seconds).*

---

### Step 2: Ingest 5 Items into the Table
1. Click directly on the name of your newly created table `XYZ-Corporate-NoSQL-Table` from the list directory.
2. Click on the **Actions** dropdown menu at the top-right and select **Explore table items**.
3. Scroll down to the *Items returned* console block and click **Create item**.
4. Construct your items with attributes. Switch to **JSON view** or use the **Form view** to add attributes:
   * **Item 1:** `ID` = `101`, Add new attribute (String): `Name` = `Alice`, `Role` = `Cloud Engineer`
   * Click **Create item**.
5. Repeat the workflow to generate 4 more distinct items:
   * **Item 2:** `ID` = `102`, `Name` = `Bob`, `Role` = `DevOps Specialist`
   * **Item 3:** `ID` = `103`, `Name` = `Charlie`, `Role` = `Security Analyst`
   * **Item 4:** `ID` = `104`, `Name` = `David`, `Role` = `Database Admin`
   * **Item 5:** `ID` = `105`, `Name` = `Emma`, `Role` = `Solutions Architect`
6. Verify that all 5 rows display successfully within your items grid console view.

---

### Step 3: Execute On-Demand Table Backup (Task 3 Requirement)
1. Inside your `XYZ-Corporate-NoSQL-Table` configuration workspace, click on the **Backups** tab.
2. In the *On-demand backups* block panel, click **Create backup**.
3. Configure backup metadata:
   * **Backup name:** `XYZ-Table-Compliance-Backup`
   * Leave retention settings as default.
4. Click **Create backup**. The console will track progress until the status turns to a green **Available** flag indication.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

DynamoDB storage is free up to 25 GB, but to keep your dashboard clean and complete the final task parameters, delete the live database asset safely:

### 1. Delete the Live Table Asset
1. Navigate back to the main **DynamoDB > Tables** structural index window.
2. Select the checkbox container next to `XYZ-Corporate-NoSQL-Table`.
3. Click on the **Delete** button positioned at the top right of the dashboard screen.
4. A confirmation safety modal will pop up. 
5. Type `delete` into the validation input field box to authorize absolute removal.
6. Click **Delete table**. *(Your backup created in Step 3 remains safe in the backup repository,
7. but the active live table tier is wiped successfully).*
