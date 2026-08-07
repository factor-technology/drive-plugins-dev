<!-- published at /help/guide/getting-started.html on the Drive host -->

# Your First Project

This walkthrough takes a new project from creation to a computed interpretation. Each step links to the full reference page for that part of the app.

## What you need

- A **pilot (offset) well log** as a LAS file containing a gamma-ray curve, plus the formation top depths for that well.
- The **active well's trajectory** (CSV or Excel with MD, inclination, azimuth) and its **LWD gamma log** (LAS) — or a WITSML server that publishes them, or a data provider who can email them.
- The **vertical-section azimuth** and a rough idea of the formation dip (or a prior structure).

## 1 · Create the project

On the [Projects page](./projects-page.md), choose **Add Project**. Pick an owner (yourself, or a group you can write to), a unique name, the spatial unit (feet or meters), and a setup style:

- **Full** — the six-step wizard described below. Recommended.
- **Quick** — a single page: upload three files and run. Good for a fast trial.

## 2 · Pilot well

In [Setup step 1](./setup/pilot-well.md), upload the pilot's LAS file and pick the gamma-ray curve. Then fill in the **Formation Tops** grid: name, color, and depth (type-log TVD, positive down) for each top, and tick **Target** on the one the well is steering in. You can paste tops straight from a spreadsheet.

## 3 · Active well

In [Setup step 2](./setup/active-well.md), choose a data source:

- **Manual File Upload** — upload the trajectory and the LAS gamma log (and optionally a well plan).
- **WITSML Connection** — pick a server, well, wellbore, trajectory, and log curve; Drive polls for new data automatically.
- **Email** — generate a project email address and have your data provider send LAS/Excel attachments to it.

## 4 · Dip and azimuth

In [Setup step 3](./setup/dip-azimuth.md), accept or set the **VS Azimuth**, then give the computation its dip prior: either an **apparent dip** (90° = flat; use your best estimate from offset data) or an uploaded **prior structure** polyline.

## 5 · Align logs

In [Setup step 4](./setup/align-logs.md), press **Auto-align Logs**. Drive finds the vertical shift (and gamma scale/offset) that best maps the pilot log onto the active log, corroborated across multiple depth windows, and sets the first MD to compute. Review the result visually — alignment is the calibration everything else rests on. You can also set the fit parameters by hand.

## 6 · Job parameters

[Setup step 5](./setup/job-parameters.md) shows the tolerances the computation balances — log tolerance, dip tolerance, curvature, faults, resolution. The defaults are sensible; for a first run, leave them alone.

## 7 · Run

In [Setup step 6](./setup/run-job.md), check the readiness message, then press **Run**. Drive navigates to the Profile tab and shows progress in the tab bar. When the job finishes, the computed interpretation appears on the cross section.

## 8 · Read the results

On the [Profile tab](./profile.md), open **Interpretations** and switch between the **MPE** and the alternative explanations; open **Marginals** to see the per-depth probability field. [Understanding the Results](./results.md) explains what you are looking at.

As drilling progresses, new data extends the interpretation: WITSML polling and email ingestion trigger extensions automatically, and manual uploads followed by **Extend** do the same.
