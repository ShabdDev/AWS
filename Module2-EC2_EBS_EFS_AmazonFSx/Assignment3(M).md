# Module 2 - Introduction to EC2, EBS, EFS and Amazon FSx

# Assignment 3 - EC2 and Amazon EFS

## Problem Statement

Create an Amazon Elastic File System (Amazon EFS) and connect it to **three different EC2 instances**. 
Each EC2 instance should use a different operating system.

Required Operating Systems:

- Ubuntu Server
- Red Hat Enterprise Linux (RHEL)
- Amazon Linux 2

The same EFS should be mounted on all three EC2 instances so that they share the same storage.

---

# Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|-------|
| Amazon EC2 (t2.micro/t3.micro depending on Region) | Yes | No (within Free Tier limits) | Three instances are required. Running them for a short duration is recommended. |
| Amazon EBS | Yes | No | Root volumes are included within Free Tier storage limits. |
| Amazon EFS | No | Yes | Amazon EFS is not included in AWS Free Tier. Storage charges apply. |
| VPC | Yes | No | No additional cost. |
| Subnet | Yes | No | No additional cost. |
| Route Table | Yes | No | No additional cost. |
| Internet Gateway | Yes | No | No additional cost. |
| Security Group | Yes | No | No additional cost. |
| Elastic IP | Conditional | Possible | Avoid allocating unused Elastic IPs. |
| Key Pair | Yes | No | No additional cost. |

---

## Free Tier Eligibility

This assignment is **NOT completely Free Tier eligible** because **Amazon EFS is a paid service**.

---

## AWS Services That Consume Credits

- Amazon EFS Storage
- Amazon EFS Mount Targets (if applicable in your Region)

---

## Recommendation

It is recommended to perform this assignment while you still have AWS Promotional Credits because Amazon EFS is a paid service. Complete the assignment, verify the results, and delete all resources immediately to minimize AWS credit usage.

---

# Architecture Diagram

```text
                        Internet
                            │
                     Internet Gateway
                            │
                     ┌───────────────┐
                     │      VPC      │
                     │ 10.0.0.0/16   │
                     └───────┬───────┘
                             │
                      Public Subnet
                    10.0.1.0/24
                             │
     ┌──────────────┬───────────────┬───────────────┐
     │              │               │
┌───────────┐ ┌────────────┐ ┌──────────────┐
│ Ubuntu    │ │ Red Hat    │ │ Amazon Linux │
│ EC2        │ │ EC2        │ │ EC2          │
└─────┬──────┘ └─────┬──────┘ └──────┬───────┘
      │              │               │
      └──────────────┼───────────────┘
                     │
               Amazon EFS
          Shared Network Storage
```

---

# Implementation Flow

```text
Create VPC
      │
      ▼
Create Public Subnet
      │
      ▼
Create Internet Gateway
      │
      ▼
Configure Route Table
      │
      ▼
Create Security Groups
      │
      ▼
Launch Ubuntu EC2
      │
      ▼
Launch Red Hat EC2
      │
      ▼
Launch Amazon Linux 2 EC2
      │
      ▼
Create Amazon EFS
      │
      ▼
Create Mount Targets
      │
      ▼
Install NFS Utilities
      │
      ▼
Mount EFS on all Instances
      │
      ▼
Verify Shared Storage
```

---

# Step 1: Create a VPC

## Navigation

```text
AWS Console
→ VPC
→ Your VPCs
→ Create VPC
```

## Configuration

| Property | Value |
|----------|-------|
| Resources to Create | VPC Only |
| Name | `Assignment-VPC` |
| IPv4 CIDR | `10.0.0.0/16` |
| IPv6 CIDR | `No IPv6 CIDR Block` |
| Tenancy | `Default` |

---

## Why is this step required?

Amazon EFS and Amazon EC2 must exist inside a Virtual Private Cloud (VPC).

The VPC provides an isolated virtual network where AWS resources can communicate securely.

Without a VPC, EC2 instances cannot communicate with Amazon EFS.

---

## Dependency

This is the first networking resource.

No previous AWS resources are required.

---

## What happens if this step is skipped?

You will not have a network to launch EC2 instances.

Amazon EFS also requires a VPC for creating Mount Targets.

The remaining assignment cannot be completed.

---

# Step 2: Create a Public Subnet

## Navigation

```text
AWS Console
→ VPC
→ Subnets
→ Create Subnet
```

## Configuration

| Property | Value |
|----------|-------|
| VPC | `Assignment-VPC` |
| Name | `Public-Subnet` |
| Availability Zone | Select any available AZ |
| IPv4 CIDR | `10.0.1.0/24` |

---

## Why is this step required?

A subnet divides the VPC into smaller networks.

EC2 instances are launched inside subnets.

Amazon EFS Mount Targets are also created inside subnets.

---

## Dependency

Depends on:

- Assignment-VPC

---

## What happens if this step is skipped?

No subnet will exist.

You will not be able to launch EC2 instances.

