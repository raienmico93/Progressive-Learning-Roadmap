# Object-Relational Mapping (ORM) Paradigms — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Object-Relational Mapping (ORM) is a programming technique that maps in-memory objects in an object-oriented language like PHP to tables in a relational database, providing a structured and predefined approach to dealing with the object-relational impedance mismatch.

**Technical Definition**

ORM is a layer of software that performs bidirectional transfer of data between a persistent data store (typically a relational database) and an in-memory data representation (the domain layer). Its goal is to keep the in-memory representation and the persistent data store independent of each other and of the mapper itself. ORM tools translate between the object model (classes, inheritance, references) and the relational model (tables, rows, foreign keys), providing transparent persistence for PHP objects.

**Beginner-Friendly Explanation**

Imagine you speak PHP and your database speaks SQL. You think in terms of objects like `$user->name`, but the database thinks in terms of tables with rows and columns. ORM is a translator that sits between the two, letting you work with PHP objects while the ORM handles all the SQL translation behind the scenes. You never have to write `INSERT INTO users (name) VALUES ('Alice')` — you just create a PHP object and save it.

### Key Characteristics

- **Bidirectional Translation:** ORM translates both from objects to database rows and from database rows back to objects.
- **Persistence Ignorance:** In well-designed ORMs (Data Mapper), domain objects do not need to know how they are persisted.
- **Query Abstraction:** ORM provides a query language or builder that abstracts SQL, allowing developers to query in object-oriented terms.
- **Identity Management:** ORMs track entity identity, ensuring that the same database row always maps to the same in-memory object instance within a request.
- **Relationship Handling:** ORMs manage relationships (one-to-many, many-to-many) and provide navigation between related objects.
- **Change Tracking:** ORMs track changes to objects and generate the appropriate `UPDATE` or `INSERT` statements when changes are flushed.

### Prerequisites

- **PHP 8.1+** with PDO extension enabled.
- **A relational database** (MySQL 8.0+, MariaDB, PostgreSQL, SQLite) with a schema prepared for the examples.
- **Composer** for installing ORM libraries (Eloquent, Doctrine).
- **Understanding of PDO basics** — prepared statements, connections, and fetch modes.
- **Familiarity with object-oriented PHP** — classes, properties, methods, interfaces, and inheritance.

### Related Programming Areas

- **Database Abstraction:** ORM is a higher-level form of database abstraction, building on the Repository and Data Mapper patterns.
- **Domain-Driven Design (DDD):** ORM is often used to persist rich domain models with entities, value objects, and aggregates.
- **Query Optimization:** ORM can generate inefficient SQL (N+1 queries, unnecessary JOINs), requiring careful tuning.
- **Testing:** ORM impacts testability — Active Record is harder to unit test without a database; Data Mapper allows pure domain objects to be tested in isolation.
- **Migrations and Schema Management:** ORMs often include schema generation and migration tools (Doctrine SchemaTool, Laravel Migrations).

### Core Concepts / Features

1. **The Impedance Mismatch** — The structural friction between relational tables and object-oriented classes.
2. **Active Record Pattern (Laravel Eloquent)** — Mapping a database row directly to a model class containing both data and persistence logic.
3. **Data Mapper Pattern (Doctrine)** — Separating entity definitions from the persistence engine (Entity Managers and Repositories).

---

## Core Concept 1: The Impedance Mismatch

### Definitions

**Core Definition**

The object-relational impedance mismatch is the lack of compatibility between object-oriented programs and relational databases — a fundamental structural friction that makes direct mapping between the two paradigms difficult.

**Technical Definition**

The impedance mismatch arises because the relational model and the object model organise data in fundamentally different ways. Relational databases require tables (not classes), rows (not objects), and foreign keys (not references). The formal mathematical model for databases ensures integrity through tables with rows, columns, and references to other tables. Object-oriented programs use classes with encapsulation, inheritance, polymorphism, and object references. The mismatch manifests in several specific areas: identity (primary keys vs. object identity), inheritance (no direct equivalent in relational tables), associations (foreign keys vs. object references), and granularity (a single object may map to multiple tables or vice versa). ORM tools provide a structured and predefined approach to dealing with this mismatch by acting as a translation mechanism from objects to relational data and backwards.

