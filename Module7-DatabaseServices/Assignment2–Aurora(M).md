# Module 7 – Database Services

# Assignment 2 – Amazon Aurora

## Problem Statement

You work for XYZ Corporation. Their application requires a SQL service that can store data which can be retrieved when required. Implement a suitable Amazon RDS database engine for the same.

While migrating, you are asked to perform the following tasks:

1. Create an Aurora DB Engine-based RDS database.
2. Create `2` Read Replicas in different Availability Zones for better infrastructure availability.

---

# 1. Free Tier / Cost Check

Before creating any AWS resources, it is important to understand whether this assignment can be completed entirely within the AWS Free Tier and whether the resources may consume AWS promotional credits.

## AWS Services Used in This Assignment

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| Amazon Aurora | Limited / Configuration-dependent | Yes, if usage exceeds applicable Free Tier allowance | Aurora pricing depends on the selected database engine, instance configuration, storage, I/O, and applicable AWS Free Tier benefits. |
| Amazon RDS | Configuration-dependent | Possibly | Aurora is managed through the Amazon RDS service. Charges depend on the selected Aurora configuration. |
| Amazon VPC | Yes | No direct charge for the VPC itself | The default VPC, subnets, route tables, and basic security groups do not have a direct hourly charge. |
| Security Group | Yes | No | Security groups themselves do not have a direct charge. |
| Aurora Storage | Limited / Configuration-dependent | Possibly | Storage charges depend on the selected Aurora configuration and applicable Free Tier allowance. |
| Aurora DB Instances | Limited / Configuration-dependent | Yes | This assignment requires a writer DB instance and `2` reader DB instances. Running multiple DB instances can consume credits or generate charges. |

## Is This Assignment Completely Free Tier Eligible?

**No. The exact assignment architecture should not be assumed to be completely Free Tier eligible.**

The assignment requires:

```text
1 Aurora Writer Instance
        +
2 Aurora Reader Instances
        =
3 Total Aurora DB Instances
```

The architecture is:

```text
                         Application / Client
                                  |
                                  v
                     +-------------------------+
                     |    Aurora DB Cluster    |
                     +-------------------------+
                                  |
               +------------------+------------------+
               |                  |                  |
               v                  v                  v
        +-------------+    +-------------+    +-------------+
        |   Writer    |    |  Reader 1   |    |  Reader 2   |
        | DB Instance |    | Read Replica|    | Read Replica|
        +-------------+    +-------------+    +-------------+
               |                  |                  |
               v                  v                  v
            AZ - 1             AZ - 2             AZ - 3
```

The exact cost depends on:

* The selected Aurora database engine.
* The selected DB instance class or Serverless configuration.
* The AWS Region.
* The number of running DB instances.
* Database storage consumption.
* I/O consumption, depending on the selected storage configuration.
* Backup storage.
* How long the cluster remains running.
* Whether the AWS account has applicable Free Tier benefits or promotional credits.

## Should This Assignment Be Completed While Promotional Credits Are Available?

**Yes.**

Because this assignment requires an Aurora cluster with:

* `1` Writer DB instance.
* `2` Reader DB instances.
* `3` total Aurora DB instances.

It is better to complete this assignment while AWS promotional credits are available.

After successful verification, delete the complete Aurora cluster and associated resources created specifically for this assignment to minimize credit consumption.

> **Important:** Do not leave the Aurora cluster running unnecessarily after completing the assignment. Database instances may continue consuming promotional credits or generating charges while they exist and remain billable.

---

# 2. Step-by-Step Solution

## Architecture Used in This Assignment

For this assignment, we will create an Amazon Aurora MySQL-Compatible Edition cluster.

The architecture will contain:

| Component | Quantity | Purpose |
| --------- | -------- | ------- |
| Aurora DB Cluster | `1` | Logical database cluster that manages the Aurora database instances and shared cluster storage. |
| Writer DB Instance | `1` | Handles write operations and can also process read operations. |
| Reader DB Instance 1 | `1` | Handles read-only traffic and improves read scalability and availability. |
| Reader DB Instance 2 | `1` | Provides an additional read replica in another Availability Zone. |
| DB Subnet Group | `1` | Defines the subnets and Availability Zones in which Aurora DB instances can be deployed. |
| Security Group | `1` | Controls inbound network access to the Aurora MySQL database. |

The target architecture is:

```text
AWS Region
|
+-- Default VPC
    |
    +-- DB Subnet Group
    |   |
    |   +-- Availability Zone 1
    |   |   |
    |   |   +-- Aurora Writer
    |   |
    |   +-- Availability Zone 2
    |   |   |
    |   |   +-- Aurora Reader 1
    |   |
    |   +-- Availability Zone 3
    |       |
    |       +-- Aurora Reader 2
    |
    +-- Security Group
        |
        +-- Controls MySQL/Aurora access on TCP port 3306
```

---

## Step 1: Select the AWS Region

### Navigation

```text
AWS Management Console
→ Top navigation bar
→ Region selector
→ Select the required AWS Region
```

For this assignment, use:

* **Recommended Region:** `Asia Pacific (Mumbai) ap-south-1`

### Configuration

Select the AWS Region before creating any database resources.

All resources created for this assignment should remain in the same AWS Region.

### Why is this step required?

AWS resources are regional. An Aurora cluster, DB subnet group, security group, and VPC configuration must be created or selected within the same AWS Region.

Selecting the Region first prevents accidental creation of dependent resources in different Regions.

### Dependency

This is the first step and has no dependency on any previously created resource.

### What happens if this step is skipped?

If resources are accidentally created in different AWS Regions:

* The security group may not be available when creating the Aurora cluster.
* The DB subnet group may not appear.
* Resources cannot be directly associated across Regions.
* Troubleshooting becomes unnecessarily difficult.
* Resources may be accidentally left running in another Region and continue consuming credits.

---

## Step 2: Verify the Default VPC and Availability Zones

Amazon Aurora requires network infrastructure in which the DB instances can be deployed.

For this assignment, we will use the existing default VPC to avoid creating unnecessary networking resources.

### Navigation

```text
AWS Management Console
→ VPC
→ Your VPCs
```

### Configuration

Locate the VPC where:

