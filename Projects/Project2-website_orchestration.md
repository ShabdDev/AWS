# Project 2: High-Availability Website Orchestration using Elastic Beanstalk and External RDS

## Problem Statement
The objective is to orchestrate a high-availability PHP website with rapid deployment capabilities. 
By decoupling the database from the application lifecycle using an external Amazon RDS instance, 
the architecture gains flexibility, enabling Blue/Green deployment strategies and independent scaling, 
ensuring the application remains resilient and performant during traffic spikes.

Industry: Internet related

Topics:
In this AWS project, you have to deploy a high-availability PHP application with
an external Amazon RDS database to Elastic Beanstalk. Running a DB instance
external to Elastic Beanstalk decouples the database from the lifecycle of your
environment. This lets you connect to the same database from multiple
environments, swap one database for another, or perform a blue/green
deployment without affecting your database.

Highlights:
Launch a DB instance in Amazon RDS
Create an Elastic Beanstalk Environment
Configure Security Groups and scaling

---

## Steps To Solve

### Step 1: Provision External Amazon RDS Database
1. Go to **RDS Console** > **Create database**.
2. **Settings:** Select **MySQL** > **Free Tier**.
3. **Configuration:**
   * **DB Instance Identifier:** `website-db`
   * **Master Username:** `dbuser`
   * **Master Password:** `P@ssword123`
4. **Networking:** Select the **Default VPC**. Ensure **Public Access** is set to `Yes` (for initial connectivity testing). Click **Create**.
5. Save the **RDS Endpoint** for the application configuration.

### Step 2: Configure the PHP Application
1. Create a `config.php` file on your local machine to connect to the external DB:
   ```php
   <?php
   $host = 'your-rds-endpoint.amazonaws.com';
   $db   = 'website_db';
   $user = 'dbuser';
   $pass = 'P@ssword123';
   $conn = new mysqli($host, $user, $pass, $db);
   if ($conn->connect_error) { die("Connection failed: " . $conn->connect_error); }
   ?>

    ```

2. Compress your application code (including `index.php` and `config.php`) into a `.zip` file.

### Step 3: Deploy via Elastic Beanstalk

1. Go to **Elastic Beanstalk Console** > **Create application**.
2. **Application Name:** `HighAvailabilityWeb`
3. **Platform:** Select **PHP**.
4. **Application Code:** Upload your `.zip` file created in Step 2.
5. **Configuration:**
* **Instance Type:** `t2.micro` (or `t3.micro`).
* **Scaling:** Enable Auto Scaling (Min: 2, Max: 4) to ensure high availability.


6. **Security Groups:** * Ensure the **Elastic Beanstalk Security Group** has an outbound rule on **Port 3306** to communicate with the RDS instance.
* Ensure the **RDS Security Group** has an inbound rule for **Port 3306** specifically from the Elastic Beanstalk Security Group ID.

---

## Critical Cleanup Process (Cost Management)

हे रिसोर्सेस हाय-कॉस्ट श्रेणीत येतात, त्यामुळे प्रोजेक्ट पूर्ण झाल्यावर हे स्टेप्स नक्की फॉलो करा:

1. **Delete Elastic Beanstalk Environment:** * Elastic Beanstalk Console मध्ये जा.
* ऍप्लिकेशन निवडा आणि **Actions > Terminate Environment** करा. (हे सर्व EC2 Instances आणि Load Balancer आपोआप डिलीट करेल).

2. **Delete RDS Instance:** * RDS Console मध्ये जा.
* `website-db` निवडा आणि **Actions > Delete** करा. (Snapshot नको असल्यास 'No' निवडा).

3. **Cleanup Network:** * Security Groups मधील पोर्ट्स डिलीट करा.
