# Module 8: Assignment 3 - AWS CloudFormation with Amazon SNS Stack Notifications

## Problem Statement
XYZ Corporation requires enhanced visibility into automated deployment cycles across multiple testing and production stages. Implement a continuous notification pipeline by expanding the baseline CloudFormation template from Task 1. Integrate an Amazon Simple Notification Service (SNS) Topic to programmatically route status emails to engineers at every transitional milestone of the infrastructure lifecycle.

---

## Tasks To Be Performed
1. Extend the AWS CloudFormation template developed in Task 1 (S3 Bucket template).
2. Integrate an Amazon SNS Topic within the infrastructure parameters to receive email alerts for every operational step of the stack status tracking lifecycle.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the CloudFormation Template with S3 and SNS Integration
1. Open a text editor (e.g., Notepad or VS Code) on your machine.
2. Paste the following structured **YAML** infrastructure blueprint:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation template provisioning an S3 bucket and an SNS Topic for email stack step notifications.'

Parameters:
  BucketNameSuffix:
    Type: String
    Default: 'sns-alert-bucket'
    Description: 'Enter a unique suffix string to maintain global namespace compliance.'
  
  NotificationEmail:
    Type: String
    Description: 'Enter your valid email address to receive CloudFormation stack state notifications.'

Resources:
  # 1. Create the S3 Bucket (From Task 1)
  CorporateS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub 'intellipaat-${BucketNameSuffix}'
      VersioningConfiguration:
        Status: Enabled

  # 2. Create the Amazon SNS Notification Topic
  StackNotificationTopic:
    Type: 'AWS::SNS::Topic'
    Properties:
      TopicName: 'XYZ-Stack-Notification-Topic'
      Subscription:
        - Endpoint: !Ref NotificationEmail
          Protocol: email

Outputs:
  S3BucketName:
    Description: 'The globally unique name of the newly provisioned S3 Bucket.'
    Value: !Ref CorporateS3Bucket
  SNSTopicARN:
    Description: 'The Amazon Resource Name (ARN) of the generated SNS Topic.'
    Value: !Ref StackNotificationTopic

```

3. Save the file locally on your desktop as **`s3-sns-notification-template.yaml`**.

---

### Step 2: Deploy the Infrastructure Stack with Notification Flags

1. Log in to the **AWS Management Console** and open the **CloudFormation** dashboard.
2. Click **Create stack** > **With new resources (standard)**.
3. Select **Template is ready**, click **Upload a template file**, choose `s3-sns-notification-template.yaml`, and click **Next**.
4. Configure the stack parameters:
* **Stack name:** `XYZ-S3-SNS-Stack`
* **BucketNameSuffix:** Add a random string (e.g., `corp-storage-july2026`) for global uniqueness.
* **NotificationEmail:** Type your active personal/work email address.


5. Click **Next** to proceed to the **Configure stack options** page.
6. **CRITICAL STEP FOR TRACKING EVENTS:** On the *Configure stack options* page, scroll down to the **Notification options** block section.
* Under *Amazon SNS topic*, choose **Create new Amazon SNS topic** or select the stack-managed tracking channel if configured natively. *(To catch all event logs directly via the engine engine loop, you can map the orchestration profile to listen to state change rules).*


7. Click **Next**, scroll to the bottom summary view, and click **Submit**.

---

### Step 3: Confirm Email Subscription

1. Immediately check the inbox of the email address you entered as the parameter.
2. You will find an official confirmation email from AWS Notifications with the subject line **"AWS Notification - Subscription Confirmation"**.
3. Open the email and click the **Confirm subscription** link. This grants AWS permissions to deliver automated event payloads to your address.
4. As the stack continues its backend compilation steps (`CREATE_IN_PROGRESS`, `RESOURCE_CREATE_COMPLETE`, `CREATE_COMPLETE`), you will receive granular structural progress update logs directly in your email.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To remove the resources cleanly and automatically deregister email endpoints without leaving orphan tracking parameters inside your console dashboards:

### 1. Wipe Underlying Storage Assets

1. Open the **Amazon S3 Console** > click on your provisioned bucket name.
2. Ensure the object folder structures inside are empty. (If test objects were generated, select them all, click **Delete**, type `permanently delete`, and clear them).

### 2. Teardown Stack Container (Removes S3 and SNS Subscriptions)

1. Navigate back to the **AWS CloudFormation Dashboard** and choose the **Stacks** panel view.
2. Select the checkbox item matching **`XYZ-S3-SNS-Stack`**.
3. Click on the **Delete** button positioned at the top context options row header.
4. Confirm by clicking **Delete stack**.
5. **Automation Result:** CloudFormation will send a teardown payload alert. You will receive a final set of automated emails mapping the progressive destruction stages of the architecture. Once complete, the S3 bucket and the SNS subscriber endpoint will be fully deleted.

