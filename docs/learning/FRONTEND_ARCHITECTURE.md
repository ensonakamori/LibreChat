# ⚛️ LibreChat Frontend Architecture

**Documented:** November 19, 2025
**Prerequisites:** [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
**Time Estimate:** 3-4 hours
**Difficulty:** 🟡 Intermediate

---

## Overview

LibreChat's frontend is a **React Single Page Application (SPA)** built with modern patterns and tools. This guide explores the architecture, patterns, and best practices used.

---

## Table of Contents

- [Architecture Summary](#architecture-summary)
- [Component Organization](#component-organization)
- [State Management](#state-management)
- [Data Fetching](#data-fetching)
- [Routing](#routing)
- [Styling](#styling)
- [Form Handling](#form-handling)
- [TypeScript Patterns](#typescript-patterns)
- [Performance Optimization](#performance-optimization)
- [Key Patterns](#key-patterns)

---

## Architecture Summary

```
client/src/
├── components/     # React components (UI)
├── hooks/          # Custom React hooks (logic)
├── store/          # Global state (Jotai/Recoil)
├── data-provider/  # API client (TanStack Query)
├── routes/         # Pages (React Router)
├── utils/          # Helper functions
└── App.tsx         # Root component
```

**Architecture Pattern:** Feature-based component organization with centralized state and data fetching.

---

## Component Organization

### Directory Structure

```
components/
├── Chat/                    # Main chat feature
│   ├── ChatView.tsx        # Container component
│   ├── Input/              # Input sub-feature
│   │   ├── ChatInput.tsx
│   │   └── SubmitButton.tsx
│   ├── Messages/           # Messages sub-feature
│   │   ├── MessageList.tsx
│   │   └── Message.tsx
│   └── Menus/              # Chat menus
│
├── Conversations/           # Conversation sidebar
│   ├── Conversations.tsx
│   ├── Convo.tsx
│   └── ConvoOptions/
│
├── ui/                      # Reusable UI components
│   ├── Button.tsx
│   ├── Dialog.tsx
│   ├── Input.tsx
│   └── ...
│
└── Auth/                    # Authentication
    ├── Login.tsx
    └── Register.tsx
```

### Component Patterns

**1. Container vs Presentational**

```typescript
// Container Component (smart - handles logic & data)
function ChatView() {
  const { data: messages, isLoading } = useMessagesQuery(conversationId)
  const sendMutation = useSendMessageMutation()

  const handleSend = (text: string) => {
    sendMutation.mutate({ text, conversationId })
  }

  return (
    <ChatPresentation
      messages={messages}
      isLoading={isLoading}
      onSend={handleSend}
    />
  )
}

// Presentational Component (dumb - just renders)
interface ChatPresentationProps {
  messages: Message[]
  isLoading: boolean
  onSend: (text: string) => void
}

function ChatPresentation({ messages, isLoading, onSend }: ChatPresentationProps) {
  if (isLoading) return <Spinner />

  return (
    <div>
      <MessageList messages={messages} />
      <ChatInput onSend={onSend} />
    </div>
  )
}
```

**2. Compound Components**

```typescript
// Parent component provides context
function Dialog({ children, ...props }: DialogProps) {
  return (
    <DialogPrimitive.Root {...props}>
      {children}
    </DialogPrimitive.Root>
  )
}

// Child components consume context
Dialog.Trigger = DialogTrigger
Dialog.Content = DialogContent
Dialog.Title = DialogTitle

// Usage
<Dialog>
  <Dialog.Trigger>Open</Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Title>Title</Dialog.Title>
    {/* Content */}
  </Dialog.Content>
</Dialog>
```

**3. Render Props**

```typescript
function MouseTracker({ render }: { render: (position: { x: number, y: number }) => React.ReactNode }) {
  const [position, setPosition] = useState({ x: 0, y: 0 })

  const handleMouseMove = (e: React.MouseEvent) => {
    setPosition({ x: e.clientX, y: e.clientY })
  }

  return (
    <div onMouseMove={handleMouseMove}>
      {render(position)}
    </div>
  )
}

// Usage
<MouseTracker render={({ x, y }) => (
  <p>Mouse at {x}, {y}</p>
)} />
```

### File Naming Conventions

```
components/
├── PascalCase.tsx         # React components
├── PascalCase.test.tsx    # Component tests
└── index.ts               # Barrel export

hooks/
├── useCamelCase.ts        # Custom hooks
└── useCamelCase.test.ts   # Hook tests

utils/
├── camelCase.ts           # Utility functions
└── camelCase.test.ts      # Utility tests
```

---

## State Management

LibreChat uses **atomic state management** with Jotai and Recoil (simpler than Redux).

### Jotai Atoms

```typescript
// client/src/store/atoms.ts
import { atom } from 'jotai'

// Simple atom
export const userAtom = atom<User | null>(null)

// Derived atom (computed)
export const userEmailAtom = atom(
  (get) => get(userAtom)?.email ?? ''
)

// Writable derived atom
export const conversationAtom = atom(
  (get) => get(conversationsAtom).find(c => c.id === get(activeIdAtom)),
  (get, set, newConversation: Conversation) => {
    set(conversationsAtom, conversations =>
      conversations.map(c => c.id === newConversation.id ? newConversation : c)
    )
  }
)
```

### Using Atoms in Components

```typescript
import { useAtom, useAtomValue, useSetAtom } from 'jotai'

function UserProfile() {
  // Read and write
  const [user, setUser] = useAtom(userAtom)

  // Read only
  const email = useAtomValue(userEmailAtom)

  // Write only
  const setConversation = useSetAtom(conversationAtom)

  return (
    <div>
      <p>{user?.name}</p>
      <p>{email}</p>
      <button onClick={() => setUser({ ...user, name: 'Updated' })}>
        Update
      </button>
    </div>
  )
}
```

### When to Use Atoms vs React Query

**Use Atoms for:**
- UI state (sidebar open/closed, selected theme)
- User preferences
- Form state (when not using react-hook-form)
- Global UI state

**Use React Query for:**
- Server data (messages, conversations, user data)
- Any data from API

```typescript
// ❌ Wrong - using atoms for server data
const [messages, setMessages] = useAtom(messagesAtom)

useEffect(() => {
  fetch('/api/messages').then(res => setMessages(res.json()))
}, [])

// ✅ Right - using React Query for server data
const { data: messages } = useMessagesQuery()
```

---

## Data Fetching

LibreChat uses **TanStack Query (React Query)** for all server data.

### Query Hooks

```typescript
// packages/data-provider/src/queries.ts
import { useQuery } from '@tanstack/react-query'

export const useMessagesQuery = (conversationId: string) => {
  return useQuery({
    queryKey: ['messages', conversationId],
    queryFn: async () => {
      const res = await apiClient.get(`/api/messages/${conversationId}`)
      return res.data
    },
    enabled: !!conversationId,  // Only fetch if conversationId exists
    staleTime: 1000 * 60 * 5,   // Consider fresh for 5 minutes
  })
}
```

### Mutation Hooks

```typescript
export const useSendMessageMutation = () => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (data: { text: string, conversationId: string }) => {
      return apiClient.post('/api/messages', data)
    },
    onSuccess: (data, variables) => {
      // Invalidate and refetch
      queryClient.invalidateQueries(['messages', variables.conversationId])
      queryClient.invalidateQueries(['conversations'])
    },
    onError: (error) => {
      toast.error('Failed to send message')
    }
  })
}
```

### Using Queries in Components

```typescript
function MessageList({ conversationId }: { conversationId: string }) {
  const { data: messages, isLoading, error, refetch } = useMessagesQuery(conversationId)

  if (isLoading) return <Spinner />
  if (error) return <Error message={error.message} onRetry={refetch} />
  if (!messages || messages.length === 0) return <Empty />

  return (
    <div className="message-list">
      {messages.map(message => (
        <Message key={message.id} {...message} />
      ))}
    </div>
  )
}
```

### Optimistic Updates

```typescript
const deleteMessageMutation = useMutation({
  mutationFn: (messageId: string) => apiClient.delete(`/messages/${messageId}`),
  onMutate: async (messageId) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries(['messages'])

    // Snapshot current value
    const previous = queryClient.getQueryData(['messages', conversationId])

    // Optimistically update
    queryClient.setQueryData(['messages', conversationId], (old: Message[]) =>
      old.filter(m => m.id !== messageId)
    )

    // Return context for rollback
    return { previous }
  },
  onError: (err, variables, context) => {
    // Rollback on error
    queryClient.setQueryData(['messages', conversationId], context.previous)
    toast.error('Failed to delete message')
  },
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries(['messages'])
  }
})
```

---

## Routing

LibreChat uses **React Router v6** for client-side routing.

### Route Configuration

```typescript
// client/src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />

        {/* Protected routes */}
        <Route element={<RequireAuth />}>
          <Route path="/chat" element={<ChatView />} />
          <Route path="/chat/:conversationId" element={<ChatView />} />
          <Route path="/agents" element={<Agents />} />
        </Route>

        {/* Redirect root to chat */}
        <Route path="/" element={<Navigate to="/chat" replace />} />

        {/* 404 */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### Protected Routes

```typescript
function RequireAuth() {
  const { data: user, isLoading } = useUserQuery()
  const location = useLocation()

  if (isLoading) return <Spinner />

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }

  return <Outlet />  // Render child routes
}
```

### Navigation

```typescript
import { useNavigate, useParams, useSearchParams } from 'react-router-dom'

function ConversationList() {
  const navigate = useNavigate()
  const { conversationId } = useParams()  // URL params
  const [searchParams] = useSearchParams()  // Query params

  const handleClick = (id: string) => {
    navigate(`/chat/${id}`)  // Programmatic navigation
  }

  return (
    <div>
      {conversations.map(conv => (
        <button
          key={conv.id}
          onClick={() => handleClick(conv.id)}
          className={conv.id === conversationId ? 'active' : ''}
        >
          {conv.title}
        </button>
      ))}
    </div>
  )
}
```

---

## Styling

LibreChat uses **Tailwind CSS** for styling.

### Tailwind Patterns

**1. Responsive Design**

```tsx
<div className="
  flex flex-col          /* Mobile: stack vertically */
  md:flex-row            /* Tablet: horizontal layout */
  lg:gap-8               /* Desktop: larger gap */
">
  <aside className="w-full md:w-64">Sidebar</aside>
  <main className="flex-1">Content</main>
</div>
```

**2. Dark Mode**

```tsx
<div className="
  bg-white dark:bg-gray-900
  text-gray-900 dark:text-white
">
  Content adapts to theme
</div>
```

**3. Component Variants**

```tsx
const buttonVariants = {
  primary: 'bg-blue-600 hover:bg-blue-700 text-white',
  secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-900',
  danger: 'bg-red-600 hover:bg-red-700 text-white',
}

function Button({ variant = 'primary', ...props }) {
  return (
    <button className={`px-4 py-2 rounded ${buttonVariants[variant]}`} {...props} />
  )
}
```

**4. Custom Utilities**

```javascript
// tailwind.config.cjs
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#...',
          500: '#...',
          900: '#...',
        }
      },
      spacing: {
        '128': '32rem',
      }
    }
  }
}

// Usage
<div className="bg-brand-500 p-128">...</div>
```

---

## Form Handling

LibreChat uses **react-hook-form** with **Zod validation**.

### Basic Form

```typescript
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

// Define schema
const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be 8+ characters'),
})

type FormData = z.infer<typeof schema>

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<FormData>({
    resolver: zodResolver(schema)
  })

  const onSubmit = async (data: FormData) => {
    await login(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email')}
        type="email"
        placeholder="Email"
      />
      {errors.email && <p className="text-red-500">{errors.email.message}</p>}

      <input
        {...register('password')}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p className="text-red-500">{errors.password.message}</p>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  )
}
```

### Complex Form with Nested Fields

```typescript
const agentSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500),
  tools: z.array(z.object({
    id: z.string(),
    name: z.string(),
    enabled: z.boolean()
  })),
  config: z.object({
    model: z.string(),
    temperature: z.number().min(0).max(2),
  })
})

