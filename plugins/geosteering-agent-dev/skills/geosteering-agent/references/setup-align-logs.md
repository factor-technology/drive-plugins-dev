<!-- §1.9.4 Align Logs — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.4 Align Logs

Step 4 calibrates the depth-and-scale mapping between the MD-0 pilot
("type") log and the active GR log, so the computation can compare
like-with-like. The UI's "Align Logs" button and the agent's align-logs
tools run the SAME server-side solver (the log-align sweep endpoint), so
their results are identical by construction. Neither the
geologist nor the agent picks an alignment window anymore — the solver
does its own windowing:

- The server windows the deepest trustworthy stretch of the active log
  at ~13 depths (100–2000 MD ft above the point where inclination
  crosses 75°) and solves each window for the depth shift that best
  registers it on the pilot's own TVD axis. The shift search is centered
  on the active well's elevation — auto-align assumes the pilot and
  active pads sit within a few hundred feet of each other (roughly zero
  shift in the depth-below-rig frame).
- A result is accepted only when **several windows independently agree**
  on the shift (corroboration). Junk shallow log — common in MWD GR —
  shows up as windows that stop agreeing, and is excluded automatically.
- Some wells' logs only correlate after smoothing; the solver retries at
  coarser matching scales (2 → 10 → 20 ft) on its own.
- On success the server persists `fit_params: {datum_tvdss, gr_scale,
  gr_offset}` to the MD-0 pilot well and sets the project's
  `md_first_to_compute` to the top of a corroborated window (500 or
  200 MD ft above the 75° crossing) — a deliberately well-aligned sample
  for the computation to start from.
- If **no** windows agree at any scale, the server changes NOTHING and
  returns "no corroborated alignment" (HTTP 422) with a per-window
  diagnostic table. This fail-closed outcome is a feature: it is the
  system saying "no reliable tie exists between these logs", not an
  error to retry.

**Alignment is optional.** A geologist may skip the button and type
fit_params directly (via sliders or numeric entry). In some
formations — Middle Bakken / Three Forks wells — the Upper and Lower
Bakken shales are so hot that pilot and active GR values diverge across
them even though the shape boundaries align. In those cases the
geologist may align vertically by eye on the shale transitions and set
the depth tie (`datum_tvdss`) manually via `set_fit_params`. After an
auto-alignment, the saved Auto (depth-warped) vs Manual (offset) display
choice is togglable via `set_warp_display`.

**Agent behavior:**
- **Start it, then poll.** `start_align_logs({project_id})` — no other
  inputs — returns a handle; poll `get_operation_status` every ~15–20 s (no
  faster) until it succeeds or fails. Warn the user first that it typically
  takes **1–3 minutes** (the solver runs up to ~40 GPU passes on a serverless
  endpoint, including cold start), which is why the synchronous `align_logs`
  can outrun a host's tool-call timeout; that one is local-runtime only
  (§1.1). The result is identical either way, so everything below applies to
  both.
- It aligns the **MD-0 pilot only** (the server's flow). For additional
  pilots, use `set_fit_params` with numbers the geologist supplies.
- **Corroborated result:** relay the fit params in geologist language
  (§1.1 vocabulary), plus how strongly the tie is supported — the picked
  window and the list of windows that agree with it (e.g. "five windows
  between 500 and 900 ft agree on the shift"). More agreeing windows =
  stronger tie.
- **Uncorroborated (422):** tell the user no reliable alignment was
  found and that nothing was changed. Summarize the per-window table if
  useful (which windows solved, how scattered their shifts were).
  One diagnosable cause: a pilot whose pad elevation differs from the
  active well's by much more than a few hundred feet puts the true
  shift outside the search window — the remedy is a manual tie.
  Sensible next steps to offer: reconsider whether this pilot correlates
  with the active interval, or tie manually with `set_fit_params`. Do
  NOT simply re-run the tool — the same inputs give the same answer.
- **Review is mandatory.** A successful alignment always returns a `review_url`.
  The agent surfaces it every time and tells the user to verify — as a
  friendly Markdown link to the Align Logs pane whose target is the tool's
  `review_url`, e.g.
  `review the [Align Logs](https://{host}/{scope}/projects/{name}/setup#align-logs) result`
  (§1.7.1). Never print "Align Logs" as a plain label with the URL dropped.
- **Numeric entry:** `set_fit_params` accepts explicit fit_params with
  optional auto_fit_range and md_first_to_compute, running no solver.
  Use this when the user supplies numbers directly.

**Interpreting alignment results.** Align Logs — button or tool —
produces the three fit parameters the UI shows as **Depth Offset**,
**GR Offset**, and **GR Scale** (the fit-params pane labels; §1.1). In
the geologist's language:

- **Depth Offset** (`datum_tvdss`, ft): the depth tie — the world TVDSS
  of the pilot's TVD-zero point (`TVDSS = datum_tvdss − pilotTvd`), i.e.
  the vertical placement that carries the pilot log into the active
  well's depth frame. Judge it by how the aligned logs look, not by its
  magnitude.
- **GR Offset** (`gr_offset`, GR units): a baseline GR correction — the
  active log reads systematically higher or lower than the pilot, common
  when tool generations or borehole conditions differ.
- **GR Scale** (`gr_scale`, dimensionless): how much the pilot's GR range
  was stretched or compressed to match the active. A scale far from 1
  signals a significant GR-environment difference — flag it for a visual
  check before accepting.

These are qualitative reads, not thresholds: what counts as "large"
depends on the play, so anchor judgement on whether the aligned logs
look right on the display rather than on a fixed number. Corroboration
adds a quantitative confidence read the old flow lacked: the
`corroborated_by_windows` list in the tool result says how many
independent windows support the shift. (Manual slider/numeric override
after auto-alignment is covered under "Alignment is optional" above.)

Tools:
1. `read_fit_params(project_id)` — per-pilot fit_params + auto_fit_range,
   plus project's md_first_to_compute.
2. `set_fit_params({project_id, pilot_well_id, fit_params,
   auto_fit_range?, md_first_to_compute?, clear_log_align?})` — direct
   numeric entry; no solver. Preserves existing datum_tvdss when the
   caller omits it. `clear_log_align: true` with the default scalars
   (gr_scale 1, gr_offset 0, datum_tvdss 0) is the UI's Reset Fit —
   it deletes the persisted align-logs warp.
3. `start_align_logs({project_id})` + `get_operation_status` — server-side
   sweep alignment + persist (MD-0 pilot; server writes fit_params and
   md_first_to_compute). Long-running (1–3 min). The finished operation
   carries fit params, the per-window diagnostic table, corroboration info,
   and a review_url the agent must surface. `corroborated: false` + the table
   means no alignment exists; nothing is persisted in that case. (`align_logs`
   is the same call run synchronously — local runtime only.)
4. `set_warp_display({project_id, pilot_well_id?, enabled})` — the
   Align Logs "Alignment:" radio — Auto (depth-warped) vs Manual
   (offset). Flips the saved warp's enabled flag and swaps datum_tvdss
   accordingly; no solver runs and the warp is kept either way.
