# 🏗️ Backend Architecture Deep Dive

**Documented:** November 19, 2025
**Target:** React developers learning backend concepts
**Time Estimate:** 6-8 hours to fully understand
**Difficulty:** 🟡 Intermediate

---

## Welcome to Backend Development!

If you're coming from React, you already understand more than you think! This guide will bridge your frontend knowledge to backend concepts using LibreChat's Express.js architecture.

---

## Table of Contents

- [Mental Model: React → Express](#mental-model-react--express)
- [The Express Server](#the-express-server)
- [Architecture Pattern: MVC-ish](#architecture-pattern-mvc-ish)
- [The Middleware Chain](#the-middleware-chain)
- [Request/Response Lifecycle](#requestresponse-lifecycle)
- [Routes: The Entry Point](#routes-the-entry-point)
- [Controllers: The Event Handlers](#controllers-the-event-handlers)
- [Services: The Business Logic](#services-the-business-logic)
- [Models: The Data Layer](#models-the-data-layer)
- [Error Handling](#error-handling)
- [Authentication & Authorization](#authentication--authorization)
- [Logging & Monitoring](#logging--monitoring)
- [API Design Patterns](#api-design-patterns)
- [Performance Considerations](#performance-considerations)

---

## Mental Model: React → Express

**If you know React, you already understand these backend concepts:**

| React Concept | Backend Equivalent | Explanation |
|--------------|-------------------|-------------|
| **Component** | Route Handler | Receives input (props/request), returns output (JSX/JSON) |
| **Props** | Request params/body | Data passed from outside |
| **useState** | Database | Persistent state storage |
| **useEffect** | Middleware | Code that runs before/after main logic |
| **Custom Hook** | Service Function | Reusable business logic |
| **Context Provider** | Middleware | Data/functionality available to all children |
| **Higher-Order Component** | Middleware Wrapper | Wraps handler with additional behavior |
| **Event Handler** | Controller Method | Responds to user actions |
| **API Call** | Database Query | Fetch external data |
| **Error Boundary** | Error Handling Middleware | Catches and handles errors |

**Key Difference:**
- **React:** User interaction → Component re-renders → UI updates
- **Express:** HTTP request → Route → Controller → Service → Database → Response

---

## The Express Server

### Server Entry Point

**File:** `api/server/index.js` (conceptual)

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const routes = require('./routes');

// Create Express app (like creating a React app)
const app = express();

// ===========================
// Global Middleware (like App-level Context)
// ===========================

// Parse JSON request bodies (like props coming in)
app.use(express.json());

// Enable CORS (allow frontend to communicate)
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true
}));

// Security headers
app.use(helmet());

// Logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // Pass to next middleware
});

// ===========================
// Routes (like React Router)
// ===========================

app.use('/api/auth', routes.auth);
app.use('/api/messages', routes.messages);
app.use('/api/conversations', routes.conversations);
app.use('/api/agents', routes.agents);

// ===========================
// Error Handling (like Error Boundary)
// ===========================

app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      status: err.status
    }
  });
});

// ===========================
// Start Server (like ReactDOM.render)
// ===========================

const PORT = process.env.PORT || 3080;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**React Analogy:**
```javascript
// React App Structure
function App() {
  return (
    <ErrorBoundary>           {/* Error handling middleware */}
      <AuthProvider>          {/* Authentication middleware */}
        <Router>              {/* Route matching */}
          <Routes>
            <Route path="/messages" element={<Messages />} />
            <Route path="/auth" element={<Auth />} />
          </Routes>
        </Router>
      </AuthProvider>
    </ErrorBoundary>
  );
}
```

---

## Architecture Pattern: MVC-ish

LibreChat follows a **modified MVC** (Model-View-Controller) pattern:

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT (React)                    │
│                    The "View"                       │
└───────────────────┬─────────────────────────────────┘
                    │ HTTP Request
                    ▼
┌─────────────────────────────────────────────────────┐
│                    ROUTES                           │
│   Define URL patterns and map to controllers       │
│   File: api/server/routes/*.js                     │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│                  CONTROLLERS                        │
│   Handle HTTP request/response logic                │
│   File: api/server/controllers/*.js                │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│                   SERVICES                          │
│   Business logic, orchestration                     │
│   File: api/server/services/*.js                   │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│                    MODELS                           │
│   Database schemas and queries                      │
│   File: api/models/*.js                            │
└───────────────────┬─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│                  DATABASE                           │
│              MongoDB Collections                    │
└─────────────────────────────────────────────────────┘
```

### Why "MVC-ish" and not pure MVC?

**Traditional MVC:**
- Model: Data + business logic
- View: UI rendering (templates)
- Controller: Request handling

**LibreChat's approach:**
- **Model:** Data schemas only (Mongoose models)
- **View:** React (separate frontend)
- **Controller:** HTTP request/response handling
- **Service:** Business logic (new layer!)

**Why the Service layer?**
- Keeps controllers thin (only HTTP concerns)
- Reusable business logic
- Easier to test
- Better separation of concerns

---

## The Middleware Chain

**What is middleware?**

Middleware is code that runs **between** receiving a request and sending a response.

**React Analogy:**
```javascript
// React: Higher-Order Component (HOC)
function withAuth(Component) {
  return function ProtectedComponent(props) {
    const user = useAuth();
    if (!user) return <Redirect to="/login" />;
    return <Component {...props} user={user} />;
  };
}

// Express: Middleware
function requireAuth(req, res, next) {
  const user = getAuthUser(req);
  if (!user) return res.status(401).json({ error: 'Unauthorized' });
  req.user = user; // Attach user to request
  next(); // Continue to next middleware/handler
}
```

### Middleware Execution Flow

```javascript
// Request comes in
app.post('/api/messages',
  // Middleware 1: Parse JSON body
  express.json(),

  // Middleware 2: Authenticate user
  requireAuth,

  // Middleware 3: Validate request
  validateMessageInput,

  // Middleware 4: Rate limiting
  rateLimiter,

  // Final handler: Controller
  messagesController.createMessage
);
// Response goes out
```

**Execution order:**
```
Request → express.json() → requireAuth → validateMessageInput
        → rateLimiter → messagesController.createMessage → Response
```

If any middleware calls `res.send()` or throws an error, the chain stops.

### Built-in Middleware Examples

**1. Body Parsers**
```javascript
// Parse JSON request bodies
app.use(express.json());

// Parse URL-encoded form data
app.use(express.urlencoded({ extended: true }));
```

**React Analogy:** Like automatically parsing props from strings to objects.

**2. CORS (Cross-Origin Resource Sharing)**
```javascript
const cors = require('cors');

app.use(cors({
  origin: 'http://localhost:3000', // Allow React dev server
  credentials: true // Allow cookies
}));
```

**Why needed?** Browser security prevents `http://localhost:3000` (React) from calling `http://localhost:3080` (API) without CORS.

**3. Static File Serving**
```javascript
// Serve uploaded files
app.use('/files', express.static('uploads'));

// Now files are accessible at:
// http://localhost:3080/files/avatar.png
```

### Custom Middleware Examples

**File:** `api/server/middleware/requireAuth.js`

```javascript
const { verifyToken } = require('../utils/jwt');

/**
 * Middleware: Require authentication
 * Verifies JWT token and attaches user to request
 */
function requireAuth(req, res, next) {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({
        error: 'No token provided'
      });
    }

    const token = authHeader.substring(7); // Remove 'Bearer '

    // Verify JWT token
    const decoded = verifyToken(token);

    // Attach user to request (available in all subsequent handlers)
    req.user = decoded;

    // Continue to next middleware/handler
    next();

  } catch (error) {
    return res.status(401).json({
      error: 'Invalid token'
    });
  }
}

module.exports = requireAuth;
```

**Usage:**
```javascript
const requireAuth = require('./middleware/requireAuth');

// Protected route - only authenticated users
app.get('/api/user/profile', requireAuth, (req, res) => {
  // req.user is available here (from middleware)
  res.json({ user: req.user });
});

// Public route - no middleware
app.get('/api/public/info', (req, res) => {
  res.json({ info: 'Public data' });
});
```

**File:** `api/server/middleware/validateRequest.js`

```javascript
const { z } = require('zod');

/**
 * Middleware factory: Validate request body against Zod schema
 */
function validateRequest(schema) {
  return (req, res, next) => {
    try {
      // Validate and parse request body
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      // Zod validation error
      return res.status(400).json({
        error: 'Validation failed',
        details: error.errors
      });
    }
  };
}

// Usage with Zod schema
const createMessageSchema = z.object({
  text: z.string().min(1).max(10000),
  conversationId: z.string(),
  model: z.enum(['gpt-4', 'claude-3', 'gemini-pro'])
});

app.post('/api/messages',
  requireAuth,
  validateRequest(createMessageSchema), // Validates body
  messagesController.create
);
```

**File:** `api/server/middleware/rateLimiter.js`

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('../config/redis');

/**
 * Rate limiting middleware
 * Limits API requests per IP address
 */
const limiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rate-limit:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false
});

module.exports = limiter;
```

**Usage:**
```javascript
// Apply to all routes
app.use('/api/', limiter);

// Or specific routes
app.post('/api/auth/login', limiter, authController.login);
```

### Error Handling Middleware

**Special middleware with 4 parameters:**

```javascript
/**
 * Global error handler
 * Must have 4 parameters: (err, req, res, next)
 */
function errorHandler(err, req, res, next) {
  // Log error
  console.error('Error:', err);

  // Don't leak error details in production
  const isDevelopment = process.env.NODE_ENV === 'development';

  res.status(err.status || 500).json({
    error: {
      message: err.message,
      ...(isDevelopment && { stack: err.stack })
    }
  });
}

// Must be LAST middleware
app.use(errorHandler);
```

**React Analogy:**
```javascript
// React Error Boundary
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    console.error('Error:', error);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

---

## Request/Response Lifecycle

### Complete Flow Example

**Scenario:** User sends a message to AI

```javascript
// 1. Client makes request
fetch('http://localhost:3080/api/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGciOiJIUzI1...'
  },
  body: JSON.stringify({
    text: 'Hello, AI!',
    conversationId: '123',
    model: 'gpt-4'
  })
});

