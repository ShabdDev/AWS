# Module 8 - CloudFormation and App Services

# Case Study – Multi-Tier Architecture

# Part 1: Requirements Analysis, Cost Check, Architecture Design, and CloudFormation Foundation

## Problem Statement

You work for XYZ Corporation. Your corporation wants to launch a new web-based application. The development team has prepared the code, but it has not been tested yet. The development team needs the system administrators to build a web server to test the code, but the system administrators are not available.

The objective of this case study is to use AWS CloudFormation to provision a reusable multi-tier infrastructure so that developers can create the required testing environment without manually depending on system administrators.

## Tasks To Be Performed

1. **Web Tier:** Launch an EC2 instance in a public subnet. The instance must allow:
   - HTTP traffic from the internet on port `80`.
   - SSH traffic from the internet on port `22`.

2. **Application Tier:** Launch an EC2 instance in a private subnet behind the Web Tier. It must allow:
   - SSH traffic only from the Web Tier.

3. **Database Tier:** Launch an Amazon RDS for MySQL DB instance in private subnets. It must allow:
   - MySQL connections on port `3306` only from the Application Tier.

4. **DNS Tier:** Set up an Amazon Route 53 public hosted zone and direct traffic to the Web Tier EC2 instance.

The proposed solution must also satisfy these requirements:

1. The development team must be able to provision the complete testing infrastructure without involving system administrators.

2. Developers should spend their time testing code rather than manually provisioning, configuring, and updating infrastructure.

3. When the development team deletes the CloudFormation stack, the RDS DB instance must not be deleted.

---

# 1. Important Architecture Clarifications

The original case-study wording refers to:

- `public subnet of Web Tier-3`
- `private subnet of Application Tier-4`

These phrases appear to describe tier relationships rather than actual AWS subnet names.

For this implementation, the architecture will use explicit resource names:

| Tier | AWS Resource | Network Placement |
| --- | --- | --- |
| Web Tier | Amazon EC2 | Public Subnet |
| Application Tier | Amazon EC2 | Private Application Subnet |
| Database Tier | Amazon RDS for MySQL | Private Database Subnets |
| DNS Tier | Amazon Route 53 | Public Hosted Zone |

Security will be implemented using **security-group references**, which is more precise than allowing entire subnet CIDR ranges.

The resulting security path is:

~~~text
Internet
   |
   | HTTP : 80
   | SSH : 22
   v
Web Tier Security Group
   |
   | SSH : 22
   v
Application Tier Security Group
   |
   | MySQL : 3306
   v
Database Tier Security Group
   |
   v
Amazon RDS for MySQL
~~~

This ensures that:

- The Web Tier accepts HTTP and SSH from the internet as explicitly required by the case study.
- The Application Tier accepts SSH only from instances associated with the Web Tier security group.
- The Database Tier accepts MySQL traffic only from instances associated with the Application Tier security group.

> **Security note:** Allowing SSH from the entire internet using `0.0.0.0/0` satisfies the literal assignment requirement but is not recommended for production. A production environment should restrict SSH to a trusted administrator IP range, use AWS Systems Manager Session Manager, or use another controlled administrative access mechanism.

---

# 2. Proposed Solution for Developer Self-Service

AWS CloudFormation will be used as the Infrastructure as Code service for this case study.

Instead of manually asking a system administrator to create:

- VPC
- Subnets
- Route tables
- Internet gateway
- Security groups
- EC2 instances
- RDS database
- Route 53 DNS records

the development team can deploy one CloudFormation template.

The workflow becomes:

~~~text
Developer
    |
    v
Upload CloudFormation Template
    |
    v
Enter Required Parameters
    |
    v
Create Stack
    |
    v
CloudFormation Automatically Provisions Infrastructure
    |
    +--------------------+
    |                    |
    v                    v
Test Application    Update Stack When Needed
    |
    v
Delete Stack After Testing
    |
    v
Temporary Resources Deleted
    |
    v
RDS DB Retained
~~~

## Why CloudFormation solves the system-administrator dependency problem

CloudFormation allows infrastructure to be defined declaratively in a YAML template.

The development team does not need to manually create every resource through different AWS Console pages. Developers can deploy the predefined infrastructure repeatedly for development and testing.

The same template can be reused for:

- Development
- Testing
- Staging
- Production-like environments

This provides:

- Repeatability
- Consistency
- Infrastructure automation
- Version-controlled infrastructure
- Reduced manual configuration
- Faster environment provisioning
- Easier infrastructure updates

---

# 3. Protecting the RDS Database During Stack Deletion

The RDS resource will use the following CloudFormation resource attributes:

~~~yaml
DeletionPolicy: Retain
UpdateReplacePolicy: Retain
~~~

## Meaning of `DeletionPolicy: Retain`

When the CloudFormation stack is deleted, CloudFormation removes the RDS resource from stack management but does not delete the actual RDS DB instance.

The database continues to exist independently.

## Meaning of `UpdateReplacePolicy: Retain`

If a future stack update changes a property that requires replacement of the RDS DB instance, CloudFormation retains the old database instead of deleting it.

Using both attributes provides protection during:

- Stack deletion.
- Resource replacement caused by stack updates.

> **Important:** A retained RDS DB instance can continue generating AWS charges after the CloudFormation stack has been deleted. It must be manually deleted later if it is no longer required.

---

# 4. Free Tier and Cost Check

AWS Free Tier eligibility depends on factors including:

- AWS account creation date.
- AWS Free Tier plan.
- Remaining promotional credits.
- Region.
- Resource configuration.
- Previous usage during the billing period.

Therefore, this case study must not be assumed to be completely free.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| --- | --- | --- | --- |
| AWS CloudFormation | Generally no additional charge for standard AWS resource provisioning | No direct infrastructure charge for standard AWS resources | The resources created by the stack are billed individually. |
| Amazon VPC | Core VPC components are generally not separately charged | Usually no direct charge for the VPC itself | NAT Gateway is deliberately not used because it can generate additional charges. |
| Amazon EC2 | Depends on account plan and eligible instance usage | Yes, when usage is not covered by Free Tier | Two EC2 instances will be created. |
| Amazon EBS | Depends on eligible storage allowance and account plan | Yes, if outside eligible allowance | EC2 root volumes consume storage. |
| Amazon RDS for MySQL | Depends on account plan, eligible DB instance class, storage, and usage | Yes, when not covered by Free Tier | RDS can continue generating charges while running. |
| Amazon Route 53 Public Hosted Zone | No standard always-free hosted-zone allowance | Yes | Public hosted zones normally have a recurring monthly charge. |
| Route 53 DNS Queries | Not generally free without applicable exceptions | Yes | DNS queries can incur usage-based charges. |
| Elastic IPv4 Address | Public IPv4 addresses can incur hourly charges | Yes | This architecture avoids creating a separate Elastic IP for the EC2 instance. |
| NAT Gateway | Not used | Not applicable | Avoided to reduce unnecessary hourly and data-processing charges. |

## Is this case study completely Free Tier eligible?

**No.**

The complete architecture should not be treated as completely free because the Route 53 public hosted zone is normally billable, and EC2, EBS, RDS, and public IPv4 usage may also incur charges depending on the AWS account's Free Tier plan and remaining allowances or credits.

## Resources most likely to consume credits or generate charges

The most important resources to monitor are:

- Two EC2 instances.
- EC2 EBS root volumes.
- One RDS MySQL DB instance.
- RDS storage.
- Route 53 public hosted zone.
- Route 53 DNS queries.
- Public IPv4 address assigned to the Web Tier EC2 instance.

## Should this assignment be completed while AWS promotional credits are available?

Yes. If promotional AWS credits are available, they can help cover eligible usage generated by the resources in this architecture.

However, credit coverage depends on the terms of the specific promotional credit.

## Which resources should be deleted immediately after verification?

After completing all verification steps:

- Delete the CloudFormation stack.
- Manually delete the retained RDS DB instance if it is no longer needed.
- Delete any remaining DB subnet group if necessary.
- Verify that the Route 53 public hosted zone has been deleted.
- Verify that both EC2 instances have been terminated.
- Verify that unnecessary EBS volumes are not left behind.

Detailed cleanup steps will be provided in the final part.

---

# 5. Architecture Design

The architecture for this case study is:

~~~text
                                  Internet
                                     |
                                     v
                          +----------------------+
                          |      Route 53        |
                          | Public Hosted Zone   |
                          +----------+-----------+
                                     |
                                     v
                          Web EC2 Public DNS/IP
                                     |
                                     v
+-----------------------------------------------------------------------+
|                               Custom VPC                              |
|                             10.0.0.0/16                               |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | Public Subnet: 10.0.1.0/24                                  |   |
|   |                                                               |   |
|   |       +-----------------------------------------------+       |   |
|   |       | Web Tier EC2                                 |       |   |
|   |       | HTTP 80  <- Internet                         |       |   |
|   |       | SSH 22   <- Internet                         |       |   |
|   |       +-----------------------+-----------------------+       |   |
|   +-------------------------------|-------------------------------+   |
|                                   | SSH 22                           |
|                                   v                                  |
|   +---------------------------------------------------------------+   |
|   | Private Application Subnet: 10.0.2.0/24                      |   |
|   |                                                               |   |
|   |       +-----------------------------------------------+       |   |
|   |       | Application Tier EC2                         |       |   |
|   |       | SSH 22 <- Web Tier Security Group            |       |   |
|   |       +-----------------------+-----------------------+       |   |
|   +-------------------------------|-------------------------------+   |
|                                   | MySQL 3306                       |
|                                   v                                  |
|   +---------------------------------------------------------------+   |
|   | Private DB Subnet A: 10.0.3.0/24                            |   |
|   | Private DB Subnet B: 10.0.4.0/24                            |   |
|   |                                                               |   |
|   |       +-----------------------------------------------+       |   |
|   |       | Amazon RDS for MySQL                         |       |   |
|   |       | 3306 <- Application Tier Security Group      |       |   |
|   |       | PubliclyAccessible: false                    |       |   |
|   |       | DeletionPolicy: Retain                       |       |   |
|   |       | UpdateReplacePolicy: Retain                  |       |   |
|   |       +-----------------------------------------------+       |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
~~~

---

# 6. Why Two Private Database Subnets Are Required

Although the case study refers to an RDS MySQL instance in a private subnet, Amazon RDS requires a DB subnet group.

For a standard VPC deployment, the DB subnet group should contain subnets in at least two Availability Zones.

Therefore, this implementation creates:

| Subnet | CIDR | Purpose |
| --- | --- | --- |
| Public Subnet | `10.0.1.0/24` | Web Tier EC2 |
| Private Application Subnet | `10.0.2.0/24` | Application Tier EC2 |
| Private DB Subnet A | `10.0.3.0/24` | RDS DB subnet group |
| Private DB Subnet B | `10.0.4.0/24` | RDS DB subnet group |

The RDS instance remains private because:

- `PubliclyAccessible` is set to `false`.
- Database subnets do not have a route to the Internet Gateway.
- The RDS security group permits port `3306` only from the Application Tier security group.

---

# 7. Dependency Flow

Resources must be created in technical dependency order.

~~~text
AWS Account
    |
    v
Choose AWS Region
    |
    v
Prepare Required Parameters
    |
    +------------------------+
    |                        |
    v                        v
