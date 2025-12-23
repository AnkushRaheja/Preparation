# 📚 Part 2: TypeScript Mastery - Complete Interview Guide

> **The Ultimate TypeScript Interview Preparation Guide**
> 
> This comprehensive guide covers everything you need to master TypeScript for technical interviews at any level - from junior to senior positions at top tech companies.

---

## 📋 Table of Contents

1. [TypeScript Fundamentals](#1-typescript-fundamentals)
2. [Type System Deep Dive](#2-type-system-deep-dive)
3. [Advanced Types](#3-advanced-types)
4. [Generics Complete Guide](#4-generics-complete-guide)
5. [Utility Types](#5-utility-types)
6. [Type Guards & Narrowing](#6-type-guards--narrowing)
7. [Declaration Files & Modules](#7-declaration-files--modules)
8. [Classes & OOP in TypeScript](#8-classes--oop-in-typescript)
9. [Decorators](#9-decorators)
10. [TypeScript with React](#10-typescript-with-react)
11. [Advanced Patterns](#11-advanced-patterns)
12. [Configuration & Best Practices](#12-configuration--best-practices)
13. [Common Interview Questions](#13-common-interview-questions)
14. [Coding Challenges](#14-coding-challenges)

---

# 1. TypeScript Fundamentals

## 1.1 What is TypeScript?

TypeScript is a **statically typed superset of JavaScript** that compiles to plain JavaScript. It was developed by Microsoft and adds optional static typing, classes, and other features to JavaScript.

```typescript
// JavaScript
function add(a, b) {
  return a + b;
}

// TypeScript - with type annotations
function add(a: number, b: number): number {
  return a + b;
}

// TypeScript catches errors at compile time
add("hello", "world");  // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

### Why TypeScript?

1. **Static Type Checking**: Catch errors at compile time instead of runtime
2. **Better IDE Support**: IntelliSense, auto-completion, refactoring
3. **Self-Documenting Code**: Types serve as documentation
4. **Easier Refactoring**: Type safety makes large refactors safer
5. **Modern JavaScript Features**: Use latest JS features with backwards compatibility

### TypeScript vs JavaScript

| Feature | JavaScript | TypeScript |
|---------|-----------|------------|
| Typing | Dynamic | Static (optional) |
| Compilation | Interpreted | Compiled to JS |
| Error Detection | Runtime | Compile time |
| IDE Support | Limited | Excellent |
| Learning Curve | Lower | Higher |

## 1.2 Basic Types

### Primitive Types

```typescript
// String
let name: string = "Jayesh";
let greeting: string = `Hello, ${name}`;

// Number (includes integers, floats, hex, binary, octal)
let age: number = 25;
let price: number = 99.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// Boolean
let isActive: boolean = true;
let hasPermission: boolean = false;

// Null and Undefined
let nothing: null = null;
let notDefined: undefined = undefined;

// Symbol
let sym: symbol = Symbol("unique");

// BigInt
let big: bigint = 100n;
```

### Arrays

```typescript
// Array of numbers
let numbers: number[] = [1, 2, 3, 4, 5];

// Alternative syntax using generic
let strings: Array<string> = ["a", "b", "c"];

// Mixed array (tuple-like, but dynamic)
let mixed: (string | number)[] = [1, "two", 3];

// Readonly array
let readonlyArray: readonly number[] = [1, 2, 3];
// readonlyArray.push(4);  // Error: Property 'push' does not exist
```

### Tuples

```typescript
// Fixed-length array with specific types at each position
let tuple: [string, number] = ["Jayesh", 25];

// Accessing tuple elements
let name = tuple[0];  // string
let age = tuple[1];   // number

// Tuple with optional element
let optionalTuple: [string, number?] = ["Jayesh"];

// Tuple with rest elements
let restTuple: [string, ...number[]] = ["Jayesh", 1, 2, 3];

// Named tuple elements (for documentation)
type Person = [name: string, age: number, isActive: boolean];
const person: Person = ["Jayesh", 25, true];

// Readonly tuple
let readonlyTuple: readonly [string, number] = ["Jayesh", 25];
```

### Enums

```typescript
// Numeric enum (default starts at 0)
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

let move: Direction = Direction.Up;
console.log(Direction[0]);  // "Up" (reverse mapping)

// Numeric enum with custom values
enum Status {
  Pending = 1,
  Approved = 2,
  Rejected = 3
}

// String enum (no reverse mapping)
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

// Const enum (inlined at compile time, more efficient)
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  InternalError = 500
}

let status = HttpStatus.OK;  // Compiled to: let status = 200;

// Heterogeneous enum (mixed, not recommended)
enum Mixed {
  No = 0,
  Yes = "YES"
}
```

### Any, Unknown, Never, Void

```typescript
// any - opts out of type checking (avoid when possible)
let anything: any = 4;
anything = "string";
anything = true;
anything.foo.bar;  // No error, but might crash at runtime

// unknown - type-safe version of any
let uncertain: unknown = 4;
uncertain = "string";
// uncertain.toUpperCase();  // Error: Object is of type 'unknown'

// Type narrowing required for unknown
if (typeof uncertain === "string") {
  uncertain.toUpperCase();  // OK
}

// never - represents values that never occur
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}

// void - absence of a return value
function logMessage(message: string): void {
  console.log(message);
  // No return statement or return undefined
}
```

## 1.3 Type Annotations vs Type Inference

```typescript
// Explicit type annotation
let explicitName: string = "Jayesh";

// Type inference - TypeScript infers the type
let inferredName = "Jayesh";  // TypeScript infers: string

// When to use annotations:
// 1. Function parameters (usually required)
function greet(name: string) {
  return `Hello, ${name}`;
}

// 2. When inference isn't possible
let parsed: number;
parsed = parseInt("42");

// 3. When you want a broader type
let id: string | number = 123;  // Without annotation: number

// Type inference examples
let x = 3;                    // number
let arr = [1, 2, 3];          // number[]
let obj = { a: 1, b: "two" }; // { a: number; b: string }

// Contextual typing
window.onmousedown = function(event) {
  // event is inferred as MouseEvent
  console.log(event.button);
};
```

## 1.4 Object Types

```typescript
// Object type annotation
let person: { name: string; age: number } = {
  name: "Jayesh",
  age: 25
};

// Optional properties
let config: { host: string; port?: number } = {
  host: "localhost"
  // port is optional
};

// Readonly properties
let point: { readonly x: number; readonly y: number } = {
  x: 10,
  y: 20
};
// point.x = 30;  // Error: Cannot assign to 'x' because it is a read-only property

// Index signatures
let dictionary: { [key: string]: number } = {
  apple: 1,
  banana: 2
};
dictionary["cherry"] = 3;  // OK

// Mixing named and index signatures
interface StringArray {
  [index: number]: string;
  length: number;    // OK - length is a number
  // name: string;   // Error - must be compatible with index
}
```

## 1.5 Union and Intersection Types

```typescript
// Union type - value can be one of several types
type StringOrNumber = string | number;

let id: StringOrNumber = 123;
id = "abc";  // Also OK

function printId(id: number | string) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(2));
  }
}

// Intersection type - combines multiple types
type Person = { name: string };
type Employee = { employeeId: number };
type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: "Jayesh",
  employeeId: 123
};

// Union with literal types
type Status = "pending" | "approved" | "rejected";
let orderStatus: Status = "pending";
// orderStatus = "cancelled";  // Error

// Discriminated unions
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}
```

## 1.6 Type Aliases vs Interfaces

```typescript
// Type alias
type Point = {
  x: number;
  y: number;
};

// Interface
interface IPoint {
  x: number;
  y: number;
}

// Both can be extended
type Point3D = Point & { z: number };

interface IPoint3D extends IPoint {
  z: number;
}

// KEY DIFFERENCES:

// 1. Interfaces can be merged (declaration merging)
interface User {
  name: string;
}

interface User {
  age: number;
}

// User now has both name and age
const user: User = { name: "Jayesh", age: 25 };

// Type aliases cannot be merged
// type User = { name: string };
// type User = { age: number };  // Error: Duplicate identifier

// 2. Type aliases can represent any type
type StringOrNumber = string | number;  // Union
type Callback = (data: string) => void; // Function
type Tuple = [string, number];          // Tuple

// Interfaces are only for object shapes

// 3. Computed properties
type Keys = 'a' | 'b' | 'c';
type Mapped = { [K in Keys]: number };  // Type alias works
// interface Mapped { [K in Keys]: number }  // Error

// WHEN TO USE WHICH:
// - Use interface for public API definitions (can be extended by consumers)
// - Use type for unions, tuples, and complex type operations
// - Be consistent within your codebase
```

---

# 2. Type System Deep Dive

## 2.1 Literal Types

```typescript
// String literal types
type Direction = "north" | "south" | "east" | "west";
let dir: Direction = "north";

// Numeric literal types
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceRoll = 4;

// Boolean literal types
type True = true;
type False = false;

// Template literal types (TypeScript 4.1+)
type Greeting = `Hello, ${string}`;
let greeting: Greeting = "Hello, Jayesh";

type EventName = `on${Capitalize<"click" | "focus" | "blur">}`;
// "onClick" | "onFocus" | "onBlur"

// Const assertions
const config = {
  host: "localhost",
  port: 8080
} as const;
// Type: { readonly host: "localhost"; readonly port: 8080 }

// Without 'as const':
const config2 = { host: "localhost", port: 8080 };
// Type: { host: string; port: number }
```

## 2.2 Type Assertions

```typescript
// Type assertions tell TypeScript "trust me, I know the type"

// Angle bracket syntax (not available in JSX)
let value: unknown = "hello";
let length: number = (<string>value).length;

// 'as' syntax (preferred, works with JSX)
let length2: number = (value as string).length;

// Non-null assertion operator
function getElement(id: string) {
  const element = document.getElementById(id);
  return element!.innerHTML;  // Assert element is not null
}

// Const assertion
let arr = [1, 2, 3] as const;  // readonly [1, 2, 3]

// Double assertion (use sparingly)
let x = "hello" as unknown as number;

// When to use type assertions:
// 1. Working with DOM elements
const canvas = document.getElementById("canvas") as HTMLCanvasElement;

// 2. When you have more information than TypeScript
interface ApiResponse {
  data: unknown;
}
const response: ApiResponse = { data: { name: "Jayesh" } };
const name = (response.data as { name: string }).name;

// WARNING: Type assertions are not type conversions!
const num = "123" as unknown as number;
console.log(num + 1);  // "1231" not 124!
```

## 2.3 Type Widening and Narrowing

```typescript
// Type Widening - TypeScript infers a broader type

let x = "hello";  // Type: string (widened from "hello")
const y = "hello";  // Type: "hello" (const prevents widening)

let obj = { name: "Jayesh" };  // { name: string }
const obj2 = { name: "Jayesh" } as const;  // { readonly name: "Jayesh" }

// Type Narrowing - TypeScript infers a more specific type

function process(value: string | number) {
  // typeof guard
  if (typeof value === "string") {
    // value is string here
    return value.toUpperCase();
  }
  // value is number here
  return value.toFixed(2);
}

// Truthiness narrowing
function printName(name: string | null) {
  if (name) {
    // name is string here
    console.log(name.toUpperCase());
  }
}

// Equality narrowing
function compare(x: string | number, y: string | boolean) {
  if (x === y) {
    // Both x and y are string here
    console.log(x.toUpperCase());
  }
}

// instanceof narrowing
function processDate(value: Date | string) {
  if (value instanceof Date) {
    return value.getFullYear();
  }
  return new Date(value).getFullYear();
}

// 'in' operator narrowing
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}
```

## 2.4 Control Flow Analysis

```typescript
// TypeScript tracks type changes through control flow

function example(x: string | number | boolean) {
  if (typeof x === "string") {
    // x is string
    return x.slice(0, 3);
  }
  // x is number | boolean
  
  if (typeof x === "number") {
    // x is number
    return x.toFixed(2);
  }
  // x is boolean
  
  return x ? "yes" : "no";
}

// Assignments affect types
let value: string | number;

value = "hello";
value.toUpperCase();  // OK, value is string

value = 42;
value.toFixed(2);     // OK, value is number

// Type predicates for custom type guards
interface Cat { meow(): void; }
interface Dog { bark(): void; }

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow();  // animal is Cat
  } else {
    animal.bark();  // animal is Dog
  }
}

// Assertion functions
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Not a string!");
  }
}

function process(value: unknown) {
  assertIsString(value);
  // value is string from this point on
  console.log(value.toUpperCase());
}
```

---

# 3. Advanced Types

## 3.1 Conditional Types

```typescript
// Basic conditional type: T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Conditional types with infer
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { name: "Jayesh", age: 25 };
}

type User = ReturnType<typeof getUser>;  // { name: string; age: number }

// Extract element type from array
type ElementType<T> = T extends (infer E)[] ? E : T;

type StringElement = ElementType<string[]>;  // string
type NumberElement = ElementType<number>;    // number

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;

type StrOrNumArray = ToArray<string | number>;  // string[] | number[]

// Prevent distribution with tuple
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type StrOrNumArraySingle = ToArrayNonDist<string | number>;  // (string | number)[]

// Real-world example: Flatten type
type Flatten<T> = T extends any[] ? T[number] : T;

type Flat = Flatten<number[]>;   // number
type NoFlat = Flatten<number>;   // number
```

## 3.2 Mapped Types

```typescript
// Create new type by transforming each property of an existing type

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];  // -? removes optional modifier
};

