# Agrology Public API v2

The Agrology Public API serves site configuration (customers, sites, nodes,
devices and their locations), sensor history, weather, microclimate
predictions, synthetic microclimate data, experiments, and the data behind
Agrology's apps: alerts, files, dashboards, charts, reports and customer
inputs. [`openapi.yml`](openapi.yml) is the complete reference, generated from
the service itself; the guides under [`docs/`](docs/) walk through each area.

## Base URL

```
https://api.agrology.ag/v2
```

## Authentication

Every request needs one of two credentials. A request with neither gets a
`401`.

- **Bearer token.** Log in at <https://grower.agrology.ag/>, click your
  username in the user menu, and click **Copy Access Token**. Tokens expire 1 hour after they
  are issued.

  ```bash
  curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/whoami
  ```

- **API key.** Ask Agrology staff for access. `GET /apikey-access` shows
  whether you have it. `POST /apikey` rotates your key and returns the new
  one once, so store it straight away; the previous key can keep working for
  about 10 minutes afterwards. Check `status` in the answer: a failed
  rotation is also a 200, with `"status": "error"`. The **Rotate API Key** button in the portal's
  user menu does the same.

  ```bash
  curl -H "x-api-key: $API_KEY" https://api.agrology.ag/v2/whoami
  ```

`GET /whoami` shows whose credential it is.

## Quickstart

Three requests take you from a credential to sensor data. The examples use a
bearer token; with an API key, send `x-api-key: $API_KEY` instead. The
responses below are trimmed and use example IDs.

**1. Check the credential.** A credential with no customer or site grants
yet also gets a 401 here, so if a token you just copied is refused, ask
Agrology staff to check your access.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/whoami
```

<!-- schema: WhoAmI -->
```json
{"email": "grower@example.com"}
```

**2. List what you can see.** `GET /access` lists the customers, sites and
nodes the credential can reach, and your role at each site. The site IDs and
node IDs here are what the data paths take.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/access
```

<!-- schema: AccessList -->
```json
{
  "siteAccess": [
    {
      "customer": "acme-farms",
      "displayName": "Acme Farms",
      "sites": {
        "north-vineyard": {
          "id": "north-vineyard",
          "displayName": "North Vineyard",
          "role": "editor",
          "units": "imperial",
          "nodes": [
            {"id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17", "displayName": "Block 1 East"},
            {"id": "8a2d5c71-0e3f-4b9a-b6d2-1c4e7f9a0b38", "displayName": "Block 2 West"}
          ]
        }
      }
    }
  ]
}
```

