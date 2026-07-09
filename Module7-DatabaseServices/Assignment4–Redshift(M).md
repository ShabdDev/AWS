# Module 7 - Database Services

# Assignment 4 – Amazon Redshift

# Part 1: Cost Check, Architecture, Prerequisites, and Redshift Data Warehouse Creation

---

## Problem Statement

You work for XYZ Corporation. Their application requires a database service that can store data which can be retrieved if required. Implement a suitable service for the same.

While migrating, you are asked to perform the following tasks:

1. Create a Redshift data warehouse.
2. Using the query editor:
   - Load some data.
   - Query the data.

---

## Assignment Objective

The objective of this assignment is to create an **Amazon Redshift data warehouse**, connect to it through **Amazon Redshift Query Editor v2**, load sample data into a table, and execute SQL queries to retrieve and analyze that data.

For this assignment, we will use **Amazon Redshift Serverless** because it allows us to create and use a Redshift data warehouse without manually provisioning or managing a traditional Redshift cluster.

The implementation will follow this workflow:

~~~text
AWS Account
    |
    v
Select AWS Region
    |
    v
Open Amazon Redshift
    |
    v
Check Redshift Serverless Free Trial Availability
    |
    v
Create Redshift Serverless Namespace
    |
    v
Create Redshift Serverless Workgroup
    |
    v
Wait for Workgroup to Become Available
    |
    v
Open Amazon Redshift Query Editor v2
    |
    v
Connect to the Redshift Data Warehouse
    |
    v
Create a Table
    |
    v
Insert Sample Data
    |
    v
Query the Data
    |
    v
Verify Results
    |
    v
Delete Redshift Resources
~~~

---

## Free Tier and Cost Check

Before creating any AWS resource, it is important to understand whether the assignment is covered by the AWS Free Tier or promotional credits.

| AWS Service | Free Tier Eligible | Uses Credits | Notes |
| ----------- | ------------------ | ------------ | ----- |
| Amazon Redshift Serverless | Free trial may be available for eligible accounts | Yes, after free-trial credits are exhausted or if the account is not eligible | Compute usage is measured using Redshift Processing Units, called RPUs. |
| Redshift Managed Storage | May be covered by available Redshift Serverless free-trial credits | Yes | Stored data can generate charges depending on account eligibility and current pricing. |
| Redshift Query Editor v2 | No separate charge for simply opening and using the editor | Indirectly | Queries executed against Redshift consume Redshift compute resources. |
| AWS KMS AWS-owned key | No separate customer charge for the AWS-owned key used by the service | Normally no separate direct charge | We will avoid creating a customer-managed KMS key because it is unnecessary for this assignment. |
| Amazon S3 | Not required for this implementation | No additional S3 resource required | We will load sample data directly with SQL `INSERT` statements, avoiding the need to create an S3 bucket. |

### Is this assignment completely Free Tier eligible?

Not necessarily.

**Amazon Redshift Serverless should not be assumed to be permanently free.** AWS offers a Redshift Serverless free trial for eligible accounts, and the remaining free-trial balance can be viewed in the Redshift console when applicable.

If your AWS account:

- Has an active Redshift Serverless free trial, the assignment may be covered by the available trial credits.
- Has AWS promotional credits that apply to Redshift, eligible usage may consume those credits.
- Has neither applicable free-trial credits nor promotional credits, Redshift Serverless usage can generate charges.

### Important cost recommendation

Before creating the data warehouse, check whether the Redshift console displays a **free trial** or available free-trial credit balance.

For this assignment:

- Create only one Redshift Serverless namespace.
- Create only one Redshift Serverless workgroup.
- Use the resources only long enough to complete the assignment.
- Run only the small SQL queries required for verification.
- Delete the Redshift Serverless workgroup immediately after verification.
- Delete the namespace after deleting the workgroup and confirming that no snapshot needs to be retained.

> **Cost Warning:** Amazon Redshift is a chargeable AWS analytics service. Do not leave the Redshift Serverless workgroup running after completing the assignment.

---

## Why Amazon Redshift Is Suitable for This Assignment

Amazon Redshift is a fully managed cloud data warehouse designed for analytical workloads and SQL-based querying of structured data.

In this assignment:

- **Redshift Serverless namespace** stores database objects and data.
- **Redshift Serverless workgroup** provides the compute resources and network configuration required to execute SQL queries.
- **Query Editor v2** provides a browser-based SQL interface for connecting to the data warehouse and executing SQL statements.

The assignment requires:

~~~text
Store Data
    +
Retrieve Data Using Queries
    =
Amazon Redshift Data Warehouse
~~~

---

## Architecture

~~~text
+------------------------------------------------------+
|                     AWS Account                      |
|                                                      |
|   +----------------------------------------------+   |
|   |          Amazon Redshift Serverless          |   |
|   |                                              |   |
|   |   +--------------------------------------+   |   |
|   |   |              Namespace               |   |   |
|   |   |                                      |   |   |
|   |   |   Database: dev                      |   |   |
|   |   |   Table: employee_data               |   |   |
|   |   |   Sample employee records            |   |   |
|   |   +--------------------------------------+   |   |
|   |                     ^                        |   |
|   |                     |                        |   |
|   |   +--------------------------------------+   |   |
|   |   |              Workgroup               |   |   |
|   |   |                                      |   |   |
|   |   |   Provides compute capacity          |   |   |
|   |   |   Executes SQL queries               |   |   |
|   |   +--------------------------------------+   |   |
|   +----------------------------------------------+   |
|                          ^                           |
|                          |                           |
|   +----------------------------------------------+   |
|   |          Amazon Redshift Query Editor v2     |   |
|   |                                              |   |
|   |   CREATE TABLE                               |   |
|   |   INSERT INTO                                |   |
|   |   SELECT                                     |   |
|   +----------------------------------------------+   |
|                                                      |
+------------------------------------------------------+
~~~

---

## Resources Created in This Assignment

| Resource | Planned Name | Purpose |
| -------- | ------------ | ------- |
| Redshift Serverless Namespace | `xyz-redshift-namespace` | Stores databases, schemas, tables, users, and data. |
| Redshift Serverless Workgroup | `xyz-redshift-workgroup` | Provides compute resources for executing SQL queries. |
| Database | `dev` | Default database used for this assignment. |
| Table | `employee_data` | Stores sample employee records. |

