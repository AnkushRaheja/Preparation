# Part 5: Next.js Framework Complete Guide

> **Master Next.js 15 App Router for enterprise applications**

---

## 5.1 Why Next.js?

Next.js is a **React framework** that provides:
- ✅ File-based routing
- ✅ Server-Side Rendering (SSR)
- ✅ Static Site Generation (SSG)
- ✅ API Routes
- ✅ Built-in optimization (images, fonts)
- ✅ Zero configuration

**CodeHealth uses Next.js 15** with the App Router.

---

## 5.2 App Router Architecture

### File Structure = Routes
```
app/
├── page.tsx                 → /
├── layout.tsx               → Wraps all pages
├── loading.tsx              → Loading UI
├── error.tsx                → Error boundary
├── not-found.tsx            → 404 page
├── dashboard/
│   ├── page.tsx             → /dashboard
│   ├── layout.tsx           → Dashboard layout
│   ├── projects/
│   │   └── page.tsx         → /dashboard/projects
│   └── settings/
│       └── page.tsx         → /dashboard/settings
└── (auth)/                  → Route Group (no URL impact)
    ├── login/
    │   └── page.tsx         → /login
    └── signup/
        └── page.tsx         → /signup
```

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | Route UI |
| `layout.tsx` | Shared UI (persists across navigation) |
| `loading.tsx` | Suspense fallback |
| `error.tsx` | Error boundary |
| `not-found.tsx` | 404 page |
| `template.tsx` | Like layout but re-mounts on navigation |

---

## 5.3 Dynamic Routes

### Basic Dynamic Route
```
app/projects/[id]/page.tsx → /projects/123
```

```typescript
// app/projects/[id]/page.tsx
interface Props {
  params: { id: string };
}

export default function ProjectPage({ params }: Props) {
  return <div>Project ID: {params.id}</div>;
}
```

### Catch-All Routes
```
app/docs/[...slug]/page.tsx → /docs/a/b/c
```

```typescript
interface Props {
  params: { slug: string[] };
}

export default function DocsPage({ params }: Props) {
  // /docs/getting-started/installation
  // params.slug = ['getting-started', 'installation']
  return <div>Path: {params.slug.join('/')}</div>;
}
```

---

## 5.4 Server vs Client Components

### Server Components (Default)
```typescript
// This is a Server Component by default
// Runs ONLY on the server
// Zero JavaScript sent to browser
export default async function ServerComponent() {
  // Can directly access databases, APIs with secrets
  const data = await db.query('SELECT * FROM users');
  
  return <div>{data.length} users</div>;
}
```

### Client Components
```typescript
'use client'; // MUST be first line

import { useState } from 'react';

export default function ClientComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

### When to Use Which?

| Feature | Server | Client |
|---------|--------|--------|
| Fetch data | ✅ | ⚠️ (useEffect) |
| Access backend | ✅ | ❌ |
| Use hooks | ❌ | ✅ |
| Event listeners | ❌ | ✅ |
| Use browser APIs | ❌ | ✅ |
| State management | ❌ | ✅ |

---

## 5.5 Data Fetching

### Server Component Fetching (Recommended)
```typescript
// Async Server Component
async function UserList() {
  // This fetch happens on the server
  const res = await fetch('https://api.example.com/users', {
    cache: 'no-store', // Always fresh data (SSR)
    // OR
    next: { revalidate: 60 } // Revalidate every 60 seconds (ISR)
  });
  const users = await res.json();
  
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### Client-Side Fetching
```typescript
'use client';

function ClientFetchExample() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  return data ? <div>{data.value}</div> : <div>Loading...</div>;
}
```

---

## 5.6 Rendering Strategies

### Static Rendering (Default)
- Pages rendered at **build time**
- Cached on CDN
- Best for static content

### Dynamic Rendering
Triggered when using:
- Dynamic functions (`cookies()`, `headers()`)
- `searchParams` in page props
- `cache: 'no-store'` in fetch

### Streaming with Suspense
```typescript
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Loading />}>
        <SlowComponent /> {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

---

## 5.7 Navigation

### Link Component
```typescript
import Link from 'next/link';

function Nav() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/dashboard">Dashboard</Link>
      <Link href={`/project/${id}`}>Project</Link>
    </nav>
  );
}
```

### Programmatic Navigation
```typescript
'use client';

import { useRouter } from 'next/navigation';

