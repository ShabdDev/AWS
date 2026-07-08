# Module 3 - Introduction to IAM and CloudWatch

# Assignment 4 – CloudWatch Dashboard

## Problem Statement

You work for XYZ Corporation. To maintain the security of the AWS account and the resources you have been asked to implement a solution that can help easily recognize and monitor the different users. Also, you will be monitoring the machines created by these users for any errors or misconfigurations.

## Tasks To Be Performed

1. Create a CloudWatch Dashboard that lets you monitor:
   - CPU Utilization
   - Network traffic
   for a particular Amazon EC2 instance.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|--------------------|--------------|-------|
| Amazon EC2 (`t2.micro` / `t3.micro` depending on region) | Yes | No (within Free Tier limits) | Used to generate metrics for CloudWatch. |
| Amazon CloudWatch (Basic Monitoring) | Yes | No | EC2 instances automatically send basic metrics every 5 minutes. |
| CloudWatch Dashboard | Partially | Yes (after Free Tier limits) | AWS provides a limited number of dashboard usage benefits. Additional dashboards or prolonged usage may incur a small monthly charge. |
| IAM | Yes | No | Required to access AWS resources securely. |

### Free Tier Eligibility

This assignment is **mostly Free Tier eligible**.

### Services That May Consume Credits

- CloudWatch Dashboards may incur a very small monthly charge depending on your AWS account usage and current AWS pricing.
- If your Free Tier has expired, the EC2 instance will also consume credits while it is running.

### Recommendation

Since you currently have AWS promotional credits, it is recommended to complete this assignment while those credits are available. Any CloudWatch Dashboard charges (if applicable) will be covered by your credits.

---

# Architecture Diagram

```text
                 +----------------------+
                 |      IAM User        |
                 +----------+-----------+
                            |
                            |
                    AWS Management Console
                            |
                            |
                +-----------v-----------+
                |     Amazon EC2        |
                |      Instance         |
                +-----------+-----------+
                            |
              CPU / Network Metrics
                            |
                            |
                +-----------v-----------+
                |   Amazon CloudWatch   |
                |      Dashboard        |
                +-----------------------+
```

---

# Dependency Flow

```text
Login to AWS
        │
        ▼
Create EC2 Instance
        │
        ▼
Wait until Instance is Running
        │
        ▼
CloudWatch Automatically Collects Metrics
        │
        ▼
Create Dashboard
        │
        ▼
Add CPU Widget
        │
        ▼
Add Network Widget
        │
        ▼
Verify Dashboard
```

---

# Step 1: Sign in to the AWS Management Console

## Navigation

```text
AWS Console
```

## Configuration

Sign in using your AWS account credentials.

No configuration changes are required in this step.

## Why is this step required?

All AWS resources are created and managed through the AWS Management Console (or AWS CLI). Logging in is the first prerequisite before performing any AWS operation.

## Dependency

None.

This is the starting point of the assignment.

## What happens if this step is skipped?

You will not be able to access any AWS services, including EC2 and CloudWatch.

---

# Step 2: Launch an Amazon EC2 Instance

> **Note:** CloudWatch can only display metrics for an existing EC2 instance. If you already have a running EC2 instance that you want to monitor, you may use it. Otherwise, create a new one by following this step.

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Launch Instances
```

## Configuration

| Setting | Value |
|---------|-------|
| **Instance Name** | `CloudWatch-EC2` |
| **AMI** | `Amazon Linux 2023` |
| **Instance Type** | `t2.micro` (or `t3.micro` if Free Tier eligible in your region) |
| **Key Pair** | Create a new key pair or use an existing one |
| **Network** | Default VPC |
| **Subnet** | Default subnet |
| **Auto-assign Public IP** | `Enabled` |
| **Security Group Name** | `CloudWatch-SG` |
| **Inbound Rule** | `SSH (22)` from your IP |
| **Storage** | `8 GiB gp3` |

## Why is this step required?

Amazon CloudWatch receives monitoring metrics from AWS resources.

Without an EC2 instance, there will be no CPU or network metrics available to display on the dashboard.

## Dependency

Depends on:

- Successful AWS login.

## What happens if this step is skipped?

No EC2 instance will exist.

As a result:

- No CPU Utilization metric
- No Network In metric
- No Network Out metric

The dashboard will remain empty because CloudWatch has no resource to monitor.

---

---

# Step 3: Wait for CloudWatch Metrics to Become Available

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select your EC2 instance
→ Monitoring
```

