# Module 4 - Introduction to Elastic Load Balancer and Auto Scaling

# Case Study – ELB, Auto Scaling Group (ASG) and Route 53

## Problem Statement

XYZ Corporation currently hosts its application on on-premises servers. As the number of users increases, the application experiences high CPU utilization and performance degradation. To handle the increased load, the company frequently purchases new physical servers, which increases operational costs.

The company has decided to migrate its infrastructure to AWS so that compute resources can automatically scale based on demand, incoming traffic can be distributed across multiple servers, and users can access the application using the company's domain name.

---

# Objectives

This case study demonstrates how to:

1. Automatically launch EC2 instances when CPU utilization exceeds **80%**.
2. Automatically terminate unnecessary EC2 instances when CPU utilization falls below **60%**.
3. Distribute incoming traffic across multiple EC2 instances using an Application Load Balancer (ALB).
4. Route the company's domain to the Application Load Balancer using Amazon Route 53.

---

# AWS Architecture

```text
                           Internet
                               │
                               │
                        Company Domain
                     (example: xyzcorp.com)
                               │
                               │
                         Amazon Route 53
                               │
                               │
                   Application Load Balancer
                               │
        ┌──────────────────────┴──────────────────────┐
        │                                             │
        │                                             │
  EC2 Instance 1                              EC2 Instance 2
        │                                             │
        └────────────── Auto Scaling Group ───────────┘
                         (Launch/Terminate)
                               │
                               │
                    CloudWatch CPU Monitoring
                 Scale Out > 80% CPU Utilization
                 Scale In  < 60% CPU Utilization
```

---

# AWS Services Used

| AWS Service | Purpose |
|-------------|---------|
| Amazon EC2 | Hosts the web application |
| Security Group | Controls inbound and outbound traffic |
| Launch Template | Stores EC2 launch configuration |
| Target Group | Registers EC2 instances for the Load Balancer |
| Application Load Balancer | Distributes incoming HTTP traffic |
| Auto Scaling Group | Automatically launches and terminates EC2 instances |
| Amazon CloudWatch | Monitors CPU utilization and triggers scaling |
| Amazon Route 53 | Maps the company domain to the Load Balancer |

---

# Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|------|
| Amazon EC2 (`t2.micro`) | Yes | No (within Free Tier limits) | 750 hours/month in Free Tier |
| Security Groups | Yes | No | No additional cost |
| Launch Template | Yes | No | Free service |
| Target Group | Yes | No | No separate charge |
| Application Load Balancer | No | Yes | Hourly and LCU charges apply |
| Auto Scaling Group | Yes | No | No charge for ASG itself; EC2 charges still apply |
| Amazon CloudWatch Basic Monitoring | Yes | No | Basic metrics are included |
| CloudWatch Alarms | Partially | May use credits | Beyond Free Tier limits, alarms incur charges |
| Route 53 Hosted Zone | No | Yes | Monthly hosted zone charge |
| Route 53 DNS Queries | No | Yes | Charged per DNS query |

---

## Free Tier Eligibility

This assignment is **NOT completely covered under the AWS Free Tier**.

The following AWS services consume AWS promotional credits:

- Application Load Balancer (ALB)
- Route 53 Hosted Zone
- Route 53 DNS Queries
- Additional CloudWatch alarms (if Free Tier limits are exceeded)

The remaining services such as EC2 (`t2.micro`), Security Groups, Launch Templates, Auto Scaling Groups, and basic CloudWatch monitoring are eligible under the AWS Free Tier when used within AWS limits.

> **Recommendation:** It is better to perform this case study while you still have AWS promotional credits because both the Application Load Balancer and Route 53 are paid services.

---

# Resource Dependency Flow

```text
VPC
 │
 ├── Public Subnets
 │
 ├── Internet Gateway
 │
 ├── Route Table
 │
 ├── Security Groups
 │
 ├── Launch Template
 │
 ├── Target Group
 │
 ├── Application Load Balancer
 │
 ├── Listener
 │
 ├── Auto Scaling Group
 │
 ├── Scaling Policies
 │
 ├── CloudWatch Alarms
 │
 └── Route53 Hosted Zone
           │
           └── Alias Record → ALB
```

