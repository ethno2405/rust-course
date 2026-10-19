# Project: Personal Finance Ledger

The ledger is the main project of the course. It starts as a few types in memory in phase 1. It ends as a tested, documented application with a command-line interface, an HTTP API, a database, and a release pipeline in phase 8. Each stage adds features to the code from the previous stage. Do not start again from zero.

If your profile names a different project domain, the mentor translates this document (see [../mentor/project-selection.md](../mentor/project-selection.md)).

**Use only generated sample data.** Do not import your real bank statements into the labs, and never commit them. The `.gitignore` file ignores database files and `data/private/`.

## Final system

```
Users           Driving adapters           Core                          Driven adapters           Outside
-----           ----------------           ----                          ---------------           -------
Terminal  ----> ledger-cli  ---+
                               +---------> ledger-core                <-- ledger-store-json     --> ledger.json
HTTP client --> ledger-api  ---+           domain types, rules,       <-- ledger-store-sqlite   --> ledger.db
                                           use cases, ports (traits)  <-- ledger-import (CSV)   <-- bank statements
                                                                      <-- ledger-rates-http     <-- exchange rate API

All crate dependencies point to ledger-core. ledger-cli and ledger-api are the composition roots.
```

## Domain

| Concept | Description |
|---|---|
| `Currency` | The currencies that the ledger supports. Start with EUR, USD, GBP, and CHF. All four have 2 minor units (cents). |
| `Money` | An amount in minor units (`i64`) and a currency. `12.50 EUR` is `1250` and `Currency::Eur`. Two amounts with different currencies cannot be added. Never use `f32` or `f64`. |
| `Account` | An identifier, a name (unique), and a currency |
| `Transaction` | An identifier, an account, a date, a description, an amount (negative for a payment, positive for income), and an optional category |
| `Category` | A name, for example "Food" or "Rent" |
| `Rule` | A categorization rule: if a transaction matches a condition, it gets a category (stage 2) |
| `Budget` | A monthly limit for a category (stage 2) |

The simplification "all currencies have 2 minor units" is a decision. Write it in the documentation of `Currency`. A currency like JPY (0 minor units) would need a change. This is a good example of a decision that an ADR records (phase 5).

## Stage 0: Domain model (phase 1)

**Code:** `labs/ledger/` as one package.

**Features**

- `Currency` (an enum) and `Money` with `Display` (`12.50 EUR`, `-0.05 EUR`), and an addition that returns an error for different currencies.
- `Account`, `Transaction`, and a `Ledger` struct that owns the accounts and the transactions.
- `Ledger` methods: add an account (reject a duplicate name), add a transaction (reject an unknown account and a currency that is not the account currency), the balance of an account, and the transactions of a month.
- Use the date crate that you selected in phase 0, exercise 0.6.
- `main` makes a ledger with sample data and prints the balances.

**Acceptance criteria**

- [ ] No floating-point type is used for money.
- [ ] `Display` of `Money` is correct for negative values between 0 and -1 (for example `-0.05 EUR`).
- [ ] Each rule (duplicate name, unknown account, wrong currency) has a test.
- [ ] `cargo clippy --all-targets -- -D warnings` passes.

## Stage 1: Command-line ledger (phase 4)

**Code:** `labs/ledger/` as one package with a library crate and a thin binary crate.

**Features**

- Commands, with `clap`:

  | Command | Result |
  |---|---|
  | `ledger account add <name> --currency <code>` | Add an account |
  | `ledger account list` | List the accounts |
  | `ledger add --account <name> --date <YYYY-MM-DD> --amount <amount> --description <text> [--category <name>]` | Add a transaction |
  | `ledger list [--account <name>] [--from <date>] [--to <date>] [--category <name>]` | List transactions |
  | `ledger balance` | Show the balance of each account |
  | `ledger import <file> --account <name>` | Import a CSV statement |
  | `ledger report --month <YYYY-MM>` | Show the spending of each category in a month |

- Storage: one JSON file. The path comes from `--file`, or the environment variable `LEDGER_FILE`, or the default `ledger.json`. The file has a `version` field. The program saves it safely (write a temporary file, then rename).
- The **generic CSV format**: a header line `date,description,amount`, dates as `YYYY-MM-DD`, amounts with a `.` as the decimal separator (`-12.50`).
- The import is all-or-nothing: if one line is wrong, the program imports nothing and reports all wrong lines with their line numbers.
- Duplicate detection: a transaction with the same account, date, amount, and description as an existing transaction is skipped. The import reports the number of skipped transactions. An import of the same file two times adds nothing the second time.
- Error types with `thiserror` in the library, `anyhow` in the binary.
- Newtypes that cannot hold invalid values: `AccountName`, `Money` from a string, the month `YYYY-MM`.
- Documentation: `#![warn(missing_docs)]`, and doc tests for `Money`.
- Sample data in `labs/ledger/sample-data/`: two CSV files with 20–50 fake transactions.

