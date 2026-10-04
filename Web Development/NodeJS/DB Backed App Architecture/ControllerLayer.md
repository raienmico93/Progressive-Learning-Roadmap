# Interface & Presentation Layer (The Controller) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The controller is the presentation-layer component that receives incoming requests, parses and validates their payloads, invokes application services, and translates the results (or errors) into HTTP-specific responses — serving as the boundary between the outside world and the application's business logic.

**Technical Definition:** In MVC and layered architectures, a Controller is a class (or set of functions) whose methods handle HTTP requests, extract and sanitise request data (body, query, params, headers, cookies), delegate work to application services, and return framework-appropriate responses. Controllers are the "glue" between the HTTP transport and the application layer — they know about HTTP (status codes, headers, cookies) but not about business rules, database queries, or domain invariants. The controller receives requests, validates inputs, delegates to the appropriate service, and returns a response. It should not contain business logic.

**Beginner-Friendly Explanation:** Imagine a restaurant again. The **controller** is the waiter who greets you at the door. They take your order (request parsing), check that you filled out the form correctly (validation), walk to the kitchen and hand the order to the chef (service invocation), then bring you your food with the right plates and utensils (HTTP response formatting). The waiter doesn't cook, doesn't invent recipes, and doesn't stock the pantry — they just make sure the customer's request reaches the right person and that the response comes back in a presentable way.

### Key Characteristics

- **Request parsing and validation:** Controllers extract and sanitise data (body, query, params, headers, cookies) before passing it to application services.
- **Thin by design:** Controllers contain no business rules — they orchestrate, not decide.
- **HTTP-aware:** Controllers know about status codes, headers, cookies, and content negotiation.
- **Error translation:** Controllers transform domain and application exceptions into structured HTTP responses (JSON, HTML, etc.).
- **Framework-specific:** Controllers are tightly coupled to their framework (Express, NestJS, Fastify, Koa, Spring).
- **Stateless:** Controllers hold no request-specific state between invocations.

### Prerequisites

- **HTTP fundamentals:** Methods, status codes, headers, request/response bodies.
- **Framework familiarity:** Express, NestJS, Fastify, or similar routing framework.
- **DTO and validation concepts:** `class-validator`, Joi, Zod, or manual validation.
- **Application service layer:** Understanding how services are invoked and what they return.
- **Error handling patterns:** Exception hierarchies and middleware pipelines.
- **Dependency injection:** Constructor injection (NestJS) or module wiring (Express).

### Related Programming Areas

- **HTTP and Web Protocols:** The controller is the primary consumer of HTTP semantics.
- **Application Service Layer:** Controllers invoke application services and map their outputs to HTTP responses.
- **DTOs and Validation:** Cross-layer data contracts ensure type-safe request parsing.
- **Middleware and Filters:** Cross-cutting concerns (auth, logging, CORS) are often implemented as middleware that runs before/after controllers.
- **Content Negotiation:** Controllers may respond with JSON, HTML, XML, or other formats based on `Accept` headers.
- **API Documentation:** OpenAPI/Swagger decorators are applied to controllers to generate documentation.

### Core Concepts

1. **Request Lifecycle Parsing** — extracting, sanitizing, and validating payload structures before they cross the threshold into application logic.
2. **Service Invocation** — orchestrating service calls while keeping controllers ultra-thin ("Thin Controller, Fat Service/Model" design).
3. **HTTP-Specific Concerns** — handling HTTP status codes, setting cookie states, managing header delivery, and transforming exceptions into structured JSON/HTML responses.

---

## Core Concept 1: Request Lifecycle Parsing

### Definitions

**Core Definition:** Request lifecycle parsing is the process of extracting data from an incoming HTTP request (body, query string, path parameters, headers, cookies), sanitizing it to remove harmful or unexpected content, and validating it against expected schemas before passing it to application services.

**Technical Definition:** Request parsing in a controller involves reading from the framework's request object — `req.body`, `req.query`, `req.params`, `req.headers`, `req.cookies` in Express; `@Body()`, `@Query()`, `@Param()`, `@Headers()` in NestJS — and transforming these raw inputs into validated, typed DTOs. Validation can occur at multiple stages: **framework-level** (body parser, query parser), **middleware-level** (Joi, Zod, class-validator via pipes), and **controller-level** (explicit checks). Sanitization includes trimming whitespace, normalising case, stripping HTML/script tags, and removing unexpected properties. Input validation is a critical security measure — never trust client input.

**Beginner-Friendly Explanation:** When someone sends a request to your API, they're handing you a form. The form might have missing fields, extra fields, wrong types, or even malicious content (like `<script>` tags). Request parsing is the process of reading the form, checking that everything is correct, throwing away anything suspicious, and converting it into a clean, typed object that your application can safely work with. It's like a bouncer at a club — only valid, safe data gets through the door.

### Purposes

- To extract structured data from raw HTTP requests (body, query, params, headers, cookies).
- To sanitize incoming data to prevent injection attacks (XSS, SQL injection, prototype pollution).
- To validate data against expected schemas before it reaches application logic.
- To convert untyped request data into typed DTOs for compile-time safety.
- To reject malformed requests with clear, structured error responses.
- To enforce size limits, character encodings, and content-type expectations.

