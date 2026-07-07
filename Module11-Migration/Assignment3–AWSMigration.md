# Module 11: Assignment 3 - Heterogeneous Database Migration via AWS Database Migration Service (DMS)

## Problem Statement
XYZ Corporation is executing a database modernization strategy to transition operational workloads into managed engines. To improve performance capabilities, implement a cross-engine heterogeneous database migration. Provision a primary source Amazon RDS MySQL instance, populate it with mock corporate data rows, establish an alternate Amazon RDS PostgreSQL destination cluster, and utilize AWS Database Migration Service (DMS) replication tasks to execute a seamless, synchronized tablespace migration.

---

## Tasks To Be Performed
1. Provision an Amazon RDS MySQL engine instance, log into the shell terminal, and insert transactional database records (Source Database).
2. Provision an Amazon RDS PostgreSQL database instance to serve as the cloud modernization target (Target Database).
3. Deploy an AWS DMS Replication Instance and establish secure source and target endpoint verification channels.
4. Launch an active Full-Load Database Migration Task to clone and transfer structural datasets across the heterogeneous environments.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision and Populate the Source RDS MySQL Database
1. Log in to your **AWS Management Console** and navigate to the **Amazon RDS Dashboard**.
2. Click **Create database**. Configure parameters as follows:
   * **Choose a database creation method:** Standard create
   * **Engine options:** MySQL
   * **Templates:** Free Tier (or Dev/Test depending on local staging capabilities)
   * **DB instance identifier:** `xyz-source-mysql`
   * **Master username:** `dbadmin`
   * **Master password:** `XYZSecurePass2026`
   * **Public access:** Yes *(Required for quick testing via local terminal setups)*
3. Click **Create database**. Wait roughly 5 minutes for status to change to *Available*, then copy the generated **Endpoint** string.
4. **Log in and Populate Data (Task 1 Validation):** Connect via your database client (e.g., MySQL Workbench or MySQL CLI) and execute the following queries:
   ```sql
   CREATE DATABASE xyz_corp;
   USE xyz_corp;
   
   CREATE TABLE employees (
       emp_id INT PRIMARY KEY,
       first_name VARCHAR(50),
       department VARCHAR(50)
   );
   
   INSERT INTO employees VALUES (1, 'Rahul', 'Cloud Operations');
   INSERT INTO employees VALUES (2, 'Priya', 'DevSecOps Core');
   
   SELECT * FROM employees;
   ```

---

### Step 2: Provision the Destination RDS PostgreSQL Database

1. Return to the **Amazon RDS Dashboard** and click **Create database**.
2. Configure parameters for the heterogeneous target profile:
* **Engine options:** PostgreSQL
* **Templates:** Free Tier
* **DB instance identifier:** `xyz-target-postgres`
* **Master username:** `postgres`
* **Master password:** `XYZTargetPass2026`
* **Public access:** Yes


3. Click **Create database**. Once initialized, copy its standalone database **Endpoint** string.

---

### Step 3: Set Up AWS DMS Replication Engine and Endpoints (Task 2 Requirement)

1. Search and navigate to the **AWS Database Migration Service (DMS)** console.
2. Click **Replication instances** from the sidebar list > Click **Create replication instance**.
* **Name:** `xyz-migration-engine`
* **Instance class:** Select a cost-effective node like **`dms.t3.micro`** or `dms.t2.micro`.
* Keep networking defaults matching your RDS configurations active and click **Create**.


3. **Configure Connection Endpoints:** Select **Endpoints** from the sidebar options and click **Create endpoint**:
* **Source Endpoint configuration:**
* **Endpoint type:** Source DB engine
* **Source engine:** MySQL
* Select check box *Select RDS DB instance* and choose `xyz-source-mysql`.
* Input credentials (`dbadmin` / `XYZSecurePass2026`). Click **Create endpoint**.


* **Target Endpoint configuration:** Click Create endpoint again.
* **Endpoint type:** Target DB engine
* **Target engine:** PostgreSQL
* Select *Select RDS DB instance* and choose `xyz-target-postgres`.
* Input credentials (`postgres` / `XYZTargetPass2026`). Click **Create endpoint**.


4. Test the endpoint configuration routes against your active replication engine to ensure validation links return a green **Successful** log text message.

---

### Step 4: Launch and Validate the Migration Task

1. Navigate to **Database migration tasks** in the left vertical directory tree panel. Click **Create task**.
2. Configure task attributes:
* **Task identifier:** `xyz-mysql-to-postgres-fullload`
* **Replication instance:** `xyz-migration-engine`
* **Source database endpoint:** `xyz-source-mysql`
* **Target database endpoint:** `xyz-target-postgres`
* **Migration type:** Migrate existing data (Full load)


3. Under **Table mappings**, expand the structural selector wizard rules, set schema name string rules to search for `%` (wildcard matching all target tables spaces), and click **Create task**.
4. **Verification:** The task status flags will shift through *Creating* and *Starting* into a live **Load complete** notation flag within a few minutes. Log directly into your PostgreSQL database console framework and run queries to confirm structural dataset synchronization:
```sql
-- Run inside target PostgreSQL instance
SELECT * FROM xyz_corp.employees;

```

*Result: The console will output 'Rahul' and 'Priya' table rows matching the MySQL datasets exactly, validating absolute structural cross-engine delivery.*

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Database migration suites lock down concurrent computing networks that sustain heavy administrative pricing fees. Erase all assets immediately upon lab sign-off using this checklist order:

### 1. Wipe out Database Migration Tasks and Replication Instances

1. Inside the **AWS DMS Console**, click on **Database migration tasks**. Highlight `xyz-mysql-to-postgres-fullload`, click **Actions**, and choose **Delete**.
2. Navigate to **Endpoints**, select both your MySQL source and PostgreSQL target string configurations row blocks, and click **Delete**.
3. Navigate to **Replication instances**, click check indicators matching `xyz-migration-engine`, click actions header tools, choose **Delete**, and authorize removal permissions.

### 2. Disassemble and Delete Active Amazon RDS Engines

1. Open the **Amazon RDS Dashboard** and navigate to the **Databases** management grid view.
2. Select the row containing **`xyz-source-mysql`** > Click **Actions** > Choose **Delete**.
* **Safety Action:** Uncheck final snapshot preservation sliders, check authorization check agreements, type `delete me` inside prompt verification fields, and execute destruction.


3. Select the row containing **`xyz-target-postgres`** > Click **Actions** > Choose **Delete** > apply the identical quick-wipe verification routines to cleanly return your AWS credit ledger tracking meters to standby parameters safely.
