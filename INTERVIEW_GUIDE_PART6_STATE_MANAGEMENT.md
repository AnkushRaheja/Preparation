# 📚 Part 6: State Management Complete Guide - Interview Mastery

> **The Ultimate State Management Interview Preparation Guide**
> 
> Master Zustand, React Context, Redux Toolkit, and modern state management patterns for technical interviews.

---

## 📋 Table of Contents

1. [Introduction to State Management](#1-introduction-to-state-management)
2. [React Built-in State](#2-react-built-in-state)
3. [Context API Deep Dive](#3-context-api-deep-dive)
4. [Zustand Complete Guide](#4-zustand-complete-guide)
5. [Redux Toolkit](#5-redux-toolkit)
6. [Jotai](#6-jotai)
7. [Recoil](#7-recoil)
8. [MobX](#8-mobx)
9. [State Machine with XState](#9-state-machine-with-xstate)
10. [Server State Management](#10-server-state-management)
11. [Comparison and When to Use What](#11-comparison-and-when-to-use-what)
12. [Interview Questions](#12-interview-questions)
13. [Coding Challenges](#13-coding-challenges)

---

# 1. Introduction to State Management

## 1.1 What is State?

State represents data that can change over time and affects what the UI renders.

```tsx
// Types of State
interface StateTypes {
  // Local State - Single component
  localState: {
    inputValue: string;
    isOpen: boolean;
    count: number;
  };
  
  // Shared State - Multiple components
  sharedState: {
    theme: 'light' | 'dark';
    user: User | null;
    cart: CartItem[];
  };
  
  // Server State - Remote data
  serverState: {
    products: Product[];
    isLoading: boolean;
    error: Error | null;
  };
  
  // URL State - Browser URL
  urlState: {
    searchParams: URLSearchParams;
    pathname: string;
  };
  
  // Form State - Form data
  formState: {
    values: Record<string, any>;
    errors: Record<string, string>;
    touched: Record<string, boolean>;
  };
}
```

## 1.2 When Do You Need State Management?

| Scenario | Solution |
|----------|----------|
| Single component state | `useState` |
| Complex component state | `useReducer` |
| Share state with children | Props drilling or Context |
| Share state across app | Zustand, Redux, or Context |
| Server data | React Query, SWR, RTK Query |
| Form state | React Hook Form, Formik |
| URL state | Router (Next.js, React Router) |

## 1.3 State Management Evolution

```
2013: Flux (Facebook)
  ↓
2015: Redux (Dan Abramov)
  ↓
2019: Context + Hooks become viable
  ↓
2020: Recoil, Jotai (Atomic state)
  ↓
2021: Zustand gains popularity
  ↓
2023+: Server Components change paradigm
```

---

# 2. React Built-in State

## 2.1 useState

```tsx
import { useState } from 'react';

// Basic usage
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// Lazy initialization
function ExpensiveComponent() {
  const [data, setData] = useState(() => {
    // Only runs on first render
    return computeExpensiveInitialValue();
  });
}

// Object state
function Form() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: '',
  });
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: value,
    }));
  };
}

// Array state
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const addTodo = (text: string) => {
    setTodos(prev => [...prev, { id: Date.now(), text, done: false }]);
  };
  
  const toggleTodo = (id: number) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, done: !todo.done } : todo
      )
    );
  };
  
  const removeTodo = (id: number) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
}
```

## 2.2 useReducer

```tsx
import { useReducer } from 'react';

// Define state and actions
interface State {
  count: number;
  loading: boolean;
  error: string | null;
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_COUNT'; payload: number }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null };

// Reducer function
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET_COUNT':
      return { ...state, count: action.payload };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload };
    default:
      return state;
  }
}

const initialState: State = {
  count: 0,
  loading: false,
  error: null,
};

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'SET_COUNT', payload: 0 })}>
        Reset
      </button>
    </div>
  );
}

// Lazy initialization
const [state, dispatch] = useReducer(reducer, initialArg, init);

function init(initialArg: number): State {
  return { count: initialArg, loading: false, error: null };
}
```

## 2.3 useState vs useReducer

| useState | useReducer |
|----------|------------|
| Simple state | Complex state logic |
| Independent values | Interdependent values |
| Primitive types | Objects with many fields |
| Few transitions | Many state transitions |
| Quick updates | Need for action history |

---

# 3. Context API Deep Dive

## 3.1 Basic Context Pattern

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Create context
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 2. Create provider
function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Create custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// 4. Usage
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      onClick={toggleTheme}
      style={{ background: theme === 'dark' ? '#333' : '#fff' }}
    >
      Toggle Theme
    </button>
  );
}

// App setup
function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  );
}
```

## 3.2 Context with Reducer

```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

// State and Actions
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
}

type CartAction =
  | { type: 'ADD_ITEM'; payload: CartItem }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' };

// Reducer
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(i => i.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          total: state.total + action.payload.price,
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
        total: state.total + action.payload.price,
      };
    }
    case 'REMOVE_ITEM': {
      const item = state.items.find(i => i.id === action.payload);
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload),
        total: state.total - (item ? item.price * item.quantity : 0),
      };
    }
    case 'UPDATE_QUANTITY': {
      const item = state.items.find(i => i.id === action.payload.id);
      if (!item) return state;
      const diff = action.payload.quantity - item.quantity;
      return {
        ...state,
        items: state.items.map(i =>
          i.id === action.payload.id
            ? { ...i, quantity: action.payload.quantity }
            : i
        ),
        total: state.total + (diff * item.price),
      };
    }
    case 'CLEAR_CART':
      return { items: [], total: 0 };
    default:
      return state;
  }
}

// Context
interface CartContextType {
  state: CartState;
  dispatch: React.Dispatch<CartAction>;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

const CartContext = createContext<CartContextType | undefined>(undefined);

// Provider
function CartProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [], total: 0 });
  
  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    dispatch({ type: 'ADD_ITEM', payload: { ...item, quantity: 1 } });
  };
  
  const removeItem = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  };
  
  const updateQuantity = (id: string, quantity: number) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
  };
  
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  return (
    <CartContext.Provider
      value={{ state, dispatch, addItem, removeItem, updateQuantity, clearCart }}
    >
      {children}
    </CartContext.Provider>
  );
}

// Hook
function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}
```

## 3.3 Context Performance Optimization

```tsx
import { createContext, useContext, useMemo, useCallback, useState } from 'react';

// Problem: All consumers re-render when any value changes
// Solution 1: Split contexts

const UserStateContext = createContext<User | null>(null);
const UserDispatchContext = createContext<((user: User | null) => void) | null>(null);

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={setUser}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}

function useUserState() {
  const context = useContext(UserStateContext);
  return context;
}

function useUserDispatch() {
  const context = useContext(UserDispatchContext);
  if (!context) throw new Error('Must be within UserProvider');
  return context;
}

// Solution 2: Memoize context value
function OptimizedProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = useCallback(async (credentials: Credentials) => {
    const user = await authService.login(credentials);
    setUser(user);
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
  }, []);
  
  const value = useMemo(
    () => ({ user, login, logout }),
    [user, login, logout]
  );
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Solution 3: Use selectors with useSyncExternalStore
import { useSyncExternalStore } from 'react';

function useSelector<T, S>(
  store: Store<T>,
  selector: (state: T) => S
): S {
  return useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState()),
    () => selector(store.getState())
  );
}
```

---

# 4. Zustand Complete Guide

## 4.1 Introduction to Zustand

Zustand is a small, fast, and scalable state management solution.

```bash
npm install zustand
```

## 4.2 Basic Store

```tsx
import { create } from 'zustand';

// Simple counter store
interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));

// Usage in component
function Counter() {
  const { count, increment, decrement, reset } = useCounterStore();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Select specific state (prevents unnecessary re-renders)
function DisplayCount() {
  const count = useCounterStore((state) => state.count);
  return <p>Count: {count}</p>;
}

function IncrementButton() {
  const increment = useCounterStore((state) => state.increment);
  return <button onClick={increment}>+</button>;
}
```

## 4.3 Advanced Zustand Patterns

```tsx
import { create } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Complex store with TypeScript
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

interface TodoStore {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  addTodo: (text: string) => void;
  toggleTodo: (id: string) => void;
  removeTodo: (id: string) => void;
  setFilter: (filter: 'all' | 'active' | 'completed') => void;
  clearCompleted: () => void;
}

const useTodoStore = create<TodoStore>()(
  devtools(
    persist(
      immer((set) => ({
        todos: [],
        filter: 'all',
        
        addTodo: (text) => set((state) => {
          state.todos.push({
            id: Date.now().toString(),
            text,
            completed: false,
          });
        }),
        
        toggleTodo: (id) => set((state) => {
          const todo = state.todos.find((t) => t.id === id);
          if (todo) {
            todo.completed = !todo.completed;
          }
        }),
        
        removeTodo: (id) => set((state) => {
          state.todos = state.todos.filter((t) => t.id !== id);
        }),
        
        setFilter: (filter) => set({ filter }),
        
        clearCompleted: () => set((state) => {
          state.todos = state.todos.filter((t) => !t.completed);
        }),
      })),
      {
        name: 'todo-storage',
      }
    ),
    { name: 'TodoStore' }
  )
);

// Computed values / selectors
const selectFilteredTodos = (state: TodoStore) => {
  switch (state.filter) {
    case 'active':
      return state.todos.filter((t) => !t.completed);
    case 'completed':
      return state.todos.filter((t) => t.completed);
    default:
      return state.todos;
  }
};

const selectTodoCount = (state: TodoStore) => ({
  total: state.todos.length,
  active: state.todos.filter((t) => !t.completed).length,
  completed: state.todos.filter((t) => t.completed).length,
});

// Usage with selectors
function TodoList() {
  const filteredTodos = useTodoStore(selectFilteredTodos);
  const { toggleTodo, removeTodo } = useTodoStore();
  
  return (
    <ul>
      {filteredTodos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => removeTodo(todo.id)}>×</button>
        </li>
      ))}
    </ul>
  );
}
```

## 4.4 Zustand with Async Actions

```tsx
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserStore {
  user: User | null;
  loading: boolean;
  error: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  fetchUser: () => Promise<void>;
}

const useUserStore = create<UserStore>((set, get) => ({
  user: null,
  loading: false,
  error: null,
  
  login: async (email, password) => {
    set({ loading: true, error: null });
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      const user = await response.json();
      set({ user, loading: false });
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Unknown error',
        loading: false 
      });
    }
  },
  
  logout: () => {
    set({ user: null });
  },
  
  fetchUser: async () => {
    const { user } = get();
    if (user) return; // Already have user
    
    set({ loading: true });
    try {
      const response = await fetch('/api/me');
      const userData = await response.json();
      set({ user: userData, loading: false });
    } catch (error) {
      set({ loading: false });
    }
  },
}));
```

## 4.5 Zustand Slices Pattern

```tsx
import { create, StateCreator } from 'zustand';

// User slice
interface UserSlice {
  user: User | null;
  setUser: (user: User | null) => void;
}

const createUserSlice: StateCreator<
  UserSlice & CartSlice & UISlice,
  [],
  [],
  UserSlice
> = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
});

// Cart slice
interface CartSlice {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

const createCartSlice: StateCreator<
  UserSlice & CartSlice & UISlice,
  [],
  [],
  CartSlice
> = (set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ 
    items: state.items.filter((i) => i.id !== id) 
  })),
  clearCart: () => set({ items: [] }),
});

// UI slice
interface UISlice {
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

const createUISlice: StateCreator<
  UserSlice & CartSlice & UISlice,
  [],
  [],
  UISlice
> = (set) => ({
  sidebarOpen: false,
  theme: 'light',
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setTheme: (theme) => set({ theme }),
});

// Combined store
type StoreState = UserSlice & CartSlice & UISlice;

const useStore = create<StoreState>()((...args) => ({
  ...createUserSlice(...args),
  ...createCartSlice(...args),
  ...createUISlice(...args),
}));

export default useStore;
```

## 4.6 Testing Zustand Stores

```tsx
import { act, renderHook } from '@testing-library/react';
import { useCounterStore } from './counterStore';

describe('counterStore', () => {
  beforeEach(() => {
    // Reset store before each test
    useCounterStore.setState({ count: 0 });
  });
  
  it('should initialize with count 0', () => {
    const { result } = renderHook(() => useCounterStore());
    expect(result.current.count).toBe(0);
  });
  
  it('should increment count', () => {
    const { result } = renderHook(() => useCounterStore());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('should decrement count', () => {
    useCounterStore.setState({ count: 5 });
    const { result } = renderHook(() => useCounterStore());
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
});
```

---

# 5. Redux Toolkit

## 5.1 Setting Up Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

## 5.2 Creating a Slice

```tsx
// features/counter/counterSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
    reset: () => initialState,
  },
});

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;
export default counterSlice.reducer;
```

## 5.3 Configuring the Store

```tsx
// app/store.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';
import todosReducer from '../features/todos/todosSlice';
import userReducer from '../features/user/userSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    todos: todosReducer,
    user: userReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// app/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

## 5.4 Provider Setup

```tsx
// main.tsx or _app.tsx
import { Provider } from 'react-redux';
import { store } from './app/store';

function App() {
  return (
    <Provider store={store}>
      <YourApp />
    </Provider>
  );
}
```

## 5.5 Usage in Components

```tsx
import { useAppDispatch, useAppSelector } from '../app/hooks';
import { increment, decrement, incrementByAmount } from './counterSlice';

function Counter() {
  const count = useAppSelector((state) => state.counter.value);
  const dispatch = useAppDispatch();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}
```

## 5.6 Async Thunks

```tsx
// features/user/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null,
};

// Async thunk
export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId: string, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error('Failed to fetch user');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

export const loginUser = createAsyncThunk(
  'user/login',
  async (credentials: { email: string; password: string }, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });
      if (!response.ok) {
        throw new Error('Login failed');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
      state.error = null;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      // fetchUser
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action: PayloadAction<User>) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      })
      // loginUser
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action: PayloadAction<User>) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export const { logout, clearError } = userSlice.actions;
export default userSlice.reducer;
```

## 5.7 RTK Query

```tsx
// services/api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

interface Post {
  id: number;
  title: string;
  body: string;
  userId: number;
}

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Post', 'User'],
  endpoints: (builder) => ({
    // Query endpoints
    getPosts: builder.query<Post[], void>({
      query: () => '/posts',
      providesTags: (result) =>
        result
          ? [...result.map(({ id }) => ({ type: 'Post' as const, id })), 'Post']
          : ['Post'],
    }),
    
    getPost: builder.query<Post, number>({
      query: (id) => `/posts/${id}`,
      providesTags: (result, error, id) => [{ type: 'Post', id }],
    }),
    
    // Mutation endpoints
    createPost: builder.mutation<Post, Omit<Post, 'id'>>({
      query: (post) => ({
        url: '/posts',
        method: 'POST',
        body: post,
      }),
      invalidatesTags: ['Post'],
    }),
    
    updatePost: builder.mutation<Post, Post>({
      query: ({ id, ...post }) => ({
        url: `/posts/${id}`,
        method: 'PUT',
        body: post,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Post', id }],
    }),
    
    deletePost: builder.mutation<void, number>({
      query: (id) => ({
        url: `/posts/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: (result, error, id) => [{ type: 'Post', id }],
    }),
  }),
});

