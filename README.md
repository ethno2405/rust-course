# Rust Application Development Course

A self-study course in application development with Rust. You learn the Rust language, ownership, traits, error handling, concurrency, async, and web services. You also learn how to design software that is easy to change: the SOLID principles, other design principles, design patterns, architecture, testing, and professional practices. You build a real application: a personal finance ledger with a command-line interface and an HTTP API.

- **For:** anyone who can program in at least one language. You do not need to know Rust.
- **Environment:** a computer with Windows, macOS, or Linux. All tools are free. See [docs/software-setup.md](docs/software-setup.md).
- **Time:** approximately 6–7 months for phases 0–8 at 6–8 hours per week. Phase 9 is open-ended.

You can use the course alone, or with an AI coding agent as your mentor. The mentor assesses your knowledge, adapts the explanations to your background, reviews your code and your design, and tracks your progress.

## Quick start with an AI mentor

1. Clone the repository:

   ```sh
   git clone https://github.com/ethno2405/rust-course.git
   cd rust-course
   ```

2. Open an AI coding agent in the repository root. The agent must be able to read and write files and run commands. Examples: Claude Code (`claude`), OpenAI Codex CLI, GitHub Copilot in agent mode, Cursor, Gemini CLI. The agent reads its instructions from [AGENTS.md](AGENTS.md) (Claude Code reads [CLAUDE.md](CLAUDE.md), which imports `AGENTS.md`).

3. Type the first prompt:

   ```
   Start the course.
   ```

4. The mentor offers three assessment modes. Select one:

   | Mode | Time | Content |
   |---|---|---|
   | Quick | ~5 minutes | Questions about your background only |
   | Standard | ~15–20 minutes | Background questions and a short adaptive quiz |
   | Full | ~45 minutes | Background questions, a longer quiz, and practical items (code reading, a design review) |

5. The mentor asks about your project. The documents use a personal finance ledger. You can select a different domain, for example a task tracker or a workout log. If you do not know, the mentor uses the ledger.

6. The mentor makes your track branch, `track/<YYYY-MM-DD>`, with the start date in the name (for example `track/2026-10-19`). It saves your profile and your progress in the `learner/` directory on this branch. Then it tells you where to start.

In each later session, open the agent in the repository root and type `Continue the course.` The agent has no memory of earlier sessions. It finds your track branch, and reads your profile and progress from `learner/`.

## Your data

- The `learner/` directory contains `profile.md` (background, levels, project, environment, preferences, mentor observations) and `progress.md` (position, completed work, notes, the newest session log entries). Older session log entries go to `session-log-archive.md`.
- The `labs/` directory contains your code. From phase 8, `.github/workflows/` contains your CI pipeline.
- You can read and edit all files. The mentor uses the content that it finds.
- **Do not use real bank statements or other real personal data in the labs.** Use the generated sample data. The `.gitignore` file ignores database files and `.env` files.

### Branches

The `master` branch is the course baseline. It contains only the course documents. Each learner has a track branch `track/<YYYY-MM-DD>` that contains the course documents, the learner data, and the code of one learner. The date is the start date. The branch name contains no personal data.

| Branch | Content |
|---|---|
| `master` | Course documents |
| `track/<YYYY-MM-DD>` | Course documents, `learner/`, `labs/`, and `.github/workflows/` |

- At the end of each session, the mentor asks whether you want to commit your data and code. It commits only on your track branch.
- On `master`, the file `learner/.gitignore` ignores all files in `learner/`, so the files cannot go to `master` by accident. When the mentor makes a track branch, it clears this file in the first commit of the branch. After that, git tracks the learner data on the branch.
- The mentor pushes only when you agree, and only to a remote that you own (for example your fork), or to a remote where the owner permits track branches.
- **Your profile contains personal data.** All persons who can read the remote can read your track branch. If the course repository is public, push your track branch to a private repository (see "Share your data with git").
- Never merge a track branch into `master`.

### Get course updates

The course documents change on `master`. To get the changes on your track branch, merge `master`:

```sh
git switch master
git pull                       # if master has a remote
git switch track/<YYYY-MM-DD>
git merge master
```

A track branch does not change the course documents, so the merge has no conflicts.

### Share your data with git

**Private repository** (do this one time if the course repository is public):

1. Make a new, empty, private repository, for example on GitHub. GitHub does not permit a private fork of a public repository.
2. Add the private repository as a second remote: `git remote add private <url>`.

Push `master` to `origin`. Push your track branches to `private`. To share a track with a different person, give that person access to the private repository. If the course repository is private, you do not need a second remote. Use `origin` in place of `private` in the commands below.

**Send** (from the old clone or computer):

1. End the session with `Save my progress and end the session.` The mentor updates your data and offers to commit it.
2. Push your track branch: `git push -u private track/<YYYY-MM-DD>`.

**Receive** (in the new clone, or for a different person):

1. Clone the course repository. Add the private remote (`git remote add private <url>`), and fetch: `git fetch private`.
2. Switch to the track branch: `git switch track/<YYYY-MM-DD>`. Git makes a local branch from `private/track/<YYYY-MM-DD>`.
3. Start the agent and type `Continue the course.` The mentor continues from the "Current position" in `progress.md`.

To see all track branches, run `git branch -a --list "track/*" "*/track/*"`.

**Before you switch branches,** commit your changes. If `learner/` contains files other than `.gitignore` on `master` (local data from an older version of the course), copy the directory to a safe location first. A checkout can overwrite ignored files and does not give a warning.

