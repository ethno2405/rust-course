# Phase 6: Concurrency and Async

**Time:** 3–4 weeks
**Prerequisites:** Phase 5
**Code:** `labs/phase-6-concurrency/` and `labs/ledger/`

## Goal

At the end of this phase, you can:

- Decide between threads, async, data parallelism, and a sequential program.
- Explain `Send` and `Sync`, and what the compiler prevents and what it does not prevent.
- Share state with `Arc`, `Mutex`, `RwLock`, and atomics, and prevent deadlocks.
- Design with message passing: channels, worker pools, and actors.
- Explain how `async` and `.await` work: futures, polling, and the runtime.
- Write async programs with Tokio: tasks, `JoinSet`, `select!`, timeouts, and channels.
- Avoid blocking in async code, and handle cancellation and graceful shutdown.
- Limit concurrency, and retry with backoff.
- Test concurrent and async code without real time and without the network.

## Reading

| Source | Part |
|---|---|
| The Rust Programming Language (the Book) | "Fearless Concurrency", "Fundamentals of Asynchronous Programming: Async, Await, Futures, and Streams" |
| Mara Bos, *Rust Atomics and Locks* (free at marabos.nl/atomics) | The chapters about the basics of concurrency, atomics, and memory ordering. Read the later chapters (building your own locks) in phase 9 if you are interested. |
| Tokio tutorial (tokio.rs/tokio/tutorial) | All |
| Alice Ryhl, "Actors with Tokio" and "Async: What is blocking?" (ryhl.io) | All |
| Asynchronous Programming in Rust (rust-lang.github.io/async-book) | The first chapters |
| Carl Fredrik Samson, *Asynchronous Programming in Rust* | Optional. Explains how futures and runtimes work inside. |
| Jon Gjengset, "Crust of Rust" (YouTube) | The episodes about channels, and about atomics and memory ordering |

## Topics

### 6.1 Concurrency, parallelism, and when to use each

- **Concurrency:** many tasks make progress in the same time period. They can run on one CPU core, one after another in small steps.
- **Parallelism:** tasks run at the same time on many cores.

| Work | Example | Tool |
|---|---|---|
| CPU-bound, with independent data | Parse 1000 files, compute statistics | Data parallelism: `rayon` (topic 6.6), or scoped threads |
| A few long-running independent activities | A file watcher and a worker | OS threads |
| Many tasks that wait for I/O | An HTTP server with 10 000 connections, 100 HTTP requests | `async` with a runtime (Tokio) |
| Small or fast work | Most command-line tools | A sequential program. Measure first. |

- Concurrency adds complexity. Use it only when a measurement or a clear requirement says that you need it.

### 6.2 Threads

```rust
use std::thread;

fn parallel_sum(data: &[u64]) -> u64 {
    let (left, right) = data.split_at(data.len() / 2);
    thread::scope(|s| {
        let a = s.spawn(|| left.iter().sum::<u64>());
        let b = s.spawn(|| right.iter().sum::<u64>());
        a.join().expect("the left thread does not panic")
            + b.join().expect("the right thread does not panic")
    })
}
```

- `thread::spawn` starts a thread. Its closure must be `'static`: it cannot borrow local variables. Use `move` and give it owned data or an `Arc`.
- `thread::scope` (stable since Rust 1.63) starts threads that can borrow local data. All threads end before `scope` returns.
- `join()` waits for the thread and returns its result. If the thread panicked, `join()` returns `Err`.

### 6.3 `Send` and `Sync`

| Trait | Meaning | Not implemented by |
|---|---|---|
| `Send` | A value of this type can move to a different thread | `Rc<T>` (its count is not atomic), raw pointers |
| `Sync` | A `&T` can go to a different thread (`T` is `Sync` if `&T` is `Send`) | `Cell<T>`, `RefCell<T>` (their checks are not thread-safe) |

- The compiler implements these traits automatically for your types when all fields implement them.
- **The compiler prevents data races.** A program that compiles has no data races in safe code.
- **The compiler does not prevent:** deadlocks, race conditions in your logic (check-then-act on two separate lock calls), starvation, or wrong order of events. You must design against them.

### 6.4 Shared state

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0_u64));
let handles: Vec<_> = (0..4)
    .map(|_| {
        let counter = Arc::clone(&counter);
        thread::spawn(move || {
            for _ in 0..1000 {
                *counter.lock().expect("no thread panics with the lock") += 1;
            }
        })
    })
    .collect();