### Syntax Rules and Structure

#### Express — Manual Parsing and Validation

```js
// routes/users.js
const express = require('express');
const Joi = require('joi');
const router = express.Router();

const createUserSchema = Joi.object({
  name: Joi.string().trim().min(1).max(100).required(),
  email: Joi.string().trim().lowercase().email().required(),
  age: Joi.number().integer().min(0).max(150).optional(),
});

router.post('/users', async (req, res, next) => {
  try {
    // 1. Extract from body (requires express.json() middleware)
    const raw = req.body;

    // 2. Validate and sanitize
    const { value, error } = createUserSchema.validate(raw, {
      abortEarly: false,   // Collect all errors, not just the first
      stripUnknown: true,  // Remove unknown properties
    });

    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details.map((d) => d.message),
      });
    }

    // 3. `value` is now a sanitized, validated DTO
    const user = await userService.create(value);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
});
```

| Component | Breakdown |
|-----------|-----------|
| `req.body` | Parsed JSON body (requires `express.json()` middleware). |
| `req.query` | Query string parameters (always strings). |
| `req.params` | URL path parameters. |
| `req.headers` | Request headers (lowercased by Node.js). |
| `req.cookies` | Parsed cookies (requires `cookie-parser` middleware). |
| `Joi.validate()` | Schema validation with `abortEarly`, `stripUnknown`. |

#### NestJS — Declarative Parsing via Decorators and Pipes

```typescript
// users/dto/create-user.dto.ts
import { IsString, IsEmail, IsOptional, IsInt, Min, Max, MaxLength, MinLength } from 'class-validator';
import { Transform } from 'class-transformer';

export class CreateUserDto {
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  @Transform(({ value }) => value?.trim())
  readonly name: string;

  @IsEmail()
  @Transform(({ value }) => value?.toLowerCase().trim())
  readonly email: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  @Max(150)
  readonly age?: number;
}
```

```typescript
// users/users.controller.ts
import { Controller, Post, Body, HttpCode, HttpStatus } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() dto: CreateUserDto): Promise<UserDto> {
    // dto is already validated and transformed by ValidationPipe
    return this.userService.create(dto);
  }
}
```

```typescript
// main.ts — Global ValidationPipe
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,             // Strip unknown properties
      forbidNonWhitelisted: true,  // Throw if unknown properties present
      transform: true,             // Auto-transform to DTO instances
      transformOptions: { enableImplicitConversion: true },
    }),
  );
  await app.listen(3000);
}
bootstrap();
```

| Component | Breakdown |
|-----------|-----------|
| `@Body()` | Binds and validates the request body. |
| `@Query()` | Binds and validates query parameters. |
| `@Param()` | Binds and validates route parameters. |
| `ValidationPipe` | Global pipe applying `class-validator` rules. |
| `whitelist: true` | Strips unknown properties from the DTO. |
| `forbidNonWhitelisted: true` | Throws 400 if unknown properties present. |
| `@Transform()` | Sanitization decorator (trim, lowercase). |

#### Syntax Rules

- **Body parsing middleware must be registered** before controllers (e.g., `express.json()`, `body-parser`).
- **Query parameters are always strings** — convert to numbers/booleans explicitly.
- **Path parameters are always strings** — parse to the expected type with validation.
- **Sanitization should happen before validation** — trim, lowercase, and strip HTML before checking schemas.
- **Unknown properties must be stripped or rejected** to prevent prototype pollution and mass-assignment attacks.
- **Content-Type must be checked** — reject unexpected content types early (`application/json`, `multipart/form-data`, etc.).
- **Size limits must be enforced** — configure `limit` on the body parser (default 100 KB in Express).

#### Constraints and Limitations

- **Raw query strings cannot express types** — `?age=25` arrives as the string `'25'`, not the number `25`.
- **Express does not validate automatically** — validation middleware or manual checks are required.
- **NestJS ValidationPipe relies on class-validator** — plain object validation libraries (Joi, Zod) require custom pipes.
- **Body parser is a middleware, not a controller concern** — but the controller is where the resulting DTO is received.
- **Sanitization cannot be perfectly automated** — context-sensitive sanitization (e.g., allowing HTML in a rich-text field) requires domain-specific rules.
- **Multipart form-data is not parsed by `express.json()`** — requires `multer` or `busboy`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Express — Full Request Parsing Pipeline

