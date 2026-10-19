# Phase 7: Web Services and Persistence

**Time:** 3–4 weeks
**Prerequisites:** Phase 6
**Code:** `labs/phase-7-web/` and `labs/ledger/`

## Goal

At the end of this phase, you can:

- Design a resource-oriented HTTP API with correct methods, status codes, and error bodies.
- Write an HTTP service with `axum`: routes, extractors, shared state, middleware, and error mapping.
- Store data in SQLite with `sqlx`: migrations, queries that the compiler checks, transactions, and a connection pool.
- Keep the domain core independent of HTTP and SQL, with adapters on both sides.
- Load typed configuration from the environment, and fail at startup if it is wrong.
- Add structured logs and traces with `tracing`.
- Protect the service at its boundary: validation, authentication, limits, and safe handling of secrets.
- Test the service with fast in-process tests and with full integration tests.

## Reading

| Source | Part |
|---|---|
| Luca Palmieri, *Zero To Production in Rust* | All chapters about the first service, tests, the database, telemetry, configuration, and error handling. The book uses `actix-web` and PostgreSQL. The course uses `axum` and SQLite. The concepts are the same. |
| `axum` documentation and examples (docs.rs/axum, the `examples/` directory in the repository) | The front page, the modules `extract`, `response`, and `middleware`, and the examples for errors, state, and graceful shutdown |
| `sqlx` documentation (docs.rs/sqlx) | The front page, the macros `query!` and `query_as!`, migrations, and `sqlx-cli` |
| `tracing` documentation (docs.rs/tracing) and the Tokio tutorial about tracing | Spans, events, subscribers |
| MDN Web Docs, "HTTP" (developer.mozilla.org) | Methods, status codes, headers, CORS |
| OWASP Top 10 (owasp.org) | The list and the summary of each risk |
| RFC 9457, "Problem Details for HTTP APIs" | The format of error bodies |

## Topics

### 7.1 HTTP API design

- **Resources** are nouns: `/accounts`, `/accounts/{id}`, `/accounts/{id}/transactions`. **Methods** are the verbs:

  | Method | Use | Safe | Idempotent |
  |---|---|---|---|
  | `GET` | Read | Yes | Yes |
  | `POST` | Create, or start a process | No | No |
  | `PUT` | Replace a resource completely | No | Yes |
  | `PATCH` | Change a part of a resource | No | No |
  | `DELETE` | Delete | No | Yes |

- **Status codes:**

  | Code | Use |
  |---|---|
  | `200 OK` | Success with a body |
  | `201 Created` | A resource was made. Add a `Location` header. |
  | `204 No Content` | Success without a body |
  | `400 Bad Request` | The request is malformed (for example invalid JSON) |
  | `401 Unauthorized` | No valid credentials |
  | `403 Forbidden` | Valid credentials, but no permission |
  | `404 Not Found` | The resource does not exist |
  | `409 Conflict` | The request conflicts with the current state (for example a duplicate) |
  | `422 Unprocessable Content` | The request is well-formed, but the values break a rule |
  | `500 Internal Server Error` | A defect or an unexpected failure. Do not show internal details to the client. |
  | `503 Service Unavailable` | Temporary. The client can retry later. |

- **Error bodies:** use one format for all errors. RFC 9457 ("Problem Details") defines a JSON format with `type`, `title`, `status`, and `detail`.
- **Lists:** add pagination (`?limit=50&after=<cursor>`) from the start. A list without a limit becomes slow when the data grows.
- **Versions:** decide how you change the API without a break of clients. Add fields; do not remove or rename them. Use `/v1/` in the path if you expect breaking changes.

### 7.2 `axum`

