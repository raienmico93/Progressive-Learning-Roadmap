# React Bundle Optimization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Bundle optimization is the practice of reducing the size and improving the delivery efficiency of the JavaScript, CSS, and asset bundles that a React application ships to the browser.

**Technical Definition:** Bundle optimization encompasses the build-time and delivery-time techniques that minimise the bytes transferred over the network, the time spent parsing and executing JavaScript, and the memory consumed by the application. In a React application, the bundle is produced by a bundler (webpack, Vite/Rollup, or a framework like Next.js) that traces the module graph, applies transformations (tree shaking, dead code elimination, minification), optionally splits the output into chunks, and emits assets that can be compressed (gzip, Brotli) and served with long-term caching. The goal is to deliver the smallest possible payload that still renders the correct UI.

**Beginner-Friendly Explanation:** Bundle optimization is about making the files your app sends to the browser as small and efficient as possible. Every kilobyte of JavaScript has to be downloaded, parsed, and executed before your users see anything. Optimizing the bundle means removing code you do not need (tree shaking), stripping out dead code, shortening variable names and whitespace (minification), compressing the files (gzip or Brotli), and analysing the result to find what is bloating the bundle.

### Key Characteristics

- **Layered pipeline:** Optimization happens at multiple stages: module format (ESM), bundler configuration (production mode), code transformation (tree shaking, DCE, minification), output configuration (chunking), and delivery (compression, caching).
- **Production mode is prerequisite:** Tree shaking and minification are only fully enabled when the bundler runs in production mode.
- **ES modules are tree-shakeable:** CommonJS (`require`) is not statically analysable and defeats tree shaking; ES modules (`import`/`export`) are required.
- **`sideEffects` governs aggressiveness:** Marking a package as `"sideEffects": false` allows bundlers to drop unused modules, but it must be accurate.
- **Compression is delivery, not bundling:** Minification reduces source size; compression (gzip/Brotli) reduces transfer size. Both are necessary.
- **Analysis drives action:** Tools like `webpack-bundle-analyzer`, `source-map-explorer`, and `rollup-plugin-visualizer` reveal what is actually inside the bundle.

### Prerequisites

- React function components and JSX.
- ES modules (`import`/`export`) and the difference between ESM and CommonJS.
- Familiarity with a bundler (webpack, Vite/Rollup, or Next.js).
- Basic understanding of HTTP compression and caching headers.

### Related Programming Areas

- **Code Splitting** — partitioning the bundle into on-demand chunks.
- **Rendering Performance** — parsing and executing the bundle blocks the main thread.
- **Core Web Vitals** — LCP and INP are directly affected by bundle size.
- **Asset Optimization** — images, fonts, and CSS contribute to the total payload.

---

## 1. Bundle Size

### Definitions

**Core Definition:** Bundle size is the total byte size of the JavaScript, CSS, and other assets that the bundler emits and the browser must download, parse, and execute.

**Technical Definition:** Bundle size is measured at multiple levels: **raw size** (uncompressed), **minified size** (after whitespace and identifier removal), **compressed size** (after gzip or Brotli), and **parsed size** (the in-memory representation after the JavaScript engine has parsed it). The metric that matters most for user experience is the **compressed transfer size** of the initial bundle, because that is what travels over the network before the first render. The parsed and executed size determines main-thread blocking time, which affects Interaction to Next Paint (INP).

**Beginner-Friendly Explanation:** Bundle size is how big your app's files are. Smaller files download faster, parse faster, and leave more room for the browser to respond to the user.

### Purposes

- To reduce the time to first meaningful content (LCP).
- To reduce main-thread blocking from parsing and executing JavaScript (INP).
- To lower memory usage on constrained devices.
- To improve cache efficiency and reduce bandwidth costs.

### Syntax Rules and Structure

**Size Metrics:**

| Metric | What it measures | Why it matters |
|--------|------------------|----------------|
| Raw size | Uncompressed bytes | Build artefact size |
| Minified size | After whitespace/identifier removal | Parse and execute cost |
| Compressed size | After gzip/Brotli | Network transfer cost |
| Parsed size | In-memory representation | Main-thread blocking |

**Syntax Rules:**

1. Track the compressed size of the initial bundle, not just the raw size.
2. Distinguish between the entry bundle and lazily loaded chunks.
3. Set a budget (e.g., entry JS ≤ 200KB compressed) and enforce it in CI.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Measuring Bundle Size in Vite

```bash
# Build and inspect the output sizes
npx vite build

# Output shows compressed sizes (gzip and Brotli)
# dist/assets/index-abc123.js   142.35 kB │ gzip: 45.67 kB │ brotli: 39.21 kB
```

