# Module 10: Assignment 2 - High-Availability Traffic Routing via AWS Global Accelerator

## Problem Statement
XYZ Corporation requires a global traffic management solution to optimize network latency and provide instant failover capabilities for its multi-region web platform. Implement an AWS Global Accelerator configuration to provision static Anycast IP addresses that route user traffic efficiently over the AWS global network backbone, tying a standalone Amazon EC2 Compute Instance in a primary region and an Application Load Balancer (ALB) deployed in an alternate geographical region as active backend routing endpoints.

---

## Tasks To Be Performed
1. Create and provision an AWS Global Accelerator.
2. Establish a multi-region endpoint distribution matrix containing:
   * **Endpoint A:** A live standalone EC2 instance deployed within Region 1 (e.g., `us-east-1`).
   * **Endpoint B:** An Application Load Balancer (ALB) deployed within Region 2 (e.g., `us-west-2`), completely separate from the EC2 instance region.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision Core Infrastructure Endpoints across Separate Regions
*(Before configuring Global Accelerator, the infrastructure components must be operational in their respective regions).*

1. **Deploy Endpoint A (EC2 Instance in Region 1):**
   * Switch your AWS console region to **N. Virginia (`us-east-1`)**.
   * Launch a baseline `t2.micro` EC2 Instance running Amazon Linux. Name it `XYZ-Primary-EC2-Endpoint`.
   * Ensure its security group permits inbound **HTTP (Port 80)** traffic.

2. **Deploy Endpoint B (Load Balancer in Region 2):**
   * Switch your AWS console region selector to **Oregon (`us-west-2`)**.
   * Launch a cost-optimized baseline `t2.micro` EC2 instance to serve as a target backend host.
   * Navigate to **EC2 > Load Balancing > Load Balancers** and click **Create load balancer**.
   * Select **Application Load Balancer (ALB)**, configure public internet-facing mappings, choose the public subnets of Oregon, link a target group containing your Oregon instance on Port 80, and name it **`XYZ-CrossRegion-ALB`**.

---

### Step 2: Configure the AWS Global Accelerator Layout
1. In the AWS Console search field, query **Global Accelerator** to open its unified global dashboard interface.
2. Click on the orange **Create accelerator** button.
3. **Configure Accelerator Parameters:**
   * **Accelerator name:** `XYZ-Global-Traffic-Optimizer`
   * **Type:** Select **Standard** (Handles standard layer-4/layer-7 application traffic routing rules).
   * Click **Next**.

4. **Define Listeners (Step 2 of the Wizard):**
   * **Ports:** Enter **`80`**
   * **Protocol:** Select **TCP**
   * **Client affinity:** Choose **None** (Allows equal distribution based on latency calculations).
   * Click **Next**.

5. **Configure Endpoint Groups (Step 3 of the Wizard):**
   * Add the first regional bucket mapping: Select **`us-east-1`** (N. Virginia region).
   * Click **Add endpoint group** to append a multi-region routing slot.
   * Add the second regional bucket mapping: Select **`us-west-2`** (Oregon region).
   * Click **Next**.

6. **Register Backend Endpoints (Step 4 of the Wizard):**
   * **For the `us-east-1` Endpoint Group:**
     * Click **Add endpoint**.
     * **Endpoint type:** Select **EC2 instance**.
     * **Endpoint identifier:** Select your deployed resource: `XYZ-Primary-EC2-Endpoint`.
   * **For the `us-west-2` Endpoint Group:**
     * Click **Add endpoint**.
     * **Endpoint type:** Select **Application Load Balancer**.
     * **Endpoint identifier:** Select your cross-region network load target: `XYZ-CrossRegion-ALB`.
7. Click **Create accelerator**.

---

### Step 3: Validate Multi-Region Routing Architecture
1. Once deployed, the status column shifts from *In Progress* to a green **Deployed** flag.
2. Copy the newly generated **Static Anycast IP Addresses** or the global **DNS Name** (e.g., `a1234567890abcdef.awsglobalaccelerator.com`) from the setup overview panel.
3. Paste the DNS string into your browser. 
4. **Result:** The Anycast network will automatically evaluate your physical testing location and route your browser traffic smoothly to whichever endpoint is closest to you geographically. If you manually simulate a resource outage by stopping the EC2 instance in `us-east-1`, the Global Accelerator will execute an automatic health-check failover loop, rerouting your traffic to the active ALB endpoint in `us-west-2` seamlessly.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To instantly stop the hourly runtime fixed fees associated with keeping Anycast IP allocations live on your account profile, run this cleanup pipeline immediately:

### 1. Disable and Delete the Global Accelerator
1. Navigate to the **AWS Global Accelerator Console**.
2. Click on the name link matching **`XYZ-Global-Traffic-Optimizer`**.
3. *Note: AWS prevents the immediate deletion of active accelerators. You must disable it first.*
4. Click on the **Edit** action button at the top header, uncheck the **Enable** toggle checkbox status switch, and click **Save**.
5. Wait roughly 1 minute for the update loop to sync. Select the accelerator row element checkbox, click **Delete**, and confirm the destruction modal command.

### 2. Teardown Regional Compute and Load Balancing Elements
1. Switch to the **Oregon (`us-west-2`)** console region: Delete `XYZ-CrossRegion-ALB`, remove its associated Target Group profiles, and terminate the background testing EC2 instance.
2. Switch to the **N. Virginia (`us-east-1`)** console region: Terminate the `XYZ-Primary-EC2-Endpoint` computing host server cleanly.
