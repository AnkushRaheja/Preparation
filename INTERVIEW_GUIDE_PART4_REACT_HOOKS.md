# 📚 Part 4: React Hooks Complete Guide - Interview Mastery

> **The Ultimate React Hooks Interview Preparation Guide**
> 
> This comprehensive guide covers everything you need to master React Hooks for technical interviews at any level - from understanding core concepts to building advanced custom hooks.

---

## 📋 Table of Contents

1. [Introduction to Hooks](#1-introduction-to-hooks)
2. [useState Deep Dive](#2-usestate-deep-dive)
3. [useEffect Complete Guide](#3-useeffect-complete-guide)
4. [useContext Mastery](#4-usecontext-mastery)
5. [useReducer Advanced Patterns](#5-usereducer-advanced-patterns)
6. [useRef and useImperativeHandle](#6-useref-and-useimperativehandle)
7. [useMemo and useCallback](#7-usememo-and-usecallback)
8. [useLayoutEffect](#8-uselayouteffect)
9. [React 18+ Hooks](#9-react-18-hooks)
10. [Custom Hooks Patterns](#10-custom-hooks-patterns)
11. [Advanced Hook Patterns](#11-advanced-hook-patterns)
12. [Testing Hooks](#12-testing-hooks)
13. [Common Mistakes and Solutions](#13-common-mistakes-and-solutions)
14. [Interview Questions](#14-interview-questions)
15. [Coding Challenges](#15-coding-challenges)

---

# 1. Introduction to Hooks

## 1.1 What Are Hooks?

Hooks are functions that let you "hook into" React state and lifecycle features from function components. They were introduced in React 16.8 to solve several problems:

### Problems Before Hooks

```jsx
// 1. Reusing stateful logic was hard (needed HOCs or render props)
// Before: Complex wrapper pattern
const EnhancedComponent = withAuth(withTheme(withAnalytics(MyComponent)));

// 2. Complex components were hard to understand
// Before: Related logic split across lifecycle methods
class UserComponent extends Component {
  componentDidMount() {
    this.fetchUser();          // User fetch
    this.subscribeToChat();    // Chat subscription
    this.startTimer();         // Timer
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();        // User fetch (again!)
    }
  }
  
  componentWillUnmount() {
    this.unsubscribeFromChat(); // Chat cleanup
    this.stopTimer();           // Timer cleanup
  }
}

// 3. Classes were confusing (this binding, complex syntax)
class MyComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this); // Annoying!
  }
  // ...
}
```

### Solution With Hooks

```jsx
// 1. Reusing stateful logic with custom hooks
function MyComponent() {
  const user = useAuth();
  const theme = useTheme();
  const analytics = useAnalytics();
  // ...
}

// 2. Related logic grouped together
function UserComponent({ userId }) {
  // User fetching - all in one place
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // Chat subscription - all in one place
  useEffect(() => {
    const unsubscribe = subscribeToChat(userId);
    return () => unsubscribe();
  }, [userId]);
  
  // Timer - all in one place
  useEffect(() => {
    const timer = setInterval(tick, 1000);
    return () => clearInterval(timer);
  }, []);
}

// 3. No class syntax needed
function MyComponent() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

## 1.2 Rules of Hooks

### Rule 1: Only Call Hooks at the Top Level

```jsx
// ❌ Don't call hooks inside conditions
function BadComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // ❌ Conditional
  }
  
  for (let i = 0; i < 5; i++) {
    useEffect(() => {}); // ❌ Inside loop
  }
  
  const handleClick = () => {
    const [value, setValue] = useState(0); // ❌ Inside function
  };
}

// ✅ Always call hooks at the top level
function GoodComponent({ isLoggedIn }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    if (isLoggedIn) {
      // Condition inside hook is fine
      fetchUser().then(setUser);
    }
  }, [isLoggedIn]);
  
  return isLoggedIn ? <div>{user?.name}</div> : <Login />;
}
```

### Why This Rule Exists

React relies on the order of hook calls to associate state with the correct hook:

```jsx
function Form() {
  // React tracks: "First hook call = name state"
  const [name, setName] = useState('');
  // React tracks: "Second hook call = email state"
  const [email, setEmail] = useState('');
  // React tracks: "Third hook call = this effect"
  useEffect(() => { /* ... */ });
}

// If we called hooks conditionally, the order could change:
// First render: [name, email, effect]
// Second render: [email, effect] ← Broken! React gets confused
```

### Rule 2: Only Call Hooks from React Functions

```jsx
// ✅ Call hooks from function components
function MyComponent() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// ✅ Call hooks from custom hooks
function useCounter(initial) {
  const [count, setCount] = useState(initial);
  const increment = useCallback(() => setCount(c => c + 1), []);
  return { count, increment };
}

// ❌ Don't call hooks from regular functions
function fetchData() {
  const [data, setData] = useState(null); // ❌ Error!
}

// ❌ Don't call hooks from class components
class MyClass extends Component {
  render() {
    const [count] = useState(0); // ❌ Error!
  }
}
```

## 1.3 Hooks at a Glance

| Hook | Purpose | Returns |
|------|---------|---------|
| `useState` | Local state | `[value, setValue]` |
| `useEffect` | Side effects | `undefined` |
| `useContext` | Context consumption | Context value |
| `useReducer` | Complex state logic | `[state, dispatch]` |
| `useCallback` | Memoize function | Memoized callback |
| `useMemo` | Memoize value | Memoized value |
| `useRef` | Mutable ref object | `{ current: value }` |
| `useImperativeHandle` | Customize ref | `undefined` |
| `useLayoutEffect` | Sync effects | `undefined` |
| `useDebugValue` | DevTools label | `undefined` |
| `useId` | Unique ID | String ID |
| `useTransition` | Non-blocking updates | `[isPending, startTransition]` |
| `useDeferredValue` | Deferred value | Deferred value |
| `useSyncExternalStore` | External store | Store state |
| `useInsertionEffect` | CSS-in-JS | `undefined` |

---

# 2. useState Deep Dive

## 2.1 Basic Usage

```jsx
import { useState } from 'react';

function Counter() {
  // Syntax: const [state, setState] = useState(initialValue);
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

## 2.2 Functional Updates

When new state depends on previous state, use functional updates:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // ❌ Problem: Multiple calls use same stale value
  const incrementThreeTimesBad = () => {
    setCount(count + 1); // count is 0, so 0 + 1 = 1
    setCount(count + 1); // count is still 0, so 0 + 1 = 1
    setCount(count + 1); // count is still 0, so 0 + 1 = 1
    // Result: count becomes 1, not 3!
  };
  
  // ✅ Solution: Functional updates always get latest value
  const incrementThreeTimesGood = () => {
    setCount(prev => prev + 1); // 0 + 1 = 1
    setCount(prev => prev + 1); // 1 + 1 = 2
    setCount(prev => prev + 1); // 2 + 1 = 3
    // Result: count becomes 3!
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThreeTimesGood}>+3</button>
    </div>
  );
}
```

### When to Use Functional Updates

```jsx
// ✅ Use functional update when depending on previous state
setCount(prev => prev + 1);
setItems(prev => [...prev, newItem]);
setUser(prev => ({ ...prev, name: 'New Name' }));

// ✅ OK to use direct value when not depending on previous
setCount(0);                    // Reset to known value
setItems(newItemsArray);        // Replace entirely
setUser({ name: 'New User' }); // Replace entirely
```

## 2.3 Lazy Initialization

For expensive initial state computations, use a function:

```jsx
// ❌ Bad: computeExpensiveValue runs on EVERY render
function BadComponent() {
  const [state, setState] = useState(computeExpensiveValue(props));
  // computeExpensiveValue is called every render, even though
  // the result is only used on first render
}

// ✅ Good: computeExpensiveValue runs only on first render
function GoodComponent() {
  const [state, setState] = useState(() => computeExpensiveValue(props));
  // Function is only called once during initialization
}

// Practical examples:
function TodoApp() {
  // Parse from localStorage only once
  const [todos, setTodos] = useState(() => {
    const saved = localStorage.getItem('todos');
    return saved ? JSON.parse(saved) : [];
  });
  
  // Complex initial calculation only once
  const [data, setData] = useState(() => {
    return generateComplexInitialData();
  });
}
```

## 2.4 Object and Array State

### Object State

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0,
    address: {
      city: '',
      country: ''
    }
  });
  
  // ❌ Wrong: Mutating state directly
  const updateNameBad = (name) => {
    user.name = name; // Mutation!
    setUser(user);    // Same reference, won't trigger re-render
  };
  
  // ✅ Correct: Creating new object
  const updateName = (name) => {
    setUser(prev => ({
      ...prev,
      name: name
    }));
  };
  
  // ✅ Updating nested property
  const updateCity = (city) => {
    setUser(prev => ({
      ...prev,
      address: {
        ...prev.address,
        city: city
      }
    }));
  };
  
  // ✅ Generic field updater
  const handleChange = (e) => {
    const { name, value } = e.target;
    setUser(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  return (
    <form>
      <input name="name" value={user.name} onChange={handleChange} />
      <input name="email" value={user.email} onChange={handleChange} />
    </form>
  );
}
```

### Array State

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: false },
    { id: 2, text: 'Build app', done: false }
  ]);
  
  // Add item
  const addTodo = (text) => {
    setTodos(prev => [
      ...prev,
      { id: Date.now(), text, done: false }
    ]);
  };
  
  // Remove item
  const removeTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  // Update item
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  // Update item at index
  const updateTodoText = (index, text) => {
    setTodos(prev => [
      ...prev.slice(0, index),
      { ...prev[index], text },
      ...prev.slice(index + 1)
    ]);
  };
  
  // Insert at position
  const insertTodo = (index, todo) => {
    setTodos(prev => [
      ...prev.slice(0, index),
      todo,
      ...prev.slice(index)
    ]);
  };
  
  // Reorder items
  const moveTodo = (fromIndex, toIndex) => {
    setTodos(prev => {
      const result = [...prev];
      const [removed] = result.splice(fromIndex, 1);
      result.splice(toIndex, 0, removed);
      return result;
    });
  };
  
  return (/* JSX */);
}
```

## 2.5 Multiple State Variables vs Single Object

```jsx
// Approach 1: Multiple state variables
function MultipleStateVariables() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  // ✅ Pros: Simple, each updates independently
  // ✅ Pros: Easy to extract into custom hooks
  // ❌ Cons: Many useState calls for complex forms
}

// Approach 2: Single object state
function SingleObjectState() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateField = (field, value) => {
    setForm(prev => ({ ...prev, [field]: value }));
  };
  
  // ✅ Pros: Single update for related data
  // ✅ Pros: Easy to reset all at once
  // ❌ Cons: Need spread operator for every update
}

// Recommendation:
// - Use multiple variables for independent values
// - Use single object for closely related values (forms)
// - Consider useReducer for complex state logic
```

## 2.6 State Batching (React 18+)

```jsx
function BatchingExample() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  // React 18: All updates are batched automatically
  const handleClick = () => {
    setCount(c => c + 1);
    setFlag(f => !f);
    // Only ONE re-render happens (batched)
  };
  
  // Even in async callbacks!
  const handleAsync = async () => {
    await fetch('/api/data');
    setCount(c => c + 1);
    setFlag(f => !f);
    // Still only ONE re-render (React 18+)
  };
  
  // Opt out of batching with flushSync
  const handleFlush = () => {
    import('react-dom').then(({ flushSync }) => {
      flushSync(() => {
        setCount(c => c + 1); // Re-renders immediately
      });
      flushSync(() => {
        setFlag(f => !f);     // Re-renders again
      });
    });
  };
}
```

## 2.7 TypeScript with useState

```tsx
import { useState } from 'react';

// Inferred types
const [count, setCount] = useState(0);           // number
const [name, setName] = useState('');            // string
const [active, setActive] = useState(false);     // boolean

// Explicit types for complex cases
interface User {
  id: number;
  name: string;
  email: string;
}

const [user, setUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);
const [status, setStatus] = useState<'idle' | 'loading' | 'error'>('idle');

// With initial value that doesn't match full type
interface Config {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
}

// Type tells TS what the full shape will be
const [config, setConfig] = useState<Config>({
  theme: 'light',
  language: 'en',
  notifications: true
});

// Union types for state machines
type Status = 
  | { type: 'idle' }
  | { type: 'loading' }
  | { type: 'success'; data: User[] }
  | { type: 'error'; message: string };

const [status, setStatus] = useState<Status>({ type: 'idle' });
```

---

# 3. useEffect Complete Guide

## 3.1 Understanding useEffect

useEffect is for **side effects** - operations that affect something outside the component:

- Data fetching
- Subscriptions
- DOM manipulation
- Timers
- Logging
- Local storage

```jsx
import { useEffect } from 'react';

function Example() {
  useEffect(() => {
    // This is the effect
    console.log('Effect ran');
    
    // This is the cleanup (optional)
    return () => {
      console.log('Cleanup ran');
    };
  }, [/* dependencies */]);
}
```

## 3.2 Dependency Array Explained

```jsx
// 1. No dependency array - runs after EVERY render
useEffect(() => {
  console.log('Runs after every render');
});

// 2. Empty array - runs only on mount (and cleanup on unmount)
useEffect(() => {
  console.log('Runs once on mount');
  return () => console.log('Runs on unmount');
}, []);

// 3. With dependencies - runs when dependencies change
useEffect(() => {
  console.log('Runs when userId changes');
  fetchUser(userId);
}, [userId]);

// 4. Multiple dependencies
useEffect(() => {
  console.log('Runs when userId OR postId changes');
  fetchUserPost(userId, postId);
}, [userId, postId]);
```

### How Dependency Comparison Works

```jsx
function Component({ user }) {
  const [count, setCount] = useState(0);
  
  // React compares dependencies with Object.is (shallow comparison)
  useEffect(() => {
    console.log('Effect ran');
  }, [user, count]);
  
  // For objects: Same reference = no re-run, new reference = re-run
  // { name: 'John' } !== { name: 'John' } (different references!)
  
  // For primitives: Same value = no re-run
  // 1 === 1 (true)
  // 'hello' === 'hello' (true)
}
```

## 3.3 Common Use Cases

### Data Fetching

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // Track if component is still mounted
    let isMounted = true;
    
    const fetchUser = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        
        // Only update state if still mounted
        if (isMounted) {
          setUser(data);
          setLoading(false);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
          setLoading(false);
        }
      }
    };
    
    fetchUser();
    
    // Cleanup: Prevent state updates after unmount
    return () => {
      isMounted = false;
    };
  }, [userId]);
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <div>{user?.name}</div>;
}
```

### With AbortController (Better Approach)

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`, {
          signal: controller.signal
        });
        const data = await response.json();
        setUser(data);
      } catch (err) {
        if (err.name === 'AbortError') {
          // Request was aborted, ignore
          return;
        }
        // Handle other errors
        console.error(err);
      }
    };
    
    fetchUser();
    
    return () => controller.abort();
  }, [userId]);
  
  return user ? <div>{user.name}</div> : <Spinner />;
}
```

### Event Listeners

```jsx
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup: Remove listener on unmount
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Empty deps = setup once, cleanup once
  
  return <div>{size.width} x {size.height}</div>;
}
```

### Subscriptions

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    // Subscribe to chat room
    const connection = createConnection(roomId);
    connection.connect();
    
    connection.on('message', (message) => {
      setMessages(prev => [...prev, message]);
    });
    
    // Cleanup: Disconnect when roomId changes or unmount
    return () => connection.disconnect();
  }, [roomId]);
  
  return (
    <ul>
      {messages.map(msg => <li key={msg.id}>{msg.text}</li>)}
    </ul>
  );
}
```

### Timers

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Cleanup: Clear interval on unmount
    return () => clearInterval(interval);
  }, []);
  
  return <div>Seconds: {seconds}</div>;
}

// Controllable timer
function ControllableTimer({ isRunning }) {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    if (!isRunning) return; // Don't start if not running
    
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, [isRunning]); // Re-run when isRunning changes
  
  return <div>Seconds: {seconds}</div>;
}
```

