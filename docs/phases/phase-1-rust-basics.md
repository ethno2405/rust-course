# Phase 1: Rust Basics

**Time:** 1 week if you already wrote small Rust programs. 2 weeks if you know a language with a strong static type system (C, C++, Swift, Kotlin, Haskell, F#). 3 weeks if you know C#, Java, Go, or TypeScript. 4–6 weeks if you know only dynamic languages.
**Prerequisites:** Phase 0.
**Code:** `labs/phase-1-basics/`

## Goal

At the end of this phase, you can:

- Write small Rust programs with functions, structs, enums, and pattern matching.
- Use `Option` and `Result` in place of `null` and exceptions.
- Use the standard collections: `Vec`, `String`, `HashMap`, `HashSet`, `BTreeMap`.
- Split a program into modules and control the visibility of items.
- Write unit tests for each function that you write.
- Write code that `rustfmt` and `clippy` accept, with idiomatic names.

## Reading

| Source | Part |
|---|---|
| The Rust Programming Language (the Book) | "Programming a Guessing Game", "Common Programming Concepts", "Using Structs to Structure Related Data", "Enums and Pattern Matching", "Managing Growing Projects with Packages, Crates, and Modules", "Common Collections". Read "Understanding Ownership" quickly: phase 2 explains it in detail. |
| Rust by Example (doc.rust-lang.org/rust-by-example) | The sections for the same subjects. Use it when you prefer examples to text. |
| Rustlings (github.com/rust-lang/rustlings) | Small exercises with automatic checks. Do them in parallel with the Book. |
| Comprehensive Rust (google.github.io/comprehensive-rust) | The course from Google. A fast alternative to the Book for experienced developers. |
| Exercism, Rust track (exercism.org/tracks/rust) | More exercises, with optional reviews from human mentors |

## Topics

### 1.0 Static types and compilation (only for learners of dynamic languages)

Skip this topic if you know a statically typed language (C#, Java, Go, TypeScript, C, C++).

- **Static types:** the compiler knows the type of every value before the program runs. It rejects a program that uses a value with a wrong type. In Python or JavaScript, you find this error only when the line runs.
- **Type inference:** you do not write most types. `let x = 5;` is an `i32`. The compiler finds the type from the use of the value. You must write the types of function parameters and return values.
- **Compilation:** `cargo build` translates the whole program to machine code before it runs. Many errors that a dynamic language finds at run time (or never), the compiler finds at compile time.
- **The cost:** you must satisfy the compiler before the program runs. This feels slow at first. The benefit is that a program that compiles has fewer defects, and you can change large programs safely.

### 1.1 Rust compared with other languages

| Feature in other languages | Examples | Rust replacement |
|---|---|---|
| Classes with fields and methods | C#, Java, Python, C++ | A `struct` (data) and an `impl` block (methods) |
| Inheritance | C#, Java, Python, C++ | None. Use traits (shared behavior) and composition (a struct that contains other structs). Phase 5 explains why. |
| Interfaces | C#, Java, Go, TypeScript | Traits (phase 3) |
| `null`, `nil`, `None`, `undefined` | All | `Option<T>`: `Some(value)` or `None`. The compiler makes you handle `None`. |
| Exceptions | C#, Java, Python, C++, JavaScript | `Result<T, E>` for expected errors. `panic!` for bugs. |
| Garbage collector | C#, Java, Python, Go, JavaScript | Ownership: the compiler inserts the cleanup at the end of the owner's scope (phase 2) |
| `switch` | C#, Java, C, Go, JavaScript | `match`: it must handle all cases, and it can destructure data |
| Enums that are only named numbers | C#, Java, C, TypeScript | Rust enums can carry data in each variant (algebraic data types) |
| Packages, namespaces | All | Crates and modules |
| `private`, `public` | C#, Java, C++, TypeScript | Items are private by default. `pub` makes them public. |
| Method overloading | C#, Java, C++ | None. Use different names, generics, or traits. |
| Default parameters | C#, Python, C++, TypeScript | None. Use `Default`, a builder (phase 4), or two functions. |
| Lambdas | All | Closures: `\|x\| x + 1` (phase 3) |
| Generics | C#, Java, Go, TypeScript, C++ | Generics with trait bounds (phase 3) |
| Reflection, annotations | C#, Java, Python | None at run time. `derive` macros generate code at compile time. |

**Notes for programmers of managed languages (C#, Java, Go):**

- Values are not references by default. `let b = a;` moves or copies the value. It does not make a second reference to the same object.
- There is no `null`. Every variable has a valid value.
- A `struct` lives on the stack unless you put it in a heap container (`Box`, `Vec`, `String`).

**Notes for programmers of dynamic languages (Python, JavaScript):**

- A `Vec<T>` holds values of one type only. To hold different types, use an `enum` with one variant for each type.
- Integers have fixed sizes (`i32`, `u64`). They can overflow (topic 1.3).
- Strings are UTF-8 bytes. You cannot get a character with `s[3]` (topic 1.8).

**Notes for C and C++ programmers:**

- The compiler checks the lifetime of every reference. There are no dangling references in safe Rust.
- Move is the default for assignment, and a moved-from variable cannot be used. There is no "valid but unspecified" state.
- There is no undefined behavior in safe Rust. Array indexes are checked at run time.

### 1.2 Variables and mutability

- `let x = 5;` makes an immutable binding. `let mut x = 5;` makes a mutable binding.
- **Shadowing:** `let x = x + 1;` makes a new variable with the same name. The new variable can have a different type. Use it to change a value step by step: `let input = input.trim();`.
- `const MAX_ITEMS: usize = 100;` is a compile-time constant. It must have a type.
- `static` items have one fixed address for the whole program. You rarely need them.

### 1.3 Types

| Type | Examples | Notes |
|---|---|---|
| Signed integers | `i8`, `i16`, `i32`, `i64`, `i128`, `isize` | `i32` is the default for integer literals |
| Unsigned integers | `u8`, `u16`, `u32`, `u64`, `u128`, `usize` | `usize` is for sizes and indexes |
| Floating point | `f32`, `f64` | `f64` is the default. Never use them for money. |
| Boolean | `bool` | |
| Character | `char` | One Unicode scalar value, 4 bytes. Not a byte. |
| Tuple | `(i32, &str)` | Access with `t.0`, `t.1` |
| Array | `[i32; 4]` | Fixed length, known at compile time |

- **Integer overflow:** in a debug build, overflow causes a panic. In a release build, the value wraps around. If overflow is possible, say what you want:

  | Method | Result for `u8` 250 + 10 |
  |---|---|
  | `250u8.checked_add(10)` | `None` |
  | `250u8.wrapping_add(10)` | `4` |
  | `250u8.saturating_add(10)` | `255` |
  | `250u8.overflowing_add(10)` | `(4, true)` |

- **Conversions:** `as` converts numbers, but it can lose data silently: `300_i32 as u8` is `44`. Use `u8::try_from(x)` when the value can be out of range. Use `From` (`i64::from(x)`) when the conversion is always safe.

### 1.4 Functions, expressions, and control flow

- A block `{ ... }` is an expression. Its value is the last expression without a semicolon.
- `if` is an expression: `let label = if n < 0 { "negative" } else { "positive" };`.
- `loop` can return a value with `break value;`.
- `for item in collection` iterates over anything that implements `IntoIterator`. Use `for i in 0..n` for ranges. Prefer iteration over the items to loops over indexes.
- A function returns the value of its last expression. Use `return` only to return early.

```rust
fn classify(n: i32) -> &'static str {
    if n == 0 {
        return "zero";
    }
    if n % 2 == 0 { "even" } else { "odd" }
}
```

### 1.5 Structs and methods

```rust
#[derive(Debug, Clone, PartialEq)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // An associated function. Call it with Rectangle::square(5).
    fn square(size: u32) -> Self {
        Self { width: size, height: size }
    }

    // A method. `&self` borrows the value.
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // `&mut self` borrows the value mutably.
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
}
```

- `new` is the usual name for the main constructor. It is a convention, not a keyword.
- **Tuple structs** have unnamed fields: `struct Meters(f64);`. Use them for newtypes (phase 4).
- `#[derive(...)]` generates trait implementations: `Debug` (print with `{:?}`), `Clone`, `PartialEq` (`==`), and others. Phase 3 explains traits.

### 1.6 Enums and pattern matching

A Rust enum is a type that is one of several variants. Each variant can carry different data.

```rust
enum Command {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
}

fn describe(command: &Command) -> String {
    match command {
        Command::Quit => "quit".to_string(),
        Command::Move { x: 0, y } => format!("move vertically by {y}"),
        Command::Move { x, y } => format!("move to ({x}, {y})"),
        Command::Write(text) if text.is_empty() => "write nothing".to_string(),
        Command::Write(text) => format!("write '{text}'"),
    }
}
```

- `match` must be **exhaustive**: it handles every variant. When you add a variant, the compiler shows every `match` that you must update.
- Patterns can destructure structs, tuples, and enums, match literal values (`x: 0`), and have guards (`if text.is_empty()`).
- `if let Some(x) = value { ... }` handles one pattern.
- `let Some(x) = value else { return; };` (let-else) handles one pattern and leaves the function in all other cases.
- `matches!(value, Pattern)` gives a `bool`.
- Do not use `_ =>` as a habit. It hides new variants from the compiler.

### 1.7 `Option` and `Result`

```rust
enum Option<T> { Some(T), None }
enum Result<T, E> { Ok(T), Err(E) }
```

- `Option<T>` replaces `null`. `Result<T, E>` replaces exceptions for errors that can occur in normal operation.
- Useful methods:

  | Method | Result |
  |---|---|
  | `opt.unwrap_or(default)` | The value, or the default |
  | `opt.map(\|v\| v * 2)` | Change the value inside `Some` |
  | `opt.ok_or("missing")` | Convert `Option` to `Result` |
  | `res.ok()` | Convert `Result` to `Option` |
  | `res.map_err(\|e\| e.to_string())` | Change the error |
  | `value?` | Return early with `None` or `Err`. Phase 4 explains the details. |

- `unwrap()` and `expect("message")` give the value or panic. Use them in tests and in throwaway code. Phase 4 gives the rules for production code.

### 1.8 Collections and strings

| Type | Use |
|---|---|
| `Vec<T>` | A growable list |
| `String` | A growable, owned UTF-8 text |
| `&str` | A borrowed view of UTF-8 text. String literals are `&'static str`. |
| `HashMap<K, V>` | Key-value pairs, no order |
| `BTreeMap<K, V>` | Key-value pairs, sorted by key |
| `HashSet<T>`, `BTreeSet<T>` | Unique values |
| `VecDeque<T>` | A queue with fast insert and remove at both ends |

- A `String` is UTF-8 bytes. A character can use 1–4 bytes, so `s[3]` does not compile. Use `s.chars()` for characters, and `s.bytes()` for bytes. `s.len()` is the number of bytes, not characters.
- The **entry API** updates a map in one lookup:

  ```rust
  use std::collections::HashMap;

  let mut counts: HashMap<&str, usize> = HashMap::new();
  for word in "a b a c a".split_whitespace() {
      *counts.entry(word).or_insert(0) += 1;
  }
  ```

- `v.get(i)` returns `Option<&T>`. `v[i]` panics if `i` is out of range.

### 1.9 Modules, crates, and workspaces

- `mod inventory;` in `main.rs` or `lib.rs` loads `src/inventory.rs` (or `src/inventory/mod.rs`).
- Items are **private** by default: only the module and its children can use them. `pub` makes an item public. A `pub struct` with private fields can only be made by functions in its module. Use this to protect the rules of the type (its invariants).
- `use crate::inventory::Item;` brings a name into scope. `crate::` is the root of the current crate. `super::` is the parent module.
- A **workspace** is a set of packages that share one `Cargo.lock` and one `target/` directory. The root `Cargo.toml` has a `[workspace]` table with a `members` list. Each phase directory in `labs/` is a workspace.

### 1.10 Tests from the first day

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn adds_two_positive_numbers() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    #[should_panic]
    fn panics_on_overflow_in_debug_builds() {
        add(i32::MAX, 1);
    }
}
```

- `#[cfg(test)]` compiles the module only for `cargo test`.
- `assert!`, `assert_eq!`, and `assert_ne!` check a condition. Give a message for complex checks: `assert!(x > 0, "x was {x}")`.
- A test name says what the code does, not the name of the function: `adds_two_positive_numbers`, not `test_add`.
- `cargo test <part of a name>` runs only the matching tests. `cargo test -- --nocapture` shows the output of `println!`.
- Write the test first when you can. Then you know what the function must do before you write it.

### 1.11 Formatting and printing

- `println!("{value}")` uses `Display`, for users. `println!("{value:?}")` uses `Debug`, for developers. `{value:#?}` prints `Debug` on many lines.
- `format!` makes a `String`. `eprintln!` prints to stderr.
- Format options: `{:>8}` (align right, width 8), `{:.2}` (2 decimal places), `{:08.3}`, `{:x}` (hexadecimal).

### 1.12 Names and style

| Item | Style | Example |
|---|---|---|
| Variables, functions, modules | `snake_case` | `total_amount`, `parse_line` |
| Types, traits, enum variants | `UpperCamelCase` | `Transaction`, `ParseError` |
| Constants, statics | `SCREAMING_SNAKE_CASE` | `MAX_RETRIES` |

- Name conversion methods with these prefixes: `as_` (cheap, borrowed view), `to_` (expensive, or a new owned value), `into_` (consumes `self`). A method that returns a `bool` starts with `is_` or `has_`.
- Getters have no `get_` prefix: `fn name(&self) -> &str`.
- Good names are the first design tool. A name says what a value means in the domain, not its type: `unpaid_invoices`, not `vec2`.

## Exercises

Make the workspace `labs/phase-1-basics/`. Each exercise is a package in it. Write tests for every function. The workspace must pass `cargo clippy --all-targets -- -D warnings`.

### Exercise 1.1: Temperature converter

Write `celsius_to_fahrenheit` and `fahrenheit_to_celsius`. Write a program that reads a value and a unit from the command line, for example `cargo run -- 100 C`. Test the freezing point, the boiling point, and -40.

### Exercise 1.2: FizzBuzz with `match`

Write `fizzbuzz(n: u32) -> String`. Use one `match` on the tuple `(n % 3, n % 5)`. Do not use `if`.

### Exercise 1.3: Word frequency

1. Read a text file whose path is the first argument.
2. Count each word. Ignore case and punctuation.
3. Print the 10 most frequent words with their counts, sorted by count and then by word.
4. Test the counting function with a short text.

### Exercise 1.4: Shapes

1. Define `enum Shape { Circle { radius: f64 }, Rectangle { width: f64, height: f64 }, Triangle { a: f64, b: f64, c: f64 } }`.
2. Write `area` and `perimeter` as methods.
3. Add a function `fn new_triangle(a: f64, b: f64, c: f64) -> Option<Shape>` that returns `None` if the sides cannot make a triangle.
4. Add a fourth variant. Watch which `match` expressions the compiler asks you to change.

### Exercise 1.5: Calculator

Write `fn evaluate(input: &str) -> Result<i64, String>` for expressions with two numbers and one operator: `"3 + 4"`, `"10 / 2"`, `"-7 * 6"`.

- Return an error for a missing number, an unknown operator, division by zero, and overflow (use `checked_` methods).
- Write one test for each error case.
- Write a loop in `main` that reads lines from stdin and prints the result or the error.

### Exercise 1.6: Library module

1. Make a module `library` with a `pub struct Library` and a `pub struct Book`. The fields are private.
2. Write public functions to add a book, to borrow a book by its title, to return a book, and to list the available books.
3. A book that is borrowed cannot be borrowed again. Return a `Result` with a clear error.
4. In `main`, try to change a field of `Book` directly. Read the compiler error. Explain why private fields protect the rules of the library.

### Exercise 1.7: Rustlings

Install Rustlings (`cargo install rustlings`, then `rustlings init` in a directory outside the course repository). Do the exercises to the end of the section about error handling. Write the exercises that were hard in your engineering notebook.

### Exercise 1.8: Vending machine

Model a vending machine as a state machine with enums.

1. States: `Idle`, `HasMoney { cents: u32 }`, `Dispensing { product: Product }`.
2. Events: `InsertCoin(u32)`, `Select(Product)`, `Cancel`, `TakeProduct`.
3. Write `fn next(state: State, event: Event) -> (State, Option<Output>)` with one `match` on `(state, event)`. The output can be "return change" or "show an error".
4. Test every transition, including the events that are not valid in a state.

Compare this design with a design that uses several `bool` fields (`has_money`, `is_dispensing`). Which design permits impossible combinations?

## Project: Ledger, stage 0

See [../projects/ledger.md](../projects/ledger.md), stage 0.

## Done criteria

- [ ] I can write structs and enums with methods, and use `match` with destructuring and guards.
- [ ] I can use `Option` and `Result` without `unwrap()` in the normal code path.
- [ ] I can explain why `s[3]` does not compile for a `String`.
- [ ] I can split a package into modules and make fields private.
- [ ] I know what happens on integer overflow in debug and release builds, and how to select the behavior.
- [ ] Every function in my labs has a test with a name that describes its behavior.
- [ ] My vending machine cannot be in an impossible state.
- [ ] Ledger stage 0 is complete.

## Next

[Phase 2: Ownership, borrowing, and lifetimes](phase-2-ownership.md)
