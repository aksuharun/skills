# Next.js Integration

Uses `@milkdown/react` — same patterns as React with SSR considerations.

## Installation

```bash
npm install @milkdown/react @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## SSR / Dynamic Import

Milkdown runs in the browser only. In Next.js, always wrap the editor in `dynamic` with `ssr: false`.

```tsx
// components/Editor.tsx
import dynamic from "next/dynamic";

const MilkdownEditorWrapper = dynamic(
  () => import("./MilkdownEditorWrapper"),
  { ssr: false, loading: () => <p>Loading editor...</p> },
);

export default MilkdownEditorWrapper;
```

## Crepe + Next.js

```tsx
// components/MilkdownEditorWrapper.tsx
"use client"; // App Router
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";
import { Milkdown, MilkdownProvider, useEditor } from "@milkdown/react";

interface Props {
  defaultValue?: string;
  onChange?: (markdown: string) => void;
}

const CrepeEditorInner: React.FC<Props> = ({ defaultValue = "", onChange }) => {
  const { get } = useEditor((root) => {
    const crepe = new Crepe({ root, defaultValue });
    if (onChange) {
      crepe.on((listener) => {
        listener.markdownUpdated((markdown) => onChange(markdown));
      });
    }
    return crepe;
  });
  return <Milkdown />;
};

export default function MilkdownEditorWrapper(props: Props) {
  return (
    <MilkdownProvider>
      <CrepeEditorInner {...props} />
    </MilkdownProvider>
  );
}
```

## Kit + Next.js

```tsx
// components/MilkdownEditorWrapper.tsx
"use client";
import { Editor, rootCtx, defaultValueCtx } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";
import { nord } from "@milkdown/theme-nord";
import "@milkdown/theme-nord/style.css";
import { Milkdown, MilkdownProvider, useEditor } from "@milkdown/react";

interface Props {
  defaultValue?: string;
  onChange?: (markdown: string) => void;
  readonly?: boolean;
}

const EditorInner: React.FC<Props> = ({ defaultValue = "", onChange, readonly }) => {
  useEditor((root) =>
    Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, root);
        ctx.set(defaultValueCtx, defaultValue);
        if (onChange) {
          ctx.get(listenerCtx).markdownUpdated((ctx, markdown) => onChange(markdown));
        }
        if (readonly) {
          ctx.update(editorViewOptionsCtx, (prev) => ({ ...prev, editable: () => false }));
        }
      })
      .use(commonmark)
      .use(history)
      .use(listener),
  );
  return <Milkdown />;
};

export default function MilkdownEditorWrapper(props: Props) {
  return (
    <MilkdownProvider>
      <EditorInner {...props} />
    </MilkdownProvider>
  );
}
```

## Usage in Pages/App Router

```tsx
// app/page.tsx or pages/index.tsx
import dynamic from "next/dynamic";
import { useState } from "react";

const Editor = dynamic(() => import("../components/MilkdownEditorWrapper"), {
  ssr: false,
});

export default function Page() {
  const [content, setContent] = useState("# Hello, Next.js!");

  return (
    <main>
      <Editor defaultValue={content} onChange={setContent} />
      <pre>{content}</pre>
    </main>
  );
}
```

## Importing CSS in Next.js

Add CSS imports in `app/layout.tsx` (App Router) or `pages/_app.tsx` (Pages Router), not inside components:

```tsx
// app/layout.tsx
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";
// or
import "@milkdown/theme-nord/style.css";
```

## Accessing Editor Imperatively

```tsx
"use client";
import { useInstance } from "@milkdown/react";
import { getMarkdown, replaceAll } from "@milkdown/kit/utils";

// Inside MilkdownProvider subtree
const SaveButton: React.FC = () => {
  const [isLoading, getInstance] = useInstance();

  const save = async () => {
    const md = getInstance()?.action(getMarkdown());
    await fetch("/api/save", {
      method: "POST",
      body: JSON.stringify({ content: md }),
    });
  };

  return <button onClick={save} disabled={isLoading}>Save</button>;
};
```

## Common Pitfalls

- **CSS not loading**: Import Milkdown CSS in `layout.tsx` / `_app.tsx`, not inside dynamic components.
- **Hydration errors**: Always use `dynamic` with `ssr: false` — Milkdown relies on browser APIs.
- **"window is not defined"**: Same cause — ensure no server-side execution.
- **`"use client"` directive**: Required for all Milkdown components in App Router.
