# 🚀 Part 11: Performance, Testing & Deployment Complete Guide

> **Master frontend performance optimization, testing strategies, and modern deployment practices for interviews**

---

## 📋 Table of Contents

1. [Performance Fundamentals](#1-performance-fundamentals)
2. [Core Web Vitals](#2-core-web-vitals)
3. [Performance Profiling](#3-performance-profiling)
4. [Bundle Optimization](#4-bundle-optimization)
5. [Code Splitting & Lazy Loading](#5-code-splitting--lazy-loading)
6. [Image Optimization](#6-image-optimization)
7. [Caching Strategies](#7-caching-strategies)
8. [Testing Fundamentals](#8-testing-fundamentals)
9. [Unit Testing with Jest/Vitest](#9-unit-testing-with-jestvitest)
10. [Component Testing with RTL](#10-component-testing-with-rtl)
11. [E2E Testing with Playwright](#11-e2e-testing-with-playwright)
12. [CI/CD Pipelines](#12-cicd-pipelines)
13. [Deployment Strategies](#13-deployment-strategies)
14. [Docker & Containerization](#14-docker--containerization)
15. [Interview Questions](#15-interview-questions)
16. [Coding Challenges](#16-coding-challenges)

---

# 1. Performance Fundamentals

## 1.1 Why Performance Matters

```typescript
const PERFORMANCE_IMPACT = {
  business: {
    conversion: '1 second delay = 7% reduction in conversions',
    bounce: '53% of mobile users abandon sites that take >3s',
    seo: 'Page speed is a Google ranking factor',
    engagement: 'Faster sites have lower bounce rates',
  },
  
  user: {
    perception: 'Users perceive 100ms as instant',
    tolerance: 'Users expect pages to load in <3 seconds',
    frustration: 'Slow sites create negative brand perception',
  },
  
  technical: {
    cpu: 'JavaScript blocks the main thread',
    network: 'Large bundles delay interactivity',
    memory: 'Memory leaks cause poor performance over time',
  },
};
```

## 1.2 Performance Metrics

```typescript
// Key performance metrics

interface PerformanceMetrics {
  // Loading metrics
  TTFB: number;    // Time to First Byte
  FCP: number;     // First Contentful Paint
  LCP: number;     // Largest Contentful Paint
  
  // Interactivity metrics
  FID: number;     // First Input Delay (deprecated)
  INP: number;     // Interaction to Next Paint (new!)
  TTI: number;     // Time to Interactive
  TBT: number;     // Total Blocking Time
  
  // Visual stability
  CLS: number;     // Cumulative Layout Shift
  
  // Resource metrics
  resourceCount: number;
  transferSize: number;
  domSize: number;
}

// Measure performance with Performance API
function measurePerformance(): PerformanceMetrics {
  const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
  const paint = performance.getEntriesByType('paint');
  
  return {
    TTFB: navigation.responseStart - navigation.requestStart,
    FCP: paint.find(p => p.name === 'first-contentful-paint')?.startTime || 0,
    LCP: 0, // Requires PerformanceObserver
    FID: 0, // Requires PerformanceObserver
    INP: 0, // Requires PerformanceObserver
    TTI: 0, // Complex calculation
    TBT: 0, // Sum of blocking time
    CLS: 0, // Requires PerformanceObserver
    resourceCount: performance.getEntriesByType('resource').length,
    transferSize: navigation.transferSize,
    domSize: document.getElementsByTagName('*').length,
  };
}
```

## 1.3 Performance Observer API

```typescript
// Modern way to measure Core Web Vitals

// LCP - Largest Contentful Paint
function measureLCP(callback: (value: number) => void) {
  const observer = new PerformanceObserver((list) => {
    const entries = list.getEntries();
    const lastEntry = entries[entries.length - 1];
    callback(lastEntry.startTime);
  });
  
  observer.observe({ type: 'largest-contentful-paint', buffered: true });
  
  return () => observer.disconnect();
}

// INP - Interaction to Next Paint
function measureINP(callback: (value: number) => void) {
  let maxDuration = 0;
  
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if ((entry as any).duration > maxDuration) {
        maxDuration = (entry as any).duration;
        callback(maxDuration);
      }
    }
  });
  
  observer.observe({ type: 'event', buffered: true, durationThreshold: 16 });
  
  return () => observer.disconnect();
}

// CLS - Cumulative Layout Shift
function measureCLS(callback: (value: number) => void) {
  let clsValue = 0;
  let sessionValue = 0;
  let sessionEntries: PerformanceEntry[] = [];
  
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries() as any[]) {
      if (!entry.hadRecentInput) {
        const firstEntry = sessionEntries[0];
        const lastEntry = sessionEntries[sessionEntries.length - 1];
        
        if (
          sessionValue &&
          entry.startTime - lastEntry?.startTime < 1000 &&
          entry.startTime - firstEntry?.startTime < 5000
        ) {
          sessionValue += entry.value;
          sessionEntries.push(entry);
        } else {
          sessionValue = entry.value;
          sessionEntries = [entry];
        }
        
        if (sessionValue > clsValue) {
          clsValue = sessionValue;
          callback(clsValue);
        }
      }
    }
  });
  
  observer.observe({ type: 'layout-shift', buffered: true });
  
  return () => observer.disconnect();
}

// Complete Web Vitals hook
function useWebVitals() {
  const [vitals, setVitals] = useState({
    LCP: 0,
    INP: 0,
    CLS: 0,
  });
  
  useEffect(() => {
    const cleanups = [
      measureLCP(lcp => setVitals(v => ({ ...v, LCP: lcp }))),
      measureINP(inp => setVitals(v => ({ ...v, INP: inp }))),
      measureCLS(cls => setVitals(v => ({ ...v, CLS: cls }))),
    ];
    
    return () => cleanups.forEach(cleanup => cleanup());
  }, []);
  
  return vitals;
}
```

---

# 2. Core Web Vitals

## 2.1 LCP - Largest Contentful Paint

```typescript
// LCP measures loading performance
// Target: < 2.5 seconds

const LCP_THRESHOLDS = {
  good: 2500,      // < 2.5s
  needsImprovement: 4000, // 2.5s - 4s
  poor: Infinity,  // > 4s
};

// Common LCP elements:
// - <img> elements
// - <image> elements inside <svg>
// - <video> elements with poster image
// - Elements with background-image url()
// - Block-level elements with text content

// Optimization strategies for LCP
const LCP_OPTIMIZATIONS = {
  // 1. Optimize server response time
  server: [
    'Use a CDN',
    'Cache pages on the edge',
    'Use HTTP/2 or HTTP/3',
    'Optimize database queries',
  ],
  
  // 2. Optimize resource loading
  resources: [
    'Preload LCP image: <link rel="preload" as="image">',
    'Use responsive images with srcset',
    'Compress and optimize images',
    'Avoid lazy loading LCP image',
  ],
  
  // 3. Optimize rendering
  rendering: [
    'Inline critical CSS',
    'Remove render-blocking resources',
    'Defer non-critical JavaScript',
    'Use font-display: swap',
  ],
};

// React example: Preload LCP image
function HeroSection() {
  return (
    <>
      {/* Preload in document head via Next.js */}
      <Head>
        <link
          rel="preload"
          href="/hero-image.webp"
          as="image"
          type="image/webp"
        />
      </Head>
      
      <section className="hero">
        <img
          src="/hero-image.webp"
          alt="Hero"
          fetchPriority="high"  // New attribute!
          loading="eager"       // Don't lazy load LCP
          decoding="async"
        />
      </section>
    </>
  );
}
```

## 2.2 INP - Interaction to Next Paint

```typescript
// INP replaced FID in 2024
// Measures responsiveness throughout the page lifecycle
// Target: < 200ms

const INP_THRESHOLDS = {
  good: 200,       // < 200ms
  needsImprovement: 500, // 200ms - 500ms
  poor: Infinity,  // > 500ms
};

// INP captures all interactions, not just the first
// Includes: clicks, taps, keyboard inputs

// Common causes of poor INP:
const INP_ISSUES = {
  longTasks: 'JavaScript tasks blocking main thread > 50ms',
  heavyComponents: 'Complex re-renders on interaction',
  synchronousWork: 'Blocking operations in event handlers',
  thirdPartyScripts: 'Analytics, ads blocking interactions',
};

// Optimization strategies
const INP_OPTIMIZATIONS = {
  // 1. Break up long tasks
  breakUpTasks: `
    // BAD: Single long task
    function processData() {
      for (let i = 0; i < 1000000; i++) {
        doHeavyWork(i);
      }
    }
    
    // GOOD: Yield to main thread
    async function processData() {
      for (let i = 0; i < 1000000; i++) {
        doHeavyWork(i);
        
        if (i % 1000 === 0) {
          // Yield to allow user interactions
          await scheduler.yield?.() ?? new Promise(r => setTimeout(r, 0));
        }
      }
    }
  `,
  
  // 2. Use transitions for non-urgent updates
  useTransition: `
    function SearchResults() {
      const [query, setQuery] = useState('');
      const [results, setResults] = useState([]);
      const [isPending, startTransition] = useTransition();
      
      const handleSearch = (e) => {
        // Urgent: Update input immediately
        setQuery(e.target.value);
        
        // Non-urgent: Can be interrupted
        startTransition(() => {
          setResults(filterResults(e.target.value));
        });
      };
      
      return (
        <>
          <input value={query} onChange={handleSearch} />
          {isPending && <Spinner />}
          <ResultsList results={results} />
        </>
      );
    }
  `,
  
  // 3. Debounce expensive handlers
  debounce: `
    function SearchInput() {
      const [query, setQuery] = useState('');
      
      const debouncedSearch = useMemo(
        () => debounce((value) => {
          // Heavy search logic
        }, 300),
        []
      );
      
      return (
        <input
          value={query}
          onChange={(e) => {
            setQuery(e.target.value);
            debouncedSearch(e.target.value);
          }}
        />
      );
    }
  `,
};

// scheduler.yield() polyfill
if (!('scheduler' in globalThis)) {
  (globalThis as any).scheduler = {
    yield: () => new Promise(r => setTimeout(r, 0)),
  };
}

async function yieldToMain() {
  return (globalThis as any).scheduler?.yield?.() ?? 
    new Promise(r => setTimeout(r, 0));
}
```

## 2.3 CLS - Cumulative Layout Shift

```typescript
// CLS measures visual stability
// Target: < 0.1

const CLS_THRESHOLDS = {
  good: 0.1,       // < 0.1
  needsImprovement: 0.25, // 0.1 - 0.25
  poor: Infinity,  // > 0.25
};

// Common causes of layout shifts:
const CLS_CAUSES = [
  'Images without dimensions',
  'Ads, embeds, iframes without size',
  'Dynamically injected content',
  'Web fonts causing FOIT/FOUT',
  'Actions waiting for network before DOM update',
];

// Optimization strategies
const CLS_OPTIMIZATIONS = {
  // 1. Always include size attributes
  images: `
    // BAD
    <img src="photo.jpg" />
    
    // GOOD
    <img src="photo.jpg" width="800" height="600" />
    
    // BETTER with aspect-ratio
    <img 
      src="photo.jpg" 
      style={{ aspectRatio: '16 / 9', width: '100%' }} 
    />
  `,
  
  // 2. Reserve space for ads/embeds
  reserveSpace: `
    function AdSlot() {
      return (
        <div style={{ 
          minHeight: 250, 
          background: '#f0f0f0' 
        }}>
          {/* Ad content loads here */}
        </div>
      );
    }
  `,
  
  // 3. Use font-display
  fonts: `
    @font-face {
      font-family: 'MyFont';
      src: url('/fonts/myfont.woff2') format('woff2');
      font-display: optional; /* Best for CLS */
    }
    
    /* Options:
     * swap - Show fallback, swap when loaded (can cause shift)
     * optional - Only use if already cached (best CLS)
     * fallback - Short swap window
     * block - Invisible text until loaded
     */
  `,
  
  // 4. Transform-based animations
  animations: `
    // BAD - triggers layout
    .animate {
      transition: height 0.3s;
    }
    
    // GOOD - GPU accelerated, no layout
    .animate {
      transform: scaleY(1);
      transition: transform 0.3s;
    }
  `,
};

// React component with stable layout
function ImageWithAspectRatio({ src, alt, aspectRatio = '16/9' }) {
  return (
    <div style={{ 
      position: 'relative', 
      width: '100%', 
      aspectRatio 
    }}>
      <img
        src={src}
        alt={alt}
        style={{
          position: 'absolute',
          width: '100%',
          height: '100%',
          objectFit: 'cover',
        }}
      />
    </div>
  );
}
```

---

# 3. Performance Profiling

## 3.1 Chrome DevTools Performance Panel

```typescript
// Key areas in Performance panel

const PERFORMANCE_PANEL = {
  timeline: {
    frames: 'Visual frames over time',
    network: 'Network activity',
    main: 'Main thread activity',
    raster: 'Painting activity',
    gpu: 'GPU activity',
  },
  
  summaryChart: {
    loading: 'Blue - Network loading',
    scripting: 'Yellow - JavaScript execution',
    rendering: 'Purple - Style/layout calculations',
    painting: 'Green - Paint/composite',
    system: 'Grey - Other browser work',
    idle: 'White - Idle time',
  },
  
  bottomUp: 'Shows activities by self-time',
  callTree: 'Shows the call hierarchy',
  eventLog: 'Chronological event list',
};

// Things to look for:
const PERFORMANCE_RED_FLAGS = {
  longTasks: 'Yellow blocks > 50ms on main thread',
  layoutThrashing: 'Purple "Recalculate Style" repeated',
  forcedReflow: 'Purple "Layout" with warning icon',
  largeScriptBlocks: 'Large yellow blocks during load',
  jank: 'Frame rate dipping below 60fps',
};
```

## 3.2 React DevTools Profiler

```typescript
// React-specific profiling

// 1. Enable profiling in production
// Add to webpack.config.js for production profiling:
const ENABLE_PROFILING = `
  module.exports = {
    resolve: {
      alias: {
        'react-dom$': 'react-dom/profiling',
        'scheduler/tracing': 'scheduler/tracing-profiling',
      },
    },
  };
`;

// 2. Use Profiler component programmatically
import { Profiler, ProfilerOnRenderCallback } from 'react';

const onRenderCallback: ProfilerOnRenderCallback = (
  id,           // Profiler id
  phase,        // "mount" | "update"
  actualDuration,   // Time rendering took
  baseDuration,     // Estimated time without memoization
  startTime,        // When React started rendering
  commitTime,       // When React committed the updates
) => {
  // Log slow renders
  if (actualDuration > 16) { // Slower than 1 frame at 60fps
    console.warn(`Slow render in ${id}:`, {
      phase,
      actualDuration,
      baseDuration,
    });
  }
};

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Dashboard />
    </Profiler>
  );
}

// 3. Track renders with why-did-you-render
// npm install @welldone-software/why-did-you-render

// wdyr.ts
import React from 'react';

if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, {
    trackAllPureComponents: true,
  });
}

// Mark specific components:
function ExpensiveComponent() {
  // ...
}
ExpensiveComponent.whyDidYouRender = true;
```

## 3.3 Lighthouse

```typescript
// Lighthouse categories and metrics

const LIGHTHOUSE_CATEGORIES = {
  performance: {
    weight: 'Most important for speed',
    metrics: ['FCP', 'LCP', 'TBT', 'CLS', 'Speed Index'],
    scoring: '0-100 based on metrics',
  },
  
  accessibility: {
    weight: 'WCAG compliance',
    checks: ['Color contrast', 'ARIA', 'Keyboard navigation'],
  },
  
  bestPractices: {
    weight: 'Security and modern standards',
    checks: ['HTTPS', 'Console errors', 'Deprecated APIs'],
  },
  
  seo: {
    weight: 'Search engine optimization',
    checks: ['Meta tags', 'Mobile friendly', 'Crawlability'],
  },
};

// Run Lighthouse programmatically
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

async function runLighthouse(url: string) {
  const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });
  
  const options = {
    logLevel: 'info' as const,
    output: 'json' as const,
    port: chrome.port,
    onlyCategories: ['performance'],
  };
  
  const result = await lighthouse(url, options);
  
  await chrome.kill();
  
  const { lhr } = result!;
  
  return {
    performance: lhr.categories.performance.score! * 100,
    metrics: {
      FCP: lhr.audits['first-contentful-paint'].numericValue,
      LCP: lhr.audits['largest-contentful-paint'].numericValue,
      TBT: lhr.audits['total-blocking-time'].numericValue,
      CLS: lhr.audits['cumulative-layout-shift'].numericValue,
    },
  };
}
```

---

# 4. Bundle Optimization

## 4.1 Analyzing Bundle Size

```typescript
// Tools for bundle analysis

// 1. webpack-bundle-analyzer
// npm install --save-dev webpack-bundle-analyzer

// next.config.js (Next.js)
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config
});

// Run: ANALYZE=true npm run build

// 2. source-map-explorer
// npm install --save-dev source-map-explorer

// package.json
{
  "scripts": {
    "analyze": "source-map-explorer 'dist/**/*.js'"
  }
}

// 3. bundlephobia - Check package sizes before installing
// https://bundlephobia.com/
```

## 4.2 Tree Shaking

```typescript
// Tree shaking eliminates unused code

// Requirements:
// 1. Use ES6 modules (import/export)
// 2. Mark package as side-effect-free

// package.json
{
  "sideEffects": false,
  // or list files with side effects:
  "sideEffects": [
    "*.css",
    "./src/polyfills.js"
  ]
}

// BAD: Import entire library
import _ from 'lodash';
const result = _.pick(obj, ['a', 'b']);

// GOOD: Import only what you need
import pick from 'lodash/pick';
// or with lodash-es:
import { pick } from 'lodash-es';

// BAD: Barrel exports break tree shaking
// utils/index.ts
export * from './stringUtils';
export * from './dateUtils';
export * from './mathUtils';

// GOOD: Direct imports
import { formatDate } from './utils/dateUtils';

// Check tree shaking effectiveness
// Look for /* unused harmony export */ in production bundle
```

## 4.3 Code Minification

```typescript
// Terser configuration for maximum minification

// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          parse: {
            ecma: 2020,
          },
          compress: {
            ecma: 2020,
            comparisons: false,
            inline: 2,
            drop_console: true,      // Remove console.log in prod
            drop_debugger: true,     // Remove debugger statements
            pure_funcs: ['console.log', 'console.info'],
          },
          mangle: {
            safari10: true,
          },
          output: {
            ecma: 2020,
            comments: false,
            ascii_only: true,
          },
        },
        parallel: true,
        extractComments: false,
      }),
    ],
  },
};
```

## 4.4 Modern JavaScript for Modern Browsers

```typescript
// Serve modern JS to modern browsers

// browserslist configuration
// package.json
{
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version"
    ],
    "modern": [
      "last 2 Chrome versions",
      "last 2 Firefox versions",
      "last 2 Safari versions"
    ]
  }
}

// Differential loading (ship modern + legacy bundles)
// HTML example:
`
<script type="module" src="/modern.js"></script>
<script nomodule src="/legacy.js"></script>
`

// Next.js handles this automatically!
```

---

# 5. Code Splitting & Lazy Loading

## 5.1 Dynamic Imports

```typescript
// Basic dynamic import
const loadModule = async () => {
  const module = await import('./heavyModule');
  return module.default;
};

// React lazy loading
import { lazy, Suspense } from 'react';

// Component lazy loading
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// Named export lazy loading
const Chart = lazy(() => 
  import('./Chart').then(module => ({ default: module.Chart }))
);

// With loading states
function PageWithLazyComponent() {
  return (
    <Suspense 
      fallback={
        <div className="skeleton" style={{ height: 400 }}>
          Loading chart...
        </div>
      }
    >
      <Chart data={data} />
    </Suspense>
  );
}
```

## 5.2 Route-Based Code Splitting

```typescript
// Next.js App Router - Automatic code splitting per page

// app/dashboard/page.tsx
export default function DashboardPage() {
  // This code is automatically split into its own bundle
  return <Dashboard />;
}

// React Router with lazy loading
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

## 5.3 Component-Level Code Splitting

```typescript
// Split based on visibility
function Dashboard() {
  const [showAdvanced, setShowAdvanced] = useState(false);
  
  // Only load when needed
  const AdvancedSettings = lazy(() => import('./AdvancedSettings'));
  
  return (
    <div>
      <BasicSettings />
      
      <button onClick={() => setShowAdvanced(true)}>
        Show Advanced Settings
      </button>
      
      {showAdvanced && (
        <Suspense fallback={<Loading />}>
          <AdvancedSettings />
        </Suspense>
      )}
    </div>
  );
}

// Split based on viewport (lazy load below fold)
function useLazyLoad(ref: React.RefObject<Element>) {
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { rootMargin: '100px' } // Start loading 100px before visible
    );
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => observer.disconnect();
  }, [ref]);
  
  return isVisible;
}

const HeavyChart = lazy(() => import('./HeavyChart'));

function ChartSection() {
  const ref = useRef<HTMLDivElement>(null);
  const isVisible = useLazyLoad(ref);
  
  return (
    <div ref={ref} style={{ minHeight: 400 }}>
      {isVisible && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

## 5.4 Prefetching and Preloading

```typescript
// Prefetch on hover (for navigation)
function NavLink({ to, children }: { to: string; children: React.ReactNode }) {
  const prefetchPage = () => {
    // Dynamically import the page on hover
    import(`./pages${to}`);
  };
  
  return (
    <Link 
      to={to} 
      onMouseEnter={prefetchPage}
      onFocus={prefetchPage}
    >
      {children}
    </Link>
  );
}

// Next.js Link prefetching (automatic)
import Link from 'next/link';

function Navigation() {
  return (
    <nav>
      {/* Prefetched when visible in viewport */}
      <Link href="/dashboard">Dashboard</Link>
      
      {/* Disable prefetching */}
      <Link href="/rarely-visited" prefetch={false}>
        Rarely Visited
      </Link>
    </nav>
  );
}

// Preload critical resources
function preloadCriticalResources() {
  // Preload next page's main chunk
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = '/chunks/dashboard.js';
  document.head.appendChild(link);
  
  // Preload images
  const img = new Image();
  img.src = '/hero-image.webp';
}
```

---

# 6. Image Optimization

## 6.1 Modern Image Formats

```typescript
const IMAGE_FORMATS = {
  jpeg: {
    compression: 'Lossy',
    transparency: false,
    animation: false,
    bestFor: 'Photos',
  },
  png: {
    compression: 'Lossless',
    transparency: true,
    animation: false,
    bestFor: 'Icons, screenshots',
  },
  webp: {
    compression: 'Both lossy and lossless',
    transparency: true,
    animation: true,
    bestFor: 'Everything - 25-35% smaller than JPEG',
    support: '97%+ browser support',
  },
  avif: {
    compression: 'Both lossy and lossless',
    transparency: true,
    animation: true,
    bestFor: 'Best compression - 50% smaller than JPEG',
    support: '~85% browser support',
  },
  svg: {
    compression: 'N/A (vector)',
    transparency: true,
    animation: true,
    bestFor: 'Icons, logos, illustrations',
  },
};

// Progressive enhancement with picture element
function OptimizedImage({ src, alt, width, height }) {
  return (
    <picture>
      {/* Try AVIF first */}
      <source 
        srcSet={`${src}.avif`} 
        type="image/avif" 
      />
      {/* Fall back to WebP */}
      <source 
        srcSet={`${src}.webp`} 
        type="image/webp" 
      />
      {/* Final fallback to JPEG */}
      <img 
        src={`${src}.jpg`} 
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        decoding="async"
      />
    </picture>
  );
}
```

## 6.2 Responsive Images

```typescript
// srcset for different screen densities
function ResponsiveImage({ src, alt }) {
  return (
    <img
      src={`${src}-400.jpg`}
      srcSet={`
        ${src}-400.jpg 400w,
        ${src}-800.jpg 800w,
        ${src}-1200.jpg 1200w,
        ${src}-1600.jpg 1600w
      `}
      sizes="
        (max-width: 400px) 100vw,
        (max-width: 800px) 50vw,
        33vw
      "
      alt={alt}
      loading="lazy"
    />
  );
}

// Next.js Image component (recommended)
import Image from 'next/image';

function HeroImage() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority  // Preload LCP image
      placeholder="blur"
      blurDataURL="/hero-blur.jpg"
      sizes="100vw"
    />
  );
}

// With fill mode
function BackgroundImage() {
  return (
    <div style={{ position: 'relative', height: 400 }}>
      <Image
        src="/background.jpg"
        alt=""
        fill
        style={{ objectFit: 'cover' }}
        quality={75}
      />
    </div>
  );
}
```

## 6.3 Lazy Loading Images

```typescript
// Native lazy loading
function LazyImage({ src, alt }) {
  return (
    <img 
      src={src} 
      alt={alt}
      loading="lazy"      // Native lazy loading
      decoding="async"    // Don't block main thread
    />
  );
}

// IntersectionObserver for custom lazy loading
function useImageLazyLoad(
  ref: React.RefObject<HTMLImageElement>,
  src: string
) {
  useEffect(() => {
    const img = ref.current;
    if (!img) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          img.src = src;
          img.classList.add('loaded');
          observer.disconnect();
        }
      },
      { rootMargin: '200px' } // Start loading 200px before visible
    );
    
    observer.observe(img);
    
    return () => observer.disconnect();
  }, [ref, src]);
}

// LQIP (Low Quality Image Placeholder)
function ImageWithLQIP({ src, lqip, alt }) {
  const [loaded, setLoaded] = useState(false);
  
  return (
    <div className="image-container">
      {/* Blurred placeholder */}
      <img
        src={lqip}
        alt=""
        className="lqip"
        style={{
          filter: loaded ? 'none' : 'blur(20px)',
          transition: 'filter 0.3s',
        }}
      />
      
      {/* Actual image */}
      <img
        src={src}
        alt={alt}
        loading="lazy"
        onLoad={() => setLoaded(true)}
        style={{ opacity: loaded ? 1 : 0 }}
      />
    </div>
  );
}
```

---

# 7. Caching Strategies

## 7.1 Browser Caching

```typescript
// Cache-Control headers

const CACHE_STRATEGIES = {
  // Static assets (JS, CSS, images)
  immutable: {
    header: 'Cache-Control: public, max-age=31536000, immutable',
    use: 'Hashed filenames (main.abc123.js)',
    reason: 'Cache forever, filename changes on update',
  },
  
  // HTML pages
  noCache: {
    header: 'Cache-Control: no-cache',
    use: 'HTML documents',
    reason: 'Always validate with server',
  },
  
  // API responses
  shortTerm: {
    header: 'Cache-Control: private, max-age=60',
    use: 'Dynamic data',
    reason: 'Cache briefly, user-specific',
  },
  
  // Stale-while-revalidate
  swr: {
    header: 'Cache-Control: max-age=60, stale-while-revalidate=3600',
    use: 'Semi-dynamic content',
    reason: 'Serve stale while fetching fresh',
  },
};

// Next.js cache configuration
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:all*(svg|jpg|png|webp|avif)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=0, must-revalidate',
          },
        ],
      },
    ];
  },
};
```

## 7.2 Service Worker Caching

```typescript
// service-worker.ts

const CACHE_NAME = 'app-v1';
const STATIC_CACHE = 'static-v1';
const DYNAMIC_CACHE = 'dynamic-v1';

const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/offline.html',
  '/styles.css',
  '/main.js',
];

// Install: Cache static assets
self.addEventListener('install', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(STATIC_CACHE).then(cache => {
      return cache.addAll(STATIC_ASSETS);
    })
  );
});

// Activate: Clean old caches
self.addEventListener('activate', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.keys().then(keys => {
      return Promise.all(
        keys
          .filter(key => key !== STATIC_CACHE && key !== DYNAMIC_CACHE)
          .map(key => caches.delete(key))
      );
    })
  );
});

// Fetch strategies
self.addEventListener('fetch', (event: FetchEvent) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Cache-first for static assets
  if (STATIC_ASSETS.includes(url.pathname)) {
    event.respondWith(cacheFirst(request));
    return;
  }
  
  // Network-first for API calls
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request));
    return;
  }
  
  // Stale-while-revalidate for everything else
  event.respondWith(staleWhileRevalidate(request));
});

async function cacheFirst(request: Request): Promise<Response> {
  const cached = await caches.match(request);
  if (cached) return cached;
  
  const response = await fetch(request);
  const cache = await caches.open(STATIC_CACHE);
  cache.put(request, response.clone());
  return response;
}

async function networkFirst(request: Request): Promise<Response> {
  try {
    const response = await fetch(request);
    const cache = await caches.open(DYNAMIC_CACHE);
    cache.put(request, response.clone());
    return response;
  } catch {
    const cached = await caches.match(request);
    return cached || new Response('Offline', { status: 503 });
  }
}

async function staleWhileRevalidate(request: Request): Promise<Response> {
  const cache = await caches.open(DYNAMIC_CACHE);
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });
  
  return cached || fetchPromise;
}
```

## 7.3 React Query Caching

```typescript
import { QueryClient, QueryClientProvider, useQuery } from '@tanstack/react-query';

// Configure query client with caching
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 30 * 60 * 1000,         // 30 minutes (formerly cacheTime)
      refetchOnWindowFocus: false,
      retry: 3,
    },
  },
});

// Define cache keys
const queryKeys = {
  users: ['users'] as const,
  user: (id: string) => ['users', id] as const,
  posts: (filters: PostFilters) => ['posts', filters] as const,
};

// Use cached data
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: queryKeys.user(userId),
    queryFn: () => fetchUser(userId),
    staleTime: 10 * 60 * 1000, // Consider fresh for 10 minutes
  });
  
  if (isLoading) return <Skeleton />;
  if (error) return <Error error={error} />;
  
  return <Profile user={data} />;
}

// Optimistic updates
function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateUser,
    onMutate: async (newUser) => {
      // Cancel outgoing queries
      await queryClient.cancelQueries({ queryKey: queryKeys.user(newUser.id) });
      
      // Snapshot previous value
      const previousUser = queryClient.getQueryData(queryKeys.user(newUser.id));
      
      // Optimistically update
      queryClient.setQueryData(queryKeys.user(newUser.id), newUser);
      
      return { previousUser };
    },
    onError: (err, newUser, context) => {
      // Rollback on error
      queryClient.setQueryData(
        queryKeys.user(newUser.id),
        context?.previousUser
      );
    },
    onSettled: (data, error, newUser) => {
      // Always refetch after mutation
      queryClient.invalidateQueries({ queryKey: queryKeys.user(newUser.id) });
    },
  });
}
```

---

# 8. Testing Fundamentals

## 8.1 Testing Philosophy

```typescript
// Test pyramid

const TEST_PYRAMID = {
  unit: {
    percentage: '70%',
    speed: 'Fast (ms)',
    scope: 'Single function/component',
    cost: 'Low',
    tools: ['Jest', 'Vitest', 'Testing Library'],
  },
  
  integration: {
    percentage: '20%',
    speed: 'Medium (seconds)',
    scope: 'Multiple components working together',
    cost: 'Medium',
    tools: ['Testing Library', 'MSW'],
  },
  
  e2e: {
    percentage: '10%',
    speed: 'Slow (minutes)',
    scope: 'Full user flows',
    cost: 'High',
    tools: ['Playwright', 'Cypress'],
  },
};

// What to test
const TESTING_PRIORITIES = {
  alwaysTest: [
    'Business logic',
    'User-facing features',
    'Edge cases',
    'Error handling',
    'Accessibility',
  ],
  
  sometimesTest: [
    'Complex UI interactions',
    'Third-party integrations',
    'Performance-critical paths',
  ],
  
  rarelyTest: [
    'Implementation details',
    'Styling/CSS',
    'Third-party library code',
  ],
};
```

## 8.2 Testing Principles

```typescript
// AAA Pattern: Arrange, Act, Assert

describe('calculateTotal', () => {
  it('calculates total with tax', () => {
    // Arrange
    const items = [
      { price: 100, quantity: 2 },
      { price: 50, quantity: 1 },
    ];
    const taxRate = 0.1;
    
    // Act
    const result = calculateTotal(items, taxRate);
    
    // Assert
    expect(result).toBe(275); // (100*2 + 50) * 1.1
  });
});

// Test behavior, not implementation
describe('ShoppingCart', () => {
  // BAD: Testing implementation
  it('calls setItems when adding item', () => {
    const setItems = jest.fn();
    // Testing internal state management
  });
  
  // GOOD: Testing behavior
  it('shows the added item in the cart', () => {
    render(<ShoppingCart />);
    
    userEvent.click(screen.getByRole('button', { name: /add to cart/i }));
    
    expect(screen.getByText('Product Name')).toBeInTheDocument();
  });
});

// Each test should be independent
describe('Counter', () => {
  // DON'T share state between tests
  let counter: Counter;
  
  beforeEach(() => {
    // Reset for each test
    counter = new Counter(0);
  });
  
  it('starts at initial value', () => {
    expect(counter.value).toBe(0);
  });
  
  it('increments correctly', () => {
    counter.increment();
    expect(counter.value).toBe(1);
  });
});
```

---

# 9. Unit Testing with Jest/Vitest

## 9.1 Basic Setup

```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};

export default config;

// jest.setup.ts
import '@testing-library/jest-dom';
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// vitest.config.ts (Vitest alternative)
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
});
```

## 9.2 Testing Functions

```typescript
// utils.ts
export function formatCurrency(amount: number, currency = 'USD'): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
}

export function debounce<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
): T {
  let timeoutId: NodeJS.Timeout;
  return ((...args: Parameters<T>) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  }) as T;
}

// utils.test.ts
import { formatCurrency, debounce } from './utils';

describe('formatCurrency', () => {
  it('formats USD correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });
  
  it('formats EUR correctly', () => {
    expect(formatCurrency(1234.56, 'EUR')).toBe('€1,234.56');
  });
  
  it('handles zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
  
  it('handles negative numbers', () => {
    expect(formatCurrency(-100)).toBe('-$100.00');
  });
});

describe('debounce', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });
  
  afterEach(() => {
    jest.useRealTimers();
  });
  
  it('delays function execution', () => {
    const fn = jest.fn();
    const debouncedFn = debounce(fn, 100);
    
    debouncedFn();
    expect(fn).not.toHaveBeenCalled();
    
    jest.advanceTimersByTime(100);
    expect(fn).toHaveBeenCalledTimes(1);
  });
  
  it('only calls once for rapid calls', () => {
    const fn = jest.fn();
    const debouncedFn = debounce(fn, 100);
    
    debouncedFn();
    debouncedFn();
    debouncedFn();
    
    jest.advanceTimersByTime(100);
    expect(fn).toHaveBeenCalledTimes(1);
  });
});
```

## 9.3 Testing Async Code

```typescript
// api.ts
export async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('User not found');
  }
  return response.json();
}

// api.test.ts
import { fetchUser } from './api';
import { server } from './mocks/server';
import { rest } from 'msw';

describe('fetchUser', () => {
  it('fetches user successfully', async () => {
    const user = await fetchUser('123');
    
    expect(user).toEqual({
      id: '123',
      name: 'John Doe',
      email: 'john@example.com',
    });
  });
  
  it('throws error for non-existent user', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.status(404));
      })
    );
    
    await expect(fetchUser('999')).rejects.toThrow('User not found');
  });
  
  it('handles network error', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res) => {
        return res.networkError('Failed to connect');
      })
    );
    
    await expect(fetchUser('123')).rejects.toThrow();
  });
});

// Testing with async/await
describe('async operations', () => {
  it('resolves with value', async () => {
    const result = await someAsyncFunction();
    expect(result).toBe('expected value');
  });
  
  it('rejects with error', async () => {
    await expect(failingAsyncFunction()).rejects.toThrow('Error message');
  });
  
  // Using resolves/rejects matchers
  it('alternative syntax', () => {
    return expect(someAsyncFunction()).resolves.toBe('expected value');
  });
});
```

## 9.4 Mocking

```typescript
// Mocking modules
jest.mock('./api', () => ({
  fetchUser: jest.fn(),
  updateUser: jest.fn(),
}));

import { fetchUser, updateUser } from './api';

const mockedFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>;

beforeEach(() => {
  mockedFetchUser.mockResolvedValue({
    id: '1',
    name: 'Test User',
  });
});

// Mocking implementations
describe('with mocked API', () => {
  it('uses mocked return value', async () => {
    const user = await fetchUser('1');
    expect(user.name).toBe('Test User');
  });
  
  it('can change mock per test', async () => {
    mockedFetchUser.mockResolvedValueOnce({
      id: '2',
      name: 'Another User',
    });
    
    const user = await fetchUser('2');
    expect(user.name).toBe('Another User');
  });
});

// Spying on methods
describe('spy example', () => {
  it('tracks calls', () => {
    const obj = {
      method: (x: number) => x * 2,
    };
    
    const spy = jest.spyOn(obj, 'method');
    
    obj.method(5);
    obj.method(10);
    
    expect(spy).toHaveBeenCalledTimes(2);
    expect(spy).toHaveBeenCalledWith(5);
    expect(spy).toHaveBeenLastCalledWith(10);
  });
});
```

---

# 10. Component Testing with React Testing Library

## 10.1 Basic Component Testing

```typescript
// Button.tsx
interface ButtonProps {
  onClick?: () => void;
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
}

function Button({ onClick, disabled, loading, children }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-busy={loading}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
}

// Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);
    
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when loading', () => {
    render(<Button loading>Click me</Button>);
    
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
  });
  
  it('shows spinner when loading', () => {
    render(<Button loading>Click me</Button>);
    
    expect(screen.queryByText('Click me')).not.toBeInTheDocument();
    expect(screen.getByRole('status')).toBeInTheDocument(); // Spinner
  });
});
```

## 10.2 Testing Forms

```typescript
// LoginForm.tsx
function LoginForm({ onSubmit }: { onSubmit: (data: LoginData) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!email || !password) {
      setError('All fields are required');
      return;
    }
    
    onSubmit({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {error && <div role="alert">{error}</div>}
      
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
      />
      
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
      />
      
      <button type="submit">Log in</button>
    </form>
  );
}

// LoginForm.test.tsx
describe('LoginForm', () => {
  const user = userEvent.setup();
  
  it('submits form with valid data', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });
  
  it('shows error for empty fields', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(screen.getByRole('alert')).toHaveTextContent(/all fields are required/i);
    expect(handleSubmit).not.toHaveBeenCalled();
  });
  
  it('clears error when user types', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    // Trigger error
    await user.click(screen.getByRole('button', { name: /log in/i }));
    expect(screen.getByRole('alert')).toBeInTheDocument();
    
    // Start typing
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    // Error should still be visible (validation on submit)
  });
});
```

## 10.3 Testing with Context and Providers

```typescript
// Custom render with providers
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ThemeProvider } from './ThemeContext';

const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
    },
  },
});

interface WrapperProps {
  children: React.ReactNode;
}

function AllTheProviders({ children }: WrapperProps) {
  const queryClient = createTestQueryClient();
  
  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        {children}
      </ThemeProvider>
    </QueryClientProvider>
  );
}

const customRender = (
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllTheProviders, ...options });

// Re-export everything
export * from '@testing-library/react';
export { customRender as render };

// Usage in tests
import { render, screen } from './test-utils';

describe('Dashboard', () => {
  it('renders with providers', async () => {
    render(<Dashboard />);
    
    await screen.findByText('Dashboard');
  });
});
```

## 10.4 Testing Accessibility

```typescript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('accessibility', () => {
  it('Navigation has no a11y violations', async () => {
    const { container } = render(<Navigation />);
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  it('Form has proper labels', () => {
    render(<ContactForm />);
    
    // Check associations
    const emailInput = screen.getByLabelText(/email/i);
    expect(emailInput).toHaveAttribute('type', 'email');
    
    // Check required fields
    const nameInput = screen.getByRole('textbox', { name: /name/i });
    expect(nameInput).toHaveAttribute('required');
  });
  
  it('Modal traps focus', async () => {
    const user = userEvent.setup();
    render(<Modal isOpen={true} />);
    
    const firstFocusable = screen.getByRole('button', { name: /close/i });
    const lastFocusable = screen.getByRole('button', { name: /submit/i });
    
    // Tab to last element
    await user.tab();
    await user.tab();
    expect(lastFocusable).toHaveFocus();
    
    // Tab should wrap to first
    await user.tab();
    expect(firstFocusable).toHaveFocus();
  });
});
```

---

# 11. E2E Testing with Playwright

## 11.1 Playwright Setup

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## 11.2 Writing E2E Tests

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can sign up', async ({ page }) => {
    await page.goto('/signup');
    
    await page.getByLabel('Name').fill('John Doe');
    await page.getByLabel('Email').fill('john@example.com');
    await page.getByLabel('Password').fill('SecurePass123!');
    await page.getByLabel('Confirm Password').fill('SecurePass123!');
    
    await page.getByRole('button', { name: 'Create Account' }).click();
    
    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome, John')).toBeVisible();
  });
  
  test('user can log in', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('john@example.com');
    await page.getByLabel('Password').fill('SecurePass123!');
    
    await page.getByRole('button', { name: 'Log in' }).click();
    
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('wrong@example.com');
    await page.getByLabel('Password').fill('wrong');
    
    await page.getByRole('button', { name: 'Log in' }).click();
    
    await expect(page.getByRole('alert')).toContainText('Invalid credentials');
    await expect(page).toHaveURL('/login');
  });
});

// e2e/checkout.spec.ts
test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login helper
    await page.goto('/login');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Log in' }).click();
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('complete checkout flow', async ({ page }) => {
    // Add item to cart
    await page.goto('/products');
    await page.getByRole('button', { name: 'Add to Cart' }).first().click();
    
    // Go to cart
    await page.getByRole('link', { name: 'Cart (1)' }).click();
    await expect(page.getByTestId('cart-item')).toHaveCount(1);
    
    // Proceed to checkout
    await page.getByRole('button', { name: 'Checkout' }).click();
    
    // Fill shipping info
    await page.getByLabel('Address').fill('123 Main St');
    await page.getByLabel('City').fill('New York');
    await page.getByLabel('Zip Code').fill('10001');
    
    await page.getByRole('button', { name: 'Continue to Payment' }).click();
    
    // Fill payment (test card)
    await page.frameLocator('iframe').getByLabel('Card Number').fill('4242424242424242');
    await page.frameLocator('iframe').getByLabel('MM/YY').fill('12/25');
    await page.frameLocator('iframe').getByLabel('CVC').fill('123');
    
    await page.getByRole('button', { name: 'Place Order' }).click();
    
    // Confirmation
    await expect(page.getByRole('heading', { name: 'Order Confirmed!' })).toBeVisible();
    await expect(page.getByText(/Order #\d+/)).toBeVisible();
  });
});
```

## 11.3 Advanced Playwright Features

```typescript
// Page Object Model
class LoginPage {
  constructor(private page: Page) {}
  
  async goto() {
    await this.page.goto('/login');
  }
  
  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Log in' }).click();
  }
}

class DashboardPage {
  constructor(private page: Page) {}
  
  async getWelcomeMessage() {
    return this.page.getByTestId('welcome-message').textContent();
  }
}

// Using Page Objects
test('login with page objects', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);
  
  await loginPage.goto();
  await loginPage.login('test@example.com', 'password');
  
  await expect(page).toHaveURL('/dashboard');
  const message = await dashboardPage.getWelcomeMessage();
  expect(message).toContain('Welcome');
});

// API mocking
test('with mocked API', async ({ page }) => {
  // Mock API response
  await page.route('/api/users', async (route) => {
    await route.fulfill({
      status: 200,
      body: JSON.stringify([{ id: 1, name: 'Mocked User' }]),
    });
  });
  
  await page.goto('/users');
  await expect(page.getByText('Mocked User')).toBeVisible();
});

// Fixtures
test.use({
  storageState: 'tests/.auth/user.json', // Reuse auth state
});

// Visual regression testing
test('visual regression', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page).toHaveScreenshot('dashboard.png', {
    maxDiffPixels: 100,
  });
});
```

---

# 12. CI/CD Pipelines

## 12.1 GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test -- --coverage
      
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:e2e
      
      - uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: .next/
```

## 12.2 Vercel Deployment

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'

  preview:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy Preview
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
```

## 12.3 Environment Management

```typescript
// Environment configuration

// .env.development
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_ENV=development
DATABASE_URL=postgresql://localhost/myapp_dev

// .env.staging
NEXT_PUBLIC_API_URL=https://api.staging.myapp.com
NEXT_PUBLIC_ENV=staging
DATABASE_URL=postgresql://...

// .env.production
NEXT_PUBLIC_API_URL=https://api.myapp.com
NEXT_PUBLIC_ENV=production
DATABASE_URL=postgresql://...

// Type-safe environment variables
// src/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NEXT_PUBLIC_API_URL: z.string().url(),
  NEXT_PUBLIC_ENV: z.enum(['development', 'staging', 'production']),
  DATABASE_URL: z.string().optional(),
});

export const env = envSchema.parse({
  NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  NEXT_PUBLIC_ENV: process.env.NEXT_PUBLIC_ENV,
  DATABASE_URL: process.env.DATABASE_URL,
});
```

---

# 13. Deployment Strategies

## 13.1 Deployment Patterns

```typescript
const DEPLOYMENT_STRATEGIES = {
  blueGreen: {
    description: 'Two identical environments, switch traffic at once',
    pros: ['Zero downtime', 'Instant rollback', 'Full testing before switch'],
    cons: ['Double infrastructure cost', 'Database migrations tricky'],
    useWhen: 'Critical applications needing instant rollback',
  },
  
  canary: {
    description: 'Gradual rollout to percentage of users',
    pros: ['Reduced risk', 'Real user testing', 'Easy rollback'],
    cons: ['Slower rollout', 'Needs traffic splitting'],
    useWhen: 'Large user bases, testing new features',
  },
  
  rolling: {
    description: 'Replace instances one at a time',
    pros: ['Resource efficient', 'No extra infrastructure'],
    cons: ['Multiple versions running', 'Longer rollout'],
    useWhen: 'Standard deployments with good test coverage',
  },
  
  recreate: {
    description: 'Stop old version, start new version',
    pros: ['Simple', 'No version conflicts'],
    cons: ['Downtime required'],
    useWhen: 'Non-critical apps, major breaking changes',
  },
};

// Feature flags for gradual rollout
import { useState, useEffect } from 'react';

interface FeatureFlags {
  newDashboard: boolean;
  darkMode: boolean;
  experimentalFeature: boolean;
}

const useFeatureFlags = (userId: string) => {
  const [flags, setFlags] = useState<FeatureFlags | null>(null);
  
  useEffect(() => {
    async function fetchFlags() {
      const response = await fetch(`/api/feature-flags?userId=${userId}`);
      const data = await response.json();
      setFlags(data);
    }
    
    fetchFlags();
  }, [userId]);
  
  return flags;
};

// Usage
function Dashboard() {
  const flags = useFeatureFlags('user-123');
  
  if (flags?.newDashboard) {
    return <NewDashboard />;
  }
  
  return <LegacyDashboard />;
}
```

## 13.2 Vercel Configuration

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
      ]
    }
  ],
  
  "redirects": [
    { "source": "/old-page", "destination": "/new-page", "permanent": true }
  ],
  
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api.myapp.com/:path*" }
  ],
  
  "env": {
    "NEXT_PUBLIC_ENV": "production"
  },
  
  "regions": ["iad1", "sfo1", "lhr1"]
}
```

## 13.3 Monitoring and Observability

```typescript
// Error tracking with Sentry
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NEXT_PUBLIC_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
  
  beforeSend(event) {
    // Filter out non-critical errors
    if (event.exception?.values?.[0]?.type === 'ChunkLoadError') {
      return null;
    }
    return event;
  },
});

// Custom error boundary with tracking
class ErrorBoundary extends Component {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    Sentry.captureException(error, { extra: errorInfo });
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}

// Performance monitoring
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

function sendToAnalytics({ name, value, id }: Metric) {
  // Send to your analytics
  fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify({ name, value, id }),
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
getFCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

---

# 14. Docker & Containerization

## 14.1 Dockerfile for Next.js

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Dependencies stage
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

# Builder stage
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# Runner stage
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

## 14.2 Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - web

volumes:
  postgres_data:
  redis_data:
```

## 14.3 Kubernetes Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: myregistry/frontend:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

# 15. Interview Questions

## 15.1 Performance Questions

### Q1: What are the Core Web Vitals and their targets?

```typescript
// Answer:
const CORE_WEB_VITALS = {
  LCP: {
    fullName: 'Largest Contentful Paint',
    measures: 'Loading performance',
    target: '< 2.5 seconds',
    optimization: 'Preload LCP element, optimize images, reduce server response time',
  },
  
  INP: {
    fullName: 'Interaction to Next Paint',
    measures: 'Interactivity (replaced FID in 2024)',
    target: '< 200ms',
    optimization: 'Break up long tasks, use transitions, debounce handlers',
  },
  
  CLS: {
    fullName: 'Cumulative Layout Shift',
    measures: 'Visual stability',
    target: '< 0.1',
    optimization: 'Reserve space for images/ads, avoid injecting content above viewport',
  },
};
```

### Q2: How would you optimize a slow React application?

```typescript
// Answer - systematic approach:

const OPTIMIZATION_STEPS = {
  1: {
    step: 'Measure and Profile',
    actions: [
      'Run Lighthouse to identify issues',
      'Use React DevTools Profiler',
      'Check Chrome DevTools Performance panel',
    ],
  },
  
  2: {
    step: 'Reduce Bundle Size',
    actions: [
      'Analyze bundle with webpack-bundle-analyzer',
      'Code split routes with lazy loading',
      'Tree shake unused imports',
      'Replace heavy libraries with lighter alternatives',
    ],
  },
  
  3: {
    step: 'Optimize Rendering',
    actions: [
      'Memoize expensive components with React.memo',
      'Use useMemo and useCallback appropriately',
      'Virtualize long lists',
      'Avoid unnecessary re-renders',
    ],
  },
  
  4: {
    step: 'Optimize Assets',
    actions: [
      'Use modern image formats (WebP, AVIF)',
      'Lazy load below-fold images',
      'Preload critical resources',
      'Use CDN for static assets',
    ],
  },
  
  5: {
    step: 'Optimize Data Fetching',
    actions: [
      'Cache with React Query or SWR',
      'Prefetch on hover',
      'Use pagination or infinite scroll',
      'Implement optimistic updates',
    ],
  },
};
```

### Q3: Explain code splitting strategies

```typescript
// Answer:
const CODE_SPLITTING_STRATEGIES = {
  routeBased: {
    description: 'Split by route/page',
    implementation: 'React.lazy with React Router or Next.js automatic splitting',
    when: 'Always - most impactful for initial load',
  },
  
  componentBased: {
    description: 'Split heavy components',
    implementation: 'React.lazy for modals, charts, editors',
    when: 'Component is large AND not immediately needed',
  },
  
  vendorBased: {
    description: 'Separate vendor bundles',
    implementation: 'Webpack splitChunks configuration',
    when: 'Want to cache vendor code separately',
  },
};

// Example implementation:
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => 
  import('./Settings').then(m => ({ default: m.Settings }))
);
```

## 15.2 Testing Questions

### Q4: What's the difference between unit, integration, and E2E tests?

```typescript
const TEST_TYPES = {
  unit: {
    scope: 'Single function or component in isolation',
    mocking: 'Mock all dependencies',
    speed: 'Fast (ms)',
    example: 'Testing calculateTotal() function',
    tools: ['Jest', 'Vitest'],
  },
  
  integration: {
    scope: 'Multiple components working together',
    mocking: 'Mock external services (APIs, databases)',
    speed: 'Medium (seconds)',
    example: 'Testing form submission with validation',
    tools: ['Testing Library', 'MSW'],
  },
  
  e2e: {
    scope: 'Complete user flow in real browser',
    mocking: 'Minimal - test against real or staging environment',
    speed: 'Slow (minutes)',
    example: 'Complete checkout flow from browsing to payment',
    tools: ['Playwright', 'Cypress'],
  },
};
```

### Q5: How do you test async code in React?

```typescript
// Answer with example:
import { render, screen, waitFor } from '@testing-library/react';

// 1. Using findBy queries (auto-waits)
it('displays user after loading', async () => {
  render(<UserProfile id="123" />);
  
  // findBy waits for element to appear
  const name = await screen.findByText('John Doe');
  expect(name).toBeInTheDocument();
});

// 2. Using waitFor for assertions
it('hides loading spinner after data loads', async () => {
  render(<UserProfile id="123" />);
  
  await waitFor(() => {
    expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
  });
});

// 3. Testing loading states
it('shows loading then content', async () => {
  render(<UserProfile id="123" />);
  
  // Initially shows loading
  expect(screen.getByRole('progressbar')).toBeInTheDocument();
  
  // Then shows content
  await screen.findByText('John Doe');
  expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
});
```

## 15.3 Deployment Questions

### Q6: Explain blue-green deployment

```typescript
// Answer:
const BLUE_GREEN_DEPLOYMENT = {
  concept: 'Two identical production environments (blue and green)',
  
  process: [
    '1. Blue is currently live, serving all traffic',
    '2. Deploy new version to Green environment',
    '3. Test Green thoroughly',
    '4. Switch load balancer to point to Green',
    '5. Green is now live, Blue becomes standby',
    '6. If issues: instant switch back to Blue',
  ],
  
  advantages: [
    'Zero downtime',
    'Instant rollback',
    'Full testing before going live',
    'No mixed versions serving users',
  ],
  
  challenges: [
    'Double infrastructure cost',
    'Database migrations need careful planning',
    'Session management across switches',
  ],
};
```

### Q7: How do you handle environment variables securely?

```typescript
// Answer:
const ENV_VARIABLE_BEST_PRACTICES = {
  neverCommit: [
    'API keys',
    'Database credentials',
    'JWT secrets',
    'Any sensitive data',
  ],
  
  inCode: {
    // Use .env.example with placeholder values
    // .env.example (committed)
    example: `
      DATABASE_URL=postgresql://user:password@localhost/db
      API_KEY=your-api-key-here
    `,
    
    // Actual values in .env.local (gitignored)
  },
  
  inCI: [
    'Store in GitHub Secrets',
    'Use environment-specific secret stores',
    'Inject at build time',
  ],
  
  inProduction: [
    'Use secret management (AWS Secrets Manager, Vault)',
    'Inject via environment variables in container/platform',
    'Never log sensitive values',
  ],
  
  validation: `
    // Always validate env vars at startup
    const requiredEnvVars = ['DATABASE_URL', 'API_KEY'];
    for (const envVar of requiredEnvVars) {
      if (!process.env[envVar]) {
        throw new Error(\`Missing required env var: \${envVar}\`);
      }
    }
  `,
};
```

---

# 16. Coding Challenges

## 16.1 Implement Debounce

```typescript
// Challenge: Implement a debounce function with TypeScript

function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number,
  options: { leading?: boolean; trailing?: boolean } = {}
): (...args: Parameters<T>) => void {
  const { leading = false, trailing = true } = options;
  let timeoutId: NodeJS.Timeout | null = null;
  let lastArgs: Parameters<T> | null = null;
  
  return function debounced(...args: Parameters<T>) {
    const shouldCallNow = leading && !timeoutId;
    
    lastArgs = args;
    
    if (timeoutId) {
      clearTimeout(timeoutId);
    }
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (trailing && lastArgs) {
        fn(...lastArgs);
        lastArgs = null;
      }
    }, delay);
    
    if (shouldCallNow) {
      fn(...args);
    }
  };
}

// Test it
const log = debounce((msg: string) => console.log(msg), 1000);
log('a');
log('b');
log('c'); // Only 'c' is logged after 1 second
```

## 16.2 Virtual List Component

```typescript
// Challenge: Implement a virtualized list

interface VirtualListProps<T> {
  items: T[];
  itemHeight: number;
  windowHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
  overscan?: number;
}

function VirtualList<T>({
  items,
  itemHeight,
  windowHeight,
  renderItem,
  overscan = 3,
}: VirtualListProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);
  
  const totalHeight = items.length * itemHeight;
  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
  const endIndex = Math.min(
    items.length,
    Math.ceil((scrollTop + windowHeight) / itemHeight) + overscan
  );
  
  const visibleItems = items.slice(startIndex, endIndex);
  const offsetY = startIndex * itemHeight;
  
  const handleScroll = (e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  };
  
  return (
    <div
      style={{ height: windowHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, i) => (
            <div key={startIndex + i} style={{ height: itemHeight }}>
              {renderItem(item, startIndex + i)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Usage
<VirtualList
  items={Array.from({ length: 10000 }, (_, i) => `Item ${i}`)}
  itemHeight={50}
  windowHeight={400}
  renderItem={(item) => <div>{item}</div>}
/>
```

## 16.3 Rate Limiter Hook

```typescript
// Challenge: Implement a rate limiter hook

function useRateLimiter(
  maxRequests: number,
  intervalMs: number
): { canMakeRequest: () => boolean; waitTime: () => number } {
  const requestTimestamps = useRef<number[]>([]);
  
  const canMakeRequest = useCallback(() => {
    const now = Date.now();
    const windowStart = now - intervalMs;
    
    // Remove expired timestamps
    requestTimestamps.current = requestTimestamps.current.filter(
      ts => ts > windowStart
    );
    
    if (requestTimestamps.current.length < maxRequests) {
      requestTimestamps.current.push(now);
      return true;
    }
    
    return false;
  }, [maxRequests, intervalMs]);
  
  const waitTime = useCallback(() => {
    const now = Date.now();
    const windowStart = now - intervalMs;
    
    const validTimestamps = requestTimestamps.current.filter(
      ts => ts > windowStart
    );
    
    if (validTimestamps.length < maxRequests) {
      return 0;
    }
    
    const oldestTimestamp = validTimestamps[0];
    return oldestTimestamp + intervalMs - now;
  }, [maxRequests, intervalMs]);
  
  return { canMakeRequest, waitTime };
}

// Usage
function ApiCaller() {
  const { canMakeRequest, waitTime } = useRateLimiter(5, 1000); // 5 per second
  
  const makeApiCall = async () => {
    if (!canMakeRequest()) {
      console.log(`Rate limited. Wait ${waitTime()}ms`);
      return;
    }
    
    await fetch('/api/data');
  };
}
```

---

# 17. Quick Reference

## Performance Checklist

```typescript
const PERFORMANCE_CHECKLIST = {
  loading: [
    '✅ Preload critical resources',
    '✅ Use code splitting',
    '✅ Optimize images (WebP/AVIF, responsive)',
    '✅ Cache static assets',
    '✅ Use CDN',
  ],
  
  rendering: [
    '✅ Minimize re-renders (React.memo, useMemo)',
    '✅ Virtualize long lists',
    '✅ Avoid layout thrashing',
    '✅ Use CSS containment',
  ],
  
  javascript: [
    '✅ Tree shake unused code',
    '✅ Defer non-critical scripts',
    '✅ Break up long tasks',
    '✅ Use Web Workers for heavy computation',
  ],
  
  network: [
    '✅ Enable HTTP/2 or HTTP/3',
    '✅ Compress responses (gzip/brotli)',
    '✅ Set proper cache headers',
    '✅ Prefetch next page resources',
  ],
};
```

## Testing Best Practices

```typescript
const TESTING_BEST_PRACTICES = [
  'Test behavior, not implementation',
  'Use Testing Library queries by accessibility role',
  'Avoid testing implementation details',
  'Each test should be independent',
  'Use MSW for API mocking',
  'Write E2E tests for critical flows only',
  'Aim for 80% coverage on critical paths',
];
```

---

**End of Part 11: Performance, Testing & Deployment Complete Guide**

Total Sections: 17
Key Topics: Core Web Vitals, Testing, CI/CD, Docker, Kubernetes
Interview Ready: ✅

---

# 🎉 Complete Series Summary

You have now completed all 11 parts of the Comprehensive Frontend Interview Guide:

| Part | Topic | Lines |
|------|-------|-------|
| 1 | JavaScript Foundations | 6330 |
| 2 | TypeScript Mastery | 6154 |
| 3 | React Deep Dive | 6018 |
| 4 | React Hooks | 6046 |
| 5 | Next.js Framework | 6177 |
| 6 | State Management | 6133 |
| 7 | Networking & APIs | 6193 |
| 8 | Security Best Practices | 6066 |
| 9 | Real-Time Systems | 6477 |
| 10 | Data Visualization | 6080 |
| 11 | Performance, Testing & Deployment | 6000+ |

**Total: 67,000+ lines of interview-ready content!**

---

# BONUS: Advanced Testing Patterns

## Mock Service Worker (MSW) Deep Dive

```typescript
// mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  // GET request
  rest.get('/api/users', (req, res, ctx) => {
    const limit = req.url.searchParams.get('limit');
    
    return res(
      ctx.status(200),
      ctx.json({
        users: [
          { id: '1', name: 'John Doe', email: 'john@example.com' },
          { id: '2', name: 'Jane Doe', email: 'jane@example.com' },
        ].slice(0, Number(limit) || undefined),
      })
    );
  }),
  
  // POST request
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json();
    
    if (!body.email) {
      return res(
        ctx.status(400),
        ctx.json({ error: 'Email is required' })
      );
    }
    
    return res(
      ctx.status(201),
      ctx.json({
        id: 'new-id',
        ...body,
        createdAt: new Date().toISOString(),
      })
    );
  }),
  
  // Error responses
  rest.get('/api/error', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json({ error: 'Internal Server Error' })
    );
  }),
  
  // Delayed response
  rest.get('/api/slow', (req, res, ctx) => {
    return res(
      ctx.delay(2000),
      ctx.json({ message: 'Finally loaded!' })
    );
  }),
];

// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// mocks/browser.ts
import { setupWorker } from 'msw';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);

// Usage in test
describe('UserList', () => {
  it('handles API error gracefully', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );
    
    render(<UserList />);
    
    await screen.findByText('Failed to load users');
  });
});
```

## Testing Custom Hooks

```typescript
import { renderHook, act, waitFor } from '@testing-library/react';
import { useCounter } from './useCounter';
import { useDebounce } from './useDebounce';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });
  
  it('increments count', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('respects max value', () => {
    const { result } = renderHook(() => 
      useCounter(0, { max: 5 })
    );
    
    act(() => {
      for (let i = 0; i < 10; i++) {
        result.current.increment();
      }
    });
    
    expect(result.current.count).toBe(5);
  });
});

