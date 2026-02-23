# Angular Integration

No dedicated Angular package. Use `@milkdown/kit` (or `@milkdown/crepe`) directly with Angular's DOM references.

## Installation

```bash
npm install @milkdown/kit @milkdown/theme-nord
# For Crepe:
npm install @milkdown/crepe
```

## Basic Kit Setup

```typescript
// editor.component.ts
import {
  Component,
  ElementRef,
  ViewChild,
  OnInit,
  OnDestroy,
  Input,
  Output,
  EventEmitter,
} from "@angular/core";
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

@Component({
  selector: "app-milkdown-editor",
  template: `<div #editorRef></div>`,
})
export class MilkdownEditorComponent implements OnInit, OnDestroy {
  @ViewChild("editorRef") editorRef!: ElementRef;
  @Input() defaultValue: string = "";
  @Input() readonly: boolean = false;
  @Output() contentChange = new EventEmitter<string>();

  private editor!: Editor;

  async ngOnInit(): Promise<void> {
    this.editor = await Editor.make()
      .config(nord)
      .config((ctx) => {
        ctx.set(rootCtx, this.editorRef.nativeElement);
        ctx.set(defaultValueCtx, this.defaultValue);
        ctx.get(listenerCtx).markdownUpdated((ctx, markdown) => {
          this.contentChange.emit(markdown);
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
  }

  async ngOnDestroy(): Promise<void> {
    await this.editor?.destroy();
  }
}
```

```typescript
// editor.module.ts
import { NgModule } from "@angular/core";
import { MilkdownEditorComponent } from "./editor.component";

@NgModule({
  declarations: [MilkdownEditorComponent],
  exports: [MilkdownEditorComponent],
})
export class EditorModule {}
```

Usage in parent template:
```html
<app-milkdown-editor
  [defaultValue]="content"
  [readonly]="isReadonly"
  (contentChange)="onContentChange($event)"
></app-milkdown-editor>
```

## Crepe Setup

```typescript
import { Component, ElementRef, ViewChild, OnInit, OnDestroy, Input } from "@angular/core";
import { Crepe } from "@milkdown/crepe";
import "@milkdown/crepe/theme/common/style.css";
import "@milkdown/crepe/theme/frame.css";

@Component({
  selector: "app-crepe-editor",
  template: `<div #editorRef></div>`,
})
export class CrepeEditorComponent implements OnInit, OnDestroy {
  @ViewChild("editorRef") editorRef!: ElementRef;
  @Input() defaultValue: string = "";
  @Input() readonly: boolean = false;

  private crepe!: Crepe;

  async ngOnInit(): Promise<void> {
    this.crepe = new Crepe({
      root: this.editorRef.nativeElement,
      defaultValue: this.defaultValue,
    });
    await this.crepe.create();
    if (this.readonly) this.crepe.setReadonly(true);
  }

  getMarkdown(): string {
    return this.crepe.getMarkdown();
  }

  async ngOnDestroy(): Promise<void> {
    this.crepe?.destroy();
  }
}
```

## Programmatic Content Updates

Expose methods for parent components to call content APIs:

```typescript
import { replaceAll, getMarkdown, insert } from "@milkdown/kit/utils";

// In your component class
getContent(): string {
  return this.editor.action(getMarkdown());
}

setContent(markdown: string): void {
  this.editor.action(replaceAll(markdown));
}

insertContent(markdown: string): void {
  this.editor.action(insert(markdown));
}
```

## Dynamic Readonly Toggle

```typescript
private isReadonly = false;

setReadonly(readonly: boolean): void {
  // Crepe
  this.crepe.setReadonly(readonly);

  // Kit — requires recreating the editor or using a closure reference
  // Best approach: use a class property in the editable callback
  this.isReadonly = readonly;
  // If using ViewContainerRef, recreate editor; otherwise use a ref in the config closure:
  // editable: () => !this.isReadonly
}
```

## Angular CSS

Import Milkdown styles in `angular.json` under `styles`:

```json
"styles": [
  "src/styles.css",
  "node_modules/@milkdown/theme-nord/style.css"
]
```

Or in global `styles.css`:

```css
@import "@milkdown/theme-nord/style.css";
```

## Common Pitfalls

- **`ngAfterViewInit` vs `ngOnInit`**: Use `ngAfterViewInit` if `@ViewChild` is inside the template — it ensures the DOM is ready before creating the editor.
- **Change detection**: Milkdown updates happen outside Angular's zone. If UI doesn't update after events, call `this.cdr.detectChanges()` (inject `ChangeDetectorRef`).
- **AOT compilation**: CSS imports with bare specifiers may need `allowSyntheticDefaultImports: true` in `tsconfig.json`.
- **Multiple instances**: Each component instance creates its own editor. Destroy in `ngOnDestroy` to prevent memory leaks.
