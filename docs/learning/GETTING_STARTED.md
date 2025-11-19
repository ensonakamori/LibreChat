# 🚀 Getting Started with LibreChat

**Documented:** November 19, 2025
**Target:** Mid-level React developers new to backend
**Time Estimate:** 1-3 hours (depending on your system)
**Difficulty:** 🟢 Beginner

---

## Table of Contents

- [What You'll Accomplish](#what-youll-accomplish)
- [Prerequisites](#prerequisites)
- [Method 1: Docker Setup (Recommended)](#method-1-docker-setup-recommended)
- [Method 2: Local Installation](#method-2-local-installation)
- [Configuration](#configuration)
- [First Run](#first-run)
- [Verification](#verification)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## What You'll Accomplish

By the end of this guide, you'll have:

✅ LibreChat running on `http://localhost:3080`
✅ MongoDB database running and connected
✅ Redis cache running and connected
✅ A user account created
✅ The ability to chat with AI (if you have API keys)
✅ Development environment ready for code changes

---

## Prerequisites

### Required Knowledge
- ✅ Basic command line usage (cd, ls, running commands)
- ✅ Git basics (clone, pull, checkout)
- ✅ Text editing (for .env file)

### Required Software

| Software | Minimum Version | Check Command | Install Link |
|----------|----------------|---------------|--------------|
| **Node.js** | 20.19+ (LTS) | `node --version` | [nodejs.org](https://nodejs.org/) |
| **npm** | 10+ | `npm --version` | Included with Node.js |
| **Git** | 2.x | `git --version` | [git-scm.com](https://git-scm.com/) |
| **Docker** (Optional) | 20+ | `docker --version` | [docker.com](https://www.docker.com/get-started) |

**For Local Installation (if not using Docker):**
| Software | Minimum Version | Install Link |
|----------|----------------|--------------|
| **MongoDB** | 6.0+ | [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community) |
| **Redis** | 7.0+ | [redis.io/download](https://redis.io/download) |

### Check Your Node.js Version

```bash
node --version
# Should show v20.x.x or v22.x.x or v24.x.x
```

**If your version is older than v20:**
- Download from [nodejs.org](https://nodejs.org/)
- Choose the "LTS" version (Long Term Support)
- Or use [nvm](https://github.com/nvm-sh/nvm) to manage multiple Node versions

### 🧠 Mental Model: Why Do We Need This?

**For React developers:**

- **Node.js** - JavaScript runtime for the backend server (like your browser but for servers)
- **npm** - Package manager (like you already use for React projects)
- **MongoDB** - Database to store conversations, messages, users (unlike localStorage, this persists forever)
- **Redis** - Fast in-memory cache for sessions and temporary data (think: super-fast object storage)
- **Docker** - (Optional) Runs MongoDB + Redis in containers (easier than installing separately)

---

## Method 1: Docker Setup (Recommended)

**Why Docker?**
- ✅ Easiest setup (MongoDB + Redis automatically configured)
- ✅ Isolated from your system (won't conflict with other software)
- ✅ Matches production environment
- ✅ Easy to reset/cleanup

### Step 1: Install Docker

**Mac:** Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
**Windows:** Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
**Linux:** Follow [official Docker installation guide](https://docs.docker.com/engine/install/)

**Verify Docker is installed:**
```bash
docker --version
# Should show: Docker version 20.x.x or higher

docker compose version
# Should show: Docker Compose version v2.x.x or higher
```

### Step 2: Clone the Repository

```bash
# Navigate to where you want the project
cd ~/projects  # or wherever you keep code

# Clone LibreChat
git clone https://github.com/danny-avila/LibreChat.git

# Enter the directory
cd LibreChat
```

### Step 3: Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env
```

**Open `.env` in your text editor:**

```bash
# VS Code
code .env

# Or use any text editor
nano .env
# vim .env
# open .env (Mac default editor)
```

**Required changes:**

```bash
# MongoDB Connection (Docker automatically sets this up)
MONGO_URI=mongodb://mongodb:27017/LibreChat

# Server URLs
DOMAIN_CLIENT=http://localhost:3080
DOMAIN_SERVER=http://localhost:3080

# Session Secret (IMPORTANT: Change this!)
SESSION_SECRET=your-super-secret-key-here-change-this-to-random-string

# JWT Secret (IMPORTANT: Change this!)
JWT_SECRET=another-super-secret-key-change-this-too

# For AI chat functionality (optional for development)
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
```

**🎯 Remember This:**
**SESSION_SECRET** and **JWT_SECRET** should be **long, random strings**. In production, these protect your users' sessions. For development, use anything, but change them from the defaults!

**Quick random secret generator:**
```bash
# On Mac/Linux:
openssl rand -base64 32

# On Windows (PowerShell):
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))
```

### Step 4: Start with Docker Compose

```bash
# Start all services (MongoDB, Redis, LibreChat)
docker compose up -d

# The -d flag means "detached" (runs in background)
```

**What's happening:**
- 🐳 Docker pulls required images (first time only, takes 5-10 min)
- 🗄️ MongoDB starts on port 27017
- 🔴 Redis starts on port 6379
- 🚀 LibreChat API starts on port 3080 (backend)
- ⚛️ LibreChat Client builds and starts (frontend)

**View logs (optional):**
```bash
# Watch logs in real-time
docker compose logs -f

# View logs for specific service
docker compose logs -f api
docker compose logs -f mongodb
```

**🔍 Troubleshooting:**
If services fail to start:
```bash
# Check which services are running
docker compose ps

# Restart all services
docker compose restart

# Stop and remove everything (fresh start)
docker compose down
docker compose up -d
```

### Step 5: Wait for Build to Complete

**First time:** The client needs to build (5-10 minutes)

```bash
# Check build progress
docker compose logs -f client

# You'll see Vite building the React app
# Wait for: "VITE v6.x.x  ready in xxxms"
```

**When ready, you'll see:**
```
client_1  |   VITE v6.4.1  ready in 1234 ms
client_1  |   ➜  Local:   http://localhost:3090
```

### Step 6: Access LibreChat

Open your browser to: **http://localhost:3080**

You should see the LibreChat login/register page! 🎉

---

## Method 2: Local Installation

**Use this if:**
- You want more control
- You already have MongoDB/Redis installed
- You don't want to use Docker
- You're comfortable with backend tooling

### Step 1: Install MongoDB

**Mac (with Homebrew):**
```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
brew services start mongodb-community@7.0
```

**Windows:**
- Download from [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
- Run the installer
- Choose "Complete" installation
- Install as a service

**Linux (Ubuntu/Debian):**
```bash
# Import MongoDB GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -sc)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install
sudo apt-get update
sudo apt-get install -y mongodb-org

# Start service
sudo systemctl start mongod
sudo systemctl enable mongod
```

**Verify MongoDB is running:**
```bash
# Try connecting
mongosh
# You should see: "Current Mongosh Log ID: ..."
# Type: exit

# Or check if it's listening
netstat -an | grep 27017
# Should show: tcp4  0  0  127.0.0.1.27017  *.*  LISTEN
```

### Step 2: Install Redis

**Mac (with Homebrew):**
```bash
brew install redis
brew services start redis
```

**Windows:**
- Download from [github.com/tporadowski/redis/releases](https://github.com/tporadowski/redis/releases)
- Or use WSL (Windows Subsystem for Linux)

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

**Verify Redis is running:**
```bash
# Try connecting
redis-cli ping
# Should respond: PONG

# Or check if it's listening
netstat -an | grep 6379
```

### Step 3: Clone and Install Dependencies

```bash
# Clone the repository
git clone https://github.com/danny-avila/LibreChat.git
cd LibreChat

# Install dependencies (this takes 3-5 minutes)
npm install

# Or use the update script (recommended)
npm run update
```

**What's happening:**
- 📦 npm installs **all packages** for the monorepo (root + client + api + packages)
- This includes React, Express, MongoDB driver, and 500+ other packages
- First time takes longer (downloading packages)

### Step 4: Configure Environment

```bash
# Copy example .env
cp .env.example .env

# Edit .env
code .env  # or your preferred editor
```

**Required configuration:**

```bash
# MongoDB (local installation)
MONGO_URI=mongodb://127.0.0.1:27017/LibreChat

# Redis (local installation)
# Redis is used automatically on localhost:6379 (default)

# Server configuration
HOST=localhost
PORT=3080
DOMAIN_CLIENT=http://localhost:3080
DOMAIN_SERVER=http://localhost:3080

# Secrets (CHANGE THESE!)
SESSION_SECRET=your-random-secret-here
JWT_SECRET=your-other-random-secret-here
JWT_REFRESH_SECRET=your-refresh-secret-here

# Optional: AI Provider Keys (for actual chat functionality)
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
# GOOGLE_API_KEY=...
```

### Step 5: Build and Start

**Option A: Development Mode (Hot Reload)**

```bash
# Terminal 1: Start backend
npm run backend:dev

# Terminal 2: Start frontend (in a new terminal)
npm run frontend:dev
```

**What's happening:**
- Backend runs on `http://localhost:3080/api`
- Frontend runs on `http://localhost:3090` (Vite dev server)
- Vite proxies API requests to backend
- Both auto-reload when you change code! 🔥

**Option B: Production Mode**

```bash
# Build frontend
npm run frontend

# Start backend (serves built frontend)
npm run backend
```

**What's happening:**
- Frontend is built into `client/dist/`
- Backend serves static files from `client/dist/`
- Everything runs on `http://localhost:3080`
- No auto-reload (for production-like environment)

### Step 6: Access LibreChat

**Development mode:** http://localhost:3090
**Production mode:** http://localhost:3080

You should see the login/register page! 🎉

---

## Configuration

### Understanding .env Variables

**🧠 Mental Model:** The `.env` file is like React's environment variables, but for full-stack apps

| Variable | What It Does | Example |
|----------|-------------|---------|
| `MONGO_URI` | Database connection string | `mongodb://localhost:27017/LibreChat` |
| `PORT` | Which port the server runs on | `3080` |
| `DOMAIN_CLIENT` | Frontend URL (for CORS) | `http://localhost:3080` |
| `DOMAIN_SERVER` | Backend URL | `http://localhost:3080` |
| `SESSION_SECRET` | Encrypts session data | (random string) |
| `JWT_SECRET` | Signs JWT tokens | (random string) |
| `OPENAI_API_KEY` | Your OpenAI API key | `sk-...` |
| `ANTHROPIC_API_KEY` | Your Anthropic API key | `sk-ant-...` |

**⚠️ Common Pitfall:**
Never commit `.env` to Git! It contains secrets. LibreChat's `.gitignore` already excludes it, but double-check:

```bash
git status
# .env should NOT appear in the list
```

### Optional: Add AI Provider Keys

To actually chat with AI, you need API keys:

**OpenAI (ChatGPT, GPT-4):**
1. Sign up at [platform.openai.com](https://platform.openai.com/)
2. Go to API Keys
3. Create new key
4. Add to `.env`: `OPENAI_API_KEY=sk-...`

**Anthropic (Claude):**
1. Sign up at [console.anthropic.com](https://console.anthropic.com/)
2. Get API key
3. Add to `.env`: `ANTHROPIC_API_KEY=sk-ant-...`

**Google (Gemini):**
1. Get API key from [ai.google.dev](https://ai.google.dev/)
2. Add to `.env`: `GOOGLE_API_KEY=...`

**🎯 Tip:** You can develop without API keys! The UI works, you just can't send messages to AI. Perfect for frontend development.

---

## First Run

### Create Your Account

1. Open http://localhost:3080 (or :3090 in dev mode)
2. Click **"Sign Up"**
3. Enter email and password
4. Click **"Submit"**

**🎉 You're in!** You'll be redirected to the main chat interface.

### Explore the Interface

**What you're seeing:**

- **Left Sidebar:** Conversation history
- **Center:** Chat interface (like ChatGPT)
- **Top Bar:** AI provider selection (OpenAI, Anthropic, etc.)
- **Bottom:** Message input box

**Try it out:**
- Create a new conversation
- Select an AI provider (if you have API keys)
- Send a message!
- Explore the settings (⚙️ icon)

### 🌉 Bridge from React

**If you're used to Create React App or Vite:**

| CRA/Vite | LibreChat |
|----------|-----------|
| `npm start` | `npm run frontend:dev` (just frontend) |
| `npm run build` | `npm run frontend` (builds client) |
| Runs on 3000 | Frontend dev: 3090, Production: 3080 |
| `.env` for React | `.env` for full stack (backend + frontend) |
| Environment vars start with `REACT_APP_` or `VITE_` | Backend vars have no prefix (they're private!) |

**Key difference:** In LibreChat, the frontend is served BY the backend. The backend (Express) runs on 3080 and serves:
- API endpoints (like `/api/messages`)
- Static files (your built React app from `client/dist/`)

---

## Verification

### Check Services Are Running

**Docker:**
```bash
# All services should show "Up"
docker compose ps

# Should see:
# NAME                 SERVICE      STATUS
# librechat-api        api          Up
# librechat-client     client       Up
# librechat-mongodb    mongodb      Up
# librechat-redis      redis        Up
```

**Local Installation:**
```bash
# Check MongoDB
mongosh --eval "db.version()"
# Should show version: 6.x.x or 7.x.x

# Check Redis
redis-cli ping
# Should respond: PONG

# Check Node.js app
curl http://localhost:3080/api/health
# Should respond with: {"status":"ok"} or similar
```

### Check Database Connection

```bash
# Connect to MongoDB
mongosh

# Switch to LibreChat database
use LibreChat

# Check collections (should see users, conversations, etc.)
show collections

# Count users (should be 1 if you created an account)
db.users.countDocuments()

# Exit
exit
```

### Check Logs

**Docker:**
```bash
# View all logs
docker compose logs

# Follow logs in real-time
docker compose logs -f

# View only API logs
docker compose logs -f api
```

**Local:**
```bash
# Backend logs appear in the terminal where you ran npm run backend:dev
# Look for:
# "Server listening on port 3080"
# "MongoDB connected"
# "Redis connected"
```

---

## Common Issues

### Issue: "Cannot find module '@librechat/...'"

**Cause:** Dependencies not installed in monorepo packages

**Solution:**
```bash
# Reinstall all dependencies
npm run update

# Or manually
npm install
cd packages/data-provider && npm install && cd ../..
cd packages/data-schemas && npm install && cd ../..
```

---

### Issue: "ECONNREFUSED 127.0.0.1:27017"

**Cause:** MongoDB is not running

**Docker:**
```bash
docker compose up -d mongodb
docker compose logs mongodb
```

**Local:**
```bash
# Mac
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Verify
mongosh
```

---

### Issue: "ECONNREFUSED 127.0.0.1:6379"

**Cause:** Redis is not running

**Docker:**
```bash
docker compose up -d redis
```

**Local:**
```bash
# Mac
brew services start redis

# Linux
sudo systemctl start redis-server

# Verify
redis-cli ping
```

---

### Issue: "Port 3080 already in use"

**Cause:** Another app is using port 3080

**Solution:**
```bash
# Find what's using port 3080
# Mac/Linux:
lsof -i :3080

# Windows:
netstat -ano | findstr :3080

# Kill the process or change LibreChat port in .env:
PORT=3081
```

---

### Issue: "Vite build fails" or "Module not found"

**Cause:** Frontend dependencies or build cache issue

**Solution:**
```bash
# Clear node_modules and reinstall
cd client
rm -rf node_modules
npm install

# Clear Vite cache
rm -rf node_modules/.vite

# Rebuild
npm run build
```

---

### Issue: Docker containers keep restarting

**Cause:** Configuration error or port conflict

**Solution:**
```bash
# Check logs for errors
docker compose logs

# Common fixes:
# 1. Check .env file (SESSION_SECRET, etc.)
# 2. Stop and remove all containers
docker compose down
# 3. Rebuild from scratch
docker compose up --build -d
```

---

### Issue: "Module build failed" or TypeScript errors

**Cause:** TypeScript version mismatch or cache

**Solution:**
```bash
# Clear TypeScript cache
rm -rf client/node_modules/.cache
rm -rf api/node_modules/.cache

# Reinstall
npm install

# If using Docker, rebuild
docker compose build --no-cache
```

---

### Issue: Can't login after creating account

**Possible causes:**
1. SESSION_SECRET not set in .env
2. Redis not running
3. Cookie issues

**Solution:**
```bash
# Check .env has SESSION_SECRET
grep SESSION_SECRET .env

# Verify Redis is running
redis-cli ping  # Should respond: PONG

# Clear browser cookies for localhost:3080
# Then try again

# Check API logs for errors
docker compose logs api
# or
# Check terminal where you ran npm run backend:dev
```

---

## Next Steps

### 🎉 Congratulations!

You now have LibreChat running locally! Here's what to do next:

### Immediate Next Steps

1. **Explore the code**
   ```bash
   code .  # Open in VS Code
   ```
   - Look at `client/src/` (React frontend)
   - Look at `api/server/` (Express backend)
   - Open files referenced in the docs

2. **Read [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)**
   - Understand how everything fits together
   - See diagrams of the system

3. **Read [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)**
   - Learn where to find things
   - Understand file organization

### Suggested Learning Path

Choose based on your goals:

**Path 1: Frontend → Full-Stack**
1. ✅ Getting Started (you are here!)
2. [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React patterns
3. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Express + MongoDB
4. [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - How they connect
5. [EXERCISES.md](./EXERCISES.md) - Practice!

**Path 2: Backend Deep Dive**
1. ✅ Getting Started (you are here!)
2. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
3. [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
4. [SECURITY_GUIDE.md](./SECURITY_GUIDE.md)
5. [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)

**Path 3: Quick Contributor**
1. ✅ Getting Started (you are here!)
2. [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
3. [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)
4. [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
5. Make your first PR!

### Optional: Set Up Development Tools

**VS Code Extensions (Recommended):**
- ESLint - `dbaeumer.vscode-eslint`
- Prettier - `esbenp.prettier-vscode`
- TypeScript - Built-in
- MongoDB for VS Code - `mongodb.mongodb-vscode`
- GitLens - `eamodio.gitlens`
- Thunder Client - `rangav.vscode-thunder-client` (API testing)

**Configure VS Code:**
Create `.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

### Development Mode Workflow

```bash
# Always run these in separate terminals:

# Terminal 1: Backend (auto-reloads on changes)
npm run backend:dev

# Terminal 2: Frontend (hot module replacement)
npm run frontend:dev

# Terminal 3: Available for git, testing, etc.
```

### Useful Commands Reference

```bash
# Development
npm run backend:dev        # Start backend with auto-reload
npm run frontend:dev       # Start frontend with HMR
npm test                   # Run tests

# Production
npm run frontend          # Build frontend
npm run backend           # Start backend (production mode)

# Docker
docker compose up -d      # Start all services
docker compose down       # Stop all services
docker compose logs -f    # View logs
docker compose restart    # Restart all services

# Database
mongosh                   # Connect to MongoDB
redis-cli                 # Connect to Redis

# Maintenance
npm run update           # Update dependencies
npm run lint            # Check code style
npm run format          # Format code with Prettier
```

---

## 🎓 What You've Learned

After completing this guide, you now understand:

- ✅ **Development environment setup** - Node.js, MongoDB, Redis
- ✅ **Monorepo structure** - Multiple packages working together
- ✅ **Environment configuration** - .env files and secrets
- ✅ **Full-stack architecture** - Frontend + Backend + Database
- ✅ **Docker basics** - Containerized development (if you used Docker)
- ✅ **Development workflow** - How to run and test locally

### 🌉 Comparing to React-Only Development

| React App (CRA/Vite) | LibreChat Full-Stack |
|----------------------|----------------------|
| One `npm start` command | Separate frontend & backend processes |
| Data in localStorage/state | Data in MongoDB database |
| API calls to external services | API calls to YOUR backend |
| No secrets (all code public) | Secrets in .env (never committed) |
| One package.json | Multiple package.json (monorepo) |

---

## 🆘 Still Stuck?

### Check the Logs
- **Docker:** `docker compose logs -f`
- **Local:** Look at your terminal outputs

### Common Error Patterns

**"Cannot connect to MongoDB"**
→ MongoDB not running or wrong MONGO_URI

**"Cannot connect to Redis"**
→ Redis not running

**"Port in use"**
→ Kill other process or change port

**"Module not found"**
→ Run `npm install` or `npm run update`

**"Build failed"**
→ Clear cache: `rm -rf node_modules` then `npm install`

### Get Help

- **GitHub Issues:** [github.com/danny-avila/LibreChat/issues](https://github.com/danny-avila/LibreChat/issues)
- **Discord:** [discord.librechat.ai](https://discord.librechat.ai)
- **Docs:** [docs.librechat.ai](https://docs.librechat.ai)

---

## 📝 Summary Checklist

Before moving on, make sure you have:

- [ ] Node.js 20+ installed
- [ ] MongoDB running and connected
- [ ] Redis running and connected
- [ ] LibreChat cloned
- [ ] Dependencies installed (`npm install` or `npm run update`)
- [ ] `.env` file configured
- [ ] LibreChat running at http://localhost:3080 or :3090
- [ ] Account created and can log in
- [ ] Explored the UI
- [ ] Checked logs show no errors

**If all checked ✅, you're ready to proceed!**

---

**Next:** [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - Understand how LibreChat works

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