Amazon EFS Mount Targets also cannot be created.

---

# Step 3: Create an Internet Gateway

## Navigation

```text
AWS Console
→ VPC
→ Internet Gateways
→ Create Internet Gateway
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `Assignment-IGW` |

After creating the Internet Gateway:

```text
Select Internet Gateway

Actions

Attach to VPC

Assignment-VPC
```

---

## Why is this step required?

The Internet Gateway allows communication between the VPC and the Internet.

Without it, SSH access to EC2 instances from your computer will not work.

Package installation such as NFS utilities will also fail because Internet access is required.

---

## Dependency

Depends on:

- Assignment-VPC

---

## What happens if this step is skipped?

EC2 instances will not have Internet connectivity.

SSH connections will fail.

Package installation commands will fail.

Amazon EFS mounting packages cannot be installed.

---

# Step 4: Create a Route Table

## Navigation

```text
AWS Console
→ VPC
→ Route Tables
→ Create Route Table
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `Public-RT` |
| VPC | `Assignment-VPC` |

After creating the Route Table:

Edit Routes

Add Route

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Assignment-IGW` |

Next,

Subnet Associations

Associate

Select:

- Public-Subnet

Save Associations

---

## Why is this step required?

A Route Table determines where network traffic should be sent.

The route `0.0.0.0/0` tells AWS to send Internet traffic through the Internet Gateway.

---

## Dependency

Depends on:

- Assignment-VPC
- Assignment-IGW
- Public-Subnet

---

## What happens if this step is skipped?

The subnet will remain private.

Even though an Internet Gateway exists, EC2 instances cannot access the Internet.

SSH and software installation will fail.

---

# Step 5: Create Security Groups

Two Security Groups will be created.

1. EC2 Security Group
2. Amazon EFS Security Group

This separation follows AWS security best practices.

---

# Step 5: Create Security Groups

Two Security Groups will be created:

1. EC2 Security Group
2. Amazon EFS Security Group

Using separate Security Groups follows the AWS principle of least privilege by allowing only the required traffic between EC2 instances and Amazon EFS.

---

# Step 5.1: Create EC2 Security Group

## Navigation

```text
AWS Console
→ VPC
→ Security Groups
→ Create Security Group
```

## Configuration

| Property | Value |
|----------|-------|
| Security Group Name | `EC2-SG` |
| Description | `Security Group for EC2 Instances` |
| VPC | `Assignment-VPC` |

### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | `22` | `My IP` |

> **Note:** Selecting **My IP** allows SSH access only from your current public IP address, making it more secure than allowing access from anywhere.

### Outbound Rules

Keep the default outbound rule.

| Type | Protocol | Port | Destination |
|------|----------|------|-------------|
| All Traffic | All | All | `0.0.0.0/0` |

---

## Why is this step required?

The EC2 Security Group acts as a virtual firewall for all three EC2 instances.

It allows:

- SSH access from your computer.
- Outbound communication to download software packages.
- Communication with Amazon EFS.

---

## Dependency

Depends on:

- Assignment-VPC

---

## What happens if this step is skipped?

The EC2 instances will launch, but:

- SSH connection will fail.
- Internet package downloads will fail.
- Amazon EFS communication will not be possible.

---

# Step 5.2: Create Amazon EFS Security Group

## Navigation

```text
AWS Console
→ VPC
→ Security Groups
→ Create Security Group
```

## Configuration

| Property | Value |
|----------|-------|
| Security Group Name | `EFS-SG` |
| Description | `Security Group for Amazon EFS` |
| VPC | `Assignment-VPC` |

### Inbound Rules

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| NFS | TCP | `2049` | `EC2-SG` |

> Select **Security Group** as the source and choose `EC2-SG` instead of entering an IP address.

### Outbound Rules

Keep the default rule.

---

## Why is this step required?

Amazon EFS communicates using the **Network File System (NFS)** protocol.

NFS uses **TCP Port 2049**.

Only EC2 instances that belong to `EC2-SG` should be allowed to communicate with Amazon EFS.

---

## Dependency

Depends on:

- Assignment-VPC
- EC2-SG

---

## What happens if this step is skipped?

The EC2 instances will not be able to mount the Amazon EFS file system.

Mount commands will return connection errors.

---

# Step 6: Launch Ubuntu EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Launch Instance
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `Ubuntu-Server` |
| AMI | `Ubuntu Server 22.04 LTS` |
| Instance Type | `t2.micro` |
| Key Pair | Select your existing key pair or create a new one |
| Network | `Assignment-VPC` |
| Subnet | `Public-Subnet` |
| Auto Assign Public IP | `Enable` |
| Security Group | `EC2-SG` |
| Storage | Default (`8 GB`) |

Click **Launch Instance**.

---

## Why is this step required?

This is the first operating system required by the assignment.

It will later mount the shared Amazon EFS file system.

---

## Dependency

Depends on:

- Assignment-VPC
- Public-Subnet
- EC2-SG
- Key Pair

---

## What happens if this step is skipped?

The assignment requires three different operating systems.

Without Ubuntu, the assignment will be incomplete.

---

# Step 7: Launch Red Hat Enterprise Linux EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Launch Instance
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `RHEL-Server` |
| AMI | `Red Hat Enterprise Linux 9` |
| Instance Type | `t2.micro` |
| Key Pair | Same key pair |
| Network | `Assignment-VPC` |
| Subnet | `Public-Subnet` |
| Auto Assign Public IP | `Enable` |
| Security Group | `EC2-SG` |
| Storage | Default (`8 GB`) |

Click **Launch Instance**.

---

## Why is this step required?

The assignment specifically requires a Red Hat Linux operating system.

Using a different Linux distribution demonstrates that Amazon EFS works across multiple operating systems.

---

## Dependency

Depends on:

- Assignment-VPC
- Public-Subnet
- EC2-SG
- Key Pair

---

## What happens if this step is skipped?

The assignment requirement of using three different operating systems will not be satisfied.

---

# Step 8: Launch Amazon Linux 2 EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Launch Instance
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `AmazonLinux2-Server` |
| AMI | `Amazon Linux 2 AMI` |
| Instance Type | `t2.micro` |
| Key Pair | Same key pair |
| Network | `Assignment-VPC` |
| Subnet | `Public-Subnet` |
| Auto Assign Public IP | `Enable` |
| Security Group | `EC2-SG` |
| Storage | Default (`8 GB`) |

Click **Launch Instance**.

---

## Why is this step required?

Amazon Linux 2 is the third operating system required by the assignment.

It is an AWS-optimized Linux distribution and commonly used in production environments.

---

## Dependency

Depends on:

- Assignment-VPC
- Public-Subnet
- EC2-SG
- Key Pair

---

## What happens if this step is skipped?

The assignment will not meet the requirement of connecting Amazon EFS to three EC2 instances with different operating systems.

---

# Step 9: Verify All EC2 Instances Are Running

## Navigation

```text
AWS Console
→ EC2
→ Instances
```

Verify that all three instances display the following status:

| Instance Name | Status |
|--------------|--------|
| Ubuntu-Server | Running |
| RHEL-Server | Running |
| AmazonLinux2-Server | Running |

Also verify that each instance has:

- A Public IPv4 Address
- A Private IPv4 Address
- Passed both Status Checks

---

## Why is this step required?

Amazon EFS cannot be mounted unless the EC2 instances are successfully running.

Checking the instance status ensures the operating systems have booted correctly.

---

## Dependency

Depends on:

- Ubuntu-Server
- RHEL-Server
- AmazonLinux2-Server

---

## What happens if this step is skipped?

You may attempt to connect to an instance that is still starting or has failed to boot.

Subsequent SSH and Amazon EFS configuration steps will fail.

---

# Step 10: Connect to the Ubuntu EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ Ubuntu-Server
→ Connect
→ SSH Client
```

