# Module 12: Case Study - End-to-End Enterprise CI/CD Pipeline Orchestration via AWS DevOps Suite

## Problem Statement
XYZ Corporation is establishing a fully automated software development lifecycle (SDLC) framework natively within the AWS Cloud ecosystem. To optimize operations and maintain high availability, build a private source repository management infrastructure, implement a automated multi-stage continuous delivery pipeline, enforce structured continuous integration quality gates between Quality Assurance (QA) and Production deployment targets, and concurrently deploy the web application artifact onto a fully managed platform container.

---

## Tasks To Be Performed
1. Build a baseline application codebase and establish initial source tracking inside GitHub.
2. Mirror and migrate the external GitHub repository components into a private Amazon CodeCommit repository.
3. Configure dual-stage Amazon EC2 Target Groups mapping distinct Deployment Environments (`QA` and `Production`).
4. Architect a master continuous release pipeline using AWS CodePipeline with the following phases:
   * **Source Stage:** Private AWS CodeCommit repository engine monitoring changes.
   * **QA Deploy Stage:** Initial automated push into the dedicated QA environment.
   * **Sequential Stage Approval Gate:** Enforce strict sequential execution blocking production rollouts unless the QA layer passes successfully.
   * **Production Deploy Stage:** Subsequent deployment delivery into the live Production environment.
5. Append an independent **Third Stage** routing the finalized runtime artifacts onto an active AWS Elastic Beanstalk PHP hosting platform tier.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Initialize Local Codebase and Push to GitHub (Task 1)
1. On your local machine, open a text editor and create a simple file structure named **`index.php`**:
   ```php
   <?php echo "<h1>XYZ Corporation - Advanced DevOps Automation Platform v2026</h1>"; ?>

   ```

2. Create an deployment manifest specification file named **`appspec.yml`** *(Required by CodeDeploy to instruct agents on artifact mapping)*:
```yaml
version: 0.0
os: linux
files:
  - source: /index.php
    destination: /var/www/html/

```


3. Initialize local Git tracking, commit the two files, create a public/private repo on your GitHub profile, and execute a push sequence to host the application online.

---

### Step 2: Migrate GitHub Content to AWS CodeCommit (Task 2)

1. Log in to the **AWS Console** > Open **CodeCommit** > Click **Create repository**.
* **Repository name:** `xyz-private-core-pipeline-repo`
* Click **Create**. Copy the generated **HTTPS clone URL**.


2. Navigate to **IAM > Users > [Your Active IAM User Profile] > Security credentials**.
3. Scroll down to *AWS CodeCommit credentials for IAM users*, click **Generate credentials**, and save the secure Git username/password strings.
4. Launch your local terminal machine, download a bare mirror tracking index from GitHub, change directory, and mirror push it onto CodeCommit:
```bash
git clone --bare [https://github.com/your-github/sample-app.git](https://github.com/your-github/sample-app.git) temp-migrate
cd temp-migrate
git push --mirror [https://git-codecommit.us-east-1.amazonaws.com/v1/repos/xyz-private-core-pipeline-repo](https://git-codecommit.us-east-1.amazonaws.com/v1/repos/xyz-private-core-pipeline-repo)

```

*(Input the IAM Git credentials downloaded when prompted to finalize mirror ingestion).*
---

### Step 3: Setup Target Environments and CodeDeploy Formations (Task 3)

1. **Launch 2 Tagged Environment Hosts (EC2 Instances):**
* Go to **EC2 Dashboard > Launch instances** (Configure Number of instances to **2**).
* **OS Image:** Amazon Linux 2023 | **Size:** `t2.micro`.
* Configure an IAM instance profile containing the managed policy **`AmazonSSMManagedInstanceCore`** so CodeDeploy can operate.
* **Instance 1 Tag Configuration:** Key = `Stage`, Value = `QA`
* **Instance 2 Tag Configuration:** Key = `Stage`, Value = `Production`
* Click **Launch**. (Log into each node via SSM Session Manager and install the CodeDeploy Agent tool dependencies).


2. **Establish IAM Roles for Deployment Governance:**
* Open the **IAM Console > Roles > Create role**. Select **AWS service** > Choose **CodeDeploy**.
* Click Next through defaults, name the role **`XYZ-CodeDeploy-ServiceRole`**, and click create.

3. **Provision the Deploy Entities:**
* Open the **CodeDeploy Console > Applications** > Click **Create application**.
* **Application Name:** `xyz-enterprise-core-app` | **Compute Platform:** `EC2/On-premises`. Click create.
* Inside your new application panel, click **Create deployment group** for the QA environment:
* **Name:** `xyz-qa-deployment-group`
* **Service Role:** Select `XYZ-CodeDeploy-ServiceRole`
* **Environment Configuration:** Check *Amazon EC2 instances* > Add **Key: `Stage` | Value: `QA**`. Uncheck Load balancing. Click Create.


* Click **Create deployment group** a second time to configure the production environment:
* **Name:** `xyz-production-deployment-group`
* **Service Role:** Select `XYZ-CodeDeploy-ServiceRole`
* **Environment Configuration:** Check *Amazon EC2 instances* > Add **Key: `Stage` | Value: `Production**`. Uncheck Load balancing. Click Create.

