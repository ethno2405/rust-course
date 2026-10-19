# Phase 5: Software Design and SOLID

**Time:** 4 weeks. Plan 5 weeks if design principles are new to you.
**Prerequisites:** Phase 4
**Code:** `labs/phase-5-design/` and `labs/ledger/`

## Goal

At the end of this phase, you can:

- Explain why design matters: the cost of a change, coupling, and cohesion.
- Explain each SOLID principle, show how it maps to Rust, and name its costs.
- Apply other principles: DRY, KISS, YAGNI, the Law of Demeter, tell-don't-ask, command-query separation, and composition over inheritance.
- Model a domain with types: value objects, entities, identifiers, states, and invariants.
- Design a hexagonal architecture with a Cargo workspace, and keep the dependencies in the correct direction.
- Use the common design patterns in their Rust form, and recognize anti-patterns and code smells.
- Refactor code in small, safe steps with tests.
- Use test-driven development (TDD) and test doubles.
- Write an architecture decision record (ADR).

## Reading

| Source | Part |
|---|---|
| Robert C. Martin, *Clean Architecture* | The parts about design principles (SOLID), component principles, and architecture. The examples use Java. |
| John Ousterhout, *A Philosophy of Software Design*, 2nd edition | All. It is short. It gives a different view from Martin: deep modules, and complexity as the main enemy. |
| David Thomas and Andrew Hunt, *The Pragmatic Programmer*, 20th anniversary edition | The topics "The Essence of Good Design", "DRY—The Evils of Duplication", "Orthogonality", "Decoupling", and "Refactoring" |
| Martin Fowler, *Refactoring*, 2nd edition | The first chapters (the example and the principles), and "Bad Smells in Code". Use the catalog as a reference. |
| Scott Wlaschin, *Domain Modeling Made Functional* | The parts about domain modeling with types. The examples use F#, but they map directly to Rust enums and structs. |
| Rust Design Patterns (rust-unofficial.github.io/patterns) | Idioms, patterns, and anti-patterns |
| Alistair Cockburn, "Hexagonal Architecture" (alistair.cockburn.us) | The original article about ports and adapters |
| Sandi Metz, "The Wrong Abstraction" (sandimetz.com) | A short article about the cost of DRY |
| Kent Beck, *Test-Driven Development: By Example* | Part I (the example) |

If design principles are new to you, read *The Pragmatic Programmer* topics first.

## Topics

### 5.1 Why design matters

- Most of the cost of software comes after the first version: every change, defect fix, and new feature. **Good design makes changes cheap.** It is not about beauty.
- **Coupling:** how much one part must know about another part. When two parts are tightly coupled, a change in one part forces a change in the other part.
- **Cohesion:** how much the parts inside a module belong together. A module with high cohesion has one purpose. All its parts change for the same reason.
- The goal: **low coupling between modules, high cohesion inside modules.**
- **Complexity** (Ousterhout) shows in three ways: a simple change needs changes in many places, a developer must know much to make a change, and it is not clear which code to change.
- **Principles are heuristics.** They describe forces, not laws. Each principle has a cost: more types, more indirection, more files. A design is a set of trade-offs. A 200-line tool does not need the architecture of a large service.

### 5.2 S: the single responsibility principle (SRP)

> A module should have one, and only one, reason to change. (Robert C. Martin)

Martin later made it more exact: a module should be responsible to one **actor**, one group of people who request changes.

**Example.** One function reads a CSV bank statement, categorizes the transactions, sums them by category, and renders an HTML report. Three groups can request a change: the bank changes its file format, the user changes the categorization rules, and the designer changes the report layout. Each change touches the same function, and can break the other parts.

**In Rust:**

- Split the work into functions and modules by reason to change: `import` (file formats), `categorize` (rules), `report` (aggregation), `render` (presentation).
- Separate **calculation from I/O.** A function that computes a report takes data and returns data. A different function reads the file and prints the result. The calculation is then easy to test.
- A module is the main unit of responsibility in Rust, not a type. A module can contain several small types and functions that change together.

