<!-- published at /help/admin/witsml.html on the Drive host -->

# WITSML Servers and Polling

Drive can act as a WITSML *client*, polling an external WITSML server for new trajectory and log data and extending the interpretation automatically as the well drills ahead.

## Registering servers

WITSML servers are registered **per user**, not per project: once added, a server is available to all of your projects. Manage them from **Account → Manage WITSML Servers**; a project's WITSML tab links there.

Registration takes the server URL, your credentials, and the WITSML version (**1.3.x**, or **1.4.x** in beta). Drive reads the new server's catalog in the background, so its wells may take a minute or two to appear. The servers page lists each registered server with per-row **Refresh** and **Delete** actions. Delete asks no confirmation and removes the pollers of every project that uses the server.

## The well list and refreshing

When a server is added, Drive reads its catalog of wells, wellbores, trajectories, logs, and curves; the WITSML tab's pulldowns are populated from that snapshot. If a newly available well or curve is missing from the pulldowns, use **Refresh** to re-read the server. Refresh waits up to 90 seconds; on a large server the crawl may still be running after that, so try the pulldowns again a little later.

## Pollers

Each WITSML-connected project has two pollers, configured in [Setup step 2](../guide/setup/active-well.md#witsml-connection):

- a **trajectory poller** — *Standard* (fetches a WITSML Trajectory object) or *Continuous Inc/Azi* (synthesizes surveys from inclination/azimuth log curves, for MWD services that publish no Trajectory object), and
- a **log poller** — fetches the designated gamma-ray curve.

Pollers start **paused**. The live-updates control in [Setup step 6](../guide/setup/run-job.md#live-updates) turns both on or off together. While on, Drive fetches every ten minutes and, on new data, re-runs the job automatically to extend the interpretation; the Run button is disabled meanwhile.

Points to know:

- A poll failure (unreachable server, changed identifiers) surfaces as an error rather than a silent retry — check the project's status and the server registration.
- New data arriving mid-run restarts the job.
- **Refresh Log and Trajectory** on the project's WITSML tab fetches the latest data once, using the saved poller configuration, without running the job.
