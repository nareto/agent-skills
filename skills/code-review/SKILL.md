---
name: code-review
description: >-
  Review the changes on the current branch against the branch they will merge
  into, and record each finding with a severity from P0 to P3, where P0 and P1
  should block the merge. Use when asked to review a branch, a pull request
  that is checked out, or uncommitted changes, or as an automated review gate
  before a pull request goes to human review. Only reads code and writes a
  findings file; never fixes anything.
compatibility: Requires git and a coding agent that can read files and run shell commands.
metadata:
  version: "0.1.0"
---

# Code review

You review a change and record what you find. You do not fix anything.

## Find the change

1. Pick the base: the branch the change will merge into. Use the one the user names; otherwise the first of `origin/HEAD`, `origin/main`, `origin/master`, `main` and `master` that exists.
2. Run `git merge-base <base> HEAD`. The change is everything since that commit, including uncommitted changes to tracked files.
3. List the changed files with `git diff --stat <merge-base>` and read the full change with `git diff <merge-base>`.
4. New files that are not tracked yet do not show up in `git diff`. Check `git status --short` and read the untracked files that belong to the change.
5. Open any surrounding code, callers, tests or docs you need to judge the change.

## What to look for

- bugs and incorrect behavior, including edge cases and error paths;
- security problems (injection, secrets, unsafe permissions, unvalidated input);
- bad patterns that will make the code hard to change or easy to break;
- broken contracts: changed public APIs, CLI flags, file formats, config keys or documented behavior without matching updates;
- weak tests: missing coverage for the new behavior, assertions that cannot fail, tests that do not exercise what they claim.

Judge only the change and what it affects. Do not report problems in untouched code unless the change makes them worse or depends on them.

## Severity

Ask one question first: should this stop the merge? If yes, it is P0 or P1. If no, it is P2 or P3.

- **P0**: blocks the merge and is severe: data loss, security hole, crash or corruption in a common path, broken build or release.
- **P1**: blocks the merge: a real bug, broken contract or missing test for risky behavior that must be fixed before merging.
- **P2**: should be fixed but does not block the merge: maintainability problems, unclear behavior in rare cases, gaps in docs or tests.
- **P3**: nit: naming, wording, small style points.

Do not inflate severity to get attention, and do not report the same problem twice at different severities.

## The findings file

Write findings to the path the user gives, or by default to `.code-review/findings.yaml` at the repository root. When you create the `.code-review/` folder, also create `.code-review/.gitignore` containing `*`, so the review never shows up in `git status`.

The file is YAML with this exact shape:

```yaml
findings:
  - severity: P1            # P0, P1, P2 or P3
    file: src/example.rs    # path relative to the repository root
    line: 42                # optional, 1-based line in the new version of the file
    title: One-line summary of the problem
    why: What goes wrong, for whom, and under which conditions.
    suggestion: Optional concrete fix or direction.
```

`severity`, `file`, `title` and `why` are required. An empty file and `findings: []` both mean no findings.

If the file already exists, it holds findings from an earlier review of the same change:

1. Read it before you review.
2. A finding is already listed when it points at the same problem, even if the wording or line differs.
3. Append only new findings to the end of the `findings:` list, and keep every existing entry exactly as it is.
4. If you have nothing new, do not touch the file. An untouched file is how a script running the review again knows it is complete.

When you finish, tell the user how many findings you recorded, how many block the merge (P0 or P1), and the title of each blocking one.

## Rules

- The only files you may create or edit are the findings file and, for the default path, `.code-review/.gitignore`.
- Never edit, create, delete, format or stage any other file.
- Never commit, push, switch branches, stash or run any command that changes git state.
- Running read-only commands (git log, git diff, grep, the test suite) is fine, as long as they leave no changes behind.
- Keep `why` and `suggestion` short and specific. Quote identifiers, not whole files.
