---
theme: default
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Headless UI in Vue.js 3
  Building Flexible, Reusable Components
drawings:
  persist: false
transition: slide-left
title: Headless UI in Vue.js 3
mdc: true
---

# Headless UI in Vue.js 3

Building Flexible, Reusable Components

---
layout: section
---

# The Problem with Traditional Components

Let's see what goes wrong...

---
layout: default
---

# Traditional Button Component

```vue {all|1-10|12-27|28-43|45-70}{maxHeight:'400px'}
<template>
  <button 
    :class="buttonClass"
    :disabled="isLoading"
    @click="handleClick"
  >
    <LoadingSpinner v-if="isLoading" />
    <slot />
  </button>
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  variant: {
    type: String,
    default: 'primary',
    validator: (value) => ['primary', 'secondary', 'danger'].includes(value)
  },
  size: {
    type: String,
    default: 'medium',
    validator: (value) => ['small', 'medium', 'large'].includes(value)
  }
})

const isLoading = ref(false)
const handleClick = async () => {
  isLoading.value = true
  emit('click')
  setTimeout(() => isLoading.value = false, 2000)
}

const buttonClass = computed(() => {
  const baseClass = 'btn'
  const variantClass = `btn--${props.variant}`
  const sizeClass = `btn--${props.size}`
  const loadingClass = isLoading.value ? 'btn--loading' : ''

  return [baseClass, variantClass, sizeClass, loadingClass].join(' ')
})
</script>

<style scoped>
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.btn--primary { background: #3b82f6; color: white; }
.btn--secondary { background: #e5e7eb; color: #374151; }
.btn--danger { background: #ef4444; color: white; }

.btn--small { padding: 4px 8px; font-size: 12px; }
.btn--medium { padding: 8px 16px; font-size: 14px; }
.btn--large { padding: 12px 24px; font-size: 16px; }

.btn--loading { opacity: 0.6; cursor: not-allowed; }
</style>
```

---
layout: default
---

# Then Reality Hits...

<v-clicks>

<div>

**Problem 1: Design Evolution**
```vue
<!-- Now we need a button with an icon... -->
<button><Icon v-if="icon" :name="icon" /><slot /></button>

<!-- Now we need a link that looks like a button... -->
<router-link :to="to" :class="buttonClass"><slot /></router-link>

<!-- Now we need a div that acts like a button... -->
<div :class="buttonClass" @click="handleClick" role="button"><slot /></div>
```

</div>

<div>

**Problem 2: Logic Duplication**
- `ActionButton.vue` • `IconButton.vue` • `LinkButton.vue` • `CardButton.vue`
- All with 80% identical logic! 😱

</div>

<div>

**Problem 3: Testing Nightmare**
```javascript
// Why do I need to specify styling for logic tests?
const wrapper = mount(ActionButton, {
  props: { variant: 'primary' }  // 🤔
})
```

</div>

</v-clicks>

---
layout: default
---

# Props Explosion

```vue
<!-- Designer: "Can we customize this button?" -->
<!-- Developer: "Sure, I'll add more props..." -->

<ActionButton 
  variant="primary"
  size="large"
  :rounded="true"
  :shadow="true"
  :border="false"
  :uppercase="true"
  :loading="isLoading"
  :disabled="isDisabled"
  custom-class="special-button"
  icon="check"
  icon-position="left"
  :full-width="true"
  theme="dark"
/>
```

<div class="mt-8 text-center text-red-400 text-xl">
Props explosion = Maintenance nightmare!
</div>

---
layout: section
---

<h2>There have to be a better way</h2>

---
layout: two-cols
---

# What is Headless UI?

Separation of **behavior** from **appearance**

::right::

<div class="mt-8">

**Traditional Component**
```
[Logic + Styling] → Component
```

**Headless Component**  
```
[Logic] + [Styling] → Component
```

<div class="mt-6 p-4 bg-blue-50 rounded">
<div class="text-sm font-bold">The "headless" component provides:</div>
<ul class="text-sm mt-2">
<li>✅ State management</li>
<li>✅ Accessibility</li>
<li>✅ Keyboard interactions</li>
<li>❌ Zero visual opinions</li>
</ul>
</div>

</div>

---
layout: default
---

