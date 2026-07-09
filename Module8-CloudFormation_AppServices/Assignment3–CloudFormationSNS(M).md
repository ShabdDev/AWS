# Module 8 - CloudFormation and App Services

# Assignment 3 – CloudFormation SNS

# Part 1: Assignment Overview, Cost Check, Architecture, Prerequisites, and SNS Topic Setup

---

## Problem Statement

You work for XYZ Corporation. Your team is asked to deploy similar architecture multiple times for testing, development, and production purposes. Implement AWS CloudFormation for the tasks assigned below.

## Tasks To Be Performed

1. Use the template from CloudFormation Task 1.
2. Add notifications to the CloudFormation stack using Amazon SNS so that an email notification is received for every event during the stack creation process.

---

## Important Implementation Approach

This assignment uses two AWS services:

- **AWS CloudFormation** — Creates and manages infrastructure as code using a template.
- **Amazon Simple Notification Service (Amazon SNS)** — Sends stack event notifications to an email address.

The implementation will follow this dependency order:

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Create Amazon SNS Topic
    |
    v
Create Email Subscription
    |
    v
Confirm Email Subscription
    |
    v
Prepare CloudFormation Task 1 Template
    |
    v
Create CloudFormation Stack
    |
    +------------------------------+
    |                              |
    v                              v
CloudFormation Resources       SNS Notifications
Are Created                    Are Published
    |                              |
    +---------------+--------------+
                    |
                    v
           Email Notifications
           for Stack Events
                    |
                    v
              Verify Success
                    |
                    v
                 Cleanup
~~~

The SNS topic must exist before it can be configured as a notification destination for the CloudFormation stack.

---

# Free Tier and Cost Check

Before creating AWS resources, it is important to understand whether the services used in this assignment can generate charges.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| AWS CloudFormation | Generally no additional CloudFormation charge for standard AWS resources | Usually no direct CloudFormation infrastructure charge | You pay for the AWS resources created by the CloudFormation template. |
| Amazon SNS | Limited free usage may be available depending on account and current AWS Free Tier terms | Can consume credits if usage exceeds applicable free allowances | Email notifications and API requests are generally very low-cost for a small lab assignment. |
| Resources from CloudFormation Task 1 | Depends on the resources in the original template | Possibly | Any EC2, storage, networking, database, or other billable resources created by the template may generate charges. |

## Is This Assignment Completely Free Tier Eligible?

Not necessarily.

The SNS notification portion of this assignment typically generates negligible cost for a small lab. However, the actual cost depends primarily on the AWS resources created by the CloudFormation Task 1 template.

For example:

- If Task 1 creates only low-cost or free-tier-eligible resources, the assignment may have little or no effective cost.
- If Task 1 creates an EC2 instance, database, NAT Gateway, Elastic IP address, load balancer, or another billable resource, charges may apply.
- AWS Free Tier eligibility depends on the account's eligibility, service, region, resource type, and current AWS pricing terms.

## AWS Promotional Credits

If AWS promotional credits are available in the account, eligible charges may be deducted from those credits according to the applicable credit terms.

For a learning environment, it is still important to delete all unnecessary resources immediately after successful verification.

## Resources to Delete After Verification

After completing and verifying this assignment, delete:

1. The CloudFormation stack.
2. The SNS email subscription, if no longer required.
3. The SNS topic, if no longer required.
4. Any resources created outside the CloudFormation stack specifically for this assignment.

Deleting the CloudFormation stack normally instructs CloudFormation to delete resources managed by that stack, subject to resource configuration, deletion policies, retained resources, and deletion failures.

---

# Architecture

The logical architecture for this assignment is:

~~~text
                           +----------------------+
                           |      AWS Account     |
                           +----------+-----------+
                                      |
                                      v
                           +----------------------+
                           |   Amazon SNS Topic   |
                           |   cf-stack-alerts    |
                           +----------+-----------+
                                      |
                                      v
                           +----------------------+
                           |  Email Subscription  |
                           |  Status: Confirmed   |
                           +----------+-----------+
                                      ^
                                      |
                           Stack Event Notifications
                                      |
                           +----------+-----------+
                           |  AWS CloudFormation  |
                           |        Stack         |
                           +----------+-----------+
                                      |
                                      v
                           +----------------------+
                           | Resources Defined in |
                           |   Task 1 Template    |
                           +----------------------+
~~~

## How the Architecture Works

1. An Amazon SNS topic is created.
2. An email subscription is added to the SNS topic.
3. AWS sends a subscription confirmation email to the specified email address.
4. The subscription must be confirmed.
5. The CloudFormation stack is created using the template from Task 1.
6. During stack creation, the SNS topic is configured as the stack notification destination.
7. CloudFormation publishes stack events to the SNS topic.
8. Amazon SNS forwards the notifications to the confirmed email subscriber.

---

# Prerequisites

Before beginning the implementation, ensure that the following requirements are available:

| Prerequisite | Required | Purpose |
| ------------ | -------- | ------- |
| AWS Account | Yes | Required to access CloudFormation and SNS. |
| AWS Management Console Access | Yes | Used to create the SNS topic and CloudFormation stack. |
| Valid Email Address | Yes | Receives the SNS subscription confirmation and CloudFormation event notifications. |
| CloudFormation Task 1 Template | Yes | Required because the assignment explicitly instructs us to reuse the Task 1 template. |
| AWS Region | Yes | SNS and CloudFormation resources should be created in the intended region. |
| Required IAM Permissions | Yes | The signed-in IAM identity must have sufficient permissions for CloudFormation, SNS, and resources created by the Task 1 template. |

---

# Important Region Requirement

Use the same AWS Region consistently for the SNS topic and CloudFormation stack.

For example:

- **Region:** `Asia Pacific (Mumbai)`
- **Region Code:** `ap-south-1`

The exact region may be different if your previous CloudFormation Task 1 resources were created elsewhere.

## Why should the same region be used?

The SNS topic configured for CloudFormation stack notifications should be available in the same region where the stack is being created.

Using a consistent region also makes the lab easier to manage and reduces the risk of searching for resources in the wrong AWS region.

---

# Step 1: Sign In to the AWS Management Console

### Navigation

~~~text
AWS Management Console
→ Sign in to your AWS account
→ Open the AWS Management Console
~~~

### Configuration

After signing in, verify that you are using the intended AWS account.

Then check the AWS Region selector in the upper-right area of the AWS Management Console.

For this example:

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

If your previous CloudFormation Task 1 used another region, use that region instead.

### Why is this step required?

Every resource in this assignment is created inside an AWS account and region context.

Selecting the correct region before creating resources helps ensure that:

- The SNS topic is created in the expected region.
- The CloudFormation stack is created in the expected region.
- Resources are easier to locate and manage.
- Region-related configuration mistakes are avoided.

### Dependency

This step has no previous resource dependency.

It requires only:

- A valid AWS account.
- Permission to access the required AWS services.

### What happens if this step is skipped?

If you do not verify the region, you may accidentally:

