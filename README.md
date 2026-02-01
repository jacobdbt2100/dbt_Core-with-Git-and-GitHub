# dbt Core with Git & GitHub

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

**Note:**

```PowerShell
# Install Python packages using the "default pip" (may point to global Python)
pip install package_name

# Install Python packages using the "active Python environment"
python -m pip install package_name
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
{{ config(materialized='view') }}      -- Optional

select
    order_id,
    item,
    order_quantity,
    total_amt
from {{ source('sales_data', 'orders') }} -- # Use the `source` function to select from source # this interpretes to analytics_db.raw.orders (i.e., database.schema.table)
where total_amt > 50000
```

**dbt model materialization priority (highest > lowest):**

```PowerShell
# 1. Model config block
# Inline config inside the model SQL file; most explicit and overrides all others
{{ config(materialized='table') }}

# 2. Properties file (.yml)
# A properties file is any YAML file under models/ that defines metadata or configuration for dbt resources; schema.yml is the default name.
models:
  - name: customers
    config:
      materialized: view

# 3. dbt_project.yml
# Project- or folder-level defaults that apply broadly unless overridden
models:
  my_project:
    +materialized: view
```

**dbt alias configuration** follows the **same priority rules** as `materialized` and other dbt configs.

**alias** is an **optional dbt config** that defines the final database object name (table or view); if omitted, **dbt uses the model filename by default**.

```PowerShell
{{ config(alias='model_alias') }}      -- Optional
```

**Other common dbt model configs (besides materialized and alias):**
- `schema`: override the target schema for a model
- `database / catalog`: override the target database or catalog
- `enabled`: include or exclude a model from runs
- etc.

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
      - name: customer_id
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

### 3.9 Other `dbt` commands

```PowerShell
# Checks configs, YAML, and Jinja; does not touch the database
dbt parse      # confirm: command finishes without errors

# Renders models into final SQL; does not execute anything
dbt compile    # confirm: compiled SQL appears in target/compiled/

# Removes build artifacts (target/, dbt_packages/)
dbt clean      # confirm: folders are deleted and recreated on next run

# Runs models, tests, snapshots, and seeds in dependency order
dbt build      # confirm: models created and tests pass without errors
```

### 3.10 dbt folders and uses

**1. analyses**
  - Temporary SQL queries for exploration or reporting.
  - Not models - not materialised in your warehouse.
  - Useful for ad-hoc analysis.

**2. dbt_packages**
- Installed dbt dependencies (packages pulled from dbt Hub or Git).
- Auto-generated when you run `dbt deps`.
- Not manually edited and usually ignored by Git, just like `logs/` and `target/`.

**3. logs**
  - Stores dbt run logs.
  - Mostly for debugging or reviewing past runs.
  - You usually don’t edit anything here.

**4. macros**
  - Reusable SQL snippets (functions).
  - Helps avoid repeating SQL code.
  - Can be called in models, tests, analyses, snapshots.

**5. models**
  - Where your main SQL models live.
  - Materialised as tables or views in your warehouse.
  - Can have subfolders for organisation.

**6. seeds**
  - CSV files that dbt loads into your warehouse.
  - Good for static reference data like country codes or lookup tables.

**7. snapshots**
  - Capture slowly changing data over time.
  - Useful for tracking historical changes.
  - Stored as versioned tables in your warehouse.

**8. targets**
  - Auto-generated folder by dbt.
  - Stores compiled SQL and temporary run artefacts.
  - You usually do not edit it.

**9. tests**
  - Contains singular / custom SQL tests for models or business logic.
  - Runs alongside generic tests in schema.yml.

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

**To stop git tracking:**

```PowerShell
Remove-Item -Recurse -Force .git   # In Windows PowerShell
                                   # stop git tracking by removing the hidden .git repository folder

git status                         # check if the folder is no longer a git repository (should fail)
```

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

**Displays commit history and metadata**.

```PowerShell
git log   # Show the commit history of the current branch
```

**It shows:**
- Commit hashes (IDs)
- Author
- Date and time
- Commit messages and
- Optionally, the changes made

**Optional Variations:**

```PowerShell
git log --oneline     # Show each commit on a single line

# Press "Q" to unfreeze when PowerShell freezes after "git log", possibly due to so many logs.

git log --graph       # Show branch structure visually
git log --stat        # Show summary of files changed and lines added/removed in each commit
git log -p            # Show the actual code changes (diff) for each commit
git log --name-only   # Show only the names of files changed
git log --decorate    # Show branch and tag names on commits

git log --oneline --graph --all   # Clean visual history of all branches
```

### 4.4 Gitignore & Gitkeep

**.gitignore:**

A file that instructs Git to **exclude specified files or directories from version control**.

