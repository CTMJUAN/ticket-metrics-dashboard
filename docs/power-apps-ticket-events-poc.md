# Power Apps POC: connect TicketEvents and test an event write

This guide assumes the user has already generated a canvas app from the SharePoint Tickets list. It does not require SPFx, HTML upload, OneDrive sync, Java, or external runtime dependencies.

## 1. Create the TicketEvents SharePoint list

Create a blank list named exactly TicketEvents on the same site. Add:

| Column | Type |
|---|---|
| Title | Single line of text (built in) |
| EventKey | Single line of text; enforce unique values if available |
| TicketKey | Single line of text |
| TimestampUtc | Date and time |
| AgentId | Single line of text |
| Event | Single line of text |
| Customer | Single line of text |
| Complaint | Multiple lines of text |
| Elapsed | Single line of text |
| CompetencyTrainingReadiness | Number |
| Category | Single line of text |
| SchemaVersion | Number |

Keep the built-in ID, Created, Modified, Created By and Modified By columns. For now, do not add real customer history.

## 2. Add TicketEvents as a data source

1. Open the generated app in Power Apps Studio.
2. In the left rail, select **Data**.
3. Select **Add data**.
4. Select **SharePoint**, then the existing site connection/site URL.
5. Select TicketEvents and add it.
6. Save the app.

Power Apps only connects to lists that already exist; create the SharePoint list before adding it as a data source.

## 3. Add a temporary smoke-test button

On a test screen, add a Button with text Write test event. Set its OnSelect property to the following Power Fx formula (the sample TicketKey must exist in the Tickets list if you want to verify the relationship manually):

\`\`\`powerfx
IfError(
    Patch(
        TicketEvents,
        Defaults(TicketEvents),
        {
            Title: "POC test event",
            EventKey: Text(GUID()),
            TicketKey: "POC-0001",
            TimestampUtc: Now(),
            AgentId: User().Email,
            Event: "poc_test",
            Customer: "TEST ONLY",
            Complaint: "Synthetic test record",
            Elapsed: "00:00:00",
            SchemaVersion: 1
        }
    ),
    Notify(
        "Event write failed. Check the list connection, column names, and your SharePoint permissions.",
        NotificationType.Error
    ),
    Notify("Test event saved.", NotificationType.Success)
)
\`\`\`

If the formula editor reports that a field name is invalid, confirm the display name and the column's data type in SharePoint. If your environment uses localized Power Fx separators, use the separators Power Apps suggests.

The timestamp uses the Power Apps Now() value; SharePoint stores the date/time value and displays it in the user's configured time zone. Keep the field's meaning consistent during implementation and normalize to UTC in any migration/export layer where required.

## 4. Verify the event reached SharePoint

1. Run the app and click the button once.
2. Open the TicketEvents list separately.
3. Confirm exactly one row appears with Event = poc_test and TicketKey = POC-0001.
4. Click the button a second time and confirm a second distinct event appears. Each click intentionally uses a new GUID; this smoke test checks connectivity, not retry deduplication.
5. If the write fails, note the exact Power Apps error and whether the row appears in SharePoint anyway before retrying.

## 5. Next implementation gate

After a successful read/write test, map every current Tracker lifecycle event and every Portal correction type to the shared list model. Then build lifecycle actions with explicit error handling and reconciliation for partial failures between Tickets and TicketEvents. Do not treat two SharePoint list writes as one transaction. Do not use a hidden Power Apps button as a security boundary; enforce permissions on the SharePoint lists.
