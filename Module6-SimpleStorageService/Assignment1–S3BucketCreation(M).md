# Module 6 - Simple Storage Service (S3)

# Assignment 1 - S3 Bucket Creation

## Problem Statement

You work for XYZ Corporation. Their application requires a storage service that can store files and publicly share them if required. Implement Amazon Simple Storage Service (Amazon S3) for the same.

## Tasks To Be Performed

1. Create an S3 bucket for file storage.
2. Upload 5 objects with different file extensions.

---

# 1. Free Tier / Cost Check

Before creating AWS resources, it is important to understand whether the services used in this assignment can generate charges.

This assignment uses only Amazon Simple Storage Service (Amazon S3). Amazon S3 is an object storage service used to store files as objects inside containers called buckets.

| AWS Service                       | Free Tier Eligible                                                     | Uses Credits                                    | Notes                                                                                                                                                                                       |
| --------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Amazon S3 General Purpose Bucket  | Yes, subject to the AWS account's applicable Free Tier plan and limits | Possibly                                        | Creating a small number of general purpose buckets does not normally create a meaningful cost by itself. Charges can apply based on storage, requests, data transfer, and enabled features. |
| S3 Standard Storage               | Yes, within applicable Free Tier limits                                | Possibly                                        | The five small files used in this assignment require minimal storage. Usage beyond applicable Free Tier allowances is charged according to S3 pricing.                                      |
| S3 PUT Requests                   | Yes, within applicable Free Tier limits                                | Possibly                                        | Uploading five files generates a very small number of write requests.                                                                                                                       |
| S3 GET Requests                   | Yes, within applicable Free Tier limits                                | Possibly                                        | Opening or downloading objects generates read requests.                                                                                                                                     |
| Data Transfer Into S3             | Yes                                                                    | No for standard internet data transfer into AWS | Uploading the five assignment files into S3 does not normally incur an internet data-transfer-in charge.                                                                                    |
| Data Transfer Out to the Internet | Free within applicable allowance; chargeable beyond applicable limits  | Possibly                                        | Downloading or publicly accessing objects can generate outbound data transfer.                                                                                                              |

### Is this assignment completely Free Tier eligible?

For a normal learning exercise involving:

* `1` S3 general purpose bucket
* `5` small objects
* A very small number of upload and retrieval requests

the usage is extremely small and can generally be completed within applicable AWS Free Tier allowances or promotional credits.

However, the exact Free Tier treatment depends on when the AWS account was created and which AWS Free Tier plan applies to that account. AWS changed its Free Tier account model for new customers beginning July 15, 2025.

Therefore, do not assume that every AWS account has exactly the same Free Tier allowances.

### Which AWS services can consume credits?

Amazon S3 can consume promotional credits if billable usage is generated. Potential billable categories include:

* Object storage.
* PUT, COPY, POST, LIST, and other write-related requests.
* GET and other read-related requests.
* Data transfer out to the internet beyond applicable free allowances.
* Optional paid S3 features, if enabled.

For this assignment, only five small objects are uploaded, so expected usage is minimal.

### Is it better to complete this assignment while promotional credits are available?

Yes. If promotional credits are available, it is sensible to complete AWS practical assignments while those credits are active because they can cover eligible billable usage.

For this specific assignment, however, the expected S3 usage is extremely small. The assignment does not require expensive infrastructure such as EC2 instances, NAT Gateways, load balancers, or databases.

### Cost Optimization Decision for This Assignment

To minimize unnecessary usage:

* Create only `1` S3 general purpose bucket.
* Use the `S3 Standard` storage class for the five small assignment files.
* Keep S3 Versioning `disabled` because version history is not required by the assignment.
* Do not enable S3 Object Lock.
* Keep default encryption enabled.
* Keep Block Public Access enabled because the assignment only says files should be publicly shareable **if required**; it does not explicitly require us to make the five uploaded files public.
* Delete all five objects first during cleanup.
* Delete the empty S3 bucket immediately after verification.

---

# 2. Architecture and Resource Dependency Flow

The assignment has a simple dependency structure:

```text
AWS Account
    |
    v
Amazon S3 Service
    |
    v
S3 General Purpose Bucket
    |
    +-------------------+-------------------+-------------------+-------------------+
    |                   |                   |                   |                   |
    v                   v                   v                   v                   v
document.txt         image.jpg          data.csv           config.json         page.html
```