## Command

Change the permission of the key file:

```bash
chmod 400 Assignment-Key.pem
```

Connect to the Ubuntu instance:

```bash
ssh -i Assignment-Key.pem ubuntu@<Ubuntu-Public-IP>
```

Replace `<Ubuntu-Public-IP>` with the Public IPv4 address of your Ubuntu instance.

---

## Why is this step required?

SSH provides secure remote access to the EC2 instance so that software can be installed and Amazon EFS can be configured.

---

## Dependency

Depends on:

- Ubuntu-Server
- Key Pair

---

## What happens if this step is skipped?

You will not be able to configure or mount Amazon EFS on the Ubuntu instance.

---

# Step 11: Connect to the Red Hat EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ RHEL-Server
→ Connect
→ SSH Client
```

## Command

```bash
chmod 400 Assignment-Key.pem
```

```bash
ssh -i Assignment-Key.pem ec2-user@<RHEL-Public-IP>
```

Replace `<RHEL-Public-IP>` with the Public IPv4 address of the RHEL instance.

---

## Why is this step required?

SSH allows you to install the required NFS client packages and mount Amazon EFS.

---

## Dependency

Depends on:

- RHEL-Server
- Key Pair

---

## What happens if this step is skipped?

The Amazon EFS file system cannot be configured on the Red Hat instance.

---

# Step 12: Connect to the Amazon Linux 2 EC2 Instance

## Navigation

```text
AWS Console
→ EC2
→ Instances
→ AmazonLinux2-Server
→ Connect
→ SSH Client
```

## Command

```bash
chmod 400 Assignment-Key.pem
```

```bash
ssh -i Assignment-Key.pem ec2-user@<AmazonLinux2-Public-IP>
```

Replace `<AmazonLinux2-Public-IP>` with the Public IPv4 address of the Amazon Linux 2 instance.

---

## Why is this step required?

SSH access is required to prepare the operating system for mounting Amazon EFS.

---

## Dependency

Depends on:

- AmazonLinux2-Server
- Key Pair

---

## What happens if this step is skipped?

The Amazon Linux 2 instance will not be able to mount the shared Amazon EFS file system.

---
# Step 13: Create an Amazon EFS File System

## Navigation

```text
AWS Console
→ Amazon EFS
→ File Systems
→ Create File System
```

## Configuration

| Property | Value |
|----------|-------|
| Name | `Assignment-EFS` |
| VPC | `Assignment-VPC` |
| Encryption | `Enabled` (Recommended) |
| Automatic Backups | `Enabled` (Default) |
| Lifecycle Management | `Default` |
| Performance Mode | `General Purpose` |
| Throughput Mode | `Elastic` |

Click **Create**.

---

## Why is this step required?

Amazon Elastic File System (EFS) provides a fully managed, scalable network file system that can be mounted simultaneously by multiple EC2 instances.

Unlike Amazon EBS, which can typically be attached to only one EC2 instance at a time, Amazon EFS allows multiple EC2 instances to access the same files concurrently.

This shared storage is the main requirement of this assignment.

---

## Dependency

Depends on:

- Assignment-VPC

---

## What happens if this step is skipped?

There will be no shared file system available.

The EC2 instances will only have their individual EBS root volumes and will not be able to share files.

---

# Step 14: Create Mount Targets

After creating the file system, AWS prompts you to configure Mount Targets.

A Mount Target acts as a network endpoint that allows EC2 instances inside the VPC to communicate with Amazon EFS.

## Navigation

```text
AWS Console
→ Amazon EFS
→ File Systems
→ Assignment-EFS
→ Network
→ Manage
```

## Configuration

| Property | Value |
|----------|-------|
| Availability Zone | Same AZ as your EC2 instances |
| Subnet | `Public-Subnet` |
| Security Group | `EFS-SG` |

Save the configuration.

---

## Why is this step required?

Amazon EFS cannot be accessed directly.

Instead, EC2 instances connect to a Mount Target, which provides an IP address inside the VPC for accessing the shared file system.

Without a Mount Target, the file system exists but is unreachable.

---

## Dependency

Depends on:

- Assignment-EFS
- Public-Subnet
- EFS-SG

---

## What happens if this step is skipped?

The Amazon EFS file system cannot be mounted.

Mount commands will fail with timeout or connection errors because no network endpoint exists.

---

# Step 15: Obtain the Amazon EFS DNS Name

## Navigation

```text
AWS Console
→ Amazon EFS
→ File Systems
→ Assignment-EFS
```

Copy the **DNS Name** displayed for the file system.

It will look similar to:

```text
fs-1234567890abcdef0.efs.<region>.amazonaws.com
```

Replace `<region>` with your AWS Region (for example, `ap-south-1`).

---

## Why is this step required?

The DNS name uniquely identifies your Amazon EFS file system.

It is used in the mount command on each EC2 instance.

Using the DNS name ensures that AWS automatically resolves the correct Mount Target IP address.

---

## Dependency

Depends on:

- Assignment-EFS
- Mount Targets

---

## What happens if this step is skipped?

You will not know the address of the EFS file system.

Mount commands cannot be executed.

---

# Step 16: Install NFS Utilities on Ubuntu

Connect to the Ubuntu instance.

## Command

Update the package repository.

```bash
sudo apt update
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges. Updating the system package index is a system-level operation, so root permissions are required. |
| `apt` | **Advanced Package Tool**. Ubuntu's package manager used to install, update, remove, and manage software packages. |
| `update` | Downloads the latest package information (package lists) from the configured repositories and refreshes the local package index. It **does not** install or upgrade any software. |
Install the NFS client package.

