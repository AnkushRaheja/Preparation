# JavaScript Complete Notes

## Table of Contents
1. [Data Types](#data-types)
2. [Variables & Declarations](#variables--declarations)
3. [Scope](#scope)
4. [Hoisting](#hoisting)
5. [Closures](#closures)
6. [this Keyword](#this-keyword)
7. [Prototypes & Inheritance](#prototypes--inheritance)
8. [Asynchronous JavaScript](#asynchronous-javascript)
9. [Event Loop](#event-loop)
10. [ES6+ Features](#es6-features)
11. [Interview Questions](#interview-questions)

---

## Data Types

JavaScript has **8 data types** divided into two categories:

### Primitive Data Types (7)

```javascript
// 1. String
let name = "John";
let greeting = 'Hello';
let template = `Hello ${name}`; // Template literal

// 2. Number (integers and floats)
let age = 25;
let price = 99.99;
let infinity = Infinity;
let notANumber = NaN;

// 3. BigInt (for very large integers)
let bigNumber = 9007199254740991n;
let anotherBig = BigInt("123456789012345678901234567890");

// 4. Boolean
let isActive = true;
let isLoggedIn = false;

// 5. Undefined (variable declared but not assigned)
let x;
console.log(x); // undefined

// 6. Null (intentional absence of value)
let empty = null;

// 7. Symbol (unique identifier)
let sym1 = Symbol('description');
let sym2 = Symbol('description');
console.log(sym1 === sym2); // false (always unique)
```

### Non-Primitive (Reference) Data Type (1)

```javascript
// 8. Object (includes arrays, functions, dates, etc.)
let person = { name: "John", age: 30 };
let numbers = [1, 2, 3, 4, 5];
let greet = function() { return "Hello"; };
let today = new Date();
```

### typeof Operator

```javascript
console.log(typeof "Hello");      // "string"
console.log(typeof 42);           // "number"
console.log(typeof 42n);          // "bigint"
console.log(typeof true);         // "boolean"
console.log(typeof undefined);    // "undefined"
console.log(typeof null);         // "object" (historical bug)
console.log(typeof Symbol());     // "symbol"
console.log(typeof {});           // "object"
console.log(typeof []);           // "object"
console.log(typeof function(){}); // "function"
```

### Type Coercion

```javascript
// Implicit Coercion
console.log("5" + 3);      // "53" (number to string)
console.log("5" - 3);      // 2 (string to number)
console.log("5" * "2");    // 10
console.log(true + 1);     // 2
console.log(false + 1);    // 1
console.log("5" == 5);     // true (loose equality)
console.log("5" === 5);    // false (strict equality)

// Explicit Coercion
console.log(Number("42"));     // 42
console.log(String(42));       // "42"
console.log(Boolean(1));       // true
console.log(parseInt("42px")); // 42
console.log(parseFloat("3.14")); // 3.14
```

### Truthy and Falsy Values

```javascript
// Falsy values (6 total)
false
0
-0
"" (empty string)
null
undefined
NaN

// Everything else is truthy
// Examples of truthy values:
"0"        // non-empty string
"false"    // non-empty string
[]         // empty array
{}         // empty object
function(){} // function
```

---

## Variables & Declarations

### var, let, and const

```javascript
// var - function scoped, hoisted, can be redeclared
var x = 10;
var x = 20; // OK

// let - block scoped, hoisted (TDZ), cannot be redeclared
let y = 10;
// let y = 20; // Error: already declared

// const - block scoped, hoisted (TDZ), cannot be reassigned
const z = 10;
// z = 20; // Error: Assignment to constant variable

// const with objects (reference is constant, not the value)
const obj = { name: "John" };
obj.name = "Jane"; // OK
// obj = {}; // Error
```

### Comparison Table

| Feature | var | let | const |
|---------|-----|-----|-------|
| Scope | Function | Block | Block |
| Hoisting | Yes (undefined) | Yes (TDZ) | Yes (TDZ) |
| Redeclaration | Yes | No | No |
| Reassignment | Yes | Yes | No |
| Global object property | Yes | No | No |

---

## Scope

### Types of Scope

```javascript
// 1. Global Scope
var globalVar = "I'm global";
let globalLet = "I'm also global";

// 2. Function Scope
function myFunction() {
    var functionScoped = "Only inside function";
    let alsoFunctionScoped = "Also only inside";
}
// console.log(functionScoped); // Error

// 3. Block Scope (let and const only)
if (true) {
    var varVariable = "I escape the block";
    let letVariable = "I stay in block";
    const constVariable = "I also stay";
}
console.log(varVariable);   // Works
// console.log(letVariable); // Error

// 4. Lexical Scope (Static Scope)
function outer() {
    let outerVar = "outer";
    
    function inner() {
        console.log(outerVar); // Can access outer's variables
    }
    inner();
}
```

### Scope Chain

```javascript
let global = "global";

function outer() {
    let outerVar = "outer";
    
    function middle() {
        let middleVar = "middle";
        
        function inner() {
            let innerVar = "inner";
            // Can access: innerVar, middleVar, outerVar, global
            console.log(innerVar, middleVar, outerVar, global);
        }
        inner();
    }
    middle();
}
outer();
```

---

## Hoisting

### Variable Hoisting

```javascript
// What we write:
console.log(x); // undefined (not error!)
var x = 5;

// How JS interprets it:
var x;
console.log(x); // undefined
x = 5;

// let and const - Temporal Dead Zone (TDZ)
// console.log(y); // ReferenceError: Cannot access before initialization
let y = 5;
```

### Function Hoisting

```javascript
// Function declarations are fully hoisted
sayHello(); // Works! "Hello"
function sayHello() {
    console.log("Hello");
}

// Function expressions are NOT hoisted
// sayBye(); // TypeError: sayBye is not a function
var sayBye = function() {
    console.log("Bye");
};

// Arrow functions behave like function expressions
// greet(); // TypeError
var greet = () => console.log("Hi");
```

---

## Closures

A closure is a function that remembers and accesses variables from its outer scope, even after the outer function has returned.

```javascript
// Basic Closure
function outer() {
    let count = 0;
    
    return function inner() {
        count++;
        return count;
    };
}

const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3

// Practical Example: Private Variables
function createBankAccount(initialBalance) {
    let balance = initialBalance; // Private variable
    
    return {
        deposit(amount) {
            balance += amount;
            return balance;
        },
        withdraw(amount) {
            if (amount <= balance) {
                balance -= amount;
                return balance;
            }
            return "Insufficient funds";
        },
        getBalance() {
            return balance;
        }
    };
}

const account = createBankAccount(100);
console.log(account.getBalance()); // 100
console.log(account.deposit(50));  // 150
console.log(account.balance);      // undefined (private!)

// Closure in Loops (Common Pitfall)
// Problem with var
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 1000);
}
// Output: 3, 3, 3 (all same!)

// Solution 1: Use let
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 1000);
}
// Output: 0, 1, 2

// Solution 2: IIFE
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => console.log(j), 1000);
    })(i);
}
// Output: 0, 1, 2
```

---

## this Keyword

The value of `this` depends on **how** a function is called.

```javascript
// 1. Global Context
console.log(this); // window (browser) / global (Node.js)

// 2. Object Method
const person = {
    name: "John",
    greet() {
        console.log(this.name); // "John"
    }
};
person.greet();

// 3. Regular Function
function showThis() {
    console.log(this);
}
showThis(); // window (non-strict) / undefined (strict mode)

// 4. Arrow Functions (lexical this)
const obj = {
    name: "John",
    regular: function() {
        console.log(this.name); // "John"
    },
    arrow: () => {
        console.log(this.name); // undefined (inherits from outer scope)
    }
};

// 5. Constructor Function
function Person(name) {
    this.name = name;
}
const john = new Person("John");
console.log(john.name); // "John"

// 6. Explicit Binding: call, apply, bind
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: "John" };

greet.call(user, "Hello", "!");     // "Hello, John!"
greet.apply(user, ["Hi", "?"]);     // "Hi, John?"
const boundGreet = greet.bind(user, "Hey");
boundGreet("...");                   // "Hey, John..."
```

### this in Event Handlers

```javascript
// Regular function: this = element that triggered event
button.addEventListener('click', function() {
    console.log(this); // button element
});

// Arrow function: this = outer scope (usually window)
button.addEventListener('click', () => {
    console.log(this); // window
});
```

---

## Prototypes & Inheritance

### Prototype Chain

```javascript
// Every object has a [[Prototype]] (accessible via __proto__ or Object.getPrototypeOf)
const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true
console.log(arr.__proto__.__proto__ === Object.prototype); // true
console.log(arr.__proto__.__proto__.__proto__); // null

// Constructor Function Prototype
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    console.log(`${this.name} makes a sound`);
};

const dog = new Animal("Dog");
dog.speak(); // "Dog makes a sound"
```

### ES6 Classes

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        console.log(`${this.name} makes a sound`);
    }
    
    static info() {
        console.log("Animals are living beings");
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Call parent constructor
        this.breed = breed;
    }
    
    speak() {
        console.log(`${this.name} barks`);
    }
}

const myDog = new Dog("Buddy", "Golden Retriever");
myDog.speak(); // "Buddy barks"
Animal.info(); // "Animals are living beings"
```

---

## Asynchronous JavaScript (In-Depth)

JavaScript is **single-threaded** but uses an event-driven, non-blocking model to handle asynchronous operations. This section covers callbacks, promises, and async/await in complete detail.

### Why Asynchronous?

```javascript
// Synchronous (Blocking) - BAD for I/O operations
const data = readFileSync('large-file.txt'); // Blocks entire program
console.log(data);
console.log("This waits..."); // Doesn't run until file is read

// Asynchronous (Non-Blocking) - GOOD
readFile('large-file.txt', (data) => {
    console.log(data);
});
console.log("This runs immediately!"); // Runs while file is being read
```

---

### Callbacks

A callback is a function passed as an argument to another function, to be executed later.

```javascript
// Basic Callback Example
function greet(name, callback) {
    console.log("Hello, " + name);
    callback();
}

greet("John", function() {
    console.log("Callback executed!");
});
// Output:
// Hello, John
// Callback executed!

// Asynchronous Callback
function fetchData(callback) {
    setTimeout(() => {
        const data = { id: 1, name: "John" };
        callback(null, data); // Node.js convention: error-first callback
    }, 2000);
}

fetchData((error, data) => {
    if (error) {
        console.log("Error:", error);
        return;
    }
    console.log("Data:", data);
});
console.log("Fetching...");
// Output:
// Fetching...
// Data: { id: 1, name: "John" } (after 2 seconds)
```

#### Callback Hell (Pyramid of Doom)

```javascript
// The Problem: Deeply nested callbacks
getUser(userId, (err, user) => {
    if (err) return handleError(err);
    
    getPosts(user.id, (err, posts) => {
        if (err) return handleError(err);
        
        getComments(posts[0].id, (err, comments) => {
            if (err) return handleError(err);
            
            getLikes(comments[0].id, (err, likes) => {
                if (err) return handleError(err);
                
                // Finally do something with likes
                console.log(likes);
            });
        });
    });
});

// Problems:
// 1. Hard to read and maintain
// 2. Error handling is repetitive
// 3. Difficult to add new steps
// 4. No way to handle multiple async operations together
```

---

### Promises (Complete Guide)

A **Promise** is an object representing the eventual completion or failure of an asynchronous operation.

#### Promise States

```
┌─────────────┐
│   PENDING   │  ← Initial state
└─────────────┘
       │
       ├──────────────┬──────────────┐
       ▼              ▼              │
┌─────────────┐ ┌─────────────┐      │
│  FULFILLED  │ │  REJECTED   │      │
│  (resolved) │ │  (error)    │      │
└─────────────┘ └─────────────┘      │
                                     │
Note: Once settled (fulfilled/rejected), a promise CANNOT change state
```

#### Creating Promises

```javascript
// Basic Promise Creation
const myPromise = new Promise((resolve, reject) => {
    // Async operation here
    const success = true;
    
    if (success) {
        resolve("Operation successful!"); // Fulfill the promise
    } else {
        reject(new Error("Operation failed!")); // Reject the promise
    }
});

// Simulating async operation
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (userId > 0) {
                resolve({ id: userId, name: "John", email: "john@example.com" });
            } else {
                reject(new Error("Invalid user ID"));
            }
        }, 1000);
    });
}

// Promise that wraps an API call
function getData(url) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.onload = () => {
            if (xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(`HTTP Error: ${xhr.status}`));
            }
        };
        xhr.onerror = () => reject(new Error("Network Error"));
        xhr.send();
    });
}
```

#### Consuming Promises: .then(), .catch(), .finally()

```javascript
// .then(onFulfilled, onRejected)
myPromise.then(
    (result) => console.log("Success:", result),
    (error) => console.log("Error:", error)
);

