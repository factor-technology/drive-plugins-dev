# Pane-by-pane control inventory

Every control on the Profile tab's menu panes and log-track dialogs, with
what it does and when to touch it. Panes open from the menu strip (Scene,
Components, Zoom, Target, Marginals, Interpretations, Formations, Traces,
Derived, Help) as white cards over the top-left of the scene. Only one pane
opens at a time; the same menu item closes it. Checkbox/radio labels below
match the UI exactly. Greyed = the underlying data doesn't exist.

## Scene

Overall display frame.

- **LATERAL DOMAIN** dropdown: `MD` or `VS` — the horizontal axis of the
  section. MD is the default working view; VS matters when the user thinks
  in vertical-section distance.
- Checkboxes: **active log** (the horizontal GR strip along the top),
  **type log** (the vertical track at the left edge), **correlations**,
  **cursor info** (a floating readout box — turn OFF for screenshots),
  **legend** (line-style key — OFF for screenshots unless the audience
  needs it), **name** (the project name card in the corner — usually ON, it
  labels the image).
- **DEPTH AXIS** radios: `TVDSS` (subsea, negative below sea level — the
  frame the computed structure lives in) / `TVDTL` (type log TVD — the pilot
  well's own depth, positive down, the frame formation tops are entered in) /
  `TVD` (the active well's own depth, positive down). TVDSS is the common
  default. TVDTL is Drive's own abbreviation and new to users — spell it out
  as "type log TVD (TVDTL)" the first time you use it in a reply.

## Components

Show/hide each scene element.

- **wellbore** — the drilled trajectory. Nearly always on.
- **well plan** — the planned path. On if a plan exists.
- **projected trajectory** — the projection ahead of the bit. On while
  drilling; off for finished wells.
- **pilot wells** — pilot well sticks/markers. Usually on.
- **interpolated type logs** — the trace overlay (same toggle as the Traces
  pane's "display"). See Traces below.
- **formations** — formation top lines/bands. Usually on.
- **prior structure** — the user's prior model. Show when there's no
  computed structure, or when comparing prior vs computed is the point.
- **background image**: **display** + **crop** checkboxes and an
  **opacity** slider (10–100%) — the seismic/background image uploaded on
  the Background tab. Greyed if none uploaded. Crop limits it to its
  registered extent. Lower the opacity when the image competes with the
  marginals or the interpretation for the eye; 100% is the default.

## Zoom

Framing. All three values are click-to-edit with −/+ steppers.

- **SCALE** — the overall zoom −/+ buttons (no number): + converges on
  the wellbore, − keeps the view centered.
- **V EXAGGERATION** — vertical exaggeration multiplier (e.g.
  "8.0 ×"). The single most important number for a good-looking section.
- **HORIZONTAL** — horizontal scale as real-world units per screen inch
  ("1 in = 200 ft"); editing it preserves VE. Its −/+ steppers stretch
  the horizontal axis only.
- **Toe** button — recenters the viewport on the toe. The escape hatch
  when the scene is lost.
- **Reset** button — reset the scale values to defaults (1 in = 500 ft,
  or 150 m on SI projects; VE 8), keeping the view centered where it is.

Wheel equivalents: wheel = vertical scroll, shift+wheel = horizontal,
ctrl/⌘+wheel = zoom. Keys: ctrl/⌘ + `+`/`−`.

## Target

The target line and drilling window.

- **HORIZON EXTENSION**: **display** checkbox + **ENDING VS** numeric field
  — extends the target horizon out to a chosen VS. Off by default.
- **TARGET LINE**: a validity indicator (green check when the line is
  well-formed), **display** checkbox, **extend** checkbox (greyed until the
  well azimuth and a measured toe exist — arming it turns on click-to-extend
  picking on the canvas: leave OFF unless editing), a trash button
  (deletes the target line — never touch without explicit instruction),
  **annotate** checkbox + color swatch (labels the line; the swatch opens a
  color picker).
- **DRILLING WINDOW**: **display** checkbox, **above**/**below** numeric
  fields (corridor half-widths in depth units). Display on if a target
  line exists; only change above/below on request.

## Marginals

The per-depth probability field from the computation. Only meaningful when
a job has produced results.

- **marginals (wavelet)** checkbox — wavelet rendering. Specialist view;
  off by default.
- **marginals (color)** checkbox — the color-field rendering. The signature
  Drive view; on by default when results exist.
- Color table dropdown (e.g. `hot`) — with a preview swatch strip.
- **dB scale** toggle — logarithmic display; on is the usual choice.
- dB range dropdown (e.g. `20 dB range`) — display threshold. 20 dB is a
  good default; a larger range shows fainter structure alternatives.

## Interpretations

Two columns.

**COMPUTED** — which computed structure the scene displays:

- Radios: **none** / **MPE** / **P1 n%** / **P2 n%** / **P3 n%**. The
  percentages are each alternative's global probability; alternatives are
  greyed when the computation found none. Hotkeys `0`–`3`.
- **copy to manual** button — snapshots the current computed interpretation
  into a new manual interpretation. Not a display setting; use only on
  request.

**MANUAL** — the hand-picked interpretations:

- **show** checkbox — master visibility for manual interpretations (`m`).
- **+** (create new) and **Import interpretation** (paste picks) — creation
  tools, not display; only on request.
- Per-interpretation rows: selection radio (which one is being edited),
  show checkbox, color swatch, a locate button, a trash button (deletes —
  never without explicit instruction), name field.
- **enable picking** — arms click-to-pick on the canvas. Leave OFF during
  scene setup; a stray canvas click while armed drops a pick (Ctrl/⌘+Z
  undoes).
- **snap to structure** — picking aid; irrelevant while not picking.

## Formations

One row per formation top: color swatch, name (e.g. H3, H4, H5, TOP, BASE),
and a **FILL** checkbox. Fill shades the band below that top — filling the
target (e.g. TOP) makes the target interval pop. One or two fills read
well; filling everything turns the section into wallpaper.

## Traces

Interpolated type-log traces — the type log posted every 30 ft (10 m SI)
along the well, warped onto the displayed structure, so the section reads
like a correlation panel.

- **TYPE LOG TRACES**: **display** checkbox (same as Components →
  interpolated type logs).
- **curve** checkbox + color swatch — the thin trace line. Black means
  theme foreground.
- **fill** checkbox + **Left**/**Right** radios — shades from each curve to
  its slot edge, colored by GR value through the color table below. The
  fill side and color table are SHARED with the type log track's fill
  (each view's on/off is independent).
- Color table dropdown (e.g. `earth tones`).
- **TRACE THROW** slider — trace width; 100% = the shared GR range spans
  one station interval.

Traces hang flat before any structure is computed/picked — pointless to
show on an uncomputed project.

## Derived

Derives a new type log by back-projecting the active log through an
interpretation (Settings, Method radios: Mean / Median / Shallowest MD /
Deepest MD, Save as LAS...). A data-creation workflow, not a display pane —
out of scope for scene setup unless the user explicitly asks.

## Help

The keyboard shortcut reference.

## Type log track settings (gear icon on the left track)

Opens a modal dialog (close with its X):

- **Couple Well Log Scrolling to Cross-Section** checkbox — ties the track's
  depth window to the section's. Leave this OFF: along an inclined wellbore
  the interesting part of the pilot track and the interesting part of the
  section are at different depths, so coupling forces one of them
  off-screen. Position the track and the section independently.
- **Depth axis** radios: TVDSS / TVDTL / TVD — the same three frames as the
  Scene pane, set independently for this track.
- **Auto Scale** checkbox; **Min**/**Max** fields (enabled when Auto Scale
  is off). Prefer Auto Scale OFF with a hand-set, rounded range that
  matches the active log track's range — see "GR scales" in SKILL.md.
  Auto Scale stretches the axis to cover spikes and squashes the real
  signal. A manual scale also FIXES the GR-to-color mapping used by the
  color fills on both the track and the traces — set one when fills need
  to stay comparable across projects or screenshots.
- **Color fill**: **Fill** checkbox, **Left**/**Right** radios, color table
  dropdown — shared side/table with the Traces fill.
- Track width dropdown: SKINNY / MEDIUM / etc.

## Active log track settings (gear icon on the top strip)

Modal: **Auto Scale**, **Min**/**Max**, and a track height dropdown
(SKINNY is the working default height).

This is where you set the GR range first: judge it from the active log
over the MD range you framed, round it (20–200, not 23–183), then copy the
same Min/Max into the type log track so the two curves share an axis and
can be compared honestly.

## The floating on-canvas bar (bottom center)

Interpretation-editing controls: show/hide (eye), Pick (pencil), extend
target line, undo, redo, fault, delete. These arm live canvas gestures —
scene setup never needs them. Don't click them; don't click the canvas
while any of them is active.

## Keyboard shortcuts

| Keys | Action |
|---|---|
| `?` | Shortcuts reference |
| `esc` | Cancel picking (does NOT close panes) |
| `m` | Show/hide manual interpretations |
| `p` | Toggle picking (avoid) |
| `z` | Toggle on-scene zoom controls |
| `0`–`3` | Computed interpretation: none / MPE / alternatives |
| `Ctrl/⌘ + +`/`−` | Zoom in/out |
| wheel / `shift`+wheel / `Ctrl/⌘`+wheel | Scroll ↕ / scroll ↔ / zoom |
| `Ctrl/⌘ + z` / `Ctrl/⌘ + shift + z` | Undo / redo |
