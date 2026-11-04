# Timeseries

Timeseries endpoints return values over time for a site, one of its nodes,
or (for ground truth) one device. The sources are sensor readings (ground
truth), past weather, weather forecasts, microclimate predictions, pest and
disease predictions, synthetic microclimate data, and per-customer
microclimate summaries. Every source answers in one of three response
shapes, chosen with `responseVersion`.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/historical/ground-truth/{siteID}/{timeRange}` | Get ground-truth sensor data ([`getHistoricalGroundTruth`](../openapi.yml)) |
| `GET` | `/historical/ground-truth/{siteID}/node/{nodeID}/{timeRange}` | Get ground-truth sensor data ([`getHistoricalGroundTruthNode`](../openapi.yml)) |
| `GET` | `/historical/ground-truth/{siteID}/device/{deviceID}/{timeRange}` | Get ground-truth sensor data ([`getHistoricalGroundTruthDevice`](../openapi.yml)) |
| `GET` | `/historical/weather/{siteID}/{timeRange}` | Get historical weather data ([`getHistoricalWeather`](../openapi.yml)) |
| `GET` | `/historical/weather/{siteID}/node/{nodeID}/{timeRange}` | Get historical weather data ([`getHistoricalWeatherNode`](../openapi.yml)) |
| `GET` | `/predictions/weather/{siteID}` | Get weather forecasts ([`getPredictionsWeather`](../openapi.yml)) |
| `GET` | `/predictions/weather/{siteID}/node/{nodeID}` | Get weather forecasts ([`getPredictionsWeatherNode`](../openapi.yml)) |
| `GET` | `/predictions/weather/{siteID}/{timeRange}` | Get weather forecasts ([`getPredictionsWeatherTimeRange`](../openapi.yml)) |
| `GET` | `/predictions/weather/{siteID}/node/{nodeID}/{timeRange}` | Get weather forecasts ([`getPredictionsWeatherNodeTimeRange`](../openapi.yml)) |
| `GET` | `/predictions/microclimate/{siteID}` | Get microclimate predictions ([`getPredictionsMicroclimate`](../openapi.yml)) |
| `GET` | `/predictions/microclimate/{siteID}/node/{nodeID}` | Get microclimate predictions ([`getPredictionsMicroclimateNode`](../openapi.yml)) |
| `GET` | `/predictions/microclimate/{siteID}/{timeRange}` | Get microclimate predictions ([`getPredictionsMicroclimateTimeRange`](../openapi.yml)) |
| `GET` | `/predictions/microclimate/{siteID}/node/{nodeID}/{timeRange}` | Get microclimate predictions ([`getPredictionsMicroclimateNodeTimeRange`](../openapi.yml)) |
| `GET` | `/predictions/pest-disease/{siteID}` | Get pest & disease predictions ([`getPestDisease`](../openapi.yml)) |
| `GET` | `/predictions/pest-disease/{siteID}/node/{nodeID}` | Get pest & disease predictions ([`getPestDiseaseNode`](../openapi.yml)) |
| `GET` | `/predictions/pest-disease/{siteID}/{timeRange}` | Get pest & disease predictions ([`getPestDiseaseTimeRange`](../openapi.yml)) |
| `GET` | `/predictions/pest-disease/{siteID}/node/{nodeID}/{timeRange}` | Get pest & disease predictions ([`getPestDiseaseNodeTimeRange`](../openapi.yml)) |
| `GET` | `/synthetics/microclimate/{siteID}/{timeRange}` | Get synthetic microclimate data ([`getSyntheticsMicroclimate`](../openapi.yml)) |
| `GET` | `/synthetics/microclimate/{siteID}/node/{nodeID}/{timeRange}` | Get synthetic microclimate data ([`getSyntheticsMicroclimateNode`](../openapi.yml)) |
| `GET` | `/summaries/microclimate/customer/{customerID}` | Get forecast and actual summaries for a customer's sites ([`getMicroclimateSummaryByCustomer`](../openapi.yml)) |

Each source has a `/metrics` list of the IDs it takes in `metrics`, and
ground truth has a device and device-type list; see
[Metrics and units](metrics-and-units.md) and
[Sites and topology](sites-and-topology.md). Site and node IDs come from
`GET /access`, device IDs from
`GET /historical/ground-truth/{siteID}/devices`.

## Sources

| Source | Path prefix | What it holds |
| --- | --- | --- |
| Ground truth | `/historical/ground-truth` | readings from Agrology sensors |
| Historical weather | `/historical/weather` | past weather for the site |
| Weather forecasts | `/predictions/weather` | weather forecasts |
| Microclimate predictions | `/predictions/microclimate` | Agrology's microclimate forecasts |
| Pest and disease | `/predictions/pest-disease` | pest and disease predictions, under raw `pd`-prefixed metric keys |
| Synthetics | `/synthetics/microclimate` | synthetic microclimate metrics for past times |
| Microclimate summaries | `/summaries/microclimate/customer` | max, min and mean of one metric, past and forecast, for every site of a customer |

Historical weather comes from the US National Weather Service
(<https://www.weather.gov>) for US sites and MET Norway
(<https://api.met.no>, licensed under CC BY 4.0,
<https://creativecommons.org/licenses/by/4.0/>) for other sites. Canadian
sites: past hours from Environment and Climate Change Canada stations (Data
Source: Environment and Climate Change Canada). Data from before the switch
in early October 2026 (about 6 October) was provided by Tomorrow.io.

Weather forecasts come from the US National Weather Service
(<https://www.weather.gov>) for US sites and MET Norway
(<https://api.met.no>, licensed under CC BY 4.0,
<https://creativecommons.org/licenses/by/4.0/>) for other sites.

The 2.1 and 2.2 responses name the source in `request.source`, for example
`historical/ground-truth` or `predictions/weather`.

## Ordering

Samples are grouped by node and ordered by time, oldest first. Nodes can come
in any order, and the order is not guaranteed to be the same across
requests.

## Scopes

- **Site:** `/{source}/{siteID}/{timeRange}` returns every node at the site.
- **Node:** `/{source}/{siteID}/node/{nodeID}/{timeRange}` returns one node.
  On predictions and synthetics, `all` as the node ID means every node.
- **Device:** ground truth only,
  `/historical/ground-truth/{siteID}/device/{deviceID}/{timeRange}`.

Forecast sources (weather, microclimate, pest and disease) also have paths
with no `timeRange`. They run from now to the last stored forecast point.

## Time ranges

The path ends in a time range such as `4h-`, `6h-4h` or
`1618203722-1618290149`; [Time ranges](../README.md#time-ranges) gives the
formats. Two more forms matter here:

- On predictions, an empty end means the last stored forecast point, not
  now, and a relative time with a `+` prefix is in the future: `now-+2d`.
  Past forecast times are left out unless you add `showPast=1`.
- `newest` returns the latest-dated sample of each series. On site
  timeseries it looks at samples from 24 hours ago onwards, and a series with
  nothing in that window is left out. On predictions the latest-dated sample
  is the furthest-ahead forecast point while the series has future points,
  and its newest past point otherwise. On experiment data a series is one
  experiment group, with no time limit.

## Read ground truth

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/node/3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17/2h-?metrics=airTemp,humidity&resolution=3600&responseVersion=2.2"
```

