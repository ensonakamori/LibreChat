# 🔄 LibreChat Data Flow Guide

**Documented:** November 19, 2025
**Prerequisites:** [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
**Time Estimate:** 2 hours
**Difficulty:** 🟡 Intermediate

---

## Overview

This guide traces data flow through LibreChat for key user actions. You'll see exactly how data moves from user click → React → API → Database → AI → back to UI.

---

## Table of Contents

- [Flow 1: User Sends a Message](#flow-1-user-sends-a-message)
- [Flow 2: User Logs In](#flow-2-user-logs-in)
- [Flow 3: Loading Conversations](#flow-3-loading-conversations)
- [Flow 4: File Upload](#flow-4-file-upload)
- [Flow 5: Creating an Agent](#flow-5-creating-an-agent)
- [Key Patterns](#key-patterns)

---

## Flow 1: User Sends a Message

**Scenario:** User types "What is React?" and clicks Send

### Step-by-Step Flow

```mermaid
sequenceDiagram
    participant User
    participant React
    participant TanStack
    participant Express
    participant Controller
    participant Service
    participant MongoDB
    participant OpenAI

    User->>React: Types text & clicks Send
    React->>TanStack: useSendMessageMutation()
    TanStack->>Express: POST /api/messages
    Note over TanStack,Express: Headers: Authorization: Bearer <JWT>

    Express->>Express: requireJwtAuth middleware
    Express->>Controller: MessageController.create()
    Controller->>Service: MessageService.create()

    Service->>MongoDB: Save user message
    MongoDB-->>Service: Message saved

    Service->>MongoDB: Get conversation history
    MongoDB-->>Service: Previous messages

    Service->>Service: Build prompt with context
    Service->>OpenAI: POST /chat/completions
    OpenAI-->>Service: AI response (streaming)

    Service->>Express: Stream to client (SSE)
    Express-->>TanStack: Stream chunks
    TanStack-->>React: Update UI in real-time

    Service->>MongoDB: Save AI response
    MongoDB-->>Service: Saved

    Service-->>Controller: Complete
    Controller-->>Express: 200 OK
    Express-->>TanStack: Final response
    TanStack-->>React: Update cache
    React-->>User: Display complete message
```

### Detailed Breakdown

**1. User Action (Frontend)**

File: `client/src/components/Chat/Input/ChatInput.tsx`

```typescript
function ChatInput() {
  const sendMessageMutation = useSendMessageMutation()

  const handleSend = async () => {
    const messageText = input.trim()

    sendMessageMutation.mutate({
      text: messageText,
      conversationId: currentConversation.id,
      model: selectedModel
    })
  }

  return (
    <form onSubmit={handleSend}>
      <textarea value={input} onChange={e => setInput(e.target.value)} />
      <button type="submit">Send</button>
    </form>
  )
}
```

**2. TanStack Query Mutation**

File: `packages/data-provider/src/mutations.ts`

```typescript
export const useSendMessageMutation = () => {
  return useMutation({
    mutationFn: async (data) => {
      return apiClient.post('/api/messages', data)
    },
    onSuccess: (response) => {
      // Invalidate messages cache to refetch
      queryClient.invalidateQueries(['messages'])
    }
  })
}
```

**3. API Request**

```http
POST /api/messages HTTP/1.1
Host: localhost:3080
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "text": "What is React?",
  "conversationId": "507f1f77bcf86cd799439011",
  "model": "gpt-4"
}
```

**4. Express Route**

File: `api/server/routes/messages.js`

```javascript
const router = require('express').Router()
const { requireJwtAuth } = require('~/server/middleware')
const MessageController = require('~/server/controllers/MessageController')

router.post('/',
  requireJwtAuth,           // 1. Verify JWT token
  validateRequest,          // 2. Validate input
  MessageController.create  // 3. Handle request
)

module.exports = router
```

**5. Middleware Chain**

File: `api/server/middleware/requireJwtAuth.js`

```javascript
const passport = require('passport')

function requireJwtAuth(req, res, next) {
  passport.authenticate('jwt', { session: false }, (err, user) => {
    if (err || !user) {
      return res.status(401).json({ error: 'Unauthorized' })
    }

    req.user = user  // Attach user to request
    next()           // Continue to next middleware/controller
  })(req, res, next)
}
```

**6. Controller**

File: `api/server/controllers/MessageController.js`

```javascript
class MessageController {
  static async create(req, res) {
    try {
      const { text, conversationId, model } = req.body
      const userId = req.user.id

      // Call service layer
      const result = await MessageService.createWithAI({
        text,
        conversationId,
        userId,
        model
      })

      // Stream response
      res.setHeader('Content-Type', 'text/event-stream')
      result.stream.on('data', chunk => {
        res.write(`data: ${JSON.stringify(chunk)}\n\n`)
      })

      result.stream.on('end', () => res.end())
    } catch (error) {
      res.status(500).json({ error: error.message })
    }
  }
}
```

**7. Service Layer**

File: `api/server/services/MessageService.js`

```javascript
class MessageService {
  static async createWithAI({ text, conversationId, userId, model }) {
    // 1. Save user message
    const userMessage = await Message.create({
      text,
      conversationId,
      userId,
      sender: 'user',
      createdAt: new Date()
    })

    // 2. Get conversation history
    const history = await Message.find({ conversationId })
      .sort({ createdAt: 1 })
      .limit(10)

    // 3. Build prompt
    const messages = [
      { role: 'system', content: 'You are a helpful assistant.' },
      ...history.map(m => ({
        role: m.sender === 'user' ? 'user' : 'assistant',
        content: m.text
      })),
      { role: 'user', content: text }
    ]

    // 4. Call OpenAI
    const stream = await openai.chat.completions.create({
      model,
      messages,
      stream: true
    })

    // 5. Collect full response
    let fullResponse = ''
    stream.on('data', chunk => {
      fullResponse += chunk.choices[0]?.delta?.content || ''
    })

    stream.on('end', async () => {
      // 6. Save AI response
      await Message.create({
        text: fullResponse,
        conversationId,
        userId,
        sender: 'assistant',
        metadata: { model }
      })
    })

    return { stream, userMessage }
  }
}
```

**8. Database Operations**

```javascript
// MongoDB operations (via Mongoose)

// Save user message
await Message.create({
  text: "What is React?",
  conversationId: ObjectId("..."),
  userId: ObjectId("..."),
  sender: "user"
})
// Result: { _id: ObjectId("..."), text: "What is React?", ... }

// Get history
await Message.find({ conversationId: ObjectId("...") })
  .sort({ createdAt: 1 })
  .limit(10)
// Result: [{ text: "...", sender: "user" }, ...]

// Save AI response
await Message.create({
  text: "React is a JavaScript library...",
  conversationId: ObjectId("..."),
  userId: ObjectId("..."),
  sender: "assistant",
  metadata: { model: "gpt-4", tokens: 150 }
})
```

**9. OpenAI API Call**

```javascript
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })

const stream = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [
    { role: 'system', content: 'You are helpful' },
    { role: 'user', content: 'What is React?' }
  ],
  stream: true
})

// Stream events:
// data: {"choices":[{"delta":{"content":"React"}}]}
// data: {"choices":[{"delta":{"content":" is"}}]}
// data: {"choices":[{"delta":{"content":" a"}}]}
// ...
```

**10. Server-Sent Events (SSE)**

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"chunk":"React"}

data: {"chunk":" is"}

data: {"chunk":" a"}

data: {"chunk":" JavaScript"}

data: [DONE]
```

**11. React Updates**

```typescript
// TanStack Query handles streaming updates
const { data, isLoading } = useQuery({
  queryKey: ['messages', conversationId],
  queryFn: async () => {
    const eventSource = new EventSource('/api/messages/stream')

    eventSource.onmessage = (event) => {
      const chunk = JSON.parse(event.data)
      // Update UI with each chunk
      setStreamingMessage(prev => prev + chunk.content)
    }
  }
})

// React renders each update
{streamingMessage && (
  <div className="message assistant streaming">
    {streamingMessage}
  </div>
)}
```

---

## Flow 2: User Logs In

**Scenario:** User enters email/password and clicks Login

### Step-by-Step Flow

```mermaid
sequenceDiagram
    participant User
    participant React
    participant Express
    participant Passport
    participant MongoDB
    participant Redis

    User->>React: Enter credentials
    React->>Express: POST /api/auth/login

    Express->>Passport: LocalStrategy.authenticate()
    Passport->>MongoDB: Find user by email
    MongoDB-->>Passport: User document

    Passport->>Passport: Compare password hash
    alt Password valid
        Passport->>Express: User authenticated
        Express->>Express: Generate JWT token
        Express->>Redis: Store session
        Redis-->>Express: Session saved
        Express-->>React: { token, user }
        React->>React: Store token
        React-->>User: Redirect to /chat
    else Password invalid
        Passport-->>Express: Authentication failed
        Express-->>React: 401 Unauthorized
        React-->>User: Show error message
    end
```

### Detailed Breakdown

**1. Login Form (Frontend)**

File: `client/src/components/Auth/Login.tsx`

```typescript
function Login() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const loginMutation = useLoginMutation()

  const handleSubmit = async (e) => {
    e.preventDefault()

    loginMutation.mutate({ email, password }, {
      onSuccess: (data) => {
        // Store token
        localStorage.setItem('token', data.token)
        // Redirect
        navigate('/chat')
      },
      onError: (error) => {
        toast.error('Invalid credentials')
      }
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">Login</button>
    </form>
  )
}
```

**2. API Request**

```http
POST /api/auth/login HTTP/1.1
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**3. Express Route**

File: `api/server/routes/auth.js`

```javascript
router.post('/login',
  loginLimiter,              // Rate limiting
  checkBan,                  // Check if user is banned
  requireLocalAuth,          // Passport Local Strategy
  loginController
)
```

**4. Passport Local Strategy**

File: `api/strategies/localStrategy.js`

```javascript
const LocalStrategy = require('passport-local').Strategy

passport.use(new LocalStrategy({
  usernameField: 'email',
  passwordField: 'password'
}, async (email, password, done) => {
  try {
    // Find user
    const user = await User.findOne({ email }).select('+password')

    if (!user) {
      return done(null, false, { message: 'Invalid credentials' })
    }

    // Compare password
    const isValid = await bcrypt.compare(password, user.password)

    if (!isValid) {
      return done(null, false, { message: 'Invalid credentials' })
    }

    // Success!
    return done(null, user)
  } catch (error) {
    return done(error)
  }
}))
```

**5. Login Controller**

File: `api/server/controllers/auth/LoginController.js`

```javascript
async function loginController(req, res) {
  try {
    const user = req.user  // Set by Passport

    // Generate JWT
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    )

    // Store session in Redis
    const sessionId = generateSessionId()
    await redis.setex(
      `session:${sessionId}`,
      7 * 24 * 60 * 60,  // 7 days
      JSON.stringify({ userId: user._id, token })
    )

    // Return token
    res.json({
      token,
      user: {
        id: user._id,
        email: user.email,
        name: user.name
      }
    })
  } catch (error) {
    res.status(500).json({ error: error.message })
  }
}
```

**6. Database Query**

```javascript
// Find user
const user = await User.findOne({ email: 'user@example.com' })
  .select('+password')  // Include password field (normally excluded)

// Result:
{
  _id: ObjectId("..."),
  email: "user@example.com",
  name: "John Doe",
  password: "$2a$10$..." // bcrypt hash
}
```

**7. Password Verification**

```javascript
const bcrypt = require('bcryptjs')

// User's input
const inputPassword = 'password123'

// Stored hash
const storedHash = '$2a$10$N9qo8uLOickgx2ZMRZoMye...'

// Compare
const isValid = await bcrypt.compare(inputPassword, storedHash)
// Result: true (passwords match)
```

**8. JWT Generation**

```javascript
const jwt = require('jsonwebtoken')

const token = jwt.sign(
  {
    userId: '507f1f77bcf86cd799439011',
    email: 'user@example.com',
    iat: Math.floor(Date.now() / 1000)
  },
  process.env.JWT_SECRET,  // Secret key from .env
  { expiresIn: '7d' }
)

// Result: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiI1MDdmMWY3N2JjZjg2Y2Q3OTk0MzkwMTEiLCJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJpYXQiOjE3MDA1Nzc2MDB9...
```

**9. Session Storage (Redis)**

```javascript
const redis = require('ioredis')()

await redis.setex(
  'session:abc123',
  604800,  // 7 days in seconds
  JSON.stringify({
    userId: '507f1f77bcf86cd799439011',
    token: 'eyJhbGciOiJIUzI1...',
    createdAt: new Date()
  })
)

// Stored in Redis:
// Key: session:abc123
// Value: {"userId":"507f...","token":"eyJ...","createdAt":"2025-11-19T..."}
// TTL: 604800 seconds
```

**10. Frontend Token Storage**

```typescript
// Store in localStorage
localStorage.setItem('token', response.token)

// Future requests include token
const apiClient = axios.create({
  baseURL: '/api',
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('token')}`
  }
})
```

---

## Flow 3: Loading Conversations

**Scenario:** User opens the app and sees conversation list

### Flow Diagram

```
User lands on /chat
  ↓
React useEffect runs
  ↓
TanStack Query: useConversationsQuery()
  ↓
GET /api/conversations (with JWT)
  ↓
Express → requireJwtAuth → Controller
  ↓
ConversationService.getByUserId(userId)
  ↓
MongoDB: Conversation.find({ userId })
  ↓
Return conversations (sorted by updatedAt)
  ↓
TanStack Query caches data
  ↓
React renders conversation list
```

### Code Flow

**1. React Component**

```typescript
function ConversationList() {
  const { data: conversations, isLoading } = useConversationsQuery()

  if (isLoading) return <Spinner />

  return (
    <div className="conversation-list">
      {conversations.map(conv => (
        <ConversationItem key={conv.id} conversation={conv} />
      ))}
    </div>
  )
}
```

**2. TanStack Query**

```typescript
export const useConversationsQuery = () => {
  return useQuery({
    queryKey: ['conversations'],
    queryFn: () => apiClient.get('/api/conversations'),
    staleTime: 5 * 60 * 1000,  // Consider fresh for 5 minutes
    refetchOnWindowFocus: true  // Refetch when user returns to tab
  })
}
```

**3. API Route → Controller → Service**

```javascript
// Route
router.get('/', requireJwtAuth, ConversationController.getAll)

// Controller
class ConversationController {
  static async getAll(req, res) {
    const conversations = await ConversationService.getByUserId(req.user.id)
    res.json(conversations)
  }
}

// Service
class ConversationService {
  static async getByUserId(userId) {
    return await Conversation.find({ userId })
      .sort({ updatedAt: -1 })
      .limit(50)
      .lean()
  }
}
```

**4. Database Query**

```javascript
await Conversation.find({ userId: ObjectId("...") })
  .sort({ updatedAt: -1 })  // Most recent first
  .limit(50)
  .lean()  // Return plain objects (faster)

// Result:
[
  {
    _id: ObjectId("..."),
    conversationId: "abc123",
    title: "Chat about React",
    userId: ObjectId("..."),
    updatedAt: ISODate("2025-11-19T15:30:00Z"),
    createdAt: ISODate("2025-11-18T10:00:00Z")
  },
  // ... more conversations
]
```

---

## Flow 4: File Upload

**Scenario:** User uploads an image to chat

### Flow Summary

```
User selects file
  ↓
React FileReader reads file
  ↓
POST /api/files (multipart/form-data)
  ↓
Multer middleware parses file
  ↓
FileController receives file buffer
  ↓
Sharp processes image (resize, optimize)
  ↓
Save to S3 or MongoDB GridFS
  ↓
Create File document in MongoDB
  ↓
Return file URL
  ↓
React displays uploaded file
```

### Code Snippets

**1. Frontend File Upload**

```typescript
function FileUpload() {
  const uploadMutation = useUploadFileMutation()

  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0]
    if (!file) return

    const formData = new FormData()
    formData.append('file', file)

    uploadMutation.mutate(formData)
  }

  return <input type="file" onChange={handleFileChange} accept="image/*" />
}
```

**2. Multer Middleware**

```javascript
const multer = require('multer')

const upload = multer({
  storage: multer.memoryStorage(),  // Store in memory (not disk)
  limits: { fileSize: 10 * 1024 * 1024 },  // 10MB max
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true)
    } else {
      cb(new Error('Only images allowed'))
    }
  }
})

