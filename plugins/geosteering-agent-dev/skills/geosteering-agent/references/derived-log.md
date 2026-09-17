<!-- Hand-authored static reference for the geosteering-agent skill.
     Ships verbatim via build-skill.mjs copyRefs. Source of truth: this file
     (agent/skill/references/derived-log.md). -->

# Steering the well against itself (the derived type log)

When the type log can't explain what the well is drilling, stop correlating
against it and correlate the well **against itself**. Back-project the active
GR through a short interpretation of the structure and make that curve the
project's type log; from then on every new pass is lined up against the stratigraphic column
the well itself established. Drive needs an actual type log object to steer
against, and the Profile tab's **Derived** pane (or `derive_type_log`) mints
one.

The derived curve **terminates at the stratigraphy the wellbore has explored**:
it starts at the wellbore's own depth at the first computed position and ends
at the deepest depth the interpretation reached. Nothing is grafted on above
or below. That is the mechanism, not a limitation. The computation reads a
type log as ground truth over its whole length and treats a wellbore past
either end of it as **impossible**, so the moment the well drills
stratigraphically past an end the estimate piles up at that end, the
marginals spread, and uncertainty climbs. **That is the alarm to derive
again.**

So this is a **loop**, not a one-shot recipe: derive early, run, watch the
alarm, rebuild the interpretation through the run and extend it over the
new footage, derive again. Each cycle grows the log by whatever new
stratigraphy the footage just drilled crossed and revises it wherever the
run moved its structure over footage the log already held; successive
versions agree where the runs agree — on one real lateral that deepened
250 ft of stratigraphy over 2,900 ft, ten cycles of about 300 ft agreed to
within a couple of gAPI rms in the overlap. The loop converges on the log a
geologist would draw at the end of the well, except you have it while
drilling.

## When to start

**Right after the curve, early.** The derived log only helps footage still
ahead. Interpret the first sufficient stretch of lateral, not the prettiest.

Otherwise the trigger is a **completed run whose type-log correlation stays
bad**. The structure may look plausible, or may itself be contorted with dips
the geologist doesn't believe, precisely because the model is bending
structure to accommodate stratigraphy the type log doesn't describe:

- The log tracks disagree with the type log. On the active log track along
  the top, the black curve — the type log read along the structure the
  cross section follows, the GR that structure says the bit should have
  measured — shares no character with the measured passes in color; on
  the type log track, the backprojected MWD segments share none with the
  type log: peaks with no counterparts, different amplitudes,
  spiky-vs-smooth.
- The active log carries distinctive local character — clean low-GR
  stringers, hot streaks, washed-out zones — that appears **nowhere** on the
  type log. No amount of warping the given log will produce them.
- The tuning history is a tell. If dip wiggle-room had to be opened and
  curvature constraints dropped just to force markers to the right depths,
  the model is compensating for stratigraphy the type log lacks (the §1.4
  "good fit, wrong setup" case).

The backprojections are the interpretation's own prediction turned into
evidence: given the structure, each active-log sample hangs back onto the type
log's depth axis (TVDTL). When the segments overlay each other but not the
type log, the type log is the problem.

**No usable pilot at all** is the extreme case and works the same way: the
project bootstraps from the structure and one manual interpretation. A pilot
well with no log and no calibration takes the identity frame, and the save
creates the top-of-target marker at the interpretation's basepoint. No marker
picking, no alignment sweep first.

## The first derivation

1. **One short manual interpretation, from independent evidence.** The
   computed interpretation is (by hypothesis) compromised, so the geologist
   draws a manual one over the early lateral — long enough to capture the
   characteristic features — and adjusts it until the backprojected segments
   stack sensibly. The structural opinion comes from seismic, offset wells,
   regional dip, never from the new features themselves: wherever they then
   backproject through that structure is where they belong in the stratigraphic column. A
   straight dipping segment is fine when seismic says the stratigraphic column is straight.
   The project's dip prior is that opinion, already entered, in one of two
   forms (`read_user_prior` returns whichever the project holds, and
   `dip_type` says which the computation uses): in structure mode the
   prior structure (`read_structure`), a polyline; in constants mode there
   is no polyline, and each parameter block's apparent dip is the prior
   from that block's MD to the next block's. The tools report a block's
   dip as `apparent_dip_deg` — the number in the Dip form, 90° horizontal,
   above 90° the stratigraphic column rising ahead along the vertical
   section — with the `rise_per_100_ft` it implies (91.5° is 2.6 ft of
   rise per hundred). Draw the line at that rise, per hundred feet of
   vertical section and not of MD (through the curve the two differ, and
   a line drawn per MD rises more than the prior says) — the polyline's,
   or the block's over each block's footage, so a line under block dips
   changes dip where the blocks do — and the block's dip tolerance is the
   wiggle room you have to adjust it (the loop, step 2). The line's
   positions are `[MD, TVDSS]`, negative down: a rise of r feet per
   hundred makes the second number grow toward zero by r per hundred
   feet of vertical section (−13494 to −13493 is a foot of rise), and a
   segment whose number drops further below zero is drawn falling. A
   trial at the prior's rise that moves the log's bottom by nothing has
   been drawn falling, or parallel; check the sign before reading it as
   "no room". A brief that says
   to
   start flat, or at some dip of its own, is not independent evidence;
   the prior is the geologist's claim, and the line starts on it.
   Only the dip is the signal to the
   derivation — the computation reads the polyline as a sequence of dips
   and never its absolute depth, so the same picks shifted by hundreds of
   feet derive an identical log — but the line's depth is read by
   everything else, so draw it at the expected top-of-target depth at the
   first computed MD, not at the wellbore. The marker the first save
   creates lands at the line's depth there, so drawn at the target it is
   the top of target; and that is the frame `copy_computed_interpretation`
   writes the run's structure in (the loop, step 2) and the one the
   Profile's follow mode assumes when it hangs the target formation on a
   followed manual line. On one lateral the line was drawn at the
   wellbore, the marker was later moved 514.5 ft, and following the line
   displaced the target formation and the top of section by that amount;
   the same picks shifted 514.5 ft derived an identical log. Through the
   curve that dip is also the stratigraphic column's *scale*: the
   stratigraphic column's thickness is the wellbore's TVD descent minus the line's descent
   over the same footage (a line rising relative to the wellbore's inclination adds
   stratigraphic column, one falling relative to it removes it), so a prior dip off by a factor of two mis-scales the
   stratigraphic column by that fraction of the line's rise or fall, and every later
   estimate inherits some feet of offset at the bottom. A horizontal line is
   not a safer default — it is a dip claim nobody made, and its error is the
   whole of the prior's dip. With no pilot to check against, take the prior's
   dip and expect the lateral's re-crossings to correct the scale. There is
   nothing to try dips against here: no log exists yet, so a read-only
   derive under any dip reports geometry and nothing else, and the first
   run is the only test the first line gets on the day it is drawn.
   One later test exists. Where the well comes to ride a bed — the GR
   holding within the form's log tolerance of one value over forty feet
   or more of MD, at the landing or after it — the bed's apparent dip
   there is the wellbore's inclination over that footage, measured, and
   it is the one dip the well ever measures directly (*Riding a bed*,
   step 4 of the extension). Judge the difference by what it is worth
   over the first line, not by the size of the dip: the two dips'
   difference as rise per hundred, times the first line's own vertical
   section, is the stratigraphic column at stake, and when that exceeds
   the alarm band the first line was drawn on an opinion the well has
   since tested. Then redraw it once, from the first computed MD to the
   MD where the well came onto the bed, at the measured dip, derive bare
   and replace, Reset and Run. That is the one place the loop draws by
   hand over footage a run has computed: the run's structure there was
   solved against a log built to the first line, so it reproduces that
   line's dip and cannot supply the correction, and the log below the
   landing is that line's scale; say the two dips, the column at stake
   and the footage ridden in the entry, and from the next derivation on
   the run's structure over that footage is the line again (the loop,
   step 2). The dip sigma is the wrong gate here
   and reads as the right one: a first line runs for hundreds of feet of
   vertical section, so a difference well inside a single sigma — a
   degree or two, which no delivery's wiggle room would notice — is
   already tens of feet of column in the log. On one lateral the dip
   measured at the landing sat just inside the sigma and the first line
   it was drawn against had put twenty feet of stratigraphic column into
   the log that the rock did not have.
