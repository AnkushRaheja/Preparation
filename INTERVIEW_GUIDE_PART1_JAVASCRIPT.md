# 📚 Part 1: JavaScript Foundations - Complete Mastery Guide

> **The Ultimate JavaScript Interview Preparation Guide**
> 
> This comprehensive guide covers everything you need to master JavaScript for technical interviews at any level - from junior to senior positions at top tech companies including FAANG, startups, and everything in between.

---

## 📋 Table of Contents

1. [JavaScript Engine & Runtime](#1-javascript-engine--runtime)
2. [Execution Context & Call Stack](#2-execution-context--call-stack)
3. [The Event Loop - Complete Deep Dive](#3-the-event-loop---complete-deep-dive)
4. [Scope & Scope Chain](#4-scope--scope-chain)
5. [Closures - Complete Mastery](#5-closures---complete-mastery)
6. [Hoisting & Temporal Dead Zone](#6-hoisting--temporal-dead-zone)
7. [The 'this' Keyword - All Binding Rules](#7-the-this-keyword---all-binding-rules)
8. [Prototypes & Inheritance](#8-prototypes--inheritance)
9. [ES6+ Features Complete Guide](#9-es6-features-complete-guide)
10. [Asynchronous JavaScript](#10-asynchronous-javascript)
11. [Error Handling](#11-error-handling)
12. [Memory Management](#12-memory-management)
13. [JavaScript Design Patterns](#13-javascript-design-patterns)
14. [Common Interview Coding Problems](#14-common-interview-coding-problems)
15. [Interview Questions Bank](#15-interview-questions-bank)

---

# 1. JavaScript Engine & Runtime

## 1.1 What is a JavaScript Engine?

A **JavaScript Engine** is a program that executes JavaScript code. Every browser has its own JavaScript engine:

| Browser | Engine |
|---------|--------|
| Chrome | V8 |
| Firefox | SpiderMonkey |
| Safari | JavaScriptCore (Nitro) |
| Edge | V8 (formerly Chakra) |
| Node.js | V8 |

### How the V8 Engine Works

The V8 engine (used in Chrome and Node.js) processes JavaScript in several stages:

```
Source Code → Parser → AST → Interpreter (Ignition) → Bytecode
                                    ↓
                            Optimizing Compiler (TurboFan)
                                    ↓
                            Optimized Machine Code
```

**Step-by-step breakdown:**

1. **Parsing**: The source code is parsed into tokens and then into an Abstract Syntax Tree (AST)
2. **Interpretation**: The Ignition interpreter converts the AST into bytecode
3. **Execution**: Bytecode is executed immediately
4. **Optimization**: Hot code paths are identified and sent to TurboFan for optimization
5. **Deoptimization**: If assumptions are invalidated, code is deoptimized back to bytecode

```javascript
// Example: How V8 optimizes this function
function add(a, b) {
  return a + b;
}

// First few calls - interpreted as bytecode
add(1, 2);      // V8 observes: numbers being added
add(3, 4);      // V8 observes: still numbers
add(5, 6);      // V8 marks this as "hot" code

// After many calls with numbers, TurboFan optimizes for number addition
// But if we suddenly do:
add("hello", "world");  // Deoptimization! Back to bytecode
```

### 1.2 Just-In-Time (JIT) Compilation

JavaScript uses **JIT Compilation** - a hybrid approach between interpretation and compilation:

```
Traditional Interpreter:
Source Code → Execute line by line (slow but fast startup)

Traditional Compiler:
Source Code → Machine Code → Execute (fast execution but slow startup)

JIT Compiler (JavaScript):
Source Code → Bytecode → Execute immediately
                    ↓
            Hot code detected
                    ↓
            Compile to machine code
                    ↓
            Execute optimized code (fast!)
```

**Why JIT?**
- **Fast startup**: Begin executing immediately without waiting for full compilation
- **Optimization**: Frequently executed code gets compiled to fast machine code
- **Adaptability**: Can deoptimize and reoptimize as code patterns change

### 1.3 The JavaScript Runtime Environment

The JavaScript Runtime is more than just the engine. It includes:

```
┌─────────────────────────────────────────────────────────────┐
│                    JavaScript Runtime                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   JavaScript Engine                  │    │
│  │  ┌─────────────┐    ┌─────────────────────────────┐ │    │
│  │  │  Call Stack │    │      Memory Heap            │ │    │
│  │  │             │    │   (Object Storage)          │ │    │
│  │  │  function() │    │   { obj: ... }              │ │    │
│  │  │  function() │    │   [ array ]                 │ │    │
│  │  │  global()   │    │   "strings"                 │ │    │
│  │  └─────────────┘    └─────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    Web APIs                          │    │
│  │  setTimeout, fetch, DOM, console, localStorage...   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────┐    │
│  │  Callback Queue │    │      Microtask Queue        │    │
│  │  (Macrotasks)   │    │   (Promises, queueMicrotask)│    │
│  └─────────────────┘    └─────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    Event Loop                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Components Explained:**

1. **Call Stack**: Where function execution contexts are stacked (LIFO)
2. **Memory Heap**: Unstructured memory pool for object allocation
3. **Web APIs**: Browser-provided APIs (not part of JavaScript itself)
4. **Callback Queue**: Queue for callbacks from Web APIs (setTimeout, events)
5. **Microtask Queue**: Higher priority queue for Promises and queueMicrotask
6. **Event Loop**: Orchestrates everything - moves callbacks to call stack

---

# 2. Execution Context & Call Stack

## 2.1 What is an Execution Context?

An **Execution Context** is the environment in which JavaScript code is evaluated and executed. Think of it as a "container" that holds all the information needed to run a piece of code.

### Types of Execution Contexts

```javascript
// 1. GLOBAL EXECUTION CONTEXT (GEC)
// Created when the script first runs
// Only ONE global execution context exists

var globalVar = "I'm global";    // Stored in GEC's variable environment
function globalFunc() {}         // Stored in GEC's variable environment

// 2. FUNCTION EXECUTION CONTEXT (FEC)
// Created every time a function is INVOKED (called)
function outer() {
  var outerVar = "outer";        // Stored in outer's FEC
  
  function inner() {
    var innerVar = "inner";      // Stored in inner's FEC
  }
  
  inner();  // Creates new FEC for inner
}

outer();  // Creates new FEC for outer

// 3. EVAL EXECUTION CONTEXT
// Created when code runs inside eval() - AVOID using eval()
eval("var x = 10");
```

### 2.2 Execution Context Phases

Each execution context goes through TWO phases:

```
┌─────────────────────────────────────────────────────────────┐
│                    CREATION PHASE                            │
│                                                              │
│  1. Create the Variable Object (VO) / Lexical Environment   │
│     - Function arguments → parameters                        │
│     - Function declarations → hoisted completely            │
│     - Variable declarations → hoisted as undefined          │
│                                                              │
│  2. Create the Scope Chain                                  │
│     - Current VO + all parent VOs                           │
│                                                              │
│  3. Determine the value of 'this'                           │
│     - Based on how the function was called                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    EXECUTION PHASE                           │
│                                                              │
│  1. Execute code line by line                               │
│  2. Assign values to variables                              │
│  3. Execute function calls                                  │
└─────────────────────────────────────────────────────────────┘
```

**Detailed Example:**

```javascript
var name = "Jayesh";
var age = 25;

function greet(message) {
  var greeting = message + " " + name;
  console.log(greeting);
}

greet("Hello");
```

**Creation Phase - Global Execution Context:**
```
Global Execution Context = {
  VariableObject: {
    name: undefined,        // var declaration hoisted
    age: undefined,         // var declaration hoisted
    greet: function() {...} // function declaration hoisted COMPLETELY
  },
  ScopeChain: [Global VO],
  this: window (in browser) / global (in Node.js)
}
```

**Execution Phase - Global Execution Context:**
```
Global Execution Context = {
  VariableObject: {
    name: "Jayesh",         // Assigned during execution
    age: 25,                // Assigned during execution
    greet: function() {...} // Already set during creation
  },
  ScopeChain: [Global VO],
  this: window
}
```

**When greet("Hello") is called - Creation Phase:**
```
greet Execution Context = {
  VariableObject: {
    arguments: { 0: "Hello", length: 1 },
    message: "Hello",       // Parameter
    greeting: undefined     // var declaration hoisted
  },
  ScopeChain: [greet VO, Global VO],
  this: window
}
```

### 2.3 The Call Stack

The **Call Stack** is a LIFO (Last In, First Out) data structure that tracks function execution.

```javascript
function first() {
  console.log("First start");
  second();
  console.log("First end");
}

function second() {
  console.log("Second start");
  third();
  console.log("Second end");
}

function third() {
  console.log("Third start");
  console.log("Third end");
}

first();
```

**Call Stack Visualization:**

```
Step 1: Script starts
┌─────────────┐
│   Global    │  ← Global Execution Context created
└─────────────┘

Step 2: first() is called
┌─────────────┐
│   first()   │  ← first() pushed onto stack
├─────────────┤
│   Global    │
└─────────────┘

Step 3: second() is called from first()
┌─────────────┐
│  second()   │  ← second() pushed onto stack
├─────────────┤
│   first()   │
├─────────────┤
│   Global    │
└─────────────┘

Step 4: third() is called from second()
┌─────────────┐
│   third()   │  ← third() pushed onto stack
├─────────────┤
│  second()   │
├─────────────┤
│   first()   │
├─────────────┤
│   Global    │
└─────────────┘

Step 5: third() completes and returns
┌─────────────┐
│  second()   │  ← third() popped off stack
├─────────────┤
│   first()   │
├─────────────┤
│   Global    │
└─────────────┘

Step 6: second() completes and returns
┌─────────────┐
│   first()   │  ← second() popped off stack
├─────────────┤
│   Global    │
└─────────────┘

Step 7: first() completes and returns
┌─────────────┐
│   Global    │  ← first() popped off stack
└─────────────┘

Step 8: Script ends
(Call stack is empty)
```

**Output:**
```
First start
Second start
Third start
Third end
Second end
First end
```

### 2.4 Stack Overflow

When the call stack exceeds its maximum size, a **Stack Overflow** error occurs:

```javascript
// This will cause a stack overflow
function recursive() {
  recursive();  // Calls itself infinitely
}

recursive();
// Error: Maximum call stack size exceeded

// Correct recursion with base case
function factorial(n) {
  if (n <= 1) return 1;  // Base case - stops recursion
  return n * factorial(n - 1);
}

console.log(factorial(5));  // 120
```

**Maximum Stack Size:**
- Chrome/V8: ~10,000-16,000 frames (varies)
- Firefox: ~10,000 frames
- Safari: ~50,000 frames

### 2.5 Practical Interview Questions

**Q1: What will be logged and why?**

```javascript
console.log(a);
console.log(b);
var a = 1;
let b = 2;
```

**Answer:**
```
undefined
ReferenceError: Cannot access 'b' before initialization

Explanation:
- 'var a' is hoisted and initialized to undefined during creation phase
- 'let b' is hoisted but NOT initialized (Temporal Dead Zone)
```

**Q2: What's the output?**

```javascript
function test() {
  console.log(x);  // ?
  console.log(y);  // ?
  
  var x = 10;
  
  function y() {
    return 20;
  }
}

test();
```

**Answer:**
```
undefined
[Function: y]

Explanation:
- var x is hoisted as undefined
- function y is hoisted completely with its definition
```

---

# 3. The Event Loop - Complete Deep Dive

## 3.1 Why Do We Need the Event Loop?

JavaScript is **single-threaded**, meaning it has only ONE call stack and can execute only ONE piece of code at a time. But how does it handle:

- Network requests that take seconds
- Timers (setTimeout, setInterval)
- User interactions (clicks, scrolls)
- File operations in Node.js

The answer: **The Event Loop** with help from Web APIs and callback queues.

```javascript
// Without the event loop, this would freeze the browser:
console.log("Start");

// Imagine this takes 5 seconds
const data = fetchDataFromServer();  // BLOCKING!

console.log("End");  // User waits 5 seconds to see this

// ---------------------------------

// With async patterns and event loop:
console.log("Start");

// Non-blocking! Callback runs when data is ready
fetchDataFromServer().then(data => {
  console.log(data);
});

console.log("End");  // Runs immediately!

// Output:
// Start
// End
// (data logged when ready)
```

## 3.2 Event Loop Components

### The Complete Picture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                             CALL STACK                                     │
│                         (Executes code)                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │   Currently executing function                                       │  │
│  │   Previous function                                                  │  │
│  │   ...                                                                │  │
│  │   Global Execution Context                                           │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
         ↑                              │
         │                              │ setTimeout, fetch, etc.
         │                              ↓
┌────────┴──────────────────────────────────────────────────────────────────┐
│                              WEB APIs                                      │
│  (Browser/Node.js provides these - NOT part of JavaScript engine)          │
│                                                                            │
│  • setTimeout / setInterval    • DOM Events (click, scroll)               │
│  • fetch / XMLHttpRequest      • Console API                              │
│  • Geolocation                 • Web Workers                              │
│  • IndexedDB                   • WebSockets                               │
└───────────────────────────────────────────────────────────────────────────┘
         │                              │
         │                              │
         ↓                              ↓
┌─────────────────────────┐   ┌─────────────────────────────────────────────┐
│    MICROTASK QUEUE      │   │              MACROTASK QUEUE                │
│    (Higher Priority)    │   │            (Callback Queue)                 │
│                         │   │                                             │
│  • Promise.then/catch   │   │  • setTimeout callbacks                     │
│  • queueMicrotask       │   │  • setInterval callbacks                    │
│  • MutationObserver     │   │  • setImmediate (Node.js)                   │
│  • process.nextTick*    │   │  • I/O callbacks                            │
│                         │   │  • UI rendering                             │
│                         │   │  • requestAnimationFrame                    │
└─────────────────────────┘   └─────────────────────────────────────────────┘
         │                              │
         └──────────────┬───────────────┘
                        │
                        ↓
┌───────────────────────────────────────────────────────────────────────────┐
│                            EVENT LOOP                                      │
│                                                                            │
│  while (true) {                                                           │
│    1. Execute everything in Call Stack until empty                        │
│    2. Execute ALL microtasks until microtask queue is empty               │
│    3. Render UI if needed (browser)                                       │
│    4. Execute ONE macrotask (if any)                                      │
│    5. Go to step 1                                                        │
│  }                                                                        │
└───────────────────────────────────────────────────────────────────────────┘

* process.nextTick is Node.js specific and has even higher priority than microtasks
```

## 3.3 The Event Loop Algorithm

Here's the exact order of operations:

```javascript
// EVENT LOOP ALGORITHM (Simplified)

while (eventLoop.isRunning) {
  // 1. EXECUTE CALL STACK
  while (callStack.isNotEmpty) {
    callStack.executeTop();
  }
  
  // 2. EXECUTE ALL MICROTASKS (Promise callbacks, queueMicrotask)
  while (microtaskQueue.isNotEmpty) {
    const microtask = microtaskQueue.dequeue();
    callStack.push(microtask);
    callStack.executeTop();
    
    // If microtask adds more microtasks, they run before ANY macrotask
    // This can lead to infinite loops if you keep adding microtasks!
  }
  
  // 3. RENDER (Browser only)
  if (browser.needsToRender) {
    browser.executeAnimationFrameCallbacks();
    browser.render();
  }
  
  // 4. EXECUTE ONE MACROTASK (setTimeout, I/O, etc.)
  if (macrotaskQueue.isNotEmpty) {
    const macrotask = macrotaskQueue.dequeue();
    callStack.push(macrotask);
    callStack.executeTop();
  }
  
  // 5. Repeat
}
```

## 3.4 Microtasks vs Macrotasks

### Microtasks (Higher Priority)

```javascript
// Promise callbacks
Promise.resolve().then(() => console.log("Promise 1"));
Promise.resolve().then(() => console.log("Promise 2"));

// queueMicrotask
queueMicrotask(() => console.log("queueMicrotask"));

// MutationObserver callbacks
const observer = new MutationObserver(() => console.log("MutationObserver"));

// All microtasks run BEFORE any macrotask
```

### Macrotasks (Lower Priority)

```javascript
// setTimeout / setInterval
setTimeout(() => console.log("setTimeout"), 0);
setInterval(() => console.log("setInterval"), 1000);

// requestAnimationFrame
requestAnimationFrame(() => console.log("requestAnimationFrame"));

// I/O operations
fetch('/api/data').then(response => {
  // The fetch itself is handled by Web API
  // The .then callback is a MICROTASK
  // But the initial I/O completion is a MACROTASK
});

// Event listeners (when triggered)
button.addEventListener('click', () => console.log("click"));
```

## 3.5 Classic Event Loop Examples

### Example 1: Basic Order

```javascript
console.log("Script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise 1");
}).then(() => {
  console.log("Promise 2");
});

console.log("Script end");
```

**Output:**
```
Script start
Script end
Promise 1
Promise 2
setTimeout
```

**Explanation:**
1. `console.log("Script start")` - Sync, runs immediately
2. `setTimeout` - Callback sent to Macrotask Queue (even with 0ms delay)
3. `Promise.resolve().then()` - Callback sent to Microtask Queue
4. `console.log("Script end")` - Sync, runs immediately
5. Call Stack empty → Run ALL microtasks: "Promise 1", "Promise 2"
6. Run ONE macrotask: "setTimeout"

### Example 2: Nested Callbacks

```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
  Promise.resolve().then(() => {
    console.log("3");
  });
}, 0);

Promise.resolve().then(() => {
  console.log("4");
  setTimeout(() => {
    console.log("5");
  }, 0);
});

console.log("6");
```

**Output:**
```
1
6
4
2
3
5
```

**Step-by-step:**
1. Log "1" (sync)
2. setTimeout callback → Macrotask Queue
3. Promise.then callback → Microtask Queue
4. Log "6" (sync)
5. Call Stack empty → Process Microtasks
   - Log "4"
   - setTimeout callback (logs "5") → Macrotask Queue
6. Process ONE Macrotask (the first setTimeout)
   - Log "2"
   - Promise.then callback → Microtask Queue
7. Process ALL Microtasks
   - Log "3"
8. Process ONE Macrotask
   - Log "5"

### Example 3: The Most Tricky One

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

async1();

new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => {
  console.log("promise2");
});

console.log("script end");
```

**Output:**
```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

**Detailed Explanation:**

1. `console.log("script start")` - Sync code, logs immediately

2. `setTimeout` callback goes to Macrotask Queue

3. `async1()` is called:
   - Logs "async1 start"
   - Calls `await async2()`:
     - `async2()` runs and logs "async2"
     - `await` pauses async1 and schedules continuation as MICROTASK
   - async1() returns (paused)

4. `new Promise((resolve) => {...})`:
   - The executor function runs SYNCHRONOUSLY
   - Logs "promise1"
   - `resolve()` is called
   - `.then()` callback goes to Microtask Queue

5. `console.log("script end")` - Sync code, logs immediately

6. Call Stack is empty, process ALL Microtasks:
   - async1 continuation logs "async1 end"
   - Promise.then logs "promise2"

7. Process ONE Macrotask:
   - setTimeout logs "setTimeout"

### Example 4: Microtask Starvation

```javascript
// WARNING: This can freeze your browser!
function infiniteMicrotasks() {
  Promise.resolve().then(() => {
    console.log("Microtask");
    infiniteMicrotasks();  // Adds another microtask
  });
}

setTimeout(() => console.log("Macrotask"), 0);
infiniteMicrotasks();

// The "Macrotask" will NEVER log because microtasks
// have priority and new ones keep being added!
```

This is why you should be careful with recursive Promises - they can starve the macrotask queue and freeze UI updates.

## 3.6 setTimeout(fn, 0) Doesn't Mean 0ms

```javascript
const start = Date.now();

setTimeout(() => {
  console.log(`Actual delay: ${Date.now() - start}ms`);
}, 0);

// Block for 100ms
while (Date.now() - start < 100) {}

console.log("Blocking done");

// Output:
// Blocking done
// Actual delay: 100ms (approximately)
```

**Why?**
1. setTimeout(fn, 0) means "run as soon as possible, but not now"
2. The callback goes to the Macrotask Queue
3. It can only run when the Call Stack is empty
4. If you block the stack, the callback waits

**Minimum delay:**
- HTML5 spec sets minimum delay to 4ms after 5 nested setTimeout calls
- This is to prevent browser freezing

## 3.7 requestAnimationFrame

`requestAnimationFrame` runs BEFORE the next browser repaint (~60fps = every ~16.67ms):

```javascript
function animate() {
  // Update animation state
  element.style.left = newPosition + 'px';
  
  // Schedule next frame
  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

**Event Loop Position:**
```
Microtasks → requestAnimationFrame → Render → Macrotask
```

This makes it perfect for smooth animations because it syncs with the browser's refresh rate.

## 3.8 Node.js Event Loop Differences

Node.js has additional phases in its event loop:

```
   ┌───────────────────────────┐
┌─>│           timers          │ ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │ ← I/O callbacks deferred
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │ ← Internal use only
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │ ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │ ← socket.on('close')
   └───────────────────────────┘

Between each phase: ALL microtasks run (including process.nextTick)
```

**Node.js specific:**

```javascript
// process.nextTick - HIGHEST priority microtask
process.nextTick(() => console.log("nextTick"));

// setImmediate - Runs in "check" phase
setImmediate(() => console.log("setImmediate"));

// Order depends on context:
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));
// Could be either order in main module!

// But inside I/O callback:
const fs = require('fs');
fs.readFile('file.txt', () => {
  setTimeout(() => console.log("setTimeout"), 0);
  setImmediate(() => console.log("setImmediate"));
  // setImmediate ALWAYS runs first here
});
```

---

# 4. Scope & Scope Chain

## 4.1 What is Scope?

**Scope** determines the accessibility (visibility) of variables. It defines where variables can be accessed in your code.

### Types of Scope

```javascript
// 1. GLOBAL SCOPE
var globalVar = "I'm global";
let globalLet = "I'm also global";

// Can be accessed anywhere in the code

// 2. FUNCTION SCOPE (Local Scope)
function myFunction() {
  var functionVar = "I'm function-scoped";
  let functionLet = "I'm also function-scoped";
  
  // These can only be accessed inside this function
}

console.log(functionVar);  // ReferenceError

// 3. BLOCK SCOPE (ES6+)
if (true) {
  var blockVar = "I'm NOT block-scoped (var ignores blocks)";
  let blockLet = "I'm block-scoped";
  const blockConst = "I'm also block-scoped";
}

console.log(blockVar);   // "I'm NOT block-scoped" - var leaks out!
console.log(blockLet);   // ReferenceError - let respects blocks
console.log(blockConst); // ReferenceError - const respects blocks

// 4. MODULE SCOPE (ES6 Modules)
// Variables declared in a module are scoped to that module
// They don't pollute the global scope
```

## 4.2 Lexical Scope (Static Scope)

JavaScript uses **Lexical Scoping**, which means scope is determined by where functions are WRITTEN (declared), not where they are called.

```javascript
var name = "Global";

function outer() {
  var name = "Outer";
  
  function inner() {
    // inner() looks for 'name' in its lexical environment
    // It finds it in outer's scope
    console.log(name);  // "Outer"
  }
  
  inner();
}

outer();

// Even if we return inner and call it elsewhere:
function createGreeter() {
  var greeting = "Hello";
  
  return function greet(name) {
    // greet() still has access to 'greeting'
    // because of lexical scoping
    console.log(greeting + ", " + name);
  };
}

const greet = createGreeter();
greet("Jayesh");  // "Hello, Jayesh"
// greeting is gone from createGreeter's execution context
// but greet() still has access through lexical scope!
```

### Dynamic Scope (for comparison)

JavaScript does NOT use dynamic scope, but it's useful to understand the difference:

```javascript
// If JavaScript used DYNAMIC scope (it doesn't!):
var name = "Global";

function getName() {
  console.log(name);
}

function test() {
  var name = "Test";
  getName();  // With dynamic scope: would log "Test"
              // With lexical scope (JS): logs "Global"
}

test();

// Lexical: Based on where function is WRITTEN
// Dynamic: Based on where function is CALLED
```

## 4.3 The Scope Chain

When JavaScript looks for a variable, it searches through the **Scope Chain**:

1. Current Scope (local variables)
2. Parent Scope
3. Grandparent Scope
4. ...continues up...
5. Global Scope
6. If not found anywhere → ReferenceError

```javascript
var global = "global";

function outer() {
  var outerVar = "outer";
  
  function middle() {
    var middleVar = "middle";
    
    function inner() {
      var innerVar = "inner";
      
      // Access chain: inner → middle → outer → global
      console.log(innerVar);   // "inner" (found in inner)
      console.log(middleVar);  // "middle" (found in middle)
      console.log(outerVar);   // "outer" (found in outer)
      console.log(global);     // "global" (found in global)
    }
    
    inner();
  }
  
  middle();
}

outer();
```

**Scope Chain Visualization:**

```
┌─────────────────────────────────────────────────────────────────┐
│  GLOBAL SCOPE                                                    │
│    global = "global"                                             │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  outer() SCOPE                                               ││
│  │    outerVar = "outer"                                        ││
│  │  ┌─────────────────────────────────────────────────────────┐││
│  │  │  middle() SCOPE                                          │││
│  │  │    middleVar = "middle"                                  │││
│  │  │  ┌─────────────────────────────────────────────────────┐│││
│  │  │  │  inner() SCOPE                                       ││││
│  │  │  │    innerVar = "inner"                                ││││
│  │  │  │                                                      ││││
│  │  │  │  Scope Chain: inner → middle → outer → global        ││││
│  │  │  └─────────────────────────────────────────────────────┘│││
│  │  └─────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## 4.4 Block Scope vs Function Scope

```javascript
// FUNCTION SCOPE (var)
function functionScoped() {
  if (true) {
    var x = 10;
  }
  console.log(x);  // 10 - var ignores block boundaries
}

// BLOCK SCOPE (let/const)
function blockScoped() {
  if (true) {
    let y = 20;
    const z = 30;
  }
  console.log(y);  // ReferenceError: y is not defined
  console.log(z);  // ReferenceError: z is not defined
}

// Practical implication: Loop variables
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3 - 'i' is shared across all iterations

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
// Output: 0, 1, 2 - Each iteration gets its own 'j'
```

## 4.5 Scope Interview Questions

**Q1: What will be logged?**

```javascript
var x = 10;

function outer() {
  var x = 20;
  function inner() {
    console.log(x);
  }
  return inner;
}

var innerFn = outer();
innerFn();
```

**Answer:** `20` - The inner function remembers the scope where it was defined (closure).

**Q2: What about this?**

```javascript
var x = 10;

function outer() {
  function inner() {
    console.log(x);
  }
  return inner;
}

var innerFn = outer();
var x = 20;
innerFn();
```

**Answer:** `20` - The global `x` is reassigned before `innerFn` is called. When `inner` looks for `x`, it finds `20` in the global scope.

---

# 5. Closures - Complete Mastery

## 5.1 What is a Closure?

A **closure** is a function that has access to variables from its outer (enclosing) scope, even after the outer function has returned. It's a combination of a function and the lexical environment within which that function was declared.

### The Technical Definition

> A closure is created when a function "closes over" its lexical scope, capturing variables from that scope.

```javascript
function createGreeting(greeting) {
  // 'greeting' is captured by the inner function
  return function(name) {
    // This function is a closure
    // It has access to 'greeting' even after createGreeting returns
    return `${greeting}, ${name}!`;
  };
}

const sayHello = createGreeting("Hello");
const sayHi = createGreeting("Hi");

console.log(sayHello("Jayesh"));  // "Hello, Jayesh!"
console.log(sayHi("Jayesh"));     // "Hi, Jayesh!"

// Each closure has its own 'greeting' variable captured
```

### 5.2 How Closures Work Under the Hood

When a function is created, it gets a hidden `[[Environment]]` property that references the Lexical Environment where it was created:

```javascript
function outer() {
  let count = 0;  // This is captured in the closure
  
  function inner() {
    count++;
    return count;
  }
  
  return inner;
}

const counter = outer();
```

**Memory Representation:**

```
outer() Lexical Environment:
┌─────────────────────────────────┐
│  count: 0                       │
│  inner: function() {...}        │
│  [[outer]]: Global Environment  │
└─────────────────────────────────┘
         ↑
         │ [[Environment]] reference
         │
┌────────┴────────────────────────┐
│  inner function object          │
│  - Code: { count++; return... } │
│  - [[Environment]]: ──────────→ outer's Lexical Environment
└─────────────────────────────────┘

When outer() returns:
- outer's execution context is removed from call stack
- BUT outer's Lexical Environment is NOT garbage collected
- Because inner (now assigned to 'counter') still references it
```

### 5.3 Classic Closure Examples

#### Example 1: Private Variables

```javascript
function createBankAccount(initialBalance) {
  // Private variable - not accessible from outside
  let balance = initialBalance;
  
  return {
    deposit: function(amount) {
      if (amount > 0) {
        balance += amount;
        return balance;
      }
    },
    withdraw: function(amount) {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        return balance;
      }
      return "Insufficient funds";
    },
    getBalance: function() {
      return balance;
    }
  };
}

const account = createBankAccount(100);
console.log(account.getBalance());  // 100
console.log(account.deposit(50));   // 150
console.log(account.withdraw(30));  // 120

// Cannot access 'balance' directly
console.log(account.balance);       // undefined
```

#### Example 2: Function Factory

```javascript
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);
const tenTimes = createMultiplier(10);

console.log(double(5));    // 10
console.log(triple(5));    // 15
console.log(tenTimes(5));  // 50
```

#### Example 3: Event Handlers with State

```javascript
function createButtonHandler(buttonId) {
  let clickCount = 0;
  
  return function handleClick() {
    clickCount++;
    console.log(`Button ${buttonId} clicked ${clickCount} times`);
  };
}

// Each button gets its own counter
const button1Handler = createButtonHandler(1);
const button2Handler = createButtonHandler(2);

button1Handler();  // Button 1 clicked 1 times
button1Handler();  // Button 1 clicked 2 times
button2Handler();  // Button 2 clicked 1 times
```

#### Example 4: Memoization

```javascript
function memoize(fn) {
  const cache = {};  // Closure captures this cache
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      console.log('Returning cached result');
      return cache[key];
    }
    
    console.log('Computing new result');
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const expensiveOperation = memoize((n) => {
  // Simulate expensive computation
  let result = 0;
  for (let i = 0; i < n * 1000000; i++) {
    result += i;
  }
  return result;
});

console.log(expensiveOperation(10));  // Computing new result
console.log(expensiveOperation(10));  // Returning cached result (instant!)
console.log(expensiveOperation(20));  // Computing new result
```

### 5.4 The Loop Closure Problem (CRITICAL!)

This is one of the most asked interview questions:

```javascript
// THE PROBLEM
for (var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}

// Expected: 0, 1, 2, 3, 4
// Actual: 5, 5, 5, 5, 5

// WHY?
// - 'var i' is function-scoped (or global if in global scope)
// - There's only ONE 'i' shared by all callbacks
// - By the time callbacks run (after 1 second), the loop is done
// - i === 5 at that point
```

**Solutions:**

```javascript
// SOLUTION 1: Use 'let' (creates new binding per iteration)
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
// Output: 0, 1, 2, 3, 4

// SOLUTION 2: IIFE (Immediately Invoked Function Expression)
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);
    }, 1000);
  })(i);  // Pass 'i' as argument, captured as 'j'
}
// Output: 0, 1, 2, 3, 4

// SOLUTION 3: Use bind or arrow function with closure
for (var i = 0; i < 5; i++) {
  setTimeout(console.log.bind(null, i), 1000);
}
// Output: 0, 1, 2, 3, 4

// SOLUTION 4: Use forEach (creates new scope each iteration)
[0, 1, 2, 3, 4].forEach(function(i) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
});
// Output: 0, 1, 2, 3, 4
```

### 5.5 Stale Closures in React

Closures can cause bugs in React when used incorrectly:

```javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      // BUG: This closure captures count = 0
      console.log('Count:', count);  // Always logs 0!
      setCount(count + 1);           // Always sets to 1!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Empty dependency array means effect runs once
  
  return <p>Count: {count}</p>;  // Shows 1, not incrementing
}

// SOLUTION 1: Use functional update
setCount(prevCount => prevCount + 1);

// SOLUTION 2: Include count in dependencies (but creates new interval each time)
useEffect(() => {
  // ...
}, [count]);

// SOLUTION 3: Use useRef for values that shouldn't trigger re-render
const countRef = useRef(count);
countRef.current = count;  // Always up to date

useEffect(() => {
  const interval = setInterval(() => {
    console.log('Count:', countRef.current);  // Always current value
  }, 1000);
  
  return () => clearInterval(interval);
}, []);
```

### 5.6 Closure Memory Considerations

Closures can lead to memory leaks if not handled properly:

```javascript
// POTENTIAL MEMORY LEAK
function createHeavyObject() {
  // This large array is 10MB
  const largeData = new Array(10 * 1024 * 1024).fill('x');
  
  return function() {
    // Even if we only need a small part of the data
    console.log(largeData.length);
    // The entire 'largeData' array is kept in memory
    // because the closure references it
  };
}

const leakyFunction = createHeavyObject();
// largeData is now stuck in memory as long as leakyFunction exists

// SOLUTION: Minimize what the closure captures
function createEfficientFunction() {
  const largeData = new Array(10 * 1024 * 1024).fill('x');
  const dataLength = largeData.length;  // Extract only what we need
  
  // largeData can now be garbage collected
  return function() {
    console.log(dataLength);  // Only captures the number
  };
}
```

### 5.7 Closure Interview Questions

**Q1: What is a closure?**

**Answer:** A closure is a function bundled together with its lexical environment. It gives the function access to variables from its outer scope, even after the outer function has returned. Every function in JavaScript creates a closure.

**Q2: Implement a function that can only be called once:**

```javascript
function once(fn) {
  let called = false;
  let result;
  
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

const initializeOnce = once(() => {
  console.log('Initializing...');
  return 'Initialized!';
});

console.log(initializeOnce());  // Logs "Initializing...", returns "Initialized!"
console.log(initializeOnce());  // Returns "Initialized!" (no console log)
console.log(initializeOnce());  // Returns "Initialized!" (no console log)
```

**Q3: Create a function that logs how many times it has been called:**

```javascript
function createCounter() {
  let count = 0;
  
  return function() {
    count++;
    console.log(`Called ${count} times`);
    return count;
  };
}

const counter = createCounter();
counter();  // Called 1 times
counter();  // Called 2 times
counter();  // Called 3 times
```

**Q4: Implement debounce:**

```javascript
function debounce(fn, delay) {
  let timeoutId;  // Closure captures this
  
  return function(...args) {
    // Clear previous timeout
    clearTimeout(timeoutId);
    
    // Set new timeout
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// Usage: Only fires after user stops typing for 300ms
const handleSearch = debounce((query) => {
  console.log('Searching for:', query);
}, 300);

handleSearch('a');    // Cancelled
handleSearch('ab');   // Cancelled  
handleSearch('abc');  // Executes after 300ms
```

**Q5: Implement throttle:**

```javascript
function throttle(fn, interval) {
  let lastTime = 0;  // Closure captures this
  
  return function(...args) {
    const now = Date.now();
    
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}

// Usage: Fires at most once per 1000ms
const handleScroll = throttle(() => {
  console.log('Scroll event handled');
}, 1000);

window.addEventListener('scroll', handleScroll);
```

---

# 6. Hoisting & Temporal Dead Zone

## 6.1 What is Hoisting?

**Hoisting** is JavaScript's default behavior of moving declarations to the top of their scope during the compilation phase. However, only the DECLARATIONS are hoisted, not the initializations.

### var Hoisting

```javascript
// What you write:
console.log(x);  // undefined (not ReferenceError!)
var x = 5;
console.log(x);  // 5

// How JavaScript interprets it (conceptually):
var x;           // Declaration hoisted to top
console.log(x);  // undefined
x = 5;           // Assignment stays in place
console.log(x);  // 5
```

### Function Declaration Hoisting

Function declarations are hoisted COMPLETELY - both the name and the function body:

```javascript
// What you write:
sayHello();  // "Hello!" - Works!

function sayHello() {
  console.log("Hello!");
}

// How JavaScript interprets it:
function sayHello() {  // Entire function hoisted
  console.log("Hello!");
}

sayHello();  // "Hello!"
```

### Function Expression Hoisting

Function expressions are NOT hoisted the same way:

```javascript
// With var
console.log(sayHi);  // undefined
sayHi();             // TypeError: sayHi is not a function

var sayHi = function() {
  console.log("Hi!");
};

// Because it's interpreted as:
var sayHi;           // Only the variable declaration is hoisted
console.log(sayHi);  // undefined
sayHi();             // Error! sayHi is undefined, not a function
sayHi = function() { console.log("Hi!"); };
```

### let and const Hoisting (Temporal Dead Zone)

`let` and `const` ARE hoisted, but they are NOT initialized. They exist in a "Temporal Dead Zone" (TDZ) from the start of the block until the declaration is encountered.

```javascript
// This proves let/const are hoisted:
let x = 'outer';

function test() {
  // If 'x' wasn't hoisted, this would log 'outer'
  console.log(x);  // ReferenceError: Cannot access 'x' before initialization
  let x = 'inner';
}

test();

// The inner 'x' shadows the outer 'x' because it's hoisted
// But it's in the TDZ until the declaration line
```

## 6.2 Temporal Dead Zone (TDZ) Deep Dive

The TDZ is the period between entering a scope and the variable declaration being processed:

```javascript
{
  // TDZ for 'name' begins here
  
  console.log(typeof name);  // ReferenceError! (not "undefined")
  
  // TDZ for 'name' ends here
  let name = "Jayesh";
  
  console.log(name);  // "Jayesh"
}
```

### TDZ Visualization

```
┌─────────────────────────────────────────────────────────┐
│  {                                                       │
│    // ═══════════════════════════════════════════════   │
│    // ║         TEMPORAL DEAD ZONE (TDZ)            ║   │
│    // ║                                             ║   │
│    // ║   name exists but is uninitialized          ║   │
│    // ║   Any access throws ReferenceError          ║   │
│    // ║                                             ║   │
│    // ═══════════════════════════════════════════════   │
│                                                          │
│    let name = "Jayesh";  // TDZ ends, name is initialized│
│                                                          │
│    // name is now accessible                             │
│    console.log(name);  // "Jayesh"                       │
│  }                                                       │
└─────────────────────────────────────────────────────────┘
```

### TDZ with Function Parameters

```javascript
// Default parameters create their own scope and TDZ
function test(a = b, b = 1) {
  // This throws an error!
  // When evaluating 'a = b', 'b' is still in TDZ
}

test();  // ReferenceError: Cannot access 'b' before initialization

// This works:
function test2(a = 1, b = a) {
  console.log(a, b);  // 1, 1
}

test2();
```

### TDZ with typeof

```javascript
// Normally, typeof on undefined variable returns "undefined"
console.log(typeof undeclaredVar);  // "undefined" (no error)

// But in TDZ, typeof still throws!
{
  console.log(typeof myVar);  // ReferenceError!
  let myVar = 10;
}
```

## 6.3 Hoisting Order and Priority

When there are multiple declarations with the same name:

```javascript
// 1. Function declarations are hoisted first
// 2. Then variable declarations (but NOT assignments)
// 3. Function declarations override variable declarations
// 4. Later function declarations override earlier ones

console.log(foo);  // [Function: foo]

var foo = "variable";

function foo() {
  return "function 1";
}

function foo() {
  return "function 2";
}

console.log(foo);  // "variable"

// Interpreted as:
function foo() { return "function 1"; }  // Hoisted first
function foo() { return "function 2"; }  // Overrides previous
var foo;  // Hoisted but ignored (foo already exists)

console.log(foo);  // [Function: foo] (the second function)
foo = "variable";  // Now foo is reassigned
console.log(foo);  // "variable"
```

## 6.4 Best Practices to Avoid Hoisting Confusion

```javascript
// 1. Always declare variables at the top of their scope
function goodPractice() {
  let x, y, z;  // Declare all variables first
  
  // Then use them
  x = 1;
  y = 2;
  z = x + y;
}

// 2. Use const by default, let when needed, avoid var
const PI = 3.14159;        // Won't be reassigned
let count = 0;             // Will be reassigned
// var oldWay = "avoid";   // Don't use var

// 3. Declare functions before calling them
function greet(name) {     // Define first
  return `Hello, ${name}`;
}

const message = greet("Jayesh");  // Then call

// 4. Use function expressions for clarity
const calculate = function(a, b) {
  return a + b;
};
// Or arrow functions
const multiply = (a, b) => a * b;
```

## 6.5 Hoisting Interview Questions

**Q1: What's the output?**

```javascript
var a = 1;
function b() {
  a = 10;
  return;
  function a() {}
}
b();
console.log(a);
```

**Answer:** `1`

**Explanation:**
- Inside `b()`, `function a() {}` is hoisted to the top
- This creates a LOCAL `a` variable (the function)
- `a = 10` modifies the LOCAL `a`, not the global one
- The global `a` remains `1`

**Q2: What's the output?**

```javascript
console.log(foo);
console.log(bar);
console.log(baz);

var foo = function bar() {
  return "hello";
};

function baz() {
  return "world";
}
```

**Answer:**
```
undefined
ReferenceError: bar is not defined
[Function: baz]
```

**Explanation:**
- `foo` is hoisted as `undefined` (var)
- `bar` is a named function expression - the name is only available inside the function, not hoisted
- `baz` is a function declaration - fully hoisted

---

# 7. The 'this' Keyword - All Binding Rules

## 7.1 What is 'this'?

`this` is a special keyword that refers to the object that is executing the current function. Unlike other languages, `this` in JavaScript is NOT determined by how a function is defined, but by HOW it is CALLED.

## 7.2 The Four Binding Rules (in order of precedence)

### Rule 1: new Binding (Highest Priority)

When a function is called with the `new` keyword, `this` refers to the newly created object:

```javascript
function Person(name) {
  // 'this' is a new empty object created by 'new'
  this.name = name;
  this.sayHello = function() {
    console.log(`Hello, I'm ${this.name}`);
  };
  // Implicitly returns 'this'
}

const jayesh = new Person("Jayesh");
jayesh.sayHello();  // "Hello, I'm Jayesh"

// What 'new' does behind the scenes:
// 1. Creates a new empty object: {}
// 2. Sets the prototype of the object
// 3. Binds 'this' to the new object
// 4. Executes the constructor function
// 5. Returns the object (unless function returns another object)
```

### Rule 2: Explicit Binding (call, apply, bind)

You can explicitly set `this` using `call()`, `apply()`, or `bind()`:

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: "Jayesh" };

// call - passes arguments individually
greet.call(person, "Hello", "!");    // "Hello, I'm Jayesh!"

// apply - passes arguments as an array
greet.apply(person, ["Hi", "!!"]);   // "Hi, I'm Jayesh!!"

// bind - returns a new function with 'this' bound
const boundGreet = greet.bind(person, "Hey");
boundGreet("?");  // "Hey, I'm Jayesh?"
```

**Differences between call, apply, and bind:**

```javascript
// call: Immediately invokes, args passed individually
fn.call(thisArg, arg1, arg2, arg3);

// apply: Immediately invokes, args passed as array
fn.apply(thisArg, [arg1, arg2, arg3]);

// bind: Returns new function, doesn't invoke immediately
const boundFn = fn.bind(thisArg, arg1);
boundFn(arg2, arg3);  // Can pass more args later

// Memory trick: 
// - Call with a Comma (individual args)
// - Apply with an Array
// - Bind returns a Bound function
```

### Rule 3: Implicit Binding

When a function is called as a method of an object, `this` refers to that object:

```javascript
const user = {
  name: "Jayesh",
  greet: function() {
    console.log(`Hello, ${this.name}`);
  },
  nested: {
    name: "Nested",
    greet: function() {
      // 'this' refers to 'nested', not 'user'
      console.log(`Hello, ${this.name}`);
    }
  }
};

user.greet();         // "Hello, Jayesh"
user.nested.greet();  // "Hello, Nested"

// The object IMMEDIATELY before the dot is 'this'
// user.greet() → this = user
// user.nested.greet() → this = user.nested
```

**Implicit Binding Loss (Common Bug!):**

```javascript
const user = {
  name: "Jayesh",
  greet: function() {
    console.log(`Hello, ${this.name}`);
  }
};

user.greet();  // "Hello, Jayesh" - works!

// But if we extract the method:
const greetFn = user.greet;
greetFn();  // "Hello, undefined" - 'this' is lost!

// Why? Because greetFn() is a standalone function call
// No object before the dot → default binding applies
```

### Rule 4: Default Binding (Lowest Priority)

When none of the above rules apply, `this` defaults to:
- **Non-strict mode:** The global object (`window` in browser, `global` in Node.js)
- **Strict mode:** `undefined`

```javascript
// Non-strict mode
function showThis() {
  console.log(this);
}
showThis();  // Window object (in browser)

// Strict mode
"use strict";
function showThisStrict() {
  console.log(this);
}
showThisStrict();  // undefined
```

## 7.3 Arrow Functions and 'this'

Arrow functions do NOT have their own `this`. They inherit `this` from the enclosing scope (lexical `this`):

```javascript
const user = {
  name: "Jayesh",
  
  // Regular function: 'this' is the object
  regularGreet: function() {
    console.log(`Regular: Hello, ${this.name}`);
  },
  
  // Arrow function: 'this' is inherited from outer scope (global)
  arrowGreet: () => {
    console.log(`Arrow: Hello, ${this.name}`);
  },
  
  // Mixing regular and arrow
  delayedGreet: function() {
    // 'this' here is 'user'
    
    setTimeout(function() {
      // Regular function: 'this' is undefined (or window)
      console.log(`Timeout regular: ${this.name}`);
    }, 100);
    
    setTimeout(() => {
      // Arrow function: inherits 'this' from delayedGreet
      console.log(`Timeout arrow: ${this.name}`);
    }, 100);
  }
};

user.regularGreet();    // "Regular: Hello, Jayesh"
user.arrowGreet();      // "Arrow: Hello, undefined" (global this)
user.delayedGreet();    
// "Timeout regular: undefined"
// "Timeout arrow: Jayesh"
```

**Why Arrow Functions for Callbacks:**

```javascript
// PROBLEM: 'this' is lost in callback
class Button {
  constructor(label) {
    this.label = label;
    this.clicks = 0;
  }
  
  // This won't work as expected
  handleClickBad() {
    document.getElementById('btn').addEventListener('click', function() {
      this.clicks++;  // 'this' is the button element, not the Button instance!
      console.log(`${this.label} clicked ${this.clicks} times`);
    });
  }
  
  // SOLUTION 1: Arrow function
  handleClickArrow() {
    document.getElementById('btn').addEventListener('click', () => {
      this.clicks++;  // 'this' is the Button instance ✓
      console.log(`${this.label} clicked ${this.clicks} times`);
    });
  }
  
  // SOLUTION 2: bind
  handleClickBind() {
    document.getElementById('btn').addEventListener('click', function() {
      this.clicks++;
      console.log(`${this.label} clicked ${this.clicks} times`);
    }.bind(this));
  }
  
  // SOLUTION 3: Store reference
  handleClickStore() {
    const self = this;
    document.getElementById('btn').addEventListener('click', function() {
      self.clicks++;
      console.log(`${self.label} clicked ${self.clicks} times`);
    });
  }
}
```

## 7.4 'this' in Classes

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  
  // Method: 'this' refers to the instance
  greet() {
    console.log(`Hello, ${this.name}`);
  }
  
  // Arrow method: 'this' is always the instance (bound in constructor)
  greetArrow = () => {
    console.log(`Hello, ${this.name}`);
  };
  
  // Static method: 'this' refers to the class itself
  static describe() {
    console.log(`This is the ${this.name} class`);
  }
}

const jayesh = new Person("Jayesh");
jayesh.greet();          // "Hello, Jayesh"
jayesh.greetArrow();     // "Hello, Jayesh"
Person.describe();       // "This is the Person class"

// The advantage of arrow methods:
const greetFn = jayesh.greet;
const greetArrowFn = jayesh.greetArrow;

greetFn();        // "Hello, undefined" - 'this' is lost!
greetArrowFn();   // "Hello, Jayesh" - 'this' is bound! ✓
```

## 7.5 'this' Binding Priority Summary

```javascript
// Priority (High to Low):
// 1. new binding
// 2. Explicit binding (call/apply/bind)
// 3. Implicit binding (method call)
// 4. Default binding (standalone call)

function greet() {
  console.log(`Hello, ${this.name}`);
}

const obj1 = { name: "Object 1", greet };
const obj2 = { name: "Object 2" };

// Default binding
greet();  // "Hello, undefined" (or window.name in non-strict)

// Implicit binding
obj1.greet();  // "Hello, Object 1"

// Explicit binding wins over implicit
obj1.greet.call(obj2);  // "Hello, Object 2"

// new binding wins over explicit
function Person(name) {
  this.name = name;
}
const boundPerson = Person.bind({ name: "Bound" });
const instance = new boundPerson("Instance");
console.log(instance.name);  // "Instance" - new wins!
```

## 7.6 'this' Interview Questions

**Q1: What's the output?**

```javascript
const obj = {
  name: "Jayesh",
  getName: function() {
    return this.name;
  }
};

const getName = obj.getName;
console.log(getName());
```

**Answer:** `undefined` (or empty string if `window.name` exists)

**Explanation:** `getName` is called without an object context, so `this` defaults to global/undefined.

**Q2: What's the output?**

```javascript
const person = {
  name: "Alice",
  greet: function() {
    return function() {
      return this.name;
    };
  }
};

console.log(person.greet()());
```

**Answer:** `undefined`

**Explanation:** The inner function is called without context. The outer `.greet()` returns a function, then we call that function standalone.

**Q3: Fix the above code using arrow functions:**

```javascript
const person = {
  name: "Alice",
  greet: function() {
    return () => {
      return this.name;
    };
  }
};

console.log(person.greet()());  // "Alice"
```

**Q4: What's the output?**

```javascript
var name = "Global";

const obj = {
  name: "Object",
  methods: {
    name: "Methods",
    arrow: () => console.log(this.name),
    regular: function() {
      console.log(this.name);
    }
  }
};

obj.methods.arrow();    // ?
obj.methods.regular();  // ?
```

**Answer:**
```
"Global" (or undefined in strict mode)
"Methods"
```

**Explanation:**
- Arrow function: `this` is from enclosing scope (global, since object literals don't create scope)
- Regular function: `this` is `methods` (implicit binding)

---

# 8. Prototypes & Inheritance

## 8.1 What is a Prototype?

In JavaScript, every object has an internal link to another object called its **prototype**. When you try to access a property on an object, JavaScript first looks at the object itself. If the property isn't found, it looks at the prototype, then the prototype's prototype, and so on up the chain.

```javascript
const person = {
  name: "Jayesh",
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

// 'person' has its own 'name' and 'greet' properties
// But it also has access to toString(), hasOwnProperty(), etc.
// These come from Object.prototype

console.log(person.toString());  // "[object Object]"
console.log(person.hasOwnProperty('name'));  // true
```

## 8.2 The Prototype Chain

```javascript
const animal = {
  eats: true,
  walk() {
    console.log("Walking...");
  }
};

const dog = Object.create(animal);  // dog's prototype is animal
dog.barks = true;

const myDog = Object.create(dog);  // myDog's prototype is dog
myDog.name = "Buddy";

// Prototype chain: myDog → dog → animal → Object.prototype → null

console.log(myDog.name);   // "Buddy" (own property)
console.log(myDog.barks);  // true (from dog)
console.log(myDog.eats);   // true (from animal)
console.log(myDog.walk);   // function (from animal)
console.log(myDog.toString);  // function (from Object.prototype)
```

**Visualization:**

```
┌─────────────────────────────────────────────────────────────────┐
│  null  ← End of chain                                           │
│    ↑                                                             │
│  Object.prototype                                                │
│  { toString, hasOwnProperty, valueOf, ... }                     │
│    ↑                                                             │
│  animal                                                          │
│  { eats: true, walk: function }                                 │
│    ↑                                                             │
│  dog                                                             │
│  { barks: true }                                                 │
│    ↑                                                             │
│  myDog                                                           │
│  { name: "Buddy" }                                               │
└─────────────────────────────────────────────────────────────────┘

When we access myDog.eats:
1. Look in myDog → not found
2. Look in dog (myDog's prototype) → not found
3. Look in animal (dog's prototype) → FOUND! → return true
```

## 8.3 __proto__ vs prototype

These are often confused:

```javascript
// __proto__ (or [[Prototype]]): The actual prototype of an instance
// prototype: A property on constructor functions

function Dog(name) {
  this.name = name;
}

Dog.prototype.bark = function() {
  console.log(`${this.name} says woof!`);
};

const buddy = new Dog("Buddy");

// buddy.__proto__ === Dog.prototype
console.log(buddy.__proto__ === Dog.prototype);  // true

// Dog.prototype.__proto__ === Object.prototype
console.log(Dog.prototype.__proto__ === Object.prototype);  // true

// Object.prototype.__proto__ === null
console.log(Object.prototype.__proto__);  // null
```

**Visual Representation:**

```
┌─────────────────────────────────────────────────────────────────┐
│  Dog (function)                                                  │
│  ├── prototype ──────────────→  Dog.prototype                   │
│  │                              ├── bark: function             │
│  │                              └── __proto__ ──→ Object.prototype
│  │                                                               │
│  └── When called with 'new':                                    │
│      Creates object with __proto__ pointing to Dog.prototype    │
│                                                                  │
│  buddy (instance)                                                │
│  ├── name: "Buddy"                                              │
│  └── __proto__ ─────────────────→  Dog.prototype               │
└─────────────────────────────────────────────────────────────────┘
```

## 8.4 Constructor Functions

Before ES6 classes, constructor functions were the primary way to create objects with shared methods:

```javascript
function Person(name, age) {
  // Instance properties
  this.name = name;
  this.age = age;
}

// Shared methods (on prototype)
Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

Person.prototype.getAge = function() {
  return this.age;
};

// Static method (on constructor itself)
Person.isAdult = function(person) {
  return person.age >= 18;
};

const jayesh = new Person("Jayesh", 25);
jayesh.greet();  // "Hello, I'm Jayesh"

console.log(Person.isAdult(jayesh));  // true
```

## 8.5 ES6 Classes (Syntactic Sugar)

ES6 classes are essentially syntactic sugar over prototypal inheritance:

```javascript
class Person {
  // Constructor
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Instance method (goes to prototype)
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
  
  // Getter
  get birthYear() {
    return new Date().getFullYear() - this.age;
  }
  
  // Setter
  set birthYear(year) {
    this.age = new Date().getFullYear() - year;
  }
  
  // Static method (on class itself)
  static isAdult(person) {
    return person.age >= 18;
  }
  
  // Static property (ES2022)
  static species = "Homo sapiens";
  
  // Private field (ES2022)
  #ssn;
  
  setSsn(ssn) {
    this.#ssn = ssn;
  }
  
  getSsn() {
    return this.#ssn;
  }
}

// Under the hood, this creates:
// - Person function with constructor logic
// - Person.prototype.greet = function() {...}
// - Person.isAdult = function() {...}

const jayesh = new Person("Jayesh", 25);
jayesh.greet();  // "Hello, I'm Jayesh"
console.log(jayesh.birthYear);  // 1999 (approximately)
console.log(Person.isAdult(jayesh));  // true
console.log(Person.species);  // "Homo sapiens"
```

## 8.6 Inheritance with Classes

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
  
  eat() {
    console.log(`${this.name} is eating`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);  // MUST call super() before using 'this'
    this.breed = breed;
  }
  
  // Override parent method
  speak() {
    console.log(`${this.name} barks!`);
  }
  
  // Call parent method with super
  speakNicely() {
    super.speak();  // Calls Animal.prototype.speak
    console.log("(but quietly)");
  }
  
  // Dog-specific method
  fetch() {
    console.log(`${this.name} fetches the ball!`);
  }
}

const buddy = new Dog("Buddy", "Golden Retriever");
buddy.speak();       // "Buddy barks!"
buddy.speakNicely(); // "Buddy makes a sound" + "(but quietly)"
buddy.eat();         // "Buddy is eating" (inherited)
buddy.fetch();       // "Buddy fetches the ball!"

console.log(buddy instanceof Dog);     // true
console.log(buddy instanceof Animal);  // true
console.log(buddy instanceof Object);  // true
```

## 8.7 Prototype Methods

```javascript
const obj = { a: 1 };

// Object.getPrototypeOf - Get the prototype
console.log(Object.getPrototypeOf(obj) === Object.prototype);  // true

// Object.setPrototypeOf - Set the prototype (slow, avoid in performance-critical code)
const proto = { greet() { console.log("Hello"); } };
Object.setPrototypeOf(obj, proto);
obj.greet();  // "Hello"

// Object.create - Create object with specified prototype
const child = Object.create(proto);
child.greet();  // "Hello"

// isPrototypeOf - Check if object is in prototype chain
console.log(proto.isPrototypeOf(child));  // true

// hasOwnProperty - Check if property is directly on object (not inherited)
console.log(child.hasOwnProperty('greet'));  // false (it's on prototype)
child.name = "Child";
console.log(child.hasOwnProperty('name'));   // true
```

## 8.8 Prototype Interview Questions

**Q1: What's the output?**

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + ' speaks');
};

const cat = new Animal('Whiskers');
const speak = cat.speak;

cat.speak();
speak();
```

**Answer:**
```
"Whiskers speaks"
"undefined speaks" (or error in strict mode)
```

**Explanation:** When method is extracted and called standalone, `this` binding is lost.

**Q2: Implement inheritance without using class keyword:**

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + ' makes a sound');
};

function Dog(name, breed) {
  Animal.call(this, name);  // Call parent constructor
  this.breed = breed;
}

// Set up prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;  // Fix constructor reference

Dog.prototype.speak = function() {
  console.log(this.name + ' barks');
};

const buddy = new Dog('Buddy', 'Lab');
buddy.speak();  // "Buddy barks"
console.log(buddy instanceof Dog);     // true
console.log(buddy instanceof Animal);  // true
```

**Q3: What is `Object.create(null)` useful for?**

```javascript
// Creates object with NO prototype
const pureObject = Object.create(null);

console.log(pureObject.toString);  // undefined!
console.log(pureObject.hasOwnProperty);  // undefined!

// Useful for creating dictionaries/maps without inherited properties
pureObject.key = "value";

// No risk of collision with inherited methods
pureObject.hasOwnProperty = "I can use this as a key!";
```

---

# 9. ES6+ Features Complete Guide

## 9.1 let, const, and Block Scope

```javascript
// var - function scoped, hoisted
var x = 1;
var x = 2;  // OK - redeclaration allowed

// let - block scoped, no redeclaration
let y = 1;
// let y = 2;  // SyntaxError: already declared

// const - block scoped, cannot be reassigned
const PI = 3.14159;
// PI = 3;  // TypeError: Assignment to constant

// const with objects - can mutate properties
const obj = { a: 1 };
obj.a = 2;  // OK
obj.b = 3;  // OK
// obj = {};  // TypeError
```

## 9.2 Arrow Functions

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// Single parameter - no parentheses needed
const double = n => n * 2;

// No parameters - need parentheses
const greet = () => "Hello!";

// Multiple statements - need braces and return
const calculate = (a, b) => {
  const sum = a + b;
  return sum * 2;
};

// Returning object literal - wrap in parentheses
const createUser = (name, age) => ({ name, age });

// Key differences from regular functions:
// 1. No 'this' binding - inherits from enclosing scope
// 2. No 'arguments' object
// 3. Cannot be used as constructors (no 'new')
// 4. No 'prototype' property
```

## 9.3 Template Literals

```javascript
const name = "Jayesh";
const age = 25;

// String interpolation
const message = `Hello, ${name}! You are ${age} years old.`;

// Multi-line strings
const html = `
  <div class="card">
    <h2>${name}</h2>
    <p>Age: ${age}</p>
  </div>
`;

// Expressions in templates
const price = 100;
const tax = 0.1;
console.log(`Total: $${(price * (1 + tax)).toFixed(2)}`);

// Tagged templates
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const highlighted = highlight`Hello ${name}, you are ${age}!`;
// "Hello <mark>Jayesh</mark>, you are <mark>25</mark>!"
```

## 9.4 Destructuring

```javascript
// Array destructuring
const [a, b, c] = [1, 2, 3];
const [first, , third] = [1, 2, 3];  // Skip second
const [x, ...rest] = [1, 2, 3, 4];   // rest = [2, 3, 4]
const [p = 10, q = 20] = [5];        // Default values

// Object destructuring
const user = { name: "Jayesh", age: 25, city: "Mumbai" };
const { name, age } = user;
const { name: userName } = user;     // Rename
const { country = "India" } = user;  // Default value
const { name, ...others } = user;    // rest = { age, city }

// Nested destructuring
const data = { user: { profile: { name: "Jayesh" } } };
const { user: { profile: { name: deepName } } } = data;

// Function parameters
function greet({ name, age = 18 }) {
  console.log(`${name} is ${age}`);
}
greet({ name: "Jayesh", age: 25 });

// Swapping variables
let m = 1, n = 2;
[m, n] = [n, m];  // m = 2, n = 1
```

## 9.5 Spread and Rest Operators

```javascript
// Spread in arrays
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];        // [1, 2, 3, 4, 5]
const copy = [...arr1];              // Shallow copy
const merged = [...arr1, ...arr2];   // Merge arrays

// Spread in objects
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };      // { a: 1, b: 2, c: 3 }
const objCopy = { ...obj1 };         // Shallow copy
const override = { ...obj1, b: 10 }; // { a: 1, b: 10 }

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4, 5);  // 15

// Rest with other params (must be last)
function log(first, second, ...rest) {
  console.log(first, second, rest);
}
```

## 9.6 Enhanced Object Literals

```javascript
const name = "Jayesh";
const age = 25;

const user = {
  // Shorthand property
  name,  // Same as name: name
  age,   // Same as age: age
  
  // Shorthand method
  greet() {
    console.log(`Hello, ${this.name}`);
  },
  
  // Computed property names
  [`user_${Date.now()}`]: true,
  
  // Getters and setters
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(' ');
  }
};
```

## 9.7 Classes

```javascript
class Person {
  // Private fields (ES2022)
  #ssn;
  
  // Static properties
  static count = 0;
  
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this.#ssn = Math.random().toString(36);
    Person.count++;
  }
  
  // Instance methods
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
  
  // Getters
  get birthYear() {
    return new Date().getFullYear() - this.age;
  }
  
  // Static methods
  static getCount() {
    return Person.count;
  }
  
  // Private methods (ES2022)
  #generateId() {
    return Math.random().toString(36).substr(2, 9);
  }
}

// Inheritance
class Employee extends Person {
  constructor(name, age, role) {
    super(name, age);
    this.role = role;
  }
  
  greet() {
    super.greet();
    console.log(`I work as a ${this.role}`);
  }
}
```

## 9.8 Modules (import/export)

```javascript
// Named exports (math.js)
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export class Calculator { }

// Default export (logger.js)
export default function log(msg) { console.log(msg); }

// Named imports
import { PI, add } from './math.js';
import { add as addNumbers } from './math.js';
import * as math from './math.js';

// Default import
import log from './logger.js';
import myLogger from './logger.js';  // Any name works

// Mixed
import log, { PI, add } from './module.js';

// Dynamic import
const module = await import('./math.js');
```

## 9.9 Symbols

```javascript
// Create unique symbols
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2);  // false - always unique

// Use as object keys
const id = Symbol('id');
const user = {
  name: "Jayesh",
  [id]: 12345
};

console.log(user[id]);  // 12345
console.log(Object.keys(user));  // ['name'] - symbols hidden

// Well-known symbols
const arr = [1, 2, 3];
arr[Symbol.iterator];  // Built-in iterator

// Custom iterator
const range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    return {
      current: this.from,
      last: this.to,
      next() {
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        }
        return { done: true };
      }
    };
  }
};