```rust
use axum::{
    Json, Router,
    extract::{Path, State},
    http::StatusCode,
    response::{IntoResponse, Response},
    routing::get,
};
use std::sync::Arc;

#[derive(Clone)]
struct AppState {
    ledger: Arc<LedgerService>,
}

async fn get_account(
    State(state): State<AppState>,
    Path(id): Path<u64>,
) -> Result<Json<AccountResponse>, ApiError> {
    let account = state.ledger.account(AccountId::new(id)).await?;
    Ok(Json(AccountResponse::from(account)))
}

fn router(state: AppState) -> Router {
    Router::new()
        .route("/health", get(|| async { StatusCode::OK }))
        .route("/accounts/{id}", get(get_account))
        .with_state(state)
}

enum ApiError {
    NotFound,
    Internal(anyhow::Error),
}

impl From<LedgerError> for ApiError {
    fn from(error: LedgerError) -> Self {
        match error {
            LedgerError::AccountNotFound(_) => ApiError::NotFound,
            other => ApiError::Internal(other.into()),
        }
    }
}

impl IntoResponse for ApiError {
    fn into_response(self) -> Response {
        match self {
            ApiError::NotFound => StatusCode::NOT_FOUND.into_response(),
            ApiError::Internal(error) => {
                tracing::error!(error = ?error, "internal error");
                StatusCode::INTERNAL_SERVER_ERROR.into_response()
            }
        }
    }
}
```

- A **handler** is an async function. Its parameters are **extractors**: `Path`, `Query`, `Json`, `State`, `HeaderMap`. Its return type implements `IntoResponse`.
- The order of extractors matters: an extractor that consumes the body (`Json`, `String`) must be the last parameter.
- **Route paths:** since `axum` 0.8, a path parameter is `{id}`. Older versions and older examples use `:id`. Check the version in your `Cargo.toml`.
- **Middleware** comes from `tower` and `tower-http`: `TraceLayer` (logs), `TimeoutLayer`, `CorsLayer`, `RequestBodyLimitLayer`, `CompressionLayer`. Add them with `.layer(...)`.
- Start the server with `axum::serve(listener, app)`, and add `.with_graceful_shutdown(signal)` (phase 6).

### 7.3 State, dependencies, and the hexagon

- The HTTP layer is a **driving adapter**. It converts HTTP requests to calls of the use cases in the core, and converts the results to HTTP responses. It contains no business rules.
- **Data transfer objects (DTOs):** define separate request and response types in the HTTP adapter (`CreateAccountRequest`, `AccountResponse`). Convert them to and from domain types. The API contract then stays stable when the domain types change, and the domain does not depend on `serde` names.
- **Shared state:** the state must implement `Clone`. Put the services in an `Arc`.
- **Async ports and trait objects:** a trait with `async fn` is not dyn-compatible (phase 6, topic 6.7). Select one of these designs:

  | Design | Advantage | Cost |
  |---|---|---|
  | `#[async_trait]` on the port, and `Arc<dyn Port>` in the state | Simple types. Select the adapter at run time. | A heap allocation for each call (small next to a database query) |
  | A generic state: `AppState<S: Store>` | Static dispatch, no extra crate | The generic parameter appears in many signatures |
  | An enum of the adapters | No trait objects | The core or the composition root must know all adapters |

  Write the decision in an ADR.

### 7.4 Persistence with SQLite and `sqlx`

- **SQLite** is a database in one file. It needs no server. It is a good fit for local applications and for many small services. Change to PostgreSQL later if you need many writers on many machines: `sqlx` supports both.
- **Migrations:** SQL files that change the schema, in order. Make one with `sqlx migrate add <name>`. Run them with `sqlx migrate run`, or from the program with `sqlx::migrate!().run(&pool)`. Never change a migration after it ran on a real database. Add a new one.
- **Queries checked at compile time:** `sqlx::query_as!(AccountRow, "SELECT id, name FROM accounts WHERE id = ?", id)` connects to the database at compile time (with `DATABASE_URL`) and checks the SQL and the types. For builds without a database (for example CI), run `cargo sqlx prepare`, and commit the `.sqlx/` directory.
- **Bind parameters** (`?` or `$1`) protect against SQL injection. Never build SQL with `format!` and user input.
- **Transactions:** `let mut tx = pool.begin().await?;` ... `tx.commit().await?;`. If the transaction is dropped without `commit`, it rolls back. A transfer between two accounts must be one transaction.
- **Connection pool:** `SqlitePool` keeps open connections. Create it one time at startup, and share it.
- **Money in the database:** store the amount as an `INTEGER` in minor units, and the currency as `TEXT`. Never use `REAL`.
- **Map rows to domain types in the adapter.** A `struct AccountRow` matches the table. A `TryFrom<AccountRow> for Account` checks the rules. A row that breaks a rule is an error, not a panic.
- **ORMs:** `sqlx` is not an object-relational mapper (ORM). You write the SQL. `diesel` and `sea-orm` are ORMs. Learn SQL first. Then decide whether an ORM helps your project.