2. **Derive bare, from the project's own data.** Leave it un-spliced: in
   the pane the **Splice into current type log** switch stays off, through
   the tool omit the initial type log. Through the tool, name the saved
   interpretation and set `use_project_data`: the server reads the active
   log, the survey and the project's frame itself, so nothing but the name
   crosses the connector. Read the result on the type log track — orange
   correlations are the backprojected passes, the light curve is the derived
   log.
3. **Derive with Shallowest MD** (`least md`; the tool defaults to mean,
   so pass it — *The statistic*, below).
4. **Save and replace, with approval.** Writing the curve onto the pilot well
   replaces its log: destructive, so advisory-plus-approval. It keeps the
   well's calibration and metadata, creates the top-of-target marker if
   missing, and reports any marker now outside the log — a warning, not a
   deletion, and usually a sign the interpretation should have gone further.
   Set the **Depth Offset** on the Align Logs pane before the first save,
   and set it by hand: on a derived log it is not something to solve for.
   A real type log comes from another well, and the offset is the depth tie
   that carries it into this well's frame — that is what Align Logs solves.
   A derived log is already this well's own rock, so there is nothing to tie:
   its offset is the active log's own reference elevation, and set that way
   the log's depths read as this well's TVD. A pilot with no fit yet is saved
   at the identity, offset zero, which hangs the log's axis at sea level —
   right only where the elevation is zero, and anywhere else the whole log,
   its markers and every bit-against-top reading sit one ground elevation
   away from the well they describe. Set it once, on the bootstrap; later
   saves keep it.
5. **On a self-steered well, make an inadequate log fail loudly — before
   the reset, so the first run already has it.** Set the log tolerance on
   a block of the lateral's own, from the landing: the block that starts
   there when the project has one, else `add_param_block` at the landing;
   then `update_param_block` on it and on every block after it (§1.1 for
   the stored width), as tight as the log's own noise allows. The
   number is measured, not quoted: smooth the GR over about 5 ft of MD
   along the near-horizontal footage, take the rms of the samples against
   that smooth, and set the block at about three times it — 5 gAPI on a
   quiet log, 15 on one that scatters 4 gAPI rms, where 5 would be half
   the log's own residual and covered rock would fail on drift alone,
   each failure costing a full recompute. Never under 5 gAPI (a stored
   sigma of 1.7): at 3 the first run fails at its first MD before any
   footage is judged. The block starts at the landing — the first MD
   where the survey comes within about five degrees of horizontal and
   stays there — and never earlier. A first delivery whose bit is still
   in the curve waits: add the block on the delivery the well lands, at
   the landing, before that delivery's run; until then the curve runs at
   the usual tolerance with its alarm muted, and nothing on that footage
   is the lateral's to judge. The log was recorded by this
   bit, so in rock it holds the measurement matches it to within that
   noise, and a tolerance that tight leaves the computation no depth for
   footage it does not hold: instead of piling up quietly at an end the
   run ends in error and the job status reads **Impossible**, the alarm
   in its hard form (*Reading the state*). The curve's block stays at the
   usual tolerance: through the build the computation smooths the GR
   over several feet of TVD while the derivation filed the raw samples,
   and where the well descends fast that alone differs by more than a
   few gAPI. An *Impossible* whose last MD lies in the curve under the
   tight block is that — the block's start, not the log: move the start
   to the landing (`move_param_block`) and rerun. Never delete the block
   or loosen it for that error. The ride test (the extension, step 4)
   runs at this block's tolerance, and at the usual one every delivery
   reads as a ride: on one lateral the block was deleted for an
   *Impossible* at the heel, the lateral ran at the form's 30 gAPI, every
   delivery rode, the line ran parallel for 1,400 ft and the log ended
   6 ft short of the rock.
6. **Reset, rerun, restore ingestion.** A replaced type log invalidates all
   saved computation: `reset_job` then `trigger_job_rerun`, with approval,
   recomputing the well from scratch in typically minutes. On a WITSML
   project pause both pollers first and **re-enable them afterwards**
   (§1.9.6); an email-fed project has nothing to pause.

The derived log's depth axis is anchored so the wellbore at the first computed
position sits at its own depth, and the interpretation's depth there is where
the top-of-target marker lands when the save creates one; an existing marker
keeps its depth. That is why markers keep their meaning across a replacement
and no re-pick is needed. The bit against the top of target is one
subtraction: the bit's own position in the stratigraphic column,
`coverage.last.tvdtl_mpe` on the latest run, minus the top-of-target
marker's depth (`tot_name` from the project, its depth from the pilot
well's markers, read after the save that creates it), both on the derived
log's own axis, TVDTL, positive-down: the type log's own TVD. A derived
log is this well's own rock, so there is no second well here, but its
axis is hung at sea level by the identity fit, and the well's own TVD is
counted from its datum elevation — the same kind of number from a
different zero. Subtract on one axis, never across two, and a positive
answer is the bit below the top. An auto-picked group's terminal depth against the survey,
the log's top or bottom, and the coverage margin are not that number (an
alternative's depth is still stated against the marker). When this reading
moves more than the well did between deliveries, the computation's
correlation moved, not the well: say so.

### The statistic

