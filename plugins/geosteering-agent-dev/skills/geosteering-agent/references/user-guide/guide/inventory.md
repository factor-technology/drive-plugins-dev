<!-- published at /help/guide/inventory.html on the Drive host -->

# Inventory

The Inventory tab lists the project's data objects and lets you inspect and download each one.

The left column lists what the project holds; an item appears once it has data:

- **Active Log** — the LWD gamma log as loaded.
- **Measured Trajectory** — the surveys.
- **Planned Trajectory** — the well plan, if uploaded.
- **Pilot Log** — one entry per pilot well, named after the well.
- **Computed Interpretation (MPE)** — once the project has run: the MPE depth per MD, in TVDTL and TVDSS. Auto-picked horizons are not listed; to export one, copy it to a manual interpretation on the Profile tab.
- One entry per **manual interpretation** that has picks.

Select an item to see its rows in the center pane; the **CSV** button on the right downloads them.

Above the list:

- **Export to Native Archive** exports the entire project as a portable archive; see [Import and Export](./import-export.md).
- **Download Latest Job File** and **Download Latest Job Log** fetch the input file sent to the computation and the engine's log for the latest run. The log appears a minute or two after the run finishes.
