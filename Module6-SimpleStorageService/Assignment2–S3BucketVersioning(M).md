# Module 6 - Simple Storage Service (S3)

# Assignment 2 - S3 Bucket Versioning

## Problem Statement

You work for XYZ Corporation. Their application requires a storage service that can store files and publicly share them if required. Implement Amazon Simple Storage Service (Amazon S3) for the same.

## Tasks To Be Performed

1. Enable versioning for the bucket created in Assignment 1.
2. Re-upload any 2 files already uploaded to verify whether versioning works.

---

# 1. Free Tier / Cost Check

Before enabling S3 Versioning, it is important to understand its potential cost impact.

This assignment uses the existing Amazon S3 general purpose bucket and existing objects created in Assignment 1.

The important difference is that enabling S3 Versioning allows Amazon S3 to preserve multiple versions of the same object key. Each stored object version contributes to storage usage.

| AWS Service                       | Free Tier Eligible                                                     | Uses Credits                      | Notes                                                                                                               |
| --------------------------------- | ---------------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Amazon S3 General Purpose Bucket  | Yes, subject to the AWS account's applicable Free Tier plan and limits | Possibly                          | The existing bucket from Assignment 1 is reused. No additional bucket is required.                                  |
| S3 Standard Storage               | Yes, within applicable Free Tier limits                                | Possibly                          | Both current and noncurrent object versions consume storage.                                                        |
| S3 Versioning                     | No separate charge merely for enabling the setting                     | Indirectly                        | Enabling versioning itself does not create a separate versioning fee, but retained object versions consume storage. |
| S3 PUT Requests                   | Yes, within applicable Free Tier limits                                | Possibly                          | Re-uploading two files creates PUT requests and new object versions.                                                |
| S3 GET Requests                   | Yes, within applicable Free Tier limits                                | Possibly                          | Viewing or downloading object versions can generate read requests.                                                  |
| Data Transfer Into S3             | Generally no standard internet data-transfer-in charge                 | No for standard transfer into AWS | Re-uploading two files to S3 generally does not incur an internet data-transfer-in charge.                          |
| Data Transfer Out to the Internet | Free within applicable allowances; chargeable beyond applicable limits | Possibly                          | Downloading object versions may generate outbound data transfer.                                                    |

### Is this assignment completely Free Tier eligible?

For a small learning exercise involving:

* `1` existing S3 bucket.
* `5` existing objects from Assignment 1.
* S3 Versioning enabled on the existing bucket.
* `2` files re-uploaded.
* A very small number of S3 requests.

the expected usage is extremely small and can generally be completed within applicable AWS Free Tier allowances or promotional credits.

However, exact Free Tier treatment depends on the AWS account's applicable Free Tier plan, account creation date, storage usage, request usage, and other account-specific factors.

### Which AWS services can consume credits?

Amazon S3 can consume promotional credits if billable usage is generated.

For this assignment, potential usage includes:

* Storage used by the original object versions.
* Storage used by the newly uploaded object versions.
* PUT requests generated when the two files are re-uploaded.
* GET requests generated if object versions are downloaded or opened.
* Data transfer out beyond applicable free allowances.

### Is it better to complete this assignment while promotional credits are available?

Yes. If promotional credits are available, completing AWS practical assignments while those credits remain active can help cover eligible billable usage.

For this assignment, the actual expected cost is extremely small because only two additional object versions are created.

However, S3 Versioning requires careful cleanup because older versions remain stored unless explicitly removed.

### Important Cost Behavior of S3 Versioning

Suppose the following object exists:

```text
document.txt
```

After S3 Versioning is enabled, the same object is uploaded again using the same object key:

```text
document.txt
```

Amazon S3 does not simply overwrite and permanently discard the previous data.

Instead, the version history conceptually becomes:

```text
document.txt
├── Version 2 → Current version
└── Version 1 → Noncurrent version
```