---

## Prerequisites

Before starting the implementation, ensure that:

1. You have an active AWS account.
2. You can sign in to the AWS Management Console.
3. Your IAM identity has permission to create and delete Amazon Redshift Serverless resources.
4. Your IAM identity can access Amazon Redshift Query Editor v2.
5. You understand that Redshift Serverless can incur charges.
6. You will delete all resources immediately after completing verification.

No EC2 instance, RDS database, S3 bucket, custom VPC, or external SQL client is required for this implementation.

---

# Step 1: Sign In to the AWS Management Console and Select a Region

### Navigation

~~~text
AWS Management Console
→ Sign in
→ Open the Region selector in the upper-right corner
→ Select the region where you want to create the Redshift data warehouse
~~~

### Configuration

For this assignment, use:

| Setting | Value |
| ------- | ----- |
| AWS Region | `Asia Pacific (Mumbai) ap-south-1` |

If you have already been performing your course assignments in another AWS Region, you may continue using that region instead, provided Amazon Redshift Serverless is available there.

### Why is this step required?

AWS resources are created within specific geographical regions. Selecting the region first ensures that:

- The namespace and workgroup are created in the intended region.
- You can easily locate the resources later.
- Cleanup is performed in the correct region.
- You do not accidentally create duplicate resources across multiple regions.

### Dependency

This step has no AWS resource dependency. It only requires an active AWS account with appropriate permissions.

### What happens if this step is skipped?

You may accidentally create resources in the wrong region. Later, the Redshift resources may appear to be missing because the AWS Console is displaying a different region.

---

# Step 2: Open the Amazon Redshift Console

### Navigation

~~~text
AWS Management Console
→ Search bar
→ Search for "Redshift"
→ Select Amazon Redshift
~~~

### Action

Open the Amazon Redshift service console.

Depending on the current AWS Console interface and whether Redshift has previously been used in your account, you may see:

- A Redshift landing page.
- A provisioned cluster dashboard.
- A Redshift Serverless dashboard.
- A getting-started experience.
- A button or option to try the Redshift Serverless free trial.

The exact landing page can vary slightly based on account history, region, permissions, and AWS Console updates.

### Why is this step required?

The Amazon Redshift console is used to:

- Create the Redshift Serverless data warehouse.
- Manage namespaces.
- Manage workgroups.
- Monitor usage.
- Access Query Editor v2.
- Delete resources after verification.

### Dependency

Depends on:

- **Step 1:** Correct AWS Region selected.

### What happens if this step is skipped?

The Redshift data warehouse cannot be created or managed through the AWS Console.

---

# Step 3: Check Redshift Serverless Free Trial Availability

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Serverless dashboard or Getting started page
→ Check the Free trial section, if displayed
~~~

### Action

Before creating any Redshift resource, inspect the console for information such as:

- **Try Redshift Serverless Free Trial**
- **Free trial**
- Remaining free-trial credits
- Free-trial expiration information

The exact wording can vary depending on account eligibility and the current AWS Console interface.

### What should you do based on the result?

| Console Result | Recommended Action |
| -------------- | ------------------ |
| Free-trial credits are available | Proceed with the assignment and delete resources immediately after verification. |
| AWS promotional credits are available and applicable | Proceed carefully and monitor usage. |
| No Redshift free trial is displayed | Understand that usage may generate charges before continuing. |
| Unsure about billing | Check AWS Billing and Cost Management before creating resources. |

### Why is this step required?

Amazon Redshift Serverless is not a permanently free service. Checking eligibility before resource creation helps prevent unexpected charges and unnecessary consumption of AWS promotional credits.

### Dependency

Depends on:

- **Step 2:** Amazon Redshift console opened.

### What happens if this step is skipped?

You may create chargeable Redshift resources without first understanding whether your account has applicable free-trial credits.

---

# Step 4: Start Creating the Amazon Redshift Serverless Data Warehouse

### Navigation

The current console may present one of the following paths:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Serverless dashboard
→ Create workgroup
~~~

Or, for a first-time Redshift Serverless account:

~~~text
AWS Console
→ Amazon Redshift
→ Try Redshift Serverless Free Trial
→ Configuration
~~~

### Action

If the console offers a choice between:

- **Use default settings**
- **Customize settings**

choose:

`Customize settings`

We will explicitly configure resource names so that the architecture is easier to understand, verify, document, and clean up.

### Why is this step required?

A Redshift Serverless data warehouse consists primarily of:

1. A **namespace**, which stores data and database objects.
2. A **workgroup**, which provides compute capacity and network configuration for executing queries.

Creating the Redshift Serverless environment is therefore the main infrastructure step for this assignment.

### Dependency

Depends on:

- **Step 3:** Cost and free-trial status checked.

### What happens if this step is skipped?

There will be no Redshift data warehouse to store data or execute SQL queries against.

---

# Step 5: Configure the Redshift Serverless Namespace

### Navigation

Depending on the current console workflow:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Create workgroup or Customize settings
→ Namespace configuration
~~~

### Configuration

Use the following values:

| Setting | Value |
| ------- | ----- |
| Namespace | `xyz-redshift-namespace` |
| Database name | `dev` |
| Admin credentials | Use the console's supported default or recommended authentication option |
| Permissions | Do not add additional IAM roles for this assignment |
| Encryption | Use the default AWS-managed or AWS-owned encryption option presented by the console |

> **Important:** Do not create a customer-managed KMS key specifically for this assignment because it is unnecessary for the required tasks and can introduce additional resources and potential costs.

### What is a namespace?

A Redshift Serverless namespace is the logical storage layer of the data warehouse.

It contains resources such as:

~~~text
Namespace
    |
    +-- Databases
    |
    +-- Schemas
    |
    +-- Tables
    |
    +-- Views
    |
    +-- Users
    |
    +-- Stored Data
~~~

For this assignment:

~~~text
xyz-redshift-namespace
    |
    v
dev database
    |
    v
public schema
    |
    v
employee_data table
    |
    v
Sample employee records
~~~

### Why is this step required?

The namespace stores the actual database objects and data that will be created in later steps.

Without a namespace, there is no logical storage environment for the Redshift database.

### Dependency

Depends on:

- **Step 4:** Redshift Serverless creation process started.

