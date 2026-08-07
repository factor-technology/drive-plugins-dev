<!-- published at /help/guide/import-export.html on the Drive host -->

# Import and Export

Drive can move whole projects between systems, and export project data for use in other applications.

## Native archive

A **native archive** is a complete, portable copy of a project: settings, pilot wells and markers, active-well data, and interpretations.

- **Export** — from a project's **Inventory** tab, choose **Export to Native Archive**, which opens the export page and downloads the archive.
- **Import** — from the Projects page sidebar, choose **Import**, then supply an archive file. The project is recreated under your chosen owner and name.

Native archives are the right tool for backup, for moving a project between deployments, and for sending a complete project to support.

## CSV export

The CSV export page (reachable from a project) writes project data — trajectory, logs, and any interpretations you tick — into a single delimited file:

| Option | Meaning |
|---|---|
| **Resample interval** | The MD spacing rows are resampled onto. |
| **Delimiter** | Comma, tab, etc. |
| **Interpretations** | Checkboxes selecting which computed/manual interpretations to include as columns. |

Press **Download** to generate the file.

## Individual objects

The [Inventory tab](./inventory.md) lets you inspect and download individual objects — the active log, measured and planned trajectories, each pilot log, the computed interpretation, and each manual interpretation.