- Create the SNS topic in the wrong region.
- Create the CloudFormation stack in another region.
- Fail to find previously created resources.
- Configure the wrong notification destination.

---

# Step 2: Open the Amazon SNS Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "Simple Notification Service"
→ Select Simple Notification Service
~~~

You may also search for:

~~~text
SNS
~~~

### What is Amazon SNS?

Amazon Simple Notification Service, commonly called Amazon SNS, is a managed publish/subscribe messaging service.

For this assignment:

~~~text
CloudFormation
      |
      | Publishes stack events
      v
SNS Topic
      |
      | Delivers notification
      v
Email Subscriber
~~~

The SNS topic acts as the communication channel between CloudFormation and the email subscriber.

### Why is this step required?

The assignment requires email notifications for CloudFormation stack events.

Amazon SNS provides the notification mechanism used to deliver those events to an email address.

### Dependency

This step depends on:

- Step 1: Successful AWS Management Console sign-in.
- Correct AWS Region selection.

### What happens if this step is skipped?

Without SNS:

- There is no notification topic.
- There is no email subscription.
- CloudFormation cannot publish stack event notifications to the required email destination.

Therefore, the second assignment requirement would not be satisfied.

---

# Step 3: Create an Amazon SNS Topic

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
→ Create topic
~~~

### Configuration

Configure the SNS topic as follows:

| Setting | Value |
| ------- | ----- |
| Type | `Standard` |
| Name | `cloudformation-stack-notifications` |
| Display name | Leave blank unless required for your own identification |
| Encryption | Keep the default configuration for this lab unless your organization requires encryption |
| Access policy | Keep the default configuration |
| Delivery status logging | Keep the default configuration |
| Tags | Optional |

After entering the required values, select:

~~~text
Create topic
~~~

## Why Use a Standard SNS Topic?

A Standard SNS topic is appropriate for this assignment because CloudFormation stack notifications do not require FIFO ordering.

The topic will receive event messages generated during CloudFormation stack operations.

Examples of possible stack events include:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
CREATE_FAILED
ROLLBACK_IN_PROGRESS
ROLLBACK_COMPLETE
DELETE_IN_PROGRESS
DELETE_COMPLETE
~~~

The exact notifications received depend on the stack operation and generated CloudFormation events.

### Why is this step required?

CloudFormation needs a notification destination.

The SNS topic provides that destination.

The flow is:

~~~text
CloudFormation Stack Event
        |
        v
SNS Topic
        |
        v
Confirmed Email Subscription
        |
        v
Email Inbox
~~~

### Dependency

This step depends on:

- Step 1: Correct AWS account and region.
- Step 2: Access to the Amazon SNS console.

It does not depend on the CloudFormation stack yet because the SNS topic must be created before being selected as the stack notification destination.

### What happens if this step is skipped?

If the SNS topic is not created:

- There will be no SNS topic ARN to use for stack notifications.
- The email subscription cannot be created.
- CloudFormation stack notifications cannot be delivered through SNS.
- The assignment requirement will remain incomplete.

---

# Step 4: Record the SNS Topic ARN

After creating the topic, AWS opens the topic details page.

Locate the **ARN** field.

The ARN will look similar to:

~~~text
arn:aws:sns:ap-south-1:123456789012:cloudformation-stack-notifications
~~~

Do not copy the example ARN literally.

Your actual ARN contains:

- Your selected AWS region.
- Your AWS account ID.
- Your SNS topic name.

## ARN Structure

~~~text
arn:aws:sns:ap-south-1:123456789012:cloudformation-stack-notifications
|   |   |       |           |                       |
|   |   |       |           |                       +-- SNS topic name
|   |   |       |           +-------------------------- AWS account ID
|   |   |       +-------------------------------------- AWS region
|   |   +---------------------------------------------- AWS service
|   +-------------------------------------------------- AWS partition
+------------------------------------------------------ Amazon Resource Name prefix
~~~

### Why is this step required?

The SNS topic ARN uniquely identifies the SNS topic.

CloudFormation uses this ARN when the topic is selected as a notification destination for stack events.

### Dependency

This step depends on:

- Step 3: The SNS topic must already exist.

### What happens if this step is skipped?

You may still be able to select the topic through the AWS Console, but understanding and recording the ARN is useful for:

- Verification.
- Troubleshooting.
- Infrastructure documentation.
- Future CLI or automation use.

---

# Step 5: Create an Email Subscription for the SNS Topic

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
→ Select cloudformation-stack-notifications
→ Subscriptions
→ Create subscription
~~~

Depending on the current AWS Console layout, the **Create subscription** option may also appear directly on the SNS topic details page.

### Configuration

Configure the subscription as follows:

| Setting | Value |
| ------- | ----- |
| Topic ARN | Automatically populated with the ARN of `cloudformation-stack-notifications` |
| Protocol | `Email` |
| Endpoint | Your valid email address |

For example:

~~~text
your-email@example.com
~~~

Replace the example with the actual email address where you want to receive CloudFormation notifications.

Then select:

~~~text
Create subscription
~~~

### Important Security Note

Do not commit your real email address, AWS account ID, credentials, access keys, or other sensitive account-specific information to a public GitHub repository.

For GitHub documentation, use placeholders such as:

~~~text
your-email@example.com
123456789012
~~~

### Why is this step required?

Creating the SNS topic alone does not specify where notifications should be delivered.

The subscription connects the SNS topic to an endpoint.

In this assignment, the endpoint is an email address.

The relationship is:

~~~text
SNS Topic
    |
    v
Email Subscription
    |
    v
Email Inbox
~~~

### Dependency

This step depends on:

- Step 3: The SNS topic must already exist.
- Step 4: The SNS topic ARN must be available.

### What happens if this step is skipped?

The SNS topic may receive CloudFormation notifications, but no email subscriber will exist to receive them.

Therefore:

- No notification emails will arrive.
- The assignment's email notification requirement will not be successfully demonstrated.

---

# Step 6: Confirm the SNS Email Subscription

After creating an email subscription, its initial status will normally be:

~~~text
Pending confirmation
~~~

AWS sends a subscription confirmation message to the specified email address.

### Action

1. Open the inbox of the email address entered in Step 5.
2. Look for an AWS notification subscription confirmation email.
3. Open the email.
4. Select the **Confirm subscription** link.
5. Return to the Amazon SNS console.
6. Refresh the subscription list.

### Navigation for Verification

~~~text
AWS Management Console
→ Simple Notification Service
→ Subscriptions
→ Locate the subscription
→ Check Status
~~~

### Expected Status

The subscription should no longer show:

~~~text
Pending confirmation
~~~

Instead, it should display an active confirmed subscription with its subscription ARN available.

### Why is this step required?

Amazon SNS does not deliver normal topic notifications to an unconfirmed email subscription.

Email confirmation proves that the owner of the email address has agreed to receive notifications from the SNS topic.

The required sequence is:

~~~text
Create Email Subscription
        |
        v
