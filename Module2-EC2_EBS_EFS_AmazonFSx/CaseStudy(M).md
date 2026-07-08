# Module 2 - Introduction to EC2, EBS, EFS and Amazon FSx

# Case Study – EC2, EBS and EFS

## Problem Statement

You work for XYZ Corporation. Your corporation is working on an application and they require secured web servers on Linux to launch the application.

### Tasks To Be Performed

1. Create an instance in the **US-East-1 (N. Virginia)** region with Linux OS and manage the requirement of web servers of your company using AMI.
2. Replicate the instance in the **US-West-2 (Oregon)** region.
3. Build two EBS volumes and attach them to the instance in the **US-East-1 (N. Virginia)** region.
4. Delete one volume after detaching it and extend the size of the other volume.
5. Take backup of the remaining EBS volume.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|-------|
| Amazon EC2 (`t2.micro` or `t3.micro`, depending on account eligibility) | Yes | No (within Free Tier limits) | One Linux instance is covered within Free Tier usage limits. |
| Amazon Machine Image (AMI) | Yes | No | Creating an AMI is free. Charges apply only for snapshots stored beyond Free Tier limits. |
| Amazon EBS General Purpose SSD (`gp3`) | Yes | No (within 30 GB Free Tier limit) | Storage beyond Free Tier consumes credits. |
| Additional EBS Volume | Partially | Possible | Combined EBS storage should remain within Free Tier allocation. |
| EBS Snapshot | Partially | Possible | Snapshot storage is chargeable after Free Tier allowance is exhausted. |
| Cross-Region Deployment | Yes | Possible | Replicated EC2 instance is billable if total running instance hours exceed Free Tier. |
| Data Transfer | Mostly | Possible | Normal management traffic is negligible. Large data transfers may incur charges. |

### Free Tier Analysis

This case study is **mostly eligible under the AWS Free Tier** if:

- You use `t2.micro` (or `t3.micro` where applicable).
- Your total EBS storage remains within the Free Tier allocation.
- You terminate all EC2 instances after completing the assignment.
- You delete all snapshots and AMIs created for the assignment after verification.

### Services That May Consume AWS Promotional Credits

- Running two EC2 instances simultaneously for extended periods.
- Additional EBS storage beyond the Free Tier allocation.
- EBS snapshots.
- Any resources left running after the assignment.

### Recommendation

Since this assignment involves:

- Multiple EC2 instances
- Multiple EBS volumes
- AMI creation
- Cross-region deployment
- EBS snapshots

it is **better to complete this assignment while you still have AWS Promotional Credits**. Although most resources fit within the Free Tier, promotional credits provide an additional safety margin against unexpected charges.

---

# Architecture Diagram

```text
                         AWS Account
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
        │                                           │
 US-East-1 (N. Virginia)                 US-West-2 (Oregon)
        │                                           │
        │                                           │
   Amazon Linux EC2                         EC2 created
        │                                  from copied AMI
        │
        ├──────────────┐
        │              │
        │              │
     EBS Volume 1   EBS Volume 2
        │              │
        │              │
        │          Detached
        │              │
        │          Deleted
        │
Extended to larger size
        │
        │
   EBS Snapshot
```

---

# Implementation

## Step 1: Select the AWS Region

### Navigation

```text
AWS Console
→ Region Selector
→ US East (N. Virginia)
```

### Configuration

| Setting | Value |
|----------|-------|
| Region | `US East (N. Virginia)` |
| Region Code | `us-east-1` |

### Why is this step required?

AWS resources are created inside a specific AWS Region.

The assignment explicitly requires that the primary Linux web server be created in the **US-East-1 (N. Virginia)** region.

Every resource created afterwards (EC2, EBS, AMI, Snapshot) will belong to this region unless another region is selected.

### Dependency

None.

This is the starting point of the assignment.

### What happens if this step is skipped?

Resources may accidentally be created in another region.

Later steps such as:

- attaching EBS volumes,
- creating AMIs,
- copying AMIs,
- launching replicated instances

will fail because AWS resources cannot be attached across different regions.

---

## Step 2: Create a Security Group

### Navigation

```text
AWS Console
→ EC2
→ Network & Security
→ Security Groups
→ Create Security Group
```

### Configuration

| Property | Value |
|----------|-------|
| Security Group Name | `WebServer-SG` |
| Description | `Security Group for Linux Web Server` |
| VPC | `Default VPC` |

### Inbound Rules

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | `22` | `My IP` | Secure remote login |
| HTTP | TCP | `80` | `0.0.0.0/0` | Allow website traffic |
| HTTPS | TCP | `443` | `0.0.0.0/0` | Secure web traffic |

### Outbound Rules

Leave the default rule:

| Type | Destination |
|------|-------------|
| All Traffic | `0.0.0.0/0` |

### Why is this step required?

A Security Group acts as a virtual firewall for an EC2 instance.

It controls:

- who can connect using SSH,
- who can access the web server,
- what outbound traffic is allowed.

Without a Security Group, the EC2 instance would either be inaccessible or overly exposed depending on the configuration.