Shallowest MD, always (`least md`; the tool defaults to mean, so pass it).
A self-steered well assumes the stratigraphic column is the same at every
lateral position, so every pass through a stratigraphic depth sees the
same rock and the first pass is as good as any. Shallowest MD keeps the
first sample in MD order at each depth: wherever the line is the same as
at the last derivation a re-derivation gives those depths back unchanged,
and a depth changes only where the line over the footage that files it
moved — the run's structure revised since the last derivation, or the new
footage reaching depths no pass had. That matters because every
replace recomputes the whole well against the new log. Mean and median let
a later pass at the same depth — hundreds of samples on a long horizontal
stretch — outvote or dilute the first, so if the stratigraphic column ever
does change along the lateral they rewrite depths whose structure was
already right and the recompute moves it; under Shallowest MD the change
shows where it belongs, as a residual over the new footage, and the log
before it stands. The other statistics are for a lateral whose
stratigraphic column does vary, out of scope here (that case takes several
pilots); a disagreement between passes is not a reason to reach for them.

## The loop

Every later cycle is the same four moves:

1. **Read the alarm — the whole block, not the reason.** The coverage
   block on the latest job result says whether the well has drilled past
   an end of the log, which end, and the first MD where it showed. That
   MD is where the extension starts: the first MD of the 2 ft alarm band,
   never the first MD of the 10 ft warning band, where the well is still
   in covered rock. An alarm whose first MD is the bit itself has no
   footage past it: the line is the run's picks to the bit, saved, and
   the derivation waits for the next delivery, since a derive through
   picks the run kept inside the log adds no depth and costs a recompute.
   Read the margins and the mass at
   each end beside the reason (*Reading the state*: an end approached
   but not yet reached reports as entropy), and diff this run's
   structure against the last run's over footage both were confident
   about: a revision of ten feet or more where nothing new was drilled
   is the look-alike's tell, and the alarm does not see it. So is a run
   that puts the bit shallower in the log than the prior predicts by
   more than the wiggle room, and that check is arithmetic, made on
   every delivery until the next derivation: from the last derivation's
   end the bit descends into the stratigraphic column at the prior's
   rise minus the wellbore's, per hundred feet, times the footage since
   (the survey gives the wellbore's; a flat well under a prior of 2 ft
   per hundred is 2 ft deeper after a hundred feet, past the end of a
   log that ended at it). Read `coverage.last.tvdtl_mpe` against that
   prediction; shallower by more than the block's dip sigma as rise per
   hundred times the same footage — the stored sigma, a third of the
   tolerance the Job Parameters form shows, so a sigma of 2° is 3.5 ft
   per hundred and 3.3 ft on a 94 ft delivery, never the form's 6° —
   and the run has lifted the well out
   of new rock onto a match higher in the log, usually by re-drawing
   the last delivery's footage a few feet lower: the margin that
   reopened is the look-alike, not covered rock, and a quiet alarm says
   nothing about it. The footage since the derivation is then the
   extension (step 2) at the prior's dip, derived and run, and the
   deliveries after it judge. A lateral
   block still at its default log tolerance mutes the alarm — smeared
   cells near the passes' mean then match every pass — so a quiet block
   over such footage says nothing until the tolerance is set (the first
   derivation, step 5) and the well reset and run.
