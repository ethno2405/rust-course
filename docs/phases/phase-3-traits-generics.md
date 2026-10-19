# Phase 3: Traits, Generics, Closures, and Iterators

**Time:** 3 weeks
**Prerequisites:** Phase 2
**Code:** `labs/phase-3-traits/`

## Goal

At the end of this phase, you can:

- Define traits and implement them, with default methods and trait bounds.
- Write generic functions and types, and explain monomorphization.
- Select between static dispatch (generics) and dynamic dispatch (trait objects), and give the reason.
- Implement the common standard traits for your types.
- Explain the orphan rule and work around it with a newtype.
- Use closures and explain `Fn`, `FnMut`, and `FnOnce`.
- Write data pipelines with iterator adapters, and implement your own iterator.

## Reading

| Source | Part |
|---|---|
| The Rust Programming Language (the Book) | "Generic Types, Traits, and Lifetimes", "Functional Language Features: Iterators and Closures", "Object Oriented Programming Features", "Advanced Features" (only the parts about advanced traits) |
| Blandy, Orendorff, and Tindall, *Programming Rust*, 2nd edition | The chapters about traits and generics, utility traits, closures, and iterators |
| The standard library documentation | The module pages `std::iter`, `std::convert`, `std::cmp`, `std::fmt` |
| Rust API Guidelines (rust-lang.github.io/api-guidelines) | "Interoperability": the traits that a type must implement |
| David Drysdale, *Effective Rust* (effective-rust.com) | The items about standard traits, generics and trait objects, and iterator transforms |

## Topics

### 3.1 Traits

A trait defines behavior that many types can share. It is similar to an interface in Java, C#, Go, or TypeScript.

```rust
pub trait Describe {
    fn name(&self) -> String;

    // A default method. Implementors can use it or override it.
    fn describe(&self) -> String {
        format!("This is {}.", self.name())
    }
}

pub struct Account {
    pub label: String,
}

impl Describe for Account {
    fn name(&self) -> String {
        self.label.clone()
    }
}
```

**Differences from interfaces in other languages:**

- You can implement your trait for types that you did not write, including `i32` and `String`.
- You can implement a trait for all types that implement a different trait (a **blanket implementation**): `impl<T: Display> ToString for T`.
- A trait can have associated types and constants, and functions without `self` (for example a constructor).
- A trait is not a type. `Describe` alone is not a type. Use `impl Describe` or `dyn Describe` (topics 3.2 and 3.3).

### 3.2 Generics and static dispatch

```rust
use std::fmt::Display;

fn print_all<T: Display>(items: &[T]) {
    for item in items {
        println!("{item}");
    }
}

// The same function with `impl Trait` in the argument position.
fn print_one(item: impl Display) {
    println!("{item}");
}

// A `where` clause is easier to read when there are many bounds.
fn largest<T>(items: &[T]) -> Option<&T>
where
    T: PartialOrd,
{
    let mut largest = items.first()?;
    for item in items {
        if item > largest {
            largest = item;
        }
    }
    Some(largest)
}
```

- **Monomorphization:** the compiler makes one copy of a generic function for each concrete type that the program uses. The calls are direct and can be inlined. This is why generics in Rust have no run-time cost ("zero-cost abstraction").
- **The costs:** a longer compile time and a larger executable when you use a generic function with many types.
- `-> impl Trait` in the return position hides the concrete type from the caller. Iterators and closures often use it: `fn evens(v: &[i32]) -> impl Iterator<Item = &i32>`.

### 3.3 Trait objects and dynamic dispatch

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Circle { r: f64 }
struct Square { side: f64 }

impl Shape for Circle { fn area(&self) -> f64 { std::f64::consts::PI * self.r * self.r } }
impl Shape for Square { fn area(&self) -> f64 { self.side * self.side } }

