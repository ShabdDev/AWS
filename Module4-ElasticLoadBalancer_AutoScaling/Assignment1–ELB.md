
# Module 4: Assignment 1 - Classic to Application Load Balancer Migration

## Problem Statement
XYZ Corporation is migrating from an on-premises setup to AWS to efficiently handle increasing application traffic load. To achieve high availability and modernization, deploy a Classic Load Balancer (CLB) routing traffic to three EC2 instances serving distinct web pages, and then migrate the architecture into an Application Load Balancer (ALB).

---

## Tasks To Be Performed
1. Create a Classic Load Balancer and register 3 EC2 instances with different web pages running in them.
2. Migrate the Classic Load Balancer into an Application Load Balancer.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Launch 3 EC2 Instances with Distinct Web Pages
Launch three separate EC2 instances (`t2.micro`, Ubuntu OS) in the same Default VPC but spread across different Availability Zones if possible. Ensure their Security Group allows **HTTP (Port 80)** traffic.

Connect to each instance via **EC2 Instance Connect** and configure unique web pages:

* **Instance 1 (Web-Server-1):**
  ```bash
  sudo apt update && sudo apt install nginx -y
  echo "<h1>Welcome to Web Server 1 - App Page A</h1>" | sudo tee /var/www/html/index.html

```

* **Instance 2 (Web-Server-2):**
```bash
sudo apt update && sudo apt install nginx -y
echo "<h1>Welcome to Web Server 2 - App Page B</h1>" | sudo tee /var/www/html/index.html

```

* **Instance 3 (Web-Server-3):**
```bash
sudo apt update && sudo apt install nginx -y
echo "<h1>Welcome to Web Server 3 - App Page C</h1>" | sudo tee /var/www/html/index.html

```
---

### Step 2: Create a Classic Load Balancer (CLB)

1. Navigate to the **EC2 Dashboard** and click **Load Balancers** under the *Load Balancing* section in the left panel.
2. Click **Create load balancer**.
3. Scroll down to the **Classic Load Balancer** option and click **Create**.
4. Configure basic settings:
* **Load Balancer name:** `XYZ-Classic-LB`
* **VPC:** Select your Default VPC.
* **Listeners:** Keep default HTTP (Port 80) routing to HTTP (Port 80).


5. **Assign Security Groups:** Select a security group that allows inbound HTTP (Port 80) traffic from anywhere (`0.0.0.0/0`).
6. **Configure Health Check:** Set Ping Path to `/index.html`.
7. **Register EC2 Instances:** Select all three instances (`Web-Server-1`, `Web-Server-2`, and `Web-Server-3`) created in Step 1.
8. Review the configurations and click **Create**.
9. Once the state becomes active, copy the **DNS name** of the CLB, paste it into a browser tab, and refresh repeatedly. You will see the traffic balancing across Page A, Page B, and Page C.

---

### Step 3: Migrate Classic Load Balancer to Application Load Balancer (ALB)

#### 1. Create a Target Group for the ALB

1. In the left panel, click **Target Groups** > **Create target group**.
2. Choose **Instances** as the target type.
3. **Target group name:** `XYZ-ALB-TargetGroup`
4. Protocol: **HTTP** | Port: **80** | VPC: Default VPC.
5. Click **Next**, select all 3 web server instances, click **Include as pending below**, and click **Create target group**.

#### 2. Deploy the Application Load Balancer

1. Go back to **Load Balancers** > **Create load balancer**.
2. Select **Application Load Balancer** and click **Create**.
3. **Basic Configuration:**
* **Load balancer name:** `XYZ-Modern-ALB`
* **Scheme:** Internet-facing
* **IP address type:** IPv4


4. **Network mapping:** Select your Default VPC and check at least two Availability Zones (Subnets) where your instances reside.
5. **Security groups:** Select the HTTP-allowed security group.
6. **Listeners and routing:** Under HTTP:80, change the Default Action to **Forward to** and select your newly created target group `XYZ-ALB-TargetGroup`.
7. Click **Create load balancer**.
8. Verify by copying the new ALB **DNS name** into a browser. The application traffic will now be handled by the advanced path/routing layer of the ALB.

---

## Part 2: Step-by-Step Deletion Process (Cost Optimization)

To avoid infrastructure overhead and persistent hourly balancer retention rates against your account, delete the setup in this exact sequence:

### 1. Delete Load Balancers

1. Open the **EC2 Dashboard** > **Load Balancers**.
2. Select `XYZ-Classic-LB` and click **Actions** > **Delete load balancer**. Confirm the action.
3. Select `XYZ-Modern-ALB` and click **Actions** > **Delete load balancer**. Confirm the action.

### 2. Remove Target Group

1. Navigate to **Target Groups** from the left-side menu.
2. Select `XYZ-ALB-TargetGroup`.
3. Click **Actions** > **Delete** and confirm.

### 3. Terminate EC2 Instances

1. Go to **EC2 Dashboard** > **Instances**.
2. Select `Web-Server-1`, `Web-Server-2`, and `Web-Server-3`.
3. Click **Instance State** > **Terminate instance** and confirm permanent removal.
