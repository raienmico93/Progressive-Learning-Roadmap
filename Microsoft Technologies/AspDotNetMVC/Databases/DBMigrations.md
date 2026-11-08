# EF Core Database Migrations: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **Entity Framework Core** (EF Core) 8–11 and the **Migrations** feature for managing database schema changes. Examples use both the **.NET CLI** (`dotnet ef`) and the **Package Manager Console** (PMC) in Visual Studio. Where behaviour differs across versions (e.g., `--add` in EF Core 11, migration locking in EF Core 9), it is flagged. Migrations are the recommended mechanism for evolving database schemas in production; `EnsureCreated` is for testing and prototyping only. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Migrations are a feature of EF Core that provides a way to incrementally update a database schema to keep it in sync with the application’s data model while preserving existing data. |
| **Technical Definition** | Migrations work by comparing the current model against a **snapshot** of the previous model. EF Core tools generate **migration source files** containing `Up` (apply) and `Down` (revert) methods. These files are checked into source control and applied to databases via commands (`Update-Database`, `dotnet ef database update`), SQL scripts, or migration bundles. EF Core records applied migrations in a **history table** (`__EFMigrationsHistory`), allowing it to know which migrations have been applied and which have not. |
| **Beginner-Friendly Explanation** | Migrations are **version control for your database schema**. When you add a property to a C# entity, EF Core generates a migration file that describes how to add the corresponding column to the database. You review and commit the file, then apply it to your database. Everyone—other developers, testers, and production—applies the same migrations in order, so the database schema always matches the code. |

### Key Characteristics

- **Versioned** – Each migration is a timestamped, source-controlled file representing a discrete schema change.
- **Reversible** – Each migration has an `Up` method (apply) and a `Down` method (revert).
- **History-tracked** – Applied migrations are recorded in `__EFMigrationsHistory`.
- **Model-snapshot-driven** – EF Core compares the current model with a snapshot to generate the next migration.
- **Deployment-flexible** – Migrations can be applied via CLI tools, SQL scripts, or self-contained bundles.
- **Data-preserving** – Designed to evolve schemas without dropping the entire database.

### Prerequisites

- .NET SDK (6.0+ recommended), EF Core project with a `DbContext`.
- EF Core command-line tools installed (`dotnet tool install --global dotnet-ef`).
- A database provider package (SQL Server, PostgreSQL, SQLite, etc.).
- Basic understanding of **EF Core** (`DbContext`, `DbSet`, entities).

### Related Programming Areas

- **EF Core** – The ORM that generates and applies migrations.
- **Database schema evolution** – The practice of changing database structures over time.
- **CI/CD pipelines** – Automated deployment of migrations.
- **Team development** – Coordinating migrations across branches and developers.
- **Production deployment** – Safe, reviewed application of schema changes.

### Core Concepts / Features

1. **Migrations** – The overall feature, architecture, and history table.
2. **`Add-Migration`** – The PMC command (and its CLI equivalent `dotnet ef migrations add`).
3. **`Update-Database`** – The PMC command (and its CLI equivalent `dotnet ef database update`).
4. **Migration best practices** – Team coordination, production deployment, data safety, and versioning.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. Migrations (The Feature)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Migrations are a structured, versioned way to apply incremental changes to a database schema as the application’s data model evolves. |
| **Technical Definition** | The migrations feature in EF Core works by: (1) comparing the current model against a snapshot of the previous model to detect differences; (2) generating source files containing the operations needed to update the schema (`Up`) and revert them (`Down`); (3) recording applied migrations in the `__EFMigrationsHistory` table; and (4) providing tools to apply pending migrations to a database. The feature preserves existing data while evolving the schema. |
| **Beginner-Friendly Explanation** | Migrations are the **“recipe book” for changing your database**. Each recipe (migration) describes one change: “add a column,” “create a table,” “drop an index.” The database keeps a list of which recipes it has already followed (the history table). When you deploy, you run the new recipes—and the database catches up without losing its existing data. |

### Purposes

- **To** keep the database schema **in sync with the application’s data model** as it evolves.
- **To** provide a **versioned, auditable history** of schema changes in source control.
- **To** enable **safe, incremental schema evolution** without dropping and recreating the database.
- **To** support **team collaboration** through a shared, ordered sequence of migrations.
- **To** enable **automated deployment** through SQL scripts, bundles, or runtime migration.

