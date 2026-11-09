<!-- published at /help/admin/witsml.html on the Drive host -->

# WITSML Servers and Polling

Drive can act as a WITSML *client*, polling an external WITSML server for new trajectory and log data and extending the interpretation automatically as the well drills ahead.

## Registering servers

WITSML servers are registered **per user**, not per project: once added, a server is available to all of your projects. Manage them from **Account → Manage WITSML Servers**; a project's WITSML tab links there.

Registration takes the server URL, your credentials, and the WITSML version (**1.3.x**, or **1.4.x** in beta). Drive reads the new server's catalog in the background, so its wells may take a minute or two to appear. The servers page lists each registered server with per-row **Refresh** and **Delete** actions. When pollers use a server, **Delete** first says how many and asks you to confirm. Deleting pauses those pollers and keeps their settings; someone with Write access to each well must choose another server to resume polling. If one of them is the last source of live data for its well, that well stops receiving new data. A poller whose server was deleted shows as "No server (the server was deleted)" on its WITSML tab.

## The well list and refreshing

When a server is added, Drive reads its catalog of wells, wellbores, trajectories, logs, and curves; the WITSML tab's pulldowns are populated from that snapshot. If a newly available well or curve is missing from the pulldowns, use **Refresh** to re-read the server. Refresh waits up to 90 seconds; on a large server the crawl may still be running after that, so try the pulldowns again a little later.

## Pollers

Each WITSML-connected project has two pollers, configured in [Setup step 2](../guide/setup/active-well.md#witsml-connection):

- a **trajectory poller** — *Standard* (fetches a WITSML Trajectory object) or *Continuous Inc/Azi* (synthesizes surveys from inclination/azimuth log curves, for MWD services that publish no Trajectory object), and
- a **log poller** — fetches the designated gamma-ray curve.

Pollers start **paused**, and can run only while WITSML is the project's data source: choose it with **Use WITSML for this project** on the WITSML tab. Saving a poller's settings needs a server of your own; anyone with Write access to the project can pause polling. The live-updates control in [Setup step 6](../guide/setup/run-job.md#live-updates) turns both on or off together; on a project that has not run yet, the first Run turns them on once the run has started. While on, Drive fetches every few minutes and, on new data, re-runs the job automatically to extend the interpretation; the Run button is disabled meanwhile.

Points to know:

- A poll failure (unreachable server, changed identifiers) surfaces as an error rather than a silent retry — check the project's status and the server registration.
- New data arriving mid-run restarts the job, as a run capped at nine minutes of reverse phase that recomputes from the last saved state. Before a run's first result, every poll restarts it, new data or not.
- **Refresh Log and Trajectory** on the project's WITSML tab fetches the latest data once, using the saved poller configuration, without running the job.
