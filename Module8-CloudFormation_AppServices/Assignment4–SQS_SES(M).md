# Module 8 - CloudFormation and App Services

# Assignment 4 – SQS and SES

# Part 1: Cost Check, Architecture, Prerequisites, and FIFO SQS Queue Creation

---

## Problem Statement

You work for XYZ Corporation. Your team is asked to deploy similar architecture multiple times for testing, development, and production purposes. Implement the assigned AWS messaging and email services.

## Tasks To Be Performed

1. Create a FIFO Amazon SQS queue and test it by sending messages.
2. Register your email address in Amazon SES and send a test email to yourself.

---

## 1. Assignment Implementation Scope

This assignment uses two AWS services:

- **Amazon Simple Queue Service (Amazon SQS):** Used to create a FIFO message queue and test message delivery.
- **Amazon Simple Email Service (Amazon SES):** Used to verify an email identity and send a test email.

No EC2 instance, VPC, subnet, security group, IAM user, database, or other supporting AWS resource is required for this assignment.

The complete implementation will be performed through the AWS Management Console.

> **Important:** Although this assignment is under the CloudFormation and App Services module, the stated tasks specifically require creating and testing an SQS FIFO queue and registering an email in SES. Therefore, this solution implements those exact tasks directly through the AWS Management Console without adding unrequested infrastructure.

---

## 2. Free Tier and Cost Check

Before creating any resources, it is important to understand whether the assignment can generate AWS charges.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| Amazon SQS | Yes, subject to current AWS Free Tier terms and usage limits | Possibly, if usage exceeds applicable free allowances | AWS currently provides up to 1 million Amazon SQS requests per month free under its published SQS pricing terms. A small assignment test uses only a few requests. |
| Amazon SES | Depends on account age and applicable AWS Free Tier model | Yes, when applicable | SES pricing and free usage depend on when the AWS account was created and the applicable AWS Free Tier model. A single test email has negligible cost, but usage should still be monitored. |
| AWS Management Console | No separate charge | No | There is no separate fee for using the AWS Management Console itself. |

### Cost Assessment

For this assignment:

- The SQS portion requires only a few API requests to create the queue, send messages, receive messages, and delete messages.
- Amazon SQS currently includes 1 million free requests per month under its published pricing terms.
- The SES portion sends only one or a few test emails.
- SES eligibility and billing depend on the AWS account's applicable Free Tier model and account creation date.
- AWS accounts created under the newer AWS Free Tier model may use promotional credits toward eligible SES usage.
- For such a small lab, expected usage is minimal, but it is still good practice to delete the SQS queue after completing verification.
- An SES verified email identity does not itself continuously send emails or consume SQS requests.

### Resources That Can Incur Charges

The following activities may generate charges if applicable allowances or credits are exceeded:

- Large numbers of SQS API requests.
- Large SQS message payloads requiring multiple request units.
- Data transfer outside applicable free allowances.
- Large volumes of emails sent through SES.
- Optional SES paid features that are not required in this assignment.

### Recommended Cost-Saving Action

After successfully completing and verifying the assignment:

1. Delete the SQS FIFO queue.
2. Delete the SES verified email identity if it is no longer needed.
3. Avoid sending unnecessary additional test emails.
4. Check the AWS Billing and Cost Management console if you want to verify current account usage.

---

## 3. Architecture Overview

The assignment contains two independent workflows.

### Architecture Diagram

~~~text
                         AWS Account
                              |
              +---------------+---------------+
              |                               |
              v                               v
      Amazon SQS FIFO Queue           Amazon SES
              |                               |
              v                               v
        Send Message                  Verify Email Identity
              |                               |
              v                               v
       Receive Message                 Send Test Email
              |                               |
              v                               v
      Verify FIFO Messaging          Receive Email in Inbox
              |                               |
              +---------------+---------------+
                              |
                              v
                           Cleanup
~~~

---

## 4. Dependency Flow

The SQS and SES tasks are independent of each other.

### SQS Dependency Flow

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Open Amazon SQS
    |
    v
Create FIFO Queue
    |
    v
Send Test Messages
    |
    v
Receive Messages
    |
    v
Verify FIFO Queue Operation
    |
    v
Delete Queue During Cleanup
~~~

### SES Dependency Flow

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Open Amazon SES
    |
    v
Create Email Identity
    |
    v
Receive Verification Email
    |
    v
Verify Email Address
    |
    v
Send Test Email
    |
    v
Check Inbox
    |
    v
Delete SES Identity During Cleanup
~~~

---

## 5. Important Regional Consideration

Amazon SQS queues and Amazon SES identities are regional resources.

For this assignment, use the same AWS Region consistently.

### Recommended Region

Use:

- **Region:** `Asia Pacific (Mumbai)`
- **Region Code:** `ap-south-1`

This region is geographically suitable for a user working from India and keeps the assignment resources together in one region.

> **Important:** If you select another region, continue using that same region throughout the assignment. An SQS queue created in one region will not appear when viewing another region, and an SES email identity verified in one region may need separate verification in another region.

---

## 6. Prerequisites

Before starting, ensure that you have:

| Prerequisite | Requirement |
| ------------ | ----------- |
| AWS Account | An active AWS account |
| AWS Console Access | Permission to sign in to the AWS Management Console |
| SQS Permissions | Permission to create, view, send to, receive from, purge, and delete SQS queues |
| SES Permissions | Permission to create and delete SES identities and send test emails |
| Email Account | Access to a valid email inbox that you can open for verification |
| AWS Region | Preferably `Asia Pacific (Mumbai)` (`ap-south-1`) |

No command-line tools are required for the primary implementation.

No EC2 instance is required.

No VPC is required.

No IAM user creation is required if the currently signed-in AWS identity already has sufficient permissions.

---

# Part A: Create and Test an Amazon SQS FIFO Queue

---

## Step 1: Sign In to the AWS Management Console

### Navigation

~~~text
AWS Management Console
→ Sign in
→ Open the AWS Console home page
~~~

### Action

Sign in using your authorized AWS credentials.

After signing in, confirm that you can access the AWS Management Console.

### Why is this step required?

