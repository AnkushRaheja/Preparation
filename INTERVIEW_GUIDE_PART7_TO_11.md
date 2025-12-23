# Part 7: Networking & Security

> **Deep dive into secure API communication**

---

## 7.1 Axios vs Fetch

### Why Axios?
```typescript
// Fetch - Manual JSON parsing
const response = await fetch('/api/users');
if (!response.ok) throw new Error('Failed');
const data = await response.json();

// Axios - Automatic JSON parsing
const { data } = await axios.get('/api/users');
```

**Axios Advantages:**
- ✅ Automatic JSON transformation
- ✅ Request/Response interceptors
- ✅ Request cancellation
- ✅ Better error handling
- ✅ Progress tracking for uploads

---

## 7.2 Axios Instance & Interceptors

### Creating an Instance
```typescript
// lib/axios.ts
import axios from 'axios';

export const axiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});
```

### Request Interceptor
```typescript
axiosInstance.interceptors.request.use(
  (config) => {
    // Add auth token to every request
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
```

### Response Interceptor
```typescript
axiosInstance.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired - logout user
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

---

## 7.3 OAuth 2.0 (GitHub Authentication)

### The Flow
```
1. User clicks "Login with GitHub"
   ↓
2. Redirect to: github.com/login/oauth/authorize?client_id=XXX
   ↓
3. User approves, GitHub redirects back with ?code=ABC123
   ↓
4. Backend exchanges code for access_token (secret)
   ↓
5. Backend creates JWT session token
   ↓
6. Frontend stores JWT, user is logged in
```

### CodeHealth Implementation
```typescript
// OAuth callback handling
function OAuthCallback() {
  useEffect(() => {
    const code = searchParams.get('code');
    
    if (code) {
      // Send code to backend
      axios.post('/auth/github/callback', { code })
        .then(response => {
          localStorage.setItem('authToken', response.data.token);
          router.push('/dashboard');
        });
    }
  }, []);
}
```

---

## 7.4 JWT (JSON Web Tokens)

### Structure
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.  // Header (Base64)
eyJ1c2VySWQiOiIxMjMiLCJleHAiOjE2NTB9.  // Payload (Base64)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV    // Signature
```

### Storage Options

| Location | XSS Safe | CSRF Safe | Best For |
|----------|----------|-----------|----------|
| localStorage | ❌ | ✅ | SPAs (with caution) |
| HttpOnly Cookie | ✅ | ❌ | Server-rendered apps |
| Memory | ✅ | ✅ | Short sessions |

---

## 7.5 Security Vulnerabilities

### XSS (Cross-Site Scripting)
**Attack:** Injecting malicious scripts into your app.

```jsx
// ❌ DANGEROUS - Never use unless content is sanitized
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE - React auto-escapes content
<div>{userInput}</div>
```

### CSRF (Cross-Site Request Forgery)
**Attack:** Tricking users into making unwanted requests.

**Defenses:**
1. Use `SameSite` cookie attribute
2. CSRF tokens
3. Verify `Origin` header

### CORS (Cross-Origin Resource Sharing)
```typescript
// Backend must allow frontend origin
// Access-Control-Allow-Origin: https://codehealth.ai
```

---

# Part 8: Real-Time Systems (Socket.io)

> **WebSocket implementation for live updates**

---

## 8.1 HTTP vs WebSocket

| HTTP | WebSocket |
|------|-----------|
| Request-Response | Bidirectional |
| Connection closes after response | Persistent connection |
| Client initiates | Either side can send |
| Good for REST APIs | Good for real-time data |

---

## 8.2 Socket.io Architecture

```typescript
// lib/socket.ts - Singleton Pattern
class SocketService {
  private socket: Socket | null = null;

  connect(token: string): Socket {
    if (this.socket) return this.socket;

    this.socket = io(SOCKET_URL, {
      auth: { token },
      transports: ['websocket'],
      reconnection: true,
      reconnectionAttempts: 5,
      reconnectionDelay: 1000,
    });

    return this.socket;
  }

  disconnect(): void {
    this.socket?.disconnect();
    this.socket = null;
  }
}

export const socketService = new SocketService();
```

---

## 8.3 React Integration

```typescript
// hooks/useSocket.ts
function useSocket() {
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const token = localStorage.getItem('authToken');
    if (!token) return;

    const socket = socketService.connect(token);

    socket.on('connect', () => setIsConnected(true));
    socket.on('disconnect', () => setIsConnected(false));

    // Listen for analysis updates
    socket.on('analysis:progress', (data) => {
      console.log('Progress:', data.percentage);
    });

    return () => {
      socket.off('connect');
      socket.off('disconnect');
      socket.off('analysis:progress');
    };
  }, []);

  return { isConnected };
}
```

---

# Part 9: Data Visualization (Recharts)

> **Building interactive charts**

---

## 9.1 Recharts Components

```tsx
import {
  ResponsiveContainer,
  LineChart, Line,
  AreaChart, Area,
  BarChart, Bar,
  PieChart, Pie,
  XAxis, YAxis,
  CartesianGrid,
  Tooltip, Legend
} from 'recharts';

// Basic Line Chart
function HealthTrendChart({ data }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Line 
          type="monotone" 
          dataKey="healthScore" 
          stroke="#8884d8" 
          strokeWidth={2}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

---

# Part 10: Performance Optimization

> **Core Web Vitals and React optimization**

---

## 10.1 Core Web Vitals

| Metric | Good | Description |
|--------|------|-------------|
| LCP | < 2.5s | Largest Contentful Paint |
| INP | < 200ms | Interaction to Next Paint |
| CLS | < 0.1 | Cumulative Layout Shift |

---

## 10.2 React Optimization Patterns

### React.memo
```typescript
// Only re-renders if props change
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* Complex rendering */}</div>;
});
```

### Code Splitting
```typescript
// Dynamic import - loads only when needed
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <Spinner />,
  ssr: false
});
```

---

# Part 11: CI/CD & Deployment

> **GitHub Actions and deployment pipelines**

---

## 11.1 GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Build
        run: npm run build
```

---

## 11.2 Key CI/CD Concepts

| Term | Definition |
|------|------------|
| CI | Continuous Integration - automated testing |
| CD | Continuous Deployment - automated releases |
| Pipeline | Series of automated steps |
| Workflow | GitHub Actions configuration |
| Job | Set of steps running on same runner |
| Step | Individual command or action |

---

## 🎯 Complete Interview Question Bank

### JavaScript
1. Explain the Event Loop
2. What are closures?
3. Difference between var, let, const?
4. What is hoisting?
5. How does 'this' work?

### TypeScript
6. Interface vs Type?
7. What are generics?
8. Explain utility types
9. What is a discriminated union?

### React
10. What is Virtual DOM?
11. Explain useState vs useRef
12. When to use useMemo vs useCallback?
13. What are the Rules of Hooks?

### Next.js
14. Server vs Client Components?
15. What is the App Router?
16. How does SSR work?

### Security
17. What is XSS and how to prevent it?
18. JWT storage best practices?
19. Explain OAuth 2.0 flow

### Performance
20. What are Core Web Vitals?
21. How to optimize React re-renders?
22. Code splitting strategies?

---

**📁 All files are in: `frontend/docs/`**

Good luck with your interviews! 🚀