fn total_area(shapes: &[Box<dyn Shape>]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}
```

- `dyn Shape` is a **trait object**. Its concrete type is known only at run time. A pointer to a trait object (`&dyn Shape`, `Box<dyn Shape>`, `Arc<dyn Shape>`) has two parts: a pointer to the data and a pointer to a **vtable** (a table of function pointers).
- Only **dyn-compatible** traits can be trait objects (older documents say "object safe"). Examples of methods that make a trait not dyn-compatible: a generic method, or a method that returns `Self`. Add `where Self: Sized` to such a method to exclude it from the trait object.

| | Generics (`T: Trait`, `impl Trait`) | Trait objects (`dyn Trait`) |
|---|---|---|
| The concrete type is selected | At compile time | At run time |
| Different types in one collection | No | Yes: `Vec<Box<dyn Shape>>` |
| Speed | Direct calls, can be inlined | An indirect call through the vtable |
| Code size | One copy for each type | One copy |
| Use it when | The type is known at compile time, or speed is important | You need a collection of different types, a plugin, or a smaller executable |

- **Enums are a third option.** If you know all the types now, and you do not need to add types from outside the crate, an `enum Shape { Circle(Circle), Square(Square) }` is often simpler. Phase 5, topic 5.3, compares enums and traits in more detail.

### 3.4 The common standard traits

Implement these traits when they apply to your type. Derive them when you can.

| Trait | Gives | Derive? |
|---|---|---|
| `Debug` | `{:?}` formatting. Implement it for every public type. | Yes |
| `Display` | `{}` formatting for users, and `to_string()` | No: write it |
| `Clone`, `Copy` | Explicit and implicit copies (phase 2) | Yes |
| `PartialEq`, `Eq` | `==`. `Eq` says that equality is reflexive (not true for `f64`: `NaN != NaN`). | Yes |
| `PartialOrd`, `Ord` | `<`, `>`, sorting. Derived order: fields from top to bottom, enum variants in declaration order. | Yes, if that order is correct |
| `Hash` | Use as a `HashMap` key. Must agree with `Eq`: equal values have equal hashes. | Yes |
| `Default` | `T::default()`, and `..Default::default()` in struct literals | Yes, if all fields have a meaningful default |
| `From<T>`, `Into<T>` | Conversions that cannot fail. Implement `From`; `Into` comes free. | No |
| `TryFrom<T>` | Conversions that can fail. Returns a `Result`. | No |
| `FromStr` | `"12.50".parse::<Money>()` | No |
| `AsRef<T>` | A cheap reference conversion. `fn open(path: impl AsRef<Path>)` | No |
| `Iterator`, `IntoIterator` | `for` loops and adapters (topic 3.8) | No |
| `Error` | An error type (phase 4) | No (use `thiserror`) |

- Implement `Deref` only for smart pointer types. Do not use `Deref` to imitate inheritance (phase 5, topic 5.12).

### 3.5 Associated types and generic parameters

- **Associated type:** each type implements the trait one time, with one choice for the associated type. `Iterator` has `type Item;`: a `Vec<i32>` iterator has exactly one item type.
- **Generic parameter:** a type can implement the trait many times, once for each parameter. `From<T>`: `String` implements `From<&str>`, `From<char>`, and others.
- Use an associated type when there is one natural choice for each implementing type.

### 3.6 Coherence and the orphan rule

- **Orphan rule:** you can implement a trait for a type only if the trait or the type is defined in your crate. You cannot implement `Display` for `Vec<T>`, because both come from the standard library.
- The rule makes sure that two crates cannot give two different implementations of the same trait for the same type.
- **Workarounds:**
  - **Newtype:** `struct Lines(Vec<String>);` then `impl Display for Lines`.
  - **Extension trait:** define your own trait with the methods that you want, and implement it for the foreign type. `trait SliceExt { fn total(&self) -> i64; } impl SliceExt for [i64] { ... }`.

### 3.7 Closures

A closure is an anonymous function that can capture variables from its environment.

```rust
let threshold = 100;
let is_large = |amount: i64| amount > threshold; // Captures `threshold` by reference.
let large: Vec<i64> = vec![50, 150, 300].into_iter().filter(|a| is_large(*a)).collect();
```

| Trait | The closure | Can be called |
|---|---|---|
| `FnOnce` | Can consume its captured values | One time |
| `FnMut` | Can change its captured values | Many times, not at the same time |
| `Fn` | Only reads its captured values | Many times, also at the same time |

- Every closure implements `FnOnce`. A closure that does not consume its captures also implements `FnMut`. A closure that also does not change them implements `Fn`.
- A function parameter should accept the most general trait that it can: `FnOnce` if you call it one time, `FnMut` if you call it many times, `Fn` only if you need shared calls (for example from many threads).
- `move |x| ...` moves the captured values into the closure. You need it when the closure lives longer than the current function, for example a closure for a new thread.
- **Closures as a design tool:** a closure is the simplest form of the strategy pattern (phase 5). `fn retry<T>(attempts: u32, op: impl FnMut() -> Result<T, Error>)`.
- To store closures of different types in one collection, use trait objects: `Vec<Box<dyn Fn(&Event)>>`.

### 3.8 Iterators

- An iterator implements `Iterator`. Its only required method is `fn next(&mut self) -> Option<Self::Item>`.
- Iterators are **lazy**: an adapter does nothing until a consumer pulls the items.
- Three ways to iterate over a collection:

  | Call | Items | The collection after the loop |
  |---|---|---|
  | `v.iter()` or `for x in &v` | `&T` | Unchanged |
  | `v.iter_mut()` or `for x in &mut v` | `&mut T` | Changed in place |
  | `v.into_iter()` or `for x in v` | `T` | Moved, cannot be used |

- Adapters (return a new iterator): `map`, `filter`, `filter_map`, `flat_map`, `enumerate`, `zip`, `chain`, `take`, `skip`, `take_while`, `skip_while`, `peekable`, `rev`, `inspect`, `cloned`, `copied`.
- Consumers (use the iterator): `collect`, `sum`, `product`, `count`, `fold`, `for_each`, `any`, `all`, `find`, `position`, `min`, `max`, `min_by_key`, `max_by`, `last`, `nth`, `partition`.
- `collect` can make any type that implements `FromIterator`: `Vec`, `HashMap`, `String`, `HashSet`. It can also collect an iterator of `Result<T, E>` into `Result<Vec<T>, E>`: it stops at the first error.

```rust
let lines = ["10", "20", "x", "40"];
let parsed: Result<Vec<i32>, _> = lines.iter().map(|s| s.parse::<i32>()).collect();
assert!(parsed.is_err());
```

- **Performance:** in a release build, an iterator chain is usually as fast as a hand-written loop. Sometimes it is faster, because the compiler can remove the bounds checks.
- **Readability:** use an iterator chain when it reads as a description of the data flow. Use a `for` loop when the logic has many branches, early returns, or side effects.

### 3.9 Operator overloading

- Operators are traits in `std::ops`: `Add`, `Sub`, `Mul`, `Neg`, `Index`, `AddAssign`, and others.
- Implement them only when the meaning is clear and the same as for numbers. `Money + Money` is clear. `Account + Transaction` is not clear: write a method `account.apply(tx)`.
- An operator cannot return a `Result`. If the operation can fail (for example two different currencies), use a method: `fn checked_add(self, other: Money) -> Result<Money, CurrencyMismatch>`.

### 3.10 Generic API design

- Accept the most general input that costs nothing: `&str` in place of `&String`, `&[T]` in place of `&Vec<T>`, `impl AsRef<Path>` for paths, `impl IntoIterator<Item = T>` for sequences.
- Return concrete types or `impl Trait`. Do not make the return type generic without a reason.
- **Do not make code generic too early.** A generic function with one caller and one type is harder to read and gives no benefit. Make it generic when the second use appears (phase 5, topic 5.8: YAGNI).

### 3.11 `Sized`, marker traits, and `?Sized`

- `Sized`: the size is known at compile time. All generic parameters are `Sized` by default. `str`, `[T]`, and `dyn Trait` are not `Sized`. You use them only behind a pointer.
- `T: ?Sized` removes the default bound. Use it to accept `str` or `dyn Trait` behind a reference: `fn show<T: ?Sized + Display>(x: &T)`.
- **Marker traits** have no methods. They say something about the type: `Copy`, `Send`, `Sync` (phase 6), `Sized`, `Unpin` (phase 9).

## Exercises

Make the workspace `labs/phase-3-traits/`. Exercises marked **core** are for all learners.

### Exercise 3.1: Static and dynamic dispatch (core)

1. Define a trait `Shape` with `area` and `name`. Implement it for three shapes.
2. Write `fn largest_static<T: Shape>(shapes: &[T]) -> Option<&T>` and `fn largest_dynamic(shapes: &[Box<dyn Shape>]) -> Option<&dyn Shape>`.
3. Make a list with different shapes. Which function can you use with it? Why?
4. Write the same design with an `enum`. Add a new operation (`perimeter`) to both designs. Then add a new shape to both designs. Count the places that you must change in each design. Write the result in your engineering notebook. You use it again in phase 5.

### Exercise 3.2: Standard traits for a value type (core)

Write a type `Duration` for a time duration in whole minutes (do not use `std::time::Duration`).

1. Implement `Display` (format `1h 05m`), `FromStr` (accept `"1h 05m"`, `"90m"`, `"2h"`), `Add`, `Sub`, `Default`, `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `PartialOrd`, `Ord`, and `Hash`.
2. `Sub` must not give a negative duration. Decide what to do and document the decision. Is an operator the correct API for this? Read topic 3.9 again.
3. Implement `Sum`, so that `durations.iter().copied().sum::<Duration>()` works.
4. Test the round trip: `d.to_string().parse::<Duration>() == Ok(d)`.

