# Ticket Metrics Dashboard

A single-file HTML dashboard for ticket export CSVs. Open `ticket-dashboard.html` in Chrome or Edge — no install, no server, and no data leaves your computer.

## Features
- Load data from a OneDrive/SharePoint-synced folder (auto-refresh at a selectable interval) or by uploading CSVs
- Totals, per-agent and per-customer breakdowns (latest ticket date, paged 10 at a time, Competency / Training Rating), complaint categories and types (switchable chart; types combinable; blanks ignored), top-5 customer pie chart
- Metrics log with manual and scheduled automatic snapshots, exportable as CSV and importable from other browsers/computers; graph with selectable lines
- Light and dark themes

## Ticket Portal (dashboard + admin in one page)
`ticket-portal.html` combines this dashboard (v3.6) with the Ticket Tracker admin page (`ticket-admin.html` v1.0 from Ticket-Time-Tracker) — every feature of both, in one file with one set of tabs: **Dashboard · Admin: Tickets · Summary · Agents · Change history**.
- One shared-folder connection for both (allow editing when the browser asks, so admin corrections can be saved); the folder each separate page remembered is picked up automatically, as are the metrics log, complaint groups, snapshot schedule, theme and admin name
- Dashboard metrics use the admin-corrected tickets by default (edits, deletions, restored and added tickets count straight away); switch in Settings → *Dashboard Metrics From* → *Raw files, as they are*
- The admin's own export files (`ticket-admin-*.csv`) are skipped by the dashboard so no ticket is counted twice
- Wording for both views is in the same **EDIT LABELS HERE** list

## Changing labels
Open `ticket-dashboard.html` in a text editor and search for **EDIT LABELS HERE** (near the top of the script). Every heading, card label, column header, chart title and button label is listed there; change only the text inside the quotes, save, and reload the page.

## Versions
Each version is one commit on `main` whose message starts with its version number (e.g. `v2.9 — …`). The file name stays the same; use the commit history to compare versions or roll back.

| Version | Changes |
|---|---|
| 3.6 | Daily/Cumulative switch now covers all five summary cards (agents and avg time per ticket too); Metrics over time gets Daily / Weekly / Monthly / Cumulative views from the ticket data, plus Snapshots |
| 3.5 | Fix: ticket numbers reused by the tracker on different days are now separate tickets (each ticket_created starts a new one), so Daily shows today and In Progress counts today's paused/running tickets |
| 3.4 | Top 5 Complaints: chart shows the top 5 (category or type), with Show all to expand a full ranked list with counts and shares |
| 3.3 | Fix: newly created tickets (work-log event ticket_created) now appear in In Progress right away, and move to the totals once finished |
| 3.2 | In Progress counter next to Top Complaints: running + paused tickets from exports and work logs, longest-open first |
| 3.1 | Merges ticket-dashboard-3.0 (Claude Code: Title Case labels, summary cards no longer forced to capitals) into 2.22; empty-log message follows the snapshot button's label |
| 2.22 | All headings, card labels, column headers, chart titles and button labels gathered in one `LABELS` block (search "EDIT LABELS HERE") for easy wording/capitalisation changes |
| 2.21 | Tickets labelled "test" (customer, complaint, category or agent) left out of all metrics; switch and list in Settings → Data filters |
| 2.20 | Ratings accept the tracker's 0–5 scale; work logs: finished tickets take their latest rating/customer/complaint/category, tickets deleted after finishing are left out; Category read from work logs when present |
| 2.19 | Summary cards: Daily / Cumulative switch for tickets, customers and hours, with day-by-day navigation |
| 2.18 | Top complaints chart with a Category / Type switch (category column read automatically; hover a category to see its types); capitalised labels preferred |
| 2.17 | Main page order: summary cards → charts → Metrics over time → per-agent and per-customer breakdowns |
| 2.16 | Snapshot graph: choose lines (Tickets, Customers, Agents, Hours worked), points spaced by real date; import metrics-log CSVs (merged, duplicates skipped, older formats and Excel-resaved files OK); delete single snapshots |
| 2.15 | Snapshot graph: Hours worked on its own right-hand scale, labelled axes, value dots with hover, latest values in the legend |
| 2.14 | Snapshot graph (Metrics over time) back on the main page; snapshot buttons, schedule and log table stay in Settings |
| 2.13 | Review + stress-test fixes: Windows-1252/UTF-16 files, title rows and `sep=` lines, unterminated quotes, non-CSV files rejected, Excel serial dates, non-English complaint labels, no blocking alert when storage is full, one auto snapshot across tabs, crash fix for very large customers, ~35% faster on big files |
| 2.12 | Status breakdown treats "Ticket_finished" (and other underscore/hyphen/"Ticket" variants) as "Finished"; "Ticket_unfinished" now correctly excluded |
| 2.11 | Combine complaint types: automatic merging of spelling variants, suggested and manual combinations in Settings, blank complaints ignored |
| 2.10 | Per-customer breakdown shows each customer's latest ticket date, sorted newest first, 10 per page with Newer/Older paging |
| 2.9 | Competency / Training Rating column in the per-customer breakdown |
