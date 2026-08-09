<!-- §1.9.1 Pilot Wells — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.1 Pilot Wells

A **pilot well** is an offset well — located some distance away from the
**active well** being geosteered. It serves as a reference / type-log for
the computation's calibration. Each pilot well carries:

- A vertical gamma ray (GR) log, stored in positive-down pilot TVD.
- An MD range over which it applies.

A project always has at least one pilot well. The first one starts at
**MD 0**. Most projects have just one, but multiple are supported across
the lateral, each covering a contiguous MD range. APIs expose these MD
ranges.

**Log storage and straightening.** Logs are **stored as vertical**. If the
source log was acquired from a deviated well, the user supplies a
trajectory at ingest; the server uses it to "straighten" the log to
vertical, then **the trajectory is discarded**. Only the resulting
vertical log is part of the persistent pilot well.

**File formats — your responsibility in chat.** Drive's HTTP API serves
and accepts custom JSON for both logs and trajectories. Real users have:

- **LAS files** for logs. The web UI uses `las-lib` to convert LAS → JSON.
  Working from chat, you do the equivalent.
- **CSV (or Excel) files** for trajectories. You convert to trajectory
  JSON before delivering to the API.

**LAS — collect the GR mnemonic.** LAS files typically have several
channels with mnemonic names (one or more is gamma ray). Assume the
**first channel is measured depth**. You must collect the **GR channel
mnemonic** to use from the user. Useful pattern: parse the file, list
the available mnemonics with brief descriptions if present in the LAS
header, let the user pick.

**Verticalization trajectory — prompt as optional.** If the source log
was acquired from a deviated well, Drive uses a trajectory CSV to
straighten the log to vertical at ingest (and discards the trajectory
after). Ask the user **once** whether they have such a trajectory
file. If they don't — the common case, since pilot source wells are
typically vertical — proceed with an empty trajectory and no
transformation is applied. When the user provides one, parse it like
any other trajectory CSV (collect column indexes per the rule below)
and pass via `trajectory_waypoints` (default `[]`).

**CSV/Excel trajectories — collect the column indexes.** You must collect
the **column numbers** for measured depth, inclination, and azimuth. Useful
pattern: parse the file, display the column headers (or first row of data
if there are no headers), let the user pick.

When a user says "upload a pilot log from /path/to/foo.las", do **not**
just shovel bytes — parse, present choices, get the selections, convert,
then call the upload endpoint.

**Getting the samples in — branch on size (§1.1).** The tools cannot read the
user's disk, so the file's bytes have to reach the server one of two ways:

- **Small file** (a short channel, a few tens of KB): `read_las` the
  file's text for the channel list, call it again with the chosen mnemonic for
  the samples, and pass those samples to the upload tool.
- **Large file** (a full pilot GR channel, hundreds of KB — the usual
  case): mint an upload URL with `create_upload`, run the returned curl from
  code execution to send the raw bytes, then call the upload tool with the
  returned upload id. Send the file **unmodified** — the server parses it with
  the same LAS reader as everything else, so wrapped files, `-999.25` nulls,
  and the units row are already handled; reformatting only introduces errors.
  The send answers with the channel list, which is what you confirm the GR
  mnemonic against. Both deadlines are recoverable: the file is still on disk,
  so mint again and re-send, with nothing for the user to do.

If the send fails because the host blocks outbound network access from code
execution, that is not a Drive fault and not something you can retry around:
tell the user their organization has to allow the Drive site from code
execution, and offer the small-file path meanwhile.

**Upload sequence — three calls behind one tool.** The wizard's "upload"
button performs three API calls; `upload_pilot_log` (§1.10) bundles them:

1. `POST /pilot-wells/{wellID}/logs/GR` — body carries `log_curve` (the
   converted MD-indexed GR samples plus index/value uoms and `mnemonic`)
   and `trajectory.waypoints` (empty for already-vertical sources).
   Before the POST the tool runs the wizard's preprocessing:
   ascending/monotonic MD, NaN/null drop, median despike (project
   `despike_window` unless the arg overrides; 1 disables), index-UOM
   conversion into the project's spatial units, and gap interpolation:
   the sample interval is detected as the most frequent consecutive
   spacing (an omitted `sample_interval` arg uses it) and gaps are
   linearly interpolated onto that grid. The server stores the log as
   positive-down pilot TVD.
