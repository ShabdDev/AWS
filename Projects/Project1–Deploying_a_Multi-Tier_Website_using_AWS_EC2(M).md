# Part 1: Project Overview, Cost Check, Architecture, Prerequisites, and Network Security Setup

# Project 1 – Deploying a Multi-Tier Website Using AWS EC2

## Description

Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) Cloud. Using Amazon EC2 eliminates the need to invest in physical hardware upfront, allowing applications to be developed and deployed faster.

Amazon EC2 allows organizations to:

- Launch virtual servers called EC2 instances.
- Increase or decrease computing capacity according to application demand.
- Configure networking and security.
- Attach persistent storage.
- Automatically scale the number of instances according to traffic.
- Build highly available application architectures.

In this project, Company ABC wants to migrate an existing PHP website and MySQL database to AWS. The application must provide high availability by running multiple EC2 instances through an Auto Scaling group.

The implementation will use:

- Amazon EC2 for hosting the PHP website.
- Amazon Machine Image (AMI) for creating identical web servers.
- Launch Template for defining the EC2 configuration used by Auto Scaling.
- Auto Scaling group with a minimum of two EC2 instances.
- Application Load Balancer for distributing website traffic across multiple EC2 instances.
- Amazon RDS for hosting the MySQL database.
- Security groups for controlling communication between the load balancer, EC2 instances, and RDS database.

---

## Problem Statement

Company ABC wants to move its product to AWS. The existing product currently consists of:

1. MySQL database.
2. PHP website.

The company requires high availability for the product and therefore wants Auto Scaling enabled for the website.

The required tasks are:

1. Launch an EC2 instance.
2. Enable Auto Scaling on these instances with a minimum capacity of `2`.
3. Create an RDS instance.
4. Create the following database and table in the RDS instance:
   - **Database name:** `intel`
   - **Table name:** `data`
   - **Database password:** `intel123`
5. Change the database hostname in the website configuration.
6. Allow traffic from EC2 instances to the RDS instance.
7. Allow application traffic to the EC2 instances.

---

## Important Security Interpretation of the Assignment

The assignment says:

> Allow all-traffic to EC2 instance.

For a learning lab, this could be interpreted literally as creating an inbound security group rule with:

- **Type:** `All traffic`
- **Source:** `0.0.0.0/0`

However, exposing every protocol and every port of an EC2 instance to the entire internet is insecure and is not recommended.

Therefore, this project will implement the technically correct architecture:

- Internet users can access the Application Load Balancer using HTTP port `80`.
- The Application Load Balancer can send HTTP traffic to the EC2 instances.
- Administrative SSH access to the initial EC2 instance is restricted to `My IP`.
- EC2 instances can communicate with the RDS MySQL database on TCP port `3306`.
- The RDS database will not be publicly exposed.

This satisfies the actual application connectivity requirements while following AWS security best practices.

---

# Free Tier and Cost Check

AWS Free Tier eligibility depends on factors including:

- AWS account creation date.
- AWS Free Tier plan.
- Remaining free usage allowances.
- Promotional AWS credits.
- AWS Region.
- Selected instance classes.
- Resource running duration.

The exact Free Tier status displayed in the AWS Console should always be checked before launching resources.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| Amazon EC2 | Possibly | Yes, after applicable allowance or depending on account plan | Charges depend on instance type, operating system, storage, and running duration. |
| Amazon EBS | Possibly | Yes, after applicable allowance | The EC2 root volume can generate storage charges while it exists. |
| Amazon RDS for MySQL | Possibly | Yes, after applicable allowance or depending on account plan | Database instance runtime and storage can generate charges. |
| EC2 Auto Scaling | No additional service charge for Auto Scaling itself | Underlying resources can use credits | EC2 instances launched by the Auto Scaling group are billed normally. |
| Application Load Balancer | Generally billable | Yes | Load balancer runtime and processed capacity can generate charges. |
| Launch Template | No direct charge | No direct resource charge | Instances created from the template are billable. |
| Amazon Machine Image | AMI itself has no separate runtime charge | Snapshot storage can use credits | EBS snapshots backing a custom AMI can generate storage charges. |
| Security Groups | No direct charge | No | Security groups themselves do not incur separate charges. |
| Amazon CloudWatch | Basic monitoring is available without additional charge in many cases | Advanced usage can incur charges | Avoid creating unnecessary custom metrics, alarms, and detailed monitoring for this lab. |

## Is This Project Completely Free Tier Eligible?

No. This project should not be assumed to be completely free because it requires multiple potentially billable resources, especially:

- Multiple EC2 instances.
- An Application Load Balancer.
- An RDS MySQL database instance.
- EBS volumes.
- An EBS snapshot created for the custom AMI.

The Auto Scaling service itself does not have a separate additional charge, but the EC2 instances launched by the Auto Scaling group are billed according to normal EC2 pricing.

## Recommended Cost Strategy

If promotional AWS credits are available, it is better to complete this project while those credits remain valid.

To minimize cost:

1. Use small instance types that are eligible under your account's displayed Free Tier or credit plan.
2. Use a small RDS DB instance class suitable for the lab.
3. Keep the Auto Scaling group at exactly:
   - Minimum capacity: `2`
   - Desired capacity: `2`
   - Maximum capacity: `2`
4. Do not enable Multi-AZ for this learning lab unless specifically required.
5. Do not enable unnecessary RDS Performance Insights or Enhanced Monitoring.
6. Delete all resources immediately after completing verification.
7. Delete the Auto Scaling group before attempting to manually terminate its instances; otherwise, Auto Scaling may launch replacements.
8. Delete the custom AMI and its associated EBS snapshot during cleanup.

---

# Planned Architecture

The project will use the following architecture:

~~~text
                              Internet Users
                                    |
                                    v
                     +-----------------------------+
                     | Application Load Balancer   |
                     | Public HTTP Port: 80        |
                     +-----------------------------+
                              |             |
                              v             v
                    +-------------+   +-------------+
                    | EC2 Web 1   |   | EC2 Web 2   |
                    | PHP Website |   | PHP Website |
                    +-------------+   +-------------+
                              \             /
                               \           /
                                v         v
                         +-----------------------+
                         | Amazon RDS for MySQL  |
                         | Database: intel       |
                         | Table: data           |
                         | Port: 3306            |
                         +-----------------------+
~~~

## Architecture Components

| Component | Purpose |
| --------- | ------- |
| EC2 | Hosts the PHP website. |
| Custom AMI | Creates reusable images of the configured web server. |
| Launch Template | Defines how Auto Scaling launches new EC2 web servers. |
| Auto Scaling Group | Maintains at least two EC2 instances for availability. |
| Application Load Balancer | Distributes HTTP requests among healthy EC2 instances. |
| Target Group | Registers EC2 instances and performs health checks. |
| RDS MySQL | Hosts the application's relational database. |
| ALB Security Group | Allows HTTP traffic from internet users. |
| EC2 Security Group | Allows application traffic from the load balancer and restricted SSH administration. |
| RDS Security Group | Allows MySQL traffic only from EC2 instances. |

---

# Availability Design

To implement meaningful high availability, the Auto Scaling group will use at least two Availability Zones.

~~~text
AWS Region
|
+-- Availability Zone A
|   |
|   +-- Public Subnet A
|       |
|       +-- EC2 Web Server 1
|
+-- Availability Zone B
    |
    +-- Public Subnet B
        |
        +-- EC2 Web Server 2
~~~

If one EC2 instance becomes unhealthy, Auto Scaling can replace it.

If one Availability Zone experiences a failure, the second EC2 instance in another Availability Zone can continue serving requests, subject to the availability of the remaining architecture components.

> **Important:** The assignment requires an RDS instance but does not explicitly require RDS Multi-AZ. Therefore, this lab will use a Single-AZ RDS deployment to reduce cost. In a production architecture requiring database-level high availability, RDS Multi-AZ should be considered.

---

# Technical Dependency Flow

The implementation will follow the exact dependency order below:

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Identify Default VPC and Two Public Subnets
    |
    v
Create ALB Security Group
    |
    v
Create EC2 Security Group
    |
    v
Create RDS Security Group
    |
    v
Create RDS MySQL Instance
    |
    v
Launch Initial EC2 Instance
    |
    v
Install Apache, PHP, and MySQL Client
    |
    v
Connect EC2 to RDS
    |
    v
Create Database: intel
    |
    v
Create Table: data
    |
    v
Deploy PHP Website
    |
    v
Change Website Database Hostname to RDS Endpoint
    |
    v
Verify Website and Database Connectivity
    |
    v
Create Custom AMI from Configured EC2 Instance
    |
    v
Create Launch Template
    |
    v
Create Target Group
    |
    v
Create Application Load Balancer
    |
    v
Create Auto Scaling Group
    |
    v
Maintain Minimum 2 EC2 Instances
    |
    v
Verify Load Balancing and Auto Scaling
    |
    v
Complete Final Verification
    |
    v
Cleanup in Reverse Dependency Order
~~~

---

# Resource Naming Plan

Consistent names make resources easier to identify during implementation, verification, and cleanup.

| Resource | Name |
| -------- | ---- |
| Initial EC2 Instance | `ABC-PHP-Initial-Server` |
| EC2 Security Group | `ABC-EC2-SG` |
| RDS Security Group | `ABC-RDS-SG` |
| ALB Security Group | `ABC-ALB-SG` |
| RDS DB Identifier | `abc-mysql-db` |
| Database Name | `intel` |
| Database Table | `data` |
| Custom AMI | `ABC-PHP-Web-AMI` |
| Launch Template | `ABC-PHP-Launch-Template` |
| Target Group | `ABC-PHP-TG` |
| Application Load Balancer | `ABC-PHP-ALB` |
| Auto Scaling Group | `ABC-PHP-ASG` |

---

# Prerequisites

Before implementation, ensure that you have:

- An active AWS account.
- Permission to create EC2 resources.
- Permission to create security groups.
- Permission to create RDS resources.
- Permission to create load balancers.
- Permission to create Auto Scaling groups.
- Access to the AWS Management Console.
- An EC2 key pair or another supported connection mechanism if SSH access is required.
- At least two public subnets in different Availability Zones for the load balancer and Auto Scaling group.

---

# Step 1: Sign In to AWS and Select the Region

### Navigation

~~~text
AWS Management Console
→ Top navigation bar
→ Region selector
→ Select the AWS Region for this project
~~~

### Configuration

Use one AWS Region consistently for all project resources.

Example:

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai) ap-south-1` |

The Mumbai Region is a suitable example when working from India, but another Region may be selected if required by your course or AWS account.

### Why is this step required?

AWS resources are Region-specific. The EC2 instances, RDS database, load balancer, security groups, target group, launch template, and Auto Scaling group used by this project must be created in the same Region.

### Dependency

This step has no AWS resource dependency.

### What happens if this step is skipped?

If resources are accidentally created in different Regions:

- EC2 may not be able to use the intended security groups.
- The load balancer cannot register EC2 targets from an unrelated Region.
- The RDS database may not be accessible through the intended VPC architecture.
- Resources may appear to be missing when viewing another Region.

---

# Step 2: Identify the Default VPC and Two Public Subnets

This project will use the existing default VPC to avoid unnecessary networking complexity while still satisfying the assignment requirements.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Your VPCs
→ Locate the default VPC
~~~

Record the **VPC ID**.

Example:

| Setting | Example Value |
| ------- | ------------- |
| VPC | `vpc-0123456789abcdef0` |
| Default VPC | `Yes` |

Next, inspect the subnets.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Subnets
→ Filter by the selected default VPC
~~~

Select at least two subnets located in different Availability Zones.

Example:

| Purpose | Subnet | Availability Zone |
| ------- | ------ | ----------------- |
| Web Server / ALB Subnet A | `subnet-aaaaaaaa` | `ap-south-1a` |
| Web Server / ALB Subnet B | `subnet-bbbbbbbb` | `ap-south-1b` |

> **Important:** Use your actual VPC IDs, subnet IDs, and Availability Zones. Do not copy example IDs literally.

### Why is this step required?

The VPC provides the private network in which the EC2 instances and RDS database communicate.

Two subnets in different Availability Zones are required for the Application Load Balancer and are also used to distribute Auto Scaling instances across multiple Availability Zones.

### Dependency

This step depends on:

- Step 1: Correct AWS Region selection.

### What happens if this step is skipped?

Without identifying the VPC and suitable subnets:

- Security groups may be created in the wrong VPC.
- EC2 and RDS resources may not be able to communicate.
- Application Load Balancer creation may fail if suitable subnets are not selected.
- The Auto Scaling group cannot be correctly distributed across Availability Zones.

---

# Step 3: Create the Application Load Balancer Security Group

The load balancer requires a security group that permits users to access the PHP website.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Security group name | `ABC-ALB-SG` |
| Description | `Allows public HTTP traffic to the application load balancer` |
| VPC | Select the default VPC identified in Step 2 |

### Inbound Rules

| Type | Protocol | Port Range | Source | Purpose |
| ---- | -------- | ---------- | ------ | ------- |
| HTTP | TCP | `80` | `0.0.0.0/0` | Allows public IPv4 web traffic. |

If IPv6 is enabled and required, an additional source of `::/0` may be used. It is not necessary for completing this project.

### Outbound Rules

Keep the default outbound rule:

| Type | Protocol | Port Range | Destination |
| ---- | -------- | ---------- | ----------- |
| All traffic | All | All | `0.0.0.0/0` |

Click **Create security group**.

### Why is this step required?

The Application Load Balancer must accept incoming HTTP requests from internet users. Its security group acts as a virtual firewall controlling incoming and outgoing traffic.

### Dependency

This step depends on:

- Step 2: The default VPC must already be identified.

### What happens if this step is skipped?

Without an ALB security group allowing HTTP traffic:

- Users cannot access the website through the load balancer.
- Browser requests will time out or fail.
- Load balancing cannot be properly tested.

---

# Step 4: Create the EC2 Security Group

The EC2 security group controls traffic reaching the PHP web servers.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Security group name | `ABC-EC2-SG` |
| Description | `Allows HTTP from ALB and restricted SSH administration` |
| VPC | Select the same default VPC used by `ABC-ALB-SG` |

### Inbound Rules

Create the following rules:

| Type | Protocol | Port Range | Source | Purpose |
| ---- | -------- | ---------- | ------ | ------- |
| HTTP | TCP | `80` | Security group `ABC-ALB-SG` | Allows website traffic only from the Application Load Balancer. |
| SSH | TCP | `22` | `My IP` | Allows restricted administrative access from your current public IP address. |

> **Important:** When selecting the HTTP source, choose **Custom** and select the `ABC-ALB-SG` security group itself rather than entering `0.0.0.0/0`.

### Temporary Direct HTTP Testing Rule

Before the Application Load Balancer exists, direct browser verification of the initial EC2 instance may be useful.

If required, temporarily add:

| Type | Protocol | Port Range | Source |
| ---- | -------- | ---------- | ------ |
| HTTP | TCP | `80` | `My IP` |

This temporary rule should be removed after the Application Load Balancer is operational.

### Outbound Rules

Keep the default outbound rule:

| Type | Protocol | Port Range | Destination |
| ---- | -------- | ---------- | ----------- |
| All traffic | All | All | `0.0.0.0/0` |

Click **Create security group**.

### Why is this step required?

The PHP website runs on EC2 instances. These instances must accept:

- HTTP requests from the Application Load Balancer.
- SSH connections from the administrator when manual configuration is required.

Using the ALB security group as the source for HTTP is safer than exposing EC2 port `80` directly to the entire internet.

### Dependency

This step depends on:

- Step 2: Default VPC identified.
- Step 3: `ABC-ALB-SG` created.

### What happens if this step is skipped?

Without this security group:

- The load balancer cannot send HTTP traffic to the web servers.
- SSH configuration may not be possible.
- The PHP application cannot be accessed through the intended architecture.

---

# Step 5: Create the RDS Security Group

The RDS security group allows the PHP web servers to communicate with the MySQL database.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
~~~

Security groups used by RDS can be created through the EC2 security group interface because VPC security groups are network-level resources that can be associated with supported AWS resources.

### Configuration

| Setting | Value |
| ------- | ----- |
| Security group name | `ABC-RDS-SG` |
| Description | `Allows MySQL traffic only from ABC EC2 web servers` |
| VPC | Select the same default VPC used by the EC2 instances |

### Inbound Rules

| Type | Protocol | Port Range | Source |
| ---- | -------- | ---------- | ------ |
| MySQL/Aurora | TCP | `3306` | Security group `ABC-EC2-SG` |

Do **not** use:

~~~text
0.0.0.0/0
~~~

for the RDS MySQL inbound rule.

The source must be the EC2 security group:

~~~text
ABC-EC2-SG
~~~

### Outbound Rules

The default outbound rule may remain unchanged.

Click **Create security group**.

### Why is this step required?

The PHP application running on EC2 must connect to the MySQL database on TCP port `3306`.

Using `ABC-EC2-SG` as the source means that resources associated with the EC2 security group can initiate MySQL connections to the RDS database.

This is more secure and maintainable than allowing database connections from arbitrary public IP addresses.

### Dependency

This step depends on:

- Step 2: Default VPC identified.
- Step 4: `ABC-EC2-SG` created.

### What happens if this step is skipped?

Without the RDS security group rule:

- The EC2 instances cannot establish a MySQL connection to RDS.
- Database creation commands executed from EC2 will fail.
- The PHP website cannot read or write database records.
- The application may display database connection errors.

---

# Security Group Communication Flow

The final communication model is:

~~~text
Internet
   |
   | HTTP TCP 80
   v
+--------------------+
| ABC-ALB-SG         |
| Application Load   |
| Balancer           |
+--------------------+
   |
   | HTTP TCP 80
   | Source: ABC-ALB-SG
   v
+--------------------+
| ABC-EC2-SG         |
| PHP Web Servers    |
+--------------------+
   |
   | MySQL TCP 3306
   | Source: ABC-EC2-SG
   v
+--------------------+
| ABC-RDS-SG         |
| RDS MySQL Database |
+--------------------+
~~~

This architecture follows the principle of least privilege:

- The internet can reach only the load balancer on HTTP port `80`.
- The load balancer can reach EC2 web servers on HTTP port `80`.
- EC2 web servers can reach RDS on MySQL port `3306`.
- RDS is not directly exposed to the public internet.

---

# Resources Created So Far

At the end of Part 1, the following resources or prerequisites should exist:

| Resource | Expected State |
| -------- | -------------- |
| AWS Region | Selected |
| Default VPC | Identified |
| Two public subnets | Identified in different Availability Zones |
| `ABC-ALB-SG` | Created |
| `ABC-EC2-SG` | Created |
| `ABC-RDS-SG` | Created |

The project requirements and formatting instructions used for this implementation were provided in the uploaded assignment guidance file. :contentReference[oaicite:0]{index=0}

---

# Part 2: Create the RDS MySQL Database, Launch the Initial EC2 Web Server, and Configure Database Resources

# Step 6: Create the Amazon RDS MySQL Database Instance

The PHP website requires a MySQL database. Amazon Relational Database Service (Amazon RDS) will host and manage the MySQL database engine.

For this project, the RDS instance will use:

- **Database engine:** MySQL
- **DB instance identifier:** `abc-mysql-db`
- **Master username:** `admin`
- **Master password:** `intel123`
- **Initial database name:** `intel`
- **MySQL port:** `3306`
- **Public access:** `No`
- **Security group:** `ABC-RDS-SG`

> **Security note:** The password `intel123` is required by the assignment. It is intentionally simple and should never be used as a production database password.

---

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Create database
~~~

---

### Configuration: Database Creation Method

Select:

| Setting | Value |
| ------- | ----- |
| Database creation method | `Standard create` |

Do not select `Easy create`, because Standard create provides control over networking, security groups, public access, backups, and other database settings required for this project.

---

### Configuration: Engine Options

Configure:

| Setting | Value |
| ------- | ----- |
| Engine type | `MySQL` |
| Engine version | Use the current default MySQL version offered by AWS unless your course requires a specific version |

Do not enable unnecessary advanced engine features for this learning project.

---

### Configuration: Templates

Choose the template according to what your AWS account displays.

Preferred selection:

| Setting | Value |
| ------- | ----- |
| Template | `Free tier`, if available and eligible |

If your account does not display a Free Tier template, choose the smallest suitable configuration supported by your account and understand that charges or promotional credit consumption may occur.

---

### Configuration: Availability and Durability

The assignment requires a single RDS instance and does not explicitly require Multi-AZ deployment.

Use:

| Setting | Value |
| ------- | ----- |
| Deployment option | `Single DB instance` or equivalent Single-AZ option |

> **Important:** The exact wording can vary depending on the current RDS Console interface and selected template.

Do not select a Multi-AZ cluster or Multi-AZ DB instance deployment for this lab unless specifically required, because additional database instances can significantly increase resource usage and cost.

---

### Configuration: Settings

Configure:

| Setting | Value |
| ------- | ----- |
| DB instance identifier | `abc-mysql-db` |
| Master username | `admin` |
| Credentials management | `Self managed`, if this option is displayed |
| Master password | `intel123` |
| Confirm master password | `intel123` |

> **Important:** If AWS rejects `intel123` because of a changed password policy or account-specific restriction, use a stronger password for the RDS master user and document the difference. However, where accepted, use `intel123` to match the assignment requirement.

---

### Configuration: Instance Configuration

Select the smallest suitable DB instance class available under your account's applicable Free Tier or credit plan.

Example:

| Setting | Recommended Value |
| ------- | ----------------- |
| DB instance class | `db.t3.micro` or another small eligible class shown by your AWS Console |

> **Important:** Do not blindly select a specific instance class merely because it was historically Free Tier eligible. AWS account plans and eligibility can vary. Check the pricing and Free Tier indicators shown directly in your AWS Console.

---

### Configuration: Storage

Use a small general-purpose SSD configuration suitable for the lab.

Example:

| Setting | Value |
| ------- | ----- |
| Storage type | `General Purpose SSD (gp3)` if available |
| Allocated storage | Use the minimum value allowed by the selected engine and configuration |
| Storage autoscaling | Disable for this short-lived lab if permitted |

Disabling storage autoscaling for a temporary learning lab prevents unexpected storage growth.

---

### Configuration: Connectivity

This is one of the most important sections.

Configure:

| Setting | Value |
| ------- | ----- |
| Compute resource | `Don't connect to an EC2 compute resource` |
| Network type | `IPv4` |
| Virtual private cloud (VPC) | Select the default VPC identified in Step 2 |
| DB subnet group | Use the default DB subnet group associated with the selected VPC |
| Public access | `No` |
| VPC security group | `Choose existing` |
| Existing VPC security groups | `ABC-RDS-SG` |
| Availability Zone | `No preference` |
| Database port | `3306` |

