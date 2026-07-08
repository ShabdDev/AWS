# Module 5 - Virtual Private Cloud (VPC)

# Case Study – VPC and VPC Peering

## Problem Statement

You work for XYZ Corporation and based on the expansion requirements of your corporation you have been asked to create and set up a distinct Amazon VPC for the Production and Development teams.

### Production Network Requirements

1. Design and build a 4-tier architecture.
2. Create 5 subnets:
   - `web` (Public)
   - `app1` (Private)
   - `app2` (Private)
   - `dbcache` (Private)
   - `db` (Private)
3. Launch one EC2 instance in every subnet.
4. Allow:
   - `dbcache` instance to access the Internet.
   - `app1` subnet to access the Internet.
5. Configure Security Groups and Network ACLs.

### Development Network Requirements

1. Design and build a 2-tier architecture.
2. Create two subnets:
   - `web`
   - `db`
3. Launch one EC2 instance in each subnet.
4. Only the Development Web subnet should have Internet access.
5. Create VPC Peering between Production and Development VPCs.
6. Enable communication between Production DB subnet and Development DB subnet.

---

# AWS Free Tier / Cost Check

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
|-------------|-------------------|--------------|------|
| Amazon VPC | Yes | No | No additional charge for creating VPCs. |
| Subnets | Yes | No | Free. |
| Route Tables | Yes | No | Free. |
| Internet Gateway | Yes | No | Free. |
| NAT Gateway | **No** | **Yes** | Hourly charge plus data processing charges. |
| Elastic IP (attached to running instance) | Yes | No | One attached Elastic IP is free under Free Tier limits. |
| Security Groups | Yes | No | Free. |
| Network ACLs | Yes | No | Free. |
| VPC Peering | Yes | No | No charge for creating the peering connection. Data transfer charges may apply. |
| EC2 t2.micro / t3.micro Instances | Yes | No (within Free Tier limits) | Eligible under AWS Free Tier monthly limits. |
| Elastic Network Interfaces | Yes | No | Free. |

## Free Tier Eligibility

This case study is **not completely Free Tier eligible** because the Production network requires private subnets (`app1` and `dbcache`) to send Internet requests.

Private subnets require a **NAT Gateway** to access the Internet.

Amazon NAT Gateway is **not included in the AWS Free Tier** and therefore consumes AWS promotional credits or incurs charges.

## Recommendation

It is recommended to perform this case study while you still have AWS promotional credits because:

- NAT Gateway incurs hourly charges.
- NAT Gateway also incurs data processing charges.
- Everything else in this assignment is either free or covered under the AWS Free Tier (assuming Free Tier eligible EC2 instances are used).

---

# Target Architecture

```text
                         INTERNET
                             |
                     Internet Gateway
                             |
                +---------------------------+
                |     Production VPC        |
                |      10.0.0.0/16          |
                +---------------------------+

          Public Subnet (web)
                |
             Web EC2
                |
          NAT Gateway
                |
-------------------------------------------------------------

Private Subnet (app1) -------- App1 EC2
        |
Private Subnet (app2) -------- App2 EC2
        |
Private Subnet (dbcache) ----- DBCache EC2
        |
Private Subnet (db) ---------- DB EC2


                    ||
                    ||  VPC Peering
                    ||

+-----------------------------------------------+
|           Development VPC                     |
|             10.1.0.0/16                       |
+-----------------------------------------------+

Public Web Subnet -------- Web EC2

Private DB Subnet -------- DB EC2

```

---

# IP Address Plan

## Production VPC

| Resource | CIDR |
|-----------|------|
| VPC | `10.0.0.0/16` |
| Web | `10.0.1.0/24` |
| App1 | `10.0.2.0/24` |
| App2 | `10.0.3.0/24` |
| DBCache | `10.0.4.0/24` |
| DB | `10.0.5.0/24` |

---

## Development VPC

| Resource | CIDR |
|-----------|------|
| VPC | `10.1.0.0/16` |
| Web | `10.1.1.0/24` |
| DB | `10.1.2.0/24` |

---

# Resource Naming Convention

| Resource | Name |
|-----------|------|
| Production VPC | `Prod-VPC` |
| Development VPC | `Dev-VPC` |
| Production Internet Gateway | `Prod-IGW` |
| Development Internet Gateway | `Dev-IGW` |
| Production NAT Gateway | `Prod-NATGW` |
| Production Public Route Table | `Prod-Public-RT` |
| Production Private Route Table | `Prod-Private-RT` |
| Development Public Route Table | `Dev-Public-RT` |
| Development Private Route Table | `Dev-Private-RT` |
| Production Web EC2 | `web` |
| Production App1 EC2 | `app1` |
| Production App2 EC2 | `app2` |
| Production DBCache EC2 | `dbcache` |
| Production DB EC2 | `db` |
| Development Web EC2 | `web` |
| Development DB EC2 | `db` |

