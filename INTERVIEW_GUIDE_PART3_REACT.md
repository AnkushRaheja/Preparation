# 📚 Part 3: React Deep Dive - Complete Interview Guide

> **The Ultimate React Interview Preparation Guide**
> 
> This comprehensive guide covers everything you need to master React for technical interviews at any level - from junior to senior positions at top tech companies.

---

## 📋 Table of Contents

1. [React Fundamentals](#1-react-fundamentals)
2. [Components Deep Dive](#2-components-deep-dive)
3. [JSX In Depth](#3-jsx-in-depth)
4. [State Management](#4-state-management)
5. [Props & Data Flow](#5-props--data-flow)
6. [Component Lifecycle](#6-component-lifecycle)
7. [React Hooks Complete Guide](#7-react-hooks-complete-guide)
8. [Context API](#8-context-api)
9. [Performance Optimization](#9-performance-optimization)
10. [Patterns & Best Practices](#10-patterns--best-practices)
11. [Error Handling](#11-error-handling)
12. [Testing React Applications](#12-testing-react-applications)
13. [React Router](#13-react-router)
14. [Server-Side Rendering](#14-server-side-rendering)
15. [Common Interview Questions](#15-common-interview-questions)
16. [Coding Challenges](#16-coding-challenges)

---

# 1. React Fundamentals

## 1.1 What is React?

React is a **JavaScript library** for building user interfaces, developed by Facebook (Meta). It uses a declarative, component-based approach to create efficient and predictable UI.

### Key Features

1. **Declarative**: Describe what the UI should look like, and React handles the updates
2. **Component-Based**: Build encapsulated components that manage their own state
3. **Virtual DOM**: Efficient diffing algorithm for optimal DOM updates
4. **Unidirectional Data Flow**: Data flows down from parent to child components
5. **JSX**: JavaScript syntax extension for writing UI components

### React vs Other Frameworks

| Feature | React | Angular | Vue |
|---------|-------|---------|-----|
| Type | Library | Framework | Framework |
| Learning Curve | Moderate | Steep | Gentle |
| Size | ~45KB | ~500KB | ~80KB |
| Data Binding | One-way | Two-way | Both |
| DOM | Virtual | Real + Change Detection | Virtual |
| Language | JavaScript/TypeScript | TypeScript | JavaScript/TypeScript |

## 1.2 Virtual DOM

The Virtual DOM is a lightweight JavaScript representation of the Real DOM.

```jsx
// React creates a Virtual DOM representation like this:
const virtualElement = {
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: {
          children: 'Hello World'
        }
      },
      {
        type: 'p',
        props: {
          children: 'Welcome to React'
        }
      }
    ]
  }
};
```

### How Virtual DOM Works

1. **Initial Render**: React creates a Virtual DOM tree representing the UI
2. **State Change**: When state changes, React creates a new Virtual DOM tree
3. **Diffing**: React compares the new tree with the previous one (reconciliation)
4. **Batching**: React batches multiple updates for efficiency
5. **Patching**: Only the changed nodes are updated in the Real DOM

```jsx
// Example: Understanding Virtual DOM updates
function Counter() {
  const [count, setCount] = useState(0);
  
  // When setCount is called:
  // 1. New Virtual DOM is created with updated count
  // 2. React diffs: only the text node with count changes
  // 3. Only that text node is updated in real DOM
  
  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### Reconciliation Algorithm (React Fiber)

React Fiber is the reconciliation algorithm introduced in React 16:

```jsx
// Key concepts in React Fiber:

// 1. Work can be paused and resumed
// 2. Different priorities for different updates
// 3. Ability to abort work if no longer needed

// Priority levels (internal):
// - Immediate (synchronous)
// - User-blocking (clicks, input)
// - Normal (data fetching)
// - Low (analytics)
// - Idle (offscreen updates)

// This is why concurrent features work:
function App() {
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    // High priority: Update input immediately
    setInput(e.target.value);
    
    // Low priority: Can be interrupted
    startTransition(() => {
      setSearchResults(filterResults(e.target.value));
    });
  };
  
  return (
    <>
      <input onChange={handleChange} />
      {isPending ? <Spinner /> : <Results data={searchResults} />}
    </>
  );
}
```

## 1.3 Creating a React Application

### Create React App (CRA)

```bash
# Create new project
npx create-react-app my-app
npx create-react-app my-app --template typescript

# Project structure
my-app/
├── node_modules/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json
├── src/
│   ├── App.js
│   ├── App.css
│   ├── index.js
│   ├── index.css
│   └── components/
├── package.json
└── README.md
```

### Vite (Recommended for new projects)

```bash
# Create with Vite
npm create vite@latest my-app -- --template react
npm create vite@latest my-app -- --template react-ts

# Vite project structure
my-app/
├── node_modules/
├── public/
├── src/
│   ├── App.jsx
│   ├── App.css
│   ├── main.jsx
│   └── index.css
├── index.html
├── package.json
└── vite.config.js
```

### Entry Point

```jsx
// src/main.jsx (Vite) or src/index.js (CRA)
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

// React 18 createRoot API
const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// StrictMode helps identify:
// - Components with unsafe lifecycles
// - Legacy string ref usage
// - Deprecated findDOMNode usage
// - Unexpected side effects
// - Legacy context API usage
```

---

# 2. Components Deep Dive

## 2.1 Function Components

Function components are the modern way to write React components.

```jsx
// Basic function component
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Arrow function syntax
const Greeting = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// Implicit return (single expression)
const Greeting = ({ name }) => <h1>Hello, {name}!</h1>;

// With TypeScript
interface GreetingProps {
  name: string;
  age?: number;
}

const Greeting: React.FC<GreetingProps> = ({ name, age }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>You are {age} years old</p>}
    </div>
  );
};

// Preferred TypeScript syntax (explicit return type)
function Greeting({ name, age }: GreetingProps): JSX.Element {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>You are {age} years old</p>}
    </div>
  );
}
```

## 2.2 Class Components (Legacy)

Class components are still supported but function components with hooks are preferred.

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  // Constructor for initializing state
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
    
    // Binding methods (required for class methods)
    this.handleIncrement = this.handleIncrement.bind(this);
  }
  
  // Alternative: Class field syntax (no binding needed)
  handleDecrement = () => {
    this.setState(prevState => ({
      count: prevState.count - 1
    }));
  };
  
  handleIncrement() {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  // Lifecycle method
  componentDidMount() {
    console.log('Component mounted');
  }
  
  componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
      console.log('Count changed:', this.state.count);
    }
  }
  
  componentWillUnmount() {
    console.log('Component will unmount');
  }
  
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleIncrement}>+</button>
        <button onClick={this.handleDecrement}>-</button>
      </div>
    );
  }
}

// With TypeScript
interface CounterProps {
  initialCount?: number;
}

interface CounterState {
  count: number;
}

class Counter extends Component<CounterProps, CounterState> {
  state: CounterState = {
    count: this.props.initialCount || 0
  };
  
  render() {
    return <div>{this.state.count}</div>;
  }
}
```

## 2.3 Pure Components and React.memo

### PureComponent (Class components)

```jsx
import React, { PureComponent } from 'react';

// PureComponent implements shouldComponentUpdate with shallow comparison
class UserCard extends PureComponent {
  render() {
    console.log('UserCard rendered');
    return (
      <div>
        <h2>{this.props.name}</h2>
        <p>{this.props.email}</p>
      </div>
    );
  }
}

// Equivalent shouldComponentUpdate implementation
class UserCard extends Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Shallow comparison of props and state
    return (
      nextProps.name !== this.props.name ||
      nextProps.email !== this.props.email
    );
  }
  
  render() {
    return (
      <div>
        <h2>{this.props.name}</h2>
        <p>{this.props.email}</p>
      </div>
    );
  }
}
```

### React.memo (Function components)

```jsx
import React, { memo } from 'react';

// Basic memo - shallow comparison of props
const UserCard = memo(function UserCard({ name, email }) {
  console.log('UserCard rendered');
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
});

// Custom comparison function
const UserCard = memo(
  function UserCard({ user, onClick }) {
    return (
      <div onClick={onClick}>
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name
    );
  }
);

// When to use React.memo:
// ✅ Pure functional components with same props = same output
// ✅ Component renders often with same props
// ✅ Component has expensive rendering logic
// ❌ Props change frequently
// ❌ Simple components (memo overhead > re-render cost)
```

## 2.4 Controlled vs Uncontrolled Components

### Controlled Components

React state is the "single source of truth" for input values.

```jsx
function ControlledForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
        placeholder="Message"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Uncontrolled Components

DOM itself maintains the state, accessed via refs.

```jsx
import { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);
  const fileRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', {
      name: nameRef.current.value,
      email: emailRef.current.value,
      file: fileRef.current.files[0]
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        ref={nameRef}
        defaultValue="Default Name"  // Use defaultValue, not value
        placeholder="Name"
      />
      <input
        type="email"
        ref={emailRef}
        placeholder="Email"
      />
      {/* File inputs must be uncontrolled */}
      <input
        type="file"
        ref={fileRef}
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// Comparison:
// Controlled:
// ✅ Immediate access to form values
// ✅ Easy validation and formatting
// ✅ Conditional disabling of submit
// ❌ More code, re-renders on every keystroke

// Uncontrolled:
// ✅ Less code
// ✅ Better for simple forms
// ✅ Required for file inputs
// ❌ Harder to validate
// ❌ No instant access to values
```

## 2.5 Higher-Order Components (HOC)

A HOC is a function that takes a component and returns a new enhanced component.

```jsx
// Basic HOC pattern
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div className="spinner">Loading...</div>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);

<UserListWithLoading isLoading={isLoading} users={users} />

// HOC with configuration
function withAuth(WrappedComponent, { redirectTo = '/login' } = {}) {
  return function WithAuthComponent(props) {
    const { user, isLoading } = useAuth();
    const navigate = useNavigate();
    
    useEffect(() => {
      if (!isLoading && !user) {
        navigate(redirectTo);
      }
    }, [user, isLoading, navigate]);
    
    if (isLoading) {
      return <LoadingSpinner />;
    }
    
    if (!user) {
      return null;
    }
    
    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedSettings = withAuth(Settings, { redirectTo: '/signin' });

// Multiple HOCs
const EnhancedComponent = withAuth(withLoading(withTheme(MyComponent)));

// Or using compose
import { compose } from 'redux'; // or write your own compose

const enhance = compose(
  withAuth,
  withLoading,
  withTheme
);

const EnhancedComponent = enhance(MyComponent);
```

## 2.6 Render Props Pattern

A render prop is a prop whose value is a function that returns React elements.

```jsx
// Render props pattern
class MouseTracker extends Component {
  state = { x: 0, y: 0 };
  
  handleMouseMove = (e) => {
    this.setState({ x: e.clientX, y: e.clientY });
  };
  
  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <p>Mouse position: {x}, {y}</p>
  )}
/>

// Function as children (children prop as render prop)
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };
  
  return (
    <div onMouseMove={handleMouseMove}>
      {children(position)}
    </div>
  );
}

// Usage
<MouseTracker>
  {({ x, y }) => <p>Mouse position: {x}, {y}</p>}
</MouseTracker>

// Practical example: Data fetching
function DataFetcher({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [url]);
  
  return children({ data, loading, error });
}

// Usage
<DataFetcher url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <UserList users={data} />;
  }}
</DataFetcher>
```

## 2.7 Compound Components Pattern

Components that work together to form a complete UI.

```jsx
import { createContext, useContext, useState } from 'react';

// Create context for compound components
const TabsContext = createContext();

function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  
  return (
    <button
      className={`tab ${activeTab === id ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ id, children }) {
  const { activeTab } = useContext(TabsContext);
  
  if (activeTab !== id) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Attach sub-components
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;
Tabs.TabPanel = TabPanel;

// Usage
function App() {
  return (
    <Tabs defaultTab="tab1">
      <Tabs.TabList>
        <Tabs.Tab id="tab1">Tab 1</Tabs.Tab>
        <Tabs.Tab id="tab2">Tab 2</Tabs.Tab>
        <Tabs.Tab id="tab3">Tab 3</Tabs.Tab>
      </Tabs.TabList>
      
      <Tabs.TabPanels>
        <Tabs.TabPanel id="tab1">Content for Tab 1</Tabs.TabPanel>
        <Tabs.TabPanel id="tab2">Content for Tab 2</Tabs.TabPanel>
        <Tabs.TabPanel id="tab3">Content for Tab 3</Tabs.TabPanel>
      </Tabs.TabPanels>
    </Tabs>
  );
}
```

---

# 3. JSX In Depth

## 3.1 What is JSX?

JSX is a syntax extension for JavaScript that looks like HTML but produces React elements.

```jsx
// JSX
const element = <h1 className="greeting">Hello, World!</h1>;

// Compiles to (React 17+ with new JSX transform)
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { className: 'greeting', children: 'Hello, World!' });

// Old transform (React 16 and earlier)
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, World!'
);
```

## 3.2 JSX Rules

```jsx
// 1. Return a single root element
// ❌ This will error
function App() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}

// ✅ Wrap in a parent element
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// ✅ Or use a Fragment
function App() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}

// 2. Close all tags
// ❌ Self-closing tags must have /
<img src="image.jpg">
<input type="text">

// ✅ Correct
<img src="image.jpg" />
<input type="text" />

// 3. Use camelCase for attributes
// ❌ HTML attributes
<div class="container" onclick="handleClick()">
  <label for="name">Name</label>
</div>

// ✅ JSX uses camelCase
<div className="container" onClick={handleClick}>
  <label htmlFor="name">Name</label>
</div>

// 4. JavaScript expressions in curly braces
const name = "Jayesh";
const items = ['Apple', 'Banana', 'Cherry'];

<div>
  <h1>Hello, {name}</h1>
  <p>2 + 2 = {2 + 2}</p>
  <p>Date: {new Date().toLocaleDateString()}</p>
  <ul>
    {items.map((item, index) => (
      <li key={index}>{item}</li>
    ))}
  </ul>
</div>
```

## 3.3 Conditional Rendering

```jsx
// 1. If/else (outside JSX)
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  return <h1>Please sign in.</h1>;
}

// 2. Ternary operator
function Greeting({ isLoggedIn }) {
  return (
    <h1>
      {isLoggedIn ? 'Welcome back!' : 'Please sign in.'}
    </h1>
  );
}

// 3. Logical AND (&&)
function Notification({ hasMessages, count }) {
  return (
    <div>
      {hasMessages && <span>You have {count} messages</span>}
    </div>
  );
}

// ⚠️ Be careful with && and falsy values
function BadExample({ count }) {
  // If count is 0, React will render "0" instead of nothing!
  return <div>{count && <span>{count} items</span>}</div>;
}

// ✅ Fix: Explicit boolean check
function GoodExample({ count }) {
  return <div>{count > 0 && <span>{count} items</span>}</div>;
}

// 4. Nullish coalescing
function Username({ name }) {
  return <span>{name ?? 'Anonymous'}</span>;
}

// 5. Early return pattern
function UserProfile({ user, isLoading, error }) {
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound />;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 6. Switch/case pattern
function StatusIcon({ status }) {
  const icons = {
    pending: '⏳',
    success: '✅',
    error: '❌',
    warning: '⚠️'
  };
  
  return <span>{icons[status] ?? '❓'}</span>;
}
```

## 3.4 Lists and Keys

```jsx
// Rendering lists with map()
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input 
            type="checkbox" 
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Keys must be:
// 1. Unique among siblings (not globally)
// 2. Stable (don't use Math.random())
// 3. Predictable (prefer IDs over indexes)

// ❌ Bad: Using index as key (causes issues with reordering)
{items.map((item, index) => <Item key={index} data={item} />)}

// ❌ Bad: Using random values
{items.map(item => <Item key={Math.random()} data={item} />)}

// ✅ Good: Using unique IDs
{items.map(item => <Item key={item.id} data={item} />)}

// When to use index as key:
// - List is static (no add/remove/reorder)
// - Items have no stable IDs
// - List will never be reordered or filtered

// Key extraction in nested lists
function NestedList({ categories }) {
  return (
    <div>
      {categories.map(category => (
        <div key={category.id}>
          <h2>{category.name}</h2>
          <ul>
            {category.items.map(item => (
              <li key={item.id}>{item.name}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}

// Extracting components from lists
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </ul>
  );
}

function ListItem({ item }) {
  return (
    <li>
      <h3>{item.title}</h3>
      <p>{item.description}</p>
    </li>
  );
}
```

## 3.5 Fragments

```jsx
import { Fragment } from 'react';

// Short syntax (cannot pass key)
function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  );
}

// Long syntax (can pass key)
function Glossary({ items }) {
  return (
    <dl>
      {items.map(item => (
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}

// Table rows need Fragments
function TableRows({ data }) {
  return (
    <>
      {data.map(row => (
        <tr key={row.id}>
          <td>{row.name}</td>
          <td>{row.value}</td>
        </tr>
      ))}
    </>
  );
}
```

## 3.6 Spread Attributes

```jsx
// Spreading props
function Button({ children, ...rest }) {
  return <button {...rest}>{children}</button>;
}

<Button className="primary" onClick={handleClick} disabled={isLoading}>
  Submit
</Button>

// Forwarding all props
function Input(props) {
  return <input className="custom-input" {...props} />;
}

// Props override (order matters)
function Input({ className = '', ...props }) {
  return (
    <input 
      className={`base-input ${className}`}
      type="text"  // Can be overridden by props
      {...props}   // Spread after default props
    />
  );
}

// Picking specific props
function Card({ title, children, className, ...containerProps }) {
  return (
    <div className={`card ${className}`} {...containerProps}>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

---

# 4. State Management

## 4.1 useState Hook

```jsx
import { useState } from 'react';

// Basic usage
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// With TypeScript
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);

// Lazy initialization (for expensive computations)
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});

// Object state
function Form() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateField = (field, value) => {
    setForm(prev => ({
      ...prev,
      [field]: value
    }));
  };
  
  return (
    <div>
      <input
        value={form.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      <input
        value={form.email}
        onChange={(e) => updateField('email', e.target.value)}
      />
    </div>
  );
}

// Array state
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  };
  
  const removeTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  return (/* ... */);
}

// Functional updates (when new state depends on previous)
function Counter() {
  const [count, setCount] = useState(0);
  
  const incrementThree = () => {
    // ❌ Wrong: All three use the same stale count value
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // Result: count only increases by 1
    
    // ✅ Correct: Each uses updated value
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // Result: count increases by 3
  };
}
```

## 4.2 useReducer Hook

```jsx
import { useReducer } from 'react';

// Reducer function
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

// Component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}

// With TypeScript
interface State {
  count: number;
  error: string | null;
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET'; payload: number }
  | { type: 'ERROR'; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET':
      return { ...state, count: action.payload };
    case 'ERROR':
      return { ...state, error: action.payload };
    default:
      return state;
  }
}

// Complex example: Todo app
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number }
  | { type: 'SET_FILTER'; payload: TodoState['filter'] }
  | { type: 'CLEAR_COMPLETED' };

function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false }
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
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
    default:
      return state;
  }
}

