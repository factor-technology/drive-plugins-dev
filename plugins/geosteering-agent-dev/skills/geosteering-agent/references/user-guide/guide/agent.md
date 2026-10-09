<!-- published at /help/guide/agent.html on the Drive host -->

# The Claude Agent

Factor Drive ships a geosteering agent for Claude: a skill plus a connector that give Claude — on claude.ai, Claude Desktop, Claude Code, or Cowork — the same operations the application exposes: reading a run's results, setting up and tuning a project, alignment, reruns and resets, WITSML, archives.

Nothing installs on your machine and there is no token to paste. The connector is served by Drive itself at your Drive host with `/mcp` appended (for the production service, `https://drive.factor.technology/mcp`). The first use opens your browser to sign in to Drive and approve the permissions, so the agent can do exactly what you can do in the application, on exactly the projects you can see.

## Installing

- **Claude for Teams or Enterprise.** An organization owner adds the `factor-drive` custom connector (the URL above) under **Organization settings → Connectors**, then distributes the **Geosteering agent** plugin from the `factor-technology/drive-plugins` GitHub repository under **Organization settings → Plugins → GitHub Sync**. Each member then opens **Customize → Connectors**, finds **factor-drive**, clicks **Connect**, and allows it in the browser.
- **Individual plans.** **Customize → Plugins → Add → Add marketplace → Add from a repository**, enter `factor-technology/drive-plugins`, and sync. Install **Geosteering agent**, open its settings, and on the **Connectors** tab install and **Connect** factor-drive; allow it in the browser.
- **Other hosts.** Any host that takes MCP connectors over HTTP (Hermes, ChatGPT, …) can add the `/mcp` URL as a custom connector. The host discovers the sign-in flow on its own; there is no client ID, secret, or callback to configure.

Optionally, on the factor-drive connector set its two tool-permission groups — **Read-only tools** and **Write/delete tools** — from *Needs approval* to *Always allow*, so Claude does not pause before each Drive operation.

## Using it

Ask in plain language — *list my three most recent Factor Drive projects and their statuses*, *how does the latest run on Smith 1H look?* — or invoke the skills directly: `/geosteering-agent` for setup, tuning, and reading results, and `/cross-section-setup` to frame and tidy a project's cross section in the browser.

The agent reports a run in a sentence or two plus a link to the project's cross section, which shows the marginals better than prose. Edits that invalidate computed state — fit parameters, for example — are advisory: the agent explains the reset they cause and asks before applying them.

Tools are grouped into read, write, run, and admin permissions. A call outside what you approved is refused with a message naming the missing permission; reconnect the connector and approve the fuller set.