### Document Title

```jsx
function PageTitle({ title }) {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = title;
    
    // Restore previous title on unmount
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
  
  return null;
}

// Usage
function App() {
  return (
    <>
      <PageTitle title="My App - Home" />
      <HomePage />
    </>
  );
}
```

### Local Storage Sync

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Failed to save to localStorage:', error);
    }
  }, [key, value]);
  
  return [value, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

## 3.4 Effect Cleanup

Cleanup runs:
1. Before the effect runs again (when dependencies change)
2. When the component unmounts

```jsx
function EffectCleanupDemo({ userId }) {
  useEffect(() => {
    console.log(`Setting up for user ${userId}`);
    
    return () => {
      console.log(`Cleaning up for user ${userId}`);
    };
  }, [userId]);
}

// If userId changes from 1 to 2:
// 1. "Cleaning up for user 1"  <-- Cleanup runs first
// 2. "Setting up for user 2"   <-- Then new effect runs
```

### Why Cleanup Matters

```jsx
// ❌ Memory leak: No cleanup
function BadComponent({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    // If roomId changes, old connections stay open!
  }, [roomId]);
}

// ✅ Proper cleanup
function GoodComponent({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    
    return () => connection.disconnect(); // Clean up old connection
  }, [roomId]);
}
```

## 3.5 Avoiding Common Pitfalls

### Infinite Loops

```jsx
// ❌ Infinite loop: Object in dependencies
function Bad() {
  const [data, setData] = useState(null);
  const options = { method: 'GET' }; // New object every render!
  
  useEffect(() => {
    fetch('/api', options).then(r => r.json()).then(setData);
  }, [options]); // New reference every render = infinite loop!
}

// ✅ Fix: Move object inside effect
function Good() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const options = { method: 'GET' };
    fetch('/api', options).then(r => r.json()).then(setData);
  }, []); // No external dependencies
}

// ✅ Or use useMemo for the object
function AlsoGood() {
  const [data, setData] = useState(null);
  const options = useMemo(() => ({ method: 'GET' }), []);
  
  useEffect(() => {
    fetch('/api', options).then(r => r.json()).then(setData);
  }, [options]); // Stable reference
}
```

### Missing Dependencies

```jsx
// ❌ Missing dependency: Stale closure
function Stale({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []); // userId missing! Uses stale userId
  
  // If userId changes, this effect won't re-run
}

// ✅ Include all dependencies
function Fresh({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // Correct!
}
```

### Function Dependencies

```jsx
// ❌ Function in dependencies causes re-run every render
function Problem() {
  const fetchData = () => {
    // This function is recreated every render
  };
  
  useEffect(() => {
    fetchData();
  }, [fetchData]); // New reference every render!
}

// ✅ Solution 1: Move function inside effect
function Solution1() {
  useEffect(() => {
    const fetchData = () => {
      // Now it's inside the effect
    };
    fetchData();
  }, []); // No external function dependency
}

// ✅ Solution 2: useCallback for functions used elsewhere
function Solution2() {
  const fetchData = useCallback(() => {
    // Memoized function
  }, [/* deps */]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]); // Stable reference
}
```

## 3.6 useEffect vs Event Handlers

```jsx
// Event handler: Runs in response to user action
function Form() {
  const handleSubmit = (e) => {
    e.preventDefault();
    submitForm(data); // User clicked submit
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}

// Effect: Runs as a result of rendering
function UserProfile({ userId }) {
  useEffect(() => {
    fetchUser(userId); // Sync with external system
  }, [userId]);
}

// Rule of thumb:
// - Event handlers for user-initiated actions
// - Effects for synchronization with external systems
```

---

# 4. useContext Mastery

## 4.1 Basic Context Usage

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create context
const ThemeContext = createContext('light');

// 2. Create provider
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = {
    theme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light')
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Consume with useContext
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={toggleTheme}
      style={{ 
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
    >
      Toggle Theme
    </button>
  );
}

// 4. Wrap app with provider
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
      <ThemedButton />
    </ThemeProvider>
  );
}
```

## 4.2 Custom Hook for Context

```jsx
// ❌ Error prone: Can use context outside provider
function BadComponent() {
  const value = useContext(ThemeContext);
  // value might be undefined if not wrapped in provider
}

// ✅ Better: Custom hook with error handling
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
}

// Usage - clear error if misused
function ThemedButton() {
  const { theme, toggleTheme } = useTheme(); // Safe!
  return <button onClick={toggleTheme}>{theme}</button>;
}
```

## 4.3 TypeScript with Context

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Define types
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// Create context with undefined as default
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider
interface AuthProviderProps {
  children: ReactNode;
}

export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = async (email: string, password: string) => {
    const userData = await loginAPI(email, password);
    setUser(userData);
  };
  
  const logout = () => setUser(null);
  
  const value: AuthContextType = {
    user,
    isAuthenticated: !!user,
    login,
    logout
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook with type safety
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
}

// Usage - fully typed!
function Profile() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <p>{user?.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

## 4.4 Context Performance Optimization

### Problem: All Consumers Re-render

```jsx
// ❌ Problem: Any state change re-renders ALL consumers
function AppProviderBad({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  // New object every render - all consumers re-render!
  return (
    <AppContext.Provider value={{ 
      user, setUser, 
      theme, setTheme, 
      notifications, setNotifications 
    }}>
      {children}
    </AppContext.Provider>
  );
}
```

### Solution 1: Memoize Context Value

```jsx
function AppProviderBetter({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  // Memoize the value object
  const value = useMemo(() => ({
    user,
    setUser,
    theme,
    setTheme
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}
```

### Solution 2: Split Contexts

```jsx
// Split by domain/update frequency
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const NotificationContext = createContext([]);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <NotificationContext.Provider value={{ notifications, setNotifications }}>
          {children}
        </NotificationContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Now components only re-render when THEIR context changes
function ThemeToggle() {
  const { theme, setTheme } = useContext(ThemeContext);
  // Only re-renders when theme changes, not user or notifications
}
```

### Solution 3: Separate State and Dispatch

```jsx
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Components that only dispatch never re-render on state change
function AddButton() {
  const dispatch = useContext(DispatchContext);
  // This component never re-renders when state changes!
  return <button onClick={() => dispatch({ type: 'ADD' })}>Add</button>;
}

// Components that read state re-render as expected
function ItemList() {
  const state = useContext(StateContext);
  return <ul>{state.items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}
```

---

# 5. useReducer Advanced Patterns

## 5.1 Basic useReducer

```jsx
import { useReducer } from 'react';

// Reducer function: (state, action) => newState
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'SET':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>Set 10</button>
    </div>
  );
}
```

## 5.2 TypeScript with useReducer

```tsx
// Define state type
interface State {
  count: number;
  error: string | null;
  isLoading: boolean;
}

// Define action types (discriminated union)
type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET'; payload: number }
  | { type: 'LOADING' }
  | { type: 'ERROR'; payload: string }
  | { type: 'RESET' };

// Reducer with type safety
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET':
      return { ...state, count: action.payload }; // payload is typed as number
    case 'LOADING':
      return { ...state, isLoading: true, error: null };
    case 'ERROR':
      return { ...state, isLoading: false, error: action.payload }; // string
    case 'RESET':
      return { count: 0, error: null, isLoading: false };
    default:
      // Exhaustiveness check
      const _exhaustiveCheck: never = action;
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    error: null,
    isLoading: false
  });
  
  // TypeScript enforces correct action shapes
  dispatch({ type: 'INCREMENT' });           // ✅
  dispatch({ type: 'SET', payload: 10 });    // ✅
  dispatch({ type: 'SET', payload: 'ten' }); // ❌ Error: string not assignable to number
  dispatch({ type: 'UNKNOWN' });             // ❌ Error: not in Action type
}
```

## 5.3 Complex State Example: Todo App

```tsx
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface State {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  editingId: number | null;
}

type Action =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number }
  | { type: 'EDIT_TODO'; payload: { id: number; text: string } }
  | { type: 'SET_FILTER'; payload: State['filter'] }
  | { type: 'START_EDITING'; payload: number }
  | { type: 'STOP_EDITING' }
  | { type: 'CLEAR_COMPLETED' };

function todoReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false
          }
        ]
      };
      
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case 'EDIT_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        ),
        editingId: null
      };
      
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
      
    case 'START_EDITING':
      return { ...state, editingId: action.payload };
      
    case 'STOP_EDITING':
      return { ...state, editingId: null };
      
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all',
    editingId: null
  });
  
  const filteredTodos = useMemo(() => {
    switch (state.filter) {
      case 'active':
        return state.todos.filter(t => !t.completed);
      case 'completed':
        return state.todos.filter(t => t.completed);
      default:
        return state.todos;
    }
  }, [state.todos, state.filter]);
  
  return (/* JSX */);
}
```

## 5.4 Lazy Initialization

```jsx
// For expensive initial state computation
function init(initialCount) {
  // This function only runs once
  return {
    count: initialCount,
    history: [initialCount]
  };
}

function Counter({ initialCount = 0 }) {
  // Third argument is initializer function
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  
  // init(initialCount) is called only on first render
}

// Practical example: Load from localStorage
function init(key) {
  try {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : { items: [], filter: 'all' };
  } catch {
    return { items: [], filter: 'all' };
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, 'todos', init);
}
```

## 5.5 useState vs useReducer

```jsx
// Use useState when:
// - Simple state (primitives, simple objects)
// - Independent state updates
// - Few state transitions

function SimpleForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  return /* simple form */;
}

// Use useReducer when:
// - Complex state objects
// - Related state that updates together
// - State transitions follow specific rules
// - Need to test state logic separately

function ComplexForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const handleSubmit = async () => {
    dispatch({ type: 'SUBMIT_START' });
    try {
      await submitForm(state.values);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: error.message });
    }
  };
}
```

---

# 6. useRef and useImperativeHandle

## 6.1 useRef Basics

useRef returns a mutable object that persists across renders:

```jsx
import { useRef } from 'react';

