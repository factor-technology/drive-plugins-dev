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
alarm, extend the interpretation over the new footage, derive again. Each
cycle grows the log by whatever new stratigraphy the footage just drilled
crossed, and successive versions agree where they overlap — on one real
lateral that deepened 250 ft of stratigraphy over 2,900 ft, ten cycles of
about 300 ft agreed to within a couple of gAPI rms in the overlap. The loop
converges on the log a geologist would draw at the end of the well, except
you have it while drilling.

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
   The project's prior structure (`read_structure`) is that opinion, already
   entered: draw the line at its dip; the block's dip tolerance is the
   wiggle room you have to adjust it (the loop, step 2). Only the dip is
   the signal — the
   computation reads the polyline as a sequence of dips and never its
   absolute depth, so where it sits relative to the wellbore or the markers
   means nothing, and the line's own depth is set by the basepoint at the
   wellbore. Through the curve that dip is also the stratigraphic column's *scale*: the
   stratigraphic column's thickness is the wellbore's TVD descent minus the line's descent
   over the same footage (a line rising relative to the wellbore's inclination adds
   stratigraphic column, one falling relative to it removes it), so a prior dip off by a factor of two mis-scales the
   stratigraphic column by that fraction of the line's rise or fall, and every later
   estimate inherits some feet of offset at the bottom. A horizontal line is
   not a safer default — it is a dip claim nobody made, and its error is the
   whole of the prior's dip. With no pilot to check against, take the prior's
   dip and expect the lateral's re-crossings to correct the scale.
2. **Derive bare, from the project's own data.** Leave it un-spliced: in
   the pane the **Splice into current type log** switch stays off, through
   the tool omit the initial type log. Through the tool, name the saved
   interpretation and set `use_project_data`: the server reads the active
   log, the survey and the project's frame itself, so nothing but the name
   crosses the connector. Read the result on the type log track — orange
   correlations are the backprojected passes, the light curve is the derived
   log.
3. **Choose the statistic by reasoning** (below).
4. **Save and replace, with approval.** Writing the curve onto the pilot well
   replaces its log: destructive, so advisory-plus-approval. It keeps the
   well's calibration and metadata, creates the top-of-target marker if
   missing, and reports any marker now outside the log — a warning, not a
   deletion, and usually a sign the interpretation should have gone further.
5. **Reset, rerun, restore ingestion.** A replaced type log invalidates all
   saved computation: `reset_job` then `trigger_job_rerun`, with approval,
   recomputing the well from scratch in typically minutes. On a WITSML
   project pause both pollers first and **re-enable them afterwards**
   (§1.9.6); an email-fed project has nothing to pause.
6. **On a self-steered well, make an inadequate log fail loudly.** Set the
   log tolerance tight on the lateral's parameter blocks — about 5 gAPI
   (`update_param_block`, a block of its own from the landing if the
   project has one block; §1.1 for the stored width). The log was recorded
   by this bit, so in rock it holds the measurement matches it closely, and
   a tolerance that tight leaves the computation no depth for footage it
   does not hold: instead of piling up quietly at an end the run ends in
   error and the job status reads **Impossible**, the alarm in its hard
   form (*Reading the state*). Leave the curve's block at the usual
   tolerance: through the build the computation smooths the GR over
   several feet of TVD while the derivation filed the raw samples, and that
   alone differs by more than a few gAPI. The number is a starting point —
   too tight and covered rock fails on drift alone, and every failure
   costs a full recompute.

The derived log's depth axis is anchored so the wellbore at the first computed
position sits at its own depth, and the interpretation's depth there is where
the top-of-target marker lands. That is why markers keep their meaning across
a replacement and no re-pick is needed.

### Choosing the statistic

Where the lateral crossed the same stratigraphic depth more than once the
passes can disagree — a stringer developed at one lateral position and not
another, or one pass placed slightly wrong. The method decides **which lateral
position's character represents the stratigraphic column**:

| Method | Meaning | Behavior on a disputed feature |
|---|---|---|
| Shallowest MD | earliest (heelward) pass wins | may miss character encountered later |
| Deepest MD | latest (toeward) pass wins | most current; carries recent features forward |
| Mean | average across passes | dilutes — a clean spike survives at half strength |
| Median | majority across passes | robust; keeps features most passes agree on |

There is no house default. Ask: **which choice best represents the
stratigraphy the rest of the lateral will see?** Compare at least two on the
features that motivated the derivation and watch whether they survive — a
read-only derive per method and an rms over the overlap is enough, and with
only two passes mean and median are the same curve, so the comparison starts
at the first re-derivation. Disagreement between passes is itself information
— say so when it is material rather than averaging it away silently (one
band on one lateral differed by 29 gAPI between its first and latest pass).

## The loop

Every later cycle is the same four moves:

1. **Read the alarm.** The coverage block on the latest job result says
   whether the well has drilled past an end of the log, which end, and the
   first MD where it showed. That MD is where the extension starts.
2. **Extend the ONE manual interpretation to the bit — or reach back.**
   The line over the footage the log was derived from usually stays as it
   is, but the cycle is not confined to the newest stretch: any stretch of
   the line, however far back along the lateral, may be revised when a
   better log comes of it — passes that stack where they overlap, the same
   bed at the same depth wherever the well re-crossed it, a prediction
   with the measurement's character (below) — and the test is the same, a
   read-only derive through the revised line before saving. The log
   inherits the line's scale over the footage it was built from, and no
   run rescales it: on one blind lateral a first line at twice the true
   dip squeezed the stratigraphic column 4%, and the estimate over the
   next 2,000 ft carried 4–6 ft of it that the runs only partly corrected.
   Reach back with your own picks, never the computed structure
   (pitfall 1). From the last derivation's MD to the bit the new picks
   come in three stretches, and one construction governs all of them: a sample lands in the stratigraphic column at
   its vertical distance from the line, so the line's depth at an MD is
   fixed by the stratigraphic column depth you believe that MD's GR must sit at.
   - *Footage the run kept inside the log* — tight, single-peaked marginals,
     the estimate clear of both ends. There the run is a correlation of new
     footage against the stratigraphic column, not an echo, and its MPE **is** the line:
     begin from those picks, appended to the manual line
     (`copy_computed_interpretation` hangs the MPE at the top of target the
     way a manual line is drawn; take the picks from that copy, not raw from
     the MPE slice, which sits on the top of section). Restarting from the
     last derivation's pick and carrying the prior across this footage
     throws the correlation away — on one lateral every cycle's extension
     began at that pick, and the tens of feet of correlated footage between
     it and the first position the run had piled up at the bottom went to the
     prior each time.
   - *The run's trailing footage* — the last few hundred feet of any run are
     provisional (a toe gets revised by several feet, sometimes more than
     ten, as the next delivery lands; trust it outright only where
     successive runs agree), so the MPE there is a starting point, not a
     boundary. Replace as much of it as the new GR justifies with trial
     picks of your own: a bed the stratigraphic column already holds goes to the depth
     the stratigraphic column holds it. Test a trial by deriving read-only through it (a
     trial interpretation document rides into `derive_type_log` alongside
     `use_project_data`) and keep the picks that make the new pass stack on
     the stratigraphic column where it re-crosses covered rock; where it crosses only new
     rock there is nothing to fit and the dip is chosen as below.
   - *Footage from the point where the estimate piled up at an end of the
     log* — the run's structure there is the **deepest** (for a bottom
     alarm) structure that still keeps the well inside the log, never the
     truth, and it bends toward the wellbore. Carry none of it, and do not
     draw the line so the well just reaches the end (on one lateral that
     left the well 5 ft below the log when it had drilled 24 ft of new
     rock). From the last believed pick the line's dip is chosen by the
     procedure below.

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
   MD across a thin slice of the stratigraphic column turns the statistic
   of hundreds of varying samples into one value: the log is smeared, the
   run reproduces the smear, and nothing alarms. So before saving, with
   the survey (`read_active_trajectory`), the GR (`read_active_log`), the
   prior (`read_structure`) and the block's dip tolerance
   (`read_job_params`):
   1. *Measure the well.* The wellbore's rise per hundred feet of MD over
      the new footage. It is not a candidate dip; it is what every
      candidate is measured against.
   2. *Count the beds.* Each clean or hot excursion in the new GR with
      different rock between counts as a bed.
   3. *Compute the room each candidate gives.* Stratigraphic column
      crossed is the line's rise minus the wellbore's, per hundred feet,
      times the footage (the scale rule of the first derivation). The
      candidates are the prior's dip and moves off it within the wiggle
      room — the prior polyline carries dip only — and horizontal, only
      where the well re-crossed the same rock at the same depth.
   4. *Reject candidates without room.* Beds counted times a bed's
      thickness — feet, not tenths — is the least stratigraphic column the
      footage needs. A candidate that gives less has filed unlike rock at
      one depth.
   5. *Place the features and check them against the log at that depth.*
      A clean spike must land on a clean bed the log holds, or past an
      end. Landing on shale in covered rock rejects the candidate. Past
      the end is allowed and expected: that is the alarm arriving, and the
      tight log tolerance (the first derivation, step 6) makes the
      computation run this check itself.
   6. *Take the survivor nearest the prior.* If nothing survives inside
      the wiggle room the prior is wrong here: go past it in the direction
      the beds demand, and say so. The alarm alone is reason to move and
      the beds say how far; with none to count — a featureless GR — make
      the smallest move within the wiggle room that lands the footage past
      the end.
   7. *Audit the dip change against the survey.* The line changes dip on
      the rock's evidence only. A change that coincides with a survey
      inclination change and nothing else is the line following the well.
   8. *Derive read-only through the choice, save, run, read the verdict.*
      An end that moved by less than the alarm band, or a pass scattering
      sideways at one depth on the type log track with the cross section
      following the trial line, sends you back to step 3. After the run,
      an alarm or an *Impossible* means extend further along the same dip;
      new footage inside the log with matching character means keep.
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
   **Never** copy the computed structure over the footage the log was derived
   from: that structure was solved against this very log, so deriving through
   it feeds the log its own output.