### What happens if this step is skipped?

The Redshift Serverless data warehouse cannot store databases, schemas, tables, or sample records.

---

# Step 6: Configure the Redshift Serverless Workgroup

### Navigation

Continue in the same Redshift Serverless creation workflow:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Create workgroup or Customize settings
→ Workgroup configuration
~~~

### Configuration

Use the following values where the corresponding settings are displayed:

| Setting | Value |
| ------- | ----- |
| Workgroup name | `xyz-redshift-workgroup` |
| Namespace | `xyz-redshift-namespace` |
| Base capacity | Use the lowest capacity permitted by your console and region for this lab |
| Network and security | Keep default settings unless the console requires a specific selection |
| Publicly accessible | `Off` unless specifically required by your environment |
| Enhanced VPC routing | Keep disabled unless explicitly required |

> **Important:** The minimum permitted Redshift Serverless base capacity can vary according to AWS capabilities, region, account configuration, and feature availability. Use the lowest value that the AWS Console currently allows for this small educational assignment.

### What is a workgroup?

A workgroup is the Redshift Serverless compute and networking layer.

Its role can be understood as:

~~~text
SQL Query
    |
    v
Redshift Query Editor v2
    |
    v
Redshift Serverless Workgroup
    |
    v
Compute Resources Execute the Query
    |
    v
Namespace Data Is Read or Modified
~~~

The namespace and workgroup have different responsibilities:

| Component | Responsibility |
| --------- | -------------- |
| Namespace | Stores database objects, users, schemas, tables, and data. |
| Workgroup | Provides compute capacity and networking for query execution. |

### Why is this step required?

The namespace stores the data, but compute capacity is required to execute SQL operations such as:

- `CREATE TABLE`
- `INSERT`
- `SELECT`
- `UPDATE`
- `DELETE`

The workgroup provides this compute capability.

### Dependency

Depends on:

- **Step 5:** The namespace configuration must be available so that the workgroup can be associated with it.

### What happens if this step is skipped?

The data warehouse will not have the compute environment required to execute the SQL queries needed by the assignment.

---

# Step 7: Create the Redshift Serverless Data Warehouse

### Action

Review the configuration carefully.

Expected configuration:

| Resource | Expected Value |
| -------- | -------------- |
| Namespace | `xyz-redshift-namespace` |
| Workgroup | `xyz-redshift-workgroup` |
| Database | `dev` |
| Base capacity | Lowest capacity allowed by the current console for the lab |
| Public access | Disabled unless specifically necessary |
| Additional IAM role | Not required |
| Customer-managed KMS key | Not required |

Choose the console action that creates or saves the Redshift Serverless configuration.

Depending on the current interface, the button may be labeled according to the workflow being used, such as creating the workgroup or saving the configuration.

Do not close the console while AWS is creating the resources.

### Why is this step required?

This action provisions the actual Redshift Serverless environment required for the assignment.

After creation:

~~~text
xyz-redshift-namespace
        +
xyz-redshift-workgroup
        |
        v
Amazon Redshift Serverless Data Warehouse
        |
        v
Ready for SQL Queries
~~~

### Dependency

Depends on:

- **Step 5:** Namespace configured.
- **Step 6:** Workgroup configured.

### What happens if this step is skipped?

The configuration will not be provisioned, and there will be no Redshift data warehouse available for Query Editor v2.

---

# Step 8: Wait for the Workgroup to Become Available

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
→ Select xyz-redshift-workgroup
~~~

### Action

Wait until the workgroup is ready and available for use.

Verify that the following resources are visible:

| Resource Type | Expected Name |
| ------------- | ------------- |
| Namespace | `xyz-redshift-namespace` |
| Workgroup | `xyz-redshift-workgroup` |

Do not proceed to Query Editor v2 until the Redshift Serverless environment is ready.

### Expected State

The workgroup should indicate that it is available or ready for query execution.

The exact status wording can vary slightly with AWS Console updates.

### What should I observe?

You should be able to see:

- The workgroup named `xyz-redshift-workgroup`.
- The associated namespace named `xyz-redshift-namespace`.
- The workgroup endpoint or connection-related information.
- An option such as **Query data** for opening Query Editor v2.

### Why is this step required?

AWS needs time to provision and configure the serverless environment. Attempting to connect before provisioning finishes may result in connection errors or unavailable resources.

### Dependency

Depends on:

- **Step 7:** Redshift Serverless resources created.

### What happens if this step is skipped?

Query Editor v2 may fail to connect because the workgroup is not yet ready.

---

# Verification Step 1: Verify the Redshift Serverless Data Warehouse

### Action

Confirm that the Redshift Serverless workgroup and namespace were successfully created.

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
→ Select xyz-redshift-workgroup
~~~

### Expected Result

You should observe:

| Item | Expected Value |
| ---- | -------------- |
| Workgroup | `xyz-redshift-workgroup` |
| Namespace | `xyz-redshift-namespace` |
| Database | `dev` |
| Workgroup readiness | Available or ready for use |

### What should I observe?

The workgroup should be available and associated with the correct namespace.

There should be no creation failure or configuration error.

### Why does this confirm success?

A successfully available workgroup proves that the Redshift Serverless compute environment has been provisioned and can be used for executing SQL queries.

The associated namespace confirms that the data warehouse also has a logical storage environment for databases, schemas, tables, and data.

---

## Current Assignment Progress

~~~text
[Completed] Select AWS Region
        |
        v
[Completed] Open Amazon Redshift
        |
        v
[Completed] Check Cost and Free Trial
        |
        v
[Completed] Create Redshift Serverless Namespace
        |
        v
[Completed] Create Redshift Serverless Workgroup
        |
        v
[Completed] Verify Data Warehouse
        |
        v
[Next] Open Query Editor v2
        |
        v
[Next] Connect to the Data Warehouse
        |
        v
[Next] Create employee_data Table
        |
        v
[Next] Insert Sample Data
        |
        v
[Next] Query and Verify the Data
        |
        v
[Next] Cleanup All Redshift Resources
~~~

----

# Part 2: Query Editor v2, Data Loading, SQL Queries, and Verification

---

# Step 9: Open Amazon Redshift Query Editor v2

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Query data
→ Query in query editor v2
~~~

