# Interactive Documentation & UI Tooling — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Interactive documentation and UI tooling refers to the libraries, middleware, and front-end applications that render an OpenAPI/Swagger specification as a live, browsable, and testable API reference directly inside an Express application.

**Technical Definition:** This domain encompasses two primary rendering engines — Swagger UI and ReDoc — delivered as Express middleware (`swagger-ui-express` and `redoc-express` respectively). Swagger UI is an API console that renders OpenAPI definitions as interactive documentation with a built-in "Try it out" sandbox for executing live HTTP requests against the API. ReDoc is a documentation-first renderer that produces a three-panel, responsive API reference optimised for readability and navigation, without an interactive console. Both consume a valid OpenAPI 3.x or Swagger 2.0 document and serve a single-page application from a route within the Express server.

**Beginner-Friendly Explanation:** When you write an OpenAPI specification for your API, it's just a file describing your endpoints — it's not something humans can easily read. Interactive documentation tools take that file and turn it into a beautiful, clickable website. Swagger UI gives you a "Try it out" button so you can actually make API calls from the documentation page. ReDoc gives you a clean, three-column layout that's easier to read but doesn't let you execute requests. Both are served directly from your Express server, so your docs are always in sync with your API.

### Key Characteristics

- **Single source of truth:** Both tools render from the same OpenAPI specification file, ensuring documentation matches the API contract.
- **Zero build step:** `swagger-ui-express` and `redoc-express` are middleware — no separate build pipeline is required.
- **Swagger UI is an API console:** It prioritises interactivity with the "Try it out" feature for live API calls.
- **ReDoc is documentation-first:** It prioritises readability with a responsive three-panel layout and menu/scrolling synchronisation.
- **Theming support:** Both tools support custom CSS/JS (Swagger UI) and a full theming API (ReDoc) for branding.
- **CSP-aware:** Both middleware packages support Content Security Policy nonces for secure deployments.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express`.
- **A valid OpenAPI 3.x or Swagger 2.0 specification** (JSON or YAML).
- **For Swagger UI:** `npm install swagger-ui-express`.
- **For ReDoc:** `npm install redoc-express`.

### Related Programming Areas

- **OpenAPI Specification:** The document format that both tools render.
- **API Documentation:** The practice of describing APIs for consumers.
- **API Testing:** Swagger UI's "Try it out" feature enables manual testing.
- **API Gateways:** Many gateways import OpenAPI documents for validation and routing.
- **Developer Portals:** Interactive docs are a core component of developer experience.

### Core Concepts

1. **swagger-ui-express** — Hosting interactive docs inside Express.
2. **redoc-express** — Clean, three-panel static API documentation alternative.
3. **API Exploration and Sandboxing** — The "Try it out" feature.
4. **Customising UI Themes and Branding** — Themes, CSS, and logos.

---

## Core Concept 1: swagger-ui-express — Hosting Interactive Docs Inside Express

### Definitions

**Core Definition:** `swagger-ui-express` is an Express middleware that serves the Swagger UI front-end application, bound to a Swagger/OpenAPI document, from a route within your Express server.

**Technical Definition:** `swagger-ui-express` exposes two middleware functions: `swaggerUi.serve` (which serves the static Swagger UI assets from `swagger-ui-dist`) and `swaggerUi.setup(swaggerDocument, options)` (which generates the HTML page bound to the provided specification). The resulting route renders a fully interactive API console with expandable operations, request/response schemas, and the "Try it out" sandbox.

**Beginner-Friendly Explanation:** `swagger-ui-express` is the simplest way to add interactive API documentation to an Express app. You install it, point it at your `swagger.json` file, and visit a URL like `/api-docs` — and you get a full Swagger UI page where you can read about every endpoint and even make live API calls.

### Purposes

- To serve interactive, auto-generated API documentation from within the Express application.
- To provide a live API console for testing endpoints without leaving the documentation.
- To keep documentation in sync with the API specification at all times.
- To eliminate the need for a separate documentation hosting environment.

### Syntax Rules and Structure

**Basic Setup:**
```js
const express = require('express');
const app = express();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
```

| Component | Breakdown |
|-----------|-----------|
| `swaggerUi.serve` | Middleware that serves Swagger UI static assets. |
| `swaggerUi.setup(doc, opts)` | Middleware that generates the HTML page bound to `doc`. |
| `'/api-docs'` | The route where the documentation is mounted. |

**Express Router Setup:**
```js
const router = require('express').Router();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

