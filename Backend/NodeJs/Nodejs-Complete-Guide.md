# Node.js Complete Interview Guide

One revision document for senior Node.js interviews: 75 topics in 14 parts. ⭐ marks the topics asked most often in senior interviews.

**How to use this guide**

- First read: go top to bottom. Each part builds on the one before it.
- Quick revision: read the **Quick revision** block at the end of each topic, then the [Cheat Sheet](#75-cheat-sheet).
- Night before an interview: read [Must-Know Top 30 Questions](#70-must-know-top-30-questions).

**Every topic follows the same structure**

1. Why it matters
2. Detailed explanation, with an example for every subtopic
3. Real-time scenario: problem → diagnosis → fix
4. Common mistakes and trade-offs
5. Top interview questions, all answered
6. Quick revision

Part 14 is a question bank, so it uses its own format.

---

## The Big Picture

Connect the topics as dots. Each part answers one question and leads to the next.

```mermaid
flowchart TD
    P1["1. Foundations<br/>What is Node?"] --> P2["2. Runtime internals<br/>How does it run?"]
    P2 --> P3["3. Async programming<br/>How do I write non-blocking code?"]
    P3 --> P4["4. Core modules<br/>What does it ship with?"]
    P4 --> P5["5. Concurrency<br/>How do I use all CPU cores?"]
    P5 --> P6["6. APIs and web apps<br/>How do I build services?"]
    P6 --> P7["7. Data layer<br/>How do I store and move data?"]
    P7 --> P8["8. Security<br/>How do I protect it?"]
    P8 --> P9["9. Reliability and testing<br/>How do I keep it correct?"]
    P9 --> P10["10. Performance and debugging<br/>How do I find what is slow?"]
    P10 --> P11["11. Architecture and scaling<br/>How does it grow?"]
    P11 --> P12["12. Deployment and observability<br/>How do I run it?"]
    P12 --> P13["13. Modern Node.js<br/>What is new?"]
    P13 --> P14["14. Interview revision kit"]
```

**The one idea that connects everything:** Node runs your JavaScript on one main thread. Almost every senior question comes back to this.

| Question | Why the single thread is the answer |
|---|---|
| Why is Node good at I/O? | The thread never waits. It hands I/O to the OS or libuv and moves on. |
| Why is Node bad at CPU work? | Heavy computation blocks the only thread, so every request waits. |
| Why do streams matter? | Holding big data in memory hurts one process serving many users. |
| Why worker threads and cluster? | They add the parallelism a single thread doesn't have. |
| Why does a sync call in a handler cause an outage? | It stops the loop for every connected user. |

---

## Table of Contents

**[Part 1: Foundations](#part-1-foundations)**

- [1. What Node.js Is](#1-what-nodejs-is)
- [2. Node.js vs Browser JavaScript](#2-nodejs-vs-browser-javascript)
- [3. ⭐ Module Systems](#3--module-systems)
- [4. npm and Package Management](#4-npm-and-package-management)

**[Part 2: Runtime Internals](#part-2-runtime-internals)**

- [5. ⭐ Node.js Architecture](#5--nodejs-architecture)
- [6. ⭐ Event Loop](#6--event-loop)
- [7. V8 Engine](#7-v8-engine)
- [8. ⭐ Memory and Garbage Collection](#8--memory-and-garbage-collection)

**[Part 3: Async Programming](#part-3-async-programming)**

- [9. Callbacks](#9-callbacks)
- [10. ⭐ Promises](#10--promises)
- [11. ⭐ Async/Await](#11--asyncawait)
- [12. ⭐ EventEmitter](#12--eventemitter)
- [13. Cancellation and Timeouts](#13-cancellation-and-timeouts)
- [14. Async Context](#14-async-context)

**[Part 4: Core Modules and Data Handling](#part-4-core-modules-and-data-handling)**

- [15. ⭐ Buffers and Binary Data](#15--buffers-and-binary-data)
- [16. ⭐ Streams](#16--streams)
- [17. ⭐ Backpressure](#17--backpressure)
- [18. File System and Path](#18-file-system-and-path)
- [19. HTTP and Networking](#19-http-and-networking)
- [20. Crypto](#20-crypto)
- [21. Process and Environment](#21-process-and-environment)

**[Part 5: Concurrency and Parallelism](#part-5-concurrency-and-parallelism)**

- [22. ⭐ Worker Threads](#22--worker-threads)
- [23. ⭐ Child Processes](#23--child-processes)
- [24. ⭐ Cluster Module](#24--cluster-module)
- [25. Concurrency Comparison](#25-concurrency-comparison)

**[Part 6: Building APIs and Web Apps](#part-6-building-apis-and-web-apps)**

- [26. ⭐ Express](#26--express)
- [27. Framework Comparison](#27-framework-comparison)
- [28. ⭐ REST API Design](#28--rest-api-design)
- [29. GraphQL](#29-graphql)
- [30. gRPC](#30-grpc)
- [31. ⭐ Real-Time Communication](#31--real-time-communication)
- [32. File Uploads and Large Payloads](#32-file-uploads-and-large-payloads)
- [33. Validation and Config](#33-validation-and-config)
- [34. TypeScript with Node.js](#34-typescript-with-nodejs)

**[Part 7: Data Layer](#part-7-data-layer)**

- [35. ⭐ Database Access](#35--database-access)
- [36. ⭐ Connection Pooling](#36--connection-pooling)
- [37. Transactions and Migrations](#37-transactions-and-migrations)
- [38. ⭐ Caching](#38--caching)
- [39. Background Jobs](#39-background-jobs)
- [40. ⭐ Message Queues](#40--message-queues)

**[Part 8: Security](#part-8-security)**

- [41. ⭐ Authentication](#41--authentication)
- [42. Authorization](#42-authorization)
- [43. ⭐ OWASP for Node.js](#43--owasp-for-nodejs)
- [44. API Hardening](#44-api-hardening)
- [45. Supply Chain Security](#45-supply-chain-security)
- [46. ⭐ Rate Limiting](#46--rate-limiting)

**[Part 9: Reliability and Testing](#part-9-reliability-and-testing)**

- [47. ⭐ Error Handling](#47--error-handling)
- [48. ⭐ Graceful Shutdown](#48--graceful-shutdown)
- [49. Resilience Patterns](#49-resilience-patterns)
- [50. ⭐ Testing Strategy](#50--testing-strategy)
- [51. Testing Tools](#51-testing-tools)

**[Part 10: Performance and Debugging](#part-10-performance-and-debugging)**

- [52. ⭐ Profiling](#52--profiling)
- [53. ⭐ Debugging Memory Leaks](#53--debugging-memory-leaks)
- [54. Debugging Production Issues](#54-debugging-production-issues)
- [55. Performance Optimization](#55-performance-optimization)

**[Part 11: Architecture and Scaling](#part-11-architecture-and-scaling)**

- [56. ⭐ Scaling Node.js](#56--scaling-nodejs)
- [57. ⭐ Monolith vs Microservices](#57--monolith-vs-microservices)
- [58. Distributed Patterns](#58-distributed-patterns)
- [59. ⭐ Design Patterns in Node](#59--design-patterns-in-node)
- [60. Project Structure](#60-project-structure)
- [61. Serverless Node.js](#61-serverless-nodejs)

**[Part 12: Deployment and Observability](#part-12-deployment-and-observability)**

- [62. Docker for Node.js](#62-docker-for-nodejs)
- [63. Kubernetes Basics](#63-kubernetes-basics)
- [64. CI/CD](#64-cicd)
- [65. ⭐ Observability](#65--observability)

**[Part 13: Modern Node.js](#part-13-modern-nodejs)**

- [66. Recent Built-In Features](#66-recent-built-in-features)
- [67. Runtime Comparison](#67-runtime-comparison)
- [68. Native Extensions](#68-native-extensions)
- [69. AI in Node.js Backends](#69-ai-in-nodejs-backends)

**[Part 14: Interview Revision Kit](#part-14-interview-revision-kit)**

- [70. Must-Know Top 30 Questions](#70-must-know-top-30-questions)
- [71. Output-Based Questions](#71-output-based-questions)
- [72. Coding Tasks](#72-coding-tasks)
- [73. Production Scenarios](#73-production-scenarios)
- [74. Node.js System Design](#74-nodejs-system-design)
- [75. Cheat Sheet](#75-cheat-sheet)

---

# Part 1: Foundations

## 1. What Node.js Is

### Why it matters
Interviewers open with this to see whether you understand Node as a system or only as "JavaScript on the server". A strong answer names the parts and says when Node is the wrong choice.

### Explanation

#### V8 + libuv
Node.js is a runtime built from two main parts:

- **V8** is Google's JavaScript engine. It compiles and runs your JS.
- **libuv** is a C library that provides the event loop, async I/O and a thread pool.

Node adds C++ bindings between them, plus a standard library (`fs`, `http`, `crypto`).

```javascript
const fs = require('node:fs');

// V8 runs this line of JavaScript.
console.log('reading file');

// fs is Node's library. libuv does the read on its thread pool.
fs.readFile(__filename, 'utf8', (err, data) => {
  // libuv signals completion. The event loop calls this function, and V8 runs it.
  console.log('file length:', data.length);
});

console.log('this prints before the file length');
```

#### Runtime vs language
JavaScript is the language, defined by the ECMAScript spec. A **runtime** is the environment that runs it and gives it extra abilities. `Array.map` is the language. `fs.readFile` is the Node runtime. `document.querySelector` is the browser runtime.

```javascript
// Language: works in any JavaScript runtime
[1, 2, 3].map((n) => n * 2);

// Node runtime only
require('node:os').cpus().length;

// Browser runtime only: throws ReferenceError in Node
// document.querySelector('h1');
```

#### Where Node fits
Node is strong when work is mostly **waiting on I/O**: APIs, gateways, real-time apps, streaming and BFF layers.

```javascript
// Typical good fit: the thread is free during both awaits.
// While these wait, the same process serves thousands of other requests.
app.get('/dashboard', async (req, res) => {
  const [user, orders] = await Promise.all([
    db.users.findById(req.userId),
    ordersClient.list(req.userId),
  ]);
  res.json({ user, orders });
});
```

#### Where Node doesn't fit
Node is weak for **sustained CPU work** on the main thread: video encoding, large matrix maths, image processing and ML training.

```javascript
// Bad fit: this blocks every other request until it ends.
app.get('/primes', (req, res) => {
  let count = 0;
  for (let i = 2; i < 5_000_000; i++) {
    let prime = true;
    for (let j = 2; j * j <= i; j++) {
      if (i % j === 0) { prime = false; break; }
    }
    if (prime) count++;
  }
  res.json({ count });
});
// Fix: move this to a worker thread (Topic 22) or a separate service.
```

### Real-time scenario
**Problem:** A team builds a thumbnail service in Express. Under load, even `/health` times out and Kubernetes restarts the pods.
**Diagnosis:** Image resizing ran in pure JS on the main thread, so the event loop was blocked and health checks couldn't respond.
**Fix:** Move resizing to a worker pool, or to a separate job consumer. The API now only accepts uploads and returns a job ID.

### Common mistakes and trade-offs
- Calling Node "single-threaded" without nuance. Your JS is single-threaded, but libuv and V8 use other threads.
- Calling Node a framework or a language. It's a runtime.
- **Trade-off:** one language across the stack and great I/O concurrency, in exchange for careful handling of CPU work.

### Top interview questions
**Q1. What is Node.js?**
A JavaScript runtime built on V8 and libuv. V8 runs the code, and libuv provides the event loop and async I/O, which lets one process handle many concurrent connections.

**Q2. Is Node.js single-threaded?**
Your JavaScript runs on one main thread, but the process has several threads: the libuv pool, V8's GC and compiler threads, and any worker threads you create.

**Q3. Why is Node good for I/O-heavy applications?**
It doesn't dedicate a thread to each request. While waiting on I/O, the thread serves other requests, so memory and context-switching costs stay low.

**Q4. When would you not choose Node?**
For sustained CPU-bound work, such as video encoding or heavy numeric computing. Go, Rust or Python with native libraries are often better there. Node can still orchestrate the work.

**Q5. What is the difference between JavaScript and Node.js?**
JavaScript is a language. Node is a runtime that runs it outside the browser and adds APIs for files, networking and processes.

**Q6. What role does libuv play?**
It provides the event loop, abstracts OS async mechanisms (epoll, kqueue, IOCP), and runs a thread pool for work without a good async OS API, such as file I/O, `dns.lookup` and some crypto.

### Quick revision
Node = V8 (runs JS) + libuv (event loop, async I/O) + standard library. It's excellent for I/O and poor for CPU work on the main thread.

---

## 2. Node.js vs Browser JavaScript

### Why it matters
The same language behaves differently in the two runtimes. Knowing the differences helps with SSR, shared libraries and full-stack debugging.

### Explanation

#### Globals
The browser's main global is `window`. Node has `global`. Node's CommonJS modules also get file-level values that look global but aren't.

```javascript
console.log(typeof window);     // 'undefined' in Node
console.log(typeof global);     // 'object' in Node
console.log(__dirname);         // CommonJS only: the folder of this file
console.log(__filename);        // CommonJS only: the path of this file

// A top-level var in a module stays private to that module.
var count = 1;
console.log(global.count);      // undefined in Node
```

#### APIs
Each runtime offers what its environment needs. Many Web APIs now exist in both.

| Area | Browser | Node.js |
|---|---|---|
| UI | `document`, DOM | None |
| Files | Limited File API | `fs` |
| Processes | None | `process`, `child_process` |
| Shared | `fetch`, `URL`, `AbortController`, `crypto.subtle`, `TextEncoder` | The same, built in |
| Storage | `localStorage`, IndexedDB | Files, databases |

```javascript
// Runs unchanged in both runtimes
const url = new URL('https://example.com/search?q=node');
console.log(url.searchParams.get('q')); // 'node'
```

#### `process`
`process` is Node's view of the running program. Browsers have nothing similar.

```javascript
console.log(process.pid);            // process ID
console.log(process.version);        // Node version
console.log(process.argv.slice(2));  // command-line arguments
console.log(process.env.NODE_ENV);   // environment variable
console.log(process.memoryUsage().heapUsed);
console.log(process.uptime());       // seconds since start
```

#### `globalThis`
`globalThis` is the standard name for the global object in every runtime, so shared code doesn't need runtime checks.

```javascript
// Old approach for code shared between runtimes
const root = typeof window !== 'undefined' ? window : global;

// Modern approach
globalThis.appVersion = '1.0.0';
console.log(globalThis.appVersion);

// Safe runtime check, used in code shared by SSR and browser
const isBrowser = typeof globalThis.document !== 'undefined';
```

### Real-time scenario
**Problem:** A Next.js page crashes during server rendering with `ReferenceError: window is not defined`.
**Diagnosis:** A shared helper reads `window.localStorage` at import time, and that import runs on the server.
**Fix:** Check `typeof globalThis.window !== 'undefined'` first, or move the access into code that only runs in the browser.

### Common mistakes and trade-offs
- Assuming `this` at the top of a CommonJS file is the global object. It's `module.exports`.
- Using `__dirname` in ES Modules. Use `import.meta.dirname` instead.
- **Trade-off:** Web-standard APIs make code portable, while Node-specific APIs often offer more control.

### Top interview questions
**Q1. What are the main differences between Node and browser JavaScript?**
Same language, different environment. Browsers provide the DOM and a sandbox. Node provides files, networking and process access, with no DOM.

**Q2. What is `globalThis`?**
A standard reference to the global object in any runtime, added in ES2020. It replaces checks for `window`, `global` or `self`.

**Q3. Why isn't a top-level `var` global in Node?**
Each CommonJS file is wrapped in a function, so `var` is local to that function. ES Modules also have their own scope.

**Q4. What is `this` at the top level of a Node file?**
In CommonJS it's `module.exports`. In ES Modules it's `undefined`.

**Q5. Are `__dirname` and `__filename` truly global?**
No. They're parameters of the CommonJS module wrapper, so they don't exist in ES Modules. Use `import.meta.dirname` and `import.meta.filename` in recent Node versions.

**Q6. How do you write code that runs in both environments?**
Use standard APIs such as `fetch`, `URL` and Web Crypto, feature-detect with `globalThis`, and keep runtime-specific code behind a small adapter.

### Quick revision
Browser has DOM and `window`. Node has `process`, `fs` and `global`. `globalThis` works in both. `__dirname` isn't truly global.

---

## 3. ⭐ Module Systems

### Why it matters
Modules decide how code loads, how state is shared and why some packages fail to import. Questions on CommonJS vs ESM are very common.

### Explanation

#### CommonJS (CJS)
Node's original module system. It uses `require` and `module.exports`, loads **synchronously**, and resolves at runtime, so `require` can sit inside an `if`.

```javascript
// math.js
function add(a, b) { return a + b; }
module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5

// Conditional loading is allowed
if (process.env.DEBUG) {
  const debugTools = require('./debug-tools');
  debugTools.enable();
}
```

#### ES Modules (ESM)
The official JavaScript standard. It uses `import` and `export`. Imports are parsed before the code runs, which enables tree-shaking and top-level `await`. Use it with a `.mjs` extension or `"type": "module"` in `package.json`.

```javascript
// math.mjs
export function add(a, b) { return a + b; }
export default function multiply(a, b) { return a * b; }

// app.mjs
import multiply, { add } from './math.mjs';

const config = await loadConfig();                // top-level await is allowed
const { heavy } = await import('./heavy.mjs');    // dynamic import for lazy loading
```

| Feature | CommonJS | ES Modules |
|---|---|---|
| Syntax | `require`, `module.exports` | `import`, `export` |
| Loading | Synchronous | Asynchronous, in phases |
| Analysis | Runtime | Static, before execution |
| Top-level `await` | No | Yes |
| Exports | Copy of the value at export time | Live bindings |
| `__dirname` | Yes | `import.meta.dirname` |
| `this` at top level | `module.exports` | `undefined` |

**Live bindings vs copies:**

```javascript
// counter.cjs: the exported number is copied
let count = 0;
module.exports = { count, inc() { count++; } };
// const c = require('./counter.cjs'); c.inc(); c.count is still 0

// counter.mjs: the import is a live view of the variable
export let count = 0;
export function inc() { count++; }
// import { count, inc } from './counter.mjs'; inc(); count is now 1
```

#### Interop
- ESM can import CommonJS. `module.exports` becomes the default export.
- CommonJS can load ESM with dynamic `import()`.
- Recent Node versions also support `require()` of ESM, as long as the module graph has no top-level `await`.

```javascript
// ESM importing CommonJS
import express from 'express';

// CommonJS loading ESM on any modern version
async function main() {
  const { default: chalk } = await import('chalk');
  console.log(chalk.green('loaded'));
}

// Support both from one package, using the exports field
// package.json
// {
//   "exports": {
//     "import": "./dist/index.mjs",
//     "require": "./dist/index.cjs"
//   }
// }
```

#### Module wrapper
Before running a CommonJS file, Node wraps it in a function. This is why each file has private scope and why `require` and `__dirname` are available.

```javascript
// Node effectively runs your file like this:
(function (exports, require, module, __filename, __dirname) {
  // your file's code
});

// So this common bug makes sense:
exports = { add };             // Wrong: reassigns the local variable only
exports.add = add;             // Right: adds to the shared object
module.exports = { add };      // Right: replaces what require() returns
```

#### Resolution
When you call `require('x')`, Node checks in this order:

1. Core modules such as `fs`. The `node:` prefix makes this explicit.
2. Relative or absolute paths, trying `.js`, `.json`, `.node`, then a folder's `package.json` or `index.js`.
3. Bare names, by walking up through `node_modules` folders.

ESM is stricter: relative imports need the full file extension.

```javascript
require('node:fs');            // always the core module
require('./utils');            // ./utils.js, then ./utils/index.js
require('lodash');             // ./node_modules, then ../node_modules, and so on

console.log(require.resolve('./utils')); // shows the path Node picked
console.log(module.paths);               // shows the node_modules search list

// import { x } from './utils';    // ESM: fails, no extension
// import { x } from './utils.js'; // ESM: works
```

#### Caching
A module runs **once**. Later `require` calls return the same cached `module.exports`, keyed by resolved file path. This gives you a free singleton, and also shared mutable state.

```javascript
// config.js
console.log('loaded once');
module.exports = { retries: 3 };

// app.js
const a = require('./config');   // prints 'loaded once'
const b = require('./config');   // prints nothing
console.log(a === b);            // true
a.retries = 10;
console.log(b.retries);          // 10, same object

// Clearing the cache, for tests or hot reload
delete require.cache[require.resolve('./config')];
```

Two copies of a package in different `node_modules` folders are **different** cache entries, so an `instanceof` check can fail across them.

#### Circular dependencies
If A requires B and B requires A, one of them receives the other's **partially filled** exports.

```javascript
// a.js
exports.name = 'A';
const b = require('./b');
console.log('in a, b.name =', b.name);   // 'B'
exports.done = true;

// b.js
exports.name = 'B';
const a = require('./a');                // a is only half loaded
console.log('in b, a.done =', a.done);   // undefined

// Fixes: move shared code to a third module, or require inside the function
function useA() {
  const a = require('./a');              // resolved when called, not at load
  return a.done;
}
```

### Real-time scenario
**Problem:** After upgrading a dependency, the service crashes with `ERR_REQUIRE_ESM`.
**Diagnosis:** The new package version is ESM-only and the app is CommonJS on an older Node version.
**Fix:** Use `await import()` at that call site, or upgrade Node so `require(esm)` works. Plan the app's move to ESM as a separate task.

### Common mistakes and trade-offs
- Writing `exports = {...}` instead of `module.exports = {...}`.
- Keeping per-request data in module-level variables. It's shared across all requests.
- Publishing separate CJS and ESM builds can cause the **dual package hazard**: two instances of the same module's state.
- **Trade-off:** ESM is the standard and enables better tooling. CommonJS has simpler sync loading and a long legacy.

### Top interview questions
**Q1. What are the main differences between CommonJS and ES Modules?**
CJS loads synchronously at runtime and exports value copies. ESM is statically analysed, loads in phases, has live bindings and supports top-level `await`.

**Q2. What is the module wrapper?**
The function Node wraps around each CommonJS file. It provides private scope and injects `exports`, `require`, `module`, `__filename` and `__dirname`.

**Q3. What is the difference between `exports` and `module.exports`?**
`exports` is a shortcut pointing at `module.exports`. `require` returns `module.exports`, so reassigning `exports` breaks the link and exports nothing.

**Q4. How does module caching work?**
The first `require` runs the file and stores it in `require.cache` by resolved path. Later calls return the same object without running the file again.

**Q5. What happens with circular dependencies?**
Node returns the partially completed exports of the module still loading, so values may be `undefined`. Fix it by extracting shared code or moving the `require` into a function.

**Q6. Why is tree-shaking possible with ESM but hard with CommonJS?**
ESM imports are static, so tools can see unused exports at build time. `require` can be dynamic, so tools can't safely remove code.

**Q7. How do you use an ESM-only package from CommonJS?**
Use dynamic `import()`, which returns a promise. Recent Node versions can also `require()` ESM that has no top-level `await`.

### Quick revision
CJS is sync, dynamic and copies values. ESM is static, has live bindings and top-level `await`. Modules run once and are cached. Circular dependencies give partial exports.

---

## 4. npm and Package Management

### Why it matters
Most production Node code is dependencies. Senior engineers are expected to understand reproducible builds, version ranges and package manager trade-offs.

### Explanation

#### `package.json`
The project's manifest. It holds metadata, scripts, dependencies and entry points.

```json
{
  "name": "orders-api",
  "version": "1.4.2",
  "type": "module",
  "main": "./dist/index.js",
  "engines": { "node": ">=22" },
  "scripts": {
    "dev": "node --watch src/index.js",
    "test": "node --test",
    "start": "node dist/index.js"
  },
  "dependencies": { "express": "^5.0.0" },
  "devDependencies": { "eslint": "^9.0.0" }
}
```

| Type | Purpose | Example |
|---|---|---|
| `dependencies` | Needed at runtime | `express` |
| `devDependencies` | Needed only for development | `eslint` |
| `peerDependencies` | The host project must provide it | A React plugin needs `react` |
| `optionalDependencies` | Install failure is allowed | Native speed-ups |

#### Semver
Versions are `MAJOR.MINOR.PATCH`: breaking change, new feature, bug fix.

```text
^1.4.2   allows >=1.4.2 and <2.0.0    minor and patch updates
~1.4.2   allows >=1.4.2 and <1.5.0    patch updates only
1.4.2    exactly this version
*        any version: avoid this

Special case for 0.x: ^0.4.2 allows only >=0.4.2 and <0.5.0
```

#### Lockfiles
`package.json` holds ranges. `package-lock.json` records the **exact** version, source and integrity hash of every package in the tree, including transitive ones.

```text
package.json:       "express": "^5.0.0"       a range
package-lock.json:  "express": "5.1.0"        the exact version, plus integrity hash
```

Always commit the lockfile for applications. Without it, two builds on different days can install different code.

#### `npm ci`
`npm install` may update the lockfile. `npm ci` installs exactly what the lockfile says, deletes `node_modules` first, and fails if the lockfile and `package.json` disagree.

```bash
npm install express        # development: add a package
npm ci                     # CI and Docker: exact, reproducible install
npm ci --omit=dev          # production image: skip devDependencies
npm outdated               # see available updates
npm ls lodash              # see why a package is installed
```

#### npm vs pnpm vs yarn

| Tool | How it stores packages | Strength | Weakness |
|---|---|---|---|
| npm | Flat `node_modules` | Ships with Node | Slower, more disk use |
| pnpm | Global store plus links | Fast, saves disk, strict | Some older packages need config |
| yarn | Flat, or Plug'n'Play | Mature workspaces | PnP needs tool support |

pnpm's strictness prevents **phantom dependencies**:

```javascript
// You never listed 'debug' in package.json, but express depends on it.
// With npm's flat node_modules this works by accident:
const debug = require('debug');
// With pnpm it fails, which correctly exposes the missing dependency.
```

#### Monorepos
One repository holds many packages. Workspaces link them locally, and tools like Turborepo or Nx add task caching and affected-only builds.

```json
{
  "name": "platform",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

```bash
npm install                                   # links all workspaces
npm run test --workspace=apps/orders-api      # run in one package
npm run build --workspaces                    # run in all packages
```

```javascript
// apps/orders-api can import a sibling package directly
import { logger } from '@platform/logger';
```

### Real-time scenario
**Problem:** The build passes on Monday and fails on Tuesday with no code change.
**Diagnosis:** CI ran `npm install` and the lockfile wasn't committed, so a transitive dependency moved to a broken patch version.
**Fix:** Commit the lockfile, use `npm ci` in CI, and manage upgrades through a tool like Renovate.

### Common mistakes and trade-offs
- Putting build tools in `dependencies`, which bloats production images.
- Using `npm install` in CI or Dockerfiles.
- Ignoring `engines`, so the app runs on the wrong Node version.
- **Trade-off:** caret ranges pick up fixes automatically but need a lockfile for safety. Exact pins are predictable but need regular upgrade work.

### Top interview questions
**Q1. What is the difference between `^` and `~`?**
`^1.4.2` allows minor and patch updates below 2.0.0. `~1.4.2` allows only patch updates below 1.5.0.

**Q2. Why commit `package-lock.json`?**
It pins the full dependency tree with integrity hashes, so every environment installs identical code, and tampered packages are detected.

**Q3. What is the difference between `npm install` and `npm ci`?**
`npm install` can change the lockfile. `npm ci` follows it exactly, removes `node_modules`, and fails on any mismatch, so it suits CI.

**Q4. What are peer dependencies?**
Packages a library needs but expects the host project to supply. This prevents duplicates, such as two copies of React.

**Q5. Why might you choose pnpm?**
It's faster, saves disk through a shared store, and blocks phantom dependencies with its strict layout. It works especially well in monorepos.

**Q6. What is a phantom dependency?**
A package your code imports without declaring it, which works only because another dependency pulled it in. It can break when that dependency changes.

**Q7. How do you make Docker builds faster with npm?**
Copy `package*.json` first, run `npm ci`, then copy the source. Dependency layers stay cached until the lockfile changes.

### Quick revision
`^` is minor, `~` is patch. Commit the lockfile. Use `npm ci` in CI. pnpm is strict and fast. Workspaces power monorepos.

---

# Part 2: Runtime Internals

## 5. ⭐ Node.js Architecture

### Why it matters
"What happens when you call `fs.readFile`?" tests whether you can trace a call through every layer. It's the base for event loop, performance and debugging questions.

### Explanation

```mermaid
flowchart TD
    A["Your JavaScript"] --> B["Node standard library: fs, http, crypto"]
    B --> C["C++ bindings"]
    C --> D["V8: runs JavaScript"]
    C --> E["libuv: event loop and async I/O"]
    E --> F["OS async APIs: epoll, kqueue, IOCP<br/>(network sockets)"]
    E --> G["Thread pool, 4 threads by default<br/>(file I/O, dns.lookup, some crypto, zlib)"]
```

#### V8
V8 parses and compiles JavaScript, manages the heap and runs garbage collection. It knows nothing about files or sockets. Node exposes some of its data directly:

```javascript
const v8 = require('node:v8');

console.log(process.versions.v8);                  // V8 version in this Node build
console.log(v8.getHeapStatistics().heap_size_limit); // max heap in bytes
```

#### libuv
libuv runs the event loop and gives Node one async API across operating systems. Network I/O uses the OS's non-blocking mechanisms, so it needs no pool thread.

```javascript
const net = require('node:net');

// One thread can watch thousands of sockets.
// libuv asks the OS to report which sockets are ready.
const server = net.createServer((socket) => {
  socket.on('data', (chunk) => socket.write(chunk)); // echo
});
server.listen(4000);
console.log(process.versions.uv); // libuv version
```

#### C++ bindings
Bindings connect JavaScript to native code. When you call `fs.readFile`, the JS function calls a C++ function, which calls into libuv.

```javascript
// Simplified journey of one call
// 1. JS       fs.readFile('a.txt', callback)
// 2. C++      binding creates a request object and passes it to libuv
// 3. libuv    runs the read on a pool thread
// 4. libuv    queues completion for the event loop
// 5. C++      wraps the result into JS values
// 6. V8       runs your callback on the main thread
```

#### Call stack
The call stack tracks which function is running. A callback can run only when the stack is empty, because the main thread does one thing at a time.

```javascript
function third() { console.log('third'); }
function second() { third(); }
function first() { second(); }

setTimeout(() => console.log('timer'), 0);

first();
// Stack: first → second → third, then it unwinds and empties.
// Only then can 'timer' run, even with a 0 ms delay.

function recurse() { recurse(); }
// recurse(); // RangeError: Maximum call stack size exceeded
```

#### Thread pool
libuv's pool handles work without a usable async OS API. The default size is 4, set with `UV_THREADPOOL_SIZE`.

| Uses the pool | Doesn't use the pool |
|---|---|
| `fs` operations | TCP and UDP sockets |
| `dns.lookup` | `dns.resolve` (uses c-ares) |
| `crypto.pbkdf2`, `scrypt`, `randomBytes` (async) | HTTP requests |
| `zlib` (async) | Timers |

```javascript
const crypto = require('node:crypto');
const start = Date.now();

// With 4 pool threads, the first 4 finish together and the 5th runs later.
for (let i = 1; i <= 5; i++) {
  crypto.pbkdf2('secret', 'salt', 200_000, 64, 'sha512', () => {
    console.log(`task ${i}: ${Date.now() - start} ms`);
  });
}
// UV_THREADPOOL_SIZE=8 node pool.js   // all 5 finish together
// Set it before the pool first starts. The environment variable is safest.
```

### Real-time scenario
**Problem:** Login latency jumps from 80 ms to 2 s at peak, while CPU and event loop lag look fine at first.
**Diagnosis:** Async `bcrypt` and heavy file reads shared the 4-thread pool, so password hashes queued behind each other.
**Fix:** Raise `UV_THREADPOOL_SIZE` to fit the available cores, reduce file work on that service, and tune hash cost.

### Common mistakes and trade-offs
- Believing all async work uses the thread pool. Network I/O doesn't.
- Setting a very large pool. Beyond the core count, CPU work just competes for cores.
- **Trade-off:** a shared pool is simple, but slow file or crypto work can delay `dns.lookup`, which delays outgoing HTTP connections.

### Top interview questions
**Q1. Describe Node's architecture.**
JS code calls the standard library, which goes through C++ bindings to V8 and libuv. V8 runs JS. libuv handles the event loop, OS async I/O and the thread pool.

**Q2. What happens when you call `fs.readFile`?**
A request passes through the binding to libuv, and a pool thread does the read. On completion, the event loop picks up the result and V8 runs your callback on the main thread.

**Q3. Which operations use the thread pool?**
File system calls, `dns.lookup`, some async crypto and zlib. Network sockets use OS mechanisms instead.

**Q4. Does every HTTP request use a pool thread?**
No. Socket I/O uses epoll, kqueue or IOCP. However, `dns.lookup` during connection setup does use the pool.

**Q5. How do you size `UV_THREADPOOL_SIZE`?**
Start near the core count for CPU-heavy pool work, and go higher for slow file or DNS operations. Set it at process start, then load test.

**Q6. What is the difference between `dns.lookup` and `dns.resolve`?**
`lookup` uses the OS resolver through `getaddrinfo` on the pool, and honours `/etc/hosts`. `resolve` uses c-ares and DNS queries directly, without the pool.

### Quick revision
JS → bindings → V8 + libuv. Network uses OS async APIs. Files, `dns.lookup`, some crypto and zlib use the 4-thread pool.

---

## 6. ⭐ Event Loop

### Why it matters
This is the most asked Node topic. Interviewers often ask you to predict output, so you need the order of phases and microtasks from memory.

### Explanation

#### The six phases

```mermaid
flowchart TD
    T["1. Timers<br/>setTimeout, setInterval"] --> P["2. Pending callbacks<br/>deferred system callbacks"]
    P --> I["3. Idle, prepare<br/>internal"]
    I --> O["4. Poll<br/>I/O callbacks, waits here if idle"]
    O --> C["5. Check<br/>setImmediate"]
    C --> X["6. Close callbacks<br/>socket.on('close')"]
    X --> T
```

| Phase | Runs | Example |
|---|---|---|
| Timers | Expired timers | `setTimeout(fn, 100)` |
| Pending | Some deferred system errors | TCP `ECONNREFUSED` report |
| Idle, prepare | Internal use | None for your code |
| Poll | I/O callbacks | `fs.readFile` callback |
| Check | Immediates | `setImmediate(fn)` |
| Close | Close events | `socket.on('close', fn)` |

```javascript
const fs = require('node:fs');

fs.readFile(__filename, () => {            // poll phase
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// Always prints: immediate, timeout
// After poll comes check. Timers come in the next loop iteration.
```

The loop exits when nothing keeps it alive. `timer.unref()` tells it not to wait for that timer.

#### Microtasks
Two queues are drained **after each callback**, since Node 11:

1. The `process.nextTick` queue
2. The Promise microtask queue (`.then`, code after `await`, `queueMicrotask`)

```javascript
console.log('1 sync');

setTimeout(() => console.log('6 timeout'), 0);
setImmediate(() => console.log('7 immediate'));

Promise.resolve().then(() => console.log('4 promise'));
queueMicrotask(() => console.log('5 microtask'));
process.nextTick(() => console.log('3 nextTick'));

console.log('2 sync');
// In a CommonJS file: 1, 2, 3, 4, 5, then 6 and 7.
// Timeout vs immediate can swap here, because this code isn't in an I/O callback.
```

Important detail: in an **ES Module**, the file body already runs inside a promise job, so `4` and `5` print **before** `3`.

```javascript
// Microtasks run between callbacks, not only between phases
setTimeout(() => {
  console.log('timer 1');
  Promise.resolve().then(() => console.log('promise in timer 1'));
}, 0);
setTimeout(() => console.log('timer 2'), 0);
// Node 11+: timer 1, promise in timer 1, timer 2
```

#### `nextTick` vs `setImmediate`

| | `process.nextTick` | `setImmediate` |
|---|---|---|
| Runs | Right after the current operation | In the check phase |
| Priority | Before Promises and I/O | After poll |
| Risk | Can starve I/O | Yields to I/O |

The names are misleading: `nextTick` fires sooner than `setImmediate`.

```javascript
const { EventEmitter } = require('node:events');

// nextTick's main use: let callers attach listeners before an event fires.
class Job extends EventEmitter {
  constructor() {
    super();
    process.nextTick(() => this.emit('ready'));
  }
}
const job = new Job();
job.on('ready', () => console.log('ready')); // works: emit happens later

// setImmediate's main use: split long work and let I/O run between chunks.
function processBatch(items, i = 0) {
  const end = Math.min(i + 1000, items.length);
  for (; i < end; i++) handle(items[i]);
  if (i < items.length) setImmediate(() => processBatch(items, i));
}
```

#### Starvation
Microtask queues must be empty before the loop moves on. If callbacks keep adding to them, the loop never reaches I/O or timers.

```javascript
// Starves everything: this timer never fires
setTimeout(() => console.log('never'), 0);

function loop() { process.nextTick(loop); }
loop();

// Promises starve in the same way
// function loop2() { Promise.resolve().then(loop2); }

// setImmediate doesn't starve I/O, because the loop moves on each time
function safe() { setImmediate(safe); }
```

Blocking also happens through ordinary sync work: a large `JSON.parse`, `fs.readFileSync` in a handler, or a slow regex.

#### Event loop lag
Lag is how late the loop runs compared to when it should. It's the most direct health metric for a Node service.

```javascript
const { monitorEventLoopDelay, performance } = require('node:perf_hooks');

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

setInterval(() => {
  console.log({
    p50: histogram.percentile(50) / 1e6,   // nanoseconds to ms
    p99: histogram.percentile(99) / 1e6,
    utilization: performance.eventLoopUtilization().utilization,
  });
  histogram.reset();
}, 10_000).unref();
```

### Real-time scenario
**Problem:** p99 latency jumps to 3 s every few minutes. CPU averages 30%, and the database is healthy.
**Diagnosis:** `monitorEventLoopDelay` shows 2.8 s spikes. A CPU profile points to `JSON.parse` on a 90 MB payload from an admin export endpoint.
**Fix:** Stream the export with a streaming JSON parser, cap body size, and alert on loop delay.

### Common mistakes and trade-offs
- Expecting `setTimeout(fn, 0)` to run immediately. Node treats it as at least 1 ms, after the stack and microtasks.
- Assuming timeout-vs-immediate order is fixed in the main module. It isn't.
- Wrapping CPU work in a Promise. Promises don't move work off the thread.
- **Trade-off:** `nextTick` gives fast, predictable ordering, while `setImmediate` is kinder to I/O.

### Top interview questions
**Q1. What is the event loop?**
libuv's loop that lets one thread handle async work. It cycles through phases, running queued callbacks, and drains microtasks after each callback.

**Q2. Name the phases in order.**
Timers, pending callbacks, idle and prepare, poll, check, close callbacks.

**Q3. What is the difference between `process.nextTick` and `setImmediate`?**
`nextTick` runs right after the current operation, before Promises and the next phase. `setImmediate` runs in the check phase, after poll. Prefer `setImmediate` for yielding.

**Q4. Which runs first: `setTimeout(0)` or `setImmediate`?**
In the main module it's non-deterministic and depends on process timing. Inside an I/O callback, `setImmediate` always runs first.

**Q5. How do Promises fit into the event loop?**
They're microtasks and aren't part of a phase. They drain after each callback, right after the `nextTick` queue.

**Q6. What blocks the event loop, and how do you find it?**
Sync CPU work: big JSON, sync file calls, slow regex, large loops. Find it with `monitorEventLoopDelay`, `eventLoopUtilization` and CPU profiles.

**Q7. What happens in the poll phase when nothing is pending?**
If immediates are queued, it moves to check. If not, it waits for I/O, up to the next timer's expiry.

### Quick revision
Timers → pending → idle → poll → check → close. After each callback: `nextTick`, then Promises. Inside I/O, immediate beats timeout. Recursive microtasks starve the loop. Track lag.

---

## 7. V8 Engine

### Why it matters
For most services, you don't tune V8 directly. But V8 knowledge explains why some code is much slower than similar code, and it's a strong signal in senior interviews.

### Explanation

#### JIT compilation
V8 starts by interpreting code, then compiles **hot** functions to faster machine code, based on observed types.

```text
Source → Parser → AST → Ignition (bytecode interpreter)
                          ↓ warm code
                        Sparkplug (fast baseline compiler)
                          ↓ hotter code
                        Maglev (mid-tier optimizer)
                          ↓ hottest code
                        TurboFan (top optimizer)
```

```javascript
function add(a, b) { return a + b; }

// After many calls with numbers, V8 compiles a fast number-only version.
for (let i = 0; i < 1_000_000; i++) add(i, 1);

// This is why benchmarks need a warm-up: early runs measure slower code.
```

#### Hidden classes
Objects have no fixed layout in JS, so V8 assigns internal **hidden classes** (also called shapes or maps). Objects built with the same properties in the same order share a shape.

```javascript
// Same shape: fast
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const a = new Point(1, 2);
const b = new Point(3, 4);

// Different shapes: property order differs
const c = { x: 1, y: 2 };
const d = { y: 2, x: 1 };

// Shape changes after creation: avoid in hot code
const e = new Point(1, 2);
e.z = 3;        // new shape
delete e.x;     // can move the object to slow dictionary mode
```

#### Inline caching
At each property access, V8 remembers the shape it saw and where the value sits. Next time, it skips the lookup.

```javascript
function getX(p) { return p.x; }

// Monomorphic: one shape, the fastest
getX({ x: 1 });
getX({ x: 2 });

// Polymorphic: a few shapes, slower
getX({ x: 1, y: 2 });
getX({ y: 2, x: 1 });

// Megamorphic: many shapes, V8 stops caching at this site
getX({ a: 1, x: 1 });
getX({ b: 1, x: 1 });
getX({ c: 1, x: 1 });
getX({ d: 1, x: 1 });
getX({ e: 1, x: 1 });
```

#### Deoptimization
Optimized code relies on assumptions. If one breaks, V8 throws the code away and goes back to a slower tier.

```javascript
function total(a, b) { return a + b; }

for (let i = 0; i < 1_000_000; i++) total(i, i); // optimized for integers

total('a', 'b'); // assumption fails, so the function deoptimizes

// See it happen:
// node --trace-opt --trace-deopt app.js
```

Keep types stable in hot paths. Don't mix numbers and strings in a parameter, and initialize all fields in the constructor.

### Real-time scenario
**Problem:** A pricing function is 5× slower in production than in a benchmark.
**Diagnosis:** The benchmark used one object type. Production objects came from three sources with different property orders and optional fields, so the call site became megamorphic.
**Fix:** Normalize objects into one shape at the boundary, with all fields always set, even to `null`.

### Common mistakes and trade-offs
- Using `delete` on hot objects. Set the value to `undefined` or use a `Map`.
- Micro-optimizing code that isn't hot. Profile first.
- Benchmarking without warm-up, or with results V8 can optimize away.
- **Trade-off:** JIT gives high peak speed, but costs warm-up time and memory. That matters for short-lived serverless functions.

### Top interview questions
**Q1. How does V8 execute JavaScript?**
It parses source to an AST, Ignition runs bytecode, and hot functions move through compilers up to TurboFan, using runtime type feedback.

**Q2. What are hidden classes?**
Internal object layouts that give dynamic objects fixed offsets. Objects created with the same properties in the same order share one, so access is fast.

**Q3. What is inline caching?**
V8 stores the shape and offset seen at a property access. Monomorphic sites are the fastest, and megamorphic sites lose the benefit.

**Q4. What causes deoptimization?**
A broken assumption, such as a new type at a hot call site or a changed object shape. V8 discards the optimized code and falls back.

**Q5. Why does `delete` hurt performance?**
It can change an object's shape or move it to slower dictionary mode. Set the field to `undefined`, or use `Map` for dynamic keys.

**Q6. Why do Node apps run slower just after start?**
Code begins in the interpreter and caches are cold. Performance improves once hot code is compiled, so canary traffic should ramp up gradually.

### Quick revision
Ignition → Sparkplug → Maglev → TurboFan. Keep object shapes and types stable. Avoid `delete` in hot code. Warm up before you benchmark.

---

## 8. ⭐ Memory and Garbage Collection

### Why it matters
Memory leaks are common production incidents. Senior candidates are expected to explain how V8 manages memory, why processes get OOM-killed and what leaks look like. Topic 53 covers the debugging steps.

### Explanation

#### Heap vs stack
- **Stack:** function frames and primitive locals. Fast, small, automatically released.
- **Heap:** objects, arrays and closures. Managed by the garbage collector.

```javascript
function createUser() {
  const age = 30;                  // lives in the stack frame (conceptually)
  const user = { name: 'Asha' };   // object on the heap, reference on the stack
  return user;                     // frame is gone, heap object survives
}

const mem = process.memoryUsage();
console.log({
  rss: mem.rss,             // total process memory held in RAM
  heapTotal: mem.heapTotal, // heap V8 has reserved
  heapUsed: mem.heapUsed,   // heap in use by objects
  external: mem.external,   // C++ objects tied to JS
  arrayBuffers: mem.arrayBuffers, // includes Buffers
});
```

Buffers live **outside** the V8 heap, so `rss` can grow while `heapUsed` stays flat.

#### Generational GC
Most objects die young, so V8 splits the heap:

| Space | Holds | Algorithm | Cost |
|---|---|---|---|
| New space | Fresh objects | **Scavenge**: copies live objects between semi-spaces | Frequent, fast |
| Old space | Survivors | **Mark-Sweep-Compact**: marks reachable objects, frees the rest, compacts | Rarer, slower |

An object that survives two scavenges is promoted to old space. Marking mostly runs incrementally and concurrently, to shorten pauses.

GC uses **reachability**. Anything reachable from a root (globals, the stack, active closures) stays alive.

```javascript
let user = { name: 'Asha' };   // reachable
user = null;                   // no longer reachable, so GC can free it

// Circular references don't leak: neither object is reachable from a root
function pair() {
  const a = {};
  const b = { a };
  a.b = b;
}
pair();

// See GC activity
// node --trace-gc app.js

// WeakMap entries don't keep their keys alive
const metadata = new WeakMap();
let req = {};
metadata.set(req, { startedAt: Date.now() });
req = null; // key and metadata can both be collected
```

#### Heap limits
V8 has a maximum old-space size, based on available memory. Reaching it crashes with `FATAL ERROR: JavaScript heap out of memory`.

```bash
# Raise the old-space limit (value in MB)
node --max-old-space-size=1536 app.js

# Larger young generation: fewer scavenges for allocation-heavy apps
node --max-semi-space-size=64 app.js
```

```javascript
const v8 = require('node:v8');
console.log(v8.getHeapStatistics().heap_size_limit / 1024 / 1024, 'MB');
```

In a container with a 2 GB limit, leave room for Buffers, stacks and native memory. A limit near 75% of the container is a common starting point.

#### Memory leaks
A leak is memory that's still reachable but will never be used again.

```javascript
// 1. Unbounded cache
const cache = new Map();
app.get('/user/:id', async (req, res) => {
  cache.set(req.params.id, await load(req.params.id));  // grows forever
  res.json(cache.get(req.params.id));
});
// Fix: use an LRU cache with a max size and TTL (Topic 72).

// 2. Listener added per request
app.get('/stream', (req, res) => {
  const onTick = () => res.write('tick\n');
  bus.on('tick', onTick);
  req.on('close', () => bus.off('tick', onTick));  // the fix: remove on close
});

// 3. Forgotten timer: its closure keeps bigData alive
function start() {
  const bigData = Buffer.alloc(50 * 1024 * 1024);
  const timer = setInterval(() => console.log(bigData.length), 1000);
  return () => clearInterval(timer);  // the fix: return a way to stop it
}

// 4. Closure holding more than it needs
function makeHandler(hugeReport) {
  const { id } = hugeReport;     // the fix: keep only what you need
  return () => id;
}
```

### Real-time scenario
**Problem:** Pods restart every 6 hours with `OOMKilled`, and memory climbs in a sawtooth with a rising floor.
**Diagnosis:** Comparing heap snapshots shows thousands of retained request objects, held by a module-level `Map` used for request tracing.
**Fix:** Delete entries when responses finish, or use `AsyncLocalStorage` (Topic 14) and remove the `Map` altogether.

### Common mistakes and trade-offs
- Raising the heap limit to "fix" a leak. That only delays the crash.
- Calling `global.gc()` in production. It adds pauses and doesn't fix anything.
- Setting the heap limit equal to the container limit. Native memory then triggers an OOM kill.
- **Trade-off:** a larger heap means fewer collections, but longer pauses when they run.

### Top interview questions
**Q1. How does garbage collection work in V8?**
It's generational. New space uses fast copying scavenges, and old space uses mark-sweep-compact. Reachability from roots decides what stays.

**Q2. What is the difference between `rss`, `heapTotal` and `heapUsed`?**
`rss` is the whole process in RAM. `heapTotal` is reserved V8 heap. `heapUsed` is heap used by live objects and uncollected garbage.

**Q3. Can closures cause memory leaks?**
Yes, when a long-lived closure references large data. The data stays reachable for as long as the closure does.

**Q4. Are circular references a problem?**
No. V8 uses reachability, not reference counting, so unreachable cycles are collected.

**Q5. Why might RSS grow while `heapUsed` stays flat?**
The growth is outside the JS heap: Buffers, native addons, or allocator fragmentation.

**Q6. What is the difference between `Map` and `WeakMap` for caching?**
`Map` holds keys strongly, so entries stay forever. `WeakMap` lets keys be collected, so it suits metadata tied to an object's lifetime.

**Q7. What are the common causes of leaks in Node?**
Unbounded caches, unremoved listeners, uncleared timers, global arrays and closures holding large objects.

### Quick revision
Young objects are scavenged, survivors go to old space and mark-sweep-compact. GC is based on reachability. Buffers live outside the heap. Leaks are reachable but unused objects.

---

# Part 3: Async Programming

## 9. Callbacks

### Why it matters
Callbacks are the base of Node's async model, and many APIs and old codebases still use them. You need to convert and handle them safely.

### Explanation

#### Error-first pattern
Node's convention puts the error first. Check it before using the result.

```javascript
const fs = require('node:fs');

fs.readFile('config.json', 'utf8', (err, data) => {
  if (err) {
    console.error('read failed:', err.code);
    return;                    // this return is essential
  }
  console.log(JSON.parse(data));
});

// Writing your own
function divide(a, b, callback) {
  if (b === 0) {
    return process.nextTick(callback, new Error('division by zero'));
  }
  process.nextTick(callback, null, a / b);
}
```

Never mix sync and async callbacks in one function. It's called **releasing Zalgo**, because callers can't predict the order. `process.nextTick` above keeps both paths async.

`try/catch` also can't catch an error thrown later in a callback:

```javascript
try {
  setTimeout(() => { throw new Error('boom'); }, 0);
} catch (err) {
  console.log('never reached');   // the stack has already moved on
}
```

#### Callback hell
Nesting grows with each dependent step, and error handling is repeated everywhere.

```javascript
// Hard to read and maintain
getUser(id, (err, user) => {
  if (err) return done(err);
  getOrders(user.id, (err, orders) => {
    if (err) return done(err);
    getInvoice(orders[0].id, (err, invoice) => {
      if (err) return done(err);
      done(null, invoice);
    });
  });
});

// The same logic with async/await
async function loadInvoice(id) {
  const user = await getUser(id);
  const orders = await getOrders(user.id);
  return getInvoice(orders[0].id);
}
```

#### `util.promisify`
It converts an error-first callback function into one that returns a Promise.

```javascript
const util = require('node:util');
const fs = require('node:fs');

const readFile = util.promisify(fs.readFile);
const text = await readFile('a.txt', 'utf8');

// Keep `this` when promisifying a method
const get = util.promisify(redisClient.get).bind(redisClient);

// What it does internally
function promisify(fn) {
  return (...args) => new Promise((resolve, reject) => {
    fn(...args, (err, result) => (err ? reject(err) : resolve(result)));
  });
}

// Going the other way
const callbackStyle = util.callbackify(async () => 'done');

// Many core modules already have promise versions
const { readFile: read } = require('node:fs/promises');
```

### Real-time scenario
**Problem:** A legacy payment function sometimes calls its callback twice, and customers see duplicate confirmation emails.
**Diagnosis:** An `if (err) callback(err)` branch had no `return`, so the success path also ran.
**Fix:** Add `return`, then wrap the function with `util.promisify`. A Promise settles once, so double calls can't escape.

### Common mistakes and trade-offs
- Forgetting `return` after handling an error.
- Throwing from a callback, which becomes an uncaught exception.
- Promisifying a method and losing `this`.
- **Trade-off:** callbacks have very low overhead, but composition and error handling are much harder than with Promises.

### Top interview questions
**Q1. What is an error-first callback?**
A callback with signature `(err, result)`. `err` is `null` on success, so every caller handles failure in the same way.

**Q2. What is callback hell, and how do you avoid it?**
Deep nesting from dependent async steps. Avoid it with Promises, `async/await`, and small named functions.

**Q3. What does `util.promisify` do?**
It returns a Promise-based version of an error-first callback function.

**Q4. Why can't `try/catch` catch an error in an async callback?**
The callback runs later on a new stack, after the `try` block has finished.

**Q5. What is "releasing Zalgo"?**
A function that sometimes calls back synchronously and sometimes asynchronously, which gives unpredictable ordering. Always be consistently async.

### Quick revision
`(err, data)` with early `return`. `try/catch` doesn't cross async boundaries. Use `promisify` or `fs/promises`.

---

## 10. ⭐ Promises

### Why it matters
Promises are the core of modern async JS. Interviews test states, chaining, combinators and rejection handling, and they often ask you to implement `Promise.all`.

### Explanation

#### States and creation
A Promise is `pending`, then settles once as `fulfilled` or `rejected`.

```javascript
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const p = new Promise((resolve, reject) => {
  // The executor runs synchronously
  resolve('first');
  resolve('ignored');       // settled once, so this has no effect
});
```

#### Chaining
`.then` returns a new Promise. Returning a value passes it on, and returning a Promise waits for it. Throwing jumps to the next `.catch`.

```javascript
fetchUser(1)
  .then((user) => fetchOrders(user.id))      // waits for this Promise
  .then((orders) => orders.length)           // passes a value on
  .then((count) => {
    if (count === 0) throw new Error('no orders');
    return count;
  })
  .catch((err) => {                          // catches any error above
    console.error(err.message);
    return 0;                                // recovers the chain
  })
  .finally(() => console.log('cleanup'));    // always runs
```

#### `Promise.all`
Runs together and resolves with all results in input order. It rejects on the **first** rejection. Other operations aren't cancelled.

```javascript
const [user, orders] = await Promise.all([getUser(id), getOrders(id)]);

// Interview classic: implement it
function promiseAll(items) {
  return new Promise((resolve, reject) => {
    const results = new Array(items.length);
    let remaining = items.length;
    if (remaining === 0) return resolve(results);

    items.forEach((item, i) => {
      Promise.resolve(item).then((value) => {
        results[i] = value;                  // keep the input order
        if (--remaining === 0) resolve(results);
      }, reject);
    });
  });
}
```

#### `Promise.allSettled`
Waits for everything and never rejects. Use it when partial success is fine.

```javascript
const results = await Promise.allSettled([sendEmail(), sendSms(), sendPush()]);

results.forEach((r) => {
  if (r.status === 'fulfilled') console.log('sent', r.value);
  else console.error('failed', r.reason);
});
```

#### `Promise.race`
Settles with the first Promise to settle, success or failure. It's a classic way to add a timeout.

```javascript
function withTimeout(promise, ms) {
  let timer;
  const timeout = new Promise((_, reject) => {
    timer = setTimeout(() => reject(new Error('timeout')), ms);
  });
  return Promise.race([promise, timeout]).finally(() => clearTimeout(timer));
}
// Note: the losing operation still runs. Use AbortController to stop it (Topic 13).
```

#### `Promise.any`
Resolves with the first **success**. It rejects with `AggregateError` only if all fail.

```javascript
// Use the fastest healthy mirror
try {
  const data = await Promise.any([
    fetch('https://mirror-1.example.com/file'),
    fetch('https://mirror-2.example.com/file'),
  ]);
} catch (err) {
  console.error(err.errors);   // every failure
}
```

| Method | Resolves when | Rejects when |
|---|---|---|
| `all` | All succeed | Any fails |
| `allSettled` | All finish | Never |
| `race` | First settles with success | First settles with failure |
| `any` | First succeeds | All fail |

#### Unhandled rejections
Since Node 15, an unhandled rejection crashes the process by default.

```javascript
// No .catch, so this crashes the process
Promise.reject(new Error('boom'));

// Last-resort logging, then exit
process.on('unhandledRejection', (reason) => {
  logger.fatal({ reason }, 'unhandled rejection');
  process.exit(1);
});

// A common source: a floating promise
function handler(req, res) {
  saveAudit(req);   // missing await or .catch
  res.send('ok');
}
```

### Real-time scenario
**Problem:** A dashboard returns a 500 whenever the recommendations service is down, even though it's optional.
**Diagnosis:** All sections were loaded with `Promise.all`, so one rejection failed the whole page.
**Fix:** Use `Promise.allSettled`, return the required sections, and show a fallback for the optional one.

### Common mistakes and trade-offs
- Forgetting `return` inside `.then`, which breaks the chain.
- Wrapping a Promise in `new Promise` for no reason (the explicit construction anti-pattern).
- Assuming `Promise.all` cancels other work. It doesn't.
- **Trade-off:** `all` fails fast and is simple. `allSettled` tolerates partial failure but needs result handling.

### Top interview questions
**Q1. What are the states of a Promise?**
Pending, fulfilled and rejected. Once settled, it never changes.

**Q2. What is the difference between `Promise.all` and `allSettled`?**
`all` rejects on the first failure. `allSettled` waits for all and reports each outcome. Use `all` when everything is required.

**Q3. What is the difference between `race` and `any`?**
`race` follows the first to settle, including failures. `any` ignores failures until one succeeds.

**Q4. Does `Promise.all` run things in parallel?**
It doesn't start anything. The operations started when you called them, and `all` only waits. They're concurrent, not CPU-parallel.

**Q5. What happens on an unhandled rejection?**
Since Node 15, the default mode raises it as an uncaught exception and the process exits. Log it, then exit and let the supervisor restart.

**Q6. Is the Promise executor synchronous?**
Yes. Only `.then` callbacks run asynchronously, as microtasks.

**Q7. How do you implement `Promise.all`?**
Return a Promise, keep a results array by index and a remaining counter, resolve at zero, and reject on the first error. Handle empty input.

### Quick revision
`all`: all or fail fast. `allSettled`: everything reported. `race`: first settled. `any`: first success. Unhandled rejections crash the process.

---

## 11. ⭐ Async/Await

### Why it matters
This is how most modern Node code is written. Interviews focus on the mistakes that cost performance or lose errors.

### Explanation

#### How it works
`async` functions always return a Promise. `await` pauses that function only, not the thread. Code after it runs as a microtask.

```javascript
async function demo() {
  console.log('2 runs synchronously up to the first await');
  await null;
  console.log('4 resumes as a microtask');
}

console.log('1');
demo();
console.log('3 the thread was not blocked');
```

#### Error handling
A rejected `await` throws, so `try/catch` works. Remember to `await` inside the `try`.

```javascript
async function getProfile(id) {
  try {
    return await fetchProfile(id);   // `await` is needed here
  } catch (err) {
    logger.warn({ err, id }, 'profile fallback');
    return defaultProfile;
  }
}

// Bug: without await, the rejection escapes the try block
async function broken(id) {
  try {
    return fetchProfile(id);
  } catch (err) {
    return defaultProfile;   // never runs
  }
}

// Add context when rethrowing
async function chargeOrder(order) {
  try {
    return await gateway.charge(order.total);
  } catch (err) {
    throw new Error(`charge failed for ${order.id}`, { cause: err });
  }
}
```

#### Sequential vs parallel

```javascript
// Sequential: about 300 ms. Right only if b needs the result of a.
const a = await callA();   // 100 ms
const b = await callB();   // 100 ms
const c = await callC();   // 100 ms

// Parallel: about 100 ms
const [x, y, z] = await Promise.all([callA(), callB(), callC()]);

// A loop that's accidentally sequential
for (const id of ids) {
  await sendEmail(id);
}

// Parallel, but unbounded (see concurrency limits below)
await Promise.all(ids.map((id) => sendEmail(id)));
```

#### Common mistakes

```javascript
// 1. forEach doesn't wait
items.forEach(async (item) => { await save(item); });
console.log('runs before saves finish, and errors go unhandled');

// Fix
for (const item of items) await save(item);
// or
await Promise.all(items.map(save));

// 2. Floating promise
async function handler(req, res) {
  writeAudit(req);                 // a rejection here crashes the process
  res.send('ok');
}
// Fix: await it, or attach .catch if it's fire-and-forget
writeAudit(req).catch((err) => logger.error(err));

// 3. Forgotten await gives a Promise instead of a value
const user = getUser(id);
console.log(user.name);            // undefined

// 4. Promise started early, handled late: a rejection in the gap is unhandled
const p = failsQuickly();
await somethingSlow();
await p;

// 5. async in an EventEmitter listener: rejections aren't caught by emit
emitter.on('job', async () => { throw new Error('x'); });
```

#### Concurrency limits
`Promise.all` on 100,000 items starts them all at once, which can exhaust sockets, pools or remote rate limits. Use a bounded pool.

```javascript
async function mapLimit(items, limit, fn) {
  const results = new Array(items.length);
  let next = 0;

  async function worker() {
    while (next < items.length) {
      const i = next++;                    // safe: no await between read and increment
      results[i] = await fn(items[i], i);
    }
  }

  await Promise.all(Array.from({ length: Math.min(limit, items.length) }, worker));
  return results;
}

const users = await mapLimit(ids, 10, fetchUser);   // never more than 10 at once
```

Libraries such as `p-limit` do the same. Topic 72 has a full promise pool.

### Real-time scenario
**Problem:** A nightly job fans out 200,000 API calls with `Promise.all`. It crashes with `EMFILE` and triggers 429 responses.
**Diagnosis:** All requests started in one tick, which exhausted file descriptors and hit the partner's rate limit.
**Fix:** Process in a pool of 20 with retries and backoff. Runtime rises slightly, but the job completes reliably.

### Common mistakes and trade-offs
- `await` in a loop for independent work. It's needlessly sequential.
- `return await` is needed inside `try`, but elsewhere it adds little.
- Treating `async` as background work. CPU code in an `async` function still blocks.
- **Trade-off:** more concurrency means better throughput until downstream limits. Tune the limit to the slowest dependency.

### Top interview questions
**Q1. Does `await` block the thread?**
No. It suspends that async function and returns control to the event loop. It resumes as a microtask.

**Q2. What does an `async` function return?**
Always a Promise. Returned values become fulfilment, and thrown errors become rejection.

**Q3. How do you run async operations in parallel?**
Start them together and use `Promise.all`. Use `allSettled` for partial failures, and a pool if the count is large.

**Q4. Why doesn't `await` work inside `forEach`?**
`forEach` ignores returned Promises, so it doesn't wait. Use `for...of`, or `Promise.all` with `map`.

**Q5. When is `return await` needed?**
Inside `try/catch` or `finally`, when you want that block to handle the rejection.

**Q6. How do you limit concurrency?**
With a worker pool that pulls from a shared index, or `p-limit`. Set the limit based on downstream capacity.

**Q7. What is a floating promise?**
A Promise that's neither awaited nor caught. Its failure becomes an unhandled rejection. ESLint's `no-floating-promises` rule catches them.

### Quick revision
`await` suspends the function, not the thread. Independent work goes in `Promise.all`. `forEach` doesn't wait. Bound your concurrency.

---

## 12. ⭐ EventEmitter

### Why it matters
Streams, HTTP servers, sockets and `process` are all built on EventEmitter. "Implement your own EventEmitter" is also a popular coding question (see Topic 72).

### Explanation

#### Observer pattern
Emitters publish named events, and listeners subscribe. The emitter doesn't need to know who listens, which decouples components.

```javascript
const { EventEmitter } = require('node:events');

class OrderService extends EventEmitter {
  async place(order) {
    const saved = await db.orders.insert(order);
    this.emit('order:placed', saved);
    return saved;
  }
}

const orders = new OrderService();
orders.on('order:placed', (o) => emailService.sendReceipt(o));
orders.on('order:placed', (o) => analytics.track('order', o));
orders.once('order:placed', () => console.log('first order today'));
```

#### Synchronous emit
`emit` calls listeners **synchronously**, in registration order, before returning. Nothing here is async by default.

```javascript
const bus = new EventEmitter();

bus.on('ping', () => console.log('2 listener'));

console.log('1 before');
bus.emit('ping');
console.log('3 after');   // listener already finished

// A throwing listener stops later listeners and reaches the emit() caller
bus.on('task', () => { throw new Error('bad'); });
bus.on('task', () => console.log('skipped'));
// bus.emit('task'); // throws here

// A slow listener blocks the event loop like any other sync code.
// Defer heavy work:
bus.on('order', (o) => setImmediate(() => buildReport(o)));
```

#### Listener leaks
Listeners are strong references. Adding one per request without removal leaks memory. Node warns after 10 listeners on one event: `MaxListenersExceededWarning`.

```javascript
// Leak
function handle(req, res) {
  bus.on('update', (d) => res.write(d));
}

// Fixed
function handleFixed(req, res) {
  const onUpdate = (d) => res.write(d);
  bus.on('update', onUpdate);
  req.on('close', () => bus.off('update', onUpdate));
}

// Check for growth
console.log(bus.listenerCount('update'));

// Raise the limit only if the count is truly intentional
bus.setMaxListeners(50);

// Modern cleanup with AbortSignal
const { on } = require('node:events');
const ac = new AbortController();
(async () => {
  for await (const [msg] of on(bus, 'update', { signal: ac.signal })) {
    console.log(msg);
  }
})().catch(() => {});
ac.abort();   // ends the loop and removes the listener
```

#### `error` events
The `error` event is special. If it's emitted with no listener, the emitter **throws** and the process can crash.

```javascript
const emitter = new EventEmitter();

// emitter.emit('error', new Error('boom'));   // throws: nobody is listening

emitter.on('error', (err) => logger.error(err));
emitter.emit('error', new Error('boom'));      // logged

// Real example: a socket error with no listener crashes the service
const net = require('node:net');
const socket = net.connect(9999, 'localhost');
socket.on('error', (err) => console.error('connect failed:', err.code));

// Rejections from async listeners can be routed to 'error'
const safeBus = new EventEmitter({ captureRejections: true });
safeBus.on('work', async () => { throw new Error('async failure'); });
safeBus.on('error', (err) => console.error(err.message));
safeBus.emit('work');
```

### Real-time scenario
**Problem:** Logs fill with `MaxListenersExceededWarning`, then memory grows until the service restarts.
**Diagnosis:** Each WebSocket connection added a listener to a shared Redis subscriber, and disconnects didn't remove them.
**Fix:** Remove the listener on `close`, and use a single Redis listener with a `Map` of connections for fan-out.

### Common mistakes and trade-offs
- Assuming `emit` is async. It isn't.
- Silencing the warning by calling `setMaxListeners(0)` instead of finding the leak.
- Using events for flows that need a result or confirmation. A plain function call is clearer.
- **Trade-off:** events decouple code but hide control flow, which makes tracing harder.

### Top interview questions
**Q1. Is `emit` synchronous?**
Yes. All listeners run in order before `emit` returns. Use `setImmediate` or a queue to defer.

**Q2. What happens if you emit `error` with no listener?**
The emitter throws the error, which crashes the process if it's uncaught.

**Q3. What does `MaxListenersExceededWarning` mean?**
More than 10 listeners were added to one event. It's a hint at a possible leak, not an error.

**Q4. What is the difference between `on` and `once`?**
`on` persists until removed. `once` removes itself after the first call.

**Q5. What happens if an async listener rejects?**
`emit` doesn't see it, so it becomes an unhandled rejection. Handle it inside the listener or set `captureRejections: true`.

**Q6. How do you promisify a single event?**
Use `events.once(emitter, 'ready')`. It resolves on the event and rejects on `error`.

**Q7. When should you avoid EventEmitter?**
When events must survive a crash or cross processes. Use a message queue. In-process events are lost on restart.

### Quick revision
Observer pattern. `emit` is synchronous. Unhandled `error` throws. Remove listeners to avoid leaks.

---

## 13. Cancellation and Timeouts

### Why it matters
Every outgoing call needs a time limit. Without one, a slow dependency ties up your resources until the whole service degrades.

### Explanation

#### `AbortController`
The standard way to cancel async work. The controller triggers, and the `signal` goes to the operation.

```javascript
const ac = new AbortController();

const request = fetch('https://api.example.com/report', { signal: ac.signal });
setTimeout(() => ac.abort(), 2000);

try {
  const res = await request;
  console.log(res.status);
} catch (err) {
  if (err.name === 'AbortError') console.log('cancelled');
  else throw err;
}

// Cancel downstream work when the client disconnects
app.get('/report', async (req, res) => {
  const ac = new AbortController();
  res.on('close', () => { if (!res.writableFinished) ac.abort(); });
  const data = await buildReport({ signal: ac.signal });
  res.json(data);
});

// Your own function can honour a signal
async function processAll(items, { signal }) {
  for (const item of items) {
    signal.throwIfAborted();   // check between steps
    await handle(item);
  }
}
```

#### Timers
`AbortSignal.timeout` is the shortest route to a timeout, and `AbortSignal.any` combines signals. `timers/promises` gives awaitable timers.

```javascript
const { setTimeout: sleep } = require('node:timers/promises');

// Timeout in a single line. It raises a TimeoutError.
const res = await fetch(url, { signal: AbortSignal.timeout(3000) });

// Combine user cancellation and a timeout
const signal = AbortSignal.any([userAbort.signal, AbortSignal.timeout(5000)]);

// Awaitable sleep that can be cancelled
await sleep(1000, undefined, { signal });

// Periodic work without overlap: wait for the task, then schedule the next
async function poll() {
  while (true) {
    await refreshCache();
    await sleep(30_000);
  }
}

// A timer shouldn't keep the process alive
setInterval(reportMetrics, 10_000).unref();
```

Use a **deadline budget** for chains: if the caller allows 2 s and 1.5 s is gone, pass 500 ms to the next call.

#### Retries with backoff
Retry only transient failures. Use **exponential backoff** with **jitter**, so failed clients don't all retry at once.

```javascript
const { setTimeout: sleep } = require('node:timers/promises');

async function retry(fn, { retries = 3, baseMs = 200, maxMs = 5000 } = {}) {
  for (let attempt = 0; ; attempt++) {
    try {
      return await fn();
    } catch (err) {
      const retryable = err.status >= 500 || err.status === 429 || err.code === 'ECONNRESET';
      if (!retryable || attempt >= retries) throw err;

      const ceiling = Math.min(maxMs, baseMs * 2 ** attempt);
      await sleep(Math.random() * ceiling);    // full jitter
    }
  }
}

const data = await retry(() => paymentApi.getStatus(id));
```

Only retry operations that are safe to repeat. For a payment, send an **idempotency key** (Topic 28).

### Real-time scenario
**Problem:** A partner API hangs. Within minutes, all your requests hang too and the service runs out of connections.
**Diagnosis:** Outgoing calls had no timeout, so requests piled up. Unjittered retries then hit the recovering partner in waves.
**Fix:** Add a 2 s timeout with `AbortSignal.timeout`, backoff with jitter, and a circuit breaker (Topic 49).

### Common mistakes and trade-offs
- Using `Promise.race` for a timeout but never stopping the underlying work.
- Retrying 4xx errors or non-idempotent operations.
- Retrying at every layer, which multiplies traffic: 3 layers × 3 tries = 27 calls.
- **Trade-off:** short timeouts protect you but risk failing slow valid requests. Base them on the dependency's p99.

### Top interview questions
**Q1. How do you cancel a `fetch` in Node?**
Pass an `AbortSignal`. Call `abort()` on the controller, or use `AbortSignal.timeout(ms)`.

**Q2. Why use jitter?**
Without it, clients retry at the same moments and cause bursts that block recovery. Random delays spread the load.

**Q3. What should you retry?**
Transient failures: network resets, 429, 503 and timeouts on idempotent operations. Not validation or auth errors.

**Q4. What does `timer.unref()` do?**
It stops that timer from keeping the process alive. It's useful for background metrics and cleanup.

**Q5. What is the difference between `Promise.race` timeouts and `AbortController`?**
`race` only stops waiting. `AbortController` tells the operation to stop, which frees sockets and work.

**Q6. Why avoid `setInterval` for async polling?**
Runs can overlap if a task takes longer than the interval. A loop that awaits and then sleeps avoids that.

### Quick revision
`AbortSignal.timeout` on every external call. Retry only transient errors, with exponential backoff and jitter. `unref` background timers.

---

## 14. Async Context

### Why it matters
In a multi-threaded server, per-request data can live in thread-local storage. Node serves many requests on one thread, so it needs another way to track request context across `await`.

### Explanation

#### `AsyncLocalStorage`
It stores a value that follows the async chain, with no need to pass it through every function.

```javascript
const { AsyncLocalStorage } = require('node:async_hooks');

const storage = new AsyncLocalStorage();

async function repository() {
  await new Promise((r) => setTimeout(r, Math.random() * 50));
  console.log('user:', storage.getStore().userId);
}

// Two concurrent runs keep separate context
storage.run({ userId: 'A' }, repository);
storage.run({ userId: 'B' }, repository);
```

`storage.getStore()` returns `undefined` outside `run()`. Prefer `run` to `enterWith`, because it has a clear scope.

#### Request tracing
Create context once in middleware, then read it anywhere. Loggers can add the ID automatically.

```javascript
const { AsyncLocalStorage } = require('node:async_hooks');
const { randomUUID } = require('node:crypto');
const pino = require('pino');

const context = new AsyncLocalStorage();
const baseLogger = pino();

// 1. Middleware opens the context
app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] ?? randomUUID();
  res.setHeader('x-request-id', requestId);
  context.run({ requestId, userId: req.user?.id }, next);
});

// 2. A logger reads it, so no function needs a logger parameter
const logger = {
  info: (msg, data = {}) => baseLogger.info({ ...context.getStore(), ...data }, msg),
};

// 3. Deep code logs with the request ID automatically
async function chargeCard(order) {
  logger.info('charging card', { orderId: order.id });
}

// 4. Forward the ID to other services for tracing
async function callInventory(sku) {
  return fetch(`http://inventory/items/${sku}`, {
    headers: { 'x-request-id': context.getStore()?.requestId ?? '' },
  });
}
```

OpenTelemetry uses the same mechanism to keep the active span during a request.

### Real-time scenario
**Problem:** During an incident, logs from 5,000 concurrent requests are interleaved, so one failed payment can't be traced.
**Diagnosis:** A request ID was passed by hand through a few functions and lost in deeper layers.
**Fix:** Add `AsyncLocalStorage` middleware and a context-aware logger. One ID now filters every log line, across services.

### Common mistakes and trade-offs
- Storing per-request data in a module variable. Requests overwrite each other.
- Using it as hidden global state for business inputs. Pass real parameters explicitly.
- Context can be lost with old callback libraries or custom pools. `AsyncResource` can rebind it.
- **Trade-off:** convenient and clean, with a small overhead and less obvious data flow.

### Top interview questions
**Q1. What is `AsyncLocalStorage`?**
A Node API that keeps a store through an async call chain. It's similar to thread-local storage, for async code.

**Q2. Why not use a global variable for the current user?**
Requests interleave at each `await`, so one request would read another's value. That's a serious data-leak bug.

**Q3. What is a typical use case?**
Request IDs in logs, tenant ID, auth context and tracing spans.

**Q4. What is the difference between `run` and `enterWith`?**
`run` scopes the store to a callback. `enterWith` sets it for the rest of the current execution, which can leak context.

**Q5. How does it relate to `async_hooks`?**
It came from `async_hooks`. Use `AsyncLocalStorage`, because raw `async_hooks` is lower level and slower.

### Quick revision
`AsyncLocalStorage` = per-request context across `await`. Set it once in middleware, read it anywhere. Use it for logs and tracing.

---

# Part 4: Core Modules and Data Handling

## 15. ⭐ Buffers and Binary Data

### Why it matters
Files, sockets, crypto and images all deal in bytes. Buffers are how Node handles them, and misuse leads to corrupted text and even leaked memory contents.

### Explanation

#### What a Buffer is
A Buffer is a fixed-size chunk of raw bytes, stored outside the V8 heap.

```javascript
const buf = Buffer.from('Hi');

console.log(buf);           // <Buffer 48 69>
console.log(buf[0]);        // 72, the byte value of 'H'
console.log(buf.length);    // 2 bytes

buf[0] = 0x4a;              // Buffers are mutable
console.log(buf.toString()); // 'Ji'
```

#### Encoding
Encoding maps bytes to text. Characters and bytes aren't the same count.

```javascript
const text = 'नमस्ते';

console.log(text.length);              // UTF-16 code units
console.log(Buffer.byteLength(text));  // bytes in UTF-8: larger

const b = Buffer.from('hello');
console.log(b.toString('hex'));        // '68656c6c6f'
console.log(b.toString('base64'));     // 'aGVsbG8='
console.log(Buffer.from('aGVsbG8=', 'base64').toString()); // 'hello'
```

A multi-byte character can be split across two chunks. `chunk.toString()` on each piece then corrupts it.

```javascript
const { StringDecoder } = require('node:string_decoder');

const rupee = Buffer.from('₹');        // 3 bytes: e2 82 b9
const part1 = rupee.subarray(0, 1);
const part2 = rupee.subarray(1);

console.log(part1.toString() + part2.toString()); // broken characters

const decoder = new StringDecoder('utf8');
console.log(decoder.write(part1) + decoder.write(part2)); // '₹'

// With streams, setEncoding handles this for you
// stream.setEncoding('utf8');
```

#### Allocation

```javascript
const safe = Buffer.alloc(1024);          // zero-filled, safe
const fast = Buffer.allocUnsafe(1024);    // old memory, must be overwritten
fast.fill(0);

// Small buffers come from a shared 8 KB pool, which is why allocation is fast
console.log(Buffer.poolSize);

// Slices share memory: changing one changes the other
const original = Buffer.from('hello');
const view = original.subarray(0, 2);
view[0] = 0x4a;
console.log(original.toString());         // 'Jello'

// Use a real copy when you keep a small part of a large buffer
const copy = Buffer.from(original.subarray(0, 2));

// Join chunks once, at the end
const chunks = [];
stream.on('data', (c) => chunks.push(c));
stream.on('end', () => {
  const whole = Buffer.concat(chunks);
  console.log(whole.length);
});
```

`allocUnsafe` memory can contain old data, including secrets. Never send it before you overwrite it.

#### `Buffer` vs `TypedArray`
`Buffer` is a subclass of `Uint8Array`, with Node-specific helpers.

| | `Buffer` | `Uint8Array` |
|---|---|---|
| Available in | Node | Everywhere |
| Encoding helpers | `toString('hex')`, `from(str, 'base64')` | Needs `TextDecoder` |
| Binary read/write | `readUInt32BE` and others | Needs `DataView` |
| `slice` | Legacy: shares memory | Copies |

```javascript
const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf instanceof Uint8Array);   // true

// Parse a binary header: 2-byte type, 4-byte length, big-endian
const header = Buffer.alloc(6);
header.writeUInt16BE(7, 0);
header.writeUInt32BE(1024, 2);
console.log(header.readUInt16BE(0), header.readUInt32BE(2)); // 7 1024

// Portable code uses standard APIs
const bytes = new TextEncoder().encode('hello');
console.log(new TextDecoder().decode(bytes));
```

### Real-time scenario
**Problem:** Names in Hindi and emoji sometimes appear broken in a log-processing tool, but only in large files.
**Diagnosis:** Each stream chunk was decoded with `chunk.toString()`, and characters split at chunk boundaries were corrupted.
**Fix:** Use `stream.setEncoding('utf8')` or `StringDecoder`, which holds back incomplete bytes for the next chunk.

### Common mistakes and trade-offs
- Keeping a tiny `subarray` of a big Buffer. It keeps the entire parent in memory.
- Using `string.length` for a `Content-Length` header. Use `Buffer.byteLength`.
- Concatenating chunks as strings for binary data.
- **Trade-off:** `allocUnsafe` is faster, but needs care. Use `alloc` unless you've measured a real need.

### Top interview questions
**Q1. What is a Buffer, and why does Node have it?**
A fixed-size byte array outside the V8 heap. Node needs it for binary data from files and networks, which JS strings can't hold safely.

**Q2. What is the difference between `Buffer.alloc` and `allocUnsafe`?**
`alloc` zero-fills. `allocUnsafe` is faster but may contain old memory contents, so it can expose sensitive data.

**Q3. Is `Buffer` memory part of the V8 heap?**
No. Its contents sit outside the heap, so memory shows up in `external` and `rss`. `--max-old-space-size` doesn't cap it.

**Q4. What is the relationship between `Buffer` and `Uint8Array`?**
`Buffer` extends `Uint8Array`. It adds encodings and binary read and write helpers.

**Q5. Do `slice` and `subarray` copy data?**
On a Buffer, both share memory. Changes affect the original. Use `Buffer.from(view)` to copy.

**Q6. How do you safely decode UTF-8 from a stream?**
Use `setEncoding('utf8')` or `StringDecoder`, which handle characters split across chunks.

### Quick revision
Bytes outside the V8 heap. `alloc` is safe, `allocUnsafe` is fast. `subarray` shares memory. Characters aren't bytes.

---

## 16. ⭐ Streams

### Why it matters
Streams process data piece by piece, so memory stays flat regardless of file size. They're behind HTTP, files, compression and sockets, and they're a staple of senior interviews.

### Explanation

Without streams, the full file sits in memory for each request. With them, only small chunks do.

```javascript
const fs = require('node:fs');

// Loads 2 GB into memory per request
app.get('/video-bad', async (req, res) => {
  res.end(await fs.promises.readFile('video.mp4'));
});

// Constant memory, regardless of size
app.get('/video', (req, res) => {
  require('node:stream').pipeline(fs.createReadStream('video.mp4'), res, (err) => {
    if (err) console.error(err);
  });
});
```

#### The four stream types

| Type | Purpose | Examples |
|---|---|---|
| Readable | Source of data | `fs.createReadStream`, `req` |
| Writable | Destination | `fs.createWriteStream`, `res` |
| Duplex | Both, independent sides | TCP socket |
| Transform | Duplex that changes data | `zlib.createGzip` |

```javascript
const { Readable, Writable, Duplex, Transform } = require('node:stream');

// Readable: produce data
const numbers = Readable.from([1, 2, 3]);

// Writable: consume data
const printer = new Writable({
  write(chunk, encoding, callback) {
    console.log('got', chunk.toString());
    callback();                    // signals that it's ready for more
  },
});

// Transform: input to output
const upper = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString().toUpperCase());
  },
});

// Duplex: read side and write side are separate
const duplex = new Duplex({
  read() { this.push('from read side'); this.push(null); },
  write(chunk, encoding, callback) { console.log('written:', chunk.toString()); callback(); },
});
```

#### `pipeline`
`.pipe()` doesn't forward errors or clean up. `pipeline` does both, so use it in production.

```javascript
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');
const zlib = require('node:zlib');

// Risky: an error in one stream leaves the others open
fs.createReadStream('in.log').pipe(zlib.createGzip()).pipe(fs.createWriteStream('in.log.gz'));

// Safe: any error destroys all streams and rejects
await pipeline(
  fs.createReadStream('in.log'),
  zlib.createGzip(),
  fs.createWriteStream('in.log.gz'),
);

// Generator functions can be pipeline stages
await pipeline(
  fs.createReadStream('in.log', { encoding: 'utf8' }),
  async function* (source) {
    for await (const chunk of source) yield chunk.toUpperCase();
  },
  fs.createWriteStream('out.log'),
);
```

#### Object mode
By default, streams carry Buffers or strings. With `objectMode: true`, each chunk is a JS object, and `highWaterMark` counts objects, not bytes.

```javascript
const { Transform, Readable } = require('node:stream');
const { pipeline } = require('node:stream/promises');

const addTax = new Transform({
  objectMode: true,
  transform(order, encoding, callback) {
    callback(null, { ...order, total: order.amount * 1.18 });
  },
});

const toJsonLine = new Transform({
  writableObjectMode: true,            // objects in, text out
  transform(row, encoding, callback) {
    callback(null, JSON.stringify(row) + '\n');
  },
});

await pipeline(
  Readable.from([{ id: 1, amount: 100 }, { id: 2, amount: 200 }]),
  addTax,
  toJsonLine,
  process.stdout,
);
```

#### Async iteration
Readable streams are async iterables. `for await` is the simplest way to consume one, and it respects backpressure.

```javascript
const fs = require('node:fs');
const readline = require('node:readline');

// Process a huge file line by line
async function countErrors(path) {
  const lines = readline.createInterface({
    input: fs.createReadStream(path),
    crlfDelay: Infinity,
  });

  let errors = 0;
  for await (const line of lines) {
    if (line.includes('ERROR')) errors++;
  }
  return errors;
}

// Read an HTTP body
async function readBody(req) {
  const chunks = [];
  for await (const chunk of req) chunks.push(chunk);
  return Buffer.concat(chunks).toString();
}
```

### Real-time scenario
**Problem:** A CSV export for 3 million rows kills the pod with an OOM error.
**Diagnosis:** The endpoint loaded all rows, built one huge string and then sent it.
**Fix:** Pipe a database cursor through a CSV transform to the response using `pipeline`. Memory drops to tens of MB and the first byte arrives immediately.

### Common mistakes and trade-offs
- Using `.pipe()` without error handling. It leaks file descriptors and sockets.
- Adding a `data` listener, which switches to flowing mode and can lose backpressure control.
- Forgetting to call the callback in `write` or `transform`, which stalls the stream.
- **Trade-off:** streams use less memory and start sooner, but code is more complex, so use `readFile` for small files.

### Top interview questions
**Q1. What are streams, and why use them?**
Abstractions for handling data in chunks. They keep memory constant and start processing before all data arrives.

**Q2. What are the four types of streams?**
Readable, Writable, Duplex and Transform. A Transform is a Duplex whose output depends on its input.

**Q3. What is the difference between `pipe` and `pipeline`?**
Both connect streams and manage backpressure. Only `pipeline` forwards errors, destroys all streams and reports completion.

**Q4. What is the difference between flowing and paused mode?**
Flowing pushes data through `data` events. Paused requires `read()` calls. `pipe`, `data` listeners and `resume` start flowing.

**Q5. What is object mode?**
Streams that carry JS objects instead of bytes. `highWaterMark` then counts objects.

**Q6. How do you handle a 10 GB file in Node?**
Stream it with `createReadStream`, process it with Transforms or `readline`, and use `pipeline`. Never load it whole.

**Q7. What is the difference between Duplex and Transform?**
Duplex sides are independent, like a socket. In a Transform, writes produce the reads.

### Quick revision
Readable, Writable, Duplex, Transform. Use `pipeline`, not `.pipe()`. Object mode carries objects. `for await` is the simplest reader.

---

## 17. ⭐ Backpressure

### Why it matters
When a producer is faster than a consumer, data piles up in memory. Backpressure is how the consumer says "slow down". It's what separates a demo stream from a production one.

### Explanation

```mermaid
flowchart LR
    A["Fast source<br/>disk, 500 MB/s"] --> B["Buffer<br/>highWaterMark"]
    B --> C["Slow destination<br/>client, 2 MB/s"]
    C -. "write() returned false: stop" .-> A
    C -. "drain event: continue" .-> A
```

#### `highWaterMark`
A threshold for a stream's internal buffer. It's a **signal**, not a hard limit, and a producer that ignores it can go far past it. Defaults depend on the stream and Node version. It's bytes in normal mode and objects in object mode.

```javascript
const fs = require('node:fs');
const { Writable } = require('node:stream');

const reader = fs.createReadStream('big.log', { highWaterMark: 64 * 1024 });

const slow = new Writable({
  highWaterMark: 16 * 1024,
  write(chunk, enc, cb) { setTimeout(cb, 10); },
});

console.log(reader.readableHighWaterMark);
console.log(slow.writableHighWaterMark);
console.log(slow.writableLength);   // bytes currently buffered
```

#### `drain`
`write()` returns `false` when the buffer reaches the threshold. Stop writing and wait for `drain`.

```javascript
const { once } = require('node:events');
const fs = require('node:fs');

// Wrong: ignores write's return value, so memory grows without limit
function bad(out) {
  for (let i = 0; i < 1e8; i++) out.write(`line ${i}\n`);
}

// Right: pauses when the buffer is full
async function good(out) {
  for (let i = 0; i < 1e8; i++) {
    if (!out.write(`line ${i}\n`)) {
      await once(out, 'drain');
    }
  }
  out.end();
}

good(fs.createWriteStream('out.txt'));
```

`pipe`, `pipeline` and `for await` handle this for you. You only write it by hand when you call `write()` yourself.

```javascript
// What pipe does internally, simplified
source.on('data', (chunk) => {
  if (!dest.write(chunk)) source.pause();
});
dest.on('drain', () => source.resume());
```

#### Real-world handling
The same idea applies far outside Node streams.

```javascript
// 1. HTTP: a slow client slows your read from storage
const { pipeline } = require('node:stream/promises');
app.get('/download', async (req, res) => {
  await pipeline(s3Stream, res);    // paces reads to the client's speed
});

// 2. Queue consumer: bounded in-flight messages
const MAX_IN_FLIGHT = 20;
channel.prefetch(MAX_IN_FLIGHT);    // RabbitMQ stops sending at this limit
channel.consume('orders', async (msg) => {
  await handleMessage(msg);
  channel.ack(msg);
});

// 3. Kafka: pause partitions when downstream is slow
await consumer.run({
  eachMessage: async ({ topic, partition, message }) => {
    if (dbIsOverloaded()) {
      consumer.pause([{ topic, partitions: [partition] }]);
      setTimeout(() => consumer.resume([{ topic, partitions: [partition] }]), 5000);
    }
    await save(message);
  },
});

// 4. Load shedding: reject early when the server is saturated
app.use((req, res, next) => {
  if (monitor.eventLoopDelayMs() > 200) {
    return res.status(503).set('Retry-After', '1').end();
  }
  next();
});
```

### Real-time scenario
**Problem:** A log shipper's memory hits 4 GB whenever the destination slows down.
**Diagnosis:** It called `socket.write()` for each line without checking the return value, so unsent data queued in memory.
**Fix:** Use `pipeline`, or await `drain`. For sustained slowness, spill to disk or drop low-priority logs with a counter.

### Common mistakes and trade-offs
- Treating `highWaterMark` as a hard memory cap.
- Doing async work in a `data` listener without pausing. Chunks keep arriving.
- Unbounded in-memory queues between stages.
- **Trade-off:** backpressure protects memory but moves delay upstream. Sometimes dropping or shedding is better than waiting.

### Top interview questions
**Q1. What is backpressure?**
A flow-control signal from a slow consumer to a fast producer, so that data doesn't pile up in memory.

**Q2. What does it mean when `write()` returns `false`?**
The buffer has reached `highWaterMark`. The chunk is still accepted, but you should stop until `drain` fires.

**Q3. What happens if you ignore backpressure?**
Buffered data grows, latency and GC pressure rise, and the process can run out of memory.

**Q4. Does `pipe` handle backpressure automatically?**
Yes. It pauses the source when `write` returns `false` and resumes on `drain`. `pipeline` does too, with error handling on top.

**Q5. What is `highWaterMark`?**
The buffer threshold where a stream signals that it's full. It's in bytes, or in objects in object mode.

**Q6. How does backpressure work in a chain of Transforms?**
Each stage's buffer fills in turn, from the slowest stage back to the source, which then stops reading.

**Q7. How do you apply this to a queue consumer?**
Limit in-flight messages with prefetch or a concurrency limit. Pause consumption when downstream is overloaded.

### Quick revision
`write()` false → stop → `drain` → continue. `highWaterMark` is a signal. `pipeline` handles it. The same idea applies to queues and load shedding.

---

## 18. File System and Path

### Why it matters
Files are used for config, uploads, logs and exports. The API you pick decides whether one request blocks everyone else.

### Explanation

#### Sync vs async vs streams

```javascript
const fs = require('node:fs');
const fsp = require('node:fs/promises');

// 1. Sync: blocks the event loop. Fine at startup only.
const config = JSON.parse(fs.readFileSync('config.json', 'utf8'));

// 2. Promise: non-blocking, but loads the whole file. Fine for small files.
async function loadTemplate() {
  return fsp.readFile('template.html', 'utf8');
}

// 3. Stream: non-blocking with constant memory. Right for large files.
fs.createReadStream('huge.log').pipe(process.stdout);
```

| API | Blocks loop? | Whole file in memory? | Use for |
|---|---|---|---|
| `readFileSync` | Yes | Yes | Startup config, CLI tools |
| `fs/promises` | No | Yes | Small files in servers |
| `createReadStream` | No | No | Large files, HTTP responses |

#### Path handling
Build paths with `path`, never by joining strings. It handles separators across operating systems.

```javascript
const path = require('node:path');

path.join('uploads', 'images', 'a.png');      // uploads/images/a.png
path.resolve('uploads', 'a.png');             // absolute path from cwd
path.basename('/tmp/report.pdf');             // 'report.pdf'
path.extname('/tmp/report.pdf');              // '.pdf'
path.dirname('/tmp/report.pdf');              // '/tmp'

// Paths relative to this file, not to where the process started
const templates = path.join(__dirname, 'templates');

// Security: block path traversal such as ../../etc/passwd
const UPLOAD_DIR = path.resolve('uploads');

function safePath(userInput) {
  const full = path.resolve(UPLOAD_DIR, userInput);
  if (!full.startsWith(UPLOAD_DIR + path.sep)) {
    throw new Error('invalid path');
  }
  return full;
}
```

#### File watching
`fs.watch` reports changes through the OS, but behaviour differs by platform and events can repeat. Debounce them, or use `chokidar` for robustness.

```javascript
const fs = require('node:fs');

let timer;
fs.watch('config.json', () => {
  clearTimeout(timer);
  timer = setTimeout(reloadConfig, 100);   // debounce repeated events
});

// Promise version with for await
const { watch } = require('node:fs/promises');
const ac = new AbortController();

(async () => {
  for await (const event of watch('./config', { signal: ac.signal })) {
    console.log(event.eventType, event.filename);
  }
})().catch((err) => {
  if (err.name !== 'AbortError') throw err;
});
```

#### Large files
Stream them, and write safely. Readers should never see half-written files.

```javascript
const fs = require('node:fs');
const fsp = require('node:fs/promises');
const crypto = require('node:crypto');
const { pipeline } = require('node:stream/promises');

// Hash a 50 GB file with constant memory
async function sha256(path) {
  const hash = crypto.createHash('sha256');
  await pipeline(fs.createReadStream(path), hash);
  return hash.digest('hex');
}

// Read only one part with a byte range
const part = fs.createReadStream('video.mp4', { start: 1000, end: 1999 });

// Atomic write: write to a temp file, then rename
async function writeAtomic(path, data) {
  const temp = `${path}.${process.pid}.tmp`;
  await fsp.writeFile(temp, data);
  await fsp.rename(temp, path);      // atomic on the same file system
}

// Use try/catch instead of checking first, which avoids a race
try {
  await fsp.readFile('maybe.json');
} catch (err) {
  if (err.code !== 'ENOENT') throw err;
}
```

### Real-time scenario
**Problem:** Throughput drops by 70% after a release that added a report template.
**Diagnosis:** The new code called `fs.readFileSync` inside the request handler, so each request blocked the loop.
**Fix:** Load the template once at startup, or asynchronously with a cache.

### Common mistakes and trade-offs
- `fs.exists` followed by an operation, which is a race condition.
- Relative paths that depend on where the process was started.
- Using user input in a path without validation.
- Too many open files, causing `EMFILE`. Limit concurrency.
- **Trade-off:** sync is simple, and fine in startup code and CLIs. It's only dangerous on the request path.

### Top interview questions
**Q1. When is `readFileSync` acceptable?**
At startup, in CLI tools and in scripts. Never during request handling.

**Q2. What is the difference between `readFile` and `createReadStream`?**
`readFile` loads everything before you get it. `createReadStream` delivers chunks with constant memory.

**Q3. What is the difference between `path.join` and `path.resolve`?**
`join` concatenates and normalizes. `resolve` produces an absolute path, processing right to left and using cwd if needed.

**Q4. How do you prevent path traversal?**
Resolve the final path against a base folder and check that it stays inside. Prefer generated file names.

**Q5. How do you write a file atomically?**
Write to a temp file in the same folder, then rename it over the target.

**Q6. What does `EMFILE` mean?**
Too many open files. Limit concurrent operations, fix leaked handles, and raise the OS limit only if it's justified.

### Quick revision
Sync only at startup. Promises for small files. Streams for large ones. `path` for paths. Validate user paths. Temp file + rename is atomic.

---

## 19. HTTP and Networking

### Why it matters
Frameworks hide this layer, but production issues such as socket hangs, DNS delays, keep-alive resets and connection limits live here.

### Explanation

#### `http`
Every Node web framework sits on this module. `req` is a Readable stream, and `res` is a Writable stream.

```javascript
const http = require('node:http');

const server = http.createServer(async (req, res) => {
  if (req.method === 'POST' && req.url === '/echo') {
    const chunks = [];
    for await (const chunk of req) chunks.push(chunk);
    const body = Buffer.concat(chunks);

    res.writeHead(200, { 'content-type': 'application/json' });
    return res.end(body);
  }

  res.writeHead(404);
  res.end('not found');
});

server.listen(3000);

// Server timeouts help protect against slow clients
server.requestTimeout = 30_000;
server.headersTimeout = 10_000;
```

#### `https`
Same API plus TLS. In production, TLS is usually handled at a load balancer, but Node can do it directly.

```javascript
const https = require('node:https');
const fs = require('node:fs');

https.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
}, (req, res) => {
  res.end('secure');
}).listen(443);
```

#### `net`
TCP without HTTP. Useful for custom protocols. TCP is a byte stream, so one `data` event doesn't equal one message. You need framing.

```javascript
const net = require('node:net');

// Newline-framed protocol
net.createServer((socket) => {
  let buffer = '';
  socket.setEncoding('utf8');

  socket.on('data', (chunk) => {
    buffer += chunk;
    let i;
    while ((i = buffer.indexOf('\n')) !== -1) {
      const message = buffer.slice(0, i);
      buffer = buffer.slice(i + 1);
      socket.write(`received: ${message}\n`);
    }
  });

  socket.on('error', (err) => console.error(err.code));
}).listen(5000);
```

#### `dns`

```javascript
const dns = require('node:dns');
const dnsp = require('node:dns/promises');

// Uses the OS resolver and the libuv thread pool. Honours /etc/hosts.
dns.lookup('example.com', (err, address) => console.log(address));

// Direct DNS query, without the thread pool
const addresses = await dnsp.resolve4('example.com');
const mx = await dnsp.resolveMx('gmail.com');

// Node doesn't cache DNS itself. Each new connection can look up again.
```

#### Keep-alive
Each new HTTPS connection costs TCP and TLS handshakes. Keep-alive reuses connections. Recent Node versions enable it by default in the global agent, but set it explicitly for important clients.

```javascript
const http = require('node:http');

const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 100,       // cap per host
  maxFreeSockets: 10,    // idle sockets to keep
  timeout: 30_000,
});

http.get({ host: 'inventory.internal', path: '/items', agent }, (res) => res.resume());

// Server side: idle sockets stay open for this long
server.keepAliveTimeout = 65_000;
server.headersTimeout = 66_000;
```

Keep `keepAliveTimeout` **longer** than the load balancer's idle timeout. If Node closes first, the load balancer can reuse a dead socket, which shows up as random 502 errors.

#### HTTP/2
One connection carries many parallel streams, with header compression. Browsers require TLS for it.

```javascript
const http2 = require('node:http2');
const fs = require('node:fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem'),
});

server.on('stream', (stream, headers) => {
  stream.respond({ ':status': 200, 'content-type': 'text/plain' });
  stream.end(`you asked for ${headers[':path']}`);
});

server.listen(8443);
```

| | HTTP/1.1 | HTTP/2 |
|---|---|---|
| Requests per connection | One at a time | Many at once |
| Format | Text | Binary |
| Header compression | No | Yes (HPACK) |
| Common use | Everywhere | gRPC, browsers via proxy |

#### Native `fetch`
Node includes `fetch`, built on Undici. No `axios` needed for most calls.

```javascript
const res = await fetch('https://api.example.com/orders', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ sku: 'A1' }),
  signal: AbortSignal.timeout(3000),
});

// fetch doesn't reject on 404 or 500. Check it yourself.
if (!res.ok) {
  throw new Error(`orders failed: ${res.status}`);
}
const data = await res.json();

// Stream a response body to disk
const { pipeline } = require('node:stream/promises');
const { Readable } = require('node:stream');
const fs = require('node:fs');

const file = await fetch('https://example.com/big.zip');
await pipeline(Readable.fromWeb(file.body), fs.createWriteStream('big.zip'));
```

### Real-time scenario
**Problem:** About 0.1% of requests fail with 502 at the AWS load balancer, with no matching errors in app logs.
**Diagnosis:** The ALB's idle timeout was 60 s and Node's `keepAliveTimeout` was 5 s, so Node closed sockets the ALB was about to reuse.
**Fix:** Set `keepAliveTimeout` to 65 s and `headersTimeout` slightly higher.

### Common mistakes and trade-offs
- Not setting timeouts on outgoing requests.
- Forgetting that `fetch` resolves on 4xx and 5xx.
- Not reading a response body, which can hold the connection open.
- Assuming one TCP `data` event is one message.
- **Trade-off:** a big `maxSockets` value gives more concurrency, but it can overload the target and hide a slowdown.

### Top interview questions
**Q1. What are `req` and `res` in Node's HTTP server?**
`req` is a Readable stream, and `res` is a Writable stream. That's why you can stream bodies in both directions.

**Q2. What is keep-alive, and why does it matter?**
It reuses TCP and TLS connections across requests, which removes handshake latency and reduces CPU and port use.

**Q3. Why might an ALB return random 502s for a Node app?**
Node closed an idle connection the ALB reused. Set `keepAliveTimeout` above the load balancer's idle timeout.

**Q4. What are the benefits of HTTP/2?**
Multiplexing, binary framing and header compression on one connection. TCP-level head-of-line blocking still exists.

**Q5. Does `fetch` throw on a 500 response?**
No. It rejects only on network failures or aborts. Check `res.ok`.

**Q6. Why isn't one TCP `data` event one message?**
TCP is a byte stream with no message boundaries. You need framing, such as a delimiter or length prefix.

### Quick revision
`req` and `res` are streams. Reuse connections. `keepAliveTimeout` must exceed the LB idle timeout. `fetch` needs `res.ok` and a timeout.

---

## 20. Crypto

### Why it matters
Mistakes here cause real breaches. Interviewers want to see that you know which primitive fits which job.

### Explanation

#### Hashing
A one-way fingerprint for integrity checks. Use SHA-256 or stronger. Use **HMAC** when you also need to prove the sender knows a secret.

```javascript
const crypto = require('node:crypto');

// File or content integrity
const hash = crypto.createHash('sha256').update('hello').digest('hex');

// HMAC: verify a webhook signature
function verifyWebhook(rawBody, signatureHex, secret) {
  const expected = crypto.createHmac('sha256', secret).update(rawBody).digest();
  const received = Buffer.from(signatureHex, 'hex');

  // timingSafeEqual needs equal lengths and prevents timing attacks
  return received.length === expected.length &&
    crypto.timingSafeEqual(received, expected);
}
```

Use the **raw** body for webhook checks. Parsed-then-reserialized JSON can differ by a byte, and the check will fail.

#### Encryption
Encryption is reversible with a key. Use **AES-256-GCM**, which gives confidentiality and tamper detection. Never reuse an IV with the same key.

```javascript
const crypto = require('node:crypto');

const key = crypto.randomBytes(32);      // in practice, from a KMS or secrets manager

function encrypt(plainText) {
  const iv = crypto.randomBytes(12);     // new IV each time
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  const data = Buffer.concat([cipher.update(plainText, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return Buffer.concat([iv, tag, data]).toString('base64');
}

function decrypt(payload) {
  const raw = Buffer.from(payload, 'base64');
  const iv = raw.subarray(0, 12);
  const tag = raw.subarray(12, 28);
  const data = raw.subarray(28);

  const decipher = crypto.createDecipheriv('aes-256-gcm', key, iv);
  decipher.setAuthTag(tag);              // final() throws if data was changed
  return Buffer.concat([decipher.update(data), decipher.final()]).toString('utf8');
}

console.log(decrypt(encrypt('card token')));
```

#### Password storage
Passwords need a **slow, salted** hash: Argon2id, scrypt or bcrypt. SHA-256 is far too fast, so attackers can try billions of guesses.

```javascript
const crypto = require('node:crypto');
const { promisify } = require('node:util');
const scrypt = promisify(crypto.scrypt);

async function hashPassword(password) {
  const salt = crypto.randomBytes(16);
  const derived = await scrypt(password, salt, 64);
  return `${salt.toString('hex')}:${derived.toString('hex')}`;
}

async function verifyPassword(password, stored) {
  const [saltHex, hashHex] = stored.split(':');
  const expected = Buffer.from(hashHex, 'hex');
  const actual = await scrypt(password, Buffer.from(saltHex, 'hex'), 64);
  return crypto.timingSafeEqual(actual, expected);
}
```

Always use the async versions. `scryptSync` or `bcrypt.hashSync` in a handler blocks the loop for each login.

#### Random values
`Math.random` is predictable, so never use it for security. Use `crypto`.

```javascript
const crypto = require('node:crypto');

crypto.randomUUID();                       // unique IDs
crypto.randomBytes(32).toString('hex');    // session or reset tokens
crypto.randomInt(100000, 1000000);         // 6-digit OTP, without bias

// Store only a hash of reset tokens, so a DB leak can't be used to log in
const token = crypto.randomBytes(32).toString('hex');
const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
```

### Real-time scenario
**Problem:** A pen test shows password reset links can be predicted.
**Diagnosis:** Tokens came from `Math.random().toString(36)` and were stored in plain text.
**Fix:** Generate tokens with `crypto.randomBytes(32)`, store only their hash, add a short expiry and make them single-use.

### Common mistakes and trade-offs
- Using MD5 or SHA-1 for security purposes.
- Comparing secrets with `===`. Use `timingSafeEqual`.
- Hard-coding keys, or reusing an IV with GCM.
- **Trade-off:** higher password hash cost resists attacks but adds login latency and CPU. Tune it to around 100–300 ms on your hardware.

### Top interview questions
**Q1. What is the difference between hashing and encryption?**
Hashing is one-way, for integrity and verification. Encryption is reversible with a key, for confidentiality.

**Q2. How should passwords be stored?**
With a slow, salted hash: Argon2id, scrypt or bcrypt. Never plain text or a fast hash.

**Q3. What is a salt?**
Random data per password. It defeats rainbow tables, and identical passwords get different hashes.

**Q4. Why use `timingSafeEqual`?**
Normal comparison exits at the first difference, so response time can reveal the secret. This compares in constant time.

**Q5. Why is AES-GCM preferred?**
It's authenticated encryption, so tampering is detected on decrypt. Plain CBC doesn't provide that.

**Q6. Why not use `Math.random` for tokens?**
Its output is predictable. `crypto.randomBytes` uses a secure source.

### Quick revision
Hash for integrity, HMAC for authenticity, AES-GCM for secrets, Argon2/scrypt/bcrypt for passwords, `crypto.random*` for tokens. Compare with `timingSafeEqual`.

---

## 21. Process and Environment

### Why it matters
The way a process starts, reads config, handles signals and exits decides how it behaves in Docker, Kubernetes and CI.

### Explanation

#### `process`

```javascript
console.log(process.pid, process.ppid);
console.log(process.cwd());         // where the process started
console.log(process.argv.slice(2)); // node app.js --port 3000 → ['--port', '3000']
console.log(process.platform);
console.log(process.hrtime.bigint()); // high-resolution timer

// Built-in argument parser
const { parseArgs } = require('node:util');
const { values } = parseArgs({
  options: { port: { type: 'string', default: '3000' } },
});
console.log(values.port);

// Standard streams
process.stdout.write('output\n');
process.stderr.write('errors go here\n');
```

#### Signals
The OS uses signals to talk to a process. `SIGTERM` is a polite stop. `SIGKILL` can't be caught.

| Signal | Meaning | Sent by | Can you catch it? |
|---|---|---|---|
| `SIGINT` | Interrupt | Ctrl+C | Yes |
| `SIGTERM` | Please stop | Docker, Kubernetes | Yes |
| `SIGKILL` | Stop now | OS, Kubernetes after grace period | No |
| `SIGHUP` | Terminal closed | Often used to reload | Yes |

```javascript
function shutdown(signal) {
  console.log(`received ${signal}`);
  server.close(() => process.exit(0));
  setTimeout(() => process.exit(1), 10_000).unref();  // force exit
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

Topic 48 covers this in full.

#### Environment variables
Config that changes between environments lives in env vars. All values are **strings**, so validate them at startup.

```javascript
// Load a .env file without a package (recent Node)
// node --env-file=.env app.js
// or in code: process.loadEnvFile();

// Every value is a string
console.log(typeof process.env.PORT);        // 'string'
if (process.env.ENABLED) { /* 'false' is truthy! */ }

// Validate once, at startup
function requireEnv(name) {
  const value = process.env[name];
  if (!value) throw new Error(`missing env var: ${name}`);
  return value;
}

const config = Object.freeze({
  port: Number(process.env.PORT ?? 3000),
  databaseUrl: requireEnv('DATABASE_URL'),
  enabled: process.env.ENABLED === 'true',
});
```

`NODE_ENV=production` matters. Express and many libraries turn on caching and turn off debug output with it.

#### `os`
Information about the host.

```javascript
const os = require('node:os');

console.log(os.availableParallelism());  // CPUs available to this process
console.log(os.totalmem(), os.freemem());
console.log(os.hostname());
console.log(os.loadavg());               // [1, 5, 15] minute load averages
console.log(os.tmpdir());
```

In a container, `os.cpus()` can show the host's CPU count, not your container's quota, so a worker pool can end up oversized. Set pool sizes through config.

#### Exit codes
`0` means success. Anything else means failure. CI and orchestrators read this value.

```javascript
// Prefer setting exitCode, which lets pending output flush
process.exitCode = 1;

// process.exit() stops right away, and can cut off async log writes
// process.exit(1);

// 'exit' handlers must be synchronous
process.on('exit', (code) => console.log('exiting with', code));

// 'beforeExit' runs when the loop empties, and can schedule async work.
// It doesn't fire on process.exit() or on an uncaught exception.
process.on('beforeExit', () => console.log('loop is empty'));
```

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Uncaught fatal exception |
| 130 | Shell convention: stopped by `SIGINT` (128 + 2) |
| 137 | Shell convention: `SIGKILL`, often OOM (128 + 9) |
| 143 | Shell convention: `SIGTERM` (128 + 15) |

### Real-time scenario
**Problem:** Every deploy causes a burst of failed requests, and pods exit with code 137.
**Diagnosis:** Node ran as a child of `npm start`, which didn't forward `SIGTERM`, so Kubernetes hit the grace limit and sent `SIGKILL`.
**Fix:** Start with `node dist/index.js` directly in the Dockerfile, and add a `SIGTERM` handler.

### Common mistakes and trade-offs
- Reading `process.env` on hot paths. It's slower than a normal object, so read it once.
- `if (process.env.FLAG)` when the value is the string `'false'`.
- Calling `process.exit()` right after logging. Async logs can be lost.
- **Trade-off:** env vars are simple and portable, but visible to child processes and crash dumps. Use a secrets manager for sensitive values.

### Top interview questions
**Q1. What is the difference between `SIGTERM` and `SIGKILL`?**
`SIGTERM` can be caught for cleanup. `SIGKILL` ends the process right away and can't be caught.

**Q2. What is the difference between `process.exit()` and `process.exitCode`?**
`exit()` stops immediately. `exitCode` sets the code and lets the loop finish, so pending work and logs complete.

**Q3. What does exit code 137 mean?**
128 + 9: the process received `SIGKILL`, usually from the OOM killer or a shutdown timeout.

**Q4. What is the difference between `process.cwd()` and `__dirname`?**
`cwd()` is where the process was started. `__dirname` is the current file's folder.

**Q5. Why does `NODE_ENV=production` matter?**
Libraries use it to change behaviour, including Express view caching and less verbose errors.

**Q6. Why validate env vars at startup?**
So bad config fails the deploy right away, rather than surfacing later on a rare code path.

### Quick revision
`SIGTERM` can be handled, `SIGKILL` can't. Env values are strings, so validate at startup. Prefer `process.exitCode`. 137 = killed.

---

# Part 5: Concurrency and Parallelism

## 22. ⭐ Worker Threads

### Why it matters
Worker threads are Node's answer to CPU-bound work. Interviewers often ask how to run heavy computation without blocking users.

### Explanation

#### CPU-bound work
A worker has its own V8 instance, event loop and heap, inside the same process. It runs JS in parallel with the main thread.

```javascript
// worker.js
const { parentPort, workerData } = require('node:worker_threads');

function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
parentPort.postMessage(fib(workerData.n));
```

```javascript
// main.js
const { Worker } = require('node:worker_threads');

function runFib(n) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData: { n } });
    worker.once('message', resolve);
    worker.once('error', reject);
    worker.once('exit', (code) => {
      if (code !== 0) reject(new Error(`worker exited with ${code}`));
    });
  });
}

app.get('/fib/:n', async (req, res) => {
  res.json({ result: await runFib(Number(req.params.n)) });  // the loop stays free
});
```

Messages are **copied** with the structured clone algorithm, which costs time on big data. Transfer an `ArrayBuffer` to move it without copying.

```javascript
const data = new ArrayBuffer(100 * 1024 * 1024);
worker.postMessage(data, [data]);   // moved, not copied
console.log(data.byteLength);       // 0: no longer usable in the sender
```

#### `SharedArrayBuffer`
Lets threads share the same memory with no copying. Plain reads and writes aren't safe, so you need atomics.

```javascript
const { Worker, isMainThread, workerData } = require('node:worker_threads');

if (isMainThread) {
  const shared = new SharedArrayBuffer(4);
  const counter = new Int32Array(shared);

  const workers = [1, 2].map(() => new Worker(__filename, { workerData: shared }));

  Promise.all(workers.map((w) => new Promise((r) => w.once('exit', r))))
    .then(() => console.log(counter[0]));   // 200000, thanks to Atomics
} else {
  const counter = new Int32Array(workerData);
  for (let i = 0; i < 100_000; i++) Atomics.add(counter, 0, 1);
}
```

#### `Atomics`
Atomic operations can't be interrupted. `counter[0]++` is really read, add, write, and two threads can overlap and lose updates.

```javascript
const shared = new SharedArrayBuffer(8);
const view = new Int32Array(shared);

Atomics.add(view, 0, 1);                  // safe increment
Atomics.load(view, 0);                    // safe read
Atomics.store(view, 0, 5);                // safe write
Atomics.compareExchange(view, 0, 5, 9);   // set to 9 only if it's still 5

// Coordination: a thread sleeps until another wakes it
// Worker:  Atomics.wait(view, 1, 0);     // sleeps while view[1] is 0
// Main:    Atomics.store(view, 1, 1); Atomics.notify(view, 1);
```

#### Worker pools
Creating a worker costs real time and memory. Keep a fixed pool, and queue jobs. Libraries like **Piscina** do this well.

```javascript
const { Worker } = require('node:worker_threads');
const os = require('node:os');

class WorkerPool {
  constructor(file, size = os.availableParallelism()) {
    this.queue = [];
    this.idle = Array.from({ length: size }, () => new Worker(file));
  }

  run(data) {
    return new Promise((resolve, reject) => {
      this.queue.push({ data, resolve, reject });
      this.next();
    });
  }

  next() {
    if (!this.queue.length || !this.idle.length) return;
    const worker = this.idle.pop();
    const { data, resolve, reject } = this.queue.shift();

    const done = (fn) => (value) => {
      worker.removeAllListeners('message');
      worker.removeAllListeners('error');
      this.idle.push(worker);
      fn(value);
      this.next();
    };

    worker.once('message', done(resolve));
    worker.once('error', done(reject));   // production: replace a crashed worker
    worker.postMessage(data);
  }
}

// With Piscina
// const Piscina = require('piscina');
// const pool = new Piscina({ filename: './resize.js', maxThreads: 4 });
// const result = await pool.run({ path });
```

Cap the queue as well. When it's full, return 503 rather than queueing forever.

### Real-time scenario
**Problem:** PDF generation makes API latency jump for all users.
**Diagnosis:** Rendering held the main thread for 400 ms per document, as a CPU profile showed.
**Fix:** Run it in a Piscina pool sized to the CPU quota, with a bounded queue. For large spikes, move it to a background job.

### Common mistakes and trade-offs
- Using workers for I/O. Async I/O is already efficient, so it only adds overhead.
- Creating one worker per request.
- Posting large objects repeatedly. Copying can cost more than the work.
- **Trade-off:** real parallelism in exchange for more memory, serialization cost and more complex debugging.

### Top interview questions
**Q1. When should you use worker threads?**
For CPU-heavy JS: parsing, compression, image work, heavy calculations. Not for I/O.

**Q2. How do workers communicate?**
Through `postMessage`, which copies data, or transfer lists that move it. `SharedArrayBuffer` shares memory.

**Q3. Do workers share memory?**
Not normally: each has its own heap. Only `SharedArrayBuffer` is shared.

**Q4. What is the purpose of `Atomics`?**
It gives indivisible operations on shared memory, preventing lost updates, and supports wait and notify.

**Q5. Why use a worker pool?**
Startup is expensive. A pool reuses threads and bounds parallelism, usually to the CPU count.

**Q6. Do worker threads help a blocked event loop?**
Only if you move the blocking work into one. The main thread must stay free of it.

**Q7. What happens when a worker throws an uncaught error?**
The worker ends, and an `error` event fires on the `Worker` object in the parent. The main process keeps running.

### Quick revision
Workers = CPU parallelism in one process. Separate heaps. Messages are copied. `SharedArrayBuffer` + `Atomics` share memory. Use a pool.

---

## 23. ⭐ Child Processes

### Why it matters
Sometimes you need another program: `ffmpeg`, `git` or a Python script. Child processes also come with a classic security topic: command injection.

### Explanation

#### `spawn`
Starts a process and streams its output. Use it for long-running commands or big output.

```javascript
const { spawn } = require('node:child_process');

const ffmpeg = spawn('ffmpeg', ['-i', 'in.mp4', '-vf', 'scale=640:-1', 'out.mp4']);

ffmpeg.stderr.on('data', (d) => console.log(d.toString()));
ffmpeg.on('error', (err) => console.error('failed to start:', err.message));
ffmpeg.on('close', (code) => console.log('exited with', code));
```

#### `exec`
Runs a command **in a shell** and buffers all output. It's convenient but has an output limit, and it's dangerous with user input.

```javascript
const { exec } = require('node:child_process');
const { promisify } = require('node:util');
const execAsync = promisify(exec);

const { stdout } = await execAsync('ls -la | wc -l', { timeout: 5000 });
console.log(stdout.trim());

// DANGER: command injection
// const file = req.query.file;  // "a.txt; rm -rf /"
// exec(`cat ${file}`);          // the shell runs both commands
```

#### `execFile`
Runs a program **without a shell** by default. Arguments aren't interpreted by a shell, so this is the safer option.

```javascript
const { execFile } = require('node:child_process');
const { promisify } = require('node:util');
const execFileAsync = promisify(execFile);

async function fileInfo(userInput) {
  // Prefer an allow-list as well
  const { stdout } = await execFileAsync('file', ['--brief', '--', userInput], {
    timeout: 5000,
    maxBuffer: 1024 * 1024,
  });
  return stdout.trim();
}
// `--` stops input that starts with a dash from being read as a flag.
```

#### `fork`
A special `spawn` for Node scripts, with a built-in IPC channel.

```javascript
// parent.js
const { fork } = require('node:child_process');

const child = fork('./report-worker.js');
child.send({ reportId: 42 });
child.on('message', (result) => {
  console.log('result:', result);
  child.kill();
});
child.on('exit', (code) => console.log('child exited', code));
```

```javascript
// report-worker.js
process.on('message', async ({ reportId }) => {
  const data = await buildReport(reportId);
  process.send({ reportId, rows: data.length });
});
```

#### IPC
Inter-process communication: messages are serialized over a channel. Processes share no memory, so big payloads are slow. Pass file paths or IDs instead.

```javascript
const { fork } = require('node:child_process');

const child = fork('./worker.js', {
  serialization: 'advanced',   // supports Map, Set, Date and Buffer
});

child.send({ type: 'job', at: new Date(), tags: new Set(['a']) });

// Connection between parent and child
child.on('disconnect', () => console.log('IPC closed'));
```

| Method | Shell? | Output | Best for |
|---|---|---|---|
| `spawn` | No (default) | Streamed | Long tasks, big output |
| `exec` | Yes | Buffered | Short, trusted commands |
| `execFile` | No | Buffered | Running one program safely |
| `fork` | No | IPC + streams | Node scripts |

### Real-time scenario
**Problem:** A security audit achieves remote code execution through an image conversion endpoint.
**Diagnosis:** The code ran `exec(\`convert ${filename} out.png\`)` with a user-supplied file name.
**Fix:** Use `execFile` with an argument array, generate file names on the server, and validate file types. Run the process with low privileges.

### Common mistakes and trade-offs
- Using `exec` on large output, which hits `maxBuffer` and fails.
- Not handling `error`. A missing binary emits it, and an unhandled one crashes the process.
- Leaving a child's stdout unread, which can stall it.
- Forgetting timeouts, leaving hung children around.
- **Trade-off:** processes give the strongest isolation and let you run any language, but start slowly and use more memory than threads.

### Top interview questions
**Q1. What is the difference between `spawn` and `exec`?**
`spawn` streams output with no shell by default. `exec` uses a shell and buffers output up to a limit.

**Q2. What is the difference between `fork` and `spawn`?**
`fork` is `spawn` for Node scripts, and it adds an IPC channel for `send` and `message`.

**Q3. How do you prevent command injection?**
Avoid the shell. Use `execFile` or `spawn` with an argument array, plus allow-lists and server-generated names.

**Q4. Worker threads or child processes?**
Workers suit CPU-heavy JS with lower overhead. Child processes suit other programs or strong isolation.

**Q5. What happens to a child if the parent crashes?**
It usually keeps running as an orphan unless it detects the disconnect. Children should exit on `disconnect`.

**Q6. What is the difference between `close` and `exit` events?**
`exit` fires when the process ends. `close` fires after its stdio streams have also closed.

### Quick revision
`spawn` streams, `exec` uses a shell and buffers, `execFile` is the safe option, `fork` adds IPC. Never put user input into `exec`.

---

## 24. ⭐ Cluster Module

### Why it matters
One Node process uses one main thread, so one core. Cluster shows how Node scales on a machine, and the modern follow-up is whether you still need it in Kubernetes.

### Explanation

#### Multi-core scaling
The primary process forks workers, and each is a full Node process with its own heap and event loop. They share the listening port.

```javascript
const cluster = require('node:cluster');
const http = require('node:http');
const os = require('node:os');

if (cluster.isPrimary) {
  const count = os.availableParallelism();
  for (let i = 0; i < count; i++) cluster.fork();

  // Replace crashed workers
  cluster.on('exit', (worker, code) => {
    if (!worker.exitedAfterDisconnect) {
      console.log(`worker ${worker.process.pid} died with ${code}, restarting`);
      cluster.fork();
    }
  });
} else {
  http.createServer((req, res) => {
    res.end(`handled by ${process.pid}\n`);
  }).listen(3000);
}
```

Workers share **nothing** in memory. In-process caches, sessions and rate-limit counters are per-worker, so move them to Redis.

```javascript
// Bug under cluster: each worker has its own count
let hits = 0;
app.get('/hits', (req, res) => res.json({ hits: ++hits }));

// Fix: use a shared store
app.get('/hits', async (req, res) => res.json({ hits: await redis.incr('hits') }));
```

#### Load distribution
On Linux and macOS, the primary accepts connections and hands them out **round-robin**. On Windows, the OS decides by default. Note that this balances **connections**, not requests. A long-lived keep-alive connection stays on one worker.

```javascript
const cluster = require('node:cluster');

cluster.schedulingPolicy = cluster.SCHED_RR;    // set before fork
// cluster.SCHED_NONE lets the OS pick

// Zero-downtime restart: replace workers one at a time
async function rollingRestart() {
  for (const worker of Object.values(cluster.workers)) {
    const replacement = cluster.fork();
    await new Promise((resolve) => replacement.once('listening', resolve));
    worker.disconnect();    // finish current connections, then exit
  }
}

process.on('SIGUSR2', rollingRestart);
```

#### Cluster vs PM2 vs Kubernetes

| | `cluster` | PM2 | Kubernetes |
|---|---|---|---|
| What it is | Built-in module | Process manager | Orchestrator |
| Multi-core | Yes, with your code | `-i max` | More pods |
| Restart on crash | You write it | Built in | Built in |
| Multi-machine | No | Limited | Yes |
| Best fit | Custom control | Single VMs | Containers at scale |

```bash
# PM2: cluster mode without changing code
pm2 start app.js -i max
pm2 reload app        # zero-downtime reload
pm2 logs
```

```yaml
# Kubernetes: one process per pod, scale with replicas
spec:
  replicas: 6
  template:
    spec:
      containers:
        - name: api
          image: orders-api:1.4.2
          resources:
            requests: { cpu: "500m", memory: "512Mi" }
            limits: { memory: "768Mi" }
```

In Kubernetes, prefer **one process per pod**. Health checks, metrics and autoscaling are simpler, and the platform handles restarts.

### Real-time scenario
**Problem:** A 16-core VM runs at 7% CPU, yet latency climbs under load.
**Diagnosis:** One Node process was maxing out a single core.
**Fix:** Use PM2 cluster mode, and move sessions to Redis, because login sessions were lost when requests moved between workers.

### Common mistakes and trade-offs
- Using in-memory state with several workers.
- Running cluster inside small Kubernetes pods. This confuses CPU limits and health checks.
- Starting more workers than CPU quota allows, which adds context switching.
- **Trade-off:** cluster improves throughput and crash isolation, but each worker duplicates memory and needs shared external state.

### Top interview questions
**Q1. How does cluster work?**
A primary forks workers. Workers share the server port, and the primary distributes connections, usually round-robin.

**Q2. What is the difference between cluster and worker threads?**
Cluster uses separate processes to scale request handling. Worker threads run inside one process for CPU tasks.

**Q3. Why is state a problem with cluster?**
Each worker has its own memory. Sessions, caches and counters aren't shared, so use Redis or a database.

**Q4. Do you need cluster with Kubernetes?**
Usually no. Run one process per pod and scale replicas. It keeps resource limits and health checks simple.

**Q5. How do you do zero-downtime restarts with cluster?**
Replace workers one at a time. Wait until the new one listens, then disconnect the old one so it can drain.

**Q6. How many workers should you run?**
About one per available core, then adjust using load tests and memory limits.

### Quick revision
Cluster = multi-process on one machine. No shared memory. Round-robin by default on Linux. Prefer one process per pod in Kubernetes.

---

## 25. Concurrency Comparison

### Why it matters
This topic ties Part 5 together. Senior interviews often ask: "You have this workload. Which tool do you pick, and why?" Race conditions in a "single-threaded" runtime also surprise many candidates.

### Explanation

#### When to use each approach

```mermaid
flowchart TD
    A["What is the workload?"] --> B{"Mostly waiting on I/O?"}
    B -- Yes --> C["async/await on the main thread"]
    B -- No --> D{"Is it CPU-heavy JS?"}
    D -- Yes --> E{"Must it finish in the request?"}
    E -- Yes --> F["Worker thread pool"]
    E -- No --> G["Background job queue"]
    D -- No --> H{"Another program or language?"}
    H -- Yes --> I["Child process"]
    H -- No --> J["More capacity?<br/>Cluster or more pods"]
```

| Tool | Parallel CPU? | Memory shared? | Startup cost | Crash isolation | Best for |
|---|---|---|---|---|---|
| Async I/O | No | Yes, same thread | None | None | APIs, DB calls |
| Worker threads | Yes | Optional | Medium | Partial | CPU-heavy JS |
| Child process | Yes | No | High | Strong | Other programs |
| Cluster | Yes | No | High | Strong | Scaling on one machine |
| Job queue | Yes | No | Separate service | Strong | Slow background work |

```javascript
// Same feature, right tool for each step of an image upload
app.post('/photos', async (req, res) => {
  const key = await s3Upload(req);                  // async I/O
  const thumb = await imagePool.run({ key });       // worker thread: quick CPU task
  await videoQueue.add('transcode', { key });       // job queue: slow task
  res.status(202).json({ key, thumb });
});
```

#### Race conditions in Node
The JS thread is single, but any `await` is a point where other requests can run. **Check-then-act** across an `await` isn't atomic.

```javascript
// Race condition: two concurrent requests both see balance = 100
async function withdraw(userId, amount) {
  const { balance } = await db.getAccount(userId);
  if (balance < amount) throw new Error('insufficient funds');
  // Another request can run here
  await db.setBalance(userId, balance - amount);
}

// Fix 1: atomic conditional update in the database
async function withdrawAtomic(userId, amount) {
  const result = await db.query(
    'UPDATE accounts SET balance = balance - $1 WHERE id = $2 AND balance >= $1',
    [amount, userId],
  );
  if (result.rowCount === 0) throw new Error('insufficient funds');
}

// Fix 2: row lock in a transaction: SELECT ... FOR UPDATE (Topic 37)

// Fix 3: optimistic locking with a version column
// UPDATE accounts SET balance = $1, version = version + 1
// WHERE id = $2 AND version = $3

// Fix 4: distributed lock in Redis for work across instances
const token = randomUUID();
const locked = await redis.set(`lock:${userId}`, token, 'NX', 'PX', 5000);
if (!locked) throw new Error('busy');
```

A process-level example, and its fix:

```javascript
// Race: 100 concurrent requests all miss and all call the database
async function getConfigBad() {
  if (!cached) cached = await loadConfig();
  return cached;
}

// Fix: cache the Promise, so all callers share one call
let configPromise;
function getConfig() {
  configPromise ??= loadConfig().catch((err) => {
    configPromise = undefined;     // allow retry after failure
    throw err;
  });
  return configPromise;
}
```

### Real-time scenario
**Problem:** A flash sale oversells stock by 200 units.
**Diagnosis:** The code read stock, checked it, then wrote it back. Concurrent requests read the same value between `await`s. Several instances made a local mutex useless.
**Fix:** Use `UPDATE products SET stock = stock - 1 WHERE id = $1 AND stock > 0`, and check the affected row count.

### Common mistakes and trade-offs
- Saying "Node has no race conditions because it's single-threaded".
- Using an in-process mutex when several instances run.
- Using threads where a queue is better: if the user doesn't need the result now, go async.
- **Trade-off:** database atomic operations are simple and safe, while distributed locks add failure modes such as expiry during work.

### Top interview questions
**Q1. Can Node have race conditions?**
Yes. Anything that crosses an `await` or spans instances can interleave. Shared resources like databases are the usual place.

**Q2. How do you pick between workers, child processes and cluster?**
Workers for CPU-heavy JS, child processes for other programs or isolation, cluster or pods for more request capacity.

**Q3. What is the difference between concurrency and parallelism?**
Concurrency is making progress on many tasks by interleaving. Parallelism is running at the same instant on multiple cores.

**Q4. How do you prevent two requests from double-spending?**
Make the check and write atomic with a conditional update, a row lock or a version column.

**Q5. What is cache stampede in a single process, and how do you avoid it?**
Many callers miss at once and all load the data. Cache the in-flight Promise so they share a single call.

**Q6. Why isn't a local mutex enough in production?**
It protects one process. With several pods, use database locks or a distributed lock.

### Quick revision
I/O → async. CPU JS → workers. Other programs → child process. Capacity → cluster or pods. Slow work → queue. Every `await` is a possible race window.

---

# Part 6: Building APIs and Web Apps

## 26. ⭐ Express

### Why it matters
Express is still the most common Node framework. Interviews test how middleware works, not just how to define a route.

### Explanation

#### Routing
Routes map a method and path to handlers. `Router` groups related routes into modules.

```javascript
const express = require('express');
const app = express();
const orders = express.Router();

app.use(express.json({ limit: '100kb' }));

orders.get('/', listOrders);                  // GET /api/orders
orders.get('/:id', getOrder);                 // req.params.id
orders.post('/', requireAuth, createOrder);   // route-level middleware

app.use('/api/orders', orders);

// Order matters: the first matching route wins
app.get('/users/me', getCurrentUser);         // must come before /:id
app.get('/users/:id', getUser);
```

#### Request lifecycle

```mermaid
flowchart LR
    A["Request"] --> B["Global middleware<br/>logging, JSON, auth"]
    B --> C["Router match"]
    C --> D["Route middleware<br/>validation"]
    D --> E["Handler"]
    E --> F["Response"]
    B -. "next(err)" .-> G["Error middleware"]
    D -. "next(err)" .-> G
    E -. "throw or next(err)" .-> G
    G --> F
```

A request travels through the stack in **registration order**, until something responds.

```javascript
app.use(requestLogger);      // 1. every request
app.use(express.json());     // 2. parse body
app.use(authenticate);       // 3. attach req.user

app.get('/orders', handler); // 4. matched route

app.use((req, res) => {      // 5. nothing matched
  res.status(404).json({ error: 'not found' });
});

app.use((err, req, res, next) => {   // 6. error handler: four arguments
  req.log?.error(err);
  res.status(err.status ?? 500).json({ error: 'internal error' });
});
```

#### How middleware works internally
Express holds a list of layers. `next()` moves to the next matching layer, and `next(err)` skips to error handlers. Here's a small version:

```javascript
function miniExpress() {
  const stack = [];

  function app(req, res) {
    let i = 0;

    function next(err) {
      const layer = stack[i++];
      if (!layer) {
        res.statusCode = err ? 500 : 404;
        return res.end(err ? 'error' : 'not found');
      }

      const isErrorHandler = layer.length === 4;   // detected by parameter count

      try {
        if (err && isErrorHandler) layer(err, req, res, next);
        else if (err) next(err);                   // skip normal middleware
        else if (!isErrorHandler) layer(req, res, next);
        else next();                               // skip error handlers
      } catch (thrown) {
        next(thrown);
      }
    }

    next();
  }

  app.use = (fn) => stack.push(fn);
  return app;
}
```

Middleware can also act **after** the handler, by listening for the response to finish:

```javascript
app.use((req, res, next) => {
  const start = process.hrtime.bigint();
  res.on('finish', () => {
    const ms = Number(process.hrtime.bigint() - start) / 1e6;
    console.log(req.method, req.originalUrl, res.statusCode, ms);
  });
  next();
});
```

**Async errors:** Express 5 forwards rejected promises from handlers to the error middleware. Express 4 doesn't, so you need a wrapper.

```javascript
// Needed in Express 4
const wrap = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get('/orders/:id', wrap(async (req, res) => {
  res.json(await getOrder(req.params.id));
}));
```

### Real-time scenario
**Problem:** Some requests hang until the client times out, and logs show nothing.
**Diagnosis:** A middleware had a branch that neither called `next()` nor sent a response.
**Fix:** Make every path end with `next()` or a response, and add a server `requestTimeout` and a request timing log.

### Common mistakes and trade-offs
- Putting error middleware before routes, or defining it with three parameters.
- Sending a response twice, which raises `ERR_HTTP_HEADERS_SENT`. Use `return res.json()`.
- Running heavy middleware globally when only a few routes need it.
- **Trade-off:** Express is minimal and well known, but offers little structure and has lower raw throughput than Fastify.

### Top interview questions
**Q1. What is middleware?**
A function with `(req, res, next)` that runs in the request pipeline. It can change the request, respond, or pass control on.

**Q2. How does Express tell error middleware apart?**
By parameter count. A function with four parameters is treated as an error handler.

**Q3. What happens if you don't call `next()` or respond?**
The request hangs until something times out.

**Q4. How are async errors handled?**
Express 5 forwards rejected promises automatically. Express 4 needs a wrapper that passes them to `next`.

**Q5. What is the difference between `app.use` and `app.get`?**
`app.use` matches a path prefix for any method. `app.get` matches the exact path for GET.

**Q6. What causes `ERR_HTTP_HEADERS_SENT`?**
Writing a response after one was sent, usually from a missing `return`.

### Quick revision
A stack of layers in order. `next()` continues, `next(err)` jumps to four-argument handlers. Order matters. Express 5 catches async errors.

---

## 27. Framework Comparison

### Why it matters
"Why this framework?" is a common architecture question. Good answers compare structure, speed and team fit, rather than popularity.

### Explanation

#### Express
Minimal and unopinionated, with the largest ecosystem.

```javascript
const express = require('express');
const app = express();
app.get('/users/:id', (req, res) => res.json({ id: req.params.id }));
app.listen(3000);
```

#### Fastify
Built for speed. Schema-based validation and serialization, plus a plugin system with encapsulation. Pino logging is built in.

```javascript
const fastify = require('fastify')({ logger: true });

fastify.get('/users/:id', {
  schema: {
    params: {
      type: 'object',
      properties: { id: { type: 'integer' } },
      required: ['id'],
    },
    response: {
      200: {
        type: 'object',
        properties: { id: { type: 'integer' }, name: { type: 'string' } },
      },
    },
  },
}, async (request) => ({ id: request.params.id, name: 'Asha', password: 'not sent' }));
// The response schema speeds up JSON output and removes the password field.

fastify.listen({ port: 3000 });
```

#### NestJS
An opinionated framework with modules, dependency injection and decorators. It runs on Express or Fastify underneath.

```typescript
@Injectable()
export class UsersService {
  constructor(private readonly repo: UsersRepository) {}
  findOne(id: number) { return this.repo.findById(id); }
}

@Controller('users')
export class UsersController {
  constructor(private readonly users: UsersService) {}

  @Get(':id')
  @UseGuards(AuthGuard)
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.users.findOne(id);
  }
}

@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
})
export class UsersModule {}
```

#### Koa
Made by the original Express team. It's tiny and async-first. Its "onion" middleware model can run code before and after `await next()`.

```javascript
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx, next) => {
  const start = Date.now();
  await next();                                   // run everything downstream
  ctx.set('x-response-time', `${Date.now() - start}ms`);
});

app.use(async (ctx) => {
  ctx.body = { ok: true };
});

app.listen(3000);
```

| | Express | Fastify | NestJS | Koa |
|---|---|---|---|---|
| Style | Minimal | Schema and plugins | Opinionated, DI | Minimal, async |
| Speed | Good | Very good | Depends on adapter | Good |
| TypeScript | Added on | Good | First-class | Added on |
| Learning curve | Low | Medium | Higher | Low |
| Best fit | Small to mid APIs | High-throughput APIs | Large teams, enterprise | Custom stacks |

### Real-time scenario
**Problem:** A company's 12 Express services all have different folder layouts, so moving between teams is slow.
**Diagnosis:** Express imposes no structure, and each team invented its own.
**Fix:** Adopt NestJS for new services, or create a shared internal template. Keep stable services unchanged, because a rewrite adds risk with no user benefit.

### Common mistakes and trade-offs
- Choosing by benchmark. In most APIs, the database dominates latency.
- Using NestJS for a tiny service, where the structure costs more than it returns.
- **Trade-off:** more framework gives consistency and less freedom. Less framework gives flexibility and more decisions.

### Top interview questions
**Q1. Why is Fastify faster than Express?**
It uses schema-compiled serialization, an efficient router and low-overhead hooks.

**Q2. When would you choose NestJS?**
For large teams and complex domains, where DI, modules and conventions pay for themselves.

**Q3. Is NestJS a replacement for Express?**
No. It's a layer above an HTTP adapter, which is Express by default, with Fastify available.

**Q4. How does Koa's middleware differ from Express?**
Koa's is promise-based. `await next()` returns, so code can run on the way back up.

**Q5. Does framework speed matter?**
Rarely as much as queries and downstream calls. It matters when a service is very high-throughput and CPU-bound on overhead.

### Quick revision
Express = minimal and popular. Fastify = fast and schema-based. NestJS = structure and DI. Koa = small onion model.

---

## 28. ⭐ REST API Design

### Why it matters
Most senior roles include API design. Pagination and idempotency questions show whether you've run APIs at scale.

### Explanation

#### Resource design
Use nouns for resources and HTTP methods for actions.

```text
GET    /orders              list
POST   /orders              create
GET    /orders/42           read
PATCH  /orders/42           partial update
PUT    /orders/42           full replace
DELETE /orders/42           delete
POST   /orders/42/cancel    action that doesn't map to CRUD
GET    /users/7/orders      related resource
```

#### Versioning

| Strategy | Example | Pros | Cons |
|---|---|---|---|
| URL path | `/v1/orders` | Clear, easy to route | Multiple URLs per resource |
| Header | `Accept: application/vnd.api.v2+json` | Clean URLs | Harder to test |
| Query | `/orders?version=2` | Simple | Easy to forget |

```javascript
app.use('/v1/orders', ordersV1Router);
app.use('/v2/orders', ordersV2Router);

// Tell clients about the end of a version
app.use('/v1', (req, res, next) => {
  res.set('Deprecation', 'true');
  res.set('Sunset', 'Wed, 30 Jun 2027 00:00:00 GMT');
  next();
});
```

Avoid new versions when you can: adding an optional field isn't a breaking change.

#### Pagination
**Offset** pagination is simple but slow on deep pages, and it can skip or repeat rows when data changes. **Cursor** pagination stays fast and stable.

```javascript
// Offset: OFFSET 100000 still scans 100,000 rows
app.get('/orders', async (req, res) => {
  const limit = Math.min(Number(req.query.limit ?? 20), 100);   // always cap it
  const offset = Number(req.query.offset ?? 0);
  const rows = await db.query(
    'SELECT * FROM orders ORDER BY id DESC LIMIT $1 OFFSET $2',
    [limit, offset],
  );
  res.json({ data: rows });
});

// Cursor: uses the index to jump straight to the position
app.get('/feed', async (req, res) => {
  const limit = Math.min(Number(req.query.limit ?? 20), 100);
  const cursor = req.query.cursor
    ? Number(Buffer.from(req.query.cursor, 'base64url').toString())
    : Number.MAX_SAFE_INTEGER;

  const rows = await db.query(
    'SELECT * FROM orders WHERE id < $1 ORDER BY id DESC LIMIT $2',
    [cursor, limit + 1],                    // one extra row tells us if more exist
  );

  const hasMore = rows.length > limit;
  const page = rows.slice(0, limit);
  const nextCursor = hasMore
    ? Buffer.from(String(page.at(-1).id)).toString('base64url')
    : null;

  res.json({ data: page, nextCursor });
});
```

#### Idempotency
An operation is idempotent if repeating it has the same effect as doing it once. `GET`, `PUT` and `DELETE` should be. `POST` isn't, so a retry on a timeout can charge a card twice. Use an **idempotency key**.

```javascript
app.post('/payments', async (req, res) => {
  const key = req.get('Idempotency-Key');
  if (!key) return res.status(400).json({ error: 'Idempotency-Key required' });

  // Atomically claim the key
  const claimed = await redis.set(`idem:${key}`, 'processing', 'NX', 'EX', 86400);

  if (!claimed) {
    const saved = await redis.get(`idem:${key}`);
    if (saved === 'processing') {
      return res.status(409).json({ error: 'already in progress' });
    }
    return res.status(200).json(JSON.parse(saved));   // replay the first result
  }

  try {
    const payment = await charge(req.body);
    await redis.set(`idem:${key}`, JSON.stringify(payment), 'EX', 86400);
    res.status(201).json(payment);
  } catch (err) {
    await redis.del(`idem:${key}`);       // allow a retry after failure
    throw err;
  }
});
```

For strong guarantees, also use a database unique constraint on the key.

#### Status codes

| Code | Meaning | Use |
|---|---|---|
| 200 | OK | Read or update succeeded |
| 201 | Created | Add a `Location` header |
| 202 | Accepted | Async job started |
| 204 | No content | Delete succeeded |
| 400 | Bad request | Malformed request |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated, but not allowed |
| 404 | Not found | Doesn't exist |
| 409 | Conflict | State conflict, duplicate |
| 422 | Unprocessable | Valid syntax, invalid data |
| 429 | Too many requests | Rate limited, add `Retry-After` |
| 500 | Server error | Bug or unhandled failure |
| 502/503/504 | Upstream issues | Bad gateway, unavailable, timeout |

```javascript
// One consistent error format across the API
res.status(422).json({
  error: {
    code: 'VALIDATION_FAILED',
    message: 'Email is invalid',
    details: [{ field: 'email', issue: 'format' }],
    requestId: req.id,
  },
});
```

### Real-time scenario
**Problem:** A customer is charged twice after a mobile network drop.
**Diagnosis:** The server processed the payment, the response was lost, and the app retried the `POST`.
**Fix:** The client generates one idempotency key per payment attempt, and the server stores and replays the first result.

### Common mistakes and trade-offs
- Using verbs in URLs, such as `/getOrders`.
- Returning 200 with an error in the body.
- Having no cap on page size.
- Confusing 401 and 403.
- **Trade-off:** offset pagination allows "jump to page 50", while cursors scale better but only move forward and back.

### Top interview questions
**Q1. What makes an API RESTful?**
Resources, standard HTTP methods, stateless requests, proper status codes and cacheable responses.

**Q2. What is the difference between `PUT` and `PATCH`?**
`PUT` replaces the whole resource and is idempotent. `PATCH` changes part of it.

**Q3. What is idempotency, and why does it matter?**
Repeating the request has the same effect as doing it once. Retries are only safe with it.

**Q4. Offset or cursor pagination?**
Offset for small data and page numbers. Cursor for large or changing data, such as feeds.

**Q5. How would you version an API?**
URL versioning is the most common. Only version for breaking changes, and give a deprecation period.

**Q6. What is the difference between 401 and 403?**
401 means the identity isn't established. 403 means it's known but lacks permission.

**Q7. How do you handle long-running requests?**
Return 202 with a job ID. Clients poll a status URL or receive a webhook.

### Quick revision
Nouns + methods. Version on breaking changes only. Cursors for scale. Idempotency keys make retries safe. Use the right status codes.

---

## 29. GraphQL

### Why it matters
GraphQL is common in product companies and BFF layers. Expect questions on N+1, DataLoader and when REST is better.

### Explanation

#### Schema and resolvers
The schema defines what clients can ask for. A **resolver** is a function that returns the data for one field.

```javascript
const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    orders: [Order!]!
  }

  type Order {
    id: ID!
    total: Float!
  }

  type Query {
    user(id: ID!): User
  }

  type Mutation {
    createOrder(userId: ID!, total: Float!): Order!
  }
`;

const resolvers = {
  Query: {
    user: (parent, args, context) => context.db.users.findById(args.id),
  },
  User: {
    // Runs only if the client asks for `orders`
    orders: (user, args, context) => context.db.orders.findByUser(user.id),
  },
  Mutation: {
    createOrder: (_, { userId, total }, context) => {
      if (!context.user) throw new Error('unauthenticated');
      return context.db.orders.create({ userId, total });
    },
  },
};
```

```graphql
# The client asks for exactly what it needs, in one request
query {
  user(id: "1") {
    name
    orders { total }
  }
}
```

#### N+1 problem
Field resolvers run per item, so a list of 100 users triggers 100 more queries.

```javascript
// Query: users(limit: 100) { orders { total } }
const resolvers = {
  Query: { users: () => db.query('SELECT * FROM users LIMIT 100') },  // 1 query
  User: {
    orders: (user) => db.query('SELECT * FROM orders WHERE user_id = $1', [user.id]), // 100 queries
  },
};
```

#### DataLoader
DataLoader collects all `load()` calls made in the same tick, makes **one** batched call, and caches results within a request.

```javascript
const DataLoader = require('dataloader');

function createLoaders(db) {
  return {
    ordersByUser: new DataLoader(async (userIds) => {
      const rows = await db.query(
        'SELECT * FROM orders WHERE user_id = ANY($1)',
        [userIds],
      );

      // Must return results in the same order as the keys
      const grouped = new Map(userIds.map((id) => [id, []]));
      for (const row of rows) grouped.get(row.user_id).push(row);
      return userIds.map((id) => grouped.get(id));
    }),
  };
}

// New loaders for each request, so cached data can't leak between users
const context = ({ req }) => ({ user: req.user, loaders: createLoaders(db) });

const resolvers = {
  User: {
    orders: (user, args, ctx) => ctx.loaders.ordersByUser.load(user.id),
  },
};
// 101 queries become 2.
```

#### GraphQL vs REST

| | REST | GraphQL |
|---|---|---|
| Endpoints | Many | Usually one |
| Data shape | Server decides | Client decides |
| Over-fetching | Common | Avoided |
| HTTP caching | Easy with GET | Harder, needs client caching |
| Errors | HTTP status codes | Often 200 with `errors` |
| Rate limiting | Per endpoint | Needs query cost analysis |

Public GraphQL APIs must limit depth and complexity, because a single nested query can be very expensive.

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(6)],
});
// Stronger protection: allow only pre-registered (persisted) queries.
```

### Real-time scenario
**Problem:** A mobile home screen makes 7 REST calls and loads slowly on 4G.
**Diagnosis:** Round trips and over-fetching wasted time and bandwidth.
**Fix:** Add a GraphQL BFF so one query fetches exactly what's needed. Use DataLoader to prevent N+1.

### Common mistakes and trade-offs
- Sharing a DataLoader across requests, which can leak data between users.
- Not limiting depth, which allows denial of service.
- Only checking auth at the top. Protect sensitive fields too.
- **Trade-off:** clients get flexibility, but the server takes on complexity in caching, security and monitoring.

### Top interview questions
**Q1. What is a resolver?**
A function that returns a field's data. It receives `parent`, `args`, `context` and `info`.

**Q2. What is the N+1 problem in GraphQL?**
A list query followed by one query per item through a field resolver.

**Q3. How does DataLoader work?**
It batches keys requested in the same tick into one call, and caches them per request.

**Q4. When is REST better than GraphQL?**
Simple CRUD, public APIs that need HTTP caching, file downloads, and teams that don't need flexible queries.

**Q5. How do you secure GraphQL?**
Depth and cost limits, persisted queries, field-level auth, and turning off introspection in public production APIs.

**Q6. Why are GraphQL errors different?**
Partial data is possible, so responses often return 200 with an `errors` array. Monitoring must inspect the body.

### Quick revision
One schema, field resolvers, client picks fields. DataLoader fixes N+1. Limit depth and cost. Create loaders per request.

---

## 30. gRPC

### Why it matters
gRPC is common for internal service calls. It shows up in microservices and performance discussions.

### Explanation

#### Protobuf
Protocol Buffers define the contract in a `.proto` file and encode messages in compact binary. Field numbers, not names, go on the wire, so never reuse them.

```protobuf
syntax = "proto3";

package inventory;

service Inventory {
  rpc GetItem (ItemRequest) returns (Item);
  rpc WatchStock (ItemRequest) returns (stream StockEvent);
}

message ItemRequest {
  string sku = 1;
}

message Item {
  string sku = 1;
  int32 quantity = 2;
}

message StockEvent {
  string sku = 1;
  int32 quantity = 2;
}
```

```javascript
// Server
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const definition = protoLoader.loadSync('inventory.proto');
const { inventory } = grpc.loadPackageDefinition(definition);

const server = new grpc.Server();

server.addService(inventory.Inventory.service, {
  getItem(call, callback) {
    const item = stock.get(call.request.sku);
    if (!item) {
      return callback({ code: grpc.status.NOT_FOUND, message: 'unknown sku' });
    }
    callback(null, item);
  },
});

server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {});
```

```javascript
// Client, with a deadline on every call
const client = new inventory.Inventory('inventory:50051', grpc.credentials.createInsecure());

client.getItem(
  { sku: 'A1' },
  { deadline: Date.now() + 500 },
  (err, item) => {
    if (err) return console.error(err.code, err.message);
    console.log(item.quantity);
  },
);
```

#### Streaming
gRPC runs on HTTP/2 and supports four call types.

| Type | Client sends | Server sends | Example |
|---|---|---|---|
| Unary | One | One | Get item |
| Server streaming | One | Many | Live prices |
| Client streaming | Many | One | Upload metrics |
| Bidirectional | Many | Many | Chat |

```javascript
// Server streaming: push updates to the client
const handlers = {
  watchStock(call) {
    const onChange = (event) => {
      if (event.sku === call.request.sku) call.write(event);
    };
    stockEvents.on('change', onChange);

    call.on('cancelled', () => stockEvents.off('change', onChange));
  },
};

// Client
const stream = client.watchStock({ sku: 'A1' });
stream.on('data', (event) => console.log(event.quantity));
stream.on('error', (err) => console.error(err.details));
```

#### gRPC vs REST

| | REST | gRPC |
|---|---|---|
| Format | JSON, readable | Protobuf, binary |
| Transport | HTTP/1.1 or 2 | HTTP/2 |
| Contract | Optional (OpenAPI) | Required, generated code |
| Streaming | Limited | Built in |
| Browser support | Native | Needs gRPC-Web or a proxy |
| Debugging | `curl`, browser | Needs tools such as `grpcurl` |
| Best for | Public APIs | Internal services |

### Real-time scenario
**Problem:** The checkout flow makes 6 internal JSON calls, and serialization shows up in CPU profiles.
**Diagnosis:** Large JSON payloads and connection setup were adding latency across every hop.
**Fix:** Move the busiest internal calls to gRPC on persistent connections, with deadlines. Keep REST at the public edge.

### Common mistakes and trade-offs
- Calling without a deadline. Requests can then hang indefinitely.
- Reusing or changing field numbers, which breaks compatibility.
- L4 load balancers keep long-lived HTTP/2 connections on one pod. Use L7 or client-side balancing.
- **Trade-off:** faster, strongly typed contracts, in exchange for harder debugging and shared schema tooling.

### Top interview questions
**Q1. What is gRPC?**
An RPC framework using HTTP/2 and Protobuf, with generated clients and streaming.

**Q2. Why is gRPC faster than JSON over REST?**
Compact binary encoding, HTTP/2 multiplexing and persistent connections.

**Q3. What are the four call types?**
Unary, server streaming, client streaming and bidirectional streaming.

**Q4. How do you evolve Protobuf schemas safely?**
Add new fields with new numbers, never reuse numbers, and mark removed ones as `reserved`.

**Q5. When would you avoid gRPC?**
For public browser APIs, simple CRUD, or teams that value easy debugging over speed.

**Q6. Why is load balancing gRPC difficult?**
HTTP/2 connections are long-lived, so connection-level balancing pins traffic to one pod. Balance per request with an L7 proxy or service mesh.

### Quick revision
Protobuf + HTTP/2. Strong contracts, streaming, always use deadlines. Great internally, REST at the edge. Needs L7 balancing.

---

## 31. ⭐ Real-Time Communication

### Why it matters
Chat, live scores and notifications are common design questions. You need to choose the right transport and explain how it scales past one server.

### Explanation

#### WebSockets
One persistent, two-way TCP connection, upgraded from HTTP.

```javascript
const { WebSocketServer } = require('ws');
const http = require('node:http');

const server = http.createServer();
const wss = new WebSocketServer({ server });

wss.on('connection', (ws, req) => {
  ws.isAlive = true;
  ws.on('pong', () => { ws.isAlive = true; });

  ws.on('message', (raw) => {
    const msg = JSON.parse(raw);
    for (const client of wss.clients) {
      if (client.readyState === client.OPEN) client.send(JSON.stringify(msg));
    }
  });
});

// Heartbeat: remove dead connections
setInterval(() => {
  for (const ws of wss.clients) {
    if (!ws.isAlive) { ws.terminate(); continue; }
    ws.isAlive = false;
    ws.ping();
  }
}, 30_000);

server.listen(8080);
```

Authenticate during the upgrade, because browsers can't set custom headers on WebSockets. Use a cookie or a short-lived ticket.

```javascript
server.on('upgrade', (req, socket, head) => {
  const user = verifyTicket(new URL(req.url, 'http://x').searchParams.get('ticket'));
  if (!user) {
    socket.destroy();
    return;
  }
  wss.handleUpgrade(req, socket, head, (ws) => {
    ws.user = user;
    wss.emit('connection', ws, req);
  });
});
```

#### Socket.IO
A library on top of WebSockets. It adds reconnection, rooms, acknowledgements and fallback to HTTP polling. It uses its own protocol, so a plain WebSocket client can't connect.

```javascript
const { Server } = require('socket.io');
const io = new Server(httpServer, { cors: { origin: 'https://app.example.com' } });

io.on('connection', (socket) => {
  socket.on('join', (roomId) => socket.join(roomId));

  socket.on('chat', (msg, ack) => {
    io.to(msg.roomId).emit('chat', msg);
    ack({ delivered: true });       // the client knows it arrived
  });
});
```

#### SSE
Server-Sent Events are one-way, server to client, over plain HTTP. Browsers reconnect automatically. They're great for live updates, and common for LLM token streaming.

```javascript
app.get('/events', (req, res) => {
  res.set({
    'content-type': 'text/event-stream',
    'cache-control': 'no-cache',
    connection: 'keep-alive',
  });
  res.flushHeaders();

  const send = (data) => res.write(`data: ${JSON.stringify(data)}\n\n`);

  bus.on('price', send);
  const ping = setInterval(() => res.write(': ping\n\n'), 25_000);

  req.on('close', () => {
    bus.off('price', send);
    clearInterval(ping);
  });
});

// Browser
// const source = new EventSource('/events');
// source.onmessage = (e) => console.log(JSON.parse(e.data));
```

#### Long polling
The client sends a request, and the server holds it until there's data or a timeout. It works everywhere, but with more overhead.

```javascript
app.get('/poll', async (req, res) => {
  const ac = new AbortController();
  req.on('close', () => ac.abort());

  try {
    const [message] = await once(bus, 'message', {
      signal: AbortSignal.any([ac.signal, AbortSignal.timeout(25_000)]),
    });
    res.json({ message });
  } catch {
    if (!res.destroyed) res.status(204).end();    // client polls again
  }
});
```

| | WebSocket | SSE | Long polling |
|---|---|---|---|
| Direction | Two-way | Server to client | Simulated |
| Protocol | WS | HTTP | HTTP |
| Reconnect | You build it | Automatic | Each request |
| Binary | Yes | No | Yes |
| Best for | Chat, games | Feeds, AI tokens | Legacy fallback |

#### Scaling connections
Connections stay on one server, so users on different servers can't reach each other directly. Use a **pub/sub backplane**.

```mermaid
flowchart LR
    U1["User A"] --> S1["Server 1"]
    U2["User B"] --> S2["Server 2"]
    S1 <--> R["Redis Pub/Sub"]
    S2 <--> R
```

```javascript
// Socket.IO with the Redis adapter
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const pub = createClient({ url: process.env.REDIS_URL });
const sub = pub.duplicate();
await Promise.all([pub.connect(), sub.connect()]);

io.adapter(createAdapter(pub, sub));
// io.to('room').emit() now reaches sockets on every server.
```

Other concerns:
- Socket.IO needs **sticky sessions** while polling is in use.
- Deploys drop connections, so clients must reconnect with **jittered backoff**.
- Slow clients need backpressure: check `ws.bufferedAmount`, then drop or disconnect.
- Raise file descriptor limits and load balancer idle timeouts.

### Real-time scenario
**Problem:** After scaling to 4 servers, users only receive about 25% of chat messages.
**Diagnosis:** Each server broadcast only to its local clients.
**Fix:** Add the Redis adapter so messages cross servers. Use a durable store as well, because Redis Pub/Sub drops messages when users are offline.

### Common mistakes and trade-offs
- Using WebSockets when SSE would do.
- Having no heartbeat, so dead connections stay around.
- Trusting messages after connect. Validate and authorize each one.
- **Trade-off:** persistent connections give low latency, but cost memory per connection and complicate deployment and load balancing.

### Top interview questions
**Q1. WebSocket or SSE: how do you choose?**
WebSocket for two-way, frequent messages. SSE for one-way updates. SSE is simpler and works with standard HTTP.

**Q2. How do you scale WebSockets across servers?**
A pub/sub backplane such as Redis, NATS or Kafka, plus sticky routing if needed and client reconnection.

**Q3. What is the difference between Socket.IO and WebSocket?**
WebSocket is a protocol. Socket.IO is a library with rooms, acks, reconnection and fallback, and its own protocol.

**Q4. How do you authenticate WebSockets?**
During the upgrade with a cookie or short-lived token. Avoid long-lived tokens in URLs, because they end up in logs.

**Q5. What happens during a deploy?**
Connections drop. Drain gradually, and make clients reconnect with jitter to avoid a thundering herd.

**Q6. How do you detect dead connections?**
Heartbeats: send a ping regularly and terminate connections with no pong.

**Q7. How would you stream LLM output to a browser?**
SSE usually fits: one-way, plain HTTP and simple reconnection (Topic 69).

### Quick revision
WebSocket = two-way. SSE = server push. Long polling = fallback. Scale with Redis Pub/Sub. Heartbeats, auth at upgrade, reconnect with jitter.

---

## 32. File Uploads and Large Payloads

### Why it matters
Uploads combine streams, memory, security and cloud storage. A naive version can take down a server.

### Explanation

#### Multipart
`multipart/form-data` sends files and fields separated by boundaries. `multer` and `busboy` are the usual tools. Memory storage is only safe for small files.

```javascript
const multer = require('multer');
const crypto = require('node:crypto');
const path = require('node:path');

const upload = multer({
  storage: multer.diskStorage({
    destination: '/tmp/uploads',
    filename: (req, file, cb) => {
      cb(null, crypto.randomUUID() + path.extname(file.originalname));
    },
  }),
  limits: { fileSize: 10 * 1024 * 1024, files: 1 },
  fileFilter: (req, file, cb) => {
    cb(null, ['image/png', 'image/jpeg'].includes(file.mimetype));
  },
});

app.post('/avatar', upload.single('avatar'), (req, res) => {
  res.json({ path: req.file.path });
});

// file.mimetype comes from the client, so verify the real content
const { fileTypeFromFile } = await import('file-type');
const type = await fileTypeFromFile(req.file.path);
```

#### Streaming
Stream directly to storage, so the file never sits in memory or on local disk.

```javascript
const Busboy = require('busboy');
const { Upload } = require('@aws-sdk/lib-storage');
const { randomUUID } = require('node:crypto');

app.post('/upload', (req, res, next) => {
  const busboy = Busboy({ headers: req.headers, limits: { fileSize: 2 * 1024 ** 3 } });

  busboy.on('file', (field, file, info) => {
    const key = `uploads/${randomUUID()}`;

    const s3Upload = new Upload({
      client: s3,
      params: { Bucket: 'media', Key: key, Body: file, ContentType: info.mimeType },
    });

    s3Upload.done().then(() => res.json({ key })).catch(next);
    file.on('limit', () => s3Upload.abort());
  });

  req.pipe(busboy);
});
```

#### S3 presigned URLs
The best option for large files: the client uploads straight to S3, so your API never handles the bytes.

```mermaid
sequenceDiagram
    participant C as Client
    participant A as API
    participant S as S3
    C->>A: Request upload URL
    A->>C: Presigned URL (5 min expiry)
    C->>S: PUT file directly
    S-->>A: Event: object created
    A->>A: Scan and process
```

```javascript
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const { randomUUID } = require('node:crypto');

const s3 = new S3Client({});

app.post('/uploads/url', requireAuth, async (req, res) => {
  const { contentType } = req.body;
  if (!['image/png', 'image/jpeg'].includes(contentType)) {
    return res.status(400).json({ error: 'unsupported type' });
  }

  const key = `users/${req.user.id}/${randomUUID()}`;
  const url = await getSignedUrl(
    s3,
    new PutObjectCommand({ Bucket: 'media', Key: key, ContentType: contentType }),
    { expiresIn: 300 },
  );

  res.json({ url, key });
});
```

For very large files, use **multipart upload** so clients can resume in parts. Enforce size limits with presigned POST policies.

**Large JSON payloads:**

```javascript
// Cap body size to prevent memory abuse
app.use(express.json({ limit: '1mb' }));

// Allow more only where it's needed
app.post('/import', express.json({ limit: '20mb' }), importHandler);
```

### Real-time scenario
**Problem:** Video uploads cause API pods to OOM and block other traffic.
**Diagnosis:** Multer memory storage buffered entire files in RAM, and uploads held API connections open for minutes.
**Fix:** Use presigned multipart uploads to S3. The API just issues URLs and handles an S3 event to start processing.

### Common mistakes and trade-offs
- Trusting `Content-Type` or file names from the client.
- Saving uploads under the web root, which allows script execution.
- Setting no size limits.
- **Trade-off:** presigned URLs remove load from your servers, but validation has to happen after upload.

### Top interview questions
**Q1. How would you design large file uploads?**
Presigned multipart S3 uploads, directly from the client. Then process through an event and an async pipeline.

**Q2. Why not use Multer memory storage for large files?**
Each file sits in RAM, so a few concurrent uploads can exhaust memory.

**Q3. How do you validate uploaded files?**
Limit size, check real content via magic bytes, generate file names, scan for malware and store outside the web root.

**Q4. What is a presigned URL?**
A time-limited signed URL that lets a client perform one storage action without AWS credentials.

**Q5. How do you support resumable uploads?**
Split into parts with S3 multipart upload or the `tus` protocol. Track completed parts, and retry only failed ones.

**Q6. What happens if a user uploads and never completes the process?**
Use a temporary prefix with a lifecycle rule to remove orphaned objects and incomplete multipart uploads.

### Quick revision
Small: Multer to disk. Big: stream or, better, presigned S3. Always limit size, verify content and generate file names.

---

## 33. Validation and Config

### Why it matters
Every outside input is untrusted, and TypeScript types vanish at runtime. Validating at the edge prevents many bugs and attacks.

### Explanation

#### Zod
A TypeScript-first schema library. One schema gives runtime validation and a static type.

```typescript
import { z } from 'zod';

const CreateOrder = z.object({
  customerEmail: z.string().email(),
  items: z.array(z.object({
    sku: z.string().min(1),
    quantity: z.number().int().positive().max(100),
  })).min(1),
  couponCode: z.string().optional(),
});

type CreateOrder = z.infer<typeof CreateOrder>;  // the type comes for free

// Reusable middleware
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({ errors: result.error.issues });
  }
  req.body = result.data;    // unknown fields are stripped by default
  next();
};

app.post('/orders', validate(CreateOrder), createOrder);
```

Query strings are always strings, so coerce them explicitly:

```typescript
const ListQuery = z.object({
  limit: z.coerce.number().int().min(1).max(100).default(20),
  active: z.enum(['true', 'false']).transform((v) => v === 'true').optional(),
});
```

#### Joi
A mature, JS-first schema library with a similar purpose.

```javascript
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(18),
  password: Joi.string().min(12).required(),
  confirm: Joi.ref('password'),
});

const { error, value } = schema.validate(req.body, {
  abortEarly: false,      // collect all errors
  stripUnknown: true,     // remove extra fields
});

if (error) {
  return res.status(422).json({ errors: error.details.map((d) => d.message) });
}
```

| | Zod | Joi |
|---|---|---|
| TypeScript types | Inferred | Manual |
| Style | Chained, small | Chained, feature-rich |
| Best fit | TS projects | JS projects, Hapi |

Fastify users often use JSON Schema, with Ajv underneath, since it's compiled for speed.

#### Env configuration
Parse and validate config once at boot, and export a typed object. Fail fast if anything is wrong.

```typescript
// config.ts
import { z } from 'zod';

const Env = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

const parsed = Env.safeParse(process.env);

if (!parsed.success) {
  console.error('Invalid config:', parsed.error.flatten().fieldErrors);
  process.exit(1);
}

export const config = Object.freeze(parsed.data);
```

Commit a `.env.example`, keep real `.env` files out of git, and use a secrets manager in production.

### Real-time scenario
**Problem:** Users gain admin rights by adding `"role": "admin"` to their profile update.
**Diagnosis:** The handler passed `req.body` straight into an update, which is a **mass assignment** bug.
**Fix:** Validate with a strict schema that only allows editable fields, and update from the parsed result.

### Common mistakes and trade-offs
- Trusting TypeScript types for network input.
- Validating deep inside services rather than at the edge.
- Returning raw validation errors that reveal internal details.
- **Trade-off:** strict validation can break older clients. Reject unknown fields on sensitive routes, and strip them elsewhere.

### Top interview questions
**Q1. Why do you need runtime validation with TypeScript?**
Types are removed at compile time. Requests, env vars and API responses are unknown until checked.

**Q2. Where should validation happen?**
At the boundary, before business logic. Business rules still belong in the service layer.

**Q3. What is mass assignment?**
Passing user input directly to a data update, which lets users set fields they shouldn't.

**Q4. What is the difference between `parse` and `safeParse`?**
`parse` throws on failure. `safeParse` returns a success or failure result.

**Q5. How do you manage config across environments?**
One validated config module fed by env vars, with secrets from a manager. Fail at startup if anything's invalid.

### Quick revision
Validate at the boundary. Zod gives types and checks. Parse config once at boot. Never pass raw `req.body` to the database.

---

## 34. TypeScript with Node.js

### Why it matters
Most new backend Node services use TypeScript. Interviews ask about setup, safe API typing and how TS runs in production.

### Explanation

#### Project setup

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "sourceMap": true
  },
  "include": ["src"]
}
```

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "typecheck": "tsc --noEmit",
    "build": "tsc",
    "start": "node --enable-source-maps dist/index.js"
  }
}
```

With `NodeNext`, relative ESM imports need the output extension: `import { x } from './utils.js'`, even when the source is `utils.ts`.

#### Typing APIs

```typescript
import type { Request, Response, NextFunction } from 'express';

// Generics: Params, ResBody, ReqBody, Query
interface OrderParams { id: string }
interface OrderDto { id: string; total: number }

async function getOrder(
  req: Request<OrderParams>,
  res: Response<OrderDto | { error: string }>,
) {
  const order = await repo.find(req.params.id);
  if (!order) return res.status(404).json({ error: 'not found' });
  res.json(order);
}

// Add custom fields to Request
declare global {
  namespace Express {
    interface Request { user?: { id: string; role: 'admin' | 'user' } }
  }
}

// Model states with discriminated unions, so bad states can't compile
type PaymentResult =
  | { status: 'paid'; receiptId: string }
  | { status: 'failed'; reason: string };

function handle(result: PaymentResult) {
  if (result.status === 'paid') return result.receiptId;
  return result.reason;
}

// Branded types stop you passing the wrong ID
type UserId = string & { readonly brand: 'UserId' };
type OrderId = string & { readonly brand: 'OrderId' };

// `unknown` forces you to validate, while `any` turns checks off
function parseBody(input: unknown) {
  return CreateOrder.parse(input);
}
```

#### Native type stripping
Recent Node versions can run `.ts` files directly by removing type syntax. Node **doesn't type-check**. You still run `tsc` in CI.

```bash
node src/index.ts          # works on recent Node versions
tsc --noEmit               # type checking stays a separate step
```

```typescript
// Runs natively: types are simply erased
function greet(name: string): string {
  return `Hello ${name}`;
}

// Doesn't run with stripping alone: enums and parameter properties generate code.
// enum Status { Active }
// class A { constructor(private x: number) {} }
// Use `erasableSyntaxOnly` in tsconfig to catch these early.
```

| Option | Type-checks? | Use |
|---|---|---|
| `tsc` | Yes | CI and production builds |
| `tsx` | No | Fast development |
| Native Node | No | Scripts, simple apps |
| esbuild or SWC | No | Fast bundling |

### Real-time scenario
**Problem:** Production stack traces point to `dist/index.js:1:58231`, which is useless.
**Diagnosis:** Bundled output had no source mapping at runtime.
**Fix:** Emit source maps and start with `--enable-source-maps`, so errors map back to `.ts` files.

### Common mistakes and trade-offs
- Using `any` or `as` to silence errors.
- Treating `req.body as Order` as validation.
- Shipping `ts-node` to production instead of compiled code.
- **Trade-off:** more safety and better refactoring in exchange for a build step and config.

### Top interview questions
**Q1. How do you run TypeScript in production?**
Usually compile with `tsc` or a bundler and run JS. Recent Node can strip types natively, but type checking stays in CI.

**Q2. What is the difference between `any` and `unknown`?**
`any` disables checking. `unknown` forces you to narrow the type before use.

**Q3. Does Node check types when it runs a `.ts` file?**
No. It only strips type syntax.

**Q4. Why doesn't TypeScript protect against bad API input?**
Types are erased at runtime. Use a validator like Zod at the boundary.

**Q5. How do you add `user` to Express's `Request`?**
Use declaration merging in a global `Express.Request` interface.

**Q6. What is a discriminated union?**
A union with a shared literal field, such as `status`. TypeScript narrows the type when you check that field.

### Quick revision
`strict` on. `tsc` in CI. `unknown` beats `any`. Types vanish at runtime. Use source maps. Native Node strips types but doesn't check them.

---

# Part 7: Data Layer

## 35. ⭐ Database Access

### Why it matters
Most Node services are thin layers over a database. Your choices here have more effect on latency and correctness than the framework does.

### Explanation

#### SQL vs NoSQL from Node

| | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB, DynamoDB) |
|---|---|---|
| Schema | Enforced | Flexible |
| Relationships | Joins | Embed or duplicate |
| Transactions | Strong | Varies, often limited |
| Scaling writes | Harder | Often built in |
| Best for | Orders, payments | Catalogs, events, high-scale key access |

```javascript
// PostgreSQL with pg: always use parameters
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

const { rows } = await pool.query(
  'SELECT id, email FROM users WHERE id = $1',
  [userId],
);

// MongoDB with the official driver
const { MongoClient } = require('mongodb');
const client = new MongoClient(process.env.MONGO_URL);
const users = client.db('app').collection('users');

const user = await users.findOne({ email });
await users.updateOne({ _id: user._id }, { $inc: { logins: 1 } });

// DynamoDB: design around access patterns
const { DynamoDBDocumentClient, QueryCommand } = require('@aws-sdk/lib-dynamodb');
const result = await ddb.send(new QueryCommand({
  TableName: 'app',
  KeyConditionExpression: 'pk = :pk',
  ExpressionAttributeValues: { ':pk': `USER#${userId}` },
}));
```

#### ORM vs query builder vs raw SQL

| Option | What it does | Pros | Cons |
|---|---|---|---|
| Raw SQL | You write everything | Full control | Manual mapping |
| Query builder | Builds SQL in code | Flexible, close to SQL | Little type safety on its own |
| ORM | Maps rows to objects | Fast to build with | Hidden queries, N+1 risks |

#### Prisma
A schema file generates a fully typed client. Excellent developer experience.

```prisma
model User {
  id     Int     @id @default(autoincrement())
  email  String  @unique
  orders Order[]
}

model Order {
  id     Int  @id @default(autoincrement())
  total  Int
  userId Int
  user   User @relation(fields: [userId], references: [id])
}
```

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

const user = await prisma.user.findUnique({
  where: { email },
  include: { orders: { take: 5, orderBy: { id: 'desc' } } },
});

const raw = await prisma.$queryRaw`SELECT count(*) FROM "User"`;
```

#### TypeORM
Decorator-based entities. It's well known in NestJS projects, and supports Active Record and Data Mapper styles.

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn() id: number;
  @Column({ unique: true }) email: string;
  @OneToMany(() => Order, (order) => order.user) orders: Order[];
}

const users = await dataSource.getRepository(User).find({
  where: { email },
  relations: { orders: true },
});
```

#### Drizzle
A lightweight, SQL-like TypeScript ORM. It's a popular choice for those who want types and to stay close to SQL, including in serverless apps.

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { eq } from 'drizzle-orm';

const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
});

const orders = pgTable('orders', {
  id: serial('id').primaryKey(),
  userId: integer('user_id').notNull(),
  total: integer('total').notNull(),
});

const rows = await db
  .select({ email: users.email, total: orders.total })
  .from(users)
  .innerJoin(orders, eq(orders.userId, users.id))
  .where(eq(users.id, 1));
```

#### Knex
A proven query builder with migrations, and no object mapping.

```javascript
const knex = require('knex')({ client: 'pg', connection: process.env.DATABASE_URL });

const orders = await knex('orders')
  .join('users', 'users.id', 'orders.user_id')
  .where('users.id', userId)
  .select('orders.id', 'orders.total')
  .orderBy('orders.id', 'desc')
  .limit(20);
```

### Real-time scenario
**Problem:** An orders page takes 4 s. The ORM code looks simple.
**Diagnosis:** Query logs show `include` loading 5 joined relations and every column, returning 40 MB per page.
**Fix:** Select only the needed columns, paginate, add an index, and use raw SQL for the one heavy report.

### Common mistakes and trade-offs
- Concatenating SQL strings, which allows SQL injection.
- Not logging or inspecting the SQL your ORM generates.
- Using `SELECT *` in hot paths.
- **Trade-off:** ORMs speed up CRUD work, while complex reports are better as SQL. Mixing the two is normal.

### Top interview questions
**Q1. ORM or query builder?**
ORM for fast CRUD with stable models. Query builder or SQL when performance and query control matter.

**Q2. How do you prevent SQL injection in Node?**
Use parameterized queries. Don't concatenate input, and allow-list dynamic column names.

**Q3. SQL or NoSQL for an orders system?**
SQL, for transactions, constraints and relations. Use NoSQL for specific high-scale access patterns.

**Q4. What are the weaknesses of an ORM?**
Hidden queries, N+1 risk, over-fetching and awkward complex queries.

**Q5. Why might you choose Prisma or Drizzle?**
Prisma for a schema-first workflow and polished tooling. Drizzle for a lightweight SQL-like API.

**Q6. How do you find slow queries?**
Slow query logs, `EXPLAIN ANALYZE`, `pg_stat_statements` and tracing spans around database calls.

### Quick revision
SQL for consistency, NoSQL for scale patterns. Always parameterize. Check generated SQL. ORMs for CRUD, SQL for heavy reports.

---

## 36. ⭐ Connection Pooling

### Why it matters
Connections are expensive and limited. Pool problems often cause outages that look like "the database is slow".

### Explanation

#### What pooling does
A pool keeps open connections and lends them to requests. It avoids the setup cost of TCP, TLS and authentication on every query.

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,                          // connections in this process
  idleTimeoutMillis: 30_000,        // close idle connections
  connectionTimeoutMillis: 2_000,   // fail fast when the pool is exhausted
  statement_timeout: 5_000,         // stop runaway queries
});

// Idle clients can fail. Without this listener, the error crashes the process.
pool.on('error', (err) => logger.error({ err }, 'idle client error'));

// pool.query handles checkout and release for you
const { rows } = await pool.query('SELECT now()');
```

#### Pool sizing
The important figure is the **total** across all instances:

```text
total connections = pods × pool max (× workers per pod)

20 pods × pool of 20 = 400 connections
PostgreSQL max_connections = 200
Result: "too many connections" during deploys or autoscaling
```

Bigger isn't better. A database has limited cores, and too many active queries compete for them. A commonly quoted starting point from the HikariCP guide:

```text
connections ≈ (core_count × 2) + effective_spindle_count
```

Share that total across instances, keep room for rolling deploys (old and new pods overlap), and then load test.

For many short-lived clients, such as serverless, put **PgBouncer** or **RDS Proxy** in front.

```text
Lambda × 1000 → RDS Proxy / PgBouncer → 50 real connections → PostgreSQL
```

#### Leaks
A leak is a checked-out connection that's never released. Always release in `finally`.

```javascript
// Leak: if query throws, the client is never returned
async function bad(id) {
  const client = await pool.connect();
  const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  client.release();
  return result.rows;
}

// Correct
async function good(id) {
  const client = await pool.connect();
  try {
    const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows;
  } finally {
    client.release();
  }
}

// Better: a helper makes it impossible to forget
async function withTransaction(fn) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await fn(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

Don't hold a connection while waiting on something else:

```javascript
await withTransaction(async (tx) => {
  await tx.query('UPDATE orders SET status = $1 WHERE id = $2', ['paid', id]);
  // Bad: holds a connection and row locks during a 2 s HTTP call
  await paymentGateway.charge(order);
});
```

#### Exhaustion
When all connections are in use, new requests queue up. Latency climbs while the database itself looks idle.

```javascript
// Export these as metrics
setInterval(() => {
  metrics.gauge('db.pool.total', pool.totalCount);
  metrics.gauge('db.pool.idle', pool.idleCount);
  metrics.gauge('db.pool.waiting', pool.waitingCount);   // key alert: above 0 for long
}, 5000).unref();

// Fail fast, instead of queueing forever
app.use((err, req, res, next) => {
  if (err.message?.includes('timeout exceeded when trying to connect')) {
    return res.status(503).set('Retry-After', '1').json({ error: 'busy' });
  }
  next(err);
});
```

### Real-time scenario
**Problem:** APIs hang after a new feature is released. Database CPU is at 10%.
**Diagnosis:** `pool.waitingCount` is high. One code path returned early without releasing its client, so the pool gradually drained.
**Fix:** Use the `withTransaction` helper, add a `connectionTimeoutMillis` value, and alert on waiting clients.

### Common mistakes and trade-offs
- Sizing the pool without counting pods.
- Creating a `new Pool()` for each request.
- Keeping transactions open across network calls.
- **Trade-off:** larger pools absorb bursts but increase database contention. Smaller pools protect the database but queue requests.

### Top interview questions
**Q1. Why use a connection pool?**
Setup is costly and databases allow limited connections. Pools reuse them and cap concurrency.

**Q2. How do you size a pool?**
Start from database capacity, divide among all instances, leave deploy headroom, and validate with load tests.

**Q3. How do you detect a connection leak?**
Rising in-use count, waiting clients with an idle DB, and pool timeout errors. Find missing `release` calls.

**Q4. What happens when the pool is exhausted?**
Requests queue until timeout. Fail fast with 503, rather than letting them build up.

**Q5. How do you use databases from serverless?**
A connection proxy such as RDS Proxy or PgBouncer, or a database with an HTTP driver. Create clients outside the handler.

**Q6. Why not hold a transaction open during an HTTP call?**
It occupies a connection and locks for the duration, which reduces throughput and invites deadlocks.

### Quick revision
Total = pods × pool size. Release in `finally`. Monitor waiting clients. Use a proxy for serverless. Keep transactions short.

---

## 37. Transactions and Migrations

### Why it matters
Correctness under concurrency and zero-downtime schema changes are how senior engineers show they've operated real systems.

### Explanation

#### Transactions and isolation
A transaction makes a group of changes all-or-nothing (ACID). Isolation levels control what concurrent transactions can see.

| Level | Prevents | Still allows |
|---|---|---|
| Read Committed | Dirty reads | Non-repeatable reads, phantoms |
| Repeatable Read | Non-repeatable reads | Phantoms in standard SQL (PostgreSQL's snapshot also blocks them) |
| Serializable | All anomalies | Transactions can fail and must retry |

```javascript
// Row lock: other transactions wait for this row
async function transfer(from, to, amount) {
  return withTransaction(async (tx) => {
    // Lock in a fixed order so two transfers can't deadlock
    const ids = [from, to].sort();
    const { rows } = await tx.query(
      'SELECT id, balance FROM accounts WHERE id = ANY($1) ORDER BY id FOR UPDATE',
      [ids],
    );

    const source = rows.find((r) => r.id === from);
    if (source.balance < amount) throw new Error('insufficient funds');

    await tx.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [amount, from]);
    await tx.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [amount, to]);
  });
}

// Serializable transactions need a retry on conflict
async function serializable(fn, attempts = 3) {
  for (let i = 1; ; i++) {
    try {
      return await withTransaction(async (tx) => {
        await tx.query('SET TRANSACTION ISOLATION LEVEL SERIALIZABLE');
        return fn(tx);
      });
    } catch (err) {
      if (err.code !== '40001' || i >= attempts) throw err;   // serialization failure
    }
  }
}
```

**Optimistic vs pessimistic:** `FOR UPDATE` is pessimistic and suits high contention. A `version` column is optimistic and suits low contention.

#### Zero-downtime migrations
During a rolling deploy, old and new code run together, so each schema change must work with both. Use **expand and contract**.

```text
Rename users.name to users.full_name:

1. Expand     Add full_name (nullable). Deploy code writing to both columns.
2. Backfill   Copy old rows in small batches.
3. Switch     Deploy code reading full_name.
4. Contract   Stop writing name, then drop the column in a later release.
```

```javascript
// Backfill in batches to avoid long locks
async function backfill() {
  let updated;
  do {
    const result = await pool.query(`
      UPDATE users SET full_name = name
      WHERE id IN (
        SELECT id FROM users WHERE full_name IS NULL LIMIT 1000
      )
    `);
    updated = result.rowCount;
    await sleep(100);       // let other work through
  } while (updated > 0);
}
```

```sql
-- Doesn't block writes in PostgreSQL
CREATE INDEX CONCURRENTLY idx_orders_user ON orders (user_id);

-- Stops a migration from waiting forever behind a lock
SET lock_timeout = '3s';
```

Run migrations as a separate deployment step, not at each pod's startup. Otherwise, many pods race to apply them.

#### N+1 queries
One query fetches a list, then one more runs per row.

```javascript
// N+1: 1 + 100 queries
const orders = await db.query('SELECT * FROM orders LIMIT 100');
for (const order of orders) {
  order.user = await db.query('SELECT * FROM users WHERE id = $1', [order.user_id]);
}

// Fix 1: a join, in one query
await db.query(`
  SELECT o.*, u.email
  FROM orders o JOIN users u ON u.id = o.user_id
  LIMIT 100
`);

// Fix 2: batch load, in two queries
const ids = [...new Set(orders.map((o) => o.user_id))];
const users = await db.query('SELECT * FROM users WHERE id = ANY($1)', [ids]);
const byId = new Map(users.map((u) => [u.id, u]));
orders.forEach((o) => { o.user = byId.get(o.user_id); });

// Fix 3: ORM eager loading
await prisma.order.findMany({ take: 100, include: { user: true } });
```

### Real-time scenario
**Problem:** A deploy that adds a `NOT NULL` column with an index takes the site down for 8 minutes.
**Diagnosis:** The migration held a heavy lock on a 200 million-row table, and every query queued behind it.
**Fix:** Add a nullable column, backfill in batches, use `CREATE INDEX CONCURRENTLY`, set `lock_timeout`, and add the constraint in a safe way at the end.

### Common mistakes and trade-offs
- Rolling back code but not a destructive schema change.
- Locking rows in inconsistent order, which causes deadlocks.
- Long transactions, which hold locks and delay vacuum.
- **Trade-off:** stricter isolation reduces anomalies but increases retries and lock waits.

### Top interview questions
**Q1. What does ACID mean?**
Atomicity, Consistency, Isolation, Durability: all-or-nothing, valid states, controlled concurrency, and surviving crashes.

**Q2. What is the default isolation level in PostgreSQL?**
Read Committed. MySQL's InnoDB defaults to Repeatable Read.

**Q3. What is expand and contract?**
A staged migration in which old and new code both work: add, backfill, switch, then remove.

**Q4. Optimistic or pessimistic locking?**
Optimistic when conflicts are rare. Pessimistic when contention is high or retries are costly.

**Q5. How do you avoid deadlocks?**
Lock in a consistent order, keep transactions short, and retry deadlock errors.

**Q6. How do you find and fix N+1 queries?**
Spot many similar queries in logs or traces. Fix with joins, batch loading or eager loading.

### Quick revision
Short transactions, consistent lock order. Expand → backfill → switch → contract. Joins or batches fix N+1.

---

## 38. ⭐ Caching

### Why it matters
Caching is the fastest way to improve latency and protect a database, and it's a source of tricky bugs. "Cache invalidation" questions test your depth.

### Explanation

#### Cache layers

```text
Browser → CDN → API in-memory → Redis → Database
Fastest, least shared                 Slowest, source of truth
```

#### Redis
An in-memory store with TTLs and rich data types. It's shared, so all instances see the same data.

```javascript
const { createClient } = require('redis');
const redis = createClient({ url: process.env.REDIS_URL });
redis.on('error', (err) => logger.error(err));   // required
await redis.connect();

await redis.set('user:1', JSON.stringify(user), { EX: 300 });  // string + TTL
await redis.incr('page:views');                                // atomic counter
await redis.hSet('cart:1', 'sku-a', '2');                      // hash
await redis.zAdd('leaderboard', { score: 100, value: 'asha' }); // sorted set
await redis.sAdd('online', 'user-1');                          // set
```

#### Cache-aside
The application checks the cache, loads on a miss, then fills the cache. It's the most common pattern.

```javascript
async function getProduct(id) {
  const key = `product:${id}`;

  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const product = await db.products.findById(id);
  if (product) {
    await redis.set(key, JSON.stringify(product), { EX: 300 });
  }
  return product;
}

// Redis failure shouldn't take the app down
async function safeGet(key) {
  try {
    return await redis.get(key);
  } catch (err) {
    logger.warn({ err }, 'cache unavailable');
    return null;       // fall through to the database
  }
}
```

| Pattern | How it works | Good for |
|---|---|---|
| Cache-aside | App reads and fills | Most read-heavy data |
| Read-through | Cache loads data itself | Simpler app code |
| Write-through | Write cache and DB together | Reads right after writes |
| Write-behind | Write cache, DB later | Write-heavy, some loss acceptable |

#### Invalidation
Stale data is the main risk. Options include TTL, deleting on write, and versioned keys.

```javascript
// Delete on write: update the database first, then delete the key
async function updateProduct(id, changes) {
  const product = await db.products.update(id, changes);
  await redis.del(`product:${id}`);
  return product;
}

// Versioned keys: invalidate a whole group with one change
async function catalogKey(category) {
  const version = (await redis.get(`catalog:version:${category}`)) ?? '1';
  return `catalog:${category}:v${version}`;
}
async function invalidateCatalog(category) {
  await redis.incr(`catalog:version:${category}`);
}

// Jitter on TTLs, so keys don't all expire at once
const ttl = 300 + Math.floor(Math.random() * 60);
```

Delete rather than update the cached value. Two concurrent updates can write stale values in the wrong order. With several instances that hold in-process caches, broadcast invalidations with Pub/Sub.

#### Cache stampede
When a hot key expires, many requests miss together and all hit the database.

```javascript
// Fix 1: share in-flight loads within one process
const inFlight = new Map();

function loadOnce(key, loader) {
  if (!inFlight.has(key)) {
    inFlight.set(key, loader().finally(() => inFlight.delete(key)));
  }
  return inFlight.get(key);
}

// Fix 2: a Redis lock so only one instance recomputes
async function getWithLock(key, loader) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const lock = await redis.set(`lock:${key}`, '1', { NX: true, EX: 10 });
  if (lock) {
    try {
      const value = await loader();
      await redis.set(key, JSON.stringify(value), { EX: 300 });
      return value;
    } finally {
      await redis.del(`lock:${key}`);
    }
  }

  await sleep(50 + Math.random() * 100);     // others wait, then retry
  return getWithLock(key, loader);
}

// Fix 3: stale-while-revalidate: serve old data while one refresh runs
async function getStale(key, loader) {
  const entry = JSON.parse((await redis.get(key)) ?? 'null');

  if (entry && Date.now() < entry.freshUntil) return entry.value;

  if (entry) {
    refreshInBackground(key, loader);        // serve stale immediately
    return entry.value;
  }
  return refresh(key, loader);
}
```

Related problems: **cache penetration** (repeated lookups of missing keys: cache a short-lived null, or use a Bloom filter) and a **hot key** (one key overloads a node: add a tiny local cache).

### Real-time scenario
**Problem:** The homepage goes down every hour, on the hour.
**Diagnosis:** The cached homepage had a fixed 1-hour TTL. On expiry, thousands of concurrent misses ran the heaviest query in the system.
**Fix:** Add a Redis lock, stale-while-revalidate and TTL jitter. Refresh hot keys before they expire.

### Common mistakes and trade-offs
- Caching with no TTL.
- Leaving personalized data under a shared key, which leaks data between users.
- Treating Redis as permanent storage without a persistence plan.
- **Trade-off:** caching improves speed and reduces load, in exchange for staleness and more failure modes.

### Top interview questions
**Q1. What is cache-aside?**
The app reads the cache and, on a miss, loads the database and fills the cache. Writes invalidate the entry.

**Q2. What is a cache stampede, and how do you prevent it?**
Many concurrent misses on one key. Prevent it with locks, request coalescing, stale-while-revalidate and TTL jitter.

**Q3. How do you handle invalidation?**
TTLs as a safety net, delete on write, versioned keys for groups, and events for multi-service updates.

**Q4. Local memory or Redis?**
Local is fastest but per-instance. Redis is shared and survives app restarts. Many systems use both.

**Q5. What happens if Redis goes down?**
Treat it as optional. Use short timeouts, fall through to the database, and protect the database with rate limits or a circuit breaker.

**Q6. Why delete rather than update the cache on write?**
Concurrent updates can write stale values in the wrong order. Deleting lets the next read load fresh data.

**Q7. What are Redis eviction policies?**
Rules for a full memory limit, such as `allkeys-lru`, `allkeys-lfu`, `volatile-ttl` or `noeviction`.

### Quick revision
Cache-aside + TTL + delete on write. Stampede: lock, coalesce, jitter, serve stale. Redis failures shouldn't take the app down.

---

## 39. Background Jobs

### Why it matters
Slow work like emails, reports and media processing shouldn't run in requests. Jobs need retries, scheduling and failure handling.

### Explanation

#### BullMQ
A Redis-based job queue. Producers add jobs, and workers process them with concurrency limits.

```javascript
const { Queue, Worker } = require('bullmq');

const connection = { host: 'localhost', port: 6379 };
const emailQueue = new Queue('email', { connection });

// Producer: the API returns right away
app.post('/signup', async (req, res) => {
  const user = await createUser(req.body);
  await emailQueue.add('welcome', { userId: user.id });
  res.status(201).json(user);
});

// Consumer: usually a separate process
const worker = new Worker('email', async (job) => {
  await job.updateProgress(10);
  await sendWelcomeEmail(job.data.userId);
}, { connection, concurrency: 10 });

worker.on('failed', (job, err) => logger.error({ jobId: job?.id, err }));

process.on('SIGTERM', async () => {
  await worker.close();      // finish active jobs first
  process.exit(0);
});
```

#### Cron
Scheduled recurring jobs. With several instances, an in-process cron runs on **every** one.

```javascript
const cron = require('node-cron');

// Bug with 5 pods: this runs 5 times per day
cron.schedule('0 2 * * *', sendDailyReport);

// Fix 1: BullMQ repeatable job, created once across all instances
await reportQueue.upsertJobScheduler(
  'daily-report',
  { pattern: '0 2 * * *', tz: 'Asia/Kolkata' },
  { name: 'report' },
);

// Fix 2: take a lock so one instance wins
cron.schedule('0 2 * * *', async () => {
  const today = new Date().toISOString().slice(0, 10);
  const locked = await redis.set(`cron:report:${today}`, process.pid, { NX: true, EX: 3600 });
  if (locked) await sendDailyReport();
});

// Fix 3: Kubernetes CronJob or EventBridge Scheduler
```

```text
┌ minute (0-59)
│ ┌ hour (0-23)
│ │ ┌ day of month
│ │ │ ┌ month
│ │ │ │ ┌ day of week
* * * * *

*/5 * * * *     every 5 minutes
0 2 * * 1       2 AM every Monday
```

#### Retries
Use limited attempts with backoff. Jobs can run more than once, so handlers must be idempotent.

```javascript
await paymentQueue.add('capture', { orderId }, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 1000 },
  jobId: `capture-${orderId}`,      // same ID means no duplicate job
  removeOnComplete: { age: 86400 },
  removeOnFail: false,
});

const { UnrecoverableError } = require('bullmq');

new Worker('payment', async (job) => {
  const order = await orders.find(job.data.orderId);
  if (!order) throw new UnrecoverableError('order missing');   // don't retry
  if (order.status === 'captured') return;                     // idempotent
  await gateway.capture(order, { idempotencyKey: job.id });
}, { connection });
```

#### Dead-letter queues
Jobs that fail all attempts go to a **DLQ**, so they aren't lost and don't block others. BullMQ keeps them in the failed set, and you can move them to a separate queue.

```javascript
const deadLetters = new Queue('payment-dlq', { connection });

worker.on('failed', async (job, err) => {
  if (job && job.attemptsMade >= job.opts.attempts) {
    await deadLetters.add('dead', {
      original: job.data,
      reason: err.message,
      failedAt: new Date().toISOString(),
    });
    alerts.notify(`job ${job.id} moved to DLQ`);
  }
});

// After a fix, replay
async function replay() {
  for (const job of await deadLetters.getJobs(['waiting'])) {
    await paymentQueue.add('capture', job.data.original);
    await job.remove();
  }
}
```

### Real-time scenario
**Problem:** Customers receive 4 copies of a monthly invoice.
**Diagnosis:** `node-cron` ran in each of the 4 API pods.
**Fix:** Move the schedule to a single BullMQ job scheduler, with a unique job ID per customer and month.

### Common mistakes and trade-offs
- Running workers in the API process, so jobs compete with requests.
- Non-idempotent handlers. At-least-once delivery means repeats happen.
- Putting large payloads in Redis. Pass IDs instead.
- **Trade-off:** jobs improve response time and reliability, but require queue monitoring and eventual-consistency UX.

### Top interview questions
**Q1. When should work move to a background job?**
When it's slow, unreliable or doesn't need to finish before the response.

**Q2. How do you run cron safely across many instances?**
Use a single scheduler, a repeatable queue job, or a distributed lock.

**Q3. What is a dead-letter queue?**
A holding area for messages that keep failing. It allows inspection and replay.

**Q4. Why must job handlers be idempotent?**
Retries and redelivery can run a job twice, and side effects shouldn't repeat.

**Q5. How do you shut down a worker gracefully?**
Stop taking new jobs, finish active ones, then exit. Set a grace period longer than typical job time.

**Q6. What should a job payload contain?**
IDs and small metadata. Load fresh data in the worker.

### Quick revision
BullMQ on Redis. Separate worker processes. Attempts + exponential backoff. Idempotent handlers. DLQ for failures. One scheduler, not one per pod.

---

## 40. ⭐ Message Queues

### Why it matters
Queues decouple services and absorb spikes. "Kafka vs RabbitMQ" and "exactly-once" are standard senior questions.

### Explanation

#### Kafka vs RabbitMQ vs SQS

| | Kafka | RabbitMQ | SQS |
|---|---|---|---|
| Model | Distributed log | Broker with routing | Managed queue |
| Messages after reading | Kept for retention period | Removed after ack | Removed after delete |
| Replay | Yes | No | No |
| Ordering | Per partition | Per queue (with limits) | FIFO queues only |
| Throughput | Very high | High | High, scales automatically |
| Routing | By topic and key | Rich exchanges | Basic |
| Operations | Heavy | Medium | None |
| Best for | Event streams, analytics | Task routing | AWS workloads, simple queues |

```javascript
// Kafka: the key decides the partition, so one order stays in order
const { Kafka } = require('kafkajs');
const kafka = new Kafka({ brokers: ['kafka:9092'] });

const producer = kafka.producer({ idempotent: true });
await producer.connect();
await producer.send({
  topic: 'orders',
  messages: [{ key: order.id, value: JSON.stringify(order) }],
});

// Consumers in one group split partitions between them
const consumer = kafka.consumer({ groupId: 'billing' });
await consumer.connect();
await consumer.subscribe({ topic: 'orders' });
await consumer.run({
  eachMessage: async ({ message }) => {
    await bill(JSON.parse(message.value.toString()));
  },
});
```

```javascript
// RabbitMQ: exchange routes to queues, the consumer acks
const amqp = require('amqplib');
const conn = await amqp.connect(process.env.AMQP_URL);
const channel = await conn.createChannel();

await channel.assertExchange('orders', 'topic', { durable: true });
await channel.assertQueue('email', { durable: true });
await channel.bindQueue('email', 'orders', 'order.*');

channel.publish('orders', 'order.created', Buffer.from(JSON.stringify(order)), {
  persistent: true,
});

await channel.prefetch(10);
channel.consume('email', async (msg) => {
  try {
    await sendEmail(JSON.parse(msg.content));
    channel.ack(msg);
  } catch {
    channel.nack(msg, false, false);   // send to the dead-letter exchange
  }
});
```

```javascript
// SQS: poll, process, delete
const { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } = require('@aws-sdk/client-sqs');
const sqs = new SQSClient({});

while (running) {
  const { Messages = [] } = await sqs.send(new ReceiveMessageCommand({
    QueueUrl: url,
    MaxNumberOfMessages: 10,
    WaitTimeSeconds: 20,        // long polling
    VisibilityTimeout: 60,      // must exceed processing time
  }));

  for (const m of Messages) {
    await handle(JSON.parse(m.Body));
    await sqs.send(new DeleteMessageCommand({ QueueUrl: url, ReceiptHandle: m.ReceiptHandle }));
  }
}
```

#### Delivery guarantees

| Guarantee | Meaning | How | Risk |
|---|---|---|---|
| At-most-once | 0 or 1 time | Ack before processing | Message loss |
| At-least-once | 1 or more times | Ack after processing | Duplicates |
| Exactly-once | Effect happens once | Idempotency, or transactions in one system | Complex |

```javascript
// At-most-once: a crash after ack loses the message
channel.ack(msg);
await handleMessage(msg);

// At-least-once: a crash before ack redelivers it
await handleMessage(msg);
channel.ack(msg);
```

End-to-end **exactly-once delivery** across systems isn't practical. In practice, you combine **at-least-once delivery** with **idempotent processing**.

#### Idempotent consumers
Record processed message IDs, in the **same transaction** as the business change.

```javascript
async function handleOrderPaid(event) {
  await withTransaction(async (tx) => {
    const inserted = await tx.query(
      `INSERT INTO processed_events (event_id) VALUES ($1)
       ON CONFLICT (event_id) DO NOTHING`,
      [event.id],
    );

    if (inserted.rowCount === 0) return;    // already handled

    await tx.query(
      'UPDATE orders SET status = $1 WHERE id = $2',
      ['paid', event.orderId],
    );
  });
}
```

Natural idempotency also helps: `SET status = 'paid'` is safe to repeat, while `balance = balance + 10` isn't.

### Real-time scenario
**Problem:** During a Kafka rebalance, some customers receive duplicate loyalty points.
**Diagnosis:** Messages were processed, but offsets weren't committed before the rebalance, so they were redelivered.
**Fix:** Duplicates are expected with at-least-once. Add a unique event ID and a `processed_events` table in the same transaction.

### Common mistakes and trade-offs
- Calling a broker "exactly-once" without considering your side effects.
- SQS visibility timeouts shorter than processing time, which cause repeats.
- A poor Kafka partition key creates hot partitions, and more consumers than partitions leaves some idle.
- **Trade-off:** queues give decoupling and buffering, in exchange for eventual consistency and harder debugging.

### Top interview questions
**Q1. Kafka or RabbitMQ?**
Kafka for high-throughput event streams and replay. RabbitMQ for task queues with flexible routing.

**Q2. How does Kafka guarantee ordering?**
Only within a partition. Use a key such as order ID, so related messages land in the same partition.

**Q3. What are the delivery guarantees?**
At-most-once, at-least-once and effectively exactly-once. At-least-once with idempotent handling is the usual choice.

**Q4. How do you make a consumer idempotent?**
Store a unique event ID in the same transaction as the update, and skip duplicates.

**Q5. What is a consumer group?**
Consumers sharing a group ID split a topic's partitions. Different groups each receive all messages.

**Q6. What is SQS visibility timeout?**
How long a received message stays hidden. If it isn't deleted in time, it reappears.

**Q7. How do you handle poison messages?**
Limit retries, then send to a DLQ with error context, and alert on DLQ depth.

### Quick revision
Kafka = log and replay. RabbitMQ = routing. SQS = managed. At-least-once + idempotent consumers. Order only inside a partition.

---

# Part 8: Security

## 41. ⭐ Authentication

### Why it matters
Authentication answers "who are you?". The usual interview question is sessions vs JWT, followed by how you revoke a token.

### Explanation

#### Sessions
The server stores login state, and the browser holds only a random session ID in a cookie. Logging out is simple: delete the record.

```javascript
const session = require('express-session');
const { RedisStore } = require('connect-redis');

app.use(session({
  store: new RedisStore({ client: redis }),
  name: 'sid',
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,            // JavaScript can't read it (limits XSS theft)
    secure: true,              // HTTPS only
    sameSite: 'lax',           // helps against CSRF
    maxAge: 30 * 60 * 1000,
  },
}));

app.post('/login', async (req, res, next) => {
  const user = await verifyCredentials(req.body);
  if (!user) return res.status(401).json({ error: 'invalid credentials' });

  // New session ID after login prevents session fixation
  req.session.regenerate((err) => {
    if (err) return next(err);
    req.session.userId = user.id;
    res.json({ ok: true });
  });
});

app.post('/logout', (req, res) => {
  req.session.destroy(() => res.status(204).end());
});
```

#### JWT
A signed token that holds claims. Servers verify it without a database lookup. It's **signed, not encrypted**, so anyone can read the payload.

```text
header.payload.signature
{"alg":"RS256"}.{"sub":"42","exp":1760000000}.<signature>
```

```javascript
const jwt = require('jsonwebtoken');

function issueAccessToken(user) {
  return jwt.sign(
    { sub: user.id, role: user.role },
    process.env.JWT_PRIVATE_KEY,
    { algorithm: 'RS256', expiresIn: '15m', issuer: 'auth.example.com', audience: 'api' },
  );
}

function requireAuth(req, res, next) {
  const token = req.headers.authorization?.replace(/^Bearer /, '');
  if (!token) return res.status(401).json({ error: 'missing token' });

  try {
    req.user = jwt.verify(token, process.env.JWT_PUBLIC_KEY, {
      algorithms: ['RS256'],        // never trust the token's own alg header
      issuer: 'auth.example.com',
      audience: 'api',
    });
    next();
  } catch {
    res.status(401).json({ error: 'invalid token' });
  }
}
```

| | Sessions | JWT |
|---|---|---|
| State | Server-side | Inside the token |
| Revocation | Instant | Hard before expiry |
| Lookup per request | Yes | No |
| Best for | Browser apps | Service-to-service, mobile, distributed APIs |

#### Refresh tokens
Access tokens are short-lived. A long-lived **refresh token** gets new ones. Rotate on every use, and treat reuse as theft.

```javascript
const { randomBytes, createHash, randomUUID } = require('node:crypto');
const sha256 = (v) => createHash('sha256').update(v).digest('hex');

async function issueRefreshToken(userId, familyId = randomUUID()) {
  const token = randomBytes(32).toString('hex');
  await db.refreshTokens.insert({
    hash: sha256(token),                 // store only a hash
    userId,
    familyId,
    expiresAt: new Date(Date.now() + 30 * 86400_000),
  });
  return token;
}

app.post('/token/refresh', async (req, res) => {
  const presented = req.cookies.refresh;
  const record = await db.refreshTokens.findByHash(sha256(presented ?? ''));

  if (!record) return res.status(401).end();

  if (record.usedAt) {
    // Reused token: assume theft and revoke the entire family
    await db.refreshTokens.revokeFamily(record.familyId);
    return res.status(401).end();
  }

  await db.refreshTokens.markUsed(record.id);
  const refresh = await issueRefreshToken(record.userId, record.familyId);

  res.cookie('refresh', refresh, {
    httpOnly: true, secure: true, sameSite: 'strict', path: '/token',
  });
  res.json({ accessToken: issueAccessToken(await users.find(record.userId)) });
});
```

For instant revocation of an access token, keep a short denylist of `jti` values in Redis.

#### OAuth2
OAuth2 is for **delegated authorization**: letting an app access a resource for a user, without seeing their password. For web and mobile apps, use **Authorization Code + PKCE**.

```mermaid
sequenceDiagram
    participant U as User
    participant A as Your app
    participant P as Provider
    A->>U: Redirect to provider with state and code_challenge
    U->>P: Log in and approve
    P->>A: Redirect back with code and state
    A->>P: Exchange code with code_verifier
    P->>A: Access token
```

```javascript
const { randomBytes, createHash } = require('node:crypto');

app.get('/login/provider', (req, res) => {
  const verifier = randomBytes(32).toString('base64url');
  const state = randomBytes(16).toString('hex');
  req.session.oauth = { verifier, state };

  const url = new URL('https://provider.example.com/authorize');
  url.search = new URLSearchParams({
    response_type: 'code',
    client_id: process.env.CLIENT_ID,
    redirect_uri: 'https://app.example.com/callback',
    scope: 'openid email',
    state,
    code_challenge: createHash('sha256').update(verifier).digest('base64url'),
    code_challenge_method: 'S256',
  });
  res.redirect(url.toString());
});

app.get('/callback', async (req, res) => {
  const { verifier, state } = req.session.oauth ?? {};
  if (!state || req.query.state !== state) return res.status(400).end();  // CSRF check

  const response = await fetch('https://provider.example.com/token', {
    method: 'POST',
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code: req.query.code,
      redirect_uri: 'https://app.example.com/callback',
      client_id: process.env.CLIENT_ID,
      code_verifier: verifier,
    }),
  });
  const tokens = await response.json();
  // Next: validate the ID token and create your own session
});
```

#### OIDC
OpenID Connect adds **authentication** on top of OAuth2. It introduces an **ID token**, a JWT describing the user. "Log in with Google" is OIDC.

```javascript
const { createRemoteJWKSet, jwtVerify } = require('jose');

const jwks = createRemoteJWKSet(new URL('https://accounts.google.com/.well-known/jwks.json'));

async function verifyIdToken(idToken) {
  const { payload } = await jwtVerify(idToken, jwks, {
    issuer: 'https://accounts.google.com',
    audience: process.env.CLIENT_ID,
  });
  return { id: payload.sub, email: payload.email };
}
```

### Real-time scenario
**Problem:** A fired employee's stolen token keeps working for 7 days.
**Diagnosis:** Access JWTs lasted 7 days, and nothing could revoke them.
**Fix:** Use 15-minute access tokens and rotating refresh tokens, plus a denylist and a per-user token version check for emergencies.

### Common mistakes and trade-offs
- Storing tokens in `localStorage`, where XSS can read them. Prefer `HttpOnly` cookies.
- Putting secrets in JWT payloads.
- Accepting any `alg`, or using weak HMAC secrets.
- **Trade-off:** JWTs avoid lookups but make revocation hard. Sessions need a store but offer full control.

### Top interview questions
**Q1. Sessions or JWT?**
Sessions for first-party browser apps, with easy revocation. JWTs for distributed services and non-browser clients.

**Q2. How do you revoke a JWT?**
Short expiry, refresh token revocation, plus a denylist or token version check where immediate effect is needed.

**Q3. What is the difference between OAuth2 and OIDC?**
OAuth2 is delegated authorization. OIDC adds authentication and an ID token.

**Q4. What is PKCE?**
A one-time verifier and challenge that protects the code exchange, so an intercepted code is useless.

**Q5. Where should tokens be stored in a browser?**
`HttpOnly`, `Secure`, `SameSite` cookies are the safer choice. `localStorage` is exposed to XSS.

**Q6. What is refresh token rotation?**
Each use returns a new token and invalidates the old one. Reuse signals theft, so revoke the family.

**Q7. HS256 or RS256?**
HS256 shares one secret. RS256 signs with a private key, so services verify with a public key and can't forge tokens.

### Quick revision
Sessions are stateful and revocable. JWTs are stateless and signed, not encrypted. Short access + rotating refresh. OAuth2 = access, OIDC = login. Use PKCE.

---

## 42. Authorization

### Why it matters
Authorization answers "what can you do?". **Broken access control** tops the OWASP Top 10, so this is a critical area.

### Explanation

#### RBAC
Role-Based Access Control assigns permissions through roles. Simple and widely used.

```javascript
const permissions = {
  admin: ['order:read', 'order:write', 'order:delete'],
  support: ['order:read', 'order:write'],
  customer: ['order:read'],
};

const can = (permission) => (req, res, next) => {
  if (permissions[req.user.role]?.includes(permission)) return next();
  res.status(403).json({ error: 'forbidden' });
};

app.delete('/orders/:id', requireAuth, can('order:delete'), deleteOrder);
```

#### ABAC
Attribute-Based Access Control decides using attributes of the user, resource and environment. It handles rules that RBAC can't express.

```javascript
function canApproveExpense(user, expense, context) {
  return (
    user.role === 'manager' &&
    user.department === expense.department &&
    expense.amount <= user.approvalLimit &&
    expense.createdBy !== user.id &&          // no self-approval
    context.businessHours
  );
}
```

| | RBAC | ABAC |
|---|---|---|
| Decides by | Role | Attributes |
| Simple to audit | Yes | Harder |
| Fine-grained | Limited | Very |
| Risk | Role explosion | Rule complexity |

#### Policy checks
Centralize rules in one place, and always check **ownership** at the object level. Missing this creates **IDOR** bugs.

```javascript
// IDOR: any logged-in user can read any order
app.get('/orders/:id', requireAuth, async (req, res) => {
  res.json(await db.orders.findById(req.params.id));
});

// Fixed: query scoped to the user
app.get('/orders/:id', requireAuth, async (req, res) => {
  const order = await db.orders.findOne({ id: req.params.id, userId: req.user.sub });
  if (!order) return res.status(404).end();    // don't reveal that it exists
  res.json(order);
});

// Central policy module
const policies = {
  order: {
    read: (user, order) => user.role === 'admin' || order.userId === user.id,
    cancel: (user, order) => order.userId === user.id && order.status === 'pending',
  },
};

function authorize(user, action, resourceType, resource) {
  if (!policies[resourceType]?.[action]?.(user, resource)) {
    const err = new Error('forbidden');
    err.status = 403;
    throw err;
  }
}

// CASL is a popular library for the same idea
const { AbilityBuilder, createMongoAbility } = require('@casl/ability');

function defineAbility(user) {
  const { can, build } = new AbilityBuilder(createMongoAbility);
  can('read', 'Order', { userId: user.id });
  if (user.role === 'admin') can('manage', 'all');
  return build();
}
```

For large systems, consider policy engines such as **OPA**, or relationship-based systems such as **OpenFGA**. In multi-tenant systems, always filter by tenant ID. PostgreSQL row-level security gives a second layer of defence.

### Real-time scenario
**Problem:** A customer changes the invoice ID in a URL and sees another company's invoice.
**Diagnosis:** The endpoint checked login, not ownership: a classic IDOR.
**Fix:** Scope every query by tenant, add a central policy layer, and write automated cross-tenant access tests.

### Common mistakes and trade-offs
- Only hiding UI buttons. The server must enforce rules.
- Trusting a role sent by the client.
- Using sequential IDs as the only protection. UUIDs help, but ownership checks are what matter.
- **Trade-off:** RBAC is easy to explain, while ABAC is flexible but harder to audit.

### Top interview questions
**Q1. What is the difference between authentication and authorization?**
Authentication proves identity. Authorization decides what that identity may do.

**Q2. RBAC or ABAC?**
Start with RBAC for simple roles. Use ABAC when decisions depend on ownership, department or context.

**Q3. What is IDOR?**
Insecure Direct Object Reference: accessing another user's object by changing an ID, due to missing ownership checks.

**Q4. Where should authorization logic live?**
In a central policy layer used by services, so that all routes apply the same rules.

**Q5. How do you enforce multi-tenant isolation?**
Tenant scoping in every query, context from the verified token, RLS as backup, and cross-tenant tests.

### Quick revision
RBAC = roles. ABAC = attributes. Check ownership on every object. Enforce on the server. Centralize policies.

---

## 43. ⭐ OWASP for Node.js

### Why it matters
Senior engineers should recognize common vulnerabilities in Node code and know how to prevent them. Each section below shows the vulnerable pattern and the fix.

### Explanation

#### Injection
Untrusted input is treated as code or a query. This includes SQL, NoSQL and command injection.

```javascript
// SQL injection
db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);   // vulnerable
db.query('SELECT * FROM users WHERE email = $1', [req.body.email]);  // safe

// NoSQL injection: the body {"password": {"$ne": null}} matches any password
await users.findOne({ email: req.body.email, password: req.body.password }); // vulnerable

// Fix: validate types first
const { email, password } = z.object({
  email: z.string().email(),
  password: z.string(),
}).parse(req.body);

// Command injection
exec(`ping ${req.query.host}`);                 // vulnerable
execFile('ping', ['-c', '1', req.query.host]);  // no shell, plus validate the host
```

#### XSS
Cross-Site Scripting: an attacker's script runs in another user's browser, usually through unescaped output.

```javascript
// Vulnerable: reflected input as HTML
app.get('/search', (req, res) => {
  res.send(`<h1>Results for ${req.query.q}</h1>`);
});

// Fix 1: escape output
const escapeHtml = (s) => s.replace(/[&<>"']/g, (c) => ({
  '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;',
}[c]));

// Fix 2: sanitize when you must allow some HTML
const sanitizeHtml = require('sanitize-html');
const clean = sanitizeHtml(userHtml, { allowedTags: ['b', 'i', 'a'] });

// Fix 3: a Content Security Policy as a second layer
app.use(helmet.contentSecurityPolicy({
  directives: { defaultSrc: ["'self'"], scriptSrc: ["'self'"] },
}));
```

#### CSRF
Cross-Site Request Forgery: a malicious site makes the victim's browser send a request with their cookies. It matters for cookie auth, not for `Authorization` header tokens.

```javascript
// Primary defence: SameSite cookies
res.cookie('sid', id, { httpOnly: true, secure: true, sameSite: 'lax' });

// Defence in depth: check origin on state-changing requests
const allowed = new Set(['https://app.example.com']);

app.use((req, res, next) => {
  const safe = ['GET', 'HEAD', 'OPTIONS'].includes(req.method);
  if (safe || allowed.has(req.get('origin'))) return next();
  res.status(403).json({ error: 'bad origin' });
});

// Synchronizer token: a random value in the session, sent back in a header
app.use((req, res, next) => {
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return next();
  const sent = Buffer.from(req.get('x-csrf-token') ?? '');
  const expected = Buffer.from(req.session.csrfToken ?? '');
  const valid = sent.length > 0 && sent.length === expected.length &&
    crypto.timingSafeEqual(sent, expected);
  if (!valid) return res.status(403).end();
  next();
});
```

#### SSRF
Server-Side Request Forgery: your server is tricked into calling internal addresses, such as the cloud metadata service at `169.254.169.254`.

```javascript
// Vulnerable: fetches whatever URL the user gives
app.post('/preview', async (req, res) => {
  const page = await fetch(req.body.url);
  res.send(await page.text());
});

// Safer
const dns = require('node:dns/promises');
const net = require('node:net');
const ipaddr = require('ipaddr.js');

async function assertPublicUrl(raw) {
  const url = new URL(raw);
  if (url.protocol !== 'https:') throw new Error('https only');

  const { address } = await dns.lookup(url.hostname);
  if (!net.isIP(address) || ipaddr.parse(address).range() !== 'unicast') {
    throw new Error('blocked address');    // private, loopback, link-local
  }
  return url;
}

// Also: block redirects, allow-list domains where possible, enforce IMDSv2,
// and restrict outbound network access. Resolving separately leaves a small
// DNS-rebinding gap, so pin the checked IP in the connection for high-risk cases.
const safe = await assertPublicUrl(req.body.url);
const page = await fetch(safe, { redirect: 'error', signal: AbortSignal.timeout(3000) });
```

#### Prototype pollution
Setting `__proto__` through unsafe merges changes `Object.prototype`, which then affects every object.

```javascript
// Vulnerable deep merge
function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      target[key] ??= {};
      merge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
console.log({}.isAdmin);    // true: every object is now "admin"

// Fixes
const blocked = new Set(['__proto__', 'constructor', 'prototype']);
function safeMerge(target, source) {
  for (const key of Object.keys(source)) {
    if (blocked.has(key)) continue;
    const value = source[key];
    if (typeof value === 'object' && value !== null) {
      target[key] = safeMerge(target[key] ?? {}, value);
    } else {
      target[key] = value;
    }
  }
  return target;
}

const lookup = Object.create(null);   // no prototype to pollute
const settings = new Map();           // Map for user-controlled keys

// Hardening flag: removes __proto__ access altogether
// node --disable-proto=delete app.js
```

#### ReDoS
Regular expression Denial of Service: a bad pattern causes catastrophic backtracking. In Node, that blocks the event loop for **everyone**.

```javascript
// Nested quantifiers are the warning sign
const bad = /^(a+)+$/;
// bad.test('a'.repeat(30) + '!');   // can take a very long time

// Safer pattern
const good = /^a+$/;

// Limit input length before testing
function isValidSlug(input) {
  return input.length <= 100 && /^[a-z0-9-]+$/.test(input);
}

// Prefer vetted validators, and lint with eslint-plugin-security.
// The re2 package runs in linear time, so it's good for user-supplied patterns.
const RE2 = require('re2');
const safe = new RE2('^(a+)+$');
```

### Real-time scenario
**Problem:** One request with a specially crafted `User-Agent` causes a 100% CPU stall on a pod.
**Diagnosis:** A CPU profile showed the time inside one parsing regex, which backtracked heavily.
**Fix:** Replace the regex, limit header length, upgrade the library, and add event loop delay alerts.

### Common mistakes and trade-offs
- Thinking an ORM stops all injection. Raw queries and unvalidated objects can still inject.
- Blocking only obvious private IPs for SSRF.
- Disabling CSP because it's inconvenient.
- **Trade-off:** strict security rules add friction, so apply layered defences near your highest-risk paths first.

### Top interview questions
**Q1. How do you prevent SQL and NoSQL injection?**
Parameterized queries, schema validation, and rejecting objects where strings are expected.

**Q2. What is prototype pollution?**
Changing `Object.prototype` through keys like `__proto__`, so that properties appear on all objects.

**Q3. What is ReDoS, and why is it serious in Node?**
A regex takes exponential time on crafted input. On Node's single thread, this stops all requests.

**Q4. What is the difference between XSS and CSRF?**
XSS runs attacker script in your site. CSRF makes the browser send an unwanted authenticated request.

**Q5. How do you prevent SSRF?**
Allow-list destinations, block private ranges after DNS resolution, disable redirects, and limit network egress.

**Q6. Does JWT in an `Authorization` header need CSRF protection?**
Usually no, because browsers don't attach it automatically. Cookies do need it.

**Q7. What is the main XSS defence in a Node server?**
Context-aware output encoding, with templating that escapes by default, plus CSP as a backup.

### Quick revision
Parameterize, validate types, escape output, use `SameSite`, block private IPs, guard against `__proto__`, avoid nested regex quantifiers.

---

## 44. API Hardening

### Why it matters
Hardening is the layer of default protections every production API needs. Misconfigured CORS and leaked secrets are very common findings.

### Explanation

#### CORS
Browsers block cross-origin reads unless the server allows them. CORS is a **browser** rule, not server authentication. `curl` ignores it.

```javascript
const cors = require('cors');

const allowed = new Set(['https://app.example.com', 'https://admin.example.com']);

app.use(cors({
  origin(origin, callback) {
    // No Origin header: non-browser client, so auth still applies
    if (!origin || allowed.has(origin)) return callback(null, true);
    callback(new Error('origin not allowed'));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  maxAge: 600,      // cache preflight responses
}));

// Dangerous: reflects any origin and allows cookies
// app.use(cors({ origin: true, credentials: true }));
```

A **preflight** is an `OPTIONS` request the browser sends before non-simple requests, such as JSON POSTs or those with custom headers.

#### Helmet
Sets security headers with a single line.

```javascript
const helmet = require('helmet');

app.disable('x-powered-by');
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      imgSrc: ["'self'", 'https://cdn.example.com'],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true },
}));
```

| Header | Protects against |
|---|---|
| `Content-Security-Policy` | XSS and data injection |
| `Strict-Transport-Security` | HTTPS downgrade |
| `X-Content-Type-Options: nosniff` | MIME sniffing |
| `X-Frame-Options` / `frame-ancestors` | Clickjacking |
| `Referrer-Policy` | URL leakage |

More hardening:

```javascript
app.set('trust proxy', 1);                      // trust exactly one proxy hop
app.use(express.json({ limit: '100kb' }));      // cap body size

// Don't leak internals in production
app.use((err, req, res, next) => {
  req.log?.error(err);
  res.status(err.status ?? 500).json({
    error: err.expose ? err.message : 'internal error',
    requestId: req.id,
  });
});
```

#### Secrets management
Secrets must not be in code, images or logs. Use a manager, rotate them, and use short-lived credentials if you can.

```javascript
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

const client = new SecretsManagerClient({});

async function loadDbCredentials() {
  const { SecretString } = await client.send(
    new GetSecretValueCommand({ SecretId: 'prod/orders/db' }),
  );
  return JSON.parse(SecretString);
}

// Redact sensitive log fields
const logger = require('pino')({
  redact: ['req.headers.authorization', 'req.headers.cookie', '*.password', '*.token'],
});
```

```bash
# Scan for committed secrets in CI
gitleaks detect --source . --verbose
```

Prefer IAM roles for AWS access, so the app holds no long-lived keys. If a secret leaks, **rotate it**. Removing it from git history isn't enough.

### Real-time scenario
**Problem:** AWS keys committed in a public repo are abused within minutes to run crypto miners.
**Diagnosis:** A developer committed `.env` for debugging, and automated scanners harvest GitHub all the time.
**Fix:** Revoke and rotate immediately, audit CloudTrail, switch to IAM roles, and add gitleaks in pre-commit and CI.

### Common mistakes and trade-offs
- Treating CORS as API security.
- Using `Access-Control-Allow-Origin: *` with credentials, which browsers reject. Reflecting any origin is worse.
- Logging full request bodies or headers.
- **Trade-off:** a strict CSP blocks many attacks but needs tuning. Roll it out with report-only mode first.

### Top interview questions
**Q1. What is CORS?**
A browser mechanism that lets servers say which other origins may read responses.

**Q2. What is a preflight request?**
An `OPTIONS` check sent by the browser before non-simple requests, to confirm that they're allowed.

**Q3. Does CORS protect your API from `curl`?**
No. Only browsers enforce it, so APIs still need authentication and authorization.

**Q4. What does Helmet do?**
It sets safe HTTP security headers such as CSP, HSTS and `nosniff`.

**Q5. How should secrets be stored?**
In a secrets manager, injected at runtime, with least privilege, rotation and audit logs.

**Q6. What do you do when a secret leaks?**
Rotate it now, audit access, remove it from history, and add scanning to prevent a repeat.

### Quick revision
CORS is browser-only, and allow-list your origins. Helmet sets headers. Cap bodies. Secrets in a manager, never in git. Redact logs.

---

## 45. Supply Chain Security

### Why it matters
A typical Node app runs hundreds of third-party packages, and each one can execute code on your machine. Several npm packages have been compromised in real attacks.

### Explanation

#### `npm audit`
Checks installed packages against a database of known vulnerabilities.

```bash
npm audit                          # all findings
npm audit --omit=dev               # production dependencies only
npm audit --audit-level=high       # exit with failure at high or above: good for CI
npm audit fix                      # applies compatible upgrades
npm audit signatures               # verifies registry signatures
```

Not every finding is exploitable. Triage by reachability: a ReDoS in a build tool is less urgent than one in a request path.

Force a safe version of a nested dependency with `overrides`:

```json
{
  "overrides": {
    "vulnerable-package": "2.1.4"
  }
}
```

#### Lockfiles
The lockfile pins versions and integrity hashes. If the downloaded package doesn't match the hash, install fails.

```json
{
  "node_modules/express": {
    "version": "5.1.0",
    "resolved": "https://registry.npmjs.org/express/-/express-5.1.0.tgz",
    "integrity": "sha512-..."
  }
}
```

```bash
npm ci                             # enforces the lockfile
```

Review lockfile changes in PRs, because attackers can hide a changed `resolved` URL there. `lockfile-lint` can check for this.

#### Malicious packages

| Attack | How it works |
|---|---|
| Typosquatting | `expresss` instead of `express` |
| Account takeover | Attacker publishes a bad version of a real package |
| Install scripts | `postinstall` runs code during `npm install` |
| Dependency confusion | Public package uses the name of your private one |
| Protestware | A maintainer adds harmful code |

```bash
# Don't run lifecycle scripts by default
npm ci --ignore-scripts
npm config set ignore-scripts true

# Inspect a package before adding it
npm view some-package maintainers time.modified
```

```ini
# .npmrc: scope private packages to your registry, preventing dependency confusion
@mycompany:registry=https://npm.mycompany.com/
```

Protections for your team:

```yaml
# Renovate: wait before adopting brand-new releases
{
  "extends": ["config:recommended"],
  "minimumReleaseAge": "3 days"
}
```

Also use a private registry proxy, an SBOM, tools such as Socket or Snyk, and 2FA on npm accounts. If you publish packages, use provenance and trusted publishing from CI.

Node's permission model (Topic 66) can add runtime limits on file and network access.

### Real-time scenario
**Problem:** A popular package gets a new patch release that steals environment variables through a `postinstall` script.
**Diagnosis:** A dev machine and a CI run with a loose `npm install` picked it up. Teams using lockfiles with `npm ci` weren't affected.
**Fix:** Pin to a clean version, rotate all secrets on exposed machines, and adopt `--ignore-scripts`, `npm ci` and a release-age delay.

### Common mistakes and trade-offs
- Running `npm audit fix --force` blindly, which can cause breaking upgrades.
- Adding a package for something a few lines of code can do.
- Leaving CI tokens exposed to install scripts.
- **Trade-off:** slower dependency adoption reduces exposure to bad releases, but delays security fixes. Allow fast-tracking for CVE fixes.

### Top interview questions
**Q1. How do you secure npm dependencies?**
Lockfiles with `npm ci`, audits in CI, restricted install scripts, automated upgrades and a vetted registry.

**Q2. What is dependency confusion?**
A public package with the same name as an internal one gets installed. Scoped packages and registry config prevent it.

**Q3. Why are `postinstall` scripts risky?**
They run arbitrary code during install, with access to secrets and files.

**Q4. How does a lockfile help security?**
It pins exact versions and verifies integrity hashes, so upstream changes can't slip in.

**Q5. What is an SBOM?**
A Software Bill of Materials: a list of components. It lets you find affected services quickly during an incident.

**Q6. How do you handle an audit finding with no fix?**
Check reachability, then use `overrides`, replace the package or add a compensating control. Document the decision.

### Quick revision
`npm ci`, commit lockfiles, ignore install scripts, audit in CI, scope private packages, delay fresh releases, use an SBOM.

---

## 46. ⭐ Rate Limiting

### Why it matters
Rate limiting protects services from abuse, brute force and overload. It's also a common design and coding question (see Topic 72).

### Explanation

#### Token bucket
A bucket holds tokens that refill at a steady rate. Each request takes one. It **allows bursts** up to capacity, and enforces a long-term average.

```javascript
class TokenBucket {
  constructor(capacity, refillPerSecond) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillPerSecond = refillPerSecond;
    this.lastRefill = Date.now();
  }

  allow(cost = 1) {
    const now = Date.now();
    const seconds = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + seconds * this.refillPerSecond);
    this.lastRefill = now;

    if (this.tokens < cost) return false;
    this.tokens -= cost;
    return true;
  }
}

const bucket = new TokenBucket(10, 5);   // burst of 10, 5 per second afterwards
```

#### Sliding window
A **fixed window** can allow double traffic at a boundary: 100 requests at 12:00:59 and 100 more at 12:01:00. A sliding window fixes this.

```javascript
// Sliding window log: exact, but stores every timestamp
class SlidingWindowLog {
  constructor(limit, windowMs) {
    this.limit = limit;
    this.windowMs = windowMs;
    this.hits = new Map();
  }

  allow(key) {
    const now = Date.now();
    const recent = (this.hits.get(key) ?? []).filter((t) => now - t < this.windowMs);

    if (recent.length >= this.limit) {
      this.hits.set(key, recent);
      return false;
    }
    recent.push(now);
    this.hits.set(key, recent);
    return true;
  }
}

// Sliding window counter: an approximation using very little memory
function estimate(previousCount, currentCount, elapsedInWindow, windowMs) {
  const overlap = 1 - elapsedInWindow / windowMs;
  return previousCount * overlap + currentCount;
}
```

| Algorithm | Bursts | Memory | Accuracy |
|---|---|---|---|
| Fixed window | Boundary spike | Low | Low |
| Sliding log | Controlled | High | Exact |
| Sliding counter | Controlled | Low | Close |
| Token bucket | Allowed by design | Low | Good |
| Leaky bucket | Smoothed | Low | Good |

#### Distributed limits with Redis
An in-memory limiter works per instance, so 10 pods give 10× the limit. Keep counts in Redis, and use a **Lua script** so that read and update stay atomic.

```javascript
const script = `
  local key = KEYS[1]
  local now = tonumber(ARGV[1])
  local window = tonumber(ARGV[2])
  local limit = tonumber(ARGV[3])
  local member = ARGV[4]

  redis.call('ZREMRANGEBYSCORE', key, 0, now - window)
  local count = redis.call('ZCARD', key)

  if count >= limit then
    return {0, count}
  end

  redis.call('ZADD', key, now, member)
  redis.call('PEXPIRE', key, window)
  return {1, count + 1}
`;

const rateLimit = ({ limit, windowMs, keyFn }) => async (req, res, next) => {
  try {
    const [allowed, count] = await redis.eval(script, {
      keys: [`rate:${keyFn(req)}`],
      arguments: [String(Date.now()), String(windowMs), String(limit), randomUUID()],
    });

    res.set('RateLimit-Limit', String(limit));
    res.set('RateLimit-Remaining', String(Math.max(0, limit - count)));

    if (!allowed) {
      res.set('Retry-After', String(Math.ceil(windowMs / 1000)));
      return res.status(429).json({ error: 'too many requests' });
    }
    next();
  } catch (err) {
    logger.warn({ err }, 'rate limiter unavailable');
    next();     // fail open here. For login endpoints, consider failing closed.
  }
};

app.post('/login', rateLimit({
  limit: 5,
  windowMs: 15 * 60_000,
  keyFn: (req) => `login:${req.ip}:${req.body.email}`,
}), loginHandler);

app.use('/api', rateLimit({
  limit: 100,
  windowMs: 60_000,
  keyFn: (req) => req.user?.id ?? req.ip,
}));
```

Libraries such as `rate-limiter-flexible` and `express-rate-limit` with a Redis store cover most needs. Behind a proxy, set `trust proxy` properly, or every user will appear to share one IP.

Use several layers: CDN or WAF for floods, gateway for general limits, and the app for business rules.

### Real-time scenario
**Problem:** Attackers use leaked passwords against the login endpoint, at a slow rate from thousands of IPs.
**Diagnosis:** The per-IP limit never triggered, because each IP sent only a few attempts.
**Fix:** Limit by account as well as IP, add progressive delays and CAPTCHA, and use a breached-password check.

### Common mistakes and trade-offs
- In-memory limits across multiple instances.
- Limiting by IP alone: offices share IPs, and attackers rotate them.
- Trusting `X-Forwarded-For` from anyone.
- **Trade-off:** failing open keeps the service up but loses protection. Failing closed is safer for sensitive routes.

### Top interview questions
**Q1. Explain token bucket.**
Tokens refill at a set rate up to a capacity. Each request uses one. It permits bursts while limiting the average.

**Q2. What is the problem with fixed windows?**
Bursts at the boundary can reach twice the intended limit.

**Q3. How do you rate limit across servers?**
Shared state in Redis, updated atomically with Lua, plus limits at the gateway.

**Q4. What should you return when limited?**
429 with `Retry-After` and rate-limit headers.

**Q5. What key should you use?**
User or API key when authenticated, and IP otherwise. Use combinations for login.

**Q6. Should a limiter fail open or closed?**
Open for general APIs, to protect availability. Closed for sensitive actions such as password reset.

**Q7. Token bucket or leaky bucket?**
Token bucket accepts bursts. Leaky bucket produces a smooth, constant outflow.

### Quick revision
Token bucket allows bursts. Sliding window fixes boundaries. Use Redis + Lua for atomic distributed limits. Return 429 with `Retry-After`.

---

# Part 9: Reliability and Testing

## 47. ⭐ Error Handling

### Why it matters
Node's async model gives errors many escape routes. Knowing what to catch, what to log and when to crash is a clear senior signal.

### Explanation

#### Operational vs programmer errors

| | Operational | Programmer |
|---|---|---|
| What | Expected runtime failures | Bugs |
| Examples | Timeout, invalid input, DB down | `undefined` property, wrong argument |
| Response | Handle, retry, return 4xx/5xx | Fix the code, usually crash and restart |

```javascript
class AppError extends Error {
  constructor(message, { status = 500, code = 'INTERNAL', isOperational = true, cause } = {}) {
    super(message, { cause });
    this.name = this.constructor.name;
    this.status = status;
    this.code = code;
    this.isOperational = isOperational;
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, { status: 404, code: 'NOT_FOUND' });
  }
}

class UpstreamError extends AppError {
  constructor(service, cause) {
    super(`${service} failed`, { status: 502, code: 'UPSTREAM', cause });
  }
}

// Keep the original error with `cause`
async function getStock(sku) {
  try {
    return await inventoryClient.get(sku);
  } catch (err) {
    throw new UpstreamError('inventory', err);
  }
}
```

#### `uncaughtException`
Fires when an error reaches the top of the stack. The process is in an **unknown state**, so log, clean up quickly and exit. Don't continue.

```javascript
process.on('uncaughtException', (err, origin) => {
  logger.fatal({ err, origin }, 'uncaught exception');
  shutdown(1);
});

process.on('unhandledRejection', (reason) => {
  logger.fatal({ reason }, 'unhandled rejection');
  shutdown(1);
});

function shutdown(code) {
  server.close(() => process.exit(code));
  setTimeout(() => process.exit(code), 5000).unref();  // don't hang forever
}
```

Why not just continue? A bug may have left locks held, transactions open or data partly updated. Restarting gives a clean state.

#### Centralized handlers
Handlers throw. One piece of middleware decides the response.

```javascript
app.get('/orders/:id', async (req, res) => {
  const order = await orders.find(req.params.id);
  if (!order) throw new NotFoundError('order');
  res.json(order);
});

// Single place for error responses
app.use((err, req, res, next) => {
  if (res.headersSent) return next(err);  // the response has started

  const operational = err instanceof AppError && err.isOperational;
  const status = operational ? err.status : 500;

  if (status >= 500) req.log.error({ err }, 'request failed');
  else req.log.warn({ code: err.code }, err.message);

  res.status(status).json({
    error: {
      code: operational ? err.code : 'INTERNAL',
      message: operational ? err.message : 'Something went wrong',
      requestId: req.id,
    },
  });
});
```

Where errors escape:

```javascript
// 1. Callback: try/catch doesn't reach it
// 2. Floating promise: becomes an unhandled rejection
// 3. Unhandled 'error' event on a stream or emitter
stream.on('error', handleError);

// 4. Mid-stream failure: the status has already been sent
app.get('/export', async (req, res, next) => {
  try {
    await pipeline(dbStream, csvTransform, res);
  } catch (err) {
    if (!res.headersSent) return next(err);
    res.destroy(err);       // tells the client the download failed
  }
});

// 5. Timers and event listeners run outside your try/catch
```

### Real-time scenario
**Problem:** After an error, a pod stays up, but returns strange data for some users until restarted.
**Diagnosis:** A global `uncaughtException` handler logged the error and carried on. The failure left a shared cache half-updated.
**Fix:** Log and exit on `uncaughtException`. Let Kubernetes replace the pod, and add alerts on restart rates.

### Common mistakes and trade-offs
- Swallowing errors with an empty `catch`.
- Sending stack traces to clients.
- Throwing strings, which lose the stack.
- Logging and rethrowing at each layer, producing duplicate logs.
- **Trade-off:** crashing on bugs hurts in-flight requests, but continuing risks data corruption. Run several instances so a crash is cheap.

### Top interview questions
**Q1. What is the difference between operational and programmer errors?**
Operational errors are expected failures you handle. Programmer errors are bugs to fix, and they usually justify a crash.

**Q2. Should you continue after `uncaughtException`?**
No. Log, clean up briefly and exit, because state is unknown. A supervisor restarts the process.

**Q3. How do you handle errors in async Express handlers?**
Express 5 forwards rejections. In 4, use a wrapper. Either way, centralize responses in error middleware.

**Q4. Why extend `Error`?**
For status codes, error codes and type checks, while keeping the stack trace.

**Q5. What is `Error.cause`?**
A standard way to wrap an error and keep the original for debugging.

**Q6. How do you handle an error after the response has started?**
You can't change the status. Destroy the connection, so the client sees a failed response.

### Quick revision
Handle operational errors, crash on bugs. Use `AppError` + central middleware. Exit on `uncaughtException`. Keep `cause`. Never leak stack traces.

---

## 48. ⭐ Graceful Shutdown

### Why it matters
Every deploy stops processes. Without graceful shutdown, each release causes failed requests and interrupted work.

### Explanation

#### `SIGTERM`
Kubernetes and Docker send `SIGTERM`, wait for a grace period (30 s by default in Kubernetes), and then send `SIGKILL`.

```mermaid
sequenceDiagram
    participant K as Kubernetes
    participant A as Node app
    K->>A: SIGTERM, pod marked as terminating
    A->>A: Readiness fails, no new traffic
    A->>A: Stop accepting, finish in-flight work
    A->>A: Close DB, Redis, queues
    A->>K: Exit 0
    Note over K,A: SIGKILL if the grace period runs out
```

#### Draining connections
`server.close()` stops new connections and waits for existing ones. Idle keep-alive sockets can hold it open, so close those as well.

```javascript
const http = require('node:http');
const { setTimeout: sleep } = require('node:timers/promises');

const server = http.createServer(app);
let shuttingDown = false;

async function shutdown(signal) {
  if (shuttingDown) return;
  shuttingDown = true;
  logger.info({ signal }, 'shutting down');

  // Hard limit, kept below the Kubernetes grace period
  const force = setTimeout(() => {
    logger.error('forced shutdown');
    process.exit(1);
  }, 25_000);
  force.unref();

  // 1. Fail readiness, then give the load balancer time to notice
  await sleep(5000);

  // 2. Stop new connections, and wait for in-flight requests
  await new Promise((resolve) => {
    server.close(resolve);
    server.closeIdleConnections();
  });

  // 3. Stop background work, then close dependencies, in reverse order of startup
  await worker.close();
  await redis.quit();
  await pool.end();

  logger.info('clean exit');
  process.exit(0);
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

The `sleep` matters. Removal from load balancers isn't instant, so without it, traffic can arrive at a closed port.

For WebSockets and SSE, notify clients and close the connections, because they never finish on their own.

#### Health checks
Different probes answer different questions.

| Probe | Question | On failure |
|---|---|---|
| Startup | Has it finished booting? | Waits, then restarts |
| Liveness | Is the process stuck? | Restarts the pod |
| Readiness | Can it take traffic? | Removes from load balancer |

```javascript
// Liveness: keep it simple. Don't check dependencies here.
app.get('/livez', (req, res) => res.status(200).send('ok'));

// Readiness: can this instance do useful work?
app.get('/readyz', async (req, res) => {
  if (shuttingDown) return res.status(503).send('draining');

  try {
    await pool.query('SELECT 1');
    res.status(200).send('ready');
  } catch {
    res.status(503).send('database unavailable');
  }
});
```

```yaml
livenessProbe:
  httpGet: { path: /livez, port: 3000 }
  periodSeconds: 10
readinessProbe:
  httpGet: { path: /readyz, port: 3000 }
  periodSeconds: 5
terminationGracePeriodSeconds: 40
```

If liveness checks the database, a DB outage restarts every pod, making things worse.

### Real-time scenario
**Problem:** Every deploy produces a burst of 502 errors, and some orders are left half-finished.
**Diagnosis:** Node ran under `npm start`, which didn't pass on `SIGTERM`. There was no handler anyway, so requests died at `SIGKILL`.
**Fix:** Run `node` directly with `tini` or `--init`, add draining logic and a readiness failure, and keep the grace period above the longest request.

### Common mistakes and trade-offs
- `process.exit()` in the signal handler, with no draining.
- Grace period shorter than the slowest in-flight work.
- Not handling idle keep-alive connections, so `server.close` never finishes.
- **Trade-off:** long drains give clean exits but slow down deploys and autoscaling. Set them based on real p99 request time.

### Top interview questions
**Q1. How do you implement graceful shutdown?**
Catch `SIGTERM`, fail readiness, wait, stop accepting, finish in-flight work, close dependencies, then exit. Add a forced timeout.

**Q2. What does `server.close()` do?**
It stops new connections and calls back when existing ones finish. Handle idle keep-alive sockets separately.

**Q3. What is the difference between liveness and readiness?**
Liveness failure restarts the container. Readiness failure removes it from traffic.

**Q4. Should liveness check the database?**
No. A database outage would restart healthy pods without helping.

**Q5. Why does Node sometimes not receive `SIGTERM` in Docker?**
It's behind `npm` or a shell that doesn't forward it, or it runs as PID 1 without a handler.

**Q6. How do you shut down a queue worker?**
Stop fetching, finish or requeue current jobs, close connections and exit.

### Quick revision
`SIGTERM` → fail readiness → wait → `server.close` → finish work → close dependencies → exit. Force exit before `SIGKILL`. Liveness stays simple.

---

## 49. Resilience Patterns

### Why it matters
Dependencies fail. Resilience patterns stop one slow service from taking down everything that calls it.

### Explanation

#### Timeouts
The first and most important pattern. Every network call needs one. See Topic 13.

```javascript
const res = await fetch('http://inventory/items/1', {
  signal: AbortSignal.timeout(800),   // based on the dependency's p99
});
```

Set timeouts at each layer, with the inner ones shorter:

```text
Client 5 s > API gateway 4 s > Service 3 s > Database 2 s
```

#### Retries
Use limits, backoff and jitter, and only for idempotent work (Topic 13). A **retry budget** caps the extra load.

```javascript
class RetryBudget {
  constructor(ratio = 0.1) {
    this.ratio = ratio;       // retries limited to 10% of requests
    this.requests = 0;
    this.retries = 0;
    setInterval(() => { this.requests = 0; this.retries = 0; }, 10_000).unref();
  }

  recordRequest() { this.requests++; }

  canRetry() {
    if (this.retries >= Math.max(1, this.requests * this.ratio)) return false;
    this.retries++;
    return true;
  }
}
```

During an outage, this stops retries from multiplying traffic on a struggling service.

#### Circuit breakers
A breaker stops calling a failing dependency for a while, so requests fail fast rather than waiting.

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: failures exceed threshold
    Open --> HalfOpen: cool-down elapsed
    HalfOpen --> Closed: trial succeeds
    HalfOpen --> Open: trial fails
```

```javascript
class CircuitBreaker {
  constructor(fn, { failureThreshold = 5, resetMs = 10_000 } = {}) {
    this.fn = fn;
    this.failureThreshold = failureThreshold;
    this.resetMs = resetMs;
    this.state = 'CLOSED';
    this.failures = 0;
    this.openedAt = 0;
  }

  async fire(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.openedAt < this.resetMs) {
        throw new Error('circuit open');
      }
      this.state = 'HALF_OPEN';     // allow a trial call
    }

    try {
      const result = await this.fn(...args);
      this.state = 'CLOSED';
      this.failures = 0;
      return result;
    } catch (err) {
      this.failures++;
      if (this.state === 'HALF_OPEN' || this.failures >= this.failureThreshold) {
        this.state = 'OPEN';
        this.openedAt = Date.now();
      }
      throw err;
    }
  }
}

// Production library
const CircuitBreakerLib = require('opossum');

const breaker = new CircuitBreakerLib(getRecommendations, {
  timeout: 800,
  errorThresholdPercentage: 50,
  resetTimeout: 10_000,
  volumeThreshold: 20,
});

breaker.fallback(() => ({ items: [], source: 'fallback' }));
breaker.on('open', () => logger.warn('recommendations circuit open'));

const result = await breaker.fire(userId);
```

#### Bulkheads
Isolate resources so that one failing dependency can't use up everything, like compartments in a ship.

```javascript
class Bulkhead {
  constructor(maxConcurrent, maxQueue) {
    this.maxConcurrent = maxConcurrent;
    this.maxQueue = maxQueue;
    this.active = 0;
    this.queue = [];
  }

  async run(fn) {
    if (this.active >= this.maxConcurrent) {
      if (this.queue.length >= this.maxQueue) {
        throw new Error('bulkhead full');       // reject rather than pile up
      }
      await new Promise((resolve) => this.queue.push(resolve));
    } else {
      this.active++;
    }

    try {
      return await fn();
    } finally {
      const next = this.queue.shift();
      if (next) next();      // hand the slot over
      else this.active--;
    }
  }
}

// Each dependency gets its own limit
const paymentBulkhead = new Bulkhead(50, 100);
const reportingBulkhead = new Bulkhead(5, 10);

await reportingBulkhead.run(() => reportsClient.get(id));
```

Separate connection pools per dependency, or separate deployments for critical and bulk traffic, are the same idea.

### Real-time scenario
**Problem:** The recommendations service slows down, and checkout starts failing even though it doesn't need recommendations.
**Diagnosis:** Calls without timeouts piled up and took all the sockets and memory: a classic cascading failure.
**Fix:** Add a timeout, a circuit breaker with an empty-list fallback, and a bulkhead. Checkout stays healthy, with fewer suggestions.

### Common mistakes and trade-offs
- A circuit breaker with no fallback plan.
- Retrying without backoff during an outage.
- Thresholds based on guesses rather than real traffic.
- **Trade-off:** patterns add complexity and tuning. Apply them around dependencies that really can hurt you.

### Top interview questions
**Q1. What is a circuit breaker?**
It stops calls to a failing service after a threshold, fails fast, and tests recovery in a half-open state.

**Q2. What are its states?**
Closed passes requests. Open rejects them. Half-open allows trial requests.

**Q3. What is a bulkhead?**
A limit that isolates resources by dependency, so one failure can't starve others.

**Q4. How do retries cause outages?**
They multiply load on a struggling service. Use budgets, backoff and jitter.

**Q5. What is a cascading failure?**
One slow service exhausts its callers' resources, which spreads failure upstream.

**Q6. What is graceful degradation?**
Keeping core features running with reduced functionality, such as cached or default data.

### Quick revision
Timeout first. Retries with budgets. Breaker fails fast: closed → open → half-open. Bulkheads isolate. Always have a fallback.

---

## 50. ⭐ Testing Strategy

### Why it matters
Senior engineers are asked what to test and at which level, not only how. A good strategy gives confidence without slow, flaky pipelines.

### Explanation

```text
         /\        E2E: few, critical user journeys
        /  \
       /----\      Contract: between services
      /      \
     /--------\    Integration: real DB, real HTTP
    /          \
   /------------\  Unit: many, fast
```

For API services, many teams use more integration tests than the classic pyramid suggests, since most bugs live at the edges.

#### Unit tests
Test one piece of logic, with no I/O. Fast and focused.

```javascript
const { test } = require('node:test');
const assert = require('node:assert/strict');

function calculateDiscount(total, isMember) {
  if (total < 0) throw new RangeError('invalid total');
  return isMember && total >= 100 ? total * 0.1 : 0;
}

test('gives members 10% on orders of 100 or more', () => {
  assert.equal(calculateDiscount(200, true), 20);
});

test('gives nothing below the threshold', () => {
  assert.equal(calculateDiscount(99, true), 0);
});

test('rejects negative totals', () => {
  assert.throws(() => calculateDiscount(-1, true), RangeError);
});
```

#### Integration tests
Test how parts work together: routes, validation, repository and a real database.

```javascript
const request = require('supertest');

test('POST /orders stores the order', async () => {
  const res = await request(app)
    .post('/orders')
    .send({ sku: 'A1', quantity: 2 })
    .expect(201);

  const { rows } = await pool.query('SELECT * FROM orders WHERE id = $1', [res.body.id]);
  assert.equal(rows[0].quantity, 2);
});

test('POST /orders rejects bad input', async () => {
  await request(app).post('/orders').send({ quantity: -1 }).expect(422);
});
```

#### E2E tests
Test a real user journey through the deployed system. Slow, so keep them few.

```javascript
// Playwright: a critical journey through the real stack
import { test, expect } from '@playwright/test';

test('customer can check out', async ({ page }) => {
  await page.goto('/products/a1');
  await page.getByRole('button', { name: 'Add to cart' }).click();
  await page.goto('/checkout');
  await page.getByLabel('Card number').fill('4242424242424242');
  await page.getByRole('button', { name: 'Pay' }).click();
  await expect(page.getByText('Order confirmed')).toBeVisible();
});
```

#### Contract tests
Verify that a consumer and a provider agree on an API, with no need to run both together. **Pact** is the common tool.

```javascript
// The consumer defines what it needs
const { PactV3, MatchersV3 } = require('@pact-foundation/pact');

const provider = new PactV3({ consumer: 'checkout', provider: 'inventory' });

test('gets stock for a SKU', async () => {
  provider
    .given('sku A1 exists')
    .uponReceiving('a stock request')
    .withRequest({ method: 'GET', path: '/items/A1' })
    .willRespondWith({
      status: 200,
      body: { sku: 'A1', quantity: MatchersV3.integer(5) },
    });

  await provider.executeTest(async (mock) => {
    const item = await inventoryClient(mock.url).get('A1');
    assert.equal(item.sku, 'A1');
  });
});

// The inventory service's CI verifies this contract against its real API.
```

| Level | Speed | Finds | Count |
|---|---|---|---|
| Unit | ms | Logic bugs | Many |
| Integration | Seconds | Wiring, SQL, validation | Many |
| Contract | Seconds | Breaking API changes | Per integration |
| E2E | Minutes | Journey and config bugs | Few |

### Real-time scenario
**Problem:** Unit test coverage is 95%, yet a deploy breaks orders because a column was renamed.
**Diagnosis:** Every test mocked the repository, so the real SQL never ran.
**Fix:** Add integration tests with a real PostgreSQL via Testcontainers. Keep mocks for external APIs only.

### Common mistakes and trade-offs
- Chasing a coverage number instead of risk coverage.
- Testing implementation details, which breaks on every refactor.
- Sharing state between tests, leading to flaky runs.
- **Trade-off:** high-level tests give more confidence but run slowly and fail in less precise ways.

### Top interview questions
**Q1. How do you decide what to test?**
By risk: payment and auth get the most attention. Then business logic and integration points.

**Q2. What is the difference between unit and integration tests?**
Unit tests isolate logic with no I/O. Integration tests run real components together.

**Q3. What is contract testing?**
Checking that service consumers and providers agree on an API, independently of each other.

**Q4. How do you deal with flaky tests?**
Find the cause: time, shared state, order, network. Isolate data, control time and quarantine with a deadline.

**Q5. Is 100% coverage a good goal?**
No. Coverage shows what ran, not what was verified. Use it to spot gaps.

**Q6. How do you test failure paths?**
Fake timeouts and 500s from dependencies, then verify retries, fallbacks and error responses.

### Quick revision
Unit for logic, integration for wiring, contract for APIs, E2E for key journeys. Test by risk. Real DB in integration tests.

---

## 51. Testing Tools

### Why it matters
Tool choice affects speed and reliability. Interviews look for practical knowledge of mocking, time control and isolated test databases.

### Explanation

#### `node:test`
Built into Node, so no dependency is needed. It includes mocks, coverage and watch mode.

```javascript
const { describe, it, before, after, mock } = require('node:test');
const assert = require('node:assert/strict');

describe('orders', () => {
  before(() => { /* setup */ });
  after(() => { /* cleanup */ });

  it('calls the payment gateway', async () => {
    const charge = mock.fn(async () => ({ id: 'p1' }));
    const result = await placeOrder({ total: 100 }, { charge });

    assert.equal(result.paymentId, 'p1');
    assert.equal(charge.mock.callCount(), 1);
    assert.deepEqual(charge.mock.calls[0].arguments, [100]);
  });
});
```

```bash
node --test                              # run all tests
node --test --watch                      # re-run on changes
node --test --experimental-test-coverage # coverage report
```

#### Jest
The long-time standard: runner, assertions, mocks and snapshots in a single package.

```javascript
jest.mock('./emailClient');
const emailClient = require('./emailClient');

describe('signup', () => {
  beforeEach(() => jest.clearAllMocks());

  it('sends a welcome email', async () => {
    emailClient.send.mockResolvedValue({ id: 'e1' });

    await signup({ email: 'a@example.com' });

    expect(emailClient.send).toHaveBeenCalledWith(
      expect.objectContaining({ to: 'a@example.com' }),
    );
  });
});
```

#### Vitest
Jest-like API, fast, with excellent TypeScript and ESM support.

```typescript
import { describe, it, expect, vi } from 'vitest';
import { getRate } from './rates';

vi.mock('./client', () => ({
  fetchRate: vi.fn().mockResolvedValue(83.5),
}));

describe('getRate', () => {
  it('rounds the rate', async () => {
    await expect(getRate('USD')).resolves.toBe(84);
  });
});
```

| Tool | Setup | ESM and TS | Ecosystem |
|---|---|---|---|
| `node:test` | None | ESM yes, TS via Node or a loader | Growing |
| Jest | Medium | Needs config | Largest |
| Vitest | Low | Excellent | Large |

#### Supertest
Sends HTTP requests to an app without opening a real port. For this to work, export `app` separately from `app.listen()`.

```javascript
// app.js: exports the app
const app = express();
module.exports = app;

// server.js: the only place that listens
require('./app').listen(3000);

// orders.test.js
const request = require('supertest');
const app = require('./app');

test('requires auth', async () => {
  await request(app).get('/orders').expect(401);
});

test('returns orders', async () => {
  const res = await request(app)
    .get('/orders')
    .set('Authorization', `Bearer ${testToken}`)
    .expect(200);

  assert.equal(res.body.data.length, 2);
});
```

#### Mocking
Mock what you don't own and can't run: external APIs, clocks, randomness. Prefer real implementations for your own code. Dependency injection makes this simple.

```javascript
// 1. Inject fakes rather than patching modules
function createOrderService({ repo, gateway, clock = Date }) {
  return {
    async place(order) {
      const payment = await gateway.charge(order.total);
      return repo.save({ ...order, paymentId: payment.id, at: clock.now() });
    },
  };
}

// 2. Control time without real waits
const { mock } = require('node:test');
mock.timers.enable({ apis: ['setTimeout'] });
const fn = mock.fn();
setTimeout(fn, 5000);
mock.timers.tick(5000);
assert.equal(fn.mock.callCount(), 1);
mock.timers.reset();

// 3. Intercept outgoing HTTP
const nock = require('nock');
nock('https://api.stripe.com').post('/v1/charges').reply(500);
```

`nock` patches Node's `http` client. For native `fetch`, Undici's `MockAgent` or **MSW** are good choices.

#### Testcontainers
Starts real dependencies in Docker for tests. You get production behaviour, not an in-memory imitation.

```javascript
const { PostgreSqlContainer } = require('@testcontainers/postgresql');
const { Pool } = require('pg');

let container;
let pool;

before(async () => {
  container = await new PostgreSqlContainer('postgres:17').start();
  pool = new Pool({ connectionString: container.getConnectionUri() });
  await runMigrations(pool);
}, { timeout: 60_000 });

after(async () => {
  await pool.end();
  await container.stop();
});

beforeEach(async () => {
  await pool.query('TRUNCATE orders, customers RESTART IDENTITY CASCADE');
});
```

### Real-time scenario
**Problem:** Integration tests pass with SQLite but fail in production on PostgreSQL JSON queries.
**Diagnosis:** The fake database didn't match production behaviour.
**Fix:** Use Testcontainers with the same PostgreSQL version as production. CI time rises by a minute, but this class of bug disappears.

### Common mistakes and trade-offs
- Mocking everything, so tests only check the mocks.
- Using real `sleep` calls, making tests slow and flaky.
- Calling `app.listen` on import, which causes port conflicts.
- **Trade-off:** `node:test` avoids dependencies, while Jest and Vitest offer richer tooling and ecosystems.

### Top interview questions
**Q1. Jest, Vitest or `node:test`?**
Vitest for new TS and ESM projects. Jest for existing suites. `node:test` for a lean setup with no dependencies.

**Q2. What is the difference between a mock, stub and spy?**
A stub returns set values. A spy records calls. A mock adds expectations on how it's called.

**Q3. How do you test code that uses `setTimeout`?**
Use fake timers and advance time, which avoids real delays.

**Q4. How do you test an Express app without starting a server?**
Use Supertest, with `app` exported separately from `listen`.

**Q5. Why use Testcontainers?**
It runs real databases, so tests catch real SQL, migration and driver issues.

**Q6. How do you keep database tests isolated?**
Run each test in a rolled-back transaction, truncate tables, or use a separate schema per worker.

### Quick revision
`node:test` = built in. Vitest = modern. Jest = mature. Supertest for HTTP. Inject dependencies. Testcontainers for real databases. Mock only the edges.

---

# Part 10: Performance and Debugging

## 52. ⭐ Profiling

### Why it matters
Good engineers measure before they optimize. Interviewers want to hear which tool you'd use, and how to read its output.

### Explanation

#### `--inspect`
Connects Chrome DevTools or VS Code to a running Node process, for breakpoints and CPU and memory profiling.

```bash
node --inspect app.js                # start with the inspector on 127.0.0.1:9229
node --inspect-brk app.js            # pause on the first line
kill -SIGUSR1 <pid>                  # enable the inspector on a running process
```

Then open `chrome://inspect`. In production, never bind it to a public address, because the inspector allows remote code execution. Use an SSH tunnel or `kubectl port-forward`.

```javascript
// Capture a CPU profile in code
const inspector = require('node:inspector');
const fs = require('node:fs');

const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    setTimeout(() => {
      session.post('Profiler.stop', (err, { profile }) => {
        fs.writeFileSync('app.cpuprofile', JSON.stringify(profile));
        session.disconnect();
      });
    }, 30_000);
  });
});
// Load the .cpuprofile file into Chrome DevTools or VS Code.
```

```bash
# Built-in options without code changes
node --cpu-prof --cpu-prof-dir=./profiles app.js
node --prof app.js && node --prof-process isolate-*.log > report.txt
```

#### Flame graphs
A picture of where CPU time goes. **Width** is time spent. **Height** is call depth. Look for wide bars at the top of a stack.

```text
│ JSON.stringify ████████████████████████  ← wide and on top: the hot spot
│ serializeReport ███████████████████████████
│ buildReport ████████████████████████████████████
│ handler ████████████████████████████████████████
└────────────────────────────────────────────────
  Width = share of CPU time. X-axis isn't a timeline.
```

```bash
npx 0x app.js                       # produces an interactive flame graph
```

On Linux, `perf` with `--perf-basic-prof` captures native and JS frames together.

#### Clinic.js
A suite that diagnoses common Node problems. It isn't as actively developed as it was, but its tools are still widely known.

```bash
npx clinic doctor -- node app.js     # overall diagnosis
npx clinic flame -- node app.js      # CPU flame graph
npx clinic bubbleprof -- node app.js # async delays
npx clinic heapprofiler -- node app.js
```

#### `perf_hooks`
Built-in timing APIs for code you control.

```javascript
const {
  performance,
  PerformanceObserver,
  monitorEventLoopDelay,
  createHistogram,
} = require('node:perf_hooks');

// Measure a block
performance.mark('start');
await loadReport();
performance.mark('end');
performance.measure('load-report', 'start', 'end');

// Receive measurements and GC pauses
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(entry.entryType, entry.name, entry.duration);
  }
});
observer.observe({ entryTypes: ['measure', 'gc'] });

// Wrap a function to time every call
const timed = performance.timerify(parseOrders);

// Event loop utilization: share of time doing work
const before = performance.eventLoopUtilization();
setTimeout(() => {
  const delta = performance.eventLoopUtilization(before);
  console.log(delta.utilization);   // near 1 means saturated
}, 1000);
```

### Real-time scenario
**Problem:** CPU sits at 90% after a release, and no one is sure why.
**Diagnosis:** A 30-second CPU profile from one pod shows 40% of the time in a logging serializer that deep-clones objects.
**Fix:** Log selected fields only, and use Pino's fast serializers. CPU falls to 45%.

### Common mistakes and trade-offs
- Optimizing without a profile.
- Profiling on a tiny dataset that doesn't match production.
- Leaving the inspector exposed.
- **Trade-off:** sampling profilers have low overhead but can miss short functions. Instrumentation is precise but adds cost.

### Top interview questions
**Q1. How do you profile a Node app in production?**
Capture a short CPU profile from one instance, with the inspector via a tunnel, `--cpu-prof`, or continuous profiling tools.

**Q2. How do you read a flame graph?**
Width is CPU time, and height is the call stack. Look for wide frames at the top.

**Q3. What is event loop utilization?**
The share of time the loop is doing work rather than waiting. High values mean it's saturated.

**Q4. What does `--inspect` risk in production?**
Whoever reaches the port can run code in the process. Never expose it publicly.

**Q5. High CPU, or high latency with low CPU: what's the difference?**
High CPU suggests compute work. High latency with low CPU suggests waiting: I/O, pool queues or locks.

**Q6. What does `--cpu-prof` do?**
It writes a V8 CPU profile on exit, without any code changes.

### Quick revision
Measure first. `--inspect` + DevTools, `--cpu-prof`, flame graphs (width = time), `perf_hooks`, ELU. Keep the inspector private.

---

## 53. ⭐ Debugging Memory Leaks

### Why it matters
"Your Node service's memory keeps growing. What do you do?" is one of the most common senior scenarios. You need a clear process.

### Explanation

**The process:**
1. Confirm a leak: does memory keep rising after GC?
2. Find out where: `heapUsed`, or `external`/`rss`?
3. Capture heap snapshots over time.
4. Compare them, and find what's growing.
5. Follow the retainers to the code.
6. Fix, then confirm in a load test.

```javascript
// Step 1: track memory over time
setInterval(() => {
  const m = process.memoryUsage();
  metrics.gauge('memory.rss', m.rss);
  metrics.gauge('memory.heapUsed', m.heapUsed);
  metrics.gauge('memory.external', m.external);
}, 15_000).unref();
```

| Pattern | Likely cause |
|---|---|
| Sawtooth, flat floor | Healthy GC |
| Sawtooth, rising floor | JS heap leak |
| `heapUsed` flat, `rss` rising | Buffers, native addons, fragmentation |
| Rises then levels off | Cache warming up, probably fine |

#### Heap snapshots
A snapshot records all live objects and their references. **Warning:** it pauses the process and needs lots of memory. Take it from a pod out of the load balancer if you can.

```javascript
const v8 = require('node:v8');

// Take a snapshot on demand by signal
process.on('SIGUSR2', () => {
  const file = v8.writeHeapSnapshot();
  logger.info({ file }, 'heap snapshot written');
});
```

```bash
# Built-in triggers
node --heapsnapshot-signal=SIGUSR2 app.js
node --heapsnapshot-near-heap-limit=2 --max-old-space-size=1024 app.js

# Copy from Kubernetes
kubectl cp orders-api-abc:/app/Heap.20260101.heapsnapshot ./snap1.heapsnapshot
```

**Three-snapshot technique** in Chrome DevTools (Memory tab):

1. Warm up the app, and take snapshot 1.
2. Run load, and take snapshot 2.
3. Run more load, and take snapshot 3.
4. Select snapshot 3 and use **Comparison** against snapshot 2, or "objects allocated between 1 and 2".
5. Sort by **Delta** or **Retained Size**.
6. Click an object, and read the **Retainers** panel.

```text
Constructor     # New   # Deleted   Delta     Retained
Object          45,210  120         +45,090   38 MB
IncomingMessage  3,002  10          +2,992    29 MB   ← requests should be freed

Retainers:
  IncomingMessage
    in requests of Map
      in pendingRequests in system / Context   ← points at the source
```

- **Shallow size:** the object's own memory.
- **Retained size:** what would be freed if the object were deleted. This is the useful number.

For a lighter option, the **sampling heap profiler** has much less overhead:

```javascript
const inspector = require('node:inspector/promises');
const session = new inspector.Session();
session.connect();

await session.post('HeapProfiler.startSampling');
// ...run for a few minutes...
const { profile } = await session.post('HeapProfiler.stopSampling');
```

#### Common leak patterns

```javascript
// 1. Map that only grows
const seen = new Map();
function track(req) { seen.set(req.id, req); }

// Fix: bounded LRU, or remove on finish
res.on('finish', () => seen.delete(req.id));

// 2. Listener added on each call
function subscribe(socket) {
  bus.on('event', (d) => socket.send(d));   // never removed
}

// 3. Interval that outlives its owner
class Poller {
  start() { this.timer = setInterval(() => this.poll(), 1000); }
  stop() { clearInterval(this.timer); }     // must actually be called
}

// 4. Promises that never settle keep their closures alive
await new Promise(() => {});

// 5. A retained closure scope
function handler() {
  const big = Buffer.alloc(10_000_000);
  const unused = () => big.length;   // shares a scope with the closure below
  return () => 'hello';              // can keep `big` alive
}

// 6. Buffers: a small slice retains the whole parent
const header = bigBuffer.subarray(0, 10);
cache.set(id, header);               // retains bigBuffer
cache.set(id, Buffer.from(header));  // fixed: copy

// 7. Unbounded async queue
const queue = [];
app.post('/events', (req, res) => {
  queue.push(req.body);     // grows if consumers are slower than producers
  res.status(202).end();
});
```

### Real-time scenario
**Problem:** Memory grows by 50 MB per hour, and pods restart every 12 hours.
**Diagnosis:** A three-snapshot comparison shows thousands of `Socket` objects retained by a metrics label map. Each request added a unique URL with an order ID as a label.
**Fix:** Use route templates, such as `/orders/:id`, for labels. This also repairs the high-cardinality metrics cost.

### Common mistakes and trade-offs
- Taking one snapshot and guessing.
- Snapshotting a busy production pod and causing an outage.
- Blaming a leak when it's a cache filling up.
- **Trade-off:** snapshots are complete but expensive. Sampling is safe but less detailed.

### Top interview questions
**Q1. How do you debug a memory leak in production?**
Confirm with metrics, reproduce if possible, compare heap snapshots, find retainers, fix and verify under load.

**Q2. What is retained size?**
The memory freed if that object is removed, including everything only it keeps alive.

**Q3. How do you tell a heap leak from a native one?**
Compare `heapUsed`, `external` and `rss`. If only `rss` or `external` grows, look at Buffers and addons.

**Q4. What is the three-snapshot technique?**
Take snapshots after warm-up, after load and after more load. Objects that keep growing are leak suspects.

**Q5. What is the risk of taking a heap snapshot?**
It pauses the process and uses a lot of memory, which can trigger a crash under a tight limit.

**Q6. Can you capture a heap dump on OOM?**
Yes, with `--heapsnapshot-near-heap-limit`, though it needs headroom.

### Quick revision
Rising floor = leak. Three snapshots, compare, follow retainers. Check `heapUsed` vs `external`. Usual suspects: Maps, listeners, timers, closures, unbounded queues.

---

## 54. Debugging Production Issues

### Why it matters
Senior engineers are hired to resolve incidents. Interviewers look for a calm method: observe, narrow down, mitigate, then find the root cause.

### Explanation

**Method:** mitigate first (roll back, scale up, shed load), then investigate.

```mermaid
flowchart TD
    A["Slow service"] --> B{"CPU high?"}
    B -- Yes --> C["CPU profile: find the hot code"]
    B -- No --> D{"Event loop delay high?"}
    D -- Yes --> E["Blocking sync work or GC pauses"]
    D -- No --> F{"Which span is slow in the trace?"}
    F --> G["Database: slow query or pool wait"]
    F --> H["Upstream: dependency latency"]
    F --> I["Thread pool saturation"]
```

#### Blocked event loops
**Signs:** all routes get slower together, health checks time out, and event loop delay spikes.

```javascript
const { monitorEventLoopDelay } = require('node:perf_hooks');

const delay = monitorEventLoopDelay({ resolution: 10 });
delay.enable();

setInterval(() => {
  const p99 = delay.percentile(99) / 1e6;
  if (p99 > 100) logger.warn({ p99 }, 'event loop is blocked');
  delay.reset();
}, 5000).unref();

// Show sync calls during development
// node --trace-sync-io app.js

// Find which route is responsible
app.use((req, res, next) => {
  const start = performance.now();
  next();
  const syncMs = performance.now() - start;   // sync time in this tick
  if (syncMs > 50) logger.warn({ url: req.url, syncMs }, 'long sync work');
});
```

Typical causes: `JSON.parse` on big payloads, `*Sync` calls, slow regex, large sorts, and sync crypto.

#### High CPU
**Signs:** CPU is near the limit and latency rises with traffic.

```bash
# 1. Which process and which threads?
top -H -p <pid>

# 2. Container throttling? In Kubernetes:
kubectl top pod orders-api-abc
cat /sys/fs/cgroup/cpu.stat        # look at nr_throttled

# 3. CPU profile from one instance
kubectl port-forward pod/orders-api-abc 9229:9229
# then record in chrome://inspect
```

```javascript
// GC can also be the cause: look for frequent long pauses
const { PerformanceObserver } = require('node:perf_hooks');

new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    metrics.histogram('gc.pause.ms', entry.duration);
  }
}).observe({ entryTypes: ['gc'] });
```

Container CPU limits can cause **throttling**: latency spikes while CPU graphs look normal.

#### Slow requests
**Signs:** a few routes are slow and the event loop is healthy, so the time is being spent waiting.

```javascript
// Time each dependency to see where it goes
async function timed(name, fn) {
  const start = performance.now();
  try {
    return await fn();
  } finally {
    metrics.histogram(`dependency.${name}.ms`, performance.now() - start);
  }
}

const user = await timed('db.getUser', () => repo.getUser(id));
const stock = await timed('inventory', () => inventory.get(sku));
```

Then check:

- **Traces:** which span is the longest? (Topic 65)
- **Database:** slow query log, `EXPLAIN ANALYZE`, locks.
- **Pool:** `pool.waitingCount` above 0.
- **Thread pool:** `UV_THREADPOOL_SIZE` and many file or `dns.lookup` calls.
- **Network:** DNS lookups, TLS setup, missing keep-alive.
- **Tail latency:** check p99, not averages.

```javascript
// Capture a diagnostic report on demand: stacks, handles, system limits
process.report.reportOnSignal = true;
process.report.signal = 'SIGUSR2';
process.report.writeReport('./report.json');

// See what is keeping the process alive
console.log(process.getActiveResourcesInfo());
```

### Real-time scenario
**Problem:** p99 latency triples at 9 AM daily. CPU is 40%, and the database is healthy.
**Diagnosis:** Traces show a gap before the DB span, and `pool.waitingCount` peaks. A scheduled report holds connections for 20 s.
**Fix:** Move the report to a read replica with its own pool. Add an alert on pool wait time.

### Common mistakes and trade-offs
- Debugging before you mitigate.
- Looking at averages, which hide tail latency.
- Changing several things at once, so nothing is clear.
- **Trade-off:** more instrumentation costs a bit of overhead, but saves hours in each incident.

### Top interview questions
**Q1. An API has become slow. How do you investigate?**
Check recent changes and scope, then metrics: CPU, event loop delay, memory. Use traces to find the slow span and profile if CPU-bound.

**Q2. How do you recognize a blocked event loop?**
All routes slow together, health checks fail, and event loop delay spikes while the database looks fine.

**Q3. Low CPU but high latency: what are the causes?**
Waiting on the DB, pool, upstream services, locks or the libuv thread pool. CPU throttling also fits.

**Q4. Why use p99 instead of an average?**
Averages hide slow outliers, and a page with many calls hits those outliers often.

**Q5. What are Node diagnostic reports?**
JSON files with stacks, heap data, handles and system details, created on a crash or signal.

**Q6. What do you do first in an incident?**
Restore service. Roll back or scale up, and keep evidence. Investigate the root cause after that.

### Quick revision
Mitigate first. All routes slow → blocked loop. High CPU → profile. Low CPU, slow → trace the waits. Watch p99, pool waits and throttling.

---

## 55. Performance Optimization

### Why it matters
After measuring, you need to know what really helps. Interviews look for practical wins, and proof through load testing.

### Explanation

#### JSON costs
`JSON.parse` and `JSON.stringify` are synchronous and run on the main thread. Big payloads block everyone.

```javascript
// Measure the cost
const big = Array.from({ length: 200_000 }, (_, i) => ({ id: i, name: `item-${i}` }));
console.time('stringify');
const text = JSON.stringify(big);
console.timeEnd('stringify');
console.time('parse');
JSON.parse(text);
console.timeEnd('parse');

// Fix 1: return less data. Select fields, and paginate.
// Fix 2: schema-based serialization, as used by Fastify
const fastJson = require('fast-json-stringify');
const stringify = fastJson({
  type: 'object',
  properties: { id: { type: 'integer' }, name: { type: 'string' } },
});

// Fix 3: stream large arrays with newline-delimited JSON
const { pipeline } = require('node:stream/promises');
app.get('/export', async (req, res) => {
  res.type('application/x-ndjson');
  await pipeline(
    db.streamOrders(),
    async function* (rows) {
      for await (const row of rows) yield JSON.stringify(row) + '\n';
    },
    res,
  );
});

// Fix 4: cache serialized strings for hot responses
const body = await redis.get('catalog:json');    // no parse and re-stringify
res.type('json').send(body);
```

#### Compression
Reduces bytes, but costs CPU. Compress at the CDN or reverse proxy if you can.

```javascript
const compression = require('compression');

app.use(compression({
  threshold: 1024,                    // skip small responses
  filter: (req, res) => {
    if (req.path.startsWith('/events')) return false;   // don't buffer SSE
    return compression.filter(req, res);
  },
}));
```

```javascript
const zlib = require('node:zlib');
const { pipeline } = require('node:stream/promises');

// Static assets: compress once at build time, with the highest setting
await pipeline(
  fs.createReadStream('app.js'),
  zlib.createBrotliCompress(),
  fs.createWriteStream('app.js.br'),
);
```

Don't compress images, video or zipped files. They're already compressed, so it only burns CPU.

#### Keep-alive
Reusing connections removes handshakes, often a big win for service-to-service traffic.

```javascript
const { Agent, setGlobalDispatcher } = require('undici');

setGlobalDispatcher(new Agent({
  connections: 100,            // per-origin connection cap
  keepAliveTimeout: 30_000,
  pipelining: 1,
}));

// AWS SDK v3 with a tuned agent
const { NodeHttpHandler } = require('@smithy/node-http-handler');
const https = require('node:https');

const s3 = new S3Client({
  requestHandler: new NodeHttpHandler({
    httpsAgent: new https.Agent({ keepAlive: true, maxSockets: 100 }),
  }),
});
```

Other wins, by impact:

```text
1. Remove work: fewer queries, smaller payloads, proper indexes
2. Do it less often: caching, batching
3. Do it concurrently: Promise.all with limits
4. Move it: background jobs, worker threads
5. Micro-optimize last, and only in hot paths
```

#### Load testing
Proves whether a change helped, and finds limits before users do.

```bash
# autocannon: quick HTTP benchmark from Node
npx autocannon -c 100 -d 30 http://localhost:3000/orders
```

```javascript
// k6: scripted scenarios with pass/fail thresholds
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // ramp up
    { duration: '5m', target: 100 },   // hold
    { duration: '1m', target: 0 },     // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  const res = http.get('https://staging.example.com/orders');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}
```

| Test | Purpose |
|---|---|
| Load | Handles expected traffic |
| Stress | Where and how it breaks |
| Spike | Sudden jumps |
| Soak | Leaks over hours |

Run tests on production-like data, from a separate machine, and watch p95/p99 along with throughput.

### Real-time scenario
**Problem:** A product list endpoint handles 300 requests per second, and the target is 2,000.
**Diagnosis:** Profiling shows most time serializing 500-field objects, and a new TLS connection for each pricing call.
**Fix:** Select 12 fields, cache the serialized response for 30 s, and enable keep-alive. A k6 run confirms 2,400 requests per second at a p99 of 180 ms.

### Common mistakes and trade-offs
- Compressing in Node when a CDN could do it.
- Benchmarking on localhost with a tiny dataset.
- Reporting average latency only.
- **Trade-off:** caching and compression speed up responses, but add memory, CPU and staleness costs.

### Top interview questions
**Q1. How do you improve API throughput?**
Profile, then remove work: fewer queries, smaller payloads, caching, connection reuse and offloading CPU work.

**Q2. Why can large JSON hurt a Node server?**
Parse and stringify are synchronous, so they block the loop for all requests.

**Q3. Should compression be done in Node?**
Usually at the CDN or proxy, which frees Node's CPU. Node can do it for simple deployments.

**Q4. What is the difference between load, stress and soak tests?**
Load checks expected traffic. Stress finds the limit. Soak runs a long time and exposes leaks.

**Q5. Which metrics matter in a load test?**
Throughput, p95 and p99 latency, error rate, CPU, memory and event loop delay.

**Q6. Is `autocannon` or `k6` better?**
`autocannon` for quick endpoint tests. `k6` for scripted scenarios and CI thresholds.

### Quick revision
Remove work first. Big JSON blocks. Compress at the edge. Reuse connections. Prove it with load tests, looking at p99.

---

# Part 11: Architecture and Scaling

## 56. ⭐ Scaling Node.js

### Why it matters
"How would you scale this service to 10× traffic?" comes up in nearly every senior interview. A strong answer finds the bottleneck first, then scales the right layer.

### Explanation

#### Vertical vs horizontal
**Vertical** means a bigger machine. **Horizontal** means more instances. One Node process uses one core for JS, so extra cores only help if you run more processes.

| | Vertical | Horizontal |
|---|---|---|
| How | More CPU and RAM | More instances |
| Limit | Largest machine | Mostly your shared dependencies |
| Failure | Single point of failure | Survives instance loss |
| Code changes | None | Must be stateless |

```bash
# Vertical: more heap for one process
node --max-old-space-size=4096 app.js

# Horizontal on one machine
pm2 start app.js -i max

# Horizontal across machines
kubectl scale deployment orders-api --replicas=10
```

Scaling the API doesn't help if the database is the bottleneck. More pods can even make it worse by opening more connections. Typical order of work:

```text
1. Fix slow queries and add indexes
2. Add caching
3. Scale the app horizontally
4. Add read replicas
5. Move slow work to queues
6. Partition or shard data
```

#### Statelessness
Any instance must be able to serve any request. Move local state to shared services.

```javascript
// Stateful: breaks with 2+ instances
const sessions = new Map();
const uploads = './uploads';
let requestCount = 0;

// Stateless: state lives outside the process
await redis.set(`session:${id}`, JSON.stringify(data), { EX: 1800 });
await s3.send(new PutObjectCommand({ Bucket: 'media', Key: key, Body: file }));
await redis.incr('request:count');
```

| Local state | Move it to |
|---|---|
| Sessions | Redis or signed tokens |
| Uploaded files | S3 |
| In-memory cache | Redis, or accept per-instance copies |
| Cron jobs | One scheduler or a queue |
| WebSocket rooms | Redis Pub/Sub backplane |

#### Load balancing
A load balancer spreads traffic across healthy instances.

| Algorithm | How it works | Good for |
|---|---|---|
| Round robin | Takes turns | Similar requests |
| Least connections | Picks the least busy | Mixed request times |
| IP hash | Same client, same server | Simple stickiness |
| Weighted | More traffic to bigger servers | Mixed instance sizes |

```nginx
upstream api {
  least_conn;
  server 10.0.0.1:3000 max_fails=3 fail_timeout=30s;
  server 10.0.0.2:3000 max_fails=3 fail_timeout=30s;
  keepalive 64;
}

server {
  listen 443 ssl;

  location / {
    proxy_pass http://api;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

```javascript
// Behind a proxy, tell Express how many hops to trust,
// so req.ip is the real client and not the load balancer.
app.set('trust proxy', 1);
```

L4 balancers route TCP connections. L7 balancers read HTTP, so they can route by path and header, and they balance per request.

#### Sticky sessions
The load balancer sends a client to the same instance each time. It's a workaround for local state, and it causes uneven load and lost sessions when an instance dies.

```nginx
upstream realtime {
  ip_hash;      # same client IP goes to the same server
  server 10.0.0.1:3000;
  server 10.0.0.2:3000;
}
```

Prefer shared state. Stickiness is reasonable for Socket.IO while it uses HTTP polling.

### Real-time scenario
**Problem:** Traffic is expected to grow 10× for a sale, and the team plans to add 10× the pods.
**Diagnosis:** A load test shows PostgreSQL connections run out at 3×. The API was never the bottleneck.
**Fix:** Cache the product catalog, add a read replica, put PgBouncer in front, and queue order emails. Then scale pods and pre-warm before the event.

### Common mistakes and trade-offs
- Scaling the app when the database is the limit.
- Autoscaling only on CPU when the service is I/O-bound.
- Keeping state in memory, then adding sticky sessions to hide it.
- **Trade-off:** horizontal scaling gives resilience, but needs shared state, more network hops and more operational work.

### Top interview questions
**Q1. How do you scale a Node.js application?**
Find the bottleneck, make the service stateless, run one process per core or pod behind a load balancer, cache reads, and queue slow work.

**Q2. What does stateless mean?**
No request depends on memory or disk of one instance. Shared state lives in external stores.

**Q3. Why avoid sticky sessions?**
They cause uneven load, make failover lose state, and hide a design problem.

**Q4. What is the difference between L4 and L7 load balancing?**
L4 routes TCP connections. L7 understands HTTP and can route and balance per request.

**Q5. How do you handle 10× traffic for a known event?**
Load test first, pre-scale, cache aggressively, queue non-critical work, and prepare rate limits and feature flags.

**Q6. Why can adding pods make things worse?**
Each pod adds connections and load on shared dependencies, which can overload the database.

### Quick revision
Find the bottleneck first. Stateless + horizontal. Cache, replicas, queues. Avoid sticky sessions. Set `trust proxy` behind a load balancer.

---

## 57. ⭐ Monolith vs Microservices

### Why it matters
This is a core architecture question. A senior answer doesn't favour microservices by default. It weighs team size, domain and operating cost.

### Explanation

| | Monolith | Modular monolith | Microservices |
|---|---|---|---|
| Deploy | One unit | One unit | Many units |
| Calls | In-process | In-process, through module APIs | Network |
| Data | One database | One database, owned schemas | Database per service |
| Scaling | Whole app | Whole app | Per service |
| Complexity | Low | Medium | High |
| Best for | Early products | Growing teams | Large organizations |

#### Monolith
One codebase and one deployment. Simple to build, test and debug. Problems appear when many teams change the same code.

```javascript
// One process: a function call and a single transaction
async function placeOrder(order) {
  return db.transaction(async (tx) => {
    await inventory.reserve(tx, order.items);
    await payments.charge(tx, order);
    return orders.create(tx, order);
  });
}
```

#### Modular monolith
One deployment, with strict internal boundaries. Modules talk through public APIs only. It's often the best starting point, and it makes later extraction much easier.

```text
src/modules/
  orders/
    index.js          ← the only public entry
    orders.service.js
    orders.repo.js
  inventory/
    index.js
    inventory.service.js
  payments/
    index.js
```

```javascript
// Allowed: through the module's public API
const inventory = require('../inventory');
await inventory.reserve(items);

// Not allowed: reaching into another module's internals
// const repo = require('../inventory/inventory.repo');
```

Enforce this with ESLint `no-restricted-imports` or `dependency-cruiser`, so the rule survives deadlines.

#### Service boundaries
Split by **business capability**, not by technical layer. Each service owns its data, and nobody else touches its tables.

```text
Good: Orders, Payments, Inventory, Notifications
Bad:  Database service, Validation service, Utils service
```

```javascript
// Bad: reaches into another service's database
const stock = await inventoryDb.query('SELECT quantity FROM stock WHERE sku = $1', [sku]);

// Good: uses its API or subscribes to its events
const stock = await inventoryClient.getStock(sku);
```

Warning signs of a poor boundary: two services must always deploy together, or one request needs a chain of many synchronous calls. That's a **distributed monolith**: all the cost of microservices and none of the benefit.

#### API gateway
One entry point in front of services. It handles shared concerns like routing, auth, rate limiting and TLS.

```mermaid
flowchart LR
    C["Clients"] --> G["API Gateway<br/>auth, rate limit, routing"]
    G --> O["Orders"]
    G --> P["Payments"]
    G --> U["Users"]
```

```javascript
// A minimal gateway, to show the idea
const { createProxyMiddleware } = require('http-proxy-middleware');

app.use(helmet());
app.use(rateLimiter);
app.use(authenticate);

app.use('/orders', createProxyMiddleware({ target: 'http://orders:3000', changeOrigin: true }));
app.use('/payments', createProxyMiddleware({ target: 'http://payments:3000', changeOrigin: true }));
```

In production, teams usually use Kong, AWS API Gateway, Envoy or similar. Keep business logic out of the gateway.

#### BFF
Backend for Frontend: a small server for one client type. It combines calls and shapes data to fit that client. Node is a natural fit because the work is mostly I/O.

```javascript
// Mobile BFF: one small response instead of three large ones
app.get('/mobile/home', async (req, res) => {
  const [user, orders, offers] = await Promise.allSettled([
    usersClient.get(req.user.id),
    ordersClient.recent(req.user.id, { limit: 3 }),
    offersClient.forUser(req.user.id),
  ]);

  res.json({
    name: user.value?.firstName,
    recentOrders: orders.value?.map((o) => ({ id: o.id, status: o.status })) ?? [],
    offer: offers.value?.[0] ?? null,      // optional: page still works without it
  });
});
```

### Real-time scenario
**Problem:** A 10-person startup splits into 14 microservices. Simple features need 5 PRs, and local setup takes a day.
**Diagnosis:** Service boundaries were technical, and there were more services than teams could run.
**Fix:** Merge them into a modular monolith with clear modules. Extract only payments, which has distinct compliance and scaling needs.

### Common mistakes and trade-offs
- Starting with microservices before the domain is understood.
- Sharing a database between services.
- Synchronous call chains: availability multiplies down, so five services at 99.9% give about 99.5%.
- **Trade-off:** microservices buy team independence and targeted scaling at the price of network failures, eventual consistency and operating cost.

### Top interview questions
**Q1. When do you choose microservices?**
When teams need independent deployment, parts scale very differently, and you have the platform maturity to run them.

**Q2. What is a modular monolith?**
One deployable app with strict module boundaries. It gives simple operations with a clean path to extraction.

**Q3. How do you define service boundaries?**
Around business capabilities and data ownership, using domain-driven design ideas such as bounded contexts.

**Q4. What is a distributed monolith?**
Services that must change and deploy together, so you pay microservice costs without gaining independence.

**Q5. What does an API gateway do?**
It's a single entry point for routing, auth, rate limits and TLS. It shouldn't hold business logic.

**Q6. What is a BFF?**
A backend for one client type. It aggregates calls and shapes responses for that client.

**Q7. How do you migrate from monolith to microservices?**
Use the **strangler fig** pattern: route one capability at a time to a new service, then retire the old code.

### Quick revision
Start modular. Split by business capability. Database per service. Gateway for shared concerns, BFF per client. Beware the distributed monolith.

---

## 58. Distributed Patterns

### Why it matters
Across services, you lose single database transactions. These patterns keep data correct when the system is split.

### Explanation

#### Saga
A saga is a chain of local transactions. If a step fails, **compensating actions** undo earlier steps.

| | Orchestration | Choreography |
|---|---|---|
| Control | Central coordinator | Services react to events |
| Visibility | Easy to follow | Harder to trace |
| Coupling | To the orchestrator | Through events |
| Best for | Complex flows | Simple flows |

```javascript
// Orchestrated saga
class OrderSaga {
  async run(order) {
    const completed = [];

    const steps = [
      { name: 'reserve', run: () => inventory.reserve(order), undo: () => inventory.release(order) },
      { name: 'charge', run: () => payments.charge(order), undo: () => payments.refund(order) },
      { name: 'ship', run: () => shipping.create(order), undo: () => shipping.cancel(order) },
    ];

    for (const step of steps) {
      try {
        await step.run();
        await sagaLog.record(order.id, step.name, 'done');   // survives a crash
        completed.push(step);
      } catch (err) {
        for (const done of completed.reverse()) {
          await done.undo();    // compensations must be idempotent and retried
        }
        throw err;
      }
    }
  }
}
```

```javascript
// Choreography: each service reacts to events
bus.on('order.created', async (e) => {
  try {
    await inventory.reserve(e);
    await publish('stock.reserved', e);
  } catch {
    await publish('stock.failed', e);
  }
});

bus.on('stock.reserved', (e) => payments.charge(e));
bus.on('payment.failed', (e) => inventory.release(e));
```

For durable workflows, tools like **Temporal** or AWS Step Functions handle state and retries for you.

#### Outbox
The **dual-write problem**: saving to the database and publishing to a broker are two operations, and a crash between them loses the event.

```javascript
// Broken: a crash after commit means the event is never sent
await db.orders.insert(order);
await kafka.send({ topic: 'orders', messages: [{ value: JSON.stringify(order) }] });

// Outbox: event and data commit together
await withTransaction(async (tx) => {
  await tx.query('INSERT INTO orders (id, total) VALUES ($1, $2)', [order.id, order.total]);
  await tx.query(
    `INSERT INTO outbox (id, topic, payload) VALUES ($1, $2, $3)`,
    [randomUUID(), 'order.created', JSON.stringify(order)],
  );
});

// A relay publishes pending events. SKIP LOCKED lets several relays run safely.
async function relay() {
  await withTransaction(async (tx) => {
    const { rows } = await tx.query(`
      SELECT * FROM outbox WHERE published_at IS NULL
      ORDER BY created_at LIMIT 100
      FOR UPDATE SKIP LOCKED
    `);

    for (const event of rows) {
      await kafka.send({ topic: event.topic, messages: [{ key: event.id, value: event.payload }] });
      await tx.query('UPDATE outbox SET published_at = now() WHERE id = $1', [event.id]);
    }
  });
}
```

Delivery is at-least-once, so consumers must be idempotent (Topic 40). **Debezium** can replace the polling relay by reading the database's change log.

#### CQRS
Command Query Responsibility Segregation separates the **write model** from the **read model**, so each can be optimized on its own.

```mermaid
flowchart LR
    C["Commands"] --> W["Write model<br/>PostgreSQL"]
    W --> E["Events"]
    E --> R["Read model<br/>Elasticsearch"]
    Q["Queries"] --> R
```

```javascript
// Command: enforces rules on normalized tables
async function placeOrder(cmd) {
  const order = await orderRepo.create(cmd);
  await publish('order.placed', order);
  return order.id;
}

// Projection: keeps a denormalized read model up to date
bus.on('order.placed', async (e) => {
  await search.index({
    index: 'order-summaries',
    id: e.id,
    document: { customerName: e.customerName, total: e.total, status: e.status },
  });
});

// Query: one fast lookup, no joins
const getOrders = (customer) =>
  search.search({ index: 'order-summaries', query: { match: { customerName: customer } } });
```

The read side lags the write side. A user may not see their own change at once, so read from the write model right after a write, or update the UI optimistically.

#### Event-driven architecture
Services announce facts, and others react, without the publisher knowing who's listening.

```javascript
// A good event: past tense, versioned, with a unique ID
const event = {
  id: randomUUID(),
  type: 'order.placed',
  version: 1,
  occurredAt: new Date().toISOString(),
  data: { orderId: 'o1', customerId: 'c1', total: 4999 },
};

// Publisher doesn't know about email, analytics or fraud
await publish(event);
```

- **Event notification:** a small message saying what happened.
- **Event-carried state transfer:** the event has all data consumers need.
- **Event sourcing:** events are the source of truth, and state is rebuilt from them.

```javascript
// Event sourcing: replay events to get the current state
function balance(events) {
  return events.reduce((total, e) => {
    if (e.type === 'deposited') return total + e.amount;
    if (e.type === 'withdrawn') return total - e.amount;
    return total;
  }, 0);
}
```

### Real-time scenario
**Problem:** Orders are saved, but customers sometimes get no confirmation email.
**Diagnosis:** The service committed, then crashed or lost the broker connection before publishing.
**Fix:** Add a transactional outbox with a relay, and make the email consumer idempotent by event ID.

### Common mistakes and trade-offs
- Using two-phase commit across services, which blocks and scales poorly.
- Compensations that aren't idempotent.
- Using CQRS or event sourcing for simple CRUD.
- **Trade-off:** these patterns give availability and loose coupling, but consistency becomes eventual and debugging gets harder.

### Top interview questions
**Q1. What is a saga?**
A sequence of local transactions with compensating actions, used instead of a distributed transaction.

**Q2. Orchestration or choreography?**
Orchestration for complex flows that need visibility. Choreography for simple flows with few steps.

**Q3. What problem does the outbox pattern solve?**
The dual-write problem. The event commits with the data, so it can't be lost.

**Q4. What is CQRS?**
Separate models for writes and reads, each optimized for its job.

**Q5. What is the difference between event-driven and event sourcing?**
Event-driven uses events to communicate. Event sourcing stores events as the source of truth.

**Q6. How do you handle eventual consistency in the UI?**
Show pending states, read your own writes from the primary, and use optimistic updates.

### Quick revision
Saga = local steps + compensation. Outbox = no lost events. CQRS = separate read and write. Consumers must be idempotent.

---

## 59. ⭐ Design Patterns in Node

### Why it matters
Interviewers want patterns tied to Node specifics, such as the module cache, rather than textbook definitions.

### Explanation

#### Singleton via module cache
Modules are cached, so exporting an instance gives one shared object per process, with no special class.

```javascript
// db.js
const { Pool } = require('pg');
module.exports = new Pool({ connectionString: process.env.DATABASE_URL });

// a.js and b.js
const db = require('./db');   // both get the same pool
```

Limits:
- It's per process, so each cluster worker and worker thread gets its own.
- Two copies of a package in `node_modules` give two instances.
- Different path casing can create separate cache entries on some file systems.
- Global singletons make testing harder. Injection is a good alternative.

#### Factory
A function that picks or builds an object, so callers don't depend on the concrete class.

```javascript
class StripeGateway { charge(amount) { /* ... */ } }
class RazorpayGateway { charge(amount) { /* ... */ } }

function createPaymentGateway(region, config) {
  const gateways = {
    US: () => new StripeGateway(config.stripe),
    IN: () => new RazorpayGateway(config.razorpay),
  };

  const create = gateways[region];
  if (!create) throw new Error(`unsupported region: ${region}`);
  return create();
}

const gateway = createPaymentGateway(user.region, config);
await gateway.charge(4999);
```

Factory functions also give real privacy through closures:

```javascript
function createCounter() {
  let count = 0;
  return { increment: () => ++count };
}
```

#### Dependency injection
A module receives what it needs, rather than importing it. That makes testing and swapping simple.

```javascript
// Hard to test: tied to real modules
const db = require('./db');
const mailer = require('./mailer');

async function registerBad(user) {
  await db.users.insert(user);
  await mailer.send(user.email);
}

// Injected
function createUserService({ userRepo, mailer, logger }) {
  return {
    async register(user) {
      const saved = await userRepo.insert(user);
      await mailer.sendWelcome(saved.email);
      logger.info({ id: saved.id }, 'user registered');
      return saved;
    },
  };
}

// Composition root: all wiring happens in one place
const userService = createUserService({
  userRepo: createUserRepo(pool),
  mailer: createMailer(config.mail),
  logger,
});

// Test: no module mocking
const service = createUserService({
  userRepo: { insert: async (u) => ({ id: 1, ...u }) },
  mailer: { sendWelcome: async () => {} },
  logger: { info() {} },
});
```

Containers such as `awilix`, or NestJS's built-in one, automate this for bigger apps.

#### Repository
Hides data access behind a business-friendly interface. Services don't know about SQL.

```javascript
function createOrderRepo(pool) {
  const toDomain = (row) => ({
    id: row.id,
    customerId: row.customer_id,
    total: Number(row.total),
  });

  return {
    async findById(id) {
      const { rows } = await pool.query('SELECT * FROM orders WHERE id = $1', [id]);
      return rows[0] ? toDomain(rows[0]) : null;
    },
    async findPendingOlderThan(date) {
      const { rows } = await pool.query(
        `SELECT * FROM orders WHERE status = 'pending' AND created_at < $1`,
        [date],
      );
      return rows.map(toDomain);
    },
  };
}

// In-memory version for fast tests
function createFakeOrderRepo(seed = []) {
  const orders = new Map(seed.map((o) => [o.id, o]));
  return {
    findById: async (id) => orders.get(id) ?? null,
    findPendingOlderThan: async () => [...orders.values()],
  };
}
```

#### Middleware
A **chain of responsibility**: each step handles a request and passes it on. It isn't only for HTTP.

```javascript
// Generic pipeline, also usable for jobs or message handlers
function compose(middlewares) {
  return function run(context) {
    let index = -1;

    function dispatch(i) {
      if (i <= index) return Promise.reject(new Error('next() called twice'));
      index = i;
      const fn = middlewares[i];
      if (!fn) return Promise.resolve();
      return Promise.resolve(fn(context, () => dispatch(i + 1)));
    }

    return dispatch(0);
  };
}

const handleMessage = compose([
  async (ctx, next) => {
    const start = Date.now();
    await next();
    console.log(`handled in ${Date.now() - start} ms`);
  },
  async (ctx, next) => {
    ctx.payload = JSON.parse(ctx.raw);
    await next();
  },
  async (ctx) => {
    console.log('processing', ctx.payload.id);
  },
]);

await handleMessage({ raw: '{"id": 1}' });
```

### Real-time scenario
**Problem:** Tests take 10 minutes and often fail, because they all connect to a shared database.
**Diagnosis:** Services imported `db` directly, so nothing could be replaced.
**Fix:** Add factory-based injection and a repository layer. Unit tests use fakes and finish in seconds, and a smaller integration suite uses a real database.

### Common mistakes and trade-offs
- Adding patterns where a simple function is enough.
- Repositories that just mirror the ORM, adding code but no value.
- Storing request-specific data on singletons.
- **Trade-off:** patterns add indirection for flexibility, so use them where change or testing justifies it.

### Top interview questions
**Q1. How do you implement a Singleton in Node?**
Export an instance. Module caching gives one object per process.

**Q2. When does a module singleton break?**
Across processes and threads, with duplicated packages, or when paths resolve differently.

**Q3. Why use dependency injection in Node?**
Code is more testable and loosely coupled, with no need to patch modules.

**Q4. What is the Repository pattern?**
A data-access abstraction with domain methods. It keeps SQL or ORM details out of business logic.

**Q5. What pattern does Express middleware use?**
Chain of Responsibility.

**Q6. Which pattern does EventEmitter use?**
Observer. Listeners subscribe to named events.

### Quick revision
Singleton = module cache. Factory = choose at runtime. DI = pass dependencies in. Repository = hide data access. Middleware = chain of responsibility.

---

## 60. Project Structure

### Why it matters
Structure decides how easily a codebase grows. Interviewers ask this to see how you think about dependencies and separation of concerns.

### Explanation

#### Layered architecture
Each layer has one job, and calls go downward only.

```mermaid
flowchart TD
    A["Routes and controllers<br/>HTTP in and out"] --> B["Services<br/>business rules"]
    B --> C["Repositories<br/>data access"]
    C --> D["Database"]
```

```text
src/
  orders/                  ← organized by feature
    orders.routes.js
    orders.controller.js
    orders.service.js
    orders.repository.js
    orders.schema.js
    orders.test.js
  users/
    ...
  shared/
    errors.js
    logger.js
  config.js
  app.js                   ← builds the Express app
  server.js                ← starts listening
```

Group by **feature**, not by type. A change to orders then stays in one folder, rather than touching `controllers/`, `services/` and `models/`.

```javascript
// Controller: only HTTP concerns
async function createOrder(req, res) {
  const order = await orderService.place(req.user.id, req.body);
  res.status(201).json(order);
}

// Service: business rules, and no req or res
async function place(userId, input) {
  if (input.items.length === 0) throw new ValidationError('order is empty');
  const total = priceOrder(input.items);
  return orderRepo.create({ userId, items: input.items, total });
}

// Repository: only persistence
function create(order) {
  return db.orders.insert(order);
}
```

If a service takes `req`, you can't reuse it in a job or CLI. That's the usual sign of a leaking layer.

#### Clean architecture
Dependencies point **inward**. Business logic knows nothing about Express or PostgreSQL. Also called Hexagonal or Ports and Adapters.

```mermaid
flowchart LR
    A["Adapters: HTTP, queue, CLI"] --> B["Use cases"]
    B --> C["Domain: entities and rules"]
    D["Adapters: PostgreSQL, Stripe"] -. "implements port" .-> B
```

```text
src/
  domain/          entities and rules, no dependencies
  application/     use cases, and port interfaces
  adapters/
    http/          Express
    persistence/   PostgreSQL
    payments/      Stripe
  main.js          wires everything
```

```typescript
// domain/order.ts: pure logic
export class Order {
  constructor(public id: string, private status: 'pending' | 'paid' | 'cancelled') {}

  cancel() {
    if (this.status === 'paid') throw new Error('paid orders need a refund');
    this.status = 'cancelled';
  }
}

// application/ports.ts: what the use case needs
export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

// application/cancelOrder.ts: no Express, no SQL
export function makeCancelOrder(repo: OrderRepository) {
  return async function cancelOrder(id: string) {
    const order = await repo.findById(id);
    if (!order) throw new Error('not found');
    order.cancel();
    await repo.save(order);
  };
}

// adapters/persistence/pgOrderRepository.ts implements the port
export class PgOrderRepository implements OrderRepository {
  constructor(private pool: Pool) {}
  async findById(id: string) { /* SQL */ return null; }
  async save(order: Order) { /* SQL */ }
}
```

| | Layered | Clean / Hexagonal |
|---|---|---|
| Effort | Low | Higher |
| Framework independence | Partial | Strong |
| Testing business logic | Good | Excellent |
| Best for | Most services | Complex domains, long-lived systems |

### Real-time scenario
**Problem:** A 3,000-line `routes.js` holds SQL, validation and payment rules, and a nightly job can't reuse any of it.
**Diagnosis:** There were no layers, so business logic depended on `req` and `res`.
**Fix:** Extract services and repositories, feature by feature, starting from the most-changed code. The job now calls the same service.

### Common mistakes and trade-offs
- Business logic inside controllers.
- A `utils` folder that becomes a dumping ground.
- Applying full clean architecture to a small CRUD service.
- **Trade-off:** more layers give isolation and testability, but add files and indirection. Match it to the domain's complexity.

### Top interview questions
**Q1. How do you structure a Node.js project?**
By feature, with controller, service and repository layers, and separate `app` and `server` files.

**Q2. What belongs in a controller?**
HTTP translation only: read input, call a service and format the response.

**Q3. What is clean architecture?**
Dependencies point inward, so the domain stays independent of frameworks and databases.

**Q4. What is a port and adapter?**
A port is an interface the core needs. An adapter implements it for a real tool.

**Q5. Why separate `app.js` and `server.js`?**
Tests can import the app without opening a port.

### Quick revision
Group by feature. Controller → service → repository. Services never see `req`. Clean architecture: dependencies point inward.

---

## 61. Serverless Node.js

### Why it matters
Lambda changes assumptions about state, connections and startup time. Node is one of its most used runtimes, so this comes up often.

### Explanation

#### Lambda
You deploy a handler. AWS runs it per event and bills per use. One execution environment handles **one request at a time**, and scaling means more environments.

```javascript
// Code outside the handler runs once per environment, then gets reused
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, GetCommand } = require('@aws-sdk/lib-dynamodb');

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));

exports.handler = async (event) => {
  const { Item } = await ddb.send(new GetCommand({
    TableName: process.env.TABLE,
    Key: { pk: event.pathParameters.id },
  }));

  if (!Item) return { statusCode: 404, body: '{"error":"not found"}' };
  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify(Item),
  };
};

// SQS batch: report only failed messages so the rest aren't retried
exports.sqsHandler = async (event) => {
  const failures = [];

  await Promise.all(event.Records.map(async (record) => {
    try {
      await handle(JSON.parse(record.body));
    } catch {
      failures.push({ itemIdentifier: record.messageId });
    }
  }));

  return { batchItemFailures: failures };
};
```

Things to remember:
- The environment freezes between calls. Background work can stall, so `await` everything before returning.
- Local disk (`/tmp`) and memory can persist between calls, but it's not guaranteed.
- Maximum run time is 15 minutes.
- More memory also gives more CPU.

| Good fit | Poor fit |
|---|---|
| Spiky or low traffic | Steady high traffic (cost) |
| Event processing | Long-running work |
| Scheduled tasks | WebSocket servers holding state |
| Webhooks | Very latency-sensitive paths |

#### Cold starts
A new environment must download code, start Node and run your top-level code. Later calls on that environment are **warm**.

```text
Cold: create environment → load code → run init code → run handler
Warm: run handler
```

```javascript
// 1. Import only what you need
const { S3Client } = require('@aws-sdk/client-s3');   // not the entire SDK

// 2. Load rarely used dependencies lazily
exports.handler = async (event) => {
  if (event.path === '/pdf') {
    const { renderPdf } = require('./pdf');
    return renderPdf(event);
  }
  return { statusCode: 200, body: 'ok' };
};
```

```bash
# 3. Bundle and tree-shake to reduce package size
npx esbuild src/handler.js --bundle --minify --platform=node \
  --target=node22 --external:@aws-sdk/* --outfile=dist/handler.js
```

Other options: **provisioned concurrency** keeps environments warm at a cost. Raising memory speeds up init, because it gives more CPU.

#### Connection handling
Each environment holds its own connection, so 1,000 concurrent functions means 1,000 database connections.

```javascript
const { Pool } = require('pg');

// Outside the handler, so warm calls reuse it
const pool = new Pool({
  host: process.env.RDS_PROXY_HOST,    // RDS Proxy pools for all environments
  max: 1,                              // one request at a time needs one connection
  idleTimeoutMillis: 60_000,
  connectionTimeoutMillis: 3_000,
});

exports.handler = async (event) => {
  const { rows } = await pool.query('SELECT * FROM orders WHERE id = $1', [event.id]);
  return rows[0];
};
```

Other protections:
- **Reserved concurrency** caps how many environments can run, protecting the database.
- **DynamoDB** or HTTP-based databases avoid connection limits.
- Put an **SQS** queue in front to smooth out spikes.

### Real-time scenario
**Problem:** During a marketing campaign, Lambda scales to 2,000 environments and PostgreSQL fails with `too many connections`.
**Diagnosis:** Each environment opened its own pool, with `max: 10`.
**Fix:** Add RDS Proxy, set `max: 1`, set reserved concurrency, and use SQS for non-urgent writes.

### Common mistakes and trade-offs
- Creating clients inside the handler, which adds latency on each call.
- Using `max: 10` pools in Lambda.
- Non-idempotent handlers. Async events can be delivered more than once.
- **Trade-off:** no servers to run and scale-to-zero, in exchange for cold starts, limits and higher cost at steady load.

### Top interview questions
**Q1. What is a cold start, and how do you reduce it?**
Time to create a new environment. Reduce it with small bundles, fewer imports, more memory or provisioned concurrency.

**Q2. Why do you create clients outside the handler?**
Init code runs once per environment, so warm calls reuse connections.

**Q3. How do you connect Lambda to a relational database?**
Through RDS Proxy or PgBouncer, with a small pool and reserved concurrency.

**Q4. When is serverless a bad choice?**
Steady high load, long tasks, strict latency, or persistent connections.

**Q5. Does one Lambda environment handle concurrent requests?**
No. One at a time, so concurrency means more environments.

**Q6. How do you handle partial failures in an SQS batch?**
Return `batchItemFailures`, so only failed messages are retried.

### Quick revision
Init outside the handler. Small bundles fight cold starts. One request per environment. RDS Proxy + `max: 1`. Await everything.

---

# Part 12: Deployment and Observability

## 62. Docker for Node.js

### Why it matters
Most Node services ship as containers. Interviewers look for small, secure images and correct signal handling.

### Explanation

#### Multi-stage builds
One stage builds, and a smaller one runs. Compilers and dev dependencies stay out of the final image.

```dockerfile
# Stage 1: install all dependencies and build
FROM node:22-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: production dependencies only
FROM node:22-slim AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev

# Stage 3: the final image
FROM node:22-slim
ENV NODE_ENV=production
WORKDIR /app
COPY --from=deps --chown=node:node /app/node_modules ./node_modules
COPY --from=build --chown=node:node /app/dist ./dist
COPY --chown=node:node package.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Layer caching:** copying `package*.json` before the source means `npm ci` only re-runs when dependencies change.

#### PID 1
The first process in a container is PID 1, which has special rules. The kernel doesn't apply default signal behaviour to it, and it must reap zombie child processes. Node isn't built to do that job.

```bash
# Option 1: Docker adds a tiny init process
docker run --init orders-api
```

```dockerfile
# Option 2: install tini in the image
RUN apt-get update && apt-get install -y --no-install-recommends tini \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "dist/server.js"]
```

#### Signals
`docker stop` sends `SIGTERM`, waits 10 seconds, and then sends `SIGKILL`. The form of `CMD` decides whether Node receives the signal.

```dockerfile
# Bad: shell form. /bin/sh is PID 1 and may not forward signals.
CMD node dist/server.js

# Bad: npm may not forward signals
CMD ["npm", "start"]

# Good: exec form runs node directly
CMD ["node", "dist/server.js"]
```

```javascript
process.on('SIGTERM', () => {
  console.log('SIGTERM received');
  server.close(() => process.exit(0));
});
```

Test it: `docker stop` should finish in well under 10 seconds with exit code 0. A result of 137 means it was killed.

#### Image size
Smaller images pull faster and have fewer vulnerabilities.

| Base | Rough size | Notes |
|---|---|---|
| `node:22` | ~1 GB | Full Debian, good for builds |
| `node:22-slim` | ~200 MB | Good default |
| `node:22-alpine` | ~150 MB | musl libc can break native modules |
| Distroless | ~150 MB | No shell, so harder to debug |

```text
# .dockerignore keeps junk and secrets out of the image
node_modules
.git
.env
*.log
coverage
dist
```

```bash
docker images orders-api
docker history orders-api        # which layer is big?
trivy image orders-api           # scan for vulnerabilities
```

Other good habits: run as non-root with `USER node`, pin versions or digests, never bake secrets into layers, and use `--mount=type=secret` for private registry tokens.

### Real-time scenario
**Problem:** An image is 1.8 GB, deploys take 6 minutes, and a security scan reports 400 vulnerabilities.
**Diagnosis:** It used the full `node` image, shipped dev dependencies, and copied `.git` and test data.
**Fix:** Multi-stage build on `node:22-slim` with `.dockerignore` and `npm ci --omit=dev`. The image falls to about 250 MB.

### Common mistakes and trade-offs
- `CMD ["npm", "start"]`, which swallows signals.
- Running as root.
- `COPY . .` before `npm ci`, which breaks the cache.
- **Trade-off:** Alpine and distroless are smaller, but musl issues and no shell can slow down debugging.

### Top interview questions
**Q1. How do you write a production Dockerfile for Node?**
Multi-stage build, slim base, `npm ci --omit=dev`, non-root user, exec-form `CMD`, and `.dockerignore`.

**Q2. Why not use `npm start` in Docker?**
It adds a process that may not forward signals, so Node gets killed without cleaning up.

**Q3. What is the PID 1 problem?**
PID 1 has special signal behaviour and must reap zombies. Use `--init` or `tini`.

**Q4. Why copy `package.json` first?**
Docker caches the dependency layer until those files change.

**Q5. Alpine or slim?**
Slim by default. Alpine uses musl, which can cause problems with native modules.

**Q6. How do you keep secrets out of images?**
Inject them at runtime, and use BuildKit secrets during builds. Layers keep whatever you copied.

### Quick revision
Multi-stage, slim, `npm ci --omit=dev`, `USER node`, exec-form `CMD`, `--init`, `.dockerignore`. Test `docker stop`.

---

## 63. Kubernetes Basics

### Why it matters
Most large Node deployments run on Kubernetes. You should be able to discuss how probes, limits and autoscaling work with Node's single-threaded model.

### Explanation

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api
spec:
  replicas: 3
  strategy:
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
  selector:
    matchLabels: { app: orders-api }
  template:
    metadata:
      labels: { app: orders-api }
    spec:
      terminationGracePeriodSeconds: 40
      containers:
        - name: api
          image: registry.example.com/orders-api:1.4.2
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef: { name: orders-db, key: url }
```

#### Probes
Topic 48 covers the handlers. Here's the config.

```yaml
startupProbe:              # protects slow boots
  httpGet: { path: /livez, port: 3000 }
  failureThreshold: 30
  periodSeconds: 2
livenessProbe:             # restarts a stuck container
  httpGet: { path: /livez, port: 3000 }
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3
readinessProbe:            # controls traffic
  httpGet: { path: /readyz, port: 3000 }
  periodSeconds: 5
  failureThreshold: 2
```

A blocked event loop also fails probes. A CPU-heavy task can get a healthy pod killed, so keep liveness tolerant.

#### Resource limits
- **Requests:** what the scheduler reserves.
- **Limits:** the maximum allowed.

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    memory: "768Mi"       # going over this causes an OOM kill
    # Many teams skip a CPU limit to avoid throttling
```

Node-specific points:

```yaml
env:
  - name: NODE_OPTIONS
    value: "--max-old-space-size=576"   # about 75% of the memory limit
```

- Over the **CPU limit**, the container is throttled, which appears as latency spikes.
- Over the **memory limit**, it's killed with exit code 137.
- A single Node process rarely uses much more than 1 CPU for JS, so a pod with a limit of 4 CPUs wastes capacity.

#### Autoscaling
The **Horizontal Pod Autoscaler** adds pods based on metrics. CPU is a poor signal for I/O-heavy services.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: orders-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: orders-api
  minReplicas: 3
  maxReplicas: 30
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 65 }
    - type: Pods
      pods:
        metric: { name: http_requests_per_second }
        target: { type: AverageValue, averageValue: "200" }
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300     # avoids flapping
```

For queue workers, **KEDA** scales on queue depth, and can scale down to zero.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: email-worker
spec:
  scaleTargetRef: { name: email-worker }
  minReplicaCount: 0
  maxReplicaCount: 50
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.ap-south-1.amazonaws.com/123456789012/email
        queueLength: "20"
```

Also use a **PodDisruptionBudget**, so node maintenance can't remove too many pods at once.

### Real-time scenario
**Problem:** Pods restart during traffic peaks, which reduces capacity and causes more restarts.
**Diagnosis:** A 1-second liveness timeout failed when the event loop was busy, and Kubernetes killed pods that were only slow.
**Fix:** Relax the liveness probe, move the CPU work off the main thread, and scale on request rate earlier.

### Common mistakes and trade-offs
- Setting heap size equal to the memory limit.
- Tight CPU limits on latency-sensitive pods.
- Checking dependencies in liveness.
- **Trade-off:** small pods scale smoothly but have more overhead. Larger pods waste capacity with one Node process.

### Top interview questions
**Q1. What is the difference between requests and limits?**
Requests guide scheduling. Limits cap use. CPU is throttled at the limit, and memory causes a kill.

**Q2. How do you size memory for a Node pod?**
Set the limit from measured usage, and set the heap to about 75% of it to leave room for native memory.

**Q3. What should Node autoscale on?**
Request rate or latency for I/O-heavy APIs, CPU for compute-heavy ones, queue depth for workers.

**Q4. Should you use cluster in Kubernetes?**
Usually not. Run one process per pod and scale replicas.

**Q5. How do you get zero-downtime rolling updates?**
Use `maxUnavailable: 0`, readiness probes, graceful shutdown and backward-compatible changes.

**Q6. What is KEDA?**
Event-driven autoscaling for Kubernetes, which can scale on queue length, including down to zero.

### Quick revision
Probes: startup, liveness, readiness. Heap ≈ 75% of memory limit. CPU throttles, memory kills. Scale on the right metric. One process per pod.

---

## 64. CI/CD

### Why it matters
Teams that release safely can release often. Interviewers ask how you ship with low risk, and how quickly you can undo a bad deploy.

### Explanation

#### Pipelines

```mermaid
flowchart LR
    A["Push"] --> B["Lint and types"]
    B --> C["Unit and integration tests"]
    C --> D["Security scans"]
    D --> E["Build image"]
    E --> F["Deploy to staging"]
    F --> G["Smoke tests"]
    G --> H["Canary in production"]
    H --> I["Full rollout"]
```

```yaml
# .github/workflows/ci.yml
name: ci
on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 5s --health-retries 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/postgres
      - run: npm audit --omit=dev --audit-level=high

  image:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t registry.example.com/orders-api:${{ github.sha }} .
      - run: docker push registry.example.com/orders-api:${{ github.sha }}
```

Key rules:
- **Build once, deploy many:** the same image moves through environments.
- Tag images with the commit SHA, not `latest`.
- Fail fast: put the quick checks first.
- Prefer short-lived cloud credentials through OIDC to stored keys.

#### Blue-green deployments
Two identical environments. Deploy to the idle one, test it, then switch traffic all at once.

```yaml
# Kubernetes: switch traffic by changing the Service selector
apiVersion: v1
kind: Service
metadata:
  name: orders-api
spec:
  selector:
    app: orders-api
    version: green       # was blue. Changing it switches traffic instantly.
  ports:
    - port: 80
      targetPort: 3000
```

Rollback is a quick switch back. The cost is twice the capacity during the release, and both versions must work with the same database.

#### Canary deployments
Send a small share of traffic to the new version, watch metrics, then increase it gradually.

```yaml
# Argo Rollouts
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: orders-api
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 10m }
        - analysis:
            templates:
              - templateName: error-rate
        - setWeight: 25
        - pause: { duration: 10m }
        - setWeight: 100
```

| Strategy | Rollback speed | Extra capacity | Risk | Complexity |
|---|---|---|---|---|
| Rolling | Medium | Small | Medium | Low |
| Blue-green | Instant | 2× | Medium | Medium |
| Canary | Fast | Small | Lowest | High |

**Feature flags** separate deploy from release: ship code turned off, then enable it for 1% of users.

```javascript
if (await flags.isEnabled('new-checkout', { userId: req.user.id })) {
  return newCheckout(req, res);
}
return oldCheckout(req, res);
```

#### Rollbacks
Plan the way back before you go forward.

```bash
# Kubernetes
kubectl rollout history deployment/orders-api
kubectl rollout undo deployment/orders-api
kubectl rollout status deployment/orders-api
```

Code rolls back easily, but **data doesn't**. So database changes must stay backward compatible with the previous version, using expand and contract (Topic 37). If a change wrote bad data, you fix forward.

### Real-time scenario
**Problem:** A release raises checkout errors to 8%, and the rollback fails.
**Diagnosis:** The migration dropped a column that the old version still read, so the old code crashed.
**Fix:** Restore the column, then adopt expand and contract, and a canary with automatic error-rate rollback.

### Common mistakes and trade-offs
- Deploying `latest`, so you can't tell what's running.
- Rebuilding per environment, so staging isn't what reaches production.
- No automatic check during rollouts.
- **Trade-off:** canaries give the safest releases, but need good metrics and traffic routing.

### Top interview questions
**Q1. Describe a good Node CI/CD pipeline.**
Lint, types, tests, security scans, immutable image, staging, smoke tests, then a gradual production rollout with automatic rollback.

**Q2. What is the difference between blue-green and canary?**
Blue-green switches all traffic at once. Canary moves a small share gradually.

**Q3. How do you roll back when a schema changed?**
Keep migrations backward compatible, so old code still runs. Roll forward if data has changed.

**Q4. What are feature flags for?**
Releasing independently of deploying, with gradual rollout and an instant kill switch.

**Q5. What does "build once, deploy many" mean?**
One tested artifact goes through all environments. Only config changes.

**Q6. How do you speed up pipelines?**
Cache dependencies, run jobs in parallel, test only affected packages, and order checks fastest-first.

### Quick revision
Lint → test → scan → build once → staging → canary → full rollout. Tag by SHA. Backward-compatible migrations. Feature flags split deploy from release.

---

## 65. ⭐ Observability

### Why it matters
You can't fix what you can't see. Senior engineers design services that explain themselves during an incident, using logs, metrics and traces.

### Explanation

| Signal | Answers | Example |
|---|---|---|
| Logs | What happened in this event? | Payment failed for order 42 |
| Metrics | How much, how often, how slow? | p99 latency = 400 ms |
| Traces | Where did the time go across services? | 300 ms in inventory call |

#### Structured logging with Pino
Log JSON, not sentences, so that tools can search it. Pino is one of the fastest Node loggers, and it uses very little main-thread time.

```javascript
const pino = require('pino');

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: ['req.headers.authorization', 'req.headers.cookie', '*.password'],
  base: { service: 'orders-api', version: process.env.APP_VERSION },
});

// Bad: hard to search
console.log(`Order ${order.id} failed for ${user.id}`);

// Good: searchable fields
logger.error({ orderId: order.id, userId: user.id, err }, 'order failed');

// Attach request context automatically
const pinoHttp = require('pino-http');
app.use(pinoHttp({
  logger,
  genReqId: (req) => req.headers['x-request-id'] ?? crypto.randomUUID(),
}));

app.get('/orders/:id', (req, res) => {
  req.log.info({ orderId: req.params.id }, 'fetching order');
  res.json({});
});
```

```json
{"level":30,"time":1760000000000,"service":"orders-api","reqId":"a1b2","orderId":"42","msg":"fetching order"}
```

| Level | Use for |
|---|---|
| `fatal` | Process is about to exit |
| `error` | A failed operation that needs attention |
| `warn` | Something unusual, but handled |
| `info` | Key business events |
| `debug` | Development detail, off in production |

Write logs to stdout in containers, and let the platform ship them.

#### Metrics
Numbers over time, used for dashboards and alerts. Prometheus with `prom-client` is the common choice.

```javascript
const client = require('prom-client');

client.collectDefaultMetrics();   // includes heap, GC and event loop lag

const httpDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.3, 0.5, 1, 2, 5],
});

app.use((req, res, next) => {
  const end = httpDuration.startTimer();
  res.on('finish', () => {
    end({
      method: req.method,
      route: req.route?.path ?? 'unmatched',   // template, not the actual URL
      status: res.statusCode,
    });
  });
  next();
});

app.get('/metrics', async (req, res) => {
  res.type(client.register.contentType).send(await client.register.metrics());
});
```

```text
# Example PromQL
# p99 latency by route
histogram_quantile(0.99, sum by (le, route) (rate(http_request_duration_seconds_bucket[5m])))

# Error rate
sum(rate(http_request_duration_seconds_count{status=~"5.."}[5m]))
  / sum(rate(http_request_duration_seconds_count[5m]))
```

Frameworks for what to measure:
- **RED** for services: Rate, Errors, Duration.
- **USE** for resources: Utilization, Saturation, Errors.
- **Node extras:** event loop delay, heap, GC pauses, pool wait.

Keep label **cardinality** low: no user IDs or raw URLs. Each unique label set creates a new time series.

Alert on what users feel (SLOs), not on every internal metric.

#### OpenTelemetry tracing
A **trace** follows one request across services. Each step is a **span**. OpenTelemetry is the vendor-neutral standard, and it auto-instruments many libraries.

```javascript
// tracing.js: must load before any other module
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

const sdk = new NodeSDK({
  serviceName: 'orders-api',
  traceExporter: new OTLPTraceExporter({ url: 'http://otel-collector:4318/v1/traces' }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
process.on('SIGTERM', () => sdk.shutdown());
```

```bash
node --require ./tracing.js dist/server.js
```

```javascript
// Custom span for business logic
const { trace, SpanStatusCode } = require('@opentelemetry/api');
const tracer = trace.getTracer('orders');

async function priceOrder(order) {
  return tracer.startActiveSpan('price-order', async (span) => {
    span.setAttribute('order.items', order.items.length);
    try {
      return await calculate(order);
    } catch (err) {
      span.recordException(err);
      span.setStatus({ code: SpanStatusCode.ERROR });
      throw err;
    } finally {
      span.end();
    }
  });
}
```

```text
Trace 7f3a  total 480 ms
├─ orders-api POST /orders         480 ms
│  ├─ pg SELECT users               12 ms
│  ├─ inventory GET /stock         310 ms  ← the slow part
│  │  └─ pg SELECT stock           295 ms
│  └─ pg INSERT orders              18 ms
```

The `traceparent` header carries context between services, automatically. Add the trace ID to logs, so you can jump from a log line to its trace. **Sampling** controls cost: keep all errors and a percentage of the rest.

### Real-time scenario
**Problem:** Users report slow checkout, but every team's dashboard looks green.
**Diagnosis:** Averages looked fine. Traces showed a p99 delay in a tax service called only for certain regions.
**Fix:** Add per-route p99 alerts, sample errors and slow traces fully, and link logs with trace IDs.

### Common mistakes and trade-offs
- High-cardinality labels, which can overload Prometheus.
- Logging secrets or personal data.
- Too many alerts, which teach people to ignore them.
- **Trade-off:** more telemetry gives better insight, at real storage and processing cost. Sample, and keep what you use.

### Top interview questions
**Q1. What are the three pillars of observability?**
Logs, metrics and traces.

**Q2. Why use structured logging?**
JSON fields can be queried, aggregated and correlated, which free-form text can't do well.

**Q3. Why is Pino faster than many loggers?**
It does minimal work on the main thread, writes newline JSON, and moves formatting out of process.

**Q4. Which metrics matter most for a Node service?**
Request rate, errors, p95/p99 latency, event loop delay, heap, GC pauses, and pool waiting.

**Q5. How does distributed tracing work?**
Each request has a trace ID passed in headers. Spans record timing and are assembled by the backend.

**Q6. What is cardinality?**
The number of unique label combinations. High values cause storage and cost problems.

**Q7. How do you correlate logs with traces?**
Include the trace ID in every log line, and use the same ID in all services.

### Quick revision
Logs = events. Metrics = trends. Traces = path. Pino to stdout, Prometheus with low cardinality, OpenTelemetry for spans. Alert on SLOs.

---

# Part 13: Modern Node.js

## 66. Recent Built-In Features

### Why it matters
Interviewers use this topic to see whether your Node knowledge is current. Many packages you once installed are now built in. Exact version numbers change, so check the docs for your target version.

### Explanation

#### `node:test`
A built-in test runner with mocks, watch mode and coverage. Topic 51 has more detail.

```javascript
const { test } = require('node:test');
const assert = require('node:assert/strict');

test('adds numbers', () => {
  assert.equal(1 + 2, 3);
});

test('async work', async () => {
  const value = await Promise.resolve('ok');
  assert.equal(value, 'ok');
});

// Replaces Jest or Mocha for many projects:
// node --test
```

#### `--watch`
Restarts the process when files change. It replaces `nodemon` for most development.

```bash
node --watch src/server.js
node --watch-path=./src --watch-path=./config src/server.js
```

```json
{ "scripts": { "dev": "node --watch --env-file=.env src/server.js" } }
```

#### `--env-file`
Loads a `.env` file with no `dotenv` package.

```bash
# .env
PORT=3000
DATABASE_URL=postgres://localhost/app

# Load it
node --env-file=.env app.js

# Later files override earlier ones. Missing optional files are allowed with:
node --env-file=.env --env-file-if-exists=.env.local app.js
```

```javascript
// Or load it from code
process.loadEnvFile('.env');
console.log(process.env.PORT);

// Parse without touching process.env
const { parseEnv } = require('node:util');
console.log(parseEnv('PORT=3000'));   // { PORT: '3000' }
```

Real environment variables win over the file's values, so production settings can't be overridden by a stray `.env`.

#### Permission model
Restricts what a process can access, which limits the damage from a compromised dependency. Access is denied unless you grant it.

```bash
# Blocks file system, child processes and workers unless allowed
node --permission --allow-fs-read=/app --allow-fs-write=/tmp app.js

# Grant more, only if needed
node --permission --allow-fs-read=* --allow-child-process --allow-worker app.js
```

```javascript
// Check at runtime
if (process.permission?.has('fs.write', '/tmp')) {
  console.log('can write to /tmp');
}

// Without permission, this throws ERR_ACCESS_DENIED
const fs = require('node:fs');
try {
  fs.readFileSync('/etc/passwd');
} catch (err) {
  console.log(err.code);
}
```

It's a seat belt for trusted code, not a sandbox for hostile code. Which resources it covers, including network access, depends on the Node version.

#### Native TypeScript support
Recent Node versions run `.ts` files by stripping types. See Topic 34.

```bash
node src/server.ts
```

```typescript
interface User { id: number; name: string }

const user: User = { id: 1, name: 'Asha' };
console.log(user.name);

// Stripped and run with no type checking. Enums need a build step or a flag.
```

**Other useful built-ins:**

```javascript
// Global fetch, WebSocket client and Web Streams
const res = await fetch('https://example.com');

// Argument parsing, without yargs
const { parseArgs } = require('node:util');

// Terminal colours, without chalk
const { styleText } = require('node:util');
console.log(styleText('green', 'passed'));

// Built-in SQLite for tools, tests and small apps
const { DatabaseSync } = require('node:sqlite');
const db = new DatabaseSync(':memory:');
db.exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)');
db.prepare('INSERT INTO users (name) VALUES (?)').run('Asha');
console.log(db.prepare('SELECT * FROM users').all());

// Modern ES features
const grouped = Object.groupBy([1, 2, 3], (n) => (n % 2 ? 'odd' : 'even'));
const { promise, resolve } = Promise.withResolvers();
```

| Old package | Built-in replacement |
|---|---|
| `nodemon` | `--watch` |
| `dotenv` | `--env-file` |
| `axios`, `node-fetch` | `fetch` |
| `mocha`, simple Jest use | `node:test` |
| `yargs` (simple use) | `util.parseArgs` |
| `chalk` | `util.styleText` |
| `ws` (client only) | Global `WebSocket` |
| `uuid` | `crypto.randomUUID` |

**Release lines:** even-numbered versions become **LTS**, with long support. Use active or maintenance LTS in production. Odd-numbered versions are short-lived.

### Real-time scenario
**Problem:** A small internal service has 18 dev dependencies, and security alerts keep arriving for tools it barely uses.
**Diagnosis:** `nodemon`, `dotenv`, `node-fetch` and `mocha` were all replaceable.
**Fix:** Move to `--watch`, `--env-file`, `fetch` and `node:test`. Audit noise and install time both fall.

### Common mistakes and trade-offs
- Treating the permission model as a full sandbox.
- Assuming native TS type-checks.
- Using experimental features in production without checking stability.
- **Trade-off:** built-ins mean fewer dependencies and less risk, but mature packages may still offer more features.

### Top interview questions
**Q1. What are some recent Node features you've used?**
Native `fetch`, `node:test`, `--watch`, `--env-file`, type stripping, `require()` of ESM and the permission model.

**Q2. What does the permission model do?**
It denies file, process and worker access unless flags allow it, reducing the impact of compromised code.

**Q3. Do you still need `dotenv`?**
Often not. `--env-file` or `process.loadEnvFile` cover simple cases. Production usually injects env vars directly.

**Q4. Which Node version should you run in production?**
An active or maintenance LTS release. Plan upgrades before end of life.

**Q5. Can you replace `nodemon`?**
Yes, in most cases, with `node --watch`.

### Quick revision
`node:test`, `--watch`, `--env-file`, `--permission`, type stripping, global `fetch`. Use LTS. Fewer packages, smaller attack surface.

---

## 67. Runtime Comparison

### Why it matters
"Why not Bun or Deno?" is a common question. A good answer compares maturity, compatibility and team risk, instead of quoting benchmarks.

### Explanation

| | Node.js | Bun | Deno |
|---|---|---|---|
| Engine | V8 | JavaScriptCore | V8 |
| Written in | C++ | Zig | Rust |
| TypeScript | Type stripping | Built in | Built in |
| Package manager | npm, pnpm, yarn | `bun install`, very fast | `deno add`, JSR and npm |
| Test runner | `node:test` | `bun test` | `deno test` |
| Security default | Open, opt-in permissions | Open | Locked, opt-in permissions |
| Ecosystem | Largest | Strong Node compatibility | Good npm compatibility |
| Maturity | Longest track record | Younger | Younger |

The same server in each runtime:

```javascript
// Node.js
const http = require('node:http');
http.createServer((req, res) => res.end('hello')).listen(3000);
```

```typescript
// Bun
Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response('hello');
  },
});
```

```typescript
// Deno: run with `deno run --allow-net server.ts`
Deno.serve({ port: 3000 }, (req) => new Response('hello'));
```

Deno's secure default in practice:

```bash
deno run app.ts                                   # no file or network access
deno run --allow-read=./data --allow-net=api.example.com app.ts
```

How to decide:

```text
Stay on Node when:   production systems, big dependency trees, enterprise support
Consider Bun when:   fast tooling and startup matter, and you test compatibility
Consider Deno when:  you want secure defaults and web-standard APIs
```

A low-risk way to adopt: use Bun as a package manager or test runner while production stays on Node. Writing to **Web Standard APIs** like `fetch`, `Request` and `Response` keeps code portable. Frameworks like **Hono** run on all three.

```typescript
import { Hono } from 'hono';

const app = new Hono();
app.get('/health', (c) => c.json({ ok: true }));

export default app;   // runs on Bun, Deno, Cloudflare Workers and Node (with an adapter)
```

### Real-time scenario
**Problem:** A team wants to rewrite on Bun after seeing a benchmark with 3× the throughput.
**Diagnosis:** Their service spends 85% of its time waiting on PostgreSQL, so the runtime hardly matters.
**Fix:** Fix the queries first. Try Bun for CI installs, where it saves minutes. Test any production switch separately.

### Common mistakes and trade-offs
- Trusting "hello world" benchmarks.
- Assuming 100% Node compatibility, especially for native addons and APM agents.
- **Trade-off:** newer runtimes give speed and better defaults, at the cost of a shorter production record.

### Top interview questions
**Q1. What are the main differences between Node, Bun and Deno?**
Node has V8 and the largest ecosystem. Bun uses JavaScriptCore and bundles fast tooling. Deno focuses on security defaults and web standards.

**Q2. Would you use Bun in production?**
Only after testing dependencies, observability and failure behaviour. Tooling use is a safer first step.

**Q3. Why is Deno considered more secure by default?**
Programs can't access files, network or env without explicit permission.

**Q4. Why does Node remain the default for many companies?**
Maturity, LTS policy, huge ecosystem, hiring and support from cloud and APM vendors.

**Q5. How do you write portable JavaScript across runtimes?**
Use Web Standard APIs, and keep runtime-specific code behind adapters.

### Quick revision
Node = maturity. Bun = speed and all-in-one tooling. Deno = secure defaults. Choose by real bottlenecks, not by benchmarks.

---

## 68. Native Extensions

### Why it matters
Sometimes JavaScript isn't enough, for speed or to reuse native libraries. You should know the options and their costs, even if you rarely write them.

### Explanation

#### N-API
Node-API is a stable C interface for addons. Addons built on it keep working across Node versions without recompiling. Libraries such as `sharp` and `bcrypt` use native code.

```cpp
// addon.cc, using the node-addon-api C++ wrapper
#include <napi.h>

Napi::Number Add(const Napi::CallbackInfo& info) {
  Napi::Env env = info.Env();
  double a = info[0].As<Napi::Number>().DoubleValue();
  double b = info[1].As<Napi::Number>().DoubleValue();
  return Napi::Number::New(env, a + b);
}

Napi::Object Init(Napi::Env env, Napi::Object exports) {
  exports.Set("add", Napi::Function::New(env, Add));
  return exports;
}

NODE_API_MODULE(addon, Init)
```

```json
{
  "targets": [{
    "target_name": "addon",
    "sources": ["addon.cc"],
    "include_dirs": ["<!@(node -p \"require('node-addon-api').include\")"],
    "defines": ["NAPI_DISABLE_CPP_EXCEPTIONS"]
  }]
}
```

```bash
npm install node-addon-api
npx node-gyp configure build
```

```javascript
const addon = require('./build/Release/addon.node');
console.log(addon.add(2, 3));   // 5
```

A native call still runs on the main thread. Long work must use `Napi::AsyncWorker`, which moves it to the libuv pool. Native code can also crash the whole process with a segmentation fault.

Other ways in: **napi-rs** for Rust, and prebuilt binaries so users don't need a compiler.

#### WebAssembly
Wasm runs compiled code from Rust, C or Go in a sandbox, at near-native speed. It's portable, so there's no per-platform build, and a bug can't corrupt process memory outside its sandbox.

```javascript
// A tiny module exporting add(a, b), as raw bytes
const bytes = new Uint8Array([
  0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00,
  0x01, 0x07, 0x01, 0x60, 0x02, 0x7f, 0x7f, 0x01, 0x7f,
  0x03, 0x02, 0x01, 0x00,
  0x07, 0x07, 0x01, 0x03, 0x61, 0x64, 0x64, 0x00, 0x00,
  0x0a, 0x09, 0x01, 0x07, 0x00, 0x20, 0x00, 0x20, 0x01, 0x6a, 0x0b,
]);

WebAssembly.instantiate(bytes).then(({ instance }) => {
  console.log(instance.exports.add(2, 3));   // 5
});
```

```rust
// Rust compiled with wasm-pack
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u32 {
    if n < 2 { n } else { fibonacci(n - 1) + fibonacci(n - 2) }
}
```

```javascript
const { fibonacci } = require('./pkg/fib.js');
console.log(fibonacci(30));
```

| | Pure JS | Worker thread | Wasm | N-API |
|---|---|---|---|---|
| Speed | Good after JIT | Parallel JS | Near native | Native |
| Portable | Yes | Yes | Yes | Per platform, or prebuilds |
| Crash risk | Low | Low | Low | Can kill the process |
| OS and library access | Through Node | Through Node | Limited | Full |
| Build complexity | None | None | Medium | High |

Calling across the JS boundary has a cost. Pass large batches, not millions of tiny calls.

### Real-time scenario
**Problem:** Docker builds fail on ARM with `node-gyp` errors, after switching to Graviton instances.
**Diagnosis:** A native dependency had no ARM prebuilt binary, and the image had no compiler.
**Fix:** Upgrade to a version with ARM prebuilds. Where one isn't available, use a Wasm or pure JS alternative, or compile in a build stage.

### Common mistakes and trade-offs
- Reaching for native code before profiling.
- Blocking the main thread inside an addon.
- Forgetting platform differences such as Alpine's musl and ARM.
- **Trade-off:** native code offers top speed and library access, in exchange for harder builds and memory-safety risks.

### Top interview questions
**Q1. What is N-API?**
A stable ABI for native addons, so that they work across Node versions without rebuilding.

**Q2. When would you write a native addon?**
To use a C or C++ library, or for proven CPU hot spots that JS and Wasm can't handle.

**Q3. N-API or WebAssembly?**
Wasm for portable, sandboxed computation. N-API for OS access or existing native libraries.

**Q4. Do native addons run on another thread?**
Not by default. Use async workers, or they'll block the event loop.

**Q5. What are the risks of native addons?**
Crashes, memory bugs, compile failures and per-platform builds.

### Quick revision
N-API = stable native addons. Wasm = portable and sandboxed. Profile first. Long native work needs async workers. Batch calls across the boundary.

---

## 69. AI in Node.js Backends

### Why it matters
Node is a natural fit for AI features, because LLM calls are slow, I/O-bound and streamed. Teams now ask about streaming, tool calling, limits and cost. The examples use Anthropic's TypeScript SDK, and the same ideas apply to other providers. Model names and SDK details change quickly, so check the current docs.

### Explanation

#### Streaming LLM responses
A full answer can take many seconds. Streaming shows tokens as they're generated, so users see progress immediately. SSE fits well (Topic 31).

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();   // reads ANTHROPIC_API_KEY

app.post('/chat', async (req, res) => {
  res.set({ 'content-type': 'text/event-stream', 'cache-control': 'no-cache' });
  res.flushHeaders();

  const stream = client.messages.stream({
    model: 'claude-opus-5-5',
    max_tokens: 4000,
    messages: [{ role: 'user', content: req.body.prompt }],
  });

  // Stop generating, and paying, when the user leaves
  res.on('close', () => {
    if (!res.writableFinished) stream.abort();
  });

  try {
    for await (const event of stream) {
      if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
        res.write(`data: ${JSON.stringify({ text: event.delta.text })}\n\n`);
      }
    }

    const message = await stream.finalMessage();
    res.write(`data: ${JSON.stringify({ done: true, stop: message.stop_reason })}\n\n`);
  } catch (err) {
    req.log.error({ err }, 'stream failed');
    res.write(`data: ${JSON.stringify({ error: 'generation failed' })}\n\n`);
  } finally {
    res.end();
  }
});
```

Check for proxies that buffer output: with NGINX, send `X-Accel-Buffering: no`, and disable compression on this route. Check `stop_reason`: `max_tokens` means the answer was cut off, and `refusal` means the model declined. On the newest models, Anthropic's API also offers a server-side fallback for refusals.

#### Tool calling
The model can't run code itself. It asks for a tool call, your server runs it, and you send back the result. This repeats until the model finishes.

```mermaid
sequenceDiagram
    participant U as User
    participant S as Node server
    participant M as Model
    U->>S: Where is order 42?
    S->>M: Prompt and tool definitions
    M->>S: Call get_order(id 42)
    S->>S: Validate, authorize, run
    S->>M: Tool result
    M->>S: Final answer
    S->>U: Your order has shipped
```

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { betaZodTool } from '@anthropic-ai/sdk/helpers/beta/zod';
import { z } from 'zod';

const client = new Anthropic();

async function askSupport(userId: string, question: string) {
  const getOrder = betaZodTool({
    name: 'get_order',
    description: 'Look up the status of one of the current user\'s orders',
    inputSchema: z.object({ orderId: z.string() }),
    run: async ({ orderId }) => {
      // userId comes from your session, never from the model
      const order = await orders.findOne({ id: orderId, userId });
      return order ? JSON.stringify({ status: order.status }) : 'Order not found';
    },
  });

  // The SDK's tool runner handles the request, run, send-result loop
  return client.beta.messages.toolRunner({
    model: 'claude-opus-5-5',
    max_tokens: 4000,
    tools: [getOrder],
    messages: [{ role: 'user', content: question }],
  });
}
```

Treat model output as **untrusted input**, because text in a document or web page can try to steer the model. This is **prompt injection**. Protect yourself:

- Authorize inside each tool, using your own session data.
- Validate tool arguments with a schema.
- Give the fewest tools and permissions possible.
- Require human approval for risky actions such as refunds.
- Limit loop iterations, to stop runaway cost.

#### Rate limits
Providers limit requests and tokens per minute. The SDK already retries 429s and 5xx errors twice by default. Handle what remains, and queue background work.

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  maxRetries: 3,
  timeout: 60_000,         // milliseconds
});

async function summarize(text: string) {
  try {
    return await client.messages.create({
      model: 'claude-opus-5-5',
      max_tokens: 1000,
      messages: [{ role: 'user', content: `Summarize:\n\n${text}` }],
    });
  } catch (err) {
    if (err instanceof Anthropic.RateLimitError) {
      throw new AppError('AI is busy, try again soon', { status: 429, code: 'AI_BUSY' });
    }
    if (err instanceof Anthropic.APIConnectionError) {
      throw new AppError('AI service unreachable', { status: 503, code: 'AI_DOWN' });
    }
    if (err instanceof Anthropic.APIError) {
      throw new AppError('AI request failed', { status: 502, code: 'AI_ERROR', cause: err });
    }
    throw err;
  }
}

// Protect the shared quota: a bounded queue for background work,
// and a per-user limit so one user can't take everything
const aiQueue = new Bulkhead(20, 200);
app.post('/summarize', rateLimit({ limit: 10, windowMs: 60_000, keyFn: (r) => r.user.id }),
  async (req, res) => {
    res.json(await aiQueue.run(() => summarize(req.body.text)));
  });
```

#### Cost control
Cost follows tokens, and output tokens cost more than input tokens.

```typescript
// 1. Track usage per user and feature
const message = await client.messages.create({
  model: 'claude-opus-5-5',
  max_tokens: 1000,
  messages,
});

metrics.increment('ai.tokens.input', message.usage.input_tokens, { feature: 'support' });
metrics.increment('ai.tokens.output', message.usage.output_tokens, { feature: 'support' });

// 2. Enforce a budget before calling
async function checkBudget(userId: string) {
  const used = Number(await redis.get(`ai:tokens:${userId}:${today()}`) ?? 0);
  if (used > DAILY_TOKEN_LIMIT) {
    throw new AppError('Daily AI limit reached', { status: 429, code: 'AI_BUDGET' });
  }
}

// 3. Prompt caching: keep the large stable prefix first, and mark it as cacheable
const cached = await client.messages.create({
  model: 'claude-opus-5-5',
  max_tokens: 1000,
  system: [{
    type: 'text',
    text: LONG_PRODUCT_DOCS,
    cache_control: { type: 'ephemeral' },
  }],
  messages: [{ role: 'user', content: question }],
});
console.log(cached.usage.cache_read_input_tokens);

// 4. Cache whole answers for repeated questions
const key = `ai:${sha256(normalize(question))}`;
const hit = await redis.get(key);
if (hit) return hit;
```

Other levers:

| Lever | Saves |
|---|---|
| Limit `max_tokens` | Output cost, runaway answers |
| Lower reasoning effort for simple tasks | Output tokens and latency |
| A smaller model for simple tasks | Cost per token |
| Trim history, retrieve only relevant documents | Input tokens |
| Batch API for offline work | Cheaper non-urgent jobs |
| Cancel when the client disconnects | Wasted generation |

Measure **cost per completed task**, since a cheap answer that needs retries isn't cheap.

### Real-time scenario
**Problem:** An AI support feature's bill jumps 6× in one week, and the team doesn't know why.
**Diagnosis:** Usage tracking by feature shows the chat sends the full 200-message history and a large document on every turn. Abandoned sessions also kept generating.
**Fix:** Trim the history, retrieve only relevant sections, use prompt caching for stable text, abort on disconnect, and add per-user budgets.

### Common mistakes and trade-offs
- Running tools with IDs the model chose, and no authorization check.
- Waiting for the full response when streaming is possible.
- Logging prompts that contain personal data.
- Having no timeouts or token limits.
- **Trade-off:** larger models and more context improve quality but raise cost and latency. Use evaluations to find the cheapest setup that meets your quality bar.

### Top interview questions
**Q1. Why is Node a good fit for LLM features?**
LLM calls are long I/O waits with streamed output, and the event loop handles many at once cheaply.

**Q2. How do you stream an LLM response to a browser?**
Use the SDK's stream on the server and forward text deltas with SSE. Abort when the client disconnects.

**Q3. How does tool calling work?**
The model requests a tool with arguments. Your code validates, runs it and returns the result, in a loop until the model is done.

**Q4. What is prompt injection, and how do you limit it?**
Untrusted text tries to override your instructions. Use least privilege, validate tool calls, authorize server-side, and approve risky actions.

**Q5. How do you handle provider rate limits?**
SDK retries with backoff, per-user limits, a bounded queue for background work, and clear 429 responses.

**Q6. How do you control LLM costs?**
Track tokens by feature, set budgets and `max_tokens`, use caching, trim context and pick the right model.

**Q7. How do you test AI features?**
Mock the provider in unit tests, and run evaluations with real prompts and scoring. Output varies, so don't rely on exact string matches.

### Quick revision
Stream with SSE, abort on disconnect. Tools: validate and authorize, model output is untrusted. Handle 429s. Track tokens, cache prefixes, set budgets.

---

# Part 14: Interview Revision Kit

## 70. Must-Know Top 30 Questions

Ranked by how often they tend to appear and how much they reveal. Practise saying each answer aloud in under a minute.

**1. Explain the event loop.** *(Topic 6)*
Node runs JS on one thread. libuv cycles through timers, pending, poll, check and close phases. After each callback, it drains `nextTick` and Promise microtasks. Blocking the thread blocks everything.

**2. How does Node handle concurrency with a single thread?** *(Topics 5, 6)*
It delegates I/O to the OS and libuv's thread pool. The main thread keeps serving requests, and runs callbacks when work completes.

**3. `process.nextTick` vs `setImmediate` vs `setTimeout(0)`?** *(Topic 6)*
`nextTick` runs right after the current operation, `setImmediate` in the check phase, and `setTimeout` in the timers phase. Inside an I/O callback, immediate beats timeout.

**4. How do you handle CPU-heavy work?** *(Topics 22, 25)*
Use a worker thread pool, or a background queue if the user doesn't need the result now. Keep the main thread for I/O.

**5. What are streams, and why use them?** *(Topic 16)*
Chunked data handling with constant memory. Use `pipeline` to get error handling and cleanup.

**6. What is backpressure?** *(Topic 17)*
A slow consumer signalling a fast producer to pause. `write()` returns `false`, and `drain` means continue.

**7. Cluster vs worker threads vs child process?** *(Topics 22–25)*
Cluster scales request handling across processes. Workers run CPU tasks in one process. Child processes run other programs.

**8. How do you find a memory leak?** *(Topic 53)*
Confirm a rising floor in metrics, compare heap snapshots, follow retainers, fix and verify under load.

**9. How does garbage collection work?** *(Topic 8)*
Generational: fast scavenges in new space and mark-sweep-compact in old space, based on reachability.

**10. CommonJS vs ES Modules?** *(Topic 3)*
CJS is sync and dynamic, with copied values. ESM is static, with live bindings and top-level `await`.

**11. `Promise.all` vs `allSettled` vs `race` vs `any`?** *(Topic 10)*
All or fail fast, every outcome, first settled, first success.

**12. How do you handle errors in Node?** *(Topic 47)*
Handle operational errors, crash on bugs, centralize responses, and exit on `uncaughtException`.

**13. How do you do graceful shutdown?** *(Topic 48)*
On `SIGTERM`, fail readiness, stop accepting, finish in-flight work, close dependencies and exit, with a forced timeout.

**14. How do you scale a Node app?** *(Topic 56)*
Find the bottleneck, go stateless, run multiple instances, cache, use replicas and queue slow work.

**15. How does Express middleware work?** *(Topic 26)*
An ordered stack of functions. `next()` moves on, and `next(err)` jumps to four-argument error handlers.

**16. JWT vs sessions? How do you revoke a JWT?** *(Topic 41)*
Sessions are server-side and easy to revoke. JWTs are stateless. Use short expiry, refresh rotation and a denylist.

**17. How do you secure a Node API?** *(Topics 43, 44)*
Validate input, parameterize queries, enforce auth on every object, set headers, limit rates, and audit dependencies.

**18. How do you implement rate limiting across servers?** *(Topic 46)*
Token bucket or sliding window in Redis with atomic Lua scripts. Return 429 with `Retry-After`.

**19. How do you design caching? What's a cache stampede?** *(Topic 38)*
Cache-aside with TTLs and delete on write. A stampede is many misses at once. Fix it with locks, coalescing and jitter.

**20. How does connection pooling work?** *(Topic 36)*
Reuses limited connections. Total equals pods × pool size. Release in `finally`, and watch waiting clients.

**21. Can Node have race conditions?** *(Topic 25)*
Yes. Every `await` lets other work run. Use atomic updates, locks or version columns.

**22. Kafka vs RabbitMQ? What about exactly-once?** *(Topic 40)*
Kafka is a replayable log, and RabbitMQ is routing. In practice, use at-least-once with idempotent consumers.

**23. REST vs GraphQL vs gRPC?** *(Topics 28–30)*
REST for public APIs, GraphQL for flexible client data, gRPC for fast typed internal calls.

**24. What is idempotency, and how do you implement it?** *(Topic 28)*
Repeats have the same effect as one call. Use idempotency keys and unique constraints.

**25. Monolith or microservices?** *(Topic 57)*
Start with a modular monolith. Split for team independence or very different scaling needs.

**26. How do you keep data consistent across services?** *(Topic 58)*
Sagas with compensations, the outbox pattern and idempotent consumers.

**27. A Node API is slow. How do you debug it?** *(Topic 54)*
Check CPU and event loop delay, then traces. Profile if CPU-bound. Otherwise, find the wait.

**28. How do you monitor a Node service?** *(Topic 65)*
Structured logs, RED metrics plus event loop delay and heap, and OpenTelemetry traces.

**29. How do you write a production Dockerfile?** *(Topic 62)*
Multi-stage, slim, non-root, `npm ci --omit=dev`, exec-form `CMD`, and signal handling.

**30. How do you test Node services?** *(Topic 50)*
Unit tests for logic, integration tests with real databases, contract tests between services, and a few E2E journeys.

---

## 71. Output-Based Questions

Predict the output before reading it. Examples run as CommonJS. For each, say **why** out loud.

### Puzzle 1: Basic ordering

```javascript
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('tick'));
console.log('end');
```

```text
start
end
tick
promise
timeout
```

**Why:** Sync code runs first, then `nextTick`, then Promises, then timers.

### Puzzle 2: async/await

```javascript
async function first() {
  console.log('first start');
  await second();
  console.log('first end');
}

async function second() {
  console.log('second');
}

console.log('script start');
first();
Promise.resolve().then(() => console.log('then'));
console.log('script end');
```

```text
script start
first start
second
script end
first end
then
```

**Why:** Async functions run synchronously up to the first `await`. The rest of `first` was queued before the `.then` callback.

### Puzzle 3: Inside an I/O callback

```javascript
const fs = require('node:fs');

fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
  process.nextTick(() => console.log('tick'));
});
```

```text
tick
immediate
timeout
```

**Why:** `nextTick` runs right after the callback. Then the loop moves from poll to check, so immediate runs before the next timers phase.

### Puzzle 4: `nextTick` inside a Promise

```javascript
Promise.resolve()
  .then(() => {
    console.log('p1');
    process.nextTick(() => console.log('tick'));
  })
  .then(() => console.log('p2'));
```

```text
p1
p2
tick
```

**Why:** Once the Promise queue is draining, it empties fully, including newly added microtasks, before the `nextTick` queue is checked again.

### Puzzle 5: Microtasks between timers

```javascript
setTimeout(() => {
  console.log('t1');
  Promise.resolve().then(() => console.log('p'));
}, 0);

setTimeout(() => console.log('t2'), 0);
```

```text
t1
p
t2
```

**Why:** Since Node 11, microtasks drain after each timer callback, not after the whole phase.

### Puzzle 6: `var` vs `let` in a loop

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log('var', i), 0);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log('let', j), 0);
}
```

```text
var 3
var 3
var 3
let 0
let 1
let 2
```

**Why:** `var` has one shared binding, which is 3 by the time timers run. `let` makes a new binding on each iteration.

### Puzzle 7: The Promise executor

```javascript
console.log(1);

const p = new Promise((resolve) => {
  console.log(2);
  resolve();
  console.log(3);
});

p.then(() => console.log(5));
console.log(4);
```

```text
1
2
3
4
5
```

**Why:** The executor is synchronous, and `resolve` doesn't stop it. `.then` always runs later.

### Puzzle 8: EventEmitter is synchronous

```javascript
const { EventEmitter } = require('node:events');
const bus = new EventEmitter();

bus.on('x', () => console.log('listener 1'));
bus.on('x', () => console.log('listener 2'));

console.log('before');
bus.emit('x');
console.log('after');
```

```text
before
listener 1
listener 2
after
```

**Why:** `emit` calls listeners synchronously, in order.

### Puzzle 9: `forEach` with async

```javascript
async function run() {
  [1, 2].forEach(async (n) => {
    await null;
    console.log('item', n);
  });
  console.log('done');
}

run();
```

```text
done
item 1
item 2
```

**Why:** `forEach` ignores returned Promises.

### Puzzle 10: try/catch and Promises

```javascript
process.on('unhandledRejection', (err) => console.log('unhandled:', err.message));

function run() {
  try {
    Promise.reject(new Error('boom'));
  } catch (err) {
    console.log('caught');
  }
  console.log('continued');
}

run();
```

```text
continued
unhandled: boom
```

**Why:** The rejection isn't a synchronous throw, and nothing awaited it, so `catch` never runs.

### Puzzle 11: `return` in `finally`

```javascript
function test() {
  try {
    return 'try';
  } finally {
    console.log('finally runs');
  }
}

console.log(test());
```

```text
finally runs
try
```

**Why:** `finally` runs before the function returns. A `return` in `finally` would override the value.

### Puzzle 12: `Promise.all` doesn't cancel

```javascript
const wait = (ms, label) => new Promise((resolve) =>
  setTimeout(() => {
    console.log('finished', label);
    resolve(label);
  }, ms));

Promise.all([
  wait(20, 'slow'),
  Promise.reject(new Error('fast failure')),
]).catch((err) => console.log(err.message));
```

```text
fast failure
finished slow
```

**Why:** `Promise.all` rejects early, but the other operation keeps running.

### Puzzle 13: Chained `.then` and `.catch`

```javascript
Promise.reject(new Error('x'))
  .then(() => console.log('then 1'))
  .catch((err) => {
    console.log('catch', err.message);
    return 'recovered';
  })
  .then((value) => console.log('then 2', value));
```

```text
catch x
then 2 recovered
```

**Why:** A rejection skips `.then` until a `.catch`. A `.catch` that returns recovers the chain.

### Puzzle 14: Module cache

```javascript
const fs = require('node:fs');
const os = require('node:os');
const path = require('node:path');

const file = path.join(fs.mkdtempSync(path.join(os.tmpdir(), 'm-')), 'config.js');
fs.writeFileSync(file, "console.log('loading'); module.exports = { n: 1 };");

const a = require(file);
a.n = 2;
const b = require(file);
console.log(b.n, a === b);
```

```text
loading
2 true
```

**Why:** The file runs once. Later `require` calls return the same cached object.

### Puzzle 15: `this` in callbacks

```javascript
const counter = {
  count: 1,
  regular() {
    setTimeout(function () {
      console.log('regular', this === counter);
    }, 0);
  },
  arrow() {
    setTimeout(() => {
      console.log('arrow', this === counter);
    }, 0);
  },
};

counter.regular();
counter.arrow();
```

```text
regular false
arrow true
```

**Why:** Arrow functions keep the surrounding `this`. In Node, a regular timer callback's `this` is the `Timeout` object.

---

## 72. Coding Tasks

These are the classic Node machine-coding tasks. For each one: state the requirements, write the simplest version, then discuss edge cases. All of them run as they are.

### Task 1: Rate limiter

**Requirements:** allow N requests per key, permit short bursts, don't leak memory, and tell callers when to retry.

**Approach:** a token bucket per key. Tokens refill lazily on each call, so no timer per key is needed. The clock is injected, which makes tests simple.

```javascript
class RateLimiter {
  constructor({ capacity, refillPerSecond, now = Date.now }) {
    this.capacity = capacity;
    this.refillPerSecond = refillPerSecond;
    this.now = now;
    this.buckets = new Map();
  }

  check(key, cost = 1) {
    const time = this.now();
    const bucket = this.buckets.get(key) ?? { tokens: this.capacity, updatedAt: time };

    const elapsedSeconds = (time - bucket.updatedAt) / 1000;
    bucket.tokens = Math.min(this.capacity, bucket.tokens + elapsedSeconds * this.refillPerSecond);
    bucket.updatedAt = time;
    this.buckets.set(key, bucket);

    if (bucket.tokens >= cost) {
      bucket.tokens -= cost;
      return { allowed: true, remaining: Math.floor(bucket.tokens), retryAfterMs: 0 };
    }

    const missing = cost - bucket.tokens;
    return {
      allowed: false,
      remaining: 0,
      retryAfterMs: Math.ceil((missing / this.refillPerSecond) * 1000),
    };
  }

  // Run on a timer: a full bucket holds no information, so it can go
  cleanup() {
    const time = this.now();
    for (const [key, bucket] of this.buckets) {
      const refilled = bucket.tokens + ((time - bucket.updatedAt) / 1000) * this.refillPerSecond;
      if (refilled >= this.capacity) this.buckets.delete(key);
    }
  }
}

// Express middleware
function rateLimit(limiter, keyFn = (req) => req.ip) {
  return (req, res, next) => {
    const result = limiter.check(keyFn(req));
    res.set('RateLimit-Remaining', String(result.remaining));

    if (!result.allowed) {
      res.set('Retry-After', String(Math.ceil(result.retryAfterMs / 1000)));
      return res.status(429).json({ error: 'too many requests' });
    }
    next();
  };
}

// Demo with a fake clock
let time = 0;
const limiter = new RateLimiter({ capacity: 2, refillPerSecond: 1, now: () => time });

console.log(limiter.check('a').allowed);   // true
console.log(limiter.check('a').allowed);   // true
console.log(limiter.check('a'));           // { allowed: false, remaining: 0, retryAfterMs: 1000 }
time = 1000;
console.log(limiter.check('a').allowed);   // true, one token has refilled
```

**Complexity:** O(1) per check, and memory is O(active keys).

**Follow-ups:**
- *Multiple servers?* Move state to Redis with a Lua script (Topic 46).
- *Sliding window instead?* Store timestamps in a sorted set.
- *Why lazy refill?* A timer per key doesn't scale. Elapsed time gives the same result.

### Task 2: Retry wrapper

**Requirements:** limited attempts, exponential backoff with jitter, retry only suitable errors, per-attempt timeout and cancellation.

```javascript
function sleep(ms, signal) {
  return new Promise((resolve, reject) => {
    if (signal?.aborted) return reject(signal.reason);

    const onAbort = () => {
      clearTimeout(timer);
      reject(signal.reason);
    };
    const timer = setTimeout(() => {
      signal?.removeEventListener('abort', onAbort);
      resolve();
    }, ms);

    signal?.addEventListener('abort', onAbort, { once: true });
  });
}

async function retry(fn, options = {}) {
  const {
    attempts = 3,
    baseMs = 100,
    maxMs = 5000,
    timeoutMs,
    shouldRetry = () => true,
    onRetry = () => {},
    signal,
  } = options;

  let lastError;

  for (let attempt = 1; attempt <= attempts; attempt++) {
    signal?.throwIfAborted();

    const signals = [signal, timeoutMs && AbortSignal.timeout(timeoutMs)].filter(Boolean);
    const attemptSignal = signals.length ? AbortSignal.any(signals) : undefined;

    try {
      return await fn({ attempt, signal: attemptSignal });
    } catch (err) {
      lastError = err;
      if (signal?.aborted || attempt === attempts || !shouldRetry(err)) break;

      const ceiling = Math.min(maxMs, baseMs * 2 ** (attempt - 1));
      const delay = Math.floor(Math.random() * ceiling);   // full jitter
      onRetry({ attempt, delay, err });
      await sleep(delay, signal);
    }
  }

  throw lastError;
}

// Demo: fails twice, then succeeds
let calls = 0;

retry(async ({ attempt }) => {
  calls++;
  if (attempt < 3) throw Object.assign(new Error('temporary'), { status: 503 });
  return 'ok';
}, {
  attempts: 5,
  baseMs: 10,
  shouldRetry: (err) => err.status >= 500,
}).then((result) => console.log(result, 'after', calls, 'calls'));   // ok after 3 calls
```

**Complexity:** at most `attempts` calls, and total delay is bounded by `maxMs × (attempts − 1)`.

**Follow-ups:**
- *What shouldn't be retried?* 4xx errors, and operations that aren't idempotent.
- *Why jitter?* It spreads retries out, so clients don't all return at the same moment.
- *What about `Retry-After`?* Respect it: use that delay in place of the calculated one.

### Task 3: Promise pool

**Requirements:** run async tasks with at most N at once, keep results in input order, and handle errors.

**Approach:** start N workers that pull from a shared index. There's no `await` between reading and incrementing the index, so the single thread makes it safe.

```javascript
async function promisePool(items, concurrency, worker, { stopOnError = true } = {}) {
  if (!Number.isInteger(concurrency) || concurrency < 1) {
    throw new RangeError('concurrency must be a positive integer');
  }

  const results = new Array(items.length);
  let nextIndex = 0;
  let failed = false;
  let firstError;

  async function run() {
    while (nextIndex < items.length && !failed) {
      const index = nextIndex++;

      try {
        const value = await worker(items[index], index);
        results[index] = stopOnError ? value : { status: 'fulfilled', value };
      } catch (reason) {
        if (stopOnError) {
          failed = true;
          firstError = reason;
        } else {
          results[index] = { status: 'rejected', reason };
        }
      }
    }
  }

  const workers = Array.from({ length: Math.min(concurrency, items.length) }, run);
  await Promise.all(workers);

  if (failed) throw firstError;
  return results;
}

// A limiter for tasks that arrive over time, like the p-limit package
function createLimiter(concurrency) {
  let active = 0;
  const queue = [];

  function next() {
    if (active >= concurrency || queue.length === 0) return;
    active++;
    const { fn, resolve, reject } = queue.shift();

    Promise.resolve()
      .then(fn)
      .then(resolve, reject)
      .finally(() => {
        active--;
        next();
      });
  }

  return (fn) => new Promise((resolve, reject) => {
    queue.push({ fn, resolve, reject });
    next();
  });
}

// Demo: proves only 2 run at once, and that order is kept
let running = 0;
let peak = 0;

const task = async (n) => {
  running++;
  peak = Math.max(peak, running);
  await new Promise((r) => setTimeout(r, 10 * (6 - n)));
  running--;
  return n * 2;
};

promisePool([1, 2, 3, 4, 5], 2, task).then((results) => {
  console.log(results);              // [ 2, 4, 6, 8, 10 ]
  console.log('peak:', peak);        // peak: 2
});

const limit = createLimiter(1);
limit(async () => 'a').then(console.log);   // a
```

**Complexity:** O(n) tasks, and O(n) memory for results.

**Follow-ups:**
- *Why not chunk into batches?* Each batch waits for its slowest task. A pool keeps every slot busy.
- *Millions of items?* Pull from an async iterator and write results as you go.
- *`queue.shift()` is O(n)?* True for large arrays. Use a head index or linked list.

### Task 4: LRU cache

**Requirements:** `get` and `put` in O(1), remove the least recently used item at capacity, and optional TTL.

**Approach 1, using `Map`:** a `Map` keeps insertion order, so delete and re-insert moves a key to the newest position.

```javascript
class LRUCache {
  constructor(capacity, { ttlMs, now = Date.now } = {}) {
    if (capacity < 1) throw new RangeError('capacity must be at least 1');
    this.capacity = capacity;
    this.ttlMs = ttlMs;
    this.now = now;
    this.map = new Map();
  }

  get(key) {
    const entry = this.map.get(key);
    if (!entry) return undefined;

    if (entry.expiresAt !== undefined && entry.expiresAt <= this.now()) {
      this.map.delete(key);
      return undefined;
    }

    this.map.delete(key);
    this.map.set(key, entry);     // move to newest
    return entry.value;
  }

  put(key, value) {
    this.map.delete(key);

    if (this.map.size >= this.capacity) {
      const oldest = this.map.keys().next().value;
      this.map.delete(oldest);
    }

    const expiresAt = this.ttlMs === undefined ? undefined : this.now() + this.ttlMs;
    this.map.set(key, { value, expiresAt });
  }

  get size() {
    return this.map.size;
  }
}

const cache = new LRUCache(2);
cache.put('a', 1);
cache.put('b', 2);
cache.get('a');                  // 'a' is now the most recent
cache.put('c', 3);               // removes 'b'
console.log(cache.get('b'));     // undefined
console.log(cache.get('a'));     // 1
console.log(cache.get('c'));     // 3

let clock = 0;
const expiring = new LRUCache(2, { ttlMs: 100, now: () => clock });
expiring.put('x', 1);
clock = 101;
console.log(expiring.get('x'));  // undefined
```

**Approach 2, doubly linked list + `Map`:** the answer to expect if an interviewer says not to rely on `Map` order.

```javascript
class Node {
  constructor(key, value) {
    this.key = key;
    this.value = value;
    this.prev = null;
    this.next = null;
  }
}

class LinkedLRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map();
    this.head = new Node();         // sentinels remove null checks
    this.tail = new Node();
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }

  unlink(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
  }

  addToFront(node) {
    node.next = this.head.next;
    node.prev = this.head;
    this.head.next.prev = node;
    this.head.next = node;
  }

  get(key) {
    const node = this.map.get(key);
    if (!node) return undefined;
    this.unlink(node);
    this.addToFront(node);
    return node.value;
  }

  put(key, value) {
    const existing = this.map.get(key);
    if (existing) {
      existing.value = value;
      this.unlink(existing);
      this.addToFront(existing);
      return;
    }

    if (this.map.size >= this.capacity) {
      const oldest = this.tail.prev;
      this.unlink(oldest);
      this.map.delete(oldest.key);
    }

    const node = new Node(key, value);
    this.map.set(key, node);
    this.addToFront(node);
  }
}

const linked = new LinkedLRUCache(2);
linked.put(1, 'one');
linked.put(2, 'two');
linked.get(1);
linked.put(3, 'three');
console.log(linked.get(2));      // undefined
console.log(linked.get(1));      // one
```

**Complexity:** O(1) for `get` and `put`, and O(capacity) memory.

**Follow-ups:**
- *How do you limit by bytes?* Track a size per entry, and evict until under budget.
- *What's the difference between LRU and LFU?* LRU evicts the oldest use, while LFU evicts the least frequent use.
- *Can several processes share it?* No. Use Redis for that.

### Task 5: Custom EventEmitter

**Requirements:** `on`, `off`, `once`, `emit`, synchronous calls in order, and special handling of `error`.

```javascript
class MyEmitter {
  constructor() {
    this.events = new Map();
  }

  on(event, listener) {
    if (typeof listener !== 'function') throw new TypeError('listener must be a function');
    if (!this.events.has(event)) this.events.set(event, []);
    this.events.get(event).push(listener);
    return this;                 // allows chaining
  }

  off(event, listener) {
    const listeners = this.events.get(event);
    if (!listeners) return this;

    // Match the original function for once() wrappers too
    const index = listeners.findIndex((l) => l === listener || l.original === listener);
    if (index !== -1) listeners.splice(index, 1);
    if (listeners.length === 0) this.events.delete(event);
    return this;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      this.off(event, wrapper);  // remove first, so it can't fire twice
      listener.apply(this, args);
    };
    wrapper.original = listener;
    return this.on(event, wrapper);
  }

  emit(event, ...args) {
    const listeners = this.events.get(event);

    if (!listeners) {
      if (event === 'error') {
        throw args[0] instanceof Error ? args[0] : new Error('unhandled error event');
      }
      return false;
    }

    // Copy, so a listener can safely remove itself during the loop
    for (const listener of [...listeners]) {
      listener.apply(this, args);
    }
    return true;
  }

  listenerCount(event) {
    return this.events.get(event)?.length ?? 0;
  }
}

const emitter = new MyEmitter();
const greet = (name) => console.log('hello', name);

emitter.on('greet', greet);
emitter.once('greet', (name) => console.log('once', name));

emitter.emit('greet', 'Asha');            // hello Asha, then once Asha
emitter.emit('greet', 'Ravi');            // hello Ravi
emitter.off('greet', greet);
console.log(emitter.emit('greet', 'x'));  // false
console.log(emitter.listenerCount('greet')); // 0

try {
  emitter.emit('error', new Error('boom'));
} catch (err) {
  console.log('thrown:', err.message);    // thrown: boom
}
```

**Complexity:** `emit` is O(listeners), `on` is O(1), and `off` is O(listeners).

**Follow-ups:**
- *Why copy the array in `emit`?* Removing a listener during the loop would otherwise skip the next one.
- *Why does `once` need `.original`?* So that `off(event, fn)` still works on `once` listeners.
- *How would you add async support?* Add `emitAsync`, which awaits `Promise.all` of listener results.

---

## 73. Production Scenarios

Use one structure for all of these: **clarify → mitigate → investigate → fix → prevent**.

### Scenario 1: "The API is slow"

**Clarify:** All routes, or one? Since when? After a deploy? p50, or only p99?

**Mitigate:** Roll back a recent deploy, scale out, or shed non-critical traffic.

**Investigate:**

| Check | If true | Next step |
|---|---|---|
| All routes slow, event loop delay high | Blocked loop | CPU profile |
| CPU near limit | Compute or GC | Flame graph, GC metrics |
| CPU low, one route slow | Waiting | Trace to the slow span |
| `pool.waitingCount` > 0 | Pool exhausted | Slow queries, leaks |
| Slow span is a dependency | Upstream issue | Timeout, breaker, fallback |
| CPU throttled | Limit too low | Review CPU limits |

**Sample answer:** "I'd first check whether a recent deploy lines up, and roll back if so. Then I'd compare event loop delay and CPU. If those are healthy, I'd follow a slow trace, because the time is probably in a query or an upstream call. Once it's fixed, I'd add an alert on that signal."

**Prevent:** Per-route p99 alerts, timeouts on all calls, load tests in CI.

### Scenario 2: "Memory keeps growing"

**Clarify:** Is it a sawtooth with a rising floor? Is it `heapUsed` or `rss`? Does it follow traffic?

**Mitigate:** Staggered restarts buy time. Raising limits a little can help, but note that it's temporary.

**Investigate:**

```text
1. Compare heapUsed, external and rss over time
2. heapUsed rising → three heap snapshots, compare, follow retainers
3. external or rss rising → Buffers, streams not closed, native addons
4. Compare with deploys, to find the change that introduced it
5. Reproduce with a soak test
```

**Common causes:** unbounded `Map`, per-request listeners, uncleared timers, unbounded queues, high-cardinality metrics.

**Sample answer:** "I'd confirm the floor keeps rising after GC. Then I'd take snapshots from a pod out of rotation, compare them, and see which objects grow and what retains them. After the fix, I'd run a soak test and keep a heap-growth alert."

**Prevent:** Bound every cache and queue, and run soak tests before big releases.

### Scenario 3: "The service keeps crashing"

**Clarify:** What's the exit code? Is it all pods or some? Is there a pattern?

**Mitigate:** Roll back. If one input triggers it, block that input.

**Investigate:**

| Evidence | Meaning | Next step |
|---|---|---|
| Exit 137, `OOMKilled` | Memory limit exceeded | Scenario 2 |
| `heap out of memory` | V8 heap limit reached | Heap analysis |
| Exit 1 with a stack trace | Uncaught error | Fix the bug or handler |
| Exit 143 | `SIGTERM` | Normal stop, or eviction |
| Liveness probe failures | Blocked loop or slow boot | Event loop delay |
| Exit 139, segfault | Native addon | Check native dependencies |
| Crash on boot | Config or dependency | Validate env, check dependency |

```bash
kubectl describe pod orders-api-abc     # exit code, reason, events
kubectl logs orders-api-abc --previous  # logs from the crashed container
```

```bash
# Capture evidence automatically
node --report-uncaught-exception --report-on-fatalerror \
     --report-directory=/var/reports app.js
```

**Sample answer:** "I'd check the exit code and previous container logs first, since 137 and exit 1 lead to different places. For an uncaught error, I'd fix the bug and add a test. For a crash loop after a deploy, I'd roll back first."

**Prevent:** Lint for floating promises, error listeners on streams and sockets, alerts on restart count.

### Scenario 4: "We lost events"

**Clarify:** Which events? Between which systems? Were they lost, or just late?

**Investigate:** Look for a dual write, acks before processing, and consumer crashes. Check the DLQ.

**Fix:** Use the outbox pattern and ack after processing (Topics 40, 58). Add a reconciliation job for critical data.

### Scenario 5: "Latency spikes after every deploy"

**Clarify:** How long does it last? Is it only on new pods?

**Likely causes:** cold JIT, empty caches, connection storms, or requests being routed before pods are ready.

**Fix:** Use readiness gates, warm-up requests, a slower rollout, and a connection pool that opens gradually.

---

## 74. Node.js System Design

Use this flow: **requirements → scale estimate → API → architecture → data → bottlenecks → failures → trade-offs.**

### Design 1: Chat app

**Requirements:** 1:1 and group chat, delivery status, offline messages, history, and presence.

**Scale estimate:** 1M concurrent users. At about 50,000 connections per server, you need roughly 20 servers, plus headroom.

```mermaid
flowchart LR
    C["Clients"] --> L["Load balancer"]
    L --> G1["WS gateway 1"]
    L --> G2["WS gateway 2"]
    G1 --> K["Kafka: messages"]
    G2 --> K
    K --> M["Message service"]
    M --> D["Cassandra or DynamoDB"]
    M --> R["Redis: routing and presence"]
    R --> G1
    R --> G2
    M --> P["Push notifications"]
```

```javascript
// Gateway: durable first, then acknowledge
socket.on('send', async (msg, ack) => {
  const message = {
    id: msg.clientId,              // client-generated, for idempotency
    conversationId: msg.conversationId,
    senderId: socket.user.id,      // from auth, never from the client
    body: msg.body,
    sentAt: Date.now(),
  };

  await kafka.send({
    topic: 'messages',
    messages: [{ key: message.conversationId, value: JSON.stringify(message) }],
  });

  ack({ status: 'accepted', id: message.id });
});

// Presence: a key with a TTL, refreshed by heartbeats
setInterval(() => {
  redis.set(`presence:${userId}`, gatewayId, { EX: 30 });
}, 10_000);

// Delivery: find the user's gateway, or fall back to push
async function deliver(userId, message) {
  const gateway = await redis.get(`presence:${userId}`);
  if (gateway) await redis.publish(`gateway:${gateway}`, JSON.stringify({ userId, message }));
  else await push.send(userId, message);
}
```

**Key decisions:**
- WebSockets for two-way, low-latency traffic.
- Kafka keyed by conversation keeps order within a chat.
- A wide-column store, partitioned by conversation and sorted by time, fits history reads.
- Client-generated IDs let retries be deduplicated.

**Failure handling:** A gateway crash drops connections, so clients reconnect with jitter and fetch messages since their last seen ID.

**Trade-offs:** Large groups cause write amplification. Fan out on read for very big channels.

### Design 2: Notification service

**Requirements:** email, SMS and push; user preferences; templates; retries; no duplicates; priorities; and rate limits per provider.

**Scale estimate:** 50M notifications per day is about 600 per second on average. Peaks can be 10× that.

```mermaid
flowchart LR
    S["Services"] --> A["Notification API"]
    A --> P["Preferences and dedupe"]
    P --> Q1["High priority queue"]
    P --> Q2["Bulk queue"]
    Q1 --> W["Channel workers"]
    Q2 --> W
    W --> E["Email provider"]
    W --> M["SMS provider"]
    W --> N["Push provider"]
    W --> D["Delivery log"]
    W --> L["Dead-letter queue"]
```

```javascript
app.post('/notifications', async (req, res) => {
  const { userId, template, data, idempotencyKey, priority = 'bulk' } = req.body;

  // Atomic dedupe
  const fresh = await redis.set(`notify:${idempotencyKey}`, '1', { NX: true, EX: 86400 });
  if (!fresh) return res.status(200).json({ status: 'duplicate' });

  const preferences = await getPreferences(userId);
  const channels = allowedChannels(template, preferences);

  await Promise.all(channels.map((channel) => queues[priority].add(channel, {
    userId, template, data, channel,
  }, {
    attempts: 5,
    backoff: { type: 'exponential', delay: 2000 },
    jobId: `${idempotencyKey}:${channel}`,
  })));

  res.status(202).json({ status: 'queued' });
});

new Worker('high', async (job) => {
  const { channel, userId, template, data } = job.data;
  const content = await renderTemplate(template, data, await userLocale(userId));

  await providerLimiter[channel].run(() =>        // respects provider limits
    providers[channel].breaker.fire(userId, content),
  );

  await deliveryLog.record(job.id, 'sent');
}, { connection, concurrency: 50 });
```

**Key decisions:**
- Separate queues, so a marketing blast can't delay OTPs.
- A worker pool per channel isolates failures.
- A fallback provider sits behind each circuit breaker.
- Unsubscribe and quiet-hour rules run before sending.

**Failure handling:** Retries with backoff, then a DLQ. Provider webhooks update delivery status.

**Trade-offs:** External providers can still double-send on a timeout, so aim for effectively-once with provider idempotency keys where they exist.

### Design 3: File processing pipeline

**Requirements:** uploads up to 5 GB, virus scanning, format conversion, status tracking, retries and no blocking of API traffic.

```mermaid
flowchart LR
    C["Client"] -->|"1. Get URL"| A["API"]
    C -->|"2. Upload"| S["S3 incoming"]
    S -->|"3. Event"| Q["SQS"]
    Q --> W["Workers"]
    W --> V["Scan"]
    V --> T["Transform"]
    T --> O["S3 processed"]
    W --> D["Status database"]
    D --> A
    Q --> L["DLQ"]
```

```javascript
// Stage 1: API creates a record and a presigned URL
app.post('/files', requireAuth, async (req, res) => {
  const id = randomUUID();
  const key = `incoming/${req.user.id}/${id}`;

  await db.files.insert({ id, userId: req.user.id, key, status: 'uploading' });

  const url = await getSignedUrl(s3, new PutObjectCommand({ Bucket: BUCKET, Key: key }), {
    expiresIn: 900,
  });

  res.status(201).json({ id, url });
});

// Stage 2: the worker streams through processing
async function processFile({ key, id }) {
  // Conditional update claims the job, so redelivery can't run it twice
  const claimed = await db.files.updateWhere(
    { id, status: 'uploading' },
    { status: 'processing' },
  );
  if (!claimed) return;

  const { Body } = await s3.send(new GetObjectCommand({ Bucket: BUCKET, Key: key }));

  await scanForViruses(key);

  const output = new PassThrough();
  const upload = new Upload({
    client: s3,
    params: { Bucket: BUCKET, Key: key.replace('incoming/', 'processed/'), Body: output },
  });

  await Promise.all([
    pipeline(Body, transformStream(), output),    // constant memory
    upload.done(),
  ]);

  await db.files.update(id, { status: 'ready' });
}
```

**Key decisions:**
- Presigned uploads keep bytes away from the API.
- Queues absorb spikes and allow workers to scale on depth.
- Streams keep memory flat for any file size.
- CPU-heavy conversion uses child processes such as `ffmpeg`, or worker threads.

**Failure handling:** Idempotent claims, heartbeats that extend the SQS visibility timeout, and a DLQ with status set to `failed`.

**Trade-offs:** The process is asynchronous, so clients need polling, SSE or a webhook to learn the outcome.

---

## 75. Cheat Sheet

### Core model

| Concept | Remember |
|---|---|
| Node | V8 + libuv + standard library |
| Thread | One main JS thread; 4 libuv pool threads by default |
| Good for | I/O |
| Bad for | CPU work on the main thread |
| Loop phases | Timers → pending → idle → poll → check → close |
| Microtasks | After each callback: `nextTick`, then Promises |
| I/O callback | Immediate beats timeout |
| Thread pool | `fs`, `dns.lookup`, some crypto, zlib |

### Pick the right tool

| Need | Use |
|---|---|
| I/O concurrency | `async/await`, `Promise.all` |
| Limit concurrency | Pool or `p-limit` |
| CPU-heavy JS | Worker thread pool |
| Other programs | `spawn` or `execFile` |
| Multi-core | Cluster or PM2, or pods in Kubernetes |
| Slow background work | BullMQ, SQS |
| Large data | Streams with `pipeline` |
| Request context | `AsyncLocalStorage` |
| Cancellation | `AbortSignal.timeout` |
| Server push | SSE |
| Two-way real time | WebSocket + Redis backplane |
| Large uploads | S3 presigned URLs |
| Internal RPC | gRPC |
| Public API | REST |
| Flexible clients | GraphQL + DataLoader |
| Event replay | Kafka |
| Task routing | RabbitMQ |

### Promises

| Method | Result |
|---|---|
| `all` | All succeed, or first failure |
| `allSettled` | Every outcome |
| `race` | First settled |
| `any` | First success |

### Production defaults

```text
Timeouts on every outgoing call
Retries: limited, backoff with jitter, idempotent only
keepAliveTimeout > load balancer idle timeout
Pool total = pods × pool size, under the DB limit
Heap limit ≈ 75% of container memory
Handle SIGTERM, with a forced exit before SIGKILL
Liveness simple, readiness checks dependencies
exec-form CMD, non-root, slim multi-stage image
npm ci, commit lockfile, ignore install scripts
Logs as JSON to stdout, no secrets
Metrics with low-cardinality labels
Bound every cache and queue
```

### Security checklist

```text
Validate input at the boundary
Parameterize all queries
Check ownership on every object
Argon2id, scrypt or bcrypt for passwords
crypto.randomBytes for tokens
HttpOnly, Secure, SameSite cookies
Short JWTs, rotating refresh tokens
Allow-list CORS origins
Helmet headers
Rate limit by user and IP
Block private IPs for user URLs
Guard against __proto__
No nested regex quantifiers on user input
No user input in exec()
Secrets in a manager, never in git
```

### Debugging quick map

| Symptom | First look |
|---|---|
| All routes slow | Event loop delay |
| One route slow | Traces, slow queries |
| Low CPU, high latency | Pool waits, upstream, throttling |
| High CPU | CPU profile, GC |
| Rising memory floor | Heap snapshots |
| `rss` up, heap flat | Buffers, native code |
| Random 502s | Keep-alive timeouts |
| Exit 137 | OOM or forced kill |
| Exit 1 | Uncaught error |
| Failed requests on deploy | Graceful shutdown |
| `EMFILE` | Too many open files |
| `ECONNRESET` | Peer closed the connection |
| `ERR_HTTP_HEADERS_SENT` | Double response |
| `MaxListenersExceededWarning` | Listener leak |

### Status codes

```text
200 OK                201 Created             202 Accepted       204 No Content
400 Bad Request       401 Not Authenticated   403 Forbidden      404 Not Found
409 Conflict          422 Invalid Data        429 Rate Limited
500 Server Error      502 Bad Gateway         503 Unavailable    504 Timeout
```

### Patterns in one line

| Pattern | One line |
|---|---|
| Singleton | Module cache gives one instance per process |
| Factory | Choose the implementation at runtime |
| DI | Pass dependencies in |
| Repository | Hide data access |
| Middleware | Chain of responsibility |
| Observer | `EventEmitter` |
| Circuit breaker | Fail fast when a dependency is down |
| Bulkhead | Isolate resources per dependency |
| Saga | Local steps with compensation |
| Outbox | Commit event with data |
| CQRS | Separate reads and writes |
| Cache-aside | App loads on miss, deletes on write |

### Answer framework for any senior question

1. Give a one-line answer.
2. Explain how it works.
3. Share a real production example.
4. State the trade-off.
5. Say when you wouldn't use it.
