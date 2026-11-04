# Reports

A report definition says what a report covers (a customer, a site or a
node), over what period, in what format, and when it runs: daily, weekly,
monthly or only on demand. Each time it runs, a report run records the
outcome and the names of the files it produced, which you then download from
the customer's file storage.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/report-definitions/customer/{customerID}` | List a customer's report definitions ([`listReportDefinitionsByCustomerID`](../openapi.yml)) |
| `POST` | `/report-definitions/customer/{customerID}` | Create a report definition ([`createReportDefinition`](../openapi.yml)) |
| `PUT` | `/report-definitions/{customerID}/{id}` | Update a report definition ([`updateReportDefinition`](../openapi.yml)) |
| `DELETE` | `/report-definitions/{customerID}/{id}` | Delete a report definition ([`deleteReportDefinition`](../openapi.yml)) |
| `POST` | `/report-runs/customer/{customerID}` | Run a report now ([`createReportRun`](../openapi.yml)) |
| `GET` | `/report-runs/customer/{customerID}` | List a customer's latest report runs ([`listReportRunsByCustomerID`](../openapi.yml)) |
| `GET` | `/report-runs/report-definition-id/{customerID}/{reportDefinitionID}` | List a definition's runs ([`listReportRunsByReportDefinitionID`](../openapi.yml)) |
| `GET` | `/report-runs/{customerID}/{id}` | Show a report run ([`showReportRun`](../openapi.yml)) |

Listing and showing need Viewer access to the customer; creating, changing,
deleting and running need Editor access.

## Create a report definition

A report needs `scope`, `timeRange` and `reportFormat` to produce anything,
and `targetID` unless `scope` is `customer`. This one emails a PDF covering
the last 7 days of one site every Monday at 07:30:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @definition.json \
  https://api.agrology.ag/v2/report-definitions/customer/acme-farms
```

<!-- schema: ReportDefinitionInput -->
```json
{
  "description": "North Vineyard weekly summary",
  "reportType": "summary",
  "entryType": "weekly",
  "cadenceDayOfWeek": 1,
  "cadenceHour": 7,
  "cadenceMinute": 30,
  "emailEnabled": 1,
  "recipients": ["grower@example.com"],
  "scope": "site",
  "targetID": "north-vineyard",
  "timeRange": "l7d",
  "reportFormat": "pdf"
}
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "9d3b6a2f-4e1c-4f7a-8b0d-1c5e7a9f2b64"}
```

Schedule fields:

- `entryType`: `daily`, `weekly`, `monthly` or `manual`.
- `cadenceDayOfWeek`: 1 is Monday and 7 is Sunday.
- `cadenceDayOfMonth`: the day of a monthly report, or `99` for the last day.
- `cadenceHour` and `cadenceMinute`: the time of the run.
- `timeRange`: a relative range such as `4w-`, or one of `l7d`, `lw`, `lm`,
  `ly`, `mtd`, `ytd` or `snl`.

## Change or delete a definition

`PUT` changes only the fields you send. To clear a field, name it in
`unset`; `changeComment` is recorded in the change log.

<!-- schema: ReportDefinitionInput -->
```json
{"recipients": ["grower@example.com", "agronomist@example.com"], "unset": ["cc"], "changeComment": "Add agronomist"}
```

Both `PUT` and `DELETE` answer `204` with no body. A delete is permanent.

## Run a report now and fetch its files

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reportDefinitionID": "9d3b6a2f-4e1c-4f7a-8b0d-1c5e7a9f2b64"}' \
  https://api.agrology.ag/v2/report-runs/customer/acme-farms
```

The call returns as soon as the run starts, with the run's `id`. Check it
until `status` is `success`:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/report-runs/acme-farms/0b6d1f3e-7a2c-4d58-9e11-6f3a2b8c4d70
```

<!-- schema: ReportRun -->
```json
{
  "customerID": "acme-farms",
  "id": "0b6d1f3e-7a2c-4d58-9e11-6f3a2b8c4d70",
  "ts": 1755205200,
  "status": "success",
  "reportDefinitionID": "9d3b6a2f-4e1c-4f7a-8b0d-1c5e7a9f2b64",
  "executionTimeMS": 18432,
  "files": ["north-vineyard-summary.pdf"],
  "createdAt": 1755205200,
  "createdBy": "grower@example.com",
  "lastModified": 1755205219,
  "lastModifiedBy": "grower@example.com"
}
```

Each name in `files` is stored at `reports/<runID>/<name>`. Download it with
[Files](files.md):

```bash
curl -G -H "Authorization: Bearer $ACCESS_TOKEN" \
  --data-urlencode "file=reports/0b6d1f3e-7a2c-4d58-9e11-6f3a2b8c4d70/north-vineyard-summary.pdf" \
  https://api.agrology.ag/v2/fileserver/customer/acme-farms
```

## Gotchas

- **Values are stored as sent and not checked.** A definition with a bad or
  missing field still saves, and its runs can show `success` with no
  `files`.
- **`success` does not prove a file exists.** A run that failed can still
  show `success`, and `files` can name a file that was never stored. A run
  whose definition is invalid stays `new`.
- **`reportDefinitionID` is not checked when you run a report.** An unknown
  ID adds an empty definition to your list, and the run stays `new`.
- **A 400 can follow a saved change.** Creating, changing and deleting a
  definition, and starting a run, publish to the report scheduler afterwards.
  If that step fails you get a 400, but the change (or the run record) is
  saved.
- **An ID that does not exist is a 400** with `item does not exist`, not a
  404.
- **`GET /report-runs/customer/{customerID}` returns only the latest 10
  runs.** For older ones, list a definition's runs.