export const {
  useGetPostsQuery,
  useGetPostQuery,
  useCreatePostMutation,
  useUpdatePostMutation,
  useDeletePostMutation,
} = api;

// Usage in component
function PostList() {
  const { data: posts, isLoading, error } = useGetPostsQuery();
  const [createPost, { isLoading: isCreating }] = useCreatePostMutation();
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading posts</div>;
  
  return (
    <div>
      <button 
        onClick={() => createPost({ title: 'New', body: 'Content', userId: 1 })}
        disabled={isCreating}
      >
        Add Post
      </button>
      <ul>
        {posts?.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

# 6. Jotai

## 6.1 Introduction to Jotai

Jotai is an atomic state management library inspired by Recoil.

```bash
npm install jotai
```

## 6.2 Basic Atoms

```tsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Primitive atom
const countAtom = atom(0);

// Read-only derived atom
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Read-write derived atom
const multiplyAtom = atom(
  (get) => get(countAtom),
  (get, set, multiplier: number) => {
    set(countAtom, get(countAtom) * multiplier);
  }
);

// Usage
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtomValue(doubleCountAtom);
  const multiply = useSetAtom(multiplyAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
      <button onClick={() => multiply(2)}>×2</button>
    </div>
  );
}
```

## 6.3 Async Atoms

```tsx
import { atom, useAtom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// Async read atom
const userAtom = atom(async () => {
  const response = await fetch('/api/user');
  return response.json();
});

// Async write atom
const updateUserAtom = atom(
  null,
  async (get, set, newUser: User) => {
    const response = await fetch('/api/user', {
      method: 'PUT',
      body: JSON.stringify(newUser),
    });
    const updated = await response.json();
    set(userAtom, updated);
  }
);

// Atom with localStorage persistence
const themeAtom = atomWithStorage('theme', 'light');

// Usage with Suspense
import { Suspense } from 'react';

function UserProfile() {
  const [user] = useAtom(userAtom);
  return <div>Hello, {user.name}!</div>;
}

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile />
    </Suspense>
  );
}
```

## 6.4 Atom Families

```tsx
import { atom } from 'jotai';
import { atomFamily } from 'jotai/utils';

// Create a family of atoms
const todoAtomFamily = atomFamily(
  (id: string) => atom<Todo>({ id, text: '', completed: false })
);

// Usage
function TodoItem({ id }: { id: string }) {
  const [todo, setTodo] = useAtom(todoAtomFamily(id));
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => setTodo({ ...todo, completed: !todo.completed })}
      />
      <span>{todo.text}</span>
    </div>
  );
}
```

---

# 7. Recoil

## 7.1 Introduction to Recoil

```bash
npm install recoil
```

## 7.2 Atoms and Selectors

```tsx
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';
import { RecoilRoot } from 'recoil';

// Atoms - units of state
const textState = atom({
  key: 'textState', // unique ID
  default: '',
});

const todoListState = atom<Todo[]>({
  key: 'todoListState',
  default: [],
});

// Selectors - derived state
const charCountState = selector({
  key: 'charCountState',
  get: ({ get }) => {
    const text = get(textState);
    return text.length;
  },
});

const filteredTodoListState = selector({
  key: 'filteredTodoListState',
  get: ({ get }) => {
    const filter = get(todoListFilterState);
    const list = get(todoListState);
    
    switch (filter) {
      case 'completed':
        return list.filter((item) => item.completed);
      case 'uncompleted':
        return list.filter((item) => !item.completed);
      default:
        return list;
    }
  },
});

// Usage
function TextInput() {
  const [text, setText] = useRecoilState(textState);
  const charCount = useRecoilValue(charCountState);
  
  return (
    <div>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Type something..."
      />
      <p>Characters: {charCount}</p>
    </div>
  );
}

// App setup
function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  );
}
```

## 7.3 Async Selectors

```tsx
import { selector, selectorFamily, useRecoilValue } from 'recoil';
import { Suspense } from 'react';

// Async selector
const userSelector = selector({
  key: 'userSelector',
  get: async () => {
    const response = await fetch('/api/user');
    return response.json();
  },
});

// Parameterized async selector
const postSelector = selectorFamily({
  key: 'postSelector',
  get: (postId: number) => async () => {
    const response = await fetch(`/api/posts/${postId}`);
    return response.json();
  },
});

function UserProfile() {
  const user = useRecoilValue(userSelector);
  return <div>Hello, {user.name}!</div>;
}

function Post({ id }: { id: number }) {
  const post = useRecoilValue(postSelector(id));
  return <article>{post.title}</article>;
}

function App() {
  return (
    <RecoilRoot>
      <Suspense fallback={<div>Loading...</div>}>
        <UserProfile />
        <Post id={1} />
      </Suspense>
    </RecoilRoot>
  );
}
```

---

# 8. MobX

## 8.1 Introduction to MobX

```bash
npm install mobx mobx-react-lite
```

## 8.2 Observable State

```tsx
import { makeObservable, observable, action, computed } from 'mobx';
import { observer } from 'mobx-react-lite';

// Store class
class TodoStore {
  todos: Todo[] = [];
  filter: 'all' | 'active' | 'completed' = 'all';
  
  constructor() {
    makeObservable(this, {
      todos: observable,
      filter: observable,
      addTodo: action,
      toggleTodo: action,
      removeTodo: action,
      setFilter: action,
      filteredTodos: computed,
      remainingCount: computed,
    });
  }
  
  addTodo(text: string) {
    this.todos.push({
      id: Date.now().toString(),
      text,
      completed: false,
    });
  }
  
  toggleTodo(id: string) {
    const todo = this.todos.find((t) => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }
  
  removeTodo(id: string) {
    this.todos = this.todos.filter((t) => t.id !== id);
  }
  
  setFilter(filter: 'all' | 'active' | 'completed') {
    this.filter = filter;
  }
  
  get filteredTodos() {
    switch (this.filter) {
      case 'active':
        return this.todos.filter((t) => !t.completed);
      case 'completed':
        return this.todos.filter((t) => t.completed);
      default:
        return this.todos;
    }
  }
  
  get remainingCount() {
    return this.todos.filter((t) => !t.completed).length;
  }
}

// Create store instance
const todoStore = new TodoStore();

// Observer component
const TodoList = observer(() => {
  return (
    <ul>
      {todoStore.filteredTodos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => todoStore.toggleTodo(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => todoStore.removeTodo(todo.id)}>×</button>
        </li>
      ))}
    </ul>
  );
});

const RemainingCount = observer(() => {
  return <p>{todoStore.remainingCount} items left</p>;
});
```

## 8.3 MobX with React Context

```tsx
import { createContext, useContext } from 'react';
import { makeAutoObservable } from 'mobx';
import { observer, useLocalObservable } from 'mobx-react-lite';

class RootStore {
  userStore: UserStore;
  todoStore: TodoStore;
  
  constructor() {
    this.userStore = new UserStore(this);
    this.todoStore = new TodoStore(this);
  }
}

class UserStore {
  rootStore: RootStore;
  user: User | null = null;
  
  constructor(rootStore: RootStore) {
    this.rootStore = rootStore;
    makeAutoObservable(this);
  }
  
  setUser(user: User | null) {
    this.user = user;
  }
}

const RootStoreContext = createContext<RootStore | null>(null);

function useRootStore() {
  const store = useContext(RootStoreContext);
  if (!store) throw new Error('Must be within RootStoreProvider');
  return store;
}

function RootStoreProvider({ children }: { children: React.ReactNode }) {
  const store = useLocalObservable(() => new RootStore());
  
  return (
    <RootStoreContext.Provider value={store}>
      {children}
    </RootStoreContext.Provider>
  );
}
```

---

# 9. State Machine with XState

## 9.1 Introduction to XState

```bash
npm install xstate @xstate/react
```

## 9.2 Creating State Machines

```tsx
import { createMachine, assign } from 'xstate';
import { useMachine } from '@xstate/react';

// Toggle machine
const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: {
      on: { TOGGLE: 'active' },
    },
    active: {
      on: { TOGGLE: 'inactive' },
    },
  },
});

// Usage
function Toggle() {
  const [state, send] = useMachine(toggleMachine);
  
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.matches('active') ? 'ON' : 'OFF'}
    </button>
  );
}
```

## 9.3 Complex State Machine

```tsx
import { createMachine, assign } from 'xstate';

interface FetchContext {
  data: any;
  error: string | null;
  retries: number;
}

type FetchEvent =
  | { type: 'FETCH'; url: string }
  | { type: 'RESOLVE'; data: any }
  | { type: 'REJECT'; error: string }
  | { type: 'RETRY' };

const fetchMachine = createMachine<FetchContext, FetchEvent>({
  id: 'fetch',
  initial: 'idle',
  context: {
    data: null,
    error: null,
    retries: 0,
  },
  states: {
    idle: {
      on: {
        FETCH: 'loading',
      },
    },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: {
          target: 'success',
          actions: assign({
            data: (_, event) => event.data,
          }),
        },
        onError: {
          target: 'failure',
          actions: assign({
            error: (_, event) => event.data.message,
          }),
        },
      },
    },
    success: {
      on: {
        FETCH: 'loading',
      },
    },
    failure: {
      on: {
        RETRY: {
          target: 'loading',
          actions: assign({
            retries: (context) => context.retries + 1,
          }),
          cond: (context) => context.retries < 3,
        },
        FETCH: 'loading',
      },
    },
  },
});

// Usage
function DataFetcher({ url }: { url: string }) {
  const [state, send] = useMachine(fetchMachine, {
    services: {
      fetchData: async () => {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Fetch failed');
        return response.json();
      },
    },
  });
  
  return (
    <div>
      {state.matches('idle') && (
        <button onClick={() => send({ type: 'FETCH', url })}>
          Fetch Data
        </button>
      )}
      {state.matches('loading') && <p>Loading...</p>}
      {state.matches('success') && <pre>{JSON.stringify(state.context.data)}</pre>}
      {state.matches('failure') && (
        <div>
          <p>Error: {state.context.error}</p>
          <button onClick={() => send({ type: 'RETRY' })}>
            Retry ({state.context.retries}/3)
          </button>
        </div>
      )}
    </div>
  );
}
```

---

# 10. Server State Management

## 10.1 React Query (TanStack Query)

```bash
npm install @tanstack/react-query
```

```tsx
import { 
  QueryClient, 
  QueryClientProvider, 
  useQuery, 
  useMutation,
  useQueryClient,
} from '@tanstack/react-query';

// Setup
const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Posts />
    </QueryClientProvider>
  );
}

// Queries
function Posts() {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const response = await fetch('/api/posts');
      return response.json();
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 30 * 60 * 1000, // 30 minutes
    refetchOnWindowFocus: false,
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      {data.map((post: Post) => (
        <PostItem key={post.id} post={post} />
      ))}
    </div>
  );
}

// Mutations
function CreatePost() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: async (newPost: Partial<Post>) => {
      const response = await fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(newPost),
      });
      return response.json();
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
    onError: (error) => {
      console.error('Failed to create post:', error);
    },
  });
  
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        const formData = new FormData(e.currentTarget);
        mutation.mutate({
          title: formData.get('title') as string,
          body: formData.get('body') as string,
        });
      }}
    >
      <input name="title" placeholder="Title" />
      <textarea name="body" placeholder="Body" />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create Post'}
      </button>
    </form>
  );
}
```

## 10.2 SWR

```bash
npm install swr
```

```tsx
import useSWR, { useSWRConfig } from 'swr';

const fetcher = (url: string) => fetch(url).then((res) => res.json());

function Profile() {
  const { data, error, isLoading, mutate } = useSWR('/api/user', fetcher, {
    refreshInterval: 3000,
    revalidateOnFocus: true,
    dedupingInterval: 2000,
  });
  
  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={() => mutate()}>Refresh</button>
    </div>
  );
}

// Conditional fetching
function ConditionalFetch({ userId }: { userId: string | null }) {
  const { data } = useSWR(
    userId ? `/api/users/${userId}` : null,
    fetcher
  );
  
  if (!userId) return <div>Select a user</div>;
  if (!data) return <div>Loading...</div>;
  
  return <div>{data.name}</div>;
}

// Optimistic updates
function useTodos() {
  const { data, mutate } = useSWR('/api/todos', fetcher);
  
  const addTodo = async (text: string) => {
    const newTodo = { id: Date.now(), text, completed: false };
    
    // Optimistic update
    mutate([...(data || []), newTodo], false);
    
    // API call
    await fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify(newTodo),
    });
    
    // Revalidate
    mutate();
  };
  
  return { todos: data, addTodo };
}
```

---

# 11. Comparison and When to Use What

## 11.1 Feature Comparison Table

| Feature | Context | Zustand | Redux Toolkit | Jotai | Recoil | MobX |
|---------|---------|---------|---------------|-------|--------|------|
| Bundle Size | 0 (built-in) | ~2KB | ~10KB | ~3KB | ~20KB | ~15KB |
| Learning Curve | Low | Low | Medium | Low | Medium | Medium |
| Boilerplate | Medium | Low | Low | Very Low | Medium | Low |
| TypeScript | Built-in | Excellent | Excellent | Excellent | Good | Good |
| DevTools | No | Yes | Yes | Yes | Yes | Yes |
| Async Support | Manual | Built-in | Thunks | Built-in | Built-in | Built-in |
| SSR Support | Yes | Yes | Yes | Yes | Yes | Yes |
| Atom-based | No | No | No | Yes | Yes | No |
| Computed Values | Manual | Manual | Selectors | Built-in | Selectors | Computed |
| Middleware | No | Yes | Yes | Yes | No | Reactions |

## 11.2 When to Use Each

### Context API
```
✅ Use When:
• Theme/locale switching
• Authentication state
• Small apps
• Rare updates
• Built-in solution sufficient

❌ Avoid When:
• High-frequency updates
• Complex state logic
• Performance critical
• Large state trees
```

### Zustand
```
✅ Use When:
• Medium to large apps
• Need for performance
• Simple API preferred
• TypeScript-first
• Need persistence/devtools

❌ Avoid When:
• Very simple apps (use Context)
• Need time-travel debugging
• Strict flux architecture required
```

### Redux Toolkit
```
✅ Use When:
• Large enterprise apps
• Team familiarity with Redux
• Need RTK Query for API
• Need extensive middleware
• Time-travel debugging required

❌ Avoid When:
• Small to medium apps
• Team unfamiliar with Redux
• Simple state requirements
```

### Jotai
```
✅ Use When:
• Need atomic/granular updates
• React Suspense integration
• Bottom-up state management
• Code splitting state
• Fine-grained re-renders

❌ Avoid When:
• Need global actions
• Team prefers single store
• Need extensive middleware
```

### React Query / TanStack Query
```
✅ Use When:
• Server state management
• API data caching
• Background refetching
• Optimistic updates
• Pagination/infinite scroll

❌ Avoid When:
• Client-only state
• No server interaction
```

## 11.3 Decision Flowchart

```
Start
  │
  ├─ Is it server data? → Yes → React Query / SWR
  │
  ├─ Is it URL state? → Yes → React Router / Next.js Router
  │
  ├─ Is it form state? → Yes → React Hook Form
  │
  ├─ Is it local component state? → Yes → useState / useReducer
  │
  ├─ Shared by few components? → Yes → Lift state + props
  │
  ├─ App-wide, infrequent updates? → Yes → Context API
  │
  ├─ Small-medium app, simple needs? → Yes → Zustand
  │
  ├─ Large app, complex needs? → Yes → Redux Toolkit
  │
  └─ Need atomic/granular control? → Yes → Jotai / Recoil
```

---

# 12. Interview Questions

## 12.1 Basic Questions

### Q1: What is the difference between local and global state?

**Answer:**
- **Local state**: Owned by a single component, managed with `useState` or `useReducer`
- **Global state**: Shared across multiple components, managed with Context, Zustand, Redux, etc.

Local state is appropriate when only one component needs the data. Global state is needed when multiple unrelated components need access to the same data.

---

### Q2: When should you use useReducer instead of useState?

**Answer:**
Use `useReducer` when:
1. State has multiple sub-values
2. Next state depends on previous state
3. State logic is complex
4. Multiple actions modify the same state
5. You want to describe "what happened" vs "what to set"

```tsx
// useState - good for independent values
const [name, setName] = useState('');
const [email, setEmail] = useState('');

// useReducer - good for related values with complex transitions
const [state, dispatch] = useReducer(formReducer, initialFormState);
dispatch({ type: 'SET_FIELD', field: 'name', value: 'John' });
dispatch({ type: 'VALIDATE' });
dispatch({ type: 'SUBMIT' });
```

---

### Q3: Why does Context cause performance issues?

**Answer:**
Context re-renders ALL consumers when the value changes:

```tsx
// Problem: Every component re-renders on any change
const AppContext = createContext({ user: null, theme: 'light', cart: [] });

// Solution 1: Split contexts
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const CartContext = createContext([]);

// Solution 2: Memoize values
const value = useMemo(() => ({ user, theme }), [user, theme]);

// Solution 3: Use state management library
const useStore = create((set) => ({ ... }));
```

---

### Q4: What is prop drilling and how do you solve it?

**Answer:**
Prop drilling is passing props through intermediate components that don't need them:

```tsx
// Problem: App → Header → UserInfo → UserAvatar (all pass user prop)

// Solutions:
// 1. Context API
const UserContext = createContext(null);
// Consume directly in UserAvatar

// 2. Component composition
<App>
  <Header>
    <UserAvatar user={user} /> {/* Pass component instead of props */}
  </Header>
</App>

// 3. State management library
const user = useStore((state) => state.user);
```

---

## 12.2 Zustand Questions

### Q5: How does Zustand handle re-renders?

**Answer:**
Zustand uses selectors to prevent unnecessary re-renders:

```tsx
// Component only re-renders when `count` changes
const count = useStore((state) => state.count);

// Component re-renders on ANY state change
const { count, name, items } = useStore();

// Multiple selectors with shallow comparison
const { user, theme } = useStore(
  (state) => ({ user: state.user, theme: state.theme }),
  shallow
);
```

---

### Q6: How do you persist Zustand state?

**Answer:**
Use the persist middleware:

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useStore = create(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
    }),
    {
      name: 'user-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ user: state.user }), // Only persist user
    }
  )
);
```

---

### Q7: How do you structure Zustand for a large app?

**Answer:**
Use the slices pattern:

```tsx
// Create separate slice creators
const createUserSlice = (set) => ({
  user: null,
  login: async (creds) => { /* ... */ },
});

