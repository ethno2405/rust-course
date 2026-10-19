# Phase 0: Tools and Workflow

**Time:** 1 week. Plan 2 weeks if git and the terminal are new to you.
**Prerequisites:** Programming experience in at least one language.
**Code:** `labs/phase-0-tools/`

## Goal

At the end of this phase, you can:

- Install and update the Rust toolchain with `rustup`.
- Make, build, run, and test a Cargo project.
- Use an editor with rust-analyzer: completion, inline types, go to definition, rename.
- Read a compiler error and find the explanation of its error code.
- Format code with `rustfmt` and find problems with `clippy`.
- Find documentation for the standard library and for crates.
- Stop a program at a breakpoint and inspect its variables.
- Use git for small commits and branches.
- Evaluate a crate before you add it as a dependency.

## Reading

| Source | Part |
|---|---|
| [../software-setup.md](../software-setup.md) | All |
| The Rust Programming Language (the Book) | "Getting Started" |
| The Cargo Book (doc.rust-lang.org/cargo) | "Getting Started" and "Cargo Guide" |
| Clippy documentation (doc.rust-lang.org/clippy) | "Usage" and "Configuration" |
| Pro Git (git-scm.com/book) | "Git Basics" and "Git Branching" (only if git is new to you) |

## Topics

### 0.1 The Rust toolchain

- `rustup` installs and updates the toolchain. The toolchain contains `rustc` (the compiler), `cargo` (the build tool and package manager), `rustfmt`, `clippy`, the standard library, and its documentation.
- **Release channels:** `stable` (a new version every 6 weeks), `beta`, and `nightly` (unstable features). The course uses `stable`.
- **Editions:** an edition is a set of language rules: 2015, 2018, 2021, 2024. A new edition can change the syntax without a break of old code: each crate selects its edition, and crates of different editions work together. The course uses edition 2024 (Rust 1.85 or newer).
- Useful commands:

  | Command | Result |
  |---|---|
  | `rustup update` | Install the newest stable version |
  | `rustup show` | Show the installed toolchains and the active one |
  | `rustup component add rust-analyzer` | Add a component |
  | `rustup doc --std` | Open the standard library documentation offline |
  | `rustup doc --book` | Open the Book offline |

### 0.2 Cargo

- A **package** is a directory with a `Cargo.toml` file. It contains one or more **crates**. A crate is one compilation unit: a library (`src/lib.rs`) or a binary (`src/main.rs`).
- `Cargo.toml` describes the package: name, version, edition, and dependencies. `Cargo.lock` records the exact versions of all dependencies. Commit both files.
- Commands:

  | Command | Result |
  |---|---|
  | `cargo new <name>` | Make a binary package. Add `--lib` for a library. |
  | `cargo check` | Check the code for errors. Faster than `build`, makes no executable. |
  | `cargo build` | Build a debug executable in `target/debug/` |
  | `cargo build --release` | Build an optimized executable in `target/release/` |
  | `cargo run -- <args>` | Build and run. The arguments after `--` go to your program. |
  | `cargo test` | Build and run all tests |
  | `cargo add <crate>` | Add a dependency to `Cargo.toml` |
  | `cargo doc --open` | Build the documentation of your code and of all dependencies |
  | `cargo tree` | Show the dependency tree |
  | `cargo clean` | Delete `target/` |

- **Debug and release builds:** a debug build compiles fast and has extra checks. For example, integer overflow causes a panic in a debug build. A release build is optimized. Integer overflow wraps around in a release build, unless you enable `overflow-checks`.
- `target/` contains build outputs. It can be large. Never commit it.

### 0.3 The editor

- **rust-analyzer** is the language server for Rust. VS Code, RustRover, Neovim, Zed, Helix, and other editors use it (RustRover has its own engine).
- Learn these functions in your editor: completion, inline type hints, go to definition, find all references, rename a symbol, show the documentation on hover, quick fixes (the "light bulb"), and run a single test.
- Configure the editor to run `rustfmt` when you save a file.
- Configure rust-analyzer to run `clippy` in place of `cargo check`. In VS Code, set `"rust-analyzer.check.command": "clippy"`.

