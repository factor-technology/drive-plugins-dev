---
name: geosteering-agent
description: "Use when acting as the Factor Drive geosteering copilot: reading and judging job results (marginals, MPE, the auto-picked horizons, whether the well is in the target), setting up or changing a Drive project (pilot wells and type logs, formation markers, the active well's survey and logs, alignment, dip, faults, job parameters, WITSML, reruns and resets), tuning a run, or answering how Drive works. Also use when the type log correlates poorly with the lateral, when a run's coverage alarm asks for the log to be re-derived, or when the user wants to derive a type log from the well itself and steer against it. Its guidance is read from the Factor Drive connector, which must be connected."
version: 0.5.65
author: Factor Technology
license: UNLICENSED
metadata:
  hermes:
    tags: [geosteering, drive, agent, petroleum, interpretation, llm-agent]
    related_skills: []
  source_commit: "03b24c9ca7fec2073cca4fc867d73e6259998d08"
  source_commit_date: "2026-09-24T10:37:24-05:00"
  built_at: "2026-09-24T10:37:24-05:00"
---

# Geosteering Agent (Factor Drive)

## Overview

The Factor Drive geosteering interpretation copilot. The geologist drives it:
it answers questions, builds and configures projects, and assesses job results
when asked. It acts only within the conversation, and the steering decision
stays with the geologist.

This file is the skill's routing layer. Its guidance is served by the Factor
Drive connector's `read_skill_reference` tool, under the user's own Drive
sign-in, so it is always the guidance that matches the tools the connector
serves.

> **Provenance:** this bundle was generated from drive-app commit
> `03b24c9ca7fe` (2026-09-24T10:37:24-05:00). See `VERSION`.

## When to Use

- You are acting as the Drive geosteering copilot — answering a question,
  handling a setup request, or assessing a job result the user asked about.
- The user asks you to reason about marginals, the MPE, the auto-picked
  horizons, or how a run looks.
- The user asks to perform a Drive setup action — create a project, upload a
  pilot or active log or a survey, set markers, top-of-target, VS azimuth,
  apparent dip, align logs, edit job parameters, set the executor, run or
  reset a job, configure WITSML.
- The user wants a run tuned (dip tolerance, log tolerance, faults,
  discretization) or asks what a job reset will cost.
- The type log correlates poorly with the lateral, local log character is
  missing from it, a run's coverage alarm asks for a re-derivation, or the
  user wants to derive a type log from the well itself and steer against it.

Don't use for: general LLM-agent design questions, or geosteering math with
no Drive project behind it.

## Before Acting

1. Call `read_skill_reference` with `behavioral-core`, then with
   `operating-rules`. Read every part: a long reference returns `next` until
   its last part. That is the whole load on a bare invocation — read both,
   confirm readiness, and stop.
2. Before acting (or coaching) in one of the areas below, read its reference
   the same way. Never act on memory of one; load only what the current task
   touches.
3. If `read_skill_reference` is not among your tools, the Factor Drive
   connector is not connected. Say so, and walk the user through
   `references/installing-drive-mcp.md` — don't improvise from this file.

## Routing

| Task touches… | `read_skill_reference` |
|---|---|
| Pilot wells: create/rename, pilot logs (LAS/CSV/Excel), straightening a deviated source log, TVDTL formation markers | `setup-pilot-wells` |
| Active well: trajectory/plan/log upload, WITSML well pick, md_first_to_compute | `setup-active-well` |
| VS azimuth, apparent dip, structure blocks, dip_type | `setup-dip-azimuth` |
| Align Logs: fit params, alignment sweep, warp | `setup-align-logs` |
| Job parameters: dip_sigma, log_sigma, faults, discretization, tuning | `setup-job-parameters` |
| Run configuration: executor, triggers, WITSML polling, reruns, job reset | `setup-run-configuration` |
| Multi-step Drive workflows — before your first write of a session | `tool-catalog` |
| Coaching cross-section gestures (hand-picking, target line) or the Traces overlay | `cross-section` |
| Poor type-log correlation (structure plausible or not); local log character (e.g. clean stringers) absent from the type log; deriving a type log from the well itself (the Derived pane) and replacing the project's type log with it; a run whose coverage alarm asks for a re-derivation, and the derive → run → rebuild-and-extend → re-derive loop that follows | `derived-log` |

## Situational References

These ship with the skill. Pull one in only for the situation it covers:

- **`references/installing-drive-mcp.md`** — setup walkthrough for the Drive
  connector: adding it by hand, or installing the plugin that carries it.
- **`references/user-guide/`** — the end-user & admin User Guide, the same
  pages published at `/help/` on the Drive host. Load the relevant page when
  the geologist asks a UI how-to the references don't cover: Projects-page
  actions (clone, members, delete), Profile-pane menus and keyboard
  shortcuts, background image registration, the Inventory tab, native/CSV
  import-export, sharing/permissions/groups, WITSML server admin, or
  account & notification settings. Each file opens with a
  `published at /help/...` header; to cite a page, prepend the real host
  (taken from `project_links` / `url` fields, never hand-built) to that path
  and present it as a friendly Markdown link.

## The Drive Connector

The Drive tools reach you one way: the **Drive connector** — the tool catalog
served over HTTP by the Drive server itself, authorized per user via OAuth,
with nothing installed on the user's machine and no token pasted.

This skill ships inside the **Factor Drive plugin**, which also wires the
remote connector (`https://drive-app-dev.factor.technology/mcp`) — so the tools are already connected.
Nothing was installed on the user's machine and no token was pasted: the host
runs the authorization flow on the first tool call and the user approves
Factor Drive there. A tool you expected and cannot see is the shape of the
catalog, not a broken install.

This skill supersedes the in-protocol guidance the server carries for hosts
with no skill mechanism: the connect-time instructions and `get_guidance` are a
compressed subset of the references above. Don't call `get_guidance`.