// Better pattern: .then() + .catch()
myPromise
    .then((result) => {
        console.log("Success:", result);
        return result; // Return value becomes input for next .then()
    })
    .catch((error) => {
        console.log("Error:", error);
    })
    .finally(() => {
        console.log("Cleanup code - runs regardless of outcome");
    });

// Real Example
fetchUserData(1)
    .then((user) => {
        console.log("User:", user);
    })
    .catch((error) => {
        console.log("Failed to fetch user:", error.message);
    })
    .finally(() => {
        console.log("Request completed");
    });
```

#### Promise Chaining (Solving Callback Hell)

```javascript
// Each .then() returns a new Promise
// You can chain them for sequential async operations

fetchUser(userId)
    .then((user) => {
        console.log("Got user:", user.name);
        return fetchPosts(user.id); // Return a Promise
    })
    .then((posts) => {
        console.log("Got posts:", posts.length);
        return fetchComments(posts[0].id); // Return another Promise
    })
    .then((comments) => {
        console.log("Got comments:", comments.length);
        return fetchLikes(comments[0].id);
    })
    .then((likes) => {
        console.log("Got likes:", likes);
    })
    .catch((error) => {
        // Single catch handles errors from ANY step above
        console.log("Error occurred:", error.message);
    });

// Returning values vs Returning Promises
Promise.resolve(1)
    .then((x) => x + 1)           // Returns value 2
    .then((x) => {
        return new Promise((resolve) => {
            setTimeout(() => resolve(x + 1), 1000);
        });
    })                             // Returns Promise that resolves to 3
    .then((x) => x + 1)           // Returns value 4
    .then((x) => console.log(x)); // Output: 4 (after 1 second)