// Lazy initialization
const [state, dispatch] = useReducer(reducer, initialArg, init);

function init(initialCount) {
  return { count: initialCount };
}

const [state, dispatch] = useReducer(reducer, 0, init);
```

## 4.3 useState vs useReducer

```jsx
// Use useState when:
// - Simple state (primitives, simple objects)
// - Independent state updates
// - Few state transitions
// - State logic is simple

function SimpleForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  return (
    <form>
      <input value={name} onChange={e => setName(e.target.value)} />
      <input value={email} onChange={e => setEmail(e.target.value)} />
    </form>
  );
}

// Use useReducer when:
// - Complex state (objects with many fields)
// - Related state updates that should happen together
// - Complex state transitions
// - Need to test state logic separately
// - Many actions that modify state in different ways

function ComplexForm() {
  const [state, dispatch] = useReducer(formReducer, {
    name: '',
    email: '',
    errors: {},
    isSubmitting: false,
    isSuccess: false
  });
  
  const handleSubmit = async () => {
    dispatch({ type: 'SUBMIT_START' });
    try {
      await submitForm(state);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: error.message });
    }
  };
  
  return (/* ... */);
}
```

---

# 5. Props & Data Flow

## 5.1 Passing Props

```jsx
// Basic props
function Welcome({ name, age }) {
  return <h1>Hello, {name}! You are {age} years old.</h1>;
}

<Welcome name="Jayesh" age={25} />

// Default props
function Button({ type = 'button', children, variant = 'primary' }) {
  return (
    <button type={type} className={`btn btn-${variant}`}>
      {children}
    </button>
  );
}

// With TypeScript defaults
interface ButtonProps {
  type?: 'button' | 'submit' | 'reset';
  variant?: 'primary' | 'secondary' | 'danger';
  children: React.ReactNode;
}

function Button({ 
  type = 'button', 
  variant = 'primary', 
  children 
}: ButtonProps) {
  return (
    <button type={type} className={`btn btn-${variant}`}>
      {children}
    </button>
  );
}

// Required vs optional props
interface UserCardProps {
  name: string;           // Required
  email: string;          // Required
  avatar?: string;        // Optional
  role?: 'admin' | 'user';// Optional with specific values
}

// Children prop
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

<Card title="User Info">
  <p>This is the card content</p>
  <button>Click me</button>
</Card>
```

## 5.2 Prop Types (Runtime Validation)

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email, isAdmin, onClick, children }) {
  return (
    <div onClick={onClick}>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      {isAdmin && <span>Admin</span>}
      {children}
    </div>
  );
}

UserCard.propTypes = {
  // Basic types
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string,
  isAdmin: PropTypes.bool,
  
  // Functions
  onClick: PropTypes.func,
  
  // Children
  children: PropTypes.node,
  
  // Specific values
  status: PropTypes.oneOf(['active', 'inactive', 'pending']),
  
  // Multiple types
  id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]),
  
  // Arrays
  items: PropTypes.arrayOf(PropTypes.string),
  
  // Objects with shape
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string
  }),
  
  // Exact shape (no extra properties)
  config: PropTypes.exact({
    host: PropTypes.string,
    port: PropTypes.number
  }),
  
  // Instance of a class
  error: PropTypes.instanceOf(Error),
  
  // Custom validator
  customProp: function(props, propName, componentName) {
    if (!/^[A-Z]/.test(props[propName])) {
      return new Error(
        `${propName} in ${componentName} must start with uppercase`
      );
    }
  }
};

UserCard.defaultProps = {
  isAdmin: false,
  email: 'N/A'
};
```

## 5.3 Lifting State Up

When multiple components need to share state, lift it to their closest common ancestor.

```jsx
// Before: State in individual components (can't sync)
function TemperatureInput() {
  const [temperature, setTemperature] = useState('');
  
  return (
    <input
      value={temperature}
      onChange={(e) => setTemperature(e.target.value)}
    />
  );
}

// After: State lifted to common ancestor
function TemperatureCalculator() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const handleCelsiusChange = (temp) => {
    setScale('c');
    setTemperature(temp);
  };
  
  const handleFahrenheitChange = (temp) => {
    setScale('f');
    setTemperature(temp);
  };
  
  const celsius = scale === 'f' 
    ? tryConvert(temperature, toCelsius) 
    : temperature;
  const fahrenheit = scale === 'c' 
    ? tryConvert(temperature, toFahrenheit) 
    : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange}
      />
      <TemperatureInput
        scale="f"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange}
      />
      <BoilingVerdict celsius={parseFloat(celsius)} />
    </div>
  );
}

function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  return (
    <fieldset>
      <legend>Enter temperature in {scale === 'c' ? 'Celsius' : 'Fahrenheit'}:</legend>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}
```

## 5.4 Prop Drilling and Solutions

```jsx
// Problem: Prop drilling (passing props through many levels)
function App() {
  const [user, setUser] = useState(null);
  
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return (
    <div>
      <Header user={user} setUser={setUser} />
      <Main user={user} />
    </div>
  );
}

function Header({ user, setUser }) {
  return (
    <header>
      <Navbar user={user} setUser={setUser} />
    </header>
  );
}

function Navbar({ user, setUser }) {
  return (
    <nav>
      <UserMenu user={user} setUser={setUser} />
    </nav>
  );
}

// Solution 1: Context API
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const { user, setUser } = useContext(UserContext);
  // Use user and setUser directly
}

// Solution 2: Component Composition
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <Layout>
      <Header>
        <UserMenu user={user} setUser={setUser} />
      </Header>
      <Main>
        <Dashboard user={user} />
      </Main>
    </Layout>
  );
}

function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function Header({ children }) {
  return <header>{children}</header>;
}
```

