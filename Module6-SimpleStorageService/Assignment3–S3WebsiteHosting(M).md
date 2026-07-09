# Module 6 - Simple Storage Service (S3)

# Assignment 3 - S3 Website Hosting

## Problem Statement

You work for XYZ Corporation. Their application requires a storage service that can store files and publicly share them if required. Implement Amazon Simple Storage Service (Amazon S3) for the same.

## Tasks To Be Performed

1. Use the bucket created in the previous assignment to host a static website and upload:

   * `index.html`
   * `error.html`
2. Add a lifecycle rule for the bucket:

   * Transition objects from `S3 Standard` to `S3 Standard-IA` after `60 days`.
   * Expire objects after `200 days`.

---

# 1. Free Tier / Cost Check

This assignment reuses the existing S3 general purpose bucket created in Assignment 1 and version-enabled in Assignment 2.

The assignment introduces two additional Amazon S3 features:

* S3 static website hosting.
* S3 Lifecycle configuration.

| AWS Service / Feature              | Free Tier Eligible                                                     | Uses Credits     | Notes                                                                                 |
| ---------------------------------- | ---------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------- |
| Existing S3 General Purpose Bucket | Yes, subject to the AWS account's applicable Free Tier plan and limits | Possibly         | The existing bucket is reused; no additional bucket is required.                      |
| S3 Standard Storage                | Yes, within applicable Free Tier limits                                | Possibly         | `index.html` and `error.html` are initially stored in S3 Standard.                    |
| S3 Static Website Hosting          | No separate hosting fee                                                | Indirectly       | Charges can arise from object storage, requests, and data transfer.                   |
| S3 GET Requests                    | Yes, within applicable Free Tier limits                                | Possibly         | Opening the website endpoint generates GET requests.                                  |
| S3 PUT Requests                    | Yes, within applicable Free Tier limits                                | Possibly         | Uploading `index.html` and `error.html` generates PUT requests.                       |
| S3 Lifecycle Configuration         | No separate charge merely for creating a lifecycle rule                | Indirectly       | Lifecycle transitions and storage classes can generate applicable charges.            |
| S3 Standard-IA                     | Chargeable storage class, subject to applicable pricing                | Yes, if billable | Standard-IA has a minimum storage duration charge and a minimum billable object size. |
| Lifecycle Transition Requests      | Not necessarily free                                                   | Possibly         | Lifecycle transitions can generate per-request charges.                               |
| Data Transfer Out to the Internet  | Free within applicable allowances; chargeable beyond applicable limits | Possibly         | Public visitors accessing the website can generate outbound data transfer.            |

### Is this assignment completely Free Tier eligible?

The assignment can generally be completed with extremely low usage because it requires only:

* One existing S3 bucket.
* Two small HTML objects.
* A small number of S3 requests.
* One lifecycle rule.
* Limited website testing.

However, it should not be described as universally or unconditionally free because actual charges depend on:

* The AWS account's applicable Free Tier plan.
* Existing account usage.
* Number of requests.
* Data transferred out.
* Lifecycle transitions.
* Standard-IA storage behavior.
* Retained object versions.

### Which AWS services can consume credits?

Amazon S3 can consume promotional credits through:

* S3 Standard storage.
* S3 Standard-IA storage.
* PUT requests.
* GET requests.
* Lifecycle transition requests.
* Internet data transfer beyond applicable free allowances.
* Storage of current and noncurrent object versions.

### Is it better to complete this assignment while promotional credits are available?

Yes. Promotional credits can help cover eligible S3 usage.

For this particular assignment, the expected immediate cost is extremely small because:

* Only two small HTML files are uploaded.
* Website testing requires only a few GET requests.
* The lifecycle transition is configured for `60 days`.
* The expiration action is configured for `200 days`.

If all assignment resources are deleted immediately after verification, the lifecycle transition and expiration actions will never occur.

### Important Decision: Should We Change 60 Days and 200 Days to 1 Minute for Testing?

No.

Amazon S3 Lifecycle configuration does not support specifying transition or expiration periods in minutes.

The lifecycle rule uses day-based timing.

Therefore, configurations such as these are not valid lifecycle timings:

```text id="0bf85y"
Transition after: 1 minute
Expiration after: 1 minute
```

The assignment should be configured exactly as specified:

```text id="7wj0bn"
S3 Standard
    |
    | After 60 days
    v
S3 Standard-IA
    |
    | Object reaches age of 200 days
    v
Expiration
```

> **Important:** The `200 days` expiration value means the object expires when it reaches an age of 200 days. It does not mean 200 additional days after the 60-day transition.

Therefore:

```text id="jn3jbu"
Day 0   → Object created in S3 Standard
Day 60  → Transition to S3 Standard-IA
Day 200 → Object expires
```

The object spends approximately:

```text id="xgjjfr"
200 - 60 = 140 days
```

in S3 Standard-IA before reaching the configured expiration age.

### Why Not Use 1 Day Instead?

Although S3 Lifecycle rules can use day-based values, changing the assignment to `1 day` does not provide immediate verification.

Lifecycle actions are asynchronous. Objects do not necessarily transition or disappear exactly at midnight or at the precise moment their age reaches the configured value.

Additionally, the assignment explicitly requires:

* Transition after `60 days`.
* Expiration after `200 days`.

Therefore, the best verification method is:

1. Configure the exact required values.
2. Verify that the lifecycle rule is saved and enabled.
3. Inspect the lifecycle configuration in the AWS Console.
4. Delete the resources after verification to avoid unnecessary usage.

### Important Standard-IA Consideration for Small HTML Files

The `index.html` and `error.html` files used for a simple static website are normally very small.

For S3 Lifecycle transitions, small-object behavior is important. By default, objects smaller than `128 KB` may not transition using lifecycle rules unless the lifecycle configuration explicitly allows smaller objects through an object-size filter or equivalent supported configuration.

Therefore, for this assignment, the lifecycle rule should be understood as applying according to Amazon S3's lifecycle eligibility requirements.

The primary assignment objective is to correctly configure:

```text id="klj23a"
S3 Standard → 60 days → S3 Standard-IA → 200 days object age → Expiration
```

### Important Versioning Consideration

The bucket was version-enabled in Assignment 2.

This creates an important distinction:

* **Current version lifecycle actions** affect current object versions.
* When a current object version expires in a version-enabled bucket, Amazon S3 may add a delete marker instead of immediately permanently removing all historical versions.
* Existing noncurrent versions can remain stored unless separate noncurrent-version lifecycle actions are configured.

The assignment asks only for:

