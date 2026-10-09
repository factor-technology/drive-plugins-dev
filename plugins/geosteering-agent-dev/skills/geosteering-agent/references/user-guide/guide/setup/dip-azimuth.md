<!-- published at /help/guide/setup/dip-azimuth.html on the Drive host -->

# Step 3 · Dip and Azimuth

The computation needs a *prior* — an expectation, before seeing the gamma data, of how the target formation tilts along the lateral. This step provides it.

## VS Azimuth

The **VS Azimuth** is the plan-view direction of the vertical-section plane, in degrees. If unset, Drive computes it from the trajectory; you can override it.

::: warning Changing VS azimuth later
Apparent dip is measured in the vertical-section plane. Drive keeps the dip values you entered when the azimuth changes, so re-check that they still describe the structure in the new plane.
:::

## Dip type

Choose between two peer forms of the dip prior:

### Apparent dip

Piecewise-constant dip per MD range. Enter the dip in the directional-drilling convention: **90° is horizontal**; a bed dipping ~2° toward the toe reads as ~88° or ~92° depending on direction. Typical horizontal-play targets fall between roughly 70° and 110°.

The **Apparent Dip & Curvature** grid has one column per MD range. With one range, **Add range to vary laterally** splits it; with several, **Split Range** asks for the MD to split at (the new range copies the settings of the one it splits) and **Delete Range** removes the selected range. A range's **Start MD** can also be edited in the grid; the first range always starts at MD 0. The single-range case — one dip for the whole well — is the common one.

The grid also carries a **Curvature** row (° per 100 ft, or per 30 m), which bounds how fast dip may change laterally: **Unlimited**, or a tolerance from 15° down to 3°; see [Job Parameters](./job-parameters.md#per-range-parameters) for the details.

### Prior structure

A project-wide polyline of (VS, TVDSS) pairs describing the expected structure shape. Use it when a constant-dip prior is too crude — a known roll-over, a mapped flexure.

- **Structure…** uploads a CSV of VS/TVDSS pairs. Drive validates the file (sorted, no duplicates, matched columns) and reports problems.
- **Or use an existing interpretation…** snapshots the picks of a manual interpretation as the prior structure. The snapshot is a copy — later edits to the interpretation do not follow.

When a prior structure is set, per-range apparent dip does not apply.
