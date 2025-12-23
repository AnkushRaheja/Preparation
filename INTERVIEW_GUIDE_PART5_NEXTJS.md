# 📚 Part 5: Next.js Framework Complete Guide - Interview Mastery

> **The Ultimate Next.js Interview Preparation Guide**
> 
> Master Next.js 14+ with App Router, Server Components, and modern web development patterns for technical interviews.

---

## 📋 Table of Contents

1. [Introduction to Next.js](#1-introduction-to-nextjs)
2. [App Router Fundamentals](#2-app-router-fundamentals)
3. [Routing Deep Dive](#3-routing-deep-dive)
4. [Server Components vs Client Components](#4-server-components-vs-client-components)
5. [Data Fetching](#5-data-fetching)
6. [Caching and Revalidation](#6-caching-and-revalidation)
7. [Server Actions](#7-server-actions)
8. [Rendering Strategies](#8-rendering-strategies)
9. [Middleware](#9-middleware)
10. [API Routes](#10-api-routes)
11. [Styling](#11-styling)
12. [Optimization](#12-optimization)
13. [Authentication](#13-authentication)
14. [Deployment](#14-deployment)
15. [Interview Questions](#15-interview-questions)

---

# 1. Introduction to Next.js

## 1.1 What is Next.js?

Next.js is a React framework that provides:
- **Server-Side Rendering (SSR)** - Render pages on the server
- **Static Site Generation (SSG)** - Pre-render pages at build time
- **Incremental Static Regeneration (ISR)** - Update static pages after build
- **App Router** - File-based routing with React Server Components
- **API Routes** - Build API endpoints within your app
- **Built-in Optimizations** - Images, fonts, scripts

```bash
# Create a new Next.js app
npx create-next-app@latest my-app

# With TypeScript, Tailwind, ESLint, App Router
npx create-next-app@latest my-app --typescript --tailwind --eslint --app
```

## 1.2 Project Structure (App Router)

```
my-app/
├── app/
│   ├── layout.tsx          # Root layout
│   ├── page.tsx            # Home page (/)
│   ├── loading.tsx         # Loading UI
│   ├── error.tsx           # Error UI
│   ├── not-found.tsx       # 404 page
│   ├── globals.css         # Global styles
│   ├── about/
│   │   └── page.tsx        # /about
│   ├── blog/
│   │   ├── page.tsx        # /blog
│   │   └── [slug]/
│   │       └── page.tsx    # /blog/:slug
│   └── api/
│       └── route.ts        # API route
├── components/             # React components
├── lib/                    # Utility functions
├── public/                 # Static assets
├── next.config.js          # Next.js configuration
├── package.json
└── tsconfig.json
```

## 1.3 Pages Router vs App Router

| Feature | Pages Router | App Router |
|---------|--------------|------------|
| Directory | `pages/` | `app/` |
| Server Components | No | Yes (default) |
| Layouts | Per-page `_app.tsx` | Nested `layout.tsx` |
| Data Fetching | `getServerSideProps`, etc. | `async` components, `fetch` |
| Loading States | Manual | Built-in `loading.tsx` |
| Error Handling | `_error.tsx` | Nested `error.tsx` |
| Streaming | Limited | Full support |

---

# 2. App Router Fundamentals

## 2.1 Page Components

```tsx
// app/page.tsx - Home page
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to Next.js</h1>
      <p>This is a Server Component by default</p>
    </main>
  );
}

// app/about/page.tsx - About page
export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

## 2.2 Layouts

Layouts wrap pages and preserve state across navigation:

```tsx
// app/layout.tsx - Root layout (required)
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'My App',
  description: 'Built with Next.js',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
        {children}
        <footer>© 2024</footer>
      </body>
    </html>
  );
}

// app/dashboard/layout.tsx - Nested layout
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <aside>
        <nav>Dashboard Menu</nav>
      </aside>
      <main>{children}</main>
    </div>
  );
}
```

## 2.3 Loading States

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="loading">
      <div className="spinner" />
      <p>Loading...</p>
    </div>
  );
}

// Or use Suspense boundaries
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Loading />}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

## 2.4 Error Handling

```tsx
// app/dashboard/error.tsx
'use client'; // Error components must be Client Components

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// app/global-error.tsx - Catches root layout errors
'use client';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}
```

## 2.5 Not Found Page

```tsx
// app/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div>
      <h2>404 - Page Not Found</h2>
      <p>Could not find the requested resource</p>
      <Link href="/">Return Home</Link>
    </div>
  );
}

// Trigger programmatically
import { notFound } from 'next/navigation';

async function getUser(id: string) {
  const user = await fetchUser(id);
  if (!user) {
    notFound(); // Renders not-found.tsx
  }
  return user;
}
```

---

# 3. Routing Deep Dive

## 3.1 Dynamic Routes

```tsx
// app/blog/[slug]/page.tsx
interface Props {
  params: { slug: string };
}

export default function BlogPost({ params }: Props) {
  return <h1>Blog Post: {params.slug}</h1>;
}

// Generate static params at build time
export async function generateStaticParams() {
  const posts = await fetchPosts();
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

## 3.2 Catch-All Routes

```tsx
// app/docs/[...slug]/page.tsx
// Matches /docs/a, /docs/a/b, /docs/a/b/c
interface Props {
  params: { slug: string[] };
}

export default function DocsPage({ params }: Props) {
  // /docs/getting-started/installation
  // params.slug = ['getting-started', 'installation']
  return <div>Path: {params.slug.join('/')}</div>;
}

// app/[[...slug]]/page.tsx - Optional catch-all
// Also matches /docs (slug is undefined)
```

## 3.3 Route Groups

Organize routes without affecting URL:

```
app/
├── (marketing)/
│   ├── layout.tsx     # Marketing layout
│   ├── about/
│   │   └── page.tsx   # /about
│   └── contact/
│       └── page.tsx   # /contact
├── (shop)/
│   ├── layout.tsx     # Shop layout
│   ├── products/
│   │   └── page.tsx   # /products
│   └── cart/
│       └── page.tsx   # /cart
└── layout.tsx         # Root layout
```

## 3.4 Parallel Routes

Render multiple pages simultaneously:

```
app/
├── @dashboard/
│   └── page.tsx
├── @analytics/
│   └── page.tsx
├── layout.tsx
└── page.tsx
```

```tsx
// app/layout.tsx
export default function Layout({
  children,
  dashboard,
  analytics,
}: {
  children: React.ReactNode;
  dashboard: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <>
      {children}
      <div className="grid grid-cols-2">
        {dashboard}
        {analytics}
      </div>
    </>
  );
}
```

## 3.5 Intercepting Routes

Show content in a modal while preserving the URL:

```
app/
├── photos/
│   └── [id]/
│       └── page.tsx      # Full page view
├── @modal/
│   └── (.)photos/
│       └── [id]/
│           └── page.tsx  # Modal view
└── layout.tsx
```

Convention:
- `(.)` - Same level
- `(..)` - One level up
- `(..)(..)` - Two levels up
- `(...)` - From root

## 3.6 Navigation

```tsx
'use client';

import Link from 'next/link';
import { useRouter, usePathname, useSearchParams } from 'next/navigation';

export default function Navigation() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  const handleNavigation = () => {
    router.push('/dashboard');       // Navigate
    router.replace('/login');        // Replace current entry
    router.refresh();                // Refresh server components
    router.back();                   // Go back
    router.forward();                // Go forward
    router.prefetch('/about');       // Prefetch route
  };
  
  return (
    <nav>
      {/* Static link */}
      <Link href="/about">About</Link>
      
      {/* Dynamic link */}
      <Link href={`/blog/${post.slug}`}>Read Post</Link>
      
      {/* With query params */}
      <Link href="/search?q=hello">Search</Link>
      
      {/* Active link styling */}
      <Link 
        href="/dashboard"
        className={pathname === '/dashboard' ? 'active' : ''}
      >
        Dashboard
      </Link>
      
      {/* Prefetching disabled */}
      <Link href="/large-page" prefetch={false}>
        Large Page
      </Link>
      
      {/* Replace instead of push */}
      <Link href="/login" replace>Login</Link>
      
      {/* Programmatic navigation */}
      <button onClick={() => router.push('/settings')}>
        Settings
      </button>
    </nav>
  );
}
```

---

# 4. Server Components vs Client Components

## 4.1 Understanding the Difference

```tsx
// Server Component (default) - NO 'use client' directive
// ✅ Access backend directly
// ✅ Use async/await
// ✅ Access environment variables
// ✅ No JavaScript sent to client
// ❌ No useState, useEffect, etc.
// ❌ No browser APIs
// ❌ No event handlers

async function ServerComponent() {
  const data = await db.query('SELECT * FROM users');
  const secret = process.env.SECRET_KEY; // Safe!
  
  return <div>{data.map(u => u.name)}</div>;
}

// Client Component - HAS 'use client' directive
// ✅ useState, useEffect, etc.
// ✅ Event handlers
// ✅ Browser APIs
// ❌ Cannot be async
// ❌ Sends JavaScript to client
// ❌ No direct DB access

'use client';

import { useState } from 'react';

function ClientComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

## 4.2 When to Use Each

| Use Case | Server | Client |
|----------|--------|--------|
| Fetch data | ✅ | ❌ |
| Access backend | ✅ | ❌ |
| Use secrets | ✅ | ❌ |
| Large dependencies | ✅ | ❌ |
| Interactivity | ❌ | ✅ |
| useState/useEffect | ❌ | ✅ |
| Event handlers | ❌ | ✅ |
| Browser APIs | ❌ | ✅ |

## 4.3 Composition Patterns

```tsx
// Server Component with Client child
// ✅ This works - Server renders Client as a slot

// ServerComponent.tsx (no directive = server)
import ClientCounter from './ClientCounter';

async function ServerComponent() {
  const data = await fetchData();
  
  return (
    <div>
      <h1>{data.title}</h1>
      <ClientCounter /> {/* Client component */}
    </div>
  );
}

// ClientCounter.tsx
'use client';

import { useState } from 'react';

export default function ClientCounter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

```tsx
// Passing Server Components as children to Client
// ✅ This works - children are pre-rendered on server

// ClientWrapper.tsx
'use client';

export default function ClientWrapper({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  const [isOpen, setIsOpen] = useState(true);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children} {/* Server content passed as children */}
    </div>
  );
}

// page.tsx (Server Component)
import ClientWrapper from './ClientWrapper';
import ServerContent from './ServerContent';

export default async function Page() {
  return (
    <ClientWrapper>
      <ServerContent /> {/* Rendered on server, passed to client */}
    </ClientWrapper>
  );
}
```

## 4.4 Common Mistakes

```tsx
// ❌ Wrong: Importing Server Component into Client Component
'use client';

import ServerComponent from './ServerComponent'; // Will become client!

function ClientComponent() {
  return <ServerComponent />; // This is now a client component
}

// ✅ Correct: Pass as children or props
'use client';

function ClientComponent({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>; // Server content stays server
}
```

---

# 5. Data Fetching

## 5.1 Server Component Data Fetching

```tsx
// Direct async/await in Server Components
async function ProductsPage() {
  // This runs on the server
  const products = await fetch('https://api.example.com/products', {
    cache: 'force-cache', // Default: cache forever
  }).then(res => res.json());
  
  return (
    <ul>
      {products.map((p: Product) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}

// Multiple parallel fetches
async function Dashboard() {
  // Parallel requests - much faster!
  const [user, posts, comments] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/comments').then(r => r.json()),
  ]);
  
  return (
    <div>
      <UserCard user={user} />
      <PostList posts={posts} />
      <CommentList comments={comments} />
    </div>
  );
}
```

## 5.2 Fetch Options

```tsx
// Cache strategies
const data1 = await fetch(url, { 
  cache: 'force-cache'  // Default - cache forever (SSG-like)
});

const data2 = await fetch(url, { 
  cache: 'no-store'     // Never cache (SSR-like)
});

const data3 = await fetch(url, { 
  next: { revalidate: 60 }  // Revalidate every 60 seconds (ISR-like)
});

const data4 = await fetch(url, { 
  next: { tags: ['products'] }  // Tag for on-demand revalidation
});
```

## 5.3 Database Access

```tsx
// Direct database access in Server Components
import { db } from '@/lib/db';

async function UsersPage() {
  // No API needed - query directly!
  const users = await db.user.findMany({
    where: { active: true },
    orderBy: { createdAt: 'desc' },
  });
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 5.4 Sequential vs Parallel

```tsx
// ❌ Sequential - slow!
async function SlowPage() {
  const user = await getUser();      // Wait...
  const posts = await getPosts();     // Then wait...
  const comments = await getComments(); // Then wait...
}

// ✅ Parallel - fast!
async function FastPage() {
  const [user, posts, comments] = await Promise.all([
    getUser(),
    getPosts(),
    getComments(),
  ]);
}

// ✅ Parallel with Suspense boundaries
function StreamingPage() {
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserSection /> {/* Streams when ready */}
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <PostsSection /> {/* Streams when ready */}
      </Suspense>
    </div>
  );
}
```

---

# 6. Caching and Revalidation

## 6.1 Cache Behavior

```tsx
// Static data (cached forever)
fetch('https://api.example.com/data')
// Same as:
fetch('https://api.example.com/data', { cache: 'force-cache' })

// Dynamic data (never cached)
fetch('https://api.example.com/data', { cache: 'no-store' })

// Time-based revalidation
fetch('https://api.example.com/data', {
  next: { revalidate: 3600 } // Revalidate after 1 hour
})
```

## 6.2 Route Segment Config

```tsx
// app/products/page.tsx

// Force dynamic rendering
export const dynamic = 'force-dynamic';
// Options: 'auto' | 'force-dynamic' | 'error' | 'force-static'

// Set revalidation time for entire route
export const revalidate = 60; // seconds

// Set runtime
export const runtime = 'edge'; // 'nodejs' | 'edge'

export default async function ProductsPage() {
  const products = await getProducts();
  return <ProductList products={products} />;
}
```

## 6.3 On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const { path, tag, secret } = await request.json();
  
  // Validate secret
  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Invalid secret' }, { status: 401 });
  }
  
  // Revalidate by path
  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true, path });
  }
  
  // Revalidate by tag
  if (tag) {
    revalidateTag(tag);
    return Response.json({ revalidated: true, tag });
  }
  
  return Response.json({ error: 'No path or tag provided' }, { status: 400 });
}

// Using tags
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { tags: ['products'] }
  });
  return res.json();
}

// Revalidate all 'products' tagged requests
revalidateTag('products');
```

---

# 7. Server Actions

## 7.1 Basic Server Actions

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { db } from '@/lib/db';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  await db.post.create({
    data: { title, content },
  });
  
  revalidatePath('/posts');
  redirect('/posts');
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidatePath('/posts');
}
```

## 7.2 Using in Components

```tsx
// In Server Component
import { createPost } from './actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}

// In Client Component
'use client';

import { createPost, deletePost } from './actions';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export default function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <SubmitButton />
    </form>
  );
}
```

## 7.3 With useActionState (React 19)

```tsx
'use client';

import { useActionState } from 'react';
import { createUser } from './actions';

export default function SignupForm() {
  const [state, formAction, isPending] = useActionState(createUser, {
    message: '',
    errors: {},
  });
  
  return (
    <form action={formAction}>
      <input name="email" type="email" />
      {state.errors?.email && <p>{state.errors.email}</p>}
      
      <input name="password" type="password" />
      {state.errors?.password && <p>{state.errors.password}</p>}
      
      <button disabled={isPending}>
        {isPending ? 'Signing up...' : 'Sign Up'}
      </button>
      
      {state.message && <p>{state.message}</p>}
    </form>
  );
}

// actions.ts
'use server';

import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export async function createUser(prevState: any, formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
    password: formData.get('password'),
  });
  
  if (!validatedFields.success) {
    return {
      message: '',
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }
  
  // Create user...
  
  return { message: 'User created!', errors: {} };
}
```

---

# 8. Rendering Strategies

## 8.1 Static Rendering (Default)

```tsx
// Pages without dynamic data are rendered at build time
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// Force static even with fetch
export const dynamic = 'force-static';

export default async function ProductsPage() {
  const products = await fetch('https://api.example.com/products', {
    cache: 'force-cache'
  });
  return <ProductList products={products} />;
}
```

## 8.2 Dynamic Rendering

```tsx
// Using dynamic functions triggers dynamic rendering
import { cookies, headers } from 'next/headers';

export default async function ProfilePage() {
  const cookieStore = cookies(); // Dynamic function
  const token = cookieStore.get('token');
  
  // This page will be rendered at request time
  return <Profile token={token} />;
}

// Or force dynamic
export const dynamic = 'force-dynamic';

export default async function DashboardPage() {
  const data = await fetch('https://api.example.com/data', {
    cache: 'no-store'
  });
  return <Dashboard data={data} />;
}
```

## 8.3 Streaming with Suspense

```tsx
import { Suspense } from 'react';

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* These components stream in as they resolve */}
      <Suspense fallback={<RevenueCardSkeleton />}>
        <RevenueCard />
      </Suspense>
      
      <Suspense fallback={<UsersTableSkeleton />}>
        <UsersTable />
      </Suspense>
      
      <Suspense fallback={<ChartSkeleton />}>
        <AnalyticsChart />
      </Suspense>
    </div>
  );
}

// Each component fetches its own data
async function RevenueCard() {
  const revenue = await getRevenue(); // Slow query
  return <div>Revenue: ${revenue}</div>;
}

async function UsersTable() {
  const users = await getUsers(); // Another slow query
  return <table>{/* ... */}</table>;
}
```

---

# 9. Middleware

## 9.1 Basic Middleware

```tsx
// middleware.ts (root of project)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Log all requests
  console.log('Path:', request.nextUrl.pathname);
  
  // Continue to the route
  return NextResponse.next();
}