Pending Confirmation
        |
        v
AWS Sends Confirmation Email
        |
        v
User Confirms Subscription
        |
        v
Subscription Becomes Active
        |
        v
SNS Can Deliver Notifications
~~~

### Dependency

This step depends on:

- Step 5: The email subscription must already exist.
- Access to the email inbox used as the subscription endpoint.

### What happens if this step is skipped?

If the subscription remains in `Pending confirmation` state:

- The email endpoint will not receive normal SNS notifications.
- CloudFormation may publish events to the SNS topic, but the intended email recipient will not receive them.
- The assignment cannot be properly verified.

---

# Step 7: Verify the SNS Topic and Subscription Before Creating the Stack

Before proceeding to CloudFormation, verify that the SNS notification infrastructure is ready.

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
→ Select cloudformation-stack-notifications
~~~

### What should be visible?

Verify the following:

| Item | Expected Value or State |
| ---- | ----------------------- |
| Topic name | `cloudformation-stack-notifications` |
| Topic type | `Standard` |
| Topic ARN | Valid SNS ARN |
| Email subscription | Present |
| Subscription status | Confirmed and active |

The ARN should follow a structure similar to:

~~~text
arn:aws:sns:REGION:ACCOUNT-ID:cloudformation-stack-notifications
~~~

### Why does this verification matter?

The CloudFormation stack has not yet been created.

This verification confirms that the notification infrastructure is ready before the dependent CloudFormation stack configuration begins.

The dependency is:

~~~text
Confirmed SNS Topic and Email Subscription
                    |
                    v
       CloudFormation Stack Creation
                    |
                    v
       Stack Event Notifications
                    |
                    v
              Email Inbox
~~~

If the SNS topic or subscription is incorrectly configured, the stack could still be created successfully while the required email notifications fail.

---

# Part 2: Prepare the CloudFormation Template, Create the Stack, and Configure SNS Notifications

---

# Step 8: Prepare the CloudFormation Task 1 Template

The assignment explicitly requires:

> Use the template from CloudFormation Task 1.

Therefore, the same CloudFormation template created in Task 1 should be reused for this assignment.

The purpose of this assignment is not to replace the Task 1 architecture. Instead, the objective is to create a CloudFormation stack from that template and configure the previously created SNS topic as the stack notification destination.

## Important Clarification

The architecture is:

~~~text
Existing CloudFormation Task 1 Template
                |
                v
      Create CloudFormation Stack
                |
                +--------------------------+
                |                          |
                v                          v
       Create AWS Resources       Publish Stack Events
                                           |
                                           v
                                      SNS Topic
                                           |
                                           v
                                  Confirmed Email
                                     Subscription
                                           |
                                           v
                                       Email Inbox
~~~

The SNS notification topic does not have to be added as a resource inside the Task 1 template when using the AWS Console's stack notification configuration.

Instead, the existing SNS topic can be selected during the CloudFormation stack creation workflow.

---

## Template Requirement

Before continuing, ensure that the CloudFormation Task 1 template is available on your local computer.

For example:

~~~text
cloudformation-task-1.yaml
~~~

or:

~~~text
cloudformation-task-1.yml
~~~

or:

~~~text
cloudformation-task-1.json
~~~

The exact filename can differ depending on how the previous assignment was completed.

### Why is this step required?

CloudFormation requires a template that defines the AWS resources to be created.

The template acts as the infrastructure blueprint.

For example:

~~~text
CloudFormation Template
        |
        | Defines
        v
AWS Resources
        |
        +---- VPC
        |
        +---- Subnet
        |
        +---- Security Group
        |
        +---- EC2 Instance
        |
        +---- Other Resources
~~~

The exact resources depend on the template created in CloudFormation Task 1.

### Dependency

This step depends on:

- The completion of CloudFormation Task 1.
- Availability of the Task 1 template file.

### What happens if this step is skipped?

Without a CloudFormation template:

- The stack cannot be created.
- CloudFormation has no infrastructure definition to process.
- No stack creation events will be generated for the required implementation.
- The SNS notification requirement cannot be properly demonstrated.

---

# Step 9: Review the CloudFormation Template Before Uploading It

Before uploading the Task 1 template, open it in a text editor such as:

- Visual Studio Code
- Notepad
- Notepad++
- Any YAML-compatible or JSON-compatible editor

Verify that the template is complete and correctly formatted.

A typical CloudFormation YAML template has a structure similar to:

~~~yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: CloudFormation template reused from Task 1.

Resources:

  MyResource:
    Type: AWS::Service::ResourceType
    Properties:
      PropertyName: PropertyValue
~~~

The example above demonstrates only the basic CloudFormation template structure. Do not replace your actual Task 1 template with this generic example.

## Important YAML Formatting Rules

When reviewing a YAML CloudFormation template:

1. Use spaces for indentation.
2. Do not use tab characters for indentation.
3. Maintain consistent indentation.
4. Ensure property names are correctly spelled.
5. Ensure required resource properties are present.
6. Ensure resource references point to valid logical IDs.
7. Do not expose credentials or secrets directly inside the template.

### Why is this step required?

A syntax error or invalid property can prevent CloudFormation from creating the stack.

Reviewing the template before uploading helps prevent errors such as:

~~~text
Template format error
~~~

~~~text
Unresolved resource dependencies
~~~

~~~text
Encountered unsupported property
~~~

~~~text
Invalid template resource property
~~~

### Dependency

This step depends on:

- Step 8: The Task 1 template must be available.

### What happens if this step is skipped?

If an invalid template is uploaded:

- Stack creation may fail before resource creation begins.
- CloudFormation may generate failure events.
- The stack may enter a rollback state.
- The intended infrastructure may not be created.

---

# Step 10: Open the AWS CloudFormation Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "CloudFormation"
→ Select CloudFormation
~~~

Ensure that the AWS Region is the same region used for the SNS topic.

For this example:

| Setting | Value |
| ------- | ----- |
| Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

If you used another region in Part 1 or in CloudFormation Task 1, continue using that same region.

### Why is this step required?

AWS CloudFormation is the service responsible for:

- Reading the infrastructure template.
- Creating the stack.
- Provisioning resources.
- Tracking resource creation events.
- Publishing stack notifications to the configured SNS topic.

### Dependency

This step depends on:

- Step 1: Correct AWS account and region.
- Step 7: SNS topic and confirmed email subscription are ready.
- Step 8: Task 1 template is available.

### What happens if this step is skipped?

Without opening and using CloudFormation:

- The stack cannot be created.
- The Task 1 template cannot be deployed.
- No stack creation events will be produced.
- No CloudFormation notifications will be sent through SNS.

---

# Step 11: Start Creating the CloudFormation Stack

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Create stack
→ With new resources (standard)
~~~

Depending on the current AWS Console layout, if no stacks exist in the selected region, you may see a prominent **Create stack** button directly on the CloudFormation dashboard.

Select:

~~~text
Create stack
→ With new resources (standard)
~~~

### Why Use "With New Resources (Standard)"?