The three response versions below show this request's answer.

## Filter the data

`metrics`, `deviceType`, `position` and `filter` narrow ground-truth data,
and combine. The other sources take only `metrics`, and pest and disease
ignores it.

- `metrics`: comma-separated metric IDs, e.g. `airTemp,humidity`.
- `deviceType`: comma-separated device type IDs, e.g. `gV2,smp`.
- `position`: comma-separated position names: `Soil Chamber`,
  `Ground Level`, `Canopy`, `PM` or `GPS`.
- `filter`: one `key:v1,v2` filter. Keys, in any case: `deviceType`,
  `metrics` (or `metric`), `deviceID`, `nodeID`, `position`.

```bash
curl -G -H "Authorization: Bearer $ACCESS_TOKEN" \
  --data-urlencode "metrics=soilMoisture,soilTemperature" \
  --data-urlencode "position=Soil Chamber" \
  --data-urlencode "filter=nodeID:3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17,8a2d5c71-0e3f-4b9a-b6d2-1c4e7f9a0b38" \
  --data-urlencode "responseVersion=2.2" \
  https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/1d-
```

`consolidateNodePositionMetrics=1` merges all of a node's devices at the same
position into one series, keyed `<nodeID>-<positionID>` instead of by device
ID.

## Bucket the data