---

# 6. Component Lifecycle

## 6.1 Class Component Lifecycle

```jsx
class LifecycleDemo extends Component {
  // ============ MOUNTING ============
  
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log('1. constructor');
    // Use for: Initializing state, binding methods
    // Don't: Make API calls, subscriptions
  }
  
  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps');
    // Rarely needed - for when state depends on props over time
    // Return object to update state, or null
    return null;
  }
  
  componentDidMount() {
    console.log('4. componentDidMount');
    // Use for: API calls, subscriptions, DOM manipulation
    // Component is now in the DOM
  }
  
  render() {
    console.log('3. render');
    // Pure function - no side effects
    return <div>{this.state.count}</div>;
  }
  
  // ============ UPDATING ============
  
  shouldComponentUpdate(nextProps, nextState) {
    console.log('5. shouldComponentUpdate');
    // Return false to skip re-render
    // Default: return true
    return nextState.count !== this.state.count;
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('7. getSnapshotBeforeUpdate');
    // Capture info from DOM before update
    // Return value passed to componentDidUpdate
    return null;
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('8. componentDidUpdate');
    // Use for: API calls based on prop/state changes
    // Always compare previous values to avoid infinite loops
    if (prevState.count !== this.state.count) {
      // Do something
    }
  }
  
  // ============ UNMOUNTING ============
  
  componentWillUnmount() {
    console.log('9. componentWillUnmount');
    // Use for: Cleanup (subscriptions, timers, etc.)
  }
  
  // ============ ERROR HANDLING ============
  
  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    // Log error to service
    logErrorToService(error, info.componentStack);
  }
}
```

## 6.2 Hooks Equivalents

```jsx
import { useState, useEffect, useLayoutEffect, useRef } from 'react';

function HooksLifecycle() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  // componentDidMount + componentDidUpdate + componentWillUnmount
  useEffect(() => {
    console.log('Effect runs after render');
    
    // Cleanup function (componentWillUnmount equivalent)
    return () => {
      console.log('Cleanup before next effect or unmount');
    };
  }); // Runs after every render
  
  // componentDidMount only
  useEffect(() => {
    console.log('Runs once on mount');
    
    return () => {
      console.log('Runs once on unmount');
    };
  }, []); // Empty dependency array
  
  // componentDidUpdate (for specific value)
  useEffect(() => {
    console.log('Count changed to:', count);
  }, [count]); // Only runs when count changes
  
  // Track previous value (like prevProps/prevState)
  useEffect(() => {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;
  
  // getSnapshotBeforeUpdate equivalent
  useLayoutEffect(() => {
    // Runs synchronously after DOM mutations, before paint
    // Use for DOM measurements
  });
  
  return <div>{count}</div>;
}

// Lifecycle comparison table:
// Class                          | Hooks
// -------------------------------|---------------------------
// constructor                    | useState, useRef
// getDerivedStateFromProps       | useState + useEffect
// componentDidMount              | useEffect(() => {}, [])
// componentDidUpdate             | useEffect(() => {}, [deps])
// componentWillUnmount           | useEffect cleanup return
// shouldComponentUpdate          | React.memo
// getSnapshotBeforeUpdate        | useLayoutEffect
// componentDidCatch              | Error Boundaries (no hook)
```

---

# 7. React Hooks Complete Guide

## 7.1 Rules of Hooks

```jsx
// 1. Only call hooks at the top level
// ❌ Don't call hooks inside loops, conditions, or nested functions
function BadComponent({ condition }) {
  if (condition) {
    const [value, setValue] = useState(0);  // ❌ Conditional hook
  }
  
  for (let i = 0; i < 5; i++) {
    useEffect(() => {});  // ❌ Loop hook
  }
  
  function nestedFunction() {
    useContext(MyContext);  // ❌ Nested function hook
  }
}

// ✅ Call hooks at the top level
function GoodComponent({ condition }) {
  const [value, setValue] = useState(0);
  
  useEffect(() => {
    if (condition) {
      // Conditional logic inside hook is fine
    }
  }, [condition]);
  
  return <div>{condition ? value : 'No value'}</div>;
}

// 2. Only call hooks from React functions
// ✅ Function components
function MyComponent() {
  useState(); // OK
}

// ✅ Custom hooks
function useCustomHook() {
  useState(); // OK
}

// ❌ Regular JavaScript functions
function regularFunction() {
  useState(); // Error
}
```

## 7.2 useState in Detail

```jsx
// State batching (React 18+)
function BatchingExample() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  const handleClick = () => {
    // React 18 batches these updates (single re-render)
    setCount(c => c + 1);
    setFlag(f => !f);
    // Only one re-render happens
  };
  
  const handleAsync = async () => {
    await someAsyncOperation();
    // React 18 also batches inside async functions!
    setCount(c => c + 1);
    setFlag(f => !f);
  };
  
  // To opt out of batching (rarely needed)
  const handleFlush = () => {
    flushSync(() => {
      setCount(c => c + 1);  // Forces immediate render
    });
    flushSync(() => {
      setFlag(f => !f);      // Forces another immediate render
    });
  };
}

// Immutable updates
function ImmutableUpdates() {
  const [user, setUser] = useState({
    name: 'Jayesh',
    address: {
      city: 'Mumbai',
      country: 'India'
    }
  });
  
  // ❌ Mutating state directly
  const badUpdate = () => {
    user.name = 'John';  // ❌ Mutation
    setUser(user);       // Won't trigger re-render
  };
  
  // ✅ Immutable update
  const goodUpdate = () => {
    setUser({
      ...user,
      name: 'John'
    });
  };
  
  // ✅ Nested object update
  const updateCity = () => {
    setUser({
      ...user,
      address: {
        ...user.address,
        city: 'Delhi'
      }
    });
  };
}
```

## 7.3 useEffect in Detail

```jsx
// Effect dependencies explained
function EffectDependencies({ userId }) {
  const [user, setUser] = useState(null);
  
  // ❌ Missing dependency - stale closure problem
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []);  // userId should be in dependencies
  
  // ✅ Correct dependencies
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // ❌ Object/array in dependencies causes infinite loop
  const config = { userId };
  useEffect(() => {
    fetchUser(config.userId);
  }, [config]);  // New object reference every render!
  
  // ✅ Use primitive values or useMemo
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);  // Primitive, stable reference
}

// Cleanup patterns
function CleanupPatterns() {
  // Timer cleanup
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  // Event listener cleanup
  useEffect(() => {
    const handleResize = () => console.log(window.innerWidth);
    window.addEventListener('resize', handleResize);
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  // Subscription cleanup
  useEffect(() => {
    const unsubscribe = someObservable.subscribe(data => {
      console.log(data);
    });
    
    return () => unsubscribe();
  }, []);
  
  // Abort controller for fetch
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      });
    
    return () => controller.abort();
  }, [url]);
}
```

## 7.4 useRef in Detail

```jsx
// DOM references
function DOMRef() {
  const inputRef = useRef(null);
  const videoRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  const playVideo = () => {
    videoRef.current?.play();
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
      
      <video ref={videoRef} src="video.mp4" />
      <button onClick={playVideo}>Play</button>
    </div>
  );
}

// Mutable values (no re-render on change)
function MutableRef() {
  const renderCount = useRef(0);
  const previousValue = useRef();
  const intervalId = useRef(null);
  
  useEffect(() => {
    renderCount.current += 1;
    console.log(`Rendered ${renderCount.current} times`);
  });
  
  // Store interval ID
  const startTimer = () => {
    intervalId.current = setInterval(() => {
      console.log('Tick');
    }, 1000);
  };
  
  const stopTimer = () => {
    clearInterval(intervalId.current);
  };
  
  return <div>...</div>;
}

// Callback ref (for dynamic refs)
function CallbackRef() {
  const [height, setHeight] = useState(0);
  
  const measuredRef = useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);
  
  return (
    <div ref={measuredRef}>
      Height: {height}px
    </div>
  );
}

// Forwarding refs
const FancyInput = forwardRef((props, ref) => {
  return <input ref={ref} className="fancy" {...props} />;
});

function Parent() {
  const inputRef = useRef(null);
  
  return (
    <div>
      <FancyInput ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>
        Focus Input
      </button>
    </div>
  );
}

// useImperativeHandle (customize ref value)
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);
  
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current?.focus();
    },
    scrollIntoView: () => {
      inputRef.current?.scrollIntoView();
    },
    clear: () => {
      if (inputRef.current) {
        inputRef.current.value = '';
      }
    }
  }));
  
  return <input ref={inputRef} {...props} />;
});
```

## 7.5 useMemo and useCallback

```jsx
// useMemo - memoize computed values
function ExpensiveComponent({ items, filter }) {
  // Without useMemo: Recalculates on every render
  const filteredItems = items.filter(item => item.includes(filter));
  
  // With useMemo: Only recalculates when items or filter change
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => item.includes(filter));
  }, [items, filter]);
  
  return (
    <ul>
      {filteredItems.map(item => <li key={item}>{item}</li>)}
    </ul>
  );
}

// useCallback - memoize functions
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // Without useCallback: New function every render
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // With useCallback: Same function reference if deps don't change
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  // With dependencies
  const handleSubmit = useCallback(() => {
    console.log('Submitting:', text);
  }, [text]);
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

// When to use:
// useMemo:
// ✅ Expensive calculations
// ✅ Creating objects/arrays passed to memoized children
// ❌ Simple calculations (overhead > benefit)

// useCallback:
// ✅ Passing callbacks to optimized child components (React.memo)
// ✅ Dependencies for other hooks
// ✅ Dependencies in useEffect when function is needed
// ❌ Every function (most don't need it)
```

---

# 8. Context API

## 8.1 Creating and Using Context

```jsx
import { createContext, useContext, useState } from 'react';

// Create context with default value
const ThemeContext = createContext('light');

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer using useContext hook
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
      Toggle Theme ({theme})
    </button>
  );
}

// Usage
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

## 8.2 Context with TypeScript

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
  isLoading: boolean;
}

// Create context with undefined default
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider component
interface AuthProviderProps {
  children: ReactNode;
}

export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const login = async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      });
      const userData = await response.json();
      setUser(userData);
    } finally {
      setIsLoading(false);
    }
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{
      user,
      isAuthenticated: !!user,
      login,
      logout,
      isLoading
    }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook with type safety
export function useAuth() {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  
  return context;
}

// Usage
function Profile() {
  const { user, isAuthenticated, logout } = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  return (
    <div>
      <h1>Welcome, {user!.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

## 8.3 Context Optimization

```jsx
import { createContext, useContext, useState, useMemo, useCallback, memo } from 'react';

// Problem: Context causes unnecessary re-renders
function BadProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  // New object every render - causes all consumers to re-render!
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}

// Solution 1: Memoize the value
function GoodProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
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

// Solution 2: Split contexts by update frequency
const UserContext = createContext(null);
const ThemeContext = createContext('light');

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        {children}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Solution 3: Separate state and dispatch contexts
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function OptimizedProvider({ children }) {
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
  return <button onClick={() => dispatch({ type: 'ADD' })}>Add</button>;
}
```

---

# 9. Performance Optimization

## 9.1 React.memo Deep Dive

```jsx
import { memo, useMemo, useCallback } from 'react';

