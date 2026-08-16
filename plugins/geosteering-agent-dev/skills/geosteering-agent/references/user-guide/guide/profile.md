<!-- published at /help/guide/profile.html on the Drive host -->

# The Cross Section (Profile)

The Profile tab is Drive's main working view: an interactive cross section of the lateral showing the wellbore, the gamma logs, the computed structure with its uncertainty, and any manual interpretations — with a menu bar of click-open panes along the top.

## Menu panes

### Scene

Overall display: the **Lateral Domain** (MD or VS along the horizontal axis), toggles for the active log, type log, correlations, cursor info, legend and names, and the **Depth axis** — **TVDSS**, **TVDTL** (type-log TVD), or **TVD** (see [depth frames](./concepts.md#depth-frames)).

### Components

Show/hide each element of the scene: wellbore, well plan, projected trajectory, pilot wells, interpolated type logs, formations, prior structure — and the background image (once one is uploaded on the [Background tab](./background.md)), with a crop toggle and an **opacity** slider that fades the image so the overlays read against a busy section.

### Zoom

Three rows of framing controls, plus a **Toe** button (center the view on the trajectory toe) and a **Reset** button (restore the default scale values, keeping the view centered):

- **Scale** — the overall zoom buttons. Zooming in converges on the wellbore at the middle of the view; zooming out keeps the view centered.
- **V Exaggeration** — the vertical stretch as a multiplier (e.g. **8.0 ×**).
- **Horizontal** — the horizontal scale in real-world units per screen inch (e.g. **1 in = 200 ft**); editing it keeps the vertical exaggeration.

The two numbers are click-to-edit (Enter or Tab commits and moves to the next field) and together define the section's scale — to match another scene, copy its two numbers. Keyboard and mouse-wheel equivalents are listed under [keyboard shortcuts](#keyboard-shortcuts).

### Target

The target line: display and picking toggles, the corridor above/below the line, colors, and an optional **horizon extension** drawn out to a chosen ending VS. The target line is drawn by hand — enable picking here and click along the section.

### Marginals

The per-depth probability field from the computation, displayed as color or as wavelets, with a choice of color tables, a dB-scale toggle, and a display threshold range. See [Understanding the Results](./results.md#marginals).

### Interpretations

Two sub-panes:

- **Computed** — choose what the section displays: nothing, the **MPE**, or one of the probability-ranked alternatives (**P1 %**, **P2 %**, **P3 %** — the percentage is each alternative's global probability). Keys 0–3 switch between them. A copy button snapshots the current computed interpretation into a new manual interpretation.
- **Manual** — create (**Add**), **Import** (paste depth picks), show/hide, recolor, and rename manual interpretations; select one for editing; toggle **picking** and **snap to structure**.

### Formations

Fill and color controls for the formation bands, and the top-of-target display.

### Traces

Interpolated type-log traces — copies of the type log posted periodically along the section and warped onto the current interpretation, so you can read the interpreted section like a correlation panel. **display** shows or hides the traces (the same toggle as on the Components pane). **curve** toggles the thin trace line, with its color picker alongside. **fill** toggles a color fill from each curve to the left or right edge of its slot, coloring every depth by its GR value through the chosen color table — the same fill offered on the [type log track](#the-log-tracks). The fill side and color table are shared with the track's fill; each view's fill turns on and off independently. **Trace throw** scales trace width.

### Derived

Derive a new type log by back-projecting the active log through an interpretation — useful when no good pilot exists near the lateral.

### Help

The keyboard shortcut reference (also reproduced below).

## The log tracks

The **type log track** — the vertical track on the left edge of the section — shows the pilot type log (and, when shown, the active log hung beside it). Its gear icon opens the track settings: the GR display scale (**Auto Scale**, or a manual **Min**/**Max** — a manual scale also fixes the GR-to-color mapping of the color fills, on the track and traces alike), a **Color fill** of the log — on/off, **Left**/**Right** side, and color table — plus scroll coupling, the depth-axis choice, and the track width. The fill's side and color table are shared with the [Traces](#traces) fill; the on/off switch is the track's own.

The **active log track** — the horizontal strip across the top — shows the active well's gamma log along the lateral. Its gear icon holds just the GR display scale and the track height.

## Editing a manual interpretation

Select an interpretation for editing (Interpretations → Manual), enable **picking**, and click along the section to place picks. While picking:

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
| `m` | Display manual interpretation |
| `p` | Activate picking |
| `r` | Make a new block from picks in the selection |
| `z` | Toggle the zoom controls |
| `↑` / `↓` | Move selected block(s) up / down |
| `Ctrl/⌘ + z` | Undo |
| `Ctrl/⌘ + shift + z` | Redo |
| `Ctrl/⌘ + +` / `Ctrl/⌘ + −` | Zoom in / out |
| mouse wheel | Scroll up/down |
| `shift` + wheel | Scroll left/right |
| `Ctrl/⌘` + wheel | Zoom in/out |
| `0`–`3` | Select computed interpretation (none / MPE / alternatives) |