// ===========================
// 2. Request hits Express server
// ===========================

// Middleware chain executes:

// Step 1: Parse JSON body
app.use(express.json());
// → req.body = { text: 'Hello, AI!', ... }

// Step 2: Authenticate
requireAuth(req, res, next);
// → Verifies JWT token
// → req.user = { id: 'user123', email: 'user@example.com' }

// Step 3: Validate input
validateRequest(createMessageSchema)(req, res, next);
// → Validates req.body against schema
// → If invalid, returns 400 error and stops

// Step 4: Rate limit check
rateLimiter(req, res, next);
// → Checks Redis for request count
// → If exceeded, returns 429 error and stops

// ===========================
// 3. Controller handles request
// ===========================

async function createMessage(req, res, next) {
  try {
    const { text, conversationId, model } = req.body;
    const userId = req.user.id;

    // Call service layer
    const message = await messageService.createMessage({
      text,
      conversationId,
      model,
      userId
    });

    // Send response
    res.status(201).json({ message });

  } catch (error) {
    // Pass error to error handler
    next(error);
  }
}

// ===========================
// 4. Service executes business logic
// ===========================

async function createMessage({ text, conversationId, model, userId }) {
  // Verify user owns conversation
  const conversation = await Conversation.findById(conversationId);
  if (conversation.userId !== userId) {
    throw new ForbiddenError('Not your conversation');
  }

  // Save user message to database
  const userMessage = await Message.create({
    conversationId,
    role: 'user',
    text,
    userId
  });

  // Call AI provider
  const aiResponse = await openai.chat.completions.create({
    model,
    messages: [{ role: 'user', content: text }]
  });

  // Save AI response to database
  const aiMessage = await Message.create({
    conversationId,
    role: 'assistant',
    text: aiResponse.choices[0].message.content,
    userId
  });

  return aiMessage;
}

// ===========================
// 5. Response sent to client
// ===========================

