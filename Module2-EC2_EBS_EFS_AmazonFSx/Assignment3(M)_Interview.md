# AWS Networking, EC2 & Amazon EFS Interview Handbook

## Table of Contents

### Section 1 – Networking Fundamentals

1. What is a Network?
2. TCP/IP Model Overview
3. OSI Model vs TCP/IP Model
4. What is an IP Address (Internet Protocol Address)?
5. Types of IP Addresses
   - Public IP
   - Private IP
   - Static IP
   - Dynamic IP
   - Elastic IP (AWS)
6. IPv4 (Internet Protocol Version 4) vs IPv6 (Internet Protocol Version 6)
7. MAC Address (Media Access Control Address)
8. TCP (Transmission Control Protocol) vs UDP (User Datagram Protocol)
9. Ports
10. Firewall
    - Windows Firewall
    - Linux Firewall
    - AWS Security Group
    - AWS Network ACL

---

### Section 2 – AWS Networking Fundamentals

11. What is AWS Global Infrastructure?
12. Regions
13. Availability Zones (AZ)
14. How Availability Zones Affect AWS Resources

---

### Section 3 – AWS Networking

15. What is a VPC (Virtual Private Cloud)?
16. Why Do We Need a VPC?
17. CIDR (Classless Inter-Domain Routing)
    - What is CIDR?
    - What was Classful Addressing?
    - Why was CIDR Introduced?
    - Prefix Length
    - Network Bits
    - Host Bits
    - CIDR Calculations
    - AWS Reserved IP Addresses
18. Subnet
19. Why Are Subnets Mandatory?
20. Public Subnet
21. Private Subnet
22. Route Table
23. Internet Gateway (IGW)
24. Public IP Assignment
25. NAT Gateway
26. Security Group
27. Network ACL (NACL)
28. Security Group vs Network ACL
29. Complete AWS Packet Flow

---

### Section 4 – Amazon EC2

30. What is Amazon EC2 (Elastic Compute Cloud)?
31. EC2 Instance Types
32. Amazon Machine Image (AMI)
33. Key Pair
34. Elastic Network Interface (ENI)
35. EC2 Networking

---

### Section 5 – Amazon EFS

36. What is Amazon EFS (Elastic File System)?
37. EFS Mount Targets
38. Network File System (NFS)
39. Mounting Amazon EFS on EC2
40. Assignment Architecture

---

### Section 6 – TCP/IP Mapping of This Assignment

41. Complete TCP/IP Flow for SSH
42. Complete TCP/IP Flow for Amazon EFS
43. How Every AWS Component Fits into the TCP/IP Model

---

### Section 7 – Frequently Asked Interview Questions

- Basic Questions
- Intermediate Questions
- Advanced Questions

---

### Section 8 – Scenario-Based Questions

- Networking Scenarios
- EC2 Scenarios
- Amazon EFS Scenarios
- Security Scenarios

---

### Section 9 – Production Design Questions

- Architecture Design
- High Availability
- Scalability
- Security
- Cost Optimization

---

### Section 10 – Troubleshooting Guide

- SSH Issues
- Internet Connectivity Issues
- Security Group Issues
- Network ACL Issues
- Route Table Issues
- Internet Gateway Issues
- NAT Gateway Issues
- Public IP Issues
- Amazon EFS Issues
- DNS Issues

---

### Section 11 – Local Lab vs AWS

- VirtualBox / VMware vs AWS
- Linux Networking vs AWS Networking
- Local Firewall vs AWS Firewall
- Shared Folder vs Amazon EFS
- Local Network vs Amazon VPC

---

### Section 12 – Common Interview Mistakes

- Networking Misconceptions
- EC2 Misconceptions
- Security Misconceptions
- Amazon EFS Misconceptions
- AWS Networking Misconceptions


---


⭐Mind Map

EC2 + EFS Assignment
```
        AWS
         │
         ▼
       Region
         │
         ▼
        VPC
         │
         ▼
      Subnet
         │
         ▼
  Route Table
         │
         ▼
 Internet Gateway
         │
         ▼
 Security Group
         │
         ▼
        EC2
   ┌─────┼─────┐
   ▼     ▼     ▼
 Ubuntu Amazon RHEL
        │
        ▼
      Root EBS
        │
        ▼
        ENI
        │
        ▼
  Mount Target
        │
        ▼
     Amazon EFS
        │
        ▼
      NFS 4.1
```
---

⭐ Component relationship
```
AMI
 ↓
Creates
 ↓
Root EBS
 ↓
Attached To
 ↓
EC2
 ↓
Connected Through
 ↓
ENI
 ↓
Inside
 ↓
Subnet
 ↓
Inside
 ↓
VPC
 ↓
Protected By
 ↓
Security Group
 ↓
Accesses
 ↓
Mount Target
 ↓
Amazon EFS
```
---

⭐ Rapid Fire (30 Seconds Revision) ⭐

| Question                       | Answer                |
| ------------------------------ | --------------------- |
| EC2                            | Virtual Machine       |
| VPC                            | Virtual Network       |
| Subnet                         | Network Division      |
| ENI                            | Virtual NIC           |
| EBS                            | Block Storage         |
| EFS                            | Shared File Storage   |
| SG                             | Stateful Firewall     |
| NACL                           | Stateless Firewall    |
| IGW                            | Internet Access       |
| NAT(n/w Address Translation)   | Outbound Internet     |
| Mount Target                   | EFS Network Endpoint  |
| NFS                            | File Sharing Protocol |
| Port                           | 2049                  |

---
# Section 1 – Networking Fundamentals

---

# Q1. What is a Network?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Network |
| **Category** | Computer Networking |
| **Type** | Networking Concept |
| **Hardware / Software / Protocol** | Combination of Hardware and Software |
| **TCP/IP Layer** | Overall Concept (Foundation of the TCP/IP Model) |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Definition

A **Network** is a collection of **two or more devices** connected together to **communicate** and **share resources** using standard networking protocols.

The devices can be physical or virtual.

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Communication and Resource Sharing |
| **Communication Medium** | Wired or Wireless |
| **Uses** | File Sharing, Remote Access, Internet, Cloud Computing |
| **Examples** | Home Wi-Fi, Office LAN, Internet, AWS VPC, Router, Switches |

---

## Types of Networks

| Type | Full Form | Coverage Area | Example |
|------|-----------|---------------|---------|
| **PAN** | Personal Area Network | Few meters | Bluetooth between a phone and earbuds |
| **LAN** | Local Area Network | Home / Office / Building | Home Wi-Fi, Office Network |
| **MAN** | Metropolitan Area Network | City | ISP Network across a city |
| **WAN** | Wide Area Network | Country / Worldwide | Internet |

---

## Components of a Network

A network consists of multiple components working together.

| Component | Type | Hardware / Software / Protocol | Purpose |
|----------|------|--------------------------------|---------|
| Computer | Device | Hardware | Sends and receives data |
| Server | Device | Hardware | Provides services to clients |
| Router | Networking Device | Hardware | Connects different networks |
| Switch | Networking Device | Hardware | Connects devices within the same network |
| Network Interface Card (NIC) | Networking Device | Hardware | Connects a device to the network |
| Network Cable | Communication Medium | Hardware | Transfers data physically |
| Wi-Fi Access Point | Networking Device | Hardware | Provides wireless connectivity |
| IP Address | Logical Address | Software / Logical | Identifies a device on the network |
| MAC Address | Physical Address | Hardware | Identifies a Network Interface Card |
| TCP/IP | Protocol Suite | Software Protocol | Defines communication rules |

---

## How Does a Network Work?

- Every device connected to the network has an identity (IP Address),
- communication follows predefined networking protocols such as TCP/IP.

---

## How It Works Internally

Suppose your laptop connects to an Ubuntu EC2 instance using SSH.

```text
Laptop
    │
    ▼
Internet
    │
    ▼
AWS Network
    │
    ▼
Amazon EC2
```

The network simply provides the communication path.

The actual communication is handled by protocols like:

- TCP (Transmission Control Protocol)
- IP (Internet Protocol)
- SSH (Secure Shell)

These protocols will be discussed in the next sections.

---

## Assignment Example

This assignment uses networking in two places.

### 1. Laptop → Amazon EC2

Your laptop communicates with the Ubuntu EC2 instance using **SSH**.

```text
Laptop
      │
      ▼
Internet
      │
      ▼
Ubuntu EC2
```

---

### 2. Amazon EC2 → Amazon EFS

All three EC2 instances communicate with Amazon EFS using **NFS (Network File System)**.

```text
Ubuntu EC2
        │
RHEL EC2
        │
Amazon Linux 2
        │
        ▼
Amazon EFS
```

Without networking, these resources cannot communicate.

---

## Production Usage

In production environments, networks connect:

- Users
- Applications
- Databases
- Storage Systems
- Monitoring Tools
- CI/CD Pipelines
- Cloud Resources

A production network must provide:

- High Availability
- Security
- Scalability
- Reliable Communication

---

## Troubleshooting

### Problem

Amazon EC2 cannot communicate with Amazon EFS.

### Possible Causes

| Possible Cause | Explanation                          |
|--------------- |--------------------------------------|
| Different VPC  | EC2 and EFS are not in the same VPC. |
| Security Group | NFS Port `2049` is blocked.          |
| Network ACL    | Traffic is denied.                   |
| Route Table    | Incorrect routing configuration.     |
| Mount Target   | EFS Mount Target is unavailable.     |
| DNS Resolution | EFS DNS name cannot be resolved.     |

```
# Check DNS resolution
nslookup fs-xxxxxxxx.efs.ap-south-1.amazonaws.com

# Test NFS port
nc -zv fs-xxxxxxxx.efs.ap-south-1.amazonaws.com 2049

# Check installed NFS client
rpm -qa | grep nfs-utils          # Amazon Linux / RHEL
dpkg -l | grep nfs-common         # Ubuntu

# Check mounted file systems
mount | grep efs

# View routing table
ip route

# Verify network connectivity
ping fs-xxxxxxxx.efs.ap-south-1.amazonaws.com

# Check firewall
sudo ufw status
sudo firewall-cmd --list-all

# View mount errors
dmesg | tail -20

# Check system logs
journalctl -xe
```

---

## Common Interview Mistakes

### Mistake 1

❌ A Network means the Internet.

✅ The Internet is one example of a network.

Private networks such as Home LANs, Office Networks, and AWS VPCs are also networks.

---

### Mistake 2

❌ A network consists only of cables.

✅ A network consists of hardware, software, logical addressing, and communication protocols.

---

### Mistake 3

❌ AWS Networking is different from Computer Networking.

✅ AWS Networking follows the same networking principles used in traditional computer networks.

---

## Interview Cross Questions

### Q1. Can two computers communicate without the Internet?

**Answer:**

Yes.

If both computers are connected to the same network (LAN), they can communicate without Internet access.

---

### Q2. Is AWS VPC a Network?

**Answer:**

Yes.

A VPC (Virtual Private Cloud) is a private virtual network created inside the AWS Cloud.

---

### Q3. What is the difference between a Network and the Internet?

| Network | Internet |
|----------|----------|
| Can be Private or Public | Public Network |
| Limited or Global Coverage | Worldwide Coverage |
| Connects Specific Devices | Connects Millions of Networks Worldwide |

---

## Related Topics

- TCP/IP Model
- OSI Model
- IP Address
- TCP
- Ports
- VPC
- Subnet

---

# Q2. What is the TCP/IP Model?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Transmission Control Protocol / Internet Protocol |
| **Category** | Computer Networking |
| **Type** | Networking Model / Protocol Suite |
| **Hardware / Software / Protocol** | Software Protocol Suite |
| **TCP/IP Layer** | Entire Networking Stack |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network

---

## Definition

The **TCP/IP (Transmission Control Protocol / Internet Protocol) Model** is a standard networking model that defines **how data is transmitted between devices over a network**.

It divides the communication process into **four logical layers**, where each layer performs a specific task.

---

## Why Do We Need the TCP/IP Model?

Without a standard communication model:

- Devices from different manufacturers would not be able to communicate.
- Every application would need its own communication method.
- Data transmission would become inconsistent and unreliable.

The TCP/IP model provides a common set of rules that enables all devices connected to a network to communicate with each other.

---

## Quick Revision

| Layer                      | Purpose                                     | Examples                         |
|----------------------------|---------------------------------------------|----------------------------------|
| **Application Layer**      | Provides network services to applications   | SSH, HTTP, HTTPS, DNS, NFS       |
| **Transport Layer**        | End-to-end communication and port management| TCP, UDP                         |
| **Internet Layer**         | Logical addressing and routing              | IPv4, IPv6                       |
| **Network Access Layer**   | Physical transmission of data               | Ethernet, Wi-Fi, MAC Address, NIC|

---

## TCP/IP Model

```text
+------------------------------------------------------+
| Application Layer                                    |
|------------------------------------------------------|
| SSH | HTTP | HTTPS | DNS | NFS                       |
+------------------------------------------------------+
| Transport Layer                                      |
|------------------------------------------------------|
| TCP | UDP | Port Numbers                             |
+------------------------------------------------------+
| Internet Layer                                       |
|------------------------------------------------------|
| IPv4 | IPv6 | Routing | IP Address                   |
+------------------------------------------------------+
| Network Access Layer                                 |
|------------------------------------------------------|
| MAC Address | Ethernet | Wi-Fi | Network Interface   |
+------------------------------------------------------+
```

---

## How Does the TCP/IP Model Work?

Suppose you connect to an EC2 instance using SSH.

The communication follows these layers:

```text
SSH Command
        │
        ▼
Application Layer
        │
        ▼
TCP
        │
        ▼
IP Address
        │
        ▼
MAC Address / Network Interface
        │
        ▼
Network
```

Each layer performs its own task before passing the data to the next layer.

---

## How It Works Internally

When you execute:

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

The request flows through the TCP/IP model as follows:

| TCP/IP Layer | What Happens |
|--------------|--------------|
| **Application Layer** | SSH creates the remote login request. |
| **Transport Layer** | TCP establishes a reliable connection using Port `22`. |
| **Internet Layer** | The destination IP address identifies the EC2 instance. |
| **Network Access Layer** | The data is transmitted through the physical network to AWS. |

The response from the EC2 instance follows the same layers in reverse order.

---

## Assignment Example

This assignment uses every layer of the TCP/IP model.

| Layer | Used In This Assignment |
|-------|-------------------------|
| **Application Layer** | SSH, NFS |
| **Transport Layer** | TCP, Ports `22` and `2049` |
| **Internet Layer** | Public IP, Private IP, IPv4 |
| **Network Access Layer** | EC2 Network Interface (ENI), MAC Address |

---

## Real DevOps Usage

A DevOps Engineer uses the TCP/IP model to:

- Understand application communication.
- Troubleshoot connectivity issues.
- Configure firewalls.
- Open required ports.
- Secure cloud infrastructure.
- Diagnose network failures.

Almost every networking issue can be analyzed by identifying **which TCP/IP layer is failing**.

---

## Production Usage

Production applications continuously use the TCP/IP model.

Examples:

- Users access websites using **HTTPS**.
- Developers connect to servers using **SSH**.
- Applications communicate with databases using **TCP**.
- EC2 instances mount Amazon EFS using **NFS over TCP**.

Understanding the TCP/IP model helps identify where communication problems occur.

---

## Troubleshooting

### Problem

Unable to connect to an EC2 instance using SSH.

### Layer-wise Troubleshooting

| Layer | What to Check |
|-------|---------------|
| **Application Layer** | Is the SSH service running on the EC2 instance? |
| **Transport Layer** | Is Port `22` allowed in the Security Group and Network ACL? |
| **Internet Layer** | Does the EC2 instance have the correct Public IP? |
| **Network Access Layer** | Is the EC2 instance reachable through the network? |

---

## Common Interview Mistakes

### Mistake 1

❌ TCP/IP is only used on the Internet.

✅ TCP/IP is used on both private and public networks.

---

### Mistake 2

❌ TCP/IP and TCP are the same.

✅ TCP is one protocol within the TCP/IP protocol suite.

---

### Mistake 3

❌ Every AWS service belongs to a single TCP/IP layer.

✅ AWS services often use multiple TCP/IP layers depending on how they communicate.

---

## Interview Cross Questions

### Q1. Why is the TCP/IP model important?

**Answer:**

It provides a standard framework that allows devices and applications to communicate reliably over networks.

---

### Q2. How many layers does the TCP/IP model have?

**Answer:**

Four layers:

- Application Layer
- Transport Layer
- Internet Layer
- Network Access Layer

---

### Q3. Which TCP/IP layers are used in this assignment?

**Answer:**

All four layers are used while connecting to EC2 through SSH and mounting Amazon EFS using NFS.

---

## Related Topics

- IP Address
- TCP vs UDP
- Ports
- Firewall
- VPC
- Security Group
- Amazon EFS

---

## One-line Interview Answer

> The TCP/IP (Transmission Control Protocol / Internet Protocol) model is a four-layer networking model that defines how devices communicate over a network using standard communication protocols.
---

---

# Q3. What is an IP Address (Internet Protocol Address)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Internet Protocol Address |
| **Category** | Computer Networking |
| **Type** | Logical Address |
| **Hardware / Software / Protocol** | Software / Logical Address |
| **TCP/IP Layer** | Internet Layer |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model

---

## Definition

An **IP (Internet Protocol) Address** is a **unique logical address** assigned to a device on a network.

It identifies **where a device is located** on the network so that data can be sent to the correct destination.

Every device that communicates over a network requires an IP address.

---

## Why Do We Need an IP Address?

Without an IP address:

- Devices cannot be uniquely identified.
- Data cannot reach the intended destination.
- Communication between devices is impossible.

An IP address acts like the **address of a house**, allowing data to be delivered to the correct device.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Identifies a device on a network |
| **Address Type** | Logical Address |
| **Assigned By** | Network Administrator, ISP, DHCP Server, or AWS |
| **Used For** | Device Identification and Routing |
| **TCP/IP Layer** | Internet Layer |

---

## Types of IP Addresses

| Type | Description | Example |
|------|-------------|---------|
| **Public IP** | Reachable over the Internet | `54.x.x.x` |
| **Private IP** | Used inside private networks | `10.0.1.10` |
| **Static IP** | Remains the same | Database Server |
| **Dynamic IP** | Changes automatically | Home Wi-Fi Device |
| **Elastic IP (AWS)** | Static Public IPv4 address provided by AWS | Associated with an EC2 instance |

---

## Public IP vs Private IP

| Public IP | Private IP |
|------------|------------|
| Globally unique | Unique only within a private network |
| Accessible over the Internet | Not directly accessible from the Internet |
| Assigned by ISP or AWS | Assigned within the private network or VPC |
| Used for Internet communication | Used for internal communication |

---

## How Does an IP Address Work?

Suppose your laptop wants to connect to an EC2 instance.

```text
Laptop
IP: 192.168.1.10
        │
        ▼
Internet
        │
        ▼
Public IP
54.xx.xx.xx
        │
        ▼
Ubuntu EC2
```

Your laptop sends data to the EC2 instance's **Public IP**.

AWS then delivers the data to the correct EC2 instance.

---

## How It Works Internally

When you execute:

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

The SSH application passes the destination IP address to the **Internet Layer**.

The Internet Layer uses this IP address to determine where the packet should be delivered.

Routers on the Internet read the destination IP address and forward the packet until it reaches the AWS network and finally the EC2 instance.

---

## Assignment Example

This assignment uses two types of IP addresses.

| Resource | IP Type | Purpose |
|----------|---------|---------|
| Your Laptop | Public / Private | Initiates the SSH connection |
| Ubuntu EC2 | Public IP | Allows SSH access from your laptop |
| Ubuntu EC2 | Private IP | Communicates with Amazon EFS |
| RHEL EC2 | Private IP | Communicates with Amazon EFS |
| Amazon Linux 2 EC2 | Private IP | Communicates with Amazon EFS |
| Amazon EFS | Private IP | Shared storage inside the VPC |

---

## Real DevOps Usage

A DevOps Engineer uses IP addresses to:

- Connect to servers using SSH.
- Configure application communication.
- Troubleshoot connectivity issues.
- Configure DNS records.
- Design VPCs and subnets.
- Secure infrastructure using Security Groups and Network ACLs.

---

## Production Usage

Production environments typically use:

- **Private IPs** for internal communication between application servers, databases, and storage.
- **Public IPs** only where Internet access is required.
- **Elastic IPs** for resources that require a fixed public address.

Following this approach improves both security and scalability.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Incorrect Public IP | The EC2 Public IP has changed or is incorrect. |
| No Public IP Assigned | The EC2 instance does not have a Public IP. |
| Wrong Elastic IP | The Elastic IP is associated with another resource. |
| DNS Resolution Issue | The hostname resolves to the wrong IP address. |

---

## Interview Tip

An **IP Address identifies a device**, while a **MAC Address identifies the network interface** of that device.

---

## Common Interview Mistakes

### Mistake 1

❌ Public IP and Private IP are interchangeable.

✅ They serve different purposes.

---

### Mistake 2

❌ Every EC2 instance has a Public IP.

✅ Only EC2 instances configured with a Public IP (or Elastic IP) can be reached directly from the Internet.

---

### Mistake 3

❌ IP Address identifies the hardware.

✅ The IP Address is a **logical address**. The **MAC Address** identifies the hardware interface.

---

## Interview Cross Questions

### Q1. Can two devices have the same Private IP?

**Answer:**

Yes, if they are in different private networks or different VPCs.

---

### Q2. Can two devices have the same Public IP?

**Answer:**

No.

A Public IP must be globally unique.

---

### Q3. Can an EC2 communicate with Amazon EFS using its Public IP?

**Answer:**

No.

Amazon EFS communication occurs using **Private IP addresses** inside the VPC.

---

## Related Topics

- IPv4 vs IPv6
- MAC Address
- VPC
- CIDR
- Public IP
- Private IP
- Elastic IP

---

## One-line Interview Answer

> An IP (Internet Protocol) Address is a unique logical address that identifies a device on a network and enables data to be routed to the correct destination.
---

---

# Q4. What are the Different Types of IP Addresses?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Internet Protocol Address |
| **Category** | Computer Networking |
| **Type** | Logical Address Classification |
| **Hardware / Software /Protocol** | Software / Logical Address |
| **TCP/IP Layer** | Internet Layer |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address

---

## Definition

IP addresses can be classified based on **where they are used**, **how they are assigned**, and **their accessibility**.

Understanding these types is essential because AWS uses different IP address types for different networking requirements.

---

## Why Do We Need Different Types of IP Addresses?

Different devices have different communication requirements.

For example:

- A web server must be reachable from the Internet.
- A database should only be accessible within the private network.
- Some servers require a permanent IP address.
- Some devices can use temporary IP addresses.

Different IP address types help meet these requirements.

---

## Quick Revision

| Type | Internet Accessible | Changes Automatically | Example |
|------|:-------------------:|:---------------------:|---------|
| **Public IP** | ✅ Yes | Usually Yes | Web Server |
| **Private IP** | ❌ No | No (inside VPC) | Database, EC2 Internal Communication |
| **Static IP** | Depends | ❌ No | Company Server |
| **Dynamic IP** | Depends | ✅ Yes | Home Computer |
| **Elastic IP (AWS)** | ✅ Yes | ❌ No | Production EC2 |

---

## Types of IP Addresses

### 1. Public IP Address

A **Public IP Address** is globally unique and can be accessed from anywhere on the Internet.

#### Characteristics

- Globally unique
- Internet accessible
- Used for public-facing resources
- Assigned by ISP or AWS

#### AWS Example

```text
Laptop
     │
Internet
     │
54.xx.xx.xx
     │
Ubuntu EC2
```

In this assignment, the Ubuntu EC2 instance uses a Public IP so you can connect using SSH.

---

### 2. Private IP Address

A **Private IP Address** is used only within a private network.

It cannot be accessed directly from the Internet.

#### Characteristics

- Used for internal communication
- Not Internet routable
- Assigned from the VPC CIDR block

#### AWS Example

```text
Ubuntu EC2
10.0.1.10
      │
      ▼
Amazon EFS
10.0.1.50
```

Amazon EFS communication always uses Private IP addresses.

---

### 3. Static IP Address

A **Static IP Address** never changes unless it is manually modified.

#### Characteristics

- Permanent address
- Easier for DNS configuration
- Suitable for production servers

---

### 4. Dynamic IP Address

A **Dynamic IP Address** is automatically assigned and may change over time.

It is commonly assigned using **DHCP (Dynamic Host Configuration Protocol)**.

#### Characteristics

- Automatically assigned
- May change
- Easy to manage

Most home Internet connections use Dynamic IP addresses.

---

### 5. Elastic IP (AWS)

An **Elastic IP (EIP)** is a static Public IPv4 address provided by AWS.

Unlike a normal Public IP, an Elastic IP remains the same until you release it.

#### Characteristics

- Static Public IPv4
- AWS managed
- Can be reassociated with another EC2 instance
- Commonly used in production

---

## Comparison Table

| Feature | Public IP | Private IP | Elastic IP |
|----------|-----------|------------|------------|
| Internet Accessible | ✅ Yes | ❌ No | ✅ Yes |
| Globally Unique | ✅ Yes | ❌ No | ✅ Yes |
| Used Inside VPC | ✅ Sometimes | ✅ Yes | ✅ Sometimes |
| Changes Automatically | Usually Yes | No | ❌ No |
| Suitable for Production | Limited | Internal Communication | ✅ Yes |

---

## How It Works Internally

Suppose your laptop connects to Ubuntu EC2.

```text
Laptop
      │
Internet
      │
Public IP
54.xx.xx.xx
      │
Ubuntu EC2
Private IP
10.0.1.10
      │
Amazon EFS
Private IP
10.0.1.50
```

Notice:

- Public IP is used between your laptop and EC2.
- Private IP is used between EC2 and Amazon EFS.

---

## Assignment Example

| Resource | IP Address Used | Reason |
|-----------|-----------------|--------|
| Laptop | Public IP | Connects over the Internet |
| Ubuntu EC2 | Public + Private | SSH + Internal Communication |
| RHEL EC2 | Private | Communicates with EFS |
| Amazon Linux 2 | Private | Communicates with EFS |
| Amazon EFS | Private | Shared Storage inside VPC |

---

## Real DevOps Usage

A DevOps Engineer typically uses:

- Public IPs for Bastion Hosts, Load Balancers, and public web servers.
- Private IPs for application servers, databases, and storage.
- Elastic IPs for production resources requiring a fixed public address.

Using Private IPs wherever possible improves security.

---

## Production Usage

A typical production architecture:

```text
Internet
      │
Elastic IP
      │
Load Balancer
      │
Private EC2
      │
Database
```

Only the Load Balancer has a Public or Elastic IP.

Application servers and databases use Private IPs.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| No Public IP | EC2 cannot be reached from the Internet. |
| Incorrect Public IP | SSH request goes to another destination. |
| Elastic IP associated elsewhere | Wrong resource receives traffic. |
| Private IP used from the Internet | Private IPs are not Internet routable. |

---

## Interview Tip

Every EC2 instance always has a **Private IP**.

A **Public IP is optional**.

---

## Common Interview Mistakes

### Mistake 1

❌ Every EC2 instance has a Public IP.

✅ Every EC2 instance has a Private IP.
A Public IP is assigned only when configured.

---

### Mistake 2

❌ Amazon EFS uses Public IPs.

✅ Amazon EFS communicates using Private IPs inside the VPC.

---

### Mistake 3

❌ Elastic IP and Public IP are the same.

✅ An Elastic IP is a **static Public IPv4 address** managed by AWS.

---

## Interview Cross Questions

### Q1. Can an EC2 instance have only a Private IP?

**Answer:**

Yes.

Private EC2 instances commonly use only Private IP addresses.

---

### Q2. Why does Amazon EFS use Private IPs instead of Public IPs?

**Answer:**

Because Amazon EFS is designed for secure communication within a VPC.

---

### Q3. When should you use an Elastic IP?

**Answer:**

When a resource requires a fixed Public IPv4 address, such as a Bastion Host or production server.

---

## Related Topics

- IPv4 vs IPv6
- VPC
- CIDR
- Subnet
- Internet Gateway
- NAT Gateway
- Elastic Network Interface (ENI)

---

## One-line Interview Answer

> IP addresses are classified as Public, Private, Static, Dynamic, and Elastic IPs, each serving different networking and communication requirements.

---

---

# Q5. What is the Difference Between IPv4 (Internet Protocol Version 4) and IPv6 (Internet Protocol Version 6)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | IPv4 - Internet Protocol Version 4<br>IPv6 - Internet Protocol Version 6 |
| **Category** | Computer Networking |
| **Type** | Internet Protocol |
| **Hardware / Software / Protocol** | Software Protocol |
| **TCP/IP Layer** | Internet Layer |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ IPv4 &nbsp;&nbsp;&nbsp;❌ IPv6 |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- Types of IP Addresses

