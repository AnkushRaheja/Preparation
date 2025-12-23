# 📚 Part 8: Security Best Practices Complete Guide

> **Master frontend security for interviews: XSS, CSRF, authentication, and secure coding**

---

## 📋 Table of Contents

1. [Security Fundamentals](#1-security-fundamentals)
2. [XSS (Cross-Site Scripting)](#2-xss-cross-site-scripting)
3. [CSRF (Cross-Site Request Forgery)](#3-csrf-cross-site-request-forgery)
4. [Authentication Security](#4-authentication-security)
5. [Authorization](#5-authorization)
6. [Secure Storage](#6-secure-storage)
7. [Content Security Policy](#7-content-security-policy)
8. [HTTPS and Transport Security](#8-https-and-transport-security)
9. [Input Validation](#9-input-validation)
10. [Secure Coding Practices](#10-secure-coding-practices)
11. [Interview Questions](#11-interview-questions)
12. [Coding Challenges](#12-coding-challenges)

---

# 1. Security Fundamentals

## 1.1 The CIA Triad

```typescript
// Confidentiality, Integrity, Availability

const SECURITY_PRINCIPLES = {
  confidentiality: 'Only authorized users can access data',
  integrity: 'Data cannot be modified by unauthorized users',
  availability: 'Data is accessible when needed',
};

// Frontend concerns
const FRONTEND_SECURITY_CONCERNS = {
  confidentiality: [
    'Secure token storage',
    'No sensitive data in logs',
    'HTTPS enforcement',
  ],
  integrity: [
    'Input validation',
    'XSS prevention',
    'CSRF protection',
  ],
  availability: [
    'Rate limiting handling',
    'Error recovery',
    'Graceful degradation',
  ],
};
```

## 1.2 Common Attack Vectors

```typescript
const ATTACK_VECTORS = {
  // Injection attacks
  XSS: 'Inject malicious scripts into web pages',
  SQLInjection: 'Inject SQL through user input',
  CommandInjection: 'Execute system commands',
  
  // Authentication attacks
  BruteForce: 'Guess credentials repeatedly',
  SessionHijacking: 'Steal session tokens',
  CredentialStuffing: 'Use leaked credentials',
  
  // Request attacks
  CSRF: 'Trick users into unwanted actions',
  Clickjacking: 'Hide malicious UI under legitimate UI',
  SSRF: 'Make server request internal resources',
  
  // Data attacks
  ManInTheMiddle: 'Intercept communications',
  DataExfiltration: 'Steal sensitive data',
  InsecureStorage: 'Access improperly stored data',
};
```

## 1.3 OWASP Top 10 (Relevant to Frontend)

```typescript
const OWASP_TOP_10_FRONTEND = {
  A01_BrokenAccessControl: {
    description: 'Users can act outside permissions',
    frontendConcerns: [
      'Client-side authorization checks (supplementary only)',
      'Hiding UI elements for unauthorized actions',
      'Proper route guards',
    ],
  },
  A02_CryptographicFailures: {
    description: 'Improper cryptography usage',
    frontendConcerns: [
      'Never store sensitive data in localStorage',
      'Use HTTPS everywhere',
      'Avoid custom encryption',
    ],
  },
  A03_Injection: {
    description: 'Untrusted data sent to interpreter',
    frontendConcerns: [
      'XSS prevention',
      'Sanitize user input',
      'Use parameterized queries',
    ],
  },
  A07_XSS: {
    description: 'Execute scripts in victim browser',
    frontendConcerns: [
      'Escape output properly',
      'Use Content Security Policy',
      'Sanitize HTML input',
    ],
  },
};
```

---

# 2. XSS (Cross-Site Scripting)

## 2.1 Types of XSS

```typescript
// 1. Reflected XSS - Malicious script from URL
// Attacker: https://site.com/search?q=<script>steal()</script>
// Server reflects the query back unsanitized

// 2. Stored XSS - Malicious script stored in database
// Attacker posts: <script>steal()</script> as a comment
// All users who view the comment execute the script

// 3. DOM-based XSS - Malicious script via DOM manipulation
// Example: document.innerHTML = location.hash.slice(1)
// Attacker: https://site.com#<script>steal()</script>
```

## 2.2 XSS Prevention in React

```tsx
// React automatically escapes content - SAFE
function SafeComponent({ userInput }: { userInput: string }) {
  return <div>{userInput}</div>; // Automatically escaped
}

// dangerouslySetInnerHTML is UNSAFE without sanitization
function UnsafeComponent({ html }: { html: string }) {
  // ❌ DANGEROUS - never use with untrusted input
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// Safe usage with DOMPurify
import DOMPurify from 'dompurify';

function SafeHTMLComponent({ html }: { html: string }) {
  const sanitizedHTML = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
    ALLOWED_ATTR: ['href', 'target'],
  });
  
  return <div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />;
}
```

## 2.3 DOMPurify Configuration

```typescript
import DOMPurify from 'dompurify';

// Basic sanitization
const clean = DOMPurify.sanitize(dirty);

// Allow only specific tags
const cleanStrict = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br'],
});

// Allow specific attributes
const cleanWithLinks = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['a', 'p', 'span'],
  ALLOWED_ATTR: ['href', 'class'],
});

// Remove all HTML
const textOnly = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: [],
});

// Custom hooks for advanced use
DOMPurify.addHook('afterSanitizeAttributes', (node) => {
  if (node.tagName === 'A') {
    node.setAttribute('rel', 'noopener noreferrer');
    node.setAttribute('target', '_blank');
  }
});

// React hook for sanitization
function useSanitizedHTML(html: string, config?: DOMPurify.Config) {
  return useMemo(
    () => DOMPurify.sanitize(html, config),
    [html, config]
  );
}
```

## 2.4 URL Sanitization

```typescript
// Dangerous: javascript: protocol in URLs
const userUrl = 'javascript:alert("XSS")';

// ❌ UNSAFE
<a href={userUrl}>Click</a>

// ✅ Safe URL validation
function sanitizeUrl(url: string): string {
  try {
    const parsed = new URL(url);
    
    // Only allow http and https
    if (!['http:', 'https:'].includes(parsed.protocol)) {
      return '#';
    }
    
    return url;
  } catch {
    // Invalid URL
    return '#';
  }
}

// Usage
<a href={sanitizeUrl(userUrl)}>Click</a>

// More comprehensive URL sanitizer
function isValidUrl(url: string): boolean {
  const ALLOWED_PROTOCOLS = ['http:', 'https:', 'mailto:', 'tel:'];
  
  try {
    const parsed = new URL(url);
    return ALLOWED_PROTOCOLS.includes(parsed.protocol);
  } catch {
    // Could be a relative URL
    return url.startsWith('/') && !url.startsWith('//');
  }
}
```

---

# 3. CSRF (Cross-Site Request Forgery)

## 3.1 How CSRF Works

```typescript
// 1. User logs into bank.com (gets session cookie)
// 2. User visits evil.com
// 3. evil.com has: <img src="bank.com/transfer?to=attacker&amount=1000">
// 4. Browser sends request with bank.com cookies
// 5. Transfer happens without user consent

// CSRF exploits that browsers automatically send cookies
```

## 3.2 CSRF Prevention

```typescript
// 1. CSRF Tokens
// Server generates token, frontend includes in requests

async function fetchWithCSRF(url: string, options: RequestInit = {}) {
  const csrfToken = document
    .querySelector('meta[name="csrf-token"]')
    ?.getAttribute('content');
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken || '',
    },
  });
}

// 2. SameSite Cookies
// Server sets: Set-Cookie: session=abc; SameSite=Strict

// SameSite options:
// Strict: Cookie never sent cross-site
// Lax: Cookie sent for top-level navigation GET
// None: Cookie always sent (requires Secure)

// 3. Double Submit Cookie
async function doubleSubmitCSRF(url: string, options: RequestInit = {}) {
  const csrfCookie = document.cookie
    .split('; ')
    .find(row => row.startsWith('csrf='))
    ?.split('=')[1];
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfCookie || '',
    },
  });
}
```

## 3.3 CSRF in Next.js

```typescript
// pages/api/transfer.ts
import { NextApiRequest, NextApiResponse } from 'next';
import csrf from 'csrf';

const tokens = new csrf();

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    // Generate token for form
    const secret = req.cookies.csrfSecret || tokens.secretSync();
    const token = tokens.create(secret);
    
    res.setHeader('Set-Cookie', `csrfSecret=${secret}; HttpOnly; SameSite=Strict`);
    return res.json({ csrfToken: token });
  }
  
  if (req.method === 'POST') {
    const secret = req.cookies.csrfSecret;
    const token = req.headers['x-csrf-token'] as string;
    
    if (!tokens.verify(secret, token)) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
    
    // Process request
    return res.json({ success: true });
  }
}

// Client-side usage
function TransferForm() {
  const [csrfToken, setCsrfToken] = useState('');
  
  useEffect(() => {
    fetch('/api/transfer')
      .then(res => res.json())
      .then(data => setCsrfToken(data.csrfToken));
  }, []);
  
  const handleTransfer = async () => {
    await fetch('/api/transfer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken,
      },
      body: JSON.stringify({ amount: 100 }),
    });
  };
  
  return <button onClick={handleTransfer}>Transfer</button>;
}
```

---

# 4. Authentication Security

## 4.1 Secure Token Storage

```typescript
// ❌ localStorage - Vulnerable to XSS
localStorage.setItem('token', jwt); // Any script can read this

// ❌ sessionStorage - Vulnerable to XSS
sessionStorage.setItem('token', jwt);

// ✅ HttpOnly Cookies - Not accessible via JavaScript
// Set by server: Set-Cookie: token=jwt; HttpOnly; Secure; SameSite=Strict

// ✅ Memory storage (short-lived)
class TokenStore {
  private accessToken: string | null = null;
  
  setToken(token: string) {
    this.accessToken = token;
  }
  
  getToken(): string | null {
    return this.accessToken;
  }
  
  clearToken() {
    this.accessToken = null;
  }
}

export const tokenStore = new TokenStore();
```

## 4.2 JWT Security

```typescript
// JWT structure: header.payload.signature

interface JWTPayload {
  sub: string;      // Subject (user ID)
  iat: number;      // Issued at
  exp: number;      // Expiration
  iss: string;      // Issuer
  aud: string;      // Audience
  roles?: string[]; // Custom claims
}

// Decode JWT (note: this does NOT verify!)
function decodeJWT(token: string): JWTPayload | null {
  try {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const payload = JSON.parse(atob(base64));
    return payload;
  } catch {
    return null;
  }
}

// Check if token is expired
function isTokenExpired(token: string): boolean {
  const payload = decodeJWT(token);
  if (!payload) return true;
  return Date.now() >= payload.exp * 1000;
}

// ⚠️ IMPORTANT: Always verify JWT on server
// Frontend can only decode, server must verify signature
```

## 4.3 Refresh Token Flow

```typescript
class AuthService {
  private accessToken: string | null = null;
  private refreshPromise: Promise<string> | null = null;
  
  async getAccessToken(): Promise<string | null> {
    if (this.accessToken && !isTokenExpired(this.accessToken)) {
      return this.accessToken;
    }
    
    // Refresh if expired
    return this.refreshAccessToken();
  }
  
  async refreshAccessToken(): Promise<string> {
    // Prevent multiple simultaneous refresh calls
    if (this.refreshPromise) {
      return this.refreshPromise;
    }
    
    this.refreshPromise = this.doRefresh();
    
    try {
      const token = await this.refreshPromise;
      return token;
    } finally {
      this.refreshPromise = null;
    }
  }
  
  private async doRefresh(): Promise<string> {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include', // Send refresh token cookie
    });
    
    if (!response.ok) {
      this.logout();
      throw new Error('Refresh failed');
    }
    
    const { accessToken } = await response.json();
    this.accessToken = accessToken;
    return accessToken;
  }
  
  logout() {
    this.accessToken = null;
    // Clear cookies on server
    fetch('/api/auth/logout', { method: 'POST', credentials: 'include' });
  }
}

export const authService = new AuthService();
```

## 4.4 Password Security

```typescript
// Frontend password requirements
const PASSWORD_REQUIREMENTS = {
  minLength: 12,
  maxLength: 128,
  requireUppercase: true,
  requireLowercase: true,
  requireNumbers: true,
  requireSpecial: true,
};

function validatePassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];
  
  if (password.length < PASSWORD_REQUIREMENTS.minLength) {
    errors.push(`Password must be at least ${PASSWORD_REQUIREMENTS.minLength} characters`);
  }
  
  if (password.length > PASSWORD_REQUIREMENTS.maxLength) {
    errors.push(`Password must be less than ${PASSWORD_REQUIREMENTS.maxLength} characters`);
  }
  
  if (PASSWORD_REQUIREMENTS.requireUppercase && !/[A-Z]/.test(password)) {
    errors.push('Password must contain an uppercase letter');
  }
  
  if (PASSWORD_REQUIREMENTS.requireLowercase && !/[a-z]/.test(password)) {
    errors.push('Password must contain a lowercase letter');
  }
  
  if (PASSWORD_REQUIREMENTS.requireNumbers && !/\d/.test(password)) {
    errors.push('Password must contain a number');
  }
  
  if (PASSWORD_REQUIREMENTS.requireSpecial && !/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
    errors.push('Password must contain a special character');
  }
  
  return { valid: errors.length === 0, errors };
}

// Password strength meter
function getPasswordStrength(password: string): 'weak' | 'medium' | 'strong' {
  let score = 0;
  
  if (password.length >= 12) score++;
  if (password.length >= 16) score++;
  if (/[A-Z]/.test(password)) score++;
  if (/[a-z]/.test(password)) score++;
  if (/\d/.test(password)) score++;
  if (/[!@#$%^&*(),.?":{}|<>]/.test(password)) score++;
  
  if (score < 3) return 'weak';
  if (score < 5) return 'medium';
  return 'strong';
}
```

---

# 5. Authorization

## 5.1 Role-Based Access Control (RBAC)

```typescript
type Role = 'admin' | 'editor' | 'viewer';
type Permission = 'create' | 'read' | 'update' | 'delete';

const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  admin: ['create', 'read', 'update', 'delete'],
  editor: ['create', 'read', 'update'],
  viewer: ['read'],
};

function hasPermission(role: Role, permission: Permission): boolean {
  return ROLE_PERMISSIONS[role]?.includes(permission) ?? false;
}

// React context for auth
interface AuthContextType {
  user: User | null;
  hasPermission: (permission: Permission) => boolean;
  hasRole: (role: Role) => boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  const hasPermission = useCallback((permission: Permission) => {
    if (!user) return false;
    return ROLE_PERMISSIONS[user.role]?.includes(permission) ?? false;
  }, [user]);
  
  const hasRole = useCallback((role: Role) => {
    return user?.role === role;
  }, [user]);
  
  return (
    <AuthContext.Provider value={{ user, hasPermission, hasRole }}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage
function AdminPanel() {
  const { hasPermission } = useAuth();
  
  if (!hasPermission('delete')) {
    return <div>Access Denied</div>;
  }
  
  return <div>Admin Panel</div>;
}
```

## 5.2 Protected Routes

```tsx
// Next.js App Router
import { redirect } from 'next/navigation';
import { getSession } from '@/lib/auth';

export default async function ProtectedPage() {
  const session = await getSession();
  
  if (!session) {
    redirect('/login');
  }
  
  if (!session.user.roles.includes('admin')) {
    redirect('/unauthorized');
  }
  
  return <AdminDashboard />;
}

// Client-side route guard
function ProtectedRoute({
  children,
  requiredRole,
}: {
  children: React.ReactNode;
  requiredRole?: Role;
}) {
  const { user, loading } = useAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!loading && !user) {
      router.push('/login');
    }
    
    if (!loading && user && requiredRole && user.role !== requiredRole) {
      router.push('/unauthorized');
    }
  }, [user, loading, requiredRole, router]);
  
  if (loading) return <LoadingSpinner />;
  if (!user) return null;
  if (requiredRole && user.role !== requiredRole) return null;
  
  return <>{children}</>;
}
```

---

# 6. Secure Storage

## 6.1 Storage Options Comparison

```typescript
const STORAGE_COMPARISON = {
  localStorage: {
    pros: ['Persistent', 'Easy to use', '5MB limit'],
    cons: ['XSS vulnerable', 'Sync only', 'String only'],
    useFor: ['Non-sensitive preferences', 'Theme settings'],
    avoid: ['Tokens', 'PII', 'Sensitive data'],
  },
  sessionStorage: {
    pros: ['Tab-isolated', 'Cleared on close'],
    cons: ['XSS vulnerable', 'Lost on refresh'],
    useFor: ['Temporary UI state'],
    avoid: ['Tokens', 'Sensitive data'],
  },
  cookies: {
    pros: ['HttpOnly option', 'SameSite protection', 'Auto-sent'],
    cons: ['4KB limit', 'Sent with every request'],
    useFor: ['Session tokens', 'Auth'],
    avoid: ['Large data'],
  },
  memory: {
    pros: ['Not persistent', 'Not in DevTools'],
    cons: ['Lost on refresh', 'XSS if exposed'],
    useFor: ['Short-lived tokens'],
    avoid: ['Anything needing persistence'],
  },
};
```

## 6.2 Secure Cookie Implementation

```typescript
// Server-side: Set secure cookies
// Express example
app.post('/login', (req, res) => {
  const { accessToken, refreshToken } = generateTokens(user);
  
  // Access token - short lived, in memory is OK
  res.cookie('accessToken', accessToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 15 * 60 * 1000, // 15 minutes
  });
  
  // Refresh token - long lived, httpOnly only
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
    path: '/api/auth/refresh', // Only sent to refresh endpoint
  });
  
  res.json({ success: true });
});
```

---

# 7. Content Security Policy

## 7.1 CSP Basics

```typescript
// Content-Security-Policy header restricts resource loading

const CSP_DIRECTIVES = {
  'default-src': 'Fallback for unspecified directives',
  'script-src': 'Valid sources for JavaScript',
  'style-src': 'Valid sources for stylesheets',
  'img-src': 'Valid sources for images',
  'connect-src': 'Valid sources for fetch/XHR/WebSocket',
  'font-src': 'Valid sources for fonts',
  'frame-src': 'Valid sources for iframes',
  'media-src': 'Valid sources for audio/video',
  'object-src': 'Valid sources for plugins',
};

// Example CSP
const cspHeader = `
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  font-src 'self' https://fonts.gstatic.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
`.replace(/\s+/g, ' ').trim();
```

## 7.2 CSP in Next.js

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' blob: data:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s+/g, ' ').trim(),
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY',
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

---

# 8. HTTPS and Transport Security

## 8.1 Enforcing HTTPS

```typescript
// Redirect HTTP to HTTPS (middleware)
export function middleware(request: NextRequest) {
  if (
    process.env.NODE_ENV === 'production' &&
    request.headers.get('x-forwarded-proto') !== 'https'
  ) {
    return NextResponse.redirect(
      `https://${request.headers.get('host')}${request.nextUrl.pathname}`,
      301
    );
  }
  
  return NextResponse.next();
}

// HSTS Header
const hstsHeader = {
  key: 'Strict-Transport-Security',
  value: 'max-age=31536000; includeSubDomains; preload',
};
```

---

# 9. Input Validation

## 9.1 Client-Side Validation

```typescript
import { z } from 'zod';

// Define schemas
const UserSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string()
    .min(12, 'Password must be at least 12 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[0-9]/, 'Must contain number'),
  name: z.string()
    .min(2, 'Name too short')
    .max(100, 'Name too long')
    .regex(/^[a-zA-Z\s]+$/, 'Name can only contain letters'),
});

// Validate
function validateUser(data: unknown) {
  const result = UserSchema.safeParse(data);
  
  if (!result.success) {
    return {
      valid: false,
      errors: result.error.flatten().fieldErrors,
    };
  }
  
  return { valid: true, data: result.data };
}
```

---

# 10. Secure Coding Practices

## 10.1 Environment Variables

```typescript
// ✅ NEXT_PUBLIC_ prefix for client-side variables
// .env.local
NEXT_PUBLIC_API_URL=https://api.example.com
API_SECRET=server-only-secret

// Usage
const apiUrl = process.env.NEXT_PUBLIC_API_URL; // Available on client
const secret = process.env.API_SECRET; // Server only

// ❌ Never expose secrets to client
// This would expose the secret in the bundle!
// const secret = process.env.API_SECRET; // In client component
```

## 10.2 Dependency Security

```bash
# Check for vulnerabilities
npm audit

# Fix automatically
npm audit fix

# Check with Snyk
npx snyk test
```

---

# 11. Interview Questions

## Q1: What is XSS and how do you prevent it?

**Answer:**
XSS (Cross-Site Scripting) allows attackers to inject malicious scripts into web pages.

**Prevention:**
1. Use React's automatic escaping
2. Sanitize with DOMPurify before dangerouslySetInnerHTML
3. Validate and sanitize URLs
4. Implement Content Security Policy
5. HttpOnly cookies for tokens

```tsx
// Safe
<div>{userInput}</div>

// Sanitize if you need HTML
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

---

## Q2: What is CSRF and how do you prevent it?

**Answer:**
CSRF tricks users into making unwanted requests using their authenticated session.

**Prevention:**
1. CSRF tokens in forms and headers
2. SameSite cookie attribute
3. Check Origin/Referer headers
4. Require re-authentication for sensitive actions

---

## Q3: Where should you store JWT tokens?

**Answer:**
- **Best:** HttpOnly cookies (not accessible via JavaScript)
- **OK:** Memory (cleared on refresh)
- **Avoid:** localStorage/sessionStorage (XSS vulnerable)

---

# 12. Coding Challenges

## Challenge 1: Implement Input Sanitizer

```typescript
function sanitizeInput(input: string): string {
  return input
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}
```

## Challenge 2: Implement CSP Nonce

```typescript
function generateNonce(): string {
  const array = new Uint8Array(16);
  crypto.getRandomValues(array);
  return btoa(String.fromCharCode(...array));
}
```

---

# 13. OAuth 2.0 Security

## 13.1 OAuth Flow Types

```typescript
const OAUTH_FLOWS = {
  authorizationCode: {
    use: 'Server-side apps with backend',
    security: 'Most secure - code exchanged server-side',
    steps: [
      '1. Redirect user to authorization server',
      '2. User authenticates and grants permission',
      '3. Authorization server redirects with code',
      '4. Backend exchanges code for tokens',
      '5. Tokens stored securely on server',
    ],
  },
  authorizationCodePKCE: {
    use: 'SPAs and mobile apps',
    security: 'Secure for public clients',
    steps: [
      '1. Generate code_verifier and code_challenge',
      '2. Redirect with code_challenge',
      '3. Exchange code with code_verifier',
      '4. No client secret needed',
    ],
  },
  implicit: {
    use: 'DEPRECATED - Do not use',
    security: 'Token exposed in URL',
    reason: 'Use Authorization Code with PKCE instead',
  },
};
```

## 13.2 PKCE Implementation

```typescript
// Generate PKCE challenge
async function generatePKCE(): Promise<{
  codeVerifier: string;
  codeChallenge: string;
}> {
  // Generate random verifier (43-128 characters)
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  const codeVerifier = base64URLEncode(array);
  
  // Create challenge from verifier
  const encoder = new TextEncoder();
  const data = encoder.encode(codeVerifier);
  const digest = await crypto.subtle.digest('SHA-256', data);
  const codeChallenge = base64URLEncode(new Uint8Array(digest));
  
  return { codeVerifier, codeChallenge };
}

function base64URLEncode(buffer: Uint8Array): string {
  return btoa(String.fromCharCode(...buffer))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}

// OAuth login flow
async function initiateOAuthLogin() {
  const { codeVerifier, codeChallenge } = await generatePKCE();
  
  // Store verifier for later
  sessionStorage.setItem('pkce_verifier', codeVerifier);
  
  const params = new URLSearchParams({
    client_id: 'your-client-id',
    redirect_uri: 'http://localhost:3000/callback',
    response_type: 'code',
    scope: 'openid profile email',
    code_challenge: codeChallenge,
    code_challenge_method: 'S256',
    state: generateState(),
  });
  
  window.location.href = `https://auth.example.com/authorize?${params}`;
}

// Handle OAuth callback
async function handleOAuthCallback(code: string) {
  const codeVerifier = sessionStorage.getItem('pkce_verifier');
  sessionStorage.removeItem('pkce_verifier');
  
  const response = await fetch('https://auth.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'http://localhost:3000/callback',
      client_id: 'your-client-id',
      code_verifier: codeVerifier!,
    }),
  });
  
  return response.json();
}
```

## 13.3 State Parameter for CSRF Protection

```typescript
function generateState(): string {
  const array = new Uint8Array(16);
  crypto.getRandomValues(array);
  const state = base64URLEncode(array);
  sessionStorage.setItem('oauth_state', state);
  return state;
}

function validateState(receivedState: string): boolean {
  const savedState = sessionStorage.getItem('oauth_state');
  sessionStorage.removeItem('oauth_state');
  return savedState === receivedState;
}

// In callback handler
function OAuthCallback() {
  const searchParams = new URLSearchParams(window.location.search);
  const code = searchParams.get('code');
  const state = searchParams.get('state');
  const error = searchParams.get('error');
  
  useEffect(() => {
    if (error) {
      console.error('OAuth error:', error);
      return;
    }
    
    if (!validateState(state!)) {
      console.error('Invalid state - possible CSRF attack');
      return;
    }
    
    handleOAuthCallback(code!);
  }, []);
  
  return <div>Processing login...</div>;
}
```

---

# 14. Clickjacking Protection

## 14.1 What is Clickjacking?

```typescript
// Clickjacking: Attacker overlays transparent iframe on legitimate site
// User thinks they're clicking on the visible site
// Actually clicking on the hidden iframe

// Attack example:
// <iframe src="https://bank.com/transfer" style="opacity: 0"></iframe>
// <button style="position: absolute">Win a Prize!</button>
// User clicks "Win a Prize" but actually clicks transfer button
```

## 14.2 Prevention Methods

```typescript
// 1. X-Frame-Options header (legacy)
const headers = {
  'X-Frame-Options': 'DENY', // Never allow framing
  // or 'SAMEORIGIN' - only same origin can frame
};

// 2. Content-Security-Policy frame-ancestors (modern)
const cspHeaders = {
  'Content-Security-Policy': "frame-ancestors 'none'", // Same as X-Frame-Options: DENY
  // or "frame-ancestors 'self'" for same origin
  // or "frame-ancestors 'self' trusted-site.com"
};

// 3. JavaScript frame busting (fallback)
if (window.top !== window.self) {
  window.top!.location = window.self.location;
}

// More robust frame busting
(function() {
  if (self === top) {
    document.documentElement.style.display = 'block';
  } else {
    top!.location = self.location;
  }
})();

// Next.js middleware
export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set(
    'Content-Security-Policy',
    "frame-ancestors 'none'"
  );
  
  return response;
}
```

---

# 15. Subresource Integrity (SRI)

## 15.1 What is SRI?

```html
<!-- SRI ensures external resources haven't been tampered with -->

<!-- Without SRI - vulnerable to CDN compromise -->
<script src="https://cdn.example.com/lib.js"></script>

<!-- With SRI - browser verifies hash before execution -->
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"
></script>
```

## 15.2 Generating SRI Hashes

```typescript
// Generate SRI hash for a resource
async function generateSRIHash(content: string): Promise<string> {
  const encoder = new TextEncoder();
  const data = encoder.encode(content);
  const hashBuffer = await crypto.subtle.digest('SHA-384', data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  const hashBase64 = btoa(String.fromCharCode(...hashArray));
  return `sha384-${hashBase64}`;
}

// Usage in build script
async function addSRIToScript(url: string): Promise<string> {
  const response = await fetch(url);
  const content = await response.text();
  const hash = await generateSRIHash(content);
  
  return `<script src="${url}" integrity="${hash}" crossorigin="anonymous"></script>`;
}
```

## 15.3 SRI in Next.js

```typescript
// next.config.js with SRI
const crypto = require('crypto');

module.exports = {
  generateBuildId: async () => {
    return 'my-build-id';
  },
  
  // Generate SRI for static assets
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.output.crossOriginLoading = 'anonymous';
    }
    return config;
  },
};
```

---

# 16. Security Headers Deep Dive

## 16.1 Complete Security Headers

```typescript
const SECURITY_HEADERS = [
  // Prevent XSS
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-inline' 'unsafe-eval';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
      frame-ancestors 'none';
    `.replace(/\s+/g, ' ').trim(),
  },
  
  // Prevent clickjacking
  {
    key: 'X-Frame-Options',
    value: 'DENY',
  },
  
  // Prevent MIME type sniffing
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  
  // Enable XSS filter (legacy browsers)
  {
    key: 'X-XSS-Protection',
    value: '1; mode=block',
  },
  
  // Control referrer information
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  
  // Force HTTPS
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=31536000; includeSubDomains; preload',
  },
  
  // Control browser features
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()',
  },
  
  // Prevent DNS prefetching
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'off',
  },
];
```

## 16.2 Permissions Policy

```typescript
// Control what browser features your site can use
const permissionsPolicy = {
  // Disable features entirely
  camera: '()',
  microphone: '()',
  geolocation: '()',
  
  // Allow for same origin only
  fullscreen: '(self)',
  
  // Allow for specific origins
  payment: '(self "https://payments.example.com")',
  
  // Allow for all (not recommended for most features)
  // autoplay: '*',
};

// As header
const header = Object.entries(permissionsPolicy)
  .map(([key, value]) => `${key}=${value}`)
  .join(', ');

// Result: camera=(), microphone=(), geolocation=(), fullscreen=(self)
```

---

# 17. Secure Forms

## 17.1 Form Security Best Practices

```tsx
function SecureLoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    // Client-side validation
    const validationErrors = validateLoginForm(formData);
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      setIsSubmitting(false);
      return;
    }
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        credentials: 'include', // For cookies
        body: JSON.stringify(formData),
      });
      
      if (!response.ok) {
        const error = await response.json();
        // Generic error message to prevent user enumeration
        setErrors({ form: 'Invalid email or password' });
        return;
      }
      
      // Clear sensitive data
      setFormData({ email: '', password: '' });
      
      // Redirect to dashboard
      window.location.href = '/dashboard';
    } catch (error) {
      setErrors({ form: 'An error occurred. Please try again.' });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} autoComplete="off">
      {/* Prevent autofill for security */}
      <input type="text" style={{ display: 'none' }} />
      <input type="password" style={{ display: 'none' }} />
      
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        autoComplete="email"
        required
        aria-describedby="email-error"
      />
      {errors.email && <span id="email-error" role="alert">{errors.email}</span>}
      
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={(e) => setFormData(prev => ({ ...prev, password: e.target.value }))}
        autoComplete="current-password"
        required
        minLength={12}
        aria-describedby="password-error"
      />
      {errors.password && <span id="password-error" role="alert">{errors.password}</span>}
      
      {errors.form && <div role="alert">{errors.form}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

## 17.2 Preventing Credential Leaks

```typescript
// 1. Never log credentials
function login(email: string, password: string) {
  // ❌ Never do this
  console.log('Login attempt:', { email, password });
  
  // ✅ Safe logging
  console.log('Login attempt for:', email);
}

// 2. Clear password from memory
function handleLogin(password: string) {
  // Process password
  const hash = hashPassword(password);
  
  // ❌ Password stays in memory
  // return hash;
  
  // ✅ Clear reference (limited effectiveness in JS)
  const result = hash;
  password = '';
  return result;
}

// 3. Disable autocomplete for sensitive fields
<input type="password" autoComplete="new-password" />

// 4. Use form action instead of JavaScript for sensitive forms
// This prevents the data from being accessible in memory
<form action="/api/login" method="POST">
  <input type="email" name="email" />
  <input type="password" name="password" />
  <button type="submit">Login</button>
</form>
```

---

# 18. API Security

## 18.1 Rate Limiting Handling

```typescript
class RateLimitedClient {
  private retryAfter = 0;
  
  async request(url: string, options?: RequestInit): Promise<Response> {
    // Check if we're still rate limited
    if (Date.now() < this.retryAfter) {
      const waitTime = this.retryAfter - Date.now();
      throw new Error(`Rate limited. Try again in ${Math.ceil(waitTime / 1000)}s`);
    }
    
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      // Get retry-after header
      const retryAfter = response.headers.get('Retry-After');
      
      if (retryAfter) {
        // Could be seconds or a date
        const seconds = parseInt(retryAfter, 10);
        if (!isNaN(seconds)) {
          this.retryAfter = Date.now() + seconds * 1000;
        } else {
          this.retryAfter = new Date(retryAfter).getTime();
        }
      } else {
        // Default to 60 seconds
        this.retryAfter = Date.now() + 60000;
      }
      
      throw new RateLimitError('Rate limit exceeded', this.retryAfter);
    }
    
    return response;
  }
}

class RateLimitError extends Error {
  constructor(message: string, public retryAfter: number) {
    super(message);
    this.name = 'RateLimitError';
  }
}
```

## 18.2 Request Signing

```typescript
// Sign requests to prevent tampering
async function signRequest(
  method: string,
  url: string,
  body: string,
  secret: string
): Promise<string> {
  const timestamp = Date.now().toString();
  const message = `${method}${url}${body}${timestamp}`;
  
  const encoder = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(secret),
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['sign']
  );
  
  const signature = await crypto.subtle.sign(
    'HMAC',
    key,
    encoder.encode(message)
  );
  
  return btoa(String.fromCharCode(...new Uint8Array(signature)));
}

// Usage
async function secureRequest(url: string, body: object) {
  const bodyString = JSON.stringify(body);
  const timestamp = Date.now().toString();
  const signature = await signRequest('POST', url, bodyString, API_SECRET);
  
  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Timestamp': timestamp,
      'X-Signature': signature,
    },
    body: bodyString,
  });
}
```

---

# 19. More Interview Questions

## Q4: What is the Same-Origin Policy?

**Answer:**
The Same-Origin Policy is a security mechanism that restricts how documents or scripts from one origin can interact with resources from another origin.

**Same origin = same protocol + host + port**

```typescript
// Examples (comparing to https://example.com:443)
const ORIGIN_COMPARISON = {
  'https://example.com:443/path': 'Same origin (path differs)',
  'https://example.com': 'Same origin (default port 443)',
  'http://example.com': 'Different origin (protocol)',
  'https://api.example.com': 'Different origin (host)',
  'https://example.com:8080': 'Different origin (port)',
};
```

---

## Q5: What is the difference between authentication and authorization?

**Answer:**

```typescript
const AUTH_COMPARISON = {
  authentication: {
    question: 'Who are you?',
    methods: ['Username/password', 'OAuth', 'SSO', 'Biometrics'],
    result: 'User identity verified',
    example: 'Login process',
  },
  authorization: {
    question: 'What can you do?',
    methods: ['RBAC', 'ABAC', 'ACL', 'Permissions'],
    result: 'Access rights determined',
    example: 'Admin vs regular user privileges',
  },
};