**Acceptance criteria**

- [ ] No user input makes the program panic. Wrong input gives a clear message on stderr and an exit code that is not 0.
- [ ] Import errors show the line number and the wrong value.
- [ ] An import of the same file two times does not add duplicates.
- [ ] `main.rs` has fewer than 50 lines.
- [ ] Integration tests use the library API. CLI tests check the output and the exit code of each command.
- [ ] `cargo doc` shows documentation for every public item.

## Stage 2: Hexagonal architecture (phase 5)

**Code:** `labs/ledger/` as a Cargo workspace.

**Features**

- Change the package into a workspace with these crates:

  | Crate | Responsibility | Depends on |
  |---|---|---|
  | `ledger-core` | Domain types, rules, use cases, ports | No I/O crates. `thiserror` and the date crate are permitted. |
  | `ledger-import` | Statement formats (CSV parsing) | `ledger-core`, `csv` |
  | `ledger-store-json` | The storage port for a JSON file | `ledger-core`, `serde`, `serde_json` |
  | `ledger-cli` | The commands. The composition root. | All crates above, `clap`, `anyhow` |

- **A second import format, "bank-b":** semicolon-separated, the header `booking date;text;debit;credit`, dates as `DD.MM.YYYY`, a comma as the decimal separator, a `.` as the thousands separator (`1.234,56`), and the amount in the `debit` column or in the `credit` column. Add it without a change to `ledger-core`.
- `ledger import <file> --account <name> [--format auto|generic|bank-b]`. With `auto`, each format checks the header line and says whether it can read the file.
- **Categorization rules** in a rules file: "the description contains X" and "the amount is in a range". The first matching rule gives the category. `ledger categorize` applies the rules to all transactions without a category. Decide whether rules are an enum or a trait, and write an ADR.
- **Budgets:** `ledger budget set <category> <amount>` and `ledger budget status [--month <YYYY-MM>]`. The status shows the limit, the spent amount, and the rest, and marks the categories over the limit. The default month is the current month: inject the date (phase 5, topic 5.14).
- **Reports** are sorted in a fixed order, so that the output is the same in every run.
- **Use cases** in the core: `add_transaction`, `import_transactions`, `categorize`, `monthly_report`, `budget_status`. The CLI calls only the use cases.
- **Test doubles:** an in-memory store for the tests of the core. Contract tests for the storage port run against the in-memory store and the JSON store.
- ADRs in `labs/ledger/docs/adr/`: at least the workspace structure, the storage port, and the rules design.

**Acceptance criteria**

- [ ] `ledger-core` has no dependency on `serde_json`, `csv`, `clap`, or `anyhow`, and does not use `std::fs`. `cargo tree` proves it.
- [ ] The commit that adds the "bank-b" format does not change `ledger-core`.
- [ ] The use cases have tests that use only fakes. No core test touches the file system.
- [ ] The contract tests pass for all store adapters.
- [ ] No test depends on the current date.

## Stage 3: Concurrent import and exchange rates (phase 6)

**Code:** `labs/ledger/`. New crates: `ledger-rates-http` and the tool `ledger-gen`.

**Features**

- **`ledger-gen`:** a small tool that generates fake statements in both formats with a fixed random seed. Use it for the tests and the measurements of this stage and of stage 5.
- **Parallel import:** `ledger import <directory> --account <name>` parses all files in the directory in parallel (`rayon` or threads). An error in one file does not stop the other files. The parsed results are then applied to the store by one writer, file by file, each file all-or-nothing.
- **Accounts in different currencies.** `ledger report --month <YYYY-MM> --currency <code>` converts each transaction to the report currency with the exchange rate of its date.
- **Exchange rates** come from an HTTP API, for example the Frankfurter API, which publishes the reference rates of the European Central Bank. Check the current URL and the limits in its documentation. The adapter:
  - fetches the rates for all needed dates concurrently, with a limit of 4 requests at the same time;
  - has a timeout of 5 seconds for each request;
  - retries timeouts and server errors 3 times with exponential backoff;
  - stores the rates in a local cache file (the rates of a past date do not change).