Both versions consume storage.

If the object is updated many times, the version history can become:

```text
document.txt
├── Version 5 → Current version
├── Version 4 → Noncurrent version
├── Version 3 → Noncurrent version
├── Version 2 → Noncurrent version
└── Version 1 → Noncurrent version
```

Therefore, unused noncurrent versions should be removed during cleanup when they are no longer required.

### Cost Optimization Decision for This Assignment

To minimize unnecessary AWS usage:

* Reuse the existing S3 bucket from Assignment 1.
* Do not create another bucket.
* Enable versioning only on the existing assignment bucket.
* Re-upload only `2` files as required.
* Do not repeatedly upload the same objects unnecessarily.
* Verify the generated version IDs.
* During cleanup, permanently delete all object versions and delete markers.
* Delete the empty S3 bucket after verification if it is no longer needed for subsequent assignments.

> **Important:** Because this assignment is based on Assignment 1, do not perform Assignment 1 cleanup before starting Assignment 2. The original bucket and objects must still exist.

---

# 2. Architecture and Resource Dependency Flow

This assignment depends directly on the resources created in Assignment 1.

The starting state should be:

```text
AWS Account
    |
    v
Amazon S3
    |
    v
Existing S3 General Purpose Bucket
    |
    +-- document.txt
    +-- image.jpg
    +-- data.csv
    +-- config.json
    +-- page.html
```

For this assignment, we will re-upload:

* `document.txt`
* `data.csv`

The exact dependency flow is:

```text
Assignment 1 Completed
        |
        v
Existing S3 Bucket
        |
        v
Existing Five Objects
        |
        v
Enable S3 Versioning
        |
        v
Modify document.txt locally
        |
        v
Modify data.csv locally
        |
        v
Re-upload document.txt with the same object key
        |
        v
Re-upload data.csv with the same object key
        |
        v
Amazon S3 Creates New Versions
        |
        v
Enable "Show versions"
        |
        v
Verify Multiple Version IDs
```

After successful re-upload, the conceptual object structure becomes:

```text
S3 Bucket
│
├── document.txt
│   ├── New Version    → Current
│   └── Original       → Previous version
│
├── data.csv
│   ├── New Version    → Current
│   └── Original       → Previous version
│
├── image.jpg
├── config.json
└── page.html
```

---

# 3. Important Prerequisite: Understand the Starting State

Before enabling versioning, confirm the resources from Assignment 1 still exist.

The expected starting state is:

| Resource           | Required State                         |
| ------------------ | -------------------------------------- |
| Existing S3 bucket | Must exist                             |
| `document.txt`     | Must exist in the bucket               |
| `image.jpg`        | Must exist in the bucket               |
| `data.csv`         | Must exist in the bucket               |
| `config.json`      | Must exist in the bucket               |
| `page.html`        | Must exist in the bucket               |
| S3 Versioning      | Disabled before this assignment begins |

For the examples in this document, the existing bucket is:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`
* **AWS Region:** `Asia Pacific (Mumbai) ap-south-1`

If your actual bucket has a different name, substitute your actual bucket name throughout the assignment.

> **Important:** Do not create a new S3 bucket for this assignment. Task 1 explicitly requires enabling versioning on the bucket created in the previous assignment.

---

# 4. Step-by-Step Solution

## Step 1: Verify That the Existing S3 Bucket from Assignment 1 Is Available

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
```

### Configuration

