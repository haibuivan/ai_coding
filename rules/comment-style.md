# Comment Style

## Core Rule

Prefer self-explanatory code over comments.

Comments must explain **WHY**, not restate **WHAT** the code does.

Before keeping a comment, ask:

> If this comment is removed, is important rationale, constraint, or risk lost?

If not, remove it.

## Write Comments For

* Non-obvious hardware or platform constraints.
* Concurrency, interrupt, locking, or timing requirements.
* Assumptions and invariants.
* Ownership and lifetime constraints.
* Hidden side effects or unusual edge cases.
* Magic values whose origin cannot be expressed in code.
* Required sequences from specifications or hardware manuals.
* Workarounds for hardware, compiler, OS, or external API behavior.
* Performance trade-offs that make the code less obvious than a simpler alternative.
* Reasons a check, lock, or error-handling step was intentionally omitted.
* How two conflicting constraints (e.g. speed vs. correctness, safety vs. latency) were balanced.

State the consequence of violating a constraint when important.

## Good Examples

### Hardware constraint

```c
/* Wait for TX idle before changing the clock; reconfiguration during
 * transmission may drop the final byte. */
while (!uart_tx_idle()) {
}
```

### Concurrency

```c
/* The IRQ handler updates head concurrently; hold rx_lock while reading it. */
spin_lock(&rx_lock);
```

### Interrupt context

```c
/* CRITICAL: Called from IRQ context; blocking here can deadlock the system. */
process_rx_data();
```

### Required sequence

```c
/* Hardware ignores configuration writes until RESET_DONE is asserted. */
reset_device();
wait_for_reset_done();
configure_device();
```

### Ring-buffer invariant

```c
/* Keep one slot empty so head == tail always represents an empty ring. */
#define RX_CAPACITY (RX_RING_SIZE - 1U)
```

### Memory ownership

```c
/* The device retains this buffer until completion; the caller must not free it. */
submit_dma_request(buffer);
```

### Magic value

```c
/* Reserve 2 MiB after the firmware image for alignment and future growth. */
#define FRAMEBUFFER_BASE 0x80800000U
```

### Workaround

```c
/* Prevent CPU reordering before notifying the device that descriptors are ready. */
memory_barrier();
write_doorbell();
```

### Assumption

```c
/* Assumption: DMA addresses are 40-bit on this target; verify before hardware use. */
```

### Unresolved dependency

```c
/* TBD: Confirm whether the platform provides batch IOMMU mapping;
 * no equivalent was found in the current public headers. */
```

### Performance trade-off

```c
/* Linear scan is faster than a hash lookup here: table has <= 8 entries
 * and fits in one cache line. */
for (i = 0; i < table_size; i++) {
}
```

```c
/* Pre-allocated at init to avoid malloc() in the interrupt-driven hot path. */
static uint8_t rx_scratch[RX_SCRATCH_SIZE];
```

### Intentional omission (negative-space reasoning)

```c
/* No lock needed: this runs only during single-threaded init, before
 * the scheduler starts. */
counter = 0;
```

```c
/* Return value intentionally ignored: failure here only affects logging
 * and must not block the shutdown sequence. */
(void)flush_log_buffer();
```

```c
/* Bounds check omitted: caller guarantees index < ARRAY_SIZE via the
 * validate_request() call at the top of this function. */
entry = table[index];
```

### Conflicting constraints

```c
/* Copy instead of DMA-in-place: the source buffer may be freed by the
 * caller before the transfer completes, but a zero-copy path is required
 * for buffers > 4 KiB, so only small transfers take this slower path. */
if (size <= SMALL_TRANSFER_THRESHOLD) {
	memcpy(scratch, src, size);
}
```

```c
/* Polling instead of an interrupt-driven wait: at this clock rate the
 * transfer completes in under 2us, faster than the IRQ latency budget. */
while (!transfer_done()) {
}
```

## Do Not Write

* Comments that translate code into English.
* Comments for obvious assignments, loops, conditions, or function calls.
* Comments repeating variable or function names.
* Banners and separators.
* Author, date, or history sections.
* Phase or step markers.
* Commit, issue, or pull-request references.
* Documentation for trivial private functions.
* `TODO`, `TBD`, or assumptions without enough context to resolve them.

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
/* Phase 1: setup */
/* TODO: improve later */
/* Fix for issue #42 */
```

```c
/* Skip the check. */
if (ptr != NULL) {
}
```
*(Says nothing about* ***why*** *the check is safe to skip when it's absent — see "Intentional omission" above for the correct version.)*

## Public API Documentation

Document public APIs only when they have a non-obvious contract:

* Preconditions.
* Required lock or execution context.
* Ownership and lifetime.
* Hidden side effects.
* Error and edge-case behavior.

```c
/**
 * queue_submit - submit a request for asynchronous processing
 * @request: request that remains owned by the caller
 *
 * The request memory must remain valid until the completion callback runs.
 * Returns -EBUSY when the queue cannot accept more requests.
 */
int queue_submit(struct request *request);
```

Do not document trivial parameters or repeat the implementation.

## File Header

Keep file headers short:

```c
/*
 * drivers/gpu/power.c — GPU power management
 *
 * Power transitions must follow the platform-defined clock/reset sequence.
 */
```

Include only the relative path, module purpose, and an important module-wide constraint when necessary.

## Test Code Comments

Test code follows the same WHY-not-WHAT rule, but "obvious" means something different: a test's *setup* is usually self-explanatory, while the *reason a case exists* often is not.

* Explain what real-world bug, edge case, or regression a test protects against, especially if it isn't obvious from the test name or inputs.
* Do not restate the assertion in prose.

**Good:**

```c
/* Regression test for the off-by-one in ring_buffer_free_space() that
 * caused RX_CAPACITY+1 writes to silently overwrite unread data. */
TEST(ring_buffer, full_when_capacity_reached) {
}
```

**Bad:**

```c
/* Check that the function returns true. */
ASSERT_TRUE(result);
```

## Style

* Keep comments short and precise.
* Write comments in English.
* Place comments close to the relevant code.
* Prefer one useful comment over several fragmented comments.
* Use `CRITICAL:` only for severe risks such as crashes, races, deadlocks, data loss, security failures, or hardware damage.
* Remove or update comments when the code changes.
* Remove resolved `TBD:` and `Assumption:` comments.
* These rules are language-agnostic; adapt only the comment syntax (`//`, `#`, `"""`) to the language in use — the WHY-not-WHAT principle and categories above still apply.

## Review Checklist

Before approving a comment in review, confirm:

1. If this comment is removed, is rationale, constraint, or risk lost? If not, remove it.
2. Does it explain WHY, not WHAT?
3. Is it placed next to the code it explains, not in a separate banner or header block?
4. If it references a constraint that could become stale (a spec version, a hardware revision, a `TBD`/`Assumption`), is it still accurate as of this change?