`resolution` is a bucket size in seconds: a positive divisor of 86400 (such
as `900`, `3600` or `21600`) or a multiple of 86400. `2592000` means calendar
months, at response versions 2.1 and 2.2. Other positive values are ignored,
as if `resolution` were absent.

`statistic` says how samples in a bucket combine. The default is `mean`, and
unknown values also give `mean`. Without `resolution` it is ignored, except
for `runningTotal`.

| `statistic` | Below calendar months | At calendar months |
| --- | --- | --- |
| `mean`, `sum`, `min`, `max` | yes | yes |
| `median`, `geomean`, `varsamp`, `stddevsamp`, `varpop`, `stddevpop` | no: 400 | yes |
| `runningTotal` | yes | yes |

`runningTotal` sums each bucket and accumulates across buckets. Without
`resolution` it accumulates the raw samples. It accumulates only on 2.1-style
samples (see [Gotchas](#gotchas)).

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/historical/weather/north-vineyard/30d-?resolution=86400&statistic=runningTotal&responseVersion=2.1"
```

## Response versions

`responseVersion` picks the shape: `2.0` (the default), `2.1` or `2.2` (the
latest). Use 2.2 for new integrations. Here is the same ground-truth answer,
two hourly samples from one device on node `3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17`, in each.

**2.0** nests nodes in an array, with metric values flat on each sample.
`request` is never sent.

<!-- schema: TimeseriesV20 -->
```json
{
  "status": "ok",
  "site": "North Vineyard",
  "siteId": "north-vineyard",
  "nodes": [
    {
      "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
      "name": "Block 1 East",
      "samples": [
        {"ts": 1793797200, "dev": "0004a30b001c2d3e", "position": "Canopy", "deviceType": "gV2", "airTemp": 14.2, "humidity": 71.5},
        {"ts": 1793800800, "dev": "0004a30b001c2d3e", "position": "Canopy", "deviceType": "gV2", "airTemp": 16.8, "humidity": 63.0}
      ]
    }
  ]
}
```

**2.1** keys sites, devices and nodes by ID, puts values under `d`, and
moves position and device type to `devices`. `request` echoes the request.

<!-- schema: TimeseriesV21 -->
```json
{
  "status": "ok",
  "request": {
    "requestTime": 1793804400,
    "source": "historical/ground-truth",
    "siteId": "north-vineyard",
    "nodeId": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
    "timeRange": "2h-",
    "format": "json",
    "responseVersion": "2.1",
    "metrics": ["airTemp", "humidity"]
  },
  "sites": {
    "north-vineyard": {"id": "north-vineyard", "name": "North Vineyard"}
  },
  "devices": {
    "0004a30b001c2d3e": {"id": "0004a30b001c2d3e", "position": "Canopy", "deviceType": "gV2"}
  },
  "nodes": {
    "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17": {
      "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
      "name": "Block 1 East",
      "siteId": "north-vineyard",
      "samples": [
        {"dev": "0004a30b001c2d3e", "ts": 1793797200, "d": {"airTemp": 14.2, "humidity": 71.5}},
        {"dev": "0004a30b001c2d3e", "ts": 1793800800, "d": {"airTemp": 16.8, "humidity": 63.0}}
      ]
    }
  }
}
```

**2.2** adds `metrics`, with each metric's units, and trims `request`.

<!-- schema: TimeseriesV22 -->
```json
{
  "status": "ok",
  "request": {
    "requestTime": 1793804400,
    "source": "historical/ground-truth",
    "timeRange": "2h-",
    "format": "json",
    "responseVersion": "2.2"
  },
  "metrics": {
    "airTemp": {"id": "airTemp", "name": "", "units": "°C", "type": "number"},
    "humidity": {"id": "humidity", "name": "", "units": "%", "type": "number"}
  },
  "sites": {
    "north-vineyard": {"id": "north-vineyard", "name": "North Vineyard"}
  },
  "devices": {
    "0004a30b001c2d3e": {"id": "0004a30b001c2d3e", "position": "Canopy", "deviceType": "gV2"}
  },
  "nodes": {
    "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17": {
      "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
      "name": "Block 1 East",
      "siteId": "north-vineyard",
      "samples": [
        {"dev": "0004a30b001c2d3e", "ts": 1793797200, "d": {"airTemp": 14.2, "humidity": 71.5}},
        {"dev": "0004a30b001c2d3e", "ts": 1793800800, "d": {"airTemp": 16.8, "humidity": 63.0}}
      ]
    }
  }
}
```

Values are in the metric's stored units whatever the site's display units;
[Metrics and units](metrics-and-units.md) covers converting them. `refID`,
any string you send, comes back as `request.refID` at 2.1 and 2.2.

## Read forecasts

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/predictions/microclimate/north-vineyard/node/3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17?metrics=airTemp&responseVersion=2.2"
```

Weather and microclimate predictions and synthetics keep 2.0-style flat
samples at 2.2, except at calendar-month resolution:

<!-- schema: TimeseriesV22 -->
```json
{
  "status": "ok",
  "request": {
    "requestTime": 1793804400,
    "source": "predictions/microclimate",
    "format": "json",
    "responseVersion": "2.2"
  },
  "metrics": {
    "airTemp": {"id": "airTemp", "name": "", "units": "°C", "type": "number"}
  },
  "sites": {
    "north-vineyard": {"id": "north-vineyard", "name": "North Vineyard"}
  },
  "nodes": {
    "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17": {
      "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
      "name": "Block 1 East",
      "siteId": "north-vineyard",
      "samples": [
        {"ts": 1793808000, "airTemp": 18.4},
        {"ts": 1793811600, "airTemp": 19.1}
      ]
    }
  }
}
```

## Summarize a customer's sites

The summary gives the maximum, minimum and mean of one metric for every site
of the customer you can access: `count` + 1 past steps from sensors, then
forecast rows.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/summaries/microclimate/customer/acme-farms?metrics=airTemp&step=3600&count=2&afterCount=2&source=mc&responseVersion=2.2"
```

| Parameter | Required | Meaning |
| --- | --- | --- |
| `metrics` | yes | one of `airTemp`, `humidity`, `dewPoint`, `vpd` (`metric` also works) |
| `step` | yes | step length in seconds, 3600 to 86400 |
| `count` | yes | past window, 0 to 6 |
| `afterCount` | no | forecast window, 0 to 6, and at most 10 with `count`; defaults to `count` |
| `afterStep` | no | seconds per `afterCount`, 3600 to 86400; defaults to `step` |
| `source` | no | forecast source: `mc` (microclimate predictions, the default) or `ws` (weather forecasts) |

The response carries the metric in `metric`, and has no `request`, `sites`
or 2.2 `metrics`. Each sample is a step; a value with no data is `null`.

<!-- schema: TimeseriesV22 -->
```json
{
  "status": "ok",
  "metric": {"id": "airTemp", "displayName": "Air Temperature", "units": "°C"},
  "nodes": {
    "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17": {
      "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
      "name": "Block 1 East",
      "samples": [
        {"ts": 1793797200, "max": 15.1, "min": 13.6, "mean": 14.2},
        {"ts": 1793800800, "max": 17.3, "min": 15.0, "mean": 16.8},
        {"ts": 1793804400, "max": null, "min": null, "mean": null}
      ]
    }
  }
}
```

## Download CSV

`format=csv` gives CSV at response versions 2.0 and 2.1. At 2.2 the body is
JSON whatever `format` says.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/1d-?metrics=airTemp,humidity&format=csv&responseVersion=2.1"
```

The columns are `ts,dateTime,nodeId,nodeName`, then one per metric. On
ground truth at 2.1, `dev`, `deviceType` and `position` come before the
metric columns when the data has them:

```text
ts,dateTime,nodeId,nodeName,dev,deviceType,position,airTemp,humidity
```

## Large responses

A response over 5,000,000 bytes is uploaded to S3 and answered with a 302.
Follow `Location`, a presigned URL that expires after 1 hour. The body says
the same:

<!-- schema: RedirectBody -->
```json
{
  "code": 302,
  "message": "redirect",
  "location": "https://example-bucket.s3.amazonaws.com/responses/abc123.json?X-Amz-Expires=3600&X-Amz-Signature=..."
}
```

With `disableRedirects=1` the status is 200 but the body is still this
object, with `"code": 302`, so check for `location` before reading it as
timeseries.

## Gotchas

- **The default is 2.0, not the latest.** Without `responseVersion`, or with
  an unknown value, you get 2.0. Send `responseVersion=2.2`.
- **Predictions and synthetics put 2.0-style samples inside 2.2.** Weather
  and microclimate predictions and synthetics have flat samples at 2.2, with
  values beside `ts` rather than under `d`, except at calendar-month
  resolution.
- **2.0 ignores calendar months and `runningTotal`.** At 2.0,
  `resolution=2592000` returns raw samples for every statistic, and
  `runningTotal` does not accumulate: it gives bucket sums with `resolution`
  and raw samples without it. The same holds at 2.2 on weather and
  microclimate predictions and synthetics below calendar-month resolution.
- **Pest and disease ignores `metrics`.** Every metric comes back under its
  raw `pd`-prefixed name, and `/predictions/pest-disease/metrics` is always
  `[]`. Only `resolution=2592000` at 2.1 or 2.2 buckets anything; other
  values leave samples raw for any `statistic`, though `runningTotal` still
  accumulates at 2.1 and 2.2. With `newest`, any other accepted `resolution`
  is a 400.
- **Do not send `resolution=0`**: the request fails with a 502. Negative
  values are not rejected and give undefined results.
- **Statistics the database lacks give a 400.** Below calendar-month
  resolution use only `mean`, `sum`, `min`, `max` or `runningTotal`;
  `median`, `geomean`, `varsamp`, `stddevsamp`, `varpop` and `stddevpop`
  fail. At calendar months, `geomean` of a series with a zero or negative
  value is also a 400.
- **Summary parameter errors read oddly.** A missing or out-of-range `step`,
  `count`, `afterCount`, `afterStep` or `metrics` gives
  `no valid timeRange provided`; an unknown metric or `source` gives
  `invalid metric name provided`. `afterCount` defaults to `count`, so
  `count=6` needs an explicit `afterCount` of 4 or less.
- **Summaries have more limits.** A customer with none of its sites
  accessible to you gives 401, and one failing site makes the whole request a
  400. Forecast rows are not combined: each forecast point gets its own row,
  with `ts` rounded to the nearest `step`, so a `ts` can repeat. Past rows are
  rounded too, so the first and last can cover only part of a step. At 2.1
  and 2.2, samples can land on the wrong node when the response has more than
  one node. Do not ask for CSV: at 2.0 and 2.1 it is a 200 with an empty
  body.
- **A mistyped filter widens the query.** A filter with no known values is
  dropped, so a bad ID in `filter`, `deviceType` or `position`, or a mistyped
  node or device ID in a ground-truth path, returns the whole site. Unknown
  `metrics` IDs are ignored, and if none is known every metric comes back.
- **Historical weather is the exception:** if no `metrics` ID is known, the
  samples have no metrics. The weather `/metrics` lists omit `airTemp`, which
  both weather sources return, and list `vpd`, which historical weather never
  returns.
- **Rows with no known node go under `none`**, a node ID that matches
  nothing in `/nodes`: on ground truth at every version, and on the other
  sources at 2.1 and 2.2. Historical weather, and the other sources at 2.0,
  keep the real node ID instead.
- **`request` is missing at 2.0 and on summaries**, so `refID` is echoed
  only at 2.1 and 2.2, and never on summaries. At 2.2, `metrics` entries
  have an empty `name`; look metrics up in `/metrics` for names and unit
  options.
- **CSV is plain.** No `text/csv` Content-Type is sent, fields are not
  quoted (a comma in a node name shifts that row), and with no columns the
  body is empty.
