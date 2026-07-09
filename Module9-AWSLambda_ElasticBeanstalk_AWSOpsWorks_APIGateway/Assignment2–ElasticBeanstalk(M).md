# Module 9 - AWS Lambda, Elastic Beanstalk, AWS OpsWorks and API Gateway

# Assignment 2 – Elastic Beanstalk

# Part 1: Cost Check, Architecture, Prerequisites, PHP Application Package, and Elastic Beanstalk Environment Creation

---

## Problem Statement

You work for XYZ Corporation. Your corporation wants to launch a new web-based application and they do not want their servers to be running all the time. It should also be managed by AWS. Implement suitable solutions.

---

## Tasks To Be Performed

1. Create an Elastic Beanstalk environment with the runtime as PHP.
2. Upload a simple PHP file to the environment once created.

---

## AWS Services Used

| AWS Service | Purpose in This Assignment |
| ----------- | -------------------------- |
| AWS Elastic Beanstalk | Managed application deployment service used to create and manage the PHP web application environment. |
| Amazon EC2 | Provides the virtual server on which the PHP application runs. Elastic Beanstalk provisions and manages this instance automatically. |
| Amazon EBS | Provides persistent block storage attached to the EC2 instance. |
| Amazon S3 | May be used internally by Elastic Beanstalk to store application source bundles and application versions. |
| AWS Identity and Access Management (IAM) | Provides the permissions required by Elastic Beanstalk and its underlying resources. |
| Amazon CloudWatch | Provides monitoring metrics and operational information for the environment and EC2 instance. |

---

## Free Tier / Cost Check

AWS Elastic Beanstalk itself has **no additional service charge**. However, the underlying AWS resources created and managed by Elastic Beanstalk can generate charges.

For this assignment, we will create a **Single instance** environment instead of a load-balanced high-availability environment. This is appropriate for learning and development and avoids creating an unnecessary Elastic Load Balancer.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| AWS Elastic Beanstalk | Yes — no additional Elastic Beanstalk service charge | No direct Elastic Beanstalk charge | You pay only for the underlying AWS resources used by the environment. |
| Amazon EC2 | Depends on account eligibility and selected instance type | Yes, if usage is billable | The Elastic Beanstalk environment creates an EC2 instance for the PHP application. |
| Amazon EBS | Depends on account eligibility and current AWS Free Tier benefits | Yes, if usage is billable | Storage attached to the EC2 instance may generate charges. |
| Amazon S3 | Depends on account eligibility and usage | Yes, if usage is billable | Elastic Beanstalk may store application versions and source bundles in S3. |
| Amazon CloudWatch | Basic monitoring is generally available without additional cost within applicable limits | Possibly | Additional metrics, logs, or usage beyond included limits can generate charges. |
| Elastic Load Balancing | Not used in this assignment | No | We deliberately select a `Single instance` environment to avoid creating a load balancer. |

### Cost Conclusion

This assignment can be inexpensive, but it should **not automatically be assumed to be completely free**.

The important pricing points are:

- AWS Elastic Beanstalk itself has no additional service charge.
- You pay for the underlying AWS resources that Elastic Beanstalk provisions and uses.
- The primary potentially billable resource for this assignment is the Amazon EC2 instance.
- The attached EBS volume can also contribute to billable usage.
- Newer AWS accounts may receive promotional AWS Free Tier credits that can be applied to eligible services.
- Account eligibility, account creation date, selected AWS Free Tier plan, Region, instance type, storage, and usage determine whether actual charges occur.
- It is safer to complete the assignment while promotional AWS credits are available if your account has them.
- Delete the Elastic Beanstalk environment immediately after successful verification because terminating the environment removes its running compute resources and minimizes continued credit consumption.

> **Important:** Do not create a load-balanced environment for this assignment. A single PHP file does not require a load balancer, multiple EC2 instances, Auto Scaling capacity beyond one instance, or a database.

---

## Architecture

The architecture for this assignment is intentionally simple:

~~~text
                         User's Web Browser
                                  |
                                  | HTTP Request
                                  v
                  +--------------------------------+
                  |     AWS Elastic Beanstalk      |
                  |                                |
                  |  Application: xyz-php-app      |
                  |  Environment: xyz-php-env      |
                  |  Platform: PHP                 |
                  +---------------+----------------+
                                  |
                                  | Provisions and manages
                                  v
                  +--------------------------------+
                  |        Amazon EC2 Instance     |
                  |                                |
                  |        PHP Runtime             |
                  |        index.php               |
                  +--------------------------------+
                                  |
                                  v
                  +--------------------------------+
                  |        Web Page Response       |
                  |                                |
                  |  Hello from AWS Elastic        |
                  |  Beanstalk!                    |
                  +--------------------------------+
~~~

---

## Dependency Flow

The complete dependency order is:

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Create Simple PHP File
    |
    v
Create ZIP Application Source Bundle
    |
    v
Open AWS Elastic Beanstalk
    |
    v
Create Elastic Beanstalk Application
    |
    v
Create Web Server Environment
    |
    v
Select PHP Managed Platform
    |
    v
Select Single Instance Preset
    |
    v
Upload PHP ZIP Source Bundle
    |
    v
Elastic Beanstalk Provisions EC2 and Supporting Resources
    |
    v
Wait for Environment Health = OK
    |
    v
Open Environment Domain URL
    |
    v
Verify PHP Web Page
    |
    v
Delete Elastic Beanstalk Environment
    |
    v
Delete Elastic Beanstalk Application
    |
    v
Verify Cleanup
~~~

---

## Prerequisites

Before starting, ensure that you have:

| Requirement | Required Value |
| ----------- | -------------- |
| AWS Account | Active AWS account |
| AWS Console Access | Ability to sign in to the AWS Management Console |
| IAM Permissions | Permission to create and manage Elastic Beanstalk, EC2, IAM service roles, S3 objects, EBS resources, and related resources |
| Local Computer | Windows, Linux, or macOS |
| Text Editor | Notepad, Visual Studio Code, or another plain-text editor |
| Web Browser | Current browser capable of accessing the AWS Management Console |
| AWS Region | One Region selected consistently throughout the assignment |

---

## Resource Naming Convention

Use the following names throughout the assignment:

| Resource | Value |
| -------- | ----- |
| Elastic Beanstalk Application | `xyz-php-app` |
| Elastic Beanstalk Environment | `xyz-php-env` |
| PHP File | `index.php` |
| Application Source Bundle | `php-app.zip` |
| Environment Type | `Web server environment` |
| Platform | `PHP` |
| Environment Preset | `Single instance` |

Using consistent names makes the environment easier to identify, verify, document, and delete.

---

# Implementation

## Step 1: Sign In to the AWS Management Console

### Navigation

~~~text
AWS Management Console
→ Sign in
→ Open the AWS Console Home page
~~~

### Action

1. Sign in to your AWS account.
2. Confirm that the AWS Console Home page opens successfully.
3. Check the currently selected AWS Region in the upper-right corner of the console.

### Why is this step required?

All resources for this assignment will be created through the AWS Management Console. You must be authenticated before you can create an Elastic Beanstalk application or any underlying AWS resources.

### Dependency

This step has no AWS resource dependency. It requires only a valid AWS account and sufficient IAM permissions.

### What happens if this step is skipped?

You cannot access the Elastic Beanstalk console or create the environment.

---

## Step 2: Select the AWS Region

### Navigation

~~~text
AWS Management Console
→ Upper-right Region selector
→ Select your preferred AWS Region
~~~

### Recommended Configuration

If you have been using a particular Region for your AWS course assignments, continue using that same Region.

For example:

| Setting | Value |
| ------- | ----- |
| Region | `Asia Pacific (Mumbai) ap-south-1` |

### Important Region Note

Elastic Beanstalk resources are regional. The application, environment, EC2 instance, and associated supporting resources are created in the selected Region.

Do not change the Region during the assignment.

### Why is this step required?

Selecting the Region first ensures that all resources are created in the intended geographic location.

### Dependency

Depends on successful AWS Console sign-in from Step 1.

### What happens if this step is skipped?

The environment may be created in whichever Region is currently selected. This can make resources difficult to locate later and may complicate verification and cleanup.

---

## Step 3: Create the Simple PHP Application File

The assignment requires a simple PHP file. We will create an `index.php` file that displays a successful deployment message and basic PHP runtime information.

### Navigation

On Windows:

~~~text
Desktop or preferred working folder
→ Right-click
→ New
→ Text Document
→ Open the file in Notepad or Visual Studio Code
~~~

### File Name

Save the file exactly as:

`index.php`

> **Important for Windows users:** Make sure the actual filename is `index.php`, not `index.php.txt`.

If file extensions are hidden, enable them from:

~~~text
File Explorer
→ View
→ Show
→ File name extensions
~~~

### PHP Code

Add the following content to `index.php`:

~~~php
<?php
$appName = "XYZ Corporation PHP Application";
$message = "Hello from AWS Elastic Beanstalk!";
$runtime = PHP_VERSION;
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?php echo $appName; ?></title>
</head>
<body>
    <h1><?php echo $message; ?></h1>
    <p>Application: <?php echo $appName; ?></p>
    <p>PHP Runtime Version: <?php echo $runtime; ?></p>
    <p>Status: Application deployed successfully.</p>
</body>
</html>
~~~

### PHP Code Explanation

| Code Part | Explanation |
| --------- | ----------- |
| `<?php` | Opens a PHP code block. |
| `$appName` | Creates a PHP variable containing the application name. |
| `=` | Assignment operator used to assign a value to a variable. |
| `"XYZ Corporation PHP Application"` | String value stored in the `$appName` variable. |
| `;` | Terminates a PHP statement. |
| `$message` | Variable containing the message displayed on the web page. |
| `PHP_VERSION` | Built-in PHP constant that returns the PHP runtime version being used by the server. |
| `?>` | Closes the PHP code block. |
| `<!DOCTYPE html>` | Declares that the document uses HTML5. |
| `<html lang="en">` | Starts the HTML document and declares English as its language. |
| `<meta charset="UTF-8">` | Configures UTF-8 character encoding. |
| `<meta name="viewport" ...>` | Helps the page render correctly on different screen sizes. |
| `<title>` | Defines the browser tab title. |
| `<?php echo $appName; ?>` | Executes PHP and prints the value of `$appName` into the HTML output. |
| `<body>` | Contains the visible page content. |
| `<h1>` | Displays the main heading. |
| `<p>` | Displays a paragraph. |

### Why is this step required?

Elastic Beanstalk requires application source code to deploy. The `index.php` file is the web application's entry page and demonstrates that the PHP runtime is successfully processing server-side PHP code.

The use of `PHP_VERSION` is especially useful because it confirms that the page is not merely static HTML. The PHP runtime must execute the code to obtain and display the runtime version.

### Dependency

Depends only on having a local computer and text editor.

### What happens if this step is skipped?

There will be no custom PHP application source code to upload and deploy.

---

## Step 4: Create the ZIP Application Source Bundle

Elastic Beanstalk accepts an application source bundle. For this assignment, create a ZIP archive containing `index.php`.

### Required ZIP Structure

The ZIP archive must contain the PHP file directly at its root:

~~~text
php-app.zip
|
+-- index.php
~~~

Do **not** create this incorrect structure:

~~~text
php-app.zip
|
+-- php-app
    |
    +-- index.php
~~~

The source file should be directly inside the ZIP archive rather than unnecessarily nested inside another parent folder.

### Windows Navigation

~~~text
File Explorer
→ Open the folder containing index.php
→ Right-click index.php
→ Compress to ZIP file
~~~

Depending on the Windows version, the option may instead appear as:

~~~text
Right-click index.php
→ Send to
→ Compressed (zipped) folder
~~~

Rename the resulting archive to:

`php-app.zip`

### Configuration

| Setting | Value |
| ------- | ----- |
| Source File | `index.php` |
| Archive Format | `ZIP` |
| Archive Name | `php-app.zip` |
| Required Root File | `index.php` |

### Why is this step required?

Elastic Beanstalk uses the uploaded source bundle as an application version. The platform deploys the bundle onto the underlying environment resources.

### Dependency

Depends on the `index.php` file created in Step 3.

### What happens if this step is skipped?

You cannot upload the intended PHP application bundle during environment creation.

---

## Step 5: Open AWS Elastic Beanstalk

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "Elastic Beanstalk"
→ Select Elastic Beanstalk
~~~

### Action

Open the Elastic Beanstalk console.

Depending on your account history and the current AWS Console interface, you may see:

- A welcome page with a **Create application** button.
- An applications page.
- An environments page.
- A **Create environment** option.

AWS occasionally changes console labels and page layout slightly, but the required objective remains the same: create an Elastic Beanstalk application containing a web server environment.

### Why is this step required?

AWS Elastic Beanstalk is the managed deployment service responsible for provisioning and managing the PHP application environment.

### Dependency

Depends on Steps 1 and 2.

### What happens if this step is skipped?

The managed PHP environment cannot be created.

---

## Step 6: Start Creating the Elastic Beanstalk Application and Environment

### Navigation

A typical current workflow is:

~~~text
AWS Console
→ Elastic Beanstalk
→ Applications
→ Create application
~~~

Depending on the console page shown in your account, you may instead start from:

~~~text
AWS Console
→ Elastic Beanstalk
→ Create environment
~~~

If prompted to select an environment tier, choose:

`Web server environment`

### Application Configuration

| Setting | Value |
| ------- | ----- |
| Application name | `xyz-php-app` |
| Environment tier | `Web server environment` |
| Environment name | `xyz-php-env` |

### Application Name

Enter:

`xyz-php-app`

### Environment Name

Enter:

`xyz-php-env`

Elastic Beanstalk may automatically suggest an environment domain based on the environment name.

The final URL will typically follow a format similar to:

~~~text
http://xyz-php-env.<generated-value>.<region>.elasticbeanstalk.com
~~~

The exact generated URL varies by environment, Region, and availability.

### Why is this step required?

Elastic Beanstalk organizes deployments using two important concepts:

- An **application** is a logical container for application versions and environments.
- An **environment** is the actual running deployment containing the infrastructure that serves the application.

### Dependency

Depends on opening the Elastic Beanstalk console in Step 5.

### What happens if this step is skipped?

There will be no logical application container or running environment in which to deploy the PHP application.

---

## Step 7: Select the PHP Managed Platform

### Navigation

Within the Elastic Beanstalk environment creation workflow:

~~~text
Elastic Beanstalk
→ Create environment
→ Platform
→ Managed platform
→ Platform
→ PHP
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Platform type | `Managed platform` |
| Platform | `PHP` |
| Platform branch | Select the latest recommended supported PHP branch available in the console |
| Platform version | Select the latest recommended version offered by Elastic Beanstalk |

### Important Platform Selection Rule

Do not manually select an obsolete PHP version merely to match an older screenshot or course recording.

Choose the current AWS-recommended PHP platform branch available in your console unless your assignment explicitly requires a specific PHP version.

### Why is this step required?

The PHP managed platform provides the operating system, web server integration, PHP runtime, deployment tooling, monitoring integration, and Elastic Beanstalk management components required to run the application.

### Dependency

Depends on starting the environment creation workflow in Step 6.

### What happens if this step is skipped?

Elastic Beanstalk will not know which runtime stack should execute the PHP application. Selecting a different platform such as Python, Node.js, Java, or .NET would not satisfy the assignment requirement.

---

## Step 8: Select the Application Code

Because the PHP application bundle was already prepared locally, select the option that allows you to upload your own application code.

### Navigation

Within the environment creation workflow:

~~~text
Elastic Beanstalk
→ Application code
→ Upload your code
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Application code | `Upload your code` |
| Version label | `php-app-v1` |
| Source | `Local file` |
| File | `php-app.zip` |

### Action

1. Select **Upload your code**.
2. Enter the version label `php-app-v1` if a version label is requested.
3. Select **Local file**.
4. Choose the `php-app.zip` archive created earlier.
5. Confirm that the file has been selected successfully.

### Why is this step required?

This step provides Elastic Beanstalk with the PHP source bundle that must be deployed to the environment.

The uploaded bundle becomes an application version associated with the Elastic Beanstalk application.

### Dependency

Depends on:

- Step 3: `index.php` created.
- Step 4: `php-app.zip` created.
- Step 6: Elastic Beanstalk application and environment configuration started.
- Step 7: PHP platform selected.

### What happens if this step is skipped?

The custom PHP application will not be deployed. Elastic Beanstalk may deploy only a sample application if that option is selected instead.

---

## Step 9: Select the Single Instance Configuration Preset

### Navigation

Within the environment creation workflow:

~~~text
Elastic Beanstalk
→ Configuration preset
→ Single instance
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Environment tier | `Web server environment` |
| Preset | `Single instance` |
| Intended use | Development, learning, and assignment verification |

