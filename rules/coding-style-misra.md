# Coding Style — MISRA C Oriented

Reference: [MISRA C — Guidelines for the use of the C language in critical systems](https://misra.org.uk/misra-c/)
(published by The MISRA Consortium — official current edition is **MISRA C:2023**;
the guideline document itself is a licensed publication, not free text,
and must be purchased/accessed from [misra.org.uk](https://misra.org.uk/) for
the authoritative rule numbers, rationale, and examples)

This file is a project-level summary aligned with MISRA C intent. It is
**not** a substitute for the official MISRA C document, and it does not
by itself make a project "MISRA compliant" — compliance requires a
static analysis tool that checks the numbered rules/directives and a
documented deviation process for any rule not followed.

## General

* Follow the existing project style first.
* Do not reformat unrelated code.
* Prefer simple, readable, deterministic, and analyzable code over
  clever solutions — analyzability by tooling is a primary goal.
* Preserve the current C standard, compiler flags, and platform constraints.
* Every rule/directive that is not followed must be recorded as a
  documented, justified deviation (per the project's deviation process).

## Indentation and Layout

* Use tabs for indentation, tab width 8 (or spaces, per project — MISRA
  itself is silent on whitespace; follow project convention consistently).
* Prefer lines within 80 columns.
* Avoid more than 3 nesting levels; use helper functions to flatten
  control flow (see "Control Flow" below for the single-exit constraint).
* No trailing whitespace.
* Separate functions with one blank line.

## Braces

* Every `if`, `else`, `for`, `while`, and `do` body must use braces,
  even for a single statement (MISRA Rule 15.6) — this removes ambiguity
  and dangling-else risk entirely.

```c
if (condition) {
	do_this();
} else {
	do_other();
}

for (i = 0U; i < count; i++) {
	update(i);
}

int process(void)
{
	return 0;
}
```

## Control Flow

* **Single point of entry and exit per function** (MISRA Rule 15.5,
  advisory): prefer one `return` at the end of the function rather than
  multiple early returns. Use a local status variable and fall through
  to a single return statement.
* **Do not use `goto`** (MISRA Rule 15.1, advisory). Structure cleanup
  using nested `if`, a status variable, or a dedicated cleanup function
  instead of jump-based cleanup.
* **Do not use recursion** (MISRA Rule 17.2, required) — stack depth
  must be statically boundable.
* Every `switch` must have a `default` clause (MISRA Rule 16.4,
  required), and fallthrough between cases must be explicit and
  justified, never implicit.
* A `switch` clause must end in `break`, `return`, `goto`, or
  `throw`/equivalent — never fall through silently (MISRA Rule 16.3, required).

```c
int result;

result = ERROR_INVALID_ARGUMENT;

if (object != NULL) {
	buffer = buffer_allocate_static();
	if (buffer != NULL) {
		result = object_initialize(object, buffer);
	} else {
		result = ERROR_NO_MEMORY;
	}
}

return result;
```

## Naming

* Use `snake_case` for functions, variables, fields, and files.
* Use `UPPER_CASE` for macros, constants, and enum values.
* Public and file-scope names must be descriptive.
* Do not use Hungarian notation.
* Use verbs for functions and `is_`, `has_`, or `can_` for predicates.
* Identifiers must be distinct within the significant character limits
  of the target compiler/linker (MISRA Rule 5.x series, required).

## Types

* Use the Essential Type Model: avoid implicit conversions between
  essentially different types (signed/unsigned/char/enum/float/bool)
  (MISRA Rule 10.x, required). Make all conversions explicit with a cast.
* Use fixed-width integer types (`uint32_t`, `int16_t`, etc.) rather
  than plain `int`/`long` when the width matters (MISRA Dir 4.6, advisory).
* Use `size_t` for object sizes and array indexes.
* Use `bool` (`<stdbool.h>`) for logical state, and only use `bool`
  values in logical contexts (MISRA Rule 10.1/10.5).
* Use `const` for read-only data.
* One declaration per line; initialize every variable at the point of
  declaration (MISRA Rule 9.1, mandatory — no use of an uninitialized
  automatic variable).
* Do not use `union` unless justified and documented (MISRA Rule 19.2, advisory).

```c
uint32_t register_value = 0U;
size_t buffer_size = 0U;
const uint8_t *data = NULL;
bool is_ready = false;
```

## Functions

* One function should perform one clear task.
* Keep functions short; a single entry, single exit function is easier
  to keep short by construction.
* Use `static` for symbols private to one source file.
* Validate all inputs at every function boundary, including internal ones.
* Do not add unused parameters, variables, functions, or includes
  (MISRA Rule 2.x, advisory/required — dead code is a defect class).
* Function prototypes must be visible and identical at every call site
  (MISRA Rule 8.x, required).

## Error Handling

* Report failures explicitly through return values or output parameters.
* Every return value that indicates an error must be checked
  (MISRA Dir 4.7, required).
* No early returns mid-function; propagate the error through the
  single-exit status variable pattern shown above.
* No dynamic control transfer (`goto`, `setjmp`/`longjmp`) for error handling.

## Memory

* **No dynamic memory allocation** (`malloc`, `free`, `calloc`,
  `realloc`) in the core logic (MISRA Rule 21.3, required). Use static
  or stack allocation with bounded, known-at-compile-time sizes.
* If dynamic allocation is unavoidable in a specific subsystem (e.g. a
  bootloader stage or host tool outside the safety-relevant scope),
  isolate it clearly and document it as a deviation.
* Make ownership and lifetime explicit even for static/stack objects.

## Macros and Constants

* Prefer constants, enums, and `static inline` functions over
  function-like macros (MISRA Dir 4.9, advisory).
* If a function-like macro is unavoidable, parenthesize every argument
  and the whole expression, and wrap multi-statement macros in
  `do { } while (0)`.
* Do not use macros that hide control flow or evaluate arguments more
  than once.

```c
#define ARRAY_COUNT(array) \
	(sizeof(array) / sizeof((array)[0]))
```

## Restricted / Disallowed Standard Library Functions

MISRA restricts or forbids several standard library functions because
their behavior is unsafe, unbounded, or implementation-defined. At minimum:

* Do not use `gets` (removed from the standard; always unsafe).
* Do not use unbounded string functions (`strcpy`, `strcat`, `sprintf`)
  without a bounded alternative (`strncpy`/manual bounds, `snprintf`)
  (MISRA Rule 21.x, required).
* Do not use `atoi`, `atol`, `atof` (no error signaling on invalid input).
* Do not use `rand()` for anything security- or safety-relevant.
* Consult the official MISRA C document (Rule 21 series) for the full
  list and required alternatives before using any `<stdlib.h>`,
  `<string.h>`, or `<stdio.h>` function.

## Concurrency and Hardware

* Do not use `volatile` as a replacement for synchronization.
* Use locks, atomics, critical sections, interrupt masking, or memory
  barriers as required; MISRA C:2012 Amendment 4 adds specific guidance
  for multi-threading and atomics — consult it directly for threaded code.
* Avoid blocking operations in interrupt context.
* Use platform accessors for MMIO when available.
* Use named masks and constants instead of unexplained values.

## Headers

* Expose only public types, constants, declarations, and small inline helpers.
* Use include guards.
* Minimize header dependencies; each header must be self-contained
  (MISRA Rule 20.x, required).
* Do not define writable globals or ordinary non-static functions in headers.

## Generated and External Code

* Do not manually edit generated files.
* Do not reformat third-party code only to match this guide.
* Third-party code not written to MISRA is typically excluded from
  compliance scope but must be isolated and documented as such.

## Tooling and Compliance Process

* MISRA compliance in practice requires a static analysis tool (e.g. a
  commercial MISRA checker) configured for the target MISRA C edition;
  this document does not replace that tooling.
* Classify every guideline you deviate from as an intentional,
  reviewed, and documented deviation, per **MISRA Compliance:2020**
  (the companion document defining the compliance process).
* Guidelines are classified as **Mandatory** (no deviation permitted),
  **Required** (deviation permitted only with formal justification), or
  **Advisory** (deviation permitted with lighter-weight justification) —
  treat mandatory items in the official document as non-negotiable.