**Expected Output:** The build report displays the raw, gzip, and Brotli sizes for each emitted asset.

**Why this output:** Vite’s build reporter calculates compression sizes automatically, giving an immediate view of the transfer cost.

#### Example 2: Enforcing a Bundle Size Budget with `size-limit`

```json
// package.json
{
  "size-limit": [
    {
      "path": "dist/assets/index-*.js",
      "limit": "200 kB"
    }
  ]
}
```

**Expected Output:** CI fails if the entry bundle exceeds 200KB.

**Why this output:** `size-limit` measures the compressed size of the matched assets and compares it against the configured budget.

### Real-World Cases

- **E-commerce:** A 100KB reduction in the entry bundle can measurably improve mobile conversion.
- **SaaS dashboards:** Large entry bundles delay the first meaningful paint of the dashboard shell.
- **Content sites:** Bundle size competes with images and fonts for the critical path.

### References

- React + Vite Best Practices — https://raw.githubusercontent.com/voku/agent-skills/eaaf04fa3e1192e34e93f5bda46e5d85a8ccfc1a/skills/react-vite-best-practices/AGENTS.md
- Bundle optimization rules (50–90% dead code elimination, 40–70% initial bundle reduction) — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/react-optimise/AGENTS.md

---

## 2. Tree Shaking

### Definitions

**Core Definition:** Tree shaking is a build-time optimization that removes unused exports from ES modules, keeping only the code that is actually imported and used.

**Technical Definition:** Tree shaking analyses the static import/export graph of ES modules to determine which exports are reachable from the entry point. Unreachable exports are marked and removed by the bundler (and by a minifier such as Terser during dead code elimination). Tree shaking requires ES modules because their imports and exports are statically analysable; CommonJS `require` calls are dynamic and cannot be reliably analysed. In webpack, the `sideEffects` field in `package.json` tells the bundler whether a module’s top-level code has side effects that prevent removal. The user-visible effect of tree shaking requires `mode: 'production'` or explicit `optimization.usedExports`.

**Beginner-Friendly Explanation:** Tree shaking is like shaking a tree so that only the leaves you need stay attached. If you import one function from a library, tree shaking removes all the other functions you did not import. It only works with modern ES module syntax, not with older `require()` syntax.

### Purposes

- To eliminate unused code from third-party libraries and application modules.
- To reduce the parsed and executed JavaScript size.
- To improve load time and main-thread availability.
- To enable smaller, more focused dependency usage.

### Syntax Rules and Structure

**Tree-Shaking Requirements:**

| Requirement | Why it matters |
|-------------|----------------|
| ES modules (`import`/`export`) | Static analysis is possible |
| Production mode | Tree shaking is enabled by default |
| `sideEffects` accuracy | Prevents removal of modules with side effects |
| Named imports | Avoids importing entire namespaces |

**Syntax Rules:**

1. Use ES modules, not CommonJS.
2. Enable `mode: 'production'` in webpack or the equivalent production build in Vite/Rollup.
3. Set `"sideEffects": false` in your package (or list files with side effects).
4. Use named imports (`import { debounce } from 'lodash-es'`) rather than namespace imports or default imports from large libraries.
5. Avoid barrel files (`export * from './Button'`) that re-export everything.

**Constraints and Limitations:**

- Tree shaking cannot remove code inside a single module that is only partially used (e.g., an object with many properties); it operates at the module export level.
- External dependencies marked as `external` are not tree-shaken by the consumer’s bundler.
- Dynamic imports cannot be tree-shaken at build time.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Correct Named Imports for Tree Shaking

```jsx
// ❌ Imports the entire library namespace
import * as _ from 'lodash';
const result = _.debounce(fn, 300);

// ✅ Imports only the debounce function
import { debounce } from 'lodash-es';
const result = debounce(fn, 300);

// ✅ Even better: direct path import
import debounce from 'lodash-es/debounce';
```

**Expected Output:** The named imports allow the bundler to include only `debounce` (and its dependencies) in the final bundle.

**Why this output:** ES module named imports are statically analysable. The bundler sees that only `debounce` is used and removes the rest of the library.

#### Example 2: `sideEffects` Configuration

```json
{
  "name": "my-design-system",
  "sideEffects": ["*.css", "*.scss"]
}
```

**Expected Output:** JavaScript modules without side effects are eligible for tree shaking; CSS files are preserved because they have side effects.

**Why this output:** The `sideEffects` array marks specific file patterns as having side effects. All other modules can be safely removed if unused.

