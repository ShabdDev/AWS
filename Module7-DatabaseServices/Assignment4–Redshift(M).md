
# Module 7: Assignment 4 - Amazon Redshift Data Warehouse Provisioning and Querying

## Problem Statement
XYZ Corporation requires an enterprise-scale, high-performance columnar data warehousing solution to aggregate large volumes of transactional data and perform complex business intelligence queries. Implement an Amazon Redshift cluster, utilize the native integrated Query Editor console interface to construct tables, load structured sample records, and execute analytical data queries.

---

## Tasks To Be Performed
1. Create an Amazon Redshift data warehouse cluster.
2. Utilize the Amazon Redshift Query Editor to:
   * **a.** Load sample data records into a structured table.
   * **b.** Execute queries against the populated dataset.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create an Amazon Redshift Cluster
1. Log in to the **AWS Management Console** and search for **Amazon Redshift** to launch its service management dashboard.
2. In the top right corner or central panel, click on the **Create cluster** button.
3. Configure the cluster deployment metadata parameters:
   * **Cluster identifier:** `xyz-corporate-redshift-warehouse`
   * **What are you planning to use this cluster for?:** Select **Dev/Test** (Choose *Dev/Test* for lower-spec configurations to save resource billing).
4. **Node configuration:**
   * **Node type:** Select `dc2.large` *(CRITICAL: This is the smallest, most cost-effective compute node available for dense compute tracking workloads).*
   * **Nodes:** Select **1** (Single Node configuration to save resource billing).
5. **Database configurations:**
   * **Admin user name:** `awsuser`
   * **Admin user password:** `XYZRedshiftPass2026` (Enforce a strong password adhering to safety constraints).
6. **Additional Configurations & Network Permissions:**
   * Ensure Default VPC is selected.
   * Under *Network and security*, ensure **Publicly accessible** is turned **Enabled**.
7. Click **Create cluster**. *(Amazon Redshift configurations take approximately 5-10 minutes to initialize and deploy hardware nodes).*

---

### Step 2: Access Redshift Query Editor v2 and Create a Table
1. Once the cluster status shows a green **Available** health sign, look at the left-hand navigation sidebar menu panel and click **Query editor v2**.
2. Expand your cluster identity tree on the left panel (`xyz-corporate-redshift-warehouse`) and connect to the default database (named `dev`) using your admin login credentials (`awsuser` / `XYZRedshiftPass2026`).
3. Open a new SQL isolation tab workspace sheet and create a corporate schema table layout by executing the following query:
   ```sql
   CREATE TABLE xyz_sales_analytics (
       sale_id INT,
       product_name VARCHAR(50),
       quantity_sold INT,
       revenue DECIMAL(10,2),
       sale_date DATE
   );
    ```

4. Click **Run**. Verify the command completes successfully.

---

### Step 3: Load Data and Query the Records (Task 2a & 2b Requirements)

1. **a. Load Data:** Use standard explicit inline `INSERT` syntax commands inside the Query Editor tab window layout to populate rows:
```sql
INSERT INTO xyz_sales_analytics VALUES 
(1, 'Cloud Storage Gateway', 5, 2500.00, '2026-07-01'),
(2, 'Compute Optimizer Engine', 12, 14400.00, '2026-07-02'),
(3, 'Database Aurora Proxy', 3, 4500.00, '2026-07-03'),
(4, 'Serverless Lambda Framework', 50, 5000.00, '2026-07-04'),
(5, 'Route53 Edge Controller', 8, 1600.00, '2026-07-05');

```


2. Click **Run** to ingest all 5 storage item rows into the columnar layout blocks.
3. **b. Query Data:** Open a fresh statement line block execution window pane to pull analytics reports from the dataset:
```sql
-- Fetch all sales records where total revenue exceeds $2,000
SELECT product_name, revenue, sale_date 
FROM xyz_sales_analytics 
WHERE revenue > 2000.00 
ORDER BY revenue DESC;

```


4. Click **Run**. The query data grid panel layout directly underneath will display the returned transactional rows instantly.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent severe credit drainage due to active Redshift compute cluster tracking running constantly, delete the resources immediately using the following workflow:

### 1. Delete the Redshift Cluster Endpoint

1. Navigate back to the main **Amazon Redshift > Clusters** control directory list window.
2. Select the checkbox next to `xyz-corporate-redshift-warehouse`.
3. Click the **Cluster** actions dropdown utility button at the top and select **Delete**.
4. A deletion confirmation dialog prompt module framework will pop up.
5. **De-select / Uncheck** the box option asking to *Create final snapshot?* (Choosing no enforces instant hardware provisioning teardown).
6. Type the explicit cluster text verification string statement name `delete` inside the confirmation layout area.
7. Click **Delete cluster**. Monitor the status page until the row fully clears out from the service layout panel.
