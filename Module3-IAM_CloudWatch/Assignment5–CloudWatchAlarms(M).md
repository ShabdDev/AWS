# Module 3: Assignment 5 - CloudWatch Alarms and SNS Notifications

## Problem Statement
XYZ Corporation requires automated monitoring safeguards to prevent budget overruns and track virtual machine performance anomalies. Implement a CloudWatch Billing Alarm to trigger when estimated charges exceed a defined budget limit ($500), and configure a Resource Metric Alarm to notify administrators via Amazon SNS when an EC2 instance's CPU utilization surpasses 65%.

---

## Tasks To Be Performed
1. Create a CloudWatch billing alarm which goes off when the estimated charges go above $500.
2. Create a CloudWatch alarm which goes off to an Alarm state when the CPU utilization of an EC2 instance goes above 65%. Also add an SNS topic so that it notifies the person when the threshold is crossed.

---

## Part 1: Step-by-Step Implementation Solution

### Task 1: Create a CloudWatch Billing Alarm ($500 Threshold)

*Note: Billing metrics are stored in the US East (N. Virginia) `us-east-1` region. Ensure you are in this region to configure the alarm.*

#### Step 1: Enable Billing Alerts
1. Log in to the **AWS Management Console** with Root or Admin credentials.
2. Search for **Billing** and go to the Billing Dashboard.
3. In the left panel, click on **Billing preferences** (or Billing settings).
4. Check the box for **Receive Billing Alerts** and click **Save preferences**.

#### Step 2: Configure the Billing Alarm
1. Navigate to the **CloudWatch** console. Switch region to **N. Virginia (us-east-1)**.
2. In the left pane, click **Alarms** > **All alarms** and click **Create alarm**.
3. Click **Select metric** > **Billing** > **Total Estimated Charges**.
4. Check the box for **EstimatedCharges** (USD) and click **Select metric**.
5. Define the Alarm conditions:
   * **Threshold type:** `Static`
   * **Whenever EstimatedCharges is...** `Greater than`
   * **Define the threshold value:** `500`
6. Click **Next**.
7. In the **Configure actions** step, set the status notification trigger to **In alarm**.
8. Create a new SNS topic named `XYZ-Billing-Alert-Topic` and enter your corporate email address. Click **Create topic**.
9. Click **Next**, name the alarm `XYZ-Estimated-Charges-Above-500`, review the setup, and click **Create alarm**.
10. *Crucial:* Check your email inbox and click **Confirm subscription** in the AWS validation email.

---

### Task 2: Create EC2 CPU Utilization Alarm with SNS

#### Step 1: Create an SNS Topic for Infrastructure Alerts
1. Search for and open the **Simple Notification Service (SNS)** console.
2. Click **Topics** > **Create topic**.
   * **Type:** `Standard`
   * **Name:** `XYZ-EC2-Alerts-Topic`
3. Click **Create topic**.
4. Inside the newly created topic, click **Create subscription**.
   * **Protocol:** `Email`
   * **Endpoint:** Enter your personal/work email address.
5. Click **Create subscription** and confirm the subscription via the link sent to your email.

#### Step 2: Create the CPU Utilization Metric Alarm
1. Navigate back to the **CloudWatch** console.
2. Go to **Alarms** > **All alarms** and click **Create alarm**.
3. Click **Select metric** > **EC2** > **Per-Instance Metrics**.
4. Select your active running EC2 Instance ID and look for the **CPUUtilization** metric. Check the box and click **Select metric**.
5. Configure conditions:
   * **Statistic:** `Average`
   * **Period:** `5 minutes` (or 1 minute for faster testing)
   * **Threshold type:** `Static`
   * **Whenever CPUUtilization is...** `Greater than`
   * **Define threshold value:** `65`
6. Click **Next**.
7. Under **Notification** -> Alarm state trigger: **In alarm**.
8. Select **An existing SNS topic** and choose `XYZ-EC2-Alerts-Topic`.
9. Click **Next**. Name the alarm `XYZ-EC2-CPU-Exceeds-65`.
10. Review configurations and click **Create alarm**.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To clear redundant alerts and safely remove non-production notification paths, complete the deletion workflow in this order:

### 1. Delete CloudWatch Alarms
1. Open the **CloudWatch** dashboard -> **Alarms** > **All alarms**.
2. Select the checkbox next to `XYZ-Estimated-Charges-Above-500` and `XYZ-EC2-CPU-Exceeds-65`.
3. Click **Actions** > **Delete** and confirm the removal.

### 2. Delete Simple Notification Service (SNS) Topics
1. Open the **Amazon SNS** console -> **Topics**.
2. Select `XYZ-Billing-Alert-Topic` and `XYZ-EC2-Alerts-Topic`.
3. Click the **Delete** button.
4. Type `delete me` (or follow the screen prompt) and click confirm.