---

## Definition

**IPv4** and **IPv6** are two versions of the **Internet Protocol (IP)** used to uniquely identify devices and route data across networks.

IPv4 is the most widely used version today, while IPv6 was introduced to overcome the limitations of IPv4, especially the shortage of available IP addresses.

---

## Why Do We Need IPv6?

IPv4 provides a limited number of addresses.

As the number of Internet-connected devices increased dramatically, IPv4 addresses began to run out.

IPv6 was introduced to provide a much larger address space and additional networking improvements.

---

## Quick Revision

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address Length** | 32 Bits | 128 Bits |
| **Notation** | Decimal | Hexadecimal |
| **Address Example** | `192.168.1.10` | `2001:db8::10` |
| **Total Addresses** | ~4.3 Billion | ~340 Undecillion (2¹²⁸) |
| **NAT Required** | Yes (commonly) | Usually No |
| **Current Usage** | Most Common | Increasing Adoption |

---

## IPv4 Address Format

An IPv4 address contains **32 bits** divided into **4 octets**.

Example:

```
192.168.1.10
```

| Octet | Value |
|--------|------|
| 1 | 192 |
| 2 | 168 |
| 3 | 1 |
| 4 | 10 |

Each octet contains **8 bits**.

```
8 + 8 + 8 + 8 = 32 Bits
```

---

## IPv6 Address Format

An IPv6 address contains **128 bits** divided into **8 groups**.

Example:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

IPv6 uses **hexadecimal** values instead of decimal numbers.

---

## Comparison Table

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Bit Length | 32 | 128 |
| Address Representation | Decimal | Hexadecimal |
| Number of Addresses | ~4.3 Billion | 2¹²⁸ (Extremely Large) |
| Header Size | Variable | Fixed |
| NAT Required | Yes (commonly) | Usually No |
| Broadcast | Supported | Not Supported |
| Security | Optional | Built with IPsec support |

---

## How It Works Internally

When data is sent across a network, the Internet Layer places the destination IP address into the packet.

Example using IPv4:

```text
Laptop
192.168.1.20
      │
      ▼
Packet Destination

54.xx.xx.xx

      │
      ▼
Amazon EC2
```

Routers examine the destination IP address and forward the packet toward its destination.

The same concept applies to IPv6, but it uses a 128-bit address instead of a 32-bit address.

---

## Assignment Example

This assignment uses **IPv4**.

Examples:

| Resource | IP Version Used |
|----------|-----------------|
| Laptop | IPv4 |
| Ubuntu EC2 | IPv4 |
| RHEL EC2 | IPv4 |
| Amazon Linux 2 | IPv4 |
| Amazon EFS | IPv4 |

The EC2 instances communicate using IPv4 addresses inside the VPC.

---

## Real DevOps Usage

A DevOps Engineer works with:

- IPv4 while designing VPCs.
- CIDR blocks.
- Public and Private IPs.
- Security Groups.
- Route Tables.

As organizations adopt IPv6, DevOps Engineers also configure dual-stack (IPv4 + IPv6) environments.

---

## Production Usage

Many production environments today use:

- IPv4 only
- Dual Stack (IPv4 + IPv6)

AWS supports IPv6 for many services, allowing applications to communicate over both protocols when required.

---

## Troubleshooting

### Problem

Application is not reachable over IPv6.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| IPv6 not enabled | The VPC or subnet does not have an IPv6 CIDR block. |
| Security Group | IPv6 traffic is blocked. |
| Route Table | Missing IPv6 route. |
| Internet Gateway | IPv6 connectivity is not configured. |

---

## Interview Tip

Knowing the address length is important.

- IPv4 → **32 Bits**
- IPv6 → **128 Bits**

This is one of the most frequently asked networking interview questions.

---

## Common Interview Mistakes

### Mistake 1

❌ IPv6 is replacing IPv4 everywhere today.

✅ Many organizations still use IPv4 or Dual Stack environments.

---

### Mistake 2

❌ IPv6 is faster than IPv4.

✅ IPv6 was designed primarily to solve address exhaustion and improve networking capabilities, not simply to increase speed.

---

### Mistake 3

❌ AWS only supports IPv4.

✅ AWS supports both IPv4 and IPv6 for many networking services.

---

## Interview Cross Questions

### Q1. Why was IPv6 introduced?

**Answer:**

To overcome IPv4 address exhaustion and support the growing number of Internet-connected devices.

---

### Q2. How many bits are in an IPv4 address?

**Answer:**

32 bits.

---

### Q3. How many bits are in an IPv6 address?

**Answer:**

128 bits.

---

### Q4. Which IP version is used in this assignment?

**Answer:**

IPv4.

---

## Related Topics

- IP Address
- Public IP
- Private IP
- CIDR
- VPC
- Route Table

---

## One-line Interview Answer

> IPv4 and IPv6 are two versions of the Internet Protocol used to identify devices and route data over networks, with IPv6 providing a much larger address space than IPv4.
---

---

# Q6. What is a MAC Address (Media Access Control Address)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Media Access Control Address |
| **Category** | Computer Networking |
| **Type** | Physical Address |
| **Hardware / Software / Protocol** | Hardware Address |
| **TCP/IP Layer** | Network Access Layer |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes (Indirectly through the EC2 Network Interface) |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- IPv4 vs IPv6

---

## Definition

A **MAC (Media Access Control) Address** is a **unique physical address** assigned to a **Network Interface Card (NIC)**.

Unlike an IP Address, which identifies the location of a device on a network, a MAC Address identifies the **network interface (hardware)** itself.

Every physical or virtual network interface has its own MAC Address.

---

## Why Do We Need a MAC Address?

When devices communicate within the same local network, they must know **which network interface should receive the data**.

The MAC Address provides this unique hardware-level identification.

Without MAC Addresses:

- Devices on the same network cannot identify each other.
- Local communication would not be possible.
- Ethernet and Wi-Fi communication would fail.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Identifies a Network Interface |
| **Address Type** | Physical Address |
| **Assigned By** | Manufacturer (or Virtualization Platform) |
| **TCP/IP Layer** | Network Access Layer |
| **Example** | `00:1A:2B:3C:4D:5E` |

---

## IP Address vs MAC Address

| Feature | IP Address | MAC Address |
|---------|------------|-------------|
| Full Form | Internet Protocol Address | Media Access Control Address |
| Type | Logical Address | Physical Address |
| Layer | Internet Layer | Network Access Layer |
| Purpose | Identifies a Device on a Network | Identifies a Network Interface |
| Can Change? | Yes | Usually No |
| Example | `10.0.1.25` | `00:1A:2B:3C:4D:5E` |

---

## How Does a MAC Address Work?

Suppose two computers are connected to the same network.

```text
Computer A
IP : 192.168.1.10
MAC: AA:AA:AA:AA:AA:AA
        │
        ▼
Local Network
        │
        ▼
Computer B
IP : 192.168.1.20
MAC: BB:BB:BB:BB:BB:BB
```

The **IP Address** identifies **which device** should receive the packet.

The **MAC Address** identifies **which network interface** on that local network should receive the packet.

---

## How It Works Internally

When data is sent over a local network:

1. The sender knows the destination IP Address.
2. It uses **ARP (Address Resolution Protocol)** to find the corresponding MAC Address.
3. The packet is delivered to the destination Network Interface using the MAC Address.

```text
Destination IP
      │
      ▼
ARP Lookup
      │
      ▼
Destination MAC Address
      │
      ▼
Data Transmission
```

---

## Assignment Example

In this assignment:

```text
Laptop
        │
Internet
        │
AWS Network
        │
Ubuntu EC2
        │
Elastic Network Interface (ENI)
        │
MAC Address
```

Although we never manually configure MAC Addresses, every EC2 instance communicates through an **Elastic Network Interface (ENI)**, and every ENI has its own MAC Address.

AWS manages these MAC Addresses automatically.

---

## Real DevOps Usage

A DevOps Engineer rarely configures MAC Addresses directly.

However, they are important when:

- Troubleshooting network connectivity.
- Working with Virtual Machines.
- Configuring Load Balancers.
- Managing Elastic Network Interfaces (ENIs).
- Diagnosing duplicate network interfaces.

---

## Production Usage

MAC Addresses are commonly used by:

- Switches
- Virtual Machines
- Hypervisors
- Cloud Network Interfaces
- DHCP Servers
- ARP Communication

AWS automatically manages MAC Addresses for EC2 network interfaces.

---

## Troubleshooting

### Problem

An EC2 instance cannot communicate within the local network.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Network Interface Issue | ENI is detached or misconfigured. |
| ARP Resolution Failure | MAC Address cannot be resolved. |
| Duplicate MAC Address | Rare in AWS, but possible in virtual lab environments. |
| Operating System Issue | Network interface is down. |

---

## Interview Tip

Remember:

- **IP Address = Where is the device?**
- **MAC Address = Which network interface should receive the data?**

Interviewers commonly ask for this difference.

---

## Common Interview Mistakes

### Mistake 1

❌ IP Address and MAC Address are the same.

✅ IP identifies a device logically.

MAC identifies the network interface physically.

---

### Mistake 2

❌ MAC Address is used across the Internet.

✅ MAC Addresses are used only within the local network segment.

Routers forward packets using IP Addresses, not MAC Addresses.

---

### Mistake 3

❌ AWS users manually configure MAC Addresses.

✅ AWS automatically manages MAC Addresses for EC2 network interfaces.

---

## Interview Cross Questions

### Q1. Which address changes more frequently: IP or MAC?

**Answer:**

The **IP Address** can change.

The **MAC Address** usually remains the same for a network interface.

---

### Q2. Can two devices have the same MAC Address?

**Answer:**

They should not.

A MAC Address is intended to uniquely identify a network interface.

---

### Q3. Does an EC2 instance have a MAC Address?

**Answer:**

Yes.

Every EC2 network interface (ENI) has its own MAC Address managed by AWS.

---

### Q4. Which protocol maps an IP Address to a MAC Address?

**Answer:**

**ARP (Address Resolution Protocol)**.

---

## Related Topics

- TCP/IP Model
- IP Address
- IPv4
- Elastic Network Interface (ENI)
- ARP
- VPC

---

## One-line Interview Answer

> A MAC (Media Access Control) Address is a unique physical address assigned to a network interface that enables communication within a local network.

---

---

# Q7. What is the Difference Between TCP (Transmission Control Protocol) and UDP (User Datagram Protocol)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | TCP - Transmission Control Protocol<br>UDP - User Datagram Protocol |
| **Category** | Computer Networking |
| **Type** | Transport Layer Protocol |
| **Hardware / Software / Protocol** | Software Protocol |
| **TCP/IP Layer** | Transport Layer |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ TCP |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- IPv4 vs IPv6
- MAC Address

---

## Definition

**TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)** are transport layer protocols that define **how data is transferred between two devices**.

- **TCP** provides **reliable, ordered, and error-checked** communication.
- **UDP** provides **fast, connectionless** communication without guaranteeing delivery.

---

## Why Do We Need TCP and UDP?

After the **Internet Layer** identifies the destination using an IP Address, the **Transport Layer** determines **how** the data should be delivered.

Different applications have different requirements.

For example:

- SSH requires reliable communication.
- File transfers require reliable communication.
- Video streaming prioritizes speed over guaranteed delivery.

TCP and UDP are designed to meet these different requirements.

---

## Quick Revision

| Feature | TCP | UDP |
|---------|-----|-----|
| **Full Form** | Transmission Control Protocol | User Datagram Protocol |
| **Connection Type** | Connection-Oriented | Connectionless |
| **Reliable** | ✅ Yes | ❌ No |
| **Packet Ordering** | ✅ Maintained | ❌ Not Guaranteed |
| **Error Checking** | ✅ Yes | Limited |
| **Speed** | Slower | Faster |
| **Acknowledgement** | Required | Not Required |
| **Typical Usage** | SSH, HTTP, HTTPS, FTP, NFS | DNS, VoIP, Live Streaming, Online Gaming |

---

## How Does TCP Work?

Before sending data, TCP establishes a connection between the sender and receiver.

This process is called the **Three-Way Handshake**.

```text
Client
   │
   │ SYN
   ▼
Server
   ▲
SYN + ACK
   │
   │
ACK
   ▼

Connection Established
```

After the connection is established:

- Data is sent.
- Every packet is acknowledged.
- Lost packets are retransmitted.
- Data arrives in the correct order.

---

## How Does UDP Work?

UDP does **not** establish a connection.

The sender immediately sends the data.

```text
Sender
   │
   ▼
Receiver
```

No:

- Connection setup
- Acknowledgement
- Retransmission
- Packet ordering

This makes UDP much faster than TCP.

---

## How It Works Internally

Suppose you connect to Ubuntu EC2 using SSH.

```text
SSH
 │
 ▼
TCP
 │
 ▼
Port 22
 │
 ▼
Ubuntu EC2
```

TCP first establishes a connection.

Only after the connection is established does SSH start transmitting data.

---

Suppose you mount Amazon EFS.

```text
NFS
 │
 ▼
TCP
 │
 ▼
Port 2049
 │
 ▼
Amazon EFS
```

Again,

TCP guarantees that file data reaches Amazon EFS correctly.

---

## Why Does This Assignment Use TCP?

This assignment requires reliable communication.

| Operation | Protocol | Reason |
|-----------|----------|--------|
| SSH Login | TCP | Commands must arrive correctly. |
| Amazon EFS (NFS) | TCP | File data cannot be lost or arrive out of order. |

If UDP were used, files could become corrupted because packets may be lost or arrive in the wrong order.

---

## Assignment Example

```text
Laptop
     │
SSH
     │
TCP
     │
Port 22
     │
Ubuntu EC2
     │
NFS
     │
TCP
     │
Port 2049
     │
Amazon EFS
```

TCP is used throughout this assignment.

---

## Real DevOps Usage

A DevOps Engineer works with TCP every day.

Examples:

- SSH into Linux servers
- Access web applications
- Connect applications to databases
- Mount Amazon EFS
- Configure Load Balancers
- Monitor application connectivity

Understanding TCP helps diagnose:

- Connection failures
- Packet loss
- Timeout issues
- Slow application responses

---

## Production Usage

Production applications commonly use TCP because reliability is critical.

Examples:

| Service | Protocol |
|----------|----------|
| SSH | TCP |
| HTTP | TCP |
| HTTPS | TCP |
| MySQL | TCP |
| PostgreSQL | TCP |
| Amazon EFS (NFS) | TCP |

UDP is preferred where speed is more important than guaranteed delivery, such as DNS queries, voice calls, and live video streaming.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Port `22` blocked | Security Group or Network ACL blocks TCP Port `22`. |
| SSH service stopped | SSH daemon is not running. |
| Network issue | TCP connection cannot be established. |
| Incorrect Public IP | Traffic reaches the wrong destination. |

---

### Problem

Unable to mount Amazon EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Port `2049` blocked | TCP communication is blocked. |
| NFS service issue | NFS client package is missing. |
| Mount Target issue | Amazon EFS Mount Target is unavailable. |

---

## Interview Tip

Remember:

- **TCP = Reliable Communication**
- **UDP = Fast Communication**

Whenever reliability is important, TCP is usually the preferred protocol.

---

## Common Interview Mistakes

### Mistake 1

❌ TCP is always better than UDP.

✅ TCP and UDP are designed for different use cases.

---

### Mistake 2

❌ UDP is unreliable because it is a poor protocol.

✅ UDP intentionally sacrifices reliability to achieve lower latency and higher speed.

---

### Mistake 3

❌ Amazon EFS uses UDP.

✅ Amazon EFS uses **NFS over TCP**.

---

## Interview Cross Questions

### Q1. Why does SSH use TCP instead of UDP?

**Answer:**

SSH requires reliable communication.

Commands must reach the server correctly and in the correct order.

---

### Q2. Why does Amazon EFS use TCP?

**Answer:**

Amazon EFS transfers file data.

TCP ensures reliable, ordered, and error-checked communication.

---

### Q3. Which protocol is faster?

**Answer:**

UDP is faster because it does not establish a connection or retransmit lost packets.

---

### Q4. Which protocol is used in this assignment?

**Answer:**

TCP.

Both SSH and Amazon EFS (NFS) use TCP.

---

## Related Topics

- TCP/IP Model
- Ports
- Firewall
- Security Group
- Network ACL
- SSH
- NFS

---

## One-line Interview Answer

> TCP (Transmission Control Protocol) provides reliable, connection-oriented communication, while UDP (User Datagram Protocol) provides fast, connectionless communication without guaranteeing packet delivery.

---

---

# Q8. What are Ports?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Port (No Full Form) |
| **Category** | Computer Networking |
| **Type** | Logical Communication Endpoint |
| **Hardware / Software / Protocol** | Software / Logical Identifier |
| **TCP/IP Layer** | Transport Layer |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- TCP vs UDP

---

## Definition

A **Port** is a **logical communication endpoint** used by the **Transport Layer (TCP/UDP)** to identify **which application or service** should receive incoming data on a device.

Think of an **IP Address** as the address of a building, and a **Port** as the specific room or department inside that building.

Without ports, the operating system would not know which application should receive the incoming data.

---

## Why Do We Need Ports?

A single computer can run multiple network services simultaneously.

For example, one EC2 instance can run:

- SSH
- Apache Web Server
- NGINX
- MySQL Database
- Docker Containers

All of these services use the same IP Address.

Ports allow the operating system to identify the correct service for incoming traffic.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Identifies the application or service receiving data |
| **Works With** | TCP and UDP |
| **TCP/IP Layer** | Transport Layer |
| **Range** | `0 - 65535` |
| **Used By** | Applications and Network Services |

---

## Port Range

Ports are divided into three categories.

| Port Range | Name | Usage |
|------------|------|-------|
| `0 - 1023` | Well-Known Ports | Standard services like SSH, HTTP, HTTPS |
| `1024 - 49151` | Registered Ports | Applications and vendor-specific services |
| `49152 - 65535` | Dynamic / Ephemeral Ports | Temporary client-side connections |

---

## Common Ports

| Port | Protocol | Service | Used in This Assignment |
|------|----------|----------|:-----------------------:|
| `22` | TCP | SSH (Secure Shell) | ✅ |
| `80` | TCP | HTTP (Hypertext Transfer Protocol) | ❌ |
| `443` | TCP | HTTPS (Hypertext Transfer Protocol Secure) | ❌ |
| `2049` | TCP | NFS (Network File System) | ✅ |
| `3306` | TCP | MySQL | ❌ |
| `5432` | TCP | PostgreSQL | ❌ |
| `3389` | TCP | RDP (Remote Desktop Protocol) | ❌ |
| `53` | UDP/TCP | DNS (Domain Name System) | ❌ |

---

## How Do Ports Work?

Suppose an EC2 instance has the following services running.

```text
Ubuntu EC2
IP : 54.xx.xx.xx

│
├── Port 22   → SSH
├── Port 80   → HTTP
├── Port 443  → HTTPS
└── Port 2049 → NFS
```

Although all services use the **same IP Address**, each service listens on a different **Port Number**.

This allows multiple applications to operate simultaneously on the same machine.

---

## How It Works Internally

Suppose you execute:

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

The packet contains:

```text
Destination IP
54.xx.xx.xx

Destination Port
22
```

AWS delivers the packet to the EC2 instance.

The Linux operating system checks:

```text
Port 22

↓

SSH Service Running?

↓

Yes

↓

Deliver packet to SSH Server
```

Now consider mounting Amazon EFS.

```bash
sudo mount -t nfs4 ...
```

The packet contains:

```text
Destination Port

2049
```

Linux delivers the packet to the **NFS client**, which communicates with Amazon EFS over **Port 2049**.

---

## Assignment Example

This assignment uses two important ports.

| Port | Service | Why Is It Needed? |
|------|----------|-------------------|
| `22` | SSH | To remotely connect from your laptop to the EC2 instances. |
| `2049` | NFS | To allow the EC2 instances to mount and communicate with Amazon EFS. |

---

## Real DevOps Usage

Ports are used daily by DevOps Engineers.

Examples:

- Opening SSH access (`22`)
- Hosting websites (`80`, `443`)
- Database connectivity (`3306`, `5432`)
- Monitoring tools (Prometheus, Grafana)
- Kubernetes services
- Docker containers
- CI/CD tools such as Jenkins

Understanding ports is essential for configuring firewalls and troubleshooting connectivity issues.

---

## Production Usage

Production environments usually expose only the required ports.

Example:

```text
Internet

↓

Load Balancer

↓

443 (HTTPS)

↓

Application Server

↓

3306 (MySQL)

↓

Database
```

Only necessary ports are opened.

Unused ports remain closed to improve security.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Port `22` blocked | Security Group or Network ACL blocks SSH. |
| SSH service stopped | SSH daemon is not running. |
| OS Firewall | Linux Firewall blocks Port `22`. |

---

### Problem

Unable to mount Amazon EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Port `2049` blocked | NFS traffic is blocked. |
| Security Group | NFS port is not allowed. |
| Network ACL | Port `2049` is denied. |

---

## Interview Tip

Remember:

- **IP Address identifies the device.**
- **Port identifies the application or service.**

Both are required for successful communication.

---

## Common Interview Mistakes

### Mistake 1

❌ Ports are physical connectors.

✅ Ports are logical communication endpoints used by the operating system.

---

### Mistake 2

❌ One IP Address can support only one application.

✅ A single IP Address can support thousands of applications using different ports.

---

### Mistake 3

❌ Opening Port `22` automatically allows SSH access.

✅ The SSH service must also be running, and Security Groups, Network ACLs, Route Tables, and Firewalls must permit the traffic.

---

## Interview Cross Questions

### Q1. Can two applications use the same port simultaneously?

**Answer:**

Generally, no.

Only one application can listen on a specific port for a given IP Address and protocol combination.

---

### Q2. Why does SSH use Port `22`?

**Answer:**

Port `22` is the well-known port assigned to the SSH protocol.

---

### Q3. Why does Amazon EFS use Port `2049`?

**Answer:**

Amazon EFS uses the **NFS (Network File System)** protocol, which communicates over TCP Port `2049`.

---

### Q4. Can an application use a custom port instead of the default port?

**Answer:**

Yes.

Many applications can be configured to listen on custom ports, although standard ports are commonly used for consistency.

---

## Related Topics

- TCP vs UDP
- Firewall
- Security Group
- Network ACL
- SSH
- NFS
- Amazon EFS

---

## One-line Interview Answer

> A Port is a logical communication endpoint used by the Transport Layer to identify the specific application or service that should receive network traffic on a device.

---

---

# Q9. What is a Firewall?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Firewall |
| **Category** | Computer Networking / Security |
| **Type** | Security Component |
| **Hardware / Software / Protocol** | Can be Hardware, Software, or Cloud Managed |
| **TCP/IP Layer** | Primarily Transport Layer & Internet Layer |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- TCP vs UDP
- Ports

---

## Definition

A **Firewall** is a security system that **monitors, allows, or blocks incoming and outgoing network traffic** based on predefined security rules.

It acts as a security barrier between trusted and untrusted networks.

---

## Why Do We Need a Firewall?

Without a firewall:

- Anyone could attempt to access your server.
- Unauthorized users could connect to applications.
- Malicious traffic could reach your systems.
- Sensitive services such as databases could become publicly accessible.

A firewall protects systems by allowing only authorized network traffic.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Protects systems by filtering network traffic |
| **Works Using** | IP Addresses, Ports, Protocols, Rules |
| **Can Allow Traffic** | ✅ Yes |
| **Can Block Traffic** | ✅ Yes |
| **Types** | Hardware, Software, Cloud Firewall |

---

## Types of Firewalls

| Firewall Type | Hardware / Software | Example |
|--------------|---------------------|---------|
| **Hardware Firewall** | Hardware | Cisco ASA, Fortinet, Palo Alto |
| **Software Firewall** | Software | Windows Defender Firewall, UFW, firewalld |
| **Cloud Firewall** | Cloud Managed | AWS Security Group, AWS Network ACL |

---

## Windows Firewall

Windows includes a built-in software firewall called **Microsoft Defender Firewall**.

It can:

- Allow applications.
- Block applications.
- Allow or deny ports.
- Create inbound and outbound rules.

Example:

```text
Allow

TCP Port 22

↓

SSH Allowed
```

---

## Linux Firewall

Linux provides multiple firewall solutions.

| Firewall | Distribution |
|-----------|-------------|
| **UFW (Uncomplicated Firewall)** | Ubuntu |
| **firewalld (Firewall Daemon)** | RHEL, CentOS, Amazon Linux |
| **iptables** | Linux Kernel Firewall Framework |

Examples:

Ubuntu

```bash
sudo ufw allow 22/tcp
```

RHEL / Amazon Linux

```bash
sudo firewall-cmd --add-service=ssh --permanent
```

---

## AWS Firewalls

AWS primarily provides two firewall mechanisms.

| Firewall | Level |
|-----------|-------|
| **Security Group** | Resource Level |
| **Network ACL (Network Access Control List)** | Subnet Level |

These will be covered in detail later.

---

## How Does a Firewall Work?

Suppose someone attempts to connect to an EC2 instance.

```text
Internet
      │
      ▼
Firewall
      │
      ├── Rule Matches?
      │
      ├── Yes → Allow
      │
      └── No → Block
      ▼
EC2
```

The firewall checks:

- Source IP
- Destination IP
- Protocol (TCP/UDP)
- Port Number

It then decides whether to allow or deny the traffic.

---

## How It Works Internally

Suppose you connect to Ubuntu EC2 using SSH.

```text
Laptop

↓

TCP

↓

Port 22

↓

Firewall

↓

Allow?

↓

Ubuntu EC2
```

If the firewall allows TCP Port `22`, the connection succeeds.

Otherwise, the connection is blocked before reaching the EC2 instance.

---

## Assignment Example

This assignment uses firewalls at multiple levels.

| Component | Firewall Used | Purpose |
|-----------|---------------|---------|
| Ubuntu EC2 | Security Group | Allows SSH (`22`) |
| Amazon EFS | Security Group | Allows NFS (`2049`) |
| Ubuntu EC2 | Linux Firewall (Optional) | Can allow or block SSH/NFS |
| VPC Subnet | Network ACL (Default) | Controls subnet traffic |

---

## Real DevOps Usage

Firewalls are configured daily by DevOps Engineers.

Examples:

- Allow HTTPS (`443`) for web applications.
- Allow SSH (`22`) only from trusted IP addresses.
- Block unnecessary ports.
- Protect databases from Internet access.
- Restrict internal application communication.

---

## Production Usage

Production environments follow the **Principle of Least Privilege**.

Instead of:

```text
Allow

All Ports

All IPs
```

They use:

```text
Allow

HTTPS (443)

From Internet

↓

Allow

SSH (22)

Only from Bastion Host or Corporate VPN

↓

Allow

Database Port

Only from Application Servers
```

Only the required traffic is permitted.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Security Group | Port `22` blocked. |
| Network ACL | SSH traffic denied. |
| Linux Firewall | Port `22` blocked. |
| Windows Firewall | Port blocked (Windows EC2). |
| SSH Service | SSH daemon is not running. |

---

### Problem

Unable to mount Amazon EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Security Group | Port `2049` blocked. |
| Network ACL | NFS traffic denied. |
| Linux Firewall | Port `2049` blocked. |

---

## Interview Tip

A firewall does **not** create a network.

It **protects** an existing network by filtering traffic.

---

## Common Interview Mistakes

### Mistake 1

❌ Firewall and Security Group are the same.

✅ A Security Group is an AWS-managed virtual firewall.

A firewall is a broader security concept.

---

### Mistake 2

❌ Firewalls only block traffic.

✅ Firewalls both **allow** and **block** traffic based on configured rules.

---

### Mistake 3

❌ Every firewall is hardware.

✅ Firewalls can be:

- Hardware
- Software
- Cloud Managed

---

## Interview Cross Questions

### Q1. Is a Security Group a Firewall?

**Answer:**

Yes.

A Security Group is an AWS-managed virtual firewall that protects AWS resources.

---

### Q2. Is a Network ACL a Firewall?

**Answer:**

Yes.

A Network ACL is an AWS-managed subnet-level firewall.

---

### Q3. Can Linux have its own firewall even if AWS Security Groups are configured?

**Answer:**

Yes.

Traffic must pass both the AWS firewall and the operating system firewall.

---

### Q4. Which firewall is checked first?

**Answer:**

It depends on where the traffic is coming from.

For AWS networking, incoming traffic generally encounters the **Network ACL (subnet level)** before reaching the **Security Group (resource level)**. If an operating system firewall (such as `ufw` or `firewalld`) is enabled, it is checked after the packet reaches the EC2 instance.

---

## Related Topics

- TCP vs UDP
- Ports
- Security Group
- Network ACL
- VPC
- Route Table

