
# Module 2 - Assignment 1: Web Server Deployment on AWS EC2

## Problem Statement
You work for XYZ Corporation. Your corporation wants to launch a new web-based application using AWS Virtual Machines. Configure the resources accordingly for the tasks.

---

## Tasks To Be Performed
1. Create an instance in the US-East-1 (N. Virginia) region with an Ubuntu OS and install Nginx for making them web servers.
2. Change the default website with a page displaying the message: “Hello World”

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Launch Ubuntu EC2 Instance in US-East-1
1. Log in to the **AWS Management Console**.
2. In the top-right corner, select the region **US East (N. Virginia) `us-east-1`**.
3. Navigate to the **EC2 Dashboard** and click **Launch Instance**.
4. Configure the instance details as follows:
   * **Name:** `XYZ-Web-Server`
   * **Application and OS Image (AMI):** Select **Ubuntu** (Choose *Ubuntu Server 24.04 LTS or 22.04 LTS - Free Tier Eligible*).
   * **Instance Type:** `t2.micro` (Free Tier eligible).
   * **Key Pair:** Select an existing key pair or click *Create new key pair* to download the `.pem` file for SSH access.
   * **Network Settings:**
     * Keep default VPC and Subnet settings.
     * Auto-assign public IP: **Enable**.
     * **Security Group:** Select *Create security group*. Name it `XYZ-Web-SG`.
       * Add Inbound Rule 1: **SSH (Port 22)** | Source: `My IP` (For secure terminal access).
       * Add Inbound Rule 2: **HTTP (Port 80)** | Source: `0.0.0.0/0` (Anywhere - to allow public web traffic).

5. Review the details and click **Launch Instance**.

### Step 2: Connect to the Instance and Install Nginx
1. Once the instance state changes to **Running**, select the instance from the list and click **Connect** at the top.
2. Choose the **EC2 Instance Connect** tab and click **Connect** (This opens a secure terminal directly in your browser).
3. Once the terminal is ready, update the package manager repository by running the following command:
   ```bash
   sudo apt update
   ```

4. Install the Nginx web server by executing:
```bash
sudo apt install nginx -y

```

5. Verify that the Nginx service is active and running successfully:
```bash
sudo systemctl status nginx

```

### Step 3: Update Default Website to Display "Hello World"

1. Navigate to the Nginx default directory and overwrite the default `index.html` file to display the custom message:
```bash
echo "<h1>Hello World</h1>" | sudo tee /var/www/html/index.html

```


2. Restart the Nginx service to apply the updates:
```bash
sudo systemctl restart nginx

```


3. Go back to the AWS EC2 Console and copy the **Public IPv4 address** of your `XYZ-Web-Server` instance.
4. Open a new web browser tab, paste the copied IP address (e.g., `http://<your-instance-public-ip>`), and press Enter.
5. The webpage will successfully display the output: **Hello World**.

---

## Part 2: Step-by-Step Deletion Process (Cost Optimization)

To maintain standard hygiene of the AWS Free Tier account and avoid unnecessary storage accumulation, terminate the created infrastructure by following these steps:

1. Open the **EC2 Dashboard** and click on **Instances (running)**.
2. Select the checkbox next to `XYZ-Web-Server`.
3. Click on the **Instance State** dropdown menu at the top and select **Terminate instance**.
4. Click **Terminate** in the confirmation pop-up window to confirm.
5. *(This action permanently deletes the virtual machine, releases its Public IP, and automatically deletes the attached root EBS volume).*