function AgentForm() {
  const { register, control, handleSubmit } = useForm({
    resolver: zodResolver(agentSchema)
  })

  return (
    <form>
      <input {...register('name')} />
      <textarea {...register('description')} />

      {/* Array fields */}
      <Controller
        name="tools"
        control={control}
        render={({ field }) => (
          <ToolsSelector value={field.value} onChange={field.onChange} />
        )}
      />

      {/* Nested fields */}
      <input {...register('config.model')} />
      <input {...register('config.temperature')} type="number" step="0.1" />
    </form>
  )
}
```

---

## TypeScript Patterns

### Component Props

```typescript
// Base props
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
  isLoading?: boolean
}

// Discriminated unions
type MessageProps =
  | { type: 'user'; userId: string; text: string }
  | { type: 'assistant'; model: string; text: string; tokensUsed: number }

function Message(props: MessageProps) {
  if (props.type === 'user') {
    // TypeScript knows userId exists here
    return <div>{props.userId}: {props.text}</div>
  } else {
    // TypeScript knows model exists here
    return <div>{props.model} ({props.tokensUsed} tokens): {props.text}</div>
  }
}
```

### Generic Components

```typescript
interface ListProps<T> {
  items: T[]
  renderItem: (item: T) => React.ReactNode
  keyExtractor: (item: T) => string
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <div>
      {items.map(item => (
        <div key={keyExtractor(item)}>
          {renderItem(item)}
        </div>
      ))}
    </div>
  )
}

