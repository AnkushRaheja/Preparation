# 📚 Part 9: Real-Time Systems Complete Guide

> **Master WebSockets, Socket.io, SSE, and real-time patterns for frontend interviews**

---

## 📋 Table of Contents

1. [Real-Time Communication Fundamentals](#1-real-time-communication-fundamentals)
2. [WebSocket Protocol Deep Dive](#2-websocket-protocol-deep-dive)
3. [WebSocket API in Browser](#3-websocket-api-in-browser)
4. [Socket.io Complete Guide](#4-socketio-complete-guide)
5. [Server-Sent Events (SSE)](#5-server-sent-events-sse)
6. [Long Polling](#6-long-polling)
7. [Connection Management](#7-connection-management)
8. [Reconnection Strategies](#8-reconnection-strategies)
9. [Authentication in Real-Time](#9-authentication-in-real-time)
10. [React Integration Patterns](#10-react-integration-patterns)
11. [Real-Time State Management](#11-real-time-state-management)
12. [Interview Questions](#12-interview-questions)
13. [Coding Challenges](#13-coding-challenges)

---

# 1. Real-Time Communication Fundamentals

## 1.1 What is Real-Time Communication?

```typescript
// Real-time = Data delivered immediately as it's generated
// Use cases: Chat, Live updates, Gaming, Collaboration, Notifications

const REALTIME_TECHNIQUES = {
  webSocket: {
    type: 'Full-duplex',
    description: 'Persistent bidirectional connection',
    useCases: ['Chat', 'Gaming', 'Live collaboration'],
    pros: ['Low latency', 'Bidirectional', 'Efficient'],
    cons: ['Stateful', 'Load balancer complexity'],
  },
  
  serverSentEvents: {
    type: 'Server-to-client only',
    description: 'One-way stream from server',
    useCases: ['Live feeds', 'Notifications', 'Stock tickers'],
    pros: ['Simple', 'Auto-reconnect', 'HTTP compatible'],
    cons: ['One-way only', 'Limited browser connections'],
  },
  
  longPolling: {
    type: 'Simulated real-time',
    description: 'Client polls, server holds until data',
    useCases: ['Fallback', 'Simple updates'],
    pros: ['Works everywhere', 'Simple implementation'],
    cons: ['Resource intensive', 'Higher latency'],
  },
  
  webRTC: {
    type: 'Peer-to-peer',
    description: 'Direct client-to-client connection',
    useCases: ['Video calls', 'File sharing', 'Gaming'],
    pros: ['Low latency', 'No server for media'],
    cons: ['Complex setup', 'NAT traversal issues'],
  },
};
```

## 1.2 Comparison Chart

| Feature | WebSocket | SSE | Long Polling |
|---------|-----------|-----|--------------|
| Direction | Bidirectional | Server → Client | Client → Server |
| Connection | Persistent | Persistent | Repeated |
| Protocol | ws:// / wss:// | HTTP | HTTP |
| Binary Data | ✅ Yes | ❌ No (text only) | ✅ Yes |
| Auto-reconnect | ❌ Manual | ✅ Built-in | ❌ Manual |
| Browser Support | All modern | All modern | All |
| Max Connections | No limit | 6 per domain | No limit |

## 1.3 When to Use What

```typescript
function chooseRealTimeTechnique(requirements: Requirements): string {
  // Need bidirectional communication?
  if (requirements.bidirectional) {
    return 'WebSocket';
  }
  
  // Only need server-to-client updates?
  if (requirements.serverToClientOnly) {
    // Need text updates only?
    if (!requirements.binaryData) {
      return 'Server-Sent Events';
    }
    return 'WebSocket';
  }
  
  // Need peer-to-peer?
  if (requirements.peerToPeer) {
    return 'WebRTC';
  }
  
  // Simple infrequent updates?
  if (requirements.updateFrequency === 'low') {
    return 'Long Polling';
  }
  
  return 'WebSocket'; // Default for most real-time needs
}
```

---

# 2. WebSocket Protocol Deep Dive

## 2.1 How WebSocket Works

```typescript
// WebSocket connection lifecycle:
// 1. HTTP Upgrade handshake
// 2. Persistent TCP connection
// 3. Bidirectional frame exchange
// 4. Close handshake

const WEBSOCKET_LIFECYCLE = {
  handshake: {
    request: `
      GET /chat HTTP/1.1
      Host: example.com
      Upgrade: websocket
      Connection: Upgrade
      Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
      Sec-WebSocket-Version: 13
    `,
    response: `
      HTTP/1.1 101 Switching Protocols
      Upgrade: websocket
      Connection: Upgrade
      Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
    `,
  },
  
  frames: {
    text: 'UTF-8 encoded text messages',
    binary: 'ArrayBuffer or Blob data',
    ping: 'Keep-alive from server',
    pong: 'Response to ping',
    close: 'Graceful connection close',
  },
  
  closeCodesCommon: {
    1000: 'Normal closure',
    1001: 'Going away (page close)',
    1002: 'Protocol error',
    1003: 'Unsupported data',
    1006: 'Abnormal closure (no close frame)',
    1011: 'Server error',
  },
};
```

## 2.2 WebSocket vs HTTP

```typescript
const WEBSOCKET_VS_HTTP = {
  http: {
    model: 'Request-Response',
    connection: 'Short-lived',
    overhead: 'Headers on each request',
    latency: 'Higher (new connection each time)',
    serverPush: 'Not native',
  },
  
  webSocket: {
    model: 'Event-driven',
    connection: 'Persistent',
    overhead: 'Minimal (2-14 bytes per frame)',
    latency: 'Lower (connection reused)',
    serverPush: 'Native support',
  },
};

// When HTTP is better:
// - One-time requests
// - RESTful operations
// - Caching needed
// - Stateless operations

// When WebSocket is better:
// - Frequent updates
// - Low latency required
// - Bidirectional communication
// - Real-time collaboration
```

---

# 3. WebSocket API in Browser

## 3.1 Basic WebSocket Usage

```typescript
// Create connection
const ws = new WebSocket('wss://example.com/socket');

// Connection opened
ws.onopen = (event) => {
  console.log('Connected to server');
  ws.send('Hello Server!');
};

// Message received
ws.onmessage = (event) => {
  console.log('Message from server:', event.data);
  
  // Parse JSON if needed
  try {
    const data = JSON.parse(event.data);
    handleMessage(data);
  } catch {
    // Handle as plain text
    handleTextMessage(event.data);
  }
};

// Error occurred
ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

// Connection closed
ws.onclose = (event) => {
  console.log('Connection closed:', event.code, event.reason);
  
  if (event.code !== 1000) {
    // Abnormal close - maybe reconnect
    scheduleReconnect();
  }
};

// Send data
ws.send('text message');
ws.send(JSON.stringify({ type: 'message', content: 'Hello' }));
ws.send(new Blob(['binary data']));
ws.send(new ArrayBuffer(8));

// Close connection
ws.close(1000, 'Normal closure');
```

## 3.2 WebSocket States

```typescript
const WEBSOCKET_STATES = {
  CONNECTING: 0, // Connection not yet open
  OPEN: 1,       // Connection open and ready
  CLOSING: 2,    // Connection is closing
  CLOSED: 3,     // Connection is closed
};

function getConnectionState(ws: WebSocket): string {
  switch (ws.readyState) {
    case WebSocket.CONNECTING:
      return 'Connecting...';
    case WebSocket.OPEN:
      return 'Connected';
    case WebSocket.CLOSING:
      return 'Closing...';
    case WebSocket.CLOSED:
      return 'Disconnected';
    default:
      return 'Unknown';
  }
}

// Check before sending
function safeSend(ws: WebSocket, data: string | ArrayBuffer) {
  if (ws.readyState === WebSocket.OPEN) {
    ws.send(data);
    return true;
  }
  console.warn('Cannot send: WebSocket is not open');
  return false;
}
```

## 3.3 Complete WebSocket Client Class

```typescript
interface WebSocketMessage {
  type: string;
  payload: any;
  timestamp?: number;
}

type MessageHandler = (message: WebSocketMessage) => void;

class WebSocketClient {
  private ws: WebSocket | null = null;
  private url: string;
  private handlers: Map<string, Set<MessageHandler>> = new Map();
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  private reconnectDelay = 1000;
  private messageQueue: WebSocketMessage[] = [];
  private isIntentionallyClosed = false;
  
  constructor(url: string) {
    this.url = url;
  }
  
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.isIntentionallyClosed = false;
      this.ws = new WebSocket(this.url);
      
      this.ws.onopen = () => {
        console.log('WebSocket connected');
        this.reconnectAttempts = 0;
        this.flushMessageQueue();
        resolve();
      };
      
      this.ws.onmessage = (event) => {
        try {
          const message: WebSocketMessage = JSON.parse(event.data);
          this.dispatch(message);
        } catch (error) {
          console.error('Failed to parse message:', error);
        }
      };
      
      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
        reject(error);
      };
      
      this.ws.onclose = (event) => {
        console.log('WebSocket closed:', event.code, event.reason);
        
        if (!this.isIntentionallyClosed) {
          this.attemptReconnect();
        }
      };
    });
  }
  
  private attemptReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      this.dispatch({ type: 'connection:failed', payload: null });
      return;
    }
    
    const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts);
    this.reconnectAttempts++;
    
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
    
    setTimeout(() => {
      this.connect().catch(() => {
        // Will trigger another reconnect attempt
      });
    }, delay);
  }
  
  private flushMessageQueue() {
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift()!;
      this.send(message);
    }
  }
  
  private dispatch(message: WebSocketMessage) {
    const handlers = this.handlers.get(message.type);
    if (handlers) {
      handlers.forEach(handler => handler(message));
    }
    
    // Also dispatch to wildcard handlers
    const wildcardHandlers = this.handlers.get('*');
    if (wildcardHandlers) {
      wildcardHandlers.forEach(handler => handler(message));
    }
  }
  
  on(type: string, handler: MessageHandler): () => void {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, new Set());
    }
    this.handlers.get(type)!.add(handler);
    
    // Return unsubscribe function
    return () => {
      this.handlers.get(type)?.delete(handler);
    };
  }
  
  off(type: string, handler: MessageHandler) {
    this.handlers.get(type)?.delete(handler);
  }
  
  send(message: WebSocketMessage) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({
        ...message,
        timestamp: Date.now(),
      }));
    } else {
      // Queue message for later
      this.messageQueue.push(message);
    }
  }
  
  disconnect() {
    this.isIntentionallyClosed = true;
    this.ws?.close(1000, 'Client disconnect');
    this.ws = null;
  }
  
  get isConnected(): boolean {
    return this.ws?.readyState === WebSocket.OPEN;
  }
}

// Usage
const client = new WebSocketClient('wss://api.example.com/ws');

client.on('message', (msg) => {
  console.log('New message:', msg.payload);
});

client.on('notification', (msg) => {
  showNotification(msg.payload);
});

await client.connect();

client.send({
  type: 'join_room',
  payload: { roomId: 'general' },
});
```

---

# 4. Socket.io Complete Guide

## 4.1 What is Socket.io?

```typescript
// Socket.io is NOT the same as WebSocket!
// It's a library that provides:
// - Automatic reconnection
// - Fallback transports (polling → websocket)
// - Rooms and namespaces
// - Acknowledgments
// - Binary data support
// - Multiplexing

const SOCKETIO_VS_WEBSOCKET = {
  socketio: {
    reconnection: 'Built-in with exponential backoff',
    fallback: 'Automatic (polling, websocket)',
    rooms: 'Built-in room support',
    acks: 'Built-in acknowledgments',
    events: 'Custom event names',
    binary: 'Automatic handling',
  },
  
  websocket: {
    reconnection: 'Manual implementation',
    fallback: 'None (WebSocket only)',
    rooms: 'Manual implementation',
    acks: 'Manual implementation',
    events: 'Single onmessage handler',
    binary: 'Manual ArrayBuffer/Blob handling',
  },
};
```

## 4.2 Socket.io Client Setup

```typescript
import { io, Socket } from 'socket.io-client';

// Basic connection
const socket = io('https://api.example.com');

// With options
const socket = io('https://api.example.com', {
  // Connection options
  path: '/socket.io',
  transports: ['websocket', 'polling'],
  upgrade: true,
  
  // Reconnection options
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  randomizationFactor: 0.5,
  
  // Timeout options
  timeout: 20000,
  
  // Authentication
  auth: {
    token: 'your-jwt-token',
  },
  
  // Query parameters
  query: {
    userId: '123',
    version: '1.0',
  },
  
  // Auto connect (default: true)
  autoConnect: false,
});

// Manual connect
socket.connect();

// Disconnect
socket.disconnect();
```

## 4.3 Socket.io Events

```typescript
// Connection events
socket.on('connect', () => {
  console.log('Connected! Socket ID:', socket.id);
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  // Reasons: 
  // 'io server disconnect' - server called socket.disconnect()
  // 'io client disconnect' - client called socket.disconnect()
  // 'ping timeout' - server didn't respond
  // 'transport close' - connection lost
  // 'transport error' - connection error
});

socket.on('connect_error', (error) => {
  console.error('Connection error:', error);
});

socket.on('reconnect', (attemptNumber) => {
  console.log('Reconnected after', attemptNumber, 'attempts');
});

socket.on('reconnect_attempt', (attemptNumber) => {
  console.log('Reconnection attempt:', attemptNumber);
});

socket.on('reconnect_error', (error) => {
  console.error('Reconnection error:', error);
});

socket.on('reconnect_failed', () => {
  console.error('Reconnection failed');
});

// Custom events
socket.on('chat:message', (data) => {
  console.log('New message:', data);
});

socket.on('user:joined', (user) => {
  console.log('User joined:', user);
});

// Listen once
socket.once('welcome', (data) => {
  console.log('Welcome message:', data);
});

// Remove listener
const handler = (data: any) => console.log(data);
socket.on('event', handler);
socket.off('event', handler);

// Remove all listeners for event
socket.removeAllListeners('event');
```

## 4.4 Emitting Events

```typescript
// Basic emit
socket.emit('chat:message', { text: 'Hello!' });

// Emit with multiple arguments
socket.emit('chat:message', 'room1', { text: 'Hello!' }, Date.now());

// Emit with acknowledgment
socket.emit('chat:message', { text: 'Hello!' }, (response) => {
  console.log('Server acknowledged:', response);
});

// Emit with timeout
socket.timeout(5000).emit('chat:message', { text: 'Hello!' }, (err, response) => {
  if (err) {
    console.error('Timeout or error:', err);
  } else {
    console.log('Response:', response);
  }
});

// Volatile emit (may be dropped if not connected)
socket.volatile.emit('cursor:move', { x: 100, y: 200 });

// Binary data
const buffer = new ArrayBuffer(8);
socket.emit('binary:data', buffer);

// File upload
const file = new File(['content'], 'file.txt');
socket.emit('file:upload', file, (response) => {
  console.log('Upload complete:', response);
});
```

## 4.5 Rooms and Namespaces

```typescript
// Namespaces (client)
const chatSocket = io('https://api.example.com/chat');
const notifSocket = io('https://api.example.com/notifications');

// Each namespace has its own connection
chatSocket.on('connect', () => {
  console.log('Connected to chat namespace');
});

notifSocket.on('connect', () => {
  console.log('Connected to notifications namespace');
});

// Rooms (handled server-side, but client can request)
// Join room
socket.emit('room:join', 'room-123');

// Leave room
socket.emit('room:leave', 'room-123');

// Messages from room
socket.on('room:message', (data) => {
  console.log(`Message in ${data.room}:`, data.message);
});
```

## 4.6 Complete Socket.io Service

```typescript
import { io, Socket } from 'socket.io-client';

interface SocketEvents {
  'chat:message': (data: { userId: string; text: string; timestamp: number }) => void;
  'user:joined': (data: { userId: string; name: string }) => void;
  'user:left': (data: { userId: string }) => void;
  'typing:start': (data: { userId: string }) => void;
  'typing:stop': (data: { userId: string }) => void;
}

interface EmitEvents {
  'chat:send': (message: { text: string }) => void;
  'room:join': (roomId: string) => void;
  'room:leave': (roomId: string) => void;
  'typing:start': () => void;
  'typing:stop': () => void;
}

class SocketService {
  private socket: Socket | null = null;
  private token: string | null = null;
  
  setToken(token: string) {
    this.token = token;
  }
  
  connect(url: string): Promise<void> {
    return new Promise((resolve, reject) => {
      this.socket = io(url, {
        auth: { token: this.token },
        transports: ['websocket'],
        reconnection: true,
        reconnectionAttempts: 10,
        reconnectionDelay: 1000,
      });
      
      this.socket.on('connect', () => {
        console.log('Socket connected:', this.socket?.id);
        resolve();
      });
      
      this.socket.on('connect_error', (error) => {
        console.error('Socket connection error:', error);
        reject(error);
      });
      
      this.socket.on('disconnect', (reason) => {
        console.log('Socket disconnected:', reason);
      });
    });
  }
  
  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }
  
  on<K extends keyof SocketEvents>(event: K, handler: SocketEvents[K]): () => void {
    this.socket?.on(event, handler as any);
    return () => this.socket?.off(event, handler as any);
  }
  
  emit<K extends keyof EmitEvents>(
    event: K,
    ...args: Parameters<EmitEvents[K]>
  ) {
    this.socket?.emit(event, ...args);
  }
  
  emitWithAck<T>(event: string, data: any): Promise<T> {
    return new Promise((resolve, reject) => {
      this.socket?.timeout(10000).emit(event, data, (err: Error, response: T) => {
        if (err) {
          reject(err);
        } else {
          resolve(response);
        }
      });
    });
  }
  
  get isConnected(): boolean {
    return this.socket?.connected ?? false;
  }
  
  get socketId(): string | undefined {
    return this.socket?.id;
  }
}

export const socketService = new SocketService();
```

---

# 5. Server-Sent Events (SSE)

## 5.1 What is SSE?

```typescript
// SSE = One-way server-to-client stream over HTTP
// Uses text/event-stream content type
// Built-in reconnection
// Works with HTTP/2

const SSE_FEATURES = {
  direction: 'Server → Client only',
  protocol: 'HTTP (GET request)',
  format: 'Text-based (UTF-8)',
  reconnection: 'Automatic with Last-Event-ID',
  maxConnections: '6 per domain (HTTP/1.1), unlimited (HTTP/2)',
  contentType: 'text/event-stream',
};
```

## 5.2 EventSource API

```typescript
// Basic usage
const eventSource = new EventSource('/api/events');

// With credentials
const eventSource = new EventSource('/api/events', { 
  withCredentials: true 
});

// Handle messages
eventSource.onmessage = (event) => {
  console.log('Message:', event.data);
};

// Handle specific event types
eventSource.addEventListener('notification', (event) => {
  const notification = JSON.parse(event.data);
  showNotification(notification);
});

eventSource.addEventListener('update', (event) => {
  const update = JSON.parse(event.data);
  applyUpdate(update);
});

// Handle open
eventSource.onopen = (event) => {
  console.log('SSE connection opened');
};

// Handle errors
eventSource.onerror = (event) => {
  if (eventSource.readyState === EventSource.CLOSED) {
    console.log('SSE connection closed');
  } else if (eventSource.readyState === EventSource.CONNECTING) {
    console.log('SSE reconnecting...');
  }
};

// Close connection
eventSource.close();
```

## 5.3 SSE Server Format

```typescript
// Server response format:
// data: message1\n\n
// data: message2\n\n

// With event type:
// event: notification\n
// data: {"title": "Hello"}\n\n

// With ID (for reconnection):
// id: 123\n
// event: update\n
// data: {"value": 42}\n\n

// Multi-line data:
// data: line 1\n
// data: line 2\n\n

// Retry interval (ms):
// retry: 5000\n\n

// Next.js API Route example:
export async function GET(request: Request) {
  const stream = new ReadableStream({
    start(controller) {
      const encoder = new TextEncoder();
      let messageId = 0;
      
      const sendEvent = (event: string, data: any) => {
        const message = [
          `id: ${++messageId}`,
          `event: ${event}`,
          `data: ${JSON.stringify(data)}`,
          '',
          '',
        ].join('\n');
        
        controller.enqueue(encoder.encode(message));
      };
      
      // Send initial data
      sendEvent('connected', { time: Date.now() });
      
      // Send periodic updates
      const interval = setInterval(() => {
        sendEvent('ping', { time: Date.now() });
      }, 5000);
      
      // Cleanup on close
      request.signal.addEventListener('abort', () => {
        clearInterval(interval);
        controller.close();
      });
    },
  });
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

## 5.4 Complete SSE Client

```typescript
type EventHandler<T> = (data: T) => void;

class SSEClient {
  private eventSource: EventSource | null = null;
  private url: string;
  private handlers: Map<string, Set<EventHandler<any>>> = new Map();
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  
  constructor(url: string) {
    this.url = url;
  }
  
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.eventSource = new EventSource(this.url, {
        withCredentials: true,
      });
      
      this.eventSource.onopen = () => {
        console.log('SSE connected');
        this.reconnectAttempts = 0;
        resolve();
      };
      
      this.eventSource.onerror = (error) => {
        if (this.eventSource?.readyState === EventSource.CLOSED) {
          console.error('SSE connection closed');
          this.handleReconnect();
        }
      };
      
      // Default message handler
      this.eventSource.onmessage = (event) => {
        this.dispatch('message', event.data);
      };
    });
  }
  
  private handleReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max SSE reconnection attempts reached');
      return;
    }
    
    this.reconnectAttempts++;
    const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
    
    setTimeout(() => {
      console.log(`SSE reconnecting (attempt ${this.reconnectAttempts})`);
      this.connect();
    }, delay);
  }
  
  on<T>(event: string, handler: EventHandler<T>): () => void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
      
      // Register with EventSource
      this.eventSource?.addEventListener(event, (e: MessageEvent) => {
        try {
          const data = JSON.parse(e.data);
          this.dispatch(event, data);
        } catch {
          this.dispatch(event, e.data);
        }
      });
    }
    
    this.handlers.get(event)!.add(handler);
    
    return () => {
      this.handlers.get(event)?.delete(handler);
    };
  }
  
  private dispatch(event: string, data: any) {
    this.handlers.get(event)?.forEach(handler => handler(data));
  }
  
  disconnect() {
    this.eventSource?.close();
    this.eventSource = null;
    this.handlers.clear();
  }
  
  get isConnected(): boolean {
    return this.eventSource?.readyState === EventSource.OPEN;
  }
}

// Usage
const sse = new SSEClient('/api/events');

sse.on('notification', (data: Notification) => {
  showNotification(data);
});

sse.on('price:update', (data: { symbol: string; price: number }) => {
  updatePrice(data.symbol, data.price);
});

await sse.connect();
```

---

# 6. Long Polling

## 6.1 How Long Polling Works

```typescript
// Long Polling flow:
// 1. Client sends request
// 2. Server holds request until data available or timeout
// 3. Server responds
// 4. Client immediately sends new request
// 5. Repeat

async function longPoll(url: string) {
  while (true) {
    try {
      const response = await fetch(url, {
        method: 'GET',
        headers: { 'Accept': 'application/json' },
      });
      
      if (response.ok) {
        const data = await response.json();
        handleData(data);
      }
      
      // Immediately poll again
    } catch (error) {
      console.error('Poll error:', error);
      // Wait before retrying on error
      await delay(5000);
    }
  }
}
```

## 6.2 Complete Long Polling Client

```typescript
interface LongPollOptions {
  url: string;
  timeout?: number;
  retryDelay?: number;
  maxRetries?: number;
  onMessage: (data: any) => void;
  onError?: (error: Error) => void;
}

class LongPollingClient {
  private isPolling = false;
  private abortController: AbortController | null = null;
  private retryCount = 0;
  private options: Required<LongPollOptions>;
  private lastEventId: string | null = null;
  
  constructor(options: LongPollOptions) {
    this.options = {
      timeout: 30000,
      retryDelay: 3000,
      maxRetries: 10,
      onError: console.error,
      ...options,
    };
  }
  
  start() {
    if (this.isPolling) return;
    this.isPolling = true;
    this.poll();
  }
  
  stop() {
    this.isPolling = false;
    this.abortController?.abort();
    this.abortController = null;
  }
  
  private async poll() {
    while (this.isPolling) {
      try {
        this.abortController = new AbortController();
        
        const url = new URL(this.options.url, window.location.origin);
        if (this.lastEventId) {
          url.searchParams.set('lastEventId', this.lastEventId);
        }
        
        const response = await fetch(url.toString(), {
          method: 'GET',
          signal: this.abortController.signal,
          headers: {
            'Accept': 'application/json',
          },
        });
        
        if (response.status === 200) {
          const data = await response.json();
          
          if (data.eventId) {
            this.lastEventId = data.eventId;
          }
          
          this.options.onMessage(data);
          this.retryCount = 0;
          
        } else if (response.status === 204) {
          // No new data, poll again
          
        } else if (response.status === 408) {
          // Timeout, poll again
          
        } else {
          throw new Error(`HTTP ${response.status}`);
        }
        
      } catch (error) {
        if (error instanceof Error && error.name === 'AbortError') {
          // Intentional abort
          break;
        }
        
        this.retryCount++;
        this.options.onError?.(error as Error);
        
        if (this.retryCount >= this.options.maxRetries) {
          console.error('Max retries reached, stopping long polling');
          this.stop();
          break;
        }
        
        // Wait before retrying
        await this.delay(this.options.retryDelay * this.retryCount);
      }
    }
  }
  
  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const poller = new LongPollingClient({
  url: '/api/poll',
  onMessage: (data) => {
    console.log('Received:', data);
  },
  onError: (error) => {
    console.error('Poll error:', error);
  },
});

poller.start();

// Later
poller.stop();
```

---

# 7. Connection Management

## 7.1 Connection State Machine

```typescript
type ConnectionState = 
  | 'disconnected'
  | 'connecting'
  | 'connected'
  | 'reconnecting'
  | 'failed';

interface ConnectionEvents {
  onStateChange: (state: ConnectionState) => void;
  onConnect: () => void;
  onDisconnect: (reason: string) => void;
  onError: (error: Error) => void;
}

class ConnectionManager {
  private state: ConnectionState = 'disconnected';
  private events: Partial<ConnectionEvents> = {};
  private ws: WebSocket | null = null;
  private url: string;
  
  constructor(url: string) {
    this.url = url;
  }
  
  private setState(newState: ConnectionState) {
    if (this.state !== newState) {
      const oldState = this.state;
      this.state = newState;
      console.log(`Connection: ${oldState} → ${newState}`);
      this.events.onStateChange?.(newState);
    }
  }
  
  connect() {
    if (this.state === 'connected' || this.state === 'connecting') {
      return;
    }
    
    this.setState('connecting');
    
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      this.setState('connected');
      this.events.onConnect?.();
    };
    
    this.ws.onclose = (event) => {
      const reason = event.reason || `Code: ${event.code}`;
      
      if (event.code === 1000) {
        this.setState('disconnected');
      } else {
        this.setState('reconnecting');
        this.scheduleReconnect();
      }
      
      this.events.onDisconnect?.(reason);
    };
    
    this.ws.onerror = (error) => {
      this.events.onError?.(new Error('WebSocket error'));
    };
  }
  
  private scheduleReconnect() {
    setTimeout(() => {
      if (this.state === 'reconnecting') {
        this.connect();
      }
    }, 3000);
  }
  
  disconnect() {
    if (this.ws) {
      this.ws.close(1000, 'Client disconnect');
      this.ws = null;
    }
    this.setState('disconnected');
  }
  
  onStateChange(handler: (state: ConnectionState) => void) {
    this.events.onStateChange = handler;
    return () => { this.events.onStateChange = undefined; };
  }
  
  getState(): ConnectionState {
    return this.state;
  }
}
```

## 7.2 Heartbeat / Keep-Alive

```typescript
class HeartbeatManager {
  private interval: NodeJS.Timer | null = null;
  private timeout: NodeJS.Timer | null = null;
  private onDead: () => void;
  private pingInterval: number;
  private pongTimeout: number;
  
  constructor(options: {
    pingInterval?: number;
    pongTimeout?: number;
    onDead: () => void;
  }) {
    this.pingInterval = options.pingInterval || 25000;
    this.pongTimeout = options.pongTimeout || 5000;
    this.onDead = options.onDead;
  }
  
  start(sendPing: () => void) {
    this.stop();
    
    this.interval = setInterval(() => {
      sendPing();
      this.waitForPong();
    }, this.pingInterval);
  }
  
  private waitForPong() {
    this.timeout = setTimeout(() => {
      console.error('Heartbeat timeout - connection dead');
      this.onDead();
    }, this.pongTimeout);
  }
  
  receivedPong() {
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    }
  }
  
  stop() {
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
    }
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    }
  }
}

// Usage in WebSocket client
const heartbeat = new HeartbeatManager({
  pingInterval: 25000,
  pongTimeout: 5000,
  onDead: () => {
    console.log('Connection dead, reconnecting...');
    ws.close();
    reconnect();
  },
});

ws.onopen = () => {
  heartbeat.start(() => {
    ws.send(JSON.stringify({ type: 'ping' }));
  });
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'pong') {
    heartbeat.receivedPong();
  }
};

ws.onclose = () => {
  heartbeat.stop();
};
```

---

# 8. Reconnection Strategies

## 8.1 Exponential Backoff

```typescript
class ExponentialBackoff {
  private attempt = 0;
  private readonly baseDelay: number;
  private readonly maxDelay: number;
  private readonly maxAttempts: number;
  private readonly jitter: boolean;
  
  constructor(options: {
    baseDelay?: number;
    maxDelay?: number;
    maxAttempts?: number;
    jitter?: boolean;
  } = {}) {
    this.baseDelay = options.baseDelay || 1000;
    this.maxDelay = options.maxDelay || 30000;
    this.maxAttempts = options.maxAttempts || 10;
    this.jitter = options.jitter ?? true;
  }
  
  getNextDelay(): number | null {
    if (this.attempt >= this.maxAttempts) {
      return null; // Max attempts reached
    }
    
    let delay = this.baseDelay * Math.pow(2, this.attempt);
    delay = Math.min(delay, this.maxDelay);
    
    if (this.jitter) {
      // Add ±25% jitter
      const jitterRange = delay * 0.25;
      delay += Math.random() * jitterRange * 2 - jitterRange;
    }
    
    this.attempt++;
    return Math.round(delay);
  }
  
  reset() {
    this.attempt = 0;
  }
  
  get currentAttempt(): number {
    return this.attempt;
  }
}

// Usage
const backoff = new ExponentialBackoff({
  baseDelay: 1000,
  maxDelay: 30000,
  maxAttempts: 10,
});

async function connectWithBackoff() {
  while (true) {
    try {
      await connect();
      backoff.reset();
      return; // Success
    } catch (error) {
      const delay = backoff.getNextDelay();
      
      if (delay === null) {
        throw new Error('Max reconnection attempts reached');
      }
      
      console.log(`Reconnecting in ${delay}ms (attempt ${backoff.currentAttempt})`);
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

## 8.2 Smart Reconnection with Network Detection

```typescript
class SmartReconnection {
  private backoff = new ExponentialBackoff();
  private isOnline = navigator.onLine;
  private reconnectTimer: NodeJS.Timer | null = null;
  private onReconnect: () => Promise<void>;
  
  constructor(onReconnect: () => Promise<void>) {
    this.onReconnect = onReconnect;
    this.setupNetworkListeners();
  }
  
  private setupNetworkListeners() {
    window.addEventListener('online', () => {
      console.log('Network online');
      this.isOnline = true;
      this.attemptReconnect();
    });
    
    window.addEventListener('offline', () => {
      console.log('Network offline');
      this.isOnline = false;
      this.cancelReconnect();
    });
    
    // Visibility change
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'visible' && this.isOnline) {
        this.attemptReconnect();
      }
    });
  }
  
  scheduleReconnect() {
    if (!this.isOnline) {
      console.log('Offline, waiting for network...');
      return;
    }
    
    const delay = this.backoff.getNextDelay();
    
    if (delay === null) {
      console.error('Max reconnection attempts reached');
      return;
    }
    
    console.log(`Scheduling reconnect in ${delay}ms`);
    
    this.reconnectTimer = setTimeout(async () => {
      await this.attemptReconnect();
    }, delay);
  }
  
  private async attemptReconnect() {
    if (!this.isOnline) return;
    
    try {
      await this.onReconnect();
      this.backoff.reset();
      console.log('Reconnected successfully');
    } catch (error) {
      console.error('Reconnection failed:', error);
      this.scheduleReconnect();
    }
  }
  
  private cancelReconnect() {
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
      this.reconnectTimer = null;
    }
  }
  
  reset() {
    this.backoff.reset();
    this.cancelReconnect();
  }
}
```

---

# 9. Authentication in Real-Time

## 9.1 Token-Based Authentication

```typescript
// WebSocket with JWT
function connectWithAuth(token: string): WebSocket {
  // Option 1: Token in query string (less secure)
  const ws = new WebSocket(`wss://api.example.com?token=${token}`);
  
  // Option 2: Token in first message (more secure)
  const ws = new WebSocket('wss://api.example.com');
  ws.onopen = () => {
    ws.send(JSON.stringify({
      type: 'auth',
      token: token,
    }));
  };
  
  return ws;
}

// Socket.io with auth
const socket = io('https://api.example.com', {
  auth: {
    token: 'your-jwt-token',
  },
});

// Handle auth errors
socket.on('connect_error', (error) => {
  if (error.message === 'unauthorized') {
    // Refresh token and reconnect
    refreshToken().then(newToken => {
      socket.auth = { token: newToken };
      socket.connect();
    });
  }
});
```

## 9.2 Token Refresh in Real-Time

```typescript
class AuthenticatedSocket {
  private socket: Socket | null = null;
  private getToken: () => Promise<string>;
  
  constructor(getToken: () => Promise<string>) {
    this.getToken = getToken;
  }
  
  async connect(url: string) {
    const token = await this.getToken();
    
    this.socket = io(url, {
      auth: { token },
      reconnection: false, // Manual reconnection for auth
    });
    
    this.socket.on('connect_error', async (error) => {
      if (error.message === 'jwt expired') {
        await this.handleExpiredToken();
      }
    });
    
    // Periodic token refresh
    this.socket.on('auth:expiring', async () => {
      const newToken = await this.getToken();
      this.socket?.emit('auth:refresh', { token: newToken });
    });
  }
  
  private async handleExpiredToken() {
    try {
      const newToken = await this.getToken();
      
      if (this.socket) {
        this.socket.auth = { token: newToken };
        this.socket.connect();
      }
    } catch (error) {
      console.error('Token refresh failed:', error);
      this.socket?.disconnect();
    }
  }
  
  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }
}

// Usage
const authSocket = new AuthenticatedSocket(async () => {
  return authService.getAccessToken();
});

await authSocket.connect('https://api.example.com');
```

---

# 10. React Integration Patterns

## 10.1 WebSocket React Hook

```typescript
import { useState, useEffect, useCallback, useRef } from 'react';

interface UseWebSocketOptions {
  url: string;
  onOpen?: () => void;
  onClose?: (event: CloseEvent) => void;
  onError?: (error: Event) => void;
  onMessage?: (message: MessageEvent) => void;
  reconnect?: boolean;
  reconnectInterval?: number;
}

interface UseWebSocketResult {
  sendMessage: (message: string | object) => void;
  lastMessage: any;
  readyState: number;
  isConnected: boolean;
}

function useWebSocket(options: UseWebSocketOptions): UseWebSocketResult {
  const {
    url,
    onOpen,
    onClose,
    onError,
    onMessage,
    reconnect = true,
    reconnectInterval = 3000,
  } = options;
  
  const [lastMessage, setLastMessage] = useState<any>(null);
  const [readyState, setReadyState] = useState<number>(WebSocket.CLOSED);
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timer>();
  
  const connect = useCallback(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;
    
    ws.onopen = () => {
      setReadyState(WebSocket.OPEN);
      onOpen?.();
    };
    
    ws.onclose = (event) => {
      setReadyState(WebSocket.CLOSED);
      onClose?.(event);
      
      if (reconnect && event.code !== 1000) {
        reconnectTimeoutRef.current = setTimeout(connect, reconnectInterval);
      }
    };
    
    ws.onerror = (error) => {
      onError?.(error);
    };
    
    ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        setLastMessage(data);
      } catch {
        setLastMessage(event.data);
      }
      onMessage?.(event);
    };
  }, [url, onOpen, onClose, onError, onMessage, reconnect, reconnectInterval]);
  
  useEffect(() => {
    connect();
    
    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
      wsRef.current?.close(1000);
    };
  }, [connect]);
  
  const sendMessage = useCallback((message: string | object) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      const data = typeof message === 'string' ? message : JSON.stringify(message);
      wsRef.current.send(data);
    }
  }, []);
  
  return {
    sendMessage,
    lastMessage,
    readyState,
    isConnected: readyState === WebSocket.OPEN,
  };
}

