# 🗺️ Code Tours: Guided Walkthroughs

**Documented:** November 19, 2025
**Target:** Developers exploring LibreChat codebase
**Time Estimate:** 1-2 hours per tour
**Difficulty:** 🟡 Intermediate

---

## Follow the Code

These guided tours trace complete user flows through the codebase, showing you exactly how features work from UI click to database and back.

---

## Table of Contents

- [Tour 1: User Sends Message to AI](#tour-1-user-sends-message-to-ai)
- [Tour 2: User Login Flow](#tour-2-user-login-flow)
- [Tour 3: Loading Conversations](#tour-3-loading-conversations)
- [Tour 4: Creating a New Conversation](#tour-4-creating-a-new-conversation)
- [Tour 5: File Upload Flow](#tour-5-file-upload-flow)

---

## Tour 1: User Sends Message to AI

**Flow:** User types message → Sends → AI responds → Message saved

### Step 1: User Types in ChatInput

**File:** `client/src/components/Chat/ChatInput.tsx`

```typescript
export function ChatInput({ conversationId }: ChatInputProps) {
  const [text, setText] = useState('');
  const { streamMessage, isStreaming } = useStreamMessage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    // User clicked "Send"
    await streamMessage({
      text,           // "Hello, AI!"
      conversationId, // "507f1f77bcf86cd799439011"
      model: 'gpt-4'
    });

    setText(''); // Clear input
  };

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

**What happened:**
- User typed "Hello, AI!" in textarea
- Clicked submit button
- `handleSubmit` called `streamMessage` hook

### Step 2: Custom Hook Makes API Call

**File:** `client/src/hooks/useStreamMessage.ts`

```typescript
export function useStreamMessage() {
  const streamMessage = async ({ text, conversationId, model }) => {
    const token = localStorage.getItem('authToken');

    // Make fetch request to streaming endpoint
    const response = await fetch('http://localhost:3080/api/messages/stream', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}` // JWT token for auth
      },
      body: JSON.stringify({
        text: 'Hello, AI!',
        conversationId: '507f1f77bcf86cd799439011',
        model: 'gpt-4'
      })
    });

    // Read streaming response (Server-Sent Events)
    const reader = response.body.getReader();
    // ... streaming logic
  };
}
```

**What happened:**
- Retrieved auth token from localStorage
- Made POST request to `/api/messages/stream`
- Sent message data as JSON

### Step 3: Express Route Receives Request

**File:** `api/server/routes/messages.js`

```javascript
const express = require('express');
const router = express.Router();
const requireAuth = require('../middleware/requireAuth');
const messagesController = require('../controllers/messagesController');

// POST /api/messages/stream
router.post('/stream',
  requireAuth,              // Middleware 1: Verify JWT token
  messagesController.streamMessage  // Controller
);
```

**What happened:**
- Express matched route `/api/messages/stream`
- `requireAuth` middleware ran first (verify token)
- Then `streamMessage` controller executed

### Step 4: Auth Middleware Verifies Token

**File:** `api/server/middleware/requireAuth.js`

```javascript
const { verifyToken } = require('../utils/jwt');

function requireAuth(req, res, next) {
  // Get token from header
  const authHeader = req.headers.authorization; // "Bearer eyJhbGc..."
  const token = authHeader.substring(7); // Remove "Bearer "

  // Verify JWT token
  const decoded = verifyToken(token);
  // decoded = { id: '507f191e810c19729de860ea', email: 'user@example.com' }

  // Attach user to request
  req.user = decoded;

  // Continue to next middleware/controller
  next();
}
```

**What happened:**
- Extracted JWT token from Authorization header
- Verified token signature
- Attached user data to `req.user`
- Called `next()` to continue to controller

### Step 5: Controller Handles Request

**File:** `api/server/controllers/messagesController.js`

```javascript
async function streamMessage(req, res) {
  const { text, conversationId, model } = req.body;
  const userId = req.user.id; // From auth middleware

  // Set SSE headers for streaming
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    // Call service to process message
    await messageService.streamMessage({
      text,
      conversationId,
      model,
      userId,
      onChunk: (chunk) => {
        // Stream each chunk to client
        res.write(`data: ${JSON.stringify({ type: 'chunk', content: chunk })}\n\n`);
      }
    });

    res.end();

  } catch (error) {
    res.write(`data: ${JSON.stringify({ type: 'error', error: error.message })}\n\n`);
    res.end();
  }
}
```

**What happened:**
- Extracted request data (text, conversationId, model)
- Got userId from `req.user` (set by middleware)
- Set up SSE (Server-Sent Events) headers
- Called service layer to process message

### Step 6: Service Saves User Message

**File:** `api/server/services/messageService.js`

```javascript
const Message = require('../../models/Message');
const openai = require('../config/openai');

class MessageService {
  async streamMessage({ text, conversationId, userId, model, onChunk }) {
    // 1. Save user message to database
    const userMessage = await Message.create({
      conversationId: '507f1f77bcf86cd799439011',
      userId: '507f191e810c19729de860ea',
      role: 'user',
      text: 'Hello, AI!',
      model: 'gpt-4'
    });

    // 2. Call OpenAI with streaming
    const stream = await openai.chat.completions.create({
      model: 'gpt-4',
      messages: [{ role: 'user', content: 'Hello, AI!' }],
      stream: true // Enable streaming
    });

    // 3. Stream AI response
    let fullResponse = '';

    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content || '';

      if (content) {
        fullResponse += content;
        onChunk(content); // Send chunk to controller → client
      }
    }

    // 4. Save AI message
    const aiMessage = await Message.create({
      conversationId: '507f1f77bcf86cd799439011',
      userId: '507f191e810c19729de860ea',
      role: 'assistant',
      text: fullResponse,
      model: 'gpt-4'
    });

    return aiMessage;
  }
}
```

**What happened:**
- Saved user's message to MongoDB via Mongoose
- Called OpenAI API with streaming enabled
- For each chunk received from OpenAI:
  - Appended to full response
  - Sent chunk to client via `onChunk` callback
- Saved complete AI response to database

### Step 7: Mongoose Saves to MongoDB

**File:** `api/models/Message.js`

```javascript
const messageSchema = new mongoose.Schema({
  conversationId: { type: ObjectId, ref: 'Conversation', required: true },
  userId: { type: ObjectId, ref: 'User', required: true },
  role: { type: String, enum: ['user', 'assistant', 'system'] },
  text: { type: String, required: true },
  model: String
}, { timestamps: true });

const Message = mongoose.model('Message', messageSchema);
```

**MongoDB Operation:**
```javascript
db.messages.insertOne({
  _id: ObjectId("..."),
  conversationId: ObjectId("507f1f77bcf86cd799439011"),
  userId: ObjectId("507f191e810c19729de860ea"),
  role: "user",
  text: "Hello, AI!",
  model: "gpt-4",
  createdAt: ISODate("2025-11-19T10:00:00Z"),
  updatedAt: ISODate("2025-11-19T10:00:00Z")
})
```

**What happened:**
- Mongoose validated data against schema
- Converted data to MongoDB format
- Inserted document into `messages` collection
- Auto-generated `_id`, `createdAt`, `updatedAt`

### Step 8: Response Streams Back to Client

**Controller sends SSE chunks:**

```javascript
// First chunk
res.write('data: {"type":"chunk","content":"Hello"}\n\n');

// Second chunk
res.write('data: {"type":"chunk","content":"!"}\n\n');

// Third chunk
res.write('data: {"type":"chunk","content":" How"}\n\n');

// ... more chunks

// Complete
res.write('data: {"type":"complete","message":{...}}\n\n');
res.end();
```

**What happened:**
- Each chunk sent as SSE formatted data
- Client receives chunks in real-time
- Final "complete" event signals end

### Step 9: Frontend Displays Streaming Text

**File:** `client/src/hooks/useStreamMessage.ts`

```typescript
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value);
  const lines = chunk.split('\n\n');

  for (const line of lines) {
    if (line.startsWith('data: ')) {
      const data = JSON.parse(line.substring(6));

      if (data.type === 'chunk') {
        setStreamedText(prev => prev + data.content);
        // UI updates: "Hello" → "Hello!" → "Hello! How" → ...
      }
    }
  }
}
```

**What happened:**
- Read streamed response byte by byte
- Parsed SSE format
- Updated UI state with each chunk
- User sees AI response appear word-by-word

### Step 10: TanStack Query Invalidates Cache

```typescript
onComplete: () => {
  queryClient.invalidateQueries({
    queryKey: ['messages', conversationId]
  });
}
```

**What happened:**
- Streaming completed
- Invalidated messages cache
- TanStack Query refetched messages
- Got fresh data from database (includes new messages)
- Component re-rendered with updated list

---

## Tour 2: User Login Flow

**Flow:** User enters credentials → Submit → Token returned → Redirect to app

### Step 1: Login Form Component

**File:** `client/src/components/Auth/Login.tsx`

```typescript
function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const loginMutation = useLogin();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    await loginMutation.mutateAsync({
      email: 'user@example.com',
      password: 'mypassword123'
    });
  };
}
```

### Step 2: Login Hook

**File:** `client/src/hooks/useAuth.ts`

```typescript
export function useLogin() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ email, password }) => {
      const { data } = await apiClient.post('/api/auth/login', {
        email,
        password
      });
      return data; // { user: {...}, token: "..." }
    },

    onSuccess: (data) => {
      // Save token
      localStorage.setItem('authToken', data.token);

      // Cache user
      queryClient.setQueryData(['user'], data.user);
    }
  });
}
```

### Step 3: Backend Route

**File:** `api/server/routes/auth.js`

```javascript
router.post('/login',
  rateLimiter,  // Limit login attempts
  authController.login
);
```

### Step 4: Auth Controller

**File:** `api/server/controllers/authController.js`

```javascript
async login(req, res, next) {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ email }).select('+password');

    if (!user) {
      throw new UnauthorizedError('Invalid credentials');
    }

    // Compare password
    const isMatch = await bcrypt.compare(password, user.password);

    if (!isMatch) {
      throw new UnauthorizedError('Invalid credentials');
    }

    // Generate JWT
    const token = jwt.sign(
      { id: user._id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );

    res.json({
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      },
      token
    });

  } catch (error) {
    next(error);
  }
}
```

**What happened:**
- Found user by email in database
- Compared submitted password with hashed password
- Generated JWT token with user ID and email
- Sent user data + token to client

### Step 5: User Model

**File:** `api/models/User.js`

```javascript
const userSchema = new Schema({
  email: { type: String, unique: true, required: true },
  password: { type: String, required: true, select: false }, // Hidden by default
  name: String
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();

  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});
```

**MongoDB Query:**
```javascript
db.users.findOne({ email: 'user@example.com' })

// Returns:
{
  _id: ObjectId("507f191e810c19729de860ea"),
  email: "user@example.com",
  password: "$2a$10$...", // Bcrypt hash
  name: "John Doe"
}
```

### Step 6: Password Verification

```javascript
const bcrypt = require('bcryptjs');

// Submitted password: "mypassword123"
// Hashed in DB: "$2a$10$N9qo8uLOickgx2ZMRZoMye..."

const isMatch = await bcrypt.compare(
  'mypassword123',                      // Plain text
  '$2a$10$N9qo8uLOickgx2ZMRZoMye...'   // Hash
);
// Returns: true
```

### Step 7: JWT Generation

```javascript
const jwt = require('jsonwebtoken');

const token = jwt.sign(
  {
    id: '507f191e810c19729de860ea',
    email: 'user@example.com'
  },
  'your-secret-key',
  { expiresIn: '7d' }
);

// Returns: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Step 8: Frontend Saves Token

```typescript
onSuccess: (data) => {
  // Save to localStorage
  localStorage.setItem('authToken', data.token);

  // Cache user in TanStack Query
  queryClient.setQueryData(['user'], data.user);

  // Navigate to home
  navigate('/');
}
```

---

## Tour 3: Loading Conversations

**Flow:** App loads → Fetch conversations → Display list

### Step 1: Component Mounts

**File:** `client/src/components/Sidebar/ConversationList.tsx`

```typescript
export function ConversationList() {
  // Hook automatically runs on mount
  const { data: conversations, isLoading } = useConversations();

  if (isLoading) return <Spinner />;

  return (
    <div>
      {conversations?.map(conv => (
        <ConversationCard key={conv.id} conversation={conv} />
      ))}
    </div>
  );
}
```

### Step 2: TanStack Query Hook

**File:** `client/src/hooks/useConversations.ts`

```typescript
export function useConversations() {
  return useQuery({
    queryKey: ['conversations'],

    queryFn: async () => {
      const { data } = await apiClient.get('/api/conversations');
      return data.conversations;
    },

    staleTime: 1000 * 60 * 5 // Fresh for 5 minutes
  });
}
```

**What happens:**
1. TanStack Query checks cache for key `['conversations']`
2. If not in cache or stale, runs `queryFn`
3. Makes GET request to `/api/conversations`
4. Caches result
5. Returns data to component

### Step 3: Backend Endpoint

**File:** `api/server/routes/conversations.js`

```javascript
router.get('/',
  requireAuth,
  conversationsController.getAll
);
```

**File:** `api/server/controllers/conversationsController.js`

```javascript
async getAll(req, res, next) {
  try {
    const userId = req.user.id;

    const conversations = await conversationService.findByUser(userId);

    res.json({ conversations });

  } catch (error) {
    next(error);
  }
}
```

### Step 4: Service Layer

**File:** `api/server/services/conversationService.js`

```javascript
async findByUser(userId) {
  const conversations = await Conversation
    .find({ userId, isArchived: { $ne: true } })
    .sort({ updatedAt: -1 })
    .limit(50)
    .lean();

  return conversations;
}
```

### Step 5: MongoDB Query

```javascript
db.conversations.find({
  userId: ObjectId("507f191e810c19729de860ea"),
  isArchived: { $ne: true }
})
.sort({ updatedAt: -1 })
.limit(50)

// Returns array of conversations
[
  {
    _id: ObjectId("..."),
    userId: ObjectId("507f191e810c19729de860ea"),
    title: "Chat about React",
    model: "gpt-4",
    createdAt: ISODate("2025-11-19T10:00:00Z"),
    updatedAt: ISODate("2025-11-19T15:30:00Z")
  },
  // ... more conversations
]
```

### Step 6: Response to Frontend

```json
{
  "conversations": [
    {
      "id": "507f1f77bcf86cd799439011",
      "title": "Chat about React",
      "model": "gpt-4",
      "createdAt": "2025-11-19T10:00:00Z",
      "updatedAt": "2025-11-19T15:30:00Z"
    }
  ]
}
```

### Step 7: TanStack Query Caches Result

```typescript
// Cache structure
cache = {
  ['conversations']: {
    data: [...conversations],
    dataUpdatedAt: 1700395200000,
    status: 'success'
  }
}
```

### Step 8: Component Re-renders

```typescript
// Before: isLoading = true, data = undefined
<Spinner />

// After: isLoading = false, data = [...]
<div>
  <ConversationCard conversation={conversations[0]} />
  <ConversationCard conversation={conversations[1]} />
  // ...
</div>
```

---

## Summary

**What You Learned:**

1. **Complete request flow:** UI → Hook → API → Controller → Service → Database → Response
2. **Middleware chain:** How requests pass through multiple middleware
3. **Authentication:** How JWT tokens are generated and verified
4. **Streaming:** How Server-Sent Events deliver real-time updates
5. **Caching:** How TanStack Query caches and manages server state
6. **Database operations:** How Mongoose queries MongoDB

**Next Steps:**

- Read actual source files in LibreChat
- Set breakpoints and trace flows yourself
- Build your own feature following these patterns

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