```js
// server.js
const express = require('express');
const cookieParser = require('cookie-parser');
const { body, query, param, validationResult } = require('express-validator');

const app = express();

// --- Middleware registration ---
app.use(express.json({ limit: '100kb' }));         // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));   // Parse URL-encoded bodies
app.use(cookieParser());                           // Parse cookies

// --- Validation chains ---
const createProductValidation = [
  body('name')
    .trim()
    .isString().withMessage('name must be a string')
    .isLength({ min: 1, max: 200 }).withMessage('name must be 1-200 characters')
    .escape(),                                     // HTML-escape to prevent XSS
  body('price')
    .isFloat({ min: 0.01 }).withMessage('price must be >= 0.01')
    .toFloat(),
  body('description')
    .optional()
    .trim()
    .isLength({ max: 2000 }),
  body('tags')
    .optional()
    .isArray({ max: 20 }).withMessage('tags must be an array of at most 20'),
];

// --- Controller ---
app.post('/products', createProductValidation, async (req, res, next) => {
  try {
    // 1. Check validation result
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({
        error: 'Validation failed',
        details: errors.array().map((e) => ({
          field: e.path,
          message: e.msg,
        })),
      });
    }

    // 2. Extract sanitized data (express-validator mutates req.body in place)
    const dto = {
      name: req.body.name,
      price: req.body.price,
      description: req.body.description,
      tags: req.body.tags ?? [],
    };

    // 3. Delegate to service
    const product = await productService.create(dto);

    // 4. Return response
    res.status(201).json({
      id: product.id,
      name: product.name,
      price: product.price,
    });
  } catch (err) {
    next(err);
  }
});

// --- Error handler ---
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status ?? 500).json({
    error: err.message ?? 'Internal Server Error',
  });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Step-by-step setup:**
1. Register `express.json()` with a 100 KB size limit — prevents oversized payloads.
2. Register `express.urlencoded()` for form submissions.
3. Register `cookieParser()` for cookie access.
4. Define validation chains with `express-validator` — trim, length, type, and escape.
5. In the controller, check `validationResult(req)` — reject with 422 if any errors.
6. Extract the sanitized data and delegate to the application service.
7. Return a structured JSON response with the created resource.

**Expected behaviour:** Sending `POST /products` with `{ "name": "  Widget  ", "price": "19.99", "tags": ["a", "b"] }` produces a product with trimmed name `"Widget"` and numeric price `19.99`. Sending `{ "name": "", "price": -5 }` produces a `422` response listing both validation errors.

**Why this works:** Validation happens *before* the controller body executes. Sanitization (trim, escape) modifies the values in place, so the service receives clean data. The service is never called with invalid input.

#### Example 2: NestJS — DTO Validation, Transformation, and Custom Sanitization

```typescript
// users/dto/create-user.dto.ts
import {
  IsString, IsEmail, IsOptional, IsInt, IsEnum, Min, Max,
  MaxLength, MinLength, Matches, ValidateNested, IsArray, ArrayMaxSize,
} from 'class-validator';
import { Type, Transform } from 'class-transformer';

export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  MODERATOR = 'moderator',
}

export class AddressDto {
  @IsString()
  @MaxLength(200)
  @Transform(({ value }) => value?.trim())
  readonly street: string;

  @IsString()
  @Matches(/^[A-Z]{2}$/, { message: 'state must be a 2-letter code' })
  @Transform(({ value }) => value?.toUpperCase())
  readonly state: string;

  @IsString()
  @Matches(/^\d{5}$/, { message: 'zip must be 5 digits' })
  readonly zip: string;
}

export class CreateUserDto {
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  @Transform(({ value }) => value?.trim())
  readonly name: string;

  @IsEmail()
  @Transform(({ value }) => value?.toLowerCase().trim())
  readonly email: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  @Max(150)
  @Type(() => Number)
  readonly age?: number;

  @IsEnum(UserRole)
  @IsOptional()
  readonly role?: UserRole;

  @IsOptional()
  @IsArray()
  @ArrayMaxSize(5)
  @ValidateNested({ each: true })
  @Type(() => AddressDto)
  readonly addresses?: AddressDto[];

  @IsString()
  @Matches(/^(?=.*[A-Z])(?=.*\d).{8,}$/, {
    message: 'password must be 8+ chars with at least one uppercase letter and one digit',
  })
  readonly password: string;
}
```

```typescript
// users/users.controller.ts
import {
  Controller, Post, Body, HttpCode, HttpStatus, Get, Param, Query,
  ParseUUIDPipe, UsePipes, ValidationPipe, Patch, Delete,
} from '@nestjs/common';
import { CreateUserDto, UpdateUserDto, UserRole } from './dto';
import { UsersService } from './users.service';
import { UserDto } from './dto/user.dto';