EC2 Key Pair          Registered Domain Name
    |                        |
    +------------+-----------+
                 |
                 v
        CloudFormation Template
                 |
                 v
             Custom VPC
                 |
                 v
    Internet Gateway + Attachment
                 |
                 v
    +------------+--------------------+
    |            |                    |
    v            v                    v
Public       Application          DB Private
Subnet       Private Subnet       Subnets A + B
    |            |                    |
    v            v                    v
Public       Private Route       DB Subnet Group
Route Table  Table
    |
    v
Security Groups
    |
    +-------------------+-------------------+
    |                   |                   |
    v                   v                   v
Web Tier SG      Application Tier SG    Database SG
    |                   |                   |
    v                   v                   v
Web EC2           Application EC2        RDS MySQL
    |                                       |
    v                                       v
Route 53 DNS Record              DeletionPolicy: Retain
    |
    v
Verification
    |
    v
Stack Deletion
    |
    v
Temporary Resources Removed
    |
    v
RDS DB Retained
    |
    v
Manual Final Cleanup
~~~

---

# 8. Important Design Decision: No NAT Gateway

The Application Tier EC2 instance is located in a private subnet.

This case study does not require the Application Tier instance to:

- Download packages from the internet.
- Access public APIs.
- Install application dependencies from public repositories.

Therefore, a NAT Gateway is not required for the stated assignment tasks.

This design intentionally avoids creating a NAT Gateway because it can generate hourly and data-processing charges.

The Application Tier EC2 instance can still receive SSH connections from the Web Tier because both instances communicate through private networking inside the same VPC.

> If a real application server later requires outbound internet access for package installation, external APIs, software updates, or dependency downloads, additional outbound connectivity such as a NAT Gateway may be required.

---

# 9. Route 53 Prerequisite and Important Limitation

A Route 53 **public hosted zone requires a real domain name** to provide useful public DNS resolution.

Examples:

- `example.com`
- `mydevopslab.com`
- `test.example.com`

This case study will parameterize the domain name rather than hard-code a fake domain.

The CloudFormation template will accept:

- `DomainName`
- `RecordName`

Example:

| Parameter | Example Value |
| --- | --- |
| Domain Name | `example.com` |
| Record Name | `app.example.com` |

The Route 53 record will direct traffic to the Web Tier EC2 instance.

> **Important:** If the domain is registered outside Route 53, the domain registrar must use the Route 53 hosted zone's authoritative name servers before public DNS resolution will work.

> **Important:** If no real domain is available, the infrastructure can still be deployed and tested using the Web Tier EC2 instance's public DNS name. However, end-to-end public Route 53 verification requires control of a real domain or delegated subdomain.

---

# 10. CloudFormation Resource Plan

The CloudFormation template will create the following resources:

| Logical Resource | AWS Resource Type | Purpose |
| --- | --- | --- |
| `MultiTierVPC` | `AWS::EC2::VPC` | Custom VPC |
| `InternetGateway` | `AWS::EC2::InternetGateway` | Internet connectivity |
| `InternetGatewayAttachment` | `AWS::EC2::VPCGatewayAttachment` | Attaches Internet Gateway to VPC |
| `PublicSubnet` | `AWS::EC2::Subnet` | Hosts Web Tier EC2 |
| `AppPrivateSubnet` | `AWS::EC2::Subnet` | Hosts Application Tier EC2 |
| `DBPrivateSubnetA` | `AWS::EC2::Subnet` | First RDS subnet |
| `DBPrivateSubnetB` | `AWS::EC2::Subnet` | Second RDS subnet |
| `PublicRouteTable` | `AWS::EC2::RouteTable` | Public subnet routing |
| `DefaultPublicRoute` | `AWS::EC2::Route` | Sends internet-bound traffic to Internet Gateway |
| `PublicSubnetRouteTableAssociation` | `AWS::EC2::SubnetRouteTableAssociation` | Associates public subnet with public route table |
| `PrivateRouteTable` | `AWS::EC2::RouteTable` | Private subnet routing |
| `AppSubnetRouteTableAssociation` | `AWS::EC2::SubnetRouteTableAssociation` | Associates Application subnet |
| `DBSubnetARouteTableAssociation` | `AWS::EC2::SubnetRouteTableAssociation` | Associates first DB subnet |
| `DBSubnetBRouteTableAssociation` | `AWS::EC2::SubnetRouteTableAssociation` | Associates second DB subnet |
| `WebSecurityGroup` | `AWS::EC2::SecurityGroup` | Controls Web Tier traffic |
| `AppSecurityGroup` | `AWS::EC2::SecurityGroup` | Allows SSH from Web Tier |
| `DBSecurityGroup` | `AWS::EC2::SecurityGroup` | Allows MySQL from Application Tier |
| `WebInstance` | `AWS::EC2::Instance` | Web server |
| `AppInstance` | `AWS::EC2::Instance` | Application server |
| `DBSubnetGroup` | `AWS::RDS::DBSubnetGroup` | Provides private DB subnet placement |
| `MySQLDatabase` | `AWS::RDS::DBInstance` | MySQL database |
| `PublicHostedZone` | `AWS::Route53::HostedZone` | DNS hosted zone |
| `WebDNSRecord` | `AWS::Route53::RecordSet` | Directs DNS traffic to Web EC2 |

---

# 11. Prerequisites

Before deploying the CloudFormation template, the following prerequisites are required:

| Prerequisite | Required | Reason |
| --- | --- | --- |
| AWS Account | Yes | Required to create AWS resources |
| IAM permissions | Yes | Required for CloudFormation and dependent AWS services |
| AWS Region selected | Yes | Resources are deployed into a specific region |
| EC2 key pair | Yes | Required for SSH access to EC2 instances |
| Real domain or delegated subdomain | Required for complete Route 53 public verification | Route 53 public DNS requires domain control |
| SSH client | Yes for command-line SSH testing | Used to connect to the Web Tier and then the Application Tier |
| AWS promotional credits | No | Helpful for reducing out-of-pocket costs |

---

# Step 1: Choose the AWS Region

### Navigation

~~~text
AWS Management Console
→ Top navigation bar
→ Region selector
→ Select your preferred AWS Region
~~~

For this implementation, use:

- **Recommended Region:** `Asia Pacific (Mumbai)`
- **Region Code:** `ap-south-1`

### Configuration

| Setting | Value |
| --- | --- |
| AWS Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

### Why is this step required?

Most resources in this case study are regional.

The following resources will be created in the selected AWS Region:

- VPC
- Subnets
- Route tables
- Security groups
- EC2 instances
- RDS database

Selecting the region first prevents accidentally creating related resources in different AWS Regions.

### Dependency

This step has no resource dependency.

It should be completed before creating the EC2 key pair and deploying the CloudFormation stack.

### What happens if this step is skipped?

You may accidentally:

- Create the EC2 key pair in one region.
- Deploy the CloudFormation stack in another region.
- Receive a key-pair-not-found error.
- Create resources farther from your preferred location.
- Have difficulty finding resources later.

---

# Step 2: Create an EC2 Key Pair

The same EC2 key pair will be used for the Web Tier and Application Tier instances.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Key Pairs
→ Create key pair
~~~

### Configuration

| Setting | Value |
| --- | --- |
| Name | `multitier-key` |
| Key pair type | `RSA` |
| Private key file format | `.pem` |

Select **Create key pair**.

The private key file will be downloaded automatically.

Example filename:

`multitier-key.pem`

### Why is this step required?

The CloudFormation template will accept the existing EC2 key-pair name as a parameter.

The Web Tier instance requires SSH access from the internet according to the assignment.

The Application Tier instance will be reached through the Web Tier because the Application Tier has no public IP address.

### Dependency

Depends on:

- Step 1: Correct AWS Region selection.

The key pair must exist in the same AWS Region where the CloudFormation stack is deployed.

### What happens if this step is skipped?

CloudFormation cannot launch EC2 instances using a nonexistent key pair.

Stack creation will fail when attempting to create the EC2 instances.

---

# Step 3: Verify the EC2 Key Pair

### Navigation

~~~text
AWS Management Console
→ EC2
→ Network & Security
→ Key Pairs
~~~

### Action

Confirm that the following key pair appears:

`multitier-key`

Verify that its state and details are visible in the EC2 Console.

### Why is this step required?

This confirms that the key pair exists before it is passed to the CloudFormation stack as a parameter.

### Dependency

Depends on:

- Step 2: Create an EC2 Key Pair.

### What happens if this step is skipped?

The CloudFormation template may later fail if:

- The key pair was created in the wrong region.
- The key pair name was entered incorrectly.
- The key pair was not successfully created.

---

# Step 4: Prepare the Route 53 Domain Information

Before creating the stack, determine which real domain or delegated subdomain will be used.

### Configuration

Example only:

| Setting | Example |
| --- | --- |
| Domain Name | `example.com` |
| Application DNS Record | `app.example.com` |

Do not use `example.com` unless you actually control it.

If you own:

`mydomain.com`

you could use:

| Setting | Value |
| --- | --- |
| Domain Name | `mydomain.com` |
| Application DNS Record | `app.mydomain.com` |

### Why is this step required?

The CloudFormation stack will create:

- A Route 53 public hosted zone.
- An A record pointing to the Web Tier EC2 public IPv4 address.

Without a valid domain or delegated subdomain, the Route 53 portion cannot be completely verified through public DNS.

### Dependency

No AWS infrastructure resource dependency exists yet.

However, complete public DNS verification depends on owning or controlling the specified domain or delegated subdomain.

### What happens if this step is skipped?

The stack cannot create a meaningful public hosted zone without a domain-name parameter.

If a fake domain is used, the Route 53 resources may be created, but normal public DNS resolution will not work.

---

# 12. CloudFormation Template Strategy

The implementation will use one reusable YAML CloudFormation template.

The template will contain:

- `Parameters`
- `Mappings`
- `Resources`
- `Outputs`

## Parameters

The template will accept values such as:

- EC2 key pair name.
- Database username.
- Database password.
- Domain name.
- DNS record name.

## Mappings

A region-to-AMI mapping or a dynamic AWS Systems Manager public parameter will be used so that the EC2 instances receive a valid Amazon Linux AMI for the deployment region.

## Resources

All infrastructure components will be declared under the `Resources` section.

## Outputs

After successful deployment, the stack will display useful values such as:

- VPC ID.
- Web Tier public IP.
- Web Tier public DNS name.
- Application Tier private IP.
- RDS endpoint.
- Route 53 record name.

---

# 13. Planned Security Group Rules

## Web Tier Security Group

| Direction | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| Inbound | TCP | `80` | `0.0.0.0/0` | HTTP from internet |
| Inbound | TCP | `22` | `0.0.0.0/0` | SSH from internet as required by assignment |
| Outbound | All | All | Default allowed outbound traffic | Allows responses and required outbound communication |

## Application Tier Security Group

| Direction | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| Inbound | TCP | `22` | Web Tier Security Group | SSH only from Web Tier |
| Outbound | All | All | Default allowed outbound traffic | Allows communication to permitted destinations |

## Database Tier Security Group

| Direction | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| Inbound | TCP | `3306` | Application Tier Security Group | MySQL only from Application Tier |
| Outbound | All | All | Default allowed outbound traffic | Standard security-group behavior |

The key security chain is:

~~~text
Internet
    |
    | HTTP 80 and SSH 22
    v
WebSecurityGroup
    |
    | SSH 22 only
    v
AppSecurityGroup
    |
    | MySQL 3306 only
    v
DBSecurityGroup
~~~

---

# 14. Planned Network Configuration

| Resource | CIDR / Configuration | Purpose |
| --- | --- | --- |
| VPC | `10.0.0.0/16` | Entire multi-tier network |
| Public Subnet | `10.0.1.0/24` | Web Tier |
| Application Private Subnet | `10.0.2.0/24` | Application Tier |
| Database Private Subnet A | `10.0.3.0/24` | RDS DB subnet group |
| Database Private Subnet B | `10.0.4.0/24` | RDS DB subnet group |
| Public Route | `0.0.0.0/0 → Internet Gateway` | Internet connectivity for Web Tier |
| Application Route Table | Local VPC routing only | Keeps Application Tier private |
| Database Route Table | Local VPC routing only | Keeps Database Tier private |

---

# 15. What Will Be Implemented in the Next Part

Part 2 will provide the complete CloudFormation YAML template in GitHub-compatible Markdown format.

The implementation will include:

- CloudFormation parameters.
- Custom VPC.
- Internet Gateway.
- Four subnets.
- Public and private routing.
- Web Tier security group.
- Application Tier security group.
- Database Tier security group.
- Web EC2 instance.
- Application EC2 instance.
- RDS DB subnet group.
- RDS MySQL instance with retention protection.
- Route 53 public hosted zone.
- Route 53 DNS record.
- CloudFormation outputs.
- Detailed explanation of the major template sections.
- Stack deployment steps in dependency order.
---

# Part 2: Create the Complete CloudFormation YAML Template and Deploy the Multi-Tier Architecture Stack

This part continues directly from Part 1.

The architecture, networking plan, security-group relationships, prerequisites, cost considerations, and dependency flow were already established in Part 1.

In this part, we will:

- Create the complete CloudFormation YAML template.
- Understand the important template sections.
- Upload the template to AWS CloudFormation.
- Configure stack parameters.
- Deploy the complete multi-tier architecture.
- Wait for stack creation to complete successfully.
- Review the stack outputs.

---

# Step 5: Create the CloudFormation YAML Template

Create a new file on your local computer with the following filename:

`multi-tier-architecture.yaml`

The complete CloudFormation template is shown below.

~~~yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: >
  Multi-tier architecture for XYZ Corporation using AWS CloudFormation.
  This template creates a custom VPC, one public subnet for the Web Tier,
  one private subnet for the Application Tier, two private subnets for
  Amazon RDS, security groups, two EC2 instances, an RDS MySQL database,
  a Route 53 public hosted zone, and a DNS record pointing to the Web Tier.
  The RDS DB instance is protected with DeletionPolicy: Retain and
  UpdateReplacePolicy: Retain.

Parameters:

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Existing EC2 key pair used for SSH access to the EC2 instances.

  DBUsername:
    Type: String
    Description: Master username for the Amazon RDS MySQL database.
    Default: adminuser
    MinLength: 1
    MaxLength: 16
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: >
      The database username must begin with a letter and contain only
      alphanumeric characters.

  DBPassword:
    Type: String
    Description: Master password for the Amazon RDS MySQL database.
    NoEcho: true
    MinLength: 8
    MaxLength: 41
    AllowedPattern: '[a-zA-Z0-9]*'
    ConstraintDescription: >
      The database password must contain only alphanumeric characters
      and must be between 8 and 41 characters.

  DomainName:
    Type: String
    Description: >
      Real domain name or delegated subdomain for the Route 53 public hosted zone.
      Example: example.com

  RecordName:
    Type: String
    Description: >
      Fully qualified DNS record name for the Web Tier.
      Example: app.example.com

Resources:

  MultiTierVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MultiTier-VPC

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MultiTier-IGW

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MultiTierVPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MultiTierVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select
        - 0
        - !GetAZs ''
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: Web-Public-Subnet

  AppPrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MultiTierVPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select
        - 0
        - !GetAZs ''
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: App-Private-Subnet

  DBPrivateSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MultiTierVPC
      CidrBlock: 10.0.3.0/24
      AvailabilityZone: !Select
        - 0
        - !GetAZs ''
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: DB-Private-Subnet-A

  DBPrivateSubnetB:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MultiTierVPC
      CidrBlock: 10.0.4.0/24
      AvailabilityZone: !Select
        - 1
        - !GetAZs ''
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: DB-Private-Subnet-B

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MultiTierVPC
      Tags:
        - Key: Name
          Value: Public-Route-Table

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MultiTierVPC
      Tags:
        - Key: Name
          Value: Private-Route-Table

  AppSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref AppPrivateSubnet
      RouteTableId: !Ref PrivateRouteTable

  DBSubnetARouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DBPrivateSubnetA
      RouteTableId: !Ref PrivateRouteTable

  DBSubnetBRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DBPrivateSubnetB
      RouteTableId: !Ref PrivateRouteTable

  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH from the internet to the Web Tier.
      VpcId: !Ref MultiTierVPC
      SecurityGroupIngress:
        - Description: Allow HTTP from the internet.
          IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

        - Description: Allow SSH from the internet as required by the assignment.
          IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

      Tags:
        - Key: Name
          Value: Web-Tier-SG

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH only from the Web Tier security group.
      VpcId: !Ref MultiTierVPC
      SecurityGroupIngress:
        - Description: Allow SSH only from the Web Tier.
          IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref WebSecurityGroup

      Tags:
        - Key: Name
          Value: App-Tier-SG

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow MySQL only from the Application Tier security group.
      VpcId: !Ref MultiTierVPC
      SecurityGroupIngress:
        - Description: Allow MySQL only from the Application Tier.
          IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup

      Tags:
        - Key: Name
          Value: DB-Tier-SG

  WebInstance:
    Type: AWS::EC2::Instance
    DependsOn:
      - DefaultPublicRoute
      - PublicSubnetRouteTableAssociation
    Properties:
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebSecurityGroup

      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          dnf install -y httpd
          systemctl enable httpd
          systemctl start httpd
          cat > /var/www/html/index.html <<'EOF'
          <!DOCTYPE html>
          <html>
          <head>
              <title>XYZ Corporation Multi-Tier Application</title>
          </head>
          <body>
              <h1>XYZ Corporation Multi-Tier Architecture</h1>
              <p>Web Tier deployed successfully using AWS CloudFormation.</p>
              <p>This page is served from the public Web Tier EC2 instance.</p>
          </body>
          </html>
          EOF

      Tags:
        - Key: Name
          Value: Web-Tier-EC2

  AppInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      SubnetId: !Ref AppPrivateSubnet
      SecurityGroupIds:
        - !Ref AppSecurityGroup

      Tags:
        - Key: Name
          Value: App-Tier-EC2

  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Private subnet group for the Multi-Tier MySQL database.
      SubnetIds:
        - !Ref DBPrivateSubnetA
        - !Ref DBPrivateSubnetB

      Tags:
        - Key: Name
          Value: MultiTier-DB-Subnet-Group

  MySQLDatabase:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      DBInstanceIdentifier: xyz-multitier-mysql
      Engine: mysql
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 20
      StorageType: gp2
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      Port: 3306
      PubliclyAccessible: false
      MultiAZ: false
      BackupRetentionPeriod: 0
      DeleteAutomatedBackups: true
      DBSubnetGroupName: !Ref DBSubnetGroup
      VPCSecurityGroups:
        - !Ref DBSecurityGroup

      Tags:
        - Key: Name
          Value: MultiTier-MySQL-Database

  PublicHostedZone:
    Type: AWS::Route53::HostedZone
    Properties:
      Name: !Ref DomainName

      HostedZoneConfig:
        Comment: Public hosted zone created by the Multi-Tier CloudFormation stack.

      HostedZoneTags:
        - Key: Name
          Value: MultiTier-Public-Hosted-Zone

  WebDNSRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: !Ref PublicHostedZone
      Name: !Ref RecordName
      Type: A
      TTL: '300'
      ResourceRecords:
        - !GetAtt WebInstance.PublicIp

Outputs:

  VPCId:
    Description: ID of the custom VPC.
    Value: !Ref MultiTierVPC

  PublicSubnetId:
    Description: ID of the public Web Tier subnet.
    Value: !Ref PublicSubnet

  AppPrivateSubnetId:
    Description: ID of the private Application Tier subnet.
    Value: !Ref AppPrivateSubnet

  WebInstanceId:
    Description: Instance ID of the Web Tier EC2 instance.
    Value: !Ref WebInstance

  WebInstancePublicIP:
    Description: Public IPv4 address of the Web Tier EC2 instance.
    Value: !GetAtt WebInstance.PublicIp

  WebInstancePublicDNS:
    Description: Public DNS name of the Web Tier EC2 instance.
    Value: !GetAtt WebInstance.PublicDnsName

  AppInstanceId:
    Description: Instance ID of the Application Tier EC2 instance.
    Value: !Ref AppInstance

  AppInstancePrivateIP:
    Description: Private IPv4 address of the Application Tier EC2 instance.
    Value: !GetAtt AppInstance.PrivateIp

  RDSEndpoint:
    Description: Endpoint address of the Amazon RDS MySQL database.
    Value: !GetAtt MySQLDatabase.Endpoint.Address

  RDSPort:
    Description: Port used by the Amazon RDS MySQL database.
    Value: !GetAtt MySQLDatabase.Endpoint.Port

  HostedZoneId:
    Description: ID of the Route 53 public hosted zone.
    Value: !Ref PublicHostedZone

  ApplicationURL:
    Description: HTTP URL for accessing the Web Tier through Route 53.
    Value: !Sub 'http://${RecordName}'

  DirectWebURL:
    Description: Direct HTTP URL using the Web Tier public IPv4 address.
    Value: !Sub 'http://${WebInstance.PublicIp}'
~~~

---

# 16. Understanding the CloudFormation Template

Before deploying the template, it is important to understand the major sections.

---

## 16.1 `AWSTemplateFormatVersion`

~~~yaml
AWSTemplateFormatVersion: '2010-09-09'
~~~

This specifies the CloudFormation template format version.

`2010-09-09` is the supported CloudFormation template format version.

It does not represent:

- The creation date of your stack.
- The AWS service API version.
- The deployment date.

---

## 16.2 `Description`

The `Description` section explains the purpose of the CloudFormation template.

It helps users understand what infrastructure the template creates before deploying it.

---

## 16.3 `Parameters`

Parameters allow values to be supplied when the stack is created instead of hard-coding every environment-specific value.

This template accepts:

| Parameter | Purpose |
| --- | --- |
| `KeyName` | Existing EC2 key pair |
| `DBUsername` | RDS master username |
| `DBPassword` | RDS master password |
| `DomainName` | Route 53 hosted-zone domain |
| `RecordName` | DNS record for the Web Tier |

This makes the template reusable.

For example, the same template could be deployed with different:

- Key pairs.
- Database credentials.
- Domain names.
- DNS records.

---

## 16.4 Dynamic Amazon Linux AMI Resolution

The EC2 instances use:

~~~yaml
ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'
~~~

This uses an AWS Systems Manager public parameter to dynamically obtain the current Amazon Linux 2023 x86_64 AMI ID for the selected region.

