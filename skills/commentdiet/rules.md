# commentdiet rules

The code is the primary comment. Read each comment, then the code it
describes, and take the first section below that fits.

## 1. Delete: the comment adds nothing

No evidence needed beyond the comment itself:

- Narration that restates the next line (`// increment i`, `// return the user`).
- Change history: "updated", "fixed", "changed to", "previously", "now uses",
  dates, author tags. Commits hold history.
- Commented-out code. Git keeps it.
- Section banners and dividers that a function or file boundary already shows.
- Type information the type system already states (`@param {string}` in
  TypeScript, types repeated beside Python annotations).
- Empty or placeholder doc comments (`/** */`, `@param x - x`, a summary
  that only repeats the function's name).

## 2. Make the code say it: rename or extract

The comment explains what the code does or means. Change the code so the
comment becomes narration, then delete it.

- `rename`: a comment explaining what a name holds or does. `t // seconds`
  becomes `timeoutSeconds`; `// true when the user is an admin` above `flag`
  becomes `isAdmin`.
- `extract`: a magic value becomes a named constant; a dense expression
  becomes a named variable or small function; a commented block inside a
  long function becomes a function named after the comment.

The change is local and preserves behavior. Rename a symbol only when every
reference is in scope. A clarity fix that needs a real refactor or a public
API change keeps its comment and becomes `flag`.

## 3. Fix the stale: verify first

- Describes behavior the code no longer has: rewrite it to match, or delete it.
- Names a symbol, file or flag that no longer exists (check with one exact
  `rg -F`): fix the name, or delete the comment.
- A suppression (`eslint-disable`, `@ts-expect-error`, `# noqa`,
  `# type: ignore`) that suppresses nothing: delete it. One that is still
  needed but gives no reason: add the reason.
- TODO, FIXME or HACK without an issue link: `flag` it. Link an issue only
  when the user gives one.

## 4. Trim: the why stays, the words go

A comment that earns its place still costs every reader. Cut it to what the
code cannot carry:

- One line where possible. Link the issue, spec or doc in place of
  explaining it.
- Public API doc comments keep only meaning beyond the signature: units,
  ranges, errors raised, side effects, invariants.
- Present tense, describing the code as it is now.

## 5. Keep

- Why-comments: intent, constraints, invariants, workarounds with their
  reason, security and performance notes, links to specs or issues.
- Toolchain directives (`// @ts-check`, `"use client"`, `/* @jsx */`,
  suppressions still needed) and legal headers.

When unsure whether a comment says why or what, keep it.

## Actions

Each finding carries one action: `delete`, `rename`, `extract`, `fix-stale`,
`trim`, or `flag`. `flag` marks a spot that needs a human: an unlinked TODO,
or a clarity fix that needs a real refactor.