### Why `Single instance` is the correct choice

A single-instance environment is sufficient because this assignment requires only:

- One simple PHP file.
- One PHP runtime environment.
- Basic web application verification.

Using a high-availability configuration would unnecessarily create additional infrastructure such as a load balancer and potentially additional compute capacity.

### Why is this step required?

The configuration preset determines the basic environment architecture.

The `Single instance` preset provides the simplest architecture suitable for development and learning while minimizing unnecessary infrastructure.

### Dependency

Depends on the Elastic Beanstalk environment configuration already being in progress.

### What happens if this step is skipped?

If a high-availability preset is selected instead, Elastic Beanstalk may create an Elastic Load Balancer and additional infrastructure that is unnecessary for this assignment and may consume additional AWS credits or generate charges.

---

## Step 10: Configure Service Access

Elastic Beanstalk may require IAM roles so that the service and EC2 instance can interact with required AWS services.

### Navigation

Within the environment creation wizard:

~~~text
Elastic Beanstalk
→ Configure service access
~~~

### Possible Configuration Fields

Depending on the current AWS Console workflow and whether your account already has Elastic Beanstalk roles, you may see fields such as:

| Setting | Recommended Action |
| ------- | ------------------ |
| Service role | Select an existing suitable Elastic Beanstalk service role or allow the wizard to create the required role when offered |
| EC2 instance profile | Select an existing suitable Elastic Beanstalk EC2 instance profile or create one through the supported console workflow |
| EC2 key pair | Leave unselected unless SSH access is specifically required |

### Important Note About IAM Roles

The exact IAM role options shown can differ depending on:

- Whether this AWS account has previously used Elastic Beanstalk.
- Whether suitable service roles already exist.
- Whether an EC2 instance profile already exists.
- The IAM permissions granted to the currently signed-in user.
- Updates to the AWS Console interface.

Do not select unrelated IAM roles simply because they appear in the dropdown.

The selected role or instance profile must be appropriate for Elastic Beanstalk.

### EC2 Key Pair

For this assignment, direct SSH access to the EC2 instance is not required.

Therefore, if the console permits it, you may leave the EC2 key pair unselected.

### Why is this step required?

Elastic Beanstalk uses IAM permissions to perform management and monitoring operations. The underlying EC2 instance may also require an instance profile for interaction with AWS services.

### Dependency

Depends on the environment creation workflow started in previous steps.

### What happens if this step is skipped?

If required service access is not configured, environment creation may fail because Elastic Beanstalk or its underlying EC2 instance lacks the permissions necessary to perform required operations.

---

## Step 11: Continue Through the Environment Configuration Wizard

After configuring service access, continue through the remaining configuration pages.

The exact page names can vary slightly with AWS Console updates, but commonly include:

~~~text
Step 1: Configure environment
        |
        v
Step 2: Configure service access
        |
        v
Step 3: Set up networking, database, and tags
        |
        v
Step 4: Configure instance traffic and scaling
        |
        v
Step 5: Configure updates, monitoring, and logging
        |
        v
Step 6: Review
~~~

### Networking Configuration

For this simple assignment, use the default networking configuration unless your course specifically requires a custom VPC.

Do not create:

- A custom VPC.
- A database.
- Additional subnets.
- A NAT Gateway.
- A custom load balancer.

These resources are unnecessary for the stated assignment.

### Database Configuration

Do not add a database.

The PHP application only displays a simple webpage and does not require persistent application data.

### Tags

Tags are optional unless required by your organization or course.

If you want to identify the resources clearly, you may use:

| Key | Value |
| --- | ----- |
| `Project` | `Module9-Assignment2` |

### Instance and Scaling Configuration

Maintain the single-instance configuration.

If the console allows instance-type selection, choose an instance type that is eligible under your specific AWS account's current Free Tier benefits or promotional credits and supported by the selected Elastic Beanstalk PHP platform.

Do not assume a specific instance type is free without checking your account eligibility.

### Monitoring and Logging

For this simple assignment:

- Keep default basic monitoring unless additional monitoring is specifically required.
- Do not enable unnecessary enhanced health features, log streaming, or additional monitoring options solely for this assignment if they could create avoidable usage.

### Why is this step required?

Elastic Beanstalk needs complete environment configuration before it can provision the underlying infrastructure.

### Dependency

Depends on Steps 6 through 10.

### What happens if this step is skipped?

You cannot reach the final review page or launch the Elastic Beanstalk environment.

---

## Step 12: Review the Complete Environment Configuration

### Navigation

~~~text
Elastic Beanstalk
→ Create environment wizard
→ Review
~~~

### Verify the Important Values

Before creating the environment, confirm:

| Setting | Expected Value |
| ------- | -------------- |
| Application name | `xyz-php-app` |
| Environment name | `xyz-php-env` |
| Environment tier | `Web server environment` |
| Platform | `PHP` |
| Application code | Uploaded `php-app.zip` |
| Version label | `php-app-v1` if configured |
| Configuration preset | `Single instance` |
| Database | None |
| Load balancer | None for the single-instance environment |
| EC2 key pair | Not required for this assignment |

### Important Cost Verification

Before selecting the final create or submit button, confirm that the environment is configured as:

`Single instance`

Do not proceed with a load-balanced high-availability environment for this assignment.

### Why is this step required?

The review page provides the final opportunity to detect incorrect platform, application source, environment type, networking, IAM, or scaling configuration before AWS provisions resources.

### Dependency

Depends on completing all required previous configuration pages.

### What happens if this step is skipped?

Incorrect configuration may result in failed deployment, unnecessary infrastructure, additional AWS credit usage, or an environment that does not satisfy the assignment requirements.

---

## Step 13: Create the Elastic Beanstalk Environment

### Navigation