This is better than hard-coding an AMI ID because AMI IDs differ between AWS Regions and change when AWS publishes newer images.

### Component Breakdown

| Component | Explanation |
| --- | --- |
| `ImageId` | EC2 property specifying the machine image |
| `!Sub` | CloudFormation substitution intrinsic function |
| `resolve:ssm` | Tells CloudFormation to resolve a Systems Manager Parameter Store value |
| `/aws/service/ami-amazon-linux-latest/...` | AWS-managed public parameter containing the AMI ID |
| `al2023` | Amazon Linux 2023 |
| `x86_64` | 64-bit x86 architecture |

---

## 16.5 VPC and Subnets

The template creates:

~~~text
VPC
10.0.0.0/16
    |
    +-- Web-Public-Subnet
    |   10.0.1.0/24
    |
    +-- App-Private-Subnet
    |   10.0.2.0/24
    |
    +-- DB-Private-Subnet-A
    |   10.0.3.0/24
    |
    +-- DB-Private-Subnet-B
        10.0.4.0/24
~~~

Only the public subnet has:

~~~yaml
MapPublicIpOnLaunch: true
~~~

The private subnets use:

~~~yaml
MapPublicIpOnLaunch: false
~~~

Therefore:

- The Web Tier receives a public IPv4 address.
- The Application Tier does not receive a public IPv4 address.
- The RDS database remains private.

---

## 16.6 Internet Gateway and Public Route

The following route provides internet connectivity:

~~~yaml
DestinationCidrBlock: 0.0.0.0/0
GatewayId: !Ref InternetGateway
~~~

### Meaning

| Value | Explanation |
| --- | --- |
| `0.0.0.0/0` | Represents all IPv4 destinations |
| `GatewayId` | Specifies the Internet Gateway as the route target |
| `!Ref InternetGateway` | Retrieves the ID of the Internet Gateway created by the template |

Only the public subnet is associated with the public route table.

The Application and Database subnets use a private route table containing only the automatically created local VPC route.

---

## 16.7 Security Group References

The Application Tier does not allow SSH from a CIDR range.

Instead, it uses:

~~~yaml
SourceSecurityGroupId: !Ref WebSecurityGroup
~~~

This means port `22` is accepted only when the source network interface belongs to the Web Tier security group.

Similarly, the database security group uses:

~~~yaml
SourceSecurityGroupId: !Ref AppSecurityGroup
~~~

Therefore, MySQL port `3306` is accessible only from the Application Tier security group.

The security chain is:

~~~text
Internet
    |
    | TCP 80 and TCP 22
    v
WebSecurityGroup
    |
    | TCP 22
    v
AppSecurityGroup
    |
    | TCP 3306
    v
DBSecurityGroup
~~~

---

## 16.8 Web Tier User Data

The Web EC2 instance uses EC2 User Data to automatically install and start Apache.

The User Data script is:

~~~bash
#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
cat > /var/www/html/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>XYZ Corporation Multi-Tier Application</title>
</head>
<body>
    <h1>XYZ Corporation Multi-Tier Architecture</h1>
    <p>Web Tier deployed successfully using AWS CloudFormation.</p>
    <p>This page is served from the public Web Tier EC2 instance.</p>
</body>
</html>
EOF
~~~

### Command 1

~~~bash
#!/bin/bash
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `#!` | Shebang notation that identifies the interpreter used to execute the script |
| `/bin/bash` | Absolute path to the Bash shell interpreter |

### Command 2

~~~bash
dnf install -y httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `dnf` | Package manager used by Amazon Linux 2023 |
| `install` | Installs a software package |
| `-y` | Automatically answers yes to installation confirmation prompts |
| `httpd` | Apache HTTP Server package |

### Command 3

~~~bash
systemctl enable httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `systemctl` | Controls services managed by systemd |
| `enable` | Configures a service to start automatically during system boot |
| `httpd` | Apache HTTP Server service |

### Command 4

~~~bash
systemctl start httpd
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `systemctl` | Controls systemd services |
| `start` | Starts the specified service immediately |
| `httpd` | Apache HTTP Server service |

### Command 5

~~~bash
cat > /var/www/html/index.html <<'EOF'
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `cat` | Reads or writes text data depending on redirection usage |
| `>` | Redirects output and overwrites the destination file |
| `/var/www/html/index.html` | Default Apache web-page file created by this command |
| `<<` | Here-document operator that supplies multiple lines as command input |
| `'EOF'` | Delimiter marking the beginning and end of the here-document content; quoting prevents shell expansion inside the content |

The HTML content between the opening and closing `EOF` markers is written to:

`/var/www/html/index.html`

---

## 16.9 RDS Retention Protection

The most important requirement for stack deletion is implemented here:

~~~yaml
MySQLDatabase:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
~~~

### `DeletionPolicy: Retain`

When the CloudFormation stack is deleted:

- CloudFormation removes the database from stack management.
- The actual RDS DB instance remains in the AWS account.
- The database is not automatically deleted with the stack.

### `UpdateReplacePolicy: Retain`

If a future stack update requires replacing the database:

- The old database is retained.
- CloudFormation does not automatically delete the replaced database.

This directly satisfies the case-study requirement that the RDS DB instance must survive stack deletion.

---

# Step 6: Open AWS CloudFormation

### Navigation

~~~text
AWS Management Console
→ Search for CloudFormation
→ CloudFormation
→ Stacks
→ Create stack
→ With new resources (standard)
~~~

### Why is this step required?

AWS CloudFormation is the Infrastructure as Code service responsible for reading the YAML template and provisioning the complete architecture.

Instead of manually creating every AWS resource, CloudFormation creates them based on the resource declarations and dependencies defined in the template.

### Dependency

Depends on:

- Step 1: AWS Region selected.
- Step 2: EC2 key pair created.
- Step 3: EC2 key pair verified.
- Step 4: Route 53 domain information prepared.
- Step 5: CloudFormation YAML template created.

### What happens if this step is skipped?

The infrastructure will not be provisioned.

The development team would need to manually create every resource, which defeats the main purpose of this case study.

---

# Step 7: Upload the CloudFormation Template

On the **Create stack** page:

### Prerequisite - Prepare template

Select:

`Choose an existing template`

### Specify template

Select:

`Upload a template file`

Choose the following file from your computer:

`multi-tier-architecture.yaml`

Then select:

`Next`

### Configuration

| Setting | Value |
| --- | --- |
| Template source | `Upload a template file` |
| Filename | `multi-tier-architecture.yaml` |

### Why is this step required?

CloudFormation needs the YAML template containing the desired infrastructure definition.

The uploaded template tells CloudFormation:

- Which resources to create.
- How resources are connected.
- Which security rules to apply.
- Which values should be requested as parameters.
- Which outputs should be displayed after deployment.

### Dependency

Depends on:

- Step 5: CloudFormation template creation.

### What happens if this step is skipped?

CloudFormation has no infrastructure definition to deploy.

---

# Step 8: Specify Stack Details

On the **Specify stack details** page, enter the stack name.

### Configuration

| Setting | Value |
| --- | --- |
| Stack name | `MultiTierArchitectureStack` |

Then configure the parameters.

Example:

| Parameter | Example Value |
| --- | --- |
| `KeyName` | `multitier-key` |
| `DBUsername` | `adminuser` |
| `DBPassword` | `StrongDBPass123` |
| `DomainName` | Your actual domain, for example `yourdomain.com` |
| `RecordName` | Your actual DNS record, for example `app.yourdomain.com` |

> Replace the domain examples with a domain or delegated subdomain that you actually control.

> Do not use the example database password for production. Use a strong unique password.

### Why is this step required?

CloudFormation parameters provide deployment-specific values without modifying the YAML template.

This allows the same template to be reused across different environments.

### Dependency

Depends on:

- Existing EC2 key pair.
- Valid database credentials.
- Domain information.

### What happens if this step is skipped?

The stack cannot be created because required parameter values will be missing.

---

# Step 9: Configure Stack Options

Select **Next** to reach the stack options page.

For this assignment, no special stack option is required.

Leave optional settings at their defaults unless your course or AWS account requires otherwise.

You may optionally add stack tags.

Example:

| Key | Value |
| --- | --- |
| `Project` | `MultiTierArchitecture` |
| `Environment` | `Testing` |

Do not configure an IAM service role unless one is specifically required by your AWS account or organization.

Select:

`Next`

### Why is this step required?

This page allows optional stack-level configuration such as:

- Tags.
- IAM service roles.
- Stack failure options.
- Rollback behavior.
- Notification options.

The default configuration is sufficient for this assignment in a normal AWS account with adequate IAM permissions.

### Dependency

Depends on:

- Step 8: Stack details and parameters.

### What happens if this step is skipped?

You cannot proceed to the final review and stack creation page.

---

# Step 10: Review and Create the CloudFormation Stack

On the final review page, verify:

| Item | Expected Value |
| --- | --- |
| Stack name | `MultiTierArchitectureStack` |
| Key pair | Your existing key pair |
| DB username | Your chosen username |
| DB password | Hidden because `NoEcho: true` |
| Domain name | Your controlled domain or delegated subdomain |
| Record name | Your desired application DNS name |

Review the template and configuration carefully.

Then select:

`Submit`

Depending on the current AWS Console wording, the final action may appear as **Submit** or **Create stack**.

### Why is this step required?

This action starts the actual CloudFormation deployment.

CloudFormation begins creating resources in dependency order.

### Dependency

Depends on all previous implementation steps.

### What happens if this step is skipped?

The template remains undeployed and no architecture is created.

---

# Step 11: Monitor Stack Creation Events

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Events
~~~

The stack initially enters:

~~~text
CREATE_IN_PROGRESS
~~~

CloudFormation begins creating resources.

Typical events may include:

~~~text
MultiTierVPC                         CREATE_IN_PROGRESS
InternetGateway                     CREATE_IN_PROGRESS
PublicSubnet                        CREATE_IN_PROGRESS
AppPrivateSubnet                    CREATE_IN_PROGRESS
DBPrivateSubnetA                    CREATE_IN_PROGRESS
DBPrivateSubnetB                    CREATE_IN_PROGRESS
WebSecurityGroup                    CREATE_IN_PROGRESS
AppSecurityGroup                    CREATE_IN_PROGRESS
DBSecurityGroup                     CREATE_IN_PROGRESS
WebInstance                         CREATE_IN_PROGRESS
AppInstance                         CREATE_IN_PROGRESS
DBSubnetGroup                       CREATE_IN_PROGRESS
MySQLDatabase                       CREATE_IN_PROGRESS
PublicHostedZone                    CREATE_IN_PROGRESS
WebDNSRecord                        CREATE_IN_PROGRESS
~~~

The RDS DB instance usually takes significantly longer to create than VPC components or security groups.

Wait until the main stack status becomes:

~~~text
CREATE_COMPLETE
~~~

### Why is this step required?

CloudFormation stack creation is asynchronous.

The stack should not be considered successfully deployed until it reaches `CREATE_COMPLETE`.

### Dependency

Depends on:

- Step 10: Stack submission.

### What happens if this step is skipped?

You may attempt verification while resources are still being created.

For example:

- EC2 instances may not yet be running.
- Apache User Data may still be executing.
- RDS may still be provisioning.
- DNS records may not yet exist.