router.use('/api-docs', swaggerUi.serve);
router.get('/api-docs', swaggerUi.setup(swaggerDocument));
```

**Integration with swagger-jsdoc:**
```js
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerSpec = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

**Rules:**
- Both `swaggerUi.serve` and `swaggerUi.setup()` must be mounted on the same route path.
- When using Express Router, `router.use()` serves assets and `router.get()` renders the HTML page.
- The specification can be a JSON object (from `require()`) or a dynamically generated spec (from `swagger-jsdoc`).
- The `swagger-ui-dist` version is pulled from npm; use a lock file to ensure consistency across environments.

### Annotated Code Example

```js
// swagger-ui-basic.js
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./openapi.json');  // Your OpenAPI spec

const app = express();

// Mount Swagger UI at /api-docs
app.use(
  '/api-docs',
  swaggerUi.serve,           // Serve Swagger UI static assets
  swaggerUi.setup(swaggerDocument, {
    explorer: true,           // Show the search/explorer bar
    customSiteTitle: 'My API Documentation',
    swaggerOptions: {
      persistAuthorization: true  // Keep auth tokens across page reloads
    }
  })
);

app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.listen(3000, () => {
  console.log('Server on 3000');
  console.log('Docs: http://localhost:3000/api-docs');
});
```

**Expected Output (browser at `http://localhost:3000/api-docs`):**
```
Swagger UI page rendering:
- Title: "My API Documentation"
- Explorer bar (search/filter operations)
- GET /api/users — List users
- "Try it out" button enabled
- Authorize button (if security schemes defined)
```

**Why this output:** The middleware mounts Swagger UI at `/api-docs`. The `explorer: true` option enables the search bar for filtering operations. `customSiteTitle` sets the browser tab title. `persistAuthorization: true` keeps authentication tokens across page reloads during testing. The page renders every operation from the OpenAPI document with expandable request/response schemas and the interactive sandbox.

### Real-World Cases

- **Internal APIs:** Teams use Swagger UI as living documentation that developers can test against during development.
- **Public APIs:** Companies host Swagger UI for external developers to explore and test endpoints.
- **Microservices:** Each service hosts its own Swagger UI, or a gateway aggregates multiple specs.
- **CI/CD validation:** Swagger UI is used in staging environments to manually verify new endpoints.

---

## Core Concept 2: redoc-express — Clean, Three-Panel API Documentation

### Definitions

**Core Definition:** `redoc-express` is an Express middleware that serves the ReDoc documentation renderer — a clean, responsive, three-panel API reference that prioritises readability over interactivity.

**Technical Definition:** ReDoc is an open-source documentation engine that renders OpenAPI/Swagger specifications with a three-panel responsive design: a left sidebar with navigation (menu), a central content area with the API reference, and a right panel for code samples and schema details. It supports nested object documentation, meaningful request/response samples generated via `openapi-sampler`, code samples via vendor extensions, and a comprehensive theming API. `redoc-express` wraps ReDoc as an Express middleware with zero configuration required.

**Beginner-Friendly Explanation:** ReDoc is the "reading" alternative to Swagger UI's "testing" approach. Instead of a single-column layout with expandable operations, ReDoc shows you everything at once in a clean three-column design. The left side is a navigation menu, the middle is the documentation, and the right side shows code samples. It's easier to read and navigate, especially for large APIs — but it doesn't have a "Try it out" button.

### Purposes

- To serve clean, readable API documentation that prioritises navigation and comprehension over testing.
- To provide a responsive three-panel layout that works well on large APIs with many endpoints.
- To generate meaningful request/response samples automatically from the OpenAPI spec.
- To offer a documentation-focused alternative to Swagger UI's console-centric approach.

### Syntax Rules and Structure

**Basic Setup:**
```js
const express = require('express');
const redoc = require('redoc-express');
const app = express();

// Serve the OpenAPI spec file
app.get('/docs/swagger.json', (req, res) => {
  res.sendFile('swagger.json', { root: '.' });
});

// Mount ReDoc middleware
app.get('/docs', redoc({
  title: 'API Documentation',
  specUrl: '/docs/swagger.json'
}));
```

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `title` | String | Yes | Title displayed in the browser tab and header. |
| `specUrl` | String | Yes | URL to the OpenAPI/Swagger specification file. |
| `nonce` | String | No | CSP nonce for Content Security Policy compliance. |
| `redocOptions` | Object | No | ReDoc configuration options (theming, behaviour). |