* **Default VPC:** `Yes`

Note the following information:

* **VPC ID:** Example: `vpc-0123456789abcdef0`
* **IPv4 CIDR:** Usually similar to `172.31.0.0/16`

Do not copy the example VPC ID literally. Use the actual VPC ID displayed in your AWS account.

Next, navigate to:

```text
AWS Management Console
→ VPC
→ Subnets
```

Verify that the default VPC contains subnets in multiple Availability Zones.

For example:

| Subnet | Availability Zone |
| ------ | ----------------- |
| Default Subnet 1 | `ap-south-1a` |
| Default Subnet 2 | `ap-south-1b` |
| Default Subnet 3 | `ap-south-1c` |

The exact Availability Zones displayed in your AWS account may differ.

### Why is this step required?

Aurora DB instances need subnets in which their network interfaces can be placed.

To deploy the writer and reader instances across different Availability Zones, the database environment must have suitable subnets spanning multiple Availability Zones.

The default VPC normally already contains subnets across multiple Availability Zones, making it suitable for this learning assignment.

### Dependency

Depends on:

* **Step 1:** Correct AWS Region selected.

### What happens if this step is skipped?

If you do not verify the VPC and subnet configuration:

* The required DB subnet group may not have sufficient Availability Zone coverage.
* Aurora database creation may fail.
* Reader instances may not be placeable in the intended Availability Zones.
* You may accidentally select resources from the wrong VPC.

---

## Step 3: Create a DB Subnet Group

A DB subnet group tells Amazon RDS which VPC subnets can be used for deploying database instances.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Subnet groups
→ Create DB subnet group
```

### Configuration

Enter the following values:

| Setting | Value |
| ------- | ----- |
| **Name** | `aurora-assignment-subnet-group` |
| **Description** | `DB subnet group for Aurora Assignment 2` |
| **VPC** | Select the default VPC verified in Step 2 |

Under **Add subnets**, select Availability Zones and subnets from the default VPC.

Preferably select subnets from at least `3` different Availability Zones, if available in your AWS account.

Example:

| Availability Zone | Subnet |
| ----------------- | ------ |
| `ap-south-1a` | Select one subnet belonging to this AZ |
| `ap-south-1b` | Select one subnet belonging to this AZ |
| `ap-south-1c` | Select one subnet belonging to this AZ |

Then click:

```text
Create
```

### Why is this step required?

The DB subnet group defines the network locations where Amazon RDS can place the Aurora DB instances.

Because this assignment requires:

* `1` Writer instance.
* `2` Reader instances.
* Reader instances in different Availability Zones.

The subnet group should provide subnet coverage across multiple Availability Zones.

### Dependency

Depends on:

* **Step 1:** AWS Region selected.
* **Step 2:** Default VPC and its subnets verified.

### What happens if this step is skipped?

If a suitable DB subnet group is not available:

* You may not be able to control the network placement of the Aurora cluster properly.
* The intended multi-AZ reader architecture may not be possible.
* Database creation may fail if sufficient subnet coverage is unavailable.

---

## Step 4: Create a Security Group for the Aurora Database

The Aurora MySQL-Compatible database uses TCP port `3306`.

A security group acts as a virtual firewall that controls inbound and outbound traffic for resources associated with it.

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
```

### Configuration

Configure the security group as follows:

| Setting | Value |
| ------- | ----- |
| **Security group name** | `Aurora-Assignment-SG` |
| **Description** | `Security group for Aurora Assignment 2` |
| **VPC** | Select the same default VPC used by the DB subnet group |

### Inbound Rules

For the initial Aurora cluster creation, do not expose the database publicly to the entire internet.

If no EC2 client instance or application server is part of the assignment, you can create the security group without adding an unnecessary public inbound rule.

The database security group can later allow MySQL traffic from a trusted application security group or a specific trusted source when actual database connectivity testing is required.

### Outbound Rules

Keep the default outbound rule:

| Type | Protocol | Port Range | Destination |
| ---- | -------- | ---------- | ----------- |
| All traffic | All | All | `0.0.0.0/0` |

Click:

```text
Create security group
```

### Why is this step required?

The security group provides network-level access control for the Aurora database.

It determines which clients, applications, or EC2 instances are permitted to initiate connections to the database.

Creating a dedicated security group is better than using the default security group because:

* Database access rules remain isolated.
* The purpose of each rule is easier to understand.
* Troubleshooting is simpler.
* It follows the principle of least privilege.

### Dependency

Depends on:

* **Step 2:** The default VPC must be identified.

The security group must belong to the same VPC used by the Aurora cluster and DB subnet group.

### What happens if this step is skipped?

If no suitable security group is selected:

* The database may use an unintended security group.
* Network access may be too restrictive or too permissive.
* Future applications may be unable to connect.
* Security troubleshooting becomes more difficult.

---

## Step 5: Open the Amazon RDS Database Creation Wizard

Now that the networking prerequisites are ready, begin creating the Aurora DB cluster.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Create database
```

### Configuration

Under **Choose a database creation method**, select:

* **Database creation method:** `Standard create`

Do not select `Easy create`.

### Why is this step required?

`Standard create` exposes the detailed database configuration options required for this assignment, including:

* Aurora engine selection.
* Engine version.
* DB instance configuration.
* Availability settings.
* Connectivity settings.
* DB subnet group.
* VPC security group.
* Authentication settings.
* Additional database configuration.

Using `Easy create` hides several important configuration options and provides less control over the resulting architecture.

### Dependency

Depends on:

* **Step 1:** Correct AWS Region.
* **Step 2:** Default VPC verified.
* **Step 3:** DB subnet group created.
* **Step 4:** Dedicated Aurora security group created.

### What happens if this step is skipped?

The Aurora cluster cannot be created without starting the database creation workflow.

If `Easy create` is selected instead of `Standard create`, you may not have sufficient control over all settings required for the assignment.

---

## Step 6: Select Amazon Aurora as the Database Engine

In the **Create database** wizard, configure the database engine.

### Configuration

Under **Engine options**, select:

| Setting | Value |
| ------- | ----- |
| **Engine type** | `Amazon Aurora` |
| **Edition** | `Amazon Aurora MySQL-Compatible Edition` |

For the engine version:

* Select a currently supported Aurora MySQL version available in the AWS Console.
* Prefer the default recommended stable version shown by AWS unless your course specifically requires another version.

### Why are we using Aurora MySQL-Compatible Edition?

Amazon Aurora is a fully managed relational database engine built for the cloud and compatible with MySQL or PostgreSQL.

For this assignment, Aurora MySQL-Compatible Edition is suitable because:

* It provides SQL database functionality.
* It supports Aurora Replicas.
* Read traffic can be distributed across reader instances.
* Replicas can be placed in different Availability Zones.
* Aurora automatically manages cluster storage replication across multiple Availability Zones.

### Important Aurora Architecture Concept

In a traditional database architecture, each replica may maintain its own separate complete copy of the database storage.

Aurora uses a different architecture:

```text
                  Aurora DB Cluster
                         |
             +-----------+-----------+
             |                       |
             v                       v
      Writer DB Instance      Reader DB Instances
             |                       |
             +-----------+-----------+
                         |
                         v
              Shared Aurora Cluster Storage
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
      AZ-1              AZ-2              AZ-3