All resources in this assignment are created inside an AWS account. The AWS Management Console provides the graphical interface required to create and manage the SQS FIFO queue and SES email identity.

### Dependency

This step has no AWS resource dependency.

It requires only:

- An active AWS account.
- Valid authentication credentials.
- Appropriate IAM permissions.

### What happens if this step is skipped?

You cannot access Amazon SQS or Amazon SES and therefore cannot complete any subsequent implementation steps.

---

## Step 2: Select the AWS Region

### Navigation

~~~text
AWS Management Console
→ Top navigation bar
→ Region selector
→ Asia Pacific (Mumbai)
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

### Action

Select `Asia Pacific (Mumbai)` from the AWS Region selector.

If you intentionally choose a different region, note that region and use it consistently throughout the assignment.

### Why is this step required?

Amazon SQS queues and SES identities are regional resources. Selecting the region before creating resources ensures that you know exactly where those resources exist.

### Dependency

Depends on:

- **Step 1:** Successful AWS Management Console sign-in.

### What happens if this step is skipped?

AWS still uses whichever region is currently selected, but you may accidentally create resources in an unintended region.

This can cause confusion because:

- The SQS queue may appear to be missing when you switch regions.
- The SES email identity may not appear in another region.
- Verification and cleanup may become more difficult.

---

## Step 3: Open the Amazon SQS Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "SQS"
→ Select Simple Queue Service
~~~

### Action

Open the Amazon SQS console.

You should reach the Amazon SQS dashboard or Queues page.

### Why is this step required?

Amazon SQS is the AWS managed message queue service used for Task 1 of the assignment.

The assignment specifically requires a FIFO queue.

FIFO means:

~~~text
First-In-First-Out
~~~

A FIFO queue is designed to preserve message ordering within a message group and supports mechanisms to prevent duplicate message processing.

### Dependency

Depends on:

- **Step 1:** AWS Console access.
- **Step 2:** Correct AWS Region selection.

### What happens if this step is skipped?

You cannot create the FIFO queue required by Task 1.

---

## Step 4: Start Creating the FIFO Queue

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ Create queue
~~~

### Action

Select **Create queue**.

The queue configuration page will open.

### Why is this step required?

This opens the configuration interface where the queue type, queue name, message behavior, access policy, encryption, and other settings can be defined.

### Dependency

Depends on:

- **Step 3:** Amazon SQS console must be open.

### What happens if this step is skipped?

No queue will be created, so messages cannot be sent or tested.

---

## Step 5: Configure the FIFO Queue Type and Name

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ Create queue
→ Details
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Queue Type | `FIFO` |
| Queue Name | `xyz-assignment4-queue.fifo` |

### Action

Under **Type**, select:

`FIFO`

For the queue name, enter:

`xyz-assignment4-queue.fifo`

> **Important:** The name of an Amazon SQS FIFO queue must end with the `.fifo` suffix.

### Why is this step required?

The assignment explicitly requires a FIFO queue rather than a Standard queue.

The FIFO queue provides ordered message processing within a message group. This is useful when message processing sequence matters.

For example:

~~~text
Message 1: Create order
Message 2: Process payment
Message 3: Ship order
~~~

These operations should logically be processed in the correct sequence.

### Dependency

Depends on:

- **Step 4:** The Create queue page must be open.

### What happens if this step is skipped?

If you create a Standard queue instead:

- The assignment requirement will not be satisfied.
- Strict FIFO behavior will not be provided.
- The queue name will not use the required `.fifo` suffix.

---

## Step 6: Configure FIFO Queue Options

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Configuration
~~~

### Configuration

Use the following settings:

| Setting | Recommended Value |
| ------- | ----------------- |
| Visibility timeout | `30 seconds` |
| Message retention period | `4 days` |
| Delivery delay | `0 seconds` |
| Maximum message size | Keep the console default |
| Receive message wait time | `0 seconds` |
| Content-based deduplication | `Enabled` |
| High throughput FIFO | `Disabled` |

> **Console Variation Note:** AWS occasionally updates labels and the visual arrangement of SQS settings. If a setting appears in a slightly different section, use the equivalent current option and preserve the values listed above.

### Understanding the Important Settings

#### Visibility Timeout

Value:

`30 seconds`

When a consumer receives a message, the message temporarily becomes invisible to other consumers for the visibility timeout period.

This prevents multiple consumers from processing the same message simultaneously under normal processing conditions.

#### Message Retention Period

Value:

`4 days`

Amazon SQS stores an unconsumed message for this duration before automatically deleting it.

#### Delivery Delay

Value:

`0 seconds`

The message becomes available for processing immediately after it is sent.

#### Receive Message Wait Time

Value:

`0 seconds`

This keeps the default short-polling behavior for this small console-based assignment.

#### Content-Based Deduplication

Value:

`Enabled`

When content-based deduplication is enabled, Amazon SQS can generate a deduplication ID based on the message body.

This simplifies testing because a manually specified message deduplication ID is not required for every message sent through the console.

#### High Throughput FIFO

Value:

`Disabled`

High-throughput FIFO functionality is unnecessary for this assignment because only a few test messages will be sent.

### Why is this step required?

These settings control:

- How long messages remain stored.
- When messages become visible.
- How duplicate messages are handled.
- Whether high-throughput FIFO functionality is enabled.

For a beginner-level test assignment, the recommended values keep the configuration simple while preserving FIFO behavior.

### Dependency

Depends on:

- **Step 5:** FIFO queue type and name must already be configured.

### What happens if this step is skipped?

AWS defaults may still allow the queue to function, but failing to understand or correctly configure FIFO-specific behavior can lead to:

- Unexpected duplicate handling.
- Confusion during message testing.
- Unnecessary complexity from high-throughput settings.

---

## Step 7: Configure Encryption

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Encryption
~~~

### Configuration

Use the default Amazon SQS managed server-side encryption option if it is presented and enabled by default.

A typical suitable configuration is:

| Setting | Value |
| ------- | ----- |
| Server-side encryption | `Enabled` |
| Encryption key type | `SQS-managed key (SSE-SQS)` |