// Authentication happens first, then authorization
// 1. User proves identity (authentication)
// 2. System checks what user can access (authorization)
```

---

## Q6: How do you implement secure logout?

**Answer:**

```typescript
async function secureLogout() {
  // 1. Clear client-side state
  tokenStore.clearToken();
  
  // 2. Clear cookies via server
  await fetch('/api/auth/logout', {
    method: 'POST',
    credentials: 'include',
  });
  
  // 3. Clear any cached data
  queryClient.clear();
  
  // 4. Clear sensitive data from sessionStorage
  sessionStorage.clear();
  
  // 5. Redirect to login (use replace to prevent back button)
  window.location.replace('/login');
}

// Server-side logout
app.post('/api/auth/logout', (req, res) => {
  // Invalidate refresh token in database
  await invalidateRefreshToken(req.cookies.refreshToken);
  
  // Clear cookies
  res.clearCookie('accessToken', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
  });
  
  res.clearCookie('refreshToken', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    path: '/api/auth/refresh',
  });
  
  res.json({ success: true });
});
```

---

## Q7: What is Content Security Policy and why is it important?

**Answer:**
CSP is a security header that controls what resources can be loaded on a page, preventing XSS and data injection attacks.

```typescript
// CSP prevents:
// 1. Inline scripts (unless explicitly allowed)
// 2. Loading scripts from untrusted sources
// 3. Inline styles (optional)
// 4. Form submissions to external sites
// 5. Embedding in iframes

