```skill
---
name: milkdown
description: Build WYSIWYG Markdown editors with Milkdown — a plugin-driven, ProseMirror-based editor framework. Use when working with @milkdown/crepe, @milkdown/kit, or integrating Milkdown into any framework.
---

# Milkdown

Plugin-driven WYSIWYG Markdown editor built on ProseMirror and remark. Headless (no default styles), fully extensible, framework-agnostic.

## Two Approaches

### Crepe — Ready-to-use editor

Feature-complete with built-in UI (toolbar, slash menu, tables, code blocks, image handling). Best for quick setup.

```bash
npm install @milkdown/crepe
```

```typescript
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css"; // or classic, nord (+ -dark variants)

const crepe = new Crepe({
  root: "#app",
  defaultValue: "# Hello, Milkdown!",
});
await crepe.create();
```

Crepe API: `crepe.create()`, `crepe.destroy()`, `crepe.getMarkdown()`, `crepe.setReadonly(bool)`, `crepe.editor` (underlying Editor instance), `crepe.on(listener => { ... })`.

`crepe.on` events: `markdownUpdated`, `updated`, `selectionUpdated`, `focus`, `blur`.

```typescript
crepe.on((listener) => {
  listener.focus((ctx) => { /* editor focused */ });
  listener.blur((ctx) => { /* editor blurred */ });
  listener.markdownUpdated((ctx, markdown) => { /* content changed */ });
});
```

Disable/configure features:

```typescript
const crepe = new Crepe({
  features: { [Crepe.Feature.CodeMirror]: false },
  featureConfigs: {
    [Crepe.Feature.LinkTooltip]: { inputPlaceholder: "Enter URL..." },
  },
});
```

Features: `CodeMirror`, `ListItem`, `LinkTooltip`, `ImageBlock`, `BlockEdit`, `Table`, `Toolbar`, `Cursor`, `Placeholder`, `Latex`.

### Kit — Build from scratch

Full control over features. Import everything from `@milkdown/kit/*`.

```bash
npm install @milkdown/kit
```

```typescript
import { Editor } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { history } from "@milkdown/kit/plugin/history";
import "@milkdown/kit/prose/view/style/prosemirror.css";

const editor = await Editor.make().use(commonmark).use(history).create();
```

Key Kit plugins (all from `@milkdown/kit/plugin/*`): `history`, `clipboard`, `cursor`, `listener`, `indent`, `upload`, `block`, `tooltip`, `slash`, `trailing`.

Presets: `@milkdown/kit/preset/commonmark`, `@milkdown/kit/preset/gfm`.

Components (from `@milkdown/kit/component/*`): `code-block`, `image-block`, `image-inline`, `link-tooltip`, `list-item-block`, `table-block`.

## Essential Patterns

### Set root and default value

```typescript
import { rootCtx, defaultValueCtx } from "@milkdown/kit/core";

Editor.make().config((ctx) => {
  ctx.set(rootCtx, "#editor"); // selector or DOM node
  ctx.set(defaultValueCtx, "# Hello");
});
```

Default value types: markdown string, `{ type: "html", dom }`, `{ type: "json", value }`.

### Listeners

```typescript
import { listener, listenerCtx } from "@milkdown/kit/plugin/listener";

Editor.make()
  .config((ctx) => {
    ctx.get(listenerCtx).mounted((ctx) => { /* editor mounted */ });
    ctx.get(listenerCtx).markdownUpdated((ctx, markdown, prev) => { /* save */ });
    ctx.get(listenerCtx).updated((ctx, doc, prevDoc) => { /* ProseMirror doc */ });
    ctx.get(listenerCtx).selectionUpdated((ctx, sel, prev) => { /* selection */ });
  })
  .use(listener);
```

Events: `mounted` (after editor initializes), `markdownUpdated`, `updated`, `selectionUpdated`, `focus`, `blur`.

> markdownUpdated has performance cost on large docs. Prefer `updated` + on-demand serialization.

### Macros (via `editor.action()`)

All from `@milkdown/kit/utils`:

```typescript
import {
  insert, insertPos, replaceAll, replaceRange,
  getMarkdown, getHTML, forceUpdate, setAttr,
  callCommand, outline, markdownToSlice
} from "@milkdown/kit/utils";

// Content manipulation
editor.action(insert("# New heading"));           // insert markdown at cursor
editor.action(insert("inline text", true));        // inline insert (second arg: inline flag)
editor.action(insertPos("Hello", 0));              // insert at specific position
editor.action(replaceAll("# Fresh content"));      // replace entire document content
editor.action(replaceAll("# Fresh content", true)); // replace + flush editor state (reinitializes plugins)
editor.action(replaceRange("Hello", { from: 0, to: 5 })); // replace specific range

// Content retrieval
const md = editor.action(getMarkdown());           // get full document as markdown
const partial = editor.action(getMarkdown({ from: 0, to: 10 })); // markdown for a range
const html = editor.action(getHTML());             // get full document as HTML

// Editor state
editor.action(forceUpdate());                     // force editor to update state
editor.action(setAttr(pos, (prev) => ({ ...prev, class: "custom" }))); // update node attrs at position