router.post('/files', upload.single('file'), FileController.upload)
```

**3. File Processing with Sharp**

```javascript
const sharp = require('sharp')

class FileController {
  static async upload(req, res) {
    const file = req.file  // From multer

    // Resize and optimize image
    const processedBuffer = await sharp(file.buffer)
      .resize(1024, 1024, { fit: 'inside' })
      .jpeg({ quality: 80 })
      .toBuffer()

    // Upload to S3 or save to MongoDB
    const url = await uploadToS3(processedBuffer, file.originalname)

    // Save file record
    const fileDoc = await File.create({
      filename: file.originalname,
      mimetype: file.mimetype,
      size: processedBuffer.length,
      url,
      userId: req.user.id
    })

    res.json({ file: fileDoc })
  }
}
```

---

## Flow 5: Creating an Agent

**Scenario:** User creates a custom AI agent

### Flow Summary

```
User fills agent form
  ↓
POST /api/agents
  ↓
Validate agent data (Zod schema)
  ↓
AgentService.create()
  ↓
Save Agent to MongoDB
  ↓
Create default permissions
  ↓
Return agent data
  ↓
Invalidate agents cache
  ↓
React shows new agent
```

---

## Key Patterns

### Pattern 1: Optimistic Updates

```typescript
const deleteMutation = useMutation({
  mutationFn: (id) => apiClient.delete(`/conversations/${id}`),
  onMutate: async (id) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries(['conversations'])

    // Snapshot previous value
    const previous = queryClient.getQueryData(['conversations'])

    // Optimistically update UI
    queryClient.setQueryData(['conversations'], (old) =>
      old.filter(conv => conv.id !== id)
    )

    return { previous }
  },
  onError: (err, id, context) => {
    // Rollback on error
    queryClient.setQueryData(['conversations'], context.previous)
  }
})
```

### Pattern 2: Parallel Requests

```typescript
// Load multiple resources at once
const [conversations, messages, user] = await Promise.all([
  apiClient.get('/conversations'),
  apiClient.get('/messages'),
  apiClient.get('/user')
])
```

### Pattern 3: Request Deduplication

TanStack Query automatically deduplicates:

```typescript
// Multiple components request same data
// Only ONE network request made!

