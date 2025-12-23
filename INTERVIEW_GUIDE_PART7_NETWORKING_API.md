# 📚 Part 7: Networking & API Integration Complete Guide

> **Master HTTP, Fetch, Axios, REST, GraphQL, and API patterns for frontend interviews**

---

## 📋 Table of Contents

1. [HTTP Fundamentals](#1-http-fundamentals)
2. [Fetch API](#2-fetch-api)
3. [Axios](#3-axios)
4. [REST API Design](#4-rest-api-design)
5. [GraphQL](#5-graphql)
6. [Error Handling](#6-error-handling)
7. [Authentication](#7-authentication)
8. [Caching Strategies](#8-caching-strategies)
9. [Request Optimization](#9-request-optimization)
10. [Real-World Patterns](#10-real-world-patterns)
11. [Interview Questions](#11-interview-questions)
12. [Coding Challenges](#12-coding-challenges)

---

# 1. HTTP Fundamentals

## 1.1 HTTP Methods

```typescript
// HTTP Methods and their purposes
const HTTP_METHODS = {
  GET: 'Retrieve data (idempotent, safe)',
  POST: 'Create new resource',
  PUT: 'Replace entire resource (idempotent)',
  PATCH: 'Partial update (not idempotent)',
  DELETE: 'Remove resource (idempotent)',
  HEAD: 'Same as GET but no body',
  OPTIONS: 'Check allowed methods (CORS preflight)',
};

// Idempotent = Same result no matter how many times called
// Safe = Does not modify server state
```

## 1.2 HTTP Status Codes

```typescript
const STATUS_CODES = {
  // 1xx Informational
  100: 'Continue',
  101: 'Switching Protocols',
  
  // 2xx Success
  200: 'OK',
  201: 'Created',
  204: 'No Content',
  
  // 3xx Redirection
  301: 'Moved Permanently',
  302: 'Found (Temporary Redirect)',
  304: 'Not Modified',
  
  // 4xx Client Errors
  400: 'Bad Request',
  401: 'Unauthorized (not authenticated)',
  403: 'Forbidden (not authorized)',
  404: 'Not Found',
  409: 'Conflict',
  422: 'Unprocessable Entity',
  429: 'Too Many Requests',
  
  // 5xx Server Errors
  500: 'Internal Server Error',
  502: 'Bad Gateway',
  503: 'Service Unavailable',
  504: 'Gateway Timeout',
};

// Status code categories
function getStatusCategory(status: number): string {
  if (status >= 100 && status < 200) return 'Informational';
  if (status >= 200 && status < 300) return 'Success';
  if (status >= 300 && status < 400) return 'Redirection';
  if (status >= 400 && status < 500) return 'Client Error';
  if (status >= 500 && status < 600) return 'Server Error';
  return 'Unknown';
}
```

## 1.3 HTTP Headers

```typescript
// Common Request Headers
const REQUEST_HEADERS = {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'Authorization': 'Bearer token',
  'Cache-Control': 'no-cache',
  'User-Agent': 'Browser info',
  'Accept-Language': 'en-US,en;q=0.9',
  'X-Requested-With': 'XMLHttpRequest',
};

// Common Response Headers
const RESPONSE_HEADERS = {
  'Content-Type': 'application/json',
  'Cache-Control': 'max-age=3600',
  'ETag': '"abc123"',
  'Last-Modified': 'Wed, 21 Oct 2023 07:28:00 GMT',
  'Access-Control-Allow-Origin': '*',
  'X-RateLimit-Limit': '100',
  'X-RateLimit-Remaining': '95',
};

// CORS Headers
const CORS_HEADERS = {
  'Access-Control-Allow-Origin': 'https://example.com',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
  'Access-Control-Allow-Credentials': 'true',
  'Access-Control-Max-Age': '86400',
};
```

## 1.4 CORS Explained

```typescript
// CORS = Cross-Origin Resource Sharing
// Browser security feature that restricts cross-origin HTTP requests

// Same-origin policy: protocol + domain + port must match
// https://example.com:443 and https://example.com:443 = Same origin
// https://example.com and http://example.com = Different origin (protocol)
// https://example.com and https://api.example.com = Different origin (subdomain)

// Simple requests (no preflight)
// - Methods: GET, HEAD, POST
// - Headers: Accept, Accept-Language, Content-Language, Content-Type
// - Content-Type: application/x-www-form-urlencoded, multipart/form-data, text/plain

// Preflighted requests
// Browser sends OPTIONS request first
async function preflightExample() {
  // This triggers preflight because of custom header
  const response = await fetch('https://api.example.com/data', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Custom-Header': 'value',
    },
    body: JSON.stringify({ data: 'test' }),
  });
}

// Credentials with CORS
fetch('https://api.example.com/data', {
  credentials: 'include', // Send cookies cross-origin
});
```

---

# 2. Fetch API

## 2.1 Basic Fetch

```typescript
// GET request
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// POST request
async function createUser(user: User) {
  const response = await fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(user),
  });
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return response.json();
}

// PUT request
async function updateUser(id: string, user: User) {
  const response = await fetch(`https://api.example.com/users/${id}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(user),
  });
  
  return response.json();
}

// DELETE request
async function deleteUser(id: string) {
  const response = await fetch(`https://api.example.com/users/${id}`, {
    method: 'DELETE',
  });
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return true;
}
```

## 2.2 Fetch with TypeScript

```typescript
// Type-safe fetch wrapper
async function fetchJSON<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });
  
  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new APIError(response.status, error.message || 'Request failed');
  }
  
  return response.json();
}

// Custom error class
class APIError extends Error {
  constructor(
    public status: number,
    message: string,
    public data?: unknown
  ) {
    super(message);
    this.name = 'APIError';
  }
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

const users = await fetchJSON<User[]>('/api/users');
const user = await fetchJSON<User>('/api/users/1');
```

## 2.3 Abort Controller

```typescript
// Cancel fetch requests
function fetchWithTimeout(url: string, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  return fetch(url, { signal: controller.signal })
    .finally(() => clearTimeout(timeoutId));
}

// Cancel on component unmount
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, { signal: controller.signal });
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => {
      controller.abort();
    };
  }, [url]);
  
  return { data, loading, error };
}
```

## 2.4 File Upload

```typescript
// Upload single file
async function uploadFile(file: File) {
  const formData = new FormData();
  formData.append('file', file);
  
  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
    // Don't set Content-Type header - browser sets it with boundary
  });
  
  return response.json();
}

// Upload multiple files with progress
async function uploadWithProgress(
  files: File[],
  onProgress: (progress: number) => void
) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    
    files.forEach((file, i) => {
      formData.append(`file${i}`, file);
    });
    
    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable) {
        const progress = (event.loaded / event.total) * 100;
        onProgress(Math.round(progress));
      }
    });
    
    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });
    
    xhr.addEventListener('error', () => reject(new Error('Upload failed')));
    
    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  });
}
```

---

# 3. Axios

## 3.1 Basic Axios

```typescript
import axios from 'axios';

// Create instance with defaults
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// GET request
async function getUsers() {
  const { data } = await api.get<User[]>('/users');
  return data;
}

// POST request
async function createUser(user: CreateUserDTO) {
  const { data } = await api.post<User>('/users', user);
  return data;
}

// PUT request
async function updateUser(id: string, user: UpdateUserDTO) {
  const { data } = await api.put<User>(`/users/${id}`, user);
  return data;
}

// DELETE request
async function deleteUser(id: string) {
  await api.delete(`/users/${id}`);
}

// PATCH request
async function patchUser(id: string, updates: Partial<User>) {
  const { data } = await api.patch<User>(`/users/${id}`, updates);
  return data;
}
```

## 3.2 Axios Interceptors

```typescript
// Request interceptor
api.interceptors.request.use(
  (config) => {
    // Add auth token to every request
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
api.interceptors.response.use(
  (response) => {
    // Transform response data
    return response;
  },
  async (error) => {
    const originalRequest = error.config;
    
    // Handle 401 - token expired
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const { data } = await axios.post('/auth/refresh', { refreshToken });
        
        localStorage.setItem('token', data.accessToken);
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        
        return api(originalRequest);
      } catch (refreshError) {
        // Redirect to login
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);
```

## 3.3 Request Cancellation

```typescript
// Using CancelToken (deprecated but common)
const source = axios.CancelToken.source();

api.get('/users', {
  cancelToken: source.token,
}).catch((thrown) => {
  if (axios.isCancel(thrown)) {
    console.log('Request cancelled:', thrown.message);
  }
});

source.cancel('Operation cancelled by user');

// Using AbortController (recommended)
const controller = new AbortController();

api.get('/users', {
  signal: controller.signal,
}).catch((error) => {
  if (error.name === 'CanceledError') {
    console.log('Request cancelled');
  }
});

controller.abort();

// React hook with cancellation
function useAxios<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    api.get<T>(url, { signal: controller.signal })
      .then(({ data }) => {
        setData(data);
        setLoading(false);
      })
      .catch((err) => {
        if (!axios.isCancel(err)) {
          setError(err);
          setLoading(false);
        }
      });
    
    return () => {
      controller.abort();
    };
  }, [url]);
  
  return { data, loading, error };
}
```

## 3.4 Fetch vs Axios Comparison

| Feature | Fetch | Axios |
|---------|-------|-------|
| Browser support | Modern browsers | All (with polyfill) |
| Request/Response transforms | Manual | Automatic JSON |
| Request timeout | AbortController | Built-in |
| HTTP interceptors | No | Yes |
| Request cancellation | AbortController | AbortController |
| Upload progress | Via XMLHttpRequest | Built-in |
| Error handling | Manual status check | Automatic for non-2xx |
| Bundle size | 0 (native) | ~13KB |

---

# 4. REST API Design

## 4.1 REST Principles

```typescript
// REST = Representational State Transfer

// 1. Client-Server
// 2. Stateless - no session state on server
// 3. Cacheable
// 4. Uniform Interface
// 5. Layered System
// 6. Code on Demand (optional)

// Resource-based URLs
const endpoints = {
  // Good - noun-based, hierarchical
  '/users': 'List users',
  '/users/:id': 'Get user by ID',
  '/users/:id/posts': 'Get user posts',
  '/posts/:id/comments': 'Get post comments',
  
  // Bad - verb-based
  '/getUsers': 'Avoid',
  '/createUser': 'Avoid',
  '/deleteUserById': 'Avoid',
};
```

## 4.2 API Client Implementation

```typescript
// Type-safe API client
class APIClient {
  private baseURL: string;
  private headers: Record<string, string>;
  
  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.headers = {
      'Content-Type': 'application/json',
    };
  }
  
  setAuthToken(token: string) {
    this.headers['Authorization'] = `Bearer ${token}`;
  }
  
  clearAuthToken() {
    delete this.headers['Authorization'];
  }
  
  private async request<T>(
    method: string,
    endpoint: string,
    body?: unknown
  ): Promise<T> {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      method,
      headers: this.headers,
      body: body ? JSON.stringify(body) : undefined,
    });
    
    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new APIError(response.status, error.message, error);
    }
    
    if (response.status === 204) {
      return undefined as T;
    }
    
    return response.json();
  }
  
  get<T>(endpoint: string): Promise<T> {
    return this.request<T>('GET', endpoint);
  }
  
  post<T>(endpoint: string, body: unknown): Promise<T> {
    return this.request<T>('POST', endpoint, body);
  }
  
  put<T>(endpoint: string, body: unknown): Promise<T> {
    return this.request<T>('PUT', endpoint, body);
  }
  
  patch<T>(endpoint: string, body: unknown): Promise<T> {
    return this.request<T>('PATCH', endpoint, body);
  }
  
  delete<T>(endpoint: string): Promise<T> {
    return this.request<T>('DELETE', endpoint);
  }
}

// Usage
const api = new APIClient('https://api.example.com');
api.setAuthToken('jwt-token');

const users = await api.get<User[]>('/users');
const user = await api.post<User>('/users', { name: 'John', email: 'john@example.com' });
```

## 4.3 Query Parameters

```typescript
// Building query strings
function buildQueryString(params: Record<string, unknown>): string {
  const searchParams = new URLSearchParams();
  
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      if (Array.isArray(value)) {
        value.forEach((v) => searchParams.append(key, String(v)));
      } else {
        searchParams.append(key, String(value));
      }
    }
  });
  
  return searchParams.toString();
}

// Usage
const query = buildQueryString({
  page: 1,
  limit: 10,
  sort: 'createdAt',
  order: 'desc',
  tags: ['javascript', 'react'],
});
// Output: page=1&limit=10&sort=createdAt&order=desc&tags=javascript&tags=react

// Pagination pattern
interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

async function fetchPaginated<T>(
  endpoint: string,
  page = 1,
  limit = 10
): Promise<PaginatedResponse<T>> {
  const query = buildQueryString({ page, limit });
  return api.get<PaginatedResponse<T>>(`${endpoint}?${query}`);
}
```

---

# 5. GraphQL

## 5.1 GraphQL Basics

```typescript
// GraphQL query
const GET_USERS = `
  query GetUsers($limit: Int, $offset: Int) {
    users(limit: $limit, offset: $offset) {
      id
      name
      email
      posts {
        id
        title
      }
    }
  }
`;

// GraphQL mutation
const CREATE_USER = `
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

// Fetch GraphQL endpoint
async function graphqlFetch<T>(
  query: string,
  variables?: Record<string, unknown>
): Promise<T> {
  const response = await fetch('/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, variables }),
  });
  
  const { data, errors } = await response.json();
  
  if (errors) {
    throw new Error(errors[0].message);
  }
  
  return data;
}

// Usage
const data = await graphqlFetch<{ users: User[] }>(GET_USERS, {
  limit: 10,
  offset: 0,
});
```

## 5.2 Apollo Client

```typescript
import { ApolloClient, InMemoryCache, gql, useQuery, useMutation } from '@apollo/client';

// Setup client
const client = new ApolloClient({
  uri: 'https://api.example.com/graphql',
  cache: new InMemoryCache(),
  headers: {
    authorization: localStorage.getItem('token') || '',
  },
});

// Query hook
const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

function UserList() {
  const { loading, error, data, refetch } = useQuery(GET_USERS);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data.users.map((user: User) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// Mutation hook
const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

function CreateUserForm() {
  const [createUser, { loading, error }] = useMutation(CREATE_USER, {
    refetchQueries: [{ query: GET_USERS }],
  });
  
  const handleSubmit = async (data: CreateUserInput) => {
    await createUser({ variables: { input: data } });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
    </form>
  );
}
```

---

# 6. Error Handling

## 6.1 Error Types