**Cost:** more modules and functions. If you split too much, a reader must jump between many small pieces to understand one feature ("shallow modules", Ousterhout). Split by reason to change, not by line count.

### 5.3 O: the open/closed principle (OCP)

> Software entities should be open for extension, but closed for modification. (Bertrand Meyer; this form from Robert C. Martin)

You can add new behavior without a change to existing, tested code.

**In Rust**, traits give the extension point:

```rust
pub trait StatementFormat {
    fn name(&self) -> &str;
    fn parse(&self, input: &str) -> Result<Vec<RawTransaction>, ImportError>;
}

pub struct Importer {
    formats: Vec<Box<dyn StatementFormat>>,
}

impl Importer {
    pub fn register(&mut self, format: Box<dyn StatementFormat>) {
        self.formats.push(format);
    }
}
```

A new bank format is a new type that implements `StatementFormat`. The `Importer` does not change.

**Enums and traits: the expression problem.** Rust gives you two tools, and they are open in different directions:

| | Add a new **variant** (a new type) | Add a new **operation** |
|---|---|---|
| `enum` and `match` | Change every `match` (the compiler finds them all) | Add one function. No other change. |
| Trait and implementations | Add one type. No other change. | Change the trait and every implementation |

- Use an **enum** when the set of variants is closed and under your control, and you often add operations. Examples: the commands of your CLI, the states of a state machine, the kinds of a token.
- Use a **trait** when other code (other crates, plugins, tests) must add types, or when you often add types. Examples: storage backends, file formats, notification channels.
- An enum is not a violation of OCP. When you add a variant, the compiler shows every place to change. In a dynamic language, the same change can fail silently at run time.

**Cost:** an extension point is an abstraction. If you never add a second implementation, it only adds indirection. Add the trait when the second format appears, or when a test needs a different implementation (topic 5.14).

### 5.4 L: the Liskov substitution principle (LSP)

> Objects of a subtype must be usable through the interface of the supertype, without a change of the correct behavior. (Barbara Liskov, 1987; Liskov and Wing, 1994)

Rust has no inheritance between structs. In Rust, the principle is about **trait contracts**: every implementation of a trait must keep the promises of the trait, not only its signatures.

- The compiler checks the signatures. It cannot check the behavior. Write the contract in the documentation of the trait.
- **Examples from the standard library:** `Hash` must agree with `Eq` (equal values have equal hashes). `Ord` must be a total order that agrees with `PartialEq`. `ExactSizeIterator::len` must return the exact number of remaining items. If your implementation breaks these contracts, `HashMap`, `sort`, and other code give wrong results. This is not undefined behavior, but it is a defect.
- **Example of a violation:** a trait `TransactionStore` says "after `save(tx)` returns `Ok`, `find(tx.id)` returns the transaction". A `CacheStore` keeps only the last 100 transactions and drops older ones silently. It compiles, but code that uses a `TransactionStore` now fails in a way that the caller cannot see.
- Rules for an implementation: it must not require more from the caller (stronger preconditions), it must not give less (weaker postconditions), and it must not return errors or panic in situations where the contract does not permit them.

**Contract tests:** write the tests of the contract one time, as a generic function, and run them for every implementation:

```rust
pub fn check_store_contract<S: TransactionStore>(mut store: S) {
    let tx = sample_transaction();
    store.save(tx.clone()).expect("save works on an empty store");
    assert_eq!(store.find(tx.id()).expect("find works"), Some(tx));
}

#[test]
fn memory_store_keeps_the_contract() {
    check_store_contract(MemoryStore::default());
}
```

### 5.5 I: the interface segregation principle (ISP)

> Clients should not be forced to depend on methods that they do not use. (Robert C. Martin)

**In Rust:** prefer small, focused traits.

