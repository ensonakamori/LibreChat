# 📏 Patterns & Conventions

**Documented:** November 19, 2025
**Target:** Contributors learning LibreChat code standards
**Time Estimate:** 3-4 hours to fully understand
**Difficulty:** 🟢 Beginner

---

## Welcome to Code Standards!

Consistent code makes collaboration easier. This guide documents LibreChat's patterns, conventions, and best practices so you can write code that fits naturally into the project.

---

## Table of Contents

- [Code Style & Formatting](#code-style--formatting)
- [Naming Conventions](#naming-conventions)
- [File Organization](#file-organization)
- [Import Patterns](#import-patterns)
- [React Component Patterns](#react-component-patterns)
- [Backend Patterns](#backend-patterns)
- [TypeScript Patterns](#typescript-patterns)
- [Git Commit Conventions](#git-commit-conventions)
- [Error Handling Patterns](#error-handling-patterns)
- [Testing Patterns](#testing-patterns)
- [Documentation Patterns](#documentation-patterns)
- [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)

---

## Code Style & Formatting

### ESLint & Prettier Configuration

LibreChat uses ESLint for code quality and Prettier for formatting.

**Run before committing:**

```bash
# Check for lint errors
npm run lint

# Auto-fix lint errors
npm run lint:fix

# Format all files
npm run format

# Check formatting
npm run format:check
```

### Code Style Rules

**Indentation:**
- Use **2 spaces** (not tabs)
- Configured in `.prettierrc`

**Line Length:**
- Maximum **100 characters** per line
- Break long lines logically

**Semicolons:**
- Always use semicolons
- Configured in ESLint/Prettier

**Quotes:**
- Use **single quotes** for strings: `'hello'`
- Use **double quotes** in JSX: `<div className="foo">`

**Trailing Commas:**
- Use trailing commas in multiline objects/arrays

```javascript
// ✅ Good
const obj = {
  name: 'John',
  email: 'john@example.com', // Trailing comma
};

const arr = [
  'item1',
  'item2', // Trailing comma
];

// ❌ Bad
const obj = {
  name: 'John',
  email: 'john@example.com' // No trailing comma
};
```

**Arrow Functions:**
- Omit parentheses for single parameter
- Use implicit return when possible

```javascript
// ✅ Good
const double = n => n * 2;
const add = (a, b) => a + b;

// ❌ Bad
const double = (n) => { return n * 2; };
```

---

## Naming Conventions

### Files and Directories

**React Components:**
- PascalCase: `ChatInput.tsx`, `MessageList.tsx`
- Match component name

```
components/
  Chat/
    ChatInput.tsx      ← Component file
    MessageList.tsx
    index.ts           ← Barrel export
```

**Hooks:**
- camelCase with `use` prefix: `useMessages.ts`, `useAuth.ts`

```
hooks/
  useMessages.ts
  useAuth.ts
  useConversations.ts
```

**Utils/Helpers:**
- camelCase: `apiClient.ts`, `formatDate.ts`

```
utils/
  apiClient.ts
  formatDate.ts
  validators.ts
```

**Constants:**
- camelCase file, UPPER_CASE exports

```typescript
// constants/apiConfig.ts
export const API_BASE_URL = 'http://localhost:3080';
export const API_TIMEOUT = 30000;
export const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
```

**Backend Files:**
- camelCase for services/controllers
- PascalCase for models (match MongoDB collection)

```
api/
  models/
    User.js           ← PascalCase (model name)
    Conversation.js
  controllers/
    authController.js    ← camelCase
    messagesController.js
  services/
    messageService.js
    userService.js
```

### Variables and Functions

**Variables:**
- camelCase: `userName`, `conversationId`, `isLoading`

```typescript
const userName = 'John';
const conversationId = '123';
const isLoading = false;
```

**Booleans:**
- Use `is`, `has`, `should` prefix

```typescript
// ✅ Good
const isLoading = true;
const hasMessages = messages.length > 0;
const shouldRender = isVisible && !isLoading;

// ❌ Bad
const loading = true;
const messages = messages.length > 0; // Confusing!
```

**Functions:**
- camelCase: `fetchMessages`, `handleSubmit`
- Use verb prefix: `get`, `set`, `handle`, `fetch`, `create`, `update`, `delete`

```typescript
// ✅ Good
function fetchMessages() { }
function handleClick() { }
function validateInput() { }
function createConversation() { }

// ❌ Bad
function messages() { }  // No verb
function click() { }     // No verb
```

**Event Handlers:**
- Prefix with `handle`: `handleClick`, `handleSubmit`, `handleChange`

```typescript
function ChatInput() {
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // ...
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleInputChange} />
    </form>
  );
}
```

**React Components:**
- PascalCase: `ChatInput`, `MessageList`, `UserAvatar`

```typescript
// ✅ Good
function ChatInput() { }
function MessageList() { }

// ❌ Bad
function chatInput() { }
function messageList() { }
```

**Custom Hooks:**
- camelCase with `use` prefix: `useMessages`, `useAuth`

```typescript
// ✅ Good
function useMessages() { }
function useAuth() { }

// ❌ Bad
function getMessages() { }    // Not a hook
function messages() { }        // Not a hook
function UseMessages() { }     // PascalCase is wrong
```

**Types/Interfaces:**
- PascalCase: `User`, `Message`, `Conversation`
- Props suffix for component props: `ChatInputProps`

```typescript
// ✅ Good
interface User {
  id: string;
  name: string;
}

interface ChatInputProps {
  conversationId: string;
  onSubmit: () => void;
}

// ❌ Bad
interface user { }        // lowercase
interface IUser { }       // Hungarian notation
interface chatInputProps { } // camelCase
```

**Enums:**
- PascalCase for enum name
- UPPER_CASE for values

```typescript
enum MessageRole {
  USER = 'USER',
  ASSISTANT = 'ASSISTANT',
  SYSTEM = 'SYSTEM'
}

enum ConversationStatus {
  ACTIVE = 'ACTIVE',
  ARCHIVED = 'ARCHIVED',
  DELETED = 'DELETED'
}
```

### Constants

**File-level constants:**
- UPPER_CASE with underscores

```typescript
const MAX_MESSAGE_LENGTH = 10000;
const DEFAULT_MODEL = 'gpt-4';
const API_TIMEOUT = 30000;
```

**Object constants:**
- PascalCase for object, UPPER_CASE for keys

```typescript
const ApiEndpoints = {
  LOGIN: '/api/auth/login',
  REGISTER: '/api/auth/register',
  MESSAGES: '/api/messages'
} as const;

const ModelNames = {
  GPT_4: 'gpt-4',
  CLAUDE_3: 'claude-3-sonnet',
  GEMINI: 'gemini-pro'
} as const;
```

---

## File Organization

### Frontend Directory Structure

```
client/src/
├── components/           # React components
│   ├── Chat/            # Feature-based grouping
│   │   ├── ChatInput.tsx
│   │   ├── MessageList.tsx
│   │   ├── Message.tsx
│   │   └── index.ts     # Barrel export
│   ├── Auth/
│   │   ├── Login.tsx
│   │   ├── Register.tsx
│   │   └── index.ts
│   └── shared/          # Reusable components
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Spinner.tsx
│
├── hooks/               # Custom hooks
│   ├── useMessages.ts
│   ├── useAuth.ts
│   └── useConversations.ts
│
├── utils/               # Utility functions
│   ├── apiClient.ts
│   ├── formatDate.ts
│   └── validators.ts
│
├── stores/              # State management (Jotai/Recoil)
│   ├── auth.ts
│   ├── conversation.ts
│   └── ui.ts
│
├── types/               # TypeScript types
│   ├── user.ts
│   ├── message.ts
│   └── conversation.ts
│
├── config/              # Configuration
│   ├── queryClient.ts
│   └── constants.ts
│
└── App.tsx              # Main app component
```

### Backend Directory Structure

```
api/
├── models/              # Mongoose models
│   ├── User.js
│   ├── Conversation.js
│   └── Message.js
│
├── server/
│   ├── routes/          # Route definitions
│   │   ├── auth.js
│   │   ├── messages.js
│   │   └── index.js     # Combines all routes
│   │
│   ├── controllers/     # Request handlers
│   │   ├── authController.js
│   │   ├── messagesController.js
│   │   └── conversationsController.js
│   │
│   ├── services/        # Business logic
│   │   ├── messageService.js
│   │   ├── userService.js
│   │   └── conversationService.js
│   │
│   ├── middleware/      # Express middleware
│   │   ├── requireAuth.js
│   │   ├── validateRequest.js
│   │   └── errorHandler.js
│   │
│   ├── utils/           # Utility functions
│   │   ├── logger.js
│   │   ├── jwt.js
│   │   └── errors.js
│   │
│   ├── config/          # Configuration
│   │   ├── database.js
│   │   ├── passport.js
│   │   └── redis.js
│   │
│   └── index.js         # Server entry point
```

### Barrel Exports

**Use `index.ts` to export multiple items from a directory:**

```typescript
// components/Chat/index.ts
export { ChatInput } from './ChatInput';
export { MessageList } from './MessageList';
export { Message } from './Message';

// Usage
import { ChatInput, MessageList, Message } from '~/components/Chat';
```

**When NOT to use barrel exports:**
- Large components (causes unnecessary re-renders)
- Components with side effects

---

## Import Patterns

### Import Order

**Order imports in this sequence:**

```typescript
// 1. External dependencies (React, libraries)
import { useState, useEffect } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { z } from 'zod';

// 2. Internal modules (utils, types, hooks)
import { formatDate } from '~/utils/formatDate';
import { User, Message } from '~/types';
import { useAuth } from '~/hooks/useAuth';

// 3. Components
import { Button } from '~/components/shared/Button';
import { ChatInput } from '~/components/Chat/ChatInput';

// 4. Styles (if using CSS modules)
import styles from './Component.module.css';

// 5. Types (if separate file)
import type { ComponentProps } from './types';
```

### Path Aliases

**Use `~` alias for `src/` directory:**

```typescript
// ✅ Good
import { apiClient } from '~/utils/apiClient';
import { ChatInput } from '~/components/Chat/ChatInput';

// ❌ Bad
import { apiClient } from '../../../utils/apiClient';
import { ChatInput } from '../../Chat/ChatInput';
```

**Configuration:** `tsconfig.json`

```json
{
  "compilerOptions": {
    "paths": {
      "~/*": ["./src/*"]
    }
  }
}
```

### Named vs Default Exports

**Prefer named exports:**

```typescript
// ✅ Good: Named export
export function ChatInput() {
  return <div>...</div>;
}

// Usage
import { ChatInput } from '~/components/Chat/ChatInput';

// ❌ Avoid: Default export
export default function ChatInput() {
  return <div>...</div>;
}

// Usage (can be renamed, confusing)
import Input from '~/components/Chat/ChatInput'; // Different name!
```

**Exception:** Use default exports for page components (React Router)

```typescript
// app/routes/chat.tsx
export default function ChatPage() {
  return <Chat />;
}
```

---

## React Component Patterns

### Component Structure

**Standard component structure:**

```typescript
// 1. Imports
import { useState } from 'react';
import { useMessages } from '~/hooks/useMessages';
import { Button } from '~/components/shared/Button';

// 2. Types
interface ChatInputProps {
  conversationId: string;
  onSubmit?: () => void;
}

// 3. Component
export function ChatInput({ conversationId, onSubmit }: ChatInputProps) {
  // 3a. Hooks (useState, useEffect, custom hooks)
  const [text, setText] = useState('');
  const createMessage = useCreateMessage();

  // 3b. Event handlers
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // ...
  };

  const handleChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    setText(e.target.value);
  };

  // 3c. Effects
  useEffect(() => {
    // ...
  }, []);

  // 3d. Render helpers (if complex)
  const renderCharacterCount = () => {
    return <span>{text.length} / 10000</span>;
  };

  // 3e. Return JSX
  return (
    <form onSubmit={handleSubmit}>
      <textarea value={text} onChange={handleChange} />
      {renderCharacterCount()}
      <Button type="submit">Send</Button>
    </form>
  );
}
```

### Props Destructuring

**Destructure props in function signature:**

```typescript
// ✅ Good
function ChatInput({ conversationId, onSubmit }: ChatInputProps) {
  return <div>{conversationId}</div>;
}

// ❌ Bad
function ChatInput(props: ChatInputProps) {
  return <div>{props.conversationId}</div>;
}
```

### Conditional Rendering

**Use early returns for loading/error states:**

```typescript
function ConversationList() {
  const { data, isLoading, error } = useConversations();

  // Early return for loading
  if (isLoading) {
    return <Spinner />;
  }

  // Early return for error
  if (error) {
    return <Error message={error.message} />;
  }

  // Early return for no data
  if (!data || data.length === 0) {
    return <EmptyState />;
  }

  // Main render
  return (
    <div>
      {data.map(conv => (
        <ConversationCard key={conv.id} conversation={conv} />
      ))}
    </div>
  );
}
```

**Inline conditionals for small toggles:**

```typescript
// Show/hide element
{isVisible && <Component />}

// Ternary for alternatives
{isLoading ? <Spinner /> : <Content />}

// Null check
{user?.name ?? 'Guest'}
```

### Key Props

**Always use unique, stable keys:**

```typescript
// ✅ Good: Use unique ID
{messages.map(msg => (
  <Message key={msg.id} {...msg} />
))}

// ❌ Bad: Using index (unstable on reorder)
{messages.map((msg, index) => (
  <Message key={index} {...msg} />
))}

// ❌ Bad: Using non-unique field
{messages.map(msg => (
  <Message key={msg.text} {...msg} />  // text might duplicate!
))}
```

### Component Composition

**Compose small, focused components:**

```typescript
// ✅ Good: Small, focused components
function ChatMessage({ message }: { message: Message }) {
  return (
    <div className="message">
      <MessageAvatar user={message.user} />
      <MessageContent text={message.text} />
      <MessageTimestamp time={message.createdAt} />
    </div>
  );
}

// ❌ Bad: One giant component
function ChatMessage({ message }: { message: Message }) {
  return (
    <div className="message">
      <div className="avatar">
        <img src={message.user.avatar} />
        <span>{message.user.name}</span>
        {message.user.isOnline && <span className="online-dot" />}
      </div>
      <div className="content">
        <p>{message.text}</p>
        {message.edited && <span>Edited</span>}
        {message.attachments.map(att => (
          <div key={att.id}>
            <img src={att.url} />
            <span>{att.filename}</span>
          </div>
        ))}
      </div>
      <div className="timestamp">
        {formatDistanceToNow(new Date(message.createdAt))}
      </div>
    </div>
  );
}
```

---

## Backend Patterns

### Controller Pattern

**Controllers handle HTTP, not business logic:**

```javascript
// ✅ Good: Thin controller
async function createMessage(req, res, next) {
  try {
    const { text, conversationId } = req.body;
    const userId = req.user.id;

    // Delegate to service
    const message = await messageService.createMessage({
      text,
      conversationId,
      userId
    });

    res.status(201).json({ message });

  } catch (error) {
    next(error);
  }
}

// ❌ Bad: Business logic in controller
async function createMessage(req, res, next) {
  try {
    // Too much logic here!
    const conversation = await Conversation.findById(req.body.conversationId);
    if (!conversation) {
      return res.status(404).json({ error: 'Not found' });
    }

    if (conversation.userId !== req.user.id) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    const userMessage = await Message.create({
      conversationId: req.body.conversationId,
      role: 'user',
      text: req.body.text
    });

    const aiResponse = await callOpenAI(req.body.text);

    const aiMessage = await Message.create({
      conversationId: req.body.conversationId,
      role: 'assistant',
      text: aiResponse
    });

    res.json({ message: aiMessage });

  } catch (error) {
    next(error);
  }
}
```

### Service Pattern

**Services contain reusable business logic:**

```javascript
// ✅ Good: Service with business logic
class MessageService {
  async createMessage({ text, conversationId, userId }) {
    // Verify ownership
    const conversation = await this.verifyOwnership(conversationId, userId);

    // Save user message
    const userMessage = await Message.create({
      conversationId,
      userId,
      role: 'user',
      text
    });

    // Get AI response
    const aiText = await this.getAIResponse(conversation, text);

    // Save AI message
    const aiMessage = await Message.create({
      conversationId,
      userId,
      role: 'assistant',
      text: aiText
    });

    return aiMessage;
  }

  async verifyOwnership(conversationId, userId) {
    const conversation = await Conversation.findById(conversationId);

    if (!conversation) {
      throw new NotFoundError('Conversation not found');
    }

    if (conversation.userId.toString() !== userId) {
      throw new ForbiddenError('Not your conversation');
    }

    return conversation;
  }

  async getAIResponse(conversation, text) {
    // ... AI logic
  }
}
```

### Error Handling

**Use custom error classes:**

```javascript
// Define custom errors
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.status = 404;
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.status = 400;
  }
}

// Throw in services
if (!user) {
  throw new NotFoundError('User not found');
}

if (!email.includes('@')) {
  throw new ValidationError('Invalid email');
}

// Catch in global error handler
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      status: err.status
    }
  });
});
```

---

## TypeScript Patterns

### Type Annotations

**Annotate function parameters and return types:**

```typescript
// ✅ Good: Explicit types
function formatDate(date: Date): string {
  return date.toLocaleDateString();
}

async function fetchMessages(conversationId: string): Promise<Message[]> {
  const { data } = await apiClient.get(`/api/messages/${conversationId}`);
  return data.messages;
}

// ❌ Bad: No type annotations
function formatDate(date) {  // Any type!
  return date.toLocaleDateString();
}
```

**Let TypeScript infer when obvious:**

```typescript
// ✅ Good: Let TypeScript infer
const count = 5;  // Inferred as number
const name = 'John';  // Inferred as string
const isActive = true;  // Inferred as boolean

// ❌ Bad: Unnecessary annotations
const count: number = 5;
const name: string = 'John';
```

### Interface vs Type

**Use interface for objects:**

```typescript
// ✅ Good: Interface for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

interface Message {
  id: string;
  text: string;
  userId: string;
}
```

**Use type for unions, primitives, utilities:**

```typescript
// ✅ Good: Type for unions
type MessageRole = 'user' | 'assistant' | 'system';
type Status = 'idle' | 'loading' | 'success' | 'error';

// ✅ Good: Type for mapped types
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

### Avoid `any`

```typescript
// ❌ Bad: Using any
function processData(data: any) {
  return data.map((item: any) => item.value);
}

// ✅ Good: Proper types
interface DataItem {
  value: number;
}

function processData(data: DataItem[]): number[] {
  return data.map(item => item.value);
}

// ✅ Good: Use unknown if type truly unknown
function processUnknown(data: unknown) {
  if (Array.isArray(data)) {
    return data.length;
  }
  return 0;
}
```

### Utility Types

```typescript
// Pick: Select subset of properties
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

type UserPublic = Pick<User, 'id' | 'name' | 'email'>;
// { id: string; name: string; email: string; }

// Omit: Exclude properties
type UserWithoutPassword = Omit<User, 'password'>;
// { id: string; name: string; email: string; }

// Partial: Make all properties optional
type PartialUser = Partial<User>;
// { id?: string; name?: string; email?: string; password?: string; }

// Required: Make all properties required
type RequiredUser = Required<PartialUser>;

// Record: Object with specific keys
type UserRoles = Record<string, 'admin' | 'user'>;
// { [key: string]: 'admin' | 'user' }
```

---

## Git Commit Conventions

### Commit Message Format

**Format:** `type(scope): subject`

```
feat(chat): add streaming message support
^--^ ^---^  ^-------------------------^
│    │      │
│    │      └─> Summary (imperative mood, lowercase)
│    └────────> Optional scope
└─────────────> Type
```

### Commit Types

```
feat:     New feature
fix:      Bug fix
docs:     Documentation changes
style:    Code style (formatting, semicolons, etc)
refactor: Code refactoring (no feat/fix)
test:     Add/update tests
chore:    Build, dependencies, tooling
perf:     Performance improvement
ci:       CI/CD changes
```

### Examples

**Good commit messages:**

```bash
feat(auth): add JWT token refresh
fix(chat): resolve message duplication bug
docs(readme): update installation instructions
refactor(api): extract message service layer
test(messages): add unit tests for validation
chore(deps): upgrade react to 18.3
perf(db): add index on conversationId field
```

**Multi-line commits:**

```bash
git commit -m "feat(chat): add file upload support

- Added file upload endpoint
- Integrated Sharp for image processing
- Updated Message model with fileIds field
- Added frontend file picker component

Closes #123"
```

**Bad commit messages:**

```bash
# ❌ Too vague
git commit -m "fixes"
git commit -m "updates"
git commit -m "changes"

# ❌ Not descriptive
git commit -m "fix bug"
git commit -m "add feature"

# ❌ Wrong tense
git commit -m "fixing bug"  # Use imperative: "fix bug"
git commit -m "added feature"  # Use imperative: "add feature"
```

---

## Error Handling Patterns

### Try-Catch Placement

```typescript
// ✅ Good: Handle errors at the right level
function ChatInput() {
  const createMessage = useCreateMessage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      await createMessage.mutateAsync({ text, conversationId });
      setText(''); // Only clear on success

    } catch (error) {
      // Show user-friendly error message
      toast.error('Failed to send message');
    }
  };
}

// ❌ Bad: Swallowing errors
async function handleSubmit() {
  try {
    await createMessage.mutateAsync({ text, conversationId });
  } catch (error) {
    // Nothing! User has no feedback
  }
}
```

### Error Messages

```typescript
// ✅ Good: User-friendly, actionable errors
throw new Error('Message text is required and must be between 1-10000 characters');
throw new Error('Invalid conversation ID. Please try refreshing the page.');

// ❌ Bad: Technical, vague errors
throw new Error('Invalid input');
throw new Error('Error');
throw new Error(JSON.stringify(error)); // Don't stringify!
```

---

## Testing Patterns

### Test File Naming

```
component.tsx      → component.test.tsx
utils.ts           → utils.test.ts
messageService.js  → messageService.spec.js
```

### Test Structure

```typescript
import { render, screen } from '@testing-library/react';
import { ChatInput } from './ChatInput';

describe('ChatInput', () => {
  describe('rendering', () => {
    it('renders textarea', () => {
      render(<ChatInput conversationId="123" />);
      expect(screen.getByRole('textbox')).toBeInTheDocument();
    });

    it('renders send button', () => {
      render(<ChatInput conversationId="123" />);
      expect(screen.getByRole('button', { name: /send/i })).toBeInTheDocument();
    });
  });

  describe('user interactions', () => {
    it('calls onSubmit when form is submitted', () => {
      const onSubmit = jest.fn();
      render(<ChatInput conversationId="123" onSubmit={onSubmit} />);

      // ... test interaction
    });
  });

  describe('validation', () => {
    it('shows error for empty message', () => {
      // ... test validation
    });
  });
});
```

---

## Documentation Patterns

### JSDoc Comments

**Document complex functions:**

```typescript
/**
 * Processes a user message before sending to AI provider.
 *
 * Steps:
 * 1. Sanitizes input (removes dangerous content)
 * 2. Applies rate limiting checks
 * 3. Adds conversation context
 *
 * @param text - The user's message text
 * @param conversationId - ID of the conversation
 * @returns Processed message object ready for AI API
 * @throws {ValidationError} If message is empty or exceeds max length
 *
 * @example
 * const processed = processMessage('Hello!', '123');
 * // Returns: { text: 'Hello!', sanitized: true, context: [...] }
 */
async function processMessage(text: string, conversationId: string): Promise<ProcessedMessage> {
  // ...
}
```

### Inline Comments

**Explain WHY, not WHAT:**

```typescript
// ✅ Good: Explains WHY
// Debounce to avoid excessive API calls while user is typing
const debouncedSearch = debounce(search, 300);

// We use a ref instead of state to avoid re-renders on every cursor position change
const cursorPositionRef = useRef(0);

// ❌ Bad: Explains obvious WHAT
// Set loading to true
setLoading(true);

// Loop through messages
for (const message of messages) {
  // ...
}
```

---

## Common Anti-Patterns to Avoid

### 1. Magic Numbers/Strings

```typescript
// ❌ Bad
if (user.role === 'admin') { }
setTimeout(() => {}, 5000);

// ✅ Good
const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user'
} as const;

const DEBOUNCE_DELAY = 5000;

if (user.role === USER_ROLES.ADMIN) { }
setTimeout(() => {}, DEBOUNCE_DELAY);
```

### 2. Deeply Nested Conditionals

```typescript
// ❌ Bad
function processMessage(message) {
  if (message) {
    if (message.text) {
      if (message.text.length > 0) {
        if (message.text.length < 10000) {
          // Process message
        }
      }
    }
  }
}

// ✅ Good: Early returns
function processMessage(message) {
  if (!message) return;
  if (!message.text) return;
  if (message.text.length === 0) return;
  if (message.text.length >= 10000) return;

  // Process message
}
```

### 3. Mutating Props

```typescript
// ❌ Bad
function ChatInput({ initialText }: { initialText: string }) {
  initialText = 'Modified!'; // Mutating prop!
}

// ✅ Good
function ChatInput({ initialText }: { initialText: string }) {
  const [text, setText] = useState(initialText);
  setText('Modified'); // Modify local state
}
```

### 4. Overusing useEffect

```typescript
// ❌ Bad: Unnecessary useEffect
function Component({ userId }: { userId: string }) {
  const [userName, setUserName] = useState('');

  useEffect(() => {
    const user = users.find(u => u.id === userId);
    setUserName(user?.name ?? '');
  }, [userId, users]);

  return <div>{userName}</div>;
}

// ✅ Good: Calculate directly
function Component({ userId }: { userId: string }) {
  const userName = users.find(u => u.id === userId)?.name ?? '';
  return <div>{userName}</div>;
}
```

### 5. Not Cleaning Up Effects

```typescript
// ❌ Bad: Memory leak
useEffect(() => {
  const interval = setInterval(() => {
    console.log('tick');
  }, 1000);
  // No cleanup!
}, []);

// ✅ Good: Cleanup
useEffect(() => {
  const interval = setInterval(() => {
    console.log('tick');
  }, 1000);

  return () => clearInterval(interval);
}, []);
```

---

## Summary

**Key Conventions:**

1. **Code Style:** 2 spaces, single quotes, trailing commas
2. **Naming:**
   - Files: PascalCase (components), camelCase (utils/hooks)
   - Variables: camelCase
   - Constants: UPPER_CASE
   - Types: PascalCase
3. **Imports:** External → Internal → Components → Styles
4. **Components:** Small, focused, well-typed
5. **Backend:** Thin controllers, fat services
6. **Git:** `type(scope): message` format
7. **Testing:** Describe behavior, not implementation
8. **Documentation:** Explain WHY, not WHAT

---

**Next Steps:**

- Read [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Apply these patterns
- Read [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Daily development practices
- Review actual LibreChat code to see patterns in action

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
