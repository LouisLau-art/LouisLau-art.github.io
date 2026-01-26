---
title: 'Una UI 核心组件 API 摘要 (精简版)'
description: '这篇文章不是给人类看的'
pubDate: 'Dec 18 2025'
tags: ["Archive"]
---

## 数据展示与布局 (Data Display & Layout)

### Accordion (手风琴)

**描述:** 一组垂直堆叠的交互式标题，每个标题都会显示一个内容部分 **核心属性 (Props):**

| Prop                        | 默认值              | 类型          | 描述                                              |
| --------------------------- | ---------------- | ----------- | ----------------------------------------------- |
| `items`                     | `[]`             | `array`     | 设置手风琴项目。                                        |
| `item.value`                | -                | `string`    | 用于渲染和状态跟踪的唯一值。                                  |
| `type`                      | `single`         | `'single' \ | 'multiple'`                                     |
| `defaultValue`              | -                | `string \   | string[]`                                       |
| `unmountOnHide`             | `true`           | `boolean`   | 设置为 `false` 可保持内容被挂载，即使手风琴已关闭（出于性能考虑，默认关闭不渲染）,。 |
| `_accordionTrigger.leading` | -                | `string`    | 为手风琴设置自定义前置图标。                                  |
| `accordion`                 | `border divider` | `{variant}` | 手风琴的变体。                                         |

**插槽 (Slots):**

| Name               | Props                   | 描述                      |
| ------------------ | ----------------------- | ----------------------- |
| `default`          | `{ modelValue }`        | 填充 `AccordionItem` 组件。  |
| `item`             | `{ open, item, index }` | 手风琴的项目。                 |
| `trigger`          | `{ open, item, index }` | 触发按钮。                   |
| `content`          | `{ open, item, index }` | 手风琴的内容。                 |
| `#{value}-content` | -                       | 针对特定项目（按其 `value`）定制内容。 |

### Aspect Ratio (纵横比)

**描述:** 在所需的纵横比内显示内容。 **核心属性 (Props):**

| Prop           | 默认值         | 类型                  | 描述                |
| -------------- | ----------- | ------------------- | ----------------- |
| `ratio`        | `1`         | `number`            | 期望的纵横比。例如：`16/9`。 |
| `aspect-ratio` | `soft-gray` | `{variant}-{color}` | 更改组件的样式。          |
| `rounded`      | `md`        | `string`            | 设置圆角。             |

**插槽 (Slots):**

| Name      | Props    | 描述    |
| --------- | -------- | ----- |
| `default` | `aspect` | 默认插槽。 |

### Avatar (头像)

**描述:** 一个带有回退机制的图像元素，用于表示用户。 **核心属性 (Props):**

| Prop      | 默认值     | 类型          | 描述                              |
| --------- | ------- | ----------- | ------------------------------- |
| `src`     | -       | `string`    | 图像源。                            |
| `alt`     | -       | `string`    | 替代文本。                           |
| `label`   | -       | `string`    | 加载时的占位符（默认为 `alt` 属性中每个单词的首字母）。 |
| `avatar`  | `soft`  | `{variant}` | 头像的变体。                          |
| `icon`    | `false` | `boolean`   | 如果为 `true`，`label` 将被包装为图标组件。   |
| `size`    | `md`    | `string`    | 设置头像的大小。                        |
| `rounded` | `full`  | `string`    | 设置圆角。                           |

**插槽 (Slots):**

| Name       | Props | 描述       |
| ---------- | ----- | -------- |
| `default`  | -     | 头像的图像元素。 |
| `fallback` | -     | 头像的回退元素。 |

### Avatar Group (头像组)

**描述:** 显示一组 `Avatar` 组件。 **核心属性 (Props):**

| Prop             | 默认值     | 类型       | 描述                |
| ---------------- | ------- | -------- | ----------------- |
| `max`            |         | `number` | 在隐藏其余头像之前显示的最大数量。 |
| `overflow-label` | `+${N}` | `string` | 覆盖默认的溢出标签。        |
| `size`           | `md`    | `string` | 设置头像的大小。          |
| `square`         | `2.5em` | `string` | 设置头像为方形。          |

**插槽 (Slots):**

| Name      | Props | 描述                     |
| --------- | ----- | ---------------------- |
| `default` | -     | `AvatarGroup` 组件的默认插槽。 |

### Badge (徽章)

**描述:** 显示一个徽章或看起来像徽章的组件。 **核心属性 (Props):**

| Prop       | 默认值                 | 类型                  | 描述        |
| ---------- | ------------------- | ------------------- | --------- |
| `label`    | -                   | `string`            | 徽章的标签。    |
| `badge`    | `soft`              | `{variant}`         | 徽章的变体。    |
| `badge`    | `{variant}-primary` | `{variant}-{color}` | 徽章的颜色。    |
| `size`     | `xs`                | `string`            | 更改大小。     |
| `closable` | `false`             | `boolean`           | 添加一个关闭按钮。 |

**事件 (Events):**

| Event Name | 描述                              |
| ---------- | ------------------------------- |
| `@close`   | 点击关闭图标时触发事件（与 `closable` 配合使用）。 |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 徽章标签。 |

### Card (卡片)

**描述:** 显示带有页眉、内容和页脚的卡片。 **核心属性 (Props):**

| Prop          | 默认值                 | 类型                  | 描述               |
| ------------- | ------------------- | ------------------- | ---------------- |
| `title`       | -                   | `string`            | 设置卡片页眉中显示的主标题文本。 |
| `description` | -                   | `string`            | 提供显示在标题下方的副文本。   |
| `card`        | `outline`           | `{variant}`         | 控制卡片的视觉样式。       |
| `card`        | `{variant}-primary` | `{variant}-{color}` | 结合变体和颜色来定义卡片外观。  |

**插槽 (Slots):**

| Name      | Props | 描述                     |
| --------- | ----- | ---------------------- |
| `header`  | -     | 卡片的页眉部分。               |
| `default` | -     | 卡片的主要内容区域。             |
| `title`   | -     | 覆盖 `title` 属性的自定义标题内容。 |
| `footer`  | -     | 卡片的页脚部分。               |

### Collapsible (可折叠)

**描述:** 一个可展开/折叠面板的交互式组件。 **核心属性 (Props):**

