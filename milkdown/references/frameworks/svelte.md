# Svelte Integration

No dedicated Svelte package. Use `@milkdown/kit` or `@milkdown/crepe` with Svelte's `use:` action or `onMount`.

## Installation

```bash
npm install @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## Kit Setup

```svelte
<!-- MilkdownEditor.svelte -->
<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { Editor, rootCtx, defaultValueCtx, editorViewOptionsCtx } from "@milkdown/kit/core";
  import { commonmark } from "@milkdown/kit/preset/commonmark";
  import { history } from "@milkdown/kit/plugin/history";
  import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";
  import { nord } from "@milkdown/theme-nord";
  import "@milkdown/theme-nord/style.css";

  export let defaultValue: string = "";
  export let readonly: boolean = false;

  let editorRef: HTMLDivElement;
  let editor: Editor;

  export function getMarkdown(): string {
    return editor?.action(getMarkdownMacro());
  }

  export function setContent(markdown: string): void {
    editor?.action(replaceAll(markdown));
  }

  onMount(async () => {
    editor = await Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, editorRef);
        ctx.set(defaultValueCtx, defaultValue);
        if (readonly) {
          ctx.update(editorViewOptionsCtx, (prev) => ({
            ...prev,
            editable: () => false,
          }));
        }
      })
      .use(commonmark)
      .use(history)
      .create();
  });

  onDestroy(async () => {
    await editor?.destroy();
  });
</script>

<div bind:this={editorRef}></div>
```

## Kit with Listeners and Events

```svelte
<script lang="ts">
  import { createEventDispatcher, onMount, onDestroy } from "svelte";
  import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

  export let defaultValue: string = "";

  const dispatch = createEventDispatcher<{ change: string }>();
  let editorRef: HTMLDivElement;
  let editor: Editor;

  onMount(async () => {
    editor = await Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, editorRef);
        ctx.set(defaultValueCtx, defaultValue);
        ctx.get(listenerCtx)
          .markdownUpdated((ctx, markdown) => dispatch("change", markdown))
          .updated((ctx, doc) => { /* raw ProseMirror doc */ })
          .selectionUpdated((ctx, selection) => { /* selection changed */ });
      })
      .use(commonmark)
      .use(listener)
      .create();
  });

  onDestroy(async () => { await editor?.destroy(); });
</script>

<div bind:this={editorRef}></div>
```

Usage:
```svelte
<MilkdownEditor {defaultValue} on:change={(e) => (content = e.detail)} />
```

## Crepe Setup

```svelte
<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { Crepe } from "@milkdown/crepe";
  import "@milkdown/crepe/theme/common/style.css";
  import "@milkdown/crepe/theme/frame.css";
  import { createEventDispatcher } from "svelte";

  export let defaultValue: string = "";

  const dispatch = createEventDispatcher<{ change: string }>();
  let editorRef: HTMLDivElement;
  let crepe: Crepe;

  export function getMarkdown(): string {
    return crepe?.getMarkdown() ?? "";
  }

  onMount(async () => {
    crepe = new Crepe({ root: editorRef, defaultValue });
    crepe.on((listener) => {
      listener.markdownUpdated((markdown) => dispatch("change", markdown));
    });
    await crepe.create();
  });

  onDestroy(() => { crepe?.destroy(); });
</script>

<div bind:this={editorRef}></div>
```

## Using Svelte `use:` Action

```svelte
<script lang="ts">
  import { Editor, rootCtx } from "@milkdown/kit/core";
  import { commonmark } from "@milkdown/kit/preset/commonmark";

  function milkdownEditor(node: HTMLElement) {
    let editor: Editor;

    Editor.make()
      .config((ctx) => { ctx.set(rootCtx, node); })
      .use(commonmark)
      .create()
      .then((e) => { editor = e; });

    return {
      destroy() {
        editor?.destroy();
      },
    };
  }
</script>

<div use:milkdownEditor></div>
```

## Auto-save with Svelte Store

```svelte
<script lang="ts">
  import { writable } from "svelte/store";
  import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

  export const content = writable("");

  onMount(async () => {
    editor = await Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, editorRef);
        ctx.get(listenerCtx).markdownUpdated((ctx, md) => content.set(md));
      })
      .use(commonmark)
      .use(listener)
      .create();
  });
</script>
```

## Readonly Toggle

```svelte
<script lang="ts">
  import { editorViewOptionsCtx } from "@milkdown/kit/core";

  export let readonly = false;

  onMount(async () => {
    editor = await Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, editorRef);
        // Reactive: reads `readonly` prop at call time
        ctx.update(editorViewOptionsCtx, (prev) => ({
          ...prev,
          editable: () => !readonly,
        }));
      })
      .use(commonmark)
      .create();
  });
</script>
```

> To react to `readonly` prop changes, you need to recreate the editor: `editor.destroy()` then `editor.create()`.

## SvelteKit Considerations

Add Milkdown to `optimizeDeps` in `vite.config.ts` if you encounter bundling issues:

```typescript
// vite.config.ts
export default {
  optimizeDeps: {
    include: ["@milkdown/kit", "@milkdown/crepe"],
  },
};
```

For SSR routes, guard editor creation in `onMount` (browser-only). Never instantiate Milkdown at module level.