```

#### Promise Static Methods

```javascript
// 1. Promise.resolve() - Creates already-resolved promise
const resolved = Promise.resolve("Already done!");
resolved.then(console.log); // "Already done!"

// 2. Promise.reject() - Creates already-rejected promise
const rejected = Promise.reject(new Error("Failed!"));
rejected.catch(console.log); // Error: Failed!

// 3. Promise.all() - Wait for ALL promises, fail if ANY fails
const p1 = fetch('/api/users');
const p2 = fetch('/api/posts');
const p3 = fetch('/api/comments');

Promise.all([p1, p2, p3])
    .then(([users, posts, comments]) => {
        // All three completed successfully
        console.log("Users:", users);
        console.log("Posts:", posts);
        console.log("Comments:", comments);
    })
    .catch((error) => {
        // If ANY promise rejects, we come here
        console.log("One of them failed:", error);
    });

// Example with timing
const slow = new Promise(resolve => setTimeout(() => resolve("slow"), 3000));
const fast = new Promise(resolve => setTimeout(() => resolve("fast"), 1000));

Promise.all([slow, fast]).then(console.log);
// Output after 3 seconds: ["slow", "fast"]
// Note: Waits for ALL to complete

// 4. Promise.allSettled() - Wait for ALL, get all results (ES2020)
const promises = [
    Promise.resolve("Success 1"),
    Promise.reject("Error 1"),
    Promise.resolve("Success 2")
];

Promise.allSettled(promises).then((results) => {
    console.log(results);
    // [
    //   { status: "fulfilled", value: "Success 1" },
    //   { status: "rejected", reason: "Error 1" },
    //   { status: "fulfilled", value: "Success 2" }
    // ]
});

// 5. Promise.race() - Returns first SETTLED (resolved OR rejected)
const p1 = new Promise(resolve => setTimeout(() => resolve("First"), 1000));
const p2 = new Promise(resolve => setTimeout(() => resolve("Second"), 2000));

Promise.race([p1, p2]).then(console.log); // "First" (after 1 second)

// Race with rejection
const fast = new Promise((_, reject) => setTimeout(() => reject("Error!"), 500));
const slow = new Promise(resolve => setTimeout(() => resolve("Success"), 1000));

Promise.race([fast, slow]).catch(console.log); // "Error!" (rejection wins)

// 6. Promise.any() - Returns first FULFILLED, ignores rejections (ES2021)
const promises = [
    Promise.reject("Error 1"),
    new Promise(resolve => setTimeout(() => resolve("Success!"), 1000)),
    Promise.reject("Error 2")
];

