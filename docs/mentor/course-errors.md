# Errors in the Course

This document tells the AI mentor what to do when the mentor or the learner finds an error in the course. Read it only when you find an error, or when the learner asks for a change to a course document.

The course documents and the mentor instructions can contain errors: wrong facts, wrong commands, broken links, unclear text, code examples that do not compile, or crate APIs that changed. When you or the learner find an error, suggest a pull request (PR) with the fix to the course repository. Then all learners get the fix.

1. **Check that it is an error.** Reproduce it: run the command, compile the code, or read the source (for example the crate documentation for the version in `Cargo.lock`).
2. **Tell the learner** what is wrong and what the fix is. Suggest a PR. Explain that a PR sends the fix to the maintainers of the course for a review.
3. **If the learner agrees,** do these steps:
   1. Commit or back up the learner work on the track branch first (see `AGENTS.md`, "Session end", step 4).
   2. Update the local `master` from the course repository: `git switch master`, then `git pull`. Make a fix branch from `master`: `git switch -c fix/<short-name>`. **Never make a fix branch from a track branch:** a track branch contains personal data.
   3. Make the fix. Keep it small: one error for each PR. Follow `AGENTS.md`, "Writing style" and "How to add documents". Check the fix: compile the changed code, run the changed command, and check the links.
   4. Commit only the files of the fix, with a one-line commit message in the imperative. Never commit learner files (see `AGENTS.md`, "Track branches").
   5. Push the fix branch to the fork of the learner. Most learners cannot push to the course repository. If the learner has no fork, help them make one: `gh repo fork --remote --remote-name fork`, or the "Fork" button on GitHub and `git remote add fork <url>`. Then push: `git push -u fork fix/<short-name>`.
   6. Open the PR against the `master` branch of the course repository: `gh pr create`, or the link that `git push` prints. In the description, write what was wrong, how it was found, the fix, and how the fix was checked. **Never put profile data, progress data, or learner code in a PR.**
   7. Switch back to the track branch. To use the fix before the maintainers merge it, merge the fix branch: `git merge fix/<short-name>`. Later, `git merge master` brings the merged fix. If the maintainers changed the fix in the review, a conflict can occur: resolve it with the version from `master`.
4. **If the learner does not want a PR,** or cannot make one (for example no GitHub account): make the fix on a local fix branch in the same way, and merge it into the track branch. Tell the learner that the fix exists only in their copy. Write it in "Notes for later sessions" in `learner/progress.md`, so that later sessions know about it.
5. **If the correct fix is not clear,** suggest a GitHub issue in place of a PR. The issue describes the error, the steps to see it, and the expected result.

The learner decides about each push and each PR. Do not push or open a PR without the agreement of the learner.
