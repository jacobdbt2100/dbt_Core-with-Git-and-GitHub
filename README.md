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

Your **profiles.yml** file (editable) is located here:

**Windows**:
> C:\Users\<YourUser>\.dbt\profiles.yml

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

## 3. Working with dbt Models
### 3.1 Create a model file

In **models/**, create **customers_view.sql**:

```sql
-- {{ config(materialized='view') }}
-- Optional: overrides the model’s materialization defined in schema.yml (not advised) or dbt_project.yml
-- (model-level config has the highest priority)

SELECT
    customer_id,
    name,
    gender,
    annual_income
FROM {{ source('raw', 'customers') }} --analytics.raw.customers
WHERE annual_income > 50000
```

### 3.2 Define the source

Create **models/schema.yml**:

```yml
version: 2

sources:
  - name: raw_customers_table # (or some other descriptive name; not restricted)
    schema: raw  # raw/source schema
    description: "customers raw data"
    tables:
      - name: customers

--Optional
models:
  - name: customers_view
    description: --add description here
    columns:
      - name: customer_id
        data_tests: --(this test is inside schema.yml; other options are- inside the "/tests" folder, as a standalone test macro in "macros/tests/"
          -  not_null
          -  unique
      - name: annual_income
        data_tests:
          - not_null 
    config:
      materialized: view --can be overridden or simply configured in the model(.sql) file
      alias: --transformed_customers --optional; to change the model name in the transformed schema
```
