# 🔌 Frontend-Backend Integration Guide

**Documented:** November 19, 2025
**Target:** React developers learning full-stack integration
**Time Estimate:** 4-6 hours to fully understand
**Difficulty:** 🟡 Intermediate

---

## Connecting the Dots

You've learned the frontend (React) and backend (Express). Now let's connect them! This guide shows you exactly how data flows from your React components to the API and database.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [API Client Setup](#api-client-setup)
- [TanStack Query Integration](#tanstack-query-integration)
- [Authentication Flow](#authentication-flow)
- [Making API Calls](#making-api-calls)
- [Error Handling Across the Stack](#error-handling-across-the-stack)
- [Real-time Communication: Server-Sent Events](#real-time-communication-server-sent-events)
- [File Uploads](#file-uploads)
- [Complete Examples](#complete-examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Architecture Overview

### The Complete Stack

```
┌─────────────────────────────────────────────────────────┐
│                   BROWSER (React)                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Component (ChatInput.tsx)                      │   │
│  │    ↓ calls                                      │   │
│  │  Custom Hook (useSubmitMessage)                 │   │
│  │    ↓ uses                                       │   │
│  │  TanStack Query (useMutation)                   │   │
│  │    ↓ uses                                       │   │
│  │  API Client (axios)                             │   │
│  └─────────────────┬───────────────────────────────┘   │
└────────────────────┼───────────────────────────────────┘
                     │ HTTP Request (JSON)
                     │ POST /api/messages
                     │ Headers: { Authorization: Bearer token }
                     │ Body: { text: '...', conversationId: '...' }
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   SERVER (Express)                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Middleware Chain:                              │   │
│  │    1. express.json() → Parse body               │   │
│  │    2. cors() → Allow cross-origin               │   │
│  │    3. requireAuth → Verify JWT token            │   │
│  │    4. validateRequest → Validate input          │   │
│  │    ↓                                            │   │
│  │  Route (/api/messages)                          │   │
│  │    ↓                                            │   │
│  │  Controller (messagesController.createMessage)  │   │
│  │    ↓ calls                                      │   │
│  │  Service (messageService.createMessage)         │   │
│  │    ↓ uses                                       │   │
│  │  Model (Message.create)                         │   │
│  └─────────────────┬───────────────────────────────┘   │
└────────────────────┼───────────────────────────────────┘
                     │ MongoDB Query
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   DATABASE (MongoDB)                    │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Collection: messages                           │   │
│  │    - Insert document                            │   │
│  │    - Return saved document                      │   │
│  └─────────────────┬───────────────────────────────┘   │
└────────────────────┼───────────────────────────────────┘
                     │ Saved document
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   SERVER (Express)                      │
│  Service returns message → Controller returns JSON     │
└────────────────────┬───────────────────────────────────┘
                     │ HTTP Response (JSON)
                     │ Status: 201 Created
                     │ Body: { message: { id: '...', text: '...', ... } }
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   BROWSER (React)                       │
│  TanStack Query updates cache → Component re-renders    │
└─────────────────────────────────────────────────────────┘
```

---

## API Client Setup

### Axios Instance Configuration

**File:** `client/src/utils/apiClient.ts`

```typescript
import axios, { AxiosError, AxiosRequestConfig } from 'axios';

/**
 * API Client Configuration
 * Centralized axios instance with defaults
 */

const apiClient = axios.create({
  // Base URL from environment variable
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3080',

  // Timeout after 30 seconds
  timeout: 30000,

  // Default headers
  headers: {
    'Content-Type': 'application/json'
  },

  // Send cookies with requests (for sessions)
  withCredentials: true
});

// ===========================
// Request Interceptor
// ===========================

apiClient.interceptors.request.use(
  (config) => {
    // Add JWT token to all requests
    const token = localStorage.getItem('authToken');

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // Log request in development
    if (import.meta.env.DEV) {
      console.log(`→ ${config.method?.toUpperCase()} ${config.url}`, config.data);
    }

    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// ===========================
// Response Interceptor
// ===========================

apiClient.interceptors.response.use(
  (response) => {
    // Log response in development
    if (import.meta.env.DEV) {
      console.log(`← ${response.status} ${response.config.url}`, response.data);
    }

    return response;
  },
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };

    // Handle 401 Unauthorized (token expired)
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Try to refresh token
        const { data } = await axios.post('/api/auth/refresh');
        const newToken = data.token;

        // Save new token
        localStorage.setItem('authToken', newToken);

        // Retry original request with new token
        if (originalRequest.headers) {
          originalRequest.headers.Authorization = `Bearer ${newToken}`;
        }

        return apiClient(originalRequest);

      } catch (refreshError) {
        // Refresh failed - redirect to login
        localStorage.removeItem('authToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    // Log error in development
    if (import.meta.env.DEV) {
      console.error('API Error:', error.response?.data || error.message);
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

**Why an API client?**

✅ **Benefits:**
- Centralized configuration (baseURL, headers)
- Automatic token attachment
- Global error handling
- Request/response logging
- Token refresh logic

---

## TanStack Query Integration

### Query Client Setup

**File:** `client/src/config/queryClient.ts`

```typescript
import { QueryClient } from '@tanstack/react-query';

/**
 * TanStack Query Configuration
 */
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Refetch on window focus (user comes back to tab)
      refetchOnWindowFocus: true,

      // Refetch on reconnect (network comes back)
      refetchOnReconnect: true,

      // Don't refetch on mount if data is fresh
      refetchOnMount: true,

      // Retry failed queries 3 times
      retry: 3,

      // Retry with exponential backoff
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),

      // Data is considered fresh for 5 minutes
      staleTime: 1000 * 60 * 5,

      // Keep unused data in cache for 10 minutes
      gcTime: 1000 * 60 * 10
    },

    mutations: {
      // Retry mutations once
      retry: 1,

      // Retry delay
      retryDelay: 1000
    }
  }
});
```

**Provider Setup:**

**File:** `client/src/main.tsx`

```typescript
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from './config/queryClient';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />

      {/* DevTools (only in development) */}
      {import.meta.env.DEV && <ReactQueryDevtools initialIsOpen={false} />}
    </QueryClientProvider>
  </React.StrictMode>
);
```

---

## Authentication Flow

### Complete Auth Implementation

**1. Login Component**

**File:** `client/src/components/Auth/Login.tsx`

```typescript
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useLogin } from '~/hooks/useAuth';

export function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const navigate = useNavigate();

  const loginMutation = useLogin();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      await loginMutation.mutateAsync({ email, password });
      navigate('/'); // Redirect to home on success
    } catch (error) {
      // Error handled by mutation
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />

      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />

      <button type="submit" disabled={loginMutation.isPending}>
        {loginMutation.isPending ? 'Logging in...' : 'Login'}
      </button>

      {loginMutation.isError && (
        <p className="error">
          {loginMutation.error.response?.data?.error?.message || 'Login failed'}
        </p>
      )}
    </form>
  );
}
```

**2. Auth Hook**

**File:** `client/src/hooks/useAuth.ts`

```typescript
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import apiClient from '~/utils/apiClient';
import { useNavigate } from 'react-router-dom';