for handle in handles {
    handle.join().expect("the thread does not panic");
}
```

- `Arc<T>` gives shared ownership across threads. `Mutex<T>` gives exclusive access. The data is inside the mutex: you cannot use it without the lock.
- The lock is released when the `MutexGuard` is dropped (RAII). **Hold a lock for the shortest time.** Do not call unknown code or do I/O while you hold it.
- **Poisoning:** if a thread panics while it holds the lock, `lock()` returns `Err` in other threads. The data can be in a broken state.
- `RwLock<T>`: many readers or one writer. It is faster than `Mutex` only when reads are much more frequent and long.
- **Atomics** (`AtomicU64`, `AtomicBool`): lock-free operations on single values. Use them for counters and flags. For a counter, `Ordering::Relaxed` is enough. For a flag that publishes other data, use `Release` to store and `Acquire` to load. If you are not sure, read *Rust Atomics and Locks* before you use anything other than `SeqCst`.
- **Deadlock:** thread A holds lock 1 and waits for lock 2, and thread B holds lock 2 and waits for lock 1. Prevent it: always take locks in the same global order, hold one lock at a time, or put the data in one lock.

### 6.5 Message passing

> Do not communicate by sharing memory. Share memory by communicating. (Go proverb)

- A **channel** sends values from one thread to another. The value moves: the sender cannot use it after the send.
- `std::sync::mpsc::channel()` is unbounded. `mpsc::sync_channel(n)` is bounded: `send` blocks when `n` values wait. The `crossbeam-channel` crate has faster channels with more features.
- **Bounded channels give backpressure.** If the consumer is slow, the producer waits. An unbounded channel can grow without limit and use all memory.
- When all senders are dropped, the receiver gets an error (or `None`). Use this to stop a worker cleanly.
- **Worker pool:** N threads take jobs from one channel and send results on a second channel.
- **Actor:** a task that owns its state, and a channel that brings it messages. No locks: only the actor touches the state. A request that needs an answer carries a one-shot channel for the reply.

### 6.6 Data parallelism with `rayon`

```rust
use rayon::prelude::*;

let total: u64 = files
    .par_iter()
    .map(|path| count_lines(path))
    .sum();
```

- `par_iter()` splits the work over a thread pool. The code looks like a sequential iterator chain.
- Use it for CPU-bound work on independent items. Measure: for small inputs, the parallel version can be slower.

### 6.7 How async works

- An `async fn` returns a **future**: a value that represents work that is not complete. Calling it does nothing. The work starts when something **polls** the future.
- `.await` polls a future. If the future is not ready (for example it waits for a network packet), the current task gives the thread back to the runtime, and the runtime runs a different task.
- The compiler converts each `async fn` into a state machine. Each `.await` is a state.
- A **runtime** (executor) polls the tasks. It gets events from the operating system (epoll, kqueue, IOCP) and wakes the tasks that can make progress. The standard library has no runtime. Tokio is the most used runtime.
- **Async is cooperative.** A task runs until it reaches an `.await` that is not ready. A task that computes for a long time without `.await`, or calls a blocking function, stops all other tasks on its thread.
- `async fn` in traits is stable since Rust 1.75. Such a trait is not dyn-compatible. For `dyn Trait` with async methods, use the `async-trait` crate. Async closures (`async || { ... }`) are stable since Rust 1.85.

### 6.8 Tokio

```rust
use std::time::Duration;
use tokio::task::JoinSet;

#[tokio::main]
async fn main() {
    let mut tasks = JoinSet::new();
    for id in 0..3_u64 {
        tasks.spawn(async move {
            tokio::time::sleep(Duration::from_millis(100 * id)).await;
            id
        });
    }
    while let Some(result) = tasks.join_next().await {
        match result {
            Ok(id) => println!("task {id} is done"),
            Err(e) => eprintln!("a task failed: {e}"),
        }
    }
}
```

| Item | Use |
|---|---|
| `#[tokio::main]` | Start the multi-threaded runtime and run `main` on it |
| `tokio::spawn(future)` | Run a future as an independent task. It must be `Send + 'static`. |
| `JoinSet` | A group of tasks. Wait for each result. Dropping the set cancels its tasks. |
| `tokio::select!` | Wait for the first of several futures. The other futures are dropped (cancelled). |
| `tokio::time::timeout(d, future)` | Cancel a future after a time |
| `tokio::sync::mpsc`, `oneshot`, `broadcast`, `watch` | Channels: many to one, one reply, one to many, the latest value |
| `tokio::sync::Semaphore` | Limit how many tasks do something at the same time |
| `tokio::sync::Mutex` | A mutex that you can hold across an `.await` |
| `tokio::task::spawn_blocking` | Run blocking code on a separate thread pool |

