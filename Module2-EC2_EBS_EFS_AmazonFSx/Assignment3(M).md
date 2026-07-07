# Module 2: EC2 and EFS Assignment (Mandatory)

## Problem Statement
Configure shared network storage across a multi-OS environment by creating an Amazon Elastic File System (EFS) and mounting it concurrently on three separate EC2 instances running different Operating Systems.

---

## Tasks To Be Performed
1. Create an Amazon EFS file system.
2. Launch 3 different EC2 instances with distinct Operating Systems:
   * Ubuntu Server
   * Red Hat Enterprise Linux (RHEL)
   * Amazon Linux 2
3. Connect the EFS file system to all 3 instances simultaneously and verify shared data accessibility.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create a Security Group for EFS
1. Log in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. Click **Security Groups** from the left panel and click **Create security group**.
   * **Security group name:** `EFS-Shared-SG`
   * **Description:** Allow NFS traffic from EC2 instances.
   * **VPC:** Select your Default VPC.
3. In **Inbound rules**, click **Add rule**:
   * **Type:** `NFS` (Port 2049)
   * **Source:** `0.0.0.0/0` (or select the specific security group of your EC2 instances for better security).
4. Click **Create security group**.

### Step 2: Create an Amazon EFS File System
1. Search for and navigate to the **EFS** console.
2. Click **Create file system**.
3. In the pop-up box:
   * **Name:** `Multi-OS-Shared-EFS`
   * **VPC:** Select your Default VPC.
4. Click **Customize** to ensure proper network attachment:
   * Go to the **Network** tab.
   * For every Availability Zone listed, replace the default security group with the `EFS-Shared-SG` you created in Step 1.
5. Click **Next** through the remaining steps and click **Create**. Note down the **File System ID** (e.g., `fs-0123456789abcdef0`).

### Step 3: Launch 3 Different OS EC2 Instances
Go back to the **EC2 Dashboard** -> **Launch Instance**. Create three individual instances with the same Key Pair, default VPC, and a security group that allows SSH (Port 22).

1. **Instance 1 (Amazon Linux 2):**
   * **Name:** `OS-Amazon-Linux`
   * **AMI:** Amazon Linux 2 AMI
   * **Instance Type:** `t2.micro`
2. **Instance 2 (Ubuntu):**
   * **Name:** `OS-Ubuntu`
   * **AMI:** Ubuntu Server 24.04 LTS (or 22.04 LTS)
   * **Instance Type:** `t2.micro`
3. **Instance 3 (Red Hat RHEL):**
   * **Name:** `OS-RedHat`
   * **AMI:** Red Hat Enterprise Linux 9 (RHEL 9)
   * **Instance Type:** `t2.micro`

Click **Launch Instance** for each.

### Step 4: Mount EFS on Each Instance

Connect to each instance individually using **EC2 Instance Connect** (or SSH) and run the respective commands based on their OS type.

#### **1. Configuration on Amazon Linux 2 Instance:**
```bash
# Update system and install Amazon EFS utilities
sudo yum update -y
sudo yum install -y amazon-efs-utils

# Create a mount directory
sudo mkdir -p /mnt/shared-efs

# Mount the EFS file system (Replace YOUR_EFS_ID with actual ID)
sudo mount -t efs -o tls YOUR_EFS_ID:/ /mnt/shared-efs

```

#### **2. Configuration on Ubuntu Instance:**

```bash
# Update package lists and install EFS helper dependencies
sudo apt update -y
sudo apt install -y binutils

# Clone and install amazon-efs-utils from source for Ubuntu compatibility
git clone [https://github.com/aws/efs-utils](https://github.com/aws/efs-utils)
cd efs-utils
./build-deb.sh
sudo apt-get install ./build/amazon-efs-utils*deb -y

# Create mount directory and mount EFS
sudo mkdir -p /mnt/shared-efs
sudo mount -t efs -o tls YOUR_EFS_ID:/ /mnt/shared-efs

```

#### **3. Configuration on Red Hat Linux (RHEL) Instance:**

```bash
# Install development tools and dependencies
sudo yum install -y rpm-build make standard-test-roles git

# Clone and install amazon-efs-utils for RHEL
git clone [https://github.com/aws/efs-utils](https://github.com/aws/efs-utils)
cd efs-utils
make rpm
sudo yum install -y build/amazon-efs-utils*rpm

# Create mount directory and mount EFS
sudo mkdir -p /mnt/shared-efs
sudo mount -t efs -o tls YOUR_EFS_ID:/ /mnt/shared-efs

```

### Step 5: Verify Shared Storage (The Proof)

1. In the **Amazon Linux** terminal, create a file inside the shared directory:
```bash
cd /mnt/shared-efs
echo "Hello from Amazon Linux! EFS is working perfectly across multiple OS environments." | sudo tee shared-demo.txt

```


2. Now, check the **Ubuntu** and **Red Hat** terminals by executing:
```bash
cat /mnt/shared-efs/shared-demo.txt

```


*(If the file and text are successfully visible in all three distinct OS sessions, the architecture is correctly configured).*

---

## Part 2: Step-by-Step Deletion Process (Cost Optimization)

To avoid active persistent volume accumulation and prevent continuous API call charges against your account balance, perform the destruction steps in this sequential order:

### 1. Delete Amazon EFS (Highest Priority)

1. Go to the **Amazon EFS** console dashboard.
2. Select the checkbox next to `Multi-OS-Shared-EFS`.
3. Click on the **Delete** button.
4. Type the unique **File System ID** into the text confirmation box and click **Delete file system** to confirm permanent removal.

### 2. Terminate All 3 EC2 Instances

1. Navigate back to the **EC2 Dashboard** > **Instances**.
2. Select the checkboxes for all three instances: `OS-Amazon-Linux`, `OS-Ubuntu`, and `OS-RedHat`.
3. Click the **Instance State** dropdown option at the top menu and click **Terminate instance**.
4. Confirm by clicking **Terminate** in the pop-up panel.

### 3. Remove Security Groups

1. Go to **Security Groups** under Network & Security section.
2. Find `EFS-Shared-SG`, select it, click **Actions** > **Delete security groups**.
