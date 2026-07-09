# Module 7 – Database Services

# Assignment 1 – MariaDB

## Problem Statement

You work for XYZ Corporation. Their application requires a SQL service that can store data which can be retrieved when required. Implement a suitable Amazon RDS database engine for the same.

While migrating, you are asked to perform the following tasks:

1. Create a MariaDB engine-based Amazon RDS database.
2. Connect to the database using the following methods:
   - SQL Client for Windows.
   - Linux-based EC2 instance.

---

# 1. Free Tier / Cost Check

Before creating the resources, it is important to understand which AWS services are used in this assignment and whether they are covered by the AWS Free Tier or promotional credits.

> **Important:** AWS Free Tier eligibility depends on the AWS account type, account creation date, selected instance classes, storage configuration, Region, and remaining promotional credits. Always check the estimated monthly cost displayed by the AWS Console before creating a resource.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|-------|
| Amazon RDS for MariaDB | Depends on account and selected configuration | Yes, if not covered by available Free Tier benefits | The DB instance, allocated storage, backup storage beyond applicable allowances, and additional features can incur charges. |
| Amazon EC2 | Depends on account and selected instance type | Yes, if not covered by available Free Tier benefits | Used as the Linux client machine for connecting to the MariaDB database. |
| Amazon VPC | Generally no additional charge for the VPC itself | Usually No | The default VPC, subnets, route tables, and security groups generally do not have separate hourly charges. |
| Public IPv4 Address | Generally chargeable | Yes | A public IPv4 address attached to the EC2 instance can incur hourly charges. |
| Data Transfer | Depends on traffic direction and volume | Possibly | Small amounts of traffic during this lab are usually minimal, but AWS data-transfer pricing rules still apply. |
| Windows SQL Client | Depends on selected software | No AWS charge for locally installed free client software | A locally installed SQL client such as MySQL Workbench can be used to connect to MariaDB. |

## Free Tier Conclusion

This assignment should **not automatically be assumed to be completely free**.

The primary resources that may consume AWS promotional credits are:

- The Amazon RDS MariaDB DB instance.
- The Amazon EC2 Linux instance.
- The public IPv4 address associated with the EC2 instance.
- Any additional storage, backup, or data-transfer usage beyond applicable allowances.

If you currently have AWS promotional credits, it is beneficial to complete this assignment while those credits are still valid, particularly because an RDS database instance is a continuously running resource until it is stopped or deleted.

To minimize costs:

- Use the smallest eligible DB instance class offered by your AWS account and Region for this lab.
- Use the smallest suitable EC2 instance type available under your account's Free Tier or credit plan.
- Use minimal storage suitable for the assignment.
- Do not enable optional paid features unless required.
- Delete all resources immediately after completing verification.

---

# 2. Architecture and Dependency Order

The assignment uses the following architecture:

```text
                    ┌──────────────────────────┐
                    │      Windows Computer    │
                    │                          │
                    │   MySQL Workbench /      │
                    │   Compatible SQL Client  │
                    └─────────────┬────────────┘
                                  │
                                  │ TCP Port 3306
                                  ▼
                    ┌──────────────────────────┐
                    │                          │
                    │    Amazon RDS MariaDB    │
                    │                          │
                    │    Database Port: 3306   │
                    │                          │
                    └─────────────▲────────────┘
                                  │
                                  │ TCP Port 3306
                                  │
                    ┌─────────────┴────────────┐
                    │                          │
                    │    Linux EC2 Instance    │
                    │                          │
                    │    MariaDB Client        │
                    │                          │
                    └──────────────────────────┘
```

## Resource Dependency Flow

```text
Default VPC and Subnets
        │
        ▼
Create RDS Security Group
        │
        ▼
Create EC2 Security Group
        │
        ▼
Create DB Subnet Group
        │
        ▼
Create MariaDB RDS Database
        │
        ▼
Create Linux EC2 Instance
        │
        ├─────────────────────────────┐
        ▼                             ▼
Connect from Windows           Connect from Linux EC2
SQL Client                     using MariaDB Client
        │                             │
        └──────────────┬──────────────┘
                       ▼
              Verify Database Access
                       │
                       ▼
                 Delete Resources
```

---

# Step 1: Verify the AWS Region

The RDS database, EC2 instance, VPC, subnets, and security groups should be created in the same AWS Region.

### Navigation

