# Code Review Checklist

**Phase:** 5–8. The mentor also uses it for every code review.
**Related:** [phase 5](../phases/phase-5-software-design.md), [phase 8, topic 8.7](../phases/phase-8-professional-practices.md), [mentoring](../mentor/mentoring.md)

## Summary

This checklist helps you review Rust code in a fixed order: from the problems that cost the most to the problems that cost the least. Use it for your own code before you ask for a review, and for the code of other people.

## Explanation

### Before the review

- [ ] The change has a clear purpose. The description says why it is necessary.
- [ ] The change is small enough to review (fewer than 400 changed lines). If it is not, ask the author to split it.
- [ ] `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass. Do not review what a tool can check.

### 1. Correctness

- [ ] The code does what the description says, also for the edge cases: empty input, one item, the maximum value, negative values, Unicode text.
- [ ] Each `Result` and `Option` is handled. No `unwrap()` on a value that can be `None` or `Err` in normal operation.
- [ ] Integer arithmetic cannot overflow, or the overflow behavior is explicit (`checked_`, `saturating_`).
- [ ] Money and other exact values do not use floating-point types. Rounding rules are explicit.
- [ ] `as` conversions cannot lose data, or the loss is intended and commented.
- [ ] Indexes and slices cannot be out of range, and string slices are on character boundaries.
- [ ] **Concurrency:** no lock is held across an `.await` or during I/O. Locks are taken in a fixed order. Channels are bounded where the producer can be faster than the consumer.
- [ ] **Async:** no blocking calls in async functions. Network calls have timeouts.

### 2. Tests

- [ ] New behavior has tests. A fixed defect has a test that fails without the fix.
- [ ] Each test can fail: it checks the result, not only that the code runs.
- [ ] The test names describe the behavior.
- [ ] The tests do not depend on the real time, the network, random values without a seed, or the order of other tests.
- [ ] The edge cases from section 1 are tested.

### 3. Design

- [ ] Each module and type has one responsibility. The names say what it is.
- [ ] The dependencies point in the correct direction: the domain does not depend on adapters, frameworks, or I/O.
- [ ] The types make invalid states impossible: enums in place of `bool` flags and strings, newtypes in place of primitives.
- [ ] The public API is as small as possible. Fields are private when the type has rules.
- [ ] No duplicated knowledge. But no abstraction with only one use and no test that needs it (YAGNI).
- [ ] Trait implementations keep the contract of the trait (Liskov).
- [ ] Traits are small and focused (interface segregation).
- [ ] Errors are meaningful types in libraries, and have context in applications. An error is not both logged and returned.

### 4. Idioms and ownership

- [ ] Parameters borrow when they only read: `&str`, `&[T]`, `&T`.
- [ ] No `clone()` that only makes the borrow checker quiet.
- [ ] Iterators where they make the code clearer. Loops where the logic has many branches.
- [ ] Names follow the conventions: `as_`, `to_`, `into_`, no `get_` prefix, `is_` for `bool`.
- [ ] The common traits are implemented or derived for public types (`Debug`, `Clone`, `PartialEq`, ...).
- [ ] No `Deref` to imitate inheritance. No `Rc<RefCell<T>>` where a clear owner is possible.

### 5. Security and data

- [ ] No secrets, tokens, or real personal data in the code, the tests, or the logs.
- [ ] Input from outside (files, HTTP, the command line) is validated at the boundary.
- [ ] SQL uses bind parameters.
- [ ] New dependencies are well known, maintained, and have a compatible license.
- [ ] `unsafe` (phase 9 only) has a `// SAFETY:` comment that explains why it is sound.

### 6. Readability and documentation

- [ ] A reader understands the code without the author. Comments explain why, not what.
- [ ] Public items have doc comments, with `# Errors` and `# Panics` where they apply.
- [ ] The README, the ADRs, and the changelog are updated if the change affects them.

### How to write the comments

- Write about the code, not about the person: "This function can panic on empty input", not "You forgot empty input".
- Explain why: the problem that the code causes, not only the rule.
- Mark each comment as **blocking** (must change before the merge) or **suggestion** (the author decides).
- Name one or two things that are done well.
- Ask questions when you are not sure: "Can `items` be empty here?"

## Example

A review comment:

> **Blocking.** `src/import.rs:42`: `amount.parse::<f64>()` loses precision for some amounts (for example `0.1 + 0.2` is not `0.3`). Over many transactions, the balance can be wrong by some cents. Parse the amount into minor units as an integer, as `Money::from_str` does.

## Common mistakes

| Mistake | Result | Fix |
|---|---|---|
| The reviewer discusses the format and the lint findings | Time is lost. The important problems are not found. | Let `rustfmt` and `clippy` do this in CI. |
| The review starts with names and style | The reviewer is tired before the correctness and design checks | Use the order of this checklist. |
| The change is too large to review | The reviewer approves without real checks | Split the change into small pull requests. |
| All comments look equally important | The author does not know what must change | Mark comments as blocking or suggestion. |

## Sources

- Google, "Code Review Developer Guide" (google.github.io/eng-practices)
- Titus Winters, Tom Manshreck, and Hyrum Wright, *Software Engineering at Google*, the chapter about code review
- Rust API Guidelines, the checklist (rust-lang.github.io/api-guidelines/checklist.html)