```

The writer and Aurora Replicas connect to the same distributed cluster storage system.

This architecture helps provide:

* High availability.
* Fast replica creation.
* Read scalability.
* Automatic storage replication.

### Dependency

Depends on:

* **Step 5:** The Amazon RDS database creation wizard must be open.

### What happens if this step is skipped?

If another database engine is selected, such as standard MySQL or PostgreSQL, the resulting database will not be an Amazon Aurora cluster and the primary requirement of the assignment will not be satisfied.

---

## Step 7: Select the Database Template and Availability Configuration

Continue in the **Create database** wizard.

### Configuration

Under **Templates**, the available choices may include:

* `Production`
* `Dev/Test`
* `Free tier`, when available for the selected engine and account configuration.

For this assignment, select the lowest-cost configuration that still allows the required Aurora architecture.

> **Important:** AWS Console options can vary depending on the AWS account, Region, selected Aurora engine version, and current Free Tier eligibility. Do not assume that every account will display exactly the same template options.

For the required architecture, remember that the final environment must contain:

```text
1 Writer DB Instance
2 Reader DB Instances
```

If the initial creation wizard offers an option to create Aurora Replicas automatically, ensure that the resulting architecture and Availability Zone placement match the assignment requirements.

Otherwise, create the initial Aurora cluster first and add the two Aurora Replicas manually in later steps.

### Why is this step required?

The selected template influences default settings such as:

* Availability.
* Performance.
* Monitoring.
* Backup configuration.
* Cost-related defaults.

For a learning assignment, unnecessary production-grade optional features should be avoided unless they are required by the problem statement.

### Dependency

Depends on:

* **Step 6:** Amazon Aurora MySQL-Compatible Edition must be selected.

### What happens if this step is skipped?

Incorrect template selection can:

* Enable unnecessary features.
* Increase resource consumption.
* Increase AWS promotional credit usage.
* Create a configuration different from the assignment requirements.

---

## Step 8: Configure the Aurora DB Cluster Identifier and Credentials

Continue in the **Create database** wizard.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Create database
→ Settings
```

### Configuration

Under the **Settings** section, configure the following values:

| Setting | Value |
| ------- | ----- |
| **DB cluster identifier** | `aurora-assignment-cluster` |
| **Master username** | `admin` |
| **Credentials management** | `Self managed` |
| **Master password** | Create a strong password |
| **Confirm master password** | Enter the same password again |

Example password format:

```text
Aurora@Assignment2026!
```

> **Important:** The password shown above is only an example. For a real environment, create your own strong and unique password. Do not commit actual database passwords to GitHub.

### What is a DB Cluster Identifier?

The DB cluster identifier is the unique name used to identify the Aurora cluster within the selected AWS Region.

In this assignment:

```text
aurora-assignment-cluster
```

identifies the complete Aurora cluster containing:

```text
Aurora Cluster
|
+-- Writer DB Instance
|
+-- Reader DB Instance 1
|
+-- Reader DB Instance 2
```

### What is the Master Username?

The master username is the administrative database user created when the Aurora cluster is provisioned.

For this assignment:

```text
admin
```

will be used as the database administrator username.

This user can later be used to:

* Connect to the database.
* Create databases.
* Create tables.
* Insert data.
* Retrieve data.
* Manage database users and privileges, subject to Amazon RDS restrictions.

### Why is this step required?

The Aurora cluster needs:

* A unique cluster identifier.
* An administrative username.
* Authentication credentials.

Without these values, the database cannot be properly identified or administratively accessed.

### Dependency

Depends on:

* **Step 5:** RDS database creation wizard opened.
* **Step 6:** Amazon Aurora selected as the database engine.
* **Step 7:** Database template and availability configuration selected.

### What happens if this step is skipped?

The database creation wizard cannot complete without the required cluster identification and authentication configuration.

Incorrect credentials can also prevent future database connections.

---

## Step 9: Configure the Aurora DB Instance

The DB instance configuration determines the compute capacity allocated to the Aurora database instance.

### Navigation

Continue in the same **Create database** wizard and locate:

```text
Instance configuration
```

### Configuration

The exact options shown by AWS can vary based on:

* AWS Region.
* Aurora engine version.
* AWS account type.
* Free Tier eligibility.
* Selected database template.

Select the smallest suitable DB instance class available for your account and Aurora configuration.

For a provisioned Aurora configuration, an example may be:

| Setting | Value |
| ------- | ----- |
| **DB instance class** | Smallest suitable supported instance class displayed by AWS |

Possible instance classes may include options such as:

```text
db.t3.medium
db.t4g.medium
```

Do not select a larger instance class unless required by the assignment.

> **Cost Warning:** This assignment requires a total of `3` DB instances: `1` writer and `2` readers. The selected DB instance class significantly affects AWS credit consumption and potential charges.

### Important Cost Calculation Concept

The assignment requires:

```text
1 Writer
+
2 Readers
=
3 Running DB Instances
```

Therefore, if each DB instance has an hourly compute charge, the total compute consumption is approximately:

```text
Hourly cost of one DB instance × 3
```

Additional charges may also apply for:

* Aurora storage.
* I/O operations, depending on the storage configuration.
* Backup storage beyond applicable allowances.
* Other optional features.

### Why is this step required?

Every Aurora DB instance requires compute resources.

The instance class determines:

* CPU capacity.
* Memory capacity.
* Network performance.
* Database workload capability.
* Hourly compute cost.

### Dependency

Depends on:

* **Step 8:** Cluster identifier and credentials configured.

### What happens if this step is skipped?

The Aurora database cannot run without a valid compute configuration.

Selecting an unnecessarily large instance class can consume promotional credits significantly faster.

---

## Step 10: Configure Connectivity

The connectivity configuration determines:

* Which VPC contains the Aurora cluster.
* Which subnets can host the DB instances.
* Whether the database receives public network accessibility.
* Which security group controls database traffic.

### Navigation

Continue in the same **Create database** wizard and locate:

```text
Connectivity
```

### Configuration

Configure the following settings:

| Setting | Value |
| ------- | ----- |
| **Compute resource** | `Don't connect to an EC2 compute resource` |
| **Network type** | `IPv4` |
| **Virtual private cloud (VPC)** | Select the default VPC verified in Step 2 |
| **DB subnet group** | `aurora-assignment-subnet-group` |
| **Public access** | `No` |
| **VPC security group** | Choose existing |
| **Existing VPC security group** | `Aurora-Assignment-SG` |
| **Availability Zone** | `No preference` for the initial writer unless a specific AZ is intentionally selected |
| **Database port** | `3306` |

### Why Select "Don't Connect to an EC2 Compute Resource"?

This assignment specifically requires:

* One Aurora database cluster.
* Two Aurora Read Replicas.

It does not require an EC2 instance.

Therefore, automatically connecting the database to an EC2 instance is unnecessary.

### Why Use IPv4?

IPv4 provides standard network connectivity for this learning assignment and is widely supported across AWS networking environments.

### Why Use the Default VPC?

The default VPC already provides:

* A VPC network.
* Multiple subnets.
* Multiple Availability Zones.
* Route tables.
* Network infrastructure required by the assignment.

Using it avoids creating unnecessary networking resources when the assignment only focuses on Aurora.

### Why Set Public Access to No?

Setting:

```text
Public access: No
```

means the database does not receive a publicly reachable endpoint from the internet.

This follows AWS security best practices because databases should generally remain private unless public access is specifically required.

The assignment does not require public database access.

### Why Use Port 3306?

Aurora MySQL-Compatible Edition uses the standard MySQL port:

```text
TCP 3306
```

Applications and database clients use this port when connecting to the Aurora MySQL-compatible database.

### Why is this step required?

The Aurora cluster requires network configuration so AWS knows:

* Where to deploy the database.
* Which VPC to use.
* Which subnets are available.
* Which security group controls access.
* Which network port the database uses.

### Dependency

Depends on:

* **Step 2:** Default VPC verified.
* **Step 3:** DB subnet group created.
* **Step 4:** Aurora security group created.
* **Step 9:** DB instance configuration selected.

### What happens if this step is skipped?

Incorrect connectivity configuration can cause:

* Database creation failure.
* Wrong VPC placement.
* Incorrect subnet placement.
* Security group mismatch.
* Inability for applications to connect.
* Unnecessary public exposure.

---

## Step 11: Configure Database Authentication

Continue in the **Create database** wizard and locate the database authentication options.

### Configuration

For this assignment, use:

| Setting | Value |
| ------- | ----- |
| **Database authentication** | `Password authentication` |

Depending on the current AWS Console and selected Aurora version, additional authentication options may be available.

For this basic assignment, password authentication is sufficient.

### Why is this step required?

Database authentication determines how clients prove their identity before connecting to the database.

With password authentication, the client provides:

```text
Username
+
Password
```

For example:

```text
Username: admin
Password: Your configured master password
```

### Dependency

Depends on:

* **Step 8:** Master username and password configured.

### What happens if this step is skipped?

A valid authentication method is required to securely connect to the database.

Without proper credentials, users and applications cannot establish authenticated database sessions.

---

## Step 12: Configure Monitoring Options

Monitoring features can provide additional database performance metrics but may also introduce unnecessary complexity or additional cost for a basic assignment.

### Navigation

Continue in the same **Create database** wizard and locate the monitoring configuration.

### Configuration

For this assignment, avoid enabling unnecessary paid or advanced monitoring features unless required by your course.

Use the lowest-cost configuration available.

Possible settings may include:

| Setting | Recommended Assignment Configuration |
| ------- | ------------------------------------ |
| **Database Insights** | Standard or lowest-cost available option |
| **Performance Insights** | Keep disabled if optional and not required |
| **Enhanced Monitoring** | Disabled unless required |
| **Log exports** | Not required for this assignment |

> **Important:** The exact names and available options can change depending on the AWS Console, Aurora engine version, and account configuration.

### Why is this step required?

Monitoring settings should be reviewed before creating the database because optional features can:

* Increase observability.
* Generate additional metrics.
* Increase operational complexity.
* Potentially increase AWS usage or cost.

The assignment only requires creation of an Aurora cluster and two read replicas, so advanced monitoring is not necessary.

### Dependency

Depends on:

* The previous Aurora configuration steps.

### What happens if this step is skipped?

AWS may use default monitoring settings.

This may enable features you did not intentionally choose or make it harder to understand which services are active.

---

## Step 13: Configure Additional Database Settings

Continue in the **Create database** wizard and review the additional configuration options.

### Configuration

Use the following configuration where available:

| Setting | Value |
| ------- | ----- |
| **Initial database name** | `assignmentdb` |
| **DB cluster parameter group** | Default |
| **DB parameter group** | Default |
| **Backup retention period** | Keep the minimum suitable value available for the assignment |
| **Encryption** | Keep the AWS default configuration |
| **Deletion protection** | `Disabled` |
| **Auto minor version upgrade** | Keep the default setting |
| **Maintenance window** | `No preference` |

### Initial Database Name

Use:

```text
assignmentdb
```

This creates an initial logical database inside the Aurora MySQL-compatible cluster.

### Why Disable Deletion Protection?

