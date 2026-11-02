---
name: docdiet
description: Perform an on-demand, bounded documentation review and update while keeping canonical docs accurate and lean.
disable-model-invocation: true
---

# $docdiet

Use this skill only when the user explicitly requests a documentation review,
cleanup, or update (normally by invoking `$docdiet`). It is a standalone,
on-demand review/update skill, not an automatic per-task hook. Review or update
only the documentation scope named by the user or clearly exposed by the
requested change; do not start an unrelated audit.

An explicit request such as `$docdiet all docs` or `$docdiet project-wide`
authorizes a bounded project-wide pass. In that mode, scope is limited to
documentation files under the repository root with `.md`, `.mdx`, `.rst`, or
`.adoc` extensions, plus root-level `README*` and `CONTRIBUTING*` files.
Exclude generated or vendored directories and files, changelogs/release notes,
licenses or legal material, codes of conduct, and repository-protected files.
Discovery is limited to one `rg --files` listing using those extensions and
exclusions, capped at 200 paths; inspect only paths returned by that listing.
Use at most one additional exact, path-limited `rg -l -F` reference search for
a changed public concept. If the cap or search boundary is reached, report the
unreviewed remainder rather than expanding scope.

## Decision rules

1. **Gate scope first.** If no documentation file or section is named, no
   documentation is being touched, and no public rename/removal needs a
   reference check, stop. Do not inspect documentation to decide whether to
   inspect documentation.
2. **Edit the canonical source.** Document only durable information developers,
   users, operators, or maintainers need: contracts, setup, configuration,
   operations, constraints, invariants, non-obvious behavior, migrations, and
   critical warnings. Omit implementation trivia, temporary state, debugging
   history, inferred facts, and completion summaries.
3. **Replace, do not accumulate.** Update the existing explanation, remove
   obsolete text, and link to one canonical source when another document needs
   a pointer. Keep historical detail only for migration, compatibility, or
   architectural rationale.
4. **Keep cleanup local.** While editing a documentation file, clean only the
   touched section and nearby text that has clear evidence of being stale,
   duplicated, contradictory, superseded, misleading, unnecessarily verbose,
   or tied to the changed/removed concept. If evidence requires substantial
   investigation, leave it alone.
5. **Bound reference searches.** For a renamed, removed, or changed public
   concept, at most one exact, path-limited search is allowed. Prefer
   `rg -l -F` with documentation globs and report filenames, not full matching
   lines. Outside the explicit project-wide mode above, do not inventory
   unrelated docs, browse documentation trees, run repo-wide link checks,
   investigate unrelated history, or create a cleanup agent.
6. **Create files rarely.** Add a documentation file only for durable,
   distinct information with no suitable canonical home already known from the
   task.

## Protection

Do not opportunistically rewrite generated docs, changelogs/release notes,
licenses or legal material, codes of conduct, vendored/third-party docs, or
repository-protected files. Respect repository instructions and preserve
meaningful frontmatter and metadata.

If a Markdown file contains:

```yaml
---
disable-agentic-editing: true
---
```

do not modify it unless the user explicitly requests that file. This marker is
a `docdiet` convention: no tool enforces it, repository instructions cannot
remove its protection, and only an explicit request for that file overrides it.
Check protection only for candidate files; do not scan the repository for
markers.

## Completion

If documentation was touched, statically check the touched file/section only:
the final behavior is documented where needed, obsolete text was removed only
with evidence, examples and commands still match the implementation, and
protected content was unchanged. In project-wide mode, apply these checks to
each touched file; do not broaden validation into a repository-wide link or
example check. Do not execute examples or commands unless the implementation
task already requires it. Then stop.

> Document what lasts. Replace rather than accumulate. Clean what the task
> exposes. Do not hunt.
