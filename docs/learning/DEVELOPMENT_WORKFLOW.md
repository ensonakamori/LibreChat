# ⚙️ Development Workflow

**Documented:** November 19, 2025
**Target:** Contributors working on LibreChat
**Time Estimate:** Reference guide (use daily)
**Difficulty:** 🟢 Beginner

---

## Daily Development Practices

This guide covers the daily workflow for developing features, fixing bugs, and contributing to LibreChat.

---

## Table of Contents

- [Initial Setup](#initial-setup)
- [Starting Development](#starting-development)
- [Git Workflow](#git-workflow)
- [Making Changes](#making-changes)
- [Testing Your Changes](#testing-your-changes)
- [Code Quality Checks](#code-quality-checks)
- [Committing Changes](#committing-changes)
- [Creating Pull Requests](#creating-pull-requests)
- [Code Review Process](#code-review-process)
- [After Merge](#after-merge)
- [Daily Commands Reference](#daily-commands-reference)

---

## Initial Setup

### 1. Fork and Clone

```bash
# Fork repository on GitHub (click "Fork" button)

# Clone YOUR fork
git clone https://github.com/YOUR_USERNAME/LibreChat.git
cd LibreChat

# Add upstream remote (original repo)
git remote add upstream https://github.com/danny-avila/LibreChat.git

# Verify remotes
git remote -v
# origin    https://github.com/YOUR_USERNAME/LibreChat.git
# upstream  https://github.com/danny-avila/LibreChat.git
```

### 2. Install Dependencies

```bash
# Install all dependencies (monorepo)
npm install

# Or install individually
npm install --prefix client
npm install --prefix api
```

### 3. Setup Environment

```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your configuration
# - Database URLs
# - API keys
# - Ports
```

### 4. Start Development Servers

```bash
# Option 1: Start all services with Docker
npm run docker:dev

# Option 2: Start manually
# Terminal 1: MongoDB
mongod --dbpath ./data/db

# Terminal 2: Redis
redis-server

# Terminal 3: Backend
npm run backend:dev

# Terminal 4: Frontend
npm run frontend:dev
```

---

## Starting Development

### 1. Pull Latest Changes

**Before starting any work:**

```bash
# Switch to main branch
git checkout main

# Pull latest from upstream
git pull upstream main

# Update your fork
git push origin main

# Update dependencies (if package.json changed)
npm install
```

### 2. Check for Issues

**Look for:**
- Assigned issue (comment "I'd like to work on this")
- Good first issues: `label:good-first-issue`
- Help wanted: `label:help-wanted`

```bash
# Filter issues on GitHub
https://github.com/danny-avila/LibreChat/issues?q=is:issue+is:open+label:"good+first+issue"
```

---

## Git Workflow

### 1. Create Feature Branch

```bash
# Create and switch to new branch
git checkout -b feat/add-message-reactions

# Or for bug fixes
git checkout -b fix/message-duplication

# Branch naming convention:
# feat/    - New feature
# fix/     - Bug fix
# docs/    - Documentation
# refactor/ - Code refactoring
# test/    - Tests
# chore/   - Maintenance
```

### 2. Make Changes

```bash
# See what files changed
git status

# See specific changes
git diff

# See changes in specific file
git diff client/src/components/Chat/Message.tsx
```

### 3. Stage Changes

```bash
# Stage specific files
git add client/src/components/Chat/Message.tsx

# Stage all changes
git add .

# Stage all files in directory
git add client/src/components/Chat/

# Unstage file
git restore --staged file.txt
```

### 4. Commit Changes

```bash
# Commit with message
git commit -m "feat(chat): add message reactions

- Added reaction picker component
- Updated Message model with reactions field
- Added API endpoint for adding reactions

Closes #123"

# Amend last commit (if you forgot something)
git add forgotten-file.txt
git commit --amend --no-edit
```

### 5. Push to Your Fork

```bash
# First push (creates remote branch)
git push -u origin feat/add-message-reactions

# Subsequent pushes
git push
```

### 6. Keep Branch Updated

```bash
# If main branch has new commits

# Option 1: Rebase (preferred)
git checkout main
git pull upstream main
git checkout feat/add-message-reactions
git rebase main

# Resolve conflicts if any
# Then:
git push --force-with-lease

# Option 2: Merge
git checkout feat/add-message-reactions
git merge main
git push
```

---

## Making Changes

### Frontend Changes

**1. Add/modify component:**

```bash
# Edit component
code client/src/components/Chat/Message.tsx

# Check changes in browser
# App should hot-reload automatically

# If not reloading:
# Stop server (Ctrl+C) and restart
npm run frontend:dev
```

**2. Add new dependency:**

```bash
# Install in client
npm install --prefix client react-icons

# Import in code
import { FaHeart } from 'react-icons/fa';
```

**3. Check TypeScript errors:**

```bash
# Type check
npm run type-check

# Or let IDE (VS Code) show errors
```

### Backend Changes

**1. Add/modify endpoint:**

```bash
# Edit route/controller/service
code api/server/routes/messages.js
code api/server/controllers/messagesController.js
code api/server/services/messageService.js

# Restart backend server
# Ctrl+C in backend terminal
npm run backend:dev
```

**2. Add new dependency:**

```bash
# Install in api
npm install --prefix api express-rate-limit

# Import and use
const rateLimit = require('express-rate-limit');
```

**3. Database changes:**

```bash
# Edit model
code api/models/Message.js

# If schema changed significantly, create migration
mkdir -p migrations
code migrations/add-reactions-field.js
```

---

## Testing Your Changes

### Manual Testing

**Frontend:**
1. Open http://localhost:3000
2. Test user flow manually
3. Check browser console for errors
4. Test in different browsers (Chrome, Firefox, Safari)
5. Test responsive design (mobile, tablet)

**Backend:**
```bash
# Test API endpoint with curl
curl -X POST http://localhost:3080/api/messages \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"text": "Test message", "conversationId": "123"}'

# Or use Postman/Insomnia
```

### Automated Tests

**Run all tests:**
```bash
# Backend tests
npm test

# Frontend tests
npm run test:frontend

# E2E tests
npm run test:e2e
```

**Run specific tests:**
```bash
# Single test file
npm test -- messageService.test.js

# Tests matching pattern
npm test -- messages

# Watch mode (re-run on changes)
npm test -- --watch
```

**Run with coverage:**
```bash
npm test -- --coverage

# View coverage report
open coverage/lcov-report/index.html
```

### Add Tests for Your Changes

**Frontend test example:**
```typescript
// client/src/components/Chat/Message.test.tsx
import { render, screen } from '@testing-library/react';
import { Message } from './Message';

describe('Message with reactions', () => {
  it('displays reaction count', () => {
    const message = {
      id: '1',
      text: 'Hello',
      reactions: [{ type: 'heart', count: 5 }]
    };

    render(<Message message={message} />);

    expect(screen.getByText('❤️ 5')).toBeInTheDocument();
  });
});
```

**Backend test example:**
```javascript
// api/server/services/messageService.test.js
const messageService = require('./messageService');

describe('addReaction', () => {
  it('adds reaction to message', async () => {
    const result = await messageService.addReaction({
      messageId: '123',
      userId: 'user1',
      reactionType: 'heart'
    });

    expect(result.reactions).toContainEqual({
      type: 'heart',
      count: 1
    });
  });
});
```

---

## Code Quality Checks

### Before Committing

**1. Lint code:**
```bash
# Check for lint errors
npm run lint

# Auto-fix lint errors
npm run lint:fix

# Check specific file
npx eslint client/src/components/Chat/Message.tsx
```

**2. Format code:**
```bash
# Format all files
npm run format

# Check formatting
npm run format:check

# Format specific file
npx prettier --write client/src/components/Chat/Message.tsx
```

**3. Type check:**
```bash
# Check TypeScript errors
npm run type-check

# Check specific file
npx tsc --noEmit client/src/components/Chat/Message.tsx
```

### Pre-commit Hook

**LibreChat may have pre-commit hooks via Husky:**

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

**Hooks automatically run on `git commit`**

---

## Committing Changes

### Commit Message Format

**Format:** `type(scope): subject`

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Add/update tests
- `chore`: Maintenance

**Examples:**
```bash
# Good commits
git commit -m "feat(chat): add message reactions"
git commit -m "fix(auth): resolve token expiration bug"
git commit -m "docs(readme): update installation instructions"
git commit -m "refactor(api): extract message service layer"
git commit -m "test(messages): add unit tests for reactions"

# Bad commits
git commit -m "fixes"
git commit -m "updated stuff"
git commit -m "wip"
```

### Multi-line Commits

```bash
git commit -m "feat(chat): add message reactions

- Added ReactionPicker component
- Updated Message model with reactions array
- Created API endpoint POST /api/messages/:id/reactions
- Added tests for reaction functionality

Closes #123"
```

### When to Commit

**Commit frequently:**
- After completing a logical unit of work
- Before switching tasks
- At end of day (even if incomplete)

**Use descriptive messages:**
- Explain WHAT changed
- Explain WHY (if not obvious)

---

## Creating Pull Requests

### 1. Push to Your Fork

```bash
git push origin feat/add-message-reactions
```

### 2. Create PR on GitHub

1. Go to your fork: `https://github.com/YOUR_USERNAME/LibreChat`
2. Click "Compare & pull request"
3. Fill out template:

```markdown
## Description
Added message reactions feature allowing users to react to messages with emojis.

## Changes
- Added `ReactionPicker` component
- Updated `Message` model with `reactions` field
- Created API endpoint: `POST /api/messages/:id/reactions`
- Added frontend and backend tests

## Testing
- [x] Manual testing in browser
- [x] Added unit tests
- [x] All existing tests pass
- [x] Tested on mobile viewport
- [x] No console errors

## Screenshots
![Message with reactions](https://...)

## Closes
Closes #123
```

4. Click "Create pull request"

### 3. Wait for CI Checks

**Automated checks run:**
- Lint
- Type check
- Tests
- Build

**If checks fail:**
```bash
# View error details on GitHub
# Fix issues locally
git add .
git commit -m "fix: resolve CI errors"
git push
# PR automatically updates
```

---

## Code Review Process

### Reviewer Feedback

**Common feedback:**
- "Please add types"
- "Extract this to a function"
- "Add error handling"
- "Write tests for this"

**How to respond:**

**Good:**
```markdown
Good catch! I've added types and error handling. Let me know if this looks better.
```

**Bad:**
```markdown
That's not needed
I disagree
Works fine for me
```

### Addressing Feedback

```bash
# Make requested changes
code client/src/components/Chat/Message.tsx

# Commit changes
git add .
git commit -m "refactor: address review feedback

- Added TypeScript types
- Extracted helper function
- Added error handling"

# Push to PR
git push
# PR updates automatically
```

### Requesting Re-review

**After addressing all feedback:**
- Click "Re-request review" button on GitHub
- Or comment: "Ready for re-review"

---

## After Merge

### 1. Clean Up

```bash
# Switch to main
git checkout main

# Pull merged changes
git pull upstream main

# Update your fork
git push origin main

# Delete feature branch (optional)
git branch -d feat/add-message-reactions
git push origin --delete feat/add-message-reactions
```

### 2. Celebrate! 🎉

You're now a LibreChat contributor!

---

## Daily Commands Reference

### Quick Start

```bash
# Start development
npm run dev
```

### Git Commands

```bash
# Status and diff
git status
git diff

# Create branch
git checkout -b feat/my-feature

# Stage and commit
git add .
git commit -m "feat: add feature"

# Push
git push origin feat/my-feature

# Update from main
git checkout main && git pull upstream main
git checkout feat/my-feature && git rebase main
```

### Testing

```bash
# Run tests
npm test
npm run test:frontend
npm run test:e2e

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage
```

### Code Quality

```bash
# Lint
npm run lint
npm run lint:fix

# Format
npm run format

# Type check
npm run type-check
```

### Build

```bash
# Build for production
npm run build

# Check build size
npm run analyze
```

---

## Troubleshooting

### Port Already in Use

```bash
# Find process
lsof -ti:3080

# Kill process
kill -9 $(lsof -ti:3080)
```

### Dependencies Out of Sync

```bash
# Delete and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Database Issues

```bash
# Reset database (WARNING: deletes all data)
mongosh
> use librechat
> db.dropDatabase()
```

### Git Conflicts

```bash
# If rebase conflicts
# 1. Fix conflicts in files
# 2. Stage resolved files
git add .

# 3. Continue rebase
git rebase --continue

# Or abort and try merge instead
git rebase --abort
git merge main
```

---

## Summary

**Development Workflow:**

1. ✅ Pull latest changes
2. ✅ Create feature branch
3. ✅ Make changes
4. ✅ Test thoroughly
5. ✅ Lint and format
6. ✅ Commit with good message
7. ✅ Push to fork
8. ✅ Create PR
9. ✅ Address feedback
10. ✅ Merge and celebrate!

**Remember:**
- Commit frequently
- Test everything
- Write good commit messages
- Be responsive to feedback
- Ask questions when stuck

---

**Next Steps:**

- Read [TESTING_GUIDE.md](./TESTING_GUIDE.md) - Testing strategies
- Read [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md) - Debugging techniques
- Start contributing!

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