interface LoginCredentials {
  email: string;
  password: string;
}

interface AuthResponse {
  user: {
    id: string;
    name: string;
    email: string;
  };
  token: string;
}

/**
 * Hook: Login user
 */
export function useLogin() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (credentials: LoginCredentials) => {
      const { data } = await apiClient.post<AuthResponse>('/api/auth/login', credentials);
      return data;
    },

    onSuccess: (data) => {
      // Save token to localStorage
      localStorage.setItem('authToken', data.token);

      // Save user to cache
      queryClient.setQueryData(['user'], data.user);
    },

    onError: (error) => {
      console.error('Login failed:', error);
    }
  });
}

/**
 * Hook: Register user
 */
export function useRegister() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (credentials: LoginCredentials & { name: string }) => {
      const { data } = await apiClient.post<AuthResponse>('/api/auth/register', credentials);
      return data;
    },

    onSuccess: (data) => {
      localStorage.setItem('authToken', data.token);
      queryClient.setQueryData(['user'], data.user);
    }
  });
}

/**
 * Hook: Get current user
 */
export function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: async () => {
      const token = localStorage.getItem('authToken');

      if (!token) {
        return null;
      }

      const { data } = await apiClient.get('/api/auth/me');
      return data.user;
    },
    retry: false // Don't retry if unauthorized
  });
}