2. `PUT /pilot-wells/{wellID}` — reads the well first, then sets its
   display `name`, `start_md`, `las_file_name`. On a replace it
   PRESERVES tuned `name` / `start_md` / `auto_fit_range` /
   `fit_params`; the wizard defaults — `auto_fit_range: {top: 0,
   bottom: 200}` (positive pilot TVD) and identity `fit_params:
   {datum_tvdss: 0, gr_offset: 0, gr_scale: 1}` (the calibration
   baseline) — fill only fields a genuinely new well never had.
   `reset_fit: true` opts back into stamping the defaults, discarding
   the tuned calibration — state-invalidating per §1.5.5, requiring a
   job reset.
3. `PUT /projects/{name}` (full-replace — GET the project, mutate, PUT it
   back) — sets `pilot_log_curve_mnemonic`; `param_blocks` round-trip
   untouched (the wizard never touches them here).

`upload_pilot_log` writes into a **pre-existing** pilot-well row — it
does not create the row. The wizard's "Pilot Well" step uses
`create_pilot_well` (a separate `POST /pilot-wells`) to make the row
first; project create does not auto-create one. Typical flow from chat:

1. `create_pilot_well({project_id, name?, start_md?})` → `{id}`
2. `upload_pilot_log({..., pilot_well_id: id})`
3. `set_pilot_well_markers({..., markers: [...]})` to define formation tops
4. `set_top_of_target({project_id, marker_name})` to designate which marker is the ToT

Use `list_pilot_wells` to discover existing wells (e.g. on a re-upload
after the user already created the well via the UI). Range CRUD lives
on the UI's Pilot Log Ranges grid: `rename_pilot_well` edits the
display name only (safe, non-destructive metadata), and
`delete_pilot_well` destroys a range's log and markers — irreversible,
and refused for the first (MD 0) range, which defines the project TVD
frame.

**No pilot datum elevation.** Pilot wells carry no reference elevation —
do not ask for one, and do not send one. The pilot log lives in its own
positive-down TVD; `fit_params.datum_tvdss` is the single bridge into the
world frame (`TVDSS = datum_tvdss − pilotTvd`).

**Markers (formation tops).** Each pilot well carries a list of named
depth picks: `{name, color, index_domain, depth_index}`. Pilot wells are
vertical and `index_domain` is `"TVD"`. Marker storage is
**full-replace** — every PUT carries the entire desired list; there's no
per-marker patch/delete endpoint. Markers have no server-side ids;
they're identified by `name` within the list.

**Coordinate convention — no translation.** The user speaks **Type Log
TVD**: positive feet downward, the very numbers in the pilot-well grid
(a top at 14,385 ft means 14,385). `depth_index` stores exactly that
number — write and read it as-is, with `index_domain: "TVD"`. Never
negate in either direction.

**Top of target — project-level pointer, not a marker attribute.** The
project carries a `tot_name` field that names one of the pilot-well
markers as the computation's top-of-target horizon. This is set/cleared via
`set_top_of_target` (which does a GET-then-PUT on the project). Drive
does NOT validate that the named marker exists — keep the name in sync
with the marker list. **There is no base-of-target concept** — the
computation only models top-of-target.

**Multi-pilot semantics (user-enforced invariants).** When a project has
more than one pilot well:

- The **first pilot (MD 0) defines the project TVD coordinate system**;
  all other pilots share it. Non-first pilots tell you nothing about
  structure — only ToT structure is computed.
- **ToT must be flattened across pilots**: the user must enter the
  **same type-log TVD depth** for the ToT marker on every pilot well. The UI
  does not enforce this — when you see/edit pilot markers, verify and
  flag mismatches.
- **Same marker set on every pilot**: every formation on the first
  pilot must appear (by `name`) on every other pilot. UI does not
  enforce this either; flag missing/extra markers.
- **Inter-pilot interpolation**: between two adjacent pilots Drive
  builds a pseudo-type-log by warping each layer's depth axis to match
  the bracketing pilots' thicknesses and linearly interpolating the GR
  samples within each segment.
- Interpolation is currently in **MD space** (should be VS/X-Y; future
  work). Workaround for high-fidelity early-lateral: insert a duplicate
  of the MD 0 pilot at the lateral's start MD, so MD ≈ VS over the
  interesting range.