Alternatively, depending on the current AWS Console interface:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
→ Select xyz-redshift-workgroup
→ Query data
~~~

The console should open **Amazon Redshift Query Editor v2**.

### What is Amazon Redshift Query Editor v2?

Amazon Redshift Query Editor v2 is a browser-based SQL client provided by AWS.

It allows you to:

- Connect to an Amazon Redshift data warehouse.
- Create databases, schemas, and tables.
- Insert data.
- Execute SQL queries.
- View query results.
- Save SQL queries.
- Browse database objects.

For this assignment, Query Editor v2 will be used to:

~~~text
Connect to Redshift Serverless
        |
        v
Create employee_data Table
        |
        v
Insert Sample Employee Records
        |
        v
Execute SELECT Queries
        |
        v
Verify Retrieved Data
~~~

### Why is this step required?

The assignment explicitly requires using the query editor to:

1. Load some data.
2. Query the data.

Query Editor v2 provides the SQL interface required to perform both tasks directly from the AWS Management Console.

### Dependency

Depends on:

- **Step 8:** The Redshift Serverless workgroup must be created and available.

Required resources:

- `xyz-redshift-workgroup`
- `xyz-redshift-namespace`
- `dev` database

### What happens if this step is skipped?

You will not have the SQL interface needed to create the table, load sample data, or execute queries for this assignment.

---

# Step 10: Connect Query Editor v2 to the Redshift Serverless Workgroup

### Navigation

Inside Query Editor v2:

~~~text
Amazon Redshift Query Editor v2
→ Database pane
→ Select or create a connection
→ Choose Serverless
→ Select xyz-redshift-workgroup
→ Select database dev
→ Connect
~~~

The exact connection dialog may vary slightly depending on:

- AWS Console updates.
- Authentication method configured for the namespace.
- IAM permissions.
- Whether a previous connection already exists.

### Configuration

Use the following values where applicable:

| Setting | Value |
| ------- | ----- |
| Connection type | `Serverless` |
| Workgroup | `xyz-redshift-workgroup` |
| Database | `dev` |
| Authentication | Use the authentication method supported by the namespace configuration created in Part 1 |

If Query Editor v2 offers an option to connect using your IAM identity or temporary database credentials and your IAM permissions support it, use that option.

Do not create additional credentials or IAM roles unless your account configuration specifically requires them.

### Expected Result

After a successful connection, the left-side database explorer should display database objects associated with the `dev` database.

You may see a hierarchy similar to:

~~~text
xyz-redshift-workgroup
    |
    v
dev
    |
    v
Schemas
    |
    v
public
~~~

### Why is this step required?

Opening Query Editor v2 alone does not automatically guarantee that an active database connection exists.

The SQL editor must be connected to:

- The correct Redshift Serverless workgroup.
- The correct database.

This ensures that all SQL commands are executed against the intended data warehouse.

### Dependency

Depends on:

- **Step 9:** Query Editor v2 opened.
- `xyz-redshift-workgroup` must be available.
- `dev` database must exist.

### What happens if this step is skipped?

SQL commands cannot be executed because Query Editor v2 will not know which Redshift data warehouse and database should process them.

---

# Step 11: Create the Sample Employee Table

### Navigation

~~~text
Amazon Redshift Query Editor v2
→ Connect to xyz-redshift-workgroup
→ Select dev database
→ Open a new SQL editor tab
~~~

### Action

Enter the following SQL statement into the SQL editor:

~~~sql
CREATE TABLE employee_data (
    employee_id INTEGER,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    job_title VARCHAR(100),
    salary DECIMAL(10,2),
    joining_date DATE
);
~~~

Run the SQL statement using the **Run** button.

### Command Explanation

The complete SQL statement is:

~~~sql
CREATE TABLE employee_data (
    employee_id INTEGER,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    job_title VARCHAR(100),
    salary DECIMAL(10,2),
    joining_date DATE
);
~~~

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `CREATE` | SQL keyword used to create a new database object. |
| `TABLE` | Specifies that the database object being created is a table. |
| `employee_data` | The name assigned to the new table. |
| `(` | Begins the list of column definitions. |
| `employee_id` | Column used to store the unique numerical identifier of an employee. |
| `INTEGER` | Data type used to store whole numbers. |
| `,` | Separates one column definition from the next column definition. |
| `employee_name` | Column used to store the employee's name. |
| `VARCHAR(100)` | Variable-length character data type that allows a maximum of 100 characters. |
| `department` | Column used to store the department name. |
| `VARCHAR(50)` | Variable-length character data type that allows a maximum of 50 characters. |
| `job_title` | Column used to store the employee's job title. |
| `VARCHAR(100)` | Allows the job title to contain up to 100 characters. |
| `salary` | Column used to store the employee's salary. |
| `DECIMAL(10,2)` | Exact numeric data type allowing up to 10 total digits, with 2 digits after the decimal point. |
| `joining_date` | Column used to store the employee's joining date. |
| `DATE` | Data type used to store calendar dates. |
| `)` | Ends the list of column definitions. |
| `;` | Terminates the SQL statement. |

### Table Structure

After successful creation, the table has the following logical structure:

| Column | Data Type | Example Value | Purpose |
| ------ | --------- | ------------- | ------- |
| `employee_id` | `INTEGER` | `101` | Employee identifier |
| `employee_name` | `VARCHAR(100)` | `Aarav Sharma` | Employee name |
| `department` | `VARCHAR(50)` | `Engineering` | Department name |
| `job_title` | `VARCHAR(100)` | `DevOps Engineer` | Employee's role |
| `salary` | `DECIMAL(10,2)` | `85000.00` | Employee salary |
| `joining_date` | `DATE` | `2023-01-15` | Date employee joined |

### Expected Result

Query Editor v2 should report that the SQL statement completed successfully.

The exact success message can vary according to the current Query Editor v2 interface.

### Why is this step required?

Data must be stored inside a database table before it can be retrieved using SQL queries.

The `employee_data` table provides a structured location for storing the sample records required by the assignment.

### Dependency

Depends on:

- **Step 10:** Successful connection to the `dev` database.

### What happens if this step is skipped?

The subsequent `INSERT` statement will fail because the target table `employee_data` will not exist.

---

# Step 12: Verify That the Table Was Created

### Action

Refresh the database object explorer in Query Editor v2 if necessary.