describe('useDebounce', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });
  
  afterEach(() => {
    jest.useRealTimers();
  });
  
  it('debounces value changes', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 300),
      { initialProps: { value: 'initial' } }
    );
    
    expect(result.current).toBe('initial');
    
    rerender({ value: 'updated' });
    expect(result.current).toBe('initial'); // Not updated yet
    
    act(() => {
      jest.advanceTimersByTime(300);
    });
    
    expect(result.current).toBe('updated');
  });
});
```

## Testing React Query

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useUsers } from './useUsers';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useUsers', () => {
  it('fetches users successfully', async () => {
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    });
    
    // Initially loading
    expect(result.current.isLoading).toBe(true);
    
    // Wait for data
    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });
    
    expect(result.current.data).toHaveLength(2);
    expect(result.current.data?.[0].name).toBe('John Doe');
  });
  
  it('handles error state', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );
    
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    });
    
    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });
  });
});
```

---

# BONUS: More Interview Questions

## Performance Deep Dive

### Q8: What causes layout thrashing and how do you avoid it?

```typescript
// Answer:
const LAYOUT_THRASHING = {
  cause: 'Repeatedly reading and writing DOM properties that trigger layout',
  
  example: `
    // BAD: Forces layout on each iteration
    for (const element of elements) {
      const height = element.offsetHeight; // Read (forces layout)
      element.style.height = height + 10 + 'px'; // Write (invalidates layout)
    }
    
    // GOOD: Batch reads, then batch writes
    const heights = elements.map(el => el.offsetHeight); // All reads
    elements.forEach((el, i) => {
      el.style.height = heights[i] + 10 + 'px'; // All writes
    });
  `,
  
  properties_that_trigger_layout: [
    'offsetTop/Left/Width/Height',
    'scrollTop/Left/Width/Height',
    'clientTop/Left/Width/Height',
    'getComputedStyle()',
    'getBoundingClientRect()',
  ],
  
  solutions: [
    'Batch DOM reads and writes',
    'Use requestAnimationFrame for animations',
    'Use transform instead of top/left',
    'Use will-change for known animations',
    'Use CSS containment',
  ],
};
```