const createCartSlice = (set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
});

// Combine into one store
const useStore = create((...args) => ({
  ...createUserSlice(...args),
  ...createCartSlice(...args),
}));
```

---

## 12.3 Redux Questions

### Q8: What is the data flow in Redux?

**Answer:**
Redux follows unidirectional data flow:

```
1. User Action → Component dispatches action
2. Action → Describes "what happened"
3. Reducer → Pure function that calculates new state
4. Store → Holds the state tree
5. UI → Subscribes to store, re-renders on changes
```

```tsx
// 1. Dispatch action
dispatch(incrementBy(5));

// 2. Action (created by action creator)
{ type: 'counter/incrementBy', payload: 5 }

// 3. Reducer processes
function counterReducer(state, action) {
  if (action.type === 'counter/incrementBy') {
    return { ...state, value: state.value + action.payload };
  }
  return state;
}

// 4. Store updates
// 5. Connected components re-render
```

---

### Q9: What are Redux middleware?

**Answer:**
Middleware extends Redux with custom functionality between dispatching and reaching the reducer:

```tsx
// Custom logging middleware
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('Next state:', store.getState());
  return result;
};

// Built-in middleware
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(loggerMiddleware),
});
```

---

### Q10: What is RTK Query?

**Answer:**
RTK Query is a data fetching library built into Redux Toolkit:

```tsx
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Post'],
  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => '/posts',
      providesTags: ['Post'],
    }),
    createPost: builder.mutation({
      query: (post) => ({ url: '/posts', method: 'POST', body: post }),
      invalidatesTags: ['Post'],
    }),
  }),
});

// Auto-generated hooks
const { useGetPostsQuery, useCreatePostMutation } = api;
```

Key features:
- Automatic caching
- Request deduplication
- Automatic refetching
- Optimistic updates
- Invalidation via tags

---

## 12.4 Advanced Questions

### Q11: How do you handle form state?

**Answer:**
Multiple approaches based on complexity:

```tsx
// 1. Simple forms - useState
const [name, setName] = useState('');

// 2. Complex forms - useReducer
const [form, dispatch] = useReducer(formReducer, initialState);

// 3. Large forms - React Hook Form
const { register, handleSubmit } = useForm();

// 4. Server sync - Server Actions (Next.js)
async function submitForm(formData: FormData) {
  'use server';
  // Handle submission
}
```

---

### Q12: What is optimistic updates?

**Answer:**
Optimistic updates immediately reflect changes in the UI before server confirmation:

```tsx
// React Query optimistic update
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    
    // Snapshot previous value
    const previous = queryClient.getQueryData(['todos']);
    
    // Optimistically update
    queryClient.setQueryData(['todos'], (old) =>
      old.map((t) => (t.id === newTodo.id ? newTodo : t))
    );
    
    return { previous };
  },
  onError: (err, newTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context.previous);
  },
  onSettled: () => {
    // Refetch after success or error
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

---

### Q13: How would you implement undo/redo?

**Answer:**

```tsx
interface HistoryState<T> {
  past: T[];
  present: T;
  future: T[];
}

function historyReducer<T>(state: HistoryState<T>, action: HistoryAction<T>) {
  switch (action.type) {
    case 'SET':
      return {
        past: [...state.past, state.present],
        present: action.payload,
        future: [],
      };
    case 'UNDO':
      if (state.past.length === 0) return state;
      const previous = state.past[state.past.length - 1];
      return {
        past: state.past.slice(0, -1),
        present: previous,
        future: [state.present, ...state.future],
      };
    case 'REDO':
      if (state.future.length === 0) return state;
      const next = state.future[0];
      return {
        past: [...state.past, state.present],
        present: next,
        future: state.future.slice(1),
      };
    default:
      return state;
  }
}
```

---

### Q14: What is a selector and why use it?

**Answer:**
Selectors are functions that extract specific pieces of state:

```tsx
// Benefits:
// 1. Encapsulation - logic in one place
// 2. Memoization - prevent recalculation
// 3. Composition - combine selectors
// 4. Testability - easy to test pure functions

// Basic selector
const selectUser = (state) => state.user;

// Computed selector
const selectFullName = (state) => 
  `${state.user.firstName} ${state.user.lastName}`;

// Memoized selector (using reselect)
import { createSelector } from '@reduxjs/toolkit';

const selectTodos = (state) => state.todos;
const selectFilter = (state) => state.filter;

const selectFilteredTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    if (filter === 'all') return todos;
    return todos.filter((t) => t.completed === (filter === 'completed'));
  }
);
```

---

### Q15: How do you test state management?

**Answer:**

```tsx
// Testing Zustand store
import { act, renderHook } from '@testing-library/react';
import { useStore } from './store';

beforeEach(() => {
  useStore.setState({ count: 0 });
});

test('increment increases count', () => {
  const { result } = renderHook(() => useStore());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

// Testing Redux slice
import counterReducer, { increment, decrement } from './counterSlice';

test('should increment', () => {
  const previousState = { value: 3 };
  expect(counterReducer(previousState, increment())).toEqual({ value: 4 });
});

// Testing with mock store
import { configureStore } from '@reduxjs/toolkit';
import { render, screen } from '@testing-library/react';
import { Provider } from 'react-redux';

function renderWithStore(ui, preloadedState) {
  const store = configureStore({
    reducer: { counter: counterReducer },
    preloadedState,
  });
  return render(<Provider store={store}>{ui}</Provider>);
}
```

---

# 13. Coding Challenges

## 13.1 Implement useLocalStorage Hook

```tsx
import { useState, useEffect } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  // Get stored value or use initial
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Update localStorage when value changes
  useEffect(() => {
    if (typeof window === 'undefined') return;
    
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setStoredValue] as const;
}

// Usage
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

---

## 13.2 Implement a Shopping Cart Store with Zustand

```tsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
}

interface CartItem extends Product {
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (product: Product) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  // Computed
  totalItems: () => number;
  totalPrice: () => number;
}

const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      
      addItem: (product) => set((state) => {
        const existing = state.items.find((i) => i.id === product.id);
        if (existing) {
          return {
            items: state.items.map((i) =>
              i.id === product.id
                ? { ...i, quantity: i.quantity + 1 }
                : i
            ),
          };
        }
        return {
          items: [...state.items, { ...product, quantity: 1 }],
        };
      }),
      
      removeItem: (productId) => set((state) => ({
        items: state.items.filter((i) => i.id !== productId),
      })),
      
      updateQuantity: (productId, quantity) => set((state) => {
        if (quantity <= 0) {
          return { items: state.items.filter((i) => i.id !== productId) };
        }
        return {
          items: state.items.map((i) =>
            i.id === productId ? { ...i, quantity } : i
          ),
        };
      }),
      
      clearCart: () => set({ items: [] }),
      
      totalItems: () => get().items.reduce((sum, i) => sum + i.quantity, 0),
      
      totalPrice: () => 
        get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
    }),
    { name: 'cart-storage' }
  )
);

export default useCartStore;
```

---

## 13.3 Implement Global Notification System

```tsx
import { create } from 'zustand';

type NotificationType = 'success' | 'error' | 'warning' | 'info';

interface Notification {
  id: string;
  message: string;
  type: NotificationType;
  duration?: number;
}

interface NotificationStore {
  notifications: Notification[];
  add: (message: string, type?: NotificationType, duration?: number) => void;
  remove: (id: string) => void;
  success: (message: string) => void;
  error: (message: string) => void;
  warning: (message: string) => void;
  info: (message: string) => void;
}

const useNotificationStore = create<NotificationStore>((set, get) => ({
  notifications: [],
  
  add: (message, type = 'info', duration = 5000) => {
    const id = Math.random().toString(36).slice(2);
    
    set((state) => ({
      notifications: [...state.notifications, { id, message, type }],
    }));
    
    if (duration > 0) {
      setTimeout(() => {
        get().remove(id);
      }, duration);
    }
  },
  
  remove: (id) => set((state) => ({
    notifications: state.notifications.filter((n) => n.id !== id),
  })),
  
  success: (message) => get().add(message, 'success'),
  error: (message) => get().add(message, 'error'),
  warning: (message) => get().add(message, 'warning'),
  info: (message) => get().add(message, 'info'),
}));

// Notification Component
function NotificationContainer() {
  const { notifications, remove } = useNotificationStore();
  
  return (
    <div className="fixed bottom-4 right-4 space-y-2">
      {notifications.map((notification) => (
        <div
          key={notification.id}
          className={`p-4 rounded shadow-lg ${getColorClass(notification.type)}`}
        >
          {notification.message}
          <button onClick={() => remove(notification.id)}>×</button>
        </div>
      ))}
    </div>
  );
}

