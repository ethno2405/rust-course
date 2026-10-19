# Phase 8: Professional Practices

**Time:** 3–4 weeks
**Prerequisites:** Phase 7
**Code:** `labs/phase-8-practices/`, `labs/ledger/`, and `.github/workflows/`

## Goal

At the end of this phase, you can:

- Write a test strategy: what to test at which level, and with which test doubles.
- Use property-based tests, snapshot tests, mutation tests, and coverage, and know the limits of each.
- Measure performance with benchmarks and a profiler before you optimize.
- Keep the dependencies healthy and safe: licenses, advisories, unused crates, and versions.
- Build a CI pipeline that checks every change.
- Release an application: versions, a changelog, binaries for several operating systems, and a container image.
- Review code and receive a review in a useful way.
- Work with code that has no tests, and change it safely.

## Reading

| Source | Part |
|---|---|
| Jon Gjengset, *Rust for Rustaceans* | The chapters about testing, project structure, and Rust in a wider ecosystem |
| David Drysdale, *Effective Rust* (effective-rust.com) | The items about dependencies, tooling, testing, and documentation |
| Titus Winters, Tom Manshreck, and Hyrum Wright, *Software Engineering at Google* (free at abseil.io/resources/swe-book) | The chapters about code review, testing overview, unit testing, and test doubles |
| Michael Feathers, *Working Effectively with Legacy Code* | The first part: the mechanics of change, and characterization tests |
| The Rust Performance Book (nnethercote.github.io/perf-book) | All. It is short. |
| The Cargo Book | "Profiles", "Features", "Publishing on crates.io", "SemVer Compatibility" |
| Keep a Changelog (keepachangelog.com) and Semantic Versioning (semver.org) | All |
| The documentation of `proptest`, `insta`, `criterion`, `mockall`, `cargo-deny`, `cargo-mutants`, `cargo-llvm-cov` | The front page and the "getting started" section |

## Topics

### 8.1 Test strategy

- **Test pyramid:** many fast unit tests, fewer integration tests, very few end-to-end tests. Each level up is slower and finds defects that the lower levels cannot find.
- A test strategy answers: which risks do we test against, at which level, with which tools, and what runs in CI?
- **Good tests** are fast, deterministic (the same result every time), independent of each other, and test behavior, not implementation. A good test fails only when the behavior is wrong.
- **Test data builders:** a helper that makes a valid test object with default values, so that each test sets only the values that matter: `TransactionBuilder::new().amount("-12.50 EUR").build()`.
- **Mocks:** `mockall` generates mock implementations of traits. Use mocks when the interaction is the behavior (for example "send exactly one email"). Prefer fakes for storage and other stateful ports (phase 5, topic 5.14).
- **Flaky tests** (tests that sometimes fail) destroy the trust in the test suite. Fix or remove them at once. Common causes: real time, the network, shared state, and the order of a `HashMap`.
- `cargo nextest run` runs tests faster than `cargo test` and gives better reports. It does not run doc tests: run `cargo test --doc` too.

### 8.2 More kinds of tests

| Kind | Tool | What it finds | Limit |
|---|---|---|---|
| Property-based tests | `proptest` | Inputs that you did not think of. It generates many random inputs and checks a property that must be true for all of them. When a test fails, it **shrinks** the input to the smallest failing case. | You must find good properties. |
| Snapshot tests | `insta` | Unwanted changes in large outputs (reports, CLI output, JSON). The first run stores the output. Later runs compare with it. | A snapshot is not a specification: review each change with `cargo insta review`. |
| Mutation tests | `cargo-mutants` | Tests that do not check enough. It changes your code (for example `<` to `<=`, or a function body to `Default::default()`), and checks that a test fails. A change that no test finds is a "missed mutant". | Slow. Run it on the core crate, not in every CI run. |
| Fuzzing | `cargo-fuzz` | Crashes and panics on malformed input, for parsers. It needs a nightly toolchain, and works best on Linux and macOS (or WSL2). | Finds crashes, not wrong results |
| Coverage | `cargo-llvm-cov` | Code that no test runs | 100 % coverage does not mean correct tests. Use coverage to find untested code, not as a goal. |

**Good properties for tests:**

