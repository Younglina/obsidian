---
name: vue-to-react
description: Convert Vue 3 components to React TypeScript components. Use when users want to migrate Vue files (.vue) to React (.tsx), including single file or batch directory conversion. Handles Composition API, template syntax, scoped styles to CSS Modules, and dependency mapping (ant-design-vue to antd, Pinia to Zustand, Vue Router to React Router).
---

# Vue to React Converter

Convert Vue 3 components to React TypeScript components with proper mapping of Composition API, template syntax, styles, and dependencies.

## Workflow

### Step 1: Gather Information

Ask user for:

1. **Input path**: Vue file or directory to convert
2. **Output directory**: Where to place React components

### Step 2: Analyze Input

- **Single .vue file**: Enter single file conversion mode
- **Directory**: Scan all `.vue` files, list them, and ask for confirmation before proceeding

### Step 3: Execute Conversion

For each Vue file:

#### 3.1 File Structure

```
Source: src/components/Button/index.vue
Output:
  - {output}/components/Button/index.tsx
  - {output}/components/Button/index.module.css (if has styles)
```

#### 3.2 Handle Conflicts

If target file exists, ask user: Overwrite / Skip / Overwrite All / Skip All

#### 3.3 Apply Mappings

- **Script/Template/Style mappings**: See [references/mapping-rules.md](references/mapping-rules.md)
- **Dependency mappings**: See [references/dependency-mapping.md](references/dependency-mapping.md)

#### 3.4 Handle Unsupported Features

Add `// TODO: [Vue-to-React]` comment for:

- Custom directives
- Complex `v-model` modifiers
- `Teleport`, `Transition` components
- Complex dynamic components (advanced `<component :is="...">` beyond simple icons)
- mixins
- Complex `$refs` usage

**Note:** Simple icon rendering with `<component :is="icon" />` is handled automatically with the pattern in references/mapping-rules.md

### Step 4: Generate Report

output with Chinese

```markdown
# Vue to React Conversion Report

## Statistics

- Total files: X
- Success: X
- Needs review: X
- Skipped: X

## Successfully Converted

- src/components/Button/index.vue -> output/components/Button/index.tsx

## Manual Review Required

| File             | Line | Reason                   |
| ---------------- | ---- | ------------------------ |
| Button/index.tsx | 23   | Custom directive v-focus |

## Dependency Changes

Required npm packages:

- antd (replaces ant-design-vue)
- zustand (replaces pinia)
- react-router-dom (replaces vue-router)
- clsx (for className merging)
```

## Example

**Input (Vue):**

```vue
<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <Button @click="handleAdd">Add</Button>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed } from 'vue';
  import { Button } from 'ant-design-vue';

  const props = defineProps<{ title: string }>();
  const emit = defineEmits<{ (e: 'add', item: Item): void }>();
  const items = ref<Item[]>([]);

  const handleAdd = () => {
    const newItem = { id: Date.now(), name: 'New' };
    items.value.push(newItem);
    emit('add', newItem);
  };
</script>

<style scoped>
  .container {
    padding: 16px;
  }
</style>
```

**Output (React):**

```tsx
// index.tsx
import { Button } from 'antd';
import React, { useState } from 'react';
import styles from './index.module.css';

interface Props {
  title: string;
  onAdd?: (item: Item) => void;
}

export const Component: React.FC<Props> = ({ title, onAdd }) => {
  const [items, setItems] = useState<Item[]>([]);

  const handleAdd = () => {
    const newItem = { id: Date.now(), name: 'New' };
    setItems((prev) => [...prev, newItem]);
    onAdd?.(newItem);
  };

  return (
    <div className={styles.container}>
      <h1>{title}</h1>
      <Button onClick={handleAdd}>Add</Button>
    </div>
  );
};
```

```css
/* index.module.css */
.container {
  padding: 16px;
}
```

## Guidelines

1. **Consistency**: Follow modern React best practices
2. **Progress**: Report progress after each file in batch mode
3. **Conservative**: Add TODO rather than generate incorrect code
4. **Confirm**: Ask user when Ant Design component mapping is uncertain
