# Token Usage

An AI agent reads and writes text in tokens. A token is a part of a word. The usage limits and the price of an AI agent depend on the number of tokens. This document tells you how to use fewer tokens in the course. The AI mentor gives you one of these tips at the start of each session.

## Why a session uses many tokens

With each message, the agent sends the full conversation to the AI model again: the instructions in `AGENTS.md`, the files that the agent read, the command output, and all earlier messages. Thus a message in a long session costs more than a message in a short session.

## Tips

1. **Keep sessions short.** Do one topic or one exercise in one session. Then say `Save my progress and end the session`, and start a new session. The mentor saves your position in `learner/progress.md`, so the new session loses nothing.
2. **Select a smaller AI model.** A smaller model is sufficient for explanations, quizzes, and simple code reviews. Use the largest model only for difficult debugging or design work. Your agent has a setting for the model.
3. **Run long commands in your own terminal.** Examples are a full build, a test run, or `cargo clippy` on the full workspace. Paste only the important lines.
4. **Paste only the first compiler error.** Later errors are often results of the first error. Use quiet output, for example `cargo test -q` and `cargo build -q`.
5. **Ask for a hint first.** A hint is shorter than a full solution, and you learn more.
6. **Ask for a review of your changes only.** For example: `Review my changes since the last commit.`
7. **Do not let the agent read large files that it does not need.** Examples are `target/`, the full `Cargo.lock`, generated files, and logs.
8. **Switch off the agent tools that you do not need.** Each tool of the agent (for example a browser tool or a plugin) adds its description to each message. Switch on a tool only when you need it.
9. **Ask a precise question.** `Explain why this borrow of v fails` uses fewer tokens than `Explain the borrow checker`.

To stop the tips at the start of each session, say `Stop the token usage tips.`