The assignment explicitly requires cleanup after verification.

If deletion protection remains enabled:

```text
Delete cluster
        |
        v
Deletion blocked
```

You would first have to modify the cluster, disable deletion protection, wait for the modification, and then attempt deletion again.

For a temporary learning environment that will be deleted immediately after verification, disabling deletion protection simplifies cleanup.

> **Production Note:** In production environments, deletion protection is generally recommended for critical databases to prevent accidental deletion.

### Why is this step required?

These additional settings control:

* Initial database creation.
* Backup behavior.
* Encryption.
* Maintenance.
* Parameter configuration.
* Protection against accidental deletion.

Reviewing these settings helps prevent unnecessary features or cleanup difficulties.

### Dependency

Depends on:

* All previous Aurora creation configuration steps.

### What happens if this step is skipped?

AWS may apply default settings that:

* Are unnecessary for the assignment.
* Make deletion more difficult.
* Increase backup retention.
* Differ from the intended learning configuration.

---

## Step 14: Review the Estimated Monthly Cost

Before creating the Aurora database, review the estimated cost information displayed by AWS.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Create database
→ Review the Estimated monthly costs section
```

### Action

Review all cost estimates shown by AWS before clicking **Create database**.

Pay particular attention to:

* DB instance compute charges.
* Number of DB instances.
* Aurora storage.
* I/O configuration.
* Backup storage.
* Monitoring features.

### Important Cost Consideration

At the initial creation stage, you may have:

```text
1 Writer DB Instance
```

After completing the assignment, you will have:

```text
1 Writer DB Instance
+
2 Reader DB Instances
=
3 Total DB Instances
```

Therefore, the final cost or promotional credit consumption will be higher than the initial estimate if the estimate only reflects the first writer instance.

### Why is this step required?

AWS pricing varies by:

* Region.
* Instance class.
* Aurora version.
* Storage configuration.
* Account eligibility.
* Free Tier program.
* Optional features.

The AWS Console estimate is therefore more relevant to your actual configuration than relying only on a static example.

### Dependency

Depends on:

* All previous database configuration steps.

### What happens if this step is skipped?

You may accidentally create:

* An expensive DB instance class.
* Unnecessary monitoring features.
* A configuration that consumes promotional credits faster than expected.

---

## Step 15: Create the Initial Aurora Cluster and Writer Instance

After reviewing all settings and cost information, create the database.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Create database
→ Review configuration
→ Create database
```

### Action

Click:

```text
Create database
```

AWS will begin provisioning:

```text
Aurora DB Cluster
        |
        v
Initial Writer DB Instance
```

The status may initially display:

```text
Creating
```

Wait until the writer DB instance status becomes:

```text
Available
```

### Expected Initial Architecture

After successful creation, the database list should contain an architecture similar to:

```text
aurora-assignment-cluster
|
+-- aurora-assignment-instance
    |
    +-- Role: Writer
    +-- Status: Available
```

The exact automatically generated instance identifier may differ depending on your configuration.

### Why is this step required?

The writer instance is the primary DB instance in the Aurora cluster.

It handles:

* `INSERT` operations.
* `UPDATE` operations.
* `DELETE` operations.
* `CREATE` operations.
* Other database write transactions.
* Read operations when directly connected to the writer.

The two required read replicas will be added after the initial cluster becomes available.

### Dependency

Depends on:

* Steps 1 through 14 being correctly completed.

### What happens if this step is skipped?

There will be no Aurora cluster to which the two read replicas can be added.

The assignment cannot proceed without the initial Aurora cluster and writer DB instance.

---

## Step 16: Verify the Initial Writer Instance Is Available

Before creating the read replicas, verify that the initial Aurora cluster and writer instance are fully available.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
```

### Configuration to Verify

Confirm the following:

| Property | Expected Value |
| -------- | -------------- |
| **Cluster identifier** | `aurora-assignment-cluster` |
| **Engine** | Aurora MySQL |
| **Writer instances** | `1` |
| **Writer status** | `Available` |
| **Reader instances** | `0` at this stage |

### Why is this step required?

A read replica should be created only after the initial Aurora cluster is successfully provisioned and available.

This confirms that the parent cluster exists and is ready to accept additional Aurora Replica instances.

### Dependency

Depends on:

* **Step 15:** Initial Aurora cluster and writer instance created.

### What happens if this step is skipped?

Attempting to continue before the writer is fully available may:

* Prevent replica creation.
* Cause configuration options to remain unavailable.
* Create confusion about whether the initial database deployment succeeded.

---

## Step 17: Create the First Aurora Read Replica

Now create the first Aurora Replica.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-assignment-cluster
→ Actions
→ Add reader
```

Depending on the current AWS Console interface, the option may appear as:

```text
Add reader
```

or as an equivalent option for adding an Aurora Replica.

### Configuration

Configure the first reader as follows:

| Setting | Value |
| ------- | ----- |
| **DB instance identifier** | `aurora-reader-1` |
| **DB instance class** | Use the smallest suitable supported class consistent with the cluster configuration |
| **Availability Zone** | Select an AZ different from the writer's AZ where possible |
| **Public access** | `No` |
| **VPC security group** | `Aurora-Assignment-SG` |
| **Monitoring** | Avoid unnecessary optional paid features |

For example, if the writer is located in:

```text
ap-south-1a
```

place Reader 1 in:

```text
ap-south-1b
```

The actual Availability Zones available to your AWS account may differ.

### Availability Zone Example

```text
Writer
AZ: ap-south-1a

Reader 1
AZ: ap-south-1b
```

### Action

Review the settings and click the appropriate button to create the reader.

The initial status may display:

```text
Creating
```

Wait until the status becomes:

```text
Available
```

### Why is this step required?

The first Aurora Replica provides:

* Read scalability.
* Additional database availability.
* A potential failover target.
* Reduced read workload on the writer.

Applications can use the Aurora reader endpoint to distribute eligible read-only queries across available Aurora Replicas.

### Dependency

Depends on:

* **Step 15:** Aurora cluster created.
* **Step 16:** Writer instance verified as available.

### What happens if this step is skipped?

The assignment requirement to create two read replicas will not be satisfied.

The Aurora cluster will contain only the writer instance and will have less read scalability and fewer failover targets.