// Only run on specific paths
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

## 9.2 Authentication Middleware

```tsx
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');
  const isAuthPage = request.nextUrl.pathname.startsWith('/login');
  const isProtectedRoute = request.nextUrl.pathname.startsWith('/dashboard');
  
  // Redirect to login if accessing protected route without token
  if (isProtectedRoute && !token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('from', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }
  
  // Redirect to dashboard if already logged in
  if (isAuthPage && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/login'],
};
```

## 9.3 Internationalization

```tsx
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const locales = ['en', 'es', 'fr'];
const defaultLocale = 'en';

function getLocale(request: NextRequest): string {
  const acceptLanguage = request.headers.get('accept-language');
  // Parse and match locale
  return defaultLocale;
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Check if locale is in path
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameHasLocale) return;
  
  // Redirect to localized path
  const locale = getLocale(request);
  request.nextUrl.pathname = `/${locale}${pathname}`;
  
  return NextResponse.redirect(request.nextUrl);
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

# 10. API Routes

## 10.1 Route Handlers

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/users
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = searchParams.get('page') || '1';
  
  const users = await db.user.findMany({
    skip: (parseInt(page) - 1) * 10,
    take: 10,
  });
  
  return NextResponse.json(users);
}

// POST /api/users
export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const user = await db.user.create({
    data: body,
  });
  
  return NextResponse.json(user, { status: 201 });
}
```

## 10.2 Dynamic Routes

```tsx
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface Props {
  params: { id: string };
}

// GET /api/users/123
export async function GET(request: NextRequest, { params }: Props) {
  const user = await db.user.findUnique({
    where: { id: params.id },
  });
  
  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
  
  return NextResponse.json(user);
}

// PATCH /api/users/123
export async function PATCH(request: NextRequest, { params }: Props) {
  const body = await request.json();
  
  const user = await db.user.update({
    where: { id: params.id },
    data: body,
  });
  
  return NextResponse.json(user);
}

// DELETE /api/users/123
export async function DELETE(request: NextRequest, { params }: Props) {
  await db.user.delete({
    where: { id: params.id },
  });
  
  return new NextResponse(null, { status: 204 });
}
```

## 10.3 Headers and Cookies

```tsx
import { NextRequest, NextResponse } from 'next/server';
import { cookies, headers } from 'next/headers';

export async function GET(request: NextRequest) {
  // Read headers
  const headersList = headers();
  const auth = headersList.get('authorization');
  
  // Read cookies
  const cookieStore = cookies();
  const token = cookieStore.get('token');
  
  // Set response headers
  const response = NextResponse.json({ data: 'value' });
  response.headers.set('X-Custom-Header', 'value');
  
  // Set cookies
  response.cookies.set('session', 'abc123', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 60 * 24 * 7, // 1 week
  });
  
  return response;
}
```

---

# 11. Styling

## 11.1 CSS Modules

```tsx
// app/components/Button.module.css
.button {
  padding: 0.5rem 1rem;
  border-radius: 4px;
  border: none;
  cursor: pointer;
}

.primary {
  background: blue;
  color: white;
}

.secondary {
  background: gray;
  color: white;
}

// app/components/Button.tsx
import styles from './Button.module.css';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

export default function Button({ variant = 'primary', children }: ButtonProps) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

## 11.2 Tailwind CSS

```tsx
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981',
      },
    },
  },
  plugins: [],
};

export default config;

// Usage in components
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-xl font-bold text-gray-900 mb-4">{title}</h2>
      <div className="text-gray-600">{children}</div>
    </div>
  );
}
```

## 11.3 Global Styles

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom global styles */
:root {
  --primary-color: #3B82F6;
  --secondary-color: #10B981;
}

* {
  box-sizing: border-box;
}

body {
  font-family: var(--font-inter), system-ui, sans-serif;
  line-height: 1.6;
}

/* Custom utilities */
@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

## 11.4 Styled Components / CSS-in-JS

```tsx
// Use 'use client' for CSS-in-JS libraries
'use client';

import styled from 'styled-components';

const StyledButton = styled.button`
  padding: 0.5rem 1rem;
  background: ${props => props.$primary ? 'blue' : 'gray'};
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.9;
  }
`;

export default function Button({ primary, children }) {
  return <StyledButton $primary={primary}>{children}</StyledButton>;
}

// For SSR, configure registry in layout
// app/lib/StyledComponentsRegistry.tsx
'use client';

import { useState } from 'react';
import { useServerInsertedHTML } from 'next/navigation';
import { ServerStyleSheet, StyleSheetManager } from 'styled-components';

export default function StyledComponentsRegistry({
  children,
}: {
  children: React.ReactNode;
}) {
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet());

  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement();
    styledComponentsStyleSheet.instance.clearTag();
    return <>{styles}</>;
  });

  if (typeof window !== 'undefined') return <>{children}</>;

  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  );
}
```

---

# 12. Optimization

## 12.1 Image Optimization

```tsx
import Image from 'next/image';

// Basic usage
function Avatar() {
  return (
    <Image
      src="/avatar.jpg"
      alt="User avatar"
      width={64}
      height={64}
    />
  );
}

// Fill container
function HeroImage() {
  return (
    <div className="relative h-96 w-full">
      <Image
        src="/hero.jpg"
        alt="Hero"
        fill
        style={{ objectFit: 'cover' }}
        priority // Load immediately (above fold)
      />
    </div>
  );
}

// Remote images (configure in next.config.js)
function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <Image
      src={src}
      alt={alt}
      width={300}
      height={300}
      placeholder="blur"
      blurDataURL="/placeholder.png"
    />
  );
}

// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
        port: '',
        pathname: '/products/**',
      },
    ],
  },
};
```

## 12.2 Font Optimization

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}

// Local fonts
import localFont from 'next/font/local';

const myFont = localFont({
  src: './fonts/MyFont.woff2',
  display: 'swap',
});
```

## 12.3 Script Optimization

```tsx
import Script from 'next/script';

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      {children}
      
      {/* Load after page is interactive */}
      <Script
        src="https://example.com/analytics.js"
        strategy="afterInteractive"
      />
      
      {/* Load when browser is idle */}
      <Script
        src="https://example.com/widget.js"
        strategy="lazyOnload"
      />
      
      {/* Inline script */}
      <Script id="show-banner" strategy="afterInteractive">
        {`console.log('Page loaded')`}
      </Script>
      
      {/* With event handlers */}
      <Script
        src="https://example.com/sdk.js"
        strategy="afterInteractive"
        onLoad={() => console.log('Script loaded')}
        onError={() => console.log('Script failed')}
      />
    </>
  );
}
```

## 12.4 Lazy Loading

```tsx
import dynamic from 'next/dynamic';

// Lazy load a component
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false, // Disable server-side rendering
});

// Dynamic imports with named exports
const Tabs = dynamic(() => import('./Tabs').then(mod => mod.Tabs));

// Lazy load based on condition
function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && <HeavyChart />}
    </div>
  );
}
```

## 12.5 Metadata and SEO

```tsx
// Static metadata
export const metadata = {
  title: 'My App',
  description: 'My awesome Next.js app',
  openGraph: {
    title: 'My App',
    description: 'My awesome Next.js app',
    url: 'https://example.com',
    siteName: 'My App',
    images: [
      {
        url: 'https://example.com/og.png',
        width: 1200,
        height: 630,
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My App',
    description: 'My awesome Next.js app',
    images: ['https://example.com/og.png'],
  },
};

// Dynamic metadata
export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}
```

---

# 13. Authentication

## 13.1 NextAuth.js (Auth.js)

```tsx
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import Google from 'next-auth/providers/google';
import Credentials from 'next-auth/providers/credentials';

const handler = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
    Google({
      clientId: process.env.GOOGLE_ID!,
      clientSecret: process.env.GOOGLE_SECRET!,
    }),
    Credentials({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        const user = await verifyCredentials(
          credentials?.email,
          credentials?.password
        );
        return user;
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.id = token.id;
      session.user.role = token.role;
      return session;
    },
  },
  pages: {
    signIn: '/login',
    error: '/auth/error',
  },
});

export { handler as GET, handler as POST };
```

## 13.2 Session Handling

```tsx
// Server Component
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const session = await getServerSession(authOptions);
  
  if (!session) {
    redirect('/login');
  }
  
  return <Dashboard user={session.user} />;
}

// Client Component
'use client';

import { useSession, signIn, signOut } from 'next-auth/react';

export default function AuthButton() {
  const { data: session, status } = useSession();
  
  if (status === 'loading') {
    return <div>Loading...</div>;
  }
  
  if (session) {
    return (
      <div>
        <span>Welcome, {session.user?.name}</span>
        <button onClick={() => signOut()}>Sign Out</button>
      </div>
    );
  }
  
  return <button onClick={() => signIn()}>Sign In</button>;
}
```

## 13.3 Protecting Routes

```tsx
// middleware.ts
import { withAuth } from 'next-auth/middleware';

export default withAuth({
  pages: {
    signIn: '/login',
  },
});

export const config = {
  matcher: ['/dashboard/:path*', '/settings/:path*'],
};

// Or manual protection
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';

export default async function ProtectedPage() {
  const session = await getServerSession();
  
  if (!session) {
    redirect('/login?callbackUrl=/protected');
  }
  
  return <div>Protected content for {session.user?.email}</div>;
}
```

---

# 14. Deployment

## 14.1 Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Production deployment
vercel --prod

# Environment variables
vercel env add NEXT_PUBLIC_API_URL
```

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "framework": "nextjs",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_API_URL": "https://api.example.com"
  }
}
```

## 14.2 Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

```js
// next.config.js for standalone output
module.exports = {
  output: 'standalone',
};
```

## 14.3 Static Export

```js
// next.config.js
module.exports = {
  output: 'export',
  images: {
    unoptimized: true, // Required for static export
  },
};
```

```bash
# Build static files
npm run build

# Files are in 'out' directory
```

## 14.4 Environment Variables

```bash
# .env.local (local development, git ignored)
DATABASE_URL="postgresql://..."
NEXTAUTH_SECRET="your-secret"

# .env (shared defaults)
NEXT_PUBLIC_APP_NAME="My App"

# .env.production (production defaults)
NEXT_PUBLIC_API_URL="https://api.example.com"
```

```tsx
// Access in code
// Server-side (all env vars)
const dbUrl = process.env.DATABASE_URL;

// Client-side (only NEXT_PUBLIC_*)
const appName = process.env.NEXT_PUBLIC_APP_NAME;
```