---

## One-line Interview Answer

> A Firewall is a security system that monitors and filters network traffic by allowing or blocking connections based on predefined rules.

---

---

# Q10. What is AWS Global Infrastructure?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Amazon Web Services Global Infrastructure |
| **Category** | AWS Cloud |
| **Type** | Cloud Infrastructure |
| **Hardware / Software / Protocol** | Global Physical Infrastructure Managed by AWS |
| **TCP/IP Layer** | Not Applicable |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- Firewall

---

## Definition

**AWS Global Infrastructure** is the worldwide network of **physical data centers and networking resources** owned and managed by Amazon Web Services (AWS).

It provides the physical foundation on which AWS services such as **Amazon EC2**, **Amazon EFS**, **Amazon RDS**, **Amazon S3**, and many others operate.

When you create any AWS resource, it is ultimately hosted in this global infrastructure.

---

## Why Do We Need AWS Global Infrastructure?

AWS customers are located all over the world.

To provide:

- Low Latency
- High Availability
- Fault Tolerance
- Disaster Recovery
- Compliance with Regional Regulations

AWS operates data centers across multiple geographic locations.

---

## Quick Revision

| Component | Purpose |
|-----------|---------|
| **Region** | Geographic location containing multiple Availability Zones |
| **Availability Zone (AZ)** | One or more physically separate data centers within a Region |
| **Edge Location** | Delivers content closer to users using services like CloudFront |
| **Local Zone** | Extends AWS services closer to large population centers |
| **Wavelength Zone** | Brings AWS services closer to 5G networks |

---

## AWS Global Infrastructure Overview

```text
AWS Global Infrastructure

│

├── Region
│      │
│      ├── Availability Zone A
│      ├── Availability Zone B
│      └── Availability Zone C
│
├── Region
│      │
│      ├── Availability Zone A
│      ├── Availability Zone B
│      └── Availability Zone C
│
└── Region
       │
       ├── Availability Zone A
       ├── Availability Zone B
       └── Availability Zone C
```

---

## Components of AWS Global Infrastructure

| Component | Description |
|-----------|-------------|
| **Region** | A separate geographic location containing multiple Availability Zones |
| **Availability Zone (AZ)** | One or more physically isolated data centers within a Region |
| **Edge Location** | Used for caching and delivering content closer to end users |
| **Local Zone** | Extends AWS services to metropolitan areas with low latency requirements |
| **Wavelength Zone** | Integrates AWS services with telecommunications 5G networks |

---

## How It Works Internally

When you create an EC2 instance, AWS does not place it randomly.

The placement follows this hierarchy:

```text
AWS Global Infrastructure

↓

Choose Region

↓

Choose Availability Zone

↓

Choose VPC

↓

Choose Subnet

↓

Launch EC2
```

Every AWS resource follows this hierarchy.

---

## Assignment Example

For this assignment:

```text
AWS Global Infrastructure

↓

Mumbai Region
(ap-south-1)

↓

Availability Zone
(ap-south-1a)

↓

VPC

↓

Public Subnet

↓

Ubuntu EC2

↓

Amazon EFS
```

All resources are created inside a specific AWS Region and Availability Zone.

---

## Real DevOps Usage

A DevOps Engineer considers AWS Global Infrastructure when:

- Designing highly available applications.
- Deploying resources across multiple Availability Zones.
- Planning disaster recovery.
- Reducing latency by selecting the nearest Region.
- Meeting data residency and compliance requirements.

---

## Production Usage

Production applications commonly use:

- Multiple Availability Zones for High Availability.
- Multiple Regions for Disaster Recovery.
- Edge Locations for faster content delivery.
- Local Zones for latency-sensitive applications.

This improves resilience and user experience.

---

## Troubleshooting

### Problem

Application is experiencing high latency.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong Region | Resources are deployed far from end users. |
| Single Availability Zone | No redundancy if the AZ becomes unavailable. |
| Missing Edge Locations | Static content is served from a distant Region. |

---

## Interview Tip

Remember the hierarchy:

```text
AWS Global Infrastructure

↓

Region

↓

Availability Zone

↓

VPC

↓

Subnet

↓

AWS Resource
```

Interviewers frequently ask where a VPC or EC2 instance exists within AWS.

---

## Common Interview Mistakes

### Mistake 1

❌ AWS Global Infrastructure consists only of Regions.

✅ It also includes Availability Zones, Edge Locations, Local Zones, and Wavelength Zones.

---

### Mistake 2

❌ A Region contains only one Availability Zone.

✅ Every AWS Region contains multiple Availability Zones.

---

### Mistake 3

❌ AWS resources are automatically distributed across multiple Regions.

✅ Resources are created only in the Region you select unless you explicitly deploy them elsewhere.

---

## Interview Cross Questions

### Q1. What is AWS Global Infrastructure?

**Answer:**

It is the worldwide network of AWS Regions, Availability Zones, Edge Locations, Local Zones, and Wavelength Zones that host AWS services.

---

### Q2. Where are EC2 instances created?

**Answer:**

EC2 instances are created inside a specific **Availability Zone** within a selected **AWS Region**.

---

### Q3. Does every AWS service use AWS Global Infrastructure?

**Answer:**

Yes.

All AWS services run on AWS Global Infrastructure, although the underlying components they use may differ.

---

## Related Topics

- AWS Region
- Availability Zone
- VPC
- Subnet
- Amazon EC2
- Amazon EFS

---

## One-line Interview Answer

> AWS Global Infrastructure is the worldwide network of Regions, Availability Zones, and other infrastructure components that host and deliver AWS services.

---

---

# Q11. What is an AWS Region?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | AWS Region |
| **Category** | AWS Global Infrastructure |
| **Type** | Geographic Location |
| **Hardware / Software / Protocol** | Physical Infrastructure (Managed by AWS) |
| **TCP/IP Layer** | Not Applicable |
| **Interview Level** | 🟢 Basic |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- AWS Global Infrastructure

---

## Definition

An **AWS Region** is a **geographic location** that contains **multiple Availability Zones (AZs)**.

Each Region is physically separated from other Regions and operates independently.

Examples:

- ap-south-1 (Mumbai)
- us-east-1 (N. Virginia)
- eu-west-1 (Ireland)

---

## Why Do We Need AWS Regions?

AWS Regions help provide:

- Low Latency
- High Availability
- Disaster Recovery
- Regulatory Compliance
- Data Residency

Instead of hosting all AWS resources in one location, AWS distributes them across multiple Regions worldwide.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Geographic location for deploying AWS resources |
| **Contains** | Multiple Availability Zones |
| **Managed By** | AWS |
| **Example** | `ap-south-1` (Mumbai) |

---

## Some Common AWS Regions

| Region Name | Region Code | Location |
|-------------|-------------|----------|
| Mumbai | `ap-south-1` | India |
| Hyderabad | `ap-south-2` | India |
| N. Virginia | `us-east-1` | USA |
| Ohio | `us-east-2` | USA |
| Ireland | `eu-west-1` | Europe |
| Singapore | `ap-southeast-1` | Singapore |

---

## How Does an AWS Region Work?

```text
AWS Global Infrastructure

        │

        ▼

Region (Mumbai)

        │

        ├── Availability Zone A

        ├── Availability Zone B

        └── Availability Zone C
```

A Region is an independent geographical area.

Resources created in one Region do **not** automatically exist in another Region.

---

## How It Works Internally

When you launch an AWS resource, AWS first asks you to choose a Region.

```text
Choose AWS Region

↓

Choose Availability Zone

↓

Create VPC

↓

Create Subnet

↓

Launch EC2
```

The selected Region determines where the resource will physically reside.

---

## Assignment Example

For this assignment, we selected:

```text
AWS Region

↓

Mumbai

(ap-south-1)

↓

Availability Zone

↓

VPC

↓

EC2

↓

Amazon EFS
```

All resources must be created within the same Region because Amazon EFS cannot be mounted across Regions.

---

## Real DevOps Usage

A DevOps Engineer selects a Region based on:

- Application users
- Compliance requirements
- Service availability
- Cost
- Disaster recovery strategy

---

## Production Usage

Production applications often use:

- One Region for primary deployment.
- Multiple Availability Zones for High Availability.
- A second Region for Disaster Recovery (DR).

Example:

```text
Primary

Mumbai

↓

Backup

Singapore
```

---

## Troubleshooting

### Problem

Unable to attach or connect AWS resources.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong Region | Resources are created in different Regions. |
| Service Availability | Required AWS service is unavailable in the selected Region. |
| Incorrect AWS Console Region | Looking in the wrong Region. |

---

## Interview Tip

Always verify the selected Region before troubleshooting.

Many "missing resource" issues occur because the AWS Console is set to a different Region.

---

## Common Interview Mistakes

### Mistake 1

❌ A Region contains only one Availability Zone.

✅ Every AWS Region contains multiple Availability Zones.

---

### Mistake 2

❌ Resources automatically replicate across Regions.

✅ Resources remain in the selected Region unless replication is explicitly configured.

---

### Mistake 3

❌ An EC2 instance in one Region can directly mount an Amazon EFS file system in another Region.

✅ Amazon EFS is Regional and can only be mounted by resources within the same Region.

---

## Interview Cross Questions

### Q1. Can two VPCs exist in different Regions?

**Answer:**

Yes.

Every Region can contain multiple VPCs.

---

### Q2. Can one VPC span multiple Regions?

**Answer:**

No.

A VPC belongs to a single AWS Region.

---

### Q3. Which Region was used in this assignment?

**Answer:**

`ap-south-1` (Mumbai).

---

### Q4. Why is Mumbai commonly selected for users in India?

**Answer:**

Because it provides lower network latency for users located in India compared to distant Regions.

---

## Related Topics

- AWS Global Infrastructure
- Availability Zone
- VPC
- Subnet
- Amazon EC2
- Amazon EFS

---

## One-line Interview Answer

> An AWS Region is a geographic location containing multiple Availability Zones where AWS resources are deployed.

---

---

# Q12. What is an Availability Zone (AZ)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Availability Zone |
| **Category** | AWS Global Infrastructure |
| **Type** | Physical Infrastructure |
| **Hardware / Software / Protocol** | One or More Physically Separate Data Centers Managed by AWS |
| **TCP/IP Layer** | Not Applicable |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- AWS Global Infrastructure
- AWS Region

---

## Definition

An **Availability Zone (AZ)** is **one or more physically separate data centers** within an AWS Region.

Each AZ has:

- Independent Power Supply
- Independent Cooling
- Independent Physical Security
- Independent Networking

At the same time, all AZs within a Region are connected using **high-speed, low-latency private networks**.

---

## Why Do We Need Availability Zones?

If an entire data center fails because of:

- Power Failure
- Network Failure
- Fire
- Flood
- Hardware Failure

applications running in another Availability Zone continue working.

Availability Zones provide:

- High Availability
- Fault Tolerance
- Business Continuity

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Isolates failures within an AWS Region |
| **Contains** | One or More Data Centers |
| **Managed By** | AWS |
| **Example** | `ap-south-1a`, `ap-south-1b`, `ap-south-1c` |

---

## AWS Region vs Availability Zone

| AWS Region | Availability Zone |
|------------|-------------------|
| Geographic Location | Physical Data Center(s) inside a Region |
| Contains Multiple AZs | Belongs to One Region |
| Example: Mumbai (`ap-south-1`) | Example: `ap-south-1a` |

---

## How Does an Availability Zone Work?

```text
AWS Region (Mumbai)

│

├── Availability Zone A
│      ├── EC2
│      ├── EFS Mount Target
│      └── Subnet
│
├── Availability Zone B
│      ├── EC2
│      ├── EFS Mount Target
│      └── Subnet
│
└── Availability Zone C
       ├── EC2
       ├── EFS Mount Target
       └── Subnet
```

Every AWS resource is deployed inside an Availability Zone.

---

## How It Works Internally

When launching an EC2 instance:

```text
Choose Region

↓

Choose Availability Zone

↓

Choose VPC

↓

Choose Subnet

↓

Launch EC2
```

The selected subnet automatically determines the Availability Zone because **a subnet belongs to exactly one Availability Zone**.

---

## How Availability Zones Affect AWS Resources

| AWS Resource | How Availability Zone Affects It |
|--------------|----------------------------------|
| **VPC** | Spans all Availability Zones within a Region. |
| **Subnet** | Belongs to exactly one Availability Zone. |
| **EC2** | Runs in one Availability Zone. |
| **Amazon EFS** | Creates Mount Targets in selected Availability Zones. |
| **Amazon RDS** | Uses multiple Availability Zones for Multi-AZ deployments. |
| **Application Load Balancer (ALB)** | Can span multiple Availability Zones for High Availability. |
| **Auto Scaling Group (ASG)** | Launches EC2 instances across multiple Availability Zones. |

---

## Assignment Example

For this assignment:

```text
AWS Region

ap-south-1

↓

Availability Zone

ap-south-1a

↓

Subnet

↓

Ubuntu EC2

↓

Amazon EFS Mount Target
```

All EC2 instances and Amazon EFS Mount Targets should be created in compatible Availability Zones for proper communication.

---

## Real DevOps Usage

A DevOps Engineer uses multiple Availability Zones to:

- Improve High Availability.
- Reduce downtime.
- Deploy fault-tolerant applications.
- Configure Auto Scaling.
- Configure Load Balancers.
- Deploy highly available databases.

---

## Production Usage

A production architecture typically looks like this:

```text
Region

│

├── AZ-A
│      ├── EC2
│      └── ALB
│
├── AZ-B
│      ├── EC2
│      └── ALB
│
└── Amazon RDS (Multi-AZ)
```

If one Availability Zone becomes unavailable, applications in the other Availability Zone continue serving users.

---

## Troubleshooting

### Problem

Unable to create an EC2 instance in a subnet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Capacity Issue | Selected Availability Zone has limited capacity. |
| Wrong Subnet | Subnet belongs to a different Availability Zone. |
| Resource Availability | The required instance type is unavailable in that Availability Zone. |

---

### Problem

Amazon EFS cannot be mounted.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing Mount Target | No Mount Target exists in the required Availability Zone. |
| Security Group | NFS traffic is blocked. |
| Incorrect Subnet | EC2 is launched in an unexpected Availability Zone. |

---

## Interview Tip

Remember:

- **VPC → Regional Resource**
- **Subnet → Availability Zone Resource**
- **EC2 → Availability Zone Resource**

This is one of the most commonly asked AWS networking interview questions.

---

## Common Interview Mistakes

### Mistake 1

❌ A subnet can span multiple Availability Zones.

✅ A subnet belongs to exactly one Availability Zone.

---

### Mistake 2

❌ A VPC belongs to one Availability Zone.

✅ A VPC spans all Availability Zones in a Region.

---

### Mistake 3

❌ An EC2 instance can move freely between Availability Zones.

✅ An EC2 instance is launched into a specific Availability Zone. To use another Availability Zone, you typically launch a new instance there (or create an AMI and launch from it).

---

## Interview Cross Questions

### Q1. Can one subnet span multiple Availability Zones?

**Answer:**

No.

Every subnet belongs to exactly one Availability Zone.

---

### Q2. Can one VPC span multiple Availability Zones?

**Answer:**

Yes.

A VPC spans all Availability Zones within its Region.

---

### Q3. Why do companies use multiple Availability Zones?

**Answer:**

To achieve High Availability and Fault Tolerance.

---

### Q4. Which AWS resources are Availability Zone-specific?

**Answer:**

Examples include:

- EC2
- Subnets
- EBS Volumes
- Amazon EFS Mount Targets

---

## Related Topics

- AWS Global Infrastructure
- AWS Region
- VPC
- Subnet
- Amazon EC2
- Amazon EFS
- Auto Scaling
- Application Load Balancer

---

## One-line Interview Answer

> An Availability Zone (AZ) is one or more physically separate data centers within an AWS Region that provide fault isolation and high availability for AWS resources.

---

---

# Q13. What is a VPC (Virtual Private Cloud)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Virtual Private Cloud |
| **Category** | AWS Networking |
| **Type** | Virtual Network |
| **Hardware / Software / Protocol** | Software-Defined Virtual Network Managed by AWS |
| **TCP/IP Layer** | Internet Layer (Virtual Networking) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- AWS Global Infrastructure
- AWS Region
- Availability Zone

---

## Definition

A **VPC (Virtual Private Cloud)** is a **logically isolated virtual network** created inside an **AWS Region**, where you launch and manage AWS resources such as:

- Amazon EC2
- Amazon EFS
- Amazon RDS
- Application Load Balancer
- NAT Gateway

A VPC provides complete control over your network configuration, including:

- IP Address Range (CIDR)
- Subnets
- Route Tables
- Internet Connectivity
- Security

Think of a VPC as **your own private network inside the AWS Cloud**.

---

## Why Do We Need a VPC?

Imagine AWS without a VPC.

Every customer's resources would exist in one shared network.

Problems:

- Customers could potentially communicate with each other.
- IP address conflicts would occur.
- Security would be extremely difficult.
- Network customization would be impossible.

A VPC solves these problems by giving every customer an isolated private network.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Provides an isolated virtual network in AWS |
| **Scope** | Regional Resource |
| **Contains** | Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, EC2, EFS |
| **Managed By** | AWS |
| **Communication** | Uses private IP addresses inside the VPC |

---

## Why is it Called a "Virtual Private Cloud"?

Let's understand each word.

| Word | Meaning |
|------|---------|
| **Virtual** | Software-defined network created by AWS instead of physical networking hardware owned by you. |
| **Private** | Logically isolated so other AWS customers cannot directly access your network. |
| **Cloud** | Runs inside AWS's global cloud infrastructure instead of your own data center. |

---

## How Does a VPC Work?

```text
AWS Region

        │

        ▼

+--------------------------------------+
|               VPC                    |
|                                      |
|  +------------+   +---------------+  |
|  | Subnet A   |   | Subnet B      |  |
|  |            |   |               |  |
|  | Ubuntu EC2 |   | Amazon Linux  |  |
|  | RHEL EC2   |   |               |  |
|  +------------+   +---------------+  |
|                                      |
|          Amazon EFS                  |
+--------------------------------------+
```

Every AWS resource is launched **inside a subnet**, and every subnet belongs to a **VPC**.

---

## How It Works Internally

When you launch an EC2 instance, AWS performs the following steps:

```text
AWS Region

↓

VPC

↓

Subnet

↓

Private IP Assignment

↓

Network Interface (ENI)

↓

EC2 Instance
```

The VPC provides the networking environment in which all communication occurs.

---

## Components of a VPC

| Component | Purpose |
|-----------|---------|
| **CIDR Block** | Defines the IP address range of the VPC. |
| **Subnet** | Divides the VPC into smaller networks. |
| **Route Table** | Controls where network traffic is sent. |
| **Internet Gateway (IGW)** | Enables Internet connectivity for public resources. |
| **NAT Gateway** | Allows private resources to access the Internet without being directly reachable. |
| **Security Group** | Instance-level virtual firewall. |
| **Network ACL (NACL)** | Subnet-level virtual firewall. |

---

## Assignment Example

For this assignment, the architecture is:

```text
AWS Region

↓

VPC
10.0.0.0/16

↓

Public Subnet

↓

Ubuntu EC2

↓

Amazon Linux

↓

RHEL

↓

Amazon EFS
```

All three EC2 instances and Amazon EFS are created inside the same VPC so they can communicate using private IP addresses.

---

## Real DevOps Usage

A DevOps Engineer creates and manages VPCs to:

- Design secure network architectures.
- Separate environments (Development, Testing, Production).
- Control IP addressing.
- Connect applications and databases.
- Configure Internet access.
- Deploy highly available infrastructure.

---

## Production Usage

A typical production architecture:

```text
VPC

│

├── Public Subnet
│      ├── Application Load Balancer
│      └── NAT Gateway
│
├── Private Subnet
│      ├── Application EC2
│      └── ECS/EKS
│
└── Private Database Subnet
       └── Amazon RDS
```

The VPC provides the overall network, while subnets separate workloads based on their purpose and security requirements.

---

## Troubleshooting

### Problem

EC2 instances cannot communicate with Amazon EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Different VPCs | EC2 and EFS are created in different VPCs. |
| Security Group | Required traffic is blocked. |
| Route Table | Incorrect routing configuration. |
| Mount Target | EFS Mount Target is missing. |

---

### Problem

Cannot find an EC2 instance in the AWS Console.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong Region | The EC2 instance exists in another Region. |
| Wrong VPC | Looking in a different VPC. |

---

## Interview Tip

Remember the hierarchy:

```text
AWS Global Infrastructure

↓

Region

↓

VPC

↓

Subnet

↓

EC2
```

A VPC is **Regional**, while a Subnet belongs to **one Availability Zone**.

---

## Common Interview Mistakes

### Mistake 1

❌ A VPC provides security.

✅ A VPC provides networking.

Security is provided by Security Groups and Network ACLs.

---

### Mistake 2

❌ Resources are launched directly into a VPC.

✅ Resources are launched into a Subnet, and the Subnet belongs to the VPC.

---

### Mistake 3

❌ A VPC belongs to one Availability Zone.

✅ A VPC spans all Availability Zones within its Region.

---

### Mistake 4

❌ Every VPC automatically has Internet access.

✅ A VPC requires an Internet Gateway and appropriate Route Table configuration for Internet connectivity.

---

## Interview Cross Questions

### Q1. Can a VPC span multiple Availability Zones?

**Answer:**

Yes.

A VPC spans all Availability Zones within a single AWS Region.

---

### Q2. Can a VPC span multiple Regions?

**Answer:**

No.

A VPC is always limited to one AWS Region.

---

### Q3. Can you launch an EC2 instance directly into a VPC?

**Answer:**

No.

Every EC2 instance must be launched into a Subnet.

---

### Q4. Why is a VPC called "Virtual"?

**Answer:**

Because AWS creates and manages the network using software rather than requiring customers to own physical networking hardware.

---

### Q5. What is the main purpose of a VPC?

**Answer:**

To provide a logically isolated virtual network where AWS resources can communicate securely.

---

## Related Topics

- CIDR
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway
- Security Group
- Network ACL

---

## One-line Interview Answer

> A VPC (Virtual Private Cloud) is a logically isolated virtual network within an AWS Region that allows you to securely launch and manage AWS resources with full control over networking and IP addressing.

---

---

# Q14. What is CIDR (Classless Inter-Domain Routing)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Classless Inter-Domain Routing |
| **Category** | Computer Networking / AWS Networking |
| **Type** | IP Addressing Method |
| **Hardware / Software / Protocol** | Networking Standard |
| **TCP/IP Layer** | Internet Layer |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- IPv4
- VPC

---

## Definition

**CIDR (Classless Inter-Domain Routing)** is a method of allocating IP addresses and dividing networks using a **prefix length**.

Instead of using fixed network sizes, CIDR allows network administrators to create networks of different sizes based on actual requirements.

In AWS, every **VPC** and **Subnet** is assigned a CIDR block.

---

## Why Do We Need CIDR?

Without CIDR:

- IP addresses would be wasted.
- Networks could only be created in fixed sizes.
- Small and large organizations would receive the same predefined network sizes.

CIDR allows flexible IP allocation, reducing address wastage and improving network scalability.

---

## Why is it Called "Classless"?

Before CIDR, IPv4 used **Classful Addressing**.

Networks were divided into fixed classes.

| Class | Default Prefix | Number of IPs |
|--------|---------------|--------------:|
| A | `/8` | 16,777,216 |
| B | `/16` | 65,536 |
| C | `/24` | 256 |

Suppose a company needed **500 IP addresses**.

With Classful Addressing:

- Class C (`/24`) → Too small (256 IPs)
- Class B (`/16`) → Too large (65,536 IPs)

This caused significant IP address wastage.

CIDR removed these fixed classes.

Instead of choosing Class A, B, or C, we can now choose the exact network size required.

Hence the name **Classless Inter-Domain Routing**.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Defines the IP address range of a network |
| **Used In** | VPC, Subnets, Route Tables |
| **Notation** | `/16`, `/24`, `/28` |
| **Represents** | Number of Network Bits |

---

## CIDR Format

```
Network Address / Prefix Length
```

Example:

```
10.0.0.0/16
```

Where:

- `10.0.0.0` → Network Address
- `/16` → Prefix Length

---

## Understanding Prefix Length

IPv4 contains **32 bits**.

The prefix tells us how many bits belong to the **Network**.

The remaining bits belong to the **Host**.

| CIDR | Network Bits | Host Bits |
|------|-------------:|----------:|
| `/8` | 8 | 24 |
| `/16` | 16 | 16 |
| `/24` | 24 | 8 |
| `/28` | 28 | 4 |
| `/32` | 32 | 0 |

Formula:

```
Host Bits = 32 − Prefix Length
```

---

## CIDR Calculation

| CIDR | Host Bits | Formula | Total IP Addresses |
|------|----------:|---------|-------------------:|
| `/24` | 8 | 2⁸ | 256 |
| `/25` | 7 | 2⁷ | 128 |
| `/26` | 6 | 2⁶ | 64 |
| `/27` | 5 | 2⁵ | 32 |
| `/28` | 4 | 2⁴ | 16 |
| `/29` | 3 | 2³ | 8 |

Formula:

```
Total IPs = 2^(Host Bits)
```

---

## AWS Example

Suppose we create a VPC.

```
VPC

10.0.0.0/16
```

This means:

| Property | Value |
|-----------|------|
| Network Bits | 16 |
| Host Bits | 16 |
| Total IP Addresses | 65,536 |

Now divide it into subnets.

```
VPC

10.0.0.0/16

│

├── 10.0.1.0/24

├── 10.0.2.0/24

├── 10.0.3.0/24

└── 10.0.4.0/24
```

Each subnet now contains:

| CIDR | Total IPs |
|------|----------:|
| `/24` | 256 |

---

## AWS Reserved IP Addresses

AWS reserves **5 IP addresses** in every subnet.

Example:

```
10.0.1.0/24
```

| Reserved IP | Purpose |
|-------------|---------|
| `10.0.1.0` | Network Address |
| `10.0.1.1` | VPC Router |
| `10.0.1.2` | Amazon DNS |
| `10.0.1.3` | Reserved by AWS |
| `10.0.1.255` | Reserved |

Therefore,

```
256

− 5

=

251 Usable IP Addresses
```

---

## How It Works Internally

When an EC2 instance is launched:

```
VPC

10.0.0.0/16

↓

Subnet

10.0.1.0/24

↓

AWS assigns

10.0.1.10

↓

EC2
```

AWS always assigns the EC2 instance's **Private IP Address** from the **Subnet CIDR**, not directly from the VPC CIDR.

---

## Assignment Example

For this assignment:

```
VPC

10.0.0.0/16

↓

Public Subnet

10.0.1.0/24

↓

Ubuntu EC2

10.0.1.10

↓

Amazon EFS

10.0.1.50
```

All resources receive their Private IP addresses from the subnet's CIDR block.

---

## Real DevOps Usage

A DevOps Engineer uses CIDR to:

- Design VPCs.
- Plan Subnets.
- Avoid overlapping IP ranges.
- Configure VPNs.
- Connect multiple VPCs.
- Build scalable network architectures.

Poor CIDR planning can make future expansion difficult.

---

## Production Usage

Production environments usually allocate CIDR blocks with future growth in mind.

Example:

```
VPC

10.0.0.0/16

↓

Public

10.0.1.0/24

↓

Private App

10.0.10.0/24

↓

Database

10.0.20.0/24
```

This leaves unused ranges for future subnets.

---

## Troubleshooting

### Problem

Unable to create a new subnet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| CIDR Overlap | New subnet overlaps an existing subnet. |
| CIDR Outside VPC | Subnet CIDR is outside the VPC CIDR range. |
| No Available Address Space | Entire VPC CIDR has already been allocated. |

---

## Interview Tip

Remember these three formulas.

```
IPv4 = 32 Bits

Host Bits = 32 − Prefix

Total IPs = 2^(Host Bits)
```

These are among the most frequently asked CIDR interview questions.

---

## Common Interview Mistakes

### Mistake 1

❌ EC2 receives its Private IP from the VPC CIDR.

✅ EC2 receives its Private IP from the **Subnet CIDR**.

---

### Mistake 2

❌ `/24` means 24 IP addresses.

✅ `/24` means **24 Network Bits**, not 24 IP addresses.

---

### Mistake 3

❌ CIDR is an AWS feature.

✅ CIDR is a standard networking concept used worldwide. AWS uses it for VPCs and subnets.

---

## Interview Cross Questions

### Q1. Why was CIDR introduced?

**Answer:**

To replace Classful Addressing and reduce IP address wastage by allowing flexible network sizes.

---

### Q2. What does `/24` mean?

**Answer:**

The first 24 bits represent the network portion, leaving 8 bits for host addresses.

---

### Q3. Who assigns an EC2 instance's Private IP?