The five objects depend on the S3 bucket because an S3 object cannot be uploaded without a destination bucket.

The dependency order is therefore:

```text
Step 1: Prepare five files with different extensions
                |
                v
Step 2: Open the Amazon S3 Console
                |
                v
Step 3: Create the S3 bucket
                |
                v
Step 4: Upload all five objects
                |
                v
Step 5: Verify the bucket and objects
                |
                v
Step 6: Delete the five objects
                |
                v
Step 7: Delete the empty S3 bucket
```

---

# 3. Step-by-Step Solution

## Step 1: Prepare Five Files with Different File Extensions

### Navigation

This step is performed on your local computer before opening the AWS Management Console.

Example location on Windows:

```text
Windows File Explorer
→ Documents
→ Create a new folder
→ Name the folder: S3-Assignment-1-Files
```

### Configuration

Create the following five files:

| File Number | File Name      | File Extension | Purpose                   |
| ----------- | -------------- | -------------- | ------------------------- |
| 1           | `document.txt` | `.txt`         | Plain-text file           |
| 2           | `image.jpg`    | `.jpg`         | Image file                |
| 3           | `data.csv`     | `.csv`         | Comma-separated data file |
| 4           | `config.json`  | `.json`        | JSON configuration file   |
| 5           | `page.html`    | `.html`        | HTML web page             |

You can use any existing valid files with these extensions.

For a clean and reproducible assignment, create a folder named:

* **Folder Name:** `S3-Assignment-1-Files`

Place all five files inside this folder.

An example folder structure is:

```text
S3-Assignment-1-Files/
├── document.txt
├── image.jpg
├── data.csv
├── config.json
└── page.html
```

> **Important:** A `.jpg` file should be an actual image file. Simply renaming a text file from `.txt` to `.jpg` changes only the filename extension and does not convert the underlying content into a valid JPEG image.

### Commands

No terminal command is required because the assignment can be completed entirely through the AWS Management Console.

You can create the files using applications such as:

* Notepad for `document.txt`.
* Any existing JPEG image for `image.jpg`.
* Notepad or a spreadsheet application for `data.csv`.
* Notepad or a code editor for `config.json`.
* Notepad or a code editor for `page.html`.

### Why is this step required?

The assignment specifically requires uploading five objects with different file extensions.

In Amazon S3 terminology, every uploaded file becomes an **object**.

An S3 object consists primarily of:

* Object data.
* An object key, which identifies the object inside the bucket.
* Metadata associated with the object.

Preparing all five files before creating and uploading resources ensures that the assignment can be completed in the correct dependency order without interrupting the workflow later.

### Dependency

This is the first implementation step and has no dependency on any previously created AWS resource.

It only requires:

* A local computer.
* Five files with different file extensions.

### What happens if this step is skipped?

If this step is skipped, the required five files will not be ready for upload.

As a result:

* Task 2 of the assignment cannot be completed.
* The S3 bucket may be created successfully, but it will not contain the required five objects with different extensions.
* Verification of the complete assignment will fail.

---

## Step 2: Sign In to the AWS Management Console and Open Amazon S3

### Navigation

```text
AWS Management Console
→ Search bar
→ Search for S3
→ Select S3
```

### Configuration

Before creating the bucket, confirm that you are signed in to the correct AWS account.

Amazon S3 bucket names are globally unique across AWS accounts within an AWS partition. Therefore, the exact example bucket name shown in this document may already be taken by another AWS customer.

For this assignment, use the following naming pattern:

* **Bucket Name Pattern:** `xyz-corporation-file-storage-yourname-uniqueid`

Example:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`

If this name is unavailable, add another unique suffix.

Example:

* `xyz-corporation-file-storage-shabdali-2026-01`

### Why is this step required?

The Amazon S3 console provides the interface required to:

* Create the S3 bucket.
* Configure bucket settings.
* Upload objects.
* View object properties.
* Verify stored files.
* Delete objects and the bucket during cleanup.

### Dependency

This step depends on:

* An active AWS account.
* Permission to access Amazon S3.
* Successful sign-in to the AWS Management Console.

### What happens if this step is skipped?

Without accessing Amazon S3:

* The bucket cannot be created.
* Objects cannot be uploaded.
* The assignment cannot be implemented.

---

## Step 3: Create the Amazon S3 Bucket

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ Create bucket
```

