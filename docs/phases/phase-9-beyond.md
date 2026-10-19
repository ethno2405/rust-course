# Phase 9: Beyond

**Time:** Open-ended. Each module takes 1–4 weeks.
**Prerequisites:** Phase 8. Modules 9.2 and 9.3 need only phase 4.
**Code:** `labs/phase-9-<module>/` and `labs/ledger/`

## Goal

This phase has independent modules. Select the modules that fit your goal. Ask the mentor for a suggestion. Each module has a reading list, the subjects to learn, and a project.

| Module | Topic | Fits the goal |
|---|---|---|
| 9.1 | `unsafe` Rust and FFI | Systems programming, libraries, integration with C, C++, or Python |
| 9.2 | Macros | Library design, less boilerplate |
| 9.3 | Advanced traits and types | Library design, a deep understanding of the language |
| 9.4 | Terminal and desktop user interfaces | Tools for end users |
| 9.5 | WebAssembly and web front ends | Web development |
| 9.6 | Distributed services | Backend development |
| 9.7 | Performance and low-level optimization | High-performance systems |
| 9.8 | Embedded Rust | Microcontrollers |
| 9.9 | Open source | All goals |

## Modules

### 9.1 `unsafe` Rust and FFI

**Why:** the standard library and many crates use `unsafe` code inside safe APIs. You need it to call C libraries, to call Rust from other languages, and to write some data structures.

**Learn:**

- The five things that `unsafe` permits: dereference a raw pointer, call an `unsafe` function, access a mutable static, implement an `unsafe` trait, and access the fields of a union. Everything else stays checked.
- **Soundness:** a safe API must not permit undefined behavior for any input. The `unsafe` block is correct only if all its preconditions are true. Write them in a `// SAFETY:` comment.
- Undefined behavior in Rust: data races, dangling or misaligned pointers, invalid values (a `bool` that is 3), aliasing a `&mut T`.
- **Miri:** an interpreter that finds undefined behavior in tests. `rustup +nightly component add miri`, then `cargo +nightly miri test`.
- **FFI:** `extern "C"` functions, `#[repr(C)]` types, `CString` and `CStr`, ownership across the boundary. In edition 2024, `extern` blocks must be marked `unsafe extern`.
- Tools: `bindgen` (C headers to Rust), `cbindgen` (Rust to C headers), `cxx` (safe interoperation with C++), `PyO3` and `maturin` (Python extensions).

**Reading:** The Rustonomicon (doc.rust-lang.org/nomicon). The Book, "Advanced Features", the part about `unsafe`. Mara Bos, *Rust Atomics and Locks*, the chapters about building your own locks and channels.

**Project:** write a Python extension module with PyO3 that exposes the ledger CSV parser and the monthly report. Measure the speed against a pure Python implementation. Or: write a safe Rust wrapper around a small C library (for example `zlib` or `sqlite3` without `sqlx`), with tests under Miri for the parts that do not call C.

**Exception to the course conventions:** this module is the only place where `unsafe_code` is not forbidden. Set the lint to `deny` in the lab and use `#[allow(unsafe_code)]` only on the modules that need it.

### 9.2 Macros

**Why:** macros remove repetitive code and make small languages inside Rust. `serde`, `clap`, `tokio`, `sqlx`, and `thiserror` all use procedural macros.

**Learn:** phase 3, topic 3.12, teaches the basics of `macro_rules!`. This module goes deeper.

- **Declarative macros** (`macro_rules!`): recursive macros, token-tree munchers, internal rules, and the export of macros with `$crate`.
- **Procedural macros:** derive macros, attribute macros, and function-like macros. A separate crate with `proc-macro = true`. The crates `syn` (parse), `quote` (generate), and `proc-macro2`.
- Good error messages in macros: `syn::Error` with a span.
- `cargo expand` shows the code after macro expansion.
- When not to write a macro: a function or a generic function is easier to read, to debug, and to document.

**Reading:** The Little Book of Rust Macros (veykril.github.io/tlborm). David Tolnay, the `proc-macro-workshop` repository on GitHub (exercises with tests). The Book, "Advanced Features", the part about macros.

**Project:** write a derive macro `#[derive(Builder)]` that generates a builder for a struct (the `proc-macro-workshop` has this exercise with tests). Use it for one type in the ledger.

### 9.3 Advanced traits and types

**Learn:**

- Generic associated types (GATs), higher-ranked trait bounds (`for<'a> Fn(&'a str)`), `PhantomData`, variance (a short introduction).
- `Pin` and `Unpin`: why self-referential futures need them.
- Sealed traits (a public trait that other crates cannot implement), extension traits, marker traits.
- `impl Trait` in more positions, and the lifetime capture rules of edition 2024.
- Zero-sized types and type-level state (more type-state).

