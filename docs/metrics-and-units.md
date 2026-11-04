# Metrics and units

A metric is one kind of measurement, such as `airTemp` or `humidity`, with
an ID you pass in the `metrics` query parameter of a timeseries call. Each
data source has its own metric list, and `/metrics` lists every public
metric. The API always returns values in the metric's stored `units`; a
metric's `unitOptions` say how a client may convert them for display, and
`defaultUnits` say which option to show for metric and imperial users. This
guide is the reference for those conversion rules.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/metrics` | List all public metrics ([`listMetrics`](../openapi.yml)) |
| `GET` | `/historical/ground-truth/metrics` | List ground-truth metrics ([`getHistoricalGroundTruthMetrics`](../openapi.yml)) |
| `GET` | `/historical/weather/metrics` | List weather-service metrics ([`getHistoricalWeatherMetrics`](../openapi.yml)) |
| `GET` | `/predictions/weather/metrics` | List weather-forecast metrics ([`getPredictionsWeatherMetrics`](../openapi.yml)) |
| `GET` | `/predictions/microclimate/metrics` | List microclimate-prediction metrics ([`getPredictionsMicroclimateMetrics`](../openapi.yml)) |
| `GET` | `/predictions/pest-disease/metrics` | List pest and disease metrics ([`getPestDiseaseMetrics`](../openapi.yml)) |
| `GET` | `/synthetics/microclimate/metrics` | List synthetic microclimate metrics ([`getSyntheticsMicroclimateMetrics`](../openapi.yml)) |

All seven return a `MetricList`. Each source list gives the IDs for that
source's timeseries calls (see [Timeseries](timeseries.md)):

| List | Data source | Timeseries paths |
| --- | --- | --- |
| `/historical/ground-truth/metrics` | Sensor data | `/historical/ground-truth/...` |
| `/historical/weather/metrics` | Weather-service history | `/historical/weather/...` |
| `/predictions/weather/metrics` | Weather forecasts | `/predictions/weather/...` |
| `/predictions/microclimate/metrics` | Microclimate predictions | `/predictions/microclimate/...` |
| `/predictions/pest-disease/metrics` | Pest and disease predictions | `/predictions/pest-disease/...` |
| `/synthetics/microclimate/metrics` | Synthetic microclimate data | `/synthetics/microclimate/...` |

`/metrics` is the one to read for units, unit options and defaults: some
endpoints return metric IDs that their own list omits (see
[Gotchas](#gotchas)).

## List the metrics for a data source

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/historical/ground-truth/metrics
```

<!-- schema: MetricList -->
```json
[
  {
    "id": "airTemp",
    "displayName": "Air Temperature",
    "units": "°C",
    "category": "Climate",
    "sources": ["ground-truth", "synthetics", "predictions"],
    "isPublic": true,
    "unitOptions": [
      {"id": "degC", "label": "°C", "factor": 1},
      {"id": "degF", "label": "°F", "factor": 1.8, "offset": 32}
    ],
    "defaultUnits": {"metric": "degC", "imperial": "degF"}
  },
  {
    "id": "humidity",
    "displayName": "Humidity",
    "units": "%",
    "category": "Climate",
    "sources": ["ground-truth", "weather", "predictions"],
    "isPublic": true
  }
]
```

`sources` names the data sources whose list includes the metric.
`humidity` has no `unitOptions`: it is shown only in its `units`.