### Why is this step required?

Server-side encryption protects the contents of messages stored in the queue.

Using the SQS-managed encryption key keeps the lab simple because:

- AWS manages the encryption key.
- No custom AWS KMS key needs to be created.
- No additional KMS key dependency is introduced.

### Dependency

Depends on:

- The queue configuration page being open.

It has no dependency on a customer-managed AWS KMS key when using `SSE-SQS`.

### What happens if this step is skipped?

Depending on the current console defaults, the queue may still be created. However, explicitly reviewing encryption settings helps ensure that stored messages receive appropriate server-side protection.

---

## Step 8: Configure the Queue Access Policy

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Access policy
~~~

### Configuration

For this assignment, keep the basic/default access configuration that allows only the queue owner or authorized identities in the AWS account to access the queue.

Do not make the queue publicly accessible.

### Why is this step required?

The access policy controls which AWS principals can perform operations on the queue.

For this assignment, public access is unnecessary because testing is performed from the authenticated AWS Management Console.

### Dependency

Depends on:

- An authenticated AWS identity with sufficient SQS permissions.

### What happens if this step is skipped?

The default queue-owner access configuration is generally sufficient for this lab. However, incorrectly changing the policy could:

- Prevent you from sending or receiving messages.
- Accidentally grant unnecessary access.
- Create a security risk.

---

## Step 9: Leave the Dead-Letter Queue Option Disabled

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Dead-letter queue
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Dead-letter queue | `Disabled` |

### Why is this step required?

A dead-letter queue is useful in production systems for isolating messages that repeatedly fail processing.

However, this assignment only requires:

- Creating one FIFO queue.
- Sending test messages.
- Receiving and verifying those messages.

Creating another queue as a dead-letter queue would add an unnecessary resource and dependency.

### Dependency

No additional dependency is required.

### What happens if this step is skipped?

If the option remains disabled by default, there is no problem.

If you enable it without creating a compatible dead-letter queue, additional configuration will be required unnecessarily.

---

## Step 10: Add Resource Tags

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Tags
~~~

### Configuration

Add the following tag:

| Key | Value |
| --- | ----- |
| `Name` | `XYZ-Assignment4-FIFO-Queue` |

### Why is this step required?

Tags help identify AWS resources according to:

- Project.
- Assignment.
- Environment.
- Team.
- Cost allocation requirements.

Although tagging is not explicitly required by the assignment, adding a simple identification tag helps prevent accidental deletion of unrelated resources and makes cleanup easier.

### Dependency

Depends on:

- The queue creation page being open.

### What happens if this step is skipped?

The queue will still function, but it may be more difficult to identify among multiple AWS resources.

---

## Step 11: Create the FIFO Queue

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Create queue
→ Review settings
→ Create queue
~~~

### Action

Review the configuration and select **Create queue**.

The expected queue name is:

`xyz-assignment4-queue.fifo`

### Expected Result

After successful creation, the queue should appear on the SQS Queues page.

Expected information includes:

~~~text
Queue name: xyz-assignment4-queue.fifo
Type: FIFO
Status: Available for use
~~~

### Why is this step required?

This action creates the actual Amazon SQS FIFO queue resource.

After creation, the queue can accept messages.

### Dependency

Depends on successful completion of:

- **Step 5:** Queue type and name configuration.
- **Step 6:** FIFO settings.
- **Step 7:** Encryption review.
- **Step 8:** Access policy review.
- **Step 9:** Dead-letter queue configuration.
- **Step 10:** Tags.

### What happens if this step is skipped?

The queue resource will not exist, and message testing cannot begin.

---

## Step 12: Open the FIFO Queue for Message Testing

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ Select xyz-assignment4-queue.fifo
→ Send and receive messages
~~~

### Action

Select the newly created queue and then select **Send and receive messages**.

This opens the interface used to:

- Send messages.
- Poll for messages.
- Receive messages.
- Inspect message contents.
- Delete received messages.

### Why is this step required?

The assignment explicitly requires testing the FIFO queue by sending messages.

This interface provides a direct way to test the queue without writing application code or using the AWS CLI.

### Dependency

Depends on:

- **Step 11:** The FIFO queue must already exist.

### What happens if this step is skipped?

The queue may exist successfully, but Task 1 will remain incomplete because its messaging functionality has not yet been tested.

---

# Part 2: Test the FIFO SQS Queue and Register an Email Identity in Amazon SES

---

## Step 13: Send the First Test Message to the FIFO Queue

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ Select xyz-assignment4-queue.fifo
→ Send and receive messages
→ Send message
~~~

### Configuration

Configure the first message as follows:

| Setting | Value |
| ------- | ----- |
| Message body | `This is Message 1 from XYZ Corporation` |
| Message group ID | `xyz-assignment4-group` |
| Message deduplication ID | Leave empty when content-based deduplication is enabled |

### Message Body

Enter the following text in the **Message body** field:

~~~text
This is Message 1 from XYZ Corporation
~~~

### Message Group ID

Enter:

`xyz-assignment4-group`

### Message Deduplication ID

Because content-based deduplication was enabled while creating the queue, leave the **Message deduplication ID** field empty if the console permits it.

Amazon SQS generates a deduplication ID based on the SHA-256 hash of the message body.

> **Important:** If content-based deduplication was not enabled during queue creation, you must provide a unique Message deduplication ID, such as `message-1`.

### Action

After entering the required values, select **Send message**.

### Expected Result

A success notification should appear indicating that the message was sent successfully.

The exact wording may vary slightly as the AWS Console interface changes.

### Why is this step required?

The assignment requires testing the FIFO queue by sending messages.

This first message establishes the beginning of the message sequence that will later be received and verified.

The **Message group ID** is especially important for FIFO queues because ordering is guaranteed within the same message group.

### Dependency

Depends on:

- **Step 11:** The FIFO queue must exist.
- **Step 12:** The Send and receive messages page must be open.

### What happens if this step is skipped?

No message will enter the queue, so there will be nothing to receive or verify.

---

## Step 14: Send the Second Test Message

### Navigation

