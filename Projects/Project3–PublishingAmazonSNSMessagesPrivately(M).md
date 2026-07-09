# Project 3 – Publishing Amazon SNS Messages Privately

# Part 1: Project Overview, Cost Analysis, Architecture, Prerequisites, and Initial AWS Setup

## Industry

Healthcare

## Problem Statement

How can patient records and medical reports be securely published online and delivered privately to the intended party?

In this project, we will build a secure AWS architecture for a hospital use case in which medical report notifications are published from an Amazon EC2 instance to an Amazon SNS topic without sending the publishing traffic over the public internet.

The EC2 instance will run inside an Amazon VPC. An **Interface VPC Endpoint for Amazon SNS**, powered by **AWS PrivateLink**, will provide private connectivity between the VPC and Amazon SNS.

AWS officially documents that an Amazon SNS interface VPC endpoint allows applications inside a VPC to publish messages to Amazon SNS while keeping the traffic within the AWS network rather than sending it across the public internet.

> **Important healthcare security note:** This project demonstrates private network connectivity and secure message publishing for educational purposes. For an actual production healthcare system, patient records and protected health information should not be placed directly into notification messages without implementing all required encryption, access control, auditing, data-retention, privacy, and regulatory controls.

---

## Project Highlights

1. Use AWS CloudFormation to create the Amazon VPC infrastructure.
2. Launch an Amazon EC2 instance inside the VPC.
3. Create an Amazon SNS topic.
4. Connect the VPC privately to Amazon SNS by using an Interface VPC Endpoint.
5. Use AWS PrivateLink so that SNS publishing traffic does not traverse the public internet.
6. Attach an IAM role to the EC2 instance with permission to publish only to the intended SNS topic.
7. Publish a test hospital report notification privately from the EC2 instance.
8. Verify successful private message publishing.
9. Delete all created resources in reverse dependency order to minimize AWS charges.

---

## Important Architecture Decision

The phrase **"Connect VPC with AWS SNS"** is implemented by creating an **Interface VPC Endpoint for Amazon SNS**.

The regional SNS endpoint service name follows this pattern:

~~~text
com.amazonaws.<region>.sns
~~~

For example, in the Mumbai Region:

~~~text
com.amazonaws.ap-south-1.sns
~~~

The interface endpoint is powered by AWS PrivateLink.

This means the publishing path becomes:

~~~text
EC2 Instance
      |
      | HTTPS on TCP port 443
      v
SNS Interface VPC Endpoint
      |
      | AWS PrivateLink
      v
Amazon SNS Topic
~~~

The SNS publishing traffic stays on the AWS network and does not need to traverse the public internet.

---

## Free Tier and Cost Check

AWS Free Tier eligibility depends on factors such as:

- AWS account creation date.
- Whether the account uses the newer AWS Free Tier credit-based model.
- Whether promotional credits are available.
- Region.
- Resource type.
- Usage duration.
- Whether Free Tier limits have already been consumed.

The most important cost consideration in this project is the **Interface VPC Endpoint**, because AWS PrivateLink interface endpoints generally incur hourly and data-processing charges.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| AWS CloudFormation | Generally no additional charge for creating standard AWS resources | No direct CloudFormation charge for this project | You pay for the AWS resources provisioned by the stack. |
| Amazon VPC | Core VPC components are generally available without a direct hourly VPC charge | Usually no direct credit usage for the VPC itself | Some VPC features, including interface endpoints, public IPv4 addresses, and NAT gateways, can incur charges. |
| Public Subnet | No separate hourly charge | No direct charge | Resources placed inside the subnet can still incur charges. |
| Internet Gateway | No hourly charge by itself | No direct charge | Data transfer and other associated resources may still incur charges. |
| Route Table | No separate hourly charge | No direct charge | Standard VPC routing component. |
| Security Groups | No separate hourly charge | No direct charge | Used as virtual firewalls. |
| Amazon EC2 | Depends on account eligibility and selected instance type | May consume AWS credits | The instance can incur compute charges if it is not covered by Free Tier or promotional credits. |
| Amazon EBS | Depends on account eligibility and usage | May consume AWS credits | The EC2 root volume can incur storage charges. |
| Public IPv4 Address | Can incur charges depending on account benefits and eligibility | May consume credits | We will minimize usage duration and delete resources immediately after verification. |
| Amazon SNS | Has limited free usage allowances depending on the operation and delivery type | May consume credits after applicable allowance | A few test API requests and messages normally produce very small usage. |
| Interface VPC Endpoint for SNS | **Not generally Free Tier** | **Yes, where applicable** | This is the primary chargeable networking resource in the project. Interface endpoints are billed based on endpoint usage and data processing according to AWS PrivateLink pricing. |
| IAM | No additional charge | No | IAM role and policy creation do not have separate charges. |

### Is this project completely Free Tier eligible?

**No.**

The complete architecture should not be considered completely Free Tier eligible because the **Amazon SNS Interface VPC Endpoint powered by AWS PrivateLink is a chargeable resource**.

The EC2 instance and EBS storage may also incur charges depending on:

- Your AWS account age.
- Your AWS Free Tier plan.
- Your remaining AWS promotional credits.
- The selected instance type.
- Your previous monthly usage.

### Which resource is most likely to incur charges?

The primary resource that should be watched carefully is:

~~~text
Amazon SNS Interface VPC Endpoint
        |
        v
Powered by AWS PrivateLink
        |
        v
Hourly endpoint charges + data-processing charges may apply
~~~

### Should this project be completed while AWS promotional credits are available?

Yes. If promotional credits are available and applicable to these services, completing the project while those credits remain available can help offset eligible charges.

However, AWS credits should never be assumed to cover every service or every charge. Billing eligibility depends on the specific credit terms.

### Resources to delete immediately after verification

Delete the following resources after successful verification:

1. SNS subscription, if one is created.
2. SNS topic.
3. SNS Interface VPC Endpoint.
4. EC2 instance.
5. CloudFormation stack and all infrastructure resources created by it.
6. IAM instance profile and IAM role if they are not managed by the CloudFormation stack.
7. Any additional test resources created during troubleshooting.

The **Interface VPC Endpoint** should be deleted promptly because it can continue generating hourly charges while provisioned.

---

## Architecture

~~~text
                           AWS Cloud
                               |
                 +-------------+-------------+
                 |                           |
                 |       Amazon SNS          |
                 |                           |
                 |   +-------------------+   |
                 |   | Hospital Reports  |   |
                 |   |    SNS Topic      |   |
                 |   +---------^---------+   |
                 |             |             |
                 +-------------|-------------+
                               |
                       AWS PrivateLink
                               |
                               |
+---------------------------------------------------------------+
|                        Custom Amazon VPC                      |
|                                                               |
|   +-------------------------------------------------------+   |
|   |                    Public Subnet                      |   |
|   |                                                       |   |
|   |   +----------------------+                            |   |
|   |   |    EC2 Instance      |                            |   |
|   |   |                      |                            |   |
|   |   | AWS CLI              |                            |   |
|   |   | IAM Role             |                            |   |
|   |   | Hospital App/Test    |                            |   |
|   |   +----------+-----------+                            |   |
|   |              |                                        |   |
|   |              | HTTPS TCP 443                          |   |
|   |              v                                        |   |
|   |   +---------------------------+                       |   |
|   |   | SNS Interface VPC Endpoint|                       |   |
|   |   |   Private IP Address      |                       |   |
|   |   +-------------+-------------+                       |   |
|   |                 |                                     |   |
|   +-----------------|-------------------------------------+   |
|                     |                                         |
+---------------------|-----------------------------------------+
                      |
                      v
              Amazon SNS Service
                      |
                      v
          Securely Publish Test Message
~~~

---

## How the Private Message Publishing Works

~~~text
1. The EC2 instance runs inside the custom VPC.

2. The EC2 instance receives AWS credentials automatically through
   its attached IAM role.

3. The EC2 instance executes the AWS CLI command:

   aws sns publish

4. DNS resolves the regional Amazon SNS service endpoint through
   the enabled private DNS configuration.

5. The request reaches the SNS Interface VPC Endpoint through
   HTTPS on TCP port 443.

6. AWS PrivateLink carries the request privately to Amazon SNS.

7. Amazon SNS receives and publishes the message to the intended topic.

8. The SNS API returns a MessageId when publishing succeeds.
~~~

---

## Dependency Flow

The project will be implemented in strict dependency order.

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Create CloudFormation Template
    |
    v
Deploy CloudFormation Stack
    |
    +---------------------------+
    |                           |
    v                           v
Custom VPC                 Public Subnet
    |                           |
    +-------------+-------------+
                  |
                  v
          Internet Gateway
                  |
                  v
             Route Table
                  |
                  v
           Security Groups
                  |
                  v
             IAM Role
                  |
                  v
              SNS Topic
                  |
                  v
       SNS Interface VPC Endpoint
                  |
                  v
             EC2 Instance
                  |
                  v
         Connect to EC2 Instance
                  |
                  v
       Verify AWS CLI Credentials
                  |
                  v
     Publish Message to Amazon SNS
                  |
                  v
       Verify Returned MessageId
                  |
                  v
      Verify Private Connectivity
                  |
                  v
              Cleanup
~~~

---

## Resources That Will Be Created

To make the project reproducible and aligned with the requirement to use AWS CloudFormation, the infrastructure will be created primarily through a CloudFormation template.

| Resource | Planned Name | Purpose |
| -------- | ------------ | ------- |
| CloudFormation Stack | `hospital-private-sns-stack` | Manages project infrastructure as code. |
| VPC | `Hospital-VPC` | Provides an isolated virtual network. |
| Public Subnet | `Hospital-Public-Subnet` | Hosts the EC2 instance and SNS interface endpoint for this lab architecture. |
| Internet Gateway | `Hospital-IGW` | Provides internet connectivity required for direct administrative access to the lab EC2 instance. |
| Route Table | `Hospital-Public-RT` | Routes public-subnet internet traffic through the Internet Gateway. |
| EC2 Security Group | `Hospital-EC2-SG` | Controls inbound and outbound EC2 traffic. |
| VPC Endpoint Security Group | `Hospital-SNS-Endpoint-SG` | Permits HTTPS traffic from the EC2 security group to the SNS interface endpoint. |
| IAM Role | `Hospital-EC2-SNS-Role` | Grants the EC2 instance permission to publish to the intended SNS topic. |
| IAM Instance Profile | Created through CloudFormation | Makes the IAM role attachable to the EC2 instance. |
| SNS Topic | `hospital-private-reports` | Receives privately published test hospital report notifications. |
| SNS Interface VPC Endpoint | Regional SNS service endpoint | Provides private connectivity from the VPC to Amazon SNS through AWS PrivateLink. |
| EC2 Instance | `Hospital-SNS-Publisher` | Hosts the AWS CLI and publishes the test SNS message. |

---

## Security Design

This project will use several security controls rather than granting unrestricted access.

### Security Control 1: IAM Role Instead of Hard-Coded Access Keys

The EC2 instance will use an IAM role.

We will **not** manually configure permanent AWS access keys on the EC2 instance.

~~~text
Incorrect approach:

EC2
    |
    v
Hard-coded Access Key + Secret Access Key


Preferred approach:

EC2
    |
    v
IAM Instance Profile
    |
    v
Temporary AWS Credentials
    |
    v
Permission to Publish to Intended SNS Topic
~~~

### Security Control 2: Least-Privilege SNS Permission

The EC2 IAM role will be granted permission to perform:

~~~text
sns:Publish
~~~

against the SNS topic created for this project.

The goal is to avoid granting broad administrative permissions such as:

~~~text
sns:*
~~~

or:

~~~text
Action: "*"
Resource: "*"
~~~

unless technically required for a specific operation.

### Security Control 3: HTTPS Only

Amazon SNS interface VPC endpoint communication uses HTTPS.

The endpoint security group will permit:

| Type | Protocol | Port | Source |
| ---- | -------- | ---- | ------ |
| HTTPS | TCP | `443` | EC2 Security Group |

Using a security-group reference is safer than allowing TCP port `443` from every IPv4 address.

We will **not** use this endpoint inbound rule:

~~~text
HTTPS
TCP
443
0.0.0.0/0
~~~

Instead, the architecture will use:

~~~text
Source:
Hospital-EC2-SG
~~~

This means only resources associated with the authorized EC2 security group can initiate the permitted HTTPS connection to the SNS interface endpoint.

### Security Control 4: AWS PrivateLink

The EC2 instance communicates with Amazon SNS through an interface VPC endpoint.

~~~text
EC2
 |
 | HTTPS 443
 v
SNS Interface Endpoint
 |
 | AWS PrivateLink
 v
