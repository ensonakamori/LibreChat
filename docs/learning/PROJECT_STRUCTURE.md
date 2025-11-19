# 📂 LibreChat Project Structure

**Documented:** November 19, 2025
**Target:** Mid-level React developers learning full-stack
**Time Estimate:** 1 hour
**Difficulty:** 🟢 Beginner

---

## Table of Contents

- [Overview](#overview)
- [Root Directory](#root-directory)
- [Frontend (client/)](#frontend-client)
- [Backend (api/)](#backend-api)
- [Shared Packages (packages/)](#shared-packages-packages)
- [Configuration Files](#configuration-files)
- [Common Patterns](#common-patterns)
- [Where to Find Things](#where-to-find-things)
- [Import Paths](#import-paths)

---

## Overview

LibreChat is a **monorepo** organized using **npm workspaces**. This means one repository contains multiple related projects that can share code.

### 🧠 Mental Model

Think of it like a mall:
- **Root** = The mall building (contains everything)
- **client/** = Apple Store (frontend/UI)
- **api/** = Restaurant kitchen (backend/cooking)
- **packages/** = Shared warehouse (common supplies)
- **config/** = Mall management office (utilities)

### High-Level Structure

```
LibreChat/
├── client/              # React frontend (SPA)
├── api/                 # Express backend (REST API)
├── packages/            # Shared code (used by client & api)
├── config/              # CLI utility scripts
├── e2e/                 # End-to-end tests (Playwright)
├── docs/                # Documentation (you are here!)
└── [config files]       # Docker, Git, ESLint, etc.
```

---

## Root Directory

```
LibreChat/
├── .devcontainer/           # VS Code dev container config
├── .github/                 # GitHub workflows (CI/CD)
├── .husky/                  # Git hooks (pre-commit, etc.)
├── .vscode/                 # VS Code workspace settings
├── api/                     # ← Backend (Express server)
├── client/                  # ← Frontend (React app)
├── config/                  # CLI scripts for admin tasks
├── docs/                    # Documentation
├── e2e/                     # Playwright E2E tests
├── helm/                    # Kubernetes deployment charts
├── packages/                # ← Shared packages (monorepo)
├── redis-config/            # Redis configuration
├── utils/                   # Utility scripts
│
├── .dockerignore            # Files to ignore in Docker
├── .env.example             # Environment variables template
├── .gitignore               # Files to ignore in Git
├── .prettierrc              # Code formatting rules
├── docker-compose.yml       # Docker services definition
├── eslint.config.mjs        # ESLint configuration
├── librechat.example.yaml   # LibreChat config example
├── package.json             # ← Root package.json (workspaces)
├── package-lock.json        # Locked dependency versions
└── README.md                # Project README
```

### Key Root Files

**package.json**
- Defines npm workspaces: `client`, `api`, `packages/*`
- Contains root-level scripts: `backend`, `frontend`, `test`, etc.
- Development dependencies (ESLint, Prettier, Playwright)

**docker-compose.yml**
- Defines services: MongoDB, Redis, LibreChat API, Client
- Used for local development with Docker

**.env.example**
- Template for environment variables
- Copy to `.env` and configure

**eslint.config.mjs**
- Linting rules for the entire project
- Shared across client and API

---

## Frontend (client/)

The React single-page application.

```
client/
├── public/                      # Static assets
│   ├── assets/                 # Images, logos, icons
│   └── index.html              # HTML template
│
├── scripts/                     # Build scripts
│   └── post-build.cjs          # Post-build processing
│
├── src/                         # ← Source code (main work here)
│   ├── @types/                 # TypeScript type definitions
│   │   ├── global.d.ts         # Global types
│   │   └── ...                 # Custom type definitions
│   │
│   ├── Providers/              # React context providers
│   │   ├── ThemeProvider.tsx   # Theme/dark mode
│   │   ├── QueryProvider.tsx   # TanStack Query setup
│   │   └── ...
│   │
│   ├── a11y/                   # Accessibility utilities
│   │
│   ├── common/                 # Shared utilities
│   │   ├── types.ts            # Common TypeScript types
│   │   └── ...
│   │
│   ├── components/             # ← React components (90% of work)
│   │   ├── Agents/             # Agent management UI
│   │   ├── Auth/               # Login/signup forms
│   │   ├── Chat/               # Main chat interface
│   │   │   ├── ChatView.tsx    # Main chat screen
│   │   │   ├── Input/          # Message input
│   │   │   ├── Messages/       # Message list
│   │   │   └── ...
│   │   ├── Conversations/      # Conversation sidebar
│   │   ├── Endpoints/          # AI provider selection
│   │   ├── Files/              # File upload/management
│   │   ├── Input/              # Chat input components
│   │   ├── Messages/           # Message display components
│   │   ├── Nav/                # Navigation bar
│   │   ├── Prompts/            # Prompt management
│   │   ├── SidePanel/          # Side panels (tools, files)
│   │   ├── ui/                 # Reusable UI components (Radix)
│   │   │   ├── Button.tsx
│   │   │   ├── Dialog.tsx
│   │   │   ├── Input.tsx
│   │   │   └── ...
│   │   └── index.ts            # Component exports
│   │
│   ├── constants/              # Constants and enums
│   │   ├── endpoints.ts        # AI endpoint configs
│   │   └── ...
│   │
│   ├── data-provider/          # ← API client (TanStack Query hooks)
│   │   ├── queries.ts          # useQuery hooks
│   │   ├── mutations.ts        # useMutation hooks
│   │   └── ...
│   │
│   ├── hooks/                  # ← Custom React hooks
│   │   ├── useAuth.ts          # Authentication hook
│   │   ├── useMessages.ts      # Message management
│   │   ├── useConversations.ts # Conversation management
│   │   └── ...
│   │
│   ├── locales/                # Internationalization (i18n)
│   │   ├── en.json             # English translations
│   │   ├── es.json             # Spanish translations
│   │   └── ...
│   │
│   ├── routes/                 # ← React Router routes/pages
│   │   ├── Root.tsx            # Layout wrapper
│   │   ├── Chat.tsx            # Chat page
│   │   ├── Login.tsx           # Login page
│   │   └── ...
│   │
│   ├── store/                  # ← Global state (Jotai/Recoil)
│   │   ├── atoms.ts            # State atoms
│   │   ├── selectors.ts        # Derived state
│   │   └── ...
│   │
│   ├── utils/                  # Utility functions
│   │   ├── api.ts              # API client helpers
│   │   ├── validation.ts       # Form validation
│   │   └── ...
│   │
│   ├── App.tsx                 # ← Main App component
│   ├── index.tsx               # ← Entry point
│   └── main.tsx                # Render entry
│
├── test/                        # Frontend tests
│   ├── setup.ts                # Jest setup
│   └── ...
│
├── babel.config.cjs            # Babel configuration
├── jest.config.cjs             # Jest testing config
├── package.json                # ← Frontend dependencies
├── postcss.config.cjs          # PostCSS config (Tailwind)
├── tailwind.config.cjs         # ← Tailwind CSS configuration
├── tsconfig.json               # TypeScript configuration
└── vite.config.ts              # ← Vite build configuration
```

### 🎯 Key Frontend Files

| File | Purpose | When to Edit |
|------|---------|--------------|
| `src/App.tsx` | Root component, routing setup | Rarely (add top-level providers) |
| `src/routes/Chat.tsx` | Main chat page | Often (main feature work) |
| `src/components/Chat/` | Chat interface components | Very often (UI features) |
| `src/hooks/` | Custom React hooks | Often (reusable logic) |
| `src/data-provider/` | API calls (TanStack Query) | Often (new API endpoints) |
| `src/store/` | Global state atoms | Sometimes (global state) |
| `tailwind.config.cjs` | Tailwind customization | Sometimes (design tokens) |
| `vite.config.ts` | Build configuration | Rarely (build optimization) |

### 🌉 Frontend Patterns

**Component Organization:**
```
components/
├── FeatureName/           # Feature-based folders
│   ├── FeatureName.tsx   # Main component
│   ├── SubComponent.tsx  # Sub-components
│   ├── hooks.ts          # Feature-specific hooks
│   └── index.ts          # Exports
```

**File Naming:**
- Components: `PascalCase.tsx` (e.g., `ChatView.tsx`)
- Hooks: `camelCase.ts` with `use` prefix (e.g., `useAuth.ts`)
- Utils: `camelCase.ts` (e.g., `validation.ts`)
- Constants: `UPPER_SNAKE_CASE` or `camelCase.ts`

---

## Backend (api/)

The Express REST API server.

```
api/
├── app/                         # Express app initialization
│   ├── index.js                # Express app setup
│   └── ...
│
├── cache/                       # Caching utilities
│   ├── getLogStores.js         # Log storage
│   └── ...
│
├── config/                      # Configuration
│   ├── endpoints.js            # AI endpoint configs
│   ├── parsers.js              # Request parsers
│   └── ...
│
├── db/                          # Database utilities
│   ├── connectDb.js            # MongoDB connection
│   └── ...
│
├── lib/                         # Shared libraries
│   ├── utils/                  # Utility functions
│   └── ...
│
├── models/                      # ← Mongoose schemas/models
│   ├── Agent.js                # AI Agent model
│   ├── Conversation.js         # Chat conversation
│   ├── Message.js              # Chat message
│   ├── File.js                 # File upload
│   ├── Prompt.js               # Saved prompt
│   ├── Transaction.js          # Usage tracking
│   ├── Role.js                 # User roles/permissions
│   └── index.js                # Model exports
│
├── server/                      # ← Main server code
│   ├── controllers/            # ← Request handlers
│   │   ├── AuthController.js   # Auth endpoints
│   │   ├── ConversationController.js
│   │   ├── MessageController.js
│   │   ├── FileController.js
│   │   └── ...
│   │
│   ├── middleware/             # ← Express middleware
│   │   ├── requireJwtAuth.js   # JWT authentication
│   │   ├── validateRequest.js  # Request validation
│   │   ├── errorHandler.js     # Error handling
│   │   ├── rateLimiter.js      # Rate limiting
│   │   └── ...
│   │
│   ├── routes/                 # ← API routes (URLs)
│   │   ├── auth.js             # /api/auth/*
│   │   ├── messages.js         # /api/messages/*
│   │   ├── conversations.js    # /api/conversations/*
│   │   ├── files.js            # /api/files/*
│   │   ├── agents.js           # /api/agents/*
│   │   └── index.js            # Route registration
│   │
│   ├── services/               # ← Business logic
│   │   ├── AuthService.js      # Auth logic
│   │   ├── MessageService.js   # Message logic
│   │   ├── ConversationService.js
│   │   ├── AIService.js        # AI provider integration
│   │   └── ...
│   │
│   ├── utils/                  # Server utilities
│   │   ├── logger.js           # Winston logger
│   │   ├── tokens.js           # JWT utilities
│   │   └── ...
│   │
│   ├── cleanup.js              # Cleanup tasks
│   ├── index.js                # ← Server entry point
│   └── socialLogins.js         # OAuth setup
│
├── strategies/                  # Passport.js strategies
│   ├── jwtStrategy.js          # JWT authentication
│   ├── localStrategy.js        # Username/password
│   ├── googleStrategy.js       # Google OAuth
│   └── ...
│
├── test/                        # Backend tests
│   ├── services/               # Service tests
│   └── ...
│
├── utils/                       # API utilities
│
├── jest.config.js              # Jest configuration
└── package.json                # ← Backend dependencies
```

### 🎯 Key Backend Files

| File | Purpose | When to Edit |
|------|---------|--------------|
| `server/index.js` | Server entry point | Rarely (startup logic) |
| `server/routes/` | API endpoints | Often (new routes) |
| `server/controllers/` | Request handlers | Often (endpoint logic) |
| `server/services/` | Business logic | Very often (features) |
| `server/middleware/` | Middleware functions | Sometimes (cross-cutting concerns) |
| `models/` | Database schemas | Often (data models) |
| `db/connectDb.js` | Database connection | Rarely (DB config) |

### 🌉 Backend Patterns

**MVC-ish Architecture:**
```
Request → Route → Middleware → Controller → Service → Model → Database
                                    ↓
                                Response
```

**File Organization:**
```
server/
├── routes/
│   └── messages.js          # Defines: POST /api/messages
├── controllers/
│   └── MessageController.js # Handles request, calls service
├── services/
│   └── MessageService.js    # Business logic, DB operations
└── models/
    └── Message.js           # Mongoose schema
```

**Naming Conventions:**
- Routes: `camelCase.js` (e.g., `messages.js`)
- Controllers: `PascalCase.js` (e.g., `MessageController.js`)
- Services: `PascalCase.js` (e.g., `MessageService.js`)
- Models: `PascalCase.js` (e.g., `Message.js`)
- Middleware: `camelCase.js` (e.g., `requireJwtAuth.js`)

---

## Shared Packages (packages/)

Code shared between frontend and backend.

```
packages/
├── api/                         # Shared API utilities
│   ├── src/
│   │   └── ...                 # API-related utilities
│   ├── package.json
│   └── rollup.config.js        # Build config
│
├── client/                      # Shared client utilities
│   ├── src/
│   │   └── ...                 # Client-related utilities
│   ├── package.json
│   └── rollup.config.js
│
├── data-provider/               # ← API client & React Query hooks
│   ├── src/
│   │   ├── queries.ts          # useQuery hooks
│   │   ├── mutations.ts        # useMutation hooks
│   │   └── api-client.ts       # Fetch wrapper
│   ├── react-query/            # React Query specific
│   ├── package.json
│   └── rollup.config.js
│
└── data-schemas/                # ← Shared TypeScript types (Zod)
    ├── src/
    │   ├── types.ts            # TypeScript interfaces
    │   ├── schemas.ts          # Zod validation schemas
    │   └── ...
    ├── package.json
    └── rollup.config.js
```

### Why Shared Packages?

**Without shared packages:**
```javascript
// client/src/types.ts
interface Message { id: string; text: string }

// api/models/Message.js
// Duplicate definition! Could diverge over time.
```

**With shared packages:**
```javascript
// packages/data-schemas/src/types.ts
export interface Message { id: string; text: string }

// Both import from same source:
import { Message } from '@librechat/data-schemas'
```

**Benefits:**
- ✅ **Single source of truth** - Types defined once
- ✅ **Type safety** - Frontend/backend stay in sync
- ✅ **Reusability** - Share validation logic
- ✅ **Consistency** - API contracts enforced

---

## Configuration Files

### Docker

**docker-compose.yml**
- Defines services: MongoDB, Redis, API, Client
- Used with `docker compose up`

**Dockerfile**
- Builds LibreChat Docker image
- Multi-stage build (smaller final image)

### TypeScript

**tsconfig.json** (multiple)
- `client/tsconfig.json` - Frontend TS config
- `packages/*/tsconfig.json` - Package TS configs
- Compiler options, path aliases, etc.

### Build Tools

**vite.config.ts** (`client/`)
- Vite configuration
- Plugins, build options, dev server

**rollup.config.js** (`packages/*/`)
- Build configuration for shared packages
- Bundles packages for distribution

### Linting & Formatting

**eslint.config.mjs** (root)
- ESLint rules for entire monorepo
- Shared across client & API

**.prettierrc** (root)
- Code formatting rules
- Consistent code style

### Testing

**jest.config.js** (multiple)
- `api/jest.config.js` - Backend tests
- `client/jest.config.cjs` - Frontend tests
- `packages/*/jest.config.js` - Package tests

**playwright.config.ts** (`e2e/`)
- E2E test configuration
- Browser, timeout, reporters

---

## Common Patterns

### Import Path Aliases

**Frontend (`client/`):**
```javascript
// Instead of: ../../../components/ui/Button
import { Button } from '~/components/ui/Button'
import { useAuth } from '~/hooks/useAuth'
```

Configured in `vite.config.ts`:
```javascript
resolve: {
  alias: {
    '~': path.resolve(__dirname, 'src')
  }
}
```

**Backend (`api/`):**
```javascript
// Instead of: ../../../models/Message
import Message from '~/models/Message'
import { logger } from '~/server/utils/logger'
```

Configured in `package.json`:
```json
"imports": {
  "~/*": "./*"
}
```

### Monorepo Package References

**In package.json:**
```json
{
  "dependencies": {
    "@librechat/data-provider": "*",    // ← Workspace package
    "@librechat/data-schemas": "*",     // ← Workspace package
    "react": "^18.2.0"                  // ← npm package
  }
}
```

The `*` means "use the local workspace version."

---

## Where to Find Things

### "I want to add a new React component"
→ `client/src/components/[FeatureName]/`

### "I want to add a new API endpoint"
→ `api/server/routes/` (define route)
→ `api/server/controllers/` (handle request)
→ `api/server/services/` (business logic)

### "I want to add a database field"
→ `api/models/` (update Mongoose schema)

### "I want to add a custom hook"
→ `client/src/hooks/`

### "I want to add global state"
→ `client/src/store/atoms.ts`

### "I want to modify styles"
→ Use Tailwind classes in components
→ `client/tailwind.config.cjs` for theme customization

### "I want to add API client logic"
→ `packages/data-provider/src/`

### "I want to add shared types"
→ `packages/data-schemas/src/`

### "I want to add authentication logic"
→ Backend: `api/server/services/AuthService.js`
→ Frontend: `client/src/hooks/useAuth.ts`

### "I want to add tests"
→ Frontend: `client/test/`
→ Backend: `api/test/`
→ E2E: `e2e/specs/`

### "I want to configure environment variables"
→ `.env` (copy from `.env.example`)

### "I want to modify build configuration"
→ Frontend: `client/vite.config.ts`
→ Backend: `api/package.json` (scripts)

---

## Import Paths

### Frontend Import Examples

```javascript
// Components
import { Button } from '~/components/ui/Button'
import ChatView from '~/components/Chat/ChatView'

// Hooks
import { useAuth } from '~/hooks/useAuth'
import { useMessages } from '~/hooks/useMessages'

// Utils
import { formatDate } from '~/utils/date'

// Store (Jotai)
import { userAtom } from '~/store/atoms'

// Constants
import { ENDPOINTS } from '~/constants/endpoints'

// API client (shared package)
import { useMessagesQuery } from '@librechat/data-provider'

// Types (shared package)
import type { Message } from '@librechat/data-schemas'
```

### Backend Import Examples

```javascript
// Models
import Message from '~/models/Message'
import Conversation from '~/models/Conversation'

// Services
import MessageService from '~/server/services/MessageService'

// Middleware
import requireJwtAuth from '~/server/middleware/requireJwtAuth'

// Utils
import { logger } from '~/server/utils/logger'

// Config
import { endpoints } from '~/config/endpoints'

// External packages
const express = require('express')
const mongoose = require('mongoose')
```

---

## File Size Guidelines

**Component files:**
- < 200 lines: ✅ Good
- 200-400 lines: ⚠️ Consider splitting
- \> 400 lines: 🚨 Definitely split

**Service files:**
- < 300 lines: ✅ Good
- 300-500 lines: ⚠️ Consider splitting by responsibility
- \> 500 lines: 🚨 Split into multiple services

**When to split:**
- File has multiple responsibilities
- Hard to navigate
- Lots of unrelated functions
- Complex enough to need its own folder

---

## Visual Directory Map

```
LibreChat (Root)
│
├─── 🎨 client/                    Frontend (React SPA)
│    ├─── src/
│    │    ├─── components/         UI components
│    │    ├─── hooks/              React hooks
│    │    ├─── routes/             Pages/routes
│    │    └─── store/              Global state
│    └─── package.json
│
├─── 🔧 api/                       Backend (Express API)
│    ├─── server/
│    │    ├─── routes/             API endpoints
│    │    ├─── controllers/        Request handlers
│    │    ├─── services/           Business logic
│    │    └─── middleware/         Middleware
│    ├─── models/                  Database schemas
│    └─── package.json
│
├─── 📦 packages/                  Shared code
│    ├─── data-provider/           API client
│    ├─── data-schemas/            Types
│    ├─── api/                     Shared API utils
│    └─── client/                  Shared client utils
│
├─── 🧪 e2e/                       E2E tests
│
├─── 🛠️ config/                    CLI scripts
│
└─── 📄 [config files]             Docker, ESLint, etc.
```

---

## Quick Reference Table

| I want to... | Go to... |
|--------------|----------|
| Add UI component | `client/src/components/` |
| Add React hook | `client/src/hooks/` |
| Add API endpoint | `api/server/routes/` |
| Add business logic | `api/server/services/` |
| Add database model | `api/models/` |
| Add middleware | `api/server/middleware/` |
| Add shared type | `packages/data-schemas/` |
| Add API client hook | `packages/data-provider/` |
| Add global state | `client/src/store/` |
| Add test (frontend) | `client/test/` |
| Add test (backend) | `api/test/` |
| Add E2E test | `e2e/specs/` |
| Configure build | `client/vite.config.ts` |
| Configure env vars | `.env` |
| Configure Tailwind | `client/tailwind.config.cjs` |

---

## Next Steps

Now that you know where everything lives:

1. **[DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)** - Follow data through the structure
2. **[FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)** - Deep dive into client/
3. **[BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)** - Deep dive into api/
4. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Step-by-step common tasks

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
