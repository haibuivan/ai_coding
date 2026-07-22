# Comment Style

## Scope

This document defines a Linux-inspired comment style for this project.

It follows Linux kernel conventions where applicable, while adding several project-specific rules such as `CRITICAL:`.

Project-specific rules must not be presented as upstream Linux kernel requirements.

## Core Rule

Prefer self-explanatory code over comments.

Comments should explain intent, purpose, rationale, constraints, assumptions, or risks.

Do not use comments to restate implementation mechanics that are already clear from the code.

Before keeping a comment, ask:

> If this comment is removed, is important intent, rationale, constraint, assumption, or risk lost?

If not, remove it.

This project commonly describes this principle as **WHY, not WHAT**, but the Linux kernel convention is more precisely:

> Explain what the code is intended to achieve and why it is written that way, not how each statement works.

## Write Comments For

Write comments when they preserve information that cannot be expressed clearly through names, types, functions, or code structure.

Appropriate cases include:

* Non-obvious hardware or platform constraints.
* Concurrency, interrupt, locking, or timing requirements.
* Assumptions and invariants.
* Ownership and lifetime constraints.
* Hidden side effects or unusual edge cases.
* Magic values whose origin cannot be expressed in code.
* Required sequences from specifications or hardware manuals.
* Workarounds for hardware, compiler, operating system, or external API behavior.
* Performance trade-offs that make the code less obvious than a simpler alternative.
* Reasons a check, lock, or error-handling step was intentionally omitted.
* How conflicting constraints such as performance, correctness, safety, and latency were balanced.

State the consequence of violating a constraint when that consequence is important.

## Good Examples

### Hardware Constraint

```c
/*
 * Wait for TX idle before changing the clock; reconfiguration during
 * transmission may drop the final byte.
 */
while (!uart_tx_idle()) {
}
```

### Concurrency

```c
/* The IRQ handler updates head concurrently; hold rx_lock while reading it. */
spin_lock(&rx_lock);
```

### Interrupt Context

```c
/* CRITICAL: Called from IRQ context; blocking here can deadlock the system. */
process_rx_data();
```

`CRITICAL:` is a project-specific convention. It is not an upstream Linux kernel requirement.

Use it only when violating the documented constraint may cause:

* A crash.
* A race condition.
* A deadlock.
* Data loss.
* A security failure.
* Hardware damage.

### Required Sequence

```c
/* Hardware ignores configuration writes until RESET_DONE is asserted. */
reset_device();
wait_for_reset_done();
configure_device();
```

### Ring-Buffer Invariant

```c
/* Keep one slot empty so head == tail always represents an empty ring. */
#define RX_CAPACITY (RX_RING_SIZE - 1U)
```

### Memory Ownership

```c
/* The device retains this buffer until completion; the caller must not free it. */
submit_dma_request(buffer);
```

### Magic Value

```c
/* Reserve 2 MiB after the firmware image for alignment and future growth. */
#define FRAMEBUFFER_BASE 0x80800000U
```

### Hardware or Compiler Workaround

```c
/* Prevent CPU reordering before notifying the device that descriptors are ready. */
memory_barrier();
write_doorbell();
```

### Assumption

```c
/* Assumption: DMA addresses are 40-bit on this target; verify before hardware use. */
```

### Unresolved Dependency

```c
/*
 * TBD: Confirm whether the platform provides batch IOMMU mapping;
 * no equivalent was found in the current public headers.
 */
```

A `TBD:` or `Assumption:` comment must include enough context to verify or resolve it.

Remove it once the uncertainty is resolved.

### Performance Trade-Off

```c
/*
 * Linear scan is faster than a hash lookup here: the table has at most
 * eight entries and fits in one cache line.
 */
for (i = 0; i < table_size; i++) {
}
```

```c
/* Pre-allocated at init to avoid allocation in the interrupt-driven hot path. */
static uint8_t rx_scratch[RX_SCRATCH_SIZE];
```

### Intentional Omission

```c
/*
 * No lock is needed because this runs only during single-threaded init,
 * before the scheduler starts.
 */
counter = 0;
```

```c
/*
 * Failure only affects logging and must not block the shutdown sequence.
 */
(void)flush_log_buffer();
```

```c
/*
 * Bounds checking is handled by validate_request() before this function
 * is called.
 */
entry = table[index];
```

### Conflicting Constraints

```c
/*
 * Copy small transfers because the caller may release the source buffer
 * before DMA completes. Larger transfers remain zero-copy to avoid the
 * latency and memory cost of copying.
 */
if (size <= SMALL_TRANSFER_THRESHOLD) {
	memcpy(scratch, src, size);
}
```

```c
/*
 * Poll instead of waiting for an interrupt because the transfer completes
 * in under 2 us, below the interrupt-latency budget.
 */
while (!transfer_done()) {
}
```

## Do Not Write

Do not write:

* Comments that translate code into English.
* Comments for obvious assignments, loops, conditions, or function calls.
* Comments that repeat variable or function names.
* Decorative banners or separators.
* Author, date, or history sections.
* Phase or step markers.
* Commit, issue, or pull-request references.
* Documentation for trivial private functions.
* `TODO`, `TBD`, or assumption comments without enough context to resolve them.
* Comments that become false when implementation details change.
* Comments that explain information better expressed through a function, constant, type, or variable name.

## Bad Examples

```c
i++; /* Increment i. */
```

```c
/* Initialize the UART. */
uart_init();
```

```c
/* Check whether the buffer is empty. */
if (buffer_is_empty(buffer)) {
}
```

```c
/* Write value to control register. */
mmio_write32(CONTROL_REG, value);
```

