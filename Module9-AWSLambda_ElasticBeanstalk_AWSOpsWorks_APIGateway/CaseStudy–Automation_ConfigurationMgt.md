# Module 9: Case Study - Combined Automation, Orchestration, and Serverless Event Architectures

## Problem Statement
XYZ Corporation hosts high-performance web applications within AWS and mandates a multi-faceted evaluation of cloud automation frameworks. To design a robust operational matrix, implement an AWS Elastic Beanstalk environment to automatically orchestrate a version-controlled PHP application, establish a serverless event-driven pipeline via AWS Lambda reacting to Amazon S3 object ingestion states, and provision an infrastructure-as-code configuration tier utilizing AWS OpsWorks managed Chef stacks.

---

## Tasks To Be Performed
1. **Elastic Beanstalk Layer:** Create an environment to deploy a baseline PHP web platform application.
2. **Application Revisions:** Package, upload, and deploy a modified PHP script reading `"Updated Hello World"` onto the running Beanstalk tier.
3. **Serverless Ingestion:** Create an AWS Lambda function that activates dynamically utilizing an Amazon S3 Bucket object creation event as its native operational invocation trigger.
4. **Configuration Management:** Provision an AWS OpsWorks Stack framework, launch managed instance layers, and serve a `"Hello World"` index HTML template via centralized configuration controls.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Deploy and Update the Elastic Beanstalk Environment (Task 1 & 2 Requirements)
1. **Prepare the App Package:** On your local machine, open a text editor, paste the script code block below, and save it as `index.php`:
   ```php
   <?php echo "<h1>XYZ Corporation - Updated Hello World</h1>"; ?>
   ```

2. Compress `index.php` directly into a zip archive named **`updated-hello-world.zip`**.
3. **Launch Beanstalk:** Log in to the **AWS Management Console** > open **Elastic Beanstalk** > click **Create application**.
* **Environment tier:** Web server environment
* **Application name:** `XYZ-Automation-App`
* **Platform:** Choose **PHP** (Keep default branch and version configurations active).
* **Application code:** Choose **Upload your code** > select your local compressed file packet asset: **`updated-hello-world.zip`**.


4. **Service Access Configuration:** Assign a functional EC2 instance profile containing `AWSElasticBeanstalkWebTier` access scopes, configure capacities to single instance cost-optimized tracking flags (`t2.micro` or `t3.micro`), and click **Submit**.
5. **Verification:** Wait 3-5 minutes for deployment loops to settle into a green **OK** status indication. Click the generated application URL to confirm it renders: *"XYZ Corporation - Updated Hello World"*.

---

### Step 2: Establish the S3-Triggered AWS Lambda Function (Task 3 Requirement)

1. **Create S3 Ingestion Target:** Open the **Amazon S3 Console** > click **Create bucket**. Name the bucket uniquely (e.g., `xyz-automation-trigger-bucket-2026`) and click create.
2. **Provision Lambda:** Open the **AWS Lambda Console** > click **Create function** > **Author from scratch**.
* **Function name:** `XYZ-S3-Event-Logger`
* **Runtime:** Select latest stable **Python** option (e.g., `Python 3.12`).
* **Permissions:** Choose *Create a new role with basic Lambda permissions*. Click **Create function**.


3. **Grant IAM S3 Read Rights:** Go to *Configuration* tab > *Permissions* > Click the execution role link to open IAM. Attach the managed storage tracking access policy: **`AmazonS3ReadOnlyAccess`**. Close the IAM window.
4. **Map the Event Trigger:** Return to the Lambda overview schematic canvas > Click **+ Add trigger**.
* Select **S3** from the dropdown source inventory array.
* **Bucket:** Select your target: `xyz-automation-trigger-bucket-2026`.
* **Event type:** Keep default **All object create events** (`s3:ObjectCreated:*`).
* Check the recursive safety acknowledgment box flag and click **Add**.


5. **Code Deployment:** In the *Code* tab pane workspace, overwrite the file with this tracking logic, then click **Deploy**:
```python
import json

def lambda_handler(event, context):
    # Log data attributes from the incoming S3 notification metadata object block
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']
    print(f"Serverless Trigger Active! Ingested File: {file_key} inside Bucket: {bucket_name}")
    return {'statusCode': 200, 'body': json.dumps('S3 Event Captured Perfectly.')}

```


6. **Test Invocations:** Go to your S3 console page, upload any dummy text file inside `xyz-automation-trigger-bucket-2026`. Return to Lambda > **Monitor** tab > view logs in **CloudWatch** to verify the function fired automatically and logged the string data block perfectly.

---

### Step 3: Launch OpsWorks Stack and Host Hello World Page (Task 4 Requirement)

1. Navigate to the **AWS OpsWorks Stacks Console** dashboard workspace.
2. Click **Add stack** > Select **Chef Infra Stack**.
* **Name:** `XYZ-Automation-OpsWorks-Stack`
* **VPC:** Select default VPC parameters.


3. Under **App Repository**, link a public deployment manifest repository path containing a basic `index.html` file setup parsing string text reading: `"Hello World from OpsWorks Chef Engine"`. Click **Save**.
4. Click **Add a layer** > Choose a baseline **Web App Server** layer profile template context.
5. Navigate to **Instances** within the layer row panel > Click **Add an instance** > Size: select `t2.micro` or `t3.micro` to maintain tracking cost controls > Click **Start** to boot the computing hardware.
6. Once the instance transitions to a green **Online** status sign, click on the **Apps** navigation directory link from the left structural menu, select your application manifest entry, and execute **Deploy App**.
7. **Verification:** Click the Public IP corresponding to your OpsWorks host node to verify it renders the uniform text payload framework perfectly.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Wipe this entire multi-tier resource cluster immediately upon grading to secure your credits from high enterprise maintenance consumption fees:

### 1. Dissolve the AWS OpsWorks Compute Fleet and Stack Layer

1. Open the **AWS OpsWorks Dashboard** > Navigate to **Instances** from the left panel sidebar checklists.
2. Locate the central operational engine tool controls and select **Stop All Instances**.
3. Wait 5 minutes until instances display a gray **Stopped** health flag.
4. Click the **Delete** (X) mark button next to each stopped host row element container to clear the configurations cleanly from the dashboard registry.
5. Click **Delete Stack** in the left panel list menu to clear the orchestration tracking container profile.

### 2. Terminate the Elastic Beanstalk Environment Container

1. Open the **AWS Elastic Beanstalk Console** > Click **Applications** > Select `XYZ-Automation-App`.
2. Click on your active target container link environment: `XyzAutomationApp-env`.
3. Locate the **Actions** dropdown menu options header cluster at the top right of the application management page.
4. Click **Terminate environment**. Type the explicit environment string name `XyzAutomationApp-env` to pass confirmation prompts and finalize hardware teardown.

### 3. Clear Storage Ingestion and Serverless Lambdas

1. Open the **Amazon S3 Console** > Click on your unique created bucket `xyz-automation-trigger-bucket-2026` > Click **Empty** > Type `permanently delete` to wipe contents > Click **Delete** to eliminate the storage endpoint fully.
2. Open the **AWS Lambda Console** > Select `XYZ-S3-Event-Logger` > Click **Actions** > **Delete** > Confirm to wipe the serverless microservice code asset completely.
