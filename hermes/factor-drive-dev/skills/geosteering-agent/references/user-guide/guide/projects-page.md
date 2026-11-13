<!-- published at /help/guide/projects-page.html on the Drive host -->

# Projects List

The Projects page is Factor Drive's landing page: every project you can read, in one table.

## The table

| Column | Meaning |
|---|---|
| **Name** | Two links: the cross-section icon opens the project's Profile view directly; the name opens the project's Setup tab. |
| **Well** | The well the project uses, with its owner in brackets when that differs from the project's owner, and a count when more than one project uses the well. |
| **Owner** | The scope the project lives in — a user or a group. |
| **Depth** | Total depth of the active well. |
| **Last** | Most recent activity on the project. |
| **Run Time** | Duration of the latest job run. |
| **Status** | Latest job status. |

All columns sort; click a header. The footer shows the total count, a page-size selector (25/50/100), and pagination. The list refreshes itself every minute.

## Filtering

The controls above the table narrow the list. They combine: a project must match all of them.

| Control | Keeps projects… |
|---|---|
| **Search** | whose name or well contains the text, as you type. |
| **Owner** | owned by any of the chosen scopes. |
| **Well** | on any of the chosen wells. |
| **Last** | whose last activity (latest run, or creation if never run) falls in the window: **Today**, **Since yesterday**, **This week**, **This month**, or **Custom range…**, which opens a calendar to pick a first and last day (both included). |
| **Status** | whose latest job is any of the chosen states: running, complete, error, or not run. |

Clear a control with its ✕. Search, filters, sort, page, and page size are all reflected in the URL, so a filtered view can be bookmarked or shared. A preset such as **Today** stays relative: the bookmark shows today's projects whenever it is opened.

You can also ask the agent: "my projects that errored this week" uses the same filters, and its reply links to this page with them applied.

## Per-row actions

At the right of each row:

- **Clone** — copies the project into a new name (*name*-clone by default) and owner of your choice: settings, pilot wells, logs, markers, prior structure, interpretations, notes, background image, and your view settings for the cross section. Not copied: the target line, notification choices, project members, and Automatic runs. Computed results are not copied, so the clone starts *un-run*, with Automatic runs off; on the original's well it keeps that well's data source, and on a new well it starts at manual file upload. The clone uses the same well as the original and shares its survey, gamma log and well plan: nothing is copied, and a change to them there changes them for both. Give it a new well name to start on an empty well instead; you then load its survey and gamma log, and the original's interpretations stay behind. Cloning is the recommended way to experiment with parameter changes without disturbing a live project.
- **Project members** — opens the access-control dialog, where you add or remove users and set their permission level. Requires ADMIN permission on the project; see [Sharing and Permissions](../admin/permissions.md).
- **Delete** — permanently deletes the project after confirmation. Requires WRITE permission. If other projects use the project's well, the well, its data and its data source stay. If it is the only project on its well, deleting it will also permanently delete the well's survey, gamma log, well plan and data source, which needs WRITE on the well.

## Creating a project

**Add Project** in the sidebar opens the **New project** page, which asks for:

| Field | Notes |
|---|---|
| **Owner** | Your personal scope, or any group scope you can write to. |
| **Name** | Must be unique within the owner. Naming the well fills this in with the well's name, with a number added if the owner already has a project of that name; a name you type yourself is kept. |
| **Well** | Optional. Left alone, the project gets a new well named after it, with a number added if that name is taken. Pick an existing well to share its survey, gamma log and well plan. |
| **Description** | Free text; editable later in the Setup tab. |
| **Spatial Measurements** | Feet or meters. Remembered as your default for next time. |

**Create project** opens the new project's Setup tab. The page also lists what you'll need.

::: tip Free trial
A free-trial account gets one project, in its own scope, with 20 runs and 20 alignments over 72 hours. Contact support@factor.technology to upgrade.
:::

## The sidebar

The left sidebar is the application's main navigation: **Add Project** (the New project page), **Import** (restore a project from a native archive — see [Import and Export](./import-export.md)), your account (see [Account and Notifications](../admin/account.md)), **Groups** (see [Sharing and Permissions](../admin/permissions.md)), **API** (the REST API reference), **Help** (this guide), a theme toggle (light / auto / dark), **Logout**, and the version number, which links to the release notes.
