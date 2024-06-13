# Agrology Public API v2

Agrology Public API supports accessing configuration data about sites like GPS, Node Names, etc., as well as historical telemetry data and prediction data.

## Base URL

The Public API base URL is <https://api.agrology.ag/v2>

### Authentication Security

Clients must attach either a Bearer Token or an API key to all API requests. Any API request without a Bearer Token or API Key will be rejected with a `401 Unauthorized` error.

#### Bearer Token

Attach an Authorization HTTP Header to API requests in the following format:

```bash
Authorization: Bearer $ACCESS_TOKEN
```

To get a Bearer Token, go to <https://grower.agrology.ag/>, login, and then click on your username in the application footer.

Bearer Tokens expire 1 hour after being issued, and you will need to return to the Grower's Portal to get a new one.

#### API Key

Attach an API Key HTTP Header to API requests in the following format:

```bash
x-api-key: $API_KEY
```

Contact Agrology staff to receive an API Key. Soon, the Grower's Portal will alllow users to rotate API Keys.

## User's Access List

A user can make a request to <https://api.agrology.ag/v2/access> with their Bearer Token or API Key to return a list of entities to which they have access. The output of the ACL endpoint is as follows:

```json
{
  "siteAccess": [
    {
      "customer": "agrology",
      "displayName": "Agrology Internal Use",
      "sites": {
        "abc391cd-b8cd-4e47-b7ab-8e91f9ebbe6e": {
          "id": "abc391cd-b8cd-4e47-b7ab-8e91f9ebbe6e",
          "displayName": "Device Data Testing",
          "role": "admin",
          "blocks": [
            {
              "id": "ff9d2c1b-7bf2-4a65-9944-a20f3e719810",
              "displayName": "Home"
            }
          ],
          "nodes": [
            {
              "id": "dc59edcd-1787-488a-b6f1-5632eb132667",
              "displayName": "Control",
              "block": "ff9d2c1b-7bf2-4a65-9944-a20f3e719810"
            },
            {
              "id": "9f04f006-d3ff-489f-b234-b77510ae13c0",
              "displayName": "TL Test Bench"
            },
            {
              "id": "2d58b570-a3b5-47fb-867f-5152ba2be73c",
              "displayName": "Test Bench",
              "block": "ff9d2c1b-7bf2-4a65-9944-a20f3e719810"
            }
          ]
        }
      }
    }
  ]
}
```

Note that older Customers and Sites use a slug-style ID, while newer ones use UUIDs. All Blocks and Nodes are identified by UUIDs. This documentation does not describe the relationship between Customers, Sites, Nodes and Blocks. Contact Agrology for additional information.

## Accessing Data via GeoJSON

Customer and Site Data in GeoJSON format is available in the following <b>scopes</b>:

* All Sites accessible to the User: <https://api.agrology.ag/v2/geojson/sites>

* All Sites under a particular Customer accessible to the User: <https://api.agrology.ag/v2/geojson/customer/{customerID}>

* A single Site: <https://api.agrology.ag/v2/geojson/site/{siteID}>

Example URLs:

* <https://api.agrology.ag/v2/geojson/customer/agrology>

* <https://api.agrology.ag/v2/geojson/site/austin-research-station>

### GeoJSON Format