```bash
sudo apt install -y nfs-common
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because installing software requires elevated permissions. |
| `apt` | **Advanced Package Tool**. Ubuntu's package manager used to install, update, remove, and manage software packages. |
| `install` | Tells `apt` to download and install the specified software package along with any required dependencies. |
| `-y` | Automatically answers **"Yes"** to all confirmation prompts, allowing the installation to proceed without requiring user input. |
| `nfs-common` | The Ubuntu package that installs the **NFS (Network File System) client utilities** required to connect to and mount Amazon EFS. |
---

## Why is this step required?

Ubuntu does not include the NFS client package by default.

The `nfs-common` package provides the utilities required to mount Amazon EFS.

---

## Dependency

Depends on:

- Ubuntu-Server
- Internet Connectivity
- Route Table
- Internet Gateway

---

## What happens if this step is skipped?

The `mount` command will fail because the operating system does not understand the NFS file system type.

---

# Step 17: Install NFS Utilities on Red Hat Enterprise Linux

Connect to the RHEL instance.

## Command

```bash
sudo dnf install -y nfs-utils
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because installing software requires elevated permissions. |
| `dnf` | **Dandified YUM**. The default package manager for Red Hat Enterprise Linux (RHEL) 8/9, used to install, update, remove, and manage software packages. |
| `install` | Tells `dnf` to download and install the specified software package along with any required dependencies. |
| `-y` | Automatically answers **"Yes"** to all confirmation prompts, allowing the installation to proceed without requiring user interaction. |
| `nfs-utils` | The RHEL package that installs the **NFS (Network File System) client and utility programs** required to connect to and mount Amazon EFS. |
---

