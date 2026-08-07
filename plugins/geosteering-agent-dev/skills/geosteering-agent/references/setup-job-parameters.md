<!-- §1.9.5 Job Parameters — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.5 Job Parameters

Step 5 is the **tuning surface** (§1.5). It covers both per-MD-range
`param_block` overrides and two project-level discretization knobs.

**UI tolerances vs. stored widths.** The UI labels the width fields as
tolerances — the maximum deviation the user expects (the 99.7% bound):
"Log Tolerance" = 3 × `log_sigma` under Strict/normal, or
5.81 × `log_sigma` under Forgiving/laplace (the stored value is then the
Laplace scale b); "Dip Tolerance" = 3 × `dip_sigma`. The Step 5
"Curvature" select (`delta_dip_sigma_per_x`) also displays a 3σ
tolerance, but additionally over a nominal distance — 100 ft (US) or
30 m (SI): displayed ° = 3 × stored × nominal_distance, so the stored
value per unit distance = tolerance / (3 × nominal_distance). The API
and tools still take the stored widths: treat a number the user quotes
as a UI tolerance — divide by 3 (or 5.81 under Forgiving; and by the
nominal distance for curvature) before writing it with a tool, and
convert stored widths back to tolerances when reporting. The
UI keeps the displayed Log Tolerance constant when Outlier Handling is
toggled, by rescaling the stored width; do the same when you change
`log_distribution` for the user.

**Per-block fields** (each range is a `param_block` at some `md`):
- `dip_sigma` (deg) — one-sigma deviation from the prior dip (the UI's
  "Dip Tolerance" is the 3σ value). 68% of
  dips within ±σ, 99.7% within ±3σ. Applies in both `"constants"` and
  `"structure"` dip modes — for structure mode it describes the spread
  around the laterally-varying dips implied by the polyline.

  **Memory/compute cost** scales with `dip_sigma` — higher sigma
  produces larger factors and longer runs. Bump one tier at a time.

  **Discretization interplay.** At small `dip_sigma` (≤ ~1°), the
  cone of possible dips is narrow. If `interval` (MD sampling) and
  `delta_spatial` (depth grid) are too coarse, the next MD step's
  candidate cells all fall within the same depth cell — the
  discretization has no room for variation above or below the
  centerline of the ray from the current position, and the algorithm
  has nothing to weight. Fix: shorten `interval` and/or tighten
  `delta_spatial` when lowering `dip_sigma` below ~2°. This is the
  inverse of the stair-step warning in §1.5: small sigma + coarse
  grid = dead discretization; small sigma + fine grid = clean
  constraint.
- `delta_dip_sigma_per_x` — the **curvature constraint** (UI
  "Curvature"): how fast the geologic dip is allowed to change with
  lateral distance. Stored as sigma-degrees per horizontal distance
  unit (per ft for US projects, per m for SI); `0` = unlimited
  (structure may bend freely). Higher values permit rougher structure;
  lower values keep it smoother. The UI quotes a **3σ tolerance over a
  nominal distance** — 100 ft (US) or 30 m (SI): displayed ° =
  `3 × stored × D`, so `stored = tolerance / (3 × D)`. A user's "30°"
  on a US project → `30 / (3 × 100) = 0.1`; on SI → `30 / (3 × 30) =
  0.333`. The default is the UI's "30°" choice (0.1 deg/ft for US).
  Pick `D` from the project's `spatial_som` (`read_job_params`
  surfaces it). Editable per-block via `update_param_block` /
  `add_param_block`.
