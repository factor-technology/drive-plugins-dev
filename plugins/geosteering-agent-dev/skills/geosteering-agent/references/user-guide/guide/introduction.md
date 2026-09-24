<!-- published at /help/guide/introduction.html on the Drive host -->

# Introduction

Factor Drive is a geosteering interpretation system. Given a reference ("type") gamma-ray log from a nearby pilot well and the live gamma-ray log and survey trajectory from a well being drilled horizontally, Drive computes where the top of the target formation sits in depth all along the lateral — together with the uncertainty of that answer.

Rather than producing a single line on a cross section, the computation produces a full probability distribution of structure-top depth at every measured depth, a single most-probable structural interpretation, and a set of alternative interpretations each with its own probability. See [Understanding the Results](./results.md).

## How the application is organized

Factor Drive opens on the **Projects** page, a list of every project you can see. Each project corresponds to one active well. From a project's row you open the project itself, which has four tabs:

| Tab | Purpose |
|---|---|
| **Setup** | A six-step wizard that collects everything the computation needs: pilot wells and formation tops, active-well data, dip and azimuth, log alignment, job parameters, and run configuration. |
| **Profile** | The interactive cross section — the main working view for reading computed interpretations and drawing manual ones. |
| **Background** | Upload and register a background (for example seismic) image behind the cross section. |
| **Inventory** | Browse and download every data object in the project: logs, trajectories, interpretations. |

## The basic workflow

1. **Create a project** from the Projects page.
2. **Set up** the project in the wizard: upload a pilot log and enter formation tops, load the active well's trajectory and gamma log (by file upload, WITSML connection, or email), set the vertical-section azimuth and a dip prior, align the logs, and review job parameters.
3. **Run the job.** Drive computes on a GPU backend; progress appears in the project's tab bar.
4. **Read the results** on the Profile tab: the most probable explanation, the alternatives, and the per-depth marginals.
5. **Extend as drilling progresses.** New trajectory and log data — polled from WITSML, emailed in, or uploaded — extend the interpretation incrementally rather than recomputing from scratch.

## Where to go next

If you are new to Drive, read [Key Concepts](./concepts.md) first — a few terms (pilot well, top of target, TVDSS, apparent dip) carry precise meanings here. Then follow [Your First Project](./getting-started.md) end to end.