for (const num of range) console.log(num);  // 1, 2, 3, 4, 5
```

## 9.10 Iterators and Generators

```javascript
// Generator function
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// Infinite generator
function* infiniteNumbers() {
  let n = 0;
  while (true) {
    yield n++;
  }
}

// Generator with return value
function* withReturn() {
  yield 1;
  yield 2;
  return 3;  // Not yielded in for...of
}

// Delegating generators
function* combined() {
  yield* [1, 2];
  yield* 'ab';
  yield* numberGenerator();
}

// Two-way communication
function* conversation() {
  const name = yield "What's your name?";
  const age = yield `Hello ${name}! How old are you?`;
  return `${name} is ${age} years old`;
}

const talk = conversation();
console.log(talk.next().value);         // "What's your name?"
console.log(talk.next("Jayesh").value); // "Hello Jayesh! How old are you?"
console.log(talk.next(25).value);       // "Jayesh is 25 years old"
```

## 9.11 Map and Set

```javascript
// Map - key-value pairs with any key type
const map = new Map();
map.set('name', 'Jayesh');
map.set(1, 'one');
map.set({}, 'object key');

console.log(map.get('name'));  // 'Jayesh'
console.log(map.size);         // 3
console.log(map.has('name'));  // true

map.delete('name');
map.clear();