interface User {
  name: string;
  age: number;
  email?: string;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;

// Mapped type with key remapping (TypeScript 4.1+)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number; getEmail?: () => string | undefined }

// Filter keys
type FilterByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

type StringProps = FilterByType<User, string>;  // { name: string; email?: string }

// Modify property types
type Stringify<T> = {
  [K in keyof T]: string;
};

type StringUser = Stringify<User>;  // { name: string; age: string; email: string }
```

## 3.3 Template Literal Types

```typescript
// Combine literal types with template strings

type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// With unions - creates all combinations
type Color = "red" | "blue";
type Size = "small" | "large";
type ColorSize = `${Color}-${Size}`;
// "red-small" | "red-large" | "blue-small" | "blue-large"

// Intrinsic string manipulation types
type Upper = Uppercase<"hello">;       // "HELLO"
type Lower = Lowercase<"HELLO">;       // "hello"
type Cap = Capitalize<"hello">;        // "Hello"
type Uncap = Uncapitalize<"Hello">;    // "hello"

// Event handler pattern
type EventNames = "click" | "focus" | "blur";
type EventHandlers = `on${Capitalize<EventNames>}`;
// "onClick" | "onFocus" | "onBlur"

// CSS property pattern
type CSSValue = `${number}${"px" | "em" | "rem" | "%"}`;
let width: CSSValue = "100px";
let height: CSSValue = "50%";
// let invalid: CSSValue = "10vw";  // Error

// Extract parts with infer
type ExtractRoute<T> = T extends `/${infer Route}` ? Route : never;

type UserRoute = ExtractRoute<"/users">;      // "users"
type ProductRoute = ExtractRoute<"/products">; // "products"
type Invalid = ExtractRoute<"no-slash">;       // never
```

## 3.4 Index Types and keyof

```typescript
// keyof - gets union of keys from a type
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonKeys = keyof Person;  // "name" | "age" | "email"

// Indexed access types
type NameType = Person["name"];          // string
type NameOrAge = Person["name" | "age"]; // string | number
type AllValues = Person[keyof Person];   // string | number

// Using with generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person: Person = { name: "Jayesh", age: 25, email: "j@j.com" };
const name = getProperty(person, "name");  // string
const age = getProperty(person, "age");    // number
// getProperty(person, "invalid");  // Error

// Array index access
const arr = [1, 2, 3] as const;
type First = (typeof arr)[0];      // 1
type Elements = (typeof arr)[number]; // 1 | 2 | 3

// Object with string index
const obj = { a: 1, b: 2, c: 3 };
type ObjValues = (typeof obj)[keyof typeof obj];  // number
```

## 3.5 The infer Keyword

```typescript
// infer declares a type variable within conditional type

// Infer return type
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FnReturn = GetReturnType<() => string>;  // string

// Infer parameter types
type GetParameters<T> = T extends (...args: infer P) => any ? P : never;

type FnParams = GetParameters<(a: string, b: number) => void>;  // [string, number]

// Infer first parameter
type FirstParam<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

type First = FirstParam<(a: string, b: number) => void>;  // string

// Infer array element type
type ArrayElement<T> = T extends (infer E)[] ? E : never;

type Elem = ArrayElement<number[]>;  // number

// Infer promise resolve type
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type Resolved = UnwrapPromise<Promise<string>>;  // string
type NotPromise = UnwrapPromise<number>;         // number

// Multiple inferences
type ParseRoute<T> = T extends `${infer Start}/${infer End}`
  ? { start: Start; end: End }
  : never;

type Route = ParseRoute<"users/123">;  // { start: "users"; end: "123" }

// Nested inferences
type DeepUnwrap<T> = T extends Promise<infer U> 
  ? DeepUnwrap<U> 
  : T;

type Deep = DeepUnwrap<Promise<Promise<Promise<string>>>>;  // string
```

---

# 4. Generics Complete Guide

## 4.1 Generic Functions

```typescript
// Basic generic function
function identity<T>(arg: T): T {
  return arg;
}

let str = identity<string>("hello");  // Explicit type argument
let num = identity(42);               // Type inference

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p = pair("hello", 42);  // [string, number]

// Generic constraints
function getLength<T extends { length: number }>(arg: T): number {
  return arg.length;
}

getLength("hello");     // OK
getLength([1, 2, 3]);   // OK
getLength({ length: 5 }); // OK
// getLength(42);        // Error: number doesn't have length

// Using keyof in constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Default type parameters
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

const strArr = createArray(3, "x");      // string[]
const numArr = createArray<number>(3, 0); // number[]
```

## 4.2 Generic Interfaces and Types

```typescript
// Generic interface
interface Container<T> {
  value: T;
  getValue(): T;
}

const stringContainer: Container<string> = {
  value: "hello",
  getValue() { return this.value; }
};

// Generic type alias
type Result<T> = { success: true; data: T } | { success: false; error: string };

function fetchData<T>(url: string): Result<T> {
  // Implementation
  return { success: true, data: {} as T };
}

// Generic with multiple constraints
interface Printable {
  print(): void;
}

interface Loggable {
  log(): void;
}

function process<T extends Printable & Loggable>(item: T) {
  item.print();
  item.log();
}

// Generic interface extending another
interface Repository<T> {
  find(id: string): T | null;
  save(item: T): void;
}

interface UserRepository extends Repository<User> {
  findByEmail(email: string): User | null;
}
```

## 4.3 Generic Classes

```typescript
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop());  // 2

// Generic class with constraints
class KeyValuePair<K extends string | number, V> {
  constructor(public key: K, public value: V) {}
  
  toString(): string {
    return `${this.key}: ${this.value}`;
  }
}

const pair = new KeyValuePair("name", "Jayesh");

// Static members cannot use class type parameters
class Factory<T> {
  // static instance: T;  // Error!
  
  static create<U>(value: U): Factory<U> {
    return new Factory();
  }
}
```

## 4.4 Generic Utility Patterns

```typescript
// Generic factory function
function createInstance<T>(Constructor: new () => T): T {
  return new Constructor();
}

class MyClass {
  name = "MyClass";
}

const instance = createInstance(MyClass);  // MyClass

// Constructor with parameters
function createWithArgs<T, A extends any[]>(
  Constructor: new (...args: A) => T,
  ...args: A
): T {
  return new Constructor(...args);
}

class Person {
  constructor(public name: string, public age: number) {}
}

const person = createWithArgs(Person, "Jayesh", 25);

// Generic function composition
function compose<A, B, C>(
  f: (a: A) => B,
  g: (b: B) => C
): (a: A) => C {
  return (a) => g(f(a));
}

const addOne = (x: number) => x + 1;
const double = (x: number) => x * 2;
const addOneThenDouble = compose(addOne, double);

console.log(addOneThenDouble(5));  // 12

// Generic curry function
function curry<A, B, C>(fn: (a: A, b: B) => C): (a: A) => (b: B) => C {
  return (a) => (b) => fn(a, b);
}

const add = (a: number, b: number) => a + b;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2));  // 3
```

---

# 5. Utility Types

TypeScript provides several built-in utility types for common type transformations.

## 5.1 Partial, Required, Readonly

```typescript
interface User {
  name: string;
  age: number;
  email?: string;
}

// Partial<T> - Makes all properties optional
type PartialUser = Partial<User>;
// { name?: string; age?: number; email?: string }

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

// Required<T> - Makes all properties required
type RequiredUser = Required<User>;
// { name: string; age: number; email: string }

// Readonly<T> - Makes all properties readonly
type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { name: "Jayesh", age: 25 };
// user.name = "John";  // Error: Cannot assign to 'name'

// DeepReadonly (custom)
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};
```

## 5.2 Pick, Omit, Record

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Pick<T, K> - Select specific properties
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit<T, K> - Exclude specific properties
type PublicUser = Omit<User, "password">;
// { id: number; name: string; email: string; createdAt: Date }

// Record<K, T> - Create object type with keys K and values T
type UserRoles = Record<string, string[]>;
const roles: UserRoles = {
  admin: ["read", "write", "delete"],
  user: ["read"]
};

type StatusMessages = Record<"success" | "error" | "loading", string>;
const messages: StatusMessages = {
  success: "Operation completed",
  error: "Something went wrong",
  loading: "Please wait..."
};

// Combine utilities
type CreateUserDTO = Omit<User, "id" | "createdAt">;
type UpdateUserDTO = Partial<Omit<User, "id">>;
```

## 5.3 Exclude, Extract, NonNullable

```typescript
type AllTypes = string | number | boolean | null | undefined;

// Exclude<T, U> - Exclude types from T that are assignable to U
type WithoutNull = Exclude<AllTypes, null | undefined>;
// string | number | boolean

// Extract<T, U> - Extract types from T that are assignable to U
type OnlyPrimitives = Extract<AllTypes, string | number>;
// string | number

// NonNullable<T> - Remove null and undefined
type NonNull = NonNullable<AllTypes>;
// string | number | boolean

// Practical example
type EventType = "click" | "focus" | "blur" | "scroll" | "resize";
type MouseEvents = Extract<EventType, "click" | "scroll">;
// "click" | "scroll"

type NonMouseEvents = Exclude<EventType, "click" | "scroll">;
// "focus" | "blur" | "resize"
```

## 5.4 ReturnType, Parameters, ConstructorParameters

```typescript
function createUser(name: string, age: number) {
  return { id: Math.random(), name, age };
}

// ReturnType<T> - Get return type of a function
type User = ReturnType<typeof createUser>;
// { id: number; name: string; age: number }

// Parameters<T> - Get parameter types as tuple
type CreateUserParams = Parameters<typeof createUser>;
// [string, number]

// ConstructorParameters<T> - Get constructor parameter types
class Person {
  constructor(public name: string, public age: number) {}
}

type PersonParams = ConstructorParameters<typeof Person>;
// [string, number]

// InstanceType<T> - Get instance type of a class
type PersonInstance = InstanceType<typeof Person>;
// Person

// Practical usage
function wrapFunction<T extends (...args: any[]) => any>(
  fn: T
): (...args: Parameters<T>) => Promise<ReturnType<T>> {
  return async (...args) => fn(...args);
}
```

## 5.5 Awaited and Other Utility Types

```typescript
// Awaited<T> - Unwrap Promise types (TypeScript 4.5+)
type PromiseString = Promise<string>;
type Resolved = Awaited<PromiseString>;  // string

type NestedPromise = Promise<Promise<number>>;
type DeepResolved = Awaited<NestedPromise>;  // number

// ThisParameterType<T> - Extract 'this' parameter type
function toHex(this: Number) {
  return this.toString(16);
}

type ThisType = ThisParameterType<typeof toHex>;  // Number

// OmitThisParameter<T> - Remove 'this' parameter
type ToHexType = OmitThisParameter<typeof toHex>;  // () => string

// ThisType<T> - Marker for contextual 'this'
interface HelperMethods {
  log(): void;
  save(): void;
}

const helpers: HelperMethods & ThisType<{ name: string }> = {
  log() {
    console.log(this.name);  // 'this' is { name: string }
  },
  save() {
    console.log(`Saving ${this.name}`);
  }
};
```

---

# 6. Type Guards & Narrowing

## 6.1 typeof Type Guards

```typescript
function processInput(input: string | number | boolean) {
  if (typeof input === "string") {
    // input is string
    return input.toUpperCase();
  }
  if (typeof input === "number") {
    // input is number
    return input.toFixed(2);
  }
  // input is boolean
  return input ? "yes" : "no";
}

// typeof returns: "string" | "number" | "bigint" | "boolean" | 
//                 "symbol" | "undefined" | "object" | "function"

// Note: typeof null === "object" (historical bug)
function processValue(value: string | null) {
  if (typeof value === "string") {
    return value.length;
  }
  // value is null here, not undefined
  return 0;
}
```

## 6.2 instanceof Type Guards

```typescript
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// With inheritance
class Animal {
  name: string = "";
}

class Bird extends Animal {
  fly() { console.log("Flying!"); }
}

class Fish extends Animal {
  swim() { console.log("Swimming!"); }
}

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

## 6.3 in Operator Type Guard

```typescript
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}

// With optional properties
interface Admin {
  name: string;
  privileges: string[];
}

interface Employee {
  name: string;
  startDate: Date;
}

function printInfo(person: Admin | Employee) {
  console.log(person.name);
  
  if ("privileges" in person) {
    console.log(person.privileges);
  }
  
  if ("startDate" in person) {
    console.log(person.startDate);
  }
}
```

## 6.4 Custom Type Guards (Type Predicates)

```typescript
// Type predicate: parameterName is Type
interface Cat {
  type: "cat";
  meow(): void;
}

interface Dog {
  type: "dog";
  bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return animal.type === "cat";
}

function isDog(animal: Cat | Dog): animal is Dog {
  return animal.type === "dog";
}

function makeNoise(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow();  // animal is Cat
  } else {
    animal.bark();  // animal is Dog
  }
}

// Object type guard
function isObject(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null;
}

// Array type guard
function isArray<T>(value: unknown): value is T[] {
  return Array.isArray(value);
}

// Combined type guard
function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every(item => typeof item === "string");
}
```

## 6.5 Discriminated Unions

```typescript
// Use a common property to discriminate between types
interface Circle {
  kind: "circle";
  radius: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
  }
}

// Exhaustiveness checking with never
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

function getAreaSafe(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      return assertNever(shape);  // Error if not exhaustive
  }
}
```

## 6.6 Assertion Functions

```typescript
// Assertion function: asserts condition
function assert(condition: unknown, message: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

function processValue(value: string | null) {
  assert(value !== null, "Value cannot be null");
  // value is string from here
  console.log(value.toUpperCase());
}

// Assertion function with type predicate
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Value is not a string");
  }
}

function process(value: unknown) {
  assertIsString(value);
  // value is string from here
  console.log(value.length);
}

function assertIsUser(value: unknown): asserts value is User {
  if (
    typeof value !== "object" ||
    value === null ||
    !("name" in value) ||
    !("age" in value)
  ) {
    throw new Error("Value is not a User");
  }
}
```

---

# 7. Declaration Files & Modules

## 7.1 Module Systems

```typescript
// ES Modules (Recommended)
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;

// Default export
export default class Calculator {
  add(a: number, b: number) { return a + b; }
}

// Importing
import Calculator, { add, subtract, PI } from './math';
import * as math from './math';

// Re-exporting
export { add, subtract } from './math';
export * from './math';
export * as math from './math';

// CommonJS (Node.js)
// Using require with TypeScript
const fs = require('fs');  // Type: any

// Better: use import with type definitions
import * as fs from 'fs';  // Fully typed
import { readFile } from 'fs';

// Export for CommonJS
module.exports = { add, subtract };
exports.add = add;
```

## 7.2 Declaration Files (.d.ts)

```typescript
// Declaration files provide type information for JavaScript libraries

// types.d.ts
declare module 'my-library' {
  export function doSomething(x: string): number;
  export class MyClass {
    constructor(value: number);
    getValue(): number;
  }
  export interface Config {
    debug: boolean;
    timeout: number;
  }
}

// Global declarations
declare global {
  interface Window {
    myGlobalFunction: (x: string) => void;
  }
  
  var myGlobalVar: string;
}

// Ambient declarations for existing JavaScript
declare function $(selector: string): JQueryElement;
declare class JQueryElement {
  html(content: string): this;
  css(property: string, value: string): this;
}

// Module augmentation
import { AxiosRequestConfig } from 'axios';

declare module 'axios' {
  interface AxiosRequestConfig {
    customField?: string;
  }
}
```

## 7.3 Triple-Slash Directives

```typescript
/// <reference path="./types.d.ts" />
/// <reference types="node" />
/// <reference lib="dom" />

// These are mostly used in declaration files
// Modern projects use tsconfig.json instead

