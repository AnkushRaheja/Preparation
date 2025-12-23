# Part 3: React.js Deep Dive

> **Master React internals and architecture for senior-level interviews**

---

## 3.1 How React Works Under the Hood

### The Virtual DOM
The **Virtual DOM** is a lightweight JavaScript representation of the actual DOM.

**Why Virtual DOM?**
- Real DOM operations are slow (browser repaints, reflows)
- Virtual DOM is just JavaScript objects (fast to create/compare)
- React batches changes and applies minimal updates

```javascript
// Virtual DOM representation of <div class="container"><span>Hello</span></div>
const vdom = {
  type: 'div',
  props: {
    className: 'container',
    children: {
      type: 'span',
      props: {
        children: 'Hello'
      }
    }
  }
};
```

### Reconciliation (Diffing Algorithm)
When state changes, React:
1. Creates a new Virtual DOM tree
2. Compares it with the previous tree (diffing)
3. Calculates minimal changes needed
4. Applies only those changes to the real DOM

**Diffing Rules:**
1. **Different element types**: Replace entire subtree
2. **Same element type**: Update attributes only
3. **Lists without keys**: Re-render everything
4. **Lists with keys**: Efficiently move/add/remove items

### React Fiber (React 16+)
**Fiber** is React's new reconciliation engine that enables:
- **Incremental rendering**: Split rendering work into chunks
- **Priority-based updates**: Handle high-priority updates (user input) before low-priority (data fetching)
- **Pausable/Resumable work**: Pause rendering to handle urgent tasks

```javascript
// Fiber enables features like:
// - Suspense
// - Concurrent Mode
// - useTransition
// - useDeferredValue
```

---

## 3.2 Component Types

### Functional Components (Modern - Used in CodeHealth)
```typescript
// Simple component
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

// With explicit return type
const UserCard: React.FC<{ user: User }> = ({ user }) => {
  return (
    <div className="card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

### Class Components (Legacy - Know for Interviews)
```typescript
class Counter extends React.Component<Props, State> {
  state = { count: 0 };

  componentDidMount() {
    console.log('Component mounted');
  }

  componentDidUpdate(prevProps: Props, prevState: State) {
    if (prevState.count !== this.state.count) {
      console.log('Count changed');
    }
  }

  componentWillUnmount() {
    console.log('Component will unmount');
  }

