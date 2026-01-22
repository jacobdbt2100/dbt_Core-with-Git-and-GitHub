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

```git bash
# Create a folder for your project
mkdir dbt_postgres_project

# Change directory to created folder
cd dbt_postgres_project

# Alternatively, create folder and change directory to the new folder
mkdir dbt_postgres_project && cd dbt_postgres_project

# Create virtual environment
python3 -m venv venv

# Activate it (Windows); notice the prefix "venv" after activation
venv/Scripts/activate

# (Mac/Linux)
source venv/bin/activate
```
**Fix Execution Error in PowerShell for Windows:**

```bash
If `Get-ExecutionPolicy` is `Restricted`
Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted -Force`

Reverse command after venv activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Undefined`
```