Remain on the same page:

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ xyz-assignment4-queue.fifo
→ Send and receive messages
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Message body | `This is Message 2 from XYZ Corporation` |
| Message group ID | `xyz-assignment4-group` |
| Message deduplication ID | Leave empty when content-based deduplication is enabled |

### Message Body

Enter:

~~~text
This is Message 2 from XYZ Corporation
~~~

### Message Group ID

Use the same message group ID:

`xyz-assignment4-group`

### Action

Select **Send message**.

### Expected Result

The AWS Console should display a successful message-sent notification.

### Why is this step required?

Sending a second message allows us to test FIFO ordering.

Because both messages use the same Message group ID, Amazon SQS maintains their processing order within that group.

The intended sequence is:

~~~text
Message 1
    |
    v
Message 2
~~~

### Dependency

Depends on:

- **Step 13:** The first message should already have been sent.
- The same FIFO queue must be used.

### What happens if this step is skipped?

The queue can still be tested with one message, but FIFO ordering cannot be meaningfully demonstrated with only a single message.

---

## Step 15: Send the Third Test Message

### Navigation

Remain on:

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ xyz-assignment4-queue.fifo
→ Send and receive messages
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Message body | `This is Message 3 from XYZ Corporation` |
| Message group ID | `xyz-assignment4-group` |
| Message deduplication ID | Leave empty when content-based deduplication is enabled |

### Message Body

Enter:

~~~text
This is Message 3 from XYZ Corporation
~~~

### Message Group ID

Use:

`xyz-assignment4-group`

### Action

Select **Send message**.

### Expected Result

A successful message-sent notification should appear.

At this point, the expected logical sequence is:

~~~text
This is Message 1 from XYZ Corporation
    |
    v
This is Message 2 from XYZ Corporation
    |
    v
This is Message 3 from XYZ Corporation
~~~

### Why is this step required?

Three messages make it easier to observe FIFO ordering behavior.

Because all three messages belong to the same message group, they are logically ordered according to their sending sequence.

### Dependency

Depends on:

- **Step 13:** First message sent.
- **Step 14:** Second message sent.

### What happens if this step is skipped?

The queue will still function, but using multiple messages provides clearer evidence that FIFO ordering is working.

---

# Verification of Amazon SQS FIFO Queue

---

## Verification Step 1: Confirm That Messages Were Sent Successfully

### Action

After sending all three messages, observe the success notifications displayed by the AWS Console.

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ xyz-assignment4-queue.fifo
→ Send and receive messages
~~~

### Expected Output

You should have successfully sent these three messages:

~~~text
This is Message 1 from XYZ Corporation
This is Message 2 from XYZ Corporation
This is Message 3 from XYZ Corporation
~~~

All messages should use:

~~~text
Message group ID: xyz-assignment4-group
~~~

### What should I observe?

You should observe a successful notification after each message is sent.

The queue should accept all three messages without configuration errors.

### Why does this confirm success?

A successful send operation proves that:

- The FIFO queue exists.
- Your AWS identity has permission to send messages.
- The Message group ID is valid.
- The queue is accepting FIFO messages.

However, sending messages alone does not fully verify message retrieval. The messages must also be received.

---

## Verification Step 2: Poll for Messages

### Action

On the **Send and receive messages** page, locate the **Receive messages** section.

Select:

**Poll for messages**

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ xyz-assignment4-queue.fifo
→ Send and receive messages
→ Receive messages
→ Poll for messages
~~~

### Expected Output

The AWS Console should retrieve available messages from the queue.

You should see messages corresponding to:

~~~text
This is Message 1 from XYZ Corporation
This is Message 2 from XYZ Corporation
This is Message 3 from XYZ Corporation
~~~

> **Important FIFO Behavior:** In a FIFO queue, messages in the same message group are processed sequentially. Depending on message visibility and how the AWS Console polls the queue, you may initially see only the first available message from that message group. You may need to delete the first received message and poll again before the next message becomes available.

### What should I observe?

For each received message, the AWS Console may display information such as:

- Message ID.
- Message body.
- Message group ID.
- Sequence number.
- Sent timestamp.
- Receive count.
- Other message attributes.

The exact console layout can vary over time.

### Why does this confirm success?

Receiving a message proves that:

- The message was successfully stored in Amazon SQS.
- The queue is accessible.
- The receive operation works.
- The queue can deliver messages to a consumer.

---

## Verification Step 3: Verify the First Received Message

### Action

Expand or select the first received message.

Inspect its body.

### Expected Output

~~~text
This is Message 1 from XYZ Corporation
~~~

The Message group ID should be:

`xyz-assignment4-group`

### What should I observe?

Verify that:

- The message body matches Message 1.
- The message belongs to the expected message group.
- A valid Message ID exists.
- FIFO-related information such as a sequence number may be visible.

### Why does this confirm success?

This proves that the first message successfully completed the following path:

~~~text
Sender
    |
    v
Amazon SQS FIFO Queue
    |
    v
Stored Message
    |
    v
Receive Operation
    |
    v
Message 1 Retrieved
~~~

---

## Verification Step 4: Delete the First Received Message

### Action

Select the first received message and choose the option to delete it.

Confirm the deletion if prompted.

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ xyz-assignment4-queue.fifo
→ Send and receive messages
→ Received messages
→ Select Message 1
→ Delete
~~~

### Expected Result

The first message should be permanently removed from the queue.

### What should I observe?

After deletion:

- Message 1 should no longer be available for future delivery.
- The next message in the same FIFO message group can become available for processing.

### Why does this confirm success?

Deleting the successfully processed message is important because FIFO queues preserve ordered processing within a message group.

Removing Message 1 allows Message 2 to proceed as the next message in that group.

---

## Verification Step 5: Poll Again and Verify the Second Message

### Action

Select **Poll for messages** again.

Open the newly received message.

### Expected Output

~~~text
This is Message 2 from XYZ Corporation
~~~

### What should I observe?

Verify:

- Message body is Message 2.
- Message group ID remains `xyz-assignment4-group`.
- Message 2 is retrieved after Message 1 in the logical sequence.

After verifying Message 2, delete it.

### Why does this confirm success?

