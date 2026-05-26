# dbt Setup on Windows Without Admin Access

## Purpose

This guide explains how to set up dbt Core with Snowflake on Windows when you do not have local administrator access.

It uses the Python embeddable package and installs dbt into the user profile.

## What this guide covers

- Python embeddable package
- pip
- dbt Core
- dbt Snowflake adapter
- Snowflake profile setup
- Git path setup
- Common setup issues

## Tutorial context

This setup was created while following the dbt Fundamentals VS Code course.

The course setup section focuses on installing the dbt VS Code extension and dbt Fusion engine. In this Windows environment, dbt Fusion was not reliable enough to complete the setup, so this guide documents a dbt Core alternative that supports the same core learning workflow.

Relevant course pages:

- Install the dbt VS Code Extension and dbt Fusion Engine
- Clone a Git repository
- Create a profiles.yml file

Course link:

```text
https://learn.getdbt.com/learn/course/dbt-fundamentals-vs-code/set-up-dbt-60min/getting-started
```

## Assumptions

- Commands are run in PowerShell.
- VS Code is already installed.
- Snowflake credentials are available.
- Git is either already installed or available from a user-level install.
- This setup uses dbt Core, not dbt Fusion.

## Why this guide uses dbt Core instead of dbt Fusion

This setup originally attempted to use the dbt Fusion engine. For this Windows setup, dbt Core was more reliable and easier to debug, so the final guide uses dbt Core with the Snowflake adapter.

dbt Fusion is a newer Rust-based dbt engine. dbt Labs describes it as a faster local development experience with SQL comprehension and column-level lineage. However, the local CLI and VS Code extension were listed as Preview when this guide was written, so dbt Core was the safer choice for a constrained Windows setup.

The main reasons for using dbt Core here are:

- It can be installed with Python and pip in the user profile.
- It works without local administrator access.
- It supports a straightforward Snowflake username and password profile.
- It avoids browser redirect and localhost authentication behaviour that may fail on restricted networks.
- It gives clearer, standard command-line errors when something is misconfigured.
- It matches the commands used in many dbt Core tutorials and CI/CD examples.

This does not mean dbt Fusion is wrong. Fusion may be worth revisiting later, especially for faster local development and richer IDE features. The dbt project code remains standard dbt, so it can be tested with Fusion later if the environment supports it.

For current Fusion status and installation guidance, check the official dbt documentation:

- https://docs.getdbt.com/docs/fusion/about-fusion
- https://docs.getdbt.com/guides/fusion
- https://docs.getdbt.com/docs/local/install-dbt

## 0. Define paths and make dbt available

Run these commands first in PowerShell.

```powershell
$PythonHome = "$env:USERPROFILE\tools\python311"
$DbtScripts = "$env:APPDATA\Python\Python311\Scripts"
$DbtExe = "$DbtScripts\dbt.exe"
$env:PATH="$DbtScripts;$env:PATH"
```

This adds the user-level Python scripts folder to the current PowerShell session, so `dbt` can be run directly after it is installed.

If you open a new terminal, you may need to run the PATH command again:

```powershell
$env:PATH="$env:APPDATA\Python\Python311\Scripts;$env:PATH"
```

Most commands in this guide use `dbt`.

If `dbt` is not recognised, either run the PATH command above again or replace `dbt` with the full executable path.

Example:

```powershell
& $DbtExe --version
```

## 1. Install Python embeddable package

### Download Python

Open the Python 3.11.9 release page:

```text
https://www.python.org/downloads/release/python-3119/
```

Download:

```text
Windows embeddable package (64-bit)
```

### Extract Python