---

# Step 1: Create a Security Group for the Web Servers

## Navigation

```text
AWS Console
→ EC2
→ Security Groups
→ Create Security Group
```

## Configuration

| Setting | Value |
|----------|-------|
| **Security Group Name** | `WebServer-SG` |
| **Description** | `Allows HTTP and SSH access` |
| **VPC** | Select your VPC |

### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | `22` | `My IP` |
| HTTP | TCP | `80` | `0.0.0.0/0` |

Leave outbound rules as default (`Allow All`).

## Why is this step required?

The Security Group acts as a virtual firewall.

It allows:

- SSH so administrators can connect to EC2 instances.
- HTTP so the Load Balancer can forward web traffic to the instances.

Without these rules, users or administrators cannot communicate with the EC2 instances.

## Dependency

Requires:

- Existing VPC

## What happens if this step is skipped?

The EC2 instances may launch successfully, but:

- SSH connections will fail.
- HTTP traffic from the Load Balancer will fail.
- Health checks will fail.
- The Load Balancer will mark all instances as unhealthy.
---

# Step 2: Create a Launch Template

## Navigation

```text
AWS Console
→ EC2
→ Launch Templates
→ Create Launch Template
```

## Configuration

| Setting | Value |
|---------|-------|
| **Launch Template Name** | `WebServer-LT` |
| **Template Version Description** | `Version 1` |
| **AMI** | `Amazon Linux 2023 AMI (Free Tier Eligible)` |
| **Instance Type** | `t2.micro` |
| **Key Pair** | Select your existing key pair |
| **Network Settings** | Do not include subnet (ASG will choose it) |
| **Security Group** | `WebServer-SG` |
| **Storage** | `8 GiB gp3` (Default) |
| **IAM Role** | None (unless specifically required) |

### User Data

Paste the following script into the **User Data** section.

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>XYZ Corporation</h1>" > /var/www/html/index.html
echo "<h2>Instance ID: $INSTANCE_ID</h2>" >> /var/www/html/index.html
```

---

## Command Explanation

### Command

```bash
#!/bin/bash
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `#!` | Called the **shebang**. It tells Linux which interpreter should execute the script. |
| `/bin/bash` | Specifies that the Bash shell should execute all commands in this script. |

---

### Command

```bash
yum update -y
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `yum` | Package manager used by Amazon Linux to install and update software packages. |
| `update` | Updates all installed packages to their latest available versions. |
| `-y` | Automatically answers **Yes** to all confirmation prompts so the command runs without user interaction. |

---

### Command

```bash
yum install -y httpd
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `yum` | Amazon Linux package manager. |
| `install` | Installs a software package. |
| `-y` | Automatically confirms the installation. |
| `httpd` | Apache HTTP Server package that will host the web application. |

---

### Command

```bash
systemctl enable httpd
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `systemctl` | Utility used to manage Linux services. |
| `enable` | Configures the service to start automatically during every system boot. |
| `httpd` | Apache Web Server service. |

---

### Command

```bash
systemctl start httpd
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `systemctl` | Linux service management utility. |
| `start` | Starts the service immediately. |
| `httpd` | Apache Web Server service. |

---

### Command

```bash
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `INSTANCE_ID` | Shell variable that stores the EC2 Instance ID. |
| `=` | Assignment operator that stores the command output into the variable. |
| `$()` | Command substitution. Executes the command inside the parentheses and stores its output. |
| `curl` | Command-line tool used to retrieve data from a URL. |
| `-s` | Silent mode. Suppresses progress and error messages. |
| `http://169.254.169.254` | Special AWS Instance Metadata Service (IMDS) address accessible only from inside the EC2 instance. |
| `/latest/meta-data/instance-id` | Metadata endpoint that returns the current EC2 instance ID. |

---

### Command

