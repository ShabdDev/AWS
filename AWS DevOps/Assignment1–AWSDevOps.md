# Module 12: Assignment 1 - Version Control Migration to Private AWS CodeCommit Repositories

## Problem Statement
XYZ Corporation requires a secure, private, and managed source control repository environment hosted natively within the AWS Cloud to store internal application codebases. Implement a secure Git migration pipeline by establishing a private Amazon CodeCommit repository and mirror/import an existing public GitHub repository asset codebase into the AWS ecosystem.

---

## Tasks To Be Performed
1. Create a secure, private source repository utilizing Amazon CodeCommit.
2. Configure local Git credentials and securely import/mirror code configurations from an external GitHub repository into the new CodeCommit repository endpoint.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Initialize the AWS CodeCommit Private Repository
1. Log in to the **AWS Management Console** and navigate to the **Developer Tools > CodeCommit Dashboard**.
2. Click on the orange **Create repository** button.
3. Configure repository parameters:
   * **Repository name:** `XYZ-Private-Core-Repo`
   * **Description:** 'Private enterprise repository for secure internal web application codebases.'
4. Click **Create**.
5. Once initialized, copy the **HTTPS URL** string displayed in the connection steps panel (e.g., `https://git-codecommit.us-east-1.amazonaws.com/v1/repos/XYZ-Private-Core-Repo`).

---

### Step 2: Establish IAM Git Authentication Credentials
1. Open a separate tab and navigate to the **IAM (Identity and Access Management) Console**.
2. Click on **Users** in the left sidebar pane and select your current active IAM User profile.
3. Open the **Security credentials** tab sub-panel and scroll down to the section titled **AWS CodeCommit credentials for IAM users**.
4. Click on the **Generate credentials** button.
5. **CRITICAL STEP:** Download the generated `.csv` file or copy the unique **Git Username** and **Git Password** strings displayed. *(These are distinctly required by your local terminal to gain write access to CodeCommit).* Click **Close**.

---

### Step 3: Mirror and Import GitHub Content to CodeCommit (Task 1 Validation)
1. Launch your local terminal application (e.g., Command Prompt, Git Bash, or Mac Terminal) on your workstation.
2. Execute a bare clone tracking sequence to pull down the source GitHub repository metadata from the internet locally:
   ```bash
   git clone --bare [https://github.com/your-github-username/your-sample-repo.git](https://github.com/your-github-username/your-sample-repo.git) temp-migration-repo

```

3. Shift your active terminal operational execution path directory directly into the newly downloaded package tracking container:
```bash
cd temp-migration-repo

```


4. Push and mirror the complete commit history, branches, and version tags into your private AWS managed cloud repository engine by executing:
```bash
git push --mirror [https://git-codecommit.us-east-1.amazonaws.com/v1/repos/XYZ-Private-Core-Repo](https://git-codecommit.us-east-1.amazonaws.com/v1/repos/XYZ-Private-Core-Repo)

```


5. **Authentication Prompt:** When the terminal pops up input masks requesting access data, paste the unique **Git Username** and **Git Password** values generated from the IAM Security credentials window in Step 2.
6. **Verification:** Once the push operation logs transfer completion updates, return to the **AWS CodeCommit Console** in your browser and refresh the page. You will see the entire folder tree matrix, source files, and historical commit logs from GitHub rendered inside your private AWS platform layout view perfectly.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

Execute these steps to cleanly remove the testing project registry tracking configurations from your enterprise console view once verification sign-offs are completed:

### 1. Delete the Private CodeCommit Repository Resource

1. Open the **AWS CodeCommit Console** dashboard interface window.
2. From the central repositories directory list, check the selection marker box next to **`XYZ-Private-Core-Repo`**.
3. Locate the upper functional action items header toolbar and click on **Delete repository**.
4. A safety assurance confirmation modal prompt window will pop up. Type the explicit confirmation text string token `delete` inside the verification frame matrix and click submit to instantly purge the asset.

### 2. Revoke IAM Git Access Tokens and Clean Local Cache

1. Open **IAM > Users > [Your User] > Security credentials**.
2. Scroll to the **AWS CodeCommit credentials for IAM users** workspace block grid.
3. Locate the generated credential row element tracking your active profile, click **Delete**, and confirm to instantly revoke terminal write authorizations.
4. Delete the temporary directory folder `temp-migration-repo` from your local machine desktop file management canvas to conclude server tracking cleanup steps.
