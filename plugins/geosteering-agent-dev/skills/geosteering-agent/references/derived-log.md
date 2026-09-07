<!-- Hand-authored static reference for the geosteering-agent skill.
     Ships verbatim via build-skill.mjs copyRefs. Source of truth: this file
     (agent/skill/references/derived-log.md). -->

# Steering the well against itself (the derived type log)

When the type log can't explain what the well is drilling, stop correlating
against it and correlate the well **against itself**. Back-project the active
GR through a short interpretation of the structure and make that curve the
project's type log; from then on every new pass is lined up against the column
the well itself established. Drive needs an actual type log object to steer
against, and the Profile tab's **Derived** pane (or `derive_type_log`) mints
one.

The derived curve **terminates at the stratigraphy the wellbore has explored**:
it starts at the wellbore's own depth at the first computed position and ends
at the deepest depth the interpretation reached. Nothing is grafted on above
or below. That is the mechanism, not a limitation. The computation reads a
type log as ground truth over its whole length and treats a wellbore past
either end of it as **impossible**, so the moment the well drills
stratigraphically past an end the posterior is pressed against it, the
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

- On the type log track, the backprojected MWD segments share no character
  with the type log: peaks with no counterparts, different amplitudes,
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
   backproject through that structure is where they belong in the column. A
   straight dipping segment is fine when seismic says the column is straight.
   Through the curve, the line's dip is also the column's *scale*: the
   column's thickness is the wellbore's TVD descent minus the line's fall
   over the same footage, so a prior dip twice too steep squeezes the column
   by a few percent and every later estimate inherits a few feet of offset
   at the bottom. With no pilot to check it against, prefer the gentler of
   the prior and flat over the curve, and expect the lateral's re-crossings
   to correct the rest.
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

The derived log's depth axis is anchored so the wellbore at the first computed
position sits at its own depth, and the interpretation's depth there is where
the top-of-target marker lands. That is why markers keep their meaning across
a replacement and no re-pick is needed.

### Choosing the statistic

Where the lateral crossed the same stratigraphic depth more than once the
passes can disagree — a stringer developed at one lateral position and not
another, or one pass placed slightly wrong. The method decides **which lateral
position's character represents the column**:

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
2. **Extend the ONE manual interpretation to the bit, speculatively.** Two
   kinds of footage lie between the last derivation and the bit, and the
   line treats them differently:
   - *Footage the run kept inside the log* — tight, single-peaked marginals,
     the estimate clear of both ends. There the run is a correlation of new
     footage against the log, not an echo, and its structure is the picks to
     carry. Trust it only where successive runs agree: a toe gets revised by
     several feet, sometimes more than ten, as the next delivery lands, so the
     last few hundred feet of any run are provisional.
   - *Footage past the first pressed position* — mass piled against an end,
     the estimate at the wall. The run's structure there is the **deepest**
     (for a bottom alarm) structure that still keeps the well inside the log,
     never the truth, and it bends toward the wellbore. Do not carry any of
     it, and do not draw the line so the well just reaches the end: start at
     the last believed pick and carry the project's prior structure
     (`read_structure`) or, where the well re-crossed the same rock at the
     same depth, a flat line, and let the well go as far past the end as that
     puts it. On one lateral a line drawn to meet the bottom at the last
     pressed position left the well 5 ft below the log when it had drilled
     24 ft of new rock under a flat structure; 2,000 ft of estimate were
     13–20 ft off for it.
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
  uncertainty corridor on the cross section.
- **Running out.** The estimate walks within a few feet of an end, or half
  the posterior mass piles into that band, or recent entropy runs about twice
  its baseline; the uncertainty band blooms toward the toe. The alarm names
  the end (bottom = drilled stratigraphically deeper, top = shallower) or
  entropy, and the MD it started. The alarm's band is 10 ft, and a log
  derived right after the curve has only a few feet of lateral-derived
  column at its bottom, so the flag stays up for as long as the well rides
  that zone. Read the marginals before acting: tight, single-peaked, the
  estimate a few feet clear of the end, entropy near its baseline is a
  *near-end warning* — the well is in covered rock and there is nothing to
  derive, since the log only grows when the well samples new rock. Mass at
  the wall, the estimate at the end, modes splitting, GR the log has no
  match for (hotter or cleaner than anything in the band) is the
  *excursion*, and the cue to extend.
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
  the run. Re-derive only on the alarm, or when you change your mind about
  the structure over footage the log was derived from.

Any type log can run out this way, derived or not — an original pilot ending a
couple of feet below the deepest depth the lateral reaches raises the same
alarm. The alarm also has a blind spot: new rock that mimics a feature already
in the log, within reach of the dip prior, stays confidently wrong. Treat a
clear alarm as reliable and a quiet one as "no evidence of trouble".

The look-alike has a signature of its own. A pressed toe that the next
delivery "resolves" — confidence back to a single peak, entropy down — by
revising already-confident footage by ten or more feet, or by drawing a fold
the prior structure does not have, has most likely matched new rock to a bed
higher in the log; and when the well then climbs and the structure rises in
lock-step with it, so that the well never leaves that bed, the structure is
following the wellbore. On one lateral the computation drew a 20-ft syncline
in 1,500 ft that way. The test is cheap: extend the line only through the
pressed footage, derive, and let the footage after it judge — the true
column correlates the climb with a smooth structure and the well moving
through the beds; the look-alike needs the structure to chase the well.

There is a deeper limit to know about. The well can only *scale* its own
column where it crosses the same stratigraphy more than once: a wrong dip
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
structure is flat there — and that is direct evidence, worth more than the
prior over that footage.

Run blind on a 10,000-ft lateral with no pilot at all (prior polyline plus
the well's own GR, three derivations), the loop finished within 5 ft of the
geologist's answer over half the well and within 10 ft over five sixths of
it, 3.5 ft at TD; the rest was one 2,000-ft stretch 13–20 ft off, the
footage where the well drilled 24 ft of rock the column never held and the
extension put it 5 ft below the bottom instead. That is the shape of the
limit: right wherever the well re-crossed its own column, and off by
whatever the speculation missed where it did not.

## The advisory reference log

Replacing the type log does not discard the original: the first save shelves
it as a **reference log**, an advisory copy nothing that computes ever reads —
not the job builder, not the type-log interpolation, not the cross section. It
is not a pilot well and it invalidates nothing. Later cycles shelve nothing,
since the log they replace is itself derived.

Use it for general-shape reasoning: roughly where in the column the wellbore
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

1. **Deriving through the computed structure.** The manual interpretation
   defines the backprojection, every cycle. From the second cycle on this is
   a correctness rule, not a preference: the computed structure was solved
   against the derived log. Check what the cross section is following first.
2. **Splicing.** Leave the splice switch off. Spliced values are unconfirmed
   values at depths this well never visited, and the computation reads them as
   ground truth. Splicing also silences the alarm, which is the loop's clock.
3. **Extending past the bit, or extending to quiet the alarm.** The extension
   covers footage actually drilled and stops at the bit. Speculate about
   structure, never about stratigraphy the well has not sampled.
4. **Late derivation.** A log derived at the toe explains footage already
   drilled. Raise the option as soon as the pattern appears.
5. **Treating the alarm as an error.** It is the design working: the well
   found rock the log doesn't cover yet. Report it that way and re-derive;
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
   that kept the well inside the log adds no column (the well sampled
   nothing new) and, if the run has matched a look-alike, writes the new
   passes into the wrong beds. Derive on an excursion, not on the flag.
