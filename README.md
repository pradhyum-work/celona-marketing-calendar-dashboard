# Celona Marketing Calendar Dashboard

An editable HTML front-end + sync engine for the **Celona Marketing Event Calendar**, built in the
same pattern as the Celona 2.0 Launch tracker (`project-tracker-suite`).

```
            edits in meetings (no lock contention)         publish (merge my task changes)
  index.html  ───────────────►  Marketing Event Calendar   ───────────────►  Working .xlsx (OneDrive)
  (Chrome, SheetJS,             (Claude Managed) .xlsx       ◄───────────────  (team views / edits)
   File System Access API)            ▲  6-hour scheduled 3-way sync   ▼
                                       └──────────── reconciles ──────────────┘
```

- **Code** (this dashboard + sync scripts) is versioned here on GitHub.
- **Data** stays on OneDrive — the `(Working).xlsx` the team opens — and never gets committed.
- The **Claude Managed** copy lives locally in `~/Claude folder/Claude managed Spreadsheets/` and is what the dashboard edits.

## Files

| File | Purpose |
|---|---|
| `index.html` | The dashboard. Open it in Chrome/Edge. Views: **By Event**, **By Due Date** (Overdue / This week / 2 weeks / High-priority P0–P1), **By Category**, **Events** (logistics, read-only reference). |
| `scripts/` | The sync engine (`apply_sync.py` + helpers) — 3-way merge of Claude Managed + Working against a baseline, with formatting + Done-archival re-applied. |
| `config.template.json` | Config schema. Copy to your local `~/Claude folder/.calendar-sync/config.json` and fill in absolute paths (already done on Pradhyum's machine). |

## How the team uses the dashboard

1. **Open** `index.html` in Chrome → pick the local `Claude managed Spreadsheets/Marketing Event Calendar (Claude Managed).xlsx`.
   (Next time: **Reopen last file**.)
2. **Edit tasks** during the meeting. Turn on **Autosave (local)** — it writes only to your local copy,
   so there's never a file-lock fight with Excel.
3. **🔗 Link OneDrive file** once → pick the `(Working).xlsx`.
4. **⬆ Publish to OneDrive** → merges *only your changed task cells* into the Working file,
   preserving edits other people made there. If it's open in Excel, close it and Publish again.
5. **✉ Email due-soon** → editable `.eml` drafts per owner (tasks due ≤ 14 days incl. overdue).

The **Events** tab is read-only reference — events and logistics are maintained through chat /
the `celona-event-pipeline` workflow, not edited live in the dashboard.

## Schema (Tasks sheet — the editable surface)

`ID · Task · Event · Owner · Assigned to · Due · Status · Priority · Category · Notes`

- Statuses: `Not started`, `In progress`, `Blocked`, `Done`
- Priorities: `P0`, `P1`, `P2`

## Email addresses (kept out of this file)

No email addresses are stored in `index.html`. The **Email due-soon** feature looks up each
owner/assignee's address at send time from a **`Directory`** sheet in the Excel file
(`Name | Email`), which stays local / on OneDrive and is never published here. To add or change
an address, edit the Directory sheet. The ⚙ Emails dialog can set a temporary per-browser override,
but the Directory sheet is the source of truth.

## Sync

A scheduled task already reconciles Claude Managed ↔ Working every 6 hours
(`celona-calendar-6h-sync`). The script-based engine in `scripts/` mirrors the Celona 2.0
mechanism and can run a manual Tasks sync:

```bash
python3 scripts/apply_sync.py --config "~/Claude folder/.calendar-sync/config.json"
```

## Hosting (optional)

`index.html` is self-contained (SheetJS loaded from CDN). Enable **GitHub Pages** on this repo to
serve it at a URL — the team can then open the dashboard from a link instead of a local file.
