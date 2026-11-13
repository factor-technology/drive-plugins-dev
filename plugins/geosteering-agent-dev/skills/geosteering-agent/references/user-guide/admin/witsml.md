<!-- published at /help/admin/witsml.html on the Drive host -->

# WITSML Servers and Polling

Drive can act as a WITSML *client*, polling an external WITSML server for new trajectory and log data and storing it on the well as the well drills ahead.

## Registering servers

WITSML servers are registered **per user**, not per project: once added, a server can serve the pollers of any well you have Write access to. Manage them from **Account → Manage WITSML Servers**; a project's WITSML tab links there.

Registration takes the server URL, your credentials, and the WITSML version (**1.3.x**, or **1.4.x** in beta). Drive reads the new server's catalog in the background, so its wells may take a minute or two to appear. The servers page lists each registered server with per-row **Refresh** and **Delete** actions. When pollers use a server, **Delete** first says how many and asks you to confirm. After the delete those pollers will have no server; their settings will stay. Until someone with Write access to the well chooses another server, that well will get no WITSML data and its status will show the problem. A poller whose server was deleted shows as "No server (the server was deleted)" on its WITSML tab.

## The well list and refreshing

When a server is added, Drive reads its catalog of wells, wellbores, trajectories, logs, and curves; the WITSML tab's pulldowns are populated from that snapshot. If a newly available well or curve is missing from the pulldowns, use **Refresh** to re-read the server. Refresh waits up to 90 seconds; on a large server the crawl may still be running after that, so try the pulldowns again a little later.

## Pollers

Each well with a WITSML source has two pollers, shared by every project on the well and configured in [Setup step 2](../guide/setup/active-well.md#witsml-connection):

- a **trajectory poller** — *Standard* (fetches a WITSML Trajectory object) or *Continuous Inc/Azi* (synthesizes surveys from inclination/azimuth log curves, for MWD services that publish no Trajectory object), and
- a **log poller** — fetches the designated gamma-ray curve.

Saving a poller needs Write access to the well and a server of your own, and starts no polling. The well is polled only while WITSML is its data source and its **Receiving data** switch is on; that switch also needs Write access to the well, and turning it off will stop new WITSML data for every project on the well. Whether a project runs on the new data is that project's own [Automatic runs](../guide/setup/run-job.md#automatic-runs) switch: a project with them on will extend after each poll that brings new data, and its Run button is disabled meanwhile.

Points to know:

- A poll failure (unreachable server, changed identifiers) surfaces as an error rather than a silent retry: check the well's status in the well panel and the server registration.
- New data arriving mid-run restarts the job of a project with Automatic runs on, as a run capped at nine minutes of reverse phase that recomputes from the last saved state. A poll that brings nothing new leaves a running job alone.
- **Refresh Log and Trajectory** on the WITSML tab fetches the latest data once, with the well's saved pollers, without running any job.
