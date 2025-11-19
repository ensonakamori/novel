# Documentation Execution Plan

**Created:** November 19, 2025
**Status:** In Progress
**Phase:** 1 - Analysis & Planning

---

## Project Overview

**Project Name:** Novel - Notion-style WYSIWYG Editor
**Type:** Monorepo (Turborepo + pnpm)
**Primary Technologies:**
- Tiptap v2.11.2 (Rich text editor core)
- Next.js v15.1.4 (Demo app)
- React v18.2.0 (UI library)
- Jotai v2.11.0 (State management)
- Tailwind CSS v3.4.1 (Styling)
- TypeScript v5.7.3 (Type system)
- Vercel AI SDK v3.0.12 (AI features)

**Repository Structure:**
- `/packages/headless` - Core editor library (published as "novel" on npm)
- `/apps/web` - Next.js reference implementation
- `/packages/tsconfig` - Shared TypeScript configs

**Purpose:** Provide a reusable, headless, AI-powered rich text editor component library

---

## Documentation Goals

Create comprehensive learning materials for:
1. **New developers** joining the project (mid-level, React/TS proficient)
2. **Contributors** wanting to extend or customize the editor
3. **Integrators** using the library in their own projects

---

## Planned Documentation Structure

```
docs/learning/
├── README.md                      # ✅ Central navigation hub (PHASE 2)
├── TECH_STACK_RESEARCH.md         # ✅ COMPLETED (PHASE 0)
├── EXECUTION_PLAN.md              # ✅ COMPLETED (PHASE 1)
│
├── GETTING_STARTED.md             # PHASE 3.1
├── ARCHITECTURE_OVERVIEW.md       # PHASE 3.2
├── PROJECT_STRUCTURE.md           # PHASE 3.3
├── TECH_STACK_GUIDE.md            # PHASE 3.4
├── DATA_FLOW_GUIDE.md             # PHASE 3.5
│
├── FRONTEND_ARCHITECTURE.md       # PHASE 4.1
├── EDITOR_CORE_GUIDE.md           # PHASE 4.2 (renamed from BACKEND)
├── STATE_MANAGEMENT.md            # PHASE 4.3 (NEW)
├── INTEGRATION_GUIDE.md           # PHASE 4.4
│
├── PATTERNS_AND_CONVENTIONS.md    # PHASE 5.1
├── HOW_TO_GUIDE.md                # PHASE 5.2
├── CODE_TOURS.md                  # PHASE 5.3
├── DEVELOPMENT_WORKFLOW.md        # PHASE 5.4
│
├── TESTING_GUIDE.md               # PHASE 6.1
├── DEBUGGING_GUIDE.md             # PHASE 6.2
├── AI_FEATURES_GUIDE.md           # PHASE 6.3 (NEW - AI integration)
├── API_DOCUMENTATION.md           # PHASE 6.4
├── EXTENSION_DEVELOPMENT.md       # PHASE 6.5 (NEW - custom extensions)
│
├── EXERCISES.md                   # PHASE 7.1
├── FIRST_CONTRIBUTIONS.md         # PHASE 7.2
│
├── FAQ.md                         # PHASE 8
└── diagrams/
    ├── architecture.mmd
    ├── data-flow.mmd
    ├── editor-lifecycle.mmd
    └── extension-system.mmd
```

---

## Phase-by-Phase Breakdown

### ✅ PHASE 0: TECHNOLOGY RESEARCH (COMPLETED)

**Status:** Complete
**Commit:** `bca3799`

**Output:**
- Created `TECH_STACK_RESEARCH.md`
- Researched all major dependencies
- Documented version status (current/outdated/deprecated)
- Identified upgrade paths

**Key Findings:**
- React 18 (current: 19) - major version behind
- Tailwind v3 (current: v4) - major version behind
- AI SDK v3 (current: v5) - major version behind
- Most other dependencies are current or recent

---

### ✅ PHASE 1: ANALYSIS & PLANNING (IN PROGRESS)

