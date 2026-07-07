# Module 7: Case Study - High-Availability Database Architecture, Real-Time Analytics, and Caching

## Problem Statement
XYZ Corporation operates multiple branch offices nationwide and deploys a high-throughput, data-intensive application on AWS. The system architecture must ingest continuous device telemetry streams, execute complex relational joins/updates, maintain global multi-region resilience, and process analytical payloads in near real-time. Additionally, the system must resolve downstream latency overheads driven by redundant data fetches from various branch networks.

---

## Tasks To Be Performed
1. Build a highly scalable database that is highly available and accessible, with an automated backup retention period configured for exactly 2 days.
2. Build a cross-region database architecture to facilitate global distributed read operations.
3. Establish a backend architecture to offload data workloads for near real-time business intelligence analysis.
4. Architect and implement an in-memory caching mechanism to resolve data delivery latency to regional branch networks.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision a High-Availability Amazon Aurora Cluster with 2-Day Backup Retention
1. Log in to the **AWS Management Console** and navigate to the **RDS Dashboard**.
2. Click **Create database** > Choose **Standard create** > Select **Amazon Aurora** (MySQL-Compatible).
3. Under Templates, choose **Dev/Test** (Select burstable instances like `db.t3.small` to control lab costs).
4. Configure identifiers:
   * **DB cluster identifier:** `xyz-prod-aurora-cluster`
   * **Master username:** `admin`
   * **Master password:** `XYZCorpSecurePass2026`
5. **Availability and durability (Task 1 Requirement):** Select **Create an Aurora Replica/Reader node in a different AZ** (Enforces Multi-AZ failover stability).
6. Scroll down to **Additional configuration** > **Backup** section:
   * **Backup retention period:** Change the default setting dropdown to explicitly register **2 days**.
7. Click **Create database** and wait for initialization.

---

### Step 2: Scale Reads Globally using Aurora Global Databases (Task 2 Requirement)
To enable multi-region data reads across diverse branch offices:
1. Select your primary cluster identifier `xyz-prod-aurora-cluster`.
2. Click the **Actions** dropdown menu at the top-right and select **Add AWS Region**.
3. Configure the secondary global cluster footprint:
   * **Global database identifier:** `xyz-global-network-database`
   * **Secondary Region:** Select a different regional target (e.g., *US West (Oregon)* or *eu-west-1*).
   * **Secondary Read Replica Node Class:** Match the hardware specification (`db.t3.small`).
4. Click **Add region**. AWS will build an asynchronous replication pipeline linking the multi-region read nodes.

---

### Step 3: Implement Amazon Redshift for Near Real-Time Analysis (Task 3 Requirement)
To process complex analytical payloads without placing resource contention on transactional operations:
1. Navigate to the **Amazon Redshift Dashboard** > click **Create cluster**.
2. Configure attributes:
   * **Cluster identifier:** `xyz-analytics-warehouse`
   * **Node type:** `dc2.large` | **Nodes:** `1` (Single Node structure).
   * **Database credentials:** `awsuser` / `XYZRedshiftPass2026`
3. Click **Create cluster**.
4. *Integration Linkage:* Use AWS Glue or Redshift Federated Queries to directly query operational transactional records inside Aurora via your managed analytical layer.

---

### Step 4: Deploy Amazon ElastiCache to Resolve Branch Latency (Latency Solution)
To address recurring data reads causing latency overhead across offices, inject a Redis in-memory caching layer:
1. Navigate to the **Amazon ElastiCache Dashboard** > click **Redis clusters** > **Create Redis cluster**.
2. Configure settings:
   * **Cluster engine:** `Redis` (or Valkey/Memcached based on exact API preferences).
   * **Name:** `xyz-branch-read-cache`
   * **Node type:** Choose the lowest specs available (e.g., `cache.t3.micro`).
   * **Number of replicas:** `1`
3. Click **Create**.
4. *Workflow Integration:* The application will now check `xyz-branch-read-cache` first. If a data pattern hits the cache, it is delivered to branches instantly in microseconds, completely bypassing database read overhead.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Wipe this multi-tier ecosystem immediately after evaluation to protect your credits against severe enterprise hourly runtime drainage:

### 1. Delete the Amazon ElastiCache Cluster
1. Open the **ElastiCache Dashboard** > Select **Redis clusters**.
2. Check the box for `xyz-branch-read-cache`, click **Actions** > **Delete**. Uncheck final backup requests and confirm.

### 2. Teardown the Amazon Redshift Warehouse Node
1. Navigate to the **Amazon Redshift > Clusters** index.
2. Select `xyz-analytics-warehouse`, click **Actions** > **Delete**.
3. Uncheck *Create final snapshot?*, type `delete` to authorize, and click **Delete cluster**.

### 3. Remove Regional Nodes and Disassemble Aurora Global Clusters
1. Open the **RDS > Databases** window panel list view.
2. Select the **Secondary Reader instance** residing in your secondary remote region block.
3. Click **Actions** > **Delete** and confirm. Wait for deletion to clear completely.
4. Select the parent **Global Database row** entry, click **Actions**, and choose **Remove from global database**.
5. Finally, drill down into your primary region cluster dropdown:
   * Delete the local **Reader instance** first via **Actions > Delete**.
   * Once cleared, select the final **Writer instance**, click **Actions > Delete**, uncheck final snapshot confirmation parameters, type `delete me`, and click **Delete cluster**.
