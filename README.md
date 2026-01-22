# dbt_Core Setup

---
## 1. Setup dbt Environment

### 1.1 Install the required dependencies (tools)

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

### 1.2 Create and activate a virtual environment

```PowerShell
# Create a folder for your project
mkdir dbt_project_name

# Change directory to created folder
cd dbt_project_name
mkdir dbt_project_name && cd dbt_project_name # (Alternatively, create folder and change directory to the new folder)

# Create virtual environment
python3 -m venv venv_name

# Activate virtual environment
venv/Scripts/activate # ( with Windows); notice the prefix "venv" after activation
source venv/bin/activate # ( with Mac/Linux)
```
**Fix Execution Error (if exist) in PowerShell for Windows:**

```PowerShell
If `Get-ExecutionPolicy` returns `Restricted`
Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted -Force`

Reverse command after venv activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Undefined`
```

### 1.3 Install dbt core and adapter (e.g., Databricks, Snowflake, PostgreSQL, etc.)

```PowerShell
# Install both dbt and dbt adapter
pip install dbt-core dbt-databricks # (for Databricks)
pip install dbt-core dbt-snowflake # (for Snowflake)
pip install dbt-core dbt-postgres # (for PostgreSQL)

# Create requirements.txt to make venv reproducible
pip freeze > requirements.txt # creates (or overwrites) a requirements.txt file by writing out all the packages currently installed in your environment, with their exact versions.

# Install dependencies from requirements.txt (to reproduce an existing venv when needed)
pip install -r requirements.txt # tells pip to install all the Python packages listed inside a file named requirements.txt in one go.

# Verify installed adapters
pip freeze # displays installed dependencies in requirements.txt

# Verify dbt installation
dbt --version
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

### 2.1 Initialize dbt project

- Create a `Database/Catalog` and `Schema` in Data Source Platform (e.g., Databricks, Snowflake, PostgreSQL, etc.)
- Initialize project:

```PowerShell
# Initialize
dbt init # (or "dbt init dbt_project_name" to initialize a specific dbt project out of many in the same environment)
```

**dbt will ask for:**

- **project name**: `dbt_project_name`
- **host**: # or `localhost` for postgres
- **port**: # or `5432` for postgres
- **username**: `jacobdbt2100`
- **password**: `Mbo@12345678` (typing won't appear on screen; just type and press enter)
- **dbname**: `database name`
- **schema**: `schema name`
- **threads**: 1 # the number of models dbt can run in parallel during execution
- **Adapter**: # or `postgres` for postgres
- **Profile**: `dbt_project_name`

`dbt init dbt_project_name` skips the first question: “Enter a name for your project”