| Prop          | 默认值     | 类型        | 描述                      |
| ------------- | ------- | --------- | ----------------------- |
| `defaultOpen` | `false` | `boolean` | 初次渲染时的开放状态。             |
| `disabled`    | -       | `boolean` | 为 `true` 时，阻止用户与组件交互。   |
| `open`        | -       | `boolean` | 受控的开放状态 (`v-model` 绑定)。 |

**插槽 (Slots):**

| Name      | Props  | 描述       |
| --------- | ------ | -------- |
| `default` | `open` | 可折叠根插槽。  |
| `content` | -      | 可折叠内容插槽。 |
| `trigger` | `open` | 按钮触发器插槽。 |

### Icon (图标)

**描述:** 显示来自各种图标库的图标。

- **注意:** UnaUI 默认集成了 Iconify，并包含 Lucide、Reka Icons 和 Tabler Icons。

### Indicator (指示器)

**描述:** 显示一个指示器，可用于显示任务或组件的状态。 **核心属性 (Props):**

| Prop        | 默认值                 | 类型                  | 描述                                     |
| ----------- | ------------------- | ------------------- | -------------------------------------- |
| `label`     | -                   | `string`            | 指示器的标签。                                |
| `indicator` | `solid`             | `{variant}`         | 指示器的变体。                                |
| `indicator` | `{variant}-primary` | `{variant}-{color}` | 指示器的颜色。                                |
| `ping`      | `false`             | `boolean`           | 启用 Ping 动画。                            |
| `size`      | `md`                | `string`            | 指示器的大小。                                |
| `visible`   | `true`              | `boolean`           | 显示或隐藏指示器。                              |
| `indicator` | `top-right`         | `{placement}`       | 指示器的位置（例如 `top-right`, `bottom-left`）。 |

**插槽 (Slots):**

| Name        | Props | 描述            |
| ----------- | ----- | ------------- |
| `default`   | -     | 指示器的内容。       |
| `indicator` | -     | 整个指示器（会重置位置）。 |
| `label`     | -     | 指示器的标签。       |

### Kbd (键盘按键)

**描述:** 指示通常通过键盘输入的输入。 **核心属性 (Props):**

| Prop   | 默认值              | 类型                  | 描述       |
| ------ | ---------------- | ------------------- | -------- |
| `kbd`  | `solid`          | `{variant}`         | Kbd 的变体。 |
| `kbd`  | `{variant}-gray` | `{variant}-{color}` | Kbd 的颜色。 |
| `size` | `md`             | `string`            | Kbd 的大小。 |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 内容插槽。 |

### Progress (进度条)

**描述:** 显示任务完成进度的指示器，通常显示为进度条。 **核心属性 (Props):**

| Prop            | 默认值       | 类型                                         | 描述                  |
| --------------- | --------- | ------------------------------------------ | ------------------- |
| `modelValue`    | -         | `number`                                   | 进度值 (`v-model` 绑定)。 |
| `max`           | `100`     | `number`                                   | 最大进度值。              |
| `indeterminate` | `false`   | `boolean`                                  | 使进度条不确定（无特定值时动画）。   |
| `getValueLabel` | 默认百分比     | `((value: number, max: number) => string)` | 获取可访问标签文本的函数。       |
| `rounded`       | `full`    | `string`                                   | 设置进度条的圆角。           |
| `progress`      | `primary` | `string`                                   | 设置进度条的颜色。           |
| `size`          | `md`      | `string`                                   | 设置进度条的大小。           |

**插槽 (Slots):**

| Name      | Props | 描述     |
| --------- | ----- | ------ |
| `default` | -     | 进度指示器。 |

### Separator (分隔符)

**描述:** 在视觉或语义上分隔内容。 **核心属性 (Props):**

| Prop                 | 默认值          | 类型                                 | 描述           |
| -------------------- | ------------ | ---------------------------------- | ------------ |
| `label`              | -            | `string`                           | 标签内容。        |
| `separator`          | `solid-gray` | `{variant}-{color}`                | 设置分隔符的变体和颜色。 |
| `orientation`        | `horizontal` | `horizontal, vertical`             | 设置分隔符的方向。    |
| `separator-position` | `center`     | `center, left, right, top, bottom` | 设置标签内容的位置。   |
| `size`               | -            | `string`                           | 设置分隔符的大小。    |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 标签内容。 |

### Skeleton (骨架屏)

**描述:** 用于在内容加载时显示占位符。 **核心属性 (Props):**

| Prop       | 默认值    | 类型       | 描述       |
| ---------- | ------ | -------- | -------- |
| `rounded`  | `none` | `string` | 更改骨架的形状。 |
| `skeleton` | `gray` | `string` | 更改骨架的颜色。 |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 默认插槽。 |

## 表单与输入 (Forms & Inputs)

### Checkbox (复选框)

**描述:** 允许用户在“选中”和“未选中”之间切换的控件。 **核心属性 (Props):**

| Prop                      | 默认值       | 类型                       | 描述                     |
| ------------------------- | --------- | ------------------------ | ---------------------- |
| `modelValue`              | -         | `boolean, indeterminate` | 受控选中状态 (`v-model` 绑定)。 |
| `defaultValue`            | -         | `boolean`                | 初始渲染时的选中状态。            |
| `disabled`                | -         | `boolean`                | 禁用交互。                  |
| `label`                   | -         | `string`                 | 设置复选框的标签。              |
| `color`                   | `primary` | `string`                 | 更改复选框的颜色。              |
| `size`                    | `md`      | `string`                 | 调整复选框的大小。              |
| `reverse`                 | -         | `boolean`                | 反转复选框和标签的位置。           |
| `una.checkboxCheckedIcon` | `i-check` | `string`                 | 选中时的自定义图标。             |

**插槽 (Slots):**

| Name      | Props               | 描述          |
| --------- | ------------------- | ----------- |
| `default` | -                   | 用于定制复选框的标签。 |
| `icon`    | `state, modelValue` | 用于定制复选框的图标。 |

### Combobox (组合框)

**描述:** 带有建议列表的自动完成输入和命令面板。 **核心属性 (Props):**

| Prop               | 默认值                           | 类型                      | 描述                                      |
| ------------------ | ----------------------------- | ----------------------- | --------------------------------------- |
| `items`            | -                             | `T[] \                  | NComboboxGroupProps<ExtractItemType>[]` |
| `modelValue`       | -                             | `AcceptableValue \      | AcceptableValue[]`                      |
| `multiple`         | `false`                       | `boolean`               | 启用多项选择模式。                               |
| `labelKey`         | `label`                       | `string`                | 用于在选择项中显示的键名。                           |
| `textEmpty`        | `No items found.`             | `string`                | 组合框为空时显示的文本。                            |
| `_comboboxTrigger` | `{ btn: 'solid-white', ... }` | `NComboboxTriggerProps` | 触发按钮的属性。                                |
| `size`             | `sm`                          | `string`                | 调整组件的整体大小。                              |

