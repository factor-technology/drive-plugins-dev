<!-- §1.11–1.12 cross-section coaching — extracted from agent/geosteering-agent.md §1 by build-skill.mjs. Load per the map in SKILL.md; the behavioral core is assumed context for every other file. -->

## 1.11 Interpreting in the cross-section (mouse & touch)

The cross-section is where the geologist draws by hand: the **manual
interpretation** (their picked horizon — a row of connected picks forming one
or more blocks) and the **target line** (the planned target they extend ahead
of the bit). Both are editable from chat too (the interpretation and
target-line tools, §1.10), but the on-canvas drawing is the geologist's own
gesture, so this is **coaching** knowledge — explain the gesture in the UI's
own words. The same two tools are driven two
ways: with a mouse and keyboard, or by touch through floating buttons that
appear over the cross-section (those buttons show for mouse users too, as an
alternative to the keys).

**Entering and leaving a tool.** Both are toggles, and turning one on turns the
other off (only one is active at a time):

- **Pick** (manual interpretation) — the **Pick** button on the on-canvas bar,
  or the `P` key. Separately, **Show/Hide** (or `M`) toggles whether
  interpretations are visible at all.
- **Extend target line** — the **Extend target line** button on the on-canvas
  bar, the **extend** checkbox in the side panel's Target Line section, or the
  `T` key. Extending needs the well's azimuth set and a measured toe; the
  control is greyed out until then.

**With a mouse and keyboard.** With **Pick** on, click along the log to drop
picks; three or more make a block. While picking: hold **Alt** to snap new
picks to the computed structure; hold **F** to open a fault — a horizontal line
tracks the cursor depth from the nearest block edge, click once to set the
throw, again to drop the offset block. (`A` auto-picks the current selection;
`R` builds a new block from the selected picks.) With **Pick** off but
interpretations shown you edit instead of draw: click a segment between two
picks to insert one, drag a pick to move it, drag inside a block to shift the
whole block, drag a box to select, `↑`/`↓` nudge the selected block by one
depth unit, and **Delete**/**Backspace** removes the selection. `Ctrl/⌘-Z`
undoes, `Ctrl/⌘-Shift-Z` redoes, **Esc** cancels the draw in progress; `0`–`3`
show the computed top and its alternatives; `?` lists every key. For the
**target line**, turn **Extend** on and click to add points ahead of the bit;
turn **Extend** off to adjust — hover a point and drag it.

**By touch.** The same actions run through two floating button bars (taps on a
bar never reach the canvas, so they can't drop a stray pick); long-press any
button to read its label without firing it. Before picking, the bar offers
**Show** and **Extend target line**; once interpretations are shown it adds
**Pick**, **Select**, **Undo/Redo**, **Clear selection**, and **Delete**. While
picking, it offers **Hide**, **Pick** (which finishes), **Snap**,
**Undo/Redo**, **Fault**, **Split**, and **Cancel**. On the canvas: one finger
draws while picking (drag, lift to drop a pick), or **moves** a pick or block
when picking is off (press it and drag); two fingers **pan and pinch-zoom**.
**Fault** — tap **Fault** to arm, press a block edge, drag to the throw depth,
lift, then confirm **Place fault**. **Split** — tap **Split**, then tap a block
top between two picks to insert one. For the **target line** with **Extend**
on, drag from empty space to extend the line, or press directly **on a point**
to drag that point; **Done**/**Cancel** on the bar finishes or backs out.

## 1.12 Interpolated type-log traces (display)

The cross-section can overlay the type log as small vertical GR traces, one
every 30 ft (10 m SI) of measured depth along the well. Each trace is the
effective type log at that position — between pilot wells it is the same
distance-weighted, marker-warped blend the computation and the projections
use — and each trace hangs on the currently displayed structure (the selected
computed top, or the manual interpretation when shown), so the ensemble reads
as the interpreted cross-section. Traces render only along that structure's
own lateral extent — they stop where it ends, and none render before any top
is computed or picked. All traces share one GR scale — the type log track's
manual display scale when the user has set one (Auto Scale off), else the
global min–max across every trace — mapped to a fixed width centered at each
trace's position.

Turn the overlay on with the **interpolated type logs** checkbox on the
**Components** pane or the **Traces** pane; the Traces pane also holds the
settings. **Trace throw** scales trace width (100% = the shared GR range
spans one station interval). **Curve** toggles the thin trace line and
**Color** sets its color; choosing black means the theme's proper foreground
(near-black in light mode, near-white in dark). **Fill** paints each trace's
full slot with color — adjacent slots tile the section with no gap, so the
filled ensemble reads as a continuous color panel — coloring every depth by
its GR value through a chosen color table; the curve draws over the fill,
and an **opacity** slider fades the fill. A similar color fill is available
on the type log track (the vertical GR track on the left edge of the
section) via its gear-icon settings; the color table is one shared choice
across both views, while each view's fill toggles on and off independently
(the track's fill keeps its own left/right side choice and stays opaque).
Settings persist per project.
