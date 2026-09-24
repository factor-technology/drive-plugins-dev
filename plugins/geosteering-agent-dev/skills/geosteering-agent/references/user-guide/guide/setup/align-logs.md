<!-- published at /help/guide/setup/align-logs.html on the Drive host -->

# Step 4 · Align Logs

Alignment calibrates the mapping between the pilot's type log and the active well's log. It is the foundation the whole interpretation rests on: get the alignment wrong and everything downstream inherits the error.

On multi-pilot projects, first pick which pilot the alignment applies to in the range selector.

## Auto-align

Press **Auto-align Logs**. Drive runs a server-side solver that:

1. Selects windows over the deepest trustworthy stretch of the active log, above the point where the well builds past ~75° inclination.
2. Puts both logs on a shared subsea datum and solves each window independently for the best depth shift.
3. Accepts a result only when several windows **independently agree**. Shallow junk MWD gamma shows up as windows that stop agreeing and is excluded automatically.

The run takes from a few seconds up to a couple of minutes. On success it fills in the fit parameters below and sets the **First MD to Compute** to the top of the corroborated alignment window.

If *no* windows agree, Drive changes nothing and reports that no corroborated alignment was found, with a per-window diagnostic. That is a deliberate fail-safe, not an error to retry blindly — inspect the log quality and, if appropriate, align manually.

When an automatic (depth-warped) alignment exists, an **Alignment** switch lets you choose between **Auto (depth-warped)** and **Manual (offset)**.

::: tip Always review
Whatever the solver reports, verify the alignment visually on the log track before running the job. In strongly characterized sections (a hot shale, say) many users align by eye and type the depth offset directly — that is a perfectly good workflow.
:::

## Fit parameters

Three numbers define the fit; auto-align sets them, and you can adjust each with sliders or direct entry:

| Parameter | Meaning |
|---|---|
| **Depth (TVDSS) Offset** | The vertical shift applied to the pilot log to place it in the active well's depth frame. A large value simply means the formation sits at a different depth at the two wells. |
| **GR Scale** | How much the pilot gamma range is stretched or compressed to match the active log. Values far from 1 indicate a materially different gamma environment. |
| **GR Offset** | A baseline gamma correction, for tools that read systematically higher or lower. |

The settings control adjusts the slider min/max ranges, and **Reset fit** clears the fit back to defaults.

::: warning Fit edits invalidate prior runs
Changing fit parameters invalidates previously computed state. After editing the fit on a project that has already run, use **Reset and Run** (Step 6) so the whole well is recomputed under the new calibration.
:::