### Configuration

Configure the bucket as follows.

#### General Configuration

* **Bucket type:** `General purpose`
* **Bucket name:** `xyz-corporation-file-storage-shabdali-2026`
* **AWS Region:** Select your preferred AWS Region and keep all work for this assignment in the same Region.

For this assignment, the example Region is:

* **AWS Region:** `Asia Pacific (Mumbai) ap-south-1`

> **Important:** If the example bucket name is already taken, use a different globally unique bucket name. Wherever this document shows `xyz-corporation-file-storage-shabdali-2026`, substitute your actual bucket name.

#### Object Ownership

Select:

* **Object Ownership:** `ACLs disabled (recommended)`
* **Bucket owner enforced:** `Enabled`

With ACLs disabled, access to the bucket and its objects is managed through policies instead of individual object ACLs.

#### Block Public Access Settings for This Bucket

Keep all default Block Public Access settings enabled:

* **Block all public access:** `Enabled`
* **Block public access to buckets and objects granted through new access control lists (ACLs):** `Enabled`
* **Block public access to buckets and objects granted through any access control lists (ACLs):** `Enabled`
* **Block public access to buckets and objects granted through new public bucket or access point policies:** `Enabled`
* **Block public and cross-account access to buckets and objects through any public bucket or access point policies:** `Enabled`

The problem statement says the storage service should be capable of publicly sharing files **if required**. It does not explicitly require the five assignment objects to be publicly accessible.

Therefore, public access should remain blocked for this assignment. This follows the principle of keeping resources private unless public access is explicitly required.

#### Bucket Versioning

Select:

* **Bucket Versioning:** `Disable`

Versioning is not required because the assignment does not ask us to retain multiple versions of objects.

#### Tags

No tags are required by the assignment.

You may leave the Tags section empty.

#### Default Encryption

Keep the default server-side encryption settings:

* **Encryption type:** `Server-side encryption with Amazon S3 managed keys (SSE-S3)`
* **Bucket Key:** Not applicable to `SSE-S3`

#### Advanced Settings

Keep:

* **Object Lock:** `Disable`

Object Lock is not required for this assignment.

After reviewing all settings, select:

```text
Create bucket
```

### Why is this step required?

An S3 bucket is the top-level container that stores S3 objects.

The assignment requires five files to be stored in Amazon S3. Before any file can be uploaded as an S3 object, a destination bucket must exist.

The bucket provides:

* A logical container for objects.
* A globally unique bucket name.
* A specific AWS Region where the bucket is created.
* Security and access-control settings.
* Encryption configuration.
* Storage-management capabilities.

### Dependency

This step depends on:

* Step 2: Successful access to the Amazon S3 console.
* An AWS account with permission to create S3 buckets.
* A globally unique bucket name.

### What happens if this step is skipped?

If the S3 bucket is not created:

* There will be no destination for the five files.
* No S3 objects can be uploaded for this assignment.
* Task 1 will remain incomplete.
* Task 2 cannot begin.

---

## Step 4: Open the Newly Created S3 Bucket

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ Select xyz-corporation-file-storage-shabdali-2026
```

### Configuration

Confirm that you are viewing the correct bucket.

You should see tabs such as:

* `Objects`
* `Properties`
* `Permissions`
* `Metrics`
* `Management`
* `Access Points`

Select the `Objects` tab if it is not already selected.

At this point, the bucket should be empty because no objects have been uploaded yet.

### Why is this step required?

The five files must be uploaded into the specific bucket created for this assignment.

Opening the bucket ensures that the upload operation targets the correct destination.

### Dependency

This step depends on:

* Step 3: The S3 bucket must already exist.

### What happens if this step is skipped?

If you do not open the correct bucket:

* You cannot start the object upload process for that bucket.
* You may accidentally upload the files to another existing bucket.
* The assignment resources may become mixed with unrelated resources.

---

## Step 5: Start the Object Upload Process

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Upload
```

### Configuration

On the Upload page, select:

```text
Add files
```

From your local folder named `S3-Assignment-1-Files`, select all five files:

* `document.txt`
* `image.jpg`
* `data.csv`
* `config.json`
* `page.html`

After selecting the files, confirm that all five appear in the **Files and folders** section of the S3 upload page.

The expected selection is:

| Object Number | Object Name    | Extension |
| ------------- | -------------- | --------- |
| 1             | `document.txt` | `.txt`    |
| 2             | `image.jpg`    | `.jpg`    |
| 3             | `data.csv`     | `.csv`    |
| 4             | `config.json`  | `.json`   |
| 5             | `page.html`    | `.html`   |

### Why is this step required?

Selecting the files adds them to the pending S3 upload operation.

At this stage, the files have not yet been permanently uploaded to the bucket. They are only selected and ready for the upload request.

### Dependency

This step depends on:

* Step 1: Five local files must exist.
* Step 3: The S3 bucket must exist.
* Step 4: The correct bucket must be open.

### What happens if this step is skipped?

If the files are not selected:

* There will be nothing to upload.
* The bucket will remain empty.
* Task 2 of the assignment will remain incomplete.

---

## Step 6: Configure Upload Properties and Upload the Five Objects

### Navigation

Continue from the S3 Upload page opened in the previous step:

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Upload
→ Files and folders
```

### Configuration

Before starting the upload, review the selected files and upload settings.

Confirm that all five files are listed:

| Object Number | Object Name    | File Extension |
| ------------- | -------------- | -------------- |
| 1             | `document.txt` | `.txt`         |
| 2             | `image.jpg`    | `.jpg`         |
| 3             | `data.csv`     | `.csv`         |
| 4             | `config.json`  | `.json`        |
| 5             | `page.html`    | `.html`        |

Keep the default upload settings unless your AWS Console displays additional optional configuration.

For this assignment:

* **Destination:** `s3://xyz-corporation-file-storage-shabdali-2026/`
* **Permissions:** Keep default permissions.
* **Storage class:** `S3 Standard`
* **Server-side encryption:** Use the bucket's default encryption settings.
* **Additional checksums:** Keep the AWS Console default unless explicitly required otherwise.

Do not enable public access for these objects because the assignment requires file storage and only states that files should be publicly shareable **if required**. It does not explicitly require these five objects to be publicly accessible.

After confirming all five files, select:

```text
Upload
```

Wait until the upload process completes.

The AWS Console should display a success message similar to:

```text
Upload succeeded
```

The upload status for each object should indicate success.

After successful upload, select:

```text
Close
```

You will return to the bucket's `Objects` tab.

### Commands

No terminal command is required for this step because the files are being uploaded through the AWS Management Console.

### Why is this step required?

This step performs the actual transfer of the five local files into the Amazon S3 bucket.

Before this step, the files only exist on your local computer and are merely selected in the upload interface. After the upload succeeds, each file becomes an S3 object stored inside the bucket.

For example:

```text
Local Computer
    |
    | Upload
    v
Amazon S3 Bucket
    |
    +-- document.txt
    +-- image.jpg
    +-- data.csv
    +-- config.json
    +-- page.html
```

Amazon S3 stores files as **objects**. Therefore:

```text
Local file + Upload to S3 = S3 object
```

Each object has important properties, including:

* **Object key:** The name or path that uniquely identifies the object inside the bucket.
* **Size:** The amount of data stored by the object.
* **Last modified date:** The date and time when the object was last modified.
* **Storage class:** The S3 storage class used to store the object.
* **Encryption information:** The server-side encryption mechanism protecting the object at rest.
* **Metadata:** Additional information associated with the object.

### Dependency

This step depends on:

* Step 1: The five local files must exist.
* Step 3: The S3 bucket must exist.
* Step 4: The correct bucket must be open.
* Step 5: All five files must be selected for upload.

### What happens if this step is skipped?

If the actual upload operation is skipped:

* The five files will remain only on the local computer.
* The S3 bucket will remain empty.
* No S3 objects will be created.
* Task 2 of the assignment will not be completed.
* Verification cannot confirm the presence of five objects with different extensions.

---

# 4. Verification / Output Checking

After implementation, verify both assignment requirements:

1. An S3 bucket has been created successfully.
2. The bucket contains exactly the five uploaded objects with different file extensions.

---

## Verification Step 1: Verify That the S3 Bucket Exists

### Action

Open the Amazon S3 bucket list and locate the bucket created for this assignment.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
```

Locate:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`

If you used a different unique suffix, locate your actual bucket name instead.

### Expected Output

The bucket should appear in the general purpose buckets list.

Example:

```text
Bucket name                                      AWS Region
---------------------------------------------------------------
xyz-corporation-file-storage-shabdali-2026       ap-south-1
```

The exact AWS Console columns may vary, but the bucket name should be visible.

### Why does this confirm success?

The presence of the bucket in the S3 bucket list confirms that:

* Amazon S3 successfully created the general purpose bucket.
* The bucket exists in the selected AWS Region.
* Task 1 of the assignment has been completed successfully.

If the bucket is not visible, verify:

* You are signed in to the correct AWS account.
* You are viewing the S3 general purpose bucket list.
* The bucket creation operation completed successfully.

---

## Verification Step 2: Verify That All Five Objects Exist

### Action

Open the bucket and inspect its `Objects` tab.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Expected Output

The object list should contain all five uploaded objects:

```text
Name
-------------
config.json
data.csv
document.txt
image.jpg
page.html
```

Depending on the current AWS Console sorting order, the files may appear in a different sequence.

You should observe five objects with five different extensions:

| Object Name    | Extension | Verification             |
| -------------- | --------- | ------------------------ |
| `document.txt` | `.txt`    | Plain-text object exists |
| `image.jpg`    | `.jpg`    | JPEG image object exists |
| `data.csv`     | `.csv`    | CSV object exists        |
| `config.json`  | `.json`   | JSON object exists       |
| `page.html`    | `.html`   | HTML object exists       |

### Why does this confirm success?

This confirms that:

* All five local files were successfully transferred to Amazon S3.
* Each uploaded file now exists as an S3 object.
* The objects use five different file extensions.
* Task 2 of the assignment has been completed successfully.

The assignment requires five objects with different file extensions. Seeing all five objects in the bucket directly verifies this requirement.

---

## Verification Step 3: Verify an Individual Object's Properties

### Action

Select one uploaded object and inspect its properties.

For example, select:

* **Object:** `document.txt`

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ document.txt
```

### Expected Output

The object details page should display information similar to:

```text
Object name: document.txt
S3 URI: s3://xyz-corporation-file-storage-shabdali-2026/document.txt
Storage class: Standard
Server-side encryption: Amazon S3 managed keys (SSE-S3)
```

The exact values displayed for size, entity tag, last modified date, checksum, and other metadata will depend on the actual file and upload operation.

### Why does this confirm success?

The object details page confirms that:

* The object exists inside the correct S3 bucket.
* Amazon S3 assigned the object an S3 URI.
* The object is stored using the selected storage class.
* Server-side encryption protects the object at rest according to the bucket's encryption configuration.

The S3 URI follows this general structure:

```text
s3://bucket-name/object-key
```

For this assignment:

```text
s3://xyz-corporation-file-storage-shabdali-2026/document.txt
```

Where:

| URI Part                                     | Explanation                                       |
| -------------------------------------------- | ------------------------------------------------- |
| `s3://`                                      | Identifies the resource as an Amazon S3 resource. |
| `xyz-corporation-file-storage-shabdali-2026` | The globally unique S3 bucket name.               |
| `/`                                          | Separates the bucket name from the object key.    |
| `document.txt`                               | The object's key inside the bucket.               |

---

## Verification Step 4: Verify That Objects Are Not Public by Default

### Action

Open an object's details page and inspect its Object URL.

For example:

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ document.txt
→ Object URL
```

Open the Object URL in a new browser tab.

### Expected Output

Because Block Public Access remains enabled and no public bucket policy has been configured, direct anonymous access should not succeed.

You may receive an XML response similar to:

```text
<Error>
    <Code>AccessDenied</Code>
    <Message>Access Denied</Message>
</Error>
```

The exact formatting may vary.

### Why does this confirm success?

This confirms that the bucket and objects remain private by default.

This is expected and desirable because:

* The assignment does not explicitly require public access to be enabled.
* S3 resources should remain private unless public access is specifically necessary.
* Keeping Block Public Access enabled reduces the risk of accidental data exposure.

The application requirement states that files should be publicly shareable **if required**. Amazon S3 supports public sharing through mechanisms such as bucket policies when intentionally configured, but public access is not required to complete the two tasks specified in this assignment.

---

# 5. Cleanup / Cost Optimization

After completing all verification steps, delete every AWS resource created during this assignment.

The cleanup dependency order is:

```text
Step 1: Delete all five S3 objects
                |
                v
