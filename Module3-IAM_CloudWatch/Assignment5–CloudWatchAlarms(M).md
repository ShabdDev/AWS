# Module 3 - Introduction to IAM and CloudWatch

# Assignment 5 – CloudWatch Alarms

## Problem Statement

You work for XYZ Corporation. To maintain the security of the AWS account and the resources, you have been asked to implement a solution that can help easily recognize and monitor the different users. Also, you will be monitoring the machines created by these users for any errors or misconfigurations.

## Tasks To Be Performed

1. Create a CloudWatch Billing Alarm which goes off when the estimated AWS charges exceed **$500**.
2. Create a CloudWatch Alarm that changes to the **Alarm** state when the **CPU Utilization** of an EC2 instance exceeds **65%**.
3. Create an **Amazon SNS Topic** and configure the CloudWatch Alarm to send a notification whenever the threshold is crossed.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|--------------------|--------------|-------|
| Amazon CloudWatch | Yes (Basic monitoring and alarms have Free Tier limits) | Possible | A limited number of alarms are included in the AWS Free Tier. Additional alarms are billed. |
| Amazon SNS | Yes | No (within Free Tier limits) | SNS includes a limited number of email notifications every month. |
| Amazon EC2 | Yes | No (if using `t2.micro` or `t3.micro` within Free Tier limits) | Required only for the CPU Utilization alarm. |
| AWS Billing Alarm | Yes | No | Billing metric itself does not incur additional cost. |
| Email Notifications | Yes | No | Email delivery through SNS is covered within the Free Tier limits. |

## Free Tier Conclusion

- This assignment is **mostly Free Tier eligible**.
- The EC2 instance should be created using a **Free Tier eligible instance type** such as `t2.micro` (or `t3.micro` where applicable).
- CloudWatch provides a limited number of alarms under the Free Tier. If your AWS account already contains many alarms, creating additional alarms may consume AWS promotional credits.
- If you still have AWS promotional credits available, it is recommended to complete this assignment before they expire.

---

# Architecture Diagram

```text
                       +------------------------+
                       |       AWS Billing      |
                       +-----------+------------+
                                   |
                      Estimated Charges Metric
                                   |
                                   v
                    +-----------------------------+
                    | Billing Alarm (> $500)      |
                    +--------------+--------------+
                                   |
                                   |
                                   v
                          +----------------+
                          |   SNS Topic    |
                          +-------+--------+
                                  |
                                  |
                                  v
                           Email Notification



             +----------------------+
             |   EC2 Instance       |
             +----------+-----------+
                        |
                  CPU Utilization
                        |
                        v
         +-------------------------------+
         | CPU Alarm (>65%)              |
         +---------------+---------------+
                         |
                         |
                         v
                  +--------------+
                  |  SNS Topic   |
                  +------+-------+
                         |
                         |
                         v
                  Email Notification
```

---

# Resource Dependency Flow

```text
SNS Topic
     │
     ▼
Email Subscription
     │
     ▼
Billing Alarm

Existing EC2 Instance
          │
          ▼
CPU Metric
          │
          ▼
CloudWatch CPU Alarm
          │
          ▼
SNS Notification
```

---

# Prerequisites

Before beginning this assignment, ensure the following resources are available.

| Resource | Required | Purpose |
|----------|----------|---------|
| AWS Account | Yes | Required to access AWS services. |
| Verified Email Address | Yes | Required for SNS email notifications. |
| One Running EC2 Instance | Yes | Required for monitoring CPU Utilization. |
| Billing Access Enabled | Yes | Required for creating Billing Alarms. |

---

# Step 1: Verify Billing Alarm Permissions

## Navigation

```text
AWS Console
→ Billing and Cost Management
→ Billing Preferences
```

## Configuration

Verify that the following setting is enabled.

- **Receive Billing Alerts:** `Enabled`

If it is disabled, enable it and save the changes.

