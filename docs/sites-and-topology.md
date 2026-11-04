# Sites and topology

A customer owns sites, a site has nodes (the monitoring locations within
it), and a node carries devices, the sensors that report data. These
endpoints tell you who your credential belongs to, which customers, sites
and nodes it can reach, and what is installed where, as fleet records or as
GeoJSON for a map. The IDs they return are what every data path takes.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/whoami` | Show who the credential belongs to ([`getWhoAmI`](../openapi.yml)) |
| `GET` | `/access` | List the sites and nodes you can access ([`getUserAccess`](../openapi.yml)) |
| `GET` | `/apikey-access` | Show whether you have API key access ([`getApiKeyAccess`](../openapi.yml)) |
| `POST` | `/apikey` | Rotate your API key ([`rotateApiKey`](../openapi.yml)) |
| `GET` | `/sites` | List your sites ([`listSites`](../openapi.yml)) |
| `GET` | `/nodes` | List your nodes ([`listNodes`](../openapi.yml)) |
| `GET` | `/geojson/sites` | Get your sites as GeoJSON ([`listSitesGeoJSON`](../openapi.yml)) |
| `GET` | `/geojson/site/{siteID}` | Get one site as GeoJSON ([`getSiteGeoJSON`](../openapi.yml)) |
| `GET` | `/geojson/customer/{customerID}` | Get a customer's sites as GeoJSON ([`listSitesByCustomerGeoJSON`](../openapi.yml)) |
| `GET` | `/historical/ground-truth/{siteID}/devices` | List a site's devices ([`listDevicesBySite`](../openapi.yml)) |
| `GET` | `/historical/ground-truth/device-types` | List device types ([`listDeviceTypes`](../openapi.yml)) |

Every list covers only what your credential can access. `/access`, `/sites`,
`/nodes` and the GeoJSON endpoints are cached for 5 minutes per caller.

## Check who you are

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/whoami
```

<!-- schema: WhoAmI -->
```json
{"email": "grower@example.com"}
```

The email is lower-cased. It works with a bearer token or an API key (see
[Authentication](../README.md#authentication)).

## Find your customer, site and node IDs

`GET /access` lists every customer, site and node you hold a grant for, and
your role at each site. Start here when a path asks for a `customerID`,
`siteID` or `nodeID`:

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
        },
        "south-block": {
          "id": "south-block",
          "displayName": "South Block",
          "role": "viewer",
          "units": "metric",
          "nodes": []
        }
      }
    }
  ]
}
```

- `siteAccess[].customer` is the `customerID` for customer paths, such as
  [Alerts](alerts.md), [Files](files.md) and [Reports](reports.md).
- The keys of `sites` (and each site's `id`) are the `siteID` for site
  paths.
- `nodes[].id` is the `nodeID` for node paths.
- `role` is your role at that site, e.g. `viewer`, `editor` or `admin`.
- `units` is the site's display units, `metric` or `imperial` (see
  [Display units](../README.md#display-units)).
- `appRole` appears only for Agrology staff with debug access.

The response is cached for 5 minutes. After a grant changes, add
`?refresh=1` (any non-empty value) to rebuild it.

Device IDs come from [a site's device list](#list-a-sites-devices).

## Check and rotate your API key

`GET /apikey-access` returns your email if you have API key access, and `{}`
if you do not:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/apikey-access
```

<!-- schema: ApiKeyAccess -->
```json
{"email": "grower@example.com"}
```

`POST /apikey` replaces your key. Send no body:

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/apikey
```

<!-- schema: ApiKeyRotation -->
```json
{"status": "ok", "newKey": "Z3Jvd2VyQGV4YW1wbGUuY29tfGV4YW1wbGUta2V5"}
```

`newKey` is shown once, so store it straight away, and send it as the
`x-api-key` header. Always check `status`: a failed rotation is also a 200,
with `"status": "error"` and a `detail`.

## List your sites

`GET /sites` returns full fleet records for your active sites, or `[]` when
there are none:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/sites
```

<!-- schema: Sites -->
```json
[
  {
    "id": "north-vineyard",
    "displayName": "North Vineyard",
    "status": "active",
    "customer": "acme-farms",
    "siteClassification": "Site",
    "lat": 38.5121,
    "long": -122.4712,
    "polygon": "-122.474,38.514;-122.469,38.514;-122.469,38.51;-122.474,38.51;-122.474,38.514",
    "elevation": 52,
    "tz": "America/Los_Angeles",
    "displayUnits": "imperial",
    "sizeAcres": 40,
    "products": ["microclimate"],
    "report_time": 1793795400,
    "hwMonitoringStatus": 1,
    "envMonitoringStatus": 1
  }
]
```

