# Kit & Plugins

## Plugin Lifecycle

Every plugin has three phases:

```typescript
const myPlugin: MilkdownPlugin = (ctx) => {
  // 1. Prepare: register timers, inject slices
  ctx.inject(mySlice, defaultValue);
  ctx.record(myTimer);

  return async () => {
    // 2. Run: wait for deps, do work, mark complete
    await ctx.wait(RequiredTimer);
    ctx.done(myTimer);

    return async () => {
      // 3. Cleanup: remove slices, clear timers
      ctx.remove(mySlice);
      ctx.clearTimer(myTimer);
    };
  };
};
```

## Context System (Ctx)

Shared state container across all plugins via slices:

```typescript
import { createSlice } from "@milkdown/kit/ctx";

const counterCtx = createSlice(0, "counter");

const plugin: MilkdownPlugin = (ctx) => {
  ctx.inject(counterCtx);
  return () => {
    ctx.get(counterCtx);              // read
    ctx.set(counterCtx, 1);           // write
    ctx.update(counterCtx, n => n+1); // update
    return () => ctx.remove(counterCtx);
  };
};
```

## Timers

Coordinate plugin loading order:

```typescript
import { createTimer, editorStateTimerCtx, defaultValueCtx } from "@milkdown/kit/core";

const RemoteTimer = createTimer("RemoteTimer");

const remotePlugin: MilkdownPlugin = (ctx) => {
  ctx.record(RemoteTimer);
  return async () => {
    ctx.update(editorStateTimerCtx, (t) => t.concat(RemoteTimer));
    const md = await fetchMarkdownAPI();
    ctx.set(defaultValueCtx, md);
    ctx.done(RemoteTimer);

    return async () => {
      ctx.clearTimer(RemoteTimer);
    };
  };
};
```

`ctx.waitTimers(timerSlice)` is a convenience sugar for waiting all timers in a slice at once:

```typescript
import { createSlice, Timer, SchemaReady } from "@milkdown/kit/core";

const examplePluginTimersCtx = createSlice<Timer[]>([], "example-timer");

const examplePlugin: MilkdownPlugin = (ctx) => {
  ctx.inject(examplePluginTimersCtx, [SchemaReady]);
  return async () => {
    // Wait all timers listed in the slice
    await ctx.waitTimers(examplePluginTimersCtx);
    // do work after all deps are ready
  };
};
```

## Composable Plugin Helpers

All from `@milkdown/kit/utils`:

### Schema (`$node` / `$markSchema`)

The `$node` factory receives `(ctx)` as its argument, giving access to the context inside node definitions:

```typescript
import { $node, $nodeAttr } from "@milkdown/kit/utils";

// Optional: define an attr plugin so the node's DOM attributes are configurable at runtime
const blockquoteAttr = $nodeAttr("blockquote");

const blockquote = $node("blockquote", (ctx) => ({
  content: "block+",
  group: "block",
  defining: true,
  parseDOM: [{ tag: "blockquote" }],
  toDOM: (node) => ["blockquote", ctx.get(blockquoteAttr.key)(node), 0],
  parseMarkdown: {
    match: ({ type }) => type === "blockquote",
    runner: (state, node, type) => {
      state.openNode(type).next(node.children).closeNode();
    },
  },
  toMarkdown: {
    match: (node) => node.type.name === "blockquote",
    runner: (state, node) => {
      state.openNode("blockquote").next(node.content).closeNode();
    },
  },
}));
```

### Input Rule (`$inputRule`)

```typescript
import { wrappingInputRule } from "@milkdown/kit/prose/inputrules";
import { $inputRule } from "@milkdown/kit/utils";

const wrapInBlockquoteInputRule = $inputRule(() =>
  wrappingInputRule(/^\s*>\s$/, blockquoteSchema.type()),
);
```

### Command (`$command`)

```typescript
import { wrapIn } from "@milkdown/kit/prose/commands";
import { $command, callCommand } from "@milkdown/kit/utils";

const wrapInBlockquoteCommand = $command(
  "WrapInBlockquote",
  () => () => wrapIn(blockquoteSchema.type()),
);

// Usage
editor.action(callCommand(wrapInBlockquoteCommand.key));
```

Command with argument:

```typescript
const wrapInHeadingCommand = $command(
  "WrapInHeading",
  (ctx) => (level = 1) => setBlockType(headingSchema.type(ctx), { level }),
);
editor.action(callCommand(wrapInHeadingCommand.key, 2));
```

### Keymap (`$useKeymap`)

```typescript
import { commandsCtx } from "@milkdown/kit/core";
import { $useKeymap } from "@milkdown/kit/utils";

const blockquoteKeymap = $useKeymap("blockquoteKeymap", {
  WrapInBlockquote: {
    shortcuts: "Mod-Shift-b",
    command: (ctx) => {
      const commands = ctx.get(commandsCtx);
      return () => commands.call(wrapInBlockquoteCommand.key);
    },
  },
});
```