## Why is this step required?

CloudWatch Billing metrics are disabled by default for new AWS accounts.

CloudWatch cannot monitor AWS charges unless billing alerts are enabled.

## Dependency

Depends on:

- AWS Account

## What happens if this step is skipped?

The Billing metric will not appear inside CloudWatch.

As a result, creating a Billing Alarm will not be possible.

---

# Step 2: Create an Amazon SNS Topic

## Navigation

```text
AWS Console
→ SNS
→ Topics
→ Create Topic
```

## Configuration

| Setting | Value |
|---------|-------|
| Type | `Standard` |
| Name | `Billing-CPU-Alerts` |

Leave all remaining settings at their default values.

Select **Create Topic**.

## Why is this step required?

Amazon SNS (Simple Notification Service) is responsible for sending notifications whenever a CloudWatch Alarm changes its state.

Without an SNS Topic, CloudWatch can detect problems but cannot notify anyone.

## Dependency

Depends on:

- AWS Account

## What happens if this step is skipped?

The alarms will still change between **OK**, **Alarm**, and **Insufficient Data** states, but nobody will receive any notification.

---

---

# Step 3: Create an Email Subscription for the SNS Topic

## Navigation

```text
AWS Console
→ SNS
→ Topics
→ Billing-CPU-Alerts
→ Create Subscription
```

## Configuration

| Setting | Value |
|---------|-------|
| Protocol | `Email` |
| Endpoint | `your-email@example.com` |

After entering the email address, choose **Create Subscription**.

Open your email inbox and click the **Confirm Subscription** link sent by Amazon SNS.

## Why is this step required?

An SNS Topic acts as a communication channel, but it does not know where to send notifications until at least one subscriber is added.

By subscribing an email address, CloudWatch alarms can notify you whenever an alarm changes state.

## Dependency

Depends on:

- Step 2 (SNS Topic)

## What happens if this step is skipped?

The SNS Topic will exist, but it will have no subscribers.

CloudWatch will publish notifications to the SNS Topic, but no one will receive them.

---

# Step 4: Create a CloudWatch Billing Alarm

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ All Alarms
→ Create Alarm
```

## Configuration

### Select Metric

Choose the following metric:

```text
Billing
→ Total Estimated Charge
→ USD
```

Select the metric and choose **Select Metric**.

### Configure Metric

| Setting | Value |
|---------|-------|
| Statistic | `Maximum` |
| Period | `6 Hours` |
| Threshold Type | `Static` |
| Whenever Estimated Charges is | `Greater than` |
| Threshold Value | `500` |

Choose **Next**.

### Configure Notification

| Setting | Value |
|---------|-------|
| Alarm State Trigger | `In Alarm` |
| Send Notification To | `Billing-CPU-Alerts` |

Choose **Next**.

### Alarm Name

| Setting | Value |
|---------|-------|
| Alarm Name | `Billing-Alarm-500USD` |
| Alarm Description | `Notify when AWS estimated charges exceed $500.` |

Choose **Create Alarm**.

## Why is this step required?

The billing alarm continuously monitors your AWS account's estimated charges.

If the estimated charges exceed **$500**, CloudWatch changes the alarm state to **Alarm** and immediately publishes a notification to the SNS Topic.

This helps prevent unexpected AWS costs.

## Dependency

Depends on:

- Step 1 (Billing Alerts Enabled)
- Step 2 (SNS Topic)
- Step 3 (SNS Subscription)

## What happens if this step is skipped?

AWS charges will continue increasing without any automatic notification.

You may not notice unusually high costs until you manually check the Billing Dashboard.

---

# Step 5: Create a CPU Utilization Alarm for an EC2 Instance

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ All Alarms
→ Create Alarm
```

## Configuration

### Select Metric

Navigate to:

```text
EC2
→ Per-Instance Metrics
→ CPUUtilization
```

Select the metric corresponding to your running EC2 instance.

