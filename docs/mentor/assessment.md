# Assessment

This document tells the AI mentor how to assess the knowledge of a learner. The result is the learner profile, `learner/profile.md`. The mentor uses the profile in every session.

## When to assess

| Situation | Action |
|---|---|
| `learner/profile.md` does not exist | Do a full assessment: mode selection, part A, part B, project selection, result |
| `learner/profile.md` does not exist, but `learner/progress.md` exists | Rebuild the profile. Recommend the quick mode. Keep the progress file and the current position. |
| The learner asks to re-assess one area ("Re-assess my design knowledge") | Do part B for that area only. Update the level after the learner agrees. |
| The learner completes a phase | Do the phase review (last section of this document) |

## Rules for the assessment

- Tell the learner that the assessment is not a test. Its only purpose is to select the correct start point and the correct explanation level.
- Ask **one question at a time**. Wait for the answer.
- Tell the learner that "I do not know" is a good answer. A guess gives a wrong level.
- Ask the learner not to look up answers and not to run the code.
- Do not explain the correct answer during part B, unless the learner asks. Give all corrections at the end, in the result summary.
- If a question has more than one part and the learner does not answer all parts, tell the learner which part is missing. Ask for that part before you record the result (see `docs/mentor/mentoring.md`, "Questions with more than one part").
- Keep a short record of each question and the result. You write it into the profile as evidence.

## Step 1: Mode selection

Show the three modes to the learner and ask them to select one:

| Mode | Time | Content | Accuracy |
|---|---|---|---|
| **Quick** | ~5 minutes | Part A only (self-report) | Low. The mentor trusts the answers and corrects the levels later from the learner's work. |
| **Standard** | ~15–20 minutes | Part A, then 2–4 adaptive questions for each area in part B | Good for most learners |
| **Full** | ~45 minutes | Part A, then 5–8 questions for each area in part B, with the practical items | High. Use it if the learner wants an exact start point. |

If the learner does not select a mode, use **standard**.

## Step 2: Part A, self-report (all modes)

Ask these questions. You can ask related questions together in one message.

1. Which programming languages do you know? For how many years? Which one do you know best?
2. How much Rust do you know? (none / I read some of "the Book" / I wrote small programs / I use it at work)
3. Do you know how memory works in a program: the stack, the heap, pointers or references, garbage collection? (no / a little / yes)
4. Did you write programs with threads, locks, or async code? (no / a little / yes, often)
5. Do you know design principles and patterns: SOLID, design patterns, layered architecture, unit tests with test doubles? (no / I know the names / I use some of them / I use them at work and can explain the trade-offs)
6. Which tools do you use: git, a terminal, a debugger with breakpoints, a package manager, a CI pipeline?
7. Which operating system and which editor do you use?
8. How many hours per week can you work on the course?
9. What is your goal? (for example: backend services, command-line tools, a job that uses Rust, better design skills in general, a specific project)
10. How do you want the mentor to help when you are blocked? (hints first, and a solution only when I ask / a full solution with an explanation)
11. How detailed must the explanations be? (short / normal / very detailed)

In **quick** mode, set the levels from the self-report with the rubric below, and mark them "self-reported". Then go to step 4.

## Step 3: Part B, knowledge check (standard and full modes)

### Adaptive rule

For each area:

1. Use the self-report to select the first question. Self-report "none" or "no" → start at level 1. Self-report "a little" or "I know the names" → start at level 2. Self-report "I use it at work" or "yes, often" → start at level 3.
2. A correct answer → ask the next question at a higher level (or another question at level 3).
3. A wrong answer or "I do not know" → ask the next question at a lower level.
4. Stop when the level is clear. Standard mode: 2–4 questions for each area. Full mode: 5–8 questions for each area, including the practical items.
5. A partly correct answer counts as half. Ask one more question at the same level to decide.

You can change the wording of a question. You can make new questions at the same level. The expected answers below are the minimum for "correct".

If the learner has Rust level 0 in the self-report, ask only the level 1 questions of area R. Do not make a learner with no Rust read Rust code in the other areas: use pseudocode or the language that the learner knows best.

### Area R: Rust language

