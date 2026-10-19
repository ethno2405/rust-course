# Phase 4: Errors, Modules, and API Design

**Time:** 3 weeks
**Prerequisites:** Phase 3
**Code:** `labs/phase-4-errors-apis/` and `labs/ledger/`

## Goal

At the end of this phase, you can:

- Decide when to return an error and when to panic.
- Design error types for a library, and use `thiserror` and `anyhow`.
- Give error messages that tell the user what went wrong and what to do.
- Split a project into a library crate and a thin binary crate, and into a workspace.
- Design a public API that is hard to use incorrectly: newtypes, "parse, don't validate", builders, and the Rust API Guidelines.
- Document a library with doc comments and doc tests.
- Read and write files, and serialize data with `serde`.
- Write a command-line application with `clap`, and test it.

## Reading

| Source | Part |
|---|---|
| The Rust Programming Language (the Book) | "Error Handling", "Writing Automated Tests", "An I/O Project: Building a Command Line Program", "More About Cargo and Crates.io" |
| Rust API Guidelines (rust-lang.github.io/api-guidelines) | All. Use the checklist page as a reference. |
| Luca Palmieri, "Error Handling In Rust: A Deep Dive" (lpalmieri.com) | All. It is also a chapter of *Zero To Production in Rust*. |
| Alexis King, "Parse, don't validate" (lexi-lambda.github.io) | All. The examples use Haskell, but the idea applies directly to Rust. |
| Ken Youens-Clark, *Command-Line Rust* | Use it for more CLI practice (optional) |
| Command Line Applications in Rust (rust-cli.github.io/book) | All. It is short. |
| The documentation of `thiserror`, `anyhow`, `serde`, `clap` (docs.rs) | The front page and the examples |

## Topics

### 4.1 Errors and panics

| Situation | Use | Example |
|---|---|---|
| The failure is possible in normal operation, and the caller can react | `Result<T, E>` | A file does not exist. A user typed a wrong date. A network request timed out. |
| The failure is a bug: a broken invariant that the code promised | `panic!`, `unreachable!`, `expect("invariant")` | An index that the code computed is out of range. A state that the design makes impossible. |
| A test, an example, or a throwaway prototype | `unwrap()` | |

- A panic unwinds the stack of the current thread and drops all values on it. It is not an exception: do not use `catch_unwind` for control flow.
- **Rule for this course:** no `unwrap()` in library code or in the normal code path of an application. Use `expect` with a message that names the invariant: `.expect("the regex is a valid constant")`, not `.expect("failed")`.

### 4.2 Error types

**In a library:** define an error type that the caller can inspect. Usually this is an enum with one variant for each kind of failure that the caller can handle differently.

```rust
use std::path::PathBuf;

#[derive(Debug, thiserror::Error)]
pub enum ImportError {
    #[error("cannot read the file {path:?}")]
    Io {
        path: PathBuf,
        #[source]
        source: std::io::Error,
    },
    #[error("line {line}: {message}")]
    Parse { line: usize, message: String },
    #[error("the file has no header line")]
    MissingHeader,
}
```

- `thiserror` generates the `Display` and `Error` implementations. You can write them by hand (exercise 4.1 does it) and get the same result.
- `#[source]` (or a field with the name `source`) links the cause. `Error::source()` returns it. A reporter can print the whole chain.
- `#[from]` generates a `From` implementation, so that `?` converts the error automatically. Use it only when the conversion needs no context.

**In an application:** you usually do not need to inspect the error. You need a good message for the user. Use `anyhow`:

```rust
use anyhow::{Context, Result};

fn load(path: &std::path::Path) -> Result<String> {
    std::fs::read_to_string(path)
        .with_context(|| format!("cannot load the ledger from {}", path.display()))
}

fn main() -> Result<()> {
    let text = load(std::path::Path::new("ledger.json"))?;
    println!("{} bytes", text.len());
    Ok(())
}
```

- If `main` returns `Err`, the program prints the error with its chain ("Caused by: ...") and exits with code 1.
- **Rule:** libraries use `thiserror` (or hand-written error types). Binaries use `anyhow`.

### 4.3 Error design