Locate the existing bucket from Assignment 1:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`

If you used a different unique bucket name in Assignment 1, select that bucket instead.

### Commands

No terminal command is required because this step is performed through the AWS Management Console.

### Why is this step required?

Assignment 2 explicitly depends on the S3 bucket created in Assignment 1.

Before enabling versioning, we must verify that the correct bucket exists so that:

* Versioning is not accidentally enabled on an unrelated bucket.
* The original five objects remain available.
* The two required files can be re-uploaded using the same object keys.

### Dependency

This step depends on:

* Assignment 1 having been completed.
* The S3 bucket from Assignment 1 still existing.
* The user having permission to access the bucket.

### What happens if this step is skipped?

If this verification is skipped:

* You may accidentally configure the wrong S3 bucket.
* The required original objects may not exist.
* Re-uploading the same two files may not demonstrate the intended versioning behavior.

---

## Step 2: Verify That the Five Original Objects Still Exist

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Configuration

Confirm that the following five objects are present:

| Object Number | Object Name    | File Extension |
| ------------- | -------------- | -------------- |
| 1             | `document.txt` | `.txt`         |
| 2             | `image.jpg`    | `.jpg`         |
| 3             | `data.csv`     | `.csv`         |
| 4             | `config.json`  | `.json`        |
| 5             | `page.html`    | `.html`        |

The expected object list is similar to:

```text
config.json
data.csv
document.txt
image.jpg
page.html
```

The exact display order may differ depending on the AWS Console sorting configuration.

### Commands

No terminal command is required because this verification is performed through the AWS Management Console.

### Why is this step required?

The assignment requires us to:

> Re-upload any 2 files already uploaded to verify if versioning works.

Therefore, the original objects must already exist before the files are re-uploaded.

For this solution, we will use:

* `document.txt`
* `data.csv`

These two objects must already exist in the bucket.

### Dependency

This step depends on:

* Step 1: The correct S3 bucket must exist.
* Assignment 1: The original five objects must have been uploaded.

### What happens if this step is skipped?

If the original objects do not exist and you upload the files after enabling versioning:

* Amazon S3 will create only the first versioned version of each new object.
* You may not observe the expected old and new content history in the way intended by the assignment.
* The assignment requirement to re-upload already uploaded files will not be satisfied.

---

## Step 3: Check the Current S3 Versioning Status

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Bucket Versioning
```

### Configuration

Before changing anything, observe the current versioning status.

Expected starting configuration:

* **Bucket Versioning:** `Disabled`

Select:

```text
Edit
```

Do not save the change yet until the next step.

### Commands

No terminal command is required because the bucket versioning setting is being managed through the AWS Management Console.

### Why is this step required?

This step establishes the starting state before enabling versioning.

S3 Versioning has three conceptual states:

| State       | Meaning                                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Unversioned | Versioning has never been enabled on the bucket.                                                                                       |
| Enabled     | New object uploads receive unique version IDs.                                                                                         |
| Suspended   | Versioning was previously enabled but is no longer generating normal new version IDs in the same way. Existing versions remain stored. |

For a bucket created in Assignment 1 with versioning disabled, the expected initial state is unversioned.

### Dependency

This step depends on:

* Step 1: The existing S3 bucket must be available.
* Permission to view and modify the bucket's properties.

### What happens if this step is skipped?

Versioning can technically be enabled without separately checking the initial status, but skipping this verification removes an important baseline.

You would not clearly establish:

* Whether versioning was previously disabled.
* Whether the bucket was already version-enabled.
* Whether the observed versions were created during this assignment or existed beforehand.

---

## Step 4: Enable S3 Versioning

### Navigation

Continue from:

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Bucket Versioning
→ Edit
```

### Configuration

Select:

* **Bucket Versioning:** `Enable`

Then select:

```text
Save changes
```

Wait for the AWS Console to confirm that the change has been saved.

After saving, the bucket properties should show:

* **Bucket Versioning:** `Enabled`

### Commands

No terminal command is required because versioning is enabled through the AWS Management Console.

### Why is this step required?

S3 Versioning allows multiple versions of an object to exist under the same object key.

Without versioning:

```text
Upload document.txt
        |
        v
Upload document.txt again
        |
        v
Previous object data is overwritten
```

With versioning enabled:

```text
Upload document.txt
        |
        v
Original object exists
        |
        v
Upload document.txt again
        |
        v