---

# Step 1: Create the Production VPC

## Navigation

```text
AWS Console

→ VPC

→ Your VPCs

→ Create VPC
```

## Configuration

| Setting | Value |
|---------|------|
| Resources to Create | `VPC Only` |
| Name | `Prod-VPC` |
| IPv4 CIDR | `10.0.0.0/16` |
| IPv6 | `No IPv6 CIDR Block` |
| Tenancy | `Default` |

Click **Create VPC**.

## Why is this step required?

A Virtual Private Cloud (VPC) is an isolated virtual network inside AWS.

All networking resources such as subnets, route tables, gateways, and EC2 instances must exist inside a VPC.

The Production environment requires its own isolated network.

## Dependency

None.

This is the first networking resource.

## What happens if this step is skipped?

No subnets, Internet Gateway, NAT Gateway, Route Tables, Security Groups, or EC2 instances can be created for the Production environment because all these resources require a VPC.

---

# Step 2: Create the Development VPC

## Navigation

```text
AWS Console

→ VPC

→ Your VPCs

→ Create VPC
```

## Configuration

| Setting | Value |
|---------|------|
| Resources to Create | `VPC Only` |
| Name | `Dev-VPC` |
| IPv4 CIDR | `10.1.0.0/16` |
| IPv6 | `No IPv6 CIDR Block` |
| Tenancy | `Default` |

Click **Create VPC**.

## Why is this step required?

The Development environment should be isolated from the Production environment.

Using a separate VPC provides network isolation, improves security, and allows independent management of resources.

## Dependency

None.

## What happens if this step is skipped?

Development resources cannot be created because subnets and EC2 instances require a VPC.

---

# Step 3: Create the Production Subnets

## Navigation

```text
AWS Console

→ VPC

→ Subnets

→ Create Subnet
```

## Configuration

Create the following five subnets.

| Name | CIDR | Type |
|-------|------|------|
| `web` | `10.0.1.0/24` | Public |
| `app1` | `10.0.2.0/24` | Private |
| `app2` | `10.0.3.0/24` | Private |
| `dbcache` | `10.0.4.0/24` | Private |
| `db` | `10.0.5.0/24` | Private |

Choose the `Prod-VPC` while creating every subnet.

---

## Why is this step required?

Subnets divide a VPC into smaller logical networks.

In this architecture:

- The `web` subnet hosts Internet-facing resources.
- The `app1` and `app2` subnets host application servers.
- The `dbcache` subnet hosts caching servers.
- The `db` subnet hosts database servers.

Keeping each tier in a separate subnet improves security, availability, and network management.

## Dependency

- Step 1: Production VPC must already exist.

## What happens if this step is skipped?

EC2 instances cannot be launched because every EC2 instance must belong to a subnet.

The required 4-tier architecture also cannot be implemented.

---

# Step 4: Create the Development Subnets

## Navigation

```text
AWS Console

→ VPC

→ Subnets

→ Create Subnet
```

## Configuration

Select **`Dev-VPC`** and create the following subnets.

| Name | CIDR | Type |
|------|------|------|
| `web` | `10.1.1.0/24` | Public |
| `db` | `10.1.2.0/24` | Private |

Click **Create Subnet**.

## Why is this step required?

The Development environment requires only two tiers:

- Web Tier
- Database Tier

Each subnet provides network isolation between application layers.

## Dependency

- Step 2: Development VPC

## What happens if this step is skipped?

Development EC2 instances cannot be launched.

---

# Step 5: Create the Production Internet Gateway

## Navigation

```text
AWS Console

→ VPC

→ Internet Gateways

→ Create Internet Gateway
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-IGW` |

Click **Create Internet Gateway**.

After creation:

- Select the Internet Gateway.
- Choose **Actions → Attach to VPC**.
- Select **`Prod-VPC`**.
- Click **Attach Internet Gateway**.

## Why is this step required?

An Internet Gateway (IGW) allows communication between resources inside the VPC and the public Internet.

Without an Internet Gateway, even public subnets cannot communicate with the Internet.

## Dependency

- Production VPC must already exist.

## What happens if this step is skipped?

Public subnet instances cannot:

- Download software
- Receive Internet traffic
- Access public websites
- Be reached using their public IP

---

# Step 6: Create the Development Internet Gateway

## Navigation

```text
AWS Console

→ VPC

→ Internet Gateways

→ Create Internet Gateway
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Dev-IGW` |

Create the Internet Gateway.

Attach it to **`Dev-VPC`**.

## Why is this step required?

The Development Web subnet also needs Internet connectivity.

The Internet Gateway enables inbound and outbound Internet communication.

## Dependency

- Development VPC

## What happens if this step is skipped?

The Development Web EC2 instance cannot access the Internet.

---

# Step 7: Allocate an Elastic IP for the NAT Gateway

## Navigation

```text
AWS Console

→ EC2

→ Elastic IPs

→ Allocate Elastic IP Address
```

## Configuration

Accept the default options.

Click **Allocate**.

Rename the Elastic IP:

| Name | Value |
|------|------|
| Name | `Prod-NAT-EIP` |

## Why is this step required?

A NAT Gateway requires a public Elastic IP address.

This public IP is used by the NAT Gateway to communicate with the Internet on behalf of private subnet instances.

## Dependency

- None

## What happens if this step is skipped?

The NAT Gateway cannot be created.

Private subnets will not have Internet access.

---

# Step 8: Create the Production NAT Gateway

## Navigation

```text
AWS Console

→ VPC

→ NAT Gateways

→ Create NAT Gateway
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-NATGW` |
| Subnet | `web` Public Subnet |
| Connectivity Type | `Public` |
| Elastic IP | `Prod-NAT-EIP` |

Click **Create NAT Gateway**.

Wait until the status changes to **Available**.

## Why is this step required?

Private subnets cannot directly communicate with the Internet.

The NAT Gateway allows outbound Internet access while preventing inbound Internet connections.

This satisfies the assignment requirement that:

- `app1`
- `dbcache`

must be able to send Internet requests.

## Dependency

- Production Public Subnet
- Production Internet Gateway
- Elastic IP

## What happens if this step is skipped?

Private subnet instances cannot:

- Install software
- Run `yum update`
- Download packages
- Access AWS repositories
- Reach external APIs

---

# Step 9: Create the Production Public Route Table

## Navigation

```text
AWS Console

→ VPC

→ Route Tables

→ Create Route Table
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-Public-RT` |
| VPC | `Prod-VPC` |

Click **Create Route Table**.

### Add Route

Choose **Routes → Edit Routes → Add Route**

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Prod-IGW` |

Save the route.

### Associate Subnet

Choose **Subnet Associations**

Associate only:

- `web`

## Why is this step required?

The Public Route Table directs Internet traffic from the public subnet to the Internet Gateway.

Only the Web subnet should use this route table.

## Dependency

- Internet Gateway
- Production Web subnet

## What happens if this step is skipped?

The Web EC2 instance will not have Internet connectivity.

---

# Step 10: Create the Production Private Route Table

## Navigation

```text
AWS Console

→ VPC

→ Route Tables

→ Create Route Table
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-Private-RT` |
| VPC | `Prod-VPC` |

Click **Create Route Table**.

### Add Route

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Prod-NATGW` |

Save.

### Associate Subnets

Associate:

- `app1`
- `app2`
- `dbcache`
- `db`

## Why is this step required?

Private subnet traffic destined for the Internet is forwarded to the NAT Gateway.

Instances remain private because they still do not have public IP addresses.

## Dependency

- NAT Gateway
- Private Subnets

## What happens if this step is skipped?

Private subnet instances will remain completely isolated and cannot access the Internet.

Although the assignment only explicitly requires Internet access for `app1` and `dbcache`, using one private route table with a NAT Gateway is a common AWS design for all private subnets. Internet access can still be further controlled using Security Groups or Network ACLs if stricter restrictions are required.

---

# Step 11: Create the Development Public Route Table

## Navigation

```text
AWS Console

→ VPC

→ Route Tables

→ Create Route Table
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Dev-Public-RT` |
| VPC | `Dev-VPC` |

Add the following route.

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Dev-IGW` |

Associate only:

- `web`

## Why is this step required?

The Development Web subnet is the only subnet that should communicate with the Internet.

This route table enables that connectivity.

## Dependency

- Development Internet Gateway
- Development Web Subnet

## What happens if this step is skipped?

The Development Web EC2 instance cannot access the Internet.

---

# Step 12: Create the Development Private Route Table

## Navigation

```text
AWS Console

→ VPC