```typescript
// Custom error classes
class NetworkError extends Error {
  constructor(message = 'Network error') {
    super(message);
    this.name = 'NetworkError';
  }
}

class APIError extends Error {
  constructor(
    public status: number,
    message: string,
    public code?: string,
    public details?: unknown
  ) {
    super(message);
    this.name = 'APIError';
  }
  
  get isClientError(): boolean {
    return this.status >= 400 && this.status < 500;
  }
  
  get isServerError(): boolean {
    return this.status >= 500;
  }
  
  get isUnauthorized(): boolean {
    return this.status === 401;
  }
  
  get isForbidden(): boolean {
    return this.status === 403;
  }
  
  get isNotFound(): boolean {
    return this.status === 404;
  }
}

class ValidationError extends Error {
  constructor(
    public errors: Record<string, string[]>
  ) {
    super('Validation failed');
    this.name = 'ValidationError';
  }
}
```

## 6.2 Global Error Handler

```typescript
// Error handler service
class ErrorHandler {
  handle(error: unknown): void {
    if (error instanceof APIError) {
      this.handleAPIError(error);
    } else if (error instanceof NetworkError) {
      this.handleNetworkError(error);
    } else if (error instanceof ValidationError) {
      this.handleValidationError(error);
    } else {
      this.handleUnknownError(error);
    }
  }
  
  private handleAPIError(error: APIError): void {
    switch (error.status) {
      case 401:
        // Redirect to login
        window.location.href = '/login';
        break;
      case 403:
        toast.error('You do not have permission to perform this action');
        break;
      case 404:
        toast.error('Resource not found');
        break;
      case 429:
        toast.error('Too many requests. Please try again later.');
        break;
      default:
        toast.error(error.message || 'An error occurred');
    }
  }
  
  private handleNetworkError(error: NetworkError): void {
    toast.error('Network error. Please check your connection.');
  }
  
  private handleValidationError(error: ValidationError): void {
    const messages = Object.values(error.errors).flat();
    toast.error(messages.join('\n'));
  }
  
  private handleUnknownError(error: unknown): void {
    console.error('Unknown error:', error);
    toast.error('An unexpected error occurred');
  }
}

const errorHandler = new ErrorHandler();
```

## 6.3 React Error Boundary for API

```tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback: ReactNode | ((error: Error, retry: () => void) => ReactNode);
}

interface State {
  error: Error | null;
}

class APIErrorBoundary extends Component<Props, State> {
  state: State = { error: null };
  
  static getDerivedStateFromError(error: Error): State {
    return { error };
  }
  
  retry = () => {
    this.setState({ error: null });
  };
  
  render() {
    if (this.state.error) {
      if (typeof this.props.fallback === 'function') {
        return this.props.fallback(this.state.error, this.retry);
      }
      return this.props.fallback;
    }
    
    return this.props.children;
  }
}

// Usage
<APIErrorBoundary
  fallback={(error, retry) => (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={retry}>Retry</button>
    </div>
  )}
>
  <UserList />
</APIErrorBoundary>
```

---

# 7. Authentication

## 7.1 JWT Authentication

```typescript
// JWT structure: header.payload.signature
interface JWTPayload {
  sub: string;      // Subject (user ID)
  exp: number;      // Expiration time
  iat: number;      // Issued at
  roles?: string[]; // User roles
}

// Decode JWT (without verification)
function decodeJWT(token: string): JWTPayload {
  const base64Url = token.split('.')[1];
  const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
  const jsonPayload = decodeURIComponent(
    atob(base64)
      .split('')
      .map((c) => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
      .join('')
  );
  return JSON.parse(jsonPayload);
}

// Check if token is expired
function isTokenExpired(token: string): boolean {
  try {
    const payload = decodeJWT(token);
    return Date.now() >= payload.exp * 1000;
  } catch {
    return true;
  }
}

// Auth service
class AuthService {
  private accessToken: string | null = null;
  private refreshToken: string | null = null;
  
  setTokens(access: string, refresh: string) {
    this.accessToken = access;
    this.refreshToken = refresh;
    localStorage.setItem('accessToken', access);
    localStorage.setItem('refreshToken', refresh);
  }
  
  getAccessToken(): string | null {
    if (!this.accessToken) {
      this.accessToken = localStorage.getItem('accessToken');
    }
    return this.accessToken;
  }
  
  async refreshAccessToken(): Promise<string> {
    const refresh = this.refreshToken || localStorage.getItem('refreshToken');
    
    if (!refresh) {
      throw new Error('No refresh token');
    }
    
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken: refresh }),
    });
    
    if (!response.ok) {
      this.logout();
      throw new Error('Failed to refresh token');
    }
    
    const { accessToken, refreshToken } = await response.json();
    this.setTokens(accessToken, refreshToken);
    return accessToken;
  }
  
  logout() {
    this.accessToken = null;
    this.refreshToken = null;
    localStorage.removeItem('accessToken');
    localStorage.removeItem('refreshToken');
  }
  
  isAuthenticated(): boolean {
    const token = this.getAccessToken();
    return token !== null && !isTokenExpired(token);
  }
}

export const authService = new AuthService();
```

## 7.2 OAuth 2.0 Flow

```typescript
// OAuth 2.0 Authorization Code Flow
class OAuthService {
  private clientId: string;
  private redirectUri: string;
  private authUrl: string;
  
  constructor(config: OAuthConfig) {
    this.clientId = config.clientId;
    this.redirectUri = config.redirectUri;
    this.authUrl = config.authUrl;
  }
  
  // Step 1: Redirect to OAuth provider
  initiateLogin(scopes: string[] = ['openid', 'profile', 'email']) {
    const state = this.generateState();
    sessionStorage.setItem('oauth_state', state);
    
    const params = new URLSearchParams({
      client_id: this.clientId,
      redirect_uri: this.redirectUri,
      response_type: 'code',
      scope: scopes.join(' '),
      state,
    });
    
    window.location.href = `${this.authUrl}?${params}`;
  }
  
  // Step 2: Handle callback
  async handleCallback(): Promise<AuthResult> {
    const params = new URLSearchParams(window.location.search);
    const code = params.get('code');
    const state = params.get('state');
    const savedState = sessionStorage.getItem('oauth_state');
    
    if (!code) {
      throw new Error('No authorization code');
    }
    
    if (state !== savedState) {
      throw new Error('Invalid state');
    }
    
    sessionStorage.removeItem('oauth_state');
    
    // Exchange code for tokens
    const response = await fetch('/api/auth/oauth/callback', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ code }),
    });
    
    if (!response.ok) {
      throw new Error('Token exchange failed');
    }
    
    return response.json();
  }
  
  private generateState(): string {
    return crypto.randomUUID();
  }
}
```

---

# 8. Caching Strategies

## 8.1 HTTP Caching

```typescript
// Cache-Control header values
const CACHE_STRATEGIES = {
  // No caching
  noCache: 'no-store, no-cache, must-revalidate',
  
  // Cache for 1 hour, revalidate after
  standard: 'public, max-age=3600, stale-while-revalidate=86400',
  
  // Immutable (for hashed assets)
  immutable: 'public, max-age=31536000, immutable',
  
  // Private (user-specific data)
  private: 'private, max-age=3600',
};

// ETag-based caching
async function fetchWithETag(url: string) {
  const cachedETag = localStorage.getItem(`etag:${url}`);
  const cachedData = localStorage.getItem(`data:${url}`);
  
  const headers: HeadersInit = {};
  if (cachedETag) {
    headers['If-None-Match'] = cachedETag;
  }
  
  const response = await fetch(url, { headers });
  
  if (response.status === 304 && cachedData) {
    // Not modified, use cached data
    return JSON.parse(cachedData);
  }
  
  const data = await response.json();
  const etag = response.headers.get('ETag');
  
  if (etag) {
    localStorage.setItem(`etag:${url}`, etag);
    localStorage.setItem(`data:${url}`, JSON.stringify(data));
  }
  
  return data;
}
```

## 8.2 Client-Side Caching

```typescript
// Simple in-memory cache
class Cache<T> {
  private cache = new Map<string, { data: T; expiry: number }>();
  
  set(key: string, data: T, ttlMs: number = 60000) {
    this.cache.set(key, {
      data,
      expiry: Date.now() + ttlMs,
    });
  }
  
  get(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) return null;
    
    if (Date.now() > entry.expiry) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  has(key: string): boolean {
    return this.get(key) !== null;
  }
  
  delete(key: string) {
    this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
}

// Cached API function
const apiCache = new Cache<unknown>();

async function cachedFetch<T>(
  url: string,
  options?: { ttl?: number; force?: boolean }
): Promise<T> {
  const cacheKey = url;
  
  if (!options?.force) {
    const cached = apiCache.get(cacheKey);
    if (cached) return cached as T;
  }
  
  const data = await fetch(url).then((r) => r.json());
  apiCache.set(cacheKey, data, options?.ttl);
  
  return data;
}
```

## 8.3 React Query Caching

```typescript
import { QueryClient, QueryClientProvider, useQuery } from '@tanstack/react-query';

// Configure cache behavior
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 30 * 60 * 1000,        // 30 minutes (formerly cacheTime)
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
    },
  },
});

// Query with cache key
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 10 * 60 * 1000, // Override default
  });
}

// Prefetching
async function prefetchUser(userId: string) {
  await queryClient.prefetchQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
}

// Invalidation
function invalidateUser(userId: string) {
  queryClient.invalidateQueries({ queryKey: ['user', userId] });
}

// Manual cache update
function updateUserCache(userId: string, data: User) {
  queryClient.setQueryData(['user', userId], data);
}
```

---

# 9. Request Optimization

## 9.1 Debouncing and Throttling

```typescript
// Debounce - wait for pause in calls
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeoutId: NodeJS.Timeout;
  
  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), wait);
  };
}

// Throttle - limit call frequency
function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle = false;
  
  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Debounced search
const debouncedSearch = debounce(async (query: string) => {
  const results = await api.get(`/search?q=${query}`);
  setResults(results);
}, 300);

// Usage in React
function SearchInput() {
  const [query, setQuery] = useState('');
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
}
```

## 9.2 Request Batching

```typescript
// Batch multiple requests into one
class RequestBatcher<T, R> {
  private queue: Array<{
    item: T;
    resolve: (value: R) => void;
    reject: (error: Error) => void;
  }> = [];
  private timeout: NodeJS.Timeout | null = null;
  
  constructor(
    private batchFn: (items: T[]) => Promise<R[]>,
    private delay = 50,
    private maxBatchSize = 100
  ) {}
  
  async add(item: T): Promise<R> {
    return new Promise((resolve, reject) => {
      this.queue.push({ item, resolve, reject });
      
      if (this.queue.length >= this.maxBatchSize) {
        this.flush();
      } else if (!this.timeout) {
        this.timeout = setTimeout(() => this.flush(), this.delay);
      }
    });
  }
  
  private async flush() {
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    }
    
    const batch = this.queue;
    this.queue = [];
    
    try {
      const items = batch.map((b) => b.item);
      const results = await this.batchFn(items);
      
      batch.forEach((b, i) => b.resolve(results[i]));
    } catch (error) {
      batch.forEach((b) => b.reject(error as Error));
    }
  }
}

// Usage: Batch user fetches
const userBatcher = new RequestBatcher<string, User>(
  async (ids) => {
    const response = await api.post('/users/batch', { ids });
    return response.data;
  }
);

// These will be batched into one request
const user1 = userBatcher.add('user-1');
const user2 = userBatcher.add('user-2');
const user3 = userBatcher.add('user-3');
```

## 9.3 Retry Logic

```typescript
interface RetryOptions {
  maxRetries?: number;
  initialDelay?: number;
  maxDelay?: number;
  backoffMultiplier?: number;
  retryCondition?: (error: Error) => boolean;
}

async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const {
    maxRetries = 3,
    initialDelay = 1000,
    maxDelay = 30000,
    backoffMultiplier = 2,
    retryCondition = () => true,
  } = options;
  
  let lastError: Error;
  let delay = initialDelay;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt === maxRetries || !retryCondition(lastError)) {
        throw lastError;
      }
      
      await new Promise((resolve) => setTimeout(resolve, delay));
      delay = Math.min(delay * backoffMultiplier, maxDelay);
    }
  }
  
  throw lastError!;
}

// Usage
const data = await fetchWithRetry(
  () => api.get('/unstable-endpoint'),
  {
    maxRetries: 3,
    retryCondition: (error) => {
      // Only retry on network errors or 5xx
      if (error instanceof APIError) {
        return error.isServerError;
      }
      return true;
    },
  }
);
```

---

# 10. Real-World Patterns

## 10.1 API Service Layer

```typescript
// Base API service
class BaseAPIService {
  protected api: APIClient;
  
  constructor() {
    this.api = new APIClient(process.env.NEXT_PUBLIC_API_URL!);
  }
}

// User service
class UserService extends BaseAPIService {
  async getAll(params?: { page?: number; limit?: number }): Promise<PaginatedResponse<User>> {
    const query = buildQueryString(params || {});
    return this.api.get(`/users?${query}`);
  }
  
  async getById(id: string): Promise<User> {
    return this.api.get(`/users/${id}`);
  }
  
  async create(data: CreateUserDTO): Promise<User> {
    return this.api.post('/users', data);
  }
  
  async update(id: string, data: UpdateUserDTO): Promise<User> {
    return this.api.patch(`/users/${id}`, data);
  }
  
  async delete(id: string): Promise<void> {
    return this.api.delete(`/users/${id}`);
  }
  
  async uploadAvatar(id: string, file: File): Promise<User> {
    const formData = new FormData();
    formData.append('avatar', file);
    
    return fetch(`${this.api.baseURL}/users/${id}/avatar`, {
      method: 'POST',
      body: formData,
    }).then((r) => r.json());
  }
}

export const userService = new UserService();
```

## 10.2 React Query Integration