New current version is created
        |
        +---------------------------+
        |                           |
        v                           v
Current Version              Previous Version
Unique Version ID            Earlier version state
```

Every new versioned upload receives a unique version ID.

This enables capabilities such as:

* Recovering accidentally overwritten objects.
* Restoring previous object versions.
* Protecting against unintended application changes.
* Maintaining object history.

### Dependency

This step depends on:

* Step 3: The current versioning status has been checked.
* The existing S3 bucket from Assignment 1.
* Permission to modify the bucket's versioning configuration.

### What happens if this step is skipped?

If versioning remains disabled:

* Re-uploading `document.txt` will overwrite the existing object.
* Re-uploading `data.csv` will overwrite the existing object.
* Separate version IDs will not be generated as required.
* Previous object content will not be retained as versioned history.
* The core objective of Assignment 2 will fail.

---

## Step 5: Verify That Bucket Versioning Is Enabled

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Bucket Versioning
```

### Configuration

Confirm:

* **Bucket Versioning:** `Enabled`

### Expected Output

The AWS Console should show:

```text
Bucket Versioning
Enabled
```

### Why is this step required?

The two files must be re-uploaded only after versioning is enabled.

The required sequence is:

```text
Original objects already exist
        |
        v
Enable S3 Versioning
        |
        v
Confirm Versioning = Enabled
        |
        v
Re-upload the same object keys
        |
        v
New object versions are created
```

Verifying the setting prevents accidental re-upload before versioning is active.

### Dependency

This step depends on:

* Step 4: S3 Versioning must have been enabled successfully.

### What happens if this step is skipped?

If you immediately re-upload files without confirming the setting:

* A failed or unsaved configuration change could go unnoticed.
* The expected versioning behavior may not occur.
* Troubleshooting becomes more difficult because the prerequisite state was not verified.

---

## Step 6: Modify the First Existing File Before Re-uploading

### Navigation

On your local Windows computer:

```text
Windows File Explorer
→ Documents
→ S3-Assignment-1-Files
→ Open document.txt
```

### Configuration

Open `document.txt` in a text editor such as Notepad.

Suppose the original file contained:

```text
This is the original document uploaded in Assignment 1.
```

Change its content to:

```text
This is the updated document uploaded in Assignment 2 to test S3 Versioning.
```

Save the file.

Keep the exact same filename:

* **File Name:** `document.txt`

Do not rename it to:

* `document-v2.txt`
* `new-document.txt`
* `document-updated.txt`

The same object key must be used to create another version of the existing S3 object.

### Commands

No terminal command is required because the file is being edited locally using a text editor.

### Why is this step required?

Technically, re-uploading a file with the same object key after versioning is enabled can create a new object version even if the content is unchanged.

However, changing the content makes verification much clearer.

The version history becomes logically distinguishable:

```text
document.txt
│
├── Current Version
│   └── "This is the updated document uploaded in Assignment 2..."
│
└── Previous Version
    └── "This is the original document uploaded in Assignment 1."
```

This allows you to verify not only that multiple version IDs exist, but also that different historical content can be retained.

### Dependency

This step depends on:

* The original `document.txt` file being available locally.
* Step 4: S3 Versioning having been enabled.

### What happens if this step is skipped?

If the file is re-uploaded without modifying its content:

* S3 Versioning can still create a new version.
* However, both versions may contain identical data.
* It becomes harder to visually demonstrate the practical benefit of retaining previous versions.

---

## Step 7: Modify the Second Existing File Before Re-uploading

### Navigation

On your local Windows computer:

```text
Windows File Explorer
→ Documents
→ S3-Assignment-1-Files
→ Open data.csv
```

### Configuration

Suppose the original `data.csv` contains:

```text
id,name,department
1,Alice,Development
2,Bob,Operations
```

Update it to:

```text
id,name,department
1,Alice,Development
2,Bob,Operations
3,Charlie,DevOps
```

