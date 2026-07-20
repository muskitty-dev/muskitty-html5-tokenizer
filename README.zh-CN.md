# muskitty-html5-tokenizer

[English](README.md) | [简体中文](README.zh-CN.md)

[![crates.io](https://img.shields.io/crates/v/muskitty-html5-tokenizer.svg)](https://crates.io/crates/muskitty-html5-tokenizer)
[![Documentation](https://docs.rs/muskitty-html5-tokenizer/badge.svg)](https://docs.rs/muskitty-html5-tokenizer)
[![License](https://img.shields.io/crates/l/muskitty-html5-tokenizer.svg)](https://github.com/muskitty-dev/muskitty-html5-tokenizer/blob/main/LICENSE)
[![CI](https://github.com/muskitty-dev/muskitty-html5-tokenizer/actions/workflows/ci.yml/badge.svg)](https://github.com/muskitty-dev/muskitty-html5-tokenizer/actions/workflows/ci.yml)

一个从零开始用纯 Rust 实现的 HTML5 分词器，遵循
[WHATWG HTML Living Standard §13.2.5](https://html.spec.whatwg.org/multipage/parsing.html#tokenization)
规范，零运行时依赖。

属于 [MusKitty](https://github.com/muskitty-dev) 浏览器引擎项目的一部分。

## 状态

| 组件 | 规范覆盖率 | 测试通过率 |
|------|------------|------------|
| **Tokenizer** (§13.2.5) | 85/85 states | [99.8%](https://github.com/html5lib/html5lib-tests) (7022/7036) |

- 零 `unsafe` 代码
- 零 C/C++ 依赖
- 零运行时依赖
- 仅使用 Rust 稳定版工具链
- 以 html5lib-tests 测试套件作为基准

## 安装

在你的 `Cargo.toml` 中添加：

```toml
[dependencies]
muskitty-html5-tokenizer = "0.1.0"
```

或运行：

```bash
cargo add muskitty-html5-tokenizer
```

## 快速开始

```rust
use muskitty_html5_tokenizer::{HtmlTokenizer, Token, Tokenizer};

let mut t = HtmlTokenizer::new("<p>hello</p>");
while let Some(token) = t.next_token() {
    // 处理 token
}
```

## 架构

```
muskitty-html5-tokenizer/
  src/
    types.rs          Token, TagToken, DoctypeToken, State definitions
    trait_def.rs      Tokenizer trait (supports reentrancy)
    impls.rs          HtmlTokenizer — 85-state machine (~5900 lines)
    entities.rs       2231 WHATWG named character references
    lib.rs            Public API: HtmlTokenizer + Tokenizer trait + types
```

### 什么是分词器？

HTML5 分词器是一个确定性状态机（§13.2.5），它消费 Unicode 码点并产生
token（Doctype、Tag、Comment、Character、EOF、ProcessingInstruction）。
这些 token 随后会被树构造阶段消费，用以构建 DOM。

### 可重入设计

根据 §13.2.1，分词器是可重入的：树构造阶段可以暂停解析（例如为了执行
`<script>`），然后再恢复分词器。`Tokenizer` trait 暴露了
`set_state`、`set_appropriate_end_tag_name` 和 `set_foreign_content`
方法，使树构造器可以切换内容模型（例如进入 `<title>` 时从 Data 切换到
RCDATA）。

## 构建

```bash
cargo check                               # Workspace check
cargo check -p muskitty-html5-tokenizer   # Tokenizer crate only
```

## 测试

```bash
# Unit tests (145 tests)
cargo test -p muskitty-html5-tokenizer --lib

# html5lib tokenizer suite (7036 tests)
cargo test --test html5lib_tokenizer -- --nocapture

# All tests
cargo test
```

### 测试夹具

测试使用 [html5lib-tests](https://github.com/html5lib/html5lib-tests) 套件：

- `tests/data/tokenizer/*.test` — 14 个分词器夹具文件

## 设计原则

1. **以 WHATWG 为准** — 实现严格遵循规范。WPT 和 Chromium 仅作为次要参考。
2. **对齐规范，而非对齐测试** — 测试用于验证代码；除非规范证明测试有误，否则绝不修改代码以通过测试。
3. **零运行时依赖** — 仅有 `serde_json` 作为测试夹具的 dev-dependency。
4. **零 unsafe** — 纯 safe Rust。
5. **外科手术式修改** — 每次 diff 都只做任务所需的最小改动。

## 规范参考

本实现参考了：

- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/) — 主要权威来源
  - §13.2.5: Tokenization
- [html5lib-tests](https://github.com/html5lib/html5lib-tests) — 测试基准

## 许可证

基于 Apache License, Version 2.0 授权。详见 [LICENSE](LICENSE)。

Copyright 2026 MusCat / MusKitty Bit-Torch Community
