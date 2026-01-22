# dbt_Core Setup

---
## 1. Environment Setup

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

# Alternatively, create folder and change directory to the new folder
mkdir dbt_project_name && cd dbt_project_name

# Create virtual environment
python3 -m venv venv_name

# Activate virtual environment ( with Windows); notice the prefix "venv" after activation
venv/Scripts/activate

# Activate virtual environment ( with Mac/Linux)
source venv/bin/activate
```
**Fix Execution Error in PowerShell for Windows:**

```PowerShell
If `Get-ExecutionPolicy` returns `Restricted`
Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted -Force`

Reverse command after venv activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Undefined`
```

### 1.3 Install dbt adapter (e.g, Databricks, Snowflake, PostgreSQL, etc.)

```PowerShell
# Install both dbt and dbt adapter
pip install dbt-core dbt-databricks # (for Databricks)
pip install dbt-core dbt-snowflake # (for Snowflake)
pip install dbt-core dbt-postgres # (for PostgreSQL)

# Freeze dependencies (to make venv reproducible)
pip freeze > requirements.txt
```
Alternatively, edit existing requirements.txt (for a previously used venv) to contain required dependencies ( dbt-core, dbt-postgres) and run;
```bash
pip install -r requirements.txt
```

> **pip freeze > requirements.txt** displays list of installed packages and save the list into a file called requirements.txt

> **Later (for reuse)**
Anyone cloning this Git repo just needs to do:
```bash
# Create virtual environment
python3 -m venv venv

#Activate it
venv\Scripts\activate (for Windows) or source venv/bin/activate (for Mac/Linux)

# Install dependencies from requirements.txt
pip install -r requirements.txt
```
> That recreates the same dbt environment exactly.

```bash
# Verify installed adapters
pip freeze

# Verify dbt installation
dbt --version
```
Expected output (example):

```yml
Core:
  - installed version: 1.8.5
Plugins:
  - postgres: 1.8.5
```