It demonstrates that messages in the same FIFO message group are being processed sequentially.

The verified order is now:

~~~text
1. This is Message 1 from XYZ Corporation
2. This is Message 2 from XYZ Corporation
~~~

---

## Verification Step 6: Poll Again and Verify the Third Message

### Action

After deleting Message 2, select **Poll for messages** again.

Inspect the returned message.

### Expected Output

~~~text
This is Message 3 from XYZ Corporation
~~~

### What should I observe?

Verify:

- Message 3 is successfully retrieved.
- It belongs to the same message group.
- It follows Message 1 and Message 2 in the intended sequence.

After verification, delete Message 3.

### Why does this confirm success?

The complete verified sequence is:

~~~text
Message 1
    |
    v
Message 2
    |
    v
Message 3
~~~

This successfully demonstrates the required FIFO queue behavior for messages belonging to the same message group.

---

## Verification Step 7: Confirm That the Queue Is Empty

### Action

After deleting all three messages, select **Poll for messages** again.

### Expected Output

No messages should be returned.

Depending on the current AWS Console interface, you may see an empty message list or an indication that no messages are available.

### What should I observe?

The queue should no longer return:

- Message 1.
- Message 2.
- Message 3.

### Why does this confirm success?

This confirms that:

- All three messages were successfully sent.
- All three messages were successfully received.
- Their order was verified.
- All three messages were successfully deleted after processing.

Task 1 of the assignment is now successfully implemented and verified.

---

# Part B: Register an Email Address in Amazon SES

---

## Step 16: Open the Amazon SES Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "SES"
→ Select Amazon Simple Email Service
~~~

### Action

Open the Amazon SES console.

Confirm that the selected AWS Region is still:

`Asia Pacific (Mumbai)`

Region code:

`ap-south-1`

### Why is this step required?

Amazon SES is the AWS email service required for Task 2.

The assignment requires:

1. Registering your email address in SES.
2. Sending a test email to yourself.

### Dependency

Depends on:

- An active AWS account.
- AWS Console access.
- Appropriate SES permissions.
- Access to a valid email inbox.

It does not depend on the SQS queue because SQS and SES are independent in this assignment.

### What happens if this step is skipped?

You cannot verify an email identity or send the required SES test email.

---

## Step 17: Open the SES Identities Page

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
~~~

### Action

Open the **Identities** page.

Depending on the current AWS Console interface, the left navigation may display **Identities** directly under the **Configuration** section.

### Why is this step required?

Before sending email through Amazon SES, you must create and verify an identity.

An SES identity can be:

- An email address.
- A domain.

For this assignment, use an individual email address because the task explicitly requires registering your mail and sending a test email to yourself.

### Dependency

Depends on:

- **Step 16:** Amazon SES console must be open.

### What happens if this step is skipped?

SES will not have a verified sender identity available for the test email.

---

## Step 18: Start Creating an SES Email Identity

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Create identity
~~~

### Action

Select **Create identity**.

### Why is this step required?

This opens the SES identity creation page where the email address can be registered for verification.

### Dependency

Depends on:

- **Step 17:** The SES Identities page must be open.

### What happens if this step is skipped?

No email identity will be registered, so the assignment's SES requirement cannot be completed.

---

## Step 19: Configure the SES Email Identity

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Create identity
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| Identity type | `Email address` |
| Email address | `your-own-email@example.com` |

> **Important:** Replace `your-own-email@example.com` with an actual email address that you own and can access.

For example:

~~~text
yourname@gmail.com
~~~

or:

~~~text
yourname@outlook.com
~~~

Do not enter the example address literally.

### Action

1. Select **Email address** as the identity type.
2. Enter your actual email address.
3. Review any optional settings.
4. Select **Create identity**.

### Expected Result

The identity should be created with a verification status similar to:

`Unverified`

or:

`Pending`

The exact wording can vary depending on the current SES console interface.

Amazon SES sends a verification email to the address you entered.

### Why is this step required?

Amazon SES requires proof that you control the email address before allowing it to be used as a verified identity.

This helps prevent unauthorized email spoofing and abuse.

### Dependency

Depends on:

- **Step 18:** The Create identity page must be open.
- Access to a valid email inbox.

### What happens if this step is skipped?

The email identity will not be registered, and you cannot use it as the verified sender required for the SES test email.

---

## Step 20: Open the SES Verification Email

### Navigation

This step takes place in your email provider rather than the AWS Console.

~~~text
Open your email inbox
→ Check Inbox
→ Search for the Amazon Web Services verification email
→ Open the verification message
~~~

### Action

Open the verification email sent by Amazon SES.

If you do not immediately see the message, check:

- Inbox.
- Spam folder.
- Junk folder.
- Promotions folder, if applicable.

### Expected Result

You should receive an email from AWS containing an email verification link.

### Why is this step required?

Creating an SES identity does not automatically prove ownership of the email address.

You must access the mailbox and use the verification link sent by AWS.

### Dependency

Depends on:

- **Step 19:** The SES email identity must have been created successfully.
- Access to the specified email inbox.

### What happens if this step is skipped?

The SES identity remains unverified or pending.

An unverified identity cannot be used as the verified sender required for normal SES test sending.

---

## Step 21: Verify the Email Address

### Action

Inside the verification email, select the verification link provided by Amazon Web Services.

### Expected Result

A confirmation page should indicate that the email address was successfully verified.

Return to the Amazon SES console after verification.

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Select or refresh the email identity
~~~

### Expected Identity Status

`Verified`

If the status does not update immediately, refresh the page after a short wait.

### Why is this step required?

SES only permits a registered email identity to be used as a verified sender after ownership has been confirmed.

Verification establishes that you control the specified email address.

### Dependency

Depends on:

- **Step 20:** The SES verification email must have been received.
- The verification link must still be valid.

### What happens if this step is skipped?

The email identity remains unverified, preventing successful completion of the required SES email test.

---

## Verification Step 8: Confirm the SES Identity Is Verified

### Action

Return to the SES Identities page and inspect the email identity.

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Locate your email address
~~~

### Expected Output

The identity should show a verification status of:

`Verified`