const strictCSP = `
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' https:;
  connect-src 'self';
  frame-ancestors 'none';
  form-action 'self';
  upgrade-insecure-requests;
`;

// Report violations
const cspWithReporting = `
  ${strictCSP}
  report-uri /api/csp-report;
`;
```

---

## Q8: How do you handle sensitive data in React?

**Answer:**

```typescript
// 1. Never store sensitive data in state that persists
// ❌ Bad
const [creditCard, setCreditCard] = useState('');

// ✅ Good - use refs for sensitive data
const creditCardRef = useRef('');

// 2. Clear sensitive data after use
useEffect(() => {
  return () => {
    creditCardRef.current = '';
  };
}, []);

// 3. Mask sensitive data in UI
function MaskedInput({ value }: { value: string }) {
  const masked = `****${value.slice(-4)}`;
  return <span>{masked}</span>;
}

// 4. Use secure input types
<input type="password" autoComplete="off" />

// 5. Disable copy/paste for sensitive fields
<input
  onCopy={(e) => e.preventDefault()}
  onPaste={(e) => e.preventDefault()}
  onCut={(e) => e.preventDefault()}
/>
```

---

## Q9: What is a Man-in-the-Middle attack?

**Answer:**
A MITM attack intercepts communication between two parties without their knowledge.

**Prevention:**
```typescript
const MITM_PREVENTION = {
  https: 'Encrypt all traffic with TLS',
  hsts: 'Force HTTPS with Strict-Transport-Security',
  certificatePinning: 'Verify specific certificates (mobile apps)',
  contentIntegrity: 'Use SRI for external resources',
};

// HSTS prevents downgrade attacks
const hstsHeader = {
  key: 'Strict-Transport-Security',
  value: 'max-age=31536000; includeSubDomains; preload',
};
```

---

## Q10: How do you prevent SQL injection in frontend?

**Answer:**
SQL injection is primarily a backend concern, but frontend can help:

```typescript
// 1. Validate input format
function validateId(id: string): boolean {
  return /^\d+$/.test(id); // Only digits
}

// 2. Never construct SQL-like queries client-side
// ❌ Bad (if you're using client-side DB like IndexedDB)
const query = `SELECT * FROM users WHERE name = '${userInput}'`;

// ✅ Good - parameterized queries
const query = 'SELECT * FROM users WHERE name = ?';
const params = [userInput];

// 3. Use ORMs and query builders
// They handle parameterization automatically
prisma.user.findMany({
  where: { name: userInput }, // Safely parameterized
});
```

---

# 20. More Coding Challenges

## Challenge 3: Implement Secure Password Input

```tsx
function SecurePasswordInput() {
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);
  
  // Clear password on unmount
  useEffect(() => {
    return () => {
      setPassword('');
    };
  }, []);
  
  // Prevent dev tools inspection
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.key === 'F12' || (e.ctrlKey && e.shiftKey && e.key === 'I')) {
        // Optional: clear password when dev tools opened
        setPassword('');
      }
    };
    
    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, []);
  
  return (
    <div className="password-input">
      <input
        ref={inputRef}
        type={showPassword ? 'text' : 'password'}
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        autoComplete="new-password"
        onCopy={(e) => e.preventDefault()}
        onCut={(e) => e.preventDefault()}
      />
      <button
        type="button"
        onClick={() => setShowPassword(!showPassword)}
        onBlur={() => setShowPassword(false)}
      >
        {showPassword ? '🙈' : '👁'}
      </button>
    </div>
  );
}
```

## Challenge 4: Implement Token Rotation

```typescript
class TokenRotation {
  private accessToken: string | null = null;
  private tokenVersion = 0;
  private refreshLock = false;
  
  async getToken(): Promise<string> {
    const currentVersion = this.tokenVersion;
    
    if (this.accessToken && !this.isExpired(this.accessToken)) {
      return this.accessToken;
    }
    
    // Wait if refresh is in progress
    while (this.refreshLock) {
      await new Promise(r => setTimeout(r, 100));
      
      // Check if token was refreshed
      if (this.tokenVersion !== currentVersion && this.accessToken) {
        return this.accessToken;
      }
    }
    
    return this.refreshToken();
  }
  
  private async refreshToken(): Promise<string> {
    this.refreshLock = true;
    
    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        credentials: 'include',
      });
      
      if (!response.ok) {
        throw new Error('Refresh failed');
      }
      
      const { accessToken, rotatedRefreshToken } = await response.json();
      
      this.accessToken = accessToken;
      this.tokenVersion++;
      
      return accessToken;
    } finally {
      this.refreshLock = false;
    }
  }
  
  private isExpired(token: string): boolean {
    const payload = JSON.parse(atob(token.split('.')[1]));
    return Date.now() >= payload.exp * 1000 - 30000; // 30s buffer
  }
}
```

## Challenge 5: Implement Content Security Policy Builder

```typescript
class CSPBuilder {
  private directives: Map<string, Set<string>> = new Map();
  
  defaultSrc(...sources: string[]): this {
    return this.addDirective('default-src', sources);
  }
  
  scriptSrc(...sources: string[]): this {
    return this.addDirective('script-src', sources);
  }
  
  styleSrc(...sources: string[]): this {
    return this.addDirective('style-src', sources);
  }
  
  imgSrc(...sources: string[]): this {
    return this.addDirective('img-src', sources);
  }
  
  connectSrc(...sources: string[]): this {
    return this.addDirective('connect-src', sources);
  }
  
  fontSrc(...sources: string[]): this {
    return this.addDirective('font-src', sources);
  }
  
  frameSrc(...sources: string[]): this {
    return this.addDirective('frame-src', sources);
  }
  
  frameAncestors(...sources: string[]): this {
    return this.addDirective('frame-ancestors', sources);
  }
  
  reportUri(uri: string): this {
    return this.addDirective('report-uri', [uri]);
  }
  
  private addDirective(name: string, sources: string[]): this {
    if (!this.directives.has(name)) {
      this.directives.set(name, new Set());
    }
    sources.forEach(s => this.directives.get(name)!.add(s));
    return this;
  }
  
  build(): string {
    return Array.from(this.directives.entries())
      .map(([directive, sources]) => 
        `${directive} ${Array.from(sources).join(' ')}`
      )
      .join('; ');
  }
}

// Usage
const csp = new CSPBuilder()
  .defaultSrc("'self'")
  .scriptSrc("'self'", "'unsafe-inline'")
  .styleSrc("'self'", "'unsafe-inline'")
  .imgSrc("'self'", 'https:', 'data:')
  .connectSrc("'self'", 'https://api.example.com')
  .frameAncestors("'none'")
  .build();

// Result: "default-src 'self'; script-src 'self' 'unsafe-inline'; ..."
```

## Challenge 6: Implement Session Timeout Handler

```typescript
class SessionManager {
  private timeoutId: NodeJS.Timeout | null = null;
  private warningId: NodeJS.Timeout | null = null;
  private lastActivity = Date.now();
  
  private readonly SESSION_TIMEOUT = 30 * 60 * 1000; // 30 minutes
  private readonly WARNING_BEFORE = 5 * 60 * 1000; // 5 minutes warning
  
  constructor(
    private onWarning: () => void,
    private onTimeout: () => void
  ) {
    this.setupActivityListeners();
    this.resetTimers();
  }
  
  private setupActivityListeners() {
    const events = ['mousedown', 'keydown', 'scroll', 'touchstart'];
    
    const activityHandler = this.throttle(() => {
      this.lastActivity = Date.now();
      this.resetTimers();
    }, 1000);
    
    events.forEach(event => {
      window.addEventListener(event, activityHandler);
    });
  }
  
  private resetTimers() {
    if (this.timeoutId) clearTimeout(this.timeoutId);
    if (this.warningId) clearTimeout(this.warningId);
    
    this.warningId = setTimeout(() => {
      this.onWarning();
    }, this.SESSION_TIMEOUT - this.WARNING_BEFORE);
    
    this.timeoutId = setTimeout(() => {
      this.onTimeout();
    }, this.SESSION_TIMEOUT);
  }
  
  extendSession() {
    this.resetTimers();
  }
  
  private throttle(fn: Function, delay: number) {
    let lastCall = 0;
    return () => {
      const now = Date.now();
      if (now - lastCall >= delay) {
        lastCall = now;
        fn();
      }
    };
  }
  
  destroy() {
    if (this.timeoutId) clearTimeout(this.timeoutId);
    if (this.warningId) clearTimeout(this.warningId);
  }
}

// Usage
const session = new SessionManager(
  () => {
    // Show warning modal
    showModal('Your session will expire in 5 minutes. Extend?');
  },
  () => {
    // Logout user
    logout();
    redirect('/login?reason=timeout');
  }
);
```

---

# 21. Security Audit Checklist

## 21.1 Authentication Checklist

```typescript
const AUTH_CHECKLIST = [
  // Password security
  { item: 'Minimum 12 character passwords', status: false },
  { item: 'Password strength indicator', status: false },
  { item: 'No password in URL params', status: false },
  { item: 'Secure password reset flow', status: false },
  
  // Token security
  { item: 'HttpOnly cookies for tokens', status: false },
  { item: 'Short-lived access tokens (< 15 min)', status: false },
  { item: 'Refresh token rotation', status: false },
  { item: 'Token invalidation on logout', status: false },
  
  // Session security
  { item: 'Session timeout implemented', status: false },
  { item: 'Activity-based session extension', status: false },
  { item: 'Concurrent session limits', status: false },
];
```

## 21.2 XSS Prevention Checklist

```typescript
const XSS_CHECKLIST = [
  { item: 'No dangerouslySetInnerHTML without sanitization', status: false },
  { item: 'DOMPurify for user HTML content', status: false },
  { item: 'URL validation for user-provided links', status: false },
  { item: 'CSP header configured', status: false },
  { item: 'No inline event handlers', status: false },
  { item: 'No eval() or new Function()', status: false },
];
```

## 21.3 Data Protection Checklist

```typescript
const DATA_CHECKLIST = [
  { item: 'HTTPS enforced', status: false },
  { item: 'HSTS header set', status: false },
  { item: 'No sensitive data in localStorage', status: false },
  { item: 'API responses sanitized', status: false },
  { item: 'Error messages dont leak info', status: false },
  { item: 'Logging excludes sensitive data', status: false },
];
```

---

# 22. Third-Party Script Security

## 22.1 Risks of Third-Party Scripts

```typescript
const THIRD_PARTY_RISKS = {
  dataExfiltration: 'Scripts can access your DOM and send data to external servers',
  sessionHijacking: 'Scripts can steal cookies if not HttpOnly',
  keylogging: 'Scripts can capture user input',
  domManipulation: 'Scripts can modify your page content',
  supplyChainAttack: 'Compromised CDN serves malicious code',
};

// Common third-party scripts:
// - Analytics (Google Analytics, Mixpanel)
// - Ads networks
// - Chat widgets
// - Social media embeds
// - Customer support tools
```

## 22.2 Securing Third-Party Scripts

```typescript
// 1. Use Subresource Integrity
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123"
  crossorigin="anonymous"
></script>

// 2. Load in isolated iframe
function loadThirdPartyInIframe(src: string) {
  const iframe = document.createElement('iframe');
  iframe.src = src;
  iframe.sandbox = 'allow-scripts'; // Restrict capabilities
  iframe.style.display = 'none';
  document.body.appendChild(iframe);
}

// 3. Defer loading until needed
function lazyLoadScript(src: string): Promise<void> {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    script.onload = () => resolve();
    script.onerror = reject;
    document.head.appendChild(script);
  });
}

// 4. Monitor for changes
function monitorScriptIntegrity(scripts: string[]) {
  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      mutation.addedNodes.forEach((node) => {
        if (node.nodeName === 'SCRIPT') {
          const script = node as HTMLScriptElement;
          if (!scripts.includes(script.src)) {
            console.warn('Unauthorized script detected:', script.src);
            script.remove();
          }
        }
      });
    });
  });
  
  observer.observe(document.head, { childList: true });
  observer.observe(document.body, { childList: true });
}
```

## 22.3 Sandboxed Iframe Pattern

```typescript
// Embed third-party content safely
function SecureEmbed({ src, allow }: { src: string; allow?: string }) {
  return (
    <iframe
      src={src}
      sandbox={`allow-scripts ${allow || ''}`}
      referrerPolicy="no-referrer"
      loading="lazy"
      style={{ border: 'none' }}
    />
  );
}

// Sandbox attribute values:
// allow-scripts - Allow JavaScript
// allow-same-origin - Allow same-origin access
// allow-forms - Allow form submission
// allow-popups - Allow popup windows
// allow-modals - Allow alert/confirm/prompt

// Most restrictive: sandbox="" (blocks almost everything)
```

---

# 23. Secure Communication

## 23.1 PostMessage Security

```typescript
// Sending messages
function sendSecureMessage(targetWindow: Window, message: any) {
  // Always specify exact target origin
  targetWindow.postMessage(message, 'https://trusted.example.com');
  
  // ❌ Never use '*' for sensitive messages
  // targetWindow.postMessage(message, '*');
}

// Receiving messages
function setupSecureMessageListener() {
  window.addEventListener('message', (event) => {
    // 1. Validate origin
    const allowedOrigins = [
      'https://trusted.example.com',
      'https://another-trusted.com',
    ];
    
    if (!allowedOrigins.includes(event.origin)) {
      console.warn('Message from untrusted origin:', event.origin);
      return;
    }
    
    // 2. Validate message structure
    if (!isValidMessage(event.data)) {
      console.warn('Invalid message structure:', event.data);
      return;
    }
    
    // 3. Process message
    handleMessage(event.data);
  });
}

function isValidMessage(data: unknown): data is { type: string; payload: any } {
  return (
    typeof data === 'object' &&
    data !== null &&
    'type' in data &&
    typeof (data as any).type === 'string'
  );
}
```

## 23.2 Secure WebSocket Communication

```typescript
class SecureWebSocket {
  private ws: WebSocket;
  private messageQueue: any[] = [];
  
  constructor(url: string) {
    // 1. Always use wss:// (WebSocket Secure)
    if (!url.startsWith('wss://')) {
      throw new Error('Must use secure WebSocket (wss://)');
    }
    
    this.ws = new WebSocket(url);
    this.setupHandlers();
  }
  
  private setupHandlers() {
    this.ws.onmessage = (event) => {
      try {
        const message = JSON.parse(event.data);
        
        // Validate message structure
        if (!this.isValidServerMessage(message)) {
          console.warn('Invalid server message');
          return;
        }
        
        // Sanitize any HTML content
        if (message.html) {
          message.html = DOMPurify.sanitize(message.html);
        }
        
        this.handleMessage(message);
      } catch (error) {
        console.error('Failed to parse WebSocket message');
      }
    };
  }
  
  send(message: object) {
    // Never send sensitive data over WebSocket without encryption
    if (this.containsSensitiveData(message)) {
      throw new Error('Cannot send sensitive data');
    }
    
    this.ws.send(JSON.stringify(message));
  }
  
  private containsSensitiveData(message: object): boolean {
    const sensitiveKeys = ['password', 'token', 'secret', 'creditCard'];
    const messageStr = JSON.stringify(message).toLowerCase();
    return sensitiveKeys.some((key) => messageStr.includes(key));
  }
  
  private isValidServerMessage(message: unknown): boolean {
    return typeof message === 'object' && message !== null;
  }
  