```json
{
  "key": "austin-research-station",
  "description": "Austin Research Station",
  "historicalURL": "https://api.agrology.ag/v2/historical/ground-truth/{site}",
  "predictionsURL": "https://api.agrology.ag/v2/predictions/{dataset}/{site}",
  "syntheticsURL": "https://api.agrology.ag/v2/synthetics/{dataset}/{site}",
  "sites": [
    "austin-research-station"
  ],
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "elementType": "site",
        "siteClassificaiton": "lab",
        "name": "Austin Research Station",
        "customer": "agrology",
        "site": "austin-research-station",
        "role": "admin",
        "country": "US",
        "displayUnits": "imperial",
        "timezone": "America/Los_Angeles"
      },
      "geometry": {}
    },
    {
      "type": "Feature",
      "properties": {
        "elementType": "node",
        "name": "CLAY",
        "deviceIDs": [
          "70B3D57BA000163E",
          "70B3D57BA00015EA",
          "383737326F308105"
        ],
        "devices": {
          "383737326F308105": {
            "make": "agrology",
            "model": "v1_ground",
            "lastMessage": 1673603380000,
            "lastPayload": "10274a03a1b00105130b1c053505de01c6034d00000000",
            "isMonitored": true
          },
          "70B3D57BA00015EA": {
            "make": "decentlab",
            "model": "trs12",
            "lastMessage": 1673603779000,
            "lastPayload": "0215ea000355d8805500030b63",
            "isMonitored": true
          },
          "70B3D57BA000163E": {
            "make": "decentlab",
            "model": "trs21",
            "lastMessage": 1673604549000,
            "lastPayload": "02163e00034dab80560b14",
            "isMonitored": true
          }
        },
        "elevationMeters": 53,
        "site": "austin-research-station",
        "crops": [
          ""
        ],
        "id": "516dbc83-8ee1-4f18-af23-0859259c81be"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [
          -122.602875,
          45.556985
        ]
      }
    }
  ]
}
```


## Time Ranges

A Time Range consists of a Start Time and optional End Time, separated by a hyphen, e.g., `{startTime}-{endTime}`. If the End Time is omitted, it is assumed to be `$now`. When the End Time is omitted, a the hyphen separator is still required.

### Start / End Time Formats:

Start Time and End Time can be in any of the following formats. Start Time and End Time may be in different formats.

### Relative Time 

A relative time from now in the format of `<value:integer><unit:char>`. Examples: `6d`, `20m`, `3600s`, `36h`

Supported `Relative Time` units are `s`, `m` (minutes), `h`, `d`, `w`

Complex Relative Times (e.g., 2h5m) are not supported.

### Epoch Time Stamp in Seconds or Milliseconds

* An epoch timestamp in seconds, such as `1618203722`;  must be exactly 10 numeric digits long

An epoch timestamp is a 10-digit number representing the number of seconds since January 1, 1970 at midnight, and is a common numerical format for transmitting timestamps. You can use <https://epochconverter.com> or some other tool to translate between epoch and a human-readable stamp.

* An epoch timestamp in milliseconds, such as `1618203722000`; must be exactly 13 numeric digits long

### Human-Readable Times with Minute or Second accuracy

* A human-constructible timestamp in the format of `yyyyMMddhhmm`; must be exactly 12 numeric digits long and begin with `20`. Example: `202108010000` (1 Aug, 2021 at Midnight UTC)

* A human-constructible timestamp in the format of `yyyyMMddhhmmss`; must be exactly 14 numeric digits long and begin with `20`. Example: `20210801052530` (1 Aug, 2021 at 05:25:30 UTC)

### Time Range Examples

* Epoch Start + End: `1618203722-1618290149`

* Epoch Start to Now: `1618203722-`

* 4 Hours ago to Now: `4h-`

* 2h window from 6h to 4h ago: `6h-4h`

## Data Filtering

By default, the API response will include all Devices and Metrics that are applicable to the provided Site, optional Node/Block/Device, and Time Range. You can further filter the results by providing optional URL Parameters to filter by Device Type and/or Metric Name.

### Filtering by Device Type

Add a query parameter for `deviceType` to the URL that provides a comma-separated list of Device Types to include. The provided values must match `id` values in the Device Type list available at <https://api.agrology.ag/v2/historical/ground-truth/device-types>

Example Parameters:

* `?deviceType=vwc,gV1`
* `?deviceType=gV1,gV2`
* `?deviceType=co2`

### Filtering by Metric Names

