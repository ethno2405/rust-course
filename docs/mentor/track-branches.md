# Track Branches: New Branch and Reset

This document tells the AI mentor how to make a track branch for a new learner, and how to reset the data of a learner. The rules for track branches are in `AGENTS.md`, "Track branches". Read this document only when one of these procedures is necessary.

## New track branch

To make a branch for a new learner, do these steps:

1. Run `git switch -c track/<YYYY-MM-DD> master` with the date today. If a branch with this name exists, add a number: `track/<YYYY-MM-DD>-2`.
2. Replace the content of `learner/.gitignore` with one line: `# Track learner data on this branch.`
3. Run `git add -- learner/.gitignore`. Commit with the message "Track learner data on this branch".

## Reset (start from scratch)

When the learner asks to reset the course, start from scratch, or clear their data, do these steps. **A reset deletes data. Do not skip the confirmation.** Do a reset only on a track branch, never on `master`.

1. Tell the learner about the alternative: a new track branch (see "New track branch"). The old branch does not change. If the learner selects this alternative, stop here. Otherwise, ask what to reset. The learner can select one or more items:

   | Item | What is deleted |
   |---|---|
   | Profile | `learner/profile.md` |
   | Progress | `learner/progress.md` and `learner/session-log-archive.md` |
   | Code | All of `labs/` and `.github/workflows/`: committed files, uncommitted changes, untracked files, and build outputs |
   | All | All three items |

2. Show the exact list of what will be deleted. Run `git ls-files -- learner/ labs/ .github/workflows/` and `git status --porcelain --ignored -- learner/ labs/ .github/workflows/`, and show the result. `learner/.gitignore` is not deleted. If there are uncommitted changes, warn the learner that these changes cannot be recovered after the reset.
3. Offer a backup, and do it if the learner agrees: commit the uncommitted changes first (only if the learner agrees), then make a tag on the current commit: `git tag <branch>/backup-<YYYY-MM-DD>`, for example `track/2026-10-19/backup-2026-11-02`. The old data stays in the git history and in the tag.
4. Ask for a clear confirmation, for example: "Type 'yes, reset' to continue." Continue only after a clear "yes".
5. Do the reset:
   - Profile: run `git rm -q -- learner/profile.md` (if it is tracked), or delete the file.
   - Progress: run `git rm -q --ignore-unmatch -- learner/progress.md learner/session-log-archive.md`, then delete the files that remain.
   - Code: run `git rm -r -q --ignore-unmatch -- labs/ .github/workflows/`, then delete the `labs/` and `.github/` directories with all remaining files.
   - If you removed tracked files, commit with the message "Remove learner data to start the course again". Ask whether to push. Push only to a remote that the learner owns.
6. Tell the learner what you deleted and where the backups are.
7. If you deleted the profile, run the assessment in `docs/mentor/assessment.md` now. Do not ask whether the learner started the course before. If you deleted only the progress, create a new progress file from `docs/templates/progress-template.md` with the start date today.

Never delete files outside the learner files during a reset. Never delete `learner/.gitignore` or the course documents.
