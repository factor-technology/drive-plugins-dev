<!-- published at /help/guide/setup/pilot-well.html on the Drive host -->

# Step 1 · Pilot Well

The first wizard step defines the reference data the computation matches against: one or more pilot wells, each with a vertical gamma-ray log and a set of formation tops.

Throughout the wizard, the right-hand track shows the pilot type log (and, once loaded, the active log) so you can see the effect of your edits immediately.

## Pilot log ranges

The **Pilot Log Ranges** grid shows one column per pilot well, each covering a contiguous MD range of the lateral. A single pilot starting at MD 0 is the common case; add further ranges only when the geology changes enough along the lateral to warrant a second reference log — see [Key Concepts](../concepts.md#multiple-pilot-wells) for the rules multi-pilot projects must observe.

- **Upload LAS…** (**Replace LAS…** once a log is loaded) opens the *Upload Pilot Log* dialog for that range.
- **Add Range…** opens the same dialog as *Add Pilot Range*, with a **Start MD** field for the new range.
- **Delete** removes a range; the first range cannot be deleted. A range with no log of its own is ignored by the computation.

The dialog takes everything in one place. Its **Upload** button stays disabled, with a hint naming what is still missing, until the dialog is complete, so a log cannot go in without its survey, nor a survey without its log.

- **Pilot log (LAS).** Choose the LAS file. The first channel is taken as measured depth. Pick the **gamma-ray curve** from the list that appears; Drive preselects the curve the project used last, or the one gamma-ray-looking mnemonic.
- **Well path.** Leave **Vertical well** selected when the LAS depth index is already true vertical depth. Choose **Deviated well** when the log came from a deviated hole: the survey CSV picker and its column designation appear right below, and Upload waits until the MD, inclination and azimuth columns are designated. Drive straightens the log once at upload and does not keep the survey.

A pilot uploaded as deviated remembers that. When you later **Replace LAS…** on it, the dialog opens on **Deviated well** and asks for the survey again, since only the straightened log was kept.

Pilot wells have no reference elevation — the stored log is in plain type-log TVD (TVDTL), positive and measured down.

## Formation tops

The **Formation Tops** grid holds the named tops for every pilot:

| Column | Meaning |
|---|---|
| *(color)* | Display color of the formation on the cross section. |
| **Formation Name** | The top's name. On multi-pilot projects, every name on the first pilot must appear on all pilots. |
| **Depth (TVDTL)** | The top's depth in **TVDTL** — positive, measured down, in the project's spatial unit. With several pilots there is one depth column per range, headed by its MD span; only ranges with a log of their own are editable. |
| **Target** | Tick exactly one row: the formation the well is steering in. This sets the project's **top of target**. |

Toolbar buttons: **Add Top**, **Delete Top**, **Undo** (Ctrl+Z), **Redo** (Ctrl+Y). The grid accepts paste from a spreadsheet — columns in the order name, one depth per pilot, target — so you can prepare tops in Excel and paste the block in. Edits save as you make them.

::: warning Mark the target
The computation solves for the top of the target formation only. The step is not complete until one top is ticked as Target.
:::
