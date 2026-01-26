---
title: "从零学 Nuxt 全栈(2)：Vue 组件通信 - Props, Events, Composables"
pubDate: 'Dec 16 2025'
tags: ["Archive"]
description: "本篇讲解 Vue 组件之间如何传递数据：Props 父传子、Events 子传父、Composables 复用逻辑。通过分析 DrawPad 与 ColorPicker 的交互，理解组件通信的最佳实践。"
---

上一篇我们学习了 Vue 的核心概念。现在你知道了：
-   `ref()` 创建响应式变量
-   `.vue` 文件包含 script + template + style
-   `v-for` / `v-if` / `@click` 等模板语法

但有个问题：**一个页面不可能只有一个组件**。当页面变复杂，我们会拆分成多个组件。那么问题来了：

> **组件 A 怎么把数据传给组件 B？**

这就是本篇要解决的问题。

## 组件树结构

在 Atidraw 的画板页面 (`pages/draw.vue`) 中，组件结构大致如下：

```
pages/draw.vue (父组件)
├── DrawPad.vue (画板)
│   ├── ColorPicker.vue (颜色选择器)
│   ├── ColorPicker.vue (背景色选择器)
│   └── StrokePicker.vue (笔触粗细选择器)
└── ... 其他 UI
```

这形成了一个**组件树**。父组件 (`DrawPad`) 需要把一些配置传给子组件 (`ColorPicker`)，子组件选完颜色后，又要通知父组件。

## Props：父传子

**Props** 是父组件向子组件传递数据的方式。

### 类比 Spring

在 Spring 中，你可能通过**构造器注入**给一个 Bean 传递依赖：

```java
@Service
public class OrderService {
    private final UserService userService;
    
    // 构造器注入
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

Vue 的 Props 也是类似的思想：父组件"注入"数据给子组件。

### 定义 Props

在 `ColorPicker.vue` 中：

```vue
<script setup lang="ts">
// 使用 defineProps 定义接收的属性
const props = defineProps({
  label: {
    type: String,      // 类型
    default: 'Color',  // 默认值
  },
  default: {
    type: String,
    default: '#030712',
  },
  icon: {
    type: String,
    default: 'i-ph-pencil',
  },
})

// 使用 props.xxx 访问
console.log(props.default)  // '#030712'
</script>
```

### 传递 Props

在父组件 `DrawPad.vue` 中使用 `ColorPicker` 时：

```vue
<template>
  <!-- 传递 props -->
  <ColorPicker 
    icon="i-ph-paint-bucket"
    default="#f9fafb"
    @color="setBackground"
  />
</template>
```

这里 `icon="i-ph-paint-bucket"` 就是传递一个名为 `icon` 的 prop，值为字符串 `"i-ph-paint-bucket"`。

**注意**：如果要传递变量（而不是字符串字面量），需要用 `:`（v-bind）：

```vue
<ColorPicker :default="currentColor" />
```

## Events：子传父

现在用户在 `ColorPicker` 里选了一个颜色，怎么通知父组件 `DrawPad`？

答案是 **Events（自定义事件）**。

### 类比 Spring

类似于 Spring 的**事件监听机制**：

```java
// 发布事件
applicationContext.publishEvent(new ColorChangedEvent(this, "#ff0000"));

// 监听事件
@EventListener
public void onColorChanged(ColorChangedEvent event) {
    System.out.println("颜色变了：" + event.getColor());
}
```

Vue 的做法类似，只不过是在组件层面。

### 定义 Emits

在 `ColorPicker.vue` 中：

```vue
<script setup>
// 声明可以触发的事件
const emit = defineEmits(['color'])

function setColor(hex) {
  // 触发 'color' 事件，携带 hex 数据
  emit('color', hex)
}
</script>
```

`defineEmits(['color'])` 告诉 Vue：这个组件可以触发名为 `color` 的事件。

### 监听 Events

在父组件 `DrawPad.vue` 中：

```vue
<template>
  <!-- @color 监听子组件的 color 事件 -->
  <ColorPicker @color="setPenColor" />
</template>