// Usage
<List
  items={messages}
  renderItem={(message) => <Message {...message} />}
  keyExtractor={(message) => message.id}
/>
```

### Custom Hooks with TypeScript

```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key)
    return stored ? JSON.parse(stored) : initialValue
  })

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])

  return [value, setValue] as const  // Tuple type
}

// Usage
const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light')
```

---

## Performance Optimization

### React.memo

```typescript
// Expensive component
const Message = React.memo(function Message({ text, sender }: MessageProps) {
  return <div>{sender}: {text}</div>
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.text === nextProps.text && prevProps.sender === nextProps.sender
})
```

### useMemo & useCallback

```typescript
function ChatView() {
  const { data: messages } = useMessagesQuery()

  // Expensive computation - only recalculate if messages change
  const sortedMessages = useMemo(() => {
    return messages?.sort((a, b) => a.createdAt - b.createdAt) ?? []
  }, [messages])

  // Stable callback reference - prevent child re-renders
  const handleSend = useCallback((text: string) => {
    sendMessage(text)
  }, [])  // Dependencies

  return <MessageList messages={sortedMessages} onSend={handleSend} />
}
```

### Code Splitting

```typescript
import { lazy, Suspense } from 'react'

// Lazy load components
const AgentEditor = lazy(() => import('./AgentEditor'))

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <AgentEditor />
    </Suspense>
  )
}
```

### Virtual Scrolling

```typescript
import { FixedSizeList } from 'react-window'

