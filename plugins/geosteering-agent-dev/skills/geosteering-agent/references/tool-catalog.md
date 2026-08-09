<!-- §1.10 Tools — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

## 1.10 Tools

The tool catalog grows incrementally toward **UI parity** — the design goal is
that you can carry out almost any action a geologist could perform in the Drive
web UI. Only a subset is wired up today; each new capability is typically
added when a user scenario first needs it.

**The live tool schemas are the per-tool documentation.** Every drive-mcp
tool carries its full contract — semantics, depth frames, side effects,
paired follow-up writes, destructiveness, when to prefer a sibling tool — in
its own description and parameter docs, delivered with the tool list by your
MCP host. This section does not repeat them; it holds only the rules that
span tools. (Wire-level endpoint paths are deliberately undocumented per
tool; they are not something you call directly.)

**Inspect before asking.** When the user asks you to do a setup task ("do
step 4", "align the logs", "configure the pilot"), do not enumerate
prerequisites as if the project were empty. Call the relevant read tools
first — `read_project`, `list_pilot_wells`, `read_pilot_well_markers`,
`read_fit_params`, `read_active_trajectory`, `read_user_prior`,
`read_interpretation`, `read_target_line` — and only
ask the user for what is genuinely missing. The wizard's step list is a
workflow scaffold, not a per-cycle checklist; on a project that already has
pilot wells, logs, markers, and trajectory uploaded, "do step 4" means run
the alignment with the current inputs, not re-collect them.

Cross-tool rules:

- **Result reading is tiered.** `read_latest_job_result` returns a compact
  summary (run ID, MD range, probs_pct, MPE tail); the data lives behind the
  targeted readers `read_marginal` (one MD) and `read_mpe_slice` (an MD
  window). Prefer the targeted readers — far less context, and claims stay
  MD-localized (§1.6.1).
- **Cross-project work starts at `list_projects`** — the only cross-project
  tool; every other tool needs a `{scope, name}` you already hold.
- **WITSML-live projects: pause around manual runs.**
  `set_witsml_polling_enabled(false)`, run, re-enable — otherwise a
  scheduled poll restarts the manual run mid-computation (§1.9.6).
- **State-invalidating edits pair with a reset** — `reset_job` then
  `trigger_job_rerun`, advisory + approval (§1.5.5, §1.9.6).
- **Prefer clone-then-experiment.** For any change you'd otherwise gate
  behind approval, `copy_project` to a clearly-named sandbox, experiment
  there, report back.
- **Bulk samples should never transit the model.** Anything large goes through
  the `create_upload` staged send (§1.9.1).
- **Long-runners are handles, not calls.** Alignment and WITSML re-crawls run
  minutes: use `start_align_logs` / `start_refresh_witsml_server` and poll
  `get_operation_status` every ~15–20 s.

The catalog, by area (names only — contracts live in the schemas):

- Job results & status: `read_job_status`, `read_latest_job_result`,
  `read_marginal`, `read_mpe_slice`, `get_operation_status`
- Project inspection: `read_project`, `read_job_params`, `read_user_prior`,
  `read_structure`, `read_fit_params`, `read_pilot_well_markers`,
  `read_active_trajectory`, `read_well_plan`, `read_active_log`,
  `read_pilot_log`, `read_interpretation`, `read_target_line`,
  `list_projects`, `list_pilot_wells`, `list_interpretations`
- Project lifecycle: `create_project`, `copy_project`, `delete_project`,
  `update_project`
- Pilot wells (§1.9.1): `create_pilot_well`, `rename_pilot_well`,
  `delete_pilot_well`, `upload_pilot_log`, `set_pilot_well_markers`,
  `set_top_of_target`
- Active well (§1.9.2): `upload_active_trajectory`,
  `delete_active_trajectory`, `upload_well_plan`, `delete_well_plan`,
  `upload_active_log`, `delete_active_log`, `clear_well_data`,
  `set_email_config`, `set_ignore_ranges`, `set_data_source_mode`
- Dip & azimuth (§1.9.3): `set_vs_azimuth`, `set_apparent_dip`,
  `set_dip_structure_mode`, `upload_structure`, `delete_structure`,
  `set_structure_from_interpretation`
- Align logs (§1.9.4): `start_align_logs`, `set_fit_params`,
  `set_warp_display`
- Job parameters (§1.9.5): `update_param_block`, `add_param_block`,
  `delete_param_block`, `move_param_block`, `set_project_job_params`,
  `reset_job_params`
- Run configuration (§1.9.6): `set_interpretation_to_extend`,
  `set_compute_range`, `set_executor`, `set_notify_flags`, `reset_job`,
  `trigger_job_rerun`
- Manual interpretations: `create_interpretation`, `update_interpretation`,
  `rename_interpretation`, `delete_interpretation`,
  `copy_computed_interpretation`
- Target line: `set_target_line`, `delete_target_line`
- Background (seismic) image: `read_seismic_image`, `upload_seismic_image`,
  `set_seismic_image_registration`, `delete_seismic_image`
- WITSML: `list_witsml_servers`, `add_witsml_server`,
  `start_refresh_witsml_server`, `delete_witsml_server`,
  `browse_witsml_crawl`, `list_witsml_pollers`,
  `set_witsml_trajectory_poller`, `set_witsml_log_poller`,
  `refresh_witsml_pollers`, `set_witsml_polling_enabled`
- File ingest (§1.9.1): `create_upload`, `read_las`
- Links: `project_links`, `get_guidance` (an in-protocol digest of this spec
  for hosts that can't load a skill — redundant while you have this one;
  don't call it)

The tool catalog is the contract: if you need an action that's not in the
list, ask the geologist to add it rather than inventing a workaround.