- The standard library is a good example: `Read`, `Write`, `BufRead`, and `Seek` are separate traits. A function that only reads takes `impl Read`. A type implements only the traits that it can support.
- A large `Repository` trait with 12 methods forces every implementation (and every test double) to implement all 12. A report generator needs only "list the transactions". Give it a `TransactionSource` trait with one method.
- Combine small traits with a supertrait and a blanket implementation, if a client needs several of them:

  ```rust
  pub trait Store: TransactionSource + TransactionSink {}
  impl<T: TransactionSource + TransactionSink> Store for T {}
  ```

- A closure is the smallest possible interface. A function that needs only one operation can take `impl Fn(AccountId) -> Option<Account>`.

**Cost:** many small traits can make the code harder to navigate. Split a trait when different clients use different parts of it.

### 5.6 D: the dependency inversion principle (DIP)

> High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions. (Robert C. Martin)

The important word is **inversion**: the high-level code (your domain logic) **owns** the abstraction. The low-level code (the database, the HTTP client) implements it.

```text
Without DIP:   ledger-core ──depends on──► sqlx, reqwest, std::fs

With DIP:      ledger-core  defines  trait TransactionStore, trait RateProvider
                    ▲                         ▲
                    │ implements              │ implements
          ledger-store-sqlite         ledger-rates-http
```

**In Rust:**

- The domain crate defines the traits (the **ports**). Adapter crates depend on the domain crate and implement the traits. The domain crate has no dependency on `sqlx` or `reqwest`. Cargo enforces this: a crate cannot use a crate that is not in its `Cargo.toml`.
- **Generics or trait objects?**

  ```rust
  // Generics: static dispatch. The type appears in every signature that uses it.
  pub struct Converter<P: RateProvider> {
      rates: P,
  }

  // Trait object: dynamic dispatch. Simpler signatures. Easy to select at run time.
  pub struct Converter {
      rates: Box<dyn RateProvider>,
  }
  ```

  Use generics in libraries and in code where speed matters. Use trait objects (`Box<dyn Trait>`, `Arc<dyn Trait + Send + Sync>`) to connect the parts of an application. The cost of one indirect call is small compared with a database query.

- **Composition root:** connect the concrete adapters in one place, usually `main`. Only `main` knows that the store is SQLite.
- **Dependency injection is only passing values.** You do not need a framework. Pass the dependencies to the constructor.
- **Where to invert:** at the boundaries of the system: storage, network, file system, time, random numbers, and the environment. Do not make a trait for every struct.

### 5.7 Composition over inheritance

- Rust has no inheritance of data. A struct cannot extend a different struct.
- **Behavior reuse:** default methods in traits, generic functions, and blanket implementations.
- **Data reuse:** a struct contains other structs (composition), and forwards calls to them (delegation).
- **Decorator:** a wrapper that implements the same trait and adds behavior:

  ```rust
  pub struct Logged<S> {
      inner: S,
  }

  impl<S: TransactionStore> TransactionStore for Logged<S> {
      fn save(&mut self, tx: Transaction) -> Result<(), StoreError> {
          eprintln!("save transaction {}", tx.id());
          self.inner.save(tx)
      }
      // ... forward the other methods.
  }
  ```

- **Anti-pattern:** `impl Deref<Target = Inner> for Outer` to "inherit" the methods of `Inner`. It surprises readers, it does not work with traits, and clippy and the Rust Design Patterns book warn against it. Write the forwarding methods.

### 5.8 Other principles

