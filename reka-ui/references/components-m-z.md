# Components M–Z

## Menubar

Horizontal menu bar (File, Edit, View pattern).

```vue
<script setup>
import { MenubarRoot, MenubarMenu, MenubarTrigger, MenubarPortal, MenubarContent, MenubarItem, MenubarSeparator, MenubarSub, MenubarSubTrigger, MenubarSubContent, MenubarCheckboxItem, MenubarItemIndicator, MenubarRadioGroup, MenubarRadioItem } from 'reka-ui'
</script>
<template>
  <MenubarRoot>
    <MenubarMenu>
      <MenubarTrigger>File</MenubarTrigger>
      <MenubarPortal><MenubarContent><MenubarItem>New</MenubarItem></MenubarContent></MenubarPortal>
    </MenubarMenu>
  </MenubarRoot>
</template>
```

## NavigationMenu

Site navigation with content panels and viewport.

```vue
<script setup>
import { NavigationMenuRoot, NavigationMenuList, NavigationMenuItem, NavigationMenuTrigger, NavigationMenuContent, NavigationMenuLink, NavigationMenuViewport, NavigationMenuIndicator } from 'reka-ui'
</script>
<template>
  <NavigationMenuRoot>
    <NavigationMenuList>
      <NavigationMenuItem>
        <NavigationMenuTrigger>Products</NavigationMenuTrigger>
        <NavigationMenuContent>...</NavigationMenuContent>
      </NavigationMenuItem>
      <NavigationMenuItem>
        <NavigationMenuLink href="/">Home</NavigationMenuLink>
      </NavigationMenuItem>
    </NavigationMenuList>
    <NavigationMenuViewport />
  </NavigationMenuRoot>
</template>
```

## NumberField

Numeric input with increment/decrement controls.

```vue
<script setup>
import { NumberFieldRoot, NumberFieldInput, NumberFieldIncrement, NumberFieldDecrement } from 'reka-ui'
</script>
<template>
  <NumberFieldRoot v-model="value" :min="0" :max="100">
    <NumberFieldDecrement /><NumberFieldInput /><NumberFieldIncrement />
  </NumberFieldRoot>
</template>
```

## Pagination

```vue
<script setup>
import { PaginationRoot, PaginationList, PaginationListItem, PaginationPrev, PaginationNext, PaginationFirst, PaginationLast, PaginationEllipsis } from 'reka-ui'
</script>
<template>
  <PaginationRoot v-model:page="page" :total="100" :sibling-count="1">
    <PaginationList v-slot="{ items }">
      <PaginationFirst /><PaginationPrev />
      <template v-for="item in items">
        <PaginationListItem v-if="item.type === 'page'" :value="item.value" />
        <PaginationEllipsis v-else />
      </template>
      <PaginationNext /><PaginationLast />
    </PaginationList>
  </PaginationRoot>
</template>
```

## PinInput

OTP/PIN input with individual character fields.

```vue
<script setup>
import { PinInputRoot, PinInputInput } from 'reka-ui'
</script>
<template>
  <PinInputRoot v-model="value" placeholder="○">
    <PinInputInput v-for="(_, i) in 6" :key="i" :index="i" />
  </PinInputRoot>
</template>
```

## Popover

Popup positioned relative to trigger.

```vue
<script setup>
import { PopoverRoot, PopoverTrigger, PopoverPortal, PopoverContent, PopoverClose, PopoverArrow } from 'reka-ui'
</script>
<template>
  <PopoverRoot>
    <PopoverTrigger>Open</PopoverTrigger>
    <PopoverPortal>
      <PopoverContent :side-offset="5">
        Content<PopoverClose /><PopoverArrow />
      </PopoverContent>
    </PopoverPortal>
  </PopoverRoot>
</template>
```

- Data: `[data-state]` open/closed, `[data-side]`, `[data-align]`

## Progress

```vue
<script setup>
import { ProgressRoot, ProgressIndicator } from 'reka-ui'
</script>
<template>
  <ProgressRoot v-model="progress" :max="100">
    <ProgressIndicator :style="{ width: `${progress}%` }" />
  </ProgressRoot>
</template>
```

## RadioGroup

```vue
<script setup>
import { RadioGroupRoot, RadioGroupItem, RadioGroupIndicator } from 'reka-ui'
</script>
<template>
  <RadioGroupRoot v-model="value">
    <RadioGroupItem value="a"><RadioGroupIndicator /></RadioGroupItem>
    <RadioGroupItem value="b"><RadioGroupIndicator /></RadioGroupItem>
  </RadioGroupRoot>
</template>
```

- Data: `[data-state]` checked/unchecked, `[data-disabled]`

## RangeCalendar

Like Calendar but for date ranges. `v-model` accepts `{ start, end }` DateValue.

## ScrollArea