**Status:** In Progress
**Started:** November 19, 2025

**Completed:**
- ✅ Deep codebase exploration
- ✅ Identified all key components and patterns
- ✅ Mapped architecture and system design
- ✅ Created this execution plan

**Next:**
- Commit this execution plan
- Move to Phase 2

---

### PHASE 2: CREATE CENTRAL LEARNING PATH

**Target:** `docs/learning/README.md`

**Purpose:** Central navigation hub linking all documentation

**Sections:**
1. Welcome & Quick Links
2. Learning Paths (by role: new dev, contributor, integrator)
3. Technology Stack Overview (link to TECH_STACK_RESEARCH.md)
4. Document Index with descriptions
5. Getting Help & Resources

**Estimated Time:** 30-45 minutes
**Commit Message:** "docs: create central learning path README with tech stack overview"

---

### PHASE 3: FOUNDATION DOCUMENTS

Core understanding documents for new developers.

#### 3.1 - GETTING_STARTED.md

**Content:**
- Prerequisites (Node.js, pnpm, environment setup)
- Clone repository
- Install dependencies (`pnpm install`)
- Development commands (`pnpm dev`, `pnpm build`, etc.)
- Project structure overview
- First-time walkthrough
- Environment variables setup
- Common issues and troubleshooting

**Links to:**
- Package.json scripts
- Turbo.json tasks
- .env.example (if exists)

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add getting started guide with version notes"

---

#### 3.2 - ARCHITECTURE_OVERVIEW.md

**Content:**
- High-level system architecture
- Monorepo design (why Turborepo + pnpm)
- Package relationships diagram
- Component architecture (EditorRoot, EditorContent, etc.)
- Extension system overview
- State management approach (Jotai)
- API design (headless pattern)
- Mermaid diagrams:
  - System architecture
  - Component hierarchy
  - Extension flow

**Mental Models:**
- 🧠 "Editor as a composition of extensions"
- 🌉 Bridge from React components to Tiptap extensions
- 💡 Headless = Logic + Structure, No Styling

**Code Examples:**
- Basic editor setup
- Extension composition
- State access patterns

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add architecture overview with diagrams"

---

#### 3.3 - PROJECT_STRUCTURE.md

**Content:**
- Directory tree with explanations
- File naming conventions
- Import/export patterns
- Package boundaries
- Build outputs

**Sections:**
1. Monorepo Layout
2. Headless Package Structure
   - /src/components
   - /src/extensions
   - /src/plugins
   - /src/utils
3. Web App Structure
   - /app (Next.js App Router)
   - /components
   - /lib
   - /styles
4. Shared Configs (/packages/tsconfig)
5. Configuration Files (root level)

**Every directory linked to actual code**

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add project structure guide"

---

#### 3.4 - TECH_STACK_GUIDE.md

**Content:**
For each major technology:

1. **Tiptap** (Core Editor)
   - What it is, why used
   - Key concepts (Editor, Extensions, Nodes, Marks)
   - Compared to: Draft.js, Slate, Quill
   - Links to official docs
   - Examples in this project

2. **Next.js 15** (Demo App)
   - App Router patterns
   - Server Components
   - API Routes (Edge runtime)
   - Links to examples in /app

3. **React 18**
   - Hooks used in project
   - Component patterns
   - ⚠️ Note: React 19 is available

4. **Jotai** (State Management)
   - Atomic state approach
   - Atoms in this project (queryAtom, rangeAtom)
   - Store isolation (novelStore)
   - Compared to: Zustand, Redux, Context

5. **Tailwind CSS**
   - Utility-first approach
   - Theme customization
   - ⚠️ Note: v4 is available (major rewrite)

6. **Vercel AI SDK**
   - Streaming completions
   - Integration patterns
   - ⚠️ Note: v5 is available (breaking changes)

7. **TypeScript**
   - Type patterns in project
   - Extension typing
   - JSONContent types