**插槽 (Slots):**

| Name      | Props              | 描述                   |
| --------- | ------------------ | -------------------- |
| `default` | -                  | 用于使用子组件进行高级自定义渲染的插槽。 |
| `trigger` | `modelValue, open` | 默认触发按钮 *内部* 的自定义内容。  |
| `item`    | `item, selected`   | 每个组合框项的整个内容的自定义渲染。   |
| `header`  | -                  | 列表内部、项目之前渲染的内容。      |
| `footer`  | -                  | 列表内部、项目之后渲染的内容。      |

### Input / Textarea (输入框 / 文本区域)

**描述:** 显示表单输入字段或看起来像输入字段的组件。 **核心属性 (Props):**

| Prop                         | 默认值                 | 类型                              | 描述            |
| ---------------------------- | ------------------- | ------------------------------- | ------------- |
| `type`                       | -                   | `HTMLInputElement['type'] \     | textarea`     |
| `modelValue`                 | -                   | `any`                           | 输入的值。         |
| `reverse`                    | `false`             | `boolean`                       | 交换前置和后置图标的位置。 |
| `input`                      | `outline`           | `{variant}`                     | 输入的变体。        |
| `input`                      | `{variant}-primary` | `{variant}-{color}`             | 输入的颜色。        |
| `size`                       | `md`                | `string`                        | 更改输入的大小。      |
| `leading`                    | -                   | `string`                        | 显示前置图标。       |
| `trailing`                   | -                   | `string`                        | 显示后置图标。       |
| `loading`                    | -                   | `boolean`                       | 显示加载状态。       |
| `status`                     | -                   | `info, success, warning, error` | 更新输入状态。       |
| `rows` (Textarea only)       | -                   | `number`                        | 设置文本区域的行数。    |
| `autoresize` (Textarea only) | `false`             | `boolean number`                | 启用文本区域自动调整大小。 |

**事件 (Events):**

| Event       | 描述         |
| ----------- | ---------- |
| `@leading`  | 点击前置图标时触发。 |
| `@trailing` | 点击后置图标时触发。 |

**暴露的方法 (Expose):**

| Name    | 类型           | 描述       |
| ------- | ------------ | -------- |
| `focus` | `() => void` | 自动聚焦输入框。 |

**插槽 (Slots):**

| Name       | Props | 描述    |
| ---------- | ----- | ----- |
| `default`  | -     | 内容插槽。 |
| `leading`  | -     | 前置插槽。 |
| `trailing` | -     | 后置插槽。 |

### Label (标签)

**描述:** 渲染与控件关联的可访问标签。 **核心属性 (Props):**

| Prop    | 默认值      | 类型                 | 描述               |
| ------- | -------- | ------------------ | ---------------- |
| `as`    | `label`  | `AsTag, Component` | 此组件应渲染为的元素或组件。   |
| `for`   | `string` | `string`           | 标签关联元素的 ID。      |
| `label` | `string` | `string`           | 标签文本（默认插槽的替代方案）。 |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 内容插槽。 |

### Number Field (数字输入框)

**描述:** 允许用户输入和调整数值，带有可选的递增和递减控件。 **核心属性 (Props):**

| Prop                 | 默认值               | 类型                  | 描述                     |
| -------------------- | ----------------- | ------------------- | ---------------------- |
| `modelValue`         | -                 | `number \           | null`                  |
| `max`                | -                 | `number`            | 允许的最大值。                |
| `min`                | -                 | `number`            | 允许的最小值。                |
| `step`               | `1`               | `number`            | 每次递增或递减时的变化量。          |
| `disableWheelChange` | -                 | `boolean`           | 为 `true` 时，阻止滚轮滚动时值改变。 |
| `number-field`       | `outline-primary` | `{variant}-{color}` | 控制数字字段的视觉样式。           |
| `size`               | `md`              | `string`            | 更改大小。                  |
| `leading`            | -                 | `string`            | 输入前显示的图标。              |

**插槽 (Slots):**

| Name        | Props | 描述                |
| ----------- | ----- | ----------------- |
| `default`   | -     | 用于使用子组件进行高级自定义渲染。 |
| `increment` | -     | 数字字段递增按钮的自定义内容。   |
| `decrement` | -     | 数字字段递减按钮的自定义内容。   |

### Pin Input (PIN 输入)

**描述:** 用于输入短序列数字或字符的组件，通常用于验证码或 PIN 码。 **核心属性 (Props):**

| Prop         | 默认值               | 类型                  | 描述                         |
| ------------ | ----------------- | ------------------- | -------------------------- |
| `modelValue` | -                 | `PinInputValue \    | null`                      |
| `maxLength`  | -                 | `number`            | 指定 PIN 输入的字段数量。            |
| `mask`       | `boolean`         | -                   | 启用时将值屏蔽为密码字段。              |
| `otp`        | `boolean`         | -                   | 在支持的移动设备上启用 OTP 自动检测和自动完成。 |
| `groupBy`    | `0`               | `number`            | 指定要分组的输入字段数量。              |
| `pin-input`  | `outline-primary` | `{variant}-{color}` | 控制 PIN 输入的视觉样式。            |
| `size`       | `md`              | `string`            | 更改大小。                      |
| `separator`  | `false`           | `boolean \          | string`                    |

**插槽 (Slots):**

| Name        | Props | 描述                 |
| ----------- | ----- | ------------------ |
| `default`   | -     | 允许使用子组件进行高级自定义渲染。  |
| `group`     | -     | 替换整个组容器（包括输入和分隔符）。 |
| `separator` | -     | PIN 输入的自定义分隔符。     |

### Radio Group (单选组)

**描述:** 一组可选中按钮，同一时间只能选中其中一个。 **核心属性 (Props):**