| Principle | Meaning | In Rust, and the trade-off |
|---|---|---|
| **DRY**: do not repeat yourself (Hunt and Thomas) | Every piece of **knowledge** has one representation in the system | DRY is about knowledge, not about text. Two functions that look the same but change for different reasons are not duplication. "Duplication is far cheaper than the wrong abstraction" (Sandi Metz). Wait for the third copy before you extract ("rule of three"). |
| **KISS**: keep it simple | Prefer the simplest design that works | A function before a trait. An enum before a trait object. A `Vec` before a custom data structure. |
| **YAGNI**: you are not going to need it (extreme programming) | Do not build features or abstractions for a future that you only imagine | Do not add generics, traits, or configuration "for later". A later change in Rust is safe: the compiler finds all the places. |
| **Law of Demeter** | A function talks only to its direct collaborators, not to the collaborators of its collaborators | `ledger.account(id)?.balance()` is fine. `ledger.accounts[3].transactions.last().unwrap().amount.value` knows too much about the structure. Add a method. |
| **Tell, don't ask** | Tell an object what to do. Do not ask for its data and decide outside it. | `account.withdraw(amount)?` in place of `if account.balance >= amount { account.balance -= amount }`. The rule stays inside the type that owns the data. |
| **Command-query separation (CQS)** (Bertrand Meyer) | A method either changes state (a command) or returns a value (a query), not both | `&self` for queries and `&mut self` for commands make this visible in Rust. Exceptions are fine when they are clear: `Vec::pop`. |
| **Separation of concerns** | Different concerns (domain rules, storage, presentation, transport) are in different modules | The base of the hexagonal architecture (topic 5.10) |
| **Fail fast** | Detect errors as early as possible | Validate at the boundary ("parse, don't validate"). Check configuration at startup. Prefer compile errors to run-time errors. |
| **Principle of least surprise** | Code behaves as a reader expects | Follow the API Guidelines. `to_` methods do not change `self`. `Display` does not do I/O. |
| **Deep modules** (Ousterhout) | A good module has a simple interface and hides much complexity | Prefer a few functions that do much to many functions that each do little. Do not split a module only to make functions short. |

### 5.9 Domain modeling with types

- **Value object:** defined only by its value. Immutable. Two equal values are interchangeable. `Money`, `Currency`, `DateRange`, `CategoryName`. In Rust: a struct or a newtype with `Clone`, `PartialEq`, `Eq`, often `Copy`.
- **Entity:** has an identity that stays the same when its data changes. `Account`, `Transaction`. In Rust: a struct with an identifier field. Two entities are the same when their identifiers are equal.
- **Identifiers:** use newtypes: `struct AccountId(Uuid)`. Then the compiler rejects an `AccountId` where a `TransactionId` is expected.
- **Invariants:** rules that are always true for a value. Keep the fields private and check the rules in the constructor (phase 4, topic 4.7).
- **States:** use an enum, not `bool` fields or a `status: String`. Each variant carries only the data that is valid in that state.

  ```rust
  pub enum Budget {
      Draft { limit: Money },
      Active { limit: Money, spent: Money, period: Month },
      Closed { limit: Money, spent: Money, period: Month, closed_on: NaiveDate },
  }
  ```

- **Type-state:** when the valid operations depend on the state, and the state is known at compile time, make a type for each state. A method that changes the state consumes `self` and returns the new type. The compiler then rejects a call in the wrong state. Use it for workflows and protocols (a connection, a builder, an approval). Do not use it when the state comes from data at run time (for example from a database): use an enum there.
- **Functional core, imperative shell** (Gary Bernhardt): the core is pure functions on domain types. It has no I/O. The shell reads input, calls the core, and writes the output. The core is easy to test without test doubles.

### 5.10 Architecture

| Style | Idea | Dependency direction |
|---|---|---|
| **Layered** | Presentation → application → domain → infrastructure | Each layer depends on the layer below it. The domain depends on the database. |
| **Hexagonal** (ports and adapters, Alistair Cockburn) | The application core is in the center. It defines ports (traits). Adapters connect the core to the world: driving adapters (CLI, HTTP) call the core, driven adapters (database, HTTP client) implement ports. | All dependencies point to the core. |
| **Clean architecture** (Robert C. Martin) | The same idea with more rings: entities, use cases, interface adapters, frameworks. | The dependency rule: source code dependencies point only inward. |

**A hexagonal ledger as a Cargo workspace:**

```text
labs/ledger/
  Cargo.toml                  [workspace]
  crates/
    ledger-core/              Domain types, rules, use cases, ports (traits). No I/O crates.
    ledger-import/            Statement formats (CSV adapters). Depends on ledger-core.
    ledger-store-json/        TransactionStore adapter for a JSON file. Depends on ledger-core.
    ledger-cli/               The binary. Composition root. Depends on all crates.
```