Field names, including the snake_case ones such as `report_time`, are as
the API sends them. `polygon` is the boundary as text: `long,lat` pairs
split by `;`, with `|` between polygons.

## List your nodes

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://api.agrology.ag/v2/nodes
```

<!-- schema: Nodes -->
```json
[
  {
    "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
    "displayName": "Block 1 East",
    "status": "active",
    "site": "north-vineyard",
    "lat": 38.5132,
    "long": -122.4705,
    "elevation": 54,
    "crops": ["Cabernet Sauvignon"],
    "soilType": "loam",
    "activeSoilDepth": 0.6,
    "lastForecastSync": 1793793600,
    "lastWeatherSync": 1793793600
  },
  {
    "id": "8a2d5c71-0e3f-4b9a-b6d2-1c4e7f9a0b38",
    "displayName": "Block 2 West",
    "status": "active",
    "site": "north-vineyard",
    "lat": 38.5109,
    "long": -122.4731,
    "crops": null,
    "lastForecastSync": 1793793600,
    "lastWeatherSync": 1793793600
  }
]
```

`site` is the node's site ID. `crops` can be `null`.

The list leaves out inactive nodes, nodes with no latitude or longitude, and
every node at a site that is inactive, has no display name or fails to load.
An automation credential gets every active node in the fleet instead, with
none of the other filters.

## Get sites and nodes as GeoJSON

Three endpoints return the same shape, a GeoJSON FeatureCollection, for
different scopes:

| Path | Covers | `key` |
| --- | --- | --- |
| `/geojson/sites` | all your active sites | `user-access-list` |
| `/geojson/customer/{customerID}` | the customer's active sites | the customer ID |
| `/geojson/site/{siteID}` | one site | the site ID |

Each collection has a feature per site and per node at those sites.

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/geojson/site/north-vineyard
```

<!-- schema: GeoJSONCollection -->
```json
{
  "type": "FeatureCollection",
  "key": "north-vineyard",
  "description": "North Vineyard",
  "sites": ["north-vineyard"],
  "historicalURL": "https://api.agrology.ag/v2/historical/ground-truth/{site}",
  "predictionsURL": "https://api.agrology.ag/v2/predictions/{dataset}/{site}",
  "syntheticsURL": "https://api.agrology.ag/v2/synthetics/{dataset}/{site}",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "elementType": "site",
        "site": "north-vineyard",
        "name": "North Vineyard",
        "customer": "acme-farms",
        "siteClassificaiton": "Site",
        "role": "editor",
        "displayUnits": "imperial",
        "latitude": 38.5121,
        "longitude": -122.4712,
        "acreage": 40,
        "timezone": "America/Los_Angeles"
      },
      "geometry": {
        "type": "Polygon",
        "coordinates": [[
          [-122.474, 38.514], [-122.469, 38.514], [-122.469, 38.51],
          [-122.474, 38.51], [-122.474, 38.514]
        ]]
      }
    },
    {
      "type": "Feature",
      "properties": {
        "elementType": "node",
        "id": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
        "site": "north-vineyard",
        "name": "Block 1 East",
        "crops": ["Cabernet Sauvignon"],
        "deviceIDs": ["0004a30b001c2d3e"],
        "deviceIds": ["0004a30b001c2d3e"],
        "devices": {
          "0004a30b001c2d3e": {
            "model": "gV2",
            "deviceType": "gV2",
            "position": "Canopy",
            "lastMessage": 1793795400000,
            "needsMaintenance": false,
            "isMonitored": true
          }
        },
        "soilType": "loam",
        "activeSoilDepty": 0.6,
        "latitude": 38.5132,
        "longitude": -122.4705
      },
      "geometry": {"type": "Point", "coordinates": [-122.4705, 38.5132]}
    }
  ]
}
```

- `properties.elementType` says whether a feature is a `site` or a `node`.
  Site features carry the address, contact and unit fields and identify the
  site with `site`; node features carry `id`, the devices and the soil
  fields. The examples above are trimmed; `GeoJSONProperties` in
  [`openapi.yml`](../openapi.yml) lists every field.
- **Geometry is `[long, lat]`**, as in RFC 7946. A node is a `Point`; a site
  is its boundary, a `Polygon` (`[[[long, lat], ...]]`) or a
  `MultiPolygon`. `geometry` is absent for a site with no boundary or a node
  with no latitude or longitude.