### Dependency

Depends on:

- Step 1 (Correct AWS Region)

### What happens if this step is skipped?

- SSH login will not work.
- Browser cannot access the web server.
- Future testing of the application will fail.

---

## Step 3: Create the Linux EC2 Instance

### Navigation

```text
AWS Console
→ EC2
→ Instances
→ Launch Instance
```

### Configuration

| Property | Value |
|----------|-------|
| Name | `XYZ-WebServer-East` |
| Region | `US East (N. Virginia)` |
| AMI | `Amazon Linux 2023 AMI` |
| Instance Type | `t2.micro` |
| Key Pair | `Create New Key Pair` |
| Key Pair Name | `Module2-Key` |
| Key Pair Type | `RSA` |
| Private Key Format | `.pem` |
| VPC | `Default VPC` |
| Subnet | Default |
| Auto Assign Public IP | Enabled |
| Security Group | `WebServer-SG` |
| Root Volume | `8 GB gp3` |

### Why is this step required?

Amazon EC2 provides virtual servers in AWS.

This instance will host the Linux web server that XYZ Corporation will later replicate using an AMI.

This is the primary server for all remaining tasks in the assignment.

### Dependency

Depends on:

- Step 1
- Step 2

### What happens if this step is skipped?

No EC2 instance will exist.

As a result, you cannot:

- connect through SSH,
- create an AMI,
- attach EBS volumes,
- resize storage,
- create snapshots,
- replicate the server into another region.

---

## Step 4: Connect to the EC2 Instance Using SSH

### Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select XYZ-WebServer-East
→ Copy Public IPv4 Address
```

### Command

```bash
chmod 400 Module2-Key.pem
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `chmod` | Changes file permissions. |
| `400` | Gives read permission only to the file owner. No write or execute permissions are granted. This satisfies SSH security requirements. |
| `Module2-Key.pem` | The downloaded private key file used to authenticate with the EC2 instance. |

### Command

```bash
ssh -i Module2-Key.pem ec2-user@<Public-IP>
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `ssh` | Secure Shell client used to connect to a remote Linux server. |
| `-i` | Specifies the private key file to use for authentication. |
| `Module2-Key.pem` | Private key downloaded during EC2 creation. |
| `ec2-user` | Default login user for Amazon Linux. |
| `@` | Separates the username from the server address. |
| `<Public-IP>` | Public IPv4 address of the EC2 instance. Replace it with your actual instance IP address. |

### Why is this step required?

SSH provides secure remote access to the Linux server.

Without SSH access, you cannot:

- install software,
- configure the web server,
- verify storage,
- manage the operating system.

### Dependency

Depends on:

- Step 3
- Security Group allowing SSH on port `22`

### What happens if this step is skipped?

You will not be able to administer the Linux server, making the remaining implementation impossible.

---
## Step 5: Update the Linux Operating System

### Navigation

This step is performed inside the EC2 instance through the SSH terminal.

### Commands

```bash
sudo dnf update -y
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with root (administrator) privileges. Updating system packages requires administrative permissions. |
| `dnf` | **Dandified YUM**, the package manager used in Amazon Linux 2023 for installing, updating, and managing software packages. |
| `update` | Updates all installed packages on the operating system to their latest available versions from the configured repositories. |
| `-y` | Automatically answers **Yes** to all confirmation prompts, allowing the update to continue without manual intervention. |

### Why is this step required?

Updating the operating system ensures that:

- The latest security patches are installed.
- Existing software bugs are fixed.
- Newer package versions are available.
- The server is more secure before deploying the web application.

Keeping the operating system updated is considered an AWS and Linux best practice before installing additional software.

### Dependency

Depends on:

- Step 4 (SSH connection established)

### What happens if this step is skipped?

The EC2 instance may contain:

- Known security vulnerabilities
- Outdated software packages
- Compatibility issues with newly installed software

---

## Step 6: Install the Apache Web Server

### Navigation

This step is performed inside the EC2 instance through the SSH terminal.

### Commands

```bash
sudo dnf install httpd -y
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Runs the installation with administrator privileges. |
| `dnf` | Amazon Linux package manager. |
| `install` | Downloads and installs the specified package from the configured repositories. |
| `httpd` | Apache HTTP Server package. This software hosts web applications and websites. |
| `-y` | Automatically confirms the installation prompt. |

### Why is this step required?

Apache is the web server that will serve the company's web application.

Without a web server:

- No website can be hosted.
- HTTP requests cannot be processed.
- Users cannot access the application using a web browser.

### Dependency

Depends on:

- Step 5

### What happens if this step is skipped?

Although the EC2 instance will exist, it will not function as a web server.

---

## Step 7: Start the Apache Service

### Navigation

Performed inside the EC2 instance.

### Commands

```bash
sudo systemctl start httpd
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `systemctl` | Utility used to control services managed by `systemd`. |
| `start` | Starts the specified service immediately. |
| `httpd` | Name of the Apache web server service. |

### Why is this step required?

Installing Apache only copies the software to the system.

Starting the service launches the Apache web server so that it can begin accepting incoming HTTP requests.

