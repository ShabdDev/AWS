# Module 6: Assignment 3 - S3 Static Website Hosting and Lifecycle Management

## Problem Statement
XYZ Corporation aims to optimize cloud costs by hosting a low-overhead static corporate website directly via object storage instead of provisioning full compute servers. Implement Amazon S3 Static Website Hosting with public read capabilities, deploy essential fallback document endpoints (`index.html` and `error.html`), and enforce an automated Lifecycle Policy to systematically tier aging compliance files to Standard-IA after 60 days and execute final deletion after 200 days.

---

## Tasks To Be Performed
1. Use the created S3 bucket to host a static website by uploading an `index.html` file and an `error.html` page.
2. Add a Lifecycle Rule for the bucket:
   * **Transition:** Standard storage class to Standard-Infrequent Access (Standard-IA) in 60 days.
   * **Expiration:** Permanently delete the objects after 200 days.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Configure Bucket Permissions for Public Access
*(Static website hosting requires public read permissions so external users can load the site).*
1. Log in to the **AWS Management Console** and open the **Amazon S3** console.
2. Click on your active bucket name (e.g., `xyz-corporate-storage-bucket-[yourname]`).
3. Click on the **Permissions** tab.
4. Under **Block public access (bucket settings)**, click **Edit**.
5. **Uncheck** the box for *Block all public access*. Click **Save changes** and type `confirm` in the security verification box.
6. Scroll down to **Bucket policy** and click **Edit**. Paste the following public read JSON policy (replace `your-bucket-name` with your actual bucket name):
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

7. Click **Save changes**.

---

### Step 2: Upload Website Documents and Enable Hosting

1. Create two files locally on your desktop using notepad:
* **`index.html`**: Enter `<h1>Welcome to XYZ Corp Static Web Homepage!</h1>`
* **`error.html`**: Enter `<h1>Error 404: Page Not Found - XYZ Corp Cloud Security Gateway.</h1>`


2. Go to the **Objects** tab of your bucket, click **Upload**, add both files, and execute the upload.
3. Switch over to the **Properties** tab.
4. Scroll to the very bottom to find the **Static website hosting** card. Click **Edit**.
5. Configure static web parameters:
* **Static website hosting:** Select **Enable**.
* **Hosting type:** Host a static website.
* **Index document:** Type `index.html`
* **Error document:** Type `error.html`


6. Click **Save changes**.
7. Scroll back to the **Static website hosting** section. S3 has now generated a unique public URL endpoint (e.g., `http://your-bucket-name.s3-website-us-east-1.amazonaws.com`). Click it to verify your website loads successfully.

---

### Step 3: Implement Lifecycle Management Rules

1. Click on the **Management** tab located under the bucket headers overview pane.
2. Under the *Lifecycle rules* card grid, click **Create lifecycle rule**.
3. Configure the lifecycle tracking metadata:
* **Lifecycle rule name:** `XYZ-Data-Retention-Policy`
* **Rule scope:** Select **Apply to all objects in the bucket** (Check the acknowledgment box).


4. **Lifecycle rule actions:**
* Check **Transition current versions of objects between storage classes**.
* Check **Expire current versions of objects**.


5. **Configure Transitions:**
* Under Storage Class Transitions, select **Standard-Infrequent Access (Standard-IA)**.
* **Days after object creation:** Type `60`.


6. **Configure Expiration:**
* Under Expire current versions of objects -> **Days after object creation:** Type `200`.


7. Click **Create rule**. The automation policy is now actively enforced.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

Follow this order to properly reverse configurations and safely delete the S3 resources:

### 1. Disable Website Hosting & Remove Lifecycle Policies

1. Go to the **Management** tab, select `XYZ-Data-Retention-Policy`, click **Actions** > **Delete**, and confirm.
2. Go to the **Properties** tab, scroll to **Static website hosting**, click **Edit**, select **Disable**, and click **Save changes**.

### 2. Wipe Assets and Delete the Bucket Container

1. Go to the **Objects** tab, select all files (`index.html`, `error.html`), click **Delete**, type `permanently delete` into the input layout, and submit.
2. Go back to the main **Amazon S3 > Buckets** panel directory list.
3. Select the empty bucket, click **Delete**, type the exact unique string profile name of the bucket container to authorize clearance, and click **Delete bucket**.