// Basic memo
const ExpensiveList = memo(function ExpensiveList({ items, onItemClick }) {
  console.log('ExpensiveList rendered');
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

// Parent component optimization
function Parent() {
  const [count, setCount] = useState(0);
  const [items] = useState([{ id: 1, name: 'Item 1' }]);
  
  // Without useCallback, new function every render defeats memo
  const handleItemClick = useCallback((id) => {
    console.log('Clicked:', id);
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      
      {/* ExpensiveList won't re-render when count changes */}
      <ExpensiveList items={items} onItemClick={handleItemClick} />
    </div>
  );
}

// Custom comparison function
const UserCard = memo(
  function UserCard({ user, onClick }) {
    return (
      <div onClick={onClick}>
        <img src={user.avatar} alt={user.name} />
        <h3>{user.name}</h3>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Only re-render if user id or name changes
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name
    );
  }
);
```

## 9.2 Code Splitting with React.lazy

```jsx
import { lazy, Suspense, useState } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Analytics = lazy(() => import('./Analytics'));

// Named exports require adjustment
const Dashboard = lazy(() =>
  import('./Dashboard').then(module => ({ default: module.Dashboard }))
);

// With loading fallback
function App() {
  const [page, setPage] = useState('dashboard');
  
  return (
    <div>
      <nav>
        <button onClick={() => setPage('dashboard')}>Dashboard</button>
        <button onClick={() => setPage('settings')}>Settings</button>
        <button onClick={() => setPage('analytics')}>Analytics</button>
      </nav>
      
      <Suspense fallback={<div>Loading...</div>}>
        {page === 'dashboard' && <Dashboard />}
        {page === 'settings' && <Settings />}
        {page === 'analytics' && <Analytics />}
      </Suspense>
    </div>
  );
}

// Route-based code splitting
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// Preloading components
const Dashboard = lazy(() => import('./Dashboard'));

// Preload on hover
function NavLink({ to, children }) {
  const handleMouseEnter = () => {
    // Preload the component
    import('./Dashboard');
  };
  
  return (
    <Link to={to} onMouseEnter={handleMouseEnter}>
      {children}
    </Link>
  );
}
```

## 9.3 Virtualization for Long Lists

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

// Virtual list for thousands of items
function VirtualList({ items }) {
  const parentRef = useRef(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // Estimated row height
    overscan: 5  // Render 5 items above/below viewport
  });
  
  return (
    <div
      ref={parentRef}
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}

// Simple windowing without library
function SimpleVirtualList({ items, itemHeight = 50, windowHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(
    items.length - 1,
    Math.floor((scrollTop + windowHeight) / itemHeight)
  );
  
  const visibleItems = items.slice(startIndex, endIndex + 1);
  
  return (
    <div
      style={{ height: windowHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems.map((item, index) => (
          <div
            key={item.id}
            style={{
              position: 'absolute',
              top: (startIndex + index) * itemHeight,
              height: itemHeight
            }}
          >
            {item.name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 9.4 Debouncing and Throttling

```jsx
import { useState, useEffect, useCallback, useRef } from 'react';

// Custom useDebounce hook
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

// Usage
function SearchInput() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API call only happens after user stops typing for 500ms
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}

// Custom useThrottle hook
function useThrottle(callback, delay) {
  const lastRun = useRef(Date.now());
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    const now = Date.now();
    
    if (now - lastRun.current >= delay) {
      lastRun.current = now;
      callback(...args);
    } else {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        lastRun.current = Date.now();
        callback(...args);
      }, delay - (now - lastRun.current));
    }
  }, [callback, delay]);
}

// Usage: Scroll handler
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  
  const handleScroll = useThrottle(() => {
    setScrollY(window.scrollY);
  }, 100);
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);
  
  return <div>Scroll position: {scrollY}</div>;
}
```

---

# 10. Patterns & Best Practices

## 10.1 Custom Hooks Patterns

```jsx
// Data fetching hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchData() {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url, { signal: controller.signal });
        if (!response.ok) throw new Error('Failed to fetch');
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchData();
    
    return () => controller.abort();
  }, [url]);
  
  return { data, loading, error };
}

// Local storage hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
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

// Toggle hook
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return { value, toggle, setTrue, setFalse };
}

// Form hook
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
    }
  };
  
  const handleSubmit = async (onSubmit) => {
    setIsSubmitting(true);
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length > 0) {
        setIsSubmitting(false);
        return;
      }
    }
    
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}
```

## 10.2 Container/Presentational Pattern

```jsx
// Container: Handles logic and data
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filter, setFilter] = useState('');
  
  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  const filteredUsers = useMemo(() =>
    users.filter(user => 
      user.name.toLowerCase().includes(filter.toLowerCase())
    ),
    [users, filter]
  );
  
  const handleDelete = async (userId) => {
    await deleteUser(userId);
    setUsers(prev => prev.filter(u => u.id !== userId));
  };
  
  return (
    <UserList
      users={filteredUsers}
      loading={loading}
      error={error}
      filter={filter}
      onFilterChange={setFilter}
      onDelete={handleDelete}
    />
  );
}

// Presentational: Pure display component
function UserList({ 
  users, 
  loading, 
  error, 
  filter, 
  onFilterChange, 
  onDelete 
}) {
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(e) => onFilterChange(e.target.value)}
        placeholder="Filter users..."
      />
      
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name}
            <button onClick={() => onDelete(user.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

# 11. Error Handling

## 11.1 Error Boundaries

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log error to service
    console.error('Error caught:', error);
    console.error('Error info:', errorInfo.componentStack);
    
    this.setState({ error, errorInfo });
    
    // Send to error tracking service
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
          {process.env.NODE_ENV === 'development' && (
            <pre>{this.state.error?.toString()}</pre>
          )}
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary fallback={<CustomErrorPage />}>
      <Header />
      <ErrorBoundary fallback={<MainContentError />}>
        <Main />
      </ErrorBoundary>
      <Footer />
    </ErrorBoundary>
  );
}

// Hook-based alternative (using react-error-boundary)
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logErrorToService(error, info)}
      onReset={() => {
        // Reset app state here
      }}
    >
      <MyComponent />
    </ErrorBoundary>
  );
}

// Triggering error boundary from hooks
function UserProfile({ userId }) {
  const { showBoundary } = useErrorBoundary();
  
  useEffect(() => {
    fetchUser(userId).catch(showBoundary);
  }, [userId, showBoundary]);
}
```

---

# 12. Testing React Applications

## 12.1 React Testing Library

```jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Basic rendering and queries
test('renders greeting', () => {
  render(<Greeting name="Jayesh" />);
  
  // Queries
  expect(screen.getByText('Hello, Jayesh!')).toBeInTheDocument();
  expect(screen.getByRole('heading')).toHaveTextContent('Hello');
  expect(screen.queryByText('Goodbye')).not.toBeInTheDocument();
});

// Testing user interactions
test('increments counter on click', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  const button = screen.getByRole('button', { name: /increment/i });
  
  await user.click(button);
  
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// Testing forms
test('submits form with user data', async () => {
  const user = userEvent.setup();
  const handleSubmit = jest.fn();
  
  render(<LoginForm onSubmit={handleSubmit} />);
  
  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/password/i), 'password123');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});

// Testing async behavior
test('loads and displays users', async () => {
  render(<UserList />);
  
  // Loading state
  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  
  // Wait for data
  await waitFor(() => {
    expect(screen.getByText('User 1')).toBeInTheDocument();
  });
  
  // Or use findBy (combines getBy + waitFor)
  const user1 = await screen.findByText('User 1');
  expect(user1).toBeInTheDocument();
});

// Mocking API calls
import * as api from './api';

jest.mock('./api');

test('fetches users on mount', async () => {
  const mockUsers = [{ id: 1, name: 'User 1' }];
  api.fetchUsers.mockResolvedValue(mockUsers);
  
  render(<UserList />);
  
  await waitFor(() => {
    expect(screen.getByText('User 1')).toBeInTheDocument();
  });
});

// Testing with context
function renderWithProviders(ui, { initialState = {} } = {}) {
  return render(
    <AuthProvider initialState={initialState}>
      <ThemeProvider>
        {ui}
      </ThemeProvider>
    </AuthProvider>
  );
}

test('shows user name when authenticated', () => {
  renderWithProviders(<Profile />, {
    initialState: { user: { name: 'Jayesh' } }
  });
  
  expect(screen.getByText('Welcome, Jayesh')).toBeInTheDocument();
});
```

---

# 13. React Router

## 13.1 React Router v6 Basics

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  NavLink,
  Navigate,
  Outlet,
  useParams,
  useNavigate,
  useLocation,
  useSearchParams
} from 'react-router-dom';

// Basic setup
function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <NavLink 
          to="/contact"
          className={({ isActive }) => isActive ? 'active' : ''}
        >
          Contact
        </NavLink>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Nested routes
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="users" element={<Users />}>
          <Route index element={<UserList />} />
          <Route path=":userId" element={<UserProfile />} />
          <Route path="new" element={<NewUser />} />
        </Route>
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

function Layout() {
  return (
    <div>
      <Header />
      <main>
        <Outlet />  {/* Child routes render here */}
      </main>
      <Footer />
    </div>
  );
}

// Hooks
function UserProfile() {
  const { userId } = useParams();
  const navigate = useNavigate();
  const location = useLocation();
  const [searchParams, setSearchParams] = useSearchParams();
  
  const tab = searchParams.get('tab') || 'profile';
  
  return (
    <div>
      <h1>User: {userId}</h1>
      <p>Current path: {location.pathname}</p>
      
      <button onClick={() => setSearchParams({ tab: 'settings' })}>
        Settings Tab
      </button>
      
      <button onClick={() => navigate('/')}>Go Home</button>
      <button onClick={() => navigate(-1)}>Go Back</button>
    </div>
  );
}

// Protected routes
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

---

# 14. Server-Side Rendering Concepts

## 14.1 SSR vs CSR vs SSG

```jsx
// Client-Side Rendering (CSR)
// - JavaScript renders content in browser
// - Initial HTML is empty shell
// - Good for: Highly interactive apps
// - Bad for: SEO, initial load time

// Server-Side Rendering (SSR)
// - Server renders HTML for each request
// - Good for: SEO, initial load
// - Bad for: Server load, TTFB

// Static Site Generation (SSG)
// - Pages pre-rendered at build time
// - Good for: Content that doesn't change often
// - Bad for: Dynamic, personalized content

// Hydration
// - Process of making server-rendered HTML interactive
// - React attaches event handlers to existing DOM

// React Server Components (RSC)
// - Components that run only on server
// - Zero JavaScript sent to client
// - Can access server resources directly

'use server';  // Marks file as server-only

async function UserProfile({ userId }) {
  // Can access database directly
  const user = await db.users.findById(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <ClientComponent user={user} />
    </div>
  );
}
```

---

# 15. Common Interview Questions

## 15.1 Core React Questions

**Q1: What is the Virtual DOM and how does it work?**

The Virtual DOM is a lightweight JavaScript representation of the Real DOM. When state changes, React creates a new Virtual DOM tree, compares it with the previous one (diffing/reconciliation), and applies only the necessary changes to the Real DOM. This makes updates efficient by minimizing direct DOM manipulation.

---

**Q2: What's the difference between state and props?**

| State | Props |
|-------|-------|
| Owned by component | Passed from parent |
| Mutable (via setState) | Immutable |
| Can change over time | Cannot be changed by child |
| Used for internal data | Used for component configuration |
| Triggers re-render when updated | Triggers re-render when parent changes |

---

**Q3: Why do we need keys in lists?**

Keys help React identify which items have changed, been added, or removed. They should be stable, unique among siblings, and predictable. Using proper keys optimizes re-rendering by allowing React to reuse existing DOM elements when possible.

```jsx
// ❌ Bad: Index as key (problems with reordering)
{items.map((item, index) => <Item key={index} />)}

// ✅ Good: Unique ID
{items.map(item => <Item key={item.id} />)}
```

---

**Q4: What's the difference between useEffect and useLayoutEffect?**

- `useEffect`: Runs asynchronously after paint. Use for data fetching, subscriptions.
- `useLayoutEffect`: Runs synchronously after DOM mutations, before paint. Use for DOM measurements, preventing visual flicker.

---

**Q5: Explain React's Reconciliation algorithm**

React uses a heuristic O(n) algorithm that makes two assumptions:
1. Elements of different types produce different trees
2. Developer hints with `key` prop indicate stable elements across renders

The algorithm compares elements type first, then props. It uses keys to match elements in lists.

---

## 15.2 Output Questions

**Q6: What will this output?**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  };
  
  return <button onClick={handleClick}>{count}</button>;
}
// After clicking once: count = 1 (not 3!)
// Because all three setCount use the same stale count value (0)

// Fix with functional update:
setCount(prev => prev + 1);  // All three would work correctly
```

---

**Q7: What happens when this component renders?**

```jsx
function Oops() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(1);
  });
  
  return <div>{count}</div>;
}
// Infinite loop! Effect runs after every render, 
// triggers state update, which triggers another render...

// Fix: Add dependency array
useEffect(() => { setCount(1); }, []);
```

