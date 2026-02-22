# Components A–L

## Accordion

Vertically stacked collapsible sections. `type="single"` or `type="multiple"`.

```vue
<script setup>
import { AccordionRoot, AccordionItem, AccordionHeader, AccordionTrigger, AccordionContent } from 'reka-ui'
</script>
<template>
  <AccordionRoot type="single" default-value="item-1" collapsible>
    <AccordionItem value="item-1">
      <AccordionHeader><AccordionTrigger /></AccordionHeader>
      <AccordionContent />
    </AccordionItem>
  </AccordionRoot>
</template>
```

- Data: `[data-state]` open/closed, `[data-orientation]` vertical/horizontal, `[data-disabled]`
- CSS vars: `--reka-accordion-content-width`, `--reka-accordion-content-height` (use for height animation)
- Props: `unmount-on-hide` (default true) — set false to keep content in DOM for SEO/search

## AlertDialog

Modal requiring user acknowledgement. Like Dialog but focus is trapped on Cancel/Action.

```vue
<script setup>
import { AlertDialogRoot, AlertDialogTrigger, AlertDialogPortal, AlertDialogOverlay, AlertDialogContent, AlertDialogTitle, AlertDialogDescription, AlertDialogAction, AlertDialogCancel } from 'reka-ui'
</script>
<template>
  <AlertDialogRoot>
    <AlertDialogTrigger />
    <AlertDialogPortal>
      <AlertDialogOverlay />
      <AlertDialogContent>
        <AlertDialogTitle /><AlertDialogDescription />
        <AlertDialogCancel /><AlertDialogAction />
      </AlertDialogContent>
    </AlertDialogPortal>
  </AlertDialogRoot>
</template>
```

- Data: `[data-state]` open/closed

## AspectRatio

```vue
<script setup>
import { AspectRatio } from 'reka-ui'
</script>
<template>
  <AspectRatio :ratio="16 / 9"><img src="..." /></AspectRatio>
</template>
```

## Avatar

```vue
<script setup>
import { AvatarRoot, AvatarImage, AvatarFallback } from 'reka-ui'
</script>
<template>
  <AvatarRoot>
    <AvatarImage src="..." alt="User" />
    <AvatarFallback delay-ms="600">AB</AvatarFallback>
  </AvatarRoot>
</template>
```

## Calendar

Standalone date selection. `v-model` accepts `DateValue` from `@internationalized/date`.

```vue
<script setup>
import { CalendarRoot, CalendarHeader, CalendarHeading, CalendarGrid, CalendarCell, CalendarHeadCell, CalendarGridHead, CalendarGridBody, CalendarGridRow, CalendarCellTrigger, CalendarPrev, CalendarNext } from 'reka-ui'
</script>
<template>
  <CalendarRoot v-model="date" locale="en">
    <CalendarHeader>
      <CalendarPrev /><CalendarHeading /><CalendarNext />
    </CalendarHeader>
    <CalendarGrid>
      <CalendarGridHead><CalendarGridRow><CalendarHeadCell /></CalendarGridRow></CalendarGridHead>
      <CalendarGridBody><CalendarGridRow><CalendarCell><CalendarCellTrigger /></CalendarCell></CalendarGridRow></CalendarGridBody>
    </CalendarGrid>
  </CalendarRoot>
</template>
```

- Data on CellTrigger: `[data-selected]`, `[data-disabled]`, `[data-today]`, `[data-outside-month]`

## Checkbox

```vue
<script setup>
import { CheckboxRoot, CheckboxIndicator } from 'reka-ui'
</script>
<template>
  <CheckboxRoot v-model="checked">
    <CheckboxIndicator />
  </CheckboxRoot>
</template>
```

- Data: `[data-state]` checked/unchecked/indeterminate, `[data-disabled]`

## Collapsible

Single collapsible section.

```vue
<script setup>
import { CollapsibleRoot, CollapsibleTrigger, CollapsibleContent } from 'reka-ui'
</script>
<template>
  <CollapsibleRoot v-model:open="open">
    <CollapsibleTrigger />
    <CollapsibleContent />
  </CollapsibleRoot>
</template>
```

- Data: `[data-state]` open/closed, `[data-disabled]`
- CSS vars: `--reka-collapsible-content-width`, `--reka-collapsible-content-height`

## Combobox

Searchable select. Supports objects, multi-select, custom filtering, and virtualization.

