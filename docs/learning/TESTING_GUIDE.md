# 🧪 Testing Guide

**Documented:** November 19, 2025
**Target:** Contributors writing tests for LibreChat
**Time Estimate:** 4-6 hours to fully understand
**Difficulty:** 🟡 Intermediate

---

## Testing Strategy

Comprehensive testing ensures LibreChat remains reliable and maintainable. This guide covers unit tests, integration tests, and end-to-end tests.

---

## Table of Contents

- [Testing Philosophy](#testing-philosophy)
- [Test Types Overview](#test-types-overview)
- [Frontend Testing (Jest + React Testing Library)](#frontend-testing-jest--react-testing-library)
- [Backend Testing (Jest)](#backend-testing-jest)
- [End-to-End Testing (Playwright)](#end-to-end-testing-playwright)
- [Test Coverage](#test-coverage)
- [Best Practices](#best-practices)
- [Common Testing Patterns](#common-testing-patterns)
- [Mocking Strategies](#mocking-strategies)
- [CI/CD Integration](#cicd-integration)

---

## Testing Philosophy

### Test Pyramid

```
        /\
       /E2E\      ← Few, slow, expensive (user flows)
      /------\
     /Integr.\   ← Some, medium speed (API + DB)
    /----------\
   /   Unit     \ ← Many, fast, cheap (functions, components)
  /--------------\
```

**Goal:** Many fast unit tests, some integration tests, few E2E tests

### What to Test

**✅ Do test:**
- Business logic
- User interactions
- Error handling
- Edge cases
- Critical paths

**❌ Don't test:**
- Third-party libraries
- Implementation details
- Trivial code (getters/setters)

---

## Test Types Overview

| Type | Speed | Cost | Coverage | Tools |
|------|-------|------|----------|-------|
| **Unit** | Fast | Low | Functions, components | Jest |
| **Integration** | Medium | Medium | API + DB, hooks + API | Jest + Supertest |
| **E2E** | Slow | High | Complete user flows | Playwright |

---

## Frontend Testing (Jest + React Testing Library)

### Setup

**Installed packages:**
```json
{
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "jest": "^30.0.0",
    "@types/jest": "^30.0.0"
  }
}
```

**Configuration:** `client/jest.config.js`

```javascript
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapper: {
    '^~/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss)$': 'identity-obj-proxy'
  },
  transform: {
    '^.+\\.tsx?$': 'ts-jest'
  }
};
```

**Setup file:** `client/src/setupTests.ts`

```typescript
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

### Testing Components

**Example:** Test ChatInput component

**File:** `client/src/components/Chat/ChatInput.test.tsx`

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ChatInput } from './ChatInput';

// Create wrapper for providers
function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

describe('ChatInput', () => {
  const conversationId = '123';

  describe('Rendering', () => {
    it('renders textarea', () => {
      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      expect(screen.getByRole('textbox')).toBeInTheDocument();
    });

    it('renders send button', () => {
      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      expect(screen.getByRole('button', { name: /send/i })).toBeInTheDocument();
    });

    it('send button is disabled when input is empty', () => {
      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      const button = screen.getByRole('button', { name: /send/i });
      expect(button).toBeDisabled();
    });
  });

  describe('User Interactions', () => {
    it('enables send button when text is entered', async () => {
      const user = userEvent.setup();

      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      const textarea = screen.getByRole('textbox');
      const button = screen.getByRole('button', { name: /send/i });

      // Initially disabled
      expect(button).toBeDisabled();

      // Type text
      await user.type(textarea, 'Hello');

      // Now enabled
      expect(button).toBeEnabled();
    });

    it('clears input after sending message', async () => {
      const user = userEvent.setup();

      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      const textarea = screen.getByRole('textbox');

      // Type and submit
      await user.type(textarea, 'Hello');
      await user.click(screen.getByRole('button', { name: /send/i }));

      // Input should be cleared
      await waitFor(() => {
        expect(textarea).toHaveValue('');
      });
    });
  });

  describe('Validation', () => {
    it('shows error for message over 10000 characters', async () => {
      const user = userEvent.setup();

      render(<ChatInput conversationId={conversationId} />, {
        wrapper: createWrapper()
      });

      const longText = 'a'.repeat(10001);
      const textarea = screen.getByRole('textbox');

      await user.type(textarea, longText);

      expect(screen.getByText(/maximum 10000 characters/i)).toBeInTheDocument();
    });
  });
});
```

### Testing Custom Hooks

**Example:** Test useLocalStorage hook

**File:** `client/src/hooks/useLocalStorage.test.ts`

```typescript
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

  it('updates value when setter is called', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'initial'));

    act(() => {
      result.current[1]('updated');
    });

    expect(result.current[0]).toBe('updated');
  });

  it('supports functional updates', () => {
    const { result } = renderHook(() => useLocalStorage('count', 0));

    act(() => {
      result.current[1](prev => prev + 1);
    });

    expect(result.current[0]).toBe(1);
  });
});
```

### Testing with TanStack Query

**Mock API responses:**

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { useConversations } from './useConversations';

// Mock server
const server = setupServer(
  rest.get('/api/conversations', (req, res, ctx) => {
    return res(
      ctx.json({
        conversations: [
          { id: '1', title: 'Chat 1', model: 'gpt-4' },
          { id: '2', title: 'Chat 2', model: 'claude-3' }
        ]
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('useConversations', () => {
  it('fetches conversations successfully', async () => {
    const queryClient = new QueryClient();
    const wrapper = ({ children }: any) => (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );

    const { result } = renderHook(() => useConversations(), { wrapper });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    expect(result.current.data).toHaveLength(2);
    expect(result.current.data[0].title).toBe('Chat 1');
  });

  it('handles error', async () => {
    server.use(
      rest.get('/api/conversations', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    const queryClient = new QueryClient();
    const wrapper = ({ children }: any) => (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );

    const { result } = renderHook(() => useConversations(), { wrapper });

    await waitFor(() => expect(result.current.isError).toBe(true));
  });
});
```

---

## Backend Testing (Jest)

### Setup

**Configuration:** `api/jest.config.js`

```javascript
module.exports = {
  testEnvironment: 'node',
  coveragePathIgnorePatterns: ['/node_modules/'],
  testMatch: ['**/__tests__/**/*.js', '**/?(*.)+(spec|test).js']
};
```

### Testing Services

**Example:** Test MessageService

**File:** `api/server/services/messageService.test.js`

```javascript
const messageService = require('./messageService');
const Message = require('../../models/Message');
const Conversation = require('../../models/Conversation');

// Mock Mongoose models
jest.mock('../../models/Message');
jest.mock('../../models/Conversation');

describe('MessageService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('createMessage', () => {
    it('creates user message and AI response', async () => {
      const mockConversation = {
        _id: 'conv123',
        userId: 'user123',
        title: 'Test Chat'
      };

      const mockUserMessage = {
        _id: 'msg1',
        role: 'user',
        text: 'Hello',
        conversationId: 'conv123'
      };

      const mockAIMessage = {
        _id: 'msg2',
        role: 'assistant',
        text: 'Hi there!',
        conversationId: 'conv123'
      };

      Conversation.findById.mockResolvedValue(mockConversation);
      Message.create
        .mockResolvedValueOnce(mockUserMessage)
        .mockResolvedValueOnce(mockAIMessage);

      const result = await messageService.createMessage({
        text: 'Hello',
        conversationId: 'conv123',
        userId: 'user123',
        model: 'gpt-4'
      });

      expect(Conversation.findById).toHaveBeenCalledWith('conv123');
      expect(Message.create).toHaveBeenCalledTimes(2);
      expect(result.text).toBe('Hi there!');
    });

    it('throws error if conversation not found', async () => {
      Conversation.findById.mockResolvedValue(null);

      await expect(
        messageService.createMessage({
          text: 'Hello',
          conversationId: 'invalid',
          userId: 'user123',
          model: 'gpt-4'
        })
      ).rejects.toThrow('Conversation not found');
    });

    it('throws error if user does not own conversation', async () => {
      const mockConversation = {
        _id: 'conv123',
        userId: 'other-user',
        title: 'Test Chat'
      };

      Conversation.findById.mockResolvedValue(mockConversation);

      await expect(
        messageService.createMessage({
          text: 'Hello',
          conversationId: 'conv123',
          userId: 'user123',
          model: 'gpt-4'
        })
      ).rejects.toThrow('Not your conversation');
    });
  });
});
```

### Testing Controllers (Integration)

**Example:** Test MessagesController with Supertest

**File:** `api/server/controllers/messagesController.test.js`

```javascript
const request = require('supertest');
const app = require('../app');
const Message = require('../../models/Message');
const Conversation = require('../../models/Conversation');

jest.mock('../../models/Message');
jest.mock('../../models/Conversation');

describe('MessagesController', () => {
  let authToken;

  beforeAll(async () => {
    // Login to get auth token
    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    authToken = response.body.token;
  });

  describe('POST /api/messages', () => {
    it('creates message successfully', async () => {
      const mockMessage = {
        _id: 'msg123',
        text: 'Hello',
        role: 'assistant',
        conversationId: 'conv123'
      };

      Message.create.mockResolvedValue(mockMessage);
      Conversation.findById.mockResolvedValue({
        _id: 'conv123',
        userId: 'user123'
      });

      const response = await request(app)
        .post('/api/messages')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          text: 'Hello',
          conversationId: 'conv123',
          model: 'gpt-4'
        });

      expect(response.status).toBe(201);
      expect(response.body.message.text).toBe('Hello');
    });

    it('returns 401 without auth token', async () => {
      const response = await request(app)
        .post('/api/messages')
        .send({ text: 'Hello', conversationId: 'conv123' });

      expect(response.status).toBe(401);
    });

    it('returns 400 for invalid input', async () => {
      const response = await request(app)
        .post('/api/messages')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ conversationId: 'conv123' }); // Missing text

      expect(response.status).toBe(400);
    });
  });
});
```

### Testing Database Models

**Example:** Test Message model

**File:** `api/models/Message.test.js`

```javascript
const mongoose = require('mongoose');
const Message = require('./Message');

describe('Message Model', () => {
  beforeAll(async () => {
    await mongoose.connect(process.env.MONGO_TEST_URI);
  });

  afterAll(async () => {
    await mongoose.connection.close();
  });

  beforeEach(async () => {
    await Message.deleteMany({});
  });

  it('creates message with valid data', async () => {
    const messageData = {
      conversationId: new mongoose.Types.ObjectId(),
      userId: new mongoose.Types.ObjectId(),
      role: 'user',
      text: 'Hello',
      model: 'gpt-4'
    };

    const message = await Message.create(messageData);

    expect(message.text).toBe('Hello');
    expect(message.role).toBe('user');
    expect(message.createdAt).toBeDefined();
  });

  it('requires conversationId', async () => {
    const messageData = {
      userId: new mongoose.Types.ObjectId(),
      role: 'user',
      text: 'Hello'
    };

    await expect(Message.create(messageData)).rejects.toThrow();
  });

  it('validates role enum', async () => {
    const messageData = {
      conversationId: new mongoose.Types.ObjectId(),
      userId: new mongoose.Types.ObjectId(),
      role: 'invalid', // Not in enum
      text: 'Hello'
    };

    await expect(Message.create(messageData)).rejects.toThrow();
  });

  it('trims text field', async () => {
    const messageData = {
      conversationId: new mongoose.Types.ObjectId(),
      userId: new mongoose.Types.ObjectId(),
      role: 'user',
      text: '  Hello  ' // Whitespace
    };

    const message = await Message.create(messageData);

    expect(message.text).toBe('Hello'); // Trimmed
  });
});
```

---

## End-to-End Testing (Playwright)

### Setup

**Install:**
```bash
npm install --save-dev @playwright/test
npx playwright install
```

**Configuration:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  timeout: 30000,
  use: {
    baseURL: 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'firefox', use: { browserName: 'firefox' } },
    { name: 'webkit', use: { browserName: 'webkit' } }
  ]
});
```

### E2E Test Examples

**Example:** Test complete chat flow

**File:** `e2e/chat.spec.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('Chat Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('input[type="email"]', 'test@example.com');
    await page.fill('input[type="password"]', 'password');
    await page.click('button[type="submit"]');

    // Wait for redirect
    await page.waitForURL('/');
  });

  test('user can send message and receive response', async ({ page }) => {
    // Click new chat
    await page.click('text=New Chat');

    // Type message
    const textarea = page.locator('textarea[placeholder*="message"]');
    await textarea.fill('Hello, AI!');

    // Send message
    await page.click('button:has-text("Send")');

    // Wait for user message to appear
    await expect(page.locator('text=Hello, AI!')).toBeVisible();

    // Wait for AI response (streaming)
    await expect(page.locator('[data-role="assistant"]')).toBeVisible({
      timeout: 10000
    });

    // Verify response is not empty
    const response = await page.locator('[data-role="assistant"]').textContent();
    expect(response).toBeTruthy();
    expect(response.length).toBeGreaterThan(0);
  });

  test('user can create multiple conversations', async ({ page }) => {
    // Create first conversation
    await page.click('text=New Chat');
    await page.fill('textarea', 'First message');
    await page.click('button:has-text("Send")');

    // Wait for response
    await page.waitForSelector('[data-role="assistant"]');

    // Create second conversation
    await page.click('text=New Chat');
    await page.fill('textarea', 'Second message');
    await page.click('button:has-text("Send")');

    // Verify two conversations in sidebar
    const conversations = await page.locator('.conversation-item').count();
    expect(conversations).toBe(2);
  });

  test('user can switch between conversations', async ({ page }) => {
    // Create two conversations
    await page.click('text=New Chat');
    await page.fill('textarea', 'First');
    await page.click('button:has-text("Send")');
    await page.waitForSelector('[data-role="assistant"]');

    await page.click('text=New Chat');
    await page.fill('textarea', 'Second');
    await page.click('button:has-text("Send")');
    await page.waitForSelector('[data-role="assistant"]');

    // Click first conversation
    await page.click('.conversation-item:nth-child(1)');

    // Verify first message is visible
    await expect(page.locator('text=First')).toBeVisible();

    // Click second conversation
    await page.click('.conversation-item:nth-child(2)');

    // Verify second message is visible
    await expect(page.locator('text=Second')).toBeVisible();
  });

  test('displays error for failed message', async ({ page }) => {
    // Mock API to return error
    await page.route('/api/messages', route =>
      route.fulfill({ status: 500, body: 'Server error' })
    );

    await page.click('text=New Chat');
    await page.fill('textarea', 'This will fail');
    await page.click('button:has-text("Send")');

    // Verify error message displayed
    await expect(page.locator('text=Failed to send message')).toBeVisible();
  });
});
```

---

## Test Coverage

### Run Coverage

```bash
# Backend coverage
npm test -- --coverage

# Frontend coverage
npm run test:frontend -- --coverage

# View HTML report
open coverage/lcov-report/index.html
```

### Coverage Thresholds

**File:** `jest.config.js`

```javascript
module.exports = {
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
};
```

### What Coverage Means

- **Lines:** Percentage of lines executed
- **Functions:** Percentage of functions called
- **Branches:** Percentage of if/else paths taken
- **Statements:** Percentage of statements executed

**Goal:** 70-80% coverage for most projects

---

## Best Practices

### 1. Test Behavior, Not Implementation

```typescript
// ❌ Bad: Testing implementation
it('calls useState with empty string', () => {
  const useStateSpy = jest.spyOn(React, 'useState');
  render(<ChatInput />);
  expect(useStateSpy).toHaveBeenCalledWith('');
});

// ✅ Good: Testing behavior
it('renders empty textarea', () => {
  render(<ChatInput />);
  expect(screen.getByRole('textbox')).toHaveValue('');
});
```

### 2. Write Descriptive Test Names

```typescript
// ❌ Bad
it('works', () => { });
it('test 1', () => { });

// ✅ Good
it('disables send button when input is empty', () => { });
it('displays error for message over 10000 characters', () => { });
```

### 3. Arrange, Act, Assert (AAA)

```typescript
it('updates message count after sending', async () => {
  // Arrange
  const initialCount = 5;
  const mockConversation = { messageCount: initialCount };

  // Act
  await messageService.createMessage({...});

  // Assert
  expect(mockConversation.messageCount).toBe(initialCount + 2);
});
```

### 4. Don't Test Third-Party Libraries

```typescript
// ❌ Bad: Testing React Router
it('navigates correctly', () => {
  const { result } = renderHook(() => useNavigate());
  act(() => result.current('/home'));
  expect(window.location.pathname).toBe('/home');
});

// ✅ Good: Test YOUR code
it('calls navigate with /home when button clicked', () => {
  const mockNavigate = jest.fn();
  jest.mock('react-router-dom', () => ({
    useNavigate: () => mockNavigate
  }));

  render(<MyComponent />);
  fireEvent.click(screen.getByRole('button'));

  expect(mockNavigate).toHaveBeenCalledWith('/home');
});
```

---

## Common Testing Patterns

### Testing Async Code

```typescript
it('fetches data on mount', async () => {
  render(<DataComponent />);

  // Wait for element to appear
  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeInTheDocument();
  });
});
```

### Testing User Events

```typescript
it('handles button click', async () => {
  const user = userEvent.setup();
  const handleClick = jest.fn();

  render(<Button onClick={handleClick} />);

  await user.click(screen.getByRole('button'));

  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

### Testing Forms

```typescript
it('submits form with valid data', async () => {
  const user = userEvent.setup();
  const handleSubmit = jest.fn();

  render(<LoginForm onSubmit={handleSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/password/i), 'password123');
  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});
```

---

## Mocking Strategies

### Mock API Calls

```typescript
// Mock axios
jest.mock('~/utils/apiClient');
import apiClient from '~/utils/apiClient';

apiClient.get = jest.fn().mockResolvedValue({
  data: { conversations: [] }
});
```

### Mock LocalStorage

```typescript
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  clear: jest.fn()
};

global.localStorage = localStorageMock as any;
```

### Mock Timers

```typescript
it('debounces input', () => {
  jest.useFakeTimers();

  render(<SearchInput />);
  const input = screen.getByRole('textbox');

  fireEvent.change(input, { target: { value: 'test' } });

  // Fast forward 300ms
  jest.advanceTimersByTime(300);

  expect(mockSearch).toHaveBeenCalledWith('test');

  jest.useRealTimers();
});
```

---

## CI/CD Integration

### GitHub Actions Example

**File:** `.github/workflows/test.yml`

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

---

## Summary

**Testing Strategy:**

1. **Unit Tests:** Fast, isolated, many
2. **Integration Tests:** API + DB, medium speed
3. **E2E Tests:** Complete flows, few, slow

**Tools:**
- **Jest:** Unit and integration tests
- **React Testing Library:** Component tests
- **Playwright:** E2E tests

**Best Practices:**
- Test behavior, not implementation
- Write descriptive test names
- Use AAA pattern
- Mock external dependencies
- Aim for 70-80% coverage

---

**Next Steps:**

- Read [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md) - Debug issues
- Read [SECURITY_GUIDE.md](./SECURITY_GUIDE.md) - Security practices
- Write tests for your features!

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
