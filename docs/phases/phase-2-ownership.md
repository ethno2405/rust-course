# Phase 2: Ownership, Borrowing, and Lifetimes

**Time:** 2–3 weeks
**Prerequisites:** Phase 1
**Code:** `labs/phase-2-ownership/`

## Goal

At the end of this phase, you can:

- Explain the three ownership rules and what a move is.
- Explain the borrow rules: many shared references, or one mutable reference.
- Read and fix borrow checker errors by a change of the design, not only with `clone()`.
- Read and write lifetime annotations, and explain what they mean.
- Use `Box`, `Rc`, `Arc`, `Weak`, `Cell`, and `RefCell`, and know when each one is the correct tool.
- Use `Drop` for deterministic cleanup (RAII).
- Select the parameter types of a function from the ownership that it needs.

## Reading

| Source | Part |
|---|---|
| The Rust Programming Language (the Book) | "Understanding Ownership", "Generic Types, Traits, and Lifetimes" (only the part about lifetimes), "Smart Pointers" |
| Blandy, Orendorff, and Tindall, *Programming Rust*, 2nd edition | The chapters about ownership and moves, and about references |
| Rust by Example | "Scoping rules" |
| Jon Gjengset, "Crust of Rust" (YouTube) | The episodes about lifetime annotations and about smart pointers and interior mutability |
| Learning Rust With Entirely Too Many Linked Lists (rust-unofficial.github.io/too-many-lists) | The first lists. Shows why some data structures are hard with ownership. |

## Topics

### 2.0 Memory basics (only if you need them)

Skip this topic if you can explain the stack, the heap, and a dangling pointer.

- **Stack:** each function call gets a frame with its local variables. The frame is removed when the function returns. Allocation is very fast. The size of each value must be known at compile time.
- **Heap:** memory for data whose size or lifetime is not known at compile time. A program asks the allocator for memory and must give it back.
- **Pointer (reference):** a value that holds the address of other data.
- **Manual memory management** (C, C++) can cause these defects:

  | Defect | Description |
  |---|---|
  | Memory leak | Memory is never freed |
  | Use after free (dangling pointer) | The program uses memory after it was freed |
  | Double free | The program frees the same memory two times |
  | Data race | Two threads use the same memory, at least one writes, and nothing synchronizes them |