→ Route Tables

→ Create Route Table
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Dev-Private-RT` |
| VPC | `Dev-VPC` |

Do **not** add an Internet route (`0.0.0.0/0`).

Associate only the following subnet:

- `db`

## Why is this step required?

The Development Database subnet is intended to remain private.

Since it does not require Internet access, it should not have a route to the Internet Gateway.

## Dependency

- Development VPC
- Development DB subnet

## What happens if this step is skipped?

The DB subnet will use the VPC's main route table, which may not provide the intended network isolation.

---

# Step 13: Create the Production Security Group

## Navigation

```text
AWS Console

→ EC2

→ Security Groups

→ Create Security Group
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-SG` |
| VPC | `Prod-VPC` |
| Description | `Security Group for Production EC2 Instances` |

### Inbound Rules

| Type | Source |
|------|--------|
| SSH (22) | `My IP` |

### Outbound Rules

Keep the default rule:

| Type | Destination |
|------|-------------|
| All Traffic | `0.0.0.0/0` |

Click **Create Security Group**.

## Why is this step required?

Security Groups act as virtual firewalls for EC2 instances.

This configuration allows SSH access only from your current public IP while allowing outbound traffic required for updates and package downloads.

## Dependency

- Production VPC

## What happens if this step is skipped?

EC2 instances will either use the default Security Group or another group, which may not meet the assignment's security requirements.

---

# Step 14: Create the Development Security Group

## Navigation

```text
AWS Console

→ EC2

→ Security Groups

→ Create Security Group
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Dev-SG` |
| VPC | `Dev-VPC` |
| Description | `Security Group for Development EC2 Instances` |

### Inbound Rules

| Type | Source |
|------|--------|
| SSH (22) | `My IP` |

### Outbound Rules

Leave the default outbound rule:

| Type | Destination |
|------|-------------|
| All Traffic | `0.0.0.0/0` |

Click **Create Security Group**.

## Why is this step required?

Development EC2 instances also require controlled administrative access through SSH.

## Dependency

- Development VPC

## What happens if this step is skipped?

SSH access may not be available, making instance administration difficult.

---

# Step 15: Configure Network ACLs

## Navigation

```text
AWS Console

→ VPC

→ Network ACLs
```

## Configuration

Create two custom Network ACLs.

### Production NACL

Associate with:

- `web`
- `app1`
- `app2`
- `dbcache`
- `db`

### Development NACL

Associate with:

- Development `web`
- Development `db`

### Inbound Rules

| Rule | Protocol | Port | Source | Allow/Deny |
|------|----------|------|--------|------------|
| 100 | All Traffic | All | `0.0.0.0/0` | Allow |

### Outbound Rules

| Rule | Protocol | Port | Destination | Allow/Deny |
|------|----------|------|-------------|------------|
| 100 | All Traffic | All | `0.0.0.0/0` | Allow |

## Why is this step required?

Network ACLs provide subnet-level security.

Unlike Security Groups, Network ACLs evaluate both inbound and outbound traffic at the subnet level.

Using custom NACLs prepares the environment for future security enhancements.

## Dependency

- Subnets must already exist.

## What happens if this step is skipped?

The default Network ACL will still allow traffic, but the assignment specifically requires managing Network ACLs.

---

# Step 16: Launch the Production Web EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Instances

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `web` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Key Pair | Your existing key pair |
| Network | `Prod-VPC` |
| Subnet | `web` |
| Auto Assign Public IP | `Enable` |
| Security Group | `Prod-SG` |

Click **Launch Instance**.

## Why is this step required?

This EC2 instance represents the public-facing Web tier of the Production environment.

It requires a public IP address so it can communicate with the Internet.

## Dependency

- Production Web subnet
- Security Group

## What happens if this step is skipped?

The Production architecture will be incomplete because the Web tier will not exist.

---

# Step 17: Launch the Production App1 EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `app1` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Network | `Prod-VPC` |
| Subnet | `app1` |
| Auto Assign Public IP | `Disable` |
| Security Group | `Prod-SG` |

Launch the instance.

## Why is this step required?

This instance represents the first application server tier.

It resides in a private subnet and accesses the Internet only through the NAT Gateway.

## Dependency

- App1 subnet
- NAT Gateway
- Security Group

## What happens if this step is skipped?

The Production Application tier will not be complete.

---

# Step 18: Launch the Production App2 EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `app2` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Network | `Prod-VPC` |
| Subnet | `app2` |
| Auto Assign Public IP | `Disable` |
| Security Group | `Prod-SG` |

Launch the instance.

## Why is this step required?

This instance represents the second application layer in the 4-tier architecture.

## Dependency

- App2 subnet
- Security Group

## What happens if this step is skipped?

The required architecture is incomplete because one application tier is missing.

---

# Step 19: Launch the Production DBCache EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `dbcache` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Network | `Prod-VPC` |
| Subnet | `dbcache` |
| Auto Assign Public IP | `Disable` |
| Security Group | `Prod-SG` |

Launch the instance.

## Why is this step required?

This instance represents the cache layer.

According to the assignment, this instance must be able to send Internet requests using the NAT Gateway.

## Dependency

- DBCache subnet
- NAT Gateway
- Security Group

## What happens if this step is skipped?

The cache tier required by the assignment will be missing.

---

# Step 20: Launch the Production DB EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `db` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Network | `Prod-VPC` |
| Subnet | `db` |
| Auto Assign Public IP | `Disable` |
| Security Group | `Prod-SG` |

Launch the instance.

## Why is this step required?

This instance represents the database layer of the Production environment.

It remains in a private subnet for improved security.

## Dependency

- DB subnet
- Security Group

## What happens if this step is skipped?

The Production architecture will not satisfy the required four-tier design.

---

# Step 21: Launch the Development Web EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Instances

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `web` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Key Pair | Your existing key pair |
| Network | `Dev-VPC` |
| Subnet | `web` |
| Auto Assign Public IP | `Enable` |
| Security Group | `Dev-SG` |

Click **Launch Instance**.

## Why is this step required?

This instance represents the Web tier of the Development environment. Since it resides in a public subnet with a route to the Internet Gateway, it can access the Internet and be reached using its public IP (subject to Security Group rules).

## Dependency

- Development Web subnet
- Development Security Group
- Development Internet Gateway
- Development Public Route Table

## What happens if this step is skipped?

The Development environment will not have a Web tier, and the required 2-tier architecture will be incomplete.

---

# Step 22: Launch the Development DB EC2 Instance

## Navigation

```text
AWS Console

