# Module 6: Case Study - Comprehensive S3 Lifecycle, Versioning, and Web Hosting

## Problem Statement
XYZ Corporation is finalizing its storage migration strategy to AWS. The company requires an infinitely scalable, secure object storage ecosystem that enforces automatic file retention parameters (purge after 75 days), tracks historical document states to protect against accidental corruption (Versioning), and operates a low-cost corporate static storefront utilizing custom domain mappings along with fallback error web targets.

---

## Tasks To Be Performed
1. Ensure infinite data capacity and anywhere-anytime global retrieval across the cloud.
2. Configure an automated Lifecycle Rule to permanently delete all objects after 75 days.
3. Establish a recovery workflow to retrieve older versions of a file if current iterations are accidentally compromised.
4. Host a corporate static website via S3 using the custom domain name established in previous modules.
5. Deploy a custom error fallback page if incorrect path structures or domains are target routed.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create an S3 Bucket and Enable Identity Controls
1. Log in to the **AWS Management Console** and navigate to the **Amazon S3** console.
2. Click **Create bucket**.
3. **Bucket name:** Enter a globally unique name matching your company domain or deployment standard (e.g., `xyzcorp-production-storage-[yourname]`).
4. **Object Ownership:** Leave **ACLs disabled** (enforces Bucket Policies for security governance).
5. **Block Public Access:** Uncheck **Block *all* public access** *(Required since this specific bucket will host a public-facing static website).*
6. Check the acknowledgment box warning about public exposures. Click **Create bucket**.

---

### Step 2: Enforce Security Policies and Versioning Controls
1. Click on the newly created bucket and open the **Properties** tab.
2. Locate **Bucket Versioning**, click **Edit**, switch it to **Enable**, and save changes. *(Fulfills Task 3 requirement to safeguard compromised file data).*
3. Open the **Permissions** tab, scroll to **Bucket policy**, click **Edit**, and paste the public read capability rule (Replace `your-bucket-name` with your actual bucket name string):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::your-bucket-name/*"
           }
       ]
   }
   ```

4. Click **Save changes**.

---

### Step 3: Configure Automated Data Lifecycle Expiration

1. Switch to the **Management** tab of the bucket configuration workspace.
2. Under the *Lifecycle rules* window panel, click **Create lifecycle rule**.
3. Configure expiration properties:
* **Lifecycle rule name:** `XYZ-Automatic-75Day-Purge`
* **Rule scope:** Select **Apply to all objects in the bucket** and check the confirmation box.


4. **Lifecycle rule actions:** Check **Expire current versions of objects** and check **Permanently delete noncurrent versions of objects**.
5. **Configure timelines (Task 2 Requirement):**
* Under *Expire current versions of objects* -> **Days after object creation:** Type `75`.
* Under *Permanently delete noncurrent versions of objects* -> **Days after objects become noncurrent:** Type `75`.


6. Click **Create rule**.

---

### Step 4: Host Static Website and Map Custom Route 53 Domain

1. Create two web documents locally on your computer:
* **`index.html`**: Code a baseline layout reading: `<h1>XYZ Corporation Production Enterprise Cloud Infrastructure Portal</h1>`
* **`error.html`**: Code a baseline layout reading: `<h1>Error 404: Invalid Access Path - Please use a valid corporate namespace link.</h1>`


2. Go to the **Objects** tab, click **Upload**, add both files, and submit.
3. Switch to the **Properties** tab -> scroll to the bottom -> click **Edit** on **Static website hosting**.
4. Set status to **Enable**, assign `index.html` as the index target, assign `error.html` as the error target document, and click **Save changes**.
5. **Route 53 Custom Domain Mapping (Task 4 & 5 Requirement):**
* Open the **Route 53** dashboard -> Go to **Hosted zones** -> Select your corporate domain file registry.
* Click **Create record** -> Toggle **Alias** to **ON**.
* **Route traffic to:** Choose *Alias to S3 website endpoint*. Select the hosting region matching your bucket, choose your S3 bucket target domain, and execute creation.

---

### Step 5: Verification of Compromised File Recovery (Task 3 Validation)

1. Upload a file called `config.txt` containing text: `"Secure Production Tokens v1"`.
2. Simulate an accidental compromise or overwrite by saving a blank text document over `config.txt` and uploading it again.
3. To recover, open the S3 bucket -> **Objects** tab -> toggle **Show versions** to **ON**.
4. Select the top row entry (the compromised empty version) and click **Delete**.
5. Confirm deletion. The bucket will immediately surface the older version 1 file back to the active production layer.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To prevent clutter or automated lifecycle runs on accidental datasets, wipe the configurations using this order:

### 1. Remove Route 53 Mapping and S3 Rules

1. Open **Route 53**, delete the **Alias A record** linking your custom domain to the S3 web endpoint.
2. Go to your S3 bucket -> **Management** tab -> select `XYZ-Automatic-75Day-Purge` -> Click **Delete**.
3. Go to the **Properties** tab -> scroll to **Static website hosting** -> Click **Edit** -> Choose **Disable** -> Save changes.

### 2. Wipe Bucket Objects and Containers

1. Go to the **Objects** tab -> toggle **Show versions** to **ON**.
2. Select all objects, directories, delete markers, and historical versions by checking the top selection boundary box.
3. Click **Delete**, type `permanently delete` in the authorization modal, and confirm.
4. Exit back to the **S3 Buckets** homepage list. Select your empty bucket, click **Delete**, confirm by writing the unique name string, and execute permanent deletion.