// Usage
function ChatComponent() {
  const { sendMessage, lastMessage, isConnected } = useWebSocket({
    url: 'wss://api.example.com/chat',
    onOpen: () => console.log('Connected!'),
    onMessage: (msg) => console.log('Message:', msg),
  });
  
  const [input, setInput] = useState('');
  
  const handleSend = () => {
    sendMessage({ type: 'chat', text: input });
    setInput('');
  };
  
  return (
    <div>
      <div>Status: {isConnected ? 'Connected' : 'Disconnected'}</div>
      <div>Last message: {JSON.stringify(lastMessage)}</div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={handleSend} disabled={!isConnected}>Send</button>
    </div>
  );
}
```

## 10.2 Socket.io React Hook

```typescript
import { useEffect, useState, useCallback, useRef } from 'react';
import { io, Socket } from 'socket.io-client';

interface UseSocketOptions {
  url: string;
  auth?: Record<string, any>;
  autoConnect?: boolean;
}

function useSocket(options: UseSocketOptions) {
  const { url, auth, autoConnect = true } = options;
  const [isConnected, setIsConnected] = useState(false);
  const socketRef = useRef<Socket | null>(null);
  
  useEffect(() => {
    socketRef.current = io(url, {
      auth,
      autoConnect,
    });
    
    const socket = socketRef.current;
    
    socket.on('connect', () => setIsConnected(true));
    socket.on('disconnect', () => setIsConnected(false));
    
    return () => {
      socket.disconnect();
    };
  }, [url, auth, autoConnect]);
  
  const emit = useCallback((event: string, ...args: any[]) => {
    socketRef.current?.emit(event, ...args);
  }, []);
  
  const on = useCallback((event: string, handler: (...args: any[]) => void) => {
    socketRef.current?.on(event, handler);
    return () => {
      socketRef.current?.off(event, handler);
    };
  }, []);
  
  return {
    socket: socketRef.current,
    isConnected,
    emit,
    on,
  };
}