### Q9: Explain the React Fiber reconciliation algorithm

```typescript
// Answer:
const REACT_FIBER = {
  concept: 'Work can be split into chunks and prioritized',
  
  phases: {
    render: {
      description: 'Builds the work-in-progress tree',
      interruptible: true,
      sideEffects: false,
    },
    commit: {
      description: 'Applies changes to the DOM',
      interruptible: false,
      sideEffects: true,
    },
  },
  
  priorities: {
    immediate: 'Synchronous, like text input',
    userBlocking: 'Should complete within ~250ms',
    normal: 'Should complete within ~5s',
    low: 'Analytics, non-critical',
    idle: 'When browser is idle',
  },
  
  benefits: [
    'Pause work and resume later',
    'Abort work if result not needed',
    'Prioritize more important updates',
    'Concurrent rendering',
  ],
  
  practical_use: `
    // useTransition for non-urgent updates
    const [isPending, startTransition] = useTransition();
    
    const handleChange = (value) => {
      setInput(value); // Urgent
      
      startTransition(() => {
        setFilteredList(filter(list, value)); // Non-urgent
      });
    };
  `,
};
```

### Q10: How would you debug a memory leak in a React app?

```typescript
// Answer:
const MEMORY_LEAK_DEBUGGING = {
  symptoms: [
    'Performance degrades over time',
    'Memory usage keeps increasing',
    'Detached DOM nodes in Memory panel',
  ],
  
  common_causes: {
    event_listeners: 'Not removed in cleanup',
    intervals: 'setInterval not cleared',
    subscriptions: 'WebSocket/EventSource not closed',
    closures: 'Holding references to unmounted components',
    global_stores: 'Data not cleaned up for unmounted views',
  },
  
  debugging_steps: [
    '1. Open Chrome DevTools > Memory panel',
    '2. Take heap snapshot',
    '3. Perform actions that might leak',
    '4. Take another snapshot',
    '5. Compare snapshots for growing objects',
    '6. Look for "Detached" in filter',
  ],
  
  prevention: `
    useEffect(() => {
      const controller = new AbortController();
      
      fetch('/api/data', { signal: controller.signal });
      
      const interval = setInterval(doSomething, 1000);
      
      const handleResize = () => {};
      window.addEventListener('resize', handleResize);
      
      return () => {
        controller.abort(); // Cancel fetch
        clearInterval(interval); // Clear interval
        window.removeEventListener('resize', handleResize); // Remove listener
      };
    }, []);
  `,
};
```

