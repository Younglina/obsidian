# Dependency Mapping

## Table of Contents

- [Ant Design Vue to Ant Design React](#ant-design-vue-to-ant-design-react)
- [Pinia/Vuex to Zustand](#piniavuex-to-zustand)
- [Vue Router to React Router](#vue-router-to-react-router)
- [Component Imports](#component-imports)
- [Required NPM Packages](#required-npm-packages)

## Ant Design Vue to Ant Design React

**Vue:**

```vue
<script setup>
  import { Button, Input } from 'ant-design-vue';
</script>
```

**React:**

```tsx
import { Button, Input } from 'antd';
```

When component APIs differ significantly, add TODO comment and ask user how to handle.

## Pinia/Vuex to Zustand

**Vue - Pinia store:**

```typescript
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  actions: {
    increment() {
      this.count++;
    }
  }
});
```

**React - Zustand store:**

```typescript
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}));
```

## Vue Router to React Router

**Vue:**

```vue
<script setup>
  import { useRouter, useRoute } from 'vue-router';

  const router = useRouter();
  const route = useRoute();
  router.push('/home');
  const id = route.params.id;
</script>
```

**React:**

```tsx
import { useNavigate, useParams } from 'react-router-dom';

const navigate = useNavigate();
const { id } = useParams();
navigate('/home');
```

## Component Imports

Convert `.vue` imports to `.tsx` (if file is in conversion scope):

**Vue:**

```typescript
import ChildComponent from './ChildComponent.vue';
```

**React:**

```typescript
import ChildComponent from './ChildComponent';
```

## Icon Libraries

**Vue:**

```typescript
import { UserOutlined } from '@ant-design/icons-vue';
```

**React:**

```typescript
import { UserOutlined } from '@ant-design/icons';
```

### Type Changes

| Vue                                    | React                                        |
| -------------------------------------- | -------------------------------------------- | -------------------------------------------- |
| `import type { Component } from 'vue'` | `import type { ComponentType } from 'react'` |
| `icon?: Component`                     | `icon?: React.ReactNode                      | React.ComponentType<{ className?: string }>` |

## Required NPM Packages

After conversion, suggest installing:

| Vue Package    | React Package                |
| -------------- | ---------------------------- |
| ant-design-vue | antd                         |
| pinia / vuex   | zustand                      |
| vue-router     | react-router-dom             |
| -              | clsx (for className merging) |
