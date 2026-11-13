<!-- published at /help/guide/setup/active-well.html on the Drive host -->

# Step 2 · Active Well

The second step loads the well being drilled: its survey trajectory and its LWD gamma-ray log, plus an optional well plan. The guided stage first asks whether you have a well plan: on a well still to be drilled, the plan lets Drive lay out the cross section and set the VS azimuth before the first survey arrives. The computation never uses it, so a resteer of a finished well can answer No.

Two status boxes at the top, **Trajectory** and **Active Log**, show whether each is loaded, with details and a delete control. While the well's data source is manual file upload or email they also hold the upload buttons.

The well has its own name, which you can rename with Write access to the well (it renames the well for every project that uses it). Several projects can use the same well and then share its survey, gamma log, well plan and reference elevation. Changing them needs Write access to the well, and changes them for every project on it; with Read access you can run the project but not change the well's data. Uploading a survey or log starts no run by itself; on a well that is receiving data, projects with Automatic runs on will run at the source's next delivery. A mailed file that would change the stations you uploaded will be refused and will start no run; the projects will run again with the first mail that only extends your upload. A file that only extends what is stored extends the well, for every project on it. One that changes stations already stored is a revision: when a project on the well has results, Drive asks you to confirm it. Projects on the well with Automatic runs on will then reset and rerun at the source's next delivery; the others will show that the well was revised, and wait for **Reset and Run**. With Admin access to the well, the Well access button beside its name shows who has access and which projects use it, and lets you add, change or remove a person.

## The well

The well panel at the top of this step shows the well as its own object: its name, who can access it, its data source and that source's status, the **Receiving data** switch, and the projects on the well. Each project reads **not run**, **current**, **behind** (the well has data past its last run) or **stale** (the well's data was revised after its last run), with whether its Automatic runs are on. You will see the names of the projects you can open, and a count of the others. The source's settings, its address and allowed senders need Write access to the well.

**Reference elevation** — the height of the active well's depth reference (KB or rig floor) above sea level. Enter it to work in subsea depths (TVDSS), which line up with subsea maps, structure files and interpretations. Leave it at 0 only if you don't have it or don't need subsea depths; depths then read below the rig (TVD). Set it before you align the logs: alignment centers on it, so changing it later means aligning again.

## Data sources

Drive takes active-well data three ways, one tab each. The data source belongs to the well: one source for every project on it. Looking at a tab changes nothing; the tab marked **Well's source** is the well's data source. To change it, open another tab and press its button, with Write access to the well. The change will keep the data already loaded and will turn **Receiving data** off: turn it on in the well panel once the source is set up.

**Receiving data** is the well's one switch for WITSML and email. While it is off, Drive will not poll the WITSML server, and mail sent to the well's address will not be stored on the well. Your own uploads will be stored either way. A pause will take effect at once, with no question, and will stop new data for every project on the well. To stop only this project from running by itself, turn off its [Automatic runs](./run-job.md#automatic-runs) instead.

On a well that only this project uses, the guided stage's source question is the same choice. On a shared well the question is locked, and the change is made here. A project cloned onto a new well, or imported, starts at manual file upload.

### Manual file upload

- **Upload Trajectory…** (in the Trajectory box) — a CSV with at least MD, inclination, and azimuth columns; the *Define Trajectory Input* dialog maps the columns. Each upload replaces the whole trajectory, so re-upload the full file as the well grows.
- **Upload LAS File…** (in the Active Log box) — the LWD gamma log; pick the gamma curve mnemonic in the *Define Log Curve Input: Active* dialog, and optionally tick **Apply despiking filter.** with a window length in samples. Also full-replace.
- **Upload Well Plan…** — optional CSV of the planned path. The plan is drawn on the cross section beyond the last survey station and is Drive's first source for the suggested VS azimuth and dip; it is not used by the computation.

### WITSML connection

While the well is receiving data, Drive polls a WITSML server every ten minutes for new trajectory and log data and stores it on the well; projects with Automatic runs on will extend as it arrives. Servers are registered on your [WITSML Servers page](../../admin/witsml.md); here, select the server, then drill down: well → wellbore → trajectory and log → curves.

Settings of note:

| Setting | Meaning |
|---|---|
| **Trajectory Source** | *Standard* fetches a WITSML Trajectory object. *Continuous Inc/Azi* synthesizes surveys from a log's inclination and azimuth curves — for MWD services that don't publish trajectory objects; you then pick that log and its inclination, azimuth, and (optionally) depth-index mnemonics. |
| **Survey Interval** | Regrid interval for continuous surveys; 0 means no regridding. |
| **Mnemonic** | The gamma-ray curve to ingest (optionally with a separate depth-index curve). |
| **Splice Overlap (ft)** | How each fetch joins the stored data: empty replaces it entirely, 0 appends, and a positive value replaces the stored data over that overlap. |

You can look at and save these settings under any data source, with Write access to the well and a WITSML server of your own; saving them starts no polling. Polling will start once WITSML is the well's data source and **Receiving data** is on. Press **Refresh Log and Trajectory** to fetch the latest data once, without running any job, whether or not the well is receiving; it needs WITSML to be the data source. If a well or curve you expect is missing from the pulldowns, refresh the server on the WITSML Servers page so Drive re-reads its well list.

### Email

Drive can receive data by email, at one address per well. Press **Generate Address** to create one for the well (or **Subscribe** to an address another well uses), then set the **Curve Mnemonic** to extract and, optionally, the **Allowed Senders** (leave it empty to accept mail from anyone), and press **Save Configuration**. To set the column layout of emailed surveys, use **Upload Initial Trajectory...** on this tab. These settings need Write access to the well. Drive will store mail in the name of the person who last saved them. While email is the well's data source and **Receiving data** is on, LAS attachments (logs) and CSV or Excel attachments (trajectories) sent to that address will be stored on the well. Mail that arrives while it is off will not be stored. Whether this project runs on new mail is its own [Automatic runs](./run-job.md#automatic-runs) switch.

## Ignore LWD ranges

If a stretch of the active log is unusable — a tool malfunction, an obvious spike — press **Ignore…** and define **Ignore Log Ranges**: MD intervals the computation skips without editing the raw log. Use ignore ranges for bad data; use tolerances (Step 5) for data that is merely noisy.