// In tsconfig.json:
{
  "compilerOptions": {
    "types": ["node", "jest"],  // Include specific @types packages
    "typeRoots": ["./types", "./node_modules/@types"]
  }
}
```

## 7.4 Namespace vs Modules

```typescript
// Namespaces (legacy, avoid in new code)
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
  
  export class LettersOnlyValidator implements StringValidator {
    isValid(s: string) {
      return /^[A-Za-z]+$/.test(s);
    }
  }
}

const validator = new Validation.LettersOnlyValidator();

// Prefer ES Modules instead:
// validators.ts
export interface StringValidator {
  isValid(s: string): boolean;
}

export class LettersOnlyValidator implements StringValidator {
  isValid(s: string) {
    return /^[A-Za-z]+$/.test(s);
  }
}

// Usage
import { StringValidator, LettersOnlyValidator } from './validators';
```

---

# 8. Classes & OOP in TypeScript

## 8.1 Class Basics

```typescript
class Person {
  // Properties
  name: string;
  age: number;
  
  // Constructor
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  
  // Methods
  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
  
  // Static property
  static species = "Homo sapiens";
  
  // Static method
  static create(name: string): Person {
    return new Person(name, 0);
  }
}

const person = new Person("Jayesh", 25);
console.log(Person.species);  // "Homo sapiens"
```

## 8.2 Access Modifiers

```typescript
class Employee {
  public name: string;        // Accessible anywhere (default)
  private salary: number;     // Only accessible within class
  protected department: string; // Accessible in class and subclasses
  readonly id: number;        // Cannot be modified after initialization
  
  constructor(name: string, salary: number, department: string, id: number) {
    this.name = name;
    this.salary = salary;
    this.department = department;
    this.id = id;
  }
  
  // Private method
  private calculateBonus(): number {
    return this.salary * 0.1;
  }
  
  public getAnnualCompensation(): number {
    return this.salary * 12 + this.calculateBonus();
  }
}

// Parameter properties - shorthand
class Employee2 {
  constructor(
    public name: string,
    private salary: number,
    protected department: string,
    readonly id: number
  ) {}
}
```

## 8.3 Inheritance

```typescript
class Animal {
  constructor(public name: string) {}
  
  move(distance: number = 0) {
    console.log(`${this.name} moved ${distance}m`);
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);  // Must call super() before using 'this'
  }
  
  // Override method
  move(distance: number = 5) {
    console.log("Running...");
    super.move(distance);  // Call parent method
  }
  
  bark() {
    console.log("Woof!");
  }
}

const dog = new Dog("Max", "Labrador");
dog.move(10);  // "Running..." then "Max moved 10m"
```

## 8.4 Abstract Classes

```typescript
abstract class Shape {
  abstract getArea(): number;     // Must be implemented by subclasses
  abstract getPerimeter(): number;
  
  // Concrete method - shared by all subclasses
  describe(): string {
    return `Area: ${this.getArea()}, Perimeter: ${this.getPerimeter()}`;
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }
  
  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
  
  getPerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(public width: number, public height: number) {
    super();
  }
  
  getArea(): number {
    return this.width * this.height;
  }
  
  getPerimeter(): number {
    return 2 * (this.width + this.height);
  }
}

// const shape = new Shape();  // Error: Cannot create instance of abstract class
```

## 8.5 Interfaces with Classes

```typescript
interface Printable {
  print(): void;
}

interface Serializable {
  serialize(): string;
}

// Implement multiple interfaces
class Document implements Printable, Serializable {
  constructor(public content: string) {}
  
  print(): void {
    console.log(this.content);
  }
  
  serialize(): string {
    return JSON.stringify({ content: this.content });
  }
}

// Interface extending interface
interface Shape {
  getArea(): number;
}

interface ColoredShape extends Shape {
  color: string;
}

class ColoredCircle implements ColoredShape {
  constructor(
    public radius: number,
    public color: string
  ) {}
  
  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

## 8.6 Getters and Setters

```typescript
class Person {
  private _age: number = 0;
  private _fullName: string = "";
  
  get age(): number {
    return this._age;
  }
  
  set age(value: number) {
    if (value < 0 || value > 150) {
      throw new Error("Invalid age");
    }
    this._age = value;
  }
  
  get fullName(): string {
    return this._fullName;
  }
  
  set fullName(value: string) {
    if (!value || value.length < 2) {
      throw new Error("Invalid name");
    }
    this._fullName = value;
  }
}

const person = new Person();
person.age = 25;       // Uses setter
console.log(person.age); // Uses getter
// person.age = -5;    // Error: Invalid age
```

## 8.7 Generic Classes

```typescript
class Queue<T> {
  private items: T[] = [];
  
  enqueue(item: T): void {
    this.items.push(item);
  }
  
  dequeue(): T | undefined {
    return this.items.shift();
  }
  
  peek(): T | undefined {
    return this.items[0];
  }
  
  get length(): number {
    return this.items.length;
  }
}

const numberQueue = new Queue<number>();
numberQueue.enqueue(1);
numberQueue.enqueue(2);
console.log(numberQueue.dequeue());  // 1

// Generic class with constraints
class Repository<T extends { id: string }> {
  private items: Map<string, T> = new Map();
  
  add(item: T): void {
    this.items.set(item.id, item);
  }
  
  get(id: string): T | undefined {
    return this.items.get(id);
  }
  
  getAll(): T[] {
    return Array.from(this.items.values());
  }
}
```

---

# 9. Decorators

> Note: Decorators require `"experimentalDecorators": true` in tsconfig.json

## 9.1 Class Decorators

```typescript
// Simple class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return `Hello, ${this.greeting}`;
  }
}

// Decorator factory (returns decorator)
function logger(prefix: string) {
  return function(constructor: Function) {
    console.log(`${prefix}: ${constructor.name} was created`);
  };
}

@logger("DEBUG")
class MyClass {
  constructor() {
    console.log("MyClass instance created");
  }
}

// Decorator that modifies the class
function withTimestamp<T extends { new(...args: any[]): {} }>(Constructor: T) {
  return class extends Constructor {
    timestamp = new Date();
  };
}

@withTimestamp
class User {
  constructor(public name: string) {}
}

const user = new User("Jayesh");
console.log((user as any).timestamp);  // Current date
```

## 9.2 Method Decorators

```typescript
// Log method calls
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);  // Logs: Calling add with args: [2, 3] → add returned: 5

// Timing decorator
function timing(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = async function(...args: any[]) {
    const start = performance.now();
    const result = await originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };
  
  return descriptor;
}

// Memoization decorator
function memoize(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}
```

## 9.3 Property Decorators

```typescript
function required(target: any, propertyKey: string) {
  let value: any;
  
  const getter = () => value;
  const setter = (newValue: any) => {
    if (newValue === undefined || newValue === null || newValue === "") {
      throw new Error(`${propertyKey} is required`);
    }
    value = newValue;
  };
  
  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true
  });
}

class User {
  @required
  name: string = "";
  
  @required
  email: string = "";
}

// Validation decorator factory
function validate(validator: (value: any) => boolean, message: string) {
  return function(target: any, propertyKey: string) {
    let value: any;
    
    Object.defineProperty(target, propertyKey, {
      get: () => value,
      set: (newValue) => {
        if (!validator(newValue)) {
          throw new Error(`${propertyKey}: ${message}`);
        }
        value = newValue;
      }
    });
  };
}

class Person {
  @validate((v) => v.length >= 2, "Name must be at least 2 characters")
  name: string = "";
  
  @validate((v) => v >= 0 && v <= 150, "Age must be between 0 and 150")
  age: number = 0;
}
```

## 9.4 Parameter Decorators

```typescript
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  const existingRequiredParams: number[] = 
    Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
  existingRequiredParams.push(parameterIndex);
  Reflect.defineMetadata(requiredMetadataKey, existingRequiredParams, target, propertyKey);
}

function validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const requiredParams: number[] = 
      Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    
    for (const index of requiredParams) {
      if (args[index] === undefined) {
        throw new Error(`Missing required argument at position ${index}`);
      }
    }
    
    return method.apply(this, args);
  };
}

class UserService {
  @validate
  createUser(@required name: string, @required email: string, age?: number) {
    return { name, email, age };
  }
}
```

---

# 10. TypeScript with React

## 10.1 Function Components

```tsx
import React from 'react';

// Basic component with props
interface GreetingProps {
  name: string;
  age?: number;  // Optional prop
}

const Greeting: React.FC<GreetingProps> = ({ name, age }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>You are {age} years old</p>}
    </div>
  );
};

// Alternative syntax (preferred, more explicit)
function Greeting2({ name, age }: GreetingProps): JSX.Element {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>You are {age} years old</p>}
    </div>
  );
}

// Props with children
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="content">{children}</div>
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>Card content here</p>
</Card>
```

## 10.2 useState Hook

```tsx
import { useState } from 'react';

function Counter() {
  // Type is inferred from initial value
  const [count, setCount] = useState(0);
  
  // Explicit type when initial value doesn't match full type
  const [user, setUser] = useState<User | null>(null);
  
  // Type with union
  const [status, setStatus] = useState<'idle' | 'loading' | 'error'>('idle');
  
  // Complex state
  interface FormState {
    name: string;
    email: string;
    age: number;
  }
  
  const [form, setForm] = useState<FormState>({
    name: '',
    email: '',
    age: 0
  });
  
  // Update handlers
  const updateName = (name: string) => {
    setForm(prev => ({ ...prev, name }));
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

## 10.3 useEffect Hook

```tsx
import { useEffect, useState } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data: User = await response.json();
        
        if (!cancelled) {
          setUser(data);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err : new Error('Unknown error'));
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true;
    };
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>{user.name}</div>;
}
```

## 10.4 useRef Hook

```tsx
import { useRef, useEffect } from 'react';

function TextInput() {
  // DOM element ref
  const inputRef = useRef<HTMLInputElement>(null);
  
  // Mutable value ref (doesn't cause re-render)
  const renderCount = useRef(0);
  
  // Previous value ref
  const prevValueRef = useRef<string>('');
  
  useEffect(() => {
    // Focus input on mount
    inputRef.current?.focus();
  }, []);
  
  useEffect(() => {
    renderCount.current += 1;
    console.log(`Rendered ${renderCount.current} times`);
  });
  
  return (
    <input 
      ref={inputRef}
      type="text"
      placeholder="Type here..."
    />
  );
}

// Forwarding refs
interface InputProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label, value, onChange }, ref) => {
    return (
      <label>
        {label}
        <input
          ref={ref}
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </label>
    );
  }
);
```

## 10.5 useReducer Hook

```tsx
import { useReducer } from 'react';

// State type
interface CounterState {
  count: number;
  step: number;
}

// Action types using discriminated union
type CounterAction =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_STEP'; payload: number }
  | { type: 'RESET' };

// Reducer function
function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'RESET':
      return { count: 0, step: 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0, step: 1 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <p>Step: {state.step}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'SET_STEP', payload: 5 })}>
        Step = 5
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

## 10.6 useContext Hook

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Theme context
interface Theme {
  primary: string;
  secondary: string;
  background: string;
}

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  toggleDarkMode: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Custom hook for consuming context
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Provider component
function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>({
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff'
  });
  
  const toggleDarkMode = () => {
    setTheme(prev => ({
      ...prev,
      background: prev.background === '#ffffff' ? '#1a1a1a' : '#ffffff'
    }));
  };
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleDarkMode }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage
function ThemedButton() {
  const { theme, toggleDarkMode } = useTheme();
  
  return (
    <button
      style={{ backgroundColor: theme.primary }}
      onClick={toggleDarkMode}
    >
      Toggle Dark Mode
    </button>
  );
}
```

## 10.7 Custom Hooks

```tsx
import { useState, useEffect, useCallback } from 'react';

// useFetch hook
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      if (!response.ok) throw new Error('Network response was not ok');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// useLocalStorage hook
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error setting localStorage:', error);
    }
  };
  
  return [storedValue, setValue];
}

// useDebounce hook
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}
```

## 10.8 Event Handlers

```tsx
import { ChangeEvent, FormEvent, MouseEvent, KeyboardEvent } from 'react';

function Form() {
  // Input change handler
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    console.log(name, value);
  };
  
  // Select change handler
  const handleSelectChange = (e: ChangeEvent<HTMLSelectElement>) => {
    console.log(e.target.value);
  };
  
  // Form submit handler
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Handle form submission
  };
  
  // Button click handler
  const handleClick = (e: MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    console.log('Button clicked at:', e.clientX, e.clientY);
  };
  
  // Keyboard event handler
  const handleKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text"
        name="username"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
      />
      <select onChange={handleSelectChange}>
        <option value="a">A</option>
        <option value="b">B</option>
      </select>
      <button type="submit" onClick={handleClick}>Submit</button>
    </form>
  );
}
```

---

# 11. Advanced Patterns

## 11.1 Brand Types

```typescript
// Create distinct types for same underlying type
type USD = number & { readonly brand: unique symbol };
type EUR = number & { readonly brand: unique symbol };

function usd(amount: number): USD {
  return amount as USD;
}

function eur(amount: number): EUR {
  return amount as EUR;
}

function addUSD(a: USD, b: USD): USD {
  return (a + b) as USD;
}

const dollars = usd(100);
const euros = eur(100);

addUSD(dollars, dollars);  // OK
// addUSD(dollars, euros); // Error: EUR is not assignable to USD

// Branded string types
type UserId = string & { readonly __brand: 'UserId' };
type ProductId = string & { readonly __brand: 'ProductId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId) {
  // Only accepts UserId, not any string
}
```

## 11.2 Builder Pattern

```typescript
class QueryBuilder<T extends object> {
  private query: Partial<T> = {};
  
  where<K extends keyof T>(key: K, value: T[K]): this {
    this.query[key] = value;
    return this;
  }
  
  build(): Partial<T> {
    return { ...this.query };
  }
}

interface User {
  name: string;
  age: number;
  email: string;
}

const query = new QueryBuilder<User>()
  .where('name', 'Jayesh')
  .where('age', 25)
  .build();

// Fluent interface pattern
class RequestBuilder {
  private config: RequestInit = {};
  private url: string = '';
  
  setUrl(url: string): this {
    this.url = url;
    return this;
  }
  
  setMethod(method: string): this {
    this.config.method = method;
    return this;
  }
  
  setHeaders(headers: HeadersInit): this {
    this.config.headers = headers;
    return this;
  }
  
  setBody(body: BodyInit): this {
    this.config.body = body;
    return this;
  }
  
  async execute<T>(): Promise<T> {
    const response = await fetch(this.url, this.config);
    return response.json();
  }
}