// Response object:
{
  "message": {
    "id": "msg456",
    "conversationId": "123",
    "role": "assistant",
    "text": "Hello! How can I help you?",
    "createdAt": "2025-11-19T10:00:00Z"
  }
}
```

### Request Object (`req`)

**Key properties:**

```javascript
function handler(req, res, next) {
  // ===========================
  // Request Information
  // ===========================

  req.method      // 'GET', 'POST', 'PUT', 'DELETE', etc.
  req.path        // '/api/messages'
  req.url         // '/api/messages?page=1'
  req.params      // URL parameters: /api/messages/:id → { id: '123' }
  req.query       // Query string: ?page=1&limit=10 → { page: '1', limit: '10' }
  req.body        // Request body (requires express.json() middleware)
  req.headers     // HTTP headers object

  // ===========================
  // Common Headers
  // ===========================

  req.headers['content-type']     // 'application/json'
  req.headers['authorization']    // 'Bearer token...'
  req.headers['user-agent']       // Browser/client info

  // ===========================
  // Custom Properties (added by middleware)
  // ===========================

  req.user        // Added by requireAuth middleware
  req.session     // Added by session middleware
  req.file        // Added by file upload middleware
}
```

**React Analogy:**
- `req.params` → URL params in React Router: `useParams()`
- `req.query` → Query params: `useSearchParams()`
- `req.body` → Form data: `formData`
- `req.user` → User context: `useAuth()`

### Response Object (`res`)

**Key methods:**

```javascript
function handler(req, res) {
  // ===========================
  // Send JSON Response
  // ===========================

  res.json({ message: 'Success' });
  // Sets Content-Type: application/json
  // Serializes object to JSON string

  // ===========================
  // Send Plain Text
  // ===========================

  res.send('Hello, world!');

  // ===========================
  // Set Status Code
  // ===========================

  res.status(201).json({ created: true });
  res.status(400).json({ error: 'Bad request' });
  res.status(404).json({ error: 'Not found' });
  res.status(500).json({ error: 'Server error' });

  // ===========================
  // Set Headers
  // ===========================

  res.set('Content-Type', 'application/json');
  res.set('X-Custom-Header', 'value');

  // ===========================
  // Redirect
  // ===========================

  res.redirect('/login');
  res.redirect(301, 'https://example.com'); // Permanent redirect

  // ===========================
  // Set Cookie
  // ===========================

  res.cookie('sessionId', '123', {
    httpOnly: true,
    secure: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  });

  // ===========================
  // Send File
  // ===========================

  res.sendFile('/path/to/file.pdf');
  res.download('/path/to/file.pdf', 'filename.pdf');

  // ===========================
  // Streaming Response (for AI)
  // ===========================

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Send chunks of data
  res.write('data: chunk1\n\n');
  res.write('data: chunk2\n\n');
  res.end();
}
```

**Common Status Codes:**

```javascript
// Success
200  // OK - Request succeeded
201  // Created - Resource created successfully
204  // No Content - Success but no response body

// Client Errors
400  // Bad Request - Invalid input
401  // Unauthorized - Authentication required
403  // Forbidden - Authenticated but not allowed
404  // Not Found - Resource doesn't exist
422  // Unprocessable Entity - Validation failed
429  // Too Many Requests - Rate limit exceeded

// Server Errors
500  // Internal Server Error - Something went wrong
502  // Bad Gateway - Upstream service failed
503  // Service Unavailable - Server overloaded
```

---

## Routes: The Entry Point

**Routes define URL patterns and map them to controllers.**

**File:** `api/server/routes/messages.js`

```javascript
const express = require('express');
const router = express.Router();
const messagesController = require('../controllers/messagesController');
const requireAuth = require('../middleware/requireAuth');
const { validateRequest } = require('../middleware/validateRequest');
const { createMessageSchema } = require('../schemas/message');

/**
 * Routes for message operations
 * Base path: /api/messages
 */

// ===========================
// GET /api/messages/:conversationId
// Get all messages for a conversation
// ===========================
router.get('/:conversationId',
  requireAuth,
  messagesController.getMessages
);

// ===========================
// POST /api/messages
// Create a new message (send to AI)
// ===========================
router.post('/',
  requireAuth,
  validateRequest(createMessageSchema),
  messagesController.createMessage
);

// ===========================
// PUT /api/messages/:id
// Update a message (edit)
// ===========================
router.put('/:id',
  requireAuth,
  messagesController.updateMessage
);

// ===========================
// DELETE /api/messages/:id
// Delete a message
// ===========================
router.delete('/:id',
  requireAuth,
  messagesController.deleteMessage
);

module.exports = router;
```

**React Router Analogy:**

```javascript
// React Router
<Routes>
  <Route path="/messages/:conversationId" element={<Messages />} />
  <Route path="/messages/create" element={<CreateMessage />} />
</Routes>

// Express Router (backend equivalent)
router.get('/messages/:conversationId', messagesController.getMessages);
router.post('/messages', messagesController.createMessage);
```

### Nested Routers

**File:** `api/server/routes/index.js`

```javascript
const express = require('express');
const router = express.Router();

// Import route modules
const authRoutes = require('./auth');
const messageRoutes = require('./messages');
const conversationRoutes = require('./conversations');
const userRoutes = require('./user');

// Mount routes
router.use('/auth', authRoutes);           // /api/auth/*
router.use('/messages', messageRoutes);     // /api/messages/*
router.use('/conversations', conversationRoutes); // /api/conversations/*
router.use('/user', userRoutes);           // /api/user/*

module.exports = router;
```

**Main server file:**

```javascript
// api/server/index.js
const routes = require('./routes');

// Mount all API routes under /api
app.use('/api', routes);

// Now routes are available:
// POST /api/auth/login
// GET /api/messages/:conversationId
// GET /api/conversations
// etc.
```

### Route Parameters vs Query Parameters

```javascript
// URL: /api/messages/123?page=2&limit=10