function RefExample() {
  // Create a ref
  const countRef = useRef(0);
  const inputRef = useRef(null);
  
  // countRef.current can be mutated without causing re-render
  const handleClick = () => {
    countRef.current += 1;
    console.log('Clicked', countRef.current, 'times');
    // Component does NOT re-render!
  };
  
  // Access DOM element
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
      <button onClick={handleClick}>Click Me</button>
    </div>
  );
}
```

## 6.2 useRef Use Cases

### DOM Element Access

```jsx
function VideoPlayer() {
  const videoRef = useRef(null);
  
  const play = () => videoRef.current?.play();
  const pause = () => videoRef.current?.pause();
  const mute = () => {
    if (videoRef.current) {
      videoRef.current.muted = true;
    }
  };
  
  return (
    <>
      <video ref={videoRef} src="/video.mp4" />
      <button onClick={play}>Play</button>
      <button onClick={pause}>Pause</button>
      <button onClick={mute}>Mute</button>
    </>
  );
}
```

### Storing Previous Value

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current; // Returns previous value
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}, Previous: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

### Instance Variables (Survive Re-renders)

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    if (intervalRef.current) return; // Already running
    
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };
  
  const reset = () => {
    stop();
    setSeconds(0);
  };
  
  // Cleanup on unmount
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### Tracking Render Count

```jsx
function RenderCounter() {
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current += 1;
  });
  
  return <div>Rendered {renderCount.current} times</div>;
}
```

### Callback Ref Pattern

```jsx
function MeasuredComponent() {
  const [height, setHeight] = useState(0);
  
  // Callback ref: called when element mounts/unmounts
  const measuredRef = useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);
  
  return (
    <>
      <div ref={measuredRef}>Content that might change height</div>
      <p>Height: {height}px</p>
    </>
  );
}
```

## 6.3 forwardRef

Forward refs from parent to child components:

```jsx
import { forwardRef } from 'react';

// Child component forwards ref to input
const FancyInput = forwardRef(function FancyInput(props, ref) {
  return (
    <input
      ref={ref}
      className="fancy-input"
      placeholder={props.placeholder}
    />
  );
});

// Parent can access input directly
function Parent() {
  const inputRef = useRef(null);
  
  const focus = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <FancyInput ref={inputRef} placeholder="Enter text" />
      <button onClick={focus}>Focus</button>
    </>
  );
}
```

## 6.4 useImperativeHandle

Customize what's exposed to parent via ref:

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef(null);
  
  // Expose custom methods to parent
  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
    clear() {
      if (inputRef.current) {
        inputRef.current.value = '';
      }
    },
    getValue() {
      return inputRef.current?.value ?? '';
    },
    setValue(value) {
      if (inputRef.current) {
        inputRef.current.value = value;
      }
    },
    // You can expose anything, not just DOM methods
    validate() {
      const value = inputRef.current?.value ?? '';
      return value.length >= 3;
    }
  }), []); // Dependency array
  
  return <input ref={inputRef} {...props} />;
});

// Usage
function Parent() {
  const inputRef = useRef(null);
  
  const handleSubmit = () => {
    if (inputRef.current?.validate()) {
      const value = inputRef.current.getValue();
      console.log('Valid:', value);
      inputRef.current.clear();
    } else {
      console.log('Invalid!');
      inputRef.current.focus();
    }
  };
  
  return (
    <>
      <FancyInput ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

## 6.5 TypeScript with Refs

```tsx
import { useRef, forwardRef, useImperativeHandle, Ref } from 'react';

// DOM element ref
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);
const buttonRef = useRef<HTMLButtonElement>(null);

// Mutable value ref
const countRef = useRef<number>(0);
const timerRef = useRef<NodeJS.Timeout | null>(null);

// forwardRef with TypeScript
interface FancyInputProps {
  placeholder?: string;
  className?: string;
}

interface FancyInputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
}

const FancyInput = forwardRef<FancyInputHandle, FancyInputProps>(
  function FancyInput(props, ref) {
    const inputRef = useRef<HTMLInputElement>(null);
    
    useImperativeHandle(ref, () => ({
      focus() {
        inputRef.current?.focus();
      },
      clear() {
        if (inputRef.current) inputRef.current.value = '';
      },
      getValue() {
        return inputRef.current?.value ?? '';
      }
    }), []);
    
    return <input ref={inputRef} {...props} />;
  }
);

// Usage with proper typing
function Parent() {
  const inputRef = useRef<FancyInputHandle>(null);
  
  const handleClick = () => {
    inputRef.current?.focus(); // Typed!
    const value = inputRef.current?.getValue(); // string | undefined
  };
  
  return <FancyInput ref={inputRef} />;
}
```

---

# 7. useMemo and useCallback

## 7.1 useMemo - Memoizing Values

useMemo caches the result of a calculation between re-renders:

```jsx
import { useMemo } from 'react';

function ExpensiveComponent({ items, filter }) {
  // ❌ Without useMemo: Filters on EVERY render
  const filteredItems = items.filter(item => 
    item.name.includes(filter)
  );
  
  // ✅ With useMemo: Only recalculates when items or filter change
  const filteredItems = useMemo(() => {
    return items.filter(item => item.name.includes(filter));
  }, [items, filter]);
  
  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### When to Use useMemo

```jsx
// ✅ Use for expensive calculations
const sortedList = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);

// ✅ Use for object/array stability (prevent child re-renders)
const chartConfig = useMemo(() => ({
  data: processData(rawData),
  options: { animated: true }
}), [rawData]);

// ✅ Use for derived data
const statistics = useMemo(() => {
  return {
    total: items.length,
    completed: items.filter(i => i.done).length,
    percentage: (items.filter(i => i.done).length / items.length) * 100
  };
}, [items]);

// ❌ Don't use for simple calculations
const doubled = count * 2; // Just compute it directly

// ❌ Don't use if value changes every render anyway
const useless = useMemo(() => ({ timestamp: Date.now() }), []);
```

### useMemo with Complex Dependencies

```jsx
function DataGrid({ rows, columns, sortColumn, sortDirection }) {
  const sortedRows = useMemo(() => {
    if (!sortColumn) return rows;
    
    return [...rows].sort((a, b) => {
      const aVal = a[sortColumn];
      const bVal = b[sortColumn];
      const comparison = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
      return sortDirection === 'asc' ? comparison : -comparison;
    });
  }, [rows, sortColumn, sortDirection]);
  
  const visibleColumns = useMemo(() => 
    columns.filter(col => col.visible),
    [columns]
  );
  
  return (
    <table>
      <thead>
        <tr>
          {visibleColumns.map(col => (
            <th key={col.id}>{col.label}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedRows.map(row => (
          <tr key={row.id}>
            {visibleColumns.map(col => (
              <td key={col.id}>{row[col.id]}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## 7.2 useCallback - Memoizing Functions

useCallback caches a function definition between re-renders:

```jsx
import { useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // ❌ Without useCallback: New function every render
  const handleClick = () => {
    console.log('Clicked!');
  };
  
  // ✅ With useCallback: Same function reference
  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []);
  
  // With dependencies
  const handleIncrement = useCallback(() => {
    setCount(c => c + 1);
  }, []); // No deps needed - uses functional update
  
  const handleLog = useCallback(() => {
    console.log('Current count:', count);
  }, [count]); // Needs count in deps
  
  return (
    <div>
      <MemoizedChild onClick={handleClick} />
      <input value={text} onChange={e => setText(e.target.value)} />
    </div>
  );
}

// Child component with React.memo
const MemoizedChild = React.memo(function Child({ onClick }) {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});
```

### useCallback + React.memo Pattern

```jsx
// The most common useCallback pattern
function TodoApp() {
  const [todos, setTodos] = useState([]);
  
  // Memoized callbacks for memoized children
  const handleToggle = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  }, []);
  
  const handleDelete = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}

// Memoized to prevent re-renders
const TodoItem = React.memo(function TodoItem({ todo, onToggle, onDelete }) {
  console.log('TodoItem rendered:', todo.id);
  
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => onToggle(todo.id)}
      />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});
```

## 7.3 useMemo vs useCallback

```jsx
// They're essentially the same:
const memoizedValue = useMemo(() => computeValue(a, b), [a, b]);
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);

// useCallback(fn, deps) is equivalent to useMemo(() => fn, deps)

// Use useMemo for VALUES
const sortedItems = useMemo(() => items.sort(), [items]);

// Use useCallback for FUNCTIONS
const handleSort = useCallback(() => {
  setItems(prev => [...prev].sort());
}, []);
```

## 7.4 When NOT to Use

```jsx
// ❌ Don't use for cheap operations
function Component({ items }) {
  // This is fast, memoization adds overhead
  const length = useMemo(() => items.length, [items]);
  
  // Just do this:
  const length = items.length;
}

// ❌ Don't use if the value changes every render
function Component({ data }) {
  // If data changes every render, memoization is useless
  const processed = useMemo(() => process(data), [data]);
}

// ❌ Don't use without React.memo on children
function Parent() {
  // This useCallback is useless if Child isn't memoized
  const handleClick = useCallback(() => {}, []);
  
  // Child re-renders anyway because Parent re-renders
  return <Child onClick={handleClick} />;
}

function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
}
```

## 7.5 Performance Measurement

```jsx
function Component({ items }) {
  // Measure if memoization is worth it
  const sortedItems = useMemo(() => {
    console.time('sorting');
    const result = [...items].sort((a, b) => a.value - b.value);
    console.timeEnd('sorting');
    return result;
  }, [items]);
  
  // Use React DevTools Profiler to see actual render times
}
```

---

# 8. useLayoutEffect

## 8.1 useLayoutEffect vs useEffect

```jsx
// Timing comparison:
// 1. Component renders
// 2. React updates DOM
// 3. useLayoutEffect runs (SYNC, blocks paint)
// 4. Browser paints screen
// 5. useEffect runs (ASYNC, after paint)

function LayoutEffectDemo() {
  const [width, setWidth] = useState(0);
  const ref = useRef(null);
  
  // ❌ useEffect: May cause visual flicker
  useEffect(() => {
    // DOM is already painted - user might see old value briefly
    setWidth(ref.current.offsetWidth);
  }, []);
  
  // ✅ useLayoutEffect: Runs before paint
  useLayoutEffect(() => {
    // DOM is updated but not painted yet
    // Setting state here won't cause flicker
    setWidth(ref.current.offsetWidth);
  }, []);
  
  return <div ref={ref}>Width: {width}px</div>;
}
```

## 8.2 When to Use useLayoutEffect

```jsx
// ✅ DOM measurements
function Tooltip({ targetRef, children }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  
  useLayoutEffect(() => {
    if (!targetRef.current) return;
    
    const rect = targetRef.current.getBoundingClientRect();
    setPosition({
      top: rect.bottom + window.scrollY,
      left: rect.left + window.scrollX
    });
  }, [targetRef]);
  
  return (
    <div style={{ position: 'absolute', ...position }}>
      {children}
    </div>
  );
}

// ✅ Scroll position restoration
function ScrollRestoration({ scrollPosition }) {
  const ref = useRef(null);
  
  useLayoutEffect(() => {
    ref.current?.scrollTo(0, scrollPosition);
  }, [scrollPosition]);
  
  return <div ref={ref}>...</div>;
}

// ✅ Animations that need immediate DOM access
function AnimatedElement({ show }) {
  const ref = useRef(null);
  
  useLayoutEffect(() => {
    if (show && ref.current) {
      const height = ref.current.scrollHeight;
      ref.current.style.height = '0px';
      ref.current.offsetHeight; // Force reflow
      ref.current.style.height = `${height}px`;
    }
  }, [show]);
  
  return <div ref={ref} className="animated">{/* content */}</div>;
}
```

## 8.3 SSR Considerations

```jsx
// useLayoutEffect warning on server
// "useLayoutEffect does nothing on the server"

// Solution 1: Use useEffect for SSR-safe code
function Component() {
  // This works on both server and client
  useEffect(() => {
    // DOM measurement
  }, []);
}

// Solution 2: Conditional hook (not recommended)
const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;

function Component() {
  useIsomorphicLayoutEffect(() => {
    // Works on server and client
  }, []);
}
```

---

# 9. React 18+ Hooks

## 9.1 useTransition

Mark updates as non-urgent to keep UI responsive:

```jsx
import { useState, useTransition } from 'react';