| Level | Question | Expected answer |
|---|---|---|
| 1 | What is the difference between `let x = 5;` and `let mut x = 5;`? | Without `mut`, the variable cannot change after the first assignment. Variables are immutable by default. |
| 1 | Rust has no `null`. How does a function say "maybe there is no value"? | It returns `Option<T>`: `Some(value)` or `None`. |
| 1 | What does it mean that a `match` must be exhaustive? | The `match` must handle every possible value. The compiler gives an error if a case is missing. |
| 2 | Why does this not compile? `let s = String::from("a"); let t = s; println!("{s}");` | The assignment moves the `String` to `t`. `s` is not valid after the move. |
| 2 | What is the difference between `String` and `&str`? Which one do you use for a function parameter that only reads the text? | `String` owns its heap buffer and can grow. `&str` is a borrowed view of UTF-8 bytes. Use `&str` for a parameter that only reads. |
| 2 | What does the `?` operator do? | On `Err` (or `None`), it returns early from the function with the error (converted with `From`). On `Ok`, it gives the value. |
| 2 | What is the difference between `fn f(x: impl Display)` and `fn f(x: &dyn Display)`? | `impl Trait` is static dispatch: the compiler makes one copy of the function for each type. `dyn Trait` is dynamic dispatch through a vtable at run time. |
| 3 | Explain the lifetime in `fn longest<'a>(a: &'a str, b: &'a str) -> &'a str`. | The returned reference is valid only while both arguments are valid. The annotation describes a relation. It does not change how long a value lives. |
| 3 | When does `RefCell` check the borrow rules? What happens if the code breaks them? | At run time. A second mutable borrow (or a mutable borrow while a shared borrow exists) causes a panic. |
| 3 | You cannot implement `Display` for `Vec<MyType>`. Why? What do you do? | The orphan rule: the trait or the type must be local to the crate. Wrap the vector in a local newtype. |
| 3 | What is the difference between `Fn`, `FnMut`, and `FnOnce`? | `Fn` borrows its captures, `FnMut` mutably borrows them, `FnOnce` can consume them and can be called only one time. |

### Area S: memory and concurrency

| Level | Question | Expected answer |
|---|---|---|
| 1 | What is the difference between the stack and the heap? | The stack holds local variables and function calls. It is fast and has a fixed size per value. The heap holds data whose size or lifetime is not known at compile time. |
| 1 | What is a thread? | An independent sequence of execution in a process. Threads share the memory of the process. |
| 2 | What is a dangling pointer (or reference)? What is a memory leak? | A dangling pointer points to memory that was freed. A leak is memory that the program does not free and cannot use. |
| 2 | What is a race condition? | The result depends on the timing of two threads that use shared data. At least one of them writes, and there is no synchronization. |
| 2 | What is the difference between concurrency and parallelism? | Concurrency: many tasks make progress in the same time period, possibly on one CPU core. Parallelism: tasks run at the same time on many cores. |
| 3 | What is a deadlock? Give one method to prevent it. | Two or more threads wait for locks that the others hold. Always take locks in the same order, use one lock, or use timeouts. |
| 3 | What problem does a garbage collector solve? What does it cost? | It frees memory that the program cannot reach any more. Cost: pauses or extra CPU time, more memory, and no deterministic cleanup of resources. |
| 3 | An async function calls a function that blocks for 2 seconds (for example a synchronous file read or a sleep). What is the problem? | The function blocks the thread of the async runtime. Other tasks on that thread cannot run. Use an async version, or move the work to a blocking thread pool. |
| 3 | What do `Send` and `Sync` mean in Rust? (Ask only if R ≥ 2.) | `Send`: the value can move to another thread. `Sync`: a shared reference to the value can go to another thread. |

### Area D: design and practices