@Controller('users')
@UsePipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }))
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() dto: CreateUserDto): Promise<UserDto> {
    return this.usersService.create(dto);
  }

  @Get(':id')
  async findById(
    @Param('id', new ParseUUIDPipe({ version: '4' })) id: string,
  ): Promise<UserDto> {
    return this.usersService.findById(id);
  }

  @Get()
  async findAll(
    @Query('role') role?: UserRole,
    @Query('limit', new ParseIntPipe({ optional: true })) limit = 20,
    @Query('offset', new ParseIntPipe({ optional: true })) offset = 0,
  ): Promise<{ data: UserDto[]; total: number }> {
    return this.usersService.findMany({ role, limit, offset });
  }

  @Patch(':id')
  async update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() dto: UpdateUserDto,
  ): Promise<UserDto> {
    return this.usersService.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id', ParseUUIDPipe) id: string): Promise<void> {
    await this.usersService.delete(id);
  }
}
```

**Expected behaviour:**
- `POST /users` with valid JSON creates the user and returns `201 Created`.
- `POST /users` with an invalid email returns `400 Bad Request` with field-level error messages.
- `GET /users/not-a-uuid` returns `400 Bad Request` because `ParseUUIDPipe` rejects it before the handler executes.
- `GET /users?limit=abc` returns `400 Bad Request` because `ParseIntPipe` rejects non-numeric input.
- Unknown properties in the body are stripped (`whitelist: true`) or rejected (`forbidNonWhitelisted: true`).

**Why this works:** NestJS applies the `ValidationPipe` globally or at the controller level. It uses `class-transformer` to convert plain objects to DTO instances (enabling `@Transform` and `@Type`) and `class-validator` to validate them. Pipes like `ParseUUIDPipe` and `ParseIntPipe` handle parameter transformation and validation at the route level, so the handler never runs with invalid input.

### Real-World Cases

- **User registration APIs:** Validating email format, password strength, and required fields before creating an account.
- **E-commerce checkout:** Parsing and validating cart items, shipping addresses, and payment details before processing.
- **File upload endpoints:** Validating MIME types, file sizes, and sanitising filenames to prevent path traversal.
- **Multi-tenant APIs:** Extracting and validating tenant IDs from headers or path parameters before routing.
- **GraphQL resolvers:** Using input types and validation decorators to parse resolver arguments.

---

## Core Concept 2: Service Invocation (Thin Controller, Fat Service/Model)

### Definitions

**Core Definition:** Service invocation is the controller's act of delegating business work to application services while keeping the controller itself thin — no business rules, no database queries, and no complex logic. The "Thin Controller, Fat Service/Model" principle states that controllers should only orchestrate: parse, delegate, respond.

**Technical Definition:** A thin controller has minimal logic: it receives the request, invokes one (or a few) service methods, and returns the result. Business rules live in the application service or, ideally, in domain entities (the "Fat Model"). This separation ensures that the same business logic can be invoked from multiple entry points (HTTP, CLI, queue worker, tests) without duplication. Controllers are stateless, framework-specific, and disposable — the service and domain layers are the durable core of the application.

**Beginner-Friendly Explanation:** Think of a controller as a receptionist. They take your request, route it to the right department, and hand you the answer. They don't fix the problem themselves. The "Fat Service/Model" is the department that actually does the work. If you swap the receptionist (change from Express to Fastify, or add a CLI), the department doesn't change. That's the power of thin controllers.

### Purposes

- To keep business logic in one place (application/domain services) so it can be reused across delivery mechanisms.
- To make controllers easy to test — a thin controller can be tested with mock services without a database.
- To reduce duplication of business rules across HTTP, CLI, GraphQL, and queue entry points.
- To keep controllers stateless and focused on their single responsibility: HTTP orchestration.
- To make the application resilient to framework changes — swapping Express for Fastify only touches the controller layer.

### Syntax Rules and Structure

#### Thin Controller (Express)

```js
// controllers/userController.js
const userService = require('../services/userService');

async function createUser(req, res, next) {
  try {
    // Extract validated DTO from middleware
    const dto = req.validated;

    // Delegate to service — one call, no business logic
    const user = await userService.create(dto);

    // Respond
    res.status(201).json(user);
  } catch (err) {
    next(err);   // Delegate error handling to middleware
  }
}

module.exports = { createUser };
```

#### Fat Service (Express)

```js
// services/userService.js
const bcrypt = require('bcrypt');
const User = require('../models/User');
const { sendWelcomeEmail } = require('../email');