**3. Read sensor data.** This asks for the last 4 hours of air temperature
and humidity at one site, in hourly means, in the latest response shape
(see [Time ranges](#time-ranges) and [Response versions](#response-versions)).
`GET /historical/ground-truth/metrics` lists the metric IDs.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/4h-?metrics=airTemp,humidity&resolution=3600&statistic=mean&responseVersion=2.2"
```

<!-- schema: Timeseries -->
```json
{
  "status": "ok",
  "request": {
    "requestTime": 1793808000,
    "source": "historical/ground-truth",
    "timeRange": "4h-",
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

Values are in the units under `metrics` (at 2.2; pest and disease has no
`metrics`), whatever the site's display units.
The [guides](docs/) cover the other areas.

## Time ranges

Timeseries paths end in a time range: `{start}-{end}`, or a single relative
time with no end (`4h` is the same as `4h-`). An empty end means now, except
on predictions, where it means the last stored forecast point.

Start and end times may use different formats:

- **Now:** `now`.
- **Relative to now:** `<integer><unit>`, with unit `s`, `m` (minutes), `h`,
  `d` or `w`, e.g. `6d`, `20m`, `3600s`, `36h`. Combinations such as `2h5m`
  are not supported. Forecasts also take future times with a `+` prefix,
  e.g. `+2d`.
- **Epoch seconds:** exactly 10 digits, e.g. `1618203722`.
- **Epoch milliseconds:** exactly 13 digits, e.g. `1618203722000`.
- **UTC to the minute:** `yyyyMMddHHmm` (24-hour), exactly 12 digits starting with
  `20`, e.g. `202108010000` (1 Aug 2021 at midnight UTC).
- **UTC to the second:** `yyyyMMddHHmmss` (24-hour), exactly 14 digits starting with
  `20`, e.g. `20210801052530` (1 Aug 2021 at 05:25:30 UTC).

Examples:

| Range | Means |
| --- | --- |
| `1618203722-1618290149` | between two epoch times |
| `1618203722-` | from an epoch time to now |
| `4h-` | the last 4 hours |
| `6h-4h` | from 6 hours ago to 4 hours ago |
| `1d-now` | the last day |

`newest` instead of a range returns the latest sample of each series; see
[Timeseries](docs/timeseries.md).

## Response versions

Timeseries responses come in three shapes, chosen with `responseVersion`:
`2.0` (the default), `2.1` and `2.2` (the latest). Use
`responseVersion=2.2` for new integrations. An unknown value is ignored and
gives 2.0. [Timeseries](docs/timeseries.md) shows the three side by side.

## Errors

Errors from the API have a JSON body with the status and a message. A
message containing quotes makes the body invalid JSON, and API Gateway's own
`403`, `429`, `502` and `504` errors are `{"message": ...}` with no `code`,
so parse error bodies defensively:

<!-- schema: Error -->
```json
{"code": 400, "message": "no valid timeRange provided"}
```

- **400**: the request is invalid (a bad path or query parameter, or a body
  that fails validation). Many operations also answer 400 when something
  fails on Agrology's side after validation, such as a database error.
- **401**: no valid credential, or no access to what you asked for. A valid
  credential with no customer or site grants gets 401 from most operations
  (`GET /sites` answers `[]` instead). An expired token
  gives `{"code": 401, "message": "the incoming token has expired"}`, but
  other 401 bodies are not valid JSON, so read a 401 body as text.
- **302**: any response over 5,000,000 bytes is uploaded to S3 and answered
  with a redirect to a presigned URL that expires after 1 hour. The body says
  where:

  <!-- schema: RedirectBody -->
  ```json
  {
    "code": 302,
    "message": "redirect",
    "location": "https://example-bucket.s3.amazonaws.com/responses/abc123.json?X-Amz-Expires=3600&X-Amz-Signature=..."
  }
  ```

  Add `disableRedirects=1` to get status 200 instead of 302 with the same
  body (still `"code": 302`), for clients that follow redirects badly.

## Display units

Values are always in the metric's `units`. A metric's `unitOptions` list
other units it can be shown in, each with a `factor` and an optional
`offset`: for raw values, `display = value × factor + offset`.
[Metrics and units](docs/metrics-and-units.md) covers the rules for
aggregated values and the defaults for imperial and metric users.

## Weather data sources

Historical weather and weather forecasts come from the US National Weather
Service (<https://www.weather.gov>) for US sites and MET Norway
(<https://api.met.no>, licensed under CC BY 4.0,
<https://creativecommons.org/licenses/by/4.0/>) for other sites. For
Canadian sites, past hours come from Environment and Climate Change Canada
stations (Data Source: Environment and Climate Change Canada). Historical
weather from before the switch in early October 2026 (about 6 October) was
provided by Tomorrow.io.

## Endpoints

| Area | Guide | Covers |
| --- | --- | --- |
| Identity, Sites | [Sites and topology](docs/sites-and-topology.md) | who you are, what you can access, sites, nodes, devices, GeoJSON |
| Metrics | [Metrics and units](docs/metrics-and-units.md) | the metric lists for each data source, display units |
| Timeseries | [Timeseries](docs/timeseries.md) | ground truth, weather, forecasts, predictions, synthetics, summaries |
| Alerts | [Alerts](docs/alerts.md) | active alerts for a customer |
| Files | [Files](docs/files.md) | downloading and uploading customer files |
| Dashboards, Library Charts | [Dashboards and charts](docs/dashboards-and-charts.md) | dashboards and saved charts |
| Reports | [Reports](docs/reports.md) | report definitions, runs and results |
| Experiments | [Experiments](docs/experiments.md) | experiments, groups, members and experiment data |
| Customer Inputs | [Customer inputs](docs/customer-inputs.md) | categories and inputs a customer records |
| Access Management | [Access management](docs/access-management.md) | who can access a customer |
| Feedback | [`openapi.yml`](openapi.yml) (`submitFeedback`) | sending feedback to Agrology |

## OpenAPI specification

[`openapi.yml`](openapi.yml) is generated from
[agrology/public-api](https://github.com/agrology/public-api) by
`make public-spec`. Do not edit it here: change the service's spec and
regenerate.