```typescript
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '@/services/userService';

export function useUsers(params?: { page?: number; limit?: number }) {
  return useQuery({
    queryKey: ['users', params],
    queryFn: () => userService.getAll(params),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => userService.getById(id),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: userService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserDTO }) =>
      userService.update(id, data),
    onSuccess: (user) => {
      queryClient.setQueryData(['user', user.id], user);
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: userService.delete,
    onSuccess: (_, id) => {
      queryClient.removeQueries({ queryKey: ['user', id] });
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

## 10.3 Optimistic Updates

```typescript
export function useToggleTodo() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: todoService.toggle,
    
    onMutate: async (todoId: string) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      
      // Snapshot the previous value
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);
      
      // Optimistically update
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((todo) =>
          todo.id === todoId ? { ...todo, completed: !todo.completed } : todo
        )
      );
      
      // Return context with snapshot
      return { previousTodos };
    },
    
    onError: (err, todoId, context) => {
      // Rollback on error
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
      toast.error('Failed to update todo');
    },
    
    onSettled: () => {
      // Refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

---

# 11. Interview Questions

## Q1: What is the difference between PUT and PATCH?

**Answer:**
- **PUT** replaces the entire resource. You must send all fields.
- **PATCH** partially updates a resource. Send only changed fields.

```typescript
// PUT - Replace entire user
await api.put('/users/1', {
  name: 'John',
  email: 'john@example.com',
  age: 30,
  address: '123 Main St',
});

// PATCH - Update only email
await api.patch('/users/1', {
  email: 'newemail@example.com',
});
```

---

## Q2: Explain CORS and how to handle it

**Answer:**
CORS (Cross-Origin Resource Sharing) is a security mechanism that restricts cross-origin HTTP requests. The browser blocks requests to different origins unless the server explicitly allows it.

**Solutions:**
1. Server adds CORS headers
2. Use a proxy in development
3. Use CORS middleware (Express)

```typescript
// Express CORS middleware
app.use(cors({
  origin: 'https://myapp.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
}));

// Next.js API route
export async function GET() {
  return new Response(JSON.stringify(data), {
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```

---

## Q3: How do you handle API errors in React?

**Answer:**

```typescript
// 1. Try-catch in async functions
async function fetchData() {
  try {
    const data = await api.get('/data');
    return data;
  } catch (error) {
    if (error instanceof APIError) {
      handleAPIError(error);
    }
    throw error;
  }
}

// 2. React Query error handling
const { data, error, isError } = useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retry: 3,
});

if (isError) {
  return <ErrorMessage error={error} />;
}

// 3. Error Boundary for synchronous errors
<ErrorBoundary fallback={<ErrorPage />}>
  <Component />
</ErrorBoundary>

// 4. Global error handler
api.interceptors.response.use(
  (response) => response,
  (error) => {
    handleGlobalError(error);
    return Promise.reject(error);
  }
);
```

---

## Q4: What is request debouncing and when to use it?

**Answer:**
Debouncing delays execution until after a pause in calls. Use for:
- Search autocomplete
- Window resize handlers
- Form validation
- Scroll event handlers

```typescript
// Without debounce: 50 API calls for "hello"
// With debounce: 1 API call after user stops typing

const debouncedSearch = useMemo(
  () => debounce((query: string) => {
    api.get(`/search?q=${query}`);
  }, 300),
  []
);

useEffect(() => {
  if (searchQuery) {
    debouncedSearch(searchQuery);
  }
}, [searchQuery]);
```

---

## Q5: Explain the difference between REST and GraphQL

**Answer:**

| Feature | REST | GraphQL |
|---------|------|---------|
| Endpoints | Multiple endpoints | Single endpoint |
| Data fetching | Fixed response | Client specifies fields |
| Over-fetching | Common | Eliminated |
| Under-fetching | Common (multiple requests) | Eliminated |
| Caching | HTTP caching | Complex |
| Learning curve | Lower | Higher |
| Best for | Simple CRUD | Complex, nested data |

---

## Q6: How do you implement JWT refresh tokens?

**Answer:**

```typescript
// 1. Store tokens
localStorage.setItem('accessToken', accessToken);
localStorage.setItem('refreshToken', refreshToken);

// 2. Intercept 401 responses
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      
      const refreshToken = localStorage.getItem('refreshToken');
      const { accessToken } = await authService.refresh(refreshToken);
      
      localStorage.setItem('accessToken', accessToken);
      error.config.headers.Authorization = `Bearer ${accessToken}`;
      
      return api(error.config);
    }
    return Promise.reject(error);
  }
);
```

---

## Q7: What are HTTP status codes you should know?

**Answer:**
- **200 OK** - Successful GET/PUT/PATCH
- **201 Created** - Successful POST
- **204 No Content** - Successful DELETE
- **400 Bad Request** - Invalid request
- **401 Unauthorized** - Not authenticated
- **403 Forbidden** - Not authorized
- **404 Not Found** - Resource doesn't exist
- **409 Conflict** - Resource conflict
- **422 Unprocessable Entity** - Validation error
- **429 Too Many Requests** - Rate limited
- **500 Internal Server Error** - Server error
- **503 Service Unavailable** - Server temporarily down

---

## Q8: How do you cancel a fetch request?

**Answer:**

```typescript
// Using AbortController
const controller = new AbortController();

fetch('/api/data', { signal: controller.signal })
  .then((response) => response.json())
  .catch((error) => {
    if (error.name === 'AbortError') {
      console.log('Request cancelled');
    }
  });

// Cancel the request
controller.abort();

// In React useEffect
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(/* ... */);
  
  return () => controller.abort();
}, []);
```

---

# 12. Coding Challenges

## 12.1 Implement a Fetch Wrapper

```typescript
interface FetchOptions extends RequestInit {
  timeout?: number;
  retry?: number;
  retryDelay?: number;
}

async function enhancedFetch<T>(
  url: string,
  options: FetchOptions = {}
): Promise<T> {
  const {
    timeout = 10000,
    retry = 0,
    retryDelay = 1000,
    ...fetchOptions
  } = options;
  
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  const attempt = async (retriesLeft: number): Promise<T> => {
    try {
      const response = await fetch(url, {
        ...fetchOptions,
        signal: controller.signal,
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new APIError(response.status, await response.text());
      }
      
      return response.json();
    } catch (error) {
      if (retriesLeft > 0 && !(error instanceof DOMException)) {
        await new Promise((r) => setTimeout(r, retryDelay));
        return attempt(retriesLeft - 1);
      }
      throw error;
    }
  };
  
  return attempt(retry);
}

// Usage
const data = await enhancedFetch<User[]>('/api/users', {
  timeout: 5000,
  retry: 3,
  retryDelay: 1000,
});
```

## 12.2 Implement Request Queue

```typescript
class RequestQueue {
  private queue: Array<() => Promise<void>> = [];
  private running = 0;
  private maxConcurrent: number;
  
  constructor(maxConcurrent = 3) {
    this.maxConcurrent = maxConcurrent;
  }
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
    while (this.running < this.maxConcurrent && this.queue.length > 0) {
      const task = this.queue.shift()!;
      this.running++;
      
      task().finally(() => {
        this.running--;
        this.processQueue();
      });
    }
  }
}

// Usage: Limit concurrent API calls
const queue = new RequestQueue(3);

const urls = ['/api/1', '/api/2', '/api/3', '/api/4', '/api/5'];
const results = await Promise.all(
  urls.map((url) => queue.add(() => fetch(url)))
);
```

## 12.3 Implement API Mock for Testing

```typescript
type MockResponse = {
  status?: number;
  data?: unknown;
  delay?: number;
};

class MockAPI {
  private mocks = new Map<string, MockResponse>();
  
  mock(url: string, response: MockResponse) {
    this.mocks.set(url, response);
  }
  
  clear() {
    this.mocks.clear();
  }
  
  async fetch(url: string, options?: RequestInit): Promise<Response> {
    const mock = this.mocks.get(url);
    
    if (!mock) {
      throw new Error(`No mock for ${url}`);
    }
    
    if (mock.delay) {
      await new Promise((r) => setTimeout(r, mock.delay));
    }
    
    return new Response(JSON.stringify(mock.data), {
      status: mock.status || 200,
      headers: { 'Content-Type': 'application/json' },
    });
  }
}

// Usage in tests
const mockAPI = new MockAPI();

mockAPI.mock('/api/users', {
  status: 200,
  data: [{ id: 1, name: 'John' }],
  delay: 100,
});

const response = await mockAPI.fetch('/api/users');
const users = await response.json();
expect(users).toHaveLength(1);
```

---

## 🎯 Study Checklist

### HTTP Fundamentals
- [ ] HTTP methods and their purposes
- [ ] Status codes (2xx, 4xx, 5xx)
- [ ] Headers (Content-Type, Authorization, Cache-Control)
- [ ] CORS and preflight requests

### Fetch & Axios
- [ ] Basic GET, POST, PUT, DELETE
- [ ] Error handling
- [ ] Request cancellation
- [ ] Interceptors (Axios)
- [ ] File upload with progress

### API Design
- [ ] REST principles
- [ ] GraphQL basics
- [ ] Query parameters
- [ ] Pagination patterns

### Authentication
- [ ] JWT structure and validation
- [ ] Token refresh flow
- [ ] OAuth 2.0 basics

### Optimization
- [ ] Caching strategies
- [ ] Debouncing and throttling
- [ ] Request batching
- [ ] Retry logic

---

# 13. More Interview Questions

## Q9: What is the difference between fetch() and XMLHttpRequest?

**Answer:**

| Feature | fetch() | XMLHttpRequest |
|---------|---------|----------------|
| Promise-based | Yes | No (callbacks) |
| Streaming | Yes | No |
| Request cancellation | AbortController | abort() method |
| Progress events | Limited | Full support |
| Cookie handling | credentials option | withCredentials |
| Syntax | Modern, cleaner | Verbose |

```typescript
// XMLHttpRequest
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data');
xhr.onload = function() {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.onerror = function() {
  console.error('Error');
};
xhr.send();

// fetch
fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## Q10: How do you handle rate limiting?

**Answer:**

```typescript
class RateLimitedAPI {
  private retryAfter = 0;
  
  async request<T>(url: string, options?: RequestInit): Promise<T> {
    // Wait if rate limited
    if (this.retryAfter > Date.now()) {
      const waitTime = this.retryAfter - Date.now();
      await new Promise(r => setTimeout(r, waitTime));
    }
    
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      // Get retry-after header
      const retryAfter = response.headers.get('Retry-After');
      const waitSeconds = parseInt(retryAfter || '60', 10);
      this.retryAfter = Date.now() + waitSeconds * 1000;
      
      // Wait and retry
      await new Promise(r => setTimeout(r, waitSeconds * 1000));
      return this.request(url, options);
    }
    
    return response.json();
  }
}

// Exponential backoff for rate limiting
async function requestWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 5
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error instanceof APIError && error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(r => setTimeout(r, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

---

## Q11: What is HTTP/2 and how does it improve performance?

**Answer:**

HTTP/2 improvements over HTTP/1.1:

1. **Multiplexing** - Multiple requests over single connection
2. **Header compression** - HPACK compression
3. **Server push** - Server can push resources
4. **Binary protocol** - More efficient parsing
5. **Stream prioritization** - Priority order for resources

```typescript
// No code changes needed - browser handles it
// But consider:

// HTTP/1.1: Bundle files to reduce requests
import './styles.bundle.css';

// HTTP/2: Multiple small files is fine
import './header.css';
import './footer.css';
import './sidebar.css';

// Server push example (server-side)
res.push('/styles.css', {
  request: { accept: 'text/css' },
  response: { 'content-type': 'text/css' }
});
```

---

## Q12: Explain WebSocket handshake

**Answer:**

```typescript
// WebSocket upgrade handshake

// 1. Client sends upgrade request
// GET /chat HTTP/1.1
// Host: server.example.com
// Upgrade: websocket
// Connection: Upgrade
// Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
// Sec-WebSocket-Version: 13

// 2. Server responds with 101 status
// HTTP/1.1 101 Switching Protocols
// Upgrade: websocket
// Connection: Upgrade
// Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

// Client code
const ws = new WebSocket('wss://server.example.com/chat');

ws.onopen = () => {
  console.log('Connected');
  ws.send('Hello Server!');
};

ws.onmessage = (event) => {
  console.log('Message:', event.data);
};

ws.onclose = (event) => {
  console.log('Closed:', event.code, event.reason);
};

ws.onerror = (error) => {
  console.error('Error:', error);
};
```

---

## Q13: How do you implement infinite scroll with pagination?

**Answer:**

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';

interface PaginatedResponse<T> {
  data: T[];
  nextCursor?: string;
  hasMore: boolean;
}

function useInfiniteUsers() {
  return useInfiniteQuery({
    queryKey: ['users'],
    queryFn: async ({ pageParam = undefined }) => {
      const response = await fetch(
        `/api/users?cursor=${pageParam || ''}&limit=20`
      );
      return response.json() as Promise<PaginatedResponse<User>>;
    },
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    initialPageParam: undefined,
  });
}

function UserList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteUsers();
  
  const { ref, inView } = useInView();
  
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);
  
  if (isLoading) return <Spinner />;
  
  return (
    <div>
      {data?.pages.map((page, i) => (
        <Fragment key={i}>
          {page.data.map((user) => (
            <UserCard key={user.id} user={user} />
          ))}
        </Fragment>
      ))}
      
      <div ref={ref}>
        {isFetchingNextPage && <Spinner />}
      </div>
    </div>
  );
}
```

---

## Q14: What is Server-Sent Events (SSE)?

**Answer:**

SSE is a server push technology for real-time updates over HTTP.

```typescript
// Client side
function useSSE<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const eventSource = new EventSource(url);
    
    eventSource.onmessage = (event) => {
      setData(JSON.parse(event.data));
    };
    
    eventSource.onerror = (error) => {
      setError(new Error('SSE connection failed'));
      eventSource.close();
    };
    
    return () => {
      eventSource.close();
    };
  }, [url]);
  
  return { data, error };
}

// Server side (Node.js)
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  const sendEvent = (data: object) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };
  
  // Send events periodically
  const interval = setInterval(() => {
    sendEvent({ time: new Date().toISOString() });
  }, 1000);
  
  req.on('close', () => {
    clearInterval(interval);
  });
});
```

| Feature | SSE | WebSocket |
|---------|-----|-----------|
| Direction | Server → Client only | Bidirectional |
| Protocol | HTTP | WS |
| Reconnection | Automatic | Manual |
| Data format | Text only | Text & Binary |
| Best for | Notifications, feeds | Chat, games |

---

## Q15: How do you handle file downloads?

**Answer:**

```typescript
// Download file and trigger save
async function downloadFile(url: string, filename: string) {
  const response = await fetch(url);
  const blob = await response.blob();
  
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = filename;
  link.click();
  URL.revokeObjectURL(link.href);
}

// Download with progress
async function downloadWithProgress(
  url: string,
  onProgress: (progress: number) => void
): Promise<Blob> {
  const response = await fetch(url);
  const contentLength = response.headers.get('Content-Length');
  const total = parseInt(contentLength || '0', 10);
  
  const reader = response.body!.getReader();
  const chunks: Uint8Array[] = [];
  let received = 0;
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    chunks.push(value);
    received += value.length;
    
    if (total > 0) {
      onProgress((received / total) * 100);
    }
  }
  
  return new Blob(chunks);
}