### Syntax Rules and Structure

#### General Architecture

```text
Application Model (C# entities)
   │
   ▼
EF Core Tools (dotnet ef / PMC)
   │  1. Compare model to snapshot
   │  2. Generate migration files
   ▼
Migrations/
   ├── 20260101120000_InitialCreate.cs
   ├── 20260101120000_InitialCreate.Designer.cs
   └── AppDbContextModelSnapshot.cs
   │
   ▼
Database
   ├── __EFMigrationsHistory (applied migrations)
   └── Application tables
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| **Model** | C# entity classes and `DbContext` configuration. |
| **Snapshot** (`AppDbContextModelSnapshot.cs`) | A representation of the model at the time of the last migration; used to detect changes. |
| **Migration files** | `Up`/`Down` methods describing the schema change. |
| **History table** (`__EFMigrationsHistory`) | Records which migrations have been applied. |

#### General Syntax (Migration File Structure)

```csharp
public partial class AddCreatedAt : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<DateTime>(
            name: "CreatedAt",
            table: "Posts",
            type: "datetime2",
            nullable: false,
            defaultValue: new DateTime(2026, 1, 1));
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "CreatedAt",
            table: "Posts");
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Up` | Operations to apply the migration. |
| `Down` | Operations to revert the migration. |
| `MigrationBuilder` | Fluent API for schema operations (`AddColumn`, `CreateTable`, `DropColumn`, etc.). |

#### Syntax Rules

- **Migration files are checked into source control** – they are part of the codebase, like any other source file.
- **Each migration is timestamped** – the filename includes a timestamp to ensure chronological ordering.
- **The snapshot file is updated** with each new migration – it represents the model at the latest migration.
- **Migrations are applied in order** – EF Core uses the history table to determine which have been applied.

#### Constraints and Limitations

- **Migrations are provider-specific** – migrations generated for SQL Server cannot be applied to PostgreSQL without regeneration.
- **Data loss is possible** – dropping columns, tables, or changing types can lose data. Always review the generated migration.
- **Renames may be generated as drop+add** – EF Core cannot always detect a property rename; manual intervention is often required.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Initial Migration Workflow

```bash
# 1. Install EF Core tools (if not already installed)
dotnet tool install --global dotnet-ef

# 2. Add the initial migration
dotnet ef migrations add InitialCreate

# 3. Apply the migration to the database
dotnet ef database update
```

**Step-by-step explanation:**

1. The EF Core tools are installed globally.
2. `migrations add InitialCreate` compares the current model to the (empty) snapshot and generates three files in the `Migrations` folder: the migration class, its designer metadata, and the model snapshot.
3. `database update` applies the pending migration to the database and records it in `__EFMigrationsHistory`.

**Expected Output:** A `Migrations` folder containing `20260101120000_InitialCreate.cs`, its `.Designer.cs`, and `AppDbContextModelSnapshot.cs`. The database gains the tables defined in the model, and `__EFMigrationsHistory` contains one row for `InitialCreate`.

**Why this output?** The migration file describes the schema creation; applying it executes the `Up` method and records the migration in the history table.

#### Example 2: Subsequent Migration for a New Property

```bash
# 1. Add a new property to an entity (e.g., CreatedAt)
# 2. Generate a migration for the change
dotnet ef migrations add AddCreatedAt

# 3. Review the generated migration file
# 4. Apply the migration
dotnet ef database update
```

**Expected Output:** A new migration file `20260101123000_AddCreatedAt.cs` containing an `AddColumn` operation; the database gains the `CreatedAt` column; the history table gains a second row.

**Why this output?** EF Core compares the current model (with `CreatedAt`) to the snapshot (without it) and generates the difference.

### Real-World Cases

- **Feature development:** Adding a new entity or property as part of a feature branch.
- **Production deployments:** Applying reviewed SQL scripts or migration bundles to production databases.
- **Team development:** Coordinating migrations across multiple developers and branches.
- **Legacy migration:** Introducing migrations to an existing database using a baseline.

### References

- Migrations Overview – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/
- Managing Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying

---

## 2. `Add-Migration`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Add-Migration` is the Package Manager Console (PMC) command in Visual Studio that scaffolds a new migration based on changes to the EF Core model. |
| **Technical Definition** | `Add-Migration <Name>` (PMC) or `dotnet ef migrations add <Name>` (.NET CLI) compares the current model against the snapshot file (`AppDbContextModelSnapshot.cs`) and generates three files: `{timestamp}_{Name}.cs` (the migration with `Up`/`Down` methods), `{timestamp}_{Name}.Designer.cs` (metadata), and an updated `AppDbContextModelSnapshot.cs`. The migration name is used like a commit message and should describe the change. |
| **Beginner-Friendly Explanation** | `Add-Migration` is the **“record this change”** command. After you modify your C# entities, you run `Add-Migration` with a descriptive name, and EF Core writes a new migration file that describes how to update the database to match. You then review and commit the file. |