**Beginner-Friendly Explanation**

Imagine you have a Lego model of a car (the object model) and you need to store it in a flat IKEA box (the relational database). The Lego car has wheels attached to axles, doors that open, and a roof that connects to the body. The IKEA box only has flat compartments with labels. To store the Lego car, you have to disassemble it: wheels go in compartment A, doors in compartment B, and so on. To use it again, you reassemble it. The impedance mismatch is the effort required to disassemble and reassemble. ORM is a robotic assistant that handles this disassembly and reassembly automatically.

### Purposes

- To understand why ORM tools exist and what problems they solve.
- To recognise the specific areas where object-oriented code and relational databases clash.
- To evaluate the trade-offs of using ORM versus writing raw SQL.
- To design domain models that work harmoniously with the database schema.
- To anticipate performance pitfalls (N+1 queries, unnecessary joins) that stem from the mismatch.

### Syntax Rules and Structure

The impedance mismatch is a conceptual problem, not a syntax feature. It manifests through the following structural conflicts:

**Conflict 1: Identity**

- **Object model:** Objects have identity by reference. Two objects with the same data are still different objects.
- **Relational model:** Rows have identity by primary key. Two rows with the same primary key are the same row.
- **ORM solution:** ORMs use the primary key to track object identity (the Identity Map pattern).

**Conflict 2: Inheritance**

- **Object model:** Classes can inherit from other classes (`class Admin extends User`).
- **Relational model:** Tables do not support inheritance. A common solution is single-table inheritance (all columns in one table), class-table inheritance (one table per class), or concrete-table inheritance (each class gets its own table).
- **ORM solution:** ORMs provide inheritance mapping strategies (single-table, joined, table-per-class).

**Conflict 3: Associations**

- **Object model:** Objects reference other objects directly (`$order->customer`).
- **Relational model:** Rows reference other rows through foreign key columns (`customer_id`).
- **ORM solution:** ORMs translate foreign keys into object references and manage lazy or eager loading.

**Conflict 4: Granularity**

- **Object model:** A single object may contain nested value objects (e.g., an `Address` object inside a `User` object).
- **Relational model:** Nested data must be flattened into columns or separate tables.
- **ORM solution:** ORMs map embedded value objects to columns or separate tables depending on configuration.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Demonstrating the Mismatch in Raw PDO**

```php
<?php
// --- Object-oriented thinking ---
// A User has a name, an email, and a list of Order objects.
// $user->orders[0]->total is a natural way to express a relationship.

// --- Relational reality with raw PDO ---
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Fetch a user
$stmt = $pdo->prepare('SELECT id, name, email FROM users WHERE id = :id');
$stmt->execute([':id' => 1]);
$userRow = $stmt->fetch();
// $userRow is an array, not an object. No methods, no behaviour.

// Fetch the user's orders
$stmt = $pdo->prepare('SELECT id, total FROM orders WHERE user_id = :user_id');
$stmt->execute([':user_id' => $userRow['id']]);
$orderRows = $stmt->fetchAll();
// $orderRows is an array of arrays. No relationship navigation.

// Manual translation is required at every step.
echo "User: {$userRow['name']}\n";
foreach ($orderRows as $order) {
    echo "  Order #{$order['id']}: \${$order['total']}\n";
}
```

**Expected Output:**

```
User: Alice
  Order #101: $99.99
  Order #102: $49.50
```

**Why:** Without an ORM, the developer must manually translate between arrays (relational representation) and objects (object-oriented representation). Relationships require separate queries and manual association. This demonstrates the impedance mismatch in practice.

**Example 2: The Same Logic with an ORM (Conceptual)**

```php
<?php
// With an ORM, the same logic becomes object-oriented:
$user = $userRepository->find(1);
echo "User: {$user->getName()}\n";
foreach ($user->getOrders() as $order) {
    echo "  Order #{$order->getId()}: \${$order->getTotal()}\n";
}
// The ORM handles the translation between objects and database rows.
```

**Expected Output:**

```
User: Alice
  Order #101: $99.99
  Order #102: $49.50
```

**Why:** The ORM eliminates the manual translation. The developer works with objects and relationships, and the ORM generates the appropriate SQL behind the scenes.

### Real-World Cases