Save the file using the same filename:

* **File Name:** `data.csv`

Do not rename the file.

### Commands

No terminal command is required because the CSV file is being edited locally.

### Why is this step required?

Changing the second file provides another clear demonstration of object versioning.

After re-upload, the conceptual version history will be:

```text
data.csv
│
├── Current Version
│   └── Contains Alice, Bob, and Charlie
│
└── Previous Version
    └── Contains only Alice and Bob
```

This demonstrates that Amazon S3 can preserve previous object states when the same object key is updated.

### Dependency

This step depends on:

* The original `data.csv` file being available locally.
* Step 4: S3 Versioning having been enabled.

### What happens if this step is skipped?

If the second file is not prepared:

* Only one object may be re-uploaded.
* Task 2 explicitly requires re-uploading any two previously uploaded files.
* The assignment would remain incomplete.

---

## Step 8: Re-upload the Two Modified Files

### Navigation

Open the existing S3 bucket:

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

Navigate to the local folder:

```text
S3-Assignment-1-Files
```

Select the following two modified files:

* `document.txt`
* `data.csv`

Confirm that both files appear in the **Files and folders** section.

The expected selection is:

| Object Number | Object Name    | Existing in S3 | Modified Locally | Same Object Key |
| ------------- | -------------- | -------------- | ---------------- | --------------- |
| 1             | `document.txt` | Yes            | Yes              | Yes             |
| 2             | `data.csv`     | Yes            | Yes              | Yes             |

Keep the default upload settings.

Then select:

```text
Upload
```

Wait for the upload operation to complete.

The AWS Console should display a successful upload status for both objects.

After successful upload, select:

```text
Close
```

### Commands

No terminal command is required because the files are being uploaded through the AWS Management Console.

### Why is this step required?

This is the step that creates new versions of the two existing objects.

The following conditions are now true:

1. `document.txt` already exists in the bucket.
2. `data.csv` already exists in the bucket.
3. S3 Versioning is enabled.
4. Both files are uploaded again using exactly the same object keys.

Therefore, Amazon S3 creates new versions rather than permanently replacing the previous object data.

Conceptually:

```text
Before re-upload:

document.txt
└── Original object

data.csv
└── Original object


After re-upload with Versioning enabled:

document.txt
├── New version       → Current
└── Original object   → Previous version

data.csv
├── New version       → Current
└── Original object   → Previous version
```

### Dependency

This step depends on:

* Step 2: The original objects must already exist.
* Step 4: S3 Versioning must be enabled.
* Step 5: Versioning should be confirmed as enabled.
* Step 6: `document.txt` should be prepared for re-upload.
* Step 7: `data.csv` should be prepared for re-upload.

### What happens if this step is skipped?

If the two files are not re-uploaded:

* No new versions will be created for those object keys.
* You cannot demonstrate the required versioning behavior.
* Task 2 of the assignment will remain incomplete.

---

# 5. Verification / Output Checking

After completing the implementation, verify:

1. S3 Versioning is enabled on the existing bucket.
2. The two selected files were successfully re-uploaded.
3. Multiple versions are visible for the two object keys.
4. Different version IDs exist.
5. The latest versions are marked as current versions.
6. Previous object states have been retained.

---

## Verification Step 1: Verify That Bucket Versioning Is Enabled

### Action