// Usage
function ChatRoom({ roomId }: { roomId: string }) {
  const { isConnected, emit, on } = useSocket({
    url: 'https://api.example.com',
    auth: { token: 'jwt-token' },
  });
  
  const [messages, setMessages] = useState<Message[]>([]);
  
  useEffect(() => {
    const unsubscribe = on('chat:message', (message: Message) => {
      setMessages(prev => [...prev, message]);
    });
    
    return unsubscribe;
  }, [on]);
  
  useEffect(() => {
    if (isConnected) {
      emit('room:join', roomId);
    }
  }, [isConnected, emit, roomId]);
  
  const sendMessage = (text: string) => {
    emit('chat:send', { roomId, text });
  };
  
  return (
    <div>
      {messages.map(msg => (
        <div key={msg.id}>{msg.text}</div>
      ))}
    </div>
  );
}
```

---

## 10.3 Socket.io React Context

```typescript
import React, { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import { io, Socket } from 'socket.io-client';

interface SocketContextType {
  socket: Socket | null;
  isConnected: boolean;
  connect: () => void;
  disconnect: () => void;
}

const SocketContext = createContext<SocketContextType | null>(null);

interface SocketProviderProps {
  url: string;
  auth?: Record<string, any>;
  children: ReactNode;
}

export function SocketProvider({ url, auth, children }: SocketProviderProps) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  
  useEffect(() => {
    const newSocket = io(url, {
      auth,
      autoConnect: false,
    });
    
    newSocket.on('connect', () => setIsConnected(true));
    newSocket.on('disconnect', () => setIsConnected(false));
    
    setSocket(newSocket);
    
    return () => {
      newSocket.disconnect();
    };
  }, [url, auth]);
  
  const connect = () => socket?.connect();
  const disconnect = () => socket?.disconnect();
  
  return (
    <SocketContext.Provider value={{ socket, isConnected, connect, disconnect }}>
      {children}
    </SocketContext.Provider>
  );
}

export function useSocketContext() {
  const context = useContext(SocketContext);
  if (!context) {
    throw new Error('useSocketContext must be used within SocketProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <SocketProvider url="https://api.example.com" auth={{ token: 'jwt' }}>
      <ChatComponent />
    </SocketProvider>
  );
}

function ChatComponent() {
  const { socket, isConnected, connect, disconnect } = useSocketContext();
  
  useEffect(() => {
    connect();
    return () => disconnect();
  }, []);
  
  // Use socket...
}
```

---

# 11. Real-Time State Management

## 11.1 Zustand with WebSocket

```typescript
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';

interface Message {
  id: string;
  userId: string;
  text: string;
  timestamp: number;
}

interface User {
  id: string;
  name: string;
  status: 'online' | 'offline' | 'away';
}

interface ChatState {
  messages: Message[];
  users: Map<string, User>;
  typingUsers: Set<string>;
  connectionStatus: 'connected' | 'disconnected' | 'connecting';
  
  // Actions
  addMessage: (message: Message) => void;
  setMessages: (messages: Message[]) => void;
  updateUser: (user: User) => void;
  removeUser: (userId: string) => void;
  setTyping: (userId: string, isTyping: boolean) => void;
  setConnectionStatus: (status: ChatState['connectionStatus']) => void;
}

export const useChatStore = create<ChatState>()(
  subscribeWithSelector((set, get) => ({
    messages: [],
    users: new Map(),
    typingUsers: new Set(),
    connectionStatus: 'disconnected',
    
    addMessage: (message) =>
      set((state) => ({
        messages: [...state.messages, message],
      })),
    
    setMessages: (messages) =>
      set({ messages }),
    
    updateUser: (user) =>
      set((state) => {
        const newUsers = new Map(state.users);
        newUsers.set(user.id, user);
        return { users: newUsers };
      }),
    
    removeUser: (userId) =>
      set((state) => {
        const newUsers = new Map(state.users);
        newUsers.delete(userId);
        return { users: newUsers };
      }),
    
    setTyping: (userId, isTyping) =>
      set((state) => {
        const newTyping = new Set(state.typingUsers);
        if (isTyping) {
          newTyping.add(userId);
        } else {
          newTyping.delete(userId);
        }
        return { typingUsers: newTyping };
      }),
    
    setConnectionStatus: (status) =>
      set({ connectionStatus: status }),
  }))
);

// WebSocket integration
function initializeWebSocket(socket: WebSocket) {
  const store = useChatStore.getState();
  
  socket.onopen = () => {
    store.setConnectionStatus('connected');
  };
  
  socket.onclose = () => {
    store.setConnectionStatus('disconnected');
  };
  
  socket.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    switch (data.type) {
      case 'message':
        store.addMessage(data.payload);
        break;
      case 'user:join':
        store.updateUser(data.payload);
        break;
      case 'user:leave':
        store.removeUser(data.payload.userId);
        break;
      case 'typing':
        store.setTyping(data.payload.userId, data.payload.isTyping);
        break;
    }
  };
}
```

## 11.2 Redux with Real-Time Updates

```typescript
import { createSlice, PayloadAction, configureStore } from '@reduxjs/toolkit';

interface Message {
  id: string;
  userId: string;
  text: string;
  timestamp: number;
}

interface ChatState {
  messages: Message[];
  isConnected: boolean;
  typingUserIds: string[];
}

const initialState: ChatState = {
  messages: [],
  isConnected: false,
  typingUserIds: [],
};

const chatSlice = createSlice({
  name: 'chat',
  initialState,
  reducers: {
    connected(state) {
      state.isConnected = true;
    },
    disconnected(state) {
      state.isConnected = false;
    },
    messageReceived(state, action: PayloadAction<Message>) {
      state.messages.push(action.payload);
    },
    messagesLoaded(state, action: PayloadAction<Message[]>) {
      state.messages = action.payload;
    },
    userStartedTyping(state, action: PayloadAction<string>) {
      if (!state.typingUserIds.includes(action.payload)) {
        state.typingUserIds.push(action.payload);
      }
    },
    userStoppedTyping(state, action: PayloadAction<string>) {
      state.typingUserIds = state.typingUserIds.filter(
        id => id !== action.payload
      );
    },
  },
});

export const {
  connected,
  disconnected,
  messageReceived,
  messagesLoaded,
  userStartedTyping,
  userStoppedTyping,
} = chatSlice.actions;

// Redux middleware for WebSocket
function createSocketMiddleware(url: string) {
  return (store: any) => {
    let socket: WebSocket | null = null;
    
    return (next: any) => (action: any) => {
      switch (action.type) {
        case 'socket/connect':
          if (socket) socket.close();
          
          socket = new WebSocket(url);
          
          socket.onopen = () => {
            store.dispatch(connected());
          };
          
          socket.onclose = () => {
            store.dispatch(disconnected());
          };
          
          socket.onmessage = (event) => {
            const data = JSON.parse(event.data);
            
            switch (data.type) {
              case 'message':
                store.dispatch(messageReceived(data.payload));
                break;
              case 'typing:start':
                store.dispatch(userStartedTyping(data.userId));
                break;
              case 'typing:stop':
                store.dispatch(userStoppedTyping(data.userId));
                break;
            }
          };
          break;
          
        case 'socket/disconnect':
          socket?.close();
          socket = null;
          break;
          
        case 'socket/send':
          socket?.send(JSON.stringify(action.payload));
          break;
      }
      
      return next(action);
    };
  };
}