// Usage
function SaveButton() {
  const { success, error } = useNotificationStore();
  
  const handleSave = async () => {
    try {
      await saveData();
      success('Data saved successfully!');
    } catch (e) {
      error('Failed to save data');
    }
  };
  
  return <button onClick={handleSave}>Save</button>;
}
```

---

## 13.4 Implement Auth Store with Zustand

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

interface AuthStore {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
  
  login: (email: string, password: string) => Promise<void>;
  register: (email: string, password: string, name: string) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  updateProfile: (data: Partial<User>) => Promise<void>;
  clearError: () => void;
}

const useAuthStore = create<AuthStore>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      isLoading: false,
      error: null,
      
      login: async (email, password) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
          });
          
          if (!response.ok) {
            throw new Error('Invalid credentials');
          }
          
          const { user, token } = await response.json();
          set({ user, token, isAuthenticated: true, isLoading: false });
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Login failed',
            isLoading: false,
          });
        }
      },
      
      register: async (email, password, name) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch('/api/auth/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password, name }),
          });
          
          if (!response.ok) {
            throw new Error('Registration failed');
          }
          
          const { user, token } = await response.json();
          set({ user, token, isAuthenticated: true, isLoading: false });
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Registration failed',
            isLoading: false,
          });
        }
      },
      
      logout: () => {
        set({ user: null, token: null, isAuthenticated: false, error: null });
      },
      
      refreshToken: async () => {
        const { token } = get();
        if (!token) return;
        
        try {
          const response = await fetch('/api/auth/refresh', {
            method: 'POST',
            headers: { Authorization: `Bearer ${token}` },
          });
          
          if (response.ok) {
            const { token: newToken } = await response.json();
            set({ token: newToken });
          } else {
            get().logout();
          }
        } catch {
          get().logout();
        }
      },
      
      updateProfile: async (data) => {
        const { token } = get();
        set({ isLoading: true });
        
        try {
          const response = await fetch('/api/auth/profile', {
            method: 'PATCH',
            headers: {
              'Content-Type': 'application/json',
              Authorization: `Bearer ${token}`,
            },
            body: JSON.stringify(data),
          });
          
          if (response.ok) {
            const user = await response.json();
            set({ user, isLoading: false });
          }
        } catch (error) {
          set({ isLoading: false });
        }
      },
      
      clearError: () => set({ error: null }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ user: state.user, token: state.token }),
    }
  )
);

export default useAuthStore;
```

---

## 13.5 Implement Theme Store with System Preference

```tsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

type Theme = 'light' | 'dark' | 'system';
type ResolvedTheme = 'light' | 'dark';

interface ThemeStore {
  theme: Theme;
  resolvedTheme: ResolvedTheme;
  setTheme: (theme: Theme) => void;
}

const getSystemTheme = (): ResolvedTheme => {
  if (typeof window === 'undefined') return 'light';
  return window.matchMedia('(prefers-color-scheme: dark)').matches
    ? 'dark'
    : 'light';
};

const useThemeStore = create<ThemeStore>()(
  persist(
    (set, get) => ({
      theme: 'system',
      resolvedTheme: getSystemTheme(),
      
      setTheme: (theme) => {
        const resolvedTheme = theme === 'system' ? getSystemTheme() : theme;
        set({ theme, resolvedTheme });
        
        // Apply to DOM
        document.documentElement.classList.remove('light', 'dark');
        document.documentElement.classList.add(resolvedTheme);
      },
    }),
    { name: 'theme-storage' }
  )
);

// Listen for system theme changes
if (typeof window !== 'undefined') {
  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
    const { theme, setTheme } = useThemeStore.getState();
    if (theme === 'system') {
      setTheme('system');
    }
  });
}

export default useThemeStore;
```

---

# 14. Best Practices

## 14.1 General Guidelines

```markdown
1. **Keep state minimal**
   - Derive computed values instead of storing them
   - Don't duplicate data

2. **Normalize nested data**
   - Use IDs for relationships
   - Avoid deeply nested objects

3. **Colocate state**
   - Start with local state
   - Lift only when needed
   - Use global state sparingly

4. **Separate concerns**
   - UI state separate from domain state
   - Server state with React Query/SWR

5. **Use TypeScript**
   - Type your stores
   - Type actions and payloads
   - Use discriminated unions
```

## 14.2 Performance Best Practices

```tsx
// 1. Use selectors
const count = useStore((state) => state.count); // ✅
const store = useStore(); // ❌ Re-renders on any change

// 2. Shallow compare for objects
import { shallow } from 'zustand/shallow';
const { user, theme } = useStore(
  (state) => ({ user: state.user, theme: state.theme }),
  shallow
);

// 3. Memoize expensive computations
const filteredItems = useMemo(
  () => items.filter(expensive),
  [items]
);

// 4. Split large stores
// Instead of one monolithic store, use multiple smaller ones
const useUserStore = create(...);
const useCartStore = create(...);
const useUIStore = create(...);

// 5. Avoid storing serializable data
// Don't store: Functions, DOM elements, Class instances
// Do store: Plain objects, arrays, primitives
```

## 14.3 Testing Guidelines

```tsx
// 1. Reset store between tests
beforeEach(() => {
  useStore.setState(initialState);
});

// 2. Test reducers in isolation
test('increment', () => {
  expect(counterReducer({ count: 0 }, { type: 'INCREMENT' }))
    .toEqual({ count: 1 });
});

// 3. Mock async actions
jest.mock('./api', () => ({
  fetchUser: jest.fn(() => Promise.resolve({ id: 1, name: 'Test' })),
}));

// 4. Test with real store when possible
const { result } = renderHook(() => useStore());
act(() => {
  result.current.increment();
});
expect(result.current.count).toBe(1);
```

---

# 15. Quick Reference

## 15.1 Zustand Cheat Sheet

```tsx
// Create store
const useStore = create((set, get) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set({ count: get().count - 1 }),
}));

// Use with selector
const count = useStore((state) => state.count);

// Middleware
import { devtools, persist } from 'zustand/middleware';
const useStore = create(devtools(persist((set) => ({ ... }), { name: 'storage' })));

// Subscribe outside React
const unsubscribe = useStore.subscribe((state) => console.log(state));

// Get state outside React
const count = useStore.getState().count;

// Set state outside React
useStore.setState({ count: 5 });
```

## 15.2 Redux Toolkit Cheat Sheet

```tsx
// Create slice
const slice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    add: (state, action: PayloadAction<number>) => { state.value += action.payload; },
  },
});

// Create async thunk
const fetchUser = createAsyncThunk('user/fetch', async (id: string) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
});

// Use in component
const value = useAppSelector((state) => state.counter.value);
const dispatch = useAppDispatch();
dispatch(increment());
dispatch(add(5));
```

## 15.3 React Query Cheat Sheet

```tsx
// Query
const { data, isLoading, error, refetch } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 5 * 60 * 1000,
});

// Mutation
const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
});

// Prefetch
queryClient.prefetchQuery({ queryKey: ['post', id], queryFn: () => fetchPost(id) });

// Invalidate
queryClient.invalidateQueries({ queryKey: ['posts'] });
```

---

## 🎯 Study Checklist

### Core Concepts
- [ ] useState vs useReducer use cases
- [ ] Context API and its limitations
- [ ] Props drilling solutions

### Zustand
- [ ] Create basic store
- [ ] Use selectors
- [ ] Apply middleware (persist, devtools)
- [ ] Slices pattern

### Redux Toolkit
- [ ] Create slices
- [ ] Configure store
- [ ] Async thunks
- [ ] RTK Query basics

### Server State
- [ ] React Query setup
- [ ] Query and mutations
- [ ] Cache invalidation
- [ ] Optimistic updates

### Advanced
- [ ] Testing state management
- [ ] Performance optimization
- [ ] State normalization
- [ ] Undo/redo implementation

---

# 16. More Advanced Patterns

## 16.1 State Normalization

```tsx
// Problem: Nested, duplicated data
const state = {
  posts: [
    {
      id: 1,
      title: 'Post 1',
      author: { id: 1, name: 'John' },
      comments: [
        { id: 1, text: 'Great!', author: { id: 2, name: 'Jane' } },
      ],
    },
  ],
};

// Solution: Normalized flat structure
const normalizedState = {
  users: {
    byId: {
      1: { id: 1, name: 'John' },
      2: { id: 2, name: 'Jane' },
    },
    allIds: [1, 2],
  },
  posts: {
    byId: {
      1: { id: 1, title: 'Post 1', authorId: 1, commentIds: [1] },
    },
    allIds: [1],
  },
  comments: {
    byId: {
      1: { id: 1, text: 'Great!', authorId: 2, postId: 1 },
    },
    allIds: [1],
  },
};

// Zustand normalized store
import { create } from 'zustand';

interface Entity<T> {
  byId: Record<string, T>;
  allIds: string[];
}

interface NormalizedStore {
  users: Entity<User>;
  posts: Entity<Post>;
  comments: Entity<Comment>;
  
  // Actions
  addUser: (user: User) => void;
  addPost: (post: Post) => void;
  addComment: (comment: Comment) => void;
  
  // Selectors
  getUser: (id: string) => User | undefined;
  getPost: (id: string) => Post | undefined;
  getPostWithAuthor: (id: string) => (Post & { author: User }) | undefined;
}

const useNormalizedStore = create<NormalizedStore>((set, get) => ({
  users: { byId: {}, allIds: [] },
  posts: { byId: {}, allIds: [] },
  comments: { byId: {}, allIds: [] },
  
  addUser: (user) => set((state) => ({
    users: {
      byId: { ...state.users.byId, [user.id]: user },
      allIds: [...state.users.allIds, user.id],
    },
  })),
  
  addPost: (post) => set((state) => ({
    posts: {
      byId: { ...state.posts.byId, [post.id]: post },
      allIds: [...state.posts.allIds, post.id],
    },
  })),
  
  addComment: (comment) => set((state) => ({
    comments: {
      byId: { ...state.comments.byId, [comment.id]: comment },
      allIds: [...state.comments.allIds, comment.id],
    },
  })),
  
  getUser: (id) => get().users.byId[id],
  getPost: (id) => get().posts.byId[id],
  
  getPostWithAuthor: (id) => {
    const post = get().posts.byId[id];
    if (!post) return undefined;
    const author = get().users.byId[post.authorId];
    return { ...post, author };
  },
}));
```

## 16.2 Middleware Composition

```tsx
import { create, StateCreator } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Define store types
interface BearStore {
  bears: number;
  increase: (by: number) => void;
  reset: () => void;
}

// Create with multiple middleware
const useBearStore = create<BearStore>()(
  devtools(
    persist(
      subscribeWithSelector(
        immer((set) => ({
          bears: 0,
          increase: (by) => set((state) => {
            state.bears += by;
          }),
          reset: () => set({ bears: 0 }),
        }))
      ),
      { name: 'bear-storage' }
    ),
    { name: 'BearStore' }
  )
);

// Subscribe to specific slice
useBearStore.subscribe(
  (state) => state.bears,
  (bears, prevBears) => {
    console.log(`Bears changed from ${prevBears} to ${bears}`);
  }
);
```

## 16.3 Async Queue Pattern

```tsx
import { create } from 'zustand';

interface QueueItem {
  id: string;
  action: () => Promise<void>;
  status: 'pending' | 'processing' | 'completed' | 'failed';
}

interface QueueStore {
  items: QueueItem[];
  isProcessing: boolean;
  addToQueue: (action: () => Promise<void>) => void;
  processQueue: () => Promise<void>;
}

const useQueueStore = create<QueueStore>((set, get) => ({
  items: [],
  isProcessing: false,
  
  addToQueue: (action) => {
    const id = Math.random().toString(36).slice(2);
    set((state) => ({
      items: [...state.items, { id, action, status: 'pending' }],
    }));
    
    // Auto-process if not already processing
    if (!get().isProcessing) {
      get().processQueue();
    }
  },
  
  processQueue: async () => {
    const { items, isProcessing } = get();
    if (isProcessing) return;
    
    set({ isProcessing: true });
    
    for (const item of items.filter(i => i.status === 'pending')) {
      set((state) => ({
        items: state.items.map(i =>
          i.id === item.id ? { ...i, status: 'processing' } : i
        ),
      }));
      
      try {
        await item.action();
        set((state) => ({
          items: state.items.map(i =>
            i.id === item.id ? { ...i, status: 'completed' } : i
          ),
        }));
      } catch (error) {
        set((state) => ({
          items: state.items.map(i =>
            i.id === item.id ? { ...i, status: 'failed' } : i
          ),
        }));
      }
    }
    
    set({ isProcessing: false });
    
    // Clean up completed items
    set((state) => ({
      items: state.items.filter(i => i.status !== 'completed'),
    }));
  },
}));
```

## 16.4 Time-Travel Debugging

```tsx
import { create } from 'zustand';

interface TimeTravel<T> {
  past: T[];
  present: T;
  future: T[];
}

interface TimeTravelStore {
  data: TimeTravel<AppState>;
  undo: () => void;
  redo: () => void;
  canUndo: () => boolean;
  canRedo: () => boolean;
  updateState: (newState: Partial<AppState>) => void;
}

const useTimeTravelStore = create<TimeTravelStore>((set, get) => ({
  data: {
    past: [],
    present: initialState,
    future: [],
  },
  
  updateState: (newState) => set((state) => ({
    data: {
      past: [...state.data.past, state.data.present],
      present: { ...state.data.present, ...newState },
      future: [],
    },
  })),
  
  undo: () => set((state) => {
    const { past, present, future } = state.data;
    if (past.length === 0) return state;
    
    const previous = past[past.length - 1];
    const newPast = past.slice(0, -1);
    
    return {
      data: {
        past: newPast,
        present: previous,
        future: [present, ...future],
      },
    };
  }),
  
  redo: () => set((state) => {
    const { past, present, future } = state.data;
    if (future.length === 0) return state;
    
    const next = future[0];
    const newFuture = future.slice(1);
    
    return {
      data: {
        past: [...past, present],
        present: next,
        future: newFuture,
      },
    };
  }),
  
  canUndo: () => get().data.past.length > 0,
  canRedo: () => get().data.future.length > 0,
}));
```

---

# 17. Real-World Examples

## 17.1 E-Commerce Cart with Zustand

```tsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  stock: number;
}

interface CartItem extends Product {
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  couponCode: string | null;
  discount: number;
  
  // Actions
  addItem: (product: Product, quantity?: number) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  applyCoupon: (code: string) => Promise<boolean>;
  removeCoupon: () => void;
  
  // Computed
  subtotal: () => number;
  discountAmount: () => number;
  total: () => number;
  itemCount: () => number;
  isInCart: (productId: string) => boolean;
  getItemQuantity: (productId: string) => number;
}

export const useCartStore = create<CartStore>()(
  devtools(
    persist(
      immer((set, get) => ({
        items: [],
        couponCode: null,
        discount: 0,
        
        addItem: (product, quantity = 1) => set((state) => {
          const existingItem = state.items.find(i => i.id === product.id);
          
          if (existingItem) {
            const newQuantity = Math.min(
              existingItem.quantity + quantity,
              product.stock
            );
            existingItem.quantity = newQuantity;
          } else {
            state.items.push({
              ...product,
              quantity: Math.min(quantity, product.stock),
            });
          }
        }),
        
        removeItem: (productId) => set((state) => {
          state.items = state.items.filter(i => i.id !== productId);
        }),
        
        updateQuantity: (productId, quantity) => set((state) => {
          const item = state.items.find(i => i.id === productId);
          if (item) {
            if (quantity <= 0) {
              state.items = state.items.filter(i => i.id !== productId);
            } else {
              item.quantity = Math.min(quantity, item.stock);
            }
          }
        }),
        
        clearCart: () => set((state) => {
          state.items = [];
          state.couponCode = null;
          state.discount = 0;
        }),
        
        applyCoupon: async (code) => {
          try {
            const response = await fetch('/api/coupons/validate', {
              method: 'POST',
              body: JSON.stringify({ code }),
            });
            
            if (response.ok) {
              const { discount } = await response.json();
              set({ couponCode: code, discount });
              return true;
            }
            return false;
          } catch {
            return false;
          }
        },
        
        removeCoupon: () => set({ couponCode: null, discount: 0 }),
        
        subtotal: () => 
          get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
        
        discountAmount: () => 
          get().subtotal() * (get().discount / 100),
        
        total: () => 
          get().subtotal() - get().discountAmount(),
        
        itemCount: () => 
          get().items.reduce((sum, i) => sum + i.quantity, 0),
        
        isInCart: (productId) => 
          get().items.some(i => i.id === productId),
        
        getItemQuantity: (productId) => 
          get().items.find(i => i.id === productId)?.quantity ?? 0,
      })),
      { name: 'cart-storage' }
    ),
    { name: 'CartStore' }
  )
);

// Components
function CartIcon() {
  const itemCount = useCartStore((state) => state.itemCount());
  
  return (
    <button className="relative">
      🛒
      {itemCount > 0 && (
        <span className="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 text-xs">
          {itemCount}
        </span>
      )}
    </button>
  );
}

function AddToCartButton({ product }: { product: Product }) {
  const { addItem, isInCart, getItemQuantity } = useCartStore();
  const inCart = isInCart(product.id);
  const quantity = getItemQuantity(product.id);
  
  return (
    <button
      onClick={() => addItem(product)}
      disabled={quantity >= product.stock}
      className={inCart ? 'bg-green-500' : 'bg-blue-500'}
    >
      {inCart ? `In Cart (${quantity})` : 'Add to Cart'}
    </button>
  );
}

function CartSummary() {
  const { items, subtotal, discount, discountAmount, total, couponCode } = useCartStore();
  
  return (
    <div className="p-4 border rounded">
      <h2>Order Summary</h2>
      
      {items.map(item => (
        <div key={item.id} className="flex justify-between">
          <span>{item.name} × {item.quantity}</span>
          <span>${(item.price * item.quantity).toFixed(2)}</span>
        </div>
      ))}
      
      <hr />
      
      <div className="flex justify-between">
        <span>Subtotal</span>
        <span>${subtotal().toFixed(2)}</span>
      </div>
      
      {discount > 0 && (
        <div className="flex justify-between text-green-600">
          <span>Discount ({couponCode})</span>
          <span>-${discountAmount().toFixed(2)}</span>
        </div>
      )}
      
      <div className="flex justify-between font-bold text-lg">
        <span>Total</span>
        <span>${total().toFixed(2)}</span>
      </div>
    </div>
  );
}
```

## 17.2 Dashboard with Multiple Stores

```tsx
// stores/userStore.ts
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  isAuthenticated: false,
  
  login: async (credentials) => {
    const user = await authService.login(credentials);
    set({ user, isAuthenticated: true });
  },
  
  logout: () => {
    authService.logout();
    set({ user: null, isAuthenticated: false });
  },
}));

// stores/dashboardStore.ts
interface DashboardStore {
  metrics: DashboardMetrics | null;
  loading: boolean;
  error: string | null;
  dateRange: DateRange;
  
  fetchMetrics: () => Promise<void>;
  setDateRange: (range: DateRange) => void;
}

export const useDashboardStore = create<DashboardStore>((set, get) => ({
  metrics: null,
  loading: false,
  error: null,
  dateRange: { start: subDays(new Date(), 30), end: new Date() },
  
  fetchMetrics: async () => {
    set({ loading: true, error: null });
    try {
      const { dateRange } = get();
      const metrics = await dashboardService.getMetrics(dateRange);
      set({ metrics, loading: false });
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Failed to load', 
        loading: false 
      });
    }
  },
  
  setDateRange: (range) => {
    set({ dateRange: range });
    get().fetchMetrics();
  },
}));

// stores/notificationStore.ts
interface NotificationStore {
  notifications: Notification[];
  unreadCount: number;
  
  fetchNotifications: () => Promise<void>;
  markAsRead: (id: string) => void;
  markAllAsRead: () => void;
}

export const useNotificationStore = create<NotificationStore>((set, get) => ({
  notifications: [],
  unreadCount: 0,
  
  fetchNotifications: async () => {
    const notifications = await notificationService.getAll();
    const unreadCount = notifications.filter(n => !n.read).length;
    set({ notifications, unreadCount });
  },
  
  markAsRead: (id) => set((state) => ({
    notifications: state.notifications.map(n =>
      n.id === id ? { ...n, read: true } : n
    ),
    unreadCount: state.unreadCount - 1,
  })),
  
  markAllAsRead: () => set((state) => ({
    notifications: state.notifications.map(n => ({ ...n, read: true })),
    unreadCount: 0,
  })),
}));

// stores/settingsStore.ts
interface SettingsStore {
  theme: 'light' | 'dark' | 'system';
  language: string;
  sidebarCollapsed: boolean;
  
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  setLanguage: (language: string) => void;
  toggleSidebar: () => void;
}

export const useSettingsStore = create<SettingsStore>()(
  persist(
    (set) => ({
      theme: 'system',
      language: 'en',
      sidebarCollapsed: false,
      
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleSidebar: () => set((s) => ({ sidebarCollapsed: !s.sidebarCollapsed })),
    }),
    { name: 'settings-storage' }
  )
);

// Dashboard component using multiple stores
function Dashboard() {
  const user = useUserStore((s) => s.user);
  const { metrics, loading, dateRange, setDateRange } = useDashboardStore();
  const unreadCount = useNotificationStore((s) => s.unreadCount);
  const { theme, sidebarCollapsed, toggleSidebar } = useSettingsStore();
  
  useEffect(() => {
    useDashboardStore.getState().fetchMetrics();
    useNotificationStore.getState().fetchNotifications();
  }, []);
  
  return (
    <div className={theme}>
      <Header 
        user={user} 
        unreadCount={unreadCount}
        onToggleSidebar={toggleSidebar}
      />
      <Sidebar collapsed={sidebarCollapsed} />
      <main>
        <DateRangePicker value={dateRange} onChange={setDateRange} />
        {loading ? <Spinner /> : <MetricsGrid metrics={metrics} />}
      </main>
    </div>
  );
}
```

---

# 18. Output-Based Questions

## Question 1: What's the output?

```tsx
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const count = useStore((state) => state.count);
  console.log('Counter rendered');
  return <p>{count}</p>;
}

function Button() {
  const increment = useStore((state) => state.increment);
  console.log('Button rendered');
  return <button onClick={increment}>+</button>;
}

function App() {
  return (
    <>
      <Counter />
      <Button />
    </>
  );
}

// After clicking the button once, what logs appear?
```

**Answer:**
```
Counter rendered
```

Only the `Counter` component re-renders because:
- `Counter` subscribes to `state.count` which changes
- `Button` subscribes to `state.increment` which is a stable reference
- Zustand only re-renders components whose selected state changes

---

## Question 2: What's the bug?

```tsx
const useStore = create((set) => ({
  items: [],
  addItem: (item) => set({ items: [...items, item] }), // Bug here
}));
```

**Answer:**
`items` is undefined in the closure. It should reference `state.items`:

```tsx
// Fix:
addItem: (item) => set((state) => ({ items: [...state.items, item] })),
```

---

## Question 3: Why doesn't this component re-render?

```tsx
const useStore = create((set) => ({
  user: { name: 'John', age: 25 },
  updateAge: (age) => set((state) => {
    state.user.age = age; // Mutation!
    return { user: state.user };
  }),
}));

function Age() {
  const age = useStore((state) => state.user.age);
  return <p>Age: {age}</p>;
}
```

**Answer:**
The code mutates `state.user.age` but returns the same object reference. Zustand uses shallow comparison, so it doesn't detect the change.

```tsx
// Fix 1: Create new object
updateAge: (age) => set((state) => ({ 
  user: { ...state.user, age } 
})),

// Fix 2: Use immer middleware
import { immer } from 'zustand/middleware/immer';

const useStore = create(immer((set) => ({
  user: { name: 'John', age: 25 },
  updateAge: (age) => set((state) => {
    state.user.age = age; // Now works with immer
  }),
})));
```

---

## Question 4: What's wrong with this Context implementation?

```tsx
const ThemeContext = createContext({ theme: 'light', setTheme: () => {} });

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

**Answer:**
A new object is created on every render, causing all consumers to re-render even if `theme` hasn't changed.

```tsx
// Fix: Memoize the value
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

# 19. More Interview Questions

## Q16: What is the flux architecture?

**Answer:**
Flux is a unidirectional data flow pattern:

```
┌─────────────────────────────────────────────┐
│                                             │
│  Action → Dispatcher → Store → View        │
│     ↑                           │           │
│     └───────────────────────────┘           │
│           (user interaction)                │
│                                             │
└─────────────────────────────────────────────┘
```

Key principles:
1. **Single source of truth** - state in stores
2. **State is read-only** - changes only via actions
3. **Changes via pure functions** - reducers

---

## Q17: What is the difference between controlled and uncontrolled state?

**Answer:**

```tsx
// Controlled: React manages state
function ControlledInput() {
  const [value, setValue] = useState('');
  return (
    <input 
      value={value} 
      onChange={(e) => setValue(e.target.value)} 
    />
  );
}

// Uncontrolled: DOM manages state
function UncontrolledInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };
  
  return <input ref={inputRef} defaultValue="" />;
}
```

---

## Q18: How would you implement a global modal system?

**Answer:**

```tsx
import { create } from 'zustand';

type ModalType = 'confirm' | 'alert' | 'custom';

interface ModalState {
  isOpen: boolean;
  type: ModalType | null;
  title: string;
  content: React.ReactNode;
  onConfirm?: () => void;
  onCancel?: () => void;
}

interface ModalStore extends ModalState {
  openModal: (config: Omit<ModalState, 'isOpen'>) => void;
  closeModal: () => void;
  confirm: (title: string, message: string) => Promise<boolean>;
  alert: (title: string, message: string) => Promise<void>;
}

export const useModalStore = create<ModalStore>((set) => ({
  isOpen: false,
  type: null,
  title: '',
  content: null,
  onConfirm: undefined,
  onCancel: undefined,
  
  openModal: (config) => set({ isOpen: true, ...config }),
  closeModal: () => set({ isOpen: false }),
  
  confirm: (title, message) => new Promise((resolve) => {
    set({
      isOpen: true,
      type: 'confirm',
      title,
      content: message,
      onConfirm: () => {
        set({ isOpen: false });
        resolve(true);
      },
      onCancel: () => {
        set({ isOpen: false });
        resolve(false);
      },
    });
  }),
  
  alert: (title, message) => new Promise((resolve) => {
    set({
      isOpen: true,
      type: 'alert',
      title,
      content: message,
      onConfirm: () => {
        set({ isOpen: false });
        resolve();
      },
    });
  }),
}));

// Modal Component
function GlobalModal() {
  const { isOpen, type, title, content, onConfirm, onCancel, closeModal } = useModalStore();
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div className="modal">
        <h2>{title}</h2>
        <div>{content}</div>
        <div className="modal-actions">
          {type === 'confirm' && (
            <button onClick={onCancel}>Cancel</button>
          )}
          <button onClick={onConfirm}>OK</button>
        </div>
      </div>
    </div>
  );
}

// Usage
function DeleteButton({ itemId }: { itemId: string }) {
  const { confirm } = useModalStore();
  
  const handleDelete = async () => {
    const confirmed = await confirm(
      'Delete Item',
      'Are you sure you want to delete this item?'
    );
    
    if (confirmed) {
      await deleteItem(itemId);
    }
  };
  
  return <button onClick={handleDelete}>Delete</button>;
}
```

---

## Q19: How do you handle hydration mismatches with persisted state?

**Answer:**

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { useState, useEffect } from 'react';

const useStore = create(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage',
      storage: createJSONStorage(() => localStorage),
    }
  )
);

// Custom hook to handle hydration
function useHydratedStore<T, U>(
  useStore: (selector: (state: T) => U) => U,
  selector: (state: T) => U
): U | undefined {
  const [hydrated, setHydrated] = useState(false);
  const storeValue = useStore(selector);
  
  useEffect(() => {
    setHydrated(true);
  }, []);
  
  return hydrated ? storeValue : undefined;
}

// Usage - prevents SSR mismatch
function ThemeToggle() {
  const theme = useHydratedStore(useStore, (s) => s.theme);
  const setTheme = useStore((s) => s.setTheme);
  
  // Show skeleton during hydration
  if (theme === undefined) {
    return <div className="skeleton" />;
  }
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}
```

---

## Q20: What patterns help with React Server Components?

**Answer:**

```tsx
// 1. Server Component for data fetching
async function ProductsPage() {
  const products = await getProducts(); // Server-side fetch
  return <ProductList initialProducts={products} />;
}

// 2. Client Component wrapper for interactivity
'use client';

