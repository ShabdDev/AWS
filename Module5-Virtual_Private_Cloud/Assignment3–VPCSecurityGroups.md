# Module 5: Assignment 3 - EC2 Security Group Isolation (Master-Client Architecture)

## Problem Statement
XYZ Corporation requires strict host-level isolation within its virtual network environment to prevent unauthorized lateral movement. Implement a secure network access control schema using AWS Security Groups where a 'Client' backend EC2 instance is completely isolated from the public internet and can only accept Secure Shell (SSH) management traffic originated directly from a designated 'Master' bastian-style EC2 host.

---

## Tasks To Be Performed
1. Create 2 EC2 instances in any public subnet of any VPC and name them **Master** and **Client**.
2. Using custom Security Groups, ensure that the **Client** instance can *only* be accessed via SSH (Port 22) through the internal/private connection of the **Master** instance.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the Security Group for the Master Instance
1. Log in to the **AWS Management Console** and navigate to the **EC2 Dashboard**.
2. In the left navigation pane, under *Network & Security*, click on **Security Groups**.
3. Click **Create security group**.
4. Configure the Master SG details:
   * **Security group name:** `XYZ-Master-SG`
   * **Description:** Allow SSH from anywhere for administrative entry.
   * **VPC:** Select your Default VPC.
5. **Inbound rules:** Click *Add rule*.
   * **Type:** `SSH` | **Port:** `22` | **Source:** `Anywhere-IPv4` (`0.0.0.0/0`)
6. Click **Create security group**.

---

### Step 2: Create the Restrictive Security Group for the Client Instance
1. Click **Create security group** again.
2. Configure the Client SG details:
   * **Security group name:** `XYZ-Client-SG`
   * **Description:** Isolate host, allow SSH ONLY from the Master Security Group.
   * **VPC:** Select the same Default VPC.
3. **Inbound rules:** Click *Add rule*.
   * **Type:** `SSH` | **Port:** `22`
   * **Source:** Select **Custom** and type/select `XYZ-Master-SG` *(Note: Referencing the security group ID instead of an IP address allows for dynamic, secure multi-host binding).*
4. Click **Create security group**.

---

### Step 3: Launch Master and Client EC2 Instances
Navigate to **EC2 Dashboard** > **Instances** > click **Launch instances**.

1. **Launch Master Host:**
   * **Name:** `Master`
   * **OS (AMI):** Ubuntu Server (Free Tier Eligible).
   * **Instance type:** `t2.micro`.
   * **Key pair:** Select or create your standard `.pem` key pair.
   * **Network settings:** Click *Edit*. Choose your Default VPC, ensure Auto-assign public IP is **Enabled**, and under Firewall select **Select existing security group** -> choose `XYZ-Master-SG`.
   * Click **Launch instance**.

2. **Launch Client Host:**
   * Click **Launch instances** again.
   * **Name:** `Client`
   * **OS (AMI) / Instance type / Key pair:** Keep identical to the Master host setup.
   * **Network settings:** Click *Edit*. Choose the same VPC. Under Firewall select **Select existing security group** -> choose `XYZ-Client-SG`.
   * Click **Launch instance**.

---

### Step 4: Verification (Testing the Security Isolation)
1. Note down the **Public IP** of the `Master` instance and the **Private IP** of the `Client` instance.
2. Open your terminal or Git Bash and connect to the **Master** host via SSH using your key pair file:
   ```bash
   ssh -i your-key.pem ubuntu@<Master_Public_IP>
   ```

3. Once logged into the Master server, you need your key file on this server to hop to the Client.
For secure testing, you can create a temporary key file inside the Master instance:
```bash
nano client-key.pem
# Paste your private key data here, save, and exit (Ctrl+O, Enter, Ctrl+X)
chmod 400 client-key.pem
```


4. Now, attempt to SSH into the Client host from within the Master session using the Client's **Private IP**:
```bash
ssh -i client-key.pem ubuntu@<Client_Private_IP>

```


* **Result:** The connection will succeed instantly.


5. **Negative Testing:** Try to connect directly to the Client host from your local machine's terminal via its public network block. The connection will time out, proving that `XYZ-Client-SG` successfully blocked outside interference.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To avoid infrastructure overhead costs against your virtual components, perform the tear-down in this exact order:

### 1. Terminate Both EC2 Instances (Highest Priority)

1. Go to **EC2 Console** > **Instances**.
2. Select both `Master` and `Client` checkboxes.
3. Click **Instance state** > **Terminate instance**. Confirm permanent removal and wait until the state registers as *Terminated*.

### 2. Remove Custom Security Groups

1. Navigate to **Security Groups** in the left menu sidebar.
2. Select `XYZ-Client-SG`, click **Actions** > **Delete security groups**, and confirm.
3. Select `XYZ-Master-SG`, click **Actions** > **Delete security groups**, and confirm. *(Note: Master SG cannot be deleted until the rule inside Client SG referencing it is removed or the Client SG itself is deleted).*
