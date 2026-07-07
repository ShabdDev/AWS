# Module 5: Case Study - Enterprise Multi-Tier Network and Inter-VPC Peering

## Problem Statement
XYZ Corporation is expanding its cloud footprint and requires isolated network environments for its Production and Development teams. Architect a secure, multi-tier infrastructure by deploying a highly secure 4-tier Production Network and a streamlined 2-tier Development Network. Establish a secure VPC Peering connection between both environments to allow direct, private communication specifically between their database layers.

---

## Tasks To Be Performed

### Production Network:
1. Design and build a 4-tier architecture.
2. Create 5 subnets: 1 public (`web`) and 4 private (`app1`, `app2`, `dbcache`, `db`).
3. Launch EC2 instances in all subnets and name them accordingly.
4. Allow `dbcache` instance and `app1` subnet to send outbound internet requests.
5. Configure Security Groups and Network Access Control Lists (NACLs).

### Development Network:
1. Design and build a 2-tier architecture with two subnets (`web` - public, `db` - private) and launch instances in both.
2. Ensure only the `web` subnet can send internet requests.
3. Create a VPC Peering connection between the Production and Development networks.
4. Set up explicit routing connections between the `db` subnets of both networks.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision the VPC Topologies

1. Log in to the **AWS Management Console** and navigate to the **VPC Dashboard**.
2. **Create Production VPC:**
   * Click **Your VPCs** > **Create VPC** (Select *VPC only*).
   * **Name tag:** `XYZ-Production-VPC` | **IPv4 CIDR block:** `10.10.0.0/16`
   * Click **Create VPC**.
3. **Create Development VPC:**
   * Click **Create VPC** again.
   * **Name tag:** `XYZ-Development-VPC` | **IPv4 CIDR block:** `10.20.0.0/16`
   * Click **Create VPC**.

---

### Step 2: Configure Subnets & Create Gateways

#### 1. Production Subnets (N. Virginia Region)
Go to **Subnets** > **Create subnet** > Select `XYZ-Production-VPC`:
* `Prod-Web-Subnet` (Public) -> CIDR: `10.10.1.0/24` *(Enable auto-assign public IP via Subnet Actions)*
* `Prod-App1-Subnet` (Private) -> CIDR: `10.10.2.0/24`
* `Prod-App2-Subnet` (Private) -> CIDR: `10.10.3.0/24`
* `Prod-DbCache-Subnet` (Private) -> CIDR: `10.10.4.0/24`
* `Prod-Db-Subnet` (Private) -> CIDR: `10.10.5.0/24`

#### 2. Development Subnets
Click **Create subnet** > Select `XYZ-Development-VPC`:
* `Dev-Web-Subnet` (Public) -> CIDR: `10.20.1.0/24` *(Enable auto-assign public IP)*
* `Dev-Db-Subnet` (Private) -> CIDR: `10.20.2.0/24`

#### 3. Attach Internet Gateways (IGW)
* Create `Prod-IGW` -> Attach to `XYZ-Production-VPC`.
* Create `Dev-IGW` -> Attach to `XYZ-Development-VPC`.

#### 4. Deploy NAT Gateway for Production Outbound (Task 4 Requirement)
1. Go to **Elastic IPs** > **Allocate Elastic IP address**.
2. Go to **NAT Gateways** > **Create NAT gateway**.
   * **Name:** `Prod-NAT-GW`
   * **Subnet:** Select `Prod-Web-Subnet` (Public).
   * **Allocation ID:** Select the allocated Elastic IP.
3. Click **Create NAT gateway**.

---

### Step 3: Establish Routing Tables

#### 1. Production Routing
* **Public Route Table (`Prod-Public-RT`):** Associate with `Prod-Web-Subnet`. Add Route: `0.0.0.0/0` -> Target: `Prod-IGW`.
* **Private NAT Route Table (`Prod-Private-NAT-RT`):** Associate with `Prod-App1-Subnet` and `Prod-DbCache-Subnet`. Add Route: `0.0.0.0/0` -> Target: `Prod-NAT-GW` (Fulfills internet requests requirement).
* **Isolated Private Route Table (`Prod-Isolated-RT`):** Associate with `Prod-App2-Subnet` and `Prod-Db-Subnet` (No internet route).