### 0.4 Read compiler errors

The Rust compiler gives long, helpful error messages. Learn to read them from the first day.

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:3:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
  |
help: consider making this binding mutable
  |
2 |     let mut x = 5;
  |         +++
```

| Part | Content |
|---|---|
| `error[E0384]` | The error code. Run `rustc --explain E0384` for a long explanation with examples. |
| `--> src/main.rs:3:5` | The file, the line, and the column |
| Lines with `-` and `^` | The spans: the parts of the code that cause the error, with labels |
| `help:` | A suggestion. The compiler often shows the exact change. |

- Read the **first** error first. Later errors are often results of the first error.
- `warning:` lines do not stop the build, but read them. The course treats warnings as errors from phase 1.

### 0.5 Format and lint

- `cargo fmt` formats all code with the standard style. Nobody in a Rust team discusses the code format. Use the default settings.
- `cargo clippy` finds common mistakes and code that is not idiomatic. It has more than 700 lints in groups: `correctness`, `suspicious`, `style`, `complexity`, `perf` (on by default), and `pedantic`, `restriction`, `nursery` (off by default).
- Run clippy with `cargo clippy --all-targets -- -D warnings`. `--all-targets` also checks tests and examples. `-D warnings` makes each warning an error.
- Each clippy message has a link to the lint documentation. Read it: it explains why the lint exists.

### 0.6 Documentation

| Source | Content |
|---|---|
| doc.rust-lang.org/std | The standard library. Use the search field at the top. Every type has a "Source" link. |
| docs.rs | The documentation of every crate on crates.io, for every version |
| `cargo doc --open` | The documentation of your project and its exact dependency versions, offline |
| doc.rust-lang.org/reference | The Rust Reference: the exact language rules |
| The Rust API Guidelines | How a good Rust library looks (phase 4) |

- The documentation of a type lists its methods, then the traits that it implements. Many useful methods come from traits: look at the "Trait Implementations" section.
- Examples in the documentation are tested code. They compile and run.

### 0.7 Debugging

- `dbg!(expr)` prints the file, the line, the expression, and its value to stderr, then returns the value. Use it for quick checks. Remove it before you commit.
- `RUST_BACKTRACE=1 cargo run` prints the call stack when the program panics. In PowerShell, set `$env:RUST_BACKTRACE=1` first.
- A **debugger** stops the program at a breakpoint and shows the variables. VS Code uses the CodeLLDB extension. RustRover has a built-in debugger. Debug a debug build: optimized code is hard to follow.
- In VS Code, rust-analyzer shows "Run | Debug" above `main` and above each test. Click "Debug" to start the debugger.

### 0.8 Git basics

Skip this topic if you use git every day.

- The three areas: the working tree, the index (staging area), and the repository.
- Commands: `git status`, `git add`, `git commit`, `git log --oneline`, `git diff`, `git diff --staged`, `git restore`.
- Make small commits. One commit = one logical change that builds and passes its tests.
- Write the commit message in the imperative, as one line: "Add the balance function", not "added stuff".
- `cargo new` makes a git repository and a `.gitignore` file with `/target`. In this course, the labs are in the course repository, so Cargo does not make a new repository there.

### 0.9 Git branches and pull requests

- A branch is a pointer to a commit. `git switch -c <name>` makes a branch and switches to it.
- `git merge` joins two branches. `git rebase` replays your commits on a new base. Do not rebase commits that other people use.
- A **pull request** (PR) asks the team to review and merge a branch. Continuous integration (CI) runs the tests on the PR. Phase 8 teaches CI and code review.
- In this course, your work is on your track branch (see the main README).

### 0.10 Crates and dependencies

- crates.io is the registry of Rust crates. lib.rs is a different index of the same crates, with categories and rankings.
- Before you add a crate, check:

  | Check | Where |
  |---|---|
  | Downloads and the number of dependent crates | crates.io |
  | The date of the last release, and open issues | The repository |
  | The license (most crates use MIT or Apache-2.0) | `Cargo.toml` of the crate |
  | The quality of the documentation | docs.rs |
  | The number of its own dependencies | `cargo tree` after you add it |
  | Use of `unsafe` | The source, or the tool `cargo-geiger` |

- **Version requirements:** `serde = "1.0"` means "1.0.0 or newer, but less than 2.0.0". Cargo selects the newest compatible version and writes it to `Cargo.lock`.
- **Features:** a crate can have optional parts. Enable them with `cargo add tokio --features full`. Enable only the features that you need.
- **Supply chain risk:** a crate can run code on your computer at build time (build scripts and procedural macros). Use well-known crates. Check the exact name: some attackers publish crates with names similar to popular crates.

## Exercises

Make the workspace `labs/phase-0-tools/` with a `Cargo.toml` file that lists the exercise packages. Ask the mentor if you do not know how. Phase 1, topic 1.9, explains workspaces in more detail.

### Exercise 0.1: Install and check

1. Install the toolchain and the editor as [../software-setup.md](../software-setup.md) describes.
2. Run the checks in section 2 of the setup document. Write the versions in your engineering notebook.
3. Open the Book and the standard library documentation offline with `rustup doc`.

### Exercise 0.2: The first project

1. Make the package `ex-0-02-hello` in the workspace.
2. Run it with `cargo run`. Find the executable in `target/`.
3. Change the program: read your name from the first command-line argument (`std::env::args().nth(1)`) and print a greeting. Print a usage message if the argument is missing.
4. Add a function `greeting(name: &str) -> String` and a test for it. Run `cargo test`.
5. Build a release version. Compare the file size of the debug and the release executables.

### Exercise 0.3: Read compiler errors

Copy each fragment into a `main` function. Before you compile it, predict the error. Compile it, read the full message, run `rustc --explain` for the code, and fix it.

```rust
let count = 0;
count += 1;
```

```rust
let total: i32 = "42";
```

```rust
let name = String::from("Ada");
let other = name;
println!("{name}");
```

```rust
let numbers = vec![1, 2, 3];
let first = numbers[3];
println!("{first}");
```

The last fragment compiles. What happens when you run it? Why does the compiler not find this error?

### Exercise 0.4: Clippy

1. Copy this function into a new package `ex-0-04-clippy`, call it from `main`, and run `cargo clippy`.

   ```rust
   fn average(values: &Vec<f64>) -> f64 {
       let mut sum = 0.0;
       for i in 0..values.len() {
           sum = sum + values[i];
       }
       if values.len() == 0 {
           return 0.0;
       }
       return sum / values.len() as f64;
   }
   ```

2. Read the documentation of each lint that clippy reports. Fix each finding.
3. Run `cargo clippy -- -W clippy::pedantic`. Read the new findings. Decide for each one whether the change makes the code better.

### Exercise 0.5: Debugger

1. Write a function that computes the factorial of a number in a loop. Call it in `main` with the value 10.
2. Set a breakpoint in the loop. Start the debugger. Step through three iterations and watch the variables.
3. Call the function with 25. Run the debug build, then the release build. Explain the difference. (Hint: topic 0.2.)

### Exercise 0.6: Evaluate crates

1. Find two crates for date and time on crates.io (for example `chrono` and `time`).
2. Compare them with the table in topic 0.10. Write the result in your engineering notebook.
3. Select one. Write two sentences about why.

## Done criteria

- [ ] `rustc`, `cargo`, `rustfmt`, `clippy`, and rust-analyzer work on my computer.
- [ ] I can make, build, run, and test a package, and I know where Cargo puts the executable.
- [ ] I can explain the difference between a debug and a release build.
- [ ] I can read a compiler error and find the long explanation of its code.
- [ ] My code has no `cargo clippy` warnings and is formatted with `cargo fmt`.
- [ ] I can stop a program at a breakpoint and inspect a variable.
- [ ] I can evaluate a crate before I add it.

## Next

[Phase 1: Rust basics](phase-1-rust-basics.md)