This option creates a new CloudFormation stack and provisions the resources defined in the template.

It is appropriate because the assignment requires deploying the Task 1 architecture and receiving notifications during stack creation.

### Why is this step required?

A CloudFormation template is only a definition file.

The stack is the deployed and managed instance of that template.

The relationship is:

~~~text
Template
    |
    | Used to create
    v
CloudFormation Stack
    |
    | Manages
    v
AWS Resources
~~~

### Dependency

This step depends on:

- Step 10: Access to the CloudFormation console.
- The Task 1 template must be ready for upload.

### What happens if this step is skipped?

The template remains only a file and no infrastructure is deployed.

Therefore:

- No resources are created.
- No stack events occur.
- No SNS notifications are generated.

---

# Step 12: Specify the CloudFormation Template

On the **Create stack** page, locate the **Prerequisite - Prepare template** section.

### Configuration

Select:

| Setting | Value |
| ------- | ----- |
| Prepare template | `Choose an existing template` |
| Template source | `Upload a template file` |

Then select:

~~~text
Choose file
~~~

Select your CloudFormation Task 1 template from your computer.

For example:

~~~text
cloudformation-task-1.yaml
~~~

After the file is selected, choose:

~~~text
Next
~~~

## What Happens When the Template Is Uploaded?

CloudFormation uploads and processes the template.

The flow is:

~~~text
Local Template File
        |
        v
Upload to CloudFormation
        |
        v
CloudFormation Reads Template
        |
        v
Template Structure Is Processed
        |
        v
Continue to Stack Configuration
~~~

### Why is this step required?

CloudFormation needs the infrastructure definition before it can create a stack.

The uploaded template specifies:

- Which AWS resources should be created.
- Resource properties.
- Relationships between resources.
- Dependencies.
- References.
- Outputs, if defined.

### Dependency

This step depends on:

- Step 8: Task 1 template is available.
- Step 9: Template has been reviewed.
- Step 11: Stack creation workflow has started.

### What happens if this step is skipped?

CloudFormation will not know what infrastructure to create, and the stack creation workflow cannot continue.

---

# Step 13: Specify the Stack Name and Parameters

After selecting **Next**, CloudFormation displays the **Specify stack details** page.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Create stack
→ Specify stack details
~~~

### Configuration

Use a descriptive stack name:

| Setting | Value |
| ------- | ----- |
| Stack name | `cloudformation-sns-assignment-stack` |

If the Task 1 template defines parameters, configure them on this page.

For example, the template may contain parameters such as:

| Possible Parameter | Example Value |
| ------------------ | ------------- |
| Instance Type | `t2.micro` or another template-supported value |
| Key Pair | Existing EC2 key pair |
| Environment | `development` |
| VPC CIDR | `10.0.0.0/16` |

The exact parameters depend entirely on the Task 1 template.

Do not invent parameter values that are incompatible with the actual template.

### Important Stack Name Rules

A CloudFormation stack name should:

- Be unique within the selected AWS account and region among active stacks.
- Begin with an alphabetic character.
- Use only supported characters.

The following name is suitable:

~~~text
cloudformation-sns-assignment-stack
~~~

### Why is this step required?

The stack name uniquely identifies the CloudFormation stack.

If parameters exist, their values control configurable aspects of the infrastructure without requiring modifications to the template itself.

### Dependency

This step depends on:

- Step 12: Successful template selection and upload.

### What happens if this step is skipped?

The stack creation workflow cannot continue without a valid stack name.

If required template parameters are missing or invalid:

- CloudFormation will prevent continuation, or
- Stack creation may fail because the required configuration is unavailable.

---

# Step 14: Configure Stack Options

After entering the stack details, select:

~~~text
Next
~~~

This opens the **Configure stack options** page.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Create stack
→ Configure stack options
~~~

On this page, CloudFormation may display sections such as:

- Tags
- Permissions
- Stack failure options
- Rollback configuration
- Notification options
- Stack creation options
- Capabilities, depending on the template

The exact layout can vary slightly as AWS updates the Management Console.

---

# Step 15: Add the SNS Topic to CloudFormation Notification Options

On the **Configure stack options** page, locate the section related to:

~~~text
Notification options
~~~

Depending on the current AWS Console interface, you may need to expand an **Advanced options** section to find the notification configuration.

Locate the option for configuring an SNS topic for stack notifications.

### Configuration

Select the SNS topic created in Part 1:

| Setting | Value |
| ------- | ----- |
| Notification topic | `cloudformation-stack-notifications` |
| SNS Topic ARN | Your actual SNS topic ARN |

The ARN should have a structure similar to:

~~~text
arn:aws:sns:ap-south-1:123456789012:cloudformation-stack-notifications
~~~

Do not use the example account ID literally.

Use the actual topic created in Part 1.

## Important Requirement

Before continuing, verify that the selected topic is exactly:

~~~text
cloudformation-stack-notifications
~~~

Also ensure that its email subscription has already been confirmed.

---

## How the Notification Configuration Works

After the topic is configured, the notification path becomes:

~~~text
CloudFormation Stack
        |
        | Generates stack events
        v
Configured SNS Topic
        |
        | Publishes notifications
        v
Confirmed Email Subscription
        |
        | Delivers messages
        v
Email Inbox
~~~

Examples of CloudFormation events may include:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
CREATE_FAILED
ROLLBACK_IN_PROGRESS
ROLLBACK_COMPLETE
UPDATE_IN_PROGRESS
UPDATE_COMPLETE
DELETE_IN_PROGRESS
DELETE_COMPLETE
~~~

The exact events depend on the resources in the template and the stack operation performed.

### Why is this step required?

This is the central requirement of the assignment.

Without configuring the SNS topic as the CloudFormation stack notification destination, the stack can still create resources, but the required stack event notifications will not be delivered through SNS.

### Dependency

This step depends on:

- Step 3: SNS topic created.
- Step 5: Email subscription created.
- Step 6: Email subscription confirmed.
- Step 13: Stack name and parameters configured.
- Step 14: Stack options page opened.

### What happens if this step is skipped?

If no SNS notification topic is configured:

- CloudFormation will still record events in the stack's **Events** tab.
- However, those events will not be published to the required SNS topic.
- The confirmed email subscriber will not receive the required CloudFormation notifications.
- The assignment requirement will not be fulfilled.

---

# Step 16: Review Additional Stack Options

Before selecting **Next**, review the remaining configuration options.

For a basic lab assignment, the default settings are usually sufficient unless the Task 1 template requires additional permissions or capabilities.

Possible sections include:

| Option | Recommended Lab Action |
| ------ | ---------------------- |
| Tags | Optional unless required by Task 1 |
| IAM permissions | Leave default unless a specific CloudFormation service role is required |
| Stack failure options | Keep default unless the assignment specifies otherwise |
| Rollback configuration | Keep default |
| Termination protection | Keep disabled for a temporary lab unless specifically required |
| Notification options | Select the SNS topic created in Part 1 |