- Add Tokio with only the features that you need, or `cargo add tokio --features full` for the labs.
- **`std::sync::Mutex` or `tokio::sync::Mutex`?** Use the standard mutex when you do not hold the lock across an `.await`. It is faster. Use the Tokio mutex only when you must hold the lock across an `.await`. A `std::sync::MutexGuard` is not `Send`, so the compiler rejects a task that holds it across an `.await`.

### 6.9 Blocking, cancellation, and shutdown

- **Blocking code in async:** `std::thread::sleep`, `std::fs` on slow disks, a synchronous HTTP client, a long CPU calculation, or a `std::sync::Mutex` that is held for a long time. Use the async version (`tokio::time::sleep`, `tokio::fs`), or `spawn_blocking`.
- **Cancellation:** in Rust, a future is cancelled when it is dropped. It stops at its current `.await` and never continues. Its local values are dropped normally.
- **Cancellation safety:** if `select!` drops a future that already did a part of its work, that work can be lost. Example: a future that read half a message from a socket. The Tokio documentation says for each method whether it is cancellation safe.
- **Graceful shutdown:**
  1. A signal starts the shutdown (`tokio::signal::ctrl_c()`).
  2. A `CancellationToken` (crate `tokio-util`) tells all tasks to stop.
  3. Each task stops accepting new work and completes or saves its current work.
  4. The main task waits for all tasks, with a time limit.

### 6.10 Limits, timeouts, and retries

- **Limit concurrency.** Without a limit, a loop that starts one task for each of 10 000 URLs opens 10 000 connections. Use a `Semaphore`, or `buffer_unordered(n)` on a stream:

  ```rust
  use futures::stream::{self, StreamExt};

  let results: Vec<_> = stream::iter(urls)
      .map(|url| fetch(&client, url))
      .buffer_unordered(4)
      .collect()
      .await;
  ```

- **Timeouts:** every network call needs a timeout. Without it, one slow server can stop your program forever.
- **Retries:** retry only errors that can be temporary (a timeout, HTTP 503), and only operations that are safe to repeat (idempotent). Wait longer after each attempt (exponential backoff), add a random part (jitter), and stop after a maximum number of attempts.
- **Errors in concurrent code:** decide what happens to the other tasks when one task fails: continue and collect the errors, or cancel all. Write the decision in the code.

### 6.11 Test concurrent and async code

- `#[tokio::test]` runs an async test.
- `#[tokio::test(start_paused = true)]` (feature `test-util`) pauses the time. `sleep` and `timeout` then complete immediately in the order of their deadlines. A test of a retry with a backoff of 30 seconds runs in milliseconds.
- Test with fake implementations of your ports (phase 5). For HTTP clients, use a local fake server (the `wiremock` crate) in place of the real service.
- Concurrency defects often appear only sometimes. Run a suspect test many times. The `loom` crate tests all thread orderings of small concurrent code. `tokio-console` shows the state of all tasks in a running program.

## Exercises

Make the workspace `labs/phase-6-concurrency/`.

### Exercise 6.1: Parallel word count

1. Count the words in all text files of a directory. Write a sequential version first.
2. Write a version with `thread::scope` and one thread for each CPU core (`std::thread::available_parallelism`).
3. Write a version with `rayon`.
4. Measure the three versions with a large set of files, in a release build. Explain the results.

### Exercise 6.2: Deadlock

1. Write `struct Bank { accounts: Vec<Mutex<Account>> }` and `fn transfer(&self, from: usize, to: usize, amount: u64)` that locks both accounts.
2. Run many transfers in both directions from several threads. Show the deadlock.
3. Fix it with a lock order. Write a test that runs 10 000 random transfers and checks that the total money does not change.
4. Write a second design with one actor thread that owns all accounts and receives transfer messages. Compare the two designs.