- **Round trip:** `parse(format(x)) == x`.
- **Invariant:** the sum of all transactions equals the sum of all account balances.
- **Oracle:** a fast function gives the same result as a simple, slow function.
- **Idempotence:** to import the same file two times gives the same result as one import.

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn money_display_and_parse_round_trip(minor in -1_000_000_000_i64..1_000_000_000) {
        let money = Money::new(minor, Currency::Eur);
        let parsed: Money = money.to_string().parse().unwrap();
        prop_assert_eq!(parsed, money);
    }
}
```

### 8.3 Performance

- **Measure first.** Developers guess the slow part wrong most of the time.
- **Benchmarks:** `criterion` (or `divan`) runs a function many times and gives statistics. It also compares with the last run. Put benchmarks in `benches/`.
- **Profilers** show where the time goes: `samply` (Linux, macOS, Windows), `cargo flamegraph` (Linux, macOS), or the profiler of your operating system. Profile a release build with debug symbols: `[profile.release] debug = true` (only for the profile run).
- Common improvements, after a measurement shows the need:

  | Problem | Improvement |
  |---|---|
  | Many small allocations | Reuse buffers. `Vec::with_capacity`. Return `&str` in place of `String`. |
  | Unnecessary clones | Borrow (phase 2) |
  | The wrong data structure | `HashMap` for lookups, `Vec` for iteration, `BTreeMap` for ranges |
  | Slow I/O | `BufReader` and `BufWriter`. Read the file one time. |
  | CPU-bound work on one core | `rayon` (phase 6) |
  | Debug build | Measure only release builds |

- **Release profile:** `lto = true`, `codegen-units = 1`, and `strip = true` make the executable smaller and sometimes faster. They make the build slower. Measure the effect.

### 8.4 Static analysis and dependency hygiene

- **Lints:** set the workspace lints in `Cargo.toml` (phase 4). Consider some lints of the `clippy::pedantic` group. Enable them one at a time, and keep only the lints that make the code better.
- **`cargo deny check`:** checks licenses (for example, reject GPL in a project that must not use it), security advisories, banned crates, and duplicate versions. Configure it in `deny.toml` (`cargo deny init`).
- **`cargo audit`:** checks `Cargo.lock` against the RustSec advisory database.
- **`cargo machete`:** finds dependencies that the code does not use.
- **`cargo tree --duplicates`:** shows crates that are in the build in more than one version.
- **Minimum supported Rust version (MSRV):** set `rust-version` in `Cargo.toml` for a library. Test it in CI.
- **Updates:** run `cargo update` regularly and run all tests. Dependabot or Renovate can make the update pull requests automatically.
- **Features:** a library enables only the features of its dependencies that it needs. Features must be additive: enabling a feature must not break code that works without it.

### 8.5 Continuous integration

GitHub Actions reads workflows only from `.github/workflows/` in the repository root. On your track branch, this directory is a learner file (see `AGENTS.md`).

```yaml
name: CI

on: [push, pull_request]

env:
  CARGO_TERM_COLOR: always
  SQLX_OFFLINE: "true"

jobs:
  check:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    defaults:
      run:
        working-directory: labs/ledger
    steps:
      - uses: actions/checkout@v5
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy
      - uses: Swatinem/rust-cache@v2
        with:
          workspaces: labs/ledger
      - run: cargo fmt --all --check
      - run: cargo clippy --all-targets -- -D warnings
      - run: cargo test --all-targets
      - run: cargo test --doc
```

- Check the current major versions of the actions on GitHub before you use them.
- Add jobs for `cargo deny check`, coverage (upload the report as an artifact), and the MSRV.
- Make the pipeline fast: cache the build, and run the slow checks (mutation tests, fuzzing) on a schedule, not on every push.
- **A red pipeline is the first priority of the team.** Do not merge when CI fails.

### 8.6 Versions and releases

- **Semantic versioning:** `MAJOR.MINOR.PATCH`. For `0.y.z` versions, Cargo treats a change of `y` as breaking.
- **Breaking changes in Rust** include more than you expect: a new public field in a struct that users build with a struct literal, a new variant in an enum without `#[non_exhaustive]`, a new method without a default in a public trait, and a removed trait implementation (for example a type that is no longer `Send`). `cargo semver-checks` finds many of them.
- **Changelog:** keep a `CHANGELOG.md` in the "Keep a Changelog" format. Write for users: "Added", "Changed", "Fixed", "Removed".
- **Release steps:** update the version and the changelog, commit, tag (`v1.2.0`), and let CI build the release artifacts.
- **Binaries:** build release executables for Linux, Windows, and macOS in CI. A tool like `dist` (formerly `cargo-dist`) can generate the release workflow.
- **Container image:** a multi-stage `Dockerfile`: a `rust` image builds the executable, and a small runtime image (`debian:bookworm-slim` or a distroless image) runs it as a user that is not root.
- **Publish a library:** `cargo publish --dry-run` checks the package. The metadata needs a description, a license, and a repository URL.

### 8.7 Code review

- **Why:** to find defects, to share knowledge in the team, and to keep the design consistent. Most of the value is in the knowledge transfer.
- **As the author:** make small pull requests (fewer than 400 changed lines). Explain why the change is necessary in the description. Review your own diff first. Make sure that CI passes.
- **As the reviewer:** use the checklist in [../topics/code-review-checklist.md](../topics/code-review-checklist.md). Write about the code, not the person. Explain why. Say which comments block the merge and which are only suggestions ("nit"). Name what is good.
- **Automate what a tool can check.** Do not discuss format (rustfmt) or the things that clippy finds. Discuss correctness, design, tests, and names.