* Transition from Standard to Standard-IA after 60 days.
* Expiration after 200 days.

Therefore, this solution will configure the requested lifecycle actions for current object versions and will not add unrequested noncurrent-version expiration behavior.

During cleanup, all versions and delete markers will be permanently removed manually.

### Cost Optimization Decision for This Assignment

To minimize unnecessary AWS usage:

* Reuse the existing S3 bucket.
* Do not create a new bucket.
* Upload only `index.html` and `error.html`.
* Enable static website hosting only for the assignment.
* Configure exactly one lifecycle rule.
* Use the required `60-day` transition.
* Use the required `200-day` expiration.
* Do not wait 60 or 200 days for verification.
* Verify the saved lifecycle configuration directly.
* After testing, permanently delete all object versions and delete markers if no subsequent assignment depends on the bucket.
* Delete the empty bucket if it is no longer required.

---

# 2. Architecture and Resource Dependency Flow

This assignment depends directly on the existing S3 bucket from Assignments 1 and 2.

The starting architecture is:

```text id="bl7jfp"
AWS Account
    |
    v
Amazon S3
    |
    v
Existing General Purpose Bucket
    |
    +-- Bucket Versioning: Enabled
    |
    +-- Existing objects from previous assignments
```

After completing this assignment:

```text id="b5rmxq"
                           Internet User
                                 |
                                 | HTTP GET Request
                                 v
                    S3 Static Website Endpoint
                                 |
                                 v
              Existing S3 General Purpose Bucket
                    /                    \
                   /                      \
                  v                        v
           index.html                  error.html
           Index document              Error document


Lifecycle Rule:

Object Created
     |
     v
S3 Standard
     |
     | Object age = 60 days
     v
S3 Standard-IA
     |
     | Object age = 200 days
     v
Expiration Action
```

The exact implementation dependency order is:

```text id="tm2ol9"
Existing Bucket from Assignment 2
        |
        v
Verify Existing Bucket
        |
        v
Create index.html Locally
        |
        v
Create error.html Locally
        |
        v
Upload Both HTML Files
        |
        v
Configure Public Read Access
        |
        v
Enable Static Website Hosting
        |
        v
Configure Index and Error Documents
        |
        v
Open Website Endpoint
        |
        v
Verify index.html
        |
        v
Verify error.html
        |
        v
Create Lifecycle Rule
        |
        v
Configure Standard → Standard-IA at 60 Days
        |
        v
Configure Expiration at 200 Days
        |
        v
Verify Lifecycle Rule
```

---

# 3. Important Prerequisite: Understand the Starting State

Because this assignment explicitly says:

> Use the created bucket in the previous task to host static websites.

we must reuse the bucket from Assignments 1 and 2.

The expected starting state is:

| Resource / Setting     | Expected State                                           |
| ---------------------- | -------------------------------------------------------- |
| Existing S3 bucket     | Must exist                                               |
| Bucket type            | General purpose                                          |
| Existing five objects  | Should still exist if previous cleanup was not performed |
| S3 Versioning          | Enabled from Assignment 2                                |
| Static website hosting | Disabled before this assignment                          |
| Block Public Access    | Enabled from Assignment 1                                |
| `index.html`           | Not yet uploaded                                         |
| `error.html`           | Not yet uploaded                                         |
| Lifecycle rule         | Not yet configured                                       |

For this solution, the example bucket remains:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`
* **AWS Region:** `Asia Pacific (Mumbai) ap-south-1`

If your actual bucket uses a different globally unique name, replace the example bucket name with your actual bucket name throughout the assignment.

> **Important:** Do not create a new S3 bucket for this assignment. The assignment specifically requires reuse of the bucket created previously.

---

# 4. Step-by-Step Solution

## Step 1: Verify That the Existing S3 Bucket Is Available

### Navigation

```text id="dz3xl8"
AWS Management Console
→ S3
→ General purpose buckets
```

### Configuration

Locate:

* **Bucket Name:** `xyz-corporation-file-storage-shabdali-2026`

Select the bucket name to open it.

### Commands

No terminal command is required because this step is performed through the AWS Management Console.

### Why is this step required?

Assignment 3 depends on the bucket created in Assignment 1 and version-enabled in Assignment 2.

Verifying the bucket ensures that:

* The correct existing resource is reused.
* No unnecessary additional bucket is created.
* The assignment dependency is preserved.
* Website hosting and lifecycle configuration are applied to the intended bucket.

### Dependency

This step depends on:

* Assignment 1 having created the S3 bucket.
* Assignment 2 having reused and version-enabled the same bucket.
* The bucket not having been deleted during previous cleanup.

### What happens if this step is skipped?

If the bucket is not verified:

* You may accidentally configure another bucket.
* You may unnecessarily create duplicate resources.
* The assignment requirement to reuse the previously created bucket may not be satisfied.

---

## Step 2: Verify the Current Bucket Versioning Status

### Navigation

```text id="7w00l6"
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

### Commands

No terminal command is required because this verification is performed through the AWS Management Console.

### Why is this step required?

The bucket was version-enabled during Assignment 2.

Understanding this state is important before uploading `index.html` and `error.html` because each upload into a version-enabled bucket receives a unique version ID.

It also affects lifecycle expiration and cleanup behavior.

### Dependency

This step depends on:

* Step 1: The correct existing S3 bucket must be available.
* Assignment 2: Versioning should have been enabled.

### What happens if this step is skipped?

The website can still be configured without separately checking versioning, but you may misunderstand:

* Why uploaded HTML objects receive version IDs.
* Why deleting an object can create a delete marker.
* Why previous versions may remain stored.
* Why cleanup of a versioned bucket requires additional care.

---

## Step 3: Create the `index.html` File

### Navigation

On your local Windows computer:

```text id="8imfpj"
Windows File Explorer
→ Documents
→ S3-Assignment-1-Files
→ Right-click
→ New
→ Text Document
```

Rename the file:

```text id="7xtv1e"
index.html
```

> **Important:** Ensure the actual filename is `index.html`, not `index.html.txt`. If Windows hides known file extensions, enable **File name extensions** in File Explorer before renaming the file.

### Configuration

Open `index.html` in Notepad or another text editor and add:

```html id="czb8la"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>XYZ Corporation</title>
</head>
<body>
    <h1>Welcome to XYZ Corporation</h1>
    <p>This static website is hosted using Amazon S3.</p>
    <p>Module 6 - Assignment 3: S3 Website Hosting</p>
</body>
</html>
```

Save the file.

### HTML Code Explanation

