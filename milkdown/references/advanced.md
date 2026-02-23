# Advanced

## Collaborative Editing

Powered by Y.js via `@milkdown/plugin-collab`.

```bash
npm install @milkdown/plugin-collab yjs y-protocols y-prosemirror
```

Choose a [Y.js provider](https://docs.yjs.dev/ecosystem/connection-provider) (e.g., `y-websocket`).

### Setup

```typescript
import { collab, collabServiceCtx } from "@milkdown/plugin-collab";
import { Doc } from "yjs";
import { WebsocketProvider } from "y-websocket";

const editor = await Editor.make().use(commonmark).use(collab).create();

const doc = new Doc();
const wsProvider = new WebsocketProvider("<WS_HOST>", "milkdown", doc);

editor.action((ctx) => {
  ctx.get(collabServiceCtx)
    .bindDoc(doc)
    .setAwareness(wsProvider.awareness)
    .connect();
});
```

### Connect/Disconnect manually

```typescript
editor.action((ctx) => {
  const collabService = ctx.get(collabServiceCtx);
  collabService.bindDoc(doc).setAwareness(wsProvider.awareness);

  connectBtn.onclick = () => { wsProvider.connect(); collabService.connect(); };
  disconnectBtn.onclick = () => { wsProvider.disconnect(); collabService.disconnect(); };
});
```

### Default template (apply once on first sync)

```typescript
wsProvider.once("synced", (isSynced) => {
  if (isSynced) {
    collabService.applyTemplate(template).connect();
  }
});
```

`applyTemplate` only applies if remote doc is empty. Custom check:

```typescript
collabService.applyTemplate(template, (remoteNode, templateNode) => {
  return true; // return true to apply
});
```

## Code Highlighting

Via `@milkdown/plugin-highlight`. Supports multiple parsers.

```bash
npm install @milkdown/plugin-highlight
```

### Shiki (VS Code themes)

```typescript
import { highlight, highlightPluginConfig } from "@milkdown/plugin-highlight";
import { createParser } from "@milkdown/plugin-highlight/shiki";

const parser = await createParser({
  theme: "github-light",
  langs: ["javascript", "typescript", "python"],
});

Editor.make()
  .config((ctx) => { ctx.set(highlightPluginConfig.key, { parser }); })
  .use(commonmark)
  .use(highlight)
  .create();
```

### Lowlight (highlight.js)

```typescript
import { createParser } from "@milkdown/plugin-highlight/lowlight";
import { common } from "lowlight";
const parser = createParser({ common });
```

### Refractor (Prism.js)

```typescript
import { createParser } from "@milkdown/plugin-highlight/refractor";
import { refractor } from "refractor";
const parser = createParser({ refractor });
```

### Sugar High (lightweight)

```typescript
import { createParser } from "@milkdown/plugin-highlight/sugar-high";
const parser = createParser();
```

## ProseMirror API

Access all ProseMirror packages via `@milkdown/kit/prose/*`:

| ProseMirror package | Import from |
|---|---|
| prosemirror-state | `@milkdown/kit/prose/state` |
| prosemirror-view | `@milkdown/kit/prose/view` |
| prosemirror-model | `@milkdown/kit/prose/model` |
| prosemirror-commands | `@milkdown/kit/prose/commands` |
| prosemirror-keymap | `@milkdown/kit/prose/keymap` |
| prosemirror-inputrules | `@milkdown/kit/prose/inputrules` |
| prosemirror-transform | `@milkdown/kit/prose/transform` |
| prosemirror-history | `@milkdown/kit/prose/history` |
| prosemirror-tables | `@milkdown/kit/prose/tables` |
| prosemirror-schema-list | `@milkdown/kit/prose/schema-list` |
| prosemirror-dropcursor | `@milkdown/kit/prose/dropcursor` |
| prosemirror-gapcursor | `@milkdown/kit/prose/gapcursor` |
| prosemirror-changeset | `@milkdown/kit/prose/changeset` |

Always import from `@milkdown/kit/prose/*` to ensure version alignment with Milkdown.

## Crepe Theming

### Available themes

Light: `frame`, `classic`, `nord`. Dark: `frame-dark`, `classic-dark`, `nord-dark`.

```typescript
import "@milkdown/crepe/theme/common/style.css"; // always required
import "@milkdown/crepe/theme/frame.css";         // choose one
```

### CSS Variables

Override on `.crepe .milkdown`:

```css
.crepe .milkdown {
  /* Background */
  --crepe-color-background: #fffdfb;
  --crepe-color-surface: #fff8f4;
  --crepe-color-surface-low: #fff1e5;

  /* Text */
  --crepe-color-on-background: #1f1b16;
  --crepe-color-on-surface: #201b13;
  --crepe-color-on-surface-variant: #4f4539;

  /* Accent */
  --crepe-color-primary: #805610;
  --crepe-color-secondary: #fbdebc;
  --crepe-color-on-secondary: #271904;

  /* UI */
  --crepe-color-outline: #817567;
  --crepe-color-inverse: #362f27;
  --crepe-color-on-inverse: #fcefe2;
  --crepe-color-inline-code: #ba1a1a;
  --crepe-color-error: #ba1a1a;
  --crepe-color-hover: #f9ecdf;
  --crepe-color-selected: #ede0d4;
  --crepe-color-inline-area: #e4d8cc;

  /* Typography */
  --crepe-font-title: Georgia, Cambria, "Times New Roman", Times, serif;
  --crepe-font-default: "Open Sans", Arial, Helvetica, sans-serif;
  --crepe-font-code: Fira Code, Menlo, Monaco, "Courier New", Courier, monospace;

  /* Shadows */
  --crepe-shadow-1: 0px 1px 3px 1px rgba(0,0,0,0.15), 0px 1px 2px 0px rgba(0,0,0,0.3);
  --crepe-shadow-2: 0px 2px 6px 2px rgba(0,0,0,0.15), 0px 1px 2px 0px rgba(0,0,0,0.3);
}
```

## Kit Styling

### Basic CSS

```css
.milkdown .editor {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}
.milkdown .editor .heading { font-weight: 600; }
.milkdown .editor .bullet-list { padding-left: 1.5rem; }
```

### Custom attributes (Tailwind-friendly)

```typescript
import { editorViewOptionsCtx } from "@milkdown/kit/core";
import { headingAttr, paragraphAttr } from "@milkdown/kit/preset/commonmark";

Editor.make().config((ctx) => {
  ctx.update(editorViewOptionsCtx, (prev) => ({
    ...prev,
    attributes: { class: "prose mx-auto outline-hidden", spellcheck: "false" },
  }));
  ctx.set(headingAttr.key, (node) => ({
    class: `heading-${node.attrs.level} font-bold`,
  }));
  ctx.set(paragraphAttr.key, () => ({
    class: "text-base leading-relaxed",
  }));
});
```
