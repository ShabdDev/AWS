# Module 5: Assignment 1 - Custom VPC Creation with Public & Private Subnets

## Problem Statement
XYZ Corporation requires a secure, isolated network architecture to safely deploy cloud resources with granular access controls. Implement a custom Virtual Private Cloud (VPC) with a designated network range, provision one public subnet and two private subnets across multiple availability zones, and deploy a NAT Gateway to grant secure outbound internet connectivity to the private workloads.

---

## Tasks To Be Performed
1. Create a VPC with `120.0.0.0/16` CIDR block.
2. Create 1 public subnet and 2 private subnets.
3. Deploy and connect a NAT Gateway for internet connectivity to the private subnets.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the Custom VPC
1. Log in to the **AWS Management Console** and navigate to the **VPC Dashboard**.
2. Click on **Your VPCs** in the left sidebar, then click **Create VPC**.
3. Configure the VPC settings:
   * **Resources to create:** Select **VPC only** *(Note: To learn the underlying concepts, manual creation is recommended instead of using the automated wizard).*
   * **Name tag:** `XYZ-Custom-VPC`
   * **IPv4 CIDR block:** Select *IPv4 CIDR manual input*.
   * **IPv4 CIDR:** `120.0.0.0/16`
   * **Tenancy:** `Default`
4. Click **Create VPC**.

---

### Step 2: Provision the 3 Subnets
Navigate to **Subnets** in the left sidebar and click **Create subnet**. Select `XYZ-Custom-VPC`.

1. **Public Subnet 1:**
   * **Subnet name:** `XYZ-Public-Subnet-1`
   * **Availability Zone:** Select the first zone (e.g., `us-east-1a`).
   * **IPv4 CIDR block:** `120.0.1.0/24`
2. Click **Add new subnet** to create the second one:
   * **Subnet name:** `XYZ-Private-Subnet-1`
   * **Availability Zone:** Select the first zone (e.g., `us-east-1a`).
   * **IPv4 CIDR block:** `120.0.2.0/24`
3. Click **Add new subnet** to create the third one:
   * **Subnet name:** `XYZ-Private-Subnet-2`
   * **Availability Zone:** Select the second zone (e.g., `us-east-1b`).
   * **IPv4 CIDR block:** `120.0.3.0/24`
4. Click **Create subnet**.
5. *Crucial Action for Public Subnet:* Select `XYZ-Public-Subnet-1`, click **Actions** > **Edit subnet settings**, check the box for **Enable auto-assign public IPv4 address**, and click **Save**.

---

### Step 3: Deploy Internet Gateway (IGW) & Configure Public Routing
1. Go to **Internet Gateways** > **Create internet gateway**. Name it `XYZ-VPC-IGW` and click create.
2. Select `XYZ-VPC-IGW`, click **Actions** > **Attach to VPC**, choose `XYZ-Custom-VPC`, and click attach.
3. Go to **Route tables** > **Create route table**. 
   * **Name:** `XYZ-Public-RouteTable` | **VPC:** `XYZ-Custom-VPC`. Click Create.
4. Open `XYZ-Public-RouteTable`, click **Routes** tab > **Edit routes** > **Add route**.
   * Destination: `0.0.0.0/0` | Target: **Internet Gateway** (`XYZ-VPC-IGW`). Click **Save changes**.
5. Go to **Subnet associations** tab > **Edit subnet associations**. Check `XYZ-Public-Subnet-1` and click **Save associations**.

---

### Step 4: Allocate Elastic IP and Deploy NAT Gateway
1. Go to **Elastic IPs** in the left panel under *Network & Security*.
2. Click **Allocate Elastic IP address**, keep defaults, and click **Allocate**. Note the allocated IP identifier.
3. Navigate to **NAT Gateways** > **Create NAT gateway**.
   * **Name:** `XYZ-Private-NAT-GW`
   * **Subnet:** Select `XYZ-Public-Subnet-1` *(CRITICAL: NAT Gateways must always sit inside a Public Subnet to access the internet).*
   * **Connectivity type:** `Public`
   * **Elastic IP allocation ID:** Select the Elastic IP allocated above.
4. Click **Create NAT gateway**. *(Wait 2-3 minutes until the state turns to 'Available')*.

---

### Step 5: Configure Private Routing via NAT Gateway
1. Navigate back to **Route tables** > **Create route table**.
   * **Name:** `XYZ-Private-RouteTable` | **VPC:** `XYZ-Custom-VPC`. Click Create.
2. Open `XYZ-Private-RouteTable`, click **Routes** tab > **Edit routes** > **Add route**.
   * Destination: `0.0.0.0/0` | Target: **NAT Gateway** (`XYZ-Private-NAT-GW`). Click **Save changes**.
3. Go to **Subnet associations** tab > **Edit subnet associations**.
   * Check both `XYZ-Private-Subnet-1` and `XYZ-Private-Subnet-2`.
   * Click **Save associations**.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent severe credit consumption from the running NAT Gateway hourly idle rates, delete the setup in this exact sequence immediately:

### 1. Delete the NAT Gateway (Highest Priority)
1. Navigate to **NAT Gateways**.
2. Select `XYZ-Private-NAT-GW`, click **Actions** > **Delete NAT gateway**.
3. Type `delete` to confirm and wait for the status to switch completely to *Deleted*.

### 2. Release Elastic IP Allocation
1. Navigate to **Elastic IPs**.
2. Select the IP allocated for the NAT gateway.
3. Click **Actions** > **Release Elastic IP address** and confirm. *(Note: You cannot release it until the NAT Gateway is fully deleted).*

### 3. Detach and Delete Internet Gateway
1. Navigate to **Internet Gateways**.
2. Select `XYZ-VPC-IGW`, click **Actions** > **Detach from VPC**, and confirm.
3. Select `XYZ-VPC-IGW` again, click **Actions** > **Delete internet gateway**, and confirm.

### 4. Delete the Custom VPC (Automated Cleanup)
1. Navigate to **Your VPCs**.
2. Select `XYZ-Custom-VPC`.
3. Click **Actions** > **Delete VPC**.
4. Confirm the deletion. *(This action will automatically wipe the custom subnets, route tables, and dependency mapping parameters safely from your account).*
