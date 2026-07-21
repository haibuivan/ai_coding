# Commit Style

## Goal

Commit history must allow a reader or AI to understand:

* What task or capability was completed.
* Why the change was needed.
* Which subsystem was affected.
* Whether behavior or compatibility changed.

Implementation details belong in the code and diff.

## Format

```text
<type>[optional scope][!]: <summary>

[optional context and high-level outcome]

[optional footer]
```

## Types

| Type       | Use for                                    |
| ---------- | ------------------------------------------ |
| `feat`     | New capability or behavior                 |
| `fix`      | Bug or incorrect behavior                  |
| `refactor` | Structure change without behavior change   |
| `perf`     | Performance improvement                    |
| `test`     | Test additions or corrections              |
| `docs`     | Documentation only                         |
| `build`    | Build system, compiler, or dependencies    |
| `ci`       | CI/CD configuration                        |
| `style`    | Formatting only; no logic change           |
| `chore`    | Repository maintenance or supporting tasks |
| `revert`   | Revert a previous commit                   |

## Subject

* Use imperative mood: `add`, `fix`, `remove`, `prevent`.
* Use lowercase after the colon.
* Do not end with a period.
* Describe one concrete outcome.
* Keep it concise and understandable without opening the diff.
* Avoid vague summaries such as `update code`, `fix issue`, `changes`, or `work in progress`.

```text
feat(parser): add validation for malformed headers
fix(memory): prevent release before async completion
refactor(config): separate parsing from validation
```

## Scope

Use a stable subsystem, module, or responsibility:

```text
feat(transport): ...
fix(memory): ...
docs(architecture): ...
build(toolchain): ...
```

Avoid scopes based on temporary task names or arbitrary file names.

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

The commit explains the completed work and motivation. The code explains how it was implemented.

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

## References

Issue or task references are optional:

```text
Refs: #123
Closes: #456
```

The message must remain understandable without opening the referenced issue.

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
refactor(parser): separate validation from decoding

Preserve existing behavior while allowing malformed inputs to be
tested independently.
```

```text
build(toolchain): add cross-compilation configuration

Provide reproducible builds for the supported target architecture.
```

```text
docs(architecture): document module boundaries and open decisions
```

## Final Check

Before committing, verify that the message answers:

1. What was completed?
2. Why was it needed?
3. Which part of the project changed?
4. Did behavior or compatibility change?

A reader should inspect the code only to learn **how** it was implemented.

## AI Attribution

- Do not add `Co-Authored-By`, `Generated with`, `Claude-Session`, or any AI attribution to commits or pull requests.
- Commit messages must contain only information about the project change.