| Level | Question | Expected answer |
|---|---|---|
| 1 | Why do developers write automated tests? | To find defects early, to change code safely (regression), and to document the behavior. |
| 1 | What is the difference between an interface and an implementation? | The interface says what an object or module does (its contract). The implementation says how it does it. |
| 2 | What is the difference between a unit test and an integration test? | A unit test checks one small unit alone, often with test doubles. An integration test checks that many parts work together. |
| 2 | Explain the dependency inversion principle. | High-level code does not depend on low-level details. Both depend on an abstraction (an interface or a trait) that the high-level code owns. |
| 2 | Give an example of a "code smell". | Any valid example with a reason, for example: a long function, duplicate code, a long parameter list, a class or module that does too many things, primitive values for domain concepts. |
| 2 | What does "composition over inheritance" mean? | Build behavior from small objects that you combine, in place of deep class hierarchies. |
| 3 | Explain the Liskov substitution principle. Give an example of a violation. | A subtype (or an implementation of an interface) must keep the contract of the type. Example: a `Square` that extends `Rectangle` and changes both sides on `set_width`; a read-only collection that throws on `add`. |
| 3 | When does "do not repeat yourself" (DRY) make a design worse? | When two pieces of code look the same but represent different knowledge, or change for different reasons. A shared abstraction then couples them. "Duplication is far cheaper than the wrong abstraction." |
| 3 | What is a port and an adapter in a hexagonal architecture? | A port is an interface that the application core defines (for example a repository). An adapter implements the port for a specific technology (a database, HTTP, a file). |
| 3 | How do you test code that uses the current time? | Inject a clock (an interface or a parameter). Use a fixed or controllable clock in the test. |

### Area T: tools

| Level | Question | Expected answer |
|---|---|---|
| 1 | What does `git commit` do? | It records the staged changes as a new version in the local repository. |
| 1 | What does a compiler do? How is it different from an interpreter? | A compiler translates the whole source code to machine code (or another form) before the program runs. An interpreter runs the source code (or bytecode) directly. |
| 2 | What is a package manager? What is a lock file for? | A package manager downloads and builds dependencies. A lock file records the exact versions, so that every build uses the same versions. |
| 2 | How do you find the value of a variable at a specific line with a debugger? | Set a breakpoint on the line, run, and inspect the variable when the program stops. |
| 2 | What does a version number like `2.4.1` mean in semantic versioning? | Major, minor, patch. A major change breaks compatibility. A minor change adds features compatibly. A patch fixes defects. |
| 3 | What does a typical CI pipeline do? | On each push or pull request: build, format check, lint, run tests, and sometimes security checks and release steps. |
| 3 | What is the difference between a debug build and a release build? | A debug build has no or few optimizations and includes debug checks (in Rust: integer overflow checks). It compiles fast. A release build is optimized and runs fast. |
| 3 | What is the difference between `git merge` and `git rebase`? | Merge joins two histories with a merge commit. Rebase replays commits on a new base and makes a linear history. Rebase changes commit identifiers. |

### Practical items (full mode only)

Ask 1–2 items for each area, at the level of the learner.

**R, code reading.** Ask whether the code compiles. If not, why?

```rust
fn first_word(s: &String) -> &str {
    s.split(' ').next().unwrap_or("")
}

fn main() {
    let word;
    {
        let text = String::from("hello world");
        word = first_word(&text);
    }
    println!("{word}");
}
```

Expected: it does not compile. `word` borrows from `text`, and `text` is dropped at the end of the inner block. Bonus: the parameter type is better as `&str`.

```rust
let mut scores = vec![10, 20, 30];
for s in &scores {
    if *s > 15 {
        scores.push(*s * 2);
    }
}
```

Expected: it does not compile. The loop borrows `scores`, and `push` needs a mutable borrow at the same time. In other languages this is a "modified during iteration" failure at run time.