```text
AWS Management Console
→ Top-right Region selector
→ Select your preferred AWS Region
```

### Configuration

For this assignment, use:

- **Region:** `Asia Pacific (Mumbai) - ap-south-1`

### Why is this step required?

AWS resources are Region-specific. Keeping all resources in the same Region simplifies networking and resource management.

### Dependency

This is the first step and has no dependency.

### What happens if this step is skipped?

Resources may accidentally be created in different Regions, causing networking, visibility, and troubleshooting problems.

---

# Step 2: Verify the Default VPC and Subnets

For this assignment, use the existing default VPC.

### Navigation

```text
AWS Management Console
→ VPC
→ Your VPCs
```

### Configuration

Identify the VPC where:

- **Default VPC:** `Yes`

Record:

- **VPC ID:** `vpc-xxxxxxxxxxxxxxxxx`

Next:

```text
AWS Management Console
→ VPC
→ Subnets
```

Verify that multiple subnets exist in different Availability Zones.

Example:

| Subnet | Availability Zone |
|--------|-------------------|
| `subnet-xxxxxxxxxxxxxxxx1` | `ap-south-1a` |
| `subnet-xxxxxxxxxxxxxxxx2` | `ap-south-1b` |
| `subnet-xxxxxxxxxxxxxxxx3` | `ap-south-1c` |

### Why is this step required?

The VPC provides network isolation, while subnets determine where AWS resources can be placed.

### Dependency

Depends on **Step 1**.

### What happens if this step is skipped?

The EC2 instance and RDS database could accidentally be placed in different VPCs, preventing straightforward private communication.

---

# Step 3: Create the RDS Security Group

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
```

### Configuration

- **Security group name:** `MariaDB-RDS-SG`
- **Description:** `Allows MariaDB connections from Windows client and Linux EC2`
- **VPC:** Select the default VPC.

### Inbound Rule for Windows

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| MySQL/Aurora | TCP | `3306` | My IP |

Keep the default outbound rule.

Click **Create security group**.

### Why is this step required?

MariaDB listens on TCP port `3306`. The security group controls which clients can reach that port.

### Dependency

Depends on the default VPC from **Step 2**.

### What happens if this step is skipped?

Clients will not be able to connect to the MariaDB database.

---

# Step 4: Create the EC2 Security Group

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Create security group
```

### Configuration

- **Security group name:** `MariaDB-EC2-SG`
- **Description:** `Allows SSH access to Linux EC2`
- **VPC:** Select the same default VPC.

### Inbound Rule

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| SSH | TCP | `22` | My IP |

Click **Create security group**.

### Why is this step required?

SSH access is required to connect to the Linux EC2 instance.

### Dependency

Depends on **Step 2**.

### What happens if this step is skipped?

You may not be able to connect to the EC2 instance.

---

# Step 5: Allow EC2 Access to the RDS Security Group

### Navigation

```text
AWS Management Console
→ EC2
→ Security Groups
→ Select MariaDB-RDS-SG
→ Inbound rules
→ Edit inbound rules
→ Add rule
```

### Configuration

The final inbound rules should be:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| MySQL/Aurora | TCP | `3306` | My IP |
| MySQL/Aurora | TCP | `3306` | `MariaDB-EC2-SG` |

Click **Save rules**.

### Why is this step required?

It authorizes instances associated with `MariaDB-EC2-SG` to reach MariaDB on TCP port `3306`.

### Dependency

Depends on **Step 3** and **Step 4**.

### What happens if this step is skipped?

The EC2 instance will not be able to connect to the RDS database.

---

# Step 6: Create a DB Subnet Group

### Navigation

```text
AWS Management Console
→ RDS
→ Subnet groups
→ Create DB subnet group
```

### Configuration

- **Name:** `mariadb-subnet-group`
- **Description:** `DB subnet group for MariaDB assignment`
- **VPC:** Select the default VPC.

Select at least two subnets from different Availability Zones.

Example:

| Availability Zone | Subnet |
|-------------------|--------|
| `ap-south-1a` | `subnet-xxxxxxxxxxxxxxxx1` |
| `ap-south-1b` | `subnet-xxxxxxxxxxxxxxxx2` |

Click **Create**.

### Why is this step required?

The DB subnet group determines which VPC subnets RDS can use.

### Dependency

Depends on **Step 2**.

### What happens if this step is skipped?