**Answer:**

AWS assigns it automatically from the **Subnet CIDR block**.

---

### Q4. Why does AWS reserve 5 IP addresses in every subnet?

**Answer:**

AWS reserves them for networking functions such as the network address, VPC router, Amazon DNS, future AWS use, and the last reserved address.

---

## Related Topics

- VPC
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway
- IPv4

---

## One-line Interview Answer

> CIDR (Classless Inter-Domain Routing) is a flexible IP addressing method that uses prefix lengths to define network sizes and allocate IP addresses efficiently.

---

---

# Q15. What is a Subnet?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Subnetwork |
| **Category** | AWS Networking |
| **Type** | Logical Network Division |
| **Hardware / Software / Protocol** | Software-Defined Virtual Network Segment |
| **TCP/IP Layer** | Internet Layer (Virtual Networking) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- VPC
- CIDR
- AWS Region
- Availability Zone

---

## Definition

A **Subnet (Subnetwork)** is a **logical division of a VPC**.

It divides the VPC's IP address range into **smaller, manageable networks** where AWS resources such as EC2 instances, Amazon EFS Mount Targets, Amazon RDS, and Load Balancers are deployed.

Every AWS resource is launched **inside a subnet**, not directly inside a VPC.

---

## Why Do We Need a Subnet?

A VPC provides the overall network, but placing every resource into one large network would make management difficult.

Subnets help us:

- Organize resources.
- Separate public and private workloads.
- Control routing.
- Improve security.
- Deploy resources across multiple Availability Zones.
- Scale infrastructure efficiently.

Think of a VPC as a **city** and subnets as **different neighborhoods** within that city.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Divides a VPC into smaller networks |
| **Belongs To** | One VPC |
| **Availability Zone** | Exactly One AZ |
| **Contains** | EC2, EFS Mount Targets, Load Balancers, RDS, Lambda ENIs, etc. |
| **IP Source** | Subnet CIDR Block |

---

## Relationship Between VPC and Subnet

```text
AWS Region

      │

      ▼

VPC
10.0.0.0/16

      │

      ├── 10.0.1.0/24

      ├── 10.0.2.0/24

      ├── 10.0.3.0/24

      └── 10.0.4.0/24
```

The VPC provides the overall network.

Subnets divide that network into smaller sections.

---

## Why Are Subnets Mandatory?

AWS requires every resource to be launched inside a subnet because networking decisions are made at the subnet level.

Subnets determine:

- IP Address Allocation
- Availability Zone Placement
- Route Table Association
- Internet Connectivity
- Security Design
- Service Placement

Without a subnet, AWS would not know:

- Which IP address to assign.
- Which Availability Zone to use.
- Which Route Table to follow.

---

## AWS Services That Depend on Subnets

| AWS Service / Feature | Why Does It Use a Subnet? | Interview Keyword |
|------------------------|---------------------------|-------------------|
| **IP Address Allocation** | Resources receive Private IPs from the subnet CIDR. | IP Assignment |
| **Availability Zone (AZ)** | Each subnet belongs to one AZ. | AZ Placement |
| **Route Table** | Routing rules are associated with subnets. | Routing |
| **Public / Private Network** | Determined by the subnet's Route Table. | Internet Access |
| **NAT Gateway** | Must be created in a Public Subnet. | Outbound Internet |
| **Amazon EFS Mount Target** | Each Mount Target exists in a subnet. | Network Endpoint |
| **Application Load Balancer (ALB)** | Uses one or more subnets. | High Availability |
| **Amazon RDS** | Uses DB Subnet Groups. | Database Placement |
| **AWS Lambda (VPC Mode)** | Creates ENIs inside selected subnets. | Private Connectivity |
| **Auto Scaling Group (ASG)** | Launches EC2 instances into specified subnets. | Scalability |
| **Amazon ECS / Amazon EKS** | Deploys workloads into selected subnets. | Container Networking |
| **VPC Endpoints** | Interface Endpoints are created inside subnets. | Private AWS Access |

---

## How Does a Subnet Work?

Suppose we create:

```text
VPC

10.0.0.0/16
```

Now divide it.

```text
VPC

10.0.0.0/16

│

├── Public Subnet

10.0.1.0/24

│

├── Private Subnet

10.0.2.0/24

│

└── Private Database Subnet

10.0.3.0/24
```

Each subnet has:

- Its own CIDR Block.
- Its own Route Table association.
- Its own Availability Zone.
- Its own IP Address range.

---

## How It Works Internally

When you launch an EC2 instance:

```text
Choose VPC

↓

Choose Subnet

↓

AWS picks an available IP

↓

Assigns Private IP

↓

Launches EC2
```

Notice that AWS assigns the EC2's Private IP from the **Subnet CIDR**, not directly from the VPC CIDR.

---

## Assignment Example

```text
VPC

10.0.0.0/16

↓

Public Subnet

10.0.1.0/24

↓

Ubuntu EC2

10.0.1.10

↓

Amazon Linux

10.0.1.20

↓

RHEL

10.0.1.30

↓

Amazon EFS Mount Target

10.0.1.50
```

All resources communicate using Private IP addresses within the subnet.

---

## Real DevOps Usage

A DevOps Engineer uses subnets to:

- Separate Development, Testing, and Production environments.
- Deploy applications across multiple Availability Zones.
- Separate public-facing resources from private resources.
- Design secure and scalable network architectures.

---

## Production Usage

A typical production VPC contains multiple subnet types.

```text
VPC

│

├── Public Subnet
│      ├── Load Balancer
│      └── NAT Gateway
│
├── Private Application Subnet
│      ├── EC2
│      └── ECS / EKS
│
└── Private Database Subnet
       └── Amazon RDS
```

This separation improves security, scalability, and availability.

---

## Troubleshooting

### Problem

Unable to launch an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| No Available IPs | The subnet CIDR has no free IP addresses. |
| Wrong Availability Zone | The selected subnet belongs to a different AZ. |
| Incorrect Route Table | Network connectivity issues after launch. |

---

### Problem

Amazon EFS cannot be mounted.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing Mount Target | No Mount Target exists in the subnet or AZ. |
| Security Group | NFS traffic blocked. |
| Incorrect Subnet | EC2 is in an unexpected subnet or AZ. |

---

## Interview Tip

Remember:

**VPC provides the network.**

**Subnet determines where resources are launched.**

Interviewers frequently ask:

> **Can you launch an EC2 directly into a VPC?**

**Answer:**

No.

Every EC2 instance must be launched into a subnet.

---

## Common Interview Mistakes

### Mistake 1

❌ A subnet can span multiple Availability Zones.

✅ Every subnet belongs to exactly one Availability Zone.

---

### Mistake 2

❌ EC2 receives its Private IP from the VPC.

✅ EC2 receives its Private IP from the subnet CIDR.

---

### Mistake 3

❌ Public and Private are subnet types created by AWS.

✅ A subnet becomes Public or Private based on its Route Table configuration.

---

### Mistake 4

❌ Every subnet automatically has Internet access.

✅ Internet access depends on the Route Table and Internet Gateway configuration.

---

## Interview Cross Questions

### Q1. Can a subnet span multiple Availability Zones?

**Answer:**

No.

Each subnet belongs to exactly one Availability Zone.

---

### Q2. Can multiple subnets exist inside one VPC?

**Answer:**

Yes.

A VPC can contain multiple subnets.

---

### Q3. Why does AWS require every EC2 instance to be launched into a subnet?

**Answer:**

Because the subnet determines:

- Private IP assignment
- Availability Zone
- Route Table
- Network configuration

---

### Q4. Can two subnets have overlapping CIDR blocks?

**Answer:**

No.

Subnet CIDR blocks within the same VPC must not overlap.

---

## Related Topics

- VPC
- CIDR
- Route Table
- Public Subnet
- Private Subnet
- Internet Gateway
- NAT Gateway

---

## One-line Interview Answer

> A Subnet is a logical subdivision of a VPC that provides IP address allocation, Availability Zone placement, routing, and network organization for AWS resources.

---

---

# Q16. What is the Difference Between a Public Subnet and a Private Subnet?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Public Subnet / Private Subnet |
| **Category** | AWS Networking |
| **Type** | Subnet Classification |
| **Hardware / Software / Protocol** | Software-Defined Virtual Network |
| **TCP/IP Layer** | Internet Layer (Virtual Networking) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes (Public Subnet) |

---

## Prerequisites

- Network
- VPC
- CIDR
- Subnet

---

## Definition

A subnet itself is **neither Public nor Private**.

A subnet becomes **Public** or **Private** based on **its Route Table configuration**, specifically whether it has a route to an **Internet Gateway (IGW)**.

---

## Why Do We Need Public and Private Subnets?

Not every resource should be accessible from the Internet.

Examples:

- Web Servers → Internet Accessible
- Application Servers → Internal Only
- Databases → Internal Only

Separating resources into Public and Private Subnets improves:

- Security
- Network Organization
- Scalability
- High Availability

---

## Quick Revision

| Feature | Public Subnet | Private Subnet |
|----------|---------------|----------------|
| Internet Gateway Route | ✅ Yes | ❌ No |
| Direct Internet Access | ✅ Yes | ❌ No |
| Can Assign Public IP | ✅ Yes | Usually No |
| Used For | Web Servers, Bastion Hosts, NAT Gateway | Application Servers, Databases, Internal Services |

---

## How Does a Public Subnet Work?

A Public Subnet has a Route Table containing:

```text
Destination      Target

10.0.0.0/16      Local

0.0.0.0/0        Internet Gateway
```

This route allows traffic destined for the Internet to leave the VPC through the Internet Gateway.

If an EC2 instance also has a **Public IP**, it can:

- Receive traffic from the Internet.
- Send traffic to the Internet.

---

## Public Subnet Architecture

```text
Internet

      │

Internet Gateway

      │

Route Table

0.0.0.0/0 → IGW

      │

Public Subnet

      │

Ubuntu EC2

(Public + Private IP)
```

---

## How Does a Private Subnet Work?

A Private Subnet **does not** have a Route Table entry pointing to an Internet Gateway.

Instead, it may use a NAT Gateway for outbound Internet access.

```text
Destination      Target

10.0.0.0/16      Local

0.0.0.0/0        NAT Gateway
```

Resources inside the Private Subnet:

- Cannot receive unsolicited Internet traffic.
- Can access the Internet only through a NAT Gateway (if configured).

---

## Private Subnet Architecture

```text
Internet

      ▲

Internet Gateway

      ▲

NAT Gateway

      ▲

Route Table

0.0.0.0/0 → NAT

      ▲

Private Subnet

      ▲

Application EC2

(Private IP Only)
```

---

## Public vs Private Subnet

| Feature | Public Subnet | Private Subnet |
|----------|---------------|----------------|
| Internet Gateway Route | ✅ Yes | ❌ No |
| NAT Gateway Route | Optional | ✅ Usually Yes |
| Public IP Required for Internet Access | ✅ Yes | ❌ No |
| Accepts Incoming Internet Traffic | ✅ Yes (if allowed by Security Group & NACL) | ❌ No |
| Outbound Internet Access | ✅ Directly | ✅ Through NAT Gateway |
| Typical Resources | Bastion Host, Load Balancer, NAT Gateway | EC2, RDS, ECS, EKS |

---

## How It Works Internally

### Public Subnet

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Public Subnet

↓

Ubuntu EC2
```

---

### Private Subnet

```text
Application EC2

↓

Route Table

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Notice:

Internet traffic **cannot directly enter** the Private Subnet.

---

## Assignment Example

Our assignment uses:

```text
VPC

↓

Public Subnet

↓

Ubuntu EC2

↓

Amazon Linux

↓

RHEL

↓

Amazon EFS
```

We placed the EC2 instances in a Public Subnet because we needed to SSH into them from our laptops.

In a production environment, the application servers would usually be placed in Private Subnets.

---

## Real DevOps Usage

A DevOps Engineer typically designs a VPC like this:

```text
VPC

│

├── Public Subnet
│      ├── Application Load Balancer
│      ├── Bastion Host
│      └── NAT Gateway
│
├── Private Application Subnet
│      ├── EC2
│      ├── ECS
│      └── EKS
│
└── Private Database Subnet
       └── Amazon RDS
```

This architecture improves security by minimizing Internet exposure.

---

## Production Usage

Production best practice:

- Only Internet-facing resources belong in Public Subnets.
- Application Servers should be in Private Subnets.
- Databases should always be in Private Subnets.
- NAT Gateway should be placed in a Public Subnet.

---

## Troubleshooting

### Problem

EC2 has a Public IP but is not accessible from the Internet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Route Table | No `0.0.0.0/0` route to the Internet Gateway. |
| Security Group | Required port (e.g., 22 or 80) is blocked. |
| Network ACL | Traffic is denied. |
| Public IP | Not assigned or incorrect. |

---

### Problem

Private EC2 cannot access the Internet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing NAT Gateway | No outbound Internet path. |
| Route Table | Missing `0.0.0.0/0` route to NAT Gateway. |
| NAT Gateway Down | NAT is unavailable. |
| Security Group / NACL | Outbound traffic is blocked. |

---

## Interview Tip

A subnet is **not Public because it contains a Public IP**.

A subnet is Public **only if its Route Table contains a route to an Internet Gateway**.

This is one of the most common AWS interview questions.

---

## Common Interview Mistakes

### Mistake 1

❌ Public IP automatically makes a subnet Public.

✅ A subnet becomes Public only when its Route Table points to an Internet Gateway.

---

### Mistake 2

❌ Private Subnets cannot access the Internet.

✅ They can access the Internet through a NAT Gateway.

---

### Mistake 3

❌ Databases should be placed in Public Subnets.

✅ Databases should almost always be deployed in Private Subnets.

---

## Interview Cross Questions

### Q1. What makes a subnet Public?

**Answer:**

A Route Table containing a route:

```text
0.0.0.0/0 → Internet Gateway
```

---

### Q2. Can a Public Subnet contain a Private EC2 instance?

**Answer:**

Yes.

If the EC2 instance does not have a Public IP, it cannot be reached directly from the Internet, even though it is in a Public Subnet.

---

### Q3. Can a Private Subnet contain a Public EC2 instance?

**Answer:**

No.

Even if you assign a Public IP, the subnet cannot provide Internet connectivity without a Route Table pointing to an Internet Gateway.

---

### Q4. Where should a NAT Gateway be deployed?

**Answer:**

A NAT Gateway must be deployed in a **Public Subnet** because it needs Internet connectivity through an Internet Gateway.

---

## Related Topics

- Route Table
- Internet Gateway
- NAT Gateway
- Public IP
- Security Group
- Network ACL

---

## One-line Interview Answer

> A Public Subnet has a Route Table with a route to an Internet Gateway, allowing Internet connectivity, while a Private Subnet has no direct route to an Internet Gateway and typically uses a NAT Gateway for outbound Internet access.

---

---

# Q17. What is a Route Table?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Route Table |
| **Category** | AWS Networking |
| **Type** | Routing Component |
| **Hardware / Software / Protocol** | Software-Defined Routing Table |
| **TCP/IP Layer** | Internet Layer (Routing) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- VPC
- CIDR
- Subnet

---

## Definition

A **Route Table** is a collection of **routing rules (routes)** that determines **where network traffic should be sent**.

Every subnet must be associated with a Route Table.

Whenever an EC2 instance sends a packet, AWS checks the Route Table associated with its subnet to decide the next destination.

---

## Why Do We Need a Route Table?

Suppose an EC2 instance wants to communicate with:

- Another EC2 instance
- Amazon EFS
- The Internet
- Another VPC

How does AWS know where to send the packet?

The answer is:

**Route Table.**

It acts like a GPS for network traffic.

Without a Route Table, AWS would not know where packets should go.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Decides where network traffic should go |
| **Associated With** | Subnets |
| **Contains** | Destination + Target |
| **Managed By** | AWS |
| **Mandatory** | Yes |

---

## Components of a Route

Every route contains two parts.

| Component | Description |
|-----------|-------------|
| **Destination** | The IP range you want to reach. |
| **Target** | Where AWS should send the traffic. |

Example:

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | Local |
| `0.0.0.0/0` | Internet Gateway |

---

## Understanding the Default Route

When a VPC is created, AWS automatically adds:

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | Local |

This means:

> If the destination IP belongs to the same VPC, keep the traffic inside the VPC.

Example:

```text
EC2

10.0.1.10

↓

Amazon EFS

10.0.2.50

↓

Matches

10.0.0.0/16

↓

Local

↓

Traffic stays inside the VPC
```

---

## Understanding Internet Route

To access the Internet, another route is required.

| Destination | Target |
|-------------|--------|
| `0.0.0.0/0` | Internet Gateway |

This means:

> If the destination is **outside the VPC**, send the traffic to the Internet Gateway.

Example:

```text
EC2

↓

Google

8.8.8.8

↓

Matches

0.0.0.0/0

↓

Internet Gateway

↓

Internet
```

---

## How Does a Route Table Work?

Suppose an EC2 instance sends traffic to Amazon EFS.

```text
Destination IP

10.0.2.50

↓

Check Route Table

↓

Matches

10.0.0.0/16

↓

Local

↓

Deliver inside VPC
```

Now suppose the EC2 instance opens google.com.

```text
Destination IP

8.8.8.8

↓

Check Route Table

↓

Matches

0.0.0.0/0

↓

Internet Gateway

↓

Internet
```

---

## How AWS Chooses the Correct Route

AWS always uses the **Longest Prefix Match**.

Example:

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | Local |
| `10.0.1.0/24` | NAT Gateway |
| `0.0.0.0/0` | Internet Gateway |

Suppose the destination is:

```
10.0.1.25
```

AWS compares:

- Matches `/16` ✅
- Matches `/24` ✅

AWS chooses:

```
/24
```

because it is **more specific**.

This rule is called **Longest Prefix Match**.

---

## How It Works Internally

Whenever an EC2 sends a packet:

```text
Packet

↓

Destination IP

↓

Route Table Lookup

↓

Find Matching Route

↓

Select Longest Prefix Match

↓

Forward Packet
```

This happens automatically for every packet.

---

## Assignment Example

Our assignment uses:

```text
Public Route Table

Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            Internet Gateway
```

This allows:

- EC2 ↔ Amazon EFS communication inside the VPC.
- SSH access from your laptop over the Internet.

---

## Public Route Table

```text
Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            Internet Gateway
```

Meaning:

| Destination | Action |
|-------------|--------|
| Inside VPC | Keep traffic inside VPC |
| Outside VPC | Send traffic to Internet Gateway |

---

## Private Route Table

```text
Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            NAT Gateway
```

Meaning:

| Destination | Action |
|-------------|--------|
| Inside VPC | Keep traffic inside VPC |
| Outside VPC | Send traffic to NAT Gateway |

Notice:

There is **no direct Internet Gateway** in the Private Route Table.

---

## Real DevOps Usage

A DevOps Engineer configures Route Tables to:

- Create Public and Private Subnets.
- Enable Internet access.
- Connect VPCs using VPC Peering or Transit Gateway.
- Configure VPN connectivity.
- Route traffic through NAT Gateways.
- Configure VPC Endpoints.

---

## Production Usage

Typical production architecture:

```text
Public Subnet

↓

Route Table

↓

Internet Gateway

↓

Internet



Private Subnet

↓

Route Table

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Different subnets use different Route Tables depending on their purpose.

---

## Troubleshooting

### Problem

Unable to SSH into an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing Internet Route | No `0.0.0.0/0 → Internet Gateway` entry. |
| Wrong Route Table | Subnet associated with the wrong Route Table. |
| Internet Gateway Not Attached | Route target is unavailable. |

---

### Problem

Private EC2 cannot access the Internet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing NAT Route | No `0.0.0.0/0 → NAT Gateway`. |
| NAT Gateway Down | NAT is unavailable. |
| Incorrect Route Table Association | Wrong subnet is associated with the Route Table. |

---

## Interview Tip

Remember:

A Route Table does **not move packets**.

It only **decides the next destination**.

The actual forwarding is performed by AWS networking infrastructure.

---

## Common Interview Mistakes

### Mistake 1

❌ Route Tables are attached to EC2 instances.

✅ Route Tables are associated with **Subnets**.

---

### Mistake 2

❌ A Public Subnet is created by assigning Public IPs.

✅ A Public Subnet is created by associating a Route Table that contains:

```
0.0.0.0/0 → Internet Gateway
```

---

### Mistake 3

❌ Local route can be deleted.

✅ AWS automatically creates the Local route, and it cannot be removed.

---

### Mistake 4

❌ Route Tables inspect or filter packets.

✅ Route Tables only decide **where** traffic should go. They do not allow or deny traffic—that is the job of Security Groups and Network ACLs.

---

## Interview Cross Questions

### Q1. What are the two parts of every route?

**Answer:**

- Destination
- Target

---

### Q2. Why is the Local route required?

**Answer:**

It enables communication between resources within the same VPC.

---

### Q3. Can one Route Table be associated with multiple subnets?

**Answer:**

Yes.

A single Route Table can be associated with multiple subnets.

---

### Q4. Can one subnet be associated with multiple Route Tables?

**Answer:**

No.

A subnet can be associated with only one Route Table at a time.

---

### Q5. How does AWS decide which route to use?

**Answer:**

AWS selects the **Longest Prefix Match**, meaning the most specific matching route is chosen.

---

## Related Topics

- Internet Gateway
- NAT Gateway
- VPC
- Subnet
- CIDR
- Security Group
- Network ACL

---

## One-line Interview Answer

> A Route Table is a set of routing rules associated with a subnet that determines where network traffic should be sent based on the destination IP address.

---

---

# Q18. What is an Internet Gateway (IGW)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Internet Gateway |
| **Category** | AWS Networking |
| **Type** | VPC Gateway |
| **Hardware / Software / Protocol** | AWS Managed Virtual Gateway |
| **TCP/IP Layer** | Internet Layer (Routing) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- VPC
- Subnet
- Route Table

---

## Definition

An **Internet Gateway (IGW)** is an **AWS-managed gateway** that enables communication between a **VPC** and the **Internet**.

It acts as the **entry and exit point** for Internet traffic.

Without an Internet Gateway, resources inside a VPC cannot communicate directly with the Internet.

---

## Why Do We Need an Internet Gateway?

Suppose you launch an EC2 instance with:

- Public IP
- Public Subnet

Can it access the Internet?

**No.**

Because the VPC itself still has **no connection to the Internet**.

An Internet Gateway provides that connection.

Without it:

- SSH from your laptop fails.
- Websites cannot be accessed.
- Software updates fail.
- Package installation fails.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Connects a VPC to the Internet |
| **Attached To** | VPC |
| **Managed By** | AWS |
| **Internet Traffic** | Inbound and Outbound |
| **Required For** | Public Subnets |

---

## Where is an Internet Gateway Attached?

Unlike Route Tables or Security Groups:

- Internet Gateway is **attached to a VPC**.

Example:

```text
Internet

↓

Internet Gateway

↓

VPC

↓

Subnets
```

There is **one Internet Gateway per VPC** (an Internet Gateway can only be attached to one VPC at a time).

---

## How Does an Internet Gateway Work?

Suppose an EC2 instance wants to access Google.

```text
EC2

↓

Route Table

↓

0.0.0.0/0

↓

Internet Gateway

↓

Internet

↓

Google
```

The Internet Gateway forwards the traffic between the VPC and the Internet.

---

## Incoming Traffic

Now suppose your laptop connects using SSH.

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

EC2
```

The Internet Gateway also allows traffic coming **from the Internet** to reach the VPC, provided the Route Table and security rules allow it.

---

## Conditions Required for Internet Access

For an EC2 instance to communicate directly with the Internet, all of the following must be true:

| Requirement | Required |
|-------------|:--------:|
| Internet Gateway attached to VPC | ✅ |
| Route Table contains `0.0.0.0/0 → Internet Gateway` | ✅ |
| EC2 has a Public IP (or Elastic IP) | ✅ |
| Security Group allows required traffic | ✅ |
| Network ACL allows required traffic | ✅ |

If any one of these is missing, Internet communication fails.

---

## How It Works Internally

Suppose you run:

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

The packet follows this path:

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Subnet

↓

Security Group

↓

Ubuntu EC2
```

For the response:

```text
Ubuntu EC2

↓

Security Group

↓

Route Table

↓

Internet Gateway

↓

Internet

↓

Laptop
```

The Internet Gateway simply forwards traffic between the VPC and the Internet.

---

## Assignment Example

Our assignment uses:

```text
Internet

↓

Internet Gateway

↓

VPC

↓

Public Route Table

↓

Public Subnet

↓

Ubuntu EC2

↓

Amazon Linux

↓

RHEL
```

This allows you to SSH from your laptop into all three EC2 instances.

---

## Real DevOps Usage

A DevOps Engineer uses an Internet Gateway to:

- Provide Internet access to public resources.
- Deploy web servers.
- Allow administrators to SSH into Bastion Hosts.
- Enable Application Load Balancers to receive Internet traffic.

---

## Production Usage

Typical production architecture:

```text
Internet

↓

Internet Gateway

↓

Application Load Balancer

↓

Application EC2

↓

Database
```

Notice:

Only Internet-facing components communicate through the Internet Gateway.

Databases usually do not.

---

## Troubleshooting

### Problem

Unable to SSH into EC2.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Internet Gateway not attached | VPC has no Internet connection. |
| Missing Route | No `0.0.0.0/0 → Internet Gateway` route. |
| No Public IP | EC2 cannot be reached from the Internet. |
| Security Group | SSH (Port 22) blocked. |
| Network ACL | Traffic blocked. |

---

### Problem

EC2 cannot install packages.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Internet Gateway missing | No outbound Internet path. |
| Route Table | Missing Internet route. |
| DNS Resolution Issue | Unable to resolve package repository hostnames. |

---

## Interview Tip

An Internet Gateway **does not automatically provide Internet access**.

Internet access requires **all** of the following:

- Internet Gateway attached to the VPC.
- Route Table points to the Internet Gateway.
- EC2 has a Public IP (or Elastic IP).
- Security rules allow the traffic.

---

## Common Interview Mistakes

### Mistake 1

❌ Attaching an Internet Gateway automatically makes every subnet public.

✅ A subnet becomes public only if its Route Table contains:

```text
0.0.0.0/0 → Internet Gateway
```

---

### Mistake 2

❌ An EC2 with a Public IP can access the Internet without an Internet Gateway.

✅ A Public IP alone is not enough. The VPC must also have an attached Internet Gateway and the Route Table must direct traffic to it.

---

### Mistake 3

❌ Private Subnets connect directly to the Internet Gateway.

✅ Private Subnets typically use a **NAT Gateway** for outbound Internet access.

---

### Mistake 4

❌ The Internet Gateway filters or blocks traffic.

✅ The Internet Gateway provides connectivity only.

Traffic filtering is handled by **Security Groups** and **Network ACLs**.

---

## Interview Cross Questions

### Q1. Where is an Internet Gateway attached?

**Answer:**

An Internet Gateway is attached to a **VPC**.

---

### Q2. Can a VPC have more than one Internet Gateway attached at the same time?

**Answer:**

No.

A VPC can have only one attached Internet Gateway at a time.

---

### Q3. Is an Internet Gateway required for a Private Subnet?

**Answer:**

Not directly.

Private Subnets use a NAT Gateway for outbound Internet access, while the NAT Gateway itself depends on an Internet Gateway.

---

### Q4. Does an Internet Gateway provide security?

**Answer:**

No.

It provides connectivity.

Security is controlled by Security Groups and Network ACLs.

---

## Related Topics

- VPC
- Route Table
- Public Subnet
- NAT Gateway
- Public IP
- Security Group
- Network ACL

---

## One-line Interview Answer

> An Internet Gateway (IGW) is an AWS-managed gateway attached to a VPC that enables inbound and outbound communication between the VPC and the Internet.

---

---

# Q19. What is a NAT Gateway (Network Address Translation Gateway)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Network Address Translation Gateway |
| **Category** | AWS Networking |
| **Type** | Managed NAT Service |
| **Hardware / Software / Protocol** | AWS Managed Networking Service |
| **TCP/IP Layer** | Internet Layer (Routing) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ❌ No (Current Assignment) |

---

## Prerequisites

- Network
- VPC
- Subnet
- Route Table
- Internet Gateway

---

## Definition

A **NAT (Network Address Translation) Gateway** is an **AWS-managed service** that enables resources in a **Private Subnet** to **access the Internet** while preventing the Internet from initiating connections to those resources.

NFS is a general network file-sharing protocol used by many systems (including on-premises servers and cloud storage), and Amazon EFS is just one service that uses NFS.

NFS (Network File System) – A protocol for sharing files over a network.
EFS (Elastic File System) – Amazon Web Services' (AWS) managed NFS-based file storage service.
NAS (Network Attached Storage) – A dedicated device for storing and sharing files over a network.
HPC (High-Performance Computing) – Systems designed for computationally intensive workloads.
ESXi (Elastic Sky X Integrated) – VMware's bare-metal hypervisor for running virtual machines.
PV (Persistent Volume) – A Kubernetes storage resource that persists independently of pods.
It provides **outbound Internet access only**.

---

## Why Do We Need a NAT Gateway?

Suppose you have an EC2 instance running in a **Private Subnet**.

It needs to:

- Download software updates
- Install packages
- Pull Docker images
- Access AWS APIs

However,

You do **not** want users on the Internet to connect directly to that EC2 instance.

A NAT Gateway solves this problem.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Provides outbound Internet access for Private Subnets |
| **Placed In** | Public Subnet |
| **Attached To** | Route Table (Target) |
| **Allows Incoming Internet Connections** | ❌ No |
| **Allows Outgoing Internet Connections** | ✅ Yes |

---

# Why Did AWS Design NAT Gateway This Way?

Imagine AWS allowed Private EC2 instances to connect directly to the Internet.

Then anyone on the Internet could potentially attempt to reach those instances.

Instead, AWS separates the two directions of communication:

- **Private EC2 → Internet** ✅ Allowed
- **Internet → Private EC2** ❌ Not Allowed

This provides Internet access without exposing private resources.

---

## Why Must a NAT Gateway Be in a Public Subnet?

This is one of the most common interview questions.

A NAT Gateway must communicate with:

- Private EC2 instances
- The Internet

Therefore it requires:

- A Public IP (Elastic IP)
- Internet Gateway connectivity

Only a **Public Subnet** provides both.

---

## How Does a NAT Gateway Work?

```text
Private EC2

