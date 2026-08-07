<!-- published at /help/guide/results.html on the Drive host -->

# Understanding the Results

A job run produces three related outputs. Reading them together — not just the single best line — is what distinguishes Drive from deterministic geosteering tools.

## The MPE

The **Most Probable Explanation** is the single most probable trace of the target-formation top along the whole lateral. It is what the Profile tab shows when you select **MPE** under Interpretations → Computed.

The MPE depth at each MD is the top of the **target geologic structure** — not the bit, and not the wellbore.

## The alternatives

Alongside the MPE, the computation reports **nine alternative explanations**, each a complete, coherent structure trace with a global probability (the percentages shown as **P1 %**, **P2 %**, … in the Interpretations pane; they sum to roughly 100 together with the MPE's cluster).

How to read them:

- If the top alternatives cluster tightly around the MPE, the interpretation is robust — the data admits essentially one story.
- If they form **two clusters**, the data genuinely supports two competing interpretations; look at where they diverge, and what (a fault? a dip change?) distinguishes them.
- Three or more scattered clusters mean high ambiguity: treat any single line, including the MPE, with caution.

The alternatives are anchored at the current end of the wellbore — they are constructed to span the plausible depths of the structure at the bit, which makes them directly useful for the "where is the bit relative to the target *right now*" question.

## Marginals

The **marginals** (Profile → Marginals) show, at each MD, the full probability distribution of the structure-top depth — displayed as a color field or as wavelets. Sharp, narrow marginals mean the data pins the structure down; broad or multi-lobed marginals mean uncertainty, and their shape tells you what kind.

One caution: marginals are per-depth-station distributions. They tell you where the top may be *at each MD*, but not how those depths connect laterally — connectivity is what the MPE and the alternatives express.

## Steering versus diagnosis

Two different questions call for two different readings of the output:

- **Steering** ("where should the bit go next?") — look at the marginal at the bit, the alternatives' spread there, and your forward dip prior. Ahead of the bit there is no data: the projection follows *your prior*, not a trend extrapolated from the recent MPE.
- **Diagnosis** ("is something wrong with our model?") — look upstream: where did the marginals broaden, where do alternatives diverge, does the misfit localize at one MD (a candidate fault) or grow steadily (a dip prior or alignment issue)? See [Troubleshooting and Tuning](./troubleshooting.md).

## Trusting the result

A useful discipline when the computed interpretation says the well is persistently out of zone while the marginals stay sharp and the fit stays good: before concluding the well really is out of zone, re-examine the inputs — the marker pick, the alignment, the dip prior, a possible missed fault. The drilling team has independent evidence (ROP, gas shows, cuttings), and a sustained disagreement with them usually means one of the priors is wrong, not that everyone else is blind.