// Initialize with entries
const map2 = new Map([
  ['a', 1],
  ['b', 2]
]);

// Iterate
for (const [key, value] of map2) {
  console.log(key, value);
}

// Set - unique values
const set = new Set([1, 2, 3, 3, 3]);
console.log(set.size);  // 3

set.add(4);
set.has(1);   // true
set.delete(1);

// Remove duplicates from array
const arr = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(arr)];  // [1, 2, 3]

// WeakMap and WeakSet - keys are weakly held (garbage collected)
const weakMap = new WeakMap();
let obj = { data: 'value' };
weakMap.set(obj, 'metadata');
obj = null;  // Object can be garbage collected
```

## 9.12 Proxy and Reflect

```javascript
// Proxy - intercept operations on objects
const target = { name: 'Jayesh', age: 25 };

const handler = {
  get(target, prop) {
    console.log(`Getting ${prop}`);
    return prop in target ? target[prop] : 'Not found';
  },
  set(target, prop, value) {
    console.log(`Setting ${prop} to ${value}`);
    if (prop === 'age' && typeof value !== 'number') {
      throw new TypeError('Age must be a number');
    }
    target[prop] = value;
    return true;
  }
};

const proxy = new Proxy(target, handler);
console.log(proxy.name);  // "Getting name" → "Jayesh"
proxy.age = 26;           // "Setting age to 26"

