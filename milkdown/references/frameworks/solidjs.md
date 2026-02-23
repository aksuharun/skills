# SolidJS Integration

No dedicated SolidJS package. Use `@milkdown/kit` or `@milkdown/crepe` with SolidJS's reactivity primitives.

## Installation

```bash
npm install @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## Kit Setup

```tsx
// MilkdownEditor.tsx
import { onCleanup, onMount, createSignal } from "solid-js";
import {
  Editor,
  rootCtx,
  defaultValueCtx,
  editorViewOptionsCtx,
} from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";
import { nord } from "@milkdown/theme-nord";
import "@milkdown/theme-nord/style.css";

interface Props {
  defaultValue?: string;
  readonly?: boolean;
  onChange?: (markdown: string) => void;
}

const MilkdownEditor = (props: Props) => {
  let ref: HTMLDivElement | undefined;
  let editor: Editor;

  onMount(async () => {
    editor = await Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, ref!);
        ctx.set(defaultValueCtx, props.defaultValue ?? "");
        if (props.onChange) {
          ctx.get(listenerCtx).markdownUpdated((ctx, markdown) => {
            props.onChange?.(markdown);
          });
        }
        if (props.readonly) {
          ctx.update(editorViewOptionsCtx, (prev) => ({
            ...prev,
            editable: () => false,
          }));
        }
      })
      .use(commonmark)
      .use(history)
      .use(listener)
      .create();
  });

  onCleanup(async () => {
    await editor?.destroy();
  });

  return <div ref={ref} />;
};

export default MilkdownEditor;
```

## Crepe Setup

```tsx
import { onCleanup, onMount } from "solid-js";
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";

interface Props {
  defaultValue?: string;
  onChange?: (markdown: string) => void;
}

const CrepeEditor = (props: Props) => {
  let ref: HTMLDivElement | undefined;
  let crepe: Crepe;

  onMount(async () => {
    crepe = new Crepe({
      root: ref!,
      defaultValue: props.defaultValue ?? "",
    });
    if (props.onChange) {
      crepe.on((listener) => {
        listener.markdownUpdated((markdown) => props.onChange?.(markdown));
      });
    }
    await crepe.create();
  });

  onCleanup(() => { crepe?.destroy(); });

  return <div ref={ref} />;
};
```

## Reactive Content with Signals

```tsx
import { createSignal, createEffect } from "solid-js";
import { replaceAll, getMarkdown as getMarkdownMacro } from "@milkdown/kit/utils";

// Access editor imperatively via a stored ref
const MilkdownWithSignal = () => {
  let ref: HTMLDivElement | undefined;
  let editorInstance: Editor | undefined;
  const [content, setContent] = createSignal("# Hello");

  // Sync external signal → editor
  createEffect(() => {
    const md = content();
    if (editorInstance) {
      editorInstance.action(replaceAll(md));
    }
  });

  onMount(async () => {
    editorInstance = await Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, ref!);
        ctx.set(defaultValueCtx, content());
        ctx.get(listenerCtx).markdownUpdated((ctx, md) => setContent(md));
      })
      .use(commonmark)
      .use(listener)
      .create();
  });

  onCleanup(() => editorInstance?.destroy());

  return <div ref={ref} />;
};
```

> Watch out for signal-driven update loops. Use a guard (compare old vs new value) before calling `replaceAll`.

## Getting Markdown Imperatively

```tsx
const MilkdownWithRef = () => {
  let ref: HTMLDivElement | undefined;
  let editorInstance: Editor | undefined;

  onMount(async () => {
    editorInstance = await Editor.make()
      .config((ctx) => { ctx.set(rootCtx, ref!); })
      .use(commonmark)
      .create();
  });

  onCleanup(() => editorInstance?.destroy());

  const handleSave = () => {
    const md = editorInstance?.action(getMarkdownMacro());
    console.log(md);
  };

  return (
    <>
      <div ref={ref} />
      <button onClick={handleSave}>Save</button>
    </>
  );
};
```

## Lazy Loading (Vite / Solid Start)

Milkdown uses browser APIs. In SSR contexts (SolidStart), lazy-load or guard with `isServer`:

```tsx
import { isServer } from "solid-js/web";
import { lazy } from "solid-js";

const MilkdownEditor = lazy(() => import("./MilkdownEditor"));

// In component
const App = () => (
  <>{!isServer && <MilkdownEditor defaultValue="# Hello" />}</>
);
```

## Vite Config

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import solid from "vite-plugin-solid";

export default defineConfig({
  plugins: [solid()],
  optimizeDeps: {
    include: ["@milkdown/kit", "@milkdown/crepe"],
  },
});
```

## Common Pitfalls

- **`ref` timing**: In SolidJS, `ref` is set synchronously before `onMount`. Use `ref!` with the non-null assertion inside `onMount`.
- **Reactive prop changes**: SolidJS props are getters. Access `props.defaultValue` inside the `onMount` callback (not destructured outside) to get the latest value.
- **`createEffect` loops**: When syncing a signal → editor and editor → signal, always compare values to break cycles.
- **`onCleanup` vs `onDestroy`**: Use `onCleanup` for resource cleanup in SolidJS (runs on component disposal and effect re-runs).
