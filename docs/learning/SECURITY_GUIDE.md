# 🔒 Security Guide

**Documented:** November 19, 2025
**Target:** Contributors implementing secure features
**Time Estimate:** 3-4 hours to fully understand
**Difficulty:** 🟡 Intermediate

---

## Security First

Building secure applications protects users and prevents vulnerabilities. This guide covers authentication, authorization, input validation, and security best practices for LibreChat.

---

## Table of Contents

- [Security Principles](#security-principles)
- [Authentication](#authentication)
- [Authorization](#authorization)
- [Input Validation](#input-validation)
- [SQL/NoSQL Injection Prevention](#sqlnosql-injection-prevention)
- [XSS Prevention](#xss-prevention)
- [CSRF Protection](#csrf-protection)
- [Rate Limiting](#rate-limiting)
- [Secure Password Handling](#secure-password-handling)
- [Environment Variables](#environment-variables)
- [HTTPS and TLS](#https-and-tls)
- [Security Headers](#security-headers)
- [Common Vulnerabilities](#common-vulnerabilities)

---

## Security Principles

### Defense in Depth

**Multiple layers of security:**
- Frontend validation
- Backend validation
- Database constraints
- Network security

### Principle of Least Privilege

**Give minimum necessary access:**
- Users only see their data
- API keys have minimal permissions
- Database users have restricted access

### Never Trust User Input

**Validate everything:**
- URL parameters
- Request body
- Headers
- Cookies

---

## Authentication

### JWT (JSON Web Tokens)

**How LibreChat uses JWT:**

**1. User logs in:**
```javascript
// api/server/controllers/authController.js

async login(req, res) {
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

  // Generate JWT token
  const token = jwt.sign(
    { id: user._id, email: user.email }, // Payload (don't put sensitive data!)
    process.env.JWT_SECRET,              // Secret key
    { expiresIn: '7d' }                  // Expiration
  );

  res.json({
    user: { id: user._id, name: user.name, email: user.email },
    token
  });
}
```

**2. Client stores token:**
```typescript
// client/src/hooks/useAuth.ts

onSuccess: (data) => {
  // Store in localStorage
  localStorage.setItem('authToken', data.token);
}
```

**3. Client sends token with requests:**
```typescript
// client/src/utils/apiClient.ts

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken');

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

**4. Server verifies token:**
```javascript
// api/server/middleware/requireAuth.js

function requireAuth(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    throw new UnauthorizedError('No token provided');
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // Attach user to request
    next();
  } catch (error) {
    throw new UnauthorizedError('Invalid token');
  }
}
```

### JWT Security Best Practices

**✅ Do:**
- Use strong secret (32+ random characters)
- Set expiration time
- Store in httpOnly cookies (more secure than localStorage)
- Implement token refresh
- Sign with RS256 for production

**❌ Don't:**
- Put sensitive data in payload (it's not encrypted!)
- Use weak secret
- Set no expiration
- Share secret publicly
- Store in localStorage if possible (XSS vulnerable)

**Better approach: httpOnly cookies:**
```javascript
// Backend: Send token as cookie
res.cookie('authToken', token, {
  httpOnly: true,      // Not accessible via JavaScript
  secure: true,        // Only sent over HTTPS
  sameSite: 'strict',  // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});

// Frontend: Cookie sent automatically, no localStorage needed
```

---

## Authorization

**Authentication:** "Who are you?"
**Authorization:** "What can you do?"

### Role-Based Access Control (RBAC)

```javascript
// User model
const userSchema = new Schema({
  email: String,
  password: String,
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  }
});

// Middleware: Require admin role
function requireAdmin(req, res, next) {
  if (req.user.role !== 'admin') {
    throw new ForbiddenError('Admin access required');
  }
  next();
}

// Route: Admin only
router.delete('/api/users/:id', requireAuth, requireAdmin, usersController.delete);
```

### Resource Ownership

**Always verify user owns resource:**

```javascript
async function deleteConversation(req, res) {
  const { id } = req.params;
  const userId = req.user.id;

  const conversation = await Conversation.findById(id);

  if (!conversation) {
    throw new NotFoundError('Conversation not found');
  }

  // ✅ Verify ownership
  if (conversation.userId.toString() !== userId) {
    throw new ForbiddenError('Not your conversation');
  }

  await conversation.deleteOne();

  res.json({ success: true });
}
```

**Never trust client-provided IDs:**

```javascript
// ❌ VULNERABLE: Client provides userId
async function getProfile(req, res) {
  const { userId } = req.body; // Attacker can change this!
  const user = await User.findById(userId);
  res.json({ user });
}

// ✅ SECURE: Use authenticated user ID
async function getProfile(req, res) {
  const userId = req.user.id; // From verified JWT token
  const user = await User.findById(userId);
  res.json({ user });
}
```

---

## Input Validation

### Server-Side Validation (Required!)

**Never trust frontend validation alone:**

```javascript
const { z } = require('zod');

// Define schema
const createMessageSchema = z.object({
  text: z.string().min(1).max(10000),
  conversationId: z.string().regex(/^[0-9a-fA-F]{24}$/), // MongoDB ObjectId
  model: z.enum(['gpt-4', 'gpt-3.5-turbo', 'claude-3', 'gemini-pro'])
});

// Validation middleware
function validateRequest(schema) {
  return (req, res, next) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      throw new ValidationError('Invalid input', error.errors);
    }
  };
}

// Use in route
router.post('/api/messages',
  requireAuth,
  validateRequest(createMessageSchema),
  messagesController.create
);
```

### Sanitize Input

```javascript
const validator = require('validator');

// Email validation
if (!validator.isEmail(email)) {
  throw new ValidationError('Invalid email format');
}

// URL validation
if (!validator.isURL(avatarUrl)) {
  throw new ValidationError('Invalid URL');
}

// Escape HTML
const sanitizedText = validator.escape(userInput);
```

---

## SQL/NoSQL Injection Prevention

### NoSQL Injection

**Vulnerable code:**
```javascript
// ❌ VULNERABLE
const user = await User.findOne({
  email: req.body.email,
  password: req.body.password // Direct user input!
});

// Attacker sends:
// { "email": "admin@example.com", "password": { "$ne": null } }
// Query becomes: find({ email: "admin@example.com", password: { $ne: null } })
// Returns user without checking password!
```

**Secure code:**
```javascript
// ✅ SECURE: Validate input
const { email, password } = createMessageSchema.parse(req.body);

const user = await User.findOne({ email }).select('+password');

if (!user) {
  throw new UnauthorizedError('Invalid credentials');
}

// Compare password securely
const isMatch = await bcrypt.compare(password, user.password);

if (!isMatch) {
  throw new UnauthorizedError('Invalid credentials');
}
```

### Parameterized Queries

```javascript
// ✅ Good: Mongoose automatically escapes
const messages = await Message.find({ conversationId: id });

// ❌ Bad: Raw query with user input
db.collection('messages').find({ conversationId: req.params.id }); // If id is malicious object
```

---

## XSS Prevention

**Cross-Site Scripting:** Injecting malicious scripts into pages

### React's Built-in Protection

**React escapes content by default:**
```typescript
// ✅ Safe: React escapes HTML
<div>{userInput}</div>

// User input: "<script>alert('XSS')</script>"
// Rendered as: &lt;script&gt;alert('XSS')&lt;/script&gt;
```

### Dangerous Patterns

```typescript
// ❌ DANGEROUS: Renders raw HTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// User input: "<img src=x onerror='alert(1)'>"
// Script executes!

// ✅ If you must use HTML, sanitize first
import DOMPurify from 'dompurify';

const sanitized = DOMPurify.sanitize(userInput);
<div dangerouslySetInnerHTML={{ __html: sanitized }} />
```

### Content Security Policy (CSP)

**Prevent inline scripts:**
```javascript
const helmet = require('helmet');

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"], // Only allow scripts from same origin
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.openai.com']
    }
  })
);
```

---

## CSRF Protection

**Cross-Site Request Forgery:** Tricking user into making unwanted requests

### SameSite Cookies

```javascript
res.cookie('authToken', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict' // Prevents CSRF attacks
});
```

### CSRF Tokens

```javascript
const csrf = require('csurf');

app.use(csrf({ cookie: true }));

// Send token to client
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Verify token on state-changing requests
// Middleware automatically checks for _csrf parameter or X-CSRF-Token header
```

---

## Rate Limiting

**Prevent abuse and brute-force attacks:**

```javascript
const rateLimit = require('express-rate-limit');

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per window
  message: 'Too many requests, please try again later'
});

app.use('/api/', apiLimiter);

// Stricter limit for login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts
  skipSuccessfulRequests: true // Don't count successful logins
});

router.post('/api/auth/login', loginLimiter, authController.login);
```

---

## Secure Password Handling

### Never Store Plain Passwords

```javascript
// ❌ NEVER
const user = await User.create({
  email,
  password: password // Plain text! NEVER DO THIS
});

// ✅ Always hash with bcrypt
const bcrypt = require('bcryptjs');

const salt = await bcrypt.genSalt(10); // Higher is more secure but slower
const passwordHash = await bcrypt.hash(password, salt);

const user = await User.create({
  email,
  password: passwordHash // Hashed password
});
```

### Password Requirements

```javascript
const passwordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Password must contain uppercase letter')
  .regex(/[a-z]/, 'Password must contain lowercase letter')
  .regex(/[0-9]/, 'Password must contain number')
  .regex(/[^A-Za-z0-9]/, 'Password must contain special character');
```

### Password Reset

**Secure flow:**

1. User requests reset
2. Generate random token
3. Send token via email (expires in 1 hour)
4. User clicks link with token
5. Verify token not expired
6. Allow password change
7. Invalidate token

```javascript
// Generate reset token
const crypto = require('crypto');
const resetToken = crypto.randomBytes(32).toString('hex');
const resetTokenHash = crypto.createHash('sha256').update(resetToken).digest('hex');

user.passwordResetToken = resetTokenHash;
user.passwordResetExpires = Date.now() + 60 * 60 * 1000; // 1 hour
await user.save();

// Send email with: /reset-password?token=${resetToken}

// Verify token
const tokenHash = crypto.createHash('sha256').update(req.query.token).digest('hex');
const user = await User.findOne({
  passwordResetToken: tokenHash,
  passwordResetExpires: { $gt: Date.now() }
});

if (!user) {
  throw new Error('Invalid or expired token');
}
```

---

## Environment Variables

**Never commit secrets to Git:**

```bash
# .env (gitignored!)
JWT_SECRET=your-super-secret-key-min-32-chars
DATABASE_URL=mongodb://localhost:27017/librechat
OPENAI_API_KEY=sk-...

# .env.example (committed, no secrets)
JWT_SECRET=your-secret-here
DATABASE_URL=mongodb://localhost:27017/librechat
OPENAI_API_KEY=
```

**Use in code:**
```javascript
const jwtSecret = process.env.JWT_SECRET;

if (!jwtSecret) {
  throw new Error('JWT_SECRET environment variable not set');
}
```

**Generate secure secrets:**
```bash
# Generate 32-byte secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---

## HTTPS and TLS

**Always use HTTPS in production:**

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem')
};

https.createServer(options, app).listen(443);
```

**Redirect HTTP to HTTPS:**
```javascript
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
  res.end();
}).listen(80);
```

---

## Security Headers

**Use Helmet to set secure headers:**

```javascript
const helmet = require('helmet');

