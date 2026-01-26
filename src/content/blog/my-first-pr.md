---
title: '首次开源贡献：修复 Una UI 的 Checkbox 颜色验证 Bug'
description: '记录我在 Una UI 项目中的第一次 Pull Request (#553)，修复了 Checkbox 组件无法识别自定义颜色别名的问题。'
pubDate: 'Dec 21 2025'
tags: ["Archive"]
---

今天达成了一个小小的里程碑：给 [Una UI](https://github.com/una-ui/una-ui) 提交了我的第一个 Pull Request！

[Una UI](https://github.com/una-ui/una-ui) 是一个基于 Nuxt 和 UnoCSS 的原子化 UI 框架。虽然我还不太熟悉它的代码库，但我通过 GitHub CLI 发现了一个已经在那里躺了半年的 Bug Issue [#441](https://github.com/una-ui/una-ui/issues/441)。

## 🐛 发现问题

Issue 的描述是：`fix(Checkbox): prevent returning a variant if the provided value is not a color`。

简单来说，当我们在使用 Checkbox 组件时，Unocss 的动态规则会尝试解析类似 `checkbox-wrapper` 这样的类名。如果解析逻辑不够严谨，它可能会错误地生成样式。

经过深入代码可以发现（`packages/preset/src/_shortcuts/checkbox.ts`），原有的逻辑是直接调用 UnoCSS 的 `parseColor` 函数：

```typescript
export const dynamicCheckbox = [
  [/^checkbox-(.*)$/, ([, body]: string[], { theme }: RuleContext<Theme>) => {
    const color = parseColor(body, theme)
    // ... 验证逻辑
  }],
]
```

## 🔍 深入分析

为了验证问题，我编写了一个简单的测试脚本。结果非常有趣：

```javascript
// 测试结果
standard_red: VALID   ✅ (标准 CSS 颜色)
wrapper:      INVALID ❌ (静态类名后缀，正确被忽略)
primary:      INVALID ❌ (Wait, what?)
```

我发现 `parseColor` 使用的是 `preset-mini` 的默认主题，因此它无法识别 Una UI 自定义的语义化颜色别名（如 `primary`、`error`、`success` 等）。这会导致即使是默认样式 `checkbox-primary` 在经过动态规则检查时也会被判定为无效！

## 🛠️ 解决方案

找到病根后，修复方案就很清晰了。我需要让动态规则 "认识" 这些自定义别名。

1.  定义一个允许的颜色别名白名单。
2.  在调用 `parseColor` 之前优先检查这个白名单。

```typescript
// Una UI custom color aliases that should be treated as valid colors
const unaColorAliases = new Set(['primary', 'error', 'warning', 'success', 'info', 'gray'])

export const dynamicCheckbox = [
  [/^checkbox-(.*)$/, ([, body]: string[], { theme }: RuleContext<Theme>) => {
    // Check if it's a Una UI custom color alias
    if (unaColorAliases.has(body))
      return `n-${body}-600 dark:n-${body}-500`

    // Check if it's a valid CSS color
    const color = parseColor(body, theme)
    // ...
  }],
]
```

## 🧪 编写测试

为了确保修复有效且不引入回归，我专门编写了单元测试 `checkbox.test.ts`：

```typescript
it('should recognize Una UI custom color aliases', () => {
  expect(isValidCheckboxColor('primary')).toBe(true)
  expect(isValidCheckboxColor('error')).toBe(true)
  expect(isValidCheckboxColor('info')).toBe(true)
})

it('should reject non-color values', () => {
  expect(isValidCheckboxColor('wrapper')).toBe(false)
  expect(isValidCheckboxColor('pruple')).toBe(false) // 拼写错误也能拦截
})
```

运行测试：`All 6 tests passed!` 🟢

## 🚀 提交 PR

最后，利用 `gh` 命令行工具一键提交：

```bash
gh pr create --title "fix(checkbox): add support for Una UI color aliases in dynamic rule"
```

我的 PR [#553](https://github.com/una-ui/una-ui/pull/553) 现已上线！

---

**总结**：这次经历让我意识到，参与开源并不一定需要重写核心架构。从修复一个小小的 Bug 开始，编写测试验证想法，这也是一种非常棒的贡献方式！