```bash
echo "<h1>XYZ Corporation</h1>" > /var/www/html/index.html
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `echo` | Prints the specified text. |
| `"<h1>XYZ Corporation</h1>"` | HTML heading that will appear on the web page. |
| `>` | Output redirection operator. Creates the file if it does not exist or overwrites the existing file. |
| `/var/www/html/index.html` | Default web page served by Apache HTTP Server. |

---

### Command

```bash
echo "<h2>Instance ID: $INSTANCE_ID</h2>" >> /var/www/html/index.html
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `echo` | Prints the specified text. |
| `"<h2>Instance ID: $INSTANCE_ID</h2>"` | Displays the EC2 instance ID inside the web page. |
| `$INSTANCE_ID` | Retrieves the value stored in the `INSTANCE_ID` variable. |
| `>>` | Append redirection operator. Adds content to the end of the existing file instead of replacing it. |
| `/var/www/html/index.html` | Existing web page created in the previous command. |

---

## Why is this step required?

A Launch Template defines how every EC2 instance should be created.

Instead of manually configuring each server, Auto Scaling uses the Launch Template whenever it launches a new instance.

The Launch Template contains:

- Operating System
- Instance Type
- Security Group
- Storage Configuration
- Key Pair
- User Data Script

This ensures every new EC2 instance is configured identically.

---

## Dependency

Requires:

- Existing VPC
- Existing Security Group (`WebServer-SG`)
- Existing Key Pair

---

## What happens if this step is skipped?

Without a Launch Template:

- Auto Scaling Group cannot launch EC2 instances.
- Every EC2 instance would need to be created manually.
- Automatic scaling would not be possible.

---

# Step 3: Create a Target Group

## Navigation

```text
AWS Console
→ EC2
→ Target Groups
→ Create Target Group
```

## Configuration

| Setting | Value |
|---------|-------|
| **Target Type** | `Instances` |
| **Target Group Name** | `WebServer-TG` |
| **Protocol** | `HTTP` |
| **Port** | `80` |
| **VPC** | Select your VPC |
| **Health Check Protocol** | `HTTP` |
| **Health Check Path** | `/` |
| **Healthy Threshold** | Default |
| **Unhealthy Threshold** | Default |

Do **not** register any instances manually.

The Auto Scaling Group will automatically register instances after they are launched.

---

## Why is this step required?

A Target Group is responsible for maintaining the list of EC2 instances that receive traffic from the Application Load Balancer.

It also performs periodic health checks on each registered instance.

Only healthy instances receive incoming requests.

---

## Dependency

Requires:

- Existing VPC

---

## What happens if this step is skipped?

Without a Target Group:

- The Application Load Balancer has nowhere to send requests.
- Auto Scaling cannot automatically register newly launched EC2 instances.
- Users will receive errors because no backend servers are available.

---

# Step 4: Create an Application Load Balancer (ALB)

## Navigation

```text
AWS Console
→ EC2
→ Load Balancers
→ Create Load Balancer
→ Application Load Balancer
```

## Configuration

### Basic Configuration

| Setting | Value |
|---------|-------|
| **Load Balancer Name** | `XYZCorp-ALB` |
| **Scheme** | `Internet-facing` |
| **IP Address Type** | `IPv4` |
| **Load Balancer Type** | `Application Load Balancer` |

### Network Mapping

Select:

- Your existing **VPC**
- At least **two public subnets** in different Availability Zones

Example:

| Availability Zone | Subnet |
|-------------------|---------|
| `ap-south-1a` | Public Subnet 1 |
| `ap-south-1b` | Public Subnet 2 |

### Security Group

Select:

- `WebServer-SG` *(or another security group that allows HTTP port `80`)*

### Listener

| Setting | Value |
|---------|-------|
| **Protocol** | `HTTP` |
| **Port** | `80` |
| **Default Action** | Forward to `WebServer-TG` |

Click **Create Load Balancer**.

---

## Why is this step required?

An Application Load Balancer (ALB) distributes incoming HTTP requests across multiple EC2 instances.

Benefits include:

- Prevents a single server from becoming overloaded.
- Improves application availability.
- Automatically routes traffic only to healthy instances.
- Works seamlessly with Auto Scaling Groups.

---

## Dependency

Requires:

- Existing VPC
- At least **two public subnets** in different Availability Zones
- Existing Security Group
- Existing Target Group (`WebServer-TG`)

---

## What happens if this step is skipped?

Without an Application Load Balancer:

- Users connect directly to individual EC2 instances.
- Traffic cannot be distributed evenly.
- If one EC2 instance fails, users connected to it experience downtime.
- Auto Scaling instances will not receive centralized traffic.

---

# Step 5: Configure the Listener

> **Note:** A default HTTP listener is usually created during ALB creation. This step explains its configuration for learning and revision.

## Navigation

```text
AWS Console
→ EC2
→ Load Balancers
→ Select XYZCorp-ALB
→ Listeners
```

## Configuration

| Setting | Value |
|---------|-------|
| **Listener Protocol** | `HTTP` |
| **Listener Port** | `80` |
| **Default Action** | Forward to `WebServer-TG` |

Save the configuration if any changes are made.

---

## Why is this step required?

A Listener waits for incoming client requests on a specific protocol and port.

In this case:

- It listens on **HTTP port 80**.
- It forwards all requests to the Target Group, which then selects a healthy EC2 instance.

The Listener acts as the entry point for the Load Balancer.

---

## Dependency

Requires:

- Existing Application Load Balancer
- Existing Target Group

---

## What happens if this step is skipped?

Without a Listener:

- The Load Balancer receives requests but has no instructions on how to process them.
- Client requests will fail because they cannot be forwarded to backend instances.

---

# Step 6: Create an Auto Scaling Group (ASG)

## Navigation

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Create Auto Scaling Group
```

## Configuration

### Basic Details

| Setting | Value |
|---------|-------|
| **Auto Scaling Group Name** | `XYZCorp-ASG` |
| **Launch Template** | `WebServer-LT` |
| **Launch Template Version** | `Default` |

Click **Next**.

---

### Network

| Setting | Value |
|---------|-------|
| **VPC** | Select your VPC |
| **Availability Zones** | Select at least two public subnets in different Availability Zones |

Example:

- Public Subnet 1 (`ap-south-1a`)
- Public Subnet 2 (`ap-south-1b`)

Click **Next**.

---

### Attach to Existing Load Balancer

Choose:

- **Attach to an existing load balancer**

Then select:

- `WebServer-TG`

This automatically registers all EC2 instances launched by the Auto Scaling Group with the Target Group.

Click **Next**.

---

### Health Checks

| Setting | Value |
|---------|-------|
| **Health Check Type** | `EC2` and `ELB` |
| **Health Check Grace Period** | `300 seconds` |

---

### Group Size

| Setting | Value |
|---------|-------|
| **Desired Capacity** | `2` |
| **Minimum Capacity** | `2` |
| **Maximum Capacity** | `5` |

Explanation:

- Two EC2 instances are always running.
- The group can scale up to five instances when required.
- If demand decreases, extra instances are terminated, but at least two remain available.

Click **Next**.

---

### Scaling Policies

Select:

**Target Tracking Scaling Policy**

Configure:

| Setting | Value |
|---------|-------|
| **Metric Type** | `Average CPU Utilization` |
| **Target Value** | `80%` |
| **Instance Warmup** | `300 seconds` |

> **Note:** The assignment specifies scaling when CPU utilization exceeds **80%**. A Target Tracking policy automatically attempts to maintain the target around this value.

Click **Next**.

---

### Notifications

Leave as default unless notifications are specifically required.

---

### Tags

| Key | Value |
|-----|-------|
| `Name` | `ASG-WebServer` |

Enable:

- **Propagate to instances launched by this Auto Scaling Group**

This ensures every EC2 instance receives the same `Name` tag automatically.

Click **Create Auto Scaling Group**.

---

## Why is this step required?

The Auto Scaling Group automatically manages the number of EC2 instances based on demand.

It provides:

- Automatic launch of new EC2 instances during high traffic.
- Automatic termination of unnecessary instances during low traffic.
- High availability by distributing instances across multiple Availability Zones.
- Automatic registration of instances with the Target Group.

---

## Dependency

Requires:

- Launch Template
- Application Load Balancer
- Target Group
- Existing VPC
- Public Subnets
- Security Group

---

## What happens if this step is skipped?

Without an Auto Scaling Group:

- New EC2 instances must be created manually.
- Failed instances are not automatically replaced.
- The application cannot automatically adapt to changing workloads.
- High CPU utilization may lead to degraded performance or downtime.

---

# Step 7: Configure the Scale-Out Policy (CPU Utilization > 80%)

> **Note:** In the previous step, we configured a **Target Tracking Scaling Policy** with a target CPU utilization of `80%`, which is the AWS recommended approach for most production workloads.
>
> However, since the case study explicitly states that new compute resources should be deployed **when CPU utilization exceeds 80%**, this section demonstrates how to configure a **Simple Scaling Policy** with a CloudWatch Alarm to meet the assignment requirement.

## Navigation

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Select XYZCorp-ASG
→ Automatic Scaling
→ Create Dynamic Scaling Policy
```