# Why Vue.js 3 is Perfect for This

<div class="grid grid-cols-2 gap-8 mt-8">

<div>

## Composition API
```javascript
export function useToggle() {
  const isEnabled = ref(false)

  function toggle () {
    isEnabled.value = !isEnabled.value
  }
  return { isEnabled, toggle }
}
```

</div>

<div>

## Flexible Slots
```vue
<HeadlessToggle v-slot="{ isEnabled, click }">
  <MyToggle
    :class="{ disabled: !isEnabled }"
    @click="click"
  >
    Save
  </MyToggle>
</HeadlessToggle>
```

</div>

</div>

<div>
```vue
<!-- HeadlessToggle -->
<template>
  <slot
    :is-enabled="isEnabled"
    :click="toggle"
  />
</template>
<script setup>
  const { isEnabled, toggle } = useToggle();
</script>
```

</div>


---
layout: section
---

# Headless UI Fundamentals

The three-layer architecture

---
layout: default
---

# Three-Layer Architecture

<div class="grid grid-cols-3 gap-4 mt-8">

<div>

### Layer 1: Logic
<p class="font-bold text-center">Composables</p>

```javascript
// Pure behavior
export function useClickable() {
  const isLoading = ref(false)
  
  async function execute(cb) {
    isLoading.value = true
    await cb()
    isLoading.value = false
  }
  
  return { isLoading, execute }
}
```

</div>

<div>

### Layer 2: Headless
<p class="font-bold text-center">Component</p>

```vue
<template>
  <slot v-bind="slotProps" />
</template>

<script setup>
const { isLoading, execute } = 
  useClickable()

const slotProps = {
  isLoading,
  handleClick: execute
}
</script>
```

</div>

<div>

### Layer 3: Styled
<p class="font-bold text-center">Implementation</p>

```vue
<template>
  <HeadlessButton
    v-slot="{ isLoading, handleClick }"
  >
    <button 
      class="btn btn--primary"
      @click="handleClick(alo)"
    >
      <Spinner v-if="isLoading" />
      <slot />
    </button>
  </HeadlessButton>
</template>
```

</div>

</div>

---
layout: default
---

# The Magic: Same Logic, Infinite Presentations

```vue
<!-- Button variant -->
<HeadlessButton v-slot="{ isLoading, handleClick }">
  <button @click="handleClick(saveData)" :disabled="isLoading">Save</button>
</HeadlessButton>

<!-- Link variant -->
<HeadlessButton v-slot="{ isLoading, handleClick }">
  <a href="#" @click.prevent="handleClick(navigate)">Continue Reading</a>
</HeadlessButton>

<!-- Card variant -->
<HeadlessButton v-slot="{ isLoading, handleClick }">
  <div 
    class="card cursor-pointer"
    :class="{ 'opacity-50': isLoading }"
    @click="handleClick(selectItem)"
  >
    <h3>Product Card</h3>
    <p>Click to select</p>
  </div>
</HeadlessButton>
```

---
layout: default
---

# What We Just Achieved ✅

<v-clicks>

<div class="mb-4">
<h3 class="text-green-400">✅ Single source of truth</h3>
<p>Logic is centralized and tested once</p>
</div>

<div class="mb-4">
<h3 class="text-green-400">✅ Unlimited styling</h3>
<p>No constraints on visual implementation</p>
</div>

<div class="mb-4">
<h3 class="text-green-400">✅ Easy testing</h3>
<p>Test logic separately from presentation</p>
</div>

<div class="mb-4">
<h3 class="text-green-400">✅ Type safety</h3>
<p>Full TypeScript support throughout</p>
</div>

</v-clicks>

---
layout: section
---

# Example

Building a Headless Dropdown

---
layout: default
---

# Complex state management with:

<v-clicks>

- **Keyboard navigation** (arrows, home, end, enter, escape)
- **Focus management** and restoration  
- **Outside click** detection
- **ARIA attributes** for screen readers
- **Portal rendering** with `teleport`

</v-clicks>

<div class="mt-8 text-center text-lg" v-click>
Let's see how headless components handle this complexity...
</div>

---
layout: default
---

# Dropdown Logic: State Management