// Usage
const result = await new RequestBuilder()
  .setUrl('/api/users')
  .setMethod('POST')
  .setHeaders({ 'Content-Type': 'application/json' })
  .setBody(JSON.stringify({ name: 'Jayesh' }))
  .execute<User>();
```

## 11.3 Factory Pattern with Generics

```typescript
interface Shape {
  draw(): void;
  getArea(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  
  draw(): void {
    console.log(`Drawing circle with radius ${this.radius}`);
  }
  
  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  
  draw(): void {
    console.log(`Drawing rectangle ${this.width}x${this.height}`);
  }
  
  getArea(): number {
    return this.width * this.height;
  }
}

// Factory function
type ShapeType = 'circle' | 'rectangle';

type ShapeParams = {
  circle: [radius: number];
  rectangle: [width: number, height: number];
};

function createShape<T extends ShapeType>(
  type: T,
  ...args: ShapeParams[T]
): Shape {
  switch (type) {
    case 'circle':
      return new Circle(args[0] as number);
    case 'rectangle':
      return new Rectangle(args[0] as number, args[1] as number);
    default:
      throw new Error(`Unknown shape type: ${type}`);
  }
}

const circle = createShape('circle', 5);
const rectangle = createShape('rectangle', 10, 20);
```

## 11.4 Type-Safe Event Emitter

```typescript
type EventMap = Record<string, any>;

type EventKey<T extends EventMap> = string & keyof T;
type EventCallback<T> = (payload: T) => void;

class TypedEventEmitter<T extends EventMap> {
  private listeners: {
    [K in keyof T]?: Set<EventCallback<T[K]>>;
  } = {};
  
  on<K extends EventKey<T>>(eventName: K, callback: EventCallback<T[K]>): () => void {
    if (!this.listeners[eventName]) {
      this.listeners[eventName] = new Set();
    }
    this.listeners[eventName]!.add(callback);
    
    return () => this.off(eventName, callback);
  }
  
  off<K extends EventKey<T>>(eventName: K, callback: EventCallback<T[K]>): void {
    this.listeners[eventName]?.delete(callback);
  }
  
  emit<K extends EventKey<T>>(eventName: K, payload: T[K]): void {
    this.listeners[eventName]?.forEach(callback => callback(payload));
  }
}

// Usage
interface AppEvents {
  userLogin: { userId: string; timestamp: Date };
  userLogout: { userId: string };
  error: { message: string; code: number };
}

const emitter = new TypedEventEmitter<AppEvents>();

emitter.on('userLogin', ({ userId, timestamp }) => {
  console.log(`User ${userId} logged in at ${timestamp}`);
});

emitter.emit('userLogin', { userId: '123', timestamp: new Date() });
// emitter.emit('userLogin', { userId: '123' });  // Error: missing timestamp
```

## 11.5 State Machine

```typescript
type State = 'idle' | 'loading' | 'success' | 'error';

type StateTransitions = {
  idle: 'loading';
  loading: 'success' | 'error';
  success: 'idle';
  error: 'idle' | 'loading';
};

type StateData = {
  idle: null;
  loading: { startTime: Date };
  success: { data: unknown };
  error: { message: string };
};

class StateMachine {
  private _state: State = 'idle';
  private _data: StateData[State] = null;
  
  get state(): State {
    return this._state;
  }
  
  get data() {
    return this._data;
  }
  
  transition<S extends State>(
    to: StateTransitions[typeof this._state] extends infer T
      ? T extends S ? S : never
      : never,
    data: StateData[typeof to]
  ): void {
    this._state = to;
    this._data = data;
  }
}
```

---

# 12. Configuration & Best Practices

## 12.1 tsconfig.json Options

```json
{
  "compilerOptions": {
    // Target and Module
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    
    // Strict Mode Options (recommended)
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    
    // Additional Checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    
    // Module Resolution
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    },
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    
    // Emit
    "outDir": "./dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noEmit": false,
    
    // JSX
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    
    // Interop
    "isolatedModules": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 12.2 Best Practices

```typescript
// 1. Prefer interfaces for public APIs
interface User {
  id: string;
  name: string;
}

// 2. Use type for unions, tuples, and complex types
type Status = 'pending' | 'approved' | 'rejected';
type Coordinates = [number, number];

// 3. Avoid 'any', use 'unknown' instead
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}

// 4. Use const assertions for readonly data
const ROLES = ['admin', 'user', 'guest'] as const;
type Role = typeof ROLES[number];

// 5. Prefer type narrowing over type assertions
function processValue(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase();  // Type is narrowed
  }
  return value.toFixed(2);
}

// 6. Use discriminated unions for state
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// 7. Prefer object parameters for functions with many arguments
function createUser(options: {
  name: string;
  email: string;
  age?: number;
  role?: Role;
}) {
  // ...
}

// 8. Use readonly for immutable data
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

// 9. Prefer explicit return types for public functions
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// 10. Use satisfies operator (TS 4.9+)
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
} satisfies Config;
```

## 12.3 Common Mistakes to Avoid

```typescript
// ❌ Using 'any' liberally
function bad(data: any) { /* ... */ }

// ✅ Use proper types or 'unknown'
function good(data: unknown) { /* ... */ }

// ❌ Type assertions without validation
const user = JSON.parse(data) as User;

// ✅ Validate before asserting
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}
const parsed = JSON.parse(data);
if (isUser(parsed)) {
  const user = parsed;  // Type is User
}

// ❌ Ignoring null checks
function getLength(str: string | null) {
  return str.length;  // Error!
}

// ✅ Handle null properly
function getLength(str: string | null) {
  return str?.length ?? 0;
}

// ❌ Overly complex types
type ComplexType<T, U, V extends keyof T, W extends U[keyof U]> = /* ... */;

// ✅ Break into smaller, named types
type SimpleType = /* ... */;
type HelperType<T> = /* ... */;
type FinalType<T> = SimpleType & HelperType<T>;
```

---

# 13. Common Interview Questions

## 13.1 Core TypeScript Questions

**Q1: What is TypeScript and why use it?**

**Answer:** TypeScript is a statically typed superset of JavaScript that compiles to plain JavaScript. Benefits include:
- Catch errors at compile time
- Better IDE support with autocomplete
- Self-documenting code
- Easier refactoring
- Interface definitions for APIs

---

**Q2: Explain the difference between `interface` and `type`.**

**Answer:**
- Both can define object shapes
- Interfaces support declaration merging
- Types support unions, tuples, and mapped types
- Interfaces are preferred for public APIs (extendable)
- Types are preferred for complex type operations

---

**Q3: What is the difference between `any` and `unknown`?**

**Answer:**
- `any` disables type checking entirely
- `unknown` requires type checking before use
- `unknown` is type-safe, `any` is not
- Use `unknown` for values of unknown type

---

**Q4: Explain generics in TypeScript.**

**Answer:** Generics allow creating reusable components that work with multiple types while maintaining type safety.

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

---

**Q5: What are utility types? Name some common ones.**

**Answer:** Built-in type transformations:
- `Partial<T>` - Makes all properties optional
- `Required<T>` - Makes all properties required
- `Readonly<T>` - Makes all properties readonly
- `Pick<T, K>` - Selects specific properties
- `Omit<T, K>` - Excludes specific properties
- `Record<K, V>` - Creates object type with keys K

---

**Q6: What is a discriminated union?**

**Answer:** A union type where each member has a common property (discriminant) that can be used to narrow the type.

```typescript
type Result = 
  | { type: 'success'; data: string }
  | { type: 'error'; message: string };

function handleResult(result: Result) {
  if (result.type === 'success') {
    console.log(result.data);  // Type narrowed to success
  }
}
```

---

**Q7: Explain type guards.**

**Answer:** Type guards are expressions that narrow types at runtime:
- `typeof` - For primitive types
- `instanceof` - For class instances
- `in` - For property checks
- Custom predicates with `is` keyword

---

**Q8: What is the `infer` keyword?**

**Answer:** `infer` declares a type variable within a conditional type, allowing extraction of types.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

---

**Q9: What are mapped types?**

**Answer:** Types that transform each property of an existing type.

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

---

**Q10: What is the `satisfies` operator?**

**Answer:** Added in TypeScript 4.9, it validates that an expression matches a type while preserving the narrowest type.

```typescript
const config = {
  port: 3000
} satisfies Config;
// config.port is number, not number | undefined
```

---

## 13.2 Output Questions

**Q11: What's the output?**

```typescript
type A = { a: number } & { b: string };
type B = { a: number } | { b: string };

const objA: A = { a: 1, b: "hello" };  // Required: both
// const objB: B = { a: 1 };  // OK: only a
// const objB: B = { b: "hi" };  // OK: only b
```

**Answer:** Type A requires both properties (intersection), Type B requires at least one (union).

---

**Q12: What's the difference?**

```typescript
interface Array<T> {
  map<U>(fn: (item: T) => U): Array<U>;
}

type Array2<T> = {
  map<U>(fn: (item: T) => U): Array2<U>;
}
```

**Answer:** Functionally similar, but interfaces support declaration merging while type aliases don't.

---

## 13.3 Practical Questions

**Q13: How do you type a React component with children?**

```typescript
interface Props {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: Props) {
  return <div>{title}{children}</div>;
}
```

---

**Q14: How do you type a custom hook that returns multiple values?**

```typescript
function useCounter(initial: number): [number, () => void, () => void] {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  return [count, increment, decrement];
}
```

---

**Q15: How do you make a type deeply readonly?**

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};
```

---

# 14. Coding Challenges

## 14.1 Implement Utility Types from Scratch

### Implement Partial

```typescript
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Test
interface User {
  name: string;
  age: number;
  email: string;
}

type PartialUser = MyPartial<User>;
// { name?: string; age?: number; email?: string }
```

### Implement Required

```typescript
type MyRequired<T> = {
  [P in keyof T]-?: T[P];  // -? removes optional modifier
};

// Test
interface Config {
  debug?: boolean;
  timeout?: number;
}

type RequiredConfig = MyRequired<Config>;
// { debug: boolean; timeout: number }
```

### Implement Readonly

```typescript
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Deep Readonly
type DeepReadonly<T> = T extends primitive
  ? T
  : T extends Array<infer U>
  ? ReadonlyArray<DeepReadonly<U>>
  : { readonly [P in keyof T]: DeepReadonly<T[P]> };

type primitive = string | number | boolean | null | undefined;
```

### Implement Pick

```typescript
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Test
type UserPreview = MyPick<User, "name" | "age">;
// { name: string; age: number }
```

### Implement Omit

```typescript
type MyOmit<T, K extends keyof any> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};

// Alternative using Pick and Exclude
type MyOmit2<T, K extends keyof any> = MyPick<T, Exclude<keyof T, K>>;

// Test
type UserWithoutEmail = MyOmit<User, "email">;
// { name: string; age: number }
```

### Implement Exclude and Extract

```typescript
type MyExclude<T, U> = T extends U ? never : T;

type MyExtract<T, U> = T extends U ? T : never;

// Test
type Result = MyExclude<"a" | "b" | "c", "a">;  // "b" | "c"
type Result2 = MyExtract<"a" | "b" | "c", "a" | "d">;  // "a"
```

### Implement NonNullable

```typescript
type MyNonNullable<T> = T extends null | undefined ? never : T;

// Test
type Result = MyNonNullable<string | null | undefined>;  // string
```

### Implement Record

```typescript
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};

// Test
type UserRecord = MyRecord<"name" | "email", string>;
// { name: string; email: string }
```

### Implement ReturnType

```typescript
type MyReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : never;

// Test
function getUser() {
  return { name: "Jayesh", age: 25 };
}

type User = MyReturnType<typeof getUser>;
// { name: string; age: number }
```

### Implement Parameters

```typescript
type MyParameters<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

// Test
function createUser(name: string, age: number) {}

type Params = MyParameters<typeof createUser>;
// [string, number]
```

## 14.2 Advanced Type Challenges

### Implement Awaited (Unwrap Promise)

```typescript
type MyAwaited<T> = T extends Promise<infer U>
  ? U extends Promise<any>
    ? MyAwaited<U>
    : U
  : T;

// Test
type Result = MyAwaited<Promise<Promise<string>>>;  // string
```

### Implement Tuple to Union

```typescript
type TupleToUnion<T extends readonly any[]> = T[number];

// Test
type Tuple = [string, number, boolean];
type Result = TupleToUnion<Tuple>;  // string | number | boolean
```

### Implement Union to Tuple

```typescript
// This is a complex one that uses conditional type distribution
type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends 
  ((k: infer I) => void) ? I : never;

type LastOfUnion<U> = UnionToIntersection<
  U extends any ? () => U : never
> extends () => infer R ? R : never;

type UnionToTuple<T, R extends any[] = []> = [T] extends [never]
  ? R
  : UnionToTuple<Exclude<T, LastOfUnion<T>>, [LastOfUnion<T>, ...R]>;

// Test
type Result = UnionToTuple<"a" | "b" | "c">;  // ["a", "b", "c"]
```

### Implement Length of Tuple

```typescript
type Length<T extends readonly any[]> = T["length"];

// Test
type Tuple = [string, number, boolean];
type Result = Length<Tuple>;  // 3
```

### Implement First of Array

```typescript
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;

// Test
type Result = First<[1, 2, 3]>;  // 1
type Empty = First<[]>;  // never
```

### Implement Last of Array

```typescript
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

// Test
type Result = Last<[1, 2, 3]>;  // 3
```

### Implement Concat

```typescript
type Concat<T extends any[], U extends any[]> = [...T, ...U];

// Test
type Result = Concat<[1, 2], [3, 4]>;  // [1, 2, 3, 4]
```

### Implement Push and Pop

```typescript
type Push<T extends any[], U> = [...T, U];
type Pop<T extends any[]> = T extends [...infer R, any] ? R : never;
type Shift<T extends any[]> = T extends [any, ...infer R] ? R : never;
type Unshift<T extends any[], U> = [U, ...T];

// Test
type Pushed = Push<[1, 2], 3>;  // [1, 2, 3]
type Popped = Pop<[1, 2, 3]>;  // [1, 2]
```

### Implement If

```typescript
type If<C extends boolean, T, F> = C extends true ? T : F;

// Test
type Result = If<true, "yes", "no">;  // "yes"
type Result2 = If<false, "yes", "no">;  // "no"
```

### Implement Includes

```typescript
type Includes<T extends readonly any[], U> = 
  T extends [infer First, ...infer Rest]
    ? Equal<First, U> extends true
      ? true
      : Includes<Rest, U>
    : false;

// Helper type for exact equality
type Equal<X, Y> = 
  (<T>() => T extends X ? 1 : 2) extends 
  (<T>() => T extends Y ? 1 : 2) ? true : false;

