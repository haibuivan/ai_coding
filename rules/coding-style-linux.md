# Coding Style — Linux Kernel Style

Reference: [Linux kernel coding style](https://docs.kernel.org/process/coding-style.html)
(`Documentation/process/coding-style.rst` in the kernel source tree)

This file summarizes the conventions for project use. When in doubt,
the reference document above is authoritative.

## General

* Follow the existing project style first.
* Do not reformat unrelated code.
* Prefer simple, readable, deterministic code over clever solutions.
* Preserve the current C standard, compiler flags, and platform constraints.

## Indentation and Layout

* Use tabs for indentation, tab width 8.
* Prefer lines within 80 columns.
* Avoid more than 3 nesting levels; use early returns or helper functions.
* No trailing whitespace.
* Separate functions with one blank line.

```c
switch (action) {
case ACTION_START:
	return start();

case ACTION_STOP:
	return stop();

default:
	return ERROR_INVALID_ACTION;
}
```

## Braces

```c
if (condition) {
	do_this();
	do_that();
} else {
	do_other();
}

int process(void)
{
	return 0;
}

if (!object)
	return ERROR_INVALID_ARGUMENT;
```

* Control statement braces stay on the same line.
* Function opening braces go on the next line.
* Use braces on both branches when either branch has multiple statements.

## Spaces

```c
if (ready)
	process();

for (i = 0; i < count; i++)
	update(i);

sizeof(struct object)

char *buffer;
const struct object *object;

result = left + right;
value = condition ? first : second;

&value;
*pointer;
!ready;
object->member;
```

## Naming

* Use `snake_case` for functions, variables, fields, and files.
* Use `UPPER_CASE` for macros, constants, and enum values.
* Public and file-scope names must be descriptive.
* Short local names such as `i`, `ret`, and `tmp` are acceptable when obvious.
* Do not use Hungarian notation.
* Use verbs for functions and `is_`, `has_`, or `can_` for predicates.

```c
bool request_is_ready(const struct request *request);
int request_submit(struct request *request);
```

## Types

* Use fixed-width integers when width is part of hardware, protocol, storage, or ABI contracts.
* Use `size_t` for object sizes and array indexes.
* Use `bool` for logical state.
* Use `const` for read-only data.
* Prefer one declaration per line.
* Avoid typedefs for structs and pointers unless the type is opaque or ABI-defined.

```c
uint32_t register_value;
size_t buffer_size;
const uint8_t *data;
struct request *request;
```

## Comments

* Comments should explain **why**, not **what**; the code already says what.
* Avoid commenting inside a function body; if a function needs that,
  it is probably too complex and should be split.
* Use kernel-doc format for public API function comments.
* Preferred multi-line comment style:

```c
/*
 * This is the preferred style for multi-line
 * comments in the kernel-style source code.
 * Please use it consistently.
 *
 * Description: a column of asterisks on the left side,
 * with beginning and ending almost-blank lines.
 */
```

## Functions

* One function should perform one clear task.
* Keep functions short and control flow shallow.
* Use `static` for symbols private to one source file.
* Validate inputs at public module boundaries.
* Do not add unused parameters, variables, functions, or includes.
* Do not use `extern` on normal function declarations.

## Error Handling

* Report failures explicitly through return values or output parameters.
* Do not ignore return values.
* Use early returns when no cleanup is needed.
* Use `goto` for shared cleanup paths (centralized exiting).
* Cleanup labels must describe the action.
* Release resources in reverse acquisition order.

```c
buffer = buffer_allocate();
if (!buffer)
	return ERROR_NO_MEMORY;

result = object_initialize();
if (result != 0)
	goto out_free_buffer;

return 0;

out_free_buffer:
	buffer_free(buffer);
	return result;
```

## Macros and Constants

* Prefer constants, enums, and `static inline` functions over function-like macros.
* Parenthesize macro arguments and complete expressions.
* Wrap multi-statement macros in `do { } while (0)`.
* Do not use macros that hide control flow or evaluate arguments more than once.

```c
#define ARRAY_COUNT(array) \
	(sizeof(array) / sizeof((array)[0]))

#define OBJECT_RESET(object)		\
	do {				\
		(object)->state = 0U;	\
		(object)->flags = 0U;	\
	} while (0)
```

## Memory

* Do not introduce dynamic allocation unless the project permits it.
* Allocate using the pointed-to object's size.
* Do not cast allocation results in C.
* Check for allocation failure and size overflow.
* Make ownership and lifetime explicit.

```c
object = malloc(sizeof(*object));
if (!object)
	return NULL;
```

## Concurrency and Hardware

* Do not use `volatile` as a replacement for synchronization.
* Use locks, atomics, critical sections, interrupt masking, or memory barriers as required.
* Avoid blocking operations in interrupt context.
* Use platform accessors for MMIO when available.
* Preserve required register ordering, delays, barriers, and access widths.
* Use named masks and constants instead of unexplained values.

## Headers

* Expose only public types, constants, declarations, and small inline helpers.
* Use include guards.
* Minimize header dependencies.
* Use forward declarations when the complete type is unnecessary.
* Do not define writable globals or ordinary non-static functions in headers.

## Generated and External Code

* Do not manually edit generated files.
* Do not reformat third-party code only to match this guide.
* Keep compatibility changes isolated.

## Tooling

* `clang-format` can be used to auto-format and spot style issues; see
  `Documentation/dev-tools/clang-format.rst` in the kernel tree for the
  kernel's own configuration approach.
* An `.editorconfig`-compatible editor will pick up basic indentation
  settings automatically — see [editorconfig.org](https://editorconfig.org/).
