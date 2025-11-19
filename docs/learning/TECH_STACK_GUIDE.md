# 🛠️ LibreChat Technology Stack Guide

**Documented:** November 19, 2025
**Tech Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Time Estimate:** 2-3 hours
**Difficulty:** 🟡 Intermediate

---

## Overview

This guide provides deep dives into each technology used in LibreChat, explaining what it is, why it's used, how it's configured, and how to work with it effectively.

---

## Frontend Technologies

### React 18.2

**What is it?** JavaScript library for building user interfaces using components.

**Why LibreChat uses it:**
- Component-based architecture (reusable UI pieces)
- Virtual DOM for performance
- Huge ecosystem and community
- Excellent developer tools
- Industry standard (most jobs)

**Version Status (Nov 2025):**
- Project: v18.2.0
- Latest: v19.2.0
- Status: ⚠️ One major version behind (stable choice)

**Key React Concepts in LibreChat:**

**1. Function Components**
```typescript
// Modern React pattern (what LibreChat uses)
function ChatMessage({ message }: { message: Message }) {
  return (
    <div className="message">
      <p>{message.text}</p>
    </div>
  )
}
```

**2. Hooks**
```typescript
import { useState, useEffect } from 'react'

function ChatInput() {
  const [text, setText] = useState('')

  useEffect(() => {
    // Side effects here
  }, [])

  return <input value={text} onChange={e => setText(e.target.value)} />
}
```

**Used in LibreChat:**
- `client/src/components/` - All components
- `client/src/hooks/` - Custom hooks

**Resources:**
- Docs: https://react.dev/
- Release notes: https://react.dev/blog/2024/12/05/react-19

---

### TypeScript 5.3.3

**What is it?** JavaScript with static type checking.

**Why LibreChat uses it:**
- Catch errors before runtime
- Better IDE autocomplete/IntelliSense
- Self-documenting code
- Refactoring confidence
- Prevents common bugs

**Example:**
```typescript
// Without TypeScript (JavaScript)
function sendMessage(message) {
  api.post('/messages', message) // What shape is message?
}

// With TypeScript
interface Message {
  text: string
  conversationId: string
}

function sendMessage(message: Message) {
  api.post('/messages', message) // Message shape enforced!
}
```

**LibreChat TypeScript Setup:**
- `tsconfig.json` in client/, packages/
- Strict mode enabled
- Path aliases configured

**Common Patterns:**
```typescript
// Type imports
import type { Message, Conversation } from '@librechat/data-schemas'

// Interface for props
interface ChatViewProps {
  conversationId: string
  onSend: (text: string) => void
}

// Generic types
const [messages, setMessages] = useState<Message[]>([])
```

**Resources:**
- Docs: https://www.typescriptlang.org/
- Handbook: https://www.typescriptlang.org/docs/handbook/intro.html

---

### Vite 6.4.1

**What is it?** Next-generation frontend build tool.

**Why LibreChat uses it:**
- ⚡ Lightning fast dev server (instant HMR)
- Native ES modules (no bundling in dev)
- Optimized production builds
- Simple configuration
- Much faster than Webpack

**How it works:**
```
Development:
  Source files → Vite dev server → Browser (unbundled, native ESM)
  Change file → Instant HMR update (< 50ms)

Production:
  Source files → Vite build (Rollup) → Optimized bundle
  - Code splitting
  - Tree shaking
  - Minification
```

**LibreChat Vite Config:**
```typescript
// client/vite.config.ts
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '~': path.resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3090,
    proxy: {
      '/api': 'http://localhost:3080' // Proxy API requests to backend
    }
  }
})
```

**Key Features Used:**
- HMR (Hot Module Replacement)
- Path aliases (`~` → `src/`)
- API proxying (dev server)
- Production optimizations

**Resources:**
- Docs: https://vite.dev/
- Config reference: https://vite.dev/config/

---

### TanStack Query 4.28 (React Query)