Create this folder:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\tools"
```

Extract the Python embeddable ZIP file to:

```text
C:\Users\<your_user>\tools\python311
```

The final folder should look like this:

```text
C:\Users\<your_user>\tools\python311\python.exe
```

## 2. Enable site-packages

Open this file:

```text
C:\Users\<your_user>\tools\python311\python311._pth
```

Find this line:

```text
# import site
```

Change it to:

```text
import site
```

Save the file.

This allows Python to load packages installed by pip.

## 3. Install pip

Run:

```powershell
cd $PythonHome
Invoke-WebRequest -Uri "https://bootstrap.pypa.io/get-pip.py" -OutFile "get-pip.py"
.\python.exe get-pip.py
```

Verify pip works:

```powershell
.\python.exe -m pip --version
```

## 4. Install dbt Core and dbt Snowflake

Run:

```powershell
.\python.exe -m pip install --user dbt-core dbt-snowflake
```

Verify dbt works:

```powershell
dbt --version
```

If `dbt` is not recognised, use the fallback command:

```powershell
& $DbtExe --version
```

If either command works, dbt is installed.

## 5. Initialise the dbt project and profile

If you are starting a new dbt project, use `dbt init` first.

```powershell
dbt init
```

If `dbt` is not recognised, use the fallback command:

```powershell
& $DbtExe init
```

`dbt init` can create a new dbt project and guide you through creating an initial `profiles.yml` file.

If you have cloned an existing dbt project, do not use `dbt init` to recreate the project files. Instead, move into the cloned project folder and test the existing project.

Example:

```powershell
cd "C:\path\to\your\dbt_project"
dbt debug
```

If `dbt` is not recognised, use:

```powershell
& $DbtExe debug
```

If dbt cannot find a profile, use one of these options:

- Run `dbt init` from a suitable parent folder to generate a starting profile, then review the generated `profiles.yml`.
- Create `profiles.yml` manually.
- Update an existing `profiles.yml` manually.

The profile file can always be edited later.

## 6. Configure or update Snowflake profile manually

Use this section if `dbt init` did not create the profile, or if you need to change the Snowflake connection details.

Create the dbt profile folder:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.dbt"
```

Create this file:

```text
C:\Users\<your_user>\.dbt\profiles.yml
```

Example profile:

```yaml
default:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: YOUR_ACCOUNT
      user: YOUR_USERNAME
      password: "YOUR_PASSWORD"
      role: ACCOUNTADMIN
      database: ANALYTICS
      warehouse: TRANSFORMING
      schema: DBT_DEV
      threads: 4
```

Important: do not commit `profiles.yml` to Git. It contains environment-specific credentials.

## 7. Check the profile name

Open your project file:

```text
dbt_project.yml
```

Check the `profile` value:

```yaml
profile: default
```

This must match the top-level key in `profiles.yml`:

```yaml
default:
  target: dev
  outputs:
```

If these names do not match, dbt will not find the correct profile.

## 8. Test the dbt connection

Go to the folder that contains `dbt_project.yml`.

Example:

```powershell
cd "C:\path\to\your\dbt_project"
```

Run:

```powershell
dbt debug
```

If `dbt` is not recognised, use:

```powershell
& $DbtExe debug
```

Expected successful output:

```text
Connection OK
All checks passed
```

## 9. Run the dbt project

Run:

```powershell
dbt run
```

Run tests:

```powershell
dbt test
```

Generate docs:

```powershell
dbt docs generate
```

Serve docs locally:

```powershell
dbt docs serve
```

If `dbt` is not recognised, refresh the current session PATH or replace `dbt` with `& $DbtExe`.

## 10. Refresh dbt PATH if needed

If `dbt` stops working in a new PowerShell terminal, add the user-level Python scripts folder to the current session PATH again.

```powershell
$env:PATH="$env:APPDATA\Python\Python311\Scripts;$env:PATH"
```

Then check:

```powershell
dbt --version
```

If it still does not work, use the full executable path as a fallback:

```powershell
& $DbtExe --version
```

## 11. Git setup

If Git is already installed but PowerShell cannot find it, add Git to the current session path.

```powershell
$env:PATH="$env:LOCALAPPDATA\Programs\Git\cmd;$env:PATH"
```

Verify Git works:

```powershell
git --version
```

