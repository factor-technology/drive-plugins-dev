<!-- Hand-authored static reference for the geosteering-agent skill.
     Ships verbatim via build-skill.mjs copyRefs. Source of truth: this file
     (agent/skill/references/derived-log.md), distilled from a coached
     worked example on a sandbox project. -->

# Derived type log (the Profile tab's Derived pane)

Back-project the active well's GR through an interpretation to synthesize a
type log from the well itself, then make that the project's type log. This is
the modern form of an old manual-geosteering move: when the given type log
can't explain what the well is drilling, the geosteerer stops correlating
against it and starts correlating the well **against itself** — each new
encounter of a recognizable feature is lined up against the previous passes,
building a local synthetic column as the lateral marches along. Drive needs
an actual type log object to steer against, and the Derived pane is how you
mint one.

## When to reach for it — the diagnosis

The signature is a **completed job whose type-log correlation stays bad**.
The structure may look plausible — or may itself be contorted, with dips the
geologist doesn't believe, precisely because the model is bending structure
to accommodate stratigraphy the type log doesn't describe:

- On the type log track, the backprojected MWD segments don't share character
  with the type log — peaks with no counterparts, different amplitudes,
  spiky-vs-smooth.
- The active log carries distinctive local character — clean low-GR
  stringers, hot streaks, washed-out zones — that appears **nowhere** on the
  type log. Those features are markers of the local stratigraphy; no amount
  of warping the given type log will ever produce them.
- The tuning history is a tell: if dip wiggle-room had to be opened up and
  curvature constraints dropped just to force markers to the right depths,
  the model is compensating for stratigraphy the type log doesn't describe.
  (A good fit achieved that way is the §1.4 "good fit, wrong setup" case.)

Two ways the situation arrives: as a **surprise** mid-lateral, or
**anticipated** — the geologist has seen the same character on neighboring
wells and expects it. Either way the remedy is the same.

The backprojections are the interpretation's own prediction turned into
evidence: given the computed (or manual) structure, each active-log sample
hangs back onto the type-log depth axis (TVDTL). If the structure were right
AND the stratigraphy matched, the segments would overlay the type log. When
they overlay each other but not the type log, the type log is the problem.

## The workflow

Six steps. The first is the geologist's judgment; the middle is the Derived
pane; the tail is the ordinary pilot-log-replace machinery.

### 1. Force a structural opinion first — a short manual interpretation, early

The derivation runs through an interpretation, and at the moment you need it
the computed one is (by hypothesis) compromised. So the geologist picks a
**short manual interpretation over the early lateral** — enough footage to
capture the characteristic features — and adjusts it until the backprojected
segments stack sensibly on the type log track.

- The structural opinion comes from independent evidence (seismic, offset
  wells, regional dip) — not from the new features. Wherever the features
  then backproject through that structure is where they belong in the local
  column. Don't reverse this.
- Simple is fine. A straight dipping segment suffices when seismic says the
  section is straight; the general case is a real hand-picked interp.
- **Early matters.** The derived log only helps the lateral that remains
  after it's made. Interpret the first sufficient stretch, not the prettiest
  one.

Manual-interp gestures are cross-section coaching (`cross-section.md`); the
cross-section-setup skill drives them in the browser when that's in play.

### 2. Open the Derived pane and read it

Profile tab → **Derived**. The pane is display-and-save only: nothing exists
server-side until the upload step. What it shows:

- A log track on the type log's own depth axis (TVDTL). **Correlations**
  (orange) are the backprojected active-log passes through the currently
  displayed interpretation — the manual one when shown, else the selected
  computed structure. The white curve is the **derived type log** being
  assembled. (Settings also offers the initial type log for comparison, a
  legend, and the GR scale. Dark-mode quirk: the legend may swatch the
  derived curve black; on screen it draws in the theme foreground.)
- **Method radios** — Mean / Median / Shallowest MD / Deepest MD — the
  statistic that combines overlapping passes (next step).
- **Splice semantics:** the derived curve covers only the stratigraphic
  range the wellbore actually explored — from the TVDTL that
  `md_first_to_compute` maps to, down to the deepest depth any pass reached.
  Above and below, the saved product keeps the **original** type log, with a
  taper blending the seam. The wellbore is a one-dimensional sample of the
  section; where it never went, the old log is still the best you have.

### 3. Choose the statistic by reasoning, not habit

Where the lateral crossed the same stratigraphic depth more than once, the
passes can disagree — a stringer developed at one lateral position and not
another, or one pass's structural placement slightly off. The method decides
**which lateral position's character represents the column**:

| Method | Meaning | Behavior on a disputed feature |
|---|---|---|
| Shallowest MD | earliest (heelward) pass wins | may miss character encountered later |
| Deepest MD | latest (toeward) pass wins | most current; carries recent features forward |
| Mean | average across passes | dilutes — a clean spike survives at half strength |
| Median | majority across passes | robust; keeps features most passes agree on |

