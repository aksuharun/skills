# Vue 3 Integration

First-class support via `@milkdown/vue`. Requires Vue 3.x.

## Installation

```bash
npm install @milkdown/vue @milkdown/kit
# For Crepe:
npm install @milkdown/crepe
```

## Crepe + Vue

```vue
<!-- MilkdownEditor.vue -->
<template>
  <Milkdown />
</template>

<script setup lang="ts">
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";
import { Milkdown, useEditor } from "@milkdown/vue";

const props = defineProps<{ defaultValue?: string }>();

const { get } = useEditor((root) =>
  new Crepe({
    root,
    defaultValue: props.defaultValue ?? "",
  }),
);
</script>
```

```vue
<!-- App.vue -->
<template>
  <MilkdownProvider>
    <MilkdownEditor default-value="# Hello" />
  </MilkdownProvider>
</template>

<script setup lang="ts">
import { MilkdownProvider } from "@milkdown/vue";
import MilkdownEditor from "./MilkdownEditor.vue";
</script>
```

## Kit + Vue

```vue
<!-- MilkdownEditor.vue -->
<template>
  <Milkdown />
</template>

<script setup lang="ts">
import { Editor, rootCtx, defaultValueCtx } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import { gfm } from "@milkdown/kit/preset/gfm";
import { nord } from "@milkdown/theme-nord";
import "@milkdown/theme-nord/style.css";
import { Milkdown, useEditor } from "@milkdown/vue";

const props = defineProps<{ defaultValue?: string }>();

const { get } = useEditor((root) =>
  Editor.make()
    .config(nord)
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.set(defaultValueCtx, props.defaultValue ?? "");
    })
    .use(commonmark)
    .use(history),
);
</script>
```

## Accessing the Editor Instance

`useInstance()` must be inside a `MilkdownProvider` descendant — not the same component that renders the provider.

```vue
<!-- EditorControls.vue -->
<template>
  <div>
    <button @click="handleGetMarkdown" :disabled="isLoading">Get Markdown</button>
    <button @click="handleSetContent" :disabled="isLoading">Set Content</button>
  </div>
</template>

<script setup lang="ts">
import { useInstance } from "@milkdown/vue";
import { getMarkdown, replaceAll } from "@milkdown/kit/utils";

const [isLoading, getInstance] = useInstance();

const handleGetMarkdown = () => {
  if (isLoading.value) return;
  const md = getInstance()?.action(getMarkdown());
  console.log(md);
};

const handleSetContent = () => {
  if (isLoading.value) return;
  getInstance()?.action(replaceAll("# New Content"));
};
</script>
```

> In Vue, `isLoading` is a `Ref<boolean>` — always access it as `isLoading.value` in script.

```vue
<!-- App.vue — correct structure -->
<template>
  <MilkdownProvider>
    <MilkdownEditor />
    <EditorControls />
  </MilkdownProvider>
</template>
```

## Listening for Changes & Auto-save

```vue
<script setup lang="ts">
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

const { get } = useEditor((root) =>
  Editor.make()
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.get(listenerCtx)
        .markdownUpdated((ctx, markdown) => {
          localStorage.setItem("draft", markdown);
        })
        .updated((ctx, doc) => {
          const json = doc.toJSON();
        })
        .selectionUpdated((ctx, selection) => {
          // handle selection
        });
    })
    .use(commonmark)
    .use(listener),
);
</script>
```

## Controlled Mode with v-model

```vue
<!-- ControlledEditor.vue -->
<template>
  <Milkdown />
</template>

<script setup lang="ts">
import { watch, ref } from "vue";
import { Milkdown, useEditor, useInstance } from "@milkdown/vue";
import { replaceAll, getMarkdown } from "@milkdown/kit/utils";
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

const props = defineProps<{ modelValue: string }>();
const emit = defineEmits<{ "update:modelValue": [value: string] }>();
const [isLoading, getInstance] = useInstance();

// Sync prop → editor (guard against feedback loop)
const internalValue = ref(props.modelValue);
watch(
  () => props.modelValue,
  (newVal) => {
    if (newVal !== internalValue.value && !isLoading.value) {
      getInstance()?.action(replaceAll(newVal));
      internalValue.value = newVal;
    }
  },
);

useEditor((root) =>
  Editor.make()
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.set(defaultValueCtx, props.modelValue);
      ctx.get(listenerCtx).markdownUpdated((ctx, md) => {
        internalValue.value = md;
        emit("update:modelValue", md);
      });
    })
    .use(commonmark)
    .use(listener),
);
</script>
```

Usage: `<ControlledEditor v-model="content" />`

## Form Integration

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <MilkdownProvider>
      <MilkdownEditor />
      <SubmitButton />
    </MilkdownProvider>
  </form>
</template>

<script setup lang="ts">
// SubmitButton.vue (inside MilkdownProvider)
import { useInstance } from "@milkdown/vue";
import { getMarkdown } from "@milkdown/kit/utils";

const [isLoading, getInstance] = useInstance();

const handleSubmit = () => {
  if (isLoading.value) return;
  const content = getInstance()?.action(getMarkdown());
  // submit to API
};
</script>
```

## Readonly Mode

```vue
<script setup lang="ts">
import { editorViewOptionsCtx } from "@milkdown/kit/core";

// Static readonly
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

// Dynamic readonly with a reactive ref
const readonly = ref(false);
useEditor((root) =>
  Editor.make()
    .config((ctx) => {
      ctx.set(rootCtx, root);
      ctx.update(editorViewOptionsCtx, (prev) => ({
        ...prev,
        editable: () => !readonly.value,
      }));
    })
    .use(commonmark),
);
// Toggle: readonly.value = true
</script>
```

## Scoped Styles with Portals

Tooltip, slash menus, and other portal-based components render outside the component root. Use `:deep()` inside `<style scoped>`:

```vue
<style scoped>
:deep(.milkdown .editor) {
  padding: 1rem;
}
:deep(.milkdown-menu) {
  /* styles for portaled menus */
}
</style>
```