/**
 * Hook: Logout user
 */
export function useLogout() {
  const queryClient = useQueryClient();
  const navigate = useNavigate();

  return useMutation({
    mutationFn: async () => {
      // Optional: Call logout endpoint
      await apiClient.post('/api/auth/logout');
    },

    onSettled: () => {
      // Remove token
      localStorage.removeItem('authToken');

      // Clear all cached data
      queryClient.clear();

      // Redirect to login
      navigate('/login');
    }
  });
}

/**
 * Hook: Check if user is authenticated
 */
export function useAuth() {
  const { data: user, isLoading } = useUser();

  return {
    user,
    isAuthenticated: !!user,
    isLoading
  };
}
```

**3. Protected Route Component**

**File:** `client/src/components/Auth/ProtectedRoute.tsx`

```typescript
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '~/hooks/useAuth';

export function ProtectedRoute() {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return <Outlet />;
}
```

**4. Route Setup**

**File:** `client/src/App.tsx`

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { ProtectedRoute } from '~/components/Auth/ProtectedRoute';
import { Login } from '~/components/Auth/Login';
import { Chat } from '~/components/Chat';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/login" element={<Login />} />

        {/* Protected routes */}
        <Route element={<ProtectedRoute />}>
          <Route path="/" element={<Chat />} />
          <Route path="/settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Making API Calls

### Queries (GET requests)

**Example: Get conversations**

**File:** `client/src/hooks/useConversations.ts`

```typescript
import { useQuery } from '@tanstack/react-query';
import apiClient from '~/utils/apiClient';

interface Conversation {
  id: string;
  title: string;
  model: string;
  createdAt: string;
  updatedAt: string;
}

/**
 * Hook: Get all conversations
 */
export function useConversations() {
  return useQuery({
    queryKey: ['conversations'],

    queryFn: async () => {
      const { data } = await apiClient.get<{ conversations: Conversation[] }>(
        '/api/conversations'
      );
      return data.conversations;
    },

    // Custom options
    staleTime: 1000 * 60 * 5, // 5 minutes
    refetchOnWindowFocus: true
  });
}

/**
 * Hook: Get single conversation
 */
export function useConversation(conversationId: string) {
  return useQuery({
    queryKey: ['conversation', conversationId],

    queryFn: async () => {
      const { data } = await apiClient.get<{ conversation: Conversation }>(
        `/api/conversations/${conversationId}`
      );
      return data.conversation;
    },

    enabled: !!conversationId // Only run if conversationId exists
  });
}

/**
 * Hook: Get messages for conversation
 */