↓

Route Table

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Notice:

The Internet never communicates directly with the Private EC2 instance.

---

## Packet Flow

Suppose Private EC2 installs Docker.

```text
Private EC2

↓

TCP Request

↓

Route Table

↓

0.0.0.0/0

↓

NAT Gateway

↓

Internet Gateway

↓

Docker Repository
```

Docker sends the response.

```text
Docker Repository

↓

Internet

↓

Internet Gateway

↓

NAT Gateway

↓

Private EC2
```

The response reaches the EC2 because **the connection was initiated by the EC2 instance**.

---

## Why Can't the Internet Reach the Private EC2?

Suppose someone on the Internet tries:

```text
Internet

↓

Private EC2
```

There is **no route** that allows unsolicited Internet traffic to enter the Private Subnet.

The packet is dropped.

Only responses to connections initiated by the Private EC2 are returned through the NAT Gateway.

---

## Route Table Configuration

### Public Subnet (Contains NAT Gateway)

```text
Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            Internet Gateway
```

---

### Private Subnet

```text
Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            NAT Gateway
```

Notice:

Private Subnet has **no route** to the Internet Gateway.

---

## Assignment Comparison

Our assignment uses:

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Public EC2
```

If this were a production environment:

```text
Internet

↓

Internet Gateway

↓

Load Balancer

↓

Private EC2

↓

NAT Gateway

↓

Internet
```

The application servers would normally be in Private Subnets.

---

## Real DevOps Usage

A DevOps Engineer uses NAT Gateway when:

- Installing software updates
- Downloading packages
- Pulling Docker images
- Accessing GitHub
- Accessing AWS APIs
- Updating Linux repositories

while keeping servers private.

---

## Production Usage

Typical production architecture:

```text
                 Internet
                     │
             Internet Gateway
                     │
      ┌──────────────┴──────────────┐
      │                             │
 Public Subnet                 Private Subnet
      │                             │
 NAT Gateway                   Application EC2
      │                             │
      └──────────────▲──────────────┘
                     │
            Outbound Internet Only
```

Only the NAT Gateway is exposed to the Internet.

The application servers remain private.

---

## Troubleshooting

### Problem

Private EC2 cannot access the Internet.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| NAT Gateway missing | No outbound Internet path. |
| NAT Gateway in Private Subnet | Incorrect deployment. NAT Gateway must be in a Public Subnet. |
| Route Table | Missing `0.0.0.0/0 → NAT Gateway`. |
| Internet Gateway | Not attached to the VPC. |
| Security Group / NACL | Outbound traffic blocked. |

---

## Interview Tip

Remember this rule:

> **Internet Gateway is for Public Subnets.**
>
> **NAT Gateway is for Private Subnets.**

---

## Common Interview Mistakes

### Mistake 1

❌ NAT Gateway allows Internet users to connect to Private EC2 instances.

✅ NAT Gateway only allows responses to connections initiated from the Private Subnet.

---

### Mistake 2

❌ NAT Gateway can be placed in a Private Subnet.

✅ NAT Gateway must be deployed in a Public Subnet.

---

### Mistake 3

❌ Private EC2 communicates directly with the Internet Gateway.

✅ Private EC2 sends traffic to the NAT Gateway, which then uses the Internet Gateway.

---

### Mistake 4

❌ NAT Gateway replaces the Internet Gateway.

✅ NAT Gateway depends on the Internet Gateway to access the Internet.

---

## Interview Cross Questions

### Q1. Why is a NAT Gateway required?

**Answer:**

To allow resources in a Private Subnet to access the Internet without allowing direct inbound Internet access.

---

### Q2. Why must a NAT Gateway be in a Public Subnet?

**Answer:**

Because it requires Internet connectivity through an Internet Gateway and an Elastic IP to communicate with the Internet.

---

### Q3. Can the Internet initiate a connection to a Private EC2 through a NAT Gateway?

**Answer:**

No.

A NAT Gateway supports outbound connections initiated by the Private EC2. It does not allow unsolicited inbound connections from the Internet.

---

### Q4. Does a NAT Gateway need an Internet Gateway?

**Answer:**

Yes.

Without an Internet Gateway, the NAT Gateway cannot communicate with the Internet.

---

## Related Topics

- Internet Gateway
- Route Table
- Public Subnet
- Private Subnet
- Security Group
- Network ACL

---

## One-line Interview Answer

> A NAT Gateway (Network Address Translation Gateway) is an AWS-managed service that allows resources in a Private Subnet to initiate outbound Internet connections while preventing direct inbound connections from the Internet.

---

---

# Q20. How are Public and Private IP Addresses Assigned in AWS?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | AWS Networking |
| **Type** | IP Address Assignment |
| **Hardware / Software / Protocol** | AWS Managed Networking |
| **TCP/IP Layer** | Internet Layer |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- IP Address
- IPv4
- VPC
- CIDR
- Subnet
- Internet Gateway
- NAT Gateway

---

## Definition

Every EC2 instance launched inside a VPC receives a **Private IP Address**.

A **Public IP Address** is optional and is assigned only if configured.

AWS uses:

- **Private IP** → Communication inside the VPC.
- **Public IP** → Communication with the Internet.

---

## Why Do We Need Both?

AWS separates **internal** and **external** communication.

| Communication | IP Used |
|--------------|---------|
| EC2 ↔ EC2 | Private IP |
| EC2 ↔ Amazon EFS | Private IP |
| EC2 ↔ Amazon RDS | Private IP |
| Laptop ↔ EC2 | Public IP |
| Internet ↔ Web Server | Public IP |

This design improves both security and scalability.

---

## Quick Revision

| IP Type | Automatically Assigned | Can Change | Internet Accessible |
|----------|-----------------------|------------|---------------------|
| Private IP | ✅ Yes | Usually No | ❌ No |
| Public IP | Optional | ✅ Yes (unless Elastic IP) | ✅ Yes |
| Elastic IP | Manual | ❌ No | ✅ Yes |

---

# Private IP Assignment

Every EC2 instance receives a **Private IPv4 Address** from the **Subnet CIDR Block**.

Example:

```text
VPC

10.0.0.0/16

↓

Subnet

10.0.1.0/24

↓

EC2

10.0.1.10
```

Notice:

The IP is assigned from the **Subnet**, not directly from the VPC.

---

# Public IP Assignment

AWS assigns a Public IP only when one of these is true:

- **Auto-assign Public IPv4** is enabled on the subnet.
- You manually enable Public IP assignment during EC2 launch.
- You associate an Elastic IP after launch.

Example:

```text
Ubuntu EC2

Private IP

10.0.1.10

+

Public IP

54.xx.xx.xx
```

---

## Auto-assign Public IP

When a subnet has:

```
Auto-assign Public IPv4

Enabled
```

every new EC2 launched into that subnet automatically receives:

- Private IP
- Public IP

If disabled:

The EC2 receives only a Private IP unless you manually assign a Public or Elastic IP.

---

## Public IP Assignment Flow

```text
Launch EC2

↓

Assign Private IP

↓

Is Auto-assign Public IP Enabled?

        │

 ┌──────┴──────┐

Yes            No

│               │

Assign        Only

Public IP     Private IP
```

---

## Is a Public IP Alone Enough?

No.

An EC2 instance needs **all** of the following:

| Requirement | Required |
|-------------|:--------:|
| Public IP | ✅ |
| Internet Gateway attached to VPC | ✅ |
| Route Table → `0.0.0.0/0 → IGW` | ✅ |
| Security Group allows traffic | ✅ |
| Network ACL allows traffic | ✅ |

Missing any one of these prevents Internet communication.

---

## Example 1 – Public IP but No Internet

```text
Public IP

✅

↓

Internet Gateway

❌

↓

No Internet Access
```

OR

```text
Public IP

✅

↓

Route Table

No

0.0.0.0/0 → IGW

↓

No Internet Access
```

A Public IP by itself does not create Internet connectivity.

---

## Example 2 – Public Subnet but No Public IP

```text
Public Subnet

↓

Internet Gateway

↓

EC2

Private IP Only
```

Result:

- EC2 can initiate outbound Internet access (if configured appropriately).
- Internet cannot initiate a connection because the EC2 has no Public IP.

---

## Example 3 – Private Subnet with Public IP

Suppose someone manually assigns a Public IP to an EC2 in a Private Subnet.

```text
Private Subnet

↓

Public IP

↓

No Route

↓

No Internet Access
```

Since the Route Table has **no route to the Internet Gateway**, the Public IP is effectively unusable for direct Internet communication.

---

## Assignment Example

Our assignment uses:

```text
Laptop

↓

Internet

↓

Public IP

↓

Ubuntu EC2

↓

Private IP

↓

Amazon EFS
```

Notice:

- SSH uses the Public IP.
- Amazon EFS uses the Private IP.

---

## Real DevOps Usage

A DevOps Engineer typically designs networks so that:

- Bastion Hosts have Public IPs.
- Load Balancers have Public IPs.
- Application Servers use only Private IPs.
- Databases use only Private IPs.

This minimizes the attack surface.

---

## Production Usage

```text
Internet

↓

Elastic IP

↓

Application Load Balancer

↓

Private EC2

↓

Amazon RDS
```

Only Internet-facing resources receive Public or Elastic IPs.

Internal resources communicate using Private IPs.

---

## Troubleshooting

### Problem

Cannot SSH into EC2.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| No Public IP | EC2 is not reachable from the Internet. |
| Internet Gateway Missing | No Internet connectivity. |
| Route Table | Missing `0.0.0.0/0 → IGW`. |
| Security Group | Port `22` blocked. |

---

### Problem

EC2 can access EFS but not the Internet.

### Explanation

This is expected if:

- Private IP communication inside the VPC works.
- No Public IP or Internet route exists.

---

## Interview Tip

Remember these rules:

- Every EC2 gets a **Private IP**.
- A **Public IP is optional**.
- A **Public IP alone is not enough** for Internet connectivity.
- A **Private IP is always used for communication within the VPC**.

---

## Common Interview Mistakes

### Mistake 1

❌ Public IP comes from the subnet CIDR.

✅ Only Private IPs come from the subnet CIDR.

Public IPs are allocated from AWS's public address pool.

---

### Mistake 2

❌ Public IP automatically means Internet access.

✅ Internet access also requires an Internet Gateway, Route Table, and appropriate security rules.

---

### Mistake 3

❌ Private IPs are used only inside a subnet.

✅ Private IPs are used for communication throughout the VPC (subject to routing and security).

---

### Mistake 4

❌ Auto-assign Public IP affects existing EC2 instances.

✅ It applies only to **new EC2 instances** launched after the setting is enabled.

---

## Interview Cross Questions

### Q1. Does every EC2 receive a Private IP?

**Answer:**

Yes.

Every EC2 instance launched in a VPC automatically receives a Private IP.

---

### Q2. Does every EC2 receive a Public IP?

**Answer:**

No.

A Public IP is assigned only if Auto-assign Public IP is enabled or if you manually associate a Public or Elastic IP.

---

### Q3. Where does the Private IP come from?

**Answer:**

The subnet's CIDR block.

---

### Q4. Where does the Public IP come from?

**Answer:**

AWS's public IPv4 address pool (or an associated Elastic IP).

---

### Q5. Can an EC2 have Internet access with only a Public IP?

**Answer:**

No.

It also requires:

- Internet Gateway
- Route Table (`0.0.0.0/0 → IGW`)
- Appropriate Security Group
- Appropriate Network ACL

---

## Related Topics

- Internet Gateway
- NAT Gateway
- Route Table
- Subnet
- Security Group
- Elastic IP

---

## One-line Interview Answer

> Every EC2 instance in a VPC automatically receives a Private IP from its subnet, while a Public IP is optional and requires additional networking components such as an Internet Gateway and appropriate routing for Internet connectivity.

---

---

# Q21. What is a Security Group (SG)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Security Group |
| **Category** | AWS Networking / Security |
| **Type** | Virtual Firewall |
| **Hardware / Software / Protocol** | AWS Managed Virtual Firewall |
| **TCP/IP Layer** | Transport Layer (Ports & Protocols) + Internet Layer (IP Filtering) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Network
- TCP/IP Model
- IP Address
- Ports
- Firewall
- VPC
- Subnet

---

## Definition

A **Security Group (SG)** is an **AWS-managed virtual firewall** that controls **which traffic is allowed to enter or leave an AWS resource** such as an EC2 instance, Amazon EFS, Amazon RDS, or an Application Load Balancer.

Unlike a traditional firewall that protects an entire network, a Security Group protects an **individual AWS resource**.

---

## Why Do We Need a Security Group?

Suppose your EC2 instance has:

- Public IP
- Internet Gateway
- Public Route Table

Without a Security Group,

**anyone on the Internet could attempt to connect to every open service.**

A Security Group allows only the traffic you explicitly permit.

Example:

```
Allow

TCP

Port 22

Source

Your Laptop IP
```

Everything else is blocked.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Controls inbound and outbound traffic for AWS resources |
| **Applied To** | EC2, EFS, RDS, ALB, Lambda ENI, ECS, EKS, etc. |
| **Stateful** | ✅ Yes |
| **Supports Allow Rules** | ✅ Yes |
| **Supports Deny Rules** | ❌ No |
| **Default Behavior** | Deny inbound, Allow outbound |

---

# Where is a Security Group Attached?

Security Groups are attached directly to AWS resources.

```
VPC

↓

Subnet

↓

EC2

↓

Security Group
```

Unlike Route Tables:

- Route Tables → Attached to Subnets
- Security Groups → Attached to Resources

---

## Components of a Security Group Rule

Every rule contains:

| Field | Example |
|---------|---------|
| Protocol | TCP |
| Port | 22 |
| Source / Destination | 192.168.1.10/32 |
| Action | Allow |

Example:

```
TCP

22

192.168.1.10/32

ALLOW
```

---

# Inbound Rules

Inbound rules control traffic **entering** the resource.

Example:

```
Laptop

↓

SSH

↓

EC2
```

Rule:

| Protocol | Port | Source |
|----------|------|---------|
| TCP | 22 | Your Public IP |

Result:

SSH Allowed.

---

# Outbound Rules

Outbound rules control traffic **leaving** the resource.

Example:

```
EC2

↓

Amazon EFS
```

Rule:

| Protocol | Port | Destination |
|----------|------|-------------|
| TCP | 2049 | Amazon EFS Security Group |

Result:

EFS communication allowed.

---

## Why is a Security Group Stateful?

This is one of the most common interview questions.

Suppose:

```
Laptop

↓

SSH

↓

EC2
```

Inbound rule:

```
Allow

TCP

22
```

When EC2 sends the response,

AWS automatically allows it.

You **do not** need another inbound or outbound rule for the return traffic.

This behavior is called **Stateful**.

---

## Packet Flow

Suppose you SSH into Ubuntu.

```
Laptop

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Subnet

↓

Security Group

↓

Ubuntu EC2
```

If the Security Group allows:

```
TCP

22

Your IP
```

the packet reaches the EC2.

Otherwise,

AWS drops the packet.

---

## Assignment Example

Ubuntu EC2

| Direction | Protocol | Port | Source / Destination |
|------------|----------|------|----------------------|
| Inbound | TCP | 22 | Your Public IP |
| Inbound | TCP | 2049 | EFS Security Group |
| Outbound | All Traffic | All | Anywhere |

Amazon EFS

| Direction | Protocol | Port | Source |
|------------|----------|------|---------|
| Inbound | TCP | 2049 | EC2 Security Group |

This allows:

- SSH
- NFS Communication

---

## Real DevOps Usage

A DevOps Engineer configures Security Groups to:

- Allow SSH only from trusted IPs.
- Allow HTTPS from the Internet.
- Restrict database access to application servers.
- Control communication between AWS services.

---

## Production Usage

Example:

```
Internet

↓

ALB SG

↓

Application SG

↓

Database SG
```

Rules:

ALB SG

```
Allow

443

Anywhere
```

Application SG

```
Allow

8080

From ALB SG
```

Database SG

```
Allow

3306

From Application SG
```

Notice:

No resource is exposed unnecessarily.

---

## Troubleshooting

### Problem

Cannot SSH into EC2.

Possible causes:

- Port 22 missing.
- Wrong Source IP.
- Wrong Protocol.
- Security Group attached to another EC2.

---

### Problem

Cannot Mount Amazon EFS.

Possible causes:

- Port 2049 missing.
- Wrong Security Group.
- EC2 Security Group not referenced by EFS.

---

## Interview Tip

A Security Group only supports:

**ALLOW**

There are **no DENY rules**.

If traffic is not explicitly allowed,

it is automatically denied.

---

## Common Interview Mistakes

### Mistake 1

❌ Security Groups support Allow and Deny.

✅ Security Groups support **Allow rules only**.

---

### Mistake 2

❌ Security Groups are attached to Subnets.

✅ Security Groups are attached directly to AWS resources.

---

### Mistake 3

❌ Return traffic requires another rule.

✅ Security Groups are **Stateful**.

Return traffic is automatically allowed.

---

### Mistake 4

❌ Opening Port 22 allows everyone.

✅ Access also depends on the configured Source IP.

---

## Interview Cross Questions

### Q1. Is a Security Group Stateful?

**Answer:**

Yes.

Return traffic is automatically allowed.

---

### Q2. Can a Security Group deny traffic?

**Answer:**

No.

It supports only Allow rules.

Traffic not explicitly allowed is implicitly denied.

---

### Q3. Can one Security Group be attached to multiple EC2 instances?

**Answer:**

Yes.

A single Security Group can be attached to multiple AWS resources.

---

### Q4. Can one EC2 instance have multiple Security Groups?

**Answer:**

Yes.

An EC2 instance can have multiple Security Groups attached, and AWS evaluates the combined allow rules.

---

### Q5. Which AWS resources commonly use Security Groups?

**Answer:**

- EC2
- Amazon EFS
- Amazon RDS
- Application Load Balancer (ALB)
- Lambda (when attached to a VPC)
- Amazon ECS
- Amazon EKS

---

## Related Topics

- Network ACL (NACL)
- Firewall
- Route Table
- Internet Gateway
- NAT Gateway
- Amazon EFS
- Amazon EC2

---

## One-line Interview Answer

> A Security Group is an AWS-managed, stateful virtual firewall attached to AWS resources that controls inbound and outbound traffic using allow rules based on protocol, port, and source or destination.

---

---

# Q22. What is a Network ACL (NACL)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Network Access Control List |
| **Category** | AWS Networking / Security |
| **Type** | Virtual Firewall |
| **Hardware / Software / Protocol** | AWS Managed Virtual Firewall |
| **TCP/IP Layer** | Transport Layer (Ports & Protocols) + Internet Layer (IP Filtering) |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Default NACL |

---

## Prerequisites

- Network
- Firewall
- VPC
- Subnet
- Route Table
- Security Group

---

## Definition

A **Network ACL (Network Access Control List)** is an **AWS-managed virtual firewall** that controls **inbound and outbound traffic at the subnet level**.

Unlike a Security Group, which protects individual AWS resources, a Network ACL protects **an entire subnet**.

Every subnet in a VPC must be associated with a Network ACL.

---

## Why Do We Need a Network ACL?

Suppose a subnet contains:

- Ubuntu EC2
- Amazon Linux EC2
- RHEL EC2

Instead of configuring security individually for every instance, a Network ACL allows you to apply a common set of network rules to the entire subnet.

It provides an additional layer of network security before traffic reaches individual resources.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Controls traffic entering and leaving a subnet |
| **Applied To** | Subnet |
| **Stateful** | ❌ No |
| **Supports Allow Rules** | ✅ Yes |
| **Supports Deny Rules** | ✅ Yes |
| **Evaluates Rules** | Rule Number Order |

---

# Where is a Network ACL Attached?

Unlike Security Groups:

```text
VPC

↓

Subnet

↓

Network ACL

↓

EC2
```

A Network ACL protects **everything inside the subnet**.

---

## Components of a Network ACL Rule

Every rule contains:

| Field | Example |
|-------|---------|
| Rule Number | 100 |
| Protocol | TCP |
| Port Range | 22 |
| Source / Destination | 0.0.0.0/0 |
| Action | Allow / Deny |

Example:

```text
Rule 100

TCP

22

0.0.0.0/0

ALLOW
```

---

# Inbound Rules

Inbound rules control traffic entering the subnet.

Example:

```text
Internet

↓

Subnet

↓

EC2
```

If the rule allows TCP Port 22, the packet proceeds to the Security Group.

---

# Outbound Rules

Outbound rules control traffic leaving the subnet.

Example:

```text
EC2

↓

Internet
```

If outbound traffic is denied, the packet never leaves the subnet.

---

## Why is a Network ACL Stateless?

This is the biggest interview question.

Suppose:

```text
Laptop

↓

SSH

↓

EC2
```

Inbound Rule:

```
ALLOW TCP 22
```

The request reaches the EC2.

When EC2 sends the response:

AWS checks the **Outbound Rule** separately.

If outbound traffic is not allowed,

the response is dropped.

Unlike Security Groups, return traffic is **not automatically allowed**.

---

## Rule Evaluation

Rules are evaluated in ascending order.

Example:

| Rule No | Port | Action |
|---------:|------|--------|
| 100 | 22 | Allow |
| 200 | All | Deny |

AWS evaluates Rule **100** first.

As soon as a matching rule is found, evaluation stops.

---

## Default Network ACL

When a VPC is created, AWS creates a **Default Network ACL**.

Default behavior:

- Allows all inbound traffic.
- Allows all outbound traffic.

Many production environments replace or customize the default Network ACL.

---

## Assignment Example

Our assignment uses the Default Network ACL.

```text
Internet

↓

Network ACL

↓

Security Group

↓

Ubuntu EC2
```

The Network ACL allows the traffic to reach the Security Group, which then determines whether the EC2 instance should accept it.

---

## Real DevOps Usage

A DevOps Engineer uses Network ACLs to:

- Block unwanted IP ranges.
- Restrict subnet-level traffic.
- Add an extra security layer beyond Security Groups.
- Control traffic entering or leaving an entire subnet.

---

## Production Usage

Example:

```text
Internet

↓

Network ACL

↓

Public Subnet

↓

Security Group

↓

Load Balancer

↓

Application
```

Traffic must pass both the Network ACL and the Security Group before reaching the application.

---

## Troubleshooting

### Problem

Cannot SSH into EC2.

Possible causes:

- Inbound Rule blocks Port 22.
- Outbound Rule blocks return traffic.
- Wrong Rule Number.
- Wrong Network ACL associated with the subnet.

---

### Problem

Amazon EFS cannot be mounted.

Possible causes:

- Port 2049 blocked.
- Outbound response blocked.
- Incorrect Network ACL association.

---

## Interview Tip

Remember:

**Network ACL = Stateless**

Both directions must be allowed.

---

## Common Interview Mistakes

### Mistake 1

❌ Network ACLs are attached to EC2 instances.

✅ Network ACLs are attached to subnets.

---

### Mistake 2

❌ Network ACLs are Stateful.

✅ Network ACLs are Stateless.

Return traffic must be explicitly allowed.

---

### Mistake 3

❌ Network ACLs support only Allow rules.

✅ Network ACLs support both Allow and Deny rules.

---

### Mistake 4

❌ Rule order does not matter.

✅ AWS evaluates rules from the lowest rule number upward and stops at the first matching rule.

---

## Interview Cross Questions

### Q1. Is a Network ACL Stateful?

**Answer:**

No.

A Network ACL is Stateless, so inbound and outbound rules are evaluated independently.

---

### Q2. Can a Network ACL deny traffic?

**Answer:**

Yes.

Unlike Security Groups, Network ACLs support both Allow and Deny rules.

---

### Q3. Where is a Network ACL attached?

**Answer:**

A Network ACL is associated with a subnet.

---

### Q4. Can one Network ACL be associated with multiple subnets?

**Answer:**

Yes.

A single Network ACL can be associated with multiple subnets.

---

### Q5. Can one subnet have multiple Network ACLs?

**Answer:**

No.

A subnet can be associated with only one Network ACL at a time.

---

## Related Topics

- Security Group
- Firewall
- Route Table
- VPC
- Subnet

---

## One-line Interview Answer

> A Network ACL (Network Access Control List) is an AWS-managed, stateless virtual firewall associated with a subnet that controls inbound and outbound traffic using ordered allow and deny rules.

---

---

# Q23. What is the Difference Between a Security Group (SG) and a Network ACL (NACL)?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | AWS Networking / Security |
| **Type** | Comparison |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

## Prerequisites

- Firewall
- Security Group
- Network ACL

---

## Definition

Both **Security Groups (SG)** and **Network ACLs (NACLs)** are AWS-managed virtual firewalls.

The difference is **where they are applied** and **how they evaluate traffic**.

- **Security Group** protects individual AWS resources (EC2, EFS, RDS, ALB, etc.).
- **Network ACL** protects an entire subnet.

---

## Quick Revision

| Feature | Security Group | Network ACL |
|---------|----------------|-------------|
| Protects | Resource | Subnet |
| Stateful | ✅ Yes | ❌ No |
| Allow Rules | ✅ Yes | ✅ Yes |
| Deny Rules | ❌ No | ✅ Yes |

---

# Security Group vs Network ACL

| Feature | Security Group (SG) | Network ACL (NACL) |
|---------|----------------------|--------------------|
| **Full Form** | Security Group | Network Access Control List |
| **Protects** | Individual AWS Resource | Entire Subnet |
| **Attached To** | EC2, EFS, RDS, ALB, etc. | Subnet |
| **Firewall Type** | Instance/Resource Level | Subnet Level |
| **State** | Stateful | Stateless |
| **Return Traffic** | Automatically Allowed | Must Be Explicitly Allowed |
| **Supports Allow Rules** | ✅ Yes | ✅ Yes |
| **Supports Deny Rules** | ❌ No | ✅ Yes |
| **Rule Evaluation** | All Allow Rules Evaluated Together | Lowest Rule Number First |
| **Default Inbound** | Deny All | Allow All (Default NACL) |
| **Default Outbound** | Allow All | Allow All (Default NACL) |
| **Rule Number Required** | ❌ No | ✅ Yes |
| **Common Use** | Resource Protection | Network Boundary Protection |

---

# Where Are They Applied?

```text
Internet

↓

Internet Gateway

↓

Network ACL

↓

Subnet

↓

Security Group

↓

EC2
```

Traffic reaches the **Network ACL first**, then the **Security Group**.

---

# Stateful vs Stateless

## Security Group (Stateful)

```text
Laptop

↓

SSH Request

↓

EC2

↓

SSH Response

↓

Laptop
```

Only the inbound rule is required.

AWS automatically allows the return traffic.

---

## Network ACL (Stateless)

```text
Laptop

↓

SSH Request

↓

EC2

↓

SSH Response

↓

Laptop
```

You must allow:

- Inbound Request
- Outbound Response

Otherwise the response is dropped.

---

# Rule Evaluation

## Security Group

AWS combines all attached Security Group rules.

If **any allow rule matches**, the traffic is allowed.

Example:

```text
SG-1

Allow TCP 22

+

SG-2

Allow TCP 80

↓

EC2 allows:

22