function ComponentA() {
  const { data } = useConversationsQuery()  // Request #1
}

function ComponentB() {
  const { data } = useConversationsQuery()  // Same request, cached!
}
```

### Pattern 4: Cache Invalidation

```typescript
// After creating a message, refresh conversations
const sendMutation = useMutation({
  mutationFn: sendMessage,
  onSuccess: () => {
    queryClient.invalidateQueries(['conversations'])
    queryClient.invalidateQueries(['messages'])
  }
})
```

---

## Summary

Data flows through LibreChat following these patterns:

**1. User Action → React**
   - Event handler triggered
   - State updated

**2. React → TanStack Query**
   - useMutation or useQuery called
   - Query key for caching

**3. TanStack Query → API**
   - HTTP request (GET/POST/PUT/DELETE)
   - JWT token in headers

**4. API → Middleware → Controller**
   - Auth checked
   - Input validated
   - Request handled

**5. Controller → Service**
   - Business logic executed
   - External APIs called (OpenAI, etc.)

**6. Service → Database**
   - Data persisted
   - Queries executed

**7. Database → Service → Controller**
   - Data returned
   - Response formatted

**8. Controller → API → TanStack Query**
   - JSON response
   - Cache updated

**9. TanStack Query → React**
   - Component re-renders
   - UI updates

**10. React → User**
   - New UI displayed

---

**Next Steps:**
- [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React patterns deep dive
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Express patterns deep dive
- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Frontend ↔ Backend communication

---

**Back to:** [Learning Path Home](./README.md)

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