// Reflect - provides methods for interceptable operations
Reflect.get(target, 'name');
Reflect.set(target, 'age', 30);
Reflect.has(target, 'name');
Reflect.deleteProperty(target, 'age');
```

---

# 10. Asynchronous JavaScript

## 10.1 Callbacks

```javascript
// Callback pattern
function fetchData(callback) {
  setTimeout(() => {
    callback(null, { data: 'result' });
  }, 1000);
}

fetchData((error, data) => {
  if (error) {
    console.error(error);
    return;
  }
  console.log(data);
});

// Callback Hell (Pyramid of Doom)
getUserData(userId, (err, user) => {
  getOrders(user.id, (err, orders) => {
    getOrderDetails(orders[0].id, (err, details) => {
      getShippingInfo(details.id, (err, shipping) => {
        // Deeply nested - hard to read and maintain
      });
    });
  });
});
```

## 10.2 Promises

```javascript
// Creating a Promise
const promise = new Promise((resolve, reject) => {
  const success = true;
  
  if (success) {
    resolve('Operation successful');
  } else {
    reject(new Error('Operation failed'));
  }
});

// Consuming a Promise
promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Cleanup'));

// Chaining
fetchUser(userId)
  .then(user => fetchOrders(user.id))
  .then(orders => fetchOrderDetails(orders[0].id))
  .then(details => console.log(details))
  .catch(error => console.error(error));