// Usage
const blob = await downloadWithProgress('/large-file.zip', (progress) => {
  console.log(`Downloaded: ${progress.toFixed(1)}%`);
});
```

---

# 14. More Coding Challenges

## 14.1 Implement Polling Hook

```typescript
function usePolling<T>(
  fetcher: () => Promise<T>,
  interval: number,
  options: {
    enabled?: boolean;
    onError?: (error: Error) => void;
    onSuccess?: (data: T) => void;
  } = {}
) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);
  
  const { enabled = true, onError, onSuccess } = options;
  
  useEffect(() => {
    if (!enabled) return;
    
    let isMounted = true;
    
    const poll = async () => {
      try {
        const result = await fetcher();
        if (isMounted) {
          setData(result);
          setError(null);
          setLoading(false);
          onSuccess?.(result);
        }
      } catch (err) {
        if (isMounted) {
          setError(err as Error);
          setLoading(false);
          onError?.(err as Error);
        }
      }
    };
    
    // Initial fetch
    poll();
    
    // Start polling
    const intervalId = setInterval(poll, interval);
    
    return () => {
      isMounted = false;
      clearInterval(intervalId);
    };
  }, [fetcher, interval, enabled]);
  
  return { data, error, loading };
}

// Usage
function StockPrice({ symbol }: { symbol: string }) {
  const { data, loading } = usePolling(
    () => fetchStockPrice(symbol),
    5000, // Poll every 5 seconds
    {
      onSuccess: (price) => {
        if (price.change > 5) {
          toast.info(`${symbol} is up ${price.change}%!`);
        }
      },
    }
  );
  
  if (loading) return <Spinner />;
  
  return <div>${data?.price}</div>;
}
```

## 14.2 Implement Request Deduplication

```typescript
class RequestDeduplicator {
  private pending = new Map<string, Promise<any>>();
  
  async dedupe<T>(
    key: string,
    fetcher: () => Promise<T>
  ): Promise<T> {
    // Check if request is already in flight
    const existing = this.pending.get(key);
    if (existing) {
      return existing;
    }
    
    // Start new request
    const promise = fetcher().finally(() => {
      this.pending.delete(key);
    });
    
    this.pending.set(key, promise);
    return promise;
  }
}

const deduplicator = new RequestDeduplicator();

// Multiple components calling simultaneously
// Only one request is made
async function getUser(id: string) {
  return deduplicator.dedupe(
    `user-${id}`,
    () => api.get(`/users/${id}`)
  );
}

// All three calls share the same request
Promise.all([
  getUser('123'),
  getUser('123'),
  getUser('123'),
]);
```

## 14.3 Implement Request Waterfall Detection

```typescript
// Detect sequential requests that could be parallelized
class WaterfallDetector {
  private requests: Array<{ url: string; start: number; end: number }> = [];
  private threshold = 50; // ms
  
  track(url: string, duration: number) {
    const now = Date.now();
    this.requests.push({
      url,
      start: now - duration,
      end: now,
    });
    
    this.analyze();
  }
  
  private analyze() {
    const recent = this.requests.filter(
      (r) => Date.now() - r.end < 5000
    );
    
    // Find sequential requests
    for (let i = 1; i < recent.length; i++) {
      const prev = recent[i - 1];
      const curr = recent[i];
      
      // If current started right after previous ended
      if (curr.start - prev.end < this.threshold) {
        console.warn(
          `Potential waterfall detected:\n` +
          `  ${prev.url} -> ${curr.url}\n` +
          `Consider parallelizing these requests.`
        );
      }
    }
  }
}

// Middleware for axios
const detector = new WaterfallDetector();

api.interceptors.response.use((response) => {
  const duration = Date.now() - response.config.metadata?.startTime;
  detector.track(response.config.url!, duration);
  return response;
});

api.interceptors.request.use((config) => {
  config.metadata = { startTime: Date.now() };
  return config;
});
```

## 14.4 Implement Offline Queue

```typescript
class OfflineQueue {
  private queue: Array<{
    id: string;
    request: () => Promise<void>;
    retries: number;
  }> = [];
  private maxRetries = 3;
  private isOnline = navigator.onLine;
  
  constructor() {
    this.loadFromStorage();
    
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.processQueue();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }
  
  async add(id: string, request: () => Promise<void>) {
    this.queue.push({ id, request, retries: 0 });
    this.saveToStorage();
    
    if (this.isOnline) {
      this.processQueue();
    }
  }
  
  private async processQueue() {
    if (!this.isOnline || this.queue.length === 0) return;
    
    const item = this.queue[0];
    
    try {
      await item.request();
      this.queue.shift();
      this.saveToStorage();
      this.processQueue();
    } catch (error) {
      item.retries++;
      
      if (item.retries >= this.maxRetries) {
        console.error(`Failed after ${this.maxRetries} retries:`, item.id);
        this.queue.shift();
      }
      
      this.saveToStorage();
    }
  }
  
  private saveToStorage() {
    // Can't serialize functions, so store request data instead
    localStorage.setItem('offlineQueue', JSON.stringify(
      this.queue.map(q => ({ id: q.id, retries: q.retries }))
    ));
  }
  
  private loadFromStorage() {
    // Implementation depends on how requests are reconstructed
  }
}
```

## 14.5 Implement API Mocking Layer

```typescript
type MockHandler = (
  url: string,
  options?: RequestInit
) => Promise<Response> | Response;

class MockLayer {
  private mocks = new Map<string, MockHandler>();
  private enabled = false;
  private originalFetch: typeof fetch;
  
  constructor() {
    this.originalFetch = window.fetch;
  }
  
  enable() {
    this.enabled = true;
    window.fetch = this.mockFetch.bind(this);
  }
  
  disable() {
    this.enabled = false;
    window.fetch = this.originalFetch;
  }
  
  mock(pattern: string, handler: MockHandler) {
    this.mocks.set(pattern, handler);
  }
  
  private async mockFetch(
    input: RequestInfo | URL,
    init?: RequestInit
  ): Promise<Response> {
    const url = typeof input === 'string' ? input : input.toString();
    
    for (const [pattern, handler] of this.mocks) {
      if (this.matches(url, pattern)) {
        return handler(url, init);
      }
    }
    
    // Fall through to real fetch
    return this.originalFetch(input, init);
  }
  
  private matches(url: string, pattern: string): boolean {
    // Simple pattern matching
    const regex = new RegExp(
      '^' + pattern.replace(/\*/g, '.*') + '$'
    );
    return regex.test(url);
  }
}

// Usage
const mockLayer = new MockLayer();

mockLayer.mock('/api/users/*', (url) => {
  const id = url.split('/').pop();
  return new Response(
    JSON.stringify({ id, name: 'Mock User' }),
    { status: 200 }
  );
});

mockLayer.mock('/api/posts', () => {
  return new Response(
    JSON.stringify([
      { id: 1, title: 'Mock Post 1' },
      { id: 2, title: 'Mock Post 2' },
    ]),
    { status: 200 }
  );
});

mockLayer.enable();

// Now all matching requests return mock data
const user = await fetch('/api/users/123').then(r => r.json());
console.log(user); // { id: '123', name: 'Mock User' }
```

---

# 15. Advanced Topics

## 15.1 Request Signing

```typescript
// Sign requests for secure APIs (like AWS)
import crypto from 'crypto';

class SignedRequestClient {
  constructor(
    private accessKey: string,
    private secretKey: string
  ) {}
  
  async signedFetch(url: string, options: RequestInit = {}) {
    const timestamp = new Date().toISOString();
    const method = options.method || 'GET';
    const body = options.body?.toString() || '';
    
    // Create signature
    const stringToSign = `${method}\n${url}\n${timestamp}\n${body}`;
    const signature = crypto
      .createHmac('sha256', this.secretKey)
      .update(stringToSign)
      .digest('hex');
    
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'X-Access-Key': this.accessKey,
        'X-Timestamp': timestamp,
        'X-Signature': signature,
      },
    });
  }
}
```

## 15.2 Circuit Breaker Pattern

```typescript
enum CircuitState {
  CLOSED = 'CLOSED',
  OPEN = 'OPEN',
  HALF_OPEN = 'HALF_OPEN',
}

class CircuitBreaker {
  private state = CircuitState.CLOSED;
  private failures = 0;
  private lastFailure = 0;
  private successesInHalfOpen = 0;
  
  constructor(
    private threshold = 5,
    private timeout = 30000,
    private successThreshold = 2
  ) {}
  
  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() - this.lastFailure > this.timeout) {
        this.state = CircuitState.HALF_OPEN;
        this.successesInHalfOpen = 0;
      } else {
        throw new Error('Circuit is OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess() {
    if (this.state === CircuitState.HALF_OPEN) {
      this.successesInHalfOpen++;
      if (this.successesInHalfOpen >= this.successThreshold) {
        this.state = CircuitState.CLOSED;
        this.failures = 0;
      }
    } else {
      this.failures = 0;
    }
  }
  
  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
    
    if (this.failures >= this.threshold) {
      this.state = CircuitState.OPEN;
    }
  }
  
  getState() {
    return this.state;
  }
}

// Usage
const breaker = new CircuitBreaker(5, 30000);

async function fetchWithCircuitBreaker(url: string) {
  return breaker.call(() => fetch(url).then(r => r.json()));
}
```

## 15.3 Request Tracing

```typescript
// Distributed tracing for debugging
class RequestTracer {
  private traces = new Map<string, {
    traceId: string;
    spans: Array<{
      name: string;
      start: number;
      end?: number;
      metadata?: Record<string, any>;
    }>;
  }>();
  
  startTrace(): string {
    const traceId = crypto.randomUUID();
    this.traces.set(traceId, {
      traceId,
      spans: [],
    });
    return traceId;
  }
  
  startSpan(traceId: string, name: string) {
    const trace = this.traces.get(traceId);
    if (trace) {
      trace.spans.push({
        name,
        start: performance.now(),
      });
    }
  }
  
  endSpan(traceId: string, metadata?: Record<string, any>) {
    const trace = this.traces.get(traceId);
    if (trace && trace.spans.length > 0) {
      const span = trace.spans[trace.spans.length - 1];
      span.end = performance.now();
      span.metadata = metadata;
    }
  }
  
  getTrace(traceId: string) {
    return this.traces.get(traceId);
  }
  
  logTrace(traceId: string) {
    const trace = this.traces.get(traceId);
    if (!trace) return;
    
    console.group(`Trace: ${traceId}`);
    trace.spans.forEach((span) => {
      const duration = span.end ? span.end - span.start : 'ongoing';
      console.log(`${span.name}: ${duration}ms`, span.metadata);
    });
    console.groupEnd();
  }
}

// Usage with fetch
const tracer = new RequestTracer();

async function tracedFetch(url: string) {
  const traceId = tracer.startTrace();
  
  tracer.startSpan(traceId, 'fetch');
  const response = await fetch(url, {
    headers: {
      'X-Trace-Id': traceId,
    },
  });
  tracer.endSpan(traceId, { status: response.status });
  
  tracer.startSpan(traceId, 'parse');
  const data = await response.json();
  tracer.endSpan(traceId, { size: JSON.stringify(data).length });
  
  tracer.logTrace(traceId);
  
  return data;
}
```

---

# 16. API Testing Patterns

## 16.1 Integration Testing

```typescript
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' },
      ])
    );
  }),
  
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    return res(
      ctx.json({ id, name: `User ${id}` })
    );
  }),
  
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json();
    return res(
      ctx.status(201),
      ctx.json({ id: 3, ...body })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  
  return render(
    <QueryClientProvider client={queryClient}>
      {ui}
    </QueryClientProvider>
  );
}

test('displays users after loading', async () => {
  renderWithProviders(<UserList />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
    expect(screen.getByText('Jane')).toBeInTheDocument();
  });
});

test('handles error state', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  renderWithProviders(<UserList />);
  
  await waitFor(() => {
    expect(screen.getByText(/error/i)).toBeInTheDocument();
  });
});
```

## 16.2 Unit Testing API Functions

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('API Client', () => {
  const mockFetch = vi.fn();
  global.fetch = mockFetch;
  
  beforeEach(() => {
    mockFetch.mockClear();
  });
  
  it('makes GET request correctly', async () => {
    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ data: 'test' }),
    });
    
    const result = await api.get('/test');
    
    expect(mockFetch).toHaveBeenCalledWith(
      expect.stringContaining('/test'),
      expect.objectContaining({
        method: 'GET',
      })
    );
    expect(result).toEqual({ data: 'test' });
  });
  
  it('handles 401 and refreshes token', async () => {
    // First call returns 401
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 401,
    });
    
    // Token refresh succeeds
    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ accessToken: 'new-token' }),
    });
    
    // Retry succeeds
    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ data: 'test' }),
    });
    
    const result = await api.get('/protected');
    
    expect(mockFetch).toHaveBeenCalledTimes(3);
    expect(result).toEqual({ data: 'test' });
  });
  
  it('throws on non-retryable errors', async () => {
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
      json: () => Promise.resolve({ message: 'Not found' }),
    });
    
    await expect(api.get('/missing')).rejects.toThrow('Not found');
  });
});
```

---

# 17. Performance Optimization

## 17.1 Request Coalescing

```typescript
// Combine multiple requests into one
class RequestCoalescer<T, K = string> {
  private pending = new Map<K, {
    resolve: (value: T) => void;
    reject: (error: Error) => void;
  }[]>();
  private batchFn: (keys: K[]) => Promise<Map<K, T>>;
  private timeout: NodeJS.Timeout | null = null;
  private delay: number;
  
  constructor(batchFn: (keys: K[]) => Promise<Map<K, T>>, delay = 10) {
    this.batchFn = batchFn;
    this.delay = delay;
  }
  
  async get(key: K): Promise<T> {
    return new Promise((resolve, reject) => {
      if (!this.pending.has(key)) {
        this.pending.set(key, []);
      }
      
      this.pending.get(key)!.push({ resolve, reject });
      
      if (!this.timeout) {
        this.timeout = setTimeout(() => this.flush(), this.delay);
      }
    });
  }
  
  private async flush() {
    this.timeout = null;
    const keys = Array.from(this.pending.keys());
    const callbacks = new Map(this.pending);
    this.pending.clear();
    
    try {
      const results = await this.batchFn(keys);
      
      callbacks.forEach((cbs, key) => {
        const result = results.get(key);
        if (result !== undefined) {
          cbs.forEach((cb) => cb.resolve(result));
        } else {
          cbs.forEach((cb) => cb.reject(new Error(`No result for ${key}`)));
        }
      });
    } catch (error) {
      callbacks.forEach((cbs) => {
        cbs.forEach((cb) => cb.reject(error as Error));
      });
    }
  }
}

// Usage
const userCoalescer = new RequestCoalescer<User, string>(
  async (ids) => {
    const users = await api.post('/users/batch', { ids });
    return new Map(users.map((u: User) => [u.id, u]));
  }
);

// These all get batched into one request
const [user1, user2, user3] = await Promise.all([
  userCoalescer.get('1'),
  userCoalescer.get('2'),
  userCoalescer.get('3'),
]);
```