Open the bucket's `Properties` tab and inspect the Bucket Versioning setting.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Bucket Versioning
```

### Expected Output

The AWS Console should show:

```text
Bucket Versioning
Enabled
```

### Why does this confirm success?

This confirms successful completion of Task 1:

> Enable versioning for the bucket created in task 1.

With versioning enabled, new PUT operations using an existing object key can create additional object versions with unique version IDs.

---

## Verification Step 2: Verify the Current Object List

### Action

Return to the bucket's `Objects` tab.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Expected Output

With version display turned off, the bucket should still show five current objects:

```text
config.json
data.csv
document.txt
image.jpg
page.html
```

You should not expect seven separate object names simply because two files were re-uploaded.

The reason is that the re-uploaded files use the same object keys:

* `document.txt`
* `data.csv`

Without showing versions, S3 displays only the current version of each object key.

### Why does this confirm success?

This confirms an important S3 Versioning concept.

Re-uploading:

```text
document.txt
```

does not create another visible object named:

```text
document-copy.txt
```

Instead, S3 stores another version under the same object key.

Conceptually:

```text
Object Key: document.txt

Version A → Previous version
Version B → Current version
```

Therefore, the normal object view continues to show one `document.txt` entry.

---

## Verification Step 3: Enable the Show Versions Option

### Action

On the bucket's `Objects` tab, enable the option to display object versions.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Expected Output

After enabling `Show versions`, multiple entries should be visible for:

* `document.txt`
* `data.csv`

A conceptual representation is:

```text
document.txt
├── Current version
└── Previous version

data.csv
├── Current version
└── Previous version

config.json
└── Original object version/state

image.jpg
└── Original object version/state

page.html
└── Original object version/state
```

> **Important:** Because the original five objects were uploaded before S3 Versioning was enabled, their original versions may display a version ID of `null`. After versioning is enabled and the same object keys are uploaded again, the new versions receive unique, non-null version IDs.

### Why does this confirm success?

Multiple entries for the same object key demonstrate that Amazon S3 retained previous object states instead of permanently overwriting them.

This directly verifies the purpose of S3 Versioning.

---

## Verification Step 4: Verify Multiple Versions of `document.txt`

### Action

With `Show versions` enabled, locate all entries for:

* `document.txt`

### Expected Output

You should see two versions associated with the same object key.

Conceptual example:

```text
Object Key: document.txt

Version 1
Version ID: null
Type: Previous version
Created before versioning was enabled

Version 2
Version ID: 3HL4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY
Type: Current version
Created after versioning was enabled
```

> **Important:** The version ID shown above is only an example. AWS generates a unique version ID automatically. Your actual value will be different.

A conceptual table is:

| Object Key     | Version ID                      | Current Version | Meaning                                                |
| -------------- | ------------------------------- | --------------- | ------------------------------------------------------ |
| `document.txt` | `null`                          | No              | Original object uploaded before versioning was enabled |
| `document.txt` | Unique AWS-generated version ID | Yes             | New version created after versioning was enabled       |

### Why does this confirm success?

The presence of two stored versions under the same `document.txt` object key confirms that:

* The original object was retained.
* The re-upload created a new version.
* The new version has a unique version ID.
* The latest version is the current version.

This directly demonstrates successful S3 Versioning behavior.

---

## Verification Step 5: Verify Multiple Versions of `data.csv`

### Action

With `Show versions` enabled, locate all entries for:

* `data.csv`

### Expected Output

You should see two versions associated with the same object key.

Conceptual example:

```text
Object Key: data.csv

Version 1
Version ID: null
Type: Previous version
Created before versioning was enabled

Version 2
Version ID: Yk3rT8exampleUniqueVersionId9Kx
Type: Current version
Created after versioning was enabled
```

The exact version ID will be different because AWS generates it automatically.

A conceptual table is:

| Object Key | Version ID                      | Current Version | Meaning                                                |
| ---------- | ------------------------------- | --------------- | ------------------------------------------------------ |
| `data.csv` | `null`                          | No              | Original object uploaded before versioning was enabled |
| `data.csv` | Unique AWS-generated version ID | Yes             | Updated version uploaded after versioning was enabled  |

### Why does this confirm success?

This confirms that:

* `data.csv` existed before the re-upload.
* Versioning was active during the new upload.
* S3 retained the previous object state.
* S3 created a new current version with a unique version ID.

Together, `document.txt` and `data.csv` satisfy Task 2 of the assignment.

---

## Verification Step 6: Verify the Version ID of the Current `document.txt` Object

### Action

Turn off `Show versions` if necessary and open the current `document.txt` object.

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

The object details page should show information including:

```text
Object name: document.txt
Version ID: <unique-AWS-generated-version-ID>
Storage class: Standard
```

The actual version ID will be unique.

### Why does this confirm success?

In an S3 version-enabled bucket, newly uploaded objects receive unique version IDs.

A non-null version ID on the newly uploaded `document.txt` confirms that the object was uploaded after versioning became active.

---

## Verification Step 7: Verify the Version ID of the Current `data.csv` Object

### Action

Open the current `data.csv` object.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ data.csv
```