- **An error is part of the API.** Callers match on it. Adding a variant to a public enum breaks callers that match on all variants. Mark the enum `#[non_exhaustive]` if you expect new variants.
- **Do not expose the error types of your dependencies** in your public API without a reason. If your error has a variant `Sqlx(sqlx::Error)`, you cannot change the database crate without a breaking change. Wrap it, and keep it as the `source`.
- **Messages:** lowercase, without a period at the end (the reporter joins the chain). Say what the program tried to do and with which value: "cannot parse the amount '12,5O' on line 7". Do not repeat the message of the source in your own message.
- **Do not log an error and also return it.** The caller decides what to do with it. If every layer logs it, the log has the same error many times.
- **Do not ignore errors.** `let _ = file.flush();` hides a failure. If you must ignore an error, write a comment that explains why.

### 4.4 Packages, crates, and workspaces

- **Thin `main`:** put the logic in a library crate (`src/lib.rs`). Keep `src/main.rs` small: parse the arguments, connect the parts, call the library, and print the result or the error. Integration tests can then test the library directly.
- **Workspace:** when a project has several crates, use a workspace. Share settings in the root `Cargo.toml`:

  ```toml
  [workspace]
  members = ["crates/*"]
  resolver = "3"

  [workspace.package]
  edition = "2024"

  [workspace.dependencies]
  serde = { version = "1", features = ["derive"] }

  [workspace.lints.rust]
  unsafe_code = "forbid"

  [workspace.lints.clippy]
  unwrap_used = "warn"
  ```

  A member uses them with `edition.workspace = true`, `serde.workspace = true`, and `[lints] workspace = true`.

- `clippy::unwrap_used` also finds `unwrap()` in tests. Add a `clippy.toml` file with `allow-unwrap-in-tests = true` in the workspace root.

### 4.5 Visibility and encapsulation

| Visibility | Visible in |
|---|---|
| (none) | The current module and its children |
| `pub(super)` | The parent module |
| `pub(crate)` | The whole crate |
| `pub` | Everywhere, also other crates |

- Make the public API as small as possible. Every public item is a promise: a change to it is a breaking change.
- Keep fields private when the type has rules (invariants). Give constructors and methods that keep the rules.
- Use `pub use` to re-export items at the crate root, so that users write `ledger::Money` and not `ledger::domain::money::Money`.

### 4.6 The Rust API Guidelines

The most important guidelines. The code in brackets is the identifier in the guidelines.

| Guideline | Meaning |
|---|---|
| Conversions follow `as_`, `to_`, `into_` (C-CONV) | See phase 1, topic 1.12 |
| Getters have no `get_` prefix (C-GETTER) | `fn amount(&self) -> Money` |
| Types implement the common traits (C-COMMON-TRAITS) | `Debug`, `Clone`, `PartialEq`, `Default`, ... (phase 3) |
| Conversions use the standard traits (C-CONV-TRAITS) | `From`, `TryFrom`, `AsRef`, `FromStr` |
| Arguments use types, not `bool` or `Option` flags (C-CUSTOM-TYPE) | `render(Format::Html)`, not `render(true)` |
| Builders for complex values (C-BUILDER) | Topic 4.8 |
| Newtypes give static distinctions (C-NEWTYPE) | `AccountId(u64)` and `TransactionId(u64)` cannot be mixed |
| Functions validate their arguments (C-VALIDATE) | Return an error, or use a type that cannot be invalid |
| Error types are meaningful and well-behaved (C-GOOD-ERR) | Topic 4.3 |
| Documentation has examples, and `# Errors` and `# Panics` sections (C-EXAMPLE, C-FAILURE) | Topic 4.9 |

- Mark functions whose result must be used with `#[must_use]`, for example a method that returns a new value and does not change `self`.

### 4.7 Make illegal states unrepresentable

Two related ideas make an API hard to use incorrectly:

- **Make illegal states unrepresentable** (Yaron Minsky): design the types so that a value with an invalid combination of data cannot exist. Example: phase 1, exercise 1.8 (an enum in place of several `bool` fields).
- **Parse, don't validate** (Alexis King): check the input one time at the boundary, and convert it to a type that proves the check. Code after the boundary uses the type and does not check again.

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct Username(String);

