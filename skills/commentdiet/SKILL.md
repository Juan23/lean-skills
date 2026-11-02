---
name: commentdiet
description: Code review focused on comments. Proposes clearer names and code in place of explanatory comments, trims the comments that stay, and deletes noise and stale ones. Reviews a branch diff, a pull request, or a whole repo or path.
---

# $commentdiet

A code review with one axis: comments. Every comment line is read, and paid
for in context, by every future reader, human or agent. A comment keeps its
place only when the code cannot say the same thing. The rules live in
[`rules.md`](rules.md); this file is the process.

## Invocation

| Invocation | Scope |
| --- | --- |
| `$commentdiet` | Branch diff since the merge-base with the default branch |
| `$commentdiet <ref>` | Branch diff since `<ref>` (commit, branch, tag) |
| `$commentdiet pr <number or URL>` | A pull request's diff |
| `$commentdiet repo` or `$commentdiet <path>` | Every code file in the repo or path |

Every scope reports and changes nothing. A last argument of `fix`
(`$commentdiet fix`, `$commentdiet src/ fix`) also applies the findings after
the report. A first argument of `pr` or `repo` names that scope. Any other
argument is a path when it exists on disk, otherwise a ref that must resolve
with `git rev-parse --verify <arg>^{commit}`.

## 1. Pin the scope

- **Branch diff**: the fixed point is `<ref>`, or the target of
  `git symbolic-ref --short refs/remotes/origin/HEAD`, or else whichever of
  `main` and `master` exists. Take `base=$(git merge-base <fixed-point> HEAD)`.
  Files: `git diff --name-only --diff-filter=d $base` plus untracked
  `git ls-files -o --exclude-standard`. Diff: `git diff $base`, which includes
  uncommitted work.
- **Pull request**: `gh pr view <arg> --json number,headRefOid` gives the
  number and head SHA; `git fetch origin pull/<number>/head` makes the head
  readable. Files: `gh pr diff <arg> --name-only`. Diff: `gh pr diff <arg>`.
  Read a whole file with `git show <head SHA>:<path>`.
- **Repo or path**: one `git ls-files <path>` listing, keeping code files,
  capped at 200. Name the folders beyond the cap so the user can rerun on
  each.

In the diff scopes, review the comments the diff adds or edits, plus existing
comments inside changed hunks that the change made stale. Untouched comments
elsewhere belong to a repo or path run.

Drop docs, config and data files, generated and vendored code, build output,
lockfiles, test fixtures and snapshots. Drop a file with `commentdiet: off` in
its first 5 lines unless the user named it.

Done when the scope is a non-empty file list. An empty diff or listing ends
the run with one line saying so.

## 2. Review in sub-agents

The review runs in sub-agents so the main context receives findings, never
file contents. Split the file list into batches of 25 files (fewer batches of
more files when that would exceed 12 sub-agents) and spawn one sub-agent per
batch, all in parallel. A single batch still goes to a sub-agent. Each prompt
carries:

- The absolute path of [`rules.md`](rules.md), to read in full.
- The batch's files and the step 1 commands to read their diff or contents.
- The brief: "Apply every rule in rules.md to every in-scope comment in
  these files. Return each finding as a header line
  `file:line | action | -N lines | reason`, where action is one from the
  Actions list in rules.md and N is the comment lines it removes, followed by
  the replacement code or comment text in a fenced block (no block for
  `delete` and `flag`). Return nothing for comments you keep."

Done when every batch has returned.

## 3. Verify

`rename`, `extract` and `fix-stale` findings are claims about code. Read the
cited line and what it depends on before reporting. For a rename,
`git grep -w -F <old name>` (add the head SHA for a pull request) confirms
every reference is in scope. In the diff scopes only symbols the diff
introduces or file-local symbols qualify. A rename outside that, or one whose
old name also appears in strings or dynamic lookups, becomes `flag`. Drop a
refuted finding with one line on why. Done when each such finding is
confirmed, downgraded to `flag`, or dropped.

## 4. Report

List findings by action in this order: `extract`, `rename`, `fix-stale`,
`delete`, `trim`, `flag`; within each, by file and line, in the step 2
format. End with one line: findings per action and total comment lines saved.
Above 40 findings, write the full list to a temp file and show per-action
counts, the 10 largest savings, and the file's path. Done when every verified
finding is listed and the totals match it.

For a pull request, post to it only when the user says so: one review whose
inline comments carry GitHub suggestion blocks.

Without `fix`, the run ends here.

## 5. Fix

Fix needs the reviewed code checked out on a branch the user owns; for a
pull request, `git rev-parse HEAD` must equal its head SHA. Otherwise report
only. Start from a working tree whose other changes are committed so the fix
is one reviewable diff; if it is dirty, ask first.

Apply every reported finding except `flag`, then run the project's
typecheck, lint, and fast tests. Revert any change that breaks them: a
comment edit, rename or extraction preserves behavior. Done when the checks
pass and the diff holds only comment, rename and extraction changes. Report
what changed with the step 4 summary line.
