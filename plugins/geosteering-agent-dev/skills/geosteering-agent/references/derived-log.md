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
2. **Derive bare.** Leave it un-spliced: in the pane the **Splice into
   current type log** switch stays off, through the tool omit the initial type
   log. Read the result on the type log track — orange correlations are the
   backprojected passes, the light curve is the derived log.
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
features that motivated the derivation and watch whether they survive.
Disagreement between passes is itself information — say so when it is
material rather than averaging it away silently.

## The loop

Every later cycle is the same four moves:

1. **Read the alarm.** The coverage block on the latest job result says
   whether the well has drilled past an end of the log, which end, and the
   first MD where it showed. That MD is where the extension starts.
2. **Extend the ONE manual interpretation to the bit, speculatively.** Carry
   the previous cycle's interpretation over the new footage with a structure
   that could plausibly have put the well where the alarm says it is — deeper
   for a bottom alarm, shallower for a top alarm — using the prior or
   regional dip, anchored on the picks you still believe. It is a guess and
   is meant to be; the next run tests it. Stop at the bit. **Never** extend
   it by copying the computed structure the previous derived log produced:
   that structure was solved against this very log, so deriving through it
   feeds the log its own output.
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
  entropy, and the MD it started.
- **After a good re-derivation.** Margins reopen and entropy drops back. Fit
  over the footage drilled *since* the previous derivation is a real test of
  that cycle's speculation; fit over the footage the log was derived from
  tests nothing, since the log was built to match it.

Any type log can run out this way, derived or not — an original pilot ending a
couple of feet below the deepest depth the lateral reaches raises the same
alarm. The alarm also has a blind spot: new rock that mimics a feature already
in the log, within reach of the dip prior, stays confidently wrong. Treat a
clear alarm as reliable and a quiet one as "no evidence of trouble".

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
