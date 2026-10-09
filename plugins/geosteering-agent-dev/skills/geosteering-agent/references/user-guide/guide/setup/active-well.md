<!-- published at /help/guide/setup/active-well.html on the Drive host -->

# Step 2 · Active Well

The second step loads the well being drilled: its survey trajectory and its LWD gamma-ray log, plus an optional well plan.

Two status boxes at the top, **Trajectory** and **Active Log**, show whether each is loaded, with details and a delete control. In Manual mode they also hold the upload buttons.

**Reference elevation** — optionally set the active well's datum elevation. Leave it at 0 to work in plain TVD; set it if you want depths referenced subsea (TVDSS), for example to compare against a regional structure map.

## Data sources

Drive ingests active-well data three ways; pick a tab. Switching tabs keeps the data you already loaded, but it changes what is live: leaving the WITSML tab turns its pollers off, and entering the Email tab turns email ingestion on.

### Manual file upload

- **Upload Trajectory…** (in the Trajectory box) — a CSV with at least MD, inclination, and azimuth columns; the *Define Trajectory Input* dialog maps the columns. Each upload replaces the whole trajectory, so re-upload the full file as the well grows.
- **Upload LAS File…** (in the Active Log box) — the LWD gamma log; pick the gamma curve mnemonic in the *Define Log Curve Input: Active* dialog, and optionally tick **Apply despiking filter.** with a window length in samples. Also full-replace.
- **Upload Well Plan…** — optional CSV of the planned path. The plan is a visual aid on the cross section; it is not used by the computation.

### WITSML connection

Drive polls a WITSML server every ten minutes for new trajectory and log data and extends the interpretation as data arrives. Servers are registered on your [WITSML Servers page](../../admin/witsml.md); here, select the server, then drill down: well → wellbore → trajectory and log → curves.

Settings of note:

| Setting | Meaning |
|---|---|
| **Trajectory Source** | *Standard* fetches a WITSML Trajectory object. *Continuous Inc/Azi* synthesizes surveys from a log's inclination and azimuth curves — for MWD services that don't publish trajectory objects; you then pick that log and its inclination, azimuth, and (optionally) depth-index mnemonics. |
| **Survey Interval** | Regrid interval for continuous surveys; 0 means no regridding. |
| **Mnemonic** | The gamma-ray curve to ingest (optionally with a separate depth-index curve). |
| **Splice Overlap (ft)** | How each fetch joins the stored data: empty replaces it entirely, 0 appends, and a positive value replaces the stored data over that overlap. |

Polling starts paused; start it on the [Run Job step](./run-job.md#live-updates). **Refresh Log and Trajectory** fetches the latest data once, without running the job. If a well or curve you expect is missing from the pulldowns, refresh the server on the WITSML Servers page so Drive re-reads its well list.

### Email

Drive can receive data by email. **Generate Address** creates a project email address (or **Subscribe** to an existing one), then set the **Curve Mnemonic** to extract and, optionally, the **Allowed Senders** — leave it empty to accept mail from anyone — and press **Save Configuration**. LAS attachments (logs) and CSV or Excel attachments (trajectories) sent to that address are ingested automatically and the interpretation extends. **Upload Initial Trajectory...** seeds the column layout. Email updates can be paused and resumed here or on the Run Job step.

## Ignore LWD ranges

If a stretch of the active log is unusable — a tool malfunction, an obvious spike — press **Ignore…** and define **Ignore Log Ranges**: MD intervals the computation skips without editing the raw log. Use ignore ranges for bad data; use tolerances (Step 5) for data that is merely noisy.