### Navigation

~~~text
Amazon Redshift Query Editor v2
→ Database explorer
→ dev
→ Schemas
→ public
→ Tables
~~~

### Expected Result

The following table should be visible:

~~~text
employee_data
~~~

The table should contain these columns:

~~~text
employee_id
employee_name
department
job_title
salary
joining_date
~~~

### What should I observe?

You should see the `employee_data` table under the appropriate schema, typically the `public` schema.

If the table does not immediately appear:

1. Refresh the database explorer.
2. Confirm that the `CREATE TABLE` query completed successfully.
3. Confirm that Query Editor v2 is connected to the correct `dev` database.

### Why does this confirm success?

Seeing `employee_data` in the database explorer proves that the table object was successfully created in the Redshift database.

---

# Step 13: Load Sample Data into the Redshift Table

### Action

Execute the following SQL statement in Query Editor v2:

~~~sql
INSERT INTO employee_data
    (employee_id, employee_name, department, job_title, salary, joining_date)
VALUES
    (101, 'Aarav Sharma', 'Engineering', 'DevOps Engineer', 85000.00, '2023-01-15'),
    (102, 'Meera Patil', 'Engineering', 'Cloud Engineer', 92000.00, '2022-07-10'),
    (103, 'Rohan Verma', 'Finance', 'Financial Analyst', 70000.00, '2024-02-20'),
    (104, 'Ananya Singh', 'Human Resources', 'HR Manager', 78000.00, '2021-11-05'),
    (105, 'Vikram Joshi', 'Engineering', 'Site Reliability Engineer', 105000.00, '2020-06-18'),
    (106, 'Priya Nair', 'Marketing', 'Marketing Specialist', 68000.00, '2023-09-12'),
    (107, 'Kabir Mehta', 'Finance', 'Senior Accountant', 82000.00, '2022-03-25'),
    (108, 'Ishita Rao', 'Engineering', 'Software Engineer', 88000.00, '2024-01-08');
~~~

Run the SQL statement.

### Command Explanation

This SQL command inserts eight sample employee records into the `employee_data` table.

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `INSERT` | SQL keyword used to add new data to a table. |
| `INTO` | Specifies the destination table for the new records. |
| `employee_data` | Name of the target table receiving the records. |
| `(employee_id, employee_name, department, job_title, salary, joining_date)` | Explicitly specifies the table columns that will receive values. |
| `VALUES` | Introduces the actual records that will be inserted. |
| `101` through `108` | Integer employee identifiers used for the sample records. |
| Single quotes `'...'` | Used to delimit string and date literals in SQL. |
| `'Aarav Sharma'`, `'Meera Patil'`, and other names | Sample string values stored in the `employee_name` column. |
| `'Engineering'`, `'Finance'`, `'Human Resources'`, `'Marketing'` | Sample department values. |
| `'DevOps Engineer'` and other job titles | Sample values stored in the `job_title` column. |
| `85000.00` and other numeric values | Decimal salary values inserted into the `salary` column. |
| `'2023-01-15'` and other dates | Date values written in `YYYY-MM-DD` format. |
| `,` between values | Separates individual values within each row and also separates multiple row definitions. |
| `(` and `)` | Group the values belonging to each individual employee record. |
| `;` | Terminates the SQL statement. |

### Data Being Loaded

| Employee ID | Employee Name | Department | Job Title | Salary | Joining Date |
| ----------- | ------------- | ---------- | --------- | ------ | ------------ |
| `101` | Aarav Sharma | Engineering | DevOps Engineer | `85000.00` | `2023-01-15` |
| `102` | Meera Patil | Engineering | Cloud Engineer | `92000.00` | `2022-07-10` |
| `103` | Rohan Verma | Finance | Financial Analyst | `70000.00` | `2024-02-20` |
| `104` | Ananya Singh | Human Resources | HR Manager | `78000.00` | `2021-11-05` |
| `105` | Vikram Joshi | Engineering | Site Reliability Engineer | `105000.00` | `2020-06-18` |
| `106` | Priya Nair | Marketing | Marketing Specialist | `68000.00` | `2023-09-12` |
| `107` | Kabir Mehta | Finance | Senior Accountant | `82000.00` | `2022-03-25` |
| `108` | Ishita Rao | Engineering | Software Engineer | `88000.00` | `2024-01-08` |

### Expected Result

The SQL statement should complete successfully and insert eight rows.

### Why is this step required?

The assignment specifically requires loading data using the query editor.

The `INSERT INTO` command satisfies this requirement by adding sample records directly to the Redshift table.

### Dependency

Depends on:

- **Step 11:** `employee_data` table created.
- **Step 12:** Table creation verified.

### What happens if this step is skipped?

The table will remain empty, and there will be no meaningful data to retrieve during the query portion of the assignment.

---

# Verification Step 2: Verify the Number of Loaded Records

### Action

Execute:

~~~sql
SELECT COUNT(*) AS total_employees
FROM employee_data;
~~~

### Command Explanation

This query counts every row currently stored in the `employee_data` table.

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `SELECT` | SQL keyword used to retrieve data or calculate a result. |
| `COUNT(*)` | Aggregate function that counts all rows in the result set. |
| `COUNT` | SQL aggregate function used to count records. |
| `(` and `)` | Enclose the argument passed to the `COUNT` function. |
| `*` | Represents all rows for the purpose of the count operation. |
| `AS` | Creates a readable alias for the output column. |
| `total_employees` | Alias displayed as the result column name. |
| `FROM` | Specifies the source table. |
| `employee_data` | Table whose rows are counted. |
| `;` | Terminates the SQL statement. |

### Expected Output

~~~text
total_employees
---------------
8
~~~

### What should I observe?

The value of `total_employees` should be:

`8`

### Why does this confirm success?

Eight rows were inserted in Step 13. A result of `8` confirms that all expected sample records are stored in the Redshift table.

---

# Step 14: Query All Data from the Redshift Table

### Action

Execute:

~~~sql
SELECT *
FROM employee_data
ORDER BY employee_id;
~~~

### Command Explanation

This query retrieves every column and every row from `employee_data` and sorts the result by employee ID.

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `SELECT` | Retrieves data from a database table. |
| `*` | Selects all columns from the specified table. |
| `FROM` | Specifies the source table. |
| `employee_data` | The table containing the sample employee records. |
| `ORDER BY` | Sorts the returned rows according to a specified column. |
| `employee_id` | Column used to sort the records. |
| `;` | Terminates the SQL statement. |