**What is it?** Powerful data fetching and caching library.

**Why LibreChat uses it:**
- Eliminates useEffect boilerplate
- Automatic caching and deduplication
- Background refetching
- Optimistic updates
- Error retry logic

**Example:**
```typescript
// Traditional approach (lots of boilerplate)
function Messages() {
  const [messages, setMessages] = useState([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)

  useEffect(() => {
    setLoading(true)
    fetch('/api/messages')
      .then(res => res.json())
      .then(data => setMessages(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false))
  }, [])

  // Handle loading, error states...
}

// TanStack Query approach (simple!)
function Messages() {
  const { data: messages, isLoading, error } = useQuery({
    queryKey: ['messages'],
    queryFn: () => fetch('/api/messages').then(res => res.json())
  })

  // That's it! Caching, refetching, etc. handled automatically
}
```

**LibreChat Usage:**
```typescript
// packages/data-provider/src/queries.ts
export const useMessagesQuery = (conversationId: string) => {
  return useQuery({
    queryKey: ['messages', conversationId],
    queryFn: () => apiClient.get(`/messages/${conversationId}`)
  })
}

// In components:
const { data: messages } = useMessagesQuery(conversationId)
```

**Key Concepts:**
- **Query Keys:** Unique identifiers for cached data
- **Query Functions:** Async functions that fetch data
- **Mutations:** For POST/PUT/DELETE operations
- **Invalidation:** Refetch data when it changes

**Resources:**
- Docs: https://tanstack.com/query/latest
- v4 docs: https://tanstack.com/query/v4

---

### Tailwind CSS 3.4.1

**What is it?** Utility-first CSS framework.

**Why LibreChat uses it:**
- Rapid development (no switching files)
- Consistent design system
- Responsive design built-in
- Small bundle size (unused purged)
- No CSS naming conflicts

**Example:**
```tsx
// Traditional CSS
<div className="chat-message">
  <p className="message-text">Hello</p>
</div>

/* styles.css */
.chat-message {
  display: flex;
  padding: 1rem;
  margin-bottom: 0.5rem;
  background-color: #f3f4f6;
  border-radius: 0.5rem;
}

// Tailwind approach (all in JSX)
<div className="flex p-4 mb-2 bg-gray-100 rounded-lg">
  <p>Hello</p>
</div>
```

**Common Tailwind Utilities:**
```tsx
// Layout
className="flex flex-col items-center justify-between"

// Spacing
className="p-4 m-2 gap-4"  // padding, margin, gap

// Colors
className="bg-blue-500 text-white hover:bg-blue-600"

// Responsive
className="text-sm md:text-base lg:text-lg"  // mobile → tablet → desktop

// Dark mode
className="bg-white dark:bg-gray-800"
```

**LibreChat Tailwind Config:**
```javascript
// client/tailwind.config.cjs
module.exports = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      colors: {
        // Custom colors
      }
    }
  },
  plugins: [
    require('tailwindcss-radix')
  ]
}
```

**Resources:**
- Docs: https://tailwindcss.com/
- Cheat sheet: https://nerdcave.com/tailwind-cheat-sheet

---

### Radix UI

**What is it?** Unstyled, accessible UI components.

**Why LibreChat uses it:**
- Accessibility built-in (ARIA, keyboard nav)
- Unstyled (full design control)
- Composable primitives
- TypeScript support

**Example:**
```tsx
import * as Dialog from '@radix-ui/react-dialog'

function Modal() {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>
        <button>Open</button>
      </Dialog.Trigger>

      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/50" />
        <Dialog.Content className="fixed top-1/2 left-1/2 ...">
          <Dialog.Title>Title</Dialog.Title>
          <Dialog.Description>Description</Dialog.Description>
          {/* Content */}
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  )
}
```

**Used Components:**
- Dialog, Dropdown Menu, Popover
- Accordion, Tabs, Toast
- Checkbox, Radio Group, Switch

