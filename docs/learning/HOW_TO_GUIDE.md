# 🛠️ How-To Guide: Common Tasks

**Documented:** November 19, 2025
**Target:** Contributors ready to build features
**Time Estimate:** Reference guide (use as needed)
**Difficulty:** 🟡 Intermediate

---

## Practical Step-by-Step Guides

This guide provides step-by-step instructions for common development tasks in LibreChat. Use this as a cookbook when implementing features or modifications.

---

## Table of Contents

**Frontend:**
- [Add a New React Component](#add-a-new-react-component)
- [Create a Custom Hook](#create-a-custom-hook)
- [Add a Form with Validation](#add-a-form-with-validation)
- [Implement Optimistic Updates](#implement-optimistic-updates)

**Backend:**
- [Add a New API Endpoint](#add-a-new-api-endpoint)
- [Create a Database Model](#create-a-database-model)
- [Add Middleware](#add-middleware)
- [Implement Authentication](#implement-authentication)

**Full-Stack:**
- [Build a Feature End-to-End](#build-a-feature-end-to-end)
- [Add File Upload](#add-file-upload)
- [Implement Real-time Updates](#implement-real-time-updates)

**Development:**
- [Run Tests](#run-tests)
- [Debug Frontend](#debug-frontend)
- [Debug Backend](#debug-backend)
- [Fix Common Errors](#fix-common-errors)

---

## Add a New React Component

### Step 1: Plan Your Component

**Questions to answer:**
- What data does it display?
- What user interactions does it handle?
- What props does it need?
- Is it shared or feature-specific?

**Example:** Create a `UserProfileCard` component

### Step 2: Create Component File

**Location:**
- Shared component: `client/src/components/shared/UserProfileCard.tsx`
- Feature-specific: `client/src/components/[Feature]/UserProfileCard.tsx`

```bash
# Create file
touch client/src/components/shared/UserProfileCard.tsx
```

### Step 3: Define TypeScript Interface

```typescript
// client/src/components/shared/UserProfileCard.tsx

interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

interface UserProfileCardProps {
  user: User;
  onEdit?: () => void;
}
```

### Step 4: Implement Component

```typescript
export function UserProfileCard({ user, onEdit }: UserProfileCardProps) {
  return (
    <div className="rounded-lg border p-4 shadow-sm">
      {/* Avatar */}
      {user.avatar && (
        <img
          src={user.avatar}
          alt={user.name}
          className="h-16 w-16 rounded-full"
        />
      )}

      {/* Name */}
      <h3 className="mt-2 text-lg font-semibold">{user.name}</h3>

      {/* Email */}
      <p className="text-sm text-gray-600">{user.email}</p>

      {/* Edit button */}
      {onEdit && (
        <button
          onClick={onEdit}
          className="mt-4 rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600"
        >
          Edit Profile
        </button>
      )}
    </div>
  );
}
```

### Step 5: Export Component

```typescript
// client/src/components/shared/index.ts
export { UserProfileCard } from './UserProfileCard';
```

### Step 6: Use Component

```typescript
// In another component
import { UserProfileCard } from '~/components/shared';

function ProfilePage() {
  const { data: user } = useUser();

  if (!user) return null;

  return (
    <div>
      <h1>My Profile</h1>
      <UserProfileCard
        user={user}
        onEdit={() => navigate('/profile/edit')}
      />
    </div>
  );
}
```

### Step 7: Write Tests

```typescript
// client/src/components/shared/UserProfileCard.test.tsx

import { render, screen } from '@testing-library/react';
import { UserProfileCard } from './UserProfileCard';

describe('UserProfileCard', () => {
  const mockUser = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com'
  };

  it('renders user name', () => {
    render(<UserProfileCard user={mockUser} />);
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });

  it('renders user email', () => {
    render(<UserProfileCard user={mockUser} />);
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button clicked', () => {
    const onEdit = jest.fn();
    render(<UserProfileCard user={mockUser} onEdit={onEdit} />);

    const button = screen.getByRole('button', { name: /edit/i });
    button.click();

    expect(onEdit).toHaveBeenCalledTimes(1);
  });
});
```

---

## Create a Custom Hook

### Step 1: Identify Reusable Logic

**Good candidates for hooks:**
- API data fetching
- Form handling
- Authentication checks
- Local storage interactions
- Window resize/scroll listeners

**Example:** Create `useLocalStorage` hook

### Step 2: Create Hook File

```bash
touch client/src/hooks/useLocalStorage.ts
```

### Step 3: Implement Hook

```typescript
// client/src/hooks/useLocalStorage.ts

import { useState, useEffect } from 'react';

/**
 * Hook: Sync state with localStorage
 *
 * @param key - localStorage key
 * @param initialValue - Default value if key doesn't exist
 * @returns [value, setValue] tuple
 *
 * @example
 * const [name, setName] = useLocalStorage('userName', 'Guest');
 */
export function useLocalStorage<T>(key: string, initialValue: T) {
  // State to store our value
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      // Get from local storage by key
      const item = window.localStorage.getItem(key);

      // Parse stored json or return initialValue
      return item ? JSON.parse(item) : initialValue;

    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  // Return a wrapped version of useState's setter that persists
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      // Allow value to be a function (same API as useState)
      const valueToStore = value instanceof Function ? value(storedValue) : value;

      // Save state
      setStoredValue(valueToStore);

      // Save to local storage
      window.localStorage.setItem(key, JSON.stringify(valueToStore));

    } catch (error) {
      console.error('Error writing to localStorage:', error);
    }
  };

  return [storedValue, setValue] as const;
}
```

### Step 4: Export Hook

```typescript
// client/src/hooks/index.ts
export { useLocalStorage } from './useLocalStorage';
```

### Step 5: Use Hook

```typescript
// In a component
import { useLocalStorage } from '~/hooks';

function SettingsPage() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  return (
    <div>
      <label>
        Theme:
        <select value={theme} onChange={(e) => setTheme(e.target.value)}>
          <option value="light">Light</option>
          <option value="dark">Dark</option>
        </select>
      </label>
    </div>
  );
}
```

### Step 6: Write Tests

```typescript
// client/src/hooks/useLocalStorage.test.ts

import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('returns initial value when key does not exist', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('default');
  });

  it('saves value to localStorage', () => {
    const { result } = renderHook(() => useLocalStorage('key', ''));

    act(() => {
      result.current[1]('new value');
    });

    expect(localStorage.getItem('key')).toBe('"new value"');
  });

  it('loads existing value from localStorage', () => {
    localStorage.setItem('key', '"existing"');

    const { result } = renderHook(() => useLocalStorage('key', 'default'));

    expect(result.current[0]).toBe('existing');
  });
});
```

---

## Add a Form with Validation

### Step 1: Install Dependencies (if needed)

```bash
npm install react-hook-form zod @hookform/resolvers
```

### Step 2: Define Validation Schema

```typescript
// client/src/schemas/user.ts

import { z } from 'zod';

export const userProfileSchema = z.object({
  name: z
    .string()
    .min(2, 'Name must be at least 2 characters')
    .max(50, 'Name must be less than 50 characters'),

  email: z
    .string()
    .email('Invalid email format'),

  bio: z
    .string()
    .max(500, 'Bio must be less than 500 characters')
    .optional(),

  avatar: z
    .string()
    .url('Invalid avatar URL')
    .optional()
});

export type UserProfileFormData = z.infer<typeof userProfileSchema>;
```

### Step 3: Create Form Component

```typescript
// client/src/components/User/EditProfileForm.tsx

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { userProfileSchema, UserProfileFormData } from '~/schemas/user';
import { useUpdateUser } from '~/hooks/useUser';

interface EditProfileFormProps {
  initialData: UserProfileFormData;
  onSuccess?: () => void;
}

export function EditProfileForm({ initialData, onSuccess }: EditProfileFormProps) {
  const updateUser = useUpdateUser();

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<UserProfileFormData>({
    resolver: zodResolver(userProfileSchema),
    defaultValues: initialData
  });

  const onSubmit = async (data: UserProfileFormData) => {
    try {
      await updateUser.mutateAsync(data);
      onSuccess?.();
    } catch (error) {
      console.error('Failed to update profile:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      {/* Name field */}
      <div>
        <label htmlFor="name" className="block text-sm font-medium">
          Name
        </label>
        <input
          id="name"
          type="text"
          {...register('name')}
          className="mt-1 block w-full rounded border p-2"
        />
        {errors.name && (
          <p className="mt-1 text-sm text-red-600">{errors.name.message}</p>
        )}
      </div>

      {/* Email field */}
      <div>
        <label htmlFor="email" className="block text-sm font-medium">
          Email
        </label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className="mt-1 block w-full rounded border p-2"
        />
        {errors.email && (
          <p className="mt-1 text-sm text-red-600">{errors.email.message}</p>
        )}
      </div>

      {/* Bio field */}
      <div>
        <label htmlFor="bio" className="block text-sm font-medium">
          Bio
        </label>
        <textarea
          id="bio"
          {...register('bio')}
          rows={4}
          className="mt-1 block w-full rounded border p-2"
        />
        {errors.bio && (
          <p className="mt-1 text-sm text-red-600">{errors.bio.message}</p>
        )}
      </div>

      {/* Submit button */}
      <button
        type="submit"
        disabled={isSubmitting}
        className="rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600 disabled:opacity-50"
      >
        {isSubmitting ? 'Saving...' : 'Save Changes'}
      </button>

      {/* Error message */}
      {updateUser.isError && (
        <p className="text-sm text-red-600">
          Failed to update profile. Please try again.
        </p>
      )}
    </form>
  );
}
```

### Step 4: Use Form Component

```typescript
function EditProfilePage() {
  const { data: user } = useUser();
  const navigate = useNavigate();

  if (!user) return null;

  return (
    <div className="mx-auto max-w-2xl p-4">
      <h1 className="mb-4 text-2xl font-bold">Edit Profile</h1>

      <EditProfileForm
        initialData={{
          name: user.name,
          email: user.email,
          bio: user.bio,
          avatar: user.avatar
        }}
        onSuccess={() => {
          navigate('/profile');
        }}
      />
    </div>
  );
}
```

---

## Implement Optimistic Updates

### Step 1: Create Mutation with Optimistic Update

```typescript
// client/src/hooks/useMessages.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import apiClient from '~/utils/apiClient';

export function useCreateMessage() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ text, conversationId }: CreateMessageParams) => {
      const { data } = await apiClient.post('/api/messages', {
        text,
        conversationId
      });
      return data.message;
    },

    // ===========================
    // Optimistic Update
    // ===========================

    onMutate: async (newMessage) => {
      // Cancel any outgoing refetches
      await queryClient.cancelQueries({
        queryKey: ['messages', newMessage.conversationId]
      });

      // Snapshot the previous value
      const previousMessages = queryClient.getQueryData([
        'messages',
        newMessage.conversationId
      ]);

      // Optimistically update to the new value
      queryClient.setQueryData(
        ['messages', newMessage.conversationId],
        (old: Message[]) => [
          ...old,
          {
            id: `temp-${Date.now()}`, // Temporary ID
            role: 'user',
            text: newMessage.text,
            createdAt: new Date().toISOString(),
            isOptimistic: true // Flag for UI
          }
        ]
      );

      // Return context object with the snapshotted value
      return { previousMessages };
    },

    // ===========================
    // On Error: Rollback
    // ===========================

    onError: (err, newMessage, context) => {
      // Rollback to previous state
      if (context?.previousMessages) {
        queryClient.setQueryData(
          ['messages', newMessage.conversationId],
          context.previousMessages
        );
      }

      // Show error toast
      toast.error('Failed to send message');
    },

    // ===========================
    // On Success: Replace Temp with Real
    // ===========================

    onSuccess: (savedMessage, variables) => {
      // Invalidate to trigger refetch (gets real data from server)
      queryClient.invalidateQueries({
        queryKey: ['messages', variables.conversationId]
      });
    }
  });
}
```

### Step 2: Use in Component

```typescript
function ChatInput({ conversationId }: { conversationId: string }) {
  const [text, setText] = useState('');
  const createMessage = useCreateMessage();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!text.trim()) return;

    // Save text for potential retry
    const messageText = text;

    // Clear input immediately (optimistic)
    setText('');

    try {
      await createMessage.mutateAsync({
        text: messageText,
        conversationId
      });

    } catch (error) {
      // Restore text if error (so user can retry)
      setText(messageText);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        disabled={createMessage.isPending}
        placeholder="Type your message..."
      />

      <button type="submit" disabled={createMessage.isPending}>
        {createMessage.isPending ? 'Sending...' : 'Send'}
      </button>
    </form>
  );
}
```

### Step 3: Style Optimistic Messages

```typescript
function Message({ message }: { message: Message }) {
  return (
    <div
      className={`message ${
        message.isOptimistic ? 'opacity-50' : 'opacity-100'
      }`}
    >
      <p>{message.text}</p>

      {message.isOptimistic && (
        <span className="text-xs text-gray-500">Sending...</span>
      )}
    </div>
  );
}
```

---

## Add a New API Endpoint

### Step 1: Define Route

```javascript
// api/server/routes/conversations.js

const express = require('express');
const router = express.Router();
const conversationsController = require('../controllers/conversationsController');
const requireAuth = require('../middleware/requireAuth');

/**
 * POST /api/conversations
 * Create a new conversation
 */
router.post(
  '/',
  requireAuth,
  conversationsController.create
);

module.exports = router;
```

### Step 2: Create Controller

```javascript
// api/server/controllers/conversationsController.js

const conversationService = require('../services/conversationService');

class ConversationsController {
  /**
   * POST /api/conversations
   * Create a new conversation
   */
  async create(req, res, next) {
    try {
      const { title, model } = req.body;
      const userId = req.user.id;

      // Validate
      if (!title || title.trim().length === 0) {
        return res.status(400).json({
          error: { message: 'Title is required' }
        });
      }

      // Call service
      const conversation = await conversationService.create({
        title,
        model,
        userId
      });

      // Send response
      res.status(201).json({ conversation });

    } catch (error) {
      next(error);
    }
  }
}

module.exports = new ConversationsController();
```

### Step 3: Create Service

```javascript
// api/server/services/conversationService.js

const Conversation = require('../../models/Conversation');
const logger = require('../utils/logger');

class ConversationService {
  /**
   * Create a new conversation
   */
  async create({ title, model, userId }) {
    const conversation = await Conversation.create({
      title,
      model: model || 'gpt-4',
      userId
    });

    logger.info('Conversation created', {
      conversationId: conversation._id,
      userId
    });

    return conversation;
  }
}

module.exports = new ConversationService();
```

### Step 4: Mount Route in Main Router

```javascript
// api/server/routes/index.js

const express = require('express');
const router = express.Router();

const authRoutes = require('./auth');
const messageRoutes = require('./messages');
const conversationRoutes = require('./conversations'); // Add this

router.use('/auth', authRoutes);
router.use('/messages', messageRoutes);
router.use('/conversations', conversationRoutes); // Add this

module.exports = router;
```

### Step 5: Test Endpoint

```bash
# Test with curl
curl -X POST http://localhost:3080/api/conversations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"title": "New Chat", "model": "gpt-4"}'

# Expected response:
# {
#   "conversation": {
#     "id": "...",
#     "title": "New Chat",
#     "model": "gpt-4",
#     "userId": "...",
#     "createdAt": "..."
#   }
# }
```

### Step 6: Create Frontend Hook

```typescript
// client/src/hooks/useConversations.ts

export function useCreateConversation() {
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
```

---

## Create a Database Model

### Step 1: Define Schema

```javascript
// api/models/Preset.js

const mongoose = require('mongoose');
const { Schema } = mongoose;

/**
 * Preset Schema
 * Stores saved AI conversation presets (model, parameters, etc.)
 */
const presetSchema = new Schema(
  {
    userId: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      index: true
    },

    name: {
      type: String,
      required: true,
      trim: true,
      maxlength: 100
    },

    model: {
      type: String,
      required: true,
      enum: ['gpt-4', 'gpt-3.5-turbo', 'claude-3-sonnet', 'gemini-pro']
    },

    parameters: {
      temperature: {
        type: Number,
        min: 0,
        max: 2,
        default: 0.7
      },
      maxTokens: {
        type: Number,
        min: 1,
        max: 4000,
        default: 2000
      },
      topP: {
        type: Number,
        min: 0,
        max: 1,
        default: 1
      }
    },

    systemMessage: {
      type: String,
      maxlength: 1000
    },

    isDefault: {
      type: Boolean,
      default: false
    }
  },
  {
    timestamps: true
  }
);

// ===========================
// Indexes
// ===========================

presetSchema.index({ userId: 1, name: 1 }, { unique: true });

// ===========================
// Methods
// ===========================

presetSchema.methods.toClient = function () {
  return {
    id: this._id.toString(),
    name: this.name,
    model: this.model,
    parameters: this.parameters,
    systemMessage: this.systemMessage,
    isDefault: this.isDefault,
    createdAt: this.createdAt,
    updatedAt: this.updatedAt
  };
};

// ===========================
// Static Methods
// ===========================

presetSchema.statics.findByUser = function (userId) {
  return this.find({ userId }).sort({ createdAt: -1 });
};

const Preset = mongoose.model('Preset', presetSchema);

module.exports = Preset;
```

### Step 2: Create Migration (if needed)

```javascript
// migrations/create-presets-collection.js

const Preset = require('../api/models/Preset');

async function up() {
  console.log('Creating indexes for Preset collection...');

  await Preset.createIndexes();

  console.log('Preset collection ready');
}

async function down() {
  console.log('Dropping Preset collection...');

  await Preset.collection.drop();

  console.log('Preset collection dropped');
}

module.exports = { up, down };
```

### Step 3: Use Model in Service

```javascript
// api/server/services/presetService.js

const Preset = require('../../models/Preset');

class PresetService {
  async create({ userId, name, model, parameters, systemMessage }) {
    const preset = await Preset.create({
      userId,
      name,
      model,
      parameters,
      systemMessage
    });

    return preset.toClient();
  }

  async findByUser(userId) {
    const presets = await Preset.findByUser(userId);
    return presets.map(p => p.toClient());
  }

  async delete({ presetId, userId }) {
    const preset = await Preset.findById(presetId);

    if (!preset) {
      throw new NotFoundError('Preset not found');
    }

    if (preset.userId.toString() !== userId) {
      throw new ForbiddenError('Not your preset');
    }

    await preset.deleteOne();
  }
}

module.exports = new PresetService();
```

---

## Add Middleware

### Step 1: Create Middleware Function

```javascript
// api/server/middleware/rateLimiter.js

const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('../config/redis');

/**
 * Rate limiting middleware
 * Prevents abuse by limiting requests per IP
 */
const createRateLimiter = (options = {}) => {
  return rateLimit({
    store: new RedisStore({
      client: redis,
      prefix: 'rate-limit:'
    }),

    windowMs: options.windowMs || 15 * 60 * 1000, // 15 minutes
    max: options.max || 100, // Max requests per window
    message: options.message || 'Too many requests, please try again later',

    standardHeaders: true, // Return rate limit info in `RateLimit-*` headers
    legacyHeaders: false, // Disable `X-RateLimit-*` headers

    // Skip successful requests (only count errors)
    skip: (req, res) => res.statusCode < 400,

    // Handler for when limit is exceeded
    handler: (req, res) => {
      res.status(429).json({
        error: {
          message: 'Too many requests, please slow down',
          retryAfter: res.getHeader('RateLimit-Reset')
        }
      });
    }
  });
};

// Export different limiters
module.exports = {
  // General API limiter
  apiLimiter: createRateLimiter({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100
  }),

  // Strict limiter for sensitive endpoints
  authLimiter: createRateLimiter({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // Only 5 login attempts
    message: 'Too many login attempts, please try again later'
  }),

  // Generous limiter for public endpoints
  publicLimiter: createRateLimiter({
    windowMs: 15 * 60 * 1000,
    max: 1000
  })
};
```

### Step 2: Apply Middleware

```javascript
// api/server/routes/auth.js

const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const { authLimiter } = require('../middleware/rateLimiter');

// Apply rate limiting to login/register
router.post('/login', authLimiter, authController.login);
router.post('/register', authLimiter, authController.register);

module.exports = router;
```

### Step 3: Apply Globally (if needed)

```javascript
// api/server/index.js

const { apiLimiter } = require('./middleware/rateLimiter');

// Apply to all /api routes
app.use('/api', apiLimiter);
```

---

## Build a Feature End-to-End

### Example: Add "Archive Conversation" Feature

**Step 1: Update Database Model**

```javascript
// api/models/Conversation.js

const conversationSchema = new Schema({
  // ... existing fields

  // Add archived field
  isArchived: {
    type: Boolean,
    default: false
  },

  archivedAt: Date
});
```

**Step 2: Add Backend Endpoint**

```javascript
// api/server/controllers/conversationsController.js

async archiveConversation(req, res, next) {
  try {
    const { id } = req.params;
    const userId = req.user.id;

    const conversation = await conversationService.archive({
      conversationId: id,
      userId
    });

    res.json({ conversation });

  } catch (error) {
    next(error);
  }
}
```

```javascript
// api/server/services/conversationService.js

async archive({ conversationId, userId }) {
  const conversation = await Conversation.findById(conversationId);

  if (!conversation) {
    throw new NotFoundError('Conversation not found');
  }

  if (conversation.userId.toString() !== userId) {
    throw new ForbiddenError('Not your conversation');
  }

  conversation.isArchived = true;
  conversation.archivedAt = new Date();
  await conversation.save();

  return conversation;
}
```

```javascript
// api/server/routes/conversations.js

router.put('/:id/archive', requireAuth, conversationsController.archiveConversation);
```

**Step 3: Add Frontend Hook**

```typescript
// client/src/hooks/useConversations.ts

export function useArchiveConversation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (conversationId: string) => {
      const { data } = await apiClient.put(
        `/api/conversations/${conversationId}/archive`
      );
      return data.conversation;
    },

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['conversations'] });
    }
  });
}
```

**Step 4: Add UI Component**

```typescript
// client/src/components/Conversations/ArchiveButton.tsx

export function ArchiveButton({ conversationId }: { conversationId: string }) {
  const archiveConversation = useArchiveConversation();

  const handleArchive = async () => {
    if (!confirm('Archive this conversation?')) return;

    try {
      await archiveConversation.mutateAsync(conversationId);
      toast.success('Conversation archived');
    } catch (error) {
      toast.error('Failed to archive conversation');
    }
  };

  return (
    <button
      onClick={handleArchive}
      disabled={archiveConversation.isPending}
      className="text-gray-600 hover:text-gray-800"
    >
      {archiveConversation.isPending ? 'Archiving...' : 'Archive'}
    </button>
  );
}
```

**Step 5: Filter Archived Conversations**

```typescript
// Update conversations query to exclude archived

export function useConversations(includeArchived = false) {
  return useQuery({
    queryKey: ['conversations', includeArchived],

    queryFn: async () => {
      const { data } = await apiClient.get('/api/conversations', {
        params: { includeArchived }
      });
      return data.conversations;
    }
  });
}
```

```javascript
// Backend: Update service to filter

async findByUser(userId, includeArchived = false) {
  const query = { userId };

  if (!includeArchived) {
    query.isArchived = { $ne: true };
  }

  return await Conversation.find(query).sort({ updatedAt: -1 });
}
```

**Step 6: Test the Feature**

```bash
# Backend test
npm test -- conversations.test.js

# Frontend test
npm test -- ArchiveButton.test.tsx

# Manual test
npm run dev
# Click archive button, verify conversation disappears
```

---

## Run Tests

### Run All Tests

```bash
# Backend tests
npm test

# Frontend tests
npm run test:frontend

# E2E tests
npm run test:e2e
```

### Run Specific Tests

```bash
# Run single test file
npm test -- messageService.test.js

# Run tests matching pattern
npm test -- messages

# Run with coverage
npm test -- --coverage
```

### Run Tests in Watch Mode

```bash
# Backend
npm test -- --watch

# Frontend
npm run test:frontend -- --watch
```

### Run E2E Tests

```bash
# Start app first
npm run dev

# In another terminal
npm run test:e2e

# Run specific E2E test
npx playwright test tests/chat.spec.ts

# Run with UI
npx playwright test --ui
```

---

## Debug Frontend

### React DevTools

1. Install React DevTools browser extension
2. Open DevTools → React tab
3. Inspect component tree
4. View props, state, hooks

### TanStack Query DevTools

```typescript
// Already included in LibreChat

import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

**Usage:**
- Click floating icon in bottom-right
- Inspect queries, mutations
- View cache, refetch queries manually

### Console Logging

```typescript
// Debug component renders
function ChatInput() {
  console.log('ChatInput rendered');

  useEffect(() => {
    console.log('ChatInput mounted');
    return () => console.log('ChatInput unmounted');
  }, []);
}

// Debug state changes
const [text, setText] = useState('');

useEffect(() => {
  console.log('Text changed:', text);
}, [text]);

// Debug API calls (already logged by apiClient interceptor)
// Check console for:
// → POST /api/messages
// ← 201 /api/messages
```

### Breakpoints

1. Open browser DevTools → Sources
2. Find file (Cmd/Ctrl + P)
3. Click line number to set breakpoint
4. Trigger code
5. Inspect variables

---

## Debug Backend

### Console Logging

```javascript
// Simple logging
console.log('Creating message:', { text, conversationId });

// Use logger for production
const logger = require('./utils/logger');

logger.info('Message created', {
  messageId: message._id,
  userId,
  conversationId
});

logger.error('Failed to create message', {
  error: error.message,
  stack: error.stack
});
```

### Node Debugger

```bash
# Start with debugger
node --inspect api/server/index.js

# Or with nodemon
nodemon --inspect api/server/index.js
```

**Chrome DevTools:**
1. Open Chrome: `chrome://inspect`
2. Click "inspect" under your app
3. Set breakpoints
4. Trigger request
5. Inspect variables

**VS Code:**
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "program": "${workspaceFolder}/api/server/index.js",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

### Debugging Mongoose Queries

```javascript
// Enable query logging
mongoose.set('debug', true);

// Or custom logger
mongoose.set('debug', (collection, method, ...args) => {
  console.log(`${collection}.${method}`, JSON.stringify(args));
});

// Output:
// messages.find { conversationId: '123' }
// messages.findOne { _id: '456' }
```

---

## Fix Common Errors

### Error: "CORS policy" (Frontend)

**Problem:**
```
Access to fetch at 'http://localhost:3080/api/messages' from origin 'http://localhost:3000'
has been blocked by CORS policy
```

**Solution:**

```javascript
// api/server/index.js

const cors = require('cors');

app.use(cors({
  origin: 'http://localhost:3000', // Frontend URL
  credentials: true
}));
```

### Error: 401 Unauthorized

**Problem:**
```
Request failed with status code 401
```

**Debugging:**

```typescript
// Check token exists
const token = localStorage.getItem('authToken');
console.log('Token:', token); // Should not be null

// Check token is sent
// (Already logged by apiClient interceptor)

// Check token is valid
// Decode JWT at https://jwt.io
```

**Solution:**

```typescript
// Re-login to get new token
await loginMutation.mutateAsync({ email, password });
```

### Error: "Cannot find module"

**Problem:**
```
Error: Cannot find module '~/components/Chat'
```

**Solution:**

```bash
# Restart dev server (may have cached old paths)
npm run dev

# Check path alias in tsconfig.json
# "paths": { "~/*": ["./src/*"] }

# Check import uses correct path
import { Chat } from '~/components/Chat'; // Correct
import { Chat } from 'components/Chat';  // Wrong (missing ~)
```

### Error: Mongoose CastError

**Problem:**
```
CastError: Cast to ObjectId failed for value "123" at path "_id"
```

**Solution:**

```javascript
// Validate ObjectId before query
const mongoose = require('mongoose');

if (!mongoose.Types.ObjectId.isValid(id)) {
  throw new ValidationError('Invalid ID format');
}

const doc = await Model.findById(id);
```

### Error: Port already in use

**Problem:**
```
Error: listen EADDRINUSE: address already in use :::3080
```

**Solution:**

```bash
# Find process using port
lsof -ti:3080

# Kill process
kill -9 $(lsof -ti:3080)

# Or use different port
PORT=3081 npm run dev
```

---

## Summary

**Common Tasks Covered:**

**Frontend:**
- ✅ Add React component
- ✅ Create custom hook
- ✅ Build forms with validation
- ✅ Implement optimistic updates

**Backend:**
- ✅ Add API endpoint
- ✅ Create database model
- ✅ Add middleware

**Full-Stack:**
- ✅ Build feature end-to-end
- ✅ Test implementation
- ✅ Debug issues

---

**Next Steps:**

- Read [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Daily development practices
- Read [TESTING_GUIDE.md](./TESTING_GUIDE.md) - Testing strategies
- Try building a feature yourself!

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
