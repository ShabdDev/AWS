# Module 7: Assignment 2 - Amazon Aurora Cluster Deployment with High Availability Read Replicas

## Problem Statement
XYZ Corporation requires an enterprise-grade, highly available relational database infrastructure to support high-throughput applications. Implement an Amazon Aurora database cluster, provision a primary Read/Write instance, and deploy two synchronous/asynchronous Read Replicas distributed across distinct Availability Zones (AZs) to scale read operations and maximize regional infrastructure fault tolerance.

---

## Tasks To Be Performed
1. Create an Amazon Aurora Engine-based RDS Database cluster.
2. Create 2 Read Replicas in different Availability Zones for better infrastructure availability.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Initialize the Amazon Aurora Cluster
1. Log in to the **AWS Management Console** and navigate to the **RDS Dashboard**.
2. Click on the **Databases** tab in the left sidebar and select **Create database**.
3. Configure the engine and tier settings:
   * **Choose a database creation method:** `Standard create`
   * **Engine options:** `Amazon Aurora`
   * **Edition:** `Amazon Aurora MySQL-Compatible Edition` (or PostgreSQL-Compatible).
   * **Templates:** Choose **Dev/Test** *(Note: Aurora does not offer a Free Tier template. Selecting Dev/Test allows configuring low-spec burstable instances like db.t3.small or db.t4g.medium to save credits).*
4. **Settings:**
   * **DB cluster identifier:** `xyz-aurora-cluster`
   * **Master username:** `admin`
   * **Master password:** `XYZAuroraPass2026`

---

### Step 2: Configure Compute, Connectivity, and Multi-AZ Primary Node
1. **Instance configuration:** Select **Burstable classes** and choose the smallest available instance (e.g., `db.t3.small` or `db.t4g.medium`).
2. **Availability and durability:** Select **Create an Aurora Replica/Reader node in a different AZ** *(This instantly schedules the first replica instance during cluster boot).*
3. **Connectivity:**
   * **VPC:** Select your Default VPC.
   * **VPC security group:** Select *Create new* and name it `XYZ-Aurora-SG`.
4. Click **Create database**. 
5. *(Aurora takes about 7-10 minutes to fully provision the cluster topology).* Once initialized, you will see a cluster layout containing:
   * 1 Instance designated as **Writer instance** (Primary node).
   * 1 Instance designated as **Reader instance** (First Read Replica).

---

### Step 3: Deploy the Second Read Replica in a Distinct Availability Zone
To fulfill the explicit requirement of having **2 Read Replicas** distributed across different zones:
1. Inside the RDS Databases window, select your cluster tier row element (`xyz-aurora-cluster`).
2. Click the **Actions** dropdown menu at the top-right and select **Add reader**.
3. Configure the second replica parameters:
   * **DB instance identifier:** `xyz-aurora-reader-node2`
   * **DB instance class:** Keep it matching (`db.t3.small` or `db.t4g.medium`).
   * **Availability Zone:** Select a specific AZ that is **different** from the primary writer node and first reader node (e.g., if the writer is in `us-east-1a` and reader 1 is in `us-east-1b`, force reader 2 into `us-east-1c`).
4. Click **Add reader**.
5. **Verification:** Monitor the cluster workspace until all 3 nodes (1 Writer, 2 Readers) register a green status reading **Available**.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Because Amazon Aurora billing calculates compute capacity per engine hour multiplied by 3 nodes alongside global storage quotas, you must delete the entire cluster immediately after completing your verification to avoid credit exhaustion:

### 1. Delete Reader Instances (Read Replicas) First
*AWS prevents cluster deletion until all individual multi-node replica instances are wiped manually.*
1. Open the **RDS > Databases** dashboard.
2. Select the second reader instance (`xyz-aurora-reader-node2`).
3. Click **Actions** > **Delete**. Uncheck final snapshot requests, type `delete me` if prompted, and confirm.
4. Select the first reader instance (automatically generated in Step 2) under the cluster dropdown hierarchy.
5. Click **Actions** > **Delete** and confirm. Wait until both reader nodes vanish completely from the list.

### 2. Delete the Primary Writer Instance and Cluster Layer
1. Once only the Writer instance remains, select the primary **Writer instance** row.
2. Click **Actions** > **Delete**.
3. A module prompt will explain that deleting the final instance will wipe the entire Aurora cluster. 
4. **De-select** the box for *Create final snapshot?*.
5. Check the acknowledgment box stating you accept permanent data loss.
6. Type `delete me` into the safety verification prompt field and click **Delete cluster**. Verify the status transitions to *Deleting*.