---

### Step 4: Construct the Multi-Stage Sequential Release Pipeline (Task 4)

1. Open the **AWS CodePipeline Console** > Click **Create pipeline**.
2. **Settings:** Name it `xyz-master-release-lifecycle`, choose *New service role*, and click **Next**.
3. **Source Stage (Task 4a):**
* **Source Provider:** Select **AWS CodeCommit**.
* **Repository Name:** `xyz-private-core-pipeline-repo`
* **Branch Name:** `main` (or `master`). Click **Next**.


4. **Build Stage:** Click **Skip build stage** and confirm.
5. **Deploy Stage - Initial QA Delivery (Task 4b & 4c):**
* **Deploy Provider:** Select **AWS CodeDeploy**.
* **Application Name:** `xyz-enterprise-core-app`
* **Deployment Group:** Select **`xyz-qa-deployment-group`**. Click **Next**, then click **Create pipeline**.


6. **Inject Sequential Production and Gating Stages (Task 4d):**
* Once the baseline pipeline opens, click **Edit** at the top header pane.
* Scroll down past the QA stage and click **+ Add stage**. Name this new stage **`Production-Release-Stage`**.
* Inside the newly drawn stage box framework, click **+ Add action group**.
* **Action name:** `Deploy-To-Prod`
* **Action provider:** Select **AWS CodeDeploy**.
* **Input artifacts:** `SourceArtifact`
* **Application Name:** `xyz-enterprise-core-app`
* **Deployment Group:** Select **`xyz-production-deployment-group`**. Click **Done**.


*(Note: Because CodePipeline processes stages linearly, the Production stage will only execute if the previous QA Deploy stage registers a successful execution pass status).*

---

### Step 5: Append the Elastic Beanstalk Destination Tier (Task 5)

1. **Pre-requisite platform setup:** Open the **Elastic Beanstalk Console** in a separate window tab > click **Create application**.
* Name it `xyz-beanstalk-container-target`, set Platform runtime to **PHP**, choose *Sample application*, and submit using baseline single-instance configurations. Wait until its indicator registers a green OK status flag.


2. Return to your active editing canvas layout inside the **AWS CodePipeline** configurations page.
3. Scroll to the absolute bottom section of the visual orchestration diagram tree blueprint, click **+ Add stage**, and name it **`Elastic-Beanstalk-Deployment-Stage`**.
4. Click **+ Add action group** within the new container space:
* **Action name:** `Deploy-To-Beanstalk-PaaS`
* **Action provider:** Select **AWS Elastic Beanstalk**.
* **Input artifacts:** `SourceArtifact`
* **Application Name:** `xyz-beanstalk-container-target`
* **Environment Name:** Select the active live server instance (e.g., `Xyzbeanstalkcontainertarget-env`).


5. Click **Done** on the action setup panel and click the orange **Save** button at the top header to write changes live into your AWS account profile configurations.
---

### Step 6: Validate End-to-End SDLC Execution Loops

1. Click **Release change** on your master CodePipeline console window.
2. Monitor progress: CodePipeline connects to CodeCommit, downloads code assets, pushes them automatically onto the QA EC2 instance, verifies successful completion, shifts down to the Production stage to execute the second EC2 code delivery, and finishes by packaging and pushing the runtime directly onto the Elastic Beanstalk platform environment cleanly.
3. Open any connected browser window and paste the Public IPs or domain routes associated with all three testing targets to confirm identical web outputs display perfectly across the board.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

To prevent active billing charges across these heavy deployment suites and automation host clusters, tear down all active components immediately using this sequential cleanup checklist:

### 1. Wipe the Master Automation Release Pipeline Architecture

1. Open the **AWS CodePipeline Console** list grid view.
2. Check the box group matching **`xyz-master-release-lifecycle`**, click **Delete pipeline**, type confirmation tokens inside the input block, and process deletion.

### 2. Terminate the AWS Elastic Beanstalk Environment Target

1. Open the **AWS Elastic Beanstalk Console > Applications**.
2. Click into `xyz-beanstalk-container-target` -> click its active environment container panel -> click **Actions** -> choose **Terminate environment**. Type structural verification strings to safely drop all background load balancers and server components automatically.

### 3. Clear CodeDeploy Artifacts and Applications

1. Open **AWS CodeDeploy > Applications** > Click on `xyz-enterprise-core-app`.
2. Delete both the `xyz-qa-deployment-group` and `xyz-production-deployment-group` instances inside the tracking panel.
3. Go back one folder level, check the row index next to `xyz-enterprise-core-app`, and click **Delete application** to wipe tracking definitions.

### 4. Purge Private Cloud Source Repository Elements

1. Open the **AWS CodeCommit Console** dashboard screen.
2. Highlight your target repository row matching **`xyz-private-core-pipeline-repo`**.
3. Locate the administrative headers utility menu options, select **Delete repository**, type `delete` to verify security parameters, and process permanent destruction.

### 5. Terminate All Underlying Cloud Compute Instances

1. Open the **Amazon EC2 Console > Instances**.
2. Select the checkbox groups tracking **both** active environment virtual node components simultaneously.
3. Choose **Instance state** > **Terminate instance** > confirm actions to drop the virtual machines from the infrastructure grids cleanly.
