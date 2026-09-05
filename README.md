# Factor Drive plugins for Claude

A Claude plugin marketplace published by [Factor Technology](https://factor.technology).
It carries the **geosteering copilot**: the interpretation skill, the
cross-section scene-setup skill, plus a live connector to your Factor Drive
projects. Nothing is installed on your machine and there is no key to paste —
you sign in to Drive the first time a tool runs.

| Plugin | Connects to |
| --- | --- |
| `geosteering-agent-dev` | https://drive-app-dev.factor.technology |

Current release: **0.5.15**.

## Install

**Claude Desktop / Cowork** — Settings → Extensions → Plugins → **Add
marketplace**, enter `factor-technology/drive-plugins-dev`, then install **geosteering-agent-dev**
from the list.

**Claude Code**

    /plugin marketplace add factor-technology/drive-plugins-dev
    /plugin install geosteering-agent-dev@factor-drive-dev

Then ask for something in Drive — *"list my Factor Drive projects"* will do. A
browser window opens the first time: sign in and approve the permissions. Every
call runs as **your** Drive user and reaches only the projects you can already
open.

## Staying up to date

The skill's prose and the Drive tool catalog ship from the same commit and
change most weeks, so an old install drifts. Installing from this marketplace
gives you an update channel — but **turn auto-update on, because it is off by
default for marketplaces that aren't Anthropic's own**:

- **Claude Code** — `/plugin`, select `factor-drive-dev`, toggle
  auto-update on.
- **Either host, declaratively** — in `~/.claude/settings.json`:

      {
        "extraKnownMarketplaces": {
          "factor-drive-dev": {
            "source": {"source": "github", "repo": "factor-technology/drive-plugins-dev"},
            "autoUpdate": true
          }
        }
      }

  Administrators can push the same block through managed settings to enable it
  fleet-wide.

Updates are applied **at startup**, so a release lands on the next restart, not
mid-session. Without auto-update, run `/plugin` and update by hand.

## What it can't do

It works through the Drive server, so it cannot read files on your own
computer: point it at a LAS or survey file on disk and it will ask you to
upload it instead. Alignment and WITSML refreshes run in the background and are
polled rather than waited on.

---

Generated from the drive-app monorepo by `yarn workspace agent
build:marketplace` — edit the sources there, not this repo. MCP server key:
`factor-drive`.