### Real-World Cases

- **Icon libraries:** Importing individual icons (`import { FaUser } from 'react-icons/fa'`) instead of the entire icon set.
- **Utility libraries:** Using `lodash-es` with named imports instead of the monolithic `lodash`.
- **Component libraries:** Direct imports from individual component paths to avoid barrel-file overhead.

### References

- How to tree-shake in React · CoreUI — https://coreui.io/answers/how-to-tree-shake-in-react/
- Tree shaking Support Guide (web3.js) — https://raw.githubusercontent.com/web3/web3.js/53a62eda5dcb240c3f684cf9f8ef491611ec4339/docs/docs/guides/advanced/tree_shaking.md
- Production-mode requirement for tree shaking — https://github.com/vanguard-frontier-agentic/skills/blob/master/frontend/tree-shaking-dead-code-review/references/module-format-and-sideeffects.md
- Barrel files and tree shaking — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/react-optimise/AGENTS.md

---

## 3. Dead Code Elimination

### Definitions

**Core Definition:** Dead code elimination (DCE) is the process of removing code that can never be executed or whose result is never used.

**Technical Definition:** DCE operates at two levels in a JavaScript build pipeline. First, the bundler annotates and removes unused module exports (tree shaking). Second, a minifier such as Terser performs a deeper analysis of the generated code and removes unreachable branches, unused variables, and expressions with no effect. Webpack’s DCE is implemented by annotating and removing unused module exports, then relying on Terser to perform dead code elimination; preservation of some module boundaries in webpack bundles can limit the amount of optimisation Terser can perform. Development-only branches guarded by `process.env.NODE_ENV === 'production'` are a classic DCE target.

**Beginner-Friendly Explanation:** Dead code elimination is the final cleanup pass. It removes code that can never run, like development-only logging or branches that are always false. It works together with tree shaking: tree shaking removes unused exports, and DCE removes dead code inside the remaining modules.

### Purposes

- To remove development-only code from production bundles.
- To eliminate unreachable branches and unused variables.
- To reduce the final parsed and executed JavaScript size.
- To complement tree shaking for a smaller production bundle.

### Syntax Rules and Structure

**DCE Targets:**

| Target | Example |
|--------|---------|
| Development branches | `if (process.env.NODE_ENV === 'development') { ... }` |
| Unreachable code | `if (false) { ... }` |
| Unused variables | `const unused = expensive();` |
| Console/debug calls | `console.log(...)` (configurable) |

**Syntax Rules:**

1. Enable minification (`optimization.minimize: true` in webpack, which uses TerserPlugin).
2. Use `process.env.NODE_ENV` guards for development-only code; Terser replaces the expression with `"production"` and removes the dead branch.
3. Configure Terser options to drop console and debugger statements if desired.
4. DCE runs after tree shaking, so both must be enabled for maximum effect.

**Constraints and Limitations:**

- DCE cannot remove code with observable side effects unless the minifier can prove purity.
- Module boundaries preserved by the bundler can limit Terser’s analysis.
- Overly aggressive DCE (e.g., dropping all console calls) can remove error reporting; configure carefully.

### Multiple Annotated Code Examples

#### Example 1: Development-Only Branch Elimination

```jsx
if (process.env.NODE_ENV === 'development') {
  console.log('Debug mode');
  initDevTools();
}
```

```js
// After Terser in production mode:
// The entire if-block is removed because process.env.NODE_ENV
// is replaced with "production" and the condition is false.
```

**Expected Output:** The development-only logging and `initDevTools` call are absent from the production bundle.

**Why this output:** Terser performs constant folding on `process.env.NODE_ENV` and removes the unreachable branch.

#### Example 2: Configuring Terser to Drop Console Calls

```js
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true,
          },
        },
      }),
    ],
  },
};
```

**Expected Output:** `console.*` and `debugger` statements are removed from the production bundle.

**Why this output:** The `drop_console` and `drop_debugger` compress options instruct Terser to remove these calls during minification.

### Real-World Cases

- **Production builds:** Removing development warnings and debug tooling.
- **Feature flags:** Eliminating code paths for disabled features at build time.
- **Library builds:** Producing smaller distributable bundles.

### References

- Dead code elimination (Tooling.Report) — https://bundlers.tooling.report
- webpack Optimization: `minimize` and TerserPlugin — https://v4.webpack.js.org/configuration/optimization/
- Strip debug-only code from production bundles — https://raw.githubusercontent.com/pproenca/dot-skills

---

## 4. Minification

### Definitions

