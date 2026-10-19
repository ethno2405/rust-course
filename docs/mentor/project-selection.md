# Project Selection

This document tells the AI mentor how to help the learner select the domain of the main project, and how to adapt the course to the selected domain. The learner makes the decision. The mentor suggests.

The course documents use the reference project: a personal finance ledger (`docs/projects/ledger.md`). A learner can build a different application with the same stages, if the domain motivates them more. The language features, the design principles, and the practices stay the same. Only the domain changes.

## When to do the selection

| Situation | Action |
|---|---|
| At the end of the first assessment | Offer the selection. The learner can postpone it. Until then, use the ledger. |
| The learner asks ("I want to build something different") | Do the selection. |
| The learner wants to change the domain after stage 0 | Do the selection again. Tell the learner the cost: they rewrite the earlier stages for the new domain, or continue the old code and change only the new stages. Record the old domain in the notes. |

Recommend that the learner keeps the domain after stage 1. A change after stage 2 costs much time.

## Step 1: Ask

Ask these questions. Use the answers that are already in the profile, and do not ask them again.

1. Do you want the easiest path through the documents (the ledger), or an application that is closer to your interests?
2. Is there an application that you want to use yourself?
3. Is there a domain from your work that you know well? (A known domain lets you focus on the code. Do not use confidential data or code from your employer.)

## Step 2: Suggest

1. Put the ledger first, unless the answers give a clear reason for a different domain.
2. Suggest 1–2 alternatives from the table below, or a domain that the learner proposes.
3. Check a proposed domain against the requirements below. If it does not meet a requirement, tell the learner which stage gets harder, and suggest a change to the domain.

### Requirements for a domain

The domain must give the learner the same problems as the ledger. Each stage needs these elements:

| Element | Stage | Ledger | Why the course needs it |
|---|---|---|---|
| A value type with rules | 0 | `Money`: an amount in minor units and a currency. No addition of different currencies. | Newtypes, operator traits, invariants, "make illegal states unrepresentable" |
| Entities with an identity | 0 | `Transaction`, `Account` | Ownership of collections, identifiers as newtypes |
| An external file format | 1 | CSV bank statements | Parsing, error handling with line numbers, serde |
| A second, different file format | 2 | A second bank format with a different column order, date format, and decimal separator | The open/closed principle: add a format without a change to the core |
| Rules that the user configures | 2 | Categorization rules: "a description that contains 'coffee' → category Food" | The strategy pattern, traits, closures |
| Aggregation and reports | 2 | Spending per category and month, budget against actual | Iterators, maps, snapshot tests |
| Data from an external HTTP service | 3 | Exchange rates | Async, timeouts, retries, concurrency limits, test doubles for HTTP |
| Many independent files to process | 3 | A directory of statements | Threads, channels, `rayon` |
| Resources for an HTTP API | 4 | Accounts, transactions, reports | REST design, SQL, transactions |
| A calculation that has properties | 5 | The sum of all transactions equals the sum of all balances | Property-based tests |

### Alternative domains

| Element | Task tracker | Workout log | Book catalog |
|---|---|---|---|
| Value type with rules | `Duration` estimate (no negative values), `Priority` | `Weight` with a unit (kg, lb; no addition of different units without conversion), `Reps` | `Isbn` (ISBN-10 and ISBN-13 with a check digit) |
| Entities | `Task`, `Project` | `Workout`, `Exercise`, `Set` | `Book`, `Author`, `Shelf` |
| File format 1 | CSV export of a task tool | CSV export of a fitness application | CSV export of a reading website |
| File format 2 | `todo.txt` format | A different CSV layout, or GPX for runs | A library catalog export in JSON |
| User rules | Auto-tags: "a title that contains 'bug' → tag Defect" | Exercise groups: "'bench press' → group Chest" | Auto-shelves: "a subject that contains 'history' → shelf History" |
| Reports | Time per project and week, tasks done per week, overdue tasks | Volume per muscle group and week, personal records | Books per year, pages per month, reading goal against actual |
| External HTTP data | Public holidays for due dates (for example the Nager.Date API) | Weather for outdoor workouts (an open weather API) | Book metadata by ISBN (for example the Open Library API) |
| Many files | A directory of exports from many projects | A directory of exports from many devices | A directory of exports from many shelves |
| Property for tests | The sum of time per project equals the total time | Total volume equals the sum of weight × reps | The ISBN-10 and ISBN-13 conversion is reversible |

Do not invent API details. Before the learner uses an external API, ask them to check its documentation: the URL, the limits, and whether it needs a key. In tests, always use a fake HTTP server, not the real service.

## Step 3: Record

When the learner selects a domain, fill in the "Project" section of `learner/profile.md`:

- The domain and a short description of the application
- The mapping of each element in the table "Requirements for a domain" to the learner's domain
- The external HTTP service for stage 3 (or "to be selected in phase 6")

## Adapt the course to the domain

In every session, if the domain in the profile is not the ledger:

1. Read the phase document and `docs/projects/ledger.md` as usual.
2. Before you give the learner a project-specific detail (a type name, a command, a file format, an acceptance criterion), translate it to the learner's domain with the mapping in the profile.
3. Keep the difficulty the same. If a translated feature is easier than the ledger feature, add a rule that gives the same problem. Example: a task tracker has no currencies, so make the `Duration` type reject negative values and overflow, and add units (minutes, hours).
4. The exercises in the phase documents stay as they are. Only the project stages change.
5. If many learners use the same domain, suggest a project document for it in `docs/projects/` (for example `task-tracker.md`).

For the ledger, use the documents as they are.
