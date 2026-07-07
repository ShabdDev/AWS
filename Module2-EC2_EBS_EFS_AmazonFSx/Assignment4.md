# Module 2: Assignment 4 - FSx Assignment

## Problem Statement
XYZ Corporation requires faster file sharing capabilities that support data replication from on-premises infrastructure. To achieve this, deploy an Amazon FSx for Windows File Server integrated with AWS Managed Active Directory, and set up an Amazon FSx for Lustre file system attached to a high-performance Linux instance.

---

## Tasks To Be Performed
1. Create an FSx file system for a Windows file server:
   * Setup an AWS Managed Active Directory with a valid domain name.
   * Connect the FSx Windows file system to a Windows EC2 server.
2. Create an FSx file system for Lustre and attach it to an Amazon Linux 2 instance.

---

## Part 1: Step-by-Step Implementation Solution

### Task 1: FSx for Windows File Server with Active Directory

#### Step 1: Create an AWS Managed Active Directory
1. Navigate to the **Directory Service** console.
2. Click **Set up directory** and select **AWS Managed Microsoft AD**. Click **Next**.
3. Configure the directory details:
   * **Edition:** `Standard Edition`
   * **Directory DNS name:** `xyzcorp.local`
   * **Directory NetBIOS name:** `XYZCORP`
4. Set an Admin password and click **Next**.
5. Select your default VPC and Subnets, click **Next**, and click **Create directory**. *(Note: Directory creation takes 15-20 minutes).*

#### Step 2: Create FSx for Windows File Server
1. Navigate to the **FSx** console and click **Create file system**.
2. Select **Amazon FSx for Windows File Server** and click **Next**.
3. Configure the file system:
   * **File system name:** `XYZ-Windows-FSx`
   * **Deployment type:** `Single-AZ`
   * **Storage capacity:** `32 GiB` (Minimum allowed)
   * **Throughput capacity:** Keep default (`8 MB/s` or minimum available).
   * **VPC & Subnet:** Choose the same VPC used for Active Directory.
   * **Windows Authentication:** Select **AWS Managed Microsoft AD** and choose the directory created (`xyzcorp.local`).
4. Click **Next**, review configurations, and click **Create file system**.

#### Step 3: Launch Windows EC2 and Map the FSx Drive
1. Go to the **EC2 Dashboard** and launch a Windows instance (`t2.micro`, AMI: **Microsoft Windows Server 2022 Base**). 
2. In Advanced Details, select the **Domain join directory** as `xyzcorp.local` so it joins the domain automatically.
3. Once running, log in to the Windows instance via **RDP**.
4. Inside Windows, open **File Explorer**, right-click **This PC**, and select **Map network drive**.
5. Choose a drive letter and enter the FSx file share path (Found in the FSx AWS Console under the *Attach* instruction section, e.g., `\\fs-01234567.xyzcorp.local\share`). Click **Finish**.

---

### Task 2: FSx for Lustre with Amazon Linux 2

#### Step 1: Create FSx for Lustre File System
1. Go back to the **FSx** console and click **Create file system**.
2. Select **Amazon FSx for Lustre** and click **Next**.
3. Configure settings:
   * **File system name:** `XYZ-Lustre-FSx`
   * **Deployment type:** `Scratch` (Best for testing/temporary high-performance workloads).
   * **Storage capacity:** `1.2 TiB` (Minimum configuration size limit for Lustre).
   * **Network & Security:** Select your default VPC and default Security Group.
4. Click **Next** and click **Create file system**.

#### Step 2: Launch Amazon Linux 2 and Mount Lustre
1. Launch an EC2 instance with **Amazon Linux 2 AMI** (`t2.micro`) in the exact same Availability Zone where your Lustre file system is deployed.
2. Ensure the EC2 Security Group allows outbound traffic to the Lustre security group.
3. Connect to the Linux instance via **EC2 Instance Connect**.
4. Install the Lustre client plugin on the Linux server by executing:
   ```bash
   sudo amazon-linux-extras install lustre -y
   ```

5. Create a mount directory:
```bash
sudo mkdir -p /mnt/lustre

```


6. Mount the Lustre file system using the Mount Command provided in the FSx dashboard:
```bash
sudo mount -t lustre -o noatime,flock YOUR_FSX_LUSTRE_DNS_NAME@tcp:/fsx /mnt/lustre

```


7. Verify it is mounted by checking disk availability:
```bash
df -h
```
---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent severe credit exhaustion, delete these resources in this exact sequential order immediately after completing the tasks:

### 1. Delete Amazon FSx for Lustre (Deletes 1.2 TB storage allocation)

1. Open the **FSx** console.
2. Select `XYZ-Lustre-FSx`.
3. Click **Actions** > **Delete file system**.
4. Choose **No** for final backup acknowledgement, type `DELETE`, and click confirm.

### 2. Delete Amazon FSx for Windows File Server

1. In the **FSx** console, select `XYZ-Windows-FSx`.
2. Click **Actions** > **Delete file system**.
3. Skip the final backup, acknowledge, type `DELETE`, and confirm.

### 3. Delete AWS Managed Active Directory

1. Navigate to the **Directory Service** console.
2. Select `xyzcorp.local`.
3. Click **Actions** > **Delete directory** and confirm the removal. *(Note: You cannot delete the directory until the attached Windows FSx is fully deleted).*

### 4. Terminate EC2 Instances

1. Go to the **EC2 Dashboard** > **Instances**.
2. Select both your Windows and Amazon Linux 2 instances.
3. Click **Instance State** > **Terminate instance** and confirm.
