# Vue to React Mapping Rules

## Table of Contents

- [Script Mapping](#script-mapping)
- [Template Mapping](#template-mapping)
- [Style Mapping](#style-mapping)
- [TypeScript Handling](#typescript-handling)
- [Unsupported Features](#unsupported-features)

## Script Mapping

| Vue                   | React                                       | Example                                                           |
| --------------------- | ------------------------------------------- | ----------------------------------------------------------------- |
| `ref(value)`          | `useState(value)`                           | `const count = ref(0)` -> `const [count, setCount] = useState(0)` |
| `reactive({...})`     | `useState({...})`                           | Split into multiple useState if needed                            |
| `computed(() => ...)` | `useMemo(() => ..., [deps])`                | Auto-analyze dependencies                                         |
| `watch(source, cb)`   | `useEffect(() => { cb() }, [source])`       |                                                                   |
| `watchEffect(cb)`     | `useEffect(cb, [auto-deps])`                |                                                                   |
| `onMounted`           | `useEffect(..., [])`                        |                                                                   |
| `onUnmounted`         | `useEffect` cleanup function                |                                                                   |
| `onUpdated`           | `useEffect` without dependency array        |                                                                   |
| `defineProps<T>()`    | `interface Props { ... }` + function params |                                                                   |
| `defineEmits<T>()`    | Callback props `onXxx`                      |                                                                   |
| `provide/inject`      | `React.createContext` + `useContext`        |                                                                   |

## Template Mapping

| Vue                                          | React                                                |
| -------------------------------------------- | ---------------------------------------------------- |
| `v-if="cond"`                                | `{cond && <Element />}` or ternary                   |
| `v-else`                                     | Ternary else branch                                  |
| `v-else-if`                                  | Nested ternary or extract to variable                |
| `v-for="item in list"`                       | `{list.map(item => <Element key={...} />)}`          |
| `v-model="val"`                              | `value={val} onChange={e => setVal(e.target.value)}` |
| `v-show="cond"`                              | `style={{ display: cond ? undefined : 'none' }}`     |
| `v-bind:prop="val"` or `:prop="val"`         | `prop={val}`                                         |
| `v-on:event="handler"` or `@event="handler"` | `onEvent={handler}`                                  |
| `<slot>`                                     | `{children}`                                         |
| `<slot name="xxx">`                          | `{slots.xxx}` or render props                        |
| `v-html`                                     | `dangerouslySetInnerHTML={{ __html: value }}`        |
| `class` binding                              | `className` + clsx library                           |
| `style` object binding                       | `style` object (camelCase properties)                |

## Style Mapping

Convert `<style scoped>` to CSS Modules:

**Vue:**

```vue
<style scoped>
  .button {
    color: red;
  }
</style>
```

**React:**

```tsx
import styles from './index.module.css';

// Usage: className={styles.button}
```

```css
/* index.module.css */
.button {
  color: red;
}
```

## TypeScript Handling

- Preserve all type definitions and interfaces
- Generate independent `interface` for Props
- Keep generic parameters for generic components
- Export types for external use

**Vue:**

```vue
<script setup lang="ts">
  interface Item {
    id: number;
    name: string;
  }
  const props = defineProps<{
    items: Item[];
    selected?: number;
  }>();
</script>
```

**React:**

```tsx
interface Item {
  id: number;
  name: string;
}

interface Props {
  items: Item[];
  selected?: number;
}

export const Component: React.FC<Props> = ({ items, selected }) => {
  // ...
};
```

## Unsupported Features

Add `// TODO: [Vue-to-React]` comment for these cases:

- Custom directives (e.g., `v-focus`)
- Complex `v-model` modifiers (`.number`, `.trim`, `.debounce`)
- `Teleport` component (convert to React Portal)
- `Transition` animation component
- **Complex dynamic components** `<component :is="...">` EXCEPT icon rendering
- mixins (refactor to hooks or composition)
- Complex `$refs` usage
- Ant Design component API differences

## Dynamic Components & Icons

### Simple Icon Rendering (SUPPORTED)

When Vue components use `icon?: Component` with `<component :is="icon" />`, use this pattern:

**Vue:**

```vue
<script setup lang="ts">
  import type { Component } from 'vue';

  interface Props {
    icon?: Component;
  }
</script>

<template>
  <component :is="icon" class="das-tag-icon" />
</template>
```

**React:**

```tsx
import React from 'react';

interface Props {
  icon?: React.ReactNode | React.ComponentType<{ className?: string }>;
}

const Component: React.FC<Props> = ({ icon }) => {
  const renderIcon = () => {
    if (!icon) return null;

    // If icon is a JSX element (e.g., <UserOutlined />), render directly
    if (React.isValidElement(icon)) {
      return icon;
    }

    // If icon is a component type (e.g., UserOutlined), instantiate with className
    if (typeof icon === 'function') {
      return React.createElement(icon, { className: 'das-tag-icon' });
    }

    return icon;
  };

  return <div>{renderIcon()}</div>;
};
```

**Usage in calling component:**

```tsx
// Both patterns work:
<Tag icon={<UserOutlined />} />
// OR
<Tag icon={UserOutlined} />
```

Example:

```tsx
// TODO: [Vue-to-React] Custom directive v-focus needs conversion to ref + useEffect
```