→ EC2

→ Instances

→ Launch Instance
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `db` |
| AMI | `Amazon Linux 2023` |
| Instance Type | `t2.micro` |
| Key Pair | Your existing key pair |
| Network | `Dev-VPC` |
| Subnet | `db` |
| Auto Assign Public IP | `Disable` |
| Security Group | `Dev-SG` |

Click **Launch Instance**.

## Why is this step required?

This instance represents the Database tier of the Development environment. Keeping it in a private subnet improves security by preventing direct Internet access.

## Dependency

- Development DB subnet
- Development Security Group

## What happens if this step is skipped?

The Development architecture will not satisfy the assignment requirement of having separate Web and DB tiers.

---

# Step 23: Create the VPC Peering Connection

## Navigation

```text
AWS Console

→ VPC

→ Peering Connections

→ Create Peering Connection
```

## Configuration

| Setting | Value |
|---------|------|
| Name | `Prod-Dev-Peering` |
| Requester VPC | `Prod-VPC` |
| Accepter VPC | `Dev-VPC` |
| Account | `My Account` |
| Region | Current AWS Region |

Click **Create Peering Connection**.

After the request is created:

- Select the peering connection.
- Choose **Actions → Accept Request**.

The status should change to **Active**.

## Why is this step required?

A VPC Peering Connection enables private communication between two VPCs using their private IP addresses.

Traffic remains on the AWS network and does not traverse the public Internet.

## Dependency

- Production VPC
- Development VPC

## What happens if this step is skipped?

Instances in one VPC cannot communicate with instances in the other VPC using private IP addresses.

---

# Step 24: Update the Production Route Tables for VPC Peering

## Navigation

```text
AWS Console

→ VPC

→ Route Tables
```

## Configuration

Edit the Production Route Tables.

### Production Public Route Table

Add the following route.

| Destination | Target |
|-------------|--------|
| `10.1.0.0/16` | `Prod-Dev-Peering` |

### Production Private Route Table

Add the following route.

| Destination | Target |
|-------------|--------|
| `10.1.0.0/16` | `Prod-Dev-Peering` |

Save the changes.

## Why is this step required?

Even though the VPC Peering Connection exists, the route tables must know that traffic destined for the Development VPC should be sent through the peering connection.

## Dependency

- Active VPC Peering Connection

## What happens if this step is skipped?

Production instances will not know how to reach the Development VPC.

Packets destined for `10.1.0.0/16` will be dropped.

---