Step 2: Verify that the S3 bucket is empty
                |
                v
Step 3: Delete the empty S3 bucket
```

This order is important because a general purpose S3 bucket must be empty before it can be deleted through the normal S3 console deletion process.

---

## Cleanup Step 1: Delete All Five S3 Objects

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Action

Select all five objects:

* `document.txt`
* `image.jpg`
* `data.csv`
* `config.json`
* `page.html`

Then select:

```text
Delete
```

The AWS Console will open the **Delete objects** confirmation page.

Depending on the current AWS Console workflow, confirm the deletion as instructed on the screen. This commonly requires entering:

```text
permanently delete
```

Then select the final object deletion confirmation button.

Wait until the console confirms that the objects were successfully deleted.

### Why is this required?

Amazon S3 stores data as objects, and stored objects can contribute to:

* Storage usage.
* Request usage.
* Applicable charges after free allowances or credits are exhausted.

Deleting the five assignment objects removes the data created specifically for this exercise.

Object deletion is also necessary because the bucket must be empty before the bucket itself can be deleted through the normal console workflow.

### What happens if this resource is not deleted?

If the five objects are not deleted:

* They remain stored in Amazon S3.
* They can continue contributing to storage usage.
* The bucket cannot be deleted through the standard empty-bucket deletion workflow.
* Unnecessary assignment resources remain in the AWS account.

For five tiny files, the direct cost is likely extremely small, but deleting unused resources is an important cloud cost-optimization practice.

---

## Cleanup Step 2: Verify That the S3 Bucket Is Empty

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Action

Confirm that no objects remain in the bucket.

### Expected Result

The `Objects` tab should show that the bucket contains no objects.

The exact AWS Console message may vary, but the object list should be empty.

### Why is this required?

Verifying that the bucket is empty prevents the next cleanup operation from failing.

The correct dependency relationship is:

```text
Objects exist
    |
    v
Bucket cannot be deleted normally
    |
    v
Delete objects
    |
    v
Bucket becomes empty
    |
    v
Bucket can be deleted
```

### What happens if this resource is not deleted?

This step does not create or delete a resource itself. It verifies that the previous cleanup operation succeeded.

If this verification is skipped and objects remain in the bucket:

* The bucket deletion operation may fail.
* You may need to return to the bucket and repeat object deletion.
* Cleanup remains incomplete.

---

## Cleanup Step 3: Delete the Empty S3 Bucket

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
```

### Action

Select:

* **Bucket:** `xyz-corporation-file-storage-shabdali-2026`

Then select:

```text
Delete
```

The AWS Console will ask you to confirm the bucket deletion.

Enter the exact bucket name when prompted:

```text
xyz-corporation-file-storage-shabdali-2026
```

Then select:

```text
Delete bucket
```

Wait for the deletion operation to complete.

### Why is this required?

The S3 bucket was created only for this assignment.

Deleting it after verification:

* Removes unused AWS resources.
* Keeps the AWS account organized.
* Prevents future accidental uploads to an obsolete assignment bucket.
* Follows good cloud resource lifecycle management practices.

Although an empty general purpose S3 bucket itself does not normally produce a meaningful storage charge, unused resources should still be removed when they are no longer required.

### What happens if this resource is not deleted?

If the bucket is left in the AWS account:

* It remains as an unused cloud resource.
* The globally unique bucket name remains associated with your account while the bucket exists.
* Future accidental uploads could generate storage and request usage.
* The AWS account becomes cluttered with obsolete assignment resources.

---

## Cleanup Step 4: Verify That the S3 Bucket Has Been Deleted

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
```

### Action

Search for:

```text
xyz-corporation-file-storage-shabdali-2026
```

### Expected Result

The bucket should no longer appear in the general purpose buckets list.

### Why is this required?

This final verification confirms that:

* All five S3 objects were deleted.
* The S3 bucket was successfully deleted.
* No AWS resource created specifically for this assignment remains active.
* Cleanup is complete.
* Unnecessary storage and request usage from this assignment has been eliminated.

### What happens if this resource is not deleted?

This is a verification step rather than a separate resource deletion operation.

If the bucket still appears:

* The cleanup process is incomplete.
* Recheck whether the bucket is empty.
* Retry the bucket deletion process.
* Confirm that you have sufficient IAM permissions to delete the bucket.

---
