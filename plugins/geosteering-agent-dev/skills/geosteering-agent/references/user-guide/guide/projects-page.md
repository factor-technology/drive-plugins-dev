<!-- published at /help/guide/projects-page.html on the Drive host -->

# Projects List

The Projects page is Factor Drive's landing page: every project you can read, in one table.

## The table

| Column | Meaning |
|---|---|
| **Name** | Two links: the cross-section icon opens the project's Profile view directly; the name opens the Setup wizard. |
| **Owner** | The scope the project lives in — a user or a group. |
| **Depth** | Total depth of the active well. |
| **Last** | Most recent activity on the project. |
| **Run Time** | Duration of the latest job run. |
| **Status** | Latest job status. |

All columns sort; click a header. The search box above the table filters as you type. The footer shows the total count, a page-size selector (25/50/100), and pagination. The list refreshes itself every minute, and search, sort, page, and page size are all reflected in the URL, so a filtered view can be bookmarked or shared.

## Per-row actions

At the right of each row:

- **Clone** — copies the project (settings, pilot wells, logs, markers, active-well data, interpretations, notes, background image) into a new name and owner of your choice. The clone starts *un-run*: computed results are not copied. Cloning is the recommended way to experiment with parameter changes without disturbing a live project.
- **Project members** — opens the access-control dialog, where you add or remove users and set their permission level. Requires ADMIN permission on the project; see [Sharing and Permissions](../admin/permissions.md).
- **Delete** — permanently deletes the project after confirmation. Requires WRITE permission.

## Creating a project

**Add Project** in the sidebar opens the *Create a New Project* dialog:

| Field | Notes |
|---|---|
| **Owner** | Your personal scope, or any group scope you can write to. |
| **Name** | Must be unique within the owner; no slashes. The well name is the project name. |
| **Description** | Free text; editable later in the Setup wizard. |
| **Spatial Measurements** | Feet or meters. Remembered as your default for next time. |
| **Setup style** | **Full** — the six-step wizard. **Quick** — a one-page setup: pilot log, active well files, run. |

## The sidebar

The left sidebar is the application's main navigation: **Add Project**, **Import** (restore a project from a native archive — see [Import and Export](./import-export.md)), your account (see [Account and Notifications](../admin/account.md)), **Groups** (see [Sharing and Permissions](../admin/permissions.md)), **API** (the REST API reference), **Help** (this guide), a theme toggle (light / auto / dark), **Logout**, and the version number, which links to the release notes.