#[derive(Debug, PartialEq, Eq, thiserror::Error)]
pub enum UsernameError {
    #[error("a username must have 3 to 32 characters, not {0}")]
    Length(usize),
    #[error("a username can contain only a-z, 0-9, and '-', not {0:?}")]
    InvalidChar(char),
}

impl Username {
    pub fn parse(input: &str) -> Result<Self, UsernameError> {
        let len = input.chars().count();
        if !(3..=32).contains(&len) {
            return Err(UsernameError::Length(len));
        }
        if let Some(c) = input
            .chars()
            .find(|c| !(c.is_ascii_lowercase() || c.is_ascii_digit() || *c == '-'))
        {
            return Err(UsernameError::InvalidChar(c));
        }
        Ok(Self(input.to_owned()))
    }

    pub fn as_str(&self) -> &str {
        &self.0
    }
}
```

- A function that takes a `Username` does not need to check it. A function that takes a `String` must check it, or trust the caller.
- **Type-state:** a type parameter or separate types show the state of an object, so that the compiler rejects calls in the wrong state. Example: `Draft` → `Submitted` → `Approved`, and only `Approved` has a `pay()` method. Phase 5 uses it.

### 4.8 Builders

Rust has no default parameters and no overloading. For a type with many optional settings, use one of these:

- **`Default` and struct update syntax:** `ReportOptions { group_by: GroupBy::Month, ..Default::default() }`. Simple, if all fields are public and every combination is valid.
- **Builder:** a separate type with one method for each setting, and a `build()` method that checks the combination. Use it when some fields are required, when the fields are private, or when `build()` must check rules.

```rust
let request = RequestBuilder::new("https://example.com")
    .timeout(Duration::from_secs(5))
    .header("Accept", "application/json")
    .build()?;
```

### 4.9 Documentation

- `///` documents the next item. `//!` documents the module or crate that contains it.
- The first line is a one-sentence summary. Then the details. Then these sections, if they apply: `# Examples`, `# Errors` (when the function returns `Err`), `# Panics` (when it can panic), `# Safety` (for `unsafe` functions, phase 9).
- **Doc tests:** code blocks in the documentation compile and run with `cargo test`. The examples cannot become wrong without a test failure.
- `#![warn(missing_docs)]` at the top of `lib.rs` warns about public items without documentation.
- `cargo doc --open --no-deps` shows the documentation of your crates only.

### 4.10 Files and serialization

- `std::fs::read_to_string`, `std::fs::write`, `File::open` with `BufReader` for large files and `lines()`.
- `Path` (borrowed) and `PathBuf` (owned) are paths. They are not `String`: a path on Windows or Linux can contain bytes that are not valid UTF-8. Use `path.display()` to print a path.
- **serde:** `#[derive(Serialize, Deserialize)]` generates the code to convert a type to and from many formats. `serde_json` for JSON, `csv` for CSV, `toml` for TOML.
- Attributes change the mapping: `#[serde(rename_all = "snake_case")]`, `#[serde(default)]`, `#[serde(try_from = "String")]` (validate a newtype during deserialization).
- **Save a file safely:** write to a temporary file in the same directory, then rename it to the final name. A crash during the write then cannot leave a half-written file. The `tempfile` crate gives `NamedTempFile::persist` for this.
- **File format versions:** put a `version` field in files that your program stores. A later version of the program can then read old files.

### 4.11 Command-line applications

```rust
use clap::{Parser, Subcommand};
use std::path::PathBuf;

/// A personal finance ledger.
#[derive(Parser)]
#[command(version)]
struct Cli {
    /// The ledger file.
    #[arg(long, env = "LEDGER_FILE", default_value = "ledger.json")]
    file: PathBuf,

    #[command(subcommand)]
    command: Command,
}

#[derive(Subcommand)]
enum Command {
    /// Show the balance of each account.
    Balance,
    /// Import a CSV bank statement.
    Import { path: PathBuf },
}
```

- `cargo add clap --features derive,env`. The doc comments become the help text.
- Print results to **stdout** and messages and errors to **stderr**. Then the user can redirect the results to a file.
- **Exit codes:** 0 for success, not 0 for failure. `main` can return `ExitCode` (from `std::process`) or `Result`.
- Do not panic on wrong user input. A panic message is for developers, not for users.

### 4.12 Tests at the level of the crate