**Rules:**
- The specification file must be served from a route (e.g., `/docs/swagger.json`) before mounting ReDoc.
- `redoc-express` supports both OpenAPI 3.0+ and Swagger 2.0 specifications.
- The `redocOptions` object passes configuration directly to the ReDoc renderer.
- ReDoc uses a three-panel responsive layout with menu/scrolling synchronisation.
- Deep linking is supported — each operation has a unique anchor URL.

### Annotated Code Example

```js
// redoc-basic.js
const express = require('express');
const redoc = require('redoc-express');
const app = express();

// Serve the OpenAPI spec
app.get('/docs/swagger.json', (req, res) => {
  res.sendFile('openapi.json', { root: '.' });
});

// Mount ReDoc with theming
app.get('/docs', redoc({
  title: 'My API Reference',
  specUrl: '/docs/swagger.json',
  redocOptions: {
    theme: {
      colors: {
        primary: { main: '#1976d2' }
      },
      typography: {
        fontFamily: '"Inter", Helvetica, Arial, sans-serif',
        fontSize: '15px'
      },
      menu: {
        backgroundColor: '#f8f9fa'
      }
    },
    hideDownloadButton: true,
    expandResponses: '200,201,400'
  }
}));

app.listen(3000, () => {
  console.log('ReDoc on http://localhost:3000/docs');
});
```

**Expected Output (browser at `http://localhost:3000/docs`):**
```
ReDoc three-panel layout:
- Left: Navigation menu with all operations grouped by tag
- Center: API reference with schemas, parameters, and descriptions
- Right: Request/response code samples (auto-generated)
- Primary colour: blue (#1976d2)
- Font: Inter
- Download button: hidden
- Responses 200, 201, 400: expanded by default
```

**Why this output:** The `redocOptions` object controls the entire appearance and behaviour. The primary colour is set to blue, the font is Inter, and the menu background is a light grey. `hideDownloadButton: true` removes the "Download" button for the spec file. `expandResponses: '200,201,400'` automatically expands those response codes so readers see them without clicking.

### Real-World Cases

- **Large enterprise APIs:** ReDoc's three-panel layout handles hundreds of endpoints more gracefully than Swagger UI.
- **Public documentation sites:** Companies use ReDoc for developer-facing API references.
- **Internal documentation portals:** Teams that need readable docs without the testing sandbox.
- **Multi-language APIs:** ReDoc's theming API makes it easy to match corporate branding.

---

## Core Concept 3: API Exploration and Sandboxing ("Try it out")

### Definitions

**Core Definition:** The "Try it out" feature in Swagger UI allows users to execute live HTTP requests directly from the documentation page, with automatic population of parameters, request bodies, and authentication headers.

**Technical Definition:** Swagger UI's "Try it out" mode is enabled per-operation or globally via the `tryItOutEnabled` configuration option. When enabled, each operation displays a "Try it out" button that expands editable input fields for all parameters and request bodies. The "Execute" button sends the request to the API endpoint using the browser's `fetch` API, and the response (status code, headers, body) is displayed inline. The `persistAuthorization` option keeps authentication tokens (from the "Authorize" dialog) across page reloads.

**Beginner-Friendly Explanation:** "Try it out" turns your API documentation into an interactive testing tool. Instead of just reading about an endpoint, you can fill in the parameters, click "Execute," and see the actual response from your server — all without leaving the documentation page. It's like having a built-in Postman inside your Swagger UI.

### Purposes

- To enable developers to test API endpoints without writing code or using external tools.
- To provide immediate feedback on request/response behaviour.
- To validate that authentication and parameter handling work as documented.
- To reduce the friction of onboarding new API consumers.

### Syntax Rules and Structure

**Enabling "Try it out" by Default:**
```js
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, {
  swaggerOptions: {
    tryItOutEnabled: true,      // Enable "Try it out" mode by default
    persistAuthorization: true  // Keep auth tokens across reloads
  }
}));
```