### Dependency

Depends on:

- Step 6

### What happens if this step is skipped?

Apache will remain installed but inactive.

Visitors trying to access the website will receive connection errors because the web server is not running.

---

## Step 8: Enable Apache to Start Automatically

### Navigation

Performed inside the EC2 instance.

### Commands

```bash
sudo systemctl enable httpd
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Runs the command as the administrator. |
| `systemctl` | Controls Linux system services. |
| `enable` | Configures the service to automatically start during every system boot. |
| `httpd` | Apache web server service. |

### Why is this step required?

If the EC2 instance is:

- restarted,
- rebooted,
- stopped and started again,

Apache should automatically start without requiring manual intervention.

This ensures continuous availability of the hosted application.

### Dependency

Depends on:

- Step 7

### What happens if this step is skipped?

After every reboot, Apache will remain stopped until someone manually starts it.

The website will become unavailable after every restart.

---

## Step 9: Create a Sample Web Page

### Navigation

Performed inside the EC2 instance.

### Commands

```bash
echo "<h1>XYZ Corporation Web Server - US East (N. Virginia)</h1>" | sudo tee /var/www/html/index.html
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `echo` | Displays the specified text on the standard output (stdout). |
| `"<h1>XYZ Corporation Web Server - US East (N. Virginia)</h1>"` | HTML content that will become the homepage of the website. |
| `\|` | **Pipe operator**. Sends the output of the command on the left (`echo`) as the input to the command on the right (`tee`). |
| `sudo` | Executes the `tee` command with administrator privileges because writing to `/var/www/html` requires root access. |
| `tee` | Reads input from standard input (stdin) and writes it into the specified file while also displaying the content on the terminal. |
| `/var/www/html/index.html` | Default Apache homepage file that is displayed when users browse to the web server. |

### Why is this step required?

This creates a simple web page so that:

- Apache can serve content.
- We can verify that the web server is functioning correctly.
- The AMI created later already contains a configured web server.

### Dependency

Depends on:

- Step 8

### What happens if this step is skipped?

Apache will still run, but visitors may see the default Apache test page or no custom application page.

---

## Step 10: Verify the Web Server

### Navigation

AWS Console

```text
EC2
→ Instances
→ XYZ-WebServer-East
→ Copy Public IPv4 Address
```

Open a web browser and browse to:

```text
http://<Public-IP>
```

### Why is this step required?

Before creating an Amazon Machine Image (AMI), it is important to verify that:

- Apache is installed correctly.
- The service is running.
- The Security Group allows HTTP traffic.
- The web page is accessible over the internet.

Creating an AMI at this stage ensures that every future instance launched from it already contains a working web server.

### Dependency

Depends on:

- Step 9

### What happens if this step is skipped?

You may create an AMI from a server that is incorrectly configured.

Any EC2 instances launched from that AMI will inherit the same configuration issues, requiring additional troubleshooting later.

---

## Step 11: Create an Amazon Machine Image (AMI)

### Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select XYZ-WebServer-East
→ Actions
→ Image and Templates
→ Create Image
```

### Configuration

| Property | Value |
|----------|-------|
| Image Name | `XYZ-WebServer-AMI-East` |
| Image Description | `AMI for XYZ Linux Web Server` |
| Reboot Instance | `Enabled (Recommended)` |

### Why is this step required?

An **Amazon Machine Image (AMI)** is a reusable template containing:

- Operating System
- Installed software
- System configuration
- EBS volume configuration
- Application files

Creating an AMI allows the organization to launch identical web servers whenever needed without repeating the installation and configuration process.

### Dependency

Depends on:

- Step 10

### What happens if this step is skipped?

The EC2 instance cannot be replicated efficiently.

You would need to manually configure every new server from scratch, increasing deployment time and the risk of configuration inconsistencies.

---
## Step 12: Copy the AMI to the US-West-2 (Oregon) Region

### Navigation

```text
AWS Console
→ EC2
→ Images
→ AMIs
→ Select `XYZ-WebServer-AMI-East`
→ Actions
→ Copy AMI
```

### Configuration

| Property | Value |
|----------|-------|
| Destination Region | `US West (Oregon)` |
| Region Code | `us-west-2` |
| Name | `XYZ-WebServer-AMI-West` |
| Description | `Copied AMI for Oregon Region` |
| Encrypt Snapshot | `Leave default (same as source)` |

### Why is this step required?

An AMI is **region-specific**, meaning it can only be used in the AWS Region where it was created.

Since the assignment requires launching an identical EC2 instance in **US-West-2 (Oregon)**, the AMI must first be copied to that region.

Copying the AMI also copies the underlying EBS snapshot so that the new region has everything needed to launch an identical EC2 instance.

### Dependency

Depends on:

- Step 11 (AMI created in `us-east-1`)

### What happens if this step is skipped?

The AMI will only exist in **US-East-1 (N. Virginia)**.

AWS will not allow you to launch an EC2 instance from an AMI that resides in a different region.

---

## Step 13: Switch to the US-West-2 (Oregon) Region

### Navigation

```text
AWS Console
→ Region Selector
→ US West (Oregon)
```

### Configuration

| Setting | Value |
|----------|-------|
| Region | `US West (Oregon)` |
| Region Code | `us-west-2` |

### Why is this step required?

AWS resources are managed independently in each AWS Region.

To launch the replicated EC2 instance from the copied AMI, all subsequent actions must be performed in the **US-West-2** region.

### Dependency

Depends on:

- Step 12

### What happens if this step is skipped?

You will still be working in the **US-East-1** region and will not see the copied AMI.

---

## Step 14: Create a Security Group in the Oregon Region

### Navigation

```text
AWS Console
→ EC2
→ Network & Security
→ Security Groups
→ Create Security Group
```

### Configuration

| Property | Value |
|----------|-------|
| Security Group Name | `WebServer-SG-West` |
| Description | `Security Group for Oregon Web Server` |
| VPC | `Default VPC` |

### Inbound Rules

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | `22` | `My IP` | Remote administration |
| HTTP | TCP | `80` | `0.0.0.0/0` | Web traffic |
| HTTPS | TCP | `443` | `0.0.0.0/0` | Secure web traffic |

### Outbound Rules

Leave the default rule.

| Type | Destination |
|------|-------------|
| All Traffic | `0.0.0.0/0` |

### Why is this step required?

Security Groups are regional resources.

The Security Group created in **US-East-1** cannot be reused in **US-West-2**.

A new Security Group must therefore be created before launching the replicated EC2 instance.

### Dependency

Depends on:

- Step 13

### What happens if this step is skipped?

The EC2 launch wizard will not have an appropriate Security Group available.

Even if an instance launches, SSH or HTTP access may fail due to incorrect firewall settings.

---

## Step 15: Launch the Replicated EC2 Instance in Oregon

### Navigation

```text
AWS Console
→ EC2
→ AMIs
→ Select `XYZ-WebServer-AMI-West`
→ Launch Instance from AMI
```

### Configuration

| Property | Value |
|----------|-------|
| Name | `XYZ-WebServer-West` |
| AMI | `XYZ-WebServer-AMI-West` |
| Instance Type | `t2.micro` |
| Key Pair | `Module2-Key` (or create a new key pair if preferred) |
| Security Group | `WebServer-SG-West` |
| VPC | `Default VPC` |
| Auto Assign Public IP | Enabled |

### Why is this step required?

This launches a new EC2 instance using the copied AMI.

Because the AMI already contains:

- Amazon Linux
- Apache Web Server
- Website files
- System configuration

the new EC2 instance is immediately ready to function as a web server without repeating the installation process.

This demonstrates one of the primary advantages of using AMIs for rapid server deployment.

### Dependency

Depends on:

- Step 12
- Step 14

### What happens if this step is skipped?

The web server will not be replicated to the Oregon region, and Task 2 of the case study will remain incomplete.

---

## Step 16: Verify the Replicated EC2 Instance

### Navigation

```text
AWS Console
→ EC2
→ Instances
→ XYZ-WebServer-West
→ Copy Public IPv4 Address
```

Open a browser and navigate to:

```text
http://<Public-IP>
```

### Expected Result

The browser should display:

```text
XYZ Corporation Web Server - US East (N. Virginia)
```

Although the text references the original region, this is expected because the webpage was copied as part of the AMI.

This confirms that:

- The operating system was copied successfully.
- Apache is installed and running.
- The website files were replicated.
- The AMI deployment worked correctly.

### Why is this step required?

Verification ensures that the copied AMI functions correctly in the destination region before proceeding with storage-related tasks.

### Dependency

Depends on:

- Step 15

### What happens if this step is skipped?

You may not realize that the replicated server has configuration or networking issues until much later, making troubleshooting more difficult.

---

# Task 3 - Build Two EBS Volumes and Attach Them to the EC2 Instance in US-East-1

## Step 17: Switch Back to the US-East-1 (N. Virginia) Region

### Navigation

```text
AWS Console
→ Region Selector
→ US East (N. Virginia)
```

### Configuration

| Setting | Value |
|----------|-------|
| Region | `US East (N. Virginia)` |

### Why is this step required?

The assignment specifies that both additional EBS volumes must be created and attached to the EC2 instance in the **US-East-1** region.

Since EBS volumes are regional resources, they must exist in the same region—and the same Availability Zone—as the EC2 instance they will be attached to.

### Dependency

Depends on:

- Step 16

### What happens if this step is skipped?

You would remain in the Oregon region and create EBS volumes there, which cannot be attached to the EC2 instance in N. Virginia.

---

## Step 18: Identify the Availability Zone of the EC2 Instance

### Navigation

```text
AWS Console
→ EC2
→ Instances
→ Select `XYZ-WebServer-East`
→ Details Tab
```

### Configuration

Locate and note the following information:

| Property | Example |
|----------|---------|
| Availability Zone | `us-east-1a` |

> **Important:** Your Availability Zone may be different (for example, `us-east-1b` or `us-east-1c`). Use the value displayed for your EC2 instance.

### Why is this step required?

An Amazon EBS volume can only be attached to an EC2 instance if both resources are in the **same Availability Zone (AZ)**.

For example:

- EC2 in `us-east-1a` → EBS must also be in `us-east-1a`
- EC2 in `us-east-1b` → EBS must also be in `us-east-1b`

AWS will not allow an EBS volume to be attached across different Availability Zones.

### Dependency

Depends on:

- Step 17

### What happens if this step is skipped?

You may accidentally create EBS volumes in the wrong Availability Zone.

As a result:

- The **Attach Volume** option will fail.
- AWS will display an error because cross-AZ EBS attachment is not supported.

---

## Step 19: Create the First EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Create Volume
```