## IAM Capabilities

If the Task 1 template creates IAM resources, CloudFormation may require acknowledgement of capabilities such as:

~~~text
I acknowledge that AWS CloudFormation might create IAM resources.
~~~

or:

~~~text
CAPABILITY_IAM
~~~

or:

~~~text
CAPABILITY_NAMED_IAM
~~~

Only acknowledge these capabilities if the actual template contains IAM resources that require them.

Do not select capabilities unnecessarily.

### Why is this step required?

Stack options control additional behavior such as:

- Permissions.
- Rollback behavior.
- Notifications.
- Protection settings.
- IAM acknowledgements.

Reviewing them prevents accidental configuration errors.

### Dependency

This step depends on:

- Step 15: SNS notification topic configured.

### What happens if this step is skipped?

If required options or capabilities are not configured:

- CloudFormation may reject stack creation.
- IAM resource creation may fail.
- Required notifications may not be configured.

---

# Step 17: Review the CloudFormation Stack Configuration

Select:

~~~text
Next
~~~

CloudFormation opens the review page.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Create stack
→ Review and create
~~~

Review all configuration carefully.

### Configuration to Verify

| Item | Expected Configuration |
| ---- | ---------------------- |
| Stack name | `cloudformation-sns-assignment-stack` |
| Template | CloudFormation Task 1 template |
| Parameters | Valid values required by the template |
| AWS Region | Same intended region used for the SNS topic |
| Notification topic | `cloudformation-stack-notifications` |
| Email subscription | Already confirmed |

If the template creates IAM resources and CloudFormation displays a required capability acknowledgement, review it carefully and acknowledge it only when required by the template.

### Why is this step required?

This is the final opportunity to catch configuration mistakes before CloudFormation starts creating real AWS resources.

Potential mistakes include:

- Wrong template.
- Wrong stack name.
- Incorrect parameter values.
- Missing SNS topic.
- Wrong AWS region.
- Missing IAM capability acknowledgement.

### Dependency

This step depends on all previous implementation steps.

### What happens if this step is skipped?

The stack cannot be submitted for creation.

If the review itself is rushed and an incorrect configuration is submitted:

- Wrong resources may be created.
- Stack creation may fail.
- Notifications may not reach the intended email address.
- AWS charges may be generated by unintended resources.

---

# Step 18: Create the CloudFormation Stack

After verifying the configuration, select:

~~~text
Submit
~~~

Depending on AWS Console wording, the final action may appear as:

~~~text
Create stack
~~~

After submission, CloudFormation begins creating the stack.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ cloudformation-sns-assignment-stack
~~~

### Initial Expected Stack Status

You should initially see:

~~~text
CREATE_IN_PROGRESS
~~~

The stack status may later change to:

~~~text
CREATE_COMPLETE
~~~

if all resources are successfully created.

If a resource fails, you may instead see statuses such as:

~~~text
CREATE_FAILED
~~~

~~~text
ROLLBACK_IN_PROGRESS
~~~

~~~text
ROLLBACK_COMPLETE
~~~

The exact status depends on the failure and rollback configuration.

### Why is this step required?

Submitting the stack causes CloudFormation to:

1. Read the template.
2. Evaluate resource dependencies.
3. Begin resource provisioning.
4. Generate stack events.
5. Publish configured notifications to the SNS topic.
6. Continue until stack creation succeeds or fails.

### Dependency

This step depends on:

- Valid Task 1 template.
- Valid stack configuration.
- Confirmed SNS subscription.
- SNS notification topic configured for the stack.
- Required IAM permissions.

### What happens if this step is skipped?

No stack is created.

Therefore:

- No infrastructure is provisioned.
- No stack creation events are generated.
- No CloudFormation event notifications are sent.
- The assignment cannot be completed.

---

# Step 19: Observe CloudFormation Stack Events

Immediately after creating the stack, open the **Events** tab.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ cloudformation-sns-assignment-stack
→ Events
~~~

The Events tab displays the chronological sequence of stack and resource operations.

### Example Expected Events

The exact events depend on the Task 1 template.

A simplified example may look like:

~~~text
Logical ID                              Status
---------------------------------------------------------
cloudformation-sns-assignment-stack    CREATE_IN_PROGRESS
MyResource                             CREATE_IN_PROGRESS
MyResource                             CREATE_COMPLETE
cloudformation-sns-assignment-stack    CREATE_COMPLETE
~~~

If multiple resources exist, each resource may generate multiple events.

For example:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
~~~

If a resource fails:

~~~text
CREATE_FAILED
~~~

CloudFormation may then initiate rollback:

~~~text
ROLLBACK_IN_PROGRESS
ROLLBACK_COMPLETE
~~~

### What should you observe?

Verify that:

- The stack appears in the CloudFormation console.
- Stack events are generated.
- Resource-level events are visible.
- The final stack state eventually becomes `CREATE_COMPLETE` for a successful deployment.

### Why is this step required?

The Events tab provides direct evidence that CloudFormation is processing the template and managing resources.

It also provides a reference for comparing CloudFormation console events with the notifications delivered through SNS.

### Dependency

This step depends on:

- Step 18: Stack creation has started.

### What happens if this step is skipped?

The infrastructure may still be created, but you will miss an important verification opportunity.

You will not directly observe:

- Resource creation order.
- Individual resource status changes.
- Failure reasons.
- Rollback events.
- Final stack status progression.

---

# Step 20: Check the Email Inbox for SNS Notifications

While the stack is being created, open the inbox associated with the confirmed SNS email subscription.

### Action

Check:

- Inbox
- Spam folder
- Junk folder
- Promotions or filtered folders, if applicable

Look for AWS SNS notification emails related to CloudFormation stack events.

### Expected Notification Flow

~~~text
CloudFormation Event Occurs
        |
        v
CloudFormation Publishes Notification
        |
        v
SNS Topic Receives Message
        |
        v
Confirmed Email Subscription
        |
        v
Email Notification Arrives
~~~

### Possible Notification Content

Depending on the generated event, the message may contain information related to:

~~~text
StackId
Timestamp
EventId
LogicalResourceId
PhysicalResourceId
ResourceProperties
ResourceStatus
ResourceType
ResourceStatusReason
ClientRequestToken
~~~

Not every event necessarily contains meaningful values for every possible field.

### What should you observe?

You should receive email notifications associated with CloudFormation stack events.

The events may include statuses such as:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
~~~

The number of emails depends on:

- Number of resources in the template.
- Number of generated stack events.
- Whether any resources fail.
- Whether rollback occurs.

### Why is this step required?

Receiving the email notification directly proves that the complete notification path is working:

~~~text
CloudFormation
    |
    v
SNS Topic
    |
    v
Confirmed Subscription
    |
    v
Email Inbox
~~~

### Dependency

This step depends on:

- SNS topic creation.
- Email subscription creation.
- Email subscription confirmation.
- SNS topic selection in CloudFormation notification options.
- CloudFormation stack events being generated.

### What happens if this step is skipped?

