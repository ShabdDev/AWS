# Module 8: Case Study - Automated Multi-Tier Architecture Deployment via CloudFormation

## Problem Statement
XYZ Corporation is launching a new web application. Due to the unavailability of system administrators, the development team requires a fully automated, scalable Infrastructure as Code (IaC) blueprint to deploy and test their code safely. The stack must establish a secure 3-Tier network architecture (Web, App, and Database layers), integrate custom Route 53 domain routing, abstract infrastructure management away from developers, and enforce data retention safeguards preventing the accidental destruction of the RDS database when testing environments are dismantled.

---

## Tasks To Be Performed
1. **Web Tier:** Launch an EC2 instance in a public subnet allowing inbound HTTP (80) and SSH (22) traffic from the internet.
2. **Application Tier:** Launch an EC2 instance in a private subnet allowing inbound SSH (22) traffic *only* from the Web Tier subnet.
3. **Database Tier:** Provision an Amazon RDS MySQL database in an isolated private subnet allowing inbound traffic on port 3306 *only* from the Application Tier subnet.
4. **DNS Management:** Configure an Amazon Route 53 Hosted Zone and map an explicit routing record directly to the Web Tier frontend endpoint.
5. **Architectural Proposals:**
   * Propose a developer-centric automation framework allowing self-service code testing without infrastructure management overhead.
   * Configure CloudFormation policies ensuring that if a developer deletes the testing stack, the RDS production/persistent databases are **not** deleted.

---

## Part 1: Proposed Solutions (Management Questions)

### Proposal 1: Developer Self-Service Framework
To allow developers to focus entirely on testing code rather than provisioning, configuring, and updating infrastructure assets, XYZ Corporation should implement **AWS Elastic Beanstalk** or **AWS Service Catalog**:
* **AWS Elastic Beanstalk:** Developers simply upload their code bundles (Java, Node.js, Python, PHP, etc.). Beanstalk automatically handles the underlying provisioning of load balancers, EC2 auto-scaling groups, OS patching, and application health monitoring, removing the need for dedicated system administrators.

### Proposal 2: Database Retention Safeguard Policy
To guarantee that the managed RDS MySQL instance is not deleted when a developer executes a stack destruction cycle, we inject a **`DeletionPolicy: Retain`** metadata attribute statement directly onto the Database logical resource component block inside the CloudFormation template configuration. This instructs the cloud formation engine to orphan and preserve the live storage cluster intact while safely wiping the networking layers.

---

## Part 2: Step-by-Step Implementation Solution

### Step 1: Create the Comprehensive 3-Tier CloudFormation Template
1. Open a text editor (e.g., Notepad or VS Code) on your local machine.
2. Paste the following complete **YAML** multi-tier topology script:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Automated Multi-Tier Stack: Web Tier (Public EC2), App Tier (Private EC2), and Database Tier (RDS MySQL with Deletion Policy Retain).'