router.get('/messages/:conversationId', (req, res) => {
  // Route parameters (from URL path)
  const conversationId = req.params.conversationId; // '123'

  // Query parameters (from query string)
  const page = req.query.page;   // '2'
  const limit = req.query.limit; // '10'

  // Use them
  const messages = await getMessages(conversationId, page, limit);
  res.json({ messages });
});
```

**When to use each:**

- **Route params (`:id`):** Identifying resources
  - `/users/:userId` - Specific user
  - `/messages/:messageId` - Specific message

- **Query params (`?key=value`):** Filtering, sorting, pagination
  - `/messages?page=2&sort=date` - Optional filters
  - `/search?q=hello&type=messages` - Search parameters

---

## Controllers: The Event Handlers

**Controllers handle HTTP request/response logic.**

**Think of them as React event handlers:**
```javascript
// React event handler
function handleSubmit(event) {
  event.preventDefault();
  const formData = new FormData(event.target);
  // ... process and send to API
}

// Express controller
function createMessage(req, res, next) {
  const formData = req.body;
  // ... process and send response
}
```

**File:** `api/server/controllers/messagesController.js`

```javascript
const messageService = require('../services/messageService');
const { ValidationError, NotFoundError } = require('../utils/errors');

/**
 * Message Controller
 * Handles HTTP requests for message operations
 */
class MessagesController {

  /**
   * GET /api/messages/:conversationId
   * Get all messages for a conversation
   */
  async getMessages(req, res, next) {
    try {
      const { conversationId } = req.params;
      const userId = req.user.id;

      // Call service layer
      const messages = await messageService.getMessages({
        conversationId,
        userId
      });

      // Send response
      res.json({ messages });

    } catch (error) {
      next(error); // Pass to error handler
    }
  }

  /**
   * POST /api/messages
   * Create a new message
   */
  async createMessage(req, res, next) {
    try {
      const { text, conversationId, model } = req.body;
      const userId = req.user.id;

      // Validate (additional custom validation)
      if (!text || text.trim().length === 0) {
        throw new ValidationError('Message text cannot be empty');
      }

      // Call service layer
      const message = await messageService.createMessage({
        text,
        conversationId,
        model,
        userId
      });

      // Send response with 201 Created status
      res.status(201).json({ message });

    } catch (error) {
      next(error);
    }
  }

  /**
   * PUT /api/messages/:id
   * Update a message
   */
  async updateMessage(req, res, next) {
    try {
      const { id } = req.params;
      const { text } = req.body;
      const userId = req.user.id;

      const updatedMessage = await messageService.updateMessage({
        messageId: id,
        text,
        userId
      });

      res.json({ message: updatedMessage });

    } catch (error) {
      next(error);
    }
  }

  /**
   * DELETE /api/messages/:id
   * Delete a message
   */
  async deleteMessage(req, res, next) {
    try {
      const { id } = req.params;
      const userId = req.user.id;

      await messageService.deleteMessage({
        messageId: id,
        userId
      });

      // 204 No Content (success but no response body)
      res.status(204).send();

    } catch (error) {
      next(error);
    }
  }
}

module.exports = new MessagesController();
```

**Controller Best Practices:**

✅ **Do:**
- Keep controllers thin (minimal logic)
- Focus on HTTP concerns (status codes, headers, response format)
- Delegate business logic to services
- Handle errors gracefully with try/catch
- Validate input (basic validation)
- Use appropriate HTTP status codes

❌ **Don't:**
- Put business logic in controllers
- Make database calls directly
- Make API calls to external services directly
- Complex calculations or algorithms
- Tight coupling to specific implementations

**Example of what NOT to do:**

```javascript
// ❌ Bad: Too much logic in controller
async createMessage(req, res) {
  // Controller is doing too much!
  const user = await User.findById(req.user.id);
  const conversation = await Conversation.findById(req.body.conversationId);

  if (conversation.userId !== user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  const userMessage = new Message({
    conversationId: req.body.conversationId,
    role: 'user',
    text: req.body.text
  });
  await userMessage.save();

  const aiResponse = await openai.chat.completions.create({...});

  const aiMessage = new Message({
    conversationId: req.body.conversationId,
    role: 'assistant',
    text: aiResponse.choices[0].message.content
  });
  await aiMessage.save();

  res.json({ message: aiMessage });
}

// ✅ Good: Delegate to service
async createMessage(req, res, next) {
  try {
    const { text, conversationId, model } = req.body;
    const userId = req.user.id;

    // Service handles all business logic
    const message = await messageService.createMessage({
      text,
      conversationId,
      model,
      userId
    });

    res.status(201).json({ message });
  } catch (error) {
    next(error);
  }
}
```

---

## Services: The Business Logic

**Services contain reusable business logic.**

**React Analogy:** Like custom hooks!

```javascript
// React custom hook
function useMessages(conversationId) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Business logic: fetch and process messages
    fetchMessages(conversationId).then(setMessages);
  }, [conversationId]);

  return messages;
}

// Express service
class MessageService {
  async getMessages(conversationId, userId) {
    // Business logic: fetch and process messages
    const messages = await Message.find({ conversationId });
    return messages;
  }
}
```

**File:** `api/server/services/messageService.js`

```javascript
const Message = require('../../models/Message');
const Conversation = require('../../models/Conversation');
const openai = require('../config/openai');
const { ForbiddenError, NotFoundError } = require('../utils/errors');
const logger = require('../utils/logger');

/**
 * Message Service
 * Business logic for message operations
 */
class MessageService {

  /**
   * Get all messages for a conversation
   */
  async getMessages({ conversationId, userId }) {
    // Verify user owns conversation
    const conversation = await Conversation.findById(conversationId);

    if (!conversation) {
      throw new NotFoundError('Conversation not found');
    }

    if (conversation.userId.toString() !== userId) {
      throw new ForbiddenError('Not your conversation');
    }

    // Fetch messages
    const messages = await Message.find({ conversationId })
      .sort({ createdAt: 1 }) // Oldest first
      .lean(); // Convert to plain JS objects (faster)

    return messages;
  }

