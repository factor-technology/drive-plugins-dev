<!-- published at /help/guide/profile.html on the Drive host -->

# The Cross Section (Profile)

The Profile tab is Drive's main working view: an interactive cross section of the lateral showing the wellbore, the gamma logs, the computed structure with its uncertainty, and any manual interpretations — with a menu bar of click-open panes along the top.

## Menu panes

### Scene

Overall display: the **Lateral Domain** (MD or VS along the horizontal axis), toggles for the active log, type log, correlations, cursor info, legend and names, and the **Depth axis** — **TVDSS**, **TVDTL** (type-log TVD), or **TVD** (see [depth frames](./concepts.md#depth-frames)).

### Components

Show/hide each element of the scene: wellbore, well plan, projected trajectory, pilot wells, interpolated type logs, formations, prior structure — and the background image (once one is uploaded on the [Background tab](./background.md)), with a crop toggle and an **opacity** slider that fades the image so the overlays read against a busy cross section.

### Zoom

Three rows of framing controls, plus a **Toe** button (center the view on the trajectory toe) and a **Reset** button (restore the default scale values, keeping the view centered):

- **Scale** — the overall zoom buttons. Zooming in converges on the wellbore at the middle of the view; zooming out keeps the view centered.
- **V Exaggeration** — the vertical stretch as a multiplier (e.g. **8.0 ×**).
- **Horizontal** — the horizontal scale in real-world units per screen inch (e.g. **1 in = 200 ft**); editing it keeps the vertical exaggeration.

The two numbers are click-to-edit (Enter or Tab commits and moves to the next field) and together define the cross section's scale — to match another scene, copy its two numbers. Keyboard and mouse-wheel equivalents are listed under [keyboard shortcuts](#keyboard-shortcuts).

### Target

The target line: display and picking toggles, the corridor above/below the line, colors, and an optional **horizon extension** drawn out to a chosen ending VS. The target line is drawn by hand — enable picking here and click along the cross section.

### Marginals

The per-depth probability field from the computation, displayed as color or as wavelets, with a choice of color tables, a dB-scale toggle, and a display threshold range. See [Understanding the Results](./results.md#marginals).

### Interpretations

Two tables, side by side and alike: **Computed**, the horizons the computation produced, and **Manual**, the ones you pick by hand. Every row reads the same way — a radio, a checkbox, a color swatch, its name — and a copy icon on a computed row saves an editable copy into Manual. The radio is one group across both tables: **what the cross section follows**. The checkbox is **what it shows**. Neither edits anything: editing is **manual mode**.

- **Following.** The row whose radio is lit is the structure the cross section hangs on: the formation bands, the log tracks' correlations and forward projections, the traces and the cursor readouts follow it. By default that is the MPE. Choose an auto-picked horizon or a manual interpretation to hang everything on it instead. A followed manual interpretation is drawn as a plain ribbon: the radio never puts it in edit.
- **Editing.** Turn on **manual mode** (the checkbox at the top of the Manual table, or key `m`) to see the picks of the manual interpretation whose radio is lit: drag a pick to move it, drag a box to select, delete the selection. Manual mode is independent of the radio; with the MPE or an auto-picked horizon lit it has nothing to show until you choose a manual interpretation. **Enable picking** (key `p`) adds picks by clicking along the log; it needs a manual interpretation on the radio and turns manual mode on. Moving the radio to the MPE or an auto-picked horizon ends picking; manual mode stays on.
- **Showing.** Each checkbox draws or hides its row's horizon, whatever the cross section follows and whether or not manual mode or picking is on. A followed auto-picked horizon or manual interpretation is always drawn; the MPE can be hidden while the cross section still hangs on it, and its formation bands hide with it. **Select all** / **Deselect all** at the top of the pane act on every checkbox — the MPE, the auto-picks and the manual interpretations; Deselect all leaves a followed auto-pick or manual interpretation displayed (key `0` brings the MPE back). Key `0` toggles the MPE's checkbox.
- **Computed** — the [MPE](./results.md#the-mpe) heads the table, and under it come the horizons Drive auto-picks after every run, grouped by the peak of the last marginal each one ends in, by probability mass: a group's tag, **P1** for the heaviest peak, **P2** next, sits to the left of its rows, which share a tint of one color. Its caret folds the alternatives away under the group's lead horizon. A lead row reads its end depth and the group's share of the last marginal; an alternative reads the measured depth where it forks from the lead. Keys `1`–`9` show or hide the lead horizon of group P1..P9. Hovering a row previews it on the cross section. A note appears when the picks come from an older run than the current computation.
- **Manual** — create (**New**), **Import** (paste depth picks), recolor and rename manual interpretations; **Delete checked** removes every manual interpretation whose checkbox is checked (never the one whose radio is lit); toggle **manual mode**, **picking** and **snap to structure**.

### Formations

Fill and color controls for the formation bands, and the top-of-target display.

### Traces

Interpolated type-log traces — copies of the type log posted periodically along the cross section and warped onto the current interpretation, so you can read the interpreted cross section like a correlation panel. **display** shows or hides the traces (the same toggle as on the Components pane). **curve** toggles the thin trace line, with its color picker alongside. **fill** paints each trace's full slot with color — adjacent slots tile the cross section with no daylight between them — coloring every depth by its GR value through the chosen color table; the curve draws over the fill. **opacity** fades the fill so the marginals and structure read through it. The color table is shared with the [type log track](#the-log-tracks)'s fill; each view's fill turns on and off independently. **Trace throw** scales the curve's excursion; the fill is unaffected. Traces hang on the structure the cross section follows (the row whose radio is lit: the MPE, an auto-picked horizon or a manual interpretation) and appear only along the interval it covers — they stop where it ends.

### Derived

Derive a new type log by back-projecting the active log through an interpretation — useful when no good pilot exists near the lateral. The pane works from the backprojections of whatever the cross section follows (the MPE, the auto-picked horizon or the manual interpretation whose radio is lit; windowed if the type-log window is on). Choose the statistic — **Mean**, **Median**, **Shallowest MD**, or **Deepest MD** — that picks the value where the interpretation maps more than one active-log sample to the same depth. The derived curve is merged into the current type log and **Save as LAS...** writes it in pilot TVD, calibrated so the pilot's fit params reproduce the active log. The same computation is available programmatically as the `derive_type_log` agent tool / `POST .../derive-type-log` API.

### Help

The keyboard shortcut reference (also reproduced below).

## The log tracks

The **type log track** — the vertical track on the left edge of the cross section — shows the pilot type log (and, when shown, the active log hung beside it). Its gear icon opens the track settings: the GR display scale (**Auto Scale**, or a manual **Min**/**Max** — a manual scale also fixes the GR-to-color mapping of the color fills, on the track and traces alike), a **Color fill** of the log — on/off, **Left**/**Right** side, and color table — plus scroll coupling, the depth-axis choice, and the track width. The fill's color table is shared with the [Traces](#traces) fill; the side and the on/off switch are the track's own.

The **active log track** — the horizontal strip across the top — shows the active well's gamma log along the lateral. Its gear icon holds just the GR display scale and the track height.

## Editing a manual interpretation

Choose an interpretation with its radio (Interpretations → Manual), enable **picking**, and click along the cross section to place picks. With picking off the interpretation stays editable in **manual mode** — drag a pick, select and move blocks — as long as its radio is lit. While picking:

- Hold **alt** to auto-pick (Drive refines your pick against the data).
- Hold **f** to open a **fault** at the pick.
- Select picks or blocks and use the arrow keys to move blocks up/down, **delete** to remove, **r** to form a new block from selected picks, **shift** to extend a selection.
- **Snap to structure** makes picks snap onto the computed structure.

Undo/redo (Ctrl/⌘+Z, Ctrl/⌘+Shift+Z) applies to interpretation edits.

## Keyboard shortcuts

| Keys | Action |
|---|---|
| `?` | Open the shortcuts reference |
| `esc` | Cancel picking |
| `alt` (held) | Apply auto-picking while picking |
| `delete` / `backspace` | Delete selection (picks or blocks) |
| `shift` | Append to selection |
| `a` | Apply auto-picking to picks in the selection |
| `f` (held) | Open a fault while picking |
| `m` | Manual mode: show / hide the picks of the manual interpretation whose radio is lit |
| `p` | Toggle picking (adds picks to the manual interpretation whose radio is lit) |
| `r` | Make a new block from picks in the selection |
| `z` | Toggle the zoom controls |
| `↑` / `↓` | Move selected block(s) up / down |
| `Ctrl/⌘ + z` | Undo |
| `Ctrl/⌘ + shift + z` | Redo |
| `Ctrl/⌘ + +` / `Ctrl/⌘ + −` | Zoom in / out |
| mouse wheel | Scroll up/down |
| `shift` + wheel | Scroll left/right |
| `Ctrl/⌘` + wheel | Zoom in/out |
| `0` | Show / hide the computed top (MPE) |
| `1`–`9` | Show / hide the lead pick of auto-picked group P1..P9 |