#### 2. Development Routing
* **Dev Public Route Table (`Dev-Public-RT`):** Associate with `Dev-Web-Subnet`. Add Route: `0.0.0.0/0` -> Target: `Dev-IGW`.
* **Dev Private Route Table (`Dev-Private-RT`):** Associate with `Dev-Db-Subnet` (No default route to internet, keeping it isolated).

---

### Step 4: Launch Compute Workloads (7 Instances)
Navigate to **EC2 Dashboard** > **Launch instances**. Use `t2.micro` (Ubuntu) for all. Match Security Groups to tier requirements.

1. **In Production VPC:**
   * Launch host `web` in `Prod-Web-Subnet` (Attach SG allowing HTTP/SSH).
   * Launch host `app1` in `Prod-App1-Subnet`.
   * Launch host `app2` in `Prod-App2-Subnet`.
   * Launch host `dbcache` in `Prod-DbCache-Subnet`.
   * Launch host `db` in `Prod-Db-Subnet`.
2. **In Development VPC:**
   * Launch host `web` in `Dev-Web-Subnet`.
   * Launch host `db` in `Dev-Db-Subnet`.

---

### Step 5: Configure Security Groups & NACLs
1. **Security Groups:** Configure `Prod-Db-SG` to allow internal database traffic (Port 3306 or 5432) only from `Prod-App1-SG`, `Prod-App2-SG`, and the incoming Dev Peering block.
2. **Network ACLs:** Create a custom NACL for the Production Database Subnet to explicitly allow inbound traffic from the Dev Database subnet range (`10.20.2.0/24`) and block everything else except production app layers.

---

### Step 6: Create VPC Peering and Cross-Database Connection

#### 1. Establish Peering Connection
1. Go to **VPC Dashboard** > **Peering connections** > **Create peering connection**.
   * **Name:** `Prod-To-Dev-Peering`
   * **VPC ID (Requester):** `XYZ-Production-VPC`
   * **VPC ID (Accepter):** `XYZ-Development-VPC`
2. Click **Create**, then select the connection, click **Actions** > **Accept request**.

#### 2. Update Database Cross-Routing (Task 4 Requirement)
To link the database subnets privately over the peering tunnel, modify the route tables as follows:
* In **Production VPC** (Open the route table attached to `Prod-Db-Subnet`):
  * Add Route -> Destination: `10.20.2.0/24` (Dev DB Subnet) | Target: **Peering Connection** (`Prod-To-Dev-Peering`).
* In **Development VPC** (Open the route table attached to `Dev-Db-Subnet`):
  * Add Route -> Destination: `10.10.5.0/24` (Prod DB Subnet) | Target: **Peering Connection** (`Prod-To-Dev-Peering`).

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent massive billing accumulation due to active cross-regional nodes and running NAT systems, clear assets in this exact order immediately:

### 1. Terminate All 7 EC2 Instances
1. Navigate to **EC2 Dashboard** > **Instances**.
2. Select all instances running across both Production and Development tags.
3. Click **Instance state** > **Terminate instance** and confirm permanent cleanup.

### 2. Remove the NAT Gateway & Release IP
1. Go to **VPC Console** > **NAT Gateways**.
2. Select `Prod-NAT-GW`, click **Actions** > **Delete NAT gateway**, and type `delete` to confirm.
3. Go to **Elastic IPs**, select the allocated IP, click **Actions** > **Release Elastic IP address** once the NAT status registers as *Deleted*.

### 3. Delete VPC Peering Connection
1. Navigate to **Peering connections**.
2. Select `Prod-To-Dev-Peering`, click **Actions** > **Delete peering connection**, check the boxes to clear related route table rules, and confirm.

### 4. Delete VPCs (Wipes Subnets, Custom Tables, and IGWs automatically)
1. Go to **Your VPCs**.
2. Select `XYZ-Production-VPC` > click **Actions** > **Delete VPC** > Confirm.
3. Select `XYZ-Development-VPC` > click **Actions** > **Delete VPC** > Confirm.