| Option | Default | Description |
|--------|---------|-------------|
| `tryItOutEnabled` | `false` | Controls whether "Try it out" is enabled by default. |
| `persistAuthorization` | `false` | Keeps auth tokens across page reloads. |
| `displayOperationId` | `false` | Shows `operationId` in the UI. |
| `defaultModelsExpandDepth` | `1` | How deeply models are expanded by default. |

**Disabling "Try it out":**
```js
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, {
  swaggerOptions: {
    tryItOutEnabled: false  // Disable the sandbox entirely
  }
}));
```

**Rules:**
- `tryItOutEnabled: true` makes the "Try it out" button active by default; users can still toggle it off.
- `tryItOutEnabled: false` disables the sandbox entirely — useful for production documentation.
- The "Authorize" dialog (from security schemes) must be completed before authenticated endpoints can be tested.
- The sandbox uses the browser's `fetch` API, so CORS must be configured to allow requests from the documentation origin.

### Annotated Code Example

```js
// swagger-ui-try-it-out.js
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./openapi.json');
const app = express();

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, {
  swaggerOptions: {
    tryItOutEnabled: true,       // Enable sandbox by default
    persistAuthorization: true,  // Keep tokens across reloads
    displayOperationId: true,    // Show operationId in UI
    defaultModelsExpandDepth: 2  // Expand models two levels deep
  }
}));

// CORS must allow the documentation origin
app.use((req, res, next) => {
  res.set('Access-Control-Allow-Origin', '*');
  res.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  next();
});

app.listen(3000, () => console.log('Swagger UI with sandbox on 3000'));
```

**Expected Output (interaction flow):**
```
1. User opens /api-docs
2. "Authorize" button → enters Bearer token → token persists
3. Clicks on GET /api/users
4. "Try it out" button is already active (tryItOutEnabled: true)
5. Clicks "Execute"
6. Response appears inline:
   - Code: 200
   - Body: [{"id": 1, "name": "Alice"}]
   - Headers: Content-Type: application/json
```

**Why this output:** `tryItOutEnabled: true` makes the sandbox active by default, so the user doesn't have to click "Try it out" to start editing. `persistAuthorization: true` keeps the Bearer token across page reloads, so the user doesn't have to re-authenticate. `displayOperationId: true` shows the `operationId` for code generation reference. The CORS middleware allows the browser to make requests to the API from the documentation page.

### Real-World Cases

- **API onboarding:** New developers test endpoints immediately without setting up a local environment.
- **QA testing:** Manual testers use "Try it out" for exploratory testing.
- **Production docs:** "Try it out" is disabled in production for security, but enabled in staging.
- **SDK validation:** Developers verify that generated SDKs match the documented behaviour.

---

## Core Concept 4: Customising UI Themes and Branding

### Definitions

**Core Definition:** Theme customisation is the practice of modifying the appearance of Swagger UI and ReDoc — colours, fonts, logos, layout, and behaviour — to match an organisation's brand identity.

**Technical Definition:** Swagger UI supports customisation through `customCss` (inline CSS string), `customCssUrl` (URL to a CSS file), `customJs` (URL to a JavaScript file), `customSiteTitle` (browser tab title), and `customfavIcon` (favicon URL). ReDoc supports a comprehensive theming API via `redocOptions.theme`, covering colours (`primary.main`), typography (`fontFamily`, `fontSize`), spacing, and sidebar styling. Both tools accept `redocOptions` and `swaggerOptions` objects for deep configuration.

**Beginner-Friendly Explanation:** By default, Swagger UI and ReDoc have their own look. But you can change almost everything: the primary colour, the font, the logo, even hide elements you don't want. Swagger UI uses CSS overrides; ReDoc has a structured theming API that's easier to work with. The goal is to make your API documentation look like it belongs to your brand, not like a generic tool.

### Sub-Feature 4.1: Swagger UI Customisation

#### Syntax Rules and Structure

```js
const options = {
  customSiteTitle: 'My API Docs',           // Browser tab title
  customfavIcon: '/favicon.ico',            // Favicon URL
  customCss: '.swagger-ui .topbar { display: none }',  // Inline CSS
  customCssUrl: '/custom.css',              // External CSS file
  customJs: '/custom.js',                   // External JS file
  swaggerOptions: { tryItOutEnabled: true }
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));
```

