# Integration Guide: Using Novel in Your Project

> **Target Audience**: Developers integrating Novel into their applications
>
> **Prerequisites**: Basic React knowledge, package manager familiarity
>
> **What You'll Learn**: Installation, configuration, framework-specific setup, styling, and common integration patterns

---

## Table of Contents

1. [Installation](#installation)
2. [Quick Start](#quick-start)
3. [Framework Integration](#framework-integration)
4. [Component API](#component-api)
5. [Extension Configuration](#extension-configuration)
6. [Styling Approaches](#styling-approaches)
7. [AI Features Setup](#ai-features-setup)
8. [Image Upload Setup](#image-upload-setup)
9. [Common Patterns](#common-patterns)
10. [Troubleshooting](#troubleshooting)

---

## Installation

### Package Installation

```bash
# npm
npm install novel

# pnpm
pnpm add novel

# yarn
yarn add novel
```

### Peer Dependencies

Novel requires **React 18+**:

```json
{
  "peerDependencies": {
    "react": ">=18"
  }
}
```

If you don't have React 18, upgrade first:

```bash
npm install react@^18 react-dom@^18
```

---

### What Gets Installed?

When you install `novel`, you get:

**Components**:
- `EditorRoot` - Context provider
- `EditorContent` - Editor wrapper
- `EditorBubble` - Selection toolbar
- `EditorCommand` - Slash command menu
- Utility components

**Extensions**:
- Tiptap extensions (bold, italic, heading, etc.)
- Custom extensions (AI highlight, image upload, etc.)
- Mathematics, Twitter, YouTube embeds

**Utilities**:
- Image upload helpers
- Slash command utilities
- State management (Jotai atoms)

**Total Bundle Size**: ~150KB minified (tree-shakeable)

---

## Quick Start

### Minimal Example

```tsx
import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "./extensions";

export default function MyEditor() {
  return (
    <EditorRoot>
      <EditorContent
        extensions={defaultExtensions}
        className="border rounded-lg p-4"
      />
    </EditorRoot>
  );
}
```

**That's it!** You have a working editor.

---

### Basic Extensions Setup

**File**: `extensions.ts`

```typescript
import {
  StarterKit,
  Placeholder,
  TiptapLink,
  TiptapImage,
  MarkdownExtension,
  Highlight,
} from "novel";

export const defaultExtensions = [
  StarterKit.configure({
    bulletList: {
      HTMLAttributes: {
        class: "list-disc list-outside leading-3 -mt-2",
      },
    },
    orderedList: {
      HTMLAttributes: {
        class: "list-decimal list-outside leading-3 -mt-2",
      },
    },
    listItem: {
      HTMLAttributes: {
        class: "leading-normal -mb-2",
      },
    },
    code: {
      HTMLAttributes: {
        class: "rounded-md bg-muted px-1.5 py-1 font-mono font-medium",
      },
    },
  }),

  Placeholder.configure({
    placeholder: ({ node }) => {
      if (node.type.name === "heading") {
        return "What's the title?";
      }
      return "Press '/' for commands...";
    },
  }),

  TiptapLink.configure({
    HTMLAttributes: {
      class: "text-primary underline underline-offset-4",
    },
  }),

  TiptapImage.configure({
    HTMLAttributes: {
      class: "rounded-lg border border-muted",
    },
  }),

  Highlight,
  MarkdownExtension,
];
```

---

### With Slash Commands

```tsx
import {
  EditorRoot,
  EditorContent,
  EditorCommand,
  EditorCommandList,
  EditorCommandItem,
  createSuggestionItems,
} from "novel";
import { defaultExtensions } from "./extensions";

const suggestionItems = createSuggestionItems([
  {
    title: "Text",
    description: "Just start typing with plain text.",
    icon: <TextIcon />,
    command: ({ editor, range }) => {
      editor.chain().focus().deleteRange(range).clearNodes().run();
    },
  },
  {
    title: "Heading 1",
    description: "Big section heading.",
    icon: <Heading1Icon />,
    command: ({ editor, range }) => {
      editor
        .chain()
        .focus()
        .deleteRange(range)
        .setNode("heading", { level: 1 })
        .run();
    },
  },
  // ... more items
]);

export default function MyEditor() {
  return (
    <EditorRoot>
      <EditorContent extensions={defaultExtensions}>
        <EditorCommand className="z-50 h-auto max-h-[330px]">
          <EditorCommandList>
            {suggestionItems.map((item) => (
              <EditorCommandItem
                key={item.title}
                onCommand={item.command}
              >
                {item.icon}
                <div>
                  <p>{item.title}</p>
                  <p className="text-sm text-muted-foreground">
                    {item.description}
                  </p>
                </div>
              </EditorCommandItem>
            ))}
          </EditorCommandList>
        </EditorCommand>
      </EditorContent>
    </EditorRoot>
  );
}
```

---

## Framework Integration

### Next.js (App Router)

**File**: `app/editor/page.tsx`

```tsx
"use client";

import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "@/lib/extensions";
import { useState } from "react";
import type { JSONContent } from "novel";

export default function EditorPage() {
  const [content, setContent] = useState<JSONContent | null>(null);
  const [saveStatus, setSaveStatus] = useState("Saved");

  return (
    <div className="container max-w-3xl py-10">
      <div className="mb-4 flex items-center justify-between">
        <h1 className="text-2xl font-bold">Editor</h1>
        <span className="text-sm text-muted-foreground">{saveStatus}</span>
      </div>

      <EditorRoot>
        <EditorContent
          initialContent={content}
          extensions={defaultExtensions}
          onUpdate={({ editor }) => {
            setSaveStatus("Unsaved");

            // Debounce save logic here
            const json = editor.getJSON();
            setContent(json);

            // Save to database
            // await saveToDatabase(json);

            setSaveStatus("Saved");
          }}
          editorProps={{
            attributes: {
              class: "prose prose-lg dark:prose-invert focus:outline-none max-w-full",
            },
          }}
        />
      </EditorRoot>
    </div>
  );
}
```

**CSS Imports** (required):

**File**: `app/layout.tsx`

```tsx
import "./globals.css";
import "novel/styles.css";  // ← Novel styles

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

---

### Next.js (Pages Router)

**File**: `pages/editor.tsx`

```tsx
import dynamic from "next/dynamic";

// Dynamically import editor (no SSR)
const Editor = dynamic(() => import("@/components/Editor"), {
  ssr: false,
  loading: () => <div>Loading editor...</div>,
});

export default function EditorPage() {
  return (
    <div>
      <h1>My Editor</h1>
      <Editor />
    </div>
  );
}
```

**File**: `components/Editor.tsx`

```tsx
"use client";

import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "@/lib/extensions";

export default function Editor() {
  return (
    <EditorRoot>
      <EditorContent extensions={defaultExtensions} />
    </EditorRoot>
  );
}
```

**Why `ssr: false`?**
- Novel uses browser-only APIs (window, localStorage)
- Tiptap needs DOM to initialize
- Dynamic import prevents SSR errors

---

### Vite + React

**File**: `src/App.tsx`

```tsx
import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "./extensions";
import "novel/styles.css";
import "./index.css";

function App() {
  return (
    <div className="container mx-auto py-10">
      <EditorRoot>
        <EditorContent
          extensions={defaultExtensions}
          className="border rounded-lg p-4"
        />
      </EditorRoot>
    </div>
  );
}

export default App;
```

**Vite Config** (for proper HMR):

**File**: `vite.config.ts`

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  optimizeDeps: {
    include: ["novel"],
  },
});
```

---

### Remix

**File**: `app/routes/editor.tsx`

```tsx
import { ClientOnly } from "remix-utils/client-only";
import Editor from "~/components/Editor.client";

export default function EditorRoute() {
  return (
    <div className="container py-10">
      <h1>Editor</h1>
      <ClientOnly fallback={<div>Loading editor...</div>}>
        {() => <Editor />}
      </ClientOnly>
    </div>
  );
}
```

**File**: `app/components/Editor.client.tsx`

```tsx
import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "~/lib/extensions";
import "novel/styles.css";

export default function Editor() {
  return (
    <EditorRoot>
      <EditorContent extensions={defaultExtensions} />
    </EditorRoot>
  );
}
```

**Note**: `.client.tsx` suffix ensures client-only rendering.

---

### Astro

**File**: `src/pages/editor.astro`

```astro
---
import Layout from "../layouts/Layout.astro";
---

<Layout title="Editor">
  <div id="editor-root"></div>
</Layout>

<script>
  import { createRoot } from "react-dom/client";
  import Editor from "../components/Editor";

  const root = document.getElementById("editor-root");
  if (root) {
    createRoot(root).render(<Editor />);
  }
</script>

<style is:global>
  @import "novel/styles.css";
</style>
```

**File**: `src/components/Editor.tsx`

```tsx
import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "../lib/extensions";

export default function Editor() {
  return (
    <EditorRoot>
      <EditorContent extensions={defaultExtensions} />
    </EditorRoot>
  );
}
```

---

## Component API

### EditorRoot

**Purpose**: Provides Jotai state and tunnel context.

```tsx
interface EditorRootProps {
  children: React.ReactNode;
}

<EditorRoot>
  {children}
</EditorRoot>
```

**Key Points**:
- ✅ Required wrapper for all editor components
- ✅ Provides isolated state scope
- ✅ Can render multiple instances (each isolated)

**Multiple Editors**:

```tsx
// Each editor has independent state
<EditorRoot>
  <EditorContent />  {/* Editor 1 */}
</EditorRoot>

<EditorRoot>
  <EditorContent />  {/* Editor 2 */}
</EditorRoot>
```

---

### EditorContent

**Purpose**: Main editor component.

```tsx
interface EditorContentProps {
  initialContent?: JSONContent;
  extensions: Extension[];
  editorProps?: EditorProps;
  onUpdate?: ({ editor }: { editor: EditorInstance }) => void;
  onCreate?: ({ editor }: { editor: EditorInstance }) => void;
  onBlur?: ({ editor }: { editor: EditorInstance }) => void;
  onFocus?: ({ editor }: { editor: EditorInstance }) => void;
  slotAfter?: React.ReactNode;
  slotBefore?: React.ReactNode;
  children?: React.ReactNode;
  className?: string;
}

<EditorContent
  initialContent={content}
  extensions={extensions}
  onUpdate={({ editor }) => {
    const json = editor.getJSON();
    console.log("Content updated:", json);
  }}
  onCreate={({ editor }) => {
    console.log("Editor created:", editor);
  }}
  editorProps={{
    attributes: {
      class: "prose prose-lg focus:outline-none",
    },
    handleDOMEvents: {
      keydown: (view, event) => {
        // Custom keyboard handling
        return false;
      },
    },
  }}
  className="border rounded-lg p-4"
>
  {children}
</EditorContent>
```

**Props Explained**:

| Prop | Type | Description |
|------|------|-------------|
| `initialContent` | `JSONContent` | Initial document (Tiptap JSON format) |
| `extensions` | `Extension[]` | Array of Tiptap extensions |
| `editorProps` | `EditorProps` | ProseMirror editor props |
| `onUpdate` | `Function` | Called on every content change |
| `onCreate` | `Function` | Called when editor initializes |
| `onBlur` | `Function` | Called when editor loses focus |
| `onFocus` | `Function` | Called when editor gains focus |
| `slotAfter` | `ReactNode` | Rendered after editor (e.g., ImageResizer) |
| `slotBefore` | `ReactNode` | Rendered before editor |
| `children` | `ReactNode` | UI components (EditorBubble, EditorCommand) |
| `className` | `string` | CSS classes for wrapper div |

---

### EditorBubble

**Purpose**: Selection toolbar (appears when text is selected).

```tsx
import {
  EditorBubble,
  EditorBubbleItem,
  useEditor,
} from "novel";

<EditorBubble>
  <EditorBubbleItem
    onSelect={(editor) => {
      editor.chain().focus().toggleBold().run();
    }}
  >
    <BoldIcon />
  </EditorBubbleItem>

  <EditorBubbleItem
    onSelect={(editor) => {
      editor.chain().focus().toggleItalic().run();
    }}
  >
    <ItalicIcon />
  </EditorBubbleItem>
</EditorBubble>
```

**Advanced Example** (with active state):

```tsx
const BoldButton = () => {
  const { editor } = useEditor();

  if (!editor) return null;

  return (
    <EditorBubbleItem
      onSelect={() => editor.chain().focus().toggleBold().run()}
      className={editor.isActive("bold") ? "is-active" : ""}
    >
      <BoldIcon />
    </EditorBubbleItem>
  );
};

<EditorBubble>
  <BoldButton />
  {/* ... more buttons */}
</EditorBubble>
```

---

### EditorCommand

**Purpose**: Slash command menu.

```tsx
import {
  EditorCommand,
  EditorCommandList,
  EditorCommandItem,
  EditorCommandEmpty,
} from "novel";

<EditorCommand className="z-50 h-auto max-h-[330px]">
  <EditorCommandList>
    {suggestionItems.map((item) => (
      <EditorCommandItem
        key={item.title}
        onCommand={({ editor, range }) => {
          item.command({ editor, range });
        }}
      >
        <div className="flex items-center gap-2">
          {item.icon}
          <div>
            <p className="font-medium">{item.title}</p>
            <p className="text-sm text-muted-foreground">
              {item.description}
            </p>
          </div>
        </div>
      </EditorCommandItem>
    ))}
  </EditorCommandList>

  <EditorCommandEmpty>
    <p>No results found.</p>
  </EditorCommandEmpty>
</EditorCommand>
```

**Filtering**: Built-in fuzzy search powered by `cmdk`.

---

### useEditor Hook

**Purpose**: Access editor instance in child components.

```tsx
import { useEditor } from "novel";

const MyComponent = () => {
  const { editor } = useEditor();

  if (!editor) return null;

  return (
    <button
      onClick={() => {
        editor.chain().focus().toggleBold().run();
      }}
    >
      Toggle Bold
    </button>
  );
};
```

**Available Methods**:

```typescript
editor.chain()           // Chainable commands
editor.commands          // Individual commands
editor.getHTML()         // Get HTML string
editor.getJSON()         // Get JSON content
editor.getText()         // Get plain text
editor.isActive("bold")  // Check if mark/node is active
editor.can()             // Check if command can run
editor.state             // ProseMirror state
editor.view              // ProseMirror view
editor.storage           // Extension storage
```

---

## Extension Configuration

### Starter Kit Extensions

```typescript
import { StarterKit } from "novel";

StarterKit.configure({
  // Document extensions
  document: false,  // Don't override
  paragraph: {
    HTMLAttributes: {
      class: "text-base leading-relaxed",
    },
  },
  text: false,

  // Block extensions
  heading: {
    levels: [1, 2, 3, 4, 5, 6],
    HTMLAttributes: {
      class: "font-bold",
    },
  },
  blockquote: {
    HTMLAttributes: {
      class: "border-l-4 border-primary pl-4",
    },
  },
  codeBlock: {
    HTMLAttributes: {
      class: "bg-muted rounded-lg p-4 font-mono text-sm",
    },
  },

  // List extensions
  bulletList: {
    HTMLAttributes: {
      class: "list-disc list-outside ml-4",
    },
  },
  orderedList: {
    HTMLAttributes: {
      class: "list-decimal list-outside ml-4",
    },
  },
  listItem: {
    HTMLAttributes: {
      class: "leading-normal",
    },
  },

  // Mark extensions
  bold: {},
  italic: {},
  strike: {},
  code: {
    HTMLAttributes: {
      class: "bg-muted px-1 py-0.5 rounded font-mono text-sm",
    },
  },

  // Other extensions
  horizontalRule: {},
  hardBreak: {},
  dropcursor: {},
  gapcursor: {},

  // History
  history: {
    depth: 100,
    newGroupDelay: 500,
  },
});
```

---

### Individual Extensions

```typescript
import {
  TiptapLink,
  TiptapImage,
  Placeholder,
  Highlight,
  Color,
  TextStyle,
} from "novel";

export const extensions = [
  // Link
  TiptapLink.configure({
    HTMLAttributes: {
      class: "text-primary underline cursor-pointer",
      rel: "noopener noreferrer",
      target: "_blank",
    },
    openOnClick: false,  // Don't follow links in editor
    validate: (href) => /^https?:\/\//.test(href),
  }),

  // Image
  TiptapImage.configure({
    HTMLAttributes: {
      class: "rounded-lg max-w-full h-auto",
    },
    inline: false,  // Block image
    allowBase64: true,
  }),

  // Placeholder
  Placeholder.configure({
    placeholder: ({ node }) => {
      switch (node.type.name) {
        case "heading":
          return `Heading ${node.attrs.level}`;
        case "codeBlock":
          return "Enter code...";
        default:
          return "Type '/' for commands...";
      }
    },
    includeChildren: true,
  }),

  // Highlight
  Highlight.configure({
    HTMLAttributes: {
      class: "bg-yellow-200 dark:bg-yellow-800",
    },
    multicolor: true,
  }),

  // Color (text color)
  Color,
  TextStyle,
];
```

---

### Custom Extensions

```typescript
import { Node } from "@tiptap/core";
import { ReactNodeViewRenderer } from "@tiptap/react";
import { NodeViewWrapper } from "@tiptap/react";

// Define component
const CalloutComponent = ({ node, updateAttributes }) => {
  return (
    <NodeViewWrapper className="callout">
      <select
        value={node.attrs.type}
        onChange={(e) => updateAttributes({ type: e.target.value })}
      >
        <option value="info">Info</option>
        <option value="warning">Warning</option>
      </select>
      <div>{node.textContent}</div>
    </NodeViewWrapper>
  );
};

// Create extension
const Callout = Node.create({
  name: "callout",
  group: "block",
  content: "block+",

  addAttributes() {
    return {
      type: {
        default: "info",
      },
    };
  },

  addNodeView() {
    return ReactNodeViewRenderer(CalloutComponent);
  },
});

// Use in extensions array
export const extensions = [
  StarterKit,
  Callout,
  // ...
];
```

---

## Styling Approaches

### Approach 1: Tailwind CSS (Recommended)

**File**: `globals.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Prose styling for editor */
.ProseMirror {
  @apply prose prose-lg dark:prose-invert max-w-none;
  @apply focus:outline-none;
  @apply min-h-[500px];
}

/* Placeholder */
.ProseMirror .is-editor-empty:first-child::before {
  content: attr(data-placeholder);
  @apply float-left text-muted-foreground pointer-events-none h-0;
}

/* Selected node */
.ProseMirror-selectednode {
  @apply outline outline-2 outline-primary;
}

/* Images */
.ProseMirror img {
  @apply rounded-lg transition-all;
}

.ProseMirror img:hover {
  @apply cursor-pointer brightness-90;
}

.ProseMirror img.ProseMirror-selectednode {
  @apply brightness-90;
}
```

**Extension Configuration**:

```typescript
StarterKit.configure({
  heading: {
    HTMLAttributes: {
      class: "font-heading font-bold",
    },
  },
  paragraph: {
    HTMLAttributes: {
      class: "text-base leading-relaxed",
    },
  },
  code: {
    HTMLAttributes: {
      class: "rounded bg-muted px-1.5 py-0.5 font-mono text-sm",
    },
  },
});
```

---

### Approach 2: CSS Modules

**File**: `Editor.module.css`

```css
.editor {
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  padding: 1rem;
  min-height: 500px;
}

.editor:focus-within {
  border-color: var(--primary);
  box-shadow: 0 0 0 2px rgba(var(--primary-rgb), 0.1);
}

.editor :global(.ProseMirror) {
  outline: none;
}

.editor :global(.ProseMirror h1) {
  font-size: 2em;
  font-weight: bold;
  margin: 0.5em 0;
}

.editor :global(.ProseMirror p) {
  margin: 0.5em 0;
}
```

**Usage**:

```tsx
import styles from "./Editor.module.css";

<EditorContent
  className={styles.editor}
  extensions={extensions}
/>
```

---

### Approach 3: Styled Components

```tsx
import styled from "styled-components";
import { EditorContent } from "novel";

const StyledEditor = styled(EditorContent)`
  border: 1px solid ${(props) => props.theme.colors.border};
  border-radius: 8px;
  padding: 16px;

  .ProseMirror {
    outline: none;
    min-height: 500px;

    h1 {
      font-size: 2em;
      font-weight: bold;
      margin: 0.5em 0;
    }

    p {
      line-height: 1.6;
      margin: 0.5em 0;
    }

    code {
      background-color: ${(props) => props.theme.colors.muted};
      padding: 2px 6px;
      border-radius: 4px;
      font-family: monospace;
    }
  }

  &:focus-within {
    border-color: ${(props) => props.theme.colors.primary};
  }
`;

<StyledEditor extensions={extensions} />
```

---

## AI Features Setup

### OpenAI API Route

**File**: `app/api/generate/route.ts`

```typescript
import { OpenAIStream, StreamingTextResponse } from "ai";
import OpenAI from "openai";

export const runtime = "edge";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(req: Request) {
  const { prompt, option } = await req.json();

  let systemMessage = "";

  switch (option) {
    case "continue":
      systemMessage = "You are an AI writing assistant that continues existing text based on context.";
      break;
    case "improve":
      systemMessage = "You are an AI writing assistant that improves existing text.";
      break;
    case "shorter":
      systemMessage = "You are an AI writing assistant that shortens text while keeping the meaning.";
      break;
    case "longer":
      systemMessage = "You are an AI writing assistant that expands text with more details.";
      break;
    case "fix":
      systemMessage = "You are an AI writing assistant that fixes grammar and spelling errors.";
      break;
  }

  const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    stream: true,
    messages: [
      { role: "system", content: systemMessage },
      { role: "user", content: prompt },
    ],
  });

  const stream = OpenAIStream(response);
  return new StreamingTextResponse(stream);
}
```

**Environment Variables**:

```bash
# .env.local
OPENAI_API_KEY=sk-...
```

---

### AI Selector Component

```tsx
import { useCompletion } from "ai/react";
import { useEditor } from "novel";
import { toast } from "sonner";

function AISelector({ open, onOpenChange }) {
  const { editor } = useEditor();
  const [input, setInput] = useState("");

  const { completion, complete, isLoading } = useCompletion({
    api: "/api/generate",
    onResponse: (res) => {
      if (res.status === 429) {
        toast.error("Rate limit exceeded");
      }
    },
    onError: (err) => {
      toast.error(err.message);
    },
  });

  const handleContinue = () => {
    const context = editor.getText();
    complete(context, {
      body: { option: "continue" },
    });
  };

  return (
    <div className="ai-selector">
      {isLoading && <Spinner />}

      {completion && (
        <div className="prose">
          {completion}
        </div>
      )}

      <button onClick={handleContinue}>
        Continue writing
      </button>

      <button onClick={() => complete(input, { body: { option: "improve" } })}>
        Improve writing
      </button>

      {/* ... more options */}
    </div>
  );
}
```

---

## Image Upload Setup

### Vercel Blob Storage

**File**: `app/api/upload/route.ts`

```typescript
import { put } from "@vercel/blob";

export const runtime = "edge";

export async function POST(req: Request) {
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

**Environment Variables**:

```bash
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_...
```

---

### Image Upload Configuration

```tsx
import {
  EditorRoot,
  EditorContent,
  createImageUpload,
  handleImageDrop,
  handleImagePaste,
} from "novel";

const uploadFn = createImageUpload({
  validateFn: (file) => {
    if (!file.type.includes("image/")) {
      toast.error("File type not supported.");
      return false;
    }
    if (file.size / 1024 / 1024 > 20) {
      toast.error("File size too big (max 20MB).");
      return false;
    }
    return true;
  },
  onUpload: async (file) => {
    const response = await fetch("/api/upload", {
      method: "POST",
      headers: {
        "content-type": file.type,
      },
      body: file,
    });

    const { url } = await response.json();
    return url;
  },
});

<EditorContent
  extensions={extensions}
  editorProps={{
    handleDrop: (view, event, slice, moved) =>
      handleImageDrop(view, event, moved, uploadFn),
    handlePaste: (view, event) =>
      handleImagePaste(view, event, uploadFn),
  }}
/>
```

---

### Custom Upload (AWS S3, Cloudinary, etc.)

```typescript
const uploadToS3 = async (file: File): Promise<string> => {
  // 1. Get pre-signed URL
  const response = await fetch("/api/presigned-url", {
    method: "POST",
    body: JSON.stringify({
      filename: file.name,
      contentType: file.type,
    }),
  });

  const { url, fields } = await response.json();

  // 2. Upload to S3
  const formData = new FormData();
  Object.entries(fields).forEach(([key, value]) => {
    formData.append(key, value as string);
  });
  formData.append("file", file);

  await fetch(url, {
    method: "POST",
    body: formData,
  });

  // 3. Return public URL
  return `${url}/${fields.key}`;
};

const uploadFn = createImageUpload({
  onUpload: uploadToS3,
});
```

---

## Common Patterns

### Pattern 1: Saving to Database

```tsx
import { useDebouncedCallback } from "use-debounce";

function MyEditor() {
  const [saveStatus, setSaveStatus] = useState("Saved");

  const debouncedSave = useDebouncedCallback(
    async (editor) => {
      const json = editor.getJSON();

      await fetch("/api/documents/123", {
        method: "PATCH",
        body: JSON.stringify({ content: json }),
      });

      setSaveStatus("Saved");
    },
    500
  );

  return (
    <EditorContent
      extensions={extensions}
      onUpdate={({ editor }) => {
        setSaveStatus("Unsaved");
        debouncedSave(editor);
      }}
    />
  );
}
```

---

### Pattern 2: Loading from Database

```tsx
function MyEditor({ documentId }) {
  const [content, setContent] = useState<JSONContent | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/documents/${documentId}`)
      .then((res) => res.json())
      .then((data) => {
        setContent(data.content);
        setLoading(false);
      });
  }, [documentId]);

  if (loading) return <div>Loading...</div>;

  return (
    <EditorContent
      initialContent={content}
      extensions={extensions}
    />
  );
}
```

---

### Pattern 3: Controlled Editor

```tsx
function ControlledEditor() {
  const [content, setContent] = useState<JSONContent>({
    type: "doc",
    content: [],
  });

  return (
    <div>
      <button
        onClick={() => {
          // Programmatically set content
          setContent({
            type: "doc",
            content: [
              {
                type: "paragraph",
                content: [{ type: "text", text: "Hello World" }],
              },
            ],
          });
        }}
      >
        Set Content
      </button>

      <EditorContent
        initialContent={content}
        onUpdate={({ editor }) => {
          setContent(editor.getJSON());
        }}
        extensions={extensions}
      />
    </div>
  );
}
```

---

### Pattern 4: Read-Only Mode

```tsx
function ReadOnlyEditor({ content }) {
  return (
    <EditorContent
      initialContent={content}
      extensions={extensions}
      editable={false}  // ← Read-only
      editorProps={{
        attributes: {
          class: "prose prose-lg",
        },
      }}
    />
  );
}
```

---

### Pattern 5: Multiple Editors

```tsx
function MultiEditor() {
  return (
    <div className="grid grid-cols-2 gap-4">
      <EditorRoot>
        <EditorContent extensions={extensions} />
      </EditorRoot>

      <EditorRoot>
        <EditorContent extensions={extensions} />
      </EditorRoot>
    </div>
  );
}
```

**Each editor has isolated state** thanks to separate `EditorRoot` providers.

---

### Pattern 6: Character Count

```tsx
import { CharacterCount } from "novel";

const extensions = [
  StarterKit,
  CharacterCount,
  // ...
];

function MyEditor() {
  const { editor } = useEditor();

  const words = editor?.storage.characterCount.words() ?? 0;
  const characters = editor?.storage.characterCount.characters() ?? 0;

  return (
    <div>
      <div className="stats">
        {words} words, {characters} characters
      </div>

      <EditorContent extensions={extensions} />
    </div>
  );
}
```

---

### Pattern 7: Markdown Import/Export

```tsx
import { MarkdownExtension } from "novel";

const extensions = [
  StarterKit,
  MarkdownExtension,
  // ...
];

function MyEditor() {
  const { editor } = useEditor();

  const exportMarkdown = () => {
    const markdown = editor?.storage.markdown.getMarkdown();
    console.log(markdown);
  };

  const importMarkdown = (markdown: string) => {
    editor?.commands.setContent(markdown);
  };

  return (
    <div>
      <button onClick={exportMarkdown}>Export Markdown</button>

      <EditorContent extensions={extensions} />
    </div>
  );
}
```

---

## Troubleshooting

### Issue: "window is not defined" (SSR Error)

**Cause**: Novel uses browser-only APIs.

**Solution**: Disable SSR for editor component.

```tsx
// Next.js
const Editor = dynamic(() => import("@/components/Editor"), {
  ssr: false,
});

// Remix
import { ClientOnly } from "remix-utils/client-only";
<ClientOnly>{() => <Editor />}</ClientOnly>
```

---

### Issue: Styles Not Loading

**Cause**: Novel styles not imported.

**Solution**: Import styles in root layout/app.

```tsx
import "novel/styles.css";
```

Or copy styles to your own CSS file for customization.

---

### Issue: Editor Not Updating

**Cause**: `initialContent` prop changed after mount.

**Solution**: Use `key` prop to force remount.

```tsx
<EditorContent
  key={documentId}  // Force remount on ID change
  initialContent={content}
  extensions={extensions}
/>
```

---

### Issue: Extensions Not Working

**Cause**: Extensions array recreated on every render.

**Solution**: Memoize extensions or define outside component.

```tsx
// ❌ BAD: New array every render
function MyEditor() {
  const extensions = [StarterKit, Placeholder];  // Don't do this
  return <EditorContent extensions={extensions} />;
}

// ✅ GOOD: Stable reference
const extensions = [StarterKit, Placeholder];

function MyEditor() {
  return <EditorContent extensions={extensions} />;
}

// ✅ ALSO GOOD: Memoized
function MyEditor() {
  const extensions = useMemo(() => [StarterKit, Placeholder], []);
  return <EditorContent extensions={extensions} />;
}
```

---

### Issue: Image Upload Fails

**Cause**: CORS or upload function error.

**Solution**: Check upload function and API route.

```tsx
const uploadFn = createImageUpload({
  onUpload: async (file) => {
    try {
      const response = await fetch("/api/upload", {
        method: "POST",
        headers: { "content-type": file.type },
        body: file,
      });

      if (!response.ok) {
        throw new Error(`Upload failed: ${response.statusText}`);
      }

      const { url } = await response.json();
      return url;
    } catch (error) {
      console.error("Upload error:", error);
      toast.error("Upload failed");
      throw error;
    }
  },
});
```

---

### Issue: Type Errors

**Cause**: Missing TypeScript types.

**Solution**: Ensure `@types/react` is installed.

```bash
npm install -D @types/react @types/react-dom
```

---

### Issue: Slash Commands Not Showing

**Cause**: Missing `EditorCommand` component or slash extension.

**Solution**: Add both to your setup.

```tsx
import { Command } from "novel/extensions";

const extensions = [
  StarterKit,
  Command,  // ← Slash command extension
  // ...
];

<EditorContent extensions={extensions}>
  <EditorCommand>{/* ... */}</EditorCommand>
</EditorContent>
```

---

### Issue: Bubble Menu Not Appearing

**Cause**: Missing `EditorBubble` component.

**Solution**: Add inside `EditorContent`.

```tsx
<EditorContent extensions={extensions}>
  <EditorBubble>
    {/* Toolbar items */}
  </EditorBubble>
</EditorContent>
```

---

## Summary & Next Steps

### You've Learned

✅ How to install and set up Novel
✅ Framework-specific integration (Next.js, Vite, Remix)
✅ Component API and props
✅ Extension configuration
✅ Styling approaches
✅ AI features setup
✅ Image upload patterns
✅ Common integration patterns
✅ Troubleshooting tips

---

### Next Steps

1. **Explore**: Try different extensions and configurations
2. **Customize**: Build custom extensions for your use case
3. **Optimize**: Add debouncing, caching, and error handling
4. **Extend**: Integrate with your database and auth system
5. **Deploy**: Ship your editor to production

---

## Additional Resources

### Official Documentation

- **Novel**: https://novel.sh/docs
- **Tiptap**: https://tiptap.dev/docs
- **Vercel AI SDK**: https://sdk.vercel.ai/docs

### Novel Guides

- **Getting Started**: See GETTING_STARTED.md
- **Architecture**: See ARCHITECTURE_OVERVIEW.md
- **Editor Core**: See EDITOR_CORE_GUIDE.md
- **State Management**: See STATE_MANAGEMENT.md
- **Frontend**: See FRONTEND_ARCHITECTURE.md

### Community

- **GitHub**: https://github.com/steven-tey/novel
- **Discord**: https://discord.gg/tiptap (Tiptap community)
- **Twitter**: @steventey

---

**Next Document**: PATTERNS_AND_CONVENTIONS.md (Phase 5) - Code style, naming conventions, and architectural patterns used in Novel.