---

# 15. Interview Questions

## 15.1 Basic Questions

**Q1: What is Next.js?**

Next.js is a React framework that provides server-side rendering, static site generation, file-based routing, API routes, and many built-in optimizations. It enables building production-ready React applications with minimal configuration.

---

**Q2: What are the differences between Pages Router and App Router?**

| Feature | Pages Router | App Router |
|---------|--------------|------------|
| Directory | `pages/` | `app/` |
| Default Components | Client | Server |
| Data Fetching | `getServerSideProps`, `getStaticProps` | `async` components, `fetch()` |
| Layouts | `_app.tsx` (global) | Nested `layout.tsx` |
| Loading States | Manual | `loading.tsx` |
| Error Handling | `_error.tsx` | Nested `error.tsx` |
| Streaming | Limited | Full support |

---

**Q3: What are Server Components?**

Server Components are React components that render on the server and send only HTML to the client. Benefits:
- Access backend directly (database, file system)
- Keep secrets secure (environment variables)
- Reduce client-side JavaScript bundle
- Better initial page load performance

---

**Q4: When should you use 'use client'?**

Add `'use client'` when you need:
- React hooks (useState, useEffect, etc.)
- Event handlers (onClick, onChange)
- Browser APIs (window, document)
- Third-party client libraries

---

**Q5: Explain the different rendering strategies**

| Strategy | When Rendered | Use Case |
|----------|--------------|----------|
| Static (SSG) | Build time | Blog, marketing pages |
| Dynamic (SSR) | Request time | User-specific content |
| ISR | Build + background | News, e-commerce |
| Streaming | Progressive | Dashboards |

---

## 15.2 Advanced Questions

**Q6: How does caching work in Next.js?**

Next.js has four cache layers:
1. **Request Memoization** - Deduplicates identical fetches in single render
2. **Data Cache** - Persists fetch results across requests
3. **Full Route Cache** - Caches rendered HTML and RSC payload
4. **Router Cache** - Client-side cache for navigation

Control with:
- `cache: 'force-cache'` (static)
- `cache: 'no-store'` (dynamic)
- `next: { revalidate: 60 }` (ISR)
- `revalidatePath()` / `revalidateTag()` (on-demand)

---

**Q7: What are Server Actions?**

Server Actions are async functions that run on the server, triggered from client-side forms or event handlers:

```tsx
'use server';

async function createPost(formData: FormData) {
  await db.post.create({ data: { title: formData.get('title') } });
  revalidatePath('/posts');
}
```

Benefits:
- No API route needed
- Form submission without JavaScript
- Automatic form state management
- Progressive enhancement

---

**Q8: Explain Parallel and Intercepting Routes**

**Parallel Routes (@)**: Render multiple pages simultaneously in the same layout.
```
app/
├── @dashboard/page.tsx
├── @analytics/page.tsx
└── layout.tsx
```

**Intercepting Routes (.)**: Show content in a modal while preserving the URL.
```
app/
├── photos/[id]/page.tsx      # Full page
├── @modal/(.)photos/[id]/page.tsx  # Modal view
```

---

**Q9: How do you optimize performance in Next.js?**

1. **Images**: Use `next/image` for automatic optimization
2. **Fonts**: Use `next/font` for zero layout shift
3. **Scripts**: Use `next/script` with proper strategies
4. **Code Splitting**: Dynamic imports with `next/dynamic`
5. **Caching**: Configure fetch cache appropriately
6. **Streaming**: Use Suspense for progressive loading
7. **Server Components**: Reduce client JS bundle

---

**Q10: What is Middleware in Next.js?**

Middleware runs before every request, enabling:
- Authentication/authorization
- Redirects and rewrites
- Header modification
- A/B testing
- Internationalization

```tsx
// middleware.ts
export function middleware(request: NextRequest) {
  const token = request.cookies.get('token');
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}
```

---

# 16. Coding Challenges

## 16.1 Implement Protected Route

```tsx
// lib/auth.ts
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';

export async function requireAuth() {
  const session = await getServerSession();
  if (!session) {
    redirect('/login');
  }
  return session;
}

// app/dashboard/page.tsx
export default async function DashboardPage() {
  const session = await requireAuth();
  return <Dashboard user={session.user} />;
}
```

## 16.2 Implement Search with URL State

```tsx
'use client';

import { useRouter, useSearchParams } from 'next/navigation';
import { useTransition } from 'react';

export default function SearchInput() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (term: string) => {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('q', term);
    } else {
      params.delete('q');
    }
    
    startTransition(() => {
      router.push(`/search?${params.toString()}`);
    });
  };
  
  return (
    <input
      type="search"
      defaultValue={searchParams.get('q') ?? ''}
      onChange={e => handleSearch(e.target.value)}
      placeholder="Search..."
      className={isPending ? 'opacity-50' : ''}
    />
  );
}
```

## 16.3 Implement Infinite Scroll

```tsx
'use client';

import { useCallback, useEffect, useRef, useState } from 'react';
import { getPosts } from './actions';

export default function InfinitePostList({ initialPosts }) {
  const [posts, setPosts] = useState(initialPosts);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const observerRef = useRef<IntersectionObserver>();
  
  const lastPostRef = useCallback((node: HTMLElement | null) => {
    if (loading) return;
    
    if (observerRef.current) {
      observerRef.current.disconnect();
    }
    
    observerRef.current = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting && hasMore) {
        loadMore();
      }
    });
    
    if (node) {
      observerRef.current.observe(node);
    }
  }, [loading, hasMore]);
  
  const loadMore = async () => {
    setLoading(true);
    const newPosts = await getPosts(page + 1);
    setPosts(prev => [...prev, ...newPosts]);
    setPage(p => p + 1);
    setHasMore(newPosts.length > 0);
    setLoading(false);
  };
  
  return (
    <div>
      {posts.map((post, index) => (
        <div 
          key={post.id} 
          ref={index === posts.length - 1 ? lastPostRef : null}
        >
          {post.title}
        </div>
      ))}
      {loading && <div>Loading more...</div>}
    </div>
  );
}
```

---

# 17. Next.js Configuration

## 17.1 next.config.js

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Strict mode for React
  reactStrictMode: true,
  
  // Image domains
  images: {
    remotePatterns: [
      { hostname: 'images.example.com' },
      { hostname: '*.cloudinary.com' },
    ],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: 'value',
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-page',
        destination: '/new-page',
        permanent: true, // 308
      },
    ];
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.example.com/:path*',
      },
    ];
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
        ],
      },
    ];
  },
  
  // Webpack customization
  webpack: (config, { buildId, dev, isServer }) => {
    // Custom webpack config
    return config;
  },
  
  // Output mode
  output: 'standalone', // For Docker
  
  // Experimental features
  experimental: {
    serverActions: true,
  },
};

module.exports = nextConfig;
```

---

# 18. Common Patterns

## 18.1 API Client

```tsx
// lib/api.ts
class ApiClient {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    
    const response = await fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
    });
    
    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }
    
    return response.json();
  }
  
  get<T>(endpoint: string, options?: RequestInit) {
    return this.request<T>(endpoint, { ...options, method: 'GET' });
  }
  
  post<T>(endpoint: string, data: unknown, options?: RequestInit) {
    return this.request<T>(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
    });
  }
}

export const api = new ApiClient(process.env.NEXT_PUBLIC_API_URL!);
```

## 18.2 Error Boundary Pattern

```tsx
// components/ErrorBoundary.tsx
'use client';

import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Something went wrong</div>;
    }
    
    return this.props.children;
  }
}
```

## 18.3 Loading State Pattern

```tsx
// components/LoadingButton.tsx
'use client';

import { useFormStatus } from 'react-dom';

interface Props {
  children: React.ReactNode;
  loadingText?: string;
}