### Exercise 6.3: Worker pool with backpressure

1. Write a worker pool with N threads, a bounded job channel, and a result channel.
2. The producer reads lines from a large file and sends one job for each line. A job takes 1 ms.
3. Show that the memory stays small with a bounded channel. Then use an unbounded channel and compare the memory use.
4. Stop the pool cleanly: drop the sender, and wait for all workers.

### Exercise 6.4: An actor with Tokio

1. Write a cache actor: a task that owns a `HashMap<String, String>` and handles `Get`, `Set`, and `Delete` messages from an `mpsc` channel. `Get` carries a `oneshot::Sender` for the reply.
2. Write a `CacheHandle` with async methods that hide the messages. The handle implements `Clone`.
3. Use the handle from 100 tasks at the same time.
4. Write the same cache with `Arc<Mutex<HashMap<...>>>`. Compare the two designs: code size, testability, and what happens if a caller holds the data for a long time.

### Exercise 6.5: Concurrent HTTP requests

1. Fetch a list of URLs with `reqwest`. Limit the concurrency to 4.
2. Give each request a timeout of 5 seconds.
3. Retry timeouts and HTTP 5xx responses up to 3 times with exponential backoff. Do not retry 4xx responses.
4. Return a result for each URL: the body size, or the error.
5. Test with `wiremock`: a server that is slow, a server that fails two times and then works, and a server that returns 404.

### Exercise 6.6: Blocking

1. Start a runtime with one thread: `#[tokio::main(flavor = "current_thread")]`.
2. Start a task that prints a message every 100 ms. Start a second task that calls `std::thread::sleep` for 2 seconds. What happens to the first task?
3. Fix it two times: with `tokio::time::sleep`, and with `spawn_blocking`. When is each fix correct?

### Exercise 6.7: Graceful shutdown

1. Write a program with three worker tasks that take jobs from a channel. Each job takes 1–3 seconds.
2. On Ctrl+C, stop accepting new jobs, let the workers complete their current job, and exit. Use a `CancellationToken`.
3. If the workers do not stop within 5 seconds, exit anyway and print which jobs were not complete.

### Exercise 6.8: Test with paused time

1. Write a function `retry_with_backoff` that takes an async operation and a policy (attempts, base delay).
2. Test it with `start_paused = true`: check the number of attempts, the delay between attempts (use `tokio::time::Instant`), and the final error.
3. The tests must run in less than 100 ms in total.

## Project: Ledger, stage 3

See [../projects/ledger.md](../projects/ledger.md), stage 3.

## Done criteria

- [ ] I can decide between threads, async, `rayon`, and sequential code, and give the reason.
- [ ] I can explain `Send` and `Sync`, and which concurrency defects Rust prevents and which it does not prevent.
- [ ] I can show a deadlock and fix it.
- [ ] I can design with channels and actors, and explain backpressure.
- [ ] I can explain how a future is polled, and why blocking in async code is a problem.
- [ ] My network calls have timeouts and a concurrency limit, and my retries have backoff.
- [ ] My async programs shut down gracefully.
- [ ] My async tests use paused time and fake servers, and do not depend on the network.
- [ ] Ledger stage 3 is complete.

## Common problems

| Problem | Cause |
|---|---|
| `future cannot be sent between threads safely` | The task holds a value that is not `Send` (for example `Rc`, or a `std::sync::MutexGuard`) across an `.await`. Drop the value before the `.await`, or use `Arc`. |
| `borrowed value does not live long enough` with `tokio::spawn` or `thread::spawn` | The task must be `'static`. Move owned data or an `Arc` into it, or use `thread::scope` or `JoinSet` with owned data. |
| The program stops and does nothing | A deadlock, or a blocking call in an async task. Check the locks and the blocking calls. Use `tokio-console`. |
| `there is no reactor running` | A Tokio function was called outside a Tokio runtime, for example in a non-async test. Use `#[tokio::test]`. |
| Some results are lost when a `select!` loop runs | A future that is not cancellation safe was dropped. Read the cancellation safety notes of the method. |
| A test passes alone and fails with the other tests | Shared state between tests: a global, a fixed port, or a shared file. Give each test its own resources. |

## Next

[Phase 7: Web services and persistence](phase-7-web-services.md)
