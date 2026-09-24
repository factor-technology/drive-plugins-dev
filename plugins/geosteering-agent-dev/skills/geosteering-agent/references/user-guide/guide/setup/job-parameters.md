<!-- published at /help/guide/setup/job-parameters.html on the Drive host -->

# Step 5 · Job Parameters

Job parameters are **tolerances** — bounds on how far the computation may deviate from your priors to explain the gamma data. They are not direct settings of the answer. They are also coupled: tightening one pushes unexplained signal onto another mechanism (tighten dip with faults enabled, and a fault may appear to explain what dip no longer can). Adjust one thing at a time, and see [Troubleshooting and Tuning](../troubleshooting.md) for symptom-driven guidance.

## Per-range parameters

The grid has one column per MD range (add or remove ranges with the buttons above it; the range starting at MD 0 always exists and applies wherever no later range overrides it).

| Row | What it bounds |
|---|---|
| **Log Tolerance** | How far the active gamma reading may sit from the (scaled) type-log value — the 99.7% bound, in gamma units. Much of what looks like noise is real geological divergence between pilot and active well; prefer widening this tolerance over dismissing data as noisy. |
| **Outlier Handling** | **Strict** expects well-behaved readings; **Forgiving** tolerates occasional wild spikes (heavier-tailed model) — useful for noisy LWD. |
| **Dip Tolerance (°)** | How far the local dip may wander from the dip prior. Raise it to let the data override the prior; lower it to hew to the prior. Higher values cost memory and compute — raise one step at a time. |
| **Expected faults** | Expected number of faults per 10,000 ft of lateral. **0 disables fault modeling.** |
| **Min / Max fault throw** | Bounds on modeled fault throw. Throws follow a power law: small throws are common, max-throw faults rare. |
| **Upward fault bias** | Prior on throw direction: 1 = all up toward the bit, 0 = all down, 0.5 = even. |
| **Prune** | How aggressively low-probability outcomes are discarded, in dB. Less pruning (or *No pruning*) preserves fidelity at the cost of memory and time. |

*Curvature* — how fast dip may change with lateral distance — lives on the [Dip and Azimuth](./dip-azimuth.md) grid but belongs to the same family: 0 means unlimited bending; smaller values force smoother structure.

## Global parameters

| Parameter | Meaning |
|---|---|
| **Depth Resolution** | Vertical grid spacing of the computed structure. Finer resolution costs memory and time. |
| **MD Interval** | Lateral sampling interval of the computation. Coarser intervals smooth over brief gamma excursions; finer intervals react to them. |

**Reset to Defaults** restores the shipped parameter set.

::: tip Defaults first
The defaults are well-tested starting values. Run with them, look at the result, and only then tune — one parameter at a time, ideally on a [cloned project](../projects-page.md#per-row-actions) or a bounded compute range so experiments stay cheap.
:::
