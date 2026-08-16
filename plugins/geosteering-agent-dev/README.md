# Factor Drive — Geosteering

Adds the Factor Drive geosteering copilot to Claude: the interpretation skill
plus a live connection to your Drive projects.

## Install

1. Drag this file into a new Claude chat.
2. Click **Save plugin** on the card that appears.
3. Ask for something in Drive — *"list my Factor Drive projects"* will do. A
   browser window opens the first time: sign in to Factor Drive and approve
   the permissions it asks for.

Nothing is installed on your computer and there is no key or token to paste.
Every call runs as **your** Drive user and reaches only the projects you can
already open; revoke the access in Drive whenever you like.

If something later comes back saying it lacks permission, reconnect Factor
Drive and approve the fuller set — that is the whole fix, not a support case.

## What you get

- **The geosteering skill** — how to read job results, judge an
  interpretation, set a project up (pilot wells, markers, alignment, dip,
  faults), and tune a run.
- **The cross-section skill** — drives your browser to frame and tidy a
  project's Profile scene ("make U-178 look good for the morning meeting")
  and capture a presentable image. Needs Claude's browser access; sits idle
  without it.
- **The Drive tools** — the actions you would otherwise take in the Drive web
  UI at `https://drive-app-dev.factor.technology`.

## What it can't do

It works through the Drive server, so it cannot read files on your own
computer: point it at a LAS or survey file on disk and it will ask you to
upload it instead. Alignment and WITSML refreshes run in the background and
are polled rather than waited on.

## Dev variant

This artifact points at the dev backend and is named `geosteering-agent-dev`
so it can sit alongside the production plugin. Don't hand it to a customer.

---

`geosteering-agent-dev` 0.4.50 · connector `https://drive-app-dev.factor.technology/mcp` ·
built from the drive-app monorepo with `yarn workspace agent build:plugin`.