**Resources:**
- Docs: https://www.radix-ui.com/

---

### Jotai & Recoil (State Management)

**What are they?** Atomic state management libraries.

**Why LibreChat uses them:**
- Simpler than Redux
- Component-only re-renders
- TypeScript friendly
- Minimal boilerplate

**Jotai Example:**
```typescript
import { atom, useAtom } from 'jotai'

// Define atom (state)
const userAtom = atom({ name: '', email: '' })

// Use in component
function Profile() {
  const [user, setUser] = useAtom(userAtom)

  return (
    <div>
      <p>{user.name}</p>
      <button onClick={() => setUser({ ...user, name: 'John' })}>
        Update
      </button>
    </div>
  )
}
```

**LibreChat Usage:**
```typescript
// client/src/store/atoms.ts
export const conversationAtom = atom<Conversation | null>(null)
export const messagesAtom = atom<Message[]>([])

// In components
const [conversation, setConversation] = useAtom(conversationAtom)
```

**Resources:**
- Jotai: https://jotai.org/
- Recoil: https://recoiljs.org/

---

## Backend Technologies

### Node.js

**What is it?** JavaScript runtime for server-side code.

**Why LibreChat uses it:**
- Same language as frontend (JavaScript/TypeScript)
- Non-blocking I/O (perfect for APIs)
- Huge npm ecosystem
- Great for real-time apps

**Key Concepts:**

**1. Event Loop**
```javascript
// Non-blocking (good!)
console.log('Start')
setTimeout(() => console.log('Async'), 0)
console.log('End')
// Output: Start, End, Async

// Blocking (bad!)
const result = someSyncOperation() // Blocks entire server!
```

**2. Modules**
```javascript
// CommonJS (Node.js default)
const express = require('express')
module.exports = { /* ... */ }

// ES Modules (with "type": "module")
import express from 'express'
export { /* ... */ }
```

**Resources:**
- Docs: https://nodejs.org/docs/latest/api/
- Best practices: https://github.com/goldbergyoni/nodebestpractices

---

### Express 4.21

**What is it?** Minimal web framework for Node.js.

**Why LibreChat uses it:**
- Simple and unopinionated
- Huge middleware ecosystem
- Industry standard
- Easy to learn

**Basic Express App:**
```javascript
const express = require('express')
const app = express()

// Middleware
app.use(express.json()) // Parse JSON bodies

// Routes
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' })
})

app.post('/api/messages', async (req, res) => {
  const { text } = req.body
  const message = await MessageService.create(text)
  res.json(message)
})

// Start server
app.listen(3080, () => {
  console.log('Server running on port 3080')
})
```

**Key Concepts:**

**1. Middleware Chain**
```javascript
app.post('/api/messages',
  requireAuth,        // 1. Check authentication
  validateRequest,    // 2. Validate input
  createMessage       // 3. Handle request
)
```

**2. Request/Response**
```javascript
function handler(req, res) {
  req.body      // Request body (parsed JSON)
  req.params    // URL parameters (/users/:id)
  req.query     // Query string (?page=1)
  req.headers   // HTTP headers

  res.json({ data: 'value' })    // Send JSON
  res.status(404).send('Not found')
  res.redirect('/login')
}
```

**LibreChat Pattern:**
```javascript
// api/server/routes/messages.js
router.post('/', requireJwtAuth, MessageController.create)

// api/server/controllers/MessageController.js
async create(req, res) {
  const message = await MessageService.create(req.body, req.user.id)
  res.json(message)
}
```

**Resources:**
- Docs: https://expressjs.com/
- Guide: https://expressjs.com/en/guide/routing.html

---

### MongoDB 8.12

**What is it?** NoSQL document database.

**Why LibreChat uses it:**
- JSON-native (stores objects)
- Flexible schema
- Great for chat data
- Horizontal scaling