export function useMessages(conversationId: string) {
  return useQuery({
    queryKey: ['messages', conversationId],

    queryFn: async () => {
      const { data } = await apiClient.get(`/api/messages/${conversationId}`);
      return data.messages;
    },

    enabled: !!conversationId,
    staleTime: 0 // Always fetch fresh messages
  });
}
```

**Using in component:**

```typescript
function ConversationList() {
  const { data: conversations, isLoading, error } = useConversations();

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      {conversations?.map(conv => (
        <ConversationCard key={conv.id} conversation={conv} />
      ))}
    </div>
  );
}
```

### Mutations (POST/PUT/DELETE requests)

**Example: Create message**

**File:** `client/src/hooks/useMessages.ts`

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import apiClient from '~/utils/apiClient';

interface CreateMessageParams {
  conversationId: string;
  text: string;
  model: string;
}

/**
 * Hook: Create message (send to AI)
 */
export function useCreateMessage() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (params: CreateMessageParams) => {
      const { data } = await apiClient.post('/api/messages', params);
      return data.message;
    },

    onMutate: async (params) => {
      // Cancel outgoing queries for this conversation
      await queryClient.cancelQueries({
        queryKey: ['messages', params.conversationId]
      });

      // Snapshot previous value
      const previousMessages = queryClient.getQueryData([
        'messages',
        params.conversationId
      ]);

      // Optimistically update UI
      queryClient.setQueryData(
        ['messages', params.conversationId],
        (old: any[]) => [
          ...(old || []),
          {
            id: 'temp-' + Date.now(),
            role: 'user',
            text: params.text,
            createdAt: new Date().toISOString(),
            isOptimistic: true
          }
        ]
      );

      // Return context for rollback
      return { previousMessages };
    },

    onSuccess: (newMessage, params) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({
        queryKey: ['messages', params.conversationId]
      });

      // Update conversation list (new message might change updatedAt)
      queryClient.invalidateQueries({
        queryKey: ['conversations']
      });
    },

    onError: (error, params, context) => {
      // Rollback on error
      if (context?.previousMessages) {
        queryClient.setQueryData(
          ['messages', params.conversationId],
          context.previousMessages
        );
      }
    }
  });
}

/**
 * Hook: Delete message
 */
export function useDeleteMessage() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ messageId }: { messageId: string }) => {
      await apiClient.delete(`/api/messages/${messageId}`);
    },

    onSuccess: (_, variables) => {
      // Invalidate messages queries
      queryClient.invalidateQueries({ queryKey: ['messages'] });
    }
  });
}
```

**Using in component:**

```typescript
function ChatInput({ conversationId }: { conversationId: string }) {
  const [text, setText] = useState('');
  const createMessage = useCreateMessage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!text.trim()) return;

    try {
      await createMessage.mutateAsync({
        conversationId,
        text,
        model: 'gpt-4'
      });

      setText(''); // Clear input on success

    } catch (error) {
      console.error('Failed to send message:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Type your message..."
        disabled={createMessage.isPending}
      />

      <button type="submit" disabled={createMessage.isPending}>
        {createMessage.isPending ? 'Sending...' : 'Send'}
      </button>

      {createMessage.isError && (
        <p className="error">Failed to send message</p>
      )}
    </form>
  );
}
```

---

## Error Handling Across the Stack

### Error Types

**Backend error response format:**

```typescript
// Standard error response from API
interface ApiError {
  error: {
    message: string;
    status: number;
    details?: any;
  };
}
```

### Frontend Error Handling

**1. Query Error Handling**

```typescript
function ConversationList() {
  const {
    data: conversations,
    isLoading,
    error,
    refetch
  } = useConversations();

  if (isLoading) {
    return <Spinner />;
  }

  if (error) {
    const errorMessage =
      error.response?.data?.error?.message || 'Failed to load conversations';

    return (
      <div className="error">
        <p>{errorMessage}</p>
        <button onClick={() => refetch()}>Try Again</button>
      </div>
    );
  }

  return <div>{/* Render conversations */}</div>;
}
```

**2. Mutation Error Handling**

```typescript
function DeleteConversationButton({ conversationId }: { conversationId: string }) {
  const deleteConversation = useDeleteConversation();

  const handleDelete = async () => {
    try {
      await deleteConversation.mutateAsync({ conversationId });
      toast.success('Conversation deleted');

    } catch (error: any) {
      const message = error.response?.data?.error?.message || 'Delete failed';
      toast.error(message);
    }
  };

  return (
    <button onClick={handleDelete} disabled={deleteConversation.isPending}>
      Delete
    </button>
  );
}
```