function MessageList({ messages }: { messages: Message[] }) {
  const Row = ({ index, style }: { index: number, style: React.CSSProperties }) => (
    <div style={style}>
      <Message {...messages[index]} />
    </div>
  )

  return (
    <FixedSizeList
      height={600}
      itemCount={messages.length}
      itemSize={80}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  )
}
```

---

## Key Patterns

### Error Boundaries

```typescript
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false }

  static getDerivedStateFromError() {
    return { hasError: true }
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('Error caught:', error, info)
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback />
    }

    return this.props.children
  }
}

// Usage
<ErrorBoundary>
  <Chat />
</ErrorBoundary>
```

### Portal Pattern

```typescript
import { createPortal } from 'react-dom'

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">
        {children}
      </div>
    </div>,
    document.body  // Render outside root div
  )
}
```

### Context Pattern

```typescript
const ThemeContext = React.createContext<'light' | 'dark'>('light')

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')

  return (
    <ThemeContext.Provider value={theme}>
      {children}
    </ThemeContext.Provider>
  )
}

// Custom hook
function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) throw new Error('useTheme must be used within ThemeProvider')
  return context
}
```

---

## Summary

LibreChat frontend architecture follows **modern React best practices**:

**Component Organization:**
- Feature-based folders
- Container/Presentational pattern
- Compound components

**State Management:**
- Jotai/Recoil for UI state
- TanStack Query for server state
- Clear separation of concerns

**Data Fetching:**
- TanStack Query for all API calls
- Automatic caching & background refetching
- Optimistic updates

**Routing:**
- React Router v6
- Protected routes
- Programmatic navigation

**Styling:**
- Tailwind CSS utility-first
- Responsive & dark mode
- Component variants

**Forms:**
- react-hook-form + Zod
- Type-safe validation
- Error handling

**Performance:**
- React.memo, useMemo, useCallback
- Code splitting
- Virtual scrolling

---

**Next Steps:**
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Express patterns
- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Frontend ↔ Backend communication
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Build features

---

**Back to:** [Learning Path Home](./README.md)

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