```javascript {all|1-10|12-26|28-42|43-}{maxHeight:'400px'}
// composables/useDropdown.js
export function useDropdown(options = {}) {
  const { closeOnSelect = true } = options
  
  // State
  const isOpen = ref(false)
  const triggerRef = ref(null)
  const menuRef = ref(null)
  const focusedIndex = ref(-1)
  const items = ref([])
  
  // Register menu items
  const registerItem = (item) => {
    items.value.push(item)
    return () => {
      const index = items.value.indexOf(item)
      if (index > -1) {
        items.value.splice(index, 1)
      }
    }
  }
  
  // Focus management
  const focusedItem = computed(() => {
    return items.value[focusedIndex.value] || null
  })
  
  const focusNextItem = () => {
    if (focusedIndex.value < items.value.length - 1) {
      focusedIndex.value++
    } else {
      focusedIndex.value = 0 // Loop to start
    }
  }
  
  const focusPreviousItem = () => {
    if (focusedIndex.value > 0) {
      focusedIndex.value--
    } else {
      focusedIndex.value = items.value.length - 1 // Loop to end
    }
  }
  
  // Menu actions
  const open = async () => {
    isOpen.value = true
    focusedIndex.value = -1
    
    await nextTick()
    if (items.value.length > 0) {
      focusedIndex.value = 0
    }
  }
  
  const close = () => {
    isOpen.value = false
    focusedIndex.value = -1
    
    nextTick(() => {
      triggerRef.value?.focus()
    })
  }
```

---
layout: default
---

# Dropdown Logic: Keyboard Handling

```javascript {all|1-15|17-38|41-}{maxHeight:'400px'}
// Keyboard handling (continued from useDropdown)
const handleKeydown = (event) => {
  if (!isOpen.value) {
    if (event.key === 'Enter' || event.key === ' ' || event.key === 'ArrowDown') {
      event.preventDefault()
      open()
    }
    return
  }
  
  switch (event.key) {
    case 'Escape':
      event.preventDefault()
      close()
      break
      
    case 'ArrowDown':
      event.preventDefault()
      focusNextItem()
      break
      
    case 'ArrowUp':
      event.preventDefault()
      focusPreviousItem()
      break
      
    case 'Enter':
    case ' ':
      event.preventDefault()
      if (focusedItem.value) {
        selectItem(focusedItem.value)
      }
      break
      
    case 'Tab':
      close()
      break
  }
}

// Outside click detection
const handleClickOutside = (event) => {
  if (!isOpen.value) return
  
  const trigger = triggerRef.value
  const menu = menuRef.value
  
  if (
    trigger && !trigger.contains(event.target) &&
    menu && !menu.contains(event.target)
  ) {
    close()
  }
}

return {
  isOpen, focusedIndex, focusedItem,
  triggerRef, menuRef,
  open, close, toggle, selectItem, registerItem,
  handleKeydown
}
```

---
layout: default
---

# Headless Dropdown Component

```vue {all|1-14|15-21|22-}{maxHeight:'400px'}
<!-- components/HeadlessDropdown.vue -->
<template>
  <div class="dropdown-container">
    <!-- Trigger slot -->
    <div
      ref="triggerRef"
      :aria-expanded="isOpen"
      :aria-haspopup="true"
      :aria-controls="menuId"
      role="button"
      tabindex="0"
      @click="toggle"
      @keydown="handleKeydown"
    >
      <slot 
        name="trigger" 
        :is-open="isOpen"
        :toggle="toggle"
        :open="open"
        :close="close"
      />
    </div>
    
    <!-- Menu slot with Teleport -->
    <teleport to="body">
      <div
        v-if="isOpen"
        ref="menuRef"
        :id="menuId"
        class="dropdown-menu"
        role="menu"
        :aria-labelledby="triggerId"
      >
        <slot 
          name="menu"
          :is-open="isOpen"
          :focused-index="focusedIndex"
          :register-item="registerItem"
          :select-item="selectItem"
          :close="close"
        />
      </div>
    </teleport>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useDropdown } from '@/composables/useDropdown'

const { 
  isOpen, focusedIndex, triggerRef, menuRef,
  toggle, open, close, registerItem, selectItem, handleKeydown 
} = useDropdown()

const menuId = computed(() => `dropdown-menu-${Math.random().toString(36).substr(2, 9)}`)
const triggerId = computed(() => `dropdown-trigger-${Math.random().toString(36).substr(2, 9)}`)
</script>
```