### Configuration

| Property | Value |
|----------|-------|
| Volume Type | `gp3` |
| Size | `8 GiB` |
| Availability Zone | Same as the EC2 instance (for example, `us-east-1a`) |
| Name Tag | `EBS-Volume-1` |

### Why is this step required?

This creates the first additional block storage device that will later be attached to the EC2 instance.

Unlike the root volume, this EBS volume provides separate persistent storage that can be managed independently.

### Dependency

Depends on:

- Step 18

### What happens if this step is skipped?

The EC2 instance will not have the first additional storage device required by the assignment.

---

## Step 20: Create the Second EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Create Volume
```

### Configuration

| Property | Value |
|----------|-------|
| Volume Type | `gp3` |
| Size | `8 GiB` |
| Availability Zone | Same as the EC2 instance (for example, `us-east-1a`) |
| Name Tag | `EBS-Volume-2` |

> **Important:** The Availability Zone **must exactly match** the Availability Zone of `XYZ-WebServer-East`. Otherwise, the volume cannot be attached.

### Why is this step required?

The case study specifically requires creating **two separate EBS volumes** and attaching them to the EC2 instance.

Having two independent EBS volumes demonstrates how AWS allows multiple block storage devices to be attached to a single virtual machine.

### Dependency

Depends on:

- Step 18 (Availability Zone identified)

### What happens if this step is skipped?

Only one additional EBS volume will exist, making the assignment incomplete.

---

## Step 21: Attach the First EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-1`
→ Actions
→ Attach Volume
```

### Configuration

| Property | Value |
|----------|-------|
| Instance | `XYZ-WebServer-East` |
| Device Name | `/dev/xvdf` |

### Why is this step required?

Creating an EBS volume only allocates storage in AWS.

The EC2 instance cannot use the storage until the volume is attached.

After attachment, Linux detects the EBS volume as a new block storage device.

### Dependency

Depends on:

- Step 19

### What happens if this step is skipped?

The volume will remain unattached.

The operating system will not detect or use the storage.

---

## Step 22: Attach the Second EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-2`
→ Actions
→ Attach Volume
```

### Configuration

| Property | Value |
|----------|-------|
| Instance | `XYZ-WebServer-East` |
| Device Name | `/dev/xvdg` |

### Why is this step required?

This makes the second EBS volume available to the EC2 instance.

Each attached volume appears as a separate block storage device inside Linux.

### Dependency

Depends on:

- Step 20

### What happens if this step is skipped?

The second EBS volume cannot be used and Task 3 will remain incomplete.

---

## Step 23: Verify That Linux Detects Both EBS Volumes

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
lsblk
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `lsblk` | Lists all available block storage devices connected to the Linux system, including disks, partitions, and mount points. |

### Expected Output

Your output will be similar to:

```text
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda      ...      8G   0 disk
└─xvda1            8G   0 part /

xvdf      ...      8G   0 disk

xvdg      ...      8G   0 disk
```

### Why is this step required?

This verifies that Linux has successfully detected both newly attached EBS volumes.

At this stage:

- the disks exist,
- but they are **not yet formatted**,
- and they are **not yet mounted**.

### Dependency

Depends on:

- Step 21
- Step 22

### What happens if this step is skipped?

You may continue with formatting commands using incorrect device names, potentially formatting the wrong disk.

---

## Step 24: Create an EXT4 File System on the First EBS Volume

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo mkfs -t ext4 /dev/xvdf
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `mkfs` | **Make File System**. Creates a file system on a storage device. |
| `-t` | Specifies the type of file system to create. |
| `ext4` | The Linux Extended File System version 4. It is one of the most widely used Linux file systems. |
| `/dev/xvdf` | The first attached EBS volume. |

### Why is this step required?

A newly created EBS volume is a raw block storage device.

Linux cannot store files on it until a file system is created.

Formatting prepares the volume for file storage.

### Dependency

Depends on:

- Step 23

### What happens if this step is skipped?

Linux cannot mount or store files on the EBS volume.

---

## Step 25: Create an EXT4 File System on the Second EBS Volume

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo mkfs -t ext4 /dev/xvdg
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `mkfs` | Creates a new file system. |
| `-t` | Specifies the file system type. |
| `ext4` | Linux EXT4 file system. |
| `/dev/xvdg` | The second attached EBS volume. |

### Why is this step required?

Both EBS volumes require a file system before they can be mounted and used.

### Dependency

Depends on:

- Step 23

### What happens if this step is skipped?

The second EBS volume will remain unusable for storing files.

---

## Step 26: Create Mount Point Directories

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo mkdir /ebs1
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `mkdir` | Creates a new directory. |
| `/ebs1` | Directory that will act as the mount point for the first EBS volume. |

### Command

```bash
sudo mkdir /ebs2
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `mkdir` | Creates a new directory. |
| `/ebs2` | Directory that will act as the mount point for the second EBS volume. |