// Test
type Result = Includes<[1, 2, 3], 2>;  // true
type Result2 = Includes<[1, 2, 3], 4>;  // false
```

### Implement Flatten

```typescript
type Flatten<T extends any[]> = T extends [infer First, ...infer Rest]
  ? First extends any[]
    ? [...Flatten<First>, ...Flatten<Rest>]
    : [First, ...Flatten<Rest>]
  : [];

// Test
type Result = Flatten<[1, [2, [3, 4]], 5]>;  // [1, 2, 3, 4, 5]
```

## 14.3 String Type Challenges

### Implement Capitalize and Uncapitalize

```typescript
type MyCapitalize<S extends string> = S extends `${infer First}${infer Rest}`
  ? `${Uppercase<First>}${Rest}`
  : S;

type MyUncapitalize<S extends string> = S extends `${infer First}${infer Rest}`
  ? `${Lowercase<First>}${Rest}`
  : S;

// Test
type Result = MyCapitalize<"hello">;  // "Hello"
type Result2 = MyUncapitalize<"HELLO">;  // "hELLO"
```

### Implement Trim

```typescript
type TrimLeft<S extends string> = S extends `${' ' | '\n' | '\t'}${infer R}`
  ? TrimLeft<R>
  : S;

type TrimRight<S extends string> = S extends `${infer R}${' ' | '\n' | '\t'}`
  ? TrimRight<R>
  : S;

type Trim<S extends string> = TrimLeft<TrimRight<S>>;

// Test
type Result = Trim<"  hello world  ">;  // "hello world"
```

### Implement Replace

```typescript
type Replace<
  S extends string,
  From extends string,
  To extends string
> = From extends ''
  ? S
  : S extends `${infer Before}${From}${infer After}`
  ? `${Before}${To}${After}`
  : S;

type ReplaceAll<
  S extends string,
  From extends string,
  To extends string
> = From extends ''
  ? S
  : S extends `${infer Before}${From}${infer After}`
  ? ReplaceAll<`${Before}${To}${After}`, From, To>
  : S;

// Test
type Result = Replace<"hello world", "world", "there">;  // "hello there"
type Result2 = ReplaceAll<"a-b-c", "-", "_">;  // "a_b_c"
```

### Implement Split

```typescript
type Split<
  S extends string,
  D extends string
> = S extends `${infer Head}${D}${infer Tail}`
  ? [Head, ...Split<Tail, D>]
  : S extends ''
  ? []
  : [S];

// Test
type Result = Split<"a-b-c", "-">;  // ["a", "b", "c"]
```

### Implement Join

```typescript
type Join<
  T extends string[],
  D extends string
> = T extends []
  ? ''
  : T extends [infer F extends string]
  ? F
  : T extends [infer F extends string, ...infer R extends string[]]
  ? `${F}${D}${Join<R, D>}`
  : never;

// Test
type Result = Join<["a", "b", "c"], "-">;  // "a-b-c"
```

### Implement CamelCase

```typescript
type CamelCase<S extends string> = S extends `${infer Head}_${infer Tail}`
  ? `${Lowercase<Head>}${Capitalize<CamelCase<Tail>>}`
  : Lowercase<S>;

// Test
type Result = CamelCase<"hello_world_foo">;  // "helloWorldFoo"
```

### Implement KebabCase

```typescript
type KebabCase<S extends string> = S extends `${infer First}${infer Rest}`
  ? Rest extends Uncapitalize<Rest>
    ? `${Lowercase<First>}${KebabCase<Rest>}`
    : `${Lowercase<First>}-${KebabCase<Rest>}`
  : S;

// Test
type Result = KebabCase<"HelloWorldFoo">;  // "hello-world-foo"
```

## 14.4 Object Type Challenges

### Implement Merge

```typescript
type Merge<F, S> = {
  [K in keyof F | keyof S]: K extends keyof S
    ? S[K]
    : K extends keyof F
    ? F[K]
    : never;
};

// Test
type A = { a: number; b: string };
type B = { b: number; c: boolean };
type Result = Merge<A, B>;
// { a: number; b: number; c: boolean }
```

### Implement Diff

```typescript
type Diff<T, U> = {
  [K in keyof T | keyof U as K extends keyof T
    ? K extends keyof U
      ? never
      : K
    : K extends keyof U
    ? K
    : never]: K extends keyof T ? T[K] : K extends keyof U ? U[K] : never;
};

// Test
type A = { a: number; b: string; c: boolean };
type B = { b: string; c: boolean; d: number };
type Result = Diff<A, B>;  // { a: number; d: number }
```

### Implement RequiredKeys and OptionalKeys

```typescript
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// Test
interface User {
  name: string;
  age: number;
  email?: string;
}

type Required = RequiredKeys<User>;  // "name" | "age"
type Optional = OptionalKeys<User>;  // "email"
```

### Implement Mutable (Remove readonly)

```typescript
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Test
interface ReadonlyUser {
  readonly name: string;
  readonly age: number;
}

type MutableUser = Mutable<ReadonlyUser>;
// { name: string; age: number }
```

### Implement Getters and Setters

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

// Test
interface User {
  name: string;
  age: number;
}

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }

type UserSetters = Setters<User>;
// { setName: (value: string) => void; setAge: (value: number) => void }
```

---

# 15. Interview Coding Exercises

## 15.1 Type-Safe API Response Handler

```typescript
type ApiResponse<T> =
  | { status: "success"; data: T }
  | { status: "error"; error: string }
  | { status: "loading" };

function handleResponse<T>(
  response: ApiResponse<T>,
  handlers: {
    onSuccess: (data: T) => void;
    onError: (error: string) => void;
    onLoading: () => void;
  }
): void {
  switch (response.status) {
    case "success":
      handlers.onSuccess(response.data);
      break;
    case "error":
      handlers.onError(response.error);
      break;
    case "loading":
      handlers.onLoading();
      break;
  }
}

// Usage
interface User {
  id: number;
  name: string;
}

const response: ApiResponse<User> = {
  status: "success",
  data: { id: 1, name: "Jayesh" }
};

handleResponse(response, {
  onSuccess: (user) => console.log(user.name),
  onError: (err) => console.error(err),
  onLoading: () => console.log("Loading...")
});
```

## 15.2 Type-Safe Form Validation

```typescript
type ValidationRule<T> = {
  validate: (value: T) => boolean;
  message: string;
};

type FormSchema<T> = {
  [K in keyof T]: ValidationRule<T[K]>[];
};

type ValidationErrors<T> = {
  [K in keyof T]?: string[];
};

function validateForm<T extends Record<string, any>>(
  data: T,
  schema: FormSchema<T>
): ValidationErrors<T> {
  const errors: ValidationErrors<T> = {};

  for (const key in schema) {
    const rules = schema[key];
    const value = data[key];
    const fieldErrors: string[] = [];

    for (const rule of rules) {
      if (!rule.validate(value)) {
        fieldErrors.push(rule.message);
      }
    }

    if (fieldErrors.length > 0) {
      errors[key] = fieldErrors;
    }
  }

  return errors;
}

// Usage
interface UserForm {
  name: string;
  email: string;
  age: number;
}

const schema: FormSchema<UserForm> = {
  name: [
    { validate: (v) => v.length >= 2, message: "Name must be at least 2 chars" }
  ],
  email: [
    { validate: (v) => v.includes("@"), message: "Invalid email format" }
  ],
  age: [
    { validate: (v) => v >= 18, message: "Must be 18 or older" }
  ]
};

const errors = validateForm(
  { name: "J", email: "test", age: 16 },
  schema
);
```

## 15.3 Type-Safe Event System

```typescript
type EventHandler<T = void> = (payload: T) => void;

class TypedEventEmitter<Events extends Record<string, any>> {
  private handlers: {
    [K in keyof Events]?: Set<EventHandler<Events[K]>>;
  } = {};

  on<K extends keyof Events>(
    event: K,
    handler: EventHandler<Events[K]>
  ): () => void {
    if (!this.handlers[event]) {
      this.handlers[event] = new Set();
    }
    this.handlers[event]!.add(handler);
    
    return () => this.off(event, handler);
  }

  off<K extends keyof Events>(
    event: K,
    handler: EventHandler<Events[K]>
  ): void {
    this.handlers[event]?.delete(handler);
  }

  emit<K extends keyof Events>(
    event: K,
    payload: Events[K]
  ): void {
    this.handlers[event]?.forEach(handler => handler(payload));
  }
}

// Usage
interface GameEvents {
  start: { difficulty: "easy" | "medium" | "hard" };
  score: { points: number; player: string };
  end: { winner: string };
}

const game = new TypedEventEmitter<GameEvents>();

game.on("score", ({ points, player }) => {
  console.log(`${player} scored ${points} points!`);
});

game.emit("score", { points: 100, player: "Jayesh" });
```

## 15.4 Type-Safe State Machine

```typescript
type States = {
  idle: null;
  loading: { url: string };
  success: { data: unknown };
  error: { message: string };
};

type StateKey = keyof States;

type Transition = {
  idle: "loading";
  loading: "success" | "error";
  success: "idle";
  error: "idle" | "loading";
};

class StateMachine {
  private _state: StateKey = "idle";
  private _context: States[StateKey] = null;

  get state() {
    return { type: this._state, context: this._context };
  }

  transition<
    From extends StateKey,
    To extends Transition[From]
  >(to: To, context: States[To]): void {
    this._state = to;
    this._context = context;
  }

  is<S extends StateKey>(state: S): this is StateMachine & { state: { type: S; context: States[S] } } {
    return this._state === state;
  }
}
```

## 15.5 Type-Safe Query Builder

```typescript
interface WhereClause<T> {
  field: keyof T;
  operator: "=" | "!=" | ">" | "<" | ">=" | "<=" | "LIKE";
  value: T[keyof T];
}

class QueryBuilder<T extends Record<string, any>> {
  private selectFields: (keyof T)[] = [];
  private whereClauses: WhereClause<T>[] = [];
  private orderByField?: keyof T;
  private orderDirection: "ASC" | "DESC" = "ASC";
  private limitCount?: number;

  select<K extends keyof T>(...fields: K[]): this {
    this.selectFields = fields;
    return this;
  }

  where<K extends keyof T>(
    field: K,
    operator: WhereClause<T>["operator"],
    value: T[K]
  ): this {
    this.whereClauses.push({ field, operator, value });
    return this;
  }

  orderBy<K extends keyof T>(field: K, direction: "ASC" | "DESC" = "ASC"): this {
    this.orderByField = field;
    this.orderDirection = direction;
    return this;
  }

  limit(count: number): this {
    this.limitCount = count;
    return this;
  }

  toSQL(): string {
    const fields = this.selectFields.length > 0
      ? this.selectFields.join(", ")
      : "*";

    let sql = `SELECT ${fields} FROM table`;

    if (this.whereClauses.length > 0) {
      const conditions = this.whereClauses
        .map(c => `${String(c.field)} ${c.operator} '${c.value}'`)
        .join(" AND ");
      sql += ` WHERE ${conditions}`;
    }

    if (this.orderByField) {
      sql += ` ORDER BY ${String(this.orderByField)} ${this.orderDirection}`;
    }

    if (this.limitCount) {
      sql += ` LIMIT ${this.limitCount}`;
    }

    return sql;
  }
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

const query = new QueryBuilder<User>()
  .select("id", "name", "email")
  .where("age", ">=", 18)
  .where("name", "LIKE", "Jay%")
  .orderBy("name", "ASC")
  .limit(10)
  .toSQL();
```

---

# 16. TypeScript Type Gymnastics

## 16.1 Is Type Literal

```typescript
type IsLiteral<T> = T extends string
  ? string extends T
    ? false
    : true
  : T extends number
  ? number extends T
    ? false
    : true
  : T extends boolean
  ? boolean extends T
    ? false
    : true
  : false;

// Test
type A = IsLiteral<"hello">;  // true
type B = IsLiteral<string>;   // false
type C = IsLiteral<42>;       // true
type D = IsLiteral<number>;   // false
```

## 16.2 Is Empty Object

```typescript
type IsEmptyObject<T> = T extends Record<string, never>
  ? keyof T extends never
    ? true
    : false
  : false;

// Alternative
type IsEmptyObject2<T> = [keyof T] extends [never] ? true : false;

// Test
type A = IsEmptyObject<{}>;           // true
type B = IsEmptyObject<{ a: 1 }>;     // false
```

## 16.3 Get Optional Keys

```typescript
type GetOptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

type GetRequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

// Test
interface User {
  name: string;
  age?: number;
  email?: string;
}

type Optional = GetOptionalKeys<User>;  // "age" | "email"
type Required = GetRequiredKeys<User>;  // "name"
```

## 16.4 Deep Partial

```typescript
type DeepPartial<T> = T extends object
  ? T extends Function
    ? T
    : {
        [P in keyof T]?: DeepPartial<T[P]>;
      }
  : T;

// Test
interface Config {
  server: {
    port: number;
    host: string;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    url: string;
  };
}

type PartialConfig = DeepPartial<Config>;
// All properties at all levels become optional
```

## 16.5 Path Types

```typescript
type Path<T, Key extends keyof T = keyof T> = Key extends string
  ? T[Key] extends Record<string, any>
    ? `${Key}.${Path<T[Key]>}` | Key
    : Key
  : never;

type PathValue<T, P extends string> = P extends `${infer Key}.${infer Rest}`
  ? Key extends keyof T
    ? PathValue<T[Key], Rest>
    : never
  : P extends keyof T
  ? T[P]
  : never;

// Test
interface User {
  name: string;
  address: {
    city: string;
    country: {
      name: string;
      code: string;
    };
  };
}

type UserPaths = Path<User>;
// "name" | "address" | "address.city" | "address.country" | "address.country.name" | "address.country.code"

type CityType = PathValue<User, "address.city">;  // string
type CodeType = PathValue<User, "address.country.code">;  // string
```

---

# 17. Quick Reference Cheat Sheet

## Type Annotations

```typescript
// Primitives
let str: string = "hello";
let num: number = 42;
let bool: boolean = true;
let nul: null = null;
let undef: undefined = undefined;
let sym: symbol = Symbol();
let big: bigint = 100n;

// Arrays
let arr: number[] = [1, 2, 3];
let arr2: Array<string> = ["a", "b"];
let tuple: [string, number] = ["hello", 42];

// Objects
let obj: { name: string; age?: number } = { name: "Jayesh" };

// Functions
let fn: (x: number) => string = (x) => String(x);
type Fn = (x: number) => string;
```

## Type Operations