const store = configureStore({
  reducer: {
    chat: chatSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(createSocketMiddleware('wss://api.example.com/ws')),
});
```

## 11.3 React Query with Real-Time Updates

```typescript
import { useQuery, useQueryClient, useMutation } from '@tanstack/react-query';
import { useEffect } from 'react';

interface Message {
  id: string;
  text: string;
  userId: string;
  timestamp: number;
}

// Fetch messages with React Query
function useMessages(roomId: string) {
  const queryClient = useQueryClient();
  
  // Initial fetch
  const query = useQuery({
    queryKey: ['messages', roomId],
    queryFn: () => fetchMessages(roomId),
  });
  
  // Real-time updates via WebSocket
  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/rooms/${roomId}`);
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'new_message') {
        // Optimistically add new message to cache
        queryClient.setQueryData<Message[]>(
          ['messages', roomId],
          (old) => old ? [...old, data.message] : [data.message]
        );
      }
      
      if (data.type === 'message_deleted') {
        queryClient.setQueryData<Message[]>(
          ['messages', roomId],
          (old) => old?.filter(m => m.id !== data.messageId) ?? []
        );
      }
    };
    
    return () => ws.close();
  }, [roomId, queryClient]);
  
  return query;
}

// Send message with optimistic update
function useSendMessage() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async ({ roomId, text }: { roomId: string; text: string }) => {
      const response = await fetch(`/api/rooms/${roomId}/messages`, {
        method: 'POST',
        body: JSON.stringify({ text }),
      });
      return response.json();
    },
    
    onMutate: async ({ roomId, text }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['messages', roomId] });
      
      // Snapshot previous value
      const previousMessages = queryClient.getQueryData<Message[]>(['messages', roomId]);
      
      // Optimistically add message
      const optimisticMessage: Message = {
        id: `temp-${Date.now()}`,
        text,
        userId: 'current-user',
        timestamp: Date.now(),
      };
      
      queryClient.setQueryData<Message[]>(
        ['messages', roomId],
        (old) => old ? [...old, optimisticMessage] : [optimisticMessage]
      );
      
      return { previousMessages };
    },
    
    onError: (err, { roomId }, context) => {
      // Rollback on error
      if (context?.previousMessages) {
        queryClient.setQueryData(['messages', roomId], context.previousMessages);
      }
    },
    
    onSettled: (data, error, { roomId }) => {
      // Refetch to sync with server
      queryClient.invalidateQueries({ queryKey: ['messages', roomId] });
    },
  });
}
```

---

# 12. Building Real-Time Features

## 12.1 Complete Chat Application

```typescript
import { useState, useEffect, useRef, useCallback } from 'react';
import { io, Socket } from 'socket.io-client';

interface Message {
  id: string;
  userId: string;
  userName: string;
  text: string;
  timestamp: number;
  status: 'sending' | 'sent' | 'delivered' | 'read';
}

interface ChatRoom {
  id: string;
  name: string;
  members: User[];
  unreadCount: number;
}

interface User {
  id: string;
  name: string;
  avatar: string;
  status: 'online' | 'offline' | 'away';
}

// Chat Hook
function useChat(roomId: string, userId: string) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [typingUsers, setTypingUsers] = useState<string[]>([]);
  const [isConnected, setIsConnected] = useState(false);
  const socketRef = useRef<Socket | null>(null);
  const typingTimeoutRef = useRef<NodeJS.Timer>();
  
  useEffect(() => {
    const socket = io('https://api.example.com', {
      auth: { userId },
    });
    
    socketRef.current = socket;
    
    socket.on('connect', () => {
      setIsConnected(true);
      socket.emit('room:join', roomId);
    });
    
    socket.on('disconnect', () => {
      setIsConnected(false);
    });
    
    socket.on('messages:history', (history: Message[]) => {
      setMessages(history);
    });
    
    socket.on('message:new', (message: Message) => {
      setMessages(prev => [...prev, message]);
      
      // Mark as delivered
      socket.emit('message:delivered', { messageId: message.id });
    });
    
    socket.on('message:status', ({ messageId, status }) => {
      setMessages(prev =>
        prev.map(msg =>
          msg.id === messageId ? { ...msg, status } : msg
        )
      );
    });
    
    socket.on('typing:update', ({ userId, isTyping }) => {
      setTypingUsers(prev => {
        if (isTyping && !prev.includes(userId)) {
          return [...prev, userId];
        }
        if (!isTyping) {
          return prev.filter(id => id !== userId);
        }
        return prev;
      });
    });
    
    return () => {
      socket.emit('room:leave', roomId);
      socket.disconnect();
    };
  }, [roomId, userId]);
  
  const sendMessage = useCallback((text: string) => {
    const tempId = `temp-${Date.now()}`;
    
    // Optimistic update
    const optimisticMessage: Message = {
      id: tempId,
      userId,
      userName: 'You',
      text,
      timestamp: Date.now(),
      status: 'sending',
    };
    
    setMessages(prev => [...prev, optimisticMessage]);
    
    socketRef.current?.emit('message:send', { text, tempId }, (response: any) => {
      if (response.success) {
        setMessages(prev =>
          prev.map(msg =>
            msg.id === tempId ? { ...msg, id: response.id, status: 'sent' } : msg
          )
        );
      }
    });
  }, [userId]);
  
  const sendTyping = useCallback((isTyping: boolean) => {
    socketRef.current?.emit('typing:update', { isTyping });
  }, []);
  
  const handleTypingStart = useCallback(() => {
    sendTyping(true);
    
    // Auto-stop typing after 3 seconds
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }
    
    typingTimeoutRef.current = setTimeout(() => {
      sendTyping(false);
    }, 3000);
  }, [sendTyping]);
  
  const markAsRead = useCallback((messageId: string) => {
    socketRef.current?.emit('message:read', { messageId });
  }, []);
  
  return {
    messages,
    typingUsers,
    isConnected,
    sendMessage,
    handleTypingStart,
    markAsRead,
  };
}

// Chat Component
function ChatRoom({ roomId, userId }: { roomId: string; userId: string }) {
  const {
    messages,
    typingUsers,
    isConnected,
    sendMessage,
    handleTypingStart,
    markAsRead,
  } = useChat(roomId, userId);
  
  const [inputValue, setInputValue] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (inputValue.trim()) {
      sendMessage(inputValue.trim());
      setInputValue('');
    }
  };
  
  return (
    <div className="chat-container">
      <div className="connection-status">
        {isConnected ? '🟢 Connected' : '🔴 Disconnected'}
      </div>
      
      <div className="messages">
        {messages.map((message) => (
          <MessageBubble
            key={message.id}
            message={message}
            isOwn={message.userId === userId}
            onVisible={() => markAsRead(message.id)}
          />
        ))}
        <div ref={messagesEndRef} />
      </div>
      
      {typingUsers.length > 0 && (
        <div className="typing-indicator">
          {typingUsers.join(', ')} {typingUsers.length === 1 ? 'is' : 'are'} typing...
        </div>
      )}
      
      <form onSubmit={handleSubmit} className="message-input">
        <input
          type="text"
          value={inputValue}
          onChange={(e) => {
            setInputValue(e.target.value);
            handleTypingStart();
          }}
          placeholder="Type a message..."
        />
        <button type="submit" disabled={!isConnected}>
          Send
        </button>
      </form>
    </div>
  );
}

function MessageBubble({
  message,
  isOwn,
  onVisible,
}: {
  message: Message;
  isOwn: boolean;
  onVisible: () => void;
}) {
  const ref = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          onVisible();
          observer.disconnect();
        }
      },
      { threshold: 1.0 }
    );
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => observer.disconnect();
  }, [onVisible]);
  
  return (
    <div ref={ref} className={`message ${isOwn ? 'own' : 'other'}`}>
      <span className="author">{message.userName}</span>
      <p className="text">{message.text}</p>
      <span className="time">
        {new Date(message.timestamp).toLocaleTimeString()}
        {isOwn && <span className="status">{getStatusIcon(message.status)}</span>}
      </span>
    </div>
  );
}

function getStatusIcon(status: Message['status']): string {
  switch (status) {
    case 'sending': return '⏳';
    case 'sent': return '✓';
    case 'delivered': return '✓✓';
    case 'read': return '✓✓';
    default: return '';
  }
}
```

## 12.2 Presence System

```typescript
interface PresenceState {
  status: 'online' | 'away' | 'offline';
  lastSeen?: number;
  currentPage?: string;
}

class PresenceManager {
  private socket: Socket;
  private userId: string;
  private heartbeatInterval: NodeJS.Timer | null = null;
  private idleTimeout: NodeJS.Timer | null = null;
  private currentStatus: PresenceState['status'] = 'offline';
  
  constructor(socket: Socket, userId: string) {
    this.socket = socket;
    this.userId = userId;
    this.setupListeners();
  }
  
  private setupListeners() {
    // Activity detection
    const resetIdleTimer = () => {
      if (this.currentStatus === 'away') {
        this.setStatus('online');
      }
      
      if (this.idleTimeout) {
        clearTimeout(this.idleTimeout);
      }
      
      // Mark as away after 5 minutes of inactivity
      this.idleTimeout = setTimeout(() => {
        this.setStatus('away');
      }, 5 * 60 * 1000);
    };
    
    window.addEventListener('mousemove', resetIdleTimer);
    window.addEventListener('keypress', resetIdleTimer);
    window.addEventListener('click', resetIdleTimer);
    window.addEventListener('scroll', resetIdleTimer);
    
    // Page visibility
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.setStatus('away');
      } else {
        this.setStatus('online');
      }
    });
    
    // Before unload
    window.addEventListener('beforeunload', () => {
      this.setStatus('offline');
    });
  }
  
  connect() {
    this.setStatus('online');
    
    // Heartbeat every 30 seconds
    this.heartbeatInterval = setInterval(() => {
      this.socket.emit('presence:heartbeat', {
        userId: this.userId,
        status: this.currentStatus,
      });
    }, 30000);
  }
  
  disconnect() {
    this.setStatus('offline');
    
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
    }
    if (this.idleTimeout) {
      clearTimeout(this.idleTimeout);
    }
  }
  
  setStatus(status: PresenceState['status']) {
    if (this.currentStatus !== status) {
      this.currentStatus = status;
      this.socket.emit('presence:update', {
        userId: this.userId,
        status,
        timestamp: Date.now(),
      });
    }
  }
  
  setCurrentPage(page: string) {
    this.socket.emit('presence:page', {
      userId: this.userId,
      page,
    });
  }
}

// React Hook for Presence
function usePresence(userId: string, socket: Socket | null) {
  const [onlineUsers, setOnlineUsers] = useState<Map<string, PresenceState>>(new Map());
  const presenceRef = useRef<PresenceManager | null>(null);
  
  useEffect(() => {
    if (!socket) return;
    
    presenceRef.current = new PresenceManager(socket, userId);
    presenceRef.current.connect();
    
    socket.on('presence:list', (users: Record<string, PresenceState>) => {
      setOnlineUsers(new Map(Object.entries(users)));
    });
    
    socket.on('presence:updated', ({ userId, status }) => {
      setOnlineUsers(prev => {
        const next = new Map(prev);
        next.set(userId, status);
        return next;
      });
    });
    
    socket.on('presence:offline', ({ userId }) => {
      setOnlineUsers(prev => {
        const next = new Map(prev);
        next.delete(userId);
        return next;
      });
    });
    
    return () => {
      presenceRef.current?.disconnect();
    };
  }, [socket, userId]);
  
  const isOnline = useCallback((id: string) => {
    const presence = onlineUsers.get(id);
    return presence?.status === 'online';
  }, [onlineUsers]);
  
  const getStatus = useCallback((id: string) => {
    return onlineUsers.get(id)?.status || 'offline';
  }, [onlineUsers]);
  
  return { onlineUsers, isOnline, getStatus };
}
```

## 12.3 Collaborative Editing (OT/CRDT Basics)

```typescript
// Operational Transformation basics
interface Operation {
  type: 'insert' | 'delete';
  position: number;
  text?: string;
  length?: number;
  userId: string;
  timestamp: number;
}

class OTDocument {
  private content: string = '';
  private version: number = 0;
  private pendingOps: Operation[] = [];
  private socket: Socket;
  
  constructor(socket: Socket) {
    this.socket = socket;
    this.setupListeners();
  }
  
  private setupListeners() {
    this.socket.on('doc:init', ({ content, version }) => {
      this.content = content;
      this.version = version;
    });
    
    this.socket.on('doc:operation', (op: Operation) => {
      this.applyRemoteOperation(op);
    });
  }
  
  insert(position: number, text: string, userId: string) {
    const op: Operation = {
      type: 'insert',
      position,
      text,
      userId,
      timestamp: Date.now(),
    };
    
    this.applyLocalOperation(op);
    this.sendOperation(op);
  }
  
  delete(position: number, length: number, userId: string) {
    const op: Operation = {
      type: 'delete',
      position,
      length,
      userId,
      timestamp: Date.now(),
    };
    
    this.applyLocalOperation(op);
    this.sendOperation(op);
  }
  
  private applyLocalOperation(op: Operation) {
    this.content = this.applyOperation(this.content, op);
    this.pendingOps.push(op);
  }
  
  private applyRemoteOperation(op: Operation) {
    // Transform pending operations against the remote operation
    this.pendingOps = this.pendingOps.map(pending =>
      this.transform(pending, op)
    );
    
    this.content = this.applyOperation(this.content, op);
    this.version++;
  }
  
  private applyOperation(content: string, op: Operation): string {
    if (op.type === 'insert' && op.text) {
      return (
        content.slice(0, op.position) +
        op.text +
        content.slice(op.position)
      );
    }
    
    if (op.type === 'delete' && op.length) {
      return (
        content.slice(0, op.position) +
        content.slice(op.position + op.length)
      );
    }
    
    return content;
  }
  
  private transform(op1: Operation, op2: Operation): Operation {
    // Simplified OT transformation
    if (op2.type === 'insert') {
      if (op1.position > op2.position) {
        return { ...op1, position: op1.position + (op2.text?.length || 0) };
      }
    }
    
    if (op2.type === 'delete') {
      if (op1.position > op2.position) {
        return { ...op1, position: op1.position - (op2.length || 0) };
      }
    }
    
    return op1;
  }
  
  private sendOperation(op: Operation) {
    this.socket.emit('doc:operation', {
      operation: op,
      version: this.version,
    });
  }
  
  getContent(): string {
    return this.content;
  }
}

// React Hook for Collaborative Editing
function useCollaborativeEditor(documentId: string, socket: Socket | null) {
  const [content, setContent] = useState('');
  const [cursors, setCursors] = useState<Map<string, { position: number; name: string; color: string }>>(new Map());
  const docRef = useRef<OTDocument | null>(null);
  
  useEffect(() => {
    if (!socket) return;
    
    docRef.current = new OTDocument(socket);
    socket.emit('doc:join', documentId);
    
    // Cursor updates
    socket.on('cursors:update', (cursorData) => {
      setCursors(new Map(Object.entries(cursorData)));
    });
    
    // Content sync
    const syncInterval = setInterval(() => {
      if (docRef.current) {
        setContent(docRef.current.getContent());
      }
    }, 50);
    
    return () => {
      socket.emit('doc:leave', documentId);
      clearInterval(syncInterval);
    };
  }, [socket, documentId]);
  
  const handleChange = useCallback((
    newContent: string,
    changeStart: number,
    changeEnd: number,
    userId: string
  ) => {
    if (!docRef.current) return;
    
    const oldContent = docRef.current.getContent();
    
    if (newContent.length > oldContent.length) {
      // Insert
      const insertedText = newContent.slice(changeStart, changeStart + (newContent.length - oldContent.length));
      docRef.current.insert(changeStart, insertedText, userId);
    } else {
      // Delete
      const deletedLength = oldContent.length - newContent.length;
      docRef.current.delete(changeStart, deletedLength, userId);
    }
    
    setContent(newContent);
  }, []);
  
  const updateCursor = useCallback((position: number) => {
    socket?.emit('cursor:update', { position });
  }, [socket]);
  
  return { content, cursors, handleChange, updateCursor };
}
```

## 12.4 Live Notifications System

```typescript
interface Notification {
  id: string;
  type: 'info' | 'success' | 'warning' | 'error';
  title: string;
  message: string;
  timestamp: number;
  read: boolean;
  actionUrl?: string;
}

class NotificationManager {
  private socket: Socket;
  private notifications: Notification[] = [];
  private listeners: Set<(notifications: Notification[]) => void> = new Set();
  private unreadCount = 0;
  
  constructor(socket: Socket) {
    this.socket = socket;
    this.setupListeners();
  }
  
  private setupListeners() {
    this.socket.on('notification:new', (notification: Notification) => {
      this.notifications.unshift(notification);
      this.unreadCount++;
      this.notify();
      this.showBrowserNotification(notification);
    });
    
    this.socket.on('notification:list', (notifications: Notification[]) => {
      this.notifications = notifications;
      this.unreadCount = notifications.filter(n => !n.read).length;
      this.notify();
    });
  }
  
  private async showBrowserNotification(notification: Notification) {
    if (Notification.permission === 'granted' && document.hidden) {
      new Notification(notification.title, {
        body: notification.message,
        icon: '/notification-icon.png',
      });
    }
  }
  
  async requestPermission() {
    if ('Notification' in window) {
      const permission = await Notification.requestPermission();
      return permission === 'granted';
    }
    return false;
  }
  
  markAsRead(notificationId: string) {
    const notification = this.notifications.find(n => n.id === notificationId);
    if (notification && !notification.read) {
      notification.read = true;
      this.unreadCount--;
      this.socket.emit('notification:read', { id: notificationId });
      this.notify();
    }
  }
  
  markAllAsRead() {
    this.notifications.forEach(n => { n.read = true; });
    this.unreadCount = 0;
    this.socket.emit('notification:readAll');
    this.notify();
  }
  
  subscribe(listener: (notifications: Notification[]) => void) {
    this.listeners.add(listener);
    listener(this.notifications);
    return () => this.listeners.delete(listener);
  }
  
  private notify() {
    this.listeners.forEach(listener => listener(this.notifications));
  }
  
  getUnreadCount() {
    return this.unreadCount;
  }
}

// React Hook
function useNotifications(socket: Socket | null) {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const managerRef = useRef<NotificationManager | null>(null);
  
  useEffect(() => {
    if (!socket) return;
    
    managerRef.current = new NotificationManager(socket);
    
    const unsubscribe = managerRef.current.subscribe(notifs => {
      setNotifications(notifs);
      setUnreadCount(managerRef.current?.getUnreadCount() || 0);
    });
    
    return unsubscribe;
  }, [socket]);
  
  const markAsRead = useCallback((id: string) => {
    managerRef.current?.markAsRead(id);
  }, []);
  
  const markAllAsRead = useCallback(() => {
    managerRef.current?.markAllAsRead();
  }, []);
  
  const requestPermission = useCallback(async () => {
    return managerRef.current?.requestPermission() ?? false;
  }, []);
  
  return {
    notifications,
    unreadCount,
    markAsRead,
    markAllAsRead,
    requestPermission,
  };
}

// Notification Bell Component
function NotificationBell() {
  const { socket } = useSocketContext();
  const { notifications, unreadCount, markAsRead, markAllAsRead, requestPermission } = useNotifications(socket);
  const [isOpen, setIsOpen] = useState(false);
  
  useEffect(() => {
    requestPermission();
  }, []);
  
  return (
    <div className="notification-bell">
      <button onClick={() => setIsOpen(!isOpen)}>
        🔔
        {unreadCount > 0 && (
          <span className="badge">{unreadCount}</span>
        )}
      </button>
      
      {isOpen && (
        <div className="notification-dropdown">
          <div className="header">
            <span>Notifications</span>
            <button onClick={markAllAsRead}>Mark all read</button>
          </div>
          
          <div className="notification-list">
            {notifications.map(notification => (
              <NotificationItem
                key={notification.id}
                notification={notification}
                onRead={() => markAsRead(notification.id)}
              />
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

## 12.5 Real-Time Analytics Dashboard

```typescript
interface MetricUpdate {
  name: string;
  value: number;
  timestamp: number;
}

interface DashboardState {
  activeUsers: number;
  requestsPerSecond: number;
  errorRate: number;
  latencyP99: number;
  timeSeriesData: MetricUpdate[];
}

function useRealtimeDashboard() {
  const [state, setState] = useState<DashboardState>({
    activeUsers: 0,
    requestsPerSecond: 0,
    errorRate: 0,
    latencyP99: 0,
    timeSeriesData: [],
  });
  
  useEffect(() => {
    const eventSource = new EventSource('/api/metrics/stream');
    
    eventSource.addEventListener('metrics', (event) => {
      const data = JSON.parse(event.data);
      
      setState(prev => ({
        ...prev,
        activeUsers: data.activeUsers,
        requestsPerSecond: data.rps,
        errorRate: data.errorRate,
        latencyP99: data.latencyP99,
        timeSeriesData: [
          ...prev.timeSeriesData.slice(-59),
          {
            name: new Date().toLocaleTimeString(),
            value: data.rps,
            timestamp: Date.now(),
          },
        ],
      }));
    });
    
    eventSource.onerror = () => {
      console.error('SSE connection error');
    };
    
    return () => eventSource.close();
  }, []);
  
  return state;
}

function MetricsDashboard() {
  const { activeUsers, requestsPerSecond, errorRate, latencyP99, timeSeriesData } = useRealtimeDashboard();
  
  return (
    <div className="dashboard">
      <div className="metrics-grid">
        <MetricCard
          title="Active Users"
          value={activeUsers}
          icon="👥"
        />
        <MetricCard
          title="Requests/sec"
          value={requestsPerSecond}
          icon="📊"
        />
        <MetricCard
          title="Error Rate"
          value={`${errorRate.toFixed(2)}%`}
          icon="⚠️"
          alert={errorRate > 1}
        />
        <MetricCard
          title="P99 Latency"
          value={`${latencyP99}ms`}
          icon="⏱️"
        />
      </div>
      
      <div className="chart">
        <LiveChart data={timeSeriesData} />
      </div>
    </div>
  );
}
```

---

# 13. Interview Questions

## 13.1 Conceptual Questions

### Q1: What is the difference between WebSocket and HTTP?

**Answer:**
```typescript
const COMPARISON = {
  HTTP: {
    connection: 'Request-Response (short-lived)',
    direction: 'Client initiates, server responds',
    overhead: 'Headers sent with each request (~700-800 bytes)',
    useCase: 'Traditional web pages, REST APIs',
    state: 'Stateless',
  },
  
  WebSocket: {
    connection: 'Persistent bidirectional',
    direction: 'Both client and server can initiate',
    overhead: 'Minimal frame overhead (2-14 bytes)',
    useCase: 'Real-time apps, chat, gaming, live updates',
    state: 'Stateful',
  },
};

// WebSocket flow:
// 1. HTTP handshake (Upgrade request)
// 2. Connection upgraded to WebSocket
// 3. Full-duplex communication
// 4. Either party can send messages anytime
// 5. Close handshake

// Key difference: WebSocket maintains open connection
// allowing server to push data without client request
```

### Q2: Explain the WebSocket handshake process

**Answer:**
```typescript
// Step 1: Client sends HTTP Upgrade request
const clientRequest = `
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
`;

// Step 2: Server validates and responds
const serverResponse = `
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
`;

// The Sec-WebSocket-Accept is computed as:
// Base64(SHA1(Sec-WebSocket-Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))

// After 101 response, the protocol switches from HTTP to WebSocket
// and the TCP connection remains open for bidirectional communication

function computeAcceptKey(clientKey: string): string {
  const GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
  const hash = crypto.createHash('sha1');
  hash.update(clientKey + GUID);
  return hash.digest('base64');
}
```

### Q3: What are WebSocket close codes?

**Answer:**
```typescript
const WEBSOCKET_CLOSE_CODES = {
  // Normal closures
  1000: 'Normal Closure - connection completed successfully',
  1001: 'Going Away - page navigation or server shutdown',
  
  // Protocol errors
  1002: 'Protocol Error - invalid frame received',
  1003: 'Unsupported Data - received data type not supported',
  
  // Reserved
  1004: 'Reserved',
  1005: 'No Status Received - no close code provided (internal)',
  1006: 'Abnormal Closure - no close frame (connection dropped)',
  
  // Application errors
  1007: 'Invalid Payload Data - message data inconsistent',
  1008: 'Policy Violation - message violates policy',
  1009: 'Message Too Big - message exceeds size limit',
  1010: 'Mandatory Extension - required extension not negotiated',
  1011: 'Internal Error - unexpected server error',
  
  // TLS
  1015: 'TLS Handshake - TLS failure (internal only)',
  
  // Custom codes (4000-4999 available for applications)
  4000: 'Custom application error',
};

// How to use:
ws.close(1000, 'User logged out');

ws.onclose = (event) => {
  console.log(`Closed: ${event.code} - ${event.reason}`);
  
  if (event.code === 1006) {
    // Connection dropped unexpectedly - reconnect
    scheduleReconnect();
  }
};
```

### Q4: What is Socket.io and how is it different from WebSocket?

**Answer:**
```typescript
// Socket.io is a library built ON TOP of WebSocket
// It provides additional features not in the WebSocket spec

const SOCKETIO_FEATURES = {
  // Transport fallback
  transports: ['websocket', 'polling'],
  // If WebSocket fails, falls back to HTTP long-polling
  
  // Automatic reconnection
  reconnection: {
    enabled: true,
    attempts: Infinity,
    delay: 1000,
    maxDelay: 5000,
    exponentialBackoff: true,
  },
  
  // Room & Namespace support
  rooms: 'Group broadcasting to specific clients',
  namespaces: 'Separate concerns on same connection',
  
  // Acknowledgments
  acks: 'Callback when message is received by server',
  
  // Binary support
  binary: 'Automatic ArrayBuffer/Blob handling',
  
  // Custom events
  events: 'Named events instead of single onmessage',
};

// IMPORTANT: Socket.io is NOT compatible with plain WebSocket!
// Socket.io client can only connect to Socket.io server
// and vice versa

// Socket.io adds its own protocol on top of WebSocket:
// - Packet types (CONNECT, DISCONNECT, EVENT, ACK, ERROR)
// - Engine.IO for transport layer
// - Custom binary encoding
```

### Q5: What is Server-Sent Events (SSE)?

**Answer:**
```typescript
// SSE = One-way server-to-client streaming over HTTP

const SSE_CHARACTERISTICS = {
  direction: 'Server → Client only (unidirectional)',
  protocol: 'HTTP (text/event-stream)',
  reconnection: 'Automatic (sends Last-Event-ID)',
  format: 'Text-based (not binary)',
  browser: 'EventSource API',
  maxConnections: '6 per domain (HTTP/1.1), unlimited (HTTP/2)',
};

// When to use SSE vs WebSocket:
const USE_SSE_WHEN = [
  'Only server needs to push data to client',
  'Simple notification/feed updates',
  'Want auto-reconnection built-in',
  'HTTP/2 available (no connection limit)',
  'Stock tickers, news feeds, social updates',
];

const USE_WEBSOCKET_WHEN = [
  'Need bidirectional communication',
  'Client needs to send data frequently',
  'Binary data transfer needed',
  'Gaming, chat, collaborative tools',
];

// SSE Server format:
// data: Hello\n\n              // Simple message
// event: custom\ndata: Hi\n\n  // Named event
// id: 123\ndata: msg\n\n       // With ID for reconnection
// retry: 5000\n\n              // Set reconnect delay
```

### Q6: How do you handle authentication with WebSockets?

**Answer:**
```typescript
// Method 1: Token in query string (less secure)
const ws = new WebSocket(`wss://api.com/ws?token=${accessToken}`);
// Downside: Token visible in URL, logs, browser history

// Method 2: Token in first message (recommended)
const ws = new WebSocket('wss://api.com/ws');
ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    token: accessToken,
  }));
};
// Server validates before allowing other messages

// Method 3: Cookie-based (for same-origin)
// Cookies automatically sent with WebSocket handshake
const ws = new WebSocket('wss://same-origin.com/ws');
// Server validates session cookie