  private handleMessage(message: any) {
    // Handle validated message
  }
}
```

---

# 24. Error Handling Security

## 24.1 Secure Error Messages

```typescript
// ❌ Bad - exposes internal details
function handleError(error: Error) {
  return {
    error: error.message,
    stack: error.stack,
    details: {
      query: 'SELECT * FROM users WHERE id = 123',
      server: 'db-primary-01',
    },
  };
}

// ✅ Good - safe error messages
function handleErrorSecurely(error: Error): ApiError {
  // Log full error server-side
  console.error('Internal error:', error);
  
  // Return safe error to client
  return {
    error: 'An error occurred',
    code: 'INTERNAL_ERROR',
    requestId: generateRequestId(), // For support debugging
  };
}

// Map internal errors to safe messages
const ERROR_MESSAGES: Record<string, string> = {
  'INVALID_TOKEN': 'Session expired. Please login again.',
  'USER_NOT_FOUND': 'Invalid credentials.', // Don't reveal user exists
  'DUPLICATE_EMAIL': 'Invalid credentials.', // Same message to prevent enumeration
  'RATE_LIMITED': 'Too many requests. Please try again later.',
  'VALIDATION_ERROR': 'Please check your input.',
  'default': 'An error occurred. Please try again.',
};

function getSafeErrorMessage(errorCode: string): string {
  return ERROR_MESSAGES[errorCode] || ERROR_MESSAGES.default;
}
```

## 24.2 React Error Boundary for Security

```tsx
interface ErrorBoundaryState {
  hasError: boolean;
  errorId: string | null;
}

class SecureErrorBoundary extends React.Component<
  { children: React.ReactNode },
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = { hasError: false, errorId: null };
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return {
      hasError: true,
      errorId: crypto.randomUUID(),
    };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to monitoring service (not exposed to user)
    logToMonitoring({
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      errorId: this.state.errorId,
    });
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-page">
          <h1>Something went wrong</h1>
          <p>
            We've been notified of the issue. If you need help,
            reference ID: {this.state.errorId}
          </p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

---

# 25. Privacy and Data Protection

## 25.1 GDPR Compliance Helpers

```typescript
// Cookie consent management
interface ConsentPreferences {
  necessary: true; // Always required
  analytics: boolean;
  marketing: boolean;
  preferences: boolean;
}

class ConsentManager {
  private static CONSENT_KEY = 'cookie_consent';
  
  static getConsent(): ConsentPreferences | null {
    const saved = localStorage.getItem(this.CONSENT_KEY);
    if (!saved) return null;
    
    try {
      return JSON.parse(saved);
    } catch {
      return null;
    }
  }
  
  static setConsent(preferences: ConsentPreferences) {
    localStorage.setItem(this.CONSENT_KEY, JSON.stringify(preferences));
    this.applyConsent(preferences);
  }
  
  static applyConsent(preferences: ConsentPreferences) {
    if (preferences.analytics) {
      this.enableAnalytics();
    } else {
      this.disableAnalytics();
    }
    
    if (preferences.marketing) {
      this.enableMarketing();
    } else {
      this.disableMarketing();
    }
  }
  
  private static enableAnalytics() {
    // Load analytics scripts
  }
  
  private static disableAnalytics() {
    // Remove analytics cookies
    document.cookie.split(';').forEach((c) => {
      if (c.trim().startsWith('_ga') || c.trim().startsWith('_gid')) {
        document.cookie = c.replace(/^ +/, '').replace(/=.*/, '=;expires=Thu, 01 Jan 1970 00:00:00 GMT;path=/');
      }
    });
  }
  
  private static enableMarketing() {
    // Load marketing scripts
  }
  
  private static disableMarketing() {
    // Remove marketing cookies
  }
}

// React component
function CookieConsent() {
  const [showBanner, setShowBanner] = useState(false);
  
  useEffect(() => {
    if (!ConsentManager.getConsent()) {
      setShowBanner(true);
    }
  }, []);
  
  const handleAcceptAll = () => {
    ConsentManager.setConsent({
      necessary: true,
      analytics: true,
      marketing: true,
      preferences: true,
    });
    setShowBanner(false);
  };
  
  const handleRejectOptional = () => {
    ConsentManager.setConsent({
      necessary: true,
      analytics: false,
      marketing: false,
      preferences: false,
    });
    setShowBanner(false);
  };
  
  if (!showBanner) return null;
  
  return (
    <div className="cookie-banner">
      <p>We use cookies to improve your experience.</p>
      <button onClick={handleAcceptAll}>Accept All</button>
      <button onClick={handleRejectOptional}>Essential Only</button>
      <a href="/privacy">Learn More</a>
    </div>
  );
}
```

## 25.2 Data Anonymization

```typescript
// Anonymize user data for analytics
function anonymizeUser(user: User): AnonymizedUser {
  return {
    id: hashString(user.id), // One-way hash
    ageRange: getAgeRange(user.birthDate),
    region: user.country, // Keep general location
    // Don't include: name, email, exact address, phone
  };
}

function hashString(str: string): string {
  let hash = 5381;
  for (let i = 0; i < str.length; i++) {
    hash = (hash * 33) ^ str.charCodeAt(i);
  }
  return (hash >>> 0).toString(16);
}

function getAgeRange(birthDate: Date): string {
  const age = new Date().getFullYear() - birthDate.getFullYear();
  if (age < 18) return 'under-18';
  if (age < 25) return '18-24';
  if (age < 35) return '25-34';
  if (age < 45) return '35-44';
  if (age < 55) return '45-54';
  if (age < 65) return '55-64';
  return '65+';
}

// Mask PII in logs
function maskPII(data: Record<string, any>): Record<string, any> {
  const piiFields = ['email', 'phone', 'ssn', 'creditCard', 'password'];
  const masked = { ...data };
  
  for (const field of piiFields) {
    if (field in masked) {
      masked[field] = '***REDACTED***';
    }
  }
  
  return masked;
}
```

---

# 26. Security Testing

## 26.1 Automated Security Testing

```typescript
// security.test.ts
import { describe, it, expect } from 'vitest';

describe('XSS Prevention', () => {
  it('should escape HTML in user input', () => {
    const userInput = '<script>alert("xss")</script>';
    const escaped = escapeHtml(userInput);
    
    expect(escaped).not.toContain('<script>');
    expect(escaped).toContain('&lt;script&gt;');
  });
  
  it('should sanitize URLs', () => {
    const maliciousUrl = 'javascript:alert("xss")';
    const sanitized = sanitizeUrl(maliciousUrl);
    
    expect(sanitized).toBe('#');
  });
  
  it('should reject URLs with data protocol', () => {
    const dataUrl = 'data:text/html,<script>alert("xss")</script>';
    expect(isValidUrl(dataUrl)).toBe(false);
  });
});

describe('CSRF Protection', () => {
  it('should include CSRF token in requests', async () => {
    const mockFetch = vi.fn().mockResolvedValue({ ok: true });
    global.fetch = mockFetch;
    
    await securePost('/api/data', { foo: 'bar' });
    
    expect(mockFetch).toHaveBeenCalledWith(
      '/api/data',
      expect.objectContaining({
        headers: expect.objectContaining({
          'X-CSRF-Token': expect.any(String),
        }),
      })
    );
  });
});

describe('Password Security', () => {
  it('should require minimum password length', () => {
    const result = validatePassword('short');
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Password must be at least 12 characters');
  });
  
  it('should require special characters', () => {
    const result = validatePassword('NoSpecialChars123');
    expect(result.valid).toBe(false);
  });
  
  it('should accept strong passwords', () => {
    const result = validatePassword('MyStr0ng!P@ssword');
    expect(result.valid).toBe(true);
  });
});

describe('Token Security', () => {
  it('should detect expired tokens', () => {
    const expiredToken = createMockJWT({ exp: Date.now() / 1000 - 3600 });
    expect(isTokenExpired(expiredToken)).toBe(true);
  });
  
  it('should validate token format', () => {
    expect(isValidJWTFormat('not.a.valid.jwt')).toBe(false);
    expect(isValidJWTFormat('header.payload.signature')).toBe(true);
  });
});
```

## 26.2 Security Headers Testing

```typescript
// test/security-headers.test.ts
describe('Security Headers', () => {
  it('should have Content-Security-Policy header', async () => {
    const response = await fetch('/');
    const csp = response.headers.get('Content-Security-Policy');
    
    expect(csp).toBeTruthy();
    expect(csp).toContain("default-src 'self'");
  });
  
  it('should have X-Frame-Options header', async () => {
    const response = await fetch('/');
    const xfo = response.headers.get('X-Frame-Options');
    
    expect(xfo).toBe('DENY');
  });
  
  it('should have Strict-Transport-Security header', async () => {
    const response = await fetch('/');
    const hsts = response.headers.get('Strict-Transport-Security');
    
    expect(hsts).toContain('max-age=');
    expect(hsts).toContain('includeSubDomains');
  });
  
  it('should have X-Content-Type-Options header', async () => {
    const response = await fetch('/');
    const xcto = response.headers.get('X-Content-Type-Options');
    
    expect(xcto).toBe('nosniff');
  });
});
```

---

# 27. Real-World Security Patterns

## 27.1 Rate Limiting UI

```tsx
function RateLimitedButton({
  onClick,
  children,
  cooldownMs = 1000,
}: {
  onClick: () => Promise<void>;
  children: React.ReactNode;
  cooldownMs?: number;
}) {
  const [isLoading, setIsLoading] = useState(false);
  const [cooldown, setCooldown] = useState(0);
  
  const handleClick = async () => {
    if (isLoading || cooldown > 0) return;
    
    setIsLoading(true);
    
    try {
      await onClick();
    } catch (error) {
      if (error instanceof RateLimitError) {
        setCooldown(Math.ceil((error.retryAfter - Date.now()) / 1000));
      }
    } finally {
      setIsLoading(false);
    }
  };
  
  useEffect(() => {
    if (cooldown <= 0) return;
    
    const timer = setInterval(() => {
      setCooldown((prev) => Math.max(0, prev - 1));
    }, 1000);
    
    return () => clearInterval(timer);
  }, [cooldown]);
  
  return (
    <button onClick={handleClick} disabled={isLoading || cooldown > 0}>
      {cooldown > 0 ? `Wait ${cooldown}s` : isLoading ? 'Loading...' : children}
    </button>
  );
}
```

## 27.2 Secure File Upload

```tsx
const ALLOWED_FILE_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

function SecureFileUpload({
  onUpload,
}: {
  onUpload: (file: File) => Promise<void>;
}) {
  const [error, setError] = useState<string | null>(null);
  
  const validateFile = (file: File): string | null => {
    // Check file type
    if (!ALLOWED_FILE_TYPES.includes(file.type)) {
      return 'File type not allowed';
    }
    
    // Check file size
    if (file.size > MAX_FILE_SIZE) {
      return 'File too large (max 5MB)';
    }
    
    // Check file name for suspicious patterns
    if (/[<>"'&;]/.test(file.name)) {
      return 'Invalid file name';
    }
    
    // Verify magic bytes (for images)
    // Would need to read file content for full validation
    
    return null;
  };
  
  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    
    setError(null);
    
    const validationError = validateFile(file);
    if (validationError) {
      setError(validationError);
      e.target.value = ''; // Reset input
      return;
    }
    
    try {
      await onUpload(file);
    } catch (err) {
      setError('Upload failed');
    }
  };
  
  return (
    <div>
      <input
        type="file"
        accept={ALLOWED_FILE_TYPES.join(',')}
        onChange={handleFileChange}
      />
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

## 27.3 Audit Logging

```typescript
interface AuditLog {
  timestamp: Date;
  userId: string;
  action: string;
  resource: string;
  resourceId: string;
  details: Record<string, any>;
  ipAddress: string;
  userAgent: string;
}

class AuditLogger {
  private logs: AuditLog[] = [];
  
  async log(entry: Omit<AuditLog, 'timestamp' | 'ipAddress' | 'userAgent'>) {
    const auditLog: AuditLog = {
      ...entry,
      timestamp: new Date(),
      ipAddress: await this.getClientIP(),
      userAgent: navigator.userAgent,
    };
    
    // Remove any sensitive data
    if (auditLog.details) {
      auditLog.details = maskPII(auditLog.details);
    }
    
    this.logs.push(auditLog);
    
    // Send to server
    await fetch('/api/audit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(auditLog),
    });
  }
  
  private async getClientIP(): Promise<string> {
    // This would typically come from the server
    return 'client-ip-from-server';
  }
}

// Usage
const auditLogger = new AuditLogger();

async function deleteUser(userId: string) {
  await auditLogger.log({
    userId: currentUser.id,
    action: 'DELETE_USER',
    resource: 'user',
    resourceId: userId,
    details: { deletedBy: currentUser.name },
  });
  
  await api.deleteUser(userId);
}
```

---

# 28. More Interview Questions

## Q11: How do you prevent brute force attacks on login?

**Answer:**

```typescript
const BRUTE_FORCE_PREVENTION = {
  rateLimiting: 'Limit login attempts per IP/user',
  progressiveDelays: 'Increase delay after each failed attempt',
  accountLockout: 'Lock account after N failed attempts',
  captcha: 'Add CAPTCHA after several failures',
  mfa: 'Require second factor for sensitive accounts',
};

// Progressive delay implementation
class LoginRateLimiter {
  private attempts = new Map<string, { count: number; lastAttempt: Date }>();
  
  private readonly MAX_ATTEMPTS = 5;
  private readonly LOCKOUT_DURATION = 15 * 60 * 1000; // 15 minutes
  
  canAttempt(identifier: string): { allowed: boolean; retryAfter?: number } {
    const record = this.attempts.get(identifier);
    
    if (!record) {
      return { allowed: true };
    }
    
    const timeSinceLastAttempt = Date.now() - record.lastAttempt.getTime();
    
    if (record.count >= this.MAX_ATTEMPTS) {
      if (timeSinceLastAttempt < this.LOCKOUT_DURATION) {
        return {
          allowed: false,
          retryAfter: Math.ceil((this.LOCKOUT_DURATION - timeSinceLastAttempt) / 1000),
        };
      }
      
      // Reset after lockout period
      this.attempts.delete(identifier);
      return { allowed: true };
    }
    
    return { allowed: true };
  }
  
  recordFailure(identifier: string) {
    const record = this.attempts.get(identifier);
    
    if (record) {
      record.count++;
      record.lastAttempt = new Date();
    } else {
      this.attempts.set(identifier, {
        count: 1,
        lastAttempt: new Date(),
      });
    }
  }
  
  recordSuccess(identifier: string) {
    this.attempts.delete(identifier);
  }
}
```

---

## Q12: What is certificate pinning?

**Answer:**
Certificate pinning validates that the server's certificate matches a known certificate, preventing MITM attacks even with compromised CAs.

```typescript
// Mobile apps typically implement certificate pinning
// In React Native or native apps:

const PINNED_CERTIFICATES = [
  'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',
  'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=',
];

// Web apps can use Public-Key-Pins header (deprecated)
// Or use modern Expect-CT header
const expectCTHeader = {
  key: 'Expect-CT',
  value: 'max-age=86400, enforce, report-uri="https://example.com/report-ct"',
};
```

---

## Q13: How do you handle security in a microservices architecture?

**Answer:**

```typescript
const MICROSERVICES_SECURITY = {
  serviceToService: {
    mTLS: 'Mutual TLS for service authentication',
    jwtValidation: 'Verify JWT in each service',
    apiGateway: 'Central auth at gateway',
  },
  
  userAuth: {
    centralIdP: 'Use Identity Provider (Keycloak, Auth0)',
    tokenPropagation: 'Pass user token through services',
    scopedPermissions: 'Service-specific permissions',
  },
  
  networkSecurity: {
    serviceMesh: 'Use Istio or similar',
    privateNetwork: 'Services not publicly accessible',
    secretsManagement: 'Use Vault or similar',
  },
};

// API Gateway integration
async function authenticatedRequest(
  url: string,
  options: RequestInit = {}
): Promise<Response> {
  const token = await authService.getAccessToken();
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${token}`,
      'X-Request-ID': crypto.randomUUID(), // For tracing
    },
  });
}
```

---

## Q14: What is defense in depth?

**Answer:**
Defense in depth uses multiple layers of security controls so that if one layer fails, others still protect the system.

```typescript
const DEFENSE_IN_DEPTH_LAYERS = {
  network: {
    firewall: 'Block unauthorized access',
    waf: 'Web Application Firewall',
    ddosProtection: 'CDN-level protection',
  },
  
  application: {
    inputValidation: 'Sanitize all input',
    outputEncoding: 'Escape all output',
    csp: 'Content Security Policy',
    cors: 'Strict CORS configuration',
  },
  
  authentication: {
    strongPasswords: 'Enforce complexity',
    mfa: 'Multi-factor authentication',
    sessionManagement: 'Secure token handling',
  },
  
  data: {
    encryption: 'Data at rest and in transit',
    tokenization: 'Replace sensitive data with tokens',
    accessControl: 'Role-based permissions',
  },
  
  monitoring: {
    logging: 'Audit all actions',
    alerting: 'Detect anomalies',
    incidentResponse: 'Defined procedures',
  },
};
```

---

## Q15: How do you implement secure password reset?

**Answer:**

```typescript
// Secure password reset flow
async function initiatePasswordReset(email: string) {
  // 1. Don't reveal if email exists
  // Always show same message

  const user = await findUserByEmail(email);
  
  if (user) {
    // 2. Generate secure random token
    const resetToken = crypto.randomBytes(32).toString('hex');
    const tokenHash = await hash(resetToken); // Store hash, not token
    
    // 3. Short expiration
    const expiresAt = new Date(Date.now() + 15 * 60 * 1000); // 15 minutes
    
    await storeResetToken(user.id, tokenHash, expiresAt);
    
    // 4. Send email with token (token in URL, not email body visible)
    await sendEmail({
      to: email,
      subject: 'Password Reset',
      body: `Click to reset: https://example.com/reset?token=${resetToken}`,
    });
  }
  
  // 5. Always return same response
  return { message: 'If the email exists, a reset link will be sent.' };
}