2. **Rebuild the ONE manual interpretation to the bit.** The line is
   rebuilt at every derivation, never extended: over the footage a run
   has computed it is the latest run's structure, taken whole, and past
   that it is drawn by hand. It keeps its name — `derive_type_log` names
   it, the cross section follows it — and its picks are replaced
   (`update_interpretation` with the whole block geometry). Whatever
   structure files the samples is frozen into the log, a hand line or
   the run's alike, and only the well re-crossing distinctive rock
   corrects either; over covered footage the run's structure is the
   better one, since it used the data and the log, and the hand line
   knows something the run does not in exactly two places: before the
   first run exists, and past the end of the log. A line kept by hand
   over covered footage drifts off the run with nothing to pull them
   together — on one lateral the run walked 6 to 10 ft off a hand line
   held from MD 13,786 to 15,050 across a derivation, so the log was
   built through the line while the type-log track backprojected the
   passes through the run, 6 to 10 ft apart — and a line stitched from
   runs keeps each run's picks over its own footage after a later run has
   revised them (on one lateral picks appended from one run were revised
   8 ft deeper by the next, and stitching the next's after them left an
   8 ft step in the line). The log carries the line's scale over
   footage the well only deepened through, and no run rescales it (on one
   blind lateral a first line at twice the true dip squeezed the
   stratigraphic column 4%, and the estimate over the next 2,000 ft
   carried 4–6 ft of it): a first line drawn at the wrong dip is repaired
   by the landing-ride redraw (the first derivation, step 1) or by a new
   first derivation from independent evidence, which rewrites the whole
   log and is proposed as that, never from inside the loop. The line
   comes in two stretches, and one construction governs both: a sample
   lands in the stratigraphic column at its vertical distance from the
   line, so the line's depth at an MD is fixed by the stratigraphic
   column depth you believe that MD's GR must sit at.
   - *Computed footage*, from the first computed MD to the alarm's first
     MD — to the bit when the alarm is at the bit, to the reach-back MD
     on a look-alike (*Between derivations*), and after an *Impossible*
     the last completed run's structure to the failure MD. The run's
     structure there **is** the line, never hand-edited:
     `copy_computed_interpretation` takes it whole, hung at the top of
     target, the frame the line was drawn in (the first derivation,
     step 1), and reports the shift (`tot_shift_tvdss`, hundreds of
     feet); `read_mpe_slice` serves the same picks in the top-of-section
     frame and needs that shift applied. Cut them at the alarm's first
     MD. Check the first hand pick against the run's last before saving:
     a step of hundreds of feet is the other frame. Over the footage
     since the last derivation the run is a correlation of new footage
     against the stratigraphic column, not an echo, all the way to the
     alarm's first MD; the last delivery or two before it get revised by
     several feet as the next delivery lands, which the next rebuild
     takes, and they are not yours to replace with picks of your own:
     that footage is in the log and is owed no room. Restarting from the
     last derivation's pick and carrying the prior across this footage
     throws the correlation away — on one lateral every cycle's extension
     began at that pick, and the tens of feet of correlated footage
     between it and the first position the run had piled up at the
     bottom went to the prior each time. Before the rebuild read the
     run's structure against the saved line over the covered footage
     (`read_mpe_slice` against `read_interpretation`, or the cross
     section) and say the largest departure and its MD in the entry:
     that is the revision this derivation writes into the log.
   - *The new footage, from the alarm's first MD to the bit* — the
     extension, a delivery or a few on a live feed, and the only footage
     the procedure below is applied to. The run's structure there is the
     **deepest** (for a bottom alarm) structure that still keeps the well
     inside the log, never the truth, and it bends toward the wellbore.
     Carry none of it, and do not draw the line so the well just reaches
     the end (on one lateral that left the well 5 ft below the log when it
     had drilled 24 ft of new rock). From the run's last pick before the
     alarm the line's dip is chosen by the procedure below, and a trial is
     tested by deriving read-only through it (a trial interpretation
     document rides into `derive_type_log` alongside `use_project_data`).
     The room a cycle adds is what this footage crosses under the line —
     the well's descent relative to the line's dip over it, a few feet per
     delivery on a near-horizontal well — and a line that adds tens of
     feet from one delivery left the run's structure early and filed
     covered rock as new.

   **Choosing the dip of an extension.** The line is a prediction: by the
   construction above it says what GR the bit should have measured at
   every MD — the derived log read back at that MD's depth in the
   stratigraphic column — and once the log is saved the active log track
   draws exactly that, the black curve against the measured passes
   (*When to start*). On a self-steered well they are the same samples,
   filed into the stratigraphic column by the line and read back along
   it, so only the line stands between them: detail may drift as the bit
   moves away from the rock the log was built from, and more with
   distance, but the character stays. A line that lays a long stretch of
   MD across a thin slice of the stratigraphic column files hundreds of
   varying samples at a few depths, and the log keeps only the first at
   each: the run then finds the rest elsewhere in the log, drawing
   structure the rock does not have, or fails there. So before saving, with
   the survey (`read_active_trajectory`), the GR (`read_active_log`), the
   prior (the polyline from `read_structure`, or in constants mode the
   apparent dip of the block covering the new footage) and the block's dip
   tolerance (`read_job_params`):
   1. *Measure the well.* The wellbore's rise per hundred feet of MD over
      the new footage. It is not a candidate dip; it is what every
      candidate is measured against.
   2. *Count the beds.* First read the log over the depths the prior's
      dip files the new footage at (`read_pilot_log` over that range);
      only then count. A bed's thickness is the stratigraphic column its
      own footage crosses under the prior's dip, by the scale rule of
      step 3 and from the survey, never a number assumed; an excursion the
      prior gives under a foot is a bed of a foot. Each clean or hot excursion in the new GR larger
      than the block's log tolerance, with different rock between, is a
      bed, and it is owed room unless the log **as it stood before this
      extension** holds a matching excursion within a bed's thickness of
      the depth the prior's dip files it: then it is the same rock varying
      along the lateral, the log keeps its value there, the tolerance
      carries the residual, and it is owed no room. The log's average
      level is not a match, a cell the last extension filed from long
      footage is not covered rock, and at the default tolerance everything
      matches and nothing counts — which is why the tolerance is set at
      the first derivation (step 5), before the loop begins. On one
      lateral five extensions in a row counted no beds against a log
      whose cells held the passes' own mean, while the passes swung
      30 gAPI. Unlike rock at one depth is the smear; the same rock at
      another level is not.
   3. *Compute the room each candidate gives.* Stratigraphic column
      crossed is the line's rise minus the wellbore's, per hundred feet,
      times the footage (the scale rule of the first derivation). The
      room is the extension's own, from the run's last pick; what the
      run's picks did to the log's end upstream is theirs, not the
      extension's to make up. A trial measures the extension alone: the
      run's picks are the line to the alarm's first MD and are derived
      through once, and each candidate's room is its end against that
      derive, never against the log as saved. The
      candidates are the prior's dip and moves off it within the wiggle
      room — the prior carries dip only, polyline or block constant, and
      a block boundary inside the footage changes the prior's dip there —
      and horizontal, only
      where the well re-crossed the same rock at the same depth.
   4. *Reject candidates without room.* The least stratigraphic column
      an extension needs is a bed's thickness, measured (step 2), for
      each bed counted, and with none counted it is whatever the prior's
      dip gives: on a near-horizontal well under a prior of a degree or
      two that is a foot or two per delivery, and that is the column the
      footage crossed. The alarm band is a detection threshold, not a
      floor. A well drilling down-section sits at the log's end on every
      delivery whatever the last extension added, so the alarm returning
      on the next delivery says the well is still descending, not that
      the extension was short (on one lateral a floor of a few feet per
      delivery over a true two per hundred put in column the well never
      crossed, three deliveries running).
      What rejects a candidate is unlike rock at one depth: beds it
      leaves no room for, or a GR swinging beyond the tolerance filed
      into a cell or two — the prior is rejected like any other, and on
      one lateral it was kept five times while three clean spikes
      130–180 ft apart with hot shale between went into one depth, a bed
      each unroomed. With no bed counted and the prior giving the footage
      less than nothing — the well climbing faster than the prior's rise
      while the alarm holds it at the bottom — the beds rise at least as
      fast as the well: the line runs parallel to the well, the smallest
      claim the alarm allows, and it grows nothing. Save that line and
      derive nothing (it grows the log by nothing), no reset;
      the well is riding the log's end in covered rock, the alarm stands,
      and the next bed or excursion is what moves the line. Under a GR
      that keeps swinging beyond the tolerance a parallel line is the
      smear, and the beds it files at one depth are the room it owes.
      *Riding a bed.* A GR that holds within the form's log tolerance of
      one value over forty feet or more of the new footage is the well
      riding one bed, and the bed's dip over that footage is the
      wellbore's inclination there, measured. The run reports the test:
      `coverage.new_footage` gives `riding_a_bed` over the footage since
      the last derivation, with `gr_max_dev` — the largest departure of
      any of the log's own intervals from their mean — beside the
      `interval_ft` they were taken at. Say the spread and the feet in
      the entry. Do not retake it from the raw log foot by foot: a
      logging tool's foot-to-foot scatter is its own and not the rock's,
      and against it the same footage reads as one bed or as none
      depending on where the tolerance happens to sit. Whatever the prior says, the line
      runs parallel to the well over that footage and grows nothing. The
      measured dip governs the footage it was measured over and no
      further: past the bed the prior stands again. It is the rock and
      the prior is a claim, but it is the rock the well lay along, and
      carried into footage the well crosses at another inclination it
      takes stratigraphic column out of the log — on one lateral a dip
      measured along a bed at the landing, carried through a lateral
      that climbed away from it, subtracted ten feet of column from a
      log the rock owed nine. Say in every entry which dip the line is
      on and, where a bed was ridden, over what footage. Footage shorter
      than forty feet, or a GR that leaves the tolerance, measures
      nothing: no bed is counted, and the prior gives the room. The
      test's footage is the run's, since the last derivation to the
      bit, and a ride derives nothing, so while it lasts that footage
      grows a delivery at a time and its spread with it; the ride ends
      on the delivery the spread leaves the tolerance. The footage
      ridden stands as drawn, parallel through the last delivery that
      reported the ride, and the prior gives the room from that
      delivery's bit — never further back, since a line redrawn at the
      prior over ridden footage puts in column the well measured as
      absent. A fresh alarm MD while the ride holds is the same alarm,
      the well at the log's end in covered rock, and gets the same
      answer: the run's picks to the alarm's MD, the parallel line to
      the bit, saved, nothing derived, no reset (on one lateral three
      full recomputes through a parallel line moved the log's end by
      nothing). At a tolerance never set (the first derivation, step 5)
      the test passes everything.
   5. *Place the features and check them against the log at that depth.*
      A clean spike must land on a clean bed the log holds, or past an
      end. Landing on shale in covered rock rejects the candidate. Past
      the end is allowed and expected: that is the alarm arriving, and the
      tight log tolerance (the first derivation, step 5) makes the
      computation run this check itself.
   6. *Take the survivor nearest the prior.* Where every feature lands on
      matching rock under the prior, that is the prior itself, and the
      line does not move. If nothing survives inside
      the wiggle room the prior is wrong here: go past it in the direction
      the beds demand, and say so. The alarm alone is reason to move and
      the beds say how far; with none to count — a featureless GR — the
      prior stands with whatever room it gives, and where it gives less
      than nothing, or the well is riding a bed, the line runs parallel
      to the well (step 4), never a
      foot per hundred squeezed from a ladder of trials.
   7. *Audit the dip change against the survey.* The line changes dip on
      the rock's evidence only. A change that coincides with a survey
      inclination change and nothing else is the line following the well.
   8. *Derive read-only through the choice, save, run, read the verdict.*
      An end that did not move, or moved by less than the beds counted
      need, or a pass scattering
      sideways at one depth on the type log track with the cross section
      following the trial line, sends you back to step 3; an end that
      moved by more than the room step 3 gave the choice says the line
      moved before the alarm's first MD, and sends you back to the run's
      structure. After the run,
      an alarm or an *Impossible* means extend further along the same dip;
      new footage inside the log with matching character means keep.
   Read-only derives may measure the candidates — a trial through each
   reports the room it gives (its end movement, step 3) and, with the
   samples, where its features land (step 5) — and a trial rides inline
   (a trial document alongside `use_project_data`): one that names the
   saved interpretation has edited the line, and the line is edited
   once, for the survivor. They do not choose:
   the choice is still the survivor nearest the prior (step 6). A ladder
   that keeps the dip whose end moves most always keeps the steepest, and
   one that keeps the dip whose new samples best match the covered rock
   has correlated the new GR against the log to find structure, the
   computation's own job, and the run then confirms a log built to match
   it. Each extension's dip is chosen afresh from the prior over its
   footage; the line has no standing dip it "now runs at", and a move
   past the wiggle room is said aloud each time it is made, never carried
   into the next.
   On one lateral fed about a hundred feet at a time the survey eased
   twice, from 12 to 8 ft of rise per hundred, and the line eased with it
   both times until it ran parallel to the well: ten deliveries in a row
   cleared the alarm band while 1,000 ft of MD went into less than half a
   foot of stratigraphic column, and three clean spikes 130–180 ft apart,
   with hot shale between, were filed at one depth. The prior read 12.5
   the whole way. Three beds over 300 ft need 10–15 ft of room, so the
   line had to rise at least 11 per hundred; the geologist's answer was
   11.3, and under it the spikes are three beds 4–6 ft apart.
   It is a guess and is meant to be; the next run tests it. Stop at the bit.
   The run's structure over the new footage is not the line there: it was
   confined to the log, so a derive through it grows nothing (*The end
   will not move*), and the next derivation takes what the runs since
   have made of that footage.
