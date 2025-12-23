# Part 2: TypeScript Mastery

> **Everything you need to know about TypeScript for interviews**

---

## 2.1 Why TypeScript?

TypeScript is a **superset of JavaScript** that adds static type checking. It compiles to plain JavaScript.

**Benefits:**
- ✅ Catch errors at compile time, not runtime
- ✅ Better IDE support (autocomplete, refactoring)
- ✅ Self-documenting code
- ✅ Easier to maintain large codebases

**CodeHealth uses TypeScript everywhere** - see `tsconfig.json`:
```json
{
  "compilerOptions": {
    "strict": true,        // Enable all strict type-checking options
    "target": "ES2017",    // Output JavaScript version
    "jsx": "preserve"      // For Next.js/React
  }
}
```

---

## 2.2 Basic Types

```typescript
// Primitives
let name: string = "Jayesh";
let age: number = 25;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"]; // Generic syntax

// Tuple (fixed-length array with specific types)
let user: [string, number] = ["Jayesh", 25];

// Enum
enum Status {
  Pending = "PENDING",
  Active = "ACTIVE",
  Completed = "COMPLETED"
}
let currentStatus: Status = Status.Active;

// Any (avoid if possible - defeats the purpose of TypeScript)
let anything: any = "hello";
anything = 42; // No error, but no type safety

// Unknown (safer than any)
let userInput: unknown = "hello";
// Must check type before using:
if (typeof userInput === "string") {
  console.log(userInput.toUpperCase()); // OK
}

// Void (for functions that don't return)
function logMessage(msg: string): void {
  console.log(msg);
}

// Never (for functions that never return)
function throwError(message: string): never {
  throw new Error(message);
}
```

---

## 2.3 Interfaces vs Types

Both define the shape of objects, but have key differences.

### Interface (Best for Object Shapes)
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number; // Optional property
  readonly createdAt: Date; // Cannot be modified after creation
}

// Extending interfaces
interface Admin extends User {
  permissions: string[];
}

// Declaration Merging (unique to interfaces)
interface User {
  phone?: string; // This MERGES with the above User interface
}
```

### Type Alias (More Powerful)
```typescript
// Can define primitives, unions, tuples
type ID = string | number;
type Coordinates = [number, number];

// Object types
type User = {
  id: string;
  name: string;
};

// Intersection types (combine multiple types)
type Admin = User & {
  permissions: string[];
};

// Cannot do declaration merging
type User = { phone: string }; // ❌ Error: Duplicate identifier
```

### When to Use Which?
- **Interface**: For object shapes, especially when you need declaration merging or `extends`
- **Type**: For unions, intersections, primitives, tuples, or complex type manipulations

---

## 2.4 Generics (CRITICAL for Interviews)

Generics allow you to create reusable components that work with multiple types.

### Basic Generic Function
```typescript
// Without generics (loses type information)
function identity(value: any): any {
  return value;
}

// With generics (preserves type information)
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);      // T = number
const str = identity("hello");         // T inferred as string
```

### Generic Interfaces
```typescript
// API Response wrapper used in CodeHealth
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  error?: string;
}

// Usage
interface User {
  id: string;
  name: string;
}

const response: ApiResponse<User[]> = {
  success: true,
  data: [{ id: "1", name: "Jayesh" }]
};
```

### Generic Constraints
```typescript
// T must have a 'length' property
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

getLength("hello");     // ✅ OK - string has length
getLength([1, 2, 3]);   // ✅ OK - array has length
getLength(123);         // ❌ Error - number has no length
```

### Multiple Type Parameters
```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = merge({ name: "Jayesh" }, { age: 25 });
// result type: { name: string } & { age: number }
```

---

## 2.5 Utility Types (Used Extensively in CodeHealth)

TypeScript provides built-in utility types for common transformations.

### Partial<T>
Makes all properties optional.
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

// For updating a user (only some fields)
function updateUser(id: string, updates: Partial<User>) {
  // updates can have any subset of User properties
}

updateUser("1", { name: "New Name" }); // ✅ OK
```

