# Rust Application Development Course

This course teaches application development with Rust. It also teaches how to design software that is easy to understand, to test, and to change. It starts at zero for Rust. It does not start at zero for programming.

## Who this course is for

- **Required:** you can write programs in at least one programming language. You know variables, functions, loops, and data structures, and you can use git and a terminal.
- **Not required:** knowledge of Rust, memory management, concurrency, or design principles. The course teaches these topics.
- **Your background changes the time for phase 1:**

  | Your background | Phase 1 time |
  |---|---|
  | You already wrote small programs in Rust | 1 week |
  | You know C, C++, Swift, Kotlin, Scala, Haskell, OCaml, F#, or another language with a strong static type system | 2 weeks |
  | You know C#, Java, Go, or TypeScript | 3 weeks |
  | You know only dynamic languages (Python, JavaScript, Ruby, PHP, and similar) | 4–6 weeks |

The AI mentor assesses your background in the first session and saves it in `learner/profile.md`. See the [main README](../README.md) for how to start. Without an AI mentor, use the table above to plan phase 1.

The course has ten phases. Each phase has topics, reading, exercises, and a part of the main project. One main project, the **personal finance ledger**, grows from phase 1 to phase 9. Side projects give more practice.

## How to use this course

You can use the course with an AI mentor or alone. With an AI mentor, the mentor follows [mentor/](mentor/) and tracks your progress in `learner/progress.md`. Alone, follow these steps:

1. Read the phase document from top to bottom before you start the phase.
2. Do the reading and the exercises in the given order.
3. Do the project stage for the phase.
4. Do a check of the "Done criteria" list at the end of the phase. Go to the next phase only when all items are complete.
5. Write your notes, compiler errors that you did not understand, and design decisions in your engineering notebook.
6. Record your progress in `learner/progress.md`. Copy it from [templates/progress-template.md](templates/progress-template.md).

The times assume 6–8 hours of work per week.

## Roadmap

| Phase | Document | Time | Main result |
|---|---|---|---|
| 0 | [Tools and workflow](phases/phase-0-tools.md) | 1 week | You can make, build, test, lint, and debug a Cargo project. |
| 1 | [Rust basics](phases/phase-1-rust-basics.md) | 1–6 weeks, depends on your background | You can write small Rust programs with structs, enums, pattern matching, collections, and tests. |
| 2 | [Ownership, borrowing, and lifetimes](phases/phase-2-ownership.md) | 2–3 weeks | You can explain and fix borrow checker errors without `clone()` everywhere. |
| 3 | [Traits, generics, closures, and iterators](phases/phase-3-traits-generics.md) | 3 weeks | You can design with traits and write data pipelines with iterators. |
| 4 | [Errors, modules, and API design](phases/phase-4-errors-apis.md) | 3 weeks | A command-line ledger with good error messages, file storage, and documentation. |
| 5 | [Software design and SOLID](phases/phase-5-software-design.md) | 4 weeks | A ledger with a hexagonal architecture that is easy to test and to extend. |
| 6 | [Concurrency and async](phases/phase-6-concurrency-async.md) | 3–4 weeks | Parallel import and concurrent HTTP requests with limits, timeouts, and shutdown. |
| 7 | [Web services and persistence](phases/phase-7-web-services.md) | 3–4 weeks | An HTTP API with a SQL database, configuration, and structured logs. |
| 8 | [Professional practices](phases/phase-8-professional-practices.md) | 3–4 weeks | Advanced tests, benchmarks, a CI pipeline, and a release. |
| 9 | [Beyond](phases/phase-9-beyond.md) | Open-ended | `unsafe` and FFI, macros, user interfaces, WebAssembly, and more. |

## Main project: personal finance ledger

The full specification is in [projects/ledger.md](projects/ledger.md).

| Stage | Phase | Features |
|---|---|---|
| 0 | 1 | Domain model in memory: money, transactions, balances. Unit tests. |
| 1 | 4 | Command-line interface, JSON file storage, CSV import, error types, documentation. |
| 2 | 5 | Cargo workspace with a hexagonal architecture. Categorization rules, budgets, reports. Two import formats. |
| 3 | 6 | Parallel import of many files. Exchange rates from an HTTP API with limits and timeouts. |
| 4 | 7 | HTTP API with axum, SQLite with sqlx, configuration, tracing. |
| 5 | 8 | Property tests, snapshot tests, benchmarks, CI pipeline, release binaries. |
| 6 | 9 | Optional: terminal user interface, web front end, or a different extension. |

You can select a different project domain with the same stages. See [mentor/project-selection.md](mentor/project-selection.md).

## Design principles in the course

The course teaches software design in every phase, not only in phase 5.

| Phase | Design content |
|---|---|
| 1 | Names, small functions, tests from the first day |
| 2 | Ownership as a design tool: who owns the data? |
| 3 | Traits as interfaces. Static and dynamic dispatch. Strategy with closures. |
| 4 | Error design, "parse, don't validate", newtypes, API guidelines, documentation |
| 5 | SOLID, DRY, KISS, YAGNI, coupling and cohesion, domain modeling, hexagonal architecture, design patterns, refactoring, test-driven development |
| 6 | Message passing and actors. Structured concurrency. |
| 7 | Layers and boundaries in a service. Configuration. Observability. Security at the boundary. |
| 8 | Test strategy, code review, versioning, dependency hygiene, architecture decision records |

## Documents

| Document | Content |
|---|---|
| [software-setup.md](software-setup.md) | Toolchain, editor, and debugger setup on Windows, macOS, and Linux |
| [reading-list.md](reading-list.md) | Books, courses, and documentation |
| [glossary.md](glossary.md) | Terms and abbreviations |
| [token-usage.md](token-usage.md) | Tips to use fewer AI tokens in the course |
| [phases/](phases/) | One document for each phase |
| [projects/](projects/) | Project specifications and task documents |
| [topics/](topics/) | Detailed explanations of specific topics |
| [mentor/](mentor/) | Instructions for the AI mentor: assessment and teaching |
| [templates/](templates/) | Templates for new documents, the learner profile, and progress |

## General advice

1. **Read the compiler error to the end.** The Rust compiler explains the problem, shows the code, and often gives the fix. Run `rustc --explain <code>` for a longer explanation.
2. **Do not fight the borrow checker with `clone()`.** A borrow error often shows a design problem: two parts of the program want to own the same data. Change the design first. Clone only when a copy is the correct meaning.
3. **Write tests from the first day.** `cargo test` is built in. A test is the fastest way to learn what code does.
4. **Type the code yourself.** Do not copy and paste examples. The errors that you make teach you the language.
5. **Keep an engineering notebook.** Write down compiler errors that surprised you, design decisions and their reasons, and questions for later.
6. **Read good code.** Read the source of the standard library (the "Source" link in the documentation) and of well-known crates, for example `ripgrep`, `serde`, and `tokio`.
7. **Use principles as tools, not as laws.** Each design principle has a cost. A small program does not need the architecture of a large system. Learn to see when a principle helps and when it adds only complexity.