// Promise states:
// - Pending: Initial state
// - Fulfilled: Operation completed successfully
// - Rejected: Operation failed
```

## 10.3 Promise Static Methods

```javascript
// Promise.all - Wait for ALL to resolve (fails fast)
const results = await Promise.all([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);
// If ANY promise rejects, Promise.all rejects immediately

// Promise.allSettled - Wait for ALL to settle
const results = await Promise.allSettled([
  fetchUsers(),      // resolves
  fetchBadApi(),     // rejects
  fetchComments()    // resolves
]);
// results = [
//   { status: 'fulfilled', value: users },
//   { status: 'rejected', reason: error },
//   { status: 'fulfilled', value: comments }
// ]

// Promise.race - First to settle wins
const fastest = await Promise.race([
  fetchFromServer1(),
  fetchFromServer2()
]);

// Promise.any - First to FULFILL wins
const firstSuccess = await Promise.any([
  tryPrimaryDB(),
  trySecondaryDB(),
  tryCache()
]);
// Only rejects if ALL promises reject

// Promise.resolve / Promise.reject
Promise.resolve(42);              // Immediately resolved
Promise.reject(new Error('No')); // Immediately rejected
```

## 10.4 Async/Await

```javascript
// Async function always returns a Promise
async function fetchData() {
  return 'data';  // Wrapped in Promise.resolve()
}

// Await pauses execution until Promise settles
async function getUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw error;
  }
}

// Sequential vs Parallel
async function sequential() {
  const a = await fetchA();  // Wait
  const b = await fetchB();  // Then wait
  const c = await fetchC();  // Then wait
  // Total time: A + B + C
}

async function parallel() {
  const [a, b, c] = await Promise.all([
    fetchA(),
    fetchB(),
    fetchC()
  ]);
  // Total time: max(A, B, C)
}

// Error handling patterns
async function withErrorHandling() {
  // Pattern 1: try/catch
  try {
    const data = await fetchData();
  } catch (error) {
    console.error(error);
  }
  
  // Pattern 2: .catch() on await
  const data = await fetchData().catch(err => null);
  
  // Pattern 3: Wrapper function
  const [error, data] = await to(fetchData());
  if (error) handleError(error);
}

// Utility function for error handling
function to(promise) {
  return promise
    .then(data => [null, data])
    .catch(err => [err, null]);
}
```

## 10.5 Top-Level Await (ES2022)

```javascript
// In ES Modules, you can use await at the top level
// module.mjs
const response = await fetch('/api/config');
const config = await response.json();

export { config };
```

## 10.6 Async Iteration

```javascript
// Async generators
async function* asyncGenerator() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}

// for await...of
async function consumeAsync() {
  for await (const value of asyncGenerator()) {
    console.log(value);
  }
}

// Real-world: Streaming data
async function* fetchPages(url) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    yield data.items;
    hasMore = data.hasNextPage;
    page++;
  }
}
```

---

# 11. Error Handling

## 11.1 try/catch/finally

```javascript
try {
  // Code that might throw
  const result = riskyOperation();
  console.log(result);
} catch (error) {
  // Handle the error
  console.error('Error:', error.message);
} finally {
  // Always runs, even if error occurred
  cleanup();
}

// Catching specific error types
try {
  JSON.parse('invalid json');
} catch (error) {
  if (error instanceof SyntaxError) {
    console.log('Invalid JSON');
  } else if (error instanceof TypeError) {
    console.log('Type error');
  } else {
    throw error;  // Re-throw unknown errors
  }
}
```

## 11.2 Error Types

```javascript
// Built-in error types
new Error('Generic error');
new SyntaxError('Invalid syntax');
new TypeError('Wrong type');
new ReferenceError('Variable not defined');
new RangeError('Number out of range');
new URIError('Invalid URI');
new EvalError('Eval error');

// Error properties
const err = new Error('Something went wrong');
console.log(err.message);  // 'Something went wrong'
console.log(err.name);     // 'Error'
console.log(err.stack);    // Stack trace

// Custom errors
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

throw new ValidationError('Invalid email', 'email');
```

## 11.3 Async Error Handling

```javascript
// With Promises
fetchData()
  .then(data => processData(data))
  .catch(error => {
    console.error('Error:', error);
    return defaultValue;
  });

// With async/await
async function getData() {
  try {
    const data = await fetchData();
    return processData(data);
  } catch (error) {
    console.error('Error:', error);
    return defaultValue;
  }
}

// Unhandled rejections
window.addEventListener('unhandledrejection', event => {
  console.error('Unhandled rejection:', event.reason);
  event.preventDefault();
});

// Global error handler
window.addEventListener('error', event => {
  console.error('Global error:', event.error);
});
```

---

# 12. Memory Management

## 12.1 Garbage Collection

```javascript
// JavaScript automatically manages memory
// Objects are garbage collected when no longer reachable

let user = { name: 'Jayesh' };  // Object created
user = null;  // Object becomes unreachable, will be GC'd

// Mark-and-Sweep Algorithm:
// 1. Mark all reachable objects starting from roots
// 2. Sweep (delete) all unmarked objects

// Roots include:
// - Global variables
// - Currently executing functions
// - Active closures
```

## 12.2 Memory Leaks

```javascript
// 1. Accidental global variables
function leak() {
  leaked = 'oops';  // No 'var', 'let', or 'const'
}

// 2. Forgotten timers
setInterval(() => {
  const data = fetchHugeData();
  // Timer never cleared, data never released
}, 1000);

// 3. Closure leaks
function createClosure() {
  const hugeArray = new Array(1000000);
  return function() {
    console.log(hugeArray.length);
  };
}
const leak = createClosure();  // hugeArray stuck in memory

// 4. DOM references
const elements = [];
function addElement() {
  const el = document.createElement('div');
  elements.push(el);  // Reference kept even if removed from DOM
}

// 5. Event listeners not removed
element.addEventListener('click', handler);
// element.removeEventListener('click', handler);  // Forgotten
```

## 12.3 Best Practices

```javascript
// 1. Use WeakMap/WeakSet for metadata
const metadata = new WeakMap();
let obj = { data: 'value' };
metadata.set(obj, 'extra info');
obj = null;  // Both obj and metadata entry can be GC'd

// 2. Clean up event listeners
componentDidMount() {
  window.addEventListener('resize', this.handleResize);
}
componentWillUnmount() {
  window.removeEventListener('resize', this.handleResize);
}

// 3. Clear timers
const timer = setInterval(doSomething, 1000);
clearInterval(timer);  // When no longer needed

// 4. Nullify large objects
function processData() {
  let largeData = fetchLargeData();
  const result = process(largeData);
  largeData = null;  // Allow GC before function returns
  return result;
}
```

---

# 13. JavaScript Design Patterns

## 13.1 Creational Patterns

### Singleton Pattern

Ensures a class has only one instance and provides a global point of access.

```javascript
// ES6 Module Singleton (Recommended)
// config.js
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

Object.freeze(config);
export default config;

// Class-based Singleton
class Database {
  static instance = null;
  
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    
    this.connection = this.connect();
    Database.instance = this;
  }
  
  connect() {
    console.log('Connecting to database...');
    return { connected: true };
  }
  
  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}

const db1 = new Database();
const db2 = new Database();
console.log(db1 === db2);  // true

// Closure-based Singleton
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      name: 'Singleton Instance',
      timestamp: Date.now()
    };
  }
  
  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();
```

### Factory Pattern

Creates objects without specifying the exact class.

```javascript
// Simple Factory
class UserFactory {
  static createUser(type) {
    switch (type) {
      case 'admin':
        return new AdminUser();
      case 'regular':
        return new RegularUser();
      case 'guest':
        return new GuestUser();
      default:
        throw new Error(`Unknown user type: ${type}`);
    }
  }
}

const admin = UserFactory.createUser('admin');

// Factory with configuration
function createButton(options) {
  const button = document.createElement('button');
  button.textContent = options.text || 'Click';
  button.className = options.className || 'btn';
  button.onclick = options.onClick || (() => {});
  
  if (options.icon) {
    const icon = document.createElement('span');
    icon.className = `icon-${options.icon}`;
    button.prepend(icon);
  }
  
  return button;
}

const submitBtn = createButton({
  text: 'Submit',
  className: 'btn-primary',
  icon: 'check',
  onClick: () => handleSubmit()
});
```

### Builder Pattern

Separates object construction from its representation.

```javascript
class QueryBuilder {
  constructor() {
    this.query = {
      select: [],
      from: null,
      where: [],
      orderBy: null,
      limit: null
    };
  }
  
  select(...fields) {
    this.query.select = fields;
    return this;
  }
  
  from(table) {
    this.query.from = table;
    return this;
  }
  
  where(condition) {
    this.query.where.push(condition);
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderBy = { field, direction };
    return this;
  }
  
  limit(count) {
    this.query.limit = count;
    return this;
  }
  
  build() {
    let sql = `SELECT ${this.query.select.join(', ')} FROM ${this.query.from}`;
    
    if (this.query.where.length) {
      sql += ` WHERE ${this.query.where.join(' AND ')}`;
    }
    
    if (this.query.orderBy) {
      sql += ` ORDER BY ${this.query.orderBy.field} ${this.query.orderBy.direction}`;
    }
    
    if (this.query.limit) {
      sql += ` LIMIT ${this.query.limit}`;
    }
    
    return sql;
  }
}

const query = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('name')
  .limit(10)
  .build();

// "SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY name ASC LIMIT 10"
```

## 13.2 Structural Patterns

### Module Pattern

Encapsulates related code into a single unit.

```javascript
// IIFE Module
const Calculator = (function() {
  // Private variables
  let result = 0;
  
  // Private function
  function validate(n) {
    return typeof n === 'number' && !isNaN(n);
  }
  
  // Public API
  return {
    add(n) {
      if (validate(n)) result += n;
      return this;
    },
    subtract(n) {
      if (validate(n)) result -= n;
      return this;
    },
    multiply(n) {
      if (validate(n)) result *= n;
      return this;
    },
    getResult() {
      return result;
    },
    reset() {
      result = 0;
      return this;
    }
  };
})();

Calculator.add(10).multiply(2).subtract(5);
console.log(Calculator.getResult());  // 15

// Revealing Module Pattern
const Counter = (function() {
  let count = 0;
  
  function increment() {
    return ++count;
  }
  
  function decrement() {
    return --count;
  }
  
  function getCount() {
    return count;
  }
  
  function reset() {
    count = 0;
    return count;
  }
  
  return {
    increment,
    decrement,
    getCount,
    reset
  };
})();
```

### Decorator Pattern

Adds behavior to objects dynamically.

```javascript
// Function Decorator
function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with:`, args);
    const result = fn.apply(this, args);
    console.log(`${fn.name} returned:`, result);
    return result;
  };
}

function add(a, b) {
  return a + b;
}

const loggedAdd = withLogging(add);
loggedAdd(2, 3);  // Logs: Calling add with: [2, 3] → add returned: 5

// Class Method Decorator (using higher-order function)
function readonly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

// Decorator for timing
function timing(fn) {
  return async function(...args) {
    const start = performance.now();
    const result = await fn.apply(this, args);
    const end = performance.now();
    console.log(`${fn.name} took ${end - start}ms`);
    return result;
  };
}

// Decorator for retry
function retry(maxAttempts = 3) {
  return function(fn) {
    return async function(...args) {
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await fn.apply(this, args);
        } catch (error) {
          if (attempt === maxAttempts) throw error;
          console.log(`Attempt ${attempt} failed, retrying...`);
        }
      }
    };
  };
}

const fetchWithRetry = retry(3)(fetch);
```

### Proxy Pattern

Provides a surrogate or placeholder for another object.

```javascript
// Validation Proxy
function createValidatedObject(target, validators) {
  return new Proxy(target, {
    set(obj, prop, value) {
      if (validators[prop]) {
        if (!validators[prop](value)) {
          throw new Error(`Invalid value for ${prop}: ${value}`);
        }
      }
      obj[prop] = value;
      return true;
    }
  });
}

const user = createValidatedObject(
  { name: '', age: 0, email: '' },
  {
    age: (value) => typeof value === 'number' && value >= 0 && value <= 150,
    email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
  }
);

user.name = 'Jayesh';  // OK
user.age = 25;         // OK
// user.age = -5;      // Error: Invalid value for age
// user.email = 'bad'; // Error: Invalid value for email

// Lazy Loading Proxy
function createLazyLoader(loader) {
  let data = null;
  let loaded = false;
  
  return new Proxy({}, {
    get(target, prop) {
      if (!loaded) {
        data = loader();
        loaded = true;
      }
      return data[prop];
    }
  });
}

// Caching Proxy
function createCachingProxy(target) {
  const cache = new Map();
  
  return new Proxy(target, {
    apply(fn, thisArg, args) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        console.log('Cache hit!');
        return cache.get(key);
      }
      
      console.log('Cache miss, computing...');
      const result = fn.apply(thisArg, args);
      cache.set(key, result);
      return result;
    }
  });
}
```

## 13.3 Behavioral Patterns

### Observer Pattern

Defines a one-to-many dependency between objects.

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return () => this.off(event, listener);
  }
  
  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(l => l !== listener);
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach(listener => listener(...args));
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
}

// Usage
const emitter = new EventEmitter();

const unsubscribe = emitter.on('userLogin', (user) => {
  console.log(`${user.name} logged in`);
});

emitter.emit('userLogin', { name: 'Jayesh' });
unsubscribe();  // Remove listener

// Pub/Sub variation
class PubSub {
  static subscribers = {};
  
  static subscribe(topic, callback) {
    if (!this.subscribers[topic]) {
      this.subscribers[topic] = [];
    }
    this.subscribers[topic].push(callback);
  }
  