- Each crate has a clear responsibility, and `cargo tree` shows the dependency direction.
- **Use cases** (also "application services") are functions or structs in the core that do one task for a user: `import_statement`, `monthly_report`, `set_budget`. They call the ports. A driving adapter (the CLI, and later the HTTP API) calls the use cases.
- **Do not leak adapter types into the core.** The core does not know `serde_json::Value`, `sqlx::Row`, or `clap::ArgMatches`. Map them to domain types in the adapter.
- **When not to use it:** a small tool, a prototype, or code that is only a thin layer over a database. Start with modules in one crate. Split into crates when the boundaries are clear.

### 5.11 Design patterns in Rust

Many patterns from the book *Design Patterns* (Gamma, Helm, Johnson, Vlissides, 1994) look different in Rust. Some are built into the language.

| Pattern | Rust form |
|---|---|
| Strategy | A trait with implementations, or a closure parameter (`impl Fn`) |
| Builder | A builder type with a `build()` method (phase 4, topic 4.8) |
| Newtype | A tuple struct around one value (phase 4) |
| RAII guard | A value whose `Drop` releases a resource (phase 2) |
| State | An `enum` with a `match`, or type-state with one type for each state |
| Command | An `enum` of commands, or `Box<dyn FnOnce()>` |
| Observer | Callbacks (`Vec<Box<dyn Fn(&Event)>>`), or channels (phase 6) |
| Adapter | A wrapper type that implements the trait that a client expects |
| Decorator | A generic wrapper that implements the same trait (topic 5.7) |
| Factory | An associated function (`Account::new`), or a function that returns `Box<dyn Trait>` |
| Iterator | The `Iterator` trait (phase 3) |
| Visitor | Rarely needed. Use an `enum` and a `match`. |
| Singleton | Avoid it. Pass the value as a dependency. If you must have a global value, use `std::sync::OnceLock` or `LazyLock` (stable since Rust 1.70 and 1.80). |

### 5.12 Anti-patterns and code smells

A **code smell** (Kent Beck, Martin Fowler) is a sign that a design can have a problem. It is not always a problem. Look at the reason.

| Smell or anti-pattern | Sign | Better |
|---|---|---|
| Clone to satisfy the borrow checker | `.clone()` on large values, in loops, with no reason in the domain | Change the ownership (phase 2, topic 2.8) |
| `Rc<RefCell<T>>` everywhere | An object graph like in a garbage-collected language | One owner, identifiers in place of references |
| `Deref` polymorphism | `Deref` to imitate inheritance | Composition and forwarding methods |
| Stringly typed code | `status: String`, `currency: String`, `kind: &str` | Enums and newtypes |
| Primitive obsession | `f64` for money, `u64` for every identifier | Value types |
| Boolean parameters | `render(true, false)` | Enums: `render(Format::Html, Totals::Hide)` |
| `unwrap()` everywhere | Panics on input that can be wrong | `Result` and `?` |
| God module or god struct | One `utils.rs`, `Manager`, or `App` that does everything | Split by responsibility |
| Long parameter list | 5 or more parameters | A parameter struct, or a builder |
| Feature envy | A function uses more data from a different type than from its own | Move the function to that type |
| Shotgun surgery | One change needs edits in many modules | Put the knowledge in one place |
| Speculative generality | Traits, generics, or configuration that only one case uses | Remove them (YAGNI) |
| Global mutable state | `static` with a `Mutex`, used everywhere | Pass the state as a dependency |

### 5.13 Refactoring

> Refactoring: a change to the internal structure of software to make it easier to understand and cheaper to modify, without a change to its observable behavior. (Martin Fowler)

