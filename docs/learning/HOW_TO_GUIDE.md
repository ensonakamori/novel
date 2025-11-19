# How-To Guide: Practical Recipes for Common Tasks

> **Target Audience**: Developers integrating Novel into their projects
>
> **Prerequisites**: Basic understanding of Novel architecture (see ARCHITECTURE_OVERVIEW.md)
>
> **What You'll Learn**: Step-by-step solutions for common implementation tasks

---

## Table of Contents

### Getting Started
1. [How to Install Novel in Your Project](#how-to-install-novel-in-your-project)
2. [How to Create Your First Editor](#how-to-create-your-first-editor)
3. [How to Add Basic Toolbar](#how-to-add-basic-toolbar)

### Customization
4. [How to Add a Custom Extension](#how-to-add-a-custom-extension)
5. [How to Add Custom Slash Commands](#how-to-add-custom-slash-commands)
6. [How to Customize Editor Styling](#how-to-customize-editor-styling)
7. [How to Add Custom Keyboard Shortcuts](#how-to-add-custom-keyboard-shortcuts)

### Content Management
8. [How to Save and Load Content](#how-to-save-and-load-content)
9. [How to Export to Different Formats](#how-to-export-to-different-formats)
10. [How to Implement Auto-Save](#how-to-implement-auto-save)
11. [How to Handle Images](#how-to-handle-images)

### AI Features
12. [How to Integrate AI Text Generation](#how-to-integrate-ai-text-generation)
13. [How to Add Custom AI Commands](#how-to-add-custom-ai-commands)
14. [How to Implement Rate Limiting](#how-to-implement-rate-limiting)

### Advanced
15. [How to Build a Collaborative Editor](#how-to-build-a-collaborative-editor)
16. [How to Add Comments and Annotations](#how-to-add-comments-and-annotations)
17. [How to Implement Version History](#how-to-implement-version-history)
18. [How to Optimize Performance](#how-to-optimize-performance)

---

## How to Install Novel in Your Project

### Step 1: Install the Package

```bash
# Using pnpm (recommended)
pnpm add novel

# Using npm
npm install novel

# Using yarn
yarn add novel
```

### Step 2: Install Peer Dependencies

Novel requires React 18+:

```bash
pnpm add react react-dom
```

### Step 3: Set Up TypeScript (Optional but Recommended)

```bash
pnpm add -D typescript @types/react @types/react-dom
```

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

### Step 4: Verify Installation

Create `src/test-editor.tsx`:

```tsx
import { EditorRoot, EditorContent } from "novel";

export default function TestEditor() {
  return (
    <EditorRoot>
      <EditorContent className="border p-4" />
    </EditorRoot>
  );
}
```

**Expected Result**: Basic editor renders with border and padding.

---

## How to Create Your First Editor

### Goal

Create a fully functional editor with:
- Basic formatting (bold, italic, underline)
- Headings and lists
- Links and images
- Placeholder text

### Step 1: Create Editor Component

```tsx
// components/MyEditor.tsx
"use client";

import { EditorRoot, EditorContent } from "novel";
import { defaultExtensions } from "./extensions";

export default function MyEditor() {
  return (
    <EditorRoot>
      <EditorContent
        extensions={defaultExtensions}
        className="prose max-w-none p-4 min-h-[500px]"
        editorProps={{
          attributes: {
            class: "focus:outline-none",
          },
        }}
      />
    </EditorRoot>
  );
}
```

### Step 2: Define Extensions

```tsx
// components/extensions.ts
import {
  StarterKit,
  Placeholder,
  TiptapLink,
  TiptapImage,
  TiptapUnderline,
  HorizontalRule,
} from "novel/extensions";

export const defaultExtensions = [
  StarterKit.configure({
    heading: {
      levels: [1, 2, 3],  // H1, H2, H3 only
    },
    bulletList: {
      HTMLAttributes: {
        class: "list-disc list-outside leading-3 ml-4",
      },
    },
    orderedList: {
      HTMLAttributes: {
        class: "list-decimal list-outside leading-3 ml-4",
      },
    },
    code: {
      HTMLAttributes: {
        class: "rounded-md bg-muted px-1.5 py-1 font-mono font-medium",
      },
    },
  }),
  Placeholder.configure({
    placeholder: "Press '/' for commands...",
  }),
  TiptapLink.configure({
    HTMLAttributes: {
      class: "text-primary underline underline-offset-4 cursor-pointer",
    },
  }),
  TiptapImage.configure({
    HTMLAttributes: {
      class: "rounded-lg border border-muted",
    },
  }),
  TiptapUnderline,
  HorizontalRule,
];
```

### Step 3: Add Styling

```tsx
// app/globals.css (or your main CSS file)
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Prose styling for editor content */
.prose {
  @apply text-foreground;
}

.prose h1 {
  @apply text-4xl font-bold mb-4;
}

.prose h2 {
  @apply text-3xl font-bold mb-3;
}

.prose h3 {
  @apply text-2xl font-bold mb-2;
}

.prose p {
  @apply mb-4;
}

.prose a {
  @apply text-blue-600 underline;
}

.prose code {
  @apply bg-gray-100 px-1 rounded;
}
```

### Step 4: Use in Your App

```tsx
// app/page.tsx (Next.js App Router)
import MyEditor from "@/components/MyEditor";

export default function Home() {
  return (
    <main className="container mx-auto py-8">
      <h1 className="text-3xl font-bold mb-6">My Editor</h1>
      <MyEditor />
    </main>
  );
}
```

**Expected Result**: Full-featured editor with formatting toolbar, placeholder text, and styled output.

---

## How to Add Basic Toolbar

### Goal

Create a toolbar with:
- Bold, italic, underline buttons
- Heading dropdown
- Link button
- Undo/redo

### Step 1: Create Toolbar Component

```tsx
// components/EditorToolbar.tsx
import { useEditor } from "novel";
import { Button } from "@/components/ui/button";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  Bold,
  Italic,
  Underline,
  Link,
  Undo,
  Redo,
} from "lucide-react";

export function EditorToolbar() {
  const { editor } = useEditor();

  if (!editor) return null;

  return (
    <div className="border-b p-2 flex items-center gap-2">
      {/* Text formatting */}
      <Button
        size="sm"
        variant={editor.isActive("bold") ? "default" : "ghost"}
        onClick={() => editor.chain().focus().toggleBold().run()}
      >
        <Bold className="h-4 w-4" />
      </Button>

      <Button
        size="sm"
        variant={editor.isActive("italic") ? "default" : "ghost"}
        onClick={() => editor.chain().focus().toggleItalic().run()}
      >
        <Italic className="h-4 w-4" />
      </Button>

      <Button
        size="sm"
        variant={editor.isActive("underline") ? "default" : "ghost"}
        onClick={() => editor.chain().focus().toggleUnderline().run()}
      >
        <Underline className="h-4 w-4" />
      </Button>

      {/* Heading selector */}
      <Select
        value={
          editor.isActive("heading", { level: 1 })
            ? "h1"
            : editor.isActive("heading", { level: 2 })
            ? "h2"
            : editor.isActive("heading", { level: 3 })
            ? "h3"
            : "p"
        }
        onValueChange={(value) => {
          if (value === "p") {
            editor.chain().focus().setParagraph().run();
          } else {
            const level = parseInt(value.replace("h", "")) as 1 | 2 | 3;
            editor.chain().focus().toggleHeading({ level }).run();
          }
        }}
      >
        <SelectTrigger className="w-32">
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="p">Paragraph</SelectItem>
          <SelectItem value="h1">Heading 1</SelectItem>
          <SelectItem value="h2">Heading 2</SelectItem>
          <SelectItem value="h3">Heading 3</SelectItem>
        </SelectContent>
      </Select>

      {/* Link button */}
      <Button
        size="sm"
        variant={editor.isActive("link") ? "default" : "ghost"}
        onClick={() => {
          const url = window.prompt("Enter URL:");
          if (url) {
            editor.chain().focus().setLink({ href: url }).run();
          }
        }}
      >
        <Link className="h-4 w-4" />
      </Button>

      <div className="flex-1" />

      {/* Undo/Redo */}
      <Button
        size="sm"
        variant="ghost"
        onClick={() => editor.chain().focus().undo().run()}
        disabled={!editor.can().undo()}
      >
        <Undo className="h-4 w-4" />
      </Button>

      <Button
        size="sm"
        variant="ghost"
        onClick={() => editor.chain().focus().redo().run()}
        disabled={!editor.can().redo()}
      >
        <Redo className="h-4 w-4" />
      </Button>
    </div>
  );
}
```

### Step 2: Integrate Toolbar with Editor

```tsx
// components/MyEditor.tsx
import { EditorRoot, EditorContent } from "novel";
import { EditorToolbar } from "./EditorToolbar";
import { defaultExtensions } from "./extensions";

export default function MyEditor() {
  return (
    <EditorRoot>
      <div className="border rounded-lg">
        <EditorToolbar />
        <EditorContent
          extensions={defaultExtensions}
          className="prose max-w-none p-4 min-h-[500px]"
        />
      </div>
    </EditorRoot>
  );
}
```

**Expected Result**: Toolbar appears above editor with functional formatting buttons.

---

## How to Add a Custom Extension

### Goal

Create a "Callout" extension (like Notion) with different types (info, warning, success, error).

### Step 1: Create Extension File

```tsx
// extensions/callout.ts
import { Node, mergeAttributes } from "@tiptap/core";
import { ReactNodeViewRenderer } from "@tiptap/react";
import { CalloutComponent } from "./CalloutComponent";

export interface CalloutOptions {
  types: ("info" | "warning" | "success" | "error")[];
  HTMLAttributes: Record<string, any>;
}

declare module "@tiptap/core" {
  interface Commands<ReturnType> {
    callout: {
      setCallout: (options: { type: "info" | "warning" | "success" | "error" }) => ReturnType;
      toggleCallout: (options: { type: "info" | "warning" | "success" | "error" }) => ReturnType;
    };
  }
}

export const Callout = Node.create<CalloutOptions>({
  name: "callout",

  group: "block",

  content: "block+",

  defining: true,

  addOptions() {
    return {
      types: ["info", "warning", "success", "error"],
      HTMLAttributes: {},
    };
  },

  addAttributes() {
    return {
      type: {
        default: "info",
        parseHTML: (element) => element.getAttribute("data-type"),
        renderHTML: (attributes) => {
          return {
            "data-type": attributes.type,
          };
        },
      },
    };
  },

  parseHTML() {
    return [
      {
        tag: 'div[data-type="callout"]',
      },
    ];
  },

  renderHTML({ node, HTMLAttributes }) {
    return [
      "div",
      mergeAttributes(this.options.HTMLAttributes, HTMLAttributes, {
        "data-type": "callout",
        class: `callout callout-${node.attrs.type}`,
      }),
      0,
    ];
  },

  addCommands() {
    return {
      setCallout:
        (options) =>
        ({ commands }) => {
          return commands.setNode(this.name, options);
        },
      toggleCallout:
        (options) =>
        ({ commands }) => {
          return commands.toggleWrap(this.name, options);
        },
    };
  },

  addKeyboardShortcuts() {
    return {
      "Mod-Shift-c": () => this.editor.commands.toggleCallout({ type: "info" }),
    };
  },
});
```

### Step 2: Create React Component (Optional for Custom Rendering)

```tsx
// extensions/CalloutComponent.tsx
import { NodeViewWrapper, NodeViewContent } from "@tiptap/react";
import { AlertCircle, AlertTriangle, CheckCircle, Info } from "lucide-react";

const icons = {
  info: Info,
  warning: AlertTriangle,
  success: CheckCircle,
  error: AlertCircle,
};

const colors = {
  info: "bg-blue-50 border-blue-200 text-blue-900",
  warning: "bg-yellow-50 border-yellow-200 text-yellow-900",
  success: "bg-green-50 border-green-200 text-green-900",
  error: "bg-red-50 border-red-200 text-red-900",
};

export const CalloutComponent = ({ node }: any) => {
  const Icon = icons[node.attrs.type as keyof typeof icons];
  const colorClass = colors[node.attrs.type as keyof typeof colors];

  return (
    <NodeViewWrapper className={`callout border-l-4 p-4 my-4 rounded ${colorClass}`}>
      <div className="flex gap-3">
        <Icon className="h-5 w-5 flex-shrink-0 mt-0.5" />
        <NodeViewContent className="flex-1" />
      </div>
    </NodeViewWrapper>
  );
};
```

### Step 3: Add to Extensions List

```tsx
// components/extensions.ts
import { Callout } from "@/extensions/callout";

export const defaultExtensions = [
  // ... other extensions
  Callout.configure({
    types: ["info", "warning", "success", "error"],
  }),
];
```

### Step 4: Add Toolbar Button (Optional)

```tsx
// components/EditorToolbar.tsx
import { AlertCircle } from "lucide-react";

// Inside toolbar
<Button
  size="sm"
  variant={editor.isActive("callout") ? "default" : "ghost"}
  onClick={() => editor.chain().focus().toggleCallout({ type: "info" }).run()}
>
  <AlertCircle className="h-4 w-4" />
</Button>
```

**Expected Result**: Callout blocks with colored backgrounds and icons, insertable via toolbar or `Cmd+Shift+C`.

---

## How to Add Custom Slash Commands

### Goal

Add custom slash commands like `/meeting-notes`, `/todo-list`, `/code-snippet`.

### Step 1: Define Custom Suggestion Items

```tsx
// lib/slash-commands.tsx
import {
  createSuggestionItems,
  type SuggestionItem,
} from "novel/extensions";
import {
  Code,
  CheckSquare,
  Calendar,
  Heading1,
  Heading2,
  List,
} from "lucide-react";

export const customSlashCommands = createSuggestionItems([
  // Meeting Notes Template
  {
    title: "Meeting Notes",
    description: "Insert meeting notes template",
    searchTerms: ["meeting", "notes", "agenda"],
    icon: <Calendar className="w-4 h-4" />,
    command: ({ editor, range }) => {
      editor
        .chain()
        .focus()
        .deleteRange(range)
        .insertContent([
          { type: "heading", attrs: { level: 1 }, content: [{ type: "text", text: "Meeting Notes" }] },
          { type: "heading", attrs: { level: 2 }, content: [{ type: "text", text: "Date: " }] },
          { type: "heading", attrs: { level: 2 }, content: [{ type: "text", text: "Attendees:" }] },
          { type: "bulletList", content: [{ type: "listItem", content: [{ type: "paragraph" }] }] },
          { type: "heading", attrs: { level: 2 }, content: [{ type: "text", text: "Agenda:" }] },
          { type: "bulletList", content: [{ type: "listItem", content: [{ type: "paragraph" }] }] },
          { type: "heading", attrs: { level: 2 }, content: [{ type: "text", text: "Action Items:" }] },
          { type: "bulletList", content: [{ type: "listItem", content: [{ type: "paragraph" }] }] },
        ])
        .run();
    },
  },

  // Todo List
  {
    title: "Todo List",
    description: "Insert a todo list",
    searchTerms: ["todo", "task", "checkbox", "check"],
    icon: <CheckSquare className="w-4 h-4" />,
    command: ({ editor, range }) => {
      editor
        .chain()
        .focus()
        .deleteRange(range)
        .toggleTaskList()
        .run();
    },
  },

  // Code Snippet Template
  {
    title: "Code Snippet",
    description: "Insert code block with language",
    searchTerms: ["code", "snippet", "programming"],
    icon: <Code className="w-4 h-4" />,
    command: ({ editor, range }) => {
      const language = window.prompt("Programming language (e.g., typescript, python):");
      editor
        .chain()
        .focus()
        .deleteRange(range)
        .insertContent({
          type: "codeBlock",
          attrs: { language: language || "typescript" },
        })
        .run();
    },
  },

  // Quick Headers
  {
    title: "Heading 1",
    description: "Large section heading",
    searchTerms: ["h1", "heading", "title"],
    icon: <Heading1 className="w-4 h-4" />,
    command: ({ editor, range }) => {
      editor.chain().focus().deleteRange(range).setNode("heading", { level: 1 }).run();
    },
  },

  {
    title: "Heading 2",
    description: "Medium section heading",
    searchTerms: ["h2", "heading", "subtitle"],
    icon: <Heading2 className="w-4 h-4" />,
    command: ({ editor, range }) => {
      editor.chain().focus().deleteRange(range).setNode("heading", { level: 2 }).run();
    },
  },

  // Bullet List
  {
    title: "Bullet List",
    description: "Create a bullet list",
    searchTerms: ["ul", "list", "bullet"],
    icon: <List className="w-4 h-4" />,
    command: ({ editor, range }) => {
      editor.chain().focus().deleteRange(range).toggleBulletList().run();
    },
  },
]);
```

### Step 2: Configure Command Extension

```tsx
// components/extensions.ts
import { Command, renderItems } from "novel/extensions";
import { customSlashCommands } from "@/lib/slash-commands";

export const defaultExtensions = [
  // ... other extensions
  Command.configure({
    suggestion: {
      items: () => customSlashCommands,
      render: renderItems,
    },
  }),
];
```

### Step 3: Test Slash Commands

```tsx
// In your editor, type:
// /meeting → Shows "Meeting Notes" suggestion
// /todo → Shows "Todo List" suggestion
// /code → Shows "Code Snippet" suggestion
```

**Expected Result**: Typing `/` shows custom commands that insert templates when selected.

---

## How to Customize Editor Styling

### Goal

Customize editor appearance with:
- Custom fonts
- Custom colors
- Custom spacing
- Dark mode support

### Step 1: Define CSS Variables

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Editor colors */
    --editor-bg: #ffffff;
    --editor-text: #1a1a1a;
    --editor-border: #e5e7eb;
    --editor-focus: #3b82f6;

    /* Selection */
    --editor-selection-bg: #dbeafe;

    /* Placeholder */
    --editor-placeholder: #9ca3af;

    /* Code blocks */
    --editor-code-bg: #f3f4f6;
    --editor-code-text: #1f2937;

    /* Links */
    --editor-link: #3b82f6;
    --editor-link-hover: #2563eb;
  }

  .dark {
    /* Dark mode overrides */
    --editor-bg: #1a1a1a;
    --editor-text: #ffffff;
    --editor-border: #374151;
    --editor-focus: #60a5fa;
    --editor-selection-bg: #1e3a5f;
    --editor-placeholder: #6b7280;
    --editor-code-bg: #2d2d2d;
    --editor-code-text: #e5e7eb;
    --editor-link: #60a5fa;
    --editor-link-hover: #93c5fd;
  }
}
```

### Step 2: Apply Custom Styles

```css
/* app/globals.css */
.novel-editor {
  background-color: var(--editor-bg);
  color: var(--editor-text);
  border-color: var(--editor-border);
  font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  font-size: 16px;
  line-height: 1.7;
}

.novel-editor:focus-within {
  border-color: var(--editor-focus);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

/* Selection */
.novel-editor ::selection {
  background-color: var(--editor-selection-bg);
}

/* Placeholder */
.novel-editor .is-editor-empty:first-child::before {
  color: var(--editor-placeholder);
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}

/* Headings */
.novel-editor h1 {
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1.2;
  margin-top: 2rem;
  margin-bottom: 1rem;
}

.novel-editor h2 {
  font-size: 2rem;
  font-weight: 600;
  line-height: 1.3;
  margin-top: 1.5rem;
  margin-bottom: 0.75rem;
}

.novel-editor h3 {
  font-size: 1.5rem;
  font-weight: 600;
  line-height: 1.4;
  margin-top: 1.25rem;
  margin-bottom: 0.5rem;
}

/* Paragraphs */
.novel-editor p {
  margin-top: 0.75rem;
  margin-bottom: 0.75rem;
}

/* Links */
.novel-editor a {
  color: var(--editor-link);
  text-decoration: underline;
  text-underline-offset: 3px;
  cursor: pointer;
}

.novel-editor a:hover {
  color: var(--editor-link-hover);
}

/* Code */
.novel-editor code {
  background-color: var(--editor-code-bg);
  color: var(--editor-code-text);
  padding: 0.25rem 0.5rem;
  border-radius: 0.375rem;
  font-family: "Fira Code", "Consolas", monospace;
  font-size: 0.9em;
}

.novel-editor pre {
  background-color: var(--editor-code-bg);
  color: var(--editor-code-text);
  padding: 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  margin: 1rem 0;
}

.novel-editor pre code {
  background: none;
  padding: 0;
}

/* Lists */
.novel-editor ul,
.novel-editor ol {
  padding-left: 2rem;
  margin: 0.75rem 0;
}

.novel-editor li {
  margin: 0.25rem 0;
}

/* Blockquotes */
.novel-editor blockquote {
  border-left: 4px solid var(--editor-border);
  padding-left: 1rem;
  margin: 1rem 0;
  font-style: italic;
  color: var(--editor-placeholder);
}

/* Images */
.novel-editor img {
  max-width: 100%;
  height: auto;
  border-radius: 0.5rem;
  margin: 1rem 0;
}

/* Horizontal rules */
.novel-editor hr {
  border: none;
  border-top: 2px solid var(--editor-border);
  margin: 2rem 0;
}
```

### Step 3: Apply Custom Class to Editor

```tsx
// components/MyEditor.tsx
<EditorContent
  className="novel-editor prose max-w-none p-6 min-h-[600px] border rounded-lg"
  // ...
/>
```

### Step 4: Add Font Import (Optional)

```tsx
// app/layout.tsx or _app.tsx
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

**Expected Result**: Editor with custom styling, fonts, and dark mode support.

---

## How to Add Custom Keyboard Shortcuts

### Goal

Add custom keyboard shortcuts:
- `Cmd+Shift+X` - Clear formatting
- `Cmd+K` - Insert link
- `Cmd+Shift+D` - Insert current date
- `Cmd+/` - Toggle comment

### Step 1: Create Custom Keymap Extension

```tsx
// extensions/custom-keymap.ts
import { Extension } from "@tiptap/core";

export const CustomKeymap = Extension.create({
  name: "customKeymap",

  addKeyboardShortcuts() {
    return {
      // Clear formatting
      "Mod-Shift-x": () => {
        return this.editor.chain().focus().clearNodes().unsetAllMarks().run();
      },

      // Insert link (opens prompt)
      "Mod-k": () => {
        const previousUrl = this.editor.getAttributes("link").href;
        const url = window.prompt("URL", previousUrl);

        if (url === null) {
          return false;
        }

        if (url === "") {
          return this.editor.chain().focus().extendMarkRange("link").unsetLink().run();
        }

        return this.editor
          .chain()
          .focus()
          .extendMarkRange("link")
          .setLink({ href: url })
          .run();
      },

      // Insert current date
      "Mod-Shift-d": () => {
        const date = new Date().toLocaleDateString("en-US", {
          year: "numeric",
          month: "long",
          day: "numeric",
        });
        return this.editor.chain().focus().insertContent(date).run();
      },

      // Toggle comment (custom extension would be needed)
      "Mod-/": () => {
        // This would require a comment extension
        return this.editor.chain().focus().toggleComment().run();
      },

      // Save document (custom handler)
      "Mod-s": () => {
        // Prevent default browser save
        // Trigger your custom save logic
        const content = this.editor.getJSON();
        window.dispatchEvent(
          new CustomEvent("editor-save", { detail: { content } })
        );
        return true;
      },
    };
  },
});
```

### Step 2: Add to Extensions

```tsx
// components/extensions.ts
import { CustomKeymap } from "@/extensions/custom-keymap";

export const defaultExtensions = [
  // ... other extensions
  CustomKeymap,
];
```

### Step 3: Handle Save Event (Optional)

```tsx
// components/MyEditor.tsx
import { useEffect } from "react";

export default function MyEditor() {
  useEffect(() => {
    const handleSave = (e: CustomEvent) => {
      console.log("Saving content:", e.detail.content);
      // Your save logic here
      // e.g., API call, localStorage, etc.
    };

    window.addEventListener("editor-save", handleSave as EventListener);
    return () => window.removeEventListener("editor-save", handleSave as EventListener);
  }, []);

  return (
    <EditorRoot>
      {/* ... */}
    </EditorRoot>
  );
}
```

### Step 4: Display Shortcuts in Help Menu (Optional)

```tsx
// components/KeyboardShortcutsDialog.tsx
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Keyboard } from "lucide-react";

const shortcuts = [
  { keys: "Cmd+B", action: "Bold" },
  { keys: "Cmd+I", action: "Italic" },
  { keys: "Cmd+U", action: "Underline" },
  { keys: "Cmd+K", action: "Insert Link" },
  { keys: "Cmd+Shift+X", action: "Clear Formatting" },
  { keys: "Cmd+Shift+D", action: "Insert Date" },
  { keys: "Cmd+S", action: "Save Document" },
  { keys: "Cmd+Z", action: "Undo" },
  { keys: "Cmd+Shift+Z", action: "Redo" },
];

export function KeyboardShortcutsDialog() {
  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button variant="ghost" size="sm">
          <Keyboard className="h-4 w-4 mr-2" />
          Shortcuts
        </Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Keyboard Shortcuts</DialogTitle>
        </DialogHeader>
        <div className="grid gap-2">
          {shortcuts.map((shortcut) => (
            <div key={shortcut.keys} className="flex justify-between items-center">
              <span>{shortcut.action}</span>
              <kbd className="px-2 py-1 text-sm bg-muted rounded">
                {shortcut.keys}
              </kbd>
            </div>
          ))}
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

**Expected Result**: Custom keyboard shortcuts work throughout the editor, with optional help dialog showing all shortcuts.

---

## How to Save and Load Content

### Goal

Implement content persistence with:
- Save to localStorage
- Save to backend API
- Load on mount
- Handle errors

### Step 1: Save to localStorage

```tsx
// hooks/use-persisted-editor.ts
import { useEffect } from "react";
import type { EditorInstance, JSONContent } from "novel";

export function usePersistedEditor(
  editor: EditorInstance | null,
  storageKey: string = "editor-content"
) {
  // Load from localStorage on mount
  useEffect(() => {
    if (!editor) return;

    try {
      const saved = localStorage.getItem(storageKey);
      if (saved) {
        const content = JSON.parse(saved) as JSONContent;
        editor.commands.setContent(content);
      }
    } catch (error) {
      console.error("Failed to load content:", error);
    }
  }, [editor, storageKey]);

  // Save to localStorage on update
  useEffect(() => {
    if (!editor) return;

    const handleUpdate = () => {
      try {
        const json = editor.getJSON();
        localStorage.setItem(storageKey, JSON.stringify(json));
      } catch (error) {
        console.error("Failed to save content:", error);
      }
    };

    editor.on("update", handleUpdate);
    return () => {
      editor.off("update", handleUpdate);
    };
  }, [editor, storageKey]);
}
```

### Step 2: Save to Backend API

```tsx
// hooks/use-api-persisted-editor.ts
import { useEffect, useState } from "react";
import { useDebouncedCallback } from "use-debounce";
import type { EditorInstance, JSONContent } from "novel";

interface UseAPIPersistedEditorOptions {
  documentId: string;
  saveEndpoint: string;
  loadEndpoint: string;
  debounceMs?: number;
}

export function useAPIPersistedEditor(
  editor: EditorInstance | null,
  options: UseAPIPersistedEditorOptions
) {
  const { documentId, saveEndpoint, loadEndpoint, debounceMs = 1000 } = options;
  const [saveStatus, setSaveStatus] = useState<"saved" | "saving" | "error">("saved");
  const [lastSaved, setLastSaved] = useState<Date | null>(null);

  // Load from API on mount
  useEffect(() => {
    if (!editor) return;

    const loadContent = async () => {
      try {
        const response = await fetch(`${loadEndpoint}/${documentId}`);
        if (!response.ok) throw new Error("Failed to load");

        const data = await response.json();
        editor.commands.setContent(data.content);
      } catch (error) {
        console.error("Failed to load content:", error);
        setSaveStatus("error");
      }
    };

    loadContent();
  }, [editor, documentId, loadEndpoint]);

  // Debounced save function
  const debouncedSave = useDebouncedCallback(
    async (content: JSONContent) => {
      setSaveStatus("saving");

      try {
        const response = await fetch(saveEndpoint, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            documentId,
            content,
          }),
        });

        if (!response.ok) throw new Error("Failed to save");

        setSaveStatus("saved");
        setLastSaved(new Date());
      } catch (error) {
        console.error("Failed to save content:", error);
        setSaveStatus("error");
      }
    },
    debounceMs
  );

  // Save on update
  useEffect(() => {
    if (!editor) return;

    const handleUpdate = () => {
      const content = editor.getJSON();
      debouncedSave(content);
    };

    editor.on("update", handleUpdate);
    return () => {
      editor.off("update", handleUpdate);
    };
  }, [editor, debouncedSave]);

  return { saveStatus, lastSaved };
}
```

### Step 3: Use in Component

```tsx
// components/MyEditor.tsx
"use client";

import { useState } from "react";
import { EditorRoot, EditorContent, useEditor } from "novel";
import { useAPIPersistedEditor } from "@/hooks/use-api-persisted-editor";
import { defaultExtensions } from "./extensions";

export default function MyEditor({ documentId }: { documentId: string }) {
  const { editor } = useEditor();

  const { saveStatus, lastSaved } = useAPIPersistedEditor(editor, {
    documentId,
    saveEndpoint: "/api/documents/save",
    loadEndpoint: "/api/documents",
    debounceMs: 1000,
  });

  return (
    <div>
      {/* Save status indicator */}
      <div className="mb-2 text-sm text-muted-foreground">
        {saveStatus === "saving" && "Saving..."}
        {saveStatus === "saved" && lastSaved && (
          <>Saved {lastSaved.toLocaleTimeString()}</>
        )}
        {saveStatus === "error" && <span className="text-red-500">Failed to save</span>}
      </div>

      <EditorRoot>
        <EditorContent
          extensions={defaultExtensions}
          className="prose max-w-none p-4 min-h-[500px] border rounded-lg"
        />
      </EditorRoot>
    </div>
  );
}
```

### Step 4: Create Backend API Endpoint

```tsx
// app/api/documents/save/route.ts
import { NextRequest, NextResponse } from "next/server";

// This is a simplified example. In production, add:
// - Authentication
// - Database integration
// - Error handling
// - Rate limiting

export async function POST(req: NextRequest) {
  try {
    const { documentId, content } = await req.json();

    // Validate input
    if (!documentId || !content) {
      return NextResponse.json(
        { error: "Missing documentId or content" },
        { status: 400 }
      );
    }

    // Save to database (example)
    // await db.documents.update({
    //   where: { id: documentId },
    //   data: { content, updatedAt: new Date() },
    // });

    // For demo: save to JSON file or in-memory store
    console.log("Saving document:", documentId);

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error("Save error:", error);
    return NextResponse.json(
      { error: "Failed to save document" },
      { status: 500 }
    );
  }
}
```

**Expected Result**: Content automatically saves to backend with visual save status indicator.

---

## How to Export to Different Formats

### Goal

Export editor content to:
- HTML
- Markdown
- Plain text
- JSON
- PDF (bonus)

### Step 1: Create Export Utilities

```tsx
// lib/export-utils.ts
import type { EditorInstance } from "novel";
import { marked } from "marked";

export class EditorExporter {
  constructor(private editor: EditorInstance) {}

  // Export to HTML
  toHTML(): string {
    return this.editor.getHTML();
  }

  // Export to Markdown
  toMarkdown(): string {
    // Using the markdown extension
    return this.editor.storage.markdown?.getMarkdown() || "";
  }

  // Export to JSON
  toJSON(): string {
    const content = this.editor.getJSON();
    return JSON.stringify(content, null, 2);
  }

  // Export to plain text
  toText(): string {
    return this.editor.getText();
  }

  // Export to formatted HTML (with styles)
  toStyledHTML(): string {
    const html = this.toHTML();
    return `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Exported Document</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 0 auto;
      padding: 2rem;
      color: #333;
    }
    h1 { font-size: 2.5rem; margin-top: 0; }
    h2 { font-size: 2rem; margin-top: 2rem; }
    h3 { font-size: 1.5rem; margin-top: 1.5rem; }
    code {
      background: #f5f5f5;
      padding: 0.2em 0.4em;
      border-radius: 3px;
      font-family: monospace;
    }
    pre {
      background: #f5f5f5;
      padding: 1rem;
      border-radius: 5px;
      overflow-x: auto;
    }
    a { color: #0066cc; text-decoration: none; }
    a:hover { text-decoration: underline; }
    img { max-width: 100%; height: auto; }
  </style>
</head>
<body>
  ${html}
</body>
</html>
    `.trim();
  }

  // Download as file
  downloadAsFile(content: string, filename: string, mimeType: string) {
    const blob = new Blob([content], { type: mimeType });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = filename;
    link.click();
    URL.revokeObjectURL(url);
  }

  // Export methods with download
  exportHTML(filename = "document.html") {
    this.downloadAsFile(this.toStyledHTML(), filename, "text/html");
  }

  exportMarkdown(filename = "document.md") {
    this.downloadAsFile(this.toMarkdown(), filename, "text/markdown");
  }

  exportText(filename = "document.txt") {
    this.downloadAsFile(this.toText(), filename, "text/plain");
  }

  exportJSON(filename = "document.json") {
    this.downloadAsFile(this.toJSON(), filename, "application/json");
  }
}
```

### Step 2: Create Export Menu Component

```tsx
// components/ExportMenu.tsx
import { useEditor } from "novel";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { Button } from "@/components/ui/button";
import { Download, FileText, FileCode, FileJson } from "lucide-react";
import { EditorExporter } from "@/lib/export-utils";

export function ExportMenu() {
  const { editor } = useEditor();

  if (!editor) return null;

  const exporter = new EditorExporter(editor);

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="sm">
          <Download className="h-4 w-4 mr-2" />
          Export
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => exporter.exportHTML()}>
          <FileText className="h-4 w-4 mr-2" />
          HTML
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => exporter.exportMarkdown()}>
          <FileCode className="h-4 w-4 mr-2" />
          Markdown
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => exporter.exportText()}>
          <FileText className="h-4 w-4 mr-2" />
          Plain Text
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => exporter.exportJSON()}>
          <FileJson className="h-4 w-4 mr-2" />
          JSON
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

### Step 3: Add to Toolbar

```tsx
// components/EditorToolbar.tsx
import { ExportMenu } from "./ExportMenu";

export function EditorToolbar() {
  return (
    <div className="border-b p-2 flex items-center gap-2">
      {/* ... other toolbar items */}

      <div className="flex-1" />

      <ExportMenu />
    </div>
  );
}
```

### Step 4: Bonus - Export to PDF (Using jsPDF)

```bash
pnpm add jspdf html2canvas
```

```tsx
// lib/export-utils.ts (add to EditorExporter class)
import jsPDF from "jspdf";
import html2canvas from "html2canvas";

export class EditorExporter {
  // ... previous methods

  async exportPDF(filename = "document.pdf") {
    // Create temporary container
    const container = document.createElement("div");
    container.innerHTML = this.toStyledHTML();
    container.style.position = "absolute";
    container.style.left = "-9999px";
    container.style.width = "800px";
    document.body.appendChild(container);

    try {
      // Convert to canvas
      const canvas = await html2canvas(container);
      const imgData = canvas.toDataURL("image/png");

      // Create PDF
      const pdf = new jsPDF({
        orientation: "portrait",
        unit: "mm",
        format: "a4",
      });

      const imgWidth = 210; // A4 width in mm
      const imgHeight = (canvas.height * imgWidth) / canvas.width;

      pdf.addImage(imgData, "PNG", 0, 0, imgWidth, imgHeight);
      pdf.save(filename);
    } finally {
      document.body.removeChild(container);
    }
  }
}
```

**Expected Result**: Export dropdown menu that downloads content in multiple formats.

---

## How to Implement Auto-Save

### Goal

Implement auto-save with:
- Debounced saving
- Visual feedback
- Conflict resolution
- Offline support

### Step 1: Create Auto-Save Hook

```tsx
// hooks/use-auto-save.ts
import { useEffect, useState, useRef } from "react";
import { useDebouncedCallback } from "use-debounce";
import type { EditorInstance, JSONContent } from "novel";

interface AutoSaveOptions {
  documentId: string;
  saveDelay?: number;
  onSave?: (content: JSONContent) => Promise<void>;
  onError?: (error: Error) => void;
}

type SaveStatus = "idle" | "saving" | "saved" | "error" | "offline";

export function useAutoSave(editor: EditorInstance | null, options: AutoSaveOptions) {
  const { documentId, saveDelay = 2000, onSave, onError } = options;
  const [status, setStatus] = useState<SaveStatus>("idle");
  const [lastSaved, setLastSaved] = useState<Date | null>(null);
  const [changeCount, setChangeCount] = useState(0);
  const saveQueueRef = useRef<JSONContent | null>(null);
  const isOnlineRef = useRef(navigator.onLine);

  // Monitor online/offline status
  useEffect(() => {
    const handleOnline = () => {
      isOnlineRef.current = true;
      setStatus("idle");

      // Retry pending save if any
      if (saveQueueRef.current) {
        debouncedSave(saveQueueRef.current);
      }
    };

    const handleOffline = () => {
      isOnlineRef.current = false;
      setStatus("offline");
    };

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  // Debounced save function
  const debouncedSave = useDebouncedCallback(
    async (content: JSONContent) => {
      if (!isOnlineRef.current) {
        setStatus("offline");
        saveQueueRef.current = content;
        // Save to localStorage as backup
        localStorage.setItem(`draft-${documentId}`, JSON.stringify(content));
        return;
      }

      setStatus("saving");

      try {
        if (onSave) {
          await onSave(content);
        } else {
          // Default save to API
          const response = await fetch("/api/documents/save", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ documentId, content }),
          });

          if (!response.ok) {
            throw new Error("Failed to save");
          }
        }

        setStatus("saved");
        setLastSaved(new Date());
        setChangeCount(0);
        saveQueueRef.current = null;

        // Clear localStorage backup after successful save
        localStorage.removeItem(`draft-${documentId}`);

        // Reset to idle after 2 seconds
        setTimeout(() => setStatus("idle"), 2000);
      } catch (error) {
        console.error("Auto-save error:", error);
        setStatus("error");
        saveQueueRef.current = content;

        if (onError && error instanceof Error) {
          onError(error);
        }

        // Save to localStorage as backup
        localStorage.setItem(`draft-${documentId}`, JSON.stringify(content));
      }
    },
    saveDelay
  );

  // Listen for editor updates
  useEffect(() => {
    if (!editor) return;

    const handleUpdate = () => {
      const content = editor.getJSON();
      setChangeCount((prev) => prev + 1);
      debouncedSave(content);
    };

    editor.on("update", handleUpdate);

    return () => {
      editor.off("update", handleUpdate);
    };
  }, [editor, debouncedSave]);

  // Force save function
  const forceSave = async () => {
    if (!editor) return;
    const content = editor.getJSON();
    debouncedSave.flush(); // Flush any pending debounced calls
    await debouncedSave(content);
  };

  return {
    status,
    lastSaved,
    changeCount,
    forceSave,
  };
}
```

### Step 2: Create Save Status Indicator

```tsx
// components/SaveStatusIndicator.tsx
import { CheckCircle, Cloud, CloudOff, AlertCircle, Loader2 } from "lucide-react";
import type { SaveStatus } from "@/hooks/use-auto-save";

interface SaveStatusIndicatorProps {
  status: "idle" | "saving" | "saved" | "error" | "offline";
  lastSaved: Date | null;
  changeCount: number;
}

export function SaveStatusIndicator({
  status,
  lastSaved,
  changeCount,
}: SaveStatusIndicatorProps) {
  const formatTime = (date: Date) => {
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffSecs = Math.floor(diffMs / 1000);
    const diffMins = Math.floor(diffSecs / 60);

    if (diffSecs < 60) return "just now";
    if (diffMins === 1) return "1 minute ago";
    if (diffMins < 60) return `${diffMins} minutes ago`;
    return date.toLocaleTimeString();
  };

  return (
    <div className="flex items-center gap-2 text-sm">
      {status === "idle" && lastSaved && (
        <>
          <CheckCircle className="h-4 w-4 text-green-500" />
          <span className="text-muted-foreground">
            Saved {formatTime(lastSaved)}
          </span>
        </>
      )}

      {status === "saving" && (
        <>
          <Loader2 className="h-4 w-4 animate-spin text-blue-500" />
          <span className="text-muted-foreground">Saving...</span>
        </>
      )}

      {status === "saved" && (
        <>
          <CheckCircle className="h-4 w-4 text-green-500" />
          <span className="text-green-600">Saved!</span>
        </>
      )}

      {status === "offline" && (
        <>
          <CloudOff className="h-4 w-4 text-orange-500" />
          <span className="text-orange-600">
            Offline - {changeCount} unsaved {changeCount === 1 ? "change" : "changes"}
          </span>
        </>
      )}

      {status === "error" && (
        <>
          <AlertCircle className="h-4 w-4 text-red-500" />
          <span className="text-red-600">Failed to save</span>
        </>
      )}
    </div>
  );
}
```

### Step 3: Use in Editor Component

```tsx
// components/MyEditor.tsx
"use client";

import { EditorRoot, EditorContent, useEditor } from "novel";
import { useAutoSave } from "@/hooks/use-auto-save";
import { SaveStatusIndicator } from "./SaveStatusIndicator";
import { Button } from "@/components/ui/button";
import { defaultExtensions } from "./extensions";

export default function MyEditor({ documentId }: { documentId: string }) {
  const { editor } = useEditor();

  const { status, lastSaved, changeCount, forceSave } = useAutoSave(editor, {
    documentId,
    saveDelay: 2000,
    onSave: async (content) => {
      // Custom save logic
      console.log("Saving:", content);
      await new Promise((resolve) => setTimeout(resolve, 500)); // Simulate API call
    },
    onError: (error) => {
      console.error("Save failed:", error);
      // Show toast notification
    },
  });

  return (
    <div>
      <div className="flex justify-between items-center mb-4">
        <SaveStatusIndicator
          status={status}
          lastSaved={lastSaved}
          changeCount={changeCount}
        />
        <Button
          variant="outline"
          size="sm"
          onClick={forceSave}
          disabled={status === "saving"}
        >
          Save Now
        </Button>
      </div>

      <EditorRoot>
        <EditorContent
          extensions={defaultExtensions}
          className="prose max-w-none p-4 min-h-[500px] border rounded-lg"
        />
      </EditorRoot>
    </div>
  );
}
```

**Expected Result**: Auto-save with visual feedback, offline support, and manual save button.

---

## How to Handle Images

### Goal

Implement complete image handling:
- Upload via drag & drop
- Upload via paste
- Upload to cloud storage (Vercel Blob)
- Image resizing UI
- Validation

### Step 1: Set Up Vercel Blob

```bash
pnpm add @vercel/blob
```

```env
# .env.local
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_...
```

### Step 2: Create Image Upload API

```tsx
// app/api/upload/route.ts
import { put } from "@vercel/blob";
import { NextRequest, NextResponse } from "next/server";

export const runtime = "edge";

export async function POST(req: NextRequest) {
  if (!process.env.BLOB_READ_WRITE_TOKEN) {
    return NextResponse.json(
      { error: "Missing BLOB_READ_WRITE_TOKEN" },
      { status: 500 }
    );
  }

  try {
    const form = await req.formData();
    const file = form.get("file") as File;

    if (!file) {
      return NextResponse.json({ error: "No file provided" }, { status: 400 });
    }

    // Validate file type
    if (!file.type.startsWith("image/")) {
      return NextResponse.json(
        { error: "File must be an image" },
        { status: 400 }
      );
    }

    // Validate file size (max 5MB)
    const MAX_SIZE = 5 * 1024 * 1024;
    if (file.size > MAX_SIZE) {
      return NextResponse.json(
        { error: "File too large (max 5MB)" },
        { status: 400 }
      );
    }

    // Upload to Vercel Blob
    const blob = await put(file.name, file, {
      access: "public",
      addRandomSuffix: true,
    });

    return NextResponse.json({ url: blob.url });
  } catch (error) {
    console.error("Upload error:", error);
    return NextResponse.json(
      { error: "Failed to upload file" },
      { status: 500 }
    );
  }
}
```

### Step 3: Configure Image Extension with Upload

```tsx
// components/extensions.ts
import {
  UpdatedImage,
  createImageUpload,
  handleImageDrop,
  handleImagePaste,
} from "novel/plugins";

const uploadImageToBlob = async (file: File) => {
  const formData = new FormData();
  formData.append("file", file);

  const response = await fetch("/api/upload", {
    method: "POST",
    body: formData,
  });

  if (!response.ok) {
    throw new Error("Failed to upload image");
  }

  const { url } = await response.json();
  return url;
};

export const defaultExtensions = [
  // ... other extensions

  UpdatedImage.configure({
    HTMLAttributes: {
      class: "rounded-lg border border-muted cursor-pointer",
    },
  }),
];

export const imagePlugins = [
  createImageUpload({
    validateFn: (file) => {
      // Validate file type
      if (!file.type.startsWith("image/")) {
        return false;
      }

      // Validate file size (max 5MB)
      const MAX_SIZE = 5 * 1024 * 1024;
      if (file.size > MAX_SIZE) {
        return false;
      }

      return true;
    },
    onUpload: uploadImageToBlob,
  }),
];
```

### Step 4: Add Image Upload Button

```tsx
// components/EditorToolbar.tsx
import { Image as ImageIcon } from "lucide-react";
import { useEditor } from "novel";

export function EditorToolbar() {
  const { editor } = useEditor();

  const handleImageUpload = () => {
    const input = document.createElement("input");
    input.type = "file";
    input.accept = "image/*";

    input.onchange = async (e) => {
      const file = (e.target as HTMLInputElement).files?.[0];
      if (!file) return;

      // Show loading state
      editor?.commands.insertContent({
        type: "image",
        attrs: { src: "data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" },
      });

      try {
        const formData = new FormData();
        formData.append("file", file);

        const response = await fetch("/api/upload", {
          method: "POST",
          body: formData,
        });

        if (!response.ok) throw new Error("Upload failed");

        const { url } = await response.json();

        // Replace loading image with actual image
        editor?.commands.updateAttributes("image", { src: url });
      } catch (error) {
        console.error("Upload error:", error);
        // Remove loading image
        editor?.commands.deleteSelection();
        // Show error toast
        alert("Failed to upload image");
      }
    };

    input.click();
  };

  return (
    <div className="border-b p-2 flex items-center gap-2">
      {/* ... other toolbar items */}

      <Button
        size="sm"
        variant="ghost"
        onClick={handleImageUpload}
      >
        <ImageIcon className="h-4 w-4" />
      </Button>
    </div>
  );
}
```

### Step 5: Add Image Resizer

```tsx
// components/extensions.ts
import { ImageResizer } from "novel/extensions";

export const defaultExtensions = [
  // ... other extensions

  ImageResizer,
];
```

**Expected Result**: Complete image handling with drag & drop, paste, upload button, and interactive resizing.

---

## How to Integrate AI Text Generation

### Goal

Add AI-powered features:
- Continue writing
- Improve writing
- Make shorter/longer
- Fix grammar
- Custom AI commands

### Step 1: Set Up OpenAI API

```bash
pnpm add ai @ai-sdk/openai
```

```env
# .env.local
OPENAI_API_KEY=sk-...
```

### Step 2: Create AI Generation API Endpoint

```tsx
// app/api/generate/route.ts
import { streamText } from "ai";
import { openai } from "@ai-sdk/openai";
import { NextRequest } from "next/server";
import { match } from "ts-pattern";

export const runtime = "edge";

export async function POST(req: NextRequest) {
  // Validate API key
  if (!process.env.OPENAI_API_KEY) {
    return new Response("Missing OPENAI_API_KEY", { status: 500 });
  }

  try {
    const { prompt, option, command } = await req.json();

    // Validate input
    if (!prompt || typeof prompt !== "string") {
      return new Response("Invalid prompt", { status: 400 });
    }

    if (prompt.length > 5000) {
      return new Response("Prompt too long (max 5000 chars)", { status: 400 });
    }

    // Generate system message based on option
    const messages = match(option)
      .with("continue", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that continues existing text. Give more weight to later characters. Limit response to 200 characters but construct complete sentences.",
        },
        { role: "user" as const, content: prompt },
      ])
      .with("improve", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that improves existing text. Limit response to 200 characters but construct complete sentences.",
        },
        { role: "user" as const, content: `Improve this text: ${prompt}` },
      ])
      .with("shorter", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that shortens existing text. Make it concise. Limit response to 200 characters.",
        },
        { role: "user" as const, content: `Shorten this text: ${prompt}` },
      ])
      .with("longer", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that expands existing text. Add relevant details.",
        },
        { role: "user" as const, content: `Expand this text: ${prompt}` },
      ])
      .with("fix", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that fixes grammar and spelling errors. Return only the corrected text.",
        },
        { role: "user" as const, content: prompt },
      ])
      .with("zap", () => [
        {
          role: "system" as const,
          content:
            "You are an AI writing assistant that generates text based on a prompt and command.",
        },
        {
          role: "user" as const,
          content: `For this text: ${prompt}. Follow this command: ${command}`,
        },
      ])
      .otherwise(() => [
        {
          role: "system" as const,
          content: "You are a helpful AI writing assistant.",
        },
        { role: "user" as const, content: prompt },
      ]);

    // Stream response
    const result = await streamText({
      model: openai("gpt-4o-mini"),
      messages,
      maxTokens: 4096,
      temperature: 0.7,
    });

    return result.toDataStreamResponse();
  } catch (error) {
    console.error("AI generation error:", error);
    return new Response("AI generation failed", { status: 500 });
  }
}
```

### Step 3: Create AI Selector Component

```tsx
// components/AISelector.tsx
"use client";

import { useCompletion } from "ai/react";
import { useEditor } from "novel";
import { Button } from "@/components/ui/button";
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from "@/components/ui/popover";
import { Sparkles, Check, X } from "lucide-react";
import { useState } from "react";

const AI_OPTIONS = [
  { value: "continue", label: "Continue writing" },
  { value: "improve", label: "Improve writing" },
  { value: "shorter", label: "Make shorter" },
  { value: "longer", label: "Make longer" },
  { value: "fix", label: "Fix grammar" },
] as const;

export function AISelector() {
  const { editor } = useEditor();
  const [open, setOpen] = useState(false);

  const { completion, complete, isLoading } = useCompletion({
    api: "/api/generate",
    onResponse: (response) => {
      if (!response.ok) {
        alert("AI generation failed");
      }
    },
    onFinish: (_prompt, completion) => {
      // Insert completion into editor
      editor?.commands.insertContent(completion);
    },
  });

  if (!editor) return null;

  const handleAICommand = async (option: string) => {
    const { from, to } = editor.state.selection;
    const selectedText = editor.state.doc.textBetween(from, to);

    if (!selectedText && option !== "continue") {
      alert("Please select some text first");
      return;
    }

    // For "continue", use text before cursor
    const text = selectedText || editor.getText().slice(-500); // Last 500 chars

    // Trigger AI completion
    await complete(text, {
      body: { option },
    });

    setOpen(false);
  };

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button variant="ghost" size="sm">
          <Sparkles className="h-4 w-4 mr-2" />
          AI
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-64 p-2" align="start">
        <div className="flex flex-col gap-1">
          {AI_OPTIONS.map((opt) => (
            <Button
              key={opt.value}
              variant="ghost"
              size="sm"
              className="justify-start"
              onClick={() => handleAICommand(opt.value)}
              disabled={isLoading}
            >
              {opt.label}
            </Button>
          ))}
        </div>

        {isLoading && (
          <div className="mt-2 p-2 bg-muted rounded text-sm">
            Generating...
          </div>
        )}

        {completion && (
          <div className="mt-2 p-2 bg-muted rounded text-sm">
            <div className="mb-2">{completion}</div>
            <div className="flex gap-2">
              <Button
                size="sm"
                onClick={() => {
                  editor?.commands.insertContent(completion);
                  setOpen(false);
                }}
              >
                <Check className="h-4 w-4 mr-1" />
                Accept
              </Button>
              <Button variant="ghost" size="sm" onClick={() => setOpen(false)}>
                <X className="h-4 w-4 mr-1" />
                Reject
              </Button>
            </div>
          </div>
        )}
      </PopoverContent>
    </Popover>
  );
}
```

### Step 4: Add to Toolbar

```tsx
// components/EditorToolbar.tsx
import { AISelector } from "./AISelector";

export function EditorToolbar() {
  return (
    <div className="border-b p-2 flex items-center gap-2">
      {/* ... other toolbar items */}

      <AISelector />
    </div>
  );
}
```

**Expected Result**: AI dropdown that generates text continuations and improvements.

---

## How to Add Custom AI Commands

### Goal

Add custom AI commands:
- "Translate to Spanish"
- "Make it professional"
- "Make it casual"
- "Summarize"

### Step 1: Extend AI Generation API

```tsx
// app/api/generate/route.ts (update messages generation)
const messages = match(option)
  // ... existing options ...

  .with("translate-spanish", () => [
    {
      role: "system" as const,
      content: "You are a professional translator. Translate the text to Spanish.",
    },
    { role: "user" as const, content: prompt },
  ])
  .with("professional", () => [
    {
      role: "system" as const,
      content:
        "You are an AI writing assistant that makes text more professional and formal.",
    },
    { role: "user" as const, content: `Make this text professional: ${prompt}` },
  ])
  .with("casual", () => [
    {
      role: "system" as const,
      content:
        "You are an AI writing assistant that makes text more casual and friendly.",
    },
    { role: "user" as const, content: `Make this text casual: ${prompt}` },
  ])
  .with("summarize", () => [
    {
      role: "system" as const,
      content:
        "You are an AI writing assistant that creates concise summaries. Keep it under 100 words.",
    },
    { role: "user" as const, content: `Summarize: ${prompt}` },
  ])
  .otherwise(() => [
    /* default */
  ]);
```

### Step 2: Update AI Selector

```tsx
// components/AISelector.tsx
const AI_OPTIONS = [
  // ... existing options ...
  { value: "translate-spanish", label: "Translate to Spanish" },
  { value: "professional", label: "Make professional" },
  { value: "casual", label: "Make casual" },
  { value: "summarize", label: "Summarize" },
] as const;
```

### Step 3: Add Custom Slash Command for AI

```tsx
// lib/slash-commands.tsx
import { Sparkles } from "lucide-react";

export const customSlashCommands = createSuggestionItems([
  // ... existing commands ...

  {
    title: "AI: Continue writing",
    description: "Let AI continue your writing",
    searchTerms: ["ai", "continue", "write", "autocomplete"],
    icon: <Sparkles className="w-4 h-4" />,
    command: ({ editor }) => {
      // Trigger AI continuation
      const text = editor.getText().slice(-500);
      fetch("/api/generate", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ prompt: text, option: "continue" }),
      })
        .then((res) => res.text())
        .then((completion) => {
          editor.commands.insertContent(completion);
        });
    },
  },

  {
    title: "AI: Improve selection",
    description: "Improve selected text with AI",
    searchTerms: ["ai", "improve", "enhance", "better"],
    icon: <Sparkles className="w-4 h-4" />,
    command: ({ editor, range }) => {
      const text = editor.state.doc.textBetween(range.from, range.to);
      if (!text) {
        alert("Please select some text first");
        return;
      }

      editor.commands.deleteRange(range);

      fetch("/api/generate", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ prompt: text, option: "improve" }),
      })
        .then((res) => res.text())
        .then((completion) => {
          editor.commands.insertContent(completion);
        });
    },
  },
]);
```

**Expected Result**: Extended AI commands accessible via toolbar and slash commands.

---

## How to Implement Rate Limiting

### Goal

Protect AI endpoints with:
- IP-based rate limiting
- Redis-backed limits
- Informative error messages
- Rate limit headers

### Step 1: Set Up Upstash Redis

```bash
pnpm add @upstash/ratelimit @upstash/redis
```

```env
# .env.local
KV_REST_API_URL=https://...
KV_REST_API_TOKEN=...
```

### Step 2: Create Rate Limiter

```tsx
// lib/rate-limit.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

// Create Redis client
export const redis = new Redis({
  url: process.env.KV_REST_API_URL!,
  token: process.env.KV_REST_API_TOKEN!,
});

// Create rate limiter (50 requests per day per IP)
export const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(50, "1 d"),
  analytics: true,
  prefix: "novel_ai",
});

// Alternative: Token bucket (allows bursts)
export const ratelimitBurst = new Ratelimit({
  redis,
  limiter: Ratelimit.tokenBucket(10, "10 s", 5), // 10 tokens, refill 5 every 10s
  prefix: "novel_ai_burst",
});
```

### Step 3: Apply Rate Limiting to API

```tsx
// app/api/generate/route.ts
import { ratelimit } from "@/lib/rate-limit";

export async function POST(req: NextRequest) {
  // ... existing validation ...

  // Apply rate limiting
  if (process.env.KV_REST_API_URL && process.env.KV_REST_API_TOKEN) {
    const ip = req.headers.get("x-forwarded-for") || "127.0.0.1";

    const { success, limit, reset, remaining } = await ratelimit.limit(
      `ai_${ip}`
    );

    if (!success) {
      return new Response(
        JSON.stringify({
          error: "Rate limit exceeded",
          message: "You have reached your daily AI request limit. Please try again tomorrow.",
        }),
        {
          status: 429,
          headers: {
            "Content-Type": "application/json",
            "X-RateLimit-Limit": limit.toString(),
            "X-RateLimit-Remaining": remaining.toString(),
            "X-RateLimit-Reset": reset.toString(),
            "Retry-After": Math.floor((reset - Date.now()) / 1000).toString(),
          },
        }
      );
    }

    // Add rate limit headers to successful responses
    const headers = {
      "X-RateLimit-Limit": limit.toString(),
      "X-RateLimit-Remaining": remaining.toString(),
      "X-RateLimit-Reset": reset.toString(),
    };

    // Store headers to add to response later
    req.headers.set("X-Internal-Rate-Limit", JSON.stringify(headers));
  }

  // ... existing AI generation code ...
}
```

### Step 4: Display Rate Limit to User

```tsx
// components/AISelector.tsx
const { completion, complete, isLoading, error } = useCompletion({
  api: "/api/generate",
  onResponse: (response) => {
    if (response.status === 429) {
      const retryAfter = response.headers.get("Retry-After");
      const hours = retryAfter ? Math.ceil(parseInt(retryAfter) / 3600) : 24;

      alert(
        `Rate limit exceeded. You can make more AI requests in ${hours} hours.`
      );
      return;
    }

    if (!response.ok) {
      alert("AI generation failed");
    }
  },
  // ... rest of config
});
```

### Step 5: Show Rate Limit Status (Optional)

```tsx
// components/RateLimitIndicator.tsx
"use client";

import { useEffect, useState } from "react";
import { AlertCircle } from "lucide-react";

export function RateLimitIndicator() {
  const [remaining, setRemaining] = useState<number | null>(null);
  const [limit, setLimit] = useState<number | null>(null);

  useEffect(() => {
    // Intercept fetch to read rate limit headers
    const originalFetch = window.fetch;
    window.fetch = async (...args) => {
      const response = await originalFetch(...args);

      const limitHeader = response.headers.get("X-RateLimit-Limit");
      const remainingHeader = response.headers.get("X-RateLimit-Remaining");

      if (limitHeader) setLimit(parseInt(limitHeader));
      if (remainingHeader) setRemaining(parseInt(remainingHeader));

      return response;
    };

    return () => {
      window.fetch = originalFetch;
    };
  }, []);

  if (remaining === null || limit === null) return null;

  const percentage = (remaining / limit) * 100;

  return (
    <div className="flex items-center gap-2 text-sm text-muted-foreground">
      {percentage < 20 && <AlertCircle className="h-4 w-4 text-orange-500" />}
      <span>
        AI requests: {remaining}/{limit} remaining
      </span>
    </div>
  );
}
```

**Expected Result**: AI endpoints protected with rate limiting, graceful error messages, and usage indicators.

---

## How to Build a Collaborative Editor

### Goal

Enable real-time collaboration using Yjs:
- Multiple users editing simultaneously
- Real-time cursor positions
- Conflict-free merging
- Presence awareness

### Step 1: Install Yjs Dependencies

```bash
pnpm add yjs y-websocket @tiptap/extension-collaboration @tiptap/extension-collaboration-cursor
```

### Step 2: Set Up Collaboration Extension

```tsx
// components/extensions.ts
import Collaboration from "@tiptap/extension-collaboration";
import CollaborationCursor from "@tiptap/extension-collaboration-cursor";
import * as Y from "yjs";
import { WebsocketProvider } from "y-websocket";

// Create Yjs document (shared state)
export const createCollaborativeEditor = (documentId: string, username: string) => {
  const ydoc = new Y.Doc();

  // Connect to WebSocket server
  const provider = new WebsocketProvider(
    "wss://your-collaboration-server.com",
    documentId,
    ydoc
  );

  return {
    ydoc,
    provider,
    extensions: [
      // ... other extensions

      Collaboration.configure({
        document: ydoc,
      }),

      CollaborationCursor.configure({
        provider,
        user: {
          name: username,
          color: generateRandomColor(),
        },
      }),
    ],
  };
};

function generateRandomColor() {
  const colors = ["#ff6b6b", "#4ecdc4", "#45b7d1", "#f9ca24", "#6c5ce7"];
  return colors[Math.floor(Math.random() * colors.length)];
}
```

### Step 3: Use Collaborative Editor

```tsx
// components/CollaborativeEditor.tsx
"use client";

import { useEffect, useState } from "react";
import { EditorRoot, EditorContent } from "novel";
import { createCollaborativeEditor } from "./extensions";

export default function CollaborativeEditor({
  documentId,
  username,
}: {
  documentId: string;
  username: string;
}) {
  const [collabData, setCollabData] = useState<{
    extensions: any[];
    provider: any;
  } | null>(null);

  useEffect(() => {
    const { extensions, provider } = createCollaborativeEditor(
      documentId,
      username
    );
    setCollabData({ extensions, provider });

    return () => {
      provider.destroy();
    };
  }, [documentId, username]);

  if (!collabData) {
    return <div>Connecting...</div>;
  }

  return (
    <EditorRoot>
      <EditorContent
        extensions={collabData.extensions}
        className="prose max-w-none p-4 min-h-[500px] border rounded-lg"
      />
    </EditorRoot>
  );
}
```

### Step 4: Set Up Collaboration Server (Optional)

For production, you'll need a WebSocket server. Here's a minimal example using Hocuspocus:

```bash
pnpm add @hocuspocus/server @hocuspocus/extension-database
```

```tsx
// server/collaboration-server.ts
import { Server } from "@hocuspocus/server";
import { Database } from "@hocuspocus/extension-database";

const server = Server.configure({
  port: 1234,

  extensions: [
    new Database({
      fetch: async ({ documentName }) => {
        // Load document from your database
        const doc = await db.documents.findUnique({
          where: { id: documentName },
        });
        return doc?.content || null;
      },

      store: async ({ documentName, state }) => {
        // Save document to your database
        await db.documents.update({
          where: { id: documentName },
          data: { content: state },
        });
      },
    }),
  ],

  async onAuthenticate({ token }) {
    // Verify user authentication
    if (!token) {
      throw new Error("Unauthorized");
    }

    return {
      userId: token,
    };
  },
});

server.listen();
```

**Expected Result**: Real-time collaborative editing with cursor positions and conflict resolution.

---

## How to Add Comments and Annotations

### Goal

Add commenting system:
- Inline comments on selected text
- Comment threads
- Resolve/unresolve
- Mentions

### Step 1: Create Comment Extension

```tsx
// extensions/comment.ts
import { Mark, mergeAttributes } from "@tiptap/core";

export interface CommentOptions {
  HTMLAttributes: Record<string, any>;
}

declare module "@tiptap/core" {
  interface Commands<ReturnType> {
    comment: {
      setComment: (commentId: string) => ReturnType;
      unsetComment: () => ReturnType;
    };
  }
}

export const Comment = Mark.create<CommentOptions>({
  name: "comment",

  addOptions() {
    return {
      HTMLAttributes: {},
    };
  },

  addAttributes() {
    return {
      commentId: {
        default: null,
        parseHTML: (element) => element.getAttribute("data-comment-id"),
        renderHTML: (attributes) => {
          if (!attributes.commentId) return {};
          return { "data-comment-id": attributes.commentId };
        },
      },
    };
  },

  parseHTML() {
    return [
      {
        tag: "span[data-comment-id]",
      },
    ];
  },

  renderHTML({ HTMLAttributes }) {
    return [
      "span",
      mergeAttributes(this.options.HTMLAttributes, HTMLAttributes, {
        class: "bg-yellow-100 cursor-pointer hover:bg-yellow-200",
      }),
      0,
    ];
  },

  addCommands() {
    return {
      setComment:
        (commentId) =>
        ({ commands }) => {
          return commands.setMark(this.name, { commentId });
        },

      unsetComment:
        () =>
        ({ commands }) => {
          return commands.unsetMark(this.name);
        },
    };
  },
});
```

### Step 2: Create Comment UI

```tsx
// components/CommentSidebar.tsx
"use client";

import { useState } from "react";
import { useEditor } from "novel";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { MessageSquare, Check } from "lucide-react";

interface Comment {
  id: string;
  text: string;
  author: string;
  createdAt: Date;
  resolved: boolean;
  replies: Comment[];
}

export function CommentSidebar() {
  const { editor } = useEditor();
  const [comments, setComments] = useState<Record<string, Comment>>({});
  const [newComment, setNewComment] = useState("");
  const [selectedCommentId, setSelectedCommentId] = useState<string | null>(null);

  const handleAddComment = () => {
    if (!editor || !newComment.trim()) return;

    const { from, to } = editor.state.selection;
    const text = editor.state.doc.textBetween(from, to);

    if (!text) {
      alert("Please select text to comment on");
      return;
    }

    // Generate comment ID
    const commentId = `comment-${Date.now()}`;

    // Mark selected text with comment
    editor.chain().focus().setComment(commentId).run();

    // Add comment to state
    const comment: Comment = {
      id: commentId,
      text: newComment,
      author: "Current User", // Replace with actual user
      createdAt: new Date(),
      resolved: false,
      replies: [],
    };

    setComments((prev) => ({ ...prev, [commentId]: comment }));
    setNewComment("");
  };

  const handleResolveComment = (commentId: string) => {
    setComments((prev) => ({
      ...prev,
      [commentId]: { ...prev[commentId], resolved: true },
    }));

    // Remove comment mark from editor
    editor?.chain().focus().unsetComment().run();
  };

  return (
    <div className="w-80 border-l p-4 overflow-y-auto">
      <h3 className="font-semibold mb-4 flex items-center gap-2">
        <MessageSquare className="h-5 w-5" />
        Comments
      </h3>

      {/* New comment form */}
      <div className="mb-6">
        <Textarea
          placeholder="Add a comment..."
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          className="mb-2"
        />
        <Button onClick={handleAddComment} size="sm">
          Add Comment
        </Button>
      </div>

      {/* Comment list */}
      <div className="space-y-4">
        {Object.values(comments)
          .filter((c) => !c.resolved)
          .map((comment) => (
            <div
              key={comment.id}
              className="p-3 border rounded-lg hover:bg-muted cursor-pointer"
              onClick={() => setSelectedCommentId(comment.id)}
            >
              <div className="flex justify-between items-start mb-2">
                <span className="font-medium text-sm">{comment.author}</span>
                <Button
                  variant="ghost"
                  size="sm"
                  onClick={(e) => {
                    e.stopPropagation();
                    handleResolveComment(comment.id);
                  }}
                >
                  <Check className="h-4 w-4" />
                </Button>
              </div>
              <p className="text-sm">{comment.text}</p>
              <span className="text-xs text-muted-foreground">
                {comment.createdAt.toLocaleDateString()}
              </span>
            </div>
          ))}
      </div>

      {/* Resolved comments */}
      {Object.values(comments).filter((c) => c.resolved).length > 0 && (
        <div className="mt-6">
          <h4 className="font-medium text-sm text-muted-foreground mb-2">
            Resolved
          </h4>
          {Object.values(comments)
            .filter((c) => c.resolved)
            .map((comment) => (
              <div
                key={comment.id}
                className="p-2 border rounded text-sm text-muted-foreground mb-2"
              >
                {comment.text}
              </div>
            ))}
        </div>
      )}
    </div>
  );
}
```

### Step 3: Integrate with Editor

```tsx
// components/MyEditor.tsx
import { Comment } from "@/extensions/comment";
import { CommentSidebar } from "./CommentSidebar";

export const defaultExtensions = [
  // ... other extensions
  Comment,
];

export default function MyEditor() {
  return (
    <div className="flex">
      <EditorRoot>
        <EditorContent
          extensions={defaultExtensions}
          className="flex-1 prose max-w-none p-4 min-h-[500px]"
        />
      </EditorRoot>
      <CommentSidebar />
    </div>
  );
}
```

**Expected Result**: Commenting system with sidebar, inline highlights, and resolve/unresolve functionality.

---

## How to Implement Version History

### Goal

Track document versions:
- Auto-save snapshots
- Manual checkpoints
- Compare versions
- Restore previous versions

### Step 1: Create Version Storage

```tsx
// lib/version-storage.ts
import type { JSONContent } from "novel";

export interface DocumentVersion {
  id: string;
  content: JSONContent;
  createdAt: Date;
  author: string;
  label?: string; // For manual checkpoints
  autoSaved: boolean;
}

export class VersionHistory {
  private versions: DocumentVersion[] = [];
  private documentId: string;

  constructor(documentId: string) {
    this.documentId = documentId;
    this.loadVersions();
  }

  // Save a new version
  async saveVersion(
    content: JSONContent,
    author: string,
    label?: string
  ): Promise<DocumentVersion> {
    const version: DocumentVersion = {
      id: `v${Date.now()}`,
      content,
      createdAt: new Date(),
      author,
      label,
      autoSaved: !label,
    };

    this.versions.push(version);
    await this.persistVersions();

    // Keep only last 50 auto-saved versions
    this.pruneOldVersions();

    return version;
  }

  // Get all versions
  getVersions(): DocumentVersion[] {
    return [...this.versions].reverse(); // Newest first
  }

  // Get version by ID
  getVersion(id: string): DocumentVersion | undefined {
    return this.versions.find((v) => v.id === id);
  }

  // Restore a version
  restoreVersion(id: string): JSONContent | null {
    const version = this.getVersion(id);
    return version ? version.content : null;
  }

  // Compare two versions (returns diff)
  compareVersions(id1: string, id2: string): string {
    const v1 = this.getVersion(id1);
    const v2 = this.getVersion(id2);

    if (!v1 || !v2) return "Version not found";

    // Simple text comparison
    const text1 = JSON.stringify(v1.content);
    const text2 = JSON.stringify(v2.content);

    return `Version ${id1}: ${text1.length} chars\nVersion ${id2}: ${text2.length} chars`;
  }

  private loadVersions() {
    try {
      const stored = localStorage.getItem(`versions-${this.documentId}`);
      if (stored) {
        this.versions = JSON.parse(stored);
      }
    } catch (error) {
      console.error("Failed to load versions:", error);
    }
  }

  private async persistVersions() {
    try {
      localStorage.setItem(
        `versions-${this.documentId}`,
        JSON.stringify(this.versions)
      );

      // Also save to API for persistence
      await fetch("/api/versions/save", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          documentId: this.documentId,
          versions: this.versions,
        }),
      });
    } catch (error) {
      console.error("Failed to persist versions:", error);
    }
  }

  private pruneOldVersions() {
    // Keep all manual checkpoints + last 50 auto-saves
    const manualVersions = this.versions.filter((v) => !v.autoSaved);
    const autoVersions = this.versions
      .filter((v) => v.autoSaved)
      .slice(-50);

    this.versions = [...manualVersions, ...autoVersions].sort(
      (a, b) => new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
    );
  }
}
```

### Step 2: Auto-Save Versions

```tsx
// hooks/use-version-history.ts
import { useEffect, useRef } from "react";
import { useDebouncedCallback } from "use-debounce";
import type { EditorInstance } from "novel";
import { VersionHistory } from "@/lib/version-storage";

export function useVersionHistory(
  editor: EditorInstance | null,
  documentId: string,
  username: string
) {
  const versionHistoryRef = useRef<VersionHistory | null>(null);

  useEffect(() => {
    versionHistoryRef.current = new VersionHistory(documentId);
  }, [documentId]);

  // Auto-save version every 5 minutes
  const autoSaveVersion = useDebouncedCallback(
    () => {
      if (!editor || !versionHistoryRef.current) return;

      const content = editor.getJSON();
      versionHistoryRef.current.saveVersion(content, username);
    },
    5 * 60 * 1000 // 5 minutes
  );

  useEffect(() => {
    if (!editor) return;

    const handleUpdate = () => {
      autoSaveVersion();
    };

    editor.on("update", handleUpdate);
    return () => editor.off("update", handleUpdate);
  }, [editor, autoSaveVersion]);

  // Manual checkpoint
  const createCheckpoint = (label: string) => {
    if (!editor || !versionHistoryRef.current) return;

    const content = editor.getJSON();
    return versionHistoryRef.current.saveVersion(content, username, label);
  };

  // Restore version
  const restoreVersion = (versionId: string) => {
    if (!editor || !versionHistoryRef.current) return false;

    const content = versionHistoryRef.current.restoreVersion(versionId);
    if (!content) return false;

    editor.commands.setContent(content);
    return true;
  };

  return {
    versionHistory: versionHistoryRef.current,
    createCheckpoint,
    restoreVersion,
  };
}
```

### Step 3: Version History UI

```tsx
// components/VersionHistorySidebar.tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { History, Tag } from "lucide-react";
import type { DocumentVersion } from "@/lib/version-storage";
import type { VersionHistory } from "@/lib/version-storage";

interface VersionHistorySidebarProps {
  versionHistory: VersionHistory | null;
  onRestore: (versionId: string) => void;
  onCreateCheckpoint: (label: string) => void;
}

export function VersionHistorySidebar({
  versionHistory,
  onRestore,
  onCreateCheckpoint,
}: VersionHistorySidebarProps) {
  const [checkpointLabel, setCheckpointLabel] = useState("");
  const [open, setOpen] = useState(false);

  if (!versionHistory) return null;

  const versions = versionHistory.getVersions();

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button variant="outline" size="sm">
          <History className="h-4 w-4 mr-2" />
          Version History
        </Button>
      </DialogTrigger>
      <DialogContent className="max-w-2xl max-h-[80vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle>Version History</DialogTitle>
        </DialogHeader>

        {/* Create checkpoint */}
        <div className="flex gap-2 mb-4">
          <Input
            placeholder="Checkpoint name (e.g., 'Draft 1')"
            value={checkpointLabel}
            onChange={(e) => setCheckpointLabel(e.target.value)}
          />
          <Button
            onClick={() => {
              if (checkpointLabel) {
                onCreateCheckpoint(checkpointLabel);
                setCheckpointLabel("");
              }
            }}
          >
            <Tag className="h-4 w-4 mr-2" />
            Create Checkpoint
          </Button>
        </div>

        {/* Version list */}
        <div className="space-y-2">
          {versions.map((version) => (
            <div
              key={version.id}
              className="border rounded-lg p-3 hover:bg-muted"
            >
              <div className="flex justify-between items-start mb-2">
                <div>
                  {version.label && (
                    <span className="font-medium">{version.label}</span>
                  )}
                  {!version.label && (
                    <span className="text-sm text-muted-foreground">
                      Auto-saved
                    </span>
                  )}
                  <div className="text-xs text-muted-foreground">
                    {new Date(version.createdAt).toLocaleString()} by{" "}
                    {version.author}
                  </div>
                </div>
                <Button
                  variant="outline"
                  size="sm"
                  onClick={() => {
                    if (
                      confirm(
                        "Are you sure you want to restore this version? Current changes will be lost."
                      )
                    ) {
                      onRestore(version.id);
                      setOpen(false);
                    }
                  }}
                >
                  Restore
                </Button>
              </div>
            </div>
          ))}
        </div>

        {versions.length === 0 && (
          <div className="text-center text-muted-foreground py-8">
            No versions saved yet
          </div>
        )}
      </DialogContent>
    </Dialog>
  );
}
```

### Step 4: Integrate with Editor

```tsx
// components/MyEditor.tsx
import { useVersionHistory } from "@/hooks/use-version-history";
import { VersionHistorySidebar } from "./VersionHistorySidebar";

export default function MyEditor({ documentId }: { documentId: string }) {
  const { editor } = useEditor();

  const { versionHistory, createCheckpoint, restoreVersion } = useVersionHistory(
    editor,
    documentId,
    "Current User" // Replace with actual username
  );

  return (
    <div>
      <div className="mb-4">
        <VersionHistorySidebar
          versionHistory={versionHistory}
          onRestore={restoreVersion}
          onCreateCheckpoint={createCheckpoint}
        />
      </div>

      <EditorRoot>
        <EditorContent
          extensions={defaultExtensions}
          className="prose max-w-none p-4 min-h-[500px] border rounded-lg"
        />
      </EditorRoot>
    </div>
  );
}
```

**Expected Result**: Version history with auto-saves, manual checkpoints, and restore functionality.

---

## How to Optimize Performance

### Goal

Optimize editor for large documents:
- Lazy loading content
- Virtual scrolling
- Debounced updates
- Memoized components

### Step 1: Lazy Load Large Documents

```tsx
// hooks/use-lazy-content.ts
import { useEffect, useState } from "react";
import type { EditorInstance, JSONContent } from "novel";

export function useLazyContent(
  editor: EditorInstance | null,
  documentId: string
) {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!editor) return;

    const loadContent = async () => {
      try {
        setIsLoading(true);

        // Load content in chunks for large documents
        const response = await fetch(`/api/documents/${documentId}`);

        if (!response.ok) throw new Error("Failed to load document");

        const { content } = await response.json();

        // Set content with a slight delay to avoid blocking
        requestIdleCallback(() => {
          editor.commands.setContent(content);
          setIsLoading(false);
        });
      } catch (err) {
        setError(err instanceof Error ? err.message : "Failed to load");
        setIsLoading(false);
      }
    };

    loadContent();
  }, [editor, documentId]);

  return { isLoading, error };
}
```

### Step 2: Debounce Expensive Operations

```tsx
// Already covered in Auto-Save section, but here's a summary:

import { useDebouncedCallback } from "use-debounce";

const debouncedSave = useDebouncedCallback(
  (content) => {
    // Expensive operation (API call, localStorage, etc.)
    saveContent(content);
  },
  1000 // Wait 1 second after last change
);

// Use in onUpdate
editor.on("update", () => {
  debouncedSave(editor.getJSON());
});
```

### Step 3: Memoize Heavy Computations

```tsx
// components/EditorStats.tsx
import { useMemo } from "react";
import { useEditor } from "novel";

export function EditorStats() {
  const { editor } = useEditor();

  // Memoize expensive stat calculations
  const stats = useMemo(() => {
    if (!editor) return null;

    const text = editor.getText();
    const words = text.split(/\s+/).filter(Boolean).length;
    const chars = text.length;
    const paragraphs = editor.state.doc.childCount;

    // Expensive: Count headings
    const headings = editor.state.doc.descendants((node) => {
      return node.type.name === "heading";
    });

    return { words, chars, paragraphs, headings: headings.length };
  }, [editor?.state.doc]); // Only recompute when document changes

  if (!stats) return null;

  return (
    <div className="text-sm text-muted-foreground">
      {stats.words} words · {stats.chars} characters · {stats.paragraphs} paragraphs
      · {stats.headings} headings
    </div>
  );
}
```

### Step 4: Optimize Re-renders with React.memo

```tsx
// components/ToolbarButton.tsx
import { memo } from "react";
import { Button } from "@/components/ui/button";

interface ToolbarButtonProps {
  icon: React.ReactNode;
  onClick: () => void;
  isActive?: boolean;
  disabled?: boolean;
}

// Memoize to prevent re-renders when parent re-renders
export const ToolbarButton = memo(function ToolbarButton({
  icon,
  onClick,
  isActive,
  disabled,
}: ToolbarButtonProps) {
  return (
    <Button
      variant={isActive ? "default" : "ghost"}
      size="sm"
      onClick={onClick}
      disabled={disabled}
    >
      {icon}
    </Button>
  );
});
```

### Step 5: Use React.lazy for Code Splitting

```tsx
// app/editor/page.tsx
import { lazy, Suspense } from "react";

// Lazy load heavy editor component
const AdvancedEditor = lazy(() => import("@/components/AdvancedEditor"));

export default function EditorPage() {
  return (
    <Suspense fallback={<div>Loading editor...</div>}>
      <AdvancedEditor />
    </Suspense>
  );
}
```

**Expected Result**: Optimized editor that handles large documents smoothly with minimal lag.

---

## Summary

This guide covered 18 practical how-to recipes for building production-ready editors with Novel:

### Getting Started
✅ Installation and setup
✅ First editor with toolbar
✅ Basic formatting

### Customization
✅ Custom extensions (Callout example)
✅ Custom slash commands
✅ Custom styling and theming
✅ Custom keyboard shortcuts

### Content Management
✅ Save & load content (localStorage + API)
✅ Export to multiple formats (HTML, Markdown, JSON, PDF)
✅ Auto-save with visual feedback
✅ Image handling (upload, resize, drag & drop)

### AI Features
✅ AI text generation integration
✅ Custom AI commands
✅ Rate limiting with Redis

### Advanced
✅ Collaborative editing with Yjs
✅ Comments and annotations
✅ Version history and restore
✅ Performance optimization

---

## Next Steps

1. **Combine Patterns**: Mix and match these recipes to build your ideal editor
2. **Explore Examples**: Check out Novel's demo app for more patterns
3. **Read API Docs**: Dive deeper into Tiptap and ProseMirror docs
4. **Join Community**: Ask questions in GitHub Discussions
5. **Contribute**: Share your own recipes and improvements

---

**Document Version**: 1.0
**Last Updated**: November 2025
**Related Guides**:
- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Complete integration reference
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Best practices
- [EDITOR_CORE_GUIDE.md](./EDITOR_CORE_GUIDE.md) - Deep dive into Tiptap

---

*End of HOW_TO_GUIDE.md*