# Module 6: Assignment 2 - Amazon S3 Bucket Versioning and Object Recovery

## Problem Statement
XYZ Corporation requires data redundancy controls to safeguard corporate storage assets against accidental overwrites or unintended deletions. Implement Bucket Versioning on an existing Amazon S3 bucket, re-upload matching files with updated data components, and verify that AWS systematically tracks multi-generational historical states for the same object identifier.

---

## Tasks To Be Performed
1. Enable versioning for the S3 bucket created in Task 1.
2. Re-upload any 2 files already uploaded to verify if versioning works successfully.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Enable S3 Bucket Versioning
1. Log in to the **AWS Management Console** and open the **Amazon S3** console.
2. In the Left navigation or Buckets directory list, click on your previously created bucket (e.g., `xyz-corporate-storage-bucket-[yourname]`).
3. Click on the **Properties** tab located below the main bucket summary pane.
4. Locate the **Bucket Versioning** card section.
5. Click on the **Edit** button on the right.
6. Change the setting status from *Suspended/Disabled* to **Enable**.
7. Click **Save changes** to enforce the historical state tracking policy.

---

### Step 2: Modify Local Files and Re-upload to Verify
1. Open any **2 files** from your desktop that you uploaded during Assignment 1 (e.g., `document.txt` and `report.pdf`).
2. Make a minor modification to their content (e.g., add a line reading: `"Version 2 Update - Verified Content Data Change"`) and save the files locally.
3. Return to the AWS S3 console interface and click on the **Objects** tab of your bucket.
4. Observe the **Show versions** slider toggle positioned right above the objects table. (By default, it is turned *OFF*, displaying only the latest iteration).
5. Click the orange **Upload** button.
6. Click **Add files**, choose the updated `document.txt` and `report.pdf` from your machine, and click open.
7. Click **Upload** and wait for the successful confirmation message banner. Click **Close**.

---

### Step 3: Verify Versioning Mechanism History
1. Back on the **Objects** tab layout, turn the **Show versions** toggle switch to **ON**.
2. Examine the objects list hierarchy grid:
   * You will now see multiple row entries for `document.txt` and `report.pdf`.
   * The top row will list the **Latest version** with its distinct Version ID string.
   * The row directly beneath it will display the older timeline state representing the initial file uploaded in Assignment 1, alongside its unique tracking identifier.
3. This architecture confirms that Versioning is actively preserving historical states without destroying data.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

When bucket versioning is active, deleting an object normally just inserts a "Delete Marker," keeping the older versions intact and consuming storage. Follow this absolute workflow to wipe versioned assets safely:

### 1. Permanently Delete Versioned Objects
1. Open your S3 bucket -> **Objects** tab.
2. Toggle **Show versions** to **ON**.
3. Check the top structural checkbox to select **all variations and delete markers** of the files.
4. Click the **Delete** button.
5. Because versioning is active, AWS forces strict compliance: type `permanently delete` in the authorization confirmation field and click **Delete objects**.

### 2. Remove the Empty Bucket Target
1. Exit back to the main **Amazon S3 > Buckets** dashboard window.
2. Select the empty bucket checkbox container.
3. Click **Delete**, type the exact unique bucket identity string name, and click **Delete bucket** to clear the component completely.