**Reading:** Jon Gjengset, *Rust for Rustaceans*, the chapters about types and designing interfaces. The Rust Reference. Cliff L. Biffle, "The Typestate Pattern in Rust" (cliffle.com).

**Project:** write a small library with a public API that uses a sealed trait, type-state, and a GAT where they help. Publish it with `cargo publish --dry-run`, and review it against the API Guidelines checklist.

### 9.4 Terminal and desktop user interfaces

**Learn:**

- **Terminal user interface (TUI):** `ratatui` with `crossterm`. The immediate-mode rendering loop, events, and application state.
- **Desktop:** `egui` (immediate mode, simple), `iced` (Elm architecture), `Slint` (declarative markup), or `Tauri` (a web front end with a Rust back end).
- The architecture of a user interface: separate the state, the update logic, and the rendering. Test the state and the update logic without the user interface.

**Project:** ledger stage 6, option A: a TUI for the ledger with a transaction list, filters, and the monthly report. It uses the core crate and the store adapter. The core does not change.

### 9.5 WebAssembly and web front ends

**Learn:**

- WebAssembly (Wasm): the `wasm32-unknown-unknown` target, `wasm-bindgen`, `wasm-pack`, and the limits (no threads by default, no direct file system).
- Rust web front-end frameworks: `Leptos`, `Dioxus`, `Yew`. Server-side rendering and hydration.
- Shared code: the domain crate compiles to Wasm if it has no I/O. Phase 5 made this possible.

**Project:** ledger stage 6, option B: a web front end for the ledger HTTP API, or a Wasm build of the core that validates transactions in the browser.

### 9.6 Distributed services

**Learn:**

- gRPC with `tonic` and Protocol Buffers.
- Message queues (for example NATS or Kafka): at-least-once delivery, idempotent consumers, the outbox pattern.
- OpenTelemetry: traces across services, metrics.
- PostgreSQL in place of SQLite: the change of the adapter only. Run it in a container for the tests.
- Containers and deployment: Docker Compose for local work, health checks, readiness, and configuration.

**Reading:** Martin Kleppmann, *Designing Data-Intensive Applications*. The `tonic` and `opentelemetry` crate documentation.

**Project:** ledger stage 6, option C: move the storage to PostgreSQL, add an import service that receives statements through a message queue, and trace a request across both services.

### 9.7 Performance and low-level optimization

**Learn:**

- The memory hierarchy and cache-friendly data layouts (a structure of arrays in place of an array of structures).
- Allocation: count allocations, arenas (`bumpalo`), small-vector optimizations, a different global allocator.
- SIMD: auto-vectorization, and `std::arch`.
- Read the assembly output (`cargo asm`, Compiler Explorer).
- Profile-guided optimization (PGO).

**Reading:** The Rust Performance Book. Denis Bakhvalov, *Performance Analysis and Tuning on Modern CPUs* (free online).

**Project:** make the ledger import of 10 million lines as fast as possible. Record every step in `docs/performance.md`, with the measurement before and after.

### 9.8 Embedded Rust

**Learn:**

- `#![no_std]`, `core` and `alloc`, the embedded HAL traits (`embedded-hal`), peripheral access crates, `probe-rs`, and `defmt` for logging.
- Async on microcontrollers with `embassy`.
- The same design principles apply: drivers depend on the `embedded-hal` traits, not on a specific chip (dependency inversion), and you test them on your PC with fake implementations.

**Reading:** The Embedded Rust Book (docs.rust-embedded.org/book). The `embassy` documentation. If you want to learn the hardware from zero, a course about embedded C first teaches the concepts below the HAL.

**Project:** a sensor driver that depends only on the `embedded-hal` traits, with tests on the PC against a fake I2C bus, and a demo on a board (for example a Raspberry Pi Pico 2 or an STM32 Nucleo board).

### 9.9 Open source

**Learn:**

- Find a project: the labels "good first issue" and "help wanted", the This Week in Rust newsletter (call for participation), and the crates that you used in the course.
- Read `CONTRIBUTING.md`. Discuss a large change in an issue before you write the code.
- Follow the conventions of the project, not your own. Keep the pull request small. Answer reviews politely and quickly.

**Project:** make one contribution to an open-source Rust project: a documentation fix, a test, or a small defect fix. A fix of an error in this course also counts (see `CONTRIBUTING.md`).

## Project: Ledger, stage 6 (optional)

See [../projects/ledger.md](../projects/ledger.md), stage 6. Select one option from modules 9.4, 9.5, or 9.6, or propose your own extension.

## Done criteria

For each module that you select:

- [ ] I did the reading and the project of the module.
- [ ] I can explain the main concepts of the module to a different developer.
- [ ] My project follows the course conventions and passes CI.

## Next

You completed the course. Keep your engineering notebook. Build an application that you want to use yourself, and ask other people to review it.