---

## 15.3 Scenario Questions

**Q8: How would you optimize a component that re-renders too often?**

1. Use `React.memo` for memoization
2. Use `useMemo` for expensive calculations
3. Use `useCallback` for callback functions passed to children
4. Split state to minimize re-renders
5. Use `useRef` for values that don't need re-render
6. Consider using virtualization for long lists

---

**Q9: How would you handle global state?**

Options in order of complexity:
1. Context API (built-in, simple)
2. Zustand (minimal, hooks-based)
3. Redux Toolkit (feature-rich, complex apps)
4. Jotai/Recoil (atomic state)

Choose based on app size, team familiarity, and features needed.

---

# 16. Coding Challenges

## 16.1 Implement useDebounce Hook

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

## 16.2 Implement usePrevious Hook

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}
```

## 16.3 Implement useInterval Hook

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

## 16.4 Implement useOnClickOutside Hook

```jsx
function useOnClickOutside(ref, handler) {
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

## 16.5 Implement Toggle Component

```jsx
function Toggle({ onToggle, initialOn = false }) {
  const [on, setOn] = useState(initialOn);
  
  const toggle = useCallback(() => {
    setOn(prev => {
      const newValue = !prev;
      onToggle?.(newValue);
      return newValue;
    });
  }, [onToggle]);
  
  return (
    <button
      onClick={toggle}
      className={`toggle ${on ? 'on' : 'off'}`}
      aria-pressed={on}
    >
      {on ? 'ON' : 'OFF'}
    </button>
  );
}
```

---

*End of Part 3: React Deep Dive - Complete Interview Guide*

> **Total Coverage:** 6000+ lines of comprehensive React content including:
> - React fundamentals and Virtual DOM
> - Components (function, class, HOC, render props)
> - JSX in depth
> - State management (useState, useReducer)
> - Props and data flow
> - Component lifecycle
> - All hooks with detailed examples
> - Context API and optimization
> - Performance optimization techniques
> - Patterns and best practices
> - Error handling with boundaries
> - Testing with React Testing Library
> - React Router v6
> - Server-side rendering concepts
> - 30+ interview questions
> - 20+ coding challenges

---

# 17. Advanced React Concepts

## 17.1 React Concurrent Features

```jsx
import { 
  useTransition, 
  useDeferredValue, 
  Suspense,
  startTransition 
} from 'react';

// useTransition - Mark updates as non-urgent
function SearchResults() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // High priority: Update input immediately
    setQuery(value);
    
    // Low priority: Can be interrupted
    startTransition(() => {
      setResults(searchDatabase(value));
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ResultList results={results} />
    </div>
  );
}

// useDeferredValue - Defer expensive rendering
function ProductList({ query }) {
  const [products, setProducts] = useState([]);
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  
  useEffect(() => {
    fetchProducts(deferredQuery).then(setProducts);
  }, [deferredQuery]);
  
  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Suspense for data fetching
function ProfilePage({ userId }) {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <ProfileDetails userId={userId} />
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={userId} />
      </Suspense>
    </Suspense>
  );
}

// Suspense with error boundary
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Suspense fallback={<Loading />}>
        <AsyncComponent />
      </Suspense>
    </ErrorBoundary>
  );
}

// startTransition outside components
function handleClick() {
  startTransition(() => {
    // Non-urgent update
    setPage('/newPage');
  });
}
```

## 17.2 React Server Components (RSC)

```jsx
// Server Component (default in Next.js App Router)
// Can access database, file system, etc.
// No hooks, no event handlers, no browser APIs

// app/users/page.tsx
async function UsersPage() {
  // Direct database access on server
  const users = await db.user.findMany();
  
  return (
    <div>
      <h1>Users</h1>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
      {/* Client component for interactivity */}
      <AddUserButton />
    </div>
  );
}

// Client Component (opt-in with 'use client')
'use client';

import { useState } from 'react';

export function AddUserButton() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <>
      <button onClick={() => setIsOpen(true)}>Add User</button>
      {isOpen && <AddUserModal onClose={() => setIsOpen(false)} />}
    </>
  );
}

// Server Actions
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;
  
  await db.user.create({ data: { name, email } });
  revalidatePath('/users');
}

// Using server action in client component
'use client';

import { createUser } from './actions';

export function AddUserForm() {
  return (
    <form action={createUser}>
      <input name="name" placeholder="Name" />
      <input name="email" placeholder="Email" />
      <button type="submit">Create</button>
    </form>
  );
}
```

## 17.3 Portal Usage

```jsx
import { createPortal } from 'react-dom';

// Basic portal
function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.getElementById('modal-root')
  );
}

// Tooltip with portal
function Tooltip({ children, text, targetRef }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    if (targetRef.current && isVisible) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY,
        left: rect.left + rect.width / 2 + window.scrollX
      });
    }
  }, [isVisible, targetRef]);
  
  return (
    <>
      <span
        ref={targetRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
      >
        {children}
      </span>
      {isVisible && createPortal(
        <div
          className="tooltip"
          style={{
            position: 'absolute',
            top: position.top,
            left: position.left,
            transform: 'translateX(-50%)'
          }}
        >
          {text}
        </div>,
        document.body
      )}
    </>
  );
}

// Dropdown that escapes overflow:hidden
function Dropdown({ trigger, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const triggerRef = useRef(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  
  useEffect(() => {
    if (triggerRef.current && isOpen) {
      const rect = triggerRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY,
        left: rect.left + window.scrollX
      });
    }
  }, [isOpen]);
  
  return (
    <>
      <button ref={triggerRef} onClick={() => setIsOpen(!isOpen)}>
        {trigger}
      </button>
      {isOpen && createPortal(
        <div
          className="dropdown-menu"
          style={{
            position: 'absolute',
            top: position.top,
            left: position.left
          }}
        >
          {children}
        </div>,
        document.body
      )}
    </>
  );
}
```

## 17.4 Forwarding Refs

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

// Basic ref forwarding
const FancyInput = forwardRef(function FancyInput(props, ref) {
  return (
    <input
      ref={ref}
      className="fancy-input"
      {...props}
    />
  );
});

// Usage
function Parent() {
  const inputRef = useRef(null);
  
  const handleFocus = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <FancyInput ref={inputRef} placeholder="Enter text" />
      <button onClick={handleFocus}>Focus Input</button>
    </>
  );
}

// Exposing custom methods with useImperativeHandle
const CustomInput = forwardRef(function CustomInput(
  { label, ...props },
  ref
) {
  const inputRef = useRef(null);
  
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
      return inputRef.current?.value || '';
    },
    setValue(value) {
      if (inputRef.current) {
        inputRef.current.value = value;
      }
    }
  }), []);
  
  return (
    <div className="input-wrapper">
      {label && <label>{label}</label>}
      <input ref={inputRef} {...props} />
    </div>
  );
});

// Usage
function Form() {
  const inputRef = useRef(null);
  
  const handleSubmit = () => {
    const value = inputRef.current?.getValue();
    console.log('Value:', value);
    inputRef.current?.clear();
  };
  
  return (
    <form>
      <CustomInput ref={inputRef} label="Name" />
      <button type="button" onClick={handleSubmit}>Submit</button>
    </form>
  );
}

// HOC with ref forwarding
function withLogging(WrappedComponent) {
  const WithLogging = forwardRef((props, ref) => {
    useEffect(() => {
      console.log('Component mounted');
      return () => console.log('Component unmounted');
    }, []);
    
    return <WrappedComponent ref={ref} {...props} />;
  });
  
  WithLogging.displayName = `WithLogging(${WrappedComponent.displayName || WrappedComponent.name})`;
  
  return WithLogging;
}
```

---

# 18. Forms in React

## 18.1 Form Libraries (React Hook Form)

```jsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Schema validation with Zod
const signupSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
  age: z.number().min(18, 'Must be at least 18'),
  terms: z.boolean().refine(val => val, 'Must accept terms')
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword']
});

type SignupForm = z.infer<typeof signupSchema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting },
    watch,
    reset,
    setValue
  } = useForm<SignupForm>({
    resolver: zodResolver(signupSchema),
    defaultValues: {
      email: '',
      password: '',
      confirmPassword: '',
      age: 18,
      terms: false
    }
  });
  
  const onSubmit = async (data: SignupForm) => {
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log(data);
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <input
          type="password"
          {...register('password')}
          placeholder="Password"
        />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      <div>
        <input
          type="password"
          {...register('confirmPassword')}
          placeholder="Confirm Password"
        />
        {errors.confirmPassword && (
          <span>{errors.confirmPassword.message}</span>
        )}
      </div>
      
      <div>
        <input
          type="number"
          {...register('age', { valueAsNumber: true })}
        />
        {errors.age && <span>{errors.age.message}</span>}
      </div>
      
      <div>
        <label>
          <input type="checkbox" {...register('terms')} />
          Accept Terms
        </label>
        {errors.terms && <span>{errors.terms.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Sign Up'}
      </button>
    </form>
  );
}

// Controller for custom components
function FormWithSelect() {
  const { control, handleSubmit } = useForm();
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <Controller
        name="country"
        control={control}
        render={({ field }) => (
          <Select
            {...field}
            options={[
              { value: 'us', label: 'United States' },
              { value: 'uk', label: 'United Kingdom' },
              { value: 'in', label: 'India' }
            ]}
          />
        )}
      />
    </form>
  );
}
```

## 18.2 Form Validation Patterns

```jsx
// Manual validation
function ManualValidation() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      // Submit form
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <input
        type="password"
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
      />
      {errors.password && <span className="error">{errors.password}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}

// Async validation (username availability)
function UsernameField() {
  const [username, setUsername] = useState('');
  const [status, setStatus] = useState('idle');
  
  const debouncedUsername = useDebounce(username, 500);
  
  useEffect(() => {
    if (!debouncedUsername) {
      setStatus('idle');
      return;
    }
    
    let cancelled = false;
    
    const checkAvailability = async () => {
      setStatus('checking');
      
      try {
        const response = await fetch(
          `/api/check-username?username=${debouncedUsername}`
        );
        const { available } = await response.json();
        
        if (!cancelled) {
          setStatus(available ? 'available' : 'taken');
        }
      } catch {
        if (!cancelled) {
          setStatus('error');
        }
      }
    };
    
    checkAvailability();
    
    return () => {
      cancelled = true;
    };
  }, [debouncedUsername]);
  
  return (
    <div>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="Username"
      />
      {status === 'checking' && <span>Checking...</span>}
      {status === 'available' && <span className="success">✓ Available</span>}
      {status === 'taken' && <span className="error">✗ Username taken</span>}
    </div>
  );
}
```

---

# 19. Accessibility in React

## 19.1 Accessible Components

```jsx
// Accessible button
function AccessibleButton({ 
  children, 
  disabled, 
  loading, 
  onClick,
  ariaLabel 
}) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-disabled={disabled || loading}
      aria-busy={loading}
      aria-label={ariaLabel}
    >
      {loading ? (
        <>
          <span className="visually-hidden">Loading...</span>
          <Spinner aria-hidden="true" />
        </>
      ) : (
        children
      )}
    </button>
  );
}