**Core Definition:** Minification is the process of reducing source code size by removing whitespace, comments, and shortening identifiers without changing behaviour.

**Technical Definition:** Minification transforms JavaScript (and CSS) into a semantically equivalent but more compact form. It removes whitespace, comments, and unnecessary semicolons, shortens local variable and function names, and performs lightweight optimisations such as constant folding. In webpack, minification is controlled by `optimization.minimize` and is `true` by default in production mode; it uses TerserPlugin. Vite uses esbuild (or OXC) for minification by default, with Terser available for maximum compression. Minification reduces the parsed and executed size, which directly affects main-thread blocking and INP.

**Beginner-Friendly Explanation:** Minification makes your code files smaller by removing spaces, comments, and long variable names. The code behaves exactly the same, but it downloads faster and the browser can parse it more quickly.

### Purposes

- To reduce the raw and parsed size of JavaScript and CSS.
- To lower network transfer time (before compression).
- To reduce parse and execution time on the main thread.
- To complement compression for the smallest possible delivered payload.

### Syntax Rules and Structure

**Minification Configuration (webpack):**

```js
module.exports = {
  mode: 'production',
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin()],
  },
};
```

**Minification Configuration (Vite):**

```js
// vite.config.js
export default defineConfig({
  build: {
    minify: 'terser', // or 'esbuild' (default) or 'oxc'
    terserOptions: {
      compress: { drop_console: true },
    },
  },
});
```

**Syntax Rules:**

1. Enable production mode (`mode: 'production'` in webpack, default in Vite build).
2. `optimization.minimize` defaults to `true` in production mode.
3. Choose a minifier: Terser for maximum compression, esbuild/OXC for speed.
4. Minification should run after tree shaking and DCE.

**Constraints and Limitations:**

- Minification increases build time; Terser is slower than esbuild/OXC.
- Aggressive options (e.g., `drop_console`) can remove error reporting; configure deliberately.
- Minified code requires source maps for debugging.

### Multiple Annotated Code Examples

#### Example 1: Webpack Terser Configuration

```js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'production',
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true,
        terserOptions: {
          compress: { drop_console: true },
        },
      }),
    ],
  },
};
```

**Expected Output:** The production bundle is minified with Terser, and `console` calls are removed.

**Why this output:** `optimization.minimize` enables TerserPlugin. The `terserOptions` customize the compression behaviour.

