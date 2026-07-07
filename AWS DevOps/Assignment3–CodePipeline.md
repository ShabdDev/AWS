# Module 12: Assignment 3 - Continuous Deployment (CD) Automation via AWS CodePipeline

## Problem Statement
XYZ Corporation demands an automated, touchless code release mechanism to enhance developer efficiency and accelerate application delivery. To eliminate manual file handling, establish an active Continuous Deployment (CD) pipeline using AWS CodePipeline. This orchestrator must monitor an external GitHub version-control repository housing a baseline PHP application code bundle and automatically trigger direct updates onto a managed AWS Elastic Beanstalk PHP environment whenever code revisions are committed.

---

## Tasks To Be Performed
1. Leverage a simple PHP application hosted inside a GitHub repository to act as the primary operational deployment Source.
2. Build an active continuous release pipeline via Amazon CodePipeline that monitors the repository and executes automated software deliveries to an active AWS Elastic Beanstalk PHP destination container.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Pre-requisite Staging (Establish the Destination Beanstalk Tier)
*(Before connecting a pipeline, an active application endpoint target must exist).*
1. Log in to your **AWS Management Console** and navigate to the **AWS Elastic Beanstalk Console**.
2. Click **Create application**.
3. Configure application parameters:
   * **Application name:** `XYZ-Pipeline-App`
   * **Platform:** Select **PHP** from the platform runtime list options.
   * **Application code:** Choose **Sample application** *(This establishes a placeholder page while we construct our pipeline)*.
4. Advance through standard single-instance architecture options (Size: `t2.micro` or `t3.micro`) and click **Submit**. Wait roughly 3-5 minutes until environment health logs display a green **OK** check mark.

---

### Step 2: Establish the Continuous Deployment Cloud Pipeline
1. Navigate to the **Developer Tools > CodePipeline** service dashboard management panel.
2. Click on the orange **Create pipeline** button.
3. **Step 1: Choose pipeline settings:**
   * **Pipeline name:** `XYZ-Automation-Release-Pipeline`
   * **Service role:** Select **New service role** (Allows CodePipeline to securely orchestrate underlying operations). Click **Next**.

4. **Step 2: Add source stage (Task 1a Requirement):**
   * **Source provider:** Select **GitHub (Version 2)** from the integrations catalog dropdown list.
   * **Connection:** Click **Connect to GitHub**. Follow the secure authorization modal window prompt to link your GitHub profile and grant access to your repository.
   * **Repository name:** Select your target public/private PHP sample code project repository.
   * **Branch name:** Select your main code management trunk (e.g., `main` or `master`).
   * **Output artifact format:** Keep default *CodePipeline default* active. Click **Next**.

5. **Step 3: Add build stage:**
   * Since a baseline PHP application is interpretive and runs directly on server engines without compilation, a dedicated build layer is not needed for this lab.
   * Click **Skip build stage** and confirm the safety skip action popup.

6. **Step 4: Add deploy stage (Task 1b Requirement):**
   * **Deploy provider:** Select **AWS Elastic Beanstalk** from the delivery endpoint array menu.
   * **Region:** Ensure your selected tracking zone matches your active environment region.
   * **Application name:** Choose your target platform container: **`XYZ-Pipeline-App`**.
   * **Environment name:** Choose your live running server environment from the populated list filter: **`XyzPipelineApp-env`**.
   * Click **Next**.

7. **Step 5: Review:** Audit your configuration pipeline pipeline stages graph and click **Create pipeline**.

---

### Step 3: Validate End-to-End Delivery Executions
1. Upon creation, CodePipeline will run an initial execution pass. It connects to your GitHub repository, pulls the PHP application code artifacts, transfers them safely through the pipeline, and updates the Beanstalk server engine automatically.
2. Wait until both the **Source** and **Deploy** structural stages display a green **Succeeded** block.
3. **Trigger an Automated Deployment:**
   * Open your local machine terminal or navigate directly into your connected GitHub repository browser window interface.
   * Open the core `index.php` or `index.html` file, modify the visible text header (e.g., edit text to read: *"Automated Release Complete via CodePipeline 2026!"*), and commit/push the shift live to your branch.
   * Return immediately to the **AWS CodePipeline Dashboard**.
   * **Observation:** Within seconds, the pipeline engine will detect your Git commit signature, transition into an *In Progress* state automatically, and push the update live onto your Beanstalk servers without any human interaction. Refresh your Elastic Beanstalk endpoint URL link to see your new code changes live in production instantly!

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Wipe out these multi-tier continuous operations pipelines and behind-the-scenes virtual computing units immediately to maintain optimal credit metrics:

### 1. Wipe the Automation Release Pipeline Container
1. Open the **AWS CodePipeline Console** dashboard tracking interface window.
2. Select the line checkbox tracking item matching **`XYZ-Automation-Release-Pipeline`**.
3. Click the **Delete pipeline** control option button found on the header commands row, type `delete` to confirm, and process the wipe loop.

### 2. Terminate the AWS Elastic Beanstalk Web Tier Application Profile
1. Navigate directly to the **AWS Elastic Beanstalk Console**.
2. Click on **Applications** from the left vertical structural tree menu and click on your configuration matching **`XYZ-Pipeline-App`**.
3. Open the active running environment listing workspace window.
4. Locate the top interactive **Actions** utility component selection folder at the top right header space.
5. Click **Terminate environment**. Input the exact verification text string match `XyzPipelineApp-env` to pass confirmation safety checks. The backend engine will automatically drop load balancers and shut down the computing EC2 nodes within 5 minutes.
