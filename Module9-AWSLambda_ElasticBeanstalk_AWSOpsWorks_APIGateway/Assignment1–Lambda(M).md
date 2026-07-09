# Module 9 - AWS Lambda, Elastic Beanstalk, AWS OpsWorks and API Gateway

# Assignment 1 – AWS Lambda with Amazon SQS Trigger

# Part 1: Cost Check, Architecture, Prerequisites, Lambda Function, and SQS Queue Creation

---

## Problem Statement

You work for XYZ Corporation. Your corporation wants to launch a new web-based application, and they do not want their servers to be running all the time. The infrastructure should also be managed by AWS.

Implement a suitable serverless solution using AWS Lambda.

---

## Tasks To Be Performed

1. Create a sample Python Lambda function.
2. Set the Lambda trigger as Amazon SQS and send a message to test invocations.

---

## Solution Overview

This assignment uses the following AWS services:

- **AWS Lambda** — Runs Python code without requiring us to provision or manage servers.
- **Amazon Simple Queue Service (Amazon SQS)** — Stores messages in a queue and triggers the Lambda function when messages become available.
- **AWS Identity and Access Management (IAM)** — Provides the permissions required by the Lambda execution role to access Amazon SQS and write logs.
- **Amazon CloudWatch Logs** — Stores the Lambda function execution logs that we will use to verify successful invocation.

The complete event-driven flow is:

~~~text
User
  |
  | Sends a test message
  v
Amazon SQS Queue
  |
  | Detects a new message
  v
Lambda Event Source Mapping
  |
  | Invokes the function with an SQS event batch
  v
AWS Lambda Function
  |
  | Executes Python code
  v
Amazon CloudWatch Logs
  |
  | Stores execution output
  v
Verification
  |
  v
Cleanup
~~~

---

## Free Tier and Cost Check

AWS pricing and Free Tier benefits can vary based on account creation date, AWS pricing changes, region, and whether promotional AWS credits are available. Therefore, always review the current pricing information shown in your AWS account before creating resources.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| AWS Lambda | Yes, subject to current AWS Free Tier terms | Possibly | Lambda includes a monthly free usage allowance under applicable AWS Free Tier terms. Usage beyond applicable allowances may incur charges. |
| Amazon SQS | Yes, subject to current AWS Free Tier terms | Possibly | SQS provides a monthly free request allowance under applicable terms. Additional requests can incur charges. |
| AWS IAM | No direct charge for standard IAM usage | No direct service charge | The Lambda execution role is required to grant permissions to the function. |
| Amazon CloudWatch Logs | Limited free usage may apply | Possibly | Log ingestion, storage, and related usage beyond applicable allowances can incur charges. |

### Is this assignment completely free?

For a small educational test involving:

- One Lambda function
- One standard SQS queue
- One or a few test messages
- A small amount of CloudWatch logging

the cost is normally negligible and may remain within applicable AWS Free Tier allowances. However, this should not be treated as a guarantee of zero cost because AWS account eligibility, current pricing terms, region, log retention, request volume, and other account-specific factors can affect billing.

### Which resources may consume promotional credits or generate charges?

The following resources can potentially generate billable usage:

- Lambda invocations and compute duration beyond applicable allowances.
- SQS API requests beyond applicable allowances.
- CloudWatch Logs ingestion and storage beyond applicable allowances.

### Should this assignment be completed while AWS promotional credits are available?

Yes. If promotional credits are available, they can provide additional protection against small eligible charges generated while learning and testing AWS services.

### Which resources should be deleted immediately after verification?

After successful verification, delete:

1. The Lambda event source mapping by removing the SQS trigger.
2. The SQS queue.
3. The Lambda function.
4. The Lambda CloudWatch log group, if it is no longer required.
5. The Lambda execution IAM role, if it was created specifically for this assignment and is not used anywhere else.

Cleanup will be performed in reverse dependency order in the final part.

---

## Architecture

~~~text
+-------------------------+
|          User           |
| Sends a message to SQS  |
+------------+------------+
             |
             v
+-------------------------+
|       Amazon SQS        |
|      Standard Queue     |
|   Stores test message   |
+------------+------------+
             |
             | Event source mapping polls SQS
             v
+-------------------------+
|       AWS Lambda        |
|      Python Runtime     |
| Processes SQS event     |
+------------+------------+
             |
             v
+-------------------------+
| Amazon CloudWatch Logs  |
| Invocation verification |
+-------------------------+
~~~

---

## Dependency Flow

The resources must be created and configured in the correct dependency order.

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Create Python Lambda Function
    |
    +----> AWS automatically creates or associates an execution role
    |
    v
Create Amazon SQS Queue
    |
    v
Configure SQS Trigger on Lambda
    |
    +----> Required SQS permissions are added to the Lambda execution role
    |
    v
Send Test Message to SQS
    |
    v
Lambda Is Invoked Automatically
    |
    v
Verify Invocation and CloudWatch Logs
    |
    v
Cleanup in Reverse Dependency Order
~~~

---

## Important Technical Note About the SQS Trigger

When Amazon SQS is configured as a Lambda trigger, Lambda uses an **event source mapping**.

The event source mapping:

1. Polls the SQS queue for messages.
2. Retrieves available messages in batches.
3. Invokes the Lambda function with the messages.
4. Deletes successfully processed messages from the queue.

This means Amazon SQS does not directly push the message to Lambda. Instead, the Lambda service polls the SQS queue through the event source mapping.

---

## Prerequisites

Before starting this assignment, ensure that you have:

| Requirement | Required Value |
| ----------- | -------------- |
| AWS Account | Active AWS account |
| AWS Console Access | Permission to create Lambda, SQS, IAM, and CloudWatch resources |
| AWS Region | Use one consistent region for all resources |
| Programming Language | Python |
| Local Software Installation | Not required |
| EC2 Instance | Not required |
| SSH Connection | Not required |
| Server Management | Not required |

For this assignment, all resources should be created in the same AWS Region.

Example:

- **Region:** `Asia Pacific (Mumbai) - ap-south-1`

Using the same region is essential because the Lambda function and its SQS event source must be in the same AWS Region.

---

# Implementation

## Step 1: Sign In to the AWS Management Console and Select the Region

### Navigation

~~~text
AWS Management Console
→ Sign in
→ Check the Region selector in the upper-right corner
→ Select Asia Pacific (Mumbai) ap-south-1
~~~

### Configuration

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai)` |
| Region Code | `ap-south-1` |

If your course requires a different AWS Region, use that region consistently for every resource in this assignment.

### Why is this step required?

AWS resources are regional unless specifically documented otherwise. The Lambda function and SQS queue used for the event source mapping must be created in the same AWS Region.

### Dependency

This step has no resource dependency. It only requires access to an active AWS account with sufficient permissions.

### What happens if this step is skipped?

If the Lambda function and SQS queue are accidentally created in different regions, the SQS queue will not be available as a compatible event source for that Lambda function.

---

## Step 2: Open the AWS Lambda Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for Lambda
→ Select Lambda
~~~

You should now see the AWS Lambda console.

### Why is this step required?

AWS Lambda is the serverless compute service used in this assignment. It allows Python code to execute in response to events without requiring us to launch or manage an EC2 instance or continuously running server.

### Dependency

This step depends on:

- Successful AWS Console sign-in.
- Correct AWS Region selection.

### What happens if this step is skipped?

The Lambda function cannot be created, so there will be no compute resource available to process messages from the SQS queue.

---

## Step 3: Create the Sample Python Lambda Function

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ Create function
~~~

### Configuration

Select **Author from scratch**.

Configure the function as follows:

| Setting | Value |
| ------- | ----- |
| Function name | `SQS-Lambda-Function` |
| Runtime | A currently supported Python runtime shown in the Lambda console |
| Architecture | `x86_64` |
| Permissions | `Create a new role with basic Lambda permissions` |

For the Python runtime, select a currently supported Python version available in your AWS Lambda console. For example, if `Python 3.13` is available and supported in your account and region, it can be used for this assignment.

Do not enable additional options unless specifically required.

Click:

~~~text
Create function
~~~

Wait until the function is successfully created.

### Expected Result

You should see a success notification similar to:

~~~text
Successfully created the function SQS-Lambda-Function.
~~~

The function page should display:

- Function name: `SQS-Lambda-Function`
- Runtime: The selected Python runtime
- Architecture: `x86_64`
- An automatically created execution role with basic Lambda permissions

### Why is this step required?

This Lambda function is the compute component of the architecture. Later, the SQS queue will be configured as an event source for this function.

When a message becomes available in the SQS queue, the Lambda event source mapping will retrieve it and invoke this Python function.

### Dependency

This step depends on:

- Step 1: Correct AWS Region selection.
- Step 2: Access to the Lambda console.

### What happens if this step is skipped?

There will be no Lambda function available to receive and process SQS messages, so the event-driven architecture cannot be completed.

---

## Step 4: Understand the Automatically Created Lambda Execution Role

When the function is created with:

`Create a new role with basic Lambda permissions`

AWS creates an IAM execution role for the Lambda function.

The role name is typically similar to:

`SQS-Lambda-Function-role-xxxxxxxx`

The random suffix will be different in your AWS account.

### Navigation

From the Lambda function page:

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Permissions
→ Execution role
~~~

You should see the execution role associated with the function.

The basic execution role normally includes permission to write logs to Amazon CloudWatch Logs.

Later, when the SQS trigger is configured through the Lambda console, the execution role must have the required permissions to read and delete messages from the SQS queue and retrieve queue attributes. Depending on the current console workflow and permissions, AWS may add or require the necessary SQS access policy.

### Why is this step required?

A Lambda function cannot independently access AWS resources without authorization.

The execution role provides temporary AWS credentials and permissions that the Lambda function uses during execution.

For this assignment, the function needs permissions for:

- Writing logs to CloudWatch Logs.
- Reading messages from SQS.
- Deleting successfully processed SQS messages.
- Retrieving SQS queue attributes.

### Dependency

This step depends on:

- Step 3: The Lambda function must already exist.

### What happens if this step is skipped?

If the execution role does not have the required permissions, the Lambda service cannot successfully poll and process messages from the SQS queue.

Typical symptoms may include:

- Trigger creation errors.
- Access denied errors.
- Messages remaining unprocessed.
- Missing or incomplete Lambda execution behavior.

---

## Step 5: Replace the Default Lambda Code with a Sample Python Function

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Code
→ Code source
~~~

Locate the default Python file, typically:

`lambda_function.py`

Replace the existing code with:

~~~python
import json

def lambda_handler(event, context):
    print("Lambda function invoked successfully.")
    print("Complete event received:")
    print(json.dumps(event))

    for record in event.get("Records", []):
        message_body = record.get("body", "")
        message_id = record.get("messageId", "Unknown")

        print(f"Message ID: {message_id}")
        print(f"Message Body: {message_body}")

    return {
        "statusCode": 200,
        "body": json.dumps("SQS messages processed successfully.")
    }
~~~

After entering the code, click:

~~~text
Deploy
~~~

Wait for the deployment success notification.

### Code Explanation

#### `import json`

| Code Part | Explanation |
| --------- | ----------- |
| `import` | Python keyword used to load a module. |
| `json` | Python's built-in module for working with JSON data. |

This module is used to convert the Lambda event object and return message into JSON-formatted strings.

---

#### `def lambda_handler(event, context):`

| Code Part | Explanation |
| --------- | ----------- |
| `def` | Python keyword used to define a function. |
| `lambda_handler` | The function name used as the Lambda handler in this example. |
| `event` | Contains event data sent to the Lambda function. For an SQS invocation, this includes one or more SQS records. |
| `context` | Contains runtime information about the current Lambda invocation. |
| `:` | Begins the indented function body in Python. |

---

#### `print("Lambda function invoked successfully.")`

| Code Part | Explanation |
| --------- | ----------- |
| `print()` | Python built-in function that writes output to the execution logs. |
| `"Lambda function invoked successfully."` | The exact text written to CloudWatch Logs when the function executes. |

This line helps confirm that the Lambda function was invoked.

---

#### `print("Complete event received:")`

This line writes a descriptive label to the Lambda logs before displaying the complete incoming event.

---

#### `print(json.dumps(event))`

| Code Part | Explanation |
| --------- | ----------- |
| `print()` | Writes information to the Lambda execution logs. |
| `json` | The imported JSON module. |
| `.dumps()` | Converts a Python object into a JSON-formatted string. |
| `event` | The complete event object received by the Lambda function. |

This is useful for understanding the structure of the SQS event.

---

#### `for record in event.get("Records", []):`

| Code Part | Explanation |
| --------- | ----------- |
| `for` | Begins a Python loop. |
| `record` | Variable representing one SQS record during each loop iteration. |
| `in` | Iterates over the collection on its right side. |
| `event.get()` | Safely retrieves a value from the `event` dictionary. |
| `"Records"` | The key containing the SQS records delivered to Lambda. |
| `[]` | Empty-list fallback used if the `Records` key does not exist. |
| `:` | Begins the loop body. |

Lambda can receive multiple SQS messages in one invocation because event source mappings support batch processing.

---

#### `message_body = record.get("body", "")`

| Code Part | Explanation |
| --------- | ----------- |
| `message_body` | Variable used to store the SQS message body. |
| `=` | Python assignment operator. |
| `record.get()` | Safely retrieves a value from the current SQS record. |
| `"body"` | The SQS record field containing the actual message content. |
| `""` | Empty-string fallback if the field is missing. |

---

#### `message_id = record.get("messageId", "Unknown")`

| Code Part | Explanation |
| --------- | ----------- |
| `message_id` | Variable used to store the unique SQS message identifier. |
| `=` | Assignment operator. |
| `"messageId"` | SQS event field containing the message identifier. |
| `"Unknown"` | Fallback value if the identifier is unavailable. |

---

#### `print(f"Message ID: {message_id}")`

| Code Part | Explanation |
| --------- | ----------- |
| `print()` | Writes output to CloudWatch Logs. |
| `f"..."` | Python formatted string literal, also called an f-string. |
| `{message_id}` | Inserts the current value of the `message_id` variable. |

---

#### `print(f"Message Body: {message_body}")`

This line writes the actual SQS message content to CloudWatch Logs.

The value inside `{message_body}` is replaced with the message body received from Amazon SQS.

---

#### Return Statement

~~~python
return {
    "statusCode": 200,
    "body": json.dumps("SQS messages processed successfully.")
}
~~~

| Code Part | Explanation |
| --------- | ----------- |
| `return` | Ends the function and returns a result. |
| `{}` | Defines a Python dictionary. |
| `"statusCode"` | Dictionary key representing the logical result status. |
| `200` | Conventional HTTP success status code used here as a simple success indicator. |
| `"body"` | Dictionary key containing the response body. |
| `json.dumps()` | Converts the specified Python string into a JSON-formatted string. |

For an SQS-triggered Lambda function, SQS does not use this HTTP-style return object to determine an HTTP response. Successful completion of the Lambda invocation is what matters for standard SQS event source processing.

### Why is this step required?

The Python function provides the actual processing logic.

It:

1. Confirms that the function was invoked.
2. Displays the complete event.
3. Iterates through all SQS records in the event batch.
4. Extracts each message ID.
5. Extracts each message body.
6. Writes the information to CloudWatch Logs.

### Dependency

This step depends on:

- Step 3: The Lambda function must already exist.

### What happens if this step is skipped?

The Lambda function may retain its default sample code, which will not clearly demonstrate SQS message processing or log the message body in the structured way required for this assignment verification.

---

## Step 6: Create the Amazon SQS Queue

### Navigation

~~~text
AWS Console
→ Search bar
→ Search for SQS
→ Select Simple Queue Service
→ Queues
→ Create queue
~~~

### Configuration

Select:

- **Queue type:** `Standard`

Configure the queue as follows:

| Setting | Value |
| ------- | ----- |
| Type | `Standard` |
| Name | `lambda-sqs-queue` |
| Visibility timeout | Keep the default value for this basic lab unless your Lambda timeout requires a higher value |
| Message retention period | Keep default |
| Delivery delay | Keep default |
| Maximum message size | Keep default |
| Receive message wait time | Keep default |

For this basic assignment, a standard queue is sufficient because the task does not require strict FIFO ordering or exactly-once message processing semantics.

Leave the remaining settings at their defaults unless your course specifically requires different values.

Click:

~~~text
Create queue
~~~

### Expected Result

The SQS console should display the newly created queue:

`lambda-sqs-queue`

Its status should indicate that the queue is available for use.

### Why is this step required?

Amazon SQS acts as a message buffer between message producers and the Lambda function.

Instead of requiring a server to run continuously and wait for work:

1. A message is sent to SQS.
2. The message remains safely stored in the queue.
3. The Lambda event source mapping polls the queue.
4. Lambda is invoked when messages are available.
5. Successfully processed messages are removed from the queue.

This architecture is asynchronous, event-driven, and serverless.

### Dependency

This step depends on:

- Step 1: Correct AWS Region selection.

It does not depend on the Lambda function for creation, but both resources are required before configuring the SQS trigger.

### What happens if this step is skipped?

There will be no message queue to:

- Store the test message.
- Act as the Lambda event source.
- Trigger the Lambda function automatically.

Therefore, Task 2 of the assignment cannot be completed.

---

## Current Resource State After Part 1

At this point, the following resources should exist:

| Resource | Expected State |
| -------- | -------------- |
| Lambda function `SQS-Lambda-Function` | `Active` |
| Python Lambda code | `Deployed` |
| Lambda execution role | `Created and attached` |
| SQS queue `lambda-sqs-queue` | `Available` |
| SQS trigger | `Not configured yet` |
| Test SQS message | `Not sent yet` |
| Lambda invocation verification | `Pending` |

The next part will configure the SQS queue as the Lambda trigger, ensure the execution role has the required SQS permissions, send a test message, and verify automatic invocation.

----

# Part 2: Configure the SQS Trigger, Send a Test Message, and Verify Lambda Invocation

---

## Step 7: Configure the SQS Queue as a Lambda Trigger

Now that both the Lambda function and the SQS queue exist, configure the queue as an event source for the Lambda function.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Function overview
→ Add trigger
~~~

### Configuration

On the **Add trigger** page, configure the following:

| Setting | Value |
| ------- | ----- |
| Trigger source | `SQS` |
| SQS queue | `lambda-sqs-queue` |
| Batch size | `10` |
| Batch window | `0` seconds |
| Activate trigger | Enabled |

If additional optional settings are displayed, leave them at their default values for this assignment.

After reviewing the configuration, click:

~~~text
Add
~~~

### Expected Result

After the trigger is successfully created, the Lambda function overview should show:

~~~text
lambda-sqs-queue
        |
        v
SQS-Lambda-Function
~~~

The SQS trigger should also appear under:

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
~~~

The trigger status should be enabled.

### Why is this step required?

Creating the SQS queue alone does not automatically connect it to the Lambda function.

The SQS trigger creates a Lambda **event source mapping**.

The event source mapping performs the following operations:

1. Polls the SQS queue.
2. Retrieves available messages.
3. Groups messages into batches when applicable.
4. Invokes the Lambda function.
5. Sends the retrieved SQS records inside the Lambda `event` object.
6. Deletes successfully processed messages from the queue.

The event flow is:

~~~text
Message Producer
        |
        | Send message
        v
Amazon SQS Queue
        |
        | Lambda event source mapping polls the queue
        v
Lambda Function
        |
        | Python code processes the message
        v
Successful Invocation
        |
        v
Message removed from SQS
~~~

### Dependency

This step depends on:

- Step 3: The Lambda function `SQS-Lambda-Function` must exist.
- Step 6: The SQS queue `lambda-sqs-queue` must exist.
- Both resources must be located in the same AWS Region.

### What happens if this step is skipped?

The SQS queue and Lambda function will remain independent resources.

Sending a message to the queue will not automatically invoke the Lambda function because no event source mapping will exist between them.

---

## Step 8: Verify the SQS Trigger Status

After adding the trigger, verify that the event source mapping is enabled.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
~~~

Select the SQS trigger if necessary.

### What should you observe?

Verify the following:

| Property | Expected Value |
| -------- | -------------- |
| Trigger source | `SQS` |
| Queue | `lambda-sqs-queue` |
| Trigger status | `Enabled` |
| Batch size | `10` |

The exact layout or wording may vary slightly as the AWS Management Console is updated.

### Why is this step required?

An event source mapping can exist but be disabled.

If it is disabled:

- Lambda will not poll the SQS queue.
- Messages will remain in the queue.
- The Lambda function will not be automatically invoked.

### Dependency

This step depends on:

- Step 7: The SQS trigger must already have been added.

### What happens if this step is skipped?

The assignment might appear correctly configured even though the trigger is disabled or improperly configured. This could cause the test message to remain unprocessed.

---

## Step 9: Verify the Lambda Execution Role Has Required SQS Permissions

The Lambda execution role must have permission to interact with the SQS queue.

At minimum, SQS event source processing commonly requires permissions including:

- `sqs:ReceiveMessage`
- `sqs:DeleteMessage`
- `sqs:GetQueueAttributes`

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Permissions
→ Execution role
→ Click the role name
~~~

This opens the IAM role associated with the Lambda function.

The role name will typically look similar to:

`SQS-Lambda-Function-role-xxxxxxxx`

The random suffix will be different in your AWS account.

### Configuration Check

Under the role's **Permissions policies**, look for policies that provide:

1. Basic CloudWatch Logs permissions.
2. Required SQS access permissions.

The basic Lambda execution policy is typically similar to:

`AWSLambdaBasicExecutionRole-xxxxxxxx`

Depending on the AWS Console workflow and your permissions, the required SQS access may have been added automatically when the trigger was configured.

If the SQS trigger was added successfully and its status is `Enabled`, continue to Step 10.

If trigger creation failed because the execution role does not have permission to access SQS, add the required AWS-managed policy as described below.

---

## Step 9A: Add SQS Execution Permissions Only If Required

> **Important:** Perform this step only if the SQS trigger could not be created because the Lambda execution role lacks the necessary SQS permissions.

### Navigation

~~~text
AWS Console
→ IAM
→ Roles
→ Select the Lambda execution role
→ Add permissions
→ Attach policies
~~~

Search for:

`AWSLambdaSQSQueueExecutionRole`

Select the AWS-managed policy and click:

~~~text
Add permissions
~~~

### What does this policy provide?

The `AWSLambdaSQSQueueExecutionRole` AWS-managed policy provides permissions required for a Lambda function to read messages from an SQS queue and write execution logs.

The relevant permissions include:

~~~json
{
  "Effect": "Allow",
  "Action": [
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage",
    "sqs:GetQueueAttributes"
  ],
  "Resource": "*"
}
~~~

It also provides CloudWatch Logs permissions required for Lambda logging.

### JSON Policy Explanation

#### `"Effect": "Allow"`

| JSON Part | Explanation |
| --------- | ----------- |
| `"Effect"` | Specifies whether the policy statement allows or denies access. |
| `"Allow"` | Grants permission for the specified actions. |

#### `"Action"`

The `Action` array defines which AWS API operations are permitted.

| Permission | Purpose |
| ---------- | ------- |
| `sqs:ReceiveMessage` | Allows messages to be retrieved from the SQS queue. |
| `sqs:DeleteMessage` | Allows successfully processed messages to be deleted from the queue. |
| `sqs:GetQueueAttributes` | Allows Lambda to retrieve information required for processing the queue. |

#### `"Resource": "*"`

| JSON Part | Explanation |
| --------- | ----------- |
| `"Resource"` | Specifies which AWS resources the permissions apply to. |
| `"*"` | Wildcard representing all applicable resources. |

For production environments, a least-privilege customer-managed policy restricted to the specific queue ARN is preferable. For this basic lab, the AWS-managed `AWSLambdaSQSQueueExecutionRole` policy provides the standard permissions required for SQS event source processing.

### Why is this step required?

Lambda requires authorization to retrieve and delete messages from the SQS queue.

Without these permissions, the event source mapping cannot successfully process messages.

### Dependency

This step depends on:

- The Lambda execution role created in Step 3.
- The SQS queue created in Step 6.

### What happens if this step is skipped when permissions are missing?

Possible symptoms include:

- Failure while adding the SQS trigger.
- Access denied errors.
- Messages remaining in the queue.
- Lambda not being invoked by SQS.

---

## Step 10: Send a Test Message to the SQS Queue

Now send a message to the SQS queue.

This message should automatically trigger the Lambda function.

### Navigation

~~~text
AWS Console
→ SQS
→ Queues
→ lambda-sqs-queue
→ Send and receive messages
~~~

### Configuration

In the **Message body** field, enter:

`Hello from Amazon SQS! This message should invoke the Lambda function.`

Leave optional message attributes empty unless required by your course.

Click:

~~~text
Send message
~~~

### Expected Result

The SQS console should display a success notification indicating that the message was sent successfully.

A typical success indication is similar to:

~~~text
Message sent successfully.
~~~

### Important Behavior

Because the SQS trigger is enabled, Lambda may process the message very quickly.

Therefore, after sending the message:

- The message might disappear from the queue almost immediately.
- The number of available messages may return to `0`.
- This behavior is expected when Lambda successfully processes the message.

### Why is this step required?

The assignment explicitly requires sending an SQS message to test Lambda invocation.

The test message acts as the event that starts the following sequence:

~~~text
Send Message
    |
    v
Message Stored in SQS
    |
    v
Lambda Event Source Mapping Retrieves Message
    |
    v
Lambda Function Invoked
    |
    v
Python Code Processes Message
    |
    v
Execution Output Written to CloudWatch Logs
    |
    v
Successfully Processed Message Deleted from SQS
~~~

### Dependency

This step depends on:

- Step 6: The SQS queue must exist.
- Step 7: The SQS trigger must be configured.
- Step 8: The trigger must be enabled.
- Step 9: The Lambda execution role must have the required permissions.

### What happens if this step is skipped?

No SQS message will be available to invoke the Lambda function, so automatic invocation cannot be tested or verified.

---

# Verification

## Verification Step 1: Verify the Lambda Invocation Count

### Action

After sending the SQS message, return to the Lambda function and inspect its monitoring information.

Wait approximately a few seconds for AWS metrics and logs to become available.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Monitor
~~~

Review the invocation metrics.

### Expected Output

The **Invocations** metric should eventually show at least one invocation.

Example:

~~~text
Invocations: 1 or more
~~~

The exact value may be higher than `1` if:

- You sent multiple messages.
- You manually tested the Lambda function.
- A message was retried.

### What should I observe?

Look for:

| Metric | Expected Observation |
| ------ | -------------------- |
| Invocations | At least one invocation after sending the message |
| Errors | Ideally `0` |
| Duration | A successful execution duration should be recorded |

CloudWatch metrics may take a short time to appear.

### Why does this confirm success?

An increase in the invocation count after sending the SQS message indicates that the Lambda function executed.

However, the strongest verification is to inspect the CloudWatch Logs and confirm that the exact SQS message body was processed.

---

## Verification Step 2: Open the Lambda CloudWatch Logs

### Action

Open the logs generated by the Lambda function.

### Navigation

From the Lambda function:

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Monitor
→ View CloudWatch logs
~~~

Alternatively:

~~~text
AWS Console
→ CloudWatch
→ Logs
→ Log groups
→ /aws/lambda/SQS-Lambda-Function
~~~

Select the most recent log stream.

The log stream name will contain automatically generated date and runtime information.

### Expected Output

The exact output will contain AWS-generated metadata and a complete SQS event.

Look for output similar to:

~~~text
START RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Version: $LATEST

Lambda function invoked successfully.

Complete event received:

{"Records": [{"messageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "body": "Hello from Amazon SQS! This message should invoke the Lambda function.", ...}]}

Message ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

Message Body: Hello from Amazon SQS! This message should invoke the Lambda function.

END RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

REPORT RequestId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Duration: XX.XX ms
Billed Duration: XX ms
Memory Size: XXX MB
Max Memory Used: XX MB
~~~

The exact values for the following fields will be different in your account:

- Request ID
- Message ID
- Duration
- Billed duration
- Memory usage
- Log stream name
- Event metadata

### What should I observe?

Confirm that the logs contain:

1. `Lambda function invoked successfully.`
2. `Complete event received:`
3. An SQS record inside the `Records` array.
4. A valid `messageId`.
5. The exact message body:

`Hello from Amazon SQS! This message should invoke the Lambda function.`

6. An `END RequestId` entry.
7. A `REPORT RequestId` entry.

### Why does this confirm success?

This confirms the complete event-driven flow:

~~~text
Test message sent to SQS
        |
        v
SQS accepted the message
        |
        v
Lambda event source mapping retrieved it
        |
        v
Lambda function was invoked
        |
        v
Python handler received the SQS event
        |
        v
Python code extracted the message body
        |
        v
Output was written to CloudWatch Logs
~~~

Seeing the exact SQS message body in the Lambda execution logs proves that the Lambda function successfully received and processed the SQS event.

---

## Verification Step 3: Verify the Message Was Successfully Consumed

### Action

Return to the SQS queue after successful Lambda execution.

### Navigation

~~~text
AWS Console
→ SQS
→ Queues
→ lambda-sqs-queue
→ Monitoring
~~~

You may also return to:

~~~text
AWS Console
→ SQS
→ Queues
→ lambda-sqs-queue
→ Send and receive messages
~~~

### Expected Output

After successful processing, the number of available messages should normally return to:

~~~text
0
~~~

The message should no longer remain available for normal retrieval.

### What should I observe?

| Property | Expected Observation |
| -------- | -------------------- |
| Available messages | Normally `0` after successful processing |
| Lambda logs | Exact test message is visible |
| Lambda errors | Ideally `0` |
| Trigger | `Enabled` |

SQS metrics are eventually consistent, so console counts may not update immediately.

### Why does this confirm success?

For a standard successful SQS-triggered Lambda invocation:

1. Lambda retrieves the message.
2. The function processes it without raising an error.
3. The event source mapping deletes the successfully processed message from the queue.

Therefore, the combination of:

- Successful Lambda logs.
- Exact message body in those logs.
- No function error.
- Message no longer available in the queue.

provides strong evidence that the complete integration is working correctly.

---

## Verification Step 4: Verify All Assignment Requirements

### Action

Compare the implemented resources and behavior against the original assignment tasks.

| Assignment Requirement | Implementation | Verification Result |
| ---------------------- | -------------- | ------------------- |
| Create a sample Python Lambda function | Created `SQS-Lambda-Function` with Python code | Confirmed through Lambda console and deployed code |
| Set Lambda trigger as SQS | Configured `lambda-sqs-queue` as the Lambda event source | Trigger status should be `Enabled` |
| Send a message | Sent a test message through the SQS console | SQS success notification displayed |
| Test invocation | Lambda automatically processed the SQS message | Confirmed through invocation metrics and CloudWatch Logs |

### What should I observe?

Every original task should now have a corresponding successful implementation and verification result.

### Why does this confirm success?

The assignment requires a Python Lambda function and an SQS trigger tested by sending a message.

The implementation satisfies both requirements and verifies the complete event flow from SQS to Lambda.

---

# Troubleshooting

## Problem 1: SQS Trigger Cannot Be Added

### Possible Cause

The Lambda execution role may not have the required SQS permissions.

### Resolution

Verify that the Lambda execution role has the required permissions, including:

- `sqs:ReceiveMessage`
- `sqs:DeleteMessage`
- `sqs:GetQueueAttributes`

If required for this lab, attach:

`AWSLambdaSQSQueueExecutionRole`

Then retry adding the trigger.

---

## Problem 2: Message Remains in the Queue

### Possible Causes

- The SQS trigger is disabled.
- The Lambda execution role lacks SQS permissions.
- The Lambda function is failing.
- The event source mapping is not configured correctly.

### Resolution

Check:

~~~text
Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
~~~

Confirm that the SQS trigger is enabled.

Then inspect:

~~~text
Lambda
→ Functions
→ SQS-Lambda-Function
→ Monitor
→ View CloudWatch logs
~~~

Look for errors.

---

## Problem 3: Lambda Was Invoked but the Message Returns to the Queue

### Possible Cause

The Lambda function may be failing during processing.

When processing fails, the message becomes visible again after the queue's visibility timeout expires and may be retried.

### Resolution

Inspect the most recent CloudWatch log stream and look for:

- Python exceptions.
- Syntax errors.
- Runtime errors.
- Permission errors.

Correct the function code and deploy it again before retesting.

---

## Problem 4: No CloudWatch Log Group Appears

### Possible Causes

- The Lambda function has not been invoked yet.
- The execution role lacks CloudWatch Logs permissions.
- Logs have not yet propagated to the console.

### Resolution

Verify that the execution role includes basic Lambda logging permissions and confirm that the function has been invoked.

The log group should normally be:

`/aws/lambda/SQS-Lambda-Function`

---

## Current Resource State After Part 2

At this point, the following resources and configurations should exist:

| Resource or Configuration | Expected State |
| ------------------------- | -------------- |
| Lambda function `SQS-Lambda-Function` | `Active` |
| Python Lambda code | `Deployed` |
| Lambda execution role | `Attached` |
| Required SQS permissions | `Available` |
| SQS queue `lambda-sqs-queue` | `Available` |
| SQS trigger | `Enabled` |
| Test SQS message | `Successfully processed` |
| Lambda invocation | `Successful` |
| CloudWatch Logs | `Message processing output visible` |
| Cleanup | `Pending` |

All implementation and verification tasks are now complete.

The final part will remove every resource created during this assignment in reverse dependency order to minimize unnecessary charges and AWS promotional credit usage.

----

# Part 3: Cleanup Resources in Reverse Dependency Order and Verify Complete Removal

---

# Cleanup and Cost Optimization

All implementation and verification tasks have been completed successfully.

The resources created for this assignment should now be removed to:

- Prevent unnecessary AWS usage.
- Minimize AWS promotional credit consumption.
- Avoid unnecessary CloudWatch Logs storage.
- Remove unused IAM permissions.
- Keep the AWS account clean and organized.

The cleanup must be performed in **reverse dependency order**.

---

## Cleanup Dependency Flow

The resources were created and connected in the following general order:

~~~text
Lambda Function
    |
    v
Lambda Execution Role
    |
    v
SQS Queue
    |
    v
SQS Event Source Mapping / Trigger
    |
    v
CloudWatch Log Group Created After Invocation
~~~

For cleanup, remove dependent integrations before deleting the underlying resources:

~~~text
Disable and Delete SQS Trigger
        |
        v
Delete SQS Queue
        |
        v
Delete Lambda Function
        |
        v
Delete CloudWatch Log Group
        |
        v
Delete Assignment-Specific IAM Role
        |
        v
Verify Complete Cleanup
~~~

> **Important:** The CloudWatch log group is not technically required to be deleted before or after the Lambda function because it is an independent logging resource once created. However, deleting it after the Lambda function prevents the function from generating additional logs during cleanup.

---

## Cleanup Step 1: Disable the SQS Trigger

Before deleting the event source mapping, first disable it to stop Lambda from polling the SQS queue during cleanup.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
→ Select the SQS trigger
~~~

Depending on the current AWS Console layout, the trigger details may provide an option to disable the event source mapping directly. You may also see the event source mapping under the Lambda function's SQS trigger configuration.

### Action

Disable the SQS event source mapping if the console provides a separate **Disable** option.

Confirm that its status changes from:

`Enabled`

to:

`Disabled`

If the current AWS Console allows direct deletion of the trigger without requiring a separate disable operation, proceed directly to Cleanup Step 2.

### Why is this required?

Disabling the trigger stops the Lambda event source mapping from polling the SQS queue for new messages.

This is useful during cleanup because it prevents additional Lambda invocations while resources are being removed.

### Dependency

This step depends on:

- The Lambda function `SQS-Lambda-Function`.
- The SQS trigger created in Step 7.
- The SQS queue `lambda-sqs-queue`.

### What happens if this resource is not disabled?

If the trigger remains enabled while the SQS queue still exists:

- Lambda can continue polling the queue.
- New messages can invoke the Lambda function.
- Additional Lambda invocations and CloudWatch logs may be generated.

However, if the trigger is immediately deleted in the next step, a separate disable operation is not mandatory.

---

## Cleanup Step 2: Delete the SQS Trigger

The SQS trigger represents the Lambda event source mapping that connects the SQS queue to the Lambda function.

Delete this integration before deleting the underlying resources.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
→ Select the SQS trigger
→ Delete
~~~

Depending on the current AWS Console layout, you may instead select the SQS trigger from the function overview or event source mapping details and choose **Delete**.

### Action

1. Select the trigger associated with `lambda-sqs-queue`.
2. Choose **Delete**.
3. Confirm deletion if prompted.
4. Wait until the event source mapping is removed.

### Expected Result

The SQS trigger should no longer appear under:

~~~text
AWS Console
→ Lambda
→ Functions
→ SQS-Lambda-Function
→ Configuration
→ Triggers
~~~

### Why is this required?

The SQS trigger creates an event source mapping between:

- The SQS queue.
- The Lambda function.

Removing this mapping disconnects the queue from Lambda and stops automatic polling and invocation.

### Dependency

This cleanup step must be performed before deleting the SQS queue or Lambda function when following a clean reverse-dependency workflow.

### What happens if this resource is not deleted?

If the event source mapping remains enabled:

- Lambda continues polling the SQS queue.
- New messages can trigger additional Lambda invocations.
- Additional logs may be generated.
- Resource cleanup becomes less orderly.

Deleting the Lambda function may also remove its associated event source mapping, but explicitly deleting the trigger first provides clearer cleanup verification.

---

## Cleanup Step 3: Delete the SQS Queue

Now delete the SQS queue used to store and deliver the test message.

### Navigation

~~~text
AWS Console
→ SQS
→ Queues
→ Select lambda-sqs-queue
→ Delete
~~~

### Action

1. Select `lambda-sqs-queue`.
2. Click **Delete**.
3. If AWS requests confirmation, enter the required queue name or confirmation value exactly as displayed.
4. Confirm the deletion.

### Expected Result

The queue should disappear from the SQS queue list.

The following queue should no longer exist:

`lambda-sqs-queue`

### Why is this required?

The SQS queue is no longer required after the assignment has been successfully tested.

Deleting it:

- Removes the unused message queue.
- Prevents additional SQS API usage associated with the queue.
- Keeps the AWS account clean.

### Dependency

Delete the SQS event source mapping before deleting this queue.

The correct order is:

~~~text
Delete SQS Trigger
        |
        v
Delete SQS Queue
~~~

### What happens if this resource is not deleted?

An unused queue may remain in the AWS account indefinitely.

Although an idle queue with no requests generally produces little or no meaningful cost by itself, future API requests and associated usage can potentially contribute to billable usage. Unused resources also make account management more difficult.

---

## Cleanup Step 4: Delete the Lambda Function

After removing the SQS trigger and queue, delete the Lambda function.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ Select SQS-Lambda-Function
→ Actions
→ Delete
~~~

Depending on the current AWS Console interface, the **Delete** option may also be directly available from the function details page.

### Action

1. Select `SQS-Lambda-Function`.
2. Choose **Delete**.
3. Enter any requested confirmation value if prompted.
4. Confirm deletion.

### Expected Result

The following function should no longer appear in the Lambda function list:

`SQS-Lambda-Function`

### Why is this required?

The Lambda function is no longer required after successful assignment verification.

Deleting the function:

- Removes unused function code and configuration.
- Prevents accidental future invocation.
- Keeps the AWS account organized.

### Dependency

The SQS trigger should be deleted before deleting the Lambda function.

The recommended cleanup order is:

~~~text
SQS Trigger
    |
    v
SQS Queue
    |
    v
Lambda Function
~~~

### What happens if this resource is not deleted?

An idle Lambda function generally does not incur compute charges merely because it exists, but it could generate charges if invoked in the future.

Keeping unused functions can also:

- Create configuration clutter.
- Increase the risk of accidental invocation.
- Leave outdated code deployed unnecessarily.

---

## Cleanup Step 5: Delete the Lambda CloudWatch Log Group

Deleting a Lambda function does not necessarily delete its existing CloudWatch log group automatically.

The log group created for this assignment is:

`/aws/lambda/SQS-Lambda-Function`

### Navigation

~~~text
AWS Console
→ CloudWatch
→ Logs
→ Log groups
→ Select /aws/lambda/SQS-Lambda-Function
→ Actions
→ Delete log group
~~~

Depending on the current AWS Console layout, the delete option may be displayed directly after selecting the log group.

### Action

1. Find `/aws/lambda/SQS-Lambda-Function`.
2. Select the log group.
3. Choose **Delete log group**.
4. Confirm deletion.

### Expected Result

The log group should disappear from the CloudWatch Logs console.

### Why is this required?

CloudWatch log groups can remain after their associated Lambda functions are deleted.

Stored logs can contribute to CloudWatch Logs storage usage.

Deleting the log group:

- Removes test execution logs.
- Prevents unnecessary long-term log retention.
- Helps minimize AWS usage and promotional credit consumption.

### Dependency

Delete the Lambda function before deleting its log group.

This order prevents the Lambda function from being invoked again and recreating or writing additional data to the log group during cleanup.

### What happens if this resource is not deleted?

The log group can remain in the AWS account and continue storing previously generated log data according to its configured retention policy.

If the retention policy is set to never expire, the logs can remain indefinitely unless manually deleted.

---

## Cleanup Step 6: Delete the Assignment-Specific Lambda Execution IAM Role

The Lambda function was created with:

`Create a new role with basic Lambda permissions`

Therefore, AWS created an IAM execution role specifically for the function.

The role name will typically look similar to:

`SQS-Lambda-Function-role-xxxxxxxx`

The random suffix will be different in your AWS account.

> **Important:** Delete this role only if it was created specifically for this assignment and is not being used by any other AWS resource.

### Navigation

~~~text
AWS Console
→ IAM
→ Roles
→ Search for SQS-Lambda-Function-role
→ Select the assignment-specific role
→ Delete
~~~

### Action

1. Search for the Lambda execution role created for this assignment.
2. Open the role.
3. Confirm that it is not used by another Lambda function or AWS resource.
4. Choose **Delete**.
5. Enter the required confirmation value if prompted.
6. Confirm deletion.

### Expected Result

The assignment-specific IAM role should disappear from the IAM role list.

### Why is this required?

The execution role grants permissions to AWS services.

For this assignment, it may include permissions for:

- Writing Lambda logs to CloudWatch Logs.
- Receiving messages from SQS.
- Deleting processed SQS messages.
- Retrieving SQS queue attributes.

Deleting an unused assignment-specific role follows the principle of least privilege and reduces unnecessary IAM configuration.

### Dependency

Delete the Lambda function before deleting its execution role.

The correct order is:

~~~text
Lambda Function
        |
        v
Lambda Execution Role
~~~

### What happens if this resource is not deleted?

IAM roles do not normally incur direct charges merely for existing.

However, unused IAM roles can:

- Create unnecessary security exposure.
- Cause account clutter.
- Make permission auditing more difficult.
- Leave permissions available when they are no longer required.

---

# Final Cleanup Verification

After completing all cleanup steps, verify that every resource created during the assignment has been removed.

---

## Verification Step 5: Verify the SQS Queue Has Been Deleted

### Action

Open the SQS queue list and search for the assignment queue.

### Navigation

~~~text
AWS Console
→ SQS
→ Queues
→ Search for lambda-sqs-queue
~~~

### Expected Output

The queue should not appear in the queue list.

### What should I observe?

`lambda-sqs-queue` should no longer exist.

### Why does this confirm success?

The absence of the queue confirms that the SQS resource created for the assignment has been successfully removed.

---

## Verification Step 6: Verify the Lambda Function Has Been Deleted

### Action

Open the Lambda function list and search for the assignment function.

### Navigation

~~~text
AWS Console
→ Lambda
→ Functions
→ Search for SQS-Lambda-Function
~~~

### Expected Output

The function should not appear in the function list.

### What should I observe?

`SQS-Lambda-Function` should no longer exist.

### Why does this confirm success?

The absence of the function confirms that the serverless compute resource created for the assignment has been successfully deleted.

---

## Verification Step 7: Verify the CloudWatch Log Group Has Been Deleted

### Action

Search the CloudWatch log groups for the Lambda function's log group.

### Navigation

~~~text
AWS Console
→ CloudWatch
→ Logs
→ Log groups
→ Search for /aws/lambda/SQS-Lambda-Function
~~~

### Expected Output

The log group should not appear.

### What should I observe?

`/aws/lambda/SQS-Lambda-Function` should no longer exist.

### Why does this confirm success?

The absence of the log group confirms that the stored Lambda execution logs have been removed.

---

## Verification Step 8: Verify the Assignment-Specific IAM Role Has Been Deleted

### Action

Search the IAM roles list for the Lambda execution role.

### Navigation

~~~text
AWS Console
→ IAM
→ Roles
→ Search for SQS-Lambda-Function-role
~~~

### Expected Output

The assignment-specific role should not appear.

### What should I observe?

The exact automatically generated role associated with `SQS-Lambda-Function` should no longer exist.

### Why does this confirm success?

The absence of the role confirms that the temporary permissions created specifically for this assignment have been removed.

---

# Final Resource Verification Table

| Resource | Resource Name | Expected Final State |
| -------- | ------------- | -------------------- |
| Lambda SQS trigger | Event source mapping for `lambda-sqs-queue` | `Deleted` |
| Amazon SQS queue | `lambda-sqs-queue` | `Deleted` |
| AWS Lambda function | `SQS-Lambda-Function` | `Deleted` |
| CloudWatch log group | `/aws/lambda/SQS-Lambda-Function` | `Deleted` |
| Lambda execution IAM role | `SQS-Lambda-Function-role-xxxxxxxx` | `Deleted` |

---

# Final Assignment Result

The complete assignment has now been implemented, tested, verified, and cleaned up.

The completed architecture successfully demonstrated the following event-driven serverless workflow:

~~~text
User sends message
        |
        v
Amazon SQS stores the message
        |
        v
Lambda event source mapping polls SQS
        |
        v
AWS Lambda is automatically invoked
        |
        v
Python function processes the SQS message
        |
        v
Execution details are written to CloudWatch Logs
        |
        v
Successful message is removed from the SQS queue
~~~

The assignment requirements were successfully fulfilled:

| Task | Result |
| ---- | ------ |
| Create a sample Python Lambda function | `Completed` |
| Set Amazon SQS as the Lambda trigger | `Completed` |
| Send an SQS test message | `Completed` |
| Test automatic Lambda invocation | `Completed` |
| Verify the message through CloudWatch Logs | `Completed` |
| Delete all assignment resources | `Completed` |

> **Assignment complete. No additional parts are required.**