  /**
   * Create a new message and get AI response
   */
  async createMessage({ text, conversationId, model, userId }) {
    // ===========================
    // 1. Verify Ownership
    // ===========================
    const conversation = await Conversation.findById(conversationId);

    if (!conversation) {
      throw new NotFoundError('Conversation not found');
    }

    if (conversation.userId.toString() !== userId) {
      throw new ForbiddenError('Not your conversation');
    }

    // ===========================
    // 2. Save User Message
    // ===========================
    const userMessage = await Message.create({
      conversationId,
      role: 'user',
      text,
      userId,
      model
    });

    logger.info('User message created', {
      messageId: userMessage.id,
      userId,
      conversationId
    });

    // ===========================
    // 3. Build Conversation History
    // ===========================
    const previousMessages = await Message.find({ conversationId })
      .sort({ createdAt: 1 })
      .limit(20) // Last 20 messages for context
      .lean();

    const messages = previousMessages.map(msg => ({
      role: msg.role,
      content: msg.text
    }));

    // ===========================
    // 4. Call AI Provider
    // ===========================
    let aiResponse;

    try {
      aiResponse = await this.callAIProvider(model, messages);
    } catch (error) {
      logger.error('AI provider error', { error, userId, conversationId });

      // Save error message
      await Message.create({
        conversationId,
        role: 'assistant',
        text: 'Sorry, I encountered an error. Please try again.',
        error: error.message,
        userId,
        model
      });

      throw error;
    }

    // ===========================
    // 5. Save AI Response
    // ===========================
    const aiMessage = await Message.create({
      conversationId,
      role: 'assistant',
      text: aiResponse,
      userId,
      model
    });

    // ===========================
    // 6. Update Conversation
    // ===========================
    await Conversation.findByIdAndUpdate(conversationId, {
      lastMessageAt: new Date(),
      messageCount: conversation.messageCount + 2 // user + AI
    });

    logger.info('AI message created', {
      messageId: aiMessage.id,
      userId,
      conversationId,
      model
    });

    return aiMessage;
  }

  /**
   * Call AI provider based on model
   * (Abstraction for different AI APIs)
   */
  async callAIProvider(model, messages) {
    if (model.startsWith('gpt-')) {
      // OpenAI
      const response = await openai.chat.completions.create({
        model,
        messages
      });
      return response.choices[0].message.content;

    } else if (model.startsWith('claude-')) {
      // Anthropic
      const anthropic = require('../config/anthropic');
      const response = await anthropic.messages.create({
        model,
        messages
      });
      return response.content[0].text;

    } else if (model.startsWith('gemini-')) {
      // Google
      const google = require('../config/google');
      const response = await google.generateContent({
        model,
        messages
      });
      return response.text;
    }

    throw new Error(`Unsupported model: ${model}`);
  }

  /**
   * Update a message
   */
  async updateMessage({ messageId, text, userId }) {
    const message = await Message.findById(messageId);

    if (!message) {
      throw new NotFoundError('Message not found');
    }

    if (message.userId.toString() !== userId) {
      throw new ForbiddenError('Not your message');
    }

    // Only allow editing user messages
    if (message.role !== 'user') {
      throw new ForbiddenError('Can only edit your own messages');
    }

    message.text = text;
    message.edited = true;
    message.editedAt = new Date();
    await message.save();

    logger.info('Message updated', { messageId, userId });

    return message;
  }

  /**
   * Delete a message
   */
  async deleteMessage({ messageId, userId }) {
    const message = await Message.findById(messageId);

    if (!message) {
      throw new NotFoundError('Message not found');
    }

    if (message.userId.toString() !== userId) {
      throw new ForbiddenError('Not your message');
    }

    await message.deleteOne();

    logger.info('Message deleted', { messageId, userId });
  }
}

module.exports = new MessageService();
```

**Service Best Practices:**

✅ **Do:**
- Keep business logic in services (reusable)
- Make services testable (pure functions when possible)
- Handle complex orchestration
- Interact with models/database
- Call external APIs
- Implement caching strategies
- Log important operations
- Throw descriptive errors

❌ **Don't:**
- Handle HTTP requests/responses directly (that's controllers)
- Access `req` or `res` objects
- Set status codes or headers
- Return different formats for different clients

---

## Models: The Data Layer

**Models define database schemas and queries.**

**React Analogy:** Like TypeScript interfaces + validation + database operations

**File:** `api/models/Message.js`

```javascript
const mongoose = require('mongoose');

/**
 * Message Schema
 * Represents a single message in a conversation
 */
const messageSchema = new mongoose.Schema(
  {
    conversationId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Conversation',
      required: true,
      index: true // Create index for faster queries
    },

    userId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      index: true
    },

    role: {
      type: String,
      enum: ['user', 'assistant', 'system'],
      required: true
    },

    text: {
      type: String,
      required: true,
      maxlength: 10000
    },

    model: {
      type: String,
      enum: ['gpt-4', 'gpt-3.5-turbo', 'claude-3', 'gemini-pro']
    },

    edited: {
      type: Boolean,
      default: false
    },

    editedAt: Date,

    error: String // Store error message if AI call failed
  },
  {
    timestamps: true // Automatically add createdAt, updatedAt
  }
);

// ===========================
// Indexes for Performance
// ===========================

// Compound index for common query
messageSchema.index({ conversationId: 1, createdAt: -1 });

// ===========================
// Virtual Properties
// ===========================

// Virtual: not stored in DB, computed on read
messageSchema.virtual('isEdited').get(function() {
  return this.edited && this.editedAt != null;
});

// ===========================
// Instance Methods
// ===========================

messageSchema.methods.toClient = function() {
  // Transform for API response
  return {
    id: this._id.toString(),
    conversationId: this.conversationId.toString(),
    role: this.role,
    text: this.text,
    model: this.model,
    edited: this.edited,
    editedAt: this.editedAt,
    createdAt: this.createdAt
  };
};

// ===========================
// Static Methods
// ===========================

messageSchema.statics.findByConversation = function(conversationId, options = {}) {
  const { limit = 50, skip = 0 } = options;

  return this.find({ conversationId })
    .sort({ createdAt: 1 })
    .limit(limit)
    .skip(skip)
    .lean();
};

// ===========================
// Middleware (Hooks)
// ===========================

