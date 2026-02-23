# Nuxt 3 Integration

Uses `@milkdown/vue` — same patterns as Vue 3 with SSR/client considerations.

## Installation

```bash
npm install @milkdown/vue @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## Client-only Rendering

Milkdown requires browser APIs. Wrap in `<ClientOnly>` or use `defineAsyncComponent` with a placeholder.

```vue
<!-- pages/index.vue -->
<template>
  <ClientOnly fallback="Loading editor...">
    <MilkdownEditorWrapper default-value="# Hello, Nuxt!" />
  </ClientOnly>
</template>
```

## Crepe + Nuxt

```vue
<!-- components/MilkdownEditorWrapper.vue -->
<template>
  <MilkdownProvider>
    <CrepeEditorInner v-bind="$props" />
  </MilkdownProvider>
</template>

<script setup lang="ts">
import { MilkdownProvider } from "@milkdown/vue";

defineProps<{ defaultValue?: string }>();
</script>
```

```vue
<!-- components/CrepeEditorInner.vue -->
<template>
  <Milkdown />
</template>

<script setup lang="ts">
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";
import { Milkdown, useEditor } from "@milkdown/vue";

const props = defineProps<{ defaultValue?: string }>();
const emit = defineEmits<{ change: [markdown: string] }>();

const { get } = useEditor((root) => {
  const crepe = new Crepe({ root, defaultValue: props.defaultValue ?? "" });
  crepe.on((listener) => {
    listener.markdownUpdated((markdown) => emit("change", markdown));
  });
  return crepe;
});
</script>
```

## Kit + Nuxt

```vue
<!-- components/MilkdownEditorInner.vue -->
<template>
  <Milkdown />
</template>

<script setup lang="ts">
import { Editor, rootCtx, defaultValueCtx, editorViewOptionsCtx } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import { gfm } from "@milkdown/kit/preset/gfm";
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";
import { nord } from "@milkdown/theme-nord";
import { Milkdown, useEditor } from "@milkdown/vue";

const props = defineProps<{
  defaultValue?: string;
  readonly?: boolean;
}>();
const emit = defineEmits<{ change: [markdown: string] }>();

useEditor((root) =>
  Editor.make()
    .config(nord)
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.set(defaultValueCtx, props.defaultValue ?? "");
      ctx.get(listenerCtx).markdownUpdated((ctx, md) => emit("change", md));
      if (props.readonly) {
        ctx.update(editorViewOptionsCtx, (prev) => ({
          ...prev,
          editable: () => false,
        }));
      }
    })
    .use(commonmark)
    .use(history)
    .use(gfm)
    .use(listener),
);
</script>
```

```vue
<!-- components/MilkdownEditorWrapper.vue -->
<template>
  <MilkdownProvider>
    <MilkdownEditorInner v-bind="$props" @change="$emit('change', $event)" />
  </MilkdownProvider>
</template>

<script setup lang="ts">
import { MilkdownProvider } from "@milkdown/vue";

defineProps<{ defaultValue?: string; readonly?: boolean }>();
defineEmits<{ change: [markdown: string] }>();
</script>
```

## Accessing Editor From Outside

```vue
<!-- components/EditorWithControls.vue -->
<template>
  <MilkdownProvider>
    <MilkdownEditorInner />
    <EditorToolbar />
  </MilkdownProvider>
</template>
```

```vue
<!-- components/EditorToolbar.vue — inside MilkdownProvider subtree -->
<template>
  <button @click="handleSave" :disabled="isLoading">Save</button>
</template>

<script setup lang="ts">
import { useInstance } from "@milkdown/vue";
import { getMarkdown } from "@milkdown/kit/utils";

const [isLoading, getInstance] = useInstance();
// isLoading is a Ref<boolean> — use .value in script

const handleSave = () => {
  if (isLoading.value) return;
  const content = getInstance()?.action(getMarkdown());
  // persist content
};
</script>
```

## Nuxt Config for CSS

Add Milkdown CSS globally via `nuxt.config.ts`:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  css: [
    "@milkdown/crepe/theme/common/style.css",
    "@milkdown/crepe/theme/frame.css",
    // or "@milkdown/theme-nord/style.css"
  ],
});
```

## Common Pitfalls

- **SSR errors**: Always use `<ClientOnly>` wrapper — Milkdown uses `document` and `window`.
- **CSS import order**: Import `common/style.css` before theme-specific CSS.
- **`isLoading` is a Ref**: Access as `isLoading.value`, not `isLoading` in script context.
- **Teleported elements**: Slash menus and tooltips render in `<body>`. Use `:deep()` in scoped styles.