// Method 4: Socket.io auth option (best for Socket.io)
const socket = io('https://api.com', {
  auth: { token: accessToken },
});

socket.on('connect_error', (err) => {
  if (err.message === 'unauthorized') {
    // Refresh token and reconnect
    refreshToken().then(newToken => {
      socket.auth = { token: newToken };
      socket.connect();
    });
  }
});

// Token refresh during connection:
socket.on('auth:expiring', async () => {
  const newToken = await refreshToken();
  socket.emit('auth:refresh', { token: newToken });
});
```

### Q7: How do you handle reconnection in WebSockets?

**Answer:**
```typescript
class ReconnectingWebSocket {
  private ws: WebSocket | null = null;
  private url: string;
  private reconnectAttempts = 0;
  private maxAttempts = 10;
  private baseDelay = 1000;
  private maxDelay = 30000;
  private isIntentionallyClosed = false;
  
  constructor(url: string) {
    this.url = url;
  }
  
  connect() {
    this.isIntentionallyClosed = false;
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0; // Reset on successful connection
    };
    
    this.ws.onclose = (event) => {
      if (!this.isIntentionallyClosed && event.code !== 1000) {
        this.scheduleReconnect();
      }
    };
  }
  
  private scheduleReconnect() {
    if (this.reconnectAttempts >= this.maxAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }
    
    // Exponential backoff with jitter
    const delay = Math.min(
      this.baseDelay * Math.pow(2, this.reconnectAttempts) +
      Math.random() * 1000,
      this.maxDelay
    );
    
    this.reconnectAttempts++;
    
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
    
    setTimeout(() => {
      if (!this.isIntentionallyClosed) {
        this.connect();
      }
    }, delay);
  }
  
  close() {
    this.isIntentionallyClosed = true;
    this.ws?.close(1000, 'Client disconnect');
  }
}

// Key principles:
// 1. Exponential backoff - don't hammer the server
// 2. Jitter - prevent thundering herd
// 3. Max attempts - eventually give up
// 4. Reset on success - fresh start
// 5. Distinguish intentional from unintentional closes
```

### Q8: What is the difference between polling, long polling, and WebSockets?

**Answer:**
```typescript
// Regular Polling
// Client repeatedly asks server for updates at fixed intervals
async function regularPolling() {
  while (true) {
    const data = await fetch('/api/updates');
    handleUpdate(data);
    await sleep(5000); // Poll every 5 seconds
  }
}
// Wasteful if no updates, high latency (up to interval)

// Long Polling
// Client asks, server holds request until data available
async function longPolling() {
  while (true) {
    // Server holds this request until data or timeout
    const data = await fetch('/api/updates?timeout=30000');
    handleUpdate(data);
    // Immediately poll again
  }
}
// Lower latency, but still creates new connection each time

// WebSocket
// Persistent bidirectional connection
const ws = new WebSocket('wss://api.com/updates');
ws.onmessage = (event) => handleUpdate(JSON.parse(event.data));
// Lowest latency, most efficient for frequent updates

const COMPARISON = {
  polling: {
    latency: 'High (up to poll interval)',
    connections: 'New connection each poll',
    efficiency: 'Low (many empty responses)',
    complexity: 'Simple',
  },
  longPolling: {
    latency: 'Medium (instant when data ready)',
    connections: 'New connection after each response',
    efficiency: 'Medium (no empty responses)',
    complexity: 'Medium (timeout handling)',
  },
  websocket: {
    latency: 'Low (instant)',
    connections: 'Single persistent connection',
    efficiency: 'High (minimal overhead)',
    complexity: 'Higher (connection management)',
  },
};
```

## 13.2 Practical Questions

### Q9: How would you implement a typing indicator?

**Answer:**
```typescript
function useTypingIndicator(socket: Socket, roomId: string) {
  const [typingUsers, setTypingUsers] = useState<string[]>([]);
  const typingTimeoutRef = useRef<NodeJS.Timer>();
  const isTypingRef = useRef(false);
  
  useEffect(() => {
    socket.on('typing:update', ({ userId, isTyping }) => {
      setTypingUsers(prev => {
        if (isTyping && !prev.includes(userId)) {
          return [...prev, userId];
        }
        if (!isTyping) {
          return prev.filter(id => id !== userId);
        }
        return prev;
      });
    });
    
    return () => {
      socket.off('typing:update');
    };
  }, [socket]);
  
  const handleTyping = useCallback(() => {
    // Send typing start only if not already typing
    if (!isTypingRef.current) {
      isTypingRef.current = true;
      socket.emit('typing:start', { roomId });
    }
    
    // Reset timeout on each keystroke
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }
    
    // Stop typing after 2 seconds of inactivity
    typingTimeoutRef.current = setTimeout(() => {
      isTypingRef.current = false;
      socket.emit('typing:stop', { roomId });
    }, 2000);
  }, [socket, roomId]);
  
  const stopTyping = useCallback(() => {
    if (isTypingRef.current) {
      isTypingRef.current = false;
      socket.emit('typing:stop', { roomId });
      if (typingTimeoutRef.current) {
        clearTimeout(typingTimeoutRef.current);
      }
    }
  }, [socket, roomId]);
  
  return { typingUsers, handleTyping, stopTyping };
}

// Key points:
// 1. Debounce typing events to avoid spamming
// 2. Auto-stop after inactivity
// 3. Don't send duplicate start/stop events
// 4. Clean up on unmount
```

### Q10: How would you implement message delivery status (sent, delivered, read)?

**Answer:**
```typescript
type MessageStatus = 'sending' | 'sent' | 'delivered' | 'read';

interface Message {
  id: string;
  text: string;
  senderId: string;
  status: MessageStatus;
  timestamp: number;
}

function useMessageStatus(socket: Socket, userId: string) {
  const [messages, setMessages] = useState<Message[]>([]);
  
  const updateMessageStatus = (messageId: string, status: MessageStatus) => {
    setMessages(prev =>
      prev.map(msg =>
        msg.id === messageId ? { ...msg, status } : msg
      )
    );
  };
  
  useEffect(() => {
    // Server confirms message was saved
    socket.on('message:sent', ({ tempId, id }) => {
      setMessages(prev =>
        prev.map(msg =>
          msg.id === tempId ? { ...msg, id, status: 'sent' } : msg
        )
      );
    });
    
    // Recipient received the message
    socket.on('message:delivered', ({ messageId }) => {
      updateMessageStatus(messageId, 'delivered');
    });
    
    // Recipient read the message
    socket.on('message:read', ({ messageId }) => {
      updateMessageStatus(messageId, 'read');
    });
    
    // Incoming message - auto-mark as delivered
    socket.on('message:new', (message: Message) => {
      setMessages(prev => [...prev, message]);
      socket.emit('message:delivered', { messageId: message.id });
    });
    
    return () => {
      socket.off('message:sent');
      socket.off('message:delivered');
      socket.off('message:read');
      socket.off('message:new');
    };
  }, [socket]);
  
  const sendMessage = (text: string) => {
    const tempId = `temp-${Date.now()}`;
    
    // Optimistic update
    const newMessage: Message = {
      id: tempId,
      text,
      senderId: userId,
      status: 'sending',
      timestamp: Date.now(),
    };
    
    setMessages(prev => [...prev, newMessage]);
    
    socket.emit('message:send', { text, tempId });
  };
  
  const markAsRead = (messageId: string) => {
    socket.emit('message:read', { messageId });
  };
  
  return { messages, sendMessage, markAsRead };
}

// Status flow:
// 1. sending - Message is being sent
// 2. sent - Server received and stored message
// 3. delivered - Recipient's client received message
// 4. read - Recipient viewed the message
```

### Q11: How do you handle offline messages?

**Answer:**
```typescript
interface PendingMessage {
  id: string;
  text: string;
  timestamp: number;
  retries: number;
}

class OfflineMessageQueue {
  private queue: PendingMessage[] = [];
  private isOnline = navigator.onLine;
  private socket: Socket | null = null;
  private storage = localStorage;
  
  constructor() {
    this.loadFromStorage();
    this.setupListeners();
  }
  
  private loadFromStorage() {
    const stored = this.storage.getItem('pendingMessages');
    if (stored) {
      this.queue = JSON.parse(stored);
    }
  }
  
  private saveToStorage() {
    this.storage.setItem('pendingMessages', JSON.stringify(this.queue));
  }
  
  private setupListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.flushQueue();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }
  
  setSocket(socket: Socket) {
    this.socket = socket;
    
    socket.on('connect', () => {
      this.flushQueue();
    });
    
    // Confirmation that message was stored
    socket.on('message:ack', ({ tempId, id }) => {
      this.removeFromQueue(tempId);
    });
  }
  
  send(message: Omit<PendingMessage, 'retries'>) {
    const pendingMessage: PendingMessage = {
      ...message,
      retries: 0,
    };
    
    if (this.isOnline && this.socket?.connected) {
      this.socket.emit('message:send', message);
    }
    
    // Always add to queue until acknowledged
    this.queue.push(pendingMessage);
    this.saveToStorage();
  }
  
  private removeFromQueue(id: string) {
    this.queue = this.queue.filter(m => m.id !== id);
    this.saveToStorage();
  }
  
  private flushQueue() {
    if (!this.socket?.connected) return;
    
    for (const message of this.queue) {
      if (message.retries < 3) {
        this.socket.emit('message:send', message);
        message.retries++;
      } else {
        // Give up after 3 retries
        this.removeFromQueue(message.id);
        this.notifyFailed(message);
      }
    }
    
    this.saveToStorage();
  }
  
  private notifyFailed(message: PendingMessage) {
    console.error('Message failed to send:', message);
  }
  
  getPendingCount(): number {
    return this.queue.length;
  }
}

// Key points:
// 1. Persist queue to localStorage for page refreshes
// 2. Listen for online/offline events
// 3. Retry on reconnection
// 4. Remove only after server acknowledgment
// 5. Limit retries to avoid infinite loops
```

### Q12: How do you scale WebSocket connections?

**Answer:**
```typescript
// WebSocket scaling challenges and solutions:

const SCALING_STRATEGIES = {
  // 1. Horizontal scaling with sticky sessions
  stickySessions: {
    description: 'Route same client to same server',
    implementation: 'Load balancer with IP/cookie affinity',
    pros: 'Simple, session state stays local',
    cons: 'Uneven distribution, single point of failure',
  },
  
  // 2. Pub/Sub for cross-server messaging
  pubSub: {
    description: 'Redis/Kafka for inter-server communication',
    implementation: 'Each server subscribes to relevant channels',
    pros: 'Any server can broadcast to any client',
    cons: 'Additional infrastructure, latency',
  },
  
  // 3. Dedicated connection servers
  connectionServers: {
    description: 'Separate WebSocket servers from app logic',
    implementation: 'WebSocket gateways + stateless backends',
    pros: 'Independent scaling, specialized optimization',
    cons: 'Increased complexity',
  },
};

// Redis Pub/Sub for Socket.io scaling
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const pubClient = createClient({ url: 'redis://localhost:6379' });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
});

// Now socket.emit() reaches all servers
// Room broadcasts work across multiple servers

// Connection limits per process:
const LIMITS = {
  maxConnectionsPerProcess: 10000, // Typical limit
  memoryPerConnection: '2-5KB',    // Depends on message buffering
  fileDescriptorsNeeded: 1,         // Per connection
  
  // To support more connections:
  optimizations: [
    'Increase ulimit -n (file descriptors)',
    'Use connection pooling where possible',
    'Implement message batching',
    'Use binary protocols (msgpack, protobuf)',
  ],
};
```

### Q13: What is the difference between rooms and namespaces in Socket.io?

**Answer:**
```typescript
// Namespaces: Separation of concerns at connection level
// Each namespace = separate connection (multiplexed)

const mainSocket = io('/');           // Default namespace
const chatSocket = io('/chat');       // Chat namespace
const adminSocket = io('/admin');     // Admin namespace

// Use namespaces when:
// - Different authentication requirements
// - Completely separate features
// - Different connection logic

// Rooms: Grouping within a namespace
// All rooms share same connection

// Server-side:
socket.join('room-123');               // Add to room
socket.leave('room-123');              // Leave room
io.to('room-123').emit('message', data); // Broadcast to room

// Use rooms when:
// - Same feature, different groups (chat rooms)
// - Temporary groupings (game matches)
// - Selective broadcasting

const COMPARISON = {
  namespaces: {
    level: 'Connection level',
    scope: 'Global feature separation',
    auth: 'Can have different auth per namespace',
    overhead: 'New connection handshake each',
    example: '/chat, /notifications, /admin',
  },
  
  rooms: {
    level: 'Within namespace',
    scope: 'Group clients dynamically',
    auth: 'Same auth as namespace',
    overhead: 'Just internal bookkeeping',
    example: 'chat-room-123, game-456',
  },
};
```

---

## 13.3 Advanced Questions

### Q14: How would you implement real-time collaborative cursors?

**Answer:**
```typescript
interface CursorPosition {
  userId: string;
  userName: string;
  color: string;
  x: number;
  y: number;
  elementId?: string;
  timestamp: number;
}

function useCollaborativeCursors(socket: Socket, userId: string) {
  const [cursors, setCursors] = useState<Map<string, CursorPosition>>(new Map());
  const throttledEmit = useRef<NodeJS.Timer>();
  
  useEffect(() => {
    // Receive other users' cursors
    socket.on('cursor:moved', (position: CursorPosition) => {
      setCursors(prev => {
        const next = new Map(prev);
        next.set(position.userId, position);
        return next;
      });
    });
    
    // User left
    socket.on('cursor:left', ({ userId }) => {
      setCursors(prev => {
        const next = new Map(prev);
        next.delete(userId);
        return next;
      });
    });
    
    // Initial positions
    socket.on('cursors:sync', (positions: CursorPosition[]) => {
      setCursors(new Map(positions.map(p => [p.userId, p])));
    });
    
    return () => {
      socket.off('cursor:moved');
      socket.off('cursor:left');
      socket.off('cursors:sync');
    };
  }, [socket]);
  
  const updateCursor = useCallback((x: number, y: number, elementId?: string) => {
    // Throttle to 60fps max (16ms)
    if (throttledEmit.current) return;
    
    throttledEmit.current = setTimeout(() => {
      throttledEmit.current = undefined;
    }, 16);
    
    socket.volatile.emit('cursor:move', {
      userId,
      x,
      y,
      elementId,
      timestamp: Date.now(),
    });
  }, [socket, userId]);
  
  // Remove stale cursors
  useEffect(() => {
    const cleanup = setInterval(() => {
      const now = Date.now();
      setCursors(prev => {
        const next = new Map(prev);
        for (const [id, cursor] of next) {
          if (now - cursor.timestamp > 5000) {
            next.delete(id);
          }
        }
        return next;
      });
    }, 1000);
    
    return () => clearInterval(cleanup);
  }, []);
  
  return { cursors, updateCursor };
}

// Cursor Component
function RemoteCursor({ cursor }: { cursor: CursorPosition }) {
  return (
    <div
      className="remote-cursor"
      style={{
        position: 'absolute',
        left: cursor.x,
        top: cursor.y,
        pointerEvents: 'none',
        transition: 'left 50ms, top 50ms',
      }}
    >
      <svg width="24" height="24" viewBox="0 0 24 24">
        <path
          d="M5 3l14 7-7 3-3 7z"
          fill={cursor.color}
          stroke="white"
        />
      </svg>
      <span
        className="cursor-label"
        style={{ backgroundColor: cursor.color }}
      >
        {cursor.userName}
      </span>
    </div>
  );
}

// Key optimizations:
// 1. Throttle cursor updates (60fps max)
// 2. Use volatile emit (ok to drop some updates)
// 3. Smooth transitions with CSS
// 4. Clean up stale cursors
// 5. Include timestamp for ordering
```

### Q15: How do you prevent message flooding/spam?

**Answer:**
```typescript
// Client-side rate limiting
class RateLimiter {
  private tokens: number;
  private maxTokens: number;
  private refillRate: number; // tokens per second
  private lastRefill: number = Date.now();
  
  constructor(maxTokens: number, refillRate: number) {
    this.tokens = maxTokens;
    this.maxTokens = maxTokens;
    this.refillRate = refillRate;
  }
  
  tryConsume(tokens: number = 1): boolean {
    this.refill();
    
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    
    return false;
  }
  
  private refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(
      this.maxTokens,
      this.tokens + elapsed * this.refillRate
    );
    this.lastRefill = now;
  }
  
  getWaitTime(): number {
    return Math.ceil((1 - this.tokens) / this.refillRate * 1000);
  }
}

// Usage in chat
function useChatWithRateLimit(socket: Socket) {
  const limiter = useRef(new RateLimiter(5, 1)); // 5 messages, 1 per second refill
  const [cooldown, setCooldown] = useState(0);
  
  const sendMessage = useCallback((text: string) => {
    if (limiter.current.tryConsume()) {
      socket.emit('message:send', { text });
      return true;
    }
    
    const wait = limiter.current.getWaitTime();
    setCooldown(wait);
    
    setTimeout(() => setCooldown(0), wait);
    return false;
  }, [socket]);
  
  return { sendMessage, cooldown };
}

// Server-side rate limiting (important!)
// Client can bypass client-side limits

// Example with socket.io middleware:
io.use((socket, next) => {
  socket.rateLimiter = new RateLimiter(10, 2);
  next();
});

socket.on('message:send', (data, callback) => {
  if (!socket.rateLimiter.tryConsume()) {
    callback({ error: 'rate_limit', wait: socket.rateLimiter.getWaitTime() });
    return;
  }
  
  // Process message...
});

// Additional protections:
const ANTI_SPAM_MEASURES = [
  'Message length limits',
  'Duplicate message detection',
  'Slow mode (min time between messages)',
  'Temporary mute on violation',
  'IP-based limits',
  'Account age requirements',
  'CAPTCHA after threshold',
];
```

---

# 14. Coding Challenges

## 14.1 Implement a WebSocket Client with Auto-Reconnect

```typescript
// Challenge: Create a robust WebSocket client

interface WebSocketClientOptions {
  url: string;
  maxReconnectAttempts?: number;
  reconnectInterval?: number;
  heartbeatInterval?: number;
  onMessage?: (data: any) => void;
  onConnect?: () => void;
  onDisconnect?: (code: number, reason: string) => void;
  onError?: (error: Event) => void;
}

class RobustWebSocketClient {
  private ws: WebSocket | null = null;
  private options: Required<WebSocketClientOptions>;
  private reconnectAttempts = 0;
  private reconnectTimer: NodeJS.Timer | null = null;
  private heartbeatTimer: NodeJS.Timer | null = null;
  private isIntentionallyClosed = false;
  private messageQueue: any[] = [];
  