3. **Derive bare again and replace.** Shallowest MD again, same approval,
   same save. Over covered footage the new log differs from the previous
   one wherever the run moved its structure since the last derivation:
   that is the design, not an error, and the entry carries the largest
   such revision (step 2). The end moves by at most the room the
   extension's geometry gives (the line's rise minus the wellbore's,
   times the footage, step 2's procedure): a derive that moves it further
   has filed stratigraphic column the well never crossed, whatever
   produced it, and is not saved.
4. **Reset and rerun.** The replaced log invalidates the saved
   computation, so this run is Reset and Run (`reset_job`, then
   `trigger_job_rerun`). On the next run the alarm should clear and the
   margins reopen. That run also tests the extension: where its structure
   leaves the line over the extension's footage, on the cross section or
   in the MPE slice, say the feet in the entry, and the next derivation
   takes the run's structure there like any computed footage — the
   depths the extension filed that no pass reaches through the run's
   structure drop then, under Shallowest MD — and nothing is derived for
   that alone. An alarm whose first MD lies inside the last extension is
   the ordinary move: the run's structure to that MD, the hand line from
   it.

Between derivations a delivery whose run raises no alarm is first the
prior check of the loop's step 1, every time and before anything else.
The run reports it, so read it rather than rebuild it:
`coverage.prior_check` gives the bit against where the prior puts it
since the last derivation as `shallower_than_prior_ft` — positive for
shallower, less column drilled than the prior expected — beside the
`room_ft` the stored dip sigma allows over that footage, and
`look_alike` when the one exceeds the other. Say both numbers in the
entry. Where the block prior is not what governs, the field comes back
absent and the check is yours to make on the same terms. Over footage
the well is riding (the extension, step 4) the prior is not the
predictor: the parallel line puts the bit at the log's bottom still,
so read `coverage.last.tvdtl_mpe` against `coverage.log.bottom` on that
axis instead, and shallower than that by more than the room is the
lift; against the prior the number only grows with the ride, by the
column the ride declined to add. Shallower by
more than that room is the look-alike: the run has
explained the new GR by lifting the well up the log over footage it had
already placed, and the other explanation — the well deeper in section
from where the lifting began, the new GR as new rock below the log — is
the one the prior supports. Both fit the GR, so the prior is the only
arbiter the well offers, and this delivery reaches back and extends at
the prior; the runs that follow test that extension, and the next
derivation takes what they made of it. How far back is measured, not
chosen: read the run's structure against the line over the covered
footage (`read_mpe_slice` against the saved interpretation), from the
bit backward, and the reach-back MD is the last one where the two
still agree within the alarm band, with the footage before it agreeing
too. Usually that is the last derivation's end, where the well sat at
the log's bottom and the run had nothing to lift; it can lie earlier,
inside footage the last derivation took from an earlier run, and then
this run has contradicted that one there and the rebuild follows this
one to the reach-back MD.
The run reports that MD as `coverage.reach_back.md`, with `basis`
saying how it was found or why it could not be; an absent one is read
off the slice as above, not guessed at.
From the reach-back MD the line runs at the prior's dip to the bit,
through the run's picks up to it and none after; derive bare and
replace (under Shallowest MD the depths the lifted footage used to
fill drop out and the footage files below the log's end instead),
Reset and Run, and say the reach-back MD and the feet in the entry.
Reaching back further than the departure is filing covered rock as
new, and reaching back less leaves the lift in place; the departure
itself is the only measure. A drift within the room is not this: it is
the prior's own uncertainty, undetectable from inside the well, and
the line carries it. A quiet
alarm does not excuse the check, since the look-alike is what a quiet
alarm looks like. Within the room, read the run's structure over the
footage since the last derivation against the line (`read_mpe_slice`
over those MDs, or the cross section) and say the feet in the entry on
every alarm-free delivery: where it stands off the line is what the
next derivation writes into the log, and nothing is derived for it
alone. A delivery that extends the
active log and survey and leaves the type log as it is — on a replay fed
segment by segment as much as on a live feed — runs as a plain **Extend**
(`trigger_job_rerun` alone): it computes only the footage past the pointer
on the saved state and reaches the same result as a full recompute in a
fraction of the time. Reset and Run is the price of a modified type log,
paid once per derivation; a log you have not touched never needs it.

Then keep watching. **You have no background process** (§1.1): monitoring
means reading the coverage block whenever the geologist brings you a run, and
offering to turn on the coverage notification (`set_notify_flags`) so Drive
emails them when a run raises the alarm.

On a live feed the rhythm matters. Real deliveries are small — a dozen to a
hundred feet per ten-minute WITSML poll, about ninety feet per email, which
is the common case — so after a derivation the alarm typically arrives a few
deliveries later and each cycle extends the line by a few hundred feet at
most. A delivery's own run starts a minute or two after it lands, and a
queued run reads as `not_run` with an end timestamp; `read_latest_job_result`
shows the last **completed** run, so match its run id to the delivery before
judging the newest footage. Pause polling for the whole cycle (extend,
derive, reset, rerun) and resume after, or the next delivery cancels the run
you started; the full recompute after a replacement is about a minute per
few thousand feet of lateral on the standard tier. A poll that lands while
the previous delivery's run is still queued cancels that run, so a delivery
can go uncomputed until the next one; `read_job_status` reads `not_run`
with the newer end timestamp. A run that stays `not_run` for several
minutes has stalled in the compute queue, and a plain `trigger_job_rerun` is
then refused as underlapping because the stalled job already holds the
pointer: pause polling, `reset_job`, `trigger_job_rerun`.

## Reading the state

The coverage block reads differently at each stage, and misreading it is the
main way to get out of step:

- **Freshly derived.** The log's bottom sits exactly on the deepest pass *by
  construction*, so raw margins near zero mean nothing. The margins that
  matter are measured only over footage drilled **since** the derivation;
  until there is some, the block says so. Not an alarm. The entropy leg
  has no such gate and can raise the alarm on the first run after a
  derivation, before any new footage exists: not an alarm either, and it
  clears on the next delivery. The tool's `alarm.rederive` can read true
  on that run, with `post_derivation` null: that is the artifact, not an
  alarm, and nothing is derived until footage drilled since the
  derivation has been run.
- **Healthy.** Tens of feet of log below and above the estimate at the bit,
  entropy near its own baseline, marginals tight and single-peaked, a steady
  uncertainty corridor on the cross section, and on the active log track a
  black curve that keeps the passes' character over the footage drilled
  since the derivation.
- **Running out.** Over the recent footage drilled since the derivation the
  estimate sits within 2 ft of an end, or half the posterior mass piles into
  that band; or recent entropy runs about twice its baseline. The
  uncertainty band blooms toward the toe. The alarm names the end (bottom =
  drilled stratigraphically deeper, top = shallower) or entropy, and the MD
  where the pile-up began; it clears on its own once the well has climbed
  away. Short of that the block reports a 10 ft *warning band*: a log
  derived right after the curve has only a few feet of lateral-derived
  stratigraphic column at its bottom, and a well riding that zone shows a few feet of log
  below the estimate, all of the mass "near" the end, and a summary reading
  "near the bottom ... not pressed" for as long as it stays there. That is
  a *near-end warning* — the well is in covered rock and there is nothing
  to derive, since the log only grows when the well samples new rock. When
  in doubt read the marginals: tight, single-peaked, the estimate a few
  feet clear of the end, entropy near its baseline is the warning. Mass at
  the wall, the estimate at the end, modes splitting, GR the log has no
  match for (hotter or cleaner than anything in the band, or the black
  curve on the active log track running nearly constant while the passes
  keep swinging) is the *excursion*, and the cue to extend. Mass in the
  warning band climbing toward all of it over successive deliveries,
  the estimate still settled and single-peaked, is the well closing on
  an end and not yet past it: nothing to derive, since the run has not
  put the well past the end, but the moment to say so and to have the
  extension's dip already chosen (the loop, step 2) so the cycle runs on
  the delivery the alarm fires. It may also resolve on its own — the
  structure falls away, the margin reopens — and an alarm that clears
  with nothing done is exactly that.
- **Entropy.** The entropy leg fires when the marginals over the bit's
  last few positions carry about twice the entropy of the run's earlier
  ones, and it is reported as the reason only when neither end has been
  reached (the estimate within 2 ft of an end, or half the mass there).
  So an entropy alarm is a symptom with three causes, and the margins
  tell them apart. Mass piling toward an end — most of it inside the
  warning band, some at the wall (`mass_at_bottom` of a few percent or
  more; under one percent with the mass in the band is the well closing
  on the end, nothing to derive), the estimate a few feet off the end —
  is that end arriving before its own leg fires: the computation is
  clipping the posterior there, and the clipping is what flattens it.
  Read it as the end alarm. The bit tens of feet clear of both ends, one
  mode carrying most of the mass, a GR with no countable bed, is
  featureless rock: the posterior spreads because nothing in the log pins
  it, not because the log is wrong where the bit is, and it clears by
  itself when the next bed arrives (one lateral raised it six times over
  40 bedless deliveries, and every one cleared on the next delivery
  without a derivation). The test, when it matters: derive read-only
  from the footage drilled since the derivation alone, line its curve up
  with the project's log on the same depth axis, and compare the two
  over the depths the bit occupies — a match within a few gAPI says the
  log is right there and there is nothing to derive. Confident footage
  revised by ten feet or more with the structure moving in step with the
  wellbore is the look-alike (below).
- **Impossible.** The run ends in error and the job status reads
  *Impossible*: at some position no structure within the tolerances puts
  the measured GR anywhere the log holds (`read_job_status` carries the
  reason and the last MD each pass reached). With the tight tolerance of a
  self-steered well that is the alarm in its hard form — the log is
  inadequate past that MD, not deep enough or no longer representative of
  the rock the bit is in — and the answer is the loop, step 2, not a
  looser tolerance (pitfall 5). A failed run leaves no picks over the new
  footage, so the rebuild takes the last completed run's structure to the
  failure MD and the hand line from there; and a failure in rock the log
  already holds says the line filed the new footage at the wrong depth —
  the same step, not another statistic.
- **After a good re-derivation.** Margins reopen and entropy drops back. Fit
  over the footage drilled *since* the previous derivation is a real test of
  that cycle's speculation; fit over covered footage tests nothing,
  since the log was built to match it. The toe of a fresh log is its
  least trustworthy stretch: the log there came through the speculative
  part of the line, so the bit marginal often splits between your line
  and an alternative a few tens of feet away (P1 against P2). Do not
  choose between them by hand: the next derivation takes the run's
  structure there (the loop, step 2), and the next footage decides.
- **Back in covered rock.** Footage drilled since the derivation that fits
  inside the log — margins in the tens of feet, tight single-peaked
  marginals, no alarm — is the loop's success case, even when the estimate
  sits tens of feet from your speculative line: the well came back into rock
  the log already holds, and the computation is correcting the guess. Not
  when the bit sits shallower than the prior's prediction from the last
  derivation by more than the wiggle room: that is the look-alike (the
  loop, step 1), whatever the alarm says. Accept
  the run — with the active log track in view: a black curve gone nearly
  constant under swinging passes over the new footage is an earlier cycle's
  smear, not success, and at a tolerance never set the run cannot say so
  (the first derivation, step 5). The guess comes out of the log at the
  next derivation, which takes the run's structure over the extension
  and drops the depths the well never reached; re-derive only on the
  alarm or the look-alike.
- **Stuck at an end.** The same alarm on consecutive runs, a re-derivation
  between each, and the log's end not moving, the line drawn parallel to
  the well while the GR keeps swinging: that is the loop spinning, not
  the well plateauing (an end that moves a foot or two per cycle under
  the prior's dip on a near-horizontal well is the well descending, and
  the same alarm each delivery is what that looks like). The bit
  marginal says the same — its support stops at the wall and the main
  mode's range ends on it (on one run 47% of the mass sat in the last 3 ft
  above the bottom). Two readings tempt you to wait it out, and both are
  wrong. "The computed top agrees with my line within a foot" is forced,
  not evidence: piled up at the bottom, the run cannot put the well any
  deeper than the log allows, so its top of target lands wherever the
  wall puts it — a foot or two below the wellbore, which is exactly where
  a line drawn parallel to the well already sits. And "the well is above
  the top of target, so the stratigraphic column cannot extend below it" mistakes the
  line for a floor: the top-of-target marker is where the line's
  basepoint landed in the stratigraphic column and nothing more; the stratigraphic column's bottom is
  wherever the deepest sample landed, the well can be below the line, and
  new rock is stratigraphic column below the marker. Waiting for the survey to carry
  the well "into the target" does not fix it either — the well is in new
  rock now, every run until then is wrong at the bit, and the stratigraphic column will
  not grow until the line changes. Fix the line (step 2 of the loop).