**3. Global Error Boundary**

```typescript
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = {
    hasError: false,
    error: null
  };

  static getDerivedStateFromError(error: Error): State {
    return {
      hasError: true,
      error
    };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error boundary caught:', error, errorInfo);

    // Send to error tracking service
    // logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-page">
          <h1>Something went wrong</h1>
          <p>{this.state.error?.message}</p>
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

## Real-time Communication: Server-Sent Events

**Used for streaming AI responses in real-time**

### Backend: SSE Endpoint

**File:** `api/server/controllers/messagesController.js`

```javascript
/**
 * POST /api/messages/stream
 * Stream AI response using Server-Sent Events
 */
async function streamMessage(req, res) {
  const { text, conversationId, model } = req.body;
  const userId = req.user.id;

  // Set SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    // Save user message
    const userMessage = await Message.create({
      conversationId,
      userId,
      role: 'user',
      text
    });

    // Send user message
    res.write(`data: ${JSON.stringify({ type: 'userMessage', message: userMessage })}\n\n`);

    // Stream AI response
    let fullResponse = '';

    const stream = await openai.chat.completions.create({
      model,
      messages: [{ role: 'user', content: text }],
      stream: true
    });

    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content || '';

      if (content) {
        fullResponse += content;

        // Send chunk to client
        res.write(`data: ${JSON.stringify({ type: 'chunk', content })}\n\n`);
      }
    }

    // Save AI message
    const aiMessage = await Message.create({
      conversationId,
      userId,
      role: 'assistant',
      text: fullResponse,
      model
    });

    // Send complete message
    res.write(`data: ${JSON.stringify({ type: 'complete', message: aiMessage })}\n\n`);

    res.end();

  } catch (error) {
    res.write(`data: ${JSON.stringify({ type: 'error', error: error.message })}\n\n`);
    res.end();
  }
}
```

### Frontend: SSE Client

**File:** `client/src/hooks/useStreamMessage.ts`

```typescript
import { useState } from 'react';
import { useQueryClient } from '@tanstack/react-query';

interface StreamMessageParams {
  conversationId: string;
  text: string;
  model: string;
}

export function useStreamMessage() {
  const [isStreaming, setIsStreaming] = useState(false);
  const [streamedText, setStreamedText] = useState('');
  const queryClient = useQueryClient();

  const streamMessage = async (params: StreamMessageParams) => {
    setIsStreaming(true);
    setStreamedText('');

    const token = localStorage.getItem('authToken');

    const response = await fetch('http://localhost:3080/api/messages/stream', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify(params)
    });

    const reader = response.body?.getReader();
    const decoder = new TextDecoder();

    if (!reader) {
      throw new Error('No reader');
    }

    try {
      while (true) {
        const { done, value } = await reader.read();

        if (done) break;

        // Decode chunk
        const chunk = decoder.decode(value);

        // Parse SSE format (data: {...}\n\n)
        const lines = chunk.split('\n\n');

        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = JSON.parse(line.substring(6));

            if (data.type === 'chunk') {
              setStreamedText((prev) => prev + data.content);

            } else if (data.type === 'complete') {
              // Invalidate messages query to refetch
              queryClient.invalidateQueries({
                queryKey: ['messages', params.conversationId]
              });

            } else if (data.type === 'error') {
              throw new Error(data.error);
            }
          }
        }
      }
    } finally {
      setIsStreaming(false);
      setStreamedText('');
    }
  };

  return {
    streamMessage,
    isStreaming,
    streamedText
  };
}
```

**Using in component:**

```typescript
function ChatInput({ conversationId }: { conversationId: string }) {
  const [text, setText] = useState('');
  const { streamMessage, isStreaming, streamedText } = useStreamMessage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    await streamMessage({
      conversationId,
      text,
      model: 'gpt-4'
    });

    setText('');
  };

  return (
    <>
      <form onSubmit={handleSubmit}>
        <textarea
          value={text}
          onChange={(e) => setText(e.target.value)}
          disabled={isStreaming}
        />
        <button type="submit" disabled={isStreaming}>
          Send
        </button>
      </form>

      {isStreaming && (
        <div className="streaming-message">
          <p>{streamedText}</p>
          <Spinner />
        </div>
      )}
    </>
  );
}
```

---

## File Uploads

### Backend: File Upload Endpoint

**File:** `api/server/routes/files.js`

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const router = express.Router();

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1e9);
    cb(null, uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({
  storage,
  limits: {
    fileSize: 10 * 1024 * 1024 // 10MB max
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];

    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});

router.post('/upload', upload.single('file'), async (req, res) => {
  try {
    const file = req.file;

    if (!file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }

    // Save to database
    const fileDoc = await File.create({
      userId: req.user.id,
      filename: file.originalname,
      mimetype: file.mimetype,
      size: file.size,
      path: file.path
    });

    res.json({ file: fileDoc });

  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
```