3. **Derive bare again and replace.** Same statistic unless there is a reason
   to change, same approval, same save. Compare the new log against the
   previous one where they overlap and say so if they disagree by more than
   the log's own noise.
4. **Reset, rerun, resume, confirm.** On the next run the alarm should clear
   and the margins reopen.

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
  until there is some, the block says so. Not an alarm.
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
  keep swinging) is the *excursion*, and the cue to extend.
- **Impossible.** The run ends in error and the job status reads
  *Impossible*: at some position no structure within the tolerances puts
  the measured GR anywhere the log holds (`read_job_status` carries the
  reason and the last MD each pass reached). With the tight tolerance of a
  self-steered well that is the alarm in its hard form — the log is
  inadequate past that MD, not deep enough or no longer representative of
  the rock the bit is in — and the answer is the loop, step 2, not a
  looser tolerance (pitfall 5). A failed run leaves no picks over the new
  footage, so the extension begins from the last completed run's; and a
  failure in rock the log already holds says its character has drifted —
  re-derive with the statistic that carries the latest passes forward.
- **After a good re-derivation.** Margins reopen and entropy drops back. Fit
  over the footage drilled *since* the previous derivation is a real test of
  that cycle's speculation; fit over the footage the log was derived from
  tests nothing, since the log was built to match it. The toe of a fresh log
  is its least trustworthy stretch: the log there came through the
  speculative part of the line, so the bit marginal often splits between
  your line and an alternative a few tens of feet away (P1 against P2).
  Leave it; the next footage decides.
- **Back in covered rock.** Footage drilled since the derivation that fits
  inside the log — margins in the tens of feet, tight single-peaked
  marginals, no alarm — is the loop's success case, even when the estimate
  sits tens of feet from your speculative line: the well came back into rock
  the log already holds, and the computation is correcting the guess. Accept
  the run — with the active log track in view: a black curve gone nearly
  constant under swinging passes over the new footage is an earlier cycle's
  smear, not success. Re-derive only on the alarm, or when you change your
  mind about the line over footage the log was derived from, which that
  smear is.