You would need to use an existing default DB subnet group instead.

---

# Step 7: Create the MariaDB RDS Database

### Navigation

```text
AWS Management Console
→ RDS
→ Databases
→ Create database
```

### Configuration

#### Database Creation Method

- **Choose a database creation method:** `Standard create`

#### Engine Options

- **Engine type:** `MariaDB`

For the engine version, select a currently supported MariaDB version available in the AWS Console.

#### Templates

Select the lowest-cost appropriate template available for your AWS account, such as:

- **Template:** `Free tier`, if displayed and applicable.

Otherwise select:

- **Template:** `Dev/Test`

Carefully review the estimated cost before creating the database.

#### Availability and Durability

For this assignment, use a single DB instance deployment.

- **Multi-AZ deployment:** `Do not create a standby instance`

This minimizes cost.

#### Settings

- **DB instance identifier:** `mariadb-assignment-db`
- **Master username:** `admin`

For credentials management:

- **Credentials management:** `Self managed`

Create a strong password.

Example placeholder:

```text
YourStrongPasswordHere
```

> Never commit the actual database password to GitHub.

#### Instance Configuration

Select the smallest suitable burstable DB instance class available for your account and Region.

Example:

- **DB instance class:** `db.t3.micro` or another eligible small instance class shown in your console.

#### Storage

Use the minimum suitable storage available in the console.

Example configuration:

- **Storage type:** `General Purpose SSD (gp3)`
- **Allocated storage:** Use the minimum permitted value shown by AWS.
- **Storage autoscaling:** Disable for this short assignment if permitted.

#### Connectivity

Configure:

- **Compute resource:** `Don't connect to an EC2 compute resource`
- **Network type:** `IPv4`
- **VPC:** Default VPC
- **DB subnet group:** `mariadb-subnet-group`
- **Public access:** `Yes`
- **VPC security group:** `Choose existing`
- **Existing VPC security groups:** `MariaDB-RDS-SG`
- **Availability Zone:** `No preference`
- **Database port:** `3306`

> Public access is required in this lab because the Windows SQL client running outside AWS needs to connect directly to the RDS endpoint. Access is still restricted by the RDS security group's `My IP` rule.

#### Database Authentication

- **Database authentication:** `Password authentication`

#### Additional Configuration

Expand **Additional configuration**.

Set:

- **Initial database name:** `companydb`

For this short-lived assignment:

- Disable unnecessary optional paid features.
- Keep monitoring at the basic/default level unless required.
- Do not enable Multi-AZ.
- Review backup settings carefully.

Click:

```text
Create database
```

Wait until:

- **Status:** `Available`

### Why is this step required?

This creates the managed MariaDB relational database required by the assignment.

Amazon RDS manages infrastructure tasks such as database host provisioning and underlying service maintenance.

### Dependency

Depends on:

- Default VPC.
- DB subnet group.
- RDS security group.

### What happens if this step is skipped?

There will be no MariaDB server for the Windows SQL client or Linux EC2 instance to connect to.

---

# Step 8: Record the RDS Endpoint

### Navigation

```text
AWS Management Console
→ RDS
→ Databases
→ mariadb-assignment-db
→ Connectivity & security
```

### Configuration

Record:

- **Endpoint:** `mariadb-assignment-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com`
- **Port:** `3306`

> Your actual endpoint will be different.

### Why is this step required?

The endpoint is the DNS hostname clients use to locate the database server.

### Dependency

Depends on **Step 7**.

### What happens if this step is skipped?

You will not know which hostname to use when connecting from Windows or EC2.

---

# Step 9: Create the Linux EC2 Instance

### Navigation

```text
AWS Management Console
→ EC2
→ Instances
→ Launch instances
```

### Configuration

#### Name

- **Name:** `MariaDB-Linux-Client`

#### AMI

Use:

- **AMI:** `Ubuntu Server 22.04 LTS` or a currently available Ubuntu LTS AMI.

#### Instance Type

Use the smallest suitable Free Tier or credit-eligible instance type shown in your account.

Example:

- **Instance type:** `t2.micro` or `t3.micro`

#### Key Pair

Select an existing key pair or create a new one.

Example:

- **Key pair name:** `mariadb-key`

Download and securely store the `.pem` file if creating a new key pair.

#### Network Settings

Configure:

- **VPC:** Same default VPC used by RDS.
- **Subnet:** Any suitable subnet in the same VPC.
- **Auto-assign public IP:** `Enable`
- **Firewall:** Select existing security group.
- **Security group:** `MariaDB-EC2-SG`

#### Storage

Use the minimum suitable root volume.

Example:

- **Storage:** `8 GiB gp3`

Click:

```text
Launch instance
```

Wait until:

- **Instance state:** `Running`
- **Status checks:** `2/2 checks passed`

### Why is this step required?

The assignment explicitly requires connecting to MariaDB from a Linux-based EC2 instance.

### Dependency

Depends on:

- Default VPC.
- EC2 security group.
- RDS security group authorization.

### What happens if this step is skipped?

The Linux-based EC2 connection requirement cannot be completed.

---

# Step 10: Connect to the Linux EC2 Instance

### Navigation

```text
AWS Management Console
→ EC2
→ Instances
→ Select MariaDB-Linux-Client
→ Connect
```

You may use EC2 Instance Connect if supported, or SSH from your local terminal.

### Command

```bash
ssh -i "mariadb-key.pem" ubuntu@EC2_PUBLIC_IP
```

Replace:

- `mariadb-key.pem` with your actual key file.
- `EC2_PUBLIC_IP` with the public IPv4 address of your instance.

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `ssh` | Secure Shell client used to establish an encrypted remote terminal session. |
| `-i` | Specifies the identity file containing the private key used for authentication. |
| `"mariadb-key.pem"` | Private key file corresponding to the EC2 key pair. |
| `ubuntu` | Default SSH username for an Ubuntu EC2 AMI. |
| `@` | Separates the remote username from the destination hostname or IP address. |
| `EC2_PUBLIC_IP` | Placeholder for the EC2 instance's actual public IPv4 address. |

### Why is this step required?

You need terminal access to install the MariaDB client and connect to RDS.

### Dependency

Depends on **Step 9**.

### What happens if this step is skipped?

You cannot execute Linux commands on the EC2 instance.

---

# Step 11: Update the Ubuntu Package Index

### Command

```bash
sudo apt update
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the following command with administrator privileges. |
| `apt` | Ubuntu's package management command-line utility. |
| `update` | Downloads current package metadata from configured software repositories. |

### Why is this step required?

It ensures the system knows about currently available package versions before installing the MariaDB client.

### Dependency

Requires successful EC2 terminal access.

### What happens if this step is skipped?

Package installation may fail or use outdated package metadata.

---

# Step 12: Install the MariaDB Client

### Command

```bash
sudo apt install mariadb-client -y
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Runs the package installation with administrator privileges. |
| `apt` | Ubuntu package management utility. |
| `install` | Instructs `apt` to install a package. |
| `mariadb-client` | Package containing MariaDB client utilities used to connect to a MariaDB server. |
| `-y` | Automatically answers `yes` to package-manager confirmation prompts. |
| `-` | Introduces the short command-line option `y`. |

### Why is this step required?

The EC2 instance needs database client software to establish a MariaDB protocol connection to RDS.

### Dependency

Depends on **Step 11**.

### What happens if this step is skipped?

The MariaDB client command will not be available.

---

# Step 13: Verify the MariaDB Client Installation

### Command

```bash
mariadb --version
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `mariadb` | Launches the MariaDB command-line client program. |
| `--version` | Displays the installed client version and exits. |
| `--` | Prefix used for a long-form command-line option. |

### Expected Output

Example:

```text
mariadb  Ver 15.1 Distrib 10.x.x-MariaDB, for debian-linux-gnu (x86_64)
```

The exact version can differ.

### Why does this confirm success?

The output proves that the MariaDB client executable is installed and accessible through the shell's command search path.

---

# Step 14: Connect to RDS from the Linux EC2 Instance

### Command

```bash
mariadb -h mariadb-assignment-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p
```

Replace the sample endpoint with your actual RDS endpoint.

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `mariadb` | Starts the MariaDB command-line client. |
| `-h` | Specifies the remote database hostname. |
| `mariadb-assignment-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com` | Example RDS endpoint. Replace it with your actual endpoint. |
| `-P` | Specifies the TCP port number. The uppercase `P` is significant. |
| `3306` | Default TCP port used by MariaDB. |
| `-u` | Specifies the database username. |
| `admin` | Master database username configured during RDS creation. |
| `-p` | Prompts securely for the database password. The password is not written directly in the command. |

Enter the master password when prompted.

### Expected Output

```text
Welcome to the MariaDB monitor.
Commands end with ; or \g.
Your MariaDB connection id is ...
Server version: ...
```

The exact output and server version may differ.

### Why is this step required?

This fulfills the assignment requirement to connect to the MariaDB RDS database from a Linux-based EC2 instance.

### Dependency

Depends on:

- Running RDS database.
- Correct RDS endpoint.
- Port `3306` allowed from `MariaDB-EC2-SG`.
- MariaDB client installation.
- Correct username and password.

### What happens if this step is skipped?

The required Linux EC2-to-RDS connection is not demonstrated.

---

# Step 15: Verify the Database from Linux EC2

After connecting to MariaDB, run:

```sql
SHOW DATABASES;
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `SHOW` | SQL keyword used to display metadata. |
| `DATABASES` | Requests a list of databases visible to the authenticated user. |
| `;` | SQL statement terminator that tells the MariaDB client to execute the statement. |