async function completePasswordReset(token: string, newPassword: string) {
  // 1. Hash the provided token
  const tokenHash = await hash(token);
  
  // 2. Find valid reset request
  const resetRequest = await findResetRequest(tokenHash);
  
  if (!resetRequest || resetRequest.expiresAt < new Date()) {
    throw new Error('Invalid or expired reset link');
  }
  
  // 3. Validate new password
  if (!isStrongPassword(newPassword)) {
    throw new Error('Password does not meet requirements');
  }
  
  // 4. Update password
  await updateUserPassword(resetRequest.userId, newPassword);
  
  // 5. Invalidate all sessions
  await invalidateUserSessions(resetRequest.userId);
  
  // 6. Delete reset token
  await deleteResetToken(tokenHash);
  
  // 7. Notify user via email
  await sendEmail({
    to: resetRequest.email,
    subject: 'Password Changed',
    body: 'Your password has been changed. If this wasn\'t you, contact support.',
  });
}
```

---

# 29. Quick Reference Tables

## 29.1 Security Headers Reference

| Header | Purpose | Example Value |
|--------|---------|---------------|
| `Content-Security-Policy` | Prevent XSS | `default-src 'self'` |
| `X-Frame-Options` | Prevent clickjacking | `DENY` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `Strict-Transport-Security` | Force HTTPS | `max-age=31536000` |
| `Referrer-Policy` | Control referrer | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Control features | `camera=(), microphone=()` |

## 29.2 Storage Security Reference

| Storage | XSS Safe | CSRF Safe | Persistence | Best For |
|---------|----------|-----------|-------------|----------|
| HttpOnly Cookie | ✅ | ⚠️ | Persistent | Auth tokens |
| localStorage | ❌ | ✅ | Persistent | Preferences |
| sessionStorage | ❌ | ✅ | Session | Temp data |
| Memory | ⚠️ | ✅ | None | Short-lived tokens |

## 29.3 Common Vulnerabilities Reference

| Vulnerability | Prevention |
|--------------|------------|
| XSS | Sanitize, CSP, HttpOnly |
| CSRF | Tokens, SameSite, Origin check |
| SQL Injection | Parameterized queries |
| Clickjacking | X-Frame-Options, CSP |
| MITM | HTTPS, HSTS, Cert validation |
| Brute Force | Rate limiting, Lockout, CAPTCHA |

---

# 30. Security Tools Reference

## 30.1 Development Tools

```bash
# Check for vulnerabilities
npm audit
npm audit fix

# Snyk security scanning
npx snyk test

# OWASP ZAP for web app scanning
zaproxy -quickurl https://example.com

# Security linting
npm install eslint-plugin-security
```

## 30.2 Browser DevTools Security

```typescript
// Check for insecure resources
// DevTools -> Security tab

// View CSP violations
// DevTools -> Console (look for CSP errors)

// Check cookie security
// DevTools -> Application -> Cookies
// Look for: HttpOnly, Secure, SameSite flags

// Monitor network security
// DevTools -> Network -> Waterfall
// Look for mixed content warnings
```

---

# 31. Multi-Factor Authentication (MFA)

## 31.1 TOTP Implementation

```typescript
// Time-based One-Time Password (TOTP)
// Using speakeasy library

import speakeasy from 'speakeasy';
import QRCode from 'qrcode';

// Server-side: Generate secret
function generateTOTPSecret(userEmail: string): {
  secret: string;
  otpauthUrl: string;
  qrCodeDataUrl: Promise<string>;
} {
  const secret = speakeasy.generateSecret({
    name: `MyApp:${userEmail}`,
    issuer: 'MyApp',
  });
  
  return {
    secret: secret.base32,
    otpauthUrl: secret.otpauth_url!,
    qrCodeDataUrl: QRCode.toDataURL(secret.otpauth_url!),
  };
}

// Server-side: Verify TOTP
function verifyTOTP(secret: string, token: string): boolean {
  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 1, // Allow 1 step drift (30 seconds)
  });
}

// React component for MFA setup
function MFASetup() {
  const [step, setStep] = useState<'generate' | 'verify'>('generate');
  const [qrCode, setQrCode] = useState<string>('');
  const [secret, setSecret] = useState<string>('');
  const [verificationCode, setVerificationCode] = useState('');
  
  const generateSecret = async () => {
    const response = await fetch('/api/mfa/generate', { method: 'POST' });
    const data = await response.json();
    setQrCode(data.qrCodeDataUrl);
    setSecret(data.secret);
  };
  
  const verifyAndEnable = async () => {
    const response = await fetch('/api/mfa/verify', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ code: verificationCode }),
    });
    
    if (response.ok) {
      toast.success('MFA enabled successfully');
    } else {
      toast.error('Invalid code');
    }
  };
  
  useEffect(() => {
    generateSecret();
  }, []);
  
  return (
    <div className="mfa-setup">
      <h2>Set Up Two-Factor Authentication</h2>
      
      <div className="step">
        <h3>1. Scan QR Code</h3>
        <p>Use Google Authenticator or similar app</p>
        {qrCode && <img src={qrCode} alt="MFA QR Code" />}
      </div>
      
      <div className="step">
        <h3>2. Enter Verification Code</h3>
        <input
          type="text"
          maxLength={6}
          value={verificationCode}
          onChange={(e) => setVerificationCode(e.target.value.replace(/\D/g, ''))}
          placeholder="000000"
        />
        <button onClick={verifyAndEnable}>Verify & Enable</button>
      </div>
      
      <div className="backup">
        <h3>Backup Codes</h3>
        <p>Save these codes in a safe place:</p>
        {/* Display backup codes */}
      </div>
    </div>
  );
}
```

## 31.2 WebAuthn (Passkeys)

```typescript
// WebAuthn / Passkeys implementation
async function registerPasskey() {
  // 1. Get challenge from server
  const response = await fetch('/api/webauthn/register/challenge');
  const options = await response.json();
  
  // 2. Create credential
  const credential = await navigator.credentials.create({
    publicKey: {
      challenge: Uint8Array.from(
        atob(options.challenge),
        c => c.charCodeAt(0)
      ),
      rp: {
        name: 'MyApp',
        id: window.location.hostname,
      },
      user: {
        id: Uint8Array.from(options.userId, c => c.charCodeAt(0)),
        name: options.userEmail,
        displayName: options.userName,
      },
      pubKeyCredParams: [
        { alg: -7, type: 'public-key' },  // ES256
        { alg: -257, type: 'public-key' }, // RS256
      ],
      timeout: 60000,
      attestation: 'direct',
      authenticatorSelection: {
        authenticatorAttachment: 'platform',
        userVerification: 'required',
        residentKey: 'required',
      },
    },
  }) as PublicKeyCredential;
  
  // 3. Send to server
  const attestation = credential.response as AuthenticatorAttestationResponse;
  
  await fetch('/api/webauthn/register/verify', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: credential.id,
      rawId: btoa(String.fromCharCode(...new Uint8Array(credential.rawId))),
      clientDataJSON: btoa(String.fromCharCode(...new Uint8Array(attestation.clientDataJSON))),
      attestationObject: btoa(String.fromCharCode(...new Uint8Array(attestation.attestationObject))),
    }),
  });
}

async function authenticateWithPasskey() {
  // 1. Get challenge
  const response = await fetch('/api/webauthn/auth/challenge');
  const options = await response.json();
  
  // 2. Get credential
  const credential = await navigator.credentials.get({
    publicKey: {
      challenge: Uint8Array.from(
        atob(options.challenge),
        c => c.charCodeAt(0)
      ),
      timeout: 60000,
      rpId: window.location.hostname,
      userVerification: 'required',
    },
  }) as PublicKeyCredential;
  
  // 3. Verify with server
  const assertion = credential.response as AuthenticatorAssertionResponse;
  
  const result = await fetch('/api/webauthn/auth/verify', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: credential.id,
      clientDataJSON: btoa(String.fromCharCode(...new Uint8Array(assertion.clientDataJSON))),
      authenticatorData: btoa(String.fromCharCode(...new Uint8Array(assertion.authenticatorData))),
      signature: btoa(String.fromCharCode(...new Uint8Array(assertion.signature))),
    }),
  });
  
  return result.json();
}
```

---

# 32. Output-Based Security Questions

## Question 1: What's wrong with this code?

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);
  
  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <div dangerouslySetInnerHTML={{ __html: user?.bio }} />
    </div>
  );
}
```

**Answer:**

Security issues:
1. **XSS vulnerability**: `dangerouslySetInnerHTML` without sanitization
2. **No auth check**: Anyone can access any user profile
3. **No input validation**: userId not validated

**Fixed version:**

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Validate userId format
    if (!/^\d+$/.test(userId)) return;
    
    fetch(`/api/users/${userId}`, {
      credentials: 'include', // Send auth cookies
    })
      .then(res => {
        if (!res.ok) throw new Error('Not authorized');
        return res.json();
      })
      .then(data => setUser(data))
      .catch(err => console.error(err));
  }, [userId]);
  
  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      {/* Sanitize HTML */}
      <div dangerouslySetInnerHTML={{ 
        __html: DOMPurify.sanitize(user?.bio || '') 
      }} />
    </div>
  );
}
```

---

## Question 2: Identify the vulnerabilities

```typescript
app.post('/login', (req, res) => {
  const { email, password } = req.body;
  
  const user = db.query(`SELECT * FROM users WHERE email = '${email}'`);
  
  if (user && user.password === password) {
    const token = jwt.sign({ userId: user.id }, 'secret123');
    res.json({ token, message: `Welcome ${user.name}!` });
  } else {
    res.status(401).json({ error: `User ${email} not found` });
  }
});
```

**Answer:**

Vulnerabilities:
1. **SQL Injection**: `email` directly interpolated in query
2. **Plain text password**: Comparing without hashing
3. **Weak JWT secret**: 'secret123' is easily guessable
4. **User enumeration**: Different error for existing vs non-existing user
5. **Information disclosure**: Revealing email in error message

**Fixed version:**

```typescript
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Parameterized query
  const user = await db.query(
    'SELECT * FROM users WHERE email = $1',
    [email]
  );
  
  // Compare hashed password
  const isValid = user && await bcrypt.compare(password, user.passwordHash);
  
  if (isValid) {
    const token = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET, // Use env variable
      { expiresIn: '15m' }
    );
    res.json({ token });
  } else {
    // Generic error message
    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

---

## Question 3: What's the security issue?

```typescript
function SearchResults() {
  const params = new URLSearchParams(window.location.search);
  const query = params.get('q');
  
  document.getElementById('search-title')!.innerHTML = 
    `Results for: ${query}`;
  
  return <div id="search-title"></div>;
}
```

**Answer:**

**DOM-based XSS**: The `query` parameter is directly inserted into `innerHTML`.

Attack: `?q=<script>alert('XSS')</script>`

**Fix:**

```typescript
function SearchResults() {
  const params = new URLSearchParams(window.location.search);
  const query = params.get('q') || '';
  
  // Use textContent instead of innerHTML
  useEffect(() => {
    const element = document.getElementById('search-title');
    if (element) {
      element.textContent = `Results for: ${query}`;
    }
  }, [query]);
  
  // Or better, use React's safe rendering
  return <div id="search-title">Results for: {query}</div>;
}
```

---

## Question 4: Spot the vulnerability

```typescript
// API endpoint
app.get('/file', (req, res) => {
  const filename = req.query.name;
  const filepath = `/uploads/${filename}`;
  res.sendFile(filepath);
});
```

**Answer:**

**Path Traversal vulnerability**

Attack: `?name=../../../etc/passwd`

**Fix:**

```typescript
import path from 'path';

app.get('/file', (req, res) => {
  const filename = req.query.name;
  
  // Normalize and check path
  const safeName = path.basename(filename); // Remove directory components
  const filepath = path.join('/uploads', safeName);
  
  // Verify it's still within uploads directory
  if (!filepath.startsWith('/uploads/')) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  res.sendFile(filepath);
});
```

---

# 33. Security Scenarios

## Scenario 1: Implement Secure Login Form

```tsx
function SecureLoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [attempts, setAttempts] = useState(0);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Rate limiting
    if (attempts >= 5) {
      setError('Too many attempts. Please try again later.');
      return;
    }
    
    // Input validation
    if (!isValidEmail(email)) {
      setError('Please enter a valid email');
      return;
    }
    
    setIsLoading(true);
    setError('');
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        credentials: 'include',
        body: JSON.stringify({ email, password }),
      });
      
      if (!response.ok) {
        setAttempts(prev => prev + 1);
        // Generic error message
        setError('Invalid email or password');
        return;
      }
      
      // Clear sensitive data
      setPassword('');
      
      // Redirect securely
      window.location.replace('/dashboard');
    } catch {
      setError('An error occurred. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        autoComplete="email"
        required
      />
      
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        autoComplete="current-password"
        required
        minLength={12}
      />
      
      {error && <p className="error" role="alert">{error}</p>}
      
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

## Scenario 2: Implement Secure Comment System

```tsx
import DOMPurify from 'dompurify';
import { marked } from 'marked';

function CommentSection({ postId }: { postId: string }) {
  const [comments, setComments] = useState<Comment[]>([]);
  const [newComment, setNewComment] = useState('');
  
  const renderComment = (content: string): string => {
    // 1. Parse markdown
    const html = marked.parse(content);
    
    // 2. Sanitize HTML
    return DOMPurify.sanitize(html, {
      ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a', 'ul', 'ol', 'li', 'code'],
      ALLOWED_ATTR: ['href'],
    });
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate comment length
    if (newComment.length < 1 || newComment.length > 1000) {
      return;
    }
    
    const response = await fetch(`/api/posts/${postId}/comments`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ content: newComment }),
    });
    
    if (response.ok) {
      const comment = await response.json();
      setComments(prev => [...prev, comment]);
      setNewComment('');
    }
  };
  
  return (
    <div className="comments">
      {comments.map((comment) => (
        <div key={comment.id} className="comment">
          <div className="author">{comment.author}</div>
          <div 
            className="content"
            dangerouslySetInnerHTML={{ 
              __html: renderComment(comment.content) 
            }}
          />
        </div>
      ))}
      
      <form onSubmit={handleSubmit}>
        <textarea
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          maxLength={1000}
          placeholder="Write a comment (Markdown supported)"
        />
        <button type="submit">Post Comment</button>
      </form>
    </div>
  );
}
```

## Scenario 3: Implement Secure Payment Form

```tsx
function PaymentForm() {
  const [cardNumber, setCardNumber] = useState('');
  const [expiry, setExpiry] = useState('');
  const [cvv, setCvv] = useState('');
  const [isProcessing, setIsProcessing] = useState(false);
  const cardRef = useRef<string>('');
  const cvvRef = useRef<string>('');
  
  // Clear sensitive data on unmount
  useEffect(() => {
    return () => {
      cardRef.current = '';
      cvvRef.current = '';
      setCardNumber('');
      setCvv('');
    };
  }, []);
  
  const handleCardChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value.replace(/\D/g, '').slice(0, 16);
    cardRef.current = value;
    // Display masked version
    setCardNumber(formatCardNumber(value));
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsProcessing(true);
    
    try {
      // Use payment processor's SDK (e.g., Stripe)
      // Never send raw card data to your server
      const { token } = await stripe.createToken({
        number: cardRef.current,
        exp_month: expiry.split('/')[0],
        exp_year: expiry.split('/')[1],
        cvc: cvvRef.current,
      });
      
      // Send only the token to your server
      await fetch('/api/payment', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ token: token.id }),
      });
      
      // Clear sensitive data
      cardRef.current = '';
      cvvRef.current = '';
      setCardNumber('');
      setCvv('');
    } finally {
      setIsProcessing(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} autoComplete="off">
      <input
        type="text"
        inputMode="numeric"
        value={cardNumber}
        onChange={handleCardChange}
        placeholder="Card Number"
        autoComplete="cc-number"
        onCopy={(e) => e.preventDefault()}
        onPaste={(e) => e.preventDefault()}
      />
      
      <input
        type="text"
        value={expiry}
        onChange={(e) => setExpiry(formatExpiry(e.target.value))}
        placeholder="MM/YY"
        autoComplete="cc-exp"
      />
      
      <input
        type="password"
        value={cvv}
        onChange={(e) => {
          const value = e.target.value.replace(/\D/g, '').slice(0, 4);
          cvvRef.current = value;
          setCvv(value);
        }}
        placeholder="CVV"
        autoComplete="cc-csc"
        maxLength={4}
      />
      
      <button type="submit" disabled={isProcessing}>
        {isProcessing ? 'Processing...' : 'Pay Now'}
      </button>
    </form>
  );
}

function formatCardNumber(value: string): string {
  return value.replace(/(.{4})/g, '$1 ').trim();
}