<script setup>
function setPenColor(color) {
  // color 就是子组件传过来的 hex 值
  signaturePad.value.penColor = color
}
</script>
```

`@color="setPenColor"` 的意思是：当 `ColorPicker` 触发 `color` 事件时，调用 `setPenColor` 函数，并把事件携带的数据作为参数传入。

### 完整数据流

```
1. 用户点击 ColorPicker 中的红色按钮
2. ColorPicker 调用 setColor('#f87171')
3. setColor 内部执行 emit('color', '#f87171')
4. DrawPad 监听到 @color 事件
5. DrawPad 调用 setPenColor('#f87171')
6. setPenColor 修改画笔颜色
```

这就是 **Props Down, Events Up** 的单向数据流原则。

## Composables：可复用逻辑

随着项目变复杂，你会发现有些逻辑在多个组件中重复。比如：
-   多个页面都需要检测用户是否登录
-   多个地方都需要格式化日期

在 Spring 中，你会把公共逻辑抽成 `@Service`。Vue 的对应概念叫 **Composable**，通常命名为 `useXxx`。

### 什么是 Composable？

Composable 就是一个普通的 JavaScript 函数，但它：
1.  可以使用 Vue 的响应式 API（如 `ref`）
2.  可以被多个组件复用

### 示例：useUserSession

在 Atidraw 中，`pages/draw.vue` 用到了这样的代码：

```vue
<script setup>
// 这是一个 Composable，来自 nuxt-auth-utils
const { loggedIn } = useUserSession()
</script>

<template>
  <div v-if="loggedIn">
    <!-- 已登录：显示画板 -->
    <DrawPad />
  </div>
  <div v-else>
    <!-- 未登录：显示登录按钮 -->
    <UButton to="/auth/github">Sign-in with GitHub</UButton>
  </div>
</template>
```

`useUserSession()` 是 `nuxt-auth-utils` 库提供的 Composable，返回一个对象，包含 `loggedIn`（是否登录）、`user`（用户信息）等响应式变量。

### VueUse：Composable 工具库

[VueUse](https://vueuse.org/) 是一个非常流行的 Composable 集合，提供了大量开箱即用的功能：

| Composable | 功能 |
|------------|------|
| `useMouse()` | 获取鼠标位置 |
| `useLocalStorage()` | 响应式 LocalStorage |
| `useDark()` | 暗黑模式切换 |
| `useInfiniteScroll()` | 无限滚动 |

在 Atidraw 的首页 `pages/index.vue` 中就用到了：

```vue
<script setup>
import { vInfiniteScroll } from '@vueuse/components'
</script>

<template>
  <div v-infinite-scroll="[loadMore, { distance: 10 }]">
    <!-- 滚动到底部自动加载更多 -->
  </div>
</template>
```

## 实战：DrawPad 组件分析

让我们看看 `DrawPad.vue` 如何综合运用 Props 和 Events。

```vue
<script setup lang="ts">
// 1. 定义 Props
const props = defineProps({
  saveType: {
    type: String,
    default: 'image/jpeg',
  },
  saving: {
    type: Boolean,
    default: false,
  },
})

// 2. 定义 Emits
const emit = defineEmits(['draw', 'save'])

// 3. 响应式状态
const canPost = ref(false)
const canvas = ref()
const signaturePad = ref()

// 4. 生命周期钩子 (类似 Spring 的 @PostConstruct)
onMounted(() => {
  // 初始化签名板
  signaturePad.value = new SignaturePad(canvas.value, {
    penColor: '#030712',
    backgroundColor: '#f9fafb',
  })
})

// 5. 子组件事件处理
function setPenColor(color: string) {
  signaturePad.value.penColor = color
}

// 6. 触发事件给父组件
async function save() {
  const dataURL = signaturePad.value.toDataURL(props.saveType)
  emit('save', dataURL)  // 通知父组件：需要保存
}
</script>

<template>
  <div>
    <canvas ref="canvas" />
    
    <!-- 子组件：接收 @color 事件 -->
    <ColorPicker @color="setPenColor" />
    <ColorPicker icon="i-ph-paint-bucket" @color="setBackground" />
    
    <UButton :loading="saving" @click="save">
      {{ saving ? 'Sharing...' : 'Share my drawing' }}
    </UButton>
  </div>
</template>
```

**数据流总结**：
1.  父组件 (`draw.vue`) 通过 `saving` prop 告诉 `DrawPad` 是否正在保存。
2.  `DrawPad` 包含 `ColorPicker` 子组件，监听其 `@color` 事件来更新画笔颜色。
3.  用户点击保存时，`DrawPad` 触发 `save` 事件，把画布数据传给父组件。

## 小结

| 概念 | 方向 | 用途 | Spring 类比 |
|------|------|------|-------------|
| **Props** | 父 → 子 | 传递配置/数据 | 构造器注入 |
| **Events** | 子 → 父 | 通知状态变化 | `@EventListener` |
| **Composables** | 共享 | 复用逻辑 | `@Service` |

下一篇，我们将跳出纯 Vue 的世界，进入 **Nuxt** 的领域。Nuxt 在 Vue 之上做了什么？路由怎么自动生成？SSR 是什么？