80
```

---

## Network ACL

AWS evaluates rules from the **lowest rule number upward**.

Example:

| Rule No | Port | Action |
|---------:|------|--------|
| 100 | 22 | Allow |
| 200 | All | Deny |

Traffic for Port `22` matches Rule **100**, so it is allowed.

AWS stops evaluating further rules.

---

# Assignment Example

For our assignment:

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Network ACL

↓

Public Subnet

↓

Security Group

↓

Ubuntu EC2

↓

Amazon EFS
```

The packet must pass:

1. Route Table
2. Network ACL
3. Security Group

before reaching the EC2 instance.

---

# Real DevOps Usage

## Security Groups

Used to:

- Allow SSH from Admin IPs.
- Allow HTTPS from the Internet.
- Restrict database access.
- Control communication between AWS resources.

---

## Network ACLs

Used to:

- Block malicious IP ranges.
- Restrict subnet-level traffic.
- Apply organization-wide network policies.
- Add an additional security layer.

---

# Production Usage

Example:

```text
Internet

↓

Network ACL

↓

Public Subnet

↓

ALB Security Group

↓

Application Security Group

↓

Private EC2

↓

Database Security Group

↓

Amazon RDS
```

Each layer provides additional protection.

---

# Troubleshooting

## Cannot SSH into EC2

### Security Group Issue

- Port 22 missing.
- Wrong Source IP.
- Wrong Security Group attached.

---

### Network ACL Issue

- Port 22 denied.
- Outbound response denied.
- Incorrect Rule Number.

---

## Amazon EFS Cannot Be Mounted

### Security Group Issue

- Port 2049 missing.
- Wrong Security Group reference.

---

### Network ACL Issue

- Port 2049 blocked.
- Return traffic blocked.

---

# Interview Tips

Remember these four points.

| Security Group | Network ACL |
|---------------|-------------|
| Resource Level | Subnet Level |
| Stateful | Stateless |
| Allow Only | Allow + Deny |
| No Rule Numbers | Rule Numbers Required |

These are the four most frequently asked differences.

---

# Common Interview Mistakes

### Mistake 1

❌ Security Groups and Network ACLs are the same.

✅ Security Groups protect resources.

Network ACLs protect subnets.

---

### Mistake 2

❌ Security Groups support Deny rules.

✅ Security Groups support only Allow rules.

---

### Mistake 3

❌ Network ACLs automatically allow return traffic.

✅ Network ACLs are Stateless.

Return traffic must be explicitly allowed.

---

### Mistake 4

❌ Security Groups are evaluated before Network ACLs.

✅ Network ACLs are evaluated before Security Groups for inbound traffic.

---

# Interview Cross Questions

### Q1. Which is Stateful?

**Answer:**

Security Group.

---

### Q2. Which is Stateless?

**Answer:**

Network ACL.

---

### Q3. Which supports Deny rules?

**Answer:**

Network ACL.

---

### Q4. Which is attached to an EC2 instance?

**Answer:**

Security Group.

---

### Q5. Which is attached to a Subnet?

**Answer:**

Network ACL.

---

### Q6. Which firewall is evaluated first for inbound traffic?

**Answer:**

Network ACL, followed by the Security Group.

---

### Q7. Can one EC2 instance have multiple Security Groups?

**Answer:**

Yes.

AWS combines the allow rules from all attached Security Groups.

---

### Q8. Can one subnet have multiple Network ACLs?

**Answer:**

No.

A subnet can be associated with only one Network ACL at a time.

---

## Related Topics

- Firewall
- Security Group
- Network ACL
- Route Table
- Internet Gateway
- NAT Gateway

---

## One-line Interview Answer

> A Security Group is a stateful, resource-level firewall that supports only allow rules, whereas a Network ACL is a stateless, subnet-level firewall that supports both allow and deny rules and evaluates rules in numerical order.


---

# Q24. Complete Packet Flow (Laptop → EC2 → Amazon EFS)

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | AWS Networking |
| **Type** | End-to-End Packet Flow |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- VPC
- CIDR
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway
- Public IP
- Private IP
- Security Group
- Network ACL
- EC2
- Amazon EFS

---

# Why Learn This?

Almost every AWS networking interview eventually becomes one question:

> **"Explain how a packet travels from your laptop to an EC2 instance."**

If you can answer this confidently, you've understood AWS networking.

---

# Complete Architecture

```text
                    Internet
                        │
                Public IP Address
                        │
                 Internet Gateway
                        │
                 Route Table Lookup
                        │
                Network ACL (Subnet)
                        │
              Security Group (EC2)
                        │
                   Ubuntu EC2
                        │
              Private IP Communication
                        │
              Security Group (EFS)
                        │
              Amazon EFS Mount Target
```

---

# Scenario 1 - SSH from Laptop to EC2

Command:

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

---

## Step 1 - Laptop Creates the Packet

Your laptop creates an SSH request.

```text
Source IP

192.168.x.x

↓

Destination IP

54.xx.xx.xx

↓

Protocol

TCP

↓

Port

22
```

---

## Step 2 - Internet

The packet travels across the Internet.

Routers use the **Destination Public IP** to forward the packet toward AWS.

---

## Step 3 - Internet Gateway (IGW)

The packet reaches the **Internet Gateway** attached to your VPC.

The Internet Gateway provides the connection between the Internet and your VPC.

Without an Internet Gateway:

```
SSH Fails
```

---

## Step 4 - Route Table

AWS checks the Route Table associated with the subnet.

Example:

```text
Destination          Target

10.0.0.0/16          Local

0.0.0.0/0            Internet Gateway
```

The packet is destined for the EC2 instance, so AWS forwards it into the correct subnet.

---

## Step 5 - Network ACL

The packet reaches the subnet.

AWS checks the Network ACL.

Example:

```text
Allow

TCP

22

0.0.0.0/0
```

If the Network ACL blocks Port 22,

the packet is dropped.

---

## Step 6 - Security Group

The packet now reaches the EC2 Security Group.

Example:

```text
TCP

22

Your Public IP

ALLOW
```

If no matching allow rule exists,

the packet is dropped.

---

## Step 7 - Ubuntu EC2

Linux receives the packet.

The SSH daemon (`sshd`) is listening on Port 22.

SSH authentication begins.

You successfully log in.

---

# Complete SSH Flow

```text
Laptop

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Network ACL

↓

Security Group

↓

Ubuntu EC2

↓

SSH Service
```

---

# Scenario 2 - Ubuntu EC2 Mounts Amazon EFS

Now the Ubuntu EC2 instance communicates with Amazon EFS.

Notice that **the Internet is no longer involved.**

---

## Step 1

Ubuntu EC2 sends traffic.

```text
Destination

Amazon EFS

↓

Private IP
```

---

## Step 2

AWS checks the Route Table.

Destination:

```
10.0.0.0/16
```

Matches:

```
Local
```

Traffic remains inside the VPC.

---

## Step 3

Network ACL

AWS checks the subnet Network ACL.

Example:

```text
TCP

2049

ALLOW
```

---

## Step 4

Security Group

EFS Security Group

```text
TCP

2049

Source

EC2 Security Group
```

Traffic is allowed.

---

## Step 5

Amazon EFS Mount Target

Amazon EFS receives the request.

The file system is mounted successfully.

---

# Complete EFS Flow

```text
Ubuntu EC2

↓

Private IP

↓

Route Table

(Local)

↓

Network ACL

↓

EFS Security Group

↓

Amazon EFS
```

Notice:

No Public IP

No Internet

No Internet Gateway

Everything happens inside the VPC.

---

# What if the EC2 Needs Internet Access?

Suppose Ubuntu executes:

```bash
sudo apt update
```

Packet flow:

```text
Ubuntu EC2

↓

Route Table

↓

Internet Gateway

↓

Internet

↓

Ubuntu Repository
```

If Ubuntu were in a **Private Subnet**:

```text
Ubuntu EC2

↓

Route Table

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

---

# Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| **Public IP** | Makes the EC2 reachable from the Internet. |
| **Private IP** | Enables communication inside the VPC. |
| **Internet Gateway** | Connects the VPC to the Internet. |
| **Route Table** | Decides where packets should go. |
| **Network ACL** | Filters traffic at the subnet level. |
| **Security Group** | Filters traffic at the resource level. |
| **EC2** | Runs the application or service. |
| **Amazon EFS** | Provides shared file storage. |

---

# Complete Decision Flow

```text
Packet Arrives

↓

Internet Gateway Attached?

↓

Yes

↓

Correct Route?

↓

Yes

↓

Network ACL Allows?

↓

Yes

↓

Security Group Allows?

↓

Yes

↓

Service Listening on Port?

↓

Yes

↓

Connection Successful
```

If the answer is **No** at any step, the connection fails.

---

# Troubleshooting Checklist

If SSH fails, check in this order:

```text
✓ Public IP

↓

✓ Internet Gateway

↓

✓ Route Table

↓

✓ Network ACL

↓

✓ Security Group

↓

✓ SSH Service

↓

✓ Key Pair

↓

✓ Username
```

---

If Amazon EFS fails to mount:

```text
✓ Same VPC

↓

✓ Mount Target Exists

↓

✓ Security Group

↓

✓ Port 2049

↓

✓ Network ACL

↓

✓ NFS Client Installed
```

---

# Interview Tips

Remember the packet flow for **Internet → EC2**:

```text
Internet

↓

Internet Gateway

↓

Route Table

↓

Network ACL

↓

Security Group

↓

EC2
```

Remember the packet flow for **EC2 → EFS**:

```text
EC2

↓

Route Table

(Local)

↓

Network ACL

↓

Security Group

↓

Amazon EFS
```

These two flows cover the majority of AWS networking interview scenarios.

---

# Common Interview Mistakes

### Mistake 1

❌ Security Group is checked before the Network ACL.

✅ For inbound traffic, the Network ACL is evaluated before the Security Group.

---

### Mistake 2

❌ Amazon EFS uses the Internet.

✅ Amazon EFS communication stays inside the VPC using Private IP addresses.

---

### Mistake 3

❌ Public IP alone provides Internet access.

✅ Internet access also requires an Internet Gateway, a Route Table entry, and appropriate security rules.

---

### Mistake 4

❌ Route Tables provide security.

✅ Route Tables decide where traffic goes. Security Groups and Network ACLs decide whether the traffic is allowed.

---

# Interview Cross Questions

### Q1. What is the order of components for an inbound SSH request?

**Answer:**

```text
Laptop
→ Internet
→ Internet Gateway
→ Route Table
→ Network ACL
→ Security Group
→ EC2
→ SSH Service
```

---

### Q2. Does Amazon EFS use a Public IP?

**Answer:**

No.

Amazon EFS communicates using Private IP addresses through its Mount Targets.

---

### Q3. Which component decides where a packet should go?

**Answer:**

The Route Table.

---

### Q4. Which components decide whether a packet is allowed?

**Answer:**

The Network ACL and the Security Group.

---

# Related Topics

- EC2
- Amazon EFS
- VPC
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway
- Security Group
- Network ACL

---

# One-line Interview Answer

> A packet reaches an EC2 instance by passing through the Internet Gateway, Route Table, Network ACL, and Security Group, while communication between EC2 and Amazon EFS remains inside the VPC using Private IP addresses and the Local route.

---

# Q25. What is Amazon EC2 (Elastic Compute Cloud)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Elastic Compute Cloud |
| **Category** | Compute Service |
| **Type** | Infrastructure as a Service (IaaS) |
| **Hardware / Software / Protocol** | Virtual Machine Service |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- AWS Region
- Availability Zone
- VPC
- Subnet
- Route Table
- Internet Gateway
- Security Group
- Key Pair (Basic Idea)

---

# Definition

Amazon **EC2 (Elastic Compute Cloud)** is an AWS service that allows you to create and run **virtual servers** (called **instances**) in the cloud.

Instead of purchasing physical servers, AWS provides virtual machines on demand.

You choose:

- Operating System
- CPU
- Memory
- Storage
- Network
- Security

AWS creates the virtual machine within minutes.

---

# Why is it Called Elastic Compute Cloud?

| Word | Meaning |
|------|----------|
| **Elastic** | Resources can be increased or decreased whenever required. |
| **Compute** | Provides CPU and Memory to run applications. |
| **Cloud** | Runs inside AWS cloud infrastructure instead of your own data center. |

---

# Why Do We Need EC2?

Before cloud computing:

- Companies purchased physical servers.
- Installation took days or weeks.
- Hardware upgrades were expensive.
- Scaling required buying new servers.

EC2 solves these problems.

Benefits:

- Launch servers in minutes.
- Pay only for what you use.
- Scale up or down easily.
- Deploy globally.

---

# Quick Revision

| Feature | Description |
|----------|-------------|
| Service Type | Compute |
| Resource | Virtual Machine (Instance) |
| Operating Systems | Linux, Windows, macOS (supported instance types) |
| Billing | Pay as You Go |
| Managed By | AWS |

---

# EC2 Architecture

```text
AWS Region

↓

VPC

↓

Subnet

↓

EC2 Instance

↓

Operating System

↓

Applications
```

---

# What Does an EC2 Instance Contain?

Every EC2 instance is created using several AWS components.

```text
EC2 Instance

├── AMI
├── Instance Type
├── Key Pair
├── Security Group
├── EBS Volume
├── ENI
└── Private IP
```

We'll study each of these separately in the next topics.

---

# How EC2 Works

When you launch an EC2 instance:

```text
Choose Region

↓

Choose AMI

↓

Choose Instance Type

↓

Choose VPC

↓

Choose Subnet

↓

Choose Security Group

↓

Choose Key Pair

↓

Launch EC2
```

AWS automatically provisions the virtual machine.

---

# Assignment Example

Our assignment launches:

```text
Ubuntu EC2

↓

Amazon Linux EC2

↓

RHEL EC2

↓

Amazon EFS
```

All three EC2 instances are connected to the same Amazon EFS file system.

---

# Real DevOps Usage

DevOps Engineers use EC2 to:

- Host web applications
- Run APIs
- Host Jenkins
- Run Docker containers
- Build Kubernetes worker nodes
- Host monitoring tools
- Run automation scripts
- Deploy CI/CD pipelines

---

# Production Usage

Typical production architecture:

```text
Internet

↓

Application Load Balancer

↓

EC2 Auto Scaling Group

↓

Amazon RDS

↓

Amazon EFS
```

Multiple EC2 instances run behind a Load Balancer to improve availability and scalability.

---

# Troubleshooting

### Problem

Cannot connect to EC2.

Possible causes:

- Security Group
- Route Table
- Internet Gateway
- Public IP missing
- Key Pair incorrect
- SSH Service stopped

---

### Problem

EC2 is running but website is not accessible.

Possible causes:

- Application not running
- Port 80/443 blocked
- Wrong Security Group
- Wrong Route Table
- DNS issue

---

# Interview Tip

Remember:

**EC2 is only a virtual machine.**

Networking, storage, and security are provided by other AWS services.

---

# Common Interview Mistakes

### Mistake 1

❌ EC2 is a physical server.

✅ EC2 is a virtual server running on AWS infrastructure.

---

### Mistake 2

❌ EC2 includes storage by default.

✅ Storage is typically provided by EBS volumes attached to the EC2 instance.

---

### Mistake 3

❌ EC2 automatically has Internet access.

✅ Internet access depends on:

- Public IP
- Internet Gateway
- Route Table
- Security Group

---

# Interview Cross Questions

### Q1. What does EC2 stand for?

**Answer:**

Elastic Compute Cloud.

---

### Q2. Is EC2 a PaaS or IaaS?

**Answer:**

Infrastructure as a Service (IaaS).

---

### Q3. What is the main purpose of EC2?

**Answer:**

To provide scalable virtual servers in the AWS cloud.

---

### Q4. Can multiple operating systems run on EC2?

**Answer:**

Yes.

For example:

- Ubuntu
- Amazon Linux
- Red Hat Enterprise Linux
- Windows Server

---

# Related Topics

- AMI
- Key Pair
- Instance Type
- EBS
- ENI
- Security Group
- Amazon EFS

---

# One-line Interview Answer

> Amazon EC2 (Elastic Compute Cloud) is an AWS Infrastructure as a Service (IaaS) offering that provides scalable virtual servers, allowing users to run applications on demand without managing physical hardware.

---

# Q26. What are the Components Required to Launch an EC2 Instance?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EC2 |
| **Type** | EC2 Launch Components |
| **Hardware / Software / Protocol** | AWS Managed Components |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- AWS Region
- Availability Zone
- VPC
- Subnet
- Security Group
- IP Address
- EC2 Basics

---

# Definition

Before an EC2 instance can be launched, AWS requires several components that define:

- Which operating system to install
- How much CPU and RAM the instance should have
- Which network it should connect to
- How storage will be attached
- How administrators will securely access the instance

These components work together to create a fully functional virtual machine.

---

# Why Do We Need These Components?

Launching an EC2 instance is similar to assembling a new computer.

Before you can use it, you must decide:

- Which Operating System to install
- How powerful the machine should be
- Where it will be connected
- How much storage it will have
- How users will log in

AWS asks for the same information before creating an EC2 instance.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **AMI** | Operating System Template |
| **Instance Type** | CPU and Memory Configuration |
| **Key Pair** | Secure SSH Login |
| **VPC** | Virtual Network |
| **Subnet** | Determines Network Placement |
| **Security Group** | Virtual Firewall |
| **EBS Volume** | Storage Disk |
| **ENI** | Network Interface |
| **Private IP** | Internal Communication |
| **Public IP** | Internet Communication (Optional) |

---

# EC2 Launch Components

```text
                   Launch EC2

                        │

        ┌───────────────┼────────────────┐

        ▼               ▼                ▼

      AMI        Instance Type      Key Pair

        ▼               ▼                ▼

 Operating System    CPU & RAM      Secure Login

        └───────────────┼────────────────┘

                        ▼

                  Networking

                        │

        ┌───────────────┼────────────────┐

        ▼               ▼                ▼

       VPC          Subnet       Security Group

                        │

                        ▼

                 Private / Public IP

                        │

                        ▼

                    ENI (Network Card)

                        │

                        ▼

                  EBS Root Volume

                        │

                        ▼

                  EC2 Instance Ready
```

---

# What Does Each Component Do?

| Component | Why It Is Needed |
|-----------|------------------|
| **AMI** | Provides the Operating System and initial software. |
| **Instance Type** | Determines CPU, RAM and networking performance. |
| **Key Pair** | Enables secure SSH login without passwords. |
| **VPC** | Provides the virtual network. |
| **Subnet** | Decides where the EC2 instance is placed within the VPC. |
| **Security Group** | Controls inbound and outbound traffic. |
| **Private IP** | Used for communication inside the VPC. |
| **Public IP** | Allows Internet communication (when configured). |
| **ENI** | Connects the EC2 instance to the VPC network. |
| **EBS Volume** | Stores the operating system and data. |

---

# How an EC2 Instance is Created

When you click **Launch Instance**, AWS performs the following steps:

```text
Choose Region

↓

Choose AMI

↓

Choose Instance Type

↓

Choose VPC

↓

Choose Subnet

↓

Attach Security Group

↓

Attach Key Pair

↓

Create ENI

↓

Assign Private IP

↓

Assign Public IP (Optional)

↓

Attach EBS Volume

↓

Launch EC2
```

Notice that **AWS automatically creates some components**, while **you choose others**.

---

# Components Created Automatically by AWS

| Component | Created Automatically? |
|-----------|------------------------|
| ENI | ✅ Yes |
| Private IP | ✅ Yes |
| Hostname | ✅ Yes |
| Root EBS Volume | ✅ Yes (from the selected AMI) |
| Public IP | ✅ If Auto-Assign is enabled |
| Security Group | ❌ You Select or Create |
| Key Pair | ❌ You Select or Create |
| AMI | ❌ You Select |
| Instance Type | ❌ You Select |

---

# Assignment Example

For this assignment, the three EC2 instances are created using:

```text
Ubuntu AMI

↓

t2.micro

↓

Default VPC

↓

Public Subnet

↓

Security Group

↓

Key Pair

↓

Root EBS Volume

↓

Ubuntu EC2
```

The same process is repeated for:

- Amazon Linux
- Red Hat Enterprise Linux

Each instance receives its own:

- ENI
- Private IP
- Public IP (if enabled)
- Root EBS Volume

---

# Real DevOps Usage

When provisioning an EC2 instance, a DevOps Engineer typically decides:

- Which AMI should be used?
- Which instance type matches the workload?
- Which subnet should host the instance?
- Which Security Group should be attached?
- How much storage is required?
- Should the instance have a Public IP?

These decisions depend on the application's performance, security, and networking requirements.

---

# Production Usage

A production EC2 instance is usually launched with:

```text
AMI

↓

Instance Type

↓

Private Subnet

↓

Security Group

↓

EBS

↓

Application Installed
```

Unlike development environments, production application servers are commonly placed in **Private Subnets** and accessed through a Load Balancer or Bastion Host.

---

# Troubleshooting

### Problem

Unable to launch an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Invalid AMI | Selected AMI is unavailable or not supported. |
| Unsupported Instance Type | Instance type not available in the selected Availability Zone. |
| Wrong Subnet | Selected subnet has no available IP addresses. |
| Security Group | Incorrect Security Group selected. |
| EBS Limit | Storage quota reached. |

---

# Interview Tips

Remember this simple order:

```text
AMI

↓

Instance Type

↓

Network

↓

Security

↓

Storage

↓

Launch
```

This is also the order followed in the AWS Console.

---

# Common Interview Mistakes

### Mistake 1

❌ An EC2 instance consists only of an operating system.

✅ An EC2 instance also requires networking, storage, security, and compute configuration.

---

### Mistake 2

❌ AWS asks only for an AMI and Instance Type.

✅ AWS also requires networking, security, and storage configuration before launching an instance.

---

### Mistake 3

❌ ENI must be created manually before launching an EC2 instance.

✅ AWS automatically creates a primary ENI during instance launch.

---

# Interview Cross Questions

### Q1. What is the first thing you choose while launching an EC2 instance?

**Answer:**

The Amazon Machine Image (AMI).

---

### Q2. Which component determines the CPU and RAM?

**Answer:**

The Instance Type.

---

### Q3. Which component provides storage?

**Answer:**

Amazon EBS.

---

### Q4. Which component provides networking?

**Answer:**

The ENI (Elastic Network Interface), connected to a VPC and Subnet.

---

### Q5. Which component controls network access?

**Answer:**

The Security Group.

---

### Q6. Which components are automatically created by AWS?

**Answer:**

- ENI
- Private IP
- Root EBS Volume (from the selected AMI)
- Hostname
- Public IP (if configured)

---

# Related Topics

- Amazon Machine Image (AMI)
- Instance Types
- Key Pair
- Elastic Block Store (EBS)
- Elastic Network Interface (ENI)

---

# One-line Interview Answer

> An EC2 instance is created by combining an AMI, Instance Type, networking (VPC, Subnet, ENI, IP addresses), Security Group, Key Pair, and EBS storage into a virtual machine running inside AWS.

---

# Q27. What is an AMI (Amazon Machine Image)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Amazon Machine Image |
| **Category** | Amazon EC2 |
| **Type** | Machine Image / Template |
| **Hardware / Software / Protocol** | Software Image |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- EC2 Launch Components
- Operating System Basics

---

# Definition

An **Amazon Machine Image (AMI)** is a **pre-configured template** used to launch an EC2 instance.

It contains everything required to create a virtual machine, including:

- Operating System
- Root EBS Volume Snapshot
- Default Software (if any)
- Boot Configuration

Think of an AMI as a **master template** or **blueprint** for creating EC2 instances.

---

# Why Do We Need an AMI?

Imagine purchasing a new laptop.

Before using it, you need to install:

- Windows
- Ubuntu
- Red Hat
- Drivers
- Basic Software

Instead of installing everything manually every time, AWS provides AMIs.

You simply choose the required AMI, and AWS launches the EC2 instance with that operating system already installed.

---

# Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Template for launching an EC2 instance |
| **Contains** | Operating System + Root Disk Snapshot + Boot Configuration |
| **Used During** | EC2 Launch |
| **Can Create Multiple EC2s** | ✅ Yes |

---

# What Does an AMI Contain?

```text
Amazon Machine Image (AMI)

│

├── Operating System
│      (Ubuntu, Amazon Linux, RHEL, Windows)

├── Root EBS Snapshot

├── Boot Configuration

└── Optional Installed Software
```

An AMI does **not** contain:

- Running Processes
- Current RAM Contents
- Current CPU State

It is only a template used to launch new instances.

---

# How Does an AMI Work?

Suppose you select:

```
Ubuntu Server 22.04 LTS
```

AWS performs the following:

```text
Ubuntu AMI

↓

Creates Root EBS Volume

↓

Copies Operating System

↓

Creates EC2 Instance

↓

Boots Ubuntu
```

The EC2 instance is created from the AMI.

The AMI itself never changes.

---

# One AMI → Multiple EC2 Instances

```text
                 Ubuntu AMI

                      │

        ┌─────────────┼─────────────┐

        ▼             ▼             ▼

    Ubuntu EC2    Ubuntu EC2    Ubuntu EC2
```

One AMI can launch thousands of EC2 instances.

---

# Types of AMIs

| AMI Type | Description |
|-----------|-------------|
| **AWS AMI** | Official AMIs maintained by AWS. |
| **AWS Marketplace AMI** | Third-party AMIs (paid or free). |
| **Community AMI** | Shared by AWS users. |
| **Custom AMI** | Created from your own EC2 instance. |

---

# Public vs Private AMI

| Public AMI | Private AMI |
|------------|-------------|
| Can be used by anyone | Only accessible to permitted AWS accounts |
| Usually provided by AWS or Marketplace | Created by your organization |
| Good for standard OS installations | Good for company-specific server images |

---

# How a Custom AMI is Created

Suppose you install:

- Ubuntu
- Docker
- Nginx
- Monitoring Agent

Instead of repeating these steps every time:

```text
Ubuntu EC2

↓

Install Software

↓

Create AMI

↓

Launch New EC2

↓

Everything Already Installed
```

This saves time and ensures consistency.

---

# Assignment Example

For this assignment we launch:

```text
Ubuntu AMI

↓

Ubuntu EC2
```

```text
Amazon Linux AMI

↓

Amazon Linux EC2
```

```text
Red Hat AMI

↓

Red Hat EC2
```

Each operating system requires a different AMI.

---

# Real DevOps Usage

A DevOps Engineer commonly creates custom AMIs containing:

- Docker
- Java
- Python
- Monitoring Agents
- Security Patches
- Company Applications

New EC2 instances launched from that AMI are ready to use immediately.

---

# Production Usage

Instead of manually configuring every server:

```text
Base AMI

↓

Install Company Software

↓

Create Custom AMI

↓

Auto Scaling

↓

Launch Hundreds of Identical EC2 Instances
```

This ensures every server has the same configuration.

---

# Troubleshooting

### Problem

Cannot launch an EC2 instance.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| AMI Deleted | Selected AMI no longer exists. |
| Wrong Region | AMIs are Region-specific. |
| Unsupported Instance Type | Selected AMI doesn't support the chosen instance type. |

---

### Problem

Application missing after launching EC2.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong AMI Selected | The required software wasn't included in the AMI. |
| Old Custom AMI | AMI was created before the software was installed. |

---

# Interview Tips

Remember:

**AMI = Template**

EC2 = Instance created from that template.

---

# Common Interview Mistakes

### Mistake 1

❌ AMI is a running virtual machine.

✅ AMI is only a template used to create virtual machines.

---

### Mistake 2

❌ One AMI can launch only one EC2 instance.

✅ One AMI can launch any number of EC2 instances.

---

### Mistake 3

❌ Editing an EC2 instance changes the original AMI.

✅ The AMI remains unchanged after launching an EC2 instance.

---

### Mistake 4

❌ AMIs are available across all AWS Regions.

✅ AMIs are Region-specific unless copied to another Region.

---

# Interview Cross Questions

### Q1. What does AMI stand for?

**Answer:**

Amazon Machine Image.

---

### Q2. Why is an AMI required?

**Answer:**

It provides the operating system and bootable template required to launch an EC2 instance.

---

### Q3. Can one AMI launch multiple EC2 instances?

**Answer:**

Yes.

One AMI can be used to launch any number of EC2 instances.

---

### Q4. What is a Custom AMI?

**Answer:**

A Custom AMI is an image created from your own EC2 instance after installing the required software and configuration.

---

### Q5. Does an AMI store the running state of an EC2 instance?

**Answer:**

No.

It stores a bootable image (including the root volume snapshot), not the running memory or CPU state.

---

# Related Topics

- Amazon EC2
- Instance Type
- Amazon EBS
- EC2 Lifecycle

---

# One-line Interview Answer

> An Amazon Machine Image (AMI) is a pre-configured template containing an operating system and boot configuration that AWS uses to launch EC2 instances.

---

# Q28. What is an EC2 Instance Type?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EC2 |
| **Type** | Compute Configuration |
| **Hardware / Software /Protocol** | Virtual Hardware Configuration |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- Amazon Machine Image (AMI)

---

# Definition

An **Instance Type** defines the **hardware configuration** of an EC2 instance.

It determines:

- CPU (vCPU)
- Memory (RAM)
- Network Performance
- Storage Performance

Think of it as choosing the hardware specifications of a new computer before purchasing it.

The AMI decides **what software runs**, while the Instance Type decides **how powerful the machine is**.

---

# Why Do We Need Instance Types?

Different applications require different amounts of computing power.

Examples:

- Small Linux Server
- Web Server
- Database Server
- Machine Learning
- High Performance Computing

Instead of giving every server identical hardware, AWS provides different Instance Types optimized for different workloads.

---

# Quick Revision

| Component | Determines |
|-----------|------------|
| **AMI** | Operating System |
| **Instance Type** | CPU, RAM, Network Performance |
| **EBS** | Storage |
| **Security Group** | Network Security |

---

# Instance Type Naming Convention

Example:

```
t2.micro
```

Breakdown:

```
t2.micro

