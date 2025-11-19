# Code Tours: Guided Walkthroughs of Novel's Key Features

> **Target Audience**: Developers who want to understand how Novel works internally
>
> **Prerequisites**: Familiarity with React, TypeScript, and Tiptap basics
>
> **What You'll Learn**: Step-by-step code walkthroughs of major features

---

## Table of Contents

1. [Tour 1: Slash Command Flow - From `/` to Execution](#tour-1-slash-command-flow)
2. [Tour 2: AI Completion Flow - From Trigger to Stream](#tour-2-ai-completion-flow)
3. [Tour 3: Image Upload Flow - From Drop to Display](#tour-3-image-upload-flow)
4. [Tour 4: Extension Registration and Execution](#tour-4-extension-registration-and-execution)
5. [Tour 5: State Management with Jotai](#tour-5-state-management-with-jotai)
6. [Tour 6: Bubble Menu Implementation](#tour-6-bubble-menu-implementation)
7. [Tour 7: Building a Custom Extension](#tour-7-building-a-custom-extension)

---

## Tour 1: Slash Command Flow

**Feature**: Typing `/` shows a menu of commands that insert content

**Files We'll Visit**:
1. `packages/headless/src/extensions/slash-command/index.tsx` - Command extension
2. `packages/headless/src/extensions/slash-command/suggestion.ts` - Suggestion logic
3. `packages/headless/src/extensions/slash-command/groups.tsx` - Command definitions
4. `packages/headless/src/extensions/slash-command/command-list.tsx` - UI component

### Step 1: User Types "/"

**File**: `packages/headless/src/extensions/slash-command/index.tsx`

```tsx
// Line 10-20
import { Command as CommandPrimitive } from "cmdk";
import { Extension } from "@tiptap/core";
import Suggestion from "@tiptap/suggestion";

export const Command = Extension.create({
  name: "slash-command",

  addOptions() {
    return {
      suggestion: {
        char: "/",  // ← Trigger character
        // ... more config
      },
    };
  },

  addProseMirrorPlugins() {
    return [
      Suggestion({
        editor: this.editor,
        ...this.options.suggestion,  // Pass suggestion config to Tiptap
      }),
    ];
  },
});
```

**What Happens**:
1. User types `/` in the editor
2. Tiptap's `Suggestion` plugin detects the `/` character
3. Plugin calls `items()` function to get available commands
4. Plugin calls `render()` to show the command menu

---

### Step 2: Fetching Available Commands

**File**: `packages/headless/src/extensions/slash-command/groups.tsx`

```tsx
// Line 15-40
export const createSuggestionItems = (
  customItems?: SuggestionItem[]
): SuggestionItem[] => {
  const defaultItems: SuggestionItem[] = [
    {
      title: "Text",
      description: "Just start typing with plain text.",
      searchTerms: ["p", "paragraph"],
      icon: <Text size={18} />,
      command: ({ editor, range }) => {
        editor
          .chain()
          .focus()
          .deleteRange(range)  // Delete the "/"
          .setNode("paragraph")  // Insert paragraph
          .run();
      },
    },
    // ... more commands
  ];

  return [...defaultItems, ...(customItems || [])];
};
```

**Key Points**:
- Each command has:
  - `title`: Display name
  - `description`: Help text
  - `searchTerms`: What to search for (e.g., typing "/para" matches "paragraph")
  - `icon`: Visual icon
  - `command`: Function to execute when selected

---

### Step 3: Filtering Commands as User Types

**File**: `packages/headless/src/extensions/slash-command/suggestion.ts`

```tsx
// Line 25-50
export const handleCommandNavigation = (event: KeyboardEvent) => {
  if (event.key === "ArrowDown" || event.key === "ArrowUp") {
    const commandList = document.querySelector("[cmdk-list]");
    if (!commandList) return false;

    const activeItem = commandList.querySelector("[cmdk-item][data-selected]");
    // Handle arrow key navigation...
    return true;
  }
  return false;
};

export const renderItems = () => {
  let component: ReactRenderer | null = null;
  let popup: any = null;

  return {
    onStart: (props: { editor: Editor; clientRect: DOMRect }) => {
      component = new ReactRenderer(CommandList, {
        props,
        editor: props.editor,
      });

      // Show popup at cursor position
      popup = tippy("body", {
        getReferenceClientRect: props.clientRect,
        appendTo: () => document.body,
        content: component.element,
        showOnCreate: true,
        interactive: true,
        trigger: "manual",
        placement: "bottom-start",
      });
    },

    onUpdate: (props: { editor: Editor; clientRect: DOMRect }) => {
      component?.updateProps(props);

      // Update popup position as user types
      popup?.[0]?.setProps({
        getReferenceClientRect: props.clientRect,
      });
    },

    onExit: () => {
      // Clean up when menu closes
      popup?.[0]?.destroy();
      component?.destroy();
    },
  };
};
```

**What Happens**:
1. `onStart`: Creates popup menu when "/" is typed
2. `onUpdate`: Updates menu as user types (filters commands based on search)
3. `onExit`: Cleans up when menu closes (user selects command or presses Escape)

---

### Step 4: Rendering the Command Menu

**File**: `packages/headless/src/extensions/slash-command/command-list.tsx`

```tsx
// Line 30-80
export const CommandList = ({ items, command }: CommandListProps) => {
  const [selectedIndex, setSelectedIndex] = useState(0);

  const selectItem = (index: number) => {
    const item = items[index];
    if (item) {
      command(item);  // Execute selected command
    }
  };

  useEffect(() => {
    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === "ArrowDown") {
        event.preventDefault();
        setSelectedIndex((prev) => (prev + 1) % items.length);  // Move down
        return true;
      }

      if (event.key === "ArrowUp") {
        event.preventDefault();
        setSelectedIndex((prev) => (prev - 1 + items.length) % items.length);  // Move up
        return true;
      }

      if (event.key === "Enter") {
        event.preventDefault();
        selectItem(selectedIndex);  // Execute selected command
        return true;
      }

      return false;
    };

    document.addEventListener("keydown", handleKeyDown);
    return () => document.removeEventListener("keydown", handleKeyDown);
  }, [selectedIndex, items]);

  return (
    <CommandPrimitive className="slash-command">
      <CommandPrimitive.List>
        {items.map((item, index) => (
          <CommandPrimitive.Item
            key={index}
            onSelect={() => selectItem(index)}
            className={index === selectedIndex ? "selected" : ""}
          >
            {item.icon}
            <div>
              <p>{item.title}</p>
              <span>{item.description}</span>
            </div>
          </CommandPrimitive.Item>
        ))}
      </CommandPrimitive.List>
    </CommandPrimitive>
  );
};
```

**What Happens**:
1. Component renders list of filtered commands
2. User can navigate with arrow keys (managed by state)
3. Pressing Enter executes `command` function from selected item
4. `command` function (from Step 2) manipulates editor content

---

### Step 5: Executing a Command

**Back to**: `packages/headless/src/extensions/slash-command/groups.tsx`

```tsx
// Example command execution
{
  title: "Heading 1",
  command: ({ editor, range }) => {
    editor
      .chain()           // Start command chain
      .focus()          // Focus editor
      .deleteRange(range)  // Delete "/h1" text
      .setNode("heading", { level: 1 })  // Insert H1
      .run();           // Execute chain
  },
}
```

**Flow Summary**:
```
User Types "/"
  → Suggestion plugin activated
  → Fetch commands (items)
  → Render popup menu
  → User types more → filter commands
  → User selects command (Enter or click)
  → Execute command.command()
  → Editor content updated
  → Menu closes
```

---

## Tour 2: AI Completion Flow

**Feature**: AI-powered text generation (continue writing, improve, etc.)

**Files We'll Visit**:
1. `apps/web/components/tailwind/ai-selector.tsx` - Frontend AI selector
2. `apps/web/app/api/generate/route.ts` - Backend AI endpoint
3. `packages/headless/src/extensions/ai-highlight.ts` - AI highlight extension

### Step 1: User Triggers AI Command

**File**: `apps/web/components/tailwind/ai-selector.tsx`

```tsx
// Line 15-50
import { useCompletion } from "ai/react";

export const AISelectorCommands = ({ onSelect }: AISelectorCommandsProps) => {
  const { editor } = useEditor();

  const {
    completion,      // AI-generated text
    complete,        // Function to trigger AI
    isLoading,       // Loading state
  } = useCompletion({
    api: "/api/generate",  // Backend endpoint

    onResponse: (response) => {
      if (!response.ok) {
        // Handle error
        console.error("AI generation failed");
      }
    },

    onFinish: (_prompt, completion) => {
      // Called when AI finishes generating
      // Insert completion into editor
      editor?.commands.insertContent(completion);
    },
  });

  const handleAICommand = async (option: "continue" | "improve" | "shorter") => {
    const { from, to } = editor.state.selection;
    const selectedText = editor.state.doc.textBetween(from, to);

    // For "continue", use text before cursor
    const text = selectedText || editor.getText().slice(-500);

    // Trigger AI completion
    await complete(text, {
      body: { option },  // Pass option to API
    });
  };

  return (
    <CommandGroup heading="AI Commands">
      <CommandItem
        onSelect={() => handleAICommand("continue")}
        value="continue"
      >
        <Sparkles className="w-4 h-4" />
        Continue writing
      </CommandItem>
      {/* More AI commands... */}
    </CommandGroup>
  );
};
```

**What Happens**:
1. User selects AI command (e.g., "Continue writing")
2. `handleAICommand` extracts text from editor
3. Calls `complete(text, { body: { option } })` from Vercel AI SDK
4. This makes POST request to `/api/generate` with text and option

---

### Step 2: Backend AI Processing

**File**: `apps/web/app/api/generate/route.ts`

```tsx
// Line 10-100
import { streamText } from "ai";
import { openai } from "@ai-sdk/openai";
import { match } from "ts-pattern";

export const runtime = "edge";  // Use Edge Runtime for low latency

export async function POST(req: Request) {
  // STEP 1: Validate API key
  if (!process.env.OPENAI_API_KEY || process.env.OPENAI_API_KEY === "") {
    return new Response("Missing OPENAI_API_KEY", { status: 500 });
  }

  // STEP 2: Parse request body
  const { prompt, option } = await req.json();

  // STEP 3: Validate input
  if (!prompt || typeof prompt !== "string") {
    return new Response("Invalid prompt", { status: 400 });
  }

  if (prompt.length > 5000) {
    return new Response("Prompt too long", { status: 400 });
  }

  // STEP 4: Build AI messages using pattern matching
  const messages = match(option)
    .with("continue", () => [
      {
        role: "system" as const,
        content:
          "You are an AI writing assistant that continues existing text. " +
          "Give more weight to later characters. " +
          "Limit response to 200 characters but construct complete sentences.",
      },
      { role: "user" as const, content: prompt },
    ])
    .with("improve", () => [
      {
        role: "system" as const,
        content:
          "You are an AI writing assistant that improves existing text. " +
          "Limit response to 200 characters but construct complete sentences.",
      },
      { role: "user" as const, content: `Improve this text: ${prompt}` },
    ])
    .with("shorter", () => [
      {
        role: "system" as const,
        content:
          "You are an AI writing assistant that shortens existing text. " +
          "Make it concise. Limit response to 200 characters.",
      },
      { role: "user" as const, content: `Shorten this text: ${prompt}` },
    ])
    .with("longer", () => [
      {
        role: "system" as const,
        content:
          "You are an AI writing assistant that expands existing text. " +
          "Add relevant details.",
      },
      { role: "user" as const, content: `Expand this text: ${prompt}` },
    ])
    .otherwise(() => [
      {
        role: "system" as const,
        content: "You are a helpful AI writing assistant.",
      },
      { role: "user" as const, content: prompt },
    ]);

  // STEP 5: Stream AI response
  const result = await streamText({
    model: openai("gpt-4o-mini"),
    messages,
    maxTokens: 4096,
    temperature: 0.7,
  });

  // Return streaming response
  return result.toDataStreamResponse();
}
```

**Key Points**:
- Uses Edge Runtime for low latency (runs on CDN edge, close to users)
- Validates all input (security best practice)
- Uses pattern matching (`ts-pattern`) for clean option handling
- Streams response (user sees text appear in real-time)
- Returns `DataStreamResponse` compatible with Vercel AI SDK

---

### Step 3: Streaming Response to Frontend

**Flow**:
```
Backend                                Frontend
───────                               ────────
streamText()
  → Calls OpenAI API
  → Receives response stream           useCompletion hook receives stream
  → Sends chunk                        → Updates `completion` state
  → Sends chunk                        → Updates `completion` state
  → Sends chunk                        → Updates `completion` state
  → Stream complete                    → Calls onFinish() callback
                                       → Inserts completion into editor
```

**Frontend (Revisited)**: `apps/web/components/tailwind/ai-selector.tsx`

```tsx
// Line 20-30
const { completion, complete, isLoading } = useCompletion({
  api: "/api/generate",

  onFinish: (_prompt, completion) => {
    // THIS IS CALLED WHEN STREAM COMPLETES
    // `completion` contains full AI-generated text

    // Insert at cursor position
    editor?.commands.insertContent(completion);

    // Or replace selection
    editor?.chain()
      .focus()
      .deleteSelection()
      .insertContent(completion)
      .run();
  },
});
```

---

### Step 4: AI Highlighting (Visual Feedback)

**File**: `packages/headless/src/extensions/ai-highlight.ts`

```tsx
// Line 10-60
import { Mark, mergeAttributes } from "@tiptap/core";

export const AIHighlight = Mark.create({
  name: "aiHighlight",

  addAttributes() {
    return {
      color: {
        default: "#8b5cf6",  // Purple highlight
      },
    };
  },

  renderHTML({ HTMLAttributes }) {
    return [
      "span",
      mergeAttributes(HTMLAttributes, {
        style: `background-color: ${HTMLAttributes.color}22; color: ${HTMLAttributes.color};`,
        "data-ai-highlight": "true",
      }),
      0,
    ];
  },

  addCommands() {
    return {
      setAIHighlight:
        () =>
        ({ commands }) => {
          return commands.setMark("aiHighlight");
        },

      removeAIHighlight:
        () =>
        ({ commands }) => {
          return commands.unsetMark("aiHighlight");
        },
    };
  },
});

// Helper functions
export const addAIHighlight = (editor: Editor) => {
  editor.commands.setAIHighlight();
};

export const removeAIHighlight = (editor: Editor) => {
  editor.commands.unsetAIHighlight();
};
```

**Usage in AI Flow**:
```tsx
// Before AI call
editor.commands.setAIHighlight();  // Highlight selection

// After AI completes
editor.commands.insertContent(completion);
editor.commands.removeAIHighlight();  // Remove highlight
```

**Complete AI Flow Summary**:
```
1. User selects AI command → handleAICommand()
2. Extract text from editor
3. POST to /api/generate with text + option
4. Backend:
   - Validate input
   - Build messages based on option
   - Call OpenAI with streamText()
   - Stream response back
5. Frontend:
   - useCompletion receives stream
   - Updates completion state in real-time
   - onFinish → insert into editor
6. Visual feedback with AI highlight
```

---

## Tour 3: Image Upload Flow

**Feature**: Drag & drop or paste images, auto-upload to cloud storage

**Files We'll Visit**:
1. `packages/headless/src/plugins/upload-images.tsx` - Upload plugin
2. `apps/web/app/api/upload/route.ts` - Backend upload endpoint
3. `packages/headless/src/extensions/updated-image.tsx` - Image extension

### Step 1: User Drops/Pastes Image

**File**: `packages/headless/src/plugins/upload-images.tsx`

```tsx
// Line 15-40
import { Plugin, PluginKey } from "@tiptap/pm/state";

export interface UploadFn {
  (file: File): Promise<string>;  // Returns URL
}

export interface ImageUploadOptions {
  validateFn?: (file: File) => boolean;
  onUpload: UploadFn;
}

export const createImageUpload = (options: ImageUploadOptions) => {
  return new Plugin({
    key: new PluginKey("imageUpload"),

    props: {
      // Handle dropped files
      handleDrop(view, event, slice, moved) {
        if (!event.dataTransfer?.files?.length) {
          return false;  // No files dropped
        }

        const files = Array.from(event.dataTransfer.files);
        const imageFiles = files.filter(
          (file) => file.type.startsWith("image/")
        );

        if (imageFiles.length === 0) {
          return false;  // No image files
        }

        event.preventDefault();  // Prevent default browser behavior

        // Get drop position
        const coordinates = view.posAtCoords({
          left: event.clientX,
          top: event.clientY,
        });

        if (!coordinates) return false;

        // Upload each image
        imageFiles.forEach((file) => {
          uploadImage(file, view, coordinates.pos, options);
        });

        return true;  // We handled the drop
      },

      // Handle pasted files
      handlePaste(view, event, slice) {
        const files = Array.from(event.clipboardData?.files || []);
        const imageFiles = files.filter(
          (file) => file.type.startsWith("image/")
        );

        if (imageFiles.length === 0) {
          return false;  // No images pasted
        }

        event.preventDefault();

        // Upload at cursor position
        const pos = view.state.selection.from;
        imageFiles.forEach((file) => {
          uploadImage(file, view, pos, options);
        });

        return true;
      },
    },
  });
};
```

**What Happens**:
1. ProseMirror calls `handleDrop` when user drops something
2. Plugin filters for image files only
3. Plugin calls `uploadImage()` for each image file
4. Same for `handlePaste` when user pastes

---

### Step 2: Upload Process with Loading State

**File**: `packages/headless/src/plugins/upload-images.tsx` (continued)

```tsx
// Line 80-130
const uploadImage = async (
  file: File,
  view: EditorView,
  pos: number,
  options: ImageUploadOptions
) => {
  // STEP 1: Validate file
  if (options.validateFn && !options.validateFn(file)) {
    console.error("File validation failed");
    return;
  }

  // STEP 2: Create loading placeholder
  // Insert a placeholder image while uploading
  const transaction = view.state.tr.insert(pos, [
    view.state.schema.nodes.image.create({
      src: "data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==",  // 1x1 transparent GIF
      alt: "Uploading...",
      title: "Uploading...",
    }),
  ]);

  view.dispatch(transaction);

  try {
    // STEP 3: Upload file
    const url = await options.onUpload(file);

    // STEP 4: Replace placeholder with actual image
    const { state } = view;
    const $pos = state.doc.resolve(pos);

    // Find the placeholder image node
    let imageNode = null;
    let imagePos = pos;

    state.doc.nodesBetween(pos, pos + 2, (node, pos) => {
      if (node.type.name === "image") {
        imageNode = node;
        imagePos = pos;
        return false;  // Stop iteration
      }
    });

    if (imageNode) {
      // Replace placeholder with actual image
      const tr = state.tr.setNodeMarkup(imagePos, null, {
        ...imageNode.attrs,
        src: url,
        alt: file.name,
        title: file.name,
      });

      view.dispatch(tr);
    }
  } catch (error) {
    console.error("Upload failed:", error);

    // Remove placeholder on error
    const tr = view.state.tr.delete(pos, pos + 1);
    view.dispatch(tr);

    // Show error to user
    alert("Failed to upload image");
  }
};
```

**Key Steps**:
1. Validate file (size, type, etc.)
2. Insert loading placeholder (transparent GIF)
3. Upload file to backend (await options.onUpload)
4. Replace placeholder with actual image URL
5. If error, remove placeholder and show message

---

### Step 3: Backend Upload to Vercel Blob

**File**: `apps/web/app/api/upload/route.ts`

```tsx
// Line 5-60
import { put } from "@vercel/blob";
import { NextRequest, NextResponse } from "next/server";

export const runtime = "edge";

export async function POST(req: NextRequest) {
  // STEP 1: Check environment variables
  if (!process.env.BLOB_READ_WRITE_TOKEN) {
    return NextResponse.json(
      { error: "Missing BLOB_READ_WRITE_TOKEN" },
      { status: 500 }
    );
  }

  try {
    // STEP 2: Parse form data
    const form = await req.formData();
    const file = form.get("file") as File;

    if (!file) {
      return NextResponse.json(
        { error: "No file provided" },
        { status: 400 }
      );
    }

    // STEP 3: Validate file type
    if (!file.type.startsWith("image/")) {
      return NextResponse.json(
        { error: "File must be an image" },
        { status: 400 }
      );
    }

    // STEP 4: Validate file size (5MB max)
    const MAX_SIZE = 5 * 1024 * 1024;
    if (file.size > MAX_SIZE) {
      return NextResponse.json(
        { error: "File too large (max 5MB)" },
        { status: 400 }
      );
    }

    // STEP 5: Upload to Vercel Blob
    const blob = await put(file.name, file, {
      access: "public",          // Publicly accessible URL
      addRandomSuffix: true,     // Avoid name collisions
    });

    // STEP 6: Return URL
    return NextResponse.json({
      url: blob.url,  // e.g., https://xyz.public.blob.vercel-storage.com/image-abc123.png
    });
  } catch (error) {
    console.error("Upload error:", error);
    return NextResponse.json(
      { error: "Failed to upload file" },
      { status: 500 }
    );
  }
}
```

**Security Layers**:
1. ✅ Check environment variables (fail fast)
2. ✅ Validate file exists
3. ✅ Validate file type (only images)
4. ✅ Validate file size (prevent abuse)
5. ✅ Use Edge Runtime (sandboxed, secure)

---

### Step 4: Image Extension with Resizing

**File**: `packages/headless/src/extensions/updated-image.tsx`

```tsx
// Line 20-80
import { Node } from "@tiptap/core";
import { ReactNodeViewRenderer } from "@tiptap/react";
import { ImageResizer } from "./image-resizer-component";

export const UpdatedImage = Node.create({
  name: "image",

  // Image is inline content
  inline: false,
  group: "block",

  // Define attributes
  addAttributes() {
    return {
      src: {
        default: null,
        parseHTML: (element) => element.getAttribute("src"),
        renderHTML: (attributes) => ({ src: attributes.src }),
      },
      alt: {
        default: null,
        parseHTML: (element) => element.getAttribute("alt"),
        renderHTML: (attributes) => ({ alt: attributes.alt }),
      },
      title: {
        default: null,
        parseHTML: (element) => element.getAttribute("title"),
        renderHTML: (attributes) => ({ title: attributes.title }),
      },
      width: {
        default: null,
        parseHTML: (element) => element.getAttribute("width"),
        renderHTML: (attributes) => {
          if (!attributes.width) return {};
          return { width: attributes.width };
        },
      },
      height: {
        default: null,
        parseHTML: (element) => element.getAttribute("height"),
        renderHTML: (attributes) => {
          if (!attributes.height) return {};
          return { height: attributes.height };
        },
      },
    };
  },

  // Parse from HTML (when loading saved content)
  parseHTML() {
    return [
      {
        tag: "img[src]",
      },
    ];
  },

  // Render to HTML (when saving or exporting)
  renderHTML({ HTMLAttributes }) {
    return ["img", HTMLAttributes];
  },

  // Use React component for interactive resizing
  addNodeView() {
    return ReactNodeViewRenderer(ImageResizer);
  },

  // Allow images to be dragged
  draggable: true,
});
```

**ImageResizer Component** (simplified):

```tsx
// packages/headless/src/extensions/image-resizer-component.tsx
import { NodeViewWrapper } from "@tiptap/react";
import { useState } from "react";
import Moveable from "react-moveable";

export const ImageResizer = ({ node, updateAttributes }: NodeViewProps) => {
  const [target, setTarget] = useState<HTMLElement | null>(null);

  return (
    <NodeViewWrapper>
      {/* Actual image */}
      <img
        ref={setTarget}
        src={node.attrs.src}
        alt={node.attrs.alt}
        style={{
          width: node.attrs.width || "auto",
          height: node.attrs.height || "auto",
        }}
      />

      {/* Resize handles */}
      {target && (
        <Moveable
          target={target}
          resizable={true}
          onResize={({ width, height }) => {
            // Update image attributes
            updateAttributes({
              width: `${width}px`,
              height: `${height}px`,
            });
          }}
        />
      )}
    </NodeViewWrapper>
  );
};
```

**Image Upload Flow Summary**:
```
1. User drops/pastes image
   → Plugin intercepts event
2. Insert loading placeholder (transparent GIF)
3. Call onUpload(file)
   → POST to /api/upload
   → Upload to Vercel Blob
   → Return URL
4. Replace placeholder with actual image
5. User can resize with ImageResizer component
```

---

## Tour 4: Extension Registration and Execution

**Feature**: How extensions extend Tiptap's functionality

**Files We'll Visit**:
1. `packages/headless/src/components/editor.tsx` - Editor root
2. `packages/headless/src/extensions/index.ts` - Extension exports
3. `apps/web/components/tailwind/advanced-editor.tsx` - Usage example

### Step 1: Defining Extensions

**File**: `apps/web/components/tailwind/advanced-editor.tsx`

```tsx
// Line 30-80
import { StarterKit } from "novel/extensions";
import { AIHighlight } from "novel/extensions";
import { TiptapLink } from "novel/extensions";
import { TiptapImage } from "novel/extensions";

const extensions = [
  // StarterKit includes: Bold, Italic, Code, Strike, Paragraph, Heading, etc.
  StarterKit.configure({
    // Configure built-in extensions
    bulletList: {
      HTMLAttributes: {
        class: cx("list-disc list-outside leading-3 -mt-2"),
      },
    },
    orderedList: {
      HTMLAttributes: {
        class: cx("list-decimal list-outside leading-3 -mt-2"),
      },
    },
    code: {
      HTMLAttributes: {
        class: cx(
          "rounded-md bg-muted px-1.5 py-1 font-mono font-medium",
          "text-foreground"
        ),
      },
    },
  }),

  // Add custom extensions
  AIHighlight,
  TiptapLink.configure({
    HTMLAttributes: {
      class: cx(
        "text-muted-foreground underline underline-offset-[3px]",
        "hover:text-primary transition-colors cursor-pointer"
      ),
    },
  }),
  TiptapImage,
];
```

**What's Happening**:
- Extensions is just an array
- Can configure each extension
- Can add custom extensions
- Order doesn't matter (Tiptap handles priorities)

---

### Step 2: Registering Extensions with Editor

**File**: `packages/headless/src/components/editor.tsx`

```tsx
// Line 40-100
import { useEditor as useTiptapEditor, EditorProvider } from "@tiptap/react";

export const EditorRoot = ({ children }: { children: ReactNode }) => {
  return <EditorProvider>{children}</EditorProvider>;
};

export const EditorContent = ({
  extensions,
  editorProps,
  className,
  ...props
}: EditorContentProps) => {
  const editor = useTiptapEditor({
    // STEP 1: Register all extensions
    extensions: extensions,

    // STEP 2: Configure editor props
    editorProps: {
      attributes: {
        class: className || "",
      },
      ...editorProps,
    },

    // STEP 3: Initial content (optional)
    content: props.initialContent || "",

    // STEP 4: Event handlers
    onUpdate: ({ editor }) => {
      props.onUpdate?.(editor);
    },
    onCreate: ({ editor }) => {
      props.onCreate?.(editor);
    },
  });

  return (
    <EditorProvider>
      <EditorContentPrimitive editor={editor} />
    </EditorProvider>
  );
};
```

**Tiptap Internal Process** (simplified):
```typescript
// When you call useEditor({ extensions: [...] })

1. Tiptap iterates over extensions array
2. For each extension:
   a. Calls extension.configure() if you provided config
   b. Registers extension's schema (nodes/marks)
   c. Registers extension's commands
   d. Registers extension's keyboard shortcuts
   e. Registers extension's plugins
   f. Registers extension's input rules
   g. Registers extension's paste rules

3. Builds final ProseMirror schema from all extensions
4. Creates ProseMirror state with schema + plugins
5. Creates ProseMirror view connected to DOM
6. Returns editor instance
```

---

### Step 3: Extension Lifecycle

Let's trace how **Bold** extension works (from StarterKit):

**File**: `@tiptap/extension-bold/src/bold.ts` (from Tiptap library)

```tsx
// Simplified version
export const Bold = Mark.create({
  name: "bold",

  // 1. SCHEMA: Define how bold appears in document
  parseHTML() {
    return [
      { tag: "strong" },
      { tag: "b" },
      { style: "font-weight", getAttrs: (value) => /^(bold|[5-9]\d{2,})$/.test(value) },
    ];
  },

  renderHTML({ HTMLAttributes }) {
    return ["strong", HTMLAttributes, 0];
  },

  // 2. COMMANDS: How to toggle bold
  addCommands() {
    return {
      setBold:
        () =>
        ({ commands }) => {
          return commands.setMark(this.name);
        },

      toggleBold:
        () =>
        ({ commands }) => {
          return commands.toggleMark(this.name);
        },

      unsetBold:
        () =>
        ({ commands }) => {
          return commands.unsetMark(this.name);
        },
    };
  },

  // 3. KEYBOARD SHORTCUTS: Cmd+B
  addKeyboardShortcuts() {
    return {
      "Mod-b": () => this.editor.commands.toggleBold(),
      "Mod-B": () => this.editor.commands.toggleBold(),
    };
  },

  // 4. INPUT RULES: **bold**
  addInputRules() {
    return [
      markInputRule({
        find: /(?:^|\s)((?:\*\*)((?:[^*]+))(?:\*\*))$/,
        type: this.type,
      }),
    ];
  },
});
```

**When User Presses Cmd+B**:
```
1. Browser fires keydown event
2. ProseMirror's keymap plugin intercepts
3. Checks all registered keyboard shortcuts
4. Finds Bold's "Mod-b" → this.editor.commands.toggleBold()
5. toggleBold() calls ProseMirror's toggleMark transaction
6. ProseMirror updates state
7. ProseMirror re-renders view
8. User sees bold text
```

---

### Step 4: Custom Extension Registration

**File**: `packages/headless/src/extensions/ai-highlight.ts`

```tsx
// Line 10-50
export const AIHighlight = Mark.create({
  name: "aiHighlight",

  // 1. REGISTERED IN SCHEMA
  addAttributes() {
    return {
      color: { default: "#8b5cf6" },
    };
  },

  // 2. REGISTERED IN PARSER
  parseHTML() {
    return [{ tag: "span[data-ai-highlight]" }];
  },

  // 3. REGISTERED IN RENDERER
  renderHTML({ HTMLAttributes }) {
    return [
      "span",
      {
        ...HTMLAttributes,
        "data-ai-highlight": "true",
        style: `background-color: ${HTMLAttributes.color}22;`,
      },
      0,
    ];
  },

  // 4. REGISTERED AS COMMANDS
  addCommands() {
    return {
      setAIHighlight: () => ({ commands }) => commands.setMark("aiHighlight"),
      unsetAIHighlight: () => ({ commands }) => commands.unsetMark("aiHighlight"),
    };
  },
});

// USAGE:
const extensions = [StarterKit, AIHighlight];

// Later in code:
editor.commands.setAIHighlight();  // ← This command is available because extension was registered
```

**Extension Registration Flow Summary**:
```
Define Extension
  → Include in extensions array
  → Pass to useEditor({ extensions })
  → Tiptap registers:
    - Schema (nodes/marks)
    - Commands
    - Keyboard shortcuts
    - Input rules
    - Paste rules
    - Plugins
  → Extension functionality available
```

---

## Tour 5: State Management with Jotai

**Feature**: Lightweight state management for editor UI

**Files We'll Visit**:
1. `packages/headless/src/utils/atoms.ts` - Atom definitions
2. `apps/web/components/tailwind/generative/generative-menu-switch.tsx` - Usage example

### Step 1: Defining Atoms

**File**: `packages/headless/src/utils/atoms.ts`

```tsx
// Line 5-15
import { atom } from "jotai";
import type { EditorBubbleItem } from "../components/editor-bubble";

// Query atom: Stores current AI command query
export const queryAtom = atom<string>("");

// Range atom: Stores selected text range
export const rangeAtom = atom<{ from: number; to: number }>({
  from: 0,
  to: 0,
});
```

**What Are Atoms?**:
- Atoms are like tiny useState hooks that can be shared across components
- Unlike Context, they don't cause re-renders in components that don't use them
- Can be composed (derived atoms)

---

### Step 2: Using Atoms in Components

**File**: `apps/web/components/tailwind/generative/generative-menu-switch.tsx`

```tsx
// Line 10-60
import { useAtom } from "jotai";
import { queryAtom, rangeAtom } from "novel/utils/atoms";

export const GenerativeMenuSwitch = ({ open }: { open: boolean }) => {
  // Read AND write to atoms
  const [query, setQuery] = useAtom(queryAtom);
  const [range, setRange] = useAtom(rangeAtom);

  const { editor } = useEditor();

  useEffect(() => {
    if (!editor || !open) return;

    // When menu opens, capture current selection
    const { from, to } = editor.state.selection;
    const selectedText = editor.state.doc.textBetween(from, to);

    // Update atoms
    setRange({ from, to });
    setQuery(selectedText);
  }, [editor, open]);

  return (
    <div>
      {/* Display current query */}
      <p>Selected: {query}</p>

      {/* Buttons that use atoms */}
      <Button onClick={() => setQuery("")}>Clear</Button>
    </div>
  );
};
```

---

### Step 3: Atoms in AI Selector

**File**: `apps/web/components/tailwind/generative/ai-selector.tsx`

```tsx
// Line 15-50
import { useAtomValue, useSetAtom } from "jotai";
import { queryAtom, rangeAtom } from "novel/utils/atoms";

export const AISelector = () => {
  // Read-only access (doesn't trigger re-render on write)
  const query = useAtomValue(queryAtom);
  const range = useAtomValue(rangeAtom);

  // Write-only access (component doesn't re-render when value changes)
  const setQuery = useSetAtom(queryAtom);

  const { editor } = useEditor();

  const handleAICommand = (option: string) => {
    // Use range from atom
    editor?.chain()
      .focus()
      .deleteRange(range)  // Delete selected text
      .run();

    // Call AI with query from atom
    generateAI(query, option);

    // Clear query after generating
    setQuery("");
  };

  return (
    <CommandList>
      <CommandItem onSelect={() => handleAICommand("continue")}>
        Continue writing
      </CommandItem>
      {/* More commands... */}
    </CommandList>
  );
};
```

**Why Use Atoms Here?**:
- `GenerativeMenuSwitch` captures selection → stores in atoms
- `AISelector` reads from atoms → no prop drilling
- Multiple components can access same atoms
- Components only re-render when atoms they *read* change

---

### Step 4: Derived Atoms (Advanced)

```tsx
// Example: Atom that derives from another atom
import { atom } from "jotai";
import { queryAtom } from "./atoms";

// This atom automatically updates when queryAtom changes
export const queryLengthAtom = atom((get) => {
  const query = get(queryAtom);  // Read from another atom
  return query.length;
});

// Usage in component:
const queryLength = useAtomValue(queryLengthAtom);
// This component re-renders when query length changes,
// but NOT when query content changes (only length matters)
```

---

### Step 5: Jotai vs Editor State

**When to use each**:

| State Type | Use Editor State | Use Jotai Atoms |
|------------|------------------|-----------------|
| Document content | ✅ | ❌ |
| Selection position | ✅ | 🤔 Can use both |
| Formatting (bold, italic) | ✅ | ❌ |
| UI state (menus open/closed) | ❌ | ✅ |
| AI query text | ❌ | ✅ |
| Color picker selection | ❌ | ✅ |
| User preferences | ❌ | ✅ |

**Example**:
```tsx
// ❌ BAD: Storing document content in atoms
const [content, setContent] = useAtom(contentAtom);
editor.on("update", () => setContent(editor.getJSON()));  // Don't do this!

// ✅ GOOD: Storing UI state in atoms
const [isAIMenuOpen, setIsAIMenuOpen] = useAtom(aiMenuOpenAtom);

// ✅ GOOD: Storing derived state in atoms
const [selectedTextLength, setSelectedTextLength] = useAtom(selectedTextLengthAtom);
editor.on("selectionUpdate", ({ editor }) => {
  const { from, to } = editor.state.selection;
  setSelectedTextLength(to - from);
});
```

**Jotai State Management Summary**:
```
1. Define atoms (lightweight state containers)
2. Use useAtom() to read AND write
3. Use useAtomValue() for read-only (performance)
4. Use useSetAtom() for write-only (performance)
5. Atoms can derive from other atoms
6. Use for UI state, NOT document content
```

---

## Tour 6: Bubble Menu Implementation

**Feature**: Text formatting menu appears when text is selected

**Files We'll Visit**:
1. `packages/headless/src/components/editor-bubble.tsx` - Bubble menu component
2. `apps/web/components/tailwind/advanced-editor.tsx` - Usage example

### Step 1: Bubble Menu Component

**File**: `packages/headless/src/components/editor-bubble.tsx`

```tsx
// Line 15-60
import { BubbleMenu as TiptapBubbleMenu } from "@tiptap/react";
import { useEditor } from "./editor";

export interface EditorBubbleProps {
  children: React.ReactNode;
  className?: string;
}

export const EditorBubble = ({ children, className }: EditorBubbleProps) => {
  const { editor } = useEditor();

  if (!editor) return null;

  return (
    <TiptapBubbleMenu
      editor={editor}
      tippyOptions={{
        duration: 150,             // Animation duration
        placement: "top",           // Appear above selection
        popperOptions: {
          modifiers: [
            {
              name: "flip",
              options: {
                fallbackPlacements: ["bottom"],  // Flip to bottom if no space above
              },
            },
          ],
        },
      }}
      shouldShow={({ editor, from, to }) => {
        // CONDITION 1: Text must be selected
        if (from === to) {
          return false;  // No selection
        }

        // CONDITION 2: Selection must be in same block
        const isInSameBlock = editor.state.selection.$from.parent === editor.state.selection.$to.parent;
        if (!isInSameBlock) {
          return false;  // Multi-block selection
        }

        // CONDITION 3: Not in code block
        if (editor.isActive("codeBlock")) {
          return false;  // Code blocks have their own menu
        }

        return true;  // Show bubble menu
      }}
      className={className}
    >
      {children}
    </TiptapBubbleMenu>
  );
};
```

**Key Logic**:
1. `shouldShow` determines when menu appears
2. Checks: selection exists, single block, not in code block
3. Tippy.js handles positioning and animation

---

### Step 2: Bubble Menu Items

**File**: `packages/headless/src/components/editor-bubble.tsx` (continued)

```tsx
// Line 70-120
export interface EditorBubbleItemProps {
  children: React.ReactNode;
  onSelect?: (editor: Editor) => void;
  isActive?: (editor: Editor) => boolean;
  className?: string;
}

export const EditorBubbleItem = ({
  children,
  onSelect,
  isActive,
  className,
}: EditorBubbleItemProps) => {
  const { editor } = useEditor();

  if (!editor) return null;

  // Check if this item is active (e.g., bold is currently applied)
  const active = isActive ? isActive(editor) : false;

  return (
    <button
      type="button"
      onClick={() => {
        if (onSelect) {
          onSelect(editor);
          editor.chain().focus().run();  // Keep editor focused
        }
      }}
      className={cx(
        "bubble-menu-item",
        active && "bubble-menu-item-active",
        className
      )}
    >
      {children}
    </button>
  );
};
```

**Usage Pattern**:
```tsx
<EditorBubbleItem
  onSelect={(editor) => editor.chain().toggleBold().run()}
  isActive={(editor) => editor.isActive("bold")}
>
  Bold
</EditorBubbleItem>
```

---

### Step 3: Using Bubble Menu in Editor

**File**: `apps/web/components/tailwind/advanced-editor.tsx`

```tsx
// Line 100-150
import {
  EditorBubble,
  EditorBubbleItem,
} from "novel";
import { Bold, Italic, Underline, Code } from "lucide-react";

export default function AdvancedEditor() {
  return (
    <EditorRoot>
      {/* Bubble menu for text formatting */}
      <EditorBubble className="flex gap-1 p-1 bg-background border rounded-lg shadow-lg">
        <EditorBubbleItem
          onSelect={(editor) => editor.chain().toggleBold().run()}
          isActive={(editor) => editor.isActive("bold")}
        >
          <Bold className="h-4 w-4" />
        </EditorBubbleItem>

        <EditorBubbleItem
          onSelect={(editor) => editor.chain().toggleItalic().run()}
          isActive={(editor) => editor.isActive("italic")}
        >
          <Italic className="h-4 w-4" />
        </EditorBubbleItem>

        <EditorBubbleItem
          onSelect={(editor) => editor.chain().toggleUnderline().run()}
          isActive={(editor) => editor.isActive("underline")}
        >
          <Underline className="h-4 w-4" />
        </EditorBubbleItem>

        <EditorBubbleItem
          onSelect={(editor) => editor.chain().toggleCode().run()}
          isActive={(editor) => editor.isActive("code")}
        >
          <Code className="h-4 w-4" />
        </EditorBubbleItem>
      </EditorBubble>

      <EditorContent
        extensions={extensions}
        // ...
      />
    </EditorRoot>
  );
}
```

---

### Step 4: Advanced Bubble Menu with AI

**File**: `apps/web/components/tailwind/advanced-editor.tsx` (continued)

```tsx
// Line 200-250
<EditorBubble>
  <div className="flex gap-2">
    {/* Standard formatting */}
    <div className="flex gap-1 border-r pr-2">
      <EditorBubbleItem onSelect={(editor) => editor.chain().toggleBold().run()}>
        <Bold />
      </EditorBubbleItem>
      <EditorBubbleItem onSelect={(editor) => editor.chain().toggleItalic().run()}>
        <Italic />
      </EditorBubbleItem>
    </div>

    {/* AI commands */}
    <div className="flex gap-1">
      <EditorBubbleItem
        onSelect={(editor) => {
          const { from, to } = editor.state.selection;
          const text = editor.state.doc.textBetween(from, to);
          // Trigger AI improvement
          improveText(text);
        }}
      >
        <Sparkles className="h-4 w-4" />
        Improve
      </EditorBubbleItem>

      <EditorBubbleItem
        onSelect={(editor) => {
          const { from, to } = editor.state.selection;
          const text = editor.state.doc.textBetween(from, to);
          // Trigger AI shortening
          shortenText(text);
        }}
      >
        <Minimize className="h-4 w-4" />
        Shorten
      </EditorBubbleItem>
    </div>
  </div>
</EditorBubble>
```

**Bubble Menu Flow Summary**:
```
1. User selects text
2. shouldShow() checks conditions
3. If true, Tippy.js positions menu above selection
4. User clicks menu item
5. onSelect() executes command
6. editor.chain().focus() keeps focus
7. Menu stays visible (selection unchanged)
```

---

## Tour 7: Building a Custom Extension

**Goal**: Create a "Highlight" extension from scratch

### Step 1: Define Extension Structure

```tsx
// File: extensions/highlight.ts
import { Mark, mergeAttributes } from "@tiptap/core";

// STEP 1: Define options interface
export interface HighlightOptions {
  colors: string[];
  HTMLAttributes: Record<string, any>;
}

// STEP 2: Declare commands in TypeScript
declare module "@tiptap/core" {
  interface Commands<ReturnType> {
    highlight: {
      setHighlight: (attributes?: { color: string }) => ReturnType;
      toggleHighlight: (attributes?: { color: string }) => ReturnType;
      unsetHighlight: () => ReturnType;
    };
  }
}

// STEP 3: Create extension
export const Highlight = Mark.create<HighlightOptions>({
  name: "highlight",

  // STEP 4: Define default options
  addOptions() {
    return {
      colors: ["#ffeb3b", "#4caf50", "#2196f3", "#f44336"],  // Yellow, Green, Blue, Red
      HTMLAttributes: {},
    };
  },

  // STEP 5: Define attributes
  addAttributes() {
    return {
      color: {
        default: this.options.colors[0],  // Default to first color
        parseHTML: (element) => element.getAttribute("data-color") || element.style.backgroundColor,
        renderHTML: (attributes) => {
          if (!attributes.color) return {};
          return {
            "data-color": attributes.color,
            style: `background-color: ${attributes.color}`,
          };
        },
      },
    };
  },

  // STEP 6: Parse from HTML (when loading content)
  parseHTML() {
    return [
      {
        tag: "mark",
      },
      {
        style: "background-color",
        getAttrs: (value) => {
          // Only match if background color is one of our colors
          return this.options.colors.includes(value) ? {} : false;
        },
      },
    ];
  },

  // STEP 7: Render to HTML (when saving/exporting)
  renderHTML({ HTMLAttributes }) {
    return ["mark", mergeAttributes(this.options.HTMLAttributes, HTMLAttributes), 0];
  },

  // STEP 8: Add commands
  addCommands() {
    return {
      setHighlight:
        (attributes) =>
        ({ commands }) => {
          return commands.setMark(this.name, attributes);
        },

      toggleHighlight:
        (attributes) =>
        ({ commands }) => {
          return commands.toggleMark(this.name, attributes);
        },

      unsetHighlight:
        () =>
        ({ commands }) => {
          return commands.unsetMark(this.name);
        },
    };
  },

  // STEP 9: Add keyboard shortcuts
  addKeyboardShortcuts() {
    return {
      "Mod-Shift-h": () => this.editor.commands.toggleHighlight({ color: this.options.colors[0] }),
    };
  },
});
```

---

### Step 2: Register Extension

```tsx
// File: components/my-editor.tsx
import { Highlight } from "@/extensions/highlight";

const extensions = [
  StarterKit,
  Highlight.configure({
    colors: ["#ffeb3b", "#4caf50", "#2196f3", "#f44336", "#9c27b0"],
  }),
];

const editor = useEditor({
  extensions,
});
```

---

### Step 3: Add UI for Highlight

```tsx
// File: components/HighlightPicker.tsx
import { useEditor } from "novel";
import { Highlighter } from "lucide-react";
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from "@/components/ui/popover";

const COLORS = ["#ffeb3b", "#4caf50", "#2196f3", "#f44336", "#9c27b0"];

export const HighlightPicker = () => {
  const { editor } = useEditor();

  if (!editor) return null;

  return (
    <Popover>
      <PopoverTrigger asChild>
        <button
          className={editor.isActive("highlight") ? "active" : ""}
        >
          <Highlighter className="h-4 w-4" />
        </button>
      </PopoverTrigger>
      <PopoverContent className="w-auto p-2">
        <div className="flex gap-2">
          {COLORS.map((color) => (
            <button
              key={color}
              onClick={() => editor.chain().focus().toggleHighlight({ color }).run()}
              className="w-8 h-8 rounded"
              style={{ backgroundColor: color }}
            />
          ))}
          <button
            onClick={() => editor.chain().focus().unsetHighlight().run()}
            className="w-8 h-8 rounded border"
          >
            ✕
          </button>
        </div>
      </PopoverContent>
    </Popover>
  );
};
```

---

### Step 4: Test Extension

```tsx
// In your editor
<EditorBubble>
  <Bold />
  <Italic />
  <HighlightPicker />  {/* ← Your custom extension */}
</EditorBubble>

// Test cases:
// 1. Select text → Click highlight color → Text gets highlighted
// 2. Cmd+Shift+H → Text gets highlighted with default color
// 3. Save document → Reload → Highlights persist
// 4. Export to HTML → <mark style="background-color: #ffeb3b">text</mark>
```

---

### Step 5: Advanced: Input Rules

Add Markdown-style highlighting: `==highlighted text==`

```tsx
// File: extensions/highlight.ts (add to Highlight.create)
import { markInputRule } from "@tiptap/core";

export const Highlight = Mark.create({
  // ... previous code

  // STEP 10: Add input rules
  addInputRules() {
    return [
      markInputRule({
        find: /(?:^|\s)((?:==)((?:[^=]+))(?:==))$/,  // Match ==text==
        type: this.type,
        getAttributes: () => ({
          color: this.options.colors[0],  // Use default color
        }),
      }),
    ];
  },

  // ... rest of code
});

// Now typing:  This is ==highlighted== text
// Becomes:     This is <mark style="background-color: #ffeb3b">highlighted</mark> text
```

---

### Step 6: Advanced: Paste Rules

Auto-highlight when pasting from certain sources:

```tsx
// File: extensions/highlight.ts (add to Highlight.create)
import { markPasteRule } from "@tiptap/core";

export const Highlight = Mark.create({
  // ... previous code

  // STEP 11: Add paste rules
  addPasteRules() {
    return [
      markPasteRule({
        find: /==([^=]+)==/g,  // Match ==text== when pasting
        type: this.type,
        getAttributes: () => ({
          color: this.options.colors[0],
        }),
      }),
    ];
  },

  // ... rest of code
});
```

---

### Building Custom Extension Summary

**Checklist**:
1. ✅ Define extension type (Mark, Node, or Extension)
2. ✅ Define options interface
3. ✅ Declare commands in TypeScript
4. ✅ Implement `addOptions()`
5. ✅ Implement `addAttributes()`
6. ✅ Implement `parseHTML()` (loading)
7. ✅ Implement `renderHTML()` (saving)
8. ✅ Implement `addCommands()`
9. ✅ (Optional) Implement `addKeyboardShortcuts()`
10. ✅ (Optional) Implement `addInputRules()`
11. ✅ (Optional) Implement `addPasteRules()`
12. ✅ Register extension in editor
13. ✅ Create UI for extension
14. ✅ Test all functionality

**Extension Types**:
- **Mark**: Inline formatting (highlight, bold, link, comment)
- **Node**: Block content (callout, image, video, table)
- **Extension**: Behavior only (auto-save, analytics, spellcheck)

---

## Summary

We've completed 7 comprehensive code tours:

### ✅ Tour 1: Slash Command Flow
- User types "/" → Suggestion plugin → Filter commands → Render menu → Execute command
- Files: `slash-command/index.tsx`, `groups.tsx`, `command-list.tsx`

### ✅ Tour 2: AI Completion Flow
- User triggers AI → Frontend (useCompletion) → Backend (streamText) → Streaming response → Insert
- Files: `ai-selector.tsx`, `app/api/generate/route.ts`, `ai-highlight.ts`

### ✅ Tour 3: Image Upload Flow
- Drop/paste → Plugin intercepts → Upload to Vercel Blob → Replace placeholder → Interactive resize
- Files: `upload-images.tsx`, `app/api/upload/route.ts`, `updated-image.tsx`

### ✅ Tour 4: Extension Registration
- Define extensions → Pass to useEditor → Tiptap registers (schema, commands, shortcuts) → Available
- Files: `editor.tsx`, `advanced-editor.tsx`

### ✅ Tour 5: State Management with Jotai
- Define atoms → Use useAtom/useAtomValue/useSetAtom → Share state without prop drilling
- Files: `atoms.ts`, `generative-menu-switch.tsx`, `ai-selector.tsx`

### ✅ Tour 6: Bubble Menu Implementation
- Select text → shouldShow checks → Tippy positions → User clicks → Command executes
- Files: `editor-bubble.tsx`, `advanced-editor.tsx`

### ✅ Tour 7: Building Custom Extension
- Create Mark/Node/Extension → Implement methods → Register → Create UI → Test
- Complete example: Highlight extension with colors, keyboard shortcuts, input rules

---

## Next Steps for Learning

1. **Read the Actual Code**: Now that you understand the flow, read the real implementation
   - Start with simple extensions (Bold, Italic)
   - Then move to complex ones (AIHighlight, Command)

2. **Build Your Own Extension**: Practice makes perfect
   - Start simple: Color text extension
   - Then complex: Todo list with checkboxes
   - Advanced: Comments with threads

3. **Explore ProseMirror**: Understand the underlying engine
   - Read [ProseMirror Guide](https://prosemirror.net/docs/guide/)
   - Learn about transactions, decorations, plugins

4. **Join the Community**: Ask questions, share knowledge
   - [Novel GitHub Discussions](https://github.com/steven-tey/novel/discussions)
   - [Tiptap Discord](https://discord.gg/WtJ49jGshW)

---

**Document Version**: 1.0
**Last Updated**: November 2025
**Related Guides**:
- [EDITOR_CORE_GUIDE.md](./EDITOR_CORE_GUIDE.md) - Deep dive into Tiptap & ProseMirror
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step tutorials
- [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Best practices

---

*End of CODE_TOURS.md*