# Power Apps proof of concept — first build

## Goal for phase 1

Prove that a canvas app can read and write a SharePoint list for two authorized users. This is a proof of concept, not the complete Ticket Tracker/Portal replacement. Do not import production data yet.

Official Microsoft instructions:
- Create an app from a list: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/app-from-sharepoint
- Add SharePoint lists to an app: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-sharepoint-online

## Step 1 — Create the Tickets list

On the target SharePoint site, choose New > List > Blank list. Name it exactly `Tickets`. Create these columns. SharePoint's built-in `Title` column can remain and may be renamed to `Ticket Summary` for display.

| Column display name | SharePoint type | Notes |
|---|---|---|
| TicketKey | Single line of text | Stable unique key; turn on Enforce unique values if available |
| Customer | Single line of text | Customer/ship name |
| Complaint | Multiple lines of text | Plain text |
| AgentId | Single line of text | Agent who owns/created the ticket |
| Status | Choice | New, Running, Paused, Finished, Removed, Deleted |
| Category | Single line of text | Use existing tracker categories |
| CompetencyTrainingReadiness | Number | Optional 0–5 rating; blank means not rated |
| CreatedAt | Date and time | Include time |
| StartedAt | Date and time | Optional |
| LastResumeAt | Date and time | Optional; used while running |
| PausedAt | Date and time | Optional |
| CompletedAt | Date and time | Optional |
| AccumulatedMilliseconds | Number | Total accumulated active time excluding current running segment |
| SchemaVersion | Number | Start with 1 |

Keep SharePoint's built-in ID, Created, Modified, Created By, and Modified By columns. Use the built-in item ID for internal operations, but use TicketKey as the stable business key. Avoid using agent PINs or the old local PIN hashes as SharePoint authentication; use Microsoft 365 sign-in and SharePoint permissions.

For a first smoke test, manually add one clearly fake test ticket (e.g. TicketKey `POC-0001`) and no real customer data.

## Step 2 — Generate the starter canvas app

1. Open the Tickets list.
2. Choose Integrate > Power Apps > Create an app (the menu may be under the Microsoft Lists interface).
3. Name it `Ticket Tracker POC`.
4. Save the app.
5. Run Preview and confirm the fake ticket appears.
6. Create a second test item in the app, save it, and verify that it appears in the SharePoint list.
7. Do not share/publish to the full team until list permissions are reviewed.

The generated app is useful as a connection test. It does not yet implement timers, event history, Portal metrics, or admin corrections.

## Step 3 — Add a shared data source after the smoke test passes

Create these lists before implementing the real workflow:

### TicketEvents
- Title — single line of text (event label)
- EventKey — single line of text, unique if available
- TicketKey — single line of text
- TimestampUtc — date/time
- AgentId — single line of text
- Event — single line of text
- Customer — single line of text
- Complaint — multiple lines of text
- Elapsed — single line of text initially, to preserve CSV representation until units are fully mapped
- CompetencyTrainingReadiness — number, optional
- Category — single line of text
- SchemaVersion — number

Normal app users should create event rows, not edit or delete historical event rows. Set SharePoint list permissions to match the intended role model; a hidden button in Power Apps is not an authorization boundary.

### TicketCorrections
The existing Portal correction model includes field-level overrides, added tickets, deletion/restoration flags, time adjustments/deltas, and an audit array. Do not use a simplistic one-field-per-row model until the complete correction workflow has been mapped. Start this list only after the Tickets POC works and the correction model has been specified.

## Timer semantics to preserve

The existing Tracker computes elapsed time as:
- accumulated active milliseconds, plus
- the current running segment (current time minus LastResumeAt) only while Status is Running.

On pause, fold the running segment into AccumulatedMilliseconds and clear LastResumeAt. On resume, set LastResumeAt to the current time. On completion, fold the final running segment into AccumulatedMilliseconds and set CompletedAt. Use UTC timestamps consistently. Avoid relying on a screen's refresh timer as the source of truth; calculate displayed elapsed time from persisted timestamps and refresh/recalculate while a ticket is running.

The current Tracker has events such as `ticket_created`, `ticket_started`, `ticket_resumed`, `ticket_paused`, `ticket_auto_paused`, `ticket_finished`, `ticket_deleted`, `ticket_removed_by_admin`, `ticket_edited`, `competency_rated`, `competency_cleared`, `sign_in`, `sign_out`, and `account_reset`. Validate all event-producing functions in source before cutover.

## Phase 1 acceptance criteria

- App loads Tickets from the correct SharePoint site.
- Create and edit work end-to-end.
- A second authorized user can see the same test ticket.
- A read-only user cannot edit the list item.
- No OneDrive sync, Java executable, local folder picker, external CDN, or third-party runtime service is required.

## Known caveats

- SharePoint list permissions are the data security boundary. App sharing alone does not grant list permissions.
- Power Apps may display delegation warnings for unsupported formulas; keep ticket searches/filters delegable for larger lists.
- Updates across Tickets and TicketEvents are not one atomic transaction. The full app must handle partial failure, stable event keys, and reconciliation.
- Preserve the old app and CSV files as a rollback/export path until data migration and parallel-user tests pass.
