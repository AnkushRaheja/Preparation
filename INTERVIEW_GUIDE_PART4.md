# Part 4: React Hooks Complete Guide

> **Master every hook used in CodeHealth-AI**

---

## 4.1 useState (State Management)

### Basic Usage
```typescript
const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<string[]>([]);
```

### Lazy Initialization
```typescript
// ❌ BAD - Runs on every render
const [data, setData] = useState(expensiveComputation());

// ✅ GOOD - Only runs once
const [data, setData] = useState(() => expensiveComputation());
```

### Functional Updates (Critical for Closures)
```typescript
// ❌ Problem: Stale state in closures
const handleClick = () => {
  setCount(count + 1); // Uses stale `count` from closure
};

// ✅ Solution: Functional update
const handleClick = () => {
  setCount(prevCount => prevCount + 1); // Always gets latest
};
```

### Updating Objects/Arrays
```typescript
// Objects - Always create new reference
setUser(prev => ({ ...prev, name: 'New Name' }));

// Arrays - Add item
setItems(prev => [...prev, newItem]);

// Arrays - Remove item
setItems(prev => prev.filter(item => item.id !== id));

// Arrays - Update item
setItems(prev => prev.map(item => 
  item.id === id ? { ...item, name: 'Updated' } : item
));
```

---

## 4.2 useEffect (Side Effects)

### Dependency Array Explained
```typescript
// Runs on EVERY render (rarely needed)
useEffect(() => {
  console.log('Runs every render');
});

// Runs ONCE on mount (componentDidMount)
useEffect(() => {
  console.log('Runs only on mount');
}, []);

// Runs when deps change (componentDidUpdate for specific values)
useEffect(() => {
  console.log('userId changed:', userId);
}, [userId]);
```

### Cleanup Function
```typescript
// From CodeHealth useSocket.ts pattern
useEffect(() => {
  const socket = socketService.connect(token);
  
  socket.on('connect', () => setIsConnected(true));
  socket.on('disconnect', () => setIsConnected(false));

  // Cleanup: Runs before component unmounts AND before re-running effect
  return () => {
    socket.off('connect');
    socket.off('disconnect');
  };
}, [token]);
```

### Race Condition Prevention
```typescript
useEffect(() => {
  let isCancelled = false;

  async function fetchData() {
    const result = await fetch(`/api/user/${userId}`);
    const data = await result.json();
    
    // Only update state if component is still mounted
    if (!isCancelled) {
      setUser(data);
    }
  }

  fetchData();

  return () => {
    isCancelled = true;
  };
}, [userId]);
```

---

## 4.3 useCallback (Function Memoization)

**Purpose:** Returns a memoized function that only changes if dependencies change.

```typescript
// ❌ Creates new function every render
const handleClick = () => {
  console.log('Clicked');
};

// ✅ Same function reference across renders
const handleClick = useCallback(() => {
  console.log('Clicked');
}, []);

// With dependencies
const handleSubmit = useCallback((data: FormData) => {
  api.submit(data, userId); // Uses userId from scope
}, [userId]); // Recreates only when userId changes
```

### When to Use
```typescript
// Use when passing callbacks to memoized children
const Parent = () => {
  const [count, setCount] = useState(0);

  // Without useCallback, Child re-renders even when count changes
  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []);

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <MemoizedChild onClick={handleClick} />
    </>
  );
};

const MemoizedChild = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Child Button</button>;
});
```

---

## 4.4 useMemo (Value Memoization)

**Purpose:** Returns a memoized value that only recalculates when dependencies change.

```typescript
// ❌ Expensive calculation runs every render
const sortedItems = items.sort((a, b) => a.name.localeCompare(b.name));

// ✅ Only recalculates when items change
const sortedItems = useMemo(() => {
  return items.sort((a, b) => a.name.localeCompare(b.name));
}, [items]);
```

### Real CodeHealth Example
```typescript
// From observability/page.tsx pattern
const chartData = useMemo(() => {
  // Expensive: Transform 1000s of data points
  return rawData.map(point => ({
    date: formatDate(point.timestamp),
    value: calculateMetric(point),
    trend: computeTrend(point, previousPoint)
  }));
}, [rawData]); // Only recalculate when rawData changes
```