function SearchWithTransition() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // Urgent: Update input immediately
    setQuery(value);
    
    // Non-urgent: Can be interrupted
    startTransition(() => {
      setResults(searchDatabase(value));
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      
      {isPending && <Spinner />}
      
      <ul style={{ opacity: isPending ? 0.7 : 1 }}>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useTransition Use Cases

```jsx
// Tab switching without blocking
function TabPanel() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const handleTabChange = (newTab) => {
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <div>
      <Tabs onChange={handleTabChange} />
      {isPending ? <Skeleton /> : <TabContent tab={tab} />}
    </div>
  );
}

// Large list filtering
function FilterableList({ items }) {
  const [filter, setFilter] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);
  const [isPending, startTransition] = useTransition();
  
  const handleFilter = (e) => {
    const value = e.target.value;
    setFilter(value);
    
    startTransition(() => {
      setFilteredItems(
        items.filter(item => 
          item.name.toLowerCase().includes(value.toLowerCase())
        )
      );
    });
  };
  
  return (
    <div>
      <input value={filter} onChange={handleFilter} />
      <List items={filteredItems} isPending={isPending} />
    </div>
  );
}
```

## 9.2 useDeferredValue

Defer updating a value until more urgent updates are done:

```jsx
import { useState, useDeferredValue, useMemo } from 'react';

function SearchResults({ query }) {
  // query updates immediately from input
  // deferredQuery may "lag behind"
  const deferredQuery = useDeferredValue(query);
  
  // Only recompute when deferredQuery changes
  const results = useMemo(
    () => searchDatabase(deferredQuery),
    [deferredQuery]
  );
  
  // Show stale content while loading
  const isStale = query !== deferredQuery;
  
  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      {results.map(result => (
        <Result key={result.id} data={result} />
      ))}
    </div>
  );
}

function SearchPage() {
  const [query, setQuery] = useState('');
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <SearchResults query={query} />
    </div>
  );
}
```

### useTransition vs useDeferredValue

```jsx
// useTransition: You control when to mark updates as non-urgent
// - Used INSIDE the component that triggers the update
// - Returns isPending for loading state
const [isPending, startTransition] = useTransition();
startTransition(() => setQuery(value));

// useDeferredValue: React decides when to update
// - Used when you receive a value from parent
// - No isPending, but can compare values
const deferredQuery = useDeferredValue(query);
const isStale = query !== deferredQuery;
```

## 9.3 useId

Generate unique IDs for accessibility:

```jsx
import { useId } from 'react';

function FormField({ label }) {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </div>
  );
}

// Multiple related elements
function PasswordField() {
  const id = useId();
  
  return (
    <>
      <label htmlFor={`${id}-password`}>Password</label>
      <input
        id={`${id}-password`}
        type="password"
        aria-describedby={`${id}-hint`}
      />
      <p id={`${id}-hint`}>
        Password must be at least 8 characters
      </p>
    </>
  );
}
```

## 9.4 useSyncExternalStore

Subscribe to external stores (for library authors):

```jsx
import { useSyncExternalStore } from 'react';

// Subscribe to browser online status
function useOnlineStatus() {
  return useSyncExternalStore(
    // subscribe function
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    // getSnapshot (client)
    () => navigator.onLine,
    // getServerSnapshot (SSR)
    () => true
  );
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <div>{isOnline ? '✅ Online' : '❌ Offline'}</div>;
}

// Subscribe to localStorage
function useLocalStorageValue(key) {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('storage', callback);
      return () => window.removeEventListener('storage', callback);
    },
    () => localStorage.getItem(key),
    () => null
  );
}
```

---

# 10. Custom Hooks Patterns

## 10.1 Data Fetching Hooks

```jsx
// Simple fetch hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url, { signal: controller.signal });
        if (!response.ok) throw new Error('Request failed');
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => controller.abort();
  }, [url]);
  
  return { data, loading, error };
}

// With refetch capability
function useFetchWithRefresh(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [refreshIndex, setRefreshIndex] = useState(0);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => setLoading(false));
    
    return () => controller.abort();
  }, [url, refreshIndex]);
  
  const refetch = useCallback(() => {
    setLoading(true);
    setRefreshIndex(i => i + 1);
  }, []);
  
  return { data, loading, error, refetch };
}
```

## 10.2 Form Hooks

```jsx
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  }, []);
  
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  }, []);
  
  const setValue = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
  }, []);
  
  const setError = useCallback((name, error) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  }, []);
  
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);
  
  const register = useCallback((name) => ({
    name,
    value: values[name] ?? '',
    onChange: handleChange,
    onBlur: handleBlur
  }), [values, handleChange, handleBlur]);
  
  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    setValue,
    setError,
    reset,
    register
  };
}

// Usage
function SignupForm() {
  const { register, values, errors } = useForm({
    email: '',
    password: ''
  });
  
  return (
    <form>
      <input {...register('email')} type="email" />
      {errors.email && <span>{errors.email}</span>}
      
      <input {...register('password')} type="password" />
      {errors.password && <span>{errors.password}</span>}
    </form>
  );
}
```

## 10.3 UI Hooks

```jsx
// Toggle hook
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }];
}

// Disclosure hook (common in UI libraries)
function useDisclosure(initial = false) {
  const [isOpen, setIsOpen] = useState(initial);
  
  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(v => !v), []);
  
  return { isOpen, open, close, toggle };
}

// Modal usage
function App() {
  const modal = useDisclosure();
  
  return (
    <>
      <button onClick={modal.open}>Open Modal</button>
      {modal.isOpen && (
        <Modal onClose={modal.close}>
          <p>Modal content</p>
        </Modal>
      )}
    </>
  );
}

// Counter hook
function useCounter(initial = 0, { min, max, step = 1 } = {}) {
  const [count, setCount] = useState(initial);
  
  const increment = useCallback(() => {
    setCount(c => max !== undefined ? Math.min(c + step, max) : c + step);
  }, [max, step]);
  
  const decrement = useCallback(() => {
    setCount(c => min !== undefined ? Math.max(c - step, min) : c - step);
  }, [min, step]);
  
  const reset = useCallback(() => setCount(initial), [initial]);
  
  const set = useCallback((value) => {
    setCount(c => {
      let newValue = typeof value === 'function' ? value(c) : value;
      if (min !== undefined) newValue = Math.max(newValue, min);
      if (max !== undefined) newValue = Math.min(newValue, max);
      return newValue;
    });
  }, [min, max]);
  
  return { count, increment, decrement, reset, set };
}
```

## 10.4 Browser API Hooks

```jsx
// Local storage hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue];
}

// Media query hook
function useMediaQuery(query) {
  const [matches, setMatches] = useState(() => 
    window.matchMedia(query).matches
  );
  
  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    const handler = (e) => setMatches(e.matches);
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);
  
  return matches;
}

// Window size hook
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Click outside hook
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}
```

## 10.5 Debounce and Throttle Hooks

```jsx
// Debounced value
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Debounced callback
function useDebouncedCallback(callback, delay) {
  const callbackRef = useRef(callback);
  const timeoutRef = useRef(null);
  
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);
  
  return useCallback((...args) => {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => {
      callbackRef.current(...args);
    }, delay);
  }, [delay]);
}

// Throttled callback
function useThrottledCallback(callback, delay) {
  const lastRun = useRef(Date.now() - delay);
  const callbackRef = useRef(callback);
  
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);
  
  return useCallback((...args) => {
    const now = Date.now();
    if (now - lastRun.current >= delay) {
      lastRun.current = now;
      callbackRef.current(...args);
    }
  }, [delay]);
}

// Usage
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  
  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

# 11. Advanced Hook Patterns

## 11.1 Compound Hooks

```jsx
// Combine multiple hooks into a cohesive API
function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle');
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (...args) => {
    setStatus('pending');
    setValue(null);
    setError(null);
    
    try {
      const response = await asyncFunction(...args);
      setValue(response);
      setStatus('success');
      return response;
    } catch (error) {
      setError(error);
      setStatus('error');
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return {
    execute,
    status,
    value,
    error,
    isIdle: status === 'idle',
    isLoading: status === 'pending',
    isSuccess: status === 'success',
    isError: status === 'error'
  };
}

// Usage
function UserProfile({ userId }) {
  const {
    execute: fetchUser,
    value: user,
    isLoading,
    isError,
    error
  } = useAsync(() => api.getUser(userId), false);
  
  useEffect(() => {
    fetchUser();
  }, [userId, fetchUser]);
  
  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;
  return <div>{user?.name}</div>;
}
```

## 11.2 Reducer with Middleware

```jsx
// Logger middleware
function useReducerWithLogger(reducer, initialState) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  const dispatchWithLogger = useCallback((action) => {
    console.group('Action:', action.type);
    console.log('Prev State:', state);
    console.log('Action:', action);
    dispatch(action);
    // Note: state here is stale after dispatch
    console.groupEnd();
  }, [state]);
  
  return [state, dispatchWithLogger];
}

// Async middleware
function useReducerWithThunk(reducer, initialState) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  const dispatchWithThunk = useCallback((action) => {
    if (typeof action === 'function') {
      // Action is a thunk - call it with dispatch and getState
      return action(dispatchWithThunk, () => state);
    }
    return dispatch(action);
  }, [state]);
  
  return [state, dispatchWithThunk];
}

// Usage with thunk
function TodoApp() {
  const [state, dispatch] = useReducerWithThunk(todoReducer, initialState);
  
  const addTodoAsync = (text) => async (dispatch, getState) => {
    dispatch({ type: 'ADD_TODO_START' });
    
    try {
      const todo = await api.createTodo(text);
      dispatch({ type: 'ADD_TODO_SUCCESS', payload: todo });
    } catch (error) {
      dispatch({ type: 'ADD_TODO_ERROR', payload: error.message });
    }
  };
  
  return (
    <button onClick={() => dispatch(addTodoAsync('New Todo'))}>
      Add Todo
    </button>
  );
}
```

## 11.3 Hooks with Context

```jsx
// Create a hook that combines context with local logic
function createStateContext(useValue) {
  const Context = createContext(null);
  
  function Provider({ children, ...props }) {
    const value = useValue(props);
    
    return (
      <Context.Provider value={value}>
        {children}
      </Context.Provider>
    );
  }
  
  function useContextValue() {
    const context = useContext(Context);
    if (context === null) {
      throw new Error('useContextValue must be used within Provider');
    }
    return context;
  }
  
  return [Provider, useContextValue];
}

// Usage
function useCounterState({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);
  
  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialCount), [initialCount]);
  
  return { count, increment, decrement, reset };
}

const [CounterProvider, useCounter] = createStateContext(useCounterState);

function App() {
  return (
    <CounterProvider initialCount={10}>
      <Counter />
      <Controls />
    </CounterProvider>
  );
}

function Counter() {
  const { count } = useCounter();
  return <div>Count: {count}</div>;
}

function Controls() {
  const { increment, decrement, reset } = useCounter();
  return (
    <>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </>
  );
}
```

---

# 12. Testing Hooks

## 12.1 Testing with renderHook

```jsx
import { renderHook, act } from '@testing-library/react';