You may know that the stack was created, but you will not have verified the primary requirement of this assignment: receiving CloudFormation stack event notifications through SNS email delivery.

---
# Part 3: Complete Verification, Troubleshooting, and Cleanup

---

# Verification Step 1: Verify the Final CloudFormation Stack Status

After the CloudFormation stack creation process finishes, verify that the stack reached the expected successful state.

### Action

Open the CloudFormation stack and check its final status.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Stack info
~~~

### Expected Output

For a successful stack creation, the stack status should be:

~~~text
CREATE_COMPLETE
~~~

### What should I observe?

Verify the following:

| Item | Expected Result |
| ---- | --------------- |
| Stack name | `cloudformation-sns-assignment-stack` |
| Stack status | `CREATE_COMPLETE` |
| Stack operation | Creation completed successfully |
| Stack resources | Created successfully |

If the stack displays:

~~~text
CREATE_COMPLETE
~~~

it means CloudFormation successfully processed the template and completed creation of the stack.

If you see another status, such as:

~~~text
CREATE_IN_PROGRESS
~~~

wait until the operation finishes.

If you see:

~~~text
CREATE_FAILED
~~~

or:

~~~text
ROLLBACK_IN_PROGRESS
~~~

or:

~~~text
ROLLBACK_COMPLETE
~~~

the stack encountered an error and you should inspect the **Events** tab to identify the failed resource and failure reason.

### Why does this confirm success?

`CREATE_COMPLETE` is the successful terminal status for a newly created CloudFormation stack.

It confirms that:

- CloudFormation successfully processed the template.
- Required resource dependencies were resolved.
- Resources defined by the template were successfully created, subject to the final resource statuses shown in the stack.
- The stack creation operation completed without requiring rollback.

---

# Verification Step 2: Verify Individual CloudFormation Stack Events

### Action

Open the **Events** tab of the CloudFormation stack and inspect the generated events.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Events
~~~

### Expected Output

The exact events depend on the resources defined in the CloudFormation Task 1 template.

A simplified example is:

~~~text
Logical ID                              Status
------------------------------------------------------------
cloudformation-sns-assignment-stack    CREATE_IN_PROGRESS
MyResource                             CREATE_IN_PROGRESS
MyResource                             CREATE_COMPLETE
cloudformation-sns-assignment-stack    CREATE_COMPLETE
~~~

For a template containing multiple resources, additional events will appear.

Typical successful creation events include:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
~~~

### What should I observe?

Verify that:

- The stack itself generated creation events.
- Each resource defined in the template generated appropriate events.
- Resource creation progressed from `CREATE_IN_PROGRESS` to `CREATE_COMPLETE`.
- The final stack-level event shows `CREATE_COMPLETE`.

Depending on the template, resources may be created sequentially or in parallel based on their dependencies.

### Why does this confirm success?

The CloudFormation **Events** tab is the authoritative event history for stack operations.

It proves that CloudFormation:

- Evaluated the template.
- Determined resource dependencies.
- Started resource provisioning.
- Tracked individual resource status changes.
- Completed the stack creation operation.

These are also the types of events that the configured notification system is intended to publish through Amazon SNS.

---

# Verification Step 3: Verify the CloudFormation Stack Resources

### Action

Open the **Resources** tab and inspect all resources managed by the stack.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Resources
~~~

### Expected Output

The exact resources depend on the CloudFormation Task 1 template.

A typical view may contain:

~~~text
Logical ID          Resource Type                 Status
----------------------------------------------------------------
MyResource          AWS::Service::ResourceType    CREATE_COMPLETE
~~~

If the Task 1 template creates multiple resources, every successfully created resource should normally show:

~~~text
CREATE_COMPLETE
~~~

### What should I observe?

Verify the following:

| Verification Item | Expected Result |
| ----------------- | --------------- |
| Logical IDs | Match resources defined in the Task 1 template |
| Resource types | Match the intended AWS resources |
| Resource status | `CREATE_COMPLETE` |
| Physical IDs | Created where applicable |

### Why does this confirm success?

The **Resources** tab confirms that the CloudFormation template was not merely accepted but was actually used to provision and manage AWS resources.

It also provides the mapping between:

~~~text
Template Logical Resource ID
            |
            v
Actual AWS Physical Resource
~~~

---

# Verification Step 4: Verify the SNS Topic

### Action

Open the Amazon SNS console and verify that the notification topic still exists.

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
→ Select cloudformation-stack-notifications
~~~

### Expected Configuration

| Item | Expected Value |
| ---- | -------------- |
| Topic name | `cloudformation-stack-notifications` |
| Topic type | `Standard` |
| Topic ARN | Valid SNS topic ARN |
| Region | Same intended region used for the CloudFormation stack |

The topic ARN should have a structure similar to:

~~~text
arn:aws:sns:ap-south-1:123456789012:cloudformation-stack-notifications
~~~

Do not use the example account ID literally.

### What should I observe?

Verify that:

- The topic exists.
- The topic name is correct.
- The topic ARN is valid.
- The topic is in the expected AWS Region.

### Why does this confirm success?

The SNS topic is the notification destination used by CloudFormation.

Its existence confirms that the messaging component of the architecture is available.

However, this verification alone does not prove that email delivery works. The subscription and received email notifications must also be verified.

---

# Verification Step 5: Verify the SNS Email Subscription

### Action

Open the SNS subscription list and verify that the email subscription is active.

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Subscriptions
→ Locate the subscription associated with cloudformation-stack-notifications
~~~

### Expected Output

The subscription should not show:

~~~text
Pending confirmation
~~~

Instead, it should be confirmed and active, with a subscription ARN available.

### What should I observe?

Verify the following:

| Item | Expected Result |
| ---- | --------------- |
| Topic | `cloudformation-stack-notifications` |
| Protocol | `Email` |
| Endpoint | Your intended email address |
| Subscription status | Confirmed and active |
| Subscription ARN | Available |

### Why does this confirm success?

Amazon SNS does not normally deliver topic notifications to an email subscription that remains in `Pending confirmation` status.

A confirmed subscription proves that the email endpoint is eligible to receive notifications published to the SNS topic.

---

# Verification Step 6: Verify the CloudFormation Notification Topic Configuration

### Action

Open the CloudFormation stack details and inspect the configured notification options.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Stack info
~~~

Locate the stack notification configuration.

Depending on the current AWS Console layout, notification details may appear under stack information, stack details, or related stack configuration information.

### Expected Output

The configured SNS notification topic should correspond to:

~~~text
cloudformation-stack-notifications
~~~

Its ARN should have a structure similar to:

~~~text
arn:aws:sns:REGION:ACCOUNT-ID:cloudformation-stack-notifications
~~~

### What should I observe?

Verify that:

- A notification topic is associated with the CloudFormation stack.
- The configured topic is the same SNS topic created earlier.
- The topic ARN contains the correct region and topic name.

### Why does this confirm success?

This proves that CloudFormation was explicitly configured to publish stack event notifications to the intended SNS topic.