│

├── t
│     Instance Family

├── 2
│     Generation

└── micro
      Instance Size
```

---

# Instance Family

The first letter indicates the instance family.

| Family | Optimized For |
|---------|---------------|
| **T** | General Purpose (Burstable) |
| **M** | General Purpose |
| **C** | Compute Optimized |
| **R** | Memory Optimized |
| **X** | High Memory |
| **I** | Storage Optimized |
| **P** | GPU Computing |
| **G** | Graphics / GPU |

---

# Generation

Example:

```
t2.micro

↓

Generation 2
```

Another example:

```
t3.micro

↓

Generation 3
```

Higher generations usually provide:

- Better Performance
- Better Networking
- Lower Cost per Performance
- Newer Hardware

---

# Instance Size

Examples:

```
nano

micro

small

medium

large

xlarge

2xlarge

4xlarge
```

Generally:

- CPU increases
- RAM increases
- Network performance improves

as the size increases.

---

# Example

```
t2.micro
```

Means:

| Part | Meaning |
|------|----------|
| t | Burstable General Purpose Family |
| 2 | Second Generation |
| micro | Small Instance Size |

---

# Common EC2 Families

| Family | Best For |
|---------|-----------|
| **T** | Learning, Small Applications, Development |
| **M** | General Business Applications |
| **C** | CPU Intensive Workloads |
| **R** | Memory Intensive Applications |
| **P / G** | AI, ML, Graphics |

---

# Assignment Example

For this assignment we use:

```
t2.micro
```

Reason:

- AWS Free Tier Eligible
- Enough CPU and RAM
- Suitable for Linux Practice
- Suitable for Amazon EFS Lab

---

# How AWS Uses the Instance Type

```text
AMI

↓

Ubuntu

+

Instance Type

↓

t2.micro

↓

CPU

RAM

Network

↓

Launch EC2
```

The AMI provides the operating system.

The Instance Type provides the hardware resources.

---

# Real DevOps Usage

A DevOps Engineer selects an Instance Type based on:

- Application Load
- CPU Usage
- RAM Usage
- Expected Traffic
- Cost

Choosing an oversized instance increases cost.

Choosing an undersized instance reduces performance.

---

# Production Usage

Typical examples:

| Workload | Common Family |
|----------|---------------|
| Small Web Server | T |
| Business Application | M |
| Kubernetes Worker | M / C |
| Database | R |
| AI / Machine Learning | P / G |

The selection depends on workload requirements rather than using the same family for every application.

---

# Troubleshooting

### Problem

Unable to launch an EC2 instance.

### Possible Causes

| Cause | Explanation |
|--------|-------------|
| Unsupported Instance Type | Not available in the selected Availability Zone. |
| Service Limit | Instance quota exceeded. |
| Capacity Issue | AWS temporarily has insufficient capacity in that Availability Zone. |

---

### Problem

Application is slow.

### Possible Causes

| Cause | Explanation |
|--------|-------------|
| Small Instance Type | CPU or RAM is insufficient. |
| Wrong Family | Selected family doesn't match the workload. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

EC2

↓

Launch Instance

↓

Choose Instance Type
```

---

# Interview Tips

Remember:

```
AMI

=

Software

Instance Type

=

Hardware
```

This is one of the easiest ways to explain the difference in interviews.

---

# Common Interview Mistakes

### Mistake 1

❌ AMI determines CPU and RAM.

✅ Instance Type determines CPU, RAM, and networking performance.

---

### Mistake 2

❌ t2.micro is an operating system.

✅ It is an Instance Type.

---

### Mistake 3

❌ Larger Instance Types always mean better performance.

✅ Choose an Instance Type based on the application's workload and cost requirements.

---

# Interview Cross Questions

### Q1. What is an Instance Type?

**Answer:**

An Instance Type defines the virtual hardware configuration of an EC2 instance, including CPU, memory, and network performance.

---

### Q2. What does **t2.micro** mean?

**Answer:**

- **t** → Burstable General Purpose Family
- **2** → Second Generation
- **micro** → Small Instance Size

---

### Q3. Does the AMI decide CPU and RAM?

**Answer:**

No.

The AMI provides the operating system.

The Instance Type provides the hardware resources.

---

### Q4. Why did you choose **t2.micro** for this assignment?

**Answer:**

Because it is Free Tier eligible and provides sufficient resources for learning EC2 and Amazon EFS.

---

# Related Topics

- Amazon EC2
- Amazon Machine Image (AMI)
- Amazon EBS
- EC2 Lifecycle

---

# One-line Interview Answer

> An EC2 Instance Type defines the virtual hardware configuration of an EC2 instance, including CPU, memory, networking, and performance characteristics.

---

# Q29. What is Amazon EBS (Elastic Block Store)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Elastic Block Store |
| **Category** | Amazon EC2 Storage |
| **Type** | Block Storage |
| **Hardware / Software / Protocol** | AWS Managed Block Storage |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes (Root Volume) |

---

# Prerequisites

- Amazon EC2
- AMI
- Instance Type

---

# Definition

Amazon **EBS (Elastic Block Store)** is a **persistent block storage service** for Amazon EC2.

It provides storage where the operating system, applications, and user data are stored.

An EBS volume behaves like a **virtual hard disk (HDD/SSD)** attached to an EC2 instance.

---

# Why Do We Need Amazon EBS?

An EC2 instance provides:

- CPU
- RAM
- Network

But it **does not provide permanent storage**.

Just like a physical computer needs a hard disk or SSD to store the operating system and files, an EC2 instance needs storage.

Amazon EBS provides that storage.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **AMI** | Operating System Template |
| **Instance Type** | CPU & Memory |
| **EBS** | Persistent Storage |

---

# Relationship with Previous Topics

```text
AMI

↓

Creates

↓

Root EBS Volume

↓

Attached To

↓

EC2 Instance

↓

Stores

↓

Operating System
```

---

# What Does EBS Store?

An EBS volume can store:

- Operating System
- Installed Applications
- Configuration Files
- User Data
- Log Files
- Databases

Think of it as the **hard disk of your virtual machine**.

---

# How Does Amazon EBS Work?

When you launch an EC2 instance:

```text
Choose AMI

↓

AWS Creates

↓

Root EBS Volume

↓

Attach To

↓

EC2

↓

Boot Operating System
```

The operating system runs from the EBS volume.

---

# EC2 + EBS Architecture

```text
        EC2 Instance

     ┌─────────────────┐
     │ Ubuntu Linux    │
     │ Applications    │
     │ Configuration   │
     └────────┬────────┘
              │
              ▼
     Amazon EBS Volume
```

The EC2 uses the EBS volume as its primary storage.

---

# Types of EBS Volumes

AWS provides different EBS volume types.

| Volume Type | Best For |
|-------------|----------|
| **gp3** | General Purpose SSD (Recommended) |
| **gp2** | Older General Purpose SSD |
| **io2** | High Performance Databases |
| **st1** | Throughput Optimized HDD |
| **sc1** | Cold HDD Storage |

For most Linux servers and this assignment, **gp3** is the recommended choice.

---

# Root Volume vs Additional Volume

## Root Volume

Contains:

- Operating System
- Boot Files

Example:

```text
Ubuntu

↓

Root EBS Volume

↓

/

(File System)
```

---

## Additional EBS Volume

Can be attached separately.

Used for:

- Application Data
- Database Files
- Backups

---

# Persistence

One important feature of Amazon EBS is that it is **persistent storage**.

If an EC2 instance is **stopped and started**, the data stored on the EBS volume remains available.

This allows applications and operating systems to retain their files across instance restarts.

---

# Assignment Example

For our assignment:

```text
Ubuntu EC2

↓

Root EBS

↓

Ubuntu Operating System
```

```text
Amazon Linux EC2

↓

Root EBS

↓

Amazon Linux
```

```text
RHEL EC2

↓

Root EBS

↓

Red Hat Enterprise Linux
```

Each EC2 instance has its own independent Root EBS volume.

---

# Real DevOps Usage

DevOps Engineers use Amazon EBS to store:

- Operating Systems
- Application Files
- Docker Data
- Jenkins Home Directory
- Database Files
- Application Logs

---

# Production Usage

Example:

```text
Application EC2

│

├── Root EBS
│      Operating System
│
└── Additional EBS
       Application Data
```

Separating the operating system from application data makes backup and maintenance easier.

---

# Troubleshooting

### Problem

EC2 launches but the operating system does not boot.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Root EBS Missing | The boot volume is unavailable. |
| Corrupted Snapshot | AMI or snapshot is damaged. |
| Wrong Device Mapping | Incorrect root device configuration. |

---

### Problem

Disk is Full.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Small EBS Volume | Increase the EBS volume size. |
| Log Files | Old logs consuming storage. |
| Application Data | Additional storage required. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

EC2

↓

Volumes
```

---

# Interview Tips

Remember:

```
EC2

=

Computer

EBS

=

Hard Disk / SSD
```

Without EBS, the operating system has nowhere to reside.

---

# Common Interview Mistakes

### Mistake 1

❌ EC2 stores the operating system.

✅ The operating system is stored on the Root EBS volume attached to the EC2 instance.

---

### Mistake 2

❌ An AMI is the storage device.

✅ An AMI is a template. AWS creates a Root EBS volume from that template.

---

### Mistake 3

❌ All EBS volumes are shared between EC2 instances.

✅ An EBS volume is typically attached to a single EC2 instance at a time.

---

### Mistake 4

❌ Stopping an EC2 instance automatically deletes its EBS volume.

✅ By default, the Root EBS volume remains unless it is configured to be deleted when the instance is terminated.

---

# Interview Cross Questions

### Q1. What does EBS stand for?

**Answer:**

Elastic Block Store.

---

### Q2. Why is Amazon EBS required?

**Answer:**

It provides persistent block storage for EC2 instances, storing the operating system, applications, and data.

---

### Q3. What is stored in the Root EBS volume?

**Answer:**

The operating system, boot files, and system configuration.

---

### Q4. Can one EBS volume be attached to multiple EC2 instances?

**Answer:**

Normally, **No**.

A standard EBS volume is attached to one EC2 instance at a time. *(Advanced note: Multi-Attach is available only for specific volume types such as io1/io2 and specific use cases.)*

---

### Q5. Which EBS volume type is commonly recommended for general workloads?

**Answer:**

gp3 (General Purpose SSD).

---

# Related Topics

- Amazon EC2
- Amazon Machine Image (AMI)
- Instance Types
- Amazon EFS

---

# One-line Interview Answer

> Amazon EBS (Elastic Block Store) is an AWS-managed persistent block storage service that provides virtual hard disk storage for Amazon EC2 instances.

---

# Q30. What is an EC2 Key Pair?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EC2 |
| **Type** | Authentication |
| **Hardware / Software / Protocol** | Cryptographic Key Pair |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- SSH Basics
- Public & Private IP
- Security Group

---

# Definition

A **Key Pair** is a pair of cryptographic keys used to securely authenticate users when connecting to an EC2 instance.

Instead of using a username and password, AWS uses **Public Key Cryptography** for secure access.

A Key Pair consists of:

- Public Key
- Private Key

---

# Why Do We Need a Key Pair?

Suppose you launch a Linux EC2 instance.

How do you securely log in?

AWS does not create a password for Linux EC2 instances.

Instead, it uses a Key Pair.

This provides stronger security than password-based authentication.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **Public Key** | Stored on the EC2 instance |
| **Private Key** | Stored securely by the user |
| **Authentication Method** | Public Key Cryptography |
| **Default Login** | SSH |

---

# Relationship with Previous Topics

```text
AMI

↓

Launch EC2

↓

Key Pair Selected

↓

Public Key Copied to EC2

↓

Private Key Downloaded

↓

SSH Login
```

---

# What is a Key Pair?

A Key Pair contains two mathematically related keys.

```text
Key Pair

│

├── Public Key

└── Private Key
```

Both keys work together.

The Public Key can be shared.

The Private Key must always remain secret.

---

# How Does a Key Pair Work?

When an EC2 instance is launched:

```text
Create / Select Key Pair

↓

AWS Stores

Public Key

Inside EC2

↓

Private Key

Downloaded To Your Computer
```

Later, when you connect:

```text
Laptop

↓

Private Key

↓

SSH

↓

EC2

↓

Public Key Verification

↓

Login Successful
```

AWS verifies that the Private Key matches the Public Key stored on the EC2 instance.

---

# Public Key vs Private Key

| Public Key | Private Key |
|------------|-------------|
| Stored on EC2 | Stored on your computer |
| Can be shared | Must remain secret |
| Used for verification | Used for authentication |
| Cannot log in by itself | Used during SSH login |

---

# Supported Key Pair Formats

AWS supports two key formats.

| Format | Commonly Used With |
|---------|--------------------|
| **.pem** | Linux, macOS, OpenSSH |
| **.ppk** | PuTTY on Windows |

---

# Login Example

Linux / macOS

```bash
ssh -i Assignment-Key.pem ubuntu@54.xx.xx.xx
```

Windows (PuTTY)

```text
Load

Assignment-Key.ppk

↓

Open SSH Session
```

---

# Assignment Example

For this assignment:

```text
Create Key Pair

↓

Download

Assignment-Key.pem

↓

Launch Ubuntu EC2

↓

SSH Login

↓

Mount Amazon EFS
```

The same Key Pair can be used for all three EC2 instances if selected during launch.

---

# Real DevOps Usage

DevOps Engineers use Key Pairs to:

- Securely access Linux servers
- Avoid password-based authentication
- Automate SSH connections
- Access Bastion Hosts
- Troubleshoot production servers

---

# Production Usage

Typical production architecture:

```text
Administrator

↓

Private Key

↓

Bastion Host

↓

Private EC2
```

Private Keys are stored securely and access is tightly controlled.

---

# Troubleshooting

### Problem

Permission denied (publickey)

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong Private Key | The Private Key does not match the EC2's Public Key. |
| Wrong Username | Incorrect SSH username (e.g., using `ec2-user` instead of `ubuntu`). |
| Wrong File Permissions | Private Key permissions are too open (`chmod 400`). |
| Security Group | Port 22 blocked. |

---

### Problem

Lost the Private Key

### Explanation

AWS **cannot** regenerate or download the Private Key again.

Possible solutions:

- Use EC2 Instance Connect (if configured).
- Use AWS Systems Manager (if configured).
- Create a new Key Pair and replace the Public Key using recovery methods.
- Recreate the EC2 instance if recovery is not possible.

---

# AWS Console Walkthrough

```text
AWS Console

↓

EC2

↓

Network & Security

↓

Key Pairs

↓

Create Key Pair
```

---

# Interview Tips

Remember:

```text
Public Key

↓

Stored On EC2

Private Key

↓

Stored With User
```

AWS never stores or allows you to download the Private Key again after creation.

---

# Common Interview Mistakes

### Mistake 1

❌ The Private Key is stored on the EC2 instance.

✅ The Public Key is stored on the EC2 instance.

The Private Key remains with the user.

---

### Mistake 2

❌ AWS can download the Private Key again later.

✅ The Private Key is available for download only once when the Key Pair is created.

---

### Mistake 3

❌ The same Key Pair cannot be used for multiple EC2 instances.

✅ One Key Pair can be associated with multiple EC2 instances.

---

### Mistake 4

❌ A Key Pair replaces the Security Group.

✅ The Security Group controls network access, while the Key Pair authenticates the user after the network connection is allowed.

---

# Interview Cross Questions

### Q1. What is a Key Pair?

**Answer:**

A Key Pair consists of a Public Key and a Private Key used for secure authentication when connecting to an EC2 instance.

---

### Q2. Where is the Public Key stored?

**Answer:**

On the EC2 instance.

---

### Q3. Where is the Private Key stored?

**Answer:**

On the user's computer.

---

### Q4. Can AWS regenerate the Private Key?

**Answer:**

No.

AWS allows the Private Key to be downloaded only once when the Key Pair is created.

---

### Q5. Can one Key Pair be used for multiple EC2 instances?

**Answer:**

Yes.

A single Key Pair can be associated with multiple EC2 instances.

---

# Related Topics

- Amazon EC2
- Security Group
- SSH
- Amazon EBS

---

# One-line Interview Answer

> An EC2 Key Pair is a Public and Private cryptographic key pair used to securely authenticate users when connecting to an EC2 instance over SSH.

---

# Q31. What is an ENI (Elastic Network Interface)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Elastic Network Interface |
| **Category** | Amazon EC2 Networking |
| **Type** | Virtual Network Interface Card (NIC) |
| **Hardware / Software / Protocol** | AWS Managed Virtual Network Adapter |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes (Automatically Created) |

---

# Prerequisites

- VPC
- Subnet
- Security Group
- Private IP
- Public IP
- Amazon EC2

---

# Definition

An **Elastic Network Interface (ENI)** is a **virtual network interface card (NIC)** that connects an EC2 instance to a VPC network.

Just as a physical computer requires a Network Interface Card (NIC) to communicate on a network, an EC2 instance requires an ENI to communicate within a VPC.

Every EC2 instance must have at least one ENI.

---

# Why Do We Need an ENI?

Suppose an EC2 instance needs to:

- Communicate with another EC2 instance
- Access Amazon EFS
- Connect to the Internet
- Receive SSH requests

Without a network interface, none of this communication is possible.

The ENI provides the network connectivity.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **ENI** | Connects EC2 to the VPC network |
| **Private IP** | Assigned to the ENI |
| **Public IP** | Mapped to the ENI (if configured) |
| **Security Group** | Attached to the ENI |

---

# Relationship with Previous Topics

```text
VPC

↓

Subnet

↓

ENI

↓

Private IP

↓

Security Group

↓

EC2
```

Notice:

The EC2 communicates through the ENI.

---

# What Does an ENI Contain?

An ENI contains network-related information.

```text
Elastic Network Interface

│

├── Private IP Address

├── Public IP (Optional)

├── MAC Address

├── Security Group(s)

└── Subnet Information
```

---

# How Does an ENI Work?

When an EC2 instance is launched:

```text
Launch EC2

↓

AWS Creates ENI

↓

Assigns Private IP

↓

Attaches Security Group

↓

Assigns Public IP (Optional)

↓

Attaches ENI to EC2
```

The ENI becomes the EC2 instance's connection to the VPC.

---

# EC2 Networking Architecture

```text
              EC2 Instance
                    │
                    ▼
        Elastic Network Interface
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Private IP    Security Group   MAC Address
                    │
                    ▼
                 VPC Network
```

---

# Primary ENI vs Secondary ENI

## Primary ENI

- Created automatically.
- Cannot be detached while the EC2 instance is running.
- Every EC2 instance has one primary ENI.

---

## Secondary ENI

Additional ENIs can be attached to supported instance types.

Used for:

- Multiple networks
- High Availability
- Advanced networking
- Security appliances

---

# Assignment Example

For our assignment:

```text
Ubuntu EC2

↓

Primary ENI

↓

Private IP

10.0.1.10

↓

Security Group

↓

SSH + Amazon EFS
```

The same applies to:

- Amazon Linux EC2
- RHEL EC2

AWS creates one ENI for each EC2 instance automatically.

---

# Real DevOps Usage

DevOps Engineers use ENIs when:

- Multiple network interfaces are required.
- Applications communicate across different networks.
- Moving an ENI between instances during failover.
- Running network appliances.

In day-to-day administration, AWS automatically manages the primary ENI.

---

# Production Usage

Example:

```text
Application EC2

│

├── Primary ENI
│      Production Network
│
└── Secondary ENI
       Management Network
```

Multiple ENIs are generally used only for specialized networking scenarios.

---

# Troubleshooting

### Problem

EC2 cannot communicate with other resources.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Wrong Security Group | Traffic blocked. |
| Wrong Subnet | Incorrect network placement. |
| Missing Private IP | ENI configuration issue. |

---

### Problem

Cannot SSH into EC2.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Public IP Missing | Internet cannot reach the ENI. |
| Security Group | Port 22 blocked. |
| Route Table | No Internet route. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

EC2

↓

Network Interfaces
```

---

# Interview Tips

Remember:

```text
Physical Computer

↓

NIC

Virtual Machine (EC2)

↓

ENI
```

An ENI is simply the virtual version of a physical network card.

---

# Common Interview Mistakes

### Mistake 1

❌ ENI and Security Group are the same.

✅ An ENI provides network connectivity, while a Security Group filters traffic.

---

### Mistake 2

❌ An ENI stores data.

✅ An ENI stores only networking information such as IP addresses, MAC address, subnet, and Security Groups.

---

### Mistake 3

❌ Every ENI has a Public IP.

✅ Every ENI has a Private IP. A Public IP is optional.

---

### Mistake 4

❌ You must manually create an ENI before launching every EC2.

✅ AWS automatically creates the primary ENI during instance launch.

---

# Interview Cross Questions

### Q1. What is an ENI?

**Answer:**

An Elastic Network Interface is a virtual network adapter that connects an EC2 instance to a VPC network.

---

### Q2. Does every EC2 instance have an ENI?

**Answer:**

Yes.

Every EC2 instance has at least one primary ENI.

---

### Q3. Where is the Private IP assigned?

**Answer:**

The Private IP is assigned to the ENI.

---

### Q4. Can an EC2 instance have multiple ENIs?

**Answer:**

Yes.

Supported instance types can have multiple ENIs attached.

---

### Q5. Does an ENI provide security?

**Answer:**

No.

Security is provided by the Security Groups attached to the ENI.

---

# Related Topics

- Amazon EC2
- Security Group
- Subnet
- Private IP
- Public IP
- Elastic IP

---

# One-line Interview Answer

> An Elastic Network Interface (ENI) is an AWS-managed virtual network adapter that connects an EC2 instance to a VPC by providing networking components such as private IP addresses, optional public IPs, MAC addresses, and associated Security Groups.

---

# Q32. What is the Amazon EC2 Instance Lifecycle?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EC2 |
| **Type** | Instance Lifecycle |
| **Hardware / Software / Protocol** | EC2 Instance State Management |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- AMI
- EBS
- ENI

---

# Definition

The **EC2 Instance Lifecycle** describes the different **states** an EC2 instance goes through from the time it is launched until it is terminated.

Each state determines:

- Whether the instance is running.
- Whether compute charges apply.
- Whether the operating system is available.
- Whether you can connect to the instance.

---

# Why Do We Need Different States?

Suppose you have completed your work for the day.

You don't want to pay for CPU and RAM while the server is idle.

Instead of deleting the instance, you can simply stop it.

AWS provides different lifecycle states to help manage cost and availability.

---

# Quick Revision

| State | Description |
|--------|-------------|
| **Pending** | Instance is being created. |
| **Running** | Instance is powered on and ready to use. |
| **Stopping** | Instance is shutting down. |
| **Stopped** | Instance is powered off but storage is preserved. |
| **Rebooting** | Operating system is restarting. |
| **Shutting-down** | Instance is being terminated. |
| **Terminated** | Instance has been deleted permanently. |

---

# EC2 Lifecycle Diagram

```text
Launch

↓

Pending

↓

Running

↓

┌───────────────┬───────────────┐
│               │               │
▼               ▼               ▼

Reboot       Stop          Terminate

│               │               │

▼               ▼               ▼

Running      Stopped      Shutting-down

                │

                ▼

              Start

                │

                ▼

             Running
```

---

# Pending State

```text
Launch EC2

↓

Pending
```

AWS is:

- Creating the EC2 instance.
- Creating the ENI.
- Creating the Root EBS Volume.
- Assigning IP addresses.
- Booting the operating system.

This state usually lasts only a few seconds.

---

# Running State

```text
Pending

↓

Running
```

The EC2 instance is:

- Powered on.
- Operating system is running.
- SSH is available.
- Applications can run.

This is the normal working state.

---

# Reboot State

```text
Running

↓

Reboot

↓

Running
```

A reboot:

- Restarts only the operating system.
- Does **not** move the instance to another host.
- Keeps the same Private IP.
- Keeps the same EBS volume.
- Usually keeps the same Public IP.

Think of it like restarting your laptop.

---

# Stop State

```text
Running

↓

Stopping

↓

Stopped
```

When stopped:

- CPU is turned off.
- RAM is cleared.
- Operating system stops.
- Root EBS volume remains.
- Additional EBS volumes remain.

You are **not charged for compute**, but you **continue to pay for EBS storage**.

---

# Start State

```text
Stopped

↓

Start

↓

Pending

↓

Running
```

When started again:

- AWS powers on the instance.
- Operating system boots.
- EBS volumes are reattached.

Important:

If the instance uses an **auto-assigned Public IPv4 address**, AWS may assign a **new Public IP** after the instance starts.

The **Private IP** remains the same.

---

# Terminate State

```text
Running

↓

Shutting-down

↓

Terminated
```

Termination permanently deletes the EC2 instance.

By default:

- Root EBS volume is deleted.
- Additional EBS volumes may remain if configured not to delete.
- ENI is removed.
- Public IP is released.

A terminated instance **cannot be started again**.

---

# State Comparison

| State | CPU | RAM | EBS | Private IP | Public IP* |
|--------|-----|-----|-----|------------|------------|
| Pending | Starting | Initializing | Creating | Assigned | Assigned (if enabled) |
| Running | ✅ | ✅ | Attached | Same | Same |
| Stopped | ❌ | Cleared | Preserved | Same | May Change |
| Reboot | Restarted | Restarted | Preserved | Same | Usually Same |
| Terminated | Deleted | Deleted | Deleted* | Released | Released |

\* Auto-assigned Public IPv4 addresses may change after Stop/Start. Elastic IPs remain the same.

---

# Assignment Example

During this assignment you may:

```text
Launch Ubuntu EC2

↓

Running

↓

Mount Amazon EFS

↓

Stop EC2

↓

Start EC2

↓

Verify Amazon EFS Still Mounted
```

Notice:

The Root EBS volume still contains Ubuntu after the instance starts again.

---

# Real DevOps Usage

DevOps Engineers commonly:

- Stop development servers outside working hours to reduce costs.
- Reboot servers after kernel updates.
- Terminate temporary testing environments.
- Start servers when needed.

---

# Production Usage

In production:

- Reboots are used for operating system updates.
- Stop/Start is avoided unless necessary because it causes downtime.
- Termination is typically performed only when replacing or decommissioning infrastructure.

---

# Troubleshooting

### Problem

EC2 starts but SSH fails.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Public IP Changed | Update the SSH command with the new Public IP. |
| Security Group | Port 22 blocked. |
| SSH Service | SSH daemon not running. |

---

### Problem

Application missing after restart.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Data stored in temporary storage | Data wasn't stored on EBS. |
| Wrong EBS Volume | Expected volume not attached. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

EC2

↓

Instances

↓

Select Instance

↓

Instance State

↓

Start

Stop

Reboot