Amazon SNS
~~~

The SNS publishing traffic does not need to traverse the public internet.

### Security Control 5: No Real Patient Data

For this educational project, use only synthetic test information.

Example:

~~~text
Patient ID: TEST-PATIENT-1001
Report Status: Available
Message: Your test medical report is ready for secure access.
~~~

Do not use:

- Real patient names.
- Real medical diagnoses.
- Real medical report content.
- Real phone numbers unless intentionally testing a subscription you control.
- Real patient identifiers.
- Real protected health information.

---

## Region Selection

For this project, use the **Asia Pacific (Mumbai)** Region:

| Setting | Value |
| ------- | ----- |
| Region Name | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |
| SNS Service Name | `com.amazonaws.ap-south-1.sns` |

All resources should be created in the same AWS Region unless explicitly stated otherwise.

### Why use the same Region?

The following resources are regional:

- VPC.
- Subnet.
- EC2 instance.
- SNS topic.
- Interface VPC endpoint.

Using the same Region simplifies the architecture and ensures that the EC2 instance can use the regional SNS interface endpoint.

---

## Prerequisites

Before starting the implementation, ensure that the following are available:

| Prerequisite | Requirement |
| ------------ | ----------- |
| AWS Account | Active AWS account with permission to create the required resources. |
| AWS Region | `Asia Pacific (Mumbai) / ap-south-1` |
| IAM Permissions | Permission to use CloudFormation, EC2, VPC, SNS, IAM, and related resources. |
| Browser | Modern browser for AWS Management Console access. |
| SSH Client | Optional, depending on the connection method used later. |
| AWS Credits | Recommended because the SNS interface VPC endpoint is chargeable. |
| Test Data | Synthetic data only; do not use real patient information. |

---

## Step 1: Sign In to the AWS Management Console

### Navigation

~~~text
AWS Management Console
→ Sign in
→ Open the AWS Console Home
~~~

### Action

Sign in to the AWS account that you want to use for this project.

Verify that you are using the correct AWS account before creating resources.

### Why is this step required?

Every AWS resource in the project must belong to an AWS account. The account provides:

- Authentication.
- Authorization.
- Billing.
- Resource ownership.
- AWS service access.

### Dependency

This step has no project-resource dependency.

It is the starting point for the entire implementation.

### What happens if this step is skipped?

You cannot create:

- A VPC.
- An EC2 instance.
- An SNS topic.
- An interface VPC endpoint.
- A CloudFormation stack.

Therefore, all later steps would be blocked.

---

## Step 2: Select the Asia Pacific (Mumbai) Region

### Navigation

~~~text
AWS Management Console
→ Top navigation bar
→ Region selector
→ Asia Pacific (Mumbai)
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

### Action

Select:

`Asia Pacific (Mumbai)`

from the AWS Region selector.

### Why is this step required?

Most resources in this project are regional. Selecting the Region before deployment helps ensure that the following resources are created together in `ap-south-1`:

- CloudFormation stack.
- VPC.
- Subnet.
- EC2 instance.
- SNS topic.
- SNS interface endpoint.

### Dependency

Depends on:

- **Step 1:** Successful AWS Management Console sign-in.

### What happens if this step is skipped?

You may accidentally create resources in another Region.

This can cause:

- Resources not appearing where expected.
- Regional SNS endpoint mismatch.
- Troubleshooting confusion.
- Additional resource cleanup requirements.
- Unexpected charges from forgotten resources in another Region.

---

## Step 3: Check AWS Free Tier and Credit Status Before Deployment

### Navigation

~~~text
AWS Management Console
→ Billing and Cost Management
→ Home
~~~

Depending on the current AWS Console interface and account type, Free Tier and credit information may appear under areas such as:

~~~text
Billing and Cost Management
→ Free Tier
~~~

and:

~~~text
Billing and Cost Management
→ Credits
~~~

The exact labels can vary slightly based on AWS account type and Console updates.

### Action

Before creating resources, inspect:

- Current AWS Free Tier eligibility.
- Promotional credit balance.
- Credit expiration date.
- Current month-to-date charges.
- Existing EC2 usage.
- Existing public IPv4 usage.
- Any existing interface VPC endpoints.

### Why is this step required?

This project includes resources that can incur charges, especially the SNS Interface VPC Endpoint.

Checking billing status before deployment helps prevent unexpected costs.

### Dependency

Depends on:

- **Step 1:** AWS account sign-in.

It does not depend on any project resource.

### What happens if this step is skipped?

The project can still technically be deployed, but you may not know:

- Whether EC2 usage is covered.
- Whether EBS usage is covered.
- Whether credits are available.
- Whether previous resources are already consuming Free Tier allowances.
- Whether unexpected charges are accumulating.

---

## Step 4: Understand the CloudFormation Deployment Strategy

The assignment specifically highlights:

> **AWS CloudFormation to create a VPC**

Therefore, this implementation will use AWS CloudFormation to create the project infrastructure reproducibly.

The CloudFormation stack will create the following dependency chain:

~~~text
CloudFormation Stack
        |
        +--> VPC
        |
        +--> Internet Gateway
        |
        +--> Internet Gateway Attachment
        |
        +--> Public Subnet
        |
        +--> Public Route Table
        |
        +--> Public Route
        |
        +--> Subnet Route Table Association
        |
        +--> EC2 Security Group
        |
        +--> SNS Endpoint Security Group
        |
        +--> SNS Topic
        |
        +--> IAM Role
        |
        +--> IAM Instance Profile
        |
        +--> SNS Interface VPC Endpoint
        |
        +--> EC2 Instance
~~~

### Why use CloudFormation?

CloudFormation provides Infrastructure as Code.

Instead of manually creating every AWS resource individually, we define the desired architecture in a YAML template.

AWS CloudFormation then:

1. Reads the template.
2. Determines resource dependencies.
3. Creates the resources.
4. Tracks them as one stack.
5. Makes cleanup easier because stack-managed resources can be deleted together.

### Dependency

Depends on:

- **Step 1:** AWS account access.
- **Step 2:** Correct AWS Region selection.
- **Step 3:** Billing and credit review.

### What happens if this step is skipped?

Without understanding the deployment strategy, it is easy to:

- Create duplicate resources manually.
- Create resources in the wrong order.
- Misunderstand which resources are managed by CloudFormation.
- Accidentally delete or modify resources outside the stack.
- Make cleanup more difficult.

---

## Planned CloudFormation Parameters

The CloudFormation template in the next part will use parameters so that important values can be customized without editing the main resource definitions.

| Parameter | Planned Default Value | Purpose |
| --------- | --------------------- | ------- |
| `VpcCidr` | `10.0.0.0/16` | CIDR block for the custom VPC. |
| `PublicSubnetCidr` | `10.0.1.0/24` | CIDR block for the public subnet. |
| `InstanceType` | `t3.micro` | EC2 instance size for the project. |
| `LatestAmiId` | AWS Systems Manager public parameter | Dynamically resolves a current Amazon Linux AMI. |
| `AllowedSSHCidr` | User-supplied value if SSH is used | Restricts SSH access instead of allowing the entire internet. |

The exact parameter set will be finalized in the CloudFormation template.

---

## Planned Network Configuration

| Component | Configuration |
| --------- | ------------- |
| VPC CIDR | `10.0.0.0/16` |
| Public Subnet CIDR | `10.0.1.0/24` |
| DNS Support | `Enabled` |
| DNS Hostnames | `Enabled` |
| EC2 Security Group | Allows only required administrative access and outbound communication. |
| SNS Endpoint Security Group | Allows inbound HTTPS TCP `443` only from the EC2 security group. |
| SNS Private DNS | `Enabled` |
| SNS Endpoint Type | `Interface` |

---

## Why DNS Support and DNS Hostnames Matter

The VPC will have both of these settings enabled:

| VPC Setting | Value |
| ----------- | ----- |
| DNS Support | `true` |
| DNS Hostnames | `true` |

Private DNS is important because the application can use the standard regional SNS service hostname while DNS directs the request to the private IP addresses of the SNS interface endpoint.

Conceptually:

~~~text
Application command:

aws sns publish
        |
        v
Standard regional SNS hostname
        |
        v
VPC DNS resolution
        |
        v
Private IP of SNS Interface Endpoint
        |
        v
AWS PrivateLink
        |
        v
Amazon SNS
~~~

This avoids requiring the application to hard-code endpoint-specific private IP addresses.

---

## Why the SNS Endpoint Needs a Security Group

An interface VPC endpoint creates one or more endpoint network interfaces inside selected subnets.

These network interfaces have private IP addresses and use security groups.

The SNS endpoint security group must allow the EC2 instance to establish an HTTPS connection.

Required rule:

| Direction | Type | Protocol | Port | Source |
| --------- | ---- | -------- | ---- | ------ |
| Inbound | HTTPS | TCP | `443` | `Hospital-EC2-SG` |

The EC2 security group reference will be used as the source.

This creates the following access relationship:

~~~text
Hospital-EC2-SG
        |
        | HTTPS TCP 443
        v
Hospital-SNS-Endpoint-SG
        |
        v
SNS Interface Endpoint
~~~

---

## Why the EC2 Instance Needs an IAM Role

The EC2 instance must authenticate to AWS before it can publish an SNS message.

We will attach an IAM role through an instance profile.

~~~text
EC2 Instance
      |
      v
IAM Instance Profile
      |
      v
IAM Role
      |
      v
Temporary AWS Credentials
      |
      v
sns:Publish permission
      |
      v
Hospital Reports SNS Topic
~~~

This is preferred over manually running:

~~~bash
aws configure
~~~

and storing long-term IAM access keys on the server.

For this project, permanent IAM user credentials should not be copied to the EC2 instance.

---

## Planned Test Message

We will use synthetic test data such as:

~~~text
Hospital Report Notification

Patient ID: TEST-PATIENT-1001
Report Status: Available
Message: Your test medical report is ready for secure access.
~~~

This data is intentionally fictional.

The test message will be published from:

~~~text
EC2 Instance
    |
    v
SNS Interface VPC Endpoint
    |
    v
AWS PrivateLink
    |
    v
Amazon SNS Topic
~~~

A successful `aws sns publish` operation should return output containing a message identifier similar to:

~~~json
{
    "MessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
~~~

The exact `MessageId` will be different for every successful publish operation.

---

## Implementation Status After Part 1

At this point:

- The project architecture has been defined.
- The cost implications have been identified.
- The AWS Region has been selected.
- The dependency order has been established.
- The required AWS resources have been identified.
- The security model has been defined.
- The private SNS publishing path has been explained.
- No chargeable project resource needs to have been created yet.

The next part will begin the actual infrastructure deployment by creating the complete AWS CloudFormation YAML template and deploying the stack.

---

# Part 2: Create the CloudFormation Template and Deploy the AWS Infrastructure

In Part 1, the complete architecture, cost considerations, dependency order, security design, and prerequisites were established.

This part begins the actual implementation.

We will create one AWS CloudFormation YAML template that provisions the following resources in dependency order:

~~~text
CloudFormation Stack
        |
        v
Custom VPC
        |
        +----------------------------+
        |                            |
        v                            v
Internet Gateway              Public Subnet
        |                            |
        +-------------+--------------+
                      |
                      v
               Public Route Table
                      |
                      v
              Security Groups
                      |
        +-------------+-------------+
        |                           |
        v                           v
     SNS Topic                  IAM Role
        |                           |
        |                           v
        |                  IAM Instance Profile
        |                           |
        v                           |
SNS Interface VPC Endpoint          |
        |                           |
        +-------------+-------------+
                      |
                      v
                 EC2 Instance
~~~

The CloudFormation template will create all required resources in the correct dependency order.

---

## Step 5: Open AWS CloudFormation

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for CloudFormation
→ Select CloudFormation
→ Stacks
~~~

### Action

Open the AWS CloudFormation console in the same Region selected in Part 1.

Verify that the Region shown in the top navigation bar is:

`Asia Pacific (Mumbai)`

The Region code should be:

`ap-south-1`

Do not create the stack yet. We first need to prepare the CloudFormation YAML template.

### Why is this step required?

AWS CloudFormation is the Infrastructure as Code service used in this project.

The CloudFormation service will:

- Read our YAML template.
- Validate the template structure.
- Determine resource dependencies.
- Create the VPC.
- Create the subnet.
- Configure internet connectivity.
- Create security groups.
- Create the SNS topic.
- Create the IAM role.
- Create the SNS interface VPC endpoint.
- Launch the EC2 instance.
- Track all resources as part of one stack.

### Dependency

This step depends on:

- Successful AWS account sign-in.
- Correct AWS Region selection.
- Completion of the billing and cost review from Part 1.

### What happens if this step is skipped?

The infrastructure cannot be deployed using CloudFormation.

You would need to manually create every resource, which would not satisfy the project highlight requiring AWS CloudFormation to create the VPC.

---

## Step 6: Create the CloudFormation YAML Template

Create a new file on your local computer with the following filename:

`hospital-private-sns.yaml`

The complete template is provided below.

> **Important:** The template intentionally does not require an EC2 key pair. AWS Systems Manager Session Manager is used for administrative access to the EC2 instance. This avoids opening inbound SSH port `22` to the internet and avoids requiring a public IP address for SSH access.

~~~yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: >
  Project 3 - Publishing Amazon SNS Messages Privately.
  Creates a custom VPC, public subnet, Internet Gateway,
  security groups, SNS topic, IAM role, SNS interface VPC endpoint,
  and an EC2 instance that can publish messages privately to Amazon SNS.

Parameters:

  VpcCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: CIDR block for the custom hospital VPC.

  PublicSubnetCidr:
    Type: String
    Default: 10.0.1.0/24
    Description: CIDR block for the hospital public subnet.

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t2.micro
      - t3.micro
    Description: EC2 instance type used for the SNS publisher.

  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64
    Description: Latest Amazon Linux 2023 AMI obtained from the AWS Systems Manager public parameter.

Resources:

  HospitalVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Hospital-VPC

  HospitalInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Hospital-IGW

  HospitalInternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref HospitalVPC
      InternetGatewayId: !Ref HospitalInternetGateway

  HospitalPublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref HospitalVPC
      CidrBlock: !Ref PublicSubnetCidr
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select
        - 0
        - !GetAZs ''
      Tags:
        - Key: Name
          Value: Hospital-Public-Subnet

  HospitalPublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref HospitalVPC
      Tags:
        - Key: Name
          Value: Hospital-Public-RT

  HospitalDefaultRoute:
    Type: AWS::EC2::Route
    DependsOn:
      - HospitalInternetGatewayAttachment
    Properties:
      RouteTableId: !Ref HospitalPublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref HospitalInternetGateway

  HospitalPublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref HospitalPublicSubnet
      RouteTableId: !Ref HospitalPublicRouteTable

  HospitalEC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for the hospital SNS publisher EC2 instance.
      VpcId: !Ref HospitalVPC
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
          Description: Allow outbound traffic required for AWS service communication.
      Tags:
        - Key: Name
          Value: Hospital-EC2-SG

  HospitalSNSEndpointSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allows HTTPS from the hospital EC2 publisher to the SNS interface endpoint.
      VpcId: !Ref HospitalVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          SourceSecurityGroupId: !Ref HospitalEC2SecurityGroup
          Description: Allow HTTPS from Hospital EC2 security group.
      Tags:
        - Key: Name
          Value: Hospital-SNS-Endpoint-SG

  HospitalReportsTopic:
    Type: AWS::SNS::Topic
    Properties:
      TopicName: hospital-private-reports
      DisplayName: Hospital Private Reports
      Tags:
        - Key: Name
          Value: Hospital-Private-Reports-Topic

  HospitalEC2SNSRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: Hospital-EC2-SNS-Role
      Description: Allows the hospital EC2 instance to use Session Manager and publish only to the project SNS topic.
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
      Policies:
        - PolicyName: HospitalSNSPublishPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Sid: AllowPublishToHospitalTopic
                Effect: Allow
                Action:
                  - sns:Publish
                Resource: !Ref HospitalReportsTopic
      Tags:
        - Key: Name
          Value: Hospital-EC2-SNS-Role

  HospitalEC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: Hospital-EC2-SNS-Instance-Profile
      Roles:
        - !Ref HospitalEC2SNSRole

  HospitalSNSEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcEndpointType: Interface
      VpcId: !Ref HospitalVPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.sns
      PrivateDnsEnabled: true
      SubnetIds:
        - !Ref HospitalPublicSubnet
      SecurityGroupIds:
        - !Ref HospitalSNSEndpointSecurityGroup
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal: '*'
            Action:
              - sns:Publish
            Resource: !Ref HospitalReportsTopic

  HospitalEC2Instance:
    Type: AWS::EC2::Instance
    DependsOn:
      - HospitalDefaultRoute
      - HospitalPublicSubnetRouteTableAssociation
      - HospitalSNSEndpoint
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceType
      SubnetId: !Ref HospitalPublicSubnet
      SecurityGroupIds:
        - !Ref HospitalEC2SecurityGroup
      IamInstanceProfile:
        Name: !Ref HospitalEC2InstanceProfile
      MetadataOptions:
        HttpEndpoint: enabled
        HttpTokens: required
      Tags:
        - Key: Name
          Value: Hospital-SNS-Publisher

Outputs:

  VpcId:
    Description: ID of the custom hospital VPC.
    Value: !Ref HospitalVPC

  PublicSubnetId:
    Description: ID of the hospital public subnet.
    Value: !Ref HospitalPublicSubnet

  EC2InstanceId:
    Description: ID of the hospital SNS publisher EC2 instance.
    Value: !Ref HospitalEC2Instance

  SNSTopicArn:
    Description: ARN of the hospital private reports SNS topic.
    Value: !Ref HospitalReportsTopic

  SNSEndpointId:
    Description: ID of the SNS interface VPC endpoint.
    Value: !Ref HospitalSNSEndpoint

  SNSEndpointService:
    Description: Regional Amazon SNS VPC endpoint service name.
    Value: !Sub com.amazonaws.${AWS::Region}.sns
~~~

---

## CloudFormation Template Explanation

The template contains four major sections:

~~~text
AWSTemplateFormatVersion
        |
        v
Description
        |
        v
Parameters
        |
        v
Resources
        |
        v
Outputs
~~~

### `AWSTemplateFormatVersion`

~~~yaml
AWSTemplateFormatVersion: '2010-09-09'
~~~

This specifies the CloudFormation template format version.

The value `2010-09-09` is the currently defined CloudFormation template format version.

It does not represent the date when the stack is created.

---

### `Description`

~~~yaml
Description: >
  Project 3 - Publishing Amazon SNS Messages Privately.
~~~

The `Description` field explains the purpose of the template.

The `>` YAML symbol creates a folded multiline string.

---

### `Parameters`

Parameters allow values to be supplied when the stack is created.

This makes the template reusable.

The template defines:

| Parameter | Default | Purpose |
| --------- | ------- | ------- |
| `VpcCidr` | `10.0.0.0/16` | Defines the VPC IPv4 address range. |
| `PublicSubnetCidr` | `10.0.1.0/24` | Defines the public subnet IPv4 range. |
| `InstanceType` | `t3.micro` | Defines the EC2 instance type. |
| `LatestAmiId` | SSM public parameter | Dynamically retrieves a current Amazon Linux 2023 AMI ID. |

---

## Resource Explanation 1: Custom VPC

~~~yaml
HospitalVPC:
  Type: AWS::EC2::VPC
~~~

This creates the custom Amazon VPC.

Important properties:

| Property | Value | Meaning |
| -------- | ----- | ------- |
| `CidrBlock` | `10.0.0.0/16` by default | Defines the VPC IPv4 address range. |
| `EnableDnsSupport` | `true` | Enables DNS resolution inside the VPC. |
| `EnableDnsHostnames` | `true` | Enables DNS hostnames for supported resources. |

Both DNS settings are important for using private DNS with the SNS interface endpoint.

---

## Resource Explanation 2: Internet Gateway

~~~yaml
HospitalInternetGateway:
  Type: AWS::EC2::InternetGateway
~~~

This creates an Internet Gateway.

The Internet Gateway is attached to the VPC by:

~~~yaml
HospitalInternetGatewayAttachment:
  Type: AWS::EC2::VPCGatewayAttachment
~~~

The public subnet uses the Internet Gateway for outbound internet connectivity and for Systems Manager communication in this lab architecture.

> The SNS publishing path itself will use the SNS interface VPC endpoint and AWS PrivateLink rather than the Internet Gateway.

---

## Resource Explanation 3: Public Subnet

~~~yaml
HospitalPublicSubnet:
  Type: AWS::EC2::Subnet
~~~

The subnet uses:

`10.0.1.0/24`

by default.

The property:

~~~yaml
MapPublicIpOnLaunch: true
~~~

automatically assigns a public IPv4 address to instances launched into this subnet when applicable.

For this educational lab, the public connectivity supports AWS Systems Manager communication without requiring additional interface endpoints for Systems Manager.

The SNS publishing request itself still uses the SNS interface endpoint because private DNS is enabled for the SNS endpoint.

---

## Resource Explanation 4: Route Table

The following resources create the public routing configuration:

- `HospitalPublicRouteTable`
- `HospitalDefaultRoute`
- `HospitalPublicSubnetRouteTableAssociation`

The default route is:

~~~text
Destination: 0.0.0.0/0
Target: Internet Gateway
~~~

This means traffic without a more-specific route can use the Internet Gateway.

However, DNS resolution for the regional SNS service hostname will direct SNS API requests to the private IP addresses of the interface VPC endpoint when private DNS is enabled.

---

## Resource Explanation 5: EC2 Security Group

~~~yaml
HospitalEC2SecurityGroup:
  Type: AWS::EC2::SecurityGroup
~~~

This security group is attached to the EC2 instance.

It contains no inbound rule.

Therefore:

- SSH port `22` is not opened.
- HTTP port `80` is not opened.
- HTTPS port `443` is not opened for inbound internet access.

The EC2 instance is intended to be managed through AWS Systems Manager Session Manager.

The outbound rule permits the instance to initiate required connections.

---

## Resource Explanation 6: SNS Endpoint Security Group

~~~yaml
HospitalSNSEndpointSecurityGroup:
  Type: AWS::EC2::SecurityGroup
~~~

This security group is attached to the SNS interface VPC endpoint.

Its inbound rule allows:

| Protocol | Port | Source |
| -------- | ---- | ------ |
| TCP | `443` | `HospitalEC2SecurityGroup` |

This means only resources associated with the EC2 security group can initiate HTTPS connections to the SNS endpoint.

---

## Resource Explanation 7: SNS Topic

~~~yaml
HospitalReportsTopic:
  Type: AWS::SNS::Topic
~~~

This creates the SNS topic:

`hospital-private-reports`

The SNS topic is the destination for the hospital report notification message.

The CloudFormation intrinsic function:

~~~yaml
!Ref HospitalReportsTopic
~~~

returns the SNS topic ARN.

An ARN follows a structure similar to:

~~~text
arn:aws:sns:ap-south-1:123456789012:hospital-private-reports
~~~

Your actual AWS account ID will be different.

---

## Resource Explanation 8: IAM Role

~~~yaml
HospitalEC2SNSRole:
  Type: AWS::IAM::Role
~~~

This role allows the EC2 service to assume it through:

~~~yaml
Principal:
  Service:
    - ec2.amazonaws.com
~~~

The role receives two categories of permissions.

### Permission 1: Systems Manager

~~~yaml
ManagedPolicyArns:
  - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
~~~

This AWS managed policy enables the instance to function as a Systems Manager managed node when network connectivity and the SSM Agent are available.

### Permission 2: Publish to the Hospital SNS Topic

~~~yaml
Action:
  - sns:Publish
Resource: !Ref HospitalReportsTopic
~~~

This follows the principle of least privilege.

The EC2 instance is allowed to publish to the intended project SNS topic rather than receiving unrestricted SNS administrative permissions.

---

## Resource Explanation 9: IAM Instance Profile

~~~yaml
HospitalEC2InstanceProfile:
  Type: AWS::IAM::InstanceProfile
~~~

An IAM role cannot be attached directly to an EC2 instance without an instance profile.

The relationship is:

~~~text
IAM Role
    |
    v
IAM Instance Profile
    |
    v
EC2 Instance
~~~

The instance profile makes the IAM role available to the EC2 instance.

---

## Resource Explanation 10: SNS Interface VPC Endpoint

~~~yaml
HospitalSNSEndpoint:
  Type: AWS::EC2::VPCEndpoint
~~~

The endpoint type is:

~~~yaml
VpcEndpointType: Interface
~~~

The SNS service name is dynamically generated:

~~~yaml
ServiceName: !Sub com.amazonaws.${AWS::Region}.sns
~~~

In Mumbai, this becomes:

~~~text
com.amazonaws.ap-south-1.sns
~~~

Private DNS is enabled:

~~~yaml
PrivateDnsEnabled: true
~~~

The endpoint is placed inside:

`Hospital-Public-Subnet`

The endpoint security group allows HTTPS from the EC2 security group.

The endpoint policy further restricts allowed access to:

- Action: `sns:Publish`
- Resource: The project SNS topic

The effective authorization path therefore includes:

~~~text
EC2 IAM Role
    |
    | Allows sns:Publish
    v
Hospital SNS Topic
    ^
    |
    | Endpoint policy allows sns:Publish
    |
SNS Interface VPC Endpoint
~~~

Both the IAM authorization and VPC endpoint policy must permit the requested operation.

---

## Resource Explanation 11: EC2 Instance

~~~yaml
HospitalEC2Instance:
  Type: AWS::EC2::Instance
~~~

The instance uses:

- Amazon Linux 2023.
- `t3.micro` by default.
- The custom hospital subnet.
- The EC2 security group.
- The IAM instance profile.

The template also configures:

~~~yaml
MetadataOptions:
  HttpEndpoint: enabled
  HttpTokens: required
~~~

This requires Instance Metadata Service Version 2, commonly called IMDSv2.

IMDSv2 improves protection of instance metadata and temporary IAM credentials.

---

## Step 7: Save the CloudFormation Template

### Action

Save the YAML template locally as:

`hospital-private-sns.yaml`

Make sure:

- The extension is `.yaml`.
- The file is not accidentally saved as `.yaml.txt`.
- YAML indentation is preserved.
- Tabs are not introduced.
- No lines are accidentally removed.

### Why is this step required?

CloudFormation requires a valid template to create the stack.

The template defines:

- Which resources to create.
- Their properties.
- Their dependencies.
- Their security relationships.
- Their outputs.

### Dependency

Depends on:

- **Step 6:** The complete CloudFormation template.

### What happens if this step is skipped?

There will be no template file to upload to AWS CloudFormation.

The stack cannot be created.

---

## Step 8: Create the CloudFormation Stack

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Create stack
→ With new resources (standard)
~~~

Depending on the current Console interface, if no stacks exist, you may instead see a direct **Create stack** button.

### Configuration

Under **Prerequisite - Prepare template**, select:

`Choose an existing template`

Under **Specify template**, select:

`Upload a template file`

Click:

`Choose file`

Select:

`hospital-private-sns.yaml`

Then click:

`Next`

### Why is this step required?

Uploading the template tells CloudFormation which infrastructure resources should be created.

### Dependency

Depends on:

- **Step 7:** The saved `hospital-private-sns.yaml` template.

### What happens if this step is skipped?

CloudFormation will not receive the infrastructure definition, so no project resources will be created.

---

## Step 9: Specify Stack Details

### Navigation

After uploading the template:

~~~text
CloudFormation
→ Create stack
→ Specify stack details
~~~

### Configuration

Use the following values:

| Setting | Value |
| ------- | ----- |
| Stack name | `hospital-private-sns-stack` |
| VpcCidr | `10.0.0.0/16` |
| PublicSubnetCidr | `10.0.1.0/24` |
| InstanceType | `t3.micro` |
| LatestAmiId | Keep the default Systems Manager public parameter value |

### Important Instance Type Note

If `t3.micro` is unavailable for your account or causes a capacity issue, select:

`t2.micro`

The allowed values in the template are:

~~~text
t2.micro
t3.micro
~~~

### Action

After reviewing all parameters, click:

`Next`

### Why is this step required?

CloudFormation parameters allow the template to receive deployment-specific values.

The stack name uniquely identifies the deployment.

### Dependency

Depends on:

- **Step 8:** Successful template upload.

### What happens if this step is skipped?

The stack cannot proceed to deployment because CloudFormation requires a stack name and valid parameter values.

---

## Step 10: Configure Stack Options

### Navigation

~~~text
CloudFormation
→ Create stack
→ Configure stack options
~~~

### Action

For this project:

- Leave stack tags optional unless you want to add them.
- Leave the permissions section at its default unless your AWS environment requires a specific CloudFormation service role.
- Leave stack failure options at their default.
- Leave advanced options at their defaults unless your course environment specifically requires different settings.

Click:

`Next`

### Why is this step required?

This page controls optional CloudFormation behavior, including:

- Tags.
- Permissions.
- Stack failure behavior.
- Rollback behavior.
- Notifications.
- Termination protection.

For this project, the default options are sufficient.

### Dependency

Depends on:

- **Step 9:** Stack details and parameter configuration.

### What happens if this step is skipped?

You cannot reach the final review page or start stack creation.

---

## Step 11: Review and Acknowledge IAM Resource Creation

### Navigation

~~~text
CloudFormation
→ Create stack
→ Review and create
~~~

### Action

Review:

- Template.
- Stack name.
- Parameters.
- Stack options.

Because the template creates named IAM resources, CloudFormation will display a capability acknowledgement.

Select the acknowledgement similar to:

~~~text
I acknowledge that AWS CloudFormation might create IAM resources with custom names.
~~~

This corresponds to:

`CAPABILITY_NAMED_IAM`

Then click:

`Submit`

The exact button wording can vary slightly with AWS Console updates.

### Why is this step required?

The template creates:

- A named IAM role.
- A named IAM instance profile.

AWS requires explicit acknowledgement before CloudFormation creates IAM resources with custom names.

This protects users from unknowingly deploying templates that create IAM identities or permissions.

### Dependency

Depends on:

- **Step 10:** Stack options configuration.

### What happens if this step is skipped?

CloudFormation will refuse to create the stack because the required IAM capability acknowledgement was not provided.

---

## Step 12: Monitor CloudFormation Stack Creation

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Events
~~~

### Action

Wait while CloudFormation creates the resources.

You will see statuses such as:

~~~text
CREATE_IN_PROGRESS
~~~

Individual resources will also show creation events.

Wait until the main stack reaches:

~~~text
CREATE_COMPLETE
~~~

### Expected Resource Creation Flow

CloudFormation determines dependencies automatically, but the effective resource flow is similar to:

~~~text
HospitalVPC
    |
    +--> HospitalInternetGateway
    |
    +--> HospitalPublicSubnet
    |
    +--> HospitalPublicRouteTable
    |
    +--> HospitalEC2SecurityGroup
    |
    +--> HospitalSNSEndpointSecurityGroup
    |
    +--> HospitalReportsTopic
    |
    +--> HospitalEC2SNSRole
    |
    +--> HospitalEC2InstanceProfile
    |
    +--> HospitalSNSEndpoint
    |
    v
HospitalEC2Instance
~~~

### Expected Final Stack Status

~~~text
hospital-private-sns-stack
CREATE_COMPLETE
~~~

### Why is this step required?

A submitted CloudFormation template does not mean that every resource was created successfully.

You must wait for:

`CREATE_COMPLETE`

before proceeding.

### Dependency

Depends on:

- **Step 11:** Stack submission.

### What happens if this step is skipped?

You may attempt to use:

- An EC2 instance that is not ready.
- An SNS topic that does not exist yet.
- A VPC endpoint that is still pending.
- An IAM role that has not finished propagating.

Later verification steps may fail.

---

## Step 13: Inspect the CloudFormation Resources

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Resources
~~~

### Action

Review the resources created by the stack.

You should see logical resource IDs similar to:

| Logical ID | Resource Type |
| ---------- | ------------- |
| `HospitalVPC` | `AWS::EC2::VPC` |
| `HospitalInternetGateway` | `AWS::EC2::InternetGateway` |
| `HospitalInternetGatewayAttachment` | `AWS::EC2::VPCGatewayAttachment` |
| `HospitalPublicSubnet` | `AWS::EC2::Subnet` |
| `HospitalPublicRouteTable` | `AWS::EC2::RouteTable` |
| `HospitalDefaultRoute` | `AWS::EC2::Route` |
| `HospitalPublicSubnetRouteTableAssociation` | `AWS::EC2::SubnetRouteTableAssociation` |
| `HospitalEC2SecurityGroup` | `AWS::EC2::SecurityGroup` |
| `HospitalSNSEndpointSecurityGroup` | `AWS::EC2::SecurityGroup` |
| `HospitalReportsTopic` | `AWS::SNS::Topic` |
| `HospitalEC2SNSRole` | `AWS::IAM::Role` |
| `HospitalEC2InstanceProfile` | `AWS::IAM::InstanceProfile` |
| `HospitalSNSEndpoint` | `AWS::EC2::VPCEndpoint` |
| `HospitalEC2Instance` | `AWS::EC2::Instance` |

### What should I observe?

Every required resource should have a successful status such as:

~~~text
CREATE_COMPLETE
~~~

### Why does this matter?

The Resources tab confirms which actual AWS resources are managed by the stack.

It is also useful for:

- Finding physical resource IDs.
- Troubleshooting failed resources.
- Understanding CloudFormation dependencies.
- Opening resources directly in their respective AWS service consoles.

### Dependency

Depends on:

- **Step 12:** Successful CloudFormation stack creation.

### What happens if this step is skipped?

The project may still work, but you lose an important opportunity to verify that every expected infrastructure component was actually created.

---

## Step 14: Inspect the CloudFormation Outputs

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Outputs
~~~

### Expected Outputs

You should see values similar to:

| Output Key | Example Value |
| ---------- | ------------- |
| `VpcId` | `vpc-0123456789abcdef0` |
| `PublicSubnetId` | `subnet-0123456789abcdef0` |
| `EC2InstanceId` | `i-0123456789abcdef0` |
| `SNSTopicArn` | `arn:aws:sns:ap-south-1:123456789012:hospital-private-reports` |
| `SNSEndpointId` | `vpce-0123456789abcdef0` |
| `SNSEndpointService` | `com.amazonaws.ap-south-1.sns` |

Your actual resource IDs and AWS account ID will be different.

### Why is this step required?

CloudFormation outputs provide convenient references to important created resources.

The `SNSTopicArn` will be needed later when publishing the private SNS message.

### Dependency

Depends on:

- **Step 12:** Stack status `CREATE_COMPLETE`.

### What happens if this step is skipped?

You can still find the resources through individual AWS service consoles, but later commands become more difficult because you must manually locate the SNS topic ARN and other resource identifiers.

---

## Step 15: Verify the Custom VPC

### Navigation

~~~text
AWS Management Console
→ VPC
→ Your VPCs
→ Select Hospital-VPC
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Name | `Hospital-VPC` |
| IPv4 CIDR | `10.0.0.0/16` |
| DNS resolution | `Enabled` |
| DNS hostnames | `Enabled` |

### What should I observe?

The VPC should:

- Exist in `ap-south-1`.
- Have the expected CIDR block.
- Have DNS support enabled.
- Have DNS hostnames enabled.

### Why does this confirm success?

The VPC is the network boundary for:

- The EC2 instance.
- The subnet.
- The security groups.
- The SNS interface endpoint.

Private DNS for the SNS endpoint depends on the VPC DNS configuration.

### Dependency

Depends on:

- Successful creation of `HospitalVPC`.

### What happens if this step is skipped?

The stack may still function, but you would not independently confirm the foundational network configuration.

---

## Step 16: Verify the Public Subnet

### Navigation

~~~text
AWS Management Console
→ VPC
→ Subnets
→ Select Hospital-Public-Subnet
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Name | `Hospital-Public-Subnet` |
| IPv4 CIDR | `10.0.1.0/24` |
| VPC | `Hospital-VPC` |
| Auto-assign public IPv4 address | Enabled by the CloudFormation subnet configuration |

### Why is this step required?

The subnet hosts:

- The EC2 instance.
- The network interface of the SNS VPC endpoint.

### Dependency

Depends on:

- `HospitalVPC`.

### What happens if this step is skipped?

You would not verify that the EC2 instance and endpoint have a valid subnet in which to operate.

---

## Step 17: Verify the SNS Interface VPC Endpoint

### Navigation

~~~text
AWS Management Console
→ VPC
→ Endpoints
→ Select the endpoint whose service name is com.amazonaws.ap-south-1.sns
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Endpoint type | `Interface` |
| Service name | `com.amazonaws.ap-south-1.sns` |
| VPC | `Hospital-VPC` |
| Subnet | `Hospital-Public-Subnet` |
| Private DNS | `Enabled` |
| Status | `Available` |

### Expected Status

~~~text
Available
~~~

Do not continue to SNS publishing until the endpoint reaches:

`Available`

### Why is this step required?

The SNS interface endpoint is the core resource that makes private SNS API access possible.

Without it, the project would not demonstrate publishing SNS messages through AWS PrivateLink.

### Dependency

Depends on:

- VPC.
- Subnet.
- Endpoint security group.
- SNS topic for the endpoint policy.

### What happens if this step is skipped?

The EC2 instance may attempt to publish before the endpoint is ready.

Possible errors include:

- Connection timeout.
- Endpoint connection failure.
- DNS resolution issues.
- Authorization errors if the endpoint policy is incorrect.

---

## Step 18: Verify the SNS Topic

### Navigation

~~~text
AWS Management Console
→ Amazon SNS
→ Topics
→ hospital-private-reports
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Topic name | `hospital-private-reports` |
| Topic type | `Standard` |
| Region | `ap-south-1` |

The topic ARN should resemble:

~~~text
arn:aws:sns:ap-south-1:123456789012:hospital-private-reports
~~~

### Why is this step required?

The EC2 instance needs a destination topic for the private publish operation.

### Dependency

The topic itself has no network dependency on the VPC.

However, in this architecture:

- The IAM policy references the topic.
- The VPC endpoint policy references the topic.
- The EC2 instance publishes to the topic.

### What happens if this step is skipped?

You may not notice that the intended SNS destination is missing or incorrectly named.

---

## Step 19: Verify the EC2 Instance

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select Hospital-SNS-Publisher
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Name | `Hospital-SNS-Publisher` |
| Instance state | `Running` |
| Instance type | `t3.micro` or selected alternative |
| VPC | `Hospital-VPC` |
| Subnet | `Hospital-Public-Subnet` |
| IAM role | `Hospital-EC2-SNS-Role` |
| Security group | `Hospital-EC2-SG` |

### Expected State

~~~text
Running
~~~

Also wait for instance status checks to report:

~~~text
2/2 checks passed
~~~

### Why is this step required?

The EC2 instance acts as the SNS message publisher.

It needs:

- Network connectivity.
- An IAM role.
- AWS CLI access.
- Access to the SNS interface endpoint.

### Dependency

Depends on:

- VPC.
- Subnet.
- Route configuration.
- EC2 security group.
- IAM instance profile.
- SNS interface endpoint.

### What happens if this step is skipped?

You may attempt to connect to an instance that is still initializing or has failed its health checks.

---

## Step 20: Verify the IAM Role Attached to EC2

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select Hospital-SNS-Publisher
→ Security
→ IAM role
~~~

Click the IAM role:

`Hospital-EC2-SNS-Role`

This opens the IAM console.

### Expected Permissions

The role should contain:

1. AWS managed policy:

`AmazonSSMManagedInstanceCore`

2. Inline policy:

`HospitalSNSPublishPolicy`

The inline policy should permit:

~~~text
Action:
sns:Publish

Resource:
Hospital private reports SNS topic ARN
~~~

### Why is this step required?

The EC2 instance requires authorization to call the SNS `Publish` API operation.

The role provides temporary AWS credentials automatically.

### Dependency

Depends on:

- IAM role.
- IAM instance profile.
- EC2 instance.

### What happens if this step is skipped?

The project may still work if the role is configured correctly, but you would not independently verify the least-privilege authorization configuration.

If the permission is missing, the later publish command will fail with an authorization error such as:

~~~text
AccessDenied
~~~

or:

~~~text
AuthorizationError
~~~

---

## Step 21: Verify Systems Manager Managed Node Availability

Before connecting to the EC2 instance, verify whether Systems Manager recognizes it as a managed node.

### Navigation

~~~text
AWS Management Console
→ Systems Manager
→ Fleet Manager
→ Managed nodes
~~~

Depending on the current AWS Console interface, the managed node may also be visible through:

~~~text
AWS Management Console
→ Systems Manager
→ Node Management
→ Fleet Manager
~~~

### Action

Look for the EC2 instance:

`Hospital-SNS-Publisher`

Wait several minutes if necessary after initial instance launch.

### Expected Result

The instance should appear as a managed node.

### Why is this step required?

Session Manager requires the EC2 instance to:

- Run the SSM Agent.
- Have the required IAM permissions.
- Have network connectivity to Systems Manager service endpoints.

Amazon Linux 2023 normally includes the SSM Agent.

### Dependency

Depends on:

- Running EC2 instance.
- `AmazonSSMManagedInstanceCore` policy.
- Working network connectivity.
- SSM Agent availability.

### What happens if this step is skipped?

You might proceed to Session Manager without first determining whether the instance has successfully registered.

---

## Step 22: Connect to the EC2 Instance Using Session Manager

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Select Hospital-SNS-Publisher
→ Connect
→ Session Manager
→ Connect
~~~

### Expected Result

A browser-based terminal should open.

You should see a shell prompt similar to:

~~~text
sh-5.2$
~~~

or another Linux shell prompt.

The exact prompt can vary.

### Why is this step required?

We need terminal access to the EC2 instance so that we can:

- Verify the AWS CLI.
- Verify the attached IAM identity.
- Retrieve the SNS topic ARN.
- Test DNS resolution.
- Publish the SNS message.
- Confirm successful private API communication.

### Dependency

Depends on:

- Running EC2 instance.
- SSM Agent.
- IAM permissions for Systems Manager.
- Network connectivity to Systems Manager.

### What happens if this step is skipped?

You cannot execute the AWS CLI commands required to publish and verify the SNS message from inside the VPC.

---

## Step 23: Verify the Operating System

### Command

Run:

~~~bash
cat /etc/os-release
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `cat` | Reads the contents of one or more files and displays them on standard output. |
| `/etc/os-release` | Standard Linux file containing operating system identification and version information. |
| `/` | Represents the root of the Linux filesystem. |
| `etc` | Directory containing system-wide configuration files. |
| `os-release` | File containing operating system metadata. |

### Expected Output

The output should contain information similar to:

~~~text
NAME="Amazon Linux"
VERSION="2023"
ID="amzn"
~~~

The exact version information may differ because the template dynamically resolves a current Amazon Linux 2023 AMI.

### Why is this step required?

It confirms that the instance is running the intended Amazon Linux operating system.

### Dependency

Depends on:

- Successful Session Manager connection.

### What happens if this step is skipped?

SNS publishing can still work, but you lose confirmation of the operating system environment used for the project.

---

## Step 24: Verify the AWS CLI

### Command

Run:

~~~bash
aws --version
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Invokes the AWS Command Line Interface. |
| `--version` | Displays the installed AWS CLI version and environment information. |
| `--` | Standard prefix used for a long-form command-line option. |

### Expected Output

The result should resemble:

~~~text
aws-cli/2.x.x Python/3.x.x Linux/x86_64
~~~

The exact versions will vary.

### Why is this step required?

The AWS CLI will be used to call the Amazon SNS API from the EC2 instance.

### Dependency

Depends on:

- Successful EC2 connection.
- AWS CLI installation in the selected Amazon Linux AMI.

### What happens if this step is skipped?

You may reach the SNS publishing command without confirming that the required CLI is available.

---

## Step 25: Verify the EC2 IAM Identity

### Command

Run:

~~~bash
aws sts get-caller-identity
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS Command Line Interface. |
| `sts` | Selects AWS Security Token Service. |
| `get-caller-identity` | Returns details about the IAM identity whose credentials are being used for the API request. |

### Expected Output

The output should resemble:

~~~json
{
    "UserId": "AROAEXAMPLE:i-0123456789abcdef0",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/Hospital-EC2-SNS-Role/i-0123456789abcdef0"
}
~~~

Your actual values will be different.

### What should I observe?

The `Arn` should contain:

~~~text
assumed-role/Hospital-EC2-SNS-Role/
~~~

This confirms that the EC2 instance is using temporary credentials from the intended IAM role.

### Why does this confirm success?

The AWS CLI successfully authenticated to AWS using the instance role.

No permanent IAM user access keys were manually configured on the instance.

### Why is this step required?

Before testing SNS, we must confirm which AWS identity is making the API request.

### Dependency

Depends on:

- EC2 IAM role.
- IAM instance profile.
- Working AWS API connectivity.

### What happens if this step is skipped?

If the later SNS command fails, you will have less information about whether the problem is related to:

- Missing credentials.
- Wrong credentials.
- Wrong IAM role.
- Insufficient IAM permissions.

---

## Step 26: Store the SNS Topic ARN in a Shell Variable

Run the following command:

~~~bash
TOPIC_ARN=$(aws sns list-topics --region ap-south-1 --query "Topics[?contains(TopicArn, 'hospital-private-reports')].TopicArn | [0]" --output text)
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `TOPIC_ARN` | Shell variable name used to store the SNS topic ARN. |
| `=` | Assignment operator that assigns the command result to the variable. |
| `$()` | Command substitution syntax. Executes the command inside the parentheses and substitutes its output. |
| `aws` | Runs the AWS Command Line Interface. |
| `sns` | Selects the Amazon Simple Notification Service command group. |
| `list-topics` | Requests the SNS topics accessible to the current IAM identity. |
| `--region` | Explicitly specifies the AWS Region for the API request. |
| `ap-south-1` | AWS Region code for Asia Pacific (Mumbai). |
| `--query` | Applies a JMESPath query to the AWS CLI response. |
| `"Topics[?contains(TopicArn, 'hospital-private-reports')].TopicArn \| [0]"` | Filters topic ARNs containing the project topic name and returns the first matching ARN. |
| `Topics` | The array of SNS topics returned by the API. |
| `[?contains(...)]` | JMESPath filter expression. |
| `TopicArn` | The ARN field of each SNS topic. |
| `'hospital-private-reports'` | The project SNS topic name used as the filter text. |
| `\|` | JMESPath pipe operator inside the query. Sends the filtered result to the next expression. |
| `[0]` | Selects the first matching result. |
| `--output text` | Returns the selected result as plain text rather than JSON. |

### Important Authorization Note

The least-privilege EC2 IAM policy created by the template grants only:

`sns:Publish`

It does not grant:

`sns:ListTopics`

Therefore, this command may return an `AccessDenied` error with the current strict least-privilege policy.

For this project, the preferred approach is to copy the topic ARN directly from the CloudFormation **Outputs** tab rather than broadening the EC2 role merely for convenience.

If the command returns `AccessDenied`, do not treat that as an infrastructure failure. Use the manual variable assignment method below.

### Preferred Manual Variable Assignment

Copy the exact `SNSTopicArn` value from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Outputs
→ SNSTopicArn
~~~

Then run:

~~~bash
TOPIC_ARN="arn:aws:sns:ap-south-1:YOUR_ACCOUNT_ID:hospital-private-reports"
~~~

Replace:

`YOUR_ACCOUNT_ID`

with your actual AWS account ID by copying the complete ARN from the CloudFormation Outputs tab.

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `TOPIC_ARN` | Shell variable that will hold the complete SNS topic ARN. |
| `=` | Assigns the value on the right to the variable on the left. |
| `"..."` | Double quotes preserve the ARN as one complete shell string. |
| `arn` | Amazon Resource Name prefix. |
| `aws` | AWS partition identifier. |
| `sns` | Identifies Amazon SNS as the service. |
| `ap-south-1` | Mumbai AWS Region. |
| `YOUR_ACCOUNT_ID` | Placeholder that must be replaced by the actual AWS account ID. |
| `hospital-private-reports` | Name of the SNS topic created by CloudFormation. |
| `:` | ARN field separator. |

### Why is this step required?

The upcoming `aws sns publish` command requires the exact destination topic ARN.

Storing the ARN in a variable:

- Reduces typing.
- Avoids repeated copying.
- Makes commands easier to read.
- Reduces accidental ARN errors.

### Dependency

Depends on:

- SNS topic creation.
- CloudFormation Outputs.
- Active shell session.

### What happens if this step is skipped?

You can still publish by writing the complete topic ARN directly in every command, but the command becomes longer and easier to mistype.

---

## Step 27: Display and Verify the SNS Topic ARN Variable

### Command

Run:

~~~bash
echo "$TOPIC_ARN"
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `echo` | Displays text or a variable value on standard output. |
| `"$TOPIC_ARN"` | Expands the shell variable and displays its stored value. |
| `$` | Tells the shell to expand the value of the named variable. |
| `TOPIC_ARN` | Name of the variable containing the SNS topic ARN. |
| `" "` | Double quotes preserve the expanded value safely as one argument. |

### Expected Output

~~~text
arn:aws:sns:ap-south-1:123456789012:hospital-private-reports
~~~

Your account ID will be different.

### What should I observe?

The output must:

- Begin with `arn:aws:sns:`.
- Contain `ap-south-1`.
- Contain your AWS account ID.
- End with `hospital-private-reports`.

### Why does this confirm success?

It confirms that the shell variable contains the correct SNS destination before the publishing command is executed.

### Dependency

Depends on:

- **Step 26:** Successful assignment of the `TOPIC_ARN` variable.

### What happens if this step is skipped?

If the variable is empty or incorrect, the later SNS publish command may fail with errors related to:

- Missing topic ARN.
- Invalid parameter.
- Invalid ARN.
- Unauthorized resource.

---

## Implementation Status After Part 2

At this point, the following infrastructure should exist:

~~~text
AWS CloudFormation Stack
        |
        +--> Custom Hospital VPC
        |
        +--> Public Subnet
        |
        +--> Internet Gateway
        |
        +--> Public Route Table
        |
        +--> EC2 Security Group
        |
        +--> SNS Endpoint Security Group
        |
        +--> SNS Topic
        |
        +--> IAM Role
        |
        +--> IAM Instance Profile
        |
        +--> SNS Interface VPC Endpoint
        |
        +--> EC2 Publisher Instance
~~~

The following should also be verified:

- CloudFormation stack status is `CREATE_COMPLETE`.
- SNS VPC endpoint status is `Available`.
- EC2 instance state is `Running`.
- EC2 instance status checks report `2/2 checks passed`.
- Systems Manager recognizes the EC2 instance.
- Session Manager terminal access works.
- AWS CLI is available.
- The EC2 instance uses `Hospital-EC2-SNS-Role`.
- The SNS topic ARN is stored in the `TOPIC_ARN` shell variable.

The next part will perform the core project operation: verify private DNS resolution, publish the hospital report notification from the EC2 instance to Amazon SNS through the interface VPC endpoint, verify the returned `MessageId`, inspect endpoint network interfaces, troubleshoot common failures, and clean up every resource in reverse dependency order.

---

# Part 3: Verify Private SNS Connectivity, Publish the Hospital Report Notification, Troubleshoot Issues, and Perform Complete Cleanup

This part completes the project by performing the following operations:

1. Verify that the Amazon SNS hostname resolves to private IP addresses associated with the SNS interface VPC endpoint.
2. Publish a synthetic hospital report notification from the EC2 instance.
3. Verify the returned Amazon SNS `MessageId`.
4. Verify the VPC endpoint and its network interfaces.
5. Confirm that the SNS publishing path uses the interface VPC endpoint.
6. Troubleshoot common deployment, connectivity, IAM, DNS, and publishing issues.
7. Delete all project resources after successful verification to minimize AWS charges.

---

## Step 28: Verify the Current AWS Region

Before testing private SNS connectivity, verify that the AWS CLI is operating in the intended AWS Region.

### Command

Run:

~~~bash
aws configure get region
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS Command Line Interface. |
| `configure` | Selects the AWS CLI configuration command group. |
| `get` | Retrieves a specific AWS CLI configuration value. |
| `region` | Requests the currently configured default AWS Region. |

### Expected Output

If a default Region is explicitly configured, the output should be:

~~~text
ap-south-1
~~~

However, because the EC2 instance is using an IAM role and no manual `aws configure` operation was required, the command may return no output.

That is not necessarily an error.

For all important commands in this project, we explicitly use:

`--region ap-south-1`

This ensures that the request is sent to the Mumbai Region.

### Why is this step required?

Amazon SNS topics and VPC endpoints are regional resources.

The SNS topic exists in:

`ap-south-1`

Therefore, the publishing request must target the same Region.

### Dependency

Depends on:

- Active Session Manager terminal.
- AWS CLI availability.

### What happens if this step is skipped?

The later command can still succeed because we explicitly specify `--region ap-south-1`.

However, checking the Region helps prevent troubleshooting confusion caused by sending requests to the wrong AWS Region.

---

## Step 29: Verify the SNS Topic ARN Variable Again

Because shell variables exist only in the current shell session, verify that `TOPIC_ARN` still contains the expected SNS topic ARN.

### Command

Run:

~~~bash
echo "$TOPIC_ARN"
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `echo` | Displays text or a variable value on standard output. |
| `"$TOPIC_ARN"` | Expands and displays the value stored in the `TOPIC_ARN` shell variable. |
| `$` | Tells the shell to substitute the variable with its stored value. |
| `TOPIC_ARN` | Variable containing the complete SNS topic ARN. |
| `" "` | Double quotes safely preserve the expanded value as one shell argument. |

### Expected Output

~~~text
arn:aws:sns:ap-south-1:123456789012:hospital-private-reports
~~~

Your actual AWS account ID will be different.

### If the Output Is Empty

If no ARN appears, assign the variable again.

Copy the exact topic ARN from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Outputs
→ SNSTopicArn
~~~

Then run:

~~~bash
TOPIC_ARN="arn:aws:sns:ap-south-1:YOUR_ACCOUNT_ID:hospital-private-reports"
~~~

Replace the example ARN with the exact ARN copied from your CloudFormation stack output.

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `TOPIC_ARN` | Name of the shell variable. |
| `=` | Shell assignment operator. Assigns the value on the right to the variable on the left. |
| `"arn:aws:sns:ap-south-1:YOUR_ACCOUNT_ID:hospital-private-reports"` | Complete SNS topic ARN that becomes the variable value. |
| `arn` | Amazon Resource Name prefix. |
| `aws` | AWS partition. |
| `sns` | Identifies Amazon Simple Notification Service. |
| `ap-south-1` | Mumbai AWS Region. |
| `YOUR_ACCOUNT_ID` | Placeholder for your actual AWS account ID. |
| `hospital-private-reports` | Name of the SNS topic. |
| `:` | Separator between ARN components. |

### Why is this step required?

The `aws sns publish` command requires a valid destination topic ARN.

If the variable is empty, the publishing command cannot identify the SNS topic.

### Dependency

Depends on:

- SNS topic creation.
- CloudFormation output availability.

### What happens if this step is skipped?

If `TOPIC_ARN` is empty, the publish command will fail because no valid SNS topic ARN is supplied.

---

## Step 30: Determine the Regional Amazon SNS Hostname

The regional SNS API hostname for Mumbai is:

~~~text
sns.ap-south-1.amazonaws.com
~~~

This is the standard regional SNS service hostname.

Because the SNS interface VPC endpoint has:

~~~text
PrivateDnsEnabled: true
~~~

DNS queries from inside the VPC for this standard SNS hostname should resolve to private IP addresses associated with the endpoint network interface.

The expected flow is:

~~~text
EC2 Instance
    |
    | DNS query for:
    | sns.ap-south-1.amazonaws.com
    v
VPC DNS Resolver
    |
    v
Private DNS configuration
    |
    v
SNS Interface Endpoint private IP address
    |
    v
AWS PrivateLink
    |
    v
Amazon SNS
~~~

### Why is this step required?

Understanding the hostname is important because the AWS CLI normally uses the regional AWS service endpoint automatically.

We do not need to hard-code the VPC endpoint's private IP address.

### Dependency

Depends on:

- VPC DNS support.
- VPC DNS hostnames.
- SNS interface VPC endpoint.
- Private DNS enabled on the endpoint.

### What happens if this step is skipped?

The publishing command can still work, but you would not understand how the standard SNS hostname is redirected to the private interface endpoint.

---

## Step 31: Verify Private DNS Resolution for Amazon SNS

### Command

Run:

~~~bash
getent hosts sns.ap-south-1.amazonaws.com
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `getent` | Queries system databases configured through the Name Service Switch mechanism. |
| `hosts` | Tells `getent` to query the hosts database, including DNS-based hostname resolution. |
| `sns.ap-south-1.amazonaws.com` | Standard regional Amazon SNS service hostname for the Mumbai Region. |
| `.` | Separates DNS hostname labels. |

### Expected Output

The result should contain one or more private IP addresses similar to:

~~~text
10.0.1.123    sns.ap-south-1.amazonaws.com
~~~

The exact private IP address will be different.

### What should I observe?

The returned address should normally belong to the private address range of the VPC subnet.

For this project:

| Network | CIDR |
| ------- | ---- |
| VPC | `10.0.0.0/16` |
| Endpoint subnet | `10.0.1.0/24` |

Therefore, an endpoint address similar to the following is expected:

~~~text
10.0.1.x
~~~

### Why does this confirm success?

The standard SNS regional hostname resolving to the private IP address of the interface endpoint demonstrates that private DNS is directing the SNS request toward the endpoint inside the VPC.

### Why is this step required?

This is one of the strongest practical checks that:

- Private DNS is enabled.
- DNS resolution works.
- The EC2 instance can discover the private SNS interface endpoint.

### Dependency

Depends on:

- VPC DNS support.
- SNS interface endpoint status `Available`.
- Private DNS enabled.

### What happens if this step is skipped?

The SNS publish operation may still succeed, but you would lose an important verification that DNS is directing the standard SNS hostname to a private endpoint IP address.

---

## Step 32: Verify That the Resolved Address Is Private

Private IPv4 address ranges include:

~~~text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
~~~

For this project, the VPC uses:

~~~text
10.0.0.0/16
~~~

and the subnet uses:

~~~text
10.0.1.0/24
~~~

Therefore, a result similar to:

~~~text
10.0.1.123
~~~

is a private IP address inside the project subnet.

### Why is this step required?

A private IP result provides evidence that the standard SNS hostname is resolving to the interface endpoint rather than a public SNS service IP address.

### Dependency

Depends on:

- **Step 31:** Successful DNS resolution.

### What happens if this step is skipped?

You may see an IP address without determining whether it is actually private or public.

---

## Step 33: Verify the EC2 Instance IAM Identity Before Publishing

Run the identity check again immediately before the SNS publish operation.

### Command

~~~bash
aws sts get-caller-identity --region ap-south-1
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS Command Line Interface. |
| `sts` | Selects AWS Security Token Service. |
| `get-caller-identity` | Returns the AWS account, user ID, and ARN associated with the credentials making the request. |
| `--region` | Explicitly specifies the AWS Region for the CLI operation. |
| `ap-south-1` | Region code for Asia Pacific (Mumbai). |

### Expected Output

~~~json
{
    "UserId": "AROAEXAMPLE:i-0123456789abcdef0",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/Hospital-EC2-SNS-Role/i-0123456789abcdef0"
}
~~~

Your actual values will be different.

### What should I observe?

The ARN should contain:

~~~text
assumed-role/Hospital-EC2-SNS-Role/
~~~

### Why does this confirm success?

It proves that the SNS publishing request will use temporary credentials from the EC2 IAM role rather than manually stored IAM user access keys.

### Dependency

Depends on:

- EC2 IAM role.
- IAM instance profile.
- AWS API connectivity.

### What happens if this step is skipped?

If publishing fails, you may not immediately know which AWS identity attempted the operation.

---

## Step 34: Publish the Synthetic Hospital Report Notification Privately

This is the core operation of the project.

Run:

~~~bash
aws sns publish \
  --topic-arn "$TOPIC_ARN" \
  --subject "Hospital Report Available" \
  --message "Patient ID: TEST-PATIENT-1001 | Report Status: Available | Your test medical report is ready for secure access." \
  --region ap-south-1
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS Command Line Interface. |
| `sns` | Selects Amazon Simple Notification Service commands. |
| `publish` | Calls the SNS Publish API operation to send a message to the specified SNS topic. |
| `\` | Shell line-continuation character. It allows one logical command to be written across multiple physical lines. |
| `--topic-arn` | Specifies the Amazon Resource Name of the destination SNS topic. |
| `"$TOPIC_ARN"` | Expands the shell variable containing the complete SNS topic ARN. |
| `$` | Tells the shell to substitute the variable with its stored value. |
| `--subject` | Specifies a subject for the SNS message where the delivery protocol supports subjects. |
| `"Hospital Report Available"` | Synthetic subject text used for the educational healthcare notification. |
| `--message` | Specifies the actual SNS message body. |
| `"Patient ID: TEST-PATIENT-1001 \| Report Status: Available \| Your test medical report is ready for secure access."` | Synthetic test message. It does not contain real patient information. |
| `\|` inside the quoted message | A literal visual separator because it appears inside quotation marks. It is not acting as a shell pipe operator. |
| `--region` | Explicitly selects the AWS Region for the SNS API request. |
| `ap-south-1` | Region code for Asia Pacific (Mumbai). |

### Expected Output

A successful operation should return JSON similar to:

~~~json
{
    "MessageId": "12345678-abcd-1234-abcd-123456789012"
}
~~~

The actual `MessageId` will be different.

### What should I observe?

The most important success indicator is:

~~~text
MessageId
~~~

If Amazon SNS returns a `MessageId`, the publish request was accepted successfully.

### Why does this confirm success?

The result demonstrates that:

- The EC2 instance has valid temporary AWS credentials.
- The IAM role permits `sns:Publish`.
- The endpoint policy permits the operation.
- DNS resolution is functioning.
- HTTPS connectivity to the SNS interface endpoint is functioning.
- Amazon SNS accepted the message.

### Why is this step required?

This is the main objective of the project:

> Publish an Amazon SNS message privately from an EC2 instance inside an Amazon VPC.

### Dependency

Depends on all of the following:

- Running EC2 instance.
- Correct IAM role.
- Valid SNS topic.
- SNS interface VPC endpoint.
- Endpoint security group.
- Private DNS.
- Correct topic ARN.
- HTTPS connectivity on TCP port `443`.

### What happens if this step is skipped?

The core project objective would not be completed.

The infrastructure might exist, but there would be no proof that the EC2 instance can successfully publish to Amazon SNS.

---

# Verification

## Verification Step 1: Verify the SNS Publish API Response

### Action

Inspect the output returned by:

~~~bash
aws sns publish \
  --topic-arn "$TOPIC_ARN" \
  --subject "Hospital Report Available" \
  --message "Patient ID: TEST-PATIENT-1001 | Report Status: Available | Your test medical report is ready for secure access." \
  --region ap-south-1
~~~

### Command Explanation

This command was completely explained in **Step 34**.

### Expected Output

~~~json
{
    "MessageId": "12345678-abcd-1234-abcd-123456789012"
}
~~~

### What should I observe?

Verify that:

- The command does not return an error.
- The JSON output contains `MessageId`.
- The `MessageId` contains a unique identifier.

### Why does this confirm success?

Amazon SNS generates and returns a message ID after accepting the publish request.

Therefore, a returned `MessageId` confirms successful message publication.

---

## Verification Step 2: Verify the SNS Topic in the AWS Console

### Action

Open the SNS topic created by CloudFormation.

### Navigation

~~~text
AWS Management Console
→ Amazon SNS
→ Topics
→ hospital-private-reports
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Topic name | `hospital-private-reports` |
| Topic type | `Standard` |
| Region | `Asia Pacific (Mumbai)` |
| Topic ARN | Ends with `hospital-private-reports` |

### What should I observe?

The topic should exist and its ARN should exactly match the value used in the `aws sns publish` command.

### Why does this confirm success?

It verifies that the API request targeted the intended SNS topic created for the project.

---

## Verification Step 3: Verify the SNS Interface Endpoint Status

### Action

Open the VPC endpoint.

### Navigation

~~~text
AWS Management Console
→ VPC
→ Endpoints
→ Select the endpoint for com.amazonaws.ap-south-1.sns
~~~

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| Endpoint type | `Interface` |
| Service name | `com.amazonaws.ap-south-1.sns` |
| Status | `Available` |
| VPC | `Hospital-VPC` |
| Private DNS | `Enabled` |

### Expected Status

~~~text
Available
~~~

### What should I observe?

Verify that:

- The endpoint is an `Interface` endpoint.
- The service is Amazon SNS.
- The endpoint belongs to the project VPC.
- Private DNS is enabled.
- The status is `Available`.

### Why does this confirm success?

An available SNS interface endpoint is necessary for the intended AWS PrivateLink publishing path.

---

## Verification Step 4: Verify the Endpoint Network Interface and Private IP Address

### Action

On the SNS VPC endpoint details page, inspect the subnet and network interface information.

Depending on the current AWS Console interface, the endpoint details may show the endpoint network interfaces directly.

You can also navigate to:

~~~text
AWS Management Console
→ EC2
→ Network Interfaces
~~~

Locate the network interface associated with the SNS VPC endpoint.

### Expected Configuration

| Setting | Expected Value |
| ------- | -------------- |
| VPC | `Hospital-VPC` |
| Subnet | `Hospital-Public-Subnet` |
| Private IP | Address from `10.0.1.0/24` |
| Security Group | `Hospital-SNS-Endpoint-SG` |

### Expected Private IP Example

~~~text
10.0.1.123
~~~

Your actual private IP address will differ.

### What should I observe?

Compare this endpoint private IP address with the result of:

~~~bash
getent hosts sns.ap-south-1.amazonaws.com
~~~

The private IP returned by DNS should correspond to the endpoint's private network interface.

### Why does this confirm success?

It connects two pieces of evidence:

~~~text
Standard SNS hostname
        |
        v
Private DNS resolution
        |
        v
Endpoint private IP
        |
        v
SNS interface endpoint
        |
        v
AWS PrivateLink
~~~

This demonstrates that the SNS service hostname resolves to the private endpoint interface inside the VPC.

---

## Verification Step 5: Repeat the Private DNS Check

### Command

~~~bash
getent hosts sns.ap-south-1.amazonaws.com
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `getent` | Queries system databases configured through the Linux Name Service Switch. |
| `hosts` | Selects hostname and IP-address resolution. |
| `sns.ap-south-1.amazonaws.com` | Standard Amazon SNS regional hostname for Mumbai. |

### Expected Output

~~~text
10.0.1.123    sns.ap-south-1.amazonaws.com
~~~

The actual address will differ.

### What should I observe?

The IP should be private and should correspond to the SNS endpoint's network interface.

### Why does this confirm success?

The standard SNS hostname is being resolved through private DNS to the endpoint network interface.

This is evidence that the SNS request is directed through the interface VPC endpoint.

---

## Verification Step 6: Publish a Second Verification Message

### Command

~~~bash
aws sns publish \
  --topic-arn "$TOPIC_ARN" \
  --subject "Private SNS Verification" \
  --message "Verification message published from Hospital-SNS-Publisher through the SNS interface VPC endpoint." \
  --region ap-south-1
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS CLI. |
| `sns` | Selects the Amazon SNS command group. |
| `publish` | Sends a message to an SNS topic. |
| `\` | Continues the command onto the next physical line. |
| `--topic-arn` | Specifies the destination SNS topic ARN. |
| `"$TOPIC_ARN"` | Expands the variable containing the topic ARN. |
| `--subject` | Specifies the message subject. |
| `"Private SNS Verification"` | Test subject used to identify the verification operation. |
| `--message` | Specifies the SNS message body. |
| `"Verification message published from Hospital-SNS-Publisher through the SNS interface VPC endpoint."` | Synthetic verification message. |
| `--region` | Explicitly specifies the AWS Region. |
| `ap-south-1` | Mumbai Region code. |

### Expected Output

~~~json
{
    "MessageId": "87654321-dcba-4321-dcba-210987654321"
}
~~~

The actual value will differ.

### What should I observe?

A new unique `MessageId` should be returned.

### Why does this confirm success?

A second successful API operation confirms that private SNS publishing is repeatable and not the result of a one-time transient condition.

---

## Verification Step 7: Verify the Effective Security Path

The successful implementation should have the following security relationships:

~~~text
EC2 Instance
    |
    | Uses temporary credentials
    v
Hospital-EC2-SNS-Role
    |
    | Allows sns:Publish
    | only to project SNS topic
    v
Hospital SNS Topic


EC2 Instance
    |
    | Associated with:
    v
Hospital-EC2-SG
    |
    | HTTPS TCP 443
    v
Hospital-SNS-Endpoint-SG
    |
    v
SNS Interface Endpoint
    |
    | Endpoint policy permits sns:Publish
    | to project SNS topic
    v
Amazon SNS
~~~

### What should I observe?

The implementation should use all three authorization and connectivity layers:

| Layer | Purpose |
| ----- | ------- |
| EC2 IAM role | Authorizes the EC2 instance to call `sns:Publish`. |
| Endpoint security group | Controls HTTPS network connectivity to the interface endpoint. |
| VPC endpoint policy | Restricts operations permitted through the endpoint. |

### Why does this confirm success?

A successful publish operation proves that all required authorization and network controls are aligned.

If one required layer denied the request, the operation would fail.

---

# Troubleshooting

## Troubleshooting Issue 1: CloudFormation Stack Shows `ROLLBACK_COMPLETE`

### Possible Cause

One or more resources failed during creation.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Events
~~~

### Action

Look for the first resource with:

~~~text
CREATE_FAILED
~~~

Read the **Status reason**.

Common causes include:

- IAM permission restrictions.
- Duplicate IAM role name.
- Duplicate SNS topic name.
- Unsupported EC2 instance type.
- Service quota issue.
- VPC endpoint creation failure.
- AMI resolution failure.

### Resolution

Fix the reported root cause.

If the stack is in `ROLLBACK_COMPLETE`, delete the failed stack and create it again after correcting the issue.

---

## Troubleshooting Issue 2: IAM Role Name Already Exists

### Possible Error

~~~text
Hospital-EC2-SNS-Role already exists
~~~

### Cause

A role with the same custom name already exists in the AWS account.

### Resolution

Either:

1. Delete the old role if it is unused and safe to delete.

Or:

2. Change this CloudFormation property:

~~~yaml
RoleName: Hospital-EC2-SNS-Role
~~~

to a unique name such as:

~~~yaml
RoleName: Hospital-EC2-SNS-Role-Project3
~~~

Also update any documentation references accordingly.

---

## Troubleshooting Issue 3: SNS Endpoint Is Stuck in `Pending`

### Navigation

~~~text
AWS Management Console
→ VPC
→ Endpoints
→ Select SNS endpoint
~~~

### Possible Causes

- Endpoint creation is still processing.
- Subnet configuration issue.
- Security group issue.
- AWS service-side delay.
- Account quota or permission issue.

### Action

Wait several minutes and refresh.

If the status eventually becomes:

~~~text
Available
~~~

continue.

If it becomes:

~~~text
Failed
~~~

inspect the endpoint and CloudFormation events for the detailed error.

---

## Troubleshooting Issue 4: `getent` Returns a Public IP Instead of `10.0.1.x`

### Possible Causes

- Private DNS is disabled.
- VPC DNS support is disabled.
- VPC DNS hostnames are disabled.
- Wrong SNS endpoint was created.
- Wrong Region is being tested.

### Required Configuration

Verify:

| Setting | Required Value |
| ------- | -------------- |
| VPC DNS support | `Enabled` |
| VPC DNS hostnames | `Enabled` |
| SNS endpoint private DNS | `Enabled` |
| Endpoint service | `com.amazonaws.ap-south-1.sns` |
| Endpoint VPC | `Hospital-VPC` |

### Resolution

Correct the configuration before claiming that SNS publishing is private.

---

## Troubleshooting Issue 5: SNS Publish Returns `AccessDenied`

### Possible Error

~~~text
An error occurred (AuthorizationError) when calling the Publish operation
~~~

or:

~~~text
AccessDenied
~~~

### Possible Causes

- IAM role does not permit `sns:Publish`.
- Topic ARN is incorrect.
- Endpoint policy denies the request.
- EC2 instance is using the wrong IAM role.

### Verification Command

~~~bash
aws sts get-caller-identity --region ap-south-1
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `aws` | Runs the AWS CLI. |
| `sts` | Selects AWS Security Token Service. |
| `get-caller-identity` | Displays the AWS identity associated with the active credentials. |
| `--region` | Explicitly specifies the Region. |
| `ap-south-1` | Mumbai Region. |

### Expected ARN Content

~~~text
assumed-role/Hospital-EC2-SNS-Role/
~~~

Also verify that the IAM policy permits:

~~~text
Action: sns:Publish
Resource: Exact SNS topic ARN
~~~

---

## Troubleshooting Issue 6: SNS Publish Times Out

### Possible Causes

- Endpoint security group does not allow TCP `443`.
- EC2 security group blocks outbound traffic.
- SNS endpoint is not `Available`.
- DNS does not resolve to the endpoint.
- Wrong VPC or subnet configuration.

### Required Endpoint Security Group Rule

| Direction | Protocol | Port | Source |
| --------- | -------- | ---- | ------ |
| Inbound | TCP | `443` | `Hospital-EC2-SG` |

### Verification Command

~~~bash
getent hosts sns.ap-south-1.amazonaws.com
~~~

### Expected Result

A private IP associated with the SNS endpoint should be returned.

---

## Troubleshooting Issue 7: Session Manager Does Not Connect

### Possible Causes

- EC2 instance is still initializing.
- SSM Agent has not registered.
- IAM role is missing `AmazonSSMManagedInstanceCore`.
- Instance lacks network connectivity to Systems Manager endpoints.
- IAM changes have not finished propagating.

### Verification

Navigate to:

~~~text
AWS Management Console
→ Systems Manager
→ Fleet Manager
→ Managed nodes
~~~

Verify that:

`Hospital-SNS-Publisher`

appears as a managed node.

### Resolution

Wait several minutes after instance launch.

Verify:

- EC2 state is `Running`.
- Status checks are `2/2 checks passed`.
- IAM role is attached.
- `AmazonSSMManagedInstanceCore` is attached to the role.
- The instance has the network connectivity required to contact Systems Manager.

---

## Troubleshooting Issue 8: `TOPIC_ARN` Is Empty

### Verification Command

~~~bash
echo "$TOPIC_ARN"
~~~

### Expected Output

~~~text
arn:aws:sns:ap-south-1:123456789012:hospital-private-reports
~~~

### Resolution

Copy the exact topic ARN from:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Outputs
→ SNSTopicArn
~~~

Then assign it:

~~~bash
TOPIC_ARN="PASTE_YOUR_EXACT_TOPIC_ARN_HERE"
~~~

### Command Breakdown

| Command Part | Explanation |
| ------------ | ----------- |
| `TOPIC_ARN` | Shell variable name. |
| `=` | Assigns the value on the right to the variable. |
| `"PASTE_YOUR_EXACT_TOPIC_ARN_HERE"` | Placeholder that must be replaced with the exact SNS topic ARN. |

---

## Troubleshooting Issue 9: `aws sns list-topics` Returns `AccessDenied`

This is expected with the strict least-privilege IAM policy used in this project.

The EC2 role has:

~~~text
sns:Publish
~~~

but not:

~~~text
sns:ListTopics
~~~

### Resolution

Do not grant broader permissions merely for convenience.

Instead, copy the topic ARN from:

~~~text
CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Outputs
→ SNSTopicArn
~~~

This preserves least-privilege access.

---

# Cleanup and Cost Optimization

After successful verification, delete all project resources immediately if they are no longer required.

The most important resource to remove promptly is the:

`SNS Interface VPC Endpoint`

because interface endpoints can incur hourly charges while provisioned.

Because most resources were created by one CloudFormation stack, the preferred cleanup method is to delete the CloudFormation stack.

CloudFormation then attempts to remove the stack-managed resources according to their dependency relationships.

The logical reverse dependency order is:

~~~text
EC2 Instance
    |
    v
SNS Interface VPC Endpoint
    |
    v
IAM Instance Profile
    |
    v
IAM Role
    |
    v
SNS Topic
    |
    v
Security Groups
    |
    v
Route Table Association
    |
    v
Route
    |
    v
Public Route Table
    |
    v
Internet Gateway Attachment
    |
    v
Internet Gateway
    |
    v
Public Subnet
    |
    v
Custom VPC
~~~

> Because these resources belong to the CloudFormation stack, do not manually delete individual stack-managed resources before stack deletion unless troubleshooting a failed stack deletion. Manual deletion can cause resource drift and complicate cleanup.

---

## Cleanup Step 1: Close the Session Manager Session

### Navigation

Inside the active Session Manager browser terminal:

~~~text
Session Manager terminal
→ End session
~~~

Depending on the interface, you may also close the session through the Session Manager controls.

### Action

End the active Session Manager session before deleting the EC2 instance.

### Why is this required?

The EC2 instance is about to be terminated as part of stack deletion.

Closing the session prevents confusion and ensures no active administrative work remains in progress.

### Dependency

Perform this before deleting the CloudFormation stack.

### What happens if this resource is not deleted?

The session will eventually become unusable when the EC2 instance terminates, but explicitly ending it provides cleaner operational practice.

---

## Cleanup Step 2: Delete the CloudFormation Stack

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select hospital-private-sns-stack
→ Delete
~~~

### Action

Select:

`hospital-private-sns-stack`

Click:

`Delete`

Confirm stack deletion when prompted.

### Why is this required?

The stack manages the project infrastructure.

Deleting it instructs CloudFormation to remove the resources created by the template, including:

- EC2 instance.
- SNS interface VPC endpoint.
- SNS topic.
- IAM instance profile.
- IAM role.
- Security groups.
- Public subnet.
- Route table.
- Internet Gateway.
- Custom VPC.

### Dependency

Perform this only after:

- SNS message publishing succeeds.
- A `MessageId` is returned.
- Private DNS resolution is verified.
- Endpoint private IP verification is complete.
- Any required screenshots or project evidence have been saved.

### What happens if this resource is not deleted?

Resources may remain active.

Potential consequences include:

- Interface VPC endpoint hourly charges.
- EC2 compute charges where applicable.
- EBS storage charges.
- Public IPv4 charges where applicable.
- Continued consumption of AWS promotional credits.
- Unused IAM resources remaining in the account.
- Unnecessary attack surface from forgotten cloud infrastructure.

---

## Cleanup Step 3: Monitor Stack Deletion

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Events
~~~

### Action

Monitor deletion progress.

You may see:

~~~text
DELETE_IN_PROGRESS
~~~

Wait until the stack is fully deleted.

Depending on the current CloudFormation Console view, deleted stacks may disappear from the default active-stack list.

### Expected Result

The stack should no longer appear among active stacks.

### Why is this required?

A delete request does not prove that every resource was successfully removed.

Monitoring the deletion process helps detect failures.

### Dependency

Depends on:

- **Cleanup Step 2:** Stack deletion request.

### What happens if this resource is not deleted?

If stack deletion fails, some resources can remain provisioned and potentially continue generating charges.

---

## Cleanup Step 4: Verify That the EC2 Instance Is Terminated

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
~~~

### Action

Search for:

`Hospital-SNS-Publisher`

### Expected Final State

The instance should eventually show:

~~~text
Terminated
~~~

or disappear from the default active instance view after AWS removes the historical record.

### Why is this required?

EC2 instances can incur compute charges while running, depending on account eligibility and usage.

### Dependency

The EC2 instance should be deleted by CloudFormation.

### What happens if this resource is not deleted?

Possible consequences include:

- Continued EC2 compute charges.
- Continued EBS storage charges.
- Public IPv4 charges where applicable.
- Continued consumption of promotional credits.

---

## Cleanup Step 5: Verify That the SNS Interface VPC Endpoint Is Deleted

### Navigation

~~~text
AWS Management Console
→ VPC
→ Endpoints
~~~

### Action

Search for:

~~~text
com.amazonaws.ap-south-1.sns
~~~

and verify that the endpoint created for this project no longer exists.

### Expected Final State

~~~text
Deleted
~~~

The endpoint should disappear from the active endpoint list.

### Why is this required?

The interface VPC endpoint is the most important chargeable networking resource in this project.

### Dependency

CloudFormation should remove it during stack deletion.

### What happens if this resource is not deleted?

The endpoint may continue generating hourly AWS PrivateLink interface endpoint charges.

This can continue consuming promotional credits or create a bill.

---

## Cleanup Step 6: Verify That the SNS Topic Is Deleted

### Navigation

~~~text
AWS Management Console
→ Amazon SNS
→ Topics
~~~

### Action

Search for:

`hospital-private-reports`

### Expected Final State

The topic should no longer appear.

### Why is this required?

It confirms that the messaging resource created for the project was successfully removed.

### Dependency

CloudFormation should remove the SNS topic during stack deletion.

### What happens if this resource is not deleted?

The topic remains in the account and could receive future messages if permissions and publishers remain configured.

It also creates unnecessary resource clutter.

---

## Cleanup Step 7: Verify That the IAM Role Is Deleted

### Navigation

~~~text
AWS Management Console
→ IAM
→ Roles
~~~

### Action

Search for:

`Hospital-EC2-SNS-Role`

### Expected Final State

The role should no longer exist.

### Why is this required?

Unused IAM roles should be removed to reduce unnecessary permissions and security exposure.

### Dependency

The EC2 instance profile and EC2 instance dependencies must be removed before the IAM role can be deleted successfully.

CloudFormation handles these dependencies automatically when stack deletion succeeds.

### What happens if this resource is not deleted?

The unused IAM role remains in the AWS account.

Although an unused IAM role does not normally create a direct charge, unnecessary IAM permissions increase administrative complexity and potential security exposure.

---

## Cleanup Step 8: Verify That the Security Groups Are Deleted

### Navigation

~~~text
AWS Management Console
→ EC2
→ Security Groups
~~~

### Action

Search for:

- `Hospital-EC2-SG`
- `Hospital-SNS-Endpoint-SG`

### Expected Final State

Both security groups should be deleted.

### Why is this required?

It confirms that the EC2 instance and VPC endpoint network dependencies were removed correctly.

### Dependency

Security groups can be deleted only after dependent resources and network interfaces stop using them.

### What happens if these resources are not deleted?

Unused security groups usually do not create direct charges, but they cause account clutter and can complicate future security management.

---

## Cleanup Step 9: Verify That the VPC Is Deleted

### Navigation

~~~text
AWS Management Console
→ VPC
→ Your VPCs
~~~

### Action

Search for:

`Hospital-VPC`

### Expected Final State

The VPC should no longer appear.

### Why is this required?

The VPC is the top-level network container for the project.

Its successful deletion indicates that dependent networking resources were also removed.

### Dependency

The following resources must be removed before the VPC can be deleted:

- EC2 instance.
- VPC endpoint.
- Endpoint network interfaces.
- Security groups other than the default VPC security group.
- Public subnet.
- Route table associations.
- Internet Gateway attachment.

CloudFormation manages these dependencies.

### What happens if this resource is not deleted?

The VPC itself may not have a direct hourly charge, but remaining dependent resources can cause:

- Unexpected charges.
- Resource clutter.
- Security management complexity.
- Future CIDR overlap issues.

---

## Cleanup Step 10: Check for Failed Stack Deletion

If the CloudFormation stack shows:

~~~text
DELETE_FAILED
~~~

do not assume cleanup is complete.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ hospital-private-sns-stack
→ Events
~~~

### Action

Find the resource whose status is:

~~~text
DELETE_FAILED
~~~

Read its **Status reason**.

Common causes include:

- A security group is still attached to a network interface.
- An endpoint network interface is still being removed.
- An IAM resource has an external dependency.
- A resource was manually modified outside CloudFormation.
- A resource was manually deleted, causing drift or inconsistent state.

### Why is this required?

A failed stack deletion can leave chargeable resources active.

### Dependency

Only required if the stack does not delete successfully.

### What happens if this step is skipped?

You may mistakenly believe that cleanup is complete while resources continue to exist and potentially generate charges.

---

## Final Resource Verification Table

After cleanup, verify the following final states:

| Resource | Expected Final State |
| -------- | -------------------- |
| CloudFormation stack `hospital-private-sns-stack` | `Deleted` |
| EC2 instance `Hospital-SNS-Publisher` | `Terminated` or no longer active |
| SNS interface VPC endpoint | `Deleted` |
| SNS topic `hospital-private-reports` | `Deleted` |
| IAM role `Hospital-EC2-SNS-Role` | `Deleted` |
| IAM instance profile | `Deleted` |
| EC2 security group `Hospital-EC2-SG` | `Deleted` |
| Endpoint security group `Hospital-SNS-Endpoint-SG` | `Deleted` |
| Public subnet `Hospital-Public-Subnet` | `Deleted` |
| Public route table `Hospital-Public-RT` | `Deleted` |
| Internet Gateway `Hospital-IGW` | `Deleted` |
| VPC `Hospital-VPC` | `Deleted` |

---

## Final Successful Architecture

Before cleanup, the successfully implemented architecture was:

~~~text
                    Amazon SNS
                         ^
                         |
                 AWS PrivateLink
                         |
                         |
              SNS Interface Endpoint
                  Private IP Address
                         ^
                         |
                   HTTPS TCP 443
                         |
                         |
               EC2 SNS Publisher
                         |
                         v
              Temporary IAM Credentials
                         |
                         v
              Hospital-EC2-SNS-Role
                         |
                         v
         Permission: sns:Publish to intended topic
~~~

The core successful publishing flow was:

~~~text
Hospital-SNS-Publisher EC2 Instance
        |
        | 1. Uses IAM role credentials
        v
aws sns publish
        |
        | 2. Resolves regional SNS hostname
        v
Private IP of SNS Interface Endpoint
        |
        | 3. HTTPS TCP 443
        v
AWS PrivateLink
        |
        | 4. Private AWS network path
        v
Amazon SNS Topic
        |
        | 5. Accepts the message
        v
Returns MessageId
~~~

A successful result similar to the following confirms that Amazon SNS accepted the privately published message:

~~~json
{
    "MessageId": "12345678-abcd-1234-abcd-123456789012"
}
~~~

The project objectives have therefore been completed:

1. AWS CloudFormation was used to create the VPC and supporting infrastructure.
2. The VPC was connected privately to Amazon SNS through an interface VPC endpoint.
3. AWS PrivateLink provided the private service connectivity path.
4. An EC2 instance published a synthetic hospital report notification to the SNS topic.
5. IAM least-privilege permissions controlled publishing authorization.
6. Private DNS resolved the regional SNS hostname to the endpoint's private IP address.
7. The successful SNS `MessageId` verified message publication.
8. All resources were cleaned up to minimize ongoing AWS charges.

---