- Refactor in **small steps.** After each step, the code compiles and the tests pass. Commit often.
- Do not refactor and change behavior in the same step. Do one, then the other.
- If the code has no tests, write **characterization tests** first: tests that record what the code does now, also if the behavior looks wrong.
- Common refactorings in Rust:

  | Refactoring | Tool |
  |---|---|
  | Extract function, extract variable, inline | rust-analyzer assists |
  | Rename a symbol in all files | rust-analyzer "Rename" |
  | Move an item to a different module | Cut and paste. The compiler shows the broken paths. |
  | Replace a primitive with a newtype | Change the type. Follow the compiler errors. |
  | Replace `bool` parameters with an enum | Change the signature. Follow the compiler errors. |
  | Introduce a parameter struct | Group the parameters in a struct |
  | Replace a conditional with a trait, or a trait with an enum | Topic 5.3 |

- **The compiler is your strongest refactoring tool.** Change a type, then follow the errors. A refactoring in Rust that compiles is usually correct. This makes large refactorings much safer than in dynamic languages.

### 5.14 Testability and test-driven development

- **Test-driven development (TDD)** (Kent Beck): write a failing test (red), write the simplest code that passes it (green), then improve the design (refactor). Repeat in small cycles.
- TDD gives fast feedback, and it makes you design the interface from the side of the caller first.
- **Test doubles** replace a real dependency in a test:

  | Double | Behavior | Rust |
  |---|---|---|
  | Stub | Returns fixed values | A struct that implements the port with constant results |
  | Fake | A simple, working implementation | `MemoryStore` with a `Vec` or a `HashMap` |
  | Spy | Records the calls for a later check | A struct with a `RefCell<Vec<Call>>` |
  | Mock | Checks the calls against expectations | The `mockall` crate (phase 8) |

- **Prefer fakes** for ports. A fake tests behavior. A mock tests the interaction, and a test with many mocks breaks when you refactor.
- **Inject time and other non-deterministic inputs:**

  ```rust
  pub trait Clock {
      fn today(&self) -> NaiveDate;
  }

  pub struct SystemClock;

  impl Clock for SystemClock {
      fn today(&self) -> NaiveDate {
          chrono::Local::now().date_naive()
      }
  }

  pub struct FixedClock(pub NaiveDate);

  impl Clock for FixedClock {
      fn today(&self) -> NaiveDate {
          self.0
      }
  }
  ```

  Often a parameter is simpler than a trait: `fn is_overdue(&self, today: NaiveDate) -> bool`. The caller gets the date from the clock.

### 5.15 Clean code

- **Names:** a name says what a value means and why it exists. Use the words of the domain (the "ubiquitous language" of domain-driven design). Long scopes need longer names.
- **Functions:** a function does one thing at one level of abstraction. A function that reads like a list of steps calls functions that each do one step.
- **Comments** explain **why**, not what. If a comment explains what the code does, improve the names. Document the contracts of public items (phase 4).
- **Constants:** give names to magic values: `const MAX_IMPORT_LINES: usize = 100_000;`.
- **Error paths:** handle errors early and return (`let ... else`, `?`). Keep the normal path at the lowest indentation.
- **Two views:** *Clean Code* (Martin) recommends very small functions. *A Philosophy of Software Design* (Ousterhout) warns that very small functions can make "shallow" modules that are harder to understand. Both books agree on the goal: code that is easy to read and to change. Use your judgment, and ask a reviewer.

### 5.16 Architecture decision records

- An **architecture decision record (ADR)** (Michael Nygard) records one important decision: its context, the decision, and its consequences.
- Keep ADRs in the repository, in `docs/adr/`, numbered: `0001-use-a-cargo-workspace.md`.
- Format: **Title**, **Status** (proposed, accepted, superseded by ADR n), **Context** (the forces and the options), **Decision**, **Consequences** (the good and the bad results).
- An ADR is never changed after it is accepted. A new ADR replaces it.

## Exercises

Make the workspace `labs/phase-5-design/`. If you know design principles well (D = 3 in your profile), do exercises 5.2, 5.5, 5.7, and 5.9.

### Exercise 5.1: Single responsibility