### Purposes

- **To** generate a migration file that describes the difference between the current model and the last migration.
- **To** provide a **reviewable, version-controlled artifact** for each schema change.
- **To** update the model snapshot so subsequent migrations are computed correctly.
- **To** allow developers to **inspect and customise** the generated operations before applying them.

### Syntax Rules and Structure

#### PMC (Visual Studio)

```powershell
Add-Migration <Name>
Add-Migration <Name> -OutputDir <Directory>
Add-Migration <Name> -Namespace <Namespace>
```

#### .NET CLI

```bash
dotnet ef migrations add <Name>
dotnet ef migrations add <Name> --output-dir <Directory>
dotnet ef migrations add <Name> --namespace <Namespace>
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `<Name>` | The migration name (used like a commit message; e.g., `AddCreatedAt`). |
| `-OutputDir` / `--output-dir` | Optional directory for migration files (default: `Migrations`). |
| `-Namespace` / `--namespace` | Optional namespace for generated classes. |
| `--context` | Specifies the `DbContext` if multiple are registered. |
| `--project` / `--startup-project` | Specifies projects if they differ. |

#### Syntax Rules

- **Migration names should be descriptive** – e.g., `AddCreatedAt`, `CreateOrdersTable`, `RenameCustomerName`.
- **The snapshot is updated** with each new migration – do not edit the snapshot manually.
- **Migration files are timestamped** – the timestamp ensures chronological ordering.
- **The migration is not applied** – `Add-Migration` only scaffolds the file; `Update-Database` applies it.

#### Constraints and Limitations

- **Parallel migrations in team environments** can cause conflicts – coordinate and rebase.
- **The generated migration may not match your intent** – always review it, especially for renames.
- **`Add-Migration` requires the model to compile** – fix build errors before generating a migration.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Adding a Migration (PMC)

```powershell
# 1. Modify the model (e.g., add a CreatedAt property to Post)
# 2. Open Package Manager Console
# 3. Run:
Add-Migration AddCreatedAt
```

**Step-by-step explanation:**

1. A `CreatedAt` property is added to the `Post` entity.
2. The PMC command compares the current model to the snapshot.
3. Three files are generated in the `Migrations` folder.

**Expected Output:** A migration file `20260101123000_AddCreatedAt.cs` containing an `AddColumn` operation; the snapshot is updated to include `CreatedAt`.

**Why this output?** EF Core detects the new property and generates the corresponding schema operation.

#### Example 2: Specifying a Custom Output Directory

```bash
dotnet ef migrations add AddCreatedAt --output-dir Data/Migrations
```

**Expected Output:** Migration files are generated in `Data/Migrations` instead of the default `Migrations` folder.

**Why this output?** The `--output-dir` option overrides the default location.

#### Example 3: Creating and Applying in One Step (EF Core 11+)

```bash
dotnet ef database update AddCreatedAt --add
```

**Expected Output:** A migration named `AddCreatedAt` is scaffolded, compiled with Roslyn, and applied to the database in a single command.

**Why this output?** The `--add` option (EF Core 11+) combines migration generation and application, useful for containerized/Aspire scenarios.

### Real-World Cases

- **Feature branches:** Generate a migration for each schema change in the branch.
- **Custom migration directories:** Organise migrations in a dedicated project or folder.
- **Multiple DbContexts:** Use `--context` to specify which context to scaffold for.

### References

- Managing Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- EF Core tools reference (PMC) – https://learn.microsoft.com/en-us/ef/core/cli/powershell
- EF Core tools reference (.NET CLI) – https://learn.microsoft.com/en-us/ef/core/cli/dotnet

---

## 3. `Update-Database`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Update-Database` is the PMC command that applies pending migrations to the database, bringing its schema up to date with the model. |
| **Technical Definition** | `Update-Database` (PMC) or `dotnet ef database update` (.NET CLI) applies all pending migrations to the database, or migrates to a specified target migration. It executes the `Up` method of each pending migration in order, records them in `__EFMigrationsHistory`, and optionally runs `Down` methods when reverting. For local development, this is usually the simplest option; for production, SQL scripts or migration bundles are recommended. |
| **Beginner-Friendly Explanation** | `Update-Database` is the **“apply the changes”** command. After you generate a migration with `Add-Migration`, you run `Update-Database` to actually make the database match your model. It runs the new migration files and records that they have been applied. |

