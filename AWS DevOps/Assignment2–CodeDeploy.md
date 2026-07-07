# Module 12: Assignment 2 - Automated Application Deployment via AWS CodeDeploy

## Problem Statement
XYZ Corporation aims to streamline its application delivery lifecycle by eliminating manual code deployments onto compute hosts. To support the development team's efficiency goals, configure an automated software release pipeline. Provision a dedicated Amazon CodeDeploy Application utilizing the Amazon EC2 compute platform tier, map a target Deployment Group bound to specific tagged cloud server instances, and configure the necessary IAM security roles to govern seamless deployment orchestrations.

---

## Tasks To Be Performed
1. Establish and configure secure IAM Roles enabling communication protocols between EC2 and CodeDeploy.
2. Create a standardized CodeDeploy Application utilizing the Amazon EC2 compute platform target.
3. Establish an active Deployment Group bound to a tagged target Amazon EC2 Linux instance.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision Required IAM Security Governance Frameworks
*(CodeDeploy requires cross-service authorization profiles before managing server assets).*

1. **Create the EC2 Instance Instance Profile Role:**
   * Navigate to the **IAM Console** > Click **Roles** > **Create role**.
   * **Trusted entity type:** AWS service | **Use case:** **EC2**. Click **Next**.
   * Attach policy: Search and check **`AmazonSSMManagedInstanceCore`** (Allows CodeDeploy agent setup and session triage). Click **Next**.
   * **Role name:** `XYZ-EC2-Deploy-Profile-Role` > Click **Create role**.

2. **Create the Core CodeDeploy Service Role:**
   * Click **Create role** again. Select **AWS service** > Choose **CodeDeploy** from the use case list.
   * The policy **`AWSCodeDeployRole`** will be attached automatically. Click **Next**.
   * **Role name:** `XYZ-CodeDeploy-Service-Role` > Click **Create role**.

---

### Step 2: Launch a Tagged Destination Compute Target (EC2)
1. Open the **Amazon EC2 Console** and click **Launch instances**.
2. Configure parameters:
   * **Name:** `XYZ-Production-Server`
   * **OS Image:** **Amazon Linux 2023 AMI**.
   * **Instance Type:** `t2.micro`.
   * **Advanced Details > IAM instance profile:** Select **`XYZ-EC2-Deploy-Profile-Role`**.
   * **CRITICAL STEP (Tags Section):** Click *Add new tag*. Set **Key = `Environment`** and **Value = `Production`**. *(CodeDeploy targets instances using these exact tags).*
3. Click **Launch instance**.

---

### Step 3: Create the AWS CodeDeploy Application (Task 1 Validation)
1. Navigate to the **Developer Tools > CodeDeploy** console dashboard interface.
2. In the left navigation vertical tree menu, click **Applications** > Click **Create application**.
3. Configure application identity vectors:
   * **Application name:** `XYZ-Core-Web-Application`
   * **Compute platform:** Select **EC2/On-premises** from the options dropdown menu list.
4. Click **Create application**.

---

### Step 4: Create the Targeted Deployment Group (Task 2 Validation)
1. Inside your newly initialized `XYZ-Core-Web-Application` overview panel, navigate to the **Deployment groups** tab layout grid and click **Create deployment group**.
2. Configure environment orchestration mechanics parameters:
   * **Deployment group name:** `XYZ-Production-Deploy-Group`
   * **Service role:** Paste or select the ARN string tracking **`XYZ-CodeDeploy-Service-Role`**.
   * **Deployment type:** Select **In-place** (Updates existing instances sequentially).
   * **Environment configuration:** Check the box marking **Amazon EC2 instances**.
     * Under the Tag group classification matrix, enter: **Key: `Environment`** | **Value: `Production`**. *(The tracking wizard will automatically display matching count: 1 unique instance connected).*
   * **Deployment configuration:** Select **`CodeDeployDefault.AllAtOnce`**.
   * **Load balancer:** Uncheck the *Enable load balancing* selector switch to simplify testing scopes.
3. Click **Create deployment group**. The deployment interface pipeline layout structure is now fully live and awaiting delivery manifests.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Purge the deployment automations and underlying hardware computing components upon project review to secure your testing token budget metrics cleanly:

### 1. Wipe out the CodeDeploy Deploy Group and Master Application Engine
1. Navigate to the **AWS CodeDeploy Console > Applications**.
2. Click on **`XYZ-Core-Web-Application`**.
3. Select the **Deployment groups** tab index, choose `XYZ-Production-Deploy-Group`, and click **Delete**. Confirm by typing the validation verification token inside the modal wrapper box.
4. Return up one directory tree level, highlight the core application line item entry `XYZ-Core-Web-Application`, and click **Delete application** to wipe out tracking pipelines.

### 2. Terminate the Destination Amazon EC2 Host Server Node
1. Navigate to the **Amazon EC2 Console > Instances**.
2. Highlight your tagged target compute node: **`XYZ-Production-Server`**.
3. Locate active controls and click **Instance state** > **Terminate instance**. Confirm structural destruction to stop active compute clock counts immediately.
