<!-- published at /help/guide/troubleshooting.html on the Drive host -->

# Troubleshooting and Tuning

Job parameters are tolerances, and they interact. The guidance here maps common symptoms to the knobs worth considering — as options with tradeoffs, not guaranteed fixes. Change one thing at a time, and prefer cheap experiments: a [cloned project](./projects-page.md#per-row-actions), or a bounded [compute range](./setup/run-job.md#compute-range).

## Symptoms

### The run rejects surveys as impossible

Drive smooths gamma onto the MD sampling grid, so across a sharp hot/cool gamma contrast a tight **Log Tolerance** can make a survey look inexplicable. Consider a finer **MD Interval**, or a wider **Log Tolerance**.

### Stair-step artifacts in the structure

The dip freedom is too small for the grid it is working on. Loosen one of: **Dip Tolerance**, **MD Interval**, or **Depth Resolution**.

### Gamma misfit grows on heterogeneous geology

What looks like noise is often real divergence between the pilot's gamma character and the active well's. Consider raising **Log Tolerance** (e.g. toward 15–20 gAPI-equivalent) rather than blaming the tool — but first rule out an actual tool-malfunction interval, which should be an [ignore range](./setup/active-well.md#ignore-lwd-ranges), not a tolerance change.

### The structure drifts steadily away from the dip prior

Two readings: either the prior is wrong (fix the dip or the prior structure), or you believe the prior and want the computation to respect it more — then tighten **Dip Tolerance**.

### Interpretation shows sustained out-of-zone, but marginals are sharp and fit is good

Resist the urge to tighten tolerances until the answer changes. A sharp, confident, sustained out-of-zone result that contradicts independent drilling evidence usually means an *input* is wrong: re-examine the marker pick, the [alignment](./setup/align-logs.md), the dip prior, and whether a fault went unmodeled.

### Alternatives split into two clusters diverging at one MD

The data supports two stories that part ways at that depth. A fault near the divergence point is a natural hypothesis — try enabling or adjusting [fault parameters](./setup/job-parameters.md) on that range. Note the coupling: tightening dip freedom while faults are enabled can *force* a fault to explain what dip no longer can.

### Run fails with out-of-memory or timeout

This — and only this — is the signal to raise the [Executor](./setup/run-job.md#advanced-options) tier, or alternatively to reduce cost: coarser **Depth Resolution** / **MD Interval**, more aggressive **Prune**, or a smaller **Dip Tolerance**. A generic runtime error is not an out-of-memory signal.

### A one-shot run keeps restarting on a WITSML project

Polling delivered new data mid-run, which restarts the job. Pause [polling](./setup/run-job.md#witsml-polling), run, then resume.

## When you change things

- **Fit parameters** invalidate computed state: follow with **Reset and Run**.
- Tuning changes applied to the whole well also require **Reset and Run** — a plain Extend only applies them ahead of the virtual pointer.
- **Dip Tolerance** and **Prune** trade answer fidelity against memory and time; move them one step at a time.