  constructor(options: WebSocketClientOptions) {
    this.options = {
      maxReconnectAttempts: 10,
      reconnectInterval: 1000,
      heartbeatInterval: 30000,
      onMessage: () => {},
      onConnect: () => {},
      onDisconnect: () => {},
      onError: () => {},
      ...options,
    };
  }
  
  connect(): void {
    if (this.ws?.readyState === WebSocket.OPEN) return;
    
    this.isIntentionallyClosed = false;
    this.ws = new WebSocket(this.options.url);
    
    this.ws.onopen = () => {
      console.log('[WS] Connected');
      this.reconnectAttempts = 0;
      this.startHeartbeat();
      this.flushQueue();
      this.options.onConnect();
    };
    
    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        
        // Handle pong
        if (data.type === 'pong') {
          return;
        }
        
        this.options.onMessage(data);
      } catch {
        this.options.onMessage(event.data);
      }
    };
    
    this.ws.onerror = (error) => {
      console.error('[WS] Error:', error);
      this.options.onError(error);
    };
    
    this.ws.onclose = (event) => {
      console.log(`[WS] Closed: ${event.code} ${event.reason}`);
      this.stopHeartbeat();
      this.options.onDisconnect(event.code, event.reason);
      
      if (!this.isIntentionallyClosed) {
        this.scheduleReconnect();
      }
    };
  }
  
  private scheduleReconnect(): void {
    if (this.reconnectAttempts >= this.options.maxReconnectAttempts) {
      console.error('[WS] Max reconnect attempts reached');
      return;
    }
    
    const delay = this.options.reconnectInterval * Math.pow(2, this.reconnectAttempts);
    this.reconnectAttempts++;
    
    console.log(`[WS] Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
    
    this.reconnectTimer = setTimeout(() => {
      this.connect();
    }, delay);
  }
  
  private startHeartbeat(): void {
    this.heartbeatTimer = setInterval(() => {
      this.send({ type: 'ping' });
    }, this.options.heartbeatInterval);
  }
  
  private stopHeartbeat(): void {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }
  }
  
  private flushQueue(): void {
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift();
      this.send(message);
    }
  }
  
  send(data: any): boolean {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(typeof data === 'string' ? data : JSON.stringify(data));
      return true;
    }
    
    // Queue if not connected
    this.messageQueue.push(data);
    return false;
  }
  
  disconnect(): void {
    this.isIntentionallyClosed = true;
    
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
    }
    
    this.stopHeartbeat();
    this.ws?.close(1000, 'Client disconnect');
    this.ws = null;
  }
  
  get isConnected(): boolean {
    return this.ws?.readyState === WebSocket.OPEN;
  }
}
```

## 14.2 Implement Event Emitter

```typescript
// Challenge: Create a type-safe event emitter

type EventMap = Record<string, any>;
type EventHandler<T = any> = (data: T) => void;

class TypedEventEmitter<Events extends EventMap> {
  private handlers: Map<keyof Events, Set<EventHandler>> = new Map();
  
  on<K extends keyof Events>(event: K, handler: EventHandler<Events[K]>): () => void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }
    
    this.handlers.get(event)!.add(handler);
    
    // Return unsubscribe function
    return () => {
      this.handlers.get(event)?.delete(handler);
    };
  }
  
  once<K extends keyof Events>(event: K, handler: EventHandler<Events[K]>): () => void {
    const wrapper = (data: Events[K]) => {
      handler(data);
      this.off(event, wrapper);
    };
    
    return this.on(event, wrapper);
  }
  
  off<K extends keyof Events>(event: K, handler: EventHandler<Events[K]>): void {
    this.handlers.get(event)?.delete(handler);
  }
  
  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    const eventHandlers = this.handlers.get(event);
    
    if (eventHandlers) {
      eventHandlers.forEach(handler => {
        try {
          handler(data);
        } catch (error) {
          console.error(`Error in event handler for ${String(event)}:`, error);
        }
      });
    }
  }
  
  removeAllListeners<K extends keyof Events>(event?: K): void {
    if (event) {
      this.handlers.delete(event);
    } else {
      this.handlers.clear();
    }
  }
  
  listenerCount<K extends keyof Events>(event: K): number {
    return this.handlers.get(event)?.size ?? 0;
  }
}

// Usage
interface ChatEvents {
  'message': { userId: string; text: string };
  'user:joined': { userId: string; name: string };
  'user:left': { userId: string };
  'typing': { userId: string; isTyping: boolean };
}

const emitter = new TypedEventEmitter<ChatEvents>();

// Type-safe event handling
emitter.on('message', (data) => {
  console.log(data.userId, data.text); // TypeScript knows the shape
});

const unsubscribe = emitter.on('user:joined', (data) => {
  console.log(`${data.name} joined`);
});

// Emit with type checking
emitter.emit('message', { userId: '123', text: 'Hello' });

// Cleanup
unsubscribe();
```

## 14.3 Implement Debounced Typing Indicator

```typescript
// Challenge: Create efficient typing indicator

interface TypingState {
  isTyping: boolean;
  typingUsers: Set<string>;
}

function createTypingIndicator(
  sendTyping: (isTyping: boolean) => void,
  debounceMs: number = 2000
) {
  let isCurrentlyTyping = false;
  let debounceTimer: NodeJS.Timer | null = null;
  
  const startTyping = () => {
    if (!isCurrentlyTyping) {
      isCurrentlyTyping = true;
      sendTyping(true);
    }
    
    // Reset debounce timer
    if (debounceTimer) {
      clearTimeout(debounceTimer);
    }
    
    debounceTimer = setTimeout(() => {
      stopTyping();
    }, debounceMs);
  };
  
  const stopTyping = () => {
    if (isCurrentlyTyping) {
      isCurrentlyTyping = false;
      sendTyping(false);
    }
    
    if (debounceTimer) {
      clearTimeout(debounceTimer);
      debounceTimer = null;
    }
  };
  
  const cleanup = () => {
    stopTyping();
  };
  
  return { startTyping, stopTyping, cleanup };
}

// React Hook
function useTypingIndicator(socket: Socket, roomId: string) {
  const [typingUsers, setTypingUsers] = useState<Map<string, string>>(new Map());
  const typingTimeouts = useRef<Map<string, NodeJS.Timer>>(new Map());
  
  const indicator = useMemo(
    () => createTypingIndicator((isTyping) => {
      socket.emit('typing', { roomId, isTyping });
    }),
    [socket, roomId]
  );
  
  useEffect(() => {
    socket.on('typing', ({ userId, userName, isTyping }) => {
      setTypingUsers(prev => {
        const next = new Map(prev);
        
        // Clear existing timeout
        const existingTimeout = typingTimeouts.current.get(userId);
        if (existingTimeout) {
          clearTimeout(existingTimeout);
        }
        
        if (isTyping) {
          next.set(userId, userName);
          
          // Auto-remove after 5 seconds (fallback)
          const timeout = setTimeout(() => {
            setTypingUsers(p => {
              const n = new Map(p);
              n.delete(userId);
              return n;
            });
            typingTimeouts.current.delete(userId);
          }, 5000);
          
          typingTimeouts.current.set(userId, timeout);
        } else {
          next.delete(userId);
          typingTimeouts.current.delete(userId);
        }
        
        return next;
      });
    });
    
    return () => {
      indicator.cleanup();
      socket.off('typing');
      typingTimeouts.current.forEach(clearTimeout);
    };
  }, [socket, indicator]);
  
  const typingText = useMemo(() => {
    const users = Array.from(typingUsers.values());
    
    if (users.length === 0) return null;
    if (users.length === 1) return `${users[0]} is typing...`;
    if (users.length === 2) return `${users[0]} and ${users[1]} are typing...`;
    return `${users[0]} and ${users.length - 1} others are typing...`;
  }, [typingUsers]);
  
  return {
    onKeyPress: indicator.startTyping,
    onBlur: indicator.stopTyping,
    typingText,
  };
}
```

## 14.4 Implement Message Batching

```typescript
// Challenge: Batch multiple messages into single transmission

class MessageBatcher<T> {
  private batch: T[] = [];
  private timer: NodeJS.Timer | null = null;
  private readonly maxBatchSize: number;
  private readonly maxWaitMs: number;
  private readonly onFlush: (messages: T[]) => void;
  
  constructor(options: {
    maxBatchSize?: number;
    maxWaitMs?: number;
    onFlush: (messages: T[]) => void;
  }) {
    this.maxBatchSize = options.maxBatchSize ?? 10;
    this.maxWaitMs = options.maxWaitMs ?? 100;
    this.onFlush = options.onFlush;
  }
  
  add(message: T): void {
    this.batch.push(message);
    
    // Flush immediately if batch is full
    if (this.batch.length >= this.maxBatchSize) {
      this.flush();
      return;
    }
    
    // Start timer if not already running
    if (!this.timer) {
      this.timer = setTimeout(() => {
        this.flush();
      }, this.maxWaitMs);
    }
  }
  
  flush(): void {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
    
    if (this.batch.length > 0) {
      const messages = [...this.batch];
      this.batch = [];
      this.onFlush(messages);
    }
  }
  
  clear(): void {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
    this.batch = [];
  }
  
  get pendingCount(): number {
    return this.batch.length;
  }
}

// Usage with WebSocket
const batcher = new MessageBatcher<any>({
  maxBatchSize: 20,
  maxWaitMs: 50,
  onFlush: (messages) => {
    socket.emit('messages:batch', { messages });
  },
});

// High-frequency updates (e.g., cursor positions)
function handleCursorMove(x: number, y: number) {
  batcher.add({ type: 'cursor', x, y, timestamp: Date.now() });
}

// React Hook for batched sending
function useBatchedSocket(socket: Socket) {
  const batcherRef = useRef<MessageBatcher<any>>();
  
  useEffect(() => {
    batcherRef.current = new MessageBatcher({
      maxBatchSize: 10,
      maxWaitMs: 100,
      onFlush: (messages) => {
        socket.emit('batch', messages);
      },
    });
    
    return () => {
      batcherRef.current?.flush();
    };
  }, [socket]);
  
  const send = useCallback((event: string, data: any) => {
    batcherRef.current?.add({ event, data });
  }, []);
  
  return { send };
}
```

## 14.5 Implement Presence with Heartbeat

```typescript
// Challenge: Track user presence with heartbeat

interface PresenceUser {
  userId: string;
  status: 'online' | 'away' | 'offline';
  lastSeen: number;
  metadata?: Record<string, any>;
}

class PresenceSystem {
  private users: Map<string, PresenceUser> = new Map();
  private socket: Socket;
  private heartbeatInterval: NodeJS.Timer | null = null;
  private cleanupInterval: NodeJS.Timer | null = null;
  private listeners: Set<(users: Map<string, PresenceUser>) => void> = new Set();
  
  private readonly HEARTBEAT_INTERVAL = 10000; // 10 seconds
  private readonly STALE_THRESHOLD = 30000;    // 30 seconds
  private readonly OFFLINE_THRESHOLD = 60000;  // 60 seconds
  
  constructor(socket: Socket, private userId: string) {
    this.socket = socket;
    this.setupListeners();
  }
  
  private setupListeners() {
    // Receive presence updates
    this.socket.on('presence:sync', (users: PresenceUser[]) => {
      this.users = new Map(users.map(u => [u.userId, u]));
      this.notify();
    });
    
    this.socket.on('presence:update', (user: PresenceUser) => {
      this.users.set(user.userId, user);
      this.notify();
    });
    
    this.socket.on('presence:leave', ({ userId }) => {
      this.users.delete(userId);
      this.notify();
    });
  }
  
  start() {
    // Send initial presence
    this.sendHeartbeat();
    
    // Periodic heartbeat
    this.heartbeatInterval = setInterval(() => {
      this.sendHeartbeat();
    }, this.HEARTBEAT_INTERVAL);
    
    // Cleanup stale users
    this.cleanupInterval = setInterval(() => {
      this.cleanupStaleUsers();
    }, this.HEARTBEAT_INTERVAL);
    
    // Track activity for away detection
    this.setupActivityTracking();
  }
  
  stop() {
    this.socket.emit('presence:leave', { userId: this.userId });
    
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
    }
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
    }
  }
  
  private sendHeartbeat() {
    this.socket.emit('presence:heartbeat', {
      userId: this.userId,
      status: this.getCurrentStatus(),
      timestamp: Date.now(),
    });
  }
  
  private currentStatus: 'online' | 'away' = 'online';
  private lastActivity = Date.now();
  private activityTimeout: NodeJS.Timer | null = null;
  
  private setupActivityTracking() {
    const resetActivity = () => {
      this.lastActivity = Date.now();
      
      if (this.currentStatus === 'away') {
        this.currentStatus = 'online';
        this.sendHeartbeat();
      }
      
      if (this.activityTimeout) {
        clearTimeout(this.activityTimeout);
      }
      
      // Go away after 5 minutes of inactivity
      this.activityTimeout = setTimeout(() => {
        this.currentStatus = 'away';
        this.sendHeartbeat();
      }, 5 * 60 * 1000);
    };
    
    window.addEventListener('mousemove', resetActivity);
    window.addEventListener('keypress', resetActivity);
    window.addEventListener('click', resetActivity);
    
    // Handle visibility change
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        this.currentStatus = 'away';
      } else {
        this.currentStatus = 'online';
      }
      this.sendHeartbeat();
    });
  }
  
  private getCurrentStatus(): 'online' | 'away' {
    return this.currentStatus;
  }
  
  private cleanupStaleUsers() {
    const now = Date.now();
    let changed = false;
    
    for (const [userId, user] of this.users) {
      const age = now - user.lastSeen;
      
      if (age > this.OFFLINE_THRESHOLD) {
        this.users.delete(userId);
        changed = true;
      } else if (age > this.STALE_THRESHOLD && user.status === 'online') {
        user.status = 'away';
        changed = true;
      }
    }
    
    if (changed) {
      this.notify();
    }
  }
  
  subscribe(listener: (users: Map<string, PresenceUser>) => void): () => void {
    this.listeners.add(listener);
    listener(this.users);
    return () => this.listeners.delete(listener);
  }
  
  private notify() {
    this.listeners.forEach(listener => listener(this.users));
  }
  
  getUser(userId: string): PresenceUser | undefined {
    return this.users.get(userId);
  }
  
  getOnlineUsers(): PresenceUser[] {
    return Array.from(this.users.values())
      .filter(u => u.status === 'online');
  }
  
  isOnline(userId: string): boolean {
    return this.users.get(userId)?.status === 'online';
  }
}
```

## 14.6 Implement Optimistic Updates with Rollback

```typescript
// Challenge: Implement optimistic UI with conflict resolution

interface OptimisticUpdate<T> {
  id: string;
  optimisticValue: T;
  timestamp: number;
  confirmed: boolean;
}

class OptimisticUpdateManager<T> {
  private updates: Map<string, OptimisticUpdate<T>> = new Map();
  private confirmedState: T;
  private listeners: Set<(state: T) => void> = new Set();
  private readonly TIMEOUT = 10000; // 10 second timeout
  
  constructor(initialState: T) {
    this.confirmedState = initialState;
  }
  
  applyOptimistic(updateId: string, optimisticValue: T): void {
    this.updates.set(updateId, {
      id: updateId,
      optimisticValue,
      timestamp: Date.now(),
      confirmed: false,
    });
    
    this.notify();
    
    // Auto-rollback after timeout
    setTimeout(() => {
      const update = this.updates.get(updateId);
      if (update && !update.confirmed) {
        this.rollback(updateId);
      }
    }, this.TIMEOUT);
  }
  
  confirm(updateId: string, serverValue: T): void {
    const update = this.updates.get(updateId);
    
    if (update) {
      update.confirmed = true;
      this.confirmedState = serverValue;
      this.updates.delete(updateId);
      this.notify();
    }
  }
  
  rollback(updateId: string): void {
    if (this.updates.has(updateId)) {
      this.updates.delete(updateId);
      this.notify();
    }
  }
  
  rollbackAll(): void {
    this.updates.clear();
    this.notify();
  }
  
  getState(): T {
    // Return optimistic state if pending, otherwise confirmed
    const pendingUpdates = Array.from(this.updates.values())
      .filter(u => !u.confirmed)
      .sort((a, b) => a.timestamp - b.timestamp);
    
    if (pendingUpdates.length > 0) {
      // Return the latest optimistic value
      return pendingUpdates[pendingUpdates.length - 1].optimisticValue;
    }
    
    return this.confirmedState;
  }
  
  hasPending(): boolean {
    return Array.from(this.updates.values()).some(u => !u.confirmed);
  }
  
  subscribe(listener: (state: T) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  private notify(): void {
    const state = this.getState();
    this.listeners.forEach(l => l(state));
  }
}

// React Hook
function useOptimisticMessages(socket: Socket) {
  const [messages, setMessages] = useState<Message[]>([]);
  const managerRef = useRef<OptimisticUpdateManager<Message[]>>();
  
  useEffect(() => {
    managerRef.current = new OptimisticUpdateManager<Message[]>([]);
    
    const unsubscribe = managerRef.current.subscribe(setMessages);
    
    return unsubscribe;
  }, []);
  
  const sendMessage = useCallback((text: string) => {
    const tempId = `temp-${Date.now()}`;
    
    const optimisticMessage: Message = {
      id: tempId,
      text,
      userId: 'current-user',
      timestamp: Date.now(),
      status: 'pending',
    };
    
    // Apply optimistic update
    managerRef.current?.applyOptimistic(
      tempId,
      [...messages, optimisticMessage]
    );
    
    // Send to server
    socket.emit('message:send', { text, tempId }, (response: any) => {
      if (response.success) {
        managerRef.current?.confirm(
          tempId,
          [...messages.filter(m => m.id !== tempId), {
            ...optimisticMessage,
            id: response.id,
            status: 'confirmed',
          }]
        );
      } else {
        managerRef.current?.rollback(tempId);
      }
    });
  }, [socket, messages]);
  
  return { messages, sendMessage };
}
```

---

# 15. Real-World Patterns

## 15.1 Live Activity Feed

```typescript
interface Activity {
  id: string;
  type: 'comment' | 'like' | 'share' | 'follow' | 'post';
  actor: { id: string; name: string; avatar: string };
  target?: { id: string; name: string; type: string };
  metadata?: Record<string, any>;
  timestamp: number;
}

function useActivityFeed(userId: string) {
  const [activities, setActivities] = useState<Activity[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [hasMore, setHasMore] = useState(true);
  
  // SSE for real-time updates
  useEffect(() => {
    const eventSource = new EventSource(`/api/users/${userId}/activity/stream`);
    
    eventSource.addEventListener('activity', (event) => {
      const activity: Activity = JSON.parse(event.data);
      
      setActivities(prev => {
        // Deduplicate
        if (prev.some(a => a.id === activity.id)) {
          return prev;
        }
        
        // Add to front (newest first)
        return [activity, ...prev].slice(0, 100); // Keep last 100
      });
    });
    
    eventSource.onerror = () => {
      console.error('Activity stream error');
    };
    
    return () => eventSource.close();
  }, [userId]);
  
  // Initial load
  useEffect(() => {
    async function loadInitial() {
      setIsLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}/activity?limit=20`);
        const data = await response.json();
        setActivities(data.activities);
        setHasMore(data.hasMore);
      } finally {
        setIsLoading(false);
      }
    }
    
    loadInitial();
  }, [userId]);
  
  // Load more (infinite scroll)
  const loadMore = useCallback(async () => {
    if (!hasMore || isLoading) return;
    
    const lastActivity = activities[activities.length - 1];
    if (!lastActivity) return;
    
    setIsLoading(true);
    try {
      const response = await fetch(
        `/api/users/${userId}/activity?before=${lastActivity.id}&limit=20`
      );
      const data = await response.json();
      setActivities(prev => [...prev, ...data.activities]);
      setHasMore(data.hasMore);
    } finally {
      setIsLoading(false);
    }
  }, [userId, activities, hasMore, isLoading]);
  
  return { activities, isLoading, hasMore, loadMore };
}
```

## 15.2 Live Search with Debouncing

```typescript
function useLiveSearch<T>(
  searchFn: (query: string) => Promise<T[]>,
  debounceMs: number = 300
) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<T[]>([]);
  const [isSearching, setIsSearching] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const abortControllerRef = useRef<AbortController>();
  
  useEffect(() => {
    // Clear results for empty query
    if (!query.trim()) {
      setResults([]);
      setIsSearching(false);
      return;
    }
    
    setIsSearching(true);
    setError(null);
    
    // Debounce
    const timer = setTimeout(async () => {
      // Cancel previous request
      abortControllerRef.current?.abort();
      abortControllerRef.current = new AbortController();
      
      try {
        const searchResults = await searchFn(query);
        setResults(searchResults);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setIsSearching(false);
      }
    }, debounceMs);
    
    return () => {
      clearTimeout(timer);
      abortControllerRef.current?.abort();
    };
  }, [query, searchFn, debounceMs]);
  
  return {
    query,
    setQuery,
    results,
    isSearching,
    error,
    clear: () => {
      setQuery('');
      setResults([]);
    },
  };
}

