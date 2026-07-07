# Module 10: Assignment 1 - Multi-Account Management via AWS Organizations and Service Control Policies

## Problem Statement
XYZ Corporation requires a structured multi-account governance framework to isolate departmental workloads while maintaining absolute operational compliance. Implement an AWS Organization configuration to programmatically manage a hierarchical multi-account directory. Establish distinct Organizational Units (OUs) and author a restrictive Service Control Policy (SCP) that acts as a centralized security guardrail, restricting all member accounts under governance to execute Amazon EC2 operational API calls exclusively.

---

## Tasks To Be Performed
1. Initialize and launch a master AWS Organization from the root billing account profile.
2. Structure the directory hierarchy by creating 3 distinct Organizational Units named `OU1`, `OU2`, and `OU3`.
3. Author and attach a custom Service Control Policy (SCP) that explicitly permits access *only* to Amazon EC2 services, systematically blocking all other AWS service dimensions.

---

## Part 1: Step-by-Step Implementation Solution

### Step 1: Initialize the AWS Organization Framework (Task 1 Requirement)
1. Log in to the **AWS Management Console** using your root/master credentials.
2. Search for **AWS Organizations** within the global services catalog bar to load the control dashboard.
3. Click on the orange **Create an organization** button.
4. The AWS automation engine will run a background setup script, registering your current root account as the Management Account and setting up the global organization structure.

---

### Step 2: Provision the Organizational Units (OU1, OU2, OU3) (Task 2 Requirement)
1. Inside the AWS Organizations structural tree interface, navigate to the **AWS accounts** sub-directory tab list.
2. Check the radio selection box container next to the top-level **Root** node element folder.
3. Click on the **Actions** dropdown menu located at the upper right header boundary cluster and select **Create new**.
4. Configure the first OU parameter:
   * **Organizational unit name:** `OU1`
   * Click **Create organizational unit**.
5. Re-select the **Root** parent node folder check box, click **Actions** > **Create new**, name the second folder target as **`OU2`**, and click create.
6. Repeat the exact process a final time: select **Root**, click **Actions** > **Create new**, name the terminal folder directory as **`OU3`**, and commit creation rules. Your visual tree structure will now update to track all 3 distinct nested OUs.

---

### Step 3: Enable, Author, and Attach the Restrictive EC2-Only SCP (Task 3 Requirement)
1. **Enable SCP Features:** In the left-hand vertical option panel tree, click on **Policies** > Select **Service control policies**. Click the **Enable service control policies** flag button.
2. **Author the Policy:** Click on the **Create policy** button.
3. Configure the policy definition variables:
   * **Policy name:** `XYZ-EC2-Only-Guardrail`
   * **Description:** 'Centralized security rule forcing member accounts to only utilize EC2 infrastructure assets.'
4. In the **Policy editor** section canvas, overwrite the default json block with the following precise whitelisting logic mapping rules:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "AllowEC2ActionsExclusively",
               "Effect": "Allow",
               "Action": [
                   "ec2:*"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

5. Click **Create policy** at the bottom right to register the template rule inside your master schema directory.
6. **Attach the Rule to OUs:**
* Click on your newly authored `XYZ-EC2-Only-Guardrail` hyperlink policy link.
* Navigate to the **Targets** tab sub-panel layout view and click **Attach**.
* Check the boxes adjacent to your newly managed targets **`OU1`**, **`OU2`**, and **`OU3`**.
* Click **Attach policy**. The structural security layout rule is now applied live across the specified Organizational Units.

---

## Part 2: Step-by-Step Deletion Process (Clean-up)

To restore your root administrative AWS profile to standard baseline defaults without leaving trailing governance structures, execute these steps in chronological order:

### 1. Detach and Revoke the Service Control Policy Guardrail

1. Inside the **AWS Organizations Console**, click on **Policies** > **Service control policies** from the left vertical menu block.
2. Select your custom policy entry: `XYZ-EC2-Only-Guardrail`.
3. Open the **Targets** tab window, select `OU1`, `OU2`, and `OU3` checkboxes, and click **Detach**. Confirm the action.
4. Navigate back to the main list row for `XYZ-EC2-Only-Guardrail`, click **Actions** > **Delete**, and finalize its removal from the memory tier.

### 2. Disassemble the Organizational Units

1. Navigate to the **AWS accounts** dashboard tracking view page.
2. Expand the **Root** node directory to display the children folders.
3. Select the check indicator box for **`OU1`**, click **Actions** -> Select **Delete**, and confirm to erase the unit.
4. Repeat the individual row elimination workflow for both **`OU2`** and **`OU3`** folders sequentially until only your core default root billing entry remains visible.

### 3. Dissolve and Delete the AWS Organization Container

1. On the left navigation pane tracking bar interface, click on the **Settings** cog wheel selection option.
2. Scroll to the bottom of the organization profile attributes view layout panel.
3. Locate and click on the orange **Delete organization** button.
4. A safety authorization verification prompt will launch. Click confirm to instantly wipe all remaining structural meta-traces cleanly from your AWS global dashboard tracking logs.

