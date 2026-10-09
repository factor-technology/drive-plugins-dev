<!-- published at /help/guide/setup/run-job.html on the Drive host -->

# Step 6 · Run Job

The final step configures and launches the computation. A readiness message just above the buttons tells you whether the project can run and what, if anything, is missing.

## Compute range

| Field | Meaning |
|---|---|
| **First MD to Compute** | Where the computation starts. Align Logs sets this automatically; otherwise start a little above where the well enters the zone of interest. Set by the interpretation when one is chosen under *Extend existing interpretation*. |
| **Last MD to Compute** | Optional stop point. Typical use: test a parameter change on a short stretch before committing to a whole-well recompute. Disabled while *Compute to last MD in well* is ticked. |
| **Extrapolate trajectory distance** | Projects the trajectory this far past the last survey so gamma data beyond the last survey still yields a provisional structure ahead of the bit. 0 disables. |
| **Compute to last MD in well** | Convenience checkbox: always compute to the end of the data. |

## Notifications

Two checkboxes ask Drive to alert you when the computed interpretation indicates trouble: **Notify me if wellbore out of zone** and **Notify me if wellbore off target line**. Alerts are delivered through the channels configured on your [account page](../../admin/account.md).

## Other settings

- **Extend existing interpretation** — pick a saved manual interpretation for the computation to extend, instead of extending its own prior result.
- **Autotargeting** (experimental; a collapsible panel) — proposes steering targets: enable it and set the centerline depth below top of target, the corridor above/below the centerline, and a dogleg-severity penalty coefficient.
- **Executor** — the compute tier: **Standard**, **Large**, or **Extra Large**. Stay on Standard unless a run actually fails with an out-of-memory or timeout error; a generic runtime error is *not* an out-of-memory signal, and escalating the tier won't fix it.

## Live updates

On WITSML and email projects, a pause/resume control shows whether live updates are running or paused. WITSML polling starts paused: start it here once the connection is set up. While live updates are on, the **Run** button is disabled — Drive runs the job itself when data arrives — so for a deliberate run, pause, run, then resume. In WITSML mode pausing stops data retrieval; in email mode it stops only the automatic runs, and emailed data still lands.

## Running

- **Run / Extend** — the main button. It reads **Run** on a fresh (or reset) project and **Extend** when prior computed state exists; extending continues incrementally from where the last job stopped, which is why routine updates are fast. It is disabled when there is nothing new to extend.
- **Reset** — rewinds the computation to the beginning: discards computed results (your data and settings are kept) and interrupts any running job. It does not start a run by itself.
- **Reset and Run** — reset, then recompute the whole well. Required after state-invalidating edits (notably [fit parameters](./align-logs.md#fit-parameters)) and the normal way to apply a tuning change to the entire well.
- **Clear Well Data** — deletes the active well's trajectory and log from the project. Confirmation required.

Starting a run navigates to the Profile tab; progress appears at the right of the tab bar.