Custom scrollbar overlay.

```vue
<script setup>
import { ScrollAreaRoot, ScrollAreaViewport, ScrollAreaScrollbar, ScrollAreaThumb, ScrollAreaCorner } from 'reka-ui'
</script>
<template>
  <ScrollAreaRoot>
    <ScrollAreaViewport><!-- scrollable content --></ScrollAreaViewport>
    <ScrollAreaScrollbar orientation="vertical"><ScrollAreaThumb /></ScrollAreaScrollbar>
    <ScrollAreaCorner />
  </ScrollAreaRoot>
</template>
```

## Select

Dropdown select. Two positioning modes: default (macOS-style) or `position="popper"`.

```vue
<script setup>
import { SelectRoot, SelectTrigger, SelectValue, SelectIcon, SelectPortal, SelectContent, SelectViewport, SelectItem, SelectItemText, SelectItemIndicator, SelectScrollUpButton, SelectScrollDownButton } from 'reka-ui'
</script>
<template>
  <SelectRoot v-model="value">
    <SelectTrigger><SelectValue placeholder="Pick..." /><SelectIcon /></SelectTrigger>
    <SelectPortal>
      <SelectContent position="popper" :side-offset="5">
        <SelectScrollUpButton />
        <SelectViewport>
          <SelectItem value="a"><SelectItemText>A</SelectItemText><SelectItemIndicator /></SelectItem>
        </SelectViewport>
        <SelectScrollDownButton />
      </SelectContent>
    </SelectPortal>
  </SelectRoot>
</template>
```

- Match trigger width: `width: var(--reka-select-trigger-width)` on Content
- Constrain height: `max-height: var(--reka-select-content-available-height)` on Content
- Data on Item: `[data-state]` checked/unchecked, `[data-highlighted]`, `[data-disabled]`

## Separator

```vue
<script setup>
import { Separator } from 'reka-ui'
</script>
<template>
  <Separator orientation="horizontal" />
</template>
```

## Slider

Range slider. Supports multiple thumbs by passing an array to `v-model`.

```vue
<script setup>
import { SliderRoot, SliderTrack, SliderRange, SliderThumb } from 'reka-ui'
</script>
<template>
  <SliderRoot v-model="value" :max="100" :step="1">
    <SliderTrack><SliderRange /></SliderTrack>
    <SliderThumb />
  </SliderRoot>
</template>
```

## Splitter

Resizable panel layout.

```vue
<script setup>
import { SplitterGroup, SplitterPanel, SplitterResizeHandle } from 'reka-ui'
</script>
<template>
  <SplitterGroup direction="horizontal">
    <SplitterPanel :default-size="50">Left</SplitterPanel>
    <SplitterResizeHandle />
    <SplitterPanel :default-size="50">Right</SplitterPanel>
  </SplitterGroup>
</template>
```

## Stepper

Step-by-step wizard.

```vue
<script setup>
import { StepperRoot, StepperList, StepperItem, StepperTrigger, StepperContent, StepperSeparator } from 'reka-ui'
</script>
<template>
  <StepperRoot v-model="step">
    <StepperList>
      <StepperItem :step="1"><StepperTrigger /><StepperSeparator /></StepperItem>
      <StepperItem :step="2"><StepperTrigger /></StepperItem>
    </StepperList>
    <StepperContent :step="1">Step 1 content</StepperContent>
    <StepperContent :step="2">Step 2 content</StepperContent>
  </StepperRoot>
</template>
```

## Switch

```vue
<script setup>
import { SwitchRoot, SwitchThumb } from 'reka-ui'
</script>
<template>
  <SwitchRoot v-model="active"><SwitchThumb /></SwitchRoot>
</template>
```

- Data: `[data-state]` checked/unchecked, `[data-disabled]`

## Tabs

```vue
<script setup>
import { TabsRoot, TabsList, TabsTrigger, TabsContent, TabsIndicator } from 'reka-ui'
</script>
<template>
  <TabsRoot default-value="tab1">
    <TabsList>
      <TabsIndicator />
      <TabsTrigger value="tab1">Tab 1</TabsTrigger>
      <TabsTrigger value="tab2">Tab 2</TabsTrigger>
    </TabsList>
    <TabsContent value="tab1">Content 1</TabsContent>
    <TabsContent value="tab2">Content 2</TabsContent>
  </TabsRoot>
</template>
```

- Data: `[data-state]` active/inactive, `[data-orientation]`
- CSS vars: `--reka-tabs-indicator-size`, `--reka-tabs-indicator-position`

## TagsInput

Tag/chip input.