Shortcut priority (default 50, range 1-100): higher runs first. If a command returns `false`, next priority is tried.

### Remark Plugin (`$remark`)

```typescript
import directive from "remark-directive";
import { $remark } from "@milkdown/kit/utils";

const remarkDirective = $remark("remarkDirective", () => directive);
```

## Tooltip Plugin

```typescript
import { TooltipProvider, tooltipFactory } from "@milkdown/plugin-tooltip";

const [myTooltipSpec, myTooltipPlugin] = tooltipFactory("my");

// Provider manages positioning via floating-ui
const provider = new TooltipProvider({
  content: domElement,
  shouldShow: (view) => !!view.state.selection.content().size,
});

// Wire to editor
const tooltipConfig = (ctx) => {
  ctx.set(myTooltipSpec.key, {
    view: () => ({ update: provider.update, destroy: provider.destroy }),
  });
};

Editor.make().config(tooltipConfig).use(commonmark).use(myTooltipPlugin).create();
```

Framework integration: render React/Vue component, pass root DOM element to `TooltipProvider`.

## Slash Plugin

```typescript
import { SlashProvider, slashFactory } from "@milkdown/plugin-slash";

const [mySlashSpec, mySlashPlugin] = slashFactory("my");

const provider = new SlashProvider({
  content: menuElement,
  shouldShow: (view) => provider.getContent(view)?.endsWith("/") ?? false,
});
```

`provider.getContent(view)` returns text before caret — useful for filtering commands.

## Block Plugin

```typescript
import { blockSpec, blockPlugin } from "@milkdown/plugin-block";
import { BlockProvider } from "@milkdown/plugin-block/block-provider";

const provider = new BlockProvider({
  ctx,
  content: handleElement,
  getOffset: () => 8,
});
```

`BlockProvider` tracks active block (`node`, `pos`, `el`). Internal `BlockService` handles drag events automatically.

## Custom Syntax Example: Iframe

Five components needed: remark plugin, schema, parser, serializer, input rule.

```typescript
import { $remark, $node, $inputRule } from "@milkdown/kit/utils";
import directive from "remark-directive";
import { InputRule } from "@milkdown/kit/prose";

const remarkDirective = $remark("remarkDirective", () => directive);

const iframeNode = $node("iframe", () => ({
  group: "block",
  atom: true,
  isolating: true,
  marks: "",
  attrs: { src: { default: null } },
  parseDOM: [{ tag: "iframe", getAttrs: (dom) => ({ src: dom.getAttribute("src") }) }],
  toDOM: (node) => ["iframe", { ...node.attrs, contenteditable: false }, 0],
  parseMarkdown: {
    match: (node) => node.type === "leafDirective" && node.name === "iframe",
    runner: (state, node, type) => {
      state.addNode(type, { src: node.attributes.src });
    },
  },
  toMarkdown: {
    match: (node) => node.type.name === "iframe",
    runner: (state, node) => {
      state.addNode("leafDirective", undefined, undefined, {
        name: "iframe",
        attributes: { src: node.attrs.src },
      });
    },
  },
}));

const iframeInputRule = $inputRule(() =>
  new InputRule(/::iframe\{src="(?<src>[^"]+)?"?\}/, (state, match, start, end) => {
    const [, src = ""] = match;
    return state.tr.replaceWith(start - 1, end, iframeNode.type().create({ src }));
  }),
);

// Register
Editor.make().use([remarkDirective, iframeNode, iframeInputRule]).use(commonmark).create();
```

Markdown syntax: `::iframe{src="https://example.com"}`

## Custom Mark Example: Marker

```typescript
import { $markSchema, $inputRule } from "@milkdown/kit/utils";

const markSchema = $markSchema("mark", () => ({
  attrs: { color: { default: "#ffff00" } },
  parseDOM: [{ tag: "mark", getAttrs: (el) => ({ color: el.style.backgroundColor }) }],
  toDOM: (mark) => ["mark", { style: `background-color: ${mark.attrs.color}` }],
  parseMarkdown: {
    match: (node) => node.type === "mark",
    runner: (state, node, markType) => {
      state.openMark(markType, { color: node.data?.color });
      state.next(node.children);
      state.closeMark(markType);
    },
  },
  toMarkdown: {
    match: (node) => node.type.name === "mark",
    runner: (state, mark) => {
      state.withMark(mark, "mark", undefined, { data: { color: mark.attrs.color } });
    },
  },
}));
```

Markdown syntax: `==text==` or `=={#EE4B2B}colored text==` (requires custom remark plugin).