## Why is this step required?

The `nfs-utils` package contains the client software required for communicating with NFS servers such as Amazon EFS.

---

## Dependency

Depends on:

- RHEL-Server
- Internet Connectivity

---

## What happens if this step is skipped?

The Amazon EFS file system cannot be mounted.

---

# Step 18: Install NFS Utilities on Amazon Linux 2

Connect to the Amazon Linux 2 instance.

## Command

```bash
sudo yum install -y nfs-utils
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because installing software requires elevated permissions. |
| `yum` | **Yellowdog Updater, Modified**. The package manager used by Amazon Linux 2 to install, update, remove, and manage software packages. |
| `install` | Tells `yum` to download and install the specified software package along with any required dependencies. |
| `-y` | Automatically answers **"Yes"** to all confirmation prompts, allowing the installation to proceed without requiring user interaction. |
| `nfs-utils` | The package that installs the **NFS (Network File System) client and utility programs** required to connect to and mount Amazon EFS. |
---

## Why is this step required?

Amazon Linux 2 also requires the NFS client package before it can communicate with Amazon EFS.

---

## Dependency

Depends on:

- AmazonLinux2-Server
- Internet Connectivity

---

## What happens if this step is skipped?

The mount command will fail because the NFS client software is not installed.

---

# Step 19: Create a Mount Directory on Ubuntu

## Command

```bash
sudo mkdir /efs
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because creating directories in the root (`/`) location requires elevated permissions. |
| `mkdir` | **Make Directory**. Creates a new directory (folder) in the specified location. |
| `/efs` | The absolute path of the directory to be created. This directory acts as the **mount point**, where the Amazon EFS file system will be attached and accessed. |
---

## Why is this step required?

Linux mounts file systems onto directories.

The `/efs` directory will act as the mount point where the Amazon EFS file system becomes accessible.

---

## Dependency

Depends on:

- Ubuntu-Server
- nfs-common package

---

## What happens if this step is skipped?

The mount command will fail because the destination directory does not exist.

---

# Step 20: Create a Mount Directory on Red Hat Enterprise Linux

## Command

```bash
sudo mkdir /efs
```

---

## Why is this step required?

A mount point must exist before any file system can be attached.

---

## Dependency

Depends on:

- RHEL-Server
- nfs-utils package

---

## What happens if this step is skipped?

Linux cannot mount the file system.

---

# Step 21: Create a Mount Directory on Amazon Linux 2

## Command

```bash
sudo mkdir /efs
```

---

## Why is this step required?

The `/efs` directory serves as the access point for the shared Amazon EFS storage.

---

## Dependency

Depends on:

- AmazonLinux2-Server
- nfs-utils package

---

## What happens if this step is skipped?

The mount operation cannot be completed because the mount point is missing.

---

# Step 22: Mount Amazon EFS on Ubuntu

## Command

Replace `<EFS-DNS-NAME>` with your actual Amazon EFS DNS name.

```bash
sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS-NAME>:/ /efs
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because mounting a file system is a system-level operation. |
| `mount` | Linux command used to attach a file system to a directory (mount point) so that its contents can be accessed. |
| `-t` | Specifies the **type of file system** being mounted. |
| `nfs4` | Indicates that the file system type is **Network File System version 4 (NFSv4)**. Amazon EFS supports NFS version 4.1. |
| `-o` | Specifies additional **mount options** that control how the file system is mounted. |
| `nfsvers=4.1` | Instructs the mount command to use **NFS version 4.1**, which is the supported protocol version for Amazon EFS. |
| `<EFS-DNS-NAME>:/` | The DNS name of your Amazon EFS file system followed by `:/`, which represents the **root directory** of the EFS file system. Example: `fs-1234567890abcdef0.efs.ap-south-1.amazonaws.com:/` |
| `/efs` | The local **mount point** on the EC2 instance where the Amazon EFS file system will be attached and accessed. |

Example:

```bash
sudo mount -t nfs4 -o nfsvers=4.1 fs-1234567890abcdef0.efs.ap-south-1.amazonaws.com:/ /efs
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because mounting a file system is a system-level operation. |
| `mount` | Linux command used to attach a file system to a directory (mount point) so that its contents can be accessed. |
| `-t` | Specifies the **type of file system** being mounted. |
| `nfs4` | Indicates that the file system type is **Network File System version 4 (NFSv4)**. Amazon EFS uses the NFSv4.1 protocol. |
| `-o` | Specifies additional **mount options** that control how the file system should be mounted. |
| `nfsvers=4.1` | Instructs the `mount` command to use **NFS version 4.1**, which is the supported protocol version for Amazon EFS. |
| `fs-1234567890abcdef0` | The unique **File System ID** assigned to your Amazon EFS file system by AWS. Every EFS file system has a unique ID. |
| `.efs` | Indicates that the endpoint belongs to the **Amazon Elastic File System (EFS)** service. |
| `ap-south-1` | The **AWS Region** where the EFS file system is created. In this example, it is the **Asia Pacific (Mumbai)** Region. |
| `amazonaws.com` | The official AWS domain used to access AWS services. |
| `:/` | Refers to the **root directory** of the Amazon EFS file system that will be mounted. |
| `/efs` | The local **mount point** (directory) on the EC2 instance where the Amazon EFS file system will be attached and accessed. |