Without this association, the SNS topic and email subscription could exist independently without receiving CloudFormation stack events.

---

# Verification Step 7: Verify CloudFormation Event Notifications in the Email Inbox

### Action

Open the inbox associated with the confirmed SNS email subscription.

Check:

- Inbox
- Spam folder
- Junk folder
- Promotions folder
- Other filtered folders, if applicable

Look for Amazon SNS notification emails generated during the CloudFormation stack creation process.

### Expected Output

You should receive notifications associated with CloudFormation stack events.

Possible event statuses include:

~~~text
CREATE_IN_PROGRESS
CREATE_COMPLETE
~~~

Depending on the resources in the template and the stack creation process, emails may contain event information such as:

~~~text
StackId
Timestamp
EventId
LogicalResourceId
PhysicalResourceId
ResourceProperties
ResourceStatus
ResourceType
ResourceStatusReason
ClientRequestToken
~~~

The exact fields and values may vary by event.

### What should I observe?

Verify that:

- Email notifications were received.
- The notifications relate to the correct CloudFormation stack.
- The stack name or stack ID corresponds to `cloudformation-sns-assignment-stack`.
- Resource statuses correspond to events visible in the CloudFormation **Events** tab.
- Notifications were received during the stack creation lifecycle.

### Why does this confirm success?

This verifies the complete end-to-end notification path:

~~~text
CloudFormation Stack
        |
        | Generates event
        v
Amazon SNS Topic
        |
        | Publishes notification
        v
Confirmed Email Subscription
        |
        | Delivers message
        v
Email Inbox
~~~

Receiving these messages proves that:

1. CloudFormation generated stack events.
2. The stack was configured with an SNS notification topic.
3. SNS accepted the notifications.
4. The email subscription was confirmed.
5. SNS successfully delivered notifications to the email endpoint.

This directly satisfies the primary requirement of the assignment.

---

# Verification Step 8: Compare CloudFormation Events with Email Notifications

### Action

Compare the events displayed in the CloudFormation console with the SNS notification emails received in the inbox.

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Events
~~~

Then compare those events with the notification emails.

### Example Comparison

| CloudFormation Event | Expected Email Notification |
| -------------------- | --------------------------- |
| Stack `CREATE_IN_PROGRESS` | Notification related to stack creation beginning |
| Resource `CREATE_IN_PROGRESS` | Notification related to resource creation starting |
| Resource `CREATE_COMPLETE` | Notification related to successful resource creation |
| Stack `CREATE_COMPLETE` | Notification related to successful stack completion |

### What should I observe?

The stack and resource lifecycle events displayed by CloudFormation should correspond to notifications delivered through the configured SNS topic.

The exact number and timing of emails can depend on:

- Number of resources.
- Resource dependencies.
- Event generation.
- Email delivery timing.
- Mail provider filtering.

### Why does this confirm success?

This comparison verifies that the notifications are genuinely associated with CloudFormation stack activity rather than unrelated SNS messages.

---

# Troubleshooting Common Issues

The following troubleshooting guidance should be used only if the expected result is not observed.

---

## Troubleshooting Issue 1: Email Subscription Shows Pending Confirmation

### Problem

The SNS subscription status remains:

~~~text
Pending confirmation
~~~

### Possible Causes

- The confirmation email was not opened.
- The confirmation link was not selected.
- The message was delivered to spam or junk.
- The email address was entered incorrectly.
- The confirmation message expired or was deleted.

### Resolution

1. Check the inbox.
2. Check spam and junk folders.
3. Verify that the email address is correct.
4. Open the AWS subscription confirmation email.
5. Select the confirmation link.
6. Return to the SNS console.
7. Refresh the subscription list.

### Why does this matter?

An unconfirmed email subscription will not receive normal notifications from the SNS topic.

---

## Troubleshooting Issue 2: Stack Is Created but No Notification Emails Are Received

### Problem

The CloudFormation stack reaches:

~~~text
CREATE_COMPLETE
~~~

but no SNS notification emails are received.

### Possible Causes

- The email subscription is still pending confirmation.
- The wrong SNS topic was selected during stack creation.
- No notification topic was associated with the stack.
- The email was filtered into spam or junk.
- The wrong email endpoint was subscribed.
- The stack and intended SNS topic configuration do not match.
- Email delivery is delayed.

### Resolution

Verify all of the following:

| Check | Expected Result |
| ----- | --------------- |
| SNS topic exists | Yes |
| Topic name | `cloudformation-stack-notifications` |
| Subscription protocol | `Email` |
| Subscription confirmed | Yes |
| Correct email endpoint | Yes |
| CloudFormation notification topic configured | Yes |
| Correct SNS topic selected | Yes |

Also inspect:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Stack info
~~~

Verify that the correct SNS topic is associated with the stack.

---

## Troubleshooting Issue 3: CloudFormation Stack Enters ROLLBACK_COMPLETE

### Problem

Instead of:

~~~text
CREATE_COMPLETE
~~~

the stack reaches:

~~~text
ROLLBACK_COMPLETE
~~~

### Possible Causes

Common causes include:

- Invalid resource configuration.
- Missing IAM permissions.
- Invalid AMI ID.
- Unsupported instance type in the selected region or Availability Zone.
- Missing EC2 key pair.
- Invalid subnet or VPC reference.
- Resource quota exceeded.
- Resource name conflict.
- Invalid template parameter value.

### Resolution

Open the Events tab:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Events
~~~

Find the first relevant resource event showing:

~~~text
CREATE_FAILED
~~~

Read the **Status reason** field.

The first underlying resource failure usually provides the most useful information for identifying the root cause.

### Why should the first failure be investigated?

Later errors may simply be consequences of the original failure.

For example:

~~~text
EC2 Instance CREATE_FAILED
        |
        v
Stack ROLLBACK_IN_PROGRESS
        |
        v
Other Created Resources Are Deleted
        |
        v
Stack ROLLBACK_COMPLETE
~~~

The EC2 failure is the root cause in this example, while rollback events are consequences.

---

## Troubleshooting Issue 4: SNS Topic Is Not Available During Stack Configuration

### Problem

The expected SNS topic cannot be found when configuring CloudFormation notification options.

### Possible Causes

- The wrong AWS Region is selected.
- The topic was deleted.
- The topic was created in another AWS account.
- The wrong topic name is being searched for.

### Resolution

Verify the current region in both services:

~~~text
Amazon SNS
→ Check selected AWS Region
~~~

~~~text
AWS CloudFormation
→ Check selected AWS Region
~~~

The expected topic is:

~~~text
cloudformation-stack-notifications
~~~

Ensure you are working in the correct AWS account and region.

---

## Troubleshooting Issue 5: Stack Remains in CREATE_IN_PROGRESS for a Long Time

### Problem

The stack remains in:

~~~text
CREATE_IN_PROGRESS
~~~

longer than expected.

### Possible Causes

