# Ticket Metrics Dashboard

A single-file HTML dashboard for ticket export CSVs. Open `ticket-dashboard.html` in Chrome or Edge — no install, no server, and no data leaves your computer.

## Features
- Load data from a OneDrive/SharePoint-synced folder (auto-refresh at a selectable interval) or by uploading CSVs
- Totals, per-agent and per-customer breakdowns (latest ticket date, paged 10 at a time, Competency / Training Rating), complaint types (combinable, blanks ignored), top-5 customer pie chart
- Metrics log with manual and scheduled automatic snapshots, exportable as CSV
- Light and dark themes

## Versions
Each version is one commit on `main` whose message starts with its version number (e.g. `v2.9 — …`). The file name stays the same; use the commit history to compare versions or roll back.

| Version | Changes |
|---|---|
| 2.15 | Snapshot graph: Hours worked on its own right-hand scale, labelled axes, value dots with hover, latest values in the legend |
| 2.14 | Snapshot graph (Metrics over time) back on the main page; snapshot buttons, schedule and log table stay in Settings |
| 2.13 | Review + stress-test fixes: Windows-1252/UTF-16 files, title rows and `sep=` lines, unterminated quotes, non-CSV files rejected, Excel serial dates, non-English complaint labels, no blocking alert when storage is full, one auto snapshot across tabs, crash fix for very large customers, ~35% faster on big files |
| 2.12 | Status breakdown treats "Ticket_finished" (and other underscore/hyphen/"Ticket" variants) as "Finished"; "Ticket_unfinished" now correctly excluded |
| 2.11 | Combine complaint types: automatic merging of spelling variants, suggested and manual combinations in Settings, blank complaints ignored |
| 2.10 | Per-customer breakdown shows each customer's latest ticket date, sorted newest first, 10 per page with Newer/Older paging |
| 2.9 | Competency / Training Rating column in the per-customer breakdown |