## Configuration

### Scaling Policy

| Setting | Value |
|---------|-------|
| **Policy Type** | `Simple Scaling` |
| **Policy Name** | `Scale-Out-CPU80` |
| **Scaling Adjustment** | `Add 1 Capacity Unit` |
| **Cooldown** | `300 Seconds` |

Save the policy.

---

## Create CloudWatch Alarm

After creating the scaling policy:

```text
AWS Console
→ CloudWatch
→ Alarms
→ Create Alarm
```

### Configuration

| Setting | Value |
|---------|-------|
| **Metric** | `EC2 → Auto Scaling Group → CPUUtilization` |
| **Statistic** | `Average` |
| **Threshold Type** | `Static` |
| **Condition** | `Greater than 80` |
| **Evaluation Period** | `1` |
| **Period** | `5 Minutes` |
| **Action** | `Invoke Scale-Out-CPU80 Policy` |

Create the alarm.

---

## Why is this step required?

When the average CPU utilization exceeds **80%**, the existing EC2 instances are approaching their processing limits.

Instead of allowing performance to degrade, CloudWatch triggers the Auto Scaling policy, which launches an additional EC2 instance.

This helps:

- Handle increased traffic.
- Reduce CPU load.
- Maintain application performance.
- Improve availability.

---

## Dependency

Requires:

- Auto Scaling Group
- Launch Template
- CloudWatch Metrics

---

## What happens if this step is skipped?

Without a Scale-Out policy:

- CPU utilization may continue increasing.
- Existing EC2 instances may become overloaded.
- Users may experience slow response times or application failures.
- Auto Scaling will not launch additional instances during heavy traffic.

---

# Step 8: Configure the Scale-In Policy (CPU Utilization < 60%)

The assignment also requires removing unnecessary compute resources when CPU utilization falls below **60%**.