Choose **Select Metric**.

### Configure Metric

| Setting | Value |
|---------|-------|
| Statistic | `Average` |
| Period | `5 Minutes` |
| Threshold Type | `Static` |
| Whenever CPUUtilization is | `Greater than` |
| Threshold Value | `65` |

Choose **Next**.

### Configure Notification

| Setting | Value |
|---------|-------|
| Alarm State Trigger | `In Alarm` |
| Send Notification To | `Billing-CPU-Alerts` |

Choose **Next**.

### Alarm Details

| Setting | Value |
|---------|-------|
| Alarm Name | `EC2-CPU-Above-65Percent` |
| Alarm Description | `Notify when EC2 CPU utilization exceeds 65%.` |

Choose **Create Alarm**.

## Why is this step required?

This alarm continuously monitors the CPU usage of the selected EC2 instance.

If the CPU utilization rises above **65%**, CloudWatch changes the alarm state to **Alarm** and sends a notification using Amazon SNS.

This helps administrators detect high CPU usage caused by:

- Heavy application load
- Infinite loops
- Misconfigured software
- Performance bottlenecks

## Dependency

Depends on:

- Step 2 (SNS Topic)
- Step 3 (SNS Subscription)
- Running EC2 Instance

## What happens if this step is skipped?

The EC2 instance will continue running even if its CPU usage becomes extremely high.

Administrators will not receive any warning, increasing the risk of performance issues or application downtime.

---

# Step 6: Review the Created Alarms

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ All Alarms
```

## Configuration

Verify that both alarms are listed.

| Alarm | Expected Initial State |
|--------|------------------------|
| Billing-Alarm-500USD | `OK` |
| EC2-CPU-Above-65Percent | `OK` (or `Insufficient Data` for a few minutes) |

The CPU alarm may temporarily display **Insufficient Data** immediately after creation because CloudWatch is waiting to receive enough metric data from the EC2 instance.

After sufficient monitoring data is collected, the state changes automatically to **OK** unless the threshold is exceeded.

## Why is this step required?

Reviewing the alarms confirms that they have been successfully created and are actively monitoring their respective metrics.

It also allows you to verify that the SNS notification action has been associated correctly.

## Dependency

Depends on:

- Step 4
- Step 5

## What happens if this step is skipped?

You may assume the alarms were created successfully without confirming their status.

Configuration mistakes may go unnoticed until an actual incident occurs.

---

---

# Verification / Output Checking

## Verification Step 1: Verify the Billing Alarm

### Action

Open the CloudWatch Alarms page and verify that the Billing Alarm has been created successfully.

### Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ All Alarms
```

### Expected Output

You should see an alarm similar to the following:

| Alarm Name | Metric | State |
|------------|--------|-------|
| `Billing-Alarm-500USD` | EstimatedCharges | `OK` |

> **Note:** The state may temporarily show **Insufficient Data** immediately after creation. This is normal and will automatically change once CloudWatch receives sufficient billing metric data.

### Why does this confirm success?

This confirms that:

- The Billing metric was successfully selected.
- The alarm was created successfully.
- CloudWatch is monitoring the estimated AWS charges.
- The alarm is ready to notify the configured SNS Topic when the threshold exceeds **$500**.

---

# Verification Step 2: Verify the CPU Utilization Alarm

### Action

Open the CloudWatch Alarms page and verify that the CPU alarm has been created.

### Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ All Alarms
```

### Expected Output

| Alarm Name | Metric | State |
|------------|--------|-------|
| `EC2-CPU-Above-65Percent` | CPUUtilization | `OK` |

or

| Alarm Name | Metric | State |
|------------|--------|-------|
| `EC2-CPU-Above-65Percent` | CPUUtilization | `Insufficient Data` |

The **Insufficient Data** state is expected immediately after creating the alarm because CloudWatch needs enough metric data before evaluating the alarm.

### Why does this confirm success?

This confirms that:

- The correct EC2 metric was selected.
- CloudWatch is receiving CPU metrics from the EC2 instance.
- The alarm is ready to monitor CPU utilization and send notifications if the CPU usage exceeds **65%**.

---

# Verification Step 3: Verify the SNS Subscription

### Action

Open the SNS Topic and verify that the email subscription is confirmed.

### Navigation

```text
AWS Console
→ SNS
→ Topics
→ Billing-CPU-Alerts
→ Subscriptions
```

### Expected Output

| Protocol | Endpoint | Status |
|----------|----------|--------|
| Email | `your-email@example.com` | Confirmed |

### Why does this confirm success?

A confirmed subscription indicates that Amazon SNS can successfully send notifications to the specified email address.

If the status is **Pending Confirmation**, notifications will not be delivered.

---

# Verification Step 4: Verify Alarm Actions

### Action

Open each alarm and review its configured notification action.

### Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ Select Alarm
→ Actions
```

### Expected Output

Both alarms should have the following action configured:

| Alarm State | Action |
|-------------|--------|
| In Alarm | Send notification to `Billing-CPU-Alerts` |

### Why does this confirm success?

This verifies that whenever the alarm changes to the **Alarm** state, CloudWatch will automatically publish a notification to the SNS Topic.

Without this action, the alarm would change state but no notification would be sent.

---

# Cleanup / Cost Optimization

Delete every resource created during this assignment immediately after verification to avoid unnecessary AWS charges.

---

# Cleanup Dependency Order

```text
Billing Alarm
        │
        ▼
CPU Alarm
        │
        ▼
SNS Subscription
        │
        ▼
SNS Topic
```

Deleting resources in this order ensures that dependent resources are removed before deleting the parent resource.

---

# Cleanup Step 1: Delete the Billing Alarm

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ Billing-Alarm-500USD
→ Delete
```

## Action

Delete the Billing Alarm.

## Why is this required?

Once the assignment is verified, the Billing Alarm is no longer required.

Removing unused alarms helps keep your CloudWatch environment clean and avoids charges if you exceed the Free Tier limits.

## What happens if this resource is not deleted?

The alarm continues monitoring billing metrics.

If your account exceeds the Free Tier limit for CloudWatch alarms, additional charges may apply.

---

# Cleanup Step 2: Delete the CPU Utilization Alarm

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
→ EC2-CPU-Above-65Percent
→ Delete
```

## Action

Delete the CPU Utilization Alarm.

## Why is this required?

The alarm continuously evaluates EC2 metrics.

Deleting unused alarms helps minimize CloudWatch resource usage.

## What happens if this resource is not deleted?

The alarm remains active and continues monitoring the EC2 instance.

Additional CloudWatch alarm charges may apply if your account exceeds the Free Tier limits.

---

# Cleanup Step 3: Delete the SNS Subscription

## Navigation

```text
AWS Console
→ SNS
→ Topics
→ Billing-CPU-Alerts
→ Subscriptions
→ Delete
```

## Action

Delete the email subscription.

## Why is this required?

Removing the subscription prevents future notifications from being sent to your email.

It also removes the dependency before deleting the SNS Topic.

## What happens if this resource is not deleted?

The subscription remains associated with the SNS Topic and may continue receiving notifications if the topic is used in the future.

---

# Cleanup Step 4: Delete the SNS Topic

## Navigation

```text
AWS Console
→ SNS
→ Topics
→ Billing-CPU-Alerts
→ Delete
```

## Action

Delete the SNS Topic.

## Why is this required?

The SNS Topic was created specifically for this assignment.

Deleting it removes the final AWS resource created during the exercise.

## What happens if this resource is not deleted?

The SNS Topic remains in your AWS account.

Although it generally incurs little or no cost while idle, unused resources should be removed to keep the AWS environment organized and to avoid accidental use in future configurations.

---

