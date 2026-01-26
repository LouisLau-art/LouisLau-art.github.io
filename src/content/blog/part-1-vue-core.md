---
title: "从零学 Nuxt 全栈(1)：Vue 3 核心思想 - 响应式与组件化"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本篇介绍 Vue 3 最核心的概念：响应式系统 (ref)、单文件组件 (.vue)、模板语法 (v-for, v-if)。通过逐行解读 ColorPicker.vue，带你理解 Vue 的工作原理。"
---

上一篇我们搭好了环境，现在正式进入 Vue 的世界。

如果你之前用过 Thymeleaf 或 JSP，你会习惯这种模式：**服务器把数据填进模板，返回完整 HTML**。这叫**服务端渲染 (SSR)**。

Vue 的核心思想是反过来：**浏览器拿到 JavaScript，由 JavaScript 在浏览器里生成 HTML**。这叫**客户端渲染 (CSR)**。

但这还不够，Vue 最大的卖点是：**响应式**。

## 什么是响应式 (Reactivity)？

传统做法，如果你想更新页面上的一个数字：

```javascript
// 传统 JavaScript (命令式)
let count = 0
document.getElementById('counter').innerText = count

function increment() {
  count++
  // 必须手动更新 DOM！
  document.getElementById('counter').innerText = count
}
```

每次数据变了，你都要**手动**去更新 DOM。这很繁琐，也容易出错。

Vue 的响应式系统解决了这个问题：

```vue
<script setup>
import { ref } from 'vue'

// 用 ref() 包装数据，它就变成"响应式"的了
const count = ref(0)

function increment() {
  count.value++  // 改这里，页面自动更新！
}
</script>

<template>
  <button @click="increment">
    点击次数：{{ count }}
  </button>
</template>
```

你只管改数据 (`count.value++`)，Vue 会**自动**检测到变化，然后更新页面。这就是**数据驱动视图**。

### `ref()` 是什么？

`ref()` 是 Vue 3 提供的一个函数，用来创建**响应式变量**。

```typescript
import { ref } from 'vue'

const name = ref('张三')  // 创建一个响应式字符串
const age = ref(25)       // 创建一个响应式数字
const isAdmin = ref(true) // 创建一个响应式布尔值
```

**注意**：在 `<script>` 里访问 ref 的值，必须用 `.value`：

```typescript
console.log(name.value)  // '张三'
name.value = '李四'      // 修改值
```

但在 `<template>` 里，Vue 会自动解包，直接写变量名即可：

```html
<p>{{ name }}</p>  <!-- 不需要 .value -->
```

### 类比 Spring

如果你熟悉 Spring，可以这样理解：

| Vue                    | Spring                            |
| ---------------------- | --------------------------------- |
| `ref()` 创建响应式变量        | `@Autowired` 注入一个可变的 Bean         |
| 改 `ref.value` → 视图自动更新 | 类似 Observer 模式 / `@EventListener` |

Vue 内部使用 **Proxy** (ES6 特性) 拦截对象的读写操作，一旦数据变化，就通知所有依赖这个数据的地方重新渲染。

## 单文件组件 (.vue)

Vue 项目的代码都写在 `.vue` 文件里，这叫**单文件组件 (SFC, Single-File Component)**。

一个 `.vue` 文件有三部分：

```vue
<script setup lang="ts">
// 1. JavaScript/TypeScript 逻辑
import { ref } from 'vue'
const message = ref('Hello Vue!')
</script>

<template>
  <!-- 2. HTML 模板 -->
  <p>{{ message }}</p>
</template>

<style scoped>
/* 3. CSS 样式 (scoped = 只作用于当前组件) */
p {
  color: blue;
}
</style>
```

### `<script setup>` 是什么？

`<script setup>` 是 Vue 3.2 引入的语法糖，让你写更少的代码。

**以前**你需要这样写：

```vue
<script>
import { ref, defineComponent } from 'vue'

export default defineComponent({
  setup() {
    const count = ref(0)
    return { count }  // 必须手动 return
  }
})
</script>
```

**现在**用 `<script setup>`：

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)  // 自动暴露给 template，不用 return
</script>
```

简洁了很多！

## 模板语法：v-for, v-if, @click

Vue 模板里有一些特殊的"指令"（以 `v-` 开头），类似 Thymeleaf 的 `th:each`、`th:if`。

### `v-for`：循环渲染列表

```vue
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
```

类比 Thymeleaf：

```html
<li th:each="item : ${items}" th:text="${item.name}"></li>
```

### `v-if` / `v-else`：条件渲染

```vue
<template>
  <p v-if="isLoggedIn">欢迎回来！</p>
  <p v-else>请登录</p>
