# SharePoint Online migration plan for Ticket Portal + Ticket Tracker

## Status

Design and compatibility findings only. This branch does not claim that the two existing HTML applications have been converted or validated in SharePoint. Do not deploy this document as an application.

## Existing behavior inspected

- Portal repository: `CTMJUAN/ticket-metrics-dashboard`, source branch `ticket-portal`, file `ticket-portal.html`.
- Tracker repository: `CTMJUAN/Ticket-Time-Tracker`, branch `main`, file `ticket-tracker-4.12.html`.
- The portal relies on the File System Access API (`showDirectoryPicker`) and scans CSV files from a user-selected folder. It reads admin-corrected ticket records and work-log CSVs from that folder.
- The tracker stores tickets/events in browser IndexedDB and exports event logs to per-agent, per-month CSV files in a selected folder.
- These local browser stores and local files are not a multi-user shared database. Removing OneDrive sync without replacing this storage contract would break cross-user sharing.

## Critical SharePoint Online deployment constraint

Uploading the current HTML files to an ordinary document library is not a reliable deployment strategy for an interactive application. SharePoint Online may download HTML files instead of executing them depending on tenant/site configuration. Microsoft's newer HTML page experience runs HTML in a sandboxed iframe and documents that arbitrary `fetch()` requests/outbound API calls are not supported and browser `localStorage` is unavailable there.

References:
- https://support.microsoft.com/en-us/sharepoint/create-edit-publish-html-pages
- https://learn.microsoft.com/en-us/sharepoint/dev/solution-guidance/security-permissivesetting
- https://learn.microsoft.com/en-us/sharepoint/allow-or-prevent-custom-script

Therefore, the requested combination—interactive app, direct SharePoint list API access, no external runtime calls, and no admin privileges—cannot be assumed to work by merely uploading the existing HTML. Validate the allowed hosting and scripting model with the site owner before committing to a runtime.

## Recommended target

Use a SharePoint Framework (SPFx) web part with all JavaScript/CSS bundled into the package and hosted by SharePoint. Use same-site SharePoint REST APIs for data. Do not load scripts, fonts, analytics, APIs, or other assets from external domains at runtime. If SPFx deployment requires an app catalog/admin approval, request that minimum approved deployment step; do not attempt to bypass it.

If SPFx cannot be approved, pause the interactive-app migration and evaluate an approved Microsoft 365 alternative with the site owner. A static HTML upload is not a substitute for a supported application host.

## Proposed shared data model

Create lists only if the current account has permission; otherwise ask a site owner to create them. Use internal column names that are stable and avoid renaming them after deployment.

### Tickets list

- Title: human-readable ticket/customer summary
- TicketKey: single-line text, unique/enforced if supported; stable identifier shared by both apps
- Customer: single-line text
- Complaint: multiple lines of text
- AgentId: single-line text
- Status: choice (New, Running, Paused, Completed, Cancelled; map to existing source values before migration)
- Category: single-line text or choice based on current categories
- CompetencyTrainingReadiness: preserve current field meaning
- CreatedAtUtc, StartedAtUtc, PausedAtUtc, CompletedAtUtc: date/time fields as required by the existing lifecycle
- AccumulatedMilliseconds: number
- LastResumeAtUtc: date/time, nullable
- SchemaVersion: number

### TicketEvents list (append-only)

- EventKey: unique stable idempotency key
- TicketKey: lookup or text key matching Tickets.TicketKey
- TimestampUtc: date/time
- AgentId: single-line text
- Event: single-line text preserving existing event names
- Customer, Complaint: preserve the current work-log fields if needed for historical reporting
- Elapsed: preserve existing elapsed value/units exactly
- CompetencyTrainingReadiness, Category: preserve current event columns
- SourceClientId: optional diagnostic identifier (not authentication)
- SchemaVersion: number

### TicketCorrections list

- CorrectionKey: unique key
- TicketKey: stable ticket key
- Correction payload: explicit fields matching the portal's existing correction model
- Reason: required text
- CreatedAtUtc, CreatedBy: audit metadata
- SupersedesCorrectionKey: optional link to a correction being replaced

The final schema must be checked against the actual parser, admin correction model, all event types, and existing CSV headers before implementation. Do not delete or silently rewrite original event history.

## Integration contract

1. Both apps call a single shared repository/service layer for ticket reads and writes.
2. Ticket creation uses a stable TicketKey and an idempotency key to prevent duplicate creation on retry.
3. Every lifecycle transition writes the current ticket state and an event; handle partial failures explicitly and reconcile rather than assuming multi-list operations are transactional.
4. TicketEvents is append-only for normal users. Corrections are separate records and should require an appropriate SharePoint permission.
5. Use SharePoint's authenticated same-site context and existing user permissions. Never embed credentials, client secrets, or broad app tokens in HTML/JavaScript.
6. Use ETags/If-Match or equivalent concurrency checks for ticket updates. Handle 401/403/409/429 responses, retry-after, paging, and throttling.
7. Preserve the existing dashboard calculations and CSV import/export as migration/backup utilities, not as the live shared data transport.
8. Keep a schema version and an explicit migration/import tool. Validate row counts, duplicate keys, timestamps/time zones, elapsed units, CSV escaping, and correction semantics before cutover.

## No-external-network acceptance criteria

- Runtime network log contains only the SharePoint tenant/site endpoints required for the app and normal Microsoft 365 sign-in.
- No requests to public CDNs, third-party APIs, analytics, telemetry, or remote font/image hosts.
- All code dependencies are bundled with the approved SPFx package.
- No OneDrive sync client or per-user folder picker is required.
- A ticket created by one authorized user appears for another authorized user after refresh.
- Concurrent edits are detected; retries do not duplicate events.
- Users without edit permissions cannot modify tickets/corrections they are not authorized to change.
- App behavior is tested with the actual site's permission model and browser policies.

## Work still required before a functional implementation

- Confirm whether the site allows HTML pages, whether the HTML page experience is enabled, and whether the user can create lists.
- Confirm whether an SPFx package can be installed by an authorized site/app-catalog owner. An app catalog and tenant configuration are not assumed available.
- Inspect the complete ticket lifecycle and admin correction functions in both source files and map every field/event before implementing the adapter.
- Build and test the SPFx app in a development site with representative permissions and migrated test data.
- Only then replace the current UI storage calls with the SharePoint repository layer and open a reviewed pull request.
