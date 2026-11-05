# Experiments

An experiment compares sets of nodes over a period, such as a control block
against a trial block. Each experiment has groups, each group has members
(nodes), and the experiment's metrics say which values to roll up and at
what resolution. Agrology rolls the members' data up per group while the
experiment is `ongoing`, and the experiment data endpoints return those
rollups as one table of columns and rows per group.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/experiments/customer/{customerID}` | List a customer's experiments ([`listExperimentsByCustomerID`](../openapi.yml)) |
| `POST` | `/experiments/customer/{customerID}` | Create an experiment ([`createExperiment`](../openapi.yml)) |
| `GET` | `/experiments/{customerID}/{id}` | Get an experiment ([`showExperiment`](../openapi.yml)) |
| `PUT` | `/experiments/{customerID}/{id}` | Update an experiment ([`updateExperiment`](../openapi.yml)) |
| `DELETE` | `/experiments/{customerID}/{id}` | Delete an experiment ([`deleteExperiment`](../openapi.yml)) |
| `GET` | `/experiments/groups/{customerID}/{experimentID}` | List groups in an experiment ([`listExperimentGroupsByExperimentID`](../openapi.yml)) |
| `POST` | `/experiments/groups/{customerID}/{experimentID}` | Create an experiment group ([`createExperimentGroup`](../openapi.yml)) |
| `GET` | `/experiments/groups/{customerID}/{experimentID}/{id}` | Get an experiment group ([`showExperimentGroup`](../openapi.yml)) |
| `PUT` | `/experiments/groups/{customerID}/{experimentID}/{id}` | Update an experiment group ([`updateExperimentGroup`](../openapi.yml)) |
| `DELETE` | `/experiments/groups/{customerID}/{experimentID}/{id}` | Delete an experiment group ([`deleteExperimentGroup`](../openapi.yml)) |
| `GET` | `/experiments/members/{customerID}/{experimentID}/{groupID}/{nodeID}` | Get a node in an experiment group ([`showExperimentMember`](../openapi.yml)) |
| `POST` | `/experiments/members/{customerID}/{experimentID}/{groupID}/{nodeID}` | Add a node to an experiment group ([`createExperimentMember`](../openapi.yml)) |
| `PUT` | `/experiments/members/{customerID}/{experimentID}/{groupID}/{nodeID}` | Update a node in an experiment group ([`updateExperimentMember`](../openapi.yml)) |
| `DELETE` | `/experiments/members/{customerID}/{experimentID}/{groupID}/{nodeID}` | Remove a node from an experiment group ([`deleteExperimentMember`](../openapi.yml)) |
| `GET` | `/experiments/members/{customerID}/group-id/{groupID}` | List an experiment group's nodes ([`listExperimentMembersByGroupID`](../openapi.yml)) |
| `GET` | `/experiments/members/{customerID}/node-id/{nodeID}` | List a node's experiment memberships ([`listExperimentMembersByNodeID`](../openapi.yml)) |
| `GET` | `/experiments/members/{customerID}/experiment-id/{experimentID}` | List an experiment's nodes ([`listExperimentMembersByExperimentID`](../openapi.yml)) |
| `GET` | `/experiments/metrics/{customerID}/experiment-id/{experimentID}` | List an experiment's metrics ([`listExperimentMetricsByExperimentID`](../openapi.yml)) |
| `POST` | `/experiments/metrics/{customerID}/experiment-id/{experimentID}` | Add an experiment metric ([`createExperimentMetric`](../openapi.yml)) |
| `GET` | `/experiments/metrics/{customerID}/{experimentID}/{id}` | Get an experiment metric ([`showExperimentMetric`](../openapi.yml)) |
| `PUT` | `/experiments/metrics/{customerID}/{experimentID}/{id}` | Update an experiment metric ([`updateExperimentMetric`](../openapi.yml)) |
| `DELETE` | `/experiments/metrics/{customerID}/{experimentID}/{id}` | Remove an experiment metric ([`deleteExperimentMetric`](../openapi.yml)) |
| `GET` | `/experiments/data/{customerID}/e/{experimentID}/{resolution}/{timeRange}` | Get experiment data ([`getExperimentDataByExperimentID`](../openapi.yml)) |
| `GET` | `/experiments/data/{customerID}/e/{experimentID}/g/{groupID}/{resolution}/{timeRange}` | Get experiment group data ([`getExperimentDataByExperimentIDAndGroupID`](../openapi.yml)) |
| `POST` | `/experiments/data/{customerID}/e/{experimentID}/run` | Re-run experiment data ([`requestExperimentDataReRun`](../openapi.yml)) |
| `POST` | `/experiments/data/{customerID}/e/{experimentID}/regenerate` | Regenerate experiment data ([`requestExperimentDataRegenerate`](../openapi.yml)) |

Reading needs Viewer access to the customer; creating, changing and deleting
need Editor access. Re-run and regenerate are for Agrology staff: they need
Editor access and Agrology debugger access.

## Create an experiment

Set `startTime` and `endTime` in epoch **milliseconds**; every other time on
these records is in seconds. Rollups run only while `status` is `ongoing`,
and the first run uses up the build that create asks for, so create the
experiment as `planned` and set it to `ongoing` once its groups, members and
metrics are in place.

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @experiment.json \
  https://api.agrology.ag/v2/experiments/customer/acme-farms
```

