# Commit Style

## Goal

Commit history must allow a reader or AI to understand:

* What task or capability was completed.
* Why the change was needed.
* Which subsystem was affected.
* Whether behavior or compatibility changed.

Implementation details belong in the code and diff.

## Language

All commit messages must be written in English, regardless of the
spoken language used elsewhere in the project (issues, chat, docs).

## Format

```text
<type>[optional scope][!]: <summary>

[optional context and high-level outcome]

[optional footer]
```

## Types

| Type       | Use for                                          |
| ---------- | ------------------------------------------------- |
| `feat`     | New capability or behavior                       |
| `fix`      | Bug or incorrect behavior                        |
| `security` | Fix for a vulnerability or security weakness     |
| `refactor` | Structure change without behavior change         |
| `perf`     | Performance improvement                          |
| `test`     | Test additions or corrections                    |
| `docs`     | Documentation only                               |
| `build`    | Build system, compiler, packaging, or dependency changes made by hand |
| `ci`       | CI/CD configuration                              |
| `style`    | Formatting only; no logic change                 |
| `chore`    | Repository maintenance or supporting tasks        |
| `revert`   | Revert a previous commit                          |

Notes:

* Use `security` instead of `fix` when the change addresses a
  vulnerability, so security-relevant commits can be audited separately.
* Use `chore(deps)` for automated dependency bumps (e.g. Dependabot,
  Renovate) and `build(deps)` for manually upgraded dependencies with
  migration work involved. Keep the distinction consistent.

## Subject

* Use imperative mood: `add`, `fix`, `remove`, `prevent`.
* Use lowercase after the colon.
* Do not end with a period.
* Describe one concrete outcome.
* Keep the full subject line (type + scope + summary) to 72 characters
  or fewer, ideally around 50, so it displays cleanly in `git log
  --oneline` and in Git hosting UIs.
* Avoid vague summaries such as `update code`, `fix issue`, `changes`,
  or `work in progress`.

```text
feat(parser): add validation for malformed headers
fix(memory): prevent release before async completion
refactor(config): separate parsing from validation
security(auth): reject expired tokens on refresh
test(parser): cover malformed header edge cases
style(lint): apply formatter to touched files
```

## Scope

Use a stable subsystem, module, or responsibility — not a temporary
task name, ticket number, or arbitrary file name.

```text
feat(transport): ...
fix(memory): ...
docs(architecture): ...
build(toolchain): ...
```

For multi-package or monorepo projects, use the package or app name as
the scope (e.g. `feat(web): ...`, `fix(api): ...`), and keep the same
name for that package across its lifetime.

## Body

Use a body for non-trivial commits.

Explain:

* Why the change was necessary.
* What high-level task or capability is now complete.
* Important behavior, constraints, or impact.

Do not describe implementation line by line or list every modified file.

```text
feat(transport): add asynchronous request delivery

Support platforms where synchronous communication is unavailable.
Covers request submission, completion, and error propagation.
```

The commit explains the completed work and motivation. The code
explains how it was implemented.

## Atomic Commits

* One commit represents one logical change.
* Do not mix features, refactoring, formatting, and unrelated fixes.
* Each commit should build and remain reviewable whenever practical.
* Split changes when they can be understood or reverted independently.

## Breaking Changes

Add `!` after the type or scope and describe the impact in the footer:

```text
feat(api)!: make request submission asynchronous

Allow processing to continue without blocking the caller.

BREAKING CHANGE: callers must keep requests valid until completion.
```

## Reverts

When reverting a commit, keep the original subject for traceability
and explain why in the body:

```text
revert: feat(transport): add asynchronous request delivery

Reverts the change introduced in <short-sha>. Caused intermittent
timeouts under load; will be reintroduced after the root cause is
fixed.

Refs: #789
```

## References

Issue or task references are optional:

```text
Refs: #123
Closes: #456
```

The message must remain understandable without opening the referenced
issue.

## Examples

```text
chore(repo): add initial project structure

Define the initial source, include, test, and documentation layout.
```

```text
feat(config): add runtime configuration loading

Allow deployments to change supported options without rebuilding.
Covers validation, defaults, and error reporting.
```

```text
fix(queue): prevent lost wakeups during shutdown

A worker could sleep after shutdown was requested and never exit.
Ensure the shutdown state is visible before workers wait.
```

```text
security(auth): invalidate sessions on password change

Prevent a stolen session token from remaining valid after the
account owner changes their password.
```

```text
refactor(parser): separate validation from decoding

Preserve existing behavior while allowing malformed inputs to be
tested independently.
```

```text
build(toolchain): add cross-compilation configuration

Provide reproducible builds for the supported target architecture.
```

```text
chore(deps): bump lodash from 4.17.20 to 4.17.21
```

```text
docs(architecture): document module boundaries and open decisions
```

## Enforcement (optional)

If the project uses commit linting (e.g. commitlint, husky), configure
it to validate type, scope, and subject length automatically instead
of relying on manual review.

## Final Check

Before committing, verify that the message answers:

1. What was completed?
2. Why was it needed?
3. Which part of the project changed?
4. Did behavior or compatibility change?

A reader should inspect the code only to learn **how** it was
implemented.

## AI Attribution

* Do not add `Co-Authored-By`, `Generated with`, `Claude-Session`, or
  any AI attribution to commits or pull requests.
* Commit messages must contain only information about the project
  change.