**Document Example:**
```javascript
// Message document
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  conversationId: ObjectId("..."),
  text: "Hello, how are you?",
  sender: "user",
  userId: ObjectId("..."),
  metadata: {
    model: "gpt-4",
    tokens: 50
  },
  createdAt: ISODate("2025-11-19T10:30:00Z")
}
```

**Basic Operations:**
```javascript
// mongosh (MongoDB shell)
use LibreChat

// Find
db.messages.find({ userId: ObjectId("...") })

// Insert
db.messages.insertOne({
  text: "Hello",
  userId: ObjectId("...")
})

// Update
db.messages.updateOne(
  { _id: ObjectId("...") },
  { $set: { text: "Updated" } }
)

// Delete
db.messages.deleteOne({ _id: ObjectId("...") })

// Aggregation
db.messages.aggregate([
  { $match: { userId: ObjectId("...") } },
  { $group: { _id: "$conversationId", count: { $sum: 1 } } }
])
```

**Resources:**
- Docs: https://www.mongodb.com/docs/manual/
- Query guide: https://www.mongodb.com/docs/manual/tutorial/query-documents/

---

### Mongoose 8.12

**What is it?** MongoDB ODM (Object Document Mapper).

**Why LibreChat uses it:**
- Schema validation
- Type casting
- Query builder
- Middleware (hooks)
- Relationships (population)

**Schema Example:**
```javascript
const MessageSchema = new mongoose.Schema({
  text: {
    type: String,
    required: true,
    maxlength: 10000
  },
  conversationId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Conversation',
    required: true
  },
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  sender: {
    type: String,
    enum: ['user', 'assistant'],
    required: true
  },
  metadata: mongoose.Schema.Types.Mixed
}, {
  timestamps: true  // Auto createdAt/updatedAt
})

// Middleware (runs before save)
MessageSchema.pre('save', function(next) {
  console.log('Saving message:', this.text)
  next()
})

const Message = mongoose.model('Message', MessageSchema)
```

**Usage:**
```javascript
// Create
const message = await Message.create({
  text: "Hello",
  conversationId: convoId,
  userId: userId,
  sender: "user"
})

// Find
const messages = await Message.find({ conversationId: convoId })

// Population (like SQL JOIN)
const messages = await Message.find({ conversationId: convoId })
  .populate('userId', 'name email')  // Get user details
```

**Resources:**
- Docs: https://mongoosejs.com/
- Guide: https://mongoosejs.com/docs/guide.html

---

### Redis 7.x (ioredis 5.3.2)

**What is it?** In-memory key-value store.

**Why LibreChat uses it:**
- Extremely fast (sub-millisecond)
- Session storage
- Caching
- Rate limiting

**Basic Operations:**
```javascript
const Redis = require('ioredis')
const redis = new Redis()

// Set/Get
await redis.set('key', 'value')
const value = await redis.get('key')

// Expiration
await redis.setex('key', 3600, 'value')  // Expires in 1 hour

// Hash (object storage)
await redis.hset('user:123', 'name', 'John')
await redis.hget('user:123', 'name')

// Lists
await redis.lpush('messages', 'Hello')
await redis.lrange('messages', 0, -1)

// Pub/Sub
await redis.publish('channel', 'message')
await redis.subscribe('channel')
```

**LibreChat Usage:**
```javascript
// Session storage
await redis.set(`session:${sessionId}`, JSON.stringify(sessionData))

// Rate limiting
const count = await redis.incr(`ratelimit:${userId}`)
if (count > 100) throw new Error('Rate limit exceeded')
await redis.expire(`ratelimit:${userId}`, 3600)
```

**Resources:**
- Docs: https://redis.io/docs/
- ioredis: https://github.com/redis/ioredis

---

### Passport.js

**What is it?** Authentication middleware.

**Why LibreChat uses it:**
- Strategy-based (JWT, Local, OAuth, etc.)
- Well-tested
- Flexible

**Strategies Used:**

