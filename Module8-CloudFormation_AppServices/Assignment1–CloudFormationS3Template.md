# Module 8: Assignment 1 - AWS CloudFormation S3 Template Deployment

## Problem Statement
XYZ Corporation requires a repeatable, standardized mechanism to provision identical infrastructure stacks across Testing, Development, and Production environments. Implement Infrastructure as Code (IaC) by authoring an AWS CloudFormation template that programmatically deploys an Amazon S3 storage bucket with active Bucket Versioning configurations enforced automatically upon creation.

---

## Tasks To Be Performed
1. Create a CloudFormation template capable of provisioning an Amazon S3 bucket named `intellipaat-corporate-bucket-[unique-id]`.
2. Ensure the template programmatically enables Bucket Versioning configuration for the created resource.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the CloudFormation Template File
1. Open a text editor (e.g., Notepad, VS Code) on your local machine.
2. Copy and paste the following baseline **YAML** CloudFormation blueprint layout:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation template to deploy a standardized corporate S3 storage bucket with versioning enabled.'

Parameters:
  BucketNameSuffix:
    Type: String
    Default: 'production-assets'
    Description: 'Enter a unique suffix string to maintain global namespace compliance.'

Resources:
  CorporateS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      # S3 bucket names must be globally unique
      BucketName: !Sub 'intellipaat-${BucketNameSuffix}'
      VersioningConfiguration:
        Status: Enabled

Outputs:
  BucketNameOutput:
    Description: 'The globally unique name of the newly provisioned S3 Bucket.'
    Value: !Ref CorporateS3Bucket

```

3. Save the file locally on your desktop as **`s3-versioning-template.yaml`**.

---

### Step 2: Deploy the Infrastructure Stack via CloudFormation

1. Log in to the **AWS Management Console** and search for **CloudFormation** to launch the stacks dashboard dashboard.
2. Click on the orange **Create stack** button and select **With new resources (standard)**.
3. Under *Prerequisite - Prepare template*, select **Template is ready**.
4. Under *Specify template*, select **Upload a template file**, click **Choose file**, select your local `s3-versioning-template.yaml`, and click **Next**.
5. Configure stack parameters:
* **Stack name:** `XYZ-S3-Deployment-Stack`
* **BucketNameSuffix:** Modify this string parameter to include a random short number or name (e.g., `corp-storage-2026-xyz`) to prevent global namespace collisions.


6. Click **Next** through the *Configure stack options* page (keep defaults intact).
7. Scroll to the bottom of the review page panel and click **Submit**.
8. **Verification:** Monitor the deployment status event logs. Within 1 minute, the status will shift to **CREATE_COMPLETE**.
9. Switch to the **Outputs** tab to view your final generated bucket identity name. Navigate to the S3 console to verify that the bucket exists and its **Properties** tab shows **Bucket Versioning: Enabled**.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

One of the primary benefits of CloudFormation is automated infrastructure teardown. To safely remove all provisions, utilize the exact order below:

### 1. Empty the Managed S3 Bucket Objects

*Note: CloudFormation will fail to delete an S3 bucket if objects or versions have been written inside it during testing.*

1. Navigate to the **Amazon S3 Console** and click on your provisioned bucket name.
2. Under the **Objects** tab, select all test files (if any), click **Delete**, confirm via the `permanently delete` safety input prompt string, and clear the assets.

### 2. Delete the CloudFormation Stack Container

1. Return to the **AWS CloudFormation Dashboard** and click on **Stacks**.
2. Select the radio check box button corresponding to `XYZ-S3-Deployment-Stack`.
3. Click on the **Delete** button positioned at the top command cluster options menu.
4. Confirm by clicking **Delete stack**.
5. AWS automation loops will instantly dismantle the entire deployment structure, deleting the underlying version-controlled S3 bucket systematically.

