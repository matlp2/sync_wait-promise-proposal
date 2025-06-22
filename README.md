# Stage 0 Proposal: `sync_wait`

## Authors
MatLP & ChatGPT

## Problem

JavaScript requires all asynchronous behavior to be handled via `await`, which must be inside an `async` function. This breaks stack traces, complicates debugging, and makes exploratory code and prototyping difficult.

In practice, developers often want to *intentionally block* until a Promise resolves, especially when debugging or writing setup/testing/tooling scripts.

## Motivation

- WebGPU buffer reading
- Debugging GPU state
- Teaching async APIs
- Scripting tools (like WebGL/WebGPU tests)
- Startup code for environments like Deno, Bun, or Node.js

## Proposal

Introduce a new keyword `sync_wait`, which behaves like `await`, but can be used **outside async functions**, and blocks synchronously until the promise resolves or rejects.

```js
const result = sync_wait(someAsyncOperation());
```


## Why JavaScript Needs sync_wait
1. Improves Debuggability
 - Avoids async stack traces and step-over issues in debuggers.
 - Allows full variable inspection across the call stack without async indirection.
 - Makes debugging async-heavy APIs (like WebGPU or Streams) much easier.

2. Optimize with Async Contagion Only if Needed
- Developers can write simple, top-down, synchronous code in scripts, REPLs, notebooks, or browser DevTools.
- Avoids the need to prematurely restructure entire call chains into `async` functions due to a single async operation — a problem known as **async contagion**.
- With `sync_wait`, async complexity is introduced only when and where performance demands it.

3. Reduces Boilerplate in Scripting and Tooling
 - Many CLI tools and build scripts need to wait for async operations (fetch, readFile).
 - sync_wait would allow writing these tools without restructuring everything as async logic.

4. Enables Easier Testing
 - Testing frameworks or manual test scripts can synchronously call async operations.
 - Simplifies writing setup/teardown code when Promise-handling isn't the focus.

5. Simplifies Educational Code
 - New developers can write intuitive, imperative code before learning async/await.
 - Useful in teaching environments and tutorials involving fetch, GPU APIs, etc.

6. Aligns with Other Languages
 - Python and Go allow blocking network and file I/O — enabling rapid scripting and debugging.
 - JavaScript already does this in Node.js (readFileSync) and alert-like browser APIs.
 - Blocking is available in scripting-friendly languages like Python, Go, Ruby.

7. Defers Optimization to the Right Time
 - Developers can start with sync_wait to get working logic quickly, then refactor to async later when performance or architectural constraints matter.
 - This avoids premature async abstraction and lets teams build and validate real functionality first.

8. Reduces Pressure on Library Authors
 - Without sync_wait, library maintainers often feel forced to offer both async and sync versions of each function (e.g., readFile + readFileSync in Node.js).
 - With sync_wait, authors can expose only the async version, and let users synchronously wait on it if they choose, eliminating unnecessary API duplication.

9. Prior Art and Real-World Need
 - Node.js: Added sync file I/O due to strong developer demand (readFileSync, existsSync, etc.).

10. Already in Use in the JavaScript Ecosystem  
- **Browsers**: Blocking calls like `prompt()`, `alert()`, and `confirm()` have existed for decades and are used where synchronous interaction is useful and intentional.
- **Node.js**: Provides synchronous versions of file system operations (e.g., `readFileSync`, `existsSync`) for scripting and early-stage development needs.
- These precedents show that **intentional blocking behavior is acceptable and useful** in many real-world situations.
- `sync_wait` follows this same philosophy: **explicit, controlled blocking** where it's developer-initiated and ergonomically beneficial.

11. Some False Reasons to Avoid Sync
 - **UI freezes**: **any `while` loop** can completely block the browser’s main thread and freeze the UI — this is known as a **main-thread hang**. Yet we don't ban `while` loops — we trust developers to use them responsibly, or provide tooling to warn them.
 - **Timing Attacks**: Some may raise concerns that sync_wait could enable timing attacks by allowing a Promise to be awaited synchronously and measuring how long it takes to resolve. However, this concern is not unique to sync_wait — it already applies to all existing asynchronous behavior in JavaScript:
    + A developer can record timestamps before and after any await or .then() callback to measure how long a Promise took to resolve.
    + Even a micro-task or frame delay can be measured using high-resolution timers (e.g., performance.now() in some contexts).
    + If timing a secret-dependent operation is possible, it can already be exploited with async APIs — with or without sync_wait. If blocking synchronous resolution were a security risk, then the same would apply to await, .then(), and even setTimeout patterns.
 - **JavaScript is “Asynchronous by Design”**: Claiming that JavaScript is “asynchronous by design” and therefore must enforce restrictive tooling for handling async operations is misguided. Such restrictions only force developers to avoid async whenever possible due to the complexity it introduces in testing, debugging, and maintaining control flow. This often leads to convoluted code and frustrated developers. Instead, JavaScript should aim to be “nice and practical by design”—providing developers with flexible, explicit tools like `sync_wait` that make async programming more intuitive, easier to debug, and better aligned with real-world development needs.
   
12. `sync_wait` Is Explicit and Cannot Be Used by Mistake
 - One of the strengths of sync_wait is that it is intentionally and unmistakably synchronous. Unlike accidentally blocking the main thread with a poorly written while loop or a large synchronous JSON parse, the use of sync_wait would be deliberate and self-documenting. Its name makes it immediately clear to the reader — and to the developer — that this is a blocking operation.
 - This makes sync_wait fundamentally different from hidden performance traps. It would only be used where the developer has decided that synchronous waiting is the right tradeoff — for example, in tooling, testing, GPU debugging, or one-off scripts. Like readFileSync, it signals intent at the call site, making its use both readable and auditable.
 - Rather than hiding synchronous behavior, sync_wait makes it explicit, traceable, and opt-in — exactly the kind of construct responsible developers rely on in practical software development.