---
layout: default
---

# Using the Headless Dropdown

```vue {all|3-8|10-28}{maxHeight:'400px'}
<template>
  <div class="demo">
    <HeadlessDropdown>
      <template #trigger="{ isOpen }">
        <button class="dropdown-trigger">
          Actions <ChevronDownIcon :class="{ 'rotate-180': isOpen }" />
        </button>
      </template>
      
      <template #menu="{ registerItem, selectItem, focusedIndex }">
        <div class="dropdown-menu-container">
          <DropdownItem 
            :register="registerItem"
            :select="selectItem"
            :focused="focusedIndex === 0"
            @select="edit"
          >
            <EditIcon /> Edit
          </DropdownItem>
          
          <DropdownItem 
            :focused="focusedIndex === 1"
            @select="delete"
          >
            <TrashIcon /> Delete
          </DropdownItem>
        </div>
      </template>
    </HeadlessDropdown>
  </div>
</template>
```

---
layout: default
---

# Features We Get For Free 🎉

<div class="grid grid-cols-2 gap-8 mt-8">

<div>

## Keyboard Navigation
- ✅ Arrow keys for navigation
- ✅ Enter/Space to select
- ✅ Escape to close
- ✅ Home/End for first/last item
- ✅ Tab to close and continue

</div>

<div>

## Accessibility  
- ✅ ARIA attributes
- ✅ Screen reader support
- ✅ Focus management
- ✅ Role-based navigation
- ✅ Proper labeling

</div>

</div>

<div class="mt-8">

## Advanced Features
- ✅ Outside click detection • ✅ Portal rendering • ✅ Focus restoration • ✅ Flexible styling

</div>

---
layout: section
---

# Testing Headless Components

Three-layer testing strategy

---
layout: default
---

# Testing Strategy: Three Layers

<div class="grid grid-cols-3 gap-4 mt-8">

<div>

### 1. Composable Tests
<p class="font-bold text-center">Unit</p>

```javascript
// Pure logic testing
test('useToggle toggles state', () => {
  const { isEnabled, toggle } = 
    useToggle(false)
  
  expect(isEnabled.value).toBe(false)
  toggle()
  expect(isEnabled.value).toBe(true)
})
```

</div>

<div>

### 2. Headless Tests
<p class="font-bold text-center">Integration</p>

```javascript
// Slot and prop behavior
test('provides correct slot props', () => {
  const wrapper = mount(HeadlessToggle, {
    slots: { default: slotSpy }
  })
  
  expect(slotSpy).toHaveBeenCalledWith(
    expect.objectContaining({
      toggle: expect.any(Function)
    })
  )
})
```

</div>

<div>

### 3. Styled Tests
<p class="font-bold text-center">E2E</p>

```javascript
// User interaction testing  
test('switch applies correct classes', () => {
  const wrapper = mount(SwitchToggle, {
    props: { modelValue: true }
  })
  
  const switchEl = wrapper.find('.switch')
  expect(switchEl.classes())
    .toContain('switch--on')
})
```

</div>

</div>

---
layout: default
---

# Testing Composables in Isolation

```javascript {all|1-12|14-23|24-}{maxHeight:'400px'}
// tests/composables/useToggle.test.js
import { describe, it, expect, vi } from 'vitest'
import { useToggle } from '@/composables/useToggle'

describe('useToggle', () => {
  it('initializes with provided value', () => {
    const { isEnabled } = useToggle(true)
    expect(isEnabled.value).toBe(true)
    
    const { isEnabled: isEnabled2 } = useToggle(false)
    expect(isEnabled2.value).toBe(false)
  })
  
  it('toggles state correctly', () => {
    const { isEnabled, toggle } = useToggle(false)
    
    expect(isEnabled.value).toBe(false)
    toggle()
    expect(isEnabled.value).toBe(true)
    toggle()
    expect(isEnabled.value).toBe(false)
  })
  
  it('calls onChange callback when state changes', () => {
    const onChange = vi.fn()
    const { toggle } = useToggle(false, { onChange })
    
    toggle()
    expect(onChange).toHaveBeenCalledWith(true)
    
    toggle()
    expect(onChange).toHaveBeenCalledWith(false)
  })
  
  it('respects disabled state', () => {
    const onChange = vi.fn()
    const { isEnabled, toggle } = useToggle(false, { 
      disabled: true, 
      onChange 
    })
    
    toggle()
    expect(isEnabled.value).toBe(false)
    expect(onChange).not.toHaveBeenCalled()
  })
})
```