---

## Why is this step required?

This command connects the Ubuntu EC2 instance to the shared Amazon EFS file system.

After mounting, any files created inside `/efs` are stored on Amazon EFS instead of the local EBS volume.

---

## Dependency

Depends on:

- Assignment-EFS
- Mount Targets
- Ubuntu NFS Client
- EFS-SG
- EC2-SG

---

## What happens if this step is skipped?

Ubuntu will not have access to the shared file system.

---

# Step 23: Mount Amazon EFS on Red Hat Enterprise Linux

## Command

```bash
sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS-NAME>:/ /efs
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because mounting a file system is a system-level operation. |
| `mount` | Linux command used to attach a file system to a directory (mount point) so that its contents can be accessed. |
| `-t` | Specifies the **type of file system** being mounted. |
| `nfs4` | Indicates that the file system type is **Network File System version 4 (NFSv4)**. Amazon EFS uses the NFSv4.1 protocol. |
| `-o` | Specifies additional **mount options** that control how the file system should be mounted. |
| `nfsvers=4.1` | Instructs the `mount` command to use **NFS version 4.1**, which is the supported protocol version for Amazon EFS. |
| `<EFS-DNS-NAME>` | A placeholder for the **DNS name of your Amazon EFS file system**. Replace it with the actual DNS name provided by AWS, for example: `fs-1234567890abcdef0.efs.ap-south-1.amazonaws.com`. |
| `:/` | Refers to the **root directory** of the Amazon EFS file system that will be mounted. |
| `/efs` | The local **mount point** (directory) on the EC2 instance where the Amazon EFS file system will be attached and accessed. |

---

## Why is this step required?

This allows the Red Hat EC2 instance to access the same shared Amazon EFS storage used by the Ubuntu instance.

---

## Dependency

Depends on:

- Assignment-EFS
- Mount Targets
- nfs-utils
- Security Groups

---

## What happens if this step is skipped?

The Red Hat instance will not participate in the shared storage environment.

---

# Step 24: Mount Amazon EFS on Amazon Linux 2

## Command

```bash
sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS-NAME>:/ /efs
```

---

## Why is this step required?

This mounts the same Amazon EFS file system on the Amazon Linux 2 instance, allowing all three EC2 instances to share the same storage.

---

## Dependency

Depends on:

- Assignment-EFS
- Mount Targets
- nfs-utils
- Security Groups

---

## What happens if this step is skipped?

The Amazon Linux 2 instance will not have access to the shared file system.

---

# Step 25: Configure Automatic Mounting on Ubuntu

## Navigation

Not applicable. This step is performed using the terminal.

## Commands

Open the `/etc/fstab` file.

```bash
sudo nano /etc/fstab
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because the `/etc/fstab` system configuration file can only be modified by the root user. |
| `nano` | A simple, terminal-based **text editor** used to create and edit text files directly from the command line. |
| `/etc/fstab` | The **File Systems Table (fstab)** configuration file. It stores information about file systems that should be automatically mounted during system boot, including their mount points and mount options. |

---

Add the following line at the end of the file.

Replace `<EFS-DNS-NAME>` with your actual Amazon EFS DNS name.

```text
<EFS-DNS-NAME>:/ /efs nfs4 defaults,_netdev 0 0
```

### Configuration Breakdown

| Configuration Part | Explanation |
|--------------------|-------------|
| `<EFS-DNS-NAME>:/` | A placeholder for the **DNS name of your Amazon EFS file system** followed by `:/`, which represents the **root directory** of the EFS file system to be mounted. |
| `/efs` | The local **mount point** (directory) on the EC2 instance where the Amazon EFS file system will be mounted and accessed. |
| `nfs4` | Specifies that the file system type is **Network File System version 4 (NFSv4)**, which is the protocol used by Amazon EFS. |
| `defaults` | Applies the default mount options, including `rw` (read/write), `suid`, `dev`, `exec`, `auto`, `nouser`, and `async`, which are suitable for most file systems. |
| `_netdev` | Indicates that this is a **network-based file system**. During system boot, Linux waits until the network is available before attempting to mount the file system, preventing mount failures. |
| `0` (5th field) | Controls the **dump** backup utility. A value of `0` means the file system will **not** be included in dump backups. |
| `0` (6th field) | Controls the **fsck (File System Check)** order during boot. A value of `0` means Linux will **not** perform a filesystem consistency check on this network file system during startup. |

---

Save and exit the editor.

