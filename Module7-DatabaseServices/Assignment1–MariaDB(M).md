# Module 7: Assignment 1 - MariaDB RDS Deployment and Multi-Client Connectivity

## Problem Statement
XYZ Corporation requires a robust, managed relational SQL database service to securely retain and retrieve structured application transactional records. Implement an Amazon RDS database cluster leveraging the MariaDB engine, and configure multi-channel secure pathways to establish database connectivity via a local Windows SQL Client (HeidiSQL/DBeaver) and a cloud-native Linux-based EC2 endpoint.

---

## Tasks To Be Performed
1. Create a MariaDB Engine-based RDS Database instance.
2. Connect to the Database using two independent workflows:
   * **a.** SQL Client for Windows (e.g., HeidiSQL or DBeaver).
   * **b.** Linux-based EC2 Instance via Command Line interface.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create a MariaDB RDS Instance
1. Log in to the **AWS Management Console** and navigate to the **RDS Dashboard**.
2. Click on **Create database**.
3. Configure the Database parameters:
   * **Choose a database creation method:** `Standard create`
   * **Engine options:** `MariaDB`
   * **Templates:** Choose **Free tier** *(CRITICAL: Enforces low-cost t3.micro/t4g.micro single-node deployments).*
4. **Settings:**
   * **DB instance identifier:** `xyz-corporate-mariadb`
   * **Master username:** `admin`
   * **Master password:** `XYZCorpPass123` (Set a strong compliance password).
5. **Connectivity:**
   * **Compute resource:** `Don't connect to an EC2 compute resource` (We will handle this manually via Security Groups).
   * **VPC:** Select your Default VPC.
   * **Public access:** Select **Yes** *(Note: Required for Task 2a to allow your local Windows SQL Client to reach the endpoint. Ensure the security group handles restrictions).*
   * **VPC security group:** Choose *Create new* and name it `XYZ-MariaDB-SG`.
   * **Database port:** Keep default `3306`.
6. Click **Create database**. *(It will take 5-7 minutes for the status to switch from 'Creating' to 'Available').* Once available, copy the **Endpoint string** (e.g., `xyz-corporate-mariadb.xxxx.us-east-1.rds.amazonaws.com`).

---

### Step 2: Configure Inbound Security Rules
Before attempting connectivity, you must modify `XYZ-MariaDB-SG` to accept incoming SQL requests:
1. Go to **EC2 Console** > **Security Groups** > Select `XYZ-MariaDB-SG`.
2. Click **Edit inbound rules**.
3. Add two rules:
   * **Rule 1 (For Windows Client):** Type: `MYSQL/Aurora` (Port 3306) | Source: `My IP` (Your local internet IP).
   * **Rule 2 (For Linux EC2):** Type: `MYSQL/Aurora` (Port 3306) | Source: `Anywhere-IPv4` (`0.0.0.0/0`) or map it directly to your EC2's security group ID.
4. Click **Save rules**.

---

### Step 3: Connect via Way A (SQL Client for Windows)
1. Download and install a standard SQL client like **HeidiSQL** or **DBeaver** on your local machine.
2. Open the client tool and click **New Connection**.
3. Fill in the parameters using your RDS metadata:
   * **Network Type / Driver:** `MariaDB` or `MySQL`
   * **Hostname / IP (Endpoint):** Paste your RDS **Endpoint string**.
   * **User:** `admin`
   * **Password:** `XYZCorpPass123`
   * **Port:** `3306`
4. Click **Open** or **Connect**. You will successfully log into the managed engine interface shell. Run query: `CREATE DATABASE corporate_db;` to verify write parameters.

---

### Step 4: Connect via Way B (Linux-based EC2 Instance)
1. Launch a standard Linux instance (`t2.micro`, Ubuntu) named `XYZ-DB-Client-Host` in the same Default VPC. Make sure it has a public IP.
2. Connect to the EC2 instance via **EC2 Instance Connect**.
3. Install the MariaDB command-line client utilities tool inside the Linux instance session:
   ```bash
   sudo apt update -y
   sudo apt install mariadb-client -y
   ```

4. Execute the remote connection string using your endpoint pointer:
```bash
mysql -h <your_rds_endpoint_string> -P 3306 -u admin -p

```


5. Type your database password (`XYZCorpPass123`) when prompted and press Enter.
6. **Result:** You will enter the MariaDB terminal interactive shell prompt (`MariaDB [(none)]>`). Run `SHOW DATABASES;` to view the structure.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Managed database nodes accumulate running storage costs continuously if left active. Wipe your temporary infrastructure in this exact sequence immediately after validation:

### 1. Delete the Amazon RDS Database

1. Open the **RDS Dashboard** > Click **Databases** from the left panel sidebar.
2. Select the checkbox container for `xyz-corporate-mariadb`.
3. Click the **Actions** dropdown control menu at the top and select **Delete**.
4. **De-select** the checkbox for *Create final snapshot?* (Enforces fast cleanup).
5. Check the acknowledgment box confirming you understand automated backups are deleted.
6. Type `delete me` into the safety verification text input array layout and click **Delete**.

### 2. Terminate the Database Client EC2 Host

1. Go to **EC2 Dashboard** > **Instances**.
2. Select `XYZ-DB-Client-Host`.
3. Click **Instance state** > **Terminate instance** and confirm permanent removal.

### 3. Clear Security Group Templates

1. Navigate to **Security Groups** from the network settings side panel interface.
2. Check the box next to `XYZ-MariaDB-SG`.
3. Click **Actions** > **Delete security groups** and confirm removal once the backend RDS instance is completely offline.
