# Getting Started with Novel Editor

**Last Updated:** November 19, 2025
**Tech Stack Research:** [View Current Tech Stack](./TECH_STACK_RESEARCH.md)
**Target Audience:** New developers, contributors, integrators

This guide will help you set up the Novel editor development environment, understand the project structure, and make your first runs.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start (TL;DR)](#quick-start-tldr)
3. [Detailed Setup Instructions](#detailed-setup-instructions)
4. [Understanding the Project Structure](#understanding-the-project-structure)
5. [Development Commands](#development-commands)
6. [Environment Variables](#environment-variables)
7. [Your First Run](#your-first-run)
8. [Common Issues & Troubleshooting](#common-issues--troubleshooting)
9. [Next Steps](#next-steps)

---

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Software

| Software | Minimum Version | Recommended | Check Version |
|----------|----------------|-------------|---------------|
| **Node.js** | v18.17.0+ | v22.x (LTS) | `node --version` |
| **pnpm** | v8.0.0+ | v10.22.0 | `pnpm --version` |
| **Git** | v2.0+ | Latest | `git --version` |

### Installing Prerequisites

**Node.js:**
- **Recommended:** Use [nvm](https://github.com/nvm-sh/nvm) (Node Version Manager)
  ```bash
  # Install nvm (Unix/Mac)
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

  # Install Node.js 22 (LTS)
  nvm install 22
  nvm use 22
  ```
- **Alternative:** Download from [nodejs.org](https://nodejs.org/)

**pnpm:**
```bash
# Install pnpm globally
npm install -g pnpm

# Or using Homebrew (Mac)
brew install pnpm

# Or using standalone script
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

**Git:**
- **Mac:** Pre-installed or via `brew install git`
- **Linux:** `sudo apt-get install git` or `sudo yum install git`
- **Windows:** Download from [git-scm.com](https://git-scm.com/)

---

## Quick Start (TL;DR)

```bash
# 1. Clone the repository
git clone https://github.com/steven-tey/novel.git
cd novel

# 2. Install dependencies
pnpm install

# 3. Set up environment variables (optional for basic usage)
cd apps/web
cp .env.example .env.local
# Edit .env.local and add your OPENAI_API_KEY (optional)

# 4. Start development server
cd ../..  # Back to root
pnpm dev

# 5. Open http://localhost:3000 in your browser
```

**That's it!** The editor will run without API keys, but AI features won't work.

---

## Detailed Setup Instructions

### Step 1: Clone the Repository

```bash
# Using HTTPS
git clone https://github.com/steven-tey/novel.git

# Or using SSH (if you have SSH keys set up)
git clone git@github.com:steven-tey/novel.git

# Navigate into the directory
cd novel
```

**Verify you're in the right place:**
```bash
ls -la
# You should see: apps/, packages/, package.json, turbo.json, etc.
```

---

### Step 2: Install Dependencies

Novel uses **pnpm** as its package manager and **Turborepo** for the monorepo.

```bash
# Install all dependencies for all packages
pnpm install
```

**What this does:**
- Installs dependencies for the root workspace
- Installs dependencies for `/packages/headless` (the core editor library)
- Installs dependencies for `/apps/web` (the demo Next.js app)
- Installs dependencies for `/packages/tsconfig` (shared TypeScript configs)
- Creates a `node_modules` folder and `pnpm-lock.yaml`

**Expected output:**
```
Packages: +XXX
++++++++++++++++++++++++++++++++++++++++++
Progress: resolved XXX, reused XXX, downloaded X, added XXX, done
```

⚠️ **Note:** The project uses **pnpm v9.5.0** (specified in package.json), but **pnpm v10.22.0** is the latest as of November 2025. The project works with both versions.

---

### Step 3: Set Up Environment Variables (Optional)

Environment variables control AI features and file uploads. The editor works without them, but features will be limited.

#### 3.1: Navigate to the web app

```bash
cd apps/web
```

#### 3.2: Copy the example environment file

```bash
cp .env.example .env.local
```

#### 3.3: Edit `.env.local` with your credentials

```bash
# Use your preferred editor
nano .env.local
# or
code .env.local  # VS Code
# or
vim .env.local
```

**Required for AI features:**
```env
# Get your key at: https://platform.openai.com/account/api-keys
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Optional configurations:**

```env
# Optional: Custom OpenAI Base URL (for proxies or compatible APIs)
OPENAI_BASE_URL=https://api.openai.com/v1

# Optional: Vercel Blob (for image uploads)
# Get credentials: https://vercel.com/docs/storage/vercel-blob/quickstart
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_xxxxxxxxxxxxxxxx

# Optional: Vercel KV (for rate limiting AI requests)
# Get credentials: https://vercel.com/docs/storage/vercel-kv/quickstart
KV_REST_API_URL=https://xxxxxxxxx.kv.vercel-storage.com
KV_REST_API_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**What each variable does:**

| Variable | Purpose | Required? | Default Behavior Without It |
|----------|---------|-----------|------------------------------|
| `OPENAI_API_KEY` | Enables AI completions | No | AI features show error messages |
| `OPENAI_BASE_URL` | Custom API endpoint | No | Uses official OpenAI API |
| `BLOB_READ_WRITE_TOKEN` | Image upload to Vercel Blob | No | Images stored as local data URLs |
| `KV_REST_API_URL` | Rate limiting | No | No rate limiting applied |
| `KV_REST_API_TOKEN` | Rate limiting | No | No rate limiting applied |

#### 3.4: Return to project root

```bash
cd ../..  # Back to /novel
```

---

### Step 4: Verify Your Setup

Before running, let's verify everything is set up correctly:

```bash
# Check Node.js version
node --version
# Should be v18.17.0 or higher

# Check pnpm version
pnpm --version
# Should be v8.0.0 or higher

# Check that packages are installed
ls node_modules
# Should show many packages

# Verify TypeScript is available
pnpm typecheck
# Should complete without errors (or show only known type issues)
```

---

## Understanding the Project Structure

Before running the app, understand what you're working with:

```
novel/
├── apps/
│   └── web/                    # Next.js demo application
│       ├── app/                # Next.js App Router pages
│       ├── components/         # React components
│       ├── public/             # Static assets
│       └── package.json
├── packages/
│   ├── headless/              # Core "novel" library (npm package)
│   │   ├── src/
│   │   │   ├── components/    # Editor components
│   │   │   ├── extensions/    # Tiptap extensions
│   │   │   ├── plugins/       # ProseMirror plugins
│   │   │   └── utils/         # Utilities
│   │   └── package.json
│   └── tsconfig/              # Shared TypeScript configs
├── package.json               # Root workspace config
├── turbo.json                 # Turborepo configuration
├── pnpm-workspace.yaml        # pnpm workspace config
└── pnpm-lock.yaml            # Lockfile (do not edit manually)
```

**Key directories:**

- **`/apps/web`**: The demo Next.js application showcasing the editor
- **`/packages/headless`**: The reusable editor library (what gets published to npm as "novel")
- **`/packages/tsconfig`**: Shared TypeScript configurations

💡 **Mental Model:** Think of it as a library (`packages/headless`) with a demo app (`apps/web`) to showcase it.

📖 **Learn more:** [Project Structure Guide](./PROJECT_STRUCTURE.md)

---

## Development Commands

Novel uses **Turborepo** to run commands across all packages efficiently.

### Starting Development Mode

```bash
# Start all packages in development mode
pnpm dev
```

**What this does:**
- Starts the Next.js dev server for `/apps/web` (usually on port 3000)
- Watches `/packages/headless` for changes and rebuilds automatically
- Enables hot module replacement (HMR) - changes appear instantly

**Expected output:**
```
• Packages in scope: novel-next-app, novel, tsconfig
• Running dev in 3 packages
• Remote caching disabled

novel:dev: cache bypass, force executing...
novel:dev:
novel:dev: > novel@1.0.0 dev /novel/packages/headless
novel:dev: > tsup --watch

novel-next-app:dev: cache bypass, force executing...
novel-next-app:dev:   ▲ Next.js 15.1.4
novel-next-app:dev:   - Local:        http://localhost:3000
novel-next-app:dev:   - Environments: .env.local
```

**Access the app:** Open [http://localhost:3000](http://localhost:3000) in your browser

---

### Other Useful Commands

#### Building for Production

```bash
# Build all packages
pnpm build
```

This creates optimized production builds:
- `/packages/headless/dist/` - Bundled library files (CJS + ESM)
- `/apps/web/.next/` - Next.js production build

---

#### Linting and Formatting

```bash
# Check for linting issues
pnpm lint

# Fix linting issues automatically
pnpm lint:fix

# Check code formatting
pnpm format

# Fix code formatting automatically
pnpm format:fix
```

✅ **Current (Nov 2025):** Novel uses **Biome** v1.9.4 for linting and formatting (replaces ESLint + Prettier).

⚠️ **Note:** Biome v2.3.6 is available (major version behind). All patterns work the same.

---

#### Type Checking

```bash
# Run TypeScript type checking across all packages
pnpm typecheck
```

This checks for type errors without building.

---

#### Cleaning Build Artifacts

```bash
# Remove all build outputs and caches
pnpm clean
```

**Useful when:**
- Build artifacts are corrupted
- Switching branches
- Fresh rebuild needed

---

### Working with Individual Packages

You can run commands in specific packages:

```bash
# Run commands in the web app only
cd apps/web
pnpm dev

# Run commands in the headless package only
cd packages/headless
pnpm dev
pnpm build
```

---

## Environment Variables

### Required Environment Variables

**None!** The editor works without any environment variables.

### Optional Environment Variables

**For AI Features:**
- `OPENAI_API_KEY` - Enables AI completions (continue writing, improve, fix, etc.)

**For Image Uploads:**
- `BLOB_READ_WRITE_TOKEN` - Enables image upload to Vercel Blob storage

**For Rate Limiting:**
- `KV_REST_API_URL` - Vercel KV (Redis) URL
- `KV_REST_API_TOKEN` - Vercel KV authentication token

### Getting API Keys

**OpenAI API Key:**
1. Go to [platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys)
2. Sign in or create an account
3. Click "Create new secret key"
4. Copy and save the key (it won't be shown again)
5. Add to `.env.local`: `OPENAI_API_KEY=sk-...`

**Vercel Blob Storage:**
1. Create a Vercel account at [vercel.com](https://vercel.com)
2. Go to your project (or create one)
3. Navigate to Storage → Create Database → Blob
4. Copy the `BLOB_READ_WRITE_TOKEN`
5. Add to `.env.local`

**Vercel KV (Redis):**
1. In your Vercel project
2. Navigate to Storage → Create Database → KV
3. Copy `KV_REST_API_URL` and `KV_REST_API_TOKEN`
4. Add both to `.env.local`

### Using Vercel Environment Variables Locally

If you've deployed to Vercel, pull environment variables:

```bash
cd apps/web
npx vercel env pull .env.local
```

---

## Your First Run

### Step 1: Start the Development Server

```bash
# From the project root
pnpm dev
```

Wait for the compilation to complete.

---

### Step 2: Open in Browser

Visit [http://localhost:3000](http://localhost:3000)

You should see:
- ✅ The Novel editor interface
- ✅ A demo document with example content
- ✅ Ability to type and edit
- ✅ Slash commands (type `/`)
- ✅ Bubble menu when you select text
- ✅ Theme switcher (light/dark mode)

---

### Step 3: Test Basic Features

**Try these actions:**

1. **Type something**
   - Just start typing in the editor
   - Text should appear instantly

2. **Use slash commands**
   - Type `/` to open the command menu
   - Try `/heading1`, `/quote`, `/code`
   - Select a command to insert a block

3. **Format text**
   - Select some text
   - A bubble menu appears with formatting options
   - Try **Bold**, *Italic*, `Code`, etc.

4. **Test AI features** (if `OPENAI_API_KEY` is set)
   - Type some text
   - Select it
   - Click the ✨ icon in the bubble menu
   - Choose "Continue writing" or "Improve"
   - See AI completion stream in

5. **Upload an image** (with or without Blob storage)
   - Drag and drop an image
   - Or type `/image` and upload
   - Image should appear in the editor

---

### Step 4: Explore the Code

While the app is running, try making changes:

1. **Open `/apps/web/app/page.tsx`** in your editor
2. **Change the page title** or add some text
3. **Save the file**
4. **Watch the browser auto-refresh** (HMR)

🎯 **You're now ready to start developing!**

---

## Common Issues & Troubleshooting

### Issue: `pnpm: command not found`

**Solution:**
```bash
# Install pnpm globally
npm install -g pnpm

# Verify installation
pnpm --version
```

---

### Issue: Port 3000 is already in use

**Error:**
```
Error: listen EADDRINUSE: address already in use :::3000
```

**Solution 1:** Kill the process using port 3000
```bash
# Mac/Linux
lsof -ti:3000 | xargs kill -9

# Or find and kill manually
lsof -i:3000
kill -9 <PID>
```

**Solution 2:** Use a different port
```bash
# In apps/web
PORT=3001 pnpm dev
```

---

### Issue: AI features don't work

**Symptoms:**
- Clicking AI button shows an error
- Console shows API errors

**Solutions:**

1. **Check API key is set:**
   ```bash
   # In apps/web/.env.local
   OPENAI_API_KEY=sk-xxxxxxxx
   ```

2. **Verify API key is valid:**
   - Go to [platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys)
   - Check if the key is active
   - Check if you have API credits

3. **Check API key format:**
   - Should start with `sk-`
   - No quotes around the value in `.env.local`

4. **Restart the dev server:**
   ```bash
   # Stop with Ctrl+C, then
   pnpm dev
   ```

---

### Issue: Build errors after pulling latest changes

**Solution:**
```bash
# Clean everything and reinstall
pnpm clean
rm -rf node_modules pnpm-lock.yaml
pnpm install
pnpm build
```

---

### Issue: TypeScript errors

**Error:**
```
Type error: Cannot find module 'X' or its corresponding type declarations
```

**Solution:**
```bash
# Ensure all dependencies are installed
pnpm install

# Run type check to see all errors
pnpm typecheck

# If using VS Code, reload the window
# Cmd+Shift+P → "Developer: Reload Window"
```

---

### Issue: Changes not reflecting in browser

**Solutions:**

1. **Hard refresh the browser:**
   - Mac: `Cmd + Shift + R`
   - Windows/Linux: `Ctrl + Shift + R`

2. **Clear Next.js cache:**
   ```bash
   cd apps/web
   rm -rf .next
   cd ../..
   pnpm dev
   ```

3. **Check for errors in terminal:**
   - Look for compilation errors in the console

---

### Issue: `MODULE_NOT_FOUND` errors

**Solution:**
```bash
# Make sure you're in the root directory
pwd
# Should end with /novel

# Reinstall dependencies
pnpm install

# If still failing, try
rm -rf node_modules
pnpm install
```

---

### Issue: Slow installation or build

**Causes:**
- Large number of dependencies
- Slow internet connection

**Solutions:**

1. **Use pnpm's optional dependencies flag:**
   ```bash
   pnpm install --no-optional
   ```

2. **Clear pnpm cache:**
   ```bash
   pnpm store prune
   pnpm install
   ```

3. **Increase Node.js memory (for builds):**
   ```bash
   NODE_OPTIONS="--max-old-space-size=4096" pnpm build
   ```

---

## Next Steps

Congratulations! You now have Novel running locally. 🎉

### Recommended Next Steps:

1. **Understand the Architecture**
   - Read [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
   - Learn about the monorepo structure
   - Understand component hierarchy

2. **Explore the Codebase**
   - Check [Project Structure](./PROJECT_STRUCTURE.md)
   - Navigate through `/packages/headless/src`
   - Look at example components in `/apps/web/components`

3. **Learn the Technologies**
   - Review [Tech Stack Guide](./TECH_STACK_GUIDE.md)
   - Understand Tiptap, Next.js, Jotai, etc.
   - Note version differences (see [Tech Stack Research](./TECH_STACK_RESEARCH.md))

4. **Follow a Code Tour**
   - Try [Code Tours](./CODE_TOURS.md)
   - Follow a feature from start to finish
   - Understand data flow

5. **Make Your First Change**
   - Read [How-To Guide](./HOW_TO_GUIDE.md)
   - Try adding a slash command
   - Customize editor styling

6. **Contribute**
   - Check [First Contributions](./FIRST_CONTRIBUTIONS.md)
   - Find a good first issue
   - Submit a PR

---

## Quick Reference Card

**Essential Commands:**
```bash
pnpm install       # Install dependencies
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm lint:fix     # Fix linting issues
pnpm format:fix   # Fix formatting
pnpm typecheck    # Check types
pnpm clean        # Clean build artifacts
```

**Key Files:**
- `package.json` - Root workspace config
- `turbo.json` - Turborepo task configuration
- `apps/web/.env.local` - Environment variables (create this)
- `apps/web/app/page.tsx` - Demo homepage
- `packages/headless/src/index.ts` - Main library export

**Useful Links:**
- Local dev server: http://localhost:3000
- Novel homepage: https://novel.sh
- GitHub repo: https://github.com/steven-tey/novel
- Tiptap docs: https://tiptap.dev/docs
- Next.js docs: https://nextjs.org/docs

---

**Need Help?**
- Check the [FAQ](./FAQ.md)
- Read the [Debugging Guide](./DEBUGGING_GUIDE.md)
- Open an issue on [GitHub](https://github.com/steven-tey/novel/issues)

---

**Last Updated:** November 19, 2025
**Covers:** Novel v1.0.0, Next.js 15, React 18, pnpm 9+
**Status:** ✅ All instructions verified against current codebase
