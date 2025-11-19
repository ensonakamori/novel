# Development Workflow: From Code to Production

> **Target Audience**: Developers contributing to Novel or using it in production
>
> **Prerequisites**: Basic Git knowledge, understanding of Node.js projects
>
> **What You'll Learn**: Professional development workflows, patterns, and best practices

---

## Table of Contents

### Version Control
1. [Git Workflow and Branching Strategy](#git-workflow-and-branching-strategy)
2. [Commit Message Conventions](#commit-message-conventions)
3. [Pull Request Best Practices](#pull-request-best-practices)

### Development Process
4. [Local Development Setup](#local-development-setup)
5. [Building and Testing](#building-and-testing)
6. [Code Quality and Linting](#code-quality-and-linting)

### Publishing and Deployment
7. [Publishing to npm](#publishing-to-npm)
8. [Versioning Strategy](#versioning-strategy)
9. [CI/CD Pipeline](#cicd-pipeline)

### Best Practices
10. [Code Review Guidelines](#code-review-guidelines)
11. [Debugging Workflow](#debugging-workflow)
12. [Performance Optimization](#performance-optimization)

---

## Git Workflow and Branching Strategy

### Introduction: Why Branching Strategy Matters

**Mid-Level Developer Mindset**: "I'll just commit to main and push"
**Professional Mindset**: "I'll use a branching strategy that supports collaboration, testing, and safe deployments"

**The Problem with No Strategy**:
- Multiple developers overwriting each other's work
- Breaking changes deployed to production
- No way to test features before release
- Difficulty tracking what changed and when

---

### Novel's Branching Model

```
main (production-ready)
  ↓
  ├─ feat/ai-improvements (feature branch)
  ├─ fix/image-upload-bug (bug fix branch)
  ├─ docs/api-reference (documentation branch)
  └─ chore/update-deps (maintenance branch)
```

**Branch Types**:

| Prefix | Purpose | Example | Merge To |
|--------|---------|---------|----------|
| `feat/` | New features | `feat/collaborative-editing` | `main` |
| `fix/` | Bug fixes | `fix/slash-command-crash` | `main` |
| `docs/` | Documentation | `docs/integration-guide` | `main` |
| `chore/` | Maintenance | `chore/upgrade-tiptap` | `main` |
| `refactor/` | Code refactoring | `refactor/state-management` | `main` |
| `test/` | Adding tests | `test/extension-coverage` | `main` |

---

### Creating a Feature Branch

**Step 1: Update main branch**

```bash
# Always start from latest main
git checkout main
git pull origin main
```

**Step 2: Create feature branch**

```bash
# Create branch with descriptive name
git checkout -b feat/add-comment-system

# Branch naming convention: type/short-description-in-kebab-case
```

**Good Branch Names**:
- ✅ `feat/collaborative-editing`
- ✅ `fix/image-upload-memory-leak`
- ✅ `docs/extension-development-guide`

**Bad Branch Names**:
- ❌ `my-branch`
- ❌ `feature123`
- ❌ `john-changes`

---

### Working on Your Branch

```bash
# Make changes to files
# ... code, code, code ...

# Check what changed
git status
git diff

# Stage changes
git add path/to/file.ts

# Or stage all changes
git add .

# Commit with meaningful message (see next section)
git commit -m "feat(editor): add comment system with threading"

# Push to remote
git push origin feat/add-comment-system
```

**Best Practice: Commit Often, Push Regularly**

```bash
# ✅ GOOD: Small, focused commits
git commit -m "feat(comments): add Comment extension"
git commit -m "feat(comments): add CommentSidebar UI"
git commit -m "feat(comments): add resolve functionality"
git commit -m "test(comments): add unit tests"

# ❌ BAD: One massive commit
git commit -m "add comment system"  # (hundreds of files changed)
```

**Why Small Commits?**:
1. **Easier to review**: Reviewer can understand each change
2. **Easier to revert**: If one commit breaks something, revert just that commit
3. **Better history**: `git log` tells a story of how feature was built
4. **Easier to cherry-pick**: Can apply specific commits to other branches

---

### Keeping Branch Up to Date

**Problem**: Your feature branch diverges from main as others merge their changes.

```bash
# Your branch
feat/comment-system
  ↓
  A -- B -- C  (your commits)

# Meanwhile, main moved forward
main
  ↓
  D -- E -- F  (other people's commits)
```

**Solution 1: Rebase (Recommended)**

```bash
# Update main
git checkout main
git pull origin main

# Rebase your feature branch
git checkout feat/comment-system
git rebase main

# Resolve conflicts if any, then:
git push --force-with-lease origin feat/comment-system
```

**After rebase**:
```bash
feat/comment-system
  ↓
  D -- E -- F -- A' -- B' -- C'  (your commits on top of latest main)
```

**Solution 2: Merge (Alternative)**

```bash
# From your feature branch
git checkout feat/comment-system
git merge main

# Resolve conflicts, commit, then:
git push origin feat/comment-system
```

**Rebase vs Merge**:

| Aspect | Rebase | Merge |
|--------|--------|-------|
| History | Linear, clean | Shows full history with merge commits |
| Conflicts | Resolve once per commit | Resolve all at once |
| Use when | Feature branch | Long-lived branches |
| Golden rule | Never rebase public commits | Safe for any branch |

---

## Commit Message Conventions

### Introduction: Commit Messages as Documentation

**The Problem**:
```bash
# ❌ BAD commit messages (you'll see these in the wild)
git commit -m "fix"
git commit -m "update"
git commit -m "changes"
git commit -m "asdfasdf"
git commit -m "WIP"
```

**6 months later**:
```bash
git log
# What did "fix" fix? What was updated? No idea! 🤷‍♂️
```

**The Solution**: Conventional Commits

---

### Conventional Commit Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Example**:
```
feat(editor): add collaborative editing with Yjs

- Integrate Yjs for CRDT-based collaboration
- Add WebSocket provider for real-time sync
- Implement cursor position sharing
- Add presence awareness (who's editing)

Closes #123
```

---

### Commit Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(ai): add text summarization` |
| `fix` | Bug fix | `fix(upload): handle large image files` |
| `docs` | Documentation | `docs(readme): add installation guide` |
| `style` | Code style (formatting) | `style(editor): fix indentation` |
| `refactor` | Code refactoring | `refactor(state): migrate to Jotai` |
| `perf` | Performance improvement | `perf(render): memoize heavy computations` |
| `test` | Adding/updating tests | `test(extensions): add Bold extension tests` |
| `chore` | Maintenance | `chore(deps): upgrade Tiptap to v2.2.0` |
| `ci` | CI/CD changes | `ci(github): add automated testing` |

---

### Commit Scopes

Scopes indicate which part of the codebase changed:

**Common Scopes in Novel**:
- `editor` - Core editor component
- `extensions` - Tiptap extensions
- `ui` - UI components
- `api` - API routes
- `state` - State management
- `plugins` - ProseMirror plugins
- `utils` - Utility functions
- `types` - TypeScript types
- `docs` - Documentation

**Examples**:
```bash
feat(extensions): add Mathematics extension
fix(ui): EditorBubble positioning on mobile
docs(api): document image upload endpoint
refactor(state): simplify Jotai atom structure
perf(editor): reduce re-renders with useMemo
```

---

### Writing Great Commit Messages

**Rule 1: Use imperative mood** (like giving a command)

```bash
# ✅ GOOD: Imperative mood
feat(editor): add collaborative editing
fix(upload): handle file size validation
refactor(state): extract hook logic

# ❌ BAD: Past tense or present tense
feat(editor): added collaborative editing
fix(upload): handles file size validation
refactor(state): extracting hook logic
```

**Why imperative?** Matches Git's own messages:
```bash
git merge → "Merge branch 'feat/comments'"
git revert → "Revert 'feat(editor): add collaborative editing'"
```

---

**Rule 2: Keep subject line under 50 characters**

```bash
# ✅ GOOD: Concise
feat(ai): add text summarization

# ❌ BAD: Too long
feat(ai): add text summarization feature that uses OpenAI API to summarize selected text in the editor
```

**Why?** GitHub and most tools show only first 50-70 characters.

---

**Rule 3: Use body to explain "why", not "what"**

```bash
# ✅ GOOD: Explains why
feat(editor): add debounced auto-save

Auto-save was triggering on every keystroke, causing:
- High server load (100+ requests/minute per user)
- Poor UX (constant "saving..." indicator)

Debouncing saves to 500ms after last edit, reducing
server requests by 95% while maintaining data safety.

Closes #456

# ❌ BAD: Repeats what code says
feat(editor): add debounced auto-save

Added useDebouncedCallback hook with 500ms delay.
Called debouncedSave in onUpdate handler.
```

---

**Rule 4: Reference issues in footer**

```bash
feat(comments): implement comment threading

Closes #123
Related to #456
Fixes #789
```

**Keywords** that auto-close issues:
- `Closes #123`
- `Fixes #123`
- `Resolves #123`

---

### Real-World Commit Examples from Novel

```bash
# Feature with breaking change
feat(editor)!: change default extensions structure

BREAKING CHANGE: Extensions are now passed as flat array
instead of nested config object. Update your editor setup:

Before:
  extensions={{ basic: [StarterKit], custom: [MyExt] }}

After:
  extensions={[StarterKit, MyExt]}

Closes #789

# Bug fix with root cause explanation
fix(upload): prevent memory leak in image upload

Image upload was keeping references to File objects
after upload completed, causing memory to grow unbounded
with multiple uploads.

Solution: Clear file references after successful upload
and in error handler.

Fixes #234

# Performance improvement with metrics
perf(editor): optimize extension initialization

Reduced editor mount time from 450ms to 120ms by:
- Lazy loading heavy extensions (200ms saved)
- Memoizing extension config (80ms saved)
- Removing duplicate schema builds (50ms saved)

Benchmark: 1000 editor mounts went from 7.5min to 2min.

# Refactoring with rationale
refactor(state): migrate from Context to Jotai

Context caused unnecessary re-renders across entire
editor tree when AI menu state changed. Jotai atoms
provide fine-grained reactivity.

Result: 60% reduction in re-renders during AI operations.

# Documentation
docs(integration): add Next.js 15 App Router guide

Covers:
- Server vs client components
- Image optimization
- Edge runtime setup
- Common pitfalls

Based on community questions in #567, #589, #601
```

---

### Commit Message Template

Create `.gitmessage` in your home directory:

```bash
# ~/.gitmessage

# <type>(<scope>): <subject>
# |<----  Using a Maximum Of 50 Characters  ---->|


# Explain why this change is being made
# |<----   Try To Limit Each Line to a Maximum Of 72 Characters   ---->|


# Provide links or keys to any relevant tickets, articles or other resources
# Example: Closes #23


# --- COMMIT END ---
# Type can be
#    feat     (new feature)
#    fix      (bug fix)
#    refactor (refactoring production code)
#    style    (formatting, missing semi colons, etc; no code change)
#    docs     (changes to documentation)
#    test     (adding or refactoring tests; no production code change)
#    chore    (updating grunt tasks etc; no production code change)
#    perf     (performance improvements)
# --------------------
# Remember to
#    Use imperative mood ("add" not "added" or "adds")
#    Capitalize subject line
#    Do not end subject line with period
#    Separate subject from body with blank line
#    Use body to explain what and why vs. how
#    Can use multiple lines with "-" for bullet points in body
# --------------------
```

**Enable template**:
```bash
git config --global commit.template ~/.gitmessage
```

Now `git commit` (without `-m`) opens editor with this template!

---

## Pull Request Best Practices

### Introduction: PRs as Code Review and Knowledge Transfer

**Pull Request is NOT just merging code**. It's:
1. **Code review** - Catch bugs, improve quality
2. **Knowledge sharing** - Teach patterns, spread expertise
3. **Documentation** - Explain complex changes
4. **Discussion** - Collaborate on design decisions

---

### Creating a Pull Request

**Step 1: Push your feature branch**

```bash
git push origin feat/comment-system
```

**Step 2: Create PR on GitHub**

```bash
# If you have GitHub CLI
gh pr create --title "feat(comments): add comment system with threading" \
             --body "$(cat <<'EOF'
## Summary
Implements inline commenting system with:
- Comment threads on selected text
- Reply functionality
- Resolve/unresolve
- Real-time updates via Jotai

## Motivation
Users requested ability to collaborate on documents (#123).
Comments are essential for editorial workflows.

## Changes
- Add Comment extension (Mark-based)
- Add CommentSidebar UI component
- Add comment thread management
- Add resolve/unresolve functionality
- Update state management with comment atoms

## Testing
- ✅ Unit tests for Comment extension
- ✅ Integration tests for sidebar
- ✅ Manual testing on Chrome, Firefox, Safari
- ✅ Mobile responsive testing

## Screenshots
[Before/After screenshots]

## Breaking Changes
None

## Checklist
- [x] Tests pass
- [x] Linter passes
- [x] Documentation updated
- [x] Changelog updated
- [x] Backward compatible

Closes #123
EOF
)"
```

---

### PR Title Conventions

**Same as commit messages**:

```
<type>(<scope>): <description>
```

**Examples**:
- `feat(comments): add inline commenting system`
- `fix(upload): handle large files gracefully`
- `docs(integration): add Vue.js integration guide`
- `refactor(editor): simplify extension registration`

---

### PR Description Template

```markdown
## Summary
Brief overview (1-2 sentences) of what this PR does.

## Motivation
Why is this change needed? What problem does it solve?
Link to related issues.

## Changes
Detailed list of changes:
- Added X
- Modified Y
- Removed Z

## Type of Change
- [ ] New feature (non-breaking)
- [ ] Bug fix (non-breaking)
- [ ] Breaking change
- [ ] Documentation update
- [ ] Refactoring
- [ ] Performance improvement

## Testing
How was this tested?
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing
- [ ] E2E tests

Testing details:
- Browsers tested: Chrome 120, Firefox 121, Safari 17
- Devices tested: Desktop, Mobile (iOS, Android)

## Screenshots/Videos
If applicable, add screenshots or videos demonstrating the change.

## Performance Impact
If applicable, describe performance implications:
- Bundle size: +2KB (before: 150KB, after: 152KB)
- Load time: -50ms (lazy loading heavy deps)
- Memory: No significant change

## Breaking Changes
List any breaking changes and migration guide.

## Checklist
- [ ] Code follows project style guidelines
- [ ] Tests pass locally
- [ ] Linter passes
- [ ] Documentation updated
- [ ] Changelog updated
- [ ] Self-reviewed code
- [ ] Added comments for complex logic
- [ ] No new warnings
- [ ] Backward compatible (or documented breaking changes)

## Related PRs/Issues
- Closes #123
- Related to #456
- Depends on #789
```

---

### Code Review Guidelines

**As Author (Person creating PR)**:

**1. Self-Review First**
```bash
# Before requesting review, review your own code
# Look at the diff on GitHub, not just your editor
```

**Checklist**:
- ✅ Remove console.logs and debugger statements
- ✅ Remove commented-out code
- ✅ Check for TODOs (resolve or create issues)
- ✅ Ensure all tests pass
- ✅ Run linter
- ✅ Check for sensitive data (API keys, tokens)
- ✅ Verify no accidental file changes

---

**2. Make PR Reviewable**

```bash
# ✅ GOOD: Small, focused PR
feat(comments): add Comment extension
  - 5 files changed
  - +200 lines, -50 lines
  - Clear scope: just the extension

# ❌ BAD: Giant PR
feat(comments): complete commenting system
  - 50 files changed
  - +5000 lines, -200 lines
  - Mix of features, refactoring, fixes
```

**Rule of Thumb**: If PR has >500 lines changed, split it.

**How to Split**:
```bash
# Instead of one big PR:
feat(comments): add complete commenting system

# Create multiple smaller PRs:
1. feat(extensions): add Comment Mark extension
2. feat(ui): add CommentSidebar component
3. feat(state): add comment state management
4. feat(integration): integrate comments in editor
```

---

**3. Add Context in Code Comments**

```typescript
// ✅ GOOD: Explain WHY, not WHAT
// Use Set instead of Array for O(1) lookup
// Comment IDs can grow to thousands in large docs
const commentIds = new Set<string>();

// Using Jotai instead of Context to avoid
// re-rendering entire editor tree when comments change
const [comments, setComments] = useAtom(commentsAtom);

// ❌ BAD: Stating the obvious
// Create a new Set
const commentIds = new Set<string>();

// Use useAtom hook
const [comments, setComments] = useAtom(commentsAtom);
```

---

**4. Respond to Feedback Gracefully**

```markdown
# ✅ GOOD: Professional response
> Consider using Map instead of object for O(1) access

Great catch! Changed to Map. Also added benchmark
showing 40% improvement with 1000+ comments.

Committed in abc123f

# ❌ BAD: Defensive response
> Consider using Map instead of object

Object works fine. Map is overkill for this.

# ❌ BAD: Dismissive response
> Consider using Map instead of object

K, changed.
```

---

**As Reviewer (Person reviewing PR)**:

**1. Review for Different Levels**

**Level 1: Critical Issues** (Must fix before merge)
- 🔴 Security vulnerabilities
- 🔴 Data loss bugs
- 🔴 Breaking changes without migration guide
- 🔴 Performance regressions

**Level 2: Important** (Should fix)
- 🟡 Logic errors
- 🟡 Poor error handling
- 🟡 Missing tests
- 🟡 Code style violations

**Level 3: Suggestions** (Nice to have)
- 🟢 Code optimization
- 🟢 Better naming
- 🟢 Additional comments
- 🟢 Refactoring opportunities

---

**2. Give Actionable Feedback**

```markdown
# ✅ GOOD: Specific, actionable
This function could cause memory leak if component unmounts
during upload. Consider adding cleanup:

```typescript
useEffect(() => {
  let cancelled = false;

  uploadImage().then(url => {
    if (!cancelled) setImageUrl(url);
  });

  return () => { cancelled = true; };
}, []);
```

# ❌ BAD: Vague, unhelpful
This doesn't look right.

# ❌ BAD: Nitpicky
You used single quotes but we prefer double quotes.
(This should be caught by linter, not code review!)
```

---

**3. Balance Praise and Critique**

```markdown
# ✅ GOOD: Balanced
Great implementation of the Comment extension! Love how you
handled edge cases like overlapping comments.

One suggestion: Could we extract the mark merging logic into
a utility function? It's complex enough to deserve its own
unit tests.

Also, this pattern of using WeakMap for node references is
clever. Consider documenting it for other contributors.

# ❌ BAD: Only negative
Why didn't you add tests?
This naming is confusing.
The logic is too complex.

# ❌ BAD: Only positive (but ignoring real issues)
Looks great! LGTM! 👍
(But there's a memory leak, no tests, and security issue)
```

---

**4. Approve or Request Changes**

**When to Approve** ✅:
- Code works correctly
- Tests are comprehensive
- No security/performance issues
- Minor suggestions only (not blocking)

**When to Request Changes** 🔄:
- Critical bugs
- Missing tests
- Security concerns
- Significant architecture issues

**When to Comment (No Action)** 💬:
- Questions for clarification
- Suggestions (non-blocking)
- Praise/learning

---

### Merging Strategies

**1. Squash and Merge** (Recommended for features)

```bash
# Before:
feat/comments
  ├─ WIP: add comment extension
  ├─ fix: typo
  ├─ add tests
  ├─ address PR feedback
  ├─ fix linter
  └─ update docs

# After squash and merge to main:
main
  └─ feat(comments): add inline commenting system (#123)
```

**Pros**:
- Clean, linear history
- One commit per feature
- Easy to revert features

**Cons**:
- Loses detailed commit history
- Can't cherry-pick individual commits

---

**2. Rebase and Merge** (Recommended for clean branches)

```bash
# Before:
feat/comments
  ├─ feat(extension): add Comment Mark
  ├─ feat(ui): add CommentSidebar
  └─ test(comments): add comprehensive tests

# After rebase and merge to main:
main
  ├─ feat(extension): add Comment Mark
  ├─ feat(ui): add CommentSidebar
  └─ test(comments): add comprehensive tests
```

**Pros**:
- Preserves detailed history
- Linear history (no merge commits)
- Can cherry-pick individual commits

**Cons**:
- Requires clean commit history

---

**3. Merge Commit** (For long-lived branches)

```bash
# Creates merge commit
main
  └─ Merge pull request #123 from feat/comments
```

**Pros**:
- Preserves full history
- Shows when feature was merged

**Cons**:
- Creates merge commits (cluttered history)
- Harder to track individual changes

---

**Novel's Strategy**: **Squash and Merge** for most PRs

---

## Local Development Setup

### Introduction: Reproducible Development Environment

**The Problem**: "It works on my machine" 🤷‍♂️

**The Solution**: Standardized tooling and setup process

---

### Prerequisites

```bash
# Check versions
node --version    # Should be v20.x or higher
pnpm --version    # Should be v8.x or higher
git --version     # Should be v2.x or higher
```

**Install pnpm** (if not installed):
```bash
npm install -g pnpm

# Or via Homebrew (Mac)
brew install pnpm

# Or via Corepack (recommended)
corepack enable
corepack prepare pnpm@latest --activate
```

---

### Cloning the Repository

```bash
# Clone repository
git clone https://github.com/steven-tey/novel.git
cd novel

# Or clone your fork
git clone https://github.com/YOUR_USERNAME/novel.git
cd novel

# Add upstream remote (for forks)
git remote add upstream https://github.com/steven-tey/novel.git
```

---

### Installing Dependencies

```bash
# Install all dependencies (root + workspaces)
pnpm install

# This installs dependencies for:
# - Root project
# - packages/headless
# - packages/tailwind
# - apps/web

# Check installation
pnpm list --depth=0
```

**Understanding pnpm Workspaces**:

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

**What this means**:
- All `packages/*` and `apps/*` are linked together
- Can import from other workspace packages
- Shared `node_modules` (deduplicated)
- Single `pnpm-lock.yaml` for entire monorepo

---

### Environment Variables

**File**: `apps/web/.env.local` (create this file)

```bash
# .env.local (NEVER commit this file!)

# Required for AI features
OPENAI_API_KEY=sk-...

# Required for image upload
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_...

# Optional: Rate limiting
KV_REST_API_URL=https://...
KV_REST_API_TOKEN=...

# Optional: Analytics
NEXT_PUBLIC_GA_ID=G-...
```

**Create template** for team:

```bash
# .env.example (commit this)
OPENAI_API_KEY=your_openai_key_here
BLOB_READ_WRITE_TOKEN=your_vercel_blob_token_here

# Copy this to .env.local and fill in real values
```

---

### Running Development Server

```bash
# From root directory
pnpm dev

# Or run specific workspace
pnpm --filter web dev
pnpm --filter headless dev

# Or use shorthand
pnpm dev:web
pnpm dev:headless
```

**What happens when you run `pnpm dev`**:

```
1. Turborepo orchestrates builds
   ↓
2. Builds dependencies first (headless, tailwind)
   ↓
3. Starts Next.js dev server
   ↓
4. Starts watching for file changes
   ↓
5. Hot module replacement (HMR) active
```

**Open in browser**: http://localhost:3000

---

### Development Tools Setup

**VS Code Extensions** (`.vscode/extensions.json`):

```json
{
  "recommendations": [
    "biomejs.biome",
    "bradlc.vscode-tailwindcss",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

**VS Code Settings** (`.vscode/settings.json`):

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

---

## Building and Testing

### Building the Project

**Build all workspaces**:

```bash
pnpm build

# Turborepo builds in optimal order:
# 1. packages/headless (no dependencies)
# 2. packages/tailwind (depends on headless)
# 3. apps/web (depends on both packages)
```

**Build specific workspace**:

```bash
pnpm --filter headless build
pnpm --filter web build
```

**What gets built**:

```
packages/headless/
  └── dist/
      ├── index.js          # CommonJS
      ├── index.mjs         # ESM
      ├── index.d.ts        # Types
      └── index.d.mts       # ESM types

apps/web/
  └── .next/
      ├── static/           # Static assets
      ├── server/           # Server code
      └── cache/            # Build cache
```

---

### Build Configuration

**File**: `packages/headless/tsup.config.ts`

```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  // Entry points
  entry: ["src/index.tsx"],

  // Output formats
  format: ["cjs", "esm"],

  // Generate declaration files
  dts: true,

  // Split code by entry point
  splitting: true,

  // Generate source maps
  sourcemap: true,

  // Clean dist folder before build
  clean: true,

  // External dependencies (not bundled)
  external: [
    "react",
    "react-dom",
    "@tiptap/core",
    "@tiptap/react",
  ],

  // Preserve JSX for React
  jsx: "preserve",
});
```

---

### Running Tests

**Test Structure**:

```
packages/headless/
  └── src/
      ├── components/
      │   ├── editor.tsx
      │   └── editor.test.tsx
      ├── extensions/
      │   ├── ai-highlight.ts
      │   └── ai-highlight.test.ts
      └── utils/
          ├── index.ts
          └── index.test.ts
```

**Run all tests**:

```bash
pnpm test

# Run with coverage
pnpm test:coverage

# Run in watch mode
pnpm test:watch

# Run specific test file
pnpm test editor.test.tsx

# Run tests matching pattern
pnpm test --grep "AIHighlight"
```

---

### Test Configuration

**File**: `vitest.config.ts`

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],

  test: {
    // Use jsdom environment for React components
    environment: "jsdom",

    // Setup file
    setupFiles: ["./test/setup.ts"],

    // Coverage configuration
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "test/",
        "**/*.d.ts",
        "**/*.config.*",
        "**/dist/**",
      ],
    },

    // Globals (no need to import describe, it, expect)
    globals: true,
  },
});
```

---

### Writing Tests

**Unit Test Example**:

```typescript
// src/utils/index.test.ts
import { describe, it, expect } from "vitest";
import { isValidUrl, getUrlFromString } from "./index";

describe("isValidUrl", () => {
  it("returns true for valid URLs", () => {
    expect(isValidUrl("https://example.com")).toBe(true);
    expect(isValidUrl("http://localhost:3000")).toBe(true);
  });

  it("returns false for invalid URLs", () => {
    expect(isValidUrl("not a url")).toBe(false);
    expect(isValidUrl("")).toBe(false);
  });
});

describe("getUrlFromString", () => {
  it("adds https:// to bare domains", () => {
    expect(getUrlFromString("example.com")).toBe("https://example.com");
  });

  it("returns null for invalid strings", () => {
    expect(getUrlFromString("invalid")).toBe(null);
  });
});
```

**Component Test Example**:

```typescript
// src/components/editor.test.tsx
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/react";
import { EditorRoot, EditorContent } from "./editor";

describe("Editor", () => {
  it("renders without crashing", () => {
    render(
      <EditorRoot>
        <EditorContent />
      </EditorRoot>
    );

    // Editor should be in document
    expect(screen.getByRole("textbox")).toBeInTheDocument();
  });

  it("accepts initial content", () => {
    const content = { type: "doc", content: [{ type: "paragraph" }] };

    render(
      <EditorRoot>
        <EditorContent initialContent={content} />
      </EditorRoot>
    );

    // Content should be rendered
    expect(screen.getByRole("textbox")).toBeInTheDocument();
  });
});
```

---

### Test Coverage Goals

```bash
# Run coverage report
pnpm test:coverage

# Output:
---------------------------|---------|----------|---------|---------|
File                       | % Stmts | % Branch | % Funcs | % Lines |
---------------------------|---------|----------|---------|---------|
All files                  |   82.45 |    75.32 |   85.12 |   83.21 |
 components/               |   90.12 |    85.43 |   92.34 |   91.23 |
  editor.tsx               |   95.23 |    90.12 |   96.45 |   95.67 |
  editor-bubble.tsx        |   88.34 |    82.45 |   89.12 |   89.45 |
 extensions/               |   78.45 |    70.23 |   80.12 |   79.34 |
  ai-highlight.ts          |   85.67 |    78.90 |   87.23 |   86.45 |
  slash-command/           |   72.34 |    65.12 |   75.45 |   73.56 |
 utils/                    |   92.34 |    88.45 |   94.23 |   93.12 |
  index.ts                 |   95.67 |    92.34 |   96.78 |   96.12 |
---------------------------|---------|----------|---------|---------|
```

**Coverage Goals**:
- Overall: >80%
- Critical paths (editor core, extensions): >90%
- Utils: >95%
- UI components: >85%

---

## Code Quality and Linting

### Introduction: Automated Code Quality

**Manual Code Review**:
- ❌ Slow (every PR needs human review)
- ❌ Inconsistent (different reviewers, different standards)
- ❌ Misses simple issues (formatting, unused variables)

**Automated Linting**:
- ✅ Fast (runs in seconds)
- ✅ Consistent (same rules every time)
- ✅ Catches common issues before review

---

### Biome Configuration

**File**: `biome.json`

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.3/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noExplicitAny": "warn"
      },
      "complexity": {
        "noForEach": "off"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "formatWithErrors": false,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineEnding": "lf",
    "lineWidth": 100,
    "attributePosition": "auto"
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double",
      "trailingComma": "es5",
      "semicolons": "always",
      "arrowParentheses": "always"
    }
  }
}
```

---

### Running Linter

```bash
# Check for issues
pnpm lint

# Fix auto-fixable issues
pnpm lint:fix

# Format code
pnpm format

# Type check
pnpm typecheck
```

**Pre-commit Hook** (automatic):

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

pnpm lint-staged
```

**File**: `package.json`

```json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": [
      "biome check --apply --no-errors-on-unmatched",
      "git add"
    ]
  }
}
```

**What happens on `git commit`**:
```
1. Husky intercepts commit
   ↓
2. Runs lint-staged
   ↓
3. Biome checks staged files
   ↓
4. Auto-fixes issues
   ↓
5. Adds fixes to commit
   ↓
6. Commit succeeds (or fails if non-fixable issues)
```

---

### TypeScript Configuration

**File**: `tsconfig.json`

```json
{
  "compilerOptions": {
    // Strict type checking
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    // Module resolution
    "moduleResolution": "bundler",
    "module": "ESNext",
    "target": "ES2020",

    // JSX
    "jsx": "preserve",
    "jsxImportSource": "react",

    // Path mapping
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "novel": ["../../packages/headless/src"],
      "novel/*": ["../../packages/headless/src/*"]
    },

    // Output
    "outDir": "./dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,

    // Other
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src/**/*", "test/**/*"],
  "exclude": ["node_modules", "dist", ".next"]
}
```

---

### Quality Checks in CI

**GitHub Actions** (`.github/workflows/ci.yml`):

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Type check
        run: pnpm typecheck

      - name: Lint
        run: pnpm lint

      - name: Test
        run: pnpm test:coverage

      - name: Build
        run: pnpm build

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

**Quality Gates**:
- ✅ All type checks pass
- ✅ All lint checks pass
- ✅ All tests pass
- ✅ Coverage >80%
- ✅ Build succeeds

---

## Publishing to npm

### Introduction: Versioning and Package Management

**Novel is published as multiple packages**:

```
@novel/headless   - Core editor (framework-agnostic)
@novel/tailwind   - Tailwind-styled components
novel             - Complete package (re-exports both)
```

---

### Publishing Workflow

**Step 1: Update Version**

```bash
# From package directory
cd packages/headless

# Bump version (follows semver)
pnpm version patch   # 1.0.0 → 1.0.1 (bug fixes)
pnpm version minor   # 1.0.0 → 1.1.0 (new features)
pnpm version major   # 1.0.0 → 2.0.0 (breaking changes)
```

**This updates**:
- `package.json` version
- Creates git tag
- Commits version bump

---

### Semantic Versioning (semver)

**Format**: `MAJOR.MINOR.PATCH` (e.g., `2.5.3`)

| Change Type | Version | Example |
|-------------|---------|---------|
| **Breaking change** | MAJOR | `2.0.0 → 3.0.0` |
| **New feature** | MINOR | `2.5.0 → 2.6.0` |
| **Bug fix** | PATCH | `2.5.3 → 2.5.4` |

**Examples**:

```typescript
// PATCH: Bug fix (2.5.3 → 2.5.4)
// Before: Image upload breaks on large files
// After: Fixed size validation

// MINOR: New feature (2.5.0 → 2.6.0)
// Before: No comment support
// After: Added Comment extension (backward compatible)

// MAJOR: Breaking change (2.0.0 → 3.0.0)
// Before: extensions={{ basic: [StarterKit] }}
// After: extensions={[StarterKit]}  // API changed
```

---

### Pre-release Versions

```bash
# Beta releases
pnpm version 3.0.0-beta.1
pnpm version 3.0.0-beta.2

# Release candidate
pnpm version 3.0.0-rc.1

# Stable release
pnpm version 3.0.0
```

**Publishing pre-release**:

```bash
# Publish with beta tag
pnpm publish --tag beta

# Users install with:
pnpm add novel@beta
```

---

### Step 2: Build for Production

```bash
# Clean previous builds
pnpm clean

# Build all packages
pnpm build

# Verify build output
ls -la dist/

# Test built package locally
pnpm pack
# Creates: novel-headless-2.5.4.tgz

# Test in another project
cd /path/to/test-project
pnpm add /path/to/novel/packages/headless/novel-headless-2.5.4.tgz
```

---

### Step 3: Publish to npm

```bash
# Login to npm (first time only)
npm login

# Publish package
pnpm publish --access public

# Or publish all workspace packages
pnpm -r publish --access public
```

**What happens**:
```
1. Runs prepublishOnly script (build, test)
   ↓
2. Packages dist/ folder
   ↓
3. Uploads to npm registry
   ↓
4. Available at: https://www.npmjs.com/package/novel
   ↓
5. Users can install: pnpm add novel
```

---

### Publishing Checklist

```markdown
## Pre-publish Checklist

- [ ] All tests pass (`pnpm test`)
- [ ] Linter passes (`pnpm lint`)
- [ ] TypeScript compiles (`pnpm typecheck`)
- [ ] Build succeeds (`pnpm build`)
- [ ] Changelog updated (`CHANGELOG.md`)
- [ ] Version bumped (`pnpm version X.Y.Z`)
- [ ] Git tag created
- [ ] Tested locally with `pnpm pack`
- [ ] Documentation updated
- [ ] Breaking changes documented (if MAJOR)
- [ ] Migration guide written (if MAJOR)

## Publish

- [ ] `pnpm publish --access public`
- [ ] Git push with tags (`git push --tags`)
- [ ] GitHub release created
- [ ] Announcement (Twitter, Discord, etc.)
- [ ] Update examples/demos if needed
```

---

## Versioning Strategy

### Changelog Management

**File**: `CHANGELOG.md`

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Comment system with threading
- Collaborative editing with Yjs

### Changed
- Improved image upload performance

### Deprecated
- Old extension registration API (will be removed in v3.0.0)

### Removed
- Nothing

### Fixed
- Slash command crash on mobile
- Memory leak in image upload

### Security
- Updated dependencies with security patches

## [2.5.4] - 2024-11-19

### Fixed
- Fixed image upload size validation
- Fixed bubble menu positioning on mobile

## [2.5.3] - 2024-11-15

### Added
- AIHighlight extension for AI-generated text
- Mathematics extension for LaTeX

### Changed
- Optimized editor initialization (450ms → 120ms)

## [2.5.0] - 2024-11-01

### Added
- Edge runtime support for API routes
- Streaming AI completions
- Rate limiting with Upstash Redis

### Changed
- Migrated from OpenAI SDK to Vercel AI SDK

[Unreleased]: https://github.com/steven-tey/novel/compare/v2.5.4...HEAD
[2.5.4]: https://github.com/steven-tey/novel/compare/v2.5.3...v2.5.4
[2.5.3]: https://github.com/steven-tey/novel/compare/v2.5.0...v2.5.3
[2.5.0]: https://github.com/steven-tey/novel/releases/tag/v2.5.0
```

---

### Migration Guides

**For Breaking Changes** (MAJOR version):

**File**: `docs/MIGRATION.md`

```markdown
# Migration Guide

## Upgrading from v2.x to v3.0

### Breaking Changes

#### 1. Extension Registration API

**Before (v2.x)**:
```typescript
<EditorContent
  extensions={{
    basic: [StarterKit],
    custom: [MyExtension],
  }}
/>
```

**After (v3.0)**:
```typescript
<EditorContent
  extensions={[StarterKit, MyExtension]}
/>
```

**Why**: Simplified API, better TypeScript inference.

**Migration**:
```typescript
// Find all instances of:
extensions={{

// Replace with:
extensions={[
  // Flatten nested arrays
]}
```

---

#### 2. Image Upload Handler

**Before (v2.x)**:
```typescript
onImageUpload={(file) => uploadToServer(file)}
```

**After (v3.0)**:
```typescript
imageUploadOptions={{
  onUpload: (file) => uploadToServer(file),
  validateFn: (file) => file.size < 5MB,
}}
```

**Why**: More flexible, supports validation.

---

### New Features in v3.0

- Collaborative editing with Yjs
- Comment system
- Enhanced AI features
- Performance improvements

### Deprecations

- `<Editor />` component → Use `<EditorRoot>` + `<EditorContent>`
- Will be removed in v4.0

### Full Changelog

See [CHANGELOG.md](./CHANGELOG.md) for detailed changes.
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

**File**: `.github/workflows/release.yml`

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run tests
        run: pnpm test

      - name: Build packages
        run: pnpm build

      - name: Publish to npm
        run: pnpm -r publish --access public --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            See [CHANGELOG.md](https://github.com/steven-tey/novel/blob/main/CHANGELOG.md)
          draft: false
          prerelease: false
```

---

### Automated Testing

**File**: `.github/workflows/test.yml`

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [18, 20]

    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Type check
        run: pnpm typecheck

      - name: Lint
        run: pnpm lint

      - name: Test
        run: pnpm test:coverage

      - name: Build
        run: pnpm build
```

---

### Deployment Pipeline

**For Demo App** (`apps/web`):

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          BLOB_READ_WRITE_TOKEN: ${{ secrets.BLOB_READ_WRITE_TOKEN }}

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

---

## Summary

### Development Workflow Checklist

**Before Starting Work**:
- [ ] Pull latest `main` branch
- [ ] Create feature branch (`feat/feature-name`)
- [ ] Set up environment variables

**During Development**:
- [ ] Make small, focused commits
- [ ] Write meaningful commit messages
- [ ] Add tests for new features
- [ ] Run linter and tests locally
- [ ] Keep branch up to date with `main`

**Before Creating PR**:
- [ ] Self-review changes
- [ ] Run full test suite
- [ ] Update documentation
- [ ] Update changelog
- [ ] Ensure CI passes

**Code Review**:
- [ ] Address all feedback
- [ ] Re-request review after changes
- [ ] Squash or clean up commits

**After Merge**:
- [ ] Delete feature branch
- [ ] Update local `main`
- [ ] Monitor for issues

**Publishing**:
- [ ] Update version (semver)
- [ ] Update changelog
- [ ] Create git tag
- [ ] Test build locally
- [ ] Publish to npm
- [ ] Create GitHub release
- [ ] Announce release

---

## Next Steps

1. **Set up your development environment**
   - Install prerequisites
   - Clone repository
   - Configure editor

2. **Practice the workflow**
   - Create a feature branch
   - Make a small change
   - Create a PR
   - Get it reviewed and merged

3. **Learn the tools**
   - Git (branching, rebasing, cherry-picking)
   - pnpm workspaces
   - Turborepo
   - GitHub Actions

4. **Read the docs**
   - [Git Documentation](https://git-scm.com/doc)
   - [Conventional Commits](https://www.conventionalcommits.org/)
   - [Semantic Versioning](https://semver.org/)
   - [pnpm Workspaces](https://pnpm.io/workspaces)

---

**Document Version**: 1.0
**Last Updated**: November 2025
**Related Guides**:
- [GETTING_STARTED.md](./GETTING_STARTED.md) - Initial setup
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Coding patterns
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step tutorials

---

*End of DEVELOPMENT_WORKFLOW.md*