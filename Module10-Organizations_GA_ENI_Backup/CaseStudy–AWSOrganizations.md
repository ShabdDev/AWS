# Module 10: Case Study - Advanced Infrastructure Management: FSx, Global Accelerator, and ENI Orchestration

## Problem Statement
XYZ Corporation requires a high-performance, resilient cloud infrastructure to distribute administrative workloads safely across different operational teams. To achieve this milestone, configure a centralized, shared Amazon FSx storage file system mounted across multiple Linux clusters, implement high-availability Anycast traffic routing via AWS Global Accelerator servicing active Apache web tiers, and isolate administrative access topologies by configuring a secondary dedicated Elastic Network Interface (ENI) for remote SSH triage.

---

## Tasks To Be Performed
1. Create and provision an Amazon FSx file system engineered for Linux architectures and mount it concurrently across 2 running EC2 Linux server instances.
2. Build 2 independent Linux EC2 instances and initialize live production web servers on each node.
3. Establish an AWS Global Accelerator configuration mapping both EC2 instances as active high-performance load endpoints.
4. Provision an independent Elastic Network Interface (ENI), attach it to an active EC2 instance as a secondary interface card, and utilize its assigned IP address to establish an SSH terminal connection.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Launch 2 Linux EC2 Instances with Web Servers (Task 2 Requirement)
1. Log in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. Click **Launch instances**. Configure parameters as follows:
   * **Name:** `XYZ-Web-Server-01` and `XYZ-Web-Server-02` (Set Number of instances to **2**).
   * **OS Image:** Select **Amazon Linux 2023 AMI**.
   * **Instance type:** Select **t2.micro** (or `t3.micro` based on local region credit availability).
   * **Key pair:** Choose your existing key pair profile.
   * **Network settings:** Check **Allow HTTP traffic from the internet** and **Allow SSH traffic from the internet**.
3. Expand **Advanced details** at the bottom, scroll down to the **User data** input box framework, and paste the following baseline web installation script to spin up the servers automatically:
   ```bash
   #!/bin/bash
   dnf update -y
   dnf install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>XYZ Corporation - High Availability Web Instance</h1>" > /var/www/html/index.html
   ```

4. Click **Launch instance**.

---

### Step 2: Create and Mount Amazon FSx for Linux (Task 1 Requirement)

1. Search and open the **Amazon FSx Console**. Click **Create file system**.
2. Select **Amazon FSx for Lustre** (or OpenZFS) engineered for high-performance Linux throughput operations. Click **Next**.
3. **Configure storage settings:**
* **File system name:** `XYZ-Shared-Linux-Storage`
* **Storage capacity:** Enter minimum required allocation threshold (e.g., `1200 GiB` for Lustre or minimum sandbox standard boundaries).
* **VPC:** Choose your Default VPC (Ensure it matches the identical VPC where Step 1 instances reside).


4. Click **Next**, review defaults, and click **Create file system**. *(Wait 5-10 minutes for state synchronization to complete)*.
5. **Mount to EC2 instances:**
* Once status displays a green *Available* tag, click on the file system name and locate the **Attach** instruction guidelines.
* Copy the generated command line mount snippet text string (e.g., `sudo mount -t lustre ...`).
* SSH into **both** `XYZ-Web-Server-01` and `XYZ-Web-Server-02` individually via your cloud terminal, run the copied configuration scripts, and verify a single shared data tier is established across both servers concurrently.

---

### Step 3: Provision AWS Global Accelerator Traffic Endpoints (Task 3 Requirement)

1. Search and navigate to the **AWS Global Accelerator Console**. Click **Create accelerator**.
2. **Settings:** Name it `XYZ-Global-Web-Accelerator`, select type **Standard**, and click **Next**.
3. **Listeners:** Define Port as **`80`**, Protocol as **TCP**, and click **Next**.
4. **Endpoint Groups:** Choose your local production territory region (e.g., `us-east-1`) and click **Next**.
5. **Endpoints Matrix Registration:**
* Click **Add endpoint** twice. Select **EC2 instance** as type for both slots.
* Map resource 1 to `XYZ-Web-Server-01` and resource 2 to `XYZ-Web-Server-02`.


6. Click **Create accelerator**. Copy the generated global public Anycast IP strings once the status switches to **Deployed**. Paste the IP into your web browser to confirm uniform connection routing loops are fully live.

---

### Step 4: Provision and Attach Secondary Network Interface (ENI) for SSH (Task 4 Requirement)

1. Open the **EC2 Console** window and select **Network Interfaces** under the *Network & Security* left sidebar category.
2. Click **Create network interface**. Configure details:
* **Description:** `XYZ-Admin-SSH-Interface`
* **Subnet:** Select the exact identical Availability Zone subnet hosting your `XYZ-Web-Server-01` instance.
* **Security groups:** Check your baseline administrative group permitting Port 22 inbound access.


3. Click **Create network interface**. Copy the newly bound private/public interface network IP address.
4. Select your active `XYZ-Admin-SSH-Interface` row checkbox element, click **Actions** > **Attach**, choose instance target id matching `XYZ-Web-Server-01`, and confirm attachment execution.
5. **Validation Testing:** Open your local client command prompt or PuTTY environment terminal. Execute a direct SSH shell verification targeting the secondary interface configuration parameters:
```bash
ssh -i "your-key-pair.pem" ec2-user@<Your-Secondary-ENI-Public-IP>

```

*Result: You will successfully gain administrative console shell terminal entrance through the distinct interface card pathway, resolving task objectives cleanly.*

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Wipe out these multi-tier enterprise configurations systematically upon presentation review to secure account tokens from substantial running maintenance balances:

### 1. Disable and Delete the Anycast Global Accelerator

1. Navigate to the **AWS Global Accelerator Console** page.
2. Open `XYZ-Global-Web-Accelerator`, click **Edit**, disable the operations active check switch toggle, and commit updates.
3. Wait 1 minute for data propagation loops to complete, select the accelerator row, and execute **Delete**.

### 2. Disassemble and Delete the High-Cost Amazon FSx File System

1. Open the **Amazon FSx Console** and click on your item row listing `XYZ-Shared-Linux-Storage`.
2. Locate the top interactive controls header and click **Actions** > **Delete file system**.
3. Uncheck snapshot archival creation flags to skip auxiliary billing paths, type the explicit resource reference ID confirmation string pattern, and click **Delete**.

### 3. Detach and Erase the Custom Network Interface (ENI)

1. Navigate to **EC2 > Network Interfaces**.
2. Select `XYZ-Admin-SSH-Interface`. Click **Actions** > **Detach**. *(Wait roughly 30 seconds for state release processes to complete)*.
3. Select the row again, click **Actions** > **Delete**, and finalize its absolute removal from the cloud infrastructure platform.

### 4. Terminate the Running Cloud Compute Instances

1. Go to **EC2 > Instances**.
2. Check the box groups next to both `XYZ-Web-Server-01` and `XYZ-Web-Server-02`.
3. Locate upper tier control parameters and choose **Instance state** > **Terminate instance** > Click confirm inside pop-up dialog window spaces to clear background server processes entirely.