// Accessible modal
function AccessibleModal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousActiveElement = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      previousActiveElement.current = document.activeElement;
      modalRef.current?.focus();
    } else {
      previousActiveElement.current?.focus();
    }
  }, [isOpen]);
  
  // Trap focus inside modal
  const handleKeyDown = (e) => {
    if (e.key === 'Escape') {
      onClose();
    }
    
    if (e.key === 'Tab') {
      const focusableElements = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (focusableElements?.length) {
        const firstElement = focusableElements[0];
        const lastElement = focusableElements[focusableElements.length - 1];
        
        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    }
  };
  
  if (!isOpen) return null;
  
  return createPortal(
    <div
      className="modal-overlay"
      onClick={onClose}
      role="presentation"
    >
      <div
        ref={modalRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
        onKeyDown={handleKeyDown}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button
          onClick={onClose}
          aria-label="Close modal"
        >
          ×
        </button>
      </div>
    </div>,
    document.body
  );
}

// Accessible form field
function FormField({ 
  id, 
  label, 
  error, 
  required, 
  helpText,
  ...inputProps 
}) {
  const errorId = error ? `${id}-error` : undefined;
  const helpId = helpText ? `${id}-help` : undefined;
  
  return (
    <div className="form-field">
      <label htmlFor={id}>
        {label}
        {required && <span aria-hidden="true"> *</span>}
        {required && (
          <span className="visually-hidden">(required)</span>
        )}
      </label>
      
      <input
        id={id}
        aria-invalid={!!error}
        aria-describedby={[errorId, helpId].filter(Boolean).join(' ') || undefined}
        aria-required={required}
        {...inputProps}
      />
      
      {helpText && (
        <p id={helpId} className="help-text">
          {helpText}
        </p>
      )}
      
      {error && (
        <p id={errorId} className="error-text" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

// Skip link for keyboard navigation
function SkipLink() {
  return (
    <a href="#main-content" className="skip-link">
      Skip to main content
    </a>
  );
}

// Live region for announcements
function LiveAnnouncer({ message }) {
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="visually-hidden"
    >
      {message}
    </div>
  );
}
```

## 19.2 Focus Management

```jsx
// Focus trap hook
function useFocusTrap(ref, isActive) {
  useEffect(() => {
    if (!isActive) return;
    
    const element = ref.current;
    if (!element) return;
    
    const focusableElements = element.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];
    
    const handleKeyDown = (e) => {
      if (e.key !== 'Tab') return;
      
      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement?.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement?.focus();
      }
    };
    
    element.addEventListener('keydown', handleKeyDown);
    firstElement?.focus();
    
    return () => {
      element.removeEventListener('keydown', handleKeyDown);
    };
  }, [ref, isActive]);
}

// Roving tabindex for keyboard navigation
function ToolbarWithRovingTabIndex({ items }) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  const itemRefs = useRef([]);
  
  const handleKeyDown = (e, index) => {
    let newIndex = index;
    
    switch (e.key) {
      case 'ArrowRight':
        newIndex = (index + 1) % items.length;
        break;
      case 'ArrowLeft':
        newIndex = (index - 1 + items.length) % items.length;
        break;
      case 'Home':
        newIndex = 0;
        break;
      case 'End':
        newIndex = items.length - 1;
        break;
      default:
        return;
    }
    
    e.preventDefault();
    setFocusedIndex(newIndex);
    itemRefs.current[newIndex]?.focus();
  };
  
  return (
    <div role="toolbar" aria-label="Formatting options">
      {items.map((item, index) => (
        <button
          key={item.id}
          ref={(el) => (itemRefs.current[index] = el)}
          tabIndex={index === focusedIndex ? 0 : -1}
          onKeyDown={(e) => handleKeyDown(e, index)}
          onClick={() => setFocusedIndex(index)}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
}
```

---

# 20. State Machines with React

```jsx
import { useMachine } from '@xstate/react';
import { createMachine, assign } from 'xstate';

// Define a state machine
const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  context: {
    data: null,
    error: null
  },
  states: {
    idle: {
      on: { FETCH: 'loading' }
    },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: {
          target: 'success',
          actions: assign({ data: (_, event) => event.data })
        },
        onError: {
          target: 'failure',
          actions: assign({ error: (_, event) => event.data })
        }
      }
    },
    success: {
      on: { RESET: 'idle' }
    },
    failure: {
      on: { 
        RETRY: 'loading',
        RESET: 'idle'
      }
    }
  }
});

// Use in component
function DataFetcher({ url }) {
  const [state, send] = useMachine(fetchMachine, {
    services: {
      fetchData: () => fetch(url).then(res => res.json())
    }
  });
  
  return (
    <div>
      {state.matches('idle') && (
        <button onClick={() => send('FETCH')}>Fetch Data</button>
      )}
      
      {state.matches('loading') && <Spinner />}
      
      {state.matches('success') && (
        <div>
          <pre>{JSON.stringify(state.context.data, null, 2)}</pre>
          <button onClick={() => send('RESET')}>Reset</button>
        </div>
      )}
      
      {state.matches('failure') && (
        <div>
          <p>Error: {state.context.error.message}</p>
          <button onClick={() => send('RETRY')}>Retry</button>
          <button onClick={() => send('RESET')}>Reset</button>
        </div>
      )}
    </div>
  );
}

// Simple state machine without library
function useStateMachine(config) {
  const [state, setState] = useState(config.initial);
  
  const send = useCallback((event) => {
    setState(currentState => {
      const nextState = config.states[currentState]?.on?.[event];
      return nextState || currentState;
    });
  }, [config]);
  
  return [state, send];
}

// Usage
function Toggle() {
  const [state, send] = useStateMachine({
    initial: 'off',
    states: {
      off: { on: { TOGGLE: 'on' } },
      on: { on: { TOGGLE: 'off' } }
    }
  });
  
  return (
    <button onClick={() => send('TOGGLE')}>
      {state === 'on' ? 'ON' : 'OFF'}
    </button>
  );
}
```

---

# 21. Animation in React

## 21.1 CSS Transitions

```jsx
// CSS-based transitions
function FadeInOut({ show, children }) {
  return (
    <div
      style={{
        opacity: show ? 1 : 0,
        transition: 'opacity 300ms ease-in-out'
      }}
    >
      {children}
    </div>
  );
}

// Class-based transitions
function SlideIn({ show, children }) {
  return (
    <div className={`slide-container ${show ? 'visible' : 'hidden'}`}>
      {children}
    </div>
  );
}

// CSS
/*
.slide-container {
  transform: translateX(-100%);
  transition: transform 300ms ease-out;
}

.slide-container.visible {
  transform: translateX(0);
}
*/
```

## 21.2 Framer Motion

```jsx
import { motion, AnimatePresence } from 'framer-motion';

// Basic animation
function AnimatedBox() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
    >
      Content
    </motion.div>
  );
}

// AnimatePresence for exit animations
function Modal({ isOpen, onClose, children }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          className="modal-overlay"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          onClick={onClose}
        >
          <motion.div
            className="modal"
            initial={{ scale: 0.8, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            exit={{ scale: 0.8, opacity: 0 }}
            transition={{ type: 'spring', damping: 25 }}
            onClick={(e) => e.stopPropagation()}
          >
            {children}
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}

// Staggered list animation
function AnimatedList({ items }) {
  return (
    <motion.ul
      initial="hidden"
      animate="visible"
      variants={{
        visible: {
          transition: { staggerChildren: 0.1 }
        }
      }}
    >
      {items.map((item) => (
        <motion.li
          key={item.id}
          variants={{
            hidden: { opacity: 0, x: -20 },
            visible: { opacity: 1, x: 0 }
          }}
        >
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}

// Drag and drop
function DraggableCard({ children }) {
  return (
    <motion.div
      drag
      dragConstraints={{ top: 0, left: 0, right: 0, bottom: 0 }}
      dragElastic={0.1}
      whileDrag={{ scale: 1.05 }}
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
    >
      {children}
    </motion.div>
  );
}

// Layout animations
function LayoutAnimationExample() {
  const [isExpanded, setIsExpanded] = useState(false);
  
  return (
    <motion.div
      layout
      className={isExpanded ? 'card expanded' : 'card'}
      onClick={() => setIsExpanded(!isExpanded)}
    >
      <motion.h2 layout="position">Title</motion.h2>
      <AnimatePresence>
        {isExpanded && (
          <motion.p
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
          >
            Expanded content...
          </motion.p>
        )}
      </AnimatePresence>
    </motion.div>
  );
}
```

---

# 22. More Coding Challenges

## 22.1 Implement useFetch Hook

```jsx
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const optionsRef = useRef(options);
  
  useEffect(() => {
    optionsRef.current = options;
  });
  
  const execute = useCallback(async (overrideOptions = {}) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url, {
        ...optionsRef.current,
        ...overrideOptions
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
      return result;
    } catch (e) {
      setError(e);
      throw e;
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    if (options.immediate !== false) {
      execute();
    }
  }, [execute, options.immediate]);
  
  return { data, error, loading, execute, refetch: execute };
}
```

## 22.2 Implement useAsync Hook

```jsx
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
    } catch (e) {
      setError(e);
      setStatus('error');
      throw e;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { execute, status, value, error };
}
```

## 22.3 Implement useMediaQuery Hook

```jsx
function useMediaQuery(query) {
  const [matches, setMatches] = useState(() => {
    if (typeof window === 'undefined') return false;
    return window.matchMedia(query).matches;
  });
  
  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    
    const handler = (event) => setMatches(event.matches);
    
    // Modern browsers
    if (mediaQuery.addEventListener) {
      mediaQuery.addEventListener('change', handler);
    } else {
      // Legacy browsers
      mediaQuery.addListener(handler);
    }
    
    // Set initial value
    setMatches(mediaQuery.matches);
    
    return () => {
      if (mediaQuery.removeEventListener) {
        mediaQuery.removeEventListener('change', handler);
      } else {
        mediaQuery.removeListener(handler);
      }
    };
  }, [query]);
  
  return matches;
}

// Usage
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');
  
  return (
    <div>
      {isMobile && <MobileLayout />}
      {isTablet && <TabletLayout />}
      {isDesktop && <DesktopLayout />}
    </div>
  );
}
```

## 22.4 Implement useIntersectionObserver Hook

```jsx
function useIntersectionObserver(options = {}) {
  const [entry, setEntry] = useState(null);
  const [node, setNode] = useState(null);
  
  const observer = useRef(null);
  
  useEffect(() => {
    if (observer.current) {
      observer.current.disconnect();
    }
    
    observer.current = new IntersectionObserver(
      ([entry]) => setEntry(entry),
      {
        threshold: options.threshold || 0,
        root: options.root || null,
        rootMargin: options.rootMargin || '0px'
      }
    );
    
    if (node) {
      observer.current.observe(node);
    }
    
    return () => {
      observer.current?.disconnect();
    };
  }, [node, options.threshold, options.root, options.rootMargin]);
  
  return [setNode, entry];
}

// Usage: Lazy loading images
function LazyImage({ src, alt, ...props }) {
  const [ref, entry] = useIntersectionObserver({ threshold: 0.1 });
  const [isLoaded, setIsLoaded] = useState(false);
  
  const isVisible = entry?.isIntersecting;
  
  return (
    <div ref={ref}>
      {(isVisible || isLoaded) ? (
        <img
          src={src}
          alt={alt}
          onLoad={() => setIsLoaded(true)}
          {...props}
        />
      ) : (
        <div className="placeholder" style={{ height: 200 }} />
      )}
    </div>
  );
}
```

## 22.5 Implement Infinite Scroll

```jsx
function useInfiniteScroll(fetchMore, hasMore) {
  const [isFetching, setIsFetching] = useState(false);
  const [setNode, entry] = useIntersectionObserver({ threshold: 0.5 });
  
  useEffect(() => {
    if (entry?.isIntersecting && hasMore && !isFetching) {
      setIsFetching(true);
      fetchMore().finally(() => setIsFetching(false));
    }
  }, [entry?.isIntersecting, hasMore, isFetching, fetchMore]);
  
  return { setNode, isFetching };
}

// Usage
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  const fetchMore = useCallback(async () => {
    const response = await fetch(`/api/items?page=${page}`);
    const newItems = await response.json();
    
    setItems(prev => [...prev, ...newItems]);
    setPage(prev => prev + 1);
    setHasMore(newItems.length > 0);
  }, [page]);
  
  const { setNode, isFetching } = useInfiniteScroll(fetchMore, hasMore);
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      
      <div ref={setNode}>
        {isFetching && <Spinner />}
        {!hasMore && <p>No more items</p>}
      </div>
    </div>
  );
}
```

---

# 23. Quick Reference Cheat Sheet

## React Hooks Summary

```jsx
// State hooks
useState(initialValue)           // Local state
useReducer(reducer, init)        // Complex state logic

// Effects
useEffect(() => {}, [deps])      // Side effects after render
useLayoutEffect(() => {}, [])    // Before paint

// Refs
useRef(initialValue)             // Mutable ref object
useImperativeHandle(ref, () => {}) // Customize ref

// Context
useContext(Context)              // Consume context

// Performance
useMemo(() => value, [deps])     // Memoize value
useCallback(() => {}, [deps])    // Memoize function
React.memo(Component)            // Memoize component

// Concurrent
useTransition()                  // Non-blocking updates
useDeferredValue(value)          // Defer expensive renders