#### Example 2: Vite Minification with Terser

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
});
```

**Expected Output:** Vite uses Terser instead of the default esbuild minifier, producing a smaller bundle at the cost of slower builds.

**Why this output:** Setting `minify: 'terser'` switches the minifier. The `terserOptions` configure dead code elimination of console and debugger statements.

### Real-World Cases

- **All production React applications:** Minification is a baseline requirement.
- **Library authors:** Minified distributions are smaller and faster to download.
- **Performance-sensitive applications:** Terser with tuned options can squeeze out additional bytes.

### References

- webpack Optimization: `minimize` defaults to true in production — https://v4.webpack.js.org/configuration/optimization/
- React + Vite Best Practices: minification (OXC default, Terser for max compression) — https://raw.githubusercontent.com/voku/agent-skills/eaaf04fa3e1192e34e93f5bda46e5d85a8ccfc1a/skills/react-vite-best-practices/AGENTS.md
- terser-webpack-plugin — https://www.npmjs.com/package/terser-webpack-plugin

---

## 5. Compression

### Definitions

**Core Definition:** Compression is the process of encoding bundle assets with an algorithm (gzip or Brotli) to reduce their transfer size over the network.

**Technical Definition:** HTTP compression reduces the number of bytes sent over the wire by encoding the response body with a compression algorithm. The browser advertises supported algorithms via the `Accept-Encoding` header, and the server responds with a compressed body and the corresponding `Content-Encoding` header. **gzip** is universally supported; **Brotli** achieves roughly 10–20% better compression ratios than gzip for JavaScript and HTML, but is supported only by modern browsers. Compression can be applied at build time (pre-compression) using `compression-webpack-plugin` or Vite plugins, or at the server/CDN edge. Pre-compression avoids per-request CPU cost and allows the CDN to serve the pre-compressed assets directly.

**Beginner-Friendly Explanation:** Compression shrinks your files before they travel over the internet. gzip is the safe, universal option; Brotli is newer and compresses even better, but older browsers may not support it. If you use a CDN, it can often compress files automatically, or you can pre-compress them during the build.

### Purposes

- To reduce network transfer time and bandwidth cost.
- To improve load time, especially on slow connections.
- To allow the browser to receive and parse the bundle sooner.
- To complement minification for the smallest delivered payload.

### Syntax Rules and Structure

**Compression Options:**

| Algorithm | Browser support | Compression ratio | When to use |
|-----------|----------------|-------------------|-------------|
| gzip | All browsers | Baseline | Fallback for all clients |
| Brotli | Modern browsers | ~10–20% better than gzip | Preferred for static assets |
| Zstandard | Emerging | Competitive | Experimental / CDN-specific |

**Syntax Rules:**

1. Enable compression at the server or CDN level (`Content-Encoding: gzip` or `br`).
2. Pre-compress static assets at build time for CDN delivery.
3. Serve Brotli when the browser supports it; fall back to gzip otherwise.
4. Do not compress already-compressed formats (images, fonts); it wastes CPU without benefit.

**Constraints and Limitations:**

- Pre-compression generates companion files (`.gz`, `.br`) that must be served with the correct `Content-Encoding`.
- `compression-webpack-plugin` defaults to gzip-only; Brotli requires additional configuration or a different plugin.
- Very small files may not benefit from compression; set a size threshold.

### Multiple Annotated Code Examples

#### Example 1: Pre-Compression with `compression-webpack-plugin`

```js
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  plugins: [
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240, // Only compress files > 10KB
      minRatio: 0.8,
    }),
  ],
};
```

**Expected Output:** `.gz` files are emitted alongside the original assets. When served with `Content-Encoding: gzip`, the browser decompresses them transparently.

**Why this output:** The plugin compresses matching assets at build time, producing pre-compressed files that the CDN can serve without per-request CPU cost.

#### Example 2: Brotli Compression with a CDN

```text
# Conceptual CDN configuration
# Enable Brotli and gzip in the CDN dashboard.
# The CDN negotiates the algorithm based on Accept-Encoding.
# Pre-compressed .br and .gz files are served directly.
```

**Expected Output:** Browsers that support Brotli receive `.br` files; older browsers receive `.gz` files.

**Why this output:** The CDN inspects the `Accept-Encoding` header and selects the best available pre-compressed variant.

### Real-World Cases

- **CDN-delivered SPAs:** Pre-compressed assets served from edge locations.
- **Static hosting (Vercel, Netlify):** Automatic Brotli/gzip compression at the edge.
- **Self-hosted servers (Nginx):** `gzip` and `brotli` modules configured for dynamic compression.

### References

- Pre-compress assets to gzip and Brotli for CDN — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/webpack-plugin-recipes/references/assets-pre-compress-gzip-brotli.md
- CompressionWebpackPlugin (webpack) — https://v4.webpack.js.org/plugins/compression-webpack-plugin/
- GZIP vs Brotli Compression: Which One Is Best for Web Performance in 2025? — https://www.ioriver.io
- Vercel CDN Compression — https://vercel.com

---

## 6. Bundle Analysis

### Definitions

**Core Definition:** Bundle analysis is the process of inspecting the contents of a built bundle to identify which modules and dependencies contribute to its size.

**Technical Definition:** Bundle analysis tools parse the bundler’s output (and optionally its stats file or source maps) and produce visualisations — treemaps, sunbursts, flamegraphs, or network graphs — that map bundle bytes to source modules. The goal is to identify unexpectedly large dependencies, duplicate packages, and modules that were included by mistake. Analysis is the prerequisite for targeted optimization: without it, optimization becomes guesswork.

**Beginner-Friendly Explanation:** Bundle analysis is like opening up your bundle and looking at what is inside. It shows you which libraries are taking up the most space, so you know what to fix.

### Purposes

- To identify large or unexpected dependencies.
- To detect duplicate packages (e.g., two copies of React).
- To verify that tree shaking and code splitting are working.
- To prioritize optimization efforts based on data.

### Syntax Rules and Structure

**Analysis Workflow:**

| Step | Action |
|------|--------|
| 1. Build for production | Run the production build with source maps enabled |
| 2. Run the analyzer | Use the tool for your bundler |
| 3. Inspect the report | Look for large or unexpected modules |
| 4. Act | Replace, tree-shake, lazy-load, or remove |

**Syntax Rules:**

1. Always analyse the production build, not development.
2. Enable source maps so the analyzer can map bytes to source modules.
3. Compare analysis reports across releases to catch regressions.
4. Use the tool appropriate to your bundler (webpack, Vite/Rollup, or framework-specific).

### Annotated Code Examples

```bash
# webpack: generate a stats file and analyse it
npx webpack --profile --json > stats.json
npx webpack-bundle-analyzer stats.json