| Code Part                        | Explanation                                                                                     |
| -------------------------------- | ----------------------------------------------------------------------------------------------- |
| `<!DOCTYPE html>`                | Declares that the document uses HTML5.                                                          |
| `<html lang="en">`               | Starts the HTML document and declares English as its language.                                  |
| `<head>`                         | Contains metadata and configuration information that is not normally displayed as page content. |
| `<meta charset="UTF-8">`         | Sets UTF-8 character encoding.                                                                  |
| `<meta name="viewport" ...>`     | Helps the page render properly on mobile and responsive displays.                               |
| `<title>XYZ Corporation</title>` | Sets the browser tab title.                                                                     |
| `<body>`                         | Contains visible webpage content.                                                               |
| `<h1>`                           | Creates a primary heading.                                                                      |
| `<p>`                            | Creates a paragraph.                                                                            |
| `</body>`                        | Closes the visible page content section.                                                        |
| `</html>`                        | Ends the HTML document.                                                                         |

### Why is this step required?

Amazon S3 static website hosting requires an index document to serve as the default page when a visitor accesses the website root endpoint.

For this assignment:

* **Index document:** `index.html`

When a user opens the website endpoint without specifying a filename, S3 attempts to return `index.html`.

Conceptually:

```text id="0j0e7u"
User opens S3 website endpoint
        |
        v
Amazon S3 looks for index document
        |
        v
index.html found
        |
        v
Website homepage is displayed
```

### Dependency

This step depends on:

* A local computer.
* A text editor such as Notepad.

It does not yet depend on the S3 bucket because the file is being created locally.

### What happens if this step is skipped?

If `index.html` is not created and uploaded:

* The website will not have the required index document.
* Opening the root website endpoint will not display the intended homepage.
* Task 1 of the assignment will remain incomplete.

---

## Step 4: Create the `error.html` File

### Navigation

On your local Windows computer:

```text id="q4zcw3"
Windows File Explorer
→ Documents
→ S3-Assignment-1-Files
→ Right-click
→ New
→ Text Document
```

Rename the file:

```text id="nl57lr"
error.html
```

Ensure the actual filename is not `error.html.txt`.

### Configuration

Open `error.html` in Notepad or another text editor and add:

```html id="49yjhk"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Error - XYZ Corporation</title>
</head>
<body>
    <h1>404 - Page Not Found</h1>
    <p>The page you requested could not be found.</p>
    <p>Please return to the XYZ Corporation home page.</p>
</body>
</html>
```

Save the file.

### HTML Code Explanation

| Code Part                                | Explanation                                                      |
| ---------------------------------------- | ---------------------------------------------------------------- |
| `<!DOCTYPE html>`                        | Declares HTML5 document syntax.                                  |
| `<html lang="en">`                       | Starts the HTML document and identifies its language as English. |
| `<head>`                                 | Contains page metadata.                                          |
| `<meta charset="UTF-8">`                 | Configures UTF-8 character encoding.                             |
| `<meta name="viewport" ...>`             | Helps the page display correctly across screen sizes.            |
| `<title>Error - XYZ Corporation</title>` | Sets the browser tab title for the error page.                   |
| `<body>`                                 | Contains content displayed to the visitor.                       |
| `<h1>404 - Page Not Found</h1>`          | Displays the main error heading.                                 |
| `<p>`                                    | Creates explanatory paragraphs.                                  |
| `</body>`                                | Ends visible page content.                                       |
| `</html>`                                | Ends the HTML document.                                          |

### Why is this step required?

The assignment explicitly requires an `error.html` page.

When static website hosting is configured with:

* **Index document:** `index.html`
* **Error document:** `error.html`

Amazon S3 can return the configured error document when an appropriate website request results in an error.

Conceptually:

```text id="cuv3se"
User requests a missing page
        |
        v
Requested object does not exist
        |
        v
S3 website hosting detects error
        |
        v
error.html is returned as configured error document
```

### Dependency

This step depends only on:

* A local computer.
* A text editor.

### What happens if this step is skipped?

If `error.html` is not created:

* The required custom error page cannot be uploaded.
* The website cannot use the requested custom error document.
* Task 1 will remain incomplete.

---

## Step 5: Upload `index.html` and `error.html` to the Existing Bucket

### Navigation

```text id="7mv9pd"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Upload
→ Add files
```

### Configuration

Select:

* `index.html`
* `error.html`

Confirm that both files appear in the upload list.

| Object Number | Object Name  | Purpose                   |
| ------------- | ------------ | ------------------------- |
| 1             | `index.html` | Default website homepage  |
| 2             | `error.html` | Custom website error page |

Keep the default upload settings.

Then select:

```text id="tr0vtf"
Upload
```

Wait until the AWS Console confirms successful upload for both objects.

Then select:

```text id="59bdxg"
Close
```

### Commands

No terminal command is required because both files are uploaded through the AWS Management Console.

### Why is this step required?

Amazon S3 static website hosting serves objects stored inside the configured bucket.

Creating HTML files locally is not enough. They must be uploaded to S3 so the static website endpoint can retrieve them.

The dependency is:

```text id="g14qx2"
Local index.html
        |
        | Upload
        v
S3 object: index.html
        |
        v
Used as website index document


Local error.html
        |
        | Upload
        v
S3 object: error.html
        |
        v
Used as website error document
```

Because versioning is enabled, these uploaded objects receive version IDs.

### Dependency

This step depends on:

* Step 1: The existing bucket must exist.
* Step 3: `index.html` must exist locally.
* Step 4: `error.html` must exist locally.

### What happens if this step is skipped?

If the HTML files are not uploaded:

* S3 cannot serve them through the static website endpoint.
* The index document will be unavailable.
* The custom error document will be unavailable.
* Static website verification will fail.

---

## Step 6: Disable Block Public Access for the S3 Bucket

### Navigation

```text id="dx93jz"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Permissions
→ Block public access (bucket settings)
→ Edit
```

### Configuration

To allow the static website to be publicly accessible, clear:

* **Block all public access:** `Disabled`

This disables the following four Block Public Access settings for this bucket:

* **Block public access to buckets and objects granted through new access control lists (ACLs):** `Disabled`
* **Block public access to buckets and objects granted through any access control lists (ACLs):** `Disabled`
* **Block public access to buckets and objects granted through new public bucket or access point policies:** `Disabled`
* **Block public and cross-account access to buckets and objects through any public bucket or access point policies:** `Disabled`

Select:

```text id="a6vkw2"
Save changes
```

When the AWS Console asks for confirmation, enter:

```text id="q0dy6p"
confirm
```

Then select the confirmation button.

> **Important:** Disabling Block Public Access does not automatically make the objects public. It only allows a public bucket policy to take effect. A bucket policy granting public read access will be added in the next step.

### Commands

No terminal command is required because this configuration is performed through the AWS Management Console.

### Why is this step required?

The static website endpoint must be able to retrieve `index.html` and `error.html` without requiring the visitor to authenticate to AWS.

The access flow is:

```text id="fgq66s"
Internet User
      |
      v
S3 Website Endpoint
      |
      v
Anonymous GET Request
      |
      v
Bucket Policy Allows s3:GetObject
      |
      v
HTML Object Returned
```

However, if S3 Block Public Access prevents the public bucket policy from taking effect, anonymous users cannot retrieve the website files.

### Dependency

This step depends on:

* Step 1: The existing S3 bucket must exist.
* Step 5: The HTML objects should already be uploaded.

### What happens if this step is skipped?

If Block Public Access continues to prevent public bucket policies:

* The public-read bucket policy may be rejected or ineffective.
* Anonymous visitors cannot retrieve `index.html`.
* The static website endpoint may return `AccessDenied`.
* Public website verification will fail.

---

## Step 7: Add a Bucket Policy for Public Read Access

### Navigation

```text id="1tmh7r"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Permissions
→ Bucket policy
→ Edit
```

### Configuration

Add the following bucket policy:

```json id="z0hfso"
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::xyz-corporation-file-storage-shabdali-2026/*"
        }
    ]
}
```

> **Important:** If your actual bucket name is different, replace `xyz-corporation-file-storage-shabdali-2026` with your actual bucket name.

Then select:

```text id="hm2ks6"
Save changes
```

### JSON Policy Explanation

| Policy Part                                  | Explanation                                                                                                                      |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `{` and `}`                                  | Curly braces define a JSON object.                                                                                               |
| `"Version"`                                  | Specifies the IAM policy language version.                                                                                       |
| `"2012-10-17"`                               | The current standard policy language version commonly used for AWS IAM policies. It is not the date when you created the policy. |
| `"Statement"`                                | Contains one or more permission statements.                                                                                      |
| `[` and `]`                                  | Square brackets define a JSON array.                                                                                             |
| `"Sid"`                                      | Statement identifier used to give the policy statement a descriptive name.                                                       |
| `"PublicReadGetObject"`                      | Descriptive identifier for this public-read statement.                                                                           |
| `"Effect": "Allow"`                          | Explicitly grants the specified permission.                                                                                      |
| `"Principal": "*"`                           | Allows any principal, including anonymous internet users, to use the granted action.                                             |
| `"Action": "s3:GetObject"`                   | Grants permission to retrieve S3 objects.                                                                                        |
| `"Resource"`                                 | Defines which AWS resources the permission applies to.                                                                           |
| `arn:aws:s3:::`                              | Amazon Resource Name prefix for an S3 bucket resource.                                                                           |
| `xyz-corporation-file-storage-shabdali-2026` | The target bucket name.                                                                                                          |
| `/*`                                         | Wildcard representing every object key inside the bucket.                                                                        |

### Why is this step required?

S3 static website hosting does not automatically grant public read access to objects.

The bucket policy explicitly allows:

```text id="4vr0nf"
Principal: Anyone
Action: s3:GetObject
Resource: Every object inside this bucket
```

Without this permission, anonymous website visitors cannot retrieve the HTML files.

### Dependency

This step depends on:

* Step 6: Bucket-level Block Public Access settings must permit the public bucket policy.
* The correct bucket ARN must be used.

### What happens if this step is skipped?

If the public-read bucket policy is not added:

* Anonymous visitors cannot retrieve the website objects.
* The website endpoint may return `403 Access Denied`.
* `index.html` and `error.html` remain inaccessible to public visitors.

---

## Step 8: Enable Static Website Hosting

### Navigation

```text id="y48mgg"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Static website hosting
→ Edit
```

### Configuration

Configure:

* **Static website hosting:** `Enable`
* **Hosting type:** `Host a static website`
* **Index document:** `index.html`
* **Error document:** `error.html`

Then select:

```text id="nmpj2x"
Save changes
```

### Commands

No terminal command is required because static website hosting is configured through the AWS Management Console.

### Why is this step required?

Uploading HTML files alone does not configure the bucket as an S3 website.

Static website hosting tells Amazon S3:

* Which object is the default homepage.
* Which object should be used as the custom error document.
* To expose a website endpoint for browser-based HTTP access.

The resulting behavior is:

```text id="vyfw11"
Request: Website root /
        |
        v
Return index.html


Request: Missing object
        |
        v
Return error.html
```

### Dependency

This step depends on:

* Step 3: `index.html` must exist.
* Step 4: `error.html` must exist.
* Step 5: Both files must be uploaded to the S3 bucket.

### What happens if this step is skipped?

If static website hosting is not enabled:

* No S3 website endpoint will be available for the bucket.
* `index.html` will not automatically act as the root homepage through a website endpoint.
* The custom error document configuration will not be active.

---

## Step 9: Locate the S3 Static Website Endpoint

### Navigation

```text id="8xfs0v"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Static website hosting
```

### Configuration

Locate:

* **Bucket website endpoint:** The AWS-generated HTTP website endpoint.

For a bucket in the Mumbai Region, the endpoint format is generally similar to:

```text id="c5ujwb"
http://bucket-name.s3-website.ap-south-1.amazonaws.com
```

Your AWS Console displays the exact endpoint for your bucket and Region.

### Commands

No terminal command is required.

### Why is this step required?

The website endpoint is the browser-accessible address used to test the static website.

It is different from:

* An S3 object URL.
* An S3 URI.
* An S3 REST API endpoint.

The static website endpoint provides website-specific behavior such as:

* Index document handling.
* Custom error document handling.

### Dependency

This step depends on:

* Step 8: Static website hosting must be enabled successfully.

### What happens if this step is skipped?

The website may be configured correctly, but you will not have the endpoint required to verify it in a web browser.

---

## Step 10: Test the Static Website Homepage

### Navigation

Copy the **Bucket website endpoint** displayed by AWS and open it in a web browser.

### Action

Open the root website endpoint.

### Expected Output

The browser should display:

```text id="skd2pe"
Welcome to XYZ Corporation

This static website is hosted using Amazon S3.

Module 6 - Assignment 3: S3 Website Hosting
```

The browser tab title should display:

```text id="at7k2p"
XYZ Corporation
```

### Why is this step required?