### Frontend: File Upload Hook

**File:** `client/src/hooks/useFileUpload.ts`

```typescript
import { useMutation } from '@tanstack/react-query';
import apiClient from '~/utils/apiClient';

export function useFileUpload() {
  return useMutation({
    mutationFn: async (file: File) => {
      const formData = new FormData();
      formData.append('file', file);

      const { data } = await apiClient.post('/api/files/upload', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });

      return data.file;
    }
  });
}
```

**Using in component:**

```typescript
function FileUploader() {
  const uploadFile = useFileUpload();

  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];

    if (!file) return;

    try {
      const uploadedFile = await uploadFile.mutateAsync(file);
      console.log('Uploaded:', uploadedFile);

    } catch (error) {
      console.error('Upload failed:', error);
    }
  };

  return (
    <div>
      <input
        type="file"
        onChange={handleFileChange}
        accept="image/*,.pdf"
      />

      {uploadFile.isPending && <p>Uploading...</p>}
      {uploadFile.isError && <p>Upload failed</p>}
      {uploadFile.isSuccess && <p>Uploaded successfully!</p>}
    </div>
  );
}
```

---

## Complete Examples

### Example 1: Create and View Conversation

**Complete flow from UI click to database and back**

```typescript
// ===========================
// 1. Component
// ===========================

function NewConversationButton() {
  const createConversation = useCreateConversation();
  const navigate = useNavigate();

  const handleCreate = async () => {
    const conversation = await createConversation.mutateAsync({
      title: 'New Conversation',
      model: 'gpt-4'
    });

    navigate(`/c/${conversation.id}`);
  };

  return <button onClick={handleCreate}>New Chat</button>;
}

// ===========================
// 2. Hook
// ===========================

function useCreateConversation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (params: { title: string; model: string }) => {
      const { data } = await apiClient.post('/api/conversations', params);
      return data.conversation;
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['conversations'] });
    }
  });
}

// ===========================
// 3. API Client (axios interceptor adds token)
// ===========================

// POST http://localhost:3080/api/conversations
// Headers: { Authorization: Bearer eyJhbGc... }
// Body: { title: 'New Conversation', model: 'gpt-4' }

// ===========================
// 4. Express Route
// ===========================

router.post('/conversations', requireAuth, conversationsController.create);

// ===========================
// 5. Controller
// ===========================

async function create(req, res, next) {
  try {
    const { title, model } = req.body;
    const userId = req.user.id;

    const conversation = await conversationService.create({
      title,
      model,
      userId
    });

    res.status(201).json({ conversation });
  } catch (error) {
    next(error);
  }
}

// ===========================
// 6. Service
// ===========================

async function create({ title, model, userId }) {
  const conversation = await Conversation.create({
    title,
    model,
    userId
  });

  return conversation;
}

// ===========================
// 7. Model (Mongoose)
// ===========================

await Conversation.create({
  title: 'New Conversation',
  model: 'gpt-4',
  userId: ObjectId('...')
});

// ===========================
// 8. MongoDB
// ===========================

// Inserts document into 'conversations' collection

// ===========================
// 9. Response flows back
// ===========================

// Response: { conversation: { id: '...', title: '...', ... } }
// TanStack Query caches it
// Component re-renders with new data
```

