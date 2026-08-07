<!-- §1.9.3 Dip & Azimuth — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

### 1.9.3 Dip & Azimuth

Step 3 of the wizard configures the **vertical-section azimuth** (the
direction the lateral is drilled in plan view) and the **dip prior**
(how the formation tilts, as seen by the computation). Two dip modes,
selected by `project.dip_type`:
- `"constants"`: constant-dip piecewise along the lateral. Each
  `param_block` carries its own `dip_constant` (true-dip degrees) that
  applies from that block's MD up to the next block's MD (or Infinity
  for the last block). Common case: one block at `md=0` covering the
  whole well.
- `"structure"`: a project-wide 2D (vs, tvdss) polyline uploaded
  separately. `dip_constant` is irrelevant in this mode and the tools
  clear it to null across every block when the mode flips.

**Offer both dip-prior modes as peers.** When collecting Step 3 inputs,
present the two modes — constant apparent dip vs. structure polyline
(CSV of vs/tvdss pairs) — as equal options, not primary + footnote.
Many users have an offset-well interpretation they'd rather upload
than reduce to a single number; don't bury the polyline path in a
parenthetical. If the project already has saved interpretations, also
offer to snapshot one as the prior structure (see "Structure can be
sourced from an existing interpretation" below).

**`vs_azimuth` is project-level.** Stored at the project root.
`strike_constant` is stored per-block but semantically follows
vs_azimuth (`strike = vs_azimuth - 90`), so `set_vs_azimuth`
propagates the value to every block.

**Apparent dip convention.** The Drive form and the `set_apparent_dip`
tool both accept apparent dip in **trajectory-inclination coordinates**:
vertical = 0°, horizontal = 90°. This is the directional-drilling
industry convention; a flat-lying target has apparent dip ≈ 90°, a
2°-tilted bed ≈ 88°. **Do not use the textbook geological convention**
(0° = horizontal, 90° = vertical) — they are complements, and flipping
them inverts every sanity check. Typical hydrocarbon targets in
horizontal plays present apparent dips of **70° to 110°**; only
sanity-check with the user when the value falls outside that range.

Internally Drive stores `dip_constant` in a different (rotated) frame.
`set_apparent_dip` performs the conversion via the `getTrueDip` helper
ported from the UI. **Pass user-supplied apparent-dip values to
`set_apparent_dip` as-is, in the user's convention; never pre-convert,
and never reproduce the conversion math elsewhere.**

**Apparent dip vs true dip.** Geologists read *apparent* dip off the
vertical-section view. The computation wants *true* dip. Conversion
depends on the angle between section azimuth and strike (ported from
the UI's `getTrueDip` helper verbatim). A subtle consequence: a stored
`dip_constant` is true-dip, but it was computed from some prior
*apparent* dip under the then-current vs_azimuth. Changing vs_azimuth
leaves the stored dip_constant arithmetically valid but semantically
stale — so by default `set_vs_azimuth` rewrites every block's
dip_constant in the same PUT to preserve its apparent dip under the
new azimuth, exactly as the UI does; no re-entry is needed. Only with
`preserve_apparent_dips=false` does it leave dip_constant untouched
and instead flag the stale blocks, and the agent prompts the user to
re-enter apparent dip via `set_apparent_dip` on each.

**Block CRUD lives in Step 5.** Param blocks are the tuning surface
(§1.5) shared with JobParametersStep — they carry `dip_sigma`,
`log_sigma`, `cull_factor`, `fault_*`, etc., not just dip. Adding,
deleting, updating blocks is a Step 5 responsibility; Step 3 only
edits `vs_azimuth` and per-block `dip_constant` on blocks that
already exist.

**Structure polyline endpoints are orthogonal to `dip_type`.** The
server stores `vs`/`tvdss` regardless of mode; the computation only
consults them when `dip_type="structure"`. `delete_structure` does NOT
change `dip_type`. To move back to constants mode, pair
`delete_structure` with `set_apparent_dip` (which implicitly flips
`dip_type="constants"`).

**Structure can be sourced from an existing interpretation.** In the UI,
the Dip & Azimuth step offers (alongside CSV upload) a chooser to
snapshot any project interpretation's picks as the prior structure —
MD-keyed picks are converted to VS using the active trajectory
(`set_structure_from_interpretation` is the same gesture as a tool). The
snapshot does not stay linked: edits to the source interpretation do
not propagate. Common workflow: import a foreign interpretation from
another geosteering system on an offset well via the Interpretation
InfoBar's "Import interpretation" button (the upload icon), then in
Dip & Azimuth pick that interpretation from the chooser to promote it
to the prior structure here.

Tools:
1. `set_vs_azimuth({project_id, vs_azimuth_deg, preserve_apparent_dips?})` —
   project-level azimuth; propagates `strike_constant = vs_azimuth - 90`
   to every block and by default rewrites each block's `dip_constant` so
   its apparent dip is preserved under the new azimuth (UI parity — no
   re-entry needed). `preserve_apparent_dips=false` instead leaves
   dip_constant untouched and lists the stale blocks for re-entry via
   `set_apparent_dip`. In both modes each block's params are first
   gap-filled with the per-range defaults (stored values win), as the
   UI's same PUT does.
2. `set_apparent_dip({project_id, apparent_dip_deg, block_md?=0})` —
   sets `dip_type="constants"` and writes `dip_constant` on the block
   at `block_md` (via `getTrueDip` using current vs_azimuth + strike).
   Errors if no block exists at `block_md`; use `add_param_block`
   (Step 5) first for multi-range setups. `apparent_dip_deg` is required
   on every call; this is write-only, so confirm a dip with
   `read_job_params` (per-block `dip_constant`) / `read_project`
   (`dip_type`), never by re-calling this tool.
3. `set_dip_structure_mode(project_id)` — sets `dip_type="structure"`
   and clears `dip_constant` on every block. Pair with
   `upload_structure` for the polyline.
4. `upload_structure(project_id, vs[], tvdss[])` — full-replace PUT.
   `vs` strictly ascending, no duplicates; arrays same length; ≥1 pt.
5. `set_structure_from_interpretation({project_id, interpretation_name})` —
   snapshot a saved interpretation's picks as the prior structure (see
   above). Pair with `set_dip_structure_mode` when `dip_type` isn't
   already `"structure"`.
6. `read_structure(project_id)` — returns `{vs, tvdss}` or null.
7. `delete_structure(project_id)` — removes the stored polyline.