### Expected Output

The object details page should show:

```text
Object name: data.csv
Version ID: <unique-AWS-generated-version-ID>
Storage class: Standard
```

### Why does this confirm success?

The unique version ID confirms that Amazon S3 created a versioned object during the re-upload.

The two objects now demonstrate the required versioning behavior:

```text
document.txt → New unique version ID
data.csv     → New unique version ID
```

---

## Verification Step 8: Verify Previous and Current Content of `document.txt`

### Action

Enable `Show versions` and identify both versions of `document.txt`.

Open or download the previous version and the current version separately.

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
→ Select a specific document.txt version
```

### Expected Output

The previous version should contain the original Assignment 1 content, for example:

```text
This is the original document uploaded in Assignment 1.
```

The current version should contain the updated Assignment 2 content:

```text
This is the updated document uploaded in Assignment 2 to test S3 Versioning.
```

### Why does this confirm success?

This is one of the strongest practical demonstrations of S3 Versioning.

It proves that:

* The previous object data was retained.
* The updated data became the current version.
* Both object states are independently accessible through their version IDs.
* Re-uploading the same object key did not permanently destroy the previous data.

---

## Verification Step 9: Verify Previous and Current Content of `data.csv`

### Action

Enable `Show versions` and identify both versions of `data.csv`.

Open or download each version separately.

### Expected Output

The previous version should contain data similar to:

```text
id,name,department
1,Alice,Development
2,Bob,Operations
```

The current version should contain:

```text
id,name,department
1,Alice,Development
2,Bob,Operations
3,Charlie,DevOps
```

### Why does this confirm success?

The difference between the previous and current CSV content demonstrates that:

* S3 retained the earlier object state.
* The updated file became the current version.
* Multiple versions exist under the same object key.

This completes verification of the second re-uploaded object.

---

# 6. Cleanup / Cost Optimization

S3 Versioning requires careful cleanup.

Deleting a versioned object from the normal object view does not necessarily permanently remove all stored versions. In many cases, S3 creates a **delete marker**, while older object versions continue to exist and consume storage.

Therefore, the cleanup must permanently delete:

* All versions of `document.txt`.
* All versions of `data.csv`.
* The original objects or versions of `image.jpg`.
* The original objects or versions of `config.json`.
* The original objects or versions of `page.html`.
* Any delete markers, if present.
* The empty S3 bucket.

The correct cleanup dependency order is:

```text
Enable Show versions
        |
        v
Select every object version
        |
        v
Select every delete marker, if any
        |
        v
Permanently delete all versions and delete markers
        |
        v
Verify bucket is completely empty
        |
        v
Delete the empty S3 bucket
        |
        v
