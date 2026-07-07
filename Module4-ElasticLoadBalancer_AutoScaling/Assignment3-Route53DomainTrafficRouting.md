# Module 4: Assignment 3 - Route 53 Domain Traffic Routing

## Problem Statement
XYZ Corporation is optimizing its cloud infrastructure and requires an active Domain Name System (DNS) management layer to route user requests seamlessly. Implement Amazon Route 53 to configure a Public Hosted Zone and map a custom domain or subdomain directly to an active EC2 Web Server instance using its Public IPv4 address.

---

## Tasks To Be Performed
1. Use the Route 53 Hosted Zone created in the assignment.
2. Route the traffic to an EC2 instance with an Apache web server running in it using its IP address.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Pre-requisite (Ensure EC2 Web Server with Public IP is Ready)
1. Navigate to the **EC2 Dashboard** and ensure you have an active EC2 instance running (you can use the Apache instance created in Assignment 2, or launch a quick `t2.micro` Ubuntu instance).
2. Ensure the instance's Security Group allows **HTTP (Port 80)** inbound traffic.
3. Note down the **Public IPv4 address** of this instance (e.g., `54.210.43.87`).

---

### Step 2: Create a Route 53 Hosted Zone
*(If you have already created a hosted zone in your previous lab session, you can skip to Step 3. Otherwise, follow these steps to deploy one):*
1. In the AWS Management Console, search for and open **Route 53**.
2. Click on **Hosted zones** in the left sidebar or dashboard panel.
3. Click on the **Create hosted zone** button.
4. Configure Hosted Zone settings:
   * **Domain name:** Enter your custom domain name (e.g., `xyzcorp.com` or any specific training domain given by your instructor).
   * **Type:** Select **Public hosted zone**.
5. Click **Create hosted zone**. (AWS will automatically populate default **NS** (Name Server) and **SOA** records).

---

### Step 3: Create an 'A' Record to Route Traffic to the EC2 IP
1. Inside your custom Hosted Zone configuration page (e.g., `xyzcorp.com`), click the **Create record** button.
2. Configure the Record Routing Metadata:
   * **Record name:** Keep it blank to map the root domain (e.g., `xyzcorp.com`) or enter a subdomain like `www`.
   * **Record type:** Choose **A – Routes traffic to an IPv4 address and some AWS resources**.
   * **Value:** Paste the exact **Public IPv4 address** of your EC2 Web Server noted during Step 1.
   * **TTL (Seconds):** Keep default (`300` seconds).
   * **Routing policy:** Select **Simple routing**.
3. Click **Create records**.

### Step 4: Verification
1. Wait 1-2 minutes for global DNS propagation.
2. Open a standard browser tab and enter your record name (e.g., `http://www.xyzcorp.com` or your specific dummy address).
3. The browser will successfully call the Route 53 DNS namespace server, map the pointer, and fetch your Apache web page.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent AWS Route 53 from generating persistent billing overhead for keeping an allocated public DNS zone file, delete the elements in this exact sequence:

### 1. Delete Custom 'A' Records
1. Open the **Route 53** console -> **Hosted zones**.
2. Click on your custom domain name (`xyzcorp.com`).
3. Select the checkbox next to the **A Record** you created in Step 3.
4. Click the **Delete record** button at the top and confirm the operation. *(Note: Do not delete the default NS and SOA records manually).*

### 2. Delete the Hosted Zone (Highest Priority)
1. Go back to the **Hosted zones** directory menu.
2. Select your domain (`xyzcorp.com`).
3. Click the **Delete zone** control button positioned at the top right of the dashboard view.
4. Type `delete` into the validation module box and click confirm to remove the asset completely.