Promise.any(promises).then(console.log); // "Success!" (first fulfilled)

// If ALL reject, throws AggregateError
Promise.any([
    Promise.reject("Error 1"),
    Promise.reject("Error 2")
]).catch((error) => {
    console.log(error); // AggregateError: All promises were rejected
    console.log(error.errors); // ["Error 1", "Error 2"]
});
```

#### Common Promise Patterns

```javascript
// 1. Converting callback to Promise (Promisification)
function readFilePromise(filename) {
    return new Promise((resolve, reject) => {
        fs.readFile(filename, 'utf8', (err, data) => {
            if (err) reject(err);
            else resolve(data);
        });
    });
}

// 2. Timeout wrapper
function withTimeout(promise, ms) {
    const timeout = new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Timeout!")), ms);
    });
    return Promise.race([promise, timeout]);
}

withTimeout(fetch('/api/data'), 5000)
    .then(data => console.log(data))
    .catch(err => console.log(err.message)); // "Timeout!" if > 5 seconds

// 3. Retry logic
function retry(fn, retries = 3, delay = 1000) {
    return fn().catch((error) => {
        if (retries === 0) throw error;
        return new Promise(resolve => setTimeout(resolve, delay))
            .then(() => retry(fn, retries - 1, delay));
    });
}

retry(() => fetch('/api/flaky-endpoint'), 3, 2000)
    .then(console.log)
    .catch(console.log);

// 4. Sequential execution of promises
async function sequential(urls) {
    const results = [];
    for (const url of urls) {
        const result = await fetch(url);
        results.push(result);
    }
    return results;
}

// 5. Parallel execution with concurrency limit
async function parallelLimit(tasks, limit) {
    const results = [];
    const executing = [];
    
    for (const task of tasks) {
        const p = task().then(result => {
            executing.splice(executing.indexOf(p), 1);
            return result;
        });
        results.push(p);
        executing.push(p);
        
        if (executing.length >= limit) {
            await Promise.race(executing);
        }
    }
    
    return Promise.all(results);
}
```

#### Promise Error Handling

```javascript
// Errors propagate down the chain until caught
fetchUser()
    .then(user => fetchPosts(user.id))
    .then(posts => {
        throw new Error("Something went wrong!");
    })
    .then(result => {
        // This is SKIPPED
        console.log("Never reached");
    })
    .catch(error => {
        // Catches error from ANY .then() above
        console.log("Caught:", error.message);
        return "Recovered!"; // Can return value to continue chain
    })
    .then(result => {
        console.log(result); // "Recovered!"
    });

// Multiple catch blocks for different handling
fetchUser()
    .then(user => {
        if (!user.verified) {
            throw new Error("User not verified");
        }
        return fetchPosts(user.id);
    })
    .catch(error => {
        // Handle user-related errors
        console.log("User error:", error.message);
        return []; // Return empty array to continue
    })
    .then(posts => processPost(posts))
    .catch(error => {
        // Handle post-related errors
        console.log("Post error:", error.message);
    });

// IMPORTANT: Unhandled rejections
const promise = Promise.reject("Error!");
// Warning in console: Unhandled promise rejection

// Always handle rejections!
promise.catch(() => {}); // Or handle appropriately
```

---

### Async/Await (Complete Guide)

`async/await` is syntactic sugar over Promises, making asynchronous code look and behave like synchronous code.

#### Basic Syntax

```javascript
// Async function declaration
async function fetchData() {
    return "Hello";
}

// Async function expression
const fetchData = async function() {
    return "Hello";
};

// Async arrow function
const fetchData = async () => {
    return "Hello";
};

// Async method in object
const obj = {
    async getData() {
        return "Hello";
    }
};

// Async method in class
class DataFetcher {
    async fetch() {
        return "Hello";
    }
}
```

#### How async Functions Work

```javascript
// An async function ALWAYS returns a Promise
async function greet() {
    return "Hello"; // Equivalent to: return Promise.resolve("Hello")
}

greet().then(console.log); // "Hello"

// Even if you return a non-Promise value, it's wrapped in a Promise
async function getValue() {
    return 42;
}

getValue().then(value => console.log(value)); // 42

// Throwing in async function = rejected promise
async function failingFunction() {
    throw new Error("Failed!");
}

failingFunction().catch(error => console.log(error.message)); // "Failed!"
```

#### The await Keyword

```javascript
// await pauses execution until Promise settles
async function fetchUser() {
    console.log("Starting fetch...");
    
    const response = await fetch('/api/user'); // Pauses here
    console.log("Got response");
    
    const user = await response.json(); // Pauses here
    console.log("Parsed JSON");
    
    return user;
}

// IMPORTANT: await can ONLY be used inside async functions
function regularFunction() {
    // await fetch('/api'); // SyntaxError!
}

// Top-level await (ES2022) - only in ES modules
// In a .mjs file or with "type": "module" in package.json
const data = await fetch('/api/data');
console.log(data);
```

#### Error Handling with try/catch

```javascript
// Basic try/catch
async function fetchUser(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const user = await response.json();
        return user;
    } catch (error) {
        console.log("Error fetching user:", error.message);
        return null; // Return default value
    } finally {
        console.log("Fetch attempt completed");
    }
}

// Multiple try/catch blocks for granular error handling
async function processData() {
    let user;
    
    try {
        user = await fetchUser(1);
    } catch (error) {
        console.log("Failed to fetch user:", error);
        return;
    }
    
    try {
        const posts = await fetchPosts(user.id);
        return posts;
    } catch (error) {
        console.log("Failed to fetch posts:", error);
        return [];
    }
}