### What should I observe?

Confirm that:

- Your correct email address appears.
- The identity type is an email address.
- The verification status is `Verified`.

### Why does this confirm success?

A `Verified` status proves that:

- SES successfully registered the email identity.
- The verification email was delivered.
- You successfully confirmed ownership of the email address.

The email identity is now ready to be used for sending the assignment's test email.

---
# Part 3: Send and Verify the SES Test Email, Troubleshoot Common Issues, and Clean Up Resources

---

## Step 22: Open the Verified SES Email Identity

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Select your verified email address
~~~

### Action

Select the email address that you verified in Part 2.

Before continuing, confirm that its verification status is:

`Verified`

### Expected Result

The identity details page should open and show that the email address is successfully verified.

### Why is this step required?

Amazon SES requires a verified sender identity before an email can be sent from that address.

Opening the verified identity also provides access to the **Send test email** action in the SES console.

### Dependency

Depends on:

- **Step 19:** The email identity must have been created.
- **Step 21:** The verification link must have been opened successfully.
- **Verification Step 8:** The identity status must show `Verified`.

### What happens if this step is skipped?

You cannot confirm that the sender identity is verified before attempting to send the test email.

If the identity remains unverified, SES will not allow it to be used as a valid verified sender.

---

## Step 23: Open the Send Test Email Interface

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Select your verified email identity
→ Send test email
~~~

### Action

Select **Send test email**.

The exact position of this button may vary slightly as the AWS Console interface changes, but it is typically available from the verified identity details page.

### Why is this step required?

The **Send test email** interface allows you to test Amazon SES directly through the AWS Management Console without:

- Writing application code.
- Installing the AWS CLI.
- Creating an EC2 instance.
- Configuring an SMTP client.

### Dependency

Depends on:

- **Step 22:** A verified SES email identity must be selected.

### What happens if this step is skipped?

The SES identity may be verified successfully, but the second assignment task remains incomplete because no test email has been sent.

---

## Step 24: Configure the Test Email

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Select verified email identity
→ Send test email
~~~

### Configuration

Use the following values:

| Setting | Value |
| ------- | ----- |
| Email format | `Formatted` |
| From address | Your verified email identity |
| Scenario | `Custom` |
| Custom recipient | Your own verified email address |
| Subject | `AWS SES Assignment 4 Test Email` |
| Body | `This is a test email sent using Amazon SES for Module 8 Assignment 4.` |

> **Console Variation Note:** Depending on the current SES console interface, the recipient field may be displayed as **Custom recipient**, **Recipient**, or another equivalent label. Use your own verified email address as the destination.

### Subject

Enter:

`AWS SES Assignment 4 Test Email`

### Message Body

Enter:

~~~text
This is a test email sent using Amazon SES for Module 8 Assignment 4.
~~~

### Recipient

Enter the same email address that you registered and verified in Amazon SES.

For example:

~~~text
yourname@gmail.com
~~~

Replace this example with your actual verified email address.

### Why is this step required?

The test email requires:

- A sender.
- A recipient.
- A subject.
- A message body.

Using your verified email address as both sender and recipient directly satisfies the assignment requirement to send a test email to yourself.

### Dependency

Depends on:

- **Step 23:** The Send test email interface must be open.
- The sender email identity must have status `Verified`.

### What happens if this step is skipped?

There will be no valid message configuration to send, so SES cannot perform the required email delivery test.

---

## Step 25: Understand the SES Sandbox Restriction Before Sending

New or restricted Amazon SES accounts commonly operate in the SES sandbox.

In the sandbox:

- You can send emails only from verified identities.
- You can generally send emails only to verified recipient identities, except for the SES mailbox simulator.
- Sending quotas are restricted compared with production access.

For this assignment, using your own verified email address as both sender and recipient works with the sandbox restrictions.

### Expected Email Flow

~~~text
Verified SES Email Identity
          |
          |  Send test email
          v
     Amazon SES
          |
          |  Deliver message
          v
Same Verified Email Inbox
~~~

### Why is this step required?

Understanding the sandbox prevents a common error where a user attempts to send a test email to an unverified recipient and receives a rejection.

### Dependency

Depends on:

- A verified SES identity.

### What happens if this step is skipped?

You may attempt to use an unverified recipient while the account is still in the SES sandbox, causing the send operation to fail.

---

## Step 26: Send the Test Email

### Action

After reviewing all email fields, select:

**Send test email**

### Expected Result

The AWS Console should display a successful notification indicating that the test email was sent.

The exact success message may vary depending on the current SES console interface.

### Expected Email Details

~~~text
From: Your verified SES email address
To: Your verified SES email address
Subject: AWS SES Assignment 4 Test Email

This is a test email sent using Amazon SES for Module 8 Assignment 4.
~~~

### Why is this step required?

This is the actual email transmission operation required by Task 2 of the assignment.

The email follows this path:

~~~text
Verified Sender Identity
        |
        v
    Amazon SES
        |
        v
Email Delivery Processing
        |
        v
 Recipient Mail Server
        |
        v
   Your Email Inbox
~~~

### Dependency

Depends on:

- **Step 21:** Email identity verified.
- **Step 24:** Test email correctly configured.
- **Step 25:** Sandbox restrictions understood and satisfied.

### What happens if this step is skipped?

The email identity may be verified, but the assignment remains incomplete because no test email has been sent.

---

# Verification of Amazon SES Test Email

---

## Verification Step 9: Confirm the SES Send Operation Succeeded

### Action

Immediately after selecting **Send test email**, inspect the notification displayed by the AWS Console.

### Expected Output

You should see a success notification indicating that the test email was successfully submitted or sent.

The exact wording may vary with AWS Console updates.

### What should I observe?

You should not see errors related to:

- Unverified sender identity.
- Unverified recipient identity.
- Authorization failure.
- Invalid email address.
- SES sending restrictions.

### Why does this confirm success?

A successful SES console response confirms that Amazon SES accepted the message for delivery.

However, this alone does not prove that the message reached your inbox. Therefore, inbox verification is also required.

---

## Verification Step 10: Check Your Email Inbox

### Action

