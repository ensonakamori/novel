# Technology Stack Guide: Becoming a Full-Stack Architect

**Last Updated:** November 19, 2025
**Tech Stack Research:** [View Version Analysis](./TECH_STACK_RESEARCH.md)
**Target Audience:** Mid-level developers → Full-stack architects
**Reading Time:** 90-120 minutes

This is your **comprehensive guide** to every technology in Novel. Not just "what" each technology is, but **why** it was chosen, **how** it works architecturally, and **what patterns** you should master.

---

## Table of Contents

1. [Stack Overview](#stack-overview)
2. [Tiptap: The Editor Engine](#tiptap-the-editor-engine)
3. [ProseMirror: The Foundation](#prosemirror-the-foundation)
4. [React 18: The UI Layer](#react-18-the-ui-layer)
5. [Next.js 15: The Framework](#nextjs-15-the-framework)
6. [Jotai: Atomic State Management](#jotai-atomic-state-management)
7. [TypeScript: Type Safety](#typescript-type-safety)
8. [Tailwind CSS: Utility-First Styling](#tailwind-css-utility-first-styling)
9. [Vercel AI SDK: AI Integration](#vercel-ai-sdk-ai-integration)
10. [Turborepo: Monorepo Orchestration](#turborepo-monorepo-orchestration)
11. [pnpm: Package Management](#pnpm-package-management)
12. [Biome: Code Quality](#biome-code-quality)
13. [Supporting Libraries](#supporting-libraries)
14. [Technology Comparison Matrix](#technology-comparison-matrix)
15. [What to Learn Next](#what-to-learn-next)

---

## Stack Overview

### The Full Stack

```
┌─────────────────────────────────────────────┐
│  Developer Experience Layer                 │
│  ├─ Biome (Linter/Formatter)               │
│  ├─ TypeScript (Type Safety)               │
│  └─ Turborepo (Build Orchestration)        │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Application Layer                          │
│  ├─ Next.js 15 (Framework)                 │
│  ├─ React 18 (UI Library)                  │
│  └─ Tailwind CSS (Styling)                 │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Editor Layer                               │
│  ├─ Tiptap (React Wrapper)                 │
│  └─ ProseMirror (Engine)                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  State & Integration Layer                  │
│  ├─ Jotai (State Management)               │
│  ├─ Vercel AI SDK (AI Features)            │
│  └─ Supporting Libraries                    │
└─────────────────────────────────────────────┘
```

### Version Status (November 2025)

| Technology | Project Version | Latest Version | Status |
|------------|----------------|----------------|--------|
| **Tiptap** | 2.11.2 | v3.x / v2.9+ | ✅ Current for v2.x |
| **React** | 18.2.0 | 19.2.0 | 🚨 One major behind |
| **Next.js** | 15.1.4 | 16.0 | ⚠️ One major behind |
| **TypeScript** | 5.7.3 | 5.9 | ⚠️ Two minor behind |
| **Jotai** | 2.11.0 | 2.12.2 | ✅ Very recent |
| **Tailwind** | 3.4.1 | 4.1 | 🚨 Major rewrite available |
| **AI SDK** | 3.0.12 | 5.x | 🚨 Two majors behind |
| **Turborepo** | 2.3.3 | 2.6.1 | ⚠️ Three minor behind |
| **pnpm** | 9.5.0 | 10.22.0 | 🚨 One major behind |
| **Biome** | 1.9.4 | 2.3.6 | 🚨 One major behind |

**Legend:**
- ✅ Current - Latest stable or very recent
- ⚠️ Slightly behind - Minor versions, non-breaking
- 🚨 Major behind - Consider upgrading, but current version works

---

## Tiptap: The Editor Engine

### What is Tiptap?

**Tiptap** is a **headless, framework-agnostic, rich-text editor** built on top of ProseMirror.

🧠 **Mental Model**: Tiptap is to ProseMirror what Next.js is to React:
- ProseMirror = Low-level, powerful, complex
- Tiptap = High-level, developer-friendly wrapper

### Why Tiptap?

**Alternatives Considered:**
- **Draft.js** (Facebook) - Simpler but less flexible
- **Slate** (similar approach) - More low-level
- **Quill** - Opinionated, not as extensible
- **TinyMCE/CKEditor** - Heavy, commercial

**Why Tiptap Won:**
1. ✅ Built on battle-tested ProseMirror
2. ✅ Extension-based architecture (infinitely extensible)
3. ✅ Framework-agnostic (React, Vue, Vanilla JS)
4. ✅ TypeScript-first
5. ✅ Active community & great docs
6. ✅ Headless pattern (bring your own UI)

### Core Architecture

```typescript
// Tiptap's layered approach
Editor
  ├── Extensions (Features)
  │   ├── Nodes (Block elements: heading, paragraph)
  │   ├── Marks (Inline formatting: bold, italic)
  │   └── Extensions (Behaviors: history, collaboration)
  │
  ├── Schema (Document structure)
  │
  ├── State (Current document + selection)
  │
  └── View (DOM rendering)
```

### Extension System Deep Dive

**Every feature is an extension:**

```typescript
import { Node } from '@tiptap/core'

export const CustomNode = Node.create({
  name: 'customNode',

  // 1. Schema: What is this node?
  content: 'inline*',              // Can contain inline elements
  group: 'block',                  // It's a block-level node
  defining: true,                  // Defines a new context

  // 2. Attributes: What data does it store?
  addAttributes() {
    return {
      id: {
        default: null,
      },
      customData: {
        default: {},
        parseHTML: element => JSON.parse(element.getAttribute('data-custom')),
        renderHTML: attributes => ({
          'data-custom': JSON.stringify(attributes.customData),
        }),
      },
    }
  },

  // 3. Parsing: HTML → Editor
  parseHTML() {
    return [
      {
        tag: 'div[data-type="custom-node"]',
        priority: 51,              // Higher than default (50)
      },
    ]
  },

  // 4. Rendering: Editor → HTML
  renderHTML({ node, HTMLAttributes }) {
    return ['div', { ...HTMLAttributes, 'data-type': 'custom-node' }, 0]
    //                                                                  ↑
    //                                         0 = render children here
  },

  // 5. Commands: Programmatic API
  addCommands() {
    return {
      setCustomNode: attributes => ({ commands }) => {
        return commands.setNode(this.name, attributes)
      },
      toggleCustomNode: () => ({ commands }) => {
        return commands.toggleNode(this.name, 'paragraph')
      },
    }
  },

  // 6. Keyboard Shortcuts
  addKeyboardShortcuts() {
    return {
      'Mod-Shift-c': () => this.editor.commands.toggleCustomNode(),
    }
  },

  // 7. Input Rules: Auto-formatting
  addInputRules() {
    return [
      nodeInputRule({
        find: /^\$\$(.+)\$\$$/,    // Regex pattern
        type: this.type,
        getAttributes: match => ({ content: match[1] }),
      }),
    ]
  },

  // 8. Paste Rules: Smart paste
  addPasteRules() {
    return [
      nodePasteRule({
        find: /https?:\/\/example\.com\/(\w+)/g,
        type: this.type,
        getAttributes: match => ({ id: match[1] }),
      }),
    ]
  },

  // 9. NodeView: Custom rendering (React component)
  addNodeView() {
    return ReactNodeViewRenderer(CustomNodeComponent)
  },
})
```

🎯 **Pattern to Master**: Extension Lifecycle
1. **Schema** defines structure
2. **Parse/Render** handles HTML conversion
3. **Commands** provide programmatic API
4. **Keyboard shortcuts** for UX
5. **Input/Paste rules** for smart formatting
6. **NodeView** for custom React rendering

### Real Example: How AIHighlight Works

**Location**: `/packages/headless/src/extensions/ai-highlight.ts`

```typescript
export const AIHighlight = Mark.create({
  name: "ai-highlight",

  // Mark (not Node) - inline formatting like bold/italic

  parseHTML() {
    return [
      {
        tag: 'mark[data-type="ai-highlight"]',
        // Matches: <mark data-type="ai-highlight">text</mark>
      },
    ]
  },

  renderHTML({ HTMLAttributes }) {
    return [
      'mark',
      { ...HTMLAttributes, 'data-type': 'ai-highlight' },
      0,  // Children rendered here
    ]
  },

  addCommands() {
    return {
      setAIHighlight: () => ({ commands }) => {
        return commands.setMark(this.name)
      },
      toggleAIHighlight: () => ({ commands }) => {
        return commands.toggleMark(this.name)
      },
      unsetAIHighlight: () => ({ commands }) => {
        return commands.unsetMark(this.name)
      },
    }
  },

  addKeyboardShortcuts() {
    return {
      'Mod-Shift-h': () => this.editor.commands.toggleAIHighlight(),
    }
  },
})
```

**Usage in Code:**

```typescript
// Apply AI highlight to selection
editor.commands.setAIHighlight()

// Or programmatically
editor
  .chain()
  .focus()
  .insertContent('AI generated text')
  .setAIHighlight()
  .run()

// Chain multiple commands!
```

🧠 **Architectural Lesson**: Tiptap's **command chaining** pattern:

```typescript
editor
  .chain()              // Start chain
  .focus()              // Command 1
  .setHeading({ level: 1 })  // Command 2
  .insertContent('Hello')    // Command 3
  .run()                // Execute all
```

**Why chain?**
- ✅ Single transaction (one undo step)
- ✅ Atomic (all or nothing)
- ✅ More efficient (one render)

### Tiptap vs ProseMirror

| Feature | Tiptap | ProseMirror |
|---------|--------|-------------|
| **API Level** | High-level | Low-level |
| **Learning Curve** | Moderate | Steep |
| **React Integration** | Built-in | Manual |
| **Extension System** | Declarative | Imperative |
| **TypeScript** | First-class | Good support |
| **Documentation** | Excellent | Technical |
| **Use Case** | Most editors | Full control needed |

💡 **When to use ProseMirror directly?**
- Building a very custom editor
- Need maximum performance optimization
- Implementing novel editing paradigms

**For Novel**: Tiptap is perfect - extensible enough, simpler than raw ProseMirror.

### Tiptap Resources

- 📚 **Official Docs**: https://tiptap.dev/docs
- 🎓 **Extension Guide**: https://tiptap.dev/guide/custom-extensions
- 💬 **Community**: https://github.com/ueberdosis/tiptap/discussions
- 📦 **Extension List**: https://tiptap.dev/extensions

**Current Project Version**: v2.11.2
- ✅ Stable and production-ready
- ⚠️ v3.x available (major rewrite with breaking changes)
- 📖 See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for upgrade path

---

## ProseMirror: The Foundation

### What is ProseMirror?

**ProseMirror** is a **low-level editor framework** that powers Tiptap (and many other editors).

🧠 **Mental Model**: If Tiptap is React, ProseMirror is the DOM.

### Why Learn ProseMirror?

Even though you use Tiptap, understanding ProseMirror helps you:
1. ✅ Debug complex editor issues
2. ✅ Write advanced plugins
3. ✅ Understand Tiptap's architecture
4. ✅ Build custom behaviors

### Core Concepts

#### 1. Document Model

ProseMirror uses a **tree structure** (not HTML string):

```javascript
// Not this (HTML string):
"<h1>Hello</h1><p>World</p>"

// But this (Tree structure):
{
  type: "doc",
  content: [
    {
      type: "heading",
      attrs: { level: 1 },
      content: [{ type: "text", text: "Hello" }]
    },
    {
      type: "paragraph",
      content: [{ type: "text", text: "World" }]
    }
  ]
}
```

**Why tree over string?**
- ✅ Type-safe editing
- ✅ Structural validation
- ✅ Easy to transform
- ✅ Efficient updates

#### 2. Schema

Defines **what's allowed** in the document:

```typescript
const schema = new Schema({
  nodes: {
    doc: { content: "block+" },          // Document contains blocks
    paragraph: { content: "inline*" },    // Paragraph contains inline
    heading: {
      attrs: { level: { default: 1 } },
      content: "inline*"
    },
    text: { inline: true },
  },
  marks: {
    bold: {},                             // Bold is a mark (inline style)
    italic: {},
  },
})
```

**Validation Example:**

```typescript
// ✅ Valid (heading contains text)
{ type: "heading", content: [{ type: "text", text: "Hello" }] }

// ❌ Invalid (heading can't contain paragraph)
{ type: "heading", content: [{ type: "paragraph", ... }] }
//                                  ↑
//                    Schema error! Not allowed
```

💡 **Architectural Lesson**: Schema = **contract** for document structure. Like TypeScript for data.

#### 3. State

The **current state** of the editor:

```typescript
class EditorState {
  doc: Node                // Current document
  selection: Selection     // Current selection/cursor
  storedMarks: Mark[]      // Marks to apply on next insert
  // ... plugins, etc.
}
```

**Immutable**: Every change creates a NEW state (like Redux).

#### 4. Transactions

**How to change state**:

```typescript
// Create transaction
let tr = state.tr
  .insertText("Hello")
  .setSelection(/* new selection */)
  .setMeta("source", "user")

// Apply transaction → new state
let newState = state.apply(tr)
```

**Transaction = Redux Action + Reducer combined**

Pattern:
```typescript
Old State + Transaction = New State
```

#### 5. View

Renders state to DOM:

```typescript
let view = new EditorView(document.querySelector("#editor"), {
  state,
  dispatchTransaction(transaction) {
    let newState = view.state.apply(transaction)
    view.updateState(newState)
  }
})
```

### ProseMirror Plugins

**Plugins** add behaviors:

```typescript
const myPlugin = new Plugin({
  // 1. Plugin state
  state: {
    init(config, state) {
      return { count: 0 }
    },
    apply(tr, value, oldState, newState) {
      return { count: value.count + 1 }
    },
  },

  // 2. Decorations (visual overlays)
  props: {
    decorations(state) {
      return DecorationSet.create(/* decorations */)
    },
  },

  // 3. Event handlers
  props: {
    handleKeyDown(view, event) {
      if (event.key === "Tab") {
        // Custom tab handling
        return true  // Handled
      }
      return false  // Not handled
    },
  },

  // 4. View methods
  view(editorView) {
    return {
      update(view, prevState) {
        // Called on every state update
      },
      destroy() {
        // Cleanup
      },
    }
  },
})
```

**Example in Novel**: Upload Images Plugin

```typescript
// packages/headless/src/plugins/upload-images.tsx
export const UploadImagesPlugin = () => {
  return new Plugin({
    state: {
      init() {
        return DecorationSet.empty
      },
      apply(tr, set) {
        // Track upload decorations
        set = set.map(tr.mapping, tr.doc)

        const action = tr.getMeta(this)
        if (action?.add) {
          // Add upload placeholder
          const decoration = Decoration.widget(action.add.pos, () => {
            return createPlaceholderElement()
          })
          set = set.add(tr.doc, [decoration])
        }

        if (action?.remove) {
          // Remove placeholder
          set = set.remove(/* ... */)
        }

        return set
      },
    },

    props: {
      decorations(state) {
        return this.getState(state)
      },

      // Handle image drops
      handleDOMEvents: {
        drop(view, event) {
          const files = event.dataTransfer?.files
          if (files?.length) {
            // Handle file upload
            return true
          }
          return false
        },
      },
    },
  })
}
```

🎯 **Pattern to Master**: Decorations for **ephemeral UI**
- Upload progress indicators
- Collaboration cursors
- Spelling errors
- Search highlights

These don't modify document content (just visual overlays).

### When You'll Use ProseMirror Directly

In Tiptap, you access ProseMirror through:

```typescript
// In extension
addProseMirrorPlugins() {
  return [
    new Plugin({ /* ProseMirror plugin */ })
  ]
}

// In component
const editor = useEditor({
  /* config */
})

// Access ProseMirror internals
editor.view        // ProseMirror EditorView
editor.state       // ProseMirror EditorState
editor.schema      // ProseMirror Schema
```

### ProseMirror Resources

- 📚 **Guide**: https://prosemirror.net/docs/guide/
- 🔬 **Reference**: https://prosemirror.net/docs/ref/
- 💬 **Forum**: https://discuss.prosemirror.net/

💡 **Learning Path**: Learn Tiptap first, dive into ProseMirror when you need advanced features.

---

## React 18: The UI Layer

### Project Uses React 18.2.0

✅ **Current (Nov 2025)**: Stable, widely used
🚨 **Note**: React 19.2.0 available (new features, backward compatible)

### React Patterns in Novel

#### 1. Function Components with Hooks

**All components are function components:**

```tsx
// ✅ Novel's approach
export const EditorRoot: FC<EditorRootProps> = ({ children }) => {
  const tunnelInstance = useRef(tunnel()).current
  return <Provider>{children}</Provider>
}

// ❌ Not used: Class components
class EditorRoot extends Component { }
```

**Why?**
- ✅ Simpler syntax
- ✅ Hooks for state/effects
- ✅ Better TypeScript inference
- ✅ Smaller bundle size

#### 2. Custom Hooks Pattern

**Example**: `useEditor` from Tiptap

```tsx
import { useEditor } from '@tiptap/react'

function MyEditor() {
  const editor = useEditor({
    extensions: [/* ... */],
    content: '<p>Hello</p>',
    onUpdate: ({ editor }) => {
      // Called on every change
    },
  })

  return <EditorContent editor={editor} />
}
```

**Pattern**: Custom hooks encapsulate logic

```tsx
// Custom hook for Novel
export function useNovelEditor(initialContent: JSONContent) {
  const [saveStatus, setSaveStatus] = useState('Saved')

  const editor = useEditor({
    extensions: defaultExtensions,
    content: initialContent,
    onUpdate: ({ editor }) => {
      setSaveStatus('Saving...')
      debouncedSave(editor.getJSON())
    },
  })

  return { editor, saveStatus }
}

// Usage
function MyEditor() {
  const { editor, saveStatus } = useNovelEditor(content)
  return <div>{saveStatus}: <EditorContent editor={editor} /></div>
}
```

💡 **Learn This**: Extract complex logic into custom hooks for reusability.

#### 3. Context + Provider Pattern

**Novel uses this for:**
- Jotai store (state management)
- Tunnel instance (command menu portal)

```tsx
// 1. Create context
const EditorCommandTunnelContext = createContext<TunnelInstance | null>(null)

// 2. Provide value
export const EditorRoot = ({ children }) => {
  const tunnel = useRef(tunnel()).current

  return (
    <Provider store={novelStore}>
      <EditorCommandTunnelContext.Provider value={tunnel}>
        {children}
      </EditorCommandTunnelContext.Provider>
    </Provider>
  )
}

// 3. Consume with custom hook
export const useEditorCommandTunnel = () => {
  const tunnel = useContext(EditorCommandTunnelContext)
  if (!tunnel) {
    throw new Error("Must be used within EditorRoot")
  }
  return tunnel
}

// 4. Use in components
function EditorCommand() {
  const tunnel = useEditorCommandTunnel()
  return <tunnel.In>Content</tunnel.In>
}
```

**Pattern Benefits:**
- ✅ Type-safe context access
- ✅ Runtime error if used incorrectly
- ✅ Clean API for consumers

#### 4. Ref Pattern for Stable Instances

**Why `.current` pattern?**

```tsx
// ❌ BAD: Instance recreated every render
const tunnel = tunnel()  // New instance each time!

// ✅ GOOD: Stable instance
const tunnelInstance = useRef(tunnel()).current
//                                        ↑
//                     Access immediately, store in ref
```

**When to use:**
- Creating third-party instances (Tippy.js, tunnel-rat)
- Storing mutable values
- Accessing DOM nodes

#### 5. forwardRef for Ref Forwarding

```tsx
export const EditorContent = forwardRef<HTMLDivElement, Props>(
  ({ className, children, ...rest }, ref) => (
    <div ref={ref} className={className}>
      <EditorProvider {...rest}>{children}</EditorProvider>
    </div>
  ),
)
```

**Why?**
- Allows parent to access underlying DOM node
- Pattern: Common in library components

**Usage:**
```tsx
function Parent() {
  const editorRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    // Can access div!
    console.log(editorRef.current)
  }, [])

  return <EditorContent ref={editorRef} />
}
```

#### 6. Composition Pattern

```tsx
// ✅ Compose features
<EditorRoot>
  <EditorContent />
  <EditorBubble>
    <BubbleButton />
  </EditorBubble>
  <EditorCommand>
    <CommandItem />
  </EditorCommand>
</EditorRoot>

// ❌ Not: Single component with all features
<Editor
  withBubbleMenu
  withSlashCommands
  /* 50 props */
/>
```

**Why composition?**
- ✅ Tree-shakeable (unused components excluded)
- ✅ Flexible (mix and match)
- ✅ Clear hierarchy

### React 19 Updates (What's New)

⚠️ **Note**: Project uses React 18, but React 19 is available

**Key React 19 Features:**

1. **Actions** - Async transitions
```tsx
// React 19
function MyForm() {
  const [isPending, startTransition] = useTransition()

  async function handleSubmit() {
    startTransition(async () => {
      await saveData()  // Async!
    })
  }
}
```

2. **`use()` Hook** - Read promises/context
```tsx
// React 19
function Component({ dataPromise }) {
  const data = use(dataPromise)  // Suspends until ready
  return <div>{data}</div>
}
```

3. **Server Components** - Now stable
```tsx
// app/page.tsx (React 19 + Next.js)
async function Page() {
  const data = await fetch(/* ... */)  // Server-side!
  return <div>{data}</div>
}
```

💡 **Migration Note**: React 18 → 19 is mostly backward compatible. Novel would benefit from:
- Server Components for demo app
- Actions for form handling
- Better TypeScript inference

### React Resources

- 📚 **Docs**: https://react.dev
- 🆕 **React 19**: https://react.dev/blog/2024/12/05/react-19
- 🎓 **Hooks**: https://react.dev/reference/react

---

## Next.js 15: The Framework

### Project Uses Next.js 15.1.4

✅ **Current**: Latest v15.x release
⚠️ **Note**: Next.js 16 available (October 2025)

### App Router Architecture

Novel's demo (`apps/web`) uses **App Router** (not Pages Router).

**File-Based Routing:**

```
app/
├── layout.tsx           → Root layout (all pages)
├── page.tsx             → Home page (/)
├── about/page.tsx       → /about route
└── api/
    └── generate/
        └── route.ts     → /api/generate endpoint
```

#### layout.tsx (Shell)

```tsx
export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

**Rules:**
- Must return `<html>` and `<body>`
- Wraps all pages
- Can nest layouts (layout per section)

#### page.tsx (Routes)

```tsx
export default function HomePage() {
  return <div>Home</div>
}
```

**Exports:**
- `default`: Page component
- `generateMetadata`: SEO metadata
- `generateStaticParams`: Static generation params

#### API Routes (Edge Runtime)

**Novel uses Edge Runtime for AI endpoints:**

```typescript
// app/api/generate/route.ts
export const runtime = 'edge'  // ← Run on Vercel Edge Network

export async function POST(req: Request) {
  const body = await req.json()

  // Call OpenAI
  const response = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [{ role: 'user', content: body.prompt }],
    stream: true,
  })

  // Return streaming response
  return new Response(response.toReadableStream())
}
```

**Edge Runtime Benefits:**
- ✅ <50ms cold starts (vs ~1s for Node.js)
- ✅ Global deployment (low latency everywhere)
- ✅ Lower cost
- ⚠️ Limited APIs (no `fs`, limited `crypto`)

**When to use Edge:**
- API proxies (like AI completions)
- Middleware
- Serverless functions with external API calls

**When to use Node.js runtime:**
```typescript
export const runtime = 'nodejs'

export async function GET(req: Request) {
  // Full Node.js APIs available
  const fs = require('fs')
  // Database connections, etc.
}
```

### Server vs Client Components

**Default: Server Components**

```tsx
// app/page.tsx (Server Component by default)
export default function Page() {
  // Runs on server
  return <div>Hello</div>
}
```

**Client Components:**

```tsx
'use client'  // ← Opt-in to client

export default function Page() {
  const [count, setCount] = useState(0)  // Hooks work!
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

**Rules:**
- Server: No hooks, no browser APIs, can be async
- Client: Hooks, browser APIs, interactive

**Pattern in Novel:**

```tsx
// app/layout.tsx (Server)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>  {/* Client boundary */}
      </body>
    </html>
  )
}

// app/providers.tsx (Client)
'use client'

export function Providers({ children }) {
  return (
    <ThemeProvider>
      <AppContext>
        {children}
      </AppContext>
    </ThemeProvider>
  )
}
```

💡 **Pattern**: Keep Server Components where possible, mark `'use client'` only when needed.

### Data Fetching

**Server Components can fetch directly:**

```tsx
// app/page.tsx
export default async function Page() {
  const data = await fetch('https://api.example.com/data')
  const json = await data.json()

  return <div>{json.title}</div>
}
```

**No useEffect needed!**

**Client Components use hooks:**

```tsx
'use client'

export default function Page() {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetch('/api/data')
      .then(r => r.json())
      .then(setData)
  }, [])

  return <div>{data?.title}</div>
}
```

### Caching & Revalidation

```typescript
// Cached forever (static)
fetch('https://api.example.com/data')

// Cached for 60 seconds
fetch('https://api.example.com/data', {
  next: { revalidate: 60 }
})

// Never cached (dynamic)
fetch('https://api.example.com/data', {
  cache: 'no-store'
})
```

### Next.js 16 Updates (What's New)

⚠️ **Note**: Project uses v15, but v16 available

**Key Changes:**
1. **Turbopack** - Default bundler (much faster)
2. **Cache Components** - New caching primitives
3. **`proxy.ts`** - Replaces `middleware.ts`
4. **React 19 support** - Built-in

💡 **Migration Path**: v15 → v16 is incremental. Main breaking change: `middleware.ts` → `proxy.ts`

### Next.js Resources

- 📚 **Docs**: https://nextjs.org/docs
- 🆕 **v16 Release**: https://nextjs.org/blog/next-16
- 🎓 **App Router**: https://nextjs.org/docs/app

---

## Jotai: Atomic State Management

### What is Jotai?

**Jotai** (状態 = "state" in Japanese) is a **primitive and flexible state management** library for React.

🧠 **Mental Model**: Jotai atoms are like individual `useState` hooks, but global.

### Why Jotai Over Redux/Zustand/Context?

| Feature | Jotai | Redux | Zustand | Context |
|---------|-------|-------|---------|---------|
| **API Size** | Minimal | Large | Small | Built-in |
| **Boilerplate** | None | High | Low | Medium |
| **TypeScript** | Excellent | Good | Excellent | Good |
| **Re-render** | Optimal | Good | Good | Poor |
| **DevTools** | Yes | Yes | Yes | No |
| **Learning Curve** | Low | High | Low | Low |

**Why Novel Uses Jotai:**
1. ✅ **Atomic** - Each piece of state is independent
2. ✅ **Isolated** - Each editor instance gets own store
3. ✅ **Minimal API** - Just `atom()` and `useAtom()`
4. ✅ **TypeScript** - Perfect inference
5. ✅ **No Provider Hell** - Single provider for all atoms

### Core Concepts

#### 1. Atoms

**An atom is a piece of state:**

```typescript
import { atom } from 'jotai'

// Primitive atom
export const countAtom = atom(0)
//                            ↑
//                      Initial value

// Object atom
export const userAtom = atom<User | null>(null)

// Derived atom (computed)
export const doubleCountAtom = atom(
  (get) => get(countAtom) * 2
  //   ↑
  // Read other atoms
)
```

#### 2. Reading & Writing

```typescript
import { useAtom } from 'jotai'

function Counter() {
  const [count, setCount] = useAtom(countAtom)
  //           ↑                         ↑
  //      Just like useState!     Your atom

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

**Read-only:**
```typescript
import { useAtomValue } from 'jotai'

function Display() {
  const count = useAtomValue(countAtom)  // No setter
  return <div>{count}</div>
}
```

**Write-only:**
```typescript
import { useSetAtom } from 'jotai'

function Incrementer() {
  const setCount = useSetAtom(countAtom)  // No value
  return <button onClick={() => setCount(c => c + 1)}>+</button>
}
```

#### 3. Jotai in Novel

**Atoms used:**

```typescript
// packages/headless/src/utils/atoms.ts
import { atom } from 'jotai'

export const queryAtom = atom<string>("")
//                                    ↑
//                    Slash command search query

export const rangeAtom = atom<Range | null>(null)
//                                         ↑
//                     Cursor position when menu opened
```

**Store isolation:**

```typescript
// packages/headless/src/utils/store.ts
import { createStore } from 'jotai'

export const novelStore = createStore()
//                            ↑
//          Each editor instance gets this store
```

**Why custom store?**

```tsx
// Default Jotai: One global store (like Redux)
<Provider>
  <Editor />  {/* Shares state */}
  <Editor />  {/* Shares state */}
</Provider>

// Novel: Isolated stores
<Provider store={novelStore}>
  <Editor />  {/* Own state */}
</Provider>

<Provider store={novelStore}>
  <Editor />  {/* Own state */}
</Provider>
```

This allows **multiple independent editor instances** on one page!

#### 4. Derived Atoms

**Computed values:**

```typescript
const firstNameAtom = atom("John")
const lastNameAtom = atom("Doe")

const fullNameAtom = atom(
  (get) => `${get(firstNameAtom)} ${get(lastNameAtom)}`
)

// Usage
function Display() {
  const fullName = useAtomValue(fullNameAtom)
  // Automatically updates when firstName or lastName change!
}
```

**Write to derived atom:**

```typescript
const fullNameAtom = atom(
  // Read
  (get) => `${get(firstNameAtom)} ${get(lastNameAtom)}`,

  // Write
  (get, set, newFullName: string) => {
    const [first, last] = newFullName.split(' ')
    set(firstNameAtom, first)
    set(lastNameAtom, last)
  }
)
```

#### 5. Async Atoms

```typescript
const userIdAtom = atom(1)

const userAtom = atom(async (get) => {
  const userId = get(userIdAtom)
  const response = await fetch(`/api/users/${userId}`)
  return response.json()
})

// Usage (with Suspense)
function UserProfile() {
  const user = useAtomValue(userAtom)  // Suspends until loaded
  return <div>{user.name}</div>
}
```

### Pattern: Atoms for Editor State

**Novel could expand with more atoms:**

```typescript
// Hypothetical additions
export const editorReadOnlyAtom = atom(false)
export const editorThemeAtom = atom<'light' | 'dark'>('light')
export const editorFocusedAtom = atom(false)

// Usage in extensions
function MyExtension() {
  const [isReadOnly] = useAtom(editorReadOnlyAtom)
  const [theme] = useAtom(editorThemeAtom)

  // Extension behavior based on state
}
```

### Jotai DevTools

```tsx
import { DevTools } from 'jotai-devtools'

function App() {
  return (
    <>
      <DevTools />  {/* Debug atoms in browser */}
      <YourApp />
    </>
  )
}
```

### Jotai vs Zustand

**When to use Jotai:**
- ✅ Fine-grained state (many small atoms)
- ✅ Derived/computed values
- ✅ React-first approach
- ✅ Need Suspense integration

**When to use Zustand:**
- ✅ Single store approach
- ✅ Framework-agnostic (works outside React)
- ✅ Simpler mental model (one store)

💡 **Architectural Lesson**: Jotai's atomic approach prevents unnecessary re-renders.

```tsx
// Zustand: Whole store subscribers re-render
const useStore = create((set) => ({
  query: "",
  range: null,
  count: 0,  // ← Changing this...
}))

function Component() {
  const query = useStore(state => state.query)  // ← ...re-renders this!
}

// Jotai: Only atom subscribers re-render
const queryAtom = atom("")
const countAtom = atom(0)

function Component() {
  const [query] = useAtom(queryAtom)  // ← countAtom changes? No re-render!
}
```

### Jotai Resources

- 📚 **Docs**: https://jotai.org
- 🎓 **Tutorial**: https://jotai.org/docs/basics/primitives
- 💬 **Community**: https://discord.gg/poimandres

**Current Version**: v2.11.0 (very recent!)

---

## TypeScript: Type Safety

### Project Uses TypeScript 5.7.3

✅ **Current**: Very recent (Nov 2024 release)
⚠️ **Note**: TypeScript 5.9 available (August 2025)

### Why TypeScript?

**Benefits in Novel:**
1. ✅ **Catch errors early** - Before runtime
2. ✅ **Better IDE support** - Autocomplete, refactoring
3. ✅ **Self-documenting** - Types are documentation
4. ✅ **Safer refactoring** - Compiler finds all usages
5. ✅ **Editor extensions** - Type-safe extension API

### TypeScript Patterns in Novel

#### 1. Interface vs Type

**Novel uses both, strategically:**

```typescript
// ✅ Interface for component props (can be extended)
export interface EditorContentProps extends EditorProviderProps {
  readonly children?: ReactNode
  readonly className?: string
  readonly initialContent?: JSONContent
}

// ✅ Type for unions/intersections
export type SuggestionItem = {
  title: string
  description: string
  icon: ReactNode
  command?: (props: CommandProps) => void
}
```

**Rule of Thumb:**
- **Interface** - Objects that might be extended
- **Type** - Unions, intersections, mapped types

#### 2. Generic Types

**Tiptap uses generics heavily:**

```typescript
// Generic extension type
class Extension<Options = any, Storage = any> {
  options: Options
  storage: Storage
}

// Usage
const MyExtension = Extension.create<{
  placeholder: string
}>({
  addOptions() {
    return {
      placeholder: 'Type something...'
    }
  }
})
```

**In Novel:**

```typescript
// Generic create function
export const createSuggestionItems = <T extends SuggestionItem>(
  items: T[]
): T[] => items

// Preserves specific type!
const items = createSuggestionItems([
  { title: 'Heading', /* ... */ }
])
// items: Array<{ title: string, ... }>
```

#### 3. Readonly Properties

**Novel makes props readonly:**

```typescript
export interface EditorProps {
  readonly children: ReactNode    // ← Can't be modified
  readonly className?: string
}
```

**Why readonly?**
- ✅ Prevents accidental mutation
- ✅ Signals intent (data flows down)
- ✅ Functional programming style

#### 4. Type Guards

```typescript
// Check if editor has specific command
function hasCommand(editor: Editor, commandName: string): boolean {
  return commandName in editor.commands
}

// Type guard for safer checks
function isEditorActive(editor: Editor | null): editor is Editor {
  return editor !== null && editor.isEditable
}

// Usage
if (isEditorActive(editor)) {
  editor.commands.focus()  // TypeScript knows editor is not null!
}
```

#### 5. Utility Types

**Novel uses TypeScript's built-in utilities:**

```typescript
// Omit - Remove properties
export type EditorContentProps = Omit<EditorProviderProps, "content"> & {
  readonly initialContent?: JSONContent
}
//                                     ↑
//                 Remove 'content', add 'initialContent'

// Partial - Make all properties optional
type PartialExtensionOptions = Partial<ExtensionOptions>

// Pick - Select specific properties
type JustTitle = Pick<SuggestionItem, 'title' | 'description'>

// Record - Create object type
type CommandMap = Record<string, CommandFunction>
```

#### 6. Type Inference

**Let TypeScript infer when possible:**

```typescript
// ❌ Redundant type annotation
const count: number = 0

// ✅ Inferred
const count = 0  // TypeScript knows it's number

// ❌ Verbose
const items: Array<SuggestionItem> = getSuggestions()

// ✅ Inferred from function return type
const items = getSuggestions()
```

#### 7. Template Literal Types

```typescript
// Next.js route types
type Route = `/api/${'generate' | 'upload'}`
//           ↑
//      Only these paths allowed

const route: Route = '/api/generate'  // ✅
const route: Route = '/api/delete'    // ❌ Type error!
```

### TypeScript Configuration

**tsconfig.json (headless package):**

```json
{
  "extends": "tsconfig/react.json",
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,                    // ← Strictest checks
    "skipLibCheck": true,
    "declaration": true,               // ← Generate .d.ts files
    "declarationMap": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
```

**Shared config (packages/tsconfig):**

```json
// tsconfig/react.json
{
  "compilerOptions": {
    "jsx": "react-jsx",                // ← New JSX transform
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### TypeScript 5.7 Features Used

**1. Const Type Parameters**
```typescript
function createArray<const T>(items: T[]) {
  return items
}

const arr = createArray(['a', 'b'])
// Type: ['a', 'b'] (not string[])
```

**2. Improved Type Narrowing**
```typescript
function process(value: string | null) {
  if (!value) return

  // TypeScript 5.7 knows value is string here
  console.log(value.toUpperCase())
}
```

### Common TypeScript Patterns

#### Pattern 1: Props with Children

```typescript
type PropsWithChildren<P = unknown> = P & { children?: ReactNode }

// Usage
interface MyProps {
  title: string
}

type MyComponentProps = PropsWithChildren<MyProps>
// Result: { title: string; children?: ReactNode }
```

#### Pattern 2: Event Handlers

```typescript
type ClickHandler = (event: React.MouseEvent<HTMLButtonElement>) => void

interface ButtonProps {
  onClick?: ClickHandler
}
```

#### Pattern 3: Discriminated Unions

```typescript
type Action =
  | { type: 'SET_QUERY'; query: string }
  | { type: 'SET_RANGE'; range: Range }
  | { type: 'RESET' }

function reducer(state: State, action: Action) {
  switch (action.type) {
    case 'SET_QUERY':
      return { ...state, query: action.query }
      //                          ↑
      //            TypeScript knows query exists!

    case 'SET_RANGE':
      return { ...state, range: action.range }

    case 'RESET':
      return initialState
  }
}
```

### TypeScript Resources

- 📚 **Handbook**: https://www.typescriptlang.org/docs/handbook/
- 🆕 **5.7 Release**: https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/
- 🎓 **React + TypeScript**: https://react-typescript-cheatsheet.netlify.app/

---

## Tailwind CSS: Utility-First Styling

### Project Uses Tailwind v3.4.1

⚠️ **MAJOR UPDATE AVAILABLE**: Tailwind v4.1 (complete rewrite!)
🚨 **Note**: v4 has breaking changes. See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

### What is Tailwind?

**Utility-first CSS** - Style with classes instead of writing custom CSS.

```html
<!-- ❌ Traditional CSS -->
<style>
  .button {
    background-color: blue;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 0.25rem;
  }
</style>
<button class="button">Click me</button>

<!-- ✅ Tailwind -->
<button class="bg-blue-500 text-white px-4 py-2 rounded">
  Click me
</button>
```

### Why Tailwind?

**Benefits:**
1. ✅ **No naming** - No more "what should I call this class?"
2. ✅ **Consistent** - Design system built-in
3. ✅ **Fast** - No context switching (HTML → CSS → HTML)
4. ✅ **Tree-shakeable** - Only used classes in production
5. ✅ **Responsive** - `md:`, `lg:` modifiers built-in
6. ✅ **Dark mode** - `dark:` modifier built-in

**Drawbacks:**
- ⚠️ Long class names
- ⚠️ Learning curve (memorize utilities)
- ⚠️ HTML can look messy

### Tailwind in Novel

#### Configuration

**tailwind.config.ts:**

```typescript
export default {
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
  ],
  darkMode: 'class',              // ← Enable dark mode
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        // ... custom colors via CSS variables
      },
      typography: {
        DEFAULT: {
          css: {
            // Customize prose (markdown) styles
          },
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),  // ← Prose styles
  ],
}
```

#### CSS Variables Pattern

**Novel uses CSS variables for theming:**

```css
/* globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 210 40% 98%;
  }
}
```

**Usage:**

```tsx
<div className="bg-background text-foreground">
  {/* Light: white bg, dark text */}
  {/* Dark: dark bg, light text */}
</div>
```

**Why this pattern?**
- ✅ Single source of truth (CSS variables)
- ✅ Easy to switch themes
- ✅ Works with Tailwind's utilities

#### Common Patterns

**1. Responsive Design:**

```tsx
<div className="
  w-full              /* Mobile: full width */
  md:w-1/2            /* Tablet: half width */
  lg:w-1/3            /* Desktop: third width */
">
```

**2. Dark Mode:**

```tsx
<div className="
  bg-white dark:bg-gray-900
  text-black dark:text-white
">
```

**3. Hover States:**

```tsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  transition-colors     /* Smooth color transition */
">
```

**4. Focus States:**

```tsx
<input className="
  border-gray-300
  focus:border-blue-500
  focus:ring-2
  focus:ring-blue-200
"/>
```

**5. Group Hover:**

```tsx
<div className="group">
  <button className="
    group-hover:bg-blue-500
  ">
    Hover the parent!
  </button>
</div>
```

#### Prose Plugin (Typography)

**For rich text content:**

```tsx
<div className="prose prose-lg dark:prose-invert">
  {/* Automatic styling for: */}
  {/* - Headings */}
  {/* - Paragraphs */}
  {/* - Lists */}
  {/* - Blockquotes */}
  {/* - Code blocks */}
  <h1>Auto-styled!</h1>
  <p>No custom CSS needed</p>
</div>
```

**Novel's editor content uses prose:**

```typescript
// apps/web/components/tailwind/extensions.ts
heading: {
  HTMLAttributes: {
    class: 'scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl',
  },
},
```

### Tailwind v4 (What's New)

🚨 **Major Rewrite** (Jan 22, 2025)

**Key Changes:**
1. **5x faster builds**
2. **100x faster incremental builds** (microseconds!)
3. **Zero config** - Auto-detect content files
4. **Modern CSS** - Uses `@property`, `color-mix()`, etc.
5. **Single line setup** - Just `@import "tailwindcss"`

**Migration Example:**

```css
/* v3 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* v4 */
@import "tailwindcss";
```

**Breaking Changes:**
- New configuration format
- Some utilities renamed
- Plugin API changed

💡 **For Novel**: v3 → v4 migration is significant. Current v3 setup works perfectly.

### When NOT to Use Tailwind

```tsx
// ❌ Too complex for Tailwind
<div className="w-[calc(100vw-theme(spacing.64))] ...">

// ✅ Better: Custom CSS
<style>{`
  .custom-width {
    width: calc(100vw - 16rem);
  }
`}</style>
```

**Use custom CSS for:**
- Complex calculations
- Animations (keyframes)
- Very specific designs

### Tailwind Resources

- 📚 **Docs**: https://tailwindcss.com/docs
- 🆕 **v4 Release**: https://tailwindcss.com/blog/tailwindcss-v4
- 🎨 **UI Components**: https://tailwindui.com/
- 🎓 **Play**: https://play.tailwindcss.com/

---

## Vercel AI SDK: AI Integration

### Project Uses AI SDK v3.0.12

🚨 **MAJOR UPDATE**: AI SDK v5 available (breaking changes!)

### What is Vercel AI SDK?

**Unified API** for working with AI models (OpenAI, Anthropic, etc.).

🧠 **Mental Model**: Like `axios` for HTTP, but for AI APIs.

### Why Vercel AI SDK?

**Benefits:**
1. ✅ **Unified API** - Switch providers easily
2. ✅ **Streaming** - Real-time token responses
3. ✅ **React integration** - `useChat()`, `useCompletion()`
4. ✅ **Type-safe** - Full TypeScript support
5. ✅ **Edge-ready** - Works on Edge Runtime

**Alternatives:**
- Direct OpenAI SDK (more verbose)
- LangChain (overkill for simple use cases)
- DIY (reinvent the wheel)

### AI SDK in Novel

**API Route (Edge Function):**

```typescript
// apps/web/app/api/generate/route.ts
import { OpenAIStream, StreamingTextResponse } from 'ai'
import OpenAI from 'openai'

export const runtime = 'edge'

export async function POST(req: Request) {
  const { prompt, option } = await req.json()

  const response = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    stream: true,              // ← Enable streaming
    messages: [
      {
        role: 'system',
        content: 'You are a helpful writing assistant.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
  })

  // Convert to Edge-compatible stream
  const stream = OpenAIStream(response)

  return new StreamingTextResponse(stream)
}
```

**Frontend (React):**

```typescript
// apps/web/components/tailwind/generative/ai-selector.tsx
import { useCompletion } from 'ai/react'

export function AISelector() {
  const { completion, complete, isLoading } = useCompletion({
    api: '/api/generate',
    onFinish: (prompt, completion) => {
      // Insert AI text into editor
      editor.commands.insertContent(completion)
    },
  })

  return (
    <div>
      <button
        onClick={() => complete('Continue writing...')}
        disabled={isLoading}
      >
        {isLoading ? 'Generating...' : 'Generate'}
      </button>

      {completion && (
        <div>{completion}</div>  {/* Streams in real-time! */}
      )}
    </div>
  )
}
```

### Streaming Explained

**Without streaming:**
```
User clicks → Wait 5 seconds → Full response appears
```

**With streaming:**
```
User clicks → "Hello" → "world" → "this" → "is" → "AI"
                ↑ 100ms  ↑ 200ms  ↑ 300ms
```

**Implementation:**

```typescript
// Server (Edge Function)
const stream = OpenAIStream(response, {
  onStart: async () => {
    console.log('Started')
  },
  onToken: async (token) => {
    console.log('Token:', token)
  },
  onCompletion: async (completion) => {
    console.log('Done:', completion)
  },
})

return new StreamingTextResponse(stream)
```

**How it works:**
1. OpenAI sends **Server-Sent Events** (SSE)
2. `OpenAIStream` converts to **ReadableStream**
3. Frontend receives tokens as they arrive
4. `useCompletion` updates React state on each token

### AI SDK v5 (What's New)

🚨 **Breaking Changes**

**Key Updates:**
1. **New `streamText()` API**
```typescript
// v5
import { streamText } from 'ai'

const result = await streamText({
  model: openai('gpt-3.5-turbo'),
  messages: [{ role: 'user', content: 'Hello' }],
})

return result.toAIStreamResponse()
```

2. **Better TypeScript**
```typescript
// v5 has end-to-end type safety
const result = await streamText({
  model: openai('gpt-3.5-turbo'),
  messages: typed,  // Fully typed!
})
```

3. **New hooks:**
```typescript
// v5
import { useChat } from 'ai/react'

const { messages, input, handleSubmit } = useChat()
```

💡 **For Novel**: Consider upgrading to v5 for better type safety and API.

### Rate Limiting with Upstash

**Novel uses Vercel KV (Upstash Redis) for rate limiting:**

```typescript
import { Ratelimit } from '@upstash/ratelimit'
import { kv } from '@vercel/kv'

const ratelimit = new Ratelimit({
  redis: kv,
  limiter: Ratelimit.slidingWindow(50, '1 d'),
  //                                ↑   ↑
  //                       50 requests per day
})

export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for') ?? 'unknown'

  const { success, limit, reset, remaining } = await ratelimit.limit(ip)

  if (!success) {
    return new Response('Rate limit exceeded', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    })
  }

  // Process request...
}
```

**Why rate limiting?**
- ✅ Prevent API abuse
- ✅ Control costs (OpenAI charges per token)
- ✅ Fair usage across users

### AI SDK Resources

- 📚 **Docs**: https://sdk.vercel.ai/docs
- 🆕 **v5 Release**: https://vercel.com/blog/ai-sdk-5
- 🎓 **Examples**: https://github.com/vercel/ai/tree/main/examples

---

## Turborepo: Monorepo Orchestration

### Project Uses Turborepo v2.3.3

⚠️ **Note**: v2.6.1 available (3 minor versions behind)

### What is Turborepo?

**Build system** for JavaScript/TypeScript monorepos.

🧠 **Mental Model**: Like `make` for JavaScript, but smarter.

### Why Turborepo?

**Alternatives:**
- **Nx** - More features, more complex
- **Lerna** - Older, less performant
- **Rush** - Microsoft's tool, less popular
- **DIY** - npm/pnpm workspaces only (no caching)

**Why Turborepo Won:**
1. ✅ **Incremental builds** - Cache previous builds
2. ✅ **Parallel execution** - Run tasks concurrently
3. ✅ **Simple config** - One `turbo.json` file
4. ✅ **Remote caching** - Share cache across team
5. ✅ **Written in Rust** - Extremely fast

### Turbo Configuration

**turbo.json:**

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build", "typecheck"],
      //           ↑        ↑
      //           │        └─ Run typecheck in THIS package first
      //           └─ Run build in DEPENDENCIES first

      "outputs": ["dist/**", ".next/**"],
      //          ↑
      //          Cache these directories

      "cache": true
    },

    "dev": {
      "cache": false,      // ← Never cache dev
      "persistent": true   // ← Keep running
    },

    "typecheck": {
      "dependsOn": ["^topo"],
      //              ↑
      //              Respect dependency order (topology)
    }
  }
}
```

### How Turborepo Works

**1. Dependency Graph:**

```
apps/web
  └─ depends on packages/headless
      └─ depends on packages/tsconfig

packages/headless
  └─ depends on packages/tsconfig

packages/tsconfig
  └─ no dependencies
```

**2. Task Execution:**

```bash
$ pnpm build

Turborepo analyzes:
  1. packages/tsconfig has no deps → Build first
  2. packages/headless depends on tsconfig → Build second
  3. apps/web depends on headless → Build last

Execution (parallel where possible):
  ┌─ packages/tsconfig (build)
  │
  └─ packages/headless (typecheck + build)
      │
      └─ apps/web (typecheck + build)
```

**3. Caching:**

```bash
$ pnpm build
✓ Build complete (30s)

# No changes, run again:
$ pnpm build
✓ Build complete (0.5s)  ← Retrieved from cache!
```

**How caching works:**
1. Compute hash of inputs (source files, deps, config)
2. Check cache for hash
3. If hit: restore outputs
4. If miss: run task, cache outputs

### Turborepo Commands

```bash
# Run task in all packages
pnpm turbo build

# Run with cache disabled
pnpm turbo build --force

# Run in specific package
pnpm turbo build --filter=novel

# Run with dependencies
pnpm turbo build --filter=novel...

# Parallel execution (default)
pnpm turbo test  # Runs all tests in parallel

# Show what will run (dry run)
pnpm turbo build --dry-run
```

### Remote Caching

**Share cache across team:**

```bash
# Login to Vercel
pnpm turbo login

# Link to project
pnpm turbo link

# Now builds cached remotely!
pnpm turbo build
# ✓ Cache hit from teammate's build
```

**Benefits:**
- ✅ Faster CI/CD (reuse cache from local builds)
- ✅ Faster local builds (reuse cache from CI)
- ✅ Team efficiency

### Turborepo v2.6 (What's New)

⚠️ **Note**: Project uses v2.3.3, latest is v2.6.1

**New in v2.6:**
1. **Microfrontend proxy** - Run all apps on one port
2. **Bun lockfile support** - If using Bun instead of pnpm

### Turborepo Resources

- 📚 **Docs**: https://turbo.build/repo/docs
- 🎓 **Handbook**: https://turbo.build/repo/docs/handbook
- 📊 **Benchmarks**: https://turbo.build/repo/docs/faq#how-is-turborepo-different-than-other-solutions

---

## pnpm: Package Management

### Project Uses pnpm v9.5.0

🚨 **Note**: pnpm v10.22.0 available (major version behind)

### Why pnpm Over npm/Yarn?

| Feature | pnpm | npm | Yarn (Classic) | Yarn (Berry) |
|---------|------|-----|----------------|--------------|
| **Speed** | Fast | Slow | Medium | Fast |
| **Disk Space** | Efficient | Wasteful | Medium | Efficient |
| **Strictness** | Strict | Loose | Loose | Strict |
| **Monorepos** | Excellent | Poor | Good | Excellent |

**pnpm Benefits:**
1. ✅ **Content-addressable storage** - One copy of each package
2. ✅ **Strict dependency resolution** - No phantom dependencies
3. ✅ **Fast installs** - Hardlinks instead of copies
4. ✅ **Workspace protocol** - Link local packages

### How pnpm Works

**npm/Yarn:**
```
node_modules/
  ├─ react/
  ├─ react-dom/
  └─ @tiptap/
      ├─ core/
      │   └─ node_modules/     ← Duplicate React!
      │       └─ react/
      └─ react/
          └─ node_modules/     ← Another duplicate!
              └─ react/
```

**pnpm:**
```
node_modules/
  ├─ .pnpm/                    ← Actual packages (hardlinked)
  │   ├─ react@18.2.0/
  │   ├─ @tiptap+core@2.11.2/
  │   └─ @tiptap+react@2.11.2/
  │
  └─ react/                    ← Symlink to .pnpm/react@18.2.0
```

**Result:**
- ✅ One copy of React (saves disk space)
- ✅ Faster installs (just create symlinks)
- ✅ Strict (can't import unlisted dependencies)

### pnpm Workspace

**pnpm-workspace.yaml:**

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**Workspace protocol:**

```json
// apps/web/package.json
{
  "dependencies": {
    "novel": "workspace:^"
    //        ↑
    //        Link to local packages/headless
  }
}
```

**Benefits:**
- ✅ Always uses local version (not npm)
- ✅ Changes instantly reflected
- ✅ No need to publish for testing

### Common pnpm Commands

```bash
# Install all dependencies
pnpm install

# Add dependency to specific package
pnpm add react --filter novel

# Remove dependency
pnpm remove react --filter novel

# Run script in all packages
pnpm -r run build
#      ↑
#      recursive

# Update dependencies
pnpm update

# Clean all node_modules
pnpm -r exec rm -rf node_modules
```

### pnpm v10 (What's New)

🚨 **Note**: Project uses v9, latest is v10

**New in v10:**
1. **Node.js runtime installation** - Auto-install Node versions
2. **Trust policies** - Security improvements
3. **Performance** - Even faster installs

### pnpm Resources

- 📚 **Docs**: https://pnpm.io
- 🎓 **Workspaces**: https://pnpm.io/workspaces
- 📊 **Benchmarks**: https://pnpm.io/benchmarks

---

## Biome: Code Quality

### Project Uses Biome v1.9.4

🚨 **Note**: Biome v2.3.6 available (major version behind)

### What is Biome?

**One tool** that replaces:
- ESLint (linter)
- Prettier (formatter)

🧠 **Mental Model**: Swiss Army knife for code quality.

### Why Biome Over ESLint + Prettier?

**Traditional:**
```json
{
  "devDependencies": {
    "eslint": "^8.0.0",
    "eslint-config-next": "^14.0.0",
    "eslint-plugin-react": "^7.0.0",
    "prettier": "^3.0.0",
    "prettier-plugin-tailwindcss": "^0.5.0"
  }
}
```

**With Biome:**
```json
{
  "devDependencies": {
    "@biomejs/biome": "^1.9.4"
  }
}
```

**Benefits:**
1. ✅ **One tool** - Simpler setup
2. ✅ **Faster** - Written in Rust
3. ✅ **97% Prettier compatible** - Drop-in replacement
4. ✅ **370+ lint rules** - More than ESLint
5. ✅ **Smaller node_modules** - One package vs many

### Biome Configuration

**biome.json:**

```json
{
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },

  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noExplicitAny": "warn"
      }
    }
  },

  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "trailingComma": "all"
    }
  }
}
```

### Biome Commands

```bash
# Format files
pnpm biome format --write .

# Lint files
pnpm biome lint .

# Fix linting issues
pnpm biome lint --apply .

# Check everything
pnpm biome check --apply .
```

### Integration with Turbo

**Novel's setup:**

```json
// package.json
{
  "scripts": {
    "lint": "turbo lint --continue",
    "lint:fix": "turbo lint --continue -- --apply",
    "format": "turbo format --continue",
    "format:fix": "turbo format --continue -- --write"
  }
}
```

### Biome v2 (What's New)

🚨 **Note**: Project uses v1, latest is v2

**New in v2:**
1. **Plugin system** - Extensible linter
2. **Vue/Svelte/Astro support** - Multi-framework
3. **Type-aware linting** - Uses TypeScript types

### Biome Resources

- 📚 **Docs**: https://biomejs.dev
- 🔧 **VSCode Extension**: https://marketplace.visualstudio.com/items?itemName=biomejs.biome

---

## Supporting Libraries

### Key Libraries in Novel

#### 1. Radix UI

**Headless UI components:**

```tsx
import * as Dialog from '@radix-ui/react-dialog'

<Dialog.Root>
  <Dialog.Trigger>Open</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Title>Title</Dialog.Title>
      <Dialog.Description>Description</Dialog.Description>
      <Dialog.Close>Close</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

**Why Radix?**
- ✅ Accessibility built-in (ARIA)
- ✅ Unstyled (bring your own styles)
- ✅ TypeScript first
- ✅ Battle-tested

**Used for:**
- Dialog modals
- Popovers (bubble menu)
- Selectors
- Scroll areas

#### 2. tunnel-rat

**Portal alternative:**

```tsx
import tunnel from 'tunnel-rat'

const t = tunnel()

// Component A
<t.In>Content</t.In>

// Component B (different part of tree)
<t.Out />  // Content appears here!
```

**Why not ReactDOM.createPortal?**
- ✅ Better SSR support
- ✅ Simpler API
- ✅ TypeScript friendly

#### 3. cmdk

**Command palette:**

```tsx
import { Command } from 'cmdk'

<Command>
  <Command.Input />
  <Command.List>
    <Command.Item onSelect={() => {}}>Item 1</Command.Item>
    <Command.Item>Item 2</Command.Item>
  </Command.List>
</Command>
```

**Features:**
- ✅ Fuzzy search
- ✅ Keyboard navigation
- ✅ Grouping
- ✅ Styling hooks

#### 4. class-variance-authority (CVA)

**Type-safe component variants:**

```tsx
import { cva } from 'class-variance-authority'

const button = cva('button-base', {
  variants: {
    intent: {
      primary: 'bg-blue-500',
      secondary: 'bg-gray-500',
    },
    size: {
      sm: 'text-sm',
      lg: 'text-lg',
    },
  },
  defaultVariants: {
    intent: 'primary',
    size: 'sm',
  },
})

// Usage
<button className={button({ intent: 'secondary', size: 'lg' })}>
```

#### 5. use-debounce

**Debounce state updates:**

```tsx
import { useDebouncedCallback } from 'use-debounce'

const debouncedSave = useDebouncedCallback(
  (value) => save(value),
  500  // Wait 500ms after last call
)
```

#### 6. sonner

**Toast notifications:**

```tsx
import { toast } from 'sonner'

toast.success('Saved!')
toast.error('Failed to save')
toast.loading('Saving...')
```

---

## Technology Comparison Matrix

### Editor Libraries

| Feature | Tiptap | Draft.js | Slate | Quill |
|---------|---------|----------|-------|-------|
| **Learning Curve** | Medium | Medium | Hard | Easy |
| **Extensibility** | Excellent | Good | Excellent | Limited |
| **TypeScript** | First-class | Good | Good | Partial |
| **React** | Native | Native | Native | Wrapper |
| **Mobile** | Good | Poor | Good | Good |
| **Collaborative** | Yes | No | Yes | Limited |
| **Bundle Size** | ~50kb | ~100kb | ~80kb | ~150kb |

### State Management

| Feature | Jotai | Zustand | Redux | Context |
|---------|-------|---------|-------|---------|
| **API Size** | Small | Small | Large | Built-in |
| **Boilerplate** | None | Minimal | High | Medium |
| **DevTools** | Yes | Yes | Yes | Limited |
| **Async** | Excellent | Good | Middleware | Manual |
| **Performance** | Excellent | Excellent | Good | Poor |
| **Learning Curve** | Low | Low | High | Low |

### Build Tools

| Feature | Turborepo | Nx | Lerna | Rush |
|---------|-----------|-----|-------|------|
| **Speed** | Excellent | Excellent | Good | Excellent |
| **Caching** | Yes | Yes | Limited | Yes |
| **Config** | Simple | Complex | Simple | Medium |
| **Ecosystem** | Growing | Large | Mature | Niche |
| **Language** | Rust | TypeScript | JS | Node.js |

---

## What to Learn Next

### For New Developers

**Priority 1: Core Technologies**
1. ✅ Master Tiptap extensions
2. ✅ Understand ProseMirror basics
3. ✅ Learn Jotai state management
4. ✅ Get comfortable with TypeScript

**Priority 2: Framework Skills**
1. ✅ Next.js App Router patterns
2. ✅ React Server Components (for React 19)
3. ✅ Tailwind utility classes
4. ✅ Edge Runtime limitations

### For Contributors

**Deep Dives:**
1. ✅ ProseMirror plugin development
2. ✅ Custom Tiptap NodeViews
3. ✅ Advanced TypeScript (generics, mapped types)
4. ✅ Performance optimization

**Best Practices:**
1. ✅ Extension architecture patterns
2. ✅ Testing strategies
3. ✅ Documentation practices
4. ✅ Accessibility (ARIA, keyboard nav)

### For Integrators

**Essential Knowledge:**
1. ✅ Novel API (EditorRoot, EditorContent)
2. ✅ Extension configuration
3. ✅ Styling with Tailwind
4. ✅ Custom slash commands

**Optional:**
1. ✅ AI integration (if using)
2. ✅ Image uploads (Vercel Blob)
3. ✅ Rate limiting (Upstash)
4. ✅ Analytics (Vercel Analytics)

---

## Summary

You've learned the **complete technology stack** behind Novel:

### Core Technologies
- ✅ **Tiptap** - Extension-based editor
- ✅ **ProseMirror** - Low-level engine
- ✅ **React 18** - UI library
- ✅ **Next.js 15** - Full-stack framework

### State & Styling
- ✅ **Jotai** - Atomic state management
- ✅ **Tailwind CSS** - Utility-first styling
- ✅ **TypeScript** - Type safety

### Build & Quality
- ✅ **Turborepo** - Monorepo orchestration
- ✅ **pnpm** - Package management
- ✅ **Biome** - Linting & formatting

### Integration
- ✅ **Vercel AI SDK** - AI features
- ✅ **Supporting libraries** - Radix, cmdk, etc.

**Next Steps:**
1. [Editor Core Guide](./EDITOR_CORE_GUIDE.md) - Master Tiptap
2. [State Management](./STATE_MANAGEMENT.md) - Deep dive Jotai
3. [How-To Guide](./HOW_TO_GUIDE.md) - Build features

---

**You're now equipped to become a full-stack architect!** 🚀

**Last Updated:** November 19, 2025
**Reading Time:** 2+ hours
**Status:** ✅ Complete comprehensive technology guide