**Each section:**
- Brief explanation
- Why it's used here
- Key concepts
- Code examples from repo
- Official docs links
- Version status (from TECH_STACK_RESEARCH.md)

**Estimated Time:** 90 minutes
**Commit Message:** "docs: add tech stack guide with current practices"

---

#### 3.5 - DATA_FLOW_GUIDE.md

**Content:**
- Editor initialization lifecycle
- User interaction flow (typing → state → render)
- Command execution flow (slash commands)
- Extension registration and execution
- State updates via Jotai
- AI completion flow (user input → API → streaming → insertion)
- Image upload flow (drop/paste → upload → URL → insertion)

**Mermaid Diagrams:**
1. Editor Lifecycle
2. Slash Command Flow
3. AI Generation Flow
4. Image Upload Flow
5. Extension Hook Execution

**Code Examples:**
- EditorRoot setup
- Extension configuration
- Command handler
- API route → frontend integration

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add data flow guide with diagrams"

---

### PHASE 4: DEEP-DIVE DOCUMENTS

Technology-specific deep dives.

#### 4.1 - FRONTEND_ARCHITECTURE.md

**Content:**
- Next.js App Router structure
- Component organization (selectors, generative, ui)
- Styling approach (Tailwind + CSS variables)
- Theme system (next-themes)
- shadcn/ui integration
- Icon system
- Responsive design patterns

**Code Examples:**
- Component composition
- Selector patterns
- Theme switching
- shadcn/ui usage

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add frontend architecture guide"

---

#### 4.2 - EDITOR_CORE_GUIDE.md

**Content:**
- Tiptap fundamentals
- Extension system deep dive
- Node vs Mark vs Extension types
- Creating custom extensions
- Plugin development (ProseMirror plugins)
- Commands and keymaps
- Decorations and NodeViews

**Sections:**
1. Tiptap Core Concepts
2. Extension Anatomy
3. Built-in Extensions Used
4. Custom Extensions in Novel:
   - AIHighlight
   - CustomKeymap
   - SlashCommand
   - Twitter
   - Mathematics
   - UpdatedImage
5. Plugin System (UploadImagesPlugin)
6. Suggestion System (slash commands)

**Code Examples:**
- Extension structure
- Command implementation
- Plugin creation
- NodeView rendering

**Estimated Time:** 90 minutes
**Commit Message:** "docs: add editor core guide with extension system"

---

#### 4.3 - STATE_MANAGEMENT.md

**Content:**
- Why Jotai over other solutions
- Atom-based architecture
- Store isolation (novelStore)
- Atoms used in Novel:
  - queryAtom (slash command search)
  - rangeAtom (selection range)
- Creating new atoms
- Accessing state in components
- State updates and effects

**Mental Model:**
- 🧠 "Atoms are independent state containers"
- 🌉 Compared to: React Context, Zustand, Redux

**Code Examples:**
- Atom definition
- Reading atoms
- Writing atoms
- Store usage

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add state management guide with Jotai patterns"

---

#### 4.4 - INTEGRATION_GUIDE.md

**Content:**
- Installing novel package (`npm install novel`)
- Basic setup
- Configuration options
- Using with different frameworks:
  - Next.js 13+ (App Router)
  - Next.js (Pages Router)
  - React (Create React App)
  - Vite
  - Remix
- Custom styling
- Extension configuration
- AI integration (optional)
- Image upload integration

**Code Examples:**
- Minimal setup
- Next.js integration
- Vite integration
- Custom extension configuration
- Theming

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add integration guide for different frameworks"

---

### PHASE 5: PRACTICAL GUIDES

Hands-on guides for common tasks.

#### 5.1 - PATTERNS_AND_CONVENTIONS.md

**Content:**
- Code style (Biome config)
- File naming conventions
- Import/export patterns
- Component patterns:
  - Composition
  - Provider pattern
  - Headless component pattern
- TypeScript patterns:
  - Extension typing
  - Prop types
  - Generic components
- State patterns (Jotai atoms)
- Error handling
- Async patterns

