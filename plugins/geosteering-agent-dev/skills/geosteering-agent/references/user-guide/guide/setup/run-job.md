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
- **Executor** — the compute tier: **Standard**, **Large**, or **Extra Large**. Use **Large** whenever fault modeling is on (Expected faults above 0 on any range); otherwise use Standard. Beyond that, go up a tier only when a run actually fails with an out-of-memory or timeout error; a generic runtime error is *not* an out-of-memory signal, and escalating the tier won't fix it.

## Automatic runs

**Automatic runs** decide whether this project runs when its well grows. With them on, each delivery from the well's WITSML or email source will start a run of this project when the well's data reaches past its last run, and a reset and rerun when the well's data was revised; the run will be in the name of whoever turned them on. With them off, the project will wait for you to press **Extend**. The switch needs Write access to the project, not to the well, and belongs to this project alone: the other projects on the well keep their own. Automatic runs will start nothing while the well's data source is manual file upload or the well is not receiving data.

A new, cloned or imported project starts with Automatic runs off. On a project that has not run yet (or was just reset), on a well that is receiving WITSML or email data, the first **Run** can turn them on once the run has started: in the guided Run step the main button reads **Run and turn on automatic runs**, and the link **Run without automatic runs** keeps the run one-shot. From then on the first run is a run in progress like any other, and a delivery with new data will replace it. For a first run that must finish uncapped, choose **Run without automatic runs**, and turn them on once the run has finished. While they are on and the well is receiving data, the **Run** button is disabled, so for a deliberate run turn them off, run, and turn them on again once the run has finished. A delivery with new data will replace a run still in progress; one with nothing new will leave it alone. A WITSML run started this way is capped at nine minutes of reverse phase. The panel shows how the last automatic start went, for example that the user it runs as can no longer write the project.

## Running

- **Run / Extend** — the main button. It reads **Run** on a fresh (or reset) project and **Extend** when prior computed state exists; extending continues incrementally from where the last job stopped, which is why routine updates are fast. It is disabled when there is nothing new to extend.
- **Reset** — rewinds the computation to the beginning: discards computed results (your data and settings are kept) and interrupts any running job. It does not start a run by itself.
- **Reset and Run** — reset, then recompute the whole well. Required after state-invalidating edits (notably [fit parameters](./align-logs.md#fit-parameters)) and the normal way to apply a tuning change to the entire well.
- **Clear Well Data** — deletes the active well's trajectory and log, for every project on the well. It needs Write access to the well. Confirmation required.

Starting a run opens the Profile tab, whatever the well's data source; progress appears at the right of the tab bar.