### Expected Output

```text
+--------------------+
| Database           |
+--------------------+
| companydb          |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

The exact databases displayed may vary.

Next:

```sql
USE companydb;
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `USE` | Selects the active database for subsequent SQL statements. |
| `companydb` | Name of the initial database created for this assignment. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

```text
Database changed
```

---

# Step 16: Create a Test Table

### Command

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(100) NOT NULL
);
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `CREATE TABLE` | Creates a new relational database table. |
| `employees` | Name assigned to the new table. |
| `(` and `)` | Enclose the table's column definitions. |
| `id` | Name of the unique identifier column. |
| `INT` | Integer data type. |
| `PRIMARY KEY` | Uniquely identifies every row in the table. |
| `AUTO_INCREMENT` | Automatically generates the next numeric ID for newly inserted rows. |
| `,` | Separates column definitions. |
| `name` | Column storing employee names. |
| `VARCHAR(100)` | Variable-length text field supporting up to 100 characters. |
| `NOT NULL` | Prevents the column from containing a SQL `NULL` value. |
| `department` | Column storing department names. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

```text
Query OK, 0 rows affected
```

### Why is this step required?

Creating a table confirms that the connection is not merely open; it also verifies that SQL statements can be executed against the database.

### Dependency

Depends on selecting `companydb`.

### What happens if this step is skipped?

You can still prove connectivity, but you will not verify practical data storage operations.

---

# Step 17: Insert Test Data

### Command

```sql
INSERT INTO employees (name, department)
VALUES ('Alice', 'Engineering'),
       ('Bob', 'Finance');
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `INSERT INTO` | SQL statement used to add new rows to a table. |
| `employees` | Target table. |
| `(name, department)` | Specifies the columns receiving values. |
| `VALUES` | Introduces the row data to insert. |
| `'Alice'` | Text value inserted into the `name` column. |
| `'Engineering'` | Text value inserted into the `department` column. |
| `'Bob'` | Second employee name. |
| `'Finance'` | Second employee department. |
| `,` | Separates multiple rows or values, depending on context. |
| `'` | Single quotation marks delimit SQL string literals. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

```text
Query OK, 2 rows affected
```

### Why is this step required?

It confirms that data can be stored in the MariaDB database.

### Dependency

Depends on the `employees` table created in **Step 16**.

### What happens if this step is skipped?

There will be no custom test data available for retrieval verification.

---

# Step 18: Retrieve Test Data

### Command

```sql
SELECT * FROM employees;
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `SELECT` | Retrieves data from a database table. |
| `*` | Wildcard meaning all columns. |
| `FROM` | Specifies the source table. |
| `employees` | Table from which data is retrieved. |
| `;` | Terminates and executes the SQL statement. |

### Expected Output

```text
+----+-------+-------------+
| id | name  | department  |
+----+-------+-------------+
|  1 | Alice | Engineering |
|  2 | Bob   | Finance     |
+----+-------+-------------+
```

### Why does this confirm success?

The result proves that:

- The EC2 instance can reach the RDS database.
- Authentication succeeds.
- The database accepts SQL statements.
- Data can be stored.
- Stored data can be retrieved.

---

# Step 19: Exit the MariaDB Client

### Command

```sql
EXIT;
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `EXIT` | Closes the current MariaDB client session. |
| `;` | Terminates the client command. |

### Expected Output

```text
Bye
```