// Before save
messageSchema.pre('save', function(next) {
  // Trim whitespace
  if (this.text) {
    this.text = this.text.trim();
  }
  next();
});

// After delete
messageSchema.post('deleteOne', { document: true }, async function() {
  // Update conversation message count
  const Conversation = mongoose.model('Conversation');
  await Conversation.findByIdAndUpdate(this.conversationId, {
    $inc: { messageCount: -1 }
  });
});

const Message = mongoose.model('Message', messageSchema);

module.exports = Message;
```

**React + TypeScript Analogy:**

```typescript
// React: TypeScript interface
interface Message {
  id: string;
  conversationId: string;
  role: 'user' | 'assistant' | 'system';
  text: string;
  model?: string;
  edited: boolean;
  editedAt?: Date;
  createdAt: Date;
}

// Express: Mongoose schema (similar but with database)
const messageSchema = new mongoose.Schema({
  conversationId: { type: ObjectId, required: true },
  role: { type: String, enum: ['user', 'assistant', 'system'] },
  text: { type: String, required: true },
  // ... etc
});
```

**Key difference:** Mongoose schemas are validated at **runtime** (when saving to DB), TypeScript interfaces are checked at **compile time** (when writing code).

### Common Mongoose Patterns

**1. Basic CRUD Operations**

```javascript
// Create
const message = await Message.create({
  conversationId: '123',
  userId: 'user456',
  role: 'user',
  text: 'Hello!'
});

// Read (find one)
const message = await Message.findById('msg789');

// Read (find many)
const messages = await Message.find({ conversationId: '123' });

// Update
const updated = await Message.findByIdAndUpdate(
  'msg789',
  { text: 'Updated text' },
  { new: true } // Return updated document
);

// Delete
await Message.findByIdAndDelete('msg789');
// or
await message.deleteOne();
```

**2. Query Chaining**

```javascript
const messages = await Message
  .find({ conversationId: '123' })
  .where('role').equals('user')
  .where('createdAt').gte(yesterday)
  .sort({ createdAt: -1 })
  .limit(20)
  .skip(0)
  .select('text role createdAt') // Only return these fields
  .lean(); // Return plain JS objects (faster)
```

**3. Population (Joins)**

```javascript
// Populate related data
const message = await Message
  .findById('msg789')
  .populate('userId', 'name email') // Load user data
  .populate('conversationId', 'title');

// Result:
{
  _id: 'msg789',
  text: 'Hello',
  userId: {
    _id: 'user456',
    name: 'John Doe',
    email: 'john@example.com'
  },
  conversationId: {
    _id: '123',
    title: 'My Conversation'
  }
}
```

**4. Aggregation (Complex Queries)**

```javascript
// Count messages per user
const stats = await Message.aggregate([
  // Stage 1: Match documents
  { $match: { createdAt: { $gte: lastWeek } } },

  // Stage 2: Group by userId
  {
    $group: {
      _id: '$userId',
      messageCount: { $sum: 1 },
      totalCharacters: { $sum: { $strLenCP: '$text' } }
    }
  },

  // Stage 3: Sort by count
  { $sort: { messageCount: -1 } },

  // Stage 4: Limit results
  { $limit: 10 }
]);

// Result:
[
  { _id: 'user456', messageCount: 150, totalCharacters: 12000 },
  { _id: 'user789', messageCount: 120, totalCharacters: 9500 },
  // ...
]
```

---

## Error Handling

### Custom Error Classes

**File:** `api/server/utils/errors.js`

```javascript
/**
 * Base application error
 */
class AppError extends Error {
  constructor(message, status = 500) {
    super(message);
    this.status = status;
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

/**
 * 400 Bad Request - Invalid input
 */
class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}

/**
 * 401 Unauthorized - Authentication required
 */
class UnauthorizedError extends AppError {
  constructor(message = 'Authentication required') {
    super(message, 401);
  }
}

/**
 * 403 Forbidden - Not allowed
 */
class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403);
  }
}

/**
 * 404 Not Found - Resource doesn't exist
 */
class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

/**
 * 429 Too Many Requests - Rate limit exceeded
 */
class RateLimitError extends AppError {
  constructor(message = 'Too many requests') {
    super(message, 429);
  }
}

module.exports = {
  AppError,
  ValidationError,
  UnauthorizedError,
  ForbiddenError,
  NotFoundError,
  RateLimitError
};
```

### Global Error Handler

**File:** `api/server/middleware/errorHandler.js`

```javascript
const logger = require('../utils/logger');
const { AppError } = require('../utils/errors');

/**
 * Global error handling middleware
 * Catches all errors and formats response
 */
function errorHandler(err, req, res, next) {
  // Log error
  logger.error('Request error', {
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    userId: req.user?.id
  });

  // ===========================
  // Known Errors (AppError)
  // ===========================
  if (err instanceof AppError) {
    return res.status(err.status).json({
      error: {
        message: err.message,
        status: err.status,
        type: err.name
      }
    });
  }

  // ===========================
  // Mongoose Validation Error
  // ===========================
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: {
        message: 'Validation failed',
        status: 400,
        details: Object.values(err.errors).map(e => ({
          field: e.path,
          message: e.message
        }))
      }
    });
  }

  // ===========================
  // Mongoose Cast Error (Invalid ID)
  // ===========================
  if (err.name === 'CastError') {
    return res.status(400).json({
      error: {
        message: `Invalid ${err.path}: ${err.value}`,
        status: 400
      }
    });
  }

  // ===========================
  // MongoDB Duplicate Key Error
  // ===========================
  if (err.code === 11000) {
    const field = Object.keys(err.keyPattern)[0];
    return res.status(409).json({
      error: {
        message: `${field} already exists`,
        status: 409
      }
    });
  }

  // ===========================
  // JWT Error
  // ===========================
  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({
      error: {
        message: 'Invalid token',
        status: 401
      }
    });
  }

  if (err.name === 'TokenExpiredError') {
    return res.status(401).json({
      error: {
        message: 'Token expired',
        status: 401
      }
    });
  }

  // ===========================
  // Unknown Error
  // ===========================
  const isDevelopment = process.env.NODE_ENV === 'development';

  res.status(500).json({
    error: {
      message: 'Internal server error',
      status: 500,
      // Only include stack trace in development
      ...(isDevelopment && { stack: err.stack })
    }
  });
}