If Git still does not work, check where Git is installed and update the path accordingly.

## 12. Recommended .gitignore entries

For a dbt project, use these entries:

```text
target/
dbt_packages/
dbt_modules/
logs/
dbt_internal_packages/
```

Do not commit generated dbt folders.

`dbt_packages/` is created by `dbt deps` and can be recreated.

`target/` and `logs/` are generated when dbt commands run.

## 13. Common issues

### Python 3.14 crashes or behaves unexpectedly

Symptom:

```text
Python or dbt fails unexpectedly after installing with a newer Python version.
```

Fix:

Use Python 3.11 for this setup.

### dbt Fusion does not work in this setup

Symptom:

```text
dbt Fusion commands fail, authentication does not complete, or errors are difficult to trace.
```

Fix:

Use dbt Core for this setup.

Install:

```powershell
.\python.exe -m pip install --user dbt-core dbt-snowflake
```

Then check dbt:

```powershell
dbt --version
dbt debug
```

If `dbt` is not recognised, use:

```powershell
& $DbtExe --version
& $DbtExe debug
```

### externalbrowser authentication fails

Symptom:

```text
Snowflake externalbrowser authentication does not open or complete correctly.
```

Fix:

For this local tutorial setup, if username and password authentication is permitted, remove the `authenticator` field from `profiles.yml` and use the standard username and password fields.

### Snowflake or localhost network checks

Symptom:

```text
dbt cannot connect to Snowflake, authentication does not complete, or an error mentions localhost, DNS, timeout, or port 443.
```

Checks:

```powershell
nslookup <snowflake_account>.snowflakecomputing.com
Test-NetConnection <snowflake_account>.snowflakecomputing.com -Port 443
ping localhost
```

Reasons:

- `nslookup` checks whether the Snowflake account hostname resolves through DNS.
- `Test-NetConnection` checks whether HTTPS traffic to Snowflake on port 443 can connect.
- `ping localhost` checks whether `localhost` resolves locally.

If localhost or proxy behaviour appears to be causing problems, test the current PowerShell session with:

```powershell
$env:NO_PROXY="localhost,127.0.0.1"
$env:HTTP_PROXY=""
$env:HTTPS_PROXY=""
```

Then rerun:

```powershell
dbt debug
```

If `dbt` is not recognised, use:

```powershell
& $DbtExe debug
```

These settings only affect the current PowerShell session.

### Git is not recognised

Symptom:

```text
git : The term 'git' is not recognised
```

Fix:

Add Git to the current PowerShell session path:

```powershell
$env:PATH="$env:LOCALAPPDATA\Programs\Git\cmd;$env:PATH"
git --version
```

### dbt is not recognised

Symptom:

```text
dbt : The term 'dbt' is not recognised
```

Fix the current PowerShell session PATH:

```powershell
$env:PATH="$env:APPDATA\Python\Python311\Scripts;$env:PATH"
dbt --version
```

If that does not work, check whether the dbt executable exists:

```powershell
Test-Path $DbtExe
```

If the path exists, run dbt with the full executable path:

```powershell
& $DbtExe --version
```

You can use `& $DbtExe` in place of `dbt` for any command in this guide.

Example:

```powershell
& $DbtExe debug
& $DbtExe run
```

### dbt cannot find profiles.yml

Symptom:

```text
Could not find profile named ...
```

Fix:

Check that this file exists:

```text
C:\Users\<your_user>\.dbt\profiles.yml
```

Also check that the profile name in `dbt_project.yml` matches the top-level profile name in `profiles.yml`.

## 14. Useful commands

Use these commands from the folder containing `dbt_project.yml`.

```powershell
dbt debug
dbt run
dbt test
dbt docs generate
dbt docs serve
```

If `dbt` is not recognised, refresh the current session PATH or replace `dbt` with `& $DbtExe`.

## 15. Final check

Run:

```powershell
dbt debug
dbt run
```

If `dbt` is not recognised, run:

```powershell
& $DbtExe debug
& $DbtExe run
```

If both commands complete successfully, the Windows dbt setup is working.

## Appendix A. Sanitised troubleshooting command log

This appendix records the main commands that were used while troubleshooting the Windows setup.

It is intentionally sanitised for a public GitHub repository:

- Local usernames are replaced with `$env:USERPROFILE` or `<your_user>`.
- The Snowflake account hostname is replaced with `<snowflake_account>.snowflakecomputing.com`.
- Personal access paths and environment-specific values are replaced with placeholders.
- Typos, repeated `clear` commands, and commands that produced no useful setup change are not included as required setup steps.

The main setup guide above should be followed first. This appendix is useful for understanding the troubleshooting path.

### 1. Move to the development folder

```powershell
cd
ls
cd Documents
ls
cd dev0
ls
cd .\dev0\
```

Reason:

These commands were used to navigate to the local development folder and check the current directory contents.

### 2. Clone the learning repository

```powershell
gh repo clone <github_user>/<repo_name>
git clone https://github.com/<github_user>/<repo_name>
```

Reason:

The GitHub CLI clone was attempted first. The standard `git clone` command was then used as a fallback.

### 3. Install and verify Git

```powershell
winget install --id Git.Git -e --source winget
git
where git
git --version
```

Reason:

These commands checked whether Git was installed, whether PowerShell could find it, and whether the installed Git executable worked.

If Git was installed but not on PATH, this session-only PATH update was used:

```powershell
$env:PATH="$env:LOCALAPPDATA\Programs\Git\cmd;$env:PATH"
git --version
```

Reason:

This made Git available in the current PowerShell session without requiring administrator access.

### 4. Attempt the dbt Fusion tutorial setup

```powershell
dbtf debug
dbt debug
winget install dbt-labs.dbt-fusion
irm https://public.cdn.getdbt.com/fs/install/install.ps1 | iex
dbtf --version
dbt --version
Start-Process powershell
dbt --version
dbtf --version
dbt init
```

Reason:

These commands followed the dbt Fundamentals VS Code tutorial path, which uses the dbt Fusion engine and the dbt VS Code workflow.

Outcome:

Fusion was not reliable enough in this Windows environment, so the setup was changed to dbt Core.

### 5. Check Python availability

```powershell
python
python --version
winget install Python.Python.3.11
winget install Python.Python.3.11 --scope user
python --version
```

Reason:

These commands checked whether Python was already available and attempted a user-scope Python install.

Outcome:

The final guide uses the Python 3.11 embeddable package instead, because it gives a more predictable no-admin setup.

### 6. Check the manually extracted Python folder

```powershell
cd "$env:USERPROFILE\tools\python"
.\python.exe --version
$env:PATH="$env:USERPROFILE\tools\python;$env:PATH"
python --version
```

Reason:

These commands checked whether the manually extracted Python executable worked and tested adding it to the current PowerShell session PATH.

### 7. Install pip manually

```powershell
python -m ensurepip
Invoke-WebRequest -Uri "https://bootstrap.pypa.io/get-pip.py" -OutFile "get-pip.py"
python get-pip.py
python -m pip --version
```

Reason:

The embeddable Python distribution does not behave like a standard Python install. These commands attempted to install and verify pip.

The final guide uses this cleaner version from inside the Python folder:

```powershell
cd $PythonHome
Invoke-WebRequest -Uri "https://bootstrap.pypa.io/get-pip.py" -OutFile "get-pip.py"
.\python.exe get-pip.py
.\python.exe -m pip --version
```

### 8. Try `dbt init` and debug the initial profile

```powershell
dbt init
dbt debug
```

Reason:

`dbt init` was used to initialise the dbt project and create or guide the creation of `profiles.yml`.

`dbt debug` was used to test whether the project and Snowflake profile were configured correctly.

### 9. Check Snowflake network access

```powershell
nslookup <snowflake_account>.snowflakecomputing.com
Test-NetConnection <snowflake_account>.snowflakecomputing.com -Port 443
```