Open the inbox of the email address used as the SES recipient.

Check the following locations:

- Primary inbox.
- Spam folder.
- Junk folder.
- Promotions folder, if applicable.

### Expected Email

Look for an email with the subject:

`AWS SES Assignment 4 Test Email`

### Expected Message Body

~~~text
This is a test email sent using Amazon SES for Module 8 Assignment 4.
~~~

### What should I observe?

Confirm that:

- The email was received.
- The sender is your verified SES identity.
- The recipient is your own email address.
- The subject matches the configured subject.
- The body contains the expected message.

### Why does this confirm success?

Receiving the email proves that:

1. The SES identity was successfully registered.
2. The email identity was successfully verified.
3. Amazon SES accepted the test message.
4. SES processed the email for delivery.
5. The recipient mail system accepted the email.
6. The email successfully reached your mailbox.

Task 2 of the assignment is now successfully implemented and verified.

---

## Verification Step 11: Final Functional Verification of Both Assignment Tasks

### Action

Confirm the final results of both required tasks.

| Assignment Requirement | Expected Result |
| ---------------------- | --------------- |
| Create a FIFO SQS queue | `xyz-assignment4-queue.fifo` exists |
| Send messages to the FIFO queue | Three test messages were successfully sent |
| Receive messages from the FIFO queue | Messages were successfully retrieved |
| Verify FIFO message order | Message 1 → Message 2 → Message 3 |
| Register email in SES | Email identity was created |
| Verify SES email identity | Status is `Verified` |
| Send test email | SES accepted the test email |
| Receive test email | Email arrived in your own inbox |

### Expected Overall Flow

~~~text
Task 1: Amazon SQS FIFO

Create FIFO Queue
        |
        v
Send Message 1
        |
        v
Send Message 2
        |
        v
Send Message 3
        |
        v
Receive and Verify Messages
        |
        v
Delete Processed Messages
        |
        v
Task 1 Successful


Task 2: Amazon SES

Create Email Identity
        |
        v
Receive Verification Email
        |
        v
Verify Email Address
        |
        v
Send Test Email
        |
        v
Receive Email in Inbox
        |
        v
Task 2 Successful
~~~

### Why does this confirm success?

Every task in the original assignment has now been implemented and tested.

---

# Troubleshooting Common Amazon SQS Issues

---

## Troubleshooting Issue 1: FIFO Queue Name Is Rejected

### Symptom

AWS does not allow you to create the FIFO queue.

### Possible Cause

A FIFO queue name must end with:

`.fifo`

### Correct Queue Name

`xyz-assignment4-queue.fifo`

### Incorrect Example

`xyz-assignment4-queue`

### Resolution

Return to the queue name field and add the required `.fifo` suffix.

---

## Troubleshooting Issue 2: Message Group ID Is Required

### Symptom

The message cannot be sent because the Message group ID field is missing.

### Cause

Every message sent to an SQS FIFO queue requires a Message group ID.

### Resolution

Use:

`xyz-assignment4-group`

Use the same Message group ID for all three test messages when demonstrating ordered processing.

---

## Troubleshooting Issue 3: Message Deduplication ID Is Required

### Symptom

The console requires a Message deduplication ID.

### Cause

Content-based deduplication was not enabled for the FIFO queue.

### Resolution Option 1

Edit the FIFO queue and enable content-based deduplication.

### Resolution Option 2

Provide unique deduplication IDs manually:

| Message | Deduplication ID |
| ------- | ---------------- |
| Message 1 | `message-1` |
| Message 2 | `message-2` |
| Message 3 | `message-3` |

Do not reuse the same deduplication ID for different test messages within the FIFO deduplication interval.

---

## Troubleshooting Issue 4: Only One FIFO Message Appears During Polling

### Symptom

You sent three messages but initially receive only the first message.

### Cause

All three messages belong to the same FIFO Message group ID.

FIFO queues process messages in the same message group sequentially. A later message in that group may remain unavailable while an earlier message is still in flight.

### Resolution

Use this sequence:

~~~text
Poll for messages
    |
    v
Receive Message 1
    |
    v
Verify Message 1
    |
    v
Delete Message 1
    |
    v
Poll again
    |
    v
Receive Message 2
    |
    v
Delete Message 2
    |
    v
Poll again
    |
    v
Receive Message 3
~~~

This behavior is expected and demonstrates ordered processing within the FIFO message group.

---

## Troubleshooting Issue 5: Messages Reappear After Being Received

### Symptom

A previously received message becomes visible again.

### Cause

Receiving an SQS message does not automatically delete it.

During the visibility timeout, the message becomes temporarily invisible. If it is not deleted before becoming visible again, it can be delivered again.

### Resolution

After successfully verifying a message:

1. Select the message.
2. Delete it.
3. Poll for the next message.

---

# Troubleshooting Common Amazon SES Issues

---

## Troubleshooting Issue 6: SES Verification Email Does Not Arrive

### Symptom

You created an email identity, but the verification message is not visible.

### Possible Causes

- Email address was entered incorrectly.
- Verification email is in the spam or junk folder.
- Mail provider delayed delivery.
- Verification request needs to be sent again.

### Resolution

1. Verify that the email address is correct.
2. Check Inbox.
3. Check Spam.
4. Check Junk.
5. Check Promotions, if applicable.
6. Return to the SES identity details page and use the available option to resend the verification email, if presented by the current console interface.

---

## Troubleshooting Issue 7: SES Identity Remains Unverified

### Symptom

The identity status remains:

`Unverified`

or:

`Pending`

### Cause

The verification link has not been successfully opened, or the SES console has not refreshed yet.

### Resolution

1. Open the AWS verification email.
2. Select the verification link.
3. Return to Amazon SES.
4. Refresh the Identities page.
5. Confirm that the status changes to `Verified`.

---

## Troubleshooting Issue 8: Email Address Is Verified in One Region but Missing in Another

### Symptom

You verified an email identity but cannot find it after switching AWS Regions.

### Cause

Amazon SES identities are region-specific.

For example:

~~~text
Verified in:
Asia Pacific (Mumbai)
ap-south-1