### Why is this step required?

Linux mounts storage devices onto directories.

These directories become the access points for the two EBS volumes.

### Dependency

Depends on:

- Step 24
- Step 25

### What happens if this step is skipped?

The mount commands will fail because the destination directories do not exist.

---

## Step 27: Mount Both EBS Volumes

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo mount /dev/xvdf /ebs1
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Runs the command with administrator privileges. |
| `mount` | Attaches a file system to a directory in the Linux file system hierarchy. |
| `/dev/xvdf` | First EBS volume. |
| `/ebs1` | Mount point directory for the first EBS volume. |

### Command

```bash
sudo mount /dev/xvdg /ebs2
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Runs the command with administrator privileges. |
| `mount` | Mounts a storage device. |
| `/dev/xvdg` | Second EBS volume. |
| `/ebs2` | Mount point directory for the second EBS volume. |

### Why is this step required?

Mounting connects the formatted EBS volumes to the Linux file system so users and applications can read from and write to them.

Without mounting, the storage remains inaccessible.

### Dependency

Depends on:

- Step 26

### What happens if this step is skipped?

The EBS volumes will exist but cannot be accessed through the operating system.

---

## Step 28: Verify That Both EBS Volumes Are Mounted

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
df -h
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `df` | Displays disk space usage for mounted file systems. |
| `-h` | Displays sizes in a human-readable format such as KB, MB, or GB. |

### Expected Output

Example:

```text
Filesystem      Size Used Avail Use% Mounted on
/dev/xvda1       8G   ...
/dev/xvdf        8G   ...      /ebs1
/dev/xvdg        8G   ...      /ebs2
```

### Why is this step required?

This confirms that:

- both EBS volumes are mounted,
- Linux recognizes them,
- they are ready to store data.

### Dependency

Depends on:

- Step 27

### What happens if this step is skipped?

You cannot confirm that the storage configuration was successful before modifying or deleting volumes in the next task.

---
# Task 4 - Delete One EBS Volume After Detaching It and Extend the Size of the Other Volume

## Step 29: Unmount the Second EBS Volume

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo umount /ebs2
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `umount` | Detaches a mounted file system from the Linux directory structure. (Notice the command is `umount`, not `unmount`.) |
| `/ebs2` | The mount point of the second EBS volume that will be detached and deleted. |

### Why is this step required?

Before an EBS volume is detached from an EC2 instance, its file system should be unmounted.

Unmounting ensures:

- All pending data is written to disk.
- No applications are actively using the volume.
- The file system remains consistent.

### Dependency

Depends on:

- Step 28 (Both EBS volumes mounted successfully)

### What happens if this step is skipped?

Detaching a mounted volume may result in:

- Data corruption
- Incomplete writes
- File system inconsistencies

---

## Step 30: Detach the Second EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-2`
→ Actions
→ Detach Volume
```

### Action

Detach the volume from the `XYZ-WebServer-East` EC2 instance.

### Why is this step required?

AWS does not allow an attached EBS volume to be deleted.

The volume must first be detached so that it is no longer associated with any EC2 instance.

### Dependency

Depends on:

- Step 29

### What happens if this step is skipped?

The **Delete Volume** option will either be unavailable or AWS will prevent the deletion because the volume is still attached.

---

## Step 31: Delete the Second EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-2`
→ Actions
→ Delete Volume
```

### Action

Delete the detached EBS volume.

### Why is this step required?

The assignment specifically requires deleting one of the two EBS volumes after detaching it.

Deleting unused storage also helps reduce AWS costs.

### Dependency

Depends on:

- Step 30

### What happens if this step is skipped?

The assignment requirement will not be fulfilled.

Additionally, the unused EBS volume will continue to incur storage charges until it is deleted.

---

## Step 32: Modify the Size of the Remaining EBS Volume

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-1`
→ Actions
→ Modify Volume
```

### Configuration

| Property | Old Value | New Value |
|----------|-----------|-----------|
| Size | `8 GiB` | `16 GiB` |

> You may choose a larger size if required. For this case study, increasing from **8 GiB** to **16 GiB** clearly demonstrates volume expansion.

### Why is this step required?

Amazon EBS allows storage to be expanded without recreating the volume.

Increasing the volume size provides additional storage capacity while preserving the existing data.

### Dependency

Depends on:

- Step 31

### What happens if this step is skipped?

The remaining EBS volume will continue using its original size, and the assignment requirement to extend the volume will remain incomplete.

---

## Step 33: Verify That the Operating System Detects the New Disk Size

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
lsblk
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `lsblk` | Lists all available block devices and their sizes. |

### Expected Output

Example:

```text
NAME    SIZE
xvda      8G
xvdf     16G
```

The device `/dev/xvdf` should now display the updated size.

### Why is this step required?

This confirms that Linux recognizes the larger EBS volume before resizing the file system.

### Dependency

Depends on:

- Step 32

### What happens if this step is skipped?

You cannot verify whether AWS has successfully completed the storage expansion.

---

## Step 34: Expand the EXT4 File System

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
sudo resize2fs /dev/xvdf
```

