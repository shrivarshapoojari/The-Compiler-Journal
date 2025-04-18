---
title: "Is setTimeout(fn, 0) a Lie? What Timers in Node.js Actually Mean"
date: 2025-04-16
description: "A deep dive into how timers actually work in Node.js. Understand what setTimeout(fn, 0) really does, explore the event loop phases, and bust the myths around zero-delay timers."
tags: ["nodejs", "timers", "event loop", "setTimeout", "async", "javascript"]
---



> We’ve all used `setTimeout(fn, 0)` to defer code execution. But does it really run "immediately" after the current call stack? Or is it a clever trick?

This blog post dives deep into the Node.js timer internals, the event loop, and how `setTimeout` really behaves under the hood. You’ll never look at `setTimeout(fn, 0)` the same way again.

---

## 🧠 TL;DR
- `setTimeout(fn, 0)` does not execute `fn` immediately.
- The minimum delay is actually ~1ms or more, not 0.
- The callback is queued into the Timers phase of the Node.js Event Loop.
- It waits for other microtasks, I/O, and phases before it runs.

---

## 🔁 The Misconception

### What many developers think:
```js
setTimeout(() => console.log("Hi!"), 0);
```
Expected behavior:
- Executes right after current synchronous code.

Reality:
- There’s a delay. It’s queued for the next tick of the event loop, after several other phases.

---

## 🧬 What Actually Happens in Node.js

### The Timer API
When you do:
```js
setTimeout(() => console.log("callback"), 0);
```
Node:
1. Registers the callback in a **timer queue** via libuv.
2. It calculates the delay based on a clock (usually `Date.now()` or performance timers).
3. Internally schedules it to run at the **Timers** phase of the Event Loop.

If you specify `0`, it doesn't mean "right now" — it means "run after a minimum of ~1ms, depending on how busy the loop is."

---

## ⛓ Event Loop Phases Recap

Node.js event loop phases:

1. Timers (setTimeout, setInterval)
2. Pending Callbacks
3. Idle/Prepare (internal)
4. Poll (I/O)
5. Check (setImmediate)
6. Close Callbacks

Each phase processes its own queue of callbacks.

After every phase, Node processes:
- Microtasks: process.nextTick(), Promise callbacks

---

## 📚 Example: Zero Delay Is Not Zero

```js
console.log("Start");

setTimeout(() => console.log("Timeout 0ms"), 0);

Promise.resolve().then(() => console.log("Promise resolved"));

process.nextTick(() => console.log("Next Tick"));

console.log("End");
```

Output:
```
Start
End
Next Tick
Promise resolved
Timeout 0ms
```

Why?
- `process.nextTick` and Promises run before `setTimeout` because they’re in the microtask queue.
- `setTimeout` gets queued into the **Timers** phase.

---

## 📊 Timer Internals: Delay Calculation

The timer system in libuv uses:
- A red-black tree sorted by deadline time.
- A loop checks for expired timers and executes callbacks.

Even with `0`, timers are only checked at intervals (usually every 1ms).

### Delay May Be Affected By:
- System load
- Number of callbacks already queued
- CPU performance
- Event loop starvation

---

## 🛠 Real-Life Implication

### Don't assume `setTimeout(fn, 0)` is reliable for immediate execution.

Instead, use:
- `process.nextTick()` for super high priority (but be cautious — can starve the event loop)
- `Promise.resolve().then()` for microtasks

```js
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
setTimeout(() => console.log("timeout"), 0);
```

---

## 🧪 Deeper Example: Compare with setImmediate

```js
setTimeout(() => console.log("Timeout"), 0);
setImmediate(() => console.log("Immediate"));
```

Depending on context, either one can log first.

Inside an I/O callback:
```js
fs.readFile(__filename, () => {
  setTimeout(() => console.log("Timeout inside I/O"), 0);
  setImmediate(() => console.log("Immediate inside I/O"));
});
```

Output:
```
Immediate inside I/O
Timeout inside I/O
```

This is because after I/O, Node jumps directly to the Check phase, where `setImmediate` resides.

---

## ⚠️ Edge Cases and Pitfalls

- Using many `process.nextTick()`s can block the event loop.
- `setTimeout(fn, 0)` is still not truly zero-delay — always think of it as "defer to later."
- Timer accuracy is not real-time. Use worker threads or native timers if millisecond precision is crucial.

---

## 🧰 Summary Table

| Method              | Queue Type   | When It Runs                          |
|---------------------|--------------|---------------------------------------|
| process.nextTick() | Microtask    | After current operation, before Promises |
| Promise.then()     | Microtask    | After current operation, after nextTick |
| setTimeout(fn, 0)  | Macrotask    | Timers phase in next event loop cycle |
| setImmediate()     | Macrotask    | Check phase after I/O                 |

---

## 🧠 Pro Tips

- For short deferral, use Promises or nextTick
- For post-I/O deferral, use setImmediate
- Avoid chaining nextTick excessively

---

## 🛎 Conclusion

`setTimeout(fn, 0)` is not a lie — but it's not what it seems either. It doesn’t run immediately, and it’s queued behind microtasks, I/O callbacks, and more.

To truly master Node.js performance and async behavior, always understand where your function is scheduled in the event loop. Don’t just code — know when it runs.


---

## 📚 Further Reading

- [Node.js Event Loop Guide](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- [What the heck is the event loop anyway? - Philip Roberts (JSConf)](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [Node.js libuv source code](https://github.com/libuv/libuv)

---

💬 *Feel free to connect with me to discuss any project ideas or for collaboration [Connect](https://www.shrivarshapoojary.in/contact)*

---