# Vite: add rollup-plugin-visualizer and build
npx vite build
# Open the generated stats.html
```

**Expected Output:** A treemap visualization showing the size contribution of each module.

**Why this output:** The analyzer parses the stats file (webpack) or the bundle output (Vite) and maps bytes to modules, producing a size hierarchy.

### Real-World Cases

- **Detecting duplicate React:** Source Map Explorer can reveal two copies of React caused by out-of-date dependencies.
- **Finding accidental inclusions:** Large libraries pulled in by a single import.
- **Verifying code splitting:** Confirming that route chunks are separate.

### References

- webpack-bundle-analyzer — https://www.npmjs.com/package/webpack-bundle-analyzer
- rollup-plugin-visualizer — https://www.npmjs.com/package/rollup-plugin-visualizer
- source-map-explorer — https://raw.githubusercontent.com/eins78/source-map-explorer/master/README.md

---

## 7. Webpack Bundle Analyzer

### Definitions

**Core Definition:** `webpack-bundle-analyzer` is a webpack plugin and CLI utility that produces an interactive, zoomable treemap visualisation of the contents of webpack bundles.

**Technical Definition:** `webpack-bundle-analyzer` parses webpack’s stats output and generates a treemap where each rectangle’s area represents a module’s size in the bundle. It supports minified bundles and displays gzipped, Brotli, and Zstandard sizes. It can run as a plugin (starting an HTTP server or emitting a static HTML file) or as a CLI utility against a pre-generated stats file.

**Beginner-Friendly Explanation:** Webpack Bundle Analyzer is a tool that shows you a map of your webpack bundle. Each block in the map is a module, and bigger blocks mean bigger modules. It helps you spot what is taking up the most space.

### Purposes

- To visualise what is inside a webpack bundle.
- To identify the modules that contribute most to size.
- To detect modules that were included by mistake.
- To compare gzipped, Brotli, and Zstandard sizes.

### Syntax Rules and Structure

**Installation:**

```bash
npm install --save-dev webpack-bundle-analyzer
```

**Plugin Usage:**

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [new BundleAnalyzerPlugin()],
};
```

**Syntax Rules:**

1. Add `BundleAnalyzerPlugin` to the `plugins` array.
2. The default `analyzerMode` is `server`, which starts an HTTP server and opens the report in a browser.
3. Use `analyzerMode: 'static'` to emit a single HTML report file.
4. Use `defaultSizes: 'gzip'` or `'brotli'` to display compressed sizes by default.

**Constraints and Limitations:**

- Designed for webpack; Vite/Rollup use `rollup-plugin-visualizer`.
- The report can be large for very big bundles.
- Source maps are not required, but they help map minified code to source.

### Annotated Code Examples

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
      defaultSizes: 'gzip',
      openAnalyzer: false,
    }),
  ],
};
```

**Expected Output:** A `bundle-report.html` file is generated, containing an interactive treemap of the bundle with gzip sizes.

**Why this output:** `analyzerMode: 'static'` produces a self-contained HTML report. `defaultSizes: 'gzip'` shows the compressed sizes, which reflect the transfer cost.

### Real-World Cases

- **Webpack-based React applications:** Identifying large vendor chunks.
- **Monorepos:** Detecting duplicate dependencies across packages.
- **CI pipelines:** Generating a report on every release and comparing sizes.

### References

- webpack-bundle-analyzer (npm) — https://www.npmjs.com/package/webpack-bundle-analyzer

---

## 8. Source Map Explorer

### Definitions

**Core Definition:** `source-map-explorer` is a CLI tool that analyses JavaScript bundles using their source maps to determine which original source files contribute to each byte of the minified output.

**Technical Definition:** `source-map-explorer` reads a minified bundle and its source map, maps every byte in the minified output back to the original source file that produced it, and displays a treemap visualisation of the size contribution. It is bundler-agnostic: it works with any tool that produces a source map. This makes it useful for detecting duplicate packages (e.g., two copies of React) that bundler-specific analyzers might obscure.

**Beginner-Friendly Explanation:** Source Map Explorer uses source maps to show you exactly which source files your minified code came from. It is especially good at finding duplicate copies of the same library.

### Purposes

- To map minified bytes back to original source files.
- To detect duplicate packages in the bundle.
- To debug code bloat from any bundler that emits source maps.
- To provide a bundler-agnostic alternative to webpack-specific analyzers.

### Syntax Rules and Structure

**Installation:**

```bash
npm install -g source-map-explorer
```

**Usage:**

```bash
source-map-explorer bundle.min.js
source-map-explorer bundle.min.js bundle.min.js.map
```

**Syntax Rules:**

1. Generate source maps in the production build.
2. Point `source-map-explorer` at the minified bundle and its source map.
3. The tool opens an HTML treemap visualization in the browser.
4. Use `--json` or `--tsv` for machine-readable output.

**Constraints and Limitations:**

- Requires source maps; without them, it cannot map bytes to source.
- Source maps must be accurate and map back to the original files.
- Not specific to any bundler; it works wherever source maps are available.

### Annotated Code Examples

```bash
# Build with source maps
npm run build