## 17.2 Prefetching Strategies

```typescript
// Link prefetching on hover
function PrefetchLink({
  href,
  queryKey,
  prefetchFn,
  children,
}: {
  href: string;
  queryKey: string[];
  prefetchFn: () => Promise<any>;
  children: React.ReactNode;
}) {
  const queryClient = useQueryClient();
  
  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey,
      queryFn: prefetchFn,
      staleTime: 60000,
    });
  };
  
  return (
    <Link 
      href={href} 
      onMouseEnter={handleMouseEnter}
    >
      {children}
    </Link>
  );
}

// Usage
<PrefetchLink
  href="/users/123"
  queryKey={['user', '123']}
  prefetchFn={() => userService.getById('123')}
>
  View User
</PrefetchLink>

// Route-based prefetching
function usePrefetchOnRoute() {
  const router = useRouter();
  const queryClient = useQueryClient();
  
  useEffect(() => {
    const prefetchMap: Record<string, () => void> = {
      '/dashboard': () => {
        queryClient.prefetchQuery({
          queryKey: ['metrics'],
          queryFn: fetchDashboardMetrics,
        });
      },
      '/users': () => {
        queryClient.prefetchQuery({
          queryKey: ['users', { page: 1 }],
          queryFn: () => fetchUsers({ page: 1 }),
        });
      },
    };
    
    const prefetch = prefetchMap[router.pathname];
    if (prefetch) prefetch();
  }, [router.pathname, queryClient]);
}
```

## 17.3 Response Compression

```typescript
// Check for compression support
function supportsCompression(): boolean {
  return 'CompressionStream' in globalThis;
}

// Compress request body
async function compressBody(body: string): Promise<Blob> {
  const stream = new Blob([body])
    .stream()
    .pipeThrough(new CompressionStream('gzip'));
  
  return new Response(stream).blob();
}

// Decompress response
async function decompressResponse(response: Response): Promise<string> {
  const stream = response
    .body!
    .pipeThrough(new DecompressionStream('gzip'));
  
  return new Response(stream).text();
}

// Fetch with compression
async function compressedFetch(url: string, body: object) {
  const jsonBody = JSON.stringify(body);
  const compressed = await compressBody(jsonBody);
  
  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Content-Encoding': 'gzip',
    },
    body: compressed,
  });
}
```

---

# 18. Quick Reference

## 18.1 Common HTTP Headers

```typescript
const headers = {
  // Content
  'Content-Type': 'application/json',
  'Content-Length': '1234',
  'Content-Encoding': 'gzip',
  
  // Auth
  'Authorization': 'Bearer token',
  'X-API-Key': 'key',
  
  // Caching
  'Cache-Control': 'max-age=3600',
  'ETag': '"abc123"',
  'If-None-Match': '"abc123"',
  
  // CORS
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST',
  
  // Custom
  'X-Request-ID': 'uuid',
  'X-Trace-ID': 'trace-uuid',
};
```

## 18.2 Fetch Cheat Sheet

```typescript
// GET
const data = await fetch('/api/data').then(r => r.json());

// POST
const created = await fetch('/api/data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'test' }),
}).then(r => r.json());

// PUT
await fetch('/api/data/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'updated' }),
});

// DELETE
await fetch('/api/data/1', { method: 'DELETE' });

// With timeout
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);
await fetch('/api/data', { signal: controller.signal });

// With credentials
await fetch('/api/data', { credentials: 'include' });
```

## 18.3 React Query Cheat Sheet

```typescript
// Query
const { data, isLoading, error } = useQuery({
  queryKey: ['key'],
  queryFn: fetchFn,
});

// Mutation
const mutation = useMutation({
  mutationFn: createFn,
  onSuccess: () => queryClient.invalidateQueries(['key']),
});

// Prefetch
await queryClient.prefetchQuery({
  queryKey: ['key'],
  queryFn: fetchFn,
});

// Invalidate
queryClient.invalidateQueries(['key']);

// Set cache
queryClient.setQueryData(['key'], newData);

// Remove from cache
queryClient.removeQueries(['key']);
```

---

## 🎯 Complete Study Checklist

### HTTP Fundamentals
- [ ] HTTP methods (GET, POST, PUT, PATCH, DELETE)
- [ ] Status codes (2xx, 3xx, 4xx, 5xx)
- [ ] Headers (Content-Type, Authorization, Cache-Control)
- [ ] CORS and preflight requests
- [ ] HTTP/2 features

### API Clients
- [ ] Fetch API basics
- [ ] Axios interceptors
- [ ] Error handling
- [ ] Request cancellation
- [ ] File upload

### REST & GraphQL
- [ ] REST principles
- [ ] Resource-based URLs
- [ ] GraphQL queries and mutations
- [ ] Apollo Client

### Authentication
- [ ] JWT structure
- [ ] Token refresh
- [ ] OAuth 2.0 flow
- [ ] Session handling

### Caching
- [ ] HTTP caching (Cache-Control, ETag)
- [ ] Client-side caching
- [ ] React Query cache

### Optimization
- [ ] Debouncing and throttling
- [ ] Request batching
- [ ] Retry with backoff
- [ ] Prefetching

### Advanced
- [ ] Circuit breaker pattern
- [ ] Request coalescing
- [ ] Offline handling
- [ ] WebSocket basics

---

# 19. WebSocket Implementation

## 19.1 WebSocket Client Class

```typescript
type MessageHandler = (data: any) => void;
type ConnectionHandler = () => void;
type ErrorHandler = (error: Event) => void;

interface WebSocketOptions {
  reconnect?: boolean;
  reconnectInterval?: number;
  maxReconnectAttempts?: number;
  heartbeatInterval?: number;
}

class WebSocketClient {
  private ws: WebSocket | null = null;
  private url: string;
  private options: Required<WebSocketOptions>;
  private reconnectAttempts = 0;
  private messageHandlers = new Map<string, Set<MessageHandler>>();
  private heartbeatTimer: NodeJS.Timeout | null = null;
  
  private onOpenHandlers: Set<ConnectionHandler> = new Set();
  private onCloseHandlers: Set<ConnectionHandler> = new Set();
  private onErrorHandlers: Set<ErrorHandler> = new Set();
  
  constructor(url: string, options: WebSocketOptions = {}) {
    this.url = url;
    this.options = {
      reconnect: true,
      reconnectInterval: 3000,
      maxReconnectAttempts: 10,
      heartbeatInterval: 30000,
      ...options,
    };
  }
  
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket(this.url);
      
      this.ws.onopen = () => {
        this.reconnectAttempts = 0;
        this.startHeartbeat();
        this.onOpenHandlers.forEach((handler) => handler());
        resolve();
      };
      
      this.ws.onmessage = (event) => {
        try {
          const message = JSON.parse(event.data);
          const { type, payload } = message;
          
          const handlers = this.messageHandlers.get(type);
          if (handlers) {
            handlers.forEach((handler) => handler(payload));
          }
        } catch (error) {
          console.error('Failed to parse message:', error);
        }
      };
      
      this.ws.onclose = () => {
        this.stopHeartbeat();
        this.onCloseHandlers.forEach((handler) => handler());
        
        if (this.options.reconnect && 
            this.reconnectAttempts < this.options.maxReconnectAttempts) {
          this.scheduleReconnect();
        }
      };
      
      this.ws.onerror = (error) => {
        this.onErrorHandlers.forEach((handler) => handler(error));
        reject(error);
      };
    });
  }
  
  disconnect() {
    this.options.reconnect = false;
    this.stopHeartbeat();
    this.ws?.close();
    this.ws = null;
  }
  
  send(type: string, payload: any) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type, payload }));
    } else {
      throw new Error('WebSocket is not connected');
    }
  }
  
  on(type: string, handler: MessageHandler) {
    if (!this.messageHandlers.has(type)) {
      this.messageHandlers.set(type, new Set());
    }
    this.messageHandlers.get(type)!.add(handler);
    
    return () => {
      this.messageHandlers.get(type)?.delete(handler);
    };
  }
  
  onOpen(handler: ConnectionHandler) {
    this.onOpenHandlers.add(handler);
    return () => this.onOpenHandlers.delete(handler);
  }
  
  onClose(handler: ConnectionHandler) {
    this.onCloseHandlers.add(handler);
    return () => this.onCloseHandlers.delete(handler);
  }
  
  onError(handler: ErrorHandler) {
    this.onErrorHandlers.add(handler);
    return () => this.onErrorHandlers.delete(handler);
  }
  
  private scheduleReconnect() {
    this.reconnectAttempts++;
    const delay = this.options.reconnectInterval * this.reconnectAttempts;
    
    setTimeout(() => {
      console.log(`Reconnecting... (attempt ${this.reconnectAttempts})`);
      this.connect().catch(() => {});
    }, delay);
  }
  
  private startHeartbeat() {
    this.heartbeatTimer = setInterval(() => {
      this.send('ping', {});
    }, this.options.heartbeatInterval);
  }
  
  private stopHeartbeat() {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }
  }
}

// Usage
const ws = new WebSocketClient('wss://api.example.com/ws');

ws.onOpen(() => console.log('Connected'));
ws.onClose(() => console.log('Disconnected'));

ws.on('message', (data) => {
  console.log('New message:', data);
});

ws.on('notification', (data) => {
  toast.info(data.message);
});

await ws.connect();
ws.send('subscribe', { channel: 'updates' });
```

## 19.2 React WebSocket Hook

```typescript
import { useEffect, useCallback, useRef, useState } from 'react';

interface UseWebSocketOptions {
  onMessage?: (data: any) => void;
  onOpen?: () => void;
  onClose?: () => void;
  onError?: (error: Event) => void;
  reconnect?: boolean;
  reconnectInterval?: number;
}

function useWebSocket(url: string, options: UseWebSocketOptions = {}) {
  const [isConnected, setIsConnected] = useState(false);
  const [lastMessage, setLastMessage] = useState<any>(null);
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timeout>();
  
  const {
    onMessage,
    onOpen,
    onClose,
    onError,
    reconnect = true,
    reconnectInterval = 3000,
  } = options;
  
  const connect = useCallback(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;
    
    ws.onopen = () => {
      setIsConnected(true);
      onOpen?.();
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setLastMessage(data);
      onMessage?.(data);
    };
    
    ws.onclose = () => {
      setIsConnected(false);
      onClose?.();
      
      if (reconnect) {
        reconnectTimeoutRef.current = setTimeout(connect, reconnectInterval);
      }
    };
    
    ws.onerror = (error) => {
      onError?.(error);
    };
  }, [url, onMessage, onOpen, onClose, onError, reconnect, reconnectInterval]);
  
  const disconnect = useCallback(() => {
    if (reconnectTimeoutRef.current) {
      clearTimeout(reconnectTimeoutRef.current);
    }
    wsRef.current?.close();
  }, []);
  
  const send = useCallback((data: any) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(data));
    }
  }, []);
  
  useEffect(() => {
    connect();
    return () => {
      disconnect();
    };
  }, [connect, disconnect]);
  
  return {
    isConnected,
    lastMessage,
    send,
    disconnect,
    reconnect: connect,
  };
}

// Usage
function ChatRoom({ roomId }: { roomId: string }) {
  const [messages, setMessages] = useState<Message[]>([]);
  
  const { isConnected, send } = useWebSocket(
    `wss://api.example.com/chat/${roomId}`,
    {
      onMessage: (data) => {
        if (data.type === 'message') {
          setMessages((prev) => [...prev, data.payload]);
        }
      },
      onOpen: () => {
        console.log('Connected to chat room');
      },
    }
  );
  
  const sendMessage = (text: string) => {
    send({ type: 'message', payload: { text } });
  };
  
  return (
    <div>
      <span>{isConnected ? '🟢' : '🔴'}</span>
      <MessageList messages={messages} />
      <MessageInput onSend={sendMessage} />
    </div>
  );
}
```

## 19.3 WebSocket with Zustand

```typescript
import { create } from 'zustand';

interface Message {
  id: string;
  text: string;
  sender: string;
  timestamp: Date;
}

interface ChatStore {
  socket: WebSocket | null;
  isConnected: boolean;
  messages: Message[];
  
  connect: (url: string) => void;
  disconnect: () => void;
  sendMessage: (text: string) => void;
  clearMessages: () => void;
}

export const useChatStore = create<ChatStore>((set, get) => ({
  socket: null,
  isConnected: false,
  messages: [],
  
  connect: (url) => {
    const socket = new WebSocket(url);
    
    socket.onopen = () => {
      set({ socket, isConnected: true });
    };
    
    socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'message') {
        set((state) => ({
          messages: [...state.messages, data.payload],
        }));
      }
    };
    
    socket.onclose = () => {
      set({ socket: null, isConnected: false });
    };
    
    set({ socket });
  },
  
  disconnect: () => {
    const { socket } = get();
    socket?.close();
    set({ socket: null, isConnected: false });
  },
  
  sendMessage: (text) => {
    const { socket, isConnected } = get();
    
    if (socket && isConnected) {
      socket.send(JSON.stringify({
        type: 'message',
        payload: { text },
      }));
    }
  },
  
  clearMessages: () => set({ messages: [] }),
}));
```

---

# 20. More Real-World Patterns

## 20.1 API Gateway Pattern

```typescript
// Centralized API gateway for multiple services
class APIGateway {
  private services = new Map<string, string>();
  private defaultHeaders: Record<string, string> = {};
  
  registerService(name: string, baseUrl: string) {
    this.services.set(name, baseUrl);
  }
  
  setDefaultHeaders(headers: Record<string, string>) {
    this.defaultHeaders = { ...this.defaultHeaders, ...headers };
  }
  
  async request<T>(
    service: string,
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const baseUrl = this.services.get(service);
    if (!baseUrl) {
      throw new Error(`Unknown service: ${service}`);
    }
    
    const url = `${baseUrl}${endpoint}`;
    
    const response = await fetch(url, {
      ...options,
      headers: {
        ...this.defaultHeaders,
        ...options.headers,
      },
    });
    
    if (!response.ok) {
      throw new APIError(response.status, await response.text());
    }
    
    return response.json();
  }
  
  // Convenience methods
  get<T>(service: string, endpoint: string) {
    return this.request<T>(service, endpoint, { method: 'GET' });
  }
  
  post<T>(service: string, endpoint: string, data: unknown) {
    return this.request<T>(service, endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
  }
}

// Usage
const gateway = new APIGateway();

gateway.registerService('users', 'https://users.api.example.com');
gateway.registerService('orders', 'https://orders.api.example.com');
gateway.registerService('products', 'https://products.api.example.com');

gateway.setDefaultHeaders({
  'X-API-Key': 'your-api-key',
});

// Make requests to different services
const user = await gateway.get<User>('users', '/users/123');
const orders = await gateway.get<Order[]>('orders', '/orders?userId=123');
const product = await gateway.get<Product>('products', '/products/456');
```

## 20.2 Request Queue with Priority

```typescript
interface QueuedRequest<T> {
  id: string;
  priority: number;
  execute: () => Promise<T>;
  resolve: (value: T) => void;
  reject: (error: Error) => void;
}

class PriorityRequestQueue {
  private queue: QueuedRequest<any>[] = [];
  private running = 0;
  private maxConcurrent: number;
  