**Case 1: Legacy PHP Application Migration**

A legacy application uses raw PDO queries and arrays throughout. When the team attempts to introduce a rich domain model with entities and value objects, they encounter the impedance mismatch: the domain model expects objects with behaviour, but the database returns flat arrays. They adopt an ORM to bridge the gap.

**Case 2: Complex Inheritance Hierarchies**

A billing system has `Payment` as an abstract base class with `CreditCardPayment` and `BankTransferPayment` subclasses. The relational database has no concept of inheritance. The ORM maps the hierarchy using single-table inheritance, storing all payment types in one table with a `type` discriminator column.

**Case 3: Performance-Critical Reporting**

A reporting query joins five tables and aggregates millions of rows. Using an ORM to hydrate full domain objects for each row would be prohibitively slow. The team uses raw SQL with PDO for reporting (where object identity is irrelevant) and ORM for transactional CRUD operations (where domain logic matters).

---

## Core Concept 2: Active Record Pattern (Laravel Eloquent Style)

### Definitions

**Core Definition**

The Active Record pattern treats each database row as an object, and that object knows how to save, update, and delete itself — the data (the row's columns) and the persistence behaviour (CRUD) live together on the same class.

**Technical Definition**

In the Active Record pattern, a model class maps directly to a database table. Each instance of the model represents a single row. The model class contains both the data (attributes corresponding to columns) and the persistence methods (`save()`, `delete()`, `find()`, etc.). Laravel's Eloquent ORM is the canonical PHP implementation: all Eloquent models extend `Illuminate\Database\Eloquent\Model` and provide a rich set of methods for querying, relationships, and persistence. Every database table has a corresponding "model" that is used to interact with that table, allowing developers to query for data in tables as well as insert new records.

**Beginner-Friendly Explanation**

Think of an Active Record model as a self-aware row. If a database row could walk and talk, it would be an Active Record object. It knows its own data (name, email, etc.) and it knows how to save itself to the database, update itself, and delete itself. You do not need a separate "manager" or "repository" — the row itself handles everything.

### Purposes

- To provide a simple, intuitive mapping between database tables and PHP classes.
- To reduce boilerplate code by embedding CRUD operations directly on the model.
- To enable rapid application development through conventions (table naming, primary keys, timestamps).
- To provide a fluent query builder for composing database queries in an object-oriented way.
- To support relationships between models through methods that return query builders (`hasMany`, `belongsTo`).

### Syntax Rules and Structure

**Complete General Syntax: Defining an Eloquent Model**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    // Optional: specify the table name if it differs from the pluralised class name
    protected $table = 'posts';

    // Optional: specify the primary key if it is not 'id'
    protected $primaryKey = 'post_id';

    // Optional: disable timestamps if the table lacks created_at/updated_at
    public $timestamps = false;

    // Columns that can be mass-assigned
    protected $fillable = ['title', 'body', 'user_id'];

    // Relationship: a post belongs to a user
    public function user()
    {
        return $this->belongsTo(User::class);
    }

    // Relationship: a post has many comments
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

**Component Breakdown:**

- `extends Model` — All Eloquent models must extend the base `Model` class. This is the defining characteristic of Active Record: the model inherits all persistence behaviour.
- `$table` — Specifies the database table. By default, Eloquent uses the snake_case plural of the class name (`Post` → `posts`).
- `$primaryKey` — Specifies the primary key column. Default is `id`.
- `$timestamps` — When `true` (default), Eloquent automatically manages `created_at` and `updated_at` columns.
- `$fillable` — An allow-list of columns that can be mass-assigned from user input. This prevents mass-assignment vulnerabilities.
- `user()` / `comments()` — Relationship methods. These return relationship objects that act as query builders.

**Complete General Syntax: CRUD Operations with Eloquent**

```php
// Create
$post = new Post();
$post->title = 'Hello World';
$post->body = 'This is my first post.';
$post->save();

// Or using mass assignment
$post = Post::create(['title' => 'Hello', 'body' => 'World']);

// Read
$post = Post::find(1);                    // Find by primary key
$posts = Post::where('user_id', 5)->get(); // Query builder
$post = Post::where('title', 'Hello')->first(); // First match

// Update
$post = Post::find(1);
$post->title = 'Updated Title';
$post->save();

// Or bulk update
Post::where('user_id', 5)->update(['active' => true]);

// Delete
$post = Post::find(1);
$post->delete();

// Or bulk delete
Post::where('active', false)->delete();
```

**Syntax Rules:**

- The model class name is singular and PascalCase (`User`, `Post`, `OrderItem`); Eloquent derives the table name by pluralising and snake_casing it (`users`, `posts`, `order_items`).
- The primary key is assumed to be `id` and auto-incrementing. Override `$primaryKey` and `$incrementing` if different.
- `$fillable` must be defined when using `create()` or `update()` with an array of attributes.
- Relationship methods return query builder instances, so they can be chained: `$user->posts()->where('published', true)->get()`.

**Constraints and Limitations:**

- **Coupled to the database:** Business logic can become persistence-aware, blurring boundaries in complex domains.
- **Fat models:** As rules grow, models accumulate validation, queries, events, and domain behaviour, becoming hard to maintain.
- **Testing friction:** True unit tests are trickier because many tests hit the database or require heavy mocking.
- **Portability limits:** Code tightly bound to an Active Record implementation can be harder to move to other patterns later.
- **N+1 queries:** Lazy loading relationships inside loops causes query explosion. Eager loading with `with()` is required.
- **No separation of concerns:** The model class is responsible for data, persistence, relationships, and often business logic — violating the Single Responsibility Principle.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Basic Active Record Model and CRUD**

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Capsule\Manager as Capsule;

// Step 1: Bootstrap Eloquent (standalone, without full Laravel)
require 'vendor/autoload.php';

$capsule = new Capsule;
$capsule->addConnection([
    'driver'    => 'sqlite',
    'database'  => ':memory:',
]);
$capsule->setAsGlobal();
$capsule->bootEloquent();

// Step 2: Create the schema
Capsule::schema()->create('products', function ($table) {
    $table->increments('id');
    $table->string('name');
    $table->decimal('price', 10, 2);
    $table->integer('stock')->default(0);
    $table->timestamps();
});

// Step 3: Define the Active Record model
class Product extends Model
{
    protected $fillable = ['name', 'price', 'stock'];
}

// Step 4: Create a record
$product = Product::create([
    'name'  => 'Wireless Keyboard',
    'price' => 49.99,
    'stock' => 120,
]);
echo "Created product: {$product->name} (ID: {$product->id})\n";

// Step 5: Read records
$found = Product::find($product->id);
echo "Found: {$found->name} — \${$found->price}\n";

// Step 6: Update a record
$found->price = 44.99;
$found->save();
echo "Updated price to \${$found->price}\n";

// Step 7: Delete a record
$found->delete();
echo "Product deleted. Remaining count: " . Product::count() . "\n";
```

**Expected Output:**

```
Created product: Wireless Keyboard (ID: 1)
Found: Wireless Keyboard — $49.99
Updated price to $44.99
Product deleted. Remaining count: 0
```

**Why:** The `Product` model extends `Model` and defines `$fillable`. `Product::create()` inserts a new row and returns a model instance. `find()` retrieves a row by primary key. Setting a property and calling `save()` updates the row. `delete()` removes the row. All persistence logic is embedded in the model itself.

**Example 2: Active Record Relationships**

```php
<?php

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['name', 'email'];

    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    protected $fillable = ['user_id', 'title', 'body'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}

// --- Usage ---
// Create a user with posts
$user = User::create(['name' => 'Alice', 'email' => 'alice@example.com']);

$post1 = $user->posts()->create(['title' => 'First Post', 'body' => 'Hello']);
$post2 = $user->posts()->create(['title' => 'Second Post', 'body' => 'World']);

// Navigate relationships
echo "User: {$user->name}\n";
echo "Posts: {$user->posts()->count()}\n";

foreach ($user->posts as $post) {
    echo "- {$post->title} by {$post->user->name}\n";
}
```

**Expected Output:**

```
User: Alice
Posts: 2
- First Post by Alice
- Second Post by Alice
```

**Why:** The `posts()` method on `User` returns a `hasMany` relationship. `$user->posts()->create()` creates a post with the foreign key automatically set. Accessing `$user->posts` (without parentheses) triggers lazy loading and returns a collection. `$post->user` navigates back to the parent user via the `belongsTo` relationship.

### Real-World Cases

**Case 1: Laravel Application Development**

Laravel's Eloquent is the default ORM for most Laravel applications. Developers define models, use `Model::create()` for inserts, `Model::find()` for reads, and relationship methods for joins. The Active Record pattern's conventions (table naming, timestamps, primary keys) dramatically reduce boilerplate code.

**Case 2: October CMS**

October CMS's database models are named "Active Record" and are based on Eloquent ORM provided by Laravel. The pattern's simplicity makes it well-suited for CMS content management, where each record (page, blog post, user) is a self-contained entity.

**Case 3: Rapid Prototyping and MVPs**

A startup building an MVP chooses Laravel + Eloquent because the Active Record pattern allows features to ship quickly with minimal ceremony. The team accepts the trade-off of potential future refactoring if the domain grows complex enough to require a Data Mapper architecture.

---

## Core Concept 3: Data Mapper Pattern (Doctrine)

### Definitions

**Core Definition**

The Data Mapper pattern is a Data Access Layer that performs bidirectional transfer of data between a persistent data store (often a relational database) and an in-memory data representation (the domain layer), keeping both independent of each other and of the mapper itself.

**Technical Definition**

Doctrine ORM is an object-relational mapper for PHP that provides transparent persistence for PHP objects. It uses the Data Mapper pattern at its heart, aiming for a complete separation of domain/business logic from persistence in a relational database management system. Entities are plain PHP objects that do not need to extend any abstract base class or interface. They contain persistable properties — instance variables that are saved into and retrieved from the database by Doctrine's data mapping capabilities via the **EntityManager**, an implementation of the Data Mapper pattern. The EntityManager is the central access point for persistence: it manages entity lifecycles, tracks changes (Unit of Work), and coordinates database operations. Repositories provide domain-specific query methods.

**Beginner-Friendly Explanation**

Think of a Data Mapper as a translator between two people who speak different languages. The domain object (the entity) speaks "PHP" with properties and methods. The database speaks "SQL" with columns and rows. The Data Mapper (EntityManager) translates back and forth. Neither the entity nor the database needs to know the other's language. The entity is a pure PHP object with no database knowledge; the database is managed entirely by the EntityManager.

### Purposes

- To keep domain objects free of persistence concerns — entities are Plain Old PHP Objects (POPOs) with no database knowledge.
- To isolate all database-specific code in the EntityManager and repositories, keeping entities clean.
- To enable the same domain object to be persisted to different storage systems (MySQL, PostgreSQL, MongoDB) by swapping the data mapper.
- To provide a complete separation of domain/business logic from persistence.
- To support complex mapping scenarios (inheritance, embedded value objects, collections) that Active Record cannot handle elegantly.
- To allow pure unit testing of domain logic without a database.

### Syntax Rules and Structure

**Complete General Syntax: Defining a Doctrine Entity**

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ProductRepository::class)]
#[ORM\Table(name: 'products')]
class Product
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 255)]
    private string $name;

    #[ORM\Column(type: 'decimal', precision: 10, scale: 2)]
    private string $price;

    #[ORM\Column(type: 'integer')]
    private int $stock = 0;

    #[ORM\ManyToOne(targetEntity: Category::class, inversedBy: 'products')]
    #[ORM\JoinColumn(nullable: false)]
    private Category $category;

    // Getters and setters...
    public function getId(): ?int { return $this->id; }
    public function getName(): string { return $this->name; }
    public function setName(string $name): self { $this->name = $name; return $this; }
    public function getPrice(): string { return $this->price; }
    public function setPrice(string $price): self { $this->price = $price; return $this; }
    public function getStock(): int { return $this->stock; }
    public function setStock(int $stock): self { $this->stock = $stock; return $this; }
    public function getCategory(): Category { return $this->category; }
    public function setCategory(Category $category): self { $this->category = $category; return $this; }
}
```

**Component Breakdown:**

- `#[ORM\Entity]` — Marks the class as a Doctrine entity (a persistable PHP object).
- `#[ORM\Table(name: 'products')]` — Maps the entity to the `products` table.
- `#[ORM\Id]` / `#[ORM\GeneratedValue]` — Marks the primary key and configures auto-generation.
- `#[ORM\Column(type: 'string')]` — Maps a property to a database column with a specific type.
- `#[ORM\ManyToOne]` — Configures a many-to-one relationship (many products belong to one category).
- `#[ORM\JoinColumn]` — Configures the foreign key column.
- The entity does **not** extend any base class and contains no `save()` or `delete()` methods. It is a pure PHP object.

**Complete General Syntax: Persistence with EntityManager**

```php
// Create
$product = new Product();
$product->setName('Wireless Keyboard');
$product->setPrice('49.99');
$product->setStock(120);
$entityManager->persist($product);
$entityManager->flush();

// Read
$product = $entityManager->find(Product::class, 1);
$products = $entityManager->getRepository(Product::class)->findBy(['stock' => 0]);

// Update
$product = $entityManager->find(Product::class, 1);
$product->setPrice('44.99');
$entityManager->flush();

// Delete
$entityManager->remove($product);
$entityManager->flush();
```

**Component Breakdown:**

- `$entityManager->persist($product)` — Registers a new entity with the Unit of Work. The entity is not yet inserted into the database.
- `$entityManager->flush()` — Synchronises all pending changes (inserts, updates, deletes) with the database. This is the point at which SQL is executed.
- `$entityManager->find(Product::class, 1)` — Retrieves an entity by primary key. Uses the Identity Map to return the same instance if already loaded.
- `$entityManager->getRepository(Product::class)` — Retrieves the repository for a specific entity, which provides domain-specific query methods.
- `$entityManager->remove($product)` — Marks an entity for deletion. The actual `DELETE` SQL is executed on `flush()`.

**Syntax Rules:**

- Entities must be registered with the EntityManager's mapping configuration (annotations, XML, or YAML).
- All entity properties must have getters and setters (or be public) for Doctrine to hydrate them.
- The EntityManager must be closed (`$entityManager->close()`) when the request ends, or it will leak memory.
- Changes are not persisted until `flush()` is called. This allows multiple related changes to be batched into a single transaction.
- Doctrine's Identity Map ensures that `find(Product::class, 1)` called twice returns the exact same object instance.

**Constraints and Limitations:**

- **More complex setup:** Requires configuration (mapping, proxies, cache) before it can be used.
- **Steeper learning curve:** Developers must understand the Unit of Work, Identity Map, and DQL (Doctrine Query Language).
- **No lazy loading by default in some contexts:** Lazy loading requires proxy objects, which can cause unexpected queries if not managed carefully.
- **Performance overhead:** The Unit of Work and change tracking add overhead. For simple CRUD, Active Record may be faster.
- **Schema synchronisation:** Doctrine can generate schemas from entities, but the process requires care in production (migrations, not direct schema updates).

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Basic Data Mapper Entity and Persistence**

```php
<?php

use Doctrine\ORM\Mapping as ORM;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools\Setup;

// Step 1: Bootstrap Doctrine (simplified)
require 'vendor/autoload.php';

$paths = [__DIR__ . '/src/Entity'];
$isDevMode = true;
$config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
$conn = ['driver' => 'pdo_sqlite', 'memory' => true];
$entityManager = EntityManager::create($conn, $config);

// Step 2: Define the entity (pure PHP object)
#[ORM\Entity]
#[ORM\Table(name: 'users')]
class User
{
    #[ORM\Id, ORM\GeneratedValue, ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string')]
    private string $name;

    #[ORM\Column(type: 'string', unique: true)]
    private string $email;

    // Getters and setters...
    public function getId(): ?int { return $this->id; }
    public function getName(): string { return $this->name; }
    public function setName(string $name): void { $this->name = $name; }
    public function getEmail(): string { return $this->email; }
    public function setEmail(string $email): void { $this->email = $email; }
}

// Step 3: Create a record
$user = new User();
$user->setName('Alice');
$user->setEmail('alice@example.com');

$entityManager->persist($user);  // Register with Unit of Work
$entityManager->flush();         // Execute INSERT

echo "Created user: {$user->getName()} (ID: {$user->getId()})\n";

// Step 4: Read a record
$found = $entityManager->find(User::class, $user->getId());
echo "Found: {$found->getName()} ({$found->getEmail()})\n";

// Step 5: Update a record
$found->setEmail('alice.new@example.com');
$entityManager->flush();  // Execute UPDATE

echo "Updated email to: {$found->getEmail()}\n";
```

**Expected Output:**

```
Created user: Alice (ID: 1)
Found: Alice (alice@example.com)
Updated email to: alice.new@example.com
```

**Why:** The `User` entity is a pure PHP object with no persistence methods. `persist()` registers the entity with the Unit of Work; `flush()` executes the SQL. The same object instance is returned by `find()` due to the Identity Map. Changes to `$found` are automatically tracked and persisted on `flush()`.

**Example 2: Data Mapper with Repository and DQL**

```php
<?php

use Doctrine\ORM\EntityRepository;

// Custom repository with domain-specific query methods
class ProductRepository extends EntityRepository
{
    public function findInStock(): array
    {
        return $this->createQueryBuilder('p')
            ->where('p.stock > 0')
            ->orderBy('p.name', 'ASC')
            ->getQuery()
            ->getResult();
    }

    public function findByName(string $name): ?Product
    {
        return $this->findOneBy(['name' => $name]);
    }
}

// --- Usage ---
$repository = $entityManager->getRepository(Product::class);

// Using standard repository methods
$allProducts = $repository->findAll();
$product = $repository->find(1);
$outOfStock = $repository->findBy(['stock' => 0]);

// Using custom repository methods
$inStock = $repository->findInStock();
$byName = $repository->findByName('Wireless Keyboard');
```

**Expected Output:** No visible output (these are method calls).

**Why:** The custom `ProductRepository` extends Doctrine's `EntityRepository` and adds domain-specific query methods (`findInStock`, `findByName`). The repository is the place for all query logic related to a specific entity, keeping the entity itself free of query concerns.

### Real-World Cases

**Case 1: Symfony Applications with Doctrine**

Symfony's default ORM is Doctrine. Entities are pure PHP objects, and the EntityManager is injected into services. Symfony's MakerBundle generates entities and repositories with the Data Mapper pattern, and the `doctrine/orm` package provides the full Data Mapper implementation.

**Case 2: Laravel Doctrine (Alternative to Eloquent)**

Some Laravel developers replace Eloquent with Laravel Doctrine, which offers a robust alternative by implementing the Data Mapper pattern. This provides better separation of concerns for complex domains where Active Record's fat models become unmanageable.

**Case 3: High-Performance APIs with Data Mapper**

A high-performance REST API uses Doctrine's Data Mapper pattern with DQL for complex read queries. The separation of entities from persistence allows the team to write pure unit tests for domain logic and use DQL for optimised read models.

---

## References

- Wendell Adriel: Understanding Laravel Eloquent's Active Record Pattern – https://expressive.wendelladriel.com/blog/understanding-laravel-eloquents-active-record-pattern
- Doctrine ORM: Getting Started – https://www.doctrine-project.org/projects/doctrine-orm/en/latest/tutorials/getting-started.html
- Laravel: Eloquent ORM — https://laravel.com/docs/eloquent
- Martin Fowler: Data Mapper – https://martinfowler.com/eaaCatalog/dataMapper.html
- Martin Fowler: Active Record – https://martinfowler.com/eaaCatalog/activeRecord.html
- Wikipedia: Active Record Pattern – http://en.wikipedia.org/wiki/Active_record_pattern
- PHP in Action: Objects and SQL (Impedance Mismatch) – https://livebook.manning.com/book/php-in-action/chapter-20
- Jesus Valera: ORM Data Mapper vs Active Record – https://github.com/JesusValeraDev/JesusValera.dev/blob/main/content/writing/2022-12-06-orm-data-mapper-vs-active-record.md
- Doctrine ORM: Basic Mapping – https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html
- Doctrine (PHP) — Wikipedia – https://en.wikipedia.org/wiki/Doctrine_(PHP)
- Sitepoint: Laravel Doctrine — Best of Both Worlds – https://www.sitepoint.com/laravel-doctrine-best-both-worlds/
- Jurnal Nasional Teknik Elektro dan Teknologi Informasi: Active Record vs Data Mapper Performance – https://jurnal.ugm.ac.id/v3/JNTETI/article/download/17315/5765/