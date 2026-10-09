<!-- published at /help/guide/setup/align-logs.html on the Drive host -->

# Step 4 · Align Logs

Alignment calibrates the mapping between the pilot's type log and the active well's log. It is the foundation the whole interpretation rests on: get the alignment wrong and everything downstream inherits the error.

## Auto-align

Press **Auto-align Logs**. Drive runs a server-side solver that:

1. Selects windows over the deepest trustworthy stretch of the active log, above the point where the well builds past ~75° inclination.
2. Puts both logs on a shared subsea datum and solves each window independently for the best depth shift.
3. Accepts a result only when several windows **independently agree**. Shallow junk MWD gamma shows up as windows that stop agreeing and is excluded automatically.

It aligns the first pilot (the one at MD 0); on multi-pilot projects the other pilots are placed by their top-of-target markers, not by a datum of their own.

The run takes from a few seconds up to a couple of minutes. On success it fills in the fit parameters below, selects **Auto (depth-warped)** on the **Alignment** switch, and sets the **First MD to Compute** to the top of the corroborated alignment window.

If *no* windows agree, Drive changes nothing and reports that alignment failed. That is a deliberate fail-safe, not an error to retry blindly — inspect the log quality and, if appropriate, align manually.

The **Alignment** switch chooses between **Auto (depth-warped)** and **Manual (offset)**. In Auto the depth datum is the solver's and its box is disabled; switch to Manual to set your own, which is remembered if you switch back. Running Auto-align again returns the switch to Auto.

::: tip Always review
Whatever the solver reports, verify the alignment visually on the log track before running the job. In strongly characterized sections (a hot shale, say) many users align by eye and type the depth datum directly — that is a perfectly good workflow.
:::

## Fit parameters

Three numbers define the fit; auto-align sets them, and you can adjust each with sliders or direct entry. On multi-pilot projects, the range selector above picks which pilot your edits apply to.

| Parameter | Meaning |
|---|---|
| **Depth Datum (TVDSS)** | The TVDSS of the type log's zero depth (TVDSS = datum − TVDTL). It places the pilot log in the active well's depth frame; a large value simply means the formation sits at a different depth at the two wells. |
| **GR Offset** | A baseline gamma correction, for tools that read systematically higher or lower. |
| **GR Scale** | How much the pilot gamma range is stretched or compressed to match the active log. Values far from 1 indicate a materially different gamma environment. |

The gear icon opens **FitLog Parameter Ranges**, which sets the slider min/max, and **Reset** clears the fit back to defaults.

::: warning Fit edits invalidate prior runs
Changing fit parameters invalidates previously computed state. After editing the fit on a project that has already run, use **Reset and Run** (Step 6) so the whole well is recomputed under the new calibration.
:::
