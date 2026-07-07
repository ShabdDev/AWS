# Module 4: Case Study - End-to-End Elastic Infrastructure (ALB, ASG, & Route 53)

## Problem Statement
XYZ Corporation is migrating from an expensive on-premises system to AWS to efficiently handle unpredictable traffic spikes. To achieve maximum cost efficiency and high availability, architect an automated infrastructure that dynamically scales compute resources based on real-time CPU thresholds (Scale-out at >80%, Scale-in at <60%), balances incoming traffic evenly using an Application Load Balancer, and binds the traffic to the company’s public domain namespace.

---

## Tasks To Be Performed
1. **Manage scaling requirements:**
   * Automatically deploy compute resources (Scale-out) when CPU utilization exceeds 80%.
   * Automatically remove compute resources (Scale-in) when CPU utilization drops below 60%.
2. **Load Balancing:** Create an Application Load Balancer to distribute the load between active compute resources.
3. **DNS Routing:** Route the incoming traffic to the company's registered domain using Route 53.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create a Custom Launch Template & Custom AMI
1. Launch a base EC2 instance (`t2.micro`, Ubuntu) with Apache2 or Nginx pre-configured to handle web requests.
2. Create an Amazon Machine Image (AMI) from this instance and name it `XYZ-Prod-Web-AMI`.
3. Navigate to **EC2 Dashboard** > **Launch Templates** and click **Create launch template**.
   * **Template Name:** `XYZ-Prod-Template`
   * **AMI:** Select `XYZ-Prod-Web-AMI`.
   * **Instance Type:** `t2.micro`.
   * **Security Group:** Select a group that allows HTTP (Port 80) and SSH (Port 22).

---

### Step 2: Provision the Application Load Balancer (ALB)
1. Navigate to **EC2** > **Target Groups** and click **Create target group**.
   * Choose **Instances**, name it `XYZ-Production-TG`, set protocol to **HTTP (Port 80)**, and select your Default VPC. Click **Next** and create without registering targets manually (ASG will handle this).
2. Go to **Load Balancers** > **Create load balancer** > Select **Application Load Balancer**.
   * **Name:** `XYZ-Prod-ALB`
   * **Network Mapping:** Select Default VPC and check all available Subnets/AZs.
   * **Listeners and Routing:** Forward **HTTP:80** directly to `XYZ-Production-TG`.
3. Click **Create load balancer** and copy its **DNS Name** once provisioned.

---

### Step 3: Deploy Auto Scaling Group with Target Tracking Scaling Policies
1. Navigate to **Auto Scaling Groups** > **Create Auto Scaling group**.
   * **Name:** `XYZ-Dynamic-ASG`
   * **Launch Template:** Select `XYZ-Prod-Template`. Click **Next**.
   * **VPC & Subnets:** Select your Default VPC and multiple subnets. Click **Next**.
   * **Load Balancing:** Check **Attach to an existing load balancer** and select `XYZ-Production-TG`.
2. **Configure Group Size & Scaling Policies:**
   * **Desired capacity:** `2`
   * **Minimum capacity:** `1`
   * **Maximum capacity:** `5`
3. **Set Up Dynamic Scaling Policies (Task 1 Requirements):**
   * Under Scaling Policies, select **Target tracking scaling policy**.
   * **Metric type:** `Average CPU utilization`
   * **Target value:** `80` (This automatically triggers a Scale-out deployment whenever average CPU load hits >80%).
4. *Alternative/Manual Step Alarm Configuration:* To explicitly fulfill both 80% and 60% step boundary requirements, configure **Step Scaling Policies** linked with CloudWatch alarms:
   * **High-CPU Alarm:** Trigger when CPU > 80% for 1 period -> Add 1 instance.
   * **Low-CPU Alarm:** Trigger when CPU < 60% for 1 period -> Remove 1 instance.
5. Click **Next**, review all parameters, and click **Create Auto Scaling group**.

---

### Step 4: Configure Domain Traffic Routing via Route 53
1. Navigate to the **Route 53** console and open your existing **Public Hosted Zone** (e.g., `xyzcorp.com`).
2. Click **Create record**.
3. Configure the Alias record configuration parameters:
   * **Record name:** Keep it blank or use `www`.
   * **Record type:** `A – Routes traffic to an IPv4 address and some AWS resources`
   * Toggle the **Alias** switch to **ON**.
   * **Route traffic to:** Choose *Alias to Application and Classic Load Balancer*.
   * **Region:** Select the AWS region where you deployed your ALB.
   * **Choose load balancer:** Select your provisioned `XYZ-Prod-ALB`.
4. Click **Create records**. 
5. Test the architecture by accessing your corporate domain name in a browser. Route 53 will resolve it to the ALB, which will evenly hit the dynamic instances managed under your auto-scaling policies.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent severe credit exhaustion from high-availability active architecture rules, execute clean-up in this exact sequential order:

### 1. Delete the Auto Scaling Group (First Priority)
1. Open the **Auto Scaling Groups** console view.
2. Select `XYZ-Dynamic-ASG`, click **Delete**, and confirm by typing `delete`. 
3. *(This ensures all backend EC2 nodes scaling up or down are terminated immediately).*

### 2. Delete the Application Load Balancer & Target Group
1. Go to **Load Balancers**, select `XYZ-Prod-ALB`, click **Actions** > **Delete load balancer**, and confirm.
2. Go to **Target Groups**, select `XYZ-Production-TG`, click **Actions** > **Delete**, and confirm.

### 3. Clear Route 53 DNS Records
1. Open the **Route 53** console -> **Hosted zones** -> Select `xyzcorp.com`.
2. Select the **Alias A record** created for the ALB, click **Delete record**, and confirm.
3. *(Optional)* Delete the hosted zone if you do not plan to use it for future modules to stop monthly zone hosting fees.

### 4. Deregister AMI and Templates
1. Navigate to **EC2** > **AMIs**, select `XYZ-Prod-Web-AMI`, and click **Actions** > **Deregister AMI**.
2. Delete the associated disk snapshot from the **Elastic Block Store > Snapshots** panel.