**Mark outdated patterns:**
- ⚠️ React 18 patterns (note React 19 changes)
- ⚠️ Tailwind v3 utilities (note v4 differences)

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add patterns and conventions with currency markers"

---

#### 5.2 - HOW_TO_GUIDE.md

**Content:**
Step-by-step guides for common tasks:

1. How to add a new slash command
2. How to create a custom extension
3. How to add a new toolbar button
4. How to customize editor styles
5. How to add a new AI prompt
6. How to handle custom file uploads
7. How to add keyboard shortcuts
8. How to customize the bubble menu
9. How to add a new node type
10. How to persist editor content

**Each guide:**
- Step-by-step instructions
- Code examples
- File references
- Testing verification

**Estimated Time:** 90 minutes
**Commit Message:** "docs: add comprehensive how-to guide"

---

#### 5.3 - CODE_TOURS.md

**Content:**
Guided walkthroughs of key features:

**Tour 1: "Follow a Keystroke"**
- User types "/" → slash command appears
- Trace through code from input to render

**Tour 2: "AI Completion Flow"**
- User triggers AI → streaming → insertion
- Frontend → API → OpenAI → Response

**Tour 3: "Image Upload Journey"**
- Drag/drop → plugin detection → upload → insertion
- Frontend → API → Blob storage → URL

**Tour 4: "Extension Lifecycle"**
- Extension registration → initialization → usage
- Following a custom extension

**Tour 5: "State Update Flow"**
- Atom update → component re-render
- Jotai state management in action

**Each tour:**
- File-by-file walkthrough
- Line number references
- Mermaid sequence diagram
- Key concepts highlighted

**Estimated Time:** 75 minutes
**Commit Message:** "docs: add code tours for key features"

---

#### 5.4 - DEVELOPMENT_WORKFLOW.md

**Content:**
- Setting up development environment
- Running the monorepo (`pnpm dev`)
- Making changes to headless package
- Testing changes in web app
- Linting and formatting (Biome)
- Type checking (TypeScript)
- Git workflow:
  - Branch naming
  - Commit conventions (commitlint)
  - Pre-commit hooks (husky)
- Using Changesets for versioning
- Building for production
- Publishing to npm

**Commands Reference:**
- `pnpm dev` - Start dev mode
- `pnpm lint` - Lint code
- `pnpm lint:fix` - Fix lint issues
- `pnpm format:fix` - Format code
- `pnpm typecheck` - Type check
- `pnpm build` - Build all packages
- `pnpm changeset` - Create changeset
- `pnpm version:packages` - Bump versions
- `pnpm publish:packages` - Publish to npm

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add development workflow guide"

---

### PHASE 6: QUALITY & REFERENCE DOCS

#### 6.1 - TESTING_GUIDE.md

**Content:**
- Current testing setup (if any)
- ⚠️ UNCLEAR: No visible test files in initial exploration
- Recommended testing approach:
  - Unit tests for utilities
  - Component tests for editor components
  - Integration tests for extensions
  - E2E tests for web app
- Testing tools recommendations:
  - Vitest (fast, Vite-compatible)
  - React Testing Library
  - Playwright (E2E)
- Writing tests for:
  - Custom extensions
  - Commands
  - Plugins
  - State updates

**Note:** Mark sections as 🔍 NEEDS VERIFICATION if tests don't exist

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add testing guide with recommendations"

---

#### 6.2 - DEBUGGING_GUIDE.md

**Content:**
- Using browser DevTools with editor
- Inspecting Tiptap state
- Debugging extensions
- Common issues:
  - Extensions not loading
  - Commands not working
  - State not updating
  - Styling issues
  - Build errors
- Debugging AI completions
- Debugging image uploads
- Performance profiling
- Source maps

**Troubleshooting Checklist:**
- Extension registration
- Peer dependencies
- React key warnings
- Type errors
- Tailwind class conflicts

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add debugging guide"

---

#### 6.3 - AI_FEATURES_GUIDE.md

