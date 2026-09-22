# React Development Environment

Setting up a productive React development environment involves several layers: a JavaScript runtime, a package manager, a project scaffold, a dev server, build tooling, and browser-based debugging tools. This section walks through each piece.

## Node.js Runtime

**Node.js** is a JavaScript runtime built on Chrome's V8 engine that lets you run JavaScript outside the browser. It is the foundation of virtually every modern React toolchain.

- **Why it's needed** — Build tools (Vite, webpack), the dev server, package managers, and test runners all run on Node.js.
- **LTS vs. Current** — Node releases come in two tracks: **LTS** (Long-Term Support) for stability and production, and **Current** for the latest features. Most teams use the latest LTS.
- **Version management** — Tools like **nvm** (macOS/Linux), **nvm-windows**, or **fnm** let you switch Node versions per project.
- **`package.json` engines field** — You can declare a required Node version to keep teams consistent:

```json
{
  "engines": { "node": ">=20" }
}
```

## Package Managers

A package manager installs, updates, and manages the dependencies your React project relies on. It reads `package.json` and produces a lockfile to guarantee reproducible installs.

### npm
- Ships with Node.js — no separate install required.
- Uses `package.json` and `package-lock.json`.
- Commands: `npm install`, `npm run dev`, `npm ci` (clean install for CI).

### pnpm
- Uses a **content-addressable store** and hard links to save disk space and speed up installs.
- Strict dependency resolution (no phantom dependencies).
- Lockfile: `pnpm-lock.yaml`.
- Popular in monorepos.

### Yarn
- Originally created by Facebook to address npm's early performance issues.
- Two major lines: **Yarn Classic (1.x)** and **Yarn Berry (2+/3+/4)** with Plug'n'Play.
- Lockfile: `yarn.lock`.
- Commands: `yarn`, `yarn dev`, `yarn add`.

| Feature | npm | pnpm | Yarn |
|---|---|---|---|
| Bundled with Node | Yes | No | No |
| Lockfile | `package-lock.json` | `pnpm-lock.yaml` | `yarn.lock` |
| Disk efficiency | Moderate | High | Moderate |
| Monorepo support | Workspaces | Workspaces (strong) | Workspaces |

> **Tip:** Pick one package manager per project. Mixing lockfiles causes inconsistent installs.

## Project Scaffolding

Scaffolding generates a starter project with sensible defaults, so you don't wire up tooling by hand.

- **Vite** (recommended for most new projects):
  ```bash
  npm create vite@latest my-app -- --template react
  # or with TypeScript
  npm create vite@latest my-app -- --template react-ts
  ```
  Fast, minimal, uses esbuild for dev and Rollup for production.

- **Next.js** (React meta-framework with routing, SSR, and more):
  ```bash
  npx create-next-app@latest
  ```

- **Remix / React Router templates**:
  ```bash
  npx create-remix@latest
  ```

- **Create React App (CRA)** — Historically popular but now **deprecated**; the React team recommends Vite, Next.js, or similar alternatives.

After scaffolding, typical next steps:
```bash
cd my-app
npm install
npm run dev
```

## Development Servers

A **development server** serves your app locally with fast feedback while you code.

- **Hot Module Replacement (HMR)** — Updates modules in the browser without a full reload, preserving state where possible.
- **Fast Refresh** — React-specific HMR that preserves component state across edits.
- **On-demand compilation** — Modern servers (Vite, esbuild-based) compile only what the browser requests, so startup is nearly instant.
- **Default port** — Usually `http://localhost:5173` (Vite) or `http://localhost:3000` (Next.js).
- **Proxy configuration** — Dev servers can proxy API requests to a backend to avoid CORS issues during development.

Example Vite config with a proxy:
```js
// vite.config.js
export default {
  server: {
    proxy: {
      "/api": "http://localhost:8080"
    }
  }
};
```

## Production Builds

When you're ready to ship, you run a **production build** that optimizes your code.

- **Bundling** — Combines modules into a small number of files.
- **Minification** — Removes whitespace and shortens identifiers (e.g., via esbuild or Terser).
- **Tree shaking** — Eliminates unused exports.
- **Code splitting** — Splits the bundle into chunks loaded on demand (e.g., per route).
- **Asset hashing** — Filenames include content hashes for cache busting (`main.4f2a1c.js`).
- **Compression** — Gzip or Brotli output for smaller transfers.

Typical commands:
```bash
npm run build      # creates ./dist or ./build
npm run preview    # serves the production build locally
```

## Browser Developer Tools

Modern browsers ship with built-in tools essential for React work:

- **Elements panel** — Inspect the rendered DOM (note: React components aren't visible here, only the DOM output).
- **Console** — View logs, errors, and warnings.
- **Network panel** — Inspect API requests, bundle sizes, and load times.
- **Sources panel** — Set breakpoints and step through code.
- **Performance panel** — Profile rendering and identify bottlenecks.
- **Lighthouse** — Audit performance, accessibility, SEO, and best practices.

## React Developer Tools

**React Developer Tools** is a browser extension (Chrome, Firefox, Edge) that adds React-specific panels to DevTools.

- **Components tab** — Inspect the React component tree, view props, state, and hooks for any component.
- **Profiler tab** — Record render sessions to find components that re-render too often or take too long.
- **Highlight updates** — Visually flash components when they re-render.
- **Standalone app** — Also available for React Native via `react-devtools`.

## Environment Variables

Environment variables let you configure your app differently across environments (development, staging, production) without changing code.

- **Naming conventions** vary by tool:
  - Vite: `VITE_API_URL`
  - Next.js: `NEXT_PUBLIC_API_URL`
  - CRA (legacy): `REACT_APP_API_URL`
- Only variables with the correct prefix are exposed to the client — everything else stays server-side.
- Stored in `.env`, `.env.local`, `.env.production`, etc. (usually gitignored).

Example `.env.local`:
```
VITE_API_URL=https://api.example.com
```

Usage in code:
```js
const apiUrl = import.meta.env.VITE_API_URL;
```

> **Security note:** Never put secrets (API keys with write access, tokens) in client-exposed env vars — they end up in the bundle.

## Source Maps

**Source maps** map your compiled/bundled code back to the original source, making debugging far easier.

- **In development** — Enabled by default; you debug the original JSX/TS files in DevTools.
- **In production** — Often generated but hidden or uploaded to error-tracking services (e.g., Sentry) rather than shipped publicly, to avoid exposing source.
- **Types**:
  - `eval-source-map` — Fast rebuilds, good for dev.
  - `source-map` — Full external map, common for production.
  - `hidden-source-map` — Generates a map without linking it in the bundle.

Vite config example:
```js
export default {
  build: {
    sourcemap: true // or 'hidden'
  }
};
```

## Project Scripts

Scripts in `package.json` define the commands your team runs day-to-day. Common examples:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "format": "prettier --write .",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "typecheck": "tsc --noEmit"
  }
}
```

Run any script with:
```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

- **`dev`** — Starts the development server with HMR.
- **`build`** — Produces an optimized production bundle.
- **`preview`** — Serves the built output locally for testing.
- **`lint` / `format`** — Enforce code style and catch issues early.
- **`test`** — Runs your test suite.
- **`typecheck`** — Validates TypeScript types without emitting files.

---

**In summary**, a modern React development environment is built on Node.js, uses a package manager for dependencies, scaffolds a project with a tool like Vite or Next.js, runs a fast dev server with HMR, builds optimized bundles for production, and relies on browser DevTools plus React Developer Tools for debugging. Environment variables, source maps, and well-defined npm scripts tie it all together into a smooth workflow.