---

## Step 18: Create the Second Aurora Read Replica

Create the second Aurora Replica and place it in a different Availability Zone from Reader 1.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-assignment-cluster
→ Actions
→ Add reader
```

### Configuration

Configure the second reader as follows:

| Setting | Value |
| ------- | ----- |
| **DB instance identifier** | `aurora-reader-2` |
| **DB instance class** | Use the smallest suitable supported class consistent with the cluster configuration |
| **Availability Zone** | Select a different AZ from Reader 1 |
| **Public access** | `No` |
| **VPC security group** | `Aurora-Assignment-SG` |
| **Monitoring** | Avoid unnecessary optional paid features |

For example:

```text
Writer:
ap-south-1a

Reader 1:
ap-south-1b

Reader 2:
ap-south-1c
```

The assignment explicitly requires the two read replicas to be in different Availability Zones.

Therefore, the essential condition is:

```text
Reader 1 AZ != Reader 2 AZ
```

For maximum infrastructure distribution, use three different Availability Zones for the writer and two readers when your selected Region and subnet group support it.

### Action

Review the settings and create the second reader.

Wait until its status changes from:

```text
Creating
```

to:

```text
Available
```

### Why is this step required?

The second reader provides:

* Additional read capacity.
* Improved infrastructure availability.
* Another possible failover target.
* Distribution of DB instances across multiple Availability Zones.

### Dependency

Depends on:

* **Step 17:** First Aurora Replica created.

Although AWS can provision multiple readers independently, creating them sequentially makes the assignment easier to verify and troubleshoot.

### What happens if this step is skipped?

The assignment will remain incomplete because it explicitly requires:

```text
2 Read Replicas
```

Creating only one reader does not satisfy the requirement.

---

## Step 19: Verify the Final Aurora Cluster Architecture

After both readers become available, verify the complete cluster architecture.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
```

### Expected Architecture

The cluster should contain:

```text
aurora-assignment-cluster
|
+-- Writer
|   |
|   +-- Status: Available
|
+-- aurora-reader-1
|   |
|   +-- Role: Reader
|   +-- Status: Available
|
+-- aurora-reader-2
    |
    +-- Role: Reader
    +-- Status: Available
```

### Expected Instance Count

| Instance Role | Expected Count |
| ------------- | -------------- |
| Writer | `1` |
| Reader | `2` |
| Total DB Instances | `3` |

### Expected Availability Zone Distribution

Example:

| DB Instance | Role | Availability Zone |
| ----------- | ---- | ----------------- |
| Writer instance | Writer | `ap-south-1a` |
| `aurora-reader-1` | Reader | `ap-south-1b` |
| `aurora-reader-2` | Reader | `ap-south-1c` |

At minimum, verify:

```text
aurora-reader-1 AZ != aurora-reader-2 AZ
```

### Why is this step required?

This directly confirms whether the main assignment requirements have been implemented:

```text
Requirement 1:
Create an Aurora DB Engine-based RDS database.

Requirement 2:
Create 2 Read Replicas in different Availability Zones.
```

### Dependency

Depends on:

* **Step 15:** Writer created.
* **Step 17:** Reader 1 created.
* **Step 18:** Reader 2 created.

### What happens if this step is skipped?

You may not notice:

* A failed reader creation.
* A reader still in the `Creating` state.
* Both readers being placed incorrectly.
* An incomplete assignment architecture.

---

# 3. Verification / Output Checking

After completing the Aurora cluster implementation, verify that:

* The Aurora DB cluster exists.
* The database engine is Aurora MySQL-Compatible Edition.
* The cluster contains exactly `1` writer instance.
* The cluster contains exactly `2` reader instances.
* Both readers are in different Availability Zones.
* All three DB instances have the `Available` status.
* The cluster provides both writer and reader endpoints.

---

## Verification Step 1: Verify the Aurora DB Cluster

### Action

Navigate to the Amazon RDS database dashboard.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
```

Locate:

```text
aurora-assignment-cluster
```

Expand the cluster by selecting the arrow beside the cluster name.

### Expected Output

The database hierarchy should look similar to:

```text
aurora-assignment-cluster
|
+-- Writer DB Instance
|
+-- aurora-reader-1
|
+-- aurora-reader-2
```

The exact identifier of the writer DB instance may differ depending on the identifier configured or automatically generated during database creation.

### Why does this confirm success?

The presence of the Aurora cluster confirms that the first assignment requirement has been successfully implemented:

```text
Create an AuroraDB Engine based RDS Database.
```

The two reader instances under the same cluster confirm that Aurora Replicas have been added to the database architecture.

---

## Verification Step 2: Verify the Database Engine

### Action

Select:

```text
aurora-assignment-cluster
```

Open the **Configuration** tab.

Verify the database engine information.

### Expected Output

The engine should display a value similar to:

```text
Aurora MySQL
```

or:

```text
Aurora MySQL-Compatible Edition
```

The exact wording may vary depending on the current AWS Console interface.

### Why does this confirm success?

This confirms that the database was created using the Amazon Aurora engine rather than a standard Amazon RDS MySQL or PostgreSQL engine.

This directly satisfies the requirement:

```text
Create an AuroraDB Engine based RDS Database.
```

---

## Verification Step 3: Verify the Writer Instance

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
```

Identify the DB instance with the role:

```text
Writer
```

### Expected Output

The cluster should contain exactly one writer instance.

Example:

| Property | Expected Value |
| -------- | -------------- |
| **Role** | `Writer` |
| **Status** | `Available` |
| **Engine** | Aurora MySQL |

Expected architecture:

```text
Aurora Cluster
|
+-- Writer: 1
```

### Why does this confirm success?

Every Aurora cluster requires a writer DB instance to handle database write operations.

The writer handles operations such as:

```sql
INSERT
UPDATE
DELETE
CREATE
ALTER
DROP
```

It can also process read operations.

A writer instance with the `Available` status confirms that the primary database instance has been successfully provisioned and is operational.

---