// With WebSocket for real-time results
function useLiveSearchSocket<T>(socket: Socket) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<T[]>([]);
  const [isSearching, setIsSearching] = useState(false);
  const searchIdRef = useRef(0);
  
  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }
    
    setIsSearching(true);
    const searchId = ++searchIdRef.current;
    
    const timer = setTimeout(() => {
      socket.emit('search', { query, searchId });
    }, 300);
    
    return () => clearTimeout(timer);
  }, [query, socket]);
  
  useEffect(() => {
    socket.on('search:results', ({ searchId, results: newResults }) => {
      // Only accept results for the latest search
      if (searchId === searchIdRef.current) {
        setResults(newResults);
        setIsSearching(false);
      }
    });
    
    return () => {
      socket.off('search:results');
    };
  }, [socket]);
  
  return { query, setQuery, results, isSearching };
}
```

## 15.3 Real-Time Auction System

```typescript
interface Auction {
  id: string;
  title: string;
  currentBid: number;
  highestBidder?: { id: string; name: string };
  endsAt: number;
  status: 'active' | 'ended';
}

interface Bid {
  auctionId: string;
  bidderId: string;
  bidderName: string;
  amount: number;
  timestamp: number;
}

function useAuction(auctionId: string, socket: Socket) {
  const [auction, setAuction] = useState<Auction | null>(null);
  const [bidHistory, setBidHistory] = useState<Bid[]>([]);
  const [timeLeft, setTimeLeft] = useState<number>(0);
  const [error, setError] = useState<string | null>(null);
  
  // Subscribe to auction updates
  useEffect(() => {
    socket.emit('auction:join', auctionId);
    
    socket.on('auction:state', (state: Auction) => {
      setAuction(state);
      setTimeLeft(Math.max(0, state.endsAt - Date.now()));
    });
    
    socket.on('auction:bid', (bid: Bid) => {
      setBidHistory(prev => [bid, ...prev.slice(0, 49)]);
      setAuction(prev => prev ? {
        ...prev,
        currentBid: bid.amount,
        highestBidder: { id: bid.bidderId, name: bid.bidderName },
      } : null);
    });
    
    socket.on('auction:ended', (finalAuction: Auction) => {
      setAuction({ ...finalAuction, status: 'ended' });
    });
    
    socket.on('bid:error', ({ message }) => {
      setError(message);
      setTimeout(() => setError(null), 3000);
    });
    
    return () => {
      socket.emit('auction:leave', auctionId);
      socket.off('auction:state');
      socket.off('auction:bid');
      socket.off('auction:ended');
      socket.off('bid:error');
    };
  }, [auctionId, socket]);
  
  // Countdown timer
  useEffect(() => {
    if (!auction || auction.status === 'ended') return;
    
    const timer = setInterval(() => {
      setTimeLeft(prev => {
        const newTime = Math.max(0, prev - 1000);
        if (newTime === 0) {
          clearInterval(timer);
        }
        return newTime;
      });
    }, 1000);
    
    return () => clearInterval(timer);
  }, [auction]);
  
  const placeBid = useCallback((amount: number) => {
    if (!auction || auction.status === 'ended') {
      setError('Auction has ended');
      return;
    }
    
    if (amount <= auction.currentBid) {
      setError(`Bid must be higher than ${auction.currentBid}`);
      return;
    }
    
    socket.emit('auction:bid', { auctionId, amount });
  }, [auction, auctionId, socket]);
  
  return {
    auction,
    bidHistory,
    timeLeft,
    error,
    placeBid,
    formatTimeLeft: () => {
      const minutes = Math.floor(timeLeft / 60000);
      const seconds = Math.floor((timeLeft % 60000) / 1000);
      return `${minutes}:${seconds.toString().padStart(2, '0')}`;
    },
  };
}
```

---

# 16. Testing Real-Time Systems

## 16.1 Unit Testing WebSocket Handlers

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// Mock WebSocket
class MockWebSocket {
  url: string;
  readyState = WebSocket.OPEN;
  onopen: (() => void) | null = null;
  onclose: ((event: any) => void) | null = null;
  onmessage: ((event: any) => void) | null = null;
  onerror: ((event: any) => void) | null = null;
  
  constructor(url: string) {
    this.url = url;
    setTimeout(() => this.onopen?.(), 0);
  }
  
  send = vi.fn();
  close = vi.fn();
  
  // Test helpers
  simulateMessage(data: any) {
    this.onmessage?.({ data: JSON.stringify(data) });
  }
  
  simulateClose(code: number, reason: string) {
    this.readyState = WebSocket.CLOSED;
    this.onclose?.({ code, reason });
  }
}

describe('WebSocketClient', () => {
  let mockWS: MockWebSocket;
  
  beforeEach(() => {
    vi.stubGlobal('WebSocket', MockWebSocket);
  });
  
  it('should connect and send messages', async () => {
    const client = new WebSocketClient('wss://test.com');
    await client.connect();
    
    client.send({ type: 'test', payload: 'hello' });
    
    expect(mockWS.send).toHaveBeenCalledWith(
      expect.stringContaining('"type":"test"')
    );
  });
  
  it('should handle incoming messages', async () => {
    const client = new WebSocketClient('wss://test.com');
    const handler = vi.fn();
    
    client.on('message', handler);
    await client.connect();
    
    mockWS.simulateMessage({ type: 'message', payload: 'test' });
    
    expect(handler).toHaveBeenCalledWith(
      expect.objectContaining({ type: 'message', payload: 'test' })
    );
  });
  
  it('should reconnect on abnormal close', async () => {
    const client = new WebSocketClient('wss://test.com');
    await client.connect();
    
    vi.useFakeTimers();
    mockWS.simulateClose(1006, 'Abnormal close');
    
    // Fast-forward through reconnection delay
    await vi.advanceTimersByTimeAsync(1000);
    
    expect(WebSocket).toHaveBeenCalledTimes(2);
    vi.useRealTimers();
  });
});
```

## 16.2 Integration Testing with Mock Server

```typescript
import { Server } from 'socket.io';
import { io as Client, Socket as ClientSocket } from 'socket.io-client';
import { createServer } from 'http';

describe('Chat Integration', () => {
  let io: Server;
  let serverSocket: any;
  let clientSocket: ClientSocket;
  
  beforeAll((done) => {
    const httpServer = createServer();
    io = new Server(httpServer);
    
    httpServer.listen(() => {
      const port = (httpServer.address() as any).port;
      clientSocket = Client(`http://localhost:${port}`);
      
      io.on('connection', (socket) => {
        serverSocket = socket;
      });
      
      clientSocket.on('connect', done);
    });
  });
  
  afterAll(() => {
    io.close();
    clientSocket.close();
  });
  
  it('should broadcast messages to room', (done) => {
    const testMessage = { text: 'Hello!', userId: '123' };
    
    // Setup second client
    const port = (io.httpServer.address() as any).port;
    const client2 = Client(`http://localhost:${port}`);
    
    client2.on('connect', () => {
      // Both join room
      clientSocket.emit('room:join', 'test-room');
      client2.emit('room:join', 'test-room');
      
      // Listen for message on client2
      client2.on('message:new', (received) => {
        expect(received).toEqual(testMessage);
        client2.close();
        done();
      });
      
      // Send from client1
      setTimeout(() => {
        clientSocket.emit('message:send', testMessage);
      }, 100);
    });
  });
  
  it('should handle typing indicators', (done) => {
    clientSocket.on('typing:update', ({ userId, isTyping }) => {
      expect(userId).toBe('456');
      expect(isTyping).toBe(true);
      done();
    });
    
    serverSocket.emit('typing:update', { userId: '456', isTyping: true });
  });
});
```

---

# 17. Performance Optimization

## 17.1 Binary Protocol

```typescript
// Using MessagePack for efficient serialization

import { encode, decode } from '@msgpack/msgpack';

class BinaryWebSocket {
  private ws: WebSocket;
  
  constructor(url: string) {
    this.ws = new WebSocket(url);
    this.ws.binaryType = 'arraybuffer';
  }
  
  send(data: any) {
    const encoded = encode(data);
    this.ws.send(encoded);
  }
  
  onMessage(handler: (data: any) => void) {
    this.ws.onmessage = (event) => {
      if (event.data instanceof ArrayBuffer) {
        const decoded = decode(new Uint8Array(event.data));
        handler(decoded);
      }
    };
  }
}

// Comparison:
const testData = {
  type: 'message',
  userId: '12345',
  text: 'Hello, World!',
  timestamp: 1703180400000,
};

// JSON: 89 bytes
console.log(JSON.stringify(testData).length);

// MessagePack: 66 bytes (26% smaller)
console.log(encode(testData).length);

// For high-frequency data like cursor positions:
const cursorData = { x: 123.456, y: 789.012, t: 1703180400000 };
// JSON: 42 bytes
// MessagePack: 24 bytes (43% smaller)
```

## 17.2 Connection Pooling

```typescript
// For multiple Socket.io namespaces
class SocketPool {
  private sockets: Map<string, Socket> = new Map();
  private baseUrl: string;
  private auth: any;
  
  constructor(baseUrl: string, auth: any) {
    this.baseUrl = baseUrl;
    this.auth = auth;
  }
  
  getSocket(namespace: string): Socket {
    if (!this.sockets.has(namespace)) {
      const socket = io(`${this.baseUrl}/${namespace}`, {
        auth: this.auth,
        transports: ['websocket'],
        forceNew: true,
        multiplex: false,
      });
      
      this.sockets.set(namespace, socket);
    }
    
    return this.sockets.get(namespace)!;
  }
  
  disconnectAll() {
    this.sockets.forEach(socket => socket.disconnect());
    this.sockets.clear();
  }
}
```

## 17.3 Lazy Connection

```typescript
// Connect only when needed
function useLazySocket() {
  const socketRef = useRef<Socket | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  
  const connect = useCallback(() => {
    if (!socketRef.current) {
      socketRef.current = io('https://api.example.com', {
        autoConnect: true,
      });
      
      socketRef.current.on('connect', () => setIsConnected(true));
      socketRef.current.on('disconnect', () => setIsConnected(false));
    }
    
    return socketRef.current;
  }, []);
  
  const disconnect = useCallback(() => {
    socketRef.current?.disconnect();
    socketRef.current = null;
    setIsConnected(false);
  }, []);
  
  // Auto-disconnect on unmount
  useEffect(() => {
    return () => {
      disconnect();
    };
  }, [disconnect]);
  
  return { connect, disconnect, socket: socketRef.current, isConnected };
}

// Usage - only connect when entering chat
function ChatButton() {
  const { connect } = useLazySocket();
  const [showChat, setShowChat] = useState(false);
  
  const handleOpenChat = () => {
    connect();
    setShowChat(true);
  };
  
  return (
    <button onClick={handleOpenChat}>Open Chat</button>
  );
}
```

---

# 18. Quick Reference

## 18.1 WebSocket Cheat Sheet

```typescript
// Create
const ws = new WebSocket('wss://example.com');

// Events
ws.onopen = () => { /* connected */ };
ws.onmessage = (e) => { /* e.data */ };
ws.onerror = (e) => { /* error */ };
ws.onclose = (e) => { /* e.code, e.reason */ };

// Send
ws.send('text');
ws.send(JSON.stringify(obj));
ws.send(new Blob([...]));
ws.send(new ArrayBuffer(...));

// Close
ws.close(1000, 'reason');

// State
ws.readyState; // 0=CONNECTING, 1=OPEN, 2=CLOSING, 3=CLOSED
```

## 18.2 Socket.io Cheat Sheet

```typescript
// Connect
const socket = io(url, { auth, transports, ... });

// Events
socket.on('connect', () => {});
socket.on('disconnect', (reason) => {});
socket.on('eventName', (data) => {});
socket.once('eventName', (data) => {});

// Emit
socket.emit('event', data);
socket.emit('event', data, (ack) => {});
socket.timeout(5000).emit('event', data, (err, res) => {});
socket.volatile.emit('event', data); // May drop

// Rooms (server only, client can request)
socket.emit('room:join', 'roomId');

// Disconnect
socket.disconnect();
```

## 18.3 SSE Cheat Sheet

```typescript
// Create
const es = new EventSource('/events', { withCredentials: true });

// Events
es.onmessage = (e) => { /* e.data */ };
es.onopen = () => { /* connected */ };
es.onerror = () => { /* error or reconnecting */ };
es.addEventListener('eventType', (e) => { /* custom event */ });

// Close
es.close();

// State
es.readyState; // 0=CONNECTING, 1=OPEN, 2=CLOSED

// Server format
// data: message\n\n
// event: type\ndata: message\n\n
// id: 123\ndata: message\n\n
// retry: 5000\n\n
```

---

# 19. Final Interview Tips

## 19.1 Common Mistakes to Avoid

```typescript
// ❌ NOT handling disconnection
ws.onclose = () => { /* nothing */ };

// ✅ Handle with reconnection
ws.onclose = (event) => {
  if (event.code !== 1000) {
    scheduleReconnect();
  }
};

// ❌ NOT cleaning up listeners in React
useEffect(() => {
  socket.on('event', handler);
  // Missing cleanup!
}, []);

// ✅ Always cleanup
useEffect(() => {
  socket.on('event', handler);
  return () => socket.off('event', handler);
}, []);

// ❌ Sending without checking connection
ws.send(data); // May throw if not connected

// ✅ Check first
if (ws.readyState === WebSocket.OPEN) {
  ws.send(data);
}

// ❌ No rate limiting on client
input.oninput = () => socket.emit('typing'); // Spam!

// ✅ Debounce/throttle
input.oninput = debounce(() => socket.emit('typing'), 300);

// ❌ Storing sensitive data in WebSocket URL
new WebSocket(`wss://api.com?token=${secret}`); // Logged!

// ✅ Send in first message
ws.onopen = () => ws.send(JSON.stringify({ auth: secret }));
```

## 19.2 Key Patterns to Remember

1. **Exponential Backoff** - For reconnection
2. **Heartbeat/Ping-Pong** - For connection health
3. **Message Queue** - For offline support
4. **Optimistic Updates** - For responsive UI
5. **Debouncing** - For typing indicators
6. **Rate Limiting** - For spam prevention
7. **Binary Protocol** - For performance

## 19.3 Interview Question Patterns

```typescript
// Be ready to discuss:

// 1. "How would you design a chat application?"
// - Message format, delivery status, typing indicators
// - Rooms, presence, notifications
// - Offline support, reconnection

// 2. "How do you handle scaling WebSockets?"
// - Sticky sessions, Redis pub/sub
// - Connection limits, load balancing
// - Horizontal scaling strategies

// 3. "What's the difference between X and Y?"
// - WebSocket vs HTTP
// - WebSocket vs Socket.io
// - SSE vs WebSocket
// - Polling vs Long Polling vs WebSocket

// 4. "How do you handle [specific challenge]?"
// - Authentication
// - Reconnection
// - Offline messages
// - Rate limiting
// - Message ordering
```

---

**End of Part 9: Real-Time Systems Complete Guide**

Total Sections: 19
Key Topics: WebSockets, Socket.io, SSE, Long Polling, Real-Time Patterns
Interview Ready: ✅

---

# Bonus Section: Output-Based Questions

## B1. What will be logged?

```typescript
// Question 1: Connection State
const ws = new WebSocket('wss://example.com');
console.log(ws.readyState);

ws.onopen = () => {
  console.log(ws.readyState);
};

// Answer: 
// First log: 0 (CONNECTING)
// Second log: 1 (OPEN)

// Explanation: WebSocket connection is async.
// readyState is 0 (CONNECTING) immediately after creation.
// Only becomes 1 (OPEN) after successful handshake.
```

```typescript
// Question 2: Message Order
const messages: string[] = [];
const ws = new WebSocket('wss://example.com');

ws.onmessage = (e) => {
  messages.push(e.data);
};

ws.onopen = () => {
  ws.send('A');
  ws.send('B');
  ws.send('C');
};

// After server echoes back messages, what order are they in?
// Answer: ['A', 'B', 'C']

// Explanation: WebSocket guarantees message ordering.
// Messages are received in the same order they were sent.
// Unlike HTTP requests which may complete out of order.
```

```typescript
// Question 3: Close Event
const ws = new WebSocket('wss://example.com');

ws.onopen = () => {
  ws.close(1000, 'Done');
};

