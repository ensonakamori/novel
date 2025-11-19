# State Management Guide: Jotai in Novel

> **Target Audience**: Mid-level React developer learning atomic state management patterns
>
> **Prerequisites**: Understanding of React hooks, context, and state management concepts
>
> **What You'll Learn**: Jotai fundamentals, Novel's state architecture, state flow patterns, and when to use different state management approaches

---

## Table of Contents

1. [Overview](#overview)
2. [Why Jotai?](#why-jotai)
3. [Jotai Fundamentals](#jotai-fundamentals)
4. [Novel's State Architecture](#novels-state-architecture)
5. [State Flow Patterns](#state-flow-patterns)
6. [Jotai vs Editor State](#jotai-vs-editor-state)
7. [Advanced Patterns](#advanced-patterns)
8. [Performance Optimization](#performance-optimization)
9. [Comparison with Other Solutions](#comparison-with-other-solutions)
10. [Best Practices](#best-practices)

---

## Overview

Novel uses **two separate state management systems** working in harmony:

| State System | Technology | Purpose | Scope |
|--------------|------------|---------|-------|
| **UI State** | Jotai | Ephemeral UI interactions | Component-level |
| **Document State** | ProseMirror | Content and structure | Editor-level |

### Mental Model

Think of Novel's state like a **text editor application**:

- **Jotai** = UI chrome (menu visibility, search queries, modal states)
- **ProseMirror** = Document content (text, formatting, structure)

Just like Microsoft Word keeps your document content separate from whether the "Find" dialog is open, Novel separates document state (ProseMirror) from UI state (Jotai).

### Why Two State Systems?

**ProseMirror State** (document):
- ✅ Optimized for rich text editing
- ✅ Built-in undo/redo
- ✅ Transaction-based updates
- ✅ Schema validation
- ❌ Not React-friendly
- ❌ Verbose API for simple UI state

**Jotai** (UI):
- ✅ React-first design
- ✅ Minimal boilerplate
- ✅ Atomic updates (granular re-renders)
- ✅ Perfect for ephemeral state
- ❌ Overkill for document content
- ❌ No built-in undo/redo

---

## Why Jotai?

### The Problem: Slash Command State

Consider Novel's slash command feature:

```
User types "/" → Command menu opens
User types "hea" → Menu filters to "Heading 1", "Heading 2", "Heading 3"
User presses Enter → Execute command, close menu
```

**State needed**:
1. **Query**: Current search text ("hea")
2. **Range**: Cursor position where "/" was typed
3. **Menu visibility**: Is menu open?

**Challenge**: This state needs to be shared between:
- `EditorCommandOut` (receives data from Tiptap plugin)
- `EditorCommand` (renders filtered menu)
- `EditorCommandItem` (executes command with range)

**Solutions Considered**:

| Solution | Pros | Cons |
|----------|------|------|
| **React Context** | Built-in, simple | Provider hell, unnecessary re-renders |
| **Redux** | Powerful, DevTools | Too heavy, boilerplate overhead |
| **Zustand** | Minimal, fast | Not atomic, global by default |
| **Jotai** ✅ | Atomic, minimal, scoped stores | Learning curve (atoms concept) |

### Why Jotai Won

```typescript
// Jotai: 2 lines to define state
export const queryAtom = atom("");
export const rangeAtom = atom<Range | null>(null);

// Redux: ~20 lines minimum
// - Action creators
// - Reducer function
// - Store configuration
// - Type definitions

// Context: Provider hell
<QueryContext.Provider>
  <RangeContext.Provider>
    <MenuVisibilityContext.Provider>
      {/* Your app */}
    </MenuVisibilityContext.Provider>
  </RangeContext.Provider>
</QueryContext.Provider>
```

**Key Benefits**:

1. **Atomic**: Only components using `queryAtom` re-render when query changes
2. **Minimal**: Define state in one line
3. **Scoped**: Each editor instance gets isolated state
4. **TypeScript-first**: Excellent type inference
5. **React-friendly**: Works with Suspense, Concurrent Mode

---

## Jotai Fundamentals

### Atoms: The Building Blocks

**What is an atom?** A piece of state with read/write capabilities.

```typescript
import { atom } from "jotai";

// Primitive atom
const countAtom = atom(0);
//                  ↑
//            initial value

// Typed atom
const nameAtom = atom<string>("John");
const userAtom = atom<User | null>(null);

// Nullable atom
const selectionAtom = atom<Range | null>(null);
```

**Mental Model**: Atoms are like **Excel cells**. Each cell holds a value, and formulas (derived atoms) can reference other cells.

### Reading Atoms

**Three hooks for reading**:

```typescript
import { useAtom, useAtomValue, useSetAtom } from "jotai";

// 1. useAtom - Read + Write
const [count, setCount] = useAtom(countAtom);
//     ↑       ↑
//   value   setter

// 2. useAtomValue - Read only
const count = useAtomValue(countAtom);
//     ↑
//   value only

// 3. useSetAtom - Write only
const setCount = useSetAtom(countAtom);
//       ↑
//   setter only
```

**When to use which?**

```typescript
// Read + Write: Component needs both
const Counter = () => {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
};

// Read only: Component displays value
const Display = () => {
  const count = useAtomValue(countAtom);
  return <div>Count: {count}</div>;
};

// Write only: Component updates without displaying
const Incrementer = () => {
  const setCount = useSetAtom(countAtom);
  return <button onClick={() => setCount(c => c + 1)}>+1</button>;
};
```

**Performance Tip**: Use `useAtomValue` and `useSetAtom` when possible to avoid unnecessary re-renders.

---

### Writing to Atoms

**Pattern 1: Direct update**

```typescript
const setCount = useSetAtom(countAtom);

// Set value directly
setCount(5);

// Update based on previous value
setCount(prev => prev + 1);
```

**Pattern 2: From event handler**

```typescript
const MyInput = () => {
  const [query, setQuery] = useAtom(queryAtom);

  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
    />
  );
};
```

**Pattern 3: From useEffect**

```typescript
const MyComponent = ({ externalValue }) => {
  const setQuery = useSetAtom(queryAtom);

  useEffect(() => {
    setQuery(externalValue);
  }, [externalValue, setQuery]);
};
```

---

### Derived Atoms

**What?** Atoms that compute values from other atoms.

```typescript
// Base atoms
const firstNameAtom = atom("John");
const lastNameAtom = atom("Doe");

// Derived atom (read-only)
const fullNameAtom = atom((get) => {
  const firstName = get(firstNameAtom);
  const lastName = get(lastNameAtom);
  return `${firstName} ${lastName}`;
});

// Usage
const Display = () => {
  const fullName = useAtomValue(fullNameAtom);
  return <div>{fullName}</div>;  // "John Doe"
};
```

**Read-write derived atom**:

```typescript
const uppercaseAtom = atom(
  // Read
  (get) => get(nameAtom).toUpperCase(),

  // Write
  (get, set, newValue: string) => {
    set(nameAtom, newValue.toLowerCase());
  }
);

// Usage
const [name, setName] = useAtom(uppercaseAtom);
setName("JOHN");  // Stores "john" in nameAtom
console.log(name); // "JOHN"
```

**Novel Example** (hypothetical):

```typescript
// Base atoms
const queryAtom = atom("");
const itemsAtom = atom<Item[]>([...]);

// Derived: Filtered items
const filteredItemsAtom = atom((get) => {
  const query = get(queryAtom);
  const items = get(itemsAtom);

  return items.filter(item =>
    item.title.toLowerCase().includes(query.toLowerCase())
  );
});
```

---

### Async Atoms

**What?** Atoms that fetch data asynchronously.

```typescript
const userIdAtom = atom(1);

const userAtom = atom(async (get) => {
  const userId = get(userIdAtom);
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// Usage (with Suspense)
const UserProfile = () => {
  const user = useAtomValue(userAtom);  // Suspends until loaded
  return <div>{user.name}</div>;
};

// Wrap in Suspense boundary
<Suspense fallback={<Loading />}>
  <UserProfile />
</Suspense>
```

**Novel Use Case** (hypothetical):

```typescript
// Fetch AI suggestions based on query
const aiSuggestionsAtom = atom(async (get) => {
  const query = get(queryAtom);
  if (query.length < 3) return [];

  const response = await fetch(`/api/ai/suggest?q=${query}`);
  return response.json();
});
```

---

### Provider & Store

**Default Provider** (shared global state):

```typescript
import { Provider } from "jotai";

<Provider>
  <App />
</Provider>
```

**Custom Store** (isolated state):

```typescript
import { createStore, Provider } from "jotai";

const myStore = createStore();

<Provider store={myStore}>
  <App />
</Provider>
```

**Novel's Approach**:

```typescript
// File: packages/headless/src/utils/store.ts
import { createStore } from "jotai";

export const novelStore = createStore();

// File: packages/headless/src/components/editor.tsx
<Provider store={novelStore}>
  {children}
</Provider>
```

**Why?** Each `EditorRoot` gets its own store → isolated state per editor instance.

---

## Novel's State Architecture

### Atom Definitions

**File**: `packages/headless/src/utils/atoms.ts:1-10`

```typescript
import { atom } from "jotai";
import type { Range } from "@tiptap/core";

export const queryAtom = atom("");
export const rangeAtom = atom<Range | null>(null);
```

**Current State**:
- ✅ Minimal: Only 2 atoms
- ✅ Focused: Slash command functionality
- ✅ Typed: TypeScript types from Tiptap

**Future Expansion** (hypothetical):

```typescript
// Editor state
export const editorFocusedAtom = atom(false);
export const readOnlyAtom = atom(false);

// UI state
export const bubbleMenuOpenAtom = atom(false);
export const linkMenuOpenAtom = atom(false);

// Selection state
export const hasSelectionAtom = atom(false);
export const selectedTextAtom = atom("");

// Derived atoms
export const canEditAtom = atom((get) =>
  get(editorFocusedAtom) && !get(readOnlyAtom)
);
```

---

### Store Creation

**File**: `packages/headless/src/utils/store.ts:1-10`

```typescript
import { createStore } from "jotai";

// Custom store for editor isolation
export const novelStore: any = createStore();

// Re-export Jotai utilities
export * from "jotai";
```

**Why `any` type?** Avoids complex type exports. The store API is simple enough that type safety isn't critical here.

**Store Benefits**:

1. **Isolation**: Multiple editors don't share state
2. **Cleanup**: Store garbage collected when editor unmounts
3. **Testing**: Each test gets fresh store

---

### Provider Setup

**File**: `packages/headless/src/components/editor.tsx:1-30`

```typescript
import { Provider } from "jotai";
import { useRef } from "react";
import tunnel from "tunnel-rat";
import { novelStore } from "../utils/store";

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

**Component Tree**:

```
EditorRoot
  ├─ Jotai Provider (novelStore)
  │   └─ Tunnel Context Provider (for portaling)
  │       └─ children
  │           ├─ EditorContent (Tiptap)
  │           ├─ EditorCommand (reads atoms)
  │           └─ EditorBubble
```

**Multiple Editors**:

```tsx
// Each editor has isolated state
<EditorRoot>  {/* Store instance #1 */}
  <EditorContent />
</EditorRoot>

<EditorRoot>  {/* Store instance #2 */}
  <EditorContent />
</EditorRoot>
```

---

### State Flow Architecture

```mermaid
graph TB
    subgraph "ProseMirror Plugin Layer"
        Plugin[Slash Command Plugin]
        Suggestion[Suggestion Plugin]
    end

    subgraph "React Bridge Layer"
        EditorCommandOut[EditorCommandOut]
    end

    subgraph "Jotai State Layer"
        QueryAtom[queryAtom: string]
        RangeAtom[rangeAtom: Range]
    end

    subgraph "React UI Layer"
        EditorCommand[EditorCommand]
        EditorCommandItem[EditorCommandItem]
    end

    subgraph "ProseMirror State Layer"
        EditorState[editor.state]
    end

    Plugin -->|"onStart({ query, range })"| Suggestion
    Suggestion -->|props| EditorCommandOut
    EditorCommandOut -->|useSetAtom| QueryAtom
    EditorCommandOut -->|useSetAtom| RangeAtom

    QueryAtom -->|useAtom| EditorCommand
    RangeAtom -->|useAtomValue| EditorCommandItem

    EditorCommandItem -->|command({ editor, range })| EditorState
    EditorState -->|transaction| Plugin
```

---

## State Flow Patterns

### Pattern 1: Plugin → Atom (Write)

**Scenario**: Tiptap plugin detects "/" character and provides query/range data.

**File**: `packages/headless/src/components/editor-command.tsx:50-80`

```typescript
export const EditorCommandOut: FC<EditorCommandOutProps> = ({ query, range }) => {
  // Write-only hooks
  const setQuery = useSetAtom(queryAtom, { store: novelStore });
  const setRange = useSetAtom(rangeAtom, { store: novelStore });

  // Sync props to atoms
  useEffect(() => {
    setQuery(query);
  }, [query, setQuery]);

  useEffect(() => {
    setRange(range);
  }, [range, setRange]);

  // Keyboard navigation handling
  useEffect(() => {
    const navigationKeys = ["ArrowUp", "ArrowDown", "Enter"];
    const onKeyDown = (e: KeyboardEvent) => {
      if (navigationKeys.includes(e.key)) {
        e.preventDefault();
        const commandRef = document.querySelector("#slash-command");
        if (commandRef) {
          commandRef.dispatchEvent(
            new KeyboardEvent("keydown", {
              key: e.key,
              cancelable: true,
              bubbles: true,
            })
          );
        }
        return false;
      }
    };

    document.addEventListener("keydown", onKeyDown);
    return () => {
      document.removeEventListener("keydown", onKeyDown);
    };
  }, []);

  return (
    <EditorCommandTunnelContext.Consumer>
      {(tunnelInstance) => <tunnelInstance.Out />}
    </EditorCommandTunnelContext.Consumer>
  );
};
```

**Key Points**:

1. **Receives props from plugin**: `query` and `range`
2. **Writes to atoms**: `useSetAtom` for write-only access
3. **Explicit store**: `{ store: novelStore }` ensures correct store
4. **Keyboard forwarding**: DOM events forwarded to command menu

**Mental Model**: `EditorCommandOut` is a **bridge** between ProseMirror (imperative) and React (declarative).

---

### Pattern 2: Atom → UI (Read)

**Scenario**: Command menu reads query to filter items.

**File**: `packages/headless/src/components/editor-command.tsx:1-40`

```typescript
export const EditorCommand = forwardRef<
  HTMLDivElement,
  ComponentPropsWithoutRef<typeof Command>
>(({ children, className, ...rest }, ref) => {
  const [query, setQuery] = useAtom(queryAtom);

  return (
    <EditorCommandTunnelContext.Consumer>
      {(tunnelInstance) => (
        <tunnelInstance.In>
          <Command
            ref={ref}
            onKeyDown={(e) => {
              e.stopPropagation();
            }}
            id="slash-command"
            className={className}
            {...rest}
          >
            <Command.Input
              value={query}
              onValueChange={setQuery}
              style={{ display: "none" }}
            />
            {children}
          </Command>
        </tunnelInstance.In>
      )}
    </EditorCommandTunnelContext.Consumer>
  );
});
```

**Key Points**:

1. **Read + Write**: `useAtom` for bidirectional binding
2. **Binds to cmdk**: `value={query}` and `onValueChange={setQuery}`
3. **Hidden input**: `Command.Input` drives filtering, not visible
4. **Tunnel rendering**: `tunnelInstance.In` portals to menu location

**Mental Model**: `EditorCommand` is a **controlled component** with Jotai providing the state.

---

### Pattern 3: Atom → Command Execution (Read)

**Scenario**: User selects command, needs range to execute.

**File**: `packages/headless/src/components/editor-command-item.tsx:1-30`

```typescript
export const EditorCommandItem = forwardRef<
  HTMLDivElement,
  EditorCommandItemProps & ComponentPropsWithoutRef<typeof CommandItem>
>(({ children, onCommand, ...rest }, ref) => {
  const { editor } = useCurrentEditor();
  const range = useAtomValue(rangeAtom);  // Read-only

  if (!editor || !range) return null;

  return (
    <CommandItem
      ref={ref}
      {...rest}
      onSelect={() => onCommand({ editor, range })}
    >
      {children}
    </CommandItem>
  );
});
```

**Key Points**:

1. **Read-only**: `useAtomValue` for range (no writes)
2. **Null check**: Guards against missing editor or range
3. **Command callback**: Passes `editor` and `range` to handler

**Usage**:

```typescript
// File: apps/web/components/tailwind/slash-command.tsx
const suggestionItems = [
  {
    title: "Heading 1",
    command: ({ editor, range }) => {
      editor
        .chain()
        .focus()
        .deleteRange(range)           // Delete "/hea"
        .setNode("heading", { level: 1 })  // Insert H1
        .run();
    },
  },
  // ... more items
];
```

---

### Complete Flow Example

**User Action**: Types "/" then "hea" then Enter

```typescript
// 1. User types "/"
//    └─ Tiptap Suggestion plugin detects "/" character

// 2. Plugin calls onStart
onStart: (props: { editor: Editor; clientRect: DOMRect }) => {
  component = new ReactRenderer(EditorCommandOut, {
    props: { query: "", range: { from: 10, to: 11 } },
    editor: props.editor,
  });
  // ... show popup
}

// 3. EditorCommandOut receives props
const EditorCommandOut = ({ query, range }) => {
  const setQuery = useSetAtom(queryAtom);
  const setRange = useSetAtom(rangeAtom);

  useEffect(() => setQuery(""), []);           // Write: ""
  useEffect(() => setRange({ from: 10, to: 11 }), []);  // Write: range
}

// 4. User types "hea"
//    └─ Plugin calls onUpdate

onUpdate: (props: { editor: Editor; clientRect: GetReferenceClientRect }) => {
  component?.updateProps({ query: "hea", range: props.range });
}

// 5. EditorCommandOut updates atom
useEffect(() => setQuery("hea"), [query]);  // Write: "hea"

// 6. EditorCommand re-renders (subscribed to queryAtom)
const EditorCommand = () => {
  const [query] = useAtom(queryAtom);  // Read: "hea"

  return (
    <Command value={query}>
      {/* cmdk filters children by query */}
      <CommandList>
        <CommandItem>Heading 1</CommandItem>  {/* Matches "hea" */}
        <CommandItem>Heading 2</CommandItem>  {/* Matches "hea" */}
        <CommandItem>Heading 3</CommandItem>  {/* Matches "hea" */}
      </CommandList>
    </Command>
  );
}

// 7. User presses Enter
//    └─ EditorCommandItem reads range and executes

const EditorCommandItem = ({ onCommand }) => {
  const range = useAtomValue(rangeAtom);  // Read: { from: 10, to: 11 }

  return (
    <CommandItem onSelect={() => onCommand({ editor, range })}>
      Heading 1
    </CommandItem>
  );
}

// 8. Command executes
editor
  .chain()
  .focus()
  .deleteRange({ from: 10, to: 11 })  // Delete "/hea"
  .setNode("heading", { level: 1 })    // Insert H1
  .run();

// 9. Plugin cleans up
onExit: () => {
  popup?.[0]?.destroy();
  component?.destroy();
  // Atoms persist until next command
}
```

---

## Jotai vs Editor State

### Decision Matrix

| Use Jotai When | Use Editor.state When |
|----------------|----------------------|
| ✅ UI-only state (menus, modals) | ✅ Document content |
| ✅ Temporary state (search queries) | ✅ Text, nodes, marks |
| ✅ Cross-component sharing | ✅ Needs undo/redo |
| ✅ React-friendly patterns | ✅ Schema validation |
| ✅ No persistence needed | ✅ Collaborative editing |

### Examples

**Jotai** (UI state):

```typescript
// ✅ GOOD: Ephemeral UI state
const searchQueryAtom = atom("");
const modalOpenAtom = atom(false);
const selectedColorAtom = atom<string | null>(null);
const bubbleMenuPositionAtom = atom({ x: 0, y: 0 });

// ❌ BAD: Document content (use editor.state)
const documentContentAtom = atom<JSONContent>({});  // Don't do this!
const currentSelectionAtom = atom({ from: 0, to: 0 });  // Don't do this!
```

**Editor.state** (document):

```typescript
// ✅ GOOD: Document state
editor.state.doc             // Document tree
editor.state.selection       // Current selection
editor.getHTML()             // HTML export
editor.getJSON()             // JSON export

// ❌ BAD: UI state (use Jotai)
// Storing menu visibility in editor.state.plugins
// Managing search queries in editor storage
```

---

### Anti-pattern: Duplicating State

**Problem**: Storing editor state in both systems.

```typescript
// ❌ BAD: Duplicating document content
const contentAtom = atom<JSONContent>({});

const MyEditor = () => {
  const { editor } = useEditor();
  const [content, setContent] = useAtom(contentAtom);

  // Sync editor → atom (redundant!)
  useEffect(() => {
    if (editor) {
      const json = editor.getJSON();
      setContent(json);
    }
  }, [editor]);

  // Now you have TWO sources of truth - bad!
};

// ✅ GOOD: Use editor as single source of truth
const MyEditor = () => {
  const { editor } = useEditor();

  const content = editor?.getJSON();  // Read directly

  editor?.commands.setContent(newContent);  // Write directly
};
```

---

### Integration Pattern: Event-Based

**Pattern**: Editor events trigger atom updates (one-way sync).

```typescript
// Sync editor selection to atom (for UI purposes)
const selectionAtom = atom<{ from: number; to: number } | null>(null);

const MyEditor = () => {
  const setSelection = useSetAtom(selectionAtom);
  const { editor } = useEditor();

  useEffect(() => {
    if (!editor) return;

    // Listen to selection updates
    const updateSelection = () => {
      const { from, to } = editor.state.selection;
      setSelection({ from, to });
    };

    editor.on("selectionUpdate", updateSelection);
    return () => editor.off("selectionUpdate", updateSelection);
  }, [editor, setSelection]);
};

// Now other components can read selection without accessing editor
const SelectionInfo = () => {
  const selection = useAtomValue(selectionAtom);
  if (!selection) return null;

  return <div>Selection: {selection.from} to {selection.to}</div>;
};
```

**When to use**: UI components need editor state but don't have access to editor instance.

---

## Advanced Patterns

### Pattern 1: Atom Families

**What?** Create atoms dynamically based on parameters.

```typescript
import { atomFamily } from "jotai/utils";

// Create atom for each command item
const commandStateFamily = atomFamily((itemId: string) =>
  atom({ loading: false, error: null })
);

// Usage
const MyCommandItem = ({ itemId }) => {
  const [state, setState] = useAtom(commandStateFamily(itemId));

  return <div>{state.loading ? "Loading..." : "Ready"}</div>;
};
```

**Novel Use Case** (hypothetical):

```typescript
// Track loading state for each AI command option
const aiCommandLoadingFamily = atomFamily((option: string) =>
  atom(false)
);

const AICommandItem = ({ option }) => {
  const [loading, setLoading] = useAtom(aiCommandLoadingFamily(option));

  const handleClick = async () => {
    setLoading(true);
    await generateAIContent(option);
    setLoading(false);
  };

  return (
    <button onClick={handleClick}>
      {loading ? <Spinner /> : option}
    </button>
  );
};
```

---

### Pattern 2: Atom with Storage

**What?** Persist atoms to localStorage.

```typescript
import { atomWithStorage } from "jotai/utils";

// Persisted atom
const themeAtom = atomWithStorage<"light" | "dark">("theme", "light");

// Usage (auto-saves to localStorage)
const ThemeToggle = () => {
  const [theme, setTheme] = useAtom(themeAtom);

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Toggle Theme
    </button>
  );
};
```

**Novel Use Case** (hypothetical):

```typescript
// Remember user's preferred slash command sort order
const commandSortAtom = atomWithStorage<"alphabetical" | "recent">(
  "novel-command-sort",
  "alphabetical"
);
```

---

### Pattern 3: Computed Async Atoms

**What?** Atoms that derive async values.

```typescript
const queryAtom = atom("");

const searchResultsAtom = atom(async (get) => {
  const query = get(queryAtom);
  if (query.length < 2) return [];

  const response = await fetch(`/api/search?q=${query}`);
  return response.json();
});

// Usage with Suspense
const SearchResults = () => {
  const results = useAtomValue(searchResultsAtom);  // Suspends
  return <ul>{results.map(r => <li>{r.title}</li>)}</ul>;
};

<Suspense fallback={<Loading />}>
  <SearchResults />
</Suspense>
```

**Novel Use Case** (hypothetical):

```typescript
// AI-powered command suggestions
const aiSuggestionsAtom = atom(async (get) => {
  const query = get(queryAtom);
  const range = get(rangeAtom);

  if (!query || !range) return [];

  // Get context from editor
  const context = getPrevText(editor, range.from);

  const response = await fetch("/api/ai/suggest", {
    method: "POST",
    body: JSON.stringify({ query, context }),
  });

  return response.json();
});
```

---

### Pattern 4: Atom Scope Isolation

**What?** Create multiple isolated store instances.

```typescript
import { createStore, Provider } from "jotai";

// Create separate stores
const editorStore1 = createStore();
const editorStore2 = createStore();

// Isolated editors
<Provider store={editorStore1}>
  <Editor />  {/* queryAtom scoped to store1 */}
</Provider>

<Provider store={editorStore2}>
  <Editor />  {/* queryAtom scoped to store2 */}
</Provider>
```

**Novel's Implementation**:

```typescript
// File: packages/headless/src/components/editor.tsx
export const EditorRoot: FC<EditorRootProps> = ({ children }) => {
  const tunnelInstance = useRef(tunnel()).current;

  return (
    <Provider store={novelStore}>
      {children}
    </Provider>
  );
};

// Each EditorRoot instance creates new Provider with novelStore
// But novelStore is shared - WHY?
```

**Why Novel uses shared store**: Novel creates the store once and reuses it across all `EditorRoot` instances. This works because Novel is designed for **single editor per page**. If you need multiple editors, you'd modify this:

```typescript
// Multi-editor setup
export const EditorRoot: FC<EditorRootProps> = ({ children }) => {
  const store = useRef(createStore()).current;  // NEW STORE PER EDITOR
  const tunnelInstance = useRef(tunnel()).current;

  return (
    <Provider store={store}>
      <EditorCommandTunnelContext.Provider value={tunnelInstance}>
        {children}
      </EditorCommandTunnelContext.Provider>
    </Provider>
  );
};
```

---

### Pattern 5: Atom Effects

**What?** Side effects when atom mounts/updates.

```typescript
import { atomWithEffect } from "jotai-effect";

const countAtom = atomWithEffect((get, set) => {
  const count = get(countAtom);

  // Log to analytics
  console.log("Count changed:", count);

  // Cleanup
  return () => {
    console.log("Atom unmounted");
  };
});
```

**Novel Use Case** (hypothetical):

```typescript
// Track slash command usage analytics
const queryAtomWithAnalytics = atomWithEffect((get, set) => {
  const query = get(queryAtom);

  if (query.length > 0) {
    analytics.track("slash_command_query", { query });
  }
});
```

---

## Performance Optimization

### Atomic Re-renders

**Key Benefit**: Only components using an atom re-render when it changes.

```typescript
const queryAtom = atom("");
const rangeAtom = atom<Range | null>(null);

// Component A: Only re-renders when query changes
const ComponentA = () => {
  const query = useAtomValue(queryAtom);
  return <div>{query}</div>;
};

// Component B: Only re-renders when range changes
const ComponentB = () => {
  const range = useAtomValue(rangeAtom);
  return <div>{range?.from}</div>;
};

// Updating queryAtom DOES NOT re-render ComponentB ✅
```

**Comparison with Context**:

```typescript
// Context: ALL consumers re-render
const StateContext = createContext({ query: "", range: null });

const ComponentA = () => {
  const { query } = useContext(StateContext);
  return <div>{query}</div>;
};

const ComponentB = () => {
  const { range } = useContext(StateContext);
  return <div>{range?.from}</div>;
};

// Updating query RE-RENDERS BOTH COMPONENTS ❌
```

---

### Read-Only Optimization

```typescript
// ❌ SUBOPTIMAL: Component re-renders on every update (even if not writing)
const MyComponent = () => {
  const [query, setQuery] = useAtom(queryAtom);
  // setQuery is never used!

  return <div>{query}</div>;
};

// ✅ OPTIMAL: Component only re-renders when reading
const MyComponent = () => {
  const query = useAtomValue(queryAtom);

  return <div>{query}</div>;
};
```

---

### Write-Only Optimization

```typescript
// ❌ SUBOPTIMAL: Component re-renders when atom changes
const MyComponent = () => {
  const [query, setQuery] = useAtom(queryAtom);
  // query is never used!

  return <button onClick={() => setQuery("new")}>Update</button>;
};

// ✅ OPTIMAL: Component never re-renders (write-only)
const MyComponent = () => {
  const setQuery = useSetAtom(queryAtom);

  return <button onClick={() => setQuery("new")}>Update</button>;
};
```

---

### Derived Atom Memoization

Derived atoms are automatically memoized:

```typescript
const expensiveAtom = atom((get) => {
  const query = get(queryAtom);

  // Expensive computation
  return someExpensiveOperation(query);
});

// Computed value is cached until queryAtom changes
```

---

### Batching Updates

Jotai automatically batches React state updates:

```typescript
const Component = () => {
  const setQuery = useSetAtom(queryAtom);
  const setRange = useSetAtom(rangeAtom);

  const handleClick = () => {
    // Both updates batched into single re-render
    setQuery("new query");
    setRange({ from: 0, to: 5 });
  };
};
```

---

## Comparison with Other Solutions

### Jotai vs Redux

| Feature | Jotai | Redux |
|---------|-------|-------|
| **Boilerplate** | Minimal (1 line per atom) | Heavy (actions, reducers, types) |
| **Learning Curve** | Low | High |
| **TypeScript** | Excellent inference | Manual typing |
| **DevTools** | Basic | Powerful time-travel |
| **Bundle Size** | ~3KB | ~8KB |
| **Atomic Updates** | ✅ Built-in | ❌ Requires selectors |
| **Async** | ✅ Native | ❌ Requires middleware |

**Example**:

```typescript
// Jotai: 2 lines
const countAtom = atom(0);
const [count, setCount] = useAtom(countAtom);

// Redux: ~20 lines
const INCREMENT = "INCREMENT";
const increment = () => ({ type: INCREMENT });
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case INCREMENT: return state + 1;
    default: return state;
  }
};
const store = createStore(counterReducer);
const count = useSelector(state => state.counter);
const dispatch = useDispatch();
dispatch(increment());
```

---

### Jotai vs Zustand

| Feature | Jotai | Zustand |
|---------|-------|---------|
| **Philosophy** | Atomic (bottom-up) | Store (top-down) |
| **State Shape** | Atoms (granular) | Single store |
| **Updates** | Atomic | Whole store |
| **Derived State** | Derived atoms | Selectors |
| **Scoping** | Provider-based | Global by default |

**Example**:

```typescript
// Jotai: Atomic
const nameAtom = atom("John");
const ageAtom = atom(30);

const Name = () => {
  const name = useAtomValue(nameAtom);  // Only name updates trigger re-render
  return <div>{name}</div>;
};

// Zustand: Store
const useStore = create((set) => ({
  name: "John",
  age: 30,
  setName: (name) => set({ name }),
}));

const Name = () => {
  const name = useStore(state => state.name);  // Selector required for optimization
  return <div>{name}</div>;
};
```

---

### Jotai vs React Context

| Feature | Jotai | Context |
|---------|-------|---------|
| **Provider Nesting** | Single provider | Multiple nested |
| **Re-render Optimization** | Automatic | Manual (useMemo) |
| **Derived Values** | Derived atoms | useMemo |
| **Updates** | Atomic | Whole context |

**Example**:

```typescript
// Jotai: Clean
<Provider>
  <App />
</Provider>

// Context: Provider hell
<QueryContext.Provider value={query}>
  <RangeContext.Provider value={range}>
    <VisibilityContext.Provider value={visible}>
      <App />
    </VisibilityContext.Provider>
  </RangeContext.Provider>
</QueryContext.Provider>
```

---

## Best Practices

### 1. Use the Right Hook

```typescript
// ✅ Read + Write
const [value, setValue] = useAtom(atom);

// ✅ Read only
const value = useAtomValue(atom);

// ✅ Write only
const setValue = useSetAtom(atom);

// ❌ Unnecessary reads
const [value, setValue] = useAtom(atom);  // But only using setValue
```

---

### 2. Explicit Store Reference

When writing from outside React components:

```typescript
// ✅ GOOD: Explicit store
const setQuery = useSetAtom(queryAtom, { store: novelStore });

// ❌ BAD: Implicit (uses default store)
const setQuery = useSetAtom(queryAtom);
```

---

### 3. Keep Atoms Focused

```typescript
// ✅ GOOD: Granular atoms
const queryAtom = atom("");
const rangeAtom = atom<Range | null>(null);
const visibleAtom = atom(false);

// ❌ BAD: Large object atom
const slashCommandAtom = atom({
  query: "",
  range: null,
  visible: false,
  items: [],
  selectedIndex: 0,
});
// Problem: Updating query re-renders ALL consumers
```

---

### 4. Type Your Atoms

```typescript
// ✅ GOOD: Explicit types
const rangeAtom = atom<Range | null>(null);
const userAtom = atom<User | undefined>(undefined);

// ❌ BAD: Implicit (loses type safety)
const rangeAtom = atom(null);  // Type: Atom<null>
```

---

### 5. Don't Duplicate Editor State

```typescript
// ❌ BAD: Storing document in atom
const contentAtom = atom<JSONContent>({});

// ✅ GOOD: Read from editor directly
const content = editor?.getJSON();
```

---

### 6. Use Derived Atoms for Computed Values

```typescript
// ✅ GOOD: Derived atom (cached)
const filteredItemsAtom = atom((get) => {
  const query = get(queryAtom);
  return items.filter(item => item.title.includes(query));
});

// ❌ BAD: Computing in component (re-computed on every render)
const Component = () => {
  const query = useAtomValue(queryAtom);
  const filtered = items.filter(item => item.title.includes(query));
  // ...
};
```

---

### 7. Scope Stores for Isolation

```typescript
// Multiple editors: Use separate stores
const EditorRoot = ({ children }) => {
  const store = useRef(createStore()).current;

  return (
    <Provider store={store}>
      {children}
    </Provider>
  );
};
```

---

## Summary & Key Takeaways

### Jotai in Novel

**Current Implementation**:
- ✅ 2 atoms: `queryAtom`, `rangeAtom`
- ✅ Slash command state management
- ✅ Plugin-to-React bridge
- ✅ Isolated stores per editor

**Architecture**:

```
ProseMirror Plugin (imperative)
  ↓
EditorCommandOut (bridge)
  ↓
Jotai Atoms (declarative state)
  ↓
React Components (UI)
```

---

### When to Use Jotai

**Use Jotai for**:
- ✅ UI-only state (menus, modals, dropdowns)
- ✅ Temporary state (search queries, form inputs)
- ✅ Cross-component state sharing
- ✅ React-friendly patterns

**Don't Use Jotai for**:
- ❌ Document content (use editor.state)
- ❌ Text, nodes, marks (use ProseMirror)
- ❌ Undo/redo state (use Tiptap history)
- ❌ Collaborative editing (use Y.js)

---

### Performance Benefits

1. **Atomic Re-renders**: Only consuming components re-render
2. **Automatic Memoization**: Derived atoms cached
3. **Batched Updates**: Multiple updates = single render
4. **Selective Subscription**: Read/write/both optimization

---

### Core Patterns

| Pattern | Hook | Use Case |
|---------|------|----------|
| Read + Write | `useAtom` | Interactive components |
| Read Only | `useAtomValue` | Display components |
| Write Only | `useSetAtom` | Event handlers |
| Derived | `atom(get => ...)` | Computed values |
| Async | `atom(async get => ...)` | Data fetching |

---

## Next Steps

1. **Explore**: Read atom definitions in `packages/headless/src/utils/atoms.ts`
2. **Experiment**: Add a new atom for editor focus state
3. **Build**: Create a derived atom for filtered items
4. **Advanced**: Implement async atom for AI suggestions
5. **Extend**: Add localStorage persistence with `atomWithStorage`

---

## Additional Resources

### Official Documentation

- **Jotai**: https://jotai.org/docs/introduction
- **Jotai Utils**: https://jotai.org/docs/utilities/utility-overview
- **Jotai DevTools**: https://jotai.org/docs/tools/devtools

### Novel-Specific

- **Editor Core**: See EDITOR_CORE_GUIDE.md
- **Frontend Architecture**: See FRONTEND_ARCHITECTURE.md
- **Integration Guide**: See INTEGRATION_GUIDE.md (next)

### Code References

- **Atoms**: `packages/headless/src/utils/atoms.ts`
- **Store**: `packages/headless/src/utils/store.ts`
- **Provider**: `packages/headless/src/components/editor.tsx`
- **Usage**: `packages/headless/src/components/editor-command.tsx`

---

**Next Document**: INTEGRATION_GUIDE.md - How to integrate Novel into your application with different frameworks and setups.