Remove the `default` security group from the RDS instance if the Console automatically selects it and retain only:

~~~text
ABC-RDS-SG
~~~

### Why select "Don't connect to an EC2 compute resource"?

The EC2 instance has not yet been launched at this point in the dependency order.

We have already manually configured secure connectivity using security groups:

~~~text
ABC-EC2-SG
      |
      | TCP 3306
      v
ABC-RDS-SG
~~~

Therefore, AWS does not need to automatically configure an EC2 connection.

---

### Configuration: Database Authentication

Use:

| Setting | Value |
| ------- | ----- |
| Database authentication | `Password authentication` |

The PHP application will authenticate using a database username and password.

---

### Configuration: Additional Configuration

Expand **Additional configuration** if necessary.

Configure the initial database name:

| Setting | Value |
| ------- | ----- |
| Initial database name | `intel` |

This causes RDS to create the `intel` database during database provisioning.

> **Important:** The **DB instance identifier** and **initial database name** are different values:
>
> - RDS instance identifier: `abc-mysql-db`
> - MySQL database name: `intel`

Do not confuse them.

---

### Configuration: Backup

For a short-lived learning project, use a minimal backup configuration allowed by your selected template and account.

If automated backups can be disabled:

| Setting | Value |
| ------- | ----- |
| Backup retention period | `0 days` |

If the selected RDS configuration requires backups, retain the minimum permitted value.

The assignment does not require creating or restoring a database backup.

---

### Configuration: Encryption

If storage encryption is enabled by default, leave it enabled.

Do not create a customer-managed AWS KMS key specifically for this assignment unless required.

---

### Configuration: Performance Insights and Monitoring

For this temporary lab, avoid enabling unnecessary paid monitoring features.

Use:

| Setting | Recommended Value |
| ------- | ----------------- |
| Performance Insights / Database Insights | Keep the lowest-cost/default option |
| Enhanced Monitoring | Disabled unless specifically required |
| Log exports | None required |

---

### Create the Database

Review the estimated monthly cost shown by AWS before proceeding.

Then click:

~~~text
Create database
~~~

Wait until the RDS database status becomes:

~~~text
Available
~~~

This may take several minutes.

---

### Why is this step required?

The PHP application needs a persistent relational database for storing application data.

Amazon RDS manages database infrastructure tasks such as:

- Database server provisioning.
- Hardware management.
- Database engine installation.
- Storage management.
- Monitoring integration.
- Automated backup capabilities when enabled.

The EC2 instances will run the application layer, while RDS will run the database layer.

---

### Dependency

This step depends on:

- Step 1: AWS Region selected.
- Step 2: Default VPC identified.
- Step 5: `ABC-RDS-SG` created.
- Step 4: `ABC-EC2-SG` created because it is referenced by the RDS security group's inbound rule.

---

### What happens if this step is skipped?

Without the RDS MySQL database:

- The PHP application has no database server.
- The `intel` database cannot be used.
- The `data` table cannot be created.
- Database-dependent website functionality will fail.

---

# Step 7: Record the RDS Endpoint

After the RDS instance reaches the `Available` state, record its endpoint.

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Select abc-mysql-db
→ Connectivity & security
→ Endpoint & port
~~~

You should see values similar to:

| Setting | Example |
| ------- | ------- |
| Endpoint | `abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com` |
| Port | `3306` |

Your actual endpoint will be different.

Record it as:

~~~text
RDS_ENDPOINT=your-actual-rds-endpoint
~~~

Example only:

~~~text
RDS_ENDPOINT=abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com
~~~

Do not include:

- `http://`
- `https://`
- `/`
- `:3306`

when a command asks specifically for only the RDS hostname.

Correct format:

~~~text
abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com
~~~

Incorrect formats:

~~~text
http://abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com

https://abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com

abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com:3306/
~~~

---

### Why is this step required?

The RDS endpoint is the hostname used by:

- The MySQL client running on EC2.
- The PHP application.
- Database connection configuration.

The endpoint allows applications to locate the managed RDS database server.

---

### Dependency

This step depends on:

- Step 6: The RDS database must exist and reach the `Available` state.

---

### What happens if this step is skipped?

Without the RDS endpoint:

- The EC2 instance will not know which database hostname to connect to.
- The website's database configuration cannot be completed.
- MySQL connection testing cannot be performed.

---

# Step 8: Create or Identify an EC2 Key Pair

A key pair is required if you plan to connect to the EC2 instance using SSH.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Key Pairs
~~~

If you already have a suitable key pair for the selected AWS Region and possess its private key file, you may reuse it.

Otherwise:

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Key Pairs
→ Create key pair
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Name | `ABC-PHP-Key` |
| Key pair type | `RSA` |
| Private key file format | `.pem` for OpenSSH, or `.ppk` if using PuTTY directly |

For Windows PowerShell, Windows Terminal, Git Bash, macOS, or Linux with OpenSSH, `.pem` is generally suitable.

Download and securely store the private key file.

> **Important:** AWS allows the private key material to be downloaded only when the key pair is created. Do not lose this file if SSH access is required.

---

### Why is this step required?

The key pair provides cryptographic authentication for SSH access to the EC2 instance.

It is safer than traditional password-based remote login.

---

### Dependency

This step depends on:

- Step 1: The correct AWS Region must be selected because EC2 key pairs are Region-specific.

---

### What happens if this step is skipped?

If no alternative connection method such as AWS Systems Manager Session Manager is configured:

- You cannot establish an SSH session with the EC2 instance.
- You cannot manually install Apache, PHP, or the MySQL client.
- You cannot manually configure and troubleshoot the website.

---

# Step 9: Launch the Initial EC2 Web Server

The initial EC2 instance will be manually configured with:

- Apache web server.
- PHP.
- MySQL client.
- PHP MySQL extension.
- PHP website files.

After successful configuration and testing, a custom AMI will be created from this instance.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Launch instances
~~~

---

### Configuration: Name and Tags

| Setting | Value |
| ------- | ----- |
| Name | `ABC-PHP-Initial-Server` |

---

### Configuration: Application and OS Images

Use Amazon Linux.

Recommended selection:

| Setting | Value |
| ------- | ----- |
| AMI | `Amazon Linux 2023 AMI` |
| Architecture | `64-bit (x86)` |

The exact AMI ID changes by:

- AWS Region.
- Architecture.
- Time.
- AWS AMI updates.

Therefore, select the current AWS-provided Amazon Linux 2023 AMI rather than copying a hard-coded AMI ID.