function LoginButton() {
  const router = useRouter();
  
  const handleLogin = async () => {
    await login();
    router.push('/dashboard');      // Navigate
    router.replace('/dashboard');   // Navigate without history
    router.refresh();               // Re-fetch server data
    router.back();                  // Go back
  };
  
  return <button onClick={handleLogin}>Login</button>;
}
```

---

## 🎯 Part 5 Interview Questions

1. **What is the difference between App Router and Pages Router?**
2. **Explain Server Components vs Client Components.**
3. **What is `use client` directive?**
4. **How does file-based routing work?**
5. **What are route groups `(folder)`?**
6. **Explain ISR (Incremental Static Regeneration).**
7. **How does data fetching differ between Server and Client Components?**
8. **What is streaming with Suspense?**
9. **How do you handle errors in Next.js?**
10. **What is the purpose of layout.tsx?**

---

# Part 6: State Management with Zustand

> **Master the state management library used in CodeHealth-AI**

---

## 6.1 Why Zustand?

Zustand is a **lightweight, fast** state management solution.

**Comparison:**

| Feature | Zustand | Redux | Context API |
|---------|---------|-------|-------------|
| Boilerplate | Very Low | High | Low |
| Learning Curve | Easy | Steep | Easy |
| Performance | Excellent | Good | Poor |
| Bundle Size | ~2KB | ~10KB | 0 (built-in) |
| DevTools | Yes | Yes | Limited |

---

## 6.2 Creating a Store

### Basic Store
```typescript
import { create } from 'zustand';

interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

### Usage in Components
```typescript
function Counter() {
  const { count, increment, decrement } = useCounterStore();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

---

## 6.3 Selectors (Performance Optimization)

**Problem:** Without selectors, component re-renders on ANY store change.

```typescript
// ❌ BAD - Re-renders when ANY property changes
const { user, count, theme } = useStore();

// ✅ GOOD - Only re-renders when 'count' changes
const count = useStore((state) => state.count);
```

### CodeHealth Pattern
```typescript
// From authStore.ts
function UserAvatar() {
  // Only subscribes to user, not isLoading or error
  const user = useAuthStore((state) => state.authUser);
  
  return <img src={user?.avatar} alt={user?.name} />;
}

function LoadingIndicator() {
  // Only subscribes to isLoading
  const isLoading = useAuthStore((state) => state.isloggingin);
  
  return isLoading ? <Spinner /> : null;
}
```

---

## 6.4 Async Actions

Zustand handles async naturally—no middleware needed.

```typescript
interface AuthStore {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
}

export const useAuthStore = create<AuthStore>((set, get) => ({
  user: null,
  isLoading: false,
  error: null,
  
  login: async (credentials) => {
    set({ isLoading: true, error: null });
    
    try {
      const response = await axiosInstance.post('/auth/login', credentials);
      set({ user: response.data.user, isLoading: false });
    } catch (error: any) {
      set({ 
        error: error.response?.data?.message || 'Login failed',
        isLoading: false 
      });
    }
  },
  
  logout: async () => {
    await axiosInstance.post('/auth/logout');
    set({ user: null, error: null });
  },
}));
```

---

## 6.5 Accessing State Outside React

```typescript
// In axios interceptor (not a React component)
import { useAuthStore } from './authStore';

axiosInstance.interceptors.request.use((config) => {
  // getState() returns current state snapshot
  const token = useAuthStore.getState().token;
  
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

---

## 6.6 Multiple Stores (Domain Separation)

CodeHealth uses separate stores for different domains:

```typescript
// authStore.ts - Authentication
export const useAuthStore = create<AuthStore>(...);

// githubStore.ts - GitHub repositories
export const useGitHubStore = create<GitHubStore>(...);

// analysisStore.ts - Code analysis
export const useAnalysisStore = create<AnalysisStore>(...);

// notificationStore.ts - Notifications
export const useNotificationStore = create<NotificationStore>(...);
```

---

## 🎯 Part 6 Interview Questions

1. **Why choose Zustand over Redux?**
2. **What is a selector in Zustand?**
3. **How do you handle async actions?**
4. **How do you access state outside React components?**
5. **What is `set` vs `get` in Zustand?**
6. **How does Zustand prevent unnecessary re-renders?**
7. **Can you use multiple stores?**
8. **How does Zustand compare to Context API?**
9. **What is the Flux pattern?**
10. **How do you persist Zustand state?**

---

*Continue to Part 7 for Networking & Security...*
