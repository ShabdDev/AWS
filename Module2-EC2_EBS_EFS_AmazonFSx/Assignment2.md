# Module 2: Assignment 2 - EBS Assignment

## Problem Statement
You work for XYZ Corporation. Your corporation wants to launch a new web-based application using AWS Virtual Machines. Configure the resources accordingly with appropriate storage for the tasks.

---

## Tasks To Be Performed
1. Launch a Linux EC2 instance.
2. Create an EBS volume with 20 GB of storage and attach it to the created EC2 instance.
3. Resize the attached volume and make sure it reflects in the connected instance.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Launch a Linux EC2 Instance
1. Log in to the **AWS Management Console** and open the **EC2 Dashboard**.
2. Click on **Launch Instance**.
3. Configure the instance with the following details:
   * **Name:** `XYZ-Storage-Server`
   * **Application and OS Image (AMI):** Select **Amazon Linux 2023 AMI** (Free Tier Eligible).
   * **Instance Type:** `t2.micro` (or `t3.micro`).
   * **Key Pair:** Select an existing key pair or create a new one.
   * **Network Settings:** Note down the **Availability Zone** where the instance is being launched (e.g., `us-east-1a`). *The EBS volume must be created in this exact same AZ.*
4. Click **Launch Instance**.

### Step 2: Create a 20 GB EBS Volume and Attach It
1. In the left-hand navigation menu of the EC2 Dashboard, go to **Elastic Block Store** > **Volumes**.
2. Click **Create Volume**.
   * **Volume Type:** General Purpose SSD (`gp3`).
   * **Size:** `20 GiB`.
   * **Availability Zone:** Select the **exact same Availability Zone** as your EC2 instance (e.g., `us-east-1a`).
   * **Tags:** Add a key `Name` and value `XYZ-Extra-Volume`.
3. Click **Create volume**.
4. Once the volume state becomes **Available**, select the volume, click **Actions** > **Attach volume**.
5. Select your `XYZ-Storage-Server` instance from the list. 
6. Keep the default device name (e.g., `/dev/sdf`) and click **Attach volume**.

### Step 3: Verify the Volume inside the Instance
1. Go back to **Instances**, select `XYZ-Storage-Server`, and click **Connect** -> **EC2 Instance Connect** -> **Connect**.
2. Run the following command to list all attached storage devices:
   ```bash
   lsblk
   ```

*(You will see the root disk and the newly attached 20 GB disk, usually named `xvdf` or `nvme1n1` depending on the instance type).*

### Step 4: Resize the Attached EBS Volume (AWS Console)

1. Go back to the AWS Console -> **Volumes**.
2. Select your 20 GB volume (`XYZ-Extra-Volume`).
3. Click **Actions** > **Modify volume**.
4. Change the Size from `20` to `25` (or any value larger than 20 GB, e.g., `25 GiB`).
5. Click **Modify** and confirm the pop-up. The volume state will change to **Modifying** and then back to **In-Use (Optimizing)**.

### Step 5: Extend the Filesystem inside the Connected Instance

1. Go back to your connected EC2 terminal window.
2. Check the updated disk size by running:
```bash
lsblk
```

*(You will notice the block device size has increased to 25 GB, but if a filesystem was mounted, it needs to be extended).*
3. Since this is a raw disk with no partitions yet, if you create a partition or filesystem, use the appropriate command to extend it. For a standard `ext4` filesystem on a partition, you would use:
```bash
# If partition exists (e.g., nvme1n1p1 or xvdf1)
sudo growpart /dev/nvme1n1 1
sudo resize2fs /dev/nvme1n1p1

```

*(For this assignment validation, showing the updated 25 GB size in the `lsblk` output command is sufficient proof of resizing).*

---

## Part 2: Step-by-Step Deletion Process (Cost Optimization)

To clean up resources and prevent any monthly accumulation charges on your EBS quota, delete the setup in this order:

1. Open the **EC2 Dashboard** and click on **Instances (running)**.
2. Select `XYZ-Storage-Server`.
3. Click **Instance State** > **Terminate instance** and click **Terminate** to confirm.
4. *(Terminating the instance will automatically detach the extra 20/25 GB volume).*
5. Go to **Elastic Block Store** > **Volumes** from the left panel.
6. Refresh the page until the state of `XYZ-Extra-Volume` changes from *In-Use* to **Available**.
7. Select `XYZ-Extra-Volume`, click **Actions** > **Delete volume**, and confirm the deletion.