```vue
<script setup>
import { ComboboxRoot, ComboboxAnchor, ComboboxInput, ComboboxTrigger, ComboboxCancel, ComboboxPortal, ComboboxContent, ComboboxViewport, ComboboxItem, ComboboxItemIndicator, ComboboxGroup, ComboboxLabel, ComboboxSeparator } from 'reka-ui'
</script>
<template>
  <ComboboxRoot v-model="value">
    <ComboboxAnchor>
      <ComboboxInput /><ComboboxTrigger /><ComboboxCancel />
    </ComboboxAnchor>
    <ComboboxPortal>
      <ComboboxContent>
        <ComboboxViewport>
          <ComboboxItem v-for="item in items" :value="item">
            {{ item.name }}<ComboboxItemIndicator />
          </ComboboxItem>
        </ComboboxViewport>
      </ComboboxContent>
    </ComboboxPortal>
  </ComboboxRoot>
</template>
```

- Objects: `:display-value="(v) => v.name"` on `ComboboxInput`
- Multi-select: pass array to `v-model` + `multiple` prop on Root
- Custom filter: `ignore-filter` on Root + `useFilter()` composable + pass filtered items
- Virtualization: `ComboboxVirtualizer` inside `ComboboxViewport` — filter items manually first
- Prevent close on select: `@select.prevent` on `ComboboxItem`
- Data on Item: `[data-state]` checked/unchecked, `[data-highlighted]`, `[data-disabled]`

## ContextMenu

Right-click triggered menu. Same sub-part pattern as DropdownMenu.

```vue
<script setup>
import { ContextMenuRoot, ContextMenuTrigger, ContextMenuPortal, ContextMenuContent, ContextMenuItem, ContextMenuSeparator, ContextMenuSub, ContextMenuSubTrigger, ContextMenuSubContent, ContextMenuCheckboxItem, ContextMenuItemIndicator, ContextMenuRadioGroup, ContextMenuRadioItem } from 'reka-ui'
</script>
<template>
  <ContextMenuRoot>
    <ContextMenuTrigger>Right click here</ContextMenuTrigger>
    <ContextMenuPortal>
      <ContextMenuContent>
        <ContextMenuItem>Cut</ContextMenuItem>
        <ContextMenuSeparator />
        <ContextMenuCheckboxItem v-model="checked"><ContextMenuItemIndicator />Toggle</ContextMenuCheckboxItem>
        <ContextMenuSub>
          <ContextMenuSubTrigger />
          <ContextMenuPortal><ContextMenuSubContent /></ContextMenuPortal>
        </ContextMenuSub>
      </ContextMenuContent>
    </ContextMenuPortal>
  </ContextMenuRoot>
</template>
```

## DateField

Segmented date input. `v-model` accepts `DateValue` from `@internationalized/date`.

```vue
<script setup>
import { DateFieldRoot, DateFieldInput } from 'reka-ui'
</script>
<template>
  <DateFieldRoot v-model="date">
    <DateFieldInput v-slot="{ segments }">
      <template v-for="segment in segments" :key="segment.type">
        <span v-if="segment.type === 'literal'">{{ segment.value }}</span>
        <input v-else v-model="segment.value" />
      </template>
    </DateFieldInput>
  </DateFieldRoot>
</template>
```

## DatePicker

Combined DateField + Calendar popup.

```vue
<script setup>
import { DatePickerRoot, DatePickerField, DatePickerInput, DatePickerTrigger, DatePickerContent, DatePickerCalendar, DatePickerHeader, DatePickerHeading, DatePickerGrid, DatePickerCell, DatePickerCellTrigger, DatePickerGridHead, DatePickerGridBody, DatePickerGridRow, DatePickerHeadCell, DatePickerPrev, DatePickerNext } from 'reka-ui'
</script>
<template>
  <DatePickerRoot v-model="date">
    <DatePickerField><DatePickerInput /><DatePickerTrigger /></DatePickerField>
    <DatePickerContent>
      <DatePickerCalendar>
        <DatePickerHeader><DatePickerPrev /><DatePickerHeading /><DatePickerNext /></DatePickerHeader>
        <DatePickerGrid><!-- grid rows same as Calendar --></DatePickerGrid>
      </DatePickerCalendar>
    </DatePickerContent>
  </DatePickerRoot>
</template>
```

## DateRangeField

Like DateField but for start/end range. `v-model` is `{ start, end }` DateValue.