This function works, but it is hard to change and to test.

```rust
use std::collections::HashMap;
use std::fs;

pub fn monthly_report(path: &str, month: u32, html: bool) -> String {
    let text = fs::read_to_string(path).unwrap();
    let mut totals: HashMap<String, f64> = HashMap::new();
    let mut count = 0;
    for (i, line) in text.lines().enumerate() {
        if i == 0 {
            continue;
        }
        let parts: Vec<&str> = line.split(',').collect();
        if parts.len() != 4 {
            println!("bad line {}", i + 1);
            continue;
        }
        let line_month: u32 = parts[0][5..7].parse().unwrap();
        if line_month != month {
            continue;
        }
        let amount: f64 = parts[2].parse().unwrap();
        let mut category = parts[3].to_string();
        if category.is_empty() {
            if parts[1].to_lowercase().contains("coffee") {
                category = "Food".to_string();
            } else {
                category = "Other".to_string();
            }
        }
        *totals.entry(category).or_insert(0.0) += amount;
        count += 1;
    }
    let mut out = String::new();
    if html {
        out.push_str("<table>");
        for (category, total) in &totals {
            out.push_str(&format!("<tr><td>{}</td><td>{:.2}</td></tr>", category, total));
        }
        out.push_str("</table>");
    } else {
        for (category, total) in &totals {
            out.push_str(&format!("{}: {:.2}\n", category, total));
        }
    }
    out.push_str(&format!("{} transactions", count));
    out
}
```

1. List all the reasons why this function can change. List all the defects and risks that you find (there are more than eight).
2. Write characterization tests for the current behavior first. You need a test file: write it with `tempfile`.
3. Refactor in small steps, with a commit after each step. Separate the reading, the parsing, the categorization, the aggregation, and the rendering. The calculation must not do I/O.
4. Fix the defects only after the refactoring. Write a test for each defect first.

### Exercise 5.2: Open/closed and the expression problem

1. Write a notification system with two channels: email and SMS (print to stdout in place of a real send). Design it with a trait.
2. Add a third channel (a webhook). Count the files and functions that you changed.
3. Write the same system with an enum. Add the third channel. Count again.
4. Now add a new operation to both designs: "estimate the cost of a message". Count again.
5. Compare the results with exercise 3.1. Write a short rule in your engineering notebook: when do you use an enum, and when a trait?

### Exercise 5.3: Liskov and contract tests

1. Define a trait `KeyValueStore` with `get`, `put`, and `remove`. Write the contract in the documentation of the trait.
2. Write a generic contract test function (topic 5.4), with at least five checks.
3. Implement the trait with a `HashMap`, and with a JSON file. Run the contract tests on both.
4. Write a `LimitedStore` that keeps only 10 keys and drops the oldest key silently. Run the contract tests. Which check fails? Change the design so that `LimitedStore` keeps the contract (for example: return an error when the store is full).
5. Implement `Hash` and `PartialEq` for a type so that they do not agree. Show that a `HashSet` then gives a wrong result.

### Exercise 5.4: Interface segregation

Look at this trait:

```rust
pub trait LedgerRepository {
    fn add_transaction(&mut self, tx: Transaction) -> Result<(), RepoError>;
    fn update_transaction(&mut self, tx: Transaction) -> Result<(), RepoError>;
    fn delete_transaction(&mut self, id: TransactionId) -> Result<(), RepoError>;
    fn transactions(&self, filter: &Filter) -> Result<Vec<Transaction>, RepoError>;
    fn accounts(&self) -> Result<Vec<Account>, RepoError>;
    fn export_csv(&self, path: &Path) -> Result<(), RepoError>;
    fn backup(&self, dir: &Path) -> Result<(), RepoError>;
    fn send_monthly_email(&self, to: &str) -> Result<(), RepoError>;
}
```

1. A report generator needs only to read transactions. Write a test double for it with this trait. How many methods must the double implement?
2. Split the trait. Which methods are not about storage at all? Where do they belong?
3. Write the report generator again with the smaller trait, and with a closure. Compare.

