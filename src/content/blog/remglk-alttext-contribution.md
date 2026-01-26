---
pubDate: "Jan 24 2026"
title: "为 RemGlk-rs 贡献：添加图像 Alt Text 支持"
description: "介绍我如何通过实现图像 alt text 支持为 RemGlk Rust 移植项目做贡献，以提高无障碍性"
tags: ["open-source", "contribution", "rust", "accessibility", "remglk", "interactive-fiction"]
---

今天我向 [RemGlk-rs](https://github.com/curiousdannii/remglk-rs) 项目做了另一次开源贡献，这是 RemGlk 的 Rust 移植版 - 一个用于互动小说的库。我实现了图像 alt text 支持，如 [issue #4](https://github.com/curiousdannii/remglk-rs/issues/4) 中所要求的。

## 问题

问题标题为 "Support image alt texts"，没有描述。检查代码库后，我发现底层 C 库（RemGlk）已经在其结构中提供了 alt text 信息，但 Rust 移植版没有正确提取并通过 Glk 接口传播此信息。

## 解决方案

我通过以下方式实现了缺失的功能：

1. **扩展 `ImageInfo` 结构体** 以包含 `alttext` 字段

2. **修改 `get_image_info` 函数** 以从 C 字符串中提取 alt text 并将其作为 `Option<String>` 返回

3. **更新 `draw_image` 函数** 以将 alt text 从 `ImageInfo` 传递给 `BufferWindowImage`

## 技术实现

实现涉及小心处理 FFI（外部函数接口）边界：

```rust
// 从 C 字符串中提取 alt text
let alttext = if info.alttext.is_null() {
    None
} else {
    unsafe {
        let c_str = std::ffi::CStr::from_ptr(info.alttext);
        c_str.to_str()
            .ok()
            .map(|s| s.to_string())
    }
};
```

这段代码安全地从 C 字符串指针中提取 alt text，适当地处理空指针，并将 C 字符串转换为 Rust 字符串，同时正确管理不安全边界。

## 主要优势

1. **改进无障碍性**：图像现在通过 Glk 接口正确携带其 alt text 描述，使互动小说应用程序更具无障碍性。

2. **完整性**：Rust 移植版现在充分利用底层 C 库中已有的 alt text 信息。

3. **适当的错误处理**：实现使用 `Option<String>` 来优雅地处理未提供 alt text 的情况。

4. **FFI 安全性**：正确处理 C 和 Rust 代码之间的边界，具有适当的空指针检查。

## 结论

此贡献通过确保图像 alt text 通过 Glk 接口正确传播，增强了 RemGlk-rs 库的无障碍性。实现连接了底层 C 库中已有的 alt text 信息到 Rust 端，使其可供依赖此库的应用程序使用。

拉取请求已提交，可在 [curiousdannii/remglk-rs#5](https://github.com/curiousdannii/remglk-rs/pull/5) 查看。这解决了 [curiousdannii/remglk-rs#4](https://github.com/curiousdannii/remglk-rs/issues/4) 问题，并改进了库的无障碍性。