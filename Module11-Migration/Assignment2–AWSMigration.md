# Module 11: Assignment 2 - VM-to-EC2 Cloud Migration via AWS Application Migration Tools

## Problem Statement
XYZ Corporation is advancing its cloud adoption phase by migrating on-premises server workloads directly into the AWS ecosystems without rebuilding the application structures from scratch. Implement an infrastructure migration lifecycle by exporting the locally hosted Ubuntu Server virtual machine container asset, storing the serialized disk image bundle inside a secure Amazon S3 staging bucket, executing a VM Import process to generate a custom Amazon Machine Image (AMI), and launching a live EC2 production instance using that imported machine template.

---

## Tasks To Be Performed
1. Export the locally structured Ubuntu Server Virtual Machine (from Assignment 1) as an Open Virtualization Format (OVA/OVF) package image file.
2. Provision an Amazon S3 staging bucket and upload the exported virtual image binary payload into it.
3. Establish required IAM service roles (`vmimport`) and utilize the AWS CLI environment to transform the disk file into a custom AMI.
4. Launch and test an Amazon EC2 Instance generated directly from the newly migrated custom image.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Export the On-Premises Virtual Machine
1. Open **Oracle VirtualBox Manager** on your desktop machine and ensure `XYZ-OnPrem-Source-Server` is completely powered off.
2. Highlight the VM, click on the top **File** option menu header, and select **Export Appliance**.
3. Configure the export wizard preferences:
   * **Format:** Select **Open Virtualization Format 1.0 (OVA)**.
   * **File Location:** Choose your local storage destination paths and name the file package **`ubuntu-migration-source.ova`**.
4. Click **Export** and wait for the compression process to fully finalize on your local disc engine.

---

### Step 2: Establish the S3 Staging Bucket and Upload Image Payload
1. Log in to your **AWS Management Console** and navigate to the **Amazon S3 Dashboard**.
2. Click **Create bucket**. Name the bucket uniquely (e.g., `xyz-migration-staging-bucket-2026`) and set its active region. Keep all standard default parameters active and click **Create bucket**.
3. Open your newly created bucket dashboard layout path, click **Upload**, select the local compiled file asset **`ubuntu-migration-source.ova`**, and click submit. *(Note: Given file capacity limitations, wait patiently for the progress tracker execution to register complete visualization metrics).*

---

### Step 3: Configure IAM vmimport Service Roles and Metadata Files
1. Open a local text editor and create a file configuration manifest mapping the target permission bindings named **`trust-policy.json`**:
   ```json
   {
      "Version": "2012-10-17",
      "Statement": [
         {
            "Sid": "",
            "Effect": "Allow",
            "Principal": { "Service": "vmie.amazonaws.com" },
            "Action": "sts:AssumeRole",
            "Condition": {
               "StringEquals": { "sts:ExternalId": "vmimport" }
            }
         }
      ]
   }
   ```

2. Execute an AWS CLI terminal command string loop to provision a service account binding template mapping options:
```bash
aws iam create-role --role-name vmimport --assume-role-policy-document file://trust-policy.json

```


3. Author a structural definition manifest defining file specifications inside a document named **`containers.json`**:
```json
[
  {
    "Description": "Ubuntu Server Migration Stream",
    "Format": "ova",
    "UserBucket": {
      "S3Bucket": "xyz-migration-staging-bucket-2026",
      "S3Key": "ubuntu-migration-source.ova"
    }
  }
]

```
---

### Step 4: Convert the Staged Disk Package into a Custom AMI

1. Execute the absolute core data transformation tasks tool call inside your terminal interface using the AWS CLI command profile:
```bash
aws ec2 import-image --description "Imported Ubuntu Server Migration Node" --disk-containers file://containers.json

```


2. Copy the **ImportTaskId** generated inside the return payload string (e.g., `import-ami-1234abcd`).
3. Monitor migration pipeline states interactively by triggering tracing telemetry calls:
```bash
aws ec2 describe-import-image-tasks --import-task-ids import-ami-1234abcd

```


4. *Wait until the Progress metric indicators reach 100% and the central operational status changes into a live **completed** notation flag.*

---

### Step 5: Launch and Validate the New Cloud Server Instance (Task 3 Validation)

1. Open the **Amazon EC2 Console** dashboard workspace window and select **AMIs** from the *Images* sidebar sub-panel category.
2. Locate your newly compiled migrated master blueprint listed under **Owned by me** classifications.
3. Highlight your custom imported AMI item row container elements and click **Launch instance from AMI**.
4. Configure standard server parameters:
* **Name:** `XYZ-Cloud-Migrated-Webnode`
* **Instance type:** Select **t2.micro** or `t3.micro`.
* **Security groups:** Connect a layout template enabling inbound Port 22 connectivity scopes.


5. Click **Launch instance**. Once active, verify you can connect seamlessly into the host terminal console shell interface environment using your original on-premises username credentials (`adminuser`), confirming the successful migration lifecycle.

---

## Part 2: Step-by-Step Deletion Process (CRITICAL Cost Saving)

Wipe out these large data artifacts, storage buckets, custom image snapshots, and active compute hosts immediately upon assignment review to optimize budget boundaries:

### 1. Terminate the Migrated EC2 Cloud Instance Node

1. Inside the **Amazon EC2 Console**, navigate to **Instances**.
2. Check the box group matching **`XYZ-Cloud-Migrated-Webnode`**.
3. Choose **Instance state** > **Terminate instance** > Confirm structural removal actions.

### 2. Deregister Custom AMI and Purge Block Storage Snapshots

1. Go to **EC2 > AMIs**, highlight your imported cloud configuration file layout entry, click **Actions**, and select **Deregister AMI**.
2. Navigate immediately downward into **EC2 > Snapshots**. Find the storage volume block mapped directly to that imported AMI reference, click **Actions**, and execute **Delete snapshot** to avoid orphan data retention billing paths.

### 3. Clear and Remove the S3 Staging Bucket Container

1. Open the **Amazon S3 Console** > click your unique deployment bucket `xyz-migration-staging-bucket-2026`.
2. Click **Empty**, confirm permanent deletion of the huge `.ova` disk payload file package, and click submit.
3. Return to the main buckets screen index list, highlight the empty bucket container element row, and click **Delete** to finish cleanup operations cleanly.
