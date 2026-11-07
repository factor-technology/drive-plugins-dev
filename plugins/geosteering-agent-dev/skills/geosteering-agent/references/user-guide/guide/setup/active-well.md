<!-- published at /help/guide/setup/active-well.html on the Drive host -->

# Step 2 · Active Well

The second step loads the well being drilled: its survey trajectory and its LWD gamma-ray log, plus an optional well plan. The guided stage first asks whether you have a well plan: on a well still to be drilled, the plan lets Drive lay out the cross section and set the VS azimuth before the first survey arrives. The computation never uses it, so a resteer of a finished well can answer No.

Two status boxes at the top, **Trajectory** and **Active Log**, show whether each is loaded, with details and a delete control. While manual file upload is the project's data source they also hold the upload buttons.

The well has its own name, which you can rename with Write access to the well (it renames the well for every project that uses it). Several projects can use the same well and then share its survey, gamma log, well plan and reference elevation. Changing them needs Write access to the well, and changes them for every project on it; with Read access you can run the project but not change the well's data. Uploading a survey or log starts no run by itself. A file that only extends what is stored extends every project as usual. One that changes stations already stored is a revision: when a project on the well has results, Drive asks you to confirm it, and those projects reset and rerun from the start. With Admin access to the well, the Well access button beside its name shows who has access and which projects use it, and lets you add, change or remove a person.

**Reference elevation** — the height of the active well's depth reference (KB or rig floor) above sea level. Enter it to work in subsea depths (TVDSS), which line up with subsea maps, structure files and interpretations. Leave it at 0 only if you don't have it or don't need subsea depths; depths then read below the rig (TVD). Set it before you align the logs: alignment centers on it, so changing it later means aligning again.

## Data sources

Drive takes active-well data three ways, one tab each. Looking at a tab changes nothing: the tab marked **In force** is this project's data source. To change it, open another tab and press its button (**Use WITSML for this project**, and so on), then confirm.

The change keeps the data you already loaded. Leaving WITSML pauses this project's pollers. Leaving email stops automatic runs on emailed data; files sent to the project's address still update the well. Entering a source starts nothing: WITSML polling stays paused until you resume it, and email runs stay paused until the project has run.

The guided stage's source question is the same change: answering it sets the data source. A cloned or imported project starts at manual file upload.

### Manual file upload

- **Upload Trajectory…** (in the Trajectory box) — a CSV with at least MD, inclination, and azimuth columns; the *Define Trajectory Input* dialog maps the columns. Each upload replaces the whole trajectory, so re-upload the full file as the well grows.
- **Upload LAS File…** (in the Active Log box) — the LWD gamma log; pick the gamma curve mnemonic in the *Define Log Curve Input: Active* dialog, and optionally tick **Apply despiking filter.** with a window length in samples. Also full-replace.
- **Upload Well Plan…** — optional CSV of the planned path. The plan is drawn on the cross section beyond the last survey station and is Drive's first source for the suggested VS azimuth and dip; it is not used by the computation.

### WITSML connection

Drive polls a WITSML server every ten minutes for new trajectory and log data and extends the interpretation as data arrives. Servers are registered on your [WITSML Servers page](../../admin/witsml.md); here, select the server, then drill down: well → wellbore → trajectory and log → curves.

Settings of note:

| Setting | Meaning |
|---|---|
| **Trajectory Source** | *Standard* fetches a WITSML Trajectory object. *Continuous Inc/Azi* synthesizes surveys from a log's inclination and azimuth curves — for MWD services that don't publish trajectory objects; you then pick that log and its inclination, azimuth, and (optionally) depth-index mnemonics. |
| **Survey Interval** | Regrid interval for continuous surveys; 0 means no regridding. |
| **Mnemonic** | The gamma-ray curve to ingest (optionally with a separate depth-index curve). |
| **Splice Overlap (ft)** | How each fetch joins the stored data: empty replaces it entirely, 0 appends, and a positive value replaces the stored data over that overlap. |

You can look at and save these settings while another data source is in force, and they stay paused. Polling starts paused; start it on the [Run Job step](./run-job.md#live-updates). **Refresh Log and Trajectory** fetches the latest data once, without running the job; it and resuming need WITSML to be the data source. If a well or curve you expect is missing from the pulldowns, refresh the server on the WITSML Servers page so Drive re-reads its well list.

### Email

Drive can receive data by email. **Generate Address** creates a project email address (or **Subscribe** to an existing one), then set the **Curve Mnemonic** to extract and, optionally, the **Allowed Senders** — leave it empty to accept mail from anyone — and press **Save Configuration**. LAS attachments (logs) and CSV or Excel attachments (trajectories) sent to that address are ingested automatically and the interpretation extends. **Upload Initial Trajectory...** seeds the column layout. While email is in force, its automatic runs can be paused and resumed here or on the Run Job step. Files sent to the address update the well whether or not automatic runs are on, and whatever the project's data source is.

## Ignore LWD ranges

If a stretch of the active log is unusable — a tool malfunction, an obvious spike — press **Ignore…** and define **Ignore Log Ranges**: MD intervals the computation skips without editing the raw log. Use ignore ranges for bad data; use tolerances (Step 5) for data that is merely noisy.