```typescript
// Union and Intersection
type A = string | number;     // Either
type B = TypeA & TypeB;       // Both

// Keyof and typeof
type Keys = keyof User;       // Union of keys
type FnType = typeof myFn;    // Type of value

// Indexed Access
type Name = User["name"];     // Property type
type Values = User[keyof User]; // All value types

// Conditional
type IsString<T> = T extends string ? true : false;

// Mapped
type Readonly<T> = { readonly [K in keyof T]: T[K] };
```

## Utility Types Quick Reference

```typescript
Partial<T>        // All optional
Required<T>       // All required
Readonly<T>       // All readonly
Pick<T, K>        // Select keys
Omit<T, K>        // Exclude keys
Record<K, T>      // Object with keys K, values T
Exclude<T, U>     // Remove types
Extract<T, U>     // Keep types
NonNullable<T>    // Remove null/undefined
ReturnType<T>     // Function return type
Parameters<T>     // Function parameters
InstanceType<T>   // Class instance type
Awaited<T>        // Unwrap Promise
```

## React Types

```typescript
React.FC<Props>           // Function component
React.ReactNode           // Any renderable
React.ReactElement        // JSX element
React.CSSProperties       // Inline styles
React.ChangeEvent<T>      // Change event
React.FormEvent<T>        // Form event
React.MouseEvent<T>       // Mouse event
React.KeyboardEvent<T>    // Keyboard event
React.RefObject<T>        // Ref object
React.ComponentProps<C>   // Component props
```

---

## 🎯 Interview Preparation Tips

### Before the Interview

1. **Review fundamentals**: types, interfaces, generics, utility types
2. **Practice type challenges**: TypeScript Type Challenges on GitHub
3. **Know tsconfig options**: strict mode, module resolution
4. **Understand React + TypeScript**: hooks, events, context

### During the Interview

1. **Start simple**: Begin with basic types, then add complexity
2. **Explain your reasoning**: Walk through type decisions
3. **Use proper terminology**: generics, union, intersection, narrowing
4. **Handle edge cases**: null, undefined, empty arrays

### Common Mistakes to Avoid

1. Using `any` instead of proper types
2. Not handling null/undefined cases
3. Over-complicating types (keep them readable)
4. Forgetting to narrow types before use
5. Using type assertions without validation

---

*End of Part 2: TypeScript Mastery - Complete Interview Guide*

> **Total Coverage:** 6000+ lines of comprehensive TypeScript content including:
> - Core type system and fundamentals
> - Advanced types (conditional, mapped, template literals)
> - Generics complete guide
> - All utility types with implementations
> - Type guards and narrowing
> - Modules and declaration files
> - Classes and OOP patterns
> - Decorators
> - React + TypeScript integration
> - Advanced patterns (brand types, builders, event emitters)
> - tsconfig.json configuration
> - 50+ interview questions
> - 30+ coding challenges with solutions

---

# 18. Real-World TypeScript Patterns

## 18.1 API Response Typing

When working with APIs, you need robust type definitions to handle various response structures.

```typescript
// Base API response structure
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
  meta?: {
    page: number;
    perPage: number;
    total: number;
    totalPages: number;
  };
}

// Paginated response
interface PaginatedResponse<T> extends ApiResponse<T[]> {
  meta: {
    page: number;
    perPage: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

// Error response
interface ErrorResponse {
  success: false;
  error: string;
  code: string;
  details?: Record<string, string[]>;
}

// Type-safe API client
class ApiClient {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    
    if (!response.ok) {
      const error: ErrorResponse = await response.json();
      throw new Error(error.error);
    }
    
    return response.json();
  }
  
  async post<T, D>(endpoint: string, data: D): Promise<ApiResponse<T>> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    return response.json();
  }
  
  async getPaginated<T>(
    endpoint: string, 
    page: number = 1
  ): Promise<PaginatedResponse<T>> {
    return this.get<T[]>(`${endpoint}?page=${page}`) as Promise<PaginatedResponse<T>>;
  }
}

// Usage with strong typing
interface User {
  id: string;
  name: string;
  email: string;
}

const api = new ApiClient('https://api.example.com');

async function fetchUsers() {
  const response = await api.getPaginated<User>('/users');
  
  if (response.success) {
    // response.data is User[]
    response.data.forEach(user => {
      console.log(user.name);  // TypeScript knows this is string
    });
    
    // Access pagination info
    if (response.meta.hasNext) {
      // Fetch next page
    }
  }
}
```

## 18.2 Form Handling with TypeScript

Type-safe form handling is crucial in React applications.

```typescript
// Form field types
type FormFieldType = 'text' | 'email' | 'password' | 'number' | 'select' | 'checkbox';

interface FormFieldConfig<T> {
  name: keyof T;
  type: FormFieldType;
  label: string;
  placeholder?: string;
  required?: boolean;
  validation?: (value: T[keyof T]) => string | null;
  options?: { value: string; label: string }[];  // For select fields
}

// Form state with type safety
interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
}

// Type-safe form hook
function useForm<T extends Record<string, any>>(
  initialValues: T,
  onSubmit: (values: T) => Promise<void>,
  validate?: (values: T) => Partial<Record<keyof T, string>>
) {
  const [state, setState] = useState<FormState<T>>({
    values: initialValues,
    errors: {},
    touched: {},
    isSubmitting: false,
    isValid: true
  });
  
  const setFieldValue = <K extends keyof T>(field: K, value: T[K]) => {
    setState(prev => ({
      ...prev,
      values: { ...prev.values, [field]: value },
      touched: { ...prev.touched, [field]: true }
    }));
  };
  
  const setFieldError = <K extends keyof T>(field: K, error: string) => {
    setState(prev => ({
      ...prev,
      errors: { ...prev.errors, [field]: error }
    }));
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (validate) {
      const errors = validate(state.values);
      if (Object.keys(errors).length > 0) {
        setState(prev => ({ ...prev, errors, isValid: false }));
        return;
      }
    }
    
    setState(prev => ({ ...prev, isSubmitting: true }));
    
    try {
      await onSubmit(state.values);
    } finally {
      setState(prev => ({ ...prev, isSubmitting: false }));
    }
  };
  
  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    setFieldValue,
    setFieldError,
    handleSubmit
  };
}

// Example usage
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

function LoginComponent() {
  const form = useForm<LoginForm>(
    { email: '', password: '', rememberMe: false },
    async (values) => {
      await api.post('/auth/login', values);
    },
    (values) => {
      const errors: Partial<Record<keyof LoginForm, string>> = {};
      
      if (!values.email) {
        errors.email = 'Email is required';
      } else if (!values.email.includes('@')) {
        errors.email = 'Invalid email format';
      }
      
      if (!values.password) {
        errors.password = 'Password is required';
      } else if (values.password.length < 8) {
        errors.password = 'Password must be at least 8 characters';
      }
      
      return errors;
    }
  );
  
  return (
    <form onSubmit={form.handleSubmit}>
      <input
        type="email"
        value={form.values.email}
        onChange={(e) => form.setFieldValue('email', e.target.value)}
      />
      {form.errors.email && <span>{form.errors.email}</span>}
      
      <input
        type="password"
        value={form.values.password}
        onChange={(e) => form.setFieldValue('password', e.target.value)}
      />
      {form.errors.password && <span>{form.errors.password}</span>}
      
      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

## 18.3 State Management Patterns

Type-safe state management with Zustand-like patterns.

```typescript
// Store types
type SetState<T> = (
  partial: T | Partial<T> | ((state: T) => T | Partial<T>)
) => void;

type GetState<T> = () => T;

type StoreApi<T> = {
  getState: GetState<T>;
  setState: SetState<T>;
  subscribe: (listener: (state: T) => void) => () => void;
};

// Create store function
function createStore<T>(
  initializer: (set: SetState<T>, get: GetState<T>) => T
): StoreApi<T> {
  let state: T;
  const listeners = new Set<(state: T) => void>();
  
  const getState: GetState<T> = () => state;
  
  const setState: SetState<T> = (partial) => {
    const nextState = typeof partial === 'function'
      ? (partial as (state: T) => T | Partial<T>)(state)
      : partial;
    
    if (nextState !== state) {
      state = typeof nextState === 'object'
        ? { ...state, ...nextState }
        : nextState as T;
      
      listeners.forEach(listener => listener(state));
    }
  };
  
  const subscribe = (listener: (state: T) => void) => {
    listeners.add(listener);
    return () => listeners.delete(listener);
  };
  
  state = initializer(setState, getState);
  
  return { getState, setState, subscribe };
}

// Example: User store
interface UserState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setUser: (user: User) => void;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  clearError: () => void;
}

const useUserStore = createStore<UserState>((set, get) => ({
  user: null,
  isLoading: false,
  error: null,
  
  setUser: (user) => set({ user }),
  
  login: async (email, password) => {
    set({ isLoading: true, error: null });
    
    try {
      const response = await api.post<User, { email: string; password: string }>(
        '/auth/login',
        { email, password }
      );
      
      if (response.success) {
        set({ user: response.data, isLoading: false });
      }
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Login failed',
        isLoading: false 
      });
    }
  },
  
  logout: () => set({ user: null }),
  
  clearError: () => set({ error: null })
}));
```

## 18.4 Middleware Pattern

Type-safe middleware implementation for request/response processing.

```typescript
// Middleware types
type NextFunction = () => Promise<void>;
type Middleware<TContext> = (
  context: TContext,
  next: NextFunction
) => Promise<void>;

// Context type for HTTP-like requests
interface RequestContext {
  request: {
    method: string;
    url: string;
    headers: Record<string, string>;
    body: unknown;
  };
  response: {
    status: number;
    headers: Record<string, string>;
    body: unknown;
  };
  state: Record<string, unknown>;
}

// Middleware composer
class MiddlewareComposer<TContext> {
  private middlewares: Middleware<TContext>[] = [];
  
  use(middleware: Middleware<TContext>): this {
    this.middlewares.push(middleware);
    return this;
  }
  
  async execute(context: TContext): Promise<void> {
    const dispatch = async (index: number): Promise<void> => {
      if (index >= this.middlewares.length) {
        return;
      }
      
      const middleware = this.middlewares[index];
      await middleware(context, () => dispatch(index + 1));
    };
    
    await dispatch(0);
  }
}

// Example middlewares
const loggingMiddleware: Middleware<RequestContext> = async (ctx, next) => {
  console.log(`${ctx.request.method} ${ctx.request.url}`);
  const start = Date.now();
  
  await next();
  
  const duration = Date.now() - start;
  console.log(`Response: ${ctx.response.status} (${duration}ms)`);
};

const authMiddleware: Middleware<RequestContext> = async (ctx, next) => {
  const token = ctx.request.headers['authorization'];
  
  if (!token) {
    ctx.response.status = 401;
    ctx.response.body = { error: 'Unauthorized' };
    return;
  }
  
  // Validate token and add user to state
  ctx.state.user = { id: '123', name: 'Jayesh' };
  
  await next();
};

const errorMiddleware: Middleware<RequestContext> = async (ctx, next) => {
  try {
    await next();
  } catch (error) {
    ctx.response.status = 500;
    ctx.response.body = { 
      error: error instanceof Error ? error.message : 'Internal Server Error' 
    };
  }
};

// Usage
const app = new MiddlewareComposer<RequestContext>();

app
  .use(errorMiddleware)
  .use(loggingMiddleware)
  .use(authMiddleware)
  .use(async (ctx) => {
    // Handle request
    ctx.response.status = 200;
    ctx.response.body = { message: 'Hello, ' + (ctx.state.user as User).name };
  });
```

## 18.5 Repository Pattern

Type-safe repository pattern for data access.

```typescript
// Base entity interface
interface Entity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

// Repository interface
interface Repository<T extends Entity> {
  findById(id: string): Promise<T | null>;
  findAll(filter?: Partial<T>): Promise<T[]>;
  create(data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>): Promise<T>;
  update(id: string, data: Partial<Omit<T, 'id' | 'createdAt' | 'updatedAt'>>): Promise<T>;
  delete(id: string): Promise<boolean>;
}

// Query options
interface QueryOptions<T> {
  where?: Partial<T>;
  orderBy?: { field: keyof T; direction: 'asc' | 'desc' };
  limit?: number;
  offset?: number;
  include?: (keyof T)[];
}

// Extended repository interface
interface ExtendedRepository<T extends Entity> extends Repository<T> {
  findOne(options: QueryOptions<T>): Promise<T | null>;
  findMany(options: QueryOptions<T>): Promise<T[]>;
  count(filter?: Partial<T>): Promise<number>;
  exists(id: string): Promise<boolean>;
}

// User entity
interface User extends Entity {
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  profile?: {
    bio: string;
    avatar: string;
  };
}

// In-memory implementation for demonstration
class InMemoryRepository<T extends Entity> implements ExtendedRepository<T> {
  protected items: Map<string, T> = new Map();
  
  async findById(id: string): Promise<T | null> {
    return this.items.get(id) || null;
  }
  
  async findAll(filter?: Partial<T>): Promise<T[]> {
    let results = Array.from(this.items.values());
    
    if (filter) {
      results = results.filter(item => 
        Object.entries(filter).every(([key, value]) => 
          item[key as keyof T] === value
        )
      );
    }
    
    return results;
  }
  
  async create(data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>): Promise<T> {
    const now = new Date();
    const item = {
      ...data,
      id: crypto.randomUUID(),
      createdAt: now,
      updatedAt: now
    } as T;
    
    this.items.set(item.id, item);
    return item;
  }
  
  async update(
    id: string, 
    data: Partial<Omit<T, 'id' | 'createdAt' | 'updatedAt'>>
  ): Promise<T> {
    const existing = await this.findById(id);
    
    if (!existing) {
      throw new Error(`Entity with id ${id} not found`);
    }
    
    const updated = {
      ...existing,
      ...data,
      updatedAt: new Date()
    };
    
    this.items.set(id, updated);
    return updated;
  }
  
  async delete(id: string): Promise<boolean> {
    return this.items.delete(id);
  }
  
  async findOne(options: QueryOptions<T>): Promise<T | null> {
    const results = await this.findMany({ ...options, limit: 1 });
    return results[0] || null;
  }
  
  async findMany(options: QueryOptions<T>): Promise<T[]> {
    let results = await this.findAll(options.where);
    
    if (options.orderBy) {
      results.sort((a, b) => {
        const aVal = a[options.orderBy!.field];
        const bVal = b[options.orderBy!.field];
        const comparison = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        return options.orderBy!.direction === 'asc' ? comparison : -comparison;
      });
    }
    
    if (options.offset) {
      results = results.slice(options.offset);
    }
    
    if (options.limit) {
      results = results.slice(0, options.limit);
    }
    
    return results;
  }
  