## Configuration

No configuration is required.

Simply wait until the EC2 instance reaches the **Running** state.

CloudWatch automatically starts collecting **basic monitoring metrics** for every EC2 instance.

> **Note:** It may take approximately **5–10 minutes** for the first metrics to appear in CloudWatch after the instance starts.

## Why is this step required?

CloudWatch Dashboard displays metrics that already exist.

Immediately after launching an EC2 instance, there are no historical metrics available.

Waiting for CloudWatch to collect the first set of metrics ensures that graphs will contain data instead of showing **No data available**.

## Dependency

Depends on:

- Step 1 (AWS Login)
- Step 2 (EC2 Instance Running)

## What happens if this step is skipped?

When creating the dashboard:

- Widgets may appear empty.
- Graphs may display **Insufficient Data** or **No Data**.
- You may incorrectly think the dashboard configuration is wrong.

---

# Step 4: Open Amazon CloudWatch

## Navigation

```text
AWS Console
→ CloudWatch
```

## Configuration

No configuration changes are required in this step.

## Why is this step required?

Amazon CloudWatch is the AWS monitoring service responsible for:

- Collecting metrics
- Monitoring AWS resources
- Creating alarms
- Building dashboards
- Visualizing resource performance

To create a monitoring dashboard, we must first open the CloudWatch service.

## Dependency

Depends on:

- EC2 instance being created.
- CloudWatch receiving metrics from the instance.

## What happens if this step is skipped?

You cannot create or manage CloudWatch Dashboards.

---

# Step 5: Create a CloudWatch Dashboard

## Navigation

```text
AWS Console
→ CloudWatch
→ Dashboards
→ Create Dashboard
```

## Configuration

| Setting | Value |
|----------|-------|
| **Dashboard Name** | `EC2-Monitoring-Dashboard` |

Click **Create Dashboard**.

## Why is this step required?

A CloudWatch Dashboard provides a centralized view of AWS resource metrics.

Instead of opening each EC2 instance separately, administrators can monitor important performance indicators from a single screen.

This improves operational visibility and simplifies infrastructure monitoring.

## Dependency

Depends on:

- CloudWatch service.
- Existing EC2 metrics.

## What happens if this step is skipped?

CloudWatch metrics will still exist, but there will be no customized dashboard to visualize them together.

---

# Step 6: Add CPU Utilization Widget

## Navigation

```text
CloudWatch
→ Dashboards
→ EC2-Monitoring-Dashboard
→ Add Widget
→ Line
```

## Configuration

| Setting | Value |
|----------|-------|
| **Widget Type** | `Line` |
| **Metric Source** | `EC2` |
| **Metric** | `CPUUtilization` |
| **Statistic** | `Average` |
| **Period** | `5 Minutes` |
| **Instance ID** | Select your EC2 instance |
| **Widget Title** | `CPU Utilization` |

Click **Create Widget**.

## Why is this step required?

CPU Utilization indicates how much processing power is currently being used.

A consistently high CPU utilization may indicate:

- High workload
- Performance bottlenecks
- Need for a larger instance
- Application issues

A consistently low CPU utilization may indicate:

- Over-provisioned infrastructure
- Underutilized resources

Monitoring CPU helps administrators optimize both performance and cost.

## Dependency

Depends on:

- Dashboard creation.
- Existing EC2 metrics.