function ProductList({ initialProducts }: { initialProducts: Product[] }) {
  // Initialize store with server data
  const store = useProductStore();
  
  useEffect(() => {
    store.setProducts(initialProducts);
  }, [initialProducts]);
  
  return (
    <div>
      {store.products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}

// 3. Hydration-safe store
const useProductStore = create<ProductStore>((set) => ({
  products: [],
  setProducts: (products) => set({ products }),
  addProduct: (product) => set((s) => ({ 
    products: [...s.products, product] 
  })),
}));

// 4. For RSC, prefer passing data as props over global state
// Server components can't use hooks, so data flows down as props
```

---

# 20. Zustand Utilities and Helpers

## 20.1 Create Bounded Store

```tsx
import { createStore, useStore } from 'zustand';
import { createContext, useContext, useRef } from 'react';

// Create store type
interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
}

// Create store factory
const createCounterStore = (initialCount = 0) =>
  createStore<CounterStore>((set) => ({
    count: initialCount,
    increment: () => set((s) => ({ count: s.count + 1 })),
    decrement: () => set((s) => ({ count: s.count - 1 })),
  }));

// Create context
type CounterStoreType = ReturnType<typeof createCounterStore>;
const CounterContext = createContext<CounterStoreType | null>(null);

// Provider
function CounterProvider({
  children,
  initialCount = 0,
}: {
  children: React.ReactNode;
  initialCount?: number;
}) {
  const storeRef = useRef<CounterStoreType>();
  if (!storeRef.current) {
    storeRef.current = createCounterStore(initialCount);
  }
  return (
    <CounterContext.Provider value={storeRef.current}>
      {children}
    </CounterContext.Provider>
  );
}

// Bounded hook
function useCounterStore<T>(selector: (state: CounterStore) => T): T {
  const store = useContext(CounterContext);
  if (!store) throw new Error('Missing CounterProvider');
  return useStore(store, selector);
}

// Usage - each provider has its own store instance
function App() {
  return (
    <>
      <CounterProvider initialCount={0}>
        <Counter /> {/* Uses its own store */}
      </CounterProvider>
      <CounterProvider initialCount={100}>
        <Counter /> {/* Uses different store */}
      </CounterProvider>
    </>
  );
}
```

## 20.2 Computed Values Helper

```tsx
import { create, StoreApi, UseBoundStore } from 'zustand';

// Helper to add computed values
function withComputed<T extends object, C extends object>(
  store: UseBoundStore<StoreApi<T>>,
  computedFn: (state: T) => C
): UseBoundStore<StoreApi<T & C>> {
  return ((selector) => {
    const state = store((s) => s);
    const computed = computedFn(state);
    const combined = { ...state, ...computed };
    return selector ? selector(combined) : combined;
  }) as UseBoundStore<StoreApi<T & C>>;
}

// Usage
const baseStore = create<{ items: number[] }>((set) => ({
  items: [1, 2, 3, 4, 5],
}));

const useStore = withComputed(baseStore, (state) => ({
  total: state.items.reduce((a, b) => a + b, 0),
  count: state.items.length,
  average: state.items.length > 0 
    ? state.items.reduce((a, b) => a + b, 0) / state.items.length 
    : 0,
}));

function Stats() {
  const { total, count, average } = useStore();
  return (
    <div>
      <p>Total: {total}</p>
      <p>Count: {count}</p>
      <p>Average: {average}</p>
    </div>
  );
}
```

## 20.3 Action Logger Middleware

```tsx
import { StateCreator, StoreMutatorIdentifier } from 'zustand';

type Logger = <
  T extends object,
  Mps extends [StoreMutatorIdentifier, unknown][] = [],
  Mcs extends [StoreMutatorIdentifier, unknown][] = []
>(
  f: StateCreator<T, Mps, Mcs>,
  name?: string
) => StateCreator<T, Mps, Mcs>;

const logger: Logger = (f, name) => (set, get, store) => {
  const loggedSet: typeof set = (...args) => {
    const prevState = get();
    set(...args);
    const nextState = get();
    
    console.group(`[${name || 'store'}] State Update`);
    console.log('Previous:', prevState);
    console.log('Next:', nextState);
    console.log('Changed:', getDiff(prevState, nextState));
    console.groupEnd();
  };
  
  return f(loggedSet, get, store);
};

function getDiff(prev: object, next: object) {
  const diff: Record<string, { from: any; to: any }> = {};
  
  for (const key of new Set([...Object.keys(prev), ...Object.keys(next)])) {
    if (prev[key] !== next[key]) {
      diff[key] = { from: prev[key], to: next[key] };
    }
  }
  
  return diff;
}

// Usage
const useStore = create(
  logger(
    (set) => ({
      count: 0,
      increment: () => set((s) => ({ count: s.count + 1 })),
    }),
    'CounterStore'
  )
);
```

---

# 21. Migration Guides

## 21.1 Context API to Zustand

```tsx
// Before: Context API
const UserContext = createContext<UserContextType | null>(null);

function UserProvider({ children }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  
  const login = async (credentials: Credentials) => {
    setLoading(true);
    try {
      const user = await authService.login(credentials);
      setUser(user);
    } finally {
      setLoading(false);
    }
  };
  
  const logout = () => setUser(null);
  
  const value = useMemo(
    () => ({ user, loading, login, logout }),
    [user, loading]
  );
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// After: Zustand
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  loading: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

const useUserStore = create<UserStore>((set) => ({
  user: null,
  loading: false,
  
  login: async (credentials) => {
    set({ loading: true });
    try {
      const user = await authService.login(credentials);
      set({ user, loading: false });
    } catch {
      set({ loading: false });
    }
  },
  
  logout: () => set({ user: null }),
}));

// No provider needed!
// Just use the hook directly in components
```

## 21.2 Redux to Zustand

```tsx
// Before: Redux
// actions.ts
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';

const increment = () => ({ type: INCREMENT });
const decrement = () => ({ type: DECREMENT });

// reducer.ts
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case INCREMENT:
      return { count: state.count + 1 };
    case DECREMENT:
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// store.ts
const store = createStore(counterReducer);

// Component
function Counter() {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}

// After: Zustand
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
  decrement: () => set((s) => ({ count: s.count - 1 })),
}));

function Counter() {
  const { count, increment, decrement } = useCounterStore();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

---

# 22. Final Summary

## Key Takeaways

1. **Start simple** - useState for local, lift state when needed
2. **Context for rare updates** - theme, locale, auth status
3. **Zustand for most apps** - simple API, great performance
4. **Redux for enterprise** - when you need the ecosystem
5. **React Query for server state** - don't mix with client state
6. **TypeScript everywhere** - type your stores

## Decision Matrix

| App Size | State Complexity | Recommendation |
|----------|------------------|----------------|
| Small | Simple | useState + Context |
| Small | Complex | useReducer + Context |
| Medium | Simple | Zustand |
| Medium | Complex | Zustand + React Query |
| Large | Any | Redux Toolkit + RTK Query |
| Any | Server-heavy | React Query / SWR |

## Performance Tips Summary

1. Use selectors to prevent re-renders
2. Split large stores into slices
3. Memoize computed values
4. Use shallow comparison for objects
5. Separate UI state from domain state
6. Use server state libraries for API data

---

# 23. More Coding Challenges

## 23.1 Implement Debounced State

```tsx
import { create } from 'zustand';
import { debounce } from 'lodash-es';

interface SearchStore {
  query: string;
  debouncedQuery: string;
  results: SearchResult[];
  loading: boolean;
  setQuery: (query: string) => void;
  search: () => Promise<void>;
}

const useSearchStore = create<SearchStore>((set, get) => {
  // Debounced search function
  const debouncedSearch = debounce(async (query: string) => {
    if (!query.trim()) {
      set({ results: [], loading: false, debouncedQuery: '' });
      return;
    }
    
    set({ loading: true, debouncedQuery: query });
    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
      const results = await response.json();
      set({ results, loading: false });
    } catch (error) {
      set({ results: [], loading: false });
    }
  }, 300);
  
  return {
    query: '',
    debouncedQuery: '',
    results: [],
    loading: false,
    
    setQuery: (query) => {
      set({ query });
      debouncedSearch(query);
    },
    
    search: async () => {
      const { query } = get();
      await debouncedSearch(query);
    },
  };
});

// Usage
function SearchBox() {
  const { query, setQuery, loading, results } = useSearchStore();
  
  return (
    <div>
      <input
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {loading && <Spinner />}
      <ul>
        {results.map((result) => (
          <li key={result.id}>{result.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 23.2 Implement Pagination Store

```tsx
import { create } from 'zustand';

interface PaginationStore<T> {
  items: T[];
  page: number;
  pageSize: number;
  totalItems: number;
  totalPages: number;
  loading: boolean;
  error: string | null;
  
  setPage: (page: number) => void;
  setPageSize: (size: number) => void;
  fetchPage: () => Promise<void>;
  nextPage: () => void;
  prevPage: () => void;
  
  // Computed
  hasNextPage: () => boolean;
  hasPrevPage: () => boolean;
  pageRange: () => number[];
}

function createPaginationStore<T>(fetchFn: (page: number, pageSize: number) => Promise<{ items: T[]; total: number }>) {
  return create<PaginationStore<T>>((set, get) => ({
    items: [],
    page: 1,
    pageSize: 10,
    totalItems: 0,
    totalPages: 0,
    loading: false,
    error: null,
    
    setPage: (page) => {
      set({ page });
      get().fetchPage();
    },
    
    setPageSize: (pageSize) => {
      set({ pageSize, page: 1 });
      get().fetchPage();
    },
    
    fetchPage: async () => {
      const { page, pageSize } = get();
      set({ loading: true, error: null });
      
      try {
        const { items, total } = await fetchFn(page, pageSize);
        set({
          items,
          totalItems: total,
          totalPages: Math.ceil(total / pageSize),
          loading: false,
        });
      } catch (error) {
        set({
          error: error instanceof Error ? error.message : 'Failed to fetch',
          loading: false,
        });
      }
    },
    
    nextPage: () => {
      const { page, totalPages } = get();
      if (page < totalPages) {
        get().setPage(page + 1);
      }
    },
    
    prevPage: () => {
      const { page } = get();
      if (page > 1) {
        get().setPage(page - 1);
      }
    },
    
    hasNextPage: () => get().page < get().totalPages,
    hasPrevPage: () => get().page > 1,
    
    pageRange: () => {
      const { page, totalPages } = get();
      const range: number[] = [];
      const start = Math.max(1, page - 2);
      const end = Math.min(totalPages, page + 2);
      
      for (let i = start; i <= end; i++) {
        range.push(i);
      }
      
      return range;
    },
  }));
}

// Usage
const useProductPagination = createPaginationStore<Product>(
  async (page, pageSize) => {
    const response = await fetch(`/api/products?page=${page}&limit=${pageSize}`);
    const data = await response.json();
    return { items: data.products, total: data.total };
  }
);

function ProductList() {
  const {
    items,
    page,
    loading,
    nextPage,
    prevPage,
    hasNextPage,
    hasPrevPage,
    pageRange,
    fetchPage,
  } = useProductPagination();
  
  useEffect(() => {
    fetchPage();
  }, []);
  
  return (
    <div>
      {loading ? (
        <Spinner />
      ) : (
        <ul>
          {items.map((product) => (
            <li key={product.id}>{product.name}</li>
          ))}
        </ul>
      )}
      
      <div className="pagination">
        <button onClick={prevPage} disabled={!hasPrevPage()}>
          Previous
        </button>
        
        {pageRange().map((p) => (
          <button
            key={p}
            onClick={() => useProductPagination.getState().setPage(p)}
            className={p === page ? 'active' : ''}
          >
            {p}
          </button>
        ))}
        
        <button onClick={nextPage} disabled={!hasNextPage()}>
          Next
        </button>
      </div>
    </div>
  );
}
```

## 23.3 Implement Form Store with Validation

```tsx
import { create } from 'zustand';
import { z } from 'zod';

// Validation schema
const userSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
});

type UserFormData = z.infer<typeof userSchema>;

interface FormStore {
  values: UserFormData;
  errors: Partial<Record<keyof UserFormData, string>>;
  touched: Partial<Record<keyof UserFormData, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
  
  setValue: <K extends keyof UserFormData>(field: K, value: UserFormData[K]) => void;
  setTouched: (field: keyof UserFormData) => void;
  validate: () => boolean;
  validateField: (field: keyof UserFormData) => void;
  submit: (onSubmit: (data: UserFormData) => Promise<void>) => Promise<void>;
  reset: () => void;
}

const initialValues: UserFormData = {
  name: '',
  email: '',
  password: '',
  confirmPassword: '',
};

const useFormStore = create<FormStore>((set, get) => ({
  values: { ...initialValues },
  errors: {},
  touched: {},
  isSubmitting: false,
  isValid: false,
  
  setValue: (field, value) => {
    set((state) => ({
      values: { ...state.values, [field]: value },
    }));
    
    // Validate on change if field was touched
    if (get().touched[field]) {
      get().validateField(field);
    }
  },
  
  setTouched: (field) => {
    set((state) => ({
      touched: { ...state.touched, [field]: true },
    }));
    get().validateField(field);
  },
  
  validateField: (field) => {
    const result = userSchema.safeParse(get().values);
    
    if (!result.success) {
      const fieldError = result.error.errors.find((e) => e.path[0] === field);
      set((state) => ({
        errors: {
          ...state.errors,
          [field]: fieldError?.message || undefined,
        },
      }));
    } else {
      set((state) => ({
        errors: { ...state.errors, [field]: undefined },
      }));
    }
    
    get().validate();
  },
  
  validate: () => {
    const result = userSchema.safeParse(get().values);
    
    if (!result.success) {
      const errors: Partial<Record<keyof UserFormData, string>> = {};
      result.error.errors.forEach((error) => {
        const field = error.path[0] as keyof UserFormData;
        if (!errors[field]) {
          errors[field] = error.message;
        }
      });
      set({ errors, isValid: false });
      return false;
    }
    
    set({ errors: {}, isValid: true });
    return true;
  },
  
  submit: async (onSubmit) => {
    // Touch all fields
    const allTouched: Partial<Record<keyof UserFormData, boolean>> = {};
    Object.keys(get().values).forEach((key) => {
      allTouched[key as keyof UserFormData] = true;
    });
    set({ touched: allTouched });
    
    if (!get().validate()) {
      return;
    }
    
    set({ isSubmitting: true });
    try {
      await onSubmit(get().values);
      get().reset();
    } finally {
      set({ isSubmitting: false });
    }
  },
  
  reset: () => {
    set({
      values: { ...initialValues },
      errors: {},
      touched: {},
      isSubmitting: false,
      isValid: false,
    });
  },
}));