async function create(dto) {
  // Business rule: email must be unique
  const existing = await User.findByEmail(dto.email);
  if (existing) {
    const err = new Error('Email already registered');
    err.status = 409;
    err.code = 'EMAIL_TAKEN';
    throw err;
  }

  // Business rule: password must be hashed
  const passwordHash = await bcrypt.hash(dto.password, 12);

  // Persist
  const user = await User.create({ ...dto, passwordHash });

  // Side effect
  await sendWelcomeEmail(user.email);

  return user.toPublicJSON();
}
```

#### Thin Controller (NestJS)

```typescript
// users/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() dto: CreateUserDto): Promise<UserDto> {
    return this.usersService.create(dto);   // One-line delegation
  }

  @Get(':id')
  async findById(@Param('id', ParseUUIDPipe) id: string): Promise<UserDto> {
    return this.usersService.findById(id);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id', ParseUUIDPipe) id: string): Promise<void> {
    await this.usersService.delete(id);
  }
}
```

| Controller Responsibility | Service Responsibility |
|---------------------------|------------------------|
| Extract and validate request data | Enforce business rules |
| Call one or more service methods | Coordinate repositories and transactions |
| Set HTTP status codes | Manage domain invariants |
| Set headers and cookies | Dispatch side effects (emails, events) |
| Transform service results into HTTP responses | Return domain objects or DTOs |
| Delegate errors to middleware | Throw domain-specific exceptions |

#### Syntax Rules

- **One service call per controller method** (usually). If you need multiple, the logic likely belongs in a service.
- **No business conditionals** in the controller — no `if (user.role === 'admin')` checks that decide outcomes.
- **No direct database access** — controllers never import repositories or models.
- **No `try/catch` blocks that swallow errors** — use `next(err)` (Express) or let exceptions bubble (NestJS).
- **No data transformation beyond DTO mapping** — if the response needs reshaping, do it in the service or a mapper.
- **Controller methods should be < 10 lines** — if longer, business logic is probably leaking in.

#### Constraints and Limitations

- **Thin does not mean zero** — controllers still need to handle HTTP concerns (status codes, cookies, headers).
- **Over-thinning can be a smell** — a controller that does nothing but `return this.service.create(dto)` may indicate the service is doing too much.
- **Some frameworks encourage thick controllers** — e.g., Ruby on Rails, where controllers often contain ActiveRecord queries. In such frameworks, thin controllers require extra discipline.
- **Error mapping is often in the controller** — determining the correct HTTP status from a domain error may be a controller or filter responsibility.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Thin Controller vs. Fat Controller (Express)

```js
// ❌ FAT CONTROLLER — business logic in the controller
router.post('/orders', async (req, res) => {
  // Business rule: check stock
  const product = await db.products.findById(req.body.productId);
  if (!product || product.stock < req.body.quantity) {
    return res.status(409).json({ error: 'Insufficient stock' });
  }

  // Business rule: calculate total
  const total = product.price * req.body.quantity;
  const tax = total * 0.08;
  const grandTotal = total + tax;

  // Business rule: apply discount if user is premium
  const user = await db.users.findById(req.user.id);
  const discount = user.isPremium ? grandTotal * 0.1 : 0;
  const finalTotal = grandTotal - discount;

  // Persist
  const order = await db.orders.create({
    userId: req.user.id,
    productId: product.id,
    quantity: req.body.quantity,
    total: finalTotal,
  });

  // Decrement stock
  await db.products.decrementStock(product.id, req.body.quantity);

  res.status(201).json(order);
});
```

```js
// ✅ THIN CONTROLLER + FAT SERVICE
router.post('/orders', async (req, res, next) => {
  try {
    const dto = req.validated; // Populated by validation middleware
    const order = await orderService.createOrder(req.user.id, dto);
    res.status(201).json(order);
  } catch (err) {
    next(err);
  }
});

// services/orderService.js
async function createOrder(userId, dto) {
  return unitOfWork.transaction(async (manager) => {
    const product = await productRepository.findById(dto.productId, manager);
    if (!product || product.stock < dto.quantity) {
      throw new InsufficientStockError(product?.id);
    }

    const user = await userRepository.findById(userId, manager);
    const total = pricingService.calculateTotal(product, dto.quantity, user);

    product.decrementStock(dto.quantity);
    await productRepository.save(product, manager);

    const order = Order.create({ userId, productId: product.id, quantity: dto.quantity, total });
    await orderRepository.save(order, manager);

    return OrderMapper.toDto(order);
  });
}
```

**Expected behaviour:** The controller is now 5 lines. The service contains all business logic (stock check, pricing, discount, persistence) inside a transaction. Testing the pricing logic requires no HTTP request — just call `orderService.createOrder()`.

**Why this works:** The controller has a single responsibility: HTTP handling. The service has a single responsibility: business logic. Changes to pricing rules don't touch the controller. Changes to HTTP frameworks don't touch the service.

#### Example 2: NestJS — Thin Controller with Service Orchestration

```typescript
// articles/articles.controller.ts
@Controller('articles')
export class ArticlesController {
  constructor(private readonly articlesService: ArticlesService) {}

  @Get()
  async list(
    @Query() query: ListArticlesQueryDto,
    @CurrentUser() user: AuthUser,
  ): Promise<PaginatedDto<ArticleDto>> {
    return this.articlesService.list(query, user);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(
    @Body() dto: CreateArticleDto,
    @CurrentUser() user: AuthUser,
  ): Promise<ArticleDto> {
    return this.articlesService.create(dto, user);
  }

  @Patch(':id')
  async update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() dto: UpdateArticleDto,
    @CurrentUser() user: AuthUser,
  ): Promise<ArticleDto> {
    return this.articlesService.update(id, dto, user);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(
    @Param('id', ParseUUIDPipe) id: string,
    @CurrentUser() user: AuthUser,
  ): Promise<void> {
    await this.articlesService.remove(id, user);
  }
}
```

```typescript
// articles/articles.service.ts
@Injectable()
export class ArticlesService {
  constructor(
    private readonly articleRepository: ArticleRepository,
    private readonly unitOfWork: UnitOfWork,
    private readonly authorizationService: AuthorizationService,
    private readonly slugService: SlugService,
  ) {}

  async create(dto: CreateArticleDto, user: AuthUser): Promise<ArticleDto> {
    await this.authorizationService.assertCanCreateArticle(user);

    const slug = await this.slugService.generateUniqueSlug(dto.title);

    return this.unitOfWork.transaction(async (manager) => {
      const article = Article.create({
        title: dto.title,
        body: dto.body,
        slug,
        authorId: user.id,
      });
      await this.articleRepository.save(article, manager);
      return ArticleMapper.toDto(article);
    });
  }