function formatExpiry(value: string): string {
  const digits = value.replace(/\D/g, '').slice(0, 4);
  if (digits.length >= 2) {
    return `${digits.slice(0, 2)}/${digits.slice(2)}`;
  }
  return digits;
}
```

---

# 34. More Interview Q&A

## Q16: How do you prevent session fixation attacks?

**Answer:**

```typescript
// Session fixation: Attacker sets a known session ID before user logs in

const SESSION_FIXATION_PREVENTION = {
  regenerateSession: 'Create new session ID after login',
  strictSessionBinding: 'Bind session to user fingerprint',
  shortLifetime: 'Use short session expiry',
};

// Implementation
async function login(email: string, password: string) {
  const user = await authenticate(email, password);
  
  if (user) {
    // 1. Regenerate session ID
    await req.session.regenerate();
    
    // 2. Store user info in new session
    req.session.userId = user.id;
    req.session.createdAt = Date.now();
    
    // 3. Bind to user agent (basic fingerprint)
    req.session.userAgent = req.headers['user-agent'];
    
    return true;
  }
  
  return false;
}

// Verify session integrity
function validateSession(req: Request): boolean {
  if (!req.session.userId) return false;
  
  // Check session age
  if (Date.now() - req.session.createdAt > SESSION_MAX_AGE) {
    return false;
  }
  
  // Check fingerprint
  if (req.session.userAgent !== req.headers['user-agent']) {
    return false;
  }
  
  return true;
}
```

---

## Q17: What is HTTP Parameter Pollution?

**Answer:**

```typescript
// HTTP Parameter Pollution (HPP) 
// Sending multiple params with same name

// Attack: /search?category=books&category='; DROP TABLE--
// Some parsers use first, some use last, some use array

// Prevention
function getParameter(req: Request, name: string): string | undefined {
  const value = req.query[name];
  
  // If array, take only first value
  if (Array.isArray(value)) {
    return value[0];
  }
  
  return value;
}

// Or use explicit array handling
function getParameters(req: Request, name: string): string[] {
  const value = req.query[name];
  
  if (Array.isArray(value)) {
    return value.filter(v => typeof v === 'string') as string[];
  }
  
  return value ? [value as string] : [];
}
```

---

## Q18: How do you implement content spoofing prevention?

**Answer:**

```typescript
// Content spoofing: Attacker injects fake content

// 1. Always use Content-Type
res.setHeader('Content-Type', 'application/json');

// 2. Prevent MIME sniffing
res.setHeader('X-Content-Type-Options', 'nosniff');

// 3. Validate file uploads
async function validateUploadedFile(file: File): Promise<boolean> {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
  
  // Check declared type
  if (!allowedTypes.includes(file.type)) {
    return false;
  }
  
  // Check magic bytes
  const buffer = await file.arrayBuffer();
  const bytes = new Uint8Array(buffer.slice(0, 4));
  
  const signatures: Record<string, number[]> = {
    'image/jpeg': [0xFF, 0xD8, 0xFF],
    'image/png': [0x89, 0x50, 0x4E, 0x47],
    'image/gif': [0x47, 0x49, 0x46],
  };
  
  const expectedSig = signatures[file.type];
  return expectedSig.every((byte, i) => bytes[i] === byte);
}
```

---

## Q19: How do you handle security in SSR vs CSR?

**Answer:**

```typescript
const SSR_VS_CSR_SECURITY = {
  ssr: {
    advantages: [
      'Secrets stay on server',
      'Server-side validation',
      'No exposed API keys',
    ],
    concerns: [
      'XSS still possible',
      'SSRF attacks',
      'Server resource exhaustion',
    ],
  },
  csr: {
    advantages: [
      'Reduced SSRF risk',
      'Simpler server',
    ],
    concerns: [
      'All code exposed',
      'API security critical',
      'Token storage challenges',
    ],
  },
};

// SSR-specific security
// middleware.ts
export function middleware(req: NextRequest) {
  // Validate and sanitize input before rendering
  const unsafeParam = req.nextUrl.searchParams.get('q');
  
  if (unsafeParam && containsXSS(unsafeParam)) {
    return NextResponse.redirect('/error');
  }
  
  return NextResponse.next();
}

// CSR-specific security
// Always validate API responses
async function fetchData(url: string) {
  const response = await fetch(url);
  const data = await response.json();
  
  // Validate response schema
  const validated = DataSchema.safeParse(data);
  if (!validated.success) {
    throw new Error('Invalid response');
  }
  
  // Sanitize any HTML content
  if (validated.data.html) {
    validated.data.html = DOMPurify.sanitize(validated.data.html);
  }
  
  return validated.data;
}
```

---

## Q20: What is subdomain takeover?

**Answer:**

```typescript
// Subdomain takeover: Attacker claims abandoned subdomains

// Example:
// 1. company.com has CNAME: blog.company.com -> company.ghost.io
// 2. Company stops using Ghost, removes account
// 3. DNS record still points to company.ghost.io
// 4. Attacker creates ghost.io account for "company"
// 5. Attacker now controls blog.company.com

// Prevention:
// 1. Audit DNS records regularly
// 2. Remove CNAME before canceling service
// 3. Use DNS monitoring tools

// Detection in code
async function checkSubdomainHealth(subdomain: string) {
  try {
    const response = await fetch(`https://${subdomain}`);
    
    // Common takeover indicators
    const body = await response.text();
    const takeoverIndicators = [
      'There isn\'t a GitHub Pages site here',
      'NoSuchBucket',
      'Domain not found',
      'The request could not be satisfied',
    ];
    
    for (const indicator of takeoverIndicators) {
      if (body.includes(indicator)) {
        console.warn(`Potential subdomain takeover: ${subdomain}`);
        return false;
      }
    }
    
    return true;
  } catch (error) {
    console.warn(`Cannot reach subdomain: ${subdomain}`);
    return false;
  }
}
```

---

# 35. Final Security Checklist

## 35.1 Pre-Deployment Checklist

```typescript
const PRE_DEPLOYMENT_CHECKLIST = {
  authentication: [
    'Strong password requirements enforced',
    'Passwords hashed with bcrypt/argon2',
    'JWT secrets in environment variables',
    'Token expiration configured',
    'Session management implemented',
    'MFA option available',
  ],
  
  authorization: [
    'RBAC implemented',
    'API endpoints protected',
    'Frontend route guards in place',
    'Sensitive actions require re-auth',
  ],
  
  dataProtection: [
    'HTTPS enforced',
    'HSTS header set',
    'Sensitive data encrypted',
    'No secrets in code/logs',
    'PII properly handled',
  ],
  
  inputOutput: [
    'All input validated',
    'All output sanitized',
    'CSP header configured',
    'XSS prevention verified',
    'CSRF tokens implemented',
  ],
  
  dependencies: [
    'npm audit run',
    'No critical vulnerabilities',
    'Dependencies up to date',
    'Lock file committed',
  ],
  
  headers: [
    'Security headers configured',
    'CORS properly restricted',
    'Cookie flags set correctly',
  ],
  
  monitoring: [
    'Error logging without sensitive data',
    'Security event logging',
    'Alerting configured',
  ],
};
```

## 35.2 Complete Study Checklist

### XSS Prevention
- [ ] Understand reflected, stored, DOM-based XSS
- [ ] React auto-escaping behavior
- [ ] DOMPurify configuration
- [ ] URL sanitization
- [ ] CSP implementation

### CSRF Protection
- [ ] How CSRF works
- [ ] CSRF tokens
- [ ] SameSite cookies
- [ ] Double submit pattern

### Authentication
- [ ] Secure password handling
- [ ] JWT best practices
- [ ] Refresh token rotation
- [ ] OAuth/OIDC flows
- [ ] PKCE implementation

### Authorization
- [ ] RBAC implementation
- [ ] Protected routes
- [ ] Permission checks

### Secure Storage
- [ ] localStorage vs cookies
- [ ] HttpOnly cookies
- [ ] Secure token storage

### Headers & Policies
- [ ] All security headers
- [ ] CSP directives
- [ ] CORS configuration
- [ ] Permissions Policy

### Advanced Topics
- [ ] MFA/TOTP
- [ ] WebAuthn
- [ ] Session management
- [ ] Rate limiting

---

# 36. Security Incident Response

## 36.1 Detecting Security Incidents

```typescript
// Client-side anomaly detection
class SecurityMonitor {
  private eventLog: SecurityEvent[] = [];
  private readonly ALERT_THRESHOLD = 5;
  
  logEvent(event: SecurityEvent) {
    this.eventLog.push({
      ...event,
      timestamp: new Date(),
      userAgent: navigator.userAgent,
    });
    
    this.analyzePatterns();
  }
  
  private analyzePatterns() {
    const recentEvents = this.eventLog.filter(
      e => Date.now() - e.timestamp.getTime() < 60000
    );
    
    // Check for suspicious patterns
    const failedLogins = recentEvents.filter(e => e.type === 'FAILED_LOGIN');
    if (failedLogins.length >= this.ALERT_THRESHOLD) {
      this.reportIncident({
        type: 'BRUTE_FORCE_ATTEMPT',
        severity: 'HIGH',
        details: { attemptCount: failedLogins.length },
      });
    }
    
    const csrfViolations = recentEvents.filter(e => e.type === 'CSRF_VIOLATION');
    if (csrfViolations.length > 0) {
      this.reportIncident({
        type: 'CSRF_ATTACK',
        severity: 'CRITICAL',
        details: csrfViolations,
      });
    }
    
    const xssAttempts = recentEvents.filter(e => e.type === 'XSS_ATTEMPT');
    if (xssAttempts.length > 0) {
      this.reportIncident({
        type: 'XSS_ATTACK',
        severity: 'CRITICAL',
        details: xssAttempts,
      });
    }
  }
  
  private async reportIncident(incident: Incident) {
    // Send to security team
    await fetch('/api/security/incidents', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(incident),
    });
    
    // Log locally for debugging
    console.warn('[Security Incident]', incident);
  }
}

// Usage
const securityMonitor = new SecurityMonitor();

// In login handler
async function handleLogin(email: string, password: string) {
  try {
    await login(email, password);
    securityMonitor.logEvent({ type: 'LOGIN_SUCCESS', email });
  } catch (error) {
    securityMonitor.logEvent({ type: 'FAILED_LOGIN', email });
    throw error;
  }
}
```

## 36.2 CSP Violation Reporting

```typescript
// Set up CSP reporting
const cspWithReporting = `
  default-src 'self';
  script-src 'self';
  report-uri /api/csp-report;
  report-to csp-endpoint;
`;

// Report-To header for modern browsers
const reportToHeader = JSON.stringify({
  group: 'csp-endpoint',
  max_age: 10886400,
  endpoints: [{ url: '/api/csp-report' }],
});

// API endpoint to handle reports
// pages/api/csp-report.ts
export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }
  
  const report = req.body;
  
  // Log the violation
  console.warn('CSP Violation:', {
    blockedUri: report['csp-report']?.['blocked-uri'],
    violatedDirective: report['csp-report']?.['violated-directive'],
    documentUri: report['csp-report']?.['document-uri'],
  });
  
  // Store in database for analysis
  // alertSecurityTeam(report);
  
  res.status(204).end();
}
```

## 36.3 Security Event Logging

```typescript
interface SecurityLog {
  timestamp: Date;
  eventType: string;
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  userId?: string;
  ip?: string;
  details: Record<string, any>;
}

class SecurityLogger {
  private logs: SecurityLog[] = [];
  private readonly MAX_LOGS = 1000;
  
  log(log: Omit<SecurityLog, 'timestamp'>) {
    const entry: SecurityLog = {
      ...log,
      timestamp: new Date(),
      // Sanitize sensitive data
      details: this.sanitize(log.details),
    };
    
    this.logs.push(entry);
    
    if (this.logs.length > this.MAX_LOGS) {
      this.flush();
    }
    
    if (log.severity === 'CRITICAL') {
      this.alertImmediately(entry);
    }
  }
  
  private sanitize(details: Record<string, any>): Record<string, any> {
    const sensitiveKeys = ['password', 'token', 'secret', 'creditCard'];
    const sanitized = { ...details };
    
    for (const key of Object.keys(sanitized)) {
      if (sensitiveKeys.some(sk => key.toLowerCase().includes(sk))) {
        sanitized[key] = '[REDACTED]';
      }
    }
    
    return sanitized;
  }
  
  private async flush() {
    const logsToSend = [...this.logs];
    this.logs = [];
    
    await fetch('/api/security/logs', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(logsToSend),
    });
  }
  
  private async alertImmediately(log: SecurityLog) {
    await fetch('/api/security/alert', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(log),
    });
  }
}

// Usage
const securityLogger = new SecurityLogger();

// Log various security events
securityLogger.log({
  eventType: 'AUTH_FAILURE',
  severity: 'MEDIUM',
  details: { reason: 'Invalid password', attempts: 3 },
});

securityLogger.log({
  eventType: 'PRIVILEGE_ESCALATION_ATTEMPT',
  severity: 'CRITICAL',
  userId: 'user123',
  details: { attemptedRole: 'admin' },
});
```

---

# 37. Penetration Testing Basics

## 37.1 Security Testing Approach

```typescript
const SECURITY_TESTING_PHASES = {
  reconnaissance: {
    activities: [
      'Identify endpoints',
      'Map application structure',
      'Identify technologies used',
    ],
    tools: ['Browser DevTools', 'Burp Suite', 'OWASP ZAP'],
  },
  
  scanning: {
    activities: [
      'Vulnerability scanning',
      'Port scanning (if applicable)',
      'Dependency checking',
    ],
    tools: ['npm audit', 'Snyk', 'OWASP Dependency-Check'],
  },
  
  exploitation: {
    activities: [
      'Test for XSS',
      'Test for CSRF',
      'Test auth bypass',
      'Test access control',
    ],
    approach: 'Attempt to exploit identified vulnerabilities',
  },
  
  reporting: {
    contents: [
      'Vulnerability description',
      'Severity rating',
      'Steps to reproduce',
      'Remediation recommendations',
      'Proof of concept',
    ],
  },
};
```

## 37.2 Common Tests

```typescript
// XSS Test Cases
const XSS_TEST_PAYLOADS = [
  '<script>alert("XSS")</script>',
  '<img src=x onerror=alert("XSS")>',
  '<svg onload=alert("XSS")>',
  'javascript:alert("XSS")',
  '<iframe src="javascript:alert(\'XSS\')">',
  '"><script>alert("XSS")</script>',
  "'-alert('XSS')-'",
  '<a href="javascript:alert(\'XSS\')">click</a>',
];

// CSRF Test
function testCSRF(targetUrl: string, formData: FormData) {
  // Create fake form on different origin
  // Submit to target without CSRF token
  // If successful, CSRF vulnerability exists
}

// Auth Bypass Tests
const AUTH_BYPASS_TESTS = [
  'Access protected routes without token',
  'Use expired token',
  'Use token from different user',
  'Modify JWT payload without re-signing',
  'Use none algorithm in JWT',
];

// Access Control Tests
const ACCESS_CONTROL_TESTS = [
  'Access other user\'s resources by ID',
  'Modify role in request',
  'Access admin endpoints as regular user',
  'Enumerate user IDs',
];
```

## 37.3 Automated Security Testing

```typescript
// Example: Playwright security tests
import { test, expect } from '@playwright/test';

test.describe('Security Tests', () => {
  test('should not reflect XSS in search', async ({ page }) => {
    const xssPayload = '<script>alert("xss")</script>';
    
    await page.goto(`/search?q=${encodeURIComponent(xssPayload)}`);
    
    // Check that script isn't executed
    const content = await page.content();
    expect(content).not.toContain('<script>alert("xss")</script>');
    
    // Check for proper escaping
    expect(content).toContain('&lt;script&gt;');
  });
  
  test('should require authentication for protected routes', async ({ page }) => {
    // Don't login, try to access protected page
    const response = await page.goto('/dashboard');
    
    // Should redirect to login
    expect(page.url()).toContain('/login');
  });
  
  test('should not allow IDOR', async ({ page, context }) => {
    // Login as user1
    await loginAsUser(page, 'user1@example.com');
    
    // Try to access user2's profile
    const response = await page.goto('/api/users/user2-id/profile');
    
    // Should return 403
    expect(response?.status()).toBe(403);
  });
  
  test('should have security headers', async ({ page }) => {
    const response = await page.goto('/');
    const headers = response?.headers();
    
    expect(headers?.['x-frame-options']).toBe('DENY');
    expect(headers?.['x-content-type-options']).toBe('nosniff');
    expect(headers?.['strict-transport-security']).toBeTruthy();
    expect(headers?.['content-security-policy']).toBeTruthy();
  });
});
```

---

# 38. Common Security Mistakes

## 38.1 Client-Side Security Anti-Patterns

```typescript
// ❌ Mistake 1: Client-side only validation
function BuyButton({ price }: { price: number }) {
  const [discount, setDiscount] = useState(0);
  
  // Client can manipulate this!
  const finalPrice = price - discount;
  
  const handleBuy = () => {
    fetch('/api/purchase', {
      method: 'POST',
      body: JSON.stringify({ price: finalPrice }), // Attacker controls this!
    });
  };
  
  return <button onClick={handleBuy}>Buy for ${finalPrice}</button>;
}

// ✅ Fix: Send product ID, calculate price server-side
function BuyButton({ productId }: { productId: string }) {
  const handleBuy = () => {
    fetch('/api/purchase', {
      method: 'POST',
      body: JSON.stringify({ productId }), // Server looks up price
    });
  };
  
  return <button onClick={handleBuy}>Buy</button>;
}