// Form Component
function RegistrationForm() {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    setValue,
    setTouched,
    submit,
  } = useFormStore();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await submit(async (data) => {
      await registerUser(data);
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          value={values.name}
          onChange={(e) => setValue('name', e.target.value)}
          onBlur={() => setTouched('name')}
          placeholder="Name"
        />
        {touched.name && errors.name && (
          <span className="error">{errors.name}</span>
        )}
      </div>
      
      <div>
        <input
          type="email"
          value={values.email}
          onChange={(e) => setValue('email', e.target.value)}
          onBlur={() => setTouched('email')}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <input
          type="password"
          value={values.password}
          onChange={(e) => setValue('password', e.target.value)}
          onBlur={() => setTouched('password')}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>
      
      <div>
        <input
          type="password"
          value={values.confirmPassword}
          onChange={(e) => setValue('confirmPassword', e.target.value)}
          onBlur={() => setTouched('confirmPassword')}
          placeholder="Confirm Password"
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error">{errors.confirmPassword}</span>
        )}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

## 23.4 Implement WebSocket Connection Store

```tsx
import { create } from 'zustand';

type ConnectionStatus = 'disconnected' | 'connecting' | 'connected' | 'error';

interface Message {
  id: string;
  type: string;
  payload: any;
  timestamp: Date;
}

interface WebSocketStore {
  socket: WebSocket | null;
  status: ConnectionStatus;
  messages: Message[];
  error: string | null;
  
  connect: (url: string) => void;
  disconnect: () => void;
  send: (type: string, payload: any) => void;
  clearMessages: () => void;
}

const useWebSocketStore = create<WebSocketStore>((set, get) => ({
  socket: null,
  status: 'disconnected',
  messages: [],
  error: null,
  
  connect: (url) => {
    const { socket } = get();
    
    // Close existing connection
    if (socket) {
      socket.close();
    }
    
    set({ status: 'connecting', error: null });
    
    const ws = new WebSocket(url);
    
    ws.onopen = () => {
      set({ socket: ws, status: 'connected' });
    };
    
    ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        const message: Message = {
          id: Math.random().toString(36).slice(2),
          type: data.type,
          payload: data.payload,
          timestamp: new Date(),
        };
        
        set((state) => ({
          messages: [...state.messages, message],
        }));
      } catch (error) {
        console.error('Failed to parse message:', error);
      }
    };
    
    ws.onerror = () => {
      set({ status: 'error', error: 'Connection failed' });
    };
    
    ws.onclose = () => {
      set({ socket: null, status: 'disconnected' });
    };
    
    set({ socket: ws });
  },
  
  disconnect: () => {
    const { socket } = get();
    if (socket) {
      socket.close();
      set({ socket: null, status: 'disconnected' });
    }
  },
  
  send: (type, payload) => {
    const { socket, status } = get();
    
    if (socket && status === 'connected') {
      socket.send(JSON.stringify({ type, payload }));
    } else {
      console.error('WebSocket is not connected');
    }
  },
  
  clearMessages: () => {
    set({ messages: [] });
  },
}));

// Auto-reconnect hook
function useWebSocketConnection(url: string) {
  const { connect, disconnect, status } = useWebSocketStore();
  
  useEffect(() => {
    connect(url);
    
    return () => {
      disconnect();
    };
  }, [url]);
  
  // Auto reconnect on disconnect
  useEffect(() => {
    if (status === 'disconnected') {
      const timer = setTimeout(() => {
        connect(url);
      }, 3000);
      
      return () => clearTimeout(timer);
    }
  }, [status, url]);
  
  return status;
}
```

## 23.5 Implement Filters and Sorting Store

```tsx
import { create } from 'zustand';

type SortOrder = 'asc' | 'desc';

interface FilterState {
  search: string;
  category: string | null;
  priceRange: [number, number];
  inStock: boolean;
  sortBy: string;
  sortOrder: SortOrder;
}

interface FilterStore extends FilterState {
  setSearch: (search: string) => void;
  setCategory: (category: string | null) => void;
  setPriceRange: (range: [number, number]) => void;
  setInStock: (inStock: boolean) => void;
  setSortBy: (field: string) => void;
  toggleSortOrder: () => void;
  resetFilters: () => void;
  
  // Apply to data
  applyFilters: <T extends Record<string, any>>(
    items: T[],
    config: {
      searchFields: (keyof T)[];
      categoryField: keyof T;
      priceField: keyof T;
      stockField: keyof T;
    }
  ) => T[];
}

const initialFilters: FilterState = {
  search: '',
  category: null,
  priceRange: [0, Infinity],
  inStock: false,
  sortBy: 'name',
  sortOrder: 'asc',
};

const useFilterStore = create<FilterStore>((set, get) => ({
  ...initialFilters,
  
  setSearch: (search) => set({ search }),
  setCategory: (category) => set({ category }),
  setPriceRange: (priceRange) => set({ priceRange }),
  setInStock: (inStock) => set({ inStock }),
  setSortBy: (sortBy) => set({ sortBy }),
  toggleSortOrder: () => set((state) => ({
    sortOrder: state.sortOrder === 'asc' ? 'desc' : 'asc',
  })),
  resetFilters: () => set(initialFilters),
  
  applyFilters: (items, config) => {
    const { search, category, priceRange, inStock, sortBy, sortOrder } = get();
    
    let filtered = [...items];
    
    // Search filter
    if (search.trim()) {
      const searchLower = search.toLowerCase();
      filtered = filtered.filter((item) =>
        config.searchFields.some((field) =>
          String(item[field]).toLowerCase().includes(searchLower)
        )
      );
    }
    
    // Category filter
    if (category) {
      filtered = filtered.filter(
        (item) => item[config.categoryField] === category
      );
    }
    
    // Price range filter
    filtered = filtered.filter((item) => {
      const price = item[config.priceField] as number;
      return price >= priceRange[0] && price <= priceRange[1];
    });
    
    // In stock filter
    if (inStock) {
      filtered = filtered.filter((item) => item[config.stockField] as boolean);
    }
    
    // Sorting
    filtered.sort((a, b) => {
      const aVal = a[sortBy as keyof typeof a];
      const bVal = b[sortBy as keyof typeof b];
      
      if (typeof aVal === 'string' && typeof bVal === 'string') {
        return sortOrder === 'asc'
          ? aVal.localeCompare(bVal)
          : bVal.localeCompare(aVal);
      }
      
      if (typeof aVal === 'number' && typeof bVal === 'number') {
        return sortOrder === 'asc' ? aVal - bVal : bVal - aVal;
      }
      
      return 0;
    });
    
    return filtered;
  },
}));

// Usage
function ProductFilters() {
  const {
    search,
    setSearch,
    category,
    setCategory,
    priceRange,
    setPriceRange,
    inStock,
    setInStock,
    sortBy,
    setSortBy,
    sortOrder,
    toggleSortOrder,
    resetFilters,
  } = useFilterStore();
  
  return (
    <div className="filters">
      <input
        type="search"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search products..."
      />
      
      <select
        value={category || ''}
        onChange={(e) => setCategory(e.target.value || null)}
      >
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
        <option value="books">Books</option>
      </select>
      
      <div className="price-range">
        <input
          type="number"
          value={priceRange[0]}
          onChange={(e) => setPriceRange([Number(e.target.value), priceRange[1]])}
          placeholder="Min"
        />
        <input
          type="number"
          value={priceRange[1] === Infinity ? '' : priceRange[1]}
          onChange={(e) => setPriceRange([priceRange[0], Number(e.target.value) || Infinity])}
          placeholder="Max"
        />
      </div>
      
      <label>
        <input
          type="checkbox"
          checked={inStock}
          onChange={(e) => setInStock(e.target.checked)}
        />
        In Stock Only
      </label>
      
      <div className="sort">
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Name</option>
          <option value="price">Price</option>
          <option value="rating">Rating</option>
        </select>
        <button onClick={toggleSortOrder}>
          {sortOrder === 'asc' ? '↑' : '↓'}
        </button>
      </div>
      
      <button onClick={resetFilters}>Reset Filters</button>
    </div>
  );
}
```

---

# 24. Anti-Patterns to Avoid

## 24.1 Store Anti-Patterns

```tsx
// ❌ Anti-pattern 1: Storing derived state
const useStore = create((set) => ({
  items: [],
  filteredItems: [], // Don't store this!
  filter: '',
  setFilter: (filter) => set((state) => ({
    filter,
    filteredItems: state.items.filter(i => i.name.includes(filter)),
  })),
}));

// ✅ Better: Compute derived values
const useStore = create((set) => ({
  items: [],
  filter: '',
  setFilter: (filter) => set({ filter }),
}));

// Use selector to compute
const filteredItems = useStore((state) => 
  state.items.filter(i => i.name.includes(state.filter))
);

// ❌ Anti-pattern 2: Storing non-serializable data
const useStore = create((set) => ({
  callback: () => {}, // Functions change on every render
  element: document.getElementById('app'), // DOM elements
  date: new Date(), // Date objects
}));

// ✅ Better: Store only serializable data
const useStore = create((set) => ({
  timestamp: Date.now(), // Store as number
  elementId: 'app', // Store as reference
}));

// ❌ Anti-pattern 3: Not using selectors
function Component() {
  const store = useStore(); // Re-renders on ANY change
  return <p>{store.count}</p>;
}

// ✅ Better: Select only what you need
function Component() {
  const count = useStore((state) => state.count);
  return <p>{count}</p>;
}

// ❌ Anti-pattern 4: Mutating state directly
const useStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => {
    state.items.push(item); // Mutation!
    return { items: state.items };
  }),
}));

// ✅ Better: Return new array
const useStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({
    items: [...state.items, item],
  })),
}));
```

## 24.2 Context Anti-Patterns

```tsx
// ❌ Anti-pattern 1: Creating new object on every render
function Provider({ children }) {
  const [state, setState] = useState({});
  return (
    <MyContext.Provider value={{ state, setState }}> {/* New object every render */}
      {children}
    </MyContext.Provider>
  );
}

// ✅ Better: Memoize the value
function Provider({ children }) {
  const [state, setState] = useState({});
  const value = useMemo(() => ({ state, setState }), [state]);
  return (
    <MyContext.Provider value={value}>
      {children}
    </MyContext.Provider>
  );
}

// ❌ Anti-pattern 2: Putting everything in one context
const AppContext = createContext({
  user: null,
  theme: 'light',
  cart: [],
  notifications: [],
  // ... more state
});

// ✅ Better: Split into separate contexts
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const CartContext = createContext([]);
```

---

# 25. Debugging State Issues

## 25.1 Debug Tools

```tsx
// 1. Zustand DevTools
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((s) => ({ count: s.count + 1 }), false, 'increment'),
    }),
    { name: 'MyStore' }
  )
);

// 2. Console logging middleware
const log = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log('  applying', args);
      set(...args);
      console.log('  new state', get());
    },
    get,
    api
  );

const useStore = create(log((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
})));

// 3. React DevTools - useDebugValue
import { useDebugValue } from 'react';

function useMyStore() {
  const state = useStore();
  useDebugValue(state, (s) => `Count: ${s.count}`);
  return state;
}
```

## 25.2 Common Issues and Fixes

```tsx
// Issue 1: Stale closure
// ❌ Problem
const useStore = create((set, get) => ({
  count: 0,
  incrementAsync: () => {
    setTimeout(() => {
      set({ count: count + 1 }); // `count` is stale!
    }, 1000);
  },
}));

// ✅ Fix: Use get() inside the callback
const useStore = create((set, get) => ({
  count: 0,
  incrementAsync: () => {
    setTimeout(() => {
      set({ count: get().count + 1 });
    }, 1000);
  },
}));

// Issue 2: Infinite loops
// ❌ Problem
function Component() {
  const items = useStore((s) => s.items);
  const setItems = useStore((s) => s.setItems);
  
  useEffect(() => {
    setItems(items.filter(i => i.active)); // Triggers re-render
  }, [items, setItems]); // items changes, effect runs, items changes...
}

// ✅ Fix: Only run when necessary
function Component() {
  const items = useStore((s) => s.items);
  const setItems = useStore((s) => s.setItems);
  
  useEffect(() => {
    const activeItems = items.filter(i => i.active);
    if (activeItems.length !== items.length) {
      setItems(activeItems);
    }
  }, [items]);
}

// Issue 3: Re-renders from object creation in selector
// ❌ Problem
const data = useStore((s) => ({
  user: s.user,
  settings: s.settings,
})); // New object every time!

// ✅ Fix: Use shallow comparison
import { shallow } from 'zustand/shallow';

const data = useStore(
  (s) => ({
    user: s.user,
    settings: s.settings,
  }),
  shallow
);
```

---

# 26. Resources and Learning Path

## 26.1 Official Documentation

- **Zustand**: https://github.com/pmndrs/zustand
- **Redux Toolkit**: https://redux-toolkit.js.org/
- **React Query**: https://tanstack.com/query
- **Jotai**: https://jotai.org/
- **Recoil**: https://recoiljs.org/
- **MobX**: https://mobx.js.org/
- **XState**: https://xstate.js.org/

## 26.2 Learning Path

```
Week 1: React Built-in State
├── useState patterns
├── useReducer patterns
├── State lifting
└── When to use each

Week 2: Context API
├── Creating contexts
├── Provider patterns
├── Performance optimization
└── Context + Reducer combo

Week 3: Zustand
├── Basic store creation
├── Selectors and subscriptions
├── Middleware (persist, devtools)
├── Slices pattern
└── Testing stores

Week 4: Redux Toolkit (Optional)
├── Slices and reducers
├── Async thunks
├── RTK Query
└── Store configuration

Week 5: Server State
├── React Query basics
├── Queries and mutations
├── Caching strategies
├── Optimistic updates
└── Pagination patterns
```

## 26.3 Practice Projects

1. **Todo App**: Practice basic CRUD with local state
2. **Shopping Cart**: Practice shared state and persistence
3. **Dashboard**: Practice multiple stores and server state
4. **Chat App**: Practice WebSocket state and real-time updates
5. **Form Builder**: Practice complex form state and validation

---

# 27. Quick Interview Answers

## One-Sentence Answers

1. **What is state?** - Data that can change over time and affects what renders.

2. **Local vs Global state?** - Local is component-scoped; global is app-wide.

3. **Why Context causes re-renders?** - All consumers re-render when value changes.

4. **What is Zustand?** - Lightweight state management with hooks-based API.

5. **Redux vs Zustand?** - Redux is more structured; Zustand is simpler.

6. **What are selectors?** - Functions that extract specific state values.

7. **Why use React Query?** - Handles server state caching and synchronization.

8. **What is optimistic update?** - Update UI before server confirms.

9. **Flux architecture?** - Unidirectional data flow pattern.

10. **State normalization?** - Flatten nested data into lookup tables.

---

## 🎯 Full Study Checklist

### Fundamentals
- [ ] useState vs useReducer trade-offs
- [ ] State colocating principles
- [ ] Lifting state up pattern
- [ ] Prop drilling problem and solutions

### Context API
- [ ] Creating and providing context
- [ ] Custom hook pattern
- [ ] Performance optimization (split contexts, memo)
- [ ] Context + useReducer pattern

### Zustand (Primary)
- [ ] Store creation syntax
- [ ] Using selectors correctly
- [ ] Persist middleware
- [ ] DevTools integration
- [ ] Slices pattern for large apps
- [ ] Testing Zustand stores
- [ ] Subscribe outside React

### Redux Toolkit (Secondary)
- [ ] createSlice API
- [ ] configureStore setup
- [ ] Async thunks
- [ ] RTK Query basics
- [ ] Type-safe hooks

### Server State
- [ ] React Query setup
- [ ] useQuery and useMutation
- [ ] Query invalidation
- [ ] Optimistic updates
- [ ] Infinite queries

### Advanced Topics
- [ ] State normalization
- [ ] Undo/redo implementation
- [ ] Real-time state (WebSocket)
- [ ] Form state management
- [ ] Hydration handling for SSR

---

# 28. Bonus: More Scenario Questions

## Scenario 1: Multi-Tab Synchronization

**Q: How would you sync state across browser tabs?**

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface Store {
  user: User | null;
  setUser: (user: User | null) => void;
}

const useStore = create<Store>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
    }),
    {
      name: 'user-storage',
      storage: createJSONStorage(() => localStorage),
    }
  )
);

// Listen for storage events (from other tabs)
if (typeof window !== 'undefined') {
  window.addEventListener('storage', (event) => {
    if (event.key === 'user-storage') {
      const newValue = event.newValue;
      if (newValue) {
        try {
          const parsed = JSON.parse(newValue);
          useStore.setState(parsed.state);
        } catch (e) {
          console.error('Failed to parse storage event', e);
        }
      }
    }
  });
}
```

---

## Scenario 2: Feature Flags State

**Q: How would you manage feature flags in a React app?**

```tsx
import { create } from 'zustand';

interface FeatureFlags {
  darkMode: boolean;
  newDashboard: boolean;
  experimentalFeatures: boolean;
  betaFeatures: boolean;
}

interface FeatureFlagStore {
  flags: FeatureFlags;
  loading: boolean;
  fetchFlags: () => Promise<void>;
  isEnabled: (flag: keyof FeatureFlags) => boolean;
  setFlag: (flag: keyof FeatureFlags, value: boolean) => void;
}

const useFeatureFlags = create<FeatureFlagStore>((set, get) => ({
  flags: {
    darkMode: false,
    newDashboard: false,
    experimentalFeatures: false,
    betaFeatures: false,
  },
  loading: true,
  
  fetchFlags: async () => {
    set({ loading: true });
    try {
      const response = await fetch('/api/feature-flags');
      const flags = await response.json();
      set({ flags, loading: false });
    } catch (error) {
      set({ loading: false });
    }
  },
  
  isEnabled: (flag) => get().flags[flag],
  
  setFlag: (flag, value) => set((state) => ({
    flags: { ...state.flags, [flag]: value },
  })),
}));

// Hook for conditional rendering
function useFeature(flag: keyof FeatureFlags) {
  return useFeatureFlags((state) => state.flags[flag]);
}

// Component usage
function Dashboard() {
  const newDashboard = useFeature('newDashboard');
  
  if (newDashboard) {
    return <NewDashboard />;
  }
  
  return <OldDashboard />;
}
```

---

## Scenario 3: Undo Notifications

**Q: How would you implement "Undo" for a delete action?**

```tsx
import { create } from 'zustand';

interface DeletedItem {
  item: any;
  undoDeadline: number;
  timer: NodeJS.Timeout;
}

interface UndoStore {
  deletedItems: Map<string, DeletedItem>;
  deleteItem: (id: string, item: any, onDelete: () => void) => void;
  undoDelete: (id: string) => any | null;
  confirmDelete: (id: string) => void;
}

const useUndoStore = create<UndoStore>((set, get) => ({
  deletedItems: new Map(),
  
  deleteItem: (id, item, onDelete) => {
    const UNDO_TIMEOUT = 5000;
    
    const timer = setTimeout(() => {
      onDelete();
      set((state) => {
        const newMap = new Map(state.deletedItems);
        newMap.delete(id);
        return { deletedItems: newMap };
      });
    }, UNDO_TIMEOUT);
    
    set((state) => {
      const newMap = new Map(state.deletedItems);
      newMap.set(id, {
        item,
        undoDeadline: Date.now() + UNDO_TIMEOUT,
        timer,
      });
      return { deletedItems: newMap };
    });
  },
  
  undoDelete: (id) => {
    const { deletedItems } = get();
    const entry = deletedItems.get(id);
    
    if (entry) {
      clearTimeout(entry.timer);
      set((state) => {
        const newMap = new Map(state.deletedItems);
        newMap.delete(id);
        return { deletedItems: newMap };
      });
      return entry.item;
    }
    
    return null;
  },
  
  confirmDelete: (id) => {
    const { deletedItems } = get();
    const entry = deletedItems.get(id);
    
    if (entry) {
      clearTimeout(entry.timer);
      set((state) => {
        const newMap = new Map(state.deletedItems);
        newMap.delete(id);
        return { deletedItems: newMap };
      });
    }
  },
}));

// Usage
function TodoItem({ todo }: { todo: Todo }) {
  const { deleteItem, undoDelete } = useUndoStore();
  const { success } = useNotificationStore();
  
  const handleDelete = () => {
    deleteItem(
      todo.id,
      todo,
      () => api.deleteTodo(todo.id)
    );
    
    success({
      message: 'Item deleted',
      action: {
        label: 'Undo',
        onClick: () => {
          const restored = undoDelete(todo.id);
          if (restored) {
            // Add back to UI
          }
        },
      },
    });
  };
  
  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
}
```

---

## Scenario 4: Draft Auto-Save

**Q: How would you implement auto-save for drafts?**

```tsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { debounce } from 'lodash-es';

interface DraftStore {
  drafts: Record<string, { content: string; updatedAt: number }>;
  saveDraft: (id: string, content: string) => void;
  getDraft: (id: string) => string | null;
  deleteDraft: (id: string) => void;
  clearOldDrafts: () => void;
}

const useDraftStore = create<DraftStore>()(
  persist(
    (set, get) => ({
      drafts: {},
      
      saveDraft: debounce((id: string, content: string) => {
        set((state) => ({
          drafts: {
            ...state.drafts,
            [id]: { content, updatedAt: Date.now() },
          },
        }));
      }, 1000),
      
      getDraft: (id) => {
        const draft = get().drafts[id];
        return draft?.content || null;
      },
      
      deleteDraft: (id) => {
        set((state) => {
          const { [id]: _, ...rest } = state.drafts;
          return { drafts: rest };
        });
      },
      
      clearOldDrafts: () => {
        const ONE_WEEK = 7 * 24 * 60 * 60 * 1000;
        const now = Date.now();
        
        set((state) => {
          const newDrafts: Record<string, { content: string; updatedAt: number }> = {};
          
          Object.entries(state.drafts).forEach(([id, draft]) => {
            if (now - draft.updatedAt < ONE_WEEK) {
              newDrafts[id] = draft;
            }
          });
          
          return { drafts: newDrafts };
        });
      },
    }),
    { name: 'drafts-storage' }
  )
);

// Editor with auto-save
function Editor({ postId }: { postId: string }) {
  const { saveDraft, getDraft, deleteDraft } = useDraftStore();
  const [content, setContent] = useState('');
  
  // Load draft on mount
  useEffect(() => {
    const draft = getDraft(postId);
    if (draft) {
      setContent(draft);
    }
  }, [postId]);
  
  // Auto-save on change
  useEffect(() => {
    if (content) {
      saveDraft(postId, content);
    }
  }, [content, postId]);
  
  const handlePublish = async () => {
    await publishPost(postId, content);
    deleteDraft(postId);
  };
  
  return (
    <div>
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
      />
      <button onClick={handlePublish}>Publish</button>
    </div>
  );
}
```

---

## Scenario 5: Offline-First State

**Q: How would you build an offline-first app?**

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface SyncStatus {
  pending: number;
  syncing: boolean;
  lastSynced: number | null;
  error: string | null;
}

interface OfflineStore {
  items: Item[];
  pendingChanges: Change[];
  syncStatus: SyncStatus;
  isOnline: boolean;
  
  addItem: (item: Omit<Item, 'id'>) => void;
  updateItem: (id: string, updates: Partial<Item>) => void;
  deleteItem: (id: string) => void;
  sync: () => Promise<void>;
  setOnline: (online: boolean) => void;
}

const useOfflineStore = create<OfflineStore>()(
  persist(
    (set, get) => ({
      items: [],
      pendingChanges: [],
      syncStatus: {
        pending: 0,
        syncing: false,
        lastSynced: null,
        error: null,
      },
      isOnline: typeof navigator !== 'undefined' ? navigator.onLine : true,
      
      addItem: (item) => {
        const id = crypto.randomUUID();
        const newItem = { ...item, id };
        
        set((state) => ({
          items: [...state.items, newItem],
          pendingChanges: [...state.pendingChanges, { type: 'add', item: newItem }],
          syncStatus: { ...state.syncStatus, pending: state.syncStatus.pending + 1 },
        }));
        
        // Try to sync if online
        if (get().isOnline) {
          get().sync();
        }
      },
      
      updateItem: (id, updates) => {
        set((state) => ({
          items: state.items.map(i => i.id === id ? { ...i, ...updates } : i),
          pendingChanges: [...state.pendingChanges, { type: 'update', id, updates }],
          syncStatus: { ...state.syncStatus, pending: state.syncStatus.pending + 1 },
        }));
        
        if (get().isOnline) {
          get().sync();
        }
      },
      
      deleteItem: (id) => {
        set((state) => ({
          items: state.items.filter(i => i.id !== id),
          pendingChanges: [...state.pendingChanges, { type: 'delete', id }],
          syncStatus: { ...state.syncStatus, pending: state.syncStatus.pending + 1 },
        }));
        
        if (get().isOnline) {
          get().sync();
        }
      },
      
      sync: async () => {
        const { pendingChanges, syncStatus } = get();
        
        if (syncStatus.syncing || pendingChanges.length === 0) {
          return;
        }
        
        set((state) => ({
          syncStatus: { ...state.syncStatus, syncing: true, error: null },
        }));
        
        try {
          // Send all pending changes to server
          await fetch('/api/sync', {
            method: 'POST',
            body: JSON.stringify({ changes: pendingChanges }),
          });
          
          set({
            pendingChanges: [],
            syncStatus: {
              pending: 0,
              syncing: false,
              lastSynced: Date.now(),
              error: null,
            },
          });
        } catch (error) {
          set((state) => ({
            syncStatus: {
              ...state.syncStatus,
              syncing: false,
              error: 'Sync failed. Will retry when online.',
            },
          }));
        }
      },
      
      setOnline: (online) => {
        set({ isOnline: online });
        if (online) {
          get().sync();
        }
      },
    }),
    {
      name: 'offline-storage',
      storage: createJSONStorage(() => localStorage),
    }
  )
);

// Listen for online/offline events
if (typeof window !== 'undefined') {
  window.addEventListener('online', () => {
    useOfflineStore.getState().setOnline(true);
  });
  
  window.addEventListener('offline', () => {
    useOfflineStore.getState().setOnline(false);
  });
}
```

---

# 29. Common Interview Mistakes

## Mistakes to Avoid

1. **Not explaining WHY you'd choose a solution**
   - ❌ "I'd use Redux"
   - ✅ "I'd use Redux because the team is already familiar with it and we need time-travel debugging"

2. **Not considering performance**
   - ❌ "I'd put everything in Context"
   - ✅ "For frequently updating state, I'd avoid Context due to re-render issues and use Zustand instead"

3. **Not asking clarifying questions**
   - Ask about app size, team familiarity, existing tech stack

4. **Over-engineering simple problems**
   - Not every app needs global state management
   - Start with useState, only escalate when needed

5. **Mixing client and server state**
   - Server state belongs in React Query/SWR
   - Client state belongs in Zustand/Redux

---

# 30. Summary Tables

## Quick Decision Table

| Question | Answer |
|----------|--------|
| Do I need state? | If data changes and affects UI, yes |
| Where should state live? | As close to where it's used as possible |
| useState or useReducer? | useReducer for complex, related state |
| Context or Zustand? | Zustand for frequent updates |
| Redux or Zustand? | Redux for enterprise, Zustand for simplicity |
| Local or global? | Local first, global only when truly shared |

## Library Selection by Use Case

| Use Case | Recommended Library |
|----------|---------------------|
| Simple local state | useState |
| Complex local state | useReducer |
| Theme/auth globally | Context API |
| Shopping cart | Zustand |
| Enterprise dashboard | Redux Toolkit |
| API data | React Query |
| Real-time data | React Query + WebSocket |
| Forms | React Hook Form |
| Animations | Framer Motion (not state lib) |

## Performance Optimization Checklist

| Issue | Solution |
|-------|----------|
| All components re-render | Use selectors |
| Context re-renders | Split contexts |
| Expensive computations | useMemo |
| New object in selector | shallow comparison |
| Non-serializable state | Don't store functions/DOM |

---

*End of Part 6: State Management Complete Guide (6000+ lines)*

*Continue to Part 7 for Networking & API Integration...*