---

# Step 20: Install MySQL Workbench on Windows

A graphical SQL client can be used to connect from Windows.

For this assignment, use MySQL Workbench or another MariaDB-compatible SQL client.

### Action

Install MySQL Workbench on your Windows computer using the official installer.

During installation, complete the normal installation process and launch MySQL Workbench.

### Why is this step required?

The assignment explicitly requires a connection using a SQL client for Windows.

### Dependency

The RDS database must already exist and be in the `Available` state.

### What happens if this step is skipped?

The Windows SQL client connection requirement will not be demonstrated.

---

# Step 21: Connect to RDS from MySQL Workbench

Open MySQL Workbench.

Create a new connection.

### Configuration

Use:

- **Connection Name:** `AWS MariaDB RDS`
- **Connection Method:** `Standard (TCP/IP)`
- **Hostname:** Your actual RDS endpoint.
- **Port:** `3306`
- **Username:** `admin`
- **Password:** Enter or securely store the RDS master password.

Click:

```text
Test Connection
```

### Expected Output

A successful connection message should appear.

Then open the connection.

### Why is this step required?

This fulfills the requirement to connect to the MariaDB RDS database using a Windows SQL client.

### Dependency

Depends on:

- RDS status being `Available`.
- RDS public access being enabled.
- Your current public IP being allowed in `MariaDB-RDS-SG`.
- Correct endpoint.
- Correct port.
- Correct username and password.

### What happens if this step is skipped?

The Windows SQL client requirement remains incomplete.

---

# Step 22: Retrieve the Same Data from Windows

In the SQL editor, execute:

```sql
USE companydb;

SELECT * FROM employees;
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `USE` | Selects the active database. |
| `companydb` | Database containing the test table. |
| `;` | Terminates the SQL statement. |
| `SELECT` | Retrieves database records. |
| `*` | Requests every column. |
| `FROM` | Specifies the source table. |
| `employees` | Name of the source table. |

### Expected Output

```text
+----+-------+-------------+
| id | name  | department  |
+----+-------+-------------+
|  1 | Alice | Engineering |
|  2 | Bob   | Finance     |
+----+-------+-------------+
```

### Why does this confirm success?

The same data created through the Linux EC2 connection is now accessible through the Windows SQL client.

This demonstrates that:

- RDS centrally stores the data.
- Multiple authorized clients can connect.
- The Windows client connection works.
- The Linux EC2 connection works.
- Stored data can be retrieved as required by the problem statement.

---

# 3. Verification / Output Checking

# Verification Step 1: Verify the RDS Database Status

### Action

Navigate to:

```text
AWS Management Console
→ RDS
→ Databases
→ mariadb-assignment-db
```

### Expected Output

Observe:

- **Status:** `Available`
- **Engine:** `MariaDB`
- **Port:** `3306`

### Why does this confirm success?

The `Available` state confirms that the RDS DB instance has completed provisioning and is ready to accept connections.

---

# Verification Step 2: Verify Linux EC2 Connectivity

### Action

Connect from EC2 using:

```bash
mariadb -h mariadb-assignment-db.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p
```

### Expected Output

```text
Welcome to the MariaDB monitor.
Commands end with ; or \g.
```

### Why does this confirm success?

It confirms successful:

- Network routing.
- Security group authorization.
- DNS resolution.
- TCP connectivity on port `3306`.
- Database authentication.

---

# Verification Step 3: Verify Data Retrieval from Linux

### Command

```sql
USE companydb;

SELECT * FROM employees;
```

### Expected Output

```text
+----+-------+-------------+
| id | name  | department  |
+----+-------+-------------+
|  1 | Alice | Engineering |
|  2 | Bob   | Finance     |
+----+-------+-------------+
```

### Why does this confirm success?

It proves that data is stored in RDS and can be retrieved from the Linux EC2 instance.

---

# Verification Step 4: Verify Windows SQL Client Connectivity

### Action

In MySQL Workbench:

```text
Open AWS MariaDB RDS connection
→ Enter password if prompted
→ Connect
```

### Expected Output

The SQL editor should open successfully without a connection error.

### Why does this confirm success?

It proves that the Windows SQL client can establish an authenticated MariaDB connection to the RDS endpoint.

---

# Verification Step 5: Verify Data Retrieval from Windows

### Command

```sql
USE companydb;