// ❌ Mistake 2: Trusting user input for display
function UserComment({ comment }: { comment: string }) {
  return <div dangerouslySetInnerHTML={{ __html: comment }} />;
}

// ✅ Fix: Sanitize before display
function UserComment({ comment }: { comment: string }) {
  return (
    <div 
      dangerouslySetInnerHTML={{ 
        __html: DOMPurify.sanitize(comment) 
      }} 
    />
  );
}

// ❌ Mistake 3: Storing secrets in code
const API_KEY = 'sk_live_abc123'; // Exposed in bundle!

// ✅ Fix: Use environment variables (server-only)
const API_KEY = process.env.API_KEY; // Server-side only

// ❌ Mistake 4: Insecure token storage
localStorage.setItem('token', jwt); // XSS vulnerable

// ✅ Fix: Use HttpOnly cookies (set by server)
// Server: Set-Cookie: token=jwt; HttpOnly; Secure; SameSite=Strict

// ❌ Mistake 5: CORS wildcard
app.use(cors({ origin: '*' })); // Allows any origin

// ✅ Fix: Specific origins
app.use(cors({ 
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  credentials: true,
}));
```

## 38.2 API Security Mistakes

```typescript
// ❌ Mistake 1: No rate limiting
app.post('/login', (req, res) => {
  // No protection against brute force!
  return authenticate(req.body);
});

// ✅ Fix: Add rate limiting
import rateLimit from 'express-rate-limit';

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts',
});

app.post('/login', loginLimiter, (req, res) => {
  return authenticate(req.body);
});

// ❌ Mistake 2: Verbose error messages
app.get('/users/:id', async (req, res) => {
  try {
    const user = await db.findUser(req.params.id);
    res.json(user);
  } catch (error) {
    res.status(500).json({ 
      error: error.message,
      stack: error.stack, // Exposes internal details!
    });
  }
});

// ✅ Fix: Generic error messages
app.get('/users/:id', async (req, res) => {
  try {
    const user = await db.findUser(req.params.id);
    res.json(user);
  } catch (error) {
    console.error('Error fetching user:', error); // Log internally
    res.status(500).json({ error: 'An error occurred' });
  }
});

// ❌ Mistake 3: Missing input validation
app.post('/transfer', (req, res) => {
  const { toAccount, amount } = req.body;
  // What if amount is negative? What if toAccount is invalid?
  return transfer(toAccount, amount);
});

// ✅ Fix: Validate all input
import { z } from 'zod';

const TransferSchema = z.object({
  toAccount: z.string().regex(/^\d{10}$/),
  amount: z.number().positive().max(100000),
});

app.post('/transfer', (req, res) => {
  const result = TransferSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: 'Invalid input' });
  }
  return transfer(result.data.toAccount, result.data.amount);
});
```

---

# 39. Security Resources

## 39.1 Learning Resources

```typescript
const SECURITY_RESOURCES = {
  documentation: [
    { name: 'OWASP Top 10', url: 'https://owasp.org/www-project-top-ten/' },
    { name: 'OWASP Cheat Sheets', url: 'https://cheatsheetseries.owasp.org/' },
    { name: 'MDN Web Security', url: 'https://developer.mozilla.org/en-US/docs/Web/Security' },
    { name: 'Google Web Fundamentals - Security', url: 'https://developers.google.com/web/fundamentals/security' },
  ],
  
  tools: [
    { name: 'OWASP ZAP', purpose: 'Web app security scanner' },
    { name: 'Burp Suite', purpose: 'Web security testing' },
    { name: 'npm audit', purpose: 'Dependency vulnerability check' },
    { name: 'Snyk', purpose: 'Security monitoring' },
    { name: 'SonarQube', purpose: 'Code quality and security' },
  ],
  
  practiceEnvironments: [
    { name: 'OWASP WebGoat', purpose: 'Learn web app security' },
    { name: 'HackTheBox', purpose: 'Security challenges' },
    { name: 'PortSwigger Web Security Academy', purpose: 'Free web security training' },
    { name: 'DVWA', purpose: 'Damn Vulnerable Web Application' },
  ],
  
  certifications: [
    'OSCP (Offensive Security Certified Professional)',
    'CEH (Certified Ethical Hacker)',
    'CompTIA Security+',
    'GWAPT (GIAC Web Application Penetration Tester)',
  ],
};
```

## 39.2 Security Libraries

```typescript
const SECURITY_LIBRARIES = {
  sanitization: {
    'dompurify': 'XSS sanitization for HTML',
    'isomorphic-dompurify': 'SSR-compatible DOMPurify',
    'sanitize-html': 'HTML sanitization',
  },
  
  validation: {
    'zod': 'Schema validation',
    'joi': 'Object schema validation',
    'validator': 'String validation',
  },
  
  authentication: {
    'next-auth': 'Next.js authentication',
    'passport': 'Authentication middleware',
    '@auth0/auth0-react': 'Auth0 integration',
  },
  
  encryption: {
    'bcrypt': 'Password hashing',
    'argon2': 'Password hashing (modern)',
    'crypto-js': 'Client-side crypto',
  },
  
  session: {
    'express-session': 'Session management',
    'iron-session': 'Encrypted cookie sessions',
  },
  
  csrf: {
    'csurf': 'CSRF protection middleware',
    'csrf': 'CSRF token generation',
  },
  
  rateLimiting: {
    'express-rate-limit': 'Rate limiting',
    'rate-limiter-flexible': 'Flexible rate limiting',
  },
};
```

---

# 40. Quick Reference Cards

## 40.1 XSS Prevention Quick Reference

| Context | Safe Method | Example |
|---------|-------------|---------|
| HTML text content | React auto-escape | `<div>{userInput}</div>` |
| HTML attributes | React auto-escape | `<input value={userInput} />` |
| JavaScript strings | JSON.stringify | `const data = ${JSON.stringify(input)}` |
| CSS values | Validate/allowlist | `style={{ color: validateColor(input) }}` |
| URLs | Validate protocol | Check for `http:`/`https:` only |
| HTML content | DOMPurify | `DOMPurify.sanitize(html)` |

## 40.2 Security Headers Quick Reference

| Header | Value | Purpose |
|--------|-------|---------|
| CSP | `default-src 'self'` | Prevent XSS |
| X-Frame-Options | `DENY` | Prevent clickjacking |
| X-Content-Type-Options | `nosniff` | Prevent MIME sniffing |
| HSTS | `max-age=31536000` | Force HTTPS |
| Referrer-Policy | `strict-origin-when-cross-origin` | Limit referrer |
| Permissions-Policy | `camera=()` | Disable features |

## 40.3 Auth Token Quick Reference

| Aspect | Access Token | Refresh Token |
|--------|--------------|---------------|
| Lifetime | Short (15 min) | Long (7 days) |
| Storage | Memory or HttpOnly cookie | HttpOnly cookie only |
| Sent to | Any API endpoint | Refresh endpoint only |
| Contains | User claims, permissions | User ID only |
| Revocable | No (until expiry) | Yes (in database) |

## 40.4 Cookie Flags Quick Reference

| Flag | Purpose | When to Use |
|------|---------|-------------|
| `HttpOnly` | No JavaScript access | Auth tokens |
| `Secure` | HTTPS only | Production |
| `SameSite=Strict` | No cross-site sending | Most cookies |
| `SameSite=Lax` | Safe cross-site navigation | Less sensitive |
| `Path=/api` | Limit to path | API-only tokens |

---

# 41. Final Interview Tips

## 41.1 Common Security Interview Questions

```typescript
const COMMON_QUESTIONS = [
  // Conceptual
  'What is XSS and how do you prevent it?',
  'Explain CSRF and its countermeasures',
  'Where should you store JWT tokens?',
  'What is the same-origin policy?',
  'How does CORS work?',
  
  // Practical
  'How would you implement secure authentication?',
  'Design a password reset flow',
  'How do you protect against brute force attacks?',
  'Implement rate limiting for an API',
  'What security headers would you configure?',
  
  // Scenario-based
  'A user reports seeing someone else\'s data - what do you check?',
  'How would you respond to a reported XSS vulnerability?',
  'Your site is being scraped - what measures would you take?',
  'How do you secure a file upload feature?',
];
```

## 41.2 Key Points to Remember

```typescript
const KEY_SECURITY_POINTS = [
  // Input/Output
  'Never trust user input',
  'Always validate server-side',
  'Sanitize all output',
  'Use parameterized queries',
  
  // Authentication
  'Use HttpOnly cookies for tokens',
  'Implement proper session management',
  'Hash passwords with bcrypt/argon2',
  'Use short-lived access tokens',
  
  // Defense in Depth
  'Multiple layers of security',
  'CSP + sanitization + validation',
  'Server-side is the source of truth',
  
  // Best Practices
  'Keep dependencies updated',
  'Use security headers',
  'Log security events',
  'Regular security audits',
];
```

## 41.3 What Makes a Good Security Answer

```typescript
const GOOD_ANSWER_STRUCTURE = {
  understand: 'Show you understand the vulnerability',
  explain: 'Explain how attacks work',
  prevent: 'Describe prevention methods',
  implement: 'Show code examples',
  tradeoffs: 'Discuss tradeoffs and limitations',
  defense: 'Mention defense in depth',
};

// Example: "What is XSS?"
const EXAMPLE_ANSWER = `
1. UNDERSTAND: XSS lets attackers inject scripts into your pages

2. EXPLAIN: Three types - reflected, stored, DOM-based.
   Reflected: malicious script in URL
   Stored: script saved in database
   DOM-based: script manipulates DOM

3. PREVENT:
   - Use React's auto-escaping
   - DOMPurify for user HTML
   - Validate URLs
   - CSP headers

4. IMPLEMENT: Show code for DOMPurify, CSP config

5. TRADEOFFS: CSP can break inline scripts, 
   sanitization might strip legitimate content

6. DEFENSE IN DEPTH: Combine all methods -
   sanitization + CSP + HttpOnly cookies
`;
```

---

## 🎯 Complete Security Study Checklist

### Core Vulnerabilities
- [x] XSS (Reflected, Stored, DOM-based)
- [x] CSRF
- [x] SQL Injection concepts
- [x] Clickjacking
- [x] Path traversal

### Authentication
- [x] Password security
- [x] JWT best practices
- [x] OAuth 2.0 / OIDC
- [x] PKCE
- [x] Session management
- [x] MFA / TOTP
- [x] WebAuthn

### Authorization
- [x] RBAC
- [x] Protected routes
- [x] Access control

### Security Headers
- [x] CSP
- [x] CORS
- [x] HSTS
- [x] All standard headers

### Secure Storage
- [x] Token storage options
- [x] Cookie security
- [x] Sensitive data handling

### Testing & Monitoring
- [x] Security testing
- [x] Incident detection
- [x] Logging best practices

---

# 42. Bonus Coding Challenges

## Challenge 7: Implement Secure Form State

```typescript
import { useState, useCallback, useRef, useEffect } from 'react';

interface SecureFormOptions {
  sensitiveFields: string[];
  maxIdleTime?: number;
}

function useSecureForm<T extends Record<string, any>>(
  initialState: T,
  options: SecureFormOptions
) {
  const [values, setValues] = useState<T>(initialState);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const sensitiveRefs = useRef<Record<string, string>>({});
  const lastFormActivity = useRef(Date.now());
  
  // Clear sensitive fields when idle
  useEffect(() => {
    const checkIdle = () => {
      const idleTime = Date.now() - lastFormActivity.current;
      const maxIdle = options.maxIdleTime || 5 * 60 * 1000;
      
      if (idleTime > maxIdle) {
        clearSensitiveData();
      }
    };
    
    const interval = setInterval(checkIdle, 30000);
    return () => clearInterval(interval);
  }, []);
  
  const handleChange = useCallback((field: keyof T, value: string) => {
    lastFormActivity.current = Date.now();
    
    if (options.sensitiveFields.includes(field as string)) {
      sensitiveRefs.current[field as string] = value;
      setValues(prev => ({
        ...prev,
        [field]: '•'.repeat(value.length), // Display masked
      }));
    } else {
      setValues(prev => ({ ...prev, [field]: value }));
    }
  }, [options.sensitiveFields]);
  
  const getSensitiveValue = useCallback((field: keyof T): string => {
    return sensitiveRefs.current[field as string] || '';
  }, []);
  
  const clearSensitiveData = useCallback(() => {
    sensitiveRefs.current = {};
    const clearedValues = { ...values };
    for (const field of options.sensitiveFields) {
      (clearedValues as any)[field] = '';
    }
    setValues(clearedValues);
  }, [values, options.sensitiveFields]);
  
  const getSubmitData = useCallback((): T => {
    const data = { ...values };
    for (const field of options.sensitiveFields) {
      (data as any)[field] = sensitiveRefs.current[field] || '';
    }
    return data;
  }, [values, options.sensitiveFields]);
  
  return {
    values,
    errors,
    setErrors,
    handleChange,
    getSensitiveValue,
    clearSensitiveData,
    getSubmitData,
  };
}

