# Utilities Reference

## Primitive

Base component for all Reka UI parts. Provides `as` and `asChild` props.

- `as` (string | Component, default `'div'`) — element type to render
- `asChild` (boolean, default false) — merge props/behavior onto immediate child instead

```vue
<script setup>
import { Primitive, type PrimitiveProps } from 'reka-ui'
const props = withDefaults(defineProps<PrimitiveProps>(), { as: 'span' })
</script>
<template>
  <Primitive v-bind="props"><slot /></Primitive>
</template>
```

## Slot

Merges attributes onto its immediate child (unlike native Vue slots which create scoped slots).

```vue
<script setup>
import { Slot } from 'reka-ui'
</script>
<template>
  <Slot id="reka-01"><button>Click</button></Slot>
  <!-- button receives id="reka-01" -->
</template>
```

## ConfigProvider

Global config wrapper. Props:

- `dir` ('ltr' | 'rtl') — text direction for all descendants
- `locale` (string) — locale for date/number formatting
- `useId` (function) — custom ID generator (for SSR hydration with Vue < 3.5)

```vue
<ConfigProvider dir="rtl" locale="ar"><slot /></ConfigProvider>
```

## VisuallyHidden

Hides content visually while keeping it accessible to screen readers.

```vue
<script setup>
import { VisuallyHidden } from 'reka-ui'
</script>
<template>
  <VisuallyHidden>Screen reader only text</VisuallyHidden>
  <!-- Or use asChild to hide an existing element -->
  <VisuallyHidden asChild><DialogTitle>Hidden title</DialogTitle></VisuallyHidden>
</template>
```

## Presence

Controls mount/unmount with animation awareness. Used internally; can be used directly for custom patterns.

```vue
<script setup>
import { Presence } from 'reka-ui'
</script>
<template>
  <Presence :present="show">
    <div>Animated content</div>
  </Presence>
</template>
```

## FocusScope

Traps focus within a container. Used internally by Dialog/AlertDialog.

- `trapped` (boolean) — enable focus trapping
- `loop` (boolean) — loop focus at boundaries

## RovingFocus

Manages arrow-key focus navigation within a group. Used internally by RadioGroup, Tabs, ToggleGroup.

## Composables

### useForwardPropsEmits

Forward all props and emits when wrapping Reka components:

```vue
<script setup>
import type { DialogContentProps, DialogContentEmits } from 'reka-ui'
import { DialogContent, useForwardPropsEmits } from 'reka-ui'

const props = defineProps<DialogContentProps>()
const emits = defineEmits<DialogContentEmits>()
const forwarded = useForwardPropsEmits(props, emits)
</script>
<template>
  <DialogContent v-bind="forwarded"><slot /></DialogContent>
</template>
```

### useForwardProps

Forward only props (no emits):

```ts
import { useForwardProps } from 'reka-ui'
const forwarded = useForwardProps(props)
```

### useForwardExpose

Expose internal component refs and methods:

```ts
import { useForwardExpose } from 'reka-ui'
const { forwardRef } = useForwardExpose()
```

### useEmitAsProps

Convert emit functions to prop callbacks:

```ts
import { useEmitAsProps } from 'reka-ui'
const emitsAsProps = useEmitAsProps(emits)
```

### useFilter

Locale-aware string filtering for Combobox/Listbox:

```ts
import { useFilter } from 'reka-ui'
const { contains, startsWith } = useFilter({ sensitivity: 'base' })
const filtered = items.filter(item => contains(item.name, searchTerm))
```

### useDateFormatter

Locale-aware date formatting:

```ts
import { useDateFormatter } from 'reka-ui'
const formatter = useDateFormatter('en-US')
```

### useId

Generate unique IDs (SSR-safe with ConfigProvider):

```ts
import { useId } from 'reka-ui'
const id = useId()
```

### useDirection / useLocale

Access direction or locale from nearest ConfigProvider:

```ts
import { useDirection, useLocale } from 'reka-ui'
const dir = useDirection()
const locale = useLocale()
```
