# Module 9: Assignment 1 - Serverless Architecture via AWS Lambda and SQS Trigger Integration

## Problem Statement
XYZ Corporation is designing a new web application framework and mandates a serverless strategy to eliminate overhead costs associated with continuously running computing nodes. Implement an AWS Lambda function running a Python runtime environment that triggers dynamically upon receiving decentralized asynchronous task payloads distributed through an Amazon Simple Queue Service (SQS) message stream.

---

## Tasks To Be Performed
1. Create a sample Python-based AWS Lambda function to act as a serverless execution compute block.
2. Establish an Amazon SQS queue trigger mapping to dynamically invoke the Lambda function upon message arrival, and execute validation testing.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Provision the SQS Queue Engine
*(Before mapping a Lambda trigger, the event source queue must exist).*
1. Log in to the **AWS Management Console** and navigate to the **Amazon SQS Dashboard**.
2. Click on the **Create queue** button.
3. Configure the queue criteria settings:
   * **Type:** Select **Standard** (Ensures maximum throughput capabilities).
   * **Name:** `XYZ-Lambda-Trigger-Queue`
4. Leave all configuration sliders and timing metrics at default thresholds. Click **Create queue**.
5. Once active, copy the **Queue ARN** (Amazon Resource Name) string from the summary pane (e.g., `arn:aws:sqs:us-east-1:123456789012:XYZ-Lambda-Trigger-Queue`).

---

### Step 2: Establish the Python-based AWS Lambda Function
1. Search and open the **AWS Lambda Console** from the top services catalog search array.
2. Click on the orange **Create function** button.
3. Configure deployment variables:
   * **Option:** Choose **Author from scratch**.
   * **Function name:** `XYZ-Corporate-Data-Processor`
   * **Runtime:** Select the latest stable version of **Python** (e.g., `Python 3.12`).
4. **Permissions Configuration (CRITICAL):**
   * Expand *Change default execution role* > Select **Create a new role with basic Lambda permissions**.
   * *(Note: We will grant SQS read privileges to this role in the next step to allow the function to poll the messaging block).*
5. Click **Create function**.

---

### Step 3: Grant IAM Permissions to Poll SQS
1. Inside your `XYZ-Corporate-Data-Processor` window interface, open the **Configuration** tab and select **Permissions** from the left sub-panel.
2. Under **Execution role**, click on the hyperlinked role name to launch the **IAM (Identity and Access Management)** dashboard management tab.
3. Inside the IAM Summary screen, click **Add permissions** > **Attach policies**.
4. Search for the managed policy named **`AWSLambdaSQSQueueExecutionRole`**. Check the box next to it.
5. Click **Add permissions at the bottom**. This grants your serverless engine rights to read/delete messages from your SQS instance. Close the IAM window.

---

### Step 4: Map the SQS Trigger and Add Execution Code
1. Return to the **AWS Lambda Console** workspace for `XYZ-Corporate-Data-Processor`.
2. Under the *Function overview* architectural diagram widget block, click on the **+ Add trigger** button.
3. **Trigger configuration:** Choose **SQS** from the dropdown options menu list.
4. **SQS queue:** Select or paste your created resource string target: `XYZ-Lambda-Trigger-Queue`.
5. Keep Batch size as default `10` and click **Add**.
6. **Insert Python Logic:** Scroll down to the **Code** tab section, wipe out the default stub syntax, and insert the following validation workflow script:
   ```python
   import json

   def lambda_handler(event, context):
       print("Serverless Trigger Event Received Successfully!")
       
       # Iterate and parse incoming message payloads from SQS
       for record in event['Records']:
           message_body = record['body']
           print(f"Ingested Message Data Payload: {message_body}")
           
       return {
           'statusCode': 200,
           'body': json.dumps('Serverless SQS Message Processing Complete.')
       }
   ```

7. Click the **Deploy** button to commit your script shifts live onto the AWS microservices framework.

---

### Step 5: Execute Invocations Testing (Task 2 Validation)

1. Re-open the **Amazon SQS Console** in an isolated browser layout and click on `XYZ-Lambda-Trigger-Queue`.
2. Locate the top interactive structural options row and click **Send and receive messages**.
3. Inside the **Message body** workspace framework canvas, type the following text: `{"device_id": "DEV-995", "report_metric": "Normal System Health Trace"}`.
4. Click **Send message**. SQS pushes this object live onto the pipeline.
5. **Verify Invocation Log Outputs:**
* Return to your **AWS Lambda function** page -> click on the **Monitor** tab panel -> click **Logs** or select **View CloudWatch logs**.
* Open the latest log stream record.
* **Result:** You will find your console print messages showing `"Serverless Trigger Event Received Successfully!"` followed directly by your exact customized message JSON payload block printed in line. This proves the serverless execution loops operate properly.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

Even though serverless compute operates on-demand, execute clean-up workflows systematically to remove staging blocks from your enterprise workspace panels:

### 1. Remove the Event Source Trigger and Lambda Compute Block
1. Inside the **AWS Lambda Console > Functions**, select the check group next to `XYZ-Corporate-Data-Processor`.
2. Click on **Actions** > **Delete** > Type `delete` inside the validation module layout window, and click submit.

### 2. Disassemble the Amazon SQS Decoupled Messaging Queue
1. Open the **Amazon SQS Console** directory screen.
2. Highlight your target row matching `XYZ-Lambda-Trigger-Queue`.
3. Locate the functional control headers and click **Delete**.
4. Type `delete` inside the verification safety modal frame and execute absolute destruction.