Does not automatically mean verified in:
US East (N. Virginia)
us-east-1
~~~

### Resolution

Return to the region where the identity was created.

For this assignment:

`Asia Pacific (Mumbai)`

Region code:

`ap-south-1`

---

## Troubleshooting Issue 9: Email Sending Fails Because the Recipient Is Not Verified

### Symptom

SES rejects the recipient email address.

### Cause

Your SES account may still be operating in the sandbox.

In the sandbox, recipient addresses generally need to be verified, except when using the SES mailbox simulator.

### Resolution

For this assignment, use the same verified email address as:

- Sender.
- Recipient.

This satisfies the requirement to send a test email to yourself and avoids the need for SES production access.

---

## Troubleshooting Issue 10: SES Reports an Authorization Error

### Symptom

You see an access-denied or authorization-related error.

### Cause

The currently authenticated AWS identity may not have sufficient permissions for SES operations.

### Resolution

Ensure that the AWS identity has appropriate permissions to:

- Create an SES email identity.
- View SES identities.
- Send email.
- Delete the SES identity during cleanup.

If you are using an organization-managed AWS account, contact the account administrator for the necessary permissions.

---

## Troubleshooting Issue 11: SES Says the Email Was Sent but It Is Not in the Inbox

### Possible Causes

- The message was delivered to Spam.
- The message was delivered to Junk.
- The recipient email provider delayed processing.
- The recipient email provider filtered the message.

### Resolution

Check:

~~~text
Inbox
→ Spam
→ Junk
→ Promotions
→ All Mail
~~~

Wait briefly and refresh the mailbox if necessary.

Confirm that the recipient email address was entered correctly.

---

# Cleanup and Cost Optimization

After completing all verification steps, delete the resources created for this assignment.

The cleanup sequence is:

~~~text
Complete All Verification
        |
        v
Ensure No SQS Messages Are Needed
        |
        v
Delete SQS FIFO Queue
        |
        v
Delete SES Email Identity
        |
        v
Verify Final Resource State
~~~

The SQS and SES resources in this assignment do not directly depend on each other. Therefore, either can technically be deleted first.

The sequence below removes the SQS queue first and the SES identity second.

---

## Cleanup Step 1: Delete the SQS FIFO Queue

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
→ Select xyz-assignment4-queue.fifo
→ Delete
~~~

### Action

1. Select `xyz-assignment4-queue.fifo`.
2. Select **Delete**.
3. If AWS asks you to type or confirm the queue name, follow the displayed confirmation instructions.
4. Confirm the deletion.

### Why is this required?

The FIFO queue is no longer required after testing is complete.

Deleting it:

- Removes the assignment resource.
- Prevents future unintended use.
- Prevents messages from accumulating.
- Helps keep the AWS account organized.

### Dependency

Before deleting the queue:

- Complete all SQS message testing.
- Verify all required messages.
- Ensure no messages need to be preserved.

The queue has no dependency on the SES identity.

### What happens if this resource is not deleted?

The queue remains in your AWS account.

An idle SQS queue with no requests generally does not itself generate request charges merely by existing, but future API requests, message activity, or related usage can contribute to billable consumption depending on your account's pricing and free-tier eligibility.

Deleting unused lab resources is good operational practice.

---

## Cleanup Step 2: Delete the SES Verified Email Identity

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
→ Select your verified email address
→ Delete
~~~

### Action

1. Select the email identity created for this assignment.
2. Choose **Delete**.
3. Complete the confirmation requested by the AWS Console.
4. Confirm deletion.

### Why is this required?

The verified email identity is no longer required after the SES test is complete.

Deleting unused identities:

- Keeps the SES configuration clean.
- Prevents confusion during future assignments.
- Removes an unnecessary verified sender identity.

### Dependency

Before deleting the identity:

- Ensure the test email was successfully sent.
- Verify that the email arrived in your inbox.

The SES identity does not depend on the SQS queue.

### What happens if this resource is not deleted?

The verified identity remains configured in that AWS Region.

A verified identity by itself does not send emails automatically, but leaving unused identities can:

- Create administrative clutter.
- Cause confusion in future labs.
- Leave an unnecessary sending identity configured in the account.

---

## Cleanup Step 3: Verify the SQS Queue Was Deleted

### Navigation

~~~text
AWS Management Console
→ Amazon SQS
→ Queues
~~~

### Action

Search for:

`xyz-assignment4-queue.fifo`

### Expected Result

The queue should no longer appear in the queue list.

### What should I observe?

The assignment queue should be absent.

### Why does this confirm success?

The missing queue confirms that the SQS resource created for this assignment was successfully deleted.

---

## Cleanup Step 4: Verify the SES Identity Was Deleted

### Navigation

~~~text
AWS Management Console
→ Amazon SES
→ Configuration
→ Identities
~~~

### Action

Look for the email identity used in this assignment.

### Expected Result

The deleted email identity should no longer appear in the SES identity list.

### What should I observe?

Confirm that the assignment email identity is absent.

### Why does this confirm success?

This proves that the SES identity created for the assignment has been removed successfully.

---

## Final Resource Verification

| Resource | Expected Final State |
| -------- | -------------------- |
| SQS FIFO queue `xyz-assignment4-queue.fifo` | `Deleted` |
| Test messages | `Deleted with the queue or individually after processing` |
| SES verified email identity | `Deleted` |
| EC2 instance | `Not created` |
| VPC | `Not created` |
| Subnet | `Not created` |
| Security group | `Not created` |
| IAM user | `Not created` |
| Custom KMS key | `Not created` |

---

## Final Assignment Result

The following assignment requirements have been successfully completed:

~~~text
1. Created an Amazon SQS FIFO queue.
2. Sent multiple test messages to the FIFO queue.
3. Received and verified the messages.
4. Demonstrated ordered message processing within the same FIFO message group.
5. Registered an email address as an Amazon SES identity.
6. Verified ownership of the email address.
7. Sent an Amazon SES test email to the same verified address.
8. Verified successful email delivery in the inbox.
9. Troubleshot common SQS and SES issues.
10. Deleted all resources created specifically for the assignment.
~~~


