# Module 9: Assignment 2 - Deploying Serverless-Managed Web Applications via AWS Elastic Beanstalk

## Problem Statement
XYZ Corporation aims to launch a new web-based application utilizing a fully managed platform service to minimize operational overhead. The deployment team requires a system that handles provisioning, load balancing, auto-scaling, and application health monitoring automatically, ensuring compute nodes run efficiently without continuous manual administration. Implement an AWS Elastic Beanstalk environment utilizing a PHP platform runtime to host and serve application code packages.

---

## Tasks To Be Performed
1. Create an AWS Elastic Beanstalk application and environment with the runtime platform set to PHP.
2. Package and upload a simple PHP application source code file to the newly created environment to validate live web serving.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Prepare the Sample PHP Application Bundle
1. Open a text editor (e.g., Notepad or VS Code) on your local machine.
2. Paste the following baseline script to build a simple web page tracking file:
   ```php
   <?php
   echo "<h1>XYZ Corporation - Application Testing Environment</h1>";
   echo "<p>Elastic Beanstalk PHP Environment Provisioned and Running Successfully!</p>";
   echo "<p>Current Server Timestamp: " . date('Y-m-d H:i:s') . "</p>";
   ?>
   ```

3. Save the file locally as **`index.php`**.
4. **CRITICAL STEP:** Right-click on `index.php` and compress/zip it. Name the compressed file **`php-app-bundle.zip`**. *(Elastic Beanstalk requires code to be uploaded as a compressed archive format).*

---

### Step 2: Create Elastic Beanstalk Application and Environment

1. Log in to the **AWS Management Console** and search for **Elastic Beanstalk** to launch its service management console.
2. Click on the orange **Create application** (or **Create environment**) button.
3. Configure **Configure environment** parameters:
* **Environment tier:** Select **Web server environment**.
* **Application name:** `XYZ-Corporate-Web-App`
* **Environment name:** `XyzCorporateWebApp-env` (Automatically populates).


4. **Platform Configuration (Task 1 Requirement):**
* **Platform:** Select **PHP** from the dropdown menu list.
* **Platform branch:** Keep the latest recommended version (e.g., *PHP 8.2 running on 64bit Amazon Linux 2023*).
* **Platform version:** Keep default.


5. **Application Code Configuration (Task 2 Requirement):**
* Under *Application code*, select **Upload your code**.
* **Version label:** `v1.0-testing`
* Select **Local file** > Click **Choose file** > select the **`php-app-bundle.zip`** package created in Step 1.


6. Click **Next**.

---

### Step 3: Configure Service Access & Service Roles

1. **Service Role:** Choose **Create and use new service role** (Allows Beanstalk to monitor backend instances).
2. **EC2 key pair:** Select an existing key pair or leave blank if direct terminal patching isn't required.
3. **EC2 instance profile (CRITICAL):**
* If an instance profile dropdown is empty, open an isolated browser tab to the **IAM Console**, create a role for **EC2**, attach the managed policies `AWSElasticBeanstalkWebTier`, `AWSElasticBeanstalkWorkerTier`, and `AWSElasticBeanstalkMulticontainerDocker`, name it `aws-elasticbeanstalk-ec2-role`, and link it here.


4. Click **Next** through networking options (Keep Default VPC and public instance flags checked if required).
5. Under *Instance types*, ensure cost-effective tiers like **`t3.micro`** or **`t2.micro`** are selected to optimize resource limits.
6. Review the summary parameters page and click **Submit**.

---

### Step 4: Verify Live Application URL Deployment

1. Elastic Beanstalk will spend roughly 3-5 minutes setting up underlying networking interfaces, launching EC2 servers, and deploying your PHP web package.
2. Monitor logs until the **Health** state icon turns to a green **OK** check indicator mark.
3. Look at the top of the environment dashboard pane to find the generated application endpoint URL (e.g., `http://XyzCorporateWebApp-env.eba-xxxxxx.us-east-1.elasticbeanstalk.com`).
4. Click the URL link.
5. **Result:** Your browser will open the live web server rendering your custom text displaying: *"XYZ Corporation - Application Testing Environment. Elastic Beanstalk PHP Environment Provisioned and Running Successfully!"*.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Because Elastic Beanstalk provisions active EC2 virtual compute clusters and Elastic Load Balancers (ELB) in the background, keeping the environment active will quickly drain evaluation credits. Terminate all assets securely using this single workflow:

### 1. Terminate the Elastic Beanstalk Environment Container

1. Navigate to your main **AWS Elastic Beanstalk Dashboard** screen.
2. In the left panel directory structure layout, click on **Applications** and click on `XYZ-Corporate-Web-App`.
3. Click on the **Environments** list link, select `XyzCorporateWebApp-env`.
4. Locate the **Actions** dropdown menu utility component button positioned at the top right header space.
5. Click **Terminate environment**.
6. A safety warning modal prompt framework will pop up.
7. Type the exact configuration environment string name **`XyzCorporateWebApp-env`** inside the input verification box to confirm absolute teardown.
8. Click **Terminate**. The engine will safely drop the load balancers, terminate the EC2 host units, and delete autoscaling configurations cleanly from your account automatically in less than 5 minutes.