# Step 25: Update the Development Route Tables for VPC Peering

## Navigation

```text
AWS Console

→ VPC

→ Route Tables
```

## Configuration

Edit both Development Route Tables.

### Development Public Route Table

Add the following route.

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `Prod-Dev-Peering` |

### Development Private Route Table

Add the following route.

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `Prod-Dev-Peering` |

Save the route tables.

## Why is this step required?

Communication over VPC Peering is not automatic. Both VPCs must have routes pointing to each other's CIDR blocks through the peering connection.

## Dependency

- Active VPC Peering Connection

## What happens if this step is skipped?

Development instances will not be able to send traffic to the Production VPC.

Communication will fail even though the peering connection is active.

---

# Step 26: Update Security Groups to Allow Communication Between the Production and Development DB Subnets

## Navigation

```text
AWS Console

→ EC2

→ Security Groups
```

## Configuration

### Update `Prod-SG`

Add the following inbound rule.

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| All ICMP - IPv4 | ICMP | All | `10.1.2.0/24` |

If your application requires SSH access between the DB instances, also add:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | `22` | `10.1.2.0/24` |

### Update `Dev-SG`

Add the following inbound rule.

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| All ICMP - IPv4 | ICMP | All | `10.0.5.0/24` |

If SSH connectivity is required, also add:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | `22` | `10.0.5.0/24` |

Leave the outbound rules as **Allow All**.

## Why is this step required?

VPC Peering provides a network path, but Security Groups still determine whether traffic is allowed.

These rules permit communication between the Production DB subnet (`10.0.5.0/24`) and the Development DB subnet (`10.1.2.0/24`), satisfying the assignment requirement.

## Dependency

- Active VPC Peering Connection
- Route Tables configured for peering

## What happens if this step is skipped?

Although routes exist, Security Groups will block the traffic, preventing communication between the DB instances.

---

# Step 27: (Optional) Verify Internet Access from the Public and Private Instances Using AWS Systems Manager (SSM) or SSH

> **Note:** Private instances do not have public IP addresses. To test Internet access from `app1` and `dbcache`, you can:
>
> - Connect using AWS Systems Manager Session Manager (recommended), or
> - SSH through a bastion host in the `web` subnet if configured.

Run the following commands from the appropriate EC2 instances.

### Command

```bash
ping -c 4 amazon.com
```

### Command Explanation

| Command Part | Explanation |
|--------------|-------------|
| `ping` | Sends ICMP Echo Request packets to verify network connectivity. |
| `-c` | Specifies the number of packets to send before stopping. |
| `4` | Sends four ICMP packets. |
| `amazon.com` | The destination host used to verify Internet connectivity. |

Run the following command to verify package repository access.

### Command

```bash
sudo dnf check-update
```

### Command Explanation

| Command Part | Explanation |
|--------------|-------------|
| `sudo` | Executes the command with root privileges. |
| `dnf` | The package manager used by Amazon Linux 2023. |
| `check-update` | Checks for available package updates without installing them. |

## Why is this step required?

These commands verify that:

- The public Web instance has direct Internet access.
- The `app1` and `dbcache` private instances can access the Internet through the NAT Gateway.

## Dependency

- EC2 instances running
- Route tables configured
- NAT Gateway available
- Security Groups allowing outbound traffic

## What happens if this step is skipped?

You will not be able to confirm that the networking configuration satisfies the Internet connectivity requirements of the assignment.

---

# Verification

## Verification Step 1: Verify Production VPC Creation

### Action

Navigate to the VPC dashboard and verify that the Production VPC exists.

### Navigation

```text
AWS Console

→ VPC

→ Your VPCs
```

### Expected Output

You should see a VPC similar to the following.

| Name | CIDR |
|------|------|
| `Prod-VPC` | `10.0.0.0/16` |

### Why does this confirm success?

This confirms that the Production network has been created successfully and is ready to host AWS resources.

---

## Verification Step 2: Verify Development VPC Creation

### Action

Navigate to the Development VPC.

### Navigation

```text
AWS Console

→ VPC

→ Your VPCs
```

### Expected Output

| Name | CIDR |
|------|------|
| `Dev-VPC` | `10.1.0.0/16` |

### Why does this confirm success?

This confirms that the Development environment has its own isolated network.

---

## Verification Step 3: Verify All Subnets

### Action

Open the Subnets page.

### Navigation

```text
AWS Console

→ VPC

→ Subnets
```

### Expected Output

You should see the following subnets.