export function LoadingButton({ children, loadingText = 'Loading...' }: Props) {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <>
          <Spinner className="mr-2 h-4 w-4 animate-spin" />
          {loadingText}
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

---

## 🎯 Final Study Checklist

### Core Concepts
- [ ] App Router vs Pages Router
- [ ] File-based routing (dynamic, catch-all, groups)
- [ ] Layouts and templates
- [ ] Loading and error states
- [ ] Server vs Client Components

### Data Fetching
- [ ] Server Component data fetching
- [ ] Parallel vs sequential requests
- [ ] Cache options and revalidation
- [ ] Server Actions

### Rendering
- [ ] Static rendering
- [ ] Dynamic rendering
- [ ] Streaming with Suspense
- [ ] ISR (Incremental Static Regeneration)

### Optimization
- [ ] Image optimization
- [ ] Font optimization
- [ ] Script optimization
- [ ] Metadata and SEO

### Infrastructure
- [ ] Middleware
- [ ] API Routes
- [ ] Authentication
- [ ] Deployment options

---

# 19. Pages Router (Legacy)

> Note: While App Router is recommended for new projects, many existing projects use Pages Router.

## 19.1 Basic Page Structure

```tsx
// pages/index.tsx
export default function HomePage() {
  return <h1>Welcome</h1>;
}

// pages/about.tsx
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// pages/blog/[slug].tsx
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.params;
  
  return <h1>Post: {slug}</h1>;
}
```

## 19.2 getServerSideProps (SSR)

```tsx
// pages/dashboard.tsx
import { GetServerSideProps } from 'next';

interface Props {
  user: User;
  posts: Post[];
}

export default function DashboardPage({ user, posts }: Props) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  const { req, res, params, query } = context;
  
  // Access cookies
  const token = req.cookies.token;
  
  // Redirect if not authenticated
  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }
  
  // Fetch data
  const user = await getUser(token);
  const posts = await getPosts(user.id);
  
  return {
    props: {
      user,
      posts,
    },
  };
};
```

## 19.3 getStaticProps (SSG)

```tsx
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Props {
  post: Post;
}

export default function BlogPost({ post }: Props) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await getAllPosts();
  
  return {
    paths: posts.map(post => ({
      params: { slug: post.slug },
    })),
    fallback: false, // 404 for unknown slugs
    // fallback: true, // Generate on first request
    // fallback: 'blocking', // SSR then cache
  };
};

export const getStaticProps: GetStaticProps<Props> = async ({ params }) => {
  const post = await getPostBySlug(params?.slug as string);
  
  if (!post) {
    return { notFound: true };
  }
  
  return {
    props: { post },
    revalidate: 60, // ISR: regenerate every 60 seconds
  };
};
```

## 19.4 API Routes (Pages Router)

```tsx
// pages/api/users/index.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    const users = await getUsers();
    return res.status(200).json(users);
  }
  
  if (req.method === 'POST') {
    const user = await createUser(req.body);
    return res.status(201).json(user);
  }
  
  res.setHeader('Allow', ['GET', 'POST']);
  res.status(405).end(`Method ${req.method} Not Allowed`);
}

// pages/api/users/[id].ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;
  
  if (req.method === 'GET') {
    const user = await getUserById(id as string);
    if (!user) {
      return res.status(404).json({ error: 'Not found' });
    }
    return res.status(200).json(user);
  }
  
  if (req.method === 'PUT') {
    const user = await updateUser(id as string, req.body);
    return res.status(200).json(user);
  }
  
  if (req.method === 'DELETE') {
    await deleteUser(id as string);
    return res.status(204).end();
  }
}
```

## 19.5 Custom App and Document

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app';
import { SessionProvider } from 'next-auth/react';
import '../styles/globals.css';

export default function MyApp({ 
  Component, 
  pageProps: { session, ...pageProps } 
}: AppProps) {
  return (
    <SessionProvider session={session}>
      <Layout>
        <Component {...pageProps} />
      </Layout>
    </SessionProvider>
  );
}

// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        <link rel="icon" href="/favicon.ico" />
        <meta name="theme-color" content="#000000" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

---

# 20. Advanced Next.js Patterns

## 20.1 Composition Pattern for Data Fetching

```tsx
// Separate data fetching from presentation
// lib/data.ts
export async function getProducts(options?: {
  category?: string;
  limit?: number;
}) {
  const params = new URLSearchParams();
  if (options?.category) params.set('category', options.category);
  if (options?.limit) params.set('limit', String(options.limit));
  
  const res = await fetch(`${API_URL}/products?${params}`, {
    next: { revalidate: 60 }
  });
  
  if (!res.ok) throw new Error('Failed to fetch products');
  return res.json();
}

// components/ProductList.tsx
interface ProductListProps {
  getProducts: () => Promise<Product[]>;
}

export async function ProductList({ getProducts }: ProductListProps) {
  const products = await getProducts();
  
  return (
    <ul>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </ul>
  );
}

// app/products/page.tsx
import { ProductList } from '@/components/ProductList';
import { getProducts } from '@/lib/data';

export default function ProductsPage() {
  return (
    <Suspense fallback={<ProductListSkeleton />}>
      <ProductList getProducts={() => getProducts({ limit: 20 })} />
    </Suspense>
  );
}
```

## 20.2 Optimistic Updates Pattern

```tsx
'use client';

import { useOptimistic, useTransition } from 'react';
import { updateLike } from './actions';

interface Post {
  id: string;
  likes: number;
  isLiked: boolean;
}

export function LikeButton({ post }: { post: Post }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticPost, addOptimistic] = useOptimistic(
    post,
    (state, newLiked: boolean) => ({
      ...state,
      likes: newLiked ? state.likes + 1 : state.likes - 1,
      isLiked: newLiked,
    })
  );
  
  const handleLike = () => {
    startTransition(async () => {
      addOptimistic(!optimisticPost.isLiked);
      await updateLike(post.id, !optimisticPost.isLiked);
    });
  };
  
  return (
    <button onClick={handleLike} disabled={isPending}>
      {optimisticPost.isLiked ? '❤️' : '🤍'} {optimisticPost.likes}
    </button>
  );
}
```

## 20.3 Parallel Data Loading Pattern

```tsx
// Load multiple data sources in parallel with streaming
import { Suspense } from 'react';

export default async function DashboardPage() {
  // These all start fetching immediately
  return (
    <div className="grid grid-cols-2 gap-4">
      <Suspense fallback={<CardSkeleton />}>
        <RevenueCard />
      </Suspense>
      
      <Suspense fallback={<CardSkeleton />}>
        <UsersCard />
      </Suspense>
      
      <Suspense fallback={<TableSkeleton />}>
        <RecentOrdersTable />
      </Suspense>
      
      <Suspense fallback={<ChartSkeleton />}>
        <AnalyticsChart />
      </Suspense>
    </div>
  );
}

// Each component fetches its own data
async function RevenueCard() {
  const data = await getRevenue();
  return <Card title="Revenue" value={`$${data.total}`} />;
}

async function UsersCard() {
  const data = await getUserStats();
  return <Card title="Users" value={data.total} />;
}
```

## 20.4 Route Handler Middleware Pattern

```tsx
// lib/api-middleware.ts
import { NextRequest, NextResponse } from 'next/server';

type Handler = (
  request: NextRequest,
  context: { params: Record<string, string> }
) => Promise<Response>;

export function withAuth(handler: Handler): Handler {
  return async (request, context) => {
    const token = request.headers.get('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    try {
      const user = await verifyToken(token);
      // Attach user to request
      (request as any).user = user;
      return handler(request, context);
    } catch {
      return NextResponse.json(
        { error: 'Invalid token' },
        { status: 401 }
      );
    }
  };
}

export function withRateLimit(handler: Handler, limit = 100): Handler {
  return async (request, context) => {
    const ip = request.ip || 'unknown';
    const remaining = await checkRateLimit(ip, limit);
    
    if (remaining <= 0) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        { status: 429 }
      );
    }
    
    const response = await handler(request, context);
    response.headers.set('X-RateLimit-Remaining', String(remaining));
    return response;
  };
}

// app/api/protected/route.ts
import { withAuth, withRateLimit } from '@/lib/api-middleware';

async function handler(request: NextRequest) {
  const user = (request as any).user;
  return NextResponse.json({ user });
}

export const GET = withRateLimit(withAuth(handler));
```

## 20.5 Form with Server Actions Pattern

```tsx
// app/contact/page.tsx
import { submitContact } from './actions';
import { ContactForm } from './form';

export default function ContactPage() {
  return (
    <div>
      <h1>Contact Us</h1>
      <ContactForm action={submitContact} />
    </div>
  );
}

// app/contact/form.tsx
'use client';

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Sending...' : 'Send Message'}
    </button>
  );
}

interface FormState {
  success: boolean;
  message: string;
  errors?: Record<string, string[]>;
}

export function ContactForm({ action }: { action: (state: FormState, formData: FormData) => Promise<FormState> }) {
  const [state, formAction] = useActionState(action, {
    success: false,
    message: '',
  });
  
  return (
    <form action={formAction}>
      {state.message && (
        <div className={state.success ? 'success' : 'error'}>
          {state.message}
        </div>
      )}
      
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" name="name" required />
        {state.errors?.name && <p className="error">{state.errors.name[0]}</p>}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" required />
        {state.errors?.email && <p className="error">{state.errors.email[0]}</p>}
      </div>
      
      <div>
        <label htmlFor="message">Message</label>
        <textarea id="message" name="message" required />
        {state.errors?.message && <p className="error">{state.errors.message[0]}</p>}
      </div>
      
      <SubmitButton />
    </form>
  );
}

// app/contact/actions.ts
'use server';

import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
});

export async function submitContact(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  const validatedFields = schema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
    message: formData.get('message'),
  });
  
  if (!validatedFields.success) {
    return {
      success: false,
      message: 'Validation failed',
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }
  
  try {
    await sendEmail(validatedFields.data);
    return {
      success: true,
      message: 'Message sent successfully!',
    };
  } catch (error) {
    return {
      success: false,
      message: 'Failed to send message. Please try again.',
    };
  }
}
```

---

# 21. Real-World Examples

## 21.1 E-commerce Product Page

```tsx
// app/products/[id]/page.tsx
import { Suspense } from 'react';
import { notFound } from 'next/navigation';
import { getProduct, getRelatedProducts, getReviews } from '@/lib/data';
import { AddToCartButton } from '@/components/AddToCartButton';

interface Props {
  params: { id: string };
}

export async function generateMetadata({ params }: Props) {
  const product = await getProduct(params.id);
  
  if (!product) return {};
  
  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.image],
    },
  };
}

export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id);
  
  if (!product) {
    notFound();
  }
  
  return (
    <div className="max-w-6xl mx-auto">
      <div className="grid md:grid-cols-2 gap-8">
        {/* Product Images */}
        <div className="space-y-4">
          <img
            src={product.image}
            alt={product.name}
            className="w-full rounded-lg"
          />
          <div className="grid grid-cols-4 gap-2">
            {product.images.map((img, i) => (
              <img
                key={i}
                src={img}
                alt={`${product.name} ${i + 1}`}
                className="rounded cursor-pointer hover:opacity-80"
              />
            ))}
          </div>
        </div>
        
        {/* Product Info */}
        <div className="space-y-6">
          <h1 className="text-3xl font-bold">{product.name}</h1>
          <p className="text-2xl text-green-600">${product.price}</p>
          <p className="text-gray-600">{product.description}</p>
          
          <div className="space-y-4">
            <AddToCartButton productId={product.id} />
          </div>
        </div>
      </div>
      
      {/* Reviews Section */}
      <section className="mt-12">
        <h2 className="text-2xl font-bold mb-6">Customer Reviews</h2>
        <Suspense fallback={<ReviewsSkeleton />}>
          <Reviews productId={params.id} />
        </Suspense>
      </section>
      
      {/* Related Products */}
      <section className="mt-12">
        <h2 className="text-2xl font-bold mb-6">Related Products</h2>
        <Suspense fallback={<ProductGridSkeleton />}>
          <RelatedProducts productId={params.id} />
        </Suspense>
      </section>
    </div>
  );
}

async function Reviews({ productId }: { productId: string }) {
  const reviews = await getReviews(productId);
  
  return (
    <div className="space-y-4">
      {reviews.map(review => (
        <div key={review.id} className="border p-4 rounded">
          <div className="flex items-center gap-2">
            <span className="font-bold">{review.author}</span>
            <span className="text-yellow-500">{'★'.repeat(review.rating)}</span>
          </div>
          <p className="mt-2">{review.content}</p>
        </div>
      ))}
    </div>
  );
}

async function RelatedProducts({ productId }: { productId: string }) {
  const products = await getRelatedProducts(productId);
  
  return (
    <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

## 21.2 Blog with MDX

```tsx
// app/blog/[slug]/page.tsx
import { getPostBySlug, getAllPosts } from '@/lib/mdx';
import { MDXRemote } from 'next-mdx-remote/rsc';
import { notFound } from 'next/navigation';

export async function generateStaticParams() {
  const posts = await getAllPosts();
  return posts.map(post => ({ slug: post.slug }));
}

export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPostBySlug(params.slug);
  
  if (!post) return {};
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      type: 'article',
      publishedTime: post.date,
      authors: [post.author],
    },
  };
}

export default async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await getPostBySlug(params.slug);
  
  if (!post) {
    notFound();
  }
  
  return (
    <article className="max-w-3xl mx-auto prose dark:prose-invert">
      <header className="mb-8">
        <h1>{post.title}</h1>
        <div className="flex items-center gap-4 text-gray-500">
          <time dateTime={post.date}>
            {new Date(post.date).toLocaleDateString()}
          </time>
          <span>•</span>
          <span>{post.author}</span>
        </div>
      </header>
      
      <MDXRemote 
        source={post.content}
        components={{
          // Custom components for MDX
          CodeBlock,
          Callout,
          Image: NextImage,
        }}
      />
    </article>
  );
}
```

## 21.3 Dashboard with Role-Based Access

```tsx
// app/dashboard/layout.tsx
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';
import { Sidebar } from '@/components/Sidebar';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await getServerSession();
  
  if (!session) {
    redirect('/login?callbackUrl=/dashboard');
  }
  
  return (
    <div className="flex h-screen">
      <Sidebar user={session.user} />
      <main className="flex-1 overflow-auto p-8">
        {children}
      </main>
    </div>
  );
}

// app/dashboard/admin/page.tsx
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';

export default async function AdminPage() {
  const session = await getServerSession();
  
  if (session?.user?.role !== 'admin') {
    redirect('/dashboard');
  }
  
  return (
    <div>
      <h1>Admin Dashboard</h1>
      <p>Only admins can see this page.</p>
    </div>
  );
}

// components/Sidebar.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

const navItems = [
  { href: '/dashboard', label: 'Overview', roles: ['user', 'admin'] },
  { href: '/dashboard/analytics', label: 'Analytics', roles: ['user', 'admin'] },
  { href: '/dashboard/settings', label: 'Settings', roles: ['user', 'admin'] },
  { href: '/dashboard/admin', label: 'Admin', roles: ['admin'] },
];

export function Sidebar({ user }: { user: User }) {
  const pathname = usePathname();
  
  const filteredItems = navItems.filter(item => 
    item.roles.includes(user.role)
  );
  
  return (
    <nav className="w-64 bg-gray-900 text-white p-4">
      <div className="mb-8">
        <p className="font-bold">{user.name}</p>
        <p className="text-sm text-gray-400">{user.email}</p>
      </div>
      
      <ul className="space-y-2">
        {filteredItems.map(item => (
          <li key={item.href}>
            <Link
              href={item.href}
              className={`block px-4 py-2 rounded ${
                pathname === item.href
                  ? 'bg-blue-600'
                  : 'hover:bg-gray-800'
              }`}
            >
              {item.label}
            </Link>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

---

# 22. Testing Next.js Applications

## 22.1 Unit Testing Components

```tsx
// __tests__/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled</Button>);
    expect(screen.getByText('Disabled')).toBeDisabled();
  });
});
```

## 22.2 Testing Server Components

```tsx
// __tests__/app/products/page.test.tsx
import { render, screen } from '@testing-library/react';
import ProductsPage from '@/app/products/page';

// Mock data fetching
jest.mock('@/lib/data', () => ({
  getProducts: jest.fn().mockResolvedValue([
    { id: '1', name: 'Product 1', price: 99 },
    { id: '2', name: 'Product 2', price: 149 },
  ]),
}));

describe('ProductsPage', () => {
  it('renders products', async () => {
    render(await ProductsPage());
    
    expect(screen.getByText('Product 1')).toBeInTheDocument();
    expect(screen.getByText('Product 2')).toBeInTheDocument();
  });
});
```

## 22.3 Testing API Routes

```tsx
// __tests__/api/users.test.ts
import { GET, POST } from '@/app/api/users/route';
import { NextRequest } from 'next/server';

describe('GET /api/users', () => {
  it('returns users', async () => {
    const request = new NextRequest('http://localhost:3000/api/users');
    const response = await GET(request);
    const data = await response.json();
    
    expect(response.status).toBe(200);
    expect(Array.isArray(data)).toBe(true);
  });
});

describe('POST /api/users', () => {
  it('creates a user', async () => {
    const request = new NextRequest('http://localhost:3000/api/users', {
      method: 'POST',
      body: JSON.stringify({ name: 'John', email: 'john@example.com' }),
    });
    
    const response = await POST(request);
    const data = await response.json();
    
    expect(response.status).toBe(201);
    expect(data.name).toBe('John');
  });
});
```

## 22.4 E2E Testing with Playwright

```tsx
// e2e/navigation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Navigation', () => {
  test('navigates to about page', async ({ page }) => {
    await page.goto('/');
    await page.click('text=About');
    await expect(page).toHaveURL('/about');
    await expect(page.locator('h1')).toContainText('About');
  });
  
  test('shows login page for protected routes', async ({ page }) => {
    await page.goto('/dashboard');
    await expect(page).toHaveURL(/\/login/);
  });
});

// e2e/auth.spec.ts
test.describe('Authentication', () => {
  test('logs in successfully', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome')).toBeVisible();
  });
  
  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');
    
    await expect(page.locator('.error')).toContainText('Invalid credentials');
  });
});
```

---

# 23. More Interview Questions

## 23.1 Conceptual Questions

**Q11: What is the difference between `cache: 'force-cache'` and `cache: 'no-store'`?**

| Option | Behavior |
|--------|----------|
| `force-cache` | Response is cached indefinitely (default). Equivalent to SSG. |
| `no-store` | Response is never cached. Fresh request every time. Equivalent to SSR. |

---

**Q12: How do you handle errors in Next.js App Router?**

1. **error.tsx** - Catches errors in a route segment and its children
2. **global-error.tsx** - Catches errors in root layout
3. **notFound()** - Triggers not-found.tsx
4. **try/catch** - In Server Actions and API routes

```tsx
// app/products/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Error loading products</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

**Q13: What is partial prerendering?**

Partial Prerendering (experimental) combines static and dynamic rendering:
- Static shell is rendered at build time
- Dynamic parts are streamed in at request time
- Provides fast initial load with dynamic content

---

**Q14: How do you share data between Server Components?**

1. **Pass as props** - Simple and explicit
2. **fetch deduplication** - Same fetch URL is automatically deduplicated
3. **cache function** - Wrap function with React.cache()
4. **Database ORM caching** - Let your ORM handle it

```tsx
import { cache } from 'react';

export const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } });
});
```

---

**Q15: What are the benefits of the App Router over Pages Router?**

1. **Server Components** - Reduce client JS bundle
2. **Nested Layouts** - Preserve state across navigation
3. **Streaming** - Progressive page loading with Suspense
4. **Server Actions** - Direct server mutations without API routes
5. **Parallel/Intercepting Routes** - Advanced routing patterns
6. **Built-in Loading/Error States** - Declarative UI states

---

## 23.2 Scenario-Based Questions

**Q16: How would you implement real-time updates in Next.js?**

Options:
1. **Polling** - Fetch data on interval
2. **WebSockets** - Real-time bidirectional communication
3. **Server-Sent Events (SSE)** - One-way server to client
4. **React Query/SWR** - Smart polling with refetch on focus

```tsx
'use client';

import useSWR from 'swr';

