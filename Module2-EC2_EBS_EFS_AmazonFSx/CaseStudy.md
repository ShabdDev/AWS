# Module 2: Case Study: EC2 and File Systems

## Problem Statement
You are a cloud architect for a company planning to deploy a web application on AWS. The application comprises a frontend web server, a backend database, and a file storage system for user-uploaded content.

---

## Requirements

### 1. Amazon EC2
* Launch two EC2 instances (each of both Linux and Windows AMI) in different Availability Zones to ensure high availability. One will be used for the web server, and the other for the database server.
* Configure the web server instance with an appropriate instance type based on the expected traffic and the resources the web application requires.
* Secure the instances using Security Groups, allowing only the necessary traffic (e.g., HTTP/HTTPS for the web server, and specific IP/port for the database).

### 2. Amazon Elastic Block Store (EBS)
* Create two EBS volumes, one for each EC2 instance.
* Choose the appropriate volume type for each instance based on the workload.
* Attach the EBS volumes to the respective EC2 instances and mount them.

### 3. Amazon Elastic File System (EFS)
* Create an EFS file system to store user-uploaded content.
* Configure the file system to be accessible from both EC2 instances (web server and database server) for redundancy.

### 4. Amazon FSx
* Assume the application requires a Windows-based file system for certain operations. Set up an Amazon FSx for Windows File Server instance.
* Configure the FSx instance to be accessible from the compatible EC2 instance.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Launch Linux and Windows EC2 Instances
1. Log in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. Click on **Launch Instance**.
3. **Configure Linux Instance (Web Server):**
   * **Name:** `Linux-Web-Server`
   * **Application and OS Image (AMI):** Amazon Linux 2023 AMI
   * **Instance Type:** `t2.micro` (Free Tier eligible, suitable for low/medium testing traffic).
   * **Key Pair:** Create or select an existing key pair.
   * **Network Settings:** Click *Edit*. Select your default VPC and choose **Availability Zone A (AZ-a)**.
   * **Security Group:** Select *Create security group* and name it `Web-SG`. 
     * Add Inbound Rule 1: **HTTP (Port 80)** | Source: `0.0.0.0/0` (Anywhere)
     * Add Inbound Rule 2: **HTTPS (Port 443)** | Source: `0.0.0.0/0` (Anywhere)
     * Add Inbound Rule 3: **SSH (Port 22)** | Source: `My IP`
4. **Configure Windows Instance (Database/App Server):**
   * Click **Launch Instance** again.
   * **Name:** `Windows-DB-Server`
   * **Application and OS Image (AMI):** Microsoft Windows Server 2022 Base
   * **Instance Type:** `t2.micro`
   * **Network Settings:** Click *Edit*. Select **Availability Zone B (AZ-b)** to ensure High Availability across different AZs.
   * **Security Group:** Select *Create security group* and name it `DB-SG`.
     * Add Inbound Rule 1: **RDP (Port 3389)** | Source: `My IP`
     * Add Inbound Rule 2: **Custom TCP (Database Port e.g., 1433 or 3306)** | Source: Custom -> Select `Web-SG` (Allows database traffic only from the Web Server).
5. Review configurations and click **Launch Instance** for both.

### Step 2: Create and Attach EBS Volumes
1. In the left-hand menu of the EC2 Dashboard, go to **Elastic Block Store** > **Volumes**.
2. Click **Create Volume**.
   * **Volume Type:** General Purpose SSD (`gp3`).
   * **Size:** `10 GiB`.
   * **Availability Zone:** Must match your Linux instance (**AZ-a**).
   * Click **Create volume**.
3. Click **Create Volume** again for the Windows instance.
   * **Volume Type:** General Purpose SSD (`gp3`).
   * **Size:** `10 GiB`.
   * **Availability Zone:** Must match your Windows instance (**AZ-b**).
   * Click **Create volume**.
4. Select the first volume -> click **Actions** -> **Attach volume**. Select `Linux-Web-Server` from the instance list and click Attach.
5. Select the second volume -> click **Actions** -> **Attach volume**. Select `Windows-DB-Server` from the instance list and click Attach.

### Step 3: Create and Configure Amazon EFS
1. Navigate to the **EFS (Elastic File System)** console.
2. Click **Create file system**.
3. Set the name to `User-Uploads-EFS`, select your default VPC, and click **Customize**.
4. In the **Network Access** tab, ensure that mount targets are active for both AZs (**AZ-a** and **AZ-b**) where your EC2 instances are running.
5. Assign a Security Group to the mount targets that allows **NFS traffic (Port 2049)** from the `Web-SG` and `DB-SG`.
6. Click **Create** to initialize the shared file system.

### Step 4: Create Amazon FSx for Windows File Server
1. Navigate to the **FSx** console.
2. Click **Create file system** and choose **Amazon FSx for Windows File Server**. Click **Next**.
3. Configure the file system settings:
   * **File System Name:** `Windows-Shared-FSx`
   * **Deployment Type:** Single-AZ
   * **Storage Type:** SSD | **Storage Capacity:** `32 GiB` (minimum requirement).
   * **Network & Security:** Select your default VPC and choose the same Availability Zone as your Windows instance (**AZ-b**).
   * **Windows Authentication:** Choose **AWS Managed Active Directory** (Create a simple directory if you don't have one, as this is required for FSx Windows authentication).
4. Review the details and click **Create file system**. *(Note: This process takes around 15-20 minutes to complete).*

---

## Part 2: Step-by-Step Deletion Process (Cost Optimization)

To prevent your AWS credits from exhausting and avoid unwanted post-trial billing, delete all resources in the following chronological order immediately after taking project verification screenshots:

### 1. Delete Amazon FSx & Active Directory (Highest Priority)
1. Open the **FSx** console.
2. Select `Windows-Shared-FSx`.
3. Click **Actions** > **Delete file system**.
4. Choose **No** for creating a final backup, check the acknowledgement box, type `DELETE` in the field, and click **Delete file system**.
5. Go to the **Directory Service** console, select the Active Directory created for FSx, and click **Delete**.

### 2. Delete Amazon EFS
1. Open the **EFS** console.
2. Select `User-Uploads-EFS` from the file systems list.
3. Click **Delete**, enter the File System ID to confirm, and click **Delete**.

### 3. Terminate EC2 Instances and Attached EBS Volumes
1. Go back to the **EC2 Dashboard** > **Instances**.
2. Select both `Linux-Web-Server` and `Windows-DB-Server`.
3. Click **Instance State** > **Terminate instance**.
4. Click **Terminate** in the confirmation prompt. *(This will automatically delete the root volumes).*
5. Navigate to **Elastic Block Store** > **Volumes** from the left sidebar.
6. Verify if the two manually created 10 GiB `gp3` volumes are deleted. If their status shows **Available** (not In-Use), select them, click **Actions** > **Delete volume**.