  constructor(maxConcurrent = 3) {
    this.maxConcurrent = maxConcurrent;
  }
  
  async add<T>(
    execute: () => Promise<T>,
    priority = 0
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      const request: QueuedRequest<T> = {
        id: crypto.randomUUID(),
        priority,
        execute,
        resolve,
        reject,
      };
      
      // Insert in priority order
      const index = this.queue.findIndex((r) => r.priority < priority);
      if (index === -1) {
        this.queue.push(request);
      } else {
        this.queue.splice(index, 0, request);
      }
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
    while (this.running < this.maxConcurrent && this.queue.length > 0) {
      const request = this.queue.shift()!;
      this.running++;
      
      request
        .execute()
        .then(request.resolve)
        .catch(request.reject)
        .finally(() => {
          this.running--;
          this.processQueue();
        });
    }
  }
}

// Usage
const queue = new PriorityRequestQueue(3);

// High priority request
queue.add(() => fetchCriticalData(), 10);

// Normal priority
queue.add(() => fetchData(), 5);

// Low priority (background)
queue.add(() => fetchAnalytics(), 1);
```

## 20.3 Request Middleware Chain

```typescript
type NextFunction = () => Promise<Response>;
type Middleware = (
  request: Request,
  next: NextFunction
) => Promise<Response>;

class MiddlewareChain {
  private middlewares: Middleware[] = [];
  
  use(middleware: Middleware) {
    this.middlewares.push(middleware);
    return this;
  }
  
  async execute(request: Request): Promise<Response> {
    const chain = this.middlewares.reduceRight<NextFunction>(
      (next, middleware) => {
        return () => middleware(request, next);
      },
      () => fetch(request)
    );
    
    return chain();
  }
}

// Auth middleware
const authMiddleware: Middleware = async (request, next) => {
  const token = localStorage.getItem('token');
  if (token) {
    request.headers.set('Authorization', `Bearer ${token}`);
  }
  return next();
};

// Logging middleware
const loggingMiddleware: Middleware = async (request, next) => {
  const start = performance.now();
  console.log(`[Request] ${request.method} ${request.url}`);
  
  const response = await next();
  
  const duration = performance.now() - start;
  console.log(`[Response] ${response.status} (${duration.toFixed(0)}ms)`);
  
  return response;
};

// Retry middleware
const retryMiddleware: Middleware = async (request, next) => {
  let lastError: Error;
  
  for (let i = 0; i < 3; i++) {
    try {
      return await next();
    } catch (error) {
      lastError = error as Error;
      await new Promise((r) => setTimeout(r, 1000 * (i + 1)));
    }
  }
  
  throw lastError!;
};

// Usage
const chain = new MiddlewareChain()
  .use(loggingMiddleware)
  .use(authMiddleware)
  .use(retryMiddleware);

const response = await chain.execute(
  new Request('https://api.example.com/data')
);
```

## 20.4 Conditional Request Pattern

```typescript
// Only fetch if data might have changed
async function conditionalFetch<T>(
  url: string,
  cachedData: { data: T; etag?: string; lastModified?: string } | null
): Promise<{ data: T; etag?: string; lastModified?: string; fromCache: boolean }> {
  const headers: HeadersInit = {};
  
  if (cachedData?.etag) {
    headers['If-None-Match'] = cachedData.etag;
  }
  
  if (cachedData?.lastModified) {
    headers['If-Modified-Since'] = cachedData.lastModified;
  }
  
  const response = await fetch(url, { headers });
  
  if (response.status === 304 && cachedData) {
    return { ...cachedData, fromCache: true };
  }
  
  const data = await response.json();
  
  return {
    data,
    etag: response.headers.get('ETag') || undefined,
    lastModified: response.headers.get('Last-Modified') || undefined,
    fromCache: false,
  };
}

// Usage with React Query
function useConditionalQuery<T>(url: string) {
  const cacheRef = useRef<{ data: T; etag?: string; lastModified?: string } | null>(null);
  
  return useQuery({
    queryKey: [url],
    queryFn: async () => {
      const result = await conditionalFetch<T>(url, cacheRef.current);
      cacheRef.current = result;
      return result.data;
    },
  });
}
```

---

# 21. Streaming and Large Data

## 21.1 Streaming Response

```typescript
// Read streaming response
async function fetchStream(url: string) {
  const response = await fetch(url);
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  
  let result = '';
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    const chunk = decoder.decode(value, { stream: true });
    result += chunk;
    
    // Process chunk (e.g., for SSE or NDJSON)
    console.log('Received chunk:', chunk);
  }
  
  return result;
}

// Stream with progress
async function* streamWithProgress(
  url: string
): AsyncGenerator<{ chunk: string; progress: number }> {
  const response = await fetch(url);
  const contentLength = parseInt(
    response.headers.get('Content-Length') || '0',
    10
  );
  
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  let received = 0;
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    received += value.length;
    const chunk = decoder.decode(value, { stream: true });
    const progress = contentLength > 0 ? (received / contentLength) * 100 : 0;
    
    yield { chunk, progress };
  }
}

// Usage
for await (const { chunk, progress } of streamWithProgress('/large-file')) {
  console.log(`Progress: ${progress.toFixed(1)}%`);
  processChunk(chunk);
}
```

## 21.2 NDJSON (Newline Delimited JSON)

```typescript
// Parse NDJSON stream
async function* parseNDJSON<T>(url: string): AsyncGenerator<T> {
  const response = await fetch(url);
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  
  let buffer = '';
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    buffer += decoder.decode(value, { stream: true });
    
    const lines = buffer.split('\n');
    buffer = lines.pop()!; // Keep incomplete line in buffer
    
    for (const line of lines) {
      if (line.trim()) {
        yield JSON.parse(line);
      }
    }
  }
  
  // Process remaining buffer
  if (buffer.trim()) {
    yield JSON.parse(buffer);
  }
}

// Usage
for await (const item of parseNDJSON<LogEntry>('/api/logs/stream')) {
  console.log('Log entry:', item);
}
```

## 21.3 Chunked Upload

```typescript
async function uploadInChunks(
  file: File,
  chunkSize = 1024 * 1024, // 1MB
  onProgress?: (progress: number) => void
) {
  const totalChunks = Math.ceil(file.size / chunkSize);
  const uploadId = crypto.randomUUID();
  
  for (let i = 0; i < totalChunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);
    
    const formData = new FormData();
    formData.append('chunk', chunk);
    formData.append('uploadId', uploadId);
    formData.append('chunkIndex', String(i));
    formData.append('totalChunks', String(totalChunks));
    
    await fetch('/api/upload/chunk', {
      method: 'POST',
      body: formData,
    });
    
    onProgress?.((i + 1) / totalChunks * 100);
  }
  
  // Complete upload
  const response = await fetch('/api/upload/complete', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ uploadId }),
  });
  
  return response.json();
}
```

---

# 22. More Interview Q&A

## Q16: How do you handle long-polling?

**Answer:**

```typescript
async function longPoll<T>(
  url: string,
  onData: (data: T) => void,
  options: { timeout?: number; signal?: AbortSignal } = {}
): Promise<void> {
  const { timeout = 30000, signal } = options;
  
  while (!signal?.aborted) {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), timeout);
      
      const response = await fetch(url, {
        signal: controller.signal,
      });
      
      clearTimeout(timeoutId);
      
      if (response.ok) {
        const data = await response.json();
        onData(data);
      }
    } catch (error) {
      if ((error as Error).name !== 'AbortError') {
        console.error('Long poll error:', error);
        await new Promise((r) => setTimeout(r, 1000));
      }
    }
  }
}

// Usage
const controller = new AbortController();

longPoll<Message>(
  '/api/messages/poll',
  (message) => {
    console.log('New message:', message);
  },
  { signal: controller.signal }
);

// Stop polling
controller.abort();
```

---

## Q17: How do you implement request idempotency?

**Answer:**

```typescript
// Client-side idempotency key
async function idempotentPost<T>(
  url: string,
  data: unknown,
  idempotencyKey?: string
): Promise<T> {
  const key = idempotencyKey || crypto.randomUUID();
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Idempotency-Key': key,
    },
    body: JSON.stringify(data),
  });
  
  return response.json();
}

