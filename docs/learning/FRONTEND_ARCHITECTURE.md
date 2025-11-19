# Frontend Architecture Guide

> **Target Audience**: Mid-level React developer learning Next.js 15, modern styling patterns, and full-stack architecture
>
> **Prerequisites**: Understanding of React, TypeScript, and component-based architecture
>
> **What You'll Learn**: Next.js App Router, server vs client components, Tailwind CSS architecture, shadcn/ui patterns, and production-ready frontend organization

---

## Table of Contents

1. [Overview](#overview)
2. [Next.js 15 App Router](#nextjs-15-app-router)
3. [Server vs Client Components](#server-vs-client-components)
4. [Component Organization](#component-organization)
5. [Styling Architecture](#styling-architecture)
6. [shadcn/ui Integration](#shadcnui-integration)
7. [API Routes (Edge Runtime)](#api-routes-edge-runtime)
8. [State Management Patterns](#state-management-patterns)
9. [Performance Optimizations](#performance-optimizations)
10. [Best Practices & Patterns](#best-practices--patterns)

---

## Overview

Novel's frontend is built with **Next.js 15**, leveraging the App Router architecture for a modern, performant web application. The architecture emphasizes:

- **Server-first rendering** with strategic client components
- **Type-safe styling** with Tailwind CSS and CVA
- **Accessible UI** with Radix UI primitives
- **Edge-optimized** API routes for AI features
- **Modular components** with clear responsibilities

### Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Next.js** | 15.1.4 ✅ | Full-stack React framework |
| **React** | 18.2.0 ⚠️ | UI library (19.2.0 available) |
| **TypeScript** | 5.7.3 ✅ | Type safety |
| **Tailwind CSS** | 3.4.1 ⚠️ | Utility-first CSS (v4 available) |
| **Radix UI** | Latest | Accessible primitives |
| **next-themes** | Latest | Theme management |
| **Vercel AI SDK** | 3.0.12 ⚠️ | AI integration (v5 available) |

---

## Next.js 15 App Router

### Directory Structure

The App Router uses file-system based routing with special files:

```
apps/web/app/
├── api/                    # API routes (Edge runtime)
│   ├── generate/
│   │   └── route.ts       # POST /api/generate - AI text generation
│   └── upload/
│       └── route.ts       # POST /api/upload - Image uploads
├── layout.tsx             # Root layout (server component)
├── page.tsx               # Home page route
├── providers.tsx          # Client-side providers wrapper
├── favicon.ico            # Favicon
└── opengraph-image.png    # OG image for social sharing
```

**Mental Model**: Think of the `app/` directory as a **routing tree**. Each folder is a route segment, and special files (`layout.tsx`, `page.tsx`, `route.ts`) define UI or endpoints.

### Root Layout Pattern

**File**: `apps/web/app/layout.tsx:1-45`

```typescript
import type { Metadata, Viewport } from "next";
import { ReactNode } from "react";
import Providers from "./providers";
import "@/styles/globals.css";
import "@/styles/prosemirror.css";
import 'katex/dist/katex.min.css';

export const metadata: Metadata = {
  title: "Novel - Notion-style WYSIWYG editor with AI-powered autocompletion",
  description: "Novel is a Notion-style WYSIWYG editor with AI-powered autocompletion.",
  openGraph: {
    title: "Novel - Notion-style WYSIWYG editor with AI-powered autocompletion",
    description: "Novel is a Notion-style WYSIWYG editor with AI-powered autocompletion.",
  },
  twitter: {
    title: "Novel - Notion-style WYSIWYG editor with AI-powered autocompletion",
    description: "Novel is a Notion-style WYSIWYG editor with AI-powered autocompletion.",
    card: "summary_large_image",
    creator: "@steventey",
  },
  metadataBase: new URL("https://novel.sh"),
};

export const viewport: Viewport = {
  themeColor: "#ffffff",
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

**Key Architectural Decisions**:

1. **Server Component by Default**: No `"use client"` directive means this runs on the server
2. **CSS Imports at Top Level**: Global styles imported once in root layout
3. **Metadata API**: Type-safe SEO configuration (Next.js 13+)
4. **suppressHydrationWarning**: Handles theme provider hydration mismatch (dark mode class added by JS)
5. **Providers Wrapper**: Delegates client-side logic to a separate component

**Why This Pattern?**

- ✅ **SEO-friendly**: Metadata rendered server-side
- ✅ **Performance**: No JS needed for layout rendering
- ✅ **Clean separation**: Server layout, client providers

**Analogy**: Think of `layout.tsx` as the **HTML skeleton** of your app. It provides the `<html>` and `<body>` tags, loads global styles, and sets up metadata. The `Providers` component is like **injecting interactivity** into that skeleton.

---

### Providers Pattern

**File**: `apps/web/app/providers.tsx:1-50`

```typescript
"use client";

import { ThemeProvider, useTheme } from "next-themes";
import { Toaster } from "sonner";
import { Analytics } from "@vercel/analytics/react";
import { createContext, Dispatch, ReactNode, SetStateAction } from "react";
import { useLocalStorage } from "@/hooks/use-local-storage";

// Custom context for font management
export const AppContext = createContext<{
  font: string;
  setFont: Dispatch<SetStateAction<string>>;
}>({
  font: "Default",
  setFont: () => {},
});

const ToasterProvider = () => {
  const { theme } = useTheme() as {
    theme: "light" | "dark" | "system";
  };
  return <Toaster theme={theme} />;
};

export default function Providers({ children }: { children: ReactNode }) {
  const [font, setFont] = useLocalStorage<string>("novel__font", "Default");

  return (
    <ThemeProvider
      attribute="class"
      enableSystem
      disableTransitionOnChange
      defaultTheme="system"
    >
      <AppContext.Provider value={{ font, setFont }}>
        <ToasterProvider />
        {children}
        <Analytics />
      </AppContext.Provider>
    </ThemeProvider>
  );
}
```

**Why "use client"?**

This component uses:
- ✅ React hooks (`useLocalStorage`, `useTheme`)
- ✅ Browser APIs (localStorage via custom hook)
- ✅ Third-party client components (ThemeProvider, Toaster)

**Provider Composition Pattern**:

```
ThemeProvider (dark mode)
  └─ AppContext.Provider (font preferences)
      ├─ ToasterProvider (notifications)
      ├─ {children} (your app)
      └─ Analytics (Vercel analytics)
```

**Mental Model**: Providers are like **layers of context wrapping your app**. Each layer adds a feature (theming, fonts, notifications) that any child component can access.

**Key Features**:

1. **next-themes Integration**:
   - `attribute="class"`: Adds `dark` class to `<html>` for Tailwind
   - `enableSystem`: Respects OS theme preference
   - `defaultTheme="system"`: Starts with OS theme

2. **Font Persistence**:
   - Uses custom `useLocalStorage` hook
   - Persists font choice across sessions
   - Available via `useContext(AppContext)`

3. **Theme-Aware Toasts**:
   - `ToasterProvider` reads current theme
   - Passes to sonner for proper styling

**Why Separate from Layout?**

- ✅ Keeps server layout minimal (faster initial render)
- ✅ Isolates client-side logic
- ✅ Easier to test providers independently

---

### Home Page Component

**File**: `apps/web/app/page.tsx:1-100`

The home page renders the editor demo with all UI controls. It's a **client component** because it manages editor state.

**Key Pattern**: Single-page application structure. Novel doesn't use multiple routes—it's focused on the editor experience.

---

## Server vs Client Components

### The Golden Rule

> **Default to server components. Only use "use client" when you need:**
> - React hooks (useState, useEffect, useContext)
> - Event handlers (onClick, onChange)
> - Browser APIs (window, localStorage, navigator)
> - Third-party libraries that use hooks

### Novel's Component Strategy

| Component | Type | Why? |
|-----------|------|------|
| `app/layout.tsx` | Server | Static HTML, metadata, CSS imports |
| `app/providers.tsx` | Client | Uses hooks, context, browser APIs |
| `app/page.tsx` | Client | Renders interactive editor |
| `components/tailwind/advanced-editor.tsx` | Client | Editor state, event handlers |
| `components/tailwind/selectors/*.tsx` | Client | Interactive popovers, editor commands |
| `components/tailwind/ui/*.tsx` | Client | Radix UI requires client |

### Server Component Benefits

```typescript
// This runs ONLY on the server
export default function Layout({ children }) {
  // ✅ Can access server-only APIs
  // ✅ No JavaScript sent to client
  // ✅ Direct database queries possible
  // ✅ Smaller bundle size

  return <div>{children}</div>;
}
```

### Client Component Reality Check

**Common Misconception**: "Client components don't render on the server"

**Reality**: Client components **do render on the server** (SSR), then **hydrate on the client**. The `"use client"` directive means:
- ✅ Pre-rendered HTML sent to browser (fast initial load)
- ✅ JavaScript bundle includes this component (for interactivity)
- ✅ React hydrates the component on client (attaches event listeners)

**Mental Model**: Think of server components as **static snapshots** and client components as **interactive widgets** that start as snapshots, then come alive.

---

## Component Organization

### Directory Structure

```
components/tailwind/
├── advanced-editor.tsx          # Main editor component
├── extensions.ts                # Tiptap extensions configuration
├── image-upload.ts             # Image upload handler
├── slash-command.tsx           # Slash command items configuration
├── generative/                 # AI-powered components
│   ├── ai-completion-command.tsx
│   ├── ai-selector-commands.tsx
│   ├── ai-selector.tsx
│   └── generative-menu-switch.tsx
├── selectors/                  # Editor toolbar selectors
│   ├── color-selector.tsx
│   ├── link-selector.tsx
│   ├── math-selector.tsx
│   ├── node-selector.tsx
│   └── text-buttons.tsx
└── ui/                         # shadcn/ui components
    ├── button.tsx
    ├── command.tsx
    ├── dialog.tsx
    ├── popover.tsx
    └── icons/
        ├── crazy-spinner.tsx
        └── magic.tsx
```

**Organizational Principles**:

1. **Feature-based folders**: `generative/` groups AI features, `selectors/` groups toolbar controls
2. **Shared UI in `ui/`**: Reusable shadcn/ui components
3. **Flat structure**: No deep nesting—easy to find components
4. **Co-location**: Related files together (e.g., `extensions.ts` near `advanced-editor.tsx`)

---

### Main Editor Component Pattern

**File**: `apps/web/components/tailwind/advanced-editor.tsx:1-300`

```typescript
"use client";

import { useState } from "react";
import { useDebounce } from "use-debounce";
import { EditorRoot, EditorContent, type JSONContent, EditorInstance } from "novel";
import { useDebouncedCallback } from "use-debounce";
import { defaultExtensions } from "./extensions";

const TailwindAdvancedEditor = () => {
  // State management
  const [initialContent, setInitialContent] = useState<null | JSONContent>(null);
  const [saveStatus, setSaveStatus] = useState("Saved");
  const [charsCount, setCharsCount] = useState(0);

  // Popover visibility (controlled from parent)
  const [openNode, setOpenNode] = useState(false);
  const [openColor, setOpenColor] = useState(false);
  const [openLink, setOpenLink] = useState(false);
  const [openAI, setOpenAI] = useState(false);

  // Load initial content from localStorage
  useEffect(() => {
    const content = window.localStorage.getItem("novel-content");
    if (content) setInitialContent(JSON.parse(content));
    else setInitialContent(defaultEditorContent);
  }, []);

  // Debounced auto-save (500ms delay)
  const debouncedUpdates = useDebouncedCallback(async (editor: EditorInstance) => {
    const json = editor.getJSON();
    setCharsCount(editor.storage.characterCount.words());

    // Save in multiple formats
    window.localStorage.setItem("html-content", highlightCodeblocks(editor.getHTML()));
    window.localStorage.setItem("novel-content", JSON.stringify(json));
    window.localStorage.setItem("markdown", editor.storage.markdown.getMarkdown());

    setSaveStatus("Saved");
  }, 500);

  if (!initialContent) return null;

  return (
    <div className="relative w-full max-w-screen-lg">
      <div className="flex absolute right-5 top-5 z-10 gap-2">
        <div className="rounded-lg bg-accent px-2 py-1 text-sm text-muted-foreground">
          {saveStatus}
        </div>
        <div className="rounded-lg bg-accent px-2 py-1 text-sm text-muted-foreground">
          {charsCount} Words
        </div>
      </div>

      <EditorRoot>
        <EditorContent
          initialContent={initialContent}
          extensions={extensions}
          className="relative min-h-[500px] w-full border-muted bg-background"
          editorProps={{
            attributes: {
              class: "prose prose-lg dark:prose-invert prose-headings:font-title font-default focus:outline-none max-w-full",
            },
          }}
          onUpdate={({ editor }) => {
            debouncedUpdates(editor);
            setSaveStatus("Unsaved");
          }}
          slotAfter={<ImageResizer />}
        >
          {/* Bubble menu components */}
          <EditorCommand className="z-50 h-auto max-h-[330px] overflow-y-auto">
            {/* Slash command menu */}
          </EditorCommand>

          <EditorBubble>
            <NodeSelector open={openNode} onOpenChange={setOpenNode} />
            <LinkSelector open={openLink} onOpenChange={setOpenLink} />
            <TextButtons />
            <ColorSelector open={openColor} onOpenChange={setOpenColor} />
          </EditorBubble>
        </EditorContent>
      </EditorRoot>
    </div>
  );
};

export default TailwindAdvancedEditor;
```

**Key Architectural Patterns**:

1. **Controlled Popover State**: Parent component manages `open` state for all popovers
   - **Why?** Ensures only one popover open at a time
   - **Pattern**: `open` + `onOpenChange` props (controlled component pattern)

2. **Debounced Auto-Save**: 500ms delay before saving
   - **Why?** Reduces localStorage writes during typing
   - **Mental Model**: Like a "save draft" feature in Gmail—waits for you to pause

3. **Multiple Export Formats**: Saves JSON, HTML, and Markdown
   - **Why?** Different formats for different use cases (storage, display, export)
   - **Pattern**: Single source of truth (editor state) → multiple derived formats

4. **Lazy Loading**: Content loaded in `useEffect` after mount
   - **Why?** Avoids SSR/client mismatch with localStorage
   - **Pattern**: Show nothing until client-side content loads

5. **Novel Component Primitives**: Uses `EditorRoot`, `EditorContent`, `EditorBubble`
   - **Why?** Headless architecture—Novel provides logic, you provide UI
   - **Mental Model**: Like Radix UI for editors—unstyled, accessible, composable

**Performance Optimization**:

```typescript
// ❌ Bad: Saves on every keystroke
onUpdate={({ editor }) => {
  localStorage.setItem("content", JSON.stringify(editor.getJSON()));
}}

// ✅ Good: Debounced save (500ms)
const debouncedUpdates = useDebouncedCallback(async (editor) => {
  localStorage.setItem("content", JSON.stringify(editor.getJSON()));
}, 500);

onUpdate={({ editor }) => {
  debouncedUpdates(editor);
  setSaveStatus("Unsaved"); // Immediate UI feedback
}}
```

---

### Selector Component Pattern

**File**: `apps/web/components/tailwind/selectors/node-selector.tsx:1-150`

```typescript
import { Check, ChevronDown } from "lucide-react";
import { EditorBubbleItem, useEditor } from "novel";
import { Popover, PopoverContent, PopoverTrigger } from "../ui/popover";
import { Button } from "../ui/button";

export type SelectorItem = {
  name: string;
  icon: LucideIcon;
  command: (editor: ReturnType<typeof useEditor>["editor"]) => void;
  isActive: (editor: ReturnType<typeof useEditor>["editor"]) => boolean;
};

const items: SelectorItem[] = [
  {
    name: "Text",
    icon: TextIcon,
    command: (editor) => editor.chain().focus().clearNodes().run(),
    isActive: (editor) =>
      editor.isActive("paragraph") &&
      !editor.isActive("bulletList") &&
      !editor.isActive("orderedList"),
  },
  {
    name: "Heading 1",
    icon: Heading1,
    command: (editor) => editor.chain().focus().toggleHeading({ level: 1 }).run(),
    isActive: (editor) => editor.isActive("heading", { level: 1 }),
  },
  // ... more items
];

interface NodeSelectorProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export const NodeSelector = ({ open, onOpenChange }: NodeSelectorProps) => {
  const { editor } = useEditor();
  if (!editor) return null;

  const activeItem = items.filter((item) => item.isActive(editor)).pop() ?? {
    name: "Multiple",
  };

  return (
    <Popover modal={true} open={open} onOpenChange={onOpenChange}>
      <PopoverTrigger asChild>
        <Button size="sm" variant="ghost" className="gap-2">
          <span className="text-sm">{activeItem.name}</span>
          <ChevronDown className="h-4 w-4" />
        </Button>
      </PopoverTrigger>

      <PopoverContent align="start" className="w-48 p-1">
        {items.map((item) => (
          <EditorBubbleItem
            key={item.name}
            onSelect={(editor) => {
              item.command(editor);
              onOpenChange(false);
            }}
            className="flex cursor-pointer items-center justify-between"
          >
            <div className="flex items-center space-x-2">
              <item.icon className="h-4 w-4" />
              <span>{item.name}</span>
            </div>
            {activeItem.name === item.name && <Check className="h-4 w-4" />}
          </EditorBubbleItem>
        ))}
      </PopoverContent>
    </Popover>
  );
};
```

**Design Patterns Explained**:

1. **Type-Safe Item Configuration**:
   ```typescript
   export type SelectorItem = {
     name: string;                    // Display name
     icon: LucideIcon;                // Icon component
     command: (editor) => void;       // What to do when clicked
     isActive: (editor) => boolean;   // When to show checkmark
   };
   ```
   **Why?** Declarative configuration—adding new items is just data, not code changes.

2. **Active Item Detection**:
   ```typescript
   const activeItem = items.filter((item) => item.isActive(editor)).pop();
   ```
   **Mental Model**: "Which item's condition is true right now?" Falls back to "Multiple" if none match.

3. **Controlled Popover**:
   ```typescript
   <Popover modal={true} open={open} onOpenChange={onOpenChange}>
   ```
   **Why `modal={true}`?** Prevents interaction with other UI while popover is open (UX consistency).

4. **EditorBubbleItem Wrapper**:
   ```typescript
   <EditorBubbleItem
     onSelect={(editor) => {
       item.command(editor);
       onOpenChange(false);  // Close popover after selection
     }}
   >
   ```
   **Pattern**: Novel's abstraction handles editor context automatically. You just provide the command.

**Reusability**: This pattern is used for:
- Node selector (Text, H1, H2, etc.)
- Color selector (text/highlight colors)
- Link selector (edit/remove links)
- AI selector (continue, improve, etc.)

---

### AI Selector Component

**File**: `apps/web/components/tailwind/generative/ai-selector.tsx:1-200`

```typescript
"use client";

import { useCompletion } from "ai/react";
import { toast } from "sonner";
import { useEditor } from "novel";
import { Command, CommandInput, CommandList, CommandGroup } from "../ui/command";
import { ScrollArea } from "../ui/scroll-area";
import { CrazySpinner } from "../ui/icons/crazy-spinner";
import { AISelectorCommands } from "./ai-selector-commands";
import { addAIHighlight } from "novel/extensions";
import Markdown from "react-markdown";

interface AISelectorProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function AISelector({ open, onOpenChange }: AISelectorProps) {
  const { editor } = useEditor();
  const [inputValue, setInputValue] = useState("");

  // Vercel AI SDK hook for streaming completions
  const { completion, complete, isLoading } = useCompletion({
    api: "/api/generate",
    onResponse: (response) => {
      if (response.status === 429) {
        toast.error("You have reached your request limit for the day.");
        return;
      }
    },
    onFinish: (_prompt, completion) => {
      // Completion finished
    },
    onError: (e) => {
      toast.error(e.message);
    },
  });

  const hasCompletion = completion.length > 0;

  return (
    <Command className="w-[350px]">
      {hasCompletion && (
        <div className="prose dark:prose-invert p-2">
          <ScrollArea>
            <Markdown>{completion}</Markdown>
          </ScrollArea>
        </div>
      )}

      {isLoading && <CrazySpinner />}

      {!isLoading && (
        <>
          <CommandInput
            value={inputValue}
            onValueChange={setInputValue}
            autoFocus
            placeholder="Ask AI to edit or generate..."
            onFocus={() => addAIHighlight(editor)}
          />

          <CommandList>
            <CommandGroup heading="Edit or review selection">
              <AISelectorCommands
                onSelect={(value, option) => {
                  complete(value, { body: { option } });
                }}
              />
            </CommandGroup>

            {inputValue && (
              <CommandGroup heading="Use AI to do more">
                <CommandItem
                  onSelect={() => {
                    complete(inputValue, { body: { option: "zap" } });
                  }}
                  value="continue"
                >
                  <Magic className="mr-2 h-4 w-4" />
                  {inputValue}
                </CommandItem>
              </CommandGroup>
            )}
          </CommandList>
        </>
      )}
    </Command>
  );
}
```

**AI Integration Patterns**:

1. **useCompletion Hook** (Vercel AI SDK):
   ```typescript
   const { completion, complete, isLoading } = useCompletion({
     api: "/api/generate",
     onResponse: (response) => { /* handle errors */ },
     onFinish: (_prompt, completion) => { /* use result */ },
   });
   ```
   **Mental Model**: Like `useState` but for AI responses. It manages:
   - ✅ Streaming state
   - ✅ Loading state
   - ✅ Error handling
   - ✅ Response accumulation

2. **Progressive Enhancement UI**:
   ```typescript
   {hasCompletion && <Markdown>{completion}</Markdown>}
   {isLoading && <CrazySpinner />}
   {!isLoading && <CommandInput />}
   ```
   **Pattern**: Show different UI based on state (idle → loading → result)

3. **AI Highlight Visual Feedback**:
   ```typescript
   onFocus={() => addAIHighlight(editor)}
   ```
   **Why?** Visual indicator showing which text will be sent to AI

4. **Dynamic Options**:
   ```typescript
   complete(value, { body: { option: "improve" } })
   ```
   **Pattern**: Same endpoint, different `option` parameter (continue, improve, fix, etc.)

**Error Handling**:
- Rate limiting (429) → Toast notification
- Network errors → Toast with error message
- Server errors → Caught by `onError` callback

---

## Styling Architecture

### Tailwind CSS Configuration

**File**: `apps/web/tailwind.config.ts:1-100`

```typescript
import type { Config } from "tailwindcss";

const config = {
  darkMode: ["class"],  // Use class-based dark mode
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [
    require("tailwindcss-animate"),
    require("@tailwindcss/typography"),
  ],
} satisfies Config;

export default config;
```

**Key Architectural Decisions**:

1. **CSS Variable-Based Colors**:
   ```typescript
   colors: {
     background: "hsl(var(--background))",  // References CSS variable
     foreground: "hsl(var(--foreground))",
   }
   ```
   **Why HSL?** Easier to manipulate (adjust lightness for hover states)

   **Mental Model**: Tailwind classes (`bg-background`) → CSS variables (`--background`) → Actual color values (defined in `globals.css`)

2. **Semantic Color Naming**:
   - `background` / `foreground` - Base colors
   - `primary` / `secondary` - Brand colors
   - `muted` / `accent` - Supporting colors
   - `destructive` - Error/danger states

   **Why?** Theme-agnostic naming. `primary` works for any brand color.

3. **Dynamic Border Radius**:
   ```typescript
   borderRadius: {
     lg: "var(--radius)",
     md: "calc(var(--radius) - 2px)",
     sm: "calc(var(--radius) - 4px)",
   }
   ```
   **Why?** Single source of truth. Change `--radius` to adjust all components.

4. **Plugins**:
   - `tailwindcss-animate`: Adds animation utilities
   - `@tailwindcss/typography`: Prose styling for rich text

---

### CSS Variables System

**File**: `apps/web/styles/globals.css:1-150`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Base colors */
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    /* Card colors */
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    /* Popover colors */
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    /* Primary brand */
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;

    /* Secondary brand */
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    /* Muted (subtle backgrounds) */
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;

    /* Accent (hover states) */
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;

    /* Destructive (errors) */
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    /* Borders */
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;

    /* Border radius */
    --radius: 0.5rem;

    /* Novel-specific highlights */
    --novel-highlight-default: #ffffff;
    --novel-highlight-purple: #f6f3f8;
    --novel-highlight-red: #fdebeb;
    --novel-highlight-yellow: #fbf4a2;
    --novel-highlight-blue: #c1ecf9;
    --novel-highlight-green: #acf79f;
    --novel-highlight-orange: #faebdd;
    --novel-highlight-pink: #faf1f5;
    --novel-highlight-gray: #f1f1ef;
  }

  .dark {
    /* Inverted colors for dark mode */
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;

    /* Dark mode highlights */
    --novel-highlight-default: #000000;
    --novel-highlight-purple: #3f2c4b;
    --novel-highlight-red: #5c1a1a;
    --novel-highlight-yellow: #5c4b1a;
    --novel-highlight-blue: #1a3d5c;
    --novel-highlight-green: #1a5c20;
    --novel-highlight-orange: #5c3a1a;
    --novel-highlight-pink: #5c1a3a;
    --novel-highlight-gray: #3a3a3a;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

**Theme System Architecture**:

```
User toggles theme
  ↓
next-themes adds/removes .dark class on <html>
  ↓
CSS variables update automatically
  ↓
Tailwind classes reference new variable values
  ↓
All components re-styled instantly
```

**Mental Model**: Think of CSS variables as **theme slots**. The `.dark` class swaps which values fill those slots.

**Why This Works**:

1. **Single Source of Truth**: Change `--primary` in one place, affects all `bg-primary` classes
2. **Automatic Dark Mode**: No duplicate class names (`bg-white dark:bg-black`)
3. **Easier Theming**: Want a blue theme? Just change variable values
4. **Type-Safe**: Tailwind config references these variables

**Analogy**: Like a **color palette** in design software. You define colors once, use them everywhere, and swap palettes instantly.

---

### ProseMirror-Specific Styling

**File**: `apps/web/styles/prosemirror.css:1-400`

```css
/* Placeholder text */
.ProseMirror .is-editor-empty:first-child::before {
  content: attr(data-placeholder);
  float: left;
  color: hsl(var(--muted-foreground));
  pointer-events: none;
  height: 0;
}

/* Selected node outline */
.ProseMirror .ProseMirror-selectednode {
  outline: 3px solid #68cef8;
}

/* Image interactions */
.ProseMirror img {
  transition: filter 0.1s ease-in-out;

  &:hover {
    cursor: pointer;
    filter: brightness(90%);
  }

  &.ProseMirror-selectednode {
    outline: 3px solid #5abbf7;
    filter: brightness(90%);
  }
}

/* Task list styling */
ul[data-type="taskList"] {
  list-style: none;
  padding: 0;

  p {
    margin: 0;
  }

  li {
    display: flex;
    align-items: flex-start;

    > label {
      flex: 0 0 auto;
      margin-right: 0.5rem;
      user-select: none;
    }

    > div {
      flex: 1 1 auto;
    }
  }
}

/* Custom checkbox */
ul[data-type="taskList"] li > label input[type="checkbox"] {
  -webkit-appearance: none;
  appearance: none;
  background-color: hsl(var(--background));
  margin: 0;
  width: 1.2em;
  height: 1.2em;
  border: 2px solid hsl(var(--border));
  border-radius: 0.25em;
  display: grid;
  place-content: center;
  cursor: pointer;

  &::before {
    content: "";
    width: 0.65em;
    height: 0.65em;
    transform: scale(0);
    transition: 120ms transform ease-in-out;
    background-color: hsl(var(--primary));
    clip-path: polygon(14% 44%, 0 65%, 50% 100%, 100% 16%, 80% 0%, 43% 62%);
  }

  &:checked::before {
    transform: scale(1);
  }

  &:hover {
    border-color: hsl(var(--primary));
  }
}

/* Drag handle */
.drag-handle {
  position: fixed;
  opacity: 1;
  transition: opacity ease-in 0.2s;
  border-radius: 0.25rem;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 10 10' style='fill: rgba(0, 0, 0, 0.5)'%3E%3Cpath d='M3,2 C2.44771525,2 2,1.55228475 2,1 C2,0.44771525 2.44771525,0 3,0 C3.55228475,0 4,0.44771525 4,1 C4,1.55228475 3.55228475,2 3,2 Z M3,6 C2.44771525,6 2,5.55228475 2,5 C2,4.44771525 2.44771525,4 3,4 C3.55228475,4 4,4.44771525 4,5 C4,5.55228475 3.55228475,6 3,6 Z M3,10 C2.44771525,10 2,9.55228475 2,9 C2,8.44771525 2.44771525,8 3,8 C3.55228475,8 4,8.44771525 4,9 C4,9.55228475 3.55228475,10 3,10 Z M7,2 C6.44771525,2 6,1.55228475 6,1 C6,0.44771525 6.44771525,0 7,0 C7.55228475,0 8,0.44771525 8,1 C8,1.55228475 7.55228475,2 7,2 Z M7,6 C6.44771525,6 6,5.55228475 6,5 C6,4.44771525 6.44771525,4 7,4 C7.55228475,4 8,4.44771525 8,5 C8,5.55228475 7.55228475,6 7,6 Z M7,10 C6.44771525,10 6,9.55228475 6,9 C6,8.44771525 6.44771525,8 7,8 C7.55228475,8 8,8.44771525 8,9 C8,9.55228475 7.55228475,10 7,10 Z'%3E%3C/path%3E%3C/svg%3E");
  background-size: calc(0.5em + 0.375rem) calc(0.5em + 0.375rem);
  background-repeat: no-repeat;
  background-position: center;
  width: 1.2rem;
  height: 1.5rem;
  z-index: 50;
  cursor: grab;

  &:hover {
    background-color: hsl(var(--accent));
  }

  &:active {
    cursor: grabbing;
  }

  @media screen and (max-width: 600px) {
    display: none;
    pointer-events: none;
  }
}

/* AI highlight */
mark[data-type="ai-highlight"] {
  background-color: var(--novel-highlight-yellow);
  border-radius: 0.25rem;
  padding: 0.125rem 0.25rem;
}
```

**Key Patterns**:

1. **Custom Placeholder**: Uses `::before` pseudo-element with `data-placeholder` attribute
2. **Visual Feedback**: Hover/selected states for images
3. **Custom Form Controls**: Styled checkbox using `appearance: none`
4. **Drag Handle**: SVG background image for consistent look
5. **Responsive**: Drag handle hidden on mobile

**Mental Model**: ProseMirror generates HTML structure, this CSS styles it to match your design system.

---

## shadcn/ui Integration

### What is shadcn/ui?

**Not a component library**, but a **collection of copy-pasteable components** built with:
- ✅ Radix UI primitives (accessible, unstyled)
- ✅ Tailwind CSS styling
- ✅ TypeScript
- ✅ CVA for variants

**Mental Model**: Like a **starter kit** for components. You copy the code into your project and customize it.

### Component Anatomy

**File**: `apps/web/components/tailwind/ui/button.tsx:1-60`

```typescript
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

// Define variants with CVA
const buttonVariants = cva(
  // Base styles (always applied)
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = "Button";

export { Button, buttonVariants };
```

**Pattern Breakdown**:

1. **CVA (Class Variance Authority)**:
   ```typescript
   const buttonVariants = cva(
     "base-styles",  // Always applied
     {
       variants: {
         variant: { default: "...", destructive: "..." },
         size: { sm: "...", lg: "..." },
       },
       defaultVariants: { variant: "default", size: "default" },
     }
   );
   ```
   **What it does**: Type-safe variant management. Like styled-components, but with Tailwind.

   **Mental Model**: Think of it as a **function that returns class names** based on props.

2. **Polymorphic Component with Slot**:
   ```typescript
   const Comp = asChild ? Slot : "button";
   <Comp className={...} ref={ref} {...props} />
   ```
   **Why?** Allows rendering as different element:
   ```tsx
   <Button>Click me</Button>           // Renders <button>
   <Button asChild><a href="/">Home</a></Button>  // Renders <a> styled as button
   ```
   **Pattern**: Radix UI's Slot merges props onto child element.

3. **forwardRef for DOM Access**:
   ```typescript
   const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(...)
   ```
   **Why?** Allows parent components to access the button DOM node:
   ```tsx
   const buttonRef = useRef<HTMLButtonElement>(null);
   <Button ref={buttonRef}>Click</Button>
   buttonRef.current?.focus();
   ```

4. **cn() Utility**:
   ```typescript
   className={cn(buttonVariants({ variant, size, className }))}
   ```
   **What it does**: Merges Tailwind classes intelligently (later classes override earlier ones).

   **Example**:
   ```typescript
   cn("bg-red-500", "bg-blue-500")  // → "bg-blue-500" (blue wins)
   ```

**Usage Examples**:

```tsx
// Default button
<Button>Click me</Button>

// Destructive variant
<Button variant="destructive">Delete</Button>

// Small ghost button
<Button variant="ghost" size="sm">Cancel</Button>

// Button as link
<Button asChild>
  <a href="/home">Go Home</a>
</Button>

// Custom classes (merged intelligently)
<Button className="w-full">Full Width</Button>
```

---

### Popover Component Pattern

**File**: `apps/web/components/tailwind/ui/popover.tsx:1-40`

```typescript
import * as React from "react";
import * as PopoverPrimitive from "@radix-ui/react-popover";
import { cn } from "@/lib/utils";

const Popover = PopoverPrimitive.Root;
const PopoverTrigger = PopoverPrimitive.Trigger;

const PopoverContent = React.forwardRef<
  React.ElementRef<typeof PopoverPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof PopoverPrimitive.Content>
>(({ className, align = "center", sideOffset = 4, ...props }, ref) => (
  <PopoverPrimitive.Portal>
    <PopoverPrimitive.Content
      ref={ref}
      align={align}
      sideOffset={sideOffset}
      className={cn(
        "z-50 w-72 rounded-md border bg-popover p-4 text-popover-foreground shadow-md outline-none data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=open]:zoom-in-95 data-[state=closed]:zoom-out-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
        className
      )}
      {...props}
    />
  </PopoverPrimitive.Portal>
));
PopoverContent.displayName = PopoverPrimitive.Content.displayName;

export { Popover, PopoverTrigger, PopoverContent };
```

**Architectural Insights**:

1. **Radix UI Wrapper Pattern**:
   ```typescript
   const Popover = PopoverPrimitive.Root;  // Re-export as-is
   const PopoverTrigger = PopoverPrimitive.Trigger;  // Re-export as-is
   const PopoverContent = /* custom wrapper */;  // Wrap with styling
   ```
   **Why?** Root and Trigger don't need styling, but Content does.

2. **Portal for Proper Stacking**:
   ```typescript
   <PopoverPrimitive.Portal>
     <PopoverPrimitive.Content>...</PopoverPrimitive.Content>
   </PopoverPrimitive.Portal>
   ```
   **What it does**: Renders content at end of `<body>`, avoiding z-index issues.

   **Mental Model**: Like React's `createPortal()` but built-in to Radix UI.

3. **Data Attributes for Animations**:
   ```css
   data-[state=open]:animate-in
   data-[state=closed]:animate-out
   data-[side=bottom]:slide-in-from-top-2
   ```
   **Pattern**: Radix UI adds data attributes, Tailwind styles based on them.

4. **Default Props**:
   ```typescript
   align = "center"      // Center by default
   sideOffset = 4        // 4px gap from trigger
   ```

**Usage**:

```tsx
<Popover>
  <PopoverTrigger asChild>
    <Button>Open</Button>
  </PopoverTrigger>
  <PopoverContent>
    <p>Popover content here</p>
  </PopoverContent>
</Popover>
```

---

## API Routes (Edge Runtime)

### Edge Runtime Basics

**What is Edge Runtime?**

- ✅ Runs in V8 isolates (like Cloudflare Workers)
- ✅ Deployed to edge locations globally
- ✅ Lower latency (runs closer to users)
- ✅ Limited APIs (no Node.js fs, buffer, etc.)

**Mental Model**: Like serverless functions, but **lighter and faster**. Think of it as JavaScript running in a browser-like environment on the server.

---

### AI Generation Route

**File**: `apps/web/app/api/generate/route.ts:1-200`

```typescript
import { OpenAIStream, StreamingTextResponse } from "ai";
import { match } from "ts-pattern";

// Edge runtime for low latency
export const runtime = "edge";

export async function POST(req: Request): Promise<Response> {
  const { prompt, option } = await req.json();

  // Pattern matching for different AI operations
  const systemMessage = match(option)
    .with("continue", () =>
      "You are an AI writing assistant that continues existing text..."
    )
    .with("improve", () =>
      "You are an AI writing assistant that improves existing text..."
    )
    .with("shorter", () =>
      "You are an AI writing assistant that shortens existing text..."
    )
    .with("longer", () =>
      "You are an AI writing assistant that lengthens existing text..."
    )
    .with("fix", () =>
      "You are an AI writing assistant that fixes grammar and spelling..."
    )
    .with("zap", () =>
      "You are an AI writing assistant..."
    )
    .otherwise(() => "You are an AI writing assistant...");

  // Call OpenAI API
  const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    stream: true,
    messages: [
      { role: "system", content: systemMessage },
      { role: "user", content: prompt },
    ],
  });

  // Convert to ReadableStream
  const stream = OpenAIStream(response);

  // Return streaming response
  return new StreamingTextResponse(stream);
}
```

**Key Patterns**:

1. **ts-pattern for Clean Conditionals**:
   ```typescript
   const systemMessage = match(option)
     .with("continue", () => "...")
     .with("improve", () => "...")
     .otherwise(() => "...");
   ```
   **Why?** More readable than nested if/else. Type-safe exhaustiveness checking.

   **Mental Model**: Like a switch statement on steroids.

2. **Streaming Responses**:
   ```typescript
   const stream = OpenAIStream(response);
   return new StreamingTextResponse(stream);
   ```
   **What it does**: Sends chunks as they arrive from OpenAI (progressive rendering).

   **Mental Model**: Like `console.log()` appearing line-by-line instead of all at once.

3. **Edge Runtime Benefits**:
   - ⚡ Faster cold starts (~50ms vs ~500ms)
   - 🌍 Runs globally (lower latency)
   - 💰 Cheaper than full Node.js runtime

**How Streaming Works**:

```
User clicks "Continue writing"
  ↓
Frontend calls /api/generate
  ↓
Edge function calls OpenAI
  ↓
OpenAI sends first chunk → Edge forwards → Frontend renders
  ↓
OpenAI sends second chunk → Edge forwards → Frontend updates
  ↓
... (continues until complete)
  ↓
Stream closes
```

---

### Image Upload Route

**File**: `apps/web/app/api/upload/route.ts:1-50`

```typescript
import { put } from "@vercel/blob";

export const runtime = "edge";

export async function POST(req: Request): Promise<Response> {
  const file = req.body || "";
  const contentType = req.headers.get("content-type") || "image/png";
  const filename = `${Date.now()}.${contentType.split("/")[1]}`;

  const blob = await put(filename, file, {
    contentType,
    access: "public",
  });

  return Response.json(blob);
}
```

**Key Patterns**:

1. **Vercel Blob Storage**:
   ```typescript
   const blob = await put(filename, file, { access: "public" });
   ```
   **What it does**: Uploads to Vercel's CDN, returns public URL.

   **Mental Model**: Like uploading to S3, but simpler.

2. **Dynamic Filename**:
   ```typescript
   const filename = `${Date.now()}.${contentType.split("/")[1]}`;
   ```
   **Pattern**: Timestamp + file extension (e.g., `1699564800000.png`)

3. **Content-Type Header**:
   ```typescript
   const contentType = req.headers.get("content-type") || "image/png";
   ```
   **Why?** Ensures proper file type detection and serving.

---

## State Management Patterns

Novel uses multiple state management strategies:

### 1. React Context (Font Preferences)

**File**: `apps/web/app/providers.tsx:10-20`

```typescript
export const AppContext = createContext<{
  font: string;
  setFont: Dispatch<SetStateAction<string>>;
}>({
  font: "Default",
  setFont: () => {},
});

export default function Providers({ children }: { children: ReactNode }) {
  const [font, setFont] = useLocalStorage<string>("novel__font", "Default");

  return (
    <AppContext.Provider value={{ font, setFont }}>
      {children}
    </AppContext.Provider>
  );
}
```

**When to Use**: Global settings that need to persist (fonts, theme, user preferences)

---

### 2. Local Component State

```typescript
const [openNode, setOpenNode] = useState(false);
const [openColor, setOpenColor] = useState(false);
```

**When to Use**: UI state scoped to a component (popover visibility, form inputs)

---

### 3. localStorage Persistence

```typescript
const [font, setFont] = useLocalStorage<string>("novel__font", "Default");
```

**When to Use**: Persist state across page refreshes

---

### 4. Editor State (Tiptap/Novel)

```typescript
const { editor } = useEditor();  // From Novel's context
```

**When to Use**: Document content, selection, formatting state

---

## Performance Optimizations

### 1. Debounced Auto-Save

```typescript
const debouncedUpdates = useDebouncedCallback(async (editor) => {
  localStorage.setItem("content", JSON.stringify(editor.getJSON()));
}, 500);
```

**Impact**: Reduces localStorage writes from ~100/sec to ~2/sec during typing.

---

### 2. Lazy Loading Content

```typescript
useEffect(() => {
  const content = window.localStorage.getItem("novel-content");
  if (content) setInitialContent(JSON.parse(content));
}, []);

if (!initialContent) return null;
```

**Impact**: Avoids SSR hydration mismatches with localStorage.

---

### 3. Edge Runtime for API Routes

```typescript
export const runtime = "edge";
```

**Impact**: ~10x faster cold starts, lower latency globally.

---

### 4. Streaming AI Responses

```typescript
const stream = OpenAIStream(response);
return new StreamingTextResponse(stream);
```

**Impact**: Time to first byte ~100ms vs ~2s for full completion.

---

### 5. CSS Variable Theming

```css
:root { --background: 0 0% 100%; }
.dark { --background: 222.2 84% 4.9%; }
```

**Impact**: No JavaScript re-renders for theme changes. Pure CSS swapping.

---

## Best Practices & Patterns

### 1. Import Organization

```typescript
// 1. External dependencies
import { useState } from "react";
import { useEditor } from "novel";

// 2. Internal components
import { Button } from "@/components/ui/button";

// 3. Types
import type { EditorInstance } from "novel";

// 4. Utilities
import { cn } from "@/lib/utils";

// 5. Styles (if any)
import "./styles.css";
```

---

### 2. Component File Structure

```typescript
// 1. Imports
import { ... } from "...";

// 2. Types
export interface ButtonProps { ... }

// 3. Constants/Config
const buttonVariants = cva(...);

// 4. Component
const Button = forwardRef<...>((...) => { ... });

// 5. Display name (for DevTools)
Button.displayName = "Button";

// 6. Exports
export { Button, buttonVariants };
```

---

### 3. Controlled vs Uncontrolled

**Controlled (Novel's pattern)**:
```tsx
<Popover open={open} onOpenChange={setOpenChange}>
```

**Why?** Parent manages state, enabling:
- Close other popovers when one opens
- Programmatic control
- Easier testing

**Uncontrolled**:
```tsx
<Popover>  {/* Manages its own state */}
```

**When?** Simple cases with no coordination needed.

---

### 4. Props Spreading Pattern

```typescript
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => (
    <button className={cn(...)} ref={ref} {...props} />
  )
);
```

**Why?** Allows all native button props (onClick, disabled, aria-*, etc.) without listing them.

---

### 5. TypeScript Inference

```typescript
// ❌ Verbose
const items: Array<SelectorItem> = [...];

// ✅ Inferred
const items: SelectorItem[] = [...];

// ✅ Even better (type inferred from usage)
const items = [
  { name: "Text", icon: TextIcon, ... },
] satisfies SelectorItem[];
```

---

## Architecture Decision Records

### Why Next.js App Router?

**Decision**: Use App Router over Pages Router

**Reasons**:
- ✅ Server components reduce client bundle
- ✅ Better streaming support
- ✅ Simpler data fetching patterns
- ✅ Future direction of Next.js

**Trade-offs**:
- ⚠️ Newer API (less Stack Overflow answers)
- ⚠️ Client components need explicit marking

---

### Why Tailwind + CSS Variables?

**Decision**: CSS variables for colors instead of Tailwind's default system

**Reasons**:
- ✅ Single source of truth for theming
- ✅ Instant theme switching (no JS)
- ✅ Easier to maintain brand colors

**Trade-offs**:
- ⚠️ Extra layer of indirection
- ⚠️ Need to define variables in multiple places

---

### Why shadcn/ui Over Component Library?

**Decision**: Copy-paste components vs install library

**Reasons**:
- ✅ Full ownership of code
- ✅ No version lock-in
- ✅ Customize without fighting library
- ✅ Tree-shaking built-in

**Trade-offs**:
- ⚠️ Manual updates for bug fixes
- ⚠️ More initial setup

---

### Why Edge Runtime for AI Routes?

**Decision**: Edge over Node.js runtime

**Reasons**:
- ✅ Lower latency (global distribution)
- ✅ Faster cold starts
- ✅ Better for streaming

**Trade-offs**:
- ⚠️ Limited APIs (no fs, buffer)
- ⚠️ Smaller max response size

---

## Summary & Key Takeaways

### Frontend Architecture Philosophy

Novel's frontend demonstrates:

1. **Server-First Mindset**: Default to server components, opt into client
2. **Type-Safe Styling**: CVA + Tailwind for variant management
3. **Accessible by Default**: Radix UI primitives ensure WCAG compliance
4. **Performance-Conscious**: Edge runtime, streaming, debouncing
5. **Developer Experience**: Clear patterns, co-located code, type safety

### Patterns to Master

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Controlled Components** | Coordinated UI state | Popover visibility |
| **Debouncing** | Rate-limiting expensive ops | Auto-save |
| **CVA Variants** | Type-safe component styles | Button variants |
| **CSS Variables** | Dynamic theming | Dark mode |
| **Streaming Responses** | Progressive rendering | AI completions |
| **forwardRef** | DOM access from parent | Focus management |
| **Polymorphic Components** | Flexible rendering | Button as link |

### Next Steps

1. **Explore**: Read component source code in `components/tailwind/`
2. **Experiment**: Modify variants in `button.tsx`
3. **Build**: Create a new selector component
4. **Optimize**: Add your own debounced feature
5. **Theme**: Customize CSS variables for your brand

---

## Additional Resources

### Official Documentation

- **Next.js 15**: https://nextjs.org/docs
- **Tailwind CSS**: https://tailwindcss.com/docs
- **Radix UI**: https://www.radix-ui.com/primitives/docs/overview/introduction
- **CVA**: https://cva.style/docs
- **Vercel AI SDK**: https://sdk.vercel.ai/docs

### Novel-Specific

- **Tiptap Extensions**: See EDITOR_CORE_GUIDE.md (coming next)
- **State Management**: See STATE_MANAGEMENT.md (coming next)
- **Integration Guide**: See INTEGRATION_GUIDE.md (coming next)

---

**Next Document**: EDITOR_CORE_GUIDE.md - Deep dive into Tiptap, ProseMirror, and the extension system.
