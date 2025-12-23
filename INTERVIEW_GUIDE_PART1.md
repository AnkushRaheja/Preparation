# 🚀 CodeHealth-AI Frontend Interview Mastery Guide

> **Complete Study Material for Frontend Developer Interviews (Junior to Senior Level)**

This comprehensive guide covers everything you need to master the CodeHealth-AI frontend stack. Each section includes theory, practical examples from our codebase, and common interview questions.

---

## 📚 Table of Contents

1. [Part 1: JavaScript Foundations](#part-1-javascript-foundations)
2. [Part 2: TypeScript Mastery](#part-2-typescript-mastery)
3. [Part 3: React.js Deep Dive](#part-3-reactjs-deep-dive)
4. [Part 4: React Hooks Complete Guide](#part-4-react-hooks-complete-guide)
5. [Part 5: Next.js Framework](#part-5-nextjs-framework)
6. [Part 6: State Management with Zustand](#part-6-state-management-with-zustand)
7. [Part 7: Networking & API Integration](#part-7-networking--api-integration)
8. [Part 8: Security Best Practices](#part-8-security-best-practices)
9. [Part 9: Real-Time Systems (Socket.io)](#part-9-real-time-systems-socketio)
10. [Part 10: Data Visualization](#part-10-data-visualization)
11. [Part 11: Styling & UI Architecture](#part-11-styling--ui-architecture)
12. [Part 12: Performance Optimization](#part-12-performance-optimization)
13. [Part 13: CI/CD & Deployment](#part-13-cicd--deployment)

---

# Part 1: JavaScript Foundations

## 1.1 Execution Context & Call Stack

### What is Execution Context?
Every time JavaScript code runs, it runs inside an **Execution Context**. Think of it as a "box" where your code is evaluated.

**Types of Execution Contexts:**
1. **Global Execution Context (GEC)**: Created when the script first runs. There's only one.
2. **Function Execution Context (FEC)**: Created every time a function is called.
3. **Eval Execution Context**: Created inside `eval()` function (rarely used).

### The Call Stack
The **Call Stack** is a LIFO (Last In, First Out) data structure that keeps track of function calls.

```javascript
function first() {
  console.log('First function');
  second();
  console.log('First function end');
}

function second() {
  console.log('Second function');
  third();
  console.log('Second function end');
}

function third() {
  console.log('Third function');
}

first();

// Call Stack visualization:
// 1. [Global]
// 2. [Global, first()]
// 3. [Global, first(), second()]
// 4. [Global, first(), second(), third()]
// 5. [Global, first(), second()] - third() popped
// 6. [Global, first()] - second() popped
// 7. [Global] - first() popped
```

---

## 1.2 The Event Loop (CRITICAL for Interviews)

JavaScript is **single-threaded** but can handle async operations. How? The **Event Loop**.

### Components:
1. **Call Stack**: Executes synchronous code
2. **Web APIs**: Browser features (setTimeout, fetch, DOM events)
3. **Callback Queue (Task Queue/Macrotask Queue)**: setTimeout, setInterval, I/O
4. **Microtask Queue**: Promises (.then/.catch/.finally), queueMicrotask, MutationObserver

### Priority Order:
1. ✅ Call Stack (empties first)
2. ✅ Microtask Queue (ALL microtasks run before any macrotask)
3. ✅ Macrotask Queue (one task at a time)

```javascript
console.log('1: Script Start');

setTimeout(() => {
  console.log('2: setTimeout (Macrotask)');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise (Microtask)');
});

queueMicrotask(() => {
  console.log('4: queueMicrotask (Microtask)');
});

console.log('5: Script End');

// OUTPUT ORDER:
// 1: Script Start
// 5: Script End
// 3: Promise (Microtask)
// 4: queueMicrotask (Microtask)
// 2: setTimeout (Macrotask)
```

### Interview Question: "Why does setTimeout with 0ms not run immediately?"
**Answer**: Because setTimeout schedules a callback to the Macrotask Queue. The Event Loop must:
1. Empty the Call Stack
2. Empty ALL Microtasks
3. THEN pick ONE Macrotask

---

## 1.3 Closures (Most Asked Topic)

### Definition
A **closure** is a function that "remembers" variables from its outer (enclosing) scope, even after that outer function has returned.

```javascript
function createCounter() {
  let count = 0; // This variable is "enclosed" in the closure
  
  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
// The inner function "remembers" count even after createCounter() finished
```

### Real CodeHealth Example: Authentication Token Storage
```javascript
// From authStore.ts pattern
function createAuthStore() {
  let token = null; // Enclosed variable
  
  return {
    setToken: (newToken) => { token = newToken; },
    getToken: () => token,
    isAuthenticated: () => token !== null
  };
}
```

### The "Stale Closure" Problem (React Specific)
```javascript
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      // ❌ BUG: This always logs 0 because the closure captured count=0
      console.log(count);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // Empty dependency array = effect runs once with count=0

  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}

// FIX: Add count to dependencies OR use a ref
```

---

## 1.4 Hoisting

### What is Hoisting?
JavaScript moves declarations to the top of their scope during the compilation phase.

```javascript
// What you write:
console.log(x); // undefined (not ReferenceError!)
var x = 5;

// How JavaScript interprets it:
var x;           // Declaration is hoisted
console.log(x);  // undefined
x = 5;           // Assignment stays in place
```

### var vs let/const Hoisting
```javascript
// var: Hoisted and initialized with undefined
console.log(a); // undefined
var a = 1;

// let/const: Hoisted but NOT initialized (Temporal Dead Zone)
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 2;
```

### Function Hoisting
```javascript
// Function declarations are fully hoisted
sayHello(); // "Hello!" - Works!
function sayHello() {
  console.log("Hello!");
}

// Function expressions are NOT hoisted
sayBye(); // TypeError: sayBye is not a function
var sayBye = function() {
  console.log("Bye!");
};
```

---

## 1.5 this Keyword

### Rules (in order of precedence):
1. **new binding**: `this` = the newly created object
2. **Explicit binding**: `call()`, `apply()`, `bind()` explicitly set `this`
3. **Implicit binding**: `obj.method()` - `this` = `obj`
4. **Default binding**: `this` = global object (or undefined in strict mode)

```javascript
const user = {
  name: 'Jayesh',
  greet: function() {
    console.log(`Hello, ${this.name}`);
  }
};

user.greet(); // "Hello, Jayesh" (Implicit binding)

const greetFn = user.greet;
greetFn(); // "Hello, undefined" (Default binding - this = window)

// Fix with bind:
const boundGreet = user.greet.bind(user);
boundGreet(); // "Hello, Jayesh"
```

### Arrow Functions & this
Arrow functions don't have their own `this`. They inherit from the enclosing scope.

```javascript
const user = {
  name: 'Jayesh',
  greet: () => {
    // Arrow function: this = enclosing scope (window/global)
    console.log(`Hello, ${this.name}`); // "Hello, undefined"
  },
  greetCorrect: function() {
    // Regular function inside
    const inner = () => {
      // Arrow function inherits this from greetCorrect
      console.log(`Hello, ${this.name}`); // "Hello, Jayesh"
    };
    inner();
  }
};
```

---

## 1.6 Prototypes & Inheritance

### Prototype Chain
Every JavaScript object has an internal `[[Prototype]]` link to another object.

```javascript
const animal = {
  eats: true,
  walk() {
    console.log('Walking...');
  }
};

const dog = Object.create(animal);
dog.barks = true;

console.log(dog.eats);  // true (inherited from animal)
console.log(dog.barks); // true (own property)
dog.walk(); // "Walking..." (inherited method)

// Prototype chain: dog -> animal -> Object.prototype -> null
```

### ES6 Classes (Syntactic Sugar)
```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Must call super() before using 'this'
    this.breed = breed;
  }
  
  speak() {
    console.log(`${this.name} barks`);
  }
}

const rex = new Dog('Rex', 'German Shepherd');
rex.speak(); // "Rex barks"
```

---

## 1.7 Async JavaScript

### Callbacks → Promises → Async/Await Evolution

**Callbacks (Old Way - "Callback Hell"):**
```javascript
getUserData(userId, (user) => {
  getOrders(user.id, (orders) => {
    getOrderDetails(orders[0].id, (details) => {
      // Deeply nested = hard to read and maintain
    });
  });
});
```

**Promises (Better):**
```javascript
getUserData(userId)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => console.log(details))
  .catch(error => console.error(error));
```

**Async/Await (Best - Used in CodeHealth):**
```javascript
// From analysisStore.ts pattern
async function fetchFullAnalysis(repoId) {
  try {
    const response = await axiosInstance.get(`/analysis/${repoId}`);
    return response.data;
  } catch (error) {
    // Error handling
    throw new Error('Failed to fetch analysis');
  }
}
```

### Promise Methods
```javascript
// Promise.all - Wait for ALL promises (fails fast if any reject)
const [users, posts, comments] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);

// Promise.allSettled - Wait for ALL, get results even if some fail
const results = await Promise.allSettled([promise1, promise2, promise3]);
// results = [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]

// Promise.race - First to settle wins
const fastest = await Promise.race([fetchFromCDN1(), fetchFromCDN2()]);

// Promise.any - First to FULFILL wins (ignores rejections)
const firstSuccess = await Promise.any([tryServer1(), tryServer2()]);
```

---

## 🎯 Part 1 Interview Questions

1. **Explain the Event Loop in JavaScript.**
2. **What is a closure? Give an example.**
3. **What is the difference between var, let, and const?**
4. **Explain hoisting.**
5. **What is the Temporal Dead Zone?**
6. **How does 'this' work in JavaScript?**
7. **What's the difference between == and ===?**
8. **Explain prototypal inheritance.**
9. **What's the difference between Promise.all and Promise.allSettled?**
10. **How do async/await work under the hood?**

---

*Continue to Part 2 for TypeScript Mastery...*