// With retry - same key means same operation
async function idempotentPostWithRetry<T>(
  url: string,
  data: unknown,
  maxRetries = 3
): Promise<T> {
  const idempotencyKey = crypto.randomUUID();
  
  for (let i = 0; i <= maxRetries; i++) {
    try {
      return await idempotentPost<T>(url, data, idempotencyKey);
    } catch (error) {
      if (i === maxRetries) throw error;
      await new Promise((r) => setTimeout(r, 1000 * (i + 1)));
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

---

## Q18: How do you handle API versioning?

**Answer:**

```typescript
// URL versioning
const apiV1 = new APIClient('https://api.example.com/v1');
const apiV2 = new APIClient('https://api.example.com/v2');

// Header versioning
class VersionedAPI {
  constructor(private version: string) {}
  
  async request<T>(url: string, options: RequestInit = {}): Promise<T> {
    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Accept': `application/vnd.api+json; version=${this.version}`,
        // or
        'X-API-Version': this.version,
      },
    });
    
    return response.json();
  }
}

// Version detection and migration
async function detectAndMigrate() {
  const response = await fetch('/api/version');
  const { version, deprecated, sunset } = await response.json();
  
  if (deprecated) {
    console.warn(`API version ${version} is deprecated. Sunset: ${sunset}`);
  }
  
  if (version !== CURRENT_VERSION) {
    // Handle version mismatch
  }
}
```

---

## Q19: How do you handle concurrent requests to the same resource?

**Answer:**

```typescript
// Mutex for single resource
class AsyncMutex {
  private locked = false;
  private waiters: (() => void)[] = [];
  
  async acquire(): Promise<() => void> {
    while (this.locked) {
      await new Promise<void>((resolve) => {
        this.waiters.push(resolve);
      });
    }
    
    this.locked = true;
    
    return () => {
      this.locked = false;
      const next = this.waiters.shift();
      next?.();
    };
  }
}

// Resource-specific locks
class ResourceLocks {
  private locks = new Map<string, AsyncMutex>();
  
  async withLock<T>(resourceId: string, fn: () => Promise<T>): Promise<T> {
    if (!this.locks.has(resourceId)) {
      this.locks.set(resourceId, new AsyncMutex());
    }
    
    const release = await this.locks.get(resourceId)!.acquire();
    
    try {
      return await fn();
    } finally {
      release();
    }
  }
}

// Usage
const locks = new ResourceLocks();

// These will execute one at a time for user-123
await Promise.all([
  locks.withLock('user-123', () => updateUser('123', { name: 'A' })),
  locks.withLock('user-123', () => updateUser('123', { name: 'B' })),
  locks.withLock('user-123', () => updateUser('123', { name: 'C' })),
]);
```

---

## Q20: How do you implement request/response logging?

**Answer:**

```typescript
interface RequestLog {
  id: string;
  method: string;
  url: string;
  headers: Record<string, string>;
  body?: string;
  timestamp: Date;
}

interface ResponseLog {
  requestId: string;
  status: number;
  headers: Record<string, string>;
  body?: string;
  duration: number;
  timestamp: Date;
}

class RequestLogger {
  private logs: Array<{ request: RequestLog; response?: ResponseLog }> = [];
  
  async loggedFetch(
    input: RequestInfo | URL,
    init?: RequestInit
  ): Promise<Response> {
    const requestId = crypto.randomUUID();
    const startTime = performance.now();
    
    const url = typeof input === 'string' ? input : input.toString();
    
    const requestLog: RequestLog = {
      id: requestId,
      method: init?.method || 'GET',
      url,
      headers: Object.fromEntries(new Headers(init?.headers).entries()),
      body: init?.body?.toString(),
      timestamp: new Date(),
    };
    
    this.logs.push({ request: requestLog });
    
    try {
      const response = await fetch(input, init);
      
      const responseLog: ResponseLog = {
        requestId,
        status: response.status,
        headers: Object.fromEntries(response.headers.entries()),
        duration: performance.now() - startTime,
        timestamp: new Date(),
      };
      
      const log = this.logs.find((l) => l.request.id === requestId);
      if (log) {
        log.response = responseLog;
      }
      
      this.printLog(requestLog, responseLog);
      
      return response;
    } catch (error) {
      console.error(`[${requestId}] Request failed:`, error);
      throw error;
    }
  }
  
  private printLog(request: RequestLog, response: ResponseLog) {
    console.group(`[${request.id}] ${request.method} ${request.url}`);
    console.log('Request:', request);
    console.log('Response:', response);
    console.log(`Duration: ${response.duration.toFixed(0)}ms`);
    console.groupEnd();
  }
  
  getLogs() {
    return this.logs;
  }
  
  exportLogs() {
    return JSON.stringify(this.logs, null, 2);
  }
}
```

---

# 23. Security Considerations

## 23.1 Secure API Calls

```typescript
// CSRF protection
async function fetchWithCSRF(url: string, options: RequestInit = {}) {
  const csrfToken = document.querySelector<HTMLMetaElement>(
    'meta[name="csrf-token"]'
  )?.content;
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken || '',
    },
  });
}

// Sanitize response data
function sanitizeResponse<T>(data: T): T {
  if (typeof data === 'string') {
    // Remove potential XSS
    return data.replace(/<[^>]*>/g, '') as T;
  }
  
  if (Array.isArray(data)) {
    return data.map(sanitizeResponse) as T;
  }
  
  if (typeof data === 'object' && data !== null) {
    const sanitized: Record<string, unknown> = {};
    for (const [key, value] of Object.entries(data)) {
      sanitized[key] = sanitizeResponse(value);
    }
    return sanitized as T;
  }
  
  return data;
}

// Secure headers
const secureHeaders = {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Content-Security-Policy': "default-src 'self'",
};
```

## 23.2 Request Validation

```typescript
import { z } from 'zod';

// Validate API responses
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().datetime(),
});

async function fetchUserValidated(id: string) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  
  // Validate response matches expected schema
  const result = UserSchema.safeParse(data);
  
  if (!result.success) {
    console.error('Invalid API response:', result.error);
    throw new Error('Invalid API response');
  }
  
  return result.data;
}

// Validate request parameters
function validateRequestParams(params: unknown) {
  const schema = z.object({
    page: z.number().int().positive().optional(),
    limit: z.number().int().min(1).max(100).optional(),
    sort: z.enum(['asc', 'desc']).optional(),
  });
  
  return schema.parse(params);
}
```

---

# 24. Complete API Client Example

```typescript
import { z } from 'zod';

// Types
interface RequestConfig {
  timeout?: number;
  retry?: number;
  retryDelay?: number;
  cache?: boolean;
  cacheTTL?: number;
}

interface APIClientConfig {
  baseURL: string;
  defaultHeaders?: Record<string, string>;
  defaultConfig?: RequestConfig;
  onRequest?: (request: Request) => Request | Promise<Request>;
  onResponse?: (response: Response) => Response | Promise<Response>;
  onError?: (error: Error) => void;
}

// Complete production-ready API client
class ProductionAPIClient {
  private config: APIClientConfig;
  private cache = new Map<string, { data: unknown; expiry: number }>();
  
  constructor(config: APIClientConfig) {
    this.config = {
      defaultConfig: {
        timeout: 30000,
        retry: 3,
        retryDelay: 1000,
        cache: false,
        cacheTTL: 60000,
      },
      ...config,
    };
  }
  
  private async request<T>(
    method: string,
    endpoint: string,
    body?: unknown,
    requestConfig: RequestConfig = {}
  ): Promise<T> {
    const config = { ...this.config.defaultConfig, ...requestConfig };
    const url = `${this.config.baseURL}${endpoint}`;
    
    // Check cache
    if (config.cache && method === 'GET') {
      const cached = this.getFromCache<T>(url);
      if (cached) return cached;
    }
    
    // Build request
    let request = new Request(url, {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...this.config.defaultHeaders,
      },
      body: body ? JSON.stringify(body) : undefined,
    });
    
    // Apply request hook
    if (this.config.onRequest) {
      request = await this.config.onRequest(request);
    }
    
    // Execute with retry
    const response = await this.executeWithRetry(
      request,
      config.retry!,
      config.retryDelay!,
      config.timeout!
    );
    
    // Apply response hook
    let finalResponse = response;
    if (this.config.onResponse) {
      finalResponse = await this.config.onResponse(response);
    }
    
    // Handle errors
    if (!finalResponse.ok) {
      const error = await this.parseError(finalResponse);
      this.config.onError?.(error);
      throw error;
    }
    
    // Parse response
    const data = await finalResponse.json();
    
    // Cache if enabled
    if (config.cache && method === 'GET') {
      this.setCache(url, data, config.cacheTTL!);
    }
    
    return data;
  }
  
  private async executeWithRetry(
    request: Request,
    retries: number,
    delay: number,
    timeout: number
  ): Promise<Response> {
    let lastError: Error;
    
    for (let attempt = 0; attempt <= retries; attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), timeout);
        
        const response = await fetch(request.clone(), {
          signal: controller.signal,
        });
        
        clearTimeout(timeoutId);
        return response;
      } catch (error) {
        lastError = error as Error;
        
        if (attempt < retries) {
          await new Promise((r) => setTimeout(r, delay * (attempt + 1)));
        }
      }
    }
    
    throw lastError!;
  }
  
  private async parseError(response: Response): Promise<APIError> {
    try {
      const data = await response.json();
      return new APIError(response.status, data.message || 'Request failed', data);
    } catch {
      return new APIError(response.status, 'Request failed');
    }
  }
  
  private getFromCache<T>(key: string): T | null {
    const entry = this.cache.get(key);
    if (!entry) return null;
    
    if (Date.now() > entry.expiry) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data as T;
  }
  
  private setCache(key: string, data: unknown, ttl: number) {
    this.cache.set(key, {
      data,
      expiry: Date.now() + ttl,
    });
  }
  
  // Public methods
  get<T>(endpoint: string, config?: RequestConfig) {
    return this.request<T>('GET', endpoint, undefined, config);
  }
  
  post<T>(endpoint: string, body: unknown, config?: RequestConfig) {
    return this.request<T>('POST', endpoint, body, config);
  }
  
  put<T>(endpoint: string, body: unknown, config?: RequestConfig) {
    return this.request<T>('PUT', endpoint, body, config);
  }
  
  patch<T>(endpoint: string, body: unknown, config?: RequestConfig) {
    return this.request<T>('PATCH', endpoint, body, config);
  }
  
  delete<T>(endpoint: string, config?: RequestConfig) {
    return this.request<T>('DELETE', endpoint, undefined, config);
  }
  
  clearCache() {
    this.cache.clear();
  }
}

// Usage
const api = new ProductionAPIClient({
  baseURL: 'https://api.example.com',
  defaultHeaders: {
    'X-API-Version': '2',
  },
  onRequest: async (request) => {
    const token = await getAuthToken();
    request.headers.set('Authorization', `Bearer ${token}`);
    return request;
  },
  onError: (error) => {
    if (error instanceof APIError && error.status === 401) {
      redirectToLogin();
    }
  },
});

// Make requests
const users = await api.get<User[]>('/users', { cache: true });
const newUser = await api.post<User>('/users', { name: 'John' });
```

---

## 🎯 Final Complete Study Checklist

### HTTP Core
- [ ] All HTTP methods and use cases
- [ ] Status codes (100s - 500s)
- [ ] Headers deep dive
- [ ] CORS and same-origin policy
- [ ] HTTP/2 features

### API Clients
- [ ] Fetch API mastery
- [ ] Axios patterns
- [ ] Request/response interceptors
- [ ] Cancellation patterns
- [ ] File upload with progress

### REST & GraphQL
- [ ] RESTful design
- [ ] Query parameters
- [ ] GraphQL basics
- [ ] Apollo Client

### Auth
- [ ] JWT tokens
- [ ] Refresh flow
- [ ] OAuth 2.0
- [ ] CSRF protection

### Caching
- [ ] HTTP caching headers
- [ ] ETag/conditional requests
- [ ] Client-side cache
- [ ] React Query caching

### Optimization
- [ ] Debounce/throttle
- [ ] Request batching
- [ ] Retry strategies
- [ ] Prefetching
- [ ] Request coalescing

### Real-Time
- [ ] WebSocket implementation
- [ ] SSE
- [ ] Long polling
- [ ] Socket.IO basics

### Advanced
- [ ] Circuit breaker
- [ ] Request tracing
- [ ] Offline handling
- [ ] Request signing

---

# 25. Output-Based Questions

## Question 1: What will be logged?

```typescript
fetch('/api/users')
  .then(response => {
    console.log('1');
    return response.json();
  })
  .then(data => {
    console.log('2');
  })
  .catch(error => {
    console.log('3');
  })
  .finally(() => {
    console.log('4');
  });

console.log('5');
```

**Answer:** `5, 1, 2, 4` (assuming request succeeds)

**Explanation:**
- `console.log('5')` runs synchronously first
- fetch is async, so callbacks are queued
- '1' logs when response received
- '2' logs when JSON parsed
- '4' logs in finally block
- '3' only logs on error

---

## Question 2: What's wrong with this code?

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(data => setUser(data));
  
  return <div>{user?.name}</div>;
}
```

**Answer:** 

Problems:
1. **Fetch runs on every render** - no useEffect
2. **No cleanup** - can cause memory leak
3. **No loading/error states**
4. **Race condition** - stale data if userId changes

**Fixed version:**

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const controller = new AbortController();
    
    setLoading(true);
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          setLoading(false);
        }
      });
    
    return () => controller.abort();
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

---

## Question 3: What will happen?

```typescript
const response = await fetch('/api/data');
const data = await response.json();
const dataAgain = await response.json();
```

**Answer:** Error on second `response.json()` call.

**Explanation:** The response body stream can only be read once. After the first `.json()` call, the body is consumed. The second call will throw an error.

**Solution if you need to read twice:**

```typescript
const response = await fetch('/api/data');
const text = await response.text();
const data1 = JSON.parse(text);
const data2 = JSON.parse(text); // Works!
```

---

## Question 4: Why might this fail?

```typescript
fetch('https://api.example.com/data', {
  method: 'POST',
  body: JSON.stringify({ name: 'John' }),
});
```

**Answer:** Missing `Content-Type` header.

The server might not know how to parse the body without the header:

```typescript
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json', // Required!
  },
  body: JSON.stringify({ name: 'John' }),
});
```

---

## Question 5: What's the output order?

```typescript
async function fetchSequence() {
  console.log('A');
  
  const p1 = fetch('/api/1');
  const p2 = fetch('/api/2');
  
  console.log('B');
  
  await p1;
  console.log('C');
  
  await p2;
  console.log('D');
}

fetchSequence();
console.log('E');
```

**Answer:** `A, B, E, C, D`

**Explanation:**
- 'A' runs sync
- Both fetches start (no await yet)
- 'B' runs sync
- Function returns promise, 'E' runs sync
- After p1 resolves, 'C' logs
- After p2 resolves, 'D' logs

---

# 26. Common Bugs and Fixes

## Bug 1: Stale Closure in useEffect

```typescript
// ❌ Bug
function Search() {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      fetch(`/api/search?q=${query}`); // Stale!
    }, 500);
    
    return () => clearTimeout(timer);
  }, []); // Missing dependency!
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// ✅ Fix
function Search() {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      if (query) {
        fetch(`/api/search?q=${query}`);
      }
    }, 500);
    
    return () => clearTimeout(timer);
  }, [query]); // Include dependency
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

---

## Bug 2: Memory Leak on Unmount

```typescript
// ❌ Bug - memory leak
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data)); // May set state after unmount!
  }, [userId]);
  
  return <div>{user?.name}</div>;
}

// ✅ Fix - abort on unmount
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort();
  }, [userId]);
  
  return <div>{user?.name}</div>;
}
```

---

## Bug 3: Race Condition

```typescript
// ❌ Bug - race condition
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, [query]);
  
  // If user types "react" quickly:
  // Request 1: "r" (slow, takes 2s)
  // Request 2: "re" (fast, takes 0.5s)
  // Request 3: "rea" (medium, takes 1s)
  // Request 4: "reac" (fast, takes 0.3s)
  // Request 5: "react" (medium, takes 0.8s)
  // Results might show "r" results because it completes last!
  
  return <div>{/* ... */}</div>;
}

// ✅ Fix - cancel previous requests
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => setResults(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort(); // Cancel previous request
  }, [query]);
  
  return <div>{/* ... */}</div>;
}
```

---

## Bug 4: Not Checking response.ok

```typescript
// ❌ Bug - doesn't handle HTTP errors
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json(); // Parses 404 response as if it's valid!
}

// ✅ Fix - check response.ok
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return response.json();
}
```

---

# 27. Performance Tips

## 27.1 Parallel vs Sequential Requests

```typescript
// ❌ Sequential - slow
const user = await fetchUser(id);
const posts = await fetchPosts(id);
const comments = await fetchComments(id);
// Total time: user + posts + comments

// ✅ Parallel - fast
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id),
]);
// Total time: max(user, posts, comments)
```

## 27.2 Lazy Loading Data

```typescript
// Load data only when needed
function useDeletedUsers() {
  return useQuery({
    queryKey: ['deletedUsers'],
    queryFn: fetchDeletedUsers,
    enabled: false, // Don't fetch automatically
  });
}

// Usage
function AdminPanel() {
  const { data, refetch } = useDeletedUsers();
  
  return (
    <button onClick={() => refetch()}>
      View Deleted Users
    </button>
  );
}
```

## 27.3 Dependent Queries

```typescript
// Load dependent data efficiently
function useUserProfile(userId: string) {
  const userQuery = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  // Only fetch posts after user loads
  const postsQuery = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId),
    enabled: !!userQuery.data, // Wait for user
  });
  
  return {
    user: userQuery.data,
    posts: postsQuery.data,
    isLoading: userQuery.isLoading || postsQuery.isLoading,
  };
}
```

## 27.4 Request Deduplication

```typescript
// Don't make duplicate requests
const dedupeMap = new Map<string, Promise<any>>();

async function dedupedFetch<T>(url: string): Promise<T> {
  if (dedupeMap.has(url)) {
    return dedupeMap.get(url)!;
  }
  
  const promise = fetch(url)
    .then(res => res.json())
    .finally(() => {
      dedupeMap.delete(url);
    });
  
  dedupeMap.set(url, promise);
  return promise;
}
```

---

# 28. Troubleshooting Guide

## Problem: CORS Error

```
Access to fetch at 'https://api.example.com' from origin 
'http://localhost:3000' has been blocked by CORS policy
```

**Solutions:**

1. **Backend fix:** Add CORS headers
```typescript
// Express
app.use(cors({ origin: 'http://localhost:3000' }));

// Next.js
headers: {
  'Access-Control-Allow-Origin': 'http://localhost:3000',
}
```

2. **Development proxy:**
```typescript
// next.config.js
async rewrites() {
  return [
    {
      source: '/api/:path*',
      destination: 'https://api.example.com/:path*',
    },
  ];
}
```

---

## Problem: 401 Unauthorized

**Check:**
1. Token exists in storage
2. Token not expired
3. Token included in Authorization header
4. Token format correct (Bearer prefix)

```typescript
// Debug
console.log('Token:', localStorage.getItem('token'));
console.log('Headers:', request.headers.get('Authorization'));
```

---

## Problem: Request Hangs

**Check:**
1. Add timeout
2. Check network in DevTools
3. Check if server is responding
4. Check for infinite loops

```typescript
// Add timeout
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 10000);

try {
  const response = await fetch(url, { signal: controller.signal });
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Request timed out');
  }
} finally {
  clearTimeout(timeoutId);
}
```

---

## Problem: Multiple Requests on Same Action

**Common causes:**
1. Missing useEffect dependency array
2. StrictMode double-render (development only)
3. Component re-mounting
4. Multiple event listeners

```typescript
// Use AbortController to cancel duplicates
useEffect(() => {
  const controller = new AbortController();
  
  fetch(url, { signal: controller.signal });
  
  return () => controller.abort();
}, [url]);
```

---

# 29. API Design Best Practices

## 29.1 Request Naming Conventions

```typescript
// Services
class UserService {
  getAll(): Promise<User[]>
  getById(id: string): Promise<User>
  create(data: CreateUserDTO): Promise<User>
  update(id: string, data: UpdateUserDTO): Promise<User>
  delete(id: string): Promise<void>
}

// Hooks
useUsers()
useUser(id)
useCreateUser()
useUpdateUser()
useDeleteUser()
```

## 29.2 Error Response Format

```typescript
interface APIErrorResponse {
  status: number;
  code: string;
  message: string;
  details?: Record<string, string[]>;
  timestamp: string;
  requestId: string;
}

// Example
{
  "status": 422,
  "code": "VALIDATION_ERROR",
  "message": "Validation failed",
  "details": {
    "email": ["Email is required", "Email must be valid"],
    "password": ["Password must be at least 8 characters"]
  },
  "timestamp": "2024-01-15T10:30:00Z",
  "requestId": "abc-123-def"
}
```