### Expected Output

~~~text
employee_id | employee_name | department      | job_title                 | salary    | joining_date
------------+---------------+-----------------+---------------------------+-----------+-------------
101         | Aarav Sharma  | Engineering     | DevOps Engineer           | 85000.00  | 2023-01-15
102         | Meera Patil   | Engineering     | Cloud Engineer            | 92000.00  | 2022-07-10
103         | Rohan Verma   | Finance         | Financial Analyst         | 70000.00  | 2024-02-20
104         | Ananya Singh  | Human Resources | HR Manager                | 78000.00  | 2021-11-05
105         | Vikram Joshi  | Engineering     | Site Reliability Engineer | 105000.00 | 2020-06-18
106         | Priya Nair    | Marketing       | Marketing Specialist      | 68000.00  | 2023-09-12
107         | Kabir Mehta   | Finance         | Senior Accountant         | 82000.00  | 2022-03-25
108         | Ishita Rao    | Engineering     | Software Engineer         | 88000.00  | 2024-01-08
~~~

The exact visual formatting of the result grid may differ in Query Editor v2.

### Why is this step required?

The assignment requires querying the loaded data. This `SELECT` query demonstrates that records stored in Redshift can be successfully retrieved.

### Dependency

Depends on:

- **Step 13:** Sample data loaded successfully.

### What happens if this step is skipped?

The assignment requirement to query and retrieve the stored data would not be demonstrated.

---

# Verification Step 3: Query Engineering Department Employees

### Action

Execute:

~~~sql
SELECT employee_id, employee_name, job_title, salary
FROM employee_data
WHERE department = 'Engineering'
ORDER BY salary DESC;
~~~

### Command Explanation

This query retrieves only employees who belong to the Engineering department and sorts them from highest salary to lowest salary.

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `SELECT` | Retrieves data from the table. |
| `employee_id` | Returns the employee identifier. |
| `employee_name` | Returns the employee name. |
| `job_title` | Returns the employee's job title. |
| `salary` | Returns the employee's salary. |
| `,` | Separates columns in the `SELECT` list. |
| `FROM` | Identifies the source table. |
| `employee_data` | Source table containing employee records. |
| `WHERE` | Applies a filtering condition. |
| `department` | Column evaluated by the filtering condition. |
| `=` | Equality comparison operator. |
| `'Engineering'` | String value that the `department` column must match. |
| `ORDER BY` | Sorts the query result. |
| `salary` | Column used for sorting. |
| `DESC` | Sorts values in descending order, from highest to lowest. |
| `;` | Terminates the SQL statement. |

### Expected Output

~~~text
employee_id | employee_name | job_title                 | salary
------------+---------------+---------------------------+----------
105         | Vikram Joshi  | Site Reliability Engineer | 105000.00
102         | Meera Patil   | Cloud Engineer            | 92000.00
108         | Ishita Rao    | Software Engineer         | 88000.00
101         | Aarav Sharma  | DevOps Engineer           | 85000.00
~~~

### What should I observe?

Only four Engineering department employees should be returned.

The employee with the highest salary should appear first because `DESC` sorts the salary values in descending order.

### Why does this confirm success?

This demonstrates that Redshift can:

- Read stored data.
- Select specific columns.
- Filter rows using `WHERE`.
- Compare text values.
- Sort results using `ORDER BY`.
- Apply descending sorting using `DESC`.

---

# Verification Step 4: Calculate the Average Salary by Department

### Action

Execute:

~~~sql
SELECT department, ROUND(AVG(salary), 2) AS average_salary
FROM employee_data
GROUP BY department
ORDER BY average_salary DESC;
~~~

### Command Explanation

This analytical query groups employees according to department and calculates the average salary for every department.

### Command Breakdown

| SQL Part | Explanation |
| -------- | ----------- |
| `SELECT` | Retrieves or calculates values for the result set. |
| `department` | Returns the department name. |
| `ROUND(...)` | Rounds a numeric result to the requested number of decimal places. |
| `AVG(salary)` | Calculates the arithmetic average of salary values in each group. |
| `AVG` | SQL aggregate function used to calculate an average. |
| `salary` | Numeric column supplied to the `AVG` function. |
| `2` | Specifies that the result should be rounded to two decimal places. |
| `AS` | Assigns an alias to a result column. |
| `average_salary` | Readable alias for the calculated average salary. |
| `FROM` | Identifies the source table. |
| `employee_data` | Source table containing employee records. |
| `GROUP BY` | Groups records sharing the same department value. |
| `department` | Column used to form the groups. |
| `ORDER BY` | Sorts the final aggregated result. |
| `average_salary` | Calculated result column used for sorting. |
| `DESC` | Sorts from highest average salary to lowest. |
| `;` | Terminates the SQL statement. |

### Expected Output

The result should contain one row for each department.

A result similar to the following should appear:

~~~text
department      | average_salary
----------------+---------------
Engineering     | 92500.00
Human Resources | 78000.00
Finance         | 76000.00
Marketing       | 68000.00
~~~

### What should I observe?

You should see four department groups:

- Engineering
- Human Resources
- Finance
- Marketing

The Engineering department should have the highest average salary in this sample dataset.

### Why does this confirm success?

This query demonstrates an important data-warehouse capability: analytical aggregation.

It proves that Redshift can:

- Group large datasets.
- Calculate aggregate values.
- Round calculated results.
- Sort analytical output.

---

# Verification Step 5: Verify the Complete Assignment Requirements

### Action

Confirm that both requirements from the problem statement have been successfully completed.

### Requirement Verification

| Assignment Requirement | Implementation | Expected Status |
| ---------------------- | -------------- | --------------- |
| Create a Redshift data warehouse | Created `xyz-redshift-namespace` and `xyz-redshift-workgroup` | `Completed` |
| Load some data using Query Editor | Inserted eight employee records using `INSERT INTO` | `Completed` |
| Query the data | Executed `SELECT` queries in Query Editor v2 | `Completed` |

### What should I observe?

At this stage, you should have:

~~~text
Amazon Redshift Serverless
        |
        +-- Namespace: xyz-redshift-namespace
        |
        +-- Workgroup: xyz-redshift-workgroup
        |
        +-- Database: dev
                |
                +-- Schema: public
                        |
                        +-- Table: employee_data
                                |
                                +-- 8 employee records
~~~

You should also have successfully executed:

~~~text
CREATE TABLE
INSERT INTO
SELECT COUNT(*)
SELECT *
SELECT with WHERE
SELECT with ORDER BY
SELECT with AVG and GROUP BY
~~~

### Why does this confirm success?

The original assignment requires a Redshift data warehouse, data loading through the query editor, and querying that data.

All three requirements have now been directly implemented and verified.

---

## Current Assignment Progress

~~~text
[Completed] Select AWS Region
        |
        v
[Completed] Open Amazon Redshift
        |
        v
[Completed] Check Cost and Free Trial
        |
        v
[Completed] Create Redshift Serverless Namespace
        |
        v
[Completed] Create Redshift Serverless Workgroup
        |
        v
[Completed] Verify Data Warehouse
        |
        v
[Completed] Open Query Editor v2
        |
        v
[Completed] Connect to dev Database
        |
        v
[Completed] Create employee_data Table
        |
        v
[Completed] Load 8 Sample Records
        |
        v
[Completed] Query All Data
        |
        v
[Completed] Filter and Analyze Data
        |
        v
[Completed] Verify Assignment Requirements
        |
        v
[Next] Delete Redshift Serverless Workgroup
        |
        v
[Next] Delete Redshift Serverless Namespace
        |
        v
[Next] Verify Final Cleanup
~~~

---

# Part 3: Cleanup, Cost Optimization, and Final Resource Verification

---

# Cleanup and Cost Optimization

The implementation and verification requirements of this assignment are now complete.

The following resources were created:

| Resource Type | Resource Name | Can Generate Charges? | Cleanup Required |
| ------------- | ------------- | --------------------- | ---------------- |
| Redshift Serverless Workgroup | `xyz-redshift-workgroup` | Yes | Yes |
| Redshift Serverless Namespace | `xyz-redshift-namespace` | Storage-related charges may apply | Yes |
| Database | `dev` | Stored inside the namespace | Removed with namespace |
| Table | `employee_data` | Stored inside the namespace | Removed with namespace |
| Sample employee records | 8 records | Stored inside the namespace | Removed with namespace |

The resources must be deleted in the correct reverse dependency order:

~~~text
Redshift Serverless Workgroup
        |
        v
Redshift Serverless Namespace
        |
        v
Database and Table Removed
        |
        v
Verify No Assignment Resources Remain
~~~

The required cleanup order is:

~~~text
1. Delete xyz-redshift-workgroup
2. Wait until workgroup deletion completes
3. Delete xyz-redshift-namespace
4. Wait until namespace deletion completes
5. Verify that no assignment resources remain
~~~

> **Important:** Do not leave the Redshift Serverless resources running after completing the assignment. Redshift Serverless is a chargeable AWS service and may consume promotional credits or generate charges depending on account eligibility and usage.

---

# Cleanup Step 1: Close Active Query Editor v2 Sessions

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Query Editor v2
~~~

### Action

Before deleting the Redshift Serverless resources:

1. Confirm that all required SQL queries have been executed successfully.
2. Confirm that all assignment verification steps are complete.
3. Save any screenshots required for your course submission.
4. Close unnecessary SQL editor tabs if desired.
5. Return to the Amazon Redshift console.

No SQL command is required for this cleanup step.

### Why is this required?

Closing or leaving the Query Editor workspace is not normally a mandatory technical prerequisite for deleting the Redshift Serverless resources. However, completing all query operations before deletion is important because deleting the namespace will permanently remove the database objects and sample data unless a snapshot is retained.

For this assignment, we assume that:

- All verification is complete.
- No backup is required.
- No final snapshot needs to be retained.
- The sample data is disposable.

### Dependency

Depends on:

- **Verification Step 5:** All assignment requirements must already be verified.

### What happens if this step is skipped?

The resources may still be deletable, but you could accidentally delete the database before capturing required verification results or screenshots.

---

# Cleanup Step 2: Delete the Redshift Serverless Workgroup

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
→ Select xyz-redshift-workgroup
→ Actions
→ Delete workgroup
~~~

Depending on the current AWS Console interface, the deletion action may also be available directly from the workgroup details page.

### Action

Select:

`xyz-redshift-workgroup`

Then choose the option to delete the workgroup.

If the console asks for confirmation:

1. Carefully verify that the selected workgroup is `xyz-redshift-workgroup`.
2. Enter any required confirmation text if prompted.
3. Confirm the deletion.

Do not delete an unrelated Redshift workgroup.

### Expected Result

The workgroup should enter a deletion state and eventually disappear from the list of active workgroups.

You may temporarily observe a status similar to:

~~~text
Deleting
~~~

After deletion completes, the workgroup should no longer be available.

### Why is this required?

The Redshift Serverless workgroup provides the compute and networking environment used to execute SQL queries.

Deleting it is important because:

- It is no longer required after verification.
- Redshift Serverless is a chargeable AWS service.
- Leaving unnecessary resources can consume AWS promotional credits or generate charges.
- The workgroup is associated with the namespace and should be removed before deleting the namespace.

### Dependency

Depends on:

- All implementation steps being complete.
- All verification steps being complete.
- No further SQL queries being required.

The cleanup order is:

~~~text
Delete Workgroup First
        |
        v
Wait for Workgroup Deletion
        |
        v
Delete Namespace Second
~~~

### What happens if this resource is not deleted?

Leaving the workgroup unnecessarily provisioned may result in continued Redshift-related costs depending on actual usage, account eligibility, current pricing, and configuration.

It also leaves an unnecessary cloud resource in the AWS account.

---

# Cleanup Step 3: Verify Workgroup Deletion

### Action

Wait until the workgroup deletion is complete before attempting to delete the namespace.

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
~~~

### Expected Result

The following workgroup should no longer appear as an active resource:

`xyz-redshift-workgroup`

### What should I observe?

You should observe one of the following:

- `xyz-redshift-workgroup` is no longer listed.
- The workgroup temporarily shows a deletion status and then disappears.

Do not proceed until deletion has completed.

### Why does this confirm success?