// Test useCounter hook
describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });
  
  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });
  
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('should decrement counter', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  it('should reset counter', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

## 12.2 Testing Async Hooks

```jsx
import { renderHook, waitFor } from '@testing-library/react';

// Mock fetch
global.fetch = jest.fn();

describe('useFetch', () => {
  beforeEach(() => {
    fetch.mockClear();
  });
  
  it('should fetch data successfully', async () => {
    const mockData = { id: 1, name: 'Test' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });
    
    const { result } = renderHook(() => useFetch('/api/test'));
    
    // Initially loading
    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBe(null);
    
    // Wait for fetch to complete
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBe(null);
  });
  
  it('should handle fetch error', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404
    });
    
    const { result } = renderHook(() => useFetch('/api/not-found'));
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.error).toBeTruthy();
    expect(result.current.data).toBe(null);
  });
  
  it('should refetch when url changes', async () => {
    fetch.mockResolvedValue({
      ok: true,
      json: async () => ({ id: 1 })
    });
    
    const { result, rerender } = renderHook(
      ({ url }) => useFetch(url),
      { initialProps: { url: '/api/users/1' } }
    );
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(fetch).toHaveBeenCalledWith('/api/users/1', expect.any(Object));
    
    // Change URL
    rerender({ url: '/api/users/2' });
    
    await waitFor(() => {
      expect(fetch).toHaveBeenCalledWith('/api/users/2', expect.any(Object));
    });
  });
});
```

## 12.3 Testing Hooks with Context

```jsx
import { renderHook } from '@testing-library/react';

// Create wrapper with context
function createWrapper(value) {
  return function Wrapper({ children }) {
    return (
      <TestContext.Provider value={value}>
        {children}
      </TestContext.Provider>
    );
  };
}

describe('useAuth', () => {
  it('should return user from context', () => {
    const mockUser = { id: '1', name: 'John' };
    const wrapper = createWrapper({ user: mockUser, login: jest.fn() });
    
    const { result } = renderHook(() => useAuth(), { wrapper });
    
    expect(result.current.user).toEqual(mockUser);
  });
  
  it('should throw error when used outside provider', () => {
    // Suppress console.error for this test
    const spy = jest.spyOn(console, 'error').mockImplementation(() => {});
    
    expect(() => {
      renderHook(() => useAuth());
    }).toThrow('useAuth must be used within AuthProvider');
    
    spy.mockRestore();
  });
});
```

---

# 13. Common Mistakes and Solutions

## 13.1 Stale Closures

```jsx
// ❌ Stale closure problem
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log('Count:', count); // Always logs 0!
      setCount(count + 1);          // Always sets to 1!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // count not in deps = stale closure
}

// ✅ Solution 1: Function update
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(c => c + 1); // Always has latest value
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
}

// ✅ Solution 2: useRef for reading latest value
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  
  useEffect(() => {
    countRef.current = count;
  }, [count]);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log('Count:', countRef.current); // Always latest
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
}
```

## 13.2 Missing Dependencies

```jsx
// ❌ Missing dependency
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    fetchResults(query).then(setResults);
  }, []); // query missing! Won't refetch when query changes
}

// ✅ Include all dependencies
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    fetchResults(query).then(setResults);
  }, [query]); // Correct!
}

// ✅ Or use exhaustive-deps ESLint rule
// "react-hooks/exhaustive-deps": "error"
```

## 13.3 Object/Array Dependencies

```jsx
// ❌ Object in dependency - infinite loop!
function Component({ userId }) {
  const options = { userId, limit: 10 };
  
  useEffect(() => {
    fetchData(options);
  }, [options]); // New object every render!
}

// ✅ Solution 1: Primitive dependencies
function Component({ userId }) {
  useEffect(() => {
    const options = { userId, limit: 10 };
    fetchData(options);
  }, [userId]); // Only primitives
}

// ✅ Solution 2: useMemo
function Component({ userId }) {
  const options = useMemo(() => ({ userId, limit: 10 }), [userId]);
  
  useEffect(() => {
    fetchData(options);
  }, [options]); // Stable reference
}
```

## 13.4 State Update After Unmount

```jsx
// ❌ Can cause memory leak warning
function Component({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setData);
    // If component unmounts before fetch completes,
    // setData is called on unmounted component
  }, [userId]);
}

// ✅ Solution 1: Cleanup flag
function Component({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    fetchUser(userId).then(data => {
      if (isMounted) setData(data);
    });
    
    return () => { isMounted = false; };
  }, [userId]);
}

// ✅ Solution 2: AbortController
function Component({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') throw err;
      });
    
    return () => controller.abort();
  }, [userId]);
}
```

---

# 14. Interview Questions

## 14.1 Basic Questions

**Q1: What are React Hooks?**

Hooks are functions that let you use state and other React features in function components. They were introduced in React 16.8 to:
- Reuse stateful logic without changing component hierarchy
- Split complex components by related logic
- Avoid confusion of classes (this binding, lifecycle methods)

---

**Q2: What are the rules of hooks?**

1. **Only call hooks at the top level** - Don't call in loops, conditions, or nested functions
2. **Only call hooks from React functions** - Function components or custom hooks

These rules ensure hooks are called in the same order every render.

---

**Q3: useState vs useReducer - when to use each?**

| useState | useReducer |
|----------|------------|
| Simple primitives | Complex objects |
| Independent updates | Related updates |
| Few state transitions | Many state transitions |
| Logic in component | Logic in reducer (testable) |

---

**Q4: What's the difference between useEffect and useLayoutEffect?**

| useEffect | useLayoutEffect |
|-----------|-----------------|
| Runs async after paint | Runs sync before paint |
| Non-blocking | Blocks browser paint |
| Data fetching, subscriptions | DOM measurements |
| SSR-safe | SSR warning |

---

**Q5: How does useCallback differ from useMemo?**

```jsx
// useMemo: Memoizes a VALUE
const sortedItems = useMemo(() => items.sort(), [items]);

// useCallback: Memoizes a FUNCTION
const handleClick = useCallback(() => onClick(id), [id]);

// useCallback(fn, deps) === useMemo(() => fn, deps)
```

---

## 14.2 Advanced Questions

**Q6: Explain the dependency array in useEffect**

The dependency array tells React when to re-run the effect:
- **No array**: Runs after every render
- **Empty `[]`**: Runs only on mount (and cleanup on unmount)
- **`[deps]`**: Runs when any dependency changes

React compares dependencies using `Object.is` (shallow comparison).

---

**Q7: What causes infinite loops with useEffect?**

1. Missing dependency array
2. Object/array in dependencies (new reference each render)
3. State update in effect without proper condition
4. Function in dependencies without useCallback

```jsx
// ❌ Infinite loop
useEffect(() => {
  setCount(count + 1);
}); // No deps = runs every render

// ✅ Fixed
useEffect(() => {
  setCount(c => c + 1);
}, []); // Runs once
```

---

**Q8: How do you share stateful logic between components?**

Create custom hooks! They're functions that use other hooks:

```jsx
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const handleResize = () => setSize({
      width: window.innerWidth,
      height: window.innerHeight
    });
    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Reuse in any component
function Header() {
  const { width } = useWindowSize();
  return width < 768 ? <MobileHeader /> : <DesktopHeader />;
}
```

---

**Q9: What is useTransition and when would you use it?**

useTransition marks updates as non-urgent, allowing React to keep the UI responsive:

```jsx
const [isPending, startTransition] = useTransition();

const handleChange = (e) => {
  setInput(e.target.value);        // Urgent: update input
  startTransition(() => {
    setSearchResults(search(e.target.value)); // Non-urgent: can wait
  });
};
```

Use for: Tab switching, filtering large lists, any UI that shouldn't block input.

---

**Q10: Explain useSyncExternalStore**

For subscribing to external stores (non-React state):

```jsx
const value = useSyncExternalStore(
  subscribe,      // (callback) => unsubscribe
  getSnapshot,    // () => current value
  getServerSnapshot // () => server value (SSR)
);
```

Used internally by Redux, Zustand, etc.

---

# 15. Coding Challenges

## 15.1 Implement useDebounce

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}
```

## 15.2 Implement usePrevious

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}
```

## 15.3 Implement useInterval

```jsx
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);
  
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  useEffect(() => {
    if (delay === null) return;
    
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

## 15.4 Implement useToggle

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }];
}
```

## 15.5 Implement useOnMount

```jsx
function useOnMount(callback) {
  const callbackRef = useRef(callback);
  
  useEffect(() => {
    callbackRef.current();
  }, []);
}

function useOnUnmount(callback) {
  const callbackRef = useRef(callback);
  
  useEffect(() => {
    callbackRef.current = callback;
  });
  
  useEffect(() => {
    return () => callbackRef.current();
  }, []);
}
```

## 15.6 Implement useArray

```jsx
function useArray(initialValue = []) {
  const [array, setArray] = useState(initialValue);
  
  const push = useCallback((element) => {
    setArray(a => [...a, element]);
  }, []);
  
  const filter = useCallback((callback) => {
    setArray(a => a.filter(callback));
  }, []);
  
  const update = useCallback((index, newElement) => {
    setArray(a => [
      ...a.slice(0, index),
      newElement,
      ...a.slice(index + 1)
    ]);
  }, []);
  
  const remove = useCallback((index) => {
    setArray(a => [...a.slice(0, index), ...a.slice(index + 1)]);
  }, []);
  
  const clear = useCallback(() => setArray([]), []);
  
  return { array, set: setArray, push, filter, update, remove, clear };
}
```

## 15.7 Implement useEventListener

```jsx
function useEventListener(eventName, handler, element = window) {
  const savedHandler = useRef(handler);
  
  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);
  
  useEffect(() => {
    const el = element?.current ?? element;
    if (!el?.addEventListener) return;
    
    const eventListener = (event) => savedHandler.current(event);
    el.addEventListener(eventName, eventListener);
    
    return () => el.removeEventListener(eventName, eventListener);
  }, [eventName, element]);
}

// Usage
function Component() {
  const buttonRef = useRef(null);
  
  useEventListener('scroll', handleScroll); // window
  useEventListener('click', handleClick, buttonRef); // element
}
```

## 15.8 Implement useAsync

```jsx
function useAsync(asyncFunction, immediate = true) {
  const [state, setState] = useState({
    status: 'idle',
    value: null,
    error: null
  });
  
  const execute = useCallback(async (...args) => {
    setState({ status: 'pending', value: null, error: null });
    
    try {
      const response = await asyncFunction(...args);
      setState({ status: 'success', value: response, error: null });
      return response;
    } catch (error) {
      setState({ status: 'error', value: null, error });
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) execute();
  }, [execute, immediate]);
  
  return { ...state, execute };
}
```

---

## 🎯 Final Hooks Study Checklist

### Core Hooks
- [ ] useState (lazy init, functional updates)
- [ ] useEffect (deps, cleanup, async patterns)
- [ ] useContext (with custom hooks)
- [ ] useReducer (TypeScript, complex state)
- [ ] useRef (DOM, mutable values)
- [ ] useMemo (when to use)
- [ ] useCallback (with React.memo)
- [ ] useLayoutEffect (vs useEffect)

### React 18 Hooks
- [ ] useTransition
- [ ] useDeferredValue
- [ ] useId
- [ ] useSyncExternalStore

### Advanced Patterns
- [ ] Custom hooks (data, forms, UI, browser APIs)
- [ ] Compound hooks
- [ ] Hooks with context
- [ ] Testing hooks

### Common Pitfalls
- [ ] Stale closures
- [ ] Missing dependencies
- [ ] Object/array deps
- [ ] State after unmount

---

# 16. More Advanced Custom Hooks

## 16.1 useIntersectionObserver

```jsx
function useIntersectionObserver(
  ref,
  options = { threshold: 0, rootMargin: '0px' }
) {
  const [entry, setEntry] = useState(null);
  const [isIntersecting, setIsIntersecting] = useState(false);
  
  useEffect(() => {
    const node = ref?.current;
    if (!node) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        setEntry(entry);
        setIsIntersecting(entry.isIntersecting);
      },
      options
    );
    
    observer.observe(node);
    
    return () => observer.disconnect();
  }, [ref, options.threshold, options.rootMargin]);
  
  return { entry, isIntersecting };
}

// Usage: Lazy load images
function LazyImage({ src, alt }) {
  const ref = useRef(null);
  const { isIntersecting } = useIntersectionObserver(ref, {
    threshold: 0.1,
    rootMargin: '100px'
  });
  const [loaded, setLoaded] = useState(false);
  
  useEffect(() => {
    if (isIntersecting && !loaded) {
      setLoaded(true);
    }
  }, [isIntersecting, loaded]);
  
  return (
    <div ref={ref} className="lazy-image-container">
      {loaded ? (
        <img src={src} alt={alt} />
      ) : (
        <div className="placeholder" />
      )}
    </div>
  );
}
```

## 16.2 useScrollPosition

```jsx
function useScrollPosition() {
  const [scrollPosition, setScrollPosition] = useState({
    x: 0,
    y: 0,
    direction: null
  });
  const previousY = useRef(0);
  
  useEffect(() => {
    const handleScroll = () => {
      const currentY = window.scrollY;
      const direction = currentY > previousY.current ? 'down' : 'up';
      previousY.current = currentY;
      
      setScrollPosition({
        x: window.scrollX,
        y: currentY,
        direction
      });
    };
    
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return scrollPosition;
}

// Usage: Show/hide header on scroll
function Header() {
  const { direction, y } = useScrollPosition();
  const [visible, setVisible] = useState(true);
  
  useEffect(() => {
    if (y < 100) {
      setVisible(true);
    } else if (direction === 'down') {
      setVisible(false);
    } else {
      setVisible(true);
    }
  }, [direction, y]);
  
  return (
    <header
      className={`header ${visible ? 'visible' : 'hidden'}`}
    >
      <nav>...</nav>
    </header>
  );
}
```

## 16.3 useKeyPress

```jsx
function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = useState(false);
  
  useEffect(() => {
    const downHandler = (e) => {
      if (e.key === targetKey) {
        setKeyPressed(true);
      }
    };
    
    const upHandler = (e) => {
      if (e.key === targetKey) {
        setKeyPressed(false);
      }
    };
    
    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);
    
    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [targetKey]);
  
  return keyPressed;
}

// Usage: Keyboard controls for game
function Game() {
  const upPressed = useKeyPress('ArrowUp');
  const downPressed = useKeyPress('ArrowDown');
  const leftPressed = useKeyPress('ArrowLeft');
  const rightPressed = useKeyPress('ArrowRight');
  
  useEffect(() => {
    if (upPressed) movePlayer('up');
    if (downPressed) movePlayer('down');
    if (leftPressed) movePlayer('left');
    if (rightPressed) movePlayer('right');
  }, [upPressed, downPressed, leftPressed, rightPressed]);
  
  return <GameBoard />;
}
```

## 16.4 useHover

```jsx
function useHover() {
  const [isHovered, setIsHovered] = useState(false);
  const ref = useRef(null);
  
  useEffect(() => {
    const node = ref.current;
    if (!node) return;
    
    const handleMouseEnter = () => setIsHovered(true);
    const handleMouseLeave = () => setIsHovered(false);
    
    node.addEventListener('mouseenter', handleMouseEnter);
    node.addEventListener('mouseleave', handleMouseLeave);
    
    return () => {
      node.removeEventListener('mouseenter', handleMouseEnter);
      node.removeEventListener('mouseleave', handleMouseLeave);
    };
  }, []);
  
  return [ref, isHovered];
}

// Usage: Card with hover effect
function Card({ children }) {
  const [hoverRef, isHovered] = useHover();
  
  return (
    <div
      ref={hoverRef}
      className={`card ${isHovered ? 'card-hovered' : ''}`}
    >
      {children}
      {isHovered && <div className="card-overlay" />}
    </div>
  );
}
```

## 16.5 useClipboard

```jsx
function useClipboard(timeout = 2000) {
  const [copied, setCopied] = useState(false);
  const [error, setError] = useState(null);
  
  const copy = useCallback(async (text) => {
    try {
      await navigator.clipboard.writeText(text);
      setCopied(true);
      setError(null);
      
      setTimeout(() => setCopied(false), timeout);
    } catch (err) {
      setError(err);
      setCopied(false);
    }
  }, [timeout]);
  
  const reset = useCallback(() => {
    setCopied(false);
    setError(null);
  }, []);
  
  return { copied, error, copy, reset };
}

// Usage: Copy button
function CopyButton({ text }) {
  const { copied, copy } = useClipboard();
  
  return (
    <button onClick={() => copy(text)}>
      {copied ? '✓ Copied!' : 'Copy'}
    </button>
  );
}
```

## 16.6 useOnlineStatus

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(
    typeof navigator !== 'undefined' ? navigator.onLine : true
  );
  
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  return isOnline;
}