## 29.3 Pagination Response Format

```typescript
interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPrevPage: boolean;
  };
  links: {
    self: string;
    first: string;
    last: string;
    next?: string;
    prev?: string;
  };
}
```

---

# 30. Quick Interview Answers

## One-Liners for Common Questions

1. **What is REST?**
   Representational State Transfer - stateless client-server architecture using HTTP methods for CRUD operations.

2. **What is CORS?**
   Cross-Origin Resource Sharing - browser security feature that restricts cross-origin HTTP requests.

3. **Why use Axios over Fetch?**
   Automatic JSON transformation, interceptors, request cancellation, and better browser support.

4. **What is idempotency?**
   An operation that produces the same result regardless of how many times it's executed.

5. **What are HTTP codes 2xx, 4xx, 5xx?**
   Success, Client Error, Server Error respectively.

6. **What is a JWT?**
   JSON Web Token - a self-contained token for secure transmission of user claims.

7. **What is OAuth?**
   Open Authorization - a protocol for token-based authentication and authorization.

8. **What is GraphQL advantage over REST?**
   Client specifies exactly what data it needs, eliminating over-fetching.

9. **What is WebSocket?**
   Full-duplex communication protocol for real-time bidirectional data exchange.

10. **What is ETag?**
    Entity Tag - HTTP header for cache validation using content versioning.

---

# 31. Resources

## Official Documentation
- [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Axios Documentation](https://axios-http.com/docs/intro)
- [React Query Documentation](https://tanstack.com/query)
- [Apollo Client Documentation](https://www.apollographql.com/docs/react/)
- [GraphQL Specification](https://spec.graphql.org/)

## Learning Path

### Week 1: Fundamentals
- HTTP methods and status codes
- Fetch API basics
- Request/response headers
- Error handling

### Week 2: Axios & Patterns
- Axios configuration
- Interceptors
- Request cancellation
- Retry logic

### Week 3: State Management
- React Query basics
- Caching strategies
- Optimistic updates
- Infinite queries

### Week 4: Authentication
- JWT implementation
- Token refresh flow
- OAuth integration
- Secure storage

### Week 5: Advanced
- WebSocket implementation
- GraphQL with Apollo
- Performance optimization
- Testing API calls

---

# 32. Bonus: Mock Interview Scenarios

## Scenario 1: Design a Search Autocomplete

**Q: How would you implement a search autocomplete feature?**

**Answer:**

```typescript
// 1. Debounced search hook
function useSearch(initialQuery = '') {
  const [query, setQuery] = useState(initialQuery);
  const [results, setResults] = useState<SearchResult[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  
  const debouncedSearch = useMemo(
    () => debounce(async (searchQuery: string) => {
      if (!searchQuery.trim()) {
        setResults([]);
        return;
      }
      
      setIsLoading(true);
      try {
        const response = await fetch(`/api/search?q=${encodeURIComponent(searchQuery)}`);
        const data = await response.json();
        setResults(data);
      } finally {
        setIsLoading(false);
      }
    }, 300),
    []
  );
  
  useEffect(() => {
    debouncedSearch(query);
    return () => debouncedSearch.cancel();
  }, [query]);
  
  return { query, setQuery, results, isLoading };
}

// 2. Component with keyboard navigation
function SearchAutocomplete() {
  const { query, setQuery, results, isLoading } = useSearch();
  const [selectedIndex, setSelectedIndex] = useState(-1);
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setSelectedIndex(prev => 
          Math.min(prev + 1, results.length - 1)
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setSelectedIndex(prev => Math.max(prev - 1, -1));
        break;
      case 'Enter':
        if (selectedIndex >= 0) {
          handleSelect(results[selectedIndex]);
        }
        break;
      case 'Escape':
        setSelectedIndex(-1);
        break;
    }
  };
  
  return (
    <div className="search-container">
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="Search..."
      />
      {isLoading && <Spinner />}
      {results.length > 0 && (
        <ul className="results">
          {results.map((result, index) => (
            <li
              key={result.id}
              className={index === selectedIndex ? 'selected' : ''}
              onClick={() => handleSelect(result)}
            >
              {result.name}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Key Points:**
- Debounce to reduce API calls
- Cancel previous requests on new input
- Keyboard navigation
- Loading state
- URL encoding for query

---

## Scenario 2: Implement Infinite Scroll

**Q: How would you build an infinite scroll list?**

**Answer:**

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';

interface Post {
  id: string;
  title: string;
  content: string;
}

interface PostsResponse {
  posts: Post[];
  nextCursor?: string;
  hasMore: boolean;
}

function useInfinitePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: async ({ pageParam }) => {
      const url = pageParam 
        ? `/api/posts?cursor=${pageParam}` 
        : '/api/posts';
      const response = await fetch(url);
      return response.json() as Promise<PostsResponse>;
    },
    getNextPageParam: (lastPage) => 
      lastPage.hasMore ? lastPage.nextCursor : undefined,
    initialPageParam: undefined,
  });
}

function PostList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    error,
  } = useInfinitePosts();
  
  const { ref, inView } = useInView({
    threshold: 0,
    rootMargin: '100px',
  });
  
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage]);
  
  if (isLoading) return <PageSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div className="post-list">
      {data?.pages.map((page, i) => (
        <React.Fragment key={i}>
          {page.posts.map((post) => (
            <PostCard key={post.id} post={post} />
          ))}
        </React.Fragment>
      ))}
      
      <div ref={ref} className="load-more-trigger">
        {isFetchingNextPage && <Spinner />}
        {!hasNextPage && <p>No more posts</p>}
      </div>
    </div>
  );
}
```

**Key Points:**
- Intersection Observer for trigger
- Cursor-based pagination
- React Query for caching
- Loading and error states
- Pre-fetch when user is close

---

## Scenario 3: Build a File Upload Component

**Q: How would you implement a multi-file upload with progress?**

**Answer:**

```typescript
interface UploadFile {
  id: string;
  file: File;
  progress: number;
  status: 'pending' | 'uploading' | 'complete' | 'error';
  error?: string;
}

function FileUpload() {
  const [files, setFiles] = useState<UploadFile[]>([]);
  
  const handleFileSelect = (e: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFiles = Array.from(e.target.files || []);
    
    const newFiles: UploadFile[] = selectedFiles.map((file) => ({
      id: crypto.randomUUID(),
      file,
      progress: 0,
      status: 'pending',
    }));
    
    setFiles((prev) => [...prev, ...newFiles]);
    uploadFiles(newFiles);
  };
  
  const uploadFiles = async (filesToUpload: UploadFile[]) => {
    for (const uploadFile of filesToUpload) {
      try {
        setFiles((prev) =>
          prev.map((f) =>
            f.id === uploadFile.id ? { ...f, status: 'uploading' } : f
          )
        );
        
        await uploadWithProgress(
          uploadFile.file,
          (progress) => {
            setFiles((prev) =>
              prev.map((f) =>
                f.id === uploadFile.id ? { ...f, progress } : f
              )
            );
          }
        );
        
        setFiles((prev) =>
          prev.map((f) =>
            f.id === uploadFile.id
              ? { ...f, status: 'complete', progress: 100 }
              : f
          )
        );
      } catch (error) {
        setFiles((prev) =>
          prev.map((f) =>
            f.id === uploadFile.id
              ? { ...f, status: 'error', error: (error as Error).message }
              : f
          )
        );
      }
    }
  };
  
  const uploadWithProgress = (
    file: File,
    onProgress: (progress: number) => void
  ): Promise<void> => {
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      const formData = new FormData();
      formData.append('file', file);
      
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          onProgress(Math.round((e.loaded / e.total) * 100));
        }
      });
      
      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve();
        } else {
          reject(new Error(`Upload failed: ${xhr.status}`));
        }
      });
      
      xhr.addEventListener('error', () => reject(new Error('Upload failed')));
      
      xhr.open('POST', '/api/upload');
      xhr.send(formData);
    });
  };
  
  const removeFile = (id: string) => {
    setFiles((prev) => prev.filter((f) => f.id !== id));
  };
  
  return (
    <div className="file-upload">
      <input
        type="file"
        multiple
        onChange={handleFileSelect}
        accept="image/*,application/pdf"
      />
      
      <div className="file-list">
        {files.map((file) => (
          <div key={file.id} className="file-item">
            <span>{file.file.name}</span>
            <ProgressBar progress={file.progress} />
            <span className={`status ${file.status}`}>
              {file.status}
            </span>
            <button onClick={() => removeFile(file.id)}>×</button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

# 33. More Coding Challenges

## Challenge 1: Implement Promise.allSettled

```typescript
function allSettled<T>(promises: Promise<T>[]): Promise<
  Array<{ status: 'fulfilled'; value: T } | { status: 'rejected'; reason: any }>
> {
  return Promise.all(
    promises.map((promise) =>
      promise
        .then((value) => ({ status: 'fulfilled' as const, value }))
        .catch((reason) => ({ status: 'rejected' as const, reason }))
    )
  );
}

// Usage
const results = await allSettled([
  fetch('/api/users'),
  fetch('/api/invalid'),
  fetch('/api/posts'),
]);

results.forEach((result, i) => {
  if (result.status === 'fulfilled') {
    console.log(`Promise ${i} succeeded:`, result.value);
  } else {
    console.log(`Promise ${i} failed:`, result.reason);
  }
});
```

## Challenge 2: Implement Timeout Wrapper

```typescript
function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number,
  errorMessage = 'Operation timed out'
): Promise<T> {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error(errorMessage));
    }, timeoutMs);
    
    promise
      .then((value) => {
        clearTimeout(timer);
        resolve(value);
      })
      .catch((error) => {
        clearTimeout(timer);
        reject(error);
      });
  });
}

// Usage
try {
  const data = await withTimeout(
    fetch('/api/slow-endpoint'),
    5000,
    'Request took too long'
  );
} catch (error) {
  console.error(error.message);
}
```

## Challenge 3: Implement Retry with Exponential Backoff

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  options: {
    maxRetries?: number;
    initialDelay?: number;
    maxDelay?: number;
    factor?: number;
  } = {}
): Promise<T> {
  const {
    maxRetries = 3,
    initialDelay = 1000,
    maxDelay = 30000,
    factor = 2,
  } = options;
  
  let delay = initialDelay;
  let lastError: Error;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt < maxRetries) {
        // Add jitter
        const jitter = Math.random() * 0.3 * delay;
        const waitTime = Math.min(delay + jitter, maxDelay);
        
        console.log(`Attempt ${attempt + 1} failed. Retrying in ${waitTime}ms...`);
        await new Promise((r) => setTimeout(r, waitTime));
        
        delay *= factor;
      }
    }
  }
  
  throw lastError!;
}

// Usage
const data = await retryWithBackoff(
  () => fetch('/api/flaky-endpoint').then(r => r.json()),
  { maxRetries: 5, initialDelay: 500 }
);
```

## Challenge 4: Implement Request Pool

```typescript
class RequestPool {
  private running = 0;
  private queue: Array<() => Promise<void>> = [];
  
  constructor(private maxConcurrent = 5) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise<void>((resolve) => {
        this.queue.push(async () => {
          resolve();
        });
      });
    }
    
    this.running++;
    
    try {
      return await fn();
    } finally {
      this.running--;
      
      if (this.queue.length > 0) {
        const next = this.queue.shift()!;
        next();
      }
    }
  }
  
  async map<T, R>(
    items: T[],
    fn: (item: T) => Promise<R>
  ): Promise<R[]> {
    return Promise.all(
      items.map((item) => this.execute(() => fn(item)))
    );
  }
}

// Usage
const pool = new RequestPool(3);

const urls = Array.from({ length: 20 }, (_, i) => `/api/item/${i}`);
const results = await pool.map(urls, async (url) => {
  const response = await fetch(url);
  return response.json();
});
```

## Challenge 5: Implement Smart Cache

```typescript
class SmartCache<T> {
  private cache = new Map<string, {
    data: T;
    timestamp: number;
    hits: number;
  }>();
  
  private maxSize: number;
  private ttl: number;
  
  constructor(options: { maxSize?: number; ttl?: number } = {}) {
    this.maxSize = options.maxSize || 100;
    this.ttl = options.ttl || 60000;
  }
  
  async get(key: string, fetcher: () => Promise<T>): Promise<T> {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      cached.hits++;
      return cached.data;
    }
    
    const data = await fetcher();
    this.set(key, data);
    return data;
  }
  
  private set(key: string, data: T) {
    // Evict if at capacity
    if (this.cache.size >= this.maxSize) {
      this.evictLRU();
    }
    
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
      hits: 0,
    });
  }
  
  private evictLRU() {
    let lruKey: string | null = null;
    let lruHits = Infinity;
    let lruTime = Infinity;
    
    for (const [key, value] of this.cache) {
      if (
        value.hits < lruHits ||
        (value.hits === lruHits && value.timestamp < lruTime)
      ) {
        lruKey = key;
        lruHits = value.hits;
        lruTime = value.timestamp;
      }
    }
    
    if (lruKey) {
      this.cache.delete(lruKey);
    }
  }
  
  invalidate(key: string) {
    this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
}

// Usage
const cache = new SmartCache<User>({ maxSize: 50, ttl: 300000 });

const user = await cache.get(`user-${userId}`, async () => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});
```

---

# 34. Final Tips for Interviews

## Before the Interview

1. **Review HTTP basics** - methods, status codes, headers
2. **Practice writing fetch calls** without looking at docs
3. **Understand CORS** - why it exists and how to fix issues
4. **Know the difference** between localStorage, sessionStorage, cookies
5. **Practice async/await** error handling patterns

## During the Interview

1. **Ask clarifying questions** about requirements
2. **Start with the simplest solution** then optimize
3. **Think about edge cases** - loading, errors, empty states
4. **Consider performance** - caching, debouncing, batching
5. **Explain your reasoning** as you code

## Common Mistakes to Avoid

1. **Not handling errors** in fetch calls
2. **Forgetting to check** response.ok
3. **Missing Content-Type** header in POST requests
4. **Not cancelling** requests on component unmount
5. **Over-fetching** when not needed
6. **Under-fetching** causing waterfalls
7. **Storing sensitive data** in localStorage

## Key Patterns to Know

1. **Debouncing** for search inputs
2. **AbortController** for cancellation
3. **Interceptors** for auth headers
4. **Optimistic updates** for better UX
5. **Retry logic** for resilience
6. **Request batching** for efficiency
7. **Caching** with React Query

---

*End of Part 7: Networking & API Integration Complete Guide (6000+ lines)*

*Continue to Part 8 for Security Best Practices...*

