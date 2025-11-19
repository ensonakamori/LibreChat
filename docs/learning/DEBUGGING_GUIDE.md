# 🐛 Debugging Guide

**Documented:** November 19, 2025
**Target:** Developers debugging LibreChat issues
**Time Estimate:** Reference guide (use as needed)
**Difficulty:** 🟡 Intermediate

---

## Debug Like a Pro

Effective debugging saves hours of frustration. This guide covers tools and techniques for debugging frontend, backend, and full-stack issues.

---

## Table of Contents

- [Debugging Mindset](#debugging-mindset)
- [Frontend Debugging](#frontend-debugging)
- [Backend Debugging](#backend-debugging)
- [Database Debugging](#database-debugging)
- [Network Debugging](#network-debugging)
- [Common Issues](#common-issues)
- [Performance Debugging](#performance-debugging)
- [Production Debugging](#production-debugging)

---

## Debugging Mindset

### Scientific Method

1. **Observe:** What is the actual behavior?
2. **Hypothesize:** What might be causing it?
3. **Test:** Can you reproduce it?
4. **Experiment:** Change one thing at a time
5. **Verify:** Did it fix the problem?

### Questions to Ask

- Can you reproduce it consistently?
- What changed recently?
- Does it work in development but not production?
- Does it happen on all browsers/devices?
- What does the error message say exactly?

---

## Frontend Debugging

### Browser DevTools

**Chrome DevTools (Cmd+Option+I / Ctrl+Shift+I)**

**Console Tab:**
```javascript
// Log variables
console.log('User:', user);

// Log with labels
console.log('Before:', value);
console.log('After:', newValue);

// Table format
console.table(messages);

// Group related logs
console.group('API Call');
console.log('Request:', params);
console.log('Response:', data);
console.groupEnd();

// Warnings and errors
console.warn('This is deprecated');
console.error('Something went wrong');

// Timing
console.time('fetchMessages');
await fetchMessages();
console.timeEnd('fetchMessages'); // Outputs: fetchMessages: 245ms
```

**Sources Tab (Breakpoints):**
1. Open DevTools → Sources
2. Find file (Cmd+P)
3. Click line number to set breakpoint
4. Trigger code
5. Inspect variables in scope

**Conditional Breakpoints:**
```javascript
// Right-click line number → Add conditional breakpoint
// Condition: userId === 'specific-id'
```

**Debugger Statement:**
```typescript
function handleSubmit() {
  debugger; // Execution pauses here
  const data = processForm();
  return data;
}
```

### React DevTools

**Install:** [Chrome Extension](https://chrome.google.com/webstore/detail/react-developer-tools)

**Features:**
- Inspect component tree
- View props and state
- Track which components re-rendered
- Profiler for performance

**Usage:**
1. Open DevTools → React tab
2. Click component in tree
3. View props, state, hooks in right panel
4. Edit values to test changes

**Profiler:**
1. DevTools → Profiler tab
2. Click record
3. Interact with app
4. Stop recording
5. See which components rendered and how long

### TanStack Query DevTools

**Already included in LibreChat:**

```typescript
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

**Features:**
- View all queries and their state
- See cached data
- Manually refetch queries
- Clear cache

### Common Frontend Issues

**Issue: Component not re-rendering**

```typescript
// Debug
useEffect(() => {
  console.log('Component rendered with:', { prop1, prop2, state });
});

// Check:
// - Is state actually changing?
// - Are you mutating state directly? (use spread operator)
// - Is React.memo preventing re-render?
```

**Issue: Infinite re-renders**

```typescript
// ❌ Bad: Creates new object every render
function Component() {
  const config = { option: true }; // New reference every time!
  useEffect(() => {
    doSomething(config);
  }, [config]); // Runs infinitely
}

// ✅ Good: Stable reference
function Component() {
  const config = useMemo(() => ({ option: true }), []);
  useEffect(() => {
    doSomething(config);
  }, [config]);
}
```

**Issue: State not updating**

```typescript
// Debug
const [count, setCount] = useState(0);

function increment() {
  console.log('Before:', count);
  setCount(count + 1);
  console.log('After (still old!):', count); // State updates are async!

  // Use callback to see new value
  setCount(prev => {
    console.log('New value:', prev + 1);
    return prev + 1;
  });
}
```

---

## Backend Debugging

### Node.js Debugger

**Start with debugger:**
```bash
node --inspect api/server/index.js
# or with nodemon
nodemon --inspect api/server/index.js
```

**Chrome DevTools:**
1. Open Chrome: `chrome://inspect`
2. Click "inspect" under your app
3. Set breakpoints in Sources tab
4. Trigger API request
5. Inspect variables

**VS Code Debugger:**

**File:** `.vscode/launch.json`
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "program": "${workspaceFolder}/api/server/index.js",
      "skipFiles": ["<node_internals>/**"],
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

**Usage:**
1. Set breakpoints in VS Code
2. Press F5 or Debug → Start Debugging
3. Trigger request
4. Inspect variables in Debug panel

### Logging Best Practices

**Use Winston logger (not console.log):**

```javascript
const logger = require('./utils/logger');

// Different levels
logger.error('Payment failed', { userId, amount, error: err.message });
logger.warn('API rate limit approaching', { count: 95 });
logger.info('User logged in', { userId, email });
logger.debug('Cache hit', { key, ttl });

// Structured logging (easy to search)
logger.info('Message created', {
  messageId: message._id,
  conversationId,
  userId,
  model,
  duration: Date.now() - startTime
});
```

**Log levels:**
- `error`: Something failed
- `warn`: Something concerning
- `info`: Normal operations
- `debug`: Detailed debugging info

### Request Logging Middleware

```javascript
function requestLogger(req, res, next) {
  const start = Date.now();

  res.on('finish', () => {
    logger.http(`${req.method} ${req.path}`, {
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration: `${Date.now() - start}ms`,
      userId: req.user?.id
    });
  });

  next();
}

app.use(requestLogger);
```

### Common Backend Issues

**Issue: Undefined property error**

```javascript
// ❌ Crashes if user is null
const email = req.user.email; // TypeError: Cannot read property 'email' of undefined

// ✅ Safe
const email = req.user?.email;
// or
if (!req.user) {
  throw new UnauthorizedError('User not authenticated');
}
const email = req.user.email;

// Debug
console.log('req.user:', req.user);
console.log('Type:', typeof req.user);
console.log('Keys:', Object.keys(req.user || {}));
```

**Issue: Async function not awaited**

```javascript
// ❌ Bad: Promise not awaited
async function handler(req, res) {
  const data = getData(); // Returns Promise!
  res.json({ data }); // Sends Promise object, not data
}

// ✅ Good
async function handler(req, res) {
  const data = await getData();
  res.json({ data });
}

// Debug: Check if value is a Promise
console.log('Is Promise?', data instanceof Promise);
```

**Issue: Middleware not calling next()**

```javascript
// ❌ Bad: Middleware doesn't call next()
function myMiddleware(req, res, next) {
  if (condition) {
    return res.json({ ok: true });
    // next() not called! Request hangs
  }
  // Also doesn't call next()
}

// ✅ Good
function myMiddleware(req, res, next) {
  if (condition) {
    return res.json({ ok: true }); // Response sent, done
  }
  next(); // Continue to next middleware
}
```

---

## Database Debugging

### Mongoose Debug Mode

```javascript
// Enable query logging
mongoose.set('debug', true);

// Custom logger
mongoose.set('debug', (collection, method, query, doc) => {
  logger.debug('Mongoose query', {
    collection,
    method,
    query: JSON.stringify(query),
    doc: JSON.stringify(doc)
  });
});

// Output:
// messages.find { conversationId: ObjectId("...") }
// messages.findOne { _id: ObjectId("...") }
```

### Query Performance

```javascript
// Explain query
const explain = await Message
  .find({ conversationId: '123' })
  .explain('executionStats');

console.log('Execution time:', explain.executionStats.executionTimeMillis);
console.log('Documents examined:', explain.executionStats.totalDocsExamined);
console.log('Documents returned:', explain.executionStats.nReturned);

// If totalDocsExamined >> nReturned, you need an index!
```

### Common Database Issues

**Issue: Query returns empty array**

```javascript
// Debug
const result = await Message.find({ conversationId: '123' });
console.log('Query result:', result);
console.log('Length:', result.length);

// Check:
// - Is conversationId correct?
// - Is it a string or ObjectId?
const result = await Message.find({
  conversationId: new mongoose.Types.ObjectId('123')
});

// - Are there actually documents in the collection?
const count = await Message.countDocuments({});
console.log('Total messages:', count);
```

**Issue: Document not saving**

```javascript
// Debug
try {
  const message = await Message.create({ /* data */ });
  console.log('Saved:', message);
} catch (error) {
  console.error('Validation error:', error.errors);
  // Shows which fields failed validation
}

// Check:
// - Are all required fields provided?
// - Are values correct type?
// - Are enums valid?
```

---

## Network Debugging

### Browser Network Tab

**Chrome DevTools → Network:**
1. Open DevTools → Network tab
2. Trigger request
3. Click request in list
4. View:
   - **Headers:** Request/response headers
   - **Payload:** Request body
   - **Preview:** Response data
   - **Timing:** How long each phase took

**Check for:**
- Is request being sent?
- What status code? (200, 400, 401, 500?)
- Is request body correct?
- Is auth token included?
- What error message in response?

### cURL Commands

**Test API directly:**

```bash
# GET request
curl http://localhost:3080/api/conversations \
  -H "Authorization: Bearer YOUR_TOKEN"

# POST request
curl -X POST http://localhost:3080/api/messages \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"text": "Hello", "conversationId": "123"}'

# See full request/response
curl -v http://localhost:3080/api/conversations

# Save response to file
curl http://localhost:3080/api/conversations > response.json
```

### Proxy Debugging (Charles/Fiddler)

**For debugging mobile apps or seeing all network traffic:**

1. Install Charles Proxy or Fiddler
2. Configure browser/app to use proxy
3. See all HTTP/HTTPS requests
4. Inspect/modify requests before sending
5. Inspect responses

---

## Common Issues

### CORS Errors

**Error:**
```
Access to fetch at 'http://localhost:3080/api/messages' from origin 'http://localhost:3000' 
has been blocked by CORS policy
```

**Fix:**
```javascript
// api/server/index.js
const cors = require('cors');

app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true
}));
```

**Debug:**
```bash
# Check CORS headers
curl -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  --verbose \
  http://localhost:3080/api/messages
```

### 401 Unauthorized

**Debug:**
```typescript
// Frontend
const token = localStorage.getItem('authToken');
console.log('Token exists?', !!token);
console.log('Token:', token?.substring(0, 20) + '...');

// Decode JWT to see if expired
const decoded = JSON.parse(atob(token.split('.')[1]));
console.log('Expires:', new Date(decoded.exp * 1000));
console.log('Is expired?', Date.now() > decoded.exp * 1000);

// Backend
console.log('Authorization header:', req.headers.authorization);
console.log('User from token:', req.user);
```

### Memory Leaks

**Find leaks:**
```bash
# Start with heap profiling
node --inspect --expose-gc api/server/index.js

# Chrome DevTools → Memory tab → Take heap snapshot
# Perform action
# Take another snapshot
# Compare to find leaked objects
```

**Common causes:**
- Global variables
- Event listeners not removed
- Timers not cleared
- Closures holding references

---

## Performance Debugging

### React Profiler

```typescript
import { Profiler } from 'react';

function onRenderCallback(
  id, // Component id
  phase, // "mount" or "update"
  actualDuration, // Time spent rendering
  baseDuration, // Estimated time without memoization
  startTime, // When rendering started
  commitTime, // When React committed changes
) {
  console.log(`${id} took ${actualDuration}ms to render`);
}

<Profiler id="Chat" onRender={onRenderCallback}>
  <Chat />
</Profiler>
```

### Lighthouse

**Chrome DevTools → Lighthouse:**
1. Run audit
2. See performance score
3. View opportunities and diagnostics
4. Fix issues (large images, unused JS, etc.)

### Backend Performance

```javascript
// Measure function execution time
async function expensiveOperation() {
  const start = Date.now();

  const result = await doSomething();

  const duration = Date.now() - start;
  logger.info('Operation completed', { duration, result });

  return result;
}

// Find slow queries
mongoose.set('debug', (collection, method, query, doc, options) => {
  const start = Date.now();

  // Log query time
  process.nextTick(() => {
    const duration = Date.now() - start;
    if (duration > 100) { // Slow query threshold
      logger.warn('Slow query', {
        collection,
        method,
        query,
        duration
      });
    }
  });
});
```

---

## Production Debugging

### Error Tracking

**Use Sentry or similar:**

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({ dsn: process.env.SENTRY_DSN });

// Errors automatically reported
app.use(Sentry.Handlers.errorHandler());
```

### Logging

**Use log aggregation (e.g., Logtail, Papertrail):**

```javascript
const winston = require('winston');
const { Logtail } = require('@logtail/node');

const logtail = new Logtail(process.env.LOGTAIL_TOKEN);

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    logtail.transport
  ]
});

// All logs sent to Logtail for searching
logger.error('Payment failed', { userId, amount });
```

### Feature Flags

**Debug production issues safely:**

```typescript
const featureFlags = {
  newFeature: process.env.ENABLE_NEW_FEATURE === 'true'
};

if (featureFlags.newFeature) {
  // New code path
} else {
  // Old code path (safe fallback)
}
```

---

## Summary

**Debugging Tools:**

**Frontend:**
- Browser DevTools (Console, Sources, Network)
- React DevTools
- TanStack Query DevTools

**Backend:**
- Node.js debugger (--inspect)
- Winston logger
- Mongoose debug mode

**Network:**
- Network tab
- cURL
- Proxy tools (Charles, Fiddler)

**Best Practices:**
- Reproduce the issue
- Isolate the problem
- Use breakpoints and logging
- Check one thing at a time
- Document the solution

---

**Next Steps:**

- Read [SECURITY_GUIDE.md](./SECURITY_GUIDE.md) - Security best practices
- Practice debugging with breakpoints
- Set up proper logging in your code

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