  static publish(topic, data) {
    if (!this.subscribers[topic]) return;
    this.subscribers[topic].forEach(cb => cb(data));
  }
}

PubSub.subscribe('news', (article) => console.log('New article:', article));
PubSub.publish('news', { title: 'Breaking News!' });
```

### Strategy Pattern

Defines a family of algorithms and makes them interchangeable.

```javascript
// Payment strategies
const paymentStrategies = {
  creditCard: (amount) => {
    console.log(`Paying ${amount} with Credit Card`);
    return { success: true, method: 'creditCard' };
  },
  
  paypal: (amount) => {
    console.log(`Paying ${amount} with PayPal`);
    return { success: true, method: 'paypal' };
  },
  
  crypto: (amount) => {
    console.log(`Paying ${amount} with Cryptocurrency`);
    return { success: true, method: 'crypto' };
  }
};

class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  pay(amount) {
    return this.strategy(amount);
  }
}

const processor = new PaymentProcessor(paymentStrategies.creditCard);
processor.pay(100);

processor.setStrategy(paymentStrategies.paypal);
processor.pay(50);

// Validation strategies
const validators = {
  email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  phone: (value) => /^\+?[\d\s-]{10,}$/.test(value),
  password: (value) => value.length >= 8 && /[A-Z]/.test(value) && /[0-9]/.test(value)
};

function validate(value, type) {
  const validator = validators[type];
  if (!validator) throw new Error(`Unknown validator: ${type}`);
  return validator(value);
}
```

### Command Pattern

Encapsulates a request as an object.

```javascript
// Text Editor with Undo/Redo
class TextEditor {
  constructor() {
    this.content = '';
    this.history = [];
    this.historyIndex = -1;
  }
  
  execute(command) {
    command.execute();
    
    // Remove any redo history after current position
    this.history = this.history.slice(0, this.historyIndex + 1);
    this.history.push(command);
    this.historyIndex++;
  }
  
  undo() {
    if (this.historyIndex < 0) return;
    
    const command = this.history[this.historyIndex];
    command.undo();
    this.historyIndex--;
  }
  
  redo() {
    if (this.historyIndex >= this.history.length - 1) return;
    
    this.historyIndex++;
    const command = this.history[this.historyIndex];
    command.execute();
  }
}

class InsertTextCommand {
  constructor(editor, text, position) {
    this.editor = editor;
    this.text = text;
    this.position = position;
  }
  
  execute() {
    const before = this.editor.content.slice(0, this.position);
    const after = this.editor.content.slice(this.position);
    this.editor.content = before + this.text + after;
  }
  
  undo() {
    const before = this.editor.content.slice(0, this.position);
    const after = this.editor.content.slice(this.position + this.text.length);
    this.editor.content = before + after;
  }
}

const editor = new TextEditor();
editor.execute(new InsertTextCommand(editor, 'Hello', 0));
editor.execute(new InsertTextCommand(editor, ' World', 5));
console.log(editor.content);  // "Hello World"
editor.undo();
console.log(editor.content);  // "Hello"
editor.redo();
console.log(editor.content);  // "Hello World"
```

---

# 14. Common Interview Coding Problems

## 14.1 Array Methods Implementation

### Implement Array.prototype.map

```javascript
Array.prototype.myMap = function(callback, thisArg) {
  if (typeof callback !== 'function') {
    throw new TypeError(`${callback} is not a function`);
  }
  
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this) {  // Handle sparse arrays
      result[i] = callback.call(thisArg, this[i], i, this);
    }
  }
  return result;
};

// Test
[1, 2, 3].myMap(x => x * 2);  // [2, 4, 6]
```

### Implement Array.prototype.filter

```javascript
Array.prototype.myFilter = function(callback, thisArg) {
  if (typeof callback !== 'function') {
    throw new TypeError(`${callback} is not a function`);
  }
  
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && callback.call(thisArg, this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

// Test
[1, 2, 3, 4, 5].myFilter(x => x > 2);  // [3, 4, 5]
```

### Implement Array.prototype.reduce

```javascript
Array.prototype.myReduce = function(callback, initialValue) {
  if (typeof callback !== 'function') {
    throw new TypeError(`${callback} is not a function`);
  }
  
  if (this.length === 0 && initialValue === undefined) {
    throw new TypeError('Reduce of empty array with no initial value');
  }
  
  let accumulator = initialValue;
  let startIndex = 0;
  
  if (initialValue === undefined) {
    accumulator = this[0];
    startIndex = 1;
  }
  
  for (let i = startIndex; i < this.length; i++) {
    if (i in this) {
      accumulator = callback(accumulator, this[i], i, this);
    }
  }
  
  return accumulator;
};

// Test
[1, 2, 3, 4].myReduce((acc, curr) => acc + curr, 0);  // 10
```

## 14.2 Function Methods Implementation

### Implement Function.prototype.bind

```javascript
Function.prototype.myBind = function(context, ...boundArgs) {
  if (typeof this !== 'function') {
    throw new TypeError('Bind must be called on a function');
  }
  
  const fn = this;
  
  return function(...args) {
    // Handle 'new' keyword
    if (new.target) {
      return new fn(...boundArgs, ...args);
    }
    return fn.apply(context, [...boundArgs, ...args]);
  };
};

// Test
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: 'Jayesh' };
const boundGreet = greet.myBind(person, 'Hello');
console.log(boundGreet('!'));  // "Hello, Jayesh!"
```

### Implement Function.prototype.call

```javascript
Function.prototype.myCall = function(context, ...args) {
  context = context ?? globalThis;
  
  // Convert primitives to objects
  if (typeof context !== 'object') {
    context = Object(context);
  }
  
  // Use Symbol to avoid property conflicts
  const fnSymbol = Symbol('fn');
  context[fnSymbol] = this;
  
  const result = context[fnSymbol](...args);
  
  delete context[fnSymbol];
  return result;
};

// Test
function greet() {
  return `Hello, ${this.name}`;
}

console.log(greet.myCall({ name: 'Jayesh' }));  // "Hello, Jayesh"
```

### Implement Function.prototype.apply

```javascript
Function.prototype.myApply = function(context, args = []) {
  context = context ?? globalThis;
  
  if (typeof context !== 'object') {
    context = Object(context);
  }
  
  const fnSymbol = Symbol('fn');
  context[fnSymbol] = this;
  
  const result = context[fnSymbol](...args);
  
  delete context[fnSymbol];
  return result;
};

// Test
function sum(a, b, c) {
  return a + b + c + this.base;
}

console.log(sum.myApply({ base: 10 }, [1, 2, 3]));  // 16
```

## 14.3 Utility Functions

### Debounce

```javascript
function debounce(fn, delay, options = {}) {
  let timeoutId;
  let lastArgs;
  let lastThis;
  const { leading = false, trailing = true } = options;
  
  function debounced(...args) {
    lastArgs = args;
    lastThis = this;
    
    const shouldCallNow = leading && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (trailing && lastArgs) {
        fn.apply(lastThis, lastArgs);
        lastArgs = lastThis = null;
      }
    }, delay);
    
    if (shouldCallNow) {
      fn.apply(this, args);
    }
  }
  
  debounced.cancel = function() {
    clearTimeout(timeoutId);
    timeoutId = lastArgs = lastThis = null;
  };
  
  return debounced;
}
```

### Throttle

```javascript
function throttle(fn, interval, options = {}) {
  let lastTime = 0;
  let timeoutId;
  let lastArgs;
  let lastThis;
  const { leading = true, trailing = true } = options;
  
  function throttled(...args) {
    const now = Date.now();
    
    if (!lastTime && !leading) {
      lastTime = now;
    }
    
    const remaining = interval - (now - lastTime);
    lastArgs = args;
    lastThis = this;
    
    if (remaining <= 0) {
      clearTimeout(timeoutId);
      timeoutId = null;
      lastTime = now;
      fn.apply(this, args);
      lastArgs = lastThis = null;
    } else if (!timeoutId && trailing) {
      timeoutId = setTimeout(() => {
        lastTime = leading ? Date.now() : 0;
        timeoutId = null;
        fn.apply(lastThis, lastArgs);
        lastArgs = lastThis = null;
      }, remaining);
    }
  }
  
  throttled.cancel = function() {
    clearTimeout(timeoutId);
    lastTime = 0;
    timeoutId = lastArgs = lastThis = null;
  };
  
  return throttled;
}
```

### Deep Clone

```javascript
function deepClone(obj, hash = new WeakMap()) {
  // Handle primitives and null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj);
  }
  
  // Handle RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj.source, obj.flags);
  }
  
  // Handle circular references
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  // Handle Array
  if (Array.isArray(obj)) {
    const arrCopy = [];
    hash.set(obj, arrCopy);
    obj.forEach((item, index) => {
      arrCopy[index] = deepClone(item, hash);
    });
    return arrCopy;
  }
  
  // Handle Object
  const objCopy = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, objCopy);
  
  for (const key of Reflect.ownKeys(obj)) {
    objCopy[key] = deepClone(obj[key], hash);
  }
  
  return objCopy;
}

// Test
const original = {
  a: 1,
  b: { c: 2 },
  d: [1, 2, { e: 3 }],
  f: new Date(),
  g: /test/gi
};
const cloned = deepClone(original);
```

### Deep Equal

```javascript
function deepEqual(a, b) {
  // Same reference or primitive
  if (a === b) return true;
  
  // Handle null
  if (a === null || b === null) return false;
  
  // Different types
  if (typeof a !== typeof b) return false;
  
  // Primitives (already handled by ===)
  if (typeof a !== 'object') return false;
  
  // Handle Date
  if (a instanceof Date && b instanceof Date) {
    return a.getTime() === b.getTime();
  }
  
  // Handle RegExp
  if (a instanceof RegExp && b instanceof RegExp) {
    return a.toString() === b.toString();
  }
  
  // Handle Array
  if (Array.isArray(a) !== Array.isArray(b)) return false;
  
  // Compare keys
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  // Compare each key recursively
  for (const key of keysA) {
    if (!keysB.includes(key)) return false;
    if (!deepEqual(a[key], b[key])) return false;
  }
  
  return true;
}
```

### Flatten Array

```javascript
// Recursive approach
function flatten(arr, depth = Infinity) {
  if (depth < 1) return arr.slice();
  
  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      return acc.concat(flatten(item, depth - 1));
    }
    return acc.concat(item);
  }, []);
}

// Iterative approach
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  
  while (stack.length) {
    const item = stack.pop();
    
    if (Array.isArray(item)) {
      stack.push(...item);
    } else {
      result.unshift(item);
    }
  }
  
  return result;
}

// Test
flatten([1, [2, [3, [4, 5]]]], 2);  // [1, 2, 3, [4, 5]]
flatten([1, [2, [3, [4, 5]]]]);     // [1, 2, 3, 4, 5]
```

### Curry Function

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

// Test
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3));     // 6
console.log(curriedAdd(1, 2)(3));     // 6
console.log(curriedAdd(1)(2, 3));     // 6
console.log(curriedAdd(1, 2, 3));     // 6

// Curry with placeholder
const _ = Symbol('placeholder');

function curryWithPlaceholder(fn) {
  return function curried(...args) {
    const complete = args.length >= fn.length && 
                    !args.slice(0, fn.length).includes(_);
    
    if (complete) {
      return fn.apply(this, args);
    }
    
    return function(...moreArgs) {
      const newArgs = args.map(arg => 
        arg === _ && moreArgs.length ? moreArgs.shift() : arg
      );
      return curried.apply(this, [...newArgs, ...moreArgs]);
    };
  };
}

const curriedSum = curryWithPlaceholder(add);
console.log(curriedSum(_, 2, _)(1)(3));  // 6
```

### Compose and Pipe

```javascript
// Compose - right to left
function compose(...fns) {
  return function(x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}

// Pipe - left to right
function pipe(...fns) {
  return function(x) {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}

// Async compose
function composeAsync(...fns) {
  return function(x) {
    return fns.reduceRight(
      (promise, fn) => promise.then(fn),
      Promise.resolve(x)
    );
  };
}

// Test
const add5 = x => x + 5;
const multiply2 = x => x * 2;
const subtract3 = x => x - 3;

const composed = compose(subtract3, multiply2, add5);  // (x + 5) * 2 - 3
const piped = pipe(add5, multiply2, subtract3);        // Same result

console.log(composed(10));  // 27
console.log(piped(10));     // 27
```

## 14.4 Promise Implementation

```javascript
class MyPromise {
  static PENDING = 'pending';
  static FULFILLED = 'fulfilled';
  static REJECTED = 'rejected';
  
  constructor(executor) {
    this.state = MyPromise.PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];
    
    const resolve = (value) => {
      if (this.state !== MyPromise.PENDING) return;
      
      this.state = MyPromise.FULFILLED;
      this.value = value;
      this.onFulfilledCallbacks.forEach(fn => fn());
    };
    
    const reject = (reason) => {
      if (this.state !== MyPromise.PENDING) return;
      
      this.state = MyPromise.REJECTED;
      this.reason = reason;
      this.onRejectedCallbacks.forEach(fn => fn());
    };
    
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }
  
  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
    onRejected = typeof onRejected === 'function' ? onRejected : e => { throw e; };
    
    const promise2 = new MyPromise((resolve, reject) => {
      const handleFulfilled = () => {
        queueMicrotask(() => {
          try {
            const x = onFulfilled(this.value);
            this.resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      };
      
      const handleRejected = () => {
        queueMicrotask(() => {
          try {
            const x = onRejected(this.reason);
            this.resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      };
      
      if (this.state === MyPromise.FULFILLED) {
        handleFulfilled();
      } else if (this.state === MyPromise.REJECTED) {
        handleRejected();
      } else {
        this.onFulfilledCallbacks.push(handleFulfilled);
        this.onRejectedCallbacks.push(handleRejected);
      }
    });
    
    return promise2;
  }
  
  resolvePromise(promise2, x, resolve, reject) {
    if (promise2 === x) {
      reject(new TypeError('Chaining cycle detected'));
      return;
    }
    
    if (x instanceof MyPromise) {
      x.then(resolve, reject);
    } else if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
      try {
        const then = x.then;
        if (typeof then === 'function') {
          then.call(x, resolve, reject);
        } else {
          resolve(x);
        }
      } catch (error) {
        reject(error);
      }
    } else {
      resolve(x);
    }
  }
  
  catch(onRejected) {
    return this.then(null, onRejected);
  }
  
  finally(callback) {
    return this.then(
      value => MyPromise.resolve(callback()).then(() => value),
      reason => MyPromise.resolve(callback()).then(() => { throw reason; })
    );
  }
  
  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }
  
  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason));
  }
  
  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const results = [];
      let completed = 0;
      
      if (promises.length === 0) {
        resolve(results);
        return;
      }
      
      promises.forEach((promise, index) => {
        MyPromise.resolve(promise).then(
          value => {
            results[index] = value;
            completed++;
            if (completed === promises.length) {
              resolve(results);
            }
          },
          reject
        );
      });
    });
  }
  
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        MyPromise.resolve(promise).then(resolve, reject);
      });
    });
  }
}
```

---

# 15. Interview Questions Bank

## 15.1 Core JavaScript Questions

**Q1: Explain the difference between `==` and `===`.**

**Answer:** 
- `==` (loose equality) performs type coercion before comparison
- `===` (strict equality) checks both value and type without coercion

```javascript
1 == '1'     // true (string coerced to number)
1 === '1'    // false (different types)
null == undefined  // true
null === undefined // false
```

**Q2: What is the difference between `null` and `undefined`?**

**Answer:**
- `undefined`: Variable declared but not assigned, or missing property
- `null`: Explicitly assigned to indicate "no value"

```javascript
let x;
console.log(x);         // undefined
console.log(typeof x);  // "undefined"

let y = null;
console.log(y);         // null
console.log(typeof y);  // "object" (historical bug)
```

**Q3: What is the event loop?**

**Answer:** The event loop is JavaScript's mechanism for handling asynchronous operations. It continuously checks if the call stack is empty, then processes microtasks (Promises), and finally processes one macrotask (setTimeout, I/O). This allows single-threaded JavaScript to handle non-blocking operations.

**Q4: What is a closure?**

**Answer:** A closure is a function that retains access to variables from its outer (enclosing) scope, even after the outer function has returned. Closures are created every time a function is created.

```javascript
function counter() {
  let count = 0;
  return function() {
    return ++count;
  };
}

const increment = counter();
increment();  // 1
increment();  // 2 - still has access to count
```