## DateRangePicker

Like DatePicker but for date ranges. Uses `RangeCalendar` internally.

## Dialog

Modal window. Supports nested dialogs.

```vue
<script setup>
import { DialogRoot, DialogTrigger, DialogPortal, DialogOverlay, DialogContent, DialogTitle, DialogDescription, DialogClose } from 'reka-ui'
</script>
<template>
  <DialogRoot v-model:open="open">
    <DialogTrigger>Open</DialogTrigger>
    <DialogPortal>
      <DialogOverlay />
      <DialogContent>
        <DialogTitle>Title</DialogTitle>
        <DialogDescription>Desc</DialogDescription>
        <DialogClose />
      </DialogContent>
    </DialogPortal>
  </DialogRoot>
</template>
```

- Data: `[data-state]` open/closed
- Programmatic close: `v-slot="{ close }"` on `DialogRoot`
- Scrollable overlay: nest `DialogContent` inside `DialogOverlay` with `overflow-y: auto`
- Prevent close on scrollbar click: `@pointer-down-outside` handler on `DialogContent`

## DropdownMenu

Click-triggered menu with items, checkboxes, radio groups, and submenus.

```vue
<script setup>
import { DropdownMenuRoot, DropdownMenuTrigger, DropdownMenuPortal, DropdownMenuContent, DropdownMenuItem, DropdownMenuSeparator, DropdownMenuCheckboxItem, DropdownMenuItemIndicator, DropdownMenuRadioGroup, DropdownMenuRadioItem, DropdownMenuSub, DropdownMenuSubTrigger, DropdownMenuSubContent, DropdownMenuLabel, DropdownMenuGroup } from 'reka-ui'
</script>
<template>
  <DropdownMenuRoot>
    <DropdownMenuTrigger />
    <DropdownMenuPortal>
      <DropdownMenuContent :side-offset="5">
        <DropdownMenuItem>Item</DropdownMenuItem>
        <DropdownMenuSeparator />
        <DropdownMenuSub>
          <DropdownMenuSubTrigger>More</DropdownMenuSubTrigger>
          <DropdownMenuPortal><DropdownMenuSubContent /></DropdownMenuPortal>
        </DropdownMenuSub>
      </DropdownMenuContent>
    </DropdownMenuPortal>
  </DropdownMenuRoot>
</template>
```

- Data: `[data-state]` open/closed, `[data-highlighted]`, `[data-disabled]`, `[data-side]`, `[data-align]`

## Editable

Inline text with edit/preview modes.

```vue
<script setup>
import { EditableRoot, EditableArea, EditableInput, EditablePreview, EditableSubmitTrigger, EditableCancelTrigger, EditableEditTrigger } from 'reka-ui'
</script>
<template>
  <EditableRoot v-model="value">
    <EditableArea><EditableInput /><EditablePreview /></EditableArea>
    <EditableEditTrigger /><EditableSubmitTrigger /><EditableCancelTrigger />
  </EditableRoot>
</template>
```

## HoverCard

Popup card on hover with configurable open/close delays.

```vue
<script setup>
import { HoverCardRoot, HoverCardTrigger, HoverCardPortal, HoverCardContent, HoverCardArrow } from 'reka-ui'
</script>
<template>
  <HoverCardRoot :open-delay="200" :close-delay="300">
    <HoverCardTrigger as-child><a href="#">Trigger</a></HoverCardTrigger>
    <HoverCardPortal>
      <HoverCardContent :side-offset="5"><HoverCardArrow /></HoverCardContent>
    </HoverCardPortal>
  </HoverCardRoot>
</template>
```

## Label

```vue
<script setup>
import { Label } from 'reka-ui'
</script>
<template>
  <Label for="input-id">Label text</Label>
</template>
```

## Listbox

List selection. Supports single/multiple selection, objects, and virtualization.

```vue
<script setup>
import { ListboxRoot, ListboxContent, ListboxItem, ListboxItemIndicator, ListboxFilter, ListboxVirtualizer, ListboxGroup, ListboxGroupLabel } from 'reka-ui'
</script>
<template>
  <ListboxRoot v-model="selected">
    <ListboxContent>
      <ListboxItem v-for="item in items" :value="item">
        {{ item.name }}<ListboxItemIndicator />
      </ListboxItem>
    </ListboxContent>
  </ListboxRoot>
</template>
```
