# Dashboards and charts

A dashboard is a saved layout that belongs to an owner: either a user (their
email address) or a customer (`customer.<customerID>`), for a dashboard
shared with everyone on that customer. Each dashboard is filed under a
`feature` and a `featureID`, and keeps its live content in `dataField0` with
up to nine older versions for rollback. A library chart is a saved chart in a
customer's chart library, with its query and display settings stored as JSON
strings.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/dashboards/owner/{owner}/feature-id/{featureID}` | List an owner's dashboards for a feature ([`listDashboardsByFeatureIDAndOwner`](../openapi.yml)) |
| `GET` | `/dashboards/owner/{owner}` | List an owner's dashboards ([`listDashboardsByOwner`](../openapi.yml)) |
| `POST` | `/dashboards/owner/{owner}` | Create a dashboard ([`createDashboard`](../openapi.yml)) |
| `GET` | `/dashboards/{owner}/{id}` | Get a dashboard ([`showDashboard`](../openapi.yml)) |
| `PUT` | `/dashboards/{owner}/{id}` | Update a dashboard ([`updateDashboard`](../openapi.yml)) |
| `DELETE` | `/dashboards/{owner}/{id}` | Delete a dashboard ([`deleteDashboard`](../openapi.yml)) |
| `POST` | `/dashboards/{owner}/{id}` | Restore a deleted dashboard ([`undeleteDashboard`](../openapi.yml)) |
| `GET` | `/library-charts/customer/{customerID}` | List a customer's library charts ([`listLibraryChartsByCustomerID`](../openapi.yml)) |
| `POST` | `/library-charts/customer/{customerID}` | Create a library chart ([`createLibraryChart`](../openapi.yml)) |
| `GET` | `/library-charts/{customerID}/{id}` | Get a library chart ([`showLibraryChart`](../openapi.yml)) |
| `PUT` | `/library-charts/{customerID}/{id}` | Update a library chart ([`updateLibraryChart`](../openapi.yml)) |
| `DELETE` | `/library-charts/{customerID}/{id}` | Delete a library chart ([`deleteLibraryChart`](../openapi.yml)) |

You can read and change your own dashboards (`owner` is your email address).
For a `customer.<customerID>` owner, reading needs Viewer access to that
customer, and creating, changing, deleting and restoring need Editor access.
Library charts need Viewer access to the customer to list and get, and Editor
access to create, change and delete.

## Create a dashboard

`feature` and `featureID` are needed on create. This one shares a dashboard
with everyone on Acme Farms:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @dashboard.json \
  https://api.agrology.ag/v2/dashboards/owner/customer.acme-farms
```

<!-- schema: DashboardInput -->
```json
{
  "feature": "site",
  "featureID": "north-vineyard",
  "description": "North Vineyard overview",
  "order": 1,
  "dataField0": "{\"panels\":[{\"metric\":\"airTemp\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\"}]}"
}
```

The answer is `201` with the new dashboard's ID:

<!-- schema: Created -->
```json
{"status": "ok", "id": "3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36"}
```

`dataField0` is a free-form string owned by the client.

## List and get dashboards

List an owner's dashboards one feature at a time. For your own dashboards,
put your email address in the path as it is, with a literal `@`:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/dashboards/owner/grower@example.com/feature-id/north-vineyard
```

Get one by ID:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/dashboards/customer.acme-farms/3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36
```

<!-- schema: Dashboard -->
```json
{
  "id": "3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36",
  "owner": "customer.acme-farms",
  "feature": "site",
  "featureID": "north-vineyard",
  "description": "North Vineyard overview",
  "status": "active",
  "order": 1,
  "dataField0": "{\"panels\":[{\"metric\":\"airTemp\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\"},{\"metric\":\"humidity\",\"nodeID\":\"8a2d5c71-0e3f-4b9a-b6d2-1c4e7f9a0b38\"}]}",
  "dataField1": "{\"panels\":[{\"metric\":\"airTemp\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\"}]}",
  "createdAt": 1762260000,
  "createdBy": "grower@example.com",
  "lastModified": 1762263600,
  "lastModifiedBy": "grower@example.com"
}
```

Fields left empty are omitted. Both the list and the get include deleted
dashboards, with `status` `deleted`, until they expire.

## Change a dashboard

`PUT` changes only the fields you send. To clear a field, name it in
`unset`; `changeComment` is recorded in the change log. A field you both send
and name in `unset` keeps the value you sent.