## Look up a metric's display units

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/metrics
```

Trimmed to one metric and three of its options:

<!-- schema: MetricList -->
```json
[
  {
    "id": "neeUMolCO2SqMDaily",
    "displayName": "NEE Net CO₂ Flux",
    "units": "µmol CO₂/m²/day",
    "category": "Soil",
    "sources": ["synthetics"],
    "isPublic": true,
    "isBeta": true,
    "unitOptions": [
      {"id": "tCPerHaDay", "label": "t C/ha/day", "factor": 1.2011e-7},
      {"id": "umolCO2PerM2S", "label": "µmol CO₂/m²/s", "factor": 0.000011574074074074073, "rateOnly": true},
      {"id": "umolCO2PerM2Day", "label": "µmol CO₂/m²/day", "factor": 1}
    ],
    "defaultUnits": {"metric": "tCPerHaDay", "imperial": "tonCPerAcDay"}
  }
]
```

Each `UnitOption` has:

- `id`: a stable identifier, safe to persist (for example, in a saved
  user preference).
- `label`: the label to show next to the converted value.
- `factor`: the multiplier from the stored unit.
- `offset`: added after `factor`. Left out when 0; only temperature uses it.
- `rateOnly`: `true` for an option that is valid only for a rate, such as
  `umolCO2PerM2S`. Left out when false.

`unitOptions` is absent when the metric is shown only in `units`.
`defaultUnits` names one option `id` for each unit system.

## Convert a value for display

For a raw value, or a mean, median, min or max:

```text
display = stored × factor + offset
```

Other statistics scale differently, and some cannot be converted at all:

| Aggregation applied | Display value | Only when |
| --- | --- | --- |
| raw, `mean`, `median`, `min`, `max` | `stored × factor + offset` | always |
| `stddevpop`, `stddevsamp` | `stored × factor` | always |
| `varpop`, `varsamp` | `stored × factor²`, label squared | always |
| `geomean` | `stored × factor` | `offset` is 0 |
| `sum`, `runningTotal` | `stored × factor` | `offset` is 0 and `rateOnly` is not set |

Where no row applies, show the stored value with the metric's `units`. A
`sum` or `runningTotal` of a per-day metric is a total for the period, so
drop `/day` from the label.

Pick the row by the aggregation the server applied, which is not always the
`statistic` you sent:

- Without a usable `resolution`, values are raw, unless `runningTotal`
  accumulated them.
- A `runningTotal` sent as the query `statistic` never accumulates at
  `responseVersion` 2.0, on experiment and summary responses, or at 2.2 on weather and microclimate predictions
  and synthetics below calendar-month resolution. There the values are
  bucket sums with `resolution` and raw without it (see
  [`statistic`](timeseries.md#bucket-the-data)).
- At 2.0, `resolution=2592000` gives raw values for every statistic.
- An experiment column combines two aggregations. Use the raw-value rule when
  both are mean, median, min or max (ignoring case); otherwise show
  the stored value. A `runningTotal` set on the experiment metric does
  accumulate (see [Experiments](experiments.md)).

### Temperature

`airTemp` is stored in °C. Its `degF` option has `factor` 1.8 and `offset`
32:

- A raw or mean value of 14.2 °C shows as 14.2 × 1.8 + 32 = 57.56 °F.
- A standard deviation of 2.0 °C shows as 2.0 × 1.8 = 3.6 °F: no offset.
- A variance of 4.0 °C² shows as 4.0 × 1.8² = 12.96 °F².
- A sum or running total of temperatures stays in °C, because `degF` has an
  offset.

### Net ecosystem exchange (NEE)

`neeUMolCO2SqMDaily` is stored in µmol CO₂/m²/day. Its `tCPerHaDay` option
has `factor` 1.2011e-7, so a daily value of −200,000 µmol CO₂/m²/day shows
as −200,000 × 1.2011e-7 ≈ −0.024 t C/ha/day.

- Negative NEE is uptake in every unit: every NEE factor is positive, so
  conversion keeps the sign.
- A mean of daily values converts the same way. The `umolCO2PerM2S` option
  gives the daily mean rate: −200,000 µmol CO₂/m²/day ≈ −2.31 µmol CO₂/m²/s.
- A `sum` or `runningTotal` of daily values converts by `tCPerHaDay` too,
  and is a total for the period, so label it `t C/ha`. Do not offer
  `umolCO2PerM2S` for it: that option is `rateOnly`.
- The day and night portions, `neeUMolCO2SqMDay` and `neeUMolCO2SqMNight`,
  are stored in µmol CO₂/m² (an amount for the period, not a rate). Their
  options, such as `tCPerHa` and `gCPerM2`, have no per-second option, and
  their defaults are `tCPerHa` (metric) and `tonCPerAc` (imperial).

## Choose default units for a user

The unit system comes from the site: `units` on each site in
[`GET /access`](sites-and-topology.md), `metric` or `imperial`.

<!-- schema: AccessSite -->
```json
{"id": "north-vineyard", "displayName": "North Vineyard", "role": "editor", "units": "imperial"}
```

For each metric you display:

1. If the user has chosen a unit option, use that `id`.
2. Otherwise take `defaultUnits.imperial` or `defaultUnits.metric`, for the
   site's `units`, and find the option with that `id` in `unitOptions`.
3. If the metric has no `unitOptions`, or the option does not apply to the
   aggregation (see the table above), show the stored value with `units`.

At North Vineyard, `airTemp` shows in `degF` and `neeUMolCO2SqMDaily` in
`tonCPerAcDay`.

## Gotchas

- **The API never converts.** Timeseries values are always in the metric's
  stored `units`, whatever the site's display units.
- **Timeseries responses carry no unit options.** The `metrics` map in a 2.2
  response has `units` but no `unitOptions` or `defaultUnits`; look them up
  in `/metrics`.
- **The weather lists are not exact.** `/historical/weather/metrics` and
  `/predictions/weather/metrics` are the same list. Both weather sources
  return `airTemp`, which the list omits, and historical weather never
  returns `vpd`, which it lists.
- **`/predictions/pest-disease/metrics` is always `[]`.** Pest and disease
  responses use raw metric keys that start with `pd`, and those metrics are
  in no list.
- **Unknown metric IDs widen a query.** On ground truth, predictions and
  synthetics, unknown IDs in `metrics` are ignored, and if none is known
  every metric comes back. On historical weather, if none is known the
  samples have no metrics.