To verify the configuration without rebooting, run:

```bash
sudo mount -a
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because mounting file systems is a system-level operation. |
| `mount` | Linux command used to attach one or more file systems to their respective mount points. |
| `-a` | **All**. Tells the `mount` command to mount **all file systems** listed in the `/etc/fstab` file that are not currently mounted (except those marked with the `noauto` option). This is commonly used to verify that new `/etc/fstab` entries are correct without rebooting the system. |

---

## Why is this step required?

The `/etc/fstab` file stores file systems that should be mounted automatically during system boot.

Adding the Amazon EFS entry ensures that the shared file system is automatically mounted whenever the EC2 instance restarts.

The `_netdev` option tells Linux to wait until the network is available before attempting to mount the file system.

---

## Dependency

Depends on:

- Ubuntu instance
- Amazon EFS mounted successfully

---

## What happens if this step is skipped?

The EFS file system will work only until the instance is rebooted.

After a restart, you would need to mount it manually again.

---

# Step 26: Configure Automatic Mounting on Red Hat Enterprise Linux

## Navigation

Not applicable.

## Commands

```bash
sudo nano /etc/fstab
```

Add:

```text
<EFS-DNS-NAME>:/ /efs nfs4 defaults,_netdev 0 0
```

Verify:

```bash
sudo mount -a
```

---

## Why is this step required?

This ensures that the Red Hat EC2 instance automatically reconnects to Amazon EFS after every reboot.

---

## Dependency

Depends on:

- Amazon EFS mounted successfully

---

## What happens if this step is skipped?

The shared storage will disappear after a reboot until it is mounted manually.

---

# Step 27: Configure Automatic Mounting on Amazon Linux 2

## Navigation

Not applicable.

## Commands

```bash
sudo nano /etc/fstab
```

Add:

```text
<EFS-DNS-NAME>:/ /efs nfs4 defaults,_netdev 0 0
```

Verify:

```bash
sudo mount -a
```

---

## Why is this step required?

This guarantees that Amazon Linux 2 automatically mounts the EFS file system after every restart.

---

## Dependency

Depends on:

- Amazon EFS mounted successfully

---

## What happens if this step is skipped?

The mount will be lost whenever the EC2 instance restarts.

---

# Verification Step 1: Verify That Amazon EFS Is Mounted

## Action

Run the following command on **all three EC2 instances**.

## Command

```bash
df -h
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `df` | **Disk Free**. Displays information about mounted file systems, including their total size, used space, available space, and mount points. |
| `-h` | **Human-readable**. Displays disk sizes in an easy-to-read format such as **KB**, **MB**, **GB**, or **TB** instead of showing sizes only in bytes. |

----

## Expected Output

The output should contain a line similar to the following:

```text
Filesystem                                           Size  Used Avail Use% Mounted on
fs-1234567890abcdef0.efs.ap-south-1.amazonaws.com:/   8.0E     0  8.0E   0% /efs
```

> The reported size may appear extremely large because Amazon EFS is an elastic file system that automatically grows and shrinks based on the amount of data stored.

---

## Why does this confirm success?

If the EFS DNS name appears in the output and is mounted on `/efs`, the operating system has successfully connected to the Amazon EFS file system.

---

# Verification Step 2: Create a File from the Ubuntu Instance

## Action

Create a file inside the mounted EFS directory.

## Command

```bash
echo "Created from Ubuntu" | sudo tee /efs/ubuntu.txt
```
### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `echo` | Displays the specified text on the terminal's standard output (stdout). |
| `"Created from Ubuntu"` | The text string that `echo` outputs. In this assignment, it is the content that will be written to the file. |
| `\|` | **Pipe operator**. Sends the output of the command on the left (`echo`) as the input to the command on the right (`tee`). |
| `sudo` | **SuperUser DO**. Executes the `tee` command with administrator (root) privileges, ensuring it has permission to write to the mounted EFS directory if required. |
| `tee` | Reads input from standard input (stdin) and writes it to the specified file while also displaying the same content on the terminal. |
| `/efs/ubuntu.txt` | The file that will be created (or overwritten if it already exists) inside the mounted Amazon EFS file system. Since `/efs` is the EFS mount point, the file is stored on the shared EFS storage and becomes accessible from all connected EC2 instances. |

---

## Expected Output

```text
Created from Ubuntu
```

---

## Why does this confirm success?

The file is written to the Amazon EFS shared storage rather than the local EBS volume.

---

# Verification Step 3: Verify the File on the Red Hat Instance

## Action

List the contents of the mounted directory.

## Command

