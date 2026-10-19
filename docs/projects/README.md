# Projects and Tasks

This directory contains project specifications and task documents. A task document describes one piece of work: its goal, its steps, and its acceptance criteria. Use [../templates/task-template.md](../templates/task-template.md) for new tasks.

## File names

- Main project: `ledger.md`
- A different main project domain: `<domain>.md`, for example `task-tracker.md` (see [../mentor/project-selection.md](../mentor/project-selection.md))
- Side project: `<short-name>.md`, for example `chat-server.md`
- Task: `task-<phase>-<number>-<short-name>.md`, for example `task-4-01-import-command.md`

## Main project

| Project | Phases | Document |
|---|---|---|
| Personal finance ledger | 1–9 | [ledger.md](ledger.md) |

## Side projects

Side projects give more practice. Do them when you want more practice in a phase, or between phases. Make a separate document here when you start a side project.

| Project | Phase | Content |
|---|---|---|
| Text adventure | 1 | Rooms and items as structs and enums, a command parser with `match`, a state machine |
| Markdown to HTML converter | 2–3 | String slices and lifetimes, iterators, a parser that borrows its input |
| `wc`, `head`, and `cut` clones | 4 | Command-line tools with `clap`, errors, CLI tests |
| Static site generator | 4–5 | Templates, a file tree, the open/closed principle for output formats |
| Multi-user chat server over TCP | 6 | Tokio, actors, broadcast channels, graceful shutdown |
| Web crawler | 6 | Concurrency limits, timeouts, retries, deduplication, `robots.txt` |
| URL shortener service | 7 | `axum`, SQLite, configuration, rate limits |
| A small library on crates.io | 8 | API design, documentation, semantic versioning, CI, `cargo publish` |
| Python extension | 9 | PyO3, `unsafe` boundaries, performance comparison |

## Tasks

| Task | Phase | Status |
|---|---|---|
| *(no tasks yet)* | | |
