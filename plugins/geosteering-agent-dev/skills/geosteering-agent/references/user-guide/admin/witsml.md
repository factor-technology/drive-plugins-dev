<!-- published at /help/admin/witsml.html on the Drive host -->

# WITSML Servers and Polling

Drive can act as a WITSML *client*, polling an external WITSML server for new trajectory and log data and extending the interpretation automatically as the well drills ahead.

## Registering servers

WITSML servers are registered **per user**, not per project: once added, a server is available to all of your projects. Manage them from **Account → Manage WITSML Servers**, or add one inline from a project's WITSML Connection dialog.

Registration takes the server URL, your credentials, and the WITSML version (1.3.1.1 or 1.4.1.1). The servers page lists each registered server with per-row **Refresh** and **Delete** actions.

## The well list and refreshing

When a server is added, Drive reads its catalog of wells, wellbores, trajectories, logs, and curves; the WITSML Connection dialog's pulldowns are populated from that snapshot. If a newly available well or curve is missing from the pulldowns, use **Refresh** to re-read the server — the refresh is synchronous and can take up to a minute or two on large servers.

## Pollers

Each WITSML-connected project has two pollers, configured in [Setup step 2](../guide/setup/active-well.md#witsml-connection):

- a **trajectory poller** — *Standard* (fetches a WITSML Trajectory object) or *Continuous Inc-Azi* (synthesizes surveys from inclination/azimuth log curves, for MWD services that publish no Trajectory object), and
- a **log poller** — fetches the designated gamma-ray curve.

Pollers start **disabled**. The polling control in [Setup step 6](../guide/setup/run-job.md#witsml-polling) turns both on or off together. While enabled, Drive fetches on a schedule and, on new data, re-runs the job automatically to extend the interpretation.

Points to know:

- A poll failure (unreachable server, changed identifiers) surfaces as an error rather than a silent retry — check the project's status and the server registration.
- New data arriving mid-run restarts the job. For a deliberate one-shot run, pause polling first.
- A manual "fetch now" is available via the refresh action; it uses the saved poller configuration.
