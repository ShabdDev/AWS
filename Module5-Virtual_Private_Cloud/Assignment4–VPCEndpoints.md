# Module 5: Assignment 4 - VPC Gateway Endpoints for Secure Amazon S3 Access

## Problem Statement
XYZ Corporation requires a highly secure data transmission architecture where private EC2 workloads can access storage assets without traversing the public internet. Implement an Amazon VPC Gateway Endpoint for Amazon S3 to establish a private network route, keeping all storage API calls strictly within the AWS network backbone.

---

## Tasks To Be Performed
1. Create a VPC Endpoint for an Amazon S3 bucket of your choice to enable private and secure access to files.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Pre-requisites (Ensure a Custom/Default VPC and S3 Bucket exist)
1. Ensure you have an active VPC (Default VPC or the custom VPC from Assignment 1).
2. Note down your VPC's **Route Table ID** (e.g., `rtb-0123456789abcdef0`).
3. Create a quick test S3 bucket:
   * Navigate to **S3 Console** > **Create bucket**.
   * Name: `xyz-secure-test-bucket-uniqueid` (Choose a globally unique name).
   * Click **Create bucket** keeping all defaults.

---

### Step 2: Create the VPC Gateway Endpoint for S3
1. Log in to the **AWS Management Console** and navigate to the **VPC Dashboard**.
2. In the left navigation sidebar, scroll down to the *Virtual Private Cloud* section and click on **Endpoints**.
3. Click on the **Create endpoint** button.
4. Configure the endpoint settings:
   * **Name tag:** `XYZ-S3-Gateway-Endpoint`
   * **Service category:** Ensure **AWS services** is selected.
   * **Services Search:** Type `s3` in the search box and press Enter.
   * **Service Name Selection:** Select the service named `com.amazonaws.us-east-1.s3` (or your active region string) with the Type listed explicitly as **Gateway**.
5. **VPC Selection:** Select the VPC where your instances or subnets reside.
6. **Route Tables Mapping:** Check the box next to your VPC’s **Route Table** (This automatically injects the secure S3 prefix-list route entry into your subnet configuration).
7. **Policy:** Leave it as **Full access** (or restrict it to your specific bucket ARN if needed for granular security compliance).
8. Click **Create endpoint**.

---

### Step 3: Verification (How it works behind the scenes)
1. Go to **Route Tables** in the VPC console and select the Route Table you mapped above.
2. Click on the **Routes** tab.
3. You will see a new destination rule added automatically:
   * **Destination:** `pl-xxxxxxxx` (AWS Prefix List for S3)
   * **Target:** `vpce-xxxxxxxxxxxxxxxxx` (Your newly created S3 Gateway Endpoint ID)
4. Any EC2 instance running in this VPC can now use the AWS CLI to upload/download files from the S3 bucket privately without needing an Internet Gateway or a Public IP address.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To clear unnecessary network pathways and restore your route table back to its baseline default configuration, follow these clean-up steps:

### 1. Delete the VPC Endpoint
1. Open the **VPC Dashboard** > click **Endpoints** from the sidebar menu.
2. Select the checkbox next to `XYZ-S3-Gateway-Endpoint`.
3. Click the **Actions** dropdown menu at the top right and select **Delete VPC endpoints**.
4. Confirm the deletion prompt. *(This instantly and safely clears the automated prefix-list routing rows from your network's core route tables).*

### 2. Delete the Test S3 Bucket
1. Go to the **S3 Console**.
2. Select your test bucket `xyz-secure-test-bucket-uniqueid`.
3. Click **Empty** if you uploaded any test assets, confirm, and click delete.
4. Click **Delete**, type the exact bucket name to confirm permanent removal.
