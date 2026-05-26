# Generate and Preview dbt Documentation from the CLI

## Context

While working through the dbt Fundamentals VS Code course, the documentation section noted that documentation generation in the dbt VS Code Extension was not available at the time.

As of 26/05/2026, the course page stated:

```text
Currently generating documentation in the dbt VS Code Extension is not available, but it is something the dbt Labs product is avidly working on. We will update this page once documentation generation is released.
```

The dbt documentation can still be generated and viewed using the dbt Core CLI.

Run these commands from the folder that contains `dbt_project.yml`.

## 1. Generate the documentation

```powershell
dbt docs generate
```

This creates the documentation files in the `target/` folder.

## 2. Serve the documentation locally

```powershell
dbt docs serve
```

This starts a local documentation website, usually at:

```text
http://localhost:8080
```

Open the local URL in a browser to view models, sources, columns, descriptions, lineage, and data tests.

## If dbt is not recognised

Use the full dbt executable path instead:

```powershell
& "C:\Users\<your_user>\AppData\Roaming\Python\Python311\Scripts\dbt.exe" docs generate
& "C:\Users\<your_user>\AppData\Roaming\Python\Python311\Scripts\dbt.exe" docs serve
```

## Git note

Do not commit the generated `target/` folder.

Commit the documentation source files, such as YAML and Markdown files, but keep generated files ignored.

## Useful mental model

```text
dbt docs generate = build the documentation files
dbt docs serve = preview the documentation site locally
```