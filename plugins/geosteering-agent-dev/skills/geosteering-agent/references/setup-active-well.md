<!-- §1.9.2 Active Well — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.2 Active Well

Step 2 of the wizard is the **active well** — the well being
geosteered. It carries a measured trajectory (the surveys, growing as
the well drills), an LWD gamma-ray log (also growing), an optional well
plan (the intended path, used as a visual aid), an optional list of
MD ranges to ignore in the computation, and a reference elevation.

**Datum elevation: optional, default 0 (TVD).** Most projects work best
in TVD (`datum_elevation = 0`); depths read as feet below the well's
surface. A minority of users want TVDSS — e.g. to compare with a
regional structure map or import a TVDSS-referenced interpretation.
Offer TVDSS as an option once when collecting trajectory/log inputs and
default to 0 if the user doesn't specify or isn't sure. Do not press.

The wizard supports three data-source modes via radio buttons:
**Manual File Upload** (LAS for the log, CSV/Excel for trajectories),
**WITSML Connection** (Drive polls a WITSML server on a schedule), and
**Email** (the user emails LAS / Excel files to a project-specific
fmail address). The mode is stored on the project as
`data_source_mode: "manual" | "witsml" | "email"`.

**Manual upload — three independent endpoints, no shared wiring step.**
Unlike pilot wells (whose upload ends with a separate project-PUT
recording `pilot_log_curve_mnemonic`), the active-well uploads write
straight to dedicated resources:

- `PUT /active-well/trajectories/measured` — body `{waypoints:
  [{md, incl, azi, e?, n?, tvd?, vs?, mapd?}, ...]}`. MD/INCL/AZI
  required; the rest pass through if present. A `tvdss` column/field is
  accepted on input but never stored — both upload tools strip it from
  every waypoint before the PUT, exactly like the UI dialog (Drive
  derives world TVDSS from the active datum elevation and TVD; a stored
  waypoint tvdss would change the world frame). Full-replace each call —
  re-upload the longer file as the well grows. `upload_active_trajectory`
  also does the wizard's paired project PUT: persisting the survey's
  column layout (`active_measured_trajectory_column_indexes` — see
  below) and syncing `md_last`/`md_last_to_compute`.
- `PUT /active-well/well-plan` — same waypoint shape, different endpoint;
  optional.
- `PUT /active-well/logs/GR` — same `log_curve` body shape as the pilot
  POST's `log_curve` sub-object. Active logs are **never straightened**
  (no `trajectory` parameter — see source comment in
  `app/actions/log-curve/put-active-log-curve.js`). Full-replace.

