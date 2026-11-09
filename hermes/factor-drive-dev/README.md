# Factor Drive for Hermes

Adds the Factor Drive geosteering copilot to Hermes Agent. Geologists use it to
read and judge geosteering job results, and to set up, tune and run Drive
projects, by asking in plain language instead of clicking through the Drive web
UI.

## What you get

- **The geosteering skill**: how to read job results (marginals, the MPE, the
  auto-picked horizons, whether the well is in the target), set a project up
  (pilot wells, type logs, markers, alignment, dip, faults), and tune a run.
- **The Factor Drive connector**: the actions you would otherwise take in the
  Drive web UI. List and read projects and results, upload logs and surveys,
  edit job parameters, and run, rerun, or reset jobs.

## Requirements

- Hermes Agent 0.21.6 or later
- A Factor Drive account (https://drive-app-dev.factor.technology)

## Install

    hermes plugins install factor-technology/drive-plugins-dev#hermes/factor-drive-dev
    hermes plugins enable factor-drive-dev

A Hermes plugin package can't ask for OAuth, so add the connector with its own
sign-in, then turn on approvals:

    hermes mcp add factor-drive-dev --url https://drive-app-dev.factor.technology/mcp --auth oauth
    hermes config set mcp_servers.factor-drive-dev.trust untrusted

The first command opens a browser window: sign in to Factor Drive and approve
the permissions it asks for, then answer **Y** to enable the tools. Start a new
chat and ask for something in Drive; *"list my Factor Drive projects"* will do.
There is no key or token to paste. If you ever need to sign in again, run
`hermes mcp login factor-drive-dev`.

### Signing in on a headless machine

Sign-in returns to a local callback on the machine running Hermes. If your
browser is on a different machine, run the sign-in from Hermes Desktop, which
relays the callback for you, or follow Hermes's
[OAuth over SSH guide](https://hermes-agent.nousresearch.com/docs/guides/oauth-over-ssh). Drive doesn't offer device-code
sign-in, so `--flow device` won't work.

## Approvals

With `trust: untrusted`, Hermes asks before running any tool Drive doesn't mark
read-only, such as uploads, edits, job resets and deletes. Where nobody can
answer, such as a cron job, it refuses those calls. Read-only tools run without
asking.

To turn the prompts off, run
`hermes config set mcp_servers.factor-drive-dev.trust full`. Your config is
what Hermes uses: the server `hermes mcp add` creates replaces the package's own,
including the `untrusted` setting the package declares. That is why the install
sets it again.

## What it sends, and where

- **Every tool call goes to `https://drive-app-dev.factor.technology/mcp`** and runs as **your**
  Drive user. It reaches only the projects you can already open.
- **The skill reads its guidance from that same connector** (its
  `read_skill_reference` tool), so the guidance always matches the tools Drive
  serves.
- **Large LAS files go from your machine to the Drive host.** The connector's
  `create_upload` tool returns an upload address, and Hermes sends the file
  with a terminal command.
- **WITSML tools contact the WITSML servers you configure** in Drive, from the
  Drive server, to pull live well data.
- The package runs no code on your computer and sends no telemetry: it has no
  background processes, hooks, or local servers. Hermes stores your sign-in
  token at `~/.hermes/mcp-tokens/factor-drive-dev.json`.

## What it can't do

The Drive tools run on the Drive server, so they can't read files on your
computer on their own; a file has to be uploaded first. Alignment and WITSML
refreshes run in the background and are polled rather than waited on. This
package leaves out Drive's cross-section scene-setup skill, which needs Claude's
browser tools. The steering decision stays with the geologist.

## Privacy, terms and support

- Documentation: https://drive-app-dev.factor.technology/help/guide/agent.html
- Privacy policy: https://www.factor.technology/privacy
- Terms: https://www.factor.technology/tos.html
- Support: support@factor.technology

## Dev variant

This artifact points at the dev backend and is named `factor-drive-dev`
so it can sit alongside the production plugin. Don't hand it to a customer.

---

`factor-drive-dev` 0.5.83 · connector `https://drive-app-dev.factor.technology/mcp` ·
built from the drive-app monorepo with `yarn workspace agent build:plugin`.