| Kind | Location | Tests |
|---|---|---|
| Unit tests | `#[cfg(test)] mod tests` in the same file | Private functions and small units |
| Integration tests | `tests/*.rs` | The public API only, as a different crate uses it |
| Doc tests | Code blocks in `///` comments | The examples in the documentation |
| CLI tests | `tests/*.rs` with the `assert_cmd` crate | The executable: arguments, output, exit code |

- Shared test helpers go in `tests/common/mod.rs`.
- Use the `tempfile` crate for tests that write files. Each test gets its own directory, so the tests can run in parallel.

## Exercises

Make the workspace `labs/phase-4-errors-apis/`.

### Exercise 4.1: Error type by hand

1. Write a parser for a simple configuration format: `key = value` lines, `#` comments, and empty lines.
2. Define `enum ConfigError` with variants for an I/O error (with the path), a line without `=` (with the line number), an empty key, and a duplicate key (with both line numbers).
3. Implement `Display` and `std::error::Error` by hand, with `source()` for the I/O error.
4. Write a test for each error. Print each error with its full chain.

### Exercise 4.2: `thiserror` and `anyhow`

1. Change the error type of exercise 4.1 to `thiserror`. The tests must not change.
2. Write a small binary that loads a configuration file and prints a value. Use `anyhow` with `.context(...)` at each step.
3. Run it with a missing file and with a broken file. Is the message useful to a user who does not know Rust? Improve it.

### Exercise 4.3: Newtypes

1. Write `Username` (topic 4.7), `Port` (1–65535), and `Percentage` (0–100, with two decimal places, stored as an integer).
2. Implement `TryFrom`, `FromStr`, `Display`, and `AsRef<str>` where it applies.
3. Add `serde` support so that deserialization of an invalid value fails: use `#[serde(try_from = "String")]`.
4. Write a function that takes a `Port`. Show that the caller cannot give it a `u16` by mistake.

### Exercise 4.4: A command-line tool

Write `grepr`, a small version of `grep`.

1. Arguments with `clap`: a pattern, one or more files, `--ignore-case`, `--count`, `--line-number`.
2. Put the logic in `lib.rs`. `main.rs` has fewer than 30 lines.
3. Print matches to stdout, and errors (for example a missing file) to stderr. Continue with the next file after an error. The exit code is 0 if a match was found, 1 if not, and 2 if an error occurred.
4. Write unit tests for the logic and CLI tests with `assert_cmd` for the exit codes.

### Exercise 4.5: Documentation

1. Add `#![warn(missing_docs)]` to the library of exercise 4.4. Document every public item.
2. Add an `# Examples` section with a doc test to the main function. Add `# Errors` where it applies.
3. Run `cargo test --doc` and `cargo doc --open --no-deps`. Read your documentation as a new user.

### Exercise 4.6: Builder

1. Write `ReportOptions` with these settings: a date range (required), a grouping (day, week, month; default month), a list of categories to include (default all), and a currency (optional).
2. Implement it two times: with `Default` and struct update syntax, and with a builder whose `build()` rejects a date range whose end is before its start.
3. Compare the two designs. When is each one better?

### Exercise 4.7: Serialization

1. Define a struct with nested data, an enum, an optional field, and a date (`chrono::NaiveDate` with the `serde` feature).
2. Write it to JSON and read it back. Test the round trip.
3. Read a CSV file into the same struct with the `csv` crate. Report the line number of a bad record.
4. Save the JSON file safely (topic 4.10). Add a `version` field. Write a test that reads a version 1 file after you add a field in version 2.

## Project: Ledger, stage 1

See [../projects/ledger.md](../projects/ledger.md), stage 1.

## Done criteria

- [ ] I can decide between an error and a panic, and explain the decision.
- [ ] I can design an error type for a library, and I use `anyhow` with context in applications.
- [ ] My error messages tell the user what failed, with which value, and why.
- [ ] My applications have a thin `main` and a library crate with tests.
- [ ] I can explain "parse, don't validate" and use newtypes that cannot hold invalid values.
- [ ] I know the main Rust API Guidelines and apply them to my public types.
- [ ] My library crates have documentation with doc tests.
- [ ] Ledger stage 1 is complete.

## Next

[Phase 5: Software design and SOLID](phase-5-software-design.md)
