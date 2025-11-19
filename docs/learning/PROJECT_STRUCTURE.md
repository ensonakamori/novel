# Project Structure: A Deep Dive

**Last Updated:** November 19, 2025
**Target Audience:** Mid-level developers becoming full-stack architects
**Reading Time:** 45-60 minutes

This is NOT just a directory tree. This is your **architectural guide** to understanding how a modern, production-grade editor library is organized, why it's structured this way, and what patterns you should adopt in your own projects.

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [Root-Level Organization](#root-level-organization)
3. [Deep Dive: packages/headless](#deep-dive-packagesheadless)
4. [Deep Dive: apps/web](#deep-dive-appsweb)
5. [Shared Configurations](#shared-configurations)
6. [File Naming Conventions](#file-naming-conventions)
7. [Import/Export Patterns](#importexport-patterns)
8. [Architectural Patterns in File Organization](#architectural-patterns-in-file-organization)
9. [Build Output Structure](#build-output-structure)
10. [Common Pitfalls & Best Practices](#common-pitfalls--best-practices)

---

## The Big Picture

### The Monorepo Philosophy

```
novel/                                  ← You are here
├── apps/                              ← Applications (consumers)
│   └── web/                           ← Demo Next.js app
├── packages/                          ← Reusable libraries
│   ├── headless/                      ← Core editor (published)
│   └── tsconfig/                      ← Shared TS configs
├── docs/                              ← Documentation
├── .github/                           ← CI/CD workflows
├── .husky/                            ← Git hooks
├── .changeset/                        ← Version management
└── Configuration files                ← Root-level configs
```

🧠 **Mental Model**: Think of this as a **mini ecosystem**:
- **packages/** = Your internal npm organization
- **apps/** = Applications that consume packages
- **Root configs** = Shared tooling for everyone

🌉 **Compared to**:
- **Multi-repo**: Each package is separate (painful to sync)
- **Monolith**: Everything in one package (hard to publish parts)
- **Monorepo**: Best of both worlds (one repo, multiple packages)

---

## Root-Level Organization

### Configuration Files (The Foundation)

Every file at the root level serves a specific purpose. Let's understand WHY each exists:

```
novel/
├── package.json              ← Workspace root configuration
├── pnpm-workspace.yaml       ← Defines workspace packages
├── turbo.json                ← Build orchestration
├── biome.json                ← Linting & formatting rules
├── prettier.config.js        ← Prettier configuration
├── tsconfig.json             ← Base TypeScript config (minimal)
├── .gitignore                ← Git ignore rules
├── LICENSE                   ← Apache 2.0 license
├── README.md                 ← Public-facing documentation
└── SECURITY.md               ← Security policy
```

---

#### package.json (The Orchestrator)

**Location**: `/package.json`

**Purpose**: Defines the **workspace root** - this is NOT a publishable package.

**Key Sections**:

```json
{
  "name": "novel",
  "private": true,                    // ← CRITICAL: Prevents accidental publish
  "scripts": {
    "dev": "turbo dev",               // ← Runs dev in ALL packages
    "build": "turbo build",           // ← Builds ALL packages
    "lint": "turbo lint --continue",  // ← Lints ALL packages
    "typecheck": "turbo typecheck"    // ← Type-checks ALL packages
  },
  "dependencies": {
    "@changesets/cli": "^2.27.11",    // ← Version management
    "turbo": "^2.3.3"                  // ← Monorepo orchestrator
  },
  "packageManager": "pnpm@9.5.0"      // ← Enforces pnpm version
}
```

🎯 **Architectural Decision**:
- **`private: true`** prevents accidentally publishing the root package
- **Turbo scripts** run tasks across all packages in parallel
- **`packageManager` field** ensures team uses same pnpm version

💡 **Pattern to Learn**: In monorepos, the root package.json is a **task orchestrator**, not a library.

---

#### pnpm-workspace.yaml (Package Discovery)

**Location**: `/pnpm-workspace.yaml`

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**Purpose**: Tells pnpm which directories contain packages.

🧠 **How it works**:
1. pnpm scans `apps/*` and `packages/*`
2. Finds `package.json` in each subdirectory
3. Creates symlinks in `node_modules` for internal packages
4. Manages dependency hoisting

💡 **Pattern to Learn**: Workspace patterns support glob syntax. You could have:
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
  - 'tools/*'          # Build tools
  - 'examples/*'       # Example apps
```

---

#### turbo.json (Build Orchestration)

**Location**: `/turbo.json`

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build", "typecheck"],  // ← Build dependencies first
      "outputs": ["dist/**", ".next/**"]     // ← Cache these directories
    },
    "dev": {
      "cache": false,                        // ← Never cache dev
      "persistent": true                     // ← Keep running
    },
    "typecheck": {
      "dependsOn": ["^topo"]                 // ← Respect package order
    }
  }
}
```

🧠 **Understanding `dependsOn`**:
- **`"^build"`**: Run `build` in **dependencies** first (^ = upstream)
- **`"typecheck"`**: Run `typecheck` in **this** package
- **`"^topo"`**: Respect topological order (dependency graph)

**Example Flow**:
```
pnpm build
  ↓
Turbo analyzes dependency graph:
  ↓
1. Build packages/tsconfig (no dependencies)
2. Build packages/headless (depends on tsconfig)
3. Build apps/web (depends on headless)
  ↓
Cache outputs in .turbo/
```

💡 **Pattern to Learn**: Turborepo's power is **incremental builds**. If you change only the web app, it reuses cached headless package builds.

---

#### biome.json (Code Quality)

**Location**: `/biome.json`

**Purpose**: Configures **Biome** (linter + formatter, replaces ESLint + Prettier)

```json
{
  "formatter": {
    "indentStyle": "space",
    "indentWidth": 2
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  }
}
```

✅ **Current (Nov 2025)**: Uses Biome v1.9.4
⚠️ **Note**: Biome v2.3.6 available (adds Vue/Svelte support)

🎯 **Architectural Decision**: One tool (Biome) instead of two (ESLint + Prettier) reduces:
- Configuration complexity
- Node modules size (~50% smaller)
- Build time (Rust-based, much faster)

---

### Git Hooks & Changesets

```
.husky/                       ← Git hooks (pre-commit, pre-push)
  ├── pre-commit              ← Runs lint-staged before commit
  └── _/                      ← Husky internals

.changeset/                   ← Version management
  ├── config.json             ← Changeset configuration
  └── *.md                    ← Pending changesets (version bumps)
```

**Workflow**:
1. Developer makes changes
2. Runs `pnpm changeset` → Creates markdown file describing change
3. On release: `pnpm version:packages` → Bumps versions based on changesets
4. `pnpm publish:packages` → Publishes to npm

💡 **Pattern to Learn**: Changesets enable **independent versioning** in monorepos. Each package can have different versions.

---

## Deep Dive: packages/headless

This is the **core of Novel** - the library published to npm as `"novel"`.

### Complete Directory Structure

```
packages/headless/
├── src/                              ← Source code (what you edit)
│   ├── components/                   ← React components
│   │   ├── editor.tsx                ← EditorRoot, EditorContent
│   │   ├── editor-bubble.tsx         ← Selection menu
│   │   ├── editor-bubble-item.tsx    ← Bubble menu items
│   │   ├── editor-command.tsx        ← Slash command menu
│   │   ├── editor-command-item.tsx   ← Command menu items
│   │   └── index.ts                  ← Component exports
│   │
│   ├── extensions/                   ← Tiptap extensions
│   │   ├── ai-highlight.ts           ← AI text highlighting
│   │   ├── custom-keymap.ts          ← Custom keyboard shortcuts
│   │   ├── image-resizer.tsx         ← Image resize component
│   │   ├── mathematics.ts            ← LaTeX math support
│   │   ├── slash-command.tsx         ← Slash command system
│   │   ├── twitter.tsx               ← Tweet embeds
│   │   ├── updated-image.ts          ← Enhanced image extension
│   │   └── index.ts                  ← Extension exports
│   │
│   ├── plugins/                      ← ProseMirror plugins
│   │   ├── upload-images.tsx         ← Image upload handling
│   │   └── index.ts                  ← Plugin exports
│   │
│   ├── utils/                        ← Utilities
│   │   ├── atoms.ts                  ← Jotai state atoms
│   │   ├── store.ts                  ← Jotai store instance
│   │   ├── index.ts                  ← Utility function exports
│   │   └── (utility functions)
│   │
│   └── index.ts                      ← Main entry point (barrel export)
│
├── dist/                             ← Build output (generated)
│   ├── index.js                      ← ESM bundle
│   ├── index.cjs                     ← CommonJS bundle
│   ├── index.d.ts                    ← TypeScript declarations
│   └── (other generated files)
│
├── package.json                      ← Package metadata
├── tsconfig.json                     ← TypeScript configuration
├── tsup.config.ts                    ← Build configuration (tsup)
└── README.md                         ← Package documentation
```

---

### Architectural Pattern: Feature-Based Organization

🧠 **Pattern**: Code is organized by **feature type**, not by file type.

```
❌ BAD (file-type organization):
src/
  ├── components/       ← ALL components mixed
  ├── types/            ← ALL types mixed
  └── utils/            ← ALL utils mixed

✅ GOOD (feature-based):
src/
  ├── components/       ← UI components only
  ├── extensions/       ← Tiptap extensions only
  ├── plugins/          ← ProseMirror plugins only
  └── utils/            ← Cross-cutting utilities
```

**Why?**
- ✅ Easy to find related code
- ✅ Clear boundaries between concerns
- ✅ Each directory has a single responsibility
- ✅ Scales better as project grows

---

### src/components/ (React Layer)

**Purpose**: React components that consumers use directly.

#### Design Pattern: Composition over Configuration

Each component is **small** and **composable**:

```tsx
// ❌ BAD: Giant component with all features
<Editor
  withBubbleMenu
  withSlashCommands
  withAI
  theme="dark"
  // 50 more props...
/>

// ✅ GOOD: Composable components
<EditorRoot>
  <EditorContent />
  <EditorBubble>
    {/* Custom bubble menu items */}
  </EditorBubble>
  <EditorCommand>
    {/* Custom slash commands */}
  </EditorCommand>
</EditorRoot>
```

**Benefits**:
- ✅ Consumer controls what to include
- ✅ Tree-shaking removes unused code
- ✅ Easy to customize
- ✅ Clear component hierarchy

---

#### editor.tsx (The Foundation)

**Location**: `/packages/headless/src/components/editor.tsx`

**Exports**:
1. `EditorRoot` - Provider component
2. `EditorContent` - Main editor
3. `EditorProps` - TypeScript type
4. `EditorContentProps` - TypeScript type

**Let's dissect EditorRoot**:

```tsx
export const EditorRoot: FC<EditorRootProps> = ({ children }) => {
  const tunnelInstance = useRef(tunnel()).current;

  return (
    <Provider store={novelStore}>
      <EditorCommandTunnelContext.Provider value={tunnelInstance}>
        {children}
      </EditorCommandTunnelContext.Provider>
    </Provider>
  );
};
```

🧠 **What's happening here?**

1. **`tunnel()`** - Creates a "portal" for command menu
   - Why? Command menu needs to render OUTSIDE editor (for z-index, positioning)
   - Pattern: [tunnel-rat](https://github.com/pmndrs/tunnel-rat) library

2. **`useRef(...).current`** - Ensures tunnel instance is stable across renders
   - Why `.current`? So it doesn't change on re-renders
   - Pattern: **Stable instance pattern** (like `useRef` for DOM nodes)

3. **`Provider store={novelStore}`** - Jotai state provider
   - Why custom store? Each editor instance gets isolated state
   - Pattern: **Provider pattern** (like React Context)

4. **Double provider** - Jotai Provider + Context Provider
   - Why both? Jotai for state, Context for tunnel instance
   - Pattern: **Nested providers** (common in complex React apps)

💡 **Architectural Lesson**: When building reusable components:
- ✅ Isolate state (don't pollute global state)
- ✅ Use stable references for instances
- ✅ Support multiple instances on one page

---

#### editor-command.tsx (Advanced Patterns)

**Location**: `/packages/headless/src/components/editor-command.tsx`

**This file demonstrates ADVANCED React patterns:**

```tsx
// 1. Context for tunnel instance
export const EditorCommandTunnelContext = createContext<ReturnType<typeof tunnel> | null>(null);

// 2. Custom hook for accessing tunnel
export const useEditorCommandTunnel = () => {
  const tunnel = useContext(EditorCommandTunnelContext);
  if (!tunnel) throw new Error("useEditorCommandTunnel must be used within EditorRoot");
  return tunnel;
};

// 3. Portal rendering with tunnel-rat
export const EditorCommand = ({ children }: { children: ReactNode }) => {
  const tunnel = useEditorCommandTunnel();

  return (
    <>
      <tunnel.In>
        {/* Render command menu content */}
      </tunnel.In>
      <tunnel.Out />  {/* Actual DOM rendering happens here */}
    </>
  );
};
```

🧠 **Advanced Patterns Used**:

1. **Tunnel Pattern** (Portal alternative)
   - Why not `ReactDOM.createPortal`? Tunnel-rat handles SSR better
   - Use case: Render component in different DOM location

2. **Context + Custom Hook** pattern
   ```tsx
   createContext → useContext hook → Custom hook with error handling
   ```
   - Why custom hook? Provides better error messages
   - Pattern: **Safe context access**

3. **Throw on missing provider**
   - Catches developer errors early
   - Common in library development

💡 **Learn This Pattern**: When building reusable components that need context:
```tsx
// 1. Create context
const MyContext = createContext<T | null>(null);

// 2. Create safe hook
export const useMyContext = () => {
  const value = useContext(MyContext);
  if (!value) throw new Error("Must be used within Provider");
  return value;
};

// 3. Use in components
function MyComponent() {
  const value = useMyContext(); // Type-safe, error-handled
}
```

---

### src/extensions/ (The Power Layer)

**Purpose**: Tiptap extensions that add editor functionality.

🧠 **Mental Model**: Extensions are like **middleware** in Express or **plugins** in Babel.

Each extension can:
- ✅ Define new node/mark types (e.g., heading, bold)
- ✅ Add commands (e.g., `setHeading()`, `toggleBold()`)
- ✅ Add keyboard shortcuts (e.g., Cmd+B → bold)
- ✅ Add input rules (e.g., `##` → heading 2)
- ✅ Add paste rules (e.g., paste URL → link)
- ✅ Customize rendering

---

#### Extension Anatomy: ai-highlight.ts

**Location**: `/packages/headless/src/extensions/ai-highlight.ts`

```typescript
import { Mark } from "@tiptap/core";

export const AIHighlight = Mark.create({
  name: "ai-highlight",

  // 1. What HTML to render
  parseHTML() {
    return [{ tag: "mark", attrs: { "data-type": "ai-highlight" } }];
  },

  // 2. How to render it
  renderHTML({ HTMLAttributes }) {
    return ["mark", { ...HTMLAttributes, "data-type": "ai-highlight" }, 0];
  },

  // 3. What attributes it can have
  addAttributes() {
    return {
      color: {
        default: null,
        parseHTML: element => element.getAttribute('data-color'),
        renderHTML: attributes => ({ 'data-color': attributes.color }),
      },
    };
  },

  // 4. Commands it exposes
  addCommands() {
    return {
      setAIHighlight: () => ({ commands }) => {
        return commands.setMark(this.name);
      },
      unsetAIHighlight: () => ({ commands }) => {
        return commands.unsetMark(this.name);
      },
    };
  },

  // 5. Keyboard shortcuts
  addKeyboardShortcuts() {
    return {
      'Mod-Shift-h': () => this.editor.commands.toggleMark(this.name),
    };
  },
});
```

🧠 **Understanding Each Method**:

**`parseHTML()`** - How to recognize this mark in HTML
- When pasting or loading HTML, Tiptap looks for `<mark data-type="ai-highlight">`
- Pattern: **HTML→Editor mapping**

**`renderHTML()`** - How to convert back to HTML
- The `0` at the end means "render children here"
- Pattern: **Editor→HTML mapping**

**`addAttributes()`** - What data this mark can store
- `default: null` - Attribute is optional
- `parseHTML/renderHTML` - How to read/write HTML attributes
- Pattern: **Attribute schema definition**

**`addCommands()`** - Programmatic API
- `setAIHighlight()` - Apply this mark
- Returns command function that Tiptap executes
- Pattern: **Command pattern** (like Redux actions)

**`addKeyboardShortcuts()`** - Keyboard bindings
- `Mod` = Cmd on Mac, Ctrl on Windows/Linux
- Returns boolean (did it handle the key?)
- Pattern: **Event handler pattern**

💡 **Learn This**: Extension structure is **declarative**. You describe WHAT, Tiptap handles HOW.

---

#### Advanced Extension: slash-command.tsx

**Location**: `/packages/headless/src/extensions/slash-command.tsx`

This extension uses the **Suggestion plugin** pattern:

```typescript
const Command = Extension.create({
  name: "slash-command",

  addOptions() {
    return {
      suggestion: {
        char: "/",                    // ← Trigger character
        command: ({ editor, range, props }) => {
          props.command({ editor, range });
        },
      } as SuggestionOptions,
    };
  },

  addProseMirrorPlugins() {
    return [
      Suggestion({
        editor: this.editor,
        ...this.options.suggestion,
      }),
    ];
  },
});
```

🧠 **What's Happening**:

1. **Suggestion Plugin** - Tiptap's built-in autocomplete system
   - Watches for trigger character (`/`)
   - Calls callbacks when user types
   - Handles position tracking

2. **`addProseMirrorPlugins()`** - Low-level plugin integration
   - Returns array of ProseMirror plugins
   - Advanced: Direct access to ProseMirror layer
   - Pattern: **Plugin composition**

3. **Rendering with React** (in `renderItems` function):
   ```typescript
   const renderItems = () => {
     let component: ReactRenderer | null = null;
     let popup: Instance<Props>[] | null = null;

     return {
       onStart: (props) => {
         // Create React component
         component = new ReactRenderer(EditorCommandOut, {
           props,
           editor: props.editor,
         });

         // Create Tippy.js popup
         popup = tippy("body", {
           content: component.element,
           // ... popup options
         });
       },
       onUpdate: (props) => {
         component?.updateProps(props);
       },
       onExit: () => {
         popup?.[0]?.destroy();
         component?.destroy();
       },
     };
   };
   ```

🧠 **Advanced Pattern: React in ProseMirror**:
- ProseMirror is vanilla JS (no React)
- `ReactRenderer` bridges the gap
- Creates React component, mounts to DOM, syncs props

💡 **Architectural Lesson**: When integrating React with non-React libraries:
```typescript
// 1. Create renderer
const renderer = new ReactRenderer(MyComponent, { props });

// 2. Use renderer.element as DOM node
popup.setContent(renderer.element);

// 3. Update props as needed
renderer.updateProps(newProps);

// 4. Clean up when done
renderer.destroy();
```

---

### src/plugins/ (ProseMirror Layer)

**Purpose**: Low-level ProseMirror plugins for advanced features.

**Difference from Extensions**:
- Extensions: High-level Tiptap API
- Plugins: Low-level ProseMirror API

---

#### upload-images.tsx (Advanced Plugin)

**Location**: `/packages/headless/src/plugins/upload-images.tsx`

This demonstrates **ProseMirror decorations** and **plugin state**:

```typescript
export const UploadImagesPlugin = () => {
  return new Plugin({
    state: {
      init() {
        return DecorationSet.empty;
      },
      apply(tr, set) {
        // Update decorations based on transaction
        set = set.map(tr.mapping, tr.doc);

        // Add new decorations for uploading images
        const action = tr.getMeta(this);
        if (action?.add) {
          const decoration = Decoration.widget(action.add.pos, () => {
            // Create placeholder DOM node
            const placeholder = document.createElement("div");
            placeholder.className = "img-placeholder";
            return placeholder;
          });
          set = set.add(tr.doc, [decoration]);
        }

        return set;
      },
    },

    props: {
      decorations(state) {
        return this.getState(state);
      },
    },
  });
};
```

🧠 **Understanding Decorations**:

**Decorations** = Visual overlays on the document (don't modify actual content)

Use cases:
- ✅ Upload placeholders (like above)
- ✅ Collaboration cursors
- ✅ Spell-check underlines
- ✅ Search highlights

**Pattern**:
1. **Plugin state** tracks decorations
2. **`apply()`** updates on every transaction
3. **`props.decorations`** renders them

💡 **Advanced Pattern**: Use decorations for **ephemeral UI** that shouldn't be in document content.

---

### src/utils/ (The Toolbox)

#### atoms.ts (State Definitions)

**Location**: `/packages/headless/src/utils/atoms.ts`

```typescript
import { atom } from "jotai";

export const queryAtom = atom<string>("");
export const rangeAtom = atom<Range | null>(null);
```

🧠 **Jotai Atoms Explained**:

An atom is a **piece of state** that:
- Can be read by multiple components
- Can be written by multiple components
- Automatically triggers re-renders on change
- Is **independent** (not part of big state tree)

**Usage**:
```tsx
import { useAtom } from "jotai";
import { queryAtom } from "./atoms";

function MyComponent() {
  const [query, setQuery] = useAtom(queryAtom);
  // Just like useState, but global!
}
```

💡 **Pattern**: Define atoms in separate file, import where needed.

---

#### store.ts (State Isolation)

**Location**: `/packages/headless/src/utils/store.ts`

```typescript
import { createStore } from "jotai";

export const novelStore = createStore();
```

🧠 **Why Custom Store**?

**Default Jotai** = One global store (like Redux)
**Custom Store** = Isolated state per editor instance

This allows:
```tsx
// Two independent editors on same page
<EditorRoot>  {/* Store instance #1 */}
  <EditorContent />
</EditorRoot>

<EditorRoot>  {/* Store instance #2 */}
  <EditorContent />
</EditorRoot>
```

💡 **Architectural Lesson**: For reusable components, **always isolate state** to support multiple instances.

---

### index.ts (The Public API)

**Location**: `/packages/headless/src/index.ts`

```typescript
// Components
export {
  EditorRoot,
  EditorContent,
  EditorBubble,
  EditorCommand,
  // ...
} from "./components";

// Extensions
export {
  AIHighlight,
  SlashCommand,
  // ...
} from "./extensions";

// Plugins
export { UploadImagesPlugin, createImageUpload } from "./plugins";

// Utils
export { isValidUrl, getUrlFromString } from "./utils";

// State
export { queryAtom, rangeAtom, novelStore } from "./utils";

// Types
export type { EditorContentProps, SuggestionItem } from "./components";
```

🧠 **Barrel Export Pattern**:
- All exports in ONE file
- Consumers import from `"novel"`, not `"novel/components/editor"`
- Clean public API

**Benefits**:
- ✅ Consumers don't need to know internal structure
- ✅ Easy to refactor internals without breaking API
- ✅ Clear separation: public vs private

💡 **Learn This**: For libraries, use barrel exports in `index.ts`:
```typescript
// ❌ Consumer code without barrel:
import { EditorRoot } from "novel/dist/components/editor";
import { AIHighlight } from "novel/dist/extensions/ai-highlight";

// ✅ Consumer code with barrel:
import { EditorRoot, AIHighlight } from "novel";
```

---

### package.json (Library Metadata)

**Location**: `/packages/headless/package.json`

```json
{
  "name": "novel",
  "version": "1.0.0",
  "main": "dist/index.cjs",           // ← CommonJS entry (require)
  "module": "dist/index.js",          // ← ESM entry (import)
  "types": "dist/index.d.ts",         // ← TypeScript types
  "files": ["dist"],                  // ← Only ship dist/ to npm
  "sideEffects": false,               // ← Enable tree-shaking

  "peerDependencies": {
    "react": ">=18"                   // ← Require React 18+
  },

  "dependencies": {
    "@tiptap/core": "^2.11.2",        // ← Bundled dependencies
    // ...
  }
}
```

🧠 **Key Fields Explained**:

**`main` vs `module`**:
- `main`: CommonJS (`require()`) - Node.js, older bundlers
- `module`: ES Modules (`import`) - Modern bundlers (Webpack, Vite)
- Bundlers pick the right one automatically

**`sideEffects: false`**:
- Tells bundlers: "This package has no side effects"
- Enables **tree-shaking** (remove unused code)
- Example: Import only `EditorRoot`? Bundle excludes `SlashCommand` code

**`peerDependencies`**:
- "I need React, but I won't bundle it"
- Avoids having TWO React copies in consumer's bundle
- Consumer must install React separately

💡 **Pattern**: For React libraries, **always use peerDependencies** for React.

---

## Deep Dive: apps/web

The demo Next.js application showcasing the editor.

### Structure Philosophy

```
apps/web/
├── app/                      ← Next.js 15 App Router
│   ├── layout.tsx            ← Root layout (global)
│   ├── page.tsx              ← Home page
│   ├── providers.tsx         ← Client-side providers
│   └── api/                  ← API routes (Edge functions)
│       ├── generate/         ← AI completion endpoint
│       └── upload/           ← Image upload endpoint
│
├── components/               ← React components
│   ├── tailwind/             ← Demo editor implementation
│   │   ├── advanced-editor.tsx
│   │   ├── extensions.ts     ← Extension configuration
│   │   ├── slash-command.tsx ← Slash command items
│   │   ├── selectors/        ← Bubble menu components
│   │   └── generative/       ← AI features UI
│   └── ui/                   ← shadcn/ui components
│
├── lib/                      ← Utilities
│   ├── content.ts            ← Default editor content
│   └── utils.ts              ← Helper functions
│
├── styles/                   ← Global styles
│   ├── globals.css           ← Tailwind + global CSS
│   └── prosemirror.css       ← ProseMirror-specific styles
│
├── public/                   ← Static assets
│
├── .env.example              ← Environment variables template
├── next.config.js            ← Next.js configuration
├── tailwind.config.ts        ← Tailwind configuration
└── package.json              ← App dependencies
```

---

### Next.js 15 App Router Architecture

✅ **Current (Nov 2025)**: Uses Next.js 15.1.4 (App Router)
⚠️ **Note**: Next.js 16 is available (see TECH_STACK_RESEARCH.md)

**App Router Conventions**:
- `layout.tsx` = Persistent layout wrapper
- `page.tsx` = Route page component
- `route.ts` = API endpoint
- `loading.tsx` = Loading UI (Suspense boundary)
- `error.tsx` = Error boundary

---

#### app/layout.tsx (The Shell)

**Purpose**: Root layout, wraps entire app.

```tsx
export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={font.className}>
        <Providers>
          {children}
          <Analytics />
        </Providers>
        <Toaster />
      </body>
    </html>
  );
}
```

🧠 **Key Elements**:

**`suppressHydrationWarning`**:
- Needed for theme provider (client-side theme differs from SSR)
- Prevents React hydration mismatch errors

**`<Providers>`**:
- Wraps all client-side context providers
- Must be separate component (client component boundary)

**`<Analytics />`**:
- Vercel Analytics component
- Tracks page views, Web Vitals

💡 **Pattern**: Keep layout minimal, delegate to providers.

---

#### app/providers.tsx (Client Boundary)

```tsx
'use client';  // ← CRITICAL: Client component

export function Providers({ children }: { children: ReactNode }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      <AppContextProvider>
        {children}
        <Toaster />
      </AppContextProvider>
    </ThemeProvider>
  );
}
```

🧠 **Understanding `'use client'`**:

In Next.js 15 App Router:
- **Default**: Server Components (render on server)
- **`'use client'`**: Client Components (render on client)

**Why separate providers file?**
- `layout.tsx` can stay Server Component
- Only providers boundary is client
- Better performance (less JavaScript shipped)

💡 **Pattern**: Create `providers.tsx` for all client-side context.

---

#### app/api/ (Edge API Routes)

**Purpose**: Serverless API endpoints running on Vercel Edge Network.

##### app/api/generate/route.ts (AI Completion)

```typescript
export const runtime = 'edge';  // ← Run on Edge (not Node.js)

export async function POST(req: Request) {
  // Rate limiting
  const ip = req.headers.get('x-forwarded-for');
  const rateLimitResult = await ratelimit.limit(ip);

  if (!rateLimitResult.success) {
    return new Response('Rate limit exceeded', { status: 429 });
  }

  // AI completion
  const { prompt, option } = await req.json();

  const completion = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [{ role: 'user', content: prompt }],
    stream: true,
  });

  // Stream response
  return new Response(completion.toReadableStream());
}
```

🧠 **Edge Runtime vs Node.js Runtime**:

| Edge Runtime | Node.js Runtime |
|--------------|----------------|
| ✅ Fast cold starts (<50ms) | ⚠️ Slower cold starts (~1s) |
| ✅ Global deployment | ⚠️ Regional deployment |
| ⚠️ Limited APIs (no fs, crypto) | ✅ Full Node.js APIs |
| ✅ Lower cost | ⚠️ Higher cost |

**When to use Edge**:
- ✅ API routes with external API calls
- ✅ Middleware
- ✅ Serverless functions needing low latency

**When to use Node.js**:
- ✅ Need filesystem access
- ✅ Database connections
- ✅ Complex crypto operations

💡 **Pattern**: Use Edge for API proxy routes (like AI completions).

---

### components/tailwind/ (Demo Implementation)

This directory shows HOW to use the headless library.

#### advanced-editor.tsx (Full Example)

**Purpose**: Complete editor implementation with all features.

```tsx
export default function Editor() {
  const [content, setContent] = useLocalStorage('novel-content', initialContent);
  const [saveStatus, setSaveStatus] = useState('Saved');
  const debouncedUpdates = useDebouncedCallback(async (editor) => {
    const json = editor.getJSON();
    setContent(json);
    setSaveStatus('Saved');
  }, 500);

  return (
    <EditorRoot>
      <EditorContent
        extensions={defaultExtensions}
        editorProps={{
          handleDOMEvents: {
            keydown: (_view, event) => handleCommandNavigation(event),
          },
        }}
        onUpdate={({ editor }) => {
          debouncedUpdates(editor);
          setSaveStatus('Unsaved');
        }}
        slotAfter={<ImageResizer />}
      >
        <EditorCommand>
          {/* Slash commands */}
        </EditorCommand>

        <EditorBubble>
          {/* Text formatting */}
        </EditorBubble>
      </EditorContent>
    </EditorRoot>
  );
}
```

🧠 **Patterns Demonstrated**:

**1. Local Storage Persistence**:
```tsx
const [content, setContent] = useLocalStorage('key', initialValue);
```
- Custom hook for persistent state
- Automatically syncs with localStorage

**2. Debounced Save**:
```tsx
const debouncedUpdates = useDebouncedCallback(async (editor) => {
  // Save logic
}, 500);
```
- Wait 500ms after last keystroke before saving
- Prevents excessive saves while typing
- Pattern: **Debounce** (common in autosave features)

**3. Status Tracking**:
```tsx
onUpdate={() => setSaveStatus('Unsaved')}
// After save completes
setSaveStatus('Saved')
```
- Visual feedback for users
- Pattern: **Optimistic UI updates**

💡 **Learn This**: For autosave features:
```tsx
// 1. Debounce saves
const debouncedSave = useDebouncedCallback(save, 500);

// 2. Track status
const [status, setStatus] = useState('Saved');

// 3. Update on change
onUpdate={() => {
  setStatus('Saving...');
  debouncedSave();
});

// 4. Update after save
save.then(() => setStatus('Saved'));
```

---

#### extensions.ts (Configuration Example)

**Purpose**: Shows how to configure and theme extensions.

```typescript
export const defaultExtensions = [
  StarterKit.configure({
    heading: {
      levels: [1, 2, 3],
      HTMLAttributes: {
        class: 'scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl',
      },
    },
    code: {
      HTMLAttributes: {
        class: 'rounded-md bg-muted px-1.5 py-1 font-mono font-medium',
      },
    },
  }),

  TiptapImage.extend({
    addProseMirrorPlugins() {
      return [
        UploadImagesPlugin({
          imageClass: 'opacity-40 rounded-lg border border-stone-200',
        }),
      ];
    },
  }).configure({
    allowBase64: true,
    HTMLAttributes: {
      class: 'rounded-lg border border-muted',
    },
  }),

  Placeholder.configure({
    placeholder: ({ node }) => {
      if (node.type.name === 'heading') {
        return `Heading ${node.attrs.level}`;
      }
      return "Press '/' for commands...";
    },
  }),
];
```

🧠 **Configuration Patterns**:

**1. Configure Method**:
```typescript
Extension.configure({ option: value })
```
- Customizes extension behavior
- Sets default attributes

**2. Extend Method**:
```typescript
Extension.extend({
  addCommands() { /* new commands */ },
  addKeyboardShortcuts() { /* new shortcuts */ },
})
```
- Adds new functionality to existing extension
- Pattern: **Extension inheritance**

**3. Dynamic Placeholders**:
```typescript
placeholder: ({ node }) => {
  // Different placeholder based on node type
  if (node.type.name === 'heading') return 'Heading';
  return 'Type something';
}
```
- Context-aware UI hints
- Pattern: **Dynamic content**

💡 **Architectural Lesson**: Extensions are **highly composable**. You can:
- Configure built-in extensions
- Extend them with new features
- Combine multiple extensions
- Create custom extensions

---

## File Naming Conventions

### Components

```
✅ GOOD:
  editor-root.tsx
  editor-command.tsx
  slash-command.tsx

❌ BAD:
  EditorRoot.tsx       (PascalCase files)
  editor_root.tsx      (snake_case)
  editorRoot.tsx       (camelCase)
```

**Pattern**: `kebab-case.tsx` for files, `PascalCase` for exported component.

**Why?**
- ✅ Consistent, predictable
- ✅ Works on all file systems (case-insensitive)
- ✅ Matches URL conventions

---

### Extensions & Plugins

```
✅ GOOD:
  ai-highlight.ts
  custom-keymap.ts
  upload-images.tsx

❌ BAD:
  AIHighlight.ts
  customKeymap.ts
```

**Pattern**: `kebab-case.ts` for files, `PascalCase` for exported extension.

---

### Utils & Helpers

```
✅ GOOD:
  utils/index.ts       (barrel export)
  utils/atoms.ts       (state atoms)
  utils/store.ts       (store instance)

❌ BAD:
  utils/utilityFunctions.ts    (redundant "utility")
  utils/helpers.ts             (vague name)
```

**Pattern**: Descriptive, single-purpose files.

---

## Import/Export Patterns

### Barrel Exports (index.ts)

**Every directory has an index.ts that exports public API**:

```typescript
// src/components/index.ts
export { EditorRoot, EditorContent } from './editor';
export { EditorBubble } from './editor-bubble';
export { EditorCommand } from './editor-command';

// Consumers import from directory
import { EditorRoot } from 'novel/components';
```

**Benefits**:
- ✅ Clean import paths
- ✅ Control public API (can hide internal files)
- ✅ Easy refactoring (change internal structure, keep exports same)

---

### Named Exports Over Default

```typescript
// ✅ GOOD: Named export
export const EditorRoot = () => { ... };

// ❌ BAD: Default export
export default function EditorRoot() { ... }
```

**Why named exports?**
- ✅ Better IDE autocomplete
- ✅ Easier to refactor (find all references)
- ✅ Consistent naming (can't accidentally rename on import)
- ✅ Allows multiple exports per file

💡 **Exception**: Next.js page components must use default exports (framework requirement).

---

### Type-Only Imports

```typescript
// ✅ GOOD: Type-only import
import type { JSONContent } from '@tiptap/react';

// ❌ BAD: Regular import (bundles type in runtime)
import { JSONContent } from '@tiptap/react';
```

**Why `import type`?**
- ✅ Removed at compile time (no runtime cost)
- ✅ Makes it clear this is type-only
- ✅ Prevents accidental runtime usage

---

## Architectural Patterns in File Organization

### 1. Co-location Pattern

**Related code lives together**:

```
components/
  editor-command.tsx         ← Component
  editor-command-item.tsx    ← Related component

extensions/
  slash-command.tsx          ← Extension

// Both relate to "command", but organized by TYPE not FEATURE
```

**Why?**
- ✅ Components are React layer (UI)
- ✅ Extensions are Tiptap layer (logic)
- ✅ Clear separation of concerns

---

### 2. Shared Kernel Pattern

**utils/ is the "shared kernel"**:

```
utils/
  index.ts         ← Pure functions, no dependencies
  atoms.ts         ← State definitions
  store.ts         ← Store instance
```

**Rules**:
- ✅ No dependencies on components/extensions
- ✅ Can be used anywhere
- ✅ Pure, tested functions

---

### 3. Dependency Direction

```
Your App
  ↓ depends on
EditorContent (component)
  ↓ depends on
TiptapProvider (library)
  ↓ depends on
Extensions
  ↓ depends on
Utils

❌ NEVER:
  Utils depends on Extensions
  Extensions depend on Components
```

**Rule**: Dependencies flow **downward** (top depends on bottom, never reverse).

---

## Build Output Structure

### Headless Package Output

```
packages/headless/dist/
├── index.js              ← ESM bundle (import)
├── index.cjs             ← CommonJS bundle (require)
├── index.d.ts            ← TypeScript declarations
├── components/           ← Individual component files
│   ├── editor.js
│   ├── editor.d.ts
│   └── ...
├── extensions/
└── ...
```

**Generated by**: tsup (TypeScript bundler)

**Build command**: `pnpm build` → `tsup`

**Configuration**: `tsup.config.ts`

```typescript
export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'],           // ← Both formats
  dts: true,                        // ← Generate .d.ts
  external: ['react', 'react-dom'], // ← Don't bundle React
  banner: {
    js: "'use client';",            // ← Add directive for Next.js
  },
});
```

---

### Web App Output

```
apps/web/.next/
├── cache/              ← Next.js build cache
├── server/             ← Server-side code
│   └── app/            ← Route handlers
├── static/             ← Static assets
│   ├── chunks/         ← JavaScript chunks
│   └── css/            ← CSS files
└── standalone/         ← Standalone deployment (optional)
```

**Generated by**: Next.js build system

---

## Common Pitfalls & Best Practices

### ❌ Pitfall 1: Importing from Dist

```typescript
// ❌ BAD: Importing from build output
import { EditorRoot } from 'novel/dist/components/editor';

// ✅ GOOD: Import from index
import { EditorRoot } from 'novel';
```

**Why?** Dist structure can change, public API (index) stays stable.

---

### ❌ Pitfall 2: Circular Dependencies

```typescript
// editor.tsx
import { someUtil } from '../utils/editor-utils';

// editor-utils.ts
import { EditorRoot } from '../components/editor';  // ❌ CIRCULAR!
```

**Solution**: Move shared code to separate file.

---

### ❌ Pitfall 3: Importing Component Styles

```typescript
// ❌ BAD: Importing CSS in library
import './editor.css';
```

**Why?** Headless library shouldn't ship styles.

**Solution**: Document CSS classes, let consumers style.

---

### ✅ Best Practice 1: Keep Public API Minimal

```typescript
// ❌ BAD: Export everything
export * from './components';
export * from './utils';
export * from './internal';

// ✅ GOOD: Export only public API
export { EditorRoot, EditorContent } from './components';
export { AIHighlight } from './extensions';
```

---

### ✅ Best Practice 2: Use Path Aliases

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}

// Now you can:
import { EditorRoot } from '@/components/editor';
// Instead of:
import { EditorRoot } from '../../../components/editor';
```

---

### ✅ Best Practice 3: Type-Safe Barrel Exports

```typescript
// index.ts
export { EditorRoot } from './editor';
export type { EditorContentProps } from './editor';

// Consumers get:
import { EditorRoot } from 'novel';              // Component
import type { EditorContentProps } from 'novel'; // Type
```

---

## Summary: What You've Learned

### Key Architectural Patterns

1. **Monorepo Structure** - Packages + Apps in one repo
2. **Feature-Based Organization** - Group by feature type
3. **Headless Pattern** - Logic without styles
4. **Barrel Exports** - Clean public API via index.ts
5. **Dependency Direction** - Always downward
6. **Named Exports** - Better than default exports
7. **Co-location** - Related code lives together
8. **Shared Kernel** - Utils as foundation

### Directory Mental Models

```
packages/headless/src/
  ├── components/     → React layer (UI primitives)
  ├── extensions/     → Tiptap layer (editor features)
  ├── plugins/        → ProseMirror layer (low-level)
  └── utils/          → Foundation (pure functions)

apps/web/
  ├── app/            → Next.js routes
  ├── components/     → Demo components (how to use library)
  └── styles/         → Global styles
```

### Advanced Patterns Learned

- ✅ Tunnel pattern (portal alternative)
- ✅ Stable instance pattern (useRef)
- ✅ Provider isolation pattern
- ✅ Extension composition
- ✅ Plugin decorations
- ✅ Edge runtime API routes
- ✅ Debounced autosave
- ✅ Tree-shakeable exports

---

## Next Steps

Now you understand the structure. Time to explore code:

1. **[Code Tours](./CODE_TOURS.md)** - Follow features through the codebase
2. **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** - Deep dive into each technology
3. **[Editor Core Guide](./EDITOR_CORE_GUIDE.md)** - Master Tiptap extensions
4. **[How-To Guide](./HOW_TO_GUIDE.md)** - Build features yourself

---

**You're now a Novel architect!** 🎉

You understand not just WHERE code lives, but WHY it's organized this way. Use these patterns in your own projects.

---

**Last Updated:** November 19, 2025
**Reading Time:** 60+ minutes
**Status:** ✅ Comprehensive architectural guide