## Verification Step 4: Verify the First Read Replica

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
→ Select aurora-reader-1
```

Open the **Configuration** tab.

### Expected Output

Verify the following:

| Property | Expected Value |
| -------- | -------------- |
| **DB instance identifier** | `aurora-reader-1` |
| **Role** | `Reader` |
| **Status** | `Available` |
| **Engine** | Aurora MySQL |
| **Availability Zone** | Example: `ap-south-1b` |

Your actual Availability Zone may differ.

Record the actual Availability Zone because it will be compared with Reader 2.

Example:

```text
Reader 1 Availability Zone:
ap-south-1b
```

### Why does this confirm success?

The `Reader` role confirms that the DB instance is an Aurora Replica rather than the writer.

The `Available` status confirms that the reader has been successfully provisioned and is ready to process eligible read-only workloads.

---

## Verification Step 5: Verify the Second Read Replica

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
→ Select aurora-reader-2
```

Open the **Configuration** tab.

### Expected Output

Verify the following:

| Property | Expected Value |
| -------- | -------------- |
| **DB instance identifier** | `aurora-reader-2` |
| **Role** | `Reader` |
| **Status** | `Available` |
| **Engine** | Aurora MySQL |
| **Availability Zone** | Example: `ap-south-1c` |

Your actual Availability Zone may differ.

Example:

```text
Reader 2 Availability Zone:
ap-south-1c
```

### Why does this confirm success?

This confirms that the second Aurora Replica was successfully created.

The assignment explicitly requires:

```text
2 Read Replicas
```

Therefore, both Reader 1 and Reader 2 must exist and have the `Available` status.

---

## Verification Step 6: Verify That the Two Readers Are in Different Availability Zones

### Action

Compare the Availability Zone of:

```text
aurora-reader-1
```

with:

```text
aurora-reader-2
```

### Expected Output

Example:

| DB Instance | Role | Availability Zone |
| ----------- | ---- | ----------------- |
| Writer instance | Writer | `ap-south-1a` |
| `aurora-reader-1` | Reader | `ap-south-1b` |
| `aurora-reader-2` | Reader | `ap-south-1c` |

The required condition is:

```text
Reader 1 Availability Zone != Reader 2 Availability Zone
```

Example:

```text
ap-south-1b != ap-south-1c
```

### Why does this confirm success?

The second assignment requirement explicitly states:

```text
Create 2 Read Replicas in different availability zones for better infrastructure availability.
```

If the readers are in different Availability Zones, the architecture is less dependent on a single physical Availability Zone.

For example:

```text
AWS Region: ap-south-1
|
+-- Availability Zone: ap-south-1a
|   |
|   +-- Aurora Writer
|
+-- Availability Zone: ap-south-1b
|   |
|   +-- Aurora Reader 1
|
+-- Availability Zone: ap-south-1c
    |
    +-- Aurora Reader 2
```

If one Availability Zone experiences an infrastructure failure, database instances in other Availability Zones may remain available, depending on the nature and scope of the failure.

---

## Verification Step 7: Verify the Total Number of DB Instances

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Expand aurora-assignment-cluster
```

Count the DB instances belonging to the cluster.

### Expected Output

The cluster should contain:

| Role | Count |
| ---- | ----- |
| Writer | `1` |
| Reader | `2` |
| **Total** | **`3`** |

Expected architecture:

```text
Aurora DB Cluster
|
+-- Writer Instance       = 1
|
+-- Reader Instance 1     = 1
|
+-- Reader Instance 2     = 1
|
+-- Total DB Instances    = 3
```

### Why does this confirm success?

This verifies the complete assignment architecture.

The required configuration is:

```text
1 Writer
+
2 Readers
=
3 Total DB Instances
```

If fewer than three DB instances exist, the assignment is incomplete.

---

## Verification Step 8: Verify the Aurora Cluster Endpoints

Aurora provides endpoints that applications can use to connect to different database roles.

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-assignment-cluster
→ Connectivity & security
→ Endpoints
```

### Expected Output

You should see endpoints including:

| Endpoint Type | Purpose |
| ------------- | ------- |
| Writer endpoint | Connects to the current writer DB instance |
| Reader endpoint | Distributes eligible read-only connections among available Aurora Replicas |

The endpoint values will look similar to:

```text
Writer Endpoint:
aurora-assignment-cluster.cluster-xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com
```

```text
Reader Endpoint:
aurora-assignment-cluster.cluster-ro-xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com
```

The actual generated endpoint values will be unique to your AWS account and Aurora cluster.

### Why does this confirm success?

The writer endpoint confirms that applications have a stable endpoint for database write operations.

The reader endpoint confirms that the cluster has an endpoint designed for read-only workloads.

With multiple Aurora Replicas, eligible reader connections can be distributed across the available readers.

Conceptually:

```text
Application Write Query
        |
        v
Writer Endpoint
        |
        v
Aurora Writer
```

For reads:

```text
Application Read Query
        |
        v
Reader Endpoint
        |
        +-------------------+
        |                   |
        v                   v
Aurora Reader 1      Aurora Reader 2
```

---

## Verification Step 9: Verify the DB Subnet Group

### Action

Navigate to:

```text
AWS Management Console
→ Amazon RDS
→ Subnet groups
→ Select aurora-assignment-subnet-group
```

### Expected Output

Verify that the DB subnet group contains subnets from multiple Availability Zones.

Example:

| Availability Zone | Subnet |
| ----------------- | ------ |
| `ap-south-1a` | Default VPC subnet |
| `ap-south-1b` | Default VPC subnet |
| `ap-south-1c` | Default VPC subnet |

### Why does this confirm success?

The DB subnet group defines the subnets in which Amazon RDS can place DB instances.

Multiple Availability Zones in the subnet group make it possible to distribute the writer and readers across separate Availability Zones.

---

## Verification Step 10: Verify the Aurora Security Group

### Action

Navigate to:

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Select Aurora-Assignment-SG
```

Verify that the security group belongs to the same VPC as the Aurora cluster.

### Expected Output

The security group should show:

| Property | Expected Value |
| -------- | -------------- |
| **Security group name** | `Aurora-Assignment-SG` |
| **VPC** | Same VPC used by the Aurora cluster |
| **Inbound access** | No unnecessary public database access |
| **Outbound access** | Default outbound configuration unless intentionally modified |

### Why does this confirm success?

This confirms that the Aurora database uses a dedicated network security boundary and has not been unnecessarily exposed to unrestricted public database traffic.

---

# 4. Cleanup / Cost Optimization

After successfully verifying the assignment, delete all resources created specifically for this assignment.

The correct cleanup order is:

```text
1. Delete Aurora Reader 2
        |
        v