// Alternative: Handle errors per await without try/catch
async function fetchData() {
    const user = await fetchUser(1).catch(err => null);
    
    if (!user) {
        console.log("No user found");
        return;
    }
    
    const posts = await fetchPosts(user.id).catch(err => []);
    return posts;
}
```

#### Sequential vs Parallel Execution

```javascript
// SEQUENTIAL - One after another (slower)
async function sequential() {
    console.time("sequential");
    
    const user = await fetchUser();      // Wait 1 second
    const posts = await fetchPosts();    // Wait 1 second
    const comments = await fetchComments(); // Wait 1 second
    
    console.timeEnd("sequential"); // Total: ~3 seconds
    return { user, posts, comments };
}

// PARALLEL - All at once (faster)
async function parallel() {
    console.time("parallel");
    
    // Start all fetches at the same time
    const userPromise = fetchUser();
    const postsPromise = fetchPosts();
    const commentsPromise = fetchComments();
    
    // Wait for all to complete
    const user = await userPromise;
    const posts = await postsPromise;
    const comments = await commentsPromise;
    
    console.timeEnd("parallel"); // Total: ~1 second (max of all)
    return { user, posts, comments };
}

// PARALLEL with Promise.all (cleaner)
async function parallelClean() {
    console.time("parallel");
    
    const [user, posts, comments] = await Promise.all([
        fetchUser(),
        fetchPosts(),
        fetchComments()
    ]);
    
    console.timeEnd("parallel"); // Total: ~1 second
    return { user, posts, comments };
}

// When to use which?
// Sequential: When each step depends on the previous
async function dependentOperations() {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);     // Needs user.id
    const firstPostComments = await fetchComments(posts[0].id); // Needs post id
}

// Parallel: When operations are independent
async function independentOperations() {
    const [weather, news, stocks] = await Promise.all([
        fetchWeather(),
        fetchNews(),
        fetchStocks()
    ]);
}
```

#### Async/Await in Loops

```javascript
// ❌ WRONG: forEach doesn't wait for async
async function wrong() {
    const urls = ['/api/1', '/api/2', '/api/3'];
    
    urls.forEach(async (url) => {
        const data = await fetch(url);
        console.log(data); // Fires unpredictably!
    });
    
    console.log("Done!"); // Runs BEFORE fetches complete!
}

// ✅ CORRECT: for...of waits for each iteration
async function sequential() {
    const urls = ['/api/1', '/api/2', '/api/3'];
    
    for (const url of urls) {
        const response = await fetch(url);
        const data = await response.json();
        console.log(data);
    }
    
    console.log("Done!"); // Runs AFTER all fetches complete
}

// ✅ CORRECT: map + Promise.all for parallel
async function parallel() {
    const urls = ['/api/1', '/api/2', '/api/3'];
    
    const promises = urls.map(url => fetch(url));
    const responses = await Promise.all(promises);
    
    const data = await Promise.all(
        responses.map(response => response.json())
    );
    
    console.log(data); // Array of all results
    console.log("Done!");
}

// ✅ Using for await...of (for async iterables)
async function* asyncGenerator() {
    yield await Promise.resolve(1);
    yield await Promise.resolve(2);
    yield await Promise.resolve(3);
}

async function consumeGenerator() {
    for await (const value of asyncGenerator()) {
        console.log(value);
    }
}
```

#### Async/Await with Classes

```javascript
class UserService {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }
    
    async getUser(id) {
        const response = await fetch(`${this.baseUrl}/users/${id}`);
        if (!response.ok) throw new Error("User not found");
        return response.json();
    }
    
    async getUserWithPosts(id) {
        const user = await this.getUser(id);
        const posts = await this.getPosts(user.id);
        return { ...user, posts };
    }
    
    async getPosts(userId) {
        const response = await fetch(`${this.baseUrl}/posts?userId=${userId}`);
        return response.json();
    }
}

const service = new UserService('https://api.example.com');
const user = await service.getUserWithPosts(1);
```

#### Common Async/Await Patterns

```javascript
// 1. Retry with exponential backoff
async function fetchWithRetry(url, retries = 3, delay = 1000) {
    for (let i = 0; i < retries; i++) {
        try {
            return await fetch(url);
        } catch (error) {
            if (i === retries - 1) throw error;
            await new Promise(resolve => setTimeout(resolve, delay * Math.pow(2, i)));
        }
    }
}

// 2. Timeout wrapper
async function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const response = await fetch(url, { signal: controller.signal });
        return response;
    } finally {
        clearTimeout(timeoutId);
    }
}

// 3. Mutex/Lock for sequential execution
class AsyncLock {
    constructor() {
        this.promise = Promise.resolve();
    }
    
    async acquire(fn) {
        const release = this.promise;
        let resolver;
        this.promise = new Promise(resolve => resolver = resolve);
        await release;
        try {
            return await fn();
        } finally {
            resolver();
        }
    }
}

const lock = new AsyncLock();
// These will run sequentially despite being called in parallel
lock.acquire(async () => await doTask1());
lock.acquire(async () => await doTask2());

// 4. Debounced async function
function debounceAsync(fn, wait) {
    let timeoutId;
    let pendingPromise = null;
    let resolver = null;
    
    return function(...args) {
        if (timeoutId) clearTimeout(timeoutId);
        
        if (!pendingPromise) {
            pendingPromise = new Promise(resolve => resolver = resolve);
        }
        
        timeoutId = setTimeout(async () => {
            const result = await fn.apply(this, args);
            resolver(result);
            pendingPromise = null;
        }, wait);
        
        return pendingPromise;
    };
}
```

#### Converting Between Callbacks, Promises, and Async/Await

```javascript
// Callback style
function getDataCallback(callback) {
    setTimeout(() => {
        callback(null, "data");
    }, 1000);
}