// Usage: Show offline banner
function App() {
  const isOnline = useOnlineStatus();
  
  return (
    <div>
      {!isOnline && (
        <div className="offline-banner">
          You're offline. Some features may not be available.
        </div>
      )}
      <MainContent />
    </div>
  );
}
```

## 16.7 useMutationObserver

```jsx
function useMutationObserver(
  ref,
  callback,
  options = {
    attributes: true,
    childList: true,
    subtree: true
  }
) {
  useEffect(() => {
    const node = ref.current;
    if (!node) return;
    
    const observer = new MutationObserver(callback);
    observer.observe(node, options);
    
    return () => observer.disconnect();
  }, [ref, callback, options]);
}

// Usage: Track DOM changes
function DynamicContent() {
  const ref = useRef(null);
  const [changeCount, setChangeCount] = useState(0);
  
  const handleMutations = useCallback((mutations) => {
    setChangeCount(c => c + mutations.length);
  }, []);
  
  useMutationObserver(ref, handleMutations);
  
  return (
    <div ref={ref}>
      <p>This content may change dynamically</p>
      <p>Changes detected: {changeCount}</p>
    </div>
  );
}
```

## 16.8 useLockBodyScroll

```jsx
function useLockBodyScroll(lock = true) {
  useLayoutEffect(() => {
    if (!lock) return;
    
    const originalStyle = window.getComputedStyle(document.body).overflow;
    document.body.style.overflow = 'hidden';
    
    return () => {
      document.body.style.overflow = originalStyle;
    };
  }, [lock]);
}

// Usage: Modal that locks scroll
function Modal({ children, isOpen, onClose }) {
  useLockBodyScroll(isOpen);
  
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

---

# 17. Real-World Hook Recipes

## 17.1 Infinite Scroll Hook

```jsx
function useInfiniteScroll(callback, options = {}) {
  const {
    threshold = 100,
    container = null
  } = options;
  
  const observer = useRef(null);
  const isLoading = useRef(false);
  
  const lastElementRef = useCallback((node) => {
    if (isLoading.current) return;
    
    if (observer.current) {
      observer.current.disconnect();
    }
    
    observer.current = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) {
        isLoading.current = true;
        callback().finally(() => {
          isLoading.current = false;
        });
      }
    }, {
      root: container,
      rootMargin: `${threshold}px`
    });
    
    if (node) {
      observer.current.observe(node);
    }
  }, [callback, threshold, container]);
  
  return lastElementRef;
}

// Usage
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  const loadMore = useCallback(async () => {
    const newItems = await fetchItems(page);
    setItems(prev => [...prev, ...newItems]);
    setPage(p => p + 1);
    setHasMore(newItems.length > 0);
  }, [page]);
  
  const lastItemRef = useInfiniteScroll(loadMore);
  
  return (
    <ul>
      {items.map((item, index) => (
        <li
          key={item.id}
          ref={index === items.length - 1 ? lastItemRef : null}
        >
          {item.name}
        </li>
      ))}
      {!hasMore && <li>No more items</li>}
    </ul>
  );
}
```

## 17.2 Pagination Hook

```jsx
function usePagination({ totalItems, itemsPerPage = 10, initialPage = 1 }) {
  const [currentPage, setCurrentPage] = useState(initialPage);
  
  const totalPages = Math.ceil(totalItems / itemsPerPage);
  
  const canGoNext = currentPage < totalPages;
  const canGoPrevious = currentPage > 1;
  
  const goToPage = useCallback((page) => {
    setCurrentPage(Math.min(Math.max(1, page), totalPages));
  }, [totalPages]);
  
  const goToNext = useCallback(() => {
    if (canGoNext) setCurrentPage(p => p + 1);
  }, [canGoNext]);
  
  const goToPrevious = useCallback(() => {
    if (canGoPrevious) setCurrentPage(p => p - 1);
  }, [canGoPrevious]);
  
  const goToFirst = useCallback(() => setCurrentPage(1), []);
  const goToLast = useCallback(() => setCurrentPage(totalPages), [totalPages]);
  
  const startIndex = (currentPage - 1) * itemsPerPage;
  const endIndex = Math.min(startIndex + itemsPerPage, totalItems);
  
  // Generate page numbers for display
  const pages = useMemo(() => {
    const pageNumbers = [];
    const maxVisible = 5;
    
    let start = Math.max(1, currentPage - Math.floor(maxVisible / 2));
    let end = Math.min(totalPages, start + maxVisible - 1);
    
    if (end - start < maxVisible - 1) {
      start = Math.max(1, end - maxVisible + 1);
    }
    
    for (let i = start; i <= end; i++) {
      pageNumbers.push(i);
    }
    
    return pageNumbers;
  }, [currentPage, totalPages]);
  
  return {
    currentPage,
    totalPages,
    canGoNext,
    canGoPrevious,
    goToPage,
    goToNext,
    goToPrevious,
    goToFirst,
    goToLast,
    startIndex,
    endIndex,
    pages
  };
}

// Usage
function PaginatedTable({ data }) {
  const {
    currentPage,
    totalPages,
    canGoNext,
    canGoPrevious,
    goToNext,
    goToPrevious,
    goToPage,
    startIndex,
    endIndex,
    pages
  } = usePagination({
    totalItems: data.length,
    itemsPerPage: 10
  });
  
  const visibleData = data.slice(startIndex, endIndex);
  
  return (
    <div>
      <table>
        <tbody>
          {visibleData.map(item => (
            <tr key={item.id}>{/* ... */}</tr>
          ))}
        </tbody>
      </table>
      
      <div className="pagination">
        <button onClick={goToPrevious} disabled={!canGoPrevious}>
          Previous
        </button>
        
        {pages.map(page => (
          <button
            key={page}
            onClick={() => goToPage(page)}
            className={page === currentPage ? 'active' : ''}
          >
            {page}
          </button>
        ))}
        
        <button onClick={goToNext} disabled={!canGoNext}>
          Next
        </button>
        
        <span>Page {currentPage} of {totalPages}</span>
      </div>
    </div>
  );
}
```

## 17.3 Search with Filters Hook

```jsx
function useSearch(items, options = {}) {
  const {
    searchKeys = ['name'],
    debounceMs = 300
  } = options;
  
  const [query, setQuery] = useState('');
  const [filters, setFilters] = useState({});
  const [sortBy, setSortBy] = useState(null);
  const [sortOrder, setSortOrder] = useState('asc');
  
  const debouncedQuery = useDebounce(query, debounceMs);
  
  const filteredItems = useMemo(() => {
    let result = [...items];
    
    // Apply search query
    if (debouncedQuery) {
      const lowerQuery = debouncedQuery.toLowerCase();
      result = result.filter(item =>
        searchKeys.some(key =>
          String(item[key]).toLowerCase().includes(lowerQuery)
        )
      );
    }
    
    // Apply filters
    Object.entries(filters).forEach(([key, value]) => {
      if (value != null && value !== '' && value !== 'all') {
        result = result.filter(item => {
          if (Array.isArray(value)) {
            return value.includes(item[key]);
          }
          return item[key] === value;
        });
      }
    });
    
    // Apply sorting
    if (sortBy) {
      result.sort((a, b) => {
        const aVal = a[sortBy];
        const bVal = b[sortBy];
        
        let comparison = 0;
        if (aVal < bVal) comparison = -1;
        if (aVal > bVal) comparison = 1;
        
        return sortOrder === 'asc' ? comparison : -comparison;
      });
    }
    
    return result;
  }, [items, debouncedQuery, filters, sortBy, sortOrder, searchKeys]);
  
  const setFilter = useCallback((key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  }, []);
  
  const clearFilters = useCallback(() => {
    setFilters({});
    setQuery('');
    setSortBy(null);
  }, []);
  
  const toggleSort = useCallback((key) => {
    if (sortBy === key) {
      setSortOrder(order => order === 'asc' ? 'desc' : 'asc');
    } else {
      setSortBy(key);
      setSortOrder('asc');
    }
  }, [sortBy]);
  
  return {
    query,
    setQuery,
    filters,
    setFilter,
    clearFilters,
    sortBy,
    sortOrder,
    toggleSort,
    results: filteredItems,
    resultCount: filteredItems.length
  };
}

// Usage
function ProductList({ products }) {
  const {
    query,
    setQuery,
    filters,
    setFilter,
    clearFilters,
    sortBy,
    sortOrder,
    toggleSort,
    results
  } = useSearch(products, {
    searchKeys: ['name', 'description'],
    debounceMs: 300
  });
  
  return (
    <div>
      <input
        type="search"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      
      <select
        value={filters.category || 'all'}
        onChange={e => setFilter('category', e.target.value)}
      >
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      
      <button onClick={clearFilters}>Clear All</button>
      
      <table>
        <thead>
          <tr>
            <th onClick={() => toggleSort('name')}>
              Name {sortBy === 'name' && (sortOrder === 'asc' ? '↑' : '↓')}
            </th>
            <th onClick={() => toggleSort('price')}>
              Price {sortBy === 'price' && (sortOrder === 'asc' ? '↑' : '↓')}
            </th>
          </tr>
        </thead>
        <tbody>
          {results.map(product => (
            <tr key={product.id}>
              <td>{product.name}</td>
              <td>${product.price}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## 17.4 Undo/Redo Hook

```jsx
function useHistory(initialState) {
  const [history, setHistory] = useState({
    past: [],
    present: initialState,
    future: []
  });
  
  const canUndo = history.past.length > 0;
  const canRedo = history.future.length > 0;
  
  const set = useCallback((newPresent) => {
    setHistory(prev => ({
      past: [...prev.past, prev.present],
      present: typeof newPresent === 'function'
        ? newPresent(prev.present)
        : newPresent,
      future: []
    }));
  }, []);
  
  const undo = useCallback(() => {
    setHistory(prev => {
      if (prev.past.length === 0) return prev;
      
      const previous = prev.past[prev.past.length - 1];
      const newPast = prev.past.slice(0, -1);
      
      return {
        past: newPast,
        present: previous,
        future: [prev.present, ...prev.future]
      };
    });
  }, []);
  
  const redo = useCallback(() => {
    setHistory(prev => {
      if (prev.future.length === 0) return prev;
      
      const next = prev.future[0];
      const newFuture = prev.future.slice(1);
      
      return {
        past: [...prev.past, prev.present],
        present: next,
        future: newFuture
      };
    });
  }, []);
  
  const reset = useCallback((newPresent = initialState) => {
    setHistory({
      past: [],
      present: newPresent,
      future: []
    });
  }, [initialState]);
  
  return {
    state: history.present,
    set,
    undo,
    redo,
    reset,
    canUndo,
    canRedo
  };
}

// Usage: Text editor with undo/redo
function TextEditor() {
  const { state, set, undo, redo, canUndo, canRedo } = useHistory('');
  
  return (
    <div>
      <div className="toolbar">
        <button onClick={undo} disabled={!canUndo}>Undo</button>
        <button onClick={redo} disabled={!canRedo}>Redo</button>
      </div>
      <textarea
        value={state}
        onChange={e => set(e.target.value)}
      />
    </div>
  );
}
```

---

# 18. Output-Based Interview Questions

## 18.1 Question 1: What's the output?

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Effect:', count);
  });
  
  console.log('Render:', count);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

**Answer:**
1. On initial render: "Render: 0", then "Effect: 0"
2. On each click: "Render: N", then "Effect: N"

Effects run after render, so console logs in the component body run first.

---

## 18.2 Question 2: What's the issue?

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, []);
  
  return user ? <div>{user.name}</div> : <Loading />;
}
```

**Answer:**
Missing `userId` in dependency array. If `userId` prop changes, the effect won't re-run and stale data will be shown.

**Fix:**
```jsx
useEffect(() => {
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(setUser);
}, [userId]); // Include userId
```

---

## 18.3 Question 3: What will happen?

```jsx
function App() {
  const [items, setItems] = useState([1, 2, 3]);
  
  const addItem = () => {
    setItems([...items, items.length + 1]);
    setItems([...items, items.length + 2]);
  };
  
  return (
    <div>
      {items.join(', ')}
      <button onClick={addItem}>Add</button>
    </div>
  );
}
```

**Answer:**
After clicking "Add", items will be `[1, 2, 3, 5]` not `[1, 2, 3, 4, 5]`.

Both `setItems` calls use the same stale `items` value from the closure. React 18 batches these updates, but both see `items.length` as 3.

**Fix:**
```jsx
const addItem = () => {
  setItems(prev => [...prev, prev.length + 1]);
  setItems(prev => [...prev, prev.length + 1]);
};
// Result: [1, 2, 3, 4, 5]
```

---

## 18.4 Question 4: Why doesn't this work?

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    setInterval(() => {
      setSeconds(seconds + 1);
    }, 1000);
  }, []);
  
  return <div>Seconds: {seconds}</div>;
}
```

**Answer:**
The timer increments from 0 to 1 but never beyond. Due to closure, `seconds` is always 0 inside the interval callback.

Also missing cleanup - creates new interval each mount (not a problem with [] dep, but good practice).

**Fix:**
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setSeconds(s => s + 1); // Functional update
  }, 1000);
  
  return () => clearInterval(id);
}, []);
```