### Purposes

- **To** apply pending migrations to a database, bringing it up to date with the model.
- **To** migrate a database to a **specific target migration** (forward or backward).
- **To** revert all migrations to a blank state (`Update-Database -TargetMigration:0`).
- **To** provide a simple command for **local development** and testing.

### Syntax Rules and Structure

#### PMC (Visual Studio)

```powershell
Update-Database
Update-Database -TargetMigration <Name>
Update-Database -TargetMigration 0
```

#### .NET CLI

```bash
dotnet ef database update
dotnet ef database update <Name>
dotnet ef database update 0
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| (none) | Apply all pending migrations. |
| `<Name>` / `-TargetMigration <Name>` | Migrate to a specific migration (forward or backward). |
| `0` | Revert all migrations (empty database). |
| `--connection` | Specify a connection string (overrides the default). |

#### Syntax Rules

- **`Update-Database` applies migrations in order** – it uses the history table to determine which are pending.
- **Targeting a previous migration** runs the `Down` method of migrations that come after it.
- **Targeting `0`** reverts all migrations.
- **The database must be reachable** – connection strings must be valid.

#### Constraints and Limitations

- **Not recommended for production** – startup migration can cause race conditions in multi-instance deployments and grants schema-change permissions to the app. Use SQL scripts or bundles instead.
- **Data loss is possible** – reverting migrations runs `Down` methods, which may drop data.
- **Requires the SDK and source code** – cannot be run on a server without the .NET SDK and project files.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Applying All Pending Migrations

```powershell
# Apply all pending migrations
Update-Database
```

**Expected Output:** All pending migrations are applied in order; the history table is updated with each.

**Why this output?** `Update-Database` queries the history table, determines pending migrations, and executes their `Up` methods.

#### Example 2: Migrating to a Specific Migration

```bash
# Revert to a specific migration
dotnet ef database update AddCreatedAt
```

**Expected Output:** The database is migrated to the state defined by `AddCreatedAt`; any later migrations are reverted via their `Down` methods.

**Why this output?** The target migration acts as the endpoint; EF Core applies or reverts as needed to reach that state.

#### Example 3: Reverting All Migrations

```powershell
# Revert all migrations
Update-Database -TargetMigration 0
```

**Expected Output:** All migrations are reverted; the database is returned to an empty state (or the state before the first migration).

**Why this output?** Targeting `0` runs the `Down` method of every applied migration.

### Real-World Cases

- **Local development:** Apply migrations after pulling changes from source control.
- **Testing:** Apply migrations to a test database before running integration tests.
- **Rollback in development:** Revert a problematic migration during development.

### References

- Applying Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
- Update-Database (PMC) – https://learn.microsoft.com/en-us/ef/core/cli/powershell#update-database

---

## 4. Migration Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Migration best practices are guidelines for managing schema changes safely, collaboratively, and reliably across development, testing, and production environments. |
| **Technical Definition** | Best practices encompass **team coordination** (avoiding parallel migration conflicts), **production deployment** (using SQL scripts or migration bundles instead of runtime migration), **data safety** (reviewing migrations for potential data loss), **versioning** (checking migrations into source control), **idempotent scripts** (for safe re-execution), and **separating deployment identity** from runtime identity. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for migrations—they help you avoid broken deployments, data loss, merge conflicts, and production outages. |

### Purposes

- **To** prevent **data loss** from unreviewed migrations.
- **To** avoid **merge conflicts** and invalid migration sequences in team environments.
- **To** ensure **safe, repeatable deployments** to production.
- **To** enable **auditability** through source-controlled migration files.
- **To** support **CI/CD automation** through migration bundles and idempotent scripts.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Source control** | Check migration files into source control (Git). | Ignore or exclude the `Migrations` folder. |
| **Review** | Always review generated migrations before applying, especially for renames. | Apply migrations blindly—EF Core may generate drop+add for renames. |
| **Team coordination** | Coordinate in advance; avoid concurrent migrations on multiple branches. | Resolve parallel migrations by sorting or renaming files. |
| **Local database** | Ensure your local database is up to date before creating a new migration. | Create migrations against an outdated local schema. |
| **Production deployment** | Use **SQL scripts** (reviewed) or **migration bundles** for production. | Use `Update-Database` or `Database.Migrate()` at startup in production. |
| **Idempotent scripts** | Use `--idempotent` for scripts that may run against databases at different migration stages. | Run non-idempotent scripts against multiple environments. |
| **Data loss** | Review migrations for potential data loss (dropped columns, type changes). | Ignore warnings about data loss. |
| **Deployment identity** | Use a separate identity with schema-change permissions for deployment. | Grant schema-change permissions to the application’s runtime identity. |
| **Locking** | Rely on EF Core 9+ migration locking, or implement a mutex for older versions. | Allow multiple instances to apply migrations simultaneously. |
| **Baseline** | Baseline existing databases instead of applying all historical migrations. | Replay all migrations against an existing production database. |

### Annotated Code Example (Best-Practice Production Deployment with Idempotent Script)

```bash
# 1. Generate an idempotent SQL script in CI
dotnet ef migrations script --idempotent --output artifacts/migrate.sql