- A resource takes time to provision.
- A resource is waiting for another dependency.
- An underlying AWS service operation is delayed.
- A custom resource or wait condition is involved.
- A resource operation is eventually going to time out.

### Resolution

Open:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Events
~~~

Identify the latest resource showing:

~~~text
CREATE_IN_PROGRESS
~~~

Then inspect the corresponding AWS service if necessary.

Do not manually delete resources while CloudFormation is actively managing the stack unless you fully understand the consequences, because out-of-band changes can cause stack inconsistencies.

---

# Cleanup and Cost Optimization

After successful verification, delete all resources created for this assignment.

Cleanup should follow reverse dependency order:

~~~text
CloudFormation Stack
        |
        v
Wait for DELETE_COMPLETE
        |
        v
SNS Email Subscription
        |
        v
SNS Topic
        |
        v
Final Resource Verification
~~~

## Why does deletion order matter?

The CloudFormation stack depends on the SNS topic for notifications.

Therefore, deleting the stack first allows CloudFormation to send deletion lifecycle notifications while the SNS topic and confirmed email subscription still exist.

After stack deletion completes, the SNS subscription and topic can be removed.

---

# Cleanup Step 1: Delete the CloudFormation Stack

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
→ Select cloudformation-sns-assignment-stack
→ Delete
~~~

### Action

1. Select `cloudformation-sns-assignment-stack`.
2. Select **Delete**.
3. Confirm the deletion when prompted.
4. Monitor the stack deletion process.

The stack should enter:

~~~text
DELETE_IN_PROGRESS
~~~

After successful deletion, it should reach:

~~~text
DELETE_COMPLETE
~~~

The stack may then disappear from the default active stack list.

### Why is this required?

Deleting the stack instructs CloudFormation to delete the AWS resources managed by the stack, subject to:

- Deletion policies.
- Retain settings.
- Resource dependencies.
- Deletion protection.
- Deletion failures.

Some resources created by the Task 1 template may continue generating charges if the stack is not deleted.

### Dependency

Delete the CloudFormation stack before deleting the SNS topic.

This preserves the notification infrastructure while stack deletion events are generated.

### What happens if this resource is not deleted?

Possible consequences include:

- EC2 instance charges.
- Storage charges.
- Database charges.
- Load balancer charges.
- NAT Gateway charges.
- Public IPv4 address charges.
- Consumption of promotional AWS credits.

The actual charges depend on the resources defined in the Task 1 template.

---

# Cleanup Step 2: Verify Successful Stack Deletion

### Navigation

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
~~~

If necessary, change the stack status filter to include deleted stacks.

### Action

Verify that:

~~~text
cloudformation-sns-assignment-stack
~~~

has reached:

~~~text
DELETE_COMPLETE
~~~

or no longer appears in the default active stack list.

### Why is this required?

Do not assume that selecting **Delete** means every resource was successfully removed.

A stack can encounter deletion failures such as:

~~~text
DELETE_FAILED
~~~

If this happens, inspect the **Events** tab and identify the resource that could not be deleted.

### Dependency

This step depends on Cleanup Step 1.

Do not proceed under the assumption that stack resources are gone until stack deletion has been verified.

### What happens if this verification is skipped?

Resources may remain active and continue:

- Generating charges.
- Consuming AWS credits.
- Creating security exposure.
- Blocking future resource creation because of naming conflicts or dependencies.

---

# Cleanup Step 3: Delete the SNS Email Subscription

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Subscriptions
→ Select the email subscription associated with cloudformation-stack-notifications
→ Delete
~~~

### Action

Select the subscription associated with:

~~~text
cloudformation-stack-notifications
~~~

Then select **Delete** and confirm the deletion if prompted.

### Why is this required?

The subscription is no longer necessary after the CloudFormation stack has been deleted and the assignment has been verified.

Deleting it prevents future notifications from being delivered through this assignment-specific subscription.

### Dependency

This step should be performed after:

- The CloudFormation stack has been deleted.
- Stack deletion has been verified.

The subscription must be deleted before or as part of deleting the SNS topic.

### What happens if this resource is not deleted?

The subscription may remain attached to the SNS topic and could continue receiving messages if future notifications are published to that topic.

---

# Cleanup Step 4: Delete the SNS Topic

### Navigation

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
→ Select cloudformation-stack-notifications
→ Delete
~~~

### Action

1. Select the topic:

~~~text
cloudformation-stack-notifications
~~~

2. Select **Delete**.
3. Enter the topic name if the console requests deletion confirmation.
4. Confirm the deletion.

### Why is this required?

The SNS topic was created specifically for this assignment.

If it is no longer needed, deleting it keeps the AWS account clean and prevents accidental reuse or unnecessary configuration clutter.

### Dependency

Delete the SNS topic only after:

1. The CloudFormation stack has been successfully deleted.
2. Stack deletion has been verified.
3. The SNS email subscription is no longer required.

### What happens if this resource is not deleted?

An unused SNS topic generally does not create significant cost merely by existing, but leaving unnecessary resources can:

- Cause account clutter.
- Create confusion during future labs.
- Lead to accidental notification publishing.
- Make resource management more difficult.

---

# Cleanup Step 5: Perform Final Resource Verification

After completing cleanup, verify that all assignment resources have been removed.

### Navigation

Check both services:

~~~text
AWS Management Console
→ CloudFormation
→ Stacks
~~~

and:

~~~text
AWS Management Console
→ Simple Notification Service
→ Topics
~~~

and:

~~~text
AWS Management Console
→ Simple Notification Service
→ Subscriptions
~~~

### Final Resource Verification Table

| Resource | Expected Final State |
| -------- | -------------------- |
| CloudFormation stack `cloudformation-sns-assignment-stack` | `DELETE_COMPLETE` or absent from active stacks |
| Resources managed by the CloudFormation stack | `Deleted`, except any resources intentionally retained by template policy or configuration |
| SNS email subscription | `Deleted` |
| SNS topic `cloudformation-stack-notifications` | `Deleted` |

### What should I observe?

After successful cleanup:

- The CloudFormation stack should no longer appear as an active stack.
- Stack-managed resources should be deleted unless intentionally retained.
- The SNS email subscription should no longer exist.
- The SNS topic should no longer exist.

### Why does this confirm successful cleanup?

This confirms that the temporary infrastructure created for the assignment has been removed and reduces the risk of:

- Ongoing AWS charges.
- Promotional credit consumption.
- Resource conflicts.
- Unnecessary AWS account clutter.

---

# Assignment Result

The completed implementation satisfies the required tasks:

1. The CloudFormation template from Task 1 was reused.
2. An Amazon SNS topic was created.
3. An email subscription was created and confirmed.
4. The SNS topic was configured as the CloudFormation stack notification destination.
5. CloudFormation generated events during stack creation.
6. SNS delivered CloudFormation event notifications to the confirmed email address.
7. The implementation was verified through the CloudFormation console, SNS console, and email inbox.
8. All temporary resources were deleted in reverse dependency order after verification.

> **Assignment complete. No additional parts are required.**