This confirms that:

* Static website hosting is enabled.
* The website endpoint is working.
* `index.html` exists.
* The bucket policy permits public `s3:GetObject` requests.
* Amazon S3 automatically serves `index.html` as the index document.

### Dependency

This step depends on:

* Step 5: `index.html` must be uploaded.
* Step 6: Required public-access blocking must be removed.
* Step 7: Public read permission must be granted.
* Step 8: Static website hosting must be enabled.
* Step 9: The website endpoint must be available.

### What happens if this step is skipped?

The website configuration may exist, but successful public access to the homepage will not be proven.

---

## Step 11: Test the Custom Error Page

### Navigation

Take the S3 website endpoint and append a nonexistent object path.

For example:

```text id="5xjzh5"
http://bucket-name.s3-website.ap-south-1.amazonaws.com/missing-page.html
```

Use your actual AWS-generated website endpoint.

### Action

Open the nonexistent path in a web browser.

### Expected Output

The custom error page should display content similar to:

```text id="qx83gm"
404 - Page Not Found

The page you requested could not be found.

Please return to the XYZ Corporation home page.
```

### Why is this step required?

This verifies that:

* `error.html` exists.
* The error document is correctly configured.
* Amazon S3 returns the custom error document for an appropriate failed website request.

### Dependency

This step depends on:

* `error.html` being uploaded.
* Static website hosting being enabled.
* `error.html` being configured as the error document.
* Public read access being available.

### What happens if this step is skipped?

The homepage may work, but the custom error-page requirement will not be verified.

---

## Step 12: Open the S3 Lifecycle Rule Configuration

### Navigation

```text id="s01gse"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Management
→ Lifecycle rules
→ Create lifecycle rule
```

### Configuration

Configure:

* **Lifecycle rule name:** `Standard-to-StandardIA-and-Expire`
* **Choose a rule scope:** `Apply to all objects in the bucket`

AWS may display a warning explaining that the rule applies to all objects. Acknowledge the warning if required by the console.

### Commands

No terminal command is required because the lifecycle rule is configured through the AWS Management Console.

### Why is this step required?

The assignment requires a lifecycle policy that manages objects automatically according to their age.

The intended lifecycle is:

```text id="4ap35v"
Object Created
      |
      v
S3 Standard
      |
      | 60 days after object creation
      v
S3 Standard-IA
      |
      | Object reaches age of 200 days
      v
Expiration
```

### Dependency

This step depends on:

* The existing S3 bucket being available.

It does not depend on static website hosting technically, but both features are configured on the same bucket as required by the assignment.

### What happens if this step is skipped?

Without creating a lifecycle rule:

* Objects remain in their current storage class unless manually changed.
* Automatic transition after 60 days does not occur.
* Automatic expiration at 200 days does not occur.
* Task 2 remains incomplete.

---

## Step 13: Configure the Lifecycle Rule Actions

### Navigation

Continue from the lifecycle rule creation page.

### Configuration

Under **Lifecycle rule actions**, select:

* `Transition current versions of objects between storage classes`
* `Expire current versions of objects`

Because this bucket is version-enabled, AWS may also display additional actions such as:

* Transition noncurrent versions of objects between storage classes.
* Permanently delete noncurrent versions of objects.
* Delete expired object delete markers or incomplete multipart uploads.

Do not select additional actions unless explicitly required.

For this assignment, configure only the requested current-version behavior.

### Why is this step required?

These two selected actions directly map to the assignment requirements:

| Assignment Requirement                           | Lifecycle Action                                               |
| ------------------------------------------------ | -------------------------------------------------------------- |
| Transition Standard to Standard-IA after 60 days | Transition current versions of objects between storage classes |
| Expiration after 200 days                        | Expire current versions of objects                             |

### Dependency

This step depends on:

* Step 12: The lifecycle rule creation process must be started.

### What happens if this step is skipped?

If lifecycle actions are not selected:

* No transition behavior will be configured.
* No expiration behavior will be configured.
* The lifecycle rule cannot satisfy the assignment requirements.

---

## Step 14: Configure Transition from S3 Standard to S3 Standard-IA After 60 Days

### Navigation

Continue within the lifecycle rule creation page under:

```text id="ywbxjj"
Transition current versions of objects between storage classes
```

### Configuration

Configure:

* **Choose storage class transitions:** `Standard-IA`
* **Days after object creation:** `60`

The resulting lifecycle behavior is:

```text id="m5nl1f"
Day 0
Object stored in S3 Standard
        |
        | 60 days
        v
Day 60
Object becomes eligible for transition to S3 Standard-IA
```

### Important Small-Object Consideration

The website files `index.html` and `error.html` are likely smaller than `128 KB`.

Amazon S3 Lifecycle behavior for small objects must be considered carefully. Depending on the lifecycle configuration behavior and applicable S3 defaults, objects below the minimum transition size may not transition unless the rule explicitly permits the required object-size range.

The AWS Console may display information or warnings about minimum object size for lifecycle transitions.

For the course assignment, the required configuration remains:

* **Storage class:** `Standard-IA`
* **Days after object creation:** `60`

Do not change the required `60 days` to one minute because minute-based lifecycle transitions are not supported.

### Why is this step required?

S3 Standard-IA is designed for data that is accessed less frequently but still requires rapid access when needed.

The lifecycle rule automates movement from:

```text id="b6ephg"
S3 Standard
      |
      | 60 days
      v
S3 Standard-IA
```

Without lifecycle automation, storage-class changes would need to be performed manually or programmatically.

### Dependency

This step depends on:

* Step 13: The current-version transition action must be selected.

### What happens if this step is skipped?

If the Standard-IA transition is not configured:

* Objects remain in S3 Standard.
* The required automatic 60-day transition does not exist.
* Task 2(a) remains incomplete.

---

## Step 15: Configure Current Object Expiration After 200 Days

### Navigation

Continue within the same lifecycle rule creation page under:

```text id="pfnf1m"
Expire current versions of objects
```

### Configuration

Configure:

* **Days after object creation:** `200`

The complete lifecycle timeline becomes:

```text id="w1l3sn"
Day 0
Object created in S3 Standard
        |
        | 60 days
        v
Day 60
Transition to S3 Standard-IA
        |
        | Object continues aging
        v
Day 200
Current object version reaches expiration age
```

The object spends approximately:

```text id="1ecpum"
200 - 60 = 140 days
```

between the transition age and expiration age.

> **Important:** The 200-day expiration is calculated from object creation time, not as 200 additional days after the Standard-IA transition.