From the final review page:

~~~text
Elastic Beanstalk
→ Review
→ Submit or Create environment
~~~

The exact final button label may vary slightly depending on the current console workflow.

### Action

Select the final environment creation button.

Elastic Beanstalk now begins provisioning the environment.

### Provisioning Flow

~~~text
Submit Environment Configuration
            |
            v
Create Elastic Beanstalk Environment
            |
            v
Provision EC2 Instance
            |
            v
Attach Storage
            |
            v
Configure PHP Platform
            |
            v
Deploy php-app.zip
            |
            v
Start Web Application
            |
            v
Perform Health Checks
            |
            v
Environment Becomes Ready
~~~

### Expected Initial Status

During provisioning, you may observe messages indicating that the environment is:

~~~text
Launching
Creating
Updating
Pending
~~~

These are normal while AWS creates and configures the resources.

Do not begin verification until environment provisioning has completed.

### Expected Successful State

After successful deployment, the environment should eventually report a healthy state such as:

~~~text
Health: Ok
Status: Ready
~~~

The exact capitalization or presentation can vary slightly in the console.

### Why is this step required?

This action instructs Elastic Beanstalk to provision the underlying AWS infrastructure, install and configure the selected PHP platform, deploy the uploaded source bundle, and expose the application through an environment URL.

### Dependency

Depends on successful completion and review of all previous configuration steps.

### What happens if this step is skipped?

No Elastic Beanstalk environment will be provisioned, no EC2 instance will host the PHP application, and there will be no web URL to verify.

---

## Important Note Before Continuing

Do not delete or terminate any resource yet.

The next part will continue with:

- Monitoring Elastic Beanstalk environment creation.
- Understanding the environment dashboard.
- Verifying environment health.
- Opening the generated application domain.
- Confirming that `index.php` is being executed by the PHP runtime.
- Troubleshooting common deployment problems.
- Performing complete cleanup in reverse dependency order.
- Verifying that all created resources have been removed.

---

# Part 2: Environment Verification, PHP Application Testing, Troubleshooting, and Complete Cleanup

---

# Verification

## Verification Step 1: Monitor the Elastic Beanstalk Environment Creation

### Action

After selecting the final **Submit** or **Create environment** button in Part 1, remain on the Elastic Beanstalk environment page and monitor the deployment process.

Elastic Beanstalk automatically performs several operations in the background:

~~~text
Elastic Beanstalk Environment Creation Started
                    |
                    v
        Validate Environment Configuration
                    |
                    v
           Create Supporting Resources
                    |
                    v
            Launch EC2 Instance
                    |
                    v
          Configure PHP Platform
                    |
                    v
          Deploy php-app.zip Bundle
                    |
                    v
              Start Web Server
                    |
                    v
           Perform Environment Checks
                    |
                    v
            Environment Becomes Ready
~~~

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
~~~

### Expected Environment Information

During creation, the environment may temporarily display states such as:

~~~text
Status: Launching
Health: Pending
~~~

or:

~~~text
Status: Updating
Health: Pending
~~~

After successful provisioning, the expected final state is:

~~~text
Status: Ready
Health: Ok
~~~

Depending on the current Elastic Beanstalk console interface, the health status may be displayed using different capitalization or visual indicators.

### What should I observe?

Confirm that:

- The environment `xyz-php-env` exists.
- The environment status eventually becomes `Ready`.
- The health status eventually becomes `Ok`.
- No unresolved severe deployment errors are displayed.
- A domain URL is assigned to the environment.

### Why does this confirm success?

A `Ready` environment status indicates that Elastic Beanstalk has completed the current environment operation.

An `Ok` health status indicates that the environment is responding successfully according to Elastic Beanstalk health monitoring.

Together, these states indicate that the environment infrastructure was successfully created and the application deployment completed without a critical health failure.

---

## Verification Step 2: Review the Elastic Beanstalk Environment Dashboard

### Action

Open the environment overview page and inspect the environment details.

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
~~~

### Expected Information

You should observe information similar to:

| Property | Expected Value |
| -------- | -------------- |
| Application | `xyz-php-app` |
| Environment | `xyz-php-env` |
| Platform | `PHP` |
| Environment tier | `Web server` |
| Status | `Ready` |
| Health | `Ok` |
| Application version | `php-app-v1` or the version label created during deployment |
| Domain | Automatically generated Elastic Beanstalk environment URL |

### What should I observe?

Verify that:

1. The application name is correct.
2. The environment name is correct.
3. The selected platform is PHP.
4. The environment is ready.
5. The health status is healthy.
6. The deployed application version corresponds to the uploaded PHP source bundle.
7. The environment has an accessible domain URL.

### Why does this confirm success?

These details prove that the required Elastic Beanstalk application and PHP environment were created with the intended configuration.

---

## Verification Step 3: Review Environment Events

### Action

Open the environment events section and inspect the latest deployment events.

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
→ Events
~~~

Depending on the current console layout, recent events may also appear directly on the environment overview page.

### Expected Output

Successful events may include messages conceptually similar to:

~~~text
Environment creation completed successfully.

Successfully launched environment: xyz-php-env

Application available at the environment URL.
~~~

The exact wording and number of events may vary according to:

- PHP platform version.
- AWS Region.
- Elastic Beanstalk console version.
- Environment configuration.
- IAM configuration.
- Underlying resources created.

### What should I observe?

Look for successful environment creation and application deployment events.

There should be no unresolved event with a severe error indicating that:

- The EC2 instance failed to launch.
- The application source bundle could not be deployed.
- The environment creation failed.
- Required IAM permissions were unavailable.
- The PHP platform failed to initialize.

### Why does this confirm success?

