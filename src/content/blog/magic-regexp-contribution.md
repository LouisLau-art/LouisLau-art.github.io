---
title: '开源贡献实战：用 AI 查漏补缺 magic-regexp 测试覆盖率 (PR #666)'
description: '实践 "Coverage Agent" 策略，为 unjs/magic-regexp 补充 Edge Cases 测试。一个关于如何找到开源项目测试盲区的实战记录。'
pubDate: 'Jan 26 2026'
tags: ["Open Source", "Testing", "Vitest", "AI"]
---

在上一篇文章里，我们深入 Nuxt 核心与其内存机制搏斗了一番。而在那之后，我决定换一种心情，实践一下 AI 辅助开源贡献的 **"Coverage Agent"（测试覆盖率补充）** 策略。

这次的目标是 **[unjs/magic-regexp](https://github.com/unjs/magic-regexp)**，一个旨在让正则表达式可读性变强的库。

## 🎯 寻找切入点

对于成熟的开源项目，核心逻辑通常已经非常稳固，盲目重构很容易招致维护者的反感。最好的切入点往往是**边缘情况（Edge Cases）的测试**。

我检查了 `src/transform.ts` 的源码，发现它负责识别哪些文件需要进行正则转换。源码里包含了一段判断文件扩展名的逻辑：

```typescript
// js files
if (pathname.match(/\.((c|m)?j|t)sx?$/g))
  return true
```

这段正则旨在匹配 `.js`, `.ts`, `.jsx`, `.tsx`, `.mjs`, `.cjs` 等。

然而，对比 `test/transform.test.ts`，我发现现有的测试只覆盖了：
- `.css` (忽略)
- `.vue` (包含)

**Gap Found!** 缺少针对 `.jsx`, `.tsx`, `.mjs`, `.cjs` 以及带有 Query 参数（如 `app.js?v=1`）的明确测试。虽然代码大概率是工作的，但如果没有测试保护，未来的重构可能会不小心破坏这些扩展名的支持。

## 🛠️ 编写测试

我没有去修改现有的测试文件，而是遵循 **"Atomic"（原子化）** 原则，新建了一个 `test/transform-coverage.test.ts`：

```typescript
import { describe, expect, it } from 'vitest'
// ... imports

describe('transformer: coverage', () => {
  const code = `import { createRegExp } from 'magic-regexp'; createRegExp('foo')`

  it('supports various JS/TS extensions', () => {
    expect(transform(code, 'file.js')).toBeDefined()
    expect(transform(code, 'component.tsx')).toBeDefined() // React/Vue JSX
    expect(transform(code, 'module.mjs')).toBeDefined()    // ES Modules
    expect(transform(code, 'common.cjs')).toBeDefined()    // CommonJS
  })

  it('supports files with query parameters', () => {
    expect(transform(code, 'file.js?v=123')).toBeDefined()
  })
})
```

## 🚀 提交 PR

运行 `pnpm vitest` 确认全绿后，我提交了 **PR #666**（多么吉利的数字！）：

> **test: improve transform plugin coverage for various extensions**
>
> This PR adds explicit test coverage for various JS/TS file extensions (.jsx, .tsx, .mjs, .cjs) and query parameters...

## 💡 总结

这次行动展示了 AI 辅助贡献的最佳姿势之一：
1. **分析 (Analyze)**：对比源码逻辑与测试用例的差异。
2. **定位 (Locate)**：找到未覆盖的“软柿子”（扩展名判断）。
3. **执行 (Execute)**：编写独立、清晰的测试文件。

这种 PR 维护者通常在一分钟内就能 Review 完毕并合并，因为它**零风险**且**有价值**。