| Prop          | 默认值        | 类型                     | 描述                  |
| ------------- | ---------- | ---------------------- | ------------------- |
| `modelValue`  | -          | `string`               | 受控值 (`v-model` 绑定)。 |
| `items`       | -          | `RadioGroupItem[]`     | 要渲染的单选项目。           |
| `valueKey`    | `value`    | `string`               | 用于 `value` 属性的键名。   |
| `orientation` | `vertical` | `vertical, horizontal` | 控制布局方向。             |
| `radio-group` | `primary`  | `{color}`              | 设置单选组的颜色。           |
| `size`        | `md`       | `string`               | 设置单选组的大小。           |
| `item.icon`   | -          | `string`               | 特定项目的图标。            |

**插槽 (Slots):**

| Name      | Props   | 描述    |
| --------- | ------- | ----- |
| `default` | `items` | 项目插槽。 |
| `label`   | -       | 标签插槽。 |
| `icon`    | -       | 图标插槽。 |

### Select (选择框)

**描述:** 显示供用户选择的选项列表，由按钮触发。 **核心属性 (Props):**

| Prop             | 默认值          | 类型                  | 描述                           |
| ---------------- | ------------ | ------------------- | ---------------------------- |
| `items`          | -            | `T[] \              | SelectGroup[]`               |
| `modelValue`     | -            | `string`            | 受控值 (`v-model` 绑定)。          |
| `placeholder`    | -            | `string`            | 未设置值时在 `SelectValue` 中渲染的内容。 |
| `multiple`       | `false`      | `boolean`           | 启用多选模式。                      |
| `groupSeparator` | `false`      | `boolean`           | 是否在组之间显示分隔符。                 |
| `valueKey`       | -            | `string`            | 在选中值中显示的键名。                  |
| `select`         | `soft-white` | `{variant}-{color}` | 更改选择框的颜色。                    |
| `size`           | `sm`         | `string`            | 调整组件的整体大小。                   |

**插槽 (Slots):**

| Name      | Props              | 描述             |
| --------- | ------------------ | -------------- |
| `default` | `modelValue, open` | 对根元素和状态的完全访问。  |
| `trigger` | `modelValue, open` | 自定义触发器内容。      |
| `value`   | `modelValue, open` | 自定义触发器中选定值的显示。 |
| `item`    | `item`             | 自定义单个下拉项。      |

### Slider (滑块)

**描述:** 允许用户在给定范围内选择值的输入。 **核心属性 (Props):**

| Prop          | 默认值          | 类型                     | 描述                  |
| ------------- | ------------ | ---------------------- | ------------------- |
| `modelValue`  | -            | `number`               | 受控值 (`v-model` 绑定)。 |
| `max`         | `100`        | `number`               | 最大值。                |
| `min`         | `0`          | `number`               | 最小值。                |
| `step`        | `1`          | `number`               | 步进值。                |
| `orientation` | `horizontal` | `vertical, horizontal` | 滑块的方向。              |
| `slider`      | `primary`    | `string`               | 更改滑块的颜色。            |
| `size`        | `md`         | `string`               | 设置滑块的常规大小。          |

**插槽 (Slots):**

| Name           | Props | 描述            |
| -------------- | ----- | ------------- |
| `slider-track` | -     | 滑块的轨道。        |
| `slider-thumb` | -     | 滑块的滑块（Thumb）。 |

### Switch (开关)

**描述:** 允许用户在“选中”和“未选中”之间切换的控件。 **核心属性 (Props):**

| Prop               | 默认值       | 类型        | 描述                   |
| ------------------ | --------- | --------- | -------------------- |
| `modelValue`       | -         | `boolean` | 受控状态 (`v-model` 绑定)。 |
| `disabled`         | `false`   | `boolean` | 禁用交互。                |
| `switch-checked`   | `primary` | `string`  | 选中时开关的颜色。            |
| `switch-unchecked` | `gray`    | `string`  | 未选中时开关的颜色。           |
| `size`             | `md`      | `string`  | 更改开关的大小。             |
| `checkedIcon`      | -         | `string`  | 开关打开时显示的图标。          |
| `loading`          | -         | `boolean` | 设置开关处于加载状态。          |

**插槽 (Slots):**

| Name           | Props     | 描述                  |
| -------------- | --------- | ------------------- |
| `icon`         | `checked` | 开关在选中和未选中状态下的自定义图标。 |
| `loading-icon` | `checked` | 加载图标插槽。             |

### Toggle (切换按钮)

**描述:** 一个可以打开或关闭的双状态按钮。 **核心属性 (Props):**

| Prop         | 默认值           | 类型                  | 描述                     |
| ------------ | ------------- | ------------------- | ---------------------- |
| `modelValue` | -             | `boolean`           | 受控按下状态 (`v-model` 绑定)。 |
| `disabled`   | `false`       | `boolean`           | 禁用用户交互。                |
| `toggle-on`  | `soft-accent` | `{variant}-{color}` | 切换按钮打开时的颜色。            |
| `toggle-off` | `ghost-gray`  | `{variant}-{color}` | 切换按钮关闭时的颜色。            |

### Toggle Group (切换按钮组)

**描述:** 一组可切换开/关的双状态按钮。 **核心属性 (Props):**

| Prop          | 默认值           | 类型                     | 描述          |
| ------------- | ------------- | ---------------------- | ----------- |
| `items`       | `[]`          | `Array<T>`             | 在切换组中显示的项。  |
| `modelValue`  | -             | `AcceptableValue \     | Array`      |
| `type`        | -             | `single \              | multiple`   |
| `orientation` | `horizontal`  | `horizontal, vertical` | 设置切换组的方向。   |
| `toggle-on`   | `soft-accent` | `{variant}-{color}`    | 项目打开时的颜色。   |
| `size`        | `sm`          | `string`               | 调整切换组的整体大小。 |

**插槽 (Slots):**

| Name      | Props          | 描述           |
| --------- | -------------- | ------------ |
| `default` | `modelValue`   | 默认插槽，覆盖所有内容。 |
| `item`    | `item, active` | 项目静态插槽。      |

## 导航与链接 (Navigation & Links)

### Breadcrumb (面包屑)

**描述:** 使用链接层次结构显示当前资源的路径。 **核心属性 (Props):**

| Prop                | 默认值                             | 类型                  | 描述           |
| ------------------- | ------------------------------- | ------------------- | ------------ |
| `items`             | `[]`                            | `array`             | 链接数组。        |
| `separator`         | `i-radix-icons-chevron-right`   | `string`            | 分隔符图标。       |
| `ellipsis`          | `i-radix-icons-dots-horizontal` | `string`            | 省略号图标。       |
| `size`              | `sm`                            | `string`            | 调整组件的整体大小。   |
| `breadcrumb-active` | `text-primary`                  | `{variant}-{color}` | 活动面包屑的变体和颜色。 |