### 7.5 Configuration

- Read configuration from the environment: environment variables, and a `.env` file for local work (the `dotenvy` crate). Commit a `.env.example`. Do not commit `.env`.
- Load it into a typed struct at startup. Validate it, and fail with a clear message if it is wrong (fail fast). Do not read environment variables in the middle of the program.
- Order of precedence: the defaults in the code, then a configuration file, then environment variables. The `config` and `figment` crates do this. For a small service, a function that reads a few variables is enough.
- Keep secrets in a type that does not print them: the `secrecy` crate gives `SecretString`, whose `Debug` output is `[REDACTED]`.

### 7.6 Observability with `tracing`

- `tracing` records **events** (one point in time, like a log line) and **spans** (a period of time with a start and an end, like a request). Events inside a span carry the fields of the span.
- `#[tracing::instrument]` makes a span for each call of a function, with its arguments as fields. Use `skip(...)` for large or secret arguments.
- Use fields, not text: `tracing::info!(account_id = %id, amount = %amount, "transaction added")`. A log system can then search and count by field.
- `tracing-subscriber` prints the events. Use readable text in development and JSON in production. Set the level with `RUST_LOG=ledger=debug,tower_http=info`.
- `tower_http::trace::TraceLayer` makes a span for each request with the method, the path, the status, and the duration. Add a request identifier to each span.
- **Never log secrets or personal data:** passwords, tokens, full account numbers, the descriptions of a user's transactions.

### 7.7 Security at the boundary

- **Validate all input** at the boundary. Convert it to domain types (phase 4, "parse, don't validate"). Return 422 with a clear message for values that break a rule.
- **Authentication:** for an API with one user, a long random token in the `Authorization: Bearer` header is enough for the labs. Compare tokens in constant time (the `subtle` crate). For users with passwords, store only a slow hash: `argon2`. Never store a password in clear text, and never write your own cryptography.
- **Limits:** a maximum body size, a request timeout, and a maximum page size for lists.
- **Errors:** a 500 response does not contain the internal error message. Log the details on the server.
- **CORS:** enable it only for the origins that need it.
- **Dependencies:** run `cargo audit` (phase 8) to find crates with known vulnerabilities.
- **Bind address:** in the labs, bind to `127.0.0.1`. Bind to `0.0.0.0` only in a container or behind a firewall.
- Read the OWASP Top 10. For each risk, write one sentence about how your service handles it.

### 7.8 Test a service

| Level | Method | Speed |
|---|---|---|
| Use cases | Unit tests in the core with fake ports (phase 5) | Very fast |
| Handlers | Call the `Router` directly with `tower::ServiceExt::oneshot`. No network. Fake or in-memory ports. | Fast |
| Database adapter | Tests against a real SQLite database: a new in-memory database (`sqlite::memory:`) or a temporary file for each test, with the migrations | Fast |
| The whole service | Start the server on port 0 (the OS selects a free port), and send real HTTP requests with `reqwest` | Slower. Write a few. |

```rust
use axum::body::Body;
use axum::http::{Request, StatusCode};
use tower::ServiceExt; // for `oneshot`

#[tokio::test]
async fn health_returns_ok() {
    let app = router(test_state());
    let request = Request::builder().uri("/health").body(Body::empty()).unwrap();
    let response = app.oneshot(request).await.unwrap();
    assert_eq!(response.status(), StatusCode::OK);
}
```

- `#[sqlx::test]` makes a new test database for each test and runs the migrations. Read its documentation for SQLite.
- Each test makes its own data. A test does not depend on data from a different test.

