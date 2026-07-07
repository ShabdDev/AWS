# Project 1: Deploying a Scalable Multi-Tier Website using AWS EC2 and RDS

## Description
Description:
Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing
capacity in the Amazon Web Services (AWS) cloud. Using Amazon EC2
eliminates your need to invest in hardware up front so you can develop and
deploy applications faster. You can use Amazon EC2 to launch as many or as
few virtual servers as you need, configure security and networking, and manage
storage. Amazon EC2 enables you to scale up or down to handle changes in
requirements or spikes in popularity, reducing your need to forecast traffic

## Problem Statement
Company ABC aims to migrate its existing PHP-based web product onto the AWS cloud infrastructure. 
The architecture requires high availability, automatic horizontal scaling to handle traffic fluctuations, and 
a managed backend database tier. Implement a solution utilizing Auto Scaling Groups (ASG) for the web layer and 
Amazon RDS for the relational database layer.

---

## Steps To Solve

### Step 1: Provision the Relational Database (RDS)
1. **Launch RDS:** Navigate to **RDS Console** > **Create database**.
2. **Settings:** Select **MySQL** engine > Template: **Free Tier**.
3. **Configuration:**
   * **DB Instance Identifier:** `intel-db`
   * **Master Username:** `admin` (or root as per requirement)
   * **Master Password:** `intel123`
4. **Networking:** Ensure **Public Access** is set to `Yes` for testing purposes (or configure proper Security Group ingress rules). Click **Create**.
5. Once Available, note the **Endpoint URL**.

### Step 2: Configure EC2 Launch Template for Auto Scaling
1. **Launch Template:** Go to **EC2 > Launch Templates** > **Create launch template**.
   * **AMI:** Amazon Linux 2023.
   * **Instance Type:** `t2.micro`.
   * **User Data:** Paste the following script to install PHP and Apache:
     ```bash
     #!/bin/bash
     dnf update -y
     dnf install httpd php php-mysqlnd -y
     systemctl start httpd
     systemctl enable httpd
     ```
2. **Security Group:** Create a Security Group that:
   * Allows **HTTP (Port 80)** traffic from everywhere.
   * Allows **All Traffic** (Task 7 requirement).

### Step 3: Enable Auto Scaling (ASG)
1. Navigate to **EC2 > Auto Scaling Groups** > **Create Auto Scaling group**.
2. **Launch Template:** Select the template created in Step 2.
3. **Network:** Select Default VPC and subnets.
4. **Group Size:** * Desired Capacity: **2**
   * Minimum Capacity: **2**
   * Maximum Capacity: **4**
5. Click **Create**. AWS will now automatically launch 2 EC2 instances.

### Step 4: Configure Database and Connectivity
1. **Database Setup:** Connect to the RDS instance via MySQL Workbench or CLI and run:
   ```sql
   CREATE DATABASE intel;
   USE intel;
   CREATE TABLE data (id INT, name VARCHAR(50));
   ```

2. **Connectivity (Task 6):** Go to the **RDS Security Group** and add an **Inbound Rule** to allow traffic from the **EC2 Security Group ID** on **Port 3306**.
3. **Hostname Update (Task 5):** On your PHP web server (`/var/www/html/index.php`), update the connection string to point to the RDS Endpoint URL:
```php
$host = "your-rds-endpoint.aws.com";
$user = "admin";
$pass = "intel123";
$db = "intel";

```
---

## Critical Cleanup Process (Cost Management)

Since this project uses multiple compute and database instances, failure to delete them will exhaust your remaining credits.

1. **Delete Auto Scaling Group:** Navigate to **EC2 > Auto Scaling Groups**, select your group, click **Actions > Delete**. This will automatically terminate the 2 EC2 instances.
2. **Delete RDS Instance:** Navigate to **RDS > Databases**, select `intel-db`, click **Actions > Delete**. (Choose *No* for final snapshot if not required to save costs).
3. **Cleanup Network:** Delete the Launch Template and Security Groups created for this project to ensure a clean account state.