## Navigation

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Select XYZCorp-ASG
→ Automatic Scaling
→ Create Dynamic Scaling Policy
```

## Configuration

### Scaling Policy

| Setting | Value |
|---------|-------|
| **Policy Type** | `Simple Scaling` |
| **Policy Name** | `Scale-In-CPU60` |
| **Scaling Adjustment** | `Remove 1 Capacity Unit` |
| **Cooldown** | `300 Seconds` |

Save the policy.

---

## Create CloudWatch Alarm

```text
AWS Console
→ CloudWatch
→ Alarms
→ Create Alarm
```

### Configuration

| Setting | Value |
|---------|-------|
| **Metric** | `EC2 → Auto Scaling Group → CPUUtilization` |
| **Statistic** | `Average` |
| **Threshold Type** | `Static` |
| **Condition** | `Less than 60` |
| **Evaluation Period** | `1` |
| **Period** | `5 Minutes` |
| **Action** | `Invoke Scale-In-CPU60 Policy` |

Create the alarm.

---

## Why is this step required?

During periods of low traffic, running unnecessary EC2 instances wastes AWS resources and increases costs.

The Scale-In policy automatically terminates extra EC2 instances when CPU utilization remains below **60%**.

Benefits include:

- Reduced infrastructure cost.
- Efficient resource utilization.
- Automatic optimization without administrator intervention.

---

## Dependency

Requires:

- Existing Auto Scaling Group
- Existing CloudWatch Metrics
- Existing Scale-In Policy

---

## What happens if this step is skipped?

Without a Scale-In policy:

- Extra EC2 instances continue running even when demand decreases.
- AWS costs increase unnecessarily.
- Compute resources remain underutilized.

---

# Step 9: Configure Amazon Route 53

> **Prerequisite:** You must already own a registered domain name (for example, `xyzcorp.com`). If you do not own one, you can register a domain through Route 53 or another domain registrar.

## Navigation

```text
AWS Console
→ Route 53
→ Hosted Zones
→ Create Hosted Zone
```

## Configuration

### Hosted Zone

| Setting | Value |
|---------|-------|
| **Domain Name** | `your-domain-name.com` |
| **Type** | `Public Hosted Zone` |

Click **Create Hosted Zone**.

---

## Create an Alias Record

After the Hosted Zone is created:

```text
Hosted Zone
→ Create Record
```

### Record Configuration

| Setting | Value |
|---------|-------|
| **Record Name** | Leave blank (Root Domain) or enter `www` |
| **Record Type** | `A - IPv4 Address` |
| **Alias** | `Yes` |
| **Route Traffic To** | `Alias to Application Load Balancer` |
| **Region** | Same Region as the ALB |
| **Application Load Balancer** | `XYZCorp-ALB` |

Click **Create Records**.

---

## Why is this step required?

Users generally access websites using a domain name rather than an AWS Load Balancer DNS name.

Route 53 provides:

- Human-readable domain names.
- Highly available DNS service.
- Automatic routing of traffic to the Application Load Balancer.
- Easy integration with AWS services.

Instead of visiting a long ALB DNS name such as:

```text
xyzcorp-alb-123456.ap-south-1.elb.amazonaws.com
```

Users can simply access:

```text
www.your-domain-name.com
```

---

## Dependency

Requires:

- Registered domain
- Hosted Zone
- Application Load Balancer

---

## What happens if this step is skipped?

Without Route 53:

- Users must access the application using the Load Balancer's DNS name.
- The company cannot use its branded domain name.
- DNS-based routing and domain management are unavailable.

---

# Deployment Flow

```text
User
 │
 ▼
www.your-domain-name.com
 │
 ▼
Amazon Route53
 │
 ▼
Application Load Balancer
 │
 ▼
Target Group
 │
 ▼
Auto Scaling Group
 │
 ├───────────────┐
 ▼               ▼
EC2 Instance 1  EC2 Instance 2
        │
        ▼
Apache Web Server
```
---

# Verification / Output Checking

# Verification Step 1: Verify the Auto Scaling Group

## Action

Verify that the Auto Scaling Group has been created successfully and that the desired number of EC2 instances are running.

## Navigation

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Select XYZCorp-ASG
```

## Expected Output

You should observe:

| Property | Expected Value |
|----------|----------------|
| **Status** | `In Service` |
| **Desired Capacity** | `2` |
| **Minimum Capacity** | `2` |
| **Maximum Capacity** | `5` |
| **Health Status** | `Healthy` |

## Why does this confirm success?

This confirms that the Auto Scaling Group has been created successfully and is maintaining the configured number of EC2 instances.

---

# Verification Step 2: Verify EC2 Instances

## Action

Verify that the Auto Scaling Group launched EC2 instances automatically.

## Navigation

```text
AWS Console
→ EC2
→ Instances
```

## Expected Output

You should see at least two running EC2 instances.

Example:

| Instance Name | State | Health |
|--------------|-------|--------|
| ASG-WebServer | Running | 2/2 Checks Passed |
| ASG-WebServer | Running | 2/2 Checks Passed |

## Why does this confirm success?

The instances were not launched manually.

Their presence confirms that the Auto Scaling Group successfully used the Launch Template.

---

# Verification Step 3: Verify Target Group Health

## Navigation

```text
AWS Console
→ EC2
→ Target Groups
→ WebServer-TG
→ Targets
```

## Expected Output

| Instance | Health Status |
|-----------|---------------|
| Instance 1 | Healthy |
| Instance 2 | Healthy |

## Why does this confirm success?

Healthy targets indicate:

- Apache Web Server is running.
- Security Group allows HTTP traffic.
- Health checks are succeeding.
- The Application Load Balancer can forward requests to the instances.

---

# Verification Step 4: Verify the Application Load Balancer

## Navigation

```text
AWS Console
→ EC2
→ Load Balancers
→ XYZCorp-ALB
```

Copy the **DNS Name**.

Open it in a web browser.

## Expected Output

You should see a web page similar to:

```text
XYZ Corporation

Instance ID:
i-0123456789abcdef0
```

Refreshing the page multiple times may display different Instance IDs if traffic is distributed across multiple instances.

## Why does this confirm success?

This confirms that:

- The Load Balancer is reachable.
- Requests are being forwarded to healthy EC2 instances.
- Apache is serving the web page successfully.

---

# Verification Step 5: Verify Load Balancing

## Action

Refresh the Application Load Balancer DNS name several times.

## Expected Output

Example:

```text
Refresh 1

XYZ Corporation

Instance ID:
i-0a123456789abc001
```

```text
Refresh 2

XYZ Corporation

Instance ID:
i-0a123456789abc004
```

```text
Refresh 3

XYZ Corporation

Instance ID:
i-0a123456789abc002
```

## Why does this confirm success?

Different Instance IDs indicate that the Application Load Balancer is distributing incoming requests among multiple EC2 instances.

---

# Verification Step 6: Verify Route 53

## Action

Open your configured domain in a web browser.

Example:

```text
http://www.your-domain-name.com
```

## Expected Output

The same application page served through the Application Load Balancer should appear.

## Why does this confirm success?

This confirms:

- Route 53 is resolving the domain correctly.
- DNS records point to the Application Load Balancer.
- End users can access the application using the company's domain name.

---

# Verification Step 7: Verify Scale-Out

## Action

Generate CPU load on one or more EC2 instances (for example, using a CPU stress tool or a load-testing utility) until average CPU utilization exceeds `80%`.

Then monitor:

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Activity
```

## Expected Output

A new scaling activity similar to the following should appear:

```text
Launching a new EC2 instance.
Reason:
CPU Utilization exceeded 80%.
```

The number of running EC2 instances should increase.

## Why does this confirm success?

This confirms that:

- CloudWatch detected high CPU utilization.
- The Scale-Out policy was triggered.
- The Auto Scaling Group launched an additional EC2 instance automatically.

---

# Verification Step 8: Verify Scale-In

## Action

Stop generating CPU load and wait until the average CPU utilization remains below `60%`.

Monitor:

```text
AWS Console
→ EC2
→ Auto Scaling Groups
→ Activity
```

## Expected Output

A scaling activity similar to the following should appear:

```text
Terminating EC2 instance.