---

### Configuration: Instance Type

Choose a small instance type suitable for the lab and your account's current Free Tier or promotional credit plan.

Example:

| Setting | Value |
| ------- | ----- |
| Instance type | `t3.micro` if suitable and available under your account plan |

Check the AWS Console for eligibility and pricing indicators before launching.

---

### Configuration: Key Pair

Select:

| Setting | Value |
| ------- | ----- |
| Key pair | `ABC-PHP-Key` |

Or select the existing key pair you identified in Step 8.

---

### Configuration: Network Settings

Click **Edit** and configure:

| Setting | Value |
| ------- | ----- |
| VPC | Default VPC identified in Step 2 |
| Subnet | One of the public subnets identified in Step 2 |
| Auto-assign public IP | `Enable` |
| Firewall | `Select existing security group` |
| Security group | `ABC-EC2-SG` |

Do not create another unnecessary security group during instance launch.

---

### Configuration: Storage

Use a small root EBS volume suitable for the lab.

Example:

| Setting | Value |
| ------- | ----- |
| Volume type | `gp3` |
| Size | `8 GiB` |

Use the minimum practical configuration allowed by the selected AMI and your requirements.

---

### Launch the Instance

Review the configuration and click:

~~~text
Launch instance
~~~

Wait until:

| Check | Expected State |
| ----- | -------------- |
| Instance state | `Running` |
| Status checks | `2/2 checks passed` |

---

### Why is this step required?

This initial EC2 instance serves as the source web server.

It will be used to:

1. Install Apache.
2. Install PHP.
3. Install the MySQL client.
4. Connect to RDS.
5. Create and verify database resources.
6. Deploy the PHP website.
7. Test application functionality.
8. Create a reusable custom AMI.

---

### Dependency

This step depends on:

- Step 2: VPC and public subnet identified.
- Step 4: `ABC-EC2-SG` created.
- Step 8: EC2 key pair created or identified.

---

### What happens if this step is skipped?

Without the initial EC2 instance:

- The PHP application cannot be configured.
- RDS connectivity cannot be tested from the application tier.
- A configured custom AMI cannot be created.
- The Launch Template will not have the prepared application image needed by Auto Scaling.

---

# Step 10: Connect to the Initial EC2 Instance Using SSH

First obtain the instance's public IPv4 address.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select ABC-PHP-Initial-Server
→ Details
→ Public IPv4 address
~~~

Example:

~~~text
13.233.100.25
~~~

Your actual IP address will be different.

---

### Command

For Windows PowerShell, Windows Terminal, Git Bash, macOS, or Linux with OpenSSH:

~~~bash
ssh -i "ABC-PHP-Key.pem" ec2-user@YOUR_EC2_PUBLIC_IP
~~~

Example only:

~~~bash
ssh -i "ABC-PHP-Key.pem" ec2-user@13.233.100.25
~~~

Replace the example IP address with your actual EC2 public IPv4 address.

---

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `ssh` | Starts the Secure Shell client used to establish an encrypted remote terminal connection. |
| `-i` | Specifies the identity file containing the private SSH key. |
| `"ABC-PHP-Key.pem"` | The private key file corresponding to the EC2 key pair selected during instance launch. |
| `ec2-user` | The default operating-system username for Amazon Linux. |
| `@` | Separates the SSH username from the destination hostname or IP address. |
| `YOUR_EC2_PUBLIC_IP` | Placeholder that must be replaced with the actual public IPv4 address of the EC2 instance. |

If prompted:

~~~text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
~~~

Enter:

~~~text
yes
~~~

---

### Expected Output

A successful connection should display an Amazon Linux shell prompt similar to:

~~~text
[ec2-user@ip-172-31-x-x ~]$
~~~

---

### Why is this step required?

SSH provides remote command-line access to the EC2 instance so that the required software and website configuration can be installed manually.

---

### Dependency

This step depends on:

- Step 9: EC2 instance running.
- The instance must have a reachable public IPv4 address.
- `ABC-EC2-SG` must allow SSH port `22` from your current public IP.
- The correct private key file must be available.

---

### What happens if this step is skipped?

Without terminal access or another management mechanism:

- Apache cannot be manually installed.
- PHP cannot be manually installed.
- MySQL connectivity cannot be manually tested.
- Website files cannot be manually configured.

---

# Step 11: Update the EC2 Package Metadata

Run:

~~~bash
sudo dnf update -y
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the following command with elevated administrative privileges. |
| `dnf` | The package manager used by Amazon Linux 2023. |
| `update` | Updates installed packages and package metadata according to available repositories. |
| `-y` | Automatically answers `yes` to package manager confirmation prompts. |

### Why is this step required?

Updating package metadata ensures that subsequent package installation commands use currently available packages and dependencies from configured repositories.

### Dependency

This step depends on:

- Step 10: Successful terminal connection to the EC2 instance.

### What happens if this step is skipped?

Package installation may still work, but:

- Package metadata may be outdated.
- Older package versions may remain installed.
- Dependency resolution may be less reliable.

---

# Step 12: Install Apache, PHP, PHP MySQL Extension, and MariaDB Client

Amazon Linux 2023 can use the MariaDB client to connect to a MySQL-compatible RDS database.

Run:

~~~bash
sudo dnf install -y httpd php php-mysqli mariadb105
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the package installation with administrative privileges. |
| `dnf` | Amazon Linux 2023 package manager. |
| `install` | Instructs `dnf` to install one or more software packages. |
| `-y` | Automatically confirms installation prompts. |
| `httpd` | Apache HTTP Server package used to serve the PHP website. |
| `php` | PHP runtime required to execute `.php` application files. |
| `php-mysqli` | PHP extension that allows PHP applications to communicate with MySQL-compatible databases. |
| `mariadb105` | MariaDB 10.5 client package that provides a MySQL-compatible command-line client. |

---

### Verify Apache Installation

Run:

~~~bash
httpd -v
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `httpd` | Invokes the Apache HTTP Server executable. |
| `-v` | Displays the installed Apache version information. |

### Expected Output

Output should be similar to:

~~~text
Server version: Apache/2.4.x
Server built:   ...
~~~

---

### Verify PHP Installation

Run:

~~~bash
php -v
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `php` | Invokes the PHP command-line interpreter. |
| `-v` | Displays the installed PHP version. |

### Expected Output

Output should be similar to:

~~~text
PHP 8.x.x (cli)
Copyright (c) The PHP Group
~~~

---

### Verify MySQL-Compatible Client Installation

Run:

~~~bash
mysql --version
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `mysql` | Invokes the MySQL-compatible command-line database client supplied by the installed MariaDB client package. |
| `--version` | Displays the installed client version. |

### Expected Output

Output should contain version information for the MariaDB/MySQL-compatible client.

---

### Why is this step required?

Each package serves a specific application-layer function:

| Package | Purpose |
| ------- | ------- |
| `httpd` | Serves the website over HTTP. |
| `php` | Executes PHP application code. |
| `php-mysqli` | Allows PHP to connect to MySQL. |
| `mariadb105` | Provides a command-line client for testing and managing the RDS MySQL database. |

---

### Dependency

This step depends on:

- Step 11: Package metadata updated.

---

### What happens if this step is skipped?

Depending on the missing package:

- Without Apache, the website cannot be served.
- Without PHP, PHP files cannot execute correctly.
- Without `php-mysqli`, the PHP website cannot connect to MySQL.
- Without the database client, command-line database connectivity and SQL setup cannot be performed from EC2.

---

# Step 13: Start and Enable the Apache Web Server

Run:

~~~bash
sudo systemctl start httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Runs the command with administrative privileges. |
| `systemctl` | Controls services managed by the systemd service manager. |
| `start` | Starts the specified service immediately. |
| `httpd` | The Apache HTTP Server service. |

Now enable Apache to start automatically after reboot:

~~~bash
sudo systemctl enable httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Runs the command with administrative privileges. |
| `systemctl` | Manages systemd services. |
| `enable` | Configures the specified service to start automatically during system boot. |
| `httpd` | The Apache HTTP Server service. |

Verify the service:

~~~bash
sudo systemctl status httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the command with administrative privileges. |
| `systemctl` | Manages systemd services. |
| `status` | Displays the current runtime state and recent service information. |
| `httpd` | The Apache service being inspected. |

### Expected Output

Look for:

~~~text
Active: active (running)
~~~

Press `q` if the status output opens in a pager.

---

### Why is this step required?

Installing Apache does not necessarily mean the service is running.

The website requires Apache to:

- Listen for incoming HTTP requests.
- Process PHP website requests.
- Serve responses to users and the load balancer.

Enabling the service ensures that EC2 instances created from the later custom AMI can start Apache automatically after boot.

---

### Dependency

This step depends on:

- Step 12: Apache must be installed.

---

### What happens if this step is skipped?

If Apache is not started:

- HTTP requests fail.
- Target group health checks fail.
- The Application Load Balancer marks instances unhealthy.

If Apache is not enabled:

- The website may stop working after an instance reboot.

---

# Step 14: Test the Initial Apache Web Server

Create a temporary test page:

~~~bash
echo '<h1>ABC PHP Web Server is Running</h1>' | sudo tee /var/www/html/index.html
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `echo` | Produces the specified text as standard output. |
| `'<h1>ABC PHP Web Server is Running</h1>'` | HTML heading content that will appear in the browser. |
| `\|` | Pipe operator. Sends the output from `echo` to the command on its right. |
| `sudo` | Executes the following command with administrative privileges. |
| `tee` | Reads standard input and writes it to the specified file. |
| `/var/www/html/index.html` | Default Apache website file path used for the temporary test page. |

If you added the temporary HTTP rule from `My IP` in Step 4, open:

~~~text
http://YOUR_EC2_PUBLIC_IP
~~~

Expected browser output:

~~~text
ABC PHP Web Server is Running
~~~