```bash
curl -X PUT -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @change.json \
  https://api.agrology.ag/v2/dashboards/customer.acme-farms/3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36
```

<!-- schema: DashboardInput -->
```json
{
  "dataField0": "{\"panels\":[{\"metric\":\"airTemp\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\"},{\"metric\":\"vpd\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\"}]}",
  "unset": ["notes"],
  "changeComment": "Add VPD panel"
}
```

The answer is `204` with no body, whether or not anything changed.

Changing `dataField0` moves the old value to `dataField1` and each older
version down one place; the version in `dataField9` is lost. Empty slots are
skipped, so when the history has a gap, an older version past the gap can
stay in its slot instead of moving down.

## Delete and restore a dashboard

A delete is soft: it sets `status` to `deleted`, and you can restore the
dashboard for 7 days.

```bash
curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/dashboards/customer.acme-farms/3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36
```

To restore it, `POST` to the same path. That sets `status` back to `active`
and cancels the expiry:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/dashboards/customer.acme-farms/3f9a1c2e-6b4d-4e8f-a0c7-5d2b8e1f9a36
```

Both answer `204` with no body.

## Save a chart to the library

`definition` (the chart's query) and `settings` (how it is displayed) are
JSON documents encoded as strings, so the quotes inside them are escaped:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @chart.json \
  https://api.agrology.ag/v2/library-charts/customer/acme-farms
```

<!-- schema: LibraryChartInput -->
```json
{
  "description": "Block 1 East air temperature",
  "folder": "North Vineyard",
  "definition": "{\"siteID\":\"north-vineyard\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\",\"metric\":\"airTemp\",\"timeRange\":\"l7d\"}",
  "settings": "{\"chartType\":\"line\"}"
}
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "8c2e4a6b-1d3f-4b5a-9e7c-0f2d4b6a8c1e"}
```

List the customer's charts:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/library-charts/customer/acme-farms
```

<!-- schema: LibraryCharts -->
```json
[
  {
    "id": "8c2e4a6b-1d3f-4b5a-9e7c-0f2d4b6a8c1e",
    "customerID": "acme-farms",
    "description": "Block 1 East air temperature",
    "folder": "North Vineyard",
    "definition": "{\"siteID\":\"north-vineyard\",\"nodeID\":\"3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17\",\"metric\":\"airTemp\",\"timeRange\":\"l7d\"}",
    "settings": "{\"chartType\":\"line\"}",
    "createdAt": 1762260000,
    "createdBy": "grower@example.com",
    "lastModified": 1762260000,
    "lastModifiedBy": "grower@example.com"
  }
]
```

Parse `definition` and `settings` as JSON after you read them.

## Change or delete a chart

`PUT` follows the same convention as dashboards: only the fields you send
change, `unset` clears fields, and `changeComment` goes in the change log.

<!-- schema: LibraryChartInput -->
```json
{"folder": "Archive", "changeComment": "Season over"}
```

Both `PUT` and `DELETE` answer `204` with no body. Deleting a library chart
is permanent; there is no restore.

## Gotchas

- **`GET /dashboards/owner/{owner}` returns 401 for every caller.** Its
  access check needs a dashboard or feature ID, and this path has neither.
  List one feature at a time with
  `/dashboards/owner/{owner}/feature-id/{featureID}`; no route lists all of
  an owner's dashboards.
- **`owner` is not URL-decoded.** Send `@` as it is: `grower%40example.com`
  is not the same owner as `grower@example.com`.
- **Create without `feature` and `featureID` fails** with a 400.
- **Use the delete and restore operations, not `status`.** Setting `status`
  to `deleted` in a `PUT` sets no expiry, and setting it to `active` does not
  cancel one.
- **Deleted dashboards still appear** in the get and the feature list, with
  `status` `deleted`, until they expire. Filter on `status` if you show
  them.
- **Clearing `featureID` hides a dashboard from the feature list.** It can
  then be reached only by its ID.
- **Writing `dataField1` to `dataField9` is unreliable.** Sent with no other
  change they are ignored, and sent with a new `dataField0` they are dropped.
  `unset` cannot clear them.
- **A `204` from `PUT` does not mean something changed.** It is also the
  answer when there was nothing to change.
- **An ID that does not exist is a 400** with `item does not exist`, not a
  404.