Reason:
CPU Utilization below 60%.
```

The number of EC2 instances should decrease, but not below the configured minimum capacity.

## Why does this confirm success?

This confirms that:

- CloudWatch detected reduced CPU utilization.
- The Scale-In policy was triggered.
- The Auto Scaling Group automatically terminated unnecessary EC2 instances.

---

# Cleanup / Cost Optimization

> **Important:** This case study uses paid AWS services such as the Application Load Balancer and Route 53. To avoid unnecessary AWS charges or consumption of promotional credits, delete all resources immediately after completing verification.

---

# Cleanup Step 1: Delete Route 53 DNS Records

## Navigation

```text
AWS Console
→ Route 53
→ Hosted Zones
→ Select Hosted Zone
```

## Action

Delete the Alias `A` record pointing to the Application Load Balancer.

## Why is this required?

Removing DNS records prevents traffic from being routed to AWS resources that are about to be deleted.

## What happens if this resource is not deleted?

The domain continues pointing to a Load Balancer that may no longer exist, causing DNS errors and confusion.

---

# Cleanup Step 2: Delete the Hosted Zone

## Navigation

```text
AWS Console
→ Route 53
→ Hosted Zones
```

## Action

Delete the Hosted Zone.

## Why is this required?

Hosted Zones incur monthly charges.

Deleting the Hosted Zone prevents unnecessary recurring costs.

## What happens if this resource is not deleted?

Monthly Route 53 Hosted Zone charges continue even if no resources are running.

---

# Cleanup Step 3: Delete CloudWatch Alarms

## Navigation

```text
AWS Console
→ CloudWatch
→ Alarms
```

## Action

Delete:

- `Scale-Out-CPU80`
- `Scale-In-CPU60`

## Why is this required?

These alarms are no longer needed after deleting the Auto Scaling Group.

## What happens if this resource is not deleted?

Unnecessary CloudWatch resources remain in your account and may incur charges beyond the Free Tier.

---

# Cleanup Step 4: Delete the Auto Scaling Group

## Navigation

```text
AWS Console
→ EC2
→ Auto Scaling Groups
```

## Action

1. Edit the Auto Scaling Group.
2. Set:

| Property | Value |
|----------|-------|
| **Desired Capacity** | `0` |
| **Minimum Capacity** | `0` |

Wait for all EC2 instances to terminate.

Then delete the Auto Scaling Group.

## Why is this required?

The Auto Scaling Group continuously maintains EC2 instances.

It must be removed before deleting dependent resources.

## What happens if this resource is not deleted?

The Auto Scaling Group may automatically launch new EC2 instances, resulting in additional AWS charges.

---

# Cleanup Step 5: Delete the Application Load Balancer

## Navigation

```text
AWS Console
→ EC2
→ Load Balancers
```

## Action

Delete:

- `XYZCorp-ALB`

## Why is this required?

The Application Load Balancer incurs hourly charges.

Deleting it immediately stops further billing.

## What happens if this resource is not deleted?

Hourly ALB charges continue even when no traffic is received.

---

# Cleanup Step 6: Delete the Target Group

## Navigation

```text
AWS Console
→ EC2
→ Target Groups
```

## Action

Delete:

- `WebServer-TG`

## Why is this required?

The Target Group is no longer required after the Load Balancer has been removed.

## What happens if this resource is not deleted?

Unused Target Groups remain in the account, creating unnecessary clutter.

---

# Cleanup Step 7: Delete the Launch Template

## Navigation

```text
AWS Console
→ EC2
→ Launch Templates
```

## Action

Delete:

- `WebServer-LT`

## Why is this required?

The Launch Template is no longer needed once the Auto Scaling Group has been deleted.

## What happens if this resource is not deleted?

Unused Launch Templates remain in your AWS account, making future management more difficult.

---

# Cleanup Step 8: Delete the Security Group

## Navigation

```text
AWS Console
→ EC2
→ Security Groups
```

## Action

Delete:

- `WebServer-SG`

## Why is this required?

The Security Group is no longer attached to any EC2 instances after all compute resources have been removed.

## What happens if this resource is not deleted?

Unused Security Groups accumulate in the account and can make infrastructure management more complex.

---

# Cleanup Dependency Flow

Delete resources in the following order:

```text
Route53 Record
        │
        ▼
Hosted Zone
        │
        ▼
CloudWatch Alarms
        │
        ▼
Auto Scaling Group
        │
        ▼
Application Load Balancer
        │
        ▼
Target Group
        │
        ▼
Launch Template
        │
        ▼
Security Group
```

Deleting resources in this order ensures that dependent services are removed safely without dependency conflicts and minimizes AWS charges.
