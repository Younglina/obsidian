

舒尔特方格（Schulte Table）是一种经典的注意力训练工具，被广泛应用于飞行员选拔、运动员训练等专业领域。本文将分享如何将这一传统训练方法打造成一款现代化的 Web 应用。

## 什么是舒尔特方格？

舒尔特方格是由德国心理学家舒尔特发明的一种注意力训练方法。标准的舒尔特方格是一个 5×5 的表格，其中随机排列着 1-25 这 25 个数字。训练者需要按照从 1 到 25 的顺序，依次用眼睛找到并点击每个数字。

这种训练的核心价值在于：

*   **扩大视觉注意范围**：从"逐个搜索"逐渐过渡到"整体感知"
*   **提高眼球运动效率**：减少不必要的眼动，提升信息捕捉速度
*   **锻炼专注力持续性**：在限定时间内完成任务需要高度集中

## 效果预览


![](assets/用%20Vue%203%20打造一款专注力训练神器/file-20260116143933896.png)
![](assets/用%20Vue%203%20打造一款专注力训练神器/file-20260116143938126.png)
![](assets/用%20Vue%203%20打造一款专注力训练神器/file-20260116144005726.png)
![](assets/用%20Vue%203%20打造一款专注力训练神器/file-20260116144208825.png)
![](assets/用%20Vue%203%20打造一款专注力训练神器/file-20260116144208823.png)
## 设计

### 1. 游戏逻辑层：单例状态模式

游戏的核心逻辑被封装在 `useGameLogic` 这个组合式函数中。x采用了**模块级单例**的设计模式：

```javascript
// composables/useGameLogic.js
import { ref, computed, onUnmounted } from 'vue'

// Singleton state - 在模块级别创建，确保全局唯一
const gridNumbers = ref([])
const nextNumber = ref(1)
const isPlaying = ref(false)
const timeRemaining = ref(0)
const elapsedTime = ref(0)
const selectedMode = ref('30')

const GAME_SIZE = 5
const TOTAL_NUMBERS = GAME_SIZE * GAME_SIZE

// Fisher-Yates 洗牌算法
const shuffleArray = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
}

const generateGrid = () => {
  const numbers = Array.from({ length: TOTAL_NUMBERS }, (_, i) => i + 1)
  shuffleArray(numbers)
  gridNumbers.value = numbers
  return numbers
}
```

这种设计的优势在于：

*   **状态共享**：不同组件可以访问同一份游戏状态
*   **逻辑复用**：计时器、点击处理等逻辑只需编写一次
*   **测试友好**：游戏逻辑与 UI 解耦，便于单元测试

### 2. 双模式计时系统

应用支持两种训练模式：限时模式和不限时模式。这通过计算属性和条件分支实现：

```javascript
// 检查是否为不限时模式
const isUnlimitedMode = computed(() => selectedMode.value === 'unlimited')

const startGame = () => {
  generateGrid()
  nextNumber.value = 1
  elapsedTime.value = 0
  isPlaying.value = true

  if (isUnlimitedMode.value) {
    // 不限时模式：正计时
    timeRemaining.value = 0
    timerInterval.value = setInterval(() => {
      elapsedTime.value += 0.1
    }, 100)
  } else {
    // 限时模式：倒计时
    timeRemaining.value = timedTotal.value
    timerInterval.value = setInterval(() => {
      timeRemaining.value -= 0.1
      if (timeRemaining.value <= 0) {
        const record = endGame('timeout')
        if (onTimeoutCallback.value) {
          onTimeoutCallback.value(record)
        }
      }
    }, 100)
  }
}
```

限时模式增加了时间压力，更适合进阶训练；不限时模式则适合初学者熟悉规则。

### 3. IndexedDB 数据持久化

```javascript
// services/indexedDB.js
const DB_NAME = 'shuerte_db'
const DB_VERSION = 1
const STORE_NAME = 'records'

export function initDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, DB_VERSION)

    request.onupgradeneeded = (event) => {
      const db = event.target.result

      if (!db.objectStoreNames.contains(STORE_NAME)) {
        const store = db.createObjectStore(STORE_NAME, {
          keyPath: 'id',
          autoIncrement: true,
        })

        // 创建索引以支持高效查询
        store.createIndex('timestamp', 'timestamp', { unique: false })
        store.createIndex('result', 'result', { unique: false })
        store.createIndex('replayOf', 'replayOf', { unique: false })
        store.createIndex('mode', 'mode', { unique: false })
      }
    }

    request.onsuccess = () => resolve(request.result)
    request.onerror = () => reject(request.error)
  })
}
```