| VPC | Subnet |
|------|---------|
| Prod-VPC | `web` |
| Prod-VPC | `app1` |
| Prod-VPC | `app2` |
| Prod-VPC | `dbcache` |
| Prod-VPC | `db` |
| Dev-VPC | `web` |
| Dev-VPC | `db` |

### Why does this confirm success?

This confirms that all required network segments have been created.

---

## Verification Step 4: Verify Internet Gateway Attachments

### Action

Navigate to Internet Gateways.

### Navigation

```text
AWS Console

→ VPC

→ Internet Gateways
```

### Expected Output

| Internet Gateway | Attached VPC |
|------------------|--------------|
| `Prod-IGW` | `Prod-VPC` |
| `Dev-IGW` | `Dev-VPC` |

### Why does this confirm success?

This confirms that both VPCs have Internet connectivity available for their public subnets.

---

## Verification Step 5: Verify NAT Gateway

### Action

Navigate to NAT Gateways.

### Navigation

```text
AWS Console

→ VPC

→ NAT Gateways
```

### Expected Output

| Name | Status |
|------|--------|
| `Prod-NATGW` | `Available` |

### Why does this confirm success?

A status of **Available** indicates that private subnet traffic can be routed to the Internet through the NAT Gateway.

---

## Verification Step 6: Verify EC2 Instances

### Action

Navigate to the EC2 Instances page.

### Navigation

```text
AWS Console

→ EC2

→ Instances
```

### Expected Output

| Instance Name | State |
|---------------|-------|
| `web` (Production) | Running |
| `app1` | Running |
| `app2` | Running |
| `dbcache` | Running |
| `db` (Production) | Running |
| `web` (Development) | Running |
| `db` (Development) | Running |

### Why does this confirm success?

This confirms that all required EC2 instances have been launched in their respective subnets.

---

## Verification Step 7: Verify VPC Peering Connection

### Action

Navigate to Peering Connections.

### Navigation

```text
AWS Console

→ VPC

→ Peering Connections
```

### Expected Output

| Peering Connection | Status |
|--------------------|--------|
| `Prod-Dev-Peering` | `Active` |

### Why does this confirm success?

An **Active** status confirms that the VPC peering connection has been successfully established.

---

## Verification Step 8: Verify Route Tables

### Action

Open Route Tables.

### Navigation

```text
AWS Console

→ VPC

→ Route Tables
```

### Expected Output

The route tables should include the following routes.

### Production Route Tables

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Prod-IGW` (Public RT) |
| `0.0.0.0/0` | `Prod-NATGW` (Private RT) |
| `10.1.0.0/16` | `Prod-Dev-Peering` |

### Development Route Tables

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | `Dev-IGW` (Public RT) |
| `10.0.0.0/16` | `Prod-Dev-Peering` |

### Why does this confirm success?

These routes ensure:

- Internet access for public subnets.
- Internet access for selected private resources through the NAT Gateway.
- Communication between the two VPCs through the peering connection.

---

## Verification Step 9: Verify Internet Connectivity

### Action

Connect to the Production `web` instance using SSH or AWS Systems Manager and run the following command.

### Command

```bash
ping -c 4 amazon.com
```

### Command Explanation

| Command Part | Explanation |
|--------------|-------------|
| `ping` | Sends ICMP Echo Requests to verify network connectivity. |
| `-c` | Specifies the number of packets to send. |
| `4` | Sends four packets. |
| `amazon.com` | Destination host used to test Internet connectivity. |

### Expected Output

```text
PING amazon.com (...)
64 bytes from ...
64 bytes from ...
64 bytes from ...
64 bytes from ...

4 packets transmitted, 4 received
```

### Why does this confirm success?

Receiving replies confirms that the instance has outbound Internet connectivity.

---

## Verification Step 10: Verify Communication Between the Production and Development DB Subnets

### Action

From one DB instance, ping the private IP address of the DB instance in the other VPC.

### Command

```bash
ping -c 4 <Private-IP-of-Other-DB-Instance>
```

Replace `<Private-IP-of-Other-DB-Instance>` with the actual private IP address of the target EC2 instance.

### Command Explanation

| Command Part | Explanation |
|--------------|-------------|
| `ping` | Sends ICMP Echo Requests to test connectivity. |
| `-c` | Limits the number of packets sent. |
| `4` | Sends four packets. |
| `<Private-IP-of-Other-DB-Instance>` | The private IP address of the database EC2 instance in the other VPC. |

### Expected Output

```text
64 bytes from 10.x.x.x
64 bytes from 10.x.x.x