---

## 18.5 Question 5: Is this memo effective?

```jsx
const Child = React.memo(({ config }) => {
  console.log('Child rendered');
  return <div>{config.theme}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <Child config={{ theme: 'dark' }} />
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
    </>
  );
}
```

**Answer:**
NO, the memo is NOT effective. The inline object `{ theme: 'dark' }` creates a new reference every render, defeating React.memo.

**Fix:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const config = useMemo(() => ({ theme: 'dark' }), []);
  
  return (
    <>
      <Child config={config} />
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
    </>
  );
}
```

---

# 19. More Coding Challenges

## 19.1 Implement useSessionStorage

```jsx
function useSessionStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.sessionStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      setStoredValue(valueToStore);
      window.sessionStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  const removeValue = useCallback(() => {
    try {
      window.sessionStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error(error);
    }
  }, [key, initialValue]);
  
  return [storedValue, setValue, removeValue];
}
```

## 19.2 Implement useFocus

```jsx
function useFocus() {
  const ref = useRef(null);
  const [isFocused, setIsFocused] = useState(false);
  
  useEffect(() => {
    const node = ref.current;
    if (!node) return;
    
    const handleFocus = () => setIsFocused(true);
    const handleBlur = () => setIsFocused(false);
    
    node.addEventListener('focus', handleFocus);
    node.addEventListener('blur', handleBlur);
    
    return () => {
      node.removeEventListener('focus', handleFocus);
      node.removeEventListener('blur', handleBlur);
    };
  }, []);
  
  const setFocus = useCallback(() => {
    ref.current?.focus();
  }, []);
  
  return { ref, isFocused, setFocus };
}
```

## 19.3 Implement useQueue

```jsx
function useQueue(initialQueue = []) {
  const [queue, setQueue] = useState(initialQueue);
  
  const enqueue = useCallback((item) => {
    setQueue(prev => [...prev, item]);
  }, []);
  
  const dequeue = useCallback(() => {
    let removed;
    setQueue(prev => {
      if (prev.length === 0) return prev;
      [removed] = prev;
      return prev.slice(1);
    });
    return removed;
  }, []);
  
  const peek = useCallback(() => {
    return queue[0];
  }, [queue]);
  
  const clear = useCallback(() => {
    setQueue([]);
  }, []);
  
  return {
    queue,
    enqueue,
    dequeue,
    peek,
    clear,
    size: queue.length,
    isEmpty: queue.length === 0
  };
}
```

## 19.4 Implement useStep

```jsx
function useStep(maxStep, initialStep = 0) {
  const [step, setStep] = useState(initialStep);
  
  const canGoNext = step < maxStep;
  const canGoPrev = step > 0;
  
  const goNext = useCallback(() => {
    setStep(s => Math.min(s + 1, maxStep));
  }, [maxStep]);
  
  const goPrev = useCallback(() => {
    setStep(s => Math.max(s - 1, 0));
  }, []);
  
  const goTo = useCallback((newStep) => {
    setStep(Math.min(Math.max(newStep, 0), maxStep));
  }, [maxStep]);
  
  const reset = useCallback(() => {
    setStep(initialStep);
  }, [initialStep]);
  
  return {
    step,
    goNext,
    goPrev,
    goTo,
    reset,
    canGoNext,
    canGoPrev,
    isFirst: step === 0,
    isLast: step === maxStep
  };
}

// Usage: Multi-step form
function Wizard() {
  const { step, goNext, goPrev, canGoNext, canGoPrev } = useStep(3);
  
  return (
    <div>
      {step === 0 && <Step1 />}
      {step === 1 && <Step2 />}
      {step === 2 && <Step3 />}
      {step === 3 && <Review />}
      
      <button onClick={goPrev} disabled={!canGoPrev}>Back</button>
      <button onClick={goNext} disabled={!canGoNext}>Next</button>
    </div>
  );
}
```

## 19.5 Implement useMap

```jsx
function useMap(initialMap = new Map()) {
  const [map, setMap] = useState(new Map(initialMap));
  
  const set = useCallback((key, value) => {
    setMap(prev => {
      const next = new Map(prev);
      next.set(key, value);
      return next;
    });
  }, []);
  
  const deleteKey = useCallback((key) => {
    setMap(prev => {
      const next = new Map(prev);
      next.delete(key);
      return next;
    });
  }, []);
  
  const get = useCallback((key) => {
    return map.get(key);
  }, [map]);
  
  const has = useCallback((key) => {
    return map.has(key);
  }, [map]);
  
  const clear = useCallback(() => {
    setMap(new Map());
  }, []);
  
  const reset = useCallback(() => {
    setMap(new Map(initialMap));
  }, [initialMap]);
  
  return {
    map,
    set,
    delete: deleteKey,
    get,
    has,
    clear,
    reset,
    size: map.size
  };
}
```

## 19.6 Implement useSet

```jsx
function useSet(initialSet = new Set()) {
  const [set, setSet] = useState(new Set(initialSet));
  
  const add = useCallback((value) => {
    setSet(prev => new Set([...prev, value]));
  }, []);
  
  const deleteValue = useCallback((value) => {
    setSet(prev => {
      const next = new Set(prev);
      next.delete(value);
      return next;
    });
  }, []);
  
  const has = useCallback((value) => {
    return set.has(value);
  }, [set]);
  
  const toggle = useCallback((value) => {
    setSet(prev => {
      const next = new Set(prev);
      if (next.has(value)) {
        next.delete(value);
      } else {
        next.add(value);
      }
      return next;
    });
  }, []);
  
  const clear = useCallback(() => {
    setSet(new Set());
  }, []);
  
  return {
    set,
    add,
    delete: deleteValue,
    has,
    toggle,
    clear,
    size: set.size,
    values: [...set]
  };
}

// Usage: Multi-select
function MultiSelect({ options }) {
  const { values, toggle, has } = useSet();
  
  return (
    <div>
      {options.map(option => (
        <label key={option.id}>
          <input
            type="checkbox"
            checked={has(option.id)}
            onChange={() => toggle(option.id)}
          />
          {option.label}
        </label>
      ))}
      <p>Selected: {values.join(', ')}</p>
    </div>
  );
}
```

---

# 20. Hooks Cheat Sheet

## 20.1 Built-in Hooks Summary

| Hook | Purpose | Signature |
|------|---------|-----------|
| `useState` | Local state | `[value, setValue] = useState(initial)` |
| `useReducer` | Complex state | `[state, dispatch] = useReducer(reducer, initial)` |
| `useContext` | Consume context | `value = useContext(Context)` |
| `useRef` | Mutable ref | `ref = useRef(initial)` |
| `useEffect` | Side effects | `useEffect(effect, deps?)` |
| `useLayoutEffect` | Sync effects | `useLayoutEffect(effect, deps?)` |
| `useMemo` | Memoize value | `value = useMemo(create, deps)` |
| `useCallback` | Memoize function | `fn = useCallback(fn, deps)` |
| `useImperativeHandle` | Customize ref | `useImperativeHandle(ref, create, deps?)` |
| `useDebugValue` | DevTools label | `useDebugValue(value)` |
| `useId` | Unique ID | `id = useId()` |
| `useTransition` | Non-blocking | `[isPending, start] = useTransition()` |
| `useDeferredValue` | Defer value | `deferred = useDeferredValue(value)` |
| `useSyncExternalStore` | External store | `state = useSyncExternalStore(sub, get, getServer?)` |

## 20.2 Common Patterns Quick Reference

```jsx
// Data fetching
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal }).then(/*...*/);
  return () => controller.abort();
}, [url]);

// Event listener
useEffect(() => {
  const handler = (e) => {/*...*/};
  window.addEventListener('event', handler);
  return () => window.removeEventListener('event', handler);
}, []);

// Timer
useEffect(() => {
  const id = setInterval(callback, delay);
  return () => clearInterval(id);
}, [callback, delay]);

// Debounced value
const debouncedValue = useDebounce(value, 300);

// Previous value
const prevValue = usePrevious(value);

// Toggle
const [isOpen, { toggle, setTrue, setFalse }] = useToggle();

// Click outside
useClickOutside(ref, onClose);

// Media query
const isMobile = useMediaQuery('(max-width: 768px)');

// Local storage
const [value, setValue] = useLocalStorage('key', initial);
```

---

# 21. TypeScript Patterns for Hooks

## 21.1 useState with TypeScript

```tsx
// Simple types (inferred)
const [count, setCount] = useState(0);         // number
const [name, setName] = useState('');          // string
const [active, setActive] = useState(false);   // boolean

// Complex types (explicit)
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
}

const [user, setUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);

// Union types for state machines
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');

// Discriminated unions for complex state
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

const [state, setState] = useState<FetchState<User>>({ status: 'idle' });

// Handle discriminated unions
if (state.status === 'success') {
  console.log(state.data.name); // Type-safe access
}
```

## 21.2 useReducer with TypeScript

```tsx
// State type
interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
  error: string | null;
}

// Action types (discriminated union)
type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number }
  | { type: 'SET_FILTER'; payload: TodoState['filter'] }
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: Todo[] }
  | { type: 'FETCH_ERROR'; payload: string };

// Reducer with type safety
function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: action.payload, done: false }]
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(t =>
          t.id === action.payload ? { ...t, done: !t.done } : t
        )
      };
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(t => t.id !== action.payload)
      };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, todos: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    default:
      // Exhaustiveness check
      const _exhaustive: never = action;
      return state;
  }
}

// Initial state
const initialState: TodoState = {
  todos: [],
  filter: 'all',
  loading: false,
  error: null
};

// Usage
const [state, dispatch] = useReducer(todoReducer, initialState);

// TypeScript ensures correct action shapes
dispatch({ type: 'ADD_TODO', payload: 'Buy milk' });      // ✅
dispatch({ type: 'TOGGLE_TODO', payload: 1 });            // ✅
dispatch({ type: 'SET_FILTER', payload: 'active' });      // ✅
dispatch({ type: 'SET_FILTER', payload: 'invalid' });     // ❌ Error!
dispatch({ type: 'UNKNOWN' });                             // ❌ Error!
```

## 21.3 useRef with TypeScript

```tsx
// DOM element refs
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);
const buttonRef = useRef<HTMLButtonElement>(null);
const formRef = useRef<HTMLFormElement>(null);
const canvasRef = useRef<HTMLCanvasElement>(null);

// Usage with null check
const focusInput = () => {
  inputRef.current?.focus();
};

// Mutable value refs (no null)
const countRef = useRef<number>(0);           // number, starts at 0
const timerRef = useRef<number | null>(null); // number | null
const prevValueRef = useRef<string>();         // string | undefined

// Callback refs
const measuredRef = useCallback((node: HTMLDivElement | null) => {
  if (node !== null) {
    console.log('Height:', node.getBoundingClientRect().height);
  }
}, []);
```

## 21.4 useContext with TypeScript

```tsx
interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// Create context with undefined default
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