# 2. Review the script (DBA or automated check)
# 3. Apply the script as part of the deployment pipeline
# sqlcmd -S <server> -d <database> -i artifacts/migrate.sql
```

**Why this follows best practices:**

- **Idempotent script** – can be run against any database, regardless of which migrations have been applied.
- **Review step** – the script is inspected before execution, catching data-loss operations.
- **Separate deployment identity** – the script is applied by a deployment identity, not the application’s runtime identity.
- **No SDK required** – the script can be applied by a DBA or CI agent without the .NET SDK.

### Real-World Cases

- **CI/CD pipelines:** Generate idempotent scripts or bundles and apply them as a deployment job.
- **DBA-controlled environments:** Hand-reviewed SQL scripts to a DBA for execution.
- **Multi-instance deployments:** Use migration locking (EF Core 9+) to prevent concurrent migrations.
- **Existing databases:** Baseline an existing schema to avoid replaying all historical migrations.

### References

- Applying Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
- Migrations in Team Environments – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/teams
- Idempotent migration scripts – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying#idempotent-sql-scripts
- Migration bundles – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying#bundles
- Versioning Your Business App's Database Schema (Zenodo) – https://zenodo.org/records/21130477

---

## Summary Table: Migrations at a Glance

| Concept | Primary Role | Key Command / Syntax | When to Use |
|---------|--------------|----------------------|-------------|
| **Migrations** | Versioned schema evolution | `dotnet ef migrations add`, `Update-Database` | Every production application with evolving schema |
| **`Add-Migration`** | Scaffold a new migration | `Add-Migration <Name>` / `dotnet ef migrations add <Name>` | After every model change |
| **`Update-Database`** | Apply pending migrations | `Update-Database` / `dotnet ef database update` | Local development and testing |
| **Best practices** | Safe, collaborative, production-ready | SQL scripts, bundles, idempotent scripts, team coordination | Every team and production deployment |

---

## References (Full List)

- Migrations Overview – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/
- Managing Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying Migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
- Migrations in Team Environments – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/teams
- EF Core tools reference (.NET CLI) – https://learn.microsoft.com/en-us/ef/core/cli/dotnet
- EF Core tools reference (PMC) – https://learn.microsoft.com/en-us/ef/core/cli/powershell
- Idempotent SQL scripts – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying#idempotent-sql-scripts
- Migration bundles – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying#bundles
- Part 4, add a model to an ASP.NET Core MVC app (Add-Migration and Update-Database) – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-model
- Versioning Your Business App's Database Schema (Zenodo) – https://zenodo.org/records/21130477
- MigrationsSqlGenerationOptions Enum – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.migrations.migrationssqlgenerationoptions