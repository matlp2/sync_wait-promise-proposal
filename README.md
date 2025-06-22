# Stage 0 Proposal: `sync_wait`

## Authors
LPMat

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