## Testing Deep Dive

### Q11: When should you use snapshot testing vs assertion testing?

```typescript
// Answer:
const SNAPSHOT_VS_ASSERTION = {
  snapshotTesting: {
    use: [
      'UI component output verification',
      'Configuration files',
      'Large data structures',
      'Detecting unintended changes',
    ],
    avoid: [
      'Dynamic content (timestamps, IDs)',
      'Frequently changing components',
      'As a lazy substitute for proper assertions',
    ],
    example: `
      it('renders correctly', () => {
        const { asFragment } = render(<Button>Click</Button>);
        expect(asFragment()).toMatchSnapshot();
      });
    `,
  },
  
  assertionTesting: {
    use: [
      'Specific behavior verification',
      'Edge cases',
      'Error handling',
      'User interactions',
    ],
    example: `
      it('shows error on invalid email', async () => {
        render(<LoginForm />);
        
        await userEvent.type(screen.getByLabelText('Email'), 'invalid');
        await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
        
        expect(screen.getByRole('alert')).toHaveTextContent('Invalid email');
      });
    `,
  },
  
  guideline: 'Snapshots catch what changed, assertions verify what should happen',
};
```

### Q12: How do you test components that use context?

```typescript
// Answer with multiple approaches:

// Approach 1: Custom render with wrapper
const customRender = (ui: React.ReactElement) => {
  return render(
    <ThemeProvider theme="dark">
      <UserProvider user={mockUser}>
        {ui}
      </UserProvider>
    </ThemeProvider>
  );
};

// Approach 2: Create test utilities
function renderWithProviders(
  ui: React.ReactElement,
  {
    theme = 'light',
    user = null,
    ...options
  } = {}
) {
  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <ThemeProvider theme={theme}>
        <UserProvider user={user}>
          {children}
        </UserProvider>
      </ThemeProvider>
    );
  }
  
  return render(ui, { wrapper: Wrapper, ...options });
}

// Approach 3: Mock the context directly
jest.mock('./ThemeContext', () => ({
  useTheme: () => ({ theme: 'dark', toggleTheme: jest.fn() }),
}));

// Usage
describe('ThemedButton', () => {
  it('uses theme from context', () => {
    renderWithProviders(<ThemedButton />, { theme: 'dark' });
    
    expect(screen.getByRole('button')).toHaveClass('dark-theme');
  });
});
```