选择 IndexedDB 的理由：

*   **存储容量大**：localStorage 只有 5MB，IndexedDB 可存储 GB 级数据
*   **自增主键**：自动生成唯一 ID，便于管理记录
*   **索引支持**：可以按时间戳、结果等字段快速查询
*   **异步操作**：不会阻塞主线程

### 4. 重练功能的实现

一个独特的功能是"重练"——用户可以选择历史记录中的任意一次训练，使用**相同的数字排列**重新挑战：

```javascript
// 重新训练：使用指定的 gridNumbers 开始游戏
const replayGame = (gridNumbersArray, mode) => {
  selectedMode.value = mode
  gridNumbers.value = gridNumbersArray // 复用原有排列
  nextNumber.value = 1
  elapsedTime.value = 0
  isPlaying.value = true

  // 启动对应模式的计时器...
}
```

这个功能的价值在于：

*   用户可以针对"困难"的排列反复练习
*   可以比较同一排列下不同次的成绩变化
*   增加了训练的趣味性和挑战性

## 用户体验设计

### 1. 视觉反馈系统

应用提供了丰富的视觉反馈：

```scss
// 根据结果类型显示不同边框颜色
.history-item {
  &.result-success {
    border-left: 4px solid #4ade80; // 绿色 - 成功
  }
  &.result-fail {
    border-left: 4px solid #ff2d55; // 红色 - 失败
  }
  &.result-timeout {
    border-left: 4px solid #f59e0b; // 橙色 - 超时
  }
}
```

### 2. 成绩评级系统

基于完成时间给出 S/A/B/C/D 五个等级的评价：

```javascript
const getRating = (record) => {
  const time = record.timeUsed

  if (record.isUnlimited) {
    // 不限时模式：宽松标准
    if (time <= 15) return 'S'
    if (time <= 25) return 'A'
    if (time <= 35) return 'B'
    if (time <= 50) return 'C'
    return 'D'
  } else {
    // 限时模式：严格标准
    if (time <= 8) return 'S'
    if (time <= 15) return 'A'
    if (time <= 20) return 'B'
    if (time <= 25) return 'C'
    return 'D'
  }
}
```

不同模式采用不同的评级标准，确保公平性。

### 3. 庆祝动画

成功完成训练时，使用 canvas-confetti 库触发彩带效果：

```javascript
import confetti from 'canvas-confetti'

const fireConfetti = () => {
  const duration = 3000
  const animationEnd = Date.now() + duration
  const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 2000 }

  const interval = setInterval(() => {
    const timeLeft = animationEnd - Date.now()
    if (timeLeft <= 0) return clearInterval(interval)

    const particleCount = 50 * (timeLeft / duration)
    confetti({
      ...defaults,
      particleCount,
      origin: { x: Math.random() * 0.4 + 0.1, y: Math.random() - 0.2 },
    })
  }, 250)
}
```

这种即时的正向反馈能够显著提升用户的成就感和继续训练的动力。

## 统计与可视化

应用集成了 Chart.js 来展示训练数据：

```vue
<template>
  <StatsChart
    type="bar"
    :data="resultChartData"
    title="训练结果分布"
    subtitle="成功 / 失败 / 超时"
  />

  <StatsChart
    type="line"
    :data="timeChartData"
    title="成绩趋势"
    subtitle="最近 20 次训练"
  />
</template>
```

通过可视化图表，用户可以直观地看到：

*   成功率的变化
*   完成时间的趋势
*   不同模式的表现对比

## 性能优化要点

### 1. 精确的方格尺寸

为了还原标准舒尔特方格（1cm × 1cm），我使用了固定像素值而非相对单位：

```scss
.grid-cell {
  width: 38px;  // 约 1cm
  height: 38px;
  font-family: 'SimSun', 'STSong', 'Songti SC', serif;
  font-size: 16px;
}
```

### 2. 动画性能

所有动画都使用 CSS `transform` 和 `opacity` 属性，确保 GPU 加速：

```scss
@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
```

### 3. 组件懒加载

通过 Vue Router 的动态导入实现页面级代码分割：

```javascript
const routes = [
  {
    path: '/training',
    component: () => import('./pages/TrainingPage.vue')
  },
  {
    path: '/stats',
    component: () => import('./pages/StatsPage.vue')
  }
]
```

*本项目源码已开源，欢迎 Star 和 PR。技术栈：Vue 3 + Vite + TailwindCSS + IndexedDB*
在线体验：<https://younglina.wang/shulte>
源码地址：<https://github.com/younglina/shulte.git>
