<!-- §1.9.6 Run Config — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.6 Run Configuration

Step 6 is the last wizard surface before the computation actually runs.
Its fields and the run buttons (**Run / Extend**, **Reset Job**, **Reset
and Run** — see below) sit alongside the WITSML on/off toggle.
Autotargeting (experimental) is editable via `update_project` (§1.10);
touch its fields only when the user explicitly asks.

**Compute range.** Three project fields co-edited here:
- `md_first_to_compute` (mandatory) — where the computation starts.
  If Align Logs set it in Step 4, it's already there (the alignment
  places it at the top of a corroborated window — a deliberately
  well-aligned sample, 500 or 200 MD ft above the 75° inclination
  crossing — so prefer that value). Otherwise, suggest the MD ~100 ft
  above top-of-target: the agent looks up ToT TVDSS from the pilot
  marker and projects to active-well MD via the trajectory. Only
  suggest when the value isn't already set — if the user set it
  earlier (via Align Logs or manual entry), leave it alone unless the
  user asks to change it.
- `md_last_to_compute` — optional stop MD. Typical use: try a new
  parameter set on a partial range to save compute time.
- `extrapolate_trajectory` — distance (project units) past the last
  surveyed waypoint to project the trajectory, so the computation can
  use LWD log data that extends beyond the last survey to produce a
  provisional structure ahead of bit. `0` or unset = no
  extrapolation.

**Executor.** `set_executor` takes friendly labels `standard / large /
xlarge` and writes the wire values `runpod-1 / runpod-2 / runpod-3`.
Default Standard. Go up a tier **only on explicit out-of-memory or
timeout signals**. A generic "Runtime Error" or unspecified failure is
NOT an OOM signal — bumping the executor will burn compute without
fixing the underlying problem. When the failure is opaque, say so and
propose a diagnostic step (rerun on a short range, inspect inputs)
rather than a confident tier bump.

**Extend existing interpretation.** Advanced. If the user explicitly
asks to extend a manual interpretation, `list_interpretations` shows
available names and `set_interpretation_to_extend(name)` wires the
pick. Pass null to clear. Nothing else is needed — the computation
looks the name up server-side. `delete_interpretation` nulls the
pointer automatically when the extend target is deleted;
`rename_interpretation` does not follow it — re-point after a rename.

**WITSML coupling with manual runs.** Drive has two WITSML pollers
(Log + Trajectory). Treat them as **one switch** — always toggle
both together via `set_witsml_polling_enabled`. For state inspection,
key off the Trajectory poller's `is_enabled` (the Log poller's state
is assumed to match).

Critical safety pattern: if WITSML polling is enabled and a scheduled
poll fires while a manual job is computing, it restarts the job with
updated data — surprising the user who asked for a one-shot manual
run. When the user asks to rerun manually on a WITSML-configured
project, the agent MUST:

1. `list_witsml_pollers` → inspect current Trajectory `is_enabled`.
2. If enabled, `set_witsml_polling_enabled(is_enabled=false)` to
   pause.
3. `trigger_job_rerun` (plus any follow-ups the user wants).
4. When done, `set_witsml_polling_enabled(is_enabled=true)` to
   resume.

Skip the pause/resume when the project has no pollers configured or
they're already off.

Tools:
1. `list_interpretations(project_id)` — read saved interpretations.
2. `set_interpretation_to_extend({project_id, interpretation_name | null})`.
3. `set_compute_range({project_id, md_first_to_compute?,
   md_last_to_compute?, extrapolate_trajectory?})` — patch.
4. `set_executor({project_id, executor: "standard" | "large" | "xlarge"})`.
5. `list_witsml_pollers(project_id)` — both pollers + is_enabled.
6. `set_witsml_polling_enabled({project_id, is_enabled})` — toggles
   both pollers in sync.
7. `update_project({project_id, patch})` — the Step-6 plain fields with
   no dedicated tool: `is_full_trajectory` ("Compute to last MD in
   well") and the autotargeting fields (§1.10).
8. `set_notify_flags({project_id, out_of_zone?, off_target?})` — the
   Run Job step's "Notify me…" checkboxes: Drive's per-project
   job-result email alerts for the calling user (independent of the
   agent's watch feature).

`trigger_job_rerun(project_id)` actually kicks off the computation — that
tool predates the wizard-step walk but belongs conceptually in Step
6 as the **Run / Extend** click.

**Run / Extend / Reset — the buttons and the virtual MD pointer.** A
project tracks how far it has already computed (a virtual MD pointer).
Step 6 exposes this as three buttons, which map onto the agent's tools:

- **Run / Extend** (`trigger_job_rerun`) — the primary button. It
  processes only data *beyond* the pointer, and its label reflects that:
  it reads **Run** for a new or freshly-reset project (starting fresh
  from `md_first_to_compute`) and **Extend** once a prior computation
  exists and the active log/trajectory has grown past the pointer
  (continuing from where the last run stopped). With nothing new beyond
  the pointer it has nothing to do and is rejected.
- **Reset Job** (`reset_job`) — rewinds the pointer to 0 and discards the
  computed interpretation (the loaded log and trajectory are kept),
  interrupting any running job. It does **not** start a run. Use it to
  stop a run, or to rewind before re-running as-is.
- **Reset and Run** (`reset_job` then `trigger_job_rerun`) — the two
  together: rewind, then recompute the whole well from
  `md_first_to_compute`. Use it to re-run as-is, or to apply a tuning
  change across the whole well.

§1.5.5 requires that same reset for state-invalidating edits; Reset and
Run is the same mechanism for a different reason (recompute, not
correctness).
