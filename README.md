# dbt_Core Setup

---
## 1. Setup dbt Environment

### 1.1 Install the required Dependencies (tools)

| Dependency                                  | cmd Check if exist                                                                    |
| ------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Python 3.8+**                             | `python --version`, `python3 --version`, or `py -0` (to check all versions installed) |
| **pip**                                     | `pip --version` or `pip3 --version`                                                   |
| **adapter (database)**                      | `"adapter" --version` (for local setup, not cloud)                                    |
| **Git**                                     | `git --version`                                                                       |
| **VS Code** (or your preferred IDE)         | `code --version`                                                                      |

**Doc Reference:**
- [Connect to adapters](https://docs.getdbt.com/docs/connect-adapters)
- [Trusted adapters](https://docs.getdbt.com/docs/trusted-adapters)

### 1.2 Create and Activate a Virtual Environment

```PowerShell
# Create a folder for your project
mkdir dbt_project_folder

# Change directory to created folder
cd dbt_project_folder

mkdir dbt_project_folder && cd dbt_project_folder # (Alternatively, create folder and change directory to the new folder)

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
venv/Scripts/activate # (in Windows); notice the prefix "venv" after activation
source venv/bin/activate # (in Mac/Linux)
```
**Fix Execution Error (if exist) in PowerShell for Windows:**

```PowerShell
If `Get-ExecutionPolicy` returns `Restricted`
Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted -Force`

Reverse command after venv activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Undefined`
```

### 1.3 Install dbt core and Adapter (e.g., Databricks, Snowflake, PostgreSQL, etc.)

```PowerShell
# Install both dbt and dbt adapter
python -m pip install dbt-core dbt-databricks # (for Databricks)
python -m pip install dbt-core dbt-snowflake # (for Snowflake)
python -m pip install dbt-core dbt-postgres # (for PostgreSQL)

# Verify dbt installation
dbt --version

# Create requirements.txt to make venv reproducible
pip freeze > requirements.txt # creates (or overwrites) a requirements.txt file by writing out all the packages currently installed in your environment, with their exact versions.

# Install dependencies from requirements.txt (to reproduce an existing venv when needed)
pip install -r requirements.txt # tells pip to install all the Python packages listed inside a file named requirements.txt in one go.

# Verify installed adapters
pip freeze # displays installed dependencies in requirements.txt
```
Output example for `dbt --version`:

```yml
Core:
  - installed: 1.6.11
  - latest:    1.7.9

Plugins:
  - databricks: 1.6.11 # for databricks adapter
```

## 2. Connect dbt Project to Data Source

### 2.1 Initialize dbt Project and Configure Profile

- Create a `Database/Catalog` and `Schema` in Data Source Platform (e.g., Databricks, Snowflake, PostgreSQL, etc.)
- Initialize project:

```PowerShell
# Initialize
dbt init # or "dbt init dbt_project_name" (to initialize a specific dbt project out of many in the same environment)
```

**dbt will ask for:**

- **project name**: `dbt_project_name`
- **dbname**: `database name`
- **host**: dbc-71c78b23-9eaa.cloud.databricks.com # (for databricks at "Compute-Serverless Starter Warehouse-Connection details-Server hostname) or `localhost` for postgres
- **http_path**: /sql/1.0/warehouses/8e5d3729930bb8f2 # (for databricks at "Compute-Serverless Starter Warehouse-Connection details-HTTP path)
- **Desired access token option**: XXXXXXXXX # (for databricks at "Settings-Developer-Access tokens-Manage-Generate new token) # (won't appear on CLI screen; just paste and press enter)
- **Desired unity catalog option**: use Unity Catalog
- **catalog (initial catalog)**: catalog_name # or database_name
- **schema (default schema that dbt will build objects in)**: schema_name
- **threads**: 1 # the number of models dbt can run in parallel during execution

`others (for postgres):`
- **port**: # or `5432` for postgres
- **username**: `jacobdbt2100`
- **password**: `Mbo@12345678` (typing won't appear on CLI screen; just type and press enter)
- **Adapter**: `postgres` # for postgres
- **Profile**: `dbt_project_name`

Your **profiles.yml** file (editable) on **Windows** is located here:

`> C:\Users\<YourUser>\.dbt\profiles.yml`

**Example configuration**:

```yml
dbt_project_name:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: jacobdbt2100
      password: Mbo@12345678
      port: 5432
      dbname: database_name
      schema: schema_name
      threads: 1
```

### 2.2 Test the connection

```PowerShell
# Switch to project directory
cd dbt_project_name

# test connection
dbt debug # or "dbt debug --profile dbt_project_name" to target a specific dbt project
```

If successful, you’ll see:
```css
All checks passed!
```

## 3. Create and Define dbt Models, Sources & Tests
### 3.1 Create a model file

`Directory` **models/orders_model_view.sql:**

```sql
{{ config(materialized='view') }}
    -- Optional: overrides the model’s materialization defined in schema.yml (not advised) or dbt_project.yml (more preferred)
    -- model-level config has the highest priority

select
    order_id,
    item,
    order_quantity,
    total_amt
from {{ source('sales_data', 'orders') }} # Use the `source` function to select from source # this interpretes to analytics_db.raw.orders (i.e., database.schema.table)
where total_amt > 50000
```

### 3.2 Define the source(s)

`Directory` **models/source.yml:**

```yml
version: 2

sources:
  - name: sales_data          # This is the source name you'll use in dbt; choose any descriptive name
    description: "Raw sales data from the company's transactional system" # Optional
    database: analytics_db    # Optional: the database where the table lives; otherwise, dbt points to the default database
    schema: raw               # Schema in the database
    tables:
      - name: orders                  # The name you'll refer to in dbt
        identifier: orders_2026       # Optional: Actual table name in the database
        description: "Contains all orders placed by customers in 2026"
      - name: customers
        identifier: customers_info
        description: "Customer master data table"
```

### 3.3 Define the model(s)

`Directory` **models/model.yml:**

```yml
version: 2

models:
  - name: customers_view             # must match the actual model file name without the .sql extension
    description: "View of customers with key attributes and income info"
    columns:
      - name: customer_id            # must match the model file name
        description: "Unique ID for each customer"
      - name: annual_income
        description: "Customer's reported annual income"
      - name: first_name
        description: "Customer first name"
      - name: last_name
        description: "Customer last name"
    config:
      materialized: view             # Can also be 'table', 'incremental', etc.
      alias: transformed_customers   # Optional: physical table/view name in the DB
```

### 3.4 Use the "ref" function to select from other models

```sql
select *
from {{ ref('orders_model_view') }}
where order_quantity > 1
```

### 3.5 Run the model(s)

```PowerShell
# Run all models
dbt run

# Run a specific model only
dbt run --select model_name

# Run multiple specific models
dbt run --select model1 model2 model3

# Run models using pattern matching (e.g. all staging models)
dbt run --select stg_*

# Run a model and its upstream dependencies (parents)
dbt run --select +model_name

# Run a model and its downstream dependents (children)
dbt run --select model_name+

# Run a model, its upstream, and its downstream
dbt run --select +model_name+
```

**Note:**
- `--select` repalced `-m` and `--models` which are legacy aliases
- `Node selection` is a dbt feature that lets you run specific models (and their upstream/downstream dependencies) using selectors like `model`, `+model`, or `model+`

### 3.6 Define tests

**Two types of dbt tests:**

- `Generic (Built-in) tests:` pre-defined dbt tests applied via YAML.
- `Singular (Custom SQL) tests:` custom SQL queries that return rows if a condition fails.

**1. Generic (Built-in) tests**

`Directory` **models/source.yml:**

```yml
sources:
  - name: sales_data
    database: analytics_db
    schema: raw
    tables:
      - name: customers
        columns:
          - name: customer_id
            tests:
              - not_null    # Ensure no nulls in customer_id
              - unique      # Ensure customer_id is unique
          - name: customer_status
            tests:
              - not_null                    # Ensure every row has a status
              - accepted_values:            # Only allow defined statuses
                  values: ['active', 'inactive', 'prospect']
```

`Directory` **models/model.yml:**

```yml
models:
  - name: dim_customers
    columns:
      - name: customer_id
        data_tests:
          - not_null        # Ensures no null values in customer_id
          - unique          # Ensures customer_id is unique
          - relationships:  # Ensures customer_id exists in orders.customer_id
              to: ref('orders')
              field: customer_id

      - name: customer_status
        data_tests:
          - accepted_values:
              values:     # Only allows these statuses
                - active
                - inactive
                - prospect
```

**2. Singular (Custom SQL) tests**

`Directory` **tests/customers_email_valid.sql:**

```sql
-- Fails if any invalid emails exist in the customers model
select *
from {{ ref('customers') }}
where email not like '%@%.%'
```

### 3.7 Define test macros (custom generic tests)

To create reusable logic (e.g., check pattern validity across multiple tables), write a custom test macro.

`Directory` **macros/tests/email_pattern_test.sql:**

```sql
{% test email_pattern(model, column_name) %}
    select *
    from {{ model }}
    where {{ column_name }} not like '%@%.%'
{% endtest %}
```
Then, reference it in schema.yml:

```yml
columns:
  - name: email
    data_tests:
      - email_pattern
```

### 3.8 Test the source(s) and model(s)

`dbt tests` **validates data (checks data quality)** - like `missing values`, `duplicates`, `accepted values`, or `mismatched relationships between tables`.

```PowerShell
# Run all tests (models + sources)
dbt test
```

**dbt tests (models only):**

```PowerShell
# Run tests for a specific model only
dbt test --select model

# Run tests for multiple specific models
dbt test --select model1 model2 model3

# Run tests for a model and its upstream dependencies
dbt test --select +model

# Run tests for a model and its downstream dependents
dbt test --select model+

# Run tests for a model, its upstream, and its downstream
dbt test --select +model+
```

**dbt tests (sources only):**

```PowerShell
# Run all tests for all sources
dbt test --select source:*

# Run all tests for a specific source
dbt test --select source:sales_data

# Run tests for two (or more) sources
dbt test --select source:source1 source:source2 ...

# Run tests for a specific table in a source
dbt test --select source:sales_data.customers
```

**Note:**
- `Node selection` applies to `dbt test`, similar to `dbt run`

## 4. Version Control with Git

- `Git:` A version control system that tracks changes in files.
- `GitHub:` A cloud platform for hosting and collaborating on Git repositories.
- `Repository:` A project folder tracked by Git.

**.git Flow:** `Working Directory` > `Staging Area (Snapshot)` > `Repository`

**That is:** `Untracked (U) or Modified (M)` > `Staged (A)` > `Committed`

### 4.1 Git Initialize

**From the root of your dbt project folder, that is;**

`Directory` **dbt_project_name/**

```PowerShell
# Initialize git
git init # Converts the folder into a Git repository
```

**When you run git init:** Git creates a hidden `.git/` folder inside your project. That folder stores:
- Commit history
- Branches
- Configuration
- Staging area

**Note:**
- Your project is now **trackable by Git**, but it is still **local only**.
- `git init` does not connect to GitHub. It only prepares the folder for version control.

### 4.2 Git Add and Git Commit

An **initial commit** is needed to **register** the master (main) branch.

**Add:**

```PowerShell
git add .                                    # Stage all changes in the folder
git add file_name.ext                        # Stage only a specific file
git add file1.txt file2.csv file3.py         # Stage multiple specific files at once

# Stage files that share a pattern
git add *.py                                 # Stage all Python files in the folder
```

**Commit:**

```PowerShell
git commit -m "initial commit"               # Commit staged changes with a message

git commit file1.txt file2.py -m "Message"   # Commit only specific files, leaving others staged
```

### 4.3 Git Log


### 4.4 Gitignore & Gitkeep

### 4.5 Git Branches and Merges

**Branching:**

```PowerShell
git branch                   # List all local branches and show the current branch

git branch new_branch        # Create a new branch (does not switch to it)

git checkout new_branch      # Switch to an existing branch (new_branch)
git switch new_branch        # Alternative to checkout (Git 2.23+)

git checkout -b new_branch   # Create and switch to a new branch
git switch -c new_branch     # Alternative to checkout (Git 2.23+)
```

**Merging:**

```PowerShell
# In practice, switch to the branch you want to update first (usually main)
git switch branch_name        # Switch to branch_name
git merge feature_branch      # Merge changes from feature_branch into the current branch (branch_name)
```

**Rename current branch locally**

```PowerShell
git branch -m main           # Rename current branch to "main"
git branch -m master main    # Rename "master" to "main" from anywhere
```

**Update the remote branch**

```PowerShell
git push -u origin main           # Push renamed branch and track it remotely
git push origin --delete master   # Delete old branch from remote
```


### 4.6 Merge Conflicts

### 4.7 Git Rebase

### 4.8 Git Reflog and Commit History

### 4.9 Cherry Picking

### 4.10 Git Stashing

### 4.11 Push To GitHub

### 4.12 Git Clone and Push Feature Branch


**Common dbt-git workflow:**

`edit dbt model` > `dbt run` > `dbt test` > `git status` > `git add` > `git commit` > `git push`

### 4.13 Config Git Commands

```PowerShell
# Check current configuration
git config user.name
git config user.email

# Set your name and email
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Levels (Scopes) of git config;
System (--system); Applies to all users on the machine
Global (--global); Applies to your user account
Local (no flag); Applies only to the current project

# Remove a config
git config --global --unset user.name    # Use name
git config --global --unset user.email   # Use email
```




## 5. Advanced dbt Concepts