**插槽 (Slots):**

| Name        | Props                 | 描述               |
| ----------- | --------------------- | ---------------- |
| `default`   | `item index isActive` | 面包屑项目。           |
| `separator` | `item`                | 分隔符。             |
| `dropdown`  | `item`                | 如果可用（省略号）则为下拉菜单。 |

### Button (按钮)

**描述:** 显示一个按钮或看起来像按钮的组件。 **核心属性 (Props):**

| Prop       | 默认值                 | 类型                  | 描述         |
| ---------- | ------------------- | ------------------- | ---------- |
| `label`    | -                   | `string`            | 按钮的标签。     |
| `btn`      | `solid`             | `{variant}`         | 按钮的变体。     |
| `btn`      | `{variant}-primary` | `{variant}-{color}` | 按钮的颜色。     |
| `size`     | `sm`                | `string`            | 更改按钮的大小。   |
| `block`    | -                   | `boolean`           | 设置按钮为全宽。   |
| `leading`  | -                   | `string`            | 显示前置图标。    |
| `trailing` | -                   | `string`            | 显示后置图标。    |
| `loading`  | -                   | `boolean`           | 设置按钮为加载状态。 |
| `to`       | -                   | `string`            | 导航链接。      |

**插槽 (Slots):**

| Name       | Props | 描述    |
| ---------- | ----- | ----- |
| `default`  | -     | 按钮标签。 |
| `leading`  | -     | 前置图标。 |
| `trailing` | -     | 后置图标。 |
| `loading`  | -     | 加载图标。 |

### Link (链接)

**描述:** 提供一个自定义的 `<NuxtLink>` 组件来处理应用程序内的任何类型的链接。 **核心属性 (Props):**

| Prop          | 默认值     | 类型        | 描述              |
| ------------- | ------- | --------- | --------------- |
| `label`       | -       | `string`  | 链接的标签。          |
| `activeClass` | -       | `string`  | 链接激活时应用的类。      |
| `exact`       | `false` | `boolean` | 仅在路径完全匹配时触发活动类。 |
| `disabled`    | `false` | `boolean` | 链接是否禁用。         |

**插槽 (Slots):**

| Name      | Props | 描述     |
| --------- | ----- | ------ |
| `default` | -     | 链接的内容。 |

### Navigation Menu (导航菜单)

**描述:** 显示用于导航网站的链接集合。 **核心属性 (Props):**

| Prop              | 默认值           | 类型                     | 描述                          |
| ----------------- | ------------- | ---------------------- | --------------------------- |
| `items`           | `[]`          | `T`                    | 在导航菜单中显示的项。                 |
| `modelValue`      | -             | `string`               | 要激活的菜单项的受控值 (`v-model` 绑定)。 |
| `indicator`       | `false`       | `boolean`              | 设置在列表下方渲染的指示器。              |
| `orientation`     | `horizontal`  | `horizontal, vertical` | 设置菜单的方向。                    |
| `size`            | `sm`          | `string`               | 调整组件的整体大小。                  |
| `navigation-menu` | `ghost-white` | `{variant}-{color}`    | 设置导航菜单的变体和颜色。               |

**插槽 (Slots):**

| Name           | Props                     | 描述           |
| -------------- | ------------------------- | ------------ |
| `default`      | `items`                   | 默认插槽，覆盖所有内容。 |
| `trigger`      | `item, index, modelValue` | 触发器插槽。       |
| `item`         | `item, active`            | 项目静态插槽。      |
| `item-content` | `items, item`             | 内容静态插槽。      |

### Pagination (分页)

**描述:** 带有页面导航、下一页和上一页链接的分页。 **核心属性 (Props):**

| Prop                  | 默认值             | 类型                  | 描述                           |
| --------------------- | --------------- | ------------------- | ---------------------------- |
| `total`               | `0`             | `number`            | 列表中的总项目数。                    |
| `page`                | -               | `number`            | 当前页的受控值 (`v-model:page` 绑定)。 |
| `itemsPerPage`        | `10`            | `number`            | 每页的项目数。                      |
| `siblingCount`        | `2`             | `number`            | 当前页面周围显示的环绕页数。               |
| `showEdges`           | `false`         | `boolean`           | 当 `true` 时，始终显示第一页、最后一页和省略号。 |
| `size`                | `sm`            | `string`            | 调整整个分页的大小。                   |
| `pagination-selected` | `solid-primary` | `{variant}-{color}` | 选中页面的颜色。                     |

**插槽 (Slots):**

| Name        | Props        | 描述          |
| ----------- | ------------ | ----------- |
| `first`     | -            | 自定义第一页导航按钮。 |
| `list-item` | `item, page` | 自定义分页列表项组件。 |
| `ellipsis`  | -            | 自定义省略号指示器。  |

## 叠加与浮层 (Overlays & Popups)

### Alert Dialog (警告对话框)

**描述:** 一个模态对话框，用重要内容打断用户并期望响应。 **核心属性 (Props):**

| Prop                 | 默认值                            | 类型                       | 描述                                 |
| -------------------- | ------------------------------ | ------------------------ | ---------------------------------- |
| `title`              | -                              | `string`                 | 对话框的标题。                            |
| `description`        | -                              | `string`                 | 对话框的描述。                            |
| `open`               | -                              | `boolean`                | 受控的开放状态 (`v-model:open` 绑定)。       |
| `preventClose`       | -                              | `boolean`                | 如果为 `true`，对话框不会因点击叠加层或按 Esc 键而关闭。 |
| `_alertDialogAction` | `{ btn: 'soft-primary', ... }` | `AlertDialogActionProps` | 行动按钮的属性。                           |

**事件 (Events):**

| Event Name | 描述                      |
| ---------- | ----------------------- |
| `@action`  | 点击行动按钮时触发（通常用于确认破坏性操作）。 |
| `@cancel`  | 点击取消按钮或对话框被关闭时触发。       |

**插槽 (Slots):**

| Name      | Props  | 描述                       |
| --------- | ------ | ------------------------ |
| `default` | -      | 允许使用子组件进行高级自定义渲染，替换默认结构。 |
| `trigger` | `open` | 用于打开对话框的触发按钮。            |
| `action`  | -      | 行动按钮的自定义内容。              |
| `header`  | -      | 包含标题和描述的页眉部分。            |