```bash
ls -l /efs
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `ls` | **List**. Displays the files and directories present in the specified location. |
| `-l` | **Long listing format**. Shows detailed information for each file or directory, including file permissions, number of links, owner, group, file size, last modification date and time, and file name. |
| `/efs` | The directory being listed. In this assignment, `/efs` is the **mount point** where the Amazon EFS file system is mounted, so the command displays all files and folders stored in the shared EFS file system. |

---


## Expected Output

```text
ubuntu.txt
```

Display the contents of the file.

```bash
cat /efs/ubuntu.txt
```

Expected output:

```text
Created from Ubuntu
```

---

## Why does this confirm success?

If the Red Hat instance can view the file created on Ubuntu, both instances are using the same shared Amazon EFS file system.

---

# Verification Step 4: Verify the File on the Amazon Linux 2 Instance

## Action

Run the following commands.

## Command

```bash
ls -l /efs
```

```bash
cat /efs/ubuntu.txt
```

## Expected Output

```text
Created from Ubuntu
```

---

## Why does this confirm success?

The same file is visible from all three EC2 instances, confirming that Amazon EFS provides shared storage across different operating systems.

---

# Verification Step 5: Create a File from the Amazon Linux 2 Instance

## Action

Create another file.

## Command

```bash
echo "Created from Amazon Linux 2" | sudo tee /efs/amazonlinux2.txt
```

Verify the file from the Ubuntu and Red Hat instances.

```bash
ls -l /efs
```

Expected output:

```text
amazonlinux2.txt
ubuntu.txt
```

---

## Why does this confirm success?

This demonstrates two-way file sharing across all EC2 instances through the same Amazon EFS file system.

---

# Cleanup Step 1: Remove the `/etc/fstab` Entry

## Navigation

Not applicable.

## Action

Open the `/etc/fstab` file on each EC2 instance and remove the Amazon EFS entry that was added.

Save the file.

---

## Why is this required?

Removing the entry prevents the operating system from attempting to mount a file system that will no longer exist.

---

## What happens if this resource is not deleted?

Future reboots may generate mount errors because the EFS file system has already been deleted.

---

# Cleanup Step 2: Unmount Amazon EFS

## Navigation

Not applicable.

## Action

Run the following command on all three EC2 instances.

## Command

```bash
sudo umount /efs
```

### Command Breakdown

| Command Part | Explanation |
|-------------|-------------|
| `sudo` | **SuperUser DO**. Executes the command with administrator (root) privileges because unmounting a file system is a system-level operation. |
| `umount` | **Unmount**. Detaches a mounted file system from its mount point, making it no longer accessible through that directory. *(The command is spelled `umount`, not `unmount`.)* |
| `/efs` | The **mount point** from which the Amazon EFS file system will be detached. After unmounting, the `/efs` directory remains on the EC2 instance, but it will no longer provide access to the Amazon EFS file system. |

---

## Why is this required?

Unmounting disconnects the EC2 instance from the shared file system before deletion.

---

## What happens if this resource is not deleted?

Although AWS can still delete the file system, leaving mounted file systems may cause errors or stale mount references on the instances.

---

# Cleanup Step 3: Delete the Amazon EFS File System

## Navigation

```text
AWS Console
→ Amazon EFS
→ File Systems
→ Assignment-EFS
→ Delete
```

## Action

Delete the file system.

AWS automatically removes the associated Mount Targets.

> If the console requires you to delete Mount Targets first, delete each Mount Target and then delete the EFS file system.

---

## Why is this required?

Amazon EFS is a paid AWS service.

Deleting it immediately helps prevent unnecessary storage charges.

---

## What happens if this resource is not deleted?

Amazon EFS storage charges continue until the file system is deleted.

---

# Cleanup Step 4: Terminate All EC2 Instances

## Navigation

```text
AWS Console
→ EC2
→ Instances
```

## Action

Select:

- Ubuntu-Server
- RHEL-Server
- AmazonLinux2-Server

Choose:

```text
Instance State
→ Terminate Instance
```

---

## Why is this required?

Terminating the EC2 instances stops compute charges and releases the associated resources.

---

## What happens if this resource is not deleted?

The instances continue to run and may consume Free Tier hours or AWS promotional credits.

---

# Cleanup Step 5: Delete Security Groups

## Navigation

```text
AWS Console
→ VPC
→ Security Groups
```

## Action

Delete:

- `EC2-SG`
- `EFS-SG`

---

## Why is this required?

These security groups were created specifically for this assignment and are no longer needed.

---

## What happens if this resource is not deleted?

Unused security groups do not incur charges but can clutter your AWS environment and make management more difficult.

---

# Cleanup Step 6: Delete Networking Resources (If Created for This Assignment)

Perform these actions only if you created a dedicated VPC for this assignment.

## Navigation

```text
AWS Console
→ VPC
```

## Action

Delete the resources in the following order:

1. Route Table (if not the main route table)
2. Detach the Internet Gateway from the VPC
3. Delete the Internet Gateway
4. Delete the Subnet
5. Delete the VPC

---

## Why is this required?

AWS networking resources have dependencies.

For example, a VPC cannot be deleted while it still contains subnets or has an attached Internet Gateway.

Deleting resources in the correct order avoids dependency errors.

---

## What happens if this resource is not deleted?

Although these networking resources do not usually incur charges, leaving unused resources can clutter your AWS account and make future networking configurations more difficult.