4 packets transmitted, 4 received
```

### Why does this confirm success?

Successful replies confirm that:

- The VPC Peering Connection is active.
- Route tables are configured correctly.
- Security Groups permit the required traffic between the Production and Development DB subnets.

---

# Cleanup / Cost Optimization

## Cleanup Step 1: Terminate All EC2 Instances

### Navigation

```text
AWS Console

→ EC2

→ Instances
```

### Action

Terminate all seven EC2 instances.

- `web` (Production)
- `app1`
- `app2`
- `dbcache`
- `db` (Production)
- `web` (Development)
- `db` (Development)

### Why is this required?

EC2 instances incur charges while running (outside Free Tier limits). They should be terminated first because other networking resources depend on them.

### What happens if this resource is not deleted?

Running instances continue consuming compute resources and may incur charges.

---

## Cleanup Step 2: Delete the NAT Gateway

### Navigation

```text
AWS Console

→ VPC

→ NAT Gateways
```

### Action

Delete `Prod-NATGW`.

### Why is this required?

The NAT Gateway is the primary billable resource in this case study.

### What happens if this resource is not deleted?

It continues to incur hourly charges and data processing charges.

---

## Cleanup Step 3: Release the Elastic IP Address

### Navigation

```text
AWS Console

→ EC2

→ Elastic IPs
```

### Action

Release `Prod-NAT-EIP`.

### Why is this required?

The Elastic IP is no longer needed after deleting the NAT Gateway.

### What happens if this resource is not deleted?

An unattached Elastic IP address may incur charges.

---

## Cleanup Step 4: Delete the VPC Peering Connection

### Navigation

```text
AWS Console

→ VPC

→ Peering Connections
```

### Action

Delete `Prod-Dev-Peering`.

### Why is this required?

The peering connection should be removed before deleting the VPCs.

### What happens if this resource is not deleted?

The VPCs cannot be deleted while an active peering connection exists.

---

## Cleanup Step 5: Delete Custom Route Tables

### Navigation

```text
AWS Console

→ VPC

→ Route Tables
```

### Action

Delete:

- `Prod-Public-RT`
- `Prod-Private-RT`
- `Dev-Public-RT`
- `Dev-Private-RT`

Ensure that no subnet associations remain before deletion.

### Why is this required?

Custom route tables must be removed before deleting the VPC.

### What happens if this resource is not deleted?

The VPC deletion process will fail due to dependent resources.

---

## Cleanup Step 6: Delete Custom Network ACLs

### Navigation

```text
AWS Console

→ VPC

→ Network ACLs
```

### Action

Delete the custom Network ACLs created for the Production and Development VPCs.

### Why is this required?

Removing custom Network ACLs helps eliminate dependencies before deleting the VPC.

### What happens if this resource is not deleted?

The VPC may not be deleted until dependent resources are removed.

---

## Cleanup Step 7: Delete Security Groups

### Navigation

```text
AWS Console

→ EC2

→ Security Groups
```

### Action

Delete:

- `Prod-SG`
- `Dev-SG`

### Why is this required?

Security Groups are VPC resources and should be removed before deleting the VPC.

### What happens if this resource is not deleted?

The VPC cannot be deleted while custom Security Groups still exist.

---

## Cleanup Step 8: Detach and Delete Internet Gateways

### Navigation

```text
AWS Console

→ VPC

→ Internet Gateways
```

### Action

For each Internet Gateway:

1. Detach it from its VPC.
2. Delete it.

Delete:

- `Prod-IGW`
- `Dev-IGW`

### Why is this required?

Internet Gateways are attached VPC resources and must be detached before deletion.

### What happens if this resource is not deleted?

The associated VPC cannot be deleted.

---

## Cleanup Step 9: Delete All Subnets

### Navigation

```text
AWS Console

→ VPC

→ Subnets
```

### Action

Delete all Production and Development subnets.

### Why is this required?

Subnets are dependent resources within a VPC.

### What happens if this resource is not deleted?

AWS will prevent deletion of the VPC while subnets still exist.

---

## Cleanup Step 10: Delete Both VPCs

### Navigation

```text
AWS Console

→ VPC

→ Your VPCs
```

### Action

Delete:

- `Prod-VPC`
- `Dev-VPC`

### Why is this required?

This removes all remaining networking resources created for the assignment and completes the cleanup process.

### What happens if this resource is not deleted?

The VPCs and any remaining associated resources will continue to exist in your AWS account, potentially leading to unnecessary resource usage and making future lab management more difficult.

---