ws.onclose = (e) => {
  console.log(e.code);
  console.log(e.wasClean);
};

// Answer:
// First log: 1000
// Second log: true

// Explanation: 1000 is normal closure code.
// wasClean is true when close handshake completed properly.
// If connection dropped unexpectedly, wasClean would be false.
```

```typescript
// Question 4: Socket.io reconnection
const socket = io('https://example.com', {
  reconnection: true,
  reconnectionAttempts: 3,
  reconnectionDelay: 1000,
});

let attempts = 0;

socket.on('reconnect_attempt', () => {
  attempts++;
  console.log('attempt', attempts);
});

socket.on('reconnect_failed', () => {
  console.log('failed after', attempts);
});

// If connection fails and never succeeds, what logs?
// Answer:
// attempt 1
// attempt 2
// attempt 3
// failed after 3

// Explanation: Socket.io fires reconnect_attempt for each try.
// After reconnectionAttempts (3) failures, fires reconnect_failed.
```

```typescript
// Question 5: Event Listener Memory
function Component() {
  const socket = useRef(io('https://example.com'));
  
  useEffect(() => {
    socket.current.on('message', handleMessage);
    // Missing cleanup!
  }, []);
  
  return <div />;
}

// What happens if Component mounts/unmounts 5 times?
// Answer: 5 message handlers attached!

// Each mount adds a new listener without removing old ones.
// This causes handleMessage to fire 5 times per message.
// Fix: return () => socket.current.off('message', handleMessage);
```

## B2. Identify the Bug

```typescript
// Bug 1: Race condition
function Chat() {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    const ws = new WebSocket('wss://api.com');
    
    ws.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      setMessages([...messages, msg]); // BUG!
    };
    
    return () => ws.close();
  }, []);
}

// Bug: Stale closure - messages is always []
// Fix: setMessages(prev => [...prev, msg])
```

```typescript
// Bug 2: Infinite reconnection
const ws = new WebSocket('wss://api.com');

ws.onclose = (event) => {
  // Reconnect!
  new WebSocket('wss://api.com');
};

// Bug: Creates new WS but doesn't set up handlers!
// The new connection has no onclose, onmessage, etc.
// Fix: Create a connect() function that sets up all handlers
```

```typescript
// Bug 3: Memory leak
function useSocket() {
  const socket = io('https://api.com');
  
  useEffect(() => {
    return () => {
      // Forgot to disconnect!
    };
  }, []);
  
  return socket;
}

// Bug: Socket connection never closed
// Each call to useSocket creates a new persistent connection
// Fix: socket.disconnect() in cleanup
```

```typescript
// Bug 4: Typing spam
function ChatInput({ socket }) {
  const [text, setText] = useState('');
  
  const handleChange = (e) => {
    setText(e.target.value);
    socket.emit('typing', true); // BUG!
  };
  
  return <input value={text} onChange={handleChange} />;
}

// Bug: Emits typing on every keystroke
// This floods the server with typing events
// Fix: Debounce the typing emission
```

---

# Bonus Section: Mock Interview Scenarios

## Scenario 1: Design a Multiplayer Game Lobby

**Interviewer**: "Design the real-time communication for a multiplayer game lobby where players can:
1. See who's online
2. Join/create game rooms
3. See room status updates
4. Chat in lobbies"

**Strong Answer:**

```typescript
// Data structures
interface GameRoom {
  id: string;
  name: string;
  host: Player;
  players: Player[];
  maxPlayers: number;
  status: 'waiting' | 'starting' | 'in_progress';
  settings: GameSettings;
}

interface Player {
  id: string;
  name: string;
  avatar: string;
  status: 'online' | 'away' | 'in_game';
}

// Socket events architecture
const LOBBY_EVENTS = {
  // Sent from client
  'lobby:join': 'Join the main lobby',
  'room:create': 'Create a new game room',
  'room:join': 'Join an existing room',
  'room:leave': 'Leave current room',
  'room:ready': 'Toggle ready status',
  'room:start': 'Host starts the game',
  'chat:message': 'Send chat message',
  
  // Sent from server
  'players:list': 'Initial list of online players',
  'player:join': 'New player came online',
  'player:leave': 'Player went offline',
  'rooms:list': 'Initial list of rooms',
  'room:created': 'New room was created',
  'room:updated': 'Room state changed',
  'room:deleted': 'Room was closed',
  'chat:new': 'New chat message',
};

// Implementation
class GameLobbyClient {
  private socket: Socket;
  private currentRoom: string | null = null;
  
  constructor() {
    this.socket = io('/game-lobby', {
      auth: { token: getAuthToken() },
      transports: ['websocket'],
    });
    
    this.setupHandlers();
  }
  
  private setupHandlers() {
    // Presence
    this.socket.on('players:list', (players) => {
      store.dispatch(setOnlinePlayers(players));
    });
    
    this.socket.on('player:join', (player) => {
      store.dispatch(addOnlinePlayer(player));
    });
    
    // Rooms
    this.socket.on('rooms:list', (rooms) => {
      store.dispatch(setRooms(rooms));
    });
    
    this.socket.on('room:updated', (room) => {
      store.dispatch(updateRoom(room));
      
      // If we're in this room, update our view
      if (room.id === this.currentRoom) {
        store.dispatch(setCurrentRoomState(room));
      }
    });
    
    // Game start
    this.socket.on('room:starting', ({ countdown }) => {
      store.dispatch(setGameStarting(countdown));
    });
    
    this.socket.on('room:started', ({ gameId }) => {
      // Transition to game
      router.push(`/game/${gameId}`);
    });
  }
  
  createRoom(name: string, settings: GameSettings) {
    this.socket.emit('room:create', { name, settings }, (response) => {
      if (response.success) {
        this.currentRoom = response.roomId;
      }
    });
  }
  
  joinRoom(roomId: string) {
    this.socket.emit('room:join', { roomId }, (response) => {
      if (response.success) {
        this.currentRoom = roomId;
      } else {
        showError(response.error); // "Room is full", etc.
      }
    });
  }
  
  toggleReady() {
    this.socket.emit('room:ready');
  }
  
  startGame() {
    this.socket.emit('room:start', (response) => {
      if (!response.success) {
        showError(response.error); // "Not all players ready"
      }
    });
  }
}

// Key design decisions:
// 1. Namespaces for separation (/game-lobby vs /game)
// 2. Acknowledgments for user actions
// 3. Server authoritative for game state
// 4. Optimistic UI for non-critical updates (chat)
// 5. Room-based broadcasting for efficiency
```

## Scenario 2: Handle Connection Loss Gracefully

**Interviewer**: "The user is in the middle of typing a message when they lose internet connection. How do you handle this?"

**Strong Answer:**

```typescript
class ResilientChat {
  private socket: Socket;
  private messageQueue: Message[] = [];
  private isOnline = navigator.onLine;
  
  constructor() {
    this.socket = io('/chat', {
      reconnection: true,
      reconnectionAttempts: Infinity,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 30000,
    });
    
    this.setupNetworkHandlers();
    this.loadPendingFromStorage();
  }
  
  private setupNetworkHandlers() {
    // Browser online/offline events
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.showStatus('Back online');
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
      this.showStatus('You are offline');
    });
    
    // Socket events
    this.socket.on('connect', () => {
      this.showStatus('Connected');
      this.flushMessageQueue();
    });
    
    this.socket.on('disconnect', (reason) => {
      this.showStatus('Disconnected: ' + reason);
      
      if (reason === 'io server disconnect') {
        // Server kicked us, maybe auth expired
        this.handleAuthFailure();
      }
    });
    
    // Message confirmation
    this.socket.on('message:ack', ({ tempId, id }) => {
      this.confirmMessage(tempId, id);
    });
  }
  
  sendMessage(text: string) {
    const tempId = generateTempId();
    const message: Message = {
      id: tempId,
      text,
      status: 'pending',
      timestamp: Date.now(),
    };
    
    // Add to UI immediately (optimistic)
    this.addToUI(message);
    
    // Add to queue
    this.messageQueue.push(message);
    this.saveQueueToStorage();
    
    // Try to send if connected
    if (this.socket.connected) {
      this.socket.emit('message:send', { tempId, text });
    }
    // If not connected, it will be sent when we reconnect
  }
  
  private flushMessageQueue() {
    for (const msg of this.messageQueue) {
      this.socket.emit('message:send', {
        tempId: msg.id,
        text: msg.text,
        originalTimestamp: msg.timestamp,
      });
    }
  }
  
  private confirmMessage(tempId: string, realId: string) {
    // Remove from queue
    this.messageQueue = this.messageQueue.filter(m => m.id !== tempId);
    this.saveQueueToStorage();
    
    // Update UI
    this.updateMessageInUI(tempId, {
      id: realId,
      status: 'sent',
    });
  }
  
  private loadPendingFromStorage() {
    const saved = localStorage.getItem('pendingMessages');
    if (saved) {
      this.messageQueue = JSON.parse(saved);
      // Show pending messages in UI
      for (const msg of this.messageQueue) {
        this.addToUI({ ...msg, status: 'pending' });
      }
    }
  }
  
  private saveQueueToStorage() {
    localStorage.setItem('pendingMessages', JSON.stringify(this.messageQueue));
  }
}

// UI indicators to show:
// 1. Connection status bar
// 2. Pending message indicator (clock icon)
// 3. Failed message with retry button
// 4. "Reconnecting..." message
```

## Scenario 3: Scale to 1 Million Concurrent Users

**Interviewer**: "How would you design a real-time notification system that can handle 1 million concurrent connections?"

**Strong Answer:**

```typescript
// Architecture overview:

const ARCHITECTURE = {
  // Layer 1: Load Balancer
  loadBalancer: {
    type: 'Multiple HAProxy/Nginx instances',
    strategy: 'Consistent hashing by user ID',
    purpose: 'Sticky sessions for WebSocket',
  },
  
  // Layer 2: WebSocket Servers
  wsServers: {
    count: '50-100 instances',
    connectionsPerServer: '10,000-20,000',
    technology: 'Socket.io with Redis adapter',
    scaling: 'Horizontal auto-scaling',
  },
  
  // Layer 3: Message Bus
  messageBus: {
    technology: 'Redis Pub/Sub or Kafka',
    purpose: 'Cross-server message delivery',
    partitioning: 'By user ID for ordered delivery',
  },
  
  // Layer 4: Notification Service
  notificationService: {
    type: 'Stateless workers',
    scaling: 'Horizontal based on queue depth',
    purpose: 'Generate and route notifications',
  },
};

// Key optimizations:

// 1. Connection efficiency
const WS_OPTIMIZATIONS = {
  // Binary protocol to reduce bandwidth
  protocol: 'MessagePack instead of JSON',
  compression: 'Per-message deflate',
  heartbeat: 'Longer interval (60s) with tolerance',
  
  // Connection pooling from load balancer
  keepAlive: true,
  
  // Lazy loading - connect only when user views notifications
  lazyConnect: true,
};

// 2. Message batching for high-throughput
class NotificationBatcher {
  private batches: Map<string, Notification[]> = new Map();
  private timer: NodeJS.Timer | null = null;
  
  queue(userId: string, notification: Notification) {
    if (!this.batches.has(userId)) {
      this.batches.set(userId, []);
    }
    this.batches.get(userId)!.push(notification);
    
    this.scheduleFlush();
  }
  
  private scheduleFlush() {
    if (!this.timer) {
      this.timer = setTimeout(() => {
        this.flush();
      }, 50); // Batch for 50ms
    }
  }
  
  private flush() {
    for (const [userId, notifications] of this.batches) {
      redis.publish(`notifications:${userId}`, notifications);
    }
    this.batches.clear();
    this.timer = null;
  }
}

// 3. Graceful degradation
const DEGRADATION_STRATEGIES = {
  // If too many connections, new users get SSE instead
  fallbackToSSE: 'When WS servers are at 80% capacity',
  
  // If even more load, switch to polling
  fallbackToPolling: 'When SSE connections exhausted',
  
  // Notification aggregation under extreme load
  aggregation: 'Group notifications: "5 people liked your post"',
  
  // Priority queue
  priorityQueue: 'Critical notifications first, social later',
};

// 4. Geographic distribution
const GEO_DISTRIBUTION = {
  regions: ['us-east', 'us-west', 'eu', 'asia'],
  routing: 'Anycast + GeoDNS',
  replication: 'Each region has full stack',
  crossRegion: 'Async replication via Kafka',
};
```

---

# Bonus Section: Additional Patterns

## Pattern 1: Presence with Large User Counts

```typescript
// For systems with millions of users, you can't track everyone
// Use a bloom filter or count-min sketch for "is online" checks

class EfficientPresence {
  private localUsers: Set<string> = new Set();
  private redis: Redis;
  
  // Only track users relevant to current user
  async getOnlineStatus(userIds: string[]): Promise<Map<string, boolean>> {
    // Batch check with Redis
    const pipeline = this.redis.pipeline();
    
    for (const userId of userIds) {
      pipeline.exists(`presence:${userId}`);
    }
    
    const results = await pipeline.exec();
    
    const status = new Map<string, boolean>();
    userIds.forEach((userId, i) => {
      status.set(userId, results[i][1] === 1);
    });
    
    return status;
  }
  
  // Set presence with TTL
  async setOnline(userId: string) {
    await this.redis.setex(`presence:${userId}`, 30, Date.now().toString());
    await this.redis.publish('presence:online', userId);
  }
  
  // Extend presence periodically
  async heartbeat(userId: string) {
    await this.redis.expire(`presence:${userId}`, 30);
  }
  
  // Subscribe only to relevant users
  async watchUsers(userIds: string[], callback: (userId: string, online: boolean) => void) {
    const subscriber = this.redis.duplicate();
    
    // Only subscribe to channels for users we care about
    const channels = userIds.map(id => `presence:${id}`);
    await subscriber.subscribe(...channels);
    
    subscriber.on('message', (channel, message) => {
      const userId = channel.replace('presence:', '');
      callback(userId, message === 'online');
    });
    
    return () => subscriber.unsubscribe();
  }
}
```

## Pattern 2: Conflict-Free Replicated Data Types (CRDTs)

```typescript
// For truly distributed real-time collaboration

// G-Counter (grow-only counter)
class GCounter {
  private counts: Map<string, number> = new Map();
  
  constructor(private nodeId: string) {}
  
  increment() {
    const current = this.counts.get(this.nodeId) || 0;
    this.counts.set(this.nodeId, current + 1);
  }
  
  value(): number {
    let sum = 0;
    for (const count of this.counts.values()) {
      sum += count;
    }
    return sum;
  }
  
  merge(other: GCounter) {
    for (const [nodeId, count] of other.counts) {
      const current = this.counts.get(nodeId) || 0;
      this.counts.set(nodeId, Math.max(current, count));
    }
  }
  
  toJSON() {
    return Object.fromEntries(this.counts);
  }
}

// LWW-Register (last-write-wins)
class LWWRegister<T> {
  private value: T;
  private timestamp: number;
  private nodeId: string;
  
  constructor(nodeId: string, initialValue: T) {
    this.nodeId = nodeId;
    this.value = initialValue;
    this.timestamp = Date.now();
  }
  
  set(value: T) {
    this.value = value;
    this.timestamp = Date.now();
  }
  
  get(): T {
    return this.value;
  }
  
  merge(other: LWWRegister<T>) {
    if (other.timestamp > this.timestamp) {
      this.value = other.value;
      this.timestamp = other.timestamp;
    } else if (other.timestamp === this.timestamp && other.nodeId > this.nodeId) {
      // Tie-breaker: higher node ID wins
      this.value = other.value;
    }
  }
}

// Usage: Real-time collaborative counter
function useCollaborativeCounter(socket: Socket) {
  const [counter] = useState(() => new GCounter(socket.id!));
  const [displayValue, setDisplayValue] = useState(0);
  
  useEffect(() => {
    socket.on('counter:update', (remoteState) => {
      const remote = new GCounter('remote');
      Object.assign(remote, remoteState);
      counter.merge(remote);
      setDisplayValue(counter.value());
    });
    
    return () => socket.off('counter:update');
  }, [socket, counter]);
  
  const increment = () => {
    counter.increment();
    setDisplayValue(counter.value());
    socket.emit('counter:update', counter.toJSON());
  };
  
  return { value: displayValue, increment };
}
```

## Pattern 3: Efficient Diff-Based Updates

```typescript
// Only send changes, not full state

import { diff, applyChange, Change } from 'deep-diff';

class DiffSync<T extends object> {
  private lastSentState: T;
  private socket: Socket;
  
  constructor(initialState: T, socket: Socket) {
    this.lastSentState = structuredClone(initialState);
    this.socket = socket;
  }
  
  sync(currentState: T) {
    const changes = diff(this.lastSentState, currentState);
    
    if (changes && changes.length > 0) {
      // Only send the diff
      this.socket.emit('state:diff', changes);
      this.lastSentState = structuredClone(currentState);
    }
  }
  
  applyRemoteDiff(state: T, changes: Change<any>[]): T {
    const newState = structuredClone(state);
    
    for (const change of changes) {
      applyChange(newState, true, change);
    }
    
    return newState;
  }
}

// Usage
const sync = new DiffSync(initialGameState, socket);

// On state change, only transmit diff
gameEngine.onStateChange((newState) => {
  sync.sync(newState);
});

// Receive diffs from server
socket.on('state:diff', (changes) => {
  const newState = sync.applyRemoteDiff(currentState, changes);
  setGameState(newState);
});

// Benefits:
// - Reduced bandwidth (especially for large state)
// - Faster transmission
// - Easier conflict detection
```

---

# Final Summary

## What You Should Know

### Must-Know Topics
1. **WebSocket Lifecycle** - Handshake, states, close codes
2. **Socket.io vs WebSocket** - When to use each
3. **SSE** - When it's better than WebSocket
4. **Reconnection** - Exponential backoff, offline handling
5. **Authentication** - Token-based, refresh strategies
6. **React Integration** - Hooks, cleanup, state management

### Common Interview Questions
1. WebSocket vs HTTP
2. How to handle reconnection
3. How to scale WebSockets
4. Designing a chat application
5. Real-time presence tracking
6. Message ordering guarantees

### Key Patterns
1. Exponential backoff for reconnection
2. Heartbeat for connection health
3. Message queuing for offline
4. Optimistic updates for UX
5. Debouncing for typing indicators
6. Rate limiting for spam prevention

### Libraries to Know
1. **socket.io-client** - Full-featured real-time
2. **@tanstack/react-query** - Real-time with React Query
3. **zustand** - Real-time state management
4. **@msgpack/msgpack** - Binary protocol

---

**You are now ready for real-time systems interviews! 🚀**