// Navigation / structure
const headings = editor.action(outline());         // get document outline

// Advanced
const slice = editor.action(markdownToSlice("# Hello")); // convert markdown to ProseMirror Slice
editor.action(callCommand(someCommand.key, arg));  // call a registered command
```

### Commands

```typescript
import { commandsCtx } from "@milkdown/kit/core";
import { toggleEmphasisCommand, toggleStrongCommand } from "@milkdown/kit/preset/commonmark";

editor.action((ctx) => {
  ctx.get(commandsCtx).call(toggleEmphasisCommand.key);
});
```

Chain multiple commands (executes in order until one returns `true`):

```typescript
editor.action((ctx) => {
  ctx.get(commandsCtx)
    .chain()
    .pipe(toggleEmphasisCommand.key)
    .pipe(toggleStrongCommand.key)
    .run();
});
```

Mix inline (ProseMirror) commands with registered commands:

```typescript
ctx.get(commandsCtx)
  .chain()
  .inline(someInlineCommand)
  .pipe(toggleEmphasisCommand.key)
  .run();
```

### Readonly mode

```typescript
// Crepe
crepe.setReadonly(true);
// Kit
import { editorViewOptionsCtx } from "@milkdown/kit/core";
Editor.make().config((ctx) => {
  ctx.update(editorViewOptionsCtx, (prev) => ({ ...prev, editable: () => false }));
});
```

### Keyboard shortcuts

**Commonmark preset defaults:**

| Shortcut | Action |
|---|---|
| `Mod-b` | Toggle bold |
| `Mod-i` | Toggle italic |
| `Mod-e` | Toggle inline code |
| `Mod-Alt-1..6` | Turn block into heading level 1–6 |
| `Mod-Alt-0` | Wrap in paragraph |
| `Delete`/`Backspace` | Downgrade heading level |
| `Mod-Shift-b` | Wrap in blockquote |
| `Mod-Shift-7` | Wrap in ordered list |
| `Mod-Shift-8` | Wrap in bullet list |
| `Mod-Shift-c` | Wrap in code block |
| `Shift-Enter` | Insert hard break |

**GFM preset additional shortcuts:**

| Shortcut | Action |
|---|---|
| `Mod-Alt-x` | Toggle strikethrough |
| `Mod-]` | Move to next table cell |
| `Mod-[` | Move to previous table cell |
| `Mod-Enter`/`Enter` | Exit table and break if possible |

Customize (accepts single string or array of strings):

```typescript
import { blockquoteKeymap } from "@milkdown/kit/preset/commonmark";
Editor.make().config((ctx) => {
  ctx.set(blockquoteKeymap.key, {
    WrapInBlockquote: ["Mod-Shift-b", "Mod-b"], // multiple shortcuts
  });
});
```

### Styling

Headless — style via `.milkdown .editor` selectors, or add attributes:

```typescript
import { headingAttr, paragraphAttr } from "@milkdown/kit/preset/commonmark";
Editor.make().config((ctx) => {
  ctx.set(headingAttr.key, (node) => ({ class: `heading-${node.attrs.level}` }));
});
```

Crepe themes use CSS variables (`--crepe-color-*`, `--crepe-font-*`, `--crepe-shadow-*`). Override on `.crepe .milkdown`.

### Toggling plugins at runtime

```typescript
await editor.remove(somePlugin);
editor.removeConfig(someConfig);
editor.use(anotherPlugin);
await editor.create(); // re-creates editor
```

### Destroying

```typescript
await editor.destroy();       // destroy, keep config
await editor.destroy(true);   // destroy + clear plugins/configs
await editor.create();        // recreate
```

### Editor status

```typescript
import { Editor, EditorStatus } from "@milkdown/kit/core";

const editor = Editor.make().use(/* plugins */);

// Inspect current status
console.log(editor.status); // EditorStatus.Idle | OnCreate | Created | OnDestroyed | Destroyed

// Listen to status changes
editor.onStatusChange((status: EditorStatus) => {
  console.log(status);
});
```

Status lifecycle: `Idle` → `OnCreate` → `Created` → `OnDestroyed` → `Destroyed`.

### Configure remark

```typescript
import { remarkStringifyOptionsCtx } from "@milkdown/kit/core";
editor.config((ctx) => {
  ctx.set(remarkStringifyOptionsCtx, { bullet: "*", fences: true });
});
```

## References

Read only the file(s) relevant to the current task.

### Framework Integration

See **[references/frameworks/](references/frameworks/)** — pick the file matching the project's framework (react, vue, nextjs, nuxt, angular, svelte, solidjs, vue2).

### Plugin & Extension Development

- **[references/kit-and-plugins.md](references/kit-and-plugins.md)** — Custom plugin creation (schema, commands, keymaps, input rules), plugin lifecycle, context system (Ctx/Slices/Timers), composable helpers, tooltip/slash/block plugin factories, custom node (iframe) and mark (marker) examples.

### Advanced Topics

- **[references/advanced.md](references/advanced.md)** — Collaborative editing (Y.js), code highlighting (Shiki/Lowlight/Refractor/Sugar High), ProseMirror API package mapping, Crepe theming with full CSS variable reference.

```
