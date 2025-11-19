# ❓ Frequently Asked Questions (FAQ)

**Last Updated:** November 19, 2025
**For:** Mid-level React developers learning full-stack with LibreChat

---

## Table of Contents

- [General Questions](#general-questions)
- [Getting Started](#getting-started)
- [Frontend Questions](#frontend-questions)
- [Backend Questions](#backend-questions)
- [Database Questions](#database-questions)
- [Architecture & Design](#architecture--design)
- [Development Workflow](#development-workflow)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## General Questions

### What is LibreChat?

**LibreChat** is an open-source AI chat aggregator that provides a unified interface for multiple AI providers (OpenAI, Anthropic, Google, etc.). Think of it as **"one app for all AI chatbots"** - like Spotify is to music streaming services.

**Key features:**
- Multi-provider AI chat (OpenAI, Claude, Gemini, etc.)
- User authentication and conversation history
- Custom AI agents with tools
- File uploads and image generation
- Web search integration
- Code interpreter (sandboxed code execution)

---

### Is this suitable for learning backend development?

**Absolutely yes!** LibreChat is perfect for learning because:

✅ **Real-world production code** - Not a tutorial project
✅ **Modern best practices** - Up-to-date patterns (as of Nov 2025)
✅ **All JavaScript** - No new language to learn
✅ **Well-organized** - Clear separation of concerns
✅ **Active development** - See how real teams work
✅ **Comprehensive features** - Auth, databases, APIs, caching, etc.

**Compared to simple tutorials:**
- More complex (realistic)
- Better organized (scalable patterns)
- Production-ready (best practices)

---

### Do I need to know backend development to start?

**No!** This learning path assumes **zero backend experience**. We explain everything from first principles and connect backend concepts to React patterns you already know.

**Prerequisites:**
- ✅ JavaScript ES6+ (const, arrow functions, async/await)
- ✅ React basics (components, props, state, hooks)
- ✅ Command line basics (cd, ls, running commands)

**We teach you:**
- 🆕 Node.js and Express
- 🆕 MongoDB and databases
- 🆕 REST APIs
- 🆕 Authentication
- 🆕 Backend architecture

---

### How long will it take to understand the codebase?

**It depends on your goals:**

| Goal | Time Estimate |
|------|---------------|
| Get it running locally | 1-3 hours |
| Understand architecture | 4-6 hours |
| Build a simple feature | 10-15 hours |
| Confident full-stack dev | 40-60 hours |
| Deep backend expertise | 60-100 hours |

**Learning is not linear!** You'll have "aha moments" at different times. Work at your own pace.

---

### What if I get stuck?

**You have options:**

1. **Check this documentation** - Use the search/index
2. **Read the code** - Links to actual files throughout docs
3. **Check logs** - Error messages often explain the issue
4. **Ask in Discord** - [discord.librechat.ai](https://discord.librechat.ai)
5. **Search GitHub Issues** - Someone may have had the same problem
6. **Create an issue** - If you found a bug or gap in docs

**🎯 Tip:** When asking for help, include:
- What you're trying to do
- What you expected
- What actually happened
- Error messages (full stack trace)
- Your environment (OS, Node version, etc.)

---

## Getting Started

### Can I use this on Windows?

**Yes!** LibreChat works on Windows. Recommended approaches:

**Option 1: Docker (Easiest)**
- Install Docker Desktop for Windows
- Follow Docker setup in [GETTING_STARTED.md](./GETTING_STARTED.md)
- Everything runs in containers (no Windows-specific issues)

**Option 2: WSL2 (Recommended for local dev)**
- Install Windows Subsystem for Linux 2
- Install Node.js, MongoDB, Redis in WSL
- Much closer to production Linux environment

**Option 3: Native Windows**
- Install Node.js for Windows
- Install MongoDB for Windows
- Install Redis (use WSL or Windows port)
- May have path/permission issues

**Best:** Docker or WSL2 for consistent experience.

---

### Do I need API keys to develop?

**No, not for most development!**

**Without API keys:**
- ✅ UI works perfectly
- ✅ Authentication works
- ✅ Database interactions work
- ✅ Can test frontend features
- ✅ Can test backend endpoints (mock AI responses)
- ❌ Can't actually chat with AI

**Perfect for:**
- Learning frontend (React, UI components)
- Learning backend (routes, controllers, database)
- Testing features
- Building new functionality

**Need API keys for:**
- Actually chatting with OpenAI, Claude, etc.
- Testing AI-specific features
- Production use

**🎯 Tip:** Start without API keys. Add them later when you want to test AI chat.

---

### What's the difference between development and production mode?

| Aspect | Development Mode | Production Mode |
|--------|-----------------|-----------------|
| **Start command** | `npm run frontend:dev` + `npm run backend:dev` | `npm run frontend` + `npm run backend` |
| **Ports** | Frontend: 3090, Backend: 3080 | Everything: 3080 |
| **Hot reload** | ✅ Yes (instant updates) | ❌ No (must rebuild) |
| **Build time** | ⚡ Instant (no build) | 🐌 5-10 min first time |
| **Performance** | Slower (for development) | Optimized (minified, bundled) |
| **Debugging** | Easy (source maps) | Harder (minified code) |
| **Use for** | Coding, testing, learning | Testing production build, deployment |

**🎯 Recommendation:** Use **development mode** for learning and coding.

---

## Frontend Questions

### Why use TanStack Query instead of useEffect for data fetching?

**❌ Old way (useEffect):**
```javascript
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
  }, []) // Runs on mount

  // Manual refetching, caching, error retry, etc.
}
```

**✅ Modern way (TanStack Query):**
```javascript
function Messages() {
  const { data: messages, isLoading, error } = useQuery({
    queryKey: ['messages'],
    queryFn: () => fetch('/api/messages').then(res => res.json())
  })

  // Automatic:
  // - Caching
  // - Background refetching
  // - Deduplication
  // - Retry on error
  // - Loading/error states
}
```

**Why TanStack Query is better:**
- ✅ **Caching** - Don't refetch data you already have
- ✅ **Deduplication** - Multiple components, one request
- ✅ **Background refetching** - Keep data fresh automatically
- ✅ **Retry logic** - Auto-retry failed requests
- ✅ **Less boilerplate** - 10 lines → 3 lines

**Industry standard in 2025!**

---

### Why both Jotai AND Recoil for state management?

**Good question!** This might be legacy code or intentional separation:

**Possible reasons:**
1. **Migration in progress** - Moving from Recoil to Jotai (or vice versa)
2. **Different use cases** - Recoil for X, Jotai for Y
3. **Team preference** - Different developers prefer different tools

**🔍 To investigate:**
```bash
# See where each is used
grep -r "useRecoilState" client/src/
grep -r "useAtom" client/src/
```

**In your code:**
Pick one (Jotai is more actively maintained as of Nov 2025) and use it consistently.

---

### What's the difference between Tailwind CSS and CSS-in-JS?

| Tailwind CSS | CSS-in-JS (styled-components, etc.) |
|--------------|-------------------------------------|
| **Utility classes** | **Component styles** |
| `<div className="flex items-center gap-4">` | `<Flex align="center" gap={4}>` |
| Styles in HTML | Styles in JavaScript |
| No CSS files | JavaScript generates CSS |
| Learn utility class names | Write CSS in JS |
| Fast (CSS purging) | Runtime overhead (small) |
| Less code co-location | Full co-location |

**LibreChat uses Tailwind because:**
- Rapid development (no switching files)
- Consistent design system (predefined spacing, colors)
- Smaller bundle (unused classes purged)
- Industry trend (very popular in 2025)

**Learning curve:** Takes 2-3 hours to get comfortable with Tailwind classes.

---

## Backend Questions

### What's the difference between controllers and services?

**Great question!** This is a **separation of concerns** pattern:

**Controllers** = **Request/Response Handlers**
- Handle HTTP requests
- Validate input
- Call services
- Return HTTP responses
- **Thin layer** - minimal logic

**Services** = **Business Logic**
- Pure logic (no req/res)
- Database operations
- External API calls
- Complex algorithms
- **Thick layer** - where work happens

**Example:**

```javascript
// ❌ BAD: Everything in controller
app.post('/api/messages', async (req, res) => {
  const { text } = req.body
  const message = await Message.create({ text, userId: req.user.id })
  const aiResponse = await openai.chat.completions.create(...)
  const aiMessage = await Message.create({ text: aiResponse.text })
  res.json({ message, aiMessage })
})

// ✅ GOOD: Controller + Service
// Controller (thin)
app.post('/api/messages', async (req, res) => {
  const { text } = req.body
  const result = await MessageService.createWithAI(text, req.user.id)
  res.json(result)
})

// Service (thick)
class MessageService {
  static async createWithAI(text, userId) {
    const message = await Message.create({ text, userId })
    const aiResponse = await AIService.generateResponse(message)
    const aiMessage = await Message.create({ text: aiResponse, sender: 'ai' })
    return { message, aiMessage }
  }
}
```

**Why separate?**
- ✅ **Testable** - Test services without HTTP
- ✅ **Reusable** - Call services from multiple routes
- ✅ **Organized** - Clear responsibilities
- ✅ **Maintainable** - Easy to find logic

**🌉 React equivalent:**
- **Controllers** = Event handlers
- **Services** = Custom hooks (pure logic)

---

### What is middleware and how does it work?

**Middleware** = Functions that run **between** receiving a request and sending a response.

**Think of it like an assembly line:**
```
Request → Middleware 1 → Middleware 2 → Middleware 3 → Route Handler → Response
```

**Example:**
```javascript
// Middleware 1: Log request
function logRequest(req, res, next) {
  console.log(`${req.method} ${req.url}`)
  next() // Pass to next middleware
}

// Middleware 2: Check authentication
function requireAuth(req, res, next) {
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Unauthorized' })
  }
  req.user = { id: '123' } // Add user to request
  next() // Pass to next middleware
}

// Route handler (final destination)
app.get('/api/profile', logRequest, requireAuth, (req, res) => {
  res.json({ user: req.user }) // User added by middleware!
})
```

**Execution order:**
1. `logRequest` runs → logs "GET /api/profile" → calls `next()`
2. `requireAuth` runs → checks auth → adds `req.user` → calls `next()`
3. Route handler runs → accesses `req.user` → sends response

**🌉 React equivalent:**
- Middleware = Higher-Order Components (HOCs)
- `next()` = Render the wrapped component
- `req` = Props passed down

**Common middleware:**
- Authentication (`requireAuth`)
- Validation (`validateRequest`)
- Logging (`logRequest`)
- Error handling (`errorHandler`)
- Rate limiting (`rateLimit`)

---

### Why can't I just put everything in one file?

**You can!** But you shouldn't. Here's why:

**❌ One big file:**
```javascript
// server.js - 5000 lines
const express = require('express')
const app = express()

app.post('/api/messages', async (req, res) => { /* ... */ })
app.get('/api/conversations', async (req, res) => { /* ... */ })
app.post('/api/auth/login', async (req, res) => { /* ... */ })
// ... 100 more routes
// ... all business logic
// ... all database queries
```

**Problems:**
- Hard to find things
- Hard to test
- Hard to reuse code
- Merge conflicts with teammates
- Violates Single Responsibility Principle

**✅ Organized structure:**
```
server/
├── routes/
│   ├── messages.js    # Message routes
│   ├── auth.js        # Auth routes
│   └── ...
├── controllers/
│   ├── MessageController.js
│   └── ...
├── services/
│   ├── MessageService.js
│   └── ...
└── index.js          # < 50 lines!
```

**Benefits:**
- Easy to navigate
- Easy to test (import service, test it)
- Easy to reuse (call service from anywhere)
- Team-friendly (fewer conflicts)
- Follows best practices

**🎯 Rule of thumb:** If a file is > 200 lines, split it up.

---

## Database Questions

### What's the difference between SQL and NoSQL?

| SQL (PostgreSQL, MySQL) | NoSQL (MongoDB) |
|------------------------|-----------------|
| **Tables & Rows** | **Collections & Documents** |
| Structured schema | Flexible schema |
| Relations via foreign keys | Relations via refs or embedding |
| Joins | Population or aggregation |
| ACID transactions | Eventual consistency (configurable) |
| Best for: Relational data | Best for: Document-like data |

**Example:**

**SQL (PostgreSQL):**
```sql
-- Table: users
| id | email           | name    |
|----|-----------------|---------|
| 1  | user@example.com| John    |

-- Table: messages
| id | user_id | text          |
|----|---------|---------------|
| 1  | 1       | Hello world   |

-- Query: Get user's messages
SELECT * FROM messages
JOIN users ON messages.user_id = users.id
WHERE users.id = 1;
```

**NoSQL (MongoDB):**
```javascript
// Collection: users
{
  _id: ObjectId("..."),
  email: "user@example.com",
  name: "John"
}

// Collection: messages
{
  _id: ObjectId("..."),
  userId: ObjectId("..."), // Reference to user
  text: "Hello world"
}

// Query: Get user's messages
const messages = await Message.find({ userId: user._id })
  .populate('userId') // Like SQL JOIN
```

**Why MongoDB for LibreChat?**
- Chat data is **document-oriented** (messages are JSON-like)
- **Flexible schema** - Easy to add new fields (metadata, etc.)
- **Fast for reads** - Optimized for fetching conversations
- **JavaScript-native** - BSON (Binary JSON)

---

### What is Mongoose and why use it?

**Mongoose** = **ODM** (Object Document Mapper) for MongoDB

**Without Mongoose (raw MongoDB driver):**
```javascript
// No schema, no validation
const db = client.db('LibreChat')
const messages = db.collection('messages')

// Any data accepted (dangerous!)
await messages.insertOne({
  txt: "Hello", // Typo! Should be 'text'
  usr: "123"    // No validation
})
```

**With Mongoose:**
```javascript
// Define schema
const MessageSchema = new mongoose.Schema({
  text: { type: String, required: true, maxlength: 10000 },
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  createdAt: { type: Date, default: Date.now }
})

const Message = mongoose.model('Message', MessageSchema)

// Validation automatic!
await Message.create({
  txt: "Hello" // ❌ Error: 'text' is required
})

await Message.create({
  text: "Hello",
  userId: "invalid" // ❌ Error: Invalid ObjectId
})

await Message.create({
  text: "Hello",
  userId: validUserId // ✅ Success!
})
```

**Why Mongoose:**
- ✅ **Schema validation** - Catch errors before they hit DB
- ✅ **Type casting** - Automatic type conversions
- ✅ **Middleware** - Pre/post save hooks
- ✅ **Queries** - Better query API than raw driver
- ✅ **Relationships** - Population (like SQL joins)
- ✅ **Plugins** - Extend functionality

**🌉 React equivalent:**
Mongoose schemas are like **TypeScript interfaces + runtime validation**

---

### How do I query the database to see what's in it?

**Using MongoDB Shell (mongosh):**

```bash
# Connect to MongoDB
mongosh

# Show databases
show dbs

# Use LibreChat database
use LibreChat

# Show collections (tables)
show collections

# Find all users
db.users.find()

# Find all conversations for a specific user
db.conversations.find({ userId: ObjectId("...") })

# Count messages
db.messages.countDocuments()

# Find recent messages (limit 10)
db.messages.find().sort({ createdAt: -1 }).limit(10)

# Pretty print
db.messages.find().pretty()

# Exit
exit
```

**Using MongoDB Compass (GUI):**

1. Download [MongoDB Compass](https://www.mongodb.com/products/compass)
2. Connect to `mongodb://localhost:27017`
3. Navigate to `LibreChat` database
4. Click collections (users, messages, etc.)
5. Browse data visually!

**Using code (in Node.js):**

```javascript
// In a script or REPL
const mongoose = require('mongoose')
await mongoose.connect('mongodb://localhost:27017/LibreChat')

const Message = require('./api/models/Message')
const messages = await Message.find().limit(10)
console.log(messages)
```

---

## Architecture & Design

### Why use a monorepo instead of separate repos?

**Monorepo** = One repository contains multiple projects (client, API, packages)

**✅ Pros:**
- **Shared code** - `data-provider`, `data-schemas` used by both frontend & backend
- **Shared types** - TypeScript types consistent across stack
- **Atomic commits** - Change frontend + backend in one PR
- **Easier navigation** - Everything in one place
- **Single version** - One version number for the whole app

**⚠️ Cons:**
- **Larger repo** - More files to clone
- **Complex dependencies** - Package version management
- **Slower CI** - More tests to run

**Alternatives:**
- **Multi-repo** - Separate repos for frontend, backend, shared packages
- **Micro-frontends** - Each feature is a separate app

**LibreChat's choice:** Monorepo works well for this size. Tools like npm workspaces make it manageable.

---

### Why Express instead of Next.js API routes?

**Great question!** Both are valid choices:

**Next.js API Routes:**
- ✅ Simpler for small apps (everything in one framework)
- ✅ Serverless-ready (deploy to Vercel easily)
- ❌ Tied to Next.js (can't separate frontend/backend)
- ❌ Less flexibility (opinionated structure)

**Express:**
- ✅ Full control (minimal framework)
- ✅ Huge ecosystem (1000s of middleware packages)
- ✅ Can run independently (backend on different server than frontend)
- ✅ Industry standard (most Node.js jobs use Express)
- ✅ Better for learning backend (clear separation)

**LibreChat's choice:** Express because:
1. **Independence** - Backend can run on separate infrastructure
2. **Scalability** - Easier to scale backend separately
3. **Flexibility** - Not tied to Next.js
4. **Learning** - Better for understanding backend architecture

---

### Why store sessions in Redis instead of the database?

**Redis** = In-memory key-value store (super fast)
**MongoDB** = Disk-based document database (slower but persistent)

**Session requirements:**
- ⚡ **Fast reads** - Every request checks session
- 🔄 **High volume** - Many requests per second
- ⏱️ **Short-lived** - Sessions expire (minutes/hours)
- 🗑️ **Auto-cleanup** - Delete expired sessions

**Why Redis wins:**
- ⚡ **Microsecond access** (MongoDB: milliseconds)
- 🚀 **In-memory** (RAM vs disk)
- 🎯 **TTL built-in** (auto-expiration)
- 📈 **Designed for this** (key-value, cache)

**Why not MongoDB for sessions:**
- Slower (disk I/O)
- Manual cleanup (no built-in expiration)
- Not optimized for this use case

**Analogy:**
- **Redis** = Sticky notes (quick access, temporary)
- **MongoDB** = Filing cabinet (persistent, structured)

**🎯 Rule:** Use Redis for **temporary, fast-access data**. Use MongoDB for **permanent, structured data**.

---

## Development Workflow

### How do I test my changes without breaking production?

**Development is already separate from production!**

**Development:**
- Runs on `localhost:3080` or `localhost:3090`
- Uses local MongoDB database
- Separate .env file (development secrets)
- Can break things, experiment freely!

**Production:**
- Runs on your server (e.g., `librechat.example.com`)
- Uses production MongoDB database (or MongoDB Atlas)
- Production .env file (real secrets, API keys)
- Must be stable!

**Workflow:**
1. **Develop locally** - Make changes, test on localhost
2. **Test thoroughly** - Run tests (`npm test`, `npm run e2e`)
3. **Commit to branch** - `git checkout -b feature/my-feature`
4. **Create PR** - Get code review
5. **Merge to main** - After approval
6. **Deploy to production** - Manual or CI/CD

**🎯 You can't "break production" from local development!** They're completely separate environments.

---

### How do I debug backend issues?

**Method 1: Console logs (simplest)**
```javascript
// Add console.log anywhere
app.post('/api/messages', async (req, res) => {
  console.log('Request body:', req.body)
  console.log('User:', req.user)

  const message = await MessageService.create(req.body)
  console.log('Created message:', message)

  res.json(message)
})
```

Check logs in terminal where you ran `npm run backend:dev`

**Method 2: VS Code Debugger (best)**

Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "program": "${workspaceFolder}/api/server/index.js",
      "env": {
        "NODE_ENV": "development"
      }
    }
  ]
}
```

Set breakpoints, press F5, debug!

**Method 3: Node.js inspector**
```bash
# Start with inspector
node --inspect api/server/index.js

# Open Chrome to: chrome://inspect
# Click "inspect" on your app
```

**Method 4: Check logs**
```bash
# Docker logs
docker compose logs -f api

# Winston logs (if configured to file)
tail -f logs/app.log
```

---

### What's the Git workflow for contributing?

**Standard Git Flow:**

```bash
# 1. Create a branch
git checkout -b feature/my-new-feature

# 2. Make changes
# ... edit files ...

# 3. Stage changes
git add .

# 4. Commit with message
git commit -m "feat: add new feature"

# 5. Push to your fork (if external contributor)
git push origin feature/my-new-feature

# 6. Create Pull Request on GitHub

# 7. Address review feedback
# ... make more changes ...
git add .
git commit -m "fix: address review feedback"
git push

# 8. Merge (after approval)
# Done via GitHub UI
```

**Commit message format:**
```
type: short description

Longer description if needed.

- Bullet points for details
```

**Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation
- `refactor:` - Code restructure (no behavior change)
- `test:` - Add tests
- `chore:` - Maintenance (deps, config, etc.)

---

## Troubleshooting

### "Cannot find module '@librechat/...'"

**Cause:** Monorepo packages not installed or built.

**Solution:**
```bash
# Clean install everything
npm run update

# Or manually
rm -rf node_modules package-lock.json
npm install
cd packages/data-provider && npm install && npm run build
cd ../data-schemas && npm install && npm run build
cd ../api && npm install && npm run build
```

---

### "EADDRINUSE: address already in use :::3080"

**Cause:** Port 3080 is already in use.

**Solution:**
```bash
# Find what's using port 3080
lsof -i :3080  # Mac/Linux
netstat -ano | findstr :3080  # Windows

# Kill the process
kill -9 <PID>  # Mac/Linux
taskkill /PID <PID> /F  # Windows

# Or change port in .env
PORT=3081
```

---

### "Cannot connect to MongoDB"

**Cause:** MongoDB not running.

**Solution:**
```bash
# Docker
docker compose up -d mongodb

# Mac (Homebrew)
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Verify
mongosh
# Should connect without error
```

---

### "401 Unauthorized" when calling API

**Cause:** JWT token missing or invalid.

**Check:**
1. Are you logged in?
2. Is token in request headers?
3. Is `JWT_SECRET` consistent (same in .env)?
4. Has token expired?

**Solution:**
```javascript
// Check if token is being sent (in React)
console.log('Token:', localStorage.getItem('token'))

// Check backend logs for JWT errors
// Look for: "JWT invalid" or "Token expired"

// Clear cookies and re-login
localStorage.clear()
// Login again
```

---

### Vite build fails with "out of memory"

**Cause:** Large build, not enough RAM allocated.

**Solution:**
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run build

# Or in package.json:
"build": "NODE_OPTIONS=--max-old-space-size=4096 vite build"
```

---

## Contributing

### How can I contribute as a beginner?

**Great ways to start:**

1. **Documentation**
   - Fix typos
   - Improve clarity
   - Add examples
   - Translate to other languages

2. **Small bug fixes**
   - Look for issues tagged "good first issue"
   - Fix broken links
   - Update dependencies
   - Improve error messages

3. **Tests**
   - Add missing tests
   - Improve test coverage
   - Add E2E tests for features

4. **UI/UX improvements**
   - Fix accessibility issues
   - Improve mobile responsiveness
   - Better error states
   - Loading indicators

**🎯 Start here:** [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

---

### What makes a good pull request?

**✅ Good PR checklist:**

- [ ] **Clear title** - `feat: add dark mode toggle`
- [ ] **Description** - What does it do? Why?
- [ ] **Small & focused** - One feature/fix per PR
- [ ] **Tests** - Add/update tests
- [ ] **No breaking changes** - Or clearly document them
- [ ] **Follows conventions** - Code style matches project
- [ ] **Passes CI** - All tests pass
- [ ] **Screenshots** - If UI changes
- [ ] **Linked issue** - "Fixes #123"

**❌ Avoid:**
- Massive PRs (500+ lines changed)
- Multiple unrelated changes
- No description
- Breaking changes without discussion
- Failing tests
- Reformatting entire files (make formatting a separate PR)

---

### How do I know if my code follows best practices?

**Run linters:**
```bash
# Check code style
npm run lint

# Auto-fix issues
npm run lint:fix

# Format code
npm run format
```

**Check this documentation:**
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
- [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)

**Get code review:**
- Open a PR (even as draft)
- Ask for feedback
- Learn from maintainers' comments

---

## Still Have Questions?

### Documentation
- [Learning Path Home](./README.md)
- [Getting Started](./GETTING_STARTED.md)
- [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
- [All Docs](./README.md#documentation-index)

### Community
- **Discord:** [discord.librechat.ai](https://discord.librechat.ai)
- **GitHub Discussions:** [github.com/danny-avila/LibreChat/discussions](https://github.com/danny-avila/LibreChat/discussions)
- **GitHub Issues:** [github.com/danny-avila/LibreChat/issues](https://github.com/danny-avila/LibreChat/issues)

### Resources
- **Official Docs:** [docs.librechat.ai](https://docs.librechat.ai)
- **YouTube:** [youtube.com/@LibreChat](https://www.youtube.com/@LibreChat)
- **Website:** [librechat.ai](https://librechat.ai)

---

**Can't find your question?**
1. Search this doc (Ctrl+F / Cmd+F)
2. Check other learning docs
3. Ask in Discord!

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