// Convert callback to Promise (Promisification)
function getDataPromise() {
    return new Promise((resolve, reject) => {
        getDataCallback((error, data) => {
            if (error) reject(error);
            else resolve(data);
        });
    });
}

// Use with async/await
async function getData() {
    const data = await getDataPromise();
    return data;
}

// Node.js util.promisify (for Node.js callback functions)
const util = require('util');
const fs = require('fs');

const readFilePromise = util.promisify(fs.readFile);

async function readConfig() {
    const data = await readFilePromise('config.json', 'utf8');
    return JSON.parse(data);
}
```

---

### Promises vs Async/Await Comparison

```javascript
// The same operation in both styles

// Promise chain
function getUserData(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(user => fetch(`/api/posts?userId=${user.id}`))
        .then(response => response.json())
        .then(posts => ({ user, posts }))
        .catch(error => {
            console.log("Error:", error);
            throw error;
        });
}

// Async/Await
async function getUserData(userId) {
    try {
        const userResponse = await fetch(`/api/users/${userId}`);
        const user = await userResponse.json();
        
        const postsResponse = await fetch(`/api/posts?userId=${user.id}`);
        const posts = await postsResponse.json();
        
        return { user, posts };
    } catch (error) {
        console.log("Error:", error);
        throw error;
    }
}

// When to use Promises:
// - Simple single async operation
// - When you need Promise.all, Promise.race, etc.
// - Functional programming style

// When to use Async/Await:
// - Complex sequential async operations
// - When you need try/catch for error handling
// - When code readability is priority
// - When working with loops
```

---

## Event Loop

```javascript
console.log("1");

setTimeout(() => {
    console.log("2");
}, 0);

Promise.resolve().then(() => {
    console.log("3");
});

console.log("4");

// Output: 1, 4, 3, 2
```

### Execution Order

1. **Call Stack**: Synchronous code executes first
2. **Microtask Queue**: Promises, queueMicrotask, MutationObserver
3. **Macrotask Queue**: setTimeout, setInterval, setImmediate, I/O

```javascript
console.log("Start");

setTimeout(() => console.log("Timeout 1"), 0);
setTimeout(() => console.log("Timeout 2"), 0);

Promise.resolve()
    .then(() => console.log("Promise 1"))
    .then(() => console.log("Promise 2"));

queueMicrotask(() => console.log("Microtask"));

console.log("End");

// Output:
// Start
// End
// Promise 1
// Microtask
// Promise 2
// Timeout 1
// Timeout 2
```

---

## ES6+ Features

### Destructuring

```javascript
// Array Destructuring
const [a, b, ...rest] = [1, 2, 3, 4, 5];
console.log(a, b, rest); // 1, 2, [3, 4, 5]

// Object Destructuring
const { name, age, city = "Unknown" } = { name: "John", age: 30 };
console.log(name, age, city); // "John", 30, "Unknown"

// Renaming
const { name: userName } = { name: "John" };
console.log(userName); // "John"

// Nested Destructuring
const { address: { city } } = { address: { city: "NYC" } };
```

### Spread & Rest Operators

```javascript
// Spread (expands)
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }

// Rest (collects)
function sum(...numbers) {
    return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10
```

### Optional Chaining & Nullish Coalescing

```javascript
// Optional Chaining (?.)
const user = { address: { city: "NYC" } };
console.log(user?.address?.city);  // "NYC"
console.log(user?.phone?.number);  // undefined (no error)

// Nullish Coalescing (??)
const value = null ?? "default";     // "default"
const value2 = 0 ?? "default";       // 0 (only null/undefined)
const value3 = 0 || "default";       // "default" (falsy check)
```

### Other ES6+ Features

```javascript
// Template Literals
const name = "John";
const greeting = `Hello, ${name}!
This is a multiline string.`;

// Arrow Functions
const add = (a, b) => a + b;

// Default Parameters
function greet(name = "Guest") {
    return `Hello, ${name}`;
}

// Map and Set
const map = new Map();
map.set("key", "value");
map.get("key"); // "value"

const set = new Set([1, 2, 2, 3]);
console.log([...set]); // [1, 2, 3]

// for...of
for (const item of [1, 2, 3]) {
    console.log(item);
}

// Modules
export const PI = 3.14159;
export default function calculate() {}
import calculate, { PI } from './math.js';
```

---

## Interview Questions

### Section 1: Output-Based Questions

#### Question 1: Hoisting
```javascript
console.log(a);
console.log(b);
var a = 1;
let b = 2;
```
**Output:**
```
undefined
ReferenceError: Cannot access 'b' before initialization
```
**Explanation:** `var` is hoisted with value `undefined`. `let` is hoisted but in TDZ.

---

#### Question 2: Closure
```javascript
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 1000);
}
```
**Output:**
```
3
3
3
```
**Explanation:** `var` is function-scoped. By the time callbacks run, loop has finished and `i` is 3.

---

#### Question 3: this keyword
```javascript
const obj = {
    name: "John",
    getName: function() {
        return this.name;
    },
    getNameArrow: () => {
        return this.name;
    }
};

console.log(obj.getName());
console.log(obj.getNameArrow());
```
**Output:**
```
John
undefined
```
**Explanation:** Arrow functions don't have their own `this`. They inherit from outer scope (window/global).

---

#### Question 4: Event Loop
```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```
**Output:**
```
A
D
C
B
```
**Explanation:** Sync first (A, D), then microtasks (C), then macrotasks (B).

---

#### Question 5: Prototype
```javascript
function Animal() {}
Animal.prototype.speak = function() { return "Sound"; };

function Dog() {}
Dog.prototype = Object.create(Animal.prototype);