> **Important:** Direct browser testing requires an EC2 inbound HTTP rule that permits your current public IP. If only the future ALB security group is permitted, direct browser access to the EC2 public IP will not work, which is expected.

---

### Why is this step required?

This confirms that:

- Apache is installed.
- Apache is running.
- The web root is correct.
- The EC2 instance can serve HTTP content.

---

### Dependency

This step depends on:

- Step 13: Apache running.
- A suitable EC2 security group rule if direct browser testing is performed.

---

### What happens if this step is skipped?

You lose an early diagnostic checkpoint. Later website failures become harder to isolate because you will not know whether the issue originates from:

- Apache.
- PHP.
- Security groups.
- Database connectivity.
- Application code.

---

# Step 15: Connect from EC2 to the RDS MySQL Database

Use the RDS endpoint recorded in Step 7.

Run:

~~~bash
mysql -h YOUR_RDS_ENDPOINT -P 3306 -u admin -p
~~~

Example only:

~~~bash
mysql -h abc-mysql-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p
~~~

When prompted:

~~~text
Enter password:
~~~

Enter:

~~~text
intel123
~~~

The password will not appear on the screen while you type. This is normal terminal behavior.

---

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `mysql` | Starts the MySQL-compatible command-line client. |
| `-h` | Specifies the database server hostname. |
| `YOUR_RDS_ENDPOINT` | Placeholder for the actual Amazon RDS endpoint. |
| `-P` | Specifies the TCP port used for the database connection. The uppercase `P` is significant. |
| `3306` | Default MySQL database port. |
| `-u` | Specifies the database username. |
| `admin` | RDS master username configured during database creation. |
| `-p` | Prompts securely for the database password instead of exposing it directly in command history. |

---

### Expected Output

A successful connection should display something similar to:

~~~text
Welcome to the MariaDB monitor.
Commands end with ; or \g.
Your MySQL connection id is ...
Server version: 8.x.x Source distribution

MySQL [(none)]>
~~~

The exact prompt may vary depending on the installed client.

---

### Why is this step required?

This directly verifies the network and authentication path:

~~~text
EC2 Instance
    |
    | ABC-EC2-SG
    |
    | TCP 3306
    v
ABC-RDS-SG
    |
    v
RDS MySQL
~~~

A successful connection proves that:

- The RDS endpoint resolves.
- Network routing works.
- Port `3306` is permitted.
- The RDS security group correctly trusts `ABC-EC2-SG`.
- The database username and password are valid.

---

### Dependency

This step depends on:

- Step 6: RDS instance available.
- Step 7: RDS endpoint recorded.
- Step 9: EC2 instance running.
- Step 12: MySQL-compatible client installed.
- Step 5: RDS security group allows MySQL from `ABC-EC2-SG`.

---

### What happens if this step is skipped?

Without testing database connectivity:

- PHP application errors may be incorrectly blamed on PHP code.
- Security group issues may remain undetected.
- Incorrect credentials or endpoints may not be discovered before application deployment.

---

# Step 16: Verify and Create the `intel` Database

After connecting to MySQL, run:

~~~sql
SHOW DATABASES;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `SHOW` | SQL statement used to display information about database objects. |
| `DATABASES` | Requests the list of databases visible to the connected MySQL user. |
| `;` | Terminates the SQL statement and tells the MySQL client to execute it. |

### Expected Output

If `intel` was successfully created through the RDS **Initial database name** setting, output should include:

~~~text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| intel              |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
~~~

If `intel` does not exist, create it:

~~~sql
CREATE DATABASE intel;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `CREATE` | SQL command used to create a new database object. |
| `DATABASE` | Specifies that the object being created is a database. |
| `intel` | Required database name from the project assignment. |
| `;` | Terminates and executes the SQL statement. |

Expected output:

~~~text
Query OK, 1 row affected
~~~

---

### Why is this step required?

The assignment explicitly requires:

~~~text
Database name: intel
~~~

The `intel` database provides the logical container in which the required `data` table will be created.

---

### Dependency

This step depends on:

- Step 15: Successful connection from EC2 to RDS.

---

### What happens if this step is skipped?

Without the `intel` database:

- The required `data` table cannot be created in the correct database.
- The website cannot select the intended application database.
- The assignment requirement remains incomplete.

---

# Step 17: Select the `intel` Database

Run:

~~~sql
USE intel;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `USE` | Selects a database as the active database for subsequent SQL statements. |
| `intel` | The required database name. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

~~~text
Database changed
~~~

---

### Why is this step required?

The table must be created inside the `intel` database. Selecting the database ensures that subsequent table operations target the correct database.

---

### Dependency

This step depends on:

- Step 16: The `intel` database must exist.

---

### What happens if this step is skipped?

The `CREATE TABLE` command may:

- Fail because no database is selected.
- Create a table in the wrong database if another database is active.

---

# Step 18: Create the Required `data` Table

The assignment specifies the table name but does not define its column structure.

For this project, create a simple table suitable for testing PHP-to-RDS connectivity:

~~~sql
CREATE TABLE data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `CREATE TABLE` | Creates a new table in the currently selected database. |
| `data` | Required table name specified by the assignment. |
| `(` | Begins the table column definitions. |
| `id` | Name of the unique identifier column. |
| `INT` | Stores integer values. |
| `AUTO_INCREMENT` | Automatically generates the next numeric identifier for each inserted row. |
| `PRIMARY KEY` | Makes `id` the unique primary identifier of each table row. |
| `,` | Separates one column definition from the next. |
| `name` | Column used to store a sample name or text value. |
| `VARCHAR(100)` | Variable-length text field with a maximum length of 100 characters. |
| `NOT NULL` | Requires every row to contain a value for the `name` column. |
| `created_at` | Column used to record when a row was created. |
| `TIMESTAMP` | Stores date and time information. |
| `DEFAULT CURRENT_TIMESTAMP` | Automatically inserts the current database timestamp when a row is created. |
| `)` | Ends the table definition. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

~~~text
Query OK, 0 rows affected
~~~

---

### Why is this step required?

The assignment explicitly requires:

~~~text
Table name: data
~~~

The PHP application will use this table to demonstrate successful communication between the web tier and database tier.

---

### Dependency

This step depends on:

- Step 17: The `intel` database selected.

---

### What happens if this step is skipped?

Without the `data` table:

- The assignment requirement is incomplete.
- The PHP application cannot query the expected table.
- End-to-end database verification cannot be completed.

---

# Step 19: Insert Sample Data into the Table

Insert initial records:

~~~sql
INSERT INTO data (name) VALUES ('AWS Multi-Tier Website');
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `INSERT INTO` | SQL command used to add a new row to a table. |
| `data` | Destination table. |
| `(name)` | Specifies that a value will be supplied for the `name` column. |
| `VALUES` | Introduces the values that will be inserted. |
| `('AWS Multi-Tier Website')` | Text value inserted into the `name` column. |
| `;` | Terminates and executes the SQL statement. |

Insert a second record:

~~~sql
INSERT INTO data (name) VALUES ('Connected to Amazon RDS MySQL');
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `INSERT INTO` | Adds a new row to a table. |
| `data` | Destination table name. |
| `(name)` | Specifies the target column. |
| `VALUES` | Introduces the value to insert. |
| `('Connected to Amazon RDS MySQL')` | Sample text proving the purpose of the database connection. |
| `;` | Terminates and executes the SQL statement. |

---

### Why is this step required?

Although the assignment only explicitly requires creating the database and table, inserting test data provides meaningful end-to-end verification.

The PHP website will later retrieve these rows from RDS.

---

### Dependency

This step depends on:

- Step 18: The `data` table must exist.

---

### What happens if this step is skipped?

The table still exists, but:

- The PHP website has no records to display.
- Visual verification of database reads becomes less meaningful.

---

# Step 20: Verify the Database and Table

Run:

~~~sql
SHOW TABLES;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `SHOW` | Displays information about database objects. |
| `TABLES` | Requests all tables in the currently selected database. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

~~~text
+-----------------+
| Tables_in_intel |
+-----------------+
| data            |
+-----------------+
~~~

Now inspect the table structure:

~~~sql
DESCRIBE data;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `DESCRIBE` | Displays the column structure and metadata of a database table. |
| `data` | Name of the table being inspected. |
| `;` | Terminates and executes the SQL statement. |

Expected output should show columns similar to:

~~~text
+------------+--------------+------+-----+-------------------+-------------------+
| Field      | Type         | Null | Key | Default           | Extra             |
+------------+--------------+------+-----+-------------------+-------------------+
| id         | int          | NO   | PRI | NULL              | auto_increment    |
| name       | varchar(100) | NO   |     | NULL              |                   |
| created_at | timestamp    | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
+------------+--------------+------+-----+-------------------+-------------------+
~~~

Now retrieve the inserted records:

~~~sql
SELECT * FROM data;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `SELECT` | Retrieves data from a database table. |
| `*` | Wildcard representing all columns. |
| `FROM` | Specifies the source table. |
| `data` | Name of the table from which rows are retrieved. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

~~~text
+----+--------------------------------+---------------------+
| id | name                           | created_at          |
+----+--------------------------------+---------------------+
|  1 | AWS Multi-Tier Website         | YYYY-MM-DD HH:MM:SS |
|  2 | Connected to Amazon RDS MySQL  | YYYY-MM-DD HH:MM:SS |
+----+--------------------------------+---------------------+
~~~

The exact timestamps will differ.

---

### Why is this step required?

This verifies that:

- The `intel` database exists.
- The `data` table exists.
- The required table structure is valid.
- Data can be successfully written to RDS.
- Data can be successfully retrieved from RDS.

---

### Dependency

This step depends on:

- Step 18: Table created.
- Step 19: Sample records inserted.

---

### What happens if this step is skipped?

Without verification, database creation errors or incorrect table names may remain undetected until the PHP application is deployed.

