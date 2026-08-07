<!-- published at /help/guide/setup/dip-azimuth.html on the Drive host -->

# Step 3 · Dip and Azimuth

The computation needs a *prior* — an expectation, before seeing the gamma data, of how the target formation tilts along the lateral. This step provides it.

## VS Azimuth

The **VS Azimuth** is the plan-view direction of the vertical-section plane, in degrees. If unset, Drive computes it from the trajectory; you can override it.

::: warning Changing VS azimuth later
Apparent dip is measured in the vertical-section plane, so previously entered dip values become stale when the azimuth changes. Re-check your dips after changing VS azimuth.
:::

## Dip type

Choose between two peer forms of the dip prior:

### Apparent dip

Piecewise-constant dip per MD range. Enter the dip in the directional-drilling convention: **90° is flat-lying**; a bed dipping ~2° toward the toe reads as ~88° or ~92° depending on direction. Typical horizontal-play targets fall between roughly 70° and 110°.

The per-range grid has one column per MD range. Add or remove ranges with the buttons above the grid (*Add New MD Range* asks for the start MD). The single-range case — one dip for the whole well — is the common one.

The grid also carries a **Curvature** row, which bounds how fast dip may change laterally; see [Job Parameters](./job-parameters.md#curvature) for the details.

### Prior structure

A project-wide polyline of (VS, TVDSS) pairs describing the expected structure shape. Use it when a constant-dip prior is too crude — a known roll-over, a mapped flexure.

- **Structure…** uploads a CSV of VS/TVDSS pairs. Drive validates the file (sorted, no duplicates, matched columns) and reports problems.
- **Or use an existing interpretation…** snapshots the picks of a manual interpretation as the prior structure. The snapshot is a copy — later edits to the interpretation do not follow.

When a prior structure is set, per-range apparent dip does not apply.