resize2fs is correct only if the EBS volume uses the EXT4 file system, which we created earlier with mkfs -t ext4. So the command is correct for this assignment. If we had used XFS instead, the command would have been `xfs_growfs`.

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | Executes the command with administrator privileges. |
| `resize2fs` | Expands an existing EXT2/EXT3/EXT4 file system to utilize additional disk space. |
| `/dev/xvdf` | The EBS volume whose file system is being expanded. |

### Why is this step required?

Although the EBS volume has been increased to **16 GiB**, the Linux file system still occupies only the original **8 GiB**.

`resize2fs` expands the file system so that the operating system can use the newly available storage.

### Dependency

Depends on:

- Step 33

### What happens if this step is skipped?

Linux will continue using only the original storage capacity even though AWS has allocated a larger EBS volume.

---

## Step 35: Verify the Expanded File System

### Navigation

Performed inside the EC2 instance through SSH.

### Command

```bash
df -h
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `df` | Displays mounted file systems and their disk usage. |
| `-h` | Shows sizes in a human-readable format. |

### Expected Output

Example:

```text
Filesystem      Size Used Avail Mounted on
/dev/xvdf       16G   ...
```

### Why is this step required?

This confirms that:

- AWS expanded the EBS volume.
- Linux expanded the EXT4 file system.
- The additional storage is available for use.

### Dependency

Depends on:

- Step 34

### What happens if this step is skipped?

You cannot confirm that the storage expansion was successful.

---

# Task 5 - Take Backup of the Remaining EBS Volume

## Step 36: Create an EBS Snapshot

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
→ Select `EBS-Volume-1`
→ Actions
→ Create Snapshot
```

### Configuration

| Property | Value |
|----------|-------|
| Description | `Backup of Extended EBS Volume` |
| Tags (Optional) | `Name = EBS-Volume-1-Backup` |

### Why is this step required?

An **EBS Snapshot** is a point-in-time backup of an EBS volume stored in Amazon S3 (managed internally by AWS).

Snapshots can be used to:

- Recover lost data
- Restore deleted volumes
- Create new EBS volumes
- Create AMIs

### Dependency

Depends on:

- Step 35

### What happens if this step is skipped?

No backup of the EBS volume will exist.

If the volume becomes corrupted or is accidentally deleted, the stored data cannot be recovered.

---

# Verification

## Verification Step 1

### Action

Verify that both EC2 instances exist.

### Navigation

```text
AWS Console
→ EC2
→ Instances
```

### Expected Output

You should see:

| Instance Name | Region | State |
|--------------|--------|-------|
| `XYZ-WebServer-East` | `us-east-1` | Running |
| `XYZ-WebServer-West` | `us-west-2` | Running |

### Why does this confirm success?

This confirms that:

- The original EC2 instance was created.
- The AMI was copied successfully.
- The replicated EC2 instance was launched in Oregon.

---

## Verification Step 2

### Action

Open the public IPv4 address of both EC2 instances in a web browser.

### Expected Output

The sample webpage should load successfully.

Example:

```text
XYZ Corporation Web Server - US East (N. Virginia)
```

### Why does this confirm success?

This confirms:

- Apache is running.
- HTTP traffic is allowed.
- The web server is functioning correctly.

---

## Verification Step 3

### Action

Verify the mounted EBS volume.

### Command

```bash
df -h
```

### Command Explanation

| Command Part | Explanation |
|-------------|-------------|
| `df` | Displays mounted file systems. |
| `-h` | Displays disk sizes in a human-readable format. |

### Expected Output

The remaining EBS volume should display a size of approximately **16 GiB**.

### Why does this confirm success?

This confirms that:

- The EBS volume was successfully expanded.
- The Linux file system was resized correctly.

---

## Verification Step 4

### Action

Verify that the snapshot exists.

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Snapshots
```

### Expected Output

A completed snapshot for `EBS-Volume-1` should be listed.

### Why does this confirm success?

This confirms that the backup of the remaining EBS volume was created successfully.

---
# Cleanup / Cost Optimization

> **Important:** Perform the cleanup only **after** completing all verification steps. Deleting these resources earlier will prevent you from verifying the case study.

The following cleanup order follows AWS best practices and ensures that dependent resources are removed correctly while minimizing AWS charges.

---

## Cleanup Dependency Flow

```text
EBS Snapshot
      │
      ▼
Replicated EC2 Instance (US-West-2)
      │
      ▼
Original EC2 Instance (US-East-1)
      │
      ▼