| Option | Type | Description |
|--------|------|-------------|
| `customSiteTitle` | String | Custom browser tab title. |
| `customfavIcon` | String | Favicon URL. |
| `customCss` | String | Inline CSS overrides. |
| `customCssUrl` | String | URL to external CSS file. |
| `customJs` | String | URL to external JavaScript file. |
| `customJsStr` | String | Inline JavaScript string. |

**Common CSS Overrides:**
```css
/* Hide the top bar entirely */
.swagger-ui .topbar { display: none; }

/* Change the primary colour */
.swagger-ui .btn.authorize { background-color: #1976d2; }

/* Change the font */
.swagger-ui .info .title { font-family: 'Inter', sans-serif; }
```

**Rules:**
- `customCss` takes an inline CSS string; `customCssUrl` takes a URL to a CSS file.
- `customJs` loads external JavaScript; `customJsStr` accepts an inline JS string.
- The `swaggerOptions` object is passed directly to the Swagger UI client and controls behaviour (`tryItOutEnabled`, `persistAuthorization`, etc.).
- To hide the top bar, use `.swagger-ui .topbar { display: none }`.
- To inject multiple CSS files, you may need to use inline CSS or concatenate them.

#### Annotated Code Example

```js
// swagger-ui-customisation.js
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./openapi.json');
const app = express();

// Serve custom assets
app.use('/static', express.static('public'));

const options = {
  customSiteTitle: 'Acme Corp API',
  customfavIcon: '/static/favicon.ico',
  customCss: `
    .swagger-ui .topbar { display: none; }
    .swagger-ui .info .title { color: #1976d2; font-family: 'Inter', sans-serif; }
    .swagger-ui .btn.authorize { background-color: #1976d2; border-color: #1976d2; }
    .swagger-ui .opblock.opblock-get .opblock-summary-method { background: #1976d2; }
  `,
  customJsStr: `
    // Custom analytics or behaviour
    console.log('Swagger UI loaded for Acme Corp');
  `,
  swaggerOptions: {
    tryItOutEnabled: true,
    persistAuthorization: true
  }
};

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, options));

app.listen(3000, () => console.log('Branded Swagger UI on 3000'));
```

**Expected Output:**
```
Swagger UI page with:
- Top bar hidden
- Title "Acme Corp API" in browser tab
- Custom favicon
- Primary colour #1976d2 (blue) applied to authorise button and GET method badges
- "Try it out" enabled by default
- Auth tokens persisted
```

**Why this output:** The `customCss` string overrides the default Swagger UI styles: the top bar is hidden, the title is blue, and the authorise button matches the brand colour. `customSiteTitle` sets the browser tab title, and `customfavIcon` sets the favicon. `customJsStr` injects a console log for analytics. The `swaggerOptions` enable the sandbox and persist auth.

---

### Sub-Feature 4.2: ReDoc Customisation

#### Syntax Rules and Structure

```js
app.get('/docs', redoc({
  title: 'My API Reference',
  specUrl: '/docs/swagger.json',
  redocOptions: {
    theme: {
      colors: {
        primary: { main: '#1976d2' },
        success: { main: '#4caf50' },
        error: { main: '#f44336' }
      },
      typography: {
        fontFamily: '"Inter", Helvetica, Arial, sans-serif',
        fontSize: '15px',
        headings: { fontFamily: '"Inter", sans-serif' }
      },
      menu: {
        backgroundColor: '#f8f9fa',
        activeTextColor: '#1976d2'
      },
      sidebar: {
        backgroundColor: '#ffffff'
      }
    },
    hideDownloadButton: true,
    expandResponses: '200,201,400',
    requiredPropsFirst: true,
    sortPropsAlphabetically: false
  }
}));
```

| Option | Type | Description |
|--------|------|-------------|
| `theme.colors.primary.main` | Hex/RGBA | Primary highlight colour. |
| `theme.typography.fontFamily` | String | Font family for body text. |
| `theme.typography.fontSize` | String | Font size (e.g., `'15px'`). |
| `theme.menu.backgroundColor` | Hex/RGBA | Sidebar background colour. |
| `hideDownloadButton` | Boolean | Hide the "Download" button. |
| `expandResponses` | String | Response codes to expand by default (e.g., `'200,201,400'`). |
| `requiredPropsFirst` | Boolean | Show required properties first. |

**Rules:**
- The `theme` object is passed directly to the ReDoc renderer.
- `colors.primary.main` is the primary highlight colour; `primaryColor` is a convenient shorthand for the same value.
- `expandResponses` accepts a comma-separated list of response codes or `'all'`.
- `hideDownloadButton` hides the button but does not make the spec private.
- ReDoc's theming API is fully documented at `redocly.com/docs/api-reference-docs/configuration/functionality/`.

#### Annotated Code Example

```js
// redoc-customisation.js
const express = require('express');
const redoc = require('redoc-express');
const app = express();

app.get('/docs/swagger.json', (req, res) => {
  res.sendFile('openapi.json', { root: '.' });
});

app.get('/docs', redoc({
  title: 'Acme Corp API Reference',
  specUrl: '/docs/swagger.json',
  redocOptions: {
    theme: {
      colors: {
        primary: { main: '#1976d2' },
        success: { main: '#4caf50' },
        error: { main: '#f44336' }
      },
      typography: {
        fontFamily: '"Inter", Helvetica, Arial, sans-serif',
        fontSize: '15px',
        headings: { fontFamily: '"Inter", sans-serif' }
      },
      menu: {
        backgroundColor: '#f8f9fa',
        activeTextColor: '#1976d2'
      },
      sidebar: {
        backgroundColor: '#ffffff'
      }
    },
    hideDownloadButton: true,
    expandResponses: '200,201,400',
    requiredPropsFirst: true
  }
}));

app.listen(3000, () => console.log('Branded ReDoc on 3000'));
```

**Expected Output:**
```
ReDoc three-panel page with:
- Title "Acme Corp API Reference" in browser tab
- Primary colour #1976d2 (blue) for links and active menu items
- Font: Inter
- Sidebar menu background: light grey (#f8f9fa)
- Active menu item: blue (#1976d2)
- Download button: hidden
- Responses 200, 201, 400: expanded by default
- Required properties shown first
```

**Why this output:** The `theme.colors.primary.main` sets the primary highlight colour across the entire documentation. `typography.fontFamily` sets the body font to Inter. `menu.backgroundColor` and `menu.activeTextColor` style the sidebar navigation. `hideDownloadButton` removes the download button. `expandResponses` automatically expands the most common response codes so readers see them without clicking. `requiredPropsFirst` shows required properties before optional ones.

### Real-World Cases

- **Brand consistency:** Matching API documentation to a company's design system.
- **White-label documentation:** Embedding API docs in a customer-facing portal with the customer's branding.
- **Dark mode:** Using ReDoc's theming API to provide a dark mode for developer preference.
- **Logo injection:** Using Swagger UI's `customCss` to add a logo to the top bar.

---

## References

- swagger-ui-express GitHub Repository — https://github.com/scottie1984/swagger-ui-express
- Swagger UI Configuration Options — https://github.com/swagger-api/swagger-ui/blob/master/docs/usage/configuration.md
- redoc-express on npm — https://www.npmjs.com/package/redoc-express
- ReDoc Official Documentation — https://redocly.com/docs/redoc/
- ReDoc Configuration Options — https://redocly.com/docs/api-reference-docs/configuration/functionality/
- ReDoc — Reinventing OpenAPI-powered Documentation (APIs.guru Blog) — https://apis.guru/blog/redoc-reinventing-openapi-powered-documentation
- ReDoc CE React Component (Redocly) — https://redocly.com/docs/redoc/deployment/react/
- Theming Options for API Docs (Redocly) — https://redocly.com/docs/api-reference-docs/configuration/theming
- swagger-jsdoc Documentation — https://github.com/Surnet/swagger-jsdoc
- modern-swagger-theme on npm — https://www.npmjs.com/package/modern-swagger-theme
- ReDoc vs Swagger UI Comparison (ONES.com) — https://ones.com/blog/open-api-documentation-viewers/
- Express OpenAPI Extraction (swagger-jsdoc) — https://github.com/specmatic/skills/blob/main/skills/specmatic-openapi-spec-extractor/content/frameworks/express.md
- OpenAPI Specification v3.1.0 — https://spec.openapis.org/oas/v3.1.0.html
- swagger-ui-dist on npm — https://www.npmjs.com/package/swagger-ui-dist