Reason:

These commands checked whether the Snowflake hostname resolved through DNS and whether HTTPS traffic to Snowflake on port 443 was reachable.

### 10. Check localhost and proxy behaviour

```powershell
ping localhost
$env:NO_PROXY="localhost,127.0.0.1"
$env:HTTP_PROXY=""
$env:HTTPS_PROXY=""
dbt debug
```

Reason:

Fusion and browser-based authentication flows can depend on localhost redirect behaviour.

These commands checked whether localhost worked and temporarily bypassed proxy settings for localhost in the current PowerShell session.

Outcome:

This did not produce a reliable Fusion setup, so dbt Core was used instead.

### 11. Install dbt Core and the Snowflake adapter

```powershell
python -m pip install --user dbt-core dbt-snowflake
python -m dbt debug
```

Reason:

This attempted to install dbt Core and the Snowflake adapter into the user profile, then test dbt through Python.

When multiple Python versions were present, the explicit Python path was used:

```powershell
C:\Users\<your_user>\tools\python\python.exe -m pip install --user dbt-core dbt-snowflake
C:\Users\<your_user>\tools\python\python.exe -m dbt debug
```

Reason:

Using the explicit Python path removed ambiguity about which Python installation was being used.

### 12. Investigate which dbt executable was being used

```powershell
python -m site --user-base
where dbt
dir "$env:APPDATA\Python" -Recurse -Filter dbt.exe
where.exe dbt
```

Reason:

These commands located the user-level Python scripts folder and checked which `dbt.exe` PowerShell was resolving.

This helped identify PATH conflicts between dbt Fusion and dbt Core.

### 13. Remove Fusion path conflicts from the current session

```powershell
Remove-Item Function:\dbt -ErrorAction SilentlyContinue
$env:PATH = ($env:PATH -split ';' | Where-Object {$_ -notlike "*\.local\bin*"}) -join ';'
```

Reason:

These commands removed a PowerShell function alias and removed the Fusion `.local\bin` path from the current session PATH.

This helped ensure the dbt Core executable was used instead of the Fusion executable.

### 14. Use the Python 3.11 dbt Core executable directly

```powershell
cd "$env:USERPROFILE\tools\python311"
.\python.exe get-pip.py
Invoke-WebRequest -Uri "https://bootstrap.pypa.io/get-pip.py" -OutFile "get-pip.py"
.\python.exe get-pip.py
.\python.exe -m pip --version
.\python.exe -m pip install --user dbt-core dbt-snowflake
$env:PATH="$env:APPDATA\Python\Python311\Scripts;$env:PATH"
where dbt
Test-Path "$env:APPDATA\Python\Python311\Scripts\dbt.exe"
& "$env:APPDATA\Python\Python311\Scripts\dbt.exe" --version
& "$env:APPDATA\Python\Python311\Scripts\dbt.exe" debug
```

Reason:

This was the working path.

It installed dbt Core using Python 3.11, added the Python 3.11 scripts folder to the current session PATH, verified `dbt.exe` existed, checked the version, and ran `dbt debug`.

### 15. Add Git to PATH and rerun dbt checks

```powershell
$env:PATH="$env:LOCALAPPDATA\Programs\Git\cmd;$env:PATH"
git --version
dbt debug
dbt run
```

Reason:

Git needed to be available for dbt package and project workflows.

After Git was available, `dbt debug` and `dbt run` confirmed that the dbt project could connect and execute.

### 16. Review Git changes

```powershell
git diff
git diff --staged
git diff HEAD
```

Reason:

These commands reviewed unstaged changes, staged changes, and the full difference from the current `HEAD`.

If output needs to be saved for review, use temporary files and delete them before committing:

```powershell
git diff > git_diff.txt
git diff --staged > staged_diff.txt
git diff HEAD > full_diff.txt
```

Do not commit these temporary diff files unless they are intentionally part of the project documentation.