**S, concurrency.** Explain what can go wrong (in pseudocode or the learner's language): two threads run `counter = counter + 1` 1000 times each on a shared integer without a lock. What values can `counter` have at the end?

Expected: a value from 2 to 2000. The read-modify-write is not atomic, so increments are lost.

**D, design review.** Show this function and ask for the design problems:

```rust
fn process(path: &str, send_email: bool, format: &str) -> f64 {
    // 80 lines: read the file, parse CSV lines, compute the total
    // of the orders, write a report in "html" or "text" format,
    // and send an email when send_email is true.
    todo!()
}
```

Expected (2 or more): it does too many things (single responsibility); a `bool` parameter and a string `format` parameter (use enums); hard to test because it reads files and sends email (inject the dependencies); `f64` for money; the return value mixes calculation with I/O.

**T, tools.** Explain the steps from a new idea to code on the `main` branch in a team.

Expected: a branch, small commits, tests and lints run locally, push, a pull request, CI runs, a code review, a merge.

## Level rubric

Give each area (R, S, D, T) a level from 0 to 3:

| Level | Name | Description |
|---|---|---|
| 0 | None | Cannot answer level 1 questions. |
| 1 | Basic | Answers most level 1 questions. Fails most level 2 questions. |
| 2 | Working | Answers most level 2 questions. Fails most level 3 questions. |
| 3 | Strong | Answers most level 3 questions. |

In quick mode, use the self-report: "none" / "no" → 0, "I read some of the Book" / "a little" / "I know the names" → 1, "I wrote small programs" / "I use some of them" → 2, "I use it at work" / "yes, often" / "I can explain the trade-offs" → 3. Mark these levels "self-reported".

## Learning path

Use the levels to make the learning path. Write the decisions in the profile.

| Condition | Decision |
|---|---|
| R = 0 | Do all of phase 1, with Rustlings. Plan the phase 1 time from the background table in `docs/README.md`. |
| R = 1 | Do all topics and exercises of phase 1. Plan 2–3 weeks. |
| R = 2 | Read phase 1 quickly. Do exercises 1.5, 1.6, and 1.8 and project stage 0. In phases 2 and 3, do all exercises. |
| R = 3 | Do project stage 0 only in phase 1. In phases 2 and 3, do the exercises marked "core". Focus on phases 4–8. |
| S = 0 or 1 | Do topic 2.0 before phase 2. Explain the stack and the heap with diagrams. In phase 6, do all exercises. |
| S = 3 | In phase 6, focus on the Rust-specific details: `Send`, `Sync`, and the async runtime. |
| D = 0 or 1 | Before phase 5, read the parts of *The Pragmatic Programmer* that phase 5 names. Plan 5 weeks for phase 5. Explain each principle with a small example before you name it. |
| D = 3 | In phase 5, focus on the mapping of the principles to Rust (traits, enums, crates) and on the trade-offs. Do exercises 5.2, 5.5, 5.7, and 5.9. |
| T = 0 or 1 | Spend extra time on phase 0, topics 0.8 and 0.9 (git). Teach the terminal commands that the learner needs. |
| A managed language is the best known language (C#, Java, Go, TypeScript) | Compare with interfaces, classes, exceptions, and the garbage collector. Expect problems with ownership in phase 2. |
| A dynamic language is the best known language (Python, JavaScript, Ruby) | Explain static types and compilation first (topic 1.0). Expect problems with types, ownership, and traits. |
| C or C++ is the best known language | Compare with pointers, RAII, templates, and undefined behavior. Expect fast progress in phase 2. |
| Fewer than 4 hours per week | Tell the learner to double the phase times in `docs/README.md`. |

## Step 4: Project selection

Do the project selection in `docs/mentor/project-selection.md`. The learner can keep the reference project (the ledger) or select a different domain. If the learner does not want to decide now, record "Ledger (default)" in the profile.

## Step 5: Result

1. Write `learner/profile.md` from `docs/templates/profile-template.md`. Fill in all sections. Put the evidence for each level (the questions and the results).
2. Show the learner a summary: the level of each area, the learning path, the project, and the corrections for the questions that they answered wrong.
3. Ask the learner whether the summary is correct. Change the profile if the learner disagrees with good reasons.
4. If `learner/progress.md` does not exist, create it from `docs/templates/progress-template.md`. Set the start date to today.
5. If `learner/progress.md` already existed before the assessment, do not change the current position. Tell the learner their next step from the progress file. Otherwise, tell the learner the first step of their learning path: phase 0, topic 0.1.

## Phase review

When the learner says that they completed a phase:

1. Ask 3–5 questions from the "Done criteria" list of the phase. Use questions that need an explanation, not a yes or no.
2. Ask for evidence of the practical criteria: code and test results in `labs/`, the output of `cargo clippy` and `cargo test`, or a CI run.
3. Read the code of the project stage of the phase. Do a short design review with `docs/topics/code-review-checklist.md`.
4. If the learner meets all criteria, mark the phase "Done" in `learner/progress.md`.
5. If some criteria are not met, tell the learner which ones, and suggest the exercises to repeat.
6. If the results show that a level in the profile changed, propose the new level to the learner. Change it after the learner agrees. Add the review to the "Assessment history" in the profile.
7. Give the learner the phase summary: their performance in the phase, their gaps, what to improve, and a short description of the next phase. See `docs/mentor/mentoring.md`, "Overviews and summaries". Write a one-line result in the "Notes" column of the phase in `learner/progress.md`.