const dog = new Dog();
console.log(dog.speak());
console.log(dog.constructor === Dog);
```
**Output:**
```
Sound
false
```
**Explanation:** `speak` is inherited. `constructor` was overwritten when we replaced `Dog.prototype`.

---

#### Question 6: Type Coercion
```javascript
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log([] == false);
console.log(!![] == true);
```
**Output:**
```
""
"[object Object]"
"[object Object]" (or 0 in some contexts)
true
true
```

---

#### Question 7: Variable Shadowing
```javascript
var x = 10;
function foo() {
    console.log(x);
    var x = 20;
    console.log(x);
}
foo();
console.log(x);
```
**Output:**
```
undefined
20
10
```
**Explanation:** Local `x` is hoisted inside function, shadowing global `x`.

---

#### Question 8: Async/Await
```javascript
async function test() {
    console.log("1");
    await Promise.resolve();
    console.log("2");
}

console.log("3");
test();
console.log("4");
```
**Output:**
```
3
1
4
2
```
**Explanation:** After `await`, remaining code becomes a microtask.

---

#### Question 9: Object Reference
```javascript
let a = { x: 1 };
let b = a;
a.x = 2;
console.log(b.x);

a = { x: 3 };
console.log(b.x);
```
**Output:**
```
2
2
```
**Explanation:** `b` still references original object. `a = { x: 3 }` creates new reference for `a` only.

---

#### Question 10: Spread vs Reference
```javascript
const original = { a: 1, b: { c: 2 } };
const copy = { ...original };

copy.a = 10;
copy.b.c = 20;

console.log(original.a);
console.log(original.b.c);
```
**Output:**
```
1
20
```
**Explanation:** Spread creates shallow copy. Nested objects are still referenced.

---

#### Question 11: Promise Chain
```javascript
Promise.resolve(1)
    .then(x => x + 1)
    .then(x => { throw new Error("Error!") })
    .then(x => x + 1)
    .catch(err => 10)
    .then(x => console.log(x));
```
**Output:**
```
10
```
**Explanation:** Error is caught, `.catch` returns 10, chain continues.

---

#### Question 12: IIFE and Closure
```javascript
var result = [];
for (var i = 0; i < 3; i++) {
    result.push((function(j) {
        return function() { return j; };
    })(i));
}
console.log(result[0]());
console.log(result[1]());
console.log(result[2]());
```
**Output:**
```
0
1
2
```
**Explanation:** IIFE captures each value of `i` in separate closures.

---

### Section 2: Theoretical Questions

#### Q1: What is the difference between `==` and `===`?
**Answer:** 
- `==` (loose equality) compares values after type coercion
- `===` (strict equality) compares both value and type without coercion
```javascript
"5" == 5   // true (type coercion)
"5" === 5  // false (different types)
```

---

#### Q2: What is the Temporal Dead Zone (TDZ)?
**Answer:** TDZ is the period between entering a scope and the variable declaration being processed. Accessing `let` or `const` variables in TDZ throws a ReferenceError.

```javascript
{
    // TDZ starts
    console.log(x); // ReferenceError
    let x = 5;      // TDZ ends
}
```

---

#### Q3: Explain the difference between `null` and `undefined`.
**Answer:**
- `undefined`: Variable declared but not assigned; default return value of functions
- `null`: Intentional absence of value; explicitly assigned
```javascript
let a;           // undefined
let b = null;    // null
typeof undefined // "undefined"
typeof null      // "object" (historical bug)
```

---

#### Q4: What is a closure? Why is it useful?
**Answer:** A closure is a function that remembers variables from its outer scope even after the outer function has returned.

**Uses:**
- Data privacy / encapsulation
- Factory functions
- Maintaining state in async operations
- Module pattern

---

#### Q5: What is event delegation?
**Answer:** Event delegation is a technique where you attach a single event listener to a parent element instead of multiple listeners to child elements. It uses event bubbling.

```javascript
document.getElementById('parent').addEventListener('click', (e) => {
    if (e.target.matches('.child')) {
        // Handle child click
    }
});
```
**Benefits:** Better memory usage, works with dynamically added elements.

---

#### Q6: What is the difference between `call`, `apply`, and `bind`?
**Answer:**
- `call`: Invokes function immediately with comma-separated arguments
- `apply`: Invokes function immediately with array of arguments
- `bind`: Returns a new function with bound context (doesn't invoke immediately)

```javascript
func.call(context, arg1, arg2);
func.apply(context, [arg1, arg2]);
const bound = func.bind(context, arg1);
```

---

#### Q7: What is prototypal inheritance?
**Answer:** In JavaScript, objects inherit from other objects through the prototype chain. Each object has an internal `[[Prototype]]` property pointing to its prototype object. When accessing a property, JavaScript looks up the prototype chain.

---

#### Q8: What is the difference between `function declaration` and `function expression`?
**Answer:**
```javascript
// Declaration - hoisted fully
sayHi();  // Works
function sayHi() { console.log("Hi"); }

// Expression - not hoisted
sayBye(); // Error
const sayBye = function() { console.log("Bye"); };
```

---

#### Q9: What are the differences between `var`, `let`, and `const`?
**Answer:**

| Feature | var | let | const |
|---------|-----|-----|-------|
| Scope | Function | Block | Block |
| Hoisting | Yes (undefined) | Yes (TDZ) | Yes (TDZ) |
| Redeclare | Yes | No | No |
| Reassign | Yes | Yes | No |
| Global property | Yes | No | No |

---

#### Q10: Explain the event loop in JavaScript.
**Answer:** JavaScript is single-threaded. The event loop manages async operations:

1. **Call Stack**: Executes synchronous code
2. **Web APIs**: Handle async operations (setTimeout, fetch, etc.)
3. **Callback/Task Queue**: Stores macrotask callbacks
4. **Microtask Queue**: Stores promise callbacks (higher priority)

Event loop: Executes call stack → empties microtask queue → picks one macrotask → repeat.

---

#### Q11: What is the difference between shallow copy and deep copy?
**Answer:**
- **Shallow copy**: Copies first level; nested objects are still referenced
- **Deep copy**: Copies all levels; completely independent copy

```javascript
// Shallow copy
const shallow = { ...obj };
const shallow2 = Object.assign({}, obj);

