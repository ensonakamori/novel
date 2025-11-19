# Patterns and Conventions: From Frontend Developer to Full-Stack Architect

> **Target Audience**: Mid-level React developer transitioning to full-stack architecture
>
> **Prerequisites**: React, TypeScript, basic web development
>
> **What You'll Learn**: Industry best practices, architectural thinking, backend fundamentals, and how to reason about entire systems (not just slices)

---

## Table of Contents

### Part 1: Foundation
1. [Introduction: Thinking Like an Architect](#introduction-thinking-like-an-architect)
2. [Naming Conventions](#naming-conventions)
3. [File Organization](#file-organization)

### Part 2: Frontend Patterns
4. [TypeScript Patterns](#typescript-patterns)
5. [React Component Patterns](#react-component-patterns)
6. [State Management Patterns](#state-management-patterns)

### Part 3: Backend & Architecture
7. [Backend API Patterns](#backend-api-patterns)
8. [Full-Stack Architecture Principles](#full-stack-architecture-principles)
9. [Scalability & Maintainability](#scalability--maintainability)

### Part 4: Advanced Concepts
10. [Plugin Architecture](#plugin-architecture)
11. [Performance Patterns](#performance-patterns)
12. [Testing & Quality](#testing--quality)

---

## Introduction: Thinking Like an Architect

### The Shift: From Component Builder to System Designer

As a **mid-level React developer**, you're comfortable building components, managing state, and creating UIs. But to become a **full-stack architect**, you need to think beyond individual components and understand:

1. **How systems communicate** (frontend ↔ backend ↔ database)
2. **How to structure code for teams** (not just for yourself)
3. **How to make decisions that scale** (from 1 user to 1 million users)
4. **How to balance trade-offs** (performance vs. simplicity vs. cost)

### Mental Model Shift

**Frontend Developer Thinks**:
- "How do I build this component?"
- "What state does this UI need?"
- "How do I make this look good?"

**Full-Stack Architect Thinks**:
- "How does this component fit into the larger system?"
- "Where should this state live (client, server, database)?"
- "What happens when 1000 users do this simultaneously?"
- "How will this decision affect the team in 6 months?"

### The Three Layers of Architectural Thinking

```
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3: SYSTEM ARCHITECTURE                               │
│ "How do all the pieces fit together?"                      │
│                                                             │
│ • Frontend (React, Next.js)                                │
│ • Backend (API routes, edge functions)                     │
│ • Database (persistence layer)                             │
│ • External services (AI, storage, auth)                    │
│                                                             │
│ Concerns: Scalability, security, cost, reliability         │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 2: MODULE ARCHITECTURE                               │
│ "How is each part organized internally?"                   │
│                                                             │
│ • Component hierarchy                                      │
│ • State management strategy                                │
│ • API route organization                                   │
│ • Extension/plugin system                                  │
│                                                             │
│ Concerns: Maintainability, testability, reusability        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 1: CODE PATTERNS                                     │
│ "How is individual code written?"                          │
│                                                             │
│ • Naming conventions                                       │
│ • Component patterns                                       │
│ • TypeScript usage                                         │
│ • File organization                                        │
│                                                             │
│ Concerns: Readability, consistency, developer experience   │
└─────────────────────────────────────────────────────────────┘
```

**Key Insight**: Great architects master all three layers. We'll start at Layer 1 and build up to Layer 3.

---

## Naming Conventions

### Why Naming Matters at Scale

When you're the only developer, you can remember what `utils.ts` contains. When you have 10 developers and 100 files, **naming becomes architecture**.

**Bad naming** = Time wasted searching, bugs from confusion, frustration
**Good naming** = Self-documenting code, easier onboarding, fewer bugs

### Component Naming

#### Pattern 1: PascalCase + Feature Prefix

```typescript
// ✅ GOOD: Prefixed by feature/domain
EditorRoot
EditorContent
EditorBubble
EditorBubbleItem
EditorCommand
EditorCommandItem

AISelector
AISelectorCommands
AICompletionCommand

ImageResizer
ImageUploadButton

// ❌ BAD: Generic names without context
Root          // Root of what?
Content       // What content?
Bubble        // Bubble for what?
Selector      // Selects what?
```

**Why Prefix?**

1. **Namespace isolation**: Prevents naming conflicts
2. **Autocomplete grouping**: Type "Editor" and see all related components
3. **Mental model**: Components grouped by feature in your mind
4. **Refactoring safety**: Easy to find all components of a feature

**Architectural Thinking**: In a large app, you might have `EditorBubble`, `CommentBubble`, `NotificationBubble`. Prefixes create clear module boundaries.

---

#### Pattern 2: Suffix for Component Type

```typescript
// ✅ GOOD: Suffix indicates role
ColorSelector      // Selects colors
LinkSelector       // Selects/edits links
NodeSelector       // Selects node types

EditorCommandItem  // Item in command list
EditorCommandEmpty // Empty state component

ImageResizer       // Resizes images
ImageUploadButton  // Button for uploads

// ❌ BAD: Unclear purpose
Color              // Is this a component? A type? A constant?
Link               // Too generic
Node               // Conflicts with DOM Node
```

**Common Suffixes**:
- `Selector` - Choosing from options (dropdown, menu)
- `Item` - List/collection member
- `Empty` - Empty state component
- `Provider` - Context provider
- `Hook` - Custom hook (use prefix: `useLocalStorage`)
- `Button` - Action trigger
- `Modal` / `Dialog` - Overlay UI
- `Form` - Data entry

---

### File Naming

#### Pattern: kebab-case (lowercase with hyphens)

```
components/
├── editor-bubble.tsx           ✅ Matches: EditorBubble
├── editor-bubble-item.tsx      ✅ Matches: EditorBubbleItem
├── editor-command.tsx          ✅ Matches: EditorCommand
├── editor-command-item.tsx     ✅ Matches: EditorCommandItem
├── color-selector.tsx          ✅ Matches: ColorSelector
└── ai-highlight.ts             ✅ Matches: AIHighlight extension

api/
├── generate/
│   └── route.ts                ✅ Next.js convention
└── upload/
    └── route.ts                ✅ Next.js convention

utils/
├── atoms.ts                    ✅ Exports multiple atoms
├── store.ts                    ✅ Single store export
└── index.ts                    ✅ Exports helpers
```

**Why kebab-case?**

1. **Unix/Linux friendly**: No escaping needed in terminal
2. **Case-insensitive filesystems**: Works on macOS, Windows, Linux
3. **URL friendly**: `editor-bubble` works in URLs without encoding
4. **Easier to type**: No shift key needed
5. **Industry standard**: React ecosystem convention

**Architectural Principle**: File names are part of your API. They appear in imports, error messages, and build outputs. Choose a convention and stick to it religiously.

---

### Variable & Function Naming

#### Pattern: camelCase + Descriptive Names

```typescript
// ✅ GOOD: Clear purpose
const debouncedUpdates = useDebouncedCallback(...);
const highlightCodeblocks = (content: string) => ...;
const isValidTwitterUrl = (url: string) => boolean;
const createImageUpload = (options: ImageUploadOptions) => ...;

// State variables
const [saveStatus, setSaveStatus] = useState("Saved");
const [openNode, setOpenNode] = useState(false);
const [initialContent, setInitialContent] = useState(null);

// Event handlers
const handleImageDrop = (view, event, moved, uploadFn) => ...;
const handleCommandNavigation = (event) => ...;

// Refs
const instanceRef = useRef<Instance<Props> | null>(null);
const tunnelInstance = useRef(tunnel()).current;

// ❌ BAD: Unclear purpose
const data = ...;           // What data?
const temp = ...;           // Temporary what?
const result = ...;         // Result of what?
const handleClick = ...;    // Handles click on what?
const x = ...;              // Meaningless
```

**Naming Prefixes**:

| Prefix | Meaning | Example |
|--------|---------|---------|
| `is` / `has` / `can` | Boolean | `isValidUrl`, `hasSelection`, `canEdit` |
| `get` | Retrieves value | `getJSON`, `getHTML`, `getPrevText` |
| `set` | Updates value | `setContent`, `setSelection` |
| `handle` | Event handler | `handleDrop`, `handlePaste`, `handleClick` |
| `create` | Factory function | `createImageUpload`, `createSuggestionItems` |
| `use` | Custom hook | `useEditor`, `useLocalStorage`, `useDebounce` |
| `on` | Callback prop | `onUpdate`, `onCreate`, `onCommand` |

**Why This Matters**: 6 months from now, someone (maybe you!) needs to debug `handleImageDrop`. A good name tells you exactly what it does without reading the code.

---

### TypeScript Type Naming

#### Pattern 1: Interfaces End with "Props"

```typescript
// ✅ GOOD: Clear that these are component props
export interface EditorRootProps {
  readonly children: ReactNode;
}

export interface EditorBubbleProps extends Omit<BubbleMenuProps, "editor"> {
  readonly children: ReactNode;
  readonly tippyOptions?: Partial<Props>;
}

interface LinkSelectorProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

// ❌ BAD: Unclear purpose
interface Editor {
  children: ReactNode;  // Is this the editor component? Props? State?
}

interface Bubble {
  // Bubble what?
}
```

---

#### Pattern 2: Options End with "Options"

```typescript
// ✅ GOOD: Configuration objects
export interface AIHighlightOptions {
  HTMLAttributes: Record<string, string>;
}

export interface ImageUploadOptions {
  validateFn?: (file: File) => boolean;
  onUpload: (file: File) => Promise<string>;
}

// ❌ BAD: Unclear purpose
interface AIHighlight {
  HTMLAttributes: Record<string, string>;  // Is this the extension? Config?
}
```

---

#### Pattern 3: Data Types Are Descriptive Nouns

```typescript
// ✅ GOOD: Clear data structures
export interface SuggestionItem {
  title: string;
  description: string;
  icon: ReactNode;
  searchTerms?: string[];
  command?: (props: { editor: Editor; range: Range }) => void;
}

export type SelectorItem = {
  name: string;
  icon: LucideIcon;
  command: (editor: Editor) => void;
  isActive: (editor: Editor) => boolean;
};

type UploadFn = (
  file: File,
  view: EditorView,
  pos: number,
) => void;

// ❌ BAD: Vague names
type Item {  // Item of what?
  title: string;
}

type Fn = (...args: any[]) => void;  // Function that does what?
```

---

### Constants Naming

```typescript
// ✅ GOOD: UPPER_SNAKE_CASE for true constants
const TWITTER_REGEX = /^https?:\/\/(twitter\.com|x\.com)\/(?:#!\/)?(\w+)\/status(?:es)?\/(\d+)/;
const TWITTER_REGEX_GLOBAL = /https?:\/\/(twitter\.com|x\.com)\/(?:#!\/)?(\w+)\/status(?:es)?\/(\d+)/g;

const TEXT_COLORS = [
  { name: "Default", color: "var(--novel-black)" },
  { name: "Purple", color: "#9333EA" },
  // ...
];

// ✅ GOOD: camelCase for config objects
const defaultExtensions = [
  StarterKit,
  Placeholder,
  // ...
];

const suggestionItems = createSuggestionItems([
  { title: "Text", /* ... */ },
  { title: "Heading 1", /* ... */ },
]);

// ❌ BAD: Inconsistent
const twitterRegex = /.../ ;  // Should be TWITTER_REGEX
const TEXT_COLORS = [...];    // Okay, but prefer camelCase for arrays of objects
```

**Rule of Thumb**:
- **UPPER_SNAKE_CASE**: Primitive constants (numbers, strings, regex)
- **camelCase**: Objects, arrays, functions
- **PascalCase**: Classes, components, types

---

## File Organization

### The Architecture of Files: From Chaos to System

**Frontend Developer Approach**:
```
src/
├── components/
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Modal.tsx
│   └── ... 50 more files ...
├── utils/
│   └── helpers.ts  // Everything goes here
└── App.tsx
```

**Problem**: Works for 10 files. Breaks at 100 files.

**Full-Stack Architect Approach**:
```
src/
├── features/
│   ├── editor/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── index.ts
│   ├── comments/
│   │   ├── components/
│   │   ├── api/
│   │   └── index.ts
│   └── auth/
│       ├── components/
│       ├── api/
│       └── index.ts
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── App.tsx
```

**Why This Works**: Features are isolated. When you work on comments, you never touch editor code. When you delete comments feature, delete one folder.

---

### Novel's File Organization Strategy

#### Pattern 1: Package by Feature (Monorepo)

```
novel/
├── packages/
│   └── headless/              # Core editor logic (no styling)
│       ├── src/
│       │   ├── components/    # React components
│       │   ├── extensions/    # Tiptap extensions
│       │   ├── plugins/       # ProseMirror plugins
│       │   ├── utils/         # Pure functions
│       │   └── index.ts       # Public API
│       └── package.json
│
├── apps/
│   └── web/                   # Demo application
│       ├── app/               # Next.js app
│       │   ├── api/           # API routes
│       │   │   ├── generate/  # AI generation
│       │   │   └── upload/    # Image upload
│       │   ├── layout.tsx     # Root layout
│       │   └── page.tsx       # Home page
│       ├── components/
│       │   └── tailwind/      # Styled components
│       │       ├── generative/  # AI features
│       │       ├── selectors/   # UI controls
│       │       └── ui/          # Base components
│       └── lib/               # App utilities
│
└── package.json               # Workspace root
```

**Architectural Decisions**:

1. **headless package**: Reusable across different apps
2. **web app**: One specific implementation
3. **Separation**: Logic (headless) vs. Presentation (web)

**Why Monorepo?**
- ✅ Share code between packages
- ✅ Atomic commits across packages
- ✅ Unified tooling (TypeScript, linting, testing)
- ❌ More complex setup (but worth it at scale)

---

#### Pattern 2: Co-location of Related Files

```
components/
├── editor.tsx                 # Main component
├── editor-bubble.tsx          # Related to editor
├── editor-bubble-item.tsx     # Sub-component of bubble
├── editor-command.tsx         # Related to editor
├── editor-command-item.tsx    # Sub-component of command
└── index.ts                   # Exports all

selectors/
├── color-selector.tsx
├── link-selector.tsx
├── node-selector.tsx
├── math-selector.tsx
└── text-buttons.tsx

generative/
├── ai-selector.tsx
├── ai-selector-commands.tsx
├── ai-completion-command.tsx
└── generative-menu-switch.tsx
```

**Why Co-locate?**
- ✅ Related code is physically close
- ✅ Easy to understand feature boundaries
- ✅ Refactoring is moving a folder, not hunting files
- ✅ Delete feature = delete folder

**Contrast with Type-Based Organization**:

```
❌ BAD: Organized by file type
src/
├── components/
│   ├── Editor.tsx
│   ├── Bubble.tsx
│   ├── Command.tsx
│   ├── ColorSelector.tsx
│   └── ... 50 more ...
├── hooks/
│   ├── useEditor.ts
│   ├── useLocalStorage.ts
│   └── ...
└── utils/
    ├── editorUtils.ts
    ├── colorUtils.ts
    └── ...

// Problem: To understand "editor", you jump between 3 directories
```

---

#### Pattern 3: Barrel Exports (index.ts)

```typescript
// packages/headless/src/components/index.ts
export { useCurrentEditor as useEditor } from "@tiptap/react";
export { type Editor as EditorInstance } from "@tiptap/core";
export type { JSONContent } from "@tiptap/react";

export { EditorRoot, EditorContent, type EditorContentProps } from "./editor";
export { EditorBubble } from "./editor-bubble";
export { EditorBubbleItem } from "./editor-bubble-item";
export { EditorCommand, EditorCommandList } from "./editor-command";
export { EditorCommandItem, EditorCommandEmpty } from "./editor-command-item";
```

**Usage**:
```typescript
// ✅ GOOD: Import from barrel
import { EditorRoot, EditorContent, EditorBubble } from "novel";

// ❌ BAD: Import from deep paths
import { EditorRoot } from "novel/dist/components/editor";
import { EditorContent } from "novel/dist/components/editor";
import { EditorBubble } from "novel/dist/components/editor-bubble";
```

**Why Barrel Exports?**

1. **Clean API**: Hide internal structure
2. **Refactoring safe**: Can reorganize files without breaking imports
3. **Type exports**: Export types alongside values
4. **Single import point**: Better tree-shaking
5. **Versioning**: Can deprecate exports without breaking old code

**Architectural Thinking**: Barrel exports are your **public API**. Internal files can change, but the barrel export stays stable. This is how libraries version their APIs.

---

#### Pattern 4: Separation of Concerns

```
headless/                      # PURE LOGIC (no styling)
├── components/                # React components (unstyled)
├── extensions/                # Tiptap extensions (behavior only)
├── plugins/                   # ProseMirror plugins
└── utils/                     # Pure functions (no side effects)

web/                           # PRESENTATION (styling + composition)
├── components/tailwind/       # Styled implementations
├── styles/                    # CSS files
└── lib/                       # App-specific logic
```

**Why Separate?**

| Concern | Where | Why |
|---------|-------|-----|
| **Editor logic** | headless | Reusable, testable, framework-agnostic |
| **Styling** | web | App-specific, easy to theme |
| **Business logic** | web/lib | App-specific rules |
| **API routes** | web/app/api | Backend services |

**Mental Model**: Think of headless as a **library** you publish to npm. It must work for anyone (Tailwind, CSS Modules, Emotion, etc.). Web app is **one consumer** of that library.

---

### Import Organization

#### Pattern: Ordered by Scope (External → Internal → Relative)

```typescript
// 1. External dependencies (React, third-party)
import { BubbleMenu, useCurrentEditor } from "@tiptap/react";
import type { BubbleMenuProps } from "@tiptap/react";
import { forwardRef, useEffect, useMemo, useRef } from "react";
import type { ReactNode } from "react";
import type { Instance, Props } from "tippy.js";

// 2. Internal package imports (from alias like @/, novel)
import { defaultEditorContent } from "@/lib/content";
import {
  EditorCommand,
  EditorContent,
  type EditorInstance,
  EditorRoot,
} from "novel";

// 3. Relative imports (./...)
import { defaultExtensions } from "./extensions";
import { ColorSelector } from "./selectors/color-selector";
import { LinkSelector } from "./selectors/link-selector";
```

**Why This Order?**

1. **External first**: Shows what third-party dependencies exist
2. **Internal next**: Shows app/package dependencies
3. **Relative last**: Shows immediate neighbors
4. **Type imports separate**: Better tree-shaking, faster builds

**Architectural Benefit**: During code review, you can quickly see:
- "Oh, this file added a new dependency on `lodash`" (external)
- "This file now depends on the auth module" (internal)
- "This file uses these local helpers" (relative)

---

#### Pattern: Type-Only Imports

```typescript
// ✅ GOOD: Separate type imports
import { BubbleMenu } from "@tiptap/react";
import type { BubbleMenuProps } from "@tiptap/react";

import { useState } from "react";
import type { ReactNode } from "react";

// ❌ BAD: Mixed imports
import { BubbleMenu, BubbleMenuProps } from "@tiptap/react";  // BubbleMenuProps is type
import { useState, ReactNode } from "react";  // ReactNode is type
```

**Why Separate?**

1. **Build optimization**: Type imports are removed at compile time
2. **Clarity**: Instantly see what's a value vs. type
3. **Tree-shaking**: Bundlers can optimize better
4. **TypeScript compilation**: Faster type checking

**Architectural Principle**: Your import statements document dependencies. Clean imports = clear dependencies = maintainable architecture.

---

### Absolute vs Relative Imports

```typescript
// apps/web/tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}

// Usage
import { cn } from "@/lib/utils";                    // ✅ Absolute (cross-directory)
import { Button } from "@/components/ui/button";     // ✅ Absolute (cross-directory)

import { ColorSelector } from "./selectors/color-selector";  // ✅ Relative (same feature)
import { defaultExtensions } from "./extensions";            // ✅ Relative (sibling file)
```

**Rule of Thumb**:
- **Absolute imports** (`@/`): Crossing module boundaries, going up many levels
- **Relative imports** (`./`, `../`): Within the same feature/module

**Why This Matters**:
- Absolute paths don't break when moving files
- Relative paths show immediate relationships
- Too many `../../../` signals wrong file organization

**Architectural Thinking**: If you're writing `../../../../utils/helper`, your architecture is wrong. That helper should probably live closer to where it's used, or be in a shared utils folder accessible via `@/utils`.

---

## Part 1 Summary

You've learned:

✅ **Naming conventions** that scale from 1 developer to 100 developers
✅ **File organization** that separates concerns and enables team collaboration
✅ **Import patterns** that document dependencies and enable refactoring
✅ **Architectural thinking** about how decisions impact long-term maintainability

**Key Mindset Shift**: You're no longer just writing code that works. You're writing code that:
- Other developers can understand in 6 months
- Can be refactored without breaking everything
- Scales from 10 files to 1,000 files
- Communicates intent through structure, not just comments

**Next**: We'll dive into TypeScript patterns and React component patterns that enable robust, maintainable applications.

---

---

## TypeScript Patterns

### The Power of Types: From Runtime Bugs to Compile-Time Safety

**Frontend Developer Mindset**: "TypeScript is just JavaScript with types"
**Architect Mindset**: "TypeScript is a design tool that encodes business rules and prevents entire classes of bugs"

### Type vs Interface: When to Use Each

```typescript
// ✅ Use INTERFACE for:

// 1. Component props (extendable)
export interface EditorBubbleProps extends Omit<BubbleMenuProps, "editor"> {
  readonly children: ReactNode;
}

// 2. Object shapes that might be extended
export interface AIHighlightOptions {
  HTMLAttributes: Record<string, string>;
}

// 3. Public APIs (libraries)
export interface EditorInstance {
  commands: Commands;
  chain: () => ChainedCommands;
  getHTML: () => string;
  // ...
}
```

```typescript
// ✅ Use TYPE for:

// 1. Unions
export type EditorContentProps = Omit<EditorProviderProps, "content"> & {
  readonly children?: ReactNode;
  readonly className?: string;
};

// 2. Intersections
export type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean;
  };

// 3. Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 4. Function signatures
type UploadFn = (file: File, view: EditorView, pos: number) => void;

// 5. Utility type compositions
type SelectorItem = {
  name: string;
  icon: LucideIcon;
  command: (editor: ReturnType<typeof useEditor>["editor"]) => void;
  isActive: (editor: ReturnType<typeof useEditor>["editor"]) => boolean;
};
```

**Rule of Thumb**:
- **Interface**: When you need extensibility (other code might `extends` it)
- **Type**: When you need flexibility (unions, intersections, complex types)
- **Consistency**: Pick one for similar use cases across your codebase

**Architectural Decision**: Novel uses **interfaces for props** (React convention) and **types for complex compositions** (flexibility). This is an industry-standard pattern.

---

###

 Generic Types: Writing Reusable Code

**File**: `packages/headless/hooks/use-local-storage.ts`

```typescript
// ✅ EXCELLENT: Generic hook that works with any type
const useLocalStorage = <T>(
  key: string,
  initialValue: T,
): [T, (value: T) => void] => {
  const [storedValue, setStoredValue] = useState(initialValue);

  useEffect(() => {
    const item = window.localStorage.getItem(key);
    if (item) {
      setStoredValue(JSON.parse(item));
    }
  }, [key]);

  const setValue = (value: T) => {
    setStoredValue(value);
    window.localStorage.setItem(key, JSON.stringify(value));
  };

  return [storedValue, setValue];
};

// Usage - Type is inferred automatically
const [user, setUser] = useLocalStorage<User>("user", null);  // T = User | null
const [theme, setTheme] = useLocalStorage("theme", "dark");   // T = string (inferred)
const [settings, setSettings] = useLocalStorage<Settings>("settings", defaultSettings);
```

**Why Generics Matter**:

1. **Type Safety**: TypeScript knows `user` is `User | null`, not `any`
2. **Reusability**: One function works for strings, objects, arrays, anything
3. **Inference**: Often don't need to specify `<T>`, TypeScript figures it out
4. **DRY**: No need to write `useUserStorage`, `useThemeStorage`, etc.

**Backend Analogy**: Like database ORMs. You write one `findById<T>()` that works for any model, not separate `findUserById()`, `findPostById()`, etc.

---

### Utility Types: Leverage TypeScript's Power

```typescript
// 1. Omit - Remove properties
export interface EditorBubbleProps extends Omit<BubbleMenuProps, "editor"> {
  readonly children: ReactNode;
}
// Why? BubbleMenuProps has 'editor', but we provide it internally

// 2. Pick - Select specific properties
type UserBasicInfo = Pick<User, "id" | "name" | "email">;
// Why? Don't expose password, internalNotes, etc.

// 3. Partial - Make all properties optional
type PartialUser = Partial<User>;
// Why? For update operations where you only change some fields

// 4. Required - Make all properties required
type RequiredSettings = Required<Settings>;
// Why? Ensure all config is provided before initialization

// 5. Readonly - Make all properties readonly
interface EditorProps {
  readonly children: ReactNode;  // Can't be reassigned
}
// Why? Props should never be mutated in React

// 6. Record - Create object type with specific keys/values
type ErrorMessages = Record<string, string>;
const errors: ErrorMessages = {
  required: "This field is required",
  email: "Invalid email format",
};

// 7. ReturnType - Extract function return type
type EditorType = ReturnType<typeof useEditor>["editor"];
// Why? Don't duplicate the type, derive it from source

// 8. ComponentPropsWithoutRef - Get component props without ref
export const EditorCommand = forwardRef<
  HTMLDivElement,
  ComponentPropsWithoutRef<typeof Command>
>(...);
// Why? Integrates with third-party component types

// 9. VariantProps - Extract variant types (from CVA)
export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}
// Why? Type-safe variant props from CVA config
```

**Architectural Principle**: Don't Repeat Types. Derive types from source of truth using utility types. When implementation changes, types update automatically.

---

### readonly Props: Immutability by Design

```typescript
// ✅ GOOD: All props are readonly
export interface EditorProps {
  readonly children: ReactNode;
  readonly className?: string;
}

interface LinkSelectorProps {
  readonly open: boolean;
  readonly onOpenChange: (open: boolean) => void;
}

// ❌ BAD: Mutable props (TypeScript won't prevent mutation)
interface EditorProps {
  children: ReactNode;
  className?: string;
}

// Component tries to mutate (TypeScript error with readonly)
const MyComponent = ({ className }: EditorProps) => {
  className = "new-class";  // ❌ Error: Cannot assign to 'className' because it is a read-only property
};
```

**Why readonly?**

1. **React Philosophy**: Props flow down, never modified by child
2. **Debugging**: Mutation bugs are harder to trace than reassignment bugs
3. **Refactoring**: Confident that props aren't changed unexpectedly
4. **Team Safety**: Junior developers can't accidentally mutate props

**Backend Analogy**: Like marking database fields as `NOT NULL` or `UNIQUE`. Enforce constraints at the type level, not runtime.

---

### Module Augmentation: Extending Third-Party Types

```typescript
// File: extensions/ai-highlight.ts
// Declare new commands in Tiptap's type system
declare module "@tiptap/core" {
  interface Commands<ReturnType> {
    AIHighlight: {
      setAIHighlight: (attributes?: { color: string }) => ReturnType;
      toggleAIHighlight: (attributes?: { color: string }) => ReturnType;
      unsetAIHighlight: () => ReturnType;
    };
  }
}

// Now TypeScript knows about these commands
editor.commands.setAIHighlight({ color: "#yellow" });  // ✅ Type-safe
editor.commands.toggleAIHighlight();                    // ✅ Type-safe
editor.commands.unknownCommand();                        // ❌ Error: Property doesn't exist
```

**Why Module Augmentation?**

1. **Type Safety**: Autocomplete suggests your custom commands
2. **Documentation**: Types document what commands exist
3. **Refactoring**: Rename command, TypeScript finds all usages
4. **Plugin Pattern**: Each plugin declares its own API

**Architectural Principle**: When extending libraries, extend their types too. Type system should reflect runtime reality.

---

## React Component Patterns

### Pattern 1: Compound Components

**What**: Components designed to work together, sharing implicit state via context.

**File**: `packages/headless/src/components/editor.tsx`

```typescript
// Parent: Provides context
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

// Child: Consumes context (no explicit prop passing)
export const EditorContent = forwardRef<HTMLDivElement, EditorContentProps>(
  ({ className, children, initialContent, ...rest }, ref) => (
    <div ref={ref} className={className}>
      <EditorProvider {...rest} content={initialContent}>
        {children}
      </EditorProvider>
    </div>
  ),
);

// Usage
<EditorRoot>                    {/* Provides state */}
  <EditorContent {...props}>    {/* Consumes state */}
    <EditorCommand />            {/* Also consumes state */}
  </EditorContent>
</EditorRoot>
```

**Why Compound Components?**

1. **Flexible Composition**: Users control structure and layout
2. **Implicit Communication**: No prop drilling through every level
3. **Clear Hierarchy**: Visual structure matches data flow
4. **API Design**: Mirrors how developers think ("Root provides, children consume")

**Real-World Analogy**: Like HTML `<select>` and `<option>`. You compose them, browser handles the relationship.

```html
<select>                     <!-- Provides context -->
  <option value="1">One</option>    <!-- Consumes context -->
  <option value="2">Two</option>    <!-- Consumes context -->
</select>
```

**Backend Analogy**: Middleware stack. Each layer wraps the next, providing context (auth, logging, etc.).

---

### Pattern 2: forwardRef for DOM Access

**Problem**: Parent components need to access child's DOM node (for focus, scroll, measurements).

**Solution**: `forwardRef`

```typescript
// ✅ GOOD: Forwards ref to underlying DOM
export const EditorBubble = forwardRef<HTMLDivElement, EditorBubbleProps>(
  ({ children, tippyOptions, ...rest }, ref) => {
    return (
      <div ref={ref}>
        <BubbleMenu {...props}>{children}</BubbleMenu>
      </div>
    );
  },
);

EditorBubble.displayName = "EditorBubble";  // For React DevTools

// Usage - Parent can access DOM
const MyParent = () => {
  const bubbleRef = useRef<HTMLDivElement>(null);

  const scrollToBubble = () => {
    bubbleRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  return (
    <div>
      <button onClick={scrollToBubble}>Scroll to Bubble</button>
      <EditorBubble ref={bubbleRef}>...</EditorBubble>
    </div>
  );
};
```

**Why forwardRef?**

1. **Imperative Operations**: Focus input, measure size, scroll, etc.
2. **Library Integration**: Radix UI, Tippy.js require refs
3. **Testing**: Easier to query DOM in tests
4. **Animation**: GSAP, Framer Motion need DOM access

**When to Use**:
- ✅ UI component libraries (buttons, inputs, modals)
- ✅ Third-party integrations (popovers, tooltips)
- ❌ Business logic components (no DOM manipulation needed)

---

### Pattern 3: Polymorphic Components (asChild)

**Problem**: Want button behavior, but render as `<a>`, `<Link>`, or any element.

**Solution**: Radix UI's `Slot` component + `asChild` prop

```typescript
// File: components/ui/button.tsx
import { Slot } from "@radix-ui/react-slot";

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    //         ↑               ↑         ↑
    //    If true      Use Slot   Else button

    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  },
);

// Usage 1: Regular button
<Button>Click Me</Button>
// Renders: <button class="...">Click Me</button>

// Usage 2: Button styled as link
<Button asChild>
  <a href="/home">Go Home</a>
</Button>
// Renders: <a href="/home" class="...">Go Home</a>

// Usage 3: Button styled as Next.js Link
<Button asChild>
  <Link href="/dashboard">Dashboard</Link>
</Button>
// Renders: Next.js Link with button styles
```

**Why asChild?**

1. **No Wrapper Divs**: Direct rendering, clean DOM
2. **Semantic HTML**: Use correct element (`<a>` for links, `<button>` for actions)
3. **Accessibility**: Proper ARIA, keyboard navigation
4. **Flexibility**: One component, many use cases

**How Slot Works**:
```typescript
// Slot merges props onto child
<Slot className="button-styles" onClick={handler}>
  <a href="/home">Home</a>
</Slot>

// Becomes:
<a href="/home" className="button-styles" onClick={handler}>
  Home
</a>
```

**Architectural Principle**: Composition over inheritance. Don't create `<LinkButton>`, `<IconButton>`, etc. Use `asChild` for flexibility.

---

### Pattern 4: Props Spreading with Rest

```typescript
// File: editor-bubble-item.tsx
export const EditorBubbleItem = forwardRef<
  HTMLDivElement,
  EditorBubbleItemProps & Omit<ComponentPropsWithoutRef<"div">, "onSelect">
>(({ children, asChild, onSelect, ...rest }, ref) => {
  //                                          ↑
  //                                    Capture all other props
  const { editor } = useCurrentEditor();
  const Comp = asChild ? Slot : "div";

  if (!editor) return null;

  return (
    <Comp ref={ref} {...rest} onClick={() => onSelect?.(editor)}>
      {/*            ↑ Spread remaining props */}
      {children}
    </Comp>
  );
});
```

**Why Spread Props?**

1. **Flexibility**: Supports all native HTML attributes (`aria-*`, `data-*`, etc.)
2. **TypeScript Safety**: Type system ensures only valid props pass through
3. **Future-Proof**: New HTML attributes work automatically
4. **DRY**: Don't manually list 50+ possible props

**Pattern Breakdown**:
```typescript
({ children, asChild, onSelect, ...rest }, ref)
//  ↑         ↑         ↑          ↑
//  Extract   Extract   Extract    Everything else

<Comp ref={ref} {...rest} onClick={...}>
//               ↑ Spread the rest
```

**Caution**: Order matters with spreading:

```typescript
// ✅ GOOD: Spread first, then override
<Comp {...rest} onClick={handleClick} />
// User's onClick gets overridden

// ❌ BAD: Override first, then spread
<Comp onClick={handleClick} {...rest} />
// User's onClick overwrites yours
```

---

### Pattern 5: Custom Hooks for Logic Reuse

**File**: `apps/web/hooks/use-local-storage.ts`

```typescript
const useLocalStorage = <T>(
  key: string,
  initialValue: T,
): [T, (value: T) => void] => {
  const [storedValue, setStoredValue] = useState(initialValue);

  // Read from localStorage on mount
  useEffect(() => {
    const item = window.localStorage.getItem(key);
    if (item) {
      setStoredValue(JSON.parse(item));
    }
  }, [key]);

  // Write to localStorage on change
  const setValue = (value: T) => {
    setStoredValue(value);
    window.localStorage.setItem(key, JSON.stringify(value));
  };

  return [storedValue, setValue];
};

// Usage - Just like useState, but persisted
const [theme, setTheme] = useLocalStorage("theme", "dark");
const [user, setUser] = useLocalStorage<User | null>("user", null);
```

**Why Custom Hooks?**

1. **Encapsulation**: Hide complexity (localStorage API, JSON parsing)
2. **Reusability**: Use anywhere without code duplication
3. **Testing**: Test hook in isolation
4. **Composition**: Combine hooks to build complex behaviors

**Hook Naming Convention**: Always start with `use` (React requirement)

```typescript
// ✅ GOOD
useLocalStorage
useDebounce
useEditor
useCurrentUser

// ❌ BAD
localStorageHook  // Missing 'use' prefix
getLocalStorage   // Not a hook, it's a function
```

**Advanced Hook Example** (from Novel):

```typescript
// File: advanced-editor.tsx
const debouncedUpdates = useDebouncedCallback(
  async (editor: EditorInstance) => {
    const json = editor.getJSON();
    setCharsCount(editor.storage.characterCount.words());

    // Save in multiple formats
    window.localStorage.setItem("html-content", highlightCodeblocks(editor.getHTML()));
    window.localStorage.setItem("novel-content", JSON.stringify(json));
    window.localStorage.setItem("markdown", editor.storage.markdown.getMarkdown());

    setSaveStatus("Saved");
  },
  500  // Debounce 500ms
);

// Usage in onUpdate
onUpdate={({ editor }) => {
  debouncedUpdates(editor);
  setSaveStatus("Unsaved");
}}
```

**Architectural Benefit**: Hooks let you extract cross-cutting concerns (persistence, debouncing, subscriptions) from components. Components stay focused on rendering.

---

### Pattern 6: Performance Optimization with useMemo

**File**: `packages/headless/src/components/editor-bubble.tsx`

```typescript
const bubbleMenuProps: Omit<BubbleMenuProps, "children"> = useMemo(() => {
  const shouldShow: BubbleMenuProps["shouldShow"] = ({ editor, state }) => {
    const { selection } = state;
    const { empty } = selection;

    // Don't show if editor not editable, image selected, etc.
    if (!editor.isEditable || editor.isActive("image") || empty || isNodeSelection(selection)) {
      return false;
    }
    return true;
  };

  return {
    shouldShow,
    tippyOptions: {
      onCreate: (val) => {
        instanceRef.current = val;
        // ... setup
      },
      moveTransition: "transform 0.15s ease-out",
      ...tippyOptions,
    },
    editor: currentEditor,
    ...rest,
  };
}, [rest, tippyOptions, currentEditor]);
//  ↑ Only recompute when these change
```

**When to use useMemo**:

1. **Expensive Calculations**: Complex computations, large data transformations
2. **Reference Equality**: Passing objects/arrays to child components (prevent re-renders)
3. **useEffect Dependencies**: Avoid unnecessary effect runs

**When NOT to use useMemo**:

1. **Simple Values**: `const sum = a + b` (overhead > benefit)
2. **Premature Optimization**: Profile first, optimize later
3. **Everywhere**: Adds complexity, only use when needed

**Mental Model**: useMemo is like caching. Cache expensive operations, not cheap ones.

```typescript
// ❌ BAD: Overhead > benefit
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);

// ✅ GOOD: Prevents object re-creation
const config = useMemo(() => ({
  apiKey: process.env.API_KEY,
  endpoint: "https://api.example.com",
  retries: 3,
}), []);

// Without useMemo, new object every render:
const config = { apiKey: ..., endpoint: ..., retries: 3 };
// With useMemo, same object reference across renders
```

**Backend Analogy**: Like database query caching. Cache slow queries, not `SELECT 1`.

---

## State Management Patterns

### The State Hierarchy: Where Should State Live?

**Frontend Developer Question**: "Where do I put this state?"
**Architect Answer**: "Depends on who needs it and how long it should live."

### State Classification

```
┌─────────────────────────────────────────────────────────┐
│ LEVEL 5: DATABASE STATE                                │
│ Examples: User profiles, blog posts, comments          │
│ Lifetime: Forever (until deleted)                      │
│ Managed by: Backend database (PostgreSQL, MongoDB)     │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LEVEL 4: SERVER STATE                                  │
│ Examples: Cached API responses, auth tokens            │
│ Lifetime: Session or TTL                               │
│ Managed by: React Query, SWR, or manual fetch          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LEVEL 3: GLOBAL CLIENT STATE                           │
│ Examples: Current user, theme, global modals           │
│ Lifetime: Entire app session                           │
│ Managed by: Jotai, Zustand, Redux, Context             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LEVEL 2: SHARED COMPONENT STATE                        │
│ Examples: Form state, modal visibility                 │
│ Lifetime: Component tree section                       │
│ Managed by: React Context, lifted state                │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ LEVEL 1: LOCAL COMPONENT STATE                         │
│ Examples: Input value, hover state, local toggle       │
│ Lifetime: Single component                             │
│ Managed by: useState, useReducer                       │
└─────────────────────────────────────────────────────────┘
```

**Decision Tree**:

```
Does it need to persist across page refreshes?
├─ YES → Database State (Level 5)
└─ NO → Continue...
    Does it come from an API?
    ├─ YES → Server State (Level 4)
    └─ NO → Continue...
        Do multiple unrelated components need it?
        ├─ YES → Global Client State (Level 3)
        └─ NO → Continue...
            Do sibling components need it?
            ├─ YES → Shared Component State (Level 2)
            └─ NO → Local Component State (Level 1)
```

---

### Novel's State Strategy

**1. Editor Content State** → **Editor.state** (Tiptap/ProseMirror)

```typescript
// ✅ GOOD: Editor content in editor state
editor.getJSON();    // Get content
editor.getHTML();
editor.commands.setContent(json);  // Set content

// ❌ BAD: Duplicating in React state
const [content, setContent] = useState(editor.getJSON());  // DON'T
```

**Why**: Editor state is optimized for rich text (undo/redo, transactions, collaboration).

---

**2. UI State** → **Jotai Atoms**

```typescript
// File: utils/atoms.ts
export const queryAtom = atom("");            // Slash command query
export const rangeAtom = atom<Range | null>(null);  // Cursor position

// Usage
const [query, setQuery] = useAtom(queryAtom);
const range = useAtomValue(rangeAtom);
```

**Why**: Lightweight, atomic, isolated per editor instance.

---

**3. Local UI State** → **useState**

```typescript
// File: advanced-editor.tsx
const [saveStatus, setSaveStatus] = useState("Saved");
const [openNode, setOpenNode] = useState(false);
const [openColor, setOpenColor] = useState(false);
```

**Why**: Component-specific, doesn't need sharing.

---

**4. Persisted State** → **localStorage** (via custom hook)

```typescript
const [font, setFont] = useLocalStorage("novel__font", "Default");
```

**Why**: Survives page refresh, user preference.

---

### State Anti-Patterns to Avoid

```typescript
// ❌ ANTI-PATTERN 1: Prop Drilling
<GrandParent theme={theme}>
  <Parent theme={theme}>
    <Child theme={theme}>
      <GrandChild theme={theme} />  // 4 levels deep!
    </Child>
  </Parent>
</GrandParent>

// ✅ SOLUTION: Context or Global State
<ThemeProvider theme={theme}>
  <GrandParent>
    <Parent>
      <Child>
        <GrandChild />  // Reads from context
      </Child>
    </Parent>
  </GrandParent>
</ThemeProvider>
```

```typescript
// ❌ ANTI-PATTERN 2: Derived State
const [count, setCount] = useState(0);
const [doubleCount, setDoubleCount] = useState(0);  // BAD: Duplicate source of truth

useEffect(() => {
  setDoubleCount(count * 2);  // Keeping in sync is error-prone
}, [count]);

// ✅ SOLUTION: Compute on render
const [count, setCount] = useState(0);
const doubleCount = count * 2;  // Derived from source of truth
```

```typescript
// ❌ ANTI-PATTERN 3: Stale Closures
const MyComponent = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setCount(count + 1);  // BAD: Closes over stale `count`
    }, 1000);
    return () => clearInterval(interval);
  }, []);  // Empty deps = count never updates

  // ✅ SOLUTION: Use functional update
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prev => prev + 1);  // GOOD: Always latest value
    }, 1000);
    return () => clearInterval(interval);
  }, []);
};
```

---

### Jotai Atom Patterns (Novel's Choice)

**Why Jotai over Redux/Zustand/Context?**

| Feature | Jotai | Redux | Zustand | Context |
|---------|-------|-------|---------|---------|
| **Boilerplate** | Minimal | Heavy | Light | Medium |
| **Re-renders** | Atomic | Global | Selector | Global |
| **TypeScript** | Excellent | Manual | Good | Manual |
| **DevTools** | Basic | Excellent | Good | N/A |
| **Bundle Size** | 3KB | 8KB | 1.5KB | 0KB |
| **Learning Curve** | Low | High | Low | Low |

**Jotai Pattern** (from Novel):

```typescript
// 1. Define atoms (simple)
export const queryAtom = atom("");
export const rangeAtom = atom<Range | null>(null);

// 2. Provide store (isolated)
<Provider store={novelStore}>
  {children}
</Provider>

// 3. Read/write in components
const [query, setQuery] = useAtom(queryAtom);        // Read + Write
const range = useAtomValue(rangeAtom);               // Read only
const setRange = useSetAtom(rangeAtom);              // Write only

// 4. Derive atoms (computed values)
const filteredItemsAtom = atom((get) => {
  const query = get(queryAtom);
  return items.filter(item => item.title.includes(query));
});
```

**Architectural Benefits**:
- ✅ **Atomic**: Only components using `queryAtom` re-render when it changes
- ✅ **Isolated**: Each `EditorRoot` has its own store (multiple editors on page)
- ✅ **Minimal**: 2 atoms for entire slash command feature
- ✅ **Type-Safe**: TypeScript infers types automatically

**When to Use Jotai**:
- ✅ Lightweight global state (theme, user, UI flags)
- ✅ Multiple isolated instances (multiple editors)
- ✅ Derived/computed state
- ❌ Complex state machines (use XState)
- ❌ Large-scale apps with many devs (use Redux for standardization)

---

---

## Backend API Patterns

### Introduction: Backend for Frontend Developers

**Mindset Shift**: Frontend = what the user sees. Backend = what makes it work when they can't see.

**Key Differences**:

| Frontend | Backend |
|----------|---------|
| Runs in browser (untrusted) | Runs on server (trusted) |
| User controls environment | You control environment |
| State resets on refresh | State persists |
| One user at a time | Thousands of concurrent users |
| Performance = perceived speed | Performance = throughput & latency |
| Security = UI/UX | Security = data protection |

---

### Novel's Backend: API Routes in Next.js

Novel uses **Next.js API Routes** (edge runtime) for backend logic. Two endpoints:

1. `/api/generate` - AI text generation
2. `/api/upload` - Image upload to Vercel Blob

**Architecture**:

```
Frontend (Browser)          Backend (Edge Runtime)       External Services
├─ React Components     →   ├─ API Routes           →    ├─ OpenAI API
├─ Editor State             ├─ Request Handling          ├─ Vercel Blob Storage
└─ User Actions             └─ Response Streaming        └─ Rate Limiting (Upstash KV)
```

---

### Pattern 1: Edge Runtime for Performance

**File**: `apps/web/app/api/generate/route.ts`

```typescript
// This single line changes everything
export const runtime = "edge";

export async function POST(req: Request): Promise<Response> {
  // ... handler code
}
```

**What is Edge Runtime?**

**Traditional Serverless** (Node.js):
- Runs in AWS Lambda or similar
- Cold start: 500ms - 2s
- Runs in specific regions
- Full Node.js API available

**Edge Runtime**:
- Runs in V8 isolates (like Cloudflare Workers)
- Cold start: 50ms - 200ms
- Runs globally (closest to user)
- Limited API (Web standards only)

**Trade-offs**:

| Feature | Node.js | Edge |
|---------|---------|------|
| **Cold Start** | Slow (500ms-2s) | Fast (50-200ms) |
| **Global** | One region | Worldwide |
| **APIs** | Full Node.js | Web standards only |
| **Cost** | Higher | Lower |
| **Use Case** | Heavy computation | Fast responses |

**Why Novel Uses Edge**:
- ✅ AI streaming needs low latency
- ✅ Users worldwide (edge deploys globally)
- ✅ Simple logic (no need for full Node.js)
- ✅ Cost-effective for high traffic

**Backend Architecture Lesson**: **Location matters**. Running close to users reduces latency more than optimizing code.

---

### Pattern 2: Request Validation (Security First)

**File**: `apps/web/app/api/generate/route.ts`

```typescript
export async function POST(req: Request): Promise<Response> {
  // STEP 1: Validate environment (fail fast)
  if (!process.env.OPENAI_API_KEY || process.env.OPENAI_API_KEY === "") {
    return new Response(
      "Missing OPENAI_API_KEY - make sure to add it to your .env file.",
      { status: 400 }
    );
  }

  // STEP 2: Parse request body
  const { prompt, option, command } = await req.json();

  // STEP 3: Validate input (prevent abuse)
  if (!prompt || typeof prompt !== "string") {
    return new Response("Invalid prompt", { status: 400 });
  }

  if (prompt.length > 5000) {  // Prevent abuse
    return new Response("Prompt too long", { status: 400 });
  }

  // STEP 4: Rate limiting (prevent abuse)
  if (process.env.KV_REST_API_URL && process.env.KV_REST_API_TOKEN) {
    const ip = req.headers.get("x-forwarded-for");
    const ratelimit = new Ratelimit({
      redis: kv,
      limiter: Ratelimit.slidingWindow(50, "1 d"),  // 50 requests per day
    });

    const { success, limit, reset, remaining } = await ratelimit.limit(
      `novel_ratelimit_${ip}`
    );

    if (!success) {
      return new Response("You have reached your request limit for the day.", {
        status: 429,
        headers: {
          "X-RateLimit-Limit": limit.toString(),
          "X-RateLimit-Remaining": remaining.toString(),
          "X-RateLimit-Reset": reset.toString(),
        },
      });
    }
  }

  // STEP 5: Process request (only if validated)
  // ... AI logic
}
```

**Security Principles**:

1. **Never Trust User Input**: Validate everything
2. **Fail Fast**: Check environment before doing work
3. **Rate Limiting**: Protect against abuse
4. **Helpful Errors**: Tell user what's wrong (on validation errors)
5. **Secure Errors**: Don't leak internals (on system errors)

**Backend Architecture Lesson**: **Security is layered**. Each check is a gate. All gates must pass.

```
Request
  ↓
Gate 1: Environment vars? → No → 400 Bad Request
  ↓ Yes
Gate 2: Valid JSON? → No → 400 Bad Request
  ↓ Yes
Gate 3: Valid fields? → No → 400 Bad Request
  ↓ Yes
Gate 4: Rate limit? → Exceeded → 429 Too Many Requests
  ↓ Pass
Process Request → 200 OK or 500 Error
```

---

### Pattern 3: Error Handling (User-Friendly vs Secure)

```typescript
// ✅ GOOD: Helpful error for user mistakes
if (!process.env.OPENAI_API_KEY) {
  return new Response(
    "Missing OPENAI_API_KEY - make sure to add it to your .env file.",
    { status: 400 }
  );
}

if (prompt.length > 5000) {
  return new Response(
    "Prompt too long (max 5000 characters)",
    { status: 400 }
  );
}

// ✅ GOOD: Generic error for system failures
try {
  const result = await openai.chat.completions.create(...);
  return result;
} catch (error) {
  console.error("OpenAI API error:", error);  // Log for debugging
  return new Response(
    "AI service temporarily unavailable. Please try again.",
    { status: 500 }
  );
}

// ❌ BAD: Leaking internal details
catch (error) {
  return new Response(
    `Database error: ${error.message}`,  // DON'T expose DB structure
    { status: 500 }
  );
}
```

**Error Types**:

| Status | Type | Visibility | Example |
|--------|------|------------|---------|
| **4xx** | Client Error | Detailed | "Invalid email format" |
| **5xx** | Server Error | Generic | "Something went wrong" |

**Why Different**:
- **4xx**: User can fix it (be helpful)
- **5xx**: System issue (don't leak internals)

**Backend Architecture Lesson**: **Errors are a security surface**. Generic error messages prevent attackers from learning about your system.

---

### Pattern 4: Streaming Responses (Modern UX)

**File**: `apps/web/app/api/generate/route.ts`

```typescript
import { streamText } from "ai";
import { openai } from "@ai-sdk/openai";

export async function POST(req: Request) {
  // ... validation

  // Traditional approach (wait for entire response):
  // const response = await openai.chat.completions.create({...});
  // return Response.json(response);  // User waits 5-10 seconds

  // ✅ MODERN: Streaming approach
  const result = await streamText({
    model: openai("gpt-4o-mini"),
    messages: [
      { role: "system", content: systemMessage },
      { role: "user", content: prompt },
    ],
    maxTokens: 4096,
    temperature: 0.7,
  });

  return result.toDataStreamResponse();
  //     ↑ Returns immediately, streams as AI generates
}
```

**Streaming Flow**:

```
Frontend                 Backend                   OpenAI
   |                        |                        |
   |--- POST /api/generate --->|                        |
   |                        |--- Stream request ------>|
   |<-- Start stream -------|                        |
   |                        |<-- First chunk ---------|
   |<-- Chunk 1 ------------|                        |
   |  Render "Hello"        |                        |
   |                        |<-- Second chunk --------|
   |<-- Chunk 2 ------------|                        |
   |  Render "Hello world"  |                        |
   |                        |<-- Third chunk ---------|
   |<-- Chunk 3 ------------|                        |
   |  Render "Hello world!" |                        |
   |                        |<-- End stream ----------|
   |<-- Stream end ---------|                        |
```

**Why Streaming Matters**:

1. **Perceived Performance**: User sees progress immediately
2. **Actual Performance**: No waiting for complete response
3. **Better UX**: Can stop generation mid-stream
4. **Lower Memory**: Process chunks, don't buffer everything

**Backend Architecture Lesson**: **Time to First Byte (TTFB) > Total Time**. Users perceive fast response even if total time is same.

**Frontend Integration**:

```typescript
// File: apps/web/components/tailwind/generative/ai-selector.tsx
import { useCompletion } from "ai/react";

const { completion, complete, isLoading } = useCompletion({
  api: "/api/generate",
  onResponse: (response) => {
    if (response.status === 429) {
      toast.error("Rate limit exceeded");
    }
  },
});

// Trigger completion
complete(prompt, {
  body: { option: "continue" },
});

// `completion` updates in real-time as chunks arrive
return <div>{completion}</div>;
```

---

### Pattern 5: Pattern Matching for Business Logic

**File**: `apps/web/app/api/generate/route.ts`

```typescript
import { match } from "ts-pattern";

const { prompt, option, command } = await req.json();

// ✅ GOOD: Declarative pattern matching
const messages = match(option)
  .with("continue", () => [
    {
      role: "system",
      content: "You are an AI writing assistant that continues existing text based on context from prior text. Give more weight/priority to the later characters than the beginning ones. Limit your response to no more than 200 characters, but make sure to construct complete sentences."
    },
    { role: "user", content: prompt }
  ])
  .with("improve", () => [
    {
      role: "system",
      content: "You are an AI writing assistant that improves existing text. Limit your response to no more than 200 characters, but make sure to construct complete sentences."
    },
    { role: "user", content: `The existing text is: ${prompt}` }
  ])
  .with("shorter", () => [
    {
      role: "system",
      content: "You are an AI writing assistant that shortens existing text. Limit your response to no more than 200 characters, but make sure to construct complete sentences."
    },
    { role: "user", content: `The existing text is: ${prompt}` }
  ])
  .with("fix", () => [
    {
      role: "system",
      content: "You are an AI writing assistant that fixes grammar and spelling errors in existing text. Limit your response to no more than 200 characters, but make sure to construct complete sentences."
    },
    { role: "user", content: `The existing text is: ${prompt}` }
  ])
  .with("zap", () => [
    {
      role: "system",
      content: "You area an AI writing assistant that generates text based on a prompt. You take an input from the user and a command for manipulating the text"
    },
    { role: "user", content: `For this text: ${prompt}. You have to respect the command: ${command}` }
  ])
  .run();  // Execute pattern match

// ❌ BAD: Nested if-else chains
let messages;
if (option === "continue") {
  messages = [{ role: "system", content: "..." }, ...];
} else if (option === "improve") {
  messages = [{ role: "system", content: "..." }, ...];
} else if (option === "shorter") {
  messages = [{ role: "system", content: "..." }, ...];
} // ... 10 more options
```

**Why Pattern Matching?**

1. **Readability**: Clear mapping of input → output
2. **Exhaustiveness**: TypeScript ensures all cases handled
3. **Type Safety**: Autocomplete for option values
4. **Maintainability**: Easy to add new options
5. **Single Source of Truth**: All prompts in one place

**Backend Architecture Lesson**: **Business logic should read like configuration**. When adding a new AI command, you just add a new `.with()` clause.

---

### Pattern 6: Response Headers for Client Communication

```typescript
// File: apps/web/app/api/generate/route.ts

// Rate limiting headers
if (!success) {
  return new Response("You have reached your request limit for the day.", {
    status: 429,
    headers: {
      "X-RateLimit-Limit": limit.toString(),
      "X-RateLimit-Remaining": remaining.toString(),
      "X-RateLimit-Reset": reset.toString(),
    },
  });
}

// CORS headers (if needed for cross-origin)
return new Response(data, {
  headers: {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "POST, OPTIONS",
    "Content-Type": "application/json",
  },
});
```

**Standard Headers**:

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Data format | `application/json` |
| `X-RateLimit-*` | Rate limit info | See above |
| `Cache-Control` | Caching behavior | `no-store` (don't cache AI) |
| `Access-Control-*` | CORS | Cross-origin permissions |

**Backend Architecture Lesson**: **HTTP headers are metadata**. They tell the client how to interpret the response without parsing the body.

---

### Pattern 7: Environment Variables (Configuration)

```typescript
// ✅ GOOD: Read from environment
const apiKey = process.env.OPENAI_API_KEY;
const blobToken = process.env.BLOB_READ_WRITE_TOKEN;

// Validate on startup
if (!apiKey) {
  throw new Error("Missing OPENAI_API_KEY environment variable");
}

// ❌ BAD: Hardcoded secrets
const apiKey = "sk-abc123...";  // DON'T commit secrets to git

// ❌ BAD: Exposed to client
const apiKey = process.env.NEXT_PUBLIC_OPENAI_API_KEY;  // NEXT_PUBLIC_ = exposed to browser!
```

**Environment Variable Prefixes**:

```bash
# .env.local

# Server-only (secret, not exposed)
OPENAI_API_KEY=sk-...
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_...
DATABASE_URL=postgresql://...

# Client-exposed (public, sent to browser)
NEXT_PUBLIC_APP_NAME=Novel
NEXT_PUBLIC_API_URL=https://api.example.com
```

**Rule**: **NEVER** put secrets in `NEXT_PUBLIC_*` variables. They're exposed to users.

**Backend Architecture Lesson**: **Configuration is environment-specific**. Development, staging, and production have different API keys, database URLs, etc.

---

## Full-Stack Architecture Principles

### Principle 1: Separation of Concerns

**What**: Each part of the system has a single, well-defined responsibility.

**Novel's Architecture**:

```
┌─────────────────────────────────────────────────────────┐
│ PRESENTATION LAYER (Frontend)                          │
│ Responsibility: Display UI, handle user input          │
│                                                         │
│ packages/headless/    # UI logic (no styling)          │
│ apps/web/components/ # Styled implementations          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ APPLICATION LAYER (API Routes)                         │
│ Responsibility: Business logic, orchestration          │
│                                                         │
│ apps/web/app/api/generate/  # AI generation logic      │
│ apps/web/app/api/upload/    # File upload logic        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ INTEGRATION LAYER (External Services)                  │
│ Responsibility: Communication with third-party APIs    │
│                                                         │
│ OpenAI API        # AI text generation                 │
│ Vercel Blob       # File storage                       │
│ Upstash Redis     # Rate limiting                      │
└─────────────────────────────────────────────────────────┘
```

**Why This Matters**:

1. **Testability**: Test each layer independently
2. **Replaceability**: Swap OpenAI for Anthropic without touching UI
3. **Scalability**: Scale layers independently (more edge functions, not more frontend)
4. **Team Collaboration**: Frontend team doesn't need to know Redis internals

**Example**: Swapping AI Provider

```typescript
// Before: OpenAI
import { openai } from "@ai-sdk/openai";
const result = await streamText({
  model: openai("gpt-4o-mini"),
  messages,
});

// After: Anthropic (same interface)
import { anthropic } from "@ai-sdk/anthropic";
const result = await streamText({
  model: anthropic("claude-3-sonnet"),
  messages,  // Same messages format
});

// Frontend code unchanged! Separation of concerns wins.
```

---

### Principle 2: Single Source of Truth

**Bad Architecture**: Duplicated state across layers

```typescript
// ❌ BAD: Content in multiple places
const [content, setContent] = useState(...)  // React state
const editorContent = editor.getJSON()       // Editor state
const dbContent = await fetch('/api/content') // Database

// Which is the "truth"? Nobody knows. Bugs ensue.
```

**Good Architecture**: One authoritative source

```typescript
// ✅ GOOD: Editor state is source of truth for current editing
const content = editor.getJSON();

// ✅ GOOD: Database is source of truth for persisted data
const savedContent = await fetch('/api/document/123');
editor.commands.setContent(savedContent);  // Load from truth

// ✅ GOOD: Derive, don't duplicate
const wordCount = editor.storage.characterCount.words();  // Derived
const html = editor.getHTML();  // Derived
const markdown = editor.storage.markdown.getMarkdown();  // Derived
```

**Backend Example**:

```typescript
// ❌ BAD: User data in multiple places
const user = {
  name: localStorage.getItem('userName'),  // Client storage
  email: await fetch('/api/user/email'),   // API call
  avatar: context.user.avatar,             // Context
};

// ✅ GOOD: Database is source of truth
const user = await fetch('/api/user/me');
// Store in React Context for sharing
<UserContext.Provider value={user}>
```

**Architectural Lesson**: **Derived data is fine. Duplicated data is dangerous.** Always ask: "What is the single source of truth?"

---

### Principle 3: Fail Fast, Fail Loud

```typescript
// ✅ GOOD: Validate early, fail immediately
export async function POST(req: Request) {
  if (!process.env.OPENAI_API_KEY) {
    return new Response("Missing API key", { status: 500 });
    // Developer sees error immediately in logs
  }

  const { prompt } = await req.json();
  if (!prompt) {
    return new Response("Missing prompt", { status: 400 });
    // User gets immediate feedback
  }

  // Only proceed if validation passed
  const result = await generateAI(prompt);
  return result;
}

// ❌ BAD: Silent failures, unclear state
export async function POST(req: Request) {
  const apiKey = process.env.OPENAI_API_KEY || "default-key";  // DON'T
  const { prompt } = await req.json();

  try {
    const result = await generateAI(prompt || "");  // Empty prompt?
    return result || { text: "" };  // Empty result?
  } catch (e) {
    // Swallowed error - nobody knows what happened
  }
}
```

**Why Fail Fast?**

1. **Debugging**: Clear error at source, not downstream
2. **User Experience**: Immediate feedback, not timeout
3. **Cost**: Don't waste API calls on bad input
4. **Security**: Prevent unexpected behavior

**Backend Architecture Lesson**: **Errors are information**. Don't hide them. Surface them early.

---

### Principle 4: Idempotency (Backend Concept)

**What**: Performing the same operation multiple times has the same effect as performing it once.

**Example**: Image Upload

```typescript
// ❌ NOT IDEMPOTENT
// Uploading same image twice creates two files
export async function POST(req: Request) {
  const file = await req.blob();
  const filename = `${Date.now()}.png`;  // Always unique
  const url = await uploadToStorage(filename, file);
  return Response.json({ url });
}

// Problem: Network retry uploads same image twice
// Result: Two files, two URLs, two charges

// ✅ IDEMPOTENT
// Uploading same image twice returns same URL
export async function POST(req: Request) {
  const file = await req.blob();
  const hash = await computeHash(file);  // Content-based hash

  // Check if already uploaded
  const existing = await storage.get(hash);
  if (existing) {
    return Response.json({ url: existing.url });  // Return existing
  }

  // Upload new file
  const url = await uploadToStorage(hash, file);
  return Response.json({ url });
}

// Problem solved: Retry returns same URL
```

**Why This Matters**:

- **Network Retries**: Frontend might retry failed requests
- **User Mistakes**: User might click "Upload" twice
- **Cost**: Don't charge for duplicate uploads
- **Data Consistency**: Don't create duplicate records

**Architectural Lesson**: **Design for failure**. Users will retry. Networks will fail. Your API should handle it gracefully.

---

### Principle 5: Stateless Backend, Stateful Client

**Novel's Design**:

```
CLIENT (Stateful)                    SERVER (Stateless)
├─ Editor state                  →   ├─ Pure functions
├─ Undo/redo history                 ├─ No memory of previous requests
├─ Cursor position                   ├─ Each request is independent
└─ Draft content                     └─ Scales horizontally
```

**Stateless Backend** means:

```typescript
// ✅ GOOD: Stateless (each request independent)
export async function POST(req: Request) {
  const { prompt } = await req.json();

  // No server-side state needed
  const result = await generateAI(prompt);
  return Response.json(result);
}

// Each request has all the information it needs

// ❌ BAD: Stateful (remembers between requests)
let conversationHistory = [];  // DON'T

export async function POST(req: Request) {
  const { message } = await req.json();

  conversationHistory.push(message);  // State on server
  const result = await generateAI(conversationHistory);
  return Response.json(result);
}

// Problem: Works for 1 server, breaks with 2+ servers
// Problem: Memory leak (history grows forever)
// Problem: Different users see each other's history
```

**Why Stateless?**

1. **Scalability**: Add more servers, works instantly
2. **Reliability**: Server crash doesn't lose state
3. **Simplicity**: No session management
4. **Cost**: Servers can shut down when idle

**When to Use Stateful**:
- WebSocket connections (chat, collaborative editing)
- Long-running processes (video encoding)
- **But**: Novel doesn't need these for its use case

**Architectural Lesson**: **Push state to edges (client/database)**. Keep servers stateless for easy scaling.

---

## Scalability & Maintainability

### Thinking in Scale: 1 User vs 1 Million Users

**Scenario**: Novel becomes viral. Traffic goes from 10 users/day to 10,000 users/minute.

**What Breaks?**

| Component | 1 User | 1M Users | Solution |
|-----------|--------|----------|----------|
| **Frontend** | ✅ Works | ✅ Still works | Static files scale infinitely |
| **API Routes** | ✅ Works | ⚠️ Slow | Edge runtime auto-scales |
| **OpenAI API** | ✅ Works | ❌ Rate limited | Queue system, fallback models |
| **Storage** | ✅ Works | 💰 Expensive | CDN for images, compression |
| **Database** | ✅ Works | ❌ Overwhelmed | Read replicas, caching |

**Novel's Scalability Decisions**:

1. **Edge Runtime**: Scales automatically, worldwide
2. **Stateless Backend**: No session state to sync
3. **Client-Side Editing**: Editor runs in browser, zero server load
4. **Streaming**: Don't buffer large responses
5. **Rate Limiting**: Protect against abuse

---

### Code Maintainability: 6 Months Later

**Scenario**: You wrote code 6 months ago. A bug appears. Can you fix it?

**Bad Code** (unmaintainable):

```typescript
// ❌ What does this do?
function h(d) {
  return d.map(x => x.a ? x.b : x.c).filter(Boolean);
}

// ❌ Magic numbers
if (text.length > 5000) { ... }  // Why 5000?

// ❌ No error handling
const data = JSON.parse(localStorage.getItem('data'));  // Throws if invalid

// ❌ Side effects everywhere
function render() {
  setUser(getUser());  // Mutates global state
  localStorage.clear();  // Side effect
  return <div>...</div>;
}
```

**Good Code** (maintainable):

```typescript
// ✅ Clear naming
function extractValidEmails(users: User[]): string[] {
  return users
    .map(user => user.isActive ? user.email : user.backupEmail)
    .filter(Boolean);
}

// ✅ Named constants
const MAX_PROMPT_LENGTH = 5000;  // Prevent abuse, stay under API limits
if (text.length > MAX_PROMPT_LENGTH) { ... }

// ✅ Safe parsing
function getSavedData(): Data | null {
  try {
    const raw = localStorage.getItem('data');
    return raw ? JSON.parse(raw) : null;
  } catch {
    return null;  // Invalid JSON
  }
}

// ✅ Pure function
function render(user: User) {
  // No side effects, easy to test
  return <div>{user.name}</div>;
}
```

**Architectural Principles**:

1. **Clarity > Cleverness**: Code is read 10x more than written
2. **Explicit > Implicit**: Don't make readers guess
3. **Safe > Fast**: Correctness first, optimize later
4. **Tested > Perfect**: Working code > theoretically perfect code

---

## Plugin Architecture

### Introduction: Extensibility as a First-Class Concern

**Frontend Developer Mindset**: "I'll add features by modifying existing code"
**Architect Mindset**: "I'll design systems where features can be added without modifying core code"

This is the **Open/Closed Principle**: Software should be **open for extension, closed for modification**.

---

### Tiptap's Extension System: A Case Study

**File**: `packages/headless/src/extensions/`

Tiptap's architecture is built around **extensions**. Every feature (bold, italic, images, AI highlighting) is an extension.

**Core Idea**: The editor core knows nothing about bold text or images. Extensions teach it.

```
┌─────────────────────────────────────────────────────────┐
│ EDITOR CORE (Minimal)                                  │
│ - Document model                                       │
│ - Transaction system                                   │
│ - Command framework                                    │
│ - Plugin system                                        │
└─────────────────────────────────────────────────────────┘
                          ↓
    ┌─────────────────────────────────────────────┐
    │ EXTENSIONS (Everything Else)               │
    │                                             │
    │ ├─ StarterKit (basic formatting)           │
    │ ├─ Image (image support)                   │
    │ ├─ Link (link handling)                    │
    │ ├─ AIHighlight (custom feature)            │
    │ ├─ Mathematics (custom feature)            │
    │ └─ ... infinite possibilities              │
    └─────────────────────────────────────────────┘
```

**Why This Architecture?**

1. **Modularity**: Use only what you need
2. **Customization**: Add custom features without forking
3. **Team Collaboration**: Teams can work on separate extensions
4. **Testing**: Test extensions in isolation
5. **Distribution**: Ship extensions as npm packages

---

### Anatomy of an Extension

**File**: `packages/headless/src/extensions/ai-highlight.ts`

```typescript
import { Mark, mergeAttributes } from "@tiptap/core";

// 1. Define TypeScript interface for commands
declare module "@tiptap/core" {
  interface Commands<ReturnType> {
    AIHighlight: {
      setAIHighlight: (attributes?: { color: string }) => ReturnType;
      toggleAIHighlight: (attributes?: { color: string }) => ReturnType;
      unsetAIHighlight: () => ReturnType;
    };
  }
}

// 2. Define extension options
export interface AIHighlightOptions {
  HTMLAttributes: Record<string, string>;
}

// 3. Create the extension
export const AIHighlight = Mark.create<AIHighlightOptions>({
  name: "aiHighlight",

  // 4. Define how it appears in HTML
  renderHTML({ HTMLAttributes }) {
    return ["span", mergeAttributes(this.options.HTMLAttributes, HTMLAttributes), 0];
  },

  // 5. Define how it parses from HTML
  parseHTML() {
    return [
      {
        tag: "span",
        getAttrs: (node) =>
          (node as HTMLElement).getAttribute("data-ai-highlight") && null,
      },
    ];
  },

  // 6. Define commands (API for using the extension)
  addCommands() {
    return {
      setAIHighlight:
        (attributes) =>
        ({ commands }) =>
          commands.setMark(this.name, attributes),

      toggleAIHighlight:
        (attributes) =>
        ({ commands }) =>
          commands.toggleMark(this.name, attributes),

      unsetAIHighlight:
        () =>
        ({ commands }) =>
          commands.unsetMark(this.name),
    };
  },
});

// 7. Export helper functions (convenience API)
export const addAIHighlight = (editor: Editor) => {
  editor.commands.setAIHighlight();
};

export const removeAIHighlight = (editor: Editor) => {
  editor.commands.unsetAIHighlight();
};
```

**Extension Lifecycle**:

```
Install Extension
  ↓
Register with Editor (editor adds to schema)
  ↓
Commands Available (editor.commands.setAIHighlight)
  ↓
User Triggers Command (via UI or keyboard)
  ↓
Extension Creates Transaction (ProseMirror state change)
  ↓
View Updates (DOM re-renders)
```

---

### Extension Types: Mark vs Node vs Extension

**1. Mark** - Inline formatting (bold, italic, link, highlight)

```typescript
export const AIHighlight = Mark.create({
  // Marks can overlap: <span bold><span highlight>text</span></span>
});
```

**2. Node** - Block-level content (paragraph, heading, image)

```typescript
export const ImageResizer = Node.create({
  name: "image",
  // Nodes are structural: <p>paragraph</p><img /><p>paragraph</p>
});
```

**3. Extension** - Behavior without schema (keyboard shortcuts, plugins)

```typescript
export const CustomKeymap = Extension.create({
  // Pure behavior: no HTML representation
  addKeyboardShortcuts() {
    return {
      "Mod-Enter": () => this.editor.commands.saveDocument(),
    };
  },
});
```

**Decision Matrix**:

| Want to... | Use |
|------------|-----|
| Format inline text | Mark (bold, italic, color) |
| Add block content | Node (image, video, code block) |
| Add behavior | Extension (shortcuts, auto-save, analytics) |

---

### Building Custom Extensions: Real-World Example

**Scenario**: Add a "callout" feature (like Notion's callouts).

```typescript
// File: extensions/callout.ts
import { Node, mergeAttributes } from "@tiptap/core";

export interface CalloutOptions {
  types: ("info" | "warning" | "success" | "error")[];
}

export const Callout = Node.create<CalloutOptions>({
  name: "callout",

  // 1. Configure
  addOptions() {
    return {
      types: ["info", "warning", "success", "error"],
    };
  },

  // 2. Define structure (content model)
  group: "block",
  content: "block+",  // Can contain paragraphs, lists, etc.
  defining: true,

  // 3. Add attributes
  addAttributes() {
    return {
      type: {
        default: "info",
        parseHTML: (element) => element.getAttribute("data-type"),
        renderHTML: (attributes) => ({ "data-type": attributes.type }),
      },
    };
  },

  // 4. Define HTML rendering
  renderHTML({ node, HTMLAttributes }) {
    return [
      "div",
      mergeAttributes(HTMLAttributes, {
        class: `callout callout-${node.attrs.type}`,
      }),
      ["div", { class: "callout-icon" }, "ℹ️"],  // Icon
      ["div", { class: "callout-content" }, 0],   // Content slot
    ];
  },

  // 5. Define parsing from HTML
  parseHTML() {
    return [
      {
        tag: "div.callout",
        getAttrs: (node) => ({
          type: (node as HTMLElement).getAttribute("data-type") || "info",
        }),
      },
    ];
  },

  // 6. Add commands
  addCommands() {
    return {
      setCallout:
        (attrs) =>
        ({ commands }) =>
          commands.setNode(this.name, attrs),

      toggleCallout:
        (attrs) =>
        ({ commands }) =>
          commands.toggleWrap(this.name, attrs),
    };
  },

  // 7. Add keyboard shortcuts
  addKeyboardShortcuts() {
    return {
      "Mod-Shift-C": () => this.editor.commands.toggleCallout({ type: "info" }),
    };
  },
});

// Usage
const editor = useEditor({
  extensions: [
    StarterKit,
    Callout.configure({
      types: ["info", "warning", "success", "error"],
    }),
  ],
});

// In component
<button onClick={() => editor.chain().focus().toggleCallout({ type: "warning" }).run()}>
  Add Warning
</button>
```

**Architectural Benefits**:

1. **Self-Contained**: All callout logic in one file
2. **Configurable**: Can customize types, styling, shortcuts
3. **Testable**: Test extension independently
4. **Reusable**: Publish to npm, use in other projects
5. **Type-Safe**: TypeScript knows about `.toggleCallout()` command

---

### Plugin Architecture Principles

**1. Define Clear Extension Points**

```typescript
// ✅ GOOD: Multiple extension points
export const Callout = Node.create({
  addOptions() { ... },        // Configuration
  addAttributes() { ... },     // Data model
  renderHTML() { ... },        // Rendering
  parseHTML() { ... },         // Parsing
  addCommands() { ... },       // API
  addKeyboardShortcuts() { ... }, // Interactions
});

// ❌ BAD: Hardcoded, no extensibility
const callout = (type: string) => `<div class="callout-${type}">...</div>`;
```

**2. Composition Over Inheritance**

```typescript
// ✅ GOOD: Compose extensions
const editor = useEditor({
  extensions: [
    StarterKit,           // Base formatting
    Image,                // Image support
    Link,                 // Link support
    AIHighlight,          // Custom AI features
    Mathematics,          // Custom math support
    Callout,              // Custom callout
  ],
});

// Each extension is independent, works together
```

**3. Convention Over Configuration**

```typescript
// ✅ GOOD: Sensible defaults, optional config
Callout.configure({
  types: ["info", "warning"],  // Custom config
});

Callout.configure();  // Works with defaults
```

**Architectural Lesson**: **Plugin architecture = long-term flexibility**. Today you ship with 10 extensions. Next year, users create 100 custom extensions. Your core code never changes.

---

## Performance Patterns

### Introduction: Performance as Architecture

**Frontend Developer Mindset**: "I'll optimize later if it's slow"
**Architect Mindset**: "I'll design for performance from the start"

**Key Insight**: Most performance issues are architectural. Premature optimization is bad, but **premature pessimization** (choosing slow patterns) is worse.

---

### Performance Hierarchy: What to Optimize

```
┌──────────────────────────────────────────────────────┐
│ LEVEL 1: Don't Do It (Biggest Impact)              │
│ - Eliminate unnecessary work                        │
│ - Use pagination instead of loading 10,000 items    │
│ - Cache API responses                               │
│ - Lazy load off-screen content                     │
└──────────────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────────┐
│ LEVEL 2: Do It Less Often                          │
│ - Debounce/throttle                                 │
│ - Memoization                                       │
│ - Request deduplication                             │
└──────────────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────────┐
│ LEVEL 3: Do It Faster                              │
│ - Code splitting                                    │
│ - Web Workers                                       │
│ - Optimized algorithms                              │
└──────────────────────────────────────────────────────┘
```

**Rule**: Always start at Level 1. Doing unnecessary work faster is still waste.

---

### Pattern 1: Debouncing User Input

**Problem**: Saving to localStorage on every keystroke is expensive.

**File**: `apps/web/components/tailwind/advanced-editor.tsx`

```typescript
import { useDebouncedCallback } from "use-debounce";

// ❌ BAD: Runs on every keystroke (100+ times per second)
const handleUpdate = ({ editor }) => {
  const json = editor.getJSON();
  localStorage.setItem("content", JSON.stringify(json));  // Slow operation
  setSaveStatus("Saved");
};

// ✅ GOOD: Debounced (runs 500ms after user stops typing)
const debouncedUpdates = useDebouncedCallback(
  async (editor: EditorInstance) => {
    const json = editor.getJSON();

    // All expensive operations batched together
    window.localStorage.setItem("html-content", highlightCodeblocks(editor.getHTML()));
    window.localStorage.setItem("novel-content", JSON.stringify(json));
    window.localStorage.setItem("markdown", editor.storage.markdown.getMarkdown());

    setSaveStatus("Saved");
  },
  500  // Wait 500ms after last keystroke
);

// Usage
<EditorRoot>
  <EditorContent
    onUpdate={({ editor }) => {
      debouncedUpdates(editor);      // Debounced save
      setSaveStatus("Unsaved");      // Immediate UI feedback
    }}
  />
</EditorRoot>
```

**Performance Gain**: 100+ operations/second → 2 operations/second = **50x reduction**

**When to Debounce**:
- ✅ Search input (wait for user to finish typing)
- ✅ Auto-save (batch multiple edits)
- ✅ Window resize handlers
- ❌ Button clicks (user expects immediate feedback)
- ❌ Form submissions (must be immediate)

---

### Pattern 2: Memoization (Avoiding Re-computation)

**Problem**: Expensive calculations run on every render.

```typescript
// ❌ BAD: Re-filters on every render
const MyComponent = ({ items, query }) => {
  // This runs on EVERY render (even when items/query unchanged)
  const filteredItems = items.filter(item =>
    item.title.toLowerCase().includes(query.toLowerCase())
  );

  return <List items={filteredItems} />;
};

// ✅ GOOD: Memoized (only re-computes when dependencies change)
const MyComponent = ({ items, query }) => {
  const filteredItems = useMemo(
    () => items.filter(item =>
      item.title.toLowerCase().includes(query.toLowerCase())
    ),
    [items, query]  // Only recompute when these change
  );

  return <List items={filteredItems} />;
};
```

**When to Use useMemo**:

| Scenario | useMemo? | Why |
|----------|----------|-----|
| Filtering 10,000 items | ✅ Yes | Expensive computation |
| Adding two numbers | ❌ No | useMemo overhead > computation |
| Creating object for prop | ✅ Yes | Prevents child re-render |
| Formatting date | ❌ No | Fast operation |

**Rule**: Profile first. Memoize proven bottlenecks.

---

### Pattern 3: Code Splitting (Loading Less Code)

**Problem**: Loading 500KB of JavaScript when user needs 50KB.

```typescript
// ❌ BAD: Bundle all extensions upfront
import { StarterKit } from "@tiptap/starter-kit";
import { Image } from "@tiptap/extension-image";
import { Mathematics } from "./extensions/mathematics";  // Heavy: 100KB
import { CodeBlockLowlight } from "@tiptap/extension-code-block-lowlight";  // Heavy: 80KB
import { CharacterCount } from "@tiptap/extension-character-count";

const editor = useEditor({
  extensions: [StarterKit, Image, Mathematics, CodeBlockLowlight, CharacterCount],
});

// User loads 500KB even if they never use math or code blocks

// ✅ GOOD: Lazy load heavy extensions
import { lazy, Suspense } from "react";

const BasicEditor = lazy(() => import("./BasicEditor"));  // 50KB
const MathEditor = lazy(() => import("./MathEditor"));    // +100KB (only if needed)
const CodeEditor = lazy(() => import("./CodeEditor"));    // +80KB (only if needed)

const MyEditor = ({ features }) => {
  if (features.includes("math")) {
    return (
      <Suspense fallback={<Spinner />}>
        <MathEditor />
      </Suspense>
    );
  }

  return (
    <Suspense fallback={<Spinner />}>
      <BasicEditor />
    </Suspense>
  );
};

// User loads 50KB initially, 150KB only if they use math
```

**Next.js Automatic Code Splitting**:

```typescript
// File: app/editor/page.tsx
import dynamic from "next/dynamic";

// ✅ This component loads only when route is visited
const Editor = dynamic(() => import("@/components/advanced-editor"), {
  ssr: false,  // Don't render on server (editor needs browser APIs)
  loading: () => <Spinner />,
});

export default function EditorPage() {
  return <Editor />;
}
```

**Performance Gain**: Initial load: 500KB → 50KB = **90% reduction**

---

### Pattern 4: Virtualization (Rendering Less DOM)

**Problem**: Rendering 10,000 list items crashes browser.

```typescript
// ❌ BAD: Render all items (10,000 DOM nodes)
const LargeList = ({ items }) => (
  <div>
    {items.map(item => (
      <div key={item.id}>{item.title}</div>  // 10,000 divs in DOM
    ))}
  </div>
);

// ✅ GOOD: Virtualized (only render visible items)
import { useVirtualizer } from "@tanstack/react-virtual";

const VirtualizedList = ({ items }) => {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // Each item is ~50px tall
  });

  return (
    <div ref={parentRef} style={{ height: "400px", overflow: "auto" }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualItem => (
          <div
            key={items[virtualItem.index].id}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index].title}
          </div>
        ))}
      </div>
    </div>
  );
};

// Only renders ~10 visible items instead of 10,000
```

**Performance Gain**: 10,000 nodes → 10 nodes = **99.9% reduction**

**When to Virtualize**:
- ✅ Lists with 100+ items
- ✅ Tables with many rows
- ✅ Infinite scrolling feeds
- ❌ Small lists (<50 items)

---

### Pattern 5: Image Optimization

**File**: `apps/web/app/api/upload/route.ts`

```typescript
// ❌ BAD: Upload full-size images
const uploadImage = async (file: File) => {
  const blob = await put(file.name, file, { access: "public" });
  return blob.url;  // Returns 5MB image
};

// ✅ GOOD: Optimize before upload
import { put } from "@vercel/blob";

const uploadImage = async (file: File) => {
  // 1. Validate size
  const MAX_SIZE = 5 * 1024 * 1024;  // 5MB
  if (file.size > MAX_SIZE) {
    throw new Error("File too large");
  }

  // 2. Validate type
  if (!file.type.startsWith("image/")) {
    throw new Error("File must be an image");
  }

  // 3. Upload with content type
  const blob = await put(file.name, file, {
    access: "public",
    contentType: file.type,
  });

  return blob.url;
};

// Frontend: Use Next.js Image for optimization
import Image from "next/image";

<Image
  src={imageUrl}
  alt="Uploaded image"
  width={800}
  height={600}
  loading="lazy"          // Load when visible
  placeholder="blur"      // Show blur while loading
  quality={85}            // Compress to 85% quality
/>
// Next.js automatically:
// - Serves WebP format (50% smaller than JPEG)
// - Generates multiple sizes for responsive
// - Lazy loads off-screen images
```

**Performance Gain**: 5MB image → 200KB WebP = **96% reduction**

---

### Pattern 6: Database Query Optimization (Backend)

**Example**: Loading user with posts

```typescript
// ❌ BAD: N+1 query problem
const users = await db.users.findMany();  // 1 query

for (const user of users) {
  user.posts = await db.posts.findMany({
    where: { userId: user.id },
  });  // N queries (one per user)
}
// Total: 1 + N queries (slow!)

// ✅ GOOD: Single query with join
const users = await db.users.findMany({
  include: {
    posts: true,  // Join in one query
  },
});
// Total: 1 query (fast!)

// ✅ BETTER: Paginate to avoid loading everything
const users = await db.users.findMany({
  take: 20,        // Limit to 20 users
  skip: page * 20, // Offset for pagination
  include: {
    posts: {
      take: 5,     // Only 5 recent posts per user
      orderBy: { createdAt: "desc" },
    },
  },
});
// Loads 20 users + 100 posts instead of all data
```

**Backend Performance Lesson**: **Database queries compound**. N+1 queries scale linearly (slow). Joins scale logarithmically (fast).

---

### Pattern 7: Edge Caching

**File**: `apps/web/app/api/generate/route.ts`

```typescript
// ✅ Cache common prompts at edge
export async function POST(req: Request) {
  const { prompt } = await req.json();

  // Check cache first (Vercel Edge Cache)
  const cacheKey = `ai:${hashPrompt(prompt)}`;
  const cached = await edgeCache.get(cacheKey);

  if (cached) {
    return Response.json(cached);  // Return instantly
  }

  // Generate if not cached
  const result = await generateAI(prompt);

  // Cache for 1 hour
  await edgeCache.set(cacheKey, result, { ttl: 3600 });

  return Response.json(result);
}
```

**Caching Strategies**:

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Client-side** | User-specific, short-lived | Form drafts (localStorage) |
| **CDN** | Public, static | Images, CSS, JS |
| **Edge** | Global, semi-static | AI prompts, API responses |
| **Database** | Persistent, source of truth | User data, posts |

**Performance Gain**: AI request: 2s → 50ms (from cache) = **40x faster**

---

## Testing & Quality

### Introduction: Testing as Architecture Decision

**Frontend Developer Mindset**: "I'll manually test in the browser"
**Architect Mindset**: "I'll design testable code and automate testing"

**Key Insight**: **Testability is a design quality**. If code is hard to test, it's poorly designed.

---

### The Testing Pyramid

```
          ┌──────────────┐
          │   E2E Tests  │  Slow, expensive, realistic
          │   (5-10%)    │  Example: Full user flows
          └──────────────┘
       ┌────────────────────┐
       │ Integration Tests  │  Medium speed, test interactions
       │     (20-30%)       │  Example: API + Database
       └────────────────────┘
    ┌──────────────────────────┐
    │     Unit Tests           │  Fast, cheap, isolated
    │      (60-70%)            │  Example: Pure functions
    └──────────────────────────┘
```

**Why This Shape?**

1. **Unit Tests**: Fast (milliseconds), cheap to write, easy to debug
2. **Integration Tests**: Medium speed (seconds), test real interactions
3. **E2E Tests**: Slow (minutes), expensive to maintain, but catch real bugs

**Architectural Lesson**: **Invest in fast feedback**. 1,000 unit tests run in 5 seconds. 10 E2E tests run in 5 minutes.

---

### Pattern 1: Unit Testing Pure Functions

**File**: `packages/headless/src/utils/index.ts`

```typescript
// ✅ GOOD: Pure function (easy to test)
export const isValidUrl = (url: string): boolean => {
  try {
    new URL(url);
    return true;
  } catch {
    return false;
  }
};

export const getUrlFromString = (str: string): string | null => {
  if (isValidUrl(str)) return str;

  // Add https:// if missing
  const withProtocol = str.startsWith("http") ? str : `https://${str}`;
  return isValidUrl(withProtocol) ? withProtocol : null;
};

// Test file: utils.test.ts
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
  it("returns valid URL unchanged", () => {
    expect(getUrlFromString("https://example.com")).toBe("https://example.com");
  });

  it("adds https:// to bare domain", () => {
    expect(getUrlFromString("example.com")).toBe("https://example.com");
  });

  it("returns null for invalid input", () => {
    expect(getUrlFromString("not a url")).toBe(null);
  });
});
```

**Why Pure Functions Are Testable**:

1. **No side effects**: No API calls, no database, no DOM
2. **Deterministic**: Same input → same output, always
3. **Isolated**: No dependencies on other code
4. **Fast**: Run 1,000 tests in milliseconds

**Rule**: **Extract pure logic from components**. Test pure functions extensively, test components lightly.

---

### Pattern 2: Testing React Components

```typescript
// File: components/ui/button.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import { Button } from "./button";

describe("Button", () => {
  it("renders children", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });

  it("handles click events", () => {
    const handleClick = vi.fn();  // Mock function
    render(<Button onClick={handleClick}>Click</Button>);

    fireEvent.click(screen.getByText("Click"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("applies variant classes", () => {
    render(<Button variant="destructive">Delete</Button>);
    const button = screen.getByText("Delete");
    expect(button).toHaveClass("bg-destructive");
  });

  it("supports asChild prop", () => {
    render(
      <Button asChild>
        <a href="/home">Home</a>
      </Button>
    );
    const link = screen.getByText("Home");
    expect(link.tagName).toBe("A");
    expect(link).toHaveAttribute("href", "/home");
  });
});
```

**What to Test in Components**:

- ✅ Rendering with different props
- ✅ User interactions (click, type, etc.)
- ✅ Conditional rendering
- ✅ Prop combinations
- ❌ Implementation details (state variable names)
- ❌ Styling (CSS is not logic)

**Rule**: **Test behavior, not implementation**. If you refactor and tests break, you're testing implementation.

---

### Pattern 3: Testing API Routes

```typescript
// File: app/api/generate/route.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { POST } from "./route";

describe("POST /api/generate", () => {
  beforeEach(() => {
    // Set up environment
    process.env.OPENAI_API_KEY = "test-key";
  });

  it("returns 400 if prompt is missing", async () => {
    const req = new Request("http://localhost/api/generate", {
      method: "POST",
      body: JSON.stringify({}),  // Missing prompt
    });

    const response = await POST(req);
    expect(response.status).toBe(400);
  });

  it("returns 400 if prompt is too long", async () => {
    const longPrompt = "a".repeat(6000);  // > 5000 chars
    const req = new Request("http://localhost/api/generate", {
      method: "POST",
      body: JSON.stringify({ prompt: longPrompt }),
    });

    const response = await POST(req);
    expect(response.status).toBe(400);
  });

  it("generates AI response for valid input", async () => {
    // Mock OpenAI
    const mockStreamText = vi.fn().mockResolvedValue({
      toDataStreamResponse: () => new Response("Generated text"),
    });
    vi.mock("ai", () => ({ streamText: mockStreamText }));

    const req = new Request("http://localhost/api/generate", {
      method: "POST",
      body: JSON.stringify({ prompt: "Hello", option: "continue" }),
    });

    const response = await POST(req);
    expect(response.status).toBe(200);
    expect(mockStreamText).toHaveBeenCalled();
  });
});
```

**Backend Testing Strategy**:

1. **Unit test validation logic**: Fast, covers edge cases
2. **Integration test with mocked external APIs**: Tests your code, not OpenAI's
3. **E2E test in staging**: Catches real-world issues

---

### Pattern 4: Testing Custom Hooks

```typescript
// File: hooks/use-local-storage.test.ts
import { renderHook, act } from "@testing-library/react";
import { describe, it, expect, beforeEach } from "vitest";
import { useLocalStorage } from "./use-local-storage";

describe("useLocalStorage", () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it("initializes with default value", () => {
    const { result } = renderHook(() => useLocalStorage("key", "default"));
    expect(result.current[0]).toBe("default");
  });

  it("persists value to localStorage", () => {
    const { result } = renderHook(() => useLocalStorage("key", "default"));

    act(() => {
      result.current[1]("new value");  // Call setValue
    });

    expect(localStorage.getItem("key")).toBe(JSON.stringify("new value"));
    expect(result.current[0]).toBe("new value");
  });

  it("loads existing value from localStorage", () => {
    localStorage.setItem("key", JSON.stringify("existing"));

    const { result } = renderHook(() => useLocalStorage("key", "default"));
    expect(result.current[0]).toBe("existing");
  });

  it("handles invalid JSON gracefully", () => {
    localStorage.setItem("key", "invalid json{{{");

    const { result } = renderHook(() => useLocalStorage("key", "default"));
    // Should fall back to default instead of crashing
    expect(result.current[0]).toBe("default");
  });
});
```

**Hook Testing Tips**:

1. **Use `renderHook`**: Simulates React environment
2. **Use `act`**: Wraps state updates (React requirement)
3. **Test edge cases**: Invalid data, missing data, errors
4. **Clean up**: Reset localStorage between tests

---

### Pattern 5: Snapshot Testing (Use Sparingly)

```typescript
// File: components/editor-bubble.test.tsx
import { render } from "@testing-library/react";
import { EditorBubble } from "./editor-bubble";

it("matches snapshot", () => {
  const { container } = render(
    <EditorBubble>
      <button>Bold</button>
      <button>Italic</button>
    </EditorBubble>
  );

  expect(container).toMatchSnapshot();
  // Saves HTML structure to file
  // Fails if structure changes
});
```

**When to Use Snapshots**:

- ✅ Complex HTML structures (prevent accidental changes)
- ✅ Error messages (ensure consistency)
- ❌ Dynamic data (snapshots will break frequently)
- ❌ Styled components (CSS changes break tests)

**Warning**: Snapshot tests are easy to write but hard to maintain. Use for intentional structure validation only.

---

### Code Quality Tools

**1. TypeScript** - Type checking

```bash
pnpm typecheck  # Catches type errors before runtime
```

**2. Biome** - Linting & Formatting

```bash
pnpm lint       # Find code issues
pnpm format     # Auto-format code
```

**3. Testing**

```bash
pnpm test       # Run all tests
pnpm test:watch # Run tests on file change
pnpm coverage   # Generate coverage report
```

**CI/CD Pipeline** (recommended):

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - run: pnpm install
      - run: pnpm typecheck   # Must pass
      - run: pnpm lint        # Must pass
      - run: pnpm test        # Must pass
      - run: pnpm build       # Must build
```

**Architectural Lesson**: **Automate quality gates**. If it's not automated, it won't be done consistently.

---

## Summary & Key Takeaways

### The Journey: Frontend Developer → Full-Stack Architect

You started as a **mid-level React developer** comfortable building components and managing state. Through this guide, you've learned to think like a **full-stack architect** who:

1. **Sees the entire system** (frontend, backend, database, external services)
2. **Thinks in scale** (from 1 user to 1 million users)
3. **Designs for change** (extensible architecture, not hardcoded solutions)
4. **Balances trade-offs** (performance vs. simplicity, cost vs. features)
5. **Prioritizes maintainability** (code that others can understand 6 months later)

---

### Core Patterns Summary

#### **1. Naming & Organization**

✅ **Prefix components by feature** (`EditorBubble`, `AISelector`)
✅ **Use kebab-case for files** (`editor-bubble.tsx`)
✅ **Organize by feature, not file type** (co-location)
✅ **Barrel exports for clean APIs** (`index.ts`)

**Why**: Scalability from 10 files to 1,000 files

---

#### **2. TypeScript Patterns**

✅ **Interface for props**, type for unions
✅ **`readonly` for all props** (React philosophy)
✅ **Generics for reusable code** (`useLocalStorage<T>`)
✅ **Utility types to derive types** (`Omit`, `Pick`, `ReturnType`)
✅ **Module augmentation for extensions**

**Why**: Type safety = fewer runtime bugs, better refactoring

---

#### **3. React Component Patterns**

✅ **Compound components** (implicit context sharing)
✅ **`forwardRef`** (DOM access when needed)
✅ **Polymorphic `asChild`** (composition over inheritance)
✅ **Props spreading** (flexibility + type safety)
✅ **Custom hooks** (extract reusable logic)
✅ **`useMemo`** (prevent expensive re-computations)

**Why**: Flexible, reusable, performant components

---

#### **4. State Management**

✅ **Editor state for content** (Tiptap/ProseMirror)
✅ **Jotai atoms for UI state** (lightweight, atomic)
✅ **`useState` for local state** (component-specific)
✅ **localStorage for persistence** (user preferences)
❌ **Avoid prop drilling** (use context or global state)
❌ **Avoid derived state** (compute on render)

**Why**: Right state in right place = maintainability

---

#### **5. Backend API Patterns**

✅ **Edge runtime for performance** (low latency, global deployment)
✅ **Validate all input** (security layers)
✅ **Fail fast, fail loud** (immediate error feedback)
✅ **Streaming responses** (better perceived performance)
✅ **Pattern matching for business logic** (ts-pattern)
✅ **Rate limiting** (prevent abuse)
✅ **Environment variables** (never hardcode secrets)

**Why**: Secure, scalable, performant backend

---

#### **6. Full-Stack Architecture**

✅ **Separation of concerns** (presentation, application, integration layers)
✅ **Single source of truth** (derive, don't duplicate)
✅ **Stateless backend** (scales horizontally)
✅ **Idempotent APIs** (safe to retry)

**Why**: Systems that scale from 1 to 1 million users

---

#### **7. Plugin Architecture**

✅ **Extension-based design** (Tiptap pattern)
✅ **Open/Closed Principle** (open for extension, closed for modification)
✅ **Clear extension points** (configuration, commands, rendering)
✅ **Composition over inheritance** (combine extensions)

**Why**: Long-term flexibility without core changes

---

#### **8. Performance Patterns**

✅ **Don't do it** (eliminate unnecessary work)
✅ **Do it less often** (debounce, memoize)
✅ **Do it faster** (code splitting, virtualization)
✅ **Lazy load heavy code** (dynamic imports)
✅ **Optimize images** (Next.js Image, WebP)
✅ **Cache at edge** (fast global responses)

**Why**: Performance is user experience

---

#### **9. Testing & Quality**

✅ **Testing pyramid** (70% unit, 20% integration, 10% E2E)
✅ **Test pure functions** (fast, isolated, deterministic)
✅ **Test behavior, not implementation** (refactor-safe tests)
✅ **Mock external services** (control test environment)
✅ **Automate quality gates** (CI/CD)

**Why**: Confidence in changes, faster debugging

---

### Mental Model Shifts

| From (Frontend Developer) | To (Full-Stack Architect) |
|---------------------------|---------------------------|
| "How do I build this component?" | "How does this fit into the system?" |
| "This works on my machine" | "This works for 1 million users?" |
| "I'll fix bugs as they come" | "I'll design to prevent entire classes of bugs" |
| "I'll optimize later" | "I'll choose fast patterns from the start" |
| "I'll test manually" | "I'll automate testing" |
| "Frontend is my concern" | "The entire stack is my concern" |

---

### Next Steps in Your Journey

**1. Build Something** - Apply these patterns in a real project

**2. Read Novel's Code** - See patterns in production code
   - Start: `packages/headless/src/components/editor.tsx`
   - Then: `apps/web/components/tailwind/advanced-editor.tsx`
   - Finally: `apps/web/app/api/generate/route.ts`

**3. Experiment** - Break things, understand why
   - Remove Jotai, use Context (what breaks?)
   - Remove debouncing (how slow is it?)
   - Add a custom extension (how extensible is it?)

**4. Dive Deeper** - Explore advanced topics
   - Collaborative editing (Yjs, CRDTs)
   - Real-time sync (WebSockets, Server-Sent Events)
   - Advanced caching (Redis, CDN strategies)
   - Database optimization (indexes, query planning)

**5. Contribute** - Give back to open source
   - Fix bugs in Novel
   - Create custom extensions
   - Write documentation
   - Help other developers

---

### Final Thoughts

**Architecture is decision-making**. Every pattern in this guide represents a decision:

- **Jotai over Redux** - Chose simplicity and performance
- **Edge runtime over Node.js** - Chose low latency over full API access
- **Extension system** - Chose long-term flexibility over short-term simplicity
- **Stateless backend** - Chose scalability over statefulness

As you grow as an architect, you'll make these decisions yourself. The key is to:

1. **Understand the trade-offs** (no perfect solutions)
2. **Document your decisions** (help future developers)
3. **Stay flexible** (be willing to change when requirements change)
4. **Learn continuously** (best practices evolve)

**You're now equipped to**:
- ✅ Read and understand Novel's codebase
- ✅ Build production-quality React applications
- ✅ Design scalable backend APIs
- ✅ Make informed architectural decisions
- ✅ Think about systems, not just components

Welcome to the world of full-stack architecture. Build something amazing! 🚀

---

## Additional Resources

### Official Documentation
- [Novel Documentation](https://novel.sh/docs)
- [Tiptap Documentation](https://tiptap.dev)
- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

### Advanced Topics
- [ProseMirror Guide](https://prosemirror.net/docs/guide/)
- [Jotai Documentation](https://jotai.org)
- [Vercel Edge Runtime](https://edge-runtime.vercel.app)
- [Web Performance Optimization](https://web.dev/performance/)

### Community
- [Novel GitHub Discussions](https://github.com/steven-tey/novel/discussions)
- [Tiptap Community](https://github.com/ueberdosis/tiptap/discussions)
- [Next.js Discord](https://nextjs.org/discord)

---

**Document Version**: 1.0
**Last Updated**: November 2025
**Target Audience**: Mid-level React developers transitioning to full-stack architecture
**Prerequisites**: React, TypeScript, basic web development

---

*End of PATTERNS_AND_CONVENTIONS.md*