---

# Step 12: Handle a CloudFormation Creation Failure

If the stack reaches:

~~~text
CREATE_FAILED
~~~

or:

~~~text
ROLLBACK_IN_PROGRESS
~~~

do not immediately retry without identifying the root cause.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Events
~~~

Find the first resource showing:

`CREATE_FAILED`

Read its **Status reason**.

Common possible causes include:

| Error | Possible Cause |
| --- | --- |
| Key pair does not exist | Key pair is missing or belongs to another AWS Region |
| Invalid AMI or SSM resolution failure | AMI parameter cannot be resolved in the selected environment |
| DB instance identifier already exists | An RDS instance named `xyz-multitier-mysql` already exists |
| Insufficient IAM permissions | Current IAM identity cannot create one or more required resources |
| RDS capacity error | Selected DB instance class temporarily unavailable |
| Hosted zone conflict or invalid name | Domain parameter is malformed |
| Resource limit exceeded | AWS account quota has been reached |
| Unsupported instance type | Selected EC2 or RDS instance class is unavailable in the selected region or account |

### Why is this step required?

CloudFormation events provide the actual resource-level failure reason.

Guessing or repeatedly redeploying without reading the events can:

- Waste time.
- Consume additional credits.
- Leave retained resources behind.
- Cause repeated stack failures.

### Dependency

Only required if stack creation fails.

### What happens if this step is skipped?

The root cause remains unresolved, and subsequent deployment attempts may fail for the same reason.

---

# Step 13: Confirm the Stack Reaches `CREATE_COMPLETE`

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select MultiTierArchitectureStack
~~~

### Expected Stack Status

~~~text
CREATE_COMPLETE
~~~

### What should I observe?

The stack should display:

- Stack name: `MultiTierArchitectureStack`.
- Status: `CREATE_COMPLETE`.
- Resources tab containing the created AWS resources.
- Outputs tab containing generated infrastructure values.

### Why does this confirm success?

`CREATE_COMPLETE` confirms that CloudFormation successfully created all resources managed by the stack.

This includes:

- VPC.
- Internet Gateway.
- Subnets.
- Route tables.
- Security groups.
- Web Tier EC2 instance.
- Application Tier EC2 instance.
- RDS MySQL DB instance.
- Route 53 public hosted zone.
- Route 53 DNS record.

---

# Step 14: Review the CloudFormation Stack Resources

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Resources
~~~

### Expected Resources

You should see resources corresponding to:

~~~text
MultiTierVPC
InternetGateway
InternetGatewayAttachment
PublicSubnet
AppPrivateSubnet
DBPrivateSubnetA
DBPrivateSubnetB
PublicRouteTable
DefaultPublicRoute
PublicSubnetRouteTableAssociation
PrivateRouteTable
AppSubnetRouteTableAssociation
DBSubnetARouteTableAssociation
DBSubnetBRouteTableAssociation
WebSecurityGroup
AppSecurityGroup
DBSecurityGroup
WebInstance
AppInstance
DBSubnetGroup
MySQLDatabase
PublicHostedZone
WebDNSRecord
~~~

### What should I observe?

Each successfully created resource should show:

`CREATE_COMPLETE`

### Why does this confirm success?

This proves that CloudFormation created the individual infrastructure components declared in the template.

---

# Step 15: Review the CloudFormation Stack Outputs

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
~~~

### Expected Outputs

| Output Key | Purpose |
| --- | --- |
| `VPCId` | Custom VPC ID |
| `PublicSubnetId` | Public Web Tier subnet ID |
| `AppPrivateSubnetId` | Private Application Tier subnet ID |
| `WebInstanceId` | Web Tier EC2 instance ID |
| `WebInstancePublicIP` | Public IPv4 address of Web Tier |
| `WebInstancePublicDNS` | Public DNS name of Web Tier |
| `AppInstanceId` | Application Tier EC2 instance ID |
| `AppInstancePrivateIP` | Private IPv4 address of Application Tier |
| `RDSEndpoint` | RDS MySQL endpoint |
| `RDSPort` | MySQL port |
| `HostedZoneId` | Route 53 hosted-zone ID |
| `ApplicationURL` | Route 53-based application URL |
| `DirectWebURL` | Direct Web Tier URL using public IPv4 |

Example:

~~~text
VPCId                  vpc-0123456789abcdef0
WebInstancePublicIP    13.XXX.XXX.XXX
AppInstancePrivateIP   10.0.2.XXX
RDSEndpoint             xyz-multitier-mysql.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com
RDSPort                 3306
ApplicationURL          http://app.yourdomain.com
DirectWebURL            http://13.XXX.XXX.XXX
~~~

### What should I observe?

Verify that:

- The Web Tier has a public IPv4 address.
- The Application Tier has a private IPv4 address in `10.0.2.0/24`.
- The RDS endpoint is generated.
- The RDS port is `3306`.
- The Route 53 application URL matches the record name entered during stack creation.

### Why does this confirm success?

These outputs prove that the core resources were created and that CloudFormation successfully retrieved their generated attributes.

---

# 17. Important Note Before Verification

The next part will perform complete verification of every case-study requirement.

Verification will include:

- Confirming the Web Tier EC2 instance is in the public subnet.
- Confirming the Web Tier has a public IPv4 address.
- Confirming HTTP access from the internet.
- Confirming SSH access to the Web Tier.
- Confirming the Application Tier is in a private subnet.
- Confirming the Application Tier has no public IPv4 address.
- Confirming Application Tier SSH is allowed only from the Web Tier security group.
- Connecting from the Web Tier to the private Application Tier.
- Confirming the RDS DB instance is private.
- Confirming port `3306` is allowed only from the Application Tier security group.
- Testing TCP connectivity from the Application Tier to the RDS endpoint.
- Verifying the Route 53 hosted zone.
- Verifying the Route 53 DNS record.
- Testing the application through the Route 53 domain.
- Verifying that the RDS DB instance survives CloudFormation stack deletion.
- Performing final cleanup in reverse dependency order.

---
# Part 3: Verify the Complete Multi-Tier Architecture, Test Connectivity, Verify RDS Retention, and Perform Final Cleanup

This part continues directly from Part 2.

The CloudFormation stack should already have reached:

~~~text
CREATE_COMPLETE
~~~

The stack has created:

- Custom VPC.
- Public Web Tier subnet.
- Private Application Tier subnet.
- Two private Database Tier subnets.
- Internet Gateway.
- Public and private route tables.
- Web Tier security group.
- Application Tier security group.
- Database Tier security group.
- Web Tier EC2 instance.
- Application Tier EC2 instance.
- Amazon RDS for MySQL DB instance.
- Route 53 public hosted zone.
- Route 53 DNS record.

This part verifies every case-study requirement before deleting any resources.

---

# 18. Verification Order

Verification will be performed in the following dependency order:

~~~text
CloudFormation Stack
        |
        v
Verify VPC and Subnets
        |
        v
Verify Web Tier EC2
        |
        v
Test HTTP Access
        |
        v
Test SSH to Web Tier
        |
        v
Verify Application Tier EC2
        |
        v
Verify Web-to-App Security Rule
        |
        v
Test Web-to-App SSH
        |
        v
Verify RDS MySQL
        |
        v
Verify App-to-RDS Security Rule
        |
        v
Test TCP Port 3306 Connectivity
        |
        v
Verify Route 53 Hosted Zone and Record
        |
        v
Test DNS Resolution and HTTP Access
        |
        v
Delete CloudFormation Stack
        |
        v
Verify RDS Was Retained
        |
        v
Manually Delete Retained RDS
        |
        v
Final Cost and Resource Verification
~~~

---

# Verification Step 1: Verify the Custom VPC

### Action

Open the Amazon VPC console and verify that the custom VPC created by CloudFormation exists.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Your VPCs
→ Select MultiTier-VPC
~~~

### Expected Configuration

| Setting | Expected Value |
| --- | --- |
| Name | `MultiTier-VPC` |
| IPv4 CIDR | `10.0.0.0/16` |
| DNS resolution | `Enabled` |
| DNS hostnames | `Enabled` |

### What should I observe?

The VPC should have the CIDR block:

`10.0.0.0/16`

The VPC ID should match the `VPCId` value shown in:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
~~~

### Why does this confirm success?

All Web, Application, and Database Tier resources depend on the custom VPC. Verifying it confirms that the fundamental network boundary for the architecture was successfully created.

---

# Verification Step 2: Verify All Four Subnets

### Action

Open the subnet list and filter by the VPC ID of `MultiTier-VPC`.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Subnets
→ Filter by VPC
→ Select MultiTier-VPC
~~~

### Expected Configuration

| Subnet Name | CIDR Block | Public IPv4 Auto-Assign | Purpose |
| --- | --- | --- | --- |
| `Web-Public-Subnet` | `10.0.1.0/24` | Enabled | Web Tier |
| `App-Private-Subnet` | `10.0.2.0/24` | Disabled | Application Tier |
| `DB-Private-Subnet-A` | `10.0.3.0/24` | Disabled | RDS |
| `DB-Private-Subnet-B` | `10.0.4.0/24` | Disabled | RDS |

### What should I observe?

All four subnets should belong to the same custom VPC.

The two database subnets should be located in different Availability Zones.

### Why does this confirm success?

This proves that the three-tier network separation was implemented correctly and that the RDS DB subnet group can span at least two Availability Zones.

---

# Verification Step 3: Verify the Public Route Table

### Action

Open the public route table and inspect its routes and subnet associations.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Route tables
→ Select Public-Route-Table
→ Routes
~~~

### Expected Routes

| Destination | Target |
| --- | --- |
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

Then inspect:

~~~text
AWS Management Console
→ VPC
→ Route tables
→ Select Public-Route-Table
→ Subnet associations
~~~

The explicitly associated subnet should be:

`Web-Public-Subnet`

### What should I observe?

Only the Web Tier public subnet should use the public route table.

### Why does this confirm success?

A subnet is public when its route table provides a route to an Internet Gateway and the resource has appropriate public addressing.

This allows the Web Tier EC2 instance to communicate with the internet.

---

# Verification Step 4: Verify the Private Route Table

### Action

Inspect the private route table.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Route tables
→ Select Private-Route-Table
→ Routes
~~~

### Expected Route

| Destination | Target |
| --- | --- |
| `10.0.0.0/16` | `local` |

There should be no route similar to:

~~~text
0.0.0.0/0 → Internet Gateway
~~~

The associated private subnets should include:

- `App-Private-Subnet`
- `DB-Private-Subnet-A`
- `DB-Private-Subnet-B`

### What should I observe?

The Application and Database Tier subnets should have local VPC connectivity but no direct route to the Internet Gateway.

### Why does this confirm success?

This confirms that the Application and Database tiers are placed in private subnets and are not directly internet-routable through an Internet Gateway.

---

# Verification Step 5: Verify the Web Tier EC2 Instance

### Action

Open the EC2 console and inspect the Web Tier instance.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select Web-Tier-EC2
~~~

### Expected Configuration

| Setting | Expected Value |
| --- | --- |
| Name | `Web-Tier-EC2` |
| Instance state | `Running` |
| Instance type | `t2.micro` |
| Subnet | `Web-Public-Subnet` |
| Private IP range | `10.0.1.x` |
| Public IPv4 address | Present |
| Security group | `Web-Tier-SG` |