---

## Best Practices

### 1. Use Query Keys Consistently

```typescript
// ✅ Good: Consistent key structure
['conversations']                    // All conversations
['conversation', conversationId]     // Single conversation
['messages', conversationId]         // Messages for conversation
['user']                            // Current user

// ❌ Bad: Inconsistent keys
['conv']
['getConversation', conversationId]
['convo-messages-' + conversationId]
```

### 2. Invalidate Related Queries

```typescript
// When creating a message, invalidate:
queryClient.invalidateQueries({ queryKey: ['messages', conversationId] });
queryClient.invalidateQueries({ queryKey: ['conversations'] }); // (updatedAt changed)
```

### 3. Use Optimistic Updates for Instant Feedback

```typescript
onMutate: async (newMessage) => {
  // Cancel queries
  await queryClient.cancelQueries({ queryKey: ['messages', conversationId] });

  // Snapshot
  const previous = queryClient.getQueryData(['messages', conversationId]);

  // Optimistic update
  queryClient.setQueryData(['messages', conversationId], (old) => [...old, newMessage]);

  return { previous };
},

onError: (err, variables, context) => {
  // Rollback
  queryClient.setQueryData(['messages', conversationId], context.previous);
}
```

### 4. Handle Loading and Error States

```typescript
function Component() {
  const { data, isLoading, error } = useQuery(...);

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  if (!data) return null;

  return <div>{/* Render data */}</div>;
}
```

### 5. Use TypeScript for Type Safety

```typescript
// Define API response types
interface ApiResponse<T> {
  data: T;
  meta?: {
    page: number;
    total: number;
  };
}

// Use in query
const { data } = await apiClient.get<ApiResponse<Conversation[]>>('/api/conversations');
```

---

## Troubleshooting

### CORS Errors

**Error:** "Access to fetch at '...' from origin '...' has been blocked by CORS policy"

**Solution:**

```javascript
// Backend: api/server/index.js
const cors = require('cors');

app.use(cors({
  origin: 'http://localhost:3000', // React dev server
  credentials: true
}));
```

### 401 Unauthorized

**Error:** "Request failed with status code 401"

**Causes:**
- Missing token
- Expired token
- Invalid token

**Solution:**
```typescript
// Check token is saved
const token = localStorage.getItem('authToken');
console.log('Token:', token);

// Check token is sent
apiClient.interceptors.request.use((config) => {
  console.log('Headers:', config.headers);
  return config;
});
```

### Query Not Updating

**Problem:** Data doesn't update after mutation

**Solution:**
```typescript
// Invalidate queries after mutation
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ['messages'] });
}
```

---

## Summary

**Integration Flow:**

1. **Frontend:** React component calls custom hook
2. **Hook:** Uses TanStack Query for caching/state
3. **API Client:** Axios sends HTTP request with token
4. **Backend:** Express route → middleware → controller → service
5. **Database:** Mongoose model → MongoDB collection
6. **Response:** Flows back through layers
7. **Cache:** TanStack Query updates cache
8. **UI:** Component re-renders with new data

**Key Technologies:**
- **Axios:** HTTP client
- **TanStack Query:** Server state management
- **JWT:** Authentication tokens
- **SSE:** Real-time streaming
- **FormData:** File uploads

---

**Next Steps:**

- Read [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step tasks
- Read [TESTING_GUIDE.md](./TESTING_GUIDE.md) - Testing strategies
- Build a feature end-to-end!

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