### Exercise 3.3: A generic container

1. Write `struct TopN<T> { capacity: usize, items: Vec<T> }` that keeps only the N largest items that you push.
2. Which bounds does `T` need? Put the bounds only on the `impl` blocks that need them, not on the struct.
3. Implement `IntoIterator` for `TopN<T>` and for `&TopN<T>`.

### Exercise 3.4: Log analysis with iterators (core)

Read a log file with lines like `2026-10-19T10:15:00Z ERROR payment: card declined`.

1. Count the lines for each level.
2. Find the 5 most frequent error messages.
3. Find the first and the last time stamp of each component (`payment`, `auth`, ...).
4. Skip lines that do not match the format, and count them.

Use iterator adapters, not index loops. Each function takes `impl Iterator<Item = &str>` or `&str`, so that the tests do not need a file.

### Exercise 3.5: Your own iterator (core)

1. Write an iterator over the Fibonacci numbers of type `u64`. It stops (returns `None`) before an overflow. Use `checked_add`.
2. Write `struct Pairs<'a, T>` that gives all pairs of neighbor elements of a slice: `[1, 2, 3]` gives `(1, 2)` and `(2, 3)`. Then find the standard method that does the same.
3. Write a function `fn evens() -> impl Iterator<Item = u64>` with `std::iter::successors` or `(0..).step_by(2)`.