// Provider component
function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = async (email: string, password: string) => {
    const data = await api.login(email, password);
    setUser(data.user);
  };
  
  const logout = () => setUser(null);
  
  const value: AuthContextValue = {
    user,
    isAuthenticated: !!user,
    login,
    logout
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook with type safety
function useAuth(): AuthContextValue {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
}

// Usage - fully typed
function Profile() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <p>{user?.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

## 21.5 Custom Hooks with TypeScript

```tsx
// Generic fetch hook
function useFetch<T>(url: string): {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
} {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  const [refreshIndex, setRefreshIndex] = useState(0);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url, { signal: controller.signal });
        if (!response.ok) throw new Error('Failed to fetch');
        const json = await response.json() as T;
        setData(json);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => controller.abort();
  }, [url, refreshIndex]);
  
  const refetch = useCallback(() => {
    setRefreshIndex(i => i + 1);
  }, []);
  
  return { data, loading, error, refetch };
}

// Usage with type inference
interface Post {
  id: number;
  title: string;
  body: string;
}

function Posts() {
  const { data, loading, error } = useFetch<Post[]>('/api/posts');
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  return (
    <ul>
      {data?.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// Generic async hook
function useAsync<T, Args extends unknown[] = []>(
  asyncFn: (...args: Args) => Promise<T>,
  immediate = false
) {
  const [state, setState] = useState<{
    status: 'idle' | 'pending' | 'success' | 'error';
    data: T | null;
    error: Error | null;
  }>({
    status: 'idle',
    data: null,
    error: null
  });
  
  const execute = useCallback(async (...args: Args) => {
    setState({ status: 'pending', data: null, error: null });
    
    try {
      const data = await asyncFn(...args);
      setState({ status: 'success', data, error: null });
      return data;
    } catch (error) {
      const err = error instanceof Error ? error : new Error(String(error));
      setState({ status: 'error', data: null, error: err });
      throw err;
    }
  }, [asyncFn]);
  
  return {
    ...state,
    execute,
    isIdle: state.status === 'idle',
    isLoading: state.status === 'pending',
    isSuccess: state.status === 'success',
    isError: state.status === 'error'
  };
}
```

## 21.6 Event Handler Types

```tsx
// Common event types
function EventHandlerExamples() {
  // Form events
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
  };
  
  // Input change
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };
  
  // Select change
  const handleSelect = (e: React.ChangeEvent<HTMLSelectElement>) => {
    console.log(e.target.value);
  };
  
  // Textarea change
  const handleTextarea = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    console.log(e.target.value);
  };
  
  // Click events
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log(e.currentTarget);
  };
  
  // Keyboard events
  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  };
  
  // Focus events
  const handleFocus = (e: React.FocusEvent<HTMLInputElement>) => {
    console.log('Focused');
  };
  
  // Drag events
  const handleDrag = (e: React.DragEvent<HTMLDivElement>) => {
    console.log(e.dataTransfer);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} onKeyDown={handleKeyDown} />
      <button onClick={handleClick}>Submit</button>
    </form>
  );
}
```

---

# 22. Anti-Patterns to Avoid

## 22.1 Derived State Anti-Pattern

```tsx
// ❌ Bad: Storing derived state
function BadComponent({ items }: { items: Item[] }) {
  const [filteredItems, setFilteredItems] = useState(items);
  
  useEffect(() => {
    setFilteredItems(items.filter(i => i.active));
  }, [items]);
  
  // Problem: Extra state and effect, potential sync issues
}

// ✅ Good: Compute derived values
function GoodComponent({ items }: { items: Item[] }) {
  const filteredItems = useMemo(
    () => items.filter(i => i.active),
    [items]
  );
  
  // Or just compute directly if cheap
  const activeCount = items.filter(i => i.active).length;
}
```

## 22.2 Prop-State Sync Anti-Pattern

```tsx
// ❌ Bad: Copying props to state
function BadComponent({ user }: { user: User }) {
  const [localUser, setLocalUser] = useState(user);
  
  useEffect(() => {
    setLocalUser(user);
  }, [user]);
  
  // Problem: Two sources of truth, complex sync
}

// ✅ Good: Use prop directly, or lift state
function GoodComponent({ user }: { user: User }) {
  // If you need local edits, key the component
  return <UserForm key={user.id} initialUser={user} />;
}
```

## 22.3 Effect for Event Handling Anti-Pattern

```tsx
// ❌ Bad: Effect triggered by state that's set by event
function BadComponent() {
  const [shouldSubmit, setShouldSubmit] = useState(false);
  const [formData, setFormData] = useState({});
  
  useEffect(() => {
    if (shouldSubmit) {
      submitForm(formData);
      setShouldSubmit(false);
    }
  }, [shouldSubmit, formData]);
  
  const handleSubmit = () => {
    setShouldSubmit(true);
  };
}

// ✅ Good: Handle in event handler
function GoodComponent() {
  const [formData, setFormData] = useState({});
  
  const handleSubmit = async () => {
    await submitForm(formData);
    // Clear form, navigate, etc.
  };
}
```

## 22.4 Overusing useMemo/useCallback

```tsx
// ❌ Bad: Memoizing everything
function OverlyMemoized({ items }) {
  const length = useMemo(() => items.length, [items]);  // Unnecessary
  const doubled = useMemo(() => count * 2, [count]);    // Unnecessary
  const handleClick = useCallback(() => {}, []);        // No benefit without memo
}

// ✅ Good: Only memoize when needed
function ProperlyMemoized({ items }) {
  const length = items.length;           // Just compute
  const doubled = count * 2;             // Just compute
  
  const sortedItems = useMemo(           // ✅ Expensive computation
    () => [...items].sort((a, b) => a.name.localeCompare(b.name)),
    [items]
  );
  
  const handleClick = useCallback(       // ✅ Passed to memo'd child
    (id: number) => dispatch({ type: 'SELECT', payload: id }),
    [dispatch]
  );
}
```

## 22.5 Putting Everything in One Effect

```tsx
// ❌ Bad: One giant effect for unrelated things
function BadComponent({ userId, theme }) {
  useEffect(() => {
    // User data
    fetchUser(userId).then(setUser);
    
    // Analytics
    trackPageView();
    
    // Theme
    document.body.className = theme;
    
    return () => {
      // Mixed cleanup
    };
  }, [userId, theme]);
}

// ✅ Good: Separate effects for separate concerns
function GoodComponent({ userId, theme }) {
  // User data
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // Analytics
  useEffect(() => {
    trackPageView();
  }, []);
  
  // Theme
  useEffect(() => {
    document.body.className = theme;
  }, [theme]);
}
```

---

# 23. Performance Best Practices

## 23.1 Avoid Unnecessary Re-renders

```tsx
// Use React.memo for expensive pure components
const ExpensiveList = React.memo(function ExpensiveList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <ExpensiveItem key={item.id} item={item} />
      ))}
    </ul>
  );
});

// Custom comparison function
const MemoWithCustomCompare = React.memo(
  function Component({ user, onClick }) {
    return <div onClick={onClick}>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Only re-render if user.id changes
    return prevProps.user.id === nextProps.user.id;
  }
);
```

## 23.2 Optimize Context

```tsx
// Split contexts by update frequency
const UserContext = createContext(null);      // Changes rarely
const ThemeContext = createContext('light');  // Changes sometimes
const UIContext = createContext({});          // Changes often

// Memo the value
function Provider({ children }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({
    user,
    setUser
  }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Separate state from dispatch
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function StoreProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Components that only dispatch never re-render
function AddButton() {
  const dispatch = useContext(DispatchContext);
  return <button onClick={() => dispatch({ type: 'ADD' })}>Add</button>;
}
```

## 23.3 Lazy Loading

```tsx
// Lazy load components
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart data={data} />
    </Suspense>
  );
}

// Lazy load on interaction
function App() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <>
      <button onClick={() => setShowModal(true)}>Open</button>
      {showModal && (
        <Suspense fallback={<Spinner />}>
          <HeavyModal onClose={() => setShowModal(false)} />
        </Suspense>
      )}
    </>
  );
}
```

---

# 24. Debugging Hooks

## 24.1 useDebugValue

```tsx
// Custom hook with debug label
function useOnlineStatus(): boolean {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  
  // Show in React DevTools
  useDebugValue(isOnline ? 'Online' : 'Offline');
  
  useEffect(() => {
    // ...
  }, []);
  
  return isOnline;
}

// With formatter (defer expensive formatting)
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  
  // Format only when inspected in DevTools
  useDebugValue(user, u => u ? `${u.name} (${u.id})` : 'No user');
  
  // ...
  
  return user;
}
```

## 24.2 Why Did You Render

```tsx
// Debug hook to track why component re-rendered
function useWhyDidYouUpdate(name: string, props: Record<string, any>) {
  const previousProps = useRef<Record<string, any>>({});
  
  useEffect(() => {
    if (previousProps.current) {
      const allKeys = Object.keys({ ...previousProps.current, ...props });
      const changesObj: Record<string, { from: any; to: any }> = {};
      
      allKeys.forEach(key => {
        if (previousProps.current[key] !== props[key]) {
          changesObj[key] = {
            from: previousProps.current[key],
            to: props[key]
          };
        }
      });
      
      if (Object.keys(changesObj).length) {
        console.log(`[${name}] Changed props:`, changesObj);
      }
    }
    
    previousProps.current = props;
  });
}

// Usage
function MyComponent(props) {
  useWhyDidYouUpdate('MyComponent', props);
  return <div>...</div>;
}
```

## 24.3 Render Counter

```tsx
function useRenderCount(componentName: string): void {
  const renderCount = useRef(0);
  renderCount.current += 1;
  
  useEffect(() => {
    console.log(`${componentName} rendered ${renderCount.current} times`);
  });
}

function MyComponent() {
  useRenderCount('MyComponent');
  // ...
}
```

---

# 25. Final Summary

## Key Takeaways

### Rules
1. Only call hooks at the top level
2. Only call hooks from React functions
3. Use the `eslint-plugin-react-hooks` for enforcement

### State Management
- Use `useState` for simple local state
- Use `useReducer` for complex state logic
- Use Context for cross-cutting concerns
- Consider external libraries for server state (TanStack Query)

### Performance
- Measure before optimizing
- Use `useMemo` for expensive calculations
- Use `useCallback` with `React.memo`
- Split contexts by update frequency

### Effects
- Effects are for synchronization with external systems
- Always include all dependencies
- Clean up subscriptions and timers
- Use AbortController for fetch requests

### Custom Hooks
- Extract reusable logic into custom hooks
- Start hook names with `use`
- Compose smaller hooks into larger ones
- Keep hooks focused and single-purpose

---

## Quick Reference Card

```
useState:    [value, setter] = useState(initial)
useReducer:  [state, dispatch] = useReducer(reducer, initial)
useContext:  value = useContext(Context)
useRef:      ref = useRef(initial)
useEffect:   useEffect(effect, deps?)
useMemo:     value = useMemo(create, deps)
useCallback: fn = useCallback(fn, deps)

React 18+:
useTransition:     [isPending, start] = useTransition()
useDeferredValue:  deferred = useDeferredValue(value)
useId:            id = useId()
```

---

# 26. Bonus: Quick Answers for Interviews

## 26.1 One-Liners

**What are hooks?**
Functions that let you use React features (state, lifecycle) in function components.

**Why were hooks introduced?**
To reuse stateful logic, group related code, and avoid class complexity (this binding).

**What are the two rules of hooks?**
1) Only call at top level (no loops/conditions); 2) Only call from React functions.

**useState vs useReducer?**
useState for simple state; useReducer for complex state with multiple related values.

**useEffect vs useLayoutEffect?**
useEffect is async after paint; useLayoutEffect is sync before paint (for DOM measurements).

**useMemo vs useCallback?**
useMemo caches a value; useCallback caches a function (they're nearly identical).

**What is a custom hook?**
A function starting with "use" that calls other hooks to encapsulate and reuse logic.

**What is the dependency array?**
An array that tells React when to re-run effects/memos; uses Object.is for comparison.

**What causes infinite loops?**
Missing deps, objects/arrays in deps (new references), or unconditional state updates.

**How to fix stale closures?**
Use functional updates (setState(prev => ...)) or useRef to hold current values.

## 26.2 Common Mistakes Quick Reference

| Mistake | Solution |
|---------|----------|
| Mutating state directly | Create new object/array |
| Missing effect deps | Include all dependencies |
| Object in deps | Use useMemo or primitive deps |
| No cleanup in effect | Return cleanup function |
| Index as list key | Use stable unique ID |
| Async in useEffect | Define async function inside effect |
| State after unmount | Use cleanup flag or AbortController |

## 26.3 When to Use Each Hook

| Scenario | Hook |
|----------|------|
| Simple local state | useState |
| Complex state logic | useReducer |
| Global state | useContext |
| DOM reference | useRef |
| Side effects | useEffect |
| DOM measurements | useLayoutEffect |
| Expensive calculation | useMemo |
| Callback for memoized child | useCallback |
| Customize ref API | useImperativeHandle |
| Non-urgent updates | useTransition |
| Debounced render | useDeferredValue |
| Unique ID for a11y | useId |

## 26.4 Custom Hook Recipe Templates

```jsx
// Value-based hook
function useValue(initial) {
  const [value, setValue] = useState(initial);
  return [value, setValue];
}

// Effect-based hook
function useEffect_Hook(dep) {
  useEffect(() => {
    // setup
    return () => { /* cleanup */ };
  }, [dep]);
}

// Ref-based hook
function useRef_Hook(initial) {
  const ref = useRef(initial);
  // Use ref.current
  return ref;
}

// Composed hook
function useComposed(input) {
  const [state, dispatch] = useReducer(reducer, init);
  useEffect(() => { /* ... */ }, [input]);
  return { state, dispatch };
}
```

---

## 🎯 Final Study Checklist

### Fundamentals
- [ ] Understand rules of hooks
- [ ] useState (lazy init, functional updates, object/array state)
- [ ] useEffect (deps, cleanup, async patterns)
- [ ] useRef (DOM, mutable values, previous value)

### Optimization
- [ ] useMemo (expensive calculations)
- [ ] useCallback (stable function references)
- [ ] React.memo (prevent re-renders)
- [ ] Context optimization (split, memoize value)

### Advanced
- [ ] useReducer (TypeScript patterns, complex logic)
- [ ] useContext (custom hook pattern)
- [ ] useLayoutEffect (DOM measurements)
- [ ] useImperativeHandle (custom ref API)

### React 18+
- [ ] useTransition (non-blocking updates)
- [ ] useDeferredValue (deferred rendering)
- [ ] useId (accessible unique IDs)
- [ ] useSyncExternalStore (external state)

### Custom Hooks
- [ ] useDebounce, usePrevious, useToggle
- [ ] useFetch, useLocalStorage
- [ ] useClickOutside, useKeyPress
- [ ] useIntersectionObserver, useMediaQuery

### Testing
- [ ] renderHook from @testing-library/react
- [ ] act() for state updates
- [ ] waitFor() for async operations
- [ ] Mocking and wrapper patterns

---

*End of Part 4: React Hooks Complete Guide (6000+ lines)*

*Continue to Part 5 for Next.js Framework Complete Guide...*