// Deep copy
const deep = JSON.parse(JSON.stringify(obj)); // Limited
const deep2 = structuredClone(obj); // Modern way
```

---

#### Q12: What is currying in JavaScript?
**Answer:** Currying transforms a function with multiple arguments into a sequence of functions each taking a single argument.

```javascript
// Normal function
function add(a, b, c) {
    return a + b + c;
}

// Curried function
function curriedAdd(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

curriedAdd(1)(2)(3); // 6
```

---

#### Q13: What is memoization?
**Answer:** Memoization is an optimization technique that caches function results to avoid redundant calculations.

```javascript
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) {
            return cache[key];
        }
        cache[key] = fn.apply(this, args);
        return cache[key];
    };
}
```

---

#### Q14: What is debouncing and throttling?
**Answer:**
- **Debouncing**: Delays execution until after a period of inactivity
- **Throttling**: Limits execution to once per time period

```javascript
// Debounce
function debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}

// Throttle
function throttle(fn, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            fn.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}
```

---

#### Q15: What are generators in JavaScript?
**Answer:** Generators are functions that can pause and resume execution using `yield`.

```javascript
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

---

### Section 3: Tricky Output Questions

#### Question 1
```javascript
console.log(typeof typeof 1);
```
**Output:** `"string"`
**Explanation:** `typeof 1` is `"number"`, `typeof "number"` is `"string"`.

---

#### Question 2
```javascript
console.log(0.1 + 0.2 === 0.3);
```
**Output:** `false`
**Explanation:** Floating-point precision issue. `0.1 + 0.2` equals `0.30000000000000004`.

---

#### Question 3
```javascript
console.log(3 > 2 > 1);
```
**Output:** `false`
**Explanation:** `3 > 2` is `true`, `true > 1` is `false` (true becomes 1, 1 > 1 is false).

---

#### Question 4
```javascript
console.log(+"");
console.log(+null);
console.log(+undefined);
console.log(+true);
```
**Output:**
```
0
0
NaN
1
```

---

#### Question 5
```javascript
let x = [1, 2, 3];
let y = x;
let z = [...x];
x.push(4);
console.log(y.length);
console.log(z.length);
```
**Output:**
```
4
3
```

---

#### Question 6
```javascript
console.log([1, 2] + [3, 4]);
```
**Output:** `"1,23,4"`
**Explanation:** Arrays are converted to strings and concatenated.

---

#### Question 7
```javascript
const a = {};
const b = { key: 'b' };
const c = { key: 'c' };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```
**Output:** `456`
**Explanation:** Objects as keys are converted to `"[object Object]"`, so both use the same key.

---

#### Question 8
```javascript
console.log([] == ![]);
```
**Output:** `true`
**Explanation:** `![]` is `false`. `[] == false` → `"" == false` → `0 == 0` → `true`.

---

#### Question 9
```javascript
function foo() {
    return
    {
        message: "Hello"
    };
}
console.log(foo());
```
**Output:** `undefined`
**Explanation:** Automatic Semicolon Insertion (ASI) adds semicolon after `return`.

---

#### Question 10
```javascript
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
    setTimeout(function() {
        console.log('Index: ' + i + ', element: ' + arr[i]);
    }, 3000);
}
```
**Output:** (after 3 seconds)
```
Index: 4, element: undefined
Index: 4, element: undefined
Index: 4, element: undefined
Index: 4, element: undefined
```
**Explanation:** `var` is function-scoped, `i` is 4 when callbacks execute.

---

## Quick Reference Cheat Sheet

```javascript
// Equality
==    // Loose equality (with coercion)
===   // Strict equality (no coercion)

// Logical operators
&&    // AND (returns first falsy or last truthy)
||    // OR (returns first truthy or last falsy)
??    // Nullish coalescing (only null/undefined)

// Type checking
typeof x           // Returns type as string
x instanceof Class // Checks prototype chain
Array.isArray(x)   // Best way to check for array

// Object methods
Object.keys(obj)    // Array of keys
Object.values(obj)  // Array of values
Object.entries(obj) // Array of [key, value] pairs
Object.assign()     // Shallow copy/merge
Object.freeze()     // Make immutable (shallow)

// Array methods
map()      // Transform elements
filter()   // Filter elements
reduce()   // Reduce to single value
find()     // Find first match
some()     // Check if any match
every()    // Check if all match
forEach()  // Iterate (no return)

// String methods
split()    // String to array
trim()     // Remove whitespace
includes() // Check if contains
startsWith() / endsWith()

// Modern syntax
?.   // Optional chaining
??   // Nullish coalescing
...  // Spread/rest operator
``   // Template literals
=>   // Arrow functions
```

---

## Common Mistakes to Avoid

1. **Using `==` instead of `===`**
2. **Forgetting that `typeof null` is `"object"`**
3. **Not understanding `this` in different contexts**
4. **Mutating objects/arrays passed as arguments**
5. **Not handling promise rejections**
6. **Using `var` in loops with async callbacks**
7. **Comparing objects by reference instead of value**
8. **Floating-point precision issues**
9. **Automatic semicolon insertion pitfalls**
10. **Not using `const` by default**

---

*Last Updated: January 2026*