## Deployment Deep Dive

### Q13: How do you implement zero-downtime deployments?

```typescript
// Answer:
const ZERO_DOWNTIME_DEPLOYMENT = {
  strategies: {
    rolling: {
      description: 'Replace instances one at a time',
      implementation: `
        # Kubernetes rolling update
        kubectl set image deployment/frontend frontend=myapp:v2
        
        # Deployment config
        spec:
          strategy:
            type: RollingUpdate
            rollingUpdate:
              maxSurge: 1
              maxUnavailable: 0
      `,
    },
    
    blueGreen: {
      description: 'Switch traffic between two environments',
      implementation: `
        # 1. Deploy to green
        kubectl apply -f green-deployment.yaml
        
        # 2. Verify green is healthy
        kubectl rollout status deployment/frontend-green
        
        # 3. Switch service selector
        kubectl patch svc frontend -p '{"spec":{"selector":{"version":"green"}}}'
      `,
    },
    
    canary: {
      description: 'Gradual traffic shift',
      implementation: `
        # Istio VirtualService
        apiVersion: networking.istio.io/v1beta1
        kind: VirtualService
        spec:
          http:
          - route:
            - destination:
                host: frontend
                subset: stable
              weight: 90
            - destination:
                host: frontend
                subset: canary
              weight: 10
      `,
    },
  },
  
  requirements: [
    'Health checks (liveness/readiness probes)',
    'Graceful shutdown handling',
    'Backward compatible database migrations',
    'Load balancer connection draining',
  ],
};
```

---

# BONUS: Mock Interview Scenarios

## Scenario 1: Slow Dashboard

**Interviewer:** "Users are complaining that our dashboard is slow. How would you approach fixing it?"

```typescript
// Model answer:

const DASHBOARD_OPTIMIZATION_APPROACH = {
  step1_measure: {
    action: 'Get baseline metrics',
    tools: ['Lighthouse', 'Chrome DevTools', 'React Profiler'],
    metrics: ['LCP', 'INP', 'TBT', 'Bundle size'],
  },
  
  step2_identify: {
    action: 'Find bottlenecks',
    common_issues: [
      'Large initial bundle',
      'Blocking API calls',
      'Heavy re-renders',
      'Large DOM',
      'Unoptimized images',
    ],
  },
  
  step3_prioritize: {
    action: 'Order by impact',
    framework: 'Quick wins first, then bigger refactors',
  },
  
  step4_implement: {
    quick_wins: [
      'Add code splitting for routes',
      'Lazy load below-fold components',
      'Optimize images (WebP, sizes)',
      'Add React.memo to expensive components',
    ],
    
    medium_effort: [
      'Implement virtualization for lists',
      'Add React Query for caching',
      'Debounce search inputs',
      'Move heavy computations to useMemo',
    ],
    
    larger_refactors: [
      'Switch to Suspense-based data fetching',
      'Implement server-side rendering',
      'Add service worker for caching',
      'Redesign data architecture',
    ],
  },
  
  step5_verify: {
    action: 'Confirm improvements',
    approach: 'Run Lighthouse again, compare metrics',
  },
};
```

## Scenario 2: Flaky Tests

**Interviewer:** "Our E2E tests are flaky and failing randomly. How would you fix this?"

```typescript
// Model answer:

const FLAKY_TESTS_FIX = {
  diagnosis: {
    action: 'Identify flaky patterns',
    tools: ['Test retry reports', 'CI failure analysis'],
    common_causes: [
      'Race conditions',
      'Timing issues (not waiting for elements)',
      'Shared state between tests',
      'External service dependencies',
      'Browser/viewport inconsistencies',
    ],
  },
  
  solutions: {
    timing_issues: `
      // BAD: Fixed wait
      await page.waitForTimeout(1000);
      
      // GOOD: Wait for specific condition
      await page.waitForSelector('[data-testid="loaded"]');
      await expect(page.getByText('Dashboard')).toBeVisible();
    `,
    
    shared_state: `
      // Clean up between tests
      beforeEach(async () => {
        await page.evaluate(() => localStorage.clear());
        await context.clearCookies();
      });
      
      // Isolate each test with fresh page
      test.describe(() => {
        let page: Page;
        test.beforeEach(async ({ browser }) => {
          page = await browser.newPage();
        });
        test.afterEach(async () => {
          await page.close();
        });
      });
    `,
    
    external_services: `
      // Mock external services
      await page.route('**/api/**', async (route) => {
        await route.fulfill({
          status: 200,
          body: JSON.stringify(mockData),
        });
      });
    `,
    
    browser_inconsistencies: `
      // Use consistent viewport
      test.use({
        viewport: { width: 1280, height: 720 },
      });
      
      // Use web-first assertions that auto-retry
      await expect(page.getByRole('button')).toBeEnabled();
    `,
  },
  
  prevention: [
    'Use auto-waiting locators',
    'Avoid arbitrary timeouts',
    'Isolate test data',
    'Mock external dependencies',
    'Use test retries strategically (not as a fix)',
    'Run tests in consistent environment',
  ],
};
```