### 8.8 Work with existing code

- **Legacy code** (Michael Feathers): code without tests. You cannot change it safely.
- **Characterization tests:** tests that record the current behavior before you change it (phase 5, topic 5.13).
- **Seams:** places where you can change the behavior without a change to the code, for example a trait parameter. Add a seam to put a part of the code under test.
- **Strangler fig:** replace a large old system step by step. New code takes over one feature at a time, and the old code becomes smaller until you can remove it.
- **Deprecation:** mark an old API with `#[deprecated(since = "1.4.0", note = "use parse_statement instead")]`. Remove it in the next major version.

### 8.9 Documentation of a project

- **README:** what the project does, how to install it, how to use it, and how to develop it (build, test, run).
- **Architecture overview:** a short document with the parts, their responsibilities, and the dependency direction. Link the ADRs.
- **Runbook** (for a service): how to start, stop, configure, and check it, and what to do for each common failure.
- Keep the documentation next to the code, in the same repository. Review documentation changes in the same pull request as the code.

## Exercises

Make the workspace `labs/phase-8-practices/` for the exercises that do not use the ledger.

### Exercise 8.1: Test strategy

1. Write `labs/ledger/docs/testing.md`: the risks of the ledger, and for each risk the level and the kind of test that covers it.
2. Run `cargo llvm-cov --html` on the ledger. Find the untested code. Decide for each part whether it needs a test.
3. Write test data builders for the main domain types. Use them in the existing tests.

### Exercise 8.2: Property-based tests

1. Write property tests for `Money`: the round trip of `Display` and `FromStr`, and the rules of addition (commutative, an error for different currencies).
2. Write a property test for the CSV import: generate valid transactions, write them as CSV, import them, and compare.
3. Write the invariant test: for random transactions on random accounts, the sum of all transactions equals the sum of all balances.
4. If a property finds a defect, keep the shrunk input as a normal unit test.

### Exercise 8.3: Snapshot tests

1. Add `insta` snapshot tests for the monthly report (text) and for two JSON responses of the HTTP API.
2. Change the report format on purpose. Review the change with `cargo insta review`.

### Exercise 8.4: Mutation tests

1. Run `cargo mutants` on the ledger core crate.
2. For each missed mutant, decide: is a test missing, or is the mutant equivalent (it does not change the behavior)?
3. Add the missing tests. Run `cargo mutants` again. Record the numbers before and after.

### Exercise 8.5: Performance

1. Generate a test data set with one million transactions.
2. Write a `criterion` benchmark for the monthly report and for the CSV import.
3. Profile both with a profiler. Find the slowest functions.
4. Improve the slowest part. Record each change and its effect on the benchmark in `labs/ledger/docs/performance.md`. Keep only the changes that make a measurable difference.

### Exercise 8.6: CI pipeline

1. Add `.github/workflows/ci.yml` for the ledger with format, clippy, tests, and doc tests on Linux, Windows, and macOS.
2. Add `cargo deny check` with a `deny.toml` that permits only the licenses that you select.
3. Add a coverage job that uploads the HTML report as an artifact.
4. Push the track branch to your private repository, and make the pipeline pass. Make it fail on purpose with a clippy warning, then fix it.

### Exercise 8.7: Release

1. Set the version of the ledger to `1.0.0`. Write `CHANGELOG.md`.
2. Add a release workflow: on a tag `v*`, build release executables for the three operating systems and attach them to a GitHub release.
3. Write a multi-stage `Dockerfile` for the HTTP service. The final image runs as a user that is not root. Measure the image size.
4. Make a breaking change in the core crate on a branch. Run `cargo semver-checks` against the last release.

### Exercise 8.8: Code review

1. Find a small open pull request in a Rust project on GitHub (search for the label "good first issue" or a small change). Review it with the checklist. Do not post the review if you are not sure. Compare your review with the reviews of the maintainers.
2. Open a pull request on your private repository with a ledger feature. Ask the mentor for a review as a team member. Answer each comment, and fix or discuss it.

## Project: Ledger, stage 5

See [../projects/ledger.md](../projects/ledger.md), stage 5.

## Done criteria

- [ ] I can write a test strategy, and explain the pyramid and the limits of each test kind.
- [ ] I can write property tests and snapshot tests, and use mutation tests and coverage to find weak tests.
- [ ] I measured before I optimized, and I can show the effect of each change.
- [ ] My dependencies are checked for licenses and advisories in CI.
- [ ] My CI pipeline checks format, lints, and tests on three operating systems.
- [ ] I can release a version with a changelog, binaries, and a container image.
- [ ] I can review code with a checklist and give useful comments.
- [ ] Ledger stage 5 is complete.

## Next

[Phase 9: Beyond](phase-9-beyond.md)
