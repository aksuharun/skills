# Vue 2 Integration

No dedicated Vue 2 package. Use `@milkdown/kit` or `@milkdown/crepe` directly with Vue 2's lifecycle hooks.

> Vue 2 reaches end of life. Consider upgrading to Vue 3 with `@milkdown/vue` for first-class support.

## Installation

```bash
npm install @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## Kit Setup

```vue
<template>
  <div ref="editorRef"></div>
</template>

<script>
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

export default {
  name: "MilkdownEditor",
  props: {
    defaultValue: {
      type: String,
      default: "",
    },
    readonly: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return { editor: null };
  },
  async mounted() {
    this.editor = await Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, this.$refs.editorRef);
        ctx.set(defaultValueCtx, this.defaultValue);
        ctx.get(listenerCtx).markdownUpdated((ctx, markdown) => {
          this.$emit("change", markdown);
        });
        if (this.readonly) {
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
  },
  async beforeDestroy() {
    await this.editor?.destroy();
  },
  methods: {
    getMarkdown() {
      const { getMarkdown } = require("@milkdown/kit/utils");
      return this.editor?.action(getMarkdown());
    },
    setContent(markdown) {
      const { replaceAll } = require("@milkdown/kit/utils");
      this.editor?.action(replaceAll(markdown));
    },
    insertContent(markdown) {
      const { insert } = require("@milkdown/kit/utils");
      this.editor?.action(insert(markdown));
    },
  },
};
</script>
```

## Crepe Setup

```vue
<template>
  <div ref="editorRef"></div>
</template>

<script>
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";

export default {
  name: "CrepeEditor",
  props: {
    defaultValue: { type: String, default: "" },
  },
  data() {
    return { crepe: null };
  },
  async mounted() {
    this.crepe = new Crepe({
      root: this.$refs.editorRef,
      defaultValue: this.defaultValue,
    });
    this.crepe.on((listener) => {
      listener.markdownUpdated((markdown) => this.$emit("change", markdown));
    });
    await this.crepe.create();
  },
  methods: {
    getMarkdown() {
      return this.crepe?.getMarkdown();
    },
    setReadonly(readonly) {
      this.crepe?.setReadonly(readonly);
    },
  },
  beforeDestroy() {
    this.crepe?.destroy();
  },
};
</script>
```

## Reactive Default Value

Vue 2 cannot automatically sync the `defaultValue` prop to the editor after mount. Watch the prop and call `replaceAll`:

```vue
<script>
import { replaceAll } from "@milkdown/kit/utils";

export default {
  watch: {
    defaultValue(newVal) {
      this.editor?.action(replaceAll(newVal));
    },
  },
};
</script>
```

## Form Integration

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <milkdown-editor ref="milkdownRef" :default-value="initialContent" />
    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return { initialContent: "# Hello" };
  },
  methods: {
    handleSubmit() {
      const content = this.$refs.milkdownRef.getMarkdown();
      // submit to API
    },
  },
};
</script>
```

## Global Vue Styles

Vue 2 scoped styles don't apply to dynamically rendered DOM (ProseMirror content). Use unscoped styles:

```vue
<style>
/* Unscoped to reach ProseMirror content */
.milkdown .editor {
  padding: 1rem;
}
.milkdown .editor h1 {
  font-size: 2rem;
}
</style>
```

Or use `::v-deep` for scoped styles:

```vue
<style scoped>
::v-deep .milkdown .editor {
  padding: 1rem;
}
</style>
```

## Common Pitfalls

- **`mounted` vs `created`**: Use `mounted` — `$refs` are only available after the template is rendered.
- **`beforeDestroy` lifecycle**: Vue 2 uses `beforeDestroy` (not `beforeUnmount` as in Vue 3).
- **Direct DOM manipulation**: Milkdown takes ownership of the `ref` element's DOM. Don't manipulate it with Vue directives or other libraries.
- **CSS isolation**: Vue 2 scoped styles use `[data-v-xxxxxx]` attribute selectors that don't pierce ProseMirror's DOM. Use `::v-deep` or unscoped styles.