SELECT * FROM employees;
```

### Expected Output

```text
+----+-------+-------------+
| id | name  | department  |
+----+-------+-------------+
|  1 | Alice | Engineering |
|  2 | Bob   | Finance     |
+----+-------+-------------+
```

### Why does this confirm success?

It confirms that the same centralized database and data can be accessed from both required client environments.

---

# 4. Cleanup / Cost Optimization

Resources should be deleted immediately after verification to minimize AWS credit consumption.

The cleanup order is:

```text
Terminate EC2 Instance
        │
        ▼
Delete RDS Database
        │
        ▼
Delete DB Subnet Group
        │
        ▼
Delete RDS Security Group
        │
        ▼
Delete EC2 Security Group
        │
        ▼
Verify No Assignment Resources Remain
```

The default VPC and its default networking resources should not be deleted because they were not created specifically for this assignment.

---

# Cleanup Step 1: Terminate the EC2 Instance

### Navigation

```text
AWS Management Console
→ EC2
→ Instances
→ Select MariaDB-Linux-Client
→ Instance state
→ Terminate instance
```

### Action

Terminate:

- `MariaDB-Linux-Client`

Confirm termination.

### Why is this required?

A running EC2 instance consumes compute resources, and its public IPv4 address can also incur charges.

### What happens if this resource is not deleted?

The instance may continue consuming promotional credits or generating charges.

---

# Cleanup Step 2: Delete the RDS Database

### Navigation

```text
AWS Management Console
→ RDS
→ Databases
→ Select mariadb-assignment-db
→ Actions
→ Delete
```

### Action

Because this is a temporary assignment environment:

- Do not create a final snapshot unless you intentionally want to retain the data and accept potential snapshot storage costs.
- Disable retention options if offered and not required.
- Confirm permanent deletion.

Wait until the DB instance disappears from the database list.

### Why is this required?

The RDS DB instance is one of the primary potentially chargeable resources in this assignment.

### What happens if this resource is not deleted?

The RDS instance may continue consuming AWS credits or generating charges.

---

# Cleanup Step 3: Delete the DB Subnet Group

### Navigation

```text
AWS Management Console
→ RDS
→ Subnet groups
→ Select mariadb-subnet-group
→ Delete
```

### Action

Delete:

- `mariadb-subnet-group`

### Why is this required?

It removes the assignment-specific networking configuration created for RDS.

### What happens if this resource is not deleted?

A DB subnet group itself generally does not create the same type of ongoing compute charge as a running DB instance, but leaving unused resources creates clutter and can cause confusion in future labs.

---

# Cleanup Step 4: Delete the RDS Security Group

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Select MariaDB-RDS-SG
→ Actions
→ Delete security groups
```

### Action

Delete:

- `MariaDB-RDS-SG`

### Why is this required?

The security group was created specifically for this assignment and is no longer required after deleting the database.

### What happens if this resource is not deleted?

It may remain as unused infrastructure and create confusion in future assignments.

---

# Cleanup Step 5: Delete the EC2 Security Group

### Navigation

```text
AWS Management Console
→ EC2
→ Network & Security
→ Security Groups
→ Select MariaDB-EC2-SG
→ Actions
→ Delete security groups
```

### Action

Delete:

- `MariaDB-EC2-SG`

### Why is this required?

It was created specifically for the temporary Linux client instance.

### What happens if this resource is not deleted?

It remains as an unused resource and adds unnecessary clutter to the AWS account.

---

# Cleanup Step 6: Verify That All Assignment Resources Are Deleted

### Navigation

Check the following locations:

```text
AWS Management Console
→ EC2
→ Instances

AWS Management Console
→ EC2
→ Security Groups

AWS Management Console
→ RDS
→ Databases

AWS Management Console
→ RDS
→ Subnet groups
```

### Action

Verify that the following assignment-specific resources no longer exist:

- `MariaDB-Linux-Client`
- `mariadb-assignment-db`
- `mariadb-subnet-group`
- `MariaDB-RDS-SG`
- `MariaDB-EC2-SG`

Do not delete:

- Default VPC.
- Default subnets.
- Default route table.
- Default network ACL.
- Internet gateway associated with the default VPC.

### Why is this required?

A final verification prevents accidentally leaving potentially chargeable resources running.

### What happens if this step is skipped?

You may overlook an active RDS or EC2 resource that continues consuming promotional credits or generating charges.

---