### useMemo vs useCallback
```typescript
// useMemo - caches the VALUE
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// useCallback - caches the FUNCTION
const memoizedFn = useCallback(() => doSomething(a, b), [a, b]);

// They're equivalent:
useCallback(fn, deps) === useMemo(() => fn, deps)
```

---

## 4.5 useRef (Mutable Refs)

### DOM Access
```typescript
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

### Mutable Values (That Don't Trigger Re-render)
```typescript
// From CodeHealth useSocket.ts
function useSocket() {
  const hasInitializedRef = useRef(false);
  const heartbeatIntervalRef = useRef<NodeJS.Timeout | null>(null);

  useEffect(() => {
    // Prevents double initialization in React 18 Strict Mode
    if (hasInitializedRef.current) return;
    hasInitializedRef.current = true;

    // Setup heartbeat
    heartbeatIntervalRef.current = setInterval(() => {
      socketService.ping();
    }, 30000);

    return () => {
      if (heartbeatIntervalRef.current) {
        clearInterval(heartbeatIntervalRef.current);
      }
    };
  }, []);
}
```

### Previous Value Pattern
```typescript
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current; // Returns previous value (before useEffect runs)
}

// Usage
function Component({ count }: { count: number }) {
  const prevCount = usePrevious(count);
  
  return (
    <p>Current: {count}, Previous: {prevCount}</p>
  );
}
```

---

## 4.6 useContext (Global State Access)

### Creating Context
```typescript
// ThemeContext.tsx
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook for consuming
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

### Usage
```typescript
function NavBar() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <nav className={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </nav>
  );
}
```

---

## 4.7 useLayoutEffect (Synchronous Effects)

**Difference from useEffect:**
- `useEffect`: Runs AFTER browser paint (async)
- `useLayoutEffect`: Runs BEFORE browser paint (sync, blocks rendering)

```typescript
// Use when you need to measure DOM before user sees it
function Tooltip({ children, targetRef }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef<HTMLDivElement>(null);

  useLayoutEffect(() => {
    // Calculate position synchronously before paint
    // Prevents tooltip from "jumping" after appearing
    const targetRect = targetRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current?.getBoundingClientRect();
    
    setPosition({
      top: targetRect.top - (tooltipRect?.height || 0),
      left: targetRect.left
    });
  }, [targetRef]);

  return (
    <div ref={tooltipRef} style={{ top: position.top, left: position.left }}>
      {children}
    </div>
  );
}
```

---

## 4.8 Custom Hooks

### Rules for Custom Hooks
1. Name must start with `use`
2. Can call other hooks inside
3. Each call to hook gets its own state

### CodeHealth Examples

**useSocket (Simplified)**
```typescript
function useSocket() {
  const [isConnected, setIsConnected] = useState(false);
  const [socket, setSocket] = useState<Socket | null>(null);

  const connect = useCallback(() => {
    const token = localStorage.getItem('authToken');
    if (!token) return;

    const socketInstance = socketService.connect(token);
    setSocket(socketInstance);
    
    socketInstance.on('connect', () => setIsConnected(true));
    socketInstance.on('disconnect', () => setIsConnected(false));
  }, []);

  const disconnect = useCallback(() => {
    socketService.disconnect();
    setSocket(null);
    setIsConnected(false);
  }, []);

  return { socket, isConnected, connect, disconnect };
}
```

**useLocalStorage**
```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value: T | ((val: T) => T)) => {
    setStoredValue(prev => {
      const valueToStore = value instanceof Function ? value(prev) : value;
      localStorage.setItem(key, JSON.stringify(valueToStore));
      return valueToStore;
    });
  }, [key]);

  return [storedValue, setValue] as const;
}
```

---

## 🎯 Part 4 Interview Questions

1. **What is the difference between useState and useRef?**
2. **When would you use useCallback vs useMemo?**
3. **Explain the useEffect dependency array.**
4. **How do you prevent stale closures in useEffect?**
5. **What is the cleanup function in useEffect?**
6. **When would you use useLayoutEffect?**
7. **How do you create a custom hook?**
8. **What are the rules of hooks?**
9. **How does useContext work?**
10. **What is the purpose of React.memo?**

---

*Continue to Part 5 for Next.js Framework...*