// Custom hooks
function useCustomHook() {}      // Prefix with 'use'
```

## React Patterns Summary

```jsx
// Controlled component
<input value={value} onChange={onChange} />

// Uncontrolled component
<input ref={inputRef} defaultValue="default" />

// Conditional rendering
{condition && <Component />}
{condition ? <A /> : <B />}

// List rendering
{items.map(item => <Item key={item.id} {...item} />)}

// Props spreading
<Component {...props} />

// Children
<Parent>{children}</Parent>

// Render props
<DataFetcher render={(data) => <Display data={data} />} />

// HOC
const Enhanced = withFeature(Component);

// Compound components
<Tabs>
  <Tabs.Tab>One</Tabs.Tab>
  <Tabs.Panel>Content</Tabs.Panel>
</Tabs>
```

---

*End of Part 3: React Deep Dive - Complete Interview Guide*

> **Total Coverage:** 6000+ lines of comprehensive React content including:
> - React fundamentals and Virtual DOM
> - Components (function, class, HOC, render props)
> - JSX in depth with conditional rendering
> - State management (useState, useReducer)
> - Props, data flow, and prop drilling solutions
> - Component lifecycle (class and hooks)
> - All hooks with detailed examples
> - Context API with optimization techniques
> - Performance optimization (memo, lazy, virtualization)
> - Patterns and best practices
> - Error handling with boundaries
> - Testing with React Testing Library
> - React Router v6
> - Server-side rendering concepts
> - Concurrent features (useTransition, useDeferredValue)
> - React Server Components
> - Portals and ref forwarding
> - Forms with React Hook Form
> - Accessibility (ARIA, focus management)
> - State machines
> - Animation with Framer Motion
> - 50+ interview questions
> - 30+ coding challenges

---

# 24. Additional Interview Questions

## 24.1 React 18+ Features

**Q1: What is Automatic Batching in React 18?**

React 18 automatically batches multiple state updates into a single re-render, even inside promises, setTimeout, and native event handlers. Previously, batching only worked inside React event handlers.

```jsx
// React 17: Multiple re-renders
setTimeout(() => {
  setCount(c => c + 1); // Re-render
  setFlag(f => !f);     // Another re-render
}, 1000);

// React 18: Single re-render (automatic batching)
setTimeout(() => {
  setCount(c => c + 1); // Batched
  setFlag(f => !f);     // Batched - single re-render
}, 1000);

// Opt out with flushSync
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(c => c + 1);
}); // Re-renders here

