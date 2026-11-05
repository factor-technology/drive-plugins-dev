# Factor Drive Geosteering (dev)

Adds the Factor Drive geosteering copilot to Claude. Geologists use it to
read and judge geosteering job results, set up and tune Drive projects, and
frame the cross section for review, by asking in plain language instead of
clicking through the Drive web UI. It needs a Factor Drive account
(`https://drive-app-dev.factor.technology`).

## What you get

- **The geosteering skill** — how to read job results (marginals, the MPE,
  the auto-picked horizons, whether the well is in the target), set a
  project up (pilot wells, type logs, markers, alignment, dip, faults), and
  tune a run.
- **The cross-section skill** — drives your browser to frame and tidy a
  project's Profile scene ("make U-178 look good for the morning meeting")
  and capture a presentable image. Needs Claude's browser access; sits idle
  without it.
- **The Factor Drive connector** — the actions you would otherwise take in
  the Drive web UI: list and read projects and results, upload logs and
  surveys, edit job parameters, and run, rerun, or reset jobs.

## Install

- **From Claude's plugin directory:** find **Factor Drive Geosteering (dev)** and add it.
- **From a plugin file:** drag the `.plugin` file into a new Claude chat and
  click **Save plugin** on the card that appears.

Then ask for something in Drive — *"list my Factor Drive projects"* will do.
A browser window opens the first time: sign in to Factor Drive and approve
the permissions it asks for. Nothing is installed on your computer and there
is no key or token to paste.

If something later comes back saying it lacks permission, reconnect Factor
Drive and approve the fuller set — that is the whole fix, not a support case.

## What it sends, and where

- **Every tool call goes to `https://drive-app-dev.factor.technology/mcp`** and runs as **your**
  Drive user, through the OAuth sign-in above. It reaches only the projects
  you can already open; disconnect it in Claude's connector settings at any
  time.
- **The geosteering skill reads its guidance from that same connector**
  (its `read_skill_reference` tool), so the guidance always matches the
  tools Drive serves. It fetches nothing from anywhere else.
- **The cross-section skill uses Claude's browser tools** on your Drive
  project's Profile page, and takes screenshots of it when you ask for an
  image.
- **WITSML tools contact the WITSML servers you configure** in Drive, from
  the Drive server, to pull live well data.
- The plugin runs no code on your computer: it has no hooks, scripts, or
  local servers, and it stores nothing itself.

Privacy policy: https://www.factor.technology/privacy · Terms: https://www.factor.technology/tos.html ·
Support: support@factor.technology

## What it can't do

It works through the Drive server, so it cannot read files on your own
computer: point it at a LAS or survey file on disk and it will ask you to
upload it instead. Alignment and WITSML refreshes run in the background and
are polled rather than waited on. The steering decision stays with the
geologist.

## Dev variant

This artifact points at the dev backend and is named `geosteering-agent-dev`
so it can sit alongside the production plugin. Don't hand it to a customer.

---

`geosteering-agent-dev` 0.5.81 · connector `https://drive-app-dev.factor.technology/mcp` ·
built from the drive-app monorepo with `yarn workspace agent build:plugin`.