The absence of `xyz-redshift-workgroup` confirms that the Redshift Serverless compute and networking resource created for this assignment has been removed.

---

# Cleanup Step 4: Delete the Redshift Serverless Namespace

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Namespace configuration
→ Select xyz-redshift-namespace
→ Actions
→ Delete namespace
~~~

Depending on the current AWS Console interface, the delete action may also appear directly on the namespace details page.

### Action

Select:

`xyz-redshift-namespace`

Choose the option to delete the namespace.

During deletion, the AWS Console may ask whether you want to create a final snapshot.

For this assignment:

| Setting | Value |
| ------- | ----- |
| Namespace | `xyz-redshift-namespace` |
| Final snapshot | Do not retain one unless explicitly required by your instructor |
| Confirmation | Complete the confirmation requested by the console |

Because this assignment does not require a backup, and the sample data is disposable, retaining a final snapshot is unnecessary.

> **Important:** Only skip the final snapshot because this assignment does not require data preservation. In a production environment, backup, retention, recovery objectives, and organizational policies must be evaluated before deleting a data warehouse.

### Expected Result

The namespace should enter a deletion state and eventually disappear from the namespace list.

### Why is this required?

The namespace contains the logical data-storage objects created during the assignment, including:

~~~text
xyz-redshift-namespace
        |
        v
dev database
        |
        v
public schema
        |
        v
employee_data table
        |
        v
8 sample employee records
~~~

Deleting the namespace removes these assignment resources when no final snapshot is retained.

### Dependency

Depends on:

- **Cleanup Step 2:** `xyz-redshift-workgroup` deletion initiated.
- **Cleanup Step 3:** Workgroup deletion successfully verified.

The correct dependency order is:

~~~text
Workgroup
    |
    | Delete first
    v
Namespace
    |
    | Delete second
    v
Database Objects and Data Removed
~~~

### What happens if this resource is not deleted?

The namespace and its stored data remain in the AWS account.

Depending on current AWS pricing, storage consumption, account eligibility, and other configuration factors, retained Redshift resources may continue to contribute to costs or consume available promotional credits.

---

# Cleanup Step 5: Verify Namespace Deletion

### Action

Wait until the namespace deletion process is complete.

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Namespace configuration
~~~

### Expected Result

The following namespace should no longer appear as an active resource:

`xyz-redshift-namespace`

### What should I observe?

You should confirm that:

- `xyz-redshift-namespace` no longer appears in the namespace list.
- No deletion failure is displayed.
- No unwanted final snapshot was retained.

### Why does this confirm success?

The namespace contained the `dev` database, `employee_data` table, and all eight sample records.

Its successful deletion confirms that the data-storage environment created specifically for this assignment has been removed.

---

# Cleanup Step 6: Check for Unnecessary Redshift Snapshots

### Navigation

~~~text
AWS Console
→ Amazon Redshift
→ Snapshots
~~~

Depending on the current Redshift console interface, serverless snapshots may be accessible from a dedicated snapshots section or through the Redshift Serverless namespace-related pages.

### Action

Check whether a final or manual snapshot associated with `xyz-redshift-namespace` was accidentally created or retained.

If no snapshot exists, no action is required.

If a disposable snapshot was created only for this assignment and is no longer needed:

1. Select the snapshot.
2. Verify that it belongs to the assignment namespace.
3. Choose the delete action.
4. Confirm deletion.

Do not delete snapshots belonging to unrelated databases or production environments.

### Why is this required?

Snapshots retain database data for recovery purposes.

Although no snapshot is required for this assignment, one could be accidentally created during namespace deletion if the corresponding option was selected.

Unnecessary retained snapshots can consume storage and potentially contribute to costs.

### Dependency

Depends on:

- **Cleanup Step 5:** Namespace deletion completed.

### What happens if this resource is not deleted?

If an unnecessary snapshot exists and is retained, it can continue storing a copy of the Redshift data and may contribute to storage costs depending on applicable AWS pricing and account eligibility.

---

# Cleanup Step 7: Final Resource Verification

### Action

Verify that every AWS resource created specifically for this assignment has been deleted.

### Navigation

Check the following Redshift sections:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Workgroup configuration
~~~

Then:

~~~text
AWS Console
→ Amazon Redshift
→ Redshift Serverless
→ Namespace configuration
~~~

Then check snapshots:

~~~text
AWS Console
→ Amazon Redshift
→ Snapshots
~~~

### Final Resource Verification Table

| Resource | Resource Name | Expected Final State |
| -------- | ------------- | -------------------- |
| Redshift Serverless Workgroup | `xyz-redshift-workgroup` | `Deleted` |
| Redshift Serverless Namespace | `xyz-redshift-namespace` | `Deleted` |
| Database | `dev` | `Removed with namespace` |
| Schema | `public` | `Removed with namespace` |
| Table | `employee_data` | `Removed with namespace` |
| Sample employee records | 8 records | `Removed with namespace` |
| Assignment-specific snapshot | None required | `Not present` |

### What should I observe?

You should no longer find:

- `xyz-redshift-workgroup`
- `xyz-redshift-namespace`
- Any unnecessary snapshot created specifically for this assignment

The `dev` database and `employee_data` table are removed automatically as part of namespace deletion.

### Why does this confirm success?

This proves that:

- The Redshift Serverless compute environment has been removed.
- The namespace containing the assignment database and data has been removed.
- No unnecessary assignment-specific backup remains.
- The risk of unnecessary continued resource consumption has been minimized.

---

# Final Assignment Status

~~~text
[Completed] Create Amazon Redshift Data Warehouse
        |
        v
[Completed] Create Redshift Serverless Namespace
        |
        v
[Completed] Create Redshift Serverless Workgroup
        |
        v
[Completed] Connect Using Query Editor v2
        |
        v
[Completed] Create employee_data Table
        |
        v
[Completed] Load 8 Sample Employee Records
        |
        v
[Completed] Query All Data
        |
        v
[Completed] Filter Engineering Employees
        |
        v
[Completed] Calculate Average Salary by Department
        |
        v
[Completed] Verify All Assignment Requirements
        |
        v
[Completed] Delete Redshift Serverless Workgroup
        |
        v
[Completed] Delete Redshift Serverless Namespace
        |
        v
[Completed] Verify Complete Resource Cleanup
~~~