function LiveData() {
  const { data } = useSWR('/api/data', fetcher, {
    refreshInterval: 5000, // Poll every 5 seconds
    revalidateOnFocus: true,
  });
  
  return <div>{data?.value}</div>;
}
```

---

**Q17: How would you implement internationalization (i18n)?**

```tsx
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

const locales = ['en', 'es', 'fr'];
const defaultLocale = 'en';

function getLocale(request: NextRequest): string {
  const negotiator = new Negotiator({
    headers: { 'accept-language': request.headers.get('accept-language') || '' },
  });
  const languages = negotiator.languages();
  return match(languages, locales, defaultLocale);
}

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;
  
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameHasLocale) return;
  
  const locale = getLocale(request);
  request.nextUrl.pathname = `/${locale}${pathname}`;
  
  return NextResponse.redirect(request.nextUrl);
}

// app/[locale]/page.tsx
import { getDictionary } from '@/lib/i18n';

export default async function Home({ 
  params: { locale } 
}: { 
  params: { locale: string } 
}) {
  const dict = await getDictionary(locale);
  
  return <h1>{dict.welcome}</h1>;
}
```

---

**Q18: How do you handle file uploads in Next.js?**

```tsx
// app/upload/page.tsx
import { uploadFile } from './actions';

export default function UploadPage() {
  return (
    <form action={uploadFile}>
      <input type="file" name="file" accept="image/*" />
      <button type="submit">Upload</button>
    </form>
  );
}

// app/upload/actions.ts
'use server';

import { writeFile } from 'fs/promises';
import { join } from 'path';

export async function uploadFile(formData: FormData) {
  const file = formData.get('file') as File;
  
  if (!file) {
    return { error: 'No file provided' };
  }
  
  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);
  
  const filename = `${Date.now()}-${file.name}`;
  const path = join(process.cwd(), 'public/uploads', filename);
  
  await writeFile(path, buffer);
  
  return { success: true, filename };
}
```

---

# 24. Performance Monitoring

## 24.1 Web Vitals

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}

// Custom web vitals reporting
// app/web-vitals.tsx
'use client';

import { useReportWebVitals } from 'next/web-vitals';

export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log(metric);
    
    // Send to analytics
    fetch('/api/vitals', {
      method: 'POST',
      body: JSON.stringify(metric),
    });
  });
  
  return null;
}
```

## 24.2 Bundle Analysis

```bash
# Install bundle analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your config
});

# Run analysis
ANALYZE=true npm run build
```

---

# 25. Migration Guide

## 25.1 Pages Router to App Router

```
Step 1: Create app/ directory alongside pages/
Step 2: Move _app.tsx logic to app/layout.tsx
Step 3: Convert pages one at a time:
  - pages/index.tsx → app/page.tsx
  - pages/about.tsx → app/about/page.tsx
  - pages/blog/[slug].tsx → app/blog/[slug]/page.tsx

Step 4: Convert data fetching:
  - getServerSideProps → async Server Component
  - getStaticProps → async Server Component with cache
  - getStaticPaths → generateStaticParams

Step 5: Convert API routes:
  - pages/api/users.ts → app/api/users/route.ts

Step 6: Update navigation:
  - useRouter from 'next/router' → 'next/navigation'
```

## 25.2 Example Migration

```tsx
// BEFORE: pages/blog/[slug].tsx
export async function getStaticPaths() {
  const posts = await getPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking',
  };
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug);
  return { props: { post }, revalidate: 60 };
}

export default function BlogPost({ post }) {
  return <article>{post.content}</article>;
}

// AFTER: app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map(p => ({ slug: p.slug }));
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  return <article>{post.content}</article>;
}

// Configure revalidation
export const revalidate = 60;
```

---

## 🎯 Full Study Checklist

### Fundamentals
- [ ] App Router structure and conventions
- [ ] File-based routing (dynamic, catch-all, groups)
- [ ] Layouts, templates, and pages
- [ ] Loading and error handling
- [ ] Server vs Client Components

### Data
- [ ] Server Component fetching
- [ ] Caching strategies
- [ ] Revalidation (time-based, on-demand)
- [ ] Server Actions

### Rendering
- [ ] Static vs Dynamic rendering
- [ ] Streaming with Suspense
- [ ] ISR and partial prerendering

### Advanced
- [ ] Middleware patterns
- [ ] Authentication strategies
- [ ] Route handlers (API routes)
- [ ] Internationalization

### Optimization
- [ ] Image, font, script optimization
- [ ] Metadata and SEO
- [ ] Bundle analysis
- [ ] Web Vitals

### Testing & Deployment
- [ ] Component testing
- [ ] E2E testing with Playwright
- [ ] Vercel deployment
- [ ] Docker deployment

---

# 26. More Coding Challenges

## 26.1 Implement Pagination Component

```tsx
// components/Pagination.tsx
'use client';

import Link from 'next/link';
import { usePathname, useSearchParams } from 'next/navigation';

interface PaginationProps {
  totalPages: number;
  currentPage: number;
}

export function Pagination({ totalPages, currentPage }: PaginationProps) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  const createPageURL = (pageNumber: number) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', pageNumber.toString());
    return `${pathname}?${params.toString()}`;
  };
  
  const pages = Array.from({ length: totalPages }, (_, i) => i + 1);
  
  return (
    <nav className="flex items-center gap-2">
      <Link
        href={createPageURL(currentPage - 1)}
        className={`px-3 py-1 rounded ${
          currentPage <= 1 
            ? 'pointer-events-none opacity-50' 
            : 'hover:bg-gray-100'
        }`}
        aria-disabled={currentPage <= 1}
      >
        Previous
      </Link>
      
      {pages.map(page => (
        <Link
          key={page}
          href={createPageURL(page)}
          className={`px-3 py-1 rounded ${
            page === currentPage
              ? 'bg-blue-500 text-white'
              : 'hover:bg-gray-100'
          }`}
        >
          {page}
        </Link>
      ))}
      
      <Link
        href={createPageURL(currentPage + 1)}
        className={`px-3 py-1 rounded ${
          currentPage >= totalPages 
            ? 'pointer-events-none opacity-50' 
            : 'hover:bg-gray-100'
        }`}
        aria-disabled={currentPage >= totalPages}
      >
        Next
      </Link>
    </nav>
  );
}

// app/products/page.tsx
import { Pagination } from '@/components/Pagination';

interface Props {
  searchParams: { page?: string };
}

export default async function ProductsPage({ searchParams }: Props) {
  const page = Number(searchParams.page) || 1;
  const { products, totalPages } = await getProducts({ page, limit: 10 });
  
  return (
    <div>
      <ProductGrid products={products} />
      <Pagination totalPages={totalPages} currentPage={page} />
    </div>
  );
}
```

## 26.2 Implement Theme Toggle

```tsx
// components/ThemeProvider.tsx
'use client';

import { createContext, useContext, useEffect, useState } from 'react';

type Theme = 'light' | 'dark' | 'system';

interface ThemeContext {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContext | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  
  useEffect(() => {
    const stored = localStorage.getItem('theme') as Theme;
    if (stored) setTheme(stored);
  }, []);
  
  useEffect(() => {
    const root = document.documentElement;
    
    if (theme === 'system') {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark'
        : 'light';
      root.classList.toggle('dark', systemTheme === 'dark');
    } else {
      root.classList.toggle('dark', theme === 'dark');
    }
    
    localStorage.setItem('theme', theme);
  }, [theme]);
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

// components/ThemeToggle.tsx
'use client';

import { useTheme } from './ThemeProvider';

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  
  return (
    <select
      value={theme}
      onChange={e => setTheme(e.target.value as 'light' | 'dark' | 'system')}
      className="px-2 py-1 rounded border"
    >
      <option value="light">Light</option>
      <option value="dark">Dark</option>
      <option value="system">System</option>
    </select>
  );
}
```

## 26.3 Implement Debounced Search

```tsx
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// components/SearchBox.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';
import { useDebounce } from '@/hooks/useDebounce';

export function SearchBox() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const [query, setQuery] = useState(searchParams.get('q') || '');
  const debouncedQuery = useDebounce(query, 300);
  
  useEffect(() => {
    const params = new URLSearchParams(searchParams);
    
    if (debouncedQuery) {
      params.set('q', debouncedQuery);
    } else {
      params.delete('q');
    }
    
    router.push(`/search?${params.toString()}`);
  }, [debouncedQuery, router, searchParams]);
  
  return (
    <input
      type="search"
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="Search..."
      className="w-full px-4 py-2 border rounded-lg"
    />
  );
}
```

## 26.4 Implement Multi-Step Form

```tsx
// components/MultiStepForm.tsx
'use client';

import { useState } from 'react';
import { submitMultiStepForm } from './actions';

interface FormData {
  // Step 1
  name: string;
  email: string;
  // Step 2
  company: string;
  role: string;
  // Step 3
  plan: 'basic' | 'pro' | 'enterprise';
  billing: 'monthly' | 'yearly';
}

const initialData: FormData = {
  name: '',
  email: '',
  company: '',
  role: '',
  plan: 'basic',
  billing: 'monthly',
};

export function MultiStepForm() {
  const [step, setStep] = useState(1);
  const [data, setData] = useState<FormData>(initialData);
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const totalSteps = 3;
  
  const updateData = (partial: Partial<FormData>) => {
    setData(prev => ({ ...prev, ...partial }));
  };
  
  const nextStep = () => setStep(s => Math.min(s + 1, totalSteps));
  const prevStep = () => setStep(s => Math.max(s - 1, 1));
  
  const handleSubmit = async () => {
    setIsSubmitting(true);
    try {
      await submitMultiStepForm(data);
      // Handle success
    } catch (error) {
      // Handle error
    }
    setIsSubmitting(false);
  };
  
  return (
    <div className="max-w-md mx-auto">
      {/* Progress Indicator */}
      <div className="flex justify-between mb-8">
        {[1, 2, 3].map(s => (
          <div
            key={s}
            className={`w-8 h-8 rounded-full flex items-center justify-center ${
              s <= step ? 'bg-blue-500 text-white' : 'bg-gray-200'
            }`}
          >
            {s}
          </div>
        ))}
      </div>
      
      {/* Step 1: Personal Info */}
      {step === 1 && (
        <div className="space-y-4">
          <h2 className="text-xl font-bold">Personal Information</h2>
          <input
            type="text"
            value={data.name}
            onChange={e => updateData({ name: e.target.value })}
            placeholder="Name"
            className="w-full px-4 py-2 border rounded"
          />
          <input
            type="email"
            value={data.email}
            onChange={e => updateData({ email: e.target.value })}
            placeholder="Email"
            className="w-full px-4 py-2 border rounded"
          />
        </div>
      )}
      
      {/* Step 2: Company Info */}
      {step === 2 && (
        <div className="space-y-4">
          <h2 className="text-xl font-bold">Company Information</h2>
          <input
            type="text"
            value={data.company}
            onChange={e => updateData({ company: e.target.value })}
            placeholder="Company"
            className="w-full px-4 py-2 border rounded"
          />
          <input
            type="text"
            value={data.role}
            onChange={e => updateData({ role: e.target.value })}
            placeholder="Role"
            className="w-full px-4 py-2 border rounded"
          />
        </div>
      )}
      
      {/* Step 3: Plan Selection */}
      {step === 3 && (
        <div className="space-y-4">
          <h2 className="text-xl font-bold">Select Plan</h2>
          <div className="grid grid-cols-3 gap-4">
            {(['basic', 'pro', 'enterprise'] as const).map(plan => (
              <button
                key={plan}
                onClick={() => updateData({ plan })}
                className={`p-4 border rounded ${
                  data.plan === plan ? 'border-blue-500 bg-blue-50' : ''
                }`}
              >
                {plan}
              </button>
            ))}
          </div>
          <div className="flex gap-4">
            <button
              onClick={() => updateData({ billing: 'monthly' })}
              className={`flex-1 p-2 border rounded ${
                data.billing === 'monthly' ? 'border-blue-500' : ''
              }`}
            >
              Monthly
            </button>
            <button
              onClick={() => updateData({ billing: 'yearly' })}
              className={`flex-1 p-2 border rounded ${
                data.billing === 'yearly' ? 'border-blue-500' : ''
              }`}
            >
              Yearly
            </button>
          </div>
        </div>
      )}
      
      {/* Navigation */}
      <div className="flex justify-between mt-8">
        <button
          onClick={prevStep}
          disabled={step === 1}
          className="px-4 py-2 border rounded disabled:opacity-50"
        >
          Back
        </button>
        
        {step < totalSteps ? (
          <button
            onClick={nextStep}
            className="px-4 py-2 bg-blue-500 text-white rounded"
          >
            Next
          </button>
        ) : (
          <button
            onClick={handleSubmit}
            disabled={isSubmitting}
            className="px-4 py-2 bg-green-500 text-white rounded disabled:opacity-50"
          >
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        )}
      </div>
    </div>
  );
}
```

## 26.5 Implement Toast Notifications

```tsx
// contexts/ToastContext.tsx
'use client';

import { createContext, useContext, useState, useCallback } from 'react';

type ToastType = 'success' | 'error' | 'warning' | 'info';

interface Toast {
  id: string;
  message: string;
  type: ToastType;
}

interface ToastContextValue {
  toasts: Toast[];
  addToast: (message: string, type?: ToastType) => void;
  removeToast: (id: string) => void;
}

const ToastContext = createContext<ToastContextValue | undefined>(undefined);

export function ToastProvider({ children }: { children: React.ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);
  
  const addToast = useCallback((message: string, type: ToastType = 'info') => {
    const id = Math.random().toString(36).substring(7);
    setToasts(prev => [...prev, { id, message, type }]);
    
    setTimeout(() => {
      setToasts(prev => prev.filter(t => t.id !== id));
    }, 5000);
  }, []);
  
  const removeToast = useCallback((id: string) => {
    setToasts(prev => prev.filter(t => t.id !== id));
  }, []);
  
  return (
    <ToastContext.Provider value={{ toasts, addToast, removeToast }}>
      {children}
      <ToastContainer toasts={toasts} removeToast={removeToast} />
    </ToastContext.Provider>
  );
}

export function useToast() {
  const context = useContext(ToastContext);
  if (!context) throw new Error('useToast must be used within ToastProvider');
  return context;
}

// components/ToastContainer.tsx
function ToastContainer({
  toasts,
  removeToast,
}: {
  toasts: Toast[];
  removeToast: (id: string) => void;
}) {
  const colorMap = {
    success: 'bg-green-500',
    error: 'bg-red-500',
    warning: 'bg-yellow-500',
    info: 'bg-blue-500',
  };
  
  return (
    <div className="fixed bottom-4 right-4 space-y-2 z-50">
      {toasts.map(toast => (
        <div
          key={toast.id}
          className={`${colorMap[toast.type]} text-white px-4 py-2 rounded shadow-lg flex items-center gap-2`}
        >
          <span>{toast.message}</span>
          <button
            onClick={() => removeToast(toast.id)}
            className="ml-2 text-white/80 hover:text-white"
          >
            ×
          </button>
        </div>
      ))}
    </div>
  );
}

// Usage
'use client';

import { useToast } from '@/contexts/ToastContext';

function SaveButton() {
  const { addToast } = useToast();
  
  const handleSave = async () => {
    try {
      await save();
      addToast('Saved successfully!', 'success');
    } catch {
      addToast('Failed to save', 'error');
    }
  };
  
  return <button onClick={handleSave}>Save</button>;
}
```

