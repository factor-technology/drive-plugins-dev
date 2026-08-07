<!-- published at /help/guide/setup/active-well.html on the Drive host -->

# Step 2 · Active Well

The second step loads the well being drilled: its survey trajectory and its LWD gamma-ray log, plus an optional well plan.

The **Active Well Data** status boxes at the top show, at a glance, whether a trajectory and an active log are loaded, with details and a delete control for each.

**Reference elevation** — optionally set the active well's datum elevation. Leave it at 0 to work in plain TVD; set it if you want depths referenced subsea (TVDSS), for example to compare against a regional structure map.

## Data sources

Drive ingests active-well data three ways; pick a tab. Switching modes does not delete data you already loaded.

### Manual file upload

- **Upload Trajectory…** — CSV or Excel with at least MD, inclination, and azimuth columns; the *Define Trajectory Input* dialog maps the columns. Each upload replaces the whole trajectory, so re-upload the full file as the well grows.
- **Upload LAS File…** — the LWD gamma log; pick the gamma curve mnemonic in the *Define Log Curve Input: Active* dialog. Also full-replace.
- **Upload Well Plan…** — optional CSV of the planned path. The plan is a visual aid on the cross section; it is not used by the computation.

### WITSML connection

Drive polls a WITSML server for new trajectory and log data and extends the interpretation automatically as data arrives. In the *WITSML Connection* dialog, select a server (or add one — see [WITSML Servers and Polling](../../admin/witsml.md)), then drill down: well → wellbore → trajectory and log → curves.

Settings of note:

| Setting | Meaning |
|---|---|
| **Trajectory type** | *Standard* fetches a WITSML Trajectory object. *Continuous Inc-Azi* synthesizes surveys from a log's inclination and azimuth curves — for MWD services that don't publish trajectory objects. |
| **Survey Interval** | Regrid interval for continuous surveys; 0 means no regridding. |
| **Mnemonic** | The gamma-ray curve to ingest (optionally with a separate depth channel). |
| **Splice Overlap** | Overlap, in feet, used when splicing successive log fetches. |

If a well or curve you expect is missing from the pulldowns, refresh the server so Drive re-reads its well list.

### Email

Drive can receive data by email. Generate a project email address (or subscribe to an existing one), set the **Curve Mnemonic** to extract, and list the **Allowed Senders**. Attachments (LAS for logs, CSV/Excel for trajectories) sent to that address from an allowed sender are ingested automatically and the interpretation extends. Optionally upload an initial trajectory to seed the column layout.

## Ignore LWD ranges

If a stretch of the active log is unusable — a tool malfunction, an obvious spike — define **Ignore Log Ranges**: MD intervals the computation skips without editing the raw log. Use ignore ranges for bad data; use tolerances (Step 5) for data that is merely noisy.