Terminate
```

---

# Interview Tips

Remember:

- **Reboot** = Restart the operating system.
- **Stop** = Power off the virtual machine.
- **Start** = Power it back on.
- **Terminate** = Permanently delete the instance.

---

# Common Interview Mistakes

### Mistake 1

❌ Stop and Terminate are the same.

✅ Stop preserves the instance and EBS volume.

Terminate permanently deletes the instance.

---

### Mistake 2

❌ Reboot changes the Private IP.

✅ Reboot keeps the same Private IP.

---

### Mistake 3

❌ A stopped EC2 instance is billed the same as a running instance.

✅ Compute charges stop when the instance is stopped, but storage charges for EBS continue.

---

### Mistake 4

❌ A terminated EC2 instance can be started again.

✅ A terminated instance cannot be recovered or restarted.

---

# Interview Cross Questions

### Q1. Which EC2 state allows SSH access?

**Answer:**

Running.

---

### Q2. What happens to the Root EBS volume when an EC2 instance is stopped?

**Answer:**

It remains attached and preserves the operating system and data.

---

### Q3. What happens to the Private IP after Stop → Start?

**Answer:**

The Private IP remains the same.

---

### Q4. What happens to the Public IP after Stop → Start?

**Answer:**

If it is an **auto-assigned Public IPv4 address**, AWS may assign a new Public IP when the instance starts again. If an **Elastic IP** is associated, it remains the same.

---

### Q5. Can a terminated EC2 instance be restarted?

**Answer:**

No.

A terminated EC2 instance is permanently deleted.

---

# Related Topics

- Amazon EC2
- Amazon EBS
- ENI
- Public IP
- Elastic IP

---

# One-line Interview Answer

> The Amazon EC2 Instance Lifecycle defines the sequence of states an EC2 instance transitions through—such as Pending, Running, Stopped, and Terminated—while determining its availability, billing, and resource behavior.

---

# Q33. What is Amazon EFS (Elastic File System)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Elastic File System |
| **Category** | AWS Storage |
| **Type** | Managed File Storage |
| **Hardware / Software / Protocol** | AWS Managed Network File System |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- Amazon EBS
- VPC
- Subnet
- Security Group

---

# Definition

Amazon **EFS (Elastic File System)** is a fully managed **shared file storage service** that allows multiple EC2 instances to access the **same files simultaneously**.

Unlike Amazon EBS, which is typically attached to a single EC2 instance, Amazon EFS can be mounted by multiple EC2 instances at the same time.

It uses the **NFS (Network File System)** protocol for communication.

---

# Why Do We Need Amazon EFS?

Suppose you have three EC2 instances:

- Ubuntu
- Amazon Linux
- Red Hat Enterprise Linux

Each instance has its own Root EBS volume.

```
Ubuntu

↓

Own Storage
```

```
Amazon Linux

↓

Own Storage
```

```
RHEL

↓

Own Storage
```

If Ubuntu creates a file,

Amazon Linux cannot see it.

Each EC2 has its own storage.

To share files between all three servers, we use Amazon EFS.

---

# Quick Revision

| Service | Storage Type | Shared? |
|----------|--------------|---------|
| Amazon EBS | Block Storage | ❌ No |
| Amazon EFS | File Storage | ✅ Yes |

---

# Relationship with Previous Topics

```text
EC2

↓

EBS

↓

Own Storage



EC2

↓

EFS

↓

Shared Storage
```

---

# How Does Amazon EFS Work?

```
          Amazon EFS

               │

      ┌────────┼────────┐

      ▼        ▼        ▼

 Ubuntu     Amazon     RHEL

  EC2        Linux      EC2

              EC2
```

All EC2 instances see the same files.

If one instance creates a file,

every other mounted instance can immediately access it.

---

# Architecture

```text
             Amazon EFS

                  │

        ┌─────────┼─────────┐

        ▼         ▼         ▼

 Ubuntu EC2   Amazon Linux   RHEL EC2

                    EC2
```

Each EC2 has:

- Own CPU
- Own RAM
- Own Root EBS

But all share one common file system.

---

# EBS vs EFS

| Amazon EBS | Amazon EFS |
|------------|------------|
| Block Storage | File Storage |
| Usually attached to one EC2 | Shared by multiple EC2 instances |
| Acts like a hard disk | Acts like a shared network drive |
| Mounted as a block device | Mounted using NFS |

---

# Assignment Example

Our assignment creates:

```
Ubuntu EC2

↓

Amazon Linux EC2

↓

RHEL EC2

↓

Amazon EFS
```

After mounting:

```
Ubuntu

Create

hello.txt

↓

Amazon Linux

Can Read

↓

RHEL

Can Read
```

All three instances access the same file.

---

# Real DevOps Usage

DevOps Engineers use Amazon EFS for:

- Shared application files
- Website content
- Configuration files
- User uploads
- Container shared storage

---

# Production Usage

Typical production architecture:

```
Application Load Balancer

↓

EC2 Auto Scaling Group

↓

Amazon EFS
```

Every EC2 instance reads and writes to the same shared storage.

---

# Troubleshooting

### Problem

EC2 cannot mount Amazon EFS.

### Possible Causes

| Cause | Explanation |
|--------|-------------|
| Mount Target Missing | No Mount Target in the Availability Zone. |
| Security Group | Port 2049 blocked. |
| NFS Client Missing | NFS utilities not installed. |
| Wrong Mount Command | Incorrect DNS name or mount target. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

Amazon EFS

↓

Create File System
```

---

# Interview Tips

Remember:

```
EBS

=

One EC2

EFS

=

Many EC2
```

This is probably the most common interview comparison.

---

# Common Interview Mistakes

### Mistake 1

❌ Amazon EFS replaces Amazon EBS.

✅ They solve different problems.

EBS provides storage for one EC2.

EFS provides shared storage for multiple EC2 instances.

---

### Mistake 2

❌ Amazon EFS stores the operating system.

✅ The operating system remains on the Root EBS volume.

Amazon EFS stores shared files.

---

### Mistake 3

❌ Amazon EFS uses Public IP addresses.

✅ Amazon EFS communicates inside the VPC using Private IP addresses through Mount Targets.

---

# Interview Cross Questions

### Q1. What does EFS stand for?

**Answer:**

Elastic File System.

---

### Q2. Why do we need Amazon EFS?

**Answer:**

To provide a shared file system that multiple EC2 instances can access simultaneously.

---

### Q3. What protocol does Amazon EFS use?

**Answer:**

NFS (Network File System).

---

### Q4. Can Ubuntu, Amazon Linux, and RHEL mount the same Amazon EFS?

**Answer:**

Yes.

As long as they support NFS and have network connectivity, they can mount the same Amazon EFS file system.

---

### Q5. Does Amazon EFS replace Amazon EBS?

**Answer:**

No.

EBS provides block storage for individual EC2 instances, while EFS provides shared file storage for multiple EC2 instances.

---

# Related Topics

- Amazon EC2
- Amazon EBS
- Mount Targets
- NFS

---

# One-line Interview Answer

> Amazon EFS (Elastic File System) is an AWS-managed, scalable file storage service that allows multiple EC2 instances to share the same file system simultaneously using the NFS protocol.

---

# Q34. What is the Architecture of Amazon EFS?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EFS |
| **Type** | Storage Architecture |
| **Hardware / Software / Protocol** | AWS Managed Distributed File System |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- Amazon EFS
- VPC
- Subnet
- Availability Zone

---

# Definition

Amazon EFS is a **regional file system**.

Although you create **one EFS File System**, AWS stores and makes it available across multiple Availability Zones by using **Mount Targets**.

This provides:

- High Availability
- Scalability
- Fault Tolerance

---

# Why Do We Need This Architecture?

Suppose your application runs in multiple Availability Zones.

```text
AZ-1

↓

Ubuntu EC2

AZ-2

↓

Amazon Linux EC2

AZ-3

↓

RHEL EC2
```

All three instances should access the same files.

Amazon EFS provides one shared file system that can be accessed from all Availability Zones within the same Region.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **EFS File System** | Shared Storage |
| **Mount Target** | Network entry point to the EFS File System |
| **Availability Zone** | Local network access for EC2 instances |

---

# Amazon EFS Architecture

```text
                    AWS Region

                        │

             Amazon EFS File System

        ┌───────────────┼───────────────┐

        ▼               ▼               ▼

 Mount Target      Mount Target     Mount Target

    AZ-1              AZ-2             AZ-3

        │               │               │

        ▼               ▼               ▼

 Ubuntu EC2      Amazon Linux EC2     RHEL EC2
```

There is:

- **One EFS File System**
- **Multiple Mount Targets**
- **Multiple EC2 Instances**

---

# Relationship with Previous Topics

```text
EC2

↓

Private IP

↓

Mount Target

↓

Amazon EFS
```

Notice:

The EC2 instance **never communicates directly with the EFS File System**.

It connects through the Mount Target in its Availability Zone.

---

# Main Components

## 1. Amazon EFS File System

This is the shared storage created by AWS.

It stores:

- Files
- Directories
- Permissions
- Metadata

There is only one logical file system.

---

## 2. Mount Targets

Each Mount Target acts as the **network endpoint** for the EFS File System inside a subnet.

EC2 instances connect to the Mount Target, not directly to the storage backend.

Each Mount Target has:

- Private IP Address
- Security Group
- Subnet
- Availability Zone

---

## 3. EC2 Instances

Each EC2:

- Mounts the EFS File System.
- Reads and writes files.
- Uses NFS (Port 2049).

---

# Data Flow

Suppose Ubuntu creates a file.

```text
Ubuntu EC2

↓

Mount Target

↓

Amazon EFS

↓

Mount Target

↓

Amazon Linux EC2
```

The file becomes immediately available to every mounted EC2 instance.

---

# Multi-AZ Example

```text
AZ-1

Ubuntu EC2

↓

Mount Target

│

──────── Amazon EFS ────────

│

Mount Target

↓

AZ-2

Amazon Linux EC2

│

Mount Target

↓

AZ-3

RHEL EC2
```

Even though the EC2 instances are in different Availability Zones, they all access the same file system.

---

# Assignment Example

Our assignment creates:

```text
Amazon EFS

↓

Mount Target

↓

Ubuntu EC2

↓

Amazon Linux EC2

↓

RHEL EC2
```

All three operating systems mount the same EFS File System.

If Ubuntu creates:

```text
hello.txt
```

Amazon Linux and RHEL can immediately read it.

---

# Real DevOps Usage

DevOps Engineers commonly use this architecture for:

- Shared application files
- Shared website content
- Container shared storage
- Shared configuration files
- User uploads

---

# Production Usage

Typical production architecture:

```text
                    Internet

                        │

             Application Load Balancer

                        │

      ┌─────────────────┼─────────────────┐

      ▼                 ▼                 ▼

 Application EC2   Application EC2   Application EC2

        │                 │                 │

        └──────────────┬────────────────────┘

                       ▼

               Amazon EFS File System
```

Every application server accesses the same shared storage.

---

# Troubleshooting

### Problem

One EC2 can mount EFS, another cannot.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Missing Mount Target | No Mount Target in that Availability Zone. |
| Security Group | Port 2049 blocked. |
| Wrong Subnet | Network connectivity issue. |

---

### Problem

File created on Ubuntu is not visible on Amazon Linux.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| EFS Not Mounted | EC2 is using its local EBS instead of EFS. |
| Wrong Mount Command | Mounted a different file system. |
| Permissions | Linux file permissions prevent access. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

Amazon EFS

↓

File Systems

↓

Select File System

↓

Network

↓

Mount Targets
```

---

# Interview Tips

Remember:

```text
One Region

↓

One EFS File System

↓

One Mount Target Per Availability Zone

↓

Many EC2 Instances
```

This summarizes the architecture in one line.

---

# Common Interview Mistakes

### Mistake 1

❌ Each EC2 has its own EFS.

✅ Multiple EC2 instances share one EFS File System.

---

### Mistake 2

❌ EC2 communicates directly with the EFS storage backend.

✅ EC2 communicates through the Mount Target.

---

### Mistake 3

❌ Mount Targets are shared across Regions.

✅ Mount Targets exist within subnets in a specific Availability Zone and Region.

---

# Interview Cross Questions

### Q1. Is Amazon EFS Regional or Availability Zone specific?

**Answer:**

Amazon EFS is a **Regional** service.

---

### Q2. What is the purpose of a Mount Target?

**Answer:**

A Mount Target provides the network endpoint through which EC2 instances access the EFS File System.

---

### Q3. Can EC2 instances in different Availability Zones access the same EFS?

**Answer:**

Yes.

As long as appropriate Mount Targets exist and networking allows it, EC2 instances in different Availability Zones can mount the same EFS File System.

---

### Q4. Does Amazon EFS use Public IP addresses?

**Answer:**

No.

Communication occurs using Private IP addresses through the Mount Targets.

---

# Related Topics

- Mount Targets
- NFS
- Amazon EC2
- Amazon EBS
- Availability Zones

---

# One-line Interview Answer

> Amazon EFS is a Regional, shared file storage service that uses Mount Targets in different Availability Zones to provide highly available and scalable access for multiple EC2 instances.

---

# Q35. What is a Mount Target in Amazon EFS?

## Topic Information

| Property | Value |
|----------|-------|
| **Category** | Amazon EFS |
| **Type** | Network Endpoint |
| **Hardware / Software / Protocol** | AWS Managed Network Endpoint |
| **Interview Level** | 🟢 Basic → 🔴 Advanced |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EFS
- Amazon EC2
- VPC
- Subnet
- Availability Zone

---

# Definition

A **Mount Target** is a **network endpoint** that allows EC2 instances to connect to an Amazon EFS File System.

Think of it as the **entry point** to Amazon EFS.

Without a Mount Target, an EC2 instance has **no network path** to reach the EFS File System.

---

# Why Do We Need a Mount Target?

Suppose you create an Amazon EFS File System.

```
Amazon EFS

✓ Created
```

Can an EC2 instance connect to it?

**No.**

The EFS File System is only the storage.

AWS still needs a **network endpoint** inside your VPC so that EC2 instances know **where to connect**.

That network endpoint is called a **Mount Target**.

---

# Quick Revision

| Component | Purpose |
|-----------|---------|
| **Amazon EFS** | Stores the files |
| **Mount Target** | Provides network access to the EFS File System |
| **EC2** | Mounts the EFS through the Mount Target |

---

# Relationship with Previous Topics

```text
EC2

↓

Private IP

↓

Mount Target

↓

Amazon EFS
```

Notice:

EC2 **never connects directly** to the EFS storage.

It always connects through a Mount Target.

---

# What Does a Mount Target Contain?

Every Mount Target has:

- Private IP Address
- Subnet
- Availability Zone
- Security Group

It acts as the network interface for Amazon EFS inside your VPC.

---

# How Does a Mount Target Work?

Suppose Ubuntu mounts Amazon EFS.

```text
Ubuntu EC2

↓

Private IP

↓

Mount Target

↓

Amazon EFS
```

When Ubuntu creates:

```
hello.txt
```

The file is stored in Amazon EFS.

Now Amazon Linux accesses:

```text
Amazon Linux EC2

↓

Mount Target

↓

Amazon EFS

↓

hello.txt
```

The same file is immediately available.

---

# Why One Mount Target Per Availability Zone?

Suppose your VPC has three Availability Zones.

```
AZ-1

↓

Ubuntu
```

```
AZ-2

↓

Amazon Linux
```

```
AZ-3

↓

RHEL
```

AWS recommends creating one Mount Target in each Availability Zone.

```
          Amazon EFS

       ┌─────┼─────┐

       ▼     ▼     ▼

 MT-1  MT-2  MT-3

  │      │      │

AZ-1  AZ-2  AZ-3
```

Benefits:

- Lower latency
- Higher availability
- Better fault tolerance

---

# Components of a Mount Target

```text
Mount Target

│

├── Private IP

├── Subnet

├── Availability Zone

└── Security Group
```

Notice:

A Mount Target **does not** have a Public IP.

Communication remains inside the VPC.

---

# Assignment Example

Our assignment creates:

```text
Amazon EFS

↓

Mount Target

↓

Ubuntu EC2

↓

Amazon Linux EC2

↓

RHEL EC2
```

All three EC2 instances use the Mount Target to reach the same EFS File System.

---

# Real DevOps Usage

DevOps Engineers create Mount Targets to:

- Connect EC2 instances to EFS.
- Provide low-latency access within each Availability Zone.
- Improve high availability.
- Simplify shared storage access.

---

# Production Usage

Typical production architecture:

```text
                 Amazon EFS

          ┌─────────┼─────────┐

          ▼         ▼         ▼

     Mount      Mount      Mount

     Target     Target     Target

      │           │           │

      ▼           ▼           ▼

 Application  Application  Application

    EC2          EC2          EC2
```

Each Availability Zone has its own Mount Target.

All application servers access the same file system.

---

# Troubleshooting

### Problem

Unable to mount Amazon EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Mount Target Missing | No Mount Target exists in the required Availability Zone. |
| Wrong Security Group | NFS traffic blocked. |
| Wrong DNS Name | Incorrect EFS mount endpoint used. |
| Wrong Subnet | Network connectivity issue. |

---

### Problem

High latency while accessing EFS.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Cross-AZ Access | EC2 is accessing a Mount Target in another Availability Zone. |
| Missing Local Mount Target | No Mount Target in the EC2's Availability Zone. |

---

# AWS Console Walkthrough

```text
AWS Console

↓

Amazon EFS

↓

File Systems

↓

Select File System

↓

Network

↓

Create Mount Target
```

---

# Interview Tips

Remember:

```
Amazon EFS

=

Storage

Mount Target

=

Network Entry Point
```

Without a Mount Target,

EC2 cannot access Amazon EFS.

---

# Common Interview Mistakes

### Mistake 1

❌ Mount Target stores files.

✅ The EFS File System stores files.

The Mount Target only provides network connectivity.

---

### Mistake 2

❌ Mount Target has a Public IP.

✅ A Mount Target uses a Private IP inside the VPC.

---

### Mistake 3

❌ One Mount Target is required for every EC2 instance.

✅ Multiple EC2 instances in the same Availability Zone can use the same Mount Target.

---

### Mistake 4

❌ EC2 communicates directly with Amazon EFS.

✅ EC2 communicates through the Mount Target.

---

# Interview Cross Questions

### Q1. What is a Mount Target?

**Answer:**

A Mount Target is a network endpoint that enables EC2 instances to connect to an Amazon EFS File System.

---

### Q2. Why is a Mount Target required?

**Answer:**

Because it provides the network path between EC2 instances and the EFS File System.

---

### Q3. Does a Mount Target have a Public IP?

**Answer:**

No.

A Mount Target has a Private IP address inside the VPC.

---

### Q4. Why should you create one Mount Target per Availability Zone?

**Answer:**

To provide local network access, reduce latency, and improve high availability for EC2 instances in each Availability Zone.

---

### Q5. Can multiple EC2 instances use the same Mount Target?

**Answer:**

Yes.

Multiple EC2 instances in the same Availability Zone can access the same Mount Target.

---

# Related Topics

- Amazon EFS
- NFS
- Amazon EC2
- Security Group
- Availability Zone

---

# One-line Interview Answer

> A Mount Target is an AWS-managed network endpoint with a private IP address that enables EC2 instances to access an Amazon EFS File System within a VPC.

---

# Q36. What is NFS (Network File System)?

## Topic Information

| Property | Value |
|----------|-------|
| **Full Form** | Network File System |
| **Category** | Network File Sharing Protocol |
| **Type** | File Sharing Protocol |
| **Hardware / Software / Protocol** | Network Protocol |
| **Interview Level** | 🟢 Basic → 🟡 Intermediate |
| **Used in This Assignment** | ✅ Yes |

---

# Prerequisites

- Amazon EC2
- Amazon EFS
- Mount Targets
- TCP/IP Basics

---

# Definition

**NFS (Network File System)** is a network protocol that allows one computer to access files stored on another computer **as if they were stored on its own local disk**.

Amazon EFS uses **NFS version 4.1** as its communication protocol.

---

# Why Do We Need NFS?

Suppose you have three EC2 instances.

```
Ubuntu

↓

Need Shared Files
```

```
Amazon Linux

↓

Need Shared Files
```

```
RHEL

↓

Need Shared Files
```

Instead of copying files between servers,

all three systems can mount the same shared file system using NFS.

---

# Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Share files across a network |
| **Used By Amazon EFS** | ✅ Yes |
| **Default Port** | TCP 2049 |
| **Works Across** | Linux Servers |

---

# Relationship with Previous Topics

```text
EC2

↓

NFS Protocol

↓

Mount Target

↓

Amazon EFS
```

Without NFS,

the EC2 instance cannot communicate with Amazon EFS.

---

# How Does NFS Work?

Suppose Ubuntu wants to read a file.

```text
Ubuntu EC2

↓

NFS Request

↓

Mount Target

↓

Amazon EFS

↓

File Returned
```

Ubuntu treats the remote file as if it were stored on its own disk.

---

# Architecture

```text
            Amazon EFS

                 │

           NFS Protocol

                 │

        ┌────────┼────────┐

        ▼        ▼        ▼

 Ubuntu EC2   Amazon Linux   RHEL EC2

                  EC2
```

All communication uses **NFS over TCP Port 2049**.

---

# Local Storage vs NFS Storage

## Local Storage

```text
Ubuntu

↓

Own EBS

↓

Only Ubuntu Can Access
```

---

## NFS Storage

```text
Ubuntu

↓

Amazon EFS

↑

Amazon Linux

↑

RHEL
```

Every EC2 instance sees the same files.

---

# Assignment Example

Suppose Ubuntu creates:

```bash
echo "Hello" > hello.txt
```

The file is stored on Amazon EFS.

Now on Amazon Linux:

```bash
cat hello.txt
```

Output:

```text
Hello
```

The file is immediately available because both EC2 instances are using the same NFS-based file system.

---

# Why Does Amazon EFS Use NFS?

AWS chose NFS because it:

- Supports shared file systems.
- Works across multiple Linux distributions.
- Supports simultaneous access by multiple clients.
- Is a mature and widely used protocol.

This makes it ideal for shared storage in cloud environments.

---

# Real DevOps Usage

DevOps Engineers use NFS for:

- Shared application files.
- Shared web content.
- Shared configuration files.
- Shared container storage.
- User home directories.

Amazon EFS provides a managed implementation of NFS.

---

# Production Usage

Typical production architecture:

```text
Application Server 1

        │

Application Server 2

        │

Application Server 3

        │

     NFS (TCP 2049)

        │

     Amazon EFS
```

All application servers access the same shared files.

---

# Troubleshooting

### Problem

Mount command fails.

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| NFS Client Missing | Install NFS utilities. |
| Port 2049 Blocked | Security Group or NACL blocks NFS traffic. |
| Mount Target Missing | No Mount Target available. |

---

### Problem

Permission Denied

### Possible Causes

| Possible Cause | Explanation |
|---------------|-------------|
| Linux File Permissions | Incorrect owner or permissions. |
| Wrong Mount Options | Incorrect NFS mount configuration. |

---

# AWS Console Walkthrough

NFS is **not an AWS service**.

It is automatically used by Amazon EFS.

There is no separate AWS Console page for NFS.

---

# Interview Tips

Remember:

```text
Amazon EBS

↓

Block Storage

Amazon EFS

↓

File Storage

↓

Uses

↓

NFS
```

Whenever someone asks:

> **"Which protocol does Amazon EFS use?"**

The answer is:

```
NFS Version 4.1
```

---

# Common Interview Mistakes

### Mistake 1

❌ NFS is an AWS service.

✅ NFS is an industry-standard file sharing protocol used by Amazon EFS.

---

### Mistake 2

❌ Amazon EFS uses SSH for file sharing.

✅ Amazon EFS uses the NFS protocol over TCP Port 2049.

---

### Mistake 3

❌ NFS is only for Ubuntu.

✅ NFS works with many Linux distributions, including Ubuntu, Amazon Linux, and Red Hat Enterprise Linux.

---

### Mistake 4

❌ NFS replaces Amazon EFS.

✅ Amazon EFS is a managed file system service that uses the NFS protocol.

---

# Interview Cross Questions

### Q1. What does NFS stand for?

**Answer:**

Network File System.

---

### Q2. Which protocol does Amazon EFS use?

**Answer:**

NFS version 4.1.

---

### Q3. Which port does NFS use?

**Answer:**

TCP Port **2049**.

---

### Q4. Why can Ubuntu, Amazon Linux, and RHEL all mount the same Amazon EFS?

**Answer:**

Because they all support the NFS protocol used by Amazon EFS.

---

### Q5. Is NFS an AWS service?

**Answer:**

No.

NFS is a standard network file sharing protocol. Amazon EFS is an AWS-managed service built on that protocol.

---

# Related Topics

- Amazon EFS
- Mount Targets
- Amazon EC2
- Security Group

---

# One-line Interview Answer

> NFS (Network File System) is a standard file sharing protocol that allows multiple systems to access the same remote file system over a network, and Amazon EFS uses NFS version 4.1 for communication.

---

Q41. Frequently Asked Questions (Theory)

These are direct questions that interviewers commonly ask to test your understanding of AWS concepts.

Amazon VPC
Q1. What is a VPC?

Answer:

A VPC (Virtual Private Cloud) is a logically isolated virtual network in AWS where you can launch and manage AWS resources.

Q2. Why do we need a VPC?

Answer:

A VPC provides network isolation, IP address management, routing, and security for AWS resources.

Q3. Can two VPCs communicate by default?

Answer:

No.

They require services such as VPC Peering or a Transit Gateway to communicate.

Q4. What is CIDR?

Answer:

CIDR (Classless Inter-Domain Routing) is a notation used to define an IP address range for a network.

Q5. What is the difference between a VPC and a Subnet?

Answer:

A VPC defines the entire virtual network, while a subnet divides that network into smaller segments within a single Availability Zone.

Subnet
Q6. What is a Public Subnet?

Answer:

A subnet whose Route Table contains a route to an Internet Gateway.

Q7. What is a Private Subnet?

Answer:

A subnet without a direct route to an Internet Gateway.

Q8. Can a subnet span multiple Availability Zones?

Answer:

No.

A subnet belongs to exactly one Availability Zone.

Internet Gateway
Q9. What is an Internet Gateway?

Answer:

An Internet Gateway enables communication between a VPC and the Internet.

Q10. Can a Private Subnet directly access the Internet through an Internet Gateway?

Answer:

No.

A Private Subnet requires a NAT Gateway for outbound Internet access.

NAT Gateway
Q11. Why is a NAT Gateway placed in a Public Subnet?

Answer:

Because the NAT Gateway itself requires Internet access through an Internet Gateway to forward outbound traffic.

Q12. Can inbound Internet traffic reach a Private EC2 through a NAT Gateway?

Answer:

No.

A NAT Gateway supports outbound connections only.

Route Table
Q13. What is the purpose of a Route Table?

Answer:

A Route Table determines where network traffic is forwarded based on destination IP addresses.

Q14. What does the Local Route do?

Answer:

It allows communication between resources inside the same VPC.

Security Group
Q15. Is a Security Group Stateful?

Answer:

Yes.

Return traffic is automatically allowed.

Q16. Does a Security Group support Deny rules?

Answer:

No.

It supports Allow rules only.

Network ACL
Q17. Is a Network ACL Stateful?

Answer:

No.

Inbound and outbound rules are evaluated separately.

Q18. Does a Network ACL support Deny rules?

Answer:

Yes.

It supports both Allow and Deny rules.

EC2
Q19. What is Amazon EC2?

Answer:

Amazon EC2 is an Infrastructure as a Service (IaaS) offering that provides virtual servers in the AWS cloud.

Q20. What is an AMI?

Answer:

An Amazon Machine Image is a bootable template used to launch EC2 instances.

Q21. What is an Instance Type?

Answer:

It defines the virtual hardware configuration of an EC2 instance, including CPU, memory, and networking.

Q22. What is Amazon EBS?

Answer:

Amazon EBS is persistent block storage for EC2 instances.

Q23. What is an ENI?

Answer:

An Elastic Network Interface is a virtual network adapter that connects an EC2 instance to a VPC.

Q24. What is a Key Pair?

Answer:

A Public and Private cryptographic key pair used for secure SSH authentication.

Amazon EFS
Q25. What is Amazon EFS?

Answer:

Amazon EFS is a managed shared file storage service that multiple EC2 instances can mount simultaneously.

Q26. What protocol does Amazon EFS use?

Answer:

NFS version 4.1.

Q27. What is a Mount Target?

Answer:

A Mount Target is the network endpoint that enables EC2 instances to connect to an Amazon EFS File System.

Q28. What is the difference between EBS and EFS?

Answer:

EBS provides block storage for a single EC2 instance.
EFS provides shared file storage for multiple EC2 instances.
Q29. Can Ubuntu, Amazon Linux, and RHEL mount the same Amazon EFS?

Answer:

Yes.

As long as they support NFS and have network connectivity.

Q30. Which port must be opened for Amazon EFS?

Answer:

TCP Port 2049.

Assignment
Q31. Why were three different operating systems used in this assignment?

Answer:

To demonstrate that Amazon EFS provides a platform-independent shared file system that can be mounted by different Linux distributions using NFS.

Q32. Why did every EC2 instance have its own Root EBS volume?

Answer:

Because each EC2 instance requires its own operating system and local storage, while Amazon EFS provides shared storage.

Q33. Why did all EC2 instances see the same file?

Answer:

Because they mounted the same Amazon EFS File System.

Q34. Why is Port 2049 important?

Answer:

Amazon EFS communicates over the NFS protocol, which uses TCP Port 2049.

Q35. Which AWS networking components were involved in this assignment?

Answer:

VPC
Subnet
Route Table
Internet Gateway
Security Group
Network ACL
ENI
Private IP
Public IP
Amazon EFS Mount Target

------