### Exercise 3.6: Event dispatcher with closures

1. Write `struct Dispatcher { handlers: HashMap<String, Vec<Box<dyn Fn(&Event)>>> }`.
2. `on(event_name, handler)` registers a handler. `emit(event)` calls all handlers for the name of the event.
3. Register a handler that counts events in a `Rc<Cell<u32>>` that the test can read.
4. Why does the handler need `Fn` and not `FnMut` here? What changes if you want `FnMut`?

### Exercise 3.7: Orphan rule

1. Try to implement `Display` for `Vec<Duration>` (from exercise 3.2). Read the error.
2. Fix it with a newtype `struct Timesheet(Vec<Duration>)`.
3. Write an extension trait `DurationSliceExt` with a method `total()` for `[Duration]`.

### Exercise 3.8: Measure an iterator chain

1. Sum the squares of the even numbers from 1 to 10 million with an iterator chain, and with a `for` loop with an `if`.
2. Measure both in debug and in release builds with `std::time::Instant`. Explain the results.
3. Open the code in Compiler Explorer (godbolt.org) with `-C opt-level=3`. Compare the assembly of the two functions.

## Done criteria

- [ ] I can define a trait with a default method and implement it for my types and for foreign types.
- [ ] I can explain monomorphization and its costs.
- [ ] I can select between generics, trait objects, and an enum, and give the reasons.
- [ ] I implement the common standard traits for my value types.
- [ ] I can explain the orphan rule and use a newtype or an extension trait.
- [ ] I can explain `Fn`, `FnMut`, and `FnOnce` and select the correct bound for a parameter.
- [ ] I can write a data pipeline with iterator adapters, and collect results into `Result<Vec<_>, _>`.

## Next

[Phase 4: Errors, modules, and API design](phase-4-errors-apis.md)