# Analyse the bundle
source-map-explorer 'dist/assets/*.js'
```

**Expected Output:** A treemap visualization opens in the browser, showing which source files contribute to the bundle size.

**Why this output:** The tool parses the source map, maps each byte in the minified bundle to its original source file, and renders the result as a treemap.

### Real-World Cases

- **Detecting duplicate React:** A bundle with two copies of React shows two large React blocks in the treemap.
- **Bundler-agnostic analysis:** Using the same tool across webpack, Vite, and Rollup projects.
- **CI reporting:** Generating JSON output and tracking size changes over time.

### References

- source-map-explorer (README) — https://raw.githubusercontent.com/eins78/source-map-explorer/master/README.md
- Source Map Explorer (Create React App docs) — https://cdn.jsdelivr.net

---

## 9. Vite Bundle Analyzer (`rollup-plugin-visualizer`)

### Definitions

**Core Definition:** `rollup-plugin-visualizer` is a Rollup/Vite plugin that visualises the bundle contents, showing which modules are taking up space.

**Technical Definition:** `rollup-plugin-visualizer` hooks into Rollup’s (and therefore Vite’s) build pipeline and generates a report of module sizes in multiple formats: treemap, sunburst, flamegraph, network graph, raw JSON, list (YAML), and Markdown. It colours project files blue and dependencies green, making it easy to distinguish application code from `node_modules`. It is the standard bundle analysis tool for Vite projects.

**Beginner-Friendly Explanation:** Rollup Plugin Visualizer is the bundle analyzer for Vite and Rollup. It shows you a map of your bundle and tells you which modules are the biggest. It can also show why a module was included.

### Purposes

- To visualise Vite/Rollup bundle contents.
- To identify large dependencies and unexpected inclusions.
- To track bundle changes over time using list or Markdown reports.
- To understand why a module is included via the network view.

### Syntax Rules and Structure

**Installation:**

```bash
npm install --save-dev rollup-plugin-visualizer
```

**Vite Usage:**

```js
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [visualizer()],
});
```

**Syntax Rules:**

1. Add `visualizer()` to the `plugins` array; keep it last.
2. The default template is `treemap`; other options include `sunburst`, `flamegraph`, `network`, `raw-data`, `list`, and `markdown`.
3. Use `filename` to control the report file name (default `stats.{ext}`).
4. Use `open: true` to automatically open the report in the browser.

**Constraints and Limitations:**

- Requires Node.js >= 22.
- The network view can be noisy for large projects; filter helper nodes to explore clusters.
- List and Markdown reports are better suited for CI tracking than interactive exploration.

### Annotated Code Examples

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      filename: 'bundle-report.html',
      template: 'treemap',
      open: true,
    }),
  ],
});
```

**Expected Output:** After `vite build`, a `bundle-report.html` file is generated and opened in the browser, showing an interactive treemap of the bundle.

**Why this output:** The plugin hooks into the build and generates a report from the module graph. Project files are blue, dependencies are green.

### Real-World Cases

- **Vite-based React applications:** Standard bundle analysis workflow.
- **CI size tracking:** Using the `list` or `markdown` template to commit size reports and detect regressions.
- **Investigating why a module is included:** The network view shows import relationships.

### References

- rollup-plugin-visualizer (npm) — https://www.npmjs.com/package/rollup-plugin-visualizer
- Vite bundle analysis reference — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/vite/references/bundle-analyze.md
- React + Vite Best Practices (bundle analysis section) — https://raw.githubusercontent.com/voku/agent-skills/eaaf04fa3e1192e34e93f5bda46e5d85a8ccfc1a/skills/react-vite-best-practices/AGENTS.md

---

## 10. Bundle Optimization Best Practices

### Definitions

**Core Definition:** Bundle optimization best practices are the guidelines for systematically reducing bundle size while preserving correctness, caching efficiency, and maintainability.

