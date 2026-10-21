# No-SPFx workaround: Power Apps + SharePoint Lists

## Recommendation

Use a Power Apps canvas app as the browser-based front end and SharePoint Lists as the shared data store. This avoids deploying SPFx, avoids Java/desktop executables, avoids OneDrive sync, and uses Microsoft's supported SharePoint connector instead of custom HTML making arbitrary REST requests from a sandboxed page.

Microsoft documents creating a canvas app directly from a SharePoint/Microsoft List via Integrate > Power Apps > Create an app, and connecting existing canvas apps to SharePoint lists:
- https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/sharepoint-list-integration-overview
- https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-sharepoint-online

This depends on the organization allowing Power Apps and the standard SharePoint connector, and on each user having appropriate app and list permissions/licensing. It does not bypass governance. It is not a way to host the existing HTML file unchanged; the UI and lifecycle logic need to be recreated in Power Apps formulas and controls.

## Practical implementation order

1. Verify Power Apps opens for the user and that the standard SharePoint connector is available.
2. Create the lists below (or confirm the user can create them).
3. Create a starter app from the Tickets list to verify end-user connectivity, then add the other lists as data sources.
4. Build screens: Queue, Ticket Detail, Start/Pause/Resume/Complete, My Active Work, Work Log, Dashboard, Admin Corrections.
5. Preserve the current tracker lifecycle and elapsed-time units only after mapping all existing event types and transitions from source.
6. Use TicketEvents as append-only for normal workflow. Use stable event keys and conflict/error handling where Power Apps supports it. Do not assume a multi-list operation is transactional; design a reconciliation/status field for partial writes.
7. Validate app sharing and list permissions with a second test user before migration/cutover.

## Proposed lists

### Tickets

- Title (display summary)
- TicketKey (single line text, stable key; enforce uniqueness if supported)
- Customer (single line text)
- Complaint (multiple lines)
- AgentId (single line text)
- Status (choice; map existing status values exactly before use)
- Category (single line text or choice)
- CompetencyTrainingReadiness (single line text or choice)
- CreatedAtUtc (date/time)
- StartedAtUtc (date/time, nullable)
- LastResumeAtUtc (date/time, nullable)
- PausedAtUtc (date/time, nullable)
- CompletedAtUtc (date/time, nullable)
- AccumulatedMilliseconds (number)
- SchemaVersion (number)
- LastOperationKey (single line text; optional idempotency aid)

### TicketEvents (append-only in normal use)

- Title (event display label)
- EventKey (single line text; unique if supported)
- TicketKey (single line text matching Tickets.TicketKey)
- TimestampUtc (date/time)
- AgentId (single line text)
- Event (single line text preserving existing event names)
- Customer (single line text)
- Complaint (multiple lines)
- Elapsed (single line text initially, to preserve current CSV formatting exactly; normalize only after confirming units)
- CompetencyTrainingReadiness (single line text)
- Category (single line text)
- SchemaVersion (number)

### TicketCorrections

- Title (short correction summary)
- CorrectionKey (single line text)
- TicketKey (single line text)
- FieldName (single line text)
- OriginalValue (multiple lines)
- CorrectedValue (multiple lines)
- Reason (multiple lines; required)
- CreatedAtUtc (date/time)
- CreatedBy (person)
- SupersedesCorrectionKey (single line text, optional)

This is a starting schema, not a claim that all fields have been fully mapped. Check the actual Portal correction structure and Tracker event lifecycle before production use. If existing correction records are whole-ticket snapshots rather than field-by-field changes, model them accordingly instead of using the simplified fields above.

## Security and operations

- Share the app with users through the organization's approved Power Apps process.
- Grant list permissions separately; sharing the app alone does not grant data access.
- Use existing Microsoft 365 authentication. Never embed secrets or shared credentials in client code.
- Decide whether users can edit Tickets, append events, or administer corrections; do not assume hiding a button is authorization.
- Keep an export/backup path and test import counts, duplicate TicketKeys, timestamp/time-zone handling, CSV escaping, and elapsed units.
- No external runtime dependencies are needed. Power Apps and the SharePoint connector are Microsoft-hosted services, so the app itself is not a static HTML file hosted in the document library.

## Fallback if Power Apps is unavailable

Use native SharePoint Lists and list forms for basic create/read/update, with a simplified workflow. It will not reproduce the full custom dashboard/timer experience without a supported application platform. Power Automate may help with notifications or reconciliation only if allowed, but it is not required for the initial list-backed app.

## Acceptance checklist

- Two authorized users see the same ticket records.
- Lifecycle events are not duplicated after retries.
- Start/pause/resume/complete elapsed-time behavior matches the existing Tracker.
- Portal metrics and corrections match the current baseline on a representative migrated dataset.
- Read-only users cannot change records they are not authorized to edit.
- No OneDrive sync, local folder picker, Java executable, public CDN, third-party API, or external runtime dependency is required.