</template>
```

### `@click`：事件绑定

```vue
<template>
  <button @click="handleClick">点我</button>
</template>

<script setup>
function handleClick() {
  alert('被点击了！')
}
</script>
```

`@click` 是 `v-on:click` 的缩写。

### `:style` / `:class`：动态属性

冒号 `:` 是 `v-bind:` 的缩写，用来绑定动态值。

```vue
<template>
  <!-- 动态设置背景色 -->
  <div :style="{ backgroundColor: currentColor }"></div>

  <!-- 动态设置 class -->
  <button :class="{ active: isActive }">按钮</button>
</template>
```

## 实战：解读 ColorPicker.vue

现在，让我们用学到的知识，逐行解读 Atidraw 项目中的 `app/components/ColorPicker.vue`。

```vue
<script setup lang="ts">
// 1. 定义组件接收的属性 (Props)
const props = defineProps({
  label: {
    type: String,
    default: 'Color',
  },
  default: {
    type: String,
    default: '#030712',  // 默认颜色：深灰色
  },
  icon: {
    type: String,
    default: 'i-ph-pencil',
  },
})

// 2. 定义组件可以触发的事件 (Emits)
const emit = defineEmits(['color'])

// 3. 定义颜色数组
const colors = ['#f87171', '#fb923c', /* ... 更多颜色 */]
const grayColors = ['#030712', '#1f2937', /* ... */]

// 4. 创建响应式变量：当前选中的颜色
const current = ref(props.default)

// 5. 定义函数：设置颜色
function setColor(hex: string) {
  current.value = hex      // 更新当前颜色
  emit('color', hex)       // 通知父组件颜色变了
}
</script>
```

**解读**：

- `defineProps()`：声明这个组件可以接收哪些参数（类似 Java 方法的形参）。
- `defineEmits()`：声明这个组件可以向外抛出哪些事件（下一篇详细讲）。
- `ref(props.default)`：创建一个响应式变量，初始值来自 props。
- `emit('color', hex)`：触发名为 `color` 的事件，携带 `hex` 数据。

接下来是模板部分：

```vue
<template>
  <UPopover mode="hover" :ui="{ content: 'w-[156px]' }">
    <!-- 1. 触发弹窗的按钮 -->
    <template #default="{ open }">
      <UButton color="neutral" variant="ghost" square>
        <!-- 显示当前颜色的圆点 -->
        <div
          class="w-5 h-5 rounded-full"
          :style="{ backgroundColor: current }"
        />
      </UButton>
    </template>

    <!-- 2. 弹窗内容：颜色网格 -->
    <template #content>
      <div class="p-2">
        <div class="grid grid-cols-6 gap-px">
          <!-- 用 v-for 循环渲染每个颜色按钮 -->
          <button
            v-for="color in colors"
            :key="color"
            class="w-5 h-5 rounded-full cursor-pointer"
            :class="color === current ? 'border-neutral-200' : 'border-white'"
            :style="{ backgroundColor: color }"
            @click="setColor(color)"
          />
        </div>
      </div>
    </template>
  </UPopover>
</template>
```

**解读**：

- `<UPopover>` 和 `<UButton>` 是 **Nuxt UI** 组件库提供的现成组件。
- `v-for="color in colors"`：遍历 `colors` 数组，每个颜色生成一个按钮。
- `:key="color"`：Vue 要求循环时提供唯一 key，用于优化渲染。
- `:style="{ backgroundColor: color }"`：动态设置背景色。
- `@click="setColor(color)"`：点击时调用 `setColor` 函数。

## 小结

| 概念                  | 说明                                | Spring 类比            |
| ------------------- | --------------------------------- | -------------------- |
| `ref()`             | 创建响应式变量                           | 可观察对象 (Observable)   |
| `.vue` 文件           | 单文件组件 (script + template + style) | Controller + View    |
| `v-for`             | 循环渲染                              | Thymeleaf `th:each`  |
| `v-if`              | 条件渲染                              | Thymeleaf `th:if`    |
| `@click`            | 事件绑定                              | `onclick` 属性         |
| `:style` / `:class` | 动态属性绑定                            | Thymeleaf `th:style` |

下一篇，我们将学习 **组件通信**：组件之间如何传递数据？`defineProps` 和 `defineEmits` 到底怎么用？