Verify bucket deletion
```

> **Important:** If you need this same bucket for another subsequent assignment, do not perform the final bucket deletion until the dependent assignments are completed.

---

## Cleanup Step 1: Open the Versioned Object List

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Action

Enable `Show versions`.

Confirm that all stored object versions are visible.

You should expect entries associated with:

* `document.txt`
* `data.csv`
* `image.jpg`
* `config.json`
* `page.html`

For `document.txt` and `data.csv`, multiple versions should exist.

### Why is this required?

The normal object view may display only current objects and can hide previous versions.

To completely clean up a versioned bucket, all versions must be visible so they can be permanently deleted.

### What happens if this resource is not deleted?

This step does not delete a resource itself.

However, if previous versions are not displayed and permanently removed:

* Noncurrent versions may remain stored.
* Storage usage may continue.
* The bucket may not be truly empty.
* Bucket deletion may fail.

---

## Cleanup Step 2: Permanently Delete All Object Versions

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Action

Select every displayed version for all five object keys.

This includes:

* Every version of `document.txt`.
* Every version of `data.csv`.
* Every version or original object state of `image.jpg`.
* Every version or original object state of `config.json`.
* Every version or original object state of `page.html`.
* Any delete markers, if present.

Select:

```text
Delete
```

The AWS Console will open the permanent deletion confirmation page.

Follow the displayed confirmation instructions. Depending on the current console workflow, you may be required to enter:

```text
permanently delete
```

Then confirm the permanent deletion.

Wait until AWS confirms that all selected versions were deleted successfully.

### Why is this required?

Each stored S3 object version can consume storage.

For example:

```text
document.txt
├── Current version      → Consumes storage
└── Previous version     → Also consumes storage
```

Deleting only the current object from the normal object view may create a delete marker rather than permanently deleting all historical versions.

Permanent version deletion ensures that the data is actually removed.

### What happens if this resource is not deleted?

If previous versions remain:

* They continue consuming S3 storage.
* Applicable charges may continue after allowances or credits are exhausted.
* The bucket may not be considered empty.
* Bucket deletion may fail.

---

## Cleanup Step 3: Verify That No Object Versions or Delete Markers Remain

### Navigation

```text
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Action

Confirm that no object versions or delete markers remain.

### Expected Result

The object list should be empty.

There should be no remaining entries for:

```text
document.txt
data.csv
image.jpg
config.json
page.html
```

There should also be no delete markers.

### Why is this required?

A version-enabled S3 bucket can appear empty in the normal object view while still containing:

* Noncurrent object versions.
* Delete markers.

Using `Show versions` confirms whether the bucket is genuinely empty.

### What happens if this resource is not deleted?

This is a verification step rather than a resource deletion step.

If versions or delete markers remain:

* Storage usage may continue.
* The bucket deletion operation may fail.
* Cleanup is incomplete.

---

## Cleanup Step 4: Delete the Empty S3 Bucket

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

When prompted, enter the exact bucket name:

```text
xyz-corporation-file-storage-shabdali-2026
```

Then select:

```text
Delete bucket
```

Wait for the deletion operation to complete.

### Why is this required?

The S3 bucket was used for Assignment 1 and Assignment 2.

If no subsequent assignment depends on it, deleting the bucket:

* Removes the unused AWS resource.
* Keeps the AWS account organized.
* Prevents accidental future uploads.
* Completes the resource lifecycle for the practical exercise.

### What happens if this resource is not deleted?

If the empty bucket remains:

* The bucket continues to exist in your AWS account.
* Its globally unique name remains associated with the bucket.
* Future accidental uploads could generate storage and request usage.
* Unused resources can make the AWS account harder to manage.

An empty general purpose S3 bucket itself does not normally create a meaningful storage charge, but unused resources should still be removed when no longer required.

---

## Cleanup Step 5: Verify That the S3 Bucket Has Been Deleted

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

This confirms that:

* All object versions were permanently deleted.
* Any delete markers were removed.
* The S3 bucket was successfully deleted.
* No AWS resources created for Assignment 1 and Assignment 2 remain active.

### What happens if this resource is not deleted?

This is a final verification step rather than a separate resource deletion operation.

If the bucket still appears:

* Confirm that every object version has been permanently deleted.
* Confirm that no delete markers remain.
* Retry the bucket deletion.
* Verify that your IAM identity has sufficient permission to delete the bucket.

---