---

# 27. Output-Based Interview Questions

## 27.1 Question 1: What's the output?

```tsx
// app/page.tsx
export default async function Page() {
  console.log('Server: Rendering page');
  const data = await getData();
  console.log('Server: Data fetched');
  
  return (
    <div>
      <ClientComponent />
      <p>{data}</p>
    </div>
  );
}

// components/ClientComponent.tsx
'use client';

export default function ClientComponent() {
  console.log('Client: Rendering');
  return <p>Hello from client</p>;
}
```

**Answer:**
- Server console: "Server: Rendering page" → "Server: Data fetched"
- Client console: "Client: Rendering"

The Server Component runs on the server only, while ClientComponent runs on both server (during SSR) and client (during hydration).

---

## 27.2 Question 2: What's wrong with this code?

```tsx
// app/dashboard/page.tsx
'use client';

import { getUser } from '@/lib/db';

export default async function DashboardPage() {
  const user = await getUser();
  return <div>Welcome, {user.name}</div>;
}
```

**Answer:**
You cannot use `async/await` in a Client Component. Client Components cannot be async functions.

**Fix:** Either:
1. Remove `'use client'` to make it a Server Component
2. Use a separate Server Component for data fetching:

```tsx
// Server Component
import Dashboard from './Dashboard';

export default async function DashboardPage() {
  const user = await getUser();
  return <Dashboard user={user} />;
}

// Client Component
'use client';
export default function Dashboard({ user }) {
  return <div>Welcome, {user.name}</div>;
}
```

---

## 27.3 Question 3: Will this component re-render on navigation?

```tsx
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <nav>
          <Link href="/">Home</Link>
          <Link href="/about">About</Link>
        </nav>
        {children}
      </body>
    </html>
  );
}
```

**Answer:**
No, the layout component will NOT re-render when navigating between `/` and `/about`. Layouts in Next.js App Router are preserved across navigations - only the page content (`{children}`) changes.

---

## 27.4 Question 4: What's the difference?

```tsx
// Option 1
export default function Page() {
  return <Suspense fallback={<Loading />}>
    <SlowComponent />
  </Suspense>;
}

// Option 2
// loading.tsx in same folder
export default function Loading() {
  return <Loading />;
}

export default function Page() {
  return <SlowComponent />;
}
```

**Answer:**
Both achieve similar results, but:
- Option 1: More granular - only wraps SlowComponent
- Option 2: Wraps the entire page, including any other content

`loading.tsx` automatically wraps the `page.tsx` in a Suspense boundary.

---

# 28. Comparison Tables

## 28.1 Data Fetching Methods

| Method | When | Cache | Use Case |
|--------|------|-------|----------|
| Server Component + fetch | Build/Request | Configurable | Most data fetching |
| Server Action | Form submission | No | Mutations |
| API Route | External access | No | Webhooks, external API |
| Client fetch (SWR/RQ) | Client-side | Yes | Real-time updates |

## 28.2 Rendering Comparison

| Rendering | When | Benefits | Trade-offs |
|-----------|------|----------|------------|
| Static (SSG) | Build time | Fastest, cacheable | No personalization |
| Dynamic (SSR) | Request time | Fresh data | Slower TTFB |
| ISR | Background | Best of both | Stale windows |
| Client | Browser | Interactive | SEO, loading |
| Streaming | Progressive | Fast FCP | Complexity |

## 28.3 Cache Layers

| Layer | Scope | Duration | Clear |
|-------|-------|----------|-------|
| Request Memoization | Request | Single render | Automatic |
| Data Cache | Server | Persistent | revalidatePath/Tag |
| Full Route Cache | Server | Persistent | revalidatePath |
| Router Cache | Client | Session | router.refresh() |

## 28.4 Next.js vs Other Frameworks

| Feature | Next.js | Remix | Nuxt | SvelteKit |
|---------|---------|-------|------|-----------|
| React | ✅ | ✅ | ❌ Vue | ❌ Svelte |
| SSR | ✅ | ✅ | ✅ | ✅ |
| SSG | ✅ | Limited | ✅ | ✅ |
| ISR | ✅ | ❌ | ✅ | ❌ |
| Server Components | ✅ | ❌ | ❌ | ❌ |
| File Routing | ✅ | ✅ | ✅ | ✅ |
| API Routes | ✅ | ✅ | ✅ | ✅ |
| Edge Runtime | ✅ | ✅ | ✅ | ✅ |

---

# 29. Common Errors and Solutions

## 29.1 "Hydration mismatch"

**Cause:** Server and client render different content.

```tsx
// ❌ Wrong - Date is different on server and client
function Time() {
  return <p>{new Date().toLocaleTimeString()}</p>;
}

// ✅ Fix - Use useEffect for client-only values
function Time() {
  const [time, setTime] = useState('');
  
  useEffect(() => {
    setTime(new Date().toLocaleTimeString());
  }, []);
  
  return <p>{time}</p>;
}
```

## 29.2 "useState only works in Client Components"

**Cause:** Using hooks in Server Component.

```tsx
// ❌ Wrong - No 'use client' directive
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// ✅ Fix - Add 'use client'
'use client';

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

## 29.3 "Text content does not match"

**Cause:** Browser extensions or dynamic content.

```tsx
// ❌ Extensions add content
function Component() {
  return <p>Hello</p>;
}

// ✅ Fix - Suppress hydration warnings
function Component() {
  return <p suppressHydrationWarning>Hello</p>;
}
```

## 29.4 "Dynamic server usage: cookies"

**Cause:** Using dynamic functions in static route.

```tsx
// ❌ This forces dynamic rendering
export default async function Page() {
  const token = cookies().get('token'); // Dynamic!
  return <div>{token}</div>;
}

// ✅ If you need static, use middleware instead
// or declare dynamic explicitly
export const dynamic = 'force-dynamic';
```

---

# 30. Best Practices Summary

## 30.1 Component Organization

```
app/
├── (auth)/              # Group for auth pages
│   ├── login/
│   └── register/
├── (marketing)/         # Group for marketing
│   ├── about/
│   └── pricing/
├── dashboard/           # Protected area
│   ├── layout.tsx       # Dashboard layout
│   ├── page.tsx
│   └── settings/
├── api/                 # API routes
│   └── users/
│       └── route.ts
└── layout.tsx           # Root layout

components/
├── ui/                  # Generic UI components
│   ├── Button.tsx
│   └── Card.tsx
├── forms/               # Form components
│   ├── Input.tsx
│   └── Select.tsx
└── features/            # Feature-specific
    ├── auth/
    └── dashboard/

lib/
├── actions/             # Server actions
├── data/                # Data fetching
├── utils/               # Utility functions
└── validations/         # Zod schemas
```

## 30.2 Performance Checklist

- [ ] Use Server Components by default
- [ ] Minimize `'use client'` directives
- [ ] Use `next/image` for images
- [ ] Use `next/font` for fonts
- [ ] Implement proper caching strategies
- [ ] Use Suspense for streaming
- [ ] Optimize third-party scripts
- [ ] Monitor Core Web Vitals
- [ ] Analyze bundle size regularly

## 30.3 Security Checklist

- [ ] Never expose secrets to client
- [ ] Validate all inputs on server
- [ ] Use CSRF protection in forms
- [ ] Implement rate limiting
- [ ] Set proper security headers
- [ ] Validate user permissions in Server Actions
- [ ] Use parameterized database queries
- [ ] Enable strict CSP headers

---

## 🎯 Quick Reference Card

```
# App Router Structure
app/
├── layout.tsx        # Root layout (required)
├── page.tsx          # Home page (/)
├── loading.tsx       # Loading state
├── error.tsx         # Error boundary
├── not-found.tsx     # 404 page
├── route.ts          # API route
└── [dynamic]/        # Dynamic segment

# Route Config Exports
export const dynamic = 'force-dynamic' | 'force-static' | 'error' | 'auto'
export const revalidate = 60  // seconds
export const runtime = 'edge' | 'nodejs'

# Navigation
import Link from 'next/link'
import { useRouter, usePathname, useSearchParams } from 'next/navigation'
import { redirect, notFound } from 'next/navigation'

# Data Fetching
fetch(url, { cache: 'force-cache' })  // Static
fetch(url, { cache: 'no-store' })     // Dynamic
fetch(url, { next: { revalidate: 60 } })  // ISR
fetch(url, { next: { tags: ['tag'] } })   // Tagged

# Revalidation
import { revalidatePath, revalidateTag } from 'next/cache'

# Server Actions
'use server'
async function action(formData: FormData) { }

# Dynamic Functions (trigger dynamic rendering)
import { cookies, headers } from 'next/headers'
```

---

# 31. Next.js 14/15 Features

## 31.1 Partial Prerendering (Experimental)

```tsx
// app/page.tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <main>
      {/* Static shell - prerendered at build time */}
      <header>
        <h1>Welcome to our Store</h1>
        <nav>...</nav>
      </header>
      
      {/* Dynamic content - streamed at request time */}
      <Suspense fallback={<CartSkeleton />}>
        <Cart /> {/* Uses cookies(), dynamic */}
      </Suspense>
      
      {/* More static content */}
      <footer>© 2024</footer>
    </main>
  );
}

// next.config.js
module.exports = {
  experimental: {
    ppr: true, // Enable Partial Prerendering
  },
};
```

## 31.2 Server Actions Improvements

```tsx
// Inline server action with useOptimistic
'use client';

import { useOptimistic, useTransition } from 'react';

export function TodoList({ initialTodos }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticTodos, addOptimistic] = useOptimistic(
    initialTodos,
    (state, newTodo) => [...state, { id: crypto.randomUUID(), ...newTodo }]
  );
  
  async function addTodo(formData: FormData) {
    const text = formData.get('text') as string;
    startTransition(async () => {
      addOptimistic({ text, completed: false });
      await createTodo(text);
    });
  }
  
  return (
    <div>
      <form action={addTodo}>
        <input name="text" required />
        <button disabled={isPending}>Add</button>
      </form>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 31.3 Turbopack (Next.js 15)

```bash
# Enable Turbopack for development
next dev --turbo

# next.config.js - No configuration needed in Next.js 15
# Turbopack is automatically used when --turbo flag is passed
```

Benefits:
- Up to 5000x faster cold starts
- Up to 10x faster updates
- Incremental computation
- Native TypeScript/JSX support

## 31.4 Improved Metadata API

```tsx
// Static metadata with all options
export const metadata = {
  title: {
    default: 'My App',
    template: '%s | My App',
  },
  description: 'The best app ever',
  metadataBase: new URL('https://example.com'),
  alternates: {
    canonical: '/',
    languages: {
      'en-US': '/en-US',
      'es-ES': '/es-ES',
    },
  },
  openGraph: {
    title: 'My App',
    description: 'The best app ever',
    url: 'https://example.com',
    siteName: 'My App',
    images: [
      {
        url: '/og.png',
        width: 1200,
        height: 630,
        alt: 'My App',
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My App',
    description: 'The best app ever',
    creator: '@myapp',
    images: ['/twitter-image.png'],
  },
  robots: {
    index: true,
    follow: true,
    nocache: true,
    googleBot: {
      index: true,
      follow: false,
      noimageindex: true,
    },
  },
  icons: {
    icon: '/icon.png',
    shortcut: '/shortcut-icon.png',
    apple: '/apple-icon.png',
    other: {
      rel: 'apple-touch-icon-precomposed',
      url: '/apple-touch-icon.png',
    },
  },
  verification: {
    google: 'google-site-verification=...',
    yandex: 'yandex-verification=...',
  },
};

// File-based metadata
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og';

export const runtime = 'edge';

export const alt = 'My App';
export const size = { width: 1200, height: 630 };
export const contentType = 'image/png';

export default async function Image() {
  return new ImageResponse(
    <div style={{ display: 'flex', fontSize: 128, background: 'white' }}>
      Hello World!
    </div>,
    { ...size }
  );
}
```

---

# 32. Edge Runtime

## 32.1 Edge API Routes

```tsx
// app/api/hello/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const name = searchParams.get('name') || 'World';
  
  return new Response(`Hello, ${name}!`, {
    headers: { 'Content-Type': 'text/plain' },
  });
}
```

## 32.2 Edge Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

// Runs at the edge, close to users
export function middleware(request: NextRequest) {
  const country = request.geo?.country || 'US';
  
  // Rewrite based on location
  if (country === 'UK') {
    return NextResponse.rewrite(new URL('/uk', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)',
};
```

## 32.3 Edge vs Node.js Runtime

| Feature | Edge | Node.js |
|---------|------|---------|
| Cold Start | ~0ms | 250ms+ |
| Max Duration | 30s (Vercel) | 5min (Vercel) |
| Memory | 128MB | 1024MB+ |
| Node APIs | Limited | Full |
| File System | ❌ | ✅ |
| Database | Limited | ✅ |
| WebAssembly | ✅ | ✅ |
| Crypto | Partial | ✅ |

```tsx
// Force Node.js runtime for full API access
export const runtime = 'nodejs';

// Force Edge runtime for faster cold starts
export const runtime = 'edge';
```

---

# 33. Internationalization Deep Dive

## 33.1 Complete i18n Setup

```tsx
// i18n/config.ts
export const locales = ['en', 'es', 'fr', 'de', 'ja'] as const;
export type Locale = (typeof locales)[number];
export const defaultLocale: Locale = 'en';

// i18n/dictionaries.ts
const dictionaries = {
  en: () => import('./dictionaries/en.json').then(m => m.default),
  es: () => import('./dictionaries/es.json').then(m => m.default),
  fr: () => import('./dictionaries/fr.json').then(m => m.default),
  de: () => import('./dictionaries/de.json').then(m => m.default),
  ja: () => import('./dictionaries/ja.json').then(m => m.default),
};

export const getDictionary = async (locale: Locale) => {
  return dictionaries[locale]();
};

// dictionaries/en.json
{
  "nav": {
    "home": "Home",
    "about": "About",
    "contact": "Contact"
  },
  "home": {
    "title": "Welcome",
    "description": "This is our app"
  },
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "submit": "Submit"
  }
}
```

## 33.2 Locale-aware Pages

```tsx
// app/[locale]/layout.tsx
import { locales, type Locale } from '@/i18n/config';

export async function generateStaticParams() {
  return locales.map(locale => ({ locale }));
}

export default async function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode;
  params: { locale: Locale };
}) {
  return (
    <html lang={locale}>
      <body>{children}</body>
    </html>
  );
}