- **A garbage collector** (C#, Java, Go, Python, JavaScript) finds memory that the program cannot reach and frees it. This removes the first three defects. The costs: pauses or extra CPU time, more memory, and the cleanup time of an object is not known (a file stays open until the collector runs).
- **Rust** removes all four defects at compile time, without a garbage collector. The ownership rules make this possible.

### 2.1 The ownership rules

1. Each value has one **owner**: a variable, a field, or an element of a collection.
2. There is only one owner at a time.
3. When the owner goes out of scope, the value is **dropped**: Rust runs its cleanup and frees its memory.

```rust
fn main() {
    let a = String::from("ledger"); // `a` owns the String.
    let b = a;                      // The String moves to `b`. `a` is not valid now.
    takes_ownership(b);             // The String moves into the function.
    // `b` is not valid here.
}

fn takes_ownership(text: String) {
    println!("{text}");
} // `text` goes out of scope. The String is dropped and its memory is freed.
```

- A **move** copies the small part of the value (for a `String`: the pointer, the length, and the capacity). It does not copy the heap data. After the move, the compiler does not let you use the old variable.
- Moves happen on assignment, on a function call, and on a return.

### 2.2 Move, `Copy`, and `Clone`

- Types that implement `Copy` are copied, not moved: integers, floats, `bool`, `char`, shared references `&T`, and tuples and arrays of `Copy` types. A `Copy` type has no heap data and no cleanup.
- `Clone` makes an explicit deep copy: `let b = a.clone();`. A clone of a `String` allocates new heap memory and copies the bytes.
- Derive `Copy` only for small types that are values by their meaning (a point, an identifier). Do not derive it for types that can grow later: removing `Copy` is a breaking change.
- **A `clone()` is not wrong.** It is wrong when you use it only to make the borrow checker quiet, and the program does not need a second copy. Ask: "Does the program need two independent values, or does one part only need to look at the value?"

### 2.3 Borrowing

A reference borrows a value without taking ownership.

| Reference | Name | Rule |
|---|---|---|
| `&T` | Shared reference | Many can exist at the same time. Nobody can change the value through them. |
| `&mut T` | Mutable (exclusive) reference | Only one can exist, and no shared references at the same time. |

- The short form of the rule: **aliasing XOR mutability**. Either many readers, or one writer.
- This rule removes data races, and also many defects in single-threaded code. Example: a loop over a vector that pushes to the same vector. In Java, this gives a `ConcurrentModificationException` at run time. In C++, it gives undefined behavior. In Rust, it does not compile.
- A borrow lasts until its last use, not until the end of the block ("non-lexical lifetimes").

```rust
let mut items = vec![1, 2, 3];
let first = &items[0];   // Shared borrow starts.
println!("{first}");     // Last use of `first`. The borrow ends here.
items.push(4);           // OK: no shared borrow is active.
```

### 2.4 Slices

- A slice is a reference to a part of a sequence: `&str` for text, `&[T]` for arrays and vectors.
- `&s[0..5]` is the first 5 bytes of a `String` as a `&str`. A range that is not on a UTF-8 character boundary causes a panic.
- **Use slices for parameters that only read.** `fn count_words(text: &str)` accepts a `&String`, a `&str` literal, and a part of a string. `fn count_words(text: &String)` accepts only a `&String`. The same applies to `&[T]` in place of `&Vec<T>`. Clippy warns about `&String` and `&Vec<T>` parameters.

### 2.5 Lifetimes

- A **lifetime** is the part of the program where a reference is valid. The compiler checks that no reference lives longer than the value that it points to.
- Most lifetimes are inferred. You write them when the compiler cannot know the relation between the lifetimes of the inputs and the output:

  ```rust
  fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
      if a.len() >= b.len() { a } else { b }
  }
  ```

  This says: the result is valid as long as both `a` and `b` are valid. **The annotation describes a relation. It does not make a value live longer.**

- **Elision rules:** the compiler adds lifetimes in these cases, so you do not write them:
  1. Each reference parameter gets its own lifetime.
  2. If there is exactly one input lifetime, the output gets it.
  3. If a parameter is `&self` or `&mut self`, the output gets the lifetime of `self`.
- **A struct that holds a reference** needs a lifetime parameter: `struct Parser<'a> { input: &'a str }`. The struct cannot live longer than the text. This is useful for short-lived views (parsers, iterators). For long-lived data, own the data.
- `'static` means "valid for the whole program". String literals are `&'static str`. A bound `T: 'static` means "`T` contains no references that are not `'static`". It does not mean that the value lives forever.

### 2.6 `Drop` and RAII

- **RAII** (resource acquisition is initialization): an object gets a resource when it is made, and gives it back when it is dropped. Rust uses this for memory, files, sockets, and locks.
- A file closes when its `File` value is dropped. A mutex unlocks when its `MutexGuard` is dropped. You do not need `finally` or `using` blocks.
- Implement the `Drop` trait for your own cleanup. Call `drop(value)` to drop a value before the end of its scope, for example to unlock a mutex early.
- Values are dropped in the reverse order of their declaration. The fields of a struct are dropped in the order of their declaration.

### 2.7 Smart pointers and interior mutability

| Type | Ownership | Use it for |
|---|---|---|
| `Box<T>` | One owner, data on the heap | Recursive types (a tree node that contains nodes), large values, trait objects (`Box<dyn Trait>`, phase 3) |
| `Rc<T>` | Shared ownership, one thread. A reference count. | A value that many parts of a single-threaded program use, when no single part is the clear owner |
| `Arc<T>` | Shared ownership, many threads. An atomic reference count. | The same as `Rc`, across threads (phase 6) |
| `Weak<T>` | A reference that does not own | Break reference cycles: a child that points to its parent |
| `Cell<T>` | Interior mutability for `Copy` values | A counter inside a value that is shared with `&` |
| `RefCell<T>` | Interior mutability with borrow checks at run time | Change data behind a shared reference when the compiler cannot prove that it is safe. A wrong borrow causes a panic. |
| `Mutex<T>`, `RwLock<T>` | Interior mutability for many threads | Phase 6 |

- `Rc<RefCell<T>>` is a valid tool, but if your design has it everywhere, you write a garbage-collected object graph by hand. Look for a design with a clear owner first: a struct that owns a `Vec`, and indexes or identifiers in place of references.
- `Rc` cycles leak memory. Use `Weak` for the back references.

### 2.8 Work with the borrow checker

When the borrow checker rejects your code, try these methods in this order:

| Method | Example |
|---|---|
| Make the borrow shorter. Compute a value first, then change the data. | `let n = v.len(); v.push(n);` |
| Borrow fields separately. The compiler knows that two different fields do not overlap. | `let (a, b) = (&mut self.left, &mut self.right);` |
| Split a slice. | `let (left, right) = v.split_at_mut(mid);` |
| Use an index or an identifier in place of a reference. | Store `AccountId` in a transaction, not `&Account`. |
| Take the value out and put it back. | `let old = std::mem::take(&mut self.items);` or `option.take()` |
| Use the entry API of maps. | `map.entry(key).or_default().push(x);` |
| Return owned data from a function in place of references into a temporary value. | `fn names(&self) -> Vec<String>` when the caller needs the names after `self` changes |
| Change who owns the data. | Move the data into the struct that uses it most. |
| Clone, when two independent copies are the correct meaning. | A snapshot of the settings at the start of a job |

- **Self-referential structs** (a struct that holds a reference to its own field) do not work with plain references. Store an index or a range in place of the reference.
- Many borrow errors show a real design question: who owns this data, and who only looks at it? Answer this question first. Phase 5 uses the answer for the module design.

### 2.9 Ownership in function signatures

| Parameter | Meaning | Use it when |
|---|---|---|
| `T` | The function takes ownership | It stores the value or consumes it (`fn add(&mut self, tx: Transaction)`) |
| `&T` | The function reads | Most functions |
| `&mut T` | The function changes the value in place | `fn sort(items: &mut [Item])` |
| `impl Into<String>` | The caller can give a `&str` or a `String` | Constructors that store a string (`Account::new("Main")`) |

- Return owned values (`String`, `Vec<T>`) from functions that make new data. Return references only to data that the parameters (usually `self`) already own.
- `Cow<'a, str>` (clone on write) holds a borrowed or an owned value. Use it when a function usually returns its input unchanged, but sometimes must change it.

### 2.10 Memory layout

- `std::mem::size_of::<T>()` gives the size of a type in bytes.
- A `String` and a `Vec<T>` are three words on the stack: a pointer, a length, and a capacity. The data is on the heap.
- `Option<Box<T>>` and `Option<&T>` have the same size as `Box<T>` and `&T`: `None` uses the null value, which a box or a reference can never have. This is the **niche optimization**.
- An enum is as large as its largest variant, plus a tag (unless a niche holds the tag). A variant with a large field makes all values of the enum large. Box the large field if this is a problem (clippy has the lint `large_enum_variant`).

## Exercises

Make the workspace `labs/phase-2-ownership/`. Exercises marked **core** are for all learners. The other exercises are for learners who are new to ownership (see the learning path in your profile).

### Exercise 2.1: Predict the compiler (core)

For each fragment, predict: does it compile? If not, which rule does it break? Then check with the compiler. Write the results in a table in your engineering notebook.

```rust
let v = vec![1, 2, 3];
let w = v;
println!("{}", v.len());
```

```rust
let x = 5;
let y = x;
println!("{x}");
```

```rust
let mut s = String::from("a");
let r1 = &s;
let r2 = &mut s;
println!("{r1}");
```

```rust
let mut s = String::from("a");
let r1 = &s;
println!("{r1}");
let r2 = &mut s;
r2.push('b');
```

```rust
fn make() -> &String {
    let s = String::from("a");
    &s
}
```

```rust
let mut v = vec![1, 2, 3];
for x in &v {
    if *x == 2 {
        v.push(4);
    }
}
```

```rust
let names = vec![String::from("a"), String::from("b")];
for name in names {
    println!("{name}");
}
println!("{}", names.len());
```

```rust
let mut text = String::from("hello");
let len = calculate(&text);
text.push_str(" world");
fn calculate(s: &str) -> usize { s.len() }
```

### Exercise 2.2: Fix without `clone()` (core)

Fix each program without `clone()`. Use the methods from topic 2.8.

1. A function `fn longest_name(people: Vec<Person>) -> String` that the caller uses in a loop. After the call, the caller cannot use `people`. Change the signature.
2. A `struct Game { players: Vec<Player>, current: usize }` with a method that must change the current player and read all other players at the same time.
3. A method `fn reset(&mut self) -> Vec<Entry>` that must return the old entries and leave the struct with an empty list.

### Exercise 2.3: Text statistics with slices (core)

1. Write `fn longest_word(text: &str) -> Option<&str>`. Do not allocate a new `String`.
2. Write `fn words_longer_than<'a>(text: &'a str, min: usize) -> Vec<&'a str>`.
3. Write `struct Sentence<'a> { words: Vec<&'a str> }` with a constructor that splits a text. Write a test that shows that a `Sentence` cannot live longer than its text (the test does not compile: put it in a comment and explain the error).

### Exercise 2.4: A stack with `Box`

1. Write a singly linked stack: `struct Stack<T> { head: Option<Box<Node<T>>> }`.
2. Implement `push`, `pop`, `peek`, `peek_mut`, `len`, and `is_empty`.
3. Use `Option::take` in `pop`. Explain why you need it.
4. Implement `Drop` without recursion, so that a stack with one million elements does not overflow the call stack. Write a test for it.

### Exercise 2.5: A tree with `Rc` and `Weak`

1. Model an organization: each `Employee` has a name, a list of reports (`Vec<Rc<RefCell<Employee>>>`), and a manager (`Weak<RefCell<Employee>>`).
2. Write functions to add a report, to find the chain of managers from an employee to the top, and to count all people under a manager.
3. Print `Rc::strong_count` and `Rc::weak_count` at each step. Explain the numbers.
4. Change the design: store all employees in a `Vec<Employee>` and use an `EmployeeId(usize)` in place of the pointers. Compare the two designs: which one is simpler to read, to test, and to change?

### Exercise 2.6: RAII guard (core)

1. Write a `Timer` struct that stores a label and the start time (`std::time::Instant`). Its `Drop` implementation prints the label and the elapsed time.
2. Use it at the start of a function: `let _timer = Timer::new("load");`. Explain why `let _ = Timer::new("load");` does not work.
3. Write a test that proves that the drop order is the reverse of the declaration order.

### Exercise 2.7: Undo stack

1. Write a text editor buffer: `struct Editor { text: String, undo: Vec<String> }`.
2. Write `insert`, `delete_last_word`, and `undo`. `undo` restores the text before the last change.
3. Use `std::mem::replace` or `std::mem::take` where it removes a clone.

### Exercise 2.8: Sizes

Print `size_of` for these types and explain each value: `u8`, `char`, `&str`, `String`, `Vec<u64>`, `Box<u64>`, `Option<Box<u64>>`, `Option<u64>`, `Option<u32>`, `Result<u32, u32>`, `[u8; 100]`, and an enum with one large and one small variant.

## Done criteria

- [ ] I can explain the ownership rules, a move, and the difference between `Copy` and `Clone`.
- [ ] I can explain "aliasing XOR mutability", and why it also prevents defects in single-threaded code.
- [ ] I can predict whether a short program compiles, and give the reason.
- [ ] I can fix a borrow error with a change of the design, and say when a clone is the correct fix.
- [ ] I can explain a lifetime annotation in my own words.
- [ ] I can select between `Box`, `Rc`, `Arc`, `RefCell`, and an index-based design, and give the reason.
- [ ] I can select the parameter type of a function from the ownership that it needs.

## Next

[Phase 3: Traits, generics, closures, and iterators](phase-3-traits-generics.md)
