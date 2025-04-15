---
title: "Node.js Event Loop: Deep Dive into the Heart of Async"
date: 2025-04-15
description: "Explore the inner workings of Node.js's Event Loop — the engine that powers non-blocking I/O and asynchronous JavaScript execution."
tags: ["nodejs", "event loop", "javascript", "async", "libuv", "deep dive"]
---


The Event Loop is the backbone of Node.js’s non-blocking architecture. It’s the reason why a single-threaded language like JavaScript can handle thousands of concurrent I/O operations.

This guide will take you through every layer of the Event Loop — from timers and the poll phase to libuv and thread pools — in excruciating detail.

---
## 🔄 What is the Event Loop?

The Event Loop is the mechanism that coordinates and handles the execution of callbacks in JavaScript, especially for asynchronous operations.

While JavaScript itself is single-threaded, the runtime environment (Node.js or browsers) gives it superpowers by managing async work outside the main thread.

---

## ❓ Why the Event Loop Matters

JavaScript has no `sleep()` or `wait()` — it’s designed for responsiveness. The Event Loop allows:

- Non-blocking file I/O
- Concurrency without threads
- Scalable web servers
- Async control flow using promises, async/await

Without it, Node.js would freeze every time a database query or file read occurred.

---

## ⚙️ libuv: The Unsung Hero

Node.js relies on an open-source C library called **libuv** to handle:

- File I/O
- TCP/UDP
- Child processes
- Timers
- Thread pools
- Signals

**Key Idea:** JavaScript never actually touches threads — libuv does the heavy lifting.

libuv internally uses:

- epoll (Linux)
- kqueue (BSD/macOS)
- IOCP (Windows)

These are efficient, OS-level mechanisms for waiting on multiple I/O operations.

---

## 📁 Event Loop Phases

Each iteration of the Event Loop is called a **tick**. Each tick is divided into multiple **phases**, and each phase has a **FIFO queue** of callbacks.

### 🕐 Phases (in order):

1. **Timers Phase**
2. **Pending Callbacks Phase**
3. **Idle, Prepare Phase**
4. **Poll Phase**
5. **Check Phase**
6. **Close Callbacks Phase**
7. **Microtasks (nextTick and Promises)**

---

### 🧭 Phase 1: Timers

This phase handles:

- `setTimeout(fn, delay)`
- `setInterval(fn, delay)`

These are **scheduled**, not exact. Delay is a *minimum*, not a guarantee.

Example:

```js
setTimeout(() => {
  console.log('Timeout after 100ms');
}, 100);
```

If previous phases took 200ms, this runs after 200ms.

---

### 📨 Phase 2: Pending Callbacks

This phase executes certain system-level I/O callbacks. For example:

- TCP errors
- Some DNS errors

You don’t usually interact with this phase directly.

---

### 😴 Phase 3: Idle, Prepare

Used internally by libuv to prepare for the poll phase. Not exposed to users.

---

### 📡 Phase 4: Poll

This is the **heart** of the Event Loop.

- Waits for I/O events (file reads, network)
- Executes callbacks from completed I/O operations
- If no events: it blocks (waits)
- If events in queue: processes them
- If timeout set by timers phase: breaks early

Example:

```js
const fs = require('fs');

fs.readFile('file.txt', () => {
  console.log('File read complete');
});
```

The read is queued in libuv. Once done, its callback is added to the poll queue.

---

### ⏩ Phase 5: Check

This phase runs `setImmediate()` callbacks.

Example:

```js
setImmediate(() => {
  console.log('This runs in the check phase');
});
```

Useful for deferring tasks until after poll phase.

---

### 🔒 Phase 6: Close Callbacks

Handles close events like:

- `socket.on('close', ...)`
- `process.on('exit', ...)`

---

### 🧬 Microtasks: NextTick & Promises

These don’t belong to phases but are run **between** phases:

- `process.nextTick()` – highest priority
- `Promise.then()` – after nextTick

```js
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));
setTimeout(() => console.log('Timeout'), 0);
```

Output:
```
nextTick
Promise
Timeout
```

---

## ⏱️ Timers vs Immediate

```js
setTimeout(() => console.log('Timeout'), 0);
setImmediate(() => console.log('Immediate'));
```

Who wins?

- If in the main module: likely `Timeout` first.
- If inside I/O callback: `Immediate` first.

Why?

Because when the I/O ends, we are in the **poll** phase, and **check** (immediate) runs next.

---

## 🔄 I/O and Thread Pools

Node.js is single-threaded **only** at the JS level.

libuv uses a thread pool (default: 4 threads) for:

- File system (`fs`)
- DNS lookups
- Crypto
- zlib compression

You can increase it via:

```bash
UV_THREADPOOL_SIZE=8 node app.js
```

Example:

```js
const crypto = require('crypto');

for (let i = 0; i < 10; i++) {
  crypto.pbkdf2('pass', 'salt', 100000, 512, 'sha512', () => {
    console.log(`Done ${i}`);
  });
}
```

This will run 4 at a time, queue the rest.

---

## ⚠️ CPU-bound Blocking

Node.js shines for I/O-heavy apps, not CPU-bound.

Bad:

```js
while (true) {}
```

Freezes everything.

Solution:

- Use Worker Threads (since Node 10.5+)
- Offload to external service

```js
const { Worker } = require('worker_threads');

new Worker(`
  while (true) {}
`, { eval: true });
```

---

## 🧪 Event Loop Execution Example

```js
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));

fs.readFile(__filename, () => {
  console.log('readFile callback');

  setTimeout(() => console.log('inner setTimeout'), 0);
  setImmediate(() => console.log('inner setImmediate'));
});
```

Output:
```
setTimeout
setImmediate
readFile callback
inner setImmediate
inner setTimeout
```

Why?

After `readFile`, we enter poll → check → timers.

---

## 🔬 Tools to Debug the Event Loop

### 1. Chrome DevTools

```bash
node --inspect-brk index.js
```

Use Chrome's `chrome://inspect`

### 2. Flamegraphs

```bash
npx 0x app.js
```

Visualize hotspots in your code.

### 3. `clinic` toolkit

```bash
npx clinic doctor -- node app.js
```

Gives you performance bottlenecks.

---

## ✅ Best Practices

- Avoid long-running synchronous functions
- Prefer `setImmediate` over `setTimeout(fn, 0)` inside I/O
- Be careful with `process.nextTick()` — it can block the loop
- Break large loops using `setImmediate`

Bad:

```js
for (let i = 0; i < 1e9; i++) {}
```

Good:

```js
function chunkedLoop(i = 0) {
  if (i >= 1e9) return;
  if (i % 1e5 === 0) console.log(i);
  setImmediate(() => chunkedLoop(i + 1));
}
chunkedLoop();
```

---



## 🔚 Conclusion

The Event Loop is the beating heart of Node.js.

Mastering it means:

- You write more efficient code
- You debug async issues faster
- You know when and why callbacks execute

Node.js isn’t just "async/await" magic. It’s a carefully orchestrated dance between JavaScript, libuv, and the underlying OS.

Learn the rhythm. And you’ll write code that scales like a beast.

---