It is **typically** used to **exclude**:
- Secrets (e.g., .env, API keys)
- Temporary files
- Build outputs (automatically generated by tools when your project runs, compiles, or builds)
- Large datasets

**Another advantage of `.gitignore:`** It **prevents ignored files from appearing as untracked in `git status`**, keeping the working directory list clean and focused.

**Location:**
- `.gitignore` should be created in the root directory (parent folder) of your Git repository.

**Add files/folders to ignore:**

```text
file_name.txt   # Ignore file_name.txt

.env            # Ignore environment files

*.log           # Ignore all log files

build/          # Ignore any 'build' folder relative to the location of this .gitignore file

/build/         # Ignore only the 'build' folder in the root of the repository

data/*.csv      # Ignore dataset CSVs
```

- You can also have additional `.gitignore` files in subdirectories to apply rules locally to that folder as below:

`Directory` **data/**

```text
*.csv   # Ignore all CSV files only within the data directory
```

**Note:**

Once a file is **already tracked by Git**, adding it to **`.gitignore` does not remove it from version control** - you must **untrack** it first with:

```PowerShell
git rm --cached file_name
```

**.gitkeep:**

Git, **by default, does not track empty folders**, but you can keep them using a placeholder file if needed for project structure or future content. To achieve this, Place one `.gitkeep` in each folder you want to track.

### 4.5 Git Branches and Merges

**Branch:**

```PowerShell
git branch                   # List all local branches and show the current branch

git branch new_branch        # Create a new branch (does not switch to it) # The new branch is created from the head (latest commit) of the current branch (usually main) by default

git checkout new_branch      # Switch to an existing branch (new_branch)
git switch new_branch        # Alternative to checkout (Git 2.23+)

git checkout -b new_branch   # Create and switch to a new branch
git switch -c new_branch     # Alternative to checkout (Git 2.23+)
```

**Merge:**

```PowerShell
# In practice, switch to the branch you want to update first (usually main)
git switch branch_name        # Switch to branch_name
git merge feature_branch      # Merge changes from feature_branch into the current branch (branch_name)
```

`Two main types of Git merges:`
- **Fast-forward merge (FF merge):** Git moves the target branch pointer forward to the source branch’s latest commit **without creating a merge commit**, because the target branch has no new commits since the branches diverged.
- **Three-way merge (normal merge):** Git combines changes from two branches by creating a new merge commit when both branches have new commits.

It’s called **three-way** because Git uses three points to compute the merge: `Base commit`; the common ancestor of the branches, `Head of target branch`; where you are merging into, `Head of source branch`; the branch being merged.

**Rename current branch locally:**

```PowerShell
git branch -m main           # Rename current branch to "main"
git branch -m master main    # Rename "master" to "main" from anywhere
```

**Update the remote branch:**

```PowerShell
git push -u origin main           # Push renamed branch and track it remotely
git push origin --delete master   # Delete old branch from remote
```

**Delete a local branch:**

```PowerShell
git branch -d branch_name   # Delete local branch (only if fully merged)
git branch -D branch_name   # Force delete local branch (even if not merged)
```

**Delete a remote branch:**
```PowerShell
git push origin --delete branch_name   # Delete branch from remote repository
```

### 4.6 Merge Conflicts

A situation where Git **cannot automatically merge changes** because **two or more branches modified the same or overlapping lines in a file**. Git needs human decision to resolve competing changes.

### 4.7 Git Rebase

- **Rewrites commit history** by taking commits from one branch and replaying them on top of another branch.
- Changes commit hashes and order.
- Intended to create a linear history but modifies history, unlike FF merge.

**Benefits of rebase:**
- `Cleaner history;` linear commit log without unnecessary merge commits.
- `Easier to read git log;` clearer story of how changes evolved.
- `Keeps main branch tidy;` especially useful before merging feature branches.
- `Helps resolve conflicts earlier;` you handle them while updating your branch, not at merge time.

**Rebase:**

```PowerShell
git switch rebase_feature      # Move to the rebase_feature branch
git rebase main                # Replay rebase_feature commits on top of main's latest commit
```

`git rebase main` takes the commits that exist on the **current branch** (`rebase_feature`) onto the **HEAD (latest commit)** of main, then moves the **current branch** (`rebase_feature`) to point to this new history.

**Merge after Rebase:**

```PowerShell
git switch main                # Move back to main
git merge rebase_feature       # Fast-forward main to include rebased commits (no merge commit created)
```

`Merge commit` is created when both branches contain unique commits, so Git must combine histories instead of moving the branch pointer.

### 4.8 Git Reflog and Commit History

```PowerShell
git reflog          # Shows local history of HEAD movements (even hidden commits)

# Compared to "git log --oneline"
git log --oneline   # Shows visible commit history of the current branch
```

**Benefits of `git reflog`:**
- `Recover lost commits;` find commits removed by reset, rebase, or amend - commit that’s no longer in the visible history (time travel).
- `Undo mistakes;` move HEAD back to a previous state or lets you rewind your branch to a previous point in time (time travel).
- `Track local actions;` lets you see recent branch and HEAD movements not visible in git log.

**Two practical ways to `time travel` in Git:**

1. Go to a previous commit using HEAD reference
2. Go to a specific commit using its hash/ID (recommended)

```PowerShell
git reset --hard HEAD~1        # Move HEAD to the previous commit (1 step back)
git reset --hard HEAD~2        # Move HEAD back 2 commits
git reset --hard <commit-id>   # Move HEAD to a specific commit hash (recommended)
```

`git reset --hard <commit-id>` can **restore a committed file (undo push commit)** that was **deleted** from the **working directory** and **staged**.

**Similar commands:**

```PowerShell
git reset                       # Unstage all staged changes, move them back to the working directory (edits remain intact)
git reset file1.txt file2.txt   # Unstage only these files, keep others staged

git reset HEAD~1                # Move HEAD back 1 commit, unstages changes but keeps file edits in working directory
```

### 4.9 Git Diff

`git diff:` shows **changes between files or commits** - what has changed but not yet staged (**working directory**) or changes staged for commit (**staging area**), and differences between commits or branches.

**Common uses:**

```PowerShell
git diff                  # Changes in working directory not staged
git diff --staged         # Changes staged for next commit
git diff HEAD             # All changes since last commit
git diff branch1 branch2  # Differences between two branches
```

`git diff` **example output:**

```diff
diff --git a/app.py b/app.py
index e69de29..4b825dc 100644
--- a/app.py                  # original version of the file (before changes)
+++ b/app.py                  # new version of the file (after changes)
@@ -0,0 +1,3 @@
+print("Hello World")         # Line added in working directory (unstaged change)
+print("Git is fun")          # Another line added
```

### 4.10 Cherry Picking

`git cherry-pick:` applies **one or more specific commit(s)** from another branch onto the current branch, bringing in **only the chosen changes without merging the entire branch**.

```PowerShell
git switch main                       # Move to the branch where you want the commit

git cherry-pick <commit-hash>         # Apply the specified commit from another branch onto the current branch # Creates a new commit on the target branch with a new commit hash

git cherry-pick <commit1> <commit2>   # Apply multiple specific commits in order # Use original commit order to avoid conflicts
git cherry-pick a1b2c3d..f6g7h8i      # Apply all commits between these two commits
```

### 4.11 Git Stashing

`git stash` **temporarily saves uncommitted changes** in the **working directory** and **staging area**, enabling safe branch switches without committing unfinished code. Git **blocks** `git switch` or `git checkout` to **prevent uncommitted changes from being lost or overwritten**.

**Benefits:**
- Allows developers to handle urgent tasks (e.g., bug fixes) on another branch
- Preserves original work safely, so it can be reapplied later

```PowerShell
git stash push -m "WIP: message"                           # Save all uncommitted changes with a message
git stash push -m "WIP file1, file2" file1.txt file2.txt   # Stash only specific files with a message

git stash list                                             # Show all stashes with their indices

git stash apply                                            # Reapply the latest stash without removing it
git stash apply "stash@{n}"                                # Apply the stash at index n (e.g., 0 = most recent)

git stash pop                                              # Applies and removes stash@{0}
git stash pop "stash@{n}"                                  # Apply and remove a specific stash

git stash drop                                             # Drops stash@{0}
git stash drop "stash@{n}"                                 # Delete a specific stash without applying it
```

### 4.12 Push To GitHub

- **Local repository:** The Git repository on your machine where you **make commits and manage your project history locally**.
- **Remote repository:** A copy of the **repository hosted on a server** (e.g., GitHub) for **sharing, backup, and collaboration**.
- `git push` sends commits **from your local repository** to **the remote repository** (GitHub)
- This is how your local work becomes visible and accessible online to others.

**Git Config:**

```PowerShell
# Check current configuration
git config user.name
git config user.email

# Set your name and email (Global Config Recommended) # Overwrites previous Config, if any.
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Levels (Scopes) of git config;
System (--system); Applies to all users on the machine
Global (--global); Applies to your user account (RECOMMENDED)
Local (no flag); Applies only to the current project

# Remove a config
git config --global --unset user.name    # Use name
git config --global --unset user.email   # Use email
```

**Push Local Repo to Remote (HTTPS Method):**

1. **Get `Personal access token`;** Click on `Profile` > `Settings` > `Developer Settings` > `Personal acess tokens` > `Tokens (classic)` > `Generate new token` > `Generate new token (classic)`
2. **Get `url`;** Click on `Code` and copy HTTPS
3. **Confirm `remote origin`:**

```PowerShell
git remote -v    # Show linked remote repositories and their fetch/push URLs
```
4. **Add `remote origin`:**

```PowerShell
git remote add origin "url"    # Link the local repository to a remote named "origin"
```
5. Confirm `remote origin` again (`git remote -v`): shows both where Git pulls from and where it pushes to, even if they are identical.

**Different fetch/push URLs** let you **pull from one place** and **push to another**, supporting forks, access restrictions, or **specialized workflows** (e.g., **development & deployment**).

```PowerShell
git remote set-url origin https://github.com/dev-team/project.git       # Fetch updates from the development repository
git remote set-url --push origin git@github.com:deployment/project.git  # Push changes to the deployment repository
```
6. Final steps:

```PowerShell
git switch main   # Switch to branch to be merged (main)

git push origin main       # Push local 'main' to remote 'origin'
                           # Does NOT set an upstream; future pushes require specifying remote and branch

git push -u origin main    # Alternative to "git push origin main"
                           # Sets 'main' as upstream for the local branch
                           # Future 'git push' or 'git pull' can be done without specifying remote/branch

git branch --unset-upstream   # Remove upstream link; future git push/pull must specify remote/branch
```

Then **connect to GitHub** using **Personal access token** or some other method (from pop up on screen)

**If error like...**

```diff
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/user/repo.git'
hint: Updates were rejected because the remote contains work that you do
not have locally. This is usually caused by another repository pushing
to the same ref. You may want to first integrate the remote changes
(e.g., 'git pull ...') before pushing again.
```
**Explanation:**
- Git **prevents** you from **overwriting commits on the remote** that your local **branch doesn’t have**, to avoid losing someone else’s work.
- You need to **sync with the remote first**, either by `git pull` (merge or rebase) or by force pushing if you really intend to overwrite.

```PowerShell
git pull origin main   # Fetch and merge changes from the remote 'main' branch into the current local branch
                       # Might fail due to unrelated histories 

git pull origin main --allow-unrelated-histories    # Fetch and merge the remote 'main' branch even if the local and remote histories are unrelated
                                                    # Git normally refuses this when the local repo was created from scratch
                                                    # Recommended workflow to avoid this issue: create the remote repo first, then clone locally

git add .

git commit -m "commit_message"

git push origin main   # Push local 'main' to remote 'origin'
```

**Bonus: `To undo pushed commit on the remote`**

```PowerShell
git reset --hard <previous_commit_hash>   # Move your local main back (undo commit)
git push --force origin main              # Overwrite remote main with your local state
```

### 4.13 Git Clone and Push Feature Branch

**Git Clone:**

```PowerShell
git clone "url"    # Download the full repo (files + history) into a new local folder
                   # Automatically sets the remote name as "origin"
                   # No need to run git init when you clone
                   # Easier workflow: create the remote repo first, then clone locally

cd project-folder       # Change into the cloned repository folder
git status              # Confirm this is a Git repository and see current branch/status
```

**Push Feature Branch:**

Whenever we create a repository, we **do not make changes directly on the main branch**. Instead, we **create and work on** a separate branch called a **feature branch**.

```PowerShell
git switch -c feature_1      # Create and switch to a new branch named "feature_1"

# Add a new file or modify an existing one

git status                   # Check the state of the working directory and staging area
git add .                    # Stage all changes
git commit -m "commit message"  # Commit staged changes with a message

git branch                   # List branches and show the current branch (* = active)
                             # We are currently on the feature branch
                             # You could merge feature_1 into main locally, but best practice is:
                             # Push the feature branch and merge via a Pull Request on GitHub

git remote -v                # Confirm the remote named "origin"
                             # If the repo was cloned, Git automatically sets the cloned URL as origin

git push origin feature_1    # Push the local "feature_1" branch to the remote (origin)
                             # If the repo was cloned and credentials already exist, Git will not prompt again
                             # Git only prompts for a Personal Access Token when no valid credentials are available
                             # If the token is revoked or expires, Git will request a new one

# The pushed branch now appears on GitHub with a "Compare & pull request" option
# You will see two branches: main and feature_1
# Open a Pull Request by clicking "Compare & pull request"
# Add a title (optional), add a description (optional), review changes (Unified or Split view)
# Click "Create pull request" > "Merge pull request" > "Confirm merge" to accept the changes

# The new file (or modification) is now merged into the main branch
```

**Bonus: `Common dbt-git workflow`**

`edit dbt model` > `dbt run` > `dbt test` > `git status` > `git add` > `git commit` > `git push`

## 5. Advanced dbt Concepts
