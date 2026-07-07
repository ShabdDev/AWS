# Module 3: Assignment 4 - CloudWatch Dashboard Configuration

## Problem Statement
XYZ Corporation requires an operational monitoring solution to keep track of system health, detect performance anomalies, and monitor virtual machine resources for misconfigurations. Implement an Amazon CloudWatch Dashboard to aggregate and visualize core host metrics, specifically CPU utilization and network activity, for a target EC2 instance.

---

## Tasks To Be Performed
1. Create a CloudWatch Dashboard that visualizes the following metrics for a specific EC2 instance:
   * CPU Utilization
   * Network In / Network Out traffic metrics

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Pre-requisite (Ensure an EC2 Instance is Running)
*Note: CloudWatch needs an active resource to populate metrics. Ensure you have at least one EC2 instance running (e.g., from your previous Module 2 assignments).*
1. Note down the **Instance ID** of your running EC2 instance (e.g., `i-0123456789abcdef0`).

### Step 2: Create a CloudWatch Dashboard
1. Log in to the **AWS Management Console** and search for **CloudWatch**.
2. In the left-hand navigation pane, click on **Dashboards** under the *Alarms* or *Metrics* section.
3. Click on the **Create dashboard** button.
4. **Configure Dashboard Metadata:**
   * **Dashboard name:** `XYZ-EC2-Monitoring-Dashboard`
   * Click **Create dashboard**.

### Step 3: Add CPU Utilization Widget
1. After creating the dashboard, an **Add widget** screen will pop up automatically.
2. Select **Line** as the data visualization type, then click **Next**.
3. In the metric data source explorer, click on **Metrics**.
4. Navigate through the categories: **EC2** > **Per-Instance Metrics**.
5. Paste your running instance ID into the search bar.
6. Look for the metric named **CPUUtilization**, check the box next to it.
7. Change the graph title (located at the top of the widget view) to `EC2 Host CPU Utilization (%)`.
8. Click **Create widget**. The CPU metric graph is now pinned to your dashboard grid.

### Step 4: Add Network I/O Performance Widget
1. Click on the **Add widget** icon (`+`) at the top right of your custom dashboard panel.
2. Select **Line** as the graph layout type and click **Next**.
3. Under the **Metrics** tab, go back to **EC2** > **Per-Instance Metrics**.
4. Filter by your target Instance ID.
5. Search for and check the boxes next to two distinct performance metrics:
   * **NetworkIn**
   * **NetworkOut**
6. Modify the graph title header to read `Network I/O Traffic (Bytes)`.
7. Click **Create widget**.
8. Rearrange or resize the newly added graph side-by-side with your CPU graph as desired.
9. Click the **Save** button in the top banner to commit the dashboard template.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

While 3 CloudWatch dashboards are free per account, deleting stale assets is recommended to keep clean workspace quotas:

### 1. Delete the CloudWatch Dashboard
1. Navigate to the **CloudWatch Dashboard** list menu view.
2. Check the box next to `XYZ-EC2-Monitoring-Dashboard`.
3. Click the **Delete** button positioned in the upper right control action segment.
4. Confirm by clicking **Delete** on the pop-up warning module to clean your environment.