flushSync(() => {
  setFlag(f => !f);
}); // Re-renders here
```

---

**Q2: What is Concurrent Mode?**

Concurrent Mode (now Concurrent Features) allows React to interrupt rendering to handle more urgent updates. It enables features like useTransition and useDeferredValue to prioritize user interactions.

Key concepts:
- **Interruptible rendering**: React can pause rendering to handle urgent updates
- **Priority-based updates**: Different updates can have different priorities
- **Suspense**: Better support for data fetching and lazy loading

---

**Q3: What are React Server Components?**

React Server Components (RSC) are components that run only on the server:
- No JavaScript sent to client for server components
- Can directly access databases, file systems, etc.
- Cannot use hooks, event handlers, or browser APIs
- Automatically code-split and never re-render on client

```jsx
// Server Component (default in Next.js App Router)
async function UserList() {
  const users = await db.user.findMany(); // Direct DB access
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

// Client Component (opt-in)
'use client';
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

## 24.2 Architecture Questions

**Q4: How would you structure a large React application?**

```
src/
├── components/          # Shared/reusable components
│   ├── ui/             # Basic UI components
│   ├── forms/          # Form components
│   └── layouts/        # Layout components
├── features/           # Feature-based modules
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── index.ts
│   └── users/
├── hooks/              # Shared custom hooks
├── lib/                # Utilities, helpers
├── services/           # API services
├── store/              # Global state
├── types/              # TypeScript types
├── pages/              # Route pages (or app/ for Next.js)
└── App.tsx
```

Key principles:
- **Feature-based structure** for better scalability
- **Colocation** of related files
- **Single responsibility** components
- **Lazy loading** for code splitting

---

**Q5: How do you handle API calls in React?**

Options in order of preference:
1. **TanStack Query** (React Query) - Caching, background updates, optimistic updates
2. **SWR** - Lightweight, stale-while-revalidate
3. **Custom hooks** - For simple cases
4. **Redux Toolkit Query** - When using Redux

```jsx
// TanStack Query example
const { data, isLoading, error, refetch } = useQuery({
  queryKey: ['users', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000,
});

// Mutation with optimistic update
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newData) => {
    await queryClient.cancelQueries(['user', id]);
    const previousData = queryClient.getQueryData(['user', id]);
    queryClient.setQueryData(['user', id], newData);
    return { previousData };
  },
  onError: (err, newData, context) => {
    queryClient.setQueryData(['user', id], context.previousData);
  },
  onSettled: () => {
    queryClient.invalidateQueries(['user', id]);
  },
});
```

---

**Q6: How do you handle authentication in React?**

```jsx
// Auth context pattern
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    // Check session/token on mount
    checkAuth().then(setUser).finally(() => setIsLoading(false));
  }, []);
  
  const login = async (credentials) => {
    const user = await loginAPI(credentials);
    setUser(user);
    localStorage.setItem('token', user.token);
  };
  
  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };
  
  if (isLoading) return <LoadingScreen />;
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Protected route
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}
```

---

## 24.3 Debugging Questions

**Q7: How do you debug React performance problems?**

1. **React DevTools Profiler**: Record renders, identify slow components
2. **React.memo audit**: Check if memoization is working
3. **useWhyDidYouRender**: Library to log unnecessary re-renders
4. **Chrome DevTools Performance tab**: Flame charts, long tasks
5. **console.count/console.time**: Manual render counting

```jsx
// Debug hook for why component re-rendered
function useWhyDidUpdate(name, props) {
  const previousProps = useRef(props);
  
  useEffect(() => {
    const changes = {};
    for (const key of Object.keys(props)) {
      if (previousProps.current[key] !== props[key]) {
        changes[key] = {
          from: previousProps.current[key],
          to: props[key]
        };
      }
    }
    
    if (Object.keys(changes).length > 0) {
      console.log(`[${name}] Changed:`, changes);
    }
    
    previousProps.current = props;
  });
}
```

---

**Q8: What causes infinite loops in useEffect?**

Common causes:
1. **Missing dependency array**: Effect runs after every render
2. **Object/array in dependencies**: New reference each render
3. **State update in effect**: Without proper guards
4. **Function in dependencies**: Without useCallback

```jsx
// ❌ Infinite loop: No dependency array
useEffect(() => {
  setCount(count + 1);
});

// ❌ Infinite loop: Object dependency
const config = { userId };
useEffect(() => {
  fetchData(config);
}, [config]); // New object every render!

// ✅ Fix: Use primitive or useMemo
useEffect(() => {
  fetchData({ userId });
}, [userId]);
```

---

## 24.4 More Output Questions

**Q9: What will this render?**

```jsx
function App() {
  const [items, setItems] = useState(['A', 'B', 'C']);
  
  return (
    <ul>
      {items.map((item, i) => (
        <li key={i}>
          {item}
          <input />
        </li>
      ))}
      <button onClick={() => setItems(['X', ...items])}>
        Add X at start
      </button>
    </ul>
  );
}
```

**Problem**: Using index as key causes input values to shift incorrectly when items are added at the beginning. React reuses DOM elements based on keys, so the input in position 0 stays at position 0 even though the item changed.

**Fix**: Use stable, unique IDs as keys.

---

**Q10: What's wrong with this code?**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(setResults);
  }, [query]);
  
  return <ResultList results={results} />;
}
```

**Issues**:
1. **Race condition**: Fast typing can cause out-of-order responses
2. **Memory leak**: Setting state after unmount
3. **No loading/error states**

**Fix**:
```jsx
useEffect(() => {
  const controller = new AbortController();
  let cancelled = false;
  
  setLoading(true);
  
  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then(res => res.json())
    .then(data => {
      if (!cancelled) {
        setResults(data);
        setLoading(false);
      }
    })
    .catch(err => {
      if (err.name !== 'AbortError' && !cancelled) {
        setError(err);
        setLoading(false);
      }
    });
  
  return () => {
    cancelled = true;
    controller.abort();
  };
}, [query]);
```

---

# 25. Real-World React Patterns

## 25.1 Data Table Component

```jsx
function DataTable({ columns, data, sortable = true, selectable = false }) {
  const [sortConfig, setSortConfig] = useState({ key: null, direction: 'asc' });
  const [selectedRows, setSelectedRows] = useState(new Set());
  
  const sortedData = useMemo(() => {
    if (!sortConfig.key) return data;
    
    return [...data].sort((a, b) => {
      const aVal = a[sortConfig.key];
      const bVal = b[sortConfig.key];
      
      if (aVal < bVal) return sortConfig.direction === 'asc' ? -1 : 1;
      if (aVal > bVal) return sortConfig.direction === 'asc' ? 1 : -1;
      return 0;
    });
  }, [data, sortConfig]);
  
  const handleSort = (key) => {
    setSortConfig(prev => ({
      key,
      direction: prev.key === key && prev.direction === 'asc' ? 'desc' : 'asc'
    }));
  };
  
  const handleSelectAll = (e) => {
    if (e.target.checked) {
      setSelectedRows(new Set(data.map(row => row.id)));
    } else {
      setSelectedRows(new Set());
    }
  };
  
  const handleSelectRow = (id) => {
    setSelectedRows(prev => {
      const newSet = new Set(prev);
      if (newSet.has(id)) {
        newSet.delete(id);
      } else {
        newSet.add(id);
      }
      return newSet;
    });
  };
  
  const isAllSelected = selectedRows.size === data.length && data.length > 0;
  
  return (
    <table>
      <thead>
        <tr>
          {selectable && (
            <th>
              <input
                type="checkbox"
                checked={isAllSelected}
                onChange={handleSelectAll}
              />
            </th>
          )}
          {columns.map(col => (
            <th
              key={col.key}
              onClick={() => sortable && handleSort(col.key)}
              style={{ cursor: sortable ? 'pointer' : 'default' }}
            >
              {col.label}
              {sortConfig.key === col.key && (
                <span>{sortConfig.direction === 'asc' ? ' ↑' : ' ↓'}</span>
              )}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedData.map(row => (
          <tr key={row.id}>
            {selectable && (
              <td>
                <input
                  type="checkbox"
                  checked={selectedRows.has(row.id)}
                  onChange={() => handleSelectRow(row.id)}
                />
              </td>
            )}
            {columns.map(col => (
              <td key={col.key}>
                {col.render ? col.render(row[col.key], row) : row[col.key]}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Usage
<DataTable
  columns={[
    { key: 'name', label: 'Name' },
    { key: 'email', label: 'Email' },
    { key: 'status', label: 'Status', render: (value) => (
      <span className={`badge badge-${value}`}>{value}</span>
    )}
  ]}
  data={users}
  sortable
  selectable
/>
```

## 25.2 Toast Notification System

```jsx
// Toast Context
const ToastContext = createContext(null);

function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);
  
  const addToast = useCallback((message, type = 'info', duration = 3000) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    
    if (duration > 0) {
      setTimeout(() => {
        removeToast(id);
      }, duration);
    }
    
    return id;
  }, []);
  
  const removeToast = useCallback((id) => {
    setToasts(prev => prev.filter(t => t.id !== id));
  }, []);
  
  const success = useCallback((msg) => addToast(msg, 'success'), [addToast]);
  const error = useCallback((msg) => addToast(msg, 'error'), [addToast]);
  const warning = useCallback((msg) => addToast(msg, 'warning'), [addToast]);
  const info = useCallback((msg) => addToast(msg, 'info'), [addToast]);
  
  return (
    <ToastContext.Provider value={{ addToast, removeToast, success, error, warning, info }}>
      {children}
      <ToastContainer toasts={toasts} removeToast={removeToast} />
    </ToastContext.Provider>
  );
}

function ToastContainer({ toasts, removeToast }) {
  return createPortal(
    <div className="toast-container">
      <AnimatePresence>
        {toasts.map(toast => (
          <motion.div
            key={toast.id}
            initial={{ opacity: 0, y: -20, scale: 0.95 }}
            animate={{ opacity: 1, y: 0, scale: 1 }}
            exit={{ opacity: 0, scale: 0.95 }}
            className={`toast toast-${toast.type}`}
          >
            <span>{toast.message}</span>
            <button onClick={() => removeToast(toast.id)}>×</button>
          </motion.div>
        ))}
      </AnimatePresence>
    </div>,
    document.body
  );
}

function useToast() {
  const context = useContext(ToastContext);
  if (!context) throw new Error('useToast must be used within ToastProvider');
  return context;
}

// Usage
function SaveButton() {
  const toast = useToast();
  
  const handleSave = async () => {
    try {
      await saveData();
      toast.success('Saved successfully!');
    } catch (error) {
      toast.error('Failed to save');
    }
  };
  
  return <button onClick={handleSave}>Save</button>;
}
```

## 25.3 Optimistic UI Updates

```jsx
function TodoList() {
  const queryClient = useQueryClient();
  const { data: todos } = useQuery(['todos'], fetchTodos);
  
  const addMutation = useMutation({
    mutationFn: addTodo,
    onMutate: async (newTodo) => {
      // Cancel any outgoing refetches
      await queryClient.cancelQueries(['todos']);
      
      // Snapshot previous value
      const previousTodos = queryClient.getQueryData(['todos']);
      
      // Optimistically update
      queryClient.setQueryData(['todos'], old => [
        ...old,
        { ...newTodo, id: 'temp-' + Date.now(), pending: true }
      ]);
      
      return { previousTodos };
    },
    onError: (err, newTodo, context) => {
      // Rollback on error
      queryClient.setQueryData(['todos'], context.previousTodos);
      toast.error('Failed to add todo');
    },
    onSuccess: (data, variables, context) => {
      toast.success('Todo added!');
    },
    onSettled: () => {
      // Refetch after success or error
      queryClient.invalidateQueries(['todos']);
    }
  });
  
  const deleteMutation = useMutation({
    mutationFn: deleteTodo,
    onMutate: async (todoId) => {
      await queryClient.cancelQueries(['todos']);
      const previousTodos = queryClient.getQueryData(['todos']);
      
      queryClient.setQueryData(['todos'], old =>
        old.filter(t => t.id !== todoId)
      );
      
      return { previousTodos };
    },
    onError: (err, todoId, context) => {
      queryClient.setQueryData(['todos'], context.previousTodos);
      toast.error('Failed to delete');
    }
  });
  
  return (
    <ul>
      {todos?.map(todo => (
        <li
          key={todo.id}
          style={{ opacity: todo.pending ? 0.5 : 1 }}
        >
          {todo.text}
          <button 
            onClick={() => deleteMutation.mutate(todo.id)}
            disabled={todo.pending}
          >
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

## 25.4 File Upload with Progress

```jsx
function FileUpload({ onUpload }) {
  const [files, setFiles] = useState([]);
  const [uploading, setUploading] = useState(false);
  
  const handleDrop = useCallback((e) => {
    e.preventDefault();
    const droppedFiles = Array.from(e.dataTransfer.files);
    setFiles(prev => [
      ...prev,
      ...droppedFiles.map(file => ({
        file,
        id: Date.now() + Math.random(),
        progress: 0,
        status: 'pending'
      }))
    ]);
  }, []);
  
  const uploadFile = async (fileObj) => {
    const formData = new FormData();
    formData.append('file', fileObj.file);
    
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      
      xhr.upload.onprogress = (e) => {
        if (e.lengthComputable) {
          const progress = Math.round((e.loaded / e.total) * 100);
          setFiles(prev => prev.map(f =>
            f.id === fileObj.id ? { ...f, progress } : f
          ));
        }
      };
      
      xhr.onload = () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          setFiles(prev => prev.map(f =>
            f.id === fileObj.id ? { ...f, status: 'complete', progress: 100 } : f
          ));
          resolve(JSON.parse(xhr.response));
        } else {
          reject(new Error('Upload failed'));
        }
      };
      
      xhr.onerror = () => {
        setFiles(prev => prev.map(f =>
          f.id === fileObj.id ? { ...f, status: 'error' } : f
        ));
        reject(new Error('Upload failed'));
      };
      
      xhr.open('POST', '/api/upload');
      xhr.send(formData);
    });
  };
  
  const handleUpload = async () => {
    setUploading(true);
    
    const pendingFiles = files.filter(f => f.status === 'pending');
    const results = await Promise.allSettled(pendingFiles.map(uploadFile));
    
    setUploading(false);
    onUpload?.(results);
  };
  
  const removeFile = (id) => {
    setFiles(prev => prev.filter(f => f.id !== id));
  };
  
  return (
    <div
      onDrop={handleDrop}
      onDragOver={(e) => e.preventDefault()}
      className="upload-zone"
    >
      <p>Drag files here or click to select</p>
      
      <input
        type="file"
        multiple
        onChange={(e) => handleDrop({ 
          dataTransfer: { files: e.target.files }, 
          preventDefault: () => {} 
        })}
      />
      
      {files.length > 0 && (
        <div className="file-list">
          {files.map(f => (
            <div key={f.id} className="file-item">
              <span>{f.file.name}</span>
              <div className="progress-bar">
                <div 
                  className="progress-fill"
                  style={{ width: `${f.progress}%` }}
                />
              </div>
              <span>{f.status}</span>
              <button onClick={() => removeFile(f.id)}>×</button>
            </div>
          ))}
        </div>
      )}
      
      <button onClick={handleUpload} disabled={uploading}>
        {uploading ? 'Uploading...' : 'Upload All'}
      </button>
    </div>
  );
}
```

---

# 26. React Best Practices Summary

## 26.1 Component Design

1. **Single Responsibility**: Each component should do one thing well
2. **Composition over Inheritance**: Build complex UIs from simple components
3. **Controlled vs Uncontrolled**: Prefer controlled for form inputs
4. **Smart/Dumb Split**: Separate logic (containers) from presentation
5. **Prop Drilling**: Use Context for deeply nested props

## 26.2 State Management

1. **Local First**: Start with useState, lift when needed
2. **Colocation**: Keep state close to where it's used
3. **Derived State**: Compute values instead of syncing state
4. **Server State**: Use TanStack Query/SWR for API data
5. **Global State**: Zustand/Redux for truly global client state

## 26.3 Performance

1. **Measure First**: Use Profiler before optimizing
2. **Memoize Wisely**: React.memo, useMemo, useCallback when needed
3. **Virtualize Lists**: Use react-virtual for long lists
4. **Code Split**: Lazy load routes and heavy components
5. **Avoid Inline Objects**: In JSX props and dependencies

## 26.4 Testing

1. **User-Centric**: Test behavior, not implementation
2. **Integration Over Unit**: Test components together
3. **Avoid Snapshot**: Use explicit assertions
4. **Mock APIs**: Not internal modules
5. **Accessibility**: Test with screen readers

---

## 🎯 Final Study Checklist for React

### Core Concepts
- [ ] JSX and Virtual DOM
- [ ] Function vs Class components
- [ ] Props and State
- [ ] Component lifecycle
- [ ] Event handling

### Hooks
- [ ] useState, useReducer
- [ ] useEffect, useLayoutEffect
- [ ] useRef, useImperativeHandle
- [ ] useContext
- [ ] useMemo, useCallback
- [ ] Custom hooks

### Advanced Patterns
- [ ] HOC, Render Props
- [ ] Compound Components
- [ ] Context optimization
- [ ] Error Boundaries
- [ ] Portals

### Performance
- [ ] React.memo
- [ ] Code splitting
- [ ] Virtualization
- [ ] Profiler usage

### Ecosystem
- [ ] React Router
- [ ] State libraries (Zustand, Redux)
- [ ] Data fetching (TanStack Query, SWR)
- [ ] Testing (RTL, Jest)

### React 18+
- [ ] Concurrent features
- [ ] Automatic batching
- [ ] useTransition, useDeferredValue
- [ ] Server Components

---

# 27. Bonus: Quick React Interview Answers

## Common One-Liner Answers

**What is React?**
React is a JavaScript library for building user interfaces using a declarative, component-based approach with a Virtual DOM for efficient updates.

**What is the Virtual DOM?**
A lightweight JavaScript representation of the real DOM that React uses to optimize updates by minimizing actual DOM manipulations.

**What are hooks?**
Functions that let you "hook into" React state and lifecycle features from function components, introduced in React 16.8.

**Why use keys in lists?**
Keys help React identify which items changed, were added, or removed, enabling efficient re-rendering without recreating all elements.

**useState vs useReducer?**
useState for simple state; useReducer for complex state logic with multiple related values or when next state depends on previous.

**useEffect vs useLayoutEffect?**
useEffect runs asynchronously after paint; useLayoutEffect runs synchronously before paint—use for DOM measurements.

**What is Context?**
A way to pass data through the component tree without prop drilling, ideal for global state like themes or authentication.

**What is React.memo?**
A higher-order component that memoizes function components, preventing re-renders when props haven't changed.

**What are Error Boundaries?**
Class components that catch JavaScript errors in their child tree, log them, and display a fallback UI.

**What is Concurrent Mode?**
React's ability to interrupt rendering for urgent updates, enabling features like Suspense, useTransition, and useDeferredValue.

---

## Code Snippets for Quick Reference

```jsx
// useState
const [value, setValue] = useState(initial);

// useEffect
useEffect(() => { /* effect */ return () => { /* cleanup */ }; }, [deps]);

// useRef
const ref = useRef(null);

// useContext
const value = useContext(MyContext);

// useMemo
const memoized = useMemo(() => compute(a, b), [a, b]);

// useCallback
const callback = useCallback(() => fn(a), [a]);

// useReducer
const [state, dispatch] = useReducer(reducer, initial);

// Custom Hook
function useCustom() { const [x] = useState(); return x; }

// forwardRef
const Comp = forwardRef((props, ref) => <input ref={ref} />);

// lazy + Suspense
const Lazy = lazy(() => import('./Lazy'));
<Suspense fallback={<Loading />}><Lazy /></Suspense>
```

---

## Event Handler Quick Reference

```jsx
// onClick
<button onClick={handleClick}>Click</button>
<button onClick={() => handleClick(id)}>Click</button>

// onChange
<input onChange={(e) => setValue(e.target.value)} />

// onSubmit
<form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>

// onKeyDown
<input onKeyDown={(e) => e.key === 'Enter' && submit()} />

// onFocus/onBlur
<input onFocus={handleFocus} onBlur={handleBlur} />

// onMouseEnter/onMouseLeave
<div onMouseEnter={show} onMouseLeave={hide}>Hover</div>
```

---

## TypeScript Quick Reference for React

```tsx
// Component Props
interface Props {
  name: string;
  age?: number;
  onClick: (id: string) => void;
  children: React.ReactNode;
}

// Function Component
const Component: React.FC<Props> = ({ name }) => <div>{name}</div>;
// Or preferred:
function Component({ name }: Props): JSX.Element { return <div>{name}</div>; }

// Event Types
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {};
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {};
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {};
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {};

// Ref Types
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);

// State Types
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// Context Types
const MyContext = createContext<ContextType | undefined>(undefined);
```

---

## React Router v6 Quick Reference

```jsx
// Routes
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users/:id" element={<User />} />
  <Route path="*" element={<NotFound />} />
</Routes>

// Hooks
const params = useParams();           // { id: '123' }
const navigate = useNavigate();       // navigate('/path')
const location = useLocation();       // { pathname, search, state }
const [searchParams] = useSearchParams();

// Navigation
<Link to="/about">About</Link>
<NavLink to="/about" className={({ isActive }) => isActive ? 'active' : ''}>

// Protected Route
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" replace />;
  return children;
}
```

---

## Common Gotchas

1. **Mutating state directly** - Always use setter with new reference
2. **Missing dependency array** - Causes effect to run every render
3. **Object/function in deps** - New reference each render causes loop
4. **Index as key** - Causes issues with reordering/deletion
5. **Async in useEffect** - Return cleanup, not promise
6. **Missing cleanup** - Memory leaks from subscriptions/timers
7. **Stale closures** - Use functional updates or refs
8. **Too many contexts** - Consider state libraries for complex state

---

*End of Part 3: React Deep Dive - Complete Interview Guide (6000+ lines)*

*Continue to Part 4 for React Hooks Complete Guide...*