- `role` on a site feature is your role at the site.
- `classification` (the customer's classification) is a field of the
  collection itself, not of a feature, and appears only on customer
  collections.
- `lastMessage` is in epoch milliseconds.

To use the URL templates, replace `{site}` with a site ID and `{dataset}`
with a dataset, then add `/{timeRange}` to `historicalURL` and
`syntheticsURL` (see [Time ranges](../README.md#time-ranges)):

```
https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/4h-
https://api.agrology.ag/v2/predictions/weather/north-vineyard
https://api.agrology.ag/v2/synthetics/microclimate/north-vineyard/24h-
```

`{dataset}` is `weather`, `microclimate` or `pest-disease` for predictions,
and `microclimate` for synthetics.

## List a site's devices

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/historical/ground-truth/north-vineyard/devices
```

<!-- schema: Devices -->
```json
[
  {
    "id": "0004a30b001c2d3e",
    "node": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
    "position": "Canopy",
    "deviceType": "gV2",
    "serial": "GV2-004512",
    "reportTime": 1793795400000,
    "reportingStatus": "ok"
  },
  {
    "id": "0004a30b001c2d4f",
    "node": "3f9c1e2a-7b4d-4c8e-9a21-5d6f0b8e4a17",
    "position": "Soil Chamber",
    "deviceType": "smp",
    "serial": "SMP-001187",
    "reportTime": 1793794800000,
    "reportingStatus": "ok"
  }
]
```

The list holds the site's active devices. `id` is the device ID (an EUI)
that the device timeseries path takes, `node` is the node it belongs to, and
`reportTime` is its last report in epoch milliseconds.

## List device types

A device type ID is what `deviceType` takes on timeseries requests, and what
`deviceType` on a device means. Each entry lists the metrics that type
reports:

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/historical/ground-truth/device-types
```

<!-- schema: DeviceTypes -->
```json
[
  {
    "id": "gV2",
    "description": "Gateway weather station",
    "metrics": [
      {"id": "airTemp", "displayName": "Air Temperature", "units": "°C"},
      {"id": "humidity", "displayName": "Humidity", "units": "%"}
    ]
  },
  {
    "id": "vwc",
    "description": "Volumetric water content probe",
    "metrics": [
      {"id": "vwc", "displayName": "Volumetric Water Content", "units": "m³/m³"}
    ]
  }
]
```

`metrics` can be `null`.

## Gotchas

- **Site and customer IDs come in two forms.** Older ones are slugs (e.g.
  `austin-research-station`), newer ones UUIDs. Treat every ID as an opaque
  string. Node IDs are UUIDs and device IDs are EUIs.
- **No sites or nodes can mean a 401, not an empty list.** `GET /nodes`,
  `GET /geojson/sites` and `GET /geojson/customer/{customerID}` answer 401
  when nothing remains to return, and a credential with no customer or site
  grants gets a 401 from `/whoami` and `/apikey-access`. `GET /sites` returns
  `[]` instead.
- **`GET /geojson/site/{siteID}` can fail with a 502** and a raw Lambda
  error when the site fails to load. A site you cannot access, including an
  unknown ID, is a 401, and an inactive site is a 400.
- **One bad site fails a whole collection.** If any site fails to load,
  `/geojson/sites` and `/geojson/customer/{customerID}` answer 400.
- **GeoJSON includes inactive nodes**, and features have no status field.
  `GET /nodes` leaves inactive nodes out.
- **`/nodes` and GeoJSON can disagree.** `/nodes` drops nodes with no
  latitude or longitude, and every node at a site with no display name or
  that fails to load, so a node in the GeoJSON can be missing from
  `/nodes`.
- **Two GeoJSON property names are typos kept for compatibility:**
  `siteClassificaiton` (the site's label, `siteClassification` in `/sites`)
  and `activeSoilDepty` (`activeSoilDepth` in `/nodes`). `deviceIDs` and
  `deviceIds` carry the same data.
- **The GeoJSON URL templates are incomplete.** `historicalURL` and
  `syntheticsURL` need `/{timeRange}` added; with `{site}` filled in alone
  they match no route.
- **The device list leaves out devices of models the API does not
  recognise**, as well as inactive devices, so a site can have devices that
  never appear here.
- **Device type IDs can repeat.** Two entries can share an `id` with
  different `metrics`, and `deviceType=<id>` selects all of them.
- **Lists can be up to 5 minutes stale.** Only `/access` has a `refresh`
  option.
- **A rotated API key can keep working for a while,** normally about 10
  minutes, so do not rely on rotation to end access at once. A failed
  rotation may already have deleted the previous key: retry at once, or with
  a bearer token, since a caller using only that key is locked out when the
  caches expire.
