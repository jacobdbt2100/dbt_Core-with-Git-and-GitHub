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

## 3. Working with dbt Models
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

### 3.4 Use the `ref` function to select from other models

```sql
select *
from {{ ref('orders_model_view') }}
where order_quantity > 1
```







### 3.5 Run and test the model

```bash
dbt run
```
Alternatively, to run a specific model;
```bash
dbt run --select customers_view
```
`customers_view` is the only model in this example.

Generally, to run multiple selected models;
```bash
dbt run -m model1 model2 model3 ...
```
Or equivalently;
```bash
dbt run --select model1 model2 model3 ...
```

`-m`(or `--models`) is the older, common shortcut.

Check in PostgreSQL:
```sql
SELECT * FROM dbt_schema.customers_view;
```

### 3.6 Add tests (optional but recommended)

In **schema.yml**, add:
```yml
models:
  - name: customers_view
    columns:
      - name: customer_id
        data_tests:
          - not_null
          - unique
```

Then run:
```bash
dbt test
```
`dbt tests` help **validate data quality** — like `missing values`, `duplicates`, or `mismatched relationships between tables`.

> **Where Tests Live:**

**(a) Inside schema.yml** (generic tests)

✅ This is the most common and recommended way.

Example:

```yml
version: 2

models:
  - name: fact_orders
    columns:
      - name: customer_id
        data_tests:
          - relationships:
              field: customer_id
              to: ref('dim_customers')
              
  - name: customers
    description: "Customer data"
    columns:
      - name: customer_id
        data_tests:
          - not_null
          - unique
      - name: email
        data_tests:
          - not_null
```
Here, dbt will automatically generate and run tests for:
- Missing customer_id values
- Duplicate customer_id values
- Missing email values

These are **generic tests** (built-in).

**(b) Inside the /tests folder** (singular / custom SQL tests)

This is for custom SQL tests you create manually.

Example folder:

```pgsql
dbt_project/
├── models/
│   ├── staging/
│   └── marts/
├── tests/
│   └── customers_email_valid.sql
```
Content of `customers_email_valid.sql`:

```sql
-- Fail if any invalid emails exist
select *
from {{ ref('customers') }}
where email not like '%@%.%'
```
**(c) As a standalone test macro**

If you want reusable logic (e.g., check pattern validity across multiple tables), you can write a custom test macro in `/macros/tests/`.

Example macros/tests/test_email_pattern.sql:

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
**TESTS** Summary:

`tests/` **folder**: singular / custom SQL tests (complex rules, multi-column checks, business logic)

`schema.yml`: generic tests (not_null, unique, accepted_values, relationships)

**model SQL via** `config()`: can define generic tests, but **less common / not recommended**

> **Summary — What Goes Where**

| Type              | Purpose                                                                     | Location                | Defined in  |
| ----------------- | --------------------------------------------------------------------------- | ----------------------- | ----------- |
| Generic Test      | Common checks like `not_null`, `unique`, `accepted_values`, `relationships` | `schema.yml`            | YAML syntax |
| Custom SQL Test   | One-off data quality checks                                                 | `/tests/` folder        | SQL query   |
| Custom Macro Test | Reusable test logic                                                         | `/macros/tests/` folder | Jinja macro |

> **How to Run Tests:**

| **Goal**                                               | **Command**                                                       |
| ------------------------------------------------------ | ----------------------------------------------------------------- |
| All tests in project                                   | `dbt test`                                                        |
| All tests for `customers` model                        | `dbt test --select customers`                                     |
| All tests for `orders` model                           | `dbt test --select orders`                                        |
| Tests for a specific column (`customer_id`)            | `dbt test --select customers.customer_id`                         |
| Only `not_null` tests for `customers`                  | `dbt test --select customers -s test_type:not_null`               |
| Only `unique` tests for `customers`                    | `dbt test --select customers -s test_type:unique`                 |
| Only `not_null` tests for `orders`                     | `dbt test --select orders -s test_type:not_null`                  |
| Only `unique` tests for `orders`                       | `dbt test --select orders -s test_type:unique`                    |
| Specific SQL test file (e.g., `invalid_email.sql`)     | `dbt test --select invalid_email`                                 |
| Run tests for both models together                     | `dbt test --select customers orders`                              |
| Run only `not_null` and `unique` tests for both models | `dbt test --select customers orders -s test_type:not_null,unique` |

## 4. Version Control with Git
### 4.1 Initialize Git
