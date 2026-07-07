# Module 4: Assignment 2 - Auto Scaling Infrastructure Deployment

## Problem Statement
XYZ Corporation requires an automated elasticity solution to handle sudden application traffic spikes without manual hardware procurement. Implement a scalable architecture on AWS by building a custom golden AMI pre-configured with an Apache2 web server, creating an EC2 Launch Configuration, and deploying an Auto Scaling Group (ASG) with a baseline capacity of 1 and a maximum limit of 3 instances.

---

## Tasks To Be Performed
1. Create a web server AMI with Apache 2 server running in it.
2. Create a launch configuration with this AMI.
3. Use this launch configuration to create an Auto Scaling group with 1 minimum and 3 maximum instances.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create a Base Web Server and Build a Custom AMI
1. Log in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. Click **Launch Instance** to create a temporary base server:
   * **Name:** `XYZ-Base-WebServer`
   * **AMI:** **Ubuntu Server** (Free Tier Eligible).
   * **Instance Type:** `t2.micro`.
   * **Security Group:** Allow **SSH (Port 22)** and **HTTP (Port 80)**.
3. Launch and connect to the instance using **EC2 Instance Connect**.
4. Run the following commands to update repositories, install the Apache2 web server, and start the service:
   ```bash
   sudo apt update -y
   sudo apt install apache2 -y
   sudo systemctl start apache2
   sudo systemctl enable apache2
   echo "<h1>XYZ Corp - Automated Auto Scaling Instance</h1>" | sudo tee /var/www/html/index.html
   ```

5. Go back to the **EC2 Console > Instances**, select `XYZ-Base-WebServer`, click **Actions** > **Image and templates** > **Create image**.
* **Image name:** `XYZ-Apache-Golden-AMI`


6. Click **Create image**. (Navigate to **Images > AMIs** from the left menu to verify when its status changes from *pending* to *available*).
7. *Note: Once the AMI is available, you can terminate the `XYZ-Base-WebServer` instance.*

---

### Step 2: Create an EC2 Launch Configuration

1. In the left navigation pane of the EC2 Dashboard, under *Instances*, click on **Launch Configurations**.
2. Click **Create launch configuration**.
3. Configure the details:
* **Launch configuration name:** `XYZ-Apache-Launch-Config`
* **Amazon Machine Image (AMI):** Click *Owned by me* and select your `XYZ-Apache-Golden-AMI`.
* **Instance Type:** Select `t2.micro`.


4. **Advanced details & Security Groups:**
* Select an existing security group that opens **HTTP (Port 80)** traffic.
* Choose your standard login Key Pair.


5. Click **Create launch configuration**.

---

### Step 3: Deploy the Auto Scaling Group (ASG)

1. In the left panel, under *Auto Scaling*, click on **Auto Scaling Groups**.
2. Click **Create Auto Scaling group**.
3. **Step 1: Choose launch template or configuration:**
* **Auto Scaling group name:** `XYZ-Web-ASG`
* Click the link **Switch to launch configuration**, and select `XYZ-Apache-Launch-Config`. Click **Next**.


4. **Step 2: Configure settings:**
* Select your **Default VPC** and choose at least two distinct **Subnets (Availability Zones)** to ensure high availability. Click **Next**.


5. **Step 3: Configure advanced options:** Keep defaults and click **Next**.
6. **Step 4: Configure group size and scaling policies:**
* **Desired capacity:** `1`
* **Minimum capacity:** `1`
* **Maximum capacity:** `3`


7. Click **Next** through the remaining steps, review your configurations, and click **Create Auto Scaling group**.
8. **Verification:** Navigate to **EC2 > Instances**. You will observe that the ASG automatically triggers and spawns exactly 1 new instance based on your custom Apache AMI to meet the desired/minimum capacity rule.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent the Auto Scaling mechanism from continuously launching resources and consuming your account balance, clean up assets in this exact sequence:

### 1. Delete the Auto Scaling Group (Highest Priority)

1. Navigate to **Auto Scaling Groups**.
2. Select `XYZ-Web-ASG`, click **Delete**, and type `delete` to confirm.
3. *(This single action will automatically stop and terminate all underlying EC2 instances managed by this group).*

### 2. Remove Launch Configuration

1. Navigate to **Launch Configurations**.
2. Select `XYZ-Apache-Launch-Config`, click **Actions** > **Delete launch configuration**, and confirm.

### 3. Deregister AMI and Delete Attached Snapshot

1. Navigate to **Images > AMIs**.
2. Select `XYZ-Apache-Golden-AMI`, click **Actions** > **Deregister AMI**.
3. Go to **Elastic Block Store > Snapshots**.
4. Find the snapshot automatically created along with your AMI, select it, click **Actions** > **Delete snapshot**, and confirm to free up EBS allocation quotas.

```