**Technical Definition:** Best practices combine module-format discipline (ESM, `sideEffects`), production-mode configuration (tree shaking, minification, DCE), delivery optimization (compression, caching headers), code splitting, vendor chunking, and continuous analysis. The React + Vite best-practices reference groups these into build optimization, code splitting, and bundle analysis categories, with manual chunk splitting, minification (OXC default, Terser for max compression), tree shaking, gzip/Brotli compression, and content-based asset hashing as critical build optimizations. Analysis is treated as a continuous practice, not a one-off task.

**Beginner-Friendly Explanation:** Bundle optimization is not one trick; it is a set of habits. Use modern module syntax, build in production mode, split your code, compress your files, cache aggressively, and analyse the result regularly. Do these things consistently and your bundle stays small.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use ES modules** | Tree shaking requires static `import`/`export`. |
| **Set accurate `sideEffects`** | Allows aggressive tree shaking of pure modules. |
| **Enable production mode** | Turns on tree shaking and minification by default. |
| **Use named imports** | Avoid namespace imports and barrel files. |
| **Minify with Terser or OXC** | Terser for max compression; OXC/esbuild for speed. |
| **Pre-compress with gzip and Brotli** | Serve `.br` to modern browsers, `.gz` as fallback. |
| **Split at route boundaries** | Route-based code splitting reduces the initial bundle by 40–70%. |
| **Separate vendor chunks** | Isolate React and other stable dependencies for long-term caching. |
| **Use content-based hashing** | Enables long-term caching and safe cache busting. |
| **Analyse the bundle continuously** | Use webpack-bundle-analyzer, source-map-explorer, or rollup-plugin-visualizer. |
| **Set and enforce budgets** | Fail CI when the bundle exceeds a configured size limit. |
| **Avoid barrel files** | Barrel files can add 200–800ms of import cost and defeat tree shaking. |
| **Prefer tree-shakeable libraries** | Choose `lodash-es` over `lodash`, `lucide-react` over icon monoliths. |

### Annotated Code Examples

**✅ Good Practices (Vite):**

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ filename: 'stats.html', template: 'treemap' }),
  ],
  build: {
    minify: 'terser', // Max compression
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router-dom'],
        },
      },
    },
  },
});
```

**❌ Bad Practices:**

```js
// ❌ CommonJS: not tree-shakeable
const { Button } = require('./components');

// ❌ Namespace import: imports everything
import * as Icons from 'react-icons/fa';

// ❌ No production mode: tree shaking and minification disabled
mode: 'development'
```

**Expected Output:** The good configuration produces a minified, tree-shaken, code-split, vendor-chunked, analysable bundle. The bad practices produce a large, untree-shaken bundle.

**Why this output:** ES modules enable tree shaking; production mode enables DCE and minification; manual chunks enable caching; the visualizer enables continuous analysis.

### Real-World Cases

- **Vite + React projects:** The complete pipeline from the React + Vite best-practices reference.
- **Webpack + React projects:** Production mode, Terser, SplitChunksPlugin, compression-webpack-plugin.
- **Enterprise design systems:** Continuous bundle analysis and size budgets to prevent regressions.

### References

- React + Vite Best Practices (build optimization, code splitting, bundle analysis) — https://raw.githubusercontent.com/voku/agent-skills/eaaf04fa3e1192e34e93f5bda46e5d85a8ccfc1a/skills/react-vite-best-practices/AGENTS.md
- How to tree-shake in React · CoreUI — https://coreui.io/answers/how-to-tree-shake-in-react/
- Bundle optimization rules (critical impact levels) — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/react-optimise/AGENTS.md
- GZIP vs Brotli compression best practices — https://www.ioriver.io
- webpack-bundle-analyzer — https://www.npmjs.com/package/webpack-bundle-analyzer
- rollup-plugin-visualizer — https://www.npmjs.com/package/rollup-plugin-visualizer
- source-map-explorer — https://raw.githubusercontent.com/eins78/source-map-explorer/master/README.md
- Pre-compress assets to gzip and Brotli — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/webpack-plugin-recipes/references/assets-pre-compress-gzip-brotli.md
- webpack Optimization — https://v4.webpack.js.org/configuration/optimization/
- Dead code elimination (Tooling.Report) — https://bundlers.tooling.report
- Tree shaking Support Guide — https://raw.githubusercontent.com/web3/web3.js/53a62eda5dcb240c3f684cf9f8ef491611ec4339/docs/docs/guides/advanced/tree_shaking.md
- Production-mode requirement for tree shaking — https://github.com/vanguard-frontier-agentic/skills/blob/master/frontend/tree-shaking-dead-code-review/references/module-format-and-sideeffects.md
- Vite bundle analysis reference — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/vite/references/bundle-analyze.md