### Dialog (对话框)

**描述:** 一个覆盖在主窗口或另一个对话框窗口上的窗口，使下方内容处于惰性状态。 **核心属性 (Props):**

| Prop           | 默认值     | 类型        | 描述                                 |
| -------------- | ------- | --------- | ---------------------------------- |
| `title`        | -       | `string`  | 对话框的标题。                            |
| `open`         | -       | `boolean` | 受控的开放状态 (`v-model:open` 绑定)。       |
| `modal`        | `true`  | `boolean` | 对话框的模态性。                           |
| `scrollable`   | `false` | `boolean` | 如果为 `true`，对话框将具有可滚动主体。            |
| `preventClose` | -       | `boolean` | 如果为 `true`，对话框不会因点击叠加层或按 Esc 键而关闭。 |

**插槽 (Slots):**

| Name      | Props | 描述         |
| --------- | ----- | ---------- |
| `default` | -     | 主体插槽。      |
| `content` | -     | 整个内容插槽。    |
| `header`  | -     | 包含标题和描述插槽。 |
| `footer`  | -     | 页脚。        |

### Drawer (抽屉)

**描述:** 一个用于 Vue 的抽屉组件。 **核心属性 (Props):**

| Prop                    | 默认值      | 类型        | 描述                                    |
| ----------------------- | -------- | --------- | ------------------------------------- |
| `open`                  | -        | `boolean` | 受控开放状态 (`v-model:open` 绑定)。           |
| `direction`             | `bottom` | `top \    | bottom \                              |
| `snapPoints`            | -        | `number \ | string`                               |
| `dismissible`           | `true`   | `boolean` | 为 `false` 时，拖动、点击外部、按 Esc 键等操作不会关闭抽屉。 |
| `shouldScaleBackground` | -        | `boolean` | 确定对话框打开时背景内容是否应该缩小。                   |

**插槽 (Slots):**

| Name      | Props  | 描述                |
| --------- | ------ | ----------------- |
| `default` | -      | 允许使用子组件进行高级自定义渲染。 |
| `trigger` | `open` | 用于打开抽屉的触发按钮。      |
| `body`    | -      | 抽屉主体中显示的内容。       |
| `footer`  | -      | 抽屉页脚的自定义内容。       |

### Dropdown Menu (下拉菜单)

**描述:** 显示一个菜单（例如一组操作或功能），由按钮触发。 **核心属性 (Props):**

| Prop            | 默认值           | 类型                        | 描述                           |
| --------------- | ------------- | ------------------------- | ---------------------------- |
| `items`         | `[]`          | `DropdownMenuItemProps[]` | 在下拉菜单中显示的项。                  |
| `open`          | `false`       | `boolean`                 | 受控的开放状态 (`v-model:open` 绑定)。 |
| `inset`         | `false`       | `boolean`                 | 设置下拉菜单为内嵌。                   |
| `size`          | `sm`          | `string`                  | 调整组件的整体大小。                   |
| `dropdown-menu` | `solid-white` | `{variant}-{color}`       | 更改下拉菜单的颜色。                   |

**插槽 (Slots):**

| Name      | Props   | 描述     |
| --------- | ------- | ------ |
| `trigger` | -       | 触发器插槽。 |
| `item`    | `item`  | 项目插槽。  |
| `content` | `items` | 内容插槽。  |
| `group`   | `items` | 组插槽。   |

### Hover Card (悬停卡片)

**描述:** 供视力正常的用者预览链接后面的内容。 **核心属性 (Props):**

| Prop         | 默认值            | 类型                  | 描述                           |
| ------------ | -------------- | ------------------- | ---------------------------- |
| `open`       | -              | `boolean`           | 受控的开放状态 (`v-model:open` 绑定)。 |
| `openDelay`  | `700`          | `number`            | 鼠标进入触发器到卡片打开的持续时间。           |
| `closeDelay` | `300`          | `number`            | 鼠标离开触发器或内容到卡片关闭的持续时间。        |
| `hovercard`  | `outline-gray` | `{variant}-{color}` | 设置悬停卡片的变体和颜色。                |
| `arrow`      | `false`        | `boolean`           | 设置与悬停卡片一起渲染的箭头。              |

**插槽 (Slots):**

| Name      | Props  | 描述                |
| --------- | ------ | ----------------- |
| `default` | -      | 允许使用子组件进行高级自定义渲染。 |
| `trigger` | `open` | 触发器插槽。            |
| `content` | -      | 用于显示卡片全部内容的内容插槽。  |

### Popover (气泡框)

**描述:** 在门户中显示由按钮触发的丰富内容。 **核心属性 (Props):**

| Prop                         | 默认值     | 类型                         | 描述               |
| ---------------------------- | ------- | -------------------------- | ---------------- |
| `open`                       | `false` | `boolean`                  | 受控的开放状态。         |
| `modal`                      | `false` | `boolean`                  | 气泡框的模态性。         |
| `_popoverContent.side`       | `top`   | `top, right, bottom, left` | 气泡框打开时首选的渲染侧。    |
| `_popoverContent.sideOffset` | -       | `number`                   | 与触发器的距离（以像素为单位）。 |

**插槽 (Slots):**

| Name      | Props  | 描述     |
| --------- | ------ | ------ |
| `trigger` | `open` | 按钮触发器。 |
| `default` | -      | 气泡框内容。 |

### Sheet (侧边抽屉)

**描述:** 扩展 `Dialog` 组件以显示与屏幕主内容互补的内容。 **核心属性 (Props):**

| Prop            | 默认值     | 类型        | 描述                                          |
| --------------- | ------- | --------- | ------------------------------------------- |
| `title`         | -       | `string`  | 抽屉的标题。                                      |
| `open`          | -       | `boolean` | 受控的开放状态 (`v-model:open` 绑定)。                |
| `sheet`         | `right` | `string`  | 抽屉出现的一侧 (`top`, `right`, `bottom`, `left`)。 |
| `preventClose`  | -       | `boolean` | 如果为 `true`，抽屉不会因点击叠加层或按 Esc 键而关闭。           |
| `_sheetContent` | -       | `object`  | 内容属性。                                       |

**插槽 (Slots):**

| Name      | Props | 描述         |
| --------- | ----- | ---------- |
| `default` | -     | 主体插槽。      |
| `content` | -     | 整个内容插槽。    |
| `header`  | -     | 包含标题和描述插槽。 |
| `footer`  | -     | 页脚。        |

### Toast (通知/吐司)

**描述:** 临时显示的简洁消息。 **核心属性 (Props):**

| Prop                      | 默认值            | 类型                  | 描述             |
| ------------------------- | -------------- | ------------------- | -------------- |
| `title`                   | -              | `string`            | 通知标题。          |
| `description`             | -              | `string`            | 通知描述。          |
| `closable`                | `true`         | `boolean`           | 显示关闭按钮。        |
| `actions`                 | `[]`           | `Action[]`          | 操作数组。          |
| `leading`                 | -              | `string`            | 通知的前置图标。       |
| `toast`                   | `outline-gray` | `{variant}-{color}` | 设置通知的变体和颜色。    |
| `progress`                | `primary`      | `{color}`           | 设置进度条颜色。       |
| `Provider prop: duration` | `4000`         | `number`            | 设置通知的持续时间（毫秒）。 |

**插槽 (Slots):**

| Name        | Props | 描述      |
| ----------- | ----- | ------- |
| `default`   | -     | 触发器插槽。  |
| `actions`   | -     | 操作插槽。   |
| `title`     | -     | 标题插槽。   |
| `closeIcon` | -     | 关闭图标插槽。 |

### Tooltip (工具提示)

**描述:** 当元素接收到键盘焦点或鼠标悬停时，显示与该元素相关的信息的弹出窗口。 **核心属性 (Props):**

| Prop                 | 默认值     | 类型                      | 描述           |
| -------------------- | ------- | ----------------------- | ------------ |
| `content`            | -       | `string`                | 设置工具提示内容。    |
| `open`               | `false` | `boolean`               | 受控的开放状态。     |
| `disabled`           | -       | `boolean`               | 禁用工具提示。      |
| `tooltip`            | `black` | `string`                | 设置工具提示颜色。    |
| `size`               | `xs`    | `string`                | 设置工具提示的常规大小。 |
| `Content prop: side` | `top`   | `top right bottom left` | 气泡框首选的渲染侧。   |

**插槽 (Slots):**

| Name      | Props | 描述     |
| --------- | ----- | ------ |
| `default` | -     | 触发器插槽。 |
| `content` | -     | 内容插槽。  |

## 复杂与高级组件 (Complex & Advanced)

### Resizable (可调整大小面板)

**描述:** 具有键盘支持的可访问可调整大小面板组和布局。

- **注意:** 该组件使用子组件 (`ResizablePanelGroup`, `ResizablePanel`, `ResizableHandle`) 配合使用。 **核心属性 (Panel Group Props):**

| Prop         | 默认值    | 类型                     | 描述                                  |
| ------------ | ------ | ---------------------- | ----------------------------------- |
| `direction*` | -      | `vertical, horizontal` | 分组方向（必需属性）。                         |
| `autoSaveId` | `null` | `string, null`         | 用于通过 `localStorage` 自动保存分组排列的唯一 ID。 |

**核心属性 (Handle Props):**

| Prop              | 默认值                      | 类型                  | 描述           |
| ----------------- | ------------------------ | ------------------- | ------------ |
| `icon`            | `i-lucide-grip-vertical` | `boolean, string`   | 向调整大小手柄添加图标。 |
| `resizableHandle` | `solid-gray`             | `{variant}-{color}` | 自定义手柄颜色。     |
| `disabled`        | -                        | `boolean`           | 禁用拖动手柄。      |

**核心属性 (Panel Props):**

| Prop          | 默认值  | 类型        | 描述                           |
| ------------- | ---- | --------- | ---------------------------- |
| `defaultSize` | -    | `number`  | 面板的初始大小 (1-100)。             |
| `minSize`     | `10` | `number`  | 面板的最小允许大小 (1-100)。           |
| `collapsible` | -    | `boolean` | 当调整大小超出 `minSize` 时面板是否应该折叠。 |

### Scroll Area (滚动区域)

**描述:** 增强原生滚动功能，以实现自定义的跨浏览器样式。 **核心属性 (Props):**

| Prop              | 默认值        | 类型                            | 描述               |
| ----------------- | ---------- | ----------------------------- | ---------------- |
| `type`            | `hover`    | `scroll, always, auto, hover` | 描述滚动条可见性的性质。     |
| `scrollHideDelay` | `600`      | `number`                      | 滚动条隐藏前的持续时间（毫秒）。 |
| `orientation`     | `vertical` | `vertical, horizontal`        | 滚动条的方向。          |
| `scrollArea`      | `gray`     | `string`                      | 滚动区域的颜色。         |
| `rounded`         | `full`     | `string`                      | 滚动区域的圆角。         |

**插槽 (Slots):**

| Name      | Props | 描述                    |
| --------- | ----- | --------------------- |
| `default` | -     | `ScrollArea` 组件的默认插槽。 |

### Sidebar (侧边栏)

**描述:** 一个可组合、可主题化和可定制的侧边栏组件。

- **结构:** 由 `SidebarProvider`、`Sidebar`、`SidebarHeader`、`SidebarFooter`、`SidebarContent`、`SidebarGroup` 和 `SidebarTrigger` 组成。 **核心属性 (Props):**

| Prop          | 默认值         | 类型                         | 描述                |
| ------------- | ----------- | -------------------------- | ----------------- |
| `sheet`       | `left`      | `left, right`              | 抽屉的一侧。            |
| `sidebar`     | `sidebar`   | `sidebar, floating, inset` | 侧边栏的变体。           |
| `collapsible` | `offcanvas` | `offcanvas, icon, none`    | 可折叠行为。            |
| `rail`        | `true`      | `boolean`                  | 是否显示用于调整大小的侧边栏轨道。 |

**定制属性 (Customization Props):**

| Name              | Type                   | Description |
| ----------------- | ---------------------- | ----------- |
| `_sidebarContent` | `NSidebarContentProps` | 内容组件的属性。    |
| `_sidebarHeader`  | `NSidebarHeaderProps`  | 页眉组件的属性。    |

**插槽 (Slots):**

| Name      | Props | 描述                    |
| --------- | ----- | --------------------- |
| `default` | -     | 侧边栏的主要内容（包括页眉、内容和页脚）。 |
| `header`  | -     | 页眉内容。                 |
| `content` | -     | 侧边栏的主要内容。             |
| `footer`  | -     | 页脚内容。                 |

**可组合项 (Composables):**

- `useSidebar()`: 提供对侧边栏状态的响应式访问。

### Stepper (步骤条)

**描述:** 一组用于指示多步骤流程进度的步骤。 **核心属性 (Props):**

| Prop          | 默认值             | 类型                  | 描述                         |
| ------------- | --------------- | ------------------- | -------------------------- |
| `items`       | `[]`            | `T`                 | 传递给步骤条的步骤数组。               |
| `modelValue`  | -               | `number`            | 要激活的步骤的受控值 (`v-model` 绑定)。 |
| `linear`      | `true`          | `boolean`           | 步骤是否必须按顺序完成。               |
| `orientation` | `horizontal`    | `horizontal \       | vertical`                  |
| `stepper`     | `solid-primary` | `{variant}-{color}` | 设置步骤条的变体和颜色。               |
| `size`        | `md`            | `string`            | 调整组件的整体大小。                 |

**暴露的方法 (Exposed):**

| Name       | 类型                       | 描述            |
| ---------- | ------------------------ | ------------- |
| `goToStep` | `(step: number) => void` | 导航到指定索引的步骤。   |
| `nextStep` | `() => void`             | 移动到下一个步骤。     |
| `hasPrev`  | `() => boolean`          | 检查是否有前一个步骤可用。 |

**插槽 (Slots):**

| Name      | Props                         | 描述              |
| --------- | ----------------------------- | --------------- |
| `default` | `modelValue, totalSteps, ...` | 默认插槽，覆盖整个步骤条内容。 |
| `item`    | `item, step`                  | 自定义每个单独的步骤。     |
| `content` | `item`                        | 动态渲染活动步骤的内容。    |
| `trigger` | `item`                        | 覆盖每个步骤的可点击触发区域。 |

### Table (表格/数据网格)

**描述:** 使用 Tanstack 构建的强大、响应式表格和数据网格。 **核心属性 (Props):**

| Prop                      | 默认值                            | 类型                              | 描述                     |
| ------------------------- | ------------------------------ | ------------------------------- | ---------------------- |
| `columns`                 | `[]`                           | `array`                         | 表格列。                   |
| `data`                    | `[]`                           | `array`                         | 表格数据。                  |
| `rowSelection`            | -                              | `object`                        | 选中行状态 (`v-model` 绑定)。  |
| `enableMultiRowSelection` | `true`                         | `boolean`                       | 启用多行选择。                |
| `empty-text`              | `No results.`                  | `string`                        | 空文本。                   |
| `loading`                 | `false`                        | `boolean`                       | 加载状态。                  |
| `pagination`              | `{pageIndex: 0, pageSize: 10}` | `object`                        | 分页状态 (`v-model` 绑定)。   |
| `sorting`                 | -                              | `array`                         | 排序状态 (`v-model` 绑定)。   |
| `globalFilter`            | -                              | `string`                        | 全局过滤状态 (`v-model` 绑定)。 |
| `columnPinning`           | -                              | `{ left: array, right: array }` | 列固定状态 (`v-model` 绑定)。  |

**事件 (Events):**

| Event Name | 描述      |
| ---------- | ------- |
| `@select`  | 选择行时触发。 |
| `@row`     | 点击行时触发。 |

**插槽 (Slots):**

| Name              | Props    | 描述      |
| ----------------- | -------- | ------- |
| `{column}-header` | `column` | 列标题插槽。  |
| `{column}-cell`   | `cell`   | 列单元格插槽。 |
| `body`            | `table`  | 主体插槽。   |
| `expanded`        | `row`    | 展开插槽。   |
| `empty`           | -        | 空状态插槽。  |

### Tabs (选项卡)

**描述:** 一组分层的、一次只显示一个内容的区域（称为选项卡面板）。 **核心属性 (Props):**

| Prop          | 默认值          | 类型        | 描述                    |
| ------------- | ------------ | --------- | --------------------- |
| `content`     | -            | `string`  | 设置选项卡内容。              |
| `disabled`    | -            | `boolean` | 设置为禁用选项卡。             |
| `tabs-active` | `soft-black` | `string`  | 通过设置变体和颜色来控制活动选项卡的外观。 |
| `size`        | `sm`         | `string`  | 设置选项卡大小。              |

**插槽 (Slots):**

| Name      | Props   | 描述                       |
| --------- | ------- | ------------------------ |
| `default` | -       | 允许使用子组件进行高级自定义渲染，替换默认结构。 |
| `list`    | `items` | 选项卡触发器/按钮的容器。            |
| `trigger` | `value` | 可点击的选项卡按钮。               |
| `content` | `value` | 选项卡内容面板。                 |

## 表单组 (Form Group, Deprecated)

### Form Group (表单组 - 废弃)

**描述:** 各种表单组件（如 Input、Textarea、Select 等）的通用包装器。

- **🚨 注意:** 此组件已**废弃**，将在下一主要版本中移除。请使用 **Form** 组件代替。 **核心属性 (Props):**

| Prop          | 默认值     | 类型                                         | 描述          |
| ------------- | ------- | ------------------------------------------ | ----------- |
| `label`       | -       | `string`                                   | 添加标签。       |
| `required`    | `false` | `boolean`                                  | 在标签上添加 `*`。 |
| `description` | -       | `string`                                   | 添加描述。       |
| `message`     | -       | `string`                                   | 设置消息。       |
| `status`      | -       | `info, success, warning, error, undefined` | 设置状态。       |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 默认插槽。 |
| `label`   | -     | 标签插槽。 |
| `message` | -     | 消息插槽。 |

### Form (表单字段)

**描述:** 使用 VeeValidate 和 Zod 构建表单。 **核心属性 (Props):**

| Prop          | 默认值 | 类型                                         | 描述         |
| ------------- | --- | ------------------------------------------ | ---------- |
| `label`       | -   | `string`                                   | 添加标签。      |
| `name`        | -   | `string`                                   | 添加名称。      |
| `description` | -   | `string`                                   | 添加描述。      |
| `hint`        | -   | `string`                                   | 添加提示。      |
| `status`      | -   | `info, success, warning, error, undefined` | 设置表单字段的状态。 |

**插槽 (Slots):**

| Name      | Props | 描述    |
| --------- | ----- | ----- |
| `default` | -     | 默认插槽。 |
| `label`   | -     | 标签插槽。 |
| `message` | -     | 消息插槽。 |
