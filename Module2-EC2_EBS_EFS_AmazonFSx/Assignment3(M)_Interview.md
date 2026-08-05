
✅ Every abbreviation will have its full form the first time it appears.
✅ Every topic will mention whether it is Hardware, Software, Protocol, AWS Service, or Virtual Component.
✅ Every topic will specify its TCP/IP layer (where applicable).
✅ Every topic will explain how it is used in this assignment.
✅ Every topic will include interview cross-questions.
✅ Every topic will end with a one-line interview answer.
✅ Every comparison will be presented as a table.
✅ Calculations will always be shown in tables, not paragraphs.
✅ Diagrams will be used only where they genuinely improve understanding.
✅ Every command will have a word-by-word explanation (already covered in your command handbook).

--- 

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

# AWS Networking, EC2 & Amazon EFS Interview Handbook

---

# AWS Networking, Amazon EC2 & Amazon EFS Interview Handbook

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

## Prerequisites

None

This is the first concept of computer networking.

---

## Definition

A **Network** is a collection of **two or more devices** connected together to **communicate** and **share resources** using standard networking protocols.

The devices can be physical or virtual.

Examples include:

- Computers
- Servers
- Mobile Phones
- Routers
- Switches
- Cloud Resources (Amazon EC2, Amazon EFS)

---

## Why Do We Need It?

Without a network:

- Devices cannot communicate.
- Files cannot be shared.
- Internet cannot be accessed.
- Applications cannot communicate.
- Cloud services like AWS cannot function.

A network provides a communication path that allows devices to exchange information efficiently.

---

## Quick Revision

| Item | Description |
|------|-------------|
| **Purpose** | Communication and Resource Sharing |
| **Communication Medium** | Wired or Wireless |
| **Uses** | File Sharing, Remote Access, Internet, Cloud Computing |
| **Examples** | Home Wi-Fi, Office LAN, Internet, AWS VPC |

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

A network works by allowing devices to exchange data using standard communication protocols.

The communication process is:

```text
Device A
    │
    ▼
Network
    │
    ▼
Device B
```

Every device connected to the network has an identity (IP Address), and communication follows predefined networking protocols such as TCP/IP.

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

## Real DevOps Usage

Networking is one of the most frequently used skills in DevOps.

A DevOps Engineer uses networking to:

- Connect applications to databases.
- Connect application servers to shared storage.
- Configure CI/CD servers.
- Troubleshoot connectivity issues.
- Secure cloud infrastructure.
- Design scalable cloud architectures.

Almost every production issue involves networking in some form.

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

| Possible Cause | Explanation |
|---------------|-------------|
| Different VPC | EC2 and EFS are not in the same VPC. |
| Security Group | NFS Port `2049` is blocked. |
| Network ACL | Traffic is denied. |
| Route Table | Incorrect routing configuration. |
| Mount Target | EFS Mount Target is unavailable. |
| DNS Resolution | EFS DNS name cannot be resolved. |

---

## Interview Tip

A network **only provides the communication path**.

The communication rules are defined by networking protocols such as:

- TCP
- UDP
- IP

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

## One-line Interview Answer

> A Network is a collection of interconnected devices that communicate and share resources using standard networking protocols.