### Required<T>
Makes all properties required (opposite of Partial).
```typescript
interface Config {
  host?: string;
  port?: number;
}

const fullConfig: Required<Config> = {
  host: "localhost", // Required now
  port: 3000         // Required now
};
```

### Pick<T, Keys>
Select specific properties from a type.
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

// For public display (exclude password)
type PublicUser = Pick<User, "id" | "name" | "email">;
// { id: string; name: string; email: string }
```

### Omit<T, Keys>
Remove specific properties from a type.
```typescript
// Create user (no id yet - generated by DB)
type CreateUserInput = Omit<User, "id">;
// { name: string; email: string; password: string }
```

### Record<Keys, Type>
Create an object type with specific keys and value types.
```typescript
// Used in teamStore.ts for caching
interface TeamStore {
  teamMembersLoaded: Record<string, boolean>;
  // { [teamId: string]: boolean }
}

const loaded: Record<string, boolean> = {
  "team-1": true,
  "team-2": false
};
```

### ReturnType<T>
Extract the return type of a function.
```typescript
function getUser() {
  return { id: "1", name: "Jayesh" };
}

type UserType = ReturnType<typeof getUser>;
// { id: string; name: string }
```

### Parameters<T>
Extract parameter types as a tuple.
```typescript
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age}`;
}

type GreetParams = Parameters<typeof greet>;
// [string, number]
```

---

## 2.6 Discriminated Unions (Advanced Pattern)

A pattern to handle different states safely.

```typescript
// State can be one of these shapes
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

// Usage in component
function renderData(state: RequestState<User[]>) {
  switch (state.status) {
    case "idle":
      return <p>Ready to fetch</p>;
    case "loading":
      return <Spinner />;
    case "success":
      // TypeScript knows state.data exists here!
      return <UserList users={state.data} />;
    case "error":
      // TypeScript knows state.error exists here!
      return <ErrorMessage message={state.error} />;
  }
}
```

---

## 2.7 Type Guards

Functions that narrow down types at runtime.

```typescript
// typeof guard
function processValue(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is string here
    return value.toUpperCase();
  }
  // TypeScript knows value is number here
  return value.toFixed(2);
}

// instanceof guard
function logError(error: unknown) {
  if (error instanceof Error) {
    console.log(error.message); // TypeScript knows it's Error
  }
}

// Custom type guard
interface Dog { bark(): void; }
interface Cat { meow(): void; }

function isDog(animal: Dog | Cat): animal is Dog {
  return (animal as Dog).bark !== undefined;
}

function makeSound(animal: Dog | Cat) {
  if (isDog(animal)) {
    animal.bark(); // TypeScript knows it's Dog
  } else {
    animal.meow(); // TypeScript knows it's Cat
  }
}
```

---

## 2.8 Mapped Types

Transform existing types into new types.

```typescript
// Make all properties optional
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Make all properties readonly
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Make all properties nullable
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface User {
  id: string;
  name: string;
}

type NullableUser = Nullable<User>;
// { id: string | null; name: string | null }
```

---

## 2.9 Conditional Types

Types that depend on conditions.

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Extracting types
type ArrayElementType<T> = T extends (infer U)[] ? U : never;

type NumArray = ArrayElementType<number[]>;  // number
type StrArray = ArrayElementType<string[]>;  // string

// Built-in: Extract and Exclude
type Union = "a" | "b" | "c";
type Extracted = Extract<Union, "a" | "b">;  // "a" | "b"
type Excluded = Exclude<Union, "a">;          // "b" | "c"
```

---

## 🎯 Part 2 Interview Questions

1. **What is the difference between interface and type?**
2. **Explain generics with an example.**
3. **What is the difference between `any` and `unknown`?**
4. **What are utility types? Name 5 and explain them.**
5. **What is a discriminated union?**
6. **How do type guards work?**
7. **What is the `keyof` operator?**
8. **Explain mapped types.**
9. **What is the difference between `readonly` and `const`?**
10. **How do you handle null/undefined in TypeScript?**

---

*Continue to Part 3 for React.js Deep Dive...*