## What happens if this step is skipped?

The dashboard will not display CPU performance.

You will not be able to identify CPU spikes or excessive processor utilization.

---

# Step 7: Add Network Monitoring Widget

## Navigation

```text
CloudWatch
→ Dashboards
→ EC2-Monitoring-Dashboard
→ Add Widget
→ Line
```

## Configuration

| Setting | Value |
|----------|-------|
| **Widget Type** | `Line` |
| **Metric Source** | `EC2` |
| **Metrics** | `NetworkIn`, `NetworkOut` |
| **Statistic** | `Average` |
| **Period** | `5 Minutes` |
| **Instance ID** | Select your EC2 instance |
| **Widget Title** | `Network Traffic` |

Click **Create Widget**.

## Why is this step required?

Network metrics show how much data is entering and leaving the EC2 instance.

These metrics help detect:

- Heavy application traffic
- Unexpected network activity
- Possible attacks
- Bandwidth usage
- Idle servers

Monitoring network traffic is an important part of server health monitoring.

## Dependency

Depends on:

- Existing dashboard.
- EC2 metrics being available.

## What happens if this step is skipped?

The dashboard will only display CPU information.

You will not be able to monitor network utilization, making troubleshooting more difficult.

---
---

# Verification Step 1: Verify That the EC2 Instance Is Running

## Action

Verify that the EC2 instance is in the **Running** state.

## Navigation

```text
AWS Console
→ EC2
→ Instances
```

## Expected Output

The EC2 instance should display:

| Property | Expected Value |
|----------|----------------|
| **Instance State** | `Running` |
| **Status Check** | `2/2 checks passed` |

## Why does this confirm success?

CloudWatch can only collect monitoring metrics from a running EC2 instance. If the instance is stopped or terminated, no new metrics will be generated.

---

# Verification Step 2: Verify That CloudWatch Metrics Are Available

## Action

Open the Monitoring tab for the EC2 instance.

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select your EC2 instance
→ Monitoring
```

## Expected Output

You should observe graphs for metrics such as:

- CPU Utilization
- Network In
- Network Out
- Disk Read Bytes
- Disk Write Bytes
- Network Packets In
- Network Packets Out

The graphs should contain plotted data points instead of displaying **No data available**.

## Why does this confirm success?

This confirms that CloudWatch is successfully collecting performance metrics from the EC2 instance. These metrics are the data source used by the CloudWatch Dashboard.

---

# Verification Step 3: Verify the CloudWatch Dashboard

## Action

Open the CloudWatch Dashboard created during this assignment.

## Navigation

```text
AWS Console
→ CloudWatch
→ Dashboards
→ EC2-Monitoring-Dashboard
```

## Expected Output

The dashboard should contain at least two widgets:

| Widget | Expected Result |
|---------|-----------------|
| **CPU Utilization** | Displays a line graph showing CPU usage over time |
| **Network Traffic** | Displays line graphs for `NetworkIn` and `NetworkOut` |

The graphs should update automatically as CloudWatch receives new metrics.

## Why does this confirm success?

This verifies that the dashboard has been created correctly and is connected to the selected EC2 instance's CloudWatch metrics.

---

# Verification Step 4: Generate CPU Activity (Optional)

## Action

Connect to the EC2 instance and generate CPU load.

## Command

```bash
yes > /dev/null
```

### Command Explanation

| Command Part | Explanation |
|--------------|-------------|
| `yes` | Continuously prints the character `y` followed by a newline. |
| `>` | Redirects the command output to another destination instead of displaying it on the terminal. |
| `/dev/null` | A special Linux device that discards all data written to it. It prevents the terminal from filling with output. |

## Expected Output

No output is displayed because the command continuously runs in the foreground while sending all output to `/dev/null`.

After a few minutes:

- CPU Utilization graph should increase noticeably.
- CloudWatch Dashboard should reflect the increased CPU usage.

To stop the command, press:

```text
Ctrl + C
```

## Why does this confirm success?

The increased CPU utilization demonstrates that CloudWatch is collecting real-time performance metrics and updating the dashboard accordingly.

---

# Cleanup / Cost Optimization

> **Important:** After verifying the assignment, delete all created resources to avoid unnecessary AWS charges and preserve your promotional credits.

---

# Cleanup Step 1: Delete the CloudWatch Dashboard

## Navigation

```text
AWS Console
→ CloudWatch
→ Dashboards
→ EC2-Monitoring-Dashboard
```

## Action

1. Select the dashboard.
2. Click **Delete**.
3. Confirm the deletion.

## Why is this required?

The dashboard is no longer needed after verification. Removing unused dashboards keeps the AWS environment organized and helps avoid dashboard-related charges where applicable.

## What happens if this resource is not deleted?

- The dashboard remains in your AWS account.
- Depending on your account usage and AWS pricing, dashboard usage may contribute to small ongoing charges.

---

# Cleanup Step 2: Terminate the EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select CloudWatch-EC2
→ Instance State
→ Terminate Instance
```