Each upload has a mirroring delete — `delete_active_trajectory`,
`delete_well_plan`, `delete_active_log`, plus `clear_well_data` for
trajectory + log in one call — which also clears the project fields the
UI clears in the same gesture (`md_last`, `tvdss_first_to_compute`,
`md_last_to_compute`, as each delete's §1.10 entry details).
Irreversible; re-upload is the only recovery.

`upload_active_log` also runs the LAS preprocessing the wizard does
before sending: ascending MD, NaN/null dropped, monotonic-MD
(deeper-in-file repeat-section samples win), optional median despike,
index-UOM conversion into the project's spatial units (file mode reads
the source unit off the LAS ~Well STEP line), sample-interval detected
as the most frequent consecutive spacing, gaps linearly interpolated
onto that grid. After the PUT it syncs three project
fields — `active_log_curve_mnemonic`, `md_last`, `md_last_to_compute` —
using the wizard's `md_last = min(last_traj_md, last_log_md)` rule
(skipped if the trajectory hasn't been uploaded yet). The persisted
despike pair — `is_despike` / `despike_window`, the Define Log Curve
modal's "Apply despiking filter" controls — is editable via
`update_project`. Its LAS reaches the tool by the same three transport
paths as the pilot log (§1.9.1); on the remote connector mint the upload
with `kind: "active_log"`.

Column-index selection for trajectory CSV/Excel is a chat-layer
concern: the agent inspects the file, presents the user with the
columns, and gets MD/incl/azi (and optional) selections. Pass them to
`upload_active_trajectory` as file-mode `columns` (or `column_indexes`
alongside inline waypoints); the tool persists the layout to the
project, where fmail email ingest reuses it to parse emailed CSVs.
Surveys are small, so the remote connector has no staged path for them:
parse the rows yourself and pass `waypoints` inline (§1.1).

**Email mode — `fmail` attachments.** Instead of manual upload, the
user forwards LAS / Excel attachments to a project-specific address and
Drive extracts the designated GR curve on receipt. Configured via
`set_email_config({curve_mnemonic, is_enabled?})`, which wraps
`PUT /email-config`. `is_enabled: false` pauses only the automatic job
run on arrival; attachments still update the well data.
The tool also returns the computed fmail address:
`<base32(scope/name)>[+host]@fmail.factor.technology` (the `+host`
suffix is omitted when the Drive host starts with `drive`; for dev /
staging hosts it routes each install to its own inbox). If the encoded
scope/name would exceed the 64-char email local-part limit, the tool
returns `email_address: null` and the user must shorten one or both.

**Ignore LWD ranges.** Sections of the active log can be excised from
the computation without editing the raw upload — useful for tool
malfunction / obvious-spike intervals. Stored as
`project.lwdIgnoredMD: [[startMD, endMD], ...]` (numeric 2-tuples, not
named objects) and modified via `set_ignore_ranges` (full-replace
GET-then-PUT on the project). Independent of data-source mode.

**Mode switch.** `set_data_source_mode({mode})` flips
`project.data_source_mode` between `"manual" | "witsml" | "email"`.
Switching modes doesn't delete previously-uploaded data; it just
governs future ingestion behavior.

**WITSML mode — servers, crawl browsing, pollers.** Registering a WITSML
server is per-**user**, not per-project — once added it's available to
any of the user's projects. The UI's Active Well modal cascades through
Server → Well → Wellbore → (Trajectory | Log) → mnemonic pulldowns, all
of which are populated from a single `crawl` tree embedded in each
server's record. The agent exposes:

1. `add_witsml_server` — register a server with url/credentials/version.
2. `list_witsml_servers` — returns shallow summaries (no crawl) per server.
3. `browse_witsml_crawl` — walks the crawl tree by path, returning the
   next level of detail (wells / wellbores / trajectories+logs / curves).
4. `start_refresh_witsml_server` — re-crawl a server (~90 s); it returns a
   handle you poll with `get_operation_status` every ~15–20 s. The
   synchronous `refresh_witsml_server` is local-runtime only (§1.1) and can
   outrun a host's tool-call timeout even there.
5. `delete_witsml_server` — remove a server (per-user: it vanishes for
   every project at once, credentials and crawl gone for good). Warn
   first — project pollers still pointing at it are orphaned until
   re-pointed.

Then for the project's pollers (one each for trajectory and log):

6. `set_witsml_trajectory_poller` — upsert the trajectory poller.
   Supports two modes: `standard` (fetch a Trajectory object by
   `uid_trajectory`) or `continuous_inc_azi` (synthesize surveys from a
   Log's inclination + azimuth curves; for MWD services that don't
   report a separate Trajectory).
7. `set_witsml_log_poller` — upsert the GR log poller. Requires
   `uid_log` + `mnemonic`; optional `gamma_depth_mnemonic` when the
   gamma curve has its own depth channel.
8. `refresh_witsml_pollers` — one-shot fetch using the saved poller
   config. Returns 422 if the WITSML server is unreachable or the
   configured UIDs don't exist; surface the error rather than retry.

Pollers start `is_enabled: false` by default — they stay dormant until
explicitly flipped on. Flip by re-posting the same config with
`is_enabled: true`.