**Partial data:**

| The branch contains | Result |
|---|---|
| `profile.md` and `progress.md` | The mentor continues where you stopped. |
| Only `progress.md` | The mentor looks for an earlier version in the git history. If there is none, it keeps your position and offers a short assessment to rebuild your profile. |
| Only `profile.md` | The mentor looks for an earlier version in the git history. If there is none, it asks for your current phase and completed exercises, and makes a new progress file. |
| Nothing | The mentor asks whether you started before. If not, it starts a new assessment. |

**Course version:** the progress file refers to phase and exercise numbers. If you merge a different version of the course documents, the mentor checks the position and corrects it with you.

## Start from scratch

The easiest way: make a new track branch from `master`. Your old branch does not change.

```sh
git switch -c track/<YYYY-MM-DD> master   # use the date today
echo "# Track learner data on this branch." > learner/.gitignore
git add -- learner/.gitignore
git commit -m "Track learner data on this branch"
```

**With the mentor:**

```
Reset the course. I want to start from scratch.
Reset only my progress.
Delete my code in labs/ and start phase 2 again.
```

The mentor asks what to reset (profile, progress, code, or all), shows what it will delete, and offers a backup. It deletes nothing before you confirm. It resets only a track branch, never `master`. The backup is a git tag `<branch>/backup-<date>`, for example `track/2026-10-19/backup-2026-11-02`.

**Without the mentor** (run in the repository root, on your track branch):

Bash:

```sh
git tag "$(git branch --show-current)/backup-$(date +%F)"   # optional backup of committed data and code
git rm -r -q --ignore-unmatch -- learner/profile.md learner/progress.md labs .github/workflows   # delete profile, progress, and code
rm -rf learner/profile.md learner/progress.md labs .github   # delete the remaining local files
git commit -m "Remove learner data to start the course again"
```

PowerShell:

```powershell
git tag "$(git branch --show-current)/backup-$(Get-Date -Format yyyy-MM-dd)"   # optional backup of committed data and code
git rm -r -q --ignore-unmatch -- learner/profile.md learner/progress.md labs .github/workflows   # delete profile, progress, and code
Remove-Item -Recurse -Force learner/profile.md, learner/progress.md, labs, .github -ErrorAction SilentlyContinue   # delete the remaining local files
git commit -m "Remove learner data to start the course again"
```

Uncommitted changes are lost. Commit them before the reset if you want to keep them in the backup tag. Skip the `git commit` command if the branch had no committed learner files.

## Example prompts

### First session

```
Start the course.
Start the course. Use the quick assessment.
Start the course. I know C# well. Use the full assessment for Rust and design only.
Start the course. I want to build a workout log, not the ledger.
```

### Each session

```
Continue the course.
What is my next step?
Summarize my progress.
Save my progress and end the session.
Commit my code in labs/.
```

### Learn a topic

```
Explain the borrow checker at my level.
Compare Rust traits with Java interfaces and C# extension methods.
Explain the dependency inversion principle with an example from my ledger.
When do I use a trait object and when do I use generics?
Why is it a problem to block in an async function?
```

### Exercises

```
I finished exercise 2.2. Check my solution in labs/phase-2-ownership.
Give me a hint for exercise 3.4. Do not give the solution.
I am blocked on exercise 6.3. Show me the full solution and explain it.
```

### Code and design review

```
Review my code in labs/phase-4-errors-apis.
Review the design of my ledger workspace against the SOLID principles.
Does my Repository trait break the interface segregation principle?
Is this clone necessary, or can I change the design?
```

### Compiler errors and debugging

```
I get error E0502 in src/report.rs. Help me understand it step by step.
My tokio program stops and does nothing. Help me find the cause.
My integration test passes alone but fails with the other tests. Why?
```

### Quizzes and reviews

```
Quiz me on phase 2.
I completed phase 5. Do the phase review.
Re-assess my design knowledge.
```

### Profile and progress

```
Mark exercise 1.4 as done.
Update my profile: I now prefer full solutions.
I want to change my project domain to a book catalog.
Reset the course. I want to start from scratch.
```

### Setup and tools

```
Help me install Rust on my computer.
Set up the debugger in VS Code for my ledger.
Which crate do you recommend for dates, and why?
```

### Errors in the course

```
The example in topic 3.4 does not compile. Help me open a pull request with the fix.
```

## Use fewer tokens

An AI agent has usage limits, or a price for each token. To use fewer tokens, keep sessions short: do one topic or one exercise in one session, then start a new session. The mentor gives you one tip at the start of each session. All tips are in [docs/token-usage.md](docs/token-usage.md).

## Use the course without an AI mentor

Start with [docs/README.md](docs/README.md). It has the roadmap, and each phase document has its reading, exercises, and done criteria.

## Repository

| Path | Content |
|---|---|
| [AGENTS.md](AGENTS.md) | Instructions for the AI mentor |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to send a fix for an error in the course |
| [docs/](docs/) | Course documents |
| [docs/mentor/](docs/mentor/) | Assessment and teaching instructions for the AI mentor |
| `labs/` | Your code (created during the course, only on your track branch) |
| `learner/` | Your profile and progress (only on your track branch). On `master`, only `.gitignore`. |
| `.github/workflows/` | Your CI pipeline (phase 8, only on your track branch) |
| `learner-backup-<date>/` | Local backups of learner data (not committed) |

## License

[MIT](LICENSE). Crates and other third-party code that you add during the course keep their own licenses.