Add a query parameter for `metrics` to the URL that provides a comma-separated list of Metric Names to include. The provided values must match `id` values in the list available at <https://api.agrology.ag/v2/historical/ground-truth/metrics>.

Example Parameters:

* `?metrics=airTemp`
* `?metrics=airTemp,humidity`
* `?deviceType=co2&metrics=co2Concentration`

## Data Ordering

Data is grouped by Node and ordered by Timestamp in chronological order. Nodes can be presented in any order in results, and Node ordering is not guaranteed across requests.

## Accessing Historical Ground-Truth Data

### Retrieve List of Metrics in Historical Ground-Truth Data

The list of possible metrics returned in Historical Ground-Truth data is available at: <https://api.agrology.ag/v2/historical/ground-truth/metrics>

**NOTE:** The example output below is intentionally truncated.

```json
[
  {
    "id": "airTemp",
    "displayName": "Air Temperature",
    "units": "˚C"
  },
  {
    "id": "humidity",
    "displayName": "Humidity",
    "units": "%"
  },
  {
    "id": "totalVOCs",
    "displayName": "Total VOCs"
  }
]
```

### Retrieve Historical Ground-Truth Data

Historical Ground-Truth data can be accessed by Site, Node, Block or Device. A Site ID and Time Range is required for all scopes.

By Site: https://api.agrology.ag/v2/historical/ground-truth/{siteID}/{timeRange}

By Node: https://api.agrology.ag/v2/historical/ground-truth/{siteID}/node/{nodeID}/{timeRange}

By Block: https://api.agrology.ag/v2/historical/ground-truth/{siteID}/block/{blockID}/{timeRange}

By Device: https://api.agrology.ag/v2/historical/ground-truth/{siteID}/device/{deviceID}/{timeRange}

### Time Series Data Format

All time Series Data (Historical Ground-Truth, Predictions and Synthetics) are in the following format:

```json
{
  "status": "ok",
  "request": {
    "requestTime": 1718293623,
    "source": "historical/ground-truth",
    "siteId": "austin-research-station",
    "format": "json",
    "responseVersion": "2.1"
  },
  "sites": {
    "austin-research-station": {
      "id": "austin-research-station",
      "name": "Austin Research Station"
    }
  },
  "devices": {
    "70B3D57BA000163B": {
      "deviceType": "smp",
      "id": "70B3D57BA000163B"
    },
    "383737326F308105": {
      "deviceType": "gV1",
      "id": "383737326F308105",
      "position": "Soil Chamber"
    },
    "70B3D57BA000163C": {
      "deviceType": "smp",
      "id": "70B3D57BA000163C",
    }
  },
  "nodes": {
    "f5053b4e-3bd3-4acb-956d-9223a038bf5a": {
      "name": "SILT",
      "id": "f5053b4e-3bd3-4acb-956d-9223a038bf5a",
      "siteId": "austin-research-station",
      "samples": [
        {
          "ts": 1672531458,
          "dev": "70B3D57BA000163B",
          "d": {
            "battery": 2.9,
            "soilTemperature": 7.4,
            "waterPotential": -1958.2
          }
        }
      ]
    },
    "516dbc83-8ee1-4f18-af23-0859259c81be": {
      "name": "CLAY",
      "id": "516dbc83-8ee1-4f18-af23-0859259c81be",
      "samples": [
        {
          "ts": 1672531628,
          "dev": "383737326F308105",
          "d": {
            "battery": 3,
            "barometricPressure": 1010.7,
            "airTemp": 7.9,
            "humidity": 93,
            "totalVOCs": 1.255,
            "soilMoisture": 1335,
            "soilTension": 1500,
            "soilConductivity": 452,
            "soilTemperature": 7.2,
            "vpd": 0.075,
            "dewPoint": 6.88
          }
        }
      ]
    },
    "bc11540e-a33c-40c6-8df7-78d3c320dbdf": {
      "name": "SAND",
      "id": "bc11540e-a33c-40c6-8df7-78d3c320dbdf",
      "samples": [
        {
          "ts": 1672531909,
          "dev": "70B3D57BA000163C",
          "d": {
            "battery": 2.9,
            "soilTemperature": 7.4,
            "waterPotential": -10.9
          }
        }
      ]
    }
  }
}
```