- **Stuck at an end.** The same alarm on consecutive runs, a re-derivation
  between each, and the log's end moving by less than the alarm band per
  cycle: that is the loop spinning, not the well plateauing. The bit
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

Any type log can run out this way, derived or not — an original pilot ending a
couple of feet below the deepest depth the lateral reaches raises the same
alarm. The alarm also has a blind spot: new rock that mimics a feature already
in the log, within reach of the dip prior, stays confidently wrong. Treat a
clear alarm as reliable and a quiet one as "no evidence of trouble".

The look-alike has a signature of its own. A toe piled up at the bottom that the next
delivery "resolves" — confidence back to a single peak, entropy down — by
revising already-confident footage by ten or more feet, or by drawing a fold
the prior structure does not have, has most likely matched new rock to a bed
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
there for exactly this) — and a line lifted from a run against a
*differently derived* log does not transfer either, since that log had its
own scale. Where the well does come back into covered rock the computation
corrects the guess, and the loop converges. The well *can* scale a stretch
where it crosses the same rock twice at the same depth — the same hot bed
at the same TVDSS 500 ft apart with different rock between says the
structure is horizontal there — and that is direct evidence, worth more than the
prior over that footage.

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

Default to volunteering. Only the manual interpretation, the statistic, and
the approvals are judgment. The save goes straight through the connector, so
a whole cycle — derive, save, reset, rerun, resume polling — is yours to offer
end to end, with no file round-trip and no browser needed.

With browser tools you can also work the Derived pane itself and click
through the methods to compare; without them, coach the same sequence by pane
and control name. Either way user-facing language stays plain: "replace the
type log with one derived from the well itself and re-run", never tool or
field names.

## Pitfalls

1. **Deriving through the computed structure over the footage the log was
   derived from.** The manual interpretation defines the backprojection,
   every cycle. From the second cycle on this is a correctness rule, not a
   preference: over that footage the computed structure was solved against
   the derived log. Check what the cross section is following first. It
   forbids the computed structure there, not your own picks: revising the
   line over that footage by judgment is step 2's reach-back. The
   converse is as wrong: over the footage drilled since, the run *is* the
   correlation, and restarting from the last derivation's pick with the
   prior discards it.
2. **Splicing.** Leave the splice switch off. Spliced values are unconfirmed
   values at depths this well never visited, and the computation reads them as
   ground truth. Splicing also silences the alarm, which is the loop's clock.
3. **Extending past the bit, or extending to quiet the alarm.** The extension
   covers footage actually drilled and stops at the bit. Speculate about
   structure, never about stratigraphy the well has not sampled.
4. **Late derivation.** A log derived at the toe explains footage already
   drilled. Raise the option as soon as the pattern appears.
5. **Treating the alarm as an error.** It is the design working: the well
   found rock the log doesn't cover yet (an *Impossible* on a self-steered
   well says the same). Report it that way and re-derive;
   don't tune dip or log sigma to paper over it.
6. **Resetting the calibration on replace.** The save writes in the pilot's
   own frame, and preserving it is what keeps the markers where they are.
   Re-run alignment only if a warp was actually in use (replacing drops it).
7. **Forgetting to restore ingestion.** A project left with its pollers
   paused after a manual rerun quietly stops being steered.
8. **Treating a statistic as a default.** It is a judgment about the rest of
   the lateral. Compare at least two.
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
12. **Saving an extension that adds no stratigraphic column, or too little
   for its GR.** A read-only derive through the extension whose end moved
   by less than the alarm band will reproduce the alarm when saved; one
   that clears the band but crosses near-zero stratigraphic column under a
   GR that keeps varying will reproduce the smear (the loop, step 2). The
   prior is a starting point; the alarm says the footage moves, the GR
   how far. Never spend a cycle on a line that leaves the piled-up footage
   inside the log, and never wait for the trajectory to take the well past
   the end on its own.