**Q5: Explain hoisting.**

**Answer:** Hoisting is JavaScript's behavior of moving declarations to the top of their scope during compilation. `var` declarations are hoisted and initialized as `undefined`. `let` and `const` are hoisted but remain in the Temporal Dead Zone until declared. Function declarations are fully hoisted.

**Q6: What is the Temporal Dead Zone?**

**Answer:** The TDZ is the period between entering a scope and the point where a `let` or `const` variable is declared. Accessing the variable during this time throws a `ReferenceError`.

**Q7: Explain the `this` keyword.**

**Answer:** `this` refers to the object executing the current function. Its value depends on how the function is called:
1. **new binding**: Newly created object
2. **Explicit binding** (call/apply/bind): Specified object
3. **Implicit binding** (method call): Object before the dot
4. **Default binding**: Global object (or undefined in strict mode)

**Q8: What is prototypal inheritance?**

**Answer:** JavaScript objects inherit properties from other objects through the prototype chain. Every object has an internal `[[Prototype]]` link. When accessing a property, JavaScript looks up the prototype chain until it finds the property or reaches `null`.

**Q9: What is the difference between `var`, `let`, and `const`?**

**Answer:**

| Feature | var | let | const |
|---------|-----|-----|-------|
| Scope | Function | Block | Block |
| Hoisting | Yes (undefined) | Yes (TDZ) | Yes (TDZ) |
| Redeclaration | Yes | No | No |
| Reassignment | Yes | Yes | No |

**Q10: What are arrow functions and how do they differ from regular functions?**

**Answer:**
1. No own `this` - inherits from enclosing scope
2. No `arguments` object
3. Cannot be used as constructors
4. No `prototype` property
5. Cannot be used as generators

## 15.2 ES6+ Questions

**Q11: What is destructuring?**

**Answer:** Destructuring is a syntax for extracting values from arrays or properties from objects into distinct variables.

**Q12: Explain spread and rest operators.**

**Answer:**
- **Spread (`...`)**: Expands an iterable into individual elements
- **Rest (`...`)**: Collects multiple elements into an array

**Q13: What are Promises?**

**Answer:** Promises represent the eventual completion or failure of an asynchronous operation. They can be in one of three states: pending, fulfilled, or rejected.

**Q14: What is async/await?**

**Answer:** `async/await` is syntactic sugar over Promises. `async` functions always return a Promise. `await` pauses execution until the Promise settles.

**Q15: What are generators?**

**Answer:** Generators are functions that can pause and resume execution. They use `function*` syntax and `yield` to produce values on demand.

## 15.3 Advanced Questions

**Q16: Explain the difference between shallow and deep copy.**

**Answer:**
- **Shallow copy**: Only copies the first level; nested objects share references
- **Deep copy**: Recursively copies all levels; completely independent

**Q17: What is memoization?**

**Answer:** Memoization is a technique to cache function results based on inputs. If the same inputs occur again, the cached result is returned instead of recomputing.

**Q18: Explain debouncing vs throttling.**

**Answer:**
- **Debounce**: Delays execution until after a period of inactivity
- **Throttle**: Limits execution to at most once per time interval

**Q19: What is currying?**

**Answer:** Currying transforms a function with multiple arguments into a sequence of functions, each taking a single argument.

**Q20: What are WeakMap and WeakSet?**

**Answer:** They hold weak references to keys/values, allowing garbage collection if no other references exist. Keys must be objects. They are not iterable.

---

## 📝 Quick Reference Cheat Sheet

### Type Checking
```javascript
typeof undefined      // "undefined"
typeof null           // "object" (bug)
typeof []             // "object"
typeof {}             // "object"
typeof function(){}   // "function"
Array.isArray([])     // true
x instanceof Array    // true
```

### Truthy/Falsy
```javascript
// Falsy values (6 total):
false, 0, -0, 0n, "", null, undefined, NaN

// Everything else is truthy, including:
[], {}, "0", "false", function(){}
```

### Equality
```javascript
null == undefined   // true
null === undefined  // false
NaN === NaN         // false
Object.is(NaN, NaN) // true
```

### Array Methods
```javascript
// Mutating: push, pop, shift, unshift, splice, sort, reverse, fill
// Non-mutating: map, filter, reduce, slice, concat, find, findIndex
```

### Object Shortcuts
```javascript
Object.keys(obj)        // Array of keys
Object.values(obj)      // Array of values
Object.entries(obj)     // Array of [key, value] pairs
Object.assign({}, obj)  // Shallow copy
{ ...obj }              // Spread copy
Object.freeze(obj)      // Make immutable
Object.seal(obj)        // Prevent add/delete
```

### Promise Methods
```javascript
Promise.all([])        // All resolve or first reject
Promise.allSettled([]) // All settle, never rejects
Promise.race([])       // First to settle
Promise.any([])        // First to fulfill
```

---

## 🎯 Interview Tips

1. **Always explain your thought process** - Interviewers want to see how you think

2. **Ask clarifying questions** - Edge cases, input constraints, expected output

3. **Start with a brute force solution** - Then optimize

4. **Test your code** - Walk through with examples

5. **Discuss trade-offs** - Time vs space, readability vs performance

6. **Know the fundamentals** - Event loop, closures, prototypes, 'this'

7. **Practice coding by hand** - Whiteboard interviews are still common

8. **Review your code** - Look for bugs before saying you're done

9. **Stay calm** - It's okay to say "I don't know" and work through it

10. **Keep learning** - The field evolves constantly

---

# 16. DOM Manipulation & Events

## 16.1 DOM Selection Methods

```javascript
// By ID - returns single element
const element = document.getElementById('myId');

// By class - returns HTMLCollection (live)
const elements = document.getElementsByClassName('myClass');

// By tag - returns HTMLCollection (live)
const divs = document.getElementsByTagName('div');

// Query selector - returns first match
const first = document.querySelector('.myClass');
const nested = document.querySelector('div.container > p:first-child');

// Query selector all - returns NodeList (static)
const all = document.querySelectorAll('.myClass');

// Differences:
// HTMLCollection: live (updates automatically), only elements
// NodeList: can be live or static, can include text nodes

// Convert to array for array methods
const arr = Array.from(document.querySelectorAll('div'));
const arr2 = [...document.querySelectorAll('div')];
```

## 16.2 DOM Traversal

```javascript
const element = document.querySelector('.child');

// Parent traversal
element.parentNode;           // Parent node (could be any node type)
element.parentElement;        // Parent element only
element.closest('.ancestor'); // Closest matching ancestor

// Child traversal
element.children;             // HTMLCollection of child elements
element.childNodes;           // NodeList including text nodes
element.firstChild;           // First child node
element.firstElementChild;    // First child element
element.lastChild;            // Last child node
element.lastElementChild;     // Last child element

// Sibling traversal
element.nextSibling;          // Next sibling node
element.nextElementSibling;   // Next sibling element
element.previousSibling;      // Previous sibling node
element.previousElementSibling; // Previous sibling element
```

## 16.3 DOM Manipulation

```javascript
// Creating elements
const div = document.createElement('div');
div.className = 'container';
div.id = 'myDiv';
div.textContent = 'Hello World';
div.innerHTML = '<span>Hello</span>';

// Attributes
div.setAttribute('data-id', '123');
div.getAttribute('data-id');
div.removeAttribute('data-id');
div.hasAttribute('data-id');
div.dataset.id;  // Access data-* attributes

// Adding to DOM
parent.appendChild(child);           // Add at end
parent.prepend(child);               // Add at beginning
parent.insertBefore(child, reference); // Insert before reference
parent.insertAdjacentHTML('beforeend', '<div>HTML</div>');
element.after(sibling);              // Insert after element
element.before(sibling);             // Insert before element

// Removing from DOM
element.remove();                    // Modern removal
parent.removeChild(child);           // Classic removal

// Replacing
parent.replaceChild(newChild, oldChild);
oldElement.replaceWith(newElement);

// Cloning
const clone = element.cloneNode(false);  // Shallow clone
const deepClone = element.cloneNode(true); // Deep clone
```

## 16.4 Event Handling

```javascript
// Adding event listeners
element.addEventListener('click', handler);
element.addEventListener('click', handler, { once: true });
element.addEventListener('scroll', handler, { passive: true });
element.addEventListener('click', handler, { capture: true });

// Removing event listeners
element.removeEventListener('click', handler);

// Event object
element.addEventListener('click', (event) => {
  event.target;           // Element that triggered the event
  event.currentTarget;    // Element the listener is attached to
  event.type;             // Event type ('click')
  event.preventDefault(); // Prevent default behavior
  event.stopPropagation(); // Stop bubbling
  event.stopImmediatePropagation(); // Stop all other listeners
});

// Event delegation
document.addEventListener('click', (event) => {
  if (event.target.matches('.button')) {
    handleButtonClick(event.target);
  }
  if (event.target.closest('.dropdown')) {
    handleDropdownClick(event.target.closest('.dropdown'));
  }
});

// Custom events
const customEvent = new CustomEvent('myEvent', {
  detail: { data: 'custom data' },
  bubbles: true,
  cancelable: true
});
element.dispatchEvent(customEvent);

// Listening for custom events
element.addEventListener('myEvent', (e) => {
  console.log(e.detail.data);  // 'custom data'
});
```

## 16.5 Event Bubbling and Capturing

```javascript
// Event phases:
// 1. Capture phase (top to bottom)
// 2. Target phase (at target element)
// 3. Bubbling phase (bottom to top)

/*
        ┌─────────────────────────────────┐
        │           document              │
        │  ┌───────────────────────────┐  │
        │  │         <html>            │  │
        │  │  ┌─────────────────────┐  │  │
        │  │  │      <body>         │  │  │
        │  │  │  ┌───────────────┐  │  │  │
        │  │  │  │    <div>      │  │  │  │
        │  │  │  │  ┌─────────┐  │  │  │  │
   ───────────│──│──│ BUTTON ──│──│──│──│──────►  Capture
  ◄───────────│──│──│ (click) ──│──│──│──│──────  Bubbling
        │  │  │  │  └─────────┘  │  │  │  │
        │  │  │  └───────────────┘  │  │  │
        │  │  └─────────────────────┘  │  │
        │  └───────────────────────────┘  │
        └─────────────────────────────────┘
*/

// Capture phase listener
element.addEventListener('click', handler, true);
// or
element.addEventListener('click', handler, { capture: true });

// Example: Event delegation with stopPropagation
outer.addEventListener('click', () => console.log('outer'));
inner.addEventListener('click', (e) => {
  console.log('inner');
  e.stopPropagation();  // Prevents 'outer' from logging
});
```

---

# 17. Regular Expressions

## 17.1 Creating Regular Expressions

```javascript
// Literal notation
const regex1 = /pattern/flags;
const regex2 = /hello/gi;  // global, case-insensitive

// Constructor notation
const regex3 = new RegExp('pattern', 'flags');
const regex4 = new RegExp('hello', 'gi');

// Flags
// g - global (find all matches)
// i - case-insensitive
// m - multiline (^ and $ match line start/end)
// s - dotAll (. matches newlines)
// u - unicode
// y - sticky (match from lastIndex)
```

## 17.2 Common Patterns

```javascript
// Character classes
/[abc]/       // Match a, b, or c
/[^abc]/      // Match anything except a, b, c
/[a-z]/       // Match a to z
/[A-Za-z0-9]/ // Match alphanumeric

// Shorthand character classes
/\d/          // Digit [0-9]
/\D/          // Non-digit [^0-9]
/\w/          // Word character [A-Za-z0-9_]
/\W/          // Non-word character
/\s/          // Whitespace [ \t\n\r\f\v]
/\S/          // Non-whitespace
/./           // Any character (except newline)

// Anchors
/^hello/      // Start of string
/world$/      // End of string
/\bword\b/    // Word boundary

// Quantifiers
/a*/          // 0 or more
/a+/          // 1 or more
/a?/          // 0 or 1
/a{3}/        // Exactly 3
/a{2,5}/      // 2 to 5
/a{2,}/       // 2 or more

// Groups
/(abc)/       // Capturing group
/(?:abc)/     // Non-capturing group
/(?<name>abc)/ // Named capturing group
/(a|b)/       // Alternation

// Lookahead/Lookbehind
/a(?=b)/      // Positive lookahead (a followed by b)
/a(?!b)/      // Negative lookahead (a not followed by b)
/(?<=a)b/     // Positive lookbehind (b preceded by a)
/(?<!a)b/     // Negative lookbehind (b not preceded by a)
```

## 17.3 RegExp Methods

```javascript
const regex = /hello/gi;
const str = 'Hello World, hello!';

// test() - returns boolean
regex.test(str);  // true

// exec() - returns match array or null
regex.exec(str);  // ['Hello', index: 0, input: 'Hello World, hello!', groups: undefined]

// String methods with regex
str.match(regex);     // ['Hello', 'hello']
str.matchAll(regex);  // Iterator of all matches with groups
str.replace(regex, 'hi');  // 'hi World, hi!'
str.replaceAll(regex, 'hi'); // 'hi World, hi!'
str.search(regex);    // 0 (index of first match)
str.split(/\s+/);     // ['Hello', 'World,', 'hello!']
```

## 17.4 Common Validation Patterns

```javascript
// Email
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// Phone (US)
const phoneRegex = /^\+?1?[-.\s]?\(?[0-9]{3}\)?[-.\s]?[0-9]{3}[-.\s]?[0-9]{4}$/;

// URL
const urlRegex = /^(https?:\/\/)?([\da-z.-]+)\.([a-z.]{2,6})([\/\w .-]*)*\/?$/;

// Password (min 8 chars, 1 uppercase, 1 lowercase, 1 number)
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{8,}$/;

// Credit Card (Visa, MasterCard, Amex, Discover)
const ccRegex = /^(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13}|6(?:011|5[0-9]{2})[0-9]{12})$/;

// Date (YYYY-MM-DD)
const dateRegex = /^\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\d|3[01])$/;

// IP Address
const ipRegex = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;

// Hex Color
const hexColorRegex = /^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$/;
```

---

# 18. Web APIs

## 18.1 Fetch API

```javascript
// Basic GET request
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// POST request with JSON
const response = await fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ name: 'Jayesh', age: 25 })
});

// Error handling
try {
  const response = await fetch(url);
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  const data = await response.json();
  return data;
} catch (error) {
  console.error('Fetch error:', error);
}

// AbortController for cancellation
const controller = new AbortController();

fetch(url, { signal: controller.signal })
  .then(response => response.json())
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('Fetch aborted');
    }
  });

// Cancel the request
controller.abort();

// Timeout wrapper
async function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal
    });
    clearTimeout(id);
    return response;
  } catch (error) {
    clearTimeout(id);
    throw error;
  }
}
```

## 18.2 LocalStorage & SessionStorage

```javascript
// localStorage - persists until explicitly cleared
localStorage.setItem('key', 'value');
localStorage.getItem('key');          // 'value'
localStorage.removeItem('key');
localStorage.clear();

// Store objects (must stringify)
localStorage.setItem('user', JSON.stringify({ name: 'Jayesh' }));
const user = JSON.parse(localStorage.getItem('user'));

// sessionStorage - cleared when tab/window closes
sessionStorage.setItem('key', 'value');
sessionStorage.getItem('key');
sessionStorage.removeItem('key');
sessionStorage.clear();

// Storage event (for cross-tab communication)
window.addEventListener('storage', (event) => {
  console.log('Key:', event.key);
  console.log('Old value:', event.oldValue);
  console.log('New value:', event.newValue);
  console.log('URL:', event.url);
});

// Storage wrapper with expiration
const storage = {
  set(key, value, ttl = null) {
    const item = {
      value,
      timestamp: Date.now(),
      ttl
    };
    localStorage.setItem(key, JSON.stringify(item));
  },
  
  get(key) {
    const itemStr = localStorage.getItem(key);
    if (!itemStr) return null;
    
    const item = JSON.parse(itemStr);
    
    if (item.ttl && Date.now() - item.timestamp > item.ttl) {
      localStorage.removeItem(key);
      return null;
    }
    
    return item.value;
  }
};
```

## 18.3 IndexedDB (Brief Overview)