module.exports = errorHandler;
```

### Using Errors in Code

```javascript
const { NotFoundError, ForbiddenError } = require('../utils/errors');

async function deleteMessage({ messageId, userId }) {
  const message = await Message.findById(messageId);

  // Throw custom error - will be caught by error handler
  if (!message) {
    throw new NotFoundError('Message not found');
  }

  if (message.userId.toString() !== userId) {
    throw new ForbiddenError('Not your message');
  }

  await message.deleteOne();
}
```

**React Analogy:**

```javascript
// React: Error boundaries
try {
  // risky operation
} catch (error) {
  // Show error UI
  setError(error.message);
}

// Express: Error handler middleware
try {
  // risky operation
} catch (error) {
  next(error); // Pass to error handler → sends error response
}
```

---

## Authentication & Authorization

### Passport.js Strategies

**File:** `api/server/config/passport.js`

```javascript
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;
const bcrypt = require('bcryptjs');
const User = require('../../models/User');

/**
 * Local Strategy (Username + Password)
 * Used for login
 */
passport.use(
  new LocalStrategy(
    {
      usernameField: 'email', // Use email instead of username
      passwordField: 'password'
    },
    async (email, password, done) => {
      try {
        // Find user by email
        const user = await User.findOne({ email });

        if (!user) {
          return done(null, false, { message: 'Incorrect email' });
        }

        // Compare password
        const isMatch = await bcrypt.compare(password, user.password);

        if (!isMatch) {
          return done(null, false, { message: 'Incorrect password' });
        }

        // Success
        return done(null, user);

      } catch (error) {
        return done(error);
      }
    }
  )
);

/**
 * JWT Strategy
 * Used for protected routes
 */
const jwtOptions = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET
};

passport.use(
  new JwtStrategy(jwtOptions, async (payload, done) => {
    try {
      // Find user by ID from JWT payload
      const user = await User.findById(payload.id);

      if (!user) {
        return done(null, false);
      }

      return done(null, user);

    } catch (error) {
      return done(error, false);
    }
  })
);

module.exports = passport;
```

### Auth Controller

**File:** `api/server/controllers/authController.js`

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../../models/User');
const { ValidationError, UnauthorizedError } = require('../utils/errors');

/**
 * Authentication Controller
 */
class AuthController {

  /**
   * POST /api/auth/register
   * Register new user
   */
  async register(req, res, next) {
    try {
      const { name, email, password } = req.body;

      // Check if user exists
      const existingUser = await User.findOne({ email });
      if (existingUser) {
        throw new ValidationError('Email already registered');
      }

      // Hash password
      const salt = await bcrypt.genSalt(10);
      const passwordHash = await bcrypt.hash(password, salt);

      // Create user
      const user = await User.create({
        name,
        email,
        password: passwordHash
      });

      // Generate JWT
      const token = jwt.sign(
        { id: user._id, email: user.email },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      res.status(201).json({
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

  /**
   * POST /api/auth/login
   * Login user
   */
  async login(req, res, next) {
    try {
      const { email, password } = req.body;

      // Find user
      const user = await User.findOne({ email }).select('+password');

      if (!user) {
        throw new UnauthorizedError('Invalid credentials');
      }

      // Check password
      const isMatch = await bcrypt.compare(password, user.password);

      if (!isMatch) {
        throw new UnauthorizedError('Invalid credentials');
      }

      // Generate JWT
      const token = jwt.sign(
        { id: user._id, email: user.email },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      });

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

  /**
   * GET /api/auth/me
   * Get current user (protected route)
   */
  async getCurrentUser(req, res, next) {
    try {
      // req.user is set by requireAuth middleware
      const user = await User.findById(req.user.id);

      res.json({
        user: {
          id: user._id,
          name: user.name,
          email: user.email
        }
      });

    } catch (error) {
      next(error);
    }
  }

  /**
   * POST /api/auth/refresh
   * Refresh JWT token
   */
  async refreshToken(req, res, next) {
    try {
      const userId = req.user.id;

      // Generate new token
      const token = jwt.sign(
        { id: userId },
        process.env.JWT_SECRET,
        { expiresIn: '7d' }
      );

      res.json({ token });

    } catch (error) {
      next(error);
    }
  }
}

module.exports = new AuthController();
```

### Auth Routes

**File:** `api/server/routes/auth.js`

```javascript
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const requireAuth = require('../middleware/requireAuth');

// Public routes
router.post('/register', authController.register);
router.post('/login', authController.login);

// Protected routes (require authentication)
router.get('/me', requireAuth, authController.getCurrentUser);
router.post('/refresh', requireAuth, authController.refreshToken);

module.exports = router;
```

**React Analogy:**

```javascript
// React: Protected route with Context
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" />;
  return children;
}

// Express: Protected route with middleware
router.get('/api/protected', requireAuth, (req, res) => {
  // req.user is available here
  res.json({ data: 'Protected data' });
});
```

---

## Logging & Monitoring

### Winston Logger Setup

**File:** `api/server/utils/logger.js`

```javascript
const winston = require('winston');
const path = require('path');

/**
 * Logger configuration
 * Uses Winston for structured logging
 */

// Define log levels
const levels = {
  error: 0,
  warn: 1,
  info: 2,
  http: 3,
  debug: 4
};

// Define colors for each level
const colors = {
  error: 'red',
  warn: 'yellow',
  info: 'green',
  http: 'magenta',
  debug: 'blue'
};

winston.addColors(colors);

// Determine log level based on environment
const level = () => {
  const env = process.env.NODE_ENV || 'development';
  return env === 'development' ? 'debug' : 'info';
};

// Define log format
const format = winston.format.combine(
  winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
  winston.format.errors({ stack: true }),
  winston.format.splat(),
  winston.format.json()
);

// Define transports (where logs are saved)
const transports = [
  // Console output (development)
  new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize({ all: true }),
      winston.format.printf(
        info => `${info.timestamp} ${info.level}: ${info.message}`
      )
    )
  }),

  // Error logs file
  new winston.transports.File({
    filename: path.join(__dirname, '../../logs/error.log'),
    level: 'error',
    maxsize: 5242880, // 5MB
    maxFiles: 5
  }),

  // Combined logs file
  new winston.transports.File({
    filename: path.join(__dirname, '../../logs/combined.log'),
    maxsize: 5242880, // 5MB
    maxFiles: 5
  })
];

// Create logger
const logger = winston.createLogger({
  level: level(),
  levels,
  format,
  transports
});

module.exports = logger;
```