### Important Behavior in This Version-Enabled Bucket

Because versioning is enabled, expiration of a current object version does not necessarily mean every historical version is permanently erased.

When lifecycle expiration acts on a current version in a version-enabled bucket, delete-marker and noncurrent-version behavior must be considered.

The assignment does not ask for:

* Permanent deletion of noncurrent versions.
* Transition of noncurrent versions.
* Removal of expired delete markers.

Therefore, these extra lifecycle actions are not added.

### Why is this step required?

The assignment explicitly requires:

> Expiration in 200 days.

This lifecycle action automates the expiration behavior when the current object reaches the configured age.

### Dependency

This step depends on:

* Step 13: The `Expire current versions of objects` action must be selected.

### What happens if this step is skipped?

If expiration is not configured:

* Objects do not automatically reach the requested lifecycle expiration action at 200 days.
* Manual deletion would be required.
* Task 2(b) remains incomplete.

---

## Step 16: Create the Lifecycle Rule

### Navigation

Continue to the bottom of the lifecycle rule creation page.

### Configuration

Before creating the rule, verify:

| Setting                   | Required Value                      |
| ------------------------- | ----------------------------------- |
| Lifecycle rule name       | `Standard-to-StandardIA-and-Expire` |
| Rule scope                | All objects in the bucket           |
| Transition action         | Current versions                    |
| Destination storage class | `Standard-IA`                       |
| Transition age            | `60 days`                           |
| Expiration action         | Current versions                    |
| Expiration age            | `200 days`                          |

Then select:

```text id="3u2vzr"
Create rule
```

### Commands

No terminal command is required because the lifecycle rule is created through the AWS Management Console.

### Why is this step required?

Configuring fields on the lifecycle creation page does not activate the rule until it is created and saved.

Selecting `Create rule` stores the lifecycle configuration on the bucket.

### Dependency

This step depends on:

* Step 12: Rule name and scope configured.
* Step 13: Required lifecycle actions selected.
* Step 14: 60-day Standard-IA transition configured.
* Step 15: 200-day expiration configured.

### What happens if this step is skipped?

If the final create operation is not completed:

* The lifecycle rule is not saved.
* No lifecycle automation exists.
* Both lifecycle requirements remain incomplete.

---

# 5. Verification / Output Checking

After completing the implementation, verify both major assignment requirements:

1. The existing S3 bucket successfully hosts a static website using:

   * `index.html`
   * `error.html`
2. The bucket has an enabled lifecycle rule that:

   * Transitions current object versions to `S3 Standard-IA` after `60 days`.
   * Expires current object versions when they reach an age of `200 days`.

Because S3 Lifecycle actions are asynchronous and use day-based timing, verification should focus on confirming that the lifecycle configuration has been correctly saved and enabled.

---

## Verification Step 1: Verify That `index.html` and `error.html` Exist in the Bucket

### Action

Open the bucket's `Objects` tab and inspect the object list.

### Navigation

```text id="z83d2f"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
```

### Expected Output

The object list should contain:

```text id="rl4g9x"
index.html
error.html
```

If the five objects from Assignment 1 were retained, the bucket may contain:

```text id="v19bml"
config.json
data.csv
document.txt
error.html
image.jpg
index.html
page.html
```

The exact order may differ depending on the AWS Console sorting configuration.

### Why does this confirm success?

Static website hosting requires the configured index and error documents to exist as objects inside the S3 bucket.

Seeing both objects confirms that:

* `index.html` was uploaded successfully.
* `error.html` was uploaded successfully.
* Both website files are available to the S3 static website hosting configuration.

---

## Verification Step 2: Verify That Static Website Hosting Is Enabled

### Action

Inspect the bucket's static website hosting configuration.

### Navigation

```text id="9rffb5"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Static website hosting
```

### Expected Output

The AWS Console should show configuration similar to:

```text id="0aw1ko"
Static website hosting: Enabled
Hosting type: Host a static website
Index document: index.html
Error document: error.html
```

A bucket website endpoint should also be displayed.

### Why does this confirm success?

This confirms that:

* The bucket is configured as a static website.
* Amazon S3 knows which object to return for root website requests.
* Amazon S3 knows which custom document to return for applicable website errors.
* A website endpoint has been generated.

---

## Verification Step 3: Verify the Static Website Homepage

### Action

Copy the **Bucket website endpoint** displayed under the static website hosting configuration and open it in a web browser.

### Expected Output

The browser should display:

```text id="ps9ap8"
Welcome to XYZ Corporation

This static website is hosted using Amazon S3.

Module 6 - Assignment 3: S3 Website Hosting
```

The browser tab title should display:

```text id="jtzs6n"
XYZ Corporation
```

### Why does this confirm success?

This confirms that all required components work together:

```text id="3k0n0g"
Browser
    |
    v
S3 Website Endpoint
    |
    v
Public GET Request
    |
    v
Bucket Policy Allows s3:GetObject
    |
    v
S3 Finds index.html
    |
    v
Homepage Displayed Successfully
```

A successful homepage proves that:

* The website endpoint is active.
* `index.html` exists.
* The index document is configured correctly.
* Public read access is functioning.
* Amazon S3 can return the HTML object to an unauthenticated browser.

---

## Verification Step 4: Verify the Custom Error Page

### Action

Append a nonexistent filename to the S3 website endpoint.

Example format:

```text id="mlr2kd"
http://bucket-name.s3-website.ap-south-1.amazonaws.com/this-page-does-not-exist.html
```

Use your actual S3 website endpoint.

### Expected Output

The browser should display:

```text id="fm5rb6"
404 - Page Not Found

The page you requested could not be found.

Please return to the XYZ Corporation home page.
```

### Why does this confirm success?

This confirms that:

* The requested object does not exist.
* S3 static website hosting processes the failed request.
* `error.html` is correctly configured as the custom error document.
* The custom error page is publicly retrievable.

This verifies the second website file required by Task 1.

---

## Verification Step 5: Verify the Public Bucket Policy

### Action

Inspect the bucket policy.

### Navigation

```text id="b0o53u"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Permissions
→ Bucket policy
```

### Expected Output

The policy should contain an `Allow` statement granting:

```text id="m5ox7j"
Principal: *
Action: s3:GetObject
Resource: arn:aws:s3:::xyz-corporation-file-storage-shabdali-2026/*
```

The complete policy should be similar to:

```json id="pzk6i3"
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::xyz-corporation-file-storage-shabdali-2026/*"
        }
    ]
}
```

### Why does this confirm success?

This policy permits anonymous internet users to retrieve objects from the bucket.

