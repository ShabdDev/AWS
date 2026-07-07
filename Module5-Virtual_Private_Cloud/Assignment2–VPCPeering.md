# Module 5: Assignment 2 - Intra-Region and Inter-Region VPC Peering

## Problem Statement
XYZ Corporation requires isolated, secure networking environments across different geographic locations to facilitate seamless internal data communication. Implement three distinct Virtual Private Clouds (VPCs) across two AWS regions (N. Virginia and Oregon) and build secure VPC Peering connections to enable direct private routing between them.

---

## Tasks To Be Performed
1. Create 2 VPCs in the North Virginia (`us-east-1`) region named `MYVPC1` and `MYVPC2`.
2. Create 1 VPC in the Oregon (`us-west-2`) region named `VPCOregon1`.
3. Create an Intra-Region peering connection between `MYVPC1` and `MYVPC2`.
4. Create an Inter-Region peering connection between `MYVPC2` and `VPCOregon1`.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision VPCs in N. Virginia (us-east-1)
1. Switch your AWS Management Console region to **N. Virginia (us-east-1)**.
2. Open the **VPC Dashboard** > click **Your VPCs** > **Create VPC**.
3. **Configure MYVPC1:**
   * **Resources to create:** `VPC only`
   * **Name tag:** `MYVPC1`
   * **IPv4 CIDR block:** `10.1.0.0/16`
   * Click **Create VPC**.
4. **Configure MYVPC2:**
   * Click **Create VPC** again.
   * **Name tag:** `MYVPC2`
   * **IPv4 CIDR block:** `10.2.0.0/16` *(Note: CIDR ranges must not overlap for peering to work).*
   * Click **Create VPC**.

---

### Step 2: Provision VPC in Oregon (us-west-2)
1. Switch your AWS Console region dropdown (top-right corner) to **Oregon (us-west-2)**.
2. Go to **VPC Dashboard** > **Your VPCs** > **Create VPC**.
3. **Configure VPCOregon1:**
   * **Resources to create:** `VPC only`
   * **Name tag:** `VPCOregon1`
   * **IPv4 CIDR block:** `10.3.0.0/16`
   * Click **Create VPC**.

---

### Step 3: Establish Peering Connection 1 (MYVPC1 <-> MYVPC2)
1. Switch back to the **N. Virginia (us-east-1)** region.
2. In the left panel under *Virtual Private Cloud*, click on **Peering connections** > **Create peering connection**.
3. Configure the local connection metadata:
   * **Peering connection name tag:** `Peering-Local-1to2`
   * **VPC ID (Requester):** Select `MYVPC1`.
   * **Account:** My account (Local).
   * **Region:** This region (`us-east-1`).
   * **VPC ID (Accepter):** Select `MYVPC2`.
4. Click **Create peering connection**.
5. Select the newly created peering connection (`Peering-Local-1to2`), click **Actions** > **Accept request**, and confirm.

---

### Step 4: Establish Peering Connection 2 (MYVPC2 <-> VPCOregon1)
1. Remain in the **N. Virginia (us-east-1)** region.
2. Go to **Peering connections** > **Create peering connection**.
3. Configure the Inter-Region metadata:
   * **Peering connection name tag:** `Peering-InterRegion-2toOregon`
   * **VPC ID (Requester):** Select `MYVPC2`.
   * **Account:** My account.
   * **Region:** Select **Another region** and choose **US West (Oregon) (us-west-2)**.
   * **VPC ID (Accepter):** Paste the exact VPC ID of `VPCOregon1` (Copy it from your Oregon console tab).
4. Click **Create peering connection**.
5. Switch your console region to **Oregon (us-west-2)**.
6. Go to **Peering connections**, select the incoming `Peering-InterRegion-2toOregon` request, click **Actions** > **Accept request**, and confirm.

---

### Step 5: Update Route Tables (Crucial Step for Active Routing)
To allow data to flow, you must manually add routes pointing to the peering connection targets in your route tables for all VPCs:
* **In N. Virginia for MYVPC1:** Add a route to destination `10.2.0.0/16` targeting Peering Connection `Peering-Local-1to2`.
* **In N. Virginia for MYVPC2:** * Add a route to destination `10.1.0.0/16` targeting Peering Connection `Peering-Local-1to2`.
  * Add a route to destination `10.3.0.0/16` targeting Peering Connection `Peering-InterRegion-2toOregon`.
* **In Oregon for VPCOregon1:** Add a route to destination `10.2.0.0/16` targeting Peering Connection `Peering-InterRegion-2toOregon`.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To remove configurations and ensure no stale peering route components remain across your regional resources, complete the tear-down in this sequence:

### 1. Delete Peering Connections (Removes cross-links automatically)
1. In the **N. Virginia** console, navigate to **Peering connections**.
2. Select `Peering-Local-1to2`, click **Actions** > **Delete peering connection**, check the cleanup boxes, and confirm.
3. Select `Peering-InterRegion-2toOregon`, click **Actions** > **Delete peering connection**, check the cleanup boxes, and confirm.

### 2. Delete VPCs in N. Virginia
1. Go to **Your VPCs** in N. Virginia.
2. Select `MYVPC1` > click **Actions** > **Delete VPC** > Confirm.
3. Select `MYVPC2` > click **Actions** > **Delete VPC** > Confirm.

### 3. Delete VPC in Oregon
1. Switch your console to the **Oregon** region.
2. Go to **Your VPCs**, select `VPCOregon1` > click **Actions** > **Delete VPC** > Confirm.
