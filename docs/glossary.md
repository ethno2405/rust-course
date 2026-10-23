# Glossary

Add a term to this list when you find it for the first time.

| Term | Meaning |
|---|---|
| ADR | Architecture decision record. A short document that records one important design decision, its context, and its consequences. |
| Adapter | In a hexagonal architecture: code that connects the core to a technology. It implements a port (a database store) or calls the core (a CLI, an HTTP handler). |
| API | Application programming interface. The functions and types that one piece of software gives to another. |
| Arc | Atomically reference-counted pointer. Shared ownership across threads. |
| Async, await | A way to write concurrent code that waits for I/O without blocking a thread. An `async fn` returns a future. `.await` waits for it. |
| Backpressure | A slow consumer makes a fast producer wait, for example with a bounded channel. Prevents unlimited memory growth. |
| Blanket implementation | A trait implementation for all types that meet a bound: `impl<T: Display> MyTrait for T`. |
| Borrow checker | The part of the compiler that checks the ownership and borrowing rules. |
| Borrowing | Use of a value through a reference, without taking ownership. |
| Cargo | The Rust build tool and package manager. |
| CI | Continuous integration. An automatic build and test of every change, usually on a server. |
| CLI | Command-line interface. |
| Clippy | The Rust linter. Finds common mistakes and code that is not idiomatic. |
| Closure | An anonymous function that can capture values from its environment. |
| Cohesion | How much the parts of one module belong together. High cohesion is good. |
| Composition root | The one place in a program that makes the concrete objects and connects them, usually `main`. |
| Coupling | How much one module depends on the details of another. Low coupling is good. |
| CQS | Command-query separation. A method either changes state or returns a value, not both. |
| Crate | One compilation unit of Rust: a library or a binary. A package contains one or more crates. |
| CRUD | Create, read, update, delete. The four basic operations on stored data. |
| CSV | Comma-separated values. A text format for tables. |
| Data race | Two threads access the same memory at the same time, at least one writes, and nothing synchronizes them. Safe Rust prevents data races. |
| DDD | Domain-driven design. A design method that models the software on the concepts and the language of the business domain. |
| Deadlock | Two or more threads wait for each other forever, usually for locks. |
| DIP | Dependency inversion principle. The "D" in SOLID. |
| Drop | The cleanup of a value when its owner goes out of scope. The `Drop` trait adds custom cleanup. |
| DRY | Do not repeat yourself. Every piece of knowledge has one representation in the system. |
| DTO | Data transfer object. A type that only carries data across a boundary, for example an HTTP request body. |
| Dyn-compatible | A trait that can be used as a trait object (`dyn Trait`). Older documents say "object safe". |
| Edition | A set of Rust language rules (2015, 2018, 2021, 2024). Each crate selects its edition. |
| Entity | A domain object with an identity that stays the same when its data changes. |
| Enum | In Rust: a type that is one of several variants. Each variant can carry different data. |
| FFI | Foreign function interface. Calls between Rust and other languages, usually C. |
| Future | A value that represents work that is not complete. A runtime polls it until it is ready. |
| Generics | Code that works with many types. The types are parameters. |
| Hexagonal architecture | An architecture where the application core defines ports (traits), and adapters connect the core to the world. All dependencies point to the core. Also "ports and adapters". |
| Idempotent | An operation that has the same result when you do it one time or many times. |
| ISP | Interface segregation principle. The "I" in SOLID. |
| KISS | Keep it simple. Prefer the simplest design that works. |
| Lifetime | The part of a program where a reference is valid. |
| LSP | Liskov substitution principle. The "L" in SOLID. |
| Macro | Code that writes code at compile time. `macro_rules!` (declarative) or procedural macros. |
| Miri | An interpreter for Rust that finds undefined behavior in `unsafe` code. |
| Monomorphization | The compiler makes a separate copy of a generic function for each concrete type. |
| Move | The transfer of ownership of a value. The old variable cannot be used after the move. |
| MSRV | Minimum supported Rust version. The oldest compiler version that a crate supports. |
| Mutex | Mutual exclusion lock. Only one thread at a time can access the data inside. |
| Newtype | A struct with one field that gives an existing type a new meaning: `struct AccountId(u64);`. |
| OCP | Open/closed principle. The "O" in SOLID. |
| ORM | Object-relational mapper. A library that maps database tables to types and generates SQL. |
| Orphan rule | You can implement a trait for a type only if the trait or the type is defined in your crate. |
| OWASP | Open Worldwide Application Security Project. Publishes the OWASP Top 10 list of web security risks. |
| Ownership | The Rust rule that each value has exactly one owner, which drops it at the end of its scope. |
| Package | A directory with a `Cargo.toml` file. It contains one or more crates. |
| Panic | An unrecoverable error. It unwinds the stack of the thread (or stops the program). |
| Port | In a hexagonal architecture: a trait that the core defines for something that it needs (a store) or gives (a use case). |
| PR | Pull request. A request to review and merge a branch. |
| RAII | Resource acquisition is initialization. A value gets a resource when it is made and gives it back when it is dropped. |
| Rc | Reference-counted pointer. Shared ownership in one thread. |
| Refactoring | A change to the structure of code that does not change its behavior. |
| RefCell | A type that checks the borrowing rules at run time, so that you can change data behind a shared reference. |
| REST | Representational state transfer. A style for HTTP APIs with resources and standard methods. |
| rustup | The installer and version manager for the Rust toolchain. |
| Semver | Semantic versioning. `MAJOR.MINOR.PATCH`: a major change breaks compatibility. |
| Send, Sync | Marker traits. `Send`: a value can move to another thread. `Sync`: a reference to it can be shared between threads. |
| Slice | A reference to a contiguous part of a sequence: `&str`, `&[T]`. |
| SOLID | Five design principles collected by Robert C. Martin: single responsibility, open/closed, Liskov substitution, interface segregation, dependency inversion. |
| SQL | Structured Query Language. The language of relational databases. |
| SRP | Single responsibility principle. The "S" in SOLID. |
| TDD | Test-driven development. Write a failing test, make it pass, then refactor. |
| Test double | An object that replaces a real dependency in a test: a stub, a fake, a spy, or a mock. |
| Token (AI model) | The unit of text that an AI model reads and writes. A token is a part of a word. The usage limits and the price of an AI agent depend on the number of tokens. See [token-usage.md](token-usage.md). |
| Trait | A set of methods that types can implement. Similar to an interface. |
| Trait object | A value of type `dyn Trait` behind a pointer. Its concrete type is known only at run time. |
| TUI | Terminal user interface. |
| Type-state | A design where each state of an object is a different type, so that the compiler rejects operations in the wrong state. |
| UB | Undefined behavior. A program state that the language does not define. Safe Rust has no UB. |
| `unsafe` | A Rust keyword that permits five operations that the compiler cannot check. The programmer must prove that they are correct. |
| Use case | A function or type in the core that does one task for a user, for example "import a statement". Also "application service". |
| Value object | A domain object that is defined only by its value, for example an amount of money. Immutable. |
| Vtable | A table of function pointers that a trait object uses for dynamic dispatch. |
| Wasm | WebAssembly. A binary format that runs in web browsers and other runtimes. |
| Workspace | A set of Cargo packages that share one `Cargo.lock` and one `target/` directory. |
| YAGNI | You are not going to need it. Do not build features or abstractions for a future that you only imagine. |
