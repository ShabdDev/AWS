# Module 8: Assignment 4 - Provisioning AWS SQS FIFO Queue and SES Email Validation via CloudFormation

## Problem Statement
XYZ Corporation requires automated orchestration tools to establish decoupled messaging lines and reliable customer communication endpoints across various product tiers. Implement an AWS CloudFormation template to programmatically deploy a first-in, first-out (FIFO) Amazon Simple Queue Service (SQS) message queue. Additionally, register and verify an enterprise email identity within Amazon Simple Email Service (SES) to validate end-to-end sandbox message deliveries.

---

## Tasks To Be Performed
1. Create a CloudFormation template to provision a fully operational SQS FIFO queue and test it by sending sample messages.
2. Register and verify an active email identity within Amazon SES and distribute a test email transaction.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Author the CloudFormation Template for SQS FIFO Queue
1. Open a text editor (e.g., Notepad or VS Code) on your computer.
2. Copy and paste the following standardized **YAML** infrastructure layout template:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation template to deploy an Amazon SQS FIFO queue with required properties.'

Resources:
  XYZCorporateFIFOQueue:
    Type: 'AWS::SQS::Queue'
    Properties:
      # FIFO queues must strictly terminate with the '.fifo' extension suffix
      QueueName: 'XYZ-Corporate-Transactions.fifo'
      FifoQueue: true
      ContentBasedDeduplication: true

Outputs:
  QueueURL:
    Description: 'The URL endpoint of the newly deployed SQS FIFO Queue.'
    Value: !Ref XYZCorporateFIFOQueue
  QueueARN:
    Description: 'The Amazon Resource Name (ARN) of the deployed SQS FIFO Queue.'
    Value: !GetAtt XYZCorporateFIFOQueue.Arn

```

3. Save the file locally on your desktop configuration profile as **`sqs-fifo-template.yaml`**.

---

### Step 2: Deploy and Test the SQS FIFO Queue

1. Log in to the **AWS Management Console** and navigate to the **CloudFormation Dashboard**.
2. Click **Create stack** > **With new resources (standard)**.
3. Select **Template is ready**, click **Upload a template file**, choose `sqs-fifo-template.yaml` from your machine, and click **Next**.
4. **Specify stack options:** Enter Stack name as `XYZ-SQS-FIFO-Stack` and click **Next**.
5. Keep defaults on the options configurations page, click **Next**, and click **Submit**. Wait until status logs read **CREATE_COMPLETE**.
6. **Test Sending Messages (Task 1 Validation):**
* Navigate to the **Amazon SQS Console** and click on your queue named `XYZ-Corporate-Transactions.fifo`.
* Click on the **Send and receive messages** button positioned at the top right.
* In the **Message body** layout canvas, type a sample string payload: `{"transaction_id": 9942, "status": "PENDING"}`.
* Because this is a FIFO queue, provide a required **Message group ID** (e.g., `group_1`).
* Click **Send message**. Your message is now held securely in the decoupled memory tier.



---

### Step 3: Register and Verify Email Identity in Amazon SES (Task 2 Requirement)

1. Navigate to the **Amazon SES (Simple Email Service)** console management window.
2. In the left navigation pane under *Configuration*, click on **Verified identities**.
3. Click on the orange **Create identity** button.
4. Configure identity parameters:
* **Identity type:** Choose **Email address**.
* **Email address:** Type your valid personal or corporate email address (e.g., `yourname@example.com`).


5. Click **Create identity**.
6. **Execute Verification Action:** * Open the inbox of the email address you just specified.
* Locate the official verification message sent from AWS with the subject line **"Amazon Web Services – Email Address Verification Request"**.
* Click the provided URL link to authorize permissions. The SES status dashboard will transition from *Pending* to a green **Verified** status flag.


7. **Send a Test Email:**
* Inside the SES console under *Verified identities*, select your newly verified address.
* Click the **Send test email** utility command link button.
* Choose **Scenario:** `Custom` | **Subject:** `XYZ Corporate Cloud Test Email` | **Body:** `This email confirms successful integration testing of Amazon SES with the corporate app pipeline.`
* Click **Send test email**. Check your inbox to confirm the instant delivery of the alert.
---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To prevent resource clutter and clean up your sandbox environments, complete these automated teardown tasks in order:

### 1. Remove Verified Email Identities from Amazon SES

1. Open the **Amazon SES Console** > Click **Verified identities** from the sidebar menu panel.
2. Select the checkbox container positioned next to your verified email address string.
3. Locate the top action cluster options and click **Delete**. Confirm the permanent removal from the environment.

### 2. Teardown the SQS Queue Infrastructure via CloudFormation

1. Navigate to the **AWS CloudFormation Dashboard** and look at the **Stacks** partition layout view.
2. Check the box group matching **`XYZ-SQS-FIFO-Stack`**.
3. Click the **Delete** button found on the top navigation toolbar.
4. Click **Delete stack** on the confirmation pop-up modal interface.
5. **Automation Result:** CloudFormation will clear all backend memory bindings and delete the `XYZ-Corporate-Transactions.fifo` queue securely from your account profile within a few seconds.