  async list(query: ListArticlesQueryDto, user: AuthUser): Promise<PaginatedDto<ArticleDto>> {
    const criteria = ArticlesCriteria.fromQuery(query).withAuthorVisibility(user);
    const { items, total } = await this.articleRepository.findByCriteria(criteria);
    return {
      data: items.map(ArticleMapper.toDto),
      total,
      limit: query.limit,
      offset: query.offset,
    };
  }
}
```

**Expected behaviour:** All controller methods are one-liners. Business logic (slug generation, authorization, transaction, criteria building) lives in the service layer. Testing `ArticlesService.create()` requires no HTTP mocking — just mock the repository and authorization service.

**Why this works:** The controller is a pure HTTP adapter. Every use case is a single service method. The service layer is where all the orchestration happens.

### Real-World Cases

- **REST APIs with multiple clients:** A single `UsersService.create()` is called by the HTTP controller, a CLI command, a GraphQL resolver, and a queue worker.
- **Microservices:** Controllers receive messages from the API gateway and delegate to local services; the same services can be invoked by internal RPC handlers.
- **Serverless functions:** Lambda handlers are thin controllers that call services; the same services run in a long-lived server.
- **Testing:** Thin controllers are easy to unit test; the service layer is tested separately with mocked repositories.

---

## Core Concept 3: HTTP-Specific Concerns

### Definitions

**Core Definition:** HTTP-specific concerns are the transport-level responsibilities of a controller: setting correct status codes, managing response headers and cookies, handling content negotiation, and transforming exceptions into structured HTTP error responses.

**Technical Definition:** Controllers are the only layer that should know about HTTP. They map application results to status codes (`200`, `201`, `204`, `400`, `404`, `409`, `422`, `500`), set headers (`Content-Type`, `Location`, `Cache-Control`, `Set-Cookie`), manage cookie state (`HttpOnly`, `Secure`, `SameSite`, `Max-Age`), and translate application/domain exceptions into appropriate HTTP responses. In frameworks like NestJS, exception filters and interceptors centralise this translation; in Express, error-handling middleware plays the same role.

**Beginner-Friendly Explanation:** The controller is the translator between your application and the outside world. Your service says "I created a user" — the controller translates that to `201 Created` with a `Location` header. Your service throws a `UserNotFoundError` — the controller translates that to `404 Not Found` with a JSON body explaining the error. This keeps HTTP specifics out of your business logic, and business logic out of your HTTP layer.

### Purposes

- To set appropriate HTTP status codes that accurately reflect the outcome of the operation.
- To deliver headers that control caching, content type, security, and redirection.
- To manage cookie state for authentication, sessions, and user preferences.
- To transform application and domain exceptions into structured, client-friendly error responses.
- To negotiate content types based on the `Accept` header (JSON, HTML, XML).
- To ensure security headers (CORS, CSP, HSTS) are applied consistently.

### Syntax Rules and Structure

#### HTTP Status Code Categories

| Range | Category | Examples |
|-------|----------|----------|
| `1xx` | Informational | `100 Continue`, `101 Switching Protocols` |
| `2xx` | Success | `200 OK`, `201 Created`, `204 No Content` |
| `3xx` | Redirection | `301 Moved Permanently`, `302 Found`, `304 Not Modified` |
| `4xx` | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests` |
| `5xx` | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` |

#### Common Response Headers

| Header | Purpose |
|--------|---------|
| `Content-Type` | Media type of the response body (`application/json; charset=utf-8`). |
| `Content-Length` | Size of the response body in bytes. |
| `Location` | URL of a newly created resource (with `201 Created`). |
| `Cache-Control` | Caching directives (`no-store`, `max-age=3600`). |
| `ETag` | Resource version identifier for conditional requests. |
| `Set-Cookie` | Sets one or more cookies. |
| `Access-Control-Allow-Origin` | CORS: which origins may read the response. |
| `Strict-Transport-Security` | HSTS: enforce HTTPS for future requests. |
| `Content-Security-Policy` | CSP: restrict sources of scripts, styles, etc. |

#### Cookie Flags

| Flag | Purpose |
|------|---------|
| `HttpOnly` | Cookie is inaccessible to JavaScript (mitigates XSS). |
| `Secure` | Cookie is only sent over HTTPS. |
| `SameSite=Strict` | Cookie is not sent on cross-site requests (mitigates CSRF). |
| `SameSite=Lax` | Cookie is sent on top-level navigations but not on cross-site subrequests. |
| `SameSite=None` | Cookie is sent on all requests (requires `Secure`). |
| `Max-Age` / `Expires` | Cookie lifetime. |
| `Domain` / `Path` | Scope of the cookie. |

#### Syntax Rules

- **Use the correct status code** — `201` for creation, `204` for deletion without a body, `404` for missing resources, `409` for conflicts, `422` for validation errors.
- **Never return a 200 with an error in the body** — this breaks clients that rely on status codes.
- **Always set `Content-Type`** — clients and proxies rely on it.
- **Cookies must use `HttpOnly` and `Secure`** for authentication tokens.
- **Use `SameSite=Lax` or `Strict`** for session cookies; `SameSite=None; Secure` only when cross-site is required.
- **Security headers must be applied globally** — via middleware or a global filter/interceptor.
- **Exceptions should map to status codes** via a centralised error-handling mechanism.

#### Constraints and Limitations

- **Status codes are finite and sometimes ambiguous** — is `404` a missing resource or a missing route?
- **Cookie size is limited** — browsers cap cookies at ~4 KB each and ~50 per domain.
- **Header injection** must be prevented — never interpolate untrusted input into headers.
- **`Content-Length` must match the body size** — mismatches cause hangs or truncation.
- **CORS preflight requests** (`OPTIONS`) must be handled explicitly by middleware.
- **Cookie `SameSite=None` requires `Secure`** — otherwise browsers reject the cookie.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Express — Status Codes, Cookies, and Error Handling

```js
// server.js
const express = require('express');
const cookieParser = require('cookie-parser');
const helmet = require('helmet');

const app = express();

// Security headers (CSP, HSTS, X-Frame-Options, etc.)
app.use(helmet());

// CORS (example — replace with cors package for production)
app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', process.env.CORS_ORIGIN || 'https://app.example.com');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PATCH,DELETE,OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type,Authorization');
  if (req.method === 'OPTIONS') return res.status(204).end();
  next();
});