```c
/* Phase 1: setup. */
/* TODO: Improve later. */
/* Fix for issue #42. */
```

```c
/* Skip the check. */
entry = table[index];
```

The last example does not explain why omitting the check is safe.

A useful replacement would be:

```c
/*
 * Bounds checking is handled by validate_request() before this function
 * is called.
 */
entry = table[index];
```

## C Comment Format

Use `/* ... */` for normal C comments.

### Single-Line Comment

```c
/* The register must be read twice to clear the latched fault state. */
```

Do not normally use:

```c
// The register must be read twice.
```

Although the compiler accepts C++-style comments, normal Linux kernel C comments conventionally use `/* ... */`.

### Multi-Line Comment

Use a leading `*` column:

```c
/*
 * The device may still access the descriptor after completion is reported,
 * so defer freeing it until the next synchronization point.
 */
```

Do not use:

```c
/* The device may still access the descriptor after completion is reported,
   so defer freeing it until the next synchronization point. */
```

### SPDX Exception

For Linux kernel source files, the SPDX line follows the syntax required by the kernel licensing rules.

A C source file commonly uses:

```c
// SPDX-License-Identifier: GPL-2.0
```

A header file commonly uses:

```c
/* SPDX-License-Identifier: GPL-2.0 */
```

Do not copy `GPL-2.0` blindly.

The SPDX expression must match the actual license of the file.

## Public API Documentation

Use kernel-doc for exported functions, public APIs, public structures, and interfaces with non-obvious contracts.

Document information such as:

* Preconditions.
* Required locks.
* Execution context.
* Whether the function may sleep.
* Ownership and lifetime.
* Hidden side effects.
* Error behavior.
* Edge cases.
* Return values.

Do not repeat the function signature or describe trivial parameters without adding useful contract information.

### Function Example

```c
/**
 * queue_submit() - Submit a request for asynchronous processing.
 * @request: Request that remains owned by the caller.
 *
 * The request memory must remain valid until the completion callback runs.
 *
 * Context: Process context. May sleep.
 * Return: 0 on success or -EBUSY when the queue cannot accept more requests.
 */
int queue_submit(struct request *request);
```

Use `Return:` as a separate kernel-doc section.

Do not write the return contract only as ordinary prose:

```c
/**
 * queue_submit() - Submit a request.
 * @request: Request to submit.
 *
 * Returns -EBUSY when the queue is full.
 */
```

### Structure Example

```c
/**
 * struct dma_request - Request submitted to the DMA engine.
 * @buffer: Buffer that remains valid until completion.
 * @length: Number of bytes available in @buffer.
 * @complete: Callback invoked after the device releases the buffer.
 *
 * Instances may be accessed from interrupt context after submission.
 */
struct dma_request {
	void *buffer;
	size_t length;
	void (*complete)(struct dma_request *request);
};
```

## File Header

Linux kernel files must begin with the appropriate SPDX identifier.

A project file header may then include:

* The relative path.
* The module purpose.
* An important module-wide constraint.

The relative path is a project convention, not a universal upstream Linux kernel requirement.

Example:

```c
// SPDX-License-Identifier: GPL-2.0
/*
 * drivers/gpu/power.c - GPU power management
 *
 * Power transitions must follow the platform-defined clock and reset sequence.
 */
```

Keep file headers short.

Do not include:

* Author names.
* Creation dates.
* Modification history.
* Commit references.
* Decorative separators.
* Information already available from version control.

Omit the descriptive file header entirely when it adds no useful module-wide information.

The SPDX identifier must still be present when required.

## Test Code Comments

Test code follows the same intent-over-mechanics rule.

Test setup is often self-explanatory, but the reason a test exists may not be obvious.

Write comments when they explain:

* A real-world regression.
* A hardware erratum.
* A boundary condition.
* A previously observed race.
* A non-obvious input combination.
* Why a result that appears unusual is correct.

### Good

```c
/*
 * Regression test for the off-by-one in ring_buffer_free_space() that
 * allowed one write to overwrite unread data.
 */
TEST(ring_buffer, full_when_capacity_reached)
{
}
```

### Bad

```c
/* Check that the function returns true. */
ASSERT_TRUE(result);
```

The assertion already shows what is being checked.

## Style

* Keep comments short and precise.
* Write comments in English.
* Place comments next to the code they explain.
* Prefer one complete comment over several fragmented comments.
* Use complete sentences when practical.
* State consequences when violating a constraint may cause a serious failure.
* Keep terminology consistent with the code and relevant specification.
* Remove or update comments when the code changes.
* Remove resolved `TBD:` and `Assumption:` comments.
* Avoid references that become stale unless the exact revision is necessary.
* Adapt the comment syntax to the programming language while preserving the intent-over-mechanics principle.

## Review Checklist

Before approving a comment, confirm:

1. If the comment is removed, is important intent, rationale, constraint, assumption, or risk lost?
2. Does it explain purpose or reasoning instead of translating the implementation?
3. Could the same information be expressed more clearly through names, types, constants, or function structure?
4. Is it placed next to the code it explains?
5. Does it state the consequence of violating an important constraint?
6. If it describes ownership, locking, context, timing, or lifetime, is the contract complete?
7. If it uses `CRITICAL:`, is the risk severe enough to justify it?
8. If it contains `TBD:` or `Assumption:`, is there enough information to resolve it?
9. If it references a specification, hardware revision, or external behavior, is the reference still accurate?
10. Does kernel-doc use the correct function, parameter, context, and `Return:` format?
11. Does the file begin with the correct SPDX identifier when required?
12. Is the comment still accurate after the current code change?