The permission is:

```text id="v17u2f"
Who?
→ Anyone: Principal "*"

Can do what?
→ Retrieve objects: s3:GetObject

On which resources?
→ Every object in this specific bucket: /*
```

This permission enables the public S3 website endpoint to serve the HTML objects.

---

## Verification Step 6: Verify That the Lifecycle Rule Exists and Is Enabled

### Action

Open the bucket's lifecycle rule list.

### Navigation

```text id="bq2l92"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Management
→ Lifecycle rules
```

### Expected Output

A lifecycle rule should be visible with:

* **Rule Name:** `Standard-to-StandardIA-and-Expire`
* **Status:** `Enabled`

Conceptual output:

```text id="oz2q5x"
Lifecycle rule name                       Status
------------------------------------------------
Standard-to-StandardIA-and-Expire         Enabled
```

### Why does this confirm success?

The rule's presence confirms that:

* The lifecycle configuration was successfully saved.
* The rule is associated with the correct S3 bucket.
* The rule is enabled and available for lifecycle processing.

However, the existence of the rule alone is not enough. The individual transition and expiration actions should also be verified.

---

## Verification Step 7: Verify the 60-Day Standard-IA Transition

### Action

Open the lifecycle rule details.

### Navigation

```text id="qx1tsg"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Management
→ Lifecycle rules
→ Standard-to-StandardIA-and-Expire
```

### Expected Output

The lifecycle rule should show:

```text id="9k52n8"
Action: Transition current versions of objects
Storage class: Standard-IA
Days after object creation: 60
```

The expected lifecycle flow is:

```text id="9wrh10"
Day 0
Object created in S3 Standard
        |
        | 60 days
        v
Day 60
Object becomes eligible for transition to S3 Standard-IA
```

### Why does this confirm success?

This directly verifies Task 2(a):

> Transition from Standard to Standard-IA in 60 days.

The rule configuration confirms that current object versions matching the rule's scope are configured for the requested storage-class transition, subject to S3 Lifecycle eligibility requirements.

---

## Verification Step 8: Verify the 200-Day Expiration

### Action

Continue inspecting the same lifecycle rule.

### Expected Output

The lifecycle rule should show:

```text id="f1c8w5"
Action: Expire current versions of objects
Days after object creation: 200
```

The complete lifecycle timeline should be:

```text id="0q1zmh"
Day 0
Object created in S3 Standard
        |
        | 60 days
        v
Day 60
Transition to S3 Standard-IA
        |
        | Object continues aging
        v
Day 200
Current object version reaches configured expiration age
```

### Why does this confirm success?

This directly verifies Task 2(b):

> Expiration in 200 days.

The configuration confirms that the lifecycle rule includes an expiration action at the required object age.

Because the bucket is version-enabled, remember that expiration of a current version can result in versioning-specific behavior, such as the creation of a delete marker, while older noncurrent versions may remain unless separately managed.

---

## Verification Step 9: Verify the Complete Lifecycle Rule Configuration

### Action

Review the complete lifecycle rule configuration one final time.

### Expected Output

The configuration should match:

| Configuration     | Expected Value                      |
| ----------------- | ----------------------------------- |
| Rule Name         | `Standard-to-StandardIA-and-Expire` |
| Status            | `Enabled`                           |
| Rule Scope        | All objects in the bucket           |
| Transition Target | `Standard-IA`                       |
| Transition Timing | `60 days after object creation`     |
| Expiration Timing | `200 days after object creation`    |

### Why does this confirm success?

This verifies that both required lifecycle actions are configured together in the same bucket:

```text id="qyrf9e"
Current Object Version
        |
        v
Initially stored in S3 Standard
        |
        | Age reaches 60 days
        v
Transition to S3 Standard-IA
        |
        | Age reaches 200 days
        v
Expiration action
```

This completes verification of Task 2.

---

# 6. Cleanup / Cost Optimization

This assignment uses a version-enabled S3 bucket that has also been configured for public website access.

Therefore, cleanup must account for:

* Public bucket policy.
* Block Public Access settings.
* Static website hosting.
* Lifecycle rule.
* Current object versions.
* Noncurrent object versions.
* Possible delete markers.
* The S3 bucket itself.

The recommended cleanup order is:

```text id="avldjm"
Stop Public Website Exposure
        |
        v
Delete Public Bucket Policy
        |
        v
Re-enable Block Public Access
        |
        v
Disable Static Website Hosting
        |
        v
Delete Lifecycle Rule
        |
        v
Show All Object Versions
        |
        v
Permanently Delete Every Object Version
        |
        v
Permanently Delete Every Delete Marker, If Any
        |
        v
Verify Bucket Is Completely Empty
        |
        v
Delete Empty S3 Bucket
        |
        v
Verify Bucket Deletion
```

> **Important:** If another subsequent assignment explicitly depends on this same S3 bucket, do not delete the bucket or its required objects until the dependent assignment is complete.

---

## Cleanup Step 1: Delete the Public Bucket Policy

### Navigation

```text id="7x2v13"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Permissions
→ Bucket policy
→ Edit
```

### Action

Delete the public-read bucket policy.

Remove the complete policy:

```json id="pc1xzd"
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::xyz-corporation-file-storage-shabdali-2026/*"
        }
    ]
}
```

Save the changes.

Depending on the current AWS Console workflow, you may instead see a direct option to delete the bucket policy.

### Why is this required?

The bucket policy grants public read access to every object in the bucket.

Removing it stops anonymous users from retrieving bucket objects through that permission.

This follows the security principle:

```text id="gz3w1h"
Public access required for testing
        |
        v
Complete website verification
        |
        v
Public access no longer required
        |
        v
Remove public permission immediately
```

### What happens if this resource is not deleted?

If the public-read bucket policy remains:

* Objects may remain publicly accessible.
* Future objects uploaded to matching resource paths may also become publicly readable.
* The bucket retains unnecessary public exposure.

---

## Cleanup Step 2: Re-enable Block Public Access

### Navigation

```text id="0p1p94"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Permissions
→ Block public access (bucket settings)
→ Edit
```

### Action

Enable:

* **Block all public access:** `Enabled`

This should enable all four bucket-level Block Public Access options.

Select:

```text id="mk5jbb"
Save changes
```

Complete any confirmation required by the AWS Console.

### Why is this required?

Block Public Access provides an additional protection layer against accidental public exposure.

After public website testing is complete, restoring this setting follows good security practice.

### What happens if this resource is not deleted?

This step changes a security configuration rather than deleting a resource.

If Block Public Access remains disabled:

* Future public policies could expose bucket objects.
* The bucket remains less protected against accidental public access.

---

## Cleanup Step 3: Disable Static Website Hosting

### Navigation

```text id="5k2fxh"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Properties
→ Static website hosting
→ Edit
```

### Action

Configure:

* **Static website hosting:** `Disable`

Then select:

```text id="l47xfs"
Save changes
```

### Why is this required?

The website endpoint is no longer required after verification.

Disabling static website hosting removes the website-hosting configuration from the bucket.

### What happens if this resource is not deleted?

Static website hosting itself does not create a separate hosting charge, but leaving unused website configurations active can:

* Create unnecessary configuration clutter.
* Cause confusion about whether a bucket is intended to serve public website content.
* Increase the chance of accidental future exposure if public permissions are restored.

---

## Cleanup Step 4: Delete the Lifecycle Rule

### Navigation

```text id="bxv88w"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Management
→ Lifecycle rules
```

### Action

Select:

* **Lifecycle Rule:** `Standard-to-StandardIA-and-Expire`

Then select the delete option and confirm deletion.

### Why is this required?

The lifecycle rule was created specifically for this assignment.

Removing it ensures that no future objects in the bucket are automatically affected by:

* Standard-IA transitions.
* 200-day expiration actions.

### What happens if this resource is not deleted?

If the lifecycle rule remains and the bucket is reused:

* Future objects matching the rule scope may transition automatically.
* Current versions may reach the configured expiration action.
* Unexpected storage-class changes or expiration behavior could occur.

---

## Cleanup Step 5: Display All Object Versions and Delete Markers

### Navigation

```text id="nns3te"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Action

Enable `Show versions`.

Review every object version and any delete markers.

Depending on the previous assignments, you may see versions associated with:

```text id="8j96ru"
config.json
data.csv
document.txt
error.html
image.jpg
index.html
page.html
```

`document.txt` and `data.csv` should have multiple versions if Assignment 2 was completed as documented.

### Why is this required?

The bucket is version-enabled.

The normal object view can hide:

* Noncurrent versions.
* Historical versions.
* Delete markers.

A bucket that appears empty in normal view may still contain stored versions.

### What happens if this resource is not deleted?

This step does not itself delete a resource.

However, if hidden versions are not displayed:

* Stored data may remain.
* Storage usage may continue.
* The bucket may not be truly empty.
* Bucket deletion can fail.

---

## Cleanup Step 6: Permanently Delete Every Object Version and Delete Marker

### Navigation

```text id="j49jz4"
AWS Management Console
→ S3
→ General purpose buckets
→ xyz-corporation-file-storage-shabdali-2026
→ Objects
→ Show versions
```

### Action

Select:

* Every current object version.
* Every noncurrent object version.
* Every version with a `null` version ID, if displayed.
* Every delete marker, if present.

Then select:

```text id="28xpfp"
Delete
```

Follow the AWS Console confirmation process.

Depending on the current workflow, you may need to enter:

```text id="p0ld72"
permanently delete
```

Confirm permanent deletion.

### Why is this required?

Each retained object version can consume storage.

For example:

```text id="2ok3j9"
document.txt
├── Current version       → Storage usage
└── Previous version      → Storage usage

data.csv
├── Current version       → Storage usage
└── Previous version      → Storage usage
```

Deleting only the visible current object may create a delete marker rather than permanently deleting historical versions.

Therefore, all versions and delete markers must be permanently removed.

### What happens if this resource is not deleted?

If object versions remain:

* Storage usage may continue.
* Applicable charges may occur.
* The bucket may not be considered empty.
* Bucket deletion may fail.

---

## Cleanup Step 7: Verify That the Bucket Is Completely Empty

### Navigation

```text id="aq0jzd"
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

There should be no entries for:

```text id="onl5er"
config.json
data.csv
document.txt
error.html
image.jpg
index.html
page.html
```

There should also be no delete markers.

### Why is this required?

This confirms that the version-enabled bucket is genuinely empty.

The verification is important because:

```text id="p5vpx8"
Normal Objects View Empty
        |
        | Does not always mean
        v
All Versions Deleted


Show Versions Empty
        |
        v
No current versions
No noncurrent versions
No delete markers
        |
        v
Bucket genuinely empty
```

### What happens if this resource is not deleted?

This is a verification step rather than a separate deletion action.

If any version remains:

* Cleanup is incomplete.
* Storage usage may continue.
* Bucket deletion may fail.

---

## Cleanup Step 8: Delete the Empty S3 Bucket

### Navigation

```text id="7snkj4"
AWS Management Console
→ S3
→ General purpose buckets
```

### Action

Select:

* **Bucket:** `xyz-corporation-file-storage-shabdali-2026`

Then select:

```text id="5v9xqe"
Delete
```

When prompted, enter the exact bucket name:

```text id="iy1cfj"
xyz-corporation-file-storage-shabdali-2026
```

Then select:

```text id="ek17md"
Delete bucket
```

Wait for the operation to complete.

### Why is this required?

The bucket was created for the S3 assignment series.

If no subsequent assignment depends on it, deleting the bucket:

* Removes the unused AWS resource.
* Keeps the account organized.
* Prevents accidental future uploads.
* Completes the resource lifecycle.

### What happens if this resource is not deleted?

If the bucket remains:

* It remains as an unused resource in the AWS account.
* Its globally unique bucket name remains associated with the bucket.
* Future accidental uploads could generate storage and request usage.
* The AWS account can become cluttered with obsolete resources.

---

## Cleanup Step 9: Verify That the S3 Bucket Has Been Deleted

### Navigation

```text id="zn2nh4"
AWS Management Console
→ S3
→ General purpose buckets
```

### Action

Search for:

```text id="2mkr0w"
xyz-corporation-file-storage-shabdali-2026
```

### Expected Result

The bucket should no longer appear in the general purpose buckets list.

### Why is this required?

This confirms that:

* The public bucket policy was removed.
* Block Public Access protection was restored before deletion.
* Static website hosting was disabled.
* The lifecycle rule was removed.
* Every current and noncurrent object version was permanently deleted.
* Any delete markers were removed.
* The S3 bucket was successfully deleted.
* No AWS resources created specifically for this S3 assignment series remain active.

### What happens if this resource is not deleted?

This is a final verification step rather than a resource deletion operation.

If the bucket still appears:

* Confirm that all object versions have been permanently deleted.
* Confirm that no delete markers remain.
* Retry the bucket deletion.
* Verify that your IAM identity has permission to delete the bucket.

---