### Exercise 5.5: Dependency inversion and a hexagonal design

Build the core of a URL shortener.

1. Make a workspace with the crates `shortener-core`, `shortener-store-memory`, `shortener-store-file`, and `shortener-cli`.
2. The core defines `struct ShortCode`, `struct Link`, the port `trait LinkStore`, the port `trait CodeGenerator`, and the use cases `shorten(url)` and `resolve(code)`. The core has no I/O dependencies.
3. Implement both stores and a random code generator. Test the use cases with a fake store and a fixed code generator.
4. Run `cargo tree` for each crate. Draw the dependency graph. Check that all arrows point to the core.
5. Change the CLI to use the file store in place of the memory store. Which crate changed?

### Exercise 5.6: Patterns in Rust

1. **Type-state:** model an expense claim: `Draft` → `Submitted` → `Approved` or `Rejected` → `Paid`. Only an approved claim can be paid. Write a test that shows the compile error for `draft.pay()` (as a `compile_fail` doc test).
2. **Decorator:** write `Logged<S>` and `Cached<S>` for the `LinkStore` of exercise 5.5. Combine them: `Logged<Cached<FileStore>>`. The core does not change.
3. **Strategy:** write a function that sorts transactions by a strategy. Implement it with a trait, then with a closure. Which one is simpler here?

### Exercise 5.7: Test-driven development with a clock

Build a subscription module with TDD. Write each test before the code.

1. A subscription has a start date, a period (monthly or yearly), and a state (active, cancelled, expired).
2. Rules: `next_billing_date(today)`; a cancelled subscription stays usable until the end of its period; an expired subscription cannot be cancelled; a monthly subscription that starts on January 31 renews on the last day of February.
3. Inject the date (topic 5.14). No test uses the real date.
4. Commit after each green test and after each refactoring. Look at the git log: does it show the red-green-refactor cycle?

### Exercise 5.8: Find the smells

Read this design and list the smells from topic 5.12. Then write a better design: only the types and the signatures, with no function bodies.

```rust
pub struct Manager {
    pub data: Vec<(String, String, f64, bool)>,  // name, currency, amount, is_paid
    pub db: Rc<RefCell<Database>>,
    pub mode: String,                            // "test" or "prod"
}

impl Manager {
    pub fn handle(&mut self, s: &str, flag: bool, flag2: bool) -> String {
        // 150 lines: parse the command in `s`, change `data`,
        // write to the database, and format the answer as text or JSON.
        todo!()
    }
}
```

### Exercise 5.9: Design review and ADRs

1. Write ADRs for the important decisions of ledger stage 2 (see the project document): at least the workspace structure, the storage port, and the use of trait objects or generics.
2. Ask the mentor (or a different person) for a design review of your ledger. Answer the "what if" questions: a new bank format, a new storage backend, an HTTP API. How many crates and files must change for each?

## Project: Ledger, stage 2

See [../projects/ledger.md](../projects/ledger.md), stage 2.

## Done criteria

- [ ] I can explain coupling, cohesion, and why they matter for the cost of a change.
- [ ] I can explain each SOLID principle with a Rust example, and name one cost of each principle.
- [ ] I can decide between an enum and a trait for an extension point, and explain the expression problem.
- [ ] I can write contract tests for a trait.
- [ ] I can explain DRY, KISS, YAGNI, the Law of Demeter, and tell-don't-ask, and when they conflict.
- [ ] I can model a domain with value objects, entities, identifiers, and enums for states.
- [ ] My ledger has a hexagonal architecture: the core has no I/O dependencies, and `cargo tree` shows that all dependencies point to the core.
- [ ] I can refactor in small steps with tests and commits.
- [ ] I used TDD for a module, and the git history shows it.
- [ ] I wrote ADRs for my main design decisions.
- [ ] Ledger stage 2 is complete.

## Next

[Phase 6: Concurrency and async](phase-6-concurrency-async.md)
