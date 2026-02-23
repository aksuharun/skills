# React Integration

First-class support via `@milkdown/react`.

## Installation

```bash
npm install @milkdown/react @milkdown/kit
# For Crepe:
npm install @milkdown/crepe
```

## Crepe + React

```tsx
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";
import { Milkdown, MilkdownProvider, useEditor } from "@milkdown/react";

const CrepeEditor: React.FC = () => {
  const { get } = useEditor((root) => new Crepe({
    root,
    defaultValue: "# Hello, Milkdown!",
  }));
  return <Milkdown />;
};

export const App: React.FC = () => (
  <MilkdownProvider>
    <CrepeEditor />
  </MilkdownProvider>
);
```

## Kit + React

```tsx
import { Editor, rootCtx, defaultValueCtx } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import { nord } from "@milkdown/theme-nord";
import "@milkdown/theme-nord/style.css";
import { Milkdown, MilkdownProvider, useEditor } from "@milkdown/react";

const MilkdownEditor: React.FC<{ defaultValue?: string }> = ({
  defaultValue = "",
}) => {
  useEditor((root) =>
    Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, root);
        ctx.set(defaultValueCtx, defaultValue);
      })
      .use(commonmark)
      .use(history),
  );
  return <Milkdown />;
};

export const App: React.FC = () => (
  <MilkdownProvider>
    <MilkdownEditor defaultValue="# Hello" />
  </MilkdownProvider>
);
```

## Accessing the Editor Instance

`useInstance()` must be used inside `MilkdownProvider` children — not in the same component that renders the provider.

```tsx
import { useInstance } from "@milkdown/react";
import { getMarkdown, replaceAll } from "@milkdown/kit/utils";

const EditorControls: React.FC = () => {
  const [isLoading, getInstance] = useInstance();

  const handleGetMarkdown = () => {
    if (isLoading) return;
    const md = getInstance()?.action(getMarkdown());
    console.log(md);
  };

  const handleSetContent = (markdown: string) => {
    if (isLoading) return;
    getInstance()?.action(replaceAll(markdown));
  };

  return (
    <div>
      <button onClick={handleGetMarkdown} disabled={isLoading}>Get Markdown</button>
      <button onClick={() => handleSetContent("# New Content")} disabled={isLoading}>
        Set Content
      </button>
    </div>
  );
};

// ✅ Correct: both editor and controls are children of MilkdownProvider
export const EditorWithControls: React.FC = () => (
  <MilkdownProvider>
    <MilkdownEditor />
    <EditorControls />
  </MilkdownProvider>
);
```

## Listening for Changes & Auto-save

```tsx
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

const AutoSaveEditor: React.FC = () => {
  useEditor((root) =>
    Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, root);
        ctx.get(listenerCtx)
          .markdownUpdated((ctx, markdown, prev) => {
            // auto-save on every change
            localStorage.setItem("draft", markdown);
          })
          .updated((ctx, doc) => {
            // access raw ProseMirror doc
            const json = doc.toJSON();
          })
          .selectionUpdated((ctx, selection) => {
            // react to selection changes
          });
      })
      .use(commonmark)
      .use(listener),
  );
  return <Milkdown />;
};
```

> `markdownUpdated` is expensive on large docs. For performance-critical use cases, listen to `updated` and call `editor.action(getMarkdown())` only when needed (e.g., on save button click).

## Controlled Mode (debounced sync)

```tsx
import { useRef, useEffect } from "react";
import { replaceAll, getMarkdown } from "@milkdown/kit/utils";
import { useInstance } from "@milkdown/react";

const ControlledEditor: React.FC<{
  value: string;
  onChange: (md: string) => void;
}> = ({ value, onChange }) => {
  const [isLoading, getInstance] = useInstance();
  const prevValueRef = useRef(value);

  // Sync external value → editor (avoid loop)
  useEffect(() => {
    if (isLoading) return;
    if (value !== prevValueRef.current) {
      getInstance()?.action(replaceAll(value));
      prevValueRef.current = value;
    }
  }, [value, isLoading]);

  // Listen for editor changes → call onChange
  useEditor((root) =>
    Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, root);
        ctx.get(listenerCtx).markdownUpdated((ctx, md) => {
          prevValueRef.current = md;
          onChange(md);
        });
      })
      .use(commonmark)
      .use(listener),
  );

  return <Milkdown />;
};
```

## Form Integration

```tsx
import { useRef } from "react";
import { getMarkdown } from "@milkdown/kit/utils";
import { useInstance } from "@milkdown/react";

const FormEditor: React.FC = () => {
  const [isLoading, getInstance] = useInstance();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (isLoading) return;
    const content = getInstance()?.action(getMarkdown());
    // submit content to your API
    console.log(content);
  };

  return (
    <form onSubmit={handleSubmit}>
      <MilkdownEditor />
      <button type="submit" disabled={isLoading}>Submit</button>
    </form>
  );
};

// Wrap with MilkdownProvider at appropriate level
export const FormWithEditor: React.FC = () => (
  <MilkdownProvider>
    <FormEditor />
  </MilkdownProvider>
);
```

## Readonly Mode

```tsx
// Crepe
const { get } = useEditor((root) => {
  const crepe = new Crepe({ root });
  crepe.setReadonly(true);
  return crepe;
});

// Kit via editor config
useEditor((root) =>
  Editor.make()
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.update(editorViewOptionsCtx, (prev) => ({
        ...prev,
        editable: () => false,
      }));
    })
    .use(commonmark),
);

// Toggle at runtime using useInstance
const [, getInstance] = useInstance();
const toggleReadonly = (readonly: boolean) => {
  // Crepe via crepe instance
  const crepe = get(); // get() returns Crepe instance from useEditor
};
```

## Performance Tips

- Wrap `useEditor` config in `useMemo` if it depends on props that change frequently.
- Use `React.memo` on the editor wrapper component to prevent unnecessary remounts.
- Prefer `updated` listener + on-demand `getMarkdown()` over `markdownUpdated` for large documents.
- Pass `defaultValue` only once (on mount). Use `replaceAll` for subsequent programmatic updates.
