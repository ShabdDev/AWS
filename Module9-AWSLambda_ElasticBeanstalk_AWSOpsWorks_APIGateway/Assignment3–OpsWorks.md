# Module 9: Assignment 3 - Configuration Management and Code Deployment via AWS OpsWorks

## Problem Statement
XYZ Corporation is launching a new web-based infrastructure and requires automated configuration management to coordinate uniform software states across a dynamic fleet of hosts. Implement AWS OpsWorks (Stacks and Layers) utilizing Chef cookbooks to provision a base application server environment, scale the cluster horizontally by introducing multiple compute instances, and validate centralized code deployment synchronization across all live target hosts.

---

## Tasks To Be Performed
1. Create a sample AWS OpsWorks Chef stack configuration framework, initialize the core instances, and deploy the application.
2. Scale the tier horizontally by appending exactly 2 additional `t2.medium` EC2 compute instances onto the layer configuration rules.
3. Push a code change to the connected Git repository source tracking and execute a deployment synchronization to verify all instances update uniformly.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create an OpsWorks Stack and Base Web Layer
1. Log in to the **AWS Management Console** and search for **AWS OpsWorks** to open its orchestration platform.
2. Click on **Add stack** > Select **Chef Infra Stack** (or sample stack wizard configuration based on console layout options).
3. Configure **Stack Settings**:
   * **Name:** `XYZ-Corporate-Chef-Stack`
   * **Region:** Choose your active tracking territory (e.g., *us-east-1*).
   * **VPC & Subnet:** Keep Default VPC configurations active.
4. Under **App Repository Location**, configure a public sample repository target (e.g., a simple Git endpoint containing basic web configuration templates like standard HTML or node configurations). Click **Add stack**.
5. **Create a Layer:** Inside the stack control pane, click **Add a layer**.
   * **Layer type:** Select **Web App Server** or standard custom runtime tier.
   * Click **Add layer**.

---

### Step 2: Initialize Core Compute Node and Deploy Application (Task 1 Requirement)
1. Within your newly created layer block, click on **Instances** > **Add an instance**.
2. Set configuration properties:
   * **Size:** Select `t2.micro` (or default recommended size for base environment mapping).
   * Click **Add instance**.
3. Locate your newly configured node row element and click **Start** under the *Actions* header column.
4. *Wait approximately 5 minutes for AWS to spin up the EC2 virtual framework, pull configuration scripts, and transition to a green **Online** state flag status.*
5. Navigate to the **Apps** panel workspace block from the left sidebar, click **Add an app**, reference your repository layout metadata, and click **Deploy App** to run the initial code launch cycle.

---

### Step 3: Scale Horizontally with 2 Additional t2.medium Instances (Task 2 Requirement)
1. Navigate back to the **Instances** workspace layout framework structure of your active layer.
2. Click **+ Add an instance** to register the second instance slot:
   * **Instance Type / Size:** Modify the dropdown menu option to explicitly select **`t2.medium`**.
   * Click **Add instance**.
3. Click **+ Add an instance** again to register the third instance slot:
   * **Instance Type / Size:** Select **`t2.medium`**.
   * Click **Add instance**.
4. You will now see two new infrastructure units sitting in a *Stopped* state. Click **Start All Instances** (or m m click *Start* on each individual `t2.medium` instance row container).
5. Wait for the initialization cycle to complete until their health monitoring registers as a green **Online** status sign indicator.

---

### Step 4: Update Repository Code and Test Synchronized Deployment (Task 3 Requirement)
1. Navigate to your external connected source control interface (e.g., your GitHub account project page holding the testing code tracking module).
2. Open your core structural template file (e.g., `index.html` or `index.php`) and perform a minor visual layout change (e.g., change header text from *"Testing Phase"* to *"XYZ Production Grade Pipeline Active 2026"*). Commit and push the updates live to your branch.
3. Return to the **AWS OpsWorks Management Dashboard** > click **Apps** from the sidebar sub-directory tree view.
4. Select your application entry and click on the **Deploy** utility command action button.
5. Under **Command options**, ensure the execution target configuration is explicitly flagged to **Deploy** (this initiates an automatic pull loop calling the fresh git revisions).
6. Click **Deploy**. OpsWorks will automatically broadcast deployment commands down across all 3 running instances concurrently.
7. **Verification:** Open the individual public IP endpoint paths corresponding to each of the three active nodes. All distinct connections will successfully render the identical modified message text string, validating absolute synchronization.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Because OpsWorks locks down active EC2 compute structures (including multiple non-free tier `t2.medium` node hardware tiers) that track runtime hourly fees constantly, complete this teardown manual sequence immediately:

### 1. Stop and Clean All Layer Clusters
1. Inside the **AWS OpsWorks Dashboard**, select **Instances** from your left sidebar options checklist directory.
2. Locate the top collective control operations button and click **Stop All Instances**.
3. *Wait for approximately 5 minutes for the automation processes to cleanly shut down the systems and transition their labels into a gray **Stopped** notification flag.*
4. Once completely stopped, navigate to each instance line element row container and click the **Delete** button (X mark) to completely remove the hardware mappings from the cluster registry definitions.

### 2. Disassemble and Delete the OpsWorks Stack Profile Container
1. Once all underlying instances are completely wiped out from the layer configuration dashboards, look at the left-hand tracking menu options panel and click **Delete stack**.
2. A safety validation prompt frame modal interface layout window will pop up.
3. Confirm by clicking **Delete** to instantly clear all persistent configurations, custom application deployment manifests, and IAM binding traces safely from your global console profile index.