## Action

Terminate the EC2 instance created for this assignment.

## Why is this required?

Amazon EC2 is a compute service billed while resources are active (outside Free Tier limits). Terminating the instance stops compute charges and releases the associated virtual machine.

## What happens if this resource is not deleted?

- The EC2 instance may continue consuming compute resources.
- Charges may apply if your Free Tier has expired or usage exceeds Free Tier limits.
- CloudWatch will continue receiving metrics from the running instance.

---

# Cleanup Step 3: Delete the Security Group (If It Was Created Only for This Assignment)

## Navigation

```text
AWS Console
→ EC2
→ Security Groups
```

## Action

Delete the security group named:

`CloudWatch-SG`

> Delete it only if it is not attached to any other AWS resource.

## Why is this required?

Unused security groups increase account clutter and may accidentally be reused with incorrect firewall rules in future deployments.

## What happens if this resource is not deleted?

- No direct charges are incurred.
- Unused security groups remain in the account, making resource management more difficult.

---

# Cleanup Step 4: Delete the Key Pair (Optional)

## Navigation

```text
AWS Console
→ EC2
→ Key Pairs
```

## Action

Delete the key pair if it was created exclusively for this assignment and will not be used again.

## Why is this required?

Removing unused key pairs helps maintain good security practices by reducing the number of credentials stored in your AWS account.

## What happens if this resource is not deleted?

- No AWS charges are incurred.
- The key pair remains available and could be mistakenly used for future instances.

---

---

# Cleanup Dependency Order

The resources created during this assignment should be deleted in the following order:

```text
CloudWatch Dashboard
        │
        ▼
Terminate EC2 Instance
        │
        ▼
Delete Security Group (if unused)
        │
        ▼
Delete Key Pair (optional)
```

## Why is the deletion order important?

AWS resources often have dependencies on one another. Deleting them in the correct sequence helps prevent dependency errors and ensures that all resources are removed successfully.

- **CloudWatch Dashboard** should be deleted first because it references the EC2 instance metrics.
- **EC2 Instance** should be terminated before deleting networking-related resources.
- **Security Group** cannot be deleted if it is still attached to any running instance.
- **Key Pair** can be deleted at the end because it is not actively attached to the terminated instance.

Deleting resources in this order ensures a clean and organized AWS environment while minimizing the risk of dependency-related issues.

## What happens if resources are not deleted?

| Resource | Possible Impact |
|----------|-----------------|
| **CloudWatch Dashboard** | May contribute to CloudWatch dashboard charges depending on AWS pricing and account usage. |
| **EC2 Instance** | May continue consuming compute resources and incur charges if outside Free Tier limits. |
| **Security Group** | No direct cost, but increases account clutter and may lead to configuration confusion. |
| **Key Pair** | No direct cost, but leaving unused credentials is not a security best practice. |

---