```javascript
// Open database
const request = indexedDB.open('MyDatabase', 1);

request.onerror = () => console.error('Error opening DB');

request.onsuccess = (event) => {
  const db = event.target.result;
  console.log('DB opened successfully');
};

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  
  // Create object store (like a table)
  const store = db.createObjectStore('users', { keyPath: 'id' });
  store.createIndex('name', 'name', { unique: false });
};

// Add data
function addUser(db, user) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  store.add(user);
}

// Get data
function getUser(db, id) {
  return new Promise((resolve, reject) => {
    const transaction = db.transaction(['users'], 'readonly');
    const store = transaction.objectStore('users');
    const request = store.get(id);
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}
```

## 18.4 Intersection Observer

```javascript
// Lazy loading images
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
}, {
  root: null,        // viewport
  rootMargin: '0px', // margin around root
  threshold: 0.1     // 10% visible triggers callback
});

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});

// Infinite scroll
const infiniteObserver = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    loadMoreContent();
  }
}, { threshold: 1.0 });

infiniteObserver.observe(document.querySelector('#load-trigger'));
```

## 18.5 MutationObserver

```javascript
// Watch for DOM changes
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('Type:', mutation.type);
    console.log('Target:', mutation.target);
    console.log('Added nodes:', mutation.addedNodes);
    console.log('Removed nodes:', mutation.removedNodes);
  });
});

observer.observe(document.body, {
  childList: true,      // Watch for added/removed children
  subtree: true,        // Watch all descendants
  attributes: true,     // Watch for attribute changes
  characterData: true,  // Watch for text content changes
  attributeOldValue: true,
  characterDataOldValue: true
});

// Stop observing
observer.disconnect();
```

---

# 19. Advanced Coding Challenges

## 19.1 Implement EventEmitter

```javascript
class EventEmitter {
  constructor() {
    this.events = new Map();
  }
  
  on(event, listener) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event).push(listener);
    return this;
  }
  
  off(event, listener) {
    if (!this.events.has(event)) return this;
    
    const listeners = this.events.get(event);
    const index = listeners.indexOf(listener);
    if (index !== -1) {
      listeners.splice(index, 1);
    }
    return this;
  }
  
  emit(event, ...args) {
    if (!this.events.has(event)) return false;
    
    this.events.get(event).forEach(listener => {
      listener.apply(this, args);
    });
    return true;
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener.apply(this, args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
  
  removeAllListeners(event) {
    if (event) {
      this.events.delete(event);
    } else {
      this.events.clear();
    }
    return this;
  }
  
  listeners(event) {
    return this.events.get(event) || [];
  }
  
  listenerCount(event) {
    return this.listeners(event).length;
  }
}
```

## 19.2 Implement LRU Cache

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // Delete least recently used (first item)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}

// Usage
const cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);      // 1
cache.put(3, 3);   // Evicts key 2
cache.get(2);      // -1 (not found)
```

## 19.3 Implement Promise.all, race, allSettled

```javascript
// Promise.all
Promise.myAll = function(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }
    
    if (promises.length === 0) {
      return resolve([]);
    }
    
    const results = new Array(promises.length);
    let completed = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
};

// Promise.race
Promise.myRace = function(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(promise => {
      Promise.resolve(promise).then(resolve).catch(reject);
    });
  });
};

// Promise.allSettled
Promise.myAllSettled = function(promises) {
  return new Promise((resolve) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = new Array(promises.length);
    let completed = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        });
    });
  });
};

// Promise.any
Promise.myAny = function(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      reject(new AggregateError([], 'All promises were rejected'));
      return;
    }
    
    const errors = new Array(promises.length);
    let rejected = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(resolve)
        .catch(error => {
          errors[index] = error;
          rejected++;
          if (rejected === promises.length) {
            reject(new AggregateError(errors, 'All promises were rejected'));
          }
        });
    });
  });
};
```

## 19.4 Array Flattening (All Solutions)

```javascript
// 1. Recursive
function flattenRecursive(arr, depth = Infinity) {
  return depth > 0
    ? arr.reduce((acc, item) => 
        acc.concat(Array.isArray(item) 
          ? flattenRecursive(item, depth - 1) 
          : item), [])
    : arr.slice();
}

// 2. Iterative with stack
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  
  while (stack.length) {
    const next = stack.pop();
    if (Array.isArray(next)) {
      stack.push(...next);
    } else {
      result.unshift(next);
    }
  }
  
  return result;
}

// 3. Using generator
function* flattenGenerator(arr, depth = Infinity) {
  for (const item of arr) {
    if (Array.isArray(item) && depth > 0) {
      yield* flattenGenerator(item, depth - 1);
    } else {
      yield item;
    }
  }
}

// 4. Using toString (only works with numbers)
function flattenToString(arr) {
  return arr.toString().split(',').map(Number);
}

// 5. Using JSON stringify/parse (tricky, handles nested)
function flattenJSON(arr) {
  return JSON.parse(`[${JSON.stringify(arr).replace(/\[|\]/g, '')}]`);
}

// Tests
const nested = [1, [2, [3, [4, 5]]]];
console.log(flattenRecursive(nested));    // [1, 2, 3, 4, 5]
console.log(flattenIterative(nested));    // [1, 2, 3, 4, 5]
console.log([...flattenGenerator(nested)]); // [1, 2, 3, 4, 5]
```

## 19.5 Object Deep Freeze

```javascript
function deepFreeze(obj) {
  // Get all property names
  const propNames = Object.getOwnPropertyNames(obj);
  
  // Freeze nested objects first
  for (const name of propNames) {
    const value = obj[name];
    
    if (value && typeof value === 'object') {
      deepFreeze(value);
    }
  }
  
  return Object.freeze(obj);
}

// Test
const obj = {
  a: 1,
  b: {
    c: 2,
    d: {
      e: 3
    }
  },
  f: [1, 2, 3]
};

deepFreeze(obj);
obj.a = 10;        // Fails silently in non-strict mode
obj.b.c = 20;      // Also fails
obj.f.push(4);     // TypeError: Cannot add property
```

## 19.6 Implement setInterval with setTimeout

```javascript
function mySetInterval(callback, delay, ...args) {
  let cancelled = false;
  
  function tick() {
    if (!cancelled) {
      callback.apply(this, args);
      setTimeout(tick, delay);
    }
  }
  
  setTimeout(tick, delay);
  
  // Return cancel function
  return {
    clear: () => {
      cancelled = true;
    }
  };
}

// Usage
const interval = mySetInterval(() => console.log('tick'), 1000);
// After 5 seconds
setTimeout(() => interval.clear(), 5000);
```

## 19.7 Implement Promise.retry

```javascript
function retry(fn, maxRetries, delay = 0) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function attempt() {
      fn()
        .then(resolve)
        .catch(error => {
          attempts++;
          if (attempts >= maxRetries) {
            reject(error);
          } else {
            setTimeout(attempt, delay);
          }
        });
    }
    
    attempt();
  });
}

// With exponential backoff
function retryWithBackoff(fn, maxRetries, baseDelay = 1000) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function attempt() {
      fn()
        .then(resolve)
        .catch(error => {
          attempts++;
          if (attempts >= maxRetries) {
            reject(error);
          } else {
            const delay = baseDelay * Math.pow(2, attempts - 1);
            setTimeout(attempt, delay);
          }
        });
    }
    
    attempt();
  });
}

// Usage
retry(fetchData, 3, 1000)
  .then(data => console.log('Success:', data))
  .catch(err => console.log('Failed after 3 retries:', err));
```

---

# 20. Common Algorithm Questions

## 20.1 Two Sum

```javascript
function twoSum(nums, target) {
  const map = new Map();
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    
    map.set(nums[i], i);
  }
  
  return [];
}

// Time: O(n), Space: O(n)
twoSum([2, 7, 11, 15], 9);  // [0, 1]
```

## 20.2 Valid Parentheses

```javascript
function isValid(s) {
  const stack = [];
  const pairs = {
    ')': '(',
    '}': '{',
    ']': '['
  };
  
  for (const char of s) {
    if (char === '(' || char === '{' || char === '[') {
      stack.push(char);
    } else {
      if (stack.pop() !== pairs[char]) {
        return false;
      }
    }
  }
  
  return stack.length === 0;
}

isValid('()[]{}');  // true
isValid('([)]');    // false
```

## 20.3 Reverse Linked List

```javascript
function reverseList(head) {
  let prev = null;
  let current = head;
  
  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }
  
  return prev;
}

// Recursive solution
function reverseListRecursive(head) {
  if (head === null || head.next === null) {
    return head;
  }
  
  const newHead = reverseListRecursive(head.next);
  head.next.next = head;
  head.next = null;
  
  return newHead;
}
```

## 20.4 Merge Two Sorted Lists

```javascript
function mergeTwoLists(l1, l2) {
  const dummy = { next: null };
  let current = dummy;
  
  while (l1 !== null && l2 !== null) {
    if (l1.val <= l2.val) {
      current.next = l1;
      l1 = l1.next;
    } else {
      current.next = l2;
      l2 = l2.next;
    }
    current = current.next;
  }
  
  current.next = l1 !== null ? l1 : l2;
  
  return dummy.next;
}
```

## 20.5 Binary Search

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return -1;
}

// Time: O(log n)
binarySearch([1, 2, 3, 4, 5, 6, 7], 5);  // 4
```

## 20.6 Maximum Subarray (Kadane's Algorithm)

```javascript
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  
  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}

maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]);  // 6 (subarray [4, -1, 2, 1])
```

## 20.7 Valid Anagram

```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  
  const count = {};
  
  for (const char of s) {
    count[char] = (count[char] || 0) + 1;
  }
  
  for (const char of t) {
    if (!count[char]) return false;
    count[char]--;
  }
  
  return true;
}

isAnagram('anagram', 'nagaram');  // true
isAnagram('rat', 'car');          // false
```

## 20.8 Palindrome Check

```javascript
// String
function isPalindromeString(s) {
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

// Two pointers (more efficient)
function isPalindrome(s) {
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '');
  let left = 0;
  let right = cleaned.length - 1;
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }
  
  return true;
}

isPalindrome('A man, a plan, a canal: Panama');  // true
```

## 20.9 Find Duplicates

```javascript
// Using Set
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  
  for (const num of arr) {
    if (seen.has(num)) {
      duplicates.add(num);
    }
    seen.add(num);
  }
  
  return [...duplicates];
}

// Using object
function findDuplicatesObj(arr) {
  const count = {};
  const duplicates = [];
  
  for (const num of arr) {
    count[num] = (count[num] || 0) + 1;
  }
  
  for (const [num, cnt] of Object.entries(count)) {
    if (cnt > 1) {
      duplicates.push(Number(num));
    }
  }
  
  return duplicates;
}

findDuplicates([1, 2, 3, 2, 4, 3, 5]);  // [2, 3]
```

---

# 21. JavaScript Tricky Interview Questions

## 21.1 Output-Based Questions

**Q1: What's the output?**

```javascript
console.log(typeof typeof 1);
```

**Answer:** `"string"`

Explanation: `typeof 1` returns `"number"` (a string), and `typeof "number"` returns `"string"`.

---

**Q2: What's the output?**

```javascript
console.log(0.1 + 0.2 === 0.3);
```

**Answer:** `false`

Explanation: Due to floating-point precision, `0.1 + 0.2` equals `0.30000000000000004`.

Fix: `Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON`

---

**Q3: What's the output?**

```javascript
const a = {};
const b = { key: 'b' };
const c = { key: 'c' };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```

**Answer:** `456`

Explanation: Object keys are converted to strings. Both `b.toString()` and `c.toString()` return `"[object Object]"`, so they overwrite the same key.

---

**Q4: What's the output?**

```javascript
let x = 10;
let y = (x++, x);
console.log(x, y);
```

**Answer:** `11 11`

Explanation: The comma operator evaluates left to right and returns the last value. `x++` increments x to 11, then the expression returns x (11).

---

**Q5: What's the output?**

```javascript
console.log([] + []);
console.log([] + {});
console.log({} + []);
```

**Answer:**
```
""
"[object Object]"
"[object Object]"  // or 0 in some browsers when {} is parsed as a block
```

Explanation: Arrays and objects are converted to strings when using `+` operator.

---

**Q6: What's the output?**

```javascript
const obj = {
  a: 1,
  b: function() {
    return this.a;
  },
  c: () => {
    return this.a;
  }
};

console.log(obj.b());
console.log(obj.c());
```

**Answer:**
```
1
undefined
```

Explanation: Arrow functions don't have their own `this`. They inherit from the enclosing scope (global/module scope), where `this.a` is undefined.

---

**Q7: What's the output?**

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 1);
}
```

**Answer:**
```
3 3 3
0 1 2
```

Explanation: `var` is function-scoped, so all callbacks share the same `i`. `let` is block-scoped, so each iteration gets its own `j`.

---

**Q8: What's the output?**

```javascript
console.log(typeof null);
console.log(typeof undefined);
console.log(null === undefined);
console.log(null == undefined);
```

**Answer:**
```
"object"
"undefined"
false
true
```

Explanation: `typeof null` returning "object" is a historical bug. `null == undefined` is true due to type coercion.

---

**Q9: What's the output?**

```javascript
function foo() {
  return
  {
    bar: "hello"
  };
}

console.log(foo());
```

**Answer:** `undefined`

Explanation: JavaScript inserts a semicolon after `return` due to ASI (Automatic Semicolon Insertion). The object is never returned.

---

**Q10: What's the output?**

```javascript
console.log('1' - - '1');
console.log('1' + - '1');
```

**Answer:**
```
2
"1-1"
```

Explanation: 
- `'1' - - '1'` → `1 - (-1)` → `2` (subtraction coerces to numbers)
- `'1' + - '1'` → `'1' + (-1)` → `'1-1'` (+ with string does concatenation, -'1' becomes -1 first)

Wait, actually:
- `'1' + - '1'` → `'1' + (-1)` → `"1-1"` is WRONG
- The actual output is `"1-1"` only if there's no space, but with spaces it's `'1' + (-1)` = `"1-1"`

Let me recalculate:
- `-'1'` converts to `-1`
- `'1' + (-1)` = `'1-1'` because + with a string does concatenation

Actually the answer is: `"1-1"`

---

## 21.2 More Advanced Questions

**Q11: Implement a function that can be called like `add(1)(2)(3)()` and returns 6.**

```javascript
function add(x) {
  let sum = x;
  
  function inner(y) {
    if (y === undefined) {
      return sum;
    }
    sum += y;
    return inner;
  }
  
  return inner;
}

console.log(add(1)(2)(3)());  // 6
console.log(add(1)(2)(3)(4)(5)());  // 15
```

---

**Q12: Create a sleep function using async/await.**

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function demo() {
  console.log('Start');
  await sleep(2000);
  console.log('2 seconds later');
}
```

---

**Q13: Implement `_.get(object, path, defaultValue)`**

```javascript
function get(obj, path, defaultValue = undefined) {
  const keys = Array.isArray(path) 
    ? path 
    : path.replace(/\[(\d+)\]/g, '.$1').split('.');
  
  let result = obj;
  
  for (const key of keys) {
    if (result == null) {
      return defaultValue;
    }
    result = result[key];
  }
  
  return result === undefined ? defaultValue : result;
}

// Usage
const obj = { a: { b: { c: [1, 2, 3] } } };
get(obj, 'a.b.c[0]');          // 1
get(obj, 'a.b.x', 'default');  // 'default'
```

---

**Q14: Implement function composition from right to left.**

```javascript
const compose = (...fns) => (x) => fns.reduceRight((v, fn) => fn(v), x);

const add10 = x => x + 10;
const multiply2 = x => x * 2;
const subtract5 = x => x - 5;

const composed = compose(subtract5, multiply2, add10);
console.log(composed(5));  // ((5 + 10) * 2) - 5 = 25
```

---

**Q15: Implement negative array indexing.**

```javascript
function createArrayWithNegativeIndex(arr) {
  return new Proxy(arr, {
    get(target, prop) {
      const index = Number(prop);
      
      if (!isNaN(index) && index < 0) {
        return target[target.length + index];
      }
      
      return target[prop];
    }
  });
}

const arr = createArrayWithNegativeIndex([1, 2, 3, 4, 5]);
console.log(arr[-1]);  // 5
console.log(arr[-2]);  // 4
```

---

*End of Part 1: JavaScript Foundations - Complete Mastery Guide*

> **Total Coverage:** 6000+ lines of comprehensive JavaScript content including:
> - Core concepts (Engine, Event Loop, Closures, Prototypes)
> - All ES6+ features
> - Design patterns
> - 50+ coding implementations
> - 100+ interview questions
> - DOM manipulation and Web APIs
> - Algorithm questions and solutions
> - Tricky output-based questions

*Continue to Part 2 for TypeScript Mastery...*