Elastic Beanstalk events provide a chronological record of important environment operations. Successful creation and deployment events confirm that the managed deployment workflow completed.

---

## Verification Step 4: Open the Elastic Beanstalk Environment URL

### Action

From the environment overview page, select the generated environment domain or use the available link to open the running application.

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
→ Domain
→ Open the environment URL
~~~

The URL will have a generated format similar to:

~~~text
http://xyz-php-env.<generated-value>.<region>.elasticbeanstalk.com
~~~

The exact domain is generated by AWS and will differ between environments.

### Expected Output

The browser should display content similar to:

~~~text
Hello from AWS Elastic Beanstalk!

Application: XYZ Corporation PHP Application

PHP Runtime Version: <deployed PHP version>

Status: Application deployed successfully.
~~~

For example, the runtime line could appear as:

~~~text
PHP Runtime Version: 8.x.x
~~~

The exact version depends on the PHP platform branch and platform version selected during environment creation.

### What should I observe?

Confirm all of the following:

- The environment URL opens successfully.
- The browser displays the custom heading `Hello from AWS Elastic Beanstalk!`.
- The application name appears correctly.
- A PHP runtime version is dynamically displayed.
- The deployment success message appears.
- You do not see the default Elastic Beanstalk sample application instead of your custom page.

### Why does this confirm success?

The displayed page proves that:

1. The Elastic Beanstalk environment is reachable.
2. The underlying web server is responding.
3. The uploaded `index.php` file was successfully deployed.
4. The PHP runtime executed the PHP code.
5. The assignment's second task has been successfully implemented.

The dynamically generated PHP version is especially important because it proves that server-side PHP processing is working rather than merely serving a static HTML document.

---

## Verification Step 5: Verify the Underlying EC2 Instance

Elastic Beanstalk is a managed service, but the single-instance web environment uses an underlying Amazon EC2 instance.

### Action

Open the EC2 console and inspect the running instance created for the Elastic Beanstalk environment.

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Instances
~~~

### Expected Observation

You should find an EC2 instance associated with the Elastic Beanstalk environment.

The exact instance name and tags are automatically managed by Elastic Beanstalk and may vary.

Expected state:

~~~text
Instance state: Running
Status checks: 2/2 checks passed
~~~

The exact status-check presentation may vary slightly in the current EC2 console.

### What should I observe?

Confirm that:

- An EC2 instance exists for the Elastic Beanstalk environment.
- Its instance state is `Running`.
- The instance is associated with resources or tags related to the Elastic Beanstalk environment.
- The instance status checks eventually pass.

### Why does this confirm success?

Elastic Beanstalk automatically provisions and manages the compute infrastructure required by the application.

Seeing the running EC2 instance confirms that the managed environment has underlying compute capacity serving the PHP application.

> **Important:** Do not manually terminate this EC2 instance from the EC2 console during normal Elastic Beanstalk environment operation. The environment should be managed and terminated through the Elastic Beanstalk console.

---

## Verification Step 6: Verify the Deployed Application Version

### Action

Inspect the Elastic Beanstalk application versions associated with `xyz-php-app`.

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Application versions
~~~

Depending on the current AWS Console layout, the application versions section may be available from the application navigation menu.

### Expected Information

You should see the application version uploaded during environment creation.

For example:

| Property | Expected Value |
| -------- | -------------- |
| Application | `xyz-php-app` |
| Version label | `php-app-v1` |
| Source bundle | Uploaded PHP ZIP application bundle |

### What should I observe?

Verify that the uploaded PHP application version exists and corresponds to the version currently deployed to `xyz-php-env`.

### Why does this confirm success?

Elastic Beanstalk deploys source bundles as application versions. Seeing the expected application version confirms that the uploaded `php-app.zip` bundle was successfully registered with the application.

---

# Troubleshooting

## Troubleshooting Issue 1: Environment Health Does Not Become `Ok`

### Possible Symptoms

~~~text
Health: Severe
~~~

or:

~~~text
Health: Degraded
~~~

or the environment remains in a pending state for an unusually long period.

### Possible Causes

- EC2 instance launch failure.
- Unsupported instance type.
- Insufficient IAM permissions.
- Incorrect service role.
- Missing or incorrect EC2 instance profile.
- Application deployment failure.
- PHP platform initialization failure.
- AWS account restrictions.
- Insufficient service quotas.

### Investigation Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
→ Events
~~~

### Resolution

Read the most recent error events from newest to oldest and identify the first underlying failure.

Do not repeatedly recreate environments without first identifying the root cause because this can create unnecessary AWS resources and consume additional credits.

---

## Troubleshooting Issue 2: Default Elastic Beanstalk Page Appears Instead of the Custom PHP Page

### Possible Cause

The environment may have been created using the sample application rather than the uploaded `php-app.zip` bundle.

### Resolution

Verify the deployed application version.

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
~~~

If the custom application version was not deployed, upload and deploy the correct `php-app.zip` source bundle.

The archive should have this structure:

~~~text
php-app.zip
|
+-- index.php
~~~

Not:

~~~text
php-app.zip
|
+-- php-app
    |
    +-- index.php
~~~

### Why does this matter?

Elastic Beanstalk extracts the application source bundle during deployment. An unnecessary parent directory can prevent the application entry file from being located where the platform expects it.

---

## Troubleshooting Issue 3: The Browser Displays an Error Page

### Possible Errors

You may encounter:

~~~text
502 Bad Gateway
~~~

or:

~~~text
503 Service Unavailable
~~~

or another application error.

### Possible Causes

- PHP syntax error.
- Failed deployment.
- Web server initialization failure.
- Incorrect application bundle.
- Environment still being updated.
- Unhealthy EC2 instance.

### Resolution

First inspect:

~~~text
AWS Management Console
→ Elastic Beanstalk
→ xyz-php-env
→ Events
~~~

Then check environment health and wait until the environment reaches:

~~~text
Status: Ready
Health: Ok
~~~

If necessary, review the PHP source code and ensure the application bundle contains a valid `index.php` file.

---

## Troubleshooting Issue 4: `index.php` Is Actually Named `index.php.txt`

### Cause

Windows may hide known file extensions. A file displayed as `index.php` can actually be named:

`index.php.txt`

### Resolution

Enable file extensions:

~~~text
File Explorer
→ View
→ Show
→ File name extensions
~~~

Then rename the file exactly to:

`index.php`

Recreate the `php-app.zip` archive and deploy the corrected application version.

### Why does this matter?

A file named `index.php.txt` is treated as a text file rather than the intended PHP entry file.

---

## Troubleshooting Issue 5: Environment Creation Fails Due to IAM Role or Instance Profile

### Possible Cause

Elastic Beanstalk does not have the required service access configuration.

### Investigation Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ xyz-php-env
→ Events
~~~

Look for messages mentioning:

- Service role.
- Instance profile.
- IAM permissions.
- Access denied.
- Unauthorized operation.

### Resolution

Use an appropriate Elastic Beanstalk service role and EC2 instance profile.

Do not attach unrelated roles or excessive administrator permissions merely to bypass the problem.

### Why does this matter?

Elastic Beanstalk and its underlying EC2 instances require appropriate IAM permissions to perform management and operational tasks.

---

## Troubleshooting Issue 6: Environment URL Does Not Open

### Possible Causes

- Environment creation is not complete.
- Environment health is not `Ok`.
- EC2 instance has not passed its health checks.
- Application deployment failed.
- Browser cached an old error response.
- Temporary DNS propagation or network issue.

### Resolution

Verify:

~~~text
Status: Ready
Health: Ok
~~~

Then:

1. Refresh the page.
2. Try opening the environment URL in a private or incognito browser window.
3. Review the Elastic Beanstalk environment events.
4. Confirm that the custom application version is deployed.
5. Confirm that the underlying EC2 instance is running and healthy.

---

# Cleanup / Cost Optimization

After successfully verifying the PHP application, delete all resources created for the assignment.

The correct cleanup flow is:

~~~text
Verify PHP Application Successfully
                |
                v
Terminate Elastic Beanstalk Environment
                |
                v
Wait for Environment Termination
                |
                v
Verify Underlying EC2 Instance Is Terminated
                |
                v
Delete Unneeded Application Versions
                |
                v
Delete Elastic Beanstalk Application
                |
                v
Verify Supporting Resources
                |
                v
Verify Final Cleanup
~~~

The reverse dependency order is important because the environment depends on the Elastic Beanstalk application configuration and underlying resources.

---

## Cleanup Step 1: Terminate the Elastic Beanstalk Environment

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
→ xyz-php-env
→ Actions
→ Terminate environment
~~~

Depending on the current console layout, the termination option may be available from the environment's **Actions** menu.

### Action

1. Select `xyz-php-env`.
2. Open the **Actions** menu.
3. Select **Terminate environment**.
4. Enter the environment name if AWS requests confirmation.
5. Confirm termination.

### Why is this required?

The environment contains the running infrastructure that serves the PHP application.

Terminating the environment instructs Elastic Beanstalk to remove the managed environment and its associated resources.

### Dependency

Perform this step only after all application verification is complete.

The environment must be terminated before deleting the Elastic Beanstalk application.

### What happens if this resource is not deleted?

The underlying EC2 instance and associated resources may continue running and can:

- Generate AWS charges.
- Consume promotional AWS credits.
- Leave unnecessary infrastructure active.

---

## Cleanup Step 2: Wait for Complete Environment Termination

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Environments
~~~

### Action

Wait until `xyz-php-env` is completely terminated.

During termination, the environment may temporarily display a status such as:

~~~text
Terminating
~~~

Do not attempt to delete the application while the environment is still active or terminating.

### Expected Result

The environment should eventually disappear from the active environments list or show that termination has completed.

### Why is this required?

The Elastic Beanstalk application can have dependencies on its active environment. Waiting for complete termination prevents dependency conflicts during later cleanup steps.

### Dependency

Depends on Cleanup Step 1.

### What happens if this step is skipped?

Attempting to delete the application before its environment is fully terminated may fail because active or terminating environment dependencies still exist.

---

## Cleanup Step 3: Verify That the Underlying EC2 Instance Is Terminated

### Navigation

~~~text
AWS Management Console
→ EC2
→ Instances
→ Instances
~~~

### Action

Locate the EC2 instance previously associated with `xyz-php-env`.

Verify that it is no longer running.

### Expected Final State

The instance should either:

- Show the state `Terminated`, or
- Eventually disappear from the active instance view after AWS removes terminated-instance history from the console display.

### Why is this required?

The EC2 instance is one of the primary resources that may consume AWS credits or generate charges while running.

This verification confirms that terminating the Elastic Beanstalk environment successfully removed its compute capacity.

### Dependency

Depends on complete termination of `xyz-php-env`.

### What happens if this resource remains running?

A running EC2 instance can continue consuming promotional credits or generating charges.

Do not manually terminate an Elastic Beanstalk-managed instance while its environment is still active. Manage the environment through Elastic Beanstalk first.

---

## Cleanup Step 4: Delete Unneeded Elastic Beanstalk Application Versions

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Application versions
~~~

### Action

If `php-app-v1` or another application version remains:

1. Select the application version.
2. Choose the delete action.
3. If AWS provides an option to delete the corresponding source bundle from Amazon S3, select that option when the source bundle is no longer needed.
4. Confirm deletion.

### Why is this required?

