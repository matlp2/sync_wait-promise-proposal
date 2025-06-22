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
const result = sync_wait someAsyncOperation();
```


## Why JavaScript Needs `sync_wait`
### 1. Improves Debuggability
 - Avoids async stack traces and step-over issues in debuggers.
 - Allows full variable inspection across the call stack without async indirection.
 - Makes using async-heavy APIs much easier.

<br>

### 2. Optimize with "Async Contagion" Only if Needed
- Developers can choose to write simple, top-down, synchronous code.
- Avoids the need to prematurely restructure entire call chains into `async` functions due to a single async operation — a problem known as **async contagion**.
- Avoids premature async abstraction and lets teams build and validate real functionality first.
- Async complexity can be introduced later only where performance demands it.

<br>

### 3. Enables Easier Testing
 - Testing frameworks or manual test scripts can synchronously call async operations.
 - Simplifies writing setup/teardown code when Promise-handling isn't the focus.

<br>

### 4. Reduces Pressure on Library Authors
 - Without `sync_wait`, library maintainers often feel forced to offer both async and sync versions of each function (e.g., readFile + readFileSync in Node.js).
 - With `sync_wait`, authors can expose only the async version, and let users synchronously wait on it if they choose, eliminating unnecessary API duplication.

<br>

### 5. Already in Use in the JavaScript Ecosystem  
- **Browsers**: Blocking calls like `prompt()`, `alert()`, and `confirm()` have existed for decades and are used where synchronous interaction is useful and intentional.
- **Node.js**: Provides synchronous versions of file system operations (e.g., `readFileSync`, `existsSync`).
<br>These precedents show that **intentional blocking behavior is acceptable and useful** in many real-world situations.
<br>`sync_wait` follows this same philosophy: **explicit, controlled blocking**.

<br>

### 6. `sync_wait` Is Explicit and Cannot Be Used by Mistake
 - One of the strengths of `sync_wait` is that it is intentionally and unmistakably synchronous. Unlike accidentally blocking the main thread with a poorly written while loop or a large synchronous JSON parse, the use of `sync_wait` would be deliberate and self-documenting. Its name makes it clear to the reader that this is a blocking operation making its use both readable and auditable.
 - This makes `sync_wait` fundamentally different from hidden performance traps. It would only be used where the developer has decided that synchronous waiting is the right tradeoff — for example, in tooling, testing, GPU debugging, or one-off scripts.
 - Rather than hiding synchronous behavior, `sync_wait` makes it explicit, traceable, and opt-in — exactly the kind of construct responsible developers rely on in practical software development.

<br>

### 7. Discussing Common Objections — and Why They Don't Hold
 - **UI freezes**: Any `while` loop can completely block the browser’s main thread and do **main-thread hang** yet they are here to enable developers to implements their desired processes. The more general idea of making programmers totally incapable of ever doing something -just because it is slitly unsound relatively to an hyper general indulged-in paradigm- is what this argument is. This is what appened with not allowing top level awaits in script for the same reason, and then allowing it for module script, because actually it is practical even if thurther UI elements won't render until all top level awaits finishes.
 - **Alternatives Exist such as using Atomics.wait() in SharedArrayBuffer within a WebWorker**: Wrapping your entire solution in a WebWorker and get significant architectural complexity just to avoid async functions is indeed very tempting. But the goal of the this proposal is to make `sync_wait` widely available. `sync_wait` is about practicality: giving developers a clean, first-class tool to handle async behavior anywhere without friction.
 - **Timing Attacks**: Some may raise concerns that `sync_wait` could enable timing attacks by allowing a Promise to be awaited synchronously and measuring how long it takes to resolve. However, this concern is not unique to `sync_wait` — it already applies to all existing asynchronous behavior in JavaScript:
    + A developer can record timestamps before and after any await or .then() callback to measure how long a Promise took to resolve.
    + Even a micro-task or frame delay can be measured using high-resolution timers (e.g., performance.now() in some contexts).
    + If timing a secret-dependent operation is possible, it can already be exploited with async APIs — with or without `sync_wait`. If blocking synchronous resolution were a security risk, then the same would apply to await, .then(), and even setTimeout patterns.
 - **JavaScript is “Asynchronous by Design”**: Claiming that JavaScript is “asynchronous by design” and therefore must enforce restrictive tooling for handling async operations is misguided. Such restrictions only force developers to avoid async whenever possible due to the complexity it introduces in testing, debugging, and maintaining control flow. Having `sync_wait` better aligns with the expectations of a language designed for asynchronous programming: it makes it more asynchronous-focused not less.

<br>

### 8. Most Projects Never Reach a Usable Product
 - In real-world development, only a small fraction of projects make it to a usable, maintained product. The vast majority are prototypes, internal tools, abandoned explorations, or learning efforts. JavaScript has historically thrived in this space because it’s easy to write, easy to debug, and fast to iterate — making it one of the most efficient languages for turning ideas into working software.
<br>However, async contagion undermines this strength. When a single async operation forces every calling function to become async, the project’s entire control flow becomes fragmented. This can ruin the debuggability of the codebase permanently — even if the rest of the logic is otherwise simple, testable, and synchronous. In practice, this often makes entire projects unable to use debugging features.
<br>`sync_wait` is pragmatic, debuggable and developer-first.


<br>

### 9. A Feature Expected from Developpers
 - By contrast with JavaScript, scripting-friendly languages like Python, Go, and Ruby use blocking by default, with async behavior as an opt-in feature — making them ideal for debugging.