<!-- schema: ExperimentInput -->
```json
{
  "description": "Cover crop trial",
  "status": "planned",
  "notes": "Block 1 East is the control, Block 2 West the trial.",
  "startTime": 1793750400000,
  "endTime": 1809302400000,
  "positionFilter": "Soil Chamber"
}
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41"}
```

`positionFilter` keeps one position for ground-truth metrics; synthetic
metrics use every position.

## Add groups, members and metrics

Create the groups first. `groupType` is free text; the portal offers
`control`, `trial` and `other`.

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"description": "Control", "groupType": "control", "color": "#1f77b4"}' \
  https://api.agrology.ag/v2/experiments/groups/acme-farms/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "a7c2e4f6-1b3d-4a5c-9e8f-0d2b4c6a8e10"}
```

Then add each node to its group. The node goes in the path, and the body
repeats the path's `experimentID`:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"experimentID": "3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41"}' \
  https://api.agrology.ag/v2/experiments/members/acme-farms/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41/a7c2e4f6-1b3d-4a5c-9e8f-0d2b4c6a8e10/3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17"}
```

Then add the metrics. Each needs `metricID` and a `resolution` in seconds
(1800, 3600, 14400, 21600, 43200, 86400, 259200, 604800 or 2592000).
`nodeAggregator` combines each node's samples in a period, and `statistic`
then combines the group's nodes; both default to `mean`.

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @metric.json \
  https://api.agrology.ag/v2/experiments/metrics/acme-farms/experiment-id/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41
```

<!-- schema: ExperimentMetricInput -->
```json
{"metricID": "soilMoisture", "resolution": 86400, "nodeAggregator": "mean", "statistic": "mean"}
```

Finally, start the rollups:

```bash
curl -X PUT -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status": "ongoing", "changeComment": "Start trial"}' \
  https://api.agrology.ag/v2/experiments/acme-farms/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41
```

Updates answer `204` with no body. The experiment's `notice` then tells
users that a rebuild is pending.

## List an experiment's groups and nodes

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/experiments/members/acme-farms/experiment-id/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41
```

<!-- schema: ExperimentMembers -->
```json
[
  {
    "groupID": "a7c2e4f6-1b3d-4a5c-9e8f-0d2b4c6a8e10",
    "nodeID": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
    "experimentID": "3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41",
    "createdAt": 1793796000,
    "createdBy": "grower@example.com",
    "lastModified": 1793796000,
    "lastModifiedBy": "grower@example.com"
  },
  {
    "groupID": "c4e6a8b0-2d4f-4b6a-8c0e-1f3a5c7e9b21",
    "nodeID": "8a2d5c71-0e3f-4b9a-b6d2-1c4e7f9a0b38",
    "experimentID": "3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41",
    "createdAt": 1793796060,
    "createdBy": "grower@example.com",
    "lastModified": 1793796060,
    "lastModifiedBy": "grower@example.com"
  }
]
```

To see one group's nodes, use the `group-id` list; to see every group a
node is in, use the `node-id` list, where an experiment can appear more than
once.

## Get experiment data