## Exercises

Make the workspace `labs/phase-7-web/`.

### Exercise 7.1: First service

1. Write a service with `GET /health` and `POST /echo`. `/echo` accepts `{"message": "..."}` and returns it. A message longer than 280 characters returns 422 with a Problem Details body.
2. Add `TraceLayer`, a timeout of 10 seconds, and a body limit of 64 KB.
3. Write handler tests with `oneshot` for each status code.

### Exercise 7.2: A REST API in memory

1. Write a notes API: create, read, list (with pagination), update, and delete notes. Store them in memory behind a port.
2. Use the correct method and status code for each case, including 201 with `Location`, 404, and 409 for a duplicate title.
3. Use separate request and response types.
4. Write handler tests for every route and every error.

### Exercise 7.3: SQLite and `sqlx`

1. Add a SQLite adapter for the notes port. Write the schema as a migration.
2. Use `query!` or `query_as!`. Run `cargo sqlx prepare` and build with `SQLX_OFFLINE=true`.
3. Add accounts and a transfer endpoint that moves an amount between two accounts in one transaction. Write a test that proves that a failed transfer changes nothing.
4. Run the contract tests from phase 5 (exercise 5.3) against both the memory adapter and the SQLite adapter.

### Exercise 7.4: Configuration and tracing

1. Load the configuration (address, database URL, log format, API token) from the environment into a typed struct. A missing or invalid value stops the program with a clear message.
2. Add `#[tracing::instrument]` to the use cases. Log in text locally and in JSON when `LOG_FORMAT=json`.
3. Add a request identifier to each request span. Find all log lines of one request.
4. Check the logs: no token, no secret, and no note content appear in them.

### Exercise 7.5: Security

1. Protect all routes except `/health` with a bearer token. Compare the token in constant time.
2. Add a `users` table with Argon2 password hashes and a `POST /login` endpoint that returns a token.
3. Write a test that sends a title like `'); DROP TABLE notes; --`. Show that the table still exists.
4. Go through the OWASP Top 10. Write one line for each risk about your service.

### Exercise 7.6: Integration tests

1. Write a helper that starts the whole service on port 0 with a new test database, and returns its address.
2. Write 3–5 end-to-end tests with `reqwest`: create, read, a failed login, and a request without a token.
3. Run the tests in parallel. They must not share any data.

## Project: Ledger, stage 4

See [../projects/ledger.md](../projects/ledger.md), stage 4.

## Done criteria

- [ ] I can design resources, methods, status codes, and error bodies for an HTTP API.
- [ ] I can write an `axum` service with extractors, state, middleware, and an error type that maps to responses.
- [ ] My HTTP layer has no business rules, and my core has no HTTP or SQL dependencies.
- [ ] I can write migrations, queries that the compiler checks, and transactions with `sqlx`.
- [ ] My service loads typed configuration and fails fast on a wrong value.
- [ ] My service writes structured logs with request identifiers, and no secrets in the logs.
- [ ] I can explain how my service handles each OWASP Top 10 risk.
- [ ] I test my service at the level of the use cases, the handlers, the database adapter, and the whole service.
- [ ] Ledger stage 4 is complete.

## Common problems

| Problem | Cause |
|---|---|
| A long compiler error that says the handler does not implement `Handler` | An argument is not an extractor, the body extractor is not the last argument, or the future is not `Send`. Add `#[axum::debug_handler]` to the handler for a clearer error. |
| A route returns 404 although it exists | The path syntax of a different `axum` version (`:id` in place of `{id}`), or a nested router with a wrong prefix. |
| `query!` fails to compile: `DATABASE_URL must be set` | Set `DATABASE_URL` in `.env`, or use offline mode with `cargo sqlx prepare` and `SQLX_OFFLINE=true`. |
| `database is locked` with SQLite | Many writers at the same time. Keep transactions short, enable WAL mode, or use one writer. |
| Tests fail when they run in parallel | The tests share a database file or a port. Give each test its own database and port 0. |

## Next

[Phase 8: Professional practices](phase-8-professional-practices.md)
