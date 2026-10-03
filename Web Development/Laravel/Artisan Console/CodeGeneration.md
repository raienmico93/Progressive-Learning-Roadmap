# Laravel Code Generation (The `make:` Ecosystem) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `make:` ecosystem is Artisan's suite of code generation commands that scaffold classes, files, and boilerplate code for common Laravel components, accelerating development by automating repetitive file creation tasks.

**Technical Definition:** Each `make:*` command is implemented as a subclass of `Illuminate\Console\GeneratorCommand`, which extends `Illuminate\Console\Command`. The generator reads a stub file (from `stubs/` or the framework's internal stubs), performs token replacement (`{{ class }}`, `{{ namespace }}`, etc.), and writes the result to the appropriate directory. The `GeneratorCommand` class provides methods such as `getStub()`, `getDefaultNamespace()`, `getNameInput()`, and `resolveStubPath()` that subclasses override to customise generation behaviour. Commands are registered by Laravel's core service providers and are available in every application.

**Beginner-Friendly Explanation:** Laravel's `make:` commands are code generators. Instead of creating a new file, typing the namespace, class declaration, and boilerplate code manually, you run a command like `php artisan make:model User` and Laravel creates the file for you, already filled in with the correct structure. There are `make:` commands for almost every type of class in Laravel — controllers, models, migrations, jobs, events, and many more. This saves time and ensures consistency across your codebase.

### Key Characteristics

- **Stub-based generation:** Each command uses a stub file as a template, with placeholders replaced by the generated class name and namespace.
- **Auto-discovery:** Generated classes are placed in conventional directories (`app/Models`, `app/Http/Controllers`, etc.) that are automatically autoloaded by Composer.
- **Options for related files:** Many commands accept flags (e.g., `--migration`, `--factory`, `--seed`) to generate related files in a single invocation.
- **Customisable stubs:** Stubs can be published (`php artisan stub:publish`) and modified to match your application's conventions.
- **Test generation:** Many commands accept `--test` or `--pest` flags to generate accompanying test files.
- **Namespace support:** Class names can include subdirectory paths (e.g., `make:controller Auth/LoginController`) to organise generated files into subdirectories.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the `artisan` script at the project root.
- A terminal with PHP available in the PATH.
- For test generation: PHPUnit or Pest installed and configured.
- For resource generation: Eloquent models defined (for `--model` options).

### Related Programming Areas

- **Artisan Console** — The underlying CLI framework providing command parsing and execution.
- **Service Container** — Generated classes are resolved through Laravel's DI container.
- **Eloquent ORM** — Models, factories, seeders, and observers are generated for Eloquent.
- **Blade Templating** — View components and layouts are generated for Blade.
- **Testing** — Test classes are generated for PHPUnit or Pest.
- **Authorization** — Policies and gates are generated for access control.

### Core Concepts / Features

1. HTTP Layer (Controllers, Middleware, and Form Requests)
2. Database Layer (Models, Migrations, Factories, Seeders, and Casts)
3. Asynchronous & Background (Jobs, Events, Listeners, and Batch Tables)
4. Application Services (Mailables, Notifications, and Service Providers)
5. Security & Testing (Policies, Gates, Test Cases, and Pest/PHPUnit Configurations)
6. API & View Layers (Eloquent Resources, Components (Blade/Livewire), and Layouts)
7. System & Maintenance (Exceptions, Enums, Interfaces, and Scope Classes)

---

## 1. HTTP Layer

### Definitions

**Core Definition:** The HTTP layer generators create classes that handle incoming HTTP requests — controllers, middleware, and form requests — forming the entry point for all web and API traffic in a Laravel application.

**Technical Definition:** The `make:controller` command is implemented by `Illuminate\Routing\Console\ControllerMakeCommand`, which extends `GeneratorCommand`. It supports `--resource`, `--api`, `--requests`, `--invokable`, `--model`, and `--parent` options. The `make:middleware` command is implemented by `Illuminate\Routing\Console\MiddlewareMakeCommand`. The `make:request` command is implemented by `Illuminate\Foundation\Console\RequestMakeCommand`. All three commands place their output in `app/Http/Controllers`, `app/Http/Middleware`, and `app/Http/Requests` respectively (or `app/Http/Requests` in Laravel 11+).

**Beginner-Friendly Explanation:** Controllers, middleware, and form requests are the three most common classes you'll create when building the HTTP layer of your application. The `make:` commands generate these files with the correct namespace, class name, and boilerplate methods, so you can focus on writing the logic instead of setting up the file structure.

### Purposes

- To scaffold controller classes with optional resource methods (index, create, store, show, edit, update, destroy).
- To generate middleware classes for request/response filtering.
- To create form request classes for validation and authorisation logic.
- To generate related files (resource controllers, form requests) in a single command.
- To organise HTTP classes into subdirectories using namespace paths.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Controller
php artisan make:controller <Name> [--resource] [--api] [--requests] [--invokable] [--model=Model] [--parent=Parent]

# Middleware
php artisan make:middleware <Name> [--test] [--pest]

# Form Request
php artisan make:request <Name>
```

**Component Breakdown:**

- `<Name>` — The class name, optionally prefixed with a subdirectory path (e.g., `Auth/LoginController`).
- `--resource` — Generates a controller with all resource methods (index, create, store, show, edit, update, destroy).
- `--api` — Generates a resource controller without `create` and `edit` methods (for APIs).
- `--requests` — Generates form request classes for validation and injects them into the controller methods.
- `--invokable` — Generates a single-action controller with an `__invoke` method.
- `--model=<Model>` — Binds the controller to a model (used with `--resource`).
- `--parent=<Parent>` — Generates the controller in a subdirectory under the parent name.

```php
// Example: Resource controller with form requests
php artisan make:controller PostController --resource --requests --model=Post

// Example: API controller
php artisan make:controller Api/UserController --api --model=User

// Example: Invokable controller
php artisan make:controller Auth/RegisterController --invokable
```

**Syntax Rules:**

- The class name can include a subdirectory path using forward slashes (e.g., `Admin/DashboardController`).
- The `--resource` and `--api` options are mutually exclusive in practice (use `--api` for API-only controllers).
- The `--requests` option requires `--resource` or `--api` to generate meaningful form request classes.
- The `--model` option binds the controller to an Eloquent model, enabling route model binding in the generated methods.
- Middleware and form requests support `--test` and `--pest` flags for test generation (Laravel 11+).

**Constraints and Limitations:**

- **Generated controllers are minimal.** They include method stubs but no business logic. You must implement the logic yourself.
- **The `--requests` option generates one form request per method**, which can be excessive for simple controllers. Use it selectively.
- **Subdirectory paths create nested directories.** Ensure the namespace matches the directory structure (Laravel handles this automatically).
- **Form request authorisation defaults to `false`.** You must implement the `authorize()` method to return `true` or a gate/policy check.

### Annotated Code Examples

**Example 1: Generating a Resource Controller with Form Requests**

```bash
php artisan make:controller PostController --resource --requests --model=Post
```

**Expected Output:**

```
Controller created successfully.
Request created successfully.
Request created successfully.
Request created successfully.
Request created successfully.
Request created successfully.
```

**Generated Files:**

- `app/Http/Controllers/PostController.php` — Resource controller with `index`, `create`, `store`, `show`, `edit`, `update`, `destroy` methods.
- `app/Http/Requests/StorePostRequest.php` — Form request for `store`.
- `app/Http/Requests/UpdatePostRequest.php` — Form request for `update`.
- Additional request classes for `index`, `show`, and `destroy` (Laravel 11+).

**Why This Output Occurs:** The `--resource` flag generates a controller with the seven resource methods. The `--requests` flag generates a form request class for each method that requires validation, and injects them as type-hinted parameters in the controller methods. The `--model=Post` flag binds the controller to the `Post` model, enabling route model binding in the `show`, `update`, and `destroy` methods.

---

**Example 2: Generating Middleware and Form Request**

```bash
# Generate middleware
php artisan make:middleware EnsureUserIsAdmin

# Generate a form request
php artisan make:request StoreCommentRequest
```

**Expected Output:**

```
Middleware created successfully.
Request created successfully.
```

**Generated Files:**

- `app/Http/Middleware/EnsureUserIsAdmin.php` — Middleware with a `handle()` method.
- `app/Http/Requests/StoreCommentRequest.php` — Form request with `authorize()` and `rules()` methods.

**Why This Output Occurs:** The `make:middleware` command generates a class implementing the middleware contract with a `handle()` method that receives the request and a closure. The `make:request` command generates a form request class with an `authorize()` method (returning `false` by default) and a `rules()` method (returning an empty array).

### Real-World Cases

- **CRUD controllers:** `php artisan make:controller ProductController --resource --model=Product` generates a full CRUD controller bound to the `Product` model.
- **API controllers:** `php artisan make:controller Api/OrderController --api --model=Order` generates an API-only controller with `index`, `store`, `show`, `update`, and `destroy` methods.
- **Authentication middleware:** `php artisan make:middleware EnsureEmailIsVerified` generates middleware for email verification checks.
- **Validation requests:** `php artisan make:request UpdateProfileRequest` generates a form request for profile update validation.

---

## 2. Database Layer

### Definitions

**Core Definition:** The database layer generators create classes and files that define, populate, and interact with the application's database schema — models, migrations, factories, seeders, and casts.

**Technical Definition:** The `make:model` command is implemented by `Illuminate\Foundation\Console\ModelMakeCommand` and supports numerous flags for generating related files. The `make:migration` command is implemented by `Illuminate\Database\Console\Migrations\MigrateMakeCommand`. The `make:factory` command is implemented by `Illuminate\Database\Console\Factories\FactoryMakeCommand`. The `make:seeder` command is implemented by `Illuminate\Database\Console\Seeds\SeederMakeCommand`. The `make:cast` command is implemented by `Illuminate\Foundation\Console\CastMakeCommand`.

**Beginner-Friendly Explanation:** When building a database-driven application, you need models (PHP classes that represent tables), migrations (files that define the table structure), factories (classes that generate fake data), seeders (classes that populate the database), and casts (classes that convert between database and PHP types). The `make:` commands generate all of these, so you don't have to write boilerplate.

### Purposes

- To generate Eloquent model classes with proper namespace and inheritance.
- To create migration files for defining and modifying database tables.
- To generate model factories for creating fake data in tests and seeders.
- To create seeder classes for populating the database with initial or test data.
- To generate custom cast classes for converting between database values and PHP types.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Model (with optional related files)
php artisan make:model <Name> [-m|--migration] [-f|--factory] [-s|--seed] [-c|--controller] [-r|--resource] [-R|--requests] [-p|--policy] [-a|--all] [--pivot] [--morph-pivot] [--test] [--pest]

# Migration
php artisan make:migration <name> [--create=table] [--table=table] [--path=path]

# Factory
php artisan make:factory <Name> [--model=Model]

# Seeder
php artisan make:seeder <Name>

# Cast
php artisan make:cast <Name> [--inbound]
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `User`, `Post`, `OrderItem`).
- `-m|--migration` — Generate a migration file for the model's table.
- `-f|--factory` — Generate a model factory.
- `-s|--seed` — Generate a seeder class.
- `-c|--controller` — Generate a controller for the model.
- `-r|--resource` — Generate a resource controller (implies `--controller`).
- `-R|--requests` — Generate form request classes for the resource controller.
- `-p|--policy` — Generate a policy class for the model.
- `-a|--all` — Generate all of the above (migration, factory, seeder, policy, resource controller, and form requests).
- `--pivot` — Indicates the model is a pivot table model.
- `--morph-pivot` — Indicates the model is a polymorphic pivot table model.
- `--create=table` — Create a migration for a new table (used with `make:migration`).
- `--table=table` — Create a migration for an existing table.
- `--inbound` — Generate an inbound-only cast class.

```bash
# Generate a model with migration, factory, and seeder
php artisan make:model Post -mfs

# Generate everything for a model
php artisan make:model Order -a

# Generate a pivot model
php artisan make:model RoleUser --pivot

# Generate a migration for a new table
php artisan make:migration create_posts_table --create=posts

# Generate a migration to modify an existing table
php artisan make:migration add_status_to_posts_table --table=posts

# Generate a factory bound to a model
php artisan make:factory PostFactory --model=Post

# Generate an inbound-only cast
php artisan make:cast MoneyCast --inbound
```

**Syntax Rules:**

- The `--all` flag (`-a`) generates a migration, factory, seeder, policy, resource controller, and form requests.
- Short flags can be combined: `-mfs` is equivalent to `--migration --factory --seed`.
- The `--pivot` flag generates a model that extends `Pivot` instead of `Model`.
- The `--create` option for `make:migration` creates a migration with a `Schema::create` call.
- The `--table` option creates a migration with a `Schema::table` call for modifying an existing table.

**Constraints and Limitations:**

- **Migrations are timestamped.** The filename includes the current timestamp, ensuring they run in the correct order.
- **Factories require the model to exist.** If `--model` is not specified, Laravel infers the model from the factory name (e.g., `PostFactory` → `Post`).
- **Seeders do not automatically call other seeders.** You must edit the `DatabaseSeeder` class to call your seeders.
- **Custom casts must implement `CastsAttributes` or `CastsInboundAttributes`.** The generated stub includes the necessary interface and methods.

### Annotated Code Examples

**Example 1: Generating a Complete Model Stack**

```bash
php artisan make:model Order -a
```

**Expected Output:**

```
Model created successfully.
Created Migration: 2025_06_15_100000_create_orders_table
Factory created successfully.
Seeder created successfully.
Policy created successfully.
Controller created successfully.
Request created successfully.
Request created successfully.
```

**Generated Files:**

- `app/Models/Order.php` — Eloquent model.
- `database/migrations/2025_06_15_100000_create_orders_table.php` — Migration for the `orders` table.
- `database/factories/OrderFactory.php` — Factory for generating fake orders.
- `database/seeders/OrderSeeder.php` — Seeder for populating the `orders` table.
- `app/Policies/OrderPolicy.php` — Policy for authorising order actions.
- `app/Http/Controllers/OrderController.php` — Resource controller.
- `app/Http/Requests/StoreOrderRequest.php` and `UpdateOrderRequest.php` — Form requests.

**Why This Output Occurs:** The `--all` flag triggers the generation of every related file. Each generator runs independently, reading its stub and writing the output to the conventional directory. The migration is timestamped to ensure it runs after any existing migrations.

---

**Example 2: Generating a Pivot Model and Custom Cast**

```bash
# Generate a pivot model
php artisan make:model RoleUser --pivot

# Generate a custom cast
php artisan make:cast AddressCast
```

**Expected Output:**

```
Model created successfully.
Cast created successfully.
```

**Generated Files:**

- `app/Models/RoleUser.php` — Pivot model extending `Illuminate\Database\Eloquent\Relations\Pivot`.
- `app/Casts/AddressCast.php` — Custom cast implementing `CastsAttributes`.

**Why This Output Occurs:** The `--pivot` flag changes the parent class from `Model` to `Pivot`, which is appropriate for intermediate table models. The `make:cast` command generates a class implementing the `CastsAttributes` interface with `get()` and `set()` methods.

### Real-World Cases

- **E-commerce models:** `php artisan make:model Product -mfsc` generates a product model with migration, factory, seeder, and controller.
- **User profiles:** `php artisan make:model Profile -m` generates a profile model and migration.
- **Many-to-many relationships:** `php artisan make:model CourseUser --pivot` generates a pivot model for a course-user relationship.
- **Custom value objects:** `php artisan make:cast MoneyCast` generates a cast for handling monetary values with precision.

---

## 3. Asynchronous & Background

### Definitions

**Core Definition:** The asynchronous and background generators create classes for deferring work — jobs, events, listeners, and the batch table migration — enabling non-blocking processing of time-consuming tasks.

**Technical Definition:** The `make:job` command is implemented by `Illuminate\Foundation\Console\JobMakeCommand` and supports `--sync` and `--queued` flags. The `make:event` command is implemented by `Illuminate\Foundation\Console\EventMakeCommand`. The `make:listener` command is implemented by `Illuminate\Foundation\Console\ListenerMakeCommand` and supports the `--event` option to bind the listener to a specific event. The `make:queue-batches-table` command is implemented by `Illuminate\Queue\Console\BatcherMakeCommand` and generates the `job_batches` migration.

**Beginner-Friendly Explanation:** When your application needs to do something slow — like sending an email, processing a video, or calling an external API — you don't want to make the user wait. Instead, you push a "job" onto a queue, and a background worker processes it later. Events and listeners work similarly: an event is something that happened (like "OrderShipped"), and a listener is the code that reacts to it. The `make:` commands generate these classes for you.

### Purposes

- To generate job classes for background processing.
- To create event classes that represent significant application occurrences.
- To generate listener classes that react to specific events.
- To create the `job_batches` table migration for batch job processing.
- To generate the `failed_jobs` table migration for tracking failed jobs.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Job
php artisan make:job <Name> [--sync] [--queued]

# Event
php artisan make:event <Name>

# Listener
php artisan make:listener <Name> [--event=EventName] [--queued]

# Batch table
php artisan make:queue-batches-table

# Failed jobs table
php artisan make:queue-failed-table
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `ProcessPodcast`, `OrderShipped`, `SendShipmentNotification`).
- `--sync` — Generate a job that runs synchronously (implements `ShouldQueue` is omitted).
- `--queued` — Generate a job that implements `ShouldQueue` (default in Laravel 11+).
- `--event=<Event>` — Bind the listener to a specific event class.
- `--queued` — Generate a queued listener (implements `ShouldQueue`).

```bash
# Generate a queued job
php artisan make:job ProcessPodcast

# Generate a synchronous job
php artisan make:job ProcessPodcast --sync

# Generate an event
php artisan make:event OrderShipped

# Generate a listener bound to an event
php artisan make:listener SendShipmentNotification --event=OrderShipped

# Generate the batch table migration
php artisan make:queue-batches-table

# Generate the failed jobs table migration
php artisan make:queue-failed-table
```

**Syntax Rules:**

- Jobs generated by `make:job` implement `ShouldQueue` by default in Laravel 11+ (use `--sync` to opt out).
- Listeners generated with `--event` are automatically type-hinted with the event class in the `handle()` method.
- The `--queued` flag on `make:listener` makes the listener implement `ShouldQueue`.
- The `make:queue-batches-table` command generates a migration for the `job_batches` table, required for batch job processing.

**Constraints and Limitations:**

- **Jobs must be dispatched, not called directly.** Use `dispatch()` or the `dispatch()` helper.
- **Listeners are auto-discovered** in Laravel 11+ if they are type-hinted with an event in their `handle()` method. Older versions require registration in `EventServiceProvider`.
- **Batch tables must be migrated before using batch jobs.** Run `php artisan migrate` after generating the migration.
- **The `failed_jobs` table is required** for tracking failed queue jobs.

### Annotated Code Examples

**Example 1: Generating a Job, Event, and Listener**

```bash
# Step 1: Generate a job
php artisan make:job ProcessOrder

# Step 2: Generate an event
php artisan make:event OrderProcessed

# Step 3: Generate a listener bound to the event
php artisan make:listener SendOrderConfirmation --event=OrderProcessed
```

**Expected Output:**

```
Job created successfully.
Event created successfully.
Listener created successfully.
```

**Generated Files:**

- `app/Jobs/ProcessOrder.php` — Job class with `handle()` method.
- `app/Events/OrderProcessed.php` — Event class with `__construct()` method.
- `app/Listeners/SendOrderConfirmation.php` — Listener class with `handle(OrderProcessed $event)` method.

**Why This Output Occurs:** The `make:job` command generates a class implementing `ShouldQueue` with a `handle()` method. The `make:event` command generates an event class with a constructor for event data. The `make:listener` command with `--event` generates a listener with a `handle()` method type-hinted to the specified event, enabling auto-discovery.

---

**Example 2: Setting Up Batch Job Infrastructure**

```bash
# Step 1: Generate the batch table migration
php artisan make:queue-batches-table

# Step 2: Run the migration
php artisan migrate

# Step 3: Generate a batchable job
php artisan make:job ProcessPodcast
```

**Expected Output:**

```
Created Migration: 2025_06_15_100000_create_job_batches_table
Migrating: 2025_06_15_100000_create_job_batches_table
Migrated:  2025_06_15_100000_create_job_batches_table (0.02 seconds)
Job created successfully.
```

**Why This Output Occurs:** The `make:queue-batches-table` command generates a migration for the `job_batches` table, which stores batch metadata. The `migrate` command creates the table. The generated job can then use the `Batchable` trait to participate in batch processing.

### Real-World Cases

- **Email sending:** `php artisan make:job SendWelcomeEmail` generates a job for queuing welcome emails.
- **Video processing:** `php artisan make:job ProcessVideo --queued` generates a queued job for transcoding uploaded videos.
- **Order events:** `php artisan make:event OrderShipped` and `php artisan make:listener SendShipmentNotification --event=OrderShipped` generate an event-listener pair for order shipment notifications.
- **Batch data processing:** Generate the batch table, create a batchable job, and dispatch batches of jobs for large-scale data processing.

---

## 4. Application Services

### Definitions

**Core Definition:** The application services generators create classes for sending email, notifications, and registering services — mailables, notifications, and service providers.

**Technical Definition:** The `make:mail` command is implemented by `Illuminate\Foundation\Console\MailMakeCommand` and supports `--markdown` for Markdown-based email templates. The `make:notification` command is implemented by `Illuminate\Foundation\Console\NotificationMakeCommand` and also supports `--markdown`. The `make:provider` command is implemented by `Illuminate\Foundation\Console\ProviderMakeCommand`.

**Beginner-Friendly Explanation:** Mailables are classes that represent emails your application sends. Notifications are a broader concept — they can be sent via email, SMS, Slack, or database. Service providers are the central place where you register bindings, events, middleware, and other services in Laravel's bootstrap process. The `make:` commands generate all of these.

### Purposes

- To generate mailable classes for sending emails.
- To create notification classes for multi-channel notifications.
- To generate service provider classes for registering application services.
- To create Markdown email templates for mailables and notifications.
- To organise email and notification logic into dedicated classes.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Mailable
php artisan make:mail <Name> [--markdown=view] [--test] [--pest]

# Notification
php artisan make:notification <Name> [--markdown=view] [--test] [--pest]

# Service Provider
php artisan make:provider <Name>
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `WelcomeEmail`, `InvoicePaid`, `PaymentServiceProvider`).
- `--markdown=<view>` — Generate a Markdown email template in addition to the class.
- `--test` — Generate an accompanying PHPUnit test.
- `--pest` — Generate an accompanying Pest test.

```bash
# Generate a mailable
php artisan make:mail WelcomeEmail

# Generate a mailable with a Markdown template
php artisan make:mail InvoicePaid --markdown=emails.invoices.paid

# Generate a notification
php artisan make:notification InvoicePaid

# Generate a notification with a Markdown template
php artisan make:notification InvoicePaid --markdown=notifications.invoices.paid

# Generate a service provider
php artisan make:provider PaymentServiceProvider
```

**Syntax Rules:**

- Mailables are placed in `app/Mail`; notifications in `app/Notifications`; service providers in `app/Providers`.
- The `--markdown` option generates a Blade template in `resources/views/` at the specified path.
- Mailables generated with `make:mail` use the `Queueable` trait, allowing them to be queued.
- Notifications implement the `Notification` contract and include a `via()` method for specifying delivery channels.

**Constraints and Limitations:**

- **Mailables do not send themselves.** You must use the `Mail` facade or the `mail()` helper to send them.
- **Notifications must be sent to a notifiable entity.** The `Notifiable` trait must be added to the User model (or any model that receives notifications).
- **Service providers must be registered.** In Laravel 10 and below, add them to `config/app.php`; in Laravel 11+, add them to `bootstrap/providers.php`.
- **Markdown templates require the `mail` view namespace.** Laravel automatically registers the `mail` namespace for Markdown components.

### Annotated Code Examples

**Example 1: Generating a Mailable with a Markdown Template**

```bash
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```

**Expected Output:**

```
Mail created successfully.
```

**Generated Files:**

- `app/Mail/OrderShipped.php` — Mailable class with `__construct()`, `envelope()`, and `content()` methods.
- `resources/views/emails/orders/shipped.blade.php` — Markdown email template.

**Why This Output Occurs:** The `make:mail` command generates a mailable class with a `content()` method that returns a `Markdown` instance pointing to the specified view. The `--markdown` option also generates the Blade template file, pre-populated with Markdown syntax.

---

**Example 2: Generating a Notification and Service Provider**

```bash
# Generate a notification
php artisan make:notification InvoicePaid

# Generate a service provider
php artisan make:provider PaymentServiceProvider
```

**Expected Output:**

```
Notification created successfully.
Provider created successfully.
```

**Generated Files:**

- `app/Notifications/InvoicePaid.php` — Notification class with `via()` and `toMail()` methods.
- `app/Providers/PaymentServiceProvider.php` — Service provider with `register()` and `boot()` methods.

**Why This Output Occurs:** The `make:notification` command generates a class implementing the `Notification` contract with a `via()` method that returns the delivery channels. The `make:provider` command generates a service provider extending `Illuminate\Support\ServiceProvider` with empty `register()` and `boot()` methods.

### Real-World Cases

- **Transactional emails:** `php artisan make:mail OrderConfirmation --markdown=emails.orders.confirmation` generates a mailable for order confirmation emails.
- **Multi-channel notifications:** `php artisan make:notification PaymentReceived` generates a notification that can be sent via email, database, and Slack.
- **Service registration:** `php artisan make:provider RepositoryServiceProvider` generates a provider for binding repository interfaces to implementations.
- **Event-driven services:** `php artisan make:provider EventServiceProvider` generates a provider for registering event listeners (though this is auto-discovered in Laravel 11+).

---

## 5. Security & Testing

### Definitions

**Core Definition:** The security and testing generators create classes for authorisation (policies and gates) and automated testing (test cases for PHPUnit or Pest).

**Technical Definition:** The `make:policy` command is implemented by `Illuminate\Foundation\Console\PolicyMakeCommand` and supports `--model`, `--guard`, and `--requests` options. The `make:test` command is implemented by `Illuminate\Foundation\Console\TestMakeCommand` and supports `--unit`, `--feature`, `--pest`, and `--phpunit` flags. Gates are not generated by a dedicated command; they are defined in service providers or via the `Gate` facade.

**Beginner-Friendly Explanation:** Policies are classes that determine whether a user is allowed to perform an action on a resource — like "can this user edit this post?" Tests are automated checks that verify your code works as expected. The `make:` commands generate both, so you can focus on writing the authorisation logic and test assertions.

### Purposes

- To generate policy classes for model-based authorisation.
- To create test classes for unit, feature, or browser testing.
- To generate tests using either PHPUnit or Pest syntax.
- To create tests for specific components (controllers, models, jobs, etc.).
- To organise authorisation and testing logic into dedicated classes.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Policy
php artisan make:policy <Name> [--model=Model] [--guard=Guard] [--requests]

# Test
php artisan make:test <Name> [--unit] [--feature] [--pest] [--phpunit]
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `PostPolicy`, `CreatePostTest`).
- `--model=<Model>` — Bind the policy to a specific model, generating methods for `viewAny`, `view`, `create`, `update`, `delete`, `restore`, and `forceDelete`.
- `--guard=<Guard>` — Specify the authentication guard the policy applies to.
- `--requests` — Generate form request classes for the policy's actions (Laravel 11+).
- `--unit` — Generate a unit test (placed in `tests/Unit`).
- `--feature` — Generate a feature test (placed in `tests/Feature`).
- `--pest` — Generate a Pest test (default if Pest is installed).
- `--phpunit` — Generate a PHPUnit test.

```bash
# Generate a policy bound to a model
php artisan make:policy PostPolicy --model=Post

# Generate a feature test with Pest
php artisan make:test CreatePostTest --pest

# Generate a unit test
php artisan make:test UserTest --unit

# Generate a PHPUnit feature test
php artisan make:test PostControllerTest --phpunit
```

**Syntax Rules:**

- Policies are placed in `app/Policies`; tests in `tests/Feature` or `tests/Unit`.
- The `--model` option generates policy methods for standard CRUD actions.
- Without `--model`, an empty policy class is generated.
- The `--pest` flag generates a Pest test (using the `test()` function instead of class methods).
- The `--phpunit` flag generates a PHPUnit test (using class methods).

**Constraints and Limitations:**

- **Policies must be registered.** Laravel auto-discovers policies if they follow the naming convention (`ModelPolicy`), but custom policies must be registered in `AuthServiceProvider` (Laravel 10-) or `AppServiceProvider` (Laravel 11+).
- **Generated tests are empty.** They include the test class and a basic test method, but no assertions.
- **Pest tests require Pest to be installed.** If Pest is not installed, `--pest` may fail.
- **Unit tests do not have access to the full application.** They extend `PHPUnit\Framework\TestCase` (not `Tests\TestCase`).

### Annotated Code Examples

**Example 1: Generating a Policy with Model Binding**

```bash
php artisan make:policy PostPolicy --model=Post
```

**Expected Output:**

```
Policy created successfully.
```

**Generated File:** `app/Policies/PostPolicy.php` with methods:

```php
public function viewAny(User $user): bool
public function view(User $user, Post $post): bool
public function create(User $user): bool
public function update(User $user, Post $post): bool
public function delete(User $user, Post $post): bool
public function restore(User $user, Post $post): bool
public function forceDelete(User $user, Post $post): bool
```

**Why This Output Occurs:** The `--model=Post` option generates a policy with all standard CRUD authorisation methods, each type-hinted with the `User` model and the `Post` model. The policy is automatically discovered by Laravel because it follows the `ModelPolicy` naming convention.

---

**Example 2: Generating Feature Tests with Pest**

```bash
# Generate a feature test with Pest
php artisan make:test PostControllerTest --pest

# Generate a unit test with PHPUnit
php artisan make:test UserTest --unit --phpunit
```

**Expected Output:**

```
Test created successfully.
Test created successfully.
```

**Generated Files:**

- `tests/Feature/PostControllerTest.php` — Pest test with `test()` functions.
- `tests/Unit/UserTest.php` — PHPUnit test with class methods.

**Why This Output Occurs:** The `--pest` flag generates a Pest test using the `test()` function syntax. The `--unit` flag places the test in `tests/Unit`, and `--phpunit` generates a PHPUnit-style test class extending `PHPUnit\Framework\TestCase`.

### Real-World Cases

- **CRUD authorisation:** `php artisan make:policy PostPolicy --model=Post` generates a policy for authorising post actions.
- **API tests:** `php artisan make:test Api/UserControllerTest --pest` generates a Pest feature test for API endpoints.
- **Unit tests for services:** `php artisan make:test PaymentServiceTest --unit` generates a unit test for a payment service class.
- **Browser tests:** `php artisan make:test LoginTest --feature --phpunit` generates a feature test for browser-based login.

---

## 6. API & View Layers

### Definitions

**Core Definition:** The API and view layer generators create classes for transforming data into JSON responses (Eloquent resources) and for building reusable UI components (Blade components and layouts).

**Technical Definition:** The `make:resource` command is implemented by `Illuminate\Foundation\Console\ResourceMakeCommand` and supports the `--collection` flag. The `make:component` command is implemented by `Illuminate\Foundation\Console\ComponentMakeCommand` and supports `--inline` and `--view` options. The `make:layout` command (Laravel 12+) generates Blade layout components.

**Beginner-Friendly Explanation:** When building an API, you often need to transform your models into JSON — but you might not want to expose every attribute. Eloquent resources let you control exactly what data is returned. For the frontend, Blade components are reusable UI pieces (like an alert box or a card) that you can drop into any view. The `make:` commands generate both.

### Purposes

- To generate resource classes for transforming models into JSON.
- To create resource collections for transforming model collections.
- To generate Blade component classes and views.
- To create layout components for consistent page structure.
- To organise API transformation and UI component logic into dedicated classes.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Resource
php artisan make:resource <Name> [--collection]

# Component
php artisan make:component <Name> [--inline] [--view]

# Layout
php artisan make:layout <Name>
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `UserResource`, `Alert`, `AppLayout`).
- `--collection` — Generate a resource collection instead of a single resource.
- `--inline` — Generate a component with an inline render method (no separate view file).
- `--view` — Generate a component with a view file (default).

```bash
# Generate a resource
php artisan make:resource UserResource

# Generate a resource collection
php artisan make:resource UserCollection --collection

# Generate a Blade component
php artisan make:component Alert

# Generate an inline component
php artisan make:component Badge --inline

# Generate a layout component
php artisan make:layout AppLayout
```

**Syntax Rules:**

- Resources are placed in `app/Http/Resources`.
- Components are placed in `app/View/Components` (class) and `resources/views/components` (view).
- Resource collections extend `Illuminate\Http\Resources\Json\ResourceCollection`.
- Layout components extend `Illuminate\View\Component` and are typically used with `<x-layout>` syntax.

**Constraints and Limitations:**

- **Resources must be returned from routes or controllers.** They are not automatically applied to model responses.
- **Components require a view.** Unless `--inline` is used, a Blade view file is generated and must exist.
- **Layout components are only available in Laravel 12+.** Older versions use traditional Blade layouts with `@extends` and `@section`.

### Annotated Code Examples

**Example 1: Generating a Resource and Resource Collection**

```bash
# Generate a single resource
php artisan make:resource UserResource

# Generate a resource collection
php artisan make:resource UserCollection --collection
```

**Expected Output:**

```
Resource created successfully.
Resource created successfully.
```

**Generated Files:**

- `app/Http/Resources/UserResource.php` — Resource class with `toArray()` method.
- `app/Http/Resources/UserCollection.php` — Collection resource extending `ResourceCollection`.

**Why This Output Occurs:** The `make:resource` command generates a class extending `JsonResource` with a `toArray()` method that returns an array of attributes. The `--collection` flag generates a class extending `ResourceCollection` instead.

---

**Example 2: Generating a Blade Component with View**

```bash
php artisan make:component Alert
```

**Expected Output:**

```
Component created successfully.
```

**Generated Files:**

- `app/View/Components/Alert.php` — Component class with `render()` method.
- `resources/views/components/alert.blade.php` — Blade view for the component.

**Why This Output Occurs:** The `make:component` command generates a component class and a corresponding Blade view. The `render()` method returns the view, and the component can be used in templates as `<x-alert>...</x-alert>`.

### Real-World Cases

- **API responses:** `php artisan make:resource UserResource` generates a resource for transforming user models into JSON with only the desired attributes.
- **Collection responses:** `php artisan make:resource UserCollection --collection` generates a collection resource for paginated user lists.
- **UI components:** `php artisan make:component Alert` generates an alert component for displaying notifications in the UI.
- **Layouts:** `php artisan make:layout AppLayout` generates a layout component for consistent page structure (Laravel 12+).

---

## 7. System & Maintenance

### Definitions

**Core Definition:** The system and maintenance generators create classes for handling exceptions, defining enums, generating interfaces, and creating query scope classes.

**Technical Definition:** The `make:exception` command is implemented by `Illuminate\Foundation\Console\ExceptionMakeCommand` and supports `--render` and `--report` options. The `make:enum` command is implemented by `Illuminate\Foundation\Console\EnumMakeCommand` (Laravel 11+) and supports `--string` and `--int` for backed enums. The `make:interface` command is implemented by `Illuminate\Foundation\Console\InterfaceMakeCommand` (Laravel 11+). The `make:scope` command is available via community packages (e.g., `samasend/laravel-make-scope`) or Laravel 12's built-in `make:scope`.

**Beginner-Friendly Explanation:** Exceptions are custom error types that you can throw and catch. Enums are a PHP 8.1 feature for defining a set of named constants. Interfaces define contracts that classes must implement. Scopes are reusable query constraints for Eloquent models. The `make:` commands generate all of these, helping you organise your codebase with dedicated classes.

### Purposes

- To generate custom exception classes with optional `render()` and `report()` methods.
- To create backed or pure enums for type-safe constants.
- To generate interface classes for defining contracts.
- To create query scope classes for reusable model query constraints.
- To organise system-level and maintenance logic into dedicated classes.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Exception
php artisan make:exception <Name> [--render] [--report]

# Enum
php artisan make:enum <Name> [--string] [--int]

# Interface
php artisan make:interface <Name>

# Scope (Laravel 12+)
php artisan make:scope <Name> [--extend]
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `UserNotFoundException`, `OrderStatus`, `PaymentGatewayInterface`, `ActiveScope`).
- `--render` — Generate the exception with an empty `render()` method.
- `--report` — Generate the exception with an empty `report()` method.
- `--string` — Generate a string-backed enum.
- `--int` — Generate an integer-backed enum.
- `--extend` — Generate a scope class with an `extend()` method for global scopes.

```bash
# Generate a custom exception with render and report methods
php artisan make:exception PaymentFailedException --render --report

# Generate a string-backed enum
php artisan make:enum OrderStatus --string

# Generate an interface
php artisan make:interface PaymentGatewayInterface

# Generate a scope class
php artisan make:scope ActiveScope
```

**Syntax Rules:**

- Exceptions are placed in `app/Exceptions`; enums in `app/Enums`; interfaces in `app/Contracts` (Laravel 11+ default).
- The `--render` and `--report` options add empty methods that you can implement.
- Backed enums require either `--string` or `--int` to specify the backing type.
- Scope classes are placed in `app/Models/Scopes` (Laravel 12+ default).

**Constraints and Limitations:**

- **Enums require PHP 8.1+.** Laravel 10+ supports enums natively.
- **Interfaces do not generate implementations.** You must create the implementing classes separately.
- **Scope classes require Laravel 12+** for the built-in `make:scope` command. Older versions require a community package.
- **Exception `render()` and `report()` methods must be implemented.** The generated stubs are empty.

### Annotated Code Examples

**Example 1: Generating a Custom Exception and Enum**

```bash
# Generate a custom exception with render method
php artisan make:exception PaymentFailedException --render

# Generate a string-backed enum
php artisan make:enum OrderStatus --string
```

**Expected Output:**

```
Exception created successfully.
Enum created successfully.
```

**Generated Files:**

- `app/Exceptions/PaymentFailedException.php` — Exception class with `render()` method.
- `app/Enums/OrderStatus.php` — String-backed enum with `case` declarations.

**Why This Output Occurs:** The `make:exception` command generates a class extending `Exception` with a `render()` method that can return a custom HTTP response. The `--string` flag on `make:enum` generates a string-backed enum with a `: string` type declaration and empty case declarations.

---

**Example 2: Generating an Interface and Scope**

```bash
# Generate an interface
php artisan make:interface PaymentGatewayInterface

# Generate a scope class
php artisan make:scope ActiveScope
```

**Expected Output:**

```
Interface created successfully.
Scope created successfully.
```

**Generated Files:**

- `app/Contracts/PaymentGatewayInterface.php` — Interface with method signatures.
- `app/Models/Scopes/ActiveScope.php` — Scope class implementing `Scope` interface.

**Why This Output Occurs:** The `make:interface` command generates an empty interface with the specified name and namespace. The `make:scope` command generates a class implementing the `Illuminate\Database\Eloquent\Scope` interface with an `apply()` method for adding query constraints.

### Real-World Cases

- **Domain exceptions:** `php artisan make:exception InsufficientFundsException --render` generates a custom exception for insufficient funds with a custom HTTP response.
- **Status enums:** `php artisan make:enum OrderStatus --string` generates a string-backed enum for order statuses.
- **Payment interfaces:** `php artisan make:interface PaymentGatewayInterface` generates an interface for payment gateway implementations.
- **Query scopes:** `php artisan make:scope ActiveScope` generates a scope for filtering active records.

---

## References

- Laravel Artisan Console Documentation (Master) — https://laravel.com/docs/master/artisan 
- Laravel Controllers Documentation — https://laravel.com/docs/master/controllers 
- Laravel Eloquent: Getting Started — https://laravel.com/docs/master/eloquent 
- Laravel Eloquent: API Resources — https://laravel.com/docs/master/eloquent-resources 
- Laravel Migrations Documentation — https://laravel.com/docs/master/migrations 
- Laravel Queues Documentation — https://laravel.com/docs/master/queues 
- Laravel Events Documentation — https://laravel.com/docs/master/events 
- Laravel Mail Documentation — https://laravel.com/docs/master/mail 
- Laravel Notifications Documentation — https://laravel.com/docs/master/notifications 
- Laravel Authorization Documentation — https://laravel.com/docs/master/authorization 
- Laravel Testing Documentation — https://laravel.com/docs/master/testing 
- Laravel Blade Templates Documentation — https://laravel.com/docs/master/blade 
- Laravel Eloquent: Mutators & Casting — https://laravel.com/docs/master/eloquent-mutators 
- Laravel `make:enum` Command (Laravel Daily) — https://laraveldaily.com/post/laravel-11-new-artisan-make-enum-command 
- Laravel `make:interface` Command (Laravel Daily) — https://laraveldaily.com/post/laravel-11-new-artisan-make-interface-command 
- Laravel `make:job-middleware` Command (Laravel News) — https://laravel-news.com/laravel-11-26-released 
- Laravel `make:scope` Package (Laravel News) — https://laravel-news.com/laravel-scopes-generator 
- Laravel Artisan Cheatsheet (Artisan.page) — https://artisan.page/ 
- Laravel `make:exception` (Artisan.page) — https://artisan.page/13.x/makeexception 
- Laravel `make:cast` (Artisan.page) — https://artisan.page/13.x/makecast 
- Laravel `make:interface` (Artisan.page) — https://artisan.page/13.x/makeinterface