### What should I observe?

The Web Tier EC2 instance must:

- Be running.
- Be inside `Web-Public-Subnet`.
- Have a private IPv4 address from `10.0.1.0/24`.
- Have a public IPv4 address.

### Why does this confirm success?

This verifies the first placement requirement of the case study: the Web Tier EC2 instance is running in a public subnet.

---

# Verification Step 6: Verify the Web Tier Security Group

### Action

Inspect the inbound rules of `Web-Tier-SG`.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Security Groups
→ Select Web-Tier-SG
→ Inbound rules
~~~

### Expected Inbound Rules

| Type | Protocol | Port | Source |
| --- | --- | --- | --- |
| HTTP | TCP | `80` | `0.0.0.0/0` |
| SSH | TCP | `22` | `0.0.0.0/0` |

### What should I observe?

Port `80` and port `22` should be accessible from the internet.

### Why does this confirm success?

This directly verifies the first case-study requirement:

> The Web Tier instance should allow HTTP and SSH from the internet.

---

# Verification Step 7: Test HTTP Access to the Web Tier

### Action

Copy the `DirectWebURL` value from the CloudFormation stack outputs.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
→ Copy DirectWebURL
~~~

Open the URL in a browser.

Example:

~~~text
http://13.XXX.XXX.XXX
~~~

### Expected Output

The browser should display:

~~~text
XYZ Corporation Multi-Tier Architecture

Web Tier deployed successfully using AWS CloudFormation.

This page is served from the public Web Tier EC2 instance.
~~~

### What should I observe?

The Apache web page should load successfully over HTTP.

If it does not load immediately after stack creation, wait approximately one or two minutes because EC2 User Data may still be installing and starting Apache.

### Why does this confirm success?

Successful browser access proves that:

- The Web EC2 instance is running.
- The public IPv4 address is reachable.
- The public subnet has working internet routing.
- The Internet Gateway is correctly attached.
- Port `80` is allowed.
- Apache is running.
- The automatically created `index.html` file is being served.

---

# Verification Step 8: Test SSH Access to the Web Tier

### Action

Open a terminal in the directory containing:

`multitier-key.pem`

Copy the Web Tier public IPv4 address from the EC2 console or CloudFormation outputs.

For Linux, macOS, Git Bash, or WSL, first restrict the key-file permissions:

~~~bash
chmod 400 multitier-key.pem
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `chmod` | Changes filesystem permissions |
| `400` | Grants read permission to the owner and no permissions to group or others |
| `multitier-key.pem` | Private key file used for EC2 SSH authentication |

Then connect:

~~~bash
ssh -i multitier-key.pem ec2-user@WEB_PUBLIC_IP
~~~

Replace `WEB_PUBLIC_IP` with the actual public IPv4 address.

Example:

~~~bash
ssh -i multitier-key.pem ec2-user@13.XXX.XXX.XXX
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `ssh` | Secure Shell client used for encrypted remote connections |
| `-i` | Specifies the identity file containing the private key |
| `multitier-key.pem` | Private key associated with the EC2 key pair |
| `ec2-user` | Default SSH username for Amazon Linux 2023 |
| `@` | Separates the username from the destination host |
| `WEB_PUBLIC_IP` | Public IPv4 address of the Web Tier EC2 instance |

If prompted:

~~~text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
~~~

Enter:

~~~text
yes
~~~

### Expected Output

A successful session resembles:

~~~text
       __|  __|_  )
       _|  (     /   Amazon Linux 2023
      ___|\___|___|

[ec2-user@ip-10-0-1-xxx ~]$
~~~

### What should I observe?

The shell prompt should show that you are logged into a host with a private IP in the `10.0.1.x` range.

### Why does this confirm success?

This proves that:

- Port `22` is accessible.
- The Web Tier has working internet connectivity.
- The EC2 key pair works.
- SSH authentication succeeds.

---

# Verification Step 9: Verify Apache from Inside the Web Tier

While connected to the Web Tier instance, run:

~~~bash
sudo systemctl status httpd --no-pager
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `sudo` | Runs the following command with elevated administrative privileges |
| `systemctl` | Controls and inspects services managed by systemd |
| `status` | Displays the current status of a service |
| `httpd` | Apache HTTP Server service |
| `--no-pager` | Prints output directly instead of opening an interactive pager |

### Expected Output

Look for:

~~~text
Active: active (running)
~~~

Then test the local web server:

~~~bash
curl http://localhost
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `curl` | Transfers data to or from a URL |
| `http://localhost` | Sends an HTTP request to the local machine on the default HTTP port `80` |

### Expected Output

~~~text
<!DOCTYPE html>
<html>
<head>
    <title>XYZ Corporation Multi-Tier Application</title>
</head>
<body>
    <h1>XYZ Corporation Multi-Tier Architecture</h1>
    <p>Web Tier deployed successfully using AWS CloudFormation.</p>
    <p>This page is served from the public Web Tier EC2 instance.</p>
</body>
</html>
~~~

### What should I observe?

Apache should be `active (running)`, and `curl` should return the HTML page.

### Why does this confirm success?

This directly verifies the web-server process independently of browser connectivity.

---

# Verification Step 10: Verify the Application Tier EC2 Instance

### Action

Open the EC2 console and inspect the Application Tier instance.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select App-Tier-EC2
~~~

### Expected Configuration

| Setting | Expected Value |
| --- | --- |
| Name | `App-Tier-EC2` |
| Instance state | `Running` |
| Instance type | `t2.micro` |
| Subnet | `App-Private-Subnet` |
| Private IPv4 range | `10.0.2.x` |
| Public IPv4 address | None |
| Security group | `App-Tier-SG` |

### What should I observe?

The Application Tier must have no public IPv4 address.

### Why does this confirm success?

This verifies that the Application Tier is private and cannot be directly reached from the public internet.

---

# Verification Step 11: Verify the Application Tier Security Group

### Action

Inspect the inbound rules of `App-Tier-SG`.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Security Groups
→ Select App-Tier-SG
→ Inbound rules
~~~

### Expected Inbound Rule

| Protocol | Port | Source |
| --- | --- | --- |
| TCP | `22` | `Web-Tier-SG` security group ID |

There should not be an SSH rule using:

`0.0.0.0/0`

### What should I observe?

The source should be the security-group ID associated with `Web-Tier-SG`.

### Why does this confirm success?

This directly verifies the second case-study requirement: SSH to the Application Tier is permitted only from the Web Tier.

---

# Verification Step 12: Prepare the Key for Web-to-App SSH Testing

Because the Application Tier is private, it cannot be reached directly from the internet.

The connection path is:

~~~text
Your Computer
      |
      | SSH
      v
Web Tier EC2
      |
      | SSH over private VPC network
      v
Application Tier EC2
~~~

For this assignment lab, one simple testing method is to temporarily copy the private key to the Web Tier.

> **Security note:** Copying a private key onto a bastion or Web server is not recommended for production. Production environments should prefer SSH agent forwarding, AWS Systems Manager Session Manager, EC2 Instance Connect Endpoint, or another controlled access method.

First, exit the existing Web Tier SSH session if necessary:

~~~bash
exit
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `exit` | Terminates the current shell session and returns to the previous terminal |

From your local computer, copy the private key to the Web Tier:

~~~bash
scp -i multitier-key.pem multitier-key.pem ec2-user@WEB_PUBLIC_IP:/home/ec2-user/
~~~

Replace `WEB_PUBLIC_IP` with the actual Web Tier public IPv4 address.

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `scp` | Securely copies files over SSH |
| `-i` | Specifies the SSH identity file |
| First `multitier-key.pem` | Local private key used to authenticate to the Web Tier |
| Second `multitier-key.pem` | Local file being copied |
| `ec2-user` | Remote Amazon Linux username |
| `@` | Separates username and host |
| `WEB_PUBLIC_IP` | Web Tier public IPv4 address |
| `:` | Separates remote host from remote destination path |
| `/home/ec2-user/` | Destination directory on the Web Tier |

Reconnect to the Web Tier:

~~~bash
ssh -i multitier-key.pem ec2-user@WEB_PUBLIC_IP
~~~

Then restrict permissions on the copied key:

~~~bash
chmod 400 /home/ec2-user/multitier-key.pem
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `chmod` | Changes filesystem permissions |
| `400` | Allows only the file owner to read the file |
| `/home/ec2-user/multitier-key.pem` | Absolute path to the copied private key |

### Why is this step required?

The Application Tier has no public IPv4 address. Therefore, SSH must originate from a resource with network connectivity to its private IP address.

The Web Tier provides this path.

### Dependency

Depends on:

- Working SSH access to the Web Tier.
- Both EC2 instances using the same EC2 key pair.
- Application security group allowing SSH from `Web-Tier-SG`.

### What happens if this step is skipped?

The subsequent direct SSH command from the Web Tier to the Application Tier cannot authenticate using this particular testing method.

---

# Verification Step 13: Test SSH from the Web Tier to the Application Tier

Get the Application Tier private IPv4 address from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
→ AppInstancePrivateIP
~~~

From the Web Tier EC2 instance, run:

~~~bash
ssh -i /home/ec2-user/multitier-key.pem ec2-user@APP_PRIVATE_IP
~~~

Replace `APP_PRIVATE_IP` with the actual Application Tier private IPv4 address.

Example:

~~~bash
ssh -i /home/ec2-user/multitier-key.pem ec2-user@10.0.2.25
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `ssh` | Opens an encrypted Secure Shell connection |
| `-i` | Specifies the identity file |
| `/home/ec2-user/multitier-key.pem` | Private key stored temporarily on the Web Tier |
| `ec2-user` | Default Amazon Linux username |
| `@` | Separates username from destination |
| `APP_PRIVATE_IP` | Private IPv4 address of the Application Tier |

### Expected Output

~~~text
[ec2-user@ip-10-0-2-xxx ~]$
~~~

### What should I observe?

The shell prompt should now identify a host with an IP in the:

`10.0.2.x`

range.

Run:

~~~bash
hostname -I
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `hostname` | Displays or manages the system hostname |
| `-I` | Displays all configured IP addresses for the host |

### Expected Output

~~~text
10.0.2.xxx
~~~

### Why does this confirm success?

This proves that:

- The Application Tier is reachable from the Web Tier.
- Private VPC routing works.
- Port `22` is permitted from the Web Tier security group.
- The Application Tier is accessible without a public IPv4 address.

---

# Verification Step 14: Verify the RDS MySQL Instance

### Action

Open the Amazon RDS console and inspect the database.

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Select xyz-multitier-mysql
~~~

### Expected Configuration

| Setting | Expected Value |
| --- | --- |
| DB identifier | `xyz-multitier-mysql` |
| Engine | `MySQL` |
| Status | `Available` |
| DB instance class | `db.t3.micro` |
| Publicly accessible | `No` |
| Port | `3306` |
| VPC | `MultiTier-VPC` |
| Security group | `DB-Tier-SG` |

### What should I observe?

The database must show:

`Available`

and:

`Publicly accessible: No`

### Why does this confirm success?

This verifies that the database exists, is operational, and is private.

---

# Verification Step 15: Verify the Database Security Group

### Action

Inspect the inbound rules of `DB-Tier-SG`.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Security Groups
→ Select DB-Tier-SG
→ Inbound rules
~~~

### Expected Inbound Rule

| Protocol | Port | Source |
| --- | --- | --- |
| TCP | `3306` | `App-Tier-SG` security group ID |

There should not be a MySQL rule allowing:

`0.0.0.0/0`

### What should I observe?

The only inbound MySQL rule should reference the Application Tier security group.

### Why does this confirm success?

This directly verifies the third case-study requirement: the RDS MySQL database accepts port `3306` only from the Application Tier.

---

# Verification Step 16: Test Application-Tier-to-RDS Port 3306 Connectivity

Remain connected to the Application Tier.

Get the RDS endpoint from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
→ RDSEndpoint
~~~

Because the Application Tier private subnet intentionally has no NAT Gateway, avoid relying on installing additional internet packages for this test.

Amazon Linux includes Bash, which can use `/dev/tcp` for a basic TCP connectivity test.

Run:

~~~bash
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/RDS_ENDPOINT/3306' && echo "SUCCESS: RDS port 3306 is reachable from the Application Tier" || echo "FAILED: RDS port 3306 is not reachable"
~~~

Replace `RDS_ENDPOINT` with the actual RDS endpoint.

Example:

~~~bash
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/xyz-multitier-mysql.xxxxxxxxxxxx.ap-south-1.rds.amazonaws.com/3306' && echo "SUCCESS: RDS port 3306 is reachable from the Application Tier" || echo "FAILED: RDS port 3306 is not reachable"
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `timeout` | Limits how long a command may run |
| `5` | Maximum execution time in seconds |
| `bash` | Starts a Bash shell |
| `-c` | Executes the following quoted command string |
| `cat` | Reads data; here it is used as part of the TCP connection test |
| `< /dev/null` | Supplies empty input to the command |
| `>` | Redirects output |
| `/dev/tcp/` | Bash special TCP pseudo-device used to open a TCP connection |
| `RDS_ENDPOINT` | DNS endpoint of the RDS DB instance |
| `3306` | Default MySQL TCP port |
| `&&` | Runs the next command only if the preceding command succeeds |
| `echo` | Displays text |
| `"SUCCESS: ..."` | Success message |
| `\|\|` | Runs the next command if the preceding command chain fails |
| `"FAILED: ..."` | Failure message |

### Expected Output

~~~text
SUCCESS: RDS port 3306 is reachable from the Application Tier
~~~

### What should I observe?

The connection should succeed from the Application Tier.

### Why does this confirm success?

The result proves that:

- The Application Tier can resolve the RDS endpoint.
- VPC DNS works.
- Private VPC routing works.
- RDS is listening on port `3306`.
- `DB-Tier-SG` permits traffic from `App-Tier-SG`.

---

# Verification Step 17: Verify Route 53 Hosted Zone

### Action

Open the Route 53 console.

### Navigation

~~~text
AWS Management Console
→ Route 53
→ Hosted zones
→ Select your domain
~~~

### Expected Configuration

The hosted zone should contain:

- `NS` record.
- `SOA` record.
- The custom `A` record created by CloudFormation.

Example:

~~~text
app.yourdomain.com
Type: A
Value: Web Tier public IPv4 address
TTL: 300
~~~

### What should I observe?

The A record value should match the Web Tier public IPv4 address.

### Why does this confirm success?

This proves that CloudFormation created the Route 53 hosted zone and configured a DNS record targeting the Web Tier.

---

# Verification Step 18: Verify External Domain Delegation

If the domain is registered outside the newly created Route 53 hosted zone, verify that the registrar uses the exact name servers shown in the Route 53 `NS` record.

### Navigation

~~~text
AWS Management Console
→ Route 53
→ Hosted zones
→ Select your hosted zone
→ Records
→ Inspect NS record
~~~

Example nameservers resemble:

~~~text
ns-123.awsdns-45.com
ns-678.awsdns-90.net
ns-111.awsdns-22.org
ns-333.awsdns-44.co.uk
~~~

### Action

At your domain registrar, configure the authoritative name servers to match the Route 53 hosted-zone `NS` records.

### What should I observe?

The registrar and Route 53 name-server values must match.

### Why does this confirm success?

Creating a Route 53 hosted zone alone does not automatically delegate an externally registered domain to that hosted zone.

Without correct delegation, public DNS resolvers will not know that Route 53 is authoritative for the domain.

---

# Verification Step 19: Test Route 53 DNS Resolution

From your local computer, run:

~~~bash
nslookup app.yourdomain.com
~~~

Replace `app.yourdomain.com` with your actual `RecordName`.

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `nslookup` | Queries DNS to resolve domain names |
| `app.yourdomain.com` | Fully qualified domain name being queried |

### Expected Output

A successful response should contain the Web Tier public IPv4 address.

Example:

~~~text
Name:    app.yourdomain.com
Address: 13.XXX.XXX.XXX
~~~

### What should I observe?

The returned IPv4 address should match:

`WebInstancePublicIP`

from the CloudFormation stack outputs.

### Why does this confirm success?

This proves that the Route 53 DNS record resolves publicly to the Web Tier.

---

# Verification Step 20: Test the Application Through Route 53

### Action

Copy `ApplicationURL` from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Outputs
→ ApplicationURL
~~~

Open it in a browser.

Example:

~~~text
http://app.yourdomain.com
~~~

### Expected Output

~~~text
XYZ Corporation Multi-Tier Architecture

Web Tier deployed successfully using AWS CloudFormation.

This page is served from the public Web Tier EC2 instance.
~~~

### What should I observe?

The same application page previously accessed through the Web Tier public IP should now load through the Route 53 DNS name.

### Why does this confirm success?

This verifies the complete path:

~~~text
Browser
   |
   v
Public DNS Query
   |
   v
Route 53 Hosted Zone
   |
   v
A Record
   |
   v
Web Tier Public IPv4 Address
   |
   v
Apache HTTP Server
   |
   v
Application Page
~~~

---

# 19. Verification Against Every Case-Study Requirement

| Case-Study Requirement | Verification Result |
| --- | --- |
| Web EC2 instance launched in public subnet | Verified through EC2 and VPC configuration |
| Web Tier allows HTTP from internet | Verified through security group and browser |
| Web Tier allows SSH from internet | Verified through security group and SSH connection |
| Application EC2 launched in private subnet | Verified through EC2 configuration |
| Application Tier has no public IPv4 | Verified through EC2 details |
| Application Tier allows SSH only from Web Tier | Verified through security-group reference |
| Web-to-App SSH works | Verified through private SSH connection |
| RDS MySQL launched privately | Verified through RDS configuration |
| RDS allows `3306` only from Application Tier | Verified through security-group reference |
| App-to-RDS connectivity works | Verified using TCP connectivity test |
| Route 53 hosted zone created | Verified in Route 53 |
| Route 53 directs traffic to Web EC2 | Verified using DNS resolution and browser |
| Developers can self-provision infrastructure | Implemented through reusable CloudFormation template |
| RDS survives stack deletion | Verified in the following deletion test |

---

# 20. RDS Retention Test Before Final Cleanup

The case study explicitly requires:

> When the development team deletes the stack, RDS DB instances should not be deleted.

Therefore, the stack must be deleted before manually deleting the RDS database.

The expected behavior is:

~~~text
Delete CloudFormation Stack
        |
        +-----------------------------+
        |                             |
        v                             v
Temporary Stack Resources       MySQLDatabase
Deleted                         DeletionPolicy: Retain
                                      |
                                      v
                              RDS Remains Available
~~~

---

# Cleanup Step 1: Remove the Temporarily Copied Private Key from the Web Tier

Before deleting the stack, remove the private key that was temporarily copied to the Web Tier.

If you are still connected to the Application Tier, exit to the Web Tier:

~~~bash
exit
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `exit` | Terminates the Application Tier SSH session and returns to the Web Tier shell |

On the Web Tier, remove the copied key:

~~~bash
rm -f /home/ec2-user/multitier-key.pem
~~~

### Command Breakdown

| Command Part | Explanation |
| --- | --- |
| `rm` | Removes a file |
| `-f` | Forces removal without an interactive confirmation prompt |
| `/home/ec2-user/multitier-key.pem` | Private key file temporarily copied to the Web Tier |

### Why is this required?

Private SSH keys should not remain unnecessarily on remote servers.

### Dependency

Perform this before terminating the Web Tier EC2 instance.

### What happens if this resource is not deleted?

The key would remain on the Web Tier until the instance is terminated. Although termination eventually removes the instance's root volume under the default template behavior, removing sensitive key material immediately after testing is safer.

---

# Cleanup Step 2: Record the RDS Database Details Before Stack Deletion

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Select xyz-multitier-mysql
~~~

Record:

| Property | Value to Record |
| --- | --- |
| DB identifier | `xyz-multitier-mysql` |
| Status | `Available` |
| Endpoint | Your generated endpoint |
| VPC | `MultiTier-VPC` |
| Security group | `DB-Tier-SG` |
| DB subnet group | CloudFormation-created DB subnet group |

### Why is this required?

After stack deletion, the RDS instance should remain because of:

~~~yaml
DeletionPolicy: Retain
UpdateReplacePolicy: Retain
~~~

Recording its identity makes post-deletion verification straightforward.

### Dependency

Must be completed before deleting the CloudFormation stack.

### What happens if this resource is not recorded?

The RDS instance should still remain, but comparing its before-and-after state becomes less convenient.

---

# Cleanup Step 3: Delete the CloudFormation Stack

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select MultiTierArchitectureStack
→ Delete
→ Confirm deletion
~~~

### Action

Select:

`MultiTierArchitectureStack`

Then select:

`Delete`

Confirm the deletion.

### Expected Stack Status

Initially:

~~~text
DELETE_IN_PROGRESS
~~~

Eventually, the stack should disappear from the active stack list.

### Why is this required?

Deleting the stack tests the exact case-study requirement that the RDS DB instance must survive stack deletion.

It also removes temporary resources that are no longer needed.

### Dependency

Perform only after:

- All architecture verification is complete.
- Route 53 verification is complete.
- RDS details have been recorded.

### What happens if this resource is not deleted?

The following resources may continue existing:

- Two EC2 instances.
- EBS volumes.
- Public IPv4 address.
- Route 53 hosted zone.
- RDS instance.

These resources may continue consuming promotional credits or generating charges.

---

# Verification Step 21: Verify the RDS Database Was Retained

After stack deletion completes, open the RDS console.

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Find xyz-multitier-mysql
~~~

### Expected Result

The RDS database should still exist.

Its status may be:

~~~text
Available
~~~

### What should I observe?

The DB identifier should still be:

`xyz-multitier-mysql`

even though:

`MultiTierArchitectureStack`

has been deleted.

### Why does this confirm success?

This directly proves that:

~~~yaml
DeletionPolicy: Retain
~~~

worked as intended.

The RDS DB instance survived CloudFormation stack deletion, satisfying the final case-study requirement.

---

# 21. Important Retained-Resource Dependency Behavior

After the stack is deleted, the retained RDS DB instance still depends on networking resources such as:

- VPC.
- DB subnet group.
- DB subnets.
- Database security group.

CloudFormation may be unable to delete those dependent network resources while the retained RDS instance still uses them.

Therefore, one of two outcomes may occur:

1. The stack deletion succeeds while some supporting resources remain outside normal stack management.

2. Stack deletion reaches `DELETE_FAILED` for resources that are still dependencies of the retained RDS instance.

If `DELETE_FAILED` occurs, this does not mean `DeletionPolicy: Retain` failed. It can mean the retained database is correctly preventing deletion of resources it still requires.

The correct final cleanup order is:

~~~text
Retained RDS DB Instance
        |
        v
Delete RDS Manually
        |
        v
Wait Until RDS Is Fully Deleted
        |
        v
Delete Remaining DB Subnet Group
        |
        v
Delete Remaining Security Groups
        |
        v
Delete Remaining Subnets
        |
        v
Delete Remaining Route Tables
        |
        v
Detach and Delete Internet Gateway
        |
        v
Delete VPC
~~~

---

# Cleanup Step 4: Manually Delete the Retained RDS Database

Only perform this step after Verification Step 21 has confirmed that the database survived stack deletion.

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
→ Select xyz-multitier-mysql
→ Actions
→ Delete
~~~

### Action

Because this is a temporary assignment environment, choose the deletion options appropriate for your lab.

If you do not need the database data:

- Do not create a final snapshot.
- Confirm permanent deletion as requested by the console.

If the console asks whether automated backups should be retained, disable retention unless you intentionally need those backups.

### Why is this required?

The retained RDS DB instance can continue generating charges even after the CloudFormation stack is deleted.

### Dependency

This must happen only after verifying that the RDS instance survived stack deletion.

### What happens if this resource is not deleted?

The RDS DB instance and associated storage may continue:

- Existing in the AWS account.
- Consuming promotional credits.
- Generating charges.

---

# Cleanup Step 5: Wait Until the RDS Database Is Fully Deleted

### Navigation

~~~text
AWS Management Console
→ RDS
→ Databases
~~~

### Action

Wait until:

`xyz-multitier-mysql`

no longer appears in the database list.

Do not attempt to delete its dependent network resources while the DB instance is still being deleted.

### Why is this required?

RDS deletion is asynchronous.

The database may remain in:

~~~text
Deleting
~~~

for several minutes.

### Dependency

Depends on:

- Cleanup Step 4.

### What happens if this step is skipped?

AWS may reject deletion of:

- DB subnet group.
- Subnets.
- Security groups.
- VPC.

because they may still be in use.

---

# Cleanup Step 6: Check the CloudFormation Stack Deletion Status

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
~~~

Enable viewing of deleted or failed stacks if necessary.

### Action

Check whether `MultiTierArchitectureStack` was:

- Successfully deleted, or
- Left in `DELETE_FAILED`.

If the stack shows:

~~~text
DELETE_FAILED
~~~

open:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ MultiTierArchitectureStack
→ Events
~~~

Identify the resources that could not be deleted.

After the RDS DB instance is fully deleted, retry stack deletion.

### Why is this required?

The retained database may temporarily prevent deletion of supporting resources.

### Dependency

Perform after the RDS database is fully deleted.

### What happens if this resource is not checked?

Orphaned resources may remain and potentially continue generating charges.

---

# Cleanup Step 7: Delete Any Remaining Route 53 Hosted Zone

If the CloudFormation stack successfully deleted the hosted zone, no manual action is required.

If it remains, delete it manually.

### Navigation

~~~text
AWS Management Console
→ Route 53
→ Hosted zones
→ Select the hosted zone created for this assignment
~~~

### Action

Delete non-default records created for the assignment if necessary.

Then delete the hosted zone.

Do not manually delete the default `NS` and `SOA` records unless the Route 53 deletion workflow handles them as part of hosted-zone deletion.

### Why is this required?

A Route 53 public hosted zone can continue generating recurring charges while it exists.

### Dependency

The custom DNS record may need to be removed before hosted-zone deletion.

### What happens if this resource is not deleted?

The hosted zone may continue generating charges.

---

# Cleanup Step 8: Delete Any Remaining DB Subnet Group

If CloudFormation did not delete the DB subnet group, remove it after the RDS DB instance has been fully deleted.

### Navigation

~~~text
AWS Management Console
→ RDS
→ Subnet groups
→ Select MultiTier-DB-Subnet-Group
→ Delete
~~~

The physical DB subnet group name may differ because CloudFormation can generate a unique physical name.

### Why is this required?

The DB subnet group may remain as an orphaned resource after the retained RDS database is manually deleted.

### Dependency

Delete only after the RDS DB instance is completely gone.

### What happens if this resource is not deleted?

It may not directly generate significant charges, but it leaves unnecessary infrastructure and can complicate VPC cleanup.

---

# Cleanup Step 9: Verify Both EC2 Instances Are Terminated

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
~~~

### Expected Final State

The following instances should no longer be running:

- `Web-Tier-EC2`
- `App-Tier-EC2`

They may temporarily appear as:

`Terminated`

before disappearing from the console view.

### Why is this required?

Running EC2 instances consume compute resources and may generate charges.

### Dependency

Normally handled automatically by CloudFormation stack deletion.

### What happens if these resources are not deleted?

They may continue consuming AWS credits or generating compute charges.

---

# Cleanup Step 10: Verify No Unnecessary EBS Volumes Remain

### Navigation

~~~text
AWS Management Console
→ EC2
→ Elastic Block Store
→ Volumes
~~~

### Action

Look for unattached volumes associated with the deleted Web and Application Tier EC2 instances.

The template's automatically created root volumes are normally deleted with their EC2 instances, but verification is still important.

### Why is this required?

Unattached EBS volumes can continue generating storage charges.

### Dependency

Perform after EC2 termination.

### What happens if these resources are not deleted?

Storage charges can continue even when no EC2 instance is running.

---

# Cleanup Step 11: Delete Any Remaining Security Groups

Only perform this step if CloudFormation did not remove them.

Delete in this dependency order:

~~~text
DB-Tier-SG
    |
    v
App-Tier-SG
    |
    v
Web-Tier-SG
~~~

### Navigation

~~~text
AWS Management Console
→ EC2
→ Security Groups
~~~

### Why is this required?

Security groups that reference other security groups create dependencies.

For example:

- `DB-Tier-SG` references `App-Tier-SG`.
- `App-Tier-SG` references `Web-Tier-SG`.

Deleting them in reverse dependency order avoids dependency violations.

### Dependency

The RDS database and EC2 instances must already be deleted.

### What happens if these resources are not deleted?

They may prevent VPC deletion and leave unnecessary configuration behind.

---

# Cleanup Step 12: Delete Any Remaining Subnets

Only perform this step if CloudFormation did not delete them.

Delete:

- `Web-Public-Subnet`
- `App-Private-Subnet`
- `DB-Private-Subnet-A`
- `DB-Private-Subnet-B`

### Navigation

~~~text
AWS Management Console
→ VPC
→ Subnets
→ Select remaining assignment subnets
→ Actions
→ Delete subnet
~~~

### Why is this required?

A VPC cannot be deleted while dependent subnets remain.

### Dependency

Delete only after:

- EC2 instances are terminated.
- RDS DB instance is deleted.
- DB subnet group is deleted.
- Other subnet-dependent resources are removed.

### What happens if these resources are not deleted?

The custom VPC cannot be fully deleted.

---

# Cleanup Step 13: Delete Any Remaining Custom Route Tables

### Navigation

~~~text
AWS Management Console
→ VPC
→ Route tables
~~~

Delete any remaining assignment-created custom route tables after removing their explicit subnet associations.

Do not attempt to manually delete the VPC's main route table directly.

### Why is this required?

Custom route tables may remain as dependencies of the VPC.

### Dependency

Subnets and explicit associations should be removed first.

### What happens if these resources are not deleted?

They can prevent complete network cleanup.

---

# Cleanup Step 14: Detach and Delete Any Remaining Internet Gateway

### Navigation

~~~text
AWS Management Console
→ VPC
→ Internet gateways
→ Select MultiTier-IGW
→ Actions
→ Detach from a VPC
~~~

After detachment:

~~~text
AWS Management Console
→ VPC
→ Internet gateways
→ Select MultiTier-IGW
→ Actions
→ Delete internet gateway
~~~

### Why is this required?

An attached Internet Gateway can prevent VPC deletion.

### Dependency

The public resources that use internet connectivity should already be deleted.

### What happens if this resource is not deleted?

The VPC may remain undeletable because the Internet Gateway is still attached.

---

# Cleanup Step 15: Delete the Custom VPC If It Still Exists

### Navigation

~~~text
AWS Management Console
→ VPC
→ Your VPCs
→ Select MultiTier-VPC
→ Actions
→ Delete VPC
~~~

### Why is this required?

The VPC is the top-level network container for the architecture.

It should be deleted after all dependent resources are removed.

### Dependency

Delete last among the VPC networking resources.

### What happens if this resource is not deleted?

The unused VPC remains in the AWS account and contributes to infrastructure clutter.

---

# 22. Final Resource Verification Table

After completing cleanup, verify the following final states:

| Resource | Expected Final State |
| --- | --- |
| CloudFormation stack | `Deleted` |
| Web Tier EC2 instance | `Terminated / Deleted` |
| Application Tier EC2 instance | `Terminated / Deleted` |
| Web Tier EBS root volume | `Deleted` |
| Application Tier EBS root volume | `Deleted` |
| RDS MySQL instance | `Deleted after retention verification` |
| RDS automated backups | `Deleted unless intentionally retained` |
| RDS final snapshot | `Not created unless intentionally required` |
| DB subnet group | `Deleted` |
| Route 53 A record | `Deleted` |
| Route 53 public hosted zone | `Deleted` |
| Web Tier security group | `Deleted` |
| Application Tier security group | `Deleted` |
| Database Tier security group | `Deleted` |
| Public subnet | `Deleted` |
| Application private subnet | `Deleted` |
| Database private subnet A | `Deleted` |
| Database private subnet B | `Deleted` |
| Public route table | `Deleted` |
| Private route table | `Deleted` |
| Internet Gateway | `Detached and Deleted` |
| Custom VPC | `Deleted` |
| EC2 key pair | `Optional: Delete if no longer required` |

---

# 23. Final Case-Study Result

The completed implementation provides the following architecture:

~~~text
Internet
    |
    | HTTP 80
    | SSH 22
    v
+---------------------------+
| Web Tier EC2              |
| Public Subnet             |
| Public IPv4 Address       |
+-------------+-------------+
              |
              | SSH 22
              | Source: Web-Tier-SG
              v
+---------------------------+
| Application Tier EC2      |
| Private Subnet            |
| No Public IPv4 Address    |
+-------------+-------------+
              |
              | MySQL 3306
              | Source: App-Tier-SG
              v
+---------------------------+
| Amazon RDS for MySQL      |
| Private DB Subnets        |
| Publicly Accessible: No   |
| DeletionPolicy: Retain    |
+---------------------------+

Route 53
    |
    v
Application DNS Record
    |
    v
Web Tier Public IPv4 Address
~~~

The development team can deploy the complete environment using a reusable CloudFormation template without manually requesting system administrators to provision each resource.

When the stack is deleted, the RDS DB instance is retained because the CloudFormation resource contains:

~~~yaml
DeletionPolicy: Retain
UpdateReplacePolicy: Retain
~~~

This satisfies all requirements of the Multi-Tier Architecture case study.

---