## Accessing Historical Weather Service Data

Historical Weather Service data is provided by <https://tomorrow.io>.

### Retrieve List of Metrics in Historical Weather Service Data

The list of possible metrics returned in Historical Weather Service data is available at: <https://api.agrology.ag/v2/historical/weather/metrics>

**NOTE:** The example output below is intentionally truncated.

```json
[
  {
    "id": "airTemp",
    "displayName": "Air Temperature",
    "units": "˚C"
  },
  {
    "id": "humidity",
    "displayName": "Humidity",
    "units": "%"
  },
  {
    "id": "epaIndex",
    "displayName": "EPA Index"
  }
]
```

### Retrieve Historical Weather Service Data

Historical Weather Service data can be accessed by Site, Node, Block or Device. A Site ID and Time Range is required for all scopes.

By Site: https://api.agrology.ag/v2/historical/weather/{siteID}/{timeRange}

By Node: https://api.agrology.ag/v2/historical/weather/{siteID}/node/{nodeID}/{timeRange}

By Block: https://api.agrology.ag/v2/historical/weather/{siteID}/block/{blockID}/{timeRange}

### Historical Weather Service Data Format

Historical Weather Service data is in the standard Time Series format (see [above](#time-series-data-format)).

## Accessing Synthetic Microclimate Data

Synthetic Microclimate data is historical data produced by Agrology by applying ML Modeling to Historical Ground-Truth data to synthesize new metrics for the same historical time period.

### Retrieve List of Metrics in Synthetic Microclimate Data

The list of possible metrics returned in Synthetic Microclimate data is available at: <https://api.agrology.ag/v2/synthetics/microclimate/metrics>

```json
[
  {
    "id": "vwc",
    "description": "Volumetric Water Content"
  }
]
```

### Retrieve Synthetic Microclimate Data

Historical Weather Service data can be accessed by Site, Node, Block or Device. A Site ID and Time Range is required for all scopes.

By Site: https://api.agrology.ag/v2/synthetics/microclimate/{siteID}/{timeRange}

By Node: https://api.agrology.ag/v2/synthetics/microclimate/{siteID}/node/{nodeID}/{timeRange}

By Block: https://api.agrology.ag/v2/synthetics/microclimate/{siteID}/block/{blockID}/{timeRange}


### Synthetic Microclimate Data Format

Synthetic Microclimate data is in the standard Time Series format (see [above](#time-series-data-format)).

## Accessing Microclimate Predictions

Microclimate Predictions are generated by Agrology using a Statistical Model with inputs from Weather Service Forecasts and Agrology Historical Ground-Truth data for the microclimate.

### Retrieve List of Metrics in Microclimate Predictions

The list of possible metrics returned in Microclimate Predictions data is available at: <https://api.agrology.ag/v2/predictions/microclimate/metrics>

**NOTE:** The example output below is intentionally truncated.

```json
[
  {
    "id": "airTemp",
    "displayName": "Air Temperature",
    "units": "˚C"
  },
  {
    "id": "humidity",
    "displayName": "Humidity",
    "units": "%"
  },
  {
    "id": "vpd",
    "displayName": "Vapor Pressure Deficit"
  }
]
```

### Retrieve Microclimate Predictions Data

Microclimate Predictions data can be accessed by Site, Node, Block or Device. Predictions are available for up to 4 days in advance. A Site ID is required for all scopes. The Time Range can be omitted and will assume a time of Now and return all future predictions.

By Site: https://api.agrology.ag/v2/predictions/microclimate/{siteID}/{timeRange}

By Site (no time range): https://api.agrology.ag/v2/predictions/microclimate/{siteID}

By Node: https://api.agrology.ag/v2/predictions/microclimate/{siteID}/node/{nodeID}/{timeRange}

By Node (no time range): https://api.agrology.ag/v2/predictions/microclimate/{siteID}/node/{nodeID}

By Block: https://api.agrology.ag/v2/predictions/microclimate/{siteID}/block/{blockID}/{timeRange}

By Block (no time range): https://api.agrology.ag/v2/predictions/microclimate/{siteID}/block/{blockID}

### Microclimate Predictions Data Format

Microclimate Predictions data is in the standard Time Series format (see [above](#time-series-data-format)).

## Accessing Weather Service Predictions

Weather Service Predictions are supplied by <https://tomorrow.io>.

### Retrieve List of Metrics in Weather Service Predictions

The list of possible metrics returned in Weather Service Predictions data is available at: <https://api.agrology.ag/v2/predictions/weather/metrics>

**NOTE:** The example output below is intentionally truncated.

```json
[
  {
    "id": "airTemp",
    "displayName": "Air Temperature",
    "units": "˚C"
  },
  {
    "id": "humidity",
    "displayName": "Humidity",
    "units": "%"
  },
  {
    "id": "windSpeed",
    "displayName": "Wind Speed",
    "units": "m/s"
  }
]
```

### Retrieve Weather Service Predictions Data

Weather Service Predictions data can be accessed by Site, Node, Block or Device. Predictions are available for up to 4 days in advance. A Site ID is required for all scopes. The Time Range can be omitted and will assume a time of Now and return all future predictions.

By Site: https://api.agrology.ag/v2/predictions/weather/{siteID}/{timeRange}

By Site (no time range): https://api.agrology.ag/v2/predictions/weather/{siteID}

By Node: https://api.agrology.ag/v2/predictions/weather/{siteID}/node/{nodeID}/{timeRange}

By Node (no time range): https://api.agrology.ag/v2/predictions/weather/{siteID}/node/{nodeID}

By Block: https://api.agrology.ag/v2/predictions/weather/{siteID}/block/{blockID}/{timeRange}

By Block (no time range): https://api.agrology.ag/v2/predictions/weather/{siteID}/block/{blockID}

### Weather Service Predictions Data Format

Weather Service Predictions data is in the standard Time Series format (see [above](#time-series-data-format)).

## Accessing Alerts for a Customer

Alerts are generated for customers when configured thresholds are met (e.g., when the ground-truth temperature is extremely hot or cold). Alerts are aggregated to a customer-specific endpoint.

The URL for alerts is <https://api.agrology.ag/v2/alerts/customer/{customerID}>

### Example Alert Output

```json
{
  "aps": {
    "alert": "Alerts for Duckhorn Vineyards"
  },
  "alerts": [
    {
      "id": "6bd40bf2-4e5c-4b54-bdae-921a8738653b",
      "insight": "temperature",
      "startTime": "2023-03-07T13:01:41Z",
      "updateTime": "2023-03-07T13:01:41Z",
      "customer": "example-customer",
      "sites": [
        "example-site-1",
        "example-site-2"
      ],
      "eventValues": {
        "origin": "Agrology",
        "title": "Extreme Cold Temperature Warning",
        "headline": "Temperature is forecast to drop below 34 °F starting at 5am PST today at Example Vineyard1 and Example Vineyard 2",
        "location": {},
        "points": [
          [
            39.056803,
            -123.433543
          ],
          [
            39.05481,
            -123.435506
          ]
        ]
      }
    }
  ]
}
```

## OpenAPI Specification

**Note:** The Agrology API's OpenAPI specification is under development. It may include endpoints that are not documented above. Response Schemas currently lack data typing and are subject to change.

The OpenAPI Specification is available [in this project at openapi.yml](./openapi.yml)