app.use(helmet()); // Enables all helmet protections

// Or configure individually:
app.use(helmet.hidePoweredBy()); // Hide X-Powered-By header
app.use(helmet.hsts({            // Force HTTPS
  maxAge: 31536000,
  includeSubDomains: true
}));
app.use(helmet.noSniff());       // Prevent MIME sniffing
app.use(helmet.xssFilter());     // XSS filter
app.use(helmet.frameguard({      // Clickjacking protection
  action: 'deny'
}));
```

---

## Common Vulnerabilities

### OWASP Top 10

1. **Broken Access Control**
   - ✅ Verify resource ownership
   - ✅ Use role-based access control

2. **Cryptographic Failures**
   - ✅ Use HTTPS
   - ✅ Hash passwords with bcrypt
   - ✅ Secure environment variables

3. **Injection**
   - ✅ Validate input
   - ✅ Use parameterized queries
   - ✅ Sanitize data

4. **Insecure Design**
   - ✅ Threat modeling
   - ✅ Security by design

5. **Security Misconfiguration**
   - ✅ Use secure defaults
   - ✅ Security headers (Helmet)
   - ✅ Disable unnecessary features

6. **Vulnerable Components**
   - ✅ Keep dependencies updated
   - ✅ Run `npm audit`

7. **Authentication Failures**
   - ✅ Strong password policy
   - ✅ Rate limiting
   - ✅ Secure session management

8. **Data Integrity Failures**
   - ✅ Verify data from untrusted sources
   - ✅ Use CSP

9. **Logging Failures**
   - ✅ Log security events
   - ✅ Monitor logs

10. **Server-Side Request Forgery (SSRF)**
    - ✅ Validate URLs
    - ✅ Whitelist allowed domains

---

## Security Checklist

**Before deploying:**

- [ ] All secrets in environment variables (not code)
- [ ] HTTPS enabled
- [ ] Security headers configured (Helmet)
- [ ] Input validation on all endpoints
- [ ] Rate limiting enabled
- [ ] Passwords hashed with bcrypt
- [ ] JWT tokens expire
- [ ] CSRF protection enabled
- [ ] SQL/NoSQL injection prevented
- [ ] XSS protection enabled
- [ ] Dependencies updated (`npm audit`)
- [ ] Logging configured
- [ ] Error messages don't leak sensitive info
- [ ] File uploads validated (type, size)
- [ ] CORS configured correctly

---

## Summary

**Security Best Practices:**

**Authentication:**
- Use JWT with strong secret
- Set expiration
- Store in httpOnly cookies

**Authorization:**
- Verify resource ownership
- Use role-based access control
- Never trust client-provided IDs

**Input Validation:**
- Validate on server
- Use Zod schemas
- Sanitize user input

**Common Vulnerabilities:**
- NoSQL injection: Validate input
- XSS: Use React's escaping, CSP
- CSRF: SameSite cookies
- Rate limit sensitive endpoints
- Hash passwords with bcrypt

---

**Next Steps:**

- Read [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) - API reference
- Review security of your code
- Run security audit: `npm audit`

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