---

# Step 21: Exit the MySQL Client

Run:

~~~sql
exit;
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `exit` | Terminates the current MySQL client session. |
| `;` | Completes the command. |

### Expected Output

~~~text
Bye
~~~

You should return to the EC2 Linux shell prompt:

~~~text
[ec2-user@ip-172-31-x-x ~]$
~~~

---

### Why is this step required?

Database creation and verification are complete. The next tasks require Linux shell commands to create and configure the PHP website.

---

### Dependency

This step depends on:

- Completion of the required SQL operations.

---

### What happens if this step is skipped?

You remain inside the MySQL client, and Linux shell commands entered there will fail as invalid SQL statements.

---

# Resources Created and Configured So Far

At the end of Part 2, the following resources and configurations should exist:

| Resource or Configuration | Expected State |
| ------------------------- | -------------- |
| `ABC-ALB-SG` | Created |
| `ABC-EC2-SG` | Created |
| `ABC-RDS-SG` | Created |
| `abc-mysql-db` | `Available` |
| RDS endpoint | Recorded |
| RDS public access | `No` |
| RDS MySQL port | `3306` |
| Initial EC2 instance | `Running` |
| Apache | Installed, enabled, and running |
| PHP | Installed |
| PHP MySQL extension | Installed |
| MySQL-compatible client | Installed |
| Database `intel` | Created |
| Table `data` | Created |
| Sample records | Inserted and verified |

---

# Part 3: Deploy the PHP Website, Configure RDS Connectivity, Create the Custom AMI, and Prepare Auto Scaling Resources

# Step 22: Create the PHP Website with RDS Database Connectivity

The initial EC2 instance already has:

- Apache installed and running.
- PHP installed.
- The `php-mysqli` extension installed.
- Network connectivity to the RDS MySQL database.
- The `intel` database created.
- The `data` table created.
- Sample records inserted.

The next step is to replace the temporary static HTML page with a PHP website that connects to Amazon RDS and retrieves records from the `data` table.

The application will use the following database configuration:

| Setting | Value |
| ------- | ----- |
| Database host | Actual RDS endpoint |
| Database username | `admin` |
| Database password | `intel123` |
| Database name | `intel` |
| Database table | `data` |
| Database port | `3306` |

> **Important:** Replace `YOUR_RDS_ENDPOINT` in the PHP code with the actual RDS endpoint recorded in Step 7.

---

### Command

First, remove the temporary Apache test page created earlier:

~~~bash
sudo rm -f /var/www/html/index.html
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the following command with administrative privileges. |
| `rm` | Removes files or directories. |
| `-f` | Forces removal without asking for confirmation and does not report an error if the file does not exist. |
| `/var/www/html/index.html` | Absolute path of the temporary Apache test page created previously. |

---

### Command

Create the PHP application file:

~~~bash
sudo tee /var/www/html/index.php > /dev/null <<'EOF'
<?php
$servername = "YOUR_RDS_ENDPOINT";
$username = "admin";
$password = "intel123";
$dbname = "intel";
$port = 3306;

$conn = new mysqli($servername, $username, $password, $dbname, $port);

if ($conn->connect_error) {
    die("Database connection failed: " . $conn->connect_error);
}

$sql = "SELECT id, name, created_at FROM data ORDER BY id ASC";
$result = $conn->query($sql);