// Usage
function LoginForm() {
  const form = useSecureForm(
    { email: '', password: '' },
    { sensitiveFields: ['password'], maxIdleTime: 3 * 60 * 1000 }
  );
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const data = form.getSubmitData();
    
    try {
      await login(data.email, data.password);
    } finally {
      form.clearSensitiveData();
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={form.values.email}
        onChange={(e) => form.handleChange('email', e.target.value)}
      />
      <input
        type="password"
        value={form.values.password}
        onChange={(e) => form.handleChange('password', e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

## Challenge 8: Implement Request Fingerprinting

```typescript
class RequestFingerprint {
  private fingerprint: string | null = null;
  
  async generate(): Promise<string> {
    if (this.fingerprint) return this.fingerprint;
    
    const components = [
      this.getCanvasFingerprint(),
      this.getScreenInfo(),
      this.getTimezoneInfo(),
      this.getLanguageInfo(),
      navigator.userAgent,
      navigator.hardwareConcurrency,
      navigator.deviceMemory,
    ];
    
    const combined = components.join('|');
    this.fingerprint = await this.hash(combined);
    return this.fingerprint;
  }
  
  private getCanvasFingerprint(): string {
    try {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      if (!ctx) return 'no-canvas';
      
      ctx.textBaseline = 'top';
      ctx.font = '14px Arial';
      ctx.fillText('fingerprint', 0, 0);
      
      return canvas.toDataURL().slice(-100);
    } catch {
      return 'canvas-error';
    }
  }
  
  private getScreenInfo(): string {
    return `${screen.width}x${screen.height}x${screen.colorDepth}`;
  }
  
  private getTimezoneInfo(): string {
    return Intl.DateTimeFormat().resolvedOptions().timeZone;
  }
  
  private getLanguageInfo(): string {
    return navigator.languages.join(',');
  }
  
  private async hash(input: string): Promise<string> {
    const encoder = new TextEncoder();
    const data = encoder.encode(input);
    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  }
}

// Add fingerprint to requests
const fingerprinter = new RequestFingerprint();

async function secureRequest(url: string, options: RequestInit = {}) {
  const fingerprint = await fingerprinter.generate();
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-Request-Fingerprint': fingerprint,
    },
  });
}
```

## Challenge 9: Implement Secure Local Storage Wrapper

```typescript
class SecureStorage {
  private key: CryptoKey | null = null;
  private readonly ENCRYPTION_KEY = 'secure_storage_key';
  
  async init(password: string): Promise<void> {
    const encoder = new TextEncoder();
    const keyMaterial = await crypto.subtle.importKey(
      'raw',
      encoder.encode(password),
      'PBKDF2',
      false,
      ['deriveKey']
    );
    
    const salt = this.getSalt();
    
    this.key = await crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt,
        iterations: 100000,
        hash: 'SHA-256',
      },
      keyMaterial,
      { name: 'AES-GCM', length: 256 },
      false,
      ['encrypt', 'decrypt']
    );
  }
  
  private getSalt(): Uint8Array {
    const stored = localStorage.getItem(this.ENCRYPTION_KEY);
    if (stored) {
      return new Uint8Array(JSON.parse(stored));
    }
    
    const salt = crypto.getRandomValues(new Uint8Array(16));
    localStorage.setItem(this.ENCRYPTION_KEY, JSON.stringify(Array.from(salt)));
    return salt;
  }
  
  async setItem(key: string, value: any): Promise<void> {
    if (!this.key) throw new Error('Storage not initialized');
    
    const encoder = new TextEncoder();
    const data = encoder.encode(JSON.stringify(value));
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      this.key,
      data
    );
    
    const combined = new Uint8Array(iv.length + encrypted.byteLength);
    combined.set(iv);
    combined.set(new Uint8Array(encrypted), iv.length);
    
    localStorage.setItem(key, btoa(String.fromCharCode(...combined)));
  }
  
  async getItem<T>(key: string): Promise<T | null> {
    if (!this.key) throw new Error('Storage not initialized');
    
    const stored = localStorage.getItem(key);
    if (!stored) return null;
    
    try {
      const combined = new Uint8Array(
        atob(stored).split('').map(c => c.charCodeAt(0))
      );
      
      const iv = combined.slice(0, 12);
      const data = combined.slice(12);
      
      const decrypted = await crypto.subtle.decrypt(
        { name: 'AES-GCM', iv },
        this.key,
        data
      );
      
      const decoder = new TextDecoder();
      return JSON.parse(decoder.decode(decrypted));
    } catch {
      return null;
    }
  }
  
  removeItem(key: string): void {
    localStorage.removeItem(key);
  }
  
  clear(): void {
    localStorage.clear();
    this.key = null;
  }
}

// Usage
const secureStorage = new SecureStorage();

async function saveSecureData() {
  await secureStorage.init('user-password-or-derived-key');
  
  await secureStorage.setItem('sensitiveData', {
    apiKey: 'abc123',
    lastLogin: new Date().toISOString(),
  });
  
  const data = await secureStorage.getItem('sensitiveData');
  console.log(data);
}
```

## Challenge 10: Implement Anti-Tampering Detection

```typescript
class IntegrityChecker {
  private checksums = new Map<string, string>();
  
  async registerComponent(id: string, element: Element): Promise<void> {
    const content = element.outerHTML;
    const checksum = await this.hash(content);
    this.checksums.set(id, checksum);
  }
  
  async verify(id: string, element: Element): Promise<boolean> {
    const expected = this.checksums.get(id);
    if (!expected) return true; // Not registered
    
    const current = await this.hash(element.outerHTML);
    return current === expected;
  }
  
  startMonitoring(options?: {
    interval?: number;
    onTamper?: (id: string) => void;
  }): () => void {
    const { interval = 5000, onTamper } = options || {};
    
    const checkAll = async () => {
      for (const [id, _] of this.checksums) {
        const element = document.getElementById(id);
        if (!element) continue;
        
        const isValid = await this.verify(id, element);
        if (!isValid) {
          console.error(`Tampering detected: ${id}`);
          onTamper?.(id);
        }
      }
    };
    
    const intervalId = setInterval(checkAll, interval);
    return () => clearInterval(intervalId);
  }
  
  private async hash(content: string): Promise<string> {
    const encoder = new TextEncoder();
    const data = encoder.encode(content);
    const buffer = await crypto.subtle.digest('SHA-256', data);
    return Array.from(new Uint8Array(buffer))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
  }
}

// Usage
const integrity = new IntegrityChecker();

// Register critical elements
document.querySelectorAll('[data-protected]').forEach((el, i) => {
  integrity.registerComponent(`protected-${i}`, el);
});

// Start monitoring
const stopMonitoring = integrity.startMonitoring({
  interval: 10000,
  onTamper: (id) => {
    // Reload page or alert user
    window.location.reload();
  },
});
```

---

# 43. Mock Interview Scenarios

## Scenario 1: Security Incident Response

**Q: You receive an alert that user session tokens are being stolen. Walk me through your response.**

**Answer:**

```typescript
const INCIDENT_RESPONSE_PLAN = {
  immediate: [
    'Rotate all session secrets',
    'Invalidate all existing sessions',
    'Force re-authentication for all users',
  ],
  
  investigation: [
    'Check access logs for unusual patterns',
    'Review recent code changes',
    'Check for XSS vulnerabilities',
    'Verify CSP is properly configured',
    'Check for compromised dependencies',
  ],
  
  mitigation: [
    'Implement HttpOnly cookies if not used',
    'Add SameSite=Strict to session cookies',
    'Strengthen CSP headers',
    'Add session binding to IP/User-Agent',
  ],
  
  communication: [
    'Notify affected users',
    'Document incident timeline',
    'Prepare incident report',
    'Review and update security policies',
  ],
};

// Force logout all users
async function emergencyLogout() {
  // Invalidate all tokens
  await invalidateAllTokens();
  
  // Update session secret
  await rotateSessionSecret();
  
  // Clear all sessions
  await clearAllSessions();
  
  // Log the incident
  await logSecurityIncident({
    type: 'TOKEN_THEFT_RESPONSE',
    action: 'FORCE_LOGOUT_ALL',
    timestamp: new Date(),
  });
}
```

---

## Scenario 2: Security Review

**Q: Review this authentication code and identify issues.**

```typescript
// Original code
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  
  if (!user || user.password !== password) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const token = jwt.sign({ userId: user.id, role: user.role }, 'secret');
  res.json({ token, user });
});
```

**Answer:**

```typescript
// Issues identified and fixed:

// 1. Plain text password comparison - should use bcrypt
// 2. JWT secret in code - should use env variable
// 3. No token expiration - should have short lifetime
// 4. Returning full user object - may include sensitive data
// 5. No input validation - should validate email format
// 6. No rate limiting - vulnerable to brute force

// Fixed version:
import bcrypt from 'bcrypt';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
});

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
});

app.post('/login', loginLimiter, async (req, res) => {
  // Validate input
  const result = loginSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const { email, password } = result.data;
  const user = await User.findOne({ email });
  
  // Use timing-safe comparison
  const isValid = user && await bcrypt.compare(password, user.passwordHash);
  
  if (!isValid) {
    await logFailedLogin(email);
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Short-lived token with env secret
  const token = jwt.sign(
    { userId: user.id },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  
  // Set as HttpOnly cookie
  res.cookie('accessToken', token, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 15 * 60 * 1000,
  });
  
  // Return only safe user data
  res.json({
    user: { id: user.id, email: user.email, name: user.name },
  });
});
```

---

## Scenario 3: Designing Secure Feature

**Q: Design a secure password reset flow.**

**Answer:**

```typescript
// Step 1: Request password reset
async function requestPasswordReset(email: string) {
  // Always return same message to prevent enumeration
  const response = { message: 'If this email exists, you will receive a reset link.' };
  
  const user = await User.findOne({ email });
  if (!user) return response;
  
  // Generate secure token
  const resetToken = crypto.randomBytes(32).toString('hex');
  const tokenHash = await bcrypt.hash(resetToken, 10);
  
  // Store hashed token with expiration
  await PasswordReset.create({
    userId: user.id,
    tokenHash,
    expiresAt: new Date(Date.now() + 15 * 60 * 1000), // 15 minutes
  });
  
  // Send email with token
  await sendEmail({
    to: email,
    subject: 'Password Reset',
    html: `
      <p>Click to reset your password:</p>
      <a href="https://app.com/reset?token=${resetToken}">Reset Password</a>
      <p>This link expires in 15 minutes.</p>
    `,
  });
  
  return response;
}

// Step 2: Validate token and reset password
async function resetPassword(token: string, newPassword: string) {
  // Find valid reset request
  const resets = await PasswordReset.findAll({
    where: {
      expiresAt: { [Op.gt]: new Date() },
    },
  });
  
  // Check each hash (token is plaintext, stored as hash)
  let validReset = null;
  for (const reset of resets) {
    if (await bcrypt.compare(token, reset.tokenHash)) {
      validReset = reset;
      break;
    }
  }
  
  if (!validReset) {
    throw new Error('Invalid or expired reset link');
  }
  
  // Validate new password
  const passwordCheck = validatePassword(newPassword);
  if (!passwordCheck.valid) {
    throw new Error(passwordCheck.errors.join(', '));
  }
  
  // Update password
  const hashedPassword = await bcrypt.hash(newPassword, 12);
  await User.update(
    { passwordHash: hashedPassword },
    { where: { id: validReset.userId } }
  );
  
  // Invalidate all sessions
  await Session.destroy({ where: { userId: validReset.userId } });
  
  // Delete reset token
  await validReset.destroy();
  
  // Notify user
  const user = await User.findByPk(validReset.userId);
  await sendEmail({
    to: user.email,
    subject: 'Password Changed',
    html: `
      <p>Your password has been changed.</p>
      <p>If you didn't make this change, contact support immediately.</p>
    `,
  });
}
```

---

# 44. Advanced Security Patterns

## 44.1 Secure Session Binding

```typescript
class SecureSession {
  private sessionId: string;
  private fingerprint: string;
  private createdAt: number;
  
  constructor(userId: string) {
    this.sessionId = crypto.randomUUID();
    this.fingerprint = this.generateFingerprint();
    this.createdAt = Date.now();
  }
  
  private generateFingerprint(): string {
    const components = [
      navigator.userAgent,
      screen.width,
      screen.height,
      screen.colorDepth,
      new Date().getTimezoneOffset(),
      navigator.language,
    ];
    
    return components.join('|');
  }
  
  isValid(): boolean {
    // Check session age
    const maxAge = 30 * 60 * 1000; // 30 minutes
    if (Date.now() - this.createdAt > maxAge) {
      return false;
    }
    
    // Check fingerprint hasn't changed
    const currentFingerprint = this.generateFingerprint();
    if (currentFingerprint !== this.fingerprint) {
      console.warn('Session fingerprint mismatch - possible hijacking');
      return false;
    }
    
    return true;
  }
  
  refresh(): void {
    this.createdAt = Date.now();
  }
}
```

## 44.2 Secure API Key Management

```typescript
class APIKeyManager {
  private encryptedKeys: Map<string, string> = new Map();
  private encryptionKey: CryptoKey | null = null;
  
  async init(): Promise<void> {
    this.encryptionKey = await crypto.subtle.generateKey(
      { name: 'AES-GCM', length: 256 },
      true,
      ['encrypt', 'decrypt']
    );
  }
  
  async storeKey(name: string, apiKey: string): Promise<void> {
    if (!this.encryptionKey) throw new Error('Not initialized');
    
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encoder = new TextEncoder();
    
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      this.encryptionKey,
      encoder.encode(apiKey)
    );
    
    const combined = new Uint8Array(iv.length + encrypted.byteLength);
    combined.set(iv);
    combined.set(new Uint8Array(encrypted), iv.length);
    
    this.encryptedKeys.set(name, btoa(String.fromCharCode(...combined)));
  }
  
  async getKey(name: string): Promise<string | null> {
    if (!this.encryptionKey) throw new Error('Not initialized');
    
    const encrypted = this.encryptedKeys.get(name);
    if (!encrypted) return null;
    
    const combined = new Uint8Array(
      atob(encrypted).split('').map(c => c.charCodeAt(0))
    );
    
    const iv = combined.slice(0, 12);
    const data = combined.slice(12);
    
    const decrypted = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv },
      this.encryptionKey,
      data
    );
    
    return new TextDecoder().decode(decrypted);
  }
  
  revokeKey(name: string): void {
    this.encryptedKeys.delete(name);
  }
  
  revokeAll(): void {
    this.encryptedKeys.clear();
    this.encryptionKey = null;
  }
}
```

## 44.3 Secure Configuration Loading

```typescript
interface SecureConfig {
  apiUrl: string;
  features: string[];
  limits: {
    maxUploadSize: number;
    rateLimit: number;
  };
}

class SecureConfigLoader {
  private config: SecureConfig | null = null;
  private signature: string | null = null;
  
  async loadConfig(url: string): Promise<SecureConfig> {
    const response = await fetch(url);
    
    // Verify signature
    const signature = response.headers.get('X-Config-Signature');
    if (!signature || !await this.verifySignature(response, signature)) {
      throw new Error('Config signature verification failed');
    }
    
    const config = await response.json();
    
    // Validate structure
    if (!this.isValidConfig(config)) {
      throw new Error('Invalid config structure');
    }
    
    this.config = config;
    this.signature = signature;
    
    return config;
  }
  
  private async verifySignature(response: Response, signature: string): Promise<boolean> {
    const body = await response.clone().text();
    const publicKey = await this.getPublicKey();
    
    const encoder = new TextEncoder();
    const signatureBytes = Uint8Array.from(
      atob(signature), c => c.charCodeAt(0)
    );
    
    return crypto.subtle.verify(
      { name: 'RSASSA-PKCS1-v1_5' },
      publicKey,
      signatureBytes,
      encoder.encode(body)
    );
  }
  
  private async getPublicKey(): Promise<CryptoKey> {
    // Load from embedded or fetched source
    const publicKeyPem = process.env.NEXT_PUBLIC_CONFIG_PUBLIC_KEY;
    // Convert PEM to CryptoKey...
    throw new Error('Implementation required');
  }
  
  private isValidConfig(config: unknown): config is SecureConfig {
    if (typeof config !== 'object' || config === null) return false;
    
    const c = config as any;
    return (
      typeof c.apiUrl === 'string' &&
      Array.isArray(c.features) &&
      typeof c.limits === 'object' &&
      typeof c.limits.maxUploadSize === 'number' &&
      typeof c.limits.rateLimit === 'number'
    );
  }
}
```

---

# 45. Security Performance Considerations

## 45.1 Efficient Security Checks

```typescript
// Use caching for repeated validations
const tokenCache = new Map<string, { valid: boolean; expiry: number }>();

function isTokenValid(token: string): boolean {
  // Check cache first
  const cached = tokenCache.get(token);
  if (cached && cached.expiry > Date.now()) {
    return cached.valid;
  }
  
  // Expensive validation
  const valid = validateToken(token);
  
  // Cache for 5 minutes
  tokenCache.set(token, {
    valid,
    expiry: Date.now() + 5 * 60 * 1000,
  });
  
  return valid;
}

// Clean up expired cache entries periodically
setInterval(() => {
  const now = Date.now();
  for (const [key, value] of tokenCache) {
    if (value.expiry < now) {
      tokenCache.delete(key);
    }
  }
}, 60000);
```

## 45.2 Lazy Security Initialization

```typescript
// Only load security features when needed
let domPurifyInstance: DOMPurify.DOMPurifyI | null = null;

async function sanitizeHTML(html: string): Promise<string> {
  if (!domPurifyInstance) {
    const DOMPurify = await import('dompurify');
    domPurifyInstance = DOMPurify.default;
  }
  
  return domPurifyInstance.sanitize(html);
}
```

---

# 46. Quick Security Tips Summary

## 46.1 One-Liner Security Rules

```typescript
const SECURITY_ONE_LINERS = [
  // XSS
  'Never use innerHTML with user input',
  'Always use DOMPurify before dangerouslySetInnerHTML',
  'Validate URLs before using as href',
  
  // Authentication
  'Never store tokens in localStorage',
  'Use HttpOnly, Secure, SameSite cookies',
  'Keep access tokens short-lived (15 min)',
  
  // API Security
  'Validate all input on server',
  'Return generic error messages',
  'Implement rate limiting',
  
  // Headers
  'Set Content-Security-Policy',
  'Set X-Frame-Options: DENY',
  'Enable HSTS in production',
  
  // Secrets
  'Never commit secrets to git',
  'Use environment variables',
  'Rotate secrets regularly',
];
```

## 46.2 Security Checklist Before Code Review

```typescript
const CODE_REVIEW_CHECKLIST = [
  // Check for XSS vulnerabilities
  'Grep for dangerouslySetInnerHTML - is it sanitized?',
  'Check for innerHTML usage - is input escaped?',
  'Verify URL inputs are validated',
  
  // Check authentication
  'Are tokens stored securely?',
  'Are sensitive routes protected?',
  'Is there proper session management?',
  
  // Check API calls
  'Are credentials included correctly?',
  'Is error handling secure (no detail leaks)?',
  'Are inputs validated before sending?',
  
  // Check for secrets
  'Are there any hardcoded secrets?',
  'Are environment variables used correctly?',
  'Is there any sensitive data in logs?',
];
```

## 46.3 Emergency Response Cheat Sheet

```typescript
const EMERGENCY_ACTIONS = {
  tokenCompromise: [
    '1. Rotate all secrets immediately',
    '2. Invalidate all existing sessions',
    '3. Force password reset for affected users',
    '4. Review logs for unauthorized access',
    '5. Patch the vulnerability',
  ],
  
  xssDetected: [
    '1. Take affected pages offline if severe',
    '2. Identify and patch the vulnerability',
    '3. Review CSP configuration',
    '4. Check for stored XSS in database',
    '5. Notify affected users if data exposed',
  ],
  
  dataBreachSuspected: [
    '1. Isolate affected systems',
    '2. Preserve logs and evidence',
    '3. Assess scope of breach',
    '4. Notify legal/compliance team',
    '5. Prepare user notification',
  ],
};
```

---

# 47. Study Resources

## 47.1 Essential Reading

| Resource | URL | Focus |
|----------|-----|-------|
| OWASP Top 10 | owasp.org | Common vulnerabilities |
| MDN Web Security | developer.mozilla.org | Browser security |
| PortSwigger Academy | portswigger.net | Hands-on labs |
| Google Web Fundamentals | developers.google.com | Best practices |

## 47.2 Tools to Practice With

```typescript
const PRACTICE_TOOLS = {
  learningPlatforms: [
    'OWASP WebGoat',
    'HackTheBox',
    'PortSwigger Web Security Academy',
  ],
  
  scanningTools: [
    'OWASP ZAP',
    'Burp Suite',
    'npm audit',
    'Snyk',
  ],
  
  browserTools: [
    'DevTools Security tab',
    'Lighthouse Security audit',
    'Content Security Policy Evaluator',
  ],
};
```

## 47.3 Interview Preparation Timeline

```typescript
const STUDY_TIMELINE = {
  week1: [
    'XSS types and prevention',
    'CSRF and protections',
    'Basic authentication concepts',
  ],
  
  week2: [
    'JWT structure and security',
    'OAuth 2.0 flows',
    'Session management',
  ],
  
  week3: [
    'Security headers (CSP, HSTS, etc.)',
    'CORS configuration',
    'Cookie security',
  ],
  
  week4: [
    'Practice coding challenges',
    'Review common vulnerabilities',
    'Mock interviews',
  ],
};
```

---

*End of Part 8: Security Best Practices Complete Guide (6000+ lines)*

*Continue to Part 9 for Real-Time Systems (WebSockets & Socket.io)...*