### Using the Logger

```javascript
const logger = require('./utils/logger');

// Different log levels
logger.error('Payment failed', {
  userId: '123',
  amount: 99.99,
  error: error.message
});

logger.warn('API rate limit approaching', {
  userId: '123',
  requestCount: 95,
  limit: 100
});

logger.info('User logged in', {
  userId: '123',
  email: 'user@example.com'
});

logger.http('GET /api/messages', {
  path: '/api/messages',
  method: 'GET',
  statusCode: 200,
  duration: '45ms'
});

logger.debug('Cache hit', {
  key: 'messages:123',
  ttl: 300
});
```

### Request Logging Middleware

```javascript
const logger = require('../utils/logger');

/**
 * Log all HTTP requests
 */
function requestLogger(req, res, next) {
  const start = Date.now();

  // Log when response is sent
  res.on('finish', () => {
    const duration = Date.now() - start;

    logger.http(`${req.method} ${req.path}`, {
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userId: req.user?.id
    });
  });

  next();
}

app.use(requestLogger);
```

---

## API Design Patterns

### RESTful API Design

**REST principles:**

1. **Resources:** Use nouns, not verbs
2. **HTTP Methods:** GET, POST, PUT, DELETE
3. **Status Codes:** Use appropriate codes
4. **Stateless:** Each request is independent

**Good REST API design:**

```javascript
// ✅ Good
GET    /api/messages              # Get all messages
GET    /api/messages/:id          # Get specific message
POST   /api/messages              # Create message
PUT    /api/messages/:id          # Update message
DELETE /api/messages/:id          # Delete message

// ❌ Bad
GET    /api/getMessages           # Don't use verbs
POST   /api/createMessage         # Use HTTP method instead
GET    /api/message/delete/:id    # Wrong HTTP method
```

### Response Format Consistency

**Successful response:**
```javascript
res.json({
  data: { ... },
  meta: {
    page: 1,
    limit: 20,
    total: 100
  }
});
```

**Error response:**
```javascript
res.status(400).json({
  error: {
    message: 'Validation failed',
    status: 400,
    details: [...]
  }
});
```

### Pagination

```javascript
router.get('/messages', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const skip = (page - 1) * limit;

  const messages = await Message
    .find({ conversationId: req.query.conversationId })
    .sort({ createdAt: -1 })
    .limit(limit)
    .skip(skip);

  const total = await Message.countDocuments({
    conversationId: req.query.conversationId
  });

  res.json({
    data: messages,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  });
});
```

### Filtering and Sorting

```javascript
router.get('/messages', async (req, res) => {
  // Build filter
  const filter = {};

  if (req.query.conversationId) {
    filter.conversationId = req.query.conversationId;
  }

  if (req.query.role) {
    filter.role = req.query.role;
  }

  if (req.query.search) {
    filter.text = { $regex: req.query.search, $options: 'i' };
  }

  // Build sort
  const sortBy = req.query.sortBy || 'createdAt';
  const sortOrder = req.query.sortOrder === 'asc' ? 1 : -1;

  const messages = await Message
    .find(filter)
    .sort({ [sortBy]: sortOrder })
    .limit(20);

  res.json({ data: messages });
});

// Usage:
// GET /api/messages?conversationId=123&role=user&sortBy=createdAt&sortOrder=desc
```

---

## Performance Considerations

### Database Indexing

```javascript
// Add indexes for frequently queried fields
messageSchema.index({ conversationId: 1, createdAt: -1 });
messageSchema.index({ userId: 1 });
messageSchema.index({ text: 'text' }); // Full-text search index
```

### Caching with Redis

```javascript
const redis = require('./config/redis');

/**
 * Get messages with caching
 */
async function getMessages(conversationId) {
  const cacheKey = `messages:${conversationId}`;

  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // Fetch from database
  const messages = await Message.find({ conversationId });

  // Store in cache (5 minutes)
  await redis.setex(cacheKey, 300, JSON.stringify(messages));

  return messages;
}
```

### Query Optimization

```javascript
// ❌ Bad: N+1 query problem
const messages = await Message.find({ conversationId: '123' });
for (const message of messages) {
  const user = await User.findById(message.userId); // Separate query for each!
}

// ✅ Good: Use populate (single query)
const messages = await Message
  .find({ conversationId: '123' })
  .populate('userId', 'name email');
```

### Use `.lean()` for Read-Only Data

```javascript
// ❌ Slower: Returns Mongoose documents (with methods)
const messages = await Message.find({ conversationId: '123' });

// ✅ Faster: Returns plain JavaScript objects
const messages = await Message
  .find({ conversationId: '123' })
  .lean();
```

---

## Summary

**Backend Architecture Principles:**

1. **Separation of Concerns**
   - Routes → Controllers → Services → Models
   - Each layer has a specific responsibility

2. **Middleware Chain**
   - Request flows through middleware
   - Each middleware can modify req/res or stop the chain

3. **Error Handling**
   - Custom error classes
   - Global error handler
   - Consistent error responses

4. **Authentication**
   - Passport.js strategies
   - JWT tokens
   - Protected routes with middleware

5. **Logging**
   - Winston for structured logging
   - Different log levels
   - Log important operations

6. **API Design**
   - RESTful principles
   - Consistent response format
   - Proper HTTP status codes

7. **Performance**
   - Database indexing
   - Caching with Redis
   - Query optimization

---

**Next Steps:**

- Read [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - MongoDB and Mongoose deep dive
- Read [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Connect frontend to backend
- Read [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step common tasks

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