- **The core stays synchronous.** The CLI first gets all needed rates (async), then gives them to the core as a value (a rate table). This is the "functional core, imperative shell" design from phase 5. Write an ADR that compares it with an async port in the core.
- **Rates without floats:** store a rate as an integer with 6 decimal places, or use the `rust_decimal` crate. Select a rounding rule for the conversion to minor units, and document it.
- **Ctrl+C** during an import or a report stops the program cleanly. A file is imported completely or not at all.

**Acceptance criteria**

- [ ] The import of 100 generated files is faster in parallel than sequentially. The measurement is in `labs/ledger/docs/performance.md`.
- [ ] No test uses the real exchange rate API. The tests use a fake server (`wiremock`) and paused time.
- [ ] If the API is slow or down, the report fails within a known time with a clear message, or uses the cache.
- [ ] `ledger-core` does not depend on `tokio` or `reqwest`.

## Stage 4: HTTP API and database (phase 7)

**Code:** `labs/ledger/`. New crates: `ledger-store-sqlite` and `ledger-api`.

**Features**

- **SQLite store** with `sqlx` and migrations. Amounts as `INTEGER` minor units. The CLI selects the JSON store or the SQLite store by configuration.
- **HTTP API** with `axum`:

  | Method and path | Result |
  |---|---|
  | `GET /health` | 200 when the database works |
  | `GET /accounts`, `POST /accounts` | List and add accounts |
  | `GET /accounts/{id}` | One account with its balance |
  | `GET /accounts/{id}/transactions?from=&to=&category=&limit=&after=` | Transactions, with pagination |
  | `POST /accounts/{id}/transactions` | Add a transaction |
  | `POST /accounts/{id}/imports?format=` | Import a statement (the CSV text is the request body). Returns the number of imported and skipped transactions, or the errors with line numbers. |
  | `GET /reports/monthly?month=&currency=` | The monthly report |
  | `PUT /budgets/{category}`, `GET /budgets/status?month=` | Set a budget, and show the budget status |

- Errors as Problem Details (RFC 9457).
- Authentication with a bearer token from the configuration. Only `/health` is public.
- Configuration from the environment: the address, the database URL, the API token, the log format, and the base URL of the exchange rate API. Commit a `.env.example`.
- `tracing` with a request identifier for each request. JSON logs when `LOG_FORMAT=json`.
- Graceful shutdown.
- An import through the API is one database transaction.

**Acceptance criteria**

- [ ] The diff of `ledger-core` in this stage contains no HTTP or SQL code.
- [ ] The storage contract tests pass for the SQLite store.
- [ ] Every route has handler tests. 3–5 end-to-end tests run the whole service on port 0.
- [ ] A 500 response contains no internal details. The logs contain no token and no transaction descriptions.
- [ ] An import with one wrong line changes nothing in the database.

## Stage 5: Production ready (phase 8)

**Features**

- Property tests: the round trip of `Money`, the round trip of each import format, the idempotence of the import, and the invariant "the sum of all transactions equals the sum of all balances".
- Snapshot tests for the text report and for the JSON responses of the API.
- Mutation tests on `ledger-core`. Record the results in `docs/testing.md`.
- Benchmarks for the import and the monthly report with one million generated transactions. Profile and improve. Record the results in `docs/performance.md`.
- A CI pipeline in `.github/workflows/`: format, clippy, tests, and doc tests on Linux, Windows, and macOS; `cargo deny check`; coverage as an artifact.
- Release `1.0.0`: `CHANGELOG.md`, a git tag, release executables of `ledger-cli` for three operating systems, and a container image of `ledger-api` that runs as a user that is not root.
- Documentation: a `README.md` (install, use, develop), an architecture overview with links to the ADRs, and a runbook for the API.

**Acceptance criteria**

- [ ] The CI pipeline passes on the track branch in your private repository.
- [ ] Every property test and every snapshot test is in CI.
- [ ] No missed mutants in `Money` and in the categorization rules.
- [ ] The release has executables for three operating systems and a container image.
- [ ] A new developer can build, test, and run the ledger with only the README.

## Stage 6: Extension (phase 9, optional)

Select one option, or propose your own:

| Option | Module | Features |
|---|---|---|
| A | 9.4 | A terminal user interface with `ratatui`: transaction list, filters, categorization, and the report |
| B | 9.5 | A web front end for the API, or a WebAssembly build of the core |
| C | 9.6 | PostgreSQL, an import service behind a message queue, and distributed traces |

**Acceptance criteria**

- [ ] `ledger-core` does not change for options A and C. For option B, it changes only to make it compile to WebAssembly.
- [ ] The extension has tests and runs in CI.
