<!-- published at /help/guide/agent.html on the Drive host -->

# The Geosteering Agent for Claude, ChatGPT and Other Hosts

Factor Drive ships a geosteering agent that works inside the AI assistant you already use: Claude (claude.ai, Claude Desktop, Claude Code, Cowork), ChatGPT, Microsoft Copilot, Hermes, or any other host that takes MCP connectors over HTTP. It gives the assistant the same operations the application exposes: reading a run's results, setting up and tuning a project, alignment, reruns and resets, WITSML, archives.

It has two parts:

- **The connector** carries the tools. It is served by Drive itself at your Drive host with `/mcp` appended (for the production service, `https://drive.factor.technology/mcp`). Every host uses it, and it also tells the assistant Drive's conventions when it connects, so the agent behaves sensibly on hosts that have no notion of a skill.
- **The skill** carries the fuller guidance and the `/geosteering-agent` and `/cross-section-setup` commands. It is delivered as a plugin on Claude hosts. Other hosts do without it; the connector alone is enough.

Nothing installs on your machine and there is no token to paste. The first use opens your browser to sign in to Drive and approve the permissions, so the agent can do exactly what you can do in the application, on exactly the projects you can see.

## Installing

### Claude

- **Claude for Teams or Enterprise.** An organization owner adds the `factor-drive` custom connector (the URL above) under **Organization settings → Connectors**, then distributes the **Geosteering agent** plugin from the `factor-technology/drive-plugins` GitHub repository under **Organization settings → Plugins → GitHub Sync**. Each member then opens **Customize → Connectors**, finds **factor-drive**, clicks **Connect**, and allows it in the browser.
- **Individual plans.** **Customize → Plugins → Add → Add marketplace → Add from a repository**, enter `factor-technology/drive-plugins`, and sync. Install **Geosteering agent**, open its settings, and on the **Connectors** tab install and **Connect** factor-drive; allow it in the browser.

Optionally, on the factor-drive connector set its two tool-permission groups — **Read-only tools** and **Write/delete tools** — from *Needs approval* to *Always allow*, so Claude does not pause before each Drive operation.

### ChatGPT

ChatGPT takes the connector but not the skill, so there are no slash commands; ask in plain language and the agent works the same way.

1. Turn on **Developer mode**: **Settings → Apps & Connectors → Advanced settings** (OpenAI has moved this toggle; if it is not there, look under **Settings → Security**). Custom connectors are gated by plan, and on a Business, Enterprise, or Edu workspace an administrator may have to enable them.
2. **Settings → Apps & Connectors → Create**. Name it `factor-drive`, enter the `/mcp` URL above, and leave authentication as OAuth with no client ID or secret. ChatGPT discovers the sign-in flow on its own.
3. Sign in to Drive in the browser window that opens and allow the permissions.
4. In a conversation, enable **factor-drive** from the tools menu (the **+** button) and ask away.

### Other hosts

Any host that takes MCP connectors over HTTP (Microsoft Copilot, Hermes, Claude Code, …) can add the `/mcp` URL as a custom connector in its connector or integration settings. The host discovers the sign-in flow on its own; there is no client ID, secret, or callback to configure. Tools appear under the connector's name once you have signed in and allowed them.

## Using it

Ask in plain language — *list my three most recent Factor Drive projects and their statuses*, *how does the latest run on Smith 1H look?* — on any host. On Claude you can also invoke the skills directly: `/geosteering-agent` for setup, tuning, and reading results, and `/cross-section-setup` to frame and tidy a project's cross section in the browser (this one drives Chrome through the Claude in Chrome extension, so it is Claude-only).

The agent reports a run in a sentence or two plus a link to the project's cross section, which shows the marginals better than prose. Edits that invalidate computed state — fit parameters, for example — are advisory: the agent explains the reset they cause and asks before applying them.

Tools are grouped into read, write, run, and admin permissions. A call outside what you approved is refused with a message naming the missing permission; reconnect the connector and approve the fuller set.

If every call fails as unauthorized, the sign-in expired or was revoked: reconnect the connector in the host's settings. There is no token to refresh by hand.