Elastic Beanstalk application source bundles may be stored in Amazon S3.

Deleting unused application versions and their associated source bundles helps remove unnecessary storage and keeps the account clean.

### Dependency

The application version should not be actively deployed to a running environment.

Therefore, perform this step only after `xyz-php-env` has been completely terminated.

### What happens if this resource is not deleted?

Unused application versions and associated S3 source bundles may remain in the account and can contribute to storage usage.

---

## Cleanup Step 5: Delete the Elastic Beanstalk Application

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
→ xyz-php-app
→ Actions
→ Delete application
~~~

### Action

1. Select `xyz-php-app`.
2. Choose **Actions**.
3. Select **Delete application**.
4. Confirm deletion when prompted.

### Why is this required?

The application is the top-level logical Elastic Beanstalk container created specifically for this assignment.

After the environment and unnecessary application versions are removed, the application itself is no longer required.

### Dependency

Before deleting the application:

- `xyz-php-env` must be terminated.
- Unneeded application versions should be removed if they are not automatically deleted with the application.

### What happens if this resource is not deleted?

The logical Elastic Beanstalk application configuration may remain in the account even though the environment is no longer running.

While the application container itself does not represent running compute capacity, deleting it keeps the account clean and prevents confusion in future assignments.

---

## Cleanup Step 6: Verify Elastic Beanstalk Resources Are Removed

### Navigation

~~~text
AWS Management Console
→ Elastic Beanstalk
→ Applications
~~~

### Action

Verify that:

- `xyz-php-app` no longer appears.
- `xyz-php-env` is no longer active.
- No assignment-specific Elastic Beanstalk environment remains running.

### Why is this required?

This confirms that the primary managed resources created during the assignment have been removed.

### Dependency

Depends on Cleanup Steps 1 through 5.

### What happens if this verification is skipped?

You may accidentally leave active resources running and consume AWS promotional credits or generate charges.

---

## Cleanup Step 7: Verify Supporting Resources

Elastic Beanstalk normally removes environment resources when the environment is terminated. However, perform a final verification to ensure that no assignment-specific resource remains unexpectedly active.

### Verify EC2 Instances

~~~text
AWS Management Console
→ EC2
→ Instances
→ Instances
~~~

Confirm that no EC2 instance associated with `xyz-php-env` remains in the `Running` state.

### Verify EBS Volumes

~~~text
AWS Management Console
→ EC2
→ Elastic Block Store
→ Volumes
~~~

Confirm that no unnecessary EBS volume created for the terminated environment remains in the `Available` state.

An `Available` EBS volume is unattached but can still generate storage charges.

Do not delete unrelated volumes belonging to other assignments or workloads.

### Verify Load Balancers

Because this assignment used a `Single instance` environment, no Elastic Load Balancer should have been required.

You may verify:

~~~text
AWS Management Console
→ EC2
→ Load Balancing
→ Load Balancers
~~~

Confirm that no unexpected load balancer was created specifically for `xyz-php-env`.

Do not delete unrelated load balancers.

### Verify S3 Storage

Elastic Beanstalk may use S3 for application source bundles.

If you explicitly deleted the application version and selected the option to remove its source bundle, confirm that unnecessary assignment-specific application artifacts are no longer retained.

Do not delete:

- Unrelated S3 buckets.
- Files belonging to other assignments.
- AWS-managed resources whose purpose you have not identified.

### Verify CloudFormation Stacks If Present

Elastic Beanstalk may use supporting AWS provisioning mechanisms depending on platform and environment configuration.

Navigate to:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
~~~

If an assignment-specific stack associated with the terminated Elastic Beanstalk environment is visible, verify that it has been removed successfully.

Do not manually delete an active Elastic Beanstalk-managed stack before terminating the corresponding Elastic Beanstalk environment.

### Why is this required?

Supporting resources can consume credits or generate charges even after the main workload is no longer needed.

A final inspection reduces the risk of leaving billable resources active.

### Dependency

Perform this verification only after the Elastic Beanstalk environment has been completely terminated.

### What happens if this verification is skipped?

Unexpected resources such as running EC2 instances or unattached EBS volumes could remain unnoticed and continue generating charges.

---

# Final Resource Verification

After completing cleanup, verify the following expected final states:

| Resource | Expected Final State |
| -------- | -------------------- |
| Elastic Beanstalk application `xyz-php-app` | `Deleted` |
| Elastic Beanstalk environment `xyz-php-env` | `Terminated / Deleted` |
| PHP application version `php-app-v1` | `Deleted` if no longer required |
| EC2 instance managed by `xyz-php-env` | `Terminated` |
| EBS volume associated with the terminated environment | `Deleted` unless intentionally retained |
| Elastic Load Balancer | `Not created` for the intended single-instance configuration |
| Assignment-specific application source bundle | `Deleted` if no longer required |
| Assignment-specific CloudFormation stack, if created automatically | `Deleted` after environment termination |
| Local `index.php` file | May be retained locally for GitHub documentation or future practice |
| Local `php-app.zip` file | May be retained locally for future practice |

---

# Final Assignment Result

The required tasks have been completed:

1. An AWS Elastic Beanstalk web server environment was created using the `PHP` managed platform.
2. A simple `index.php` application was packaged into `php-app.zip`.
3. The custom PHP source bundle was uploaded and deployed to the Elastic Beanstalk environment.
4. The application was verified through the generated Elastic Beanstalk domain URL.
5. PHP runtime execution was confirmed by dynamically displaying the PHP version.
6. The underlying managed environment and EC2 infrastructure were verified.
7. Common deployment issues and their troubleshooting paths were documented.
8. All assignment resources were cleaned up in reverse dependency order to minimize AWS credit consumption and potential charges.

---