**1. Local (username/password)**
```javascript
passport.use(new LocalStrategy({
  usernameField: 'email'
}, async (email, password, done) => {
  const user = await User.findOne({ email })
  if (!user) return done(null, false)

  const isValid = await bcrypt.compare(password, user.password)
  if (!isValid) return done(null, false)

  return done(null, user)
}))
```

**2. JWT (token-based)**
```javascript
passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET
}, async (payload, done) => {
  const user = await User.findById(payload.userId)
  return done(null, user)
}))
```

**Resources:**
- Docs: https://www.passportjs.org/

---

## Build Tools & Development

### npm Workspaces

**What is it?** Monorepo management in npm.

**Why LibreChat uses it:**
- Share dependencies
- Cross-package development
- Single lock file

**Structure:**
```json
// Root package.json
{
  "workspaces": [
    "client",
    "api",
    "packages/*"
  ]
}
```

**Benefits:**
- Install once: `npm install` (installs all packages)
- Link packages: `@librechat/data-provider` available everywhere
- Shared dependencies: React installed once, used by all

---

### ESLint & Prettier

**What are they?** Code linting and formatting tools.

**Why use them:**
- Consistent code style
- Catch errors early
- Auto-fix issues

**Usage:**
```bash
npm run lint        # Check for issues
npm run lint:fix    # Auto-fix issues
npm run format      # Format with Prettier
```

---

### Jest 30.2

**What is it?** JavaScript testing framework.

**Why LibreChat uses it:**
- Fast and parallel
- Snapshot testing
- Mocking built-in
- Great DX

**Example:**
```javascript
describe('MessageService', () => {
  it('creates a message', async () => {
    const message = await MessageService.create({
      text: 'Hello',
      conversationId: 'abc123'
    })

    expect(message.text).toBe('Hello')
    expect(message.conversationId).toBe('abc123')
  })
})
```

**Resources:**
- Docs: https://jestjs.io/

---

### Playwright 1.56

**What is it?** End-to-end testing framework.

**Why LibreChat uses it:**
- Tests real user flows
- Multi-browser
- Auto-wait (no flaky tests)

**Example:**
```javascript
test('user can send message', async ({ page }) => {
  await page.goto('http://localhost:3080')
  await page.fill('[data-testid="message-input"]', 'Hello')
  await page.click('[data-testid="send-button"]')

  await expect(page.locator('.message')).toContainText('Hello')
})
```

**Resources:**
- Docs: https://playwright.dev/

---

## AI Integration

### OpenAI SDK 5.8.2

**What is it?** Official OpenAI API client.

**Usage:**
```javascript
const OpenAI = require('openai')
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
})

const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [
    { role: 'system', content: 'You are helpful' },
    { role: 'user', content: 'Hello!' }
  ]
})

console.log(response.choices[0].message.content)
```

---

### Anthropic SDK 0.52.0

**What is it?** Official Anthropic (Claude) API client.

**Usage:**
```javascript
const Anthropic = require('@anthropic-ai/sdk')
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
})

const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }]
})
```

---

## Summary

LibreChat uses a **modern, production-ready tech stack** with:

**Frontend:**
- React 18 + TypeScript
- Vite (build tool)
- TanStack Query (data fetching)
- Tailwind CSS (styling)
- Radix UI (accessible components)

**Backend:**
- Node.js + Express
- MongoDB + Mongoose
- Redis (caching/sessions)
- Passport.js (auth)

**Quality:**
- Jest (unit tests)
- Playwright (E2E tests)
- ESLint + Prettier (code quality)

**AI:**
- OpenAI, Anthropic, Google SDKs

All technologies are either **current** or **1-2 versions behind** (stable, production-ready choice).

---

**Next Steps:**
- [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - See how these technologies work together
- [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - Deep dive into React patterns
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Deep dive into Express patterns

---

**Back to:** [Learning Path Home](./README.md)

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