  async count(filter?: Partial<T>): Promise<number> {
    const results = await this.findAll(filter);
    return results.length;
  }
  
  async exists(id: string): Promise<boolean> {
    return this.items.has(id);
  }
}

// User repository with custom methods
class UserRepository extends InMemoryRepository<User> {
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } as Partial<User> });
  }
  
  async findByRole(role: User['role']): Promise<User[]> {
    return this.findAll({ role });
  }
  
  async updateProfile(
    id: string, 
    profile: User['profile']
  ): Promise<User> {
    return this.update(id, { profile });
  }
}
```

---

# 19. TypeScript Error Handling Patterns

## 19.1 Result Type Pattern

Instead of throwing exceptions, use Result types for explicit error handling.

```typescript
// Result type definition
type Result<T, E = Error> = 
  | { success: true; value: T }
  | { success: false; error: E };

// Result helper functions
const Ok = <T>(value: T): Result<T, never> => ({
  success: true,
  value
});

const Err = <E>(error: E): Result<never, E> => ({
  success: false,
  error
});

// Type guard
function isOk<T, E>(result: Result<T, E>): result is { success: true; value: T } {
  return result.success === true;
}

function isErr<T, E>(result: Result<T, E>): result is { success: false; error: E } {
  return result.success === false;
}

// Example usage
interface ValidationError {
  field: string;
  message: string;
}

function validateEmail(email: string): Result<string, ValidationError> {
  if (!email) {
    return Err({ field: 'email', message: 'Email is required' });
  }
  
  if (!email.includes('@')) {
    return Err({ field: 'email', message: 'Invalid email format' });
  }
  
  return Ok(email.toLowerCase().trim());
}

function validateAge(age: number): Result<number, ValidationError> {
  if (age < 0 || age > 150) {
    return Err({ field: 'age', message: 'Age must be between 0 and 150' });
  }
  
  return Ok(age);
}

// Combining results
function validateUser(data: { email: string; age: number }): Result<
  { email: string; age: number },
  ValidationError[]
> {
  const errors: ValidationError[] = [];
  
  const emailResult = validateEmail(data.email);
  if (isErr(emailResult)) {
    errors.push(emailResult.error);
  }
  
  const ageResult = validateAge(data.age);
  if (isErr(ageResult)) {
    errors.push(ageResult.error);
  }
  
  if (errors.length > 0) {
    return Err(errors);
  }
  
  return Ok({
    email: (emailResult as { success: true; value: string }).value,
    age: (ageResult as { success: true; value: number }).value
  });
}

// Usage
const result = validateUser({ email: 'invalid', age: -5 });

if (isOk(result)) {
  console.log('Valid user:', result.value);
} else {
  console.log('Validation errors:', result.error);
}
```

## 19.2 Option/Maybe Type

Type-safe handling of nullable values.

```typescript
// Option type
type Option<T> = Some<T> | None;

interface Some<T> {
  _tag: 'Some';
  value: T;
}

interface None {
  _tag: 'None';
}

// Constructor functions
const Some = <T>(value: T): Option<T> => ({ _tag: 'Some', value });
const None: Option<never> = { _tag: 'None' };

// Type guards
const isSome = <T>(option: Option<T>): option is Some<T> => 
  option._tag === 'Some';

const isNone = <T>(option: Option<T>): option is None => 
  option._tag === 'None';

// Option utilities
const fromNullable = <T>(value: T | null | undefined): Option<T> =>
  value == null ? None : Some(value);

const map = <T, U>(option: Option<T>, fn: (value: T) => U): Option<U> =>
  isSome(option) ? Some(fn(option.value)) : None;

const flatMap = <T, U>(option: Option<T>, fn: (value: T) => Option<U>): Option<U> =>
  isSome(option) ? fn(option.value) : None;

const getOrElse = <T>(option: Option<T>, defaultValue: T): T =>
  isSome(option) ? option.value : defaultValue;

const getOrThrow = <T>(option: Option<T>, errorMessage: string): T => {
  if (isNone(option)) {
    throw new Error(errorMessage);
  }
  return option.value;
};

// Example usage
interface User {
  id: string;
  name: string;
  email?: string;
  address?: {
    city: string;
    country: string;
  };
}

function getUserEmail(user: User): Option<string> {
  return fromNullable(user.email);
}

function getUserCity(user: User): Option<string> {
  return flatMap(
    fromNullable(user.address),
    (address) => fromNullable(address.city)
  );
}

const user: User = { id: '1', name: 'Jayesh' };

const email = getUserEmail(user);
const displayEmail = getOrElse(email, 'No email provided');

const city = getUserCity(user);
const displayCity = map(city, (c) => `City: ${c}`);
```

## 19.3 Either Type

More flexible than Result for representing two possible outcomes.

```typescript
// Either type
type Either<L, R> = Left<L> | Right<R>;

interface Left<L> {
  _tag: 'Left';
  left: L;
}

interface Right<R> {
  _tag: 'Right';
  right: R;
}

// Constructors
const Left = <L>(value: L): Either<L, never> => ({ _tag: 'Left', left: value });
const Right = <R>(value: R): Either<never, R> => ({ _tag: 'Right', right: value });

// Type guards
const isLeft = <L, R>(either: Either<L, R>): either is Left<L> =>
  either._tag === 'Left';

const isRight = <L, R>(either: Either<L, R>): either is Right<R> =>
  either._tag === 'Right';

// Utilities
const mapEither = <L, R, R2>(
  either: Either<L, R>,
  fn: (right: R) => R2
): Either<L, R2> =>
  isRight(either) ? Right(fn(either.right)) : either;

const mapLeft = <L, L2, R>(
  either: Either<L, R>,
  fn: (left: L) => L2
): Either<L2, R> =>
  isLeft(either) ? Left(fn(either.left)) : either;

const fold = <L, R, T>(
  either: Either<L, R>,
  onLeft: (left: L) => T,
  onRight: (right: R) => T
): T =>
  isLeft(either) ? onLeft(either.left) : onRight(either.right);

// Async version
type AsyncEither<L, R> = Promise<Either<L, R>>;

const tryCatch = async <L, R>(
  fn: () => Promise<R>,
  onError: (error: unknown) => L
): AsyncEither<L, R> => {
  try {
    const result = await fn();
    return Right(result);
  } catch (error) {
    return Left(onError(error));
  }
};

// Example
interface ApiError {
  code: string;
  message: string;
}

async function fetchUser(id: string): AsyncEither<ApiError, User> {
  return tryCatch(
    async () => {
      const response = await fetch(`/api/users/${id}`);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json();
    },
    (error) => ({
      code: 'FETCH_ERROR',
      message: error instanceof Error ? error.message : 'Unknown error'
    })
  );
}

// Usage
const userResult = await fetchUser('123');

const message = fold(
  userResult,
  (error) => `Error: ${error.message}`,
  (user) => `Welcome, ${user.name}!`
);
```

---

# 20. Performance Tips for TypeScript

## 20.1 Type Performance Optimization

Complex types can slow down the TypeScript compiler. Here are tips to improve type checking performance.

```typescript
// ❌ Deeply nested conditional types (slow)
type DeepCheck<T> = T extends object
  ? T extends Array<infer U>
    ? U extends object
      ? DeepCheck<U>
      : U
    : { [K in keyof T]: DeepCheck<T[K]> }
  : T;

// ✅ Use type caching with intermediate types
type IsArray<T> = T extends Array<infer U> ? U : never;
type IsObject<T> = T extends object ? T : never;

// ❌ Large union types (slow)
type AllNumbers = 1 | 2 | 3 | 4 | 5 | ... | 1000;

// ✅ Use ranges or branded types instead
type NumberRange = number & { __brand: 'range1to1000' };

// ❌ Recursive types without limits
type InfiniteRecursion<T> = T extends any[] 
  ? InfiniteRecursion<T[number]> 
  : T;

// ✅ Add recursion limits
type LimitedRecursion<T, Depth extends number[] = []> = 
  Depth['length'] extends 10
    ? T
    : T extends any[]
    ? LimitedRecursion<T[number], [...Depth, 1]>
    : T;
```

## 20.2 Interface vs Type Alias Performance

```typescript
// ✅ Interfaces are faster for extending
interface User {
  name: string;
  age: number;
}

interface Admin extends User {
  role: string;
}

// ❌ Type intersections can be slower
type User2 = {
  name: string;
  age: number;
};

type Admin2 = User2 & { role: string };

// ✅ Prefer interfaces for object shapes that will be extended
// ✅ Use type aliases for unions, tuples, and computed types
```

## 20.3 Project References for Large Codebases

```json
// tsconfig.json - root
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true
  },
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ]
}

// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

---

# 21. Testing TypeScript Code

## 21.1 Type Testing with Expect-Type

```typescript
import { expectType, expectError } from 'tsd';

// Test that a type works as expected
interface User {
  name: string;
  age: number;
}

type UserName = User['name'];
expectType<string>({} as UserName);

// Test that certain code produces an error
declare function takesString(s: string): void;

// expectError(takesString(123));  // Should error

// Custom type testing utilities
type Expect<T extends true> = T;
type Equal<X, Y> = 
  (<T>() => T extends X ? 1 : 2) extends 
  (<T>() => T extends Y ? 1 : 2) ? true : false;

// Usage
type Test1 = Expect<Equal<string, string>>;  // passes
// type Test2 = Expect<Equal<string, number>>;  // fails
```

## 21.2 Testing with Jest and TypeScript

```typescript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  moduleFileExtensions: ['ts', 'tsx', 'js'],
  transform: {
    '^.+\\.tsx?$': 'ts-jest'
  },
  testMatch: ['**/*.test.ts', '**/*.spec.ts']
};

// user.service.test.ts
import { UserService, User, CreateUserDTO } from './user.service';

describe('UserService', () => {
  let service: UserService;
  
  beforeEach(() => {
    service = new UserService();
  });
  
  describe('createUser', () => {
    it('should create a user with valid data', async () => {
      const dto: CreateUserDTO = {
        name: 'Jayesh',
        email: 'jayesh@example.com',
        age: 25
      };
      
      const result = await service.createUser(dto);
      
      expect(result).toMatchObject({
        name: 'Jayesh',
        email: 'jayesh@example.com',
        age: 25
      });
      expect(result.id).toBeDefined();
    });
    
    it('should throw error for invalid email', async () => {
      const dto: CreateUserDTO = {
        name: 'Test',
        email: 'invalid',
        age: 25
      };
      
      await expect(service.createUser(dto)).rejects.toThrow('Invalid email');
    });
  });
});

// Mock example
jest.mock('./api-client');

import { apiClient } from './api-client';

const mockedApiClient = apiClient as jest.Mocked<typeof apiClient>;

test('fetches user from API', async () => {
  const mockUser: User = {
    id: '1',
    name: 'Jayesh',
    email: 'jayesh@example.com',
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  mockedApiClient.get.mockResolvedValue({ success: true, data: mockUser });
  
  const result = await service.getUser('1');
  
  expect(result).toEqual(mockUser);
  expect(mockedApiClient.get).toHaveBeenCalledWith('/users/1');
});
```

---

# 22. TypeScript Compiler API Basics

## 22.1 Programmatic Type Checking

```typescript
import * as ts from 'typescript';

// Create a program
const program = ts.createProgram(['./src/index.ts'], {
  target: ts.ScriptTarget.ES2020,
  module: ts.ModuleKind.ESNext,
  strict: true
});

// Get type checker
const checker = program.getTypeChecker();

// Analyze a source file
const sourceFile = program.getSourceFile('./src/index.ts');

if (sourceFile) {
  ts.forEachChild(sourceFile, (node) => {
    if (ts.isFunctionDeclaration(node) && node.name) {
      const symbol = checker.getSymbolAtLocation(node.name);
      
      if (symbol) {
        const type = checker.getTypeOfSymbolAtLocation(symbol, node);
        const typeString = checker.typeToString(type);
        
        console.log(`Function: ${symbol.getName()}`);
        console.log(`Type: ${typeString}`);
      }
    }
  });
}

// Get diagnostics
const diagnostics = ts.getPreEmitDiagnostics(program);

diagnostics.forEach(diagnostic => {
  const message = ts.flattenDiagnosticMessageText(diagnostic.messageText, '\n');
  
  if (diagnostic.file && diagnostic.start !== undefined) {
    const { line, character } = 
      diagnostic.file.getLineAndCharacterOfPosition(diagnostic.start);
    console.log(`${diagnostic.file.fileName} (${line + 1},${character + 1}): ${message}`);
  } else {
    console.log(message);
  }
});
```

---

# 23. Additional Interview Questions

## 23.1 Scenario-Based Questions

**Q1: How would you type a function that accepts either a string or an array of strings, and always returns an array?**

```typescript
function toArray<T extends string | string[]>(input: T): string[] {
  return Array.isArray(input) ? input : [input];
}

// Or with overloads for better inference
function toArray(input: string): string[];
function toArray(input: string[]): string[];
function toArray(input: string | string[]): string[] {
  return Array.isArray(input) ? input : [input];
}
```

---

**Q2: How would you type a deep merge function?**

```typescript
type DeepMerge<T, U> = {
  [K in keyof T | keyof U]: K extends keyof U
    ? K extends keyof T
      ? T[K] extends object
        ? U[K] extends object
          ? DeepMerge<T[K], U[K]>
          : U[K]
        : U[K]
      : U[K]
    : K extends keyof T
    ? T[K]
    : never;
};

function deepMerge<T extends object, U extends object>(
  target: T,
  source: U
): DeepMerge<T, U> {
  const result = { ...target } as any;
  
  for (const key of Object.keys(source)) {
    const targetVal = (target as any)[key];
    const sourceVal = (source as any)[key];
    
    if (
      typeof targetVal === 'object' && 
      typeof sourceVal === 'object' &&
      !Array.isArray(targetVal) &&
      !Array.isArray(sourceVal)
    ) {
      result[key] = deepMerge(targetVal, sourceVal);
    } else {
      result[key] = sourceVal;
    }
  }
  
  return result;
}
```

---

**Q3: How do you type a function that returns different types based on a parameter?**

```typescript
// Function overloads
function parse(input: string, asNumber: true): number;
function parse(input: string, asNumber: false): string;
function parse(input: string, asNumber: boolean): number | string {
  return asNumber ? parseFloat(input) : input;
}

const num = parse("42", true);    // number
const str = parse("hello", false); // string

// Conditional return type
function getValue<T extends boolean>(
  flag: T
): T extends true ? number : string {
  return (flag ? 42 : "hello") as any;
}

const result1 = getValue(true);   // number
const result2 = getValue(false);  // string
```

---

**Q4: How would you implement type-safe localStorage?**