**Content:**
- AI integration architecture
- OpenAI API integration
- Vercel AI SDK usage
- Streaming completions
- Rate limiting (Upstash)
- Custom prompts
- AI selector UI
- AI completion commands
- Error handling
- Cost optimization

**Code Examples:**
- API route setup
- Frontend streaming integration
- Custom prompt implementation
- Rate limit configuration

**Estimated Time:** 60 minutes
**Commit Message:** "docs: add AI features integration guide"

---

#### 6.4 - API_DOCUMENTATION.md

**Content:**

**Headless Package API:**

1. **Components:**
   - `<EditorRoot>`
   - `<EditorContent>`
   - `<EditorBubble>`
   - `<EditorCommand>`
   - All exported component props

2. **Extensions:**
   - All custom extensions
   - Configuration options
   - Commands exposed
   - Example usage

3. **Utilities:**
   - `isValidUrl()`
   - `getUrlFromString()`
   - `getPrevText()`
   - `getAllContent()`

4. **Plugins:**
   - `UploadImagesPlugin`
   - `createImageUpload()`
   - Configuration types

5. **State (Atoms):**
   - `queryAtom`
   - `rangeAtom`
   - `novelStore`

**Web App API Routes:**
- `/api/generate` - AI completion
- `/api/upload` - Image upload

**Estimated Time:** 75 minutes
**Commit Message:** "docs: add comprehensive API documentation"

---

#### 6.5 - EXTENSION_DEVELOPMENT.md

**Content:**
- Extension development guide
- Extension types (Node, Mark, Extension)
- Extension API reference
- Creating custom nodes
- Creating custom marks
- Adding commands
- Adding keyboard shortcuts
- Adding input rules
- Adding paste rules
- Styling extensions
- Testing extensions
- Publishing extensions

**Code Examples:**
- Full extension implementation
- Node extension (custom block)
- Mark extension (custom formatting)
- Extension with commands
- Extension with input rules

**Estimated Time:** 75 minutes
**Commit Message:** "docs: add extension development guide"

---

### PHASE 7: LEARNING EXERCISES

#### 7.1 - EXERCISES.md

**Content:**

**Beginner Exercises:**
1. Add a new slash command (e.g., /note)
2. Add a new bubble menu button (e.g., highlight)
3. Customize editor theme colors
4. Add a new keyboard shortcut

**Intermediate Exercises:**
5. Create a custom node extension (e.g., callout box)
6. Add a custom AI prompt
7. Implement custom file upload handling
8. Add character limit feature

**Advanced Exercises:**
9. Create a collaborative editing extension
10. Build a custom suggestion system
11. Implement custom paste handling
12. Create a plugin for tracking changes

**Each exercise:**
- Learning objectives
- Step-by-step instructions
- Solution code
- Testing criteria
- Further exploration ideas

**Estimated Time:** 90 minutes
**Commit Message:** "docs: add hands-on exercises"

---

#### 7.2 - FIRST_CONTRIBUTIONS.md

**Content:**
- Finding good first issues
- Understanding the codebase
- Making your first change:
  1. Pick a small enhancement
  2. Create a branch
  3. Make changes
  4. Test locally
  5. Create changeset
  6. Submit PR
- PR guidelines
- Code review process
- Getting help

**Suggested First Contributions:**
- Add a new icon
- Add a new slash command
- Improve documentation
- Fix a small bug
- Add a test

**Estimated Time:** 45 minutes
**Commit Message:** "docs: add first contributions guide"

---

### PHASE 8: ACCURACY REVIEW & FINALIZATION

**Tasks:**

1. **Review All Documents** (60 minutes)
   - Check for uncertainty markers
   - Verify code links work
   - Ensure all assumptions marked
   - Confirm tech stack info current

2. **Update Central README** (30 minutes)
   - Add links to all created docs
   - Update navigation
   - Add quick reference

3. **Create FAQ** (45 minutes)
   - Common questions from docs
   - Troubleshooting Q&A
   - Integration questions
   - Extension development questions