The path carries the rollup `resolution` (one of the metric resolutions
above) and a [time range](../README.md#time-ranges). The response is always
[response version](../README.md#response-versions) 2.2, whatever
`responseVersion` says, and `format=csv` still returns JSON.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/experiments/data/acme-farms/e/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41/86400/1793750400-1794009600"
```

<!-- schema: ExperimentData -->
```json
{
  "status": "ok",
  "request": {
    "requestTime": 1794016000,
    "source": "rollups/experiments",
    "timeRange": "1793750400-1794009600",
    "format": "json",
    "responseVersion": "2.2"
  },
  "organization": "acme-farms",
  "metrics": {
    "soilMoisture": {"id": "soilMoisture", "name": "Soil Moisture"}
  },
  "experiment": {
    "id": "3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41",
    "description": "Cover crop trial",
    "groups": [
      {
        "id": "a7c2e4f6-1b3d-4a5c-9e8f-0d2b4c6a8e10",
        "description": "Control",
        "groupType": "control",
        "color": "#1f77b4",
        "data": {
          "start": 1793750400,
          "end": 1793923200,
          "resolution": 86400,
          "metrics": ["soilMoisture"],
          "nodeAggregators": ["mean"],
          "statistics": ["mean"],
          "columns": ["ts", "soilMoisture|mean|mean"],
          "rows": [
            [1793750400, 31.2],
            [1793836800, 30.8],
            [1793923200, null]
          ]
        }
      }
    ]
  }
}
```

Each group's `data.columns` is `ts` followed by one
`metric|nodeAggregator|statistic` column per metric, with the statistic
lowercased and empty parts shown as `mean`. Each row follows `columns`
order: `ts` in epoch seconds, then a number or `null` per column.

To narrow the columns, send `cols` with column names (in any case), such as
`cols=soilMoisture|mean|mean`. Or filter by `metrics` (metric IDs) and
`statistic` (the statistic part of the column); both are ignored when `cols`
is present. For one group only, use the
`/experiments/data/{customerID}/e/{experimentID}/g/{groupID}/{resolution}/{timeRange}`
path. A response over 5,000,000 bytes is a redirect; see
[Errors](../README.md#errors).

## Re-run or regenerate the data (Agrology staff)

`run` asks for the rollup to run again; `regenerate` asks for it to be
rebuilt from scratch. Both answer `204` with no body.

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/experiments/data/acme-farms/e/3f8a1c2e-5b7d-4e9a-a1c3-7d2f6b8e0a41/regenerate
```

Without Agrology debugger access the answer is a 401 whose body is not valid
JSON.

## Gotchas

- **A 400 can follow a saved change.** Creating and updating an experiment,
  adding, updating and removing a metric, and adding and removing a member
  also update the experiment's `notice`. If that step fails you get a 400,
  but the change is saved. A retry after that 400 creates a second
  experiment or metric, and a second add of the same member fails, so list
  first. Group writes, member updates and deleting an experiment skip the
  notice.
- **`startTime` and `endTime` are milliseconds.** A value in seconds is read
  as a date in January 1970. Send `startTime` on create: without it rollups
  start from 1970. An `endTime` in the past ends the experiment: a rollup run
  sets `status` to `completed`, and rollups then stop.
- **Rollups run only while `status` is `ongoing`.** An update or metric
  change on a `planned` experiment sets a `notice` saying a rebuild is
  pending, and the rebuild runs once the status is `ongoing`. On a
  `completed` experiment nothing runs it.
- **Finish groups, members and metrics before setting `ongoing`.** The first
  rollup run of an `ongoing` experiment consumes the build that create asked
  for, even with no groups yet, and group writes do not ask again.
- **Member writes need the node to be the customer's.** Adding or updating a
  member needs an active node at one of the customer's active sites, or you
  get a 404. Send `experimentID` in the body too, the same non-empty value as
  the path; a different value on update moves the member to that
  experiment's node list. Adding a node already in the group is a 400.
- **Metric values are not checked.** An unknown `statistic` or
  `nodeAggregator` stops every rollup for the experiment, and an experiment
  whose only metric has an unknown `metricID` also fails, with no error from
  the API. A `resolution` outside the list above is stored, but the data
  operations reject it. `statistic` is case-sensitive.
- **`rawValue` and `runningTotal` need care.** `rawValue` takes one node's
  value, so use it only for single-node groups. `runningTotal` accumulates
  only within each rollup run: after a full rebuild it is a total from
  `startTime`, until the next scheduled run restarts the latest periods from
  zero.
- **Metrics can share a column.** Metrics that render the same column name
  at the same `resolution` share one, so an empty `statistic` and `mean`
  share one.
- **Old data can outlive a metric change.** Changing a metric's `resolution`,
  or removing the last metric at a resolution, does not drop the data at the
  old resolution, so it still returns.
- **Deletes do not cascade.** Deleting an experiment leaves its groups,
  members, metrics and rollup data. Deleting a group leaves its members, and
  its rows stay in the experiment data, with placeholder group details, until
  the next rebuild.
- **Do not add a query `resolution` to a data request.** It changes the
  `statistic` filter, and sets one even with no `statistic`. `runningTotal`
  as a `statistic` filter drops the filter.
- **Not found is sometimes 400.** An unknown experiment ID on the experiment
  operations is a 400 with `item does not exist`. Under an experiment, an
  unknown parent in the path is a 404 and an unknown item is a 400 with `item
  does not exist`: on group operations an unknown experiment is a 404 and an
  unknown group a 400; on member operations an unknown experiment or group is
  a 404 and a node that is not in the group a 400; on metric operations an
  unknown experiment is a 404 and an unknown metric a 400.
