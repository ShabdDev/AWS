# Module 6: Assignment 1 - Amazon S3 Bucket Creation and Object Management

## Problem Statement
XYZ Corporation requires a highly scalable cloud storage solution to retain corporate files and assets, with the capabilities to securely store and conditionally share objects publicly. Implement Amazon Simple Storage Service (S3) to host resources and demonstrate bulk asset ingestion with varied document profiles.

---

## Tasks To Be Performed
1. Create an Amazon S3 Bucket for file storage.
2. Upload 5 objects with different file extensions into the created bucket.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create a Globally Unique S3 Bucket
1. Log in to the **AWS Management Console** and search for **S3** to open the Amazon S3 console.
2. On the S3 dashboard, click on the **Create bucket** button.
3. Configure the bucket creation parameters:
   * **Bucket name:** Enter a globally unique name (e.g., `xyz-corporate-storage-bucket-[yourname]`). *Note: Bucket names must contain only lowercase letters, numbers, hyphens, and must be unique across all AWS accounts globally.*
   * **AWS Region:** Select your preferred regional workspace (e.g., US East (N. Virginia) `us-east-1`).
   * **Object Ownership:** Keep **ACLs disabled (recommended)** to manage permissions via modern IAM policies.
4. **Block Public Access settings for this bucket:**
   * Leave **Block *all* public access** checked by default. *(This ensures maximum baseline security. If public sharing is needed in later tasks, specific objects or buckets can be unblocked systematically).*
5. **Bucket Versioning & Encryption:** Leave default parameters intact (Bucket Versioning: Disabled, Encryption: Managed by Amazon S3 managed keys - SSE-S3).
6. Click **Create bucket**. Your new bucket will appear in the directory panel list view.

---

### Step 2: Prepare and Upload 5 Distinct Objects
1. Create 5 sample mock files with different extensions on your local machine's desktop:
   * `document.txt` (Plain Text)
   * `image.png` or `image.jpg` (Image File)
   * `report.pdf` (PDF File)
   * `script.py` or `code.html` (Code Artifact)
   * `data.csv` (Spreadsheet/Data sheet)
2. In the Amazon S3 console, click directly on the name of your newly created bucket (`xyz-corporate-storage-bucket-[yourname]`).
3. Under the **Objects** tab view, click on the orange **Upload** button.
4. Click **Add files**, browse your local directory, select all 5 files (`.txt`, `.png`, `.pdf`, `.py`, `.csv`), and click open.
5. Review the files listed inside the upload dashboard grid to confirm that 5 distinct object extensions are detected.
6. Click the **Upload** button positioned at the bottom of the screen.
7. Wait for the green status banner to state **Upload succeeded**. Click **Close**.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

While 5 GB of standard S3 storage is completely free under the AWS Free Tier, it is best practice to clean up test datasets to avoid future cross-tier management leakage:

### 1. Empty the S3 Bucket (Deletes underlying objects)
1. Navigate back to the main **Amazon S3 > Buckets** directory list.
2. Select the checkbox next to `xyz-corporate-storage-bucket-[yourname]`.
3. Click on the **Empty** button at the top of the interface.
4. Type `permanently delete` in the validation text box to verify authorization, and click **Empty**.

### 2. Delete the S3 Bucket Template
1. Once the bucket is empty, click **Exit** to return to the bucket directory.
2. Select the empty bucket checkbox again.
3. Click on the **Delete** button.
4. Confirm your absolute deletion decision by typing the exact name of the target bucket (`xyz-corporate-storage-bucket-[yourname]`) into the safety validation prompt module.
5. Click **Delete bucket**.