4. **Final Polish** (30 minutes)
   - Fix typos
   - Standardize formatting
   - Ensure consistency
   - Verify all links

**Commits:**
- "docs: accuracy review pass - verified uncertainty markers"
- "docs: finalize learning path with all cross-references"
- "docs: add FAQ"
- "docs: final polish and validation"

---

## Key Principles for All Documents

### Required Elements

**Every document must include:**
1. Clear title and purpose statement
2. Version/currency note (e.g., "Documented: Nov 2025 | Stack Research: [link]")
3. Table of contents (if >500 lines)
4. Progressive sections (beginner → advanced)
5. "Next Steps" linking to related docs

### Accuracy Requirements

✅ **DO:**
- Verify functionality in code before documenting
- Mark uncertainties clearly
- Link to current official docs (Nov 2025)
- Distinguish facts from inferences
- Flag deprecated patterns
- Provide investigation paths

❌ **DON'T:**
- Fabricate explanations for unclear code
- Present assumptions as facts
- Document without verification
- Skip uncertainty markers

### Pedagogical Elements

Use throughout all docs:
- 🧠 **Mental Model** - Simplified conceptualization
- 🌉 **Bridge from React/JS/TS** - Familiar analogies
- 💡 **Aha Moment** - Key insight
- 🎯 **Remember This** - Mnemonic
- ⚠️ **Common Pitfall** - What to avoid
- 🔗 **Code Example** - Actual code links
- ✅ **Quick Check** - Self-test
- 🔍 **Needs Investigation** - Deeper exploration needed
- ⚠️ **Uncertainty Marker** - Not confident
- 🚧 **TODO** - Needs verification
- ✅ **Current (Nov 2025)** - Up-to-date pattern
- ⚠️ **Outdated Pattern** - Newer approaches exist
- 🚨 **Deprecated** - Don't use in new code
- 🆕 **New in 2025** - Recently introduced

### Code References

**Every concept must:**
- Link to actual code with line numbers: `[example](../path/to/file.ts#L45-L67)`
- Show both good and bad examples
- Explain WHY examples are good/bad
- Mark legacy/outdated code

### Version Awareness

- Note when patterns are version-specific
- Provide migration paths for outdated code
- Link to version-specific documentation
- Reference TECH_STACK_RESEARCH.md findings

---

## Commit Strategy

**Commit after:**
- Each major document (PHASE 3-7 items)
- Every 30-45 minutes of work
- Completing accuracy review
- Any substantial addition (500+ lines)

**Commit Message Format:**
```
docs: <clear description>

- What was added/changed
- Why it's helpful
- Any version/currency notes
```

---

## Estimated Timeline

| Phase | Estimated Time | Cumulative |
|-------|---------------|------------|
| Phase 0 | ✅ Complete | 1.5 hours |
| Phase 1 | ✅ Complete | 2.5 hours |
| Phase 2 | 45 min | 3.25 hours |
| Phase 3 | 5 hours | 8.25 hours |
| Phase 4 | 5 hours | 13.25 hours |
| Phase 5 | 5 hours | 18.25 hours |
| Phase 6 | 5.5 hours | 23.75 hours |
| Phase 7 | 2.25 hours | 26 hours |
| Phase 8 | 2.75 hours | **28.75 hours** |

**Total Estimated Time:** ~29 hours of focused documentation work

---

## Success Criteria

✅ **Complete when:**
- All 20+ documents created
- All documents follow style guide
- All code examples verified
- All links working
- Uncertainty clearly marked
- Tech stack info accurate (Nov 2025)
- Navigation structure clear
- Exercises tested
- FAQ comprehensive
- Final accuracy review passed

---

## Next Steps

1. Commit this execution plan
2. Begin Phase 2: Create central README
3. Execute phases 3-8 sequentially
4. Commit frequently
5. Push all changes when complete

---

**Document Status:** ✅ Complete
**Last Updated:** November 19, 2025
**Next Phase:** Phase 2 - Central Learning Path README