$hostname = gethostname();
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ABC Multi-Tier AWS Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f6f8;
            margin: 0;
            padding: 40px;
        }

        .container {
            max-width: 900px;
            margin: auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        h1 {
            text-align: center;
        }

        .success {
            background-color: #e8f5e9;
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 5px;
        }

        .server-info {
            background-color: #e3f2fd;
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 5px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        th,
        td {
            border: 1px solid #dddddd;
            padding: 12px;
            text-align: left;
        }

        th {
            background-color: #eeeeee;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ABC Multi-Tier Website on AWS</h1>

        <div class="success">
            <strong>Success:</strong> PHP successfully connected to Amazon RDS MySQL.
        </div>

        <div class="server-info">
            <strong>Serving EC2 Hostname:</strong>
            <?php echo htmlspecialchars($hostname); ?>
        </div>

        <h2>Records from the intel.data Table</h2>

        <table>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Created At</th>
            </tr>

            <?php
            if ($result && $result->num_rows > 0) {
                while ($row = $result->fetch_assoc()) {
                    echo "<tr>";
                    echo "<td>" . htmlspecialchars($row["id"]) . "</td>";
                    echo "<td>" . htmlspecialchars($row["name"]) . "</td>";
                    echo "<td>" . htmlspecialchars($row["created_at"]) . "</td>";
                    echo "</tr>";
                }
            } else {
                echo "<tr><td colspan='3'>No records found.</td></tr>";
            }
            ?>
        </table>
    </div>
</body>
</html>
<?php
$conn->close();
?>
EOF
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the following command with administrative privileges because `/var/www/html` normally requires elevated permissions for writing files. |
| `tee` | Reads standard input and writes it into the specified destination file. |
| `/var/www/html/index.php` | Destination PHP application file in Apache's default document root. |
| `>` | Shell output-redirection operator. |
| `/dev/null` | Special Linux device that discards output. This prevents `tee` from printing the complete PHP file contents to the terminal. |
| `<<` | Starts a shell here-document, allowing multiple lines of content to be supplied as standard input to `tee`. |
| `'EOF'` | Quoted here-document delimiter. Quoting prevents the shell from expanding PHP variables such as `$servername`, `$username`, and `$conn` while creating the file. |
| `<?php` | Begins a PHP code block. |
| `$servername` | PHP variable containing the RDS database hostname. |
| `"YOUR_RDS_ENDPOINT"` | Placeholder that must be replaced with the actual RDS endpoint. |
| `$username` | PHP variable containing the database username. |
| `"admin"` | RDS master username configured previously. |
| `$password` | PHP variable containing the database password. |
| `"intel123"` | Database password required by the assignment. |
| `$dbname` | PHP variable containing the target database name. |
| `"intel"` | Required MySQL database name. |
| `$port` | PHP variable containing the MySQL TCP port. |
| `3306` | Default MySQL network port. |
| `new mysqli(...)` | Creates a new PHP MySQLi database connection object. |
| `if ($conn->connect_error)` | Checks whether the database connection failed. |
| `die(...)` | Stops PHP execution and displays the specified error message. |
| `$sql` | Stores the SQL query executed against the RDS database. |
| `SELECT` | SQL command used to retrieve records. |
| `id, name, created_at` | Columns retrieved from the table. |
| `FROM data` | Specifies the required `data` table as the source. |
| `ORDER BY id ASC` | Sorts records by `id` in ascending order. |
| `$conn->query($sql)` | Executes the SQL query using the active database connection. |
| `gethostname()` | Returns the operating-system hostname of the EC2 instance serving the request. |
| `htmlspecialchars(...)` | Converts special HTML characters into safe HTML entities before displaying values. |
| `$result->fetch_assoc()` | Retrieves each database row as an associative array. |
| `$conn->close()` | Closes the MySQL database connection after processing is complete. |
| `EOF` | Ends the here-document input. |

---

### Why is this step required?

This step creates the actual PHP application layer.

The application proves that the EC2 web server can:

1. Execute PHP.
2. Connect to the RDS endpoint.
3. Authenticate using database credentials.
4. Select the `intel` database.
5. Query the `data` table.
6. Display database records in a browser.
7. Display the EC2 hostname serving each request.

Displaying the EC2 hostname will later help verify that the Application Load Balancer is distributing requests among multiple EC2 instances.

---

### Dependency

This step depends on:

- Step 12: PHP and `php-mysqli` installed.
- Step 13: Apache running.
- Step 16: `intel` database created.
- Step 18: `data` table created.
- Step 19: Sample records inserted.
- Step 7: RDS endpoint recorded.

---

### What happens if this step is skipped?

Without the PHP application:

- The website cannot demonstrate RDS connectivity.
- The RDS hostname cannot be configured in application code.
- End-to-end multi-tier functionality cannot be verified.
- The custom AMI would not contain the required website.

---

# Step 23: Change the Database Hostname in the Website

The assignment explicitly requires changing the hostname in the website.

The placeholder:

~~~text
YOUR_RDS_ENDPOINT
~~~

must be replaced with the actual Amazon RDS endpoint.

Suppose your actual endpoint is:

~~~text
abc-mysql-db.abcdefghijkl.ap-south-1.rds.amazonaws.com
~~~

Run:

~~~bash
sudo sed -i 's|YOUR_RDS_ENDPOINT|abc-mysql-db.abcdefghijkl.ap-south-1.rds.amazonaws.com|g' /var/www/html/index.php
~~~

Replace the example endpoint with your actual RDS endpoint.

---

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Runs the command with administrative privileges. |
| `sed` | Stream editor used to search and modify text. |
| `-i` | Modifies the specified file directly in place. |
| `'s|YOUR_RDS_ENDPOINT|actual-endpoint|g'` | Substitution expression that replaces the placeholder with the actual RDS hostname. |
| `s` | Indicates a substitution operation. |
| `\|` | Used here as the separator between the search text, replacement text, and substitution flag. |
| `YOUR_RDS_ENDPOINT` | Placeholder text to locate. |
| `abc-mysql-db.abcdefghijkl.ap-south-1.rds.amazonaws.com` | Example replacement hostname. This must be replaced with the actual RDS endpoint. |
| `g` | Global replacement flag, replacing every matching occurrence on each processed line. |
| `/var/www/html/index.php` | PHP application file being modified. |

---

### Verify the Hostname Replacement

Run:

~~~bash
grep 'servername' /var/www/html/index.php
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `grep` | Searches text for lines matching a specified pattern. |
| `'servername'` | Search pattern. |
| `/var/www/html/index.php` | File being searched. |

### Expected Output

~~~text
$servername = "abc-mysql-db.abcdefghijkl.ap-south-1.rds.amazonaws.com";
$conn = new mysqli($servername, $username, $password, $dbname, $port);
~~~

Your actual endpoint will be different.

---

### Why is this step required?

The PHP application cannot connect to RDS using a placeholder.

It needs the actual DNS hostname assigned to the RDS database instance.

This directly satisfies the assignment requirement:

~~~text
Change hostname in website
~~~

---

### Dependency

This step depends on:

- Step 22: PHP website created.
- Step 7: Actual RDS endpoint available.

---

### What happens if this step is skipped?

The PHP application will attempt to resolve the literal hostname:

~~~text
YOUR_RDS_ENDPOINT
~~~

The connection will fail because that is not the actual RDS DNS endpoint.

---

# Step 24: Verify PHP Syntax

Before testing the website, validate the PHP file syntax.

Run:

~~~bash
php -l /var/www/html/index.php
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `php` | Invokes the PHP command-line interpreter. |
| `-l` | Performs syntax checking without executing the PHP application. The lowercase letter is `l`, meaning lint. |
| `/var/www/html/index.php` | PHP file being checked. |

### Expected Output

~~~text
No syntax errors detected in /var/www/html/index.php
~~~

---

### Why is this step required?

A syntax error can prevent the entire PHP page from executing.

Checking syntax before browser testing isolates application-code problems early.

---

### Dependency

This step depends on:

- Step 22: PHP file created.
- Step 23: RDS hostname configured.

---

### What happens if this step is skipped?

A PHP syntax problem may later appear as:

- A blank page.
- HTTP `500 Internal Server Error`.
- A PHP error.
- Failed target group health checks.

---

# Step 25: Restart Apache After PHP Configuration

Restart Apache:

~~~bash
sudo systemctl restart httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the command with administrative privileges. |
| `systemctl` | Controls services managed by systemd. |
| `restart` | Stops and immediately starts the specified service. |
| `httpd` | Apache HTTP Server service. |

Verify Apache:

~~~bash
sudo systemctl is-active httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Runs the command with administrative privileges. |
| `systemctl` | Manages systemd services. |
| `is-active` | Checks whether the specified service is currently active. |
| `httpd` | Apache service being checked. |

### Expected Output

~~~text
active
~~~

---

### Why is this step required?

Restarting Apache ensures that the web server is running correctly after application configuration and package installation.

---

### Dependency

This step depends on:

- Step 13: Apache installed and enabled.
- Step 22: PHP application created.

---

### What happens if this step is skipped?

The website may still work, but restarting Apache provides a clean validation that the web server can successfully start with the installed PHP environment.

---

# Step 26: Test the PHP Website Locally from the EC2 Instance

Run:

~~~bash
curl http://localhost
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `curl` | Command-line tool used to send requests to URLs and display responses. |
| `http://` | Specifies the HTTP protocol. |
| `localhost` | Refers to the current EC2 instance itself through the local loopback interface. |

### Expected Output

The output should contain HTML including:

~~~text
ABC Multi-Tier Website on AWS
Success: PHP successfully connected to Amazon RDS MySQL.
AWS Multi-Tier Website
Connected to Amazon RDS MySQL
~~~

The HTML output will also contain the EC2 hostname.

---

### Why is this step required?

This verifies the complete application path:

~~~text
curl
  |
  v
Apache
  |
  v
PHP
  |
  v
MySQLi
  |
  v
RDS Endpoint
  |
  v
intel Database
  |
  v
data Table
  |
  v
Database Records Returned
~~~

A successful response confirms that the application works before creating the custom AMI.

---

### Dependency

This step depends on:

- Step 22: PHP application created.
- Step 23: Correct RDS hostname configured.
- Step 25: Apache running.
- RDS connectivity working.

---

### What happens if this step is skipped?

A broken application could be captured into the custom AMI and automatically replicated across multiple EC2 instances.

That would multiply the configuration problem across the Auto Scaling group.

---

# Step 27: Test the Website from a Browser

If the temporary HTTP inbound rule from `My IP` was added to `ABC-EC2-SG`, obtain the initial EC2 instance's public IPv4 address.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select ABC-PHP-Initial-Server
→ Details
→ Public IPv4 address
~~~

Open in a browser:

~~~text
http://YOUR_EC2_PUBLIC_IP
~~~

Example only:

~~~text
http://13.233.100.25
~~~

Do not use the example IP address. Use the actual public IPv4 address of your EC2 instance.

### Expected Browser Content

You should observe:

~~~text
ABC Multi-Tier Website on AWS

Success: PHP successfully connected to Amazon RDS MySQL.

Serving EC2 Hostname: ip-172-31-x-x.ap-south-1.compute.internal

Records from the intel.data Table

1    AWS Multi-Tier Website          YYYY-MM-DD HH:MM:SS
2    Connected to Amazon RDS MySQL   YYYY-MM-DD HH:MM:SS
~~~

The exact hostname and timestamps will differ.

---

### Why is this step required?

This verifies that the application works through the EC2 network interface and browser, not only through `localhost`.

---

### Dependency

This step depends on:

- Step 26: Successful local application test.
- EC2 instance having a public IPv4 address.
- Temporary HTTP access from `My IP` being permitted by `ABC-EC2-SG`.

---

### What happens if this step is skipped?

The application can still be tested later through the Application Load Balancer, but direct browser testing provides an additional troubleshooting checkpoint before creating the AMI.

---

# Step 28: Verify Apache Starts Automatically at Boot

Run:

~~~bash
sudo systemctl is-enabled httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the command with administrative privileges. |
| `systemctl` | Manages systemd services. |
| `is-enabled` | Checks whether the service is configured to start automatically during system boot. |
| `httpd` | Apache service being checked. |

### Expected Output

~~~text
enabled
~~~

If the output is not `enabled`, run:

~~~bash
sudo systemctl enable httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `sudo` | Executes the command with administrative privileges. |
| `systemctl` | Controls systemd services. |
| `enable` | Configures automatic service startup during system boot. |
| `httpd` | Apache service to enable. |

---

### Why is this step required?

Auto Scaling will launch new EC2 instances from the custom AMI.

Each new instance must automatically start Apache without manual intervention.

---

### Dependency

This step depends on:

- Apache being installed.

---

### What happens if this step is skipped?

New Auto Scaling instances may boot successfully but fail to serve the website because Apache is not running.

The target group would then mark those instances as unhealthy.

---

# Step 29: Create a Health Check Endpoint

Create a dedicated health check page:

~~~bash
echo 'OK' | sudo tee /var/www/html/health.html
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `echo` | Outputs the specified text. |
| `'OK'` | Simple text response returned by the health check page. |
| `\|` | Pipe operator that sends the output from `echo` to `tee`. |
| `sudo` | Executes `tee` with administrative privileges. |
| `tee` | Writes standard input to a file. |
| `/var/www/html/health.html` | Dedicated static health check file in Apache's document root. |

Test it:

~~~bash
curl http://localhost/health.html
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `curl` | Sends an HTTP request and displays the response. |
| `http://localhost/health.html` | Local URL of the dedicated health check endpoint. |

### Expected Output

~~~text
OK
~~~

---

### Why is this step required?

The Application Load Balancer target group needs a reliable health check path.

Using a static file such as:

~~~text
/health.html
~~~

avoids making target health dependent on successful RDS queries.

This helps distinguish:

- Web-server health.
- Database connectivity health.

---

### Dependency

This step depends on:

- Apache installed and running.

---

### What happens if this step is skipped?

The target group could still use `/`, but database problems could cause instances to be marked unhealthy even when Apache itself is functioning.

---

# Step 30: Prepare the EC2 Instance for AMI Creation

Before creating the custom AMI, perform final checks.

Run:

~~~bash
sudo systemctl is-active httpd
~~~

Expected output:

~~~text
active
~~~

Run:

~~~bash
sudo systemctl is-enabled httpd
~~~

Expected output:

~~~text
enabled
~~~

Run:

~~~bash
php -l /var/www/html/index.php
~~~

Expected output:

~~~text
No syntax errors detected in /var/www/html/index.php
~~~

Run:

~~~bash
curl http://localhost/health.html
~~~

Expected output:

~~~text
OK
~~~

Run:

~~~bash
curl -s http://localhost | grep 'PHP successfully connected'
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `curl` | Sends an HTTP request. |
| `-s` | Silent mode. Suppresses progress information. |
| `http://localhost` | Requests the PHP website from the local Apache server. |
| `\|` | Sends the HTML response from `curl` to `grep`. |
| `grep` | Searches input for matching text. |
| `'PHP successfully connected'` | Text pattern expected when the PHP page successfully connects to RDS. |

### Expected Output

The output should contain:

~~~text
<strong>Success:</strong> PHP successfully connected to Amazon RDS MySQL.
~~~

---

### Why is this step required?

A custom AMI captures the current server configuration.

Any configuration problem present now can be reproduced on every Auto Scaling instance.

Therefore, the source EC2 instance must be verified before image creation.

---

### Dependency

This step depends on:

- PHP website configured.
- Apache running and enabled.
- Health endpoint created.
- RDS connectivity working.

---

### What happens if this step is skipped?

A broken or incomplete server configuration could be replicated across the entire Auto Scaling group.

---

# Step 31: Create a Custom AMI from the Configured EC2 Instance

The custom AMI will preserve the configured web server so Auto Scaling can launch identical instances.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select ABC-PHP-Initial-Server
→ Actions
→ Image and templates
→ Create image
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Image name | `ABC-PHP-Web-AMI` |
| Image description | `Configured Amazon Linux PHP web server connected to RDS MySQL` |
| Reboot instance | Leave the default reboot behavior enabled for filesystem consistency |

> **Important:** The exact checkbox wording may vary. AWS may display an option such as `No reboot`. For a consistent image, do not select `No reboot` unless there is a specific reason.

Click:

~~~text
Create image
~~~

---

### Monitor AMI Status

### Navigation

~~~text
AWS Management Console
→ EC2
→ Images
→ AMIs
→ Owned by me
→ Select ABC-PHP-Web-AMI
~~~

Wait until:

| Property | Expected Value |
| -------- | -------------- |
| AMI state | `Available` |

Do not create the Launch Template using the custom AMI until its state becomes `Available`.

---

### Why is this step required?

The custom AMI contains the configured web server environment, including:

- Amazon Linux operating system.
- Apache.
- PHP.
- PHP MySQL extension.
- Website files.
- RDS hostname configuration.
- Health check page.
- Apache boot configuration.

The Launch Template will reference this AMI so every Auto Scaling instance receives an identical application environment.

---

### Dependency

This step depends on:

- Step 30: Initial EC2 configuration fully verified.

---

### What happens if this step is skipped?

Without a reusable machine image:

- Auto Scaling instances may not contain the PHP website.
- Each new instance would require manual configuration.
- Automatic replacement and scaling would not function as intended.

---

# Step 32: Create the EC2 Launch Template

The Launch Template defines how Auto Scaling launches EC2 instances.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Launch Templates
→ Create launch template
~~~

### Configuration: Launch Template Details

| Setting | Value |
| ------- | ----- |
| Launch template name | `ABC-PHP-Launch-Template` |
| Template version description | `Version 1 - PHP website using custom AMI` |
| Auto Scaling guidance | Select if the Console offers an option indicating the template will be used with EC2 Auto Scaling |

---

### Configuration: Application and OS Images

Choose:

~~~text
My AMIs
→ Owned by me
→ ABC-PHP-Web-AMI
~~~

Verify that the selected AMI is:

~~~text
ABC-PHP-Web-AMI
~~~

Do not accidentally select the original unconfigured Amazon Linux AMI.

---

### Configuration: Instance Type

Choose the same small instance type used for the initial EC2 server or another suitable compatible type.

Example:

| Setting | Value |
| ------- | ----- |
| Instance type | `t3.micro`, if appropriate for your account and architecture |

The selected instance type must be compatible with the architecture of the custom AMI.

---

### Configuration: Key Pair

For this learning lab, select:

| Setting | Value |
| ------- | ----- |
| Key pair | `ABC-PHP-Key` |

Including the key pair allows SSH troubleshooting of Auto Scaling instances if required.

---

### Configuration: Network Settings

For the Launch Template:

| Setting | Value |
| ------- | ----- |
| Subnet | `Don't include in launch template` |
| Firewall | `Select existing security group` |
| Security group | `ABC-EC2-SG` |

Do not hard-code a single subnet into the Launch Template.

The Auto Scaling group will later select multiple subnets in different Availability Zones.

---

### Configuration: Storage

The storage configuration inherited from the custom AMI can normally be retained.

Example:

| Setting | Value |
| ------- | ----- |
| Root volume type | `gp3` |
| Root volume size | Same as the custom AMI source volume unless adjustment is required |

---

### Advanced Details

No user data script is required because:

- Apache is already installed in the custom AMI.
- PHP is already installed.
- The website is already deployed.
- The RDS endpoint is already configured.
- Apache is enabled to start automatically.

Leave unnecessary advanced options at their defaults.

Click:

~~~text
Create launch template
~~~

---

### Why is this step required?

The Auto Scaling group does not manually configure EC2 instances.

It needs a reusable launch definition specifying:

- AMI.
- Instance type.
- Key pair.
- Security group.
- Storage.

The Launch Template provides that reusable EC2 configuration.

---

### Dependency

This step depends on:

- Step 31: `ABC-PHP-Web-AMI` must reach the `Available` state.
- Step 4: `ABC-EC2-SG` must exist.

---

### What happens if this step is skipped?

Without a Launch Template:

- The Auto Scaling group cannot know which AMI to use.
- Instance configuration is undefined.
- Automatic EC2 creation cannot be completed.

---

# Step 33: Create the Application Load Balancer Target Group

The target group defines the EC2 instances that receive traffic from the Application Load Balancer.

The Auto Scaling group will later automatically register its instances with this target group.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Load Balancing
→ Target Groups
→ Create target group
~~~

---

### Configuration: Basic Settings

| Setting | Value |
| ------- | ----- |
| Choose a target type | `Instances` |
| Target group name | `ABC-PHP-TG` |
| Protocol | `HTTP` |
| Port | `80` |
| IP address type | `IPv4` |
| VPC | Default VPC used by the EC2 and RDS resources |
| Protocol version | `HTTP1` |

---

### Configuration: Health Checks

Configure:

| Setting | Value |
| ------- | ----- |
| Health check protocol | `HTTP` |
| Health check path | `/health.html` |

If advanced health check settings are shown, the default values are generally sufficient for this lab.

---

### Register Targets

The Console may display the initial EC2 instance as an available target.

For the final Auto Scaling architecture, manual registration of the initial EC2 instance is not required because the Auto Scaling group will automatically register its managed instances.

Proceed to create the target group without manually registering the initial EC2 instance.

Click:

~~~text
Create target group
~~~

---

### Why is this step required?

The Application Load Balancer sends traffic to a target group rather than directly managing individual EC2 instances.

The target group:

- Tracks registered instances.
- Performs health checks.
- Sends traffic only to healthy targets.
- Integrates with EC2 Auto Scaling.

---

### Dependency

This step depends on:

- Step 2: VPC identified.
- Step 29: `/health.html` included in the custom AMI.
- Step 31: Configured application image created.

---

### What happens if this step is skipped?

Without a target group:

- The Application Load Balancer has no destination for requests.
- Auto Scaling instances cannot be automatically registered as application targets.
- Health checks cannot determine whether instances are ready to receive traffic.

---

# Step 34: Create the Application Load Balancer

The Application Load Balancer distributes incoming HTTP requests across healthy EC2 instances.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Load Balancing
→ Load Balancers
→ Create load balancer
→ Application Load Balancer
→ Create
~~~

---

### Configuration: Basic Configuration

| Setting | Value |
| ------- | ----- |
| Load balancer name | `ABC-PHP-ALB` |
| Scheme | `Internet-facing` |
| IP address type | `IPv4` |

---

### Configuration: Network Mapping

Select:

| Setting | Value |
| ------- | ----- |
| VPC | Default VPC identified in Step 2 |
| Availability Zones | At least two Availability Zones |
| Subnets | Select one public subnet from each selected Availability Zone |

Example:

| Availability Zone | Subnet |
| ----------------- | ------ |
| `ap-south-1a` | Public Subnet A |
| `ap-south-1b` | Public Subnet B |

The exact Availability Zone names depend on your AWS account and Region.

---

### Configuration: Security Groups

Remove the automatically selected `default` security group if present.

Select:

~~~text
ABC-ALB-SG
~~~

---

### Configuration: Listeners and Routing

Configure:

| Setting | Value |
| ------- | ----- |
| Protocol | `HTTP` |
| Port | `80` |
| Default action | Forward to `ABC-PHP-TG` |

Click:

~~~text
Create load balancer
~~~

Wait until the load balancer state becomes:

~~~text
Active
~~~

---

### Why is this step required?

The Application Load Balancer provides one public entry point for the application.

Instead of users connecting directly to individual EC2 instances:

~~~text
User
  |
  v
Application Load Balancer
  |
  +--------> EC2 Instance 1
  |
  +--------> EC2 Instance 2
~~~

The load balancer sends traffic only to registered healthy targets.

---

### Dependency

This step depends on:

- Step 3: `ABC-ALB-SG` created.
- Step 33: `ABC-PHP-TG` created.
- Step 2: At least two suitable public subnets identified.

---

### What happens if this step is skipped?

Without a load balancer:

- Users would need to connect directly to individual EC2 instances.
- There would be no single application endpoint.
- Traffic would not be automatically distributed among multiple instances.
- Instance replacement would make direct IP-based access unreliable.

---

# Step 35: Verify the Application Load Balancer State

### Navigation

~~~text
AWS Management Console
→ EC2
→ Load Balancing
→ Load Balancers
→ Select ABC-PHP-ALB
~~~

Verify:

| Property | Expected Value |
| -------- | -------------- |
| State | `Active` |
| Scheme | `internet-facing` |
| IP address type | `IPv4` |
| Listener | `HTTP:80` |
| Security group | `ABC-ALB-SG` |

Record the load balancer DNS name.

Example:

~~~text
ABC-PHP-ALB-123456789.ap-south-1.elb.amazonaws.com
~~~

Your actual DNS name will be different.

At this stage, opening the ALB DNS name may return an error such as:

~~~text
503 Service Unavailable
~~~

This can be expected because the Auto Scaling group has not yet launched and registered its two managed EC2 instances.

---

### Why is this step required?

The ALB must be active before the Auto Scaling group is connected to its target group.

Recording the DNS name prepares for final application verification.

---

### Dependency

This step depends on:

- Step 34: Application Load Balancer created.

---

### What happens if this step is skipped?

The architecture may still proceed, but load balancer configuration errors may remain unnoticed until final verification.

---

# Resources Created and Configured So Far

At the end of Part 3, the following resources and configurations should exist:

| Resource or Configuration | Expected State |
| ------------------------- | -------------- |
| PHP website | Deployed |
| RDS hostname in PHP | Configured |
| PHP syntax | Valid |
| PHP-to-RDS connection | Successful |
| `/health.html` | Returns `OK` |
| Apache | Active and enabled |
| `ABC-PHP-Web-AMI` | `Available` |
| `ABC-PHP-Launch-Template` | Created |
| `ABC-PHP-TG` | Created |
| `ABC-PHP-ALB` | `Active` |
| ALB DNS name | Recorded |
| Auto Scaling group | Not yet created |

---
