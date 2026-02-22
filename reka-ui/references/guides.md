# Guides Reference

Distilled Reka UI guides: styling, composition, animation, state, dates, i18n, SSR, virtualization, migration.

## Styling

Components ship unstyled. Style via classes, data-attribute selectors, or Tailwind.

### CSS classes

```vue
<AccordionItem class="AccordionItem" value="item-1" />
```

```css
.AccordionItem { border-bottom: 1px solid gainsboro; }
.AccordionItem[data-state="open"] { border-bottom-width: 2px; }
```

### Tailwind

```vue
<AccordionItem class="border data-[state=open]:border-b-2" value="item-1" />
```

### Scoped styles + teleported elements

Portaled content (dialogs, popovers, dropdowns) renders outside the component tree. Use `:deep()`:

```vue
<style scoped>
:deep(.DropdownMenuContent) { /* styles */ }
</style>
```

### Extending primitives

```vue
<script setup lang="ts">
import type { AccordionItemProps } from 'reka-ui'
import { AccordionItem } from 'reka-ui'

interface Props extends AccordionItemProps { foo: string }
defineProps<Props>()
</script>
<template>
  <AccordionItem v-bind="$props"><slot /></AccordionItem>
</template>
```

## Composition (asChild)

Set `asChild` to merge behavior onto the child element instead of rendering the default element:

```vue
<TooltipTrigger asChild>
  <a href="https://reka-ui.com/">Reka UI</a>
</TooltipTrigger>
```

Nest multiple primitives:

```vue
<TooltipTrigger asChild>
  <DialogTrigger asChild>
    <MyButton>Open dialog</MyButton>
  </DialogTrigger>
</TooltipTrigger>
```

When changing element types, ensure the replacement remains accessible and focusable.

## Animation

### CSS keyframes

Reka suspends unmount during exit animations. Target `data-state`:

```css
.DialogOverlay[data-state="open"] { animation: fadeIn 300ms ease-out; }
.DialogOverlay[data-state="closed"] { animation: fadeOut 300ms ease-in; }
```

### Vue Transition

Wrap components directly — no `forceMount` needed:

```vue
<Transition name="fade">
  <DialogOverlay />
</Transition>
```

### Motion Vue (recommended)

```vue
<script setup>
import { AnimatePresence, Motion } from 'motion-v'
</script>
<template>
  <AnimatePresence multiple>
    <DialogOverlay as-child>
      <Motion :initial="{ opacity: 0 }" :animate="{ opacity: 1 }" :exit="{ opacity: 0 }" />
    </DialogOverlay>
  </AnimatePresence>
</template>
```

### JS animation with forceMount

Use `forceMount` + conditional `v-if` based on animation state for libraries like `@vueuse/motion`.

## Controlled vs Uncontrolled State

**Uncontrolled** (component manages state): use `default-value`

```vue
<SwitchRoot default-value="true"><SwitchThumb /></SwitchRoot>
```

**Controlled** (parent manages state): use `v-model` or `:model-value` + `@update:model-value`

```vue
<SwitchRoot v-model="isActive"><SwitchThumb /></SwitchRoot>
```

Common mistakes:
- Using `:model-value` without `@update:model-value` → state won't update
- Using `model-value` instead of `default-value` for uncontrolled
- Computed without setter for v-model → use `computed({ get, set })`

## Dates

Calendar and date picker components use `@internationalized/date`:

```bash
npm install @internationalized/date
```

```ts
import { CalendarDate, CalendarDateTime, toZoned } from '@internationalized/date'

const date = new CalendarDate(2024, 1, 15)
const dateTime = new CalendarDateTime(2024, 1, 15, 9, 30)
```

Pass these to `v-model` on Calendar, DateField, DatePicker, RangeCalendar, DateRangePicker, DateRangeField.

## i18n / RTL

Wrap app in `ConfigProvider` with `dir`:

```vue
<script setup>
import { ConfigProvider } from 'reka-ui'
</script>
<template>
  <ConfigProvider dir="rtl"><slot /></ConfigProvider>
</template>
```

Dynamic direction with `@vueuse/core`:

```ts
import { useTextDirection } from '@vueuse/core'
const dir = useTextDirection() // reactive
```

For locale, use `ConfigProvider`'s `locale` prop or `vue-i18n`.

## Server-Side Rendering

Works with Nuxt out of the box. For Vue < 3.5, hydration IDs may mismatch. Fix:

```ts
// nuxt.config.ts
export default defineNuxtConfig({ modules: ['reka-ui/nuxt'] })
```

Provide custom `useId` via ConfigProvider for Vue < 3.5.

## Virtualization

Use `*Virtualizer` components for large lists (Combobox, Listbox, Tree):

```vue
<ComboboxViewport class="max-h-80 overflow-y-auto">
  <ComboboxVirtualizer v-slot="{ option }" :options="filtered" :estimate-size="25" :text-content="(opt) => opt.label">
    <ComboboxItem :value="option">{{ option.label }}</ComboboxItem>
  </ComboboxVirtualizer>
</ComboboxViewport>
```

Requirements:
1. Parent must have fixed height
2. Set `estimateSize` appropriately
3. Set `textContent` for typeahead accessibility
4. Filter items manually before passing to virtualizer

## Namespaced Components

Import via `reka-ui/namespaced` for grouped access:

```vue
<script setup>
import { Dialog } from 'reka-ui/namespaced'
</script>
<template>
  <Dialog.Root>
    <Dialog.Trigger>Open</Dialog.Trigger>
    <Dialog.Portal><Dialog.Content>...</Dialog.Content></Dialog.Portal>
  </Dialog.Root>
</template>
```

## Inject Context (Advanced)

Access internal component state via exported `injectContext` functions. Use cautiously — API may change.

```vue
<script setup>
import { injectAccordionRootContext, injectAccordionItemContext } from 'reka-ui'

const root = injectAccordionRootContext()
const item = injectAccordionItemContext()
</script>
```

## Migration (v1 → v2)

Key breaking changes from Radix Vue v1 / Reka UI v1:
- Package renamed from `radix-vue` to `reka-ui`
- Controlled state uses `v-model` instead of old prop names
- SelectValue no longer auto-renders ItemText content
- Popover aria attributes updated
- Arrow polygon implementation improved
- Calendar `step` prop removed
- `ComboboxEmpty` removed
- Popper props renamed
- Form components: hidden input moved inside root node