```vue
<script setup>
import { TagsInputRoot, TagsInputInput, TagsInputItem, TagsInputItemText, TagsInputItemDelete, TagsInputClear } from 'reka-ui'
</script>
<template>
  <TagsInputRoot v-model="tags">
    <TagsInputItem v-for="tag in tags" :key="tag" :value="tag">
      <TagsInputItemText /><TagsInputItemDelete />
    </TagsInputItem>
    <TagsInputInput placeholder="Add tag..." />
    <TagsInputClear />
  </TagsInputRoot>
</template>
```

## TimeField

Segmented time input (hours, minutes, seconds).

```vue
<script setup>
import { TimeFieldRoot, TimeFieldInput } from 'reka-ui'
</script>
<template>
  <TimeFieldRoot v-model="time" granularity="second">
    <TimeFieldInput v-slot="{ segments }">
      <template v-for="segment in segments" :key="segment.type">{{ segment.value }}</template>
    </TimeFieldInput>
  </TimeFieldRoot>
</template>
```

## Toast

Temporary notification. Requires `ToastProvider` wrapping the app.

```vue
<script setup>
import { ToastProvider, ToastRoot, ToastTitle, ToastDescription, ToastAction, ToastClose, ToastViewport } from 'reka-ui'
</script>
<template>
  <ToastProvider>
    <ToastRoot v-model:open="open" :duration="5000">
      <ToastTitle>Title</ToastTitle>
      <ToastDescription>Message</ToastDescription>
      <ToastAction alt-text="Undo">Undo</ToastAction>
      <ToastClose />
    </ToastRoot>
    <ToastViewport />
  </ToastProvider>
</template>
```

- Data: `[data-state]` open/closed, `[data-swipe]` start/move/cancel/end
- CSS vars: `--reka-toast-swipe-move-x/y`, `--reka-toast-swipe-end-x/y`
- `type="foreground"` for user-action toasts, `type="background"` for background tasks
- Custom hotkey: `<ToastViewport :hotkey="['altKey', 'KeyT']" />`

## Toggle

```vue
<script setup>
import { ToggleRoot } from 'reka-ui'
</script>
<template>
  <ToggleRoot v-model:pressed="pressed">Bold</ToggleRoot>
</template>
```

- Data: `[data-state]` on/off

## ToggleGroup

Group of toggles. `type="single"` or `type="multiple"`.

```vue
<script setup>
import { ToggleGroupRoot, ToggleGroupItem } from 'reka-ui'
</script>
<template>
  <ToggleGroupRoot type="single" v-model="value">
    <ToggleGroupItem value="a">A</ToggleGroupItem>
    <ToggleGroupItem value="b">B</ToggleGroupItem>
  </ToggleGroupRoot>
</template>
```

## Toolbar

Container for grouped controls (buttons, toggles, separators, links).

```vue
<script setup>
import { ToolbarRoot, ToolbarButton, ToolbarSeparator, ToolbarLink, ToolbarToggleGroup, ToolbarToggleItem } from 'reka-ui'
</script>
<template>
  <ToolbarRoot>
    <ToolbarButton>Action</ToolbarButton>
    <ToolbarSeparator />
    <ToolbarToggleGroup type="single" v-model="align">
      <ToolbarToggleItem value="left">Left</ToolbarToggleItem>
      <ToolbarToggleItem value="center">Center</ToolbarToggleItem>
    </ToolbarToggleGroup>
    <ToolbarLink href="#">Link</ToolbarLink>
  </ToolbarRoot>
</template>
```

## Tooltip

Popup on hover/focus. Requires `TooltipProvider` wrapping the app or a nearby ancestor.

```vue
<script setup>
import { TooltipProvider, TooltipRoot, TooltipTrigger, TooltipPortal, TooltipContent, TooltipArrow } from 'reka-ui'
</script>
<template>
  <TooltipProvider>
    <TooltipRoot>
      <TooltipTrigger>Hover me</TooltipTrigger>
      <TooltipPortal>
        <TooltipContent :side-offset="5"><TooltipArrow /></TooltipContent>
      </TooltipPortal>
    </TooltipRoot>
  </TooltipProvider>
</template>
```

- Provider props: `delay-duration`, `skip-delay-duration` (global defaults)
- Disabled button tooltip: wrap in `<span tabindex="0">`, set `pointer-events: none` on button
- Data: `[data-state]` closed/delayed-open/instant-open, `[data-side]`, `[data-align]`

## Tree

Hierarchical tree view with expand/collapse. Supports virtualization and multi-select.

```vue
<script setup>
import { TreeRoot, TreeItem } from 'reka-ui'
</script>
<template>
  <TreeRoot v-model="selected" :items="treeItems" :get-key="(item) => item.id">
    <TreeItem
      v-slot="{ item, isExpanded }"
      v-for="item in flatItems"
      :key="item.id"
      :value="item"
      :level="item.level"
    >
      {{ item.name }}
    </TreeItem>
  </TreeRoot>
</template>
```
