# Reading List

Each phase document tells you what to read and when. This document lists all sources in one place. Free sources are marked "(free)".

## Rust: core sources

| Source | Use it for | Phase |
|---|---|---|
| Steve Klabnik, Carol Nichols, and Chris Krycek, *The Rust Programming Language* (doc.rust-lang.org/book) (free) | The main book of the course. Read the online version: it is updated for each edition. | 0–6 |
| Rust by Example (doc.rust-lang.org/rust-by-example) (free) | The same subjects as the Book, with examples | 1–3 |
| The Rust standard library documentation (doc.rust-lang.org/std) (free) | Reference for all standard types, traits, and modules | All |
| The Rust Reference (doc.rust-lang.org/reference) (free) | The exact language rules | 3, 9 |
| The Cargo Book (doc.rust-lang.org/cargo) (free) | Packages, workspaces, profiles, features, publishing | 0, 4, 8 |
| Rust API Guidelines (rust-lang.github.io/api-guidelines) (free) | How a good Rust API looks | 3–5, 8 |
| Jim Blandy, Jason Orendorff, and Leonora Tindall, *Programming Rust*, 2nd edition (O'Reilly) | A deep, complete book about the language. A second view after the Book. | 2–6 |
| David Drysdale, *Effective Rust* (O'Reilly; effective-rust.com) (free online) | Rules for idiomatic Rust: types, traits, concepts, dependencies, tooling | 3–8 |
| Jon Gjengset, *Rust for Rustaceans* (No Starch Press) | Intermediate and advanced Rust: types, API design, testing, async, `unsafe` | 8–9 |

## Rust: exercises and courses

| Source | Use it for |
|---|---|
| Rustlings (github.com/rust-lang/rustlings) (free) | Small exercises with automatic checks |
| Comprehensive Rust, by Google (google.github.io/comprehensive-rust) (free) | A fast course for experienced developers |
| Luca Palmieri, 100 Exercises To Learn Rust (rust-exercises.com) (free) | Exercises from the basics to traits and concurrency |
| Exercism, Rust track (exercism.org/tracks/rust) (free) | Exercises with optional reviews from human mentors |
| Learning Rust With Entirely Too Many Linked Lists (rust-unofficial.github.io/too-many-lists) (free) | Ownership, `Box`, `Rc`, and `unsafe` through linked lists |
| Jon Gjengset, "Crust of Rust" (YouTube) (free) | Deep videos about lifetimes, iterators, smart pointers, channels, atomics |

## Rust: specific subjects

| Source | Subject | Phase |
|---|---|---|
| Command Line Applications in Rust (rust-cli.github.io/book) (free) | CLI tools | 4 |
| Ken Youens-Clark, *Command-Line Rust* (O'Reilly) | CLI tools, with many exercises | 4 |
| Luca Palmieri, "Error Handling In Rust: A Deep Dive" (lpalmieri.com) (free) | Error design | 4 |
| Mara Bos, *Rust Atomics and Locks* (O'Reilly; marabos.nl/atomics) (free online) | Concurrency, atomics, memory ordering, building locks | 6, 9 |
| Tokio tutorial (tokio.rs/tokio/tutorial) (free) | Async with Tokio | 6 |
| Asynchronous Programming in Rust (rust-lang.github.io/async-book) (free) | Async concepts | 6 |
| Alice Ryhl, "Actors with Tokio" and "Async: What is blocking?" (ryhl.io) (free) | Async design | 6 |
| Carl Fredrik Samson, *Asynchronous Programming in Rust* (Packt) | How futures and runtimes work inside | 6 |
| Luca Palmieri, *Zero To Production in Rust* | A production web service: tests, database, telemetry, configuration, deployment | 7 |
| The Rust Performance Book (nnethercote.github.io/perf-book) (free) | Performance | 8, 9 |
| The Rustonomicon (doc.rust-lang.org/nomicon) (free) | `unsafe` Rust | 9 |
| The Little Book of Rust Macros (veykril.github.io/tlborm) (free) | Declarative macros | 9 |
| The Embedded Rust Book (docs.rust-embedded.org/book) (free) | Rust on microcontrollers | 9 |

## Software design

| Source | Use it for | Phase |
|---|---|---|
| David Thomas and Andrew Hunt, *The Pragmatic Programmer*, 20th anniversary edition | General practices: DRY, orthogonality, decoupling, refactoring | 5 |
| Robert C. Martin, *Clean Architecture* | SOLID, component principles, architecture, the dependency rule | 5 |
| Robert C. Martin, *Clean Code* | Names, functions, comments. Read it together with Ousterhout for a balanced view. | 5 |
| John Ousterhout, *A Philosophy of Software Design*, 2nd edition | Complexity, deep modules, information hiding | 5 |
| Martin Fowler, *Refactoring*, 2nd edition | Refactoring steps and code smells | 5, 8 |
| Scott Wlaschin, *Domain Modeling Made Functional* | Domain modeling with types (the examples use F#) | 5 |
| Eric Evans, *Domain-Driven Design*, or Vlad Khononov, *Learning Domain-Driven Design* | Domain-driven design. Khononov is shorter and more modern. | 5 (optional) |
| Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, *Design Patterns* | The classic pattern catalog. Read it after phase 5, with the Rust Design Patterns book. | 5 (optional) |
| Rust Design Patterns (rust-unofficial.github.io/patterns) (free) | Idioms, patterns, and anti-patterns in Rust | 5 |
| Alistair Cockburn, "Hexagonal Architecture" (alistair.cockburn.us) (free) | Ports and adapters | 5 |
| Alexis King, "Parse, don't validate" (lexi-lambda.github.io) (free) | Type-driven validation | 4, 5 |
| Sandi Metz, "The Wrong Abstraction" (sandimetz.com) (free) | The cost of DRY | 5 |
| Michael Nygard, "Documenting Architecture Decisions" (cognitect.com) (free) | ADRs | 5 |
| Gary Bernhardt, "Boundaries" (a talk, destroyallsoftware.com) (free) | Functional core, imperative shell | 5 |

## Testing and professional practices

| Source | Use it for | Phase |
|---|---|---|
| Kent Beck, *Test-Driven Development: By Example* | TDD | 5 |
| Michael Feathers, *Working Effectively with Legacy Code* | Change code without tests safely | 8 |
| Titus Winters, Tom Manshreck, and Hyrum Wright, *Software Engineering at Google* (abseil.io/resources/swe-book) (free online) | Code review, testing, dependency management, large teams | 8 |
| Google, "Code Review Developer Guide" (google.github.io/eng-practices) (free) | Code review | 8 |
| Martin Kleppmann, *Designing Data-Intensive Applications* | Databases, replication, distributed systems | 9 |

## Web and security

| Source | Use it for | Phase |
|---|---|---|
| MDN Web Docs, "HTTP" (developer.mozilla.org) (free) | HTTP methods, status codes, headers | 7 |
| RFC 9457, "Problem Details for HTTP APIs" (free) | Error bodies | 7 |
| OWASP Top 10 (owasp.org) (free) | Web application security risks | 7 |

## News and community

| Source | Use it for |
|---|---|
| This Week in Rust (this-week-in-rust.org) (free) | Weekly news, articles, and calls for participation |
| The Rust Blog (blog.rust-lang.org) (free) | Release notes for each Rust version |
| The Rust users forum (users.rust-lang.org) (free) | Questions and answers |