Remaining EBS Volume (if not automatically deleted)
      │
      ▼
Copied AMI (US-West-2)
      │
      ▼
Original AMI (US-East-1)
      │
      ▼
Associated EBS Snapshots created by the AMIs
      │
      ▼
Security Groups (if no longer associated with any resource)
```

> **Note:** AWS prevents deletion of certain resources while they are still being used. Following the dependency order avoids these errors.

---

# Cleanup in US-West-2 (Oregon)

## Cleanup Step 1: Terminate the Replicated EC2 Instance

### Navigation

```text
AWS Console
→ Region Selector
→ US West (Oregon)
→ EC2
→ Instances
→ Select `XYZ-WebServer-West`
→ Instance State
→ Terminate Instance
```

### Action

Terminate the replicated EC2 instance.

### Why is this required?

The EC2 instance continues consuming compute resources while it is running.

Terminating it stops all compute charges associated with the instance.

### What happens if this resource is not deleted?

- Compute charges continue.
- Dependent resources such as AMIs may remain in use.
- The assignment resources remain active unnecessarily.

---

## Cleanup Step 2: Deregister the Copied AMI

### Navigation

```text
AWS Console
→ EC2
→ Images
→ AMIs
→ Select `XYZ-WebServer-AMI-West`
→ Actions
→ Deregister AMI
```

### Action

Deregister the copied AMI.

### Why is this required?

The copied AMI is no longer needed after the assignment.

Removing unused AMIs simplifies resource management.

### What happens if this resource is not deleted?

The AMI remains in your AWS account.

Its associated snapshots may continue consuming storage.

---

## Cleanup Step 3: Delete the Copied AMI Snapshot

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Snapshots
```

Locate the snapshot created automatically when the copied AMI was created.

Delete it.

### Why is this required?

Although the AMI has been deregistered, its underlying snapshot still occupies storage.

Deleting the snapshot releases the storage.

### What happens if this resource is not deleted?

Snapshot storage charges may continue.

---

# Cleanup in US-East-1 (N. Virginia)

## Cleanup Step 4: Terminate the Original EC2 Instance

### Navigation

```text
AWS Console
→ Region Selector
→ US East (N. Virginia)
→ EC2
→ Instances
→ Select `XYZ-WebServer-East`
→ Instance State
→ Terminate Instance
```

### Action

Terminate the original EC2 instance.

### Why is this required?

This stops compute usage and completes the removal of the primary server created during the assignment.

### What happens if this resource is not deleted?

The EC2 instance continues consuming compute resources.

---

## Cleanup Step 5: Delete the Remaining EBS Volume (If Required)

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Volumes
```

### Action

Locate `EBS-Volume-1`.

If it still exists after instance termination, delete it.

> **Note:** The root volume may be deleted automatically if **Delete on Termination** was enabled. However, manually created EBS volumes are **not** deleted automatically unless specifically configured.

### Why is this required?

Manually created EBS volumes continue consuming storage even after the EC2 instance has been terminated.

Deleting them prevents unnecessary storage charges.

### What happens if this resource is not deleted?

Storage charges continue even though the EC2 instance no longer exists.

---

## Cleanup Step 6: Delete the Backup Snapshot

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Snapshots
→ Select `EBS-Volume-1-Backup`
→ Delete Snapshot
```

### Action

Delete the snapshot created during Task 5.

### Why is this required?

Snapshots consume storage in AWS.

Deleting the backup removes the associated storage.

### What happens if this resource is not deleted?

Snapshot storage charges continue until it is deleted.

---

## Cleanup Step 7: Deregister the Original AMI

### Navigation

```text
AWS Console
→ EC2
→ Images
→ AMIs
→ Select `XYZ-WebServer-AMI-East`
→ Actions
→ Deregister AMI
```

### Action

Deregister the original AMI.

### Why is this required?

The AMI is no longer required after completing the assignment.

Removing unused AMIs helps keep the AWS account organized.

### What happens if this resource is not deleted?

The AMI remains available for launching instances.

Its associated snapshots also remain unless deleted separately.

---

## Cleanup Step 8: Delete the Original AMI Snapshot

### Navigation

```text
AWS Console
→ EC2
→ Elastic Block Store
→ Snapshots
```

Locate the snapshot associated with `XYZ-WebServer-AMI-East` and delete it.

### Why is this required?

An AMI is backed by one or more EBS snapshots.

Deleting the AMI does **not** automatically delete those snapshots.

Removing them frees the underlying storage.

### What happens if this resource is not deleted?

Snapshot storage charges continue.

---

## Cleanup Step 9: Delete Unused Security Groups

### Navigation

```text
AWS Console
→ EC2
→ Network & Security
→ Security Groups
```

Delete:

- `WebServer-SG`
- `WebServer-SG-West`

only if AWS reports that they are no longer associated with any resources.

### Why is this required?

Unused Security Groups do not incur charges, but deleting them keeps the AWS environment clean and avoids confusion in future assignments.

### What happens if this resource is not deleted?

There is typically **no direct cost**, but unused Security Groups accumulate over time and make resource management more difficult.

---
