  render() {
    return (
      <button onClick={() => this.setState({ count: this.state.count + 1 })}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

---

## 3.3 JSX In Depth

JSX is NOT HTML. It's syntactic sugar for `React.createElement()`.

```jsx
// JSX
<div className="container">
  <h1>Hello</h1>
  <p>World</p>
</div>

// Compiles to:
React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hello'),
  React.createElement('p', null, 'World')
);
```

### JSX Rules
```jsx
// 1. Must return single root element
return (
  <>  {/* Fragment - no extra DOM node */}
    <h1>Title</h1>
    <p>Content</p>
  </>
);

// 2. All tags must be closed
<img src="photo.jpg" />  // Self-closing
<br />

// 3. camelCase for attributes
<div className="card" onClick={handleClick} tabIndex={0}>

// 4. JavaScript expressions in curly braces
<p>Total: {price * quantity}</p>
<p>{isLoggedIn ? 'Welcome!' : 'Please login'}</p>

// 5. Inline styles as objects
<div style={{ backgroundColor: 'blue', fontSize: '16px' }}>
```

---

## 3.4 Props vs State

### Props (External Data - Immutable)
```typescript
// Parent passes data down
function Parent() {
  return <Child name="Jayesh" age={25} />;
}

// Child receives via props
function Child({ name, age }: { name: string; age: number }) {
  // ❌ Cannot modify props
  // name = "John"; // Error!
  return <p>{name} is {age} years old</p>;
}
```

### State (Internal Data - Mutable)
```typescript
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### State Updates Can Be Batched
```typescript
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
  // Result: count only increases by 1!
  // React batches these into single update
}

// Fix: Use functional updates
function handleClickCorrect() {
  setCount(c => c + 1);
  setCount(c => c + 1);
  setCount(c => c + 1);
  // Result: count increases by 3
}
```

---

## 3.5 Component Lifecycle in Functional Components

| Phase | Class Lifecycle | Hook Equivalent |
|-------|-----------------|-----------------|
| Mount | `componentDidMount` | `useEffect(() => {}, [])` |
| Update | `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| Unmount | `componentWillUnmount` | `useEffect(() => { return cleanup }, [])` |

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  // Runs on mount (empty deps) - like componentDidMount
  useEffect(() => {
    console.log('Component mounted');
    
    // Cleanup - like componentWillUnmount
    return () => {
      console.log('Component will unmount');
    };
  }, []);

  // Runs when userId changes - like componentDidUpdate
  useEffect(() => {
    console.log('userId changed, fetching new user');
    fetchUser(userId).then(setUser);
  }, [userId]);

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

---

## 3.6 Keys in Lists (Critical Interview Topic)

### Why Keys Matter
Keys help React identify which items have changed, been added, or removed.

```jsx
// ❌ BAD - Using index as key
{items.map((item, index) => (
  <ListItem key={index} item={item} />
))}
// Problem: If items reorder, React reuses wrong components

// ✅ GOOD - Using unique identifier
{items.map(item => (
  <ListItem key={item.id} item={item} />
))}
```

### What Happens Without Keys?
```jsx
// Initial: [A, B, C]
// After:   [B, C, A]

// Without proper keys:
// React thinks A, B, C all changed content
// Re-renders ALL three items

// With proper keys:
// React knows to just move A to the end
// Minimal DOM manipulation
```

---

## 3.7 Conditional Rendering

```jsx
// 1. If-else (outside JSX)
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

// 2. Ternary (inline)
return (
  <div>
    {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
  </div>
);

// 3. Logical AND (for "show or nothing")
return (
  <div>
    {hasNotifications && <NotificationBadge />}
  </div>
);

// 4. Nullish coalescing (default values)
return <p>{user.nickname ?? user.name}</p>;
```

---

## 3.8 Event Handling

```jsx
function Button() {
  // Event handler
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    event.preventDefault();
    console.log('Button clicked');
  };

  // Passing parameters
  const handleItemClick = (id: string) => () => {
    console.log('Clicked item:', id);
  };

  return (
    <>
      <button onClick={handleClick}>Click Me</button>
      <button onClick={handleItemClick('123')}>Item</button>
      
      {/* Inline (not recommended for complex logic) */}
      <button onClick={(e) => console.log(e.target)}>Inline</button>
    </>
  );
}
```

### SyntheticEvent
React wraps native events in SyntheticEvent for cross-browser compatibility.
```typescript
function Form() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault(); // Works same as native
    // Form submission logic
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" onChange={handleChange} />
    </form>
  );
}
```

---

## 3.9 Composition vs Inheritance

**React favors composition over inheritance.**

```jsx
// ❌ Inheritance approach (Don't do this)
class SpecialButton extends Button {
  render() { /* ... */ }
}

// ✅ Composition approach (Do this)
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

function App() {
  return (
    <Card title="Welcome">
      <p>This is the card content</p>
      <Button>Click me</Button>
    </Card>
  );
}
```

### Specialization Pattern
```jsx
// Generic component
function Dialog({ title, message, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <p>{message}</p>
      {children}
    </div>
  );
}

// Specialized components
function WelcomeDialog() {
  return (
    <Dialog title="Welcome!" message="Thanks for joining us!">
      <button>Continue</button>
    </Dialog>
  );
}

function ErrorDialog({ error }) {
  return (
    <Dialog title="Error" message={error}>
      <button>Retry</button>
    </Dialog>
  );
}
```

---

## 🎯 Part 3 Interview Questions

1. **What is the Virtual DOM and why is it used?**
2. **Explain the reconciliation process.**
3. **What is React Fiber?**
4. **What's the difference between props and state?**
5. **Why do we need keys in lists?**
6. **What are controlled vs uncontrolled components?**
7. **Explain the component lifecycle.**
8. **What is prop drilling and how do you solve it?**
9. **What is the children prop?**
10. **Why does React favor composition over inheritance?**

---

*Continue to Part 4 for React Hooks Complete Guide...*