- `log_sigma` — one-σ measurement uncertainty on the active GR log,
  in gAPI (e.g. `log_sigma=10` means 99.7% of readings within ±30
  gAPI of the true rock value — the UI's "Log Tolerance" of 30). With
  `log_distribution="laplace"`, the value is actually the Laplace
  scale b, not a Gaussian σ, and the UI tolerance is 5.81 × b.
- `log_distribution` — `"normal"` (Gaussian, default; UI "Outlier
  Handling: Strict") or `"laplace"` (UI "Forgiving" — heavier tails,
  more forgiving on noisy LWD data).

- `cull_factor` — pruning threshold for small-probability outcomes
  (trades memory/time for fidelity). The UI offers `{-25, -50, -100,
  -200, -300, "No pruning"}` on a log scale: wire value is
  `10^(prune/10)`, except "No pruning" → `0`. Example: UI "-25" →
  `10^-2.5 ≈ 3.16e-3`, UI "-300" → `1e-30`, UI "No pruning" → `0`.
  Smaller (or 0) = less aggressive pruning, more memory.
- `fault_num_total` — expected number of faults per 10 kft of lateral,
  assumed to follow a power-law throw distribution. `0` disables
  faults entirely.
- `fault_throw_min`, `fault_throw_max` (ft) — power-law bounds. One
  fault is expected at max-throw; smaller ones populate down to
  min-throw.
- `fault_up_weight` (0–1) — prior on upthrow vs. downthrow as viewed
  along the wellbore toward the bit. `1` = all upthrow, `0` = all
  downthrow, `0.5` = 50/50.

Two per-block params — `dip_constant` and `strike_constant` — are
Step 3's surface and must NOT be edited via Step 5 tools. The agent
redirects users to `set_apparent_dip` / `set_vs_azimuth` on attempts.
A block's `pilotWellId` is a schema mistake — agent tools never set
or read it.

**Project-level fields** (edited via `set_project_job_params`):
- `interval` — MD sampling interval for structure inference. UI
  choices `{3, 6, 12}` ft (or metric: × 0.3048); default 12 ft.
  Coarser intervals are less sensitive to brief GR excursions.
- `delta_spatial` — vertical depth-grid resolution. UI choices
  `{0.25, 0.5, 1.0}` ft (= 3, 6, 12 inches; × 0.3048 for metric);
  default 0.5 ft. Higher resolution costs memory/time. There's
  interplay between `dip_sigma`, `interval`, and `delta_spatial`: a
  small dip_sigma with a coarse delta_spatial and short interval can
  produce visible "stair-step" artifacts — loosen one of the three to
  fix.
- `sigma_md` and `inclination_sigma` appear in the project body but
  are UNUSED. Agent tools must NOT set them.

**"Impossible" from GR antialiasing.** Drive smooths the active GR
onto the `interval` grid before evaluating it, so the value the
computation sees can diverge from the raw value the user sees. Across
a strong GR contrast — canonically the Upper/Lower Bakken hot shales
adjacent to Middle Bakken or Three Forks targets, but any sharp
hot/cool interface qualifies — a tight `log_sigma` may then reject
the survey as "Impossible". Shorten `interval` or loosen `log_sigma`.

**Writes round-trip the whole project.** Drive's UI does a
`PUT /api/v2/{scope}/projects/{name}` with the full project body on
every edit, mutating `param_blocks` inline. The agent uses the same
path via `getProject → modify → putProject`.

**Range semantics.** A block at `md=m` applies from `m` up to the
next block's `md` (or ∞ for the last). The `md=0` block always
exists (whole-well default) and cannot be deleted. Splitting a range
means inserting a new block at some intermediate `md`; the two halves
inherit the old params and the user can then tune each side.
Boundaries are movable after the fact via `move_param_block`.

Tools:
1. `update_param_block({project_id, block_md, params_patch})` —
   shallow-merge patch into the block's params. Rejects
   `dip_constant`, `strike_constant`, `pilotWellId`, and any unknown
   key with a helpful error message pointing to the right tool.
2. `add_param_block({project_id, md, params_patch?})` — insert a new
   block at `md`. Params inherited from the preceding block (largest
   `md` strictly less than the new one), with optional `params_patch`
   overrides at creation time. Errors on duplicate `md` or `md ≤ 0`.
3. `delete_param_block({project_id, md})` — remove the block at `md`.
   Disallows `md=0`. The preceding block's range naturally extends
   forward.
4. `move_param_block({project_id, block_md, new_md})` — move a block's
   starting MD; the block keeps its identity and params, and the
   neighboring ranges stretch/shrink to fill. The `md=0` block cannot
   move.
5. `set_project_job_params({project_id, patch})` — update project-
   level `interval` and/or `delta_spatial`. No other keys accepted
   (the "unused" `sigma_md` / `inclination_sigma` must not be set).
6. `reset_job_params(project_id)` — the UI's "Reset to Defaults"
   button: overwrites every block's eight tuning params and the
   project-level `interval`/`delta_spatial` with the defaults; prior
   values are not recoverable. The Dip & Azimuth fields keep their
   stored values.