- **The end will not move.** A read-only derive through an extension
  whose end does not move (pitfall 12) is a verdict
  on the line, never on the log. The log's extent is the line's: its top
  is the wellbore's own depth at the first computed position, its bottom
  the deepest stratigraphic depth any sample landed at along the line,
  and nothing else bounds it — not the markers, not the footage drilled,
  not how many times it has been derived. An extension spliced from the
  run's structure cannot move the end, by construction: the run was
  confined to the log, so every one of its picks keeps the well inside
  it, and a derive through them adds no stratigraphic column however far
  the well has drilled. One lateral concluded from three such tests that
  the log's window was fixed by the markers and that running out was not
  recoverable inside the loop; it was the line. Over footage the run
  piled up, only picks of your own, at a dip that puts the footage past
  the end (step 2), grow the log.

Any type log can run out this way, derived or not — an original pilot ending a
couple of feet below the deepest depth the lateral reaches raises the same
alarm. The alarm also has a blind spot: new rock that mimics a feature already
in the log, within reach of the dip prior, stays confidently wrong. Treat a
clear alarm as reliable and a quiet one as "no evidence of trouble".

The look-alike has a signature of its own. A toe piled up at the bottom that the next
delivery "resolves" — confidence back to a single peak, entropy down — by
revising already-confident footage by ten or more feet, or by drawing a fold
the prior structure does not have (over one delivery, a departure from the
prior's dip beyond the wiggle room), has most likely matched new rock to a bed
higher in the log; and when the well then climbs and the structure rises in
lock-step with it, so that the well never leaves that bed, the structure is
following the wellbore. On one lateral the computation drew a 20-ft syncline
in 1,500 ft that way. The test is cheap: extend the line only through the
footage that piled up, derive, and let the footage after it judge — the true
stratigraphic column correlates the climb with a smooth structure and the well moving
through the beds; the look-alike needs the structure to chase the well.

There is a deeper limit to know about. The well can only *scale* its own
stratigraphic column where it crosses the same stratigraphy more than once: a wrong dip
over a stretch that only ever deepens produces a log that is stretched or
squeezed there, the computation reproduces the line that made it, and no
alarm and no conflict ever shows. On a lateral that walked 60 ft
down-section past its first derivation without coming back, a prior-structure
extension left the estimate more than 100 ft from the answer at the toe, and
nothing in the run said so. Over such footage the dip has to come from
outside the well — the prior structure, offsets, seismic, or a correlation of
the new footage against the original pilot (the retained reference log is
there for exactly this) — or from the one place the well measures it,
the footage where it rides a bed (the extension, step 4, and the first
derivation's later test) — and a line lifted from a run against a
*differently derived* log does not transfer either, since that log had its
own scale. Where the well does come back into covered rock the computation
corrects the guess, and the loop converges. The well *can* scale a stretch
where it crosses the same rock twice at the same depth — the same hot bed
at the same TVDSS 500 ft apart with different rock between says the
structure is horizontal there — and that is direct evidence, worth more than the
prior over that footage. In featureless rock the prior is all the line
has, and the structure can change sign under it unseen: on one lateral
the stratigraphic column rose 12 ft per hundred, crested, and fell 4 per
hundred, and 40 deliveries of bedless shale gave nothing to count. The
crest was found late, by the run in covered rock (the loop's success
case), never by the line. When the GR gives nothing, say the prior is
being carried rather than confirmed; do not tune.

Run blind on a 10,000-ft lateral with no pilot at all (prior polyline plus
the well's own GR, three derivations), the loop finished within 5 ft of the
geologist's answer over half the well and within 10 ft over five sixths of
it, 3.5 ft at TD; the rest was one 2,000-ft stretch 13–20 ft off, the
footage where the well drilled 24 ft of rock the stratigraphic column never held and the
extension put it 5 ft below the bottom instead. That is the shape of the
limit: right wherever the well re-crossed its own stratigraphic column, and off by
whatever the speculation missed where it did not.

## The advisory reference log

Replacing the type log does not discard the original: the first save shelves
it as a **reference log**, an advisory copy nothing that computes ever reads —
not the job builder, not the type-log interpolation, not the cross section. It
is not a pilot well and it invalidates nothing. Later cycles shelve nothing,
since the log they replace is itself derived.

Use it for general-shape reasoning: roughly where in the stratigraphic column the wellbore
sits, what the units above and below look like regionally. Never cite it as an
input to a run, never transfer its marker depths, and name which log a claim
came from when it came from the reference.

## Driving vs coaching

Default to volunteering. Only the hand-drawn extension and the approvals
are judgment. The save goes straight through the connector, so
a whole cycle — derive, save, reset, rerun, resume polling — is yours to offer
end to end, with no file round-trip and no browser needed.

With browser tools you can also work the Derived pane itself and click
through the methods to compare; without them, coach the same sequence by pane
and control name. Either way user-facing language stays plain: "replace the
type log with one derived from the well itself and re-run", never tool or
field names.

## Pitfalls

1. **Drawing by hand over computed footage.** From the first computed MD
   to the alarm's first MD the line is the latest run's structure, taken
   whole at every derivation (the loop, step 2): picks of your own there,
   the last derivation's line held where the run has since moved off it,
   or the prior carried from the last derivation's pick, all put a
   structure the run has revised into the log, and restarting from that
   pick discards the correlation the run made of the footage since. The
   one exception is the landing-ride redraw of the first line (the first
   derivation, step 1). Check what the cross section is following first.
2. **Splicing.** Leave the splice switch off. Spliced values are unconfirmed
   values at depths this well never visited, and the computation reads them as
   ground truth. Splicing also silences the alarm, which is the loop's clock.
3. **Extending past the bit, or extending to quiet the alarm.** The extension
   covers footage actually drilled and stops at the bit. Speculate about
   structure, never about stratigraphy the well has not sampled. The line
   is not an input to the computation unless the project's extend target
   names it, so extending it without a derivation changes nothing in the
   next run: a quieter alarm on the delivery after an extension alone is
   the rock, not the line, and no evidence for it (on one lateral the
   entropy halved for one delivery after a 2,300 ft extension and came
   back higher on the next).
4. **Late derivation.** A log derived at the toe explains footage already
   drilled. Raise the option as soon as the pattern appears.
5. **Treating the alarm as an error.** It is the design working: the well
   found rock the log doesn't cover yet (an *Impossible* on a self-steered
   well says the same). Report it that way and re-derive;
   don't tune dip or log sigma to paper over it, and don't delete the
   tight block for an *Impossible* in the curve — its start moves to the
   landing (the first derivation, step 5).
6. **Resetting the calibration on replace.** The save writes in the pilot's
   own frame, and preserving it is what keeps the markers where they are.
   Re-run alignment only if a warp was actually in use (replacing drops it).
7. **Forgetting to restore ingestion.** A project left with its pollers
   paused after a manual rerun quietly stops being steered.
8. **Deriving with mean or median.** A later pass at a depth the log holds
   then rewrites it, and every replace recomputes the whole well against
   the rewritten log. Shallowest MD, always (*The statistic*).
9. **Quoting stale project state.** A coached session edits the project under
   you — logs, tops, compute range. Re-read before asserting a number.
10. **Shipping the lateral through the connector.** Inline samples are for
   deriving through state that is not on the project. In the loop the
   inputs are always the project's own, and a long lateral is more pairs
   than one call carries: use `use_project_data` with the interpretation
   name. On a server that predates it, a browser page logged in to Drive
   can do the same job itself — fetch the project, its GR log, measured
   trajectory and the named interpretation from the API, and POST the
   derive body — so nothing but a summary crosses into the conversation. If
   you must inline, thin the lateral to about 1 ft where inclination exceeds
   85° (a horizontal sample spans a few hundredths of a foot of depth
   against the 0.25 ft grid) and keep full density through the curve; the
   result agrees with full density to well under 1 gAPI rms.
11. **Deriving to quiet a near-end warning.** Re-deriving through a run
   that kept the well inside the log adds no stratigraphic column (the well sampled
   nothing new) and, if the run has matched a look-alike, writes the new
   passes into the wrong beds. Derive on an excursion, not on the flag.
   The same holds for an entropy alarm with the bit clear of both ends
   (*Reading the state*): the alarm says look, and the margins say
   whether there is anything to derive.
12. **Saving an extension that adds no stratigraphic column, or too little
   for its GR.** A line parallel to the well grows nothing, which is
   right only where the well is riding a bed — the log at the job's own
   interval holding within the tolerance over forty feet or more — or the
   prior gives the footage less than nothing (step 4 of the extension);
   one that crosses near-zero
   stratigraphic column under a GR that keeps swinging
   beyond the tolerance files the smear (the loop, step 2). The alarm
   says the footage moves; the prior says how far when the GR is quiet,
   the beds when it is not. An alarm that returns on the delivery after
   an extension at the prior's dip is the well still drilling
   down-section, and the next extension is the same move, not a steeper
   one. Never spend a cycle on a line that leaves the piled-up footage
   inside the log, and never wait for the trajectory to take the well past
   the end on its own.
13. **Reset and Run on a delivery that left the type log alone.** An
   extended active log and survey with the type log untouched run as a
   plain Extend, which reaches the same result in a fraction of the time;
   a run that leaves the line over the last extension changes nothing
   until the next derivation (the loop, step 4). Reset only when the type
   log changed, a state-invalidating parameter changed, or a stalled job
   holds the pointer.
14. **Bending the line to explain GR the log already holds.** A pass within
   the log tolerance of what the log holds at the depth the prior's dip
   gives it is the rock varying along the lateral, and the tolerance
   carries it; a bend to chase it, however far inside the wiggle room, is
   structure made from that variation, and the black curve fits either way
   (the loop, step 2).
15. **A first line at a dip the prior does not hold.** A near-flat first
   line under a prior of 91.5° gives up 2.6 ft of stratigraphic column
   per hundred feet from the first delivery, and the estimate presses
   against the bottom before the alarm can say why. The apparent dip
   the tools report is the prior in degrees from horizontal (90°), not
   an angle to compare with the wellbore's inclination; convert it to
   rise per hundred feet of vertical section and draw the line on it
   (the first derivation, step 1). Nothing tests the first line on the
   day it is drawn but the first run; there is no log yet for a trial
   dip to be judged against. The test comes when the well rides a bed:
   the dip measured there, where the difference from the prior's is worth
   more than the alarm band over the first line's own vertical section,
   redraws the first line once (the first derivation, step 1). The gate is
   that column, not the dip sigma, which a first line's length makes far
   too coarse.
16. **Extending from the wrong MD.** The extension starts at the alarm's
   first MD — the 2 ft band's, not the 10 ft warning band's — and covers
   the footage since, a delivery or a few; on a look-alike, with no
   alarm, it starts at the reach-back MD, where the run left the line
   (*Between derivations*). A line that leaves the run's
   structure earlier files covered rock as new: on one lateral a line
   that left it where the margin entered the warning band, 1,300 ft
   before the alarm, rose a foot per hundred while the well dropped, and
   a delivery that should have added a few feet of stratigraphic column
   added thirty and reached the base (the loop, steps 1 and 2).
17. **Letting the trials choose the dip.** Read-only derives through the
   candidates measure them — room and landings — and that is all. A
   ladder that keeps the dip whose end moves most keeps the steepest;
   one that keeps the best match to the covered rock has done the
   computation's job by hand and then confirmed it against itself. The
   choice is the survivor nearest the prior (extend, step 6), and a line
   that has settled on some rate "ever since" has stopped starting each
   extension from the prior.
18. **A first line drawn at the wellbore.** The derivation does not care
   where the line sits — the same picks shifted 514.5 ft derived an
   identical log — but the marker the first save creates lands at the
   line's depth at the first computed MD, the run's structure arrives
   hung at the top of target (`copy_computed_interpretation`), and the
   cross section following a manual line hangs the target formation on
   it. A first line at the wellbore puts the marker at the log's top,
   and once the marker is moved to the target the line and everything
   hung on it sit hundreds of feet apart: on one lateral the marker was
   moved 514.5 ft and following the line displaced the target formation
   and the top of section by that amount. Draw it at the expected
   top-of-target depth (the first derivation, step 1).
