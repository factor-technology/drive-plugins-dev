---
name: cross-section-setup
description: >-
  Configure, frame, and beautify the cross-section scene on a Factor Drive
  project's Profile tab by driving the browser: zoom and vertical
  exaggeration, scene components, marginals, computed/manual interpretations,
  formations, target line, type-log traces, and the pilot/active log tracks.
  Use whenever the user asks to set up, tidy, frame, zoom, or "make the cross
  section look good", asks for a screenshot or image of a project's profile
  view, or vaguely wants the scene made presentable for review, a report, or
  a meeting — even if they never say "cross-section" (e.g. "get U-178 looking
  decent", "frame the lateral", "show the structure nicely", "prep the
  profile for the morning call"). Requires the Claude in Chrome browser
  tools; uses the Factor Drive MCP tools, when connected, to learn what the
  project contains.
---

# Factor Drive cross-section scene setup

Users ask for this vaguely — "make it look good" — and your job is to
translate that into a scene a geologist would happily put in a morning
report.

**A good scene answers two questions, and both matter equally:**

1. **Where is the wellbore relative to the target zone** — does it stay in
   zone, and where exactly does it wander too shallow or too deep? That's
   the cross-section proper.
2. **Does the correlation support that interpretation** — how well do the
   backprojected active-log segments match the pilot type log? That's the
   **type log track** on the left edge. It is not decoration; it is the
   evidence. A beautifully framed section with a useless log track is a
   half-finished job, and this is the most common way to get this wrong.

The scene is drawn in **SVG**, not a canvas — which matters more than it
sounds. The *accessibility* tree is useless here: `read_page` returns
unlabeled menu items and `find` can't see the scene, because SVG shapes
carry no roles or labels. So aesthetic judgment works the way it does for a
person: **look at a screenshot, judge it, change one thing, look again.**
The `zoom` action on a screen region is your magnifying glass; use it
constantly, especially on the log track, where the detail that matters is
small.

But because it's SVG, the scene is also **fully queryable with
`javascript_tool`** — axis labels are real `<text>` nodes with real screen
positions. Anything numeric you'd otherwise squint at, measure instead. See
"Measure, don't guess" below.

Mechanics for every control on all nine panes and both track dialogs are in
`references/panes.md` — read it before your first pane edit. If the user
wants a saved or cropped image, read `references/capture.md`.

## Step 0 — learn the project before touching the browser

A good scene shows what exists and hides what doesn't. If the Factor Drive
MCP tools are connected, a minute of reading saves many blind clicks — and
tells you things the UI only hints at:

- `list_projects` (`name_contains`) — find the project and its `scope`. Every
  other tool takes a `{scope, name}` pair.
- `read_project` — `tot_name` (the top-of-target marker), `vs_azimuth`, the
  GR log mnemonics, the compute range, the datum, and **`urls`**. Use
  `urls.profile` verbatim rather than assembling a URL yourself.
- `read_active_trajectory` (detail `summary`) — **this is how you tell what
  state the well is in**, and it's the single most useful call. The last
  waypoint's `incl` and `vs` settle it: inclination near 90° with a large
  `vs` means a landed lateral; a small inclination means the well hasn't
  landed. `md_last` is the toe. Two real examples: Hawk Ridge 1H ends at
  `incl 5.41°, vs 765` — still vertical; SND 621H ends at `incl 90.26°,
  vs 9827` — a full lateral.
- `read_job_status` — whether a computation has run. **Careful: `complete`
  does not mean there is a meaningful structure to display.** A pre-lateral
  well can have a completed job whose MPE is a vertical stub. Judge the well
  from the trajectory and use job status only to know whether results exist
  at all.
- `read_well_plan` (`count: 0` means none) and `read_target_line`
  (`exists: false` means none) — check these before planning to display
  them. Hawk Ridge 1H has neither, which makes a large part of the
  pre-lateral recipe moot for that project.
- `list_pilot_wells`, `list_interpretations`, `read_seismic_image` — the rest
  of what's available to show.

If the MCP isn't connected, the UI still tells you: **greyed-out controls
mean the underlying data doesn't exist.** Never fight a greyed control —
it's information, not an obstacle. Working this way costs real time, though;
runs without the MCP have had to dig through the Inventory tab and infer the
target interval by eye to learn what one `read_project` call would answer.
If the tools look unauthorized, say so rather than quietly working around
it.

Without the MCP, the Profile URL is
`https://<drive-host>/<org>/projects/<project>/profile` (name URL-encoded).

## Browser mechanics

These are specific to this app and each one has bitten someone:

- **Wait out the loading screen.** A full-screen dark overlay with an
  aphorism and a spinner covers the page while loading, and it can last
  20–30 seconds on a big project. Clicks during it are swallowed. Wait,
  screenshot, and only proceed when the scene is actually drawn.
- **Check the window size early.** A cramped viewport produces cramped
  screenshots and misleading framing judgments. If the window is small,
  resize it before you start judging.
- **Panes close by clicking empty space on the menu strip** — the blank area
  to the right of "Help". Clicking the pane's own menu item again does *not*
  close it, and Escape does not either. Clicking a different menu item
  switches panes. Close the pane before judging framing, and before any
  capture.
- **Numeric values are click-to-edit.** Click the number itself (e.g. the
  Zoom pane's VERTICAL "12.0"): it becomes a focused input with the value
  selected. Type a new value, press Enter.
- **SCALE and VERTICAL are logarithmic — roughly 1.5× per unit.** Estimating
  them linearly is a trap: typing 28 when 13 was right zooms by a factor of
  hundreds and throws the scene clean out of frame. Step with the −/+
  buttons a unit at a time and re-read the depth axis after each step, and
  keep direct entry for small corrections near a value you already trust.
- **Zoom changes can lose the scene.** Changing SCALE/VERTICAL/HORIZONTAL
  re-anchors the viewport and content can scroll clean out of frame (a wall
  of empty grey). Don't undo — click the **locate-toe pin** in the Zoom pane
  to recenter, then re-screenshot. The circular-arrow button resets zoom to
  defaults, a good escape hatch.
- **Panning.** Wheel over the cross-section scrolls it vertically. Horizontal
  panning (shift+wheel) is unreliable through the automation bridge — prefer
  adjusting SCALE plus locate-toe over trying to drag sideways.
- **Wheel over the type log track scrolls that track** independently, finely
  and predictably (roughly a few feet per tick at moderate zoom). This is
  the good way to position the track. The thin vertical slider beside it
  jumps hundreds of feet per drag — too coarse to aim with.
- **The scene is live.** If the Pick or Extend-target tool is armed, a click
  on the scene drops an interpretation pick or target point. Never click it
  to "focus" or dismiss something. Leave the floating button bar at the
  bottom center alone unless the user asked for interpretation edits. A stray
  pick undoes with Ctrl/⌘+Z.
- **Useful keys:** `0`–`3` switch the displayed computed interpretation
  (none / MPE / alternatives); `m` toggles manual interpretations; `?` opens
  the shortcut reference.
- **Settings persist per project**, in browser localStorage under the key
  `<org>/<project>`. You are editing the geologist's saved scene, not a
  throwaway view: screenshot a pane before changing it so you can restore
  anything they didn't want touched, and tell them what you changed. If a
  scene is hopelessly mangled or the user wants a genuine clean slate,
  `localStorage.removeItem("<org>/<project>")` followed by a reload restores
  factory defaults — destructive of their saved preferences, so ask first.

## Measure, don't guess

Judging "are the depth labels about 20 ft apart?" from a screenshot is
guesswork, and the guesses have been wrong. Read it off the DOM instead:

```js
const out=[];
document.querySelectorAll('svg text').forEach(t=>{
  const s=t.textContent.trim();
  if(!/^-?\d+(\.\d+)?$/.test(s)) return;          // numeric axis labels only
  const r=t.getBoundingClientRect();
  if(r.bottom<0||r.top>innerHeight||r.right<0||r.left>innerWidth) return;
  out.push({v:parseFloat(s), x:Math.round(r.x), y:Math.round(r.y)});
});
// group into vertical stacks by x — each stack is one depth axis
const cols={};
out.forEach(p=>{const k=Math.round(p.x/25)*25;(cols[k]=cols[k]||[]).push(p);});
Object.entries(cols).filter(([,v])=>v.length>=3).map(([k,v])=>{
  v.sort((a,b)=>a.y-b.y);
  const steps=[];for(let i=1;i<v.length;i++)steps.push(Math.abs(v[i].v-v[i-1].v));
  steps.sort((a,b)=>a-b);
  return {xCol:+k,count:v.length,min:Math.min(...v.map(p=>p.v)),
          max:Math.max(...v.map(p=>p.v)),medianStep:steps[steps.length>>1]};
});
```

This returns one entry per visible axis: the depth range on screen and the
spacing between labels. A real result from the type log track was
`{min:-10800, max:-10640, medianStep:20}` — 160 ft visible at 20 ft
spacing, which is the target. `medianStep: 100` means keep zooming.

**The filtering is the whole trick.** The scene SVG is tens of thousands of
units tall and the DOM holds every label for the entire well — over a
thousand of them — clipped to a small viewport. Without the
`getBoundingClientRect` check against the window you'll measure labels that
aren't on screen and conclude nonsense.

The same approach answers other questions cheaply: which formation markers
are rendered (`<text>` like `top`, `bottom`, `Pilot: Pilot Well 1`), and
where the target band sits. Use it to verify, then still look at the
screenshot to judge whether the result is *attractive* — measurement settles
facts, not aesthetics.

## The loop

1. Screenshot the loaded scene. Diagnose against the quality bar below —
   name the two or three biggest problems to yourself before acting.
2. Fix **content** first (Components / Interpretations / Marginals / Scene):
   what's shown and hidden. Framing tuning is wasted if the elements are
   about to change.
3. Fix **framing** (Zoom). After each change: locate-toe, close the pane,
   screenshot, judge.
4. Fix the **log tracks** — zoom and scroll the type log track onto the
   target interval. Don't skip this.
5. **Garnish**: formation fill, target annotation, traces — only where they
   add readability.
6. Close all panes, final screenshot, run the quality bar again.

One change, one look. Screenshots are cheap; guessing three edits deep is
how scenes get mangled.

## What "looks good" means

### Vertical framing — the most important judgment

Frame around **the target zone and the wellbore's traversal through it**,
not around the formation stack. Only the top and bottom of the target really
matter; the other formation tops are informational context. Do not stretch
the view to fit them all in — that is the classic mistake, and it shrinks
the thing the geologist actually came to see.

Set vertical exaggeration so a reader can see the wellbore rise and fall
within the zone and spot exactly where it goes out — too shallow or too
deep. An excursion of ten or twenty feet should be visible at a glance.

But don't overdo it. **The structure should still look like geology, not a
mountain range.** Gentle regional dip rendered as cliffs is worse than
useless — it misleads. If the structure looks dramatic, back the VE off. You
are looking for the value where the wellbore-vs-zone relationship reads
clearly and the formation surfaces still look like sedimentary geology.

The other failure mode is a thin ribbon of signal floating in an ocean of
empty grey — that's too little VE, or too much frame.

### Horizontal framing

Fill the width — but leave a little space to the left and right of the
computed structure rather than jamming it edge to edge. A section clipped
flush at both ends looks like a mistake even when it isn't, and the reader
loses the sense of where the lateral begins and ends.

### The type log track — the correlation panel

This track (left edge) shows the black pilot type log with the
backprojected active-log segments drawn over it in color. Its whole purpose
is letting the geologist inspect how well those segments correlate against
the pilot curve — and where several segments spread across a range of
depths, which is where the interpretation is least certain and most
interesting.

At default zoom the track shows hundreds of feet and the target interval is
a squashed sliver, often scrolled off-screen entirely. **Fix this every
time — and expect to zoom in much further than feels necessary.** Under-
zooming is the single most common defect in a finished scene.

The thing to size is the **hot zone**: the depth range where the colored
active-log segments actually live, which is usually a few tens of feet
around the target top, not the whole target interval. Give that zone
several vertical inches of screen. One inch of squashed overlay is a
failure; three or four inches of it is right.

Two concrete checks:

- **Depth gridline spacing** — measure it with the snippet in "Measure,
  don't guess" rather than estimating. 100 ft between labels means you are
  still far too zoomed out; **around 20 ft between labels is typical** for a
  readable correlation. Scale this to the hot zone's thickness — a thin fan
  wants tighter spacing, a thick one tolerates more.
- **How much height the colored segments occupy.** If the fan of colored
  segments fills less than about a third of the track height, keep zooming.

Then scroll it onto that zone with the **mouse wheel over the track**,
checking with a zoomed screenshot after each nudge.
- **Leave "Couple Well Log Scrolling to Cross-Section" off** (track gear
  dialog). It sounds helpful and isn't: the track and the section want
  different depth windows. Along an inclined wellbore the interesting part of
  the pilot track and the interesting part of the section sit at different
  depths, so coupling them guarantees one of the two is off-screen. Position
  the two independently — that freedom is the point.

**Track widths:** MEDIUM for the type log track (gear dialog) and SKINNY for
the active log track along the top is a good working default — enough room
to read the correlation without stealing width from the section.

### GR scales — turn Auto Scale off, and match the two tracks

Auto Scale sounds safe and usually isn't. Log data is full of spikes, and
auto-scaling stretches the axis to span the largest of them, which squashes
every real feature into a narrow squiggle down the middle. The curve shapes
you are trying to compare get flattened out of existence by a handful of
outliers.

Set the scales by hand instead:

1. Look at the **active log** over the MD range you actually framed — not
   the whole well — and judge where its meaningful values live, ignoring
   isolated spikes.
2. Turn Auto Scale off in the active log track's gear dialog and enter that
   Min/Max.
3. **Give the type log track the same Min/Max.** This is the point: the two
   curves are only visually comparable when they share an axis. Different
   scales make a matching pair of curves look like a mismatch, which is
   exactly the wrong conclusion to invite.

**Round to nice numbers** — 20 to 200, not 23 to 183. Odd bounds make the
gridlines meaningless and the reader do arithmetic; round ones let them
read values off the track at a glance.

A manual scale on the type log track has a second benefit: it also pins the
GR-to-color mapping used by the color fills on the track and the
interpolated traces, so fills stay comparable instead of shifting whenever
the data range changes.

### Components, interpretations, and the rest

**Components on:** wellbore; well plan (if one exists); pilot wells;
formations. **Off by default:** projected trajectory (useful while drilling),
prior structure (show when there's no computed structure, or when
prior-vs-computed is the point), interpolated type logs (turn on when a
structure exists and the section is long enough that they read as a
correlation panel rather than clutter).

**Interpretations:** show the MPE unless the user asks for alternatives —
never leave it on a 0%-probability alternative, which happens by accident and
quietly shows a structure nobody believes. Show a manual interpretation if
the project maintains one; hide a pile of stale auto-generated ones that
overprint the structure band.

**Marginals:** when a computation exists, color marginals on — Drive's
signature view. `hot` is a solid default, dB scale on, ~20 dB range. Wavelet
marginals are a specialist view; leave them off. No computation means no
marginals: show prior structure and well plan instead.

**Formations:** fill the target interval so bit-vs-target reads instantly,
and leave the other tops as unfilled lines. Watch the color clash: against a
`hot` marginals colormap a saturated fill fights the heatmap for attention,
so prefer something muted — though a mid-grey can disappear into the grey
canvas, so check your choice on screen rather than assuming.

**Target:** target line displayed with annotation; drilling window displayed
if defined. Horizon extension off unless the conversation is about what lies
ahead.

**Scene pane:** active log, type log, correlations and name on; **cursor info
and legend off** — they add boxes that photograph badly. Depth axis TVDSS
unless the user works in another frame. Lateral domain MD unless they ask
for VS.

### Adapting to the well's state

- **Full lateral, computation complete:** the recipe above as written.
- **Lateral in progress:** same, plus the projected trajectory. Frame from
  just before landing to a respectable distance ahead of the bit — the
  audience cares about what's coming. Don't over-stretch a short lateral to
  fill the width; it distorts the geology.
- **Not landed yet / still vertical** (last waypoint inclination well short
  of 90°): there is no real structure to show, even if a job has run to
  `complete` — its MPE will be a vertical stub. Turn marginals and the
  computed interpretation off. Show whatever *does* exist of the well plan,
  target line, prior structure, formations and pilot wells — the scene is
  about the plan, not the result — but check first, because a young project
  often has neither a plan nor a target line yet, and displaying an empty
  layer accomplishes nothing. **Use the MD domain, not VS:** a vertical well
  has almost no vertical-section extent, so in VS it collapses into a stripe
  while MD spreads the hole out readably. Frame the hole and the target
  interval. This scene will look sparse; that's correct, not a failure.

## Final quality bar

Look at the last screenshot and ask:

- Can a geologist see at a glance where the wellbore sits relative to the
  target zone, and where it leaves it?
- Does the structure still look like plausible geology, or did the vertical
  exaggeration turn gentle dip into cliffs?
- Is the type log track zoomed far enough — depth labels roughly 20 ft
  apart, colored segments filling a good share of the track height — that
  the correlation can actually be inspected rather than merely glimpsed?
- Do both tracks share the same hand-set, round GR range, with the curves
  using the full width rather than squashed into a spike-driven axis?
- Does the section fill the width with a little margin at each end, rather
  than being clipped flush or stranded in empty grey?
- Are annotation boxes (survey stations, dip labels) piling into an
  unreadable heap?
- Is any menu pane or dialog still open?

If the user asked for a saved or cropped image, capture per
`references/capture.md`. Delivery beyond a saved file — email, slides,
documents — is other skills' territory; hand off rather than improvise.