2. Delete Aurora Reader 1
        |
        v
3. Delete Aurora Writer / Aurora Cluster
        |
        v
4. Delete DB Subnet Group
        |
        v
5. Delete Security Group
```

> **Important:** Depending on the current Amazon RDS Console workflow, deleting the Aurora cluster may also provide options to delete the DB instances belonging to the cluster. Carefully review every deletion confirmation screen before proceeding.

The deletion order is important because AWS resources can have dependencies.

For example:

```text
Security Group
      ^
      |
Aurora DB Instance
```

If the security group is still attached to a running DB instance, AWS may prevent its deletion.

---

## Cleanup Step 1: Delete the Second Aurora Read Replica

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-reader-2
→ Actions
→ Delete
```

### Action

Review the deletion page carefully.

If AWS asks whether to create a final snapshot for the reader instance, follow the options presented by the current console. For this temporary assignment environment, do not retain unnecessary resources unless required.

Confirm the deletion.

Wait until:

```text
aurora-reader-2
```

is completely removed from the database list.

### Why is this required?

The second reader is a running Aurora DB instance and may continue consuming promotional credits or generating charges while it exists and remains billable.

### What happens if this resource is not deleted?

The reader instance may continue:

* Consuming database compute resources.
* Using promotional credits.
* Generating AWS charges.

---

## Cleanup Step 2: Delete the First Aurora Read Replica

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-reader-1
→ Actions
→ Delete
```

### Action

Review the deletion settings.

Confirm deletion and wait until:

```text
aurora-reader-1
```

is completely removed.

### Why is this required?

The first reader is also a database compute instance.

Deleting it prevents unnecessary continued resource consumption after the assignment has been verified.

### What happens if this resource is not deleted?

It may continue consuming promotional credits or generating charges even though the assignment has already been completed.

---

## Cleanup Step 3: Delete the Aurora Cluster and Writer Instance

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
→ Select aurora-assignment-cluster
→ Actions
→ Delete
```

### Action

Review all deletion options carefully.

For a temporary assignment environment that does not contain important data:

* Do not retain unnecessary final snapshots.
* Do not retain automated backups unless specifically required.
* Confirm that deletion protection is disabled.
* Follow the confirmation instructions displayed by AWS.

The console may ask you to enter a confirmation phrase or acknowledge that the database will be permanently deleted.

Confirm the deletion.

Wait until both the cluster and its remaining writer DB instance disappear from:

```text
Amazon RDS
→ Databases
```

### Why is this required?

The writer DB instance and Aurora cluster are the primary database resources created for the assignment.

Deleting them prevents continued compute and storage consumption.

### What happens if this resource is not deleted?

The Aurora environment may continue:

* Consuming DB instance compute capacity.
* Using Aurora storage.
* Retaining backups.
* Consuming promotional credits.
* Generating charges.

---

## Cleanup Step 4: Verify That No Assignment Database Resources Remain

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
```

### Action

Verify that none of the following remain:

```text
aurora-assignment-cluster
aurora-reader-1
aurora-reader-2
```

Also verify that the writer DB instance no longer exists.

### Why is this required?

Deletion operations can take several minutes.

A resource showing:

```text
Deleting
```

has not yet been fully removed.

You should wait until the resource completely disappears from the database list.

### What happens if this resource is not deleted?

Any remaining billable database resource may continue consuming promotional credits or generating charges.

---

## Cleanup Step 5: Delete the DB Subnet Group

Delete the DB subnet group only after the Aurora cluster and all DB instances using it have been completely deleted.

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Subnet groups
→ Select aurora-assignment-subnet-group
→ Delete
```

### Action

Delete:

```text
aurora-assignment-subnet-group
```

Confirm the deletion.

### Why is this required?

The DB subnet group was created specifically for this assignment.

Although a DB subnet group itself generally does not have a direct hourly charge, removing unused assignment-specific resources keeps the AWS account clean and avoids future confusion.

### What happens if this resource is not deleted?

The unused subnet group may remain in the AWS account.

Although it generally does not directly generate hourly charges, it creates unnecessary configuration clutter.

---

## Cleanup Step 6: Delete the Aurora Security Group

Delete the dedicated Aurora security group after all Aurora DB instances using it have been removed.

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Select Aurora-Assignment-SG
→ Actions
→ Delete security groups
```

### Action

Delete:

```text
Aurora-Assignment-SG
```

Confirm the deletion.

### Why is this required?

The security group was created specifically for this assignment and is no longer needed after the Aurora cluster has been deleted.

### What happens if this resource is not deleted?

Security groups generally do not have a direct hourly charge, but unused security groups:

* Create unnecessary clutter.
* Make future troubleshooting more difficult.
* Can cause confusion about which security rules are actively required.

If the security group is still attached to another resource, AWS may prevent deletion.

---

## Cleanup Step 7: Final Cost Verification

### Navigation

```text
AWS Management Console
→ Amazon RDS
→ Databases
```

### Action

Confirm that the following resources no longer exist:

| Resource | Expected Final State |
| -------- | -------------------- |
| `aurora-assignment-cluster` | Deleted |
| Writer DB instance | Deleted |
| `aurora-reader-1` | Deleted |
| `aurora-reader-2` | Deleted |
| `aurora-assignment-subnet-group` | Deleted |
| `Aurora-Assignment-SG` | Deleted |

Also check:

```text
AWS Management Console
→ Billing and Cost Management
→ Bills
```

and, if applicable:

```text
AWS Management Console
→ Billing and Cost Management
→ Credits
```

### Why is this required?

The Aurora DB instances are the primary resources in this assignment that may consume promotional credits or generate charges.

Verifying deletion ensures that no assignment-specific database compute resources were accidentally left running.

### What happens if this resource is not deleted?

Leaving Aurora DB instances running unnecessarily can continue consuming promotional credits or generating charges after the practical assignment has been completed.

---

# End of Assignment