There is no house default. Ask: **which choice best represents the
stratigraphy the remainder of the lateral will see?** Click through the
methods and watch the characteristic features — the choice is usually
decided by whether they survive. (In the worked example, Deepest MD was
defensible — most recent passes, guarantees the clean spikes — and Median
captured the same character more robustly; Median won. On another pass at
the same data a different statistic produced the better final answer:
iterate, don't sanctify.)

The disagreement between passes is itself information — say so if it's
material, rather than silently averaging it away.

### 4. Save the LAS

**Save as LAS...** opens a native OS save dialog — the one control in this
workflow browser tools cannot drive. If you're driving, ask the geologist
for that single click and where the file landed; if they're driving, it's
just their save. Default name:
`<scope>_<project>-derived-type-log.las`.

The file is written in the canonical pilot frame — positive-down pilot TVD,
with the GR values inverse-transformed through the pilot's current
calibration. Consequence: re-uploaded **with calibration preserved**, the
curve lands exactly where the pane displayed it, and the formation markers
keep their depths (same TVDTL axis). No marker re-pick, no re-fit.

### 5. Replace the pilot log through the connector

This is a pilot-log re-upload — **advisory + approval** (§1.5.5 posture).
It is also fully mechanical, so **offer to do it**: the geologist can
equally replace the log themselves in the Setup UI, and who clicks is
preference, not principle. Doing it yourself needs the file reachable from
code execution (ask for the folder if you don't have it).

1. `create_upload` (kind `pilot_log`) → run the returned curl from code
   execution → the response confirms the channel list.
2. `upload_pilot_log` with `{upload_id, gr_mnemonic}` and **`reset_fit:
   false`** — preserving `fit_params` is what makes step 4's frame
   round-trip work. Do not "helpfully" reset the calibration.
3. Note the contract: the stored align-logs warp is dropped with the
   replaced log. Re-run alignment only if warp was actually in use.

### 6. Reset, run, verify

A replaced type log invalidates all saved computation: `reset_job` then
`trigger_job_rerun` (the UI's Reset and Run) — with approval, since it
recomputes the well from scratch: visible churn, typically minutes on a
lateral of ordinary reach. If WITSML polling is on, pause it around the
manual run (§1.9.6); an email-fed project has nothing to pause.

Then judge the result the way the diagnosis was made, on the picture, and
**re-read project state before quoting any number — the geologist may have
edited the project mid-session**:

- **Type log track:** the local character should now repeat and line up —
  the same low-GR spike (or other marker) recurring pass after pass, each
  landing on the derived template's rendition of it. That recurrence is the
  visual meaning of "steering it against itself".
- **Active log strip:** the backprojection overlay should hug the measured
  log along the whole lateral, not just the interpreted stretch.
- **Cross-section:** the structural answer should be geologically plausible
  — a tight corridor near the prior's trend, apparent dips near the prior's,
  no uncertainty fan blooming toward the toe. And the loosened tuning that
  motivated all this can often be tightened back up afterward.

A mediocre outcome is cheap to retry: different statistic, better (or
longer) manual interpretation, then derive-replace-rerun again.

## Driving vs coaching

Default to volunteering. Nearly all of this is mechanical — only the manual
interp, the statistic, and the approvals are judgment, and those stay with
the geologist. Everything else, offer to do.

With browser tools available, that means operating everything except the
native save dialog: open the pane, set Settings, click through methods,
compare, and ask for exactly one click. Without browser tools, coach the
same sequence by pane and control name — the pane reads left-to-right
exactly as §2 above describes it — while still offering the connector tail
(steps 5–6) yourself, which needs no browser at all. User-facing language
stays plain in both modes: "replace the type log with the derived one and
re-run" — never tool names or field names.

## Pitfalls

1. **Deriving through the compromised computed interp.** The point of step 1
   is that the geologist's structural opinion, not the misfit computation,
   defines the backprojection. Check which interpretation the scene displays
   before opening the pane.
2. **Late derivation.** A derived log made at the toe explains footage
   already drilled and helps nothing. Raise the option as soon as the
   diagnosis pattern appears.
3. **Resetting the calibration on upload.** `reset_fit: true` breaks the
   frame round-trip and orphans the markers' alignment. Preserve fit on
   replace; the defaults are correct.
4. **Treating a statistic as the default.** The method is a judgment about
   the remainder of the lateral. Compare at least two on the features that
   motivated the derivation.
5. **Forgetting what's derived.** The project's type log is now synthetic
   below/above the splice points only in the explored range — character
   outside it is still the old pilot's, and deepening the well extends what
   a future derivation could cover. Mention the provenance when the
   geologist reasons from log character in the unexplored range.
6. **Quoting stale project state.** A coached session edits the project
   under you — pilot logs, tops, compute range can all change. Re-read
   before asserting.
