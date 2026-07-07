# Module 3: Assignment 1 - IAM Users and Groups Configuration

## Problem Statement
You work for XYZ Corporation. To maintain the security of the AWS account and the resources, you have been asked to implement a solution that can help easily recognize and monitor the different users by setting up specific IAM Users and Groups.

---

## Tasks To Be Performed
1. Create 4 IAM users named “Dev1”, “Dev2”, “Test1”, and “Test2”.
2. Create 2 groups named “Dev Team” and “Ops Team”.
3. Add Dev1 and Dev2 to the Dev Team.
4. Add Dev1, Test1 and Test2 to the Ops Team.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Create the IAM User Groups
1. Log in to the **AWS Management Console** as an Administrator.
2. Search for and navigate to the **IAM (Identity and Access Management)** dashboard.
3. In the left navigation pane, click on **User groups** and then click **Create group**.
4. **Create the "Dev Team" Group:**
   * **User group name:** `Dev Team`
   * *(Optional)* Attach a policy if required by your course (e.g., `AmazonEC2FullAccess`), otherwise leave policies blank for now.
   * Click **Create group**.
5. **Create the "Ops Team" Group:**
   * Click **Create group** again.
   * **User group name:** `Ops Team`
   * Click **Create group**.

### Step 2: Create the 4 IAM Users
1. In the left navigation pane of the IAM dashboard, click on **Users** and then click **Create user**.
2. **Configure Users:**
   * **User 1 Name:** `Dev1`
   * Click **Add another user** to create multiple users at once:
     * **User 2 Name:** `Dev2`
     * **User 3 Name:** `Test1`
     * **User 4 Name:** `Test2`
   * *Note: Do not check "Provide user access to the AWS Management Console" unless explicitly requested, as these are programmatic/test identities.*
3. Click **Next**.

### Step 3: Add Users to Respective Groups
1. On the **Set permissions** screen, select the **Add user to group** option.
2. Based on the organizational requirements, assign the users by checking the boxes next to the group names:
   * Select **Dev1** -> Check **Dev Team** and **Ops Team**.
   * Select **Dev2** -> Check **Dev Team**.
   * Select **Test1** -> Check **Ops Team**.
   * Select **Test2** -> Check **Ops Team**.
3. Click **Next** to review the user-to-group assignments:
   * **Dev Team Members:** `Dev1`, `Dev2`
   * **Ops Team Members:** `Dev1`, `Test1`, `Test2`
4. Click **Create users**.

### Step 4: Verify the Hierarchy
1. Click on **User groups** from the left menu.
2. Click on `Dev Team` -> Verify that **Dev1** and **Dev2** are listed under the *Users* tab.
3. Go back and click on `Ops Team` -> Verify that **Dev1**, **Test1**, and **Test2** are listed under the *Users* tab.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

Since IAM is free, these configurations won't cost anything. However, to keep your AWS architecture clean, you can remove them using these steps:

### 1. Delete the IAM Users
1. Go to the **IAM Dashboard** > **Users**.
2. Select the checkboxes next to `Dev1`, `Dev2`, `Test1`, and `Test2`.
3. Click the **Delete** button at the top.
4. Type `delete` in the text box to confirm and click **Delete**. (This automatically removes them from their respective groups first).

### 2. Delete the IAM User Groups
1. Navigate to **IAM Dashboard** > **User groups**.
2. Select the checkboxes next to `Dev Team` and `Ops Team`.
3. Click **Delete** at the top.
4. Confirm the permanent deletion of the groups.
