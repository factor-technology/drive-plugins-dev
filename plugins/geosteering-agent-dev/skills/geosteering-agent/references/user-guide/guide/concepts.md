<!-- published at /help/guide/concepts.html on the Drive host -->

# Key Concepts

This page defines the terms Drive uses. The definitions are precise; several differ from casual usage.

## Wells and logs

**Active well.** The well currently being geosteered. It carries a measured **trajectory** (surveys: MD, inclination, azimuth — growing as drilling progresses), a growing LWD **active log** (gamma ray, in the well's own measured-depth frame), an optional **well plan** (the intended path — a visual aid only, not used in the computation), and an optional set of **ignore ranges** (MD intervals excised from the computation, for example around a tool malfunction).

**Pilot well** (also called an offset well). A reference well whose vertical gamma-ray log serves as the **type log** the computation matches against. Each pilot carries a vertical gamma log, formation tops, and the MD range of the lateral over which it applies. Every project has at least one pilot; the first starts at MD 0 and defines the project's depth frame. Pilot logs are stored vertically, in the pilot's own TVD (positive, measured down) — if the source well was deviated, Drive straightens the log at upload time using a trajectory you supply, then discards that trajectory.

**Formation tops (markers).** Named depth picks on a pilot well, each with a color. You enter them in **type-log TVD** — positive feet (or meters), measured down.

**Top of target (ToT).** One formation top is designated the target. This is a project-level setting (the *Target* checkbox in the Formation Tops grid), and it names the single horizon the computation solves for. Drive models only the top of the target — there is no base-of-target concept in the computation.

## Depth frames

Drive works in two depth frames, and the cross section can display three depth axes (Profile → Scene → Depth axis):

- **Type log TVD** — the pilot well's own vertical depth, positive down. This is what you type when entering formation tops, and what you read off the pilot log. Pilot data is stored exactly this way: positive, increasing downward.
- **TVDSS** — true vertical depth subsea, the shared frame of the active well's trajectory and the computed structure. Negative values are below sea level.
- **TVD** — the active well's own vertical depth, positive down: the active well's reference elevation minus TVDSS.

Pilot wells carry no reference elevation. The **Depth Offset** fit parameter (see [Align Logs](./setup/align-logs.md)) is the single bridge between the frames: it places the pilot's depth zero in the shared frame, carrying the type log into the active well's depth frame.

::: tip Datum elevation
The active well's reference elevation is optional: set it if you want subsea depths, for example to compare against a regional structure map or to import a TVDSS-referenced interpretation.
:::

## Multiple pilot wells

Most projects use one pilot. When the geology changes character along a long lateral you can add pilots, each covering a contiguous MD range. Three rules apply, and Drive relies on you to observe them:

1. The first pilot (MD 0) defines the project depth frame.
2. The top-of-target marker must be entered at the same type-log TVD depth on every pilot (flattened on ToT).
3. Every formation named on the first pilot must appear, by the same name, on every other pilot.

Between adjacent pilots Drive builds an interpolated type log by warping each layer's thickness and blending gamma values.

## Dip and azimuth

**VS azimuth.** The direction of the vertical-section plane — the plan-view direction the lateral is drilled. Drive can compute it from the trajectory, or you can set it explicitly.

**Apparent dip.** The formation tilt as seen in the vertical-section view, expressed in *trajectory-inclination coordinates*: 90° is flat-lying, values below 90° dip one way, above 90° the other. This is the directional-drilling convention — not the textbook geology convention. Typical targets in horizontal plays present apparent dips between roughly 70° and 110°.

**Dip prior.** The computation needs an expectation of how the formation tilts before it sees any data. You provide it either as **apparent dip** values (piecewise constant, per MD range) or as a **prior structure** — a VS/TVDSS polyline you upload or snapshot from an existing interpretation. The two are peers; choose whichever expresses your geologic knowledge better. See [Dip and Azimuth](./setup/dip-azimuth.md).

## The job

A project is not one computation — it is a series of jobs over time, one per data extension. Drive keeps prior job state, so when new survey and log data arrive it **extends** the interpretation incrementally from where it left off. **Reset Job** discards the computed state (keeping your data and settings) so the next run recomputes the whole well; this is required after edits that invalidate prior state, notably changes to fit parameters. See [Run Job](./setup/run-job.md).

## Interpretations

**Computed interpretation.** The output of the job: the most probable explanation (MPE) of structure-top depth along the lateral, nine alternative explanations with their probabilities, and per-MD probability distributions (marginals). See [Understanding the Results](./results.md).

**Manual interpretation.** A horizon you draw by hand on the cross section — starting from scratch, from an import, or by copying the MPE. Manual interpretations can also feed back into the computation: as a prior structure, or as an interpretation the job extends. See [The Cross Section](./profile.md).
