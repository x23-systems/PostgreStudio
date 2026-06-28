# PostgreStudio

**PostgreStudio** is a single-file, browser-based PostgreSQL playground powered by [PGlite](https://pglite.dev/).

It gives you a lightweight, SSMS-like interface to design schemas, run SQL, inspect tables, seed fake data, edit rows, and export SQL dumps — directly from your browser.

No local PostgreSQL server.
No Docker.
No admin rights.
No installation required beyond opening an HTML file.

> PostgreStudio is designed for local prototyping, SQL experimentation, schema modeling, lightweight testing, and environments where installing PostgreSQL locally is not possible.

---

## Why PostgreStudio?

Sometimes you need a PostgreSQL-like working environment, but you cannot install anything on the machine you are using.

Typical cases:

* locked-down corporate laptops;
* no local administrator rights;
* no Docker Desktop;
* no PostgreSQL service;
* no access to a shared development database;
* need to quickly prototype a schema or test SQL behavior;
* need to generate a SQL dump after modeling a small database locally.

PostgreStudio uses PGlite to run PostgreSQL in WebAssembly directly inside the browser, with local persistence through browser storage.

---

## Features

### Browser-based PostgreSQL playground

* Runs PostgreSQL in the browser using PGlite.
* Persists data locally in the browser.
* Works as a single HTML file.
* No backend server required.
* No PostgreSQL installation required.

### SQL editor

* Multi-tab SQL editor.
* Syntax highlighting.
* Search and replace.
* Multi-cursor editing.
* Run selected SQL or full script.
* `Ctrl + Enter` execution shortcut.
* Light and dark themes.

### Schema explorer

Explore database objects from the left panel:

* schemas;
* tables;
* views;
* materialized views;
* sequences;
* functions;
* procedures;
* enum types;
* domains;
* indexes;
* triggers;
* constraints;
* table columns.

Tables display their row count directly in the explorer.

### SQL generation

Context menus generate ready-to-edit SQL scripts for:

* `SELECT *`;
* `INSERT`;
* `UPDATE`;
* table creation;
* view creation;
* materialized view creation;
* sequence creation;
* function creation;
* stored procedure creation;
* enum type creation;
* domain creation;
* index creation;
* trigger creation;
* schema creation.

Generated scripts open in dedicated editor tabs before execution.

### Fake data generation

PostgreStudio can generate deterministic fake data for a selected table.

It uses table metadata to infer suitable values from:

* column names;
* PostgreSQL data types;
* nullability;
* defaults;
* identity/generated columns;
* primary keys;
* foreign keys;
* enums;
* simple check constraints.

The generated seed data is inserted as SQL in a new tab and is not executed automatically.

### Result grid

Query results are displayed in a tabular grid.

For simple `SELECT * FROM schema.table` queries, PostgreStudio can generate row-level SQL actions:

* edit row;
* delete row.

The application does not modify rows directly from the grid. Instead, it opens a new SQL tab containing the generated `UPDATE` or `DELETE` statement, so the user can review and execute it manually.

### SQL dumps

PostgreStudio can export SQL dumps:

* full database structure;
* full database data;
* structure and data;
* schema-level dumps;
* table-level dumps.

The generated SQL dump opens in a new editor tab and can also be downloaded as a `.sql` file.

### Object deletion with impact analysis

Before dropping an object, PostgreStudio displays a confirmation dialog with impact information.

Supported deletion actions include:

* drop table;
* drop schema;
* drop view;
* drop materialized view;
* drop sequence;
* drop function;
* drop procedure;
* drop enum type;
* drop domain.

For tables and schemas, PostgreStudio attempts to identify dependent objects such as:

* incoming foreign keys;
* dependent views;
* related constraints;
* related indexes;
* related triggers;
* owned sequences.

The generated `DROP` command is opened in a SQL tab and executed only after explicit confirmation.

### Transaction safety

PGlite runs as a single embedded database instance in the browser. PostgreStudio therefore provides a virtual tab-level transaction guard to avoid accidental transaction leakage between editor tabs.

PostgreStudio includes:

* visible transaction state per tab;
* automatic rollback when another tab tries to use the database while one tab owns an open transaction;
* automatic rollback on execution errors;
* optional automatic atomic execution for scripts;
* manual rollback button for open transactions.

This does not turn PGlite into a multi-connection PostgreSQL server, but it prevents the most dangerous usability issue: unknowingly executing a query in another tab under a transaction started elsewhere.

---

## What PostgreStudio is not

PostgreStudio is not a replacement for:

* pgAdmin;
* DBeaver;
* DataGrip;
* SQL Server Management Studio;
* a real PostgreSQL server;
* a production database administration tool.

It is a lightweight local playground.

Use it to:

* prototype schemas;
* experiment with SQL;
* generate local seed data;
* test migrations manually;
* build small browser-local databases;
* export SQL scripts.

Do not use it to manage production data.

---

## Getting started

### Option 1 — Open the HTML file directly

Download the latest `postgrestudio.html` file from the repository and open it in a modern browser.

Recommended browsers:

* Chrome;
* Chromium;
* Microsoft Edge.

Depending on your browser and corporate security policies, loading ES modules and WebAssembly from a local `file://` page may be blocked.

### Option 2 — Serve locally over HTTP

If direct opening does not work, serve the folder locally.

With Python:

```bash
python -m http.server 8000
```

Then open:

```text
http://127.0.0.1:8000/postgrestudio.html
```

This does not require a backend application. It only serves the static HTML file locally.

---

## Usage

### Create a schema

Right-click the `Schemas` node and choose:

```text
Create schema…
```

PostgreStudio will generate a SQL tab such as:

```sql
CREATE SCHEMA IF NOT EXISTS "demo";

SET search_path TO "demo", public;
```

Review and execute the script.

### Create a table

Right-click a schema and choose:

```text
Add table…
```

Example generated script:

```sql
CREATE TABLE "demo"."customer"
(
    "customer_id" BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    "full_name" TEXT NOT NULL,
    "email" TEXT,
    "created_at" TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Query a table

Double-click a table in the explorer.

PostgreStudio opens and executes:

```sql
SELECT *
FROM "demo"."customer"
LIMIT 200;
```

### Generate fake data

Right-click a table and choose:

```text
Generate fake data…
```

Choose the number of rows and the seed. PostgreStudio opens a new SQL tab with generated `INSERT` statements.

Review the script, then execute it.

### Edit or delete a row

Open a table with `SELECT *`, then use the result grid row menu.

PostgreStudio can generate:

```sql
UPDATE "demo"."customer"
SET
  "full_name" = 'Alice Martin',
  "email" = 'alice.martin@example.test'
WHERE
  "customer_id" IS NOT DISTINCT FROM 1
RETURNING *;
```

or:

```sql
DELETE FROM "demo"."customer"
WHERE
  "customer_id" IS NOT DISTINCT FROM 1
RETURNING *;
```

### Export SQL dump

Use the export menu to generate:

* structure only;
* data only;
* structure and data.

The dump opens in a dedicated SQL tab and can be downloaded as a `.sql` file.

---

## Persistence

PostgreStudio stores the PGlite database locally in the browser.

Depending on the configuration, data may be stored using browser storage such as IndexedDB.

This means:

* data remains local to the browser profile;
* data is not sent to a remote PostgreStudio server;
* clearing browser site data may delete the local database;
* using another browser profile will create another local database;
* private browsing modes may not persist data reliably.

---

## Security and privacy

PostgreStudio is intended to run locally in the browser.

However, the default single-file version may load JavaScript and WebAssembly dependencies from public CDNs.

If you use PostgreStudio in a restricted, corporate, confidential, or offline environment, review the source code and consider vendoring all dependencies locally before use.

Recommended precautions:

* do not load production or confidential data unless you fully control the runtime;
* prefer synthetic data for testing;
* check your organization’s security policy before using it on a corporate machine;
* do not attempt to bypass enterprise endpoint controls;
* do not treat browser-local storage as a secure database vault.

---

## Limitations

### PGlite is not a PostgreSQL server

PGlite runs PostgreSQL embedded in the JavaScript environment. It is not a standalone PostgreSQL server listening on `localhost:5432`.

That means:

* external tools such as Npgsql, DBeaver, pgAdmin or `psql` cannot connect to this page as if it were a normal PostgreSQL server;
* the browser instance is not a multi-user PostgreSQL server;
* transaction behavior is constrained by the embedded execution model.

### Single embedded connection model

PostgreStudio provides tab-level guards to prevent accidental transaction leakage, but it cannot create truly independent PostgreSQL backend sessions per editor tab inside the same browser-local PGlite instance.

For realistic multi-connection, concurrency, locking, pooling or driver-level tests, use a real PostgreSQL server.

### SQL compatibility

PGlite is a real PostgreSQL build compiled to WebAssembly, but some server-level features may be unavailable or behave differently from a full PostgreSQL installation.

Always validate production migrations and critical SQL behavior against the target PostgreSQL version.

---

## Project status

PostgreStudio is an experimental developer tool.

Current focus:

* single-file usability;
* schema prototyping;
* local SQL experimentation;
* seed data generation;
* SQL dump generation;
* safe browser-local workflows.

Planned improvements may include:

* packaged offline release with vendored dependencies;
* import SQL dump workflow;
* better object dependency graph;
* richer table designer;
* richer data grid editing;
* ERD generation;
* project/workspace export;
* automated regression test suite;
* GitHub Pages demo.

---

## Repository structure

Suggested initial structure:

```text
.
├── README.md
├── LICENSE
├── postgrestudio.html
├── docs/
│   └── screenshots/
└── examples/
    └── demo.sql
```

The project currently works as a single HTML file. Future versions may optionally introduce a build pipeline, but keeping a no-build distribution is a core goal.

---

## Development

For local development, clone the repository and serve the directory:

```bash
git clone https://github.com/<your-org>/PostgreStudio.git
cd PostgreStudio
python -m http.server 8000
```

Then open:

```text
http://127.0.0.1:8000/postgrestudio.html
```

---

## Contributing

Contributions are welcome.

Useful contribution areas:

* browser compatibility testing;
* PGlite compatibility testing;
* UI improvements;
* SQL generation improvements;
* dependency analysis;
* fake data generation rules;
* accessibility;
* documentation;
* packaging and offline distribution;
* automated tests.

Before submitting a large change, please open an issue to discuss the intended direction.

---

## Naming

The project is named **PostgreStudio**.

It is not affiliated with PostgreSQL, the PostgreSQL Global Development Group, ElectricSQL, PGlite, Microsoft, Snowflake, pgAdmin, DBeaver or JetBrains.

---

## License

Choose a license before publishing.

Recommended options:

* **MIT** for maximum reuse;
* **Apache-2.0** if you want an explicit patent grant;
* **GPL-3.0** if you want derivative works to remain open source.

Example:

```text
MIT License
```

---

## Credits

PostgreStudio is built on top of:

* [PGlite](https://pglite.dev/) — PostgreSQL compiled to WebAssembly.
* [CodeMirror](https://codemirror.net/) — browser-based code editor.
* PostgreSQL — the database system whose SQL dialect and tooling inspired this project.

---

## Disclaimer

PostgreStudio is provided as-is, without warranty.

It is intended for local development, experimentation and prototyping. Do not use it as a production database administration tool.