---
layout: default
---

# Testing Benefits

<v-clicks>

<div class="mb-6">
<h3 class="text-green-400">✅ Isolation</h3>
<p>Test logic without DOM concerns</p>
</div>

<div class="mb-6">
<h3 class="text-green-400">✅ Speed</h3>
<p>Composable tests run faster than component tests</p>
</div>

<div class="mb-6">
<h3 class="text-green-400">✅ Coverage</h3>
<p>Easier to test edge cases and error conditions</p>
</div>

<div class="mb-6">
<h3 class="text-green-400">✅ Confidence</h3>
<p>Separate tests mean clearer failure points</p>
</div>

<div class="mb-6">
<h3 class="text-green-400">✅ Maintenance</h3>
<p>Changes to styling don't break logic tests</p>
</div>

</v-clicks>

---
layout: default
---

# Quick Decision Framework

<div class="text-center text-xl mb-8">
When should you go headless?
</div>

<div class="grid grid-cols-2 gap-8">

<div>

## ✅ Choose Headless When:

<v-clicks>

- **Multiple visual contexts** - Will be styled differently across the app
- **Complex interaction logic** - State management, keyboard handling, etc.
- **Frequent design changes** - Designers want to iterate on appearance
- **Reusable behavior** - Logic can be shared across components

</v-clicks>

</div>

<div>

## ❌ Skip Headless When:

<v-clicks>

- **Simple, static content** - No interaction or state management
- **One-off components** - Page-specific, unlikely to be reused
- **Quick prototypes** - Speed trumps flexibility
- **Single visual design** - No need for multiple variants

</v-clicks>

</div>

</div>

---
layout: default
---

<div class="grid grid-cols-2 gap-8 mt-2">

<div>

## ✅ Good for Headless

<v-clicks>

**Toggle Switch**
- Multiple styles (checkbox, switch, button)
- State management
- Accessibility requirements

**Data Table**  
- Sorting, filtering, pagination logic
- Many visual layouts possible
- Complex interactions

**Form Fields**
- Validation logic
- Many input types
- Consistent behavior across designs

</v-clicks>

</div>

<div>

## ❌ Skip Headless

<v-clicks>

**Footer Component**
- Static content
- Single design
- No interaction

**Hero Section**
- Page-specific
- Minimal logic
- Unlikely to be reused

**Simple Alert**
- Basic display logic
- Standard styling sufficient

</v-clicks>

</div>

</div>

---
layout: default
---

# Key Takeaways

<v-clicks>

<div class="mb-6">
<h3 class="text-blue-400">🧠 Separation of Concerns</h3>
<p>Logic and presentation are independent and testable</p>
</div>

<div class="mb-6">
<h3 class="text-green-400">🎨 Unlimited Flexibility</h3>
<p>Same behavior, infinite visual possibilities</p>
</div>

<div class="mb-6">
<h3 class="text-purple-400">🧪 Better Testing</h3>
<p>Test logic separately from presentation</p>
</div>

<div class="mb-6">
<h3 class="text-red-400">♿ Built-in Accessibility</h3>
<p>ARIA attributes and keyboard handling centralized</p>
</div>

<div class="mb-6">
<h3 class="text-yellow-400">📈 Improved Maintainability</h3>
<p>Single sources of truth, easier to update and extend</p>
</div>

</v-clicks>

---
layout: default
---

# Holistics components

- https://github.com/holistics/holistics-core/blob/master/packages/design-system/src/components/Button/HButton.vue

---
layout: default
---

# 3rd-party Libraries

- https://tanstack.com/table/latest/docs/introduction
- https://headlessui.com/v1/vue
- https://www.radix-ui.com/themes/docs/components/dropdown-menu

---
layout: default
---

# Reference

https://claude.ai/chat/d5e95a44-91a1-41d0-9613-273124587384

