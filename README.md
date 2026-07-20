# muskitty-html5-tokenizer

[English](README.md) | [简体中文](README.zh-CN.md)

[![crates.io](https://img.shields.io/crates/v/muskitty-html5-tokenizer.svg)](https://crates.io/crates/muskitty-html5-tokenizer)
[![Documentation](https://docs.rs/muskitty-html5-tokenizer/badge.svg)](https://docs.rs/muskitty-html5-tokenizer)
[![License](https://img.shields.io/crates/l/muskitty-html5-tokenizer.svg)](https://github.com/muskitty-dev/muskitty-html5-tokenizer/blob/main/LICENSE)
[![CI](https://github.com/muskitty-dev/muskitty-html5-tokenizer/actions/workflows/ci.yml/badge.svg)](https://github.com/muskitty-dev/muskitty-html5-tokenizer/actions/workflows/ci.yml)

A from-scratch HTML5 tokenizer written in pure Rust, implementing the
[WHATWG HTML Living Standard §13.2.5](https://html.spec.whatwg.org/multipage/parsing.html#tokenization)
with zero runtime dependencies.

Part of the [MusKitty](https://github.com/muskitty-dev) browser engine project.

## Status

| Component | Spec Coverage | Test Pass Rate |
|-----------|---------------|----------------|
| **Tokenizer** (§13.2.5) | 85/85 states | [99.8%](https://github.com/html5lib/html5lib-tests) (7022/7036) |

- Zero `unsafe` code
- Zero C/C++ dependencies
- Zero runtime dependencies
- Rust stable toolchain only
- html5lib-tests suite as ground truth

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
muskitty-html5-tokenizer = "0.1.0"
```

Or run:

```bash
cargo add muskitty-html5-tokenizer
```

## Quick Start

```rust
use muskitty_html5_tokenizer::{HtmlTokenizer, Token, Tokenizer};

let mut t = HtmlTokenizer::new("<p>hello</p>");
while let Some(token) = t.next_token() {
    // process token
}
```

## Architecture

```
muskitty-html5-tokenizer/
  src/
    types.rs          Token, TagToken, DoctypeToken, State definitions
    trait_def.rs      Tokenizer trait (supports reentrancy)
    impls.rs          HtmlTokenizer — 85-state machine (~5900 lines)
    entities.rs       2231 WHATWG named character references
    lib.rs            Public API: HtmlTokenizer + Tokenizer trait + types
```

### What is a Tokenizer?

The HTML5 tokenizer is a deterministic state machine (§13.2.5) that
consumes Unicode codepoints and emits tokens (Doctype, Tag, Comment,
Character, EOF, ProcessingInstruction). These tokens are consumed by
the tree construction stage to build the DOM.

### Reentrant Design

Per §13.2.1, the tokenizer is reentrant: the tree construction stage
may pause parsing (e.g. for `<script>` execution), then resume the
tokenizer. The `Tokenizer` trait exposes `set_state`,
`set_appropriate_end_tag_name`, and `set_foreign_content` so the tree
constructor can switch content models (Data → RCDATA when entering
`<title>`, etc).

## Building

```bash
cargo check                               # Workspace check
cargo check -p muskitty-html5-tokenizer   # Tokenizer crate only
```

## Testing

```bash
# Unit tests (145 tests)
cargo test -p muskitty-html5-tokenizer --lib

# html5lib tokenizer suite (7036 tests)
cargo test --test html5lib_tokenizer -- --nocapture

# All tests
cargo test
```

### Test Fixtures

Tests use the [html5lib-tests](https://github.com/html5lib/html5lib-tests) suite:

- `tests/data/tokenizer/*.test` — 14 tokenizer fixture files

## Design Principles

1. **WHATWG is ground truth** — Implementation follows the spec exactly. WPT and Chromium are secondary references.
2. **Spec-compliant, not test-compliant** — Tests verify the code; code is never modified to pass a test unless the spec proves the test is wrong.
3. **Zero runtime dependencies** — Only `serde_json` as a dev-dependency for test fixtures.
4. **Zero unsafe** — Pure safe Rust.
5. **Surgical changes** — Every diff is as small as the task requires.

## Spec Reference

This implementation references:

- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/) — Primary authority
  - §13.2.5: Tokenization
- [html5lib-tests](https://github.com/html5lib/html5lib-tests) — Test ground truth

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

Copyright 2026 MusCat / MusKitty Bit-Torch Community