Resources:
  # --- NETWORKING TIER (VPC & SUBNETS) ---
  XYZCorpVPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: '10.100.0.0/16'
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: 'XYZ-MultiTier-VPC'

  PublicWebSubnet:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref XYZCorpVPC
      CidrBlock: '10.100.1.0/24'
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: 'XYZ-Web-PublicSubnet'

  PrivateAppSubnet:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref XYZCorpVPC
      CidrBlock: '10.100.2.0/24'
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: 'XYZ-App-PrivateSubnet'

  PrivateDBSubnetA:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref XYZCorpVPC
      CidrBlock: '10.100.3.0/24'
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: 'XYZ-DB-PrivateSubnetA'

  PrivateDBSubnetB:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref XYZCorpVPC
      CidrBlock: '10.100.4.0/24'
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: 'XYZ-DB-PrivateSubnetB'

  # --- GATEWAYS & ROUTING ---
  XYZInternetGateway:
    Type: 'AWS::EC2::InternetGateway'
  
  VPCGatewayAttachment:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref XYZCorpVPC
      InternetGatewayId: !Ref XYZInternetGateway

  PublicRouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref XYZCorpVPC

  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref XYZInternetGateway

  PublicSubnetAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicWebSubnet
      RouteTableId: !Ref PublicRouteTable

  # --- SECURITY GROUPS (TASK 1, 2, 3 REQUIREMENTS) ---
  WebSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow HTTP and SSH from internet'
      VpcId: !Ref XYZCorpVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: '0.0.0.0/0'
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: '0.0.0.0/0'

  AppSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow SSH ONLY from Web Tier Subnet'
      VpcId: !Ref XYZCorpVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref WebSecurityGroup

  DBSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Allow MySQL 3306 ONLY from App Tier Subnet'
      VpcId: !Ref XYZCorpVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup

  # --- COMPUTE INSTANCES ---
  WebInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: 't2.micro'
      ImageId: 'ami-0c55b159cbfafe1f0' # Standard baseline hardware pointer template
      SubnetId: !Ref PublicWebSubnet
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: 'XYZ-Web-Server'

  AppInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: 't2.micro'
      ImageId: 'ami-0c55b159cbfafe1f0'
      SubnetId: !Ref PrivateAppSubnet
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      Tags:
        - Key: Name
          Value: 'XYZ-App-Server'

  # --- DATABASE TIER (TASK 3 & PROPOSAL 2 SEPARATION PROTECTION) ---
  RDSDBSubnetGroup:
    Type: 'AWS::RDS::DBSubnetGroup'
    Properties:
      DBSubnetGroupDescription: 'Subnets available for multi-az RDS instance allocation'
      SubnetIds:
        - !Ref PrivateDBSubnetA
        - !Ref PrivateDBSubnetB

  MySQLDatabaseInstance:
    Type: 'AWS::RDS::DBInstance'
    DeletionPolicy: Retain # CRITICAL REQUIREMENT: Prevents destruction on stack deletion
    Properties:
      AllocatedStorage: '20'
      DBInstanceClass: 'db.t3.micro'
      DBSubnetGroupName: !Ref RDSDBSubnetGroup
      Engine: 'mysql'
      MasterUsername: 'dbadmin'
      MasterUserPassword: 'XYZSecurePassword2026'
      VPCSecurityGroups:
        - !Ref DBSecurityGroup

```

3. Save this script file locally on your desktop workstation dashboard as **`multi-tier-architecture.yaml`**.

---

### Step 2: Deploy the Architecture via CloudFormation Stacks

1. Log in to the **AWS Management Console** and navigate to the **CloudFormation Console**.
2. Click **Create stack** > **With new resources (standard)**.
3. Select **Template is ready**, upload `multi-tier-architecture.yaml`, and click **Next**.
4. **Specify stack options:** Name the stack `XYZ-MultiTier-Application-Stack` and click **Next**.
5. Complete verification steps on review forms and click **Submit**. Wait approximately 7-10 minutes until status outputs print **CREATE_COMPLETE**.

---

### Step 3: Map Endpoint routing via Route 53 (Task 4 Requirement)

1. Open the **Route 53 Dashboard** > Go to **Hosted zones**.
2. Select the corporate domain registry zone generated in previous module assessments.
3. Click **Create record**.
4. Configure values:
* **Record name:** Type a sub-domain link prefix (e.g., `app`).
* **Record type:** `A - Routes traffic to an IPv4 address and some AWS resources`.
* **Value:** Paste the explicit Public IPv4 Address corresponding to your deployed `XYZ-Web-Server` (WebInstance).


5. Click **Create records** to push live synchronization rules across edge networks.

---

## Part 3: Step-by-Step Deletion Process (Verification of Safeguards)

Execute this workflow to verify that your data asset retention architecture operates perfectly while cleanly recovering computing nodes to optimize budget management:

### 1. Delete the CloudFormation Infrastructure Stack

1. Return to the **AWS CloudFormation Dashboard** and select **Stacks**.
2. Check the box item corresponding to `XYZ-MultiTier-Application-Stack`.
3. Locate the top toolbar action control menu block and click **Delete** > Confirm **Delete stack**.
4. Monitor the processing status logs carefully. CloudFormation will systematically tear down EC2 instances, drop security group rules, clear custom route mappings, and wipe network subnet structures.
5. Once the log registers **DELETE_COMPLETE**, perform validation.

### 2. Verify Database Retention Persistence (Task 5 Validation)

1. Search and open the **Amazon RDS Dashboard** from the navigation bar.
2. Click on the **Databases** view.
3. **Observation:** You will notice that while the global network structures are gone, your database instance (`MySQLDatabaseInstance`) remains online, running securely in an orphaned detached state. This validates that the `DeletionPolicy: Retain` safeguard functioned successfully.
4. **Final Clean-up:** Since this is a temporary training lab environment, manually select the remaining independent RDS row instance element, click **Actions > Delete**, uncheck final snapshot flags, authorize via the validation text string statement, and permanently wipe the asset to prevent any lingering resource tracking charges.