// app/[locale]/page.tsx
import { getDictionary, type Locale } from '@/i18n/config';

export default async function HomePage({
  params: { locale },
}: {
  params: { locale: Locale };
}) {
  const dict = await getDictionary(locale);
  
  return (
    <main>
      <h1>{dict.home.title}</h1>
      <p>{dict.home.description}</p>
    </main>
  );
}
```

## 33.3 Language Switcher

```tsx
'use client';

import { usePathname, useRouter } from 'next/navigation';
import { locales, type Locale } from '@/i18n/config';

export function LanguageSwitcher({ currentLocale }: { currentLocale: Locale }) {
  const router = useRouter();
  const pathname = usePathname();
  
  const handleChange = (newLocale: Locale) => {
    // Remove current locale from pathname
    const segments = pathname.split('/');
    segments[1] = newLocale;
    router.push(segments.join('/'));
  };
  
  return (
    <select
      value={currentLocale}
      onChange={e => handleChange(e.target.value as Locale)}
    >
      {locales.map(locale => (
        <option key={locale} value={locale}>
          {locale.toUpperCase()}
        </option>
      ))}
    </select>
  );
}
```

---

# 34. Advanced Data Patterns

## 34.1 Request Waterfall Prevention

```tsx
// ❌ Bad: Sequential requests (waterfall)
async function BadPage() {
  const user = await getUser();        // 200ms
  const posts = await getPosts();       // 300ms
  const comments = await getComments(); // 200ms
  // Total: 700ms
}

// ✅ Good: Parallel requests
async function GoodPage() {
  const [user, posts, comments] = await Promise.all([
    getUser(),      // 200ms
    getPosts(),     // 300ms ⎤
    getComments(),  // 200ms ⎦ parallel
  ]);
  // Total: 300ms (largest single request)
}

// ✅ Better: Streaming with Suspense
function BestPage() {
  return (
    <div>
      {/* Each section loads independently */}
      <Suspense fallback={<UserSkeleton />}>
        <User />
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />
      </Suspense>
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />
      </Suspense>
    </div>
  );
}
```

## 34.2 Prefetching Patterns

```tsx
// Link prefetching (automatic on hover)
<Link href="/dashboard" prefetch={true}>
  Dashboard
</Link>

// Disable prefetching for seldom-used links
<Link href="/settings" prefetch={false}>
  Settings
</Link>

// Programmatic prefetching
'use client';

import { useRouter } from 'next/navigation';

function SearchResults({ results }) {
  const router = useRouter();
  
  // Prefetch first few results on render
  useEffect(() => {
    results.slice(0, 3).forEach(result => {
      router.prefetch(`/product/${result.id}`);
    });
  }, [results, router]);
  
  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>
          <Link href={`/product/${result.id}`}>{result.name}</Link>
        </li>
      ))}
    </ul>
  );
}
```

## 34.3 Optimistic Updates with Revalidation

```tsx
'use server';

import { revalidateTag } from 'next/cache';

async function getCart() {
  return fetch('/api/cart', { next: { tags: ['cart'] } });
}

async function addToCart(productId: string) {
  'use server';
  
  await db.cart.add(productId);
  revalidateTag('cart'); // Refresh all cart data
}

// Client component with optimistic UI
'use client';

export function AddToCartButton({ productId }) {
  const [isPending, startTransition] = useTransition();
  const [isAdded, setIsAdded] = useState(false);
  
  return (
    <button
      disabled={isPending || isAdded}
      onClick={() => {
        setIsAdded(true); // Optimistic
        startTransition(async () => {
          try {
            await addToCart(productId);
          } catch {
            setIsAdded(false); // Rollback on error
          }
        });
      }}
    >
      {isAdded ? '✓ Added' : isPending ? 'Adding...' : 'Add to Cart'}
    </button>
  );
}
```

---

# 35. Production Checklist

## 35.1 Pre-Deployment Checklist

```markdown
## Performance
- [ ] Lighthouse score 90+ on all pages
- [ ] Core Web Vitals passing
- [ ] Images optimized with next/image
- [ ] Fonts loaded with next/font
- [ ] JavaScript bundle under 200KB
- [ ] Third-party scripts deferred

## SEO
- [ ] Metadata on all pages
- [ ] OpenGraph images generated
- [ ] Sitemap configured
- [ ] robots.txt configured
- [ ] Canonical URLs set
- [ ] Structured data added

## Security
- [ ] Environment variables secured
- [ ] Authentication tested
- [ ] CSRF protection enabled
- [ ] Security headers configured
- [ ] Input validation on server
- [ ] Rate limiting implemented

## Accessibility
- [ ] WCAG 2.1 AA compliant
- [ ] Keyboard navigation works
- [ ] Screen reader tested
- [ ] Color contrast sufficient
- [ ] Form labels present
- [ ] Skip links added

## Testing
- [ ] Unit tests passing
- [ ] Integration tests passing
- [ ] E2E tests passing
- [ ] Error boundaries tested
- [ ] 404/500 pages tested
- [ ] Loading states tested

## Monitoring
- [ ] Error tracking configured (Sentry)
- [ ] Analytics configured
- [ ] Web Vitals reporting
- [ ] Uptime monitoring
- [ ] Log aggregation
```

## 35.2 next.config.js Production Setup

```js
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Output standalone for Docker
  output: 'standalone',
  
  // Enable React Strict Mode
  reactStrictMode: true,
  
  // Disable powered by header
  poweredByHeader: false,
  
  // Enable compression
  compress: true,
  
  // Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    remotePatterns: [
      { hostname: 'images.example.com' },
    ],
    minimumCacheTTL: 60 * 60 * 24, // 24 hours
  },
  
  // Security headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on',
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=()',
          },
        ],
      },
    ];
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },
  
  // Rewrites (API proxy)
  async rewrites() {
    return {
      beforeFiles: [
        {
          source: '/api/external/:path*',
          destination: 'https://external-api.com/:path*',
        },
      ],
    };
  },
};

module.exports = withBundleAnalyzer(nextConfig);
```

---

# 36. Troubleshooting Guide

## 36.1 Build Errors

**Error: "Build optimization failed"**
```bash
# Check for circular dependencies
npx madge --circular src/

# Analyze bundle
ANALYZE=true npm run build
```

**Error: "Module not found"**
```bash
# Clear cache and reinstall
rm -rf .next node_modules
npm install
npm run build
```

**Error: "Out of memory"**
```bash
# Increase Node memory
NODE_OPTIONS='--max-old-space-size=4096' npm run build
```

## 36.2 Runtime Errors

**Error: "Hydration mismatch"**
- Check for client-only code (dates, random, localStorage)
- Use `useEffect` for dynamic values
- Add `suppressHydrationWarning` if needed

**Error: "Fetch failed"**
- Check API URL in environment variables
- Verify network/CORS configuration
- Check for rate limiting

**Error: "Invariant: static generation store missing"**
- Don't use dynamic functions in static routes
- Add `export const dynamic = 'force-dynamic'`

## 36.3 Performance Issues

```tsx
// Identify slow components
import { unstable_noStore as noStore } from 'next/cache';

async function SlowComponent() {
  console.time('SlowComponent');
  noStore(); // Opt out of caching for debugging
  const data = await fetchData();
  console.timeEnd('SlowComponent');
  return <div>{data}</div>;
}

// Profile with React DevTools Profiler
// Enable in development
// Use next/dynamic for code splitting
```

---

# 37. Resources and Learning Path

## 37.1 Official Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)
- [Next.js Blog](https://nextjs.org/blog)
- [Vercel Templates](https://vercel.com/templates)
- [Next.js GitHub](https://github.com/vercel/next.js)

## 37.2 Community Resources

- React Server Components RFC
- App Router Migration Guide
- Performance Best Practices
- Security Guidelines
- Accessibility Checklist

## 37.3 Learning Path

```
Beginner:
1. Complete Next.js Learn tutorial
2. Build a simple blog
3. Learn file-based routing
4. Understand data fetching basics

Intermediate:
5. Master Server vs Client Components
6. Learn caching strategies
7. Implement authentication
8. Build API routes

Advanced:
9. Optimize performance
10. Implement ISR and streaming
11. Build complex layouts
12. Master deployment strategies
```

---

## 🎯 Ultimate Study Checklist

### Week 1: Fundamentals
- [ ] App Router structure
- [ ] File conventions (layout, page, loading, error)
- [ ] Server Components basics
- [ ] Client Components basics
- [ ] Basic routing

### Week 2: Data & Rendering
- [ ] Data fetching in Server Components
- [ ] Caching strategies
- [ ] Revalidation methods
- [ ] Server Actions
- [ ] Streaming with Suspense

### Week 3: Advanced Features
- [ ] Middleware
- [ ] Route Handlers (API)
- [ ] Authentication patterns
- [ ] Internationalization
- [ ] Parallel/Intercepting Routes

### Week 4: Optimization & Deployment
- [ ] Image optimization
- [ ] Font optimization
- [ ] Bundle analysis
- [ ] Core Web Vitals
- [ ] Deployment strategies

---

# 38. Quick Answers for Interviews

## 38.1 One-Liner Definitions

| Concept | Definition |
|---------|------------|
| Server Component | Component that renders on server, sends HTML to client |
| Client Component | Component with `'use client'`, runs in browser |
| Server Action | `async` function with `'use server'`, runs on server from client triggers |
| Route Handler | API endpoint in App Router using `route.ts` |
| Middleware | Code that runs before every request |
| ISR | Incremental Static Regeneration - update static pages after build |
| Streaming | Progressive rendering using Suspense boundaries |
| Edge Runtime | Lightweight runtime for faster cold starts |
| Partial Prerendering | Static shell + dynamic holes rendering |
| React Server Components | Components that only run on server, never hydrate |

## 38.2 When To Use What

```
Server Component when:
✓ Fetching data
✓ Accessing backend resources
✓ Keeping sensitive info on server
✓ Large dependencies
✓ No interactivity needed

Client Component when:
✓ useState, useEffect needed
✓ Event handlers (onClick, onChange)
✓ Browser APIs (window, localStorage)
✓ Third-party React libraries
✓ Real-time updates
```

## 38.3 Caching Quick Reference

```
// Never cache (SSR)
fetch(url, { cache: 'no-store' })

// Cache forever (SSG)
fetch(url, { cache: 'force-cache' })

// Revalidate after N seconds (ISR)
fetch(url, { next: { revalidate: 60 } })

// Tag for on-demand revalidation
fetch(url, { next: { tags: ['products'] } })

// Clear cache
revalidatePath('/products')
revalidateTag('products')
```

---

# 39. More Interview Questions & Answers

## Q19: What is the difference between `layout.tsx` and `template.tsx`?

**Answer:**
- **layout.tsx**: Persisted across navigations, state preserved
- **template.tsx**: New instance on each navigation, state reset

```tsx
// layout.tsx - State persists
export default function Layout({ children }) {
  return <div>{children}</div>; // Same instance on /a and /b
}

// template.tsx - State resets
export default function Template({ children }) {
  return <div>{children}</div>; // New instance on every navigation
}
```

---

## Q20: How do you handle environment variables in Next.js?

**Answer:**
```bash
# Server-side only (safe for secrets)
DATABASE_URL="postgresql://..."
API_SECRET="my-secret"

# Client-side accessible (prefixed with NEXT_PUBLIC_)
NEXT_PUBLIC_API_URL="https://api.example.com"
```

```tsx
// Server Component - access all
const dbUrl = process.env.DATABASE_URL;

// Client Component - only NEXT_PUBLIC_*
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

---

## Q21: What triggers dynamic rendering?

**Answer:**
Dynamic rendering is triggered when using:
- `cookies()` or `headers()`
- `searchParams` prop
- `fetch` with `{ cache: 'no-store' }`
- Route with `export const dynamic = 'force-dynamic'`

---

## Q22: How does `generateStaticParams` work?

**Answer:**
```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();
  
  // Returns list of params to pre-render at build time
  return posts.map(post => ({
    slug: post.slug,
  }));
}

// This creates static pages for:
// /blog/hello-world
// /blog/next-js-guide
// etc.
```

---

## Q23: What is request memoization?

**Answer:**
React automatically deduplicates identical fetch requests during a single render:

```tsx
// These only make ONE actual request
async function ComponentA() {
  const data = await fetch('/api/data');
}

async function ComponentB() {
  const data = await fetch('/api/data'); // Reuses ComponentA's request
}
```

---

## Q24: How do you implement loading UI?

**Answer:**
Three approaches:

```tsx
// 1. loading.tsx (automatic Suspense boundary)
// app/dashboard/loading.tsx
export default function Loading() {
  return <Spinner />;
}

// 2. Manual Suspense
<Suspense fallback={<Spinner />}>
  <SlowComponent />
</Suspense>

// 3. useTransition for client-side
const [isPending, startTransition] = useTransition();
```

---

## Q25: What is the Router Cache?

**Answer:**
Client-side cache that stores visited routes for instant back/forward navigation:

- Prefetched routes cached for 30 seconds
- Navigated routes cached for 5 minutes
- Clear with `router.refresh()`

---

# 40. Next.js Cheat Sheets

## 40.1 File Conventions Cheat Sheet

```
app/
├── layout.tsx        # Shared UI wrapper
├── page.tsx          # Route segment UI
├── loading.tsx       # Loading UI (Suspense fallback)
├── error.tsx         # Error UI (Error boundary)
├── not-found.tsx     # 404 UI
├── route.ts          # API endpoint
├── template.tsx      # Per-navigation wrapper
├── default.tsx       # Default for parallel routes
├── global-error.tsx  # Root error handler
├── opengraph-image.tsx  # OG image generation
├── twitter-image.tsx    # Twitter image
├── icon.tsx            # Favicon generation
├── sitemap.ts          # Sitemap generation
├── robots.ts           # robots.txt generation
└── manifest.ts         # Web manifest
```

## 40.2 Routing Cheat Sheet

```
Static Route:
  app/about/page.tsx         →  /about

Dynamic Route:
  app/blog/[slug]/page.tsx   →  /blog/:slug

Catch-all:
  app/docs/[...slug]/page.tsx  →  /docs/a, /docs/a/b, etc.

Optional Catch-all:
  app/[[...slug]]/page.tsx   →  /, /a, /a/b, etc.

Route Groups:
  app/(marketing)/about/     →  /about (no 'marketing' in URL)

Parallel Routes:
  app/@modal/page.tsx        →  Renders alongside main content

Intercepting Routes:
  app/(.)photos/[id]/        →  Intercepts /photos/:id
```

## 40.3 Data Fetching Cheat Sheet

```tsx
// Server Component fetching
async function Page() {
  const data = await fetch(url, options);
  return <div>{data}</div>;
}

// Cache options
{ cache: 'force-cache' }      // Static (default)
{ cache: 'no-store' }         // Dynamic
{ next: { revalidate: 60 } }  // ISR (60 seconds)
{ next: { tags: ['tag'] } }   // Tagged cache

// Server Action
'use server';
async function action(formData: FormData) { }

// Route config
export const dynamic = 'force-dynamic';
export const revalidate = 60;
export const runtime = 'edge';
```

## 40.4 Navigation Cheat Sheet

```tsx
// Link component
import Link from 'next/link';
<Link href="/about">About</Link>
<Link href="/blog/123">Post</Link>
<Link href="/search?q=hello">Search</Link>
<Link href="/page" prefetch={false}>No prefetch</Link>
<Link href="/page" replace>Replace history</Link>

// useRouter (Client Component)
import { useRouter } from 'next/navigation';
router.push('/dashboard');    // Navigate
router.replace('/login');     // Replace
router.back();                // Go back
router.forward();             // Go forward
router.refresh();             // Refresh server components
router.prefetch('/page');     // Prefetch

// Server-side navigation
import { redirect, notFound } from 'next/navigation';
redirect('/login');           // Redirect
notFound();                   // Show 404
```

## 40.5 Metadata Cheat Sheet

```tsx
// Static metadata
export const metadata = {
  title: 'Page Title',
  description: 'Page description',
  openGraph: {
    title: 'OG Title',
    description: 'OG Description',
    images: ['/og.png'],
  },
};

// Dynamic metadata
export async function generateMetadata({ params }) {
  const post = await getPost(params.id);
  return {
    title: post.title,
    description: post.excerpt,
  };
}

// Title template
export const metadata = {
  title: {
    default: 'My Site',
    template: '%s | My Site',
  },
};
```

---

# 41. Common Patterns Implementation

## 41.1 Authentication Pattern

```tsx
// lib/auth.ts
export async function getSession() {
  const cookieStore = cookies();
  const token = cookieStore.get('token')?.value;
  if (!token) return null;
  return verifyToken(token);
}

export async function requireAuth() {
  const session = await getSession();
  if (!session) redirect('/login');
  return session;
}

// app/dashboard/page.tsx
export default async function DashboardPage() {
  const session = await requireAuth();
  return <Dashboard user={session.user} />;
}
```

## 41.2 Data Revalidation Pattern

```tsx
// actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function createPost(data: FormData) {
  await db.post.create({ data });
  
  // Revalidate specific path
  revalidatePath('/blog');
  
  // Revalidate by tag
  revalidateTag('posts');
}

// Fetch with tags
const posts = await fetch('/api/posts', {
  next: { tags: ['posts'] }
});
```

## 41.3 Error Handling Pattern

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log to error reporting service
    logError(error);
  }, [error]);
  
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## 41.4 Modal Pattern with Intercepting Routes

```
app/
├── photos/
│   └── [id]/
│       └── page.tsx          # Full photo page
├── @modal/
│   └── (.)photos/
│       └── [id]/
│           └── page.tsx      # Photo modal
└── layout.tsx                # Renders both
```

```tsx
// app/layout.tsx
export default function Layout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <>
      {children}
      {modal}
    </>
  );
}
```

---

# 42. Performance Tips

## 42.1 Reduce Client Bundle

```tsx
// ❌ Bad: Entire library in client bundle
'use client';
import moment from 'moment';

function DateComponent({ date }) {
  return <span>{moment(date).format('LLL')}</span>;
}

// ✅ Good: Format on server
function DateComponent({ date }) {
  return <span>{new Date(date).toLocaleDateString()}</span>;
}
```

## 42.2 Optimize Images

```tsx
// ✅ Always use next/image
import Image from 'next/image';

<Image
  src="/photo.jpg"
  alt="Description"
  width={800}
  height={600}
  priority={isAboveFold}
  placeholder="blur"
  blurDataURL={blurUrl}
/>
```

## 42.3 Lazy Load Components

```tsx
import dynamic from 'next/dynamic';

// Only load when needed
const Chart = dynamic(() => import('./Chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,
});
```

## 42.4 Preload Key Resources

```tsx
// Link preload
<Link href="/dashboard" prefetch>Dashboard</Link>

// DNS prefetch
<link rel="dns-prefetch" href="https://api.example.com" />

// Preconnect
<link rel="preconnect" href="https://fonts.googleapis.com" />
```

---

# Final Interview Preparation Tips

## Do's
- ✅ Understand Server vs Client Components deeply
- ✅ Know when to use each rendering strategy
- ✅ Practice explaining caching behavior
- ✅ Be ready to explain App Router file conventions
- ✅ Know the difference between Pages Router and App Router

## Don'ts
- ❌ Don't confuse `useRouter` from `next/navigation` vs `next/router`
- ❌ Don't use `'use client'` unnecessarily
- ❌ Don't ignore cache invalidation strategies
- ❌ Don't forget about error boundaries
- ❌ Don't overlook accessibility

## Key Topics to Master
1. **Server Components**: When/why to use, limitations
2. **Data Fetching**: Cache strategies, revalidation
3. **Server Actions**: Forms, mutations, error handling
4. **Streaming**: Suspense, progressive loading
5. **Middleware**: Authentication, redirects, i18n
6. **Optimization**: Images, fonts, bundles

---

# 43. Additional Scenario Questions

## Scenario 1: E-commerce Checkout Flow

**Q: How would you implement a multi-step checkout in Next.js?**

```tsx
// app/checkout/layout.tsx
export default function CheckoutLayout({ children }) {
  return (
    <div className="checkout-container">
      <CheckoutProgress />
      {children}
    </div>
  );
}

// app/checkout/cart/page.tsx
// app/checkout/shipping/page.tsx  
// app/checkout/payment/page.tsx
// app/checkout/confirmation/page.tsx

// Use URL state for step management
// Use Server Actions for mutations
// Use middleware for auth protection
```

---

## Scenario 2: Real-time Dashboard

**Q: How would you implement real-time data updates?**

```tsx
// Option 1: Polling with SWR
'use client';
import useSWR from 'swr';

function Dashboard() {
  const { data } = useSWR('/api/metrics', fetcher, {
    refreshInterval: 5000,
  });
  return <MetricsDisplay data={data} />;
}

// Option 2: Server Actions with revalidation
'use client';
function RefreshButton() {
  return (
    <button onClick={() => router.refresh()}>
      Refresh Data
    </button>
  );
}
```

---

## Scenario 3: SEO Optimization

**Q: How do you optimize SEO for a blog platform?**

```tsx
// 1. Dynamic metadata
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
    },
  };
}

// 2. Sitemap generation
// app/sitemap.ts
export default async function sitemap() {
  const posts = await getPosts();
  return posts.map(post => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8,
  }));
}

// 3. Structured data
<script type="application/ld+json">
  {JSON.stringify(structuredData)}
</script>
```

---

## Scenario 4: Multi-tenant Application

**Q: How would you build a multi-tenant SaaS app?**

```tsx
// middleware.ts
export function middleware(request: NextRequest) {
  const hostname = request.headers.get('host');
  const subdomain = hostname?.split('.')[0];
  
  // Rewrite to tenant-specific path
  request.nextUrl.pathname = `/${subdomain}${request.nextUrl.pathname}`;
  return NextResponse.rewrite(request.nextUrl);
}

// app/[tenant]/page.tsx
export default async function TenantPage({ params }) {
  const tenant = await getTenant(params.tenant);
  return <TenantDashboard tenant={tenant} />;
}
```

---

# 44. Quick Debug Reference

## Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Hydration error | Server/client mismatch | Use `useEffect` for client-only values |
| Stale data | Cache not invalidating | Use `revalidatePath/Tag` after mutations |
| Slow page load | Large client bundle | Move code to Server Components |
| 404 on dynamic route | Missing `generateStaticParams` | Add function for static generation |
| API not working | Wrong HTTP method export | Export `GET`, `POST`, etc. as named exports |
| Middleware not running | Wrong matcher config | Check `config.matcher` pattern |
| Cookies not setting | Missing secure flags | Add `httpOnly`, `secure`, `sameSite` |

---

# 45. Version Differences Cheat Sheet

## Next.js 12 → 13

| Next.js 12 | Next.js 13+ |
|------------|-------------|
| `pages/` directory | `app/` directory |
| `getServerSideProps` | `async` Server Components |
| `getStaticProps` | `fetch` with cache |
| `_app.tsx` | `layout.tsx` |
| `_document.tsx` | Root `layout.tsx` |
| API `pages/api/*` | Route handlers `app/api/*/route.ts` |

## Next.js 13 → 14

| Next.js 13 | Next.js 14 |
|------------|------------|
| `useFormState` | `useActionState` |
| Experimental Server Actions | Stable Server Actions |
| Basic metadata | Enhanced metadata API |

## Next.js 14 → 15

| Next.js 14 | Next.js 15 |
|------------|------------|
| Webpack (default) | Turbopack (default for dev) |
| React 18 | React 19 |
| `useFormState` | `useActionState` |

---

# 46. Final Code Samples

## 46.1 Complete Page Example

```tsx
// app/products/[id]/page.tsx
import { Suspense } from 'react';
import { notFound } from 'next/navigation';
import { Metadata } from 'next';

interface Props {
  params: { id: string };
}

// Static params generation
export async function generateStaticParams() {
  const products = await getProducts();
  return products.map(p => ({ id: p.id }));
}

// Dynamic metadata
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const product = await getProduct(params.id);
  if (!product) return {};
  
  return {
    title: product.name,
    description: product.description,
    openGraph: { images: [product.image] },
  };
}

// Page component
export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id);
  
  if (!product) {
    notFound();
  }
  
  return (
    <main className="product-page">
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews productId={params.id} />
      </Suspense>
    </main>
  );
}
```

## 46.2 Complete Form with Server Action

```tsx
// app/contact/page.tsx
import { ContactForm } from './form';
import { submitContact } from './actions';

export default function ContactPage() {
  return (
    <main>
      <h1>Contact Us</h1>
      <ContactForm action={submitContact} />
    </main>
  );
}

// app/contact/form.tsx
'use client';

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Sending...' : 'Send Message'}
    </button>
  );
}

export function ContactForm({ action }) {
  const [state, formAction] = useActionState(action, { message: '' });
  
  return (
    <form action={formAction}>
      <input name="name" required placeholder="Your name" />
      <input name="email" type="email" required placeholder="Email" />
      <textarea name="message" required placeholder="Message" />
      <SubmitButton />
      {state.message && <p>{state.message}</p>}
    </form>
  );
}

// app/contact/actions.ts
'use server';

export async function submitContact(prevState: any, formData: FormData) {
  const data = {
    name: formData.get('name'),
    email: formData.get('email'),
    message: formData.get('message'),
  };
  
  // Validate and send
  await sendEmail(data);
  
  return { message: 'Message sent successfully!' };
}
```

---

## Summary: Next.js Interview Preparation

This comprehensive guide covered:

1. **App Router Fundamentals** - File conventions, layouts, pages
2. **Routing** - Dynamic, catch-all, parallel, intercepting routes
3. **Server/Client Components** - When to use each, composition patterns
4. **Data Fetching** - Server Components, caching, revalidation
5. **Server Actions** - Form handling, mutations, validation
6. **Rendering Strategies** - SSG, SSR, ISR, streaming
7. **Middleware** - Auth, i18n, redirects
8. **API Routes** - Route handlers, request/response
9. **Styling** - CSS Modules, Tailwind, CSS-in-JS
10. **Optimization** - Images, fonts, scripts, SEO
11. **Authentication** - NextAuth.js, session handling
12. **Deployment** - Vercel, Docker, static export
13. **Testing** - Unit, integration, E2E
14. **Best Practices** - Performance, security, accessibility

**Total Topics Covered**: 40+ major sections
**Code Examples**: 100+ practical implementations
**Interview Questions**: 25+ with detailed answers

---

*End of Part 5: Next.js Framework Complete Guide (6000+ lines)*

*Continue to Part 6 for State Management (Zustand & Beyond)...*