app.use(express.json({ limit: '100kb' }));
app.use(cookieParser());

// --- Auth controller: setting cookies ---
app.post('/auth/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    const { user, accessToken, refreshToken } = await authService.login(email, password);

    // Set refresh token as an HttpOnly, Secure, SameSite cookie
    res.cookie('refresh_token', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 7 * 24 * 60 * 60 * 1000,   // 7 days
      path: '/auth/refresh',
    });

    // Return access token in the body (short-lived, client stores in memory)
    res.status(200).json({
      user: user.toPublicJSON(),
      accessToken,
    });
  } catch (err) {
    next(err);
  }
});

// --- Resource controller: Location header on creation ---
app.post('/articles', async (req, res, next) => {
  try {
    const dto = req.validated;
    const article = await articleService.create(dto, req.user);

    // 201 Created with Location header
    res
      .status(201)
      .set('Location', `/articles/${article.id}`)
      .json(article);
  } catch (err) {
    next(err);
  }
});

// --- Resource controller: 204 No Content on delete ---
app.delete('/articles/:id', async (req, res, next) => {
  try {
    await articleService.delete(req.params.id, req.user);
    res.status(204).end();   // No body
  } catch (err) {
    next(err);
  }
});

// --- Central error handler: transforms exceptions to HTTP responses ---
app.use((err, req, res, next) => {
  // Map domain errors to HTTP status codes
  const statusMap = {
    ValidationError: 422,
    NotFoundError: 404,
    ConflictError: 409,
    UnauthorizedError: 401,
    ForbiddenError: 403,
    RateLimitError: 429,
  };

  const status = err.status ?? statusMap[err.name] ?? 500;

  // Do not leak internal error details in production
  const body = {
    error: err.code ?? err.name ?? 'InternalServerError',
    message: process.env.NODE_ENV === 'production' && status >= 500
      ? 'Internal Server Error'
      : err.message,
  };

  if (err.details) body.details = err.details;

  res.status(status).json(body);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `POST /auth/login` on success sets a `refresh_token` cookie with `HttpOnly; Secure; SameSite=Lax` and returns `200` with the access token.
- `POST /articles` on success returns `201` with a `Location: /articles/123` header.
- `DELETE /articles/123` returns `204 No Content` with no body.
- `POST /articles` with a duplicate slug returns `409 Conflict`.
- An unexpected `TypeError` in a service returns `500 Internal Server Error` with a sanitised message in production.

**Why this works:** The controller sets the HTTP-specific parts (status, headers, cookies). The error handler maps domain exception names to HTTP status codes. The service never sees `res` and never returns status codes.

#### Example 2: NestJS — Exception Filters and Interceptors

```typescript
// common/filters/domain-exception.filter.ts
import {
  ExceptionFilter, Catch, ArgumentsHost, HttpStatus, Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

// Domain exceptions
export class ValidationError extends Error {
  constructor(message: string, public readonly details?: Array<{ field: string; message: string }>) {
    super(message);
    this.name = 'ValidationError';
  }
}
export class NotFoundError extends Error {
  constructor(message = 'Resource not found') { super(message); this.name = 'NotFoundError'; }
}
export class ConflictError extends Error {
  constructor(message = 'Conflict') { super(message); this.name = 'ConflictError'; }
}
export class ForbiddenError extends Error {
  constructor(message = 'Forbidden') { super(message); this.name = 'ForbiddenError'; }
}

@Catch()
export class DomainExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(DomainExceptionFilter.name);

  catch(exception: Error, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const { status, body } = this.mapException(exception);

    if (status >= 500) {
      this.logger.error(
        `${request.method} ${request.url} -> ${status}`,
        exception.stack,
      );
    }

    response.status(status).json({
      ...body,
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }

  private mapException(exception: Error): { status: number; body: Record<string, unknown> } {
    switch (exception.name) {
      case 'ValidationError':
        return {
          status: HttpStatus.UNPROCESSABLE_ENTITY,
          body: {
            error: 'ValidationError',
            message: exception.message,
            details: (exception as ValidationError).details,
          },
        };
      case 'NotFoundError':
        return {
          status: HttpStatus.NOT_FOUND,
          body: { error: 'NotFoundError', message: exception.message },
        };
      case 'ConflictError':
        return {
          status: HttpStatus.CONFLICT,
          body: { error: 'ConflictError', message: exception.message },
        };
      case 'ForbiddenError':
        return {
          status: HttpStatus.FORBIDDEN,
          body: { error: 'ForbiddenError', message: exception.message },
        };
      default:
        return {
          status: HttpStatus.INTERNAL_SERVER_ERROR,
          body: {
            error: 'InternalServerError',
            message: process.env.NODE_ENV === 'production'
              ? 'Internal Server Error'
              : exception.message,
          },
        };
    }
  }
}
```

```typescript
// main.ts — register globally
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { DomainExceptionFilter } from './common/filters/domain-exception.filter';
import helmet from 'helmet';
import cookieParser from 'cookie-parser';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.use(helmet());
  app.use(cookieParser());
  app.enableCors({
    origin: process.env.CORS_ORIGIN?.split(',') ?? true,
    credentials: true,
  });

  app.useGlobalFilters(new DomainExceptionFilter());

  await app.listen(3000);
}
bootstrap();
```

```typescript
// auth/auth.controller.ts
import { Controller, Post, Body, Res, HttpCode, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('login')
  @HttpCode(HttpStatus.OK)
  async login(
    @Body() dto: LoginDto,
    @Res({ passthrough: true }) res: Response,
  ): Promise<{ accessToken: string; user: UserDto }> {
    const { user, accessToken, refreshToken } = await this.authService.login(dto);

    res.cookie('refresh_token', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 7 * 24 * 60 * 60 * 1000,
      path: '/auth/refresh',
    });

    return { accessToken, user };
  }

  @Post('logout')
  @HttpCode(HttpStatus.NO_CONTENT)
  async logout(@Res({ passthrough: true }) res: Response): Promise<void> {
    res.clearCookie('refresh_token', { path: '/auth/refresh' });
  }
}
```

**Expected behaviour:**
- Successful login sets the refresh cookie and returns `200` with the access token.
- A `NotFoundError` thrown by a service results in `404 Not Found` with a structured JSON body.
- A `ValidationError` thrown by a service results in `422 Unprocessable Entity` with field-level details.
- A raw `TypeError` results in `500 Internal Server Error` with a sanitised message.
- All error responses include `path` and `timestamp` for debugging.

**Why this works:** The exception filter centralises the mapping between domain exceptions and HTTP status codes. The controller only sets HTTP-specific things (cookies, status). All other error handling is uniform across the application.

### Real-World Cases

- **Authentication flows:** Setting `HttpOnly; Secure; SameSite` cookies for refresh tokens, returning access tokens in the body.
- **REST API error handling:** Mapping `NotFoundError` to `404`, `ConflictError` to `409`, `ValidationError` to `422`.
- **File upload responses:** Returning `201 Created` with a `Location` header pointing to the uploaded resource.
- **Cache control:** Setting `Cache-Control: public, max-age=3600` for static assets; `no-store` for authenticated responses.
- **CORS:** Handling preflight `OPTIONS` requests and setting `Access-Control-Allow-*` headers.
- **Security headers:** Applying HSTS, CSP, and X-Content-Type-Options via `helmet` or equivalent middleware.

---

## References

- MDN Web Docs — HTTP Response Status Codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- MDN Web Docs — HTTP Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers
- MDN Web Docs — Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN Web Docs — SameSite Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- MDN Web Docs — Content Security Policy (CSP) — https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN Web Docs — Strict-Transport-Security (HSTS) — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- MDN Web Docs — Cross-Origin Resource Sharing (CORS) — https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 — HTTP Caching — https://www.rfc-editor.org/rfc/rfc9111
- RFC 6265 — HTTP State Management Mechanism (Cookies) — https://www.rfc-editor.org/rfc/rfc6265
- NestJS Documentation — Controllers — https://docs.nestjs.com/controllers
- NestJS Documentation — Validation — https://docs.nestjs.com/techniques/validation
- NestJS Documentation — Exception Filters — https://docs.nestjs.com/exception-filters
- NestJS Documentation — Interceptors — https://docs.nestjs.com/interceptors
- Express Documentation — Routing — https://expressjs.com/en/guide/routing.html
- Express Documentation — Error Handling — https://expressjs.com/en/guide/error-handling.html
- express-validator Documentation — https://express-validator.github.io/docs/
- class-validator Documentation — https://github.com/typestack/class-validator
- Helmet Documentation — https://helmetjs.github.io/
- OWASP Cheat Sheet Series — Input Validation — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross-Site Scripting Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html