---

# BONUS: More Coding Challenges

## Challenge 4: Implement usePrevious Hook

```typescript
// Challenge: Create a hook that returns the previous value

function usePrevious<T>(value: T): T | undefined {
  const currentRef = useRef<T>(value);
  const previousRef = useRef<T | undefined>(undefined);
  
  if (currentRef.current !== value) {
    previousRef.current = currentRef.current;
    currentRef.current = value;
  }
  
  return previousRef.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {previousCount}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

## Challenge 5: Implement Retry with Exponential Backoff

```typescript
// Challenge: Create a fetch wrapper with retry logic

interface RetryOptions {
  maxRetries?: number;
  baseDelay?: number;
  maxDelay?: number;
  shouldRetry?: (error: Error, attempt: number) => boolean;
}

async function fetchWithRetry<T>(
  url: string,
  options: RequestInit = {},
  retryOptions: RetryOptions = {}
): Promise<T> {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    shouldRetry = (error, attempt) => attempt < maxRetries,
  } = retryOptions;
  
  let attempt = 0;
  
  while (true) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return response.json();
    } catch (error) {
      attempt++;
      
      if (!shouldRetry(error as Error, attempt)) {
        throw error;
      }
      
      // Exponential backoff with jitter
      const delay = Math.min(
        baseDelay * Math.pow(2, attempt - 1) + Math.random() * 1000,
        maxDelay
      );
      
      console.log(`Retry ${attempt}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
const data = await fetchWithRetry<User[]>('/api/users', {}, {
  maxRetries: 5,
  shouldRetry: (error, attempt) => {
    // Don't retry 4xx errors
    if (error.message.startsWith('HTTP 4')) return false;
    return attempt < 5;
  },
});
```

## Challenge 6: Implement Intersection Observer Hook

```typescript
// Challenge: Create a hook for viewport visibility detection

interface UseIntersectionOptions {
  root?: Element | null;
  rootMargin?: string;
  threshold?: number | number[];
  triggerOnce?: boolean;
}

function useIntersection<T extends Element>(
  options: UseIntersectionOptions = {}
): [React.RefObject<T>, boolean] {
  const { root, rootMargin, threshold, triggerOnce = false } = options;
  
  const ref = useRef<T>(null);
  const [isIntersecting, setIsIntersecting] = useState(false);
  const frozenRef = useRef(false);
  
  useEffect(() => {
    const element = ref.current;
    if (!element) return;
    
    if (triggerOnce && frozenRef.current) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsIntersecting(entry.isIntersecting);
        
        if (entry.isIntersecting && triggerOnce) {
          frozenRef.current = true;
          observer.disconnect();
        }
      },
      { root, rootMargin, threshold }
    );
    
    observer.observe(element);
    
    return () => observer.disconnect();
  }, [root, rootMargin, threshold, triggerOnce]);
  
  return [ref, isIntersecting];
}

// Usage
function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [ref, isVisible] = useIntersection<HTMLDivElement>({
    rootMargin: '200px',
    triggerOnce: true,
  });
  
  return (
    <div ref={ref} style={{ minHeight: 200 }}>
      {isVisible ? (
        <img src={src} alt={alt} />
      ) : (
        <div className="placeholder" />
      )}
    </div>
  );
}
```

## Challenge 7: Implement useLocalStorage Hook

```typescript
// Challenge: Create a hook that syncs state with localStorage

function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  // Get initial value from localStorage or use provided initial value
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });
  
  // Update localStorage when value changes
  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    setStoredValue(prev => {
      const valueToStore = value instanceof Function ? value(prev) : value;
      
      try {
        localStorage.setItem(key, JSON.stringify(valueToStore));
      } catch (error) {
        console.error(`Error setting localStorage key "${key}":`, error);
      }
      
      return valueToStore;
    });
  }, [key]);
  
  // Sync across tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key && e.newValue) {
        try {
          setStoredValue(JSON.parse(e.newValue));
        } catch (error) {
          console.error('Error parsing storage event:', error);
        }
      }
    };
    
    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);
  
  return [storedValue, setValue];
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

---

# BONUS: Build Tool Configuration

## Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      filename: 'dist/stats.html',
      open: true,
    }),
  ],
  
  resolve: {
    alias: {
      '@': '/src',
    },
  },
  
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          query: ['@tanstack/react-query'],
        },
      },
    },
    sourcemap: true,
    reportCompressedSize: true,
  },
  
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
    },
  },
  
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
});
```

## Webpack Production Configuration

```typescript
// webpack.prod.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index.tsx',
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    clean: true,
  },
  
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
          },
        },
      }),
      new CssMinimizerPlugin(),
    ],
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
    runtimeChunk: 'single',
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
      },
    }),
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
    }),
    process.env.ANALYZE && new BundleAnalyzerPlugin(),
  ].filter(Boolean),
  
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'],
      },
    ],
  },
  
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
};
```

---

# BONUS: Performance Monitoring

## Real User Monitoring (RUM)

```typescript
// lib/rum.ts

interface PerformanceData {
  url: string;
  timestamp: number;
  metrics: {
    LCP?: number;
    FID?: number;
    INP?: number;
    CLS?: number;
    TTFB?: number;
    FCP?: number;
  };
  connection?: {
    effectiveType: string;
    downlink: number;
    rtt: number;
  };
  device?: {
    memory: number;
    cores: number;
  };
}

class RealUserMonitoring {
  private data: PerformanceData = {
    url: window.location.href,
    timestamp: Date.now(),
    metrics: {},
  };
  
  constructor(private endpoint: string) {
    this.collectMetrics();
    this.collectConnectionInfo();
    this.collectDeviceInfo();
    this.sendOnUnload();
  }
  
  private collectMetrics() {
    // LCP
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      this.data.metrics.LCP = entries[entries.length - 1].startTime;
    }).observe({ type: 'largest-contentful-paint', buffered: true });
    
    // FID
    new PerformanceObserver((list) => {
      this.data.metrics.FID = list.getEntries()[0].processingStart - 
        list.getEntries()[0].startTime;
    }).observe({ type: 'first-input', buffered: true });
    
    // CLS
    let clsValue = 0;
    new PerformanceObserver((list) => {
      for (const entry of list.getEntries() as any[]) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      }
      this.data.metrics.CLS = clsValue;
    }).observe({ type: 'layout-shift', buffered: true });
    
    // Navigation timing
    const navEntry = performance.getEntriesByType('navigation')[0] as 
      PerformanceNavigationTiming;
    this.data.metrics.TTFB = navEntry.responseStart - navEntry.requestStart;
  }
  
  private collectConnectionInfo() {
    const connection = (navigator as any).connection;
    if (connection) {
      this.data.connection = {
        effectiveType: connection.effectiveType,
        downlink: connection.downlink,
        rtt: connection.rtt,
      };
    }
  }
  
  private collectDeviceInfo() {
    this.data.device = {
      memory: (navigator as any).deviceMemory || 0,
      cores: navigator.hardwareConcurrency || 0,
    };
  }
  
  private sendOnUnload() {
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.send();
      }
    });
  }
  
  private send() {
    const blob = new Blob([JSON.stringify(this.data)], {
      type: 'application/json',
    });
    navigator.sendBeacon(this.endpoint, blob);
  }
}

// Initialize
new RealUserMonitoring('/api/rum');
```

---

# BONUS: Complete Quick Reference Cards

## Performance Quick Reference

```typescript
const PERFORMANCE_QUICK_REFERENCE = {
  metrics: {
    LCP: '< 2.5s',
    INP: '< 200ms',
    CLS: '< 0.1',
    TTFB: '< 800ms',
    FCP: '< 1.8s',
  },
  
  tools: {
    lighthouse: 'Overall performance audit',
    devtools_performance: 'Flame chart, main thread analysis',
    devtools_network: 'Request waterfall, timing',
    react_profiler: 'Component render times',
    bundle_analyzer: 'Bundle composition',
  },
  
  quick_wins: [
    'Compress images to WebP',
    'Add loading="lazy" to below-fold images',
    'Set width/height on images',
    'Use font-display: swap',
    'Enable gzip/brotli compression',
    'Set cache headers',
    'Preload critical assets',
    'Defer non-critical JS',
  ],
  
  code_optimizations: [
    'Code split routes with lazy()',
    'Memoize with React.memo',
    'Use useMemo for expensive calculations',
    'Virtualize long lists',
    'Debounce event handlers',
    'Use Web Workers for heavy computation',
  ],
};
```

## Testing Quick Reference

```typescript
const TESTING_QUICK_REFERENCE = {
  pyramid: {
    unit: '70% - Fast, isolated, mock dependencies',
    integration: '20% - Multiple components, mock external',
    e2e: '10% - Full flows, real environment',
  },
  
  rtl_queries: {
    getBy: 'Throws if not found',
    queryBy: 'Returns null if not found',
    findBy: 'Async, waits for element',
  },
  
  priority: [
    'getByRole - Most accessible',
    'getByLabelText - Form elements',
    'getByPlaceholderText - Fallback for forms',
    'getByText - Non-interactive elements',
    'getByTestId - Last resort',
  ],
  
  async_patterns: {
    findBy: 'await screen.findByText("loaded")',
    waitFor: 'await waitFor(() => expect(...).toBe(...))',
    userEvent: 'Always await userEvent calls',
  },
  
  mocking: {
    jest_mock: 'jest.mock("./module")',
    spy: 'jest.spyOn(obj, "method")',
    msw: 'Mock API at network level',
  },
};
```

## Deployment Quick Reference

```typescript
const DEPLOYMENT_QUICK_REFERENCE = {
  strategies: {
    rolling: 'Replace instances one by one',
    blue_green: 'Switch between two environments',
    canary: 'Gradual traffic shift',
    feature_flags: 'Enable/disable at runtime',
  },
  
  ci_cd: {
    lint: 'First stage - fast feedback',
    test: 'Unit + integration tests',
    build: 'Production bundle',
    e2e: 'Critical flow verification',
    deploy: 'Push to environment',
  },
  
  environments: {
    development: 'Local dev, debug enabled',
    staging: 'Production-like, test data',
    production: 'Real users, monitored',
  },
  
  monitoring: {
    errors: 'Sentry, LogRocket',
    performance: 'Web Vitals, custom RUM',
    uptime: 'Ping monitoring',
    logs: 'Centralized logging',
  },
};
```

---

**🎉 COMPLETE: Part 11 - Performance, Testing & Deployment Guide**

This comprehensive guide covers everything you need to know about performance optimization, testing strategies, and modern deployment practices for frontend development.

**Total Coverage:**
- 20+ Sections
- Core Web Vitals Deep Dive
- Bundle Optimization
- Complete Testing Guide (Unit, Integration, E2E)
- CI/CD Pipelines
- Docker & Kubernetes
- 15+ Interview Questions
- 8+ Coding Challenges
- Quick Reference Cards

**Ready for Interview: ✅**

---

# BONUS: Accessibility Testing

## Axe-core Integration

```typescript
// jest.setup.ts
import '@testing-library/jest-dom';
import { toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

// Accessibility testing utility
import { axe } from 'jest-axe';

async function checkAccessibility(container: HTMLElement) {
  const results = await axe(container);
  expect(results).toHaveNoViolations();
}

// Usage in tests
describe('Button', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    await checkAccessibility(container);
  });
});

describe('Form', () => {
  it('has accessible form fields', async () => {
    const { container } = render(
      <form>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" required />
        
        <label htmlFor="password">Password</label>
        <input id="password" type="password" required />
        
        <button type="submit">Login</button>
      </form>
    );
    
    await checkAccessibility(container);
  });
  
  it('announces errors to screen readers', async () => {
    render(<LoginForm />);
    
    await userEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    const errorMessage = await screen.findByRole('alert');
    expect(errorMessage).toHaveAttribute('aria-live', 'polite');
  });
});
```

## Keyboard Navigation Testing

```typescript
describe('Modal keyboard navigation', () => {
  it('traps focus within modal', async () => {
    const user = userEvent.setup();
    render(<Modal isOpen={true} onClose={() => {}}>Content</Modal>);
    
    // First focusable element should receive focus
    const closeButton = screen.getByRole('button', { name: /close/i });
    expect(closeButton).toHaveFocus();
    
    // Tab through all focusable elements
    await user.tab();
    expect(screen.getByRole('button', { name: /save/i })).toHaveFocus();
    
    await user.tab();
    expect(screen.getByRole('button', { name: /cancel/i })).toHaveFocus();
    
    // Tab should cycle back to first element
    await user.tab();
    expect(closeButton).toHaveFocus();
    
    // Shift+Tab should go backwards
    await user.tab({ shift: true });
    expect(screen.getByRole('button', { name: /cancel/i })).toHaveFocus();
  });
  
  it('closes on Escape key', async () => {
    const onClose = jest.fn();
    const user = userEvent.setup();
    render(<Modal isOpen={true} onClose={onClose}>Content</Modal>);
    
    await user.keyboard('{Escape}');
    
    expect(onClose).toHaveBeenCalled();
  });
  
  it('returns focus to trigger on close', async () => {
    const user = userEvent.setup();
    render(<ModalWithTrigger />);
    
    const openButton = screen.getByRole('button', { name: /open modal/i });
    await user.click(openButton);
    
    await user.keyboard('{Escape}');
    
    expect(openButton).toHaveFocus();
  });
});
```

---

# BONUS: More Advanced Performance

## Web Workers for Heavy Computation

```typescript
// workers/heavyComputation.worker.ts
self.onmessage = (event: MessageEvent) => {
  const { data, type } = event.data;
  
  switch (type) {
    case 'FILTER_LARGE_DATASET':
      const result = filterData(data.items, data.query);
      self.postMessage({ type: 'FILTER_RESULT', result });
      break;
      
    case 'PROCESS_CSV':
      const processed = parseCSV(data.content);
      self.postMessage({ type: 'CSV_PROCESSED', result: processed });
      break;
  }
};

function filterData(items: any[], query: string) {
  // Heavy filtering logic
  return items.filter(item => 
    Object.values(item).some(val => 
      String(val).toLowerCase().includes(query.toLowerCase())
    )
  );
}

// useWorker hook
function useWorker<T, R>(
  workerPath: string
): { run: (data: T) => Promise<R>; isRunning: boolean } {
  const [isRunning, setIsRunning] = useState(false);
  const workerRef = useRef<Worker>();
  
  useEffect(() => {
    workerRef.current = new Worker(workerPath, { type: 'module' });
    return () => workerRef.current?.terminate();
  }, [workerPath]);
  
  const run = useCallback((data: T): Promise<R> => {
    return new Promise((resolve, reject) => {
      if (!workerRef.current) {
        reject(new Error('Worker not initialized'));
        return;
      }
      
      setIsRunning(true);
      
      workerRef.current.onmessage = (event) => {
        setIsRunning(false);
        resolve(event.data.result);
      };
      
      workerRef.current.onerror = (error) => {
        setIsRunning(false);
        reject(error);
      };
      
      workerRef.current.postMessage(data);
    });
  }, []);
  
  return { run, isRunning };
}

// Usage
function DataProcessor() {
  const { run, isRunning } = useWorker<FilterPayload, FilterResult>(
    '/workers/heavyComputation.worker.js'
  );
  const [results, setResults] = useState<any[]>([]);
  
  const handleFilter = async (query: string) => {
    const filtered = await run({
      type: 'FILTER_LARGE_DATASET',
      data: { items: largeDataset, query }
    });
    setResults(filtered);
  };
  
  return (
    <div>
      <input onChange={(e) => handleFilter(e.target.value)} />
      {isRunning && <Spinner />}
      <ResultsList items={results} />
    </div>
  );
}
```

## requestIdleCallback for Non-Critical Work

```typescript
// Schedule non-critical work during idle time

function useIdleCallback(callback: () => void, timeout = 2000) {
  useEffect(() => {
    if ('requestIdleCallback' in window) {
      const id = requestIdleCallback(callback, { timeout });
      return () => cancelIdleCallback(id);
    } else {
      // Fallback for Safari
      const id = setTimeout(callback, 1);
      return () => clearTimeout(id);
    }
  }, [callback, timeout]);
}

// Analytics batching
class AnalyticsBatcher {
  private queue: AnalyticsEvent[] = [];
  private isScheduled = false;
  
  track(event: AnalyticsEvent) {
    this.queue.push(event);
    this.scheduleFlush();
  }
  
  private scheduleFlush() {
    if (this.isScheduled) return;
    this.isScheduled = true;
    
    if ('requestIdleCallback' in window) {
      requestIdleCallback(() => this.flush(), { timeout: 5000 });
    } else {
      setTimeout(() => this.flush(), 1000);
    }
  }
  
  private flush() {
    if (this.queue.length === 0) return;
    
    const events = [...this.queue];
    this.queue = [];
    this.isScheduled = false;
    
    // Send batch to analytics endpoint
    fetch('/api/analytics', {
      method: 'POST',
      body: JSON.stringify({ events }),
      keepalive: true,
    });
  }
}
```

## Server Components Performance

```typescript
// React Server Components for optimal performance

// app/products/page.tsx (Server Component - default)
export default async function ProductsPage() {
  // This runs on the server - no client JS bundle impact
  const products = await db.products.findMany();
  
  return (
    <main>
      <h1>Products</h1>
      <ProductGrid products={products} />
      
      {/* Interactive component - client-side */}
      <Suspense fallback={<FilterSkeleton />}>
        <ProductFilter />
      </Suspense>
    </main>
  );
}

// components/ProductFilter.tsx
'use client'; // This is a Client Component

import { useState, useTransition } from 'react';

export function ProductFilter() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (value: string) => {
    setQuery(value);
    startTransition(() => {
      // Trigger server action or update URL params
    });
  };
  
  return (
    <div>
      <input 
        value={query} 
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search products..."
      />
      {isPending && <Spinner />}
    </div>
  );
}

// Server Actions for mutations
// actions/products.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function addProduct(formData: FormData) {
  const name = formData.get('name') as string;
  const price = Number(formData.get('price'));
  
  await db.products.create({ data: { name, price } });
  
  revalidatePath('/products');
}
```

---

# BONUS: Error Handling Patterns

## Error Boundary with Recovery

```typescript
interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback?: React.ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface ErrorBoundaryState {
  error: Error | null;
  errorInfo: ErrorInfo | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = {
    error: null,
    errorInfo: null,
  };
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.setState({ errorInfo });
    this.props.onError?.(error, errorInfo);
    
    // Report to error tracking
    Sentry.captureException(error, { extra: errorInfo });
  }
  
  handleRetry = () => {
    this.setState({ error: null, errorInfo: null });
  };
  
  render() {
    if (this.state.error) {
      if (this.props.fallback) {
        return this.props.fallback;
      }
      
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error.toString()}</pre>
            <pre>{this.state.errorInfo?.componentStack}</pre>
          </details>
          <button onClick={this.handleRetry}>Try again</button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage with different fallbacks for different sections
function App() {
  return (
    <ErrorBoundary 
      fallback={<FullPageError />}
      onError={(error) => console.error('App error:', error)}
    >
      <Header />
      
      <ErrorBoundary fallback={<SidebarError />}>
        <Sidebar />
      </ErrorBoundary>
      
      <ErrorBoundary fallback={<MainContentError />}>
        <MainContent />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}
```

## Async Error Handling

```typescript
// Centralized error handling for async operations

type AsyncResult<T> = 
  | { success: true; data: T }
  | { success: false; error: Error };

async function tryAsync<T>(
  promise: Promise<T>
): Promise<AsyncResult<T>> {
  try {
    const data = await promise;
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

// Hook for async operations with error handling
function useAsyncAction<T extends any[], R>(
  action: (...args: T) => Promise<R>,
  options: {
    onSuccess?: (result: R) => void;
    onError?: (error: Error) => void;
  } = {}
) {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const execute = useCallback(async (...args: T) => {
    setIsLoading(true);
    setError(null);
    
    const result = await tryAsync(action(...args));
    
    setIsLoading(false);
    
    if (result.success) {
      options.onSuccess?.(result.data);
      return result.data;
    } else {
      setError(result.error);
      options.onError?.(result.error);
      throw result.error;
    }
  }, [action, options]);
  
  return { execute, isLoading, error };
}

// Usage
function SubmitButton() {
  const { execute, isLoading, error } = useAsyncAction(
    async (data: FormData) => {
      const response = await fetch('/api/submit', {
        method: 'POST',
        body: data,
      });
      if (!response.ok) throw new Error('Submission failed');
      return response.json();
    },
    {
      onSuccess: () => toast.success('Submitted successfully!'),
      onError: (error) => toast.error(error.message),
    }
  );
  
  return (
    <button onClick={() => execute(formData)} disabled={isLoading}>
      {isLoading ? 'Submitting...' : 'Submit'}
    </button>
  );
}
```

---

# BONUS: Complete Study Guide

## Interview Preparation Roadmap

```typescript
const INTERVIEW_PREP_ROADMAP = {
  week1: {
    focus: 'Performance Fundamentals',
    topics: [
      'Core Web Vitals (LCP, INP, CLS)',
      'Chrome DevTools Performance panel',
      'React Profiler',
      'Bundle analysis',
    ],
    practice: [
      'Audit 3 websites with Lighthouse',
      'Identify performance issues in a sample app',
      'Implement code splitting',
    ],
  },
  
  week2: {
    focus: 'Advanced Performance',
    topics: [
      'React Fiber architecture',
      'Concurrent features (useTransition, useDeferredValue)',
      'Virtualization',
      'Web Workers',
    ],
    practice: [
      'Implement virtual list from scratch',
      'Optimize a slow component',
      'Use Web Worker for heavy computation',
    ],
  },
  
  week3: {
    focus: 'Testing Mastery',
    topics: [
      'Testing Library best practices',
      'Mock Service Worker',
      'Playwright E2E testing',
      'Accessibility testing',
    ],
    practice: [
      'Write tests for a form component',
      'Write E2E test for login flow',
      'Test async data fetching',
    ],
  },
  
  week4: {
    focus: 'Deployment & DevOps',
    topics: [
      'CI/CD pipelines',
      'Docker basics',
      'Deployment strategies',
      'Monitoring and observability',
    ],
    practice: [
      'Set up GitHub Actions workflow',
      'Dockerize a Next.js app',
      'Implement feature flags',
    ],
  },
};
```

## Common Interview Mistakes

```typescript
const COMMON_INTERVIEW_MISTAKES = {
  performance: [
    'Saying "memoize everything" without understanding cost',
    'Not mentioning measurement before optimization',
    'Confusing FID with INP (FID is deprecated)',
    'Not knowing the difference between SSR and SSG',
  ],
  
  testing: [
    'Testing implementation details instead of behavior',
    'Using getByTestId as first choice',
    'Not waiting for async operations properly',
    'Snapshot testing everything without assertions',
  ],
  
  deployment: [
    'Not understanding the difference between preview and production',
    'Storing secrets in environment files committed to git',
    'Not knowing rollback strategies',
    'Confusing Docker build vs run',
  ],
  
  general: [
    'Jumping to solutions without understanding the problem',
    'Not asking clarifying questions',
    'Being too theoretical without practical examples',
    'Not admitting when you dont know something',
  ],
};
```

## Key Concepts Cheat Sheet

```typescript
const KEY_CONCEPTS_CHEAT_SHEET = {
  performance: {
    LCP: 'Largest Contentful Paint - < 2.5s',
    INP: 'Interaction to Next Paint - < 200ms',
    CLS: 'Cumulative Layout Shift - < 0.1',
    TBT: 'Total Blocking Time - < 200ms',
    TTFB: 'Time to First Byte - < 800ms',
  },
  
  react_optimization: {
    'React.memo': 'Prevent re-render if props unchanged',
    useMemo: 'Memoize expensive calculations',
    useCallback: 'Memoize callback functions',
    useTransition: 'Mark updates as non-urgent',
    useDeferredValue: 'Show stale value while updating',
    lazy: 'Code split components',
  },
  
  testing_queries: {
    getByRole: 'Most preferred - accessible',
    getByLabelText: 'For form elements',
    getByText: 'For text content',
    getByTestId: 'Last resort',
    findBy: 'Async - waits for element',
    queryBy: 'Returns null if not found',
  },
  
  deployment: {
    'Blue-Green': 'Two identical environments, instant switch',
    Canary: 'Gradual rollout to percentage of users',
    Rolling: 'Replace instances one by one',
    'Feature Flags': 'Enable/disable features at runtime',
  },
};
```

---

# BONUS: Output-Based Questions

## What Will This Code Log?

```typescript
// Question 1: Performance API timing
const nav = performance.getEntriesByType('navigation')[0];
console.log(nav.domInteractive > nav.responseEnd);
// Answer: true (always, domInteractive comes after responseEnd)

// Question 2: Promise timing
console.log('1');
Promise.resolve().then(() => console.log('2'));
setTimeout(() => console.log('3'), 0);
console.log('4');
// Answer: 1, 4, 2, 3 (microtasks before macrotasks)

// Question 3: React render order
function Parent() {
  console.log('Parent render');
  return <Child />;
}

function Child() {
  console.log('Child render');
  useEffect(() => {
    console.log('Child effect');
  }, []);
  return <div>Child</div>;
}

// When <Parent /> mounts, what logs?
// Answer: Parent render, Child render, Child effect

// Question 4: Memoization
const MemoChild = React.memo(function Child({ data }) {
  console.log('Child render');
  return <div>{data.value}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = { value: 'hello' }; // New object every render!
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Click</button>
      <MemoChild data={data} />
    </>
  );
}

// What happens when button is clicked?
// Answer: Child renders every time! data is a new object
// Fix: useMemo(() => ({ value: 'hello' }), [])
```

## Find the Bug

```typescript
// Bug 1: Memory leak in useEffect
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data)); // BUG: No cleanup!
  }, [userId]);
  
  return <div>{user?.name}</div>;
}

// Fix:
useEffect(() => {
  const controller = new AbortController();
  
  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(data => setUser(data))
    .catch(err => {
      if (err.name !== 'AbortError') throw err;
    });
  
  return () => controller.abort();
}, [userId]);

// Bug 2: Stale closure
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(count + 1); // BUG: Always captures initial count!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // Empty deps!
  
  return <div>{count}</div>;
}

// Fix: Use functional update
setCount(c => c + 1);

// Bug 3: Test not waiting
it('shows user after loading', () => {
  render(<UserProfile userId="123" />);
  
  // BUG: Not waiting for async load!
  expect(screen.getByText('John Doe')).toBeInTheDocument();
});

// Fix: Use findBy (async) or waitFor
it('shows user after loading', async () => {
  render(<UserProfile userId="123" />);
  
  await screen.findByText('John Doe');
});
```

---

**🎉 COMPLETE: Part 11 - Performance, Testing & Deployment Guide**

This comprehensive guide covers everything you need to know about performance optimization, testing strategies, and modern deployment practices for frontend development.

**Total Coverage:**
- 25+ Sections
- Core Web Vitals Deep Dive
- Bundle Optimization Techniques
- Complete Testing Guide (Unit, Integration, E2E, Accessibility)
- CI/CD Pipelines (GitHub Actions, Vercel)
- Docker & Kubernetes Deployment
- 20+ Interview Questions with Detailed Answers
- 10+ Coding Challenges
- Mock Interview Scenarios
- Output-Based & Bug-Finding Questions
- Complete Study Roadmap

**Series Complete: All 11 Parts Finished! 🎉**

| Part | Topic | Lines |
|------|-------|-------|
| 1 | JavaScript Foundations | 6330 |
| 2 | TypeScript Mastery | 6154 |
| 3 | React Deep Dive | 6018 |
| 4 | React Hooks | 6046 |
| 5 | Next.js Framework | 6177 |
| 6 | State Management | 6133 |
| 7 | Networking & APIs | 6193 |
| 8 | Security Best Practices | 6066 |
| 9 | Real-Time Systems | 6477 |
| 10 | Data Visualization | 6080 |
| 11 | Performance, Testing & Deployment | 6000+ |

**Total: 67,000+ lines of interview-ready content!**

---

# BONUS: Progressive Web Apps (PWA)

## Service Worker Registration

```typescript
// lib/registerSW.ts

export async function registerServiceWorker() {
  if ('serviceWorker' in navigator) {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/',
      });
      
      // Check for updates periodically
      setInterval(() => {
        registration.update();
      }, 60 * 60 * 1000); // Every hour
      
      // Handle updates
      registration.addEventListener('updatefound', () => {
        const newWorker = registration.installing;
        
        newWorker?.addEventListener('statechange', () => {
          if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
            // New content available
            showUpdateNotification();
          }
        });
      });
      
      return registration;
    } catch (error) {
      console.error('Service worker registration failed:', error);
    }
  }
}

// Skip waiting and reload
export function applyUpdate() {
  navigator.serviceWorker.controller?.postMessage({ type: 'SKIP_WAITING' });
  window.location.reload();
}
```

## PWA Manifest

```json
// public/manifest.json
{
  "name": "My App",
  "short_name": "MyApp",
  "description": "A progressive web application",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#0070f3",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/mobile.png",
      "sizes": "375x812",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ]
}
```

## Offline-First Data Sync

```typescript
// lib/offlineSync.ts

interface PendingAction {
  id: string;
  type: string;
  payload: any;
  timestamp: number;
}

class OfflineSyncManager {
  private db: IDBDatabase | null = null;
  private readonly storeName = 'pendingActions';
  
  async init() {
    return new Promise<IDBDatabase>((resolve, reject) => {
      const request = indexedDB.open('offlineSync', 1);
      
      request.onerror = () => reject(request.error);
      
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };
      
      request.onupgradeneeded = () => {
        const db = request.result;
        db.createObjectStore(this.storeName, { keyPath: 'id' });
      };
    });
  }
  
  async queueAction(action: Omit<PendingAction, 'id' | 'timestamp'>) {
    if (!this.db) await this.init();
    
    const pendingAction: PendingAction = {
      ...action,
      id: crypto.randomUUID(),
      timestamp: Date.now(),
    };
    
    return new Promise<void>((resolve, reject) => {
      const tx = this.db!.transaction(this.storeName, 'readwrite');
      tx.objectStore(this.storeName).add(pendingAction);
      tx.oncomplete = () => resolve();
      tx.onerror = () => reject(tx.error);
    });
  }
  
  async syncPendingActions() {
    if (!navigator.onLine) return;
    
    const actions = await this.getPendingActions();
    
    for (const action of actions) {
      try {
        await this.executeAction(action);
        await this.removeAction(action.id);
      } catch (error) {
        console.error('Sync failed for action:', action.id, error);
      }
    }
  }
  
  private async executeAction(action: PendingAction) {
    const response = await fetch('/api/sync', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(action),
    });
    
    if (!response.ok) throw new Error('Sync failed');
  }
  
  private async getPendingActions(): Promise<PendingAction[]> {
    return new Promise((resolve, reject) => {
      const tx = this.db!.transaction(this.storeName, 'readonly');
      const request = tx.objectStore(this.storeName).getAll();
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  private async removeAction(id: string) {
    return new Promise<void>((resolve, reject) => {
      const tx = this.db!.transaction(this.storeName, 'readwrite');
      tx.objectStore(this.storeName).delete(id);
      tx.oncomplete = () => resolve();
      tx.onerror = () => reject(tx.error);
    });
  }
}

// React hook
function useOfflineSync() {
  const [isPending, setIsPending] = useState(false);
  const [pendingCount, setPendingCount] = useState(0);
  const managerRef = useRef<OfflineSyncManager>();
  
  useEffect(() => {
    managerRef.current = new OfflineSyncManager();
    managerRef.current.init();
    
    const handleOnline = () => {
      managerRef.current?.syncPendingActions();
    };
    
    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, []);
  
  const queueAction = useCallback(async (type: string, payload: any) => {
    setIsPending(true);
    await managerRef.current?.queueAction({ type, payload });
    setPendingCount(c => c + 1);
    
    if (navigator.onLine) {
      await managerRef.current?.syncPendingActions();
    }
    setIsPending(false);
  }, []);
  
  return { queueAction, isPending, pendingCount };
}
```

---

# BONUS: Monitoring Dashboard

## Custom Performance Dashboard

```typescript
// components/PerformanceDashboard.tsx

interface PerformanceMetrics {
  lcp: number;
  fid: number;
  cls: number;
  ttfb: number;
  fcp: number;
  inp: number;
}

function PerformanceDashboard() {
  const [metrics, setMetrics] = useState<Partial<PerformanceMetrics>>({});
  const [history, setHistory] = useState<PerformanceMetrics[]>([]);
  
  useEffect(() => {
    // Collect Core Web Vitals
    const observers: PerformanceObserver[] = [];
    
    // LCP
    const lcpObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];
      setMetrics(m => ({ ...m, lcp: lastEntry.startTime }));
    });
    lcpObserver.observe({ type: 'largest-contentful-paint', buffered: true });
    observers.push(lcpObserver);
    
    // FID
    const fidObserver = new PerformanceObserver((list) => {
      const entry = list.getEntries()[0] as PerformanceEventTiming;
      setMetrics(m => ({ ...m, fid: entry.processingStart - entry.startTime }));
    });
    fidObserver.observe({ type: 'first-input', buffered: true });
    observers.push(fidObserver);
    
    // CLS
    let clsValue = 0;
    const clsObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries() as any[]) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      }
      setMetrics(m => ({ ...m, cls: clsValue }));
    });
    clsObserver.observe({ type: 'layout-shift', buffered: true });
    observers.push(clsObserver);
    
    return () => observers.forEach(o => o.disconnect());
  }, []);
  
  const getScoreColor = (metric: string, value: number) => {
    const thresholds: Record<string, [number, number]> = {
      lcp: [2500, 4000],
      fid: [100, 300],
      inp: [200, 500],
      cls: [0.1, 0.25],
      ttfb: [800, 1800],
      fcp: [1800, 3000],
    };
    
    const [good, poor] = thresholds[metric] || [0, 0];
    if (value <= good) return 'green';
    if (value <= poor) return 'orange';
    return 'red';
  };
  
  return (
    <div className="performance-dashboard">
      <h2>Core Web Vitals</h2>
      
      <div className="metrics-grid">
        <MetricCard
          title="LCP"
          value={metrics.lcp}
          unit="ms"
          target="< 2.5s"
          color={getScoreColor('lcp', metrics.lcp || 0)}
        />
        
        <MetricCard
          title="INP"
          value={metrics.inp}
          unit="ms"
          target="< 200ms"
          color={getScoreColor('inp', metrics.inp || 0)}
        />
        
        <MetricCard
          title="CLS"
          value={metrics.cls}
          unit=""
          target="< 0.1"
          color={getScoreColor('cls', metrics.cls || 0)}
        />
      </div>
      
      <h2>Other Metrics</h2>
      
      <div className="metrics-grid">
        <MetricCard
          title="TTFB"
          value={metrics.ttfb}
          unit="ms"
          target="< 800ms"
          color={getScoreColor('ttfb', metrics.ttfb || 0)}
        />
        
        <MetricCard
          title="FCP"
          value={metrics.fcp}
          unit="ms"
          target="< 1.8s"
          color={getScoreColor('fcp', metrics.fcp || 0)}
        />
      </div>
    </div>
  );
}

function MetricCard({ 
  title, 
  value, 
  unit, 
  target, 
  color 
}: { 
  title: string; 
  value?: number; 
  unit: string;
  target: string;
  color: string;
}) {
  return (
    <div className="metric-card" style={{ borderColor: color }}>
      <h3>{title}</h3>
      <div className="value" style={{ color }}>
        {value !== undefined ? `${value.toFixed(2)}${unit}` : 'Loading...'}
      </div>
      <div className="target">Target: {target}</div>
    </div>
  );
}
```

---

# BONUS: Real-World Scenarios

## Scenario: E-commerce Performance Optimization

```typescript
// Real-world scenario: Optimizing a slow product listing page

const ECOMMERCE_OPTIMIZATION = {
  problem: 'Product listing page has LCP of 5s and TTI of 8s',
  
  investigation: [
    '1. Run Lighthouse - identify top issues',
    '2. Check Network waterfall - find blocking resources',
    '3. React Profiler - identify slow components',
    '4. Bundle analyzer - find large dependencies',
  ],
  
  findings: {
    large_bundle: '2.5MB initial JS (moment.js, lodash full import)',
    unoptimized_images: '500KB hero image, no lazy loading',
    render_blocking: 'Synchronous API call in component body',
    no_caching: 'No service worker, no HTTP caching',
  },
  
  solutions: {
    bundle: `
      // Replace moment with date-fns
      import { format } from 'date-fns';
      
      // Replace lodash full import
      import debounce from 'lodash/debounce';
      
      // Code split heavy components
      const ProductZoom = lazy(() => import('./ProductZoom'));
    `,
    
    images: `
      <Image
        src="/hero.jpg"
        width={1200}
        height={600}
        priority  // LCP image
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
      
      // Lazy load product images
      {products.map(product => (
        <Image
          src={product.image}
          loading="lazy"
          sizes="(max-width: 768px) 100vw, 33vw"
        />
      ))}
    `,
    
    data_fetching: `
      // Move to React Query with suspense
      function ProductList() {
        const { data } = useSuspenseQuery({
          queryKey: ['products'],
          queryFn: fetchProducts,
          staleTime: 5 * 60 * 1000,
        });
        
        return <ProductGrid products={data} />;
      }
      
      // Wrap in Suspense
      <Suspense fallback={<ProductSkeleton count={12} />}>
        <ProductList />
      </Suspense>
    `,
    
    caching: `
      // Service Worker for static assets
      // HTTP headers for API responses
      'Cache-Control': 'public, max-age=60, stale-while-revalidate=3600'
    `,
  },
  
  results: {
    before: { LCP: 5000, TTI: 8000, bundleSize: '2.5MB' },
    after: { LCP: 1800, TTI: 3200, bundleSize: '450KB' },
    improvement: '64% faster LCP, 60% faster TTI, 82% smaller bundle',
  },
};
```

## Scenario: Debugging Flaky E2E Tests

```typescript
// Real-world scenario: Production-like E2E test failures

const FLAKY_TEST_DEBUGGING = {
  problem: 'E2E tests pass locally but fail 30% of time in CI',
  
  investigation: [
    '1. Check CI logs for patterns',
    '2. Run tests in headful mode locally',
    '3. Add debugging output',
    '4. Check for timing-dependent assertions',
  ],
  
  findings: {
    timing: 'Tests not waiting for async operations',
    state: 'Tests depend on order, shared state',
    environment: 'CI has slower CPU, different timing',
    selectors: 'Using fragile CSS selectors',
  },
  
  solutions: {
    timing: `
      // Before: Flaky
      await page.click('[data-testid="submit"]');
      expect(await page.textContent('.success')).toBe('Done');
      
      // After: Stable
      await page.click('[data-testid="submit"]');
      await expect(page.locator('.success')).toHaveText('Done', {
        timeout: 10000, // Generous timeout
      });
    `,
    
    state_isolation: `
      // Before: Shared database
      test.describe('User flows', () => {
        test('create user', async () => { /* creates user */ });
        test('edit user', async () => { /* depends on previous */ });
      });
      
      // After: Isolated
      test.describe('User flows', () => {
        test.beforeEach(async () => {
          await resetDatabase();
          await seedTestUser();
        });
        
        test('create user', async () => { /* independent */ });
        test('edit user', async () => { /* independent */ });
      });
    `,
    
    selectors: `
      // Before: Fragile
      await page.click('.btn.btn-primary:first-child');
      
      // After: Stable
      await page.getByRole('button', { name: 'Submit' }).click();
    `,
    
    retry_strategy: `
      // playwright.config.ts
      export default defineConfig({
        retries: process.env.CI ? 2 : 0,
        
        // Use test isolation
        use: {
          trace: 'retain-on-failure',
          video: 'retain-on-failure',
        },
      });
    `,
  },
  
  results: {
    before: { passRate: '70%', avgDuration: '5 min' },
    after: { passRate: '99.5%', avgDuration: '3 min' },
  },
};
```

---

# BONUS: Final Interview Tips

## Day Before Interview

```typescript
const DAY_BEFORE_CHECKLIST = {
  technical: [
    'Review Core Web Vitals thresholds',
    'Practice explaining React rendering lifecycle',
    'Review testing pyramid and when to use each',
    'Rehearse deployment strategies explanation',
  ],
  
  practical: [
    'Prepare development environment',
    'Test screen sharing setup',
    'Have backup internet option ready',
    'Prepare water and snacks',
  ],
  
  mental: [
    'Get good sleep (8+ hours)',
    'Review your best projects',
    'Prepare questions for interviewer',
    'Remember: its a conversation, not an exam',
  ],
};
```

## During Technical Discussion

```typescript
const INTERVIEW_COMMUNICATION = {
  when_asked_question: [
    '1. Pause and think (5-10 seconds is fine)',
    '2. Ask clarifying questions if needed',
    '3. Think out loud - explain your approach',
    '4. Start with high-level, then dive into details',
    '5. Give concrete examples from experience',
  ],
  
  when_stuck: [
    'Admit what you know vs dont know',
    'Explain how you would find the answer',
    'Pivot to related concepts you do know',
    'Ask for hints - interviewers often guide',
  ],
  
  good_phrases: [
    '"In my experience with [project]..."',
    '"One tradeoff to consider is..."',
    '"I would start by measuring..."',
    '"That depends on the requirements..."',
    '"I am not 100% sure, but my understanding is..."',
  ],
};
```

## Questions to Ask Interviewer

```typescript
const QUESTIONS_FOR_INTERVIEWER = {
  technical: [
    'What does your CI/CD pipeline look like?',
    'How do you handle performance monitoring in production?',
    'What testing strategies does the team use?',
    'How do you approach technical debt?',
  ],
  
  team: [
    'How is code review conducted?',
    'What does a typical sprint look like?',
    'How does the team stay up-to-date with new technologies?',
  ],
  
  growth: [
    'What does growth look like for this role?',
    'How does the team share knowledge?',
    'What challenging problems is the team solving?',
  ],
};
```

---

# 📚 Complete Resource List

## Official Documentation

```typescript
const OFFICIAL_DOCS = {
  performance: [
    'https://web.dev/vitals/',
    'https://developer.chrome.com/docs/devtools/performance/',
    'https://react.dev/reference/react/Profiler',
  ],
  
  testing: [
    'https://testing-library.com/docs/',
    'https://playwright.dev/docs/intro',
    'https://jestjs.io/docs/getting-started',
    'https://vitest.dev/guide/',
  ],
  
  deployment: [
    'https://docs.github.com/actions',
    'https://vercel.com/docs',
    'https://docs.docker.com/',
    'https://kubernetes.io/docs/',
  ],
};
```

## Recommended Reading

```typescript
const RECOMMENDED_BOOKS = [
  'High Performance Browser Networking - Ilya Grigorik',
  'Testing JavaScript Applications - Lucas da Costa',
  'The Art of Unit Testing - Roy Osherove',
  'Site Reliability Engineering - Google',
  'Designing Data-Intensive Applications - Martin Kleppmann',
];

const RECOMMENDED_COURSES = [
  'Testing JavaScript - Kent C. Dodds',
  'Web Performance Fundamentals - Todd Gardner',
  'Complete Intro to Containers - Brian Holt',
];
```

---

**🎉 COMPLETE: Part 11 - Performance, Testing & Deployment**

This is the final part of the Comprehensive Frontend Interview Guide series!

**Final Statistics:**
- Total Parts: 11
- Total Lines: 67,000+
- Topics Covered: JavaScript, TypeScript, React, Hooks, Next.js, State Management, APIs, Security, Real-Time Systems, Visualization, Performance, Testing, Deployment
- Interview Ready: ✅

**You are now prepared to ace your frontend interviews!**

**Good luck with your interviews! 🚀**