```typescript
interface StorageSchema {
  user: User;
  theme: 'light' | 'dark';
  preferences: {
    notifications: boolean;
    language: string;
  };
}

class TypedStorage<T extends Record<string, unknown>> {
  constructor(private storage: Storage = localStorage) {}
  
  get<K extends keyof T>(key: K): T[K] | null {
    const item = this.storage.getItem(key as string);
    return item ? JSON.parse(item) : null;
  }
  
  set<K extends keyof T>(key: K, value: T[K]): void {
    this.storage.setItem(key as string, JSON.stringify(value));
  }
  
  remove<K extends keyof T>(key: K): void {
    this.storage.removeItem(key as string);
  }
  
  clear(): void {
    this.storage.clear();
  }
}

const storage = new TypedStorage<StorageSchema>();

storage.set('theme', 'dark');  // OK
storage.set('theme', 'blue');  // Error: Type '"blue"' is not assignable

const theme = storage.get('theme');  // 'light' | 'dark' | null
```

---

**Q5: How do you type a function that works with array methods but preserves tuple types?**

```typescript
// Problem: Array methods widen tuple types
const tuple = [1, 'two', true] as const;
const mapped = tuple.map(x => x);  // (string | number | boolean)[]

// Solution: Use mapped types with tuples
type MapTuple<T extends readonly any[], F extends (x: any) => any> = {
  [K in keyof T]: F extends (x: T[K]) => infer R ? R : never;
};

function mapTuple<T extends readonly any[], F extends (x: T[number]) => any>(
  tuple: T,
  fn: F
): MapTuple<T, F> {
  return tuple.map(fn) as any;
}

const result = mapTuple([1, 'two', true] as const, x => typeof x);
// ['number', 'string', 'boolean']
```

---

*End of Part 2: TypeScript Mastery - Complete Interview Guide*

> **Total Coverage:** 6000+ lines of comprehensive TypeScript content including:
> - Core type system and fundamentals
> - Advanced types (conditional, mapped, template literals)
> - Generics complete guide
> - All utility types with implementations
> - Type guards and narrowing
> - Modules and declaration files
> - Classes and OOP patterns
> - Decorators
> - React + TypeScript integration
> - Advanced patterns (brand types, builders, event emitters)
> - Real-world patterns (API responses, forms, state management)
> - Error handling patterns (Result, Option, Either)
> - Performance optimization tips
> - Testing TypeScript code
> - 60+ interview questions with detailed answers
> - 40+ coding challenges with solutions

---

# 24. TypeScript with Modern Frameworks

## 24.1 TypeScript with Next.js

```typescript
// pages/api/users/[id].ts - API Route with TypeScript
import type { NextApiRequest, NextApiResponse } from 'next';

interface User {
  id: string;
  name: string;
  email: string;
}

type ResponseData = 
  | { user: User }
  | { error: string };

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  const { id } = req.query;
  
  if (typeof id !== 'string') {
    return res.status(400).json({ error: 'Invalid ID' });
  }
  
  // Fetch user from database
  const user: User = {
    id,
    name: 'Jayesh',
    email: 'jayesh@example.com'
  };
  
  res.status(200).json({ user });
}

// pages/users/[id].tsx - Page with TypeScript
import type { GetServerSideProps, InferGetServerSidePropsType } from 'next';

interface UserPageProps {
  user: User;
}

export const getServerSideProps: GetServerSideProps<UserPageProps> = async ({ params }) => {
  const id = params?.id as string;
  
  const res = await fetch(`${process.env.API_URL}/users/${id}`);
  const user: User = await res.json();
  
  return {
    props: { user }
  };
};

export default function UserPage({ 
  user 
}: InferGetServerSidePropsType<typeof getServerSideProps>) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// app/users/[id]/page.tsx - App Router with TypeScript
import { notFound } from 'next/navigation';

interface PageProps {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

async function getUser(id: string): Promise<User | null> {
  const res = await fetch(`${process.env.API_URL}/users/${id}`, {
    cache: 'no-store'
  });
  
  if (!res.ok) return null;
  return res.json();
}

export default async function UserPage({ params }: PageProps) {
  const user = await getUser(params.id);
  
  if (!user) {
    notFound();
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Server Actions with TypeScript
'use server';

import { revalidatePath } from 'next/cache';

interface CreateUserFormData {
  name: string;
  email: string;
}

export async function createUser(formData: FormData): Promise<{ success: boolean; user?: User; error?: string }> {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;
  
  if (!name || !email) {
    return { success: false, error: 'Name and email are required' };
  }
  
  const user: User = {
    id: crypto.randomUUID(),
    name,
    email
  };
  
  // Save to database...
  
  revalidatePath('/users');
  
  return { success: true, user };
}
```

## 24.2 TypeScript with Zustand

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Define state interface
interface UserState {
  user: User | null;
  users: User[];
  isLoading: boolean;
  error: string | null;
}

// Define actions interface
interface UserActions {
  setUser: (user: User | null) => void;
  addUser: (user: User) => void;
  removeUser: (id: string) => void;
  updateUser: (id: string, updates: Partial<User>) => void;
  fetchUsers: () => Promise<void>;
  reset: () => void;
}

// Combined store type
type UserStore = UserState & UserActions;

// Initial state
const initialState: UserState = {
  user: null,
  users: [],
  isLoading: false,
  error: null
};

// Create store with middleware
export const useUserStore = create<UserStore>()(
  persist(
    immer((set, get) => ({
      ...initialState,
      
      setUser: (user) => set((state) => {
        state.user = user;
      }),
      
      addUser: (user) => set((state) => {
        state.users.push(user);
      }),
      
      removeUser: (id) => set((state) => {
        state.users = state.users.filter(u => u.id !== id);
      }),
      
      updateUser: (id, updates) => set((state) => {
        const index = state.users.findIndex(u => u.id === id);
        if (index !== -1) {
          state.users[index] = { ...state.users[index], ...updates };
        }
      }),
      
      fetchUsers: async () => {
        set((state) => { state.isLoading = true; state.error = null; });
        
        try {
          const response = await fetch('/api/users');
          const users: User[] = await response.json();
          
          set((state) => {
            state.users = users;
            state.isLoading = false;
          });
        } catch (error) {
          set((state) => {
            state.error = error instanceof Error ? error.message : 'Failed to fetch users';
            state.isLoading = false;
          });
        }
      },
      
      reset: () => set(initialState)
    })),
    {
      name: 'user-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ user: state.user })  // Only persist user
    }
  )
);

// Selectors
export const selectCurrentUser = (state: UserStore) => state.user;
export const selectAllUsers = (state: UserStore) => state.users;
export const selectIsLoading = (state: UserStore) => state.isLoading;

// Usage in component
function UserList() {
  const users = useUserStore(selectAllUsers);
  const isLoading = useUserStore(selectIsLoading);
  const fetchUsers = useUserStore((state) => state.fetchUsers);
  
  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 24.3 TypeScript with TanStack Query

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// API functions with types
async function fetchUsers(): Promise<User[]> {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('Failed to fetch users');
  return response.json();
}

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('Failed to fetch user');
  return response.json();
}

async function createUser(data: Omit<User, 'id'>): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  if (!response.ok) throw new Error('Failed to create user');
  return response.json();
}

async function updateUser({ id, ...data }: Partial<User> & { id: string }): Promise<User> {
  const response = await fetch(`/api/users/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  if (!response.ok) throw new Error('Failed to update user');
  return response.json();
}

async function deleteUser(id: string): Promise<void> {
  const response = await fetch(`/api/users/${id}`, { method: 'DELETE' });
  if (!response.ok) throw new Error('Failed to delete user');
}

// Query keys
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: string) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

// Hooks
function useUsers() {
  return useQuery({
    queryKey: userKeys.lists(),
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000,  // 5 minutes
  });
}

function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => fetchUser(id),
    enabled: !!id,
  });
}

function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createUser,
    onSuccess: (newUser) => {
      // Update the list cache
      queryClient.setQueryData<User[]>(userKeys.lists(), (old = []) => 
        [...old, newUser]
      );
    },
    onError: (error) => {
      console.error('Failed to create user:', error);
    }
  });
}

function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateUser,
    onMutate: async (updatedUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: userKeys.detail(updatedUser.id) });
      
      // Snapshot the previous value
      const previousUser = queryClient.getQueryData<User>(userKeys.detail(updatedUser.id));
      
      // Optimistically update
      queryClient.setQueryData<User>(userKeys.detail(updatedUser.id), (old) => 
        old ? { ...old, ...updatedUser } : undefined
      );
      
      return { previousUser };
    },
    onError: (err, updatedUser, context) => {
      // Rollback on error
      if (context?.previousUser) {
        queryClient.setQueryData(userKeys.detail(updatedUser.id), context.previousUser);
      }
    },
    onSettled: (data, error, variables) => {
      // Invalidate to refetch
      queryClient.invalidateQueries({ queryKey: userKeys.detail(variables.id) });
    }
  });
}

function useDeleteUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: deleteUser,
    onSuccess: (_, deletedId) => {
      // Remove from list cache
      queryClient.setQueryData<User[]>(userKeys.lists(), (old = []) =>
        old.filter(user => user.id !== deletedId)
      );
      // Remove detail cache
      queryClient.removeQueries({ queryKey: userKeys.detail(deletedId) });
    }
  });
}

// Usage
function UserManagement() {
  const { data: users, isLoading, error } = useUsers();
  const createMutation = useCreateUser();
  const deleteMutation = useDeleteUser();
  
  const handleCreate = async () => {
    await createMutation.mutateAsync({ name: 'New User', email: 'new@example.com' });
  };
  
  const handleDelete = async (id: string) => {
    await deleteMutation.mutateAsync(id);
  };
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <button onClick={handleCreate} disabled={createMutation.isPending}>
        Add User
      </button>
      <ul>
        {users?.map(user => (
          <li key={user.id}>
            {user.name}
            <button 
              onClick={() => handleDelete(user.id)}
              disabled={deleteMutation.isPending}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

# 25. Final TypeScript Tips & Tricks

## 25.1 Const Assertions

```typescript
// Without const assertion
const config = {
  endpoint: '/api',
  timeout: 5000
};
// Type: { endpoint: string; timeout: number }

// With const assertion
const config2 = {
  endpoint: '/api',
  timeout: 5000
} as const;
// Type: { readonly endpoint: "/api"; readonly timeout: 5000 }

// Tuple with const
const tuple = [1, 2, 3] as const;
// Type: readonly [1, 2, 3] (not number[])

// Deriving types from const values
const ACTIONS = ['create', 'read', 'update', 'delete'] as const;
type Action = typeof ACTIONS[number];  // 'create' | 'read' | 'update' | 'delete'

const STATUS_CODES = {
  OK: 200,
  NOT_FOUND: 404,
  ERROR: 500
} as const;
type StatusCode = typeof STATUS_CODES[keyof typeof STATUS_CODES];  // 200 | 404 | 500
```

## 25.2 Satisfies Operator (TypeScript 4.9+)

```typescript
// Problem: Using 'as' widens the type
const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  blue: [0, 0, 255]
} as Record<string, string | number[]>;

palette.red.map(x => x);  // Error: Property 'map' does not exist

// Solution: satisfies preserves the narrower type
const palette2 = {
  red: [255, 0, 0],
  green: '#00ff00',
  blue: [0, 0, 255]
} satisfies Record<string, string | number[]>;

palette2.red.map(x => x);  // OK! TypeScript knows red is number[]
palette2.green.toUpperCase();  // OK! TypeScript knows green is string
```

## 25.3 Template Literal Type Inference

```typescript
// Extract route parameters
type ExtractParams<T extends string> = 
  T extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractParams<Rest>]: string }
    : T extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {};

type Route = '/users/:userId/posts/:postId';
type Params = ExtractParams<Route>;
// { userId: string; postId: string }

// Type-safe event names
type EventName<T extends string> = `on${Capitalize<T>}`;

type MouseEvents = EventName<'click' | 'mousedown' | 'mouseup'>;
// 'onClick' | 'onMousedown' | 'onMouseup'
```

## 25.4 Using 'declare' for Type-Only Imports

```typescript
// When you only need types, use 'declare' or 'import type'
declare const window: Window & { customProperty: string };

// Type-only imports (erased at runtime)
import type { User } from './types';
import type { FC } from 'react';

// Mixed imports
import React, { type ReactNode } from 'react';
```

## 25.5 Module Augmentation Tips

```typescript
// Extend Window interface
declare global {
  interface Window {
    analytics: AnalyticsClient;
    ENV: {
      API_URL: string;
      DEBUG: boolean;
    };
  }
}

// Extend Express Request
declare module 'express-serve-static-core' {
  interface Request {
    user?: User;
    session?: Session;
  }
}

// Extend React's intrinsic elements
declare module 'react' {
  namespace JSX {
    interface IntrinsicElements {
      'custom-element': CustomElementAttributes;
    }
  }
}
```

---

## 🎯 Final Study Checklist

### Core Concepts ✓
- [ ] Primitive types and type annotations
- [ ] Union and intersection types
- [ ] Type aliases vs interfaces
- [ ] Literal types and const assertions
- [ ] Type inference and widening/narrowing

### Advanced Types ✓
- [ ] Conditional types with `extends`
- [ ] Mapped types with `in keyof`
- [ ] Template literal types
- [ ] `infer` keyword for type extraction
- [ ] Index types with `keyof` and indexed access

### Generics ✓
- [ ] Generic functions, classes, and interfaces
- [ ] Generic constraints with `extends`
- [ ] Default type parameters
- [ ] Generic utility patterns

### Type Guards ✓
- [ ] `typeof` and `instanceof` guards
- [ ] `in` operator guard
- [ ] Custom type predicates with `is`
- [ ] Discriminated unions
- [ ] Assertion functions with `asserts`

### Utility Types ✓
- [ ] `Partial`, `Required`, `Readonly`
- [ ] `Pick`, `Omit`, `Record`
- [ ] `Exclude`, `Extract`, `NonNullable`
- [ ] `ReturnType`, `Parameters`, `Awaited`
- [ ] Custom utility type implementations

### React + TypeScript ✓
- [ ] Typing function components
- [ ] Typing hooks (useState, useEffect, useRef, etc.)
- [ ] Event handler types
- [ ] Context and Providers
- [ ] Custom hooks with generics

### Best Practices ✓
- [ ] tsconfig.json configuration
- [ ] Avoid `any`, prefer `unknown`
- [ ] Use strict mode
- [ ] Type narrowing over assertions
- [ ] Readable and maintainable types

---

*End of Part 2: TypeScript Mastery - Complete Interview Guide*

*Continue to Part 3 for React Deep Dive...*

