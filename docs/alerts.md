# Alerts

Alerts are notices Agrology raises for a customer's sites: fires detected
nearby, extreme temperature warnings, and Agrology notifications. The API
returns the customer's active, unmuted alerts for the sites you can access.
The response is shaped for an Apple push notification, so the list sits
beside an `aps` block.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/alerts/customer/{customerID}` | List a customer's active alerts ([`listAlertsByCustomer`](../openapi.yml)) |

`customerID` comes from [`GET /access`](sites-and-topology.md)
(`siteAccess[].customer`). You need Viewer access to the customer.

## List a customer's alerts

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/alerts/customer/acme-farms
```

<!-- schema: AlertList -->
```json
{
  "aps": {"alert": "Alerts for Acme Farms"},
  "alerts": [
    {
      "id": "6f1c2a9e-1b7d-4c3e-9a51-2f0e8d4b7c10",
      "insight": "fire",
      "startTime": "2026-08-14T21:05:00Z",
      "updateTime": "2026-08-14T21:05:00Z",
      "customer": "acme-farms",
      "site": "north-vineyard",
      "sites": ["north-vineyard", "south-block"],
      "eventValues": {
        "origin": "Agrology",
        "title": "Fire in Area",
        "headline": "A fire has been detected 4 miles north of North Vineyard.",
        "location": {"type": "Point", "coordinates": [38.512, -122.471]}
      }
    }
  ]
}
```

With no alerts the body is `{"aps":{}}`: both `alerts` and `aps.alert` are
left out.

<!-- schema: AlertList -->
```json
{"aps": {}}
```

## Gotchas

- **`location.coordinates` is `[latitude, longitude]`**, the opposite order to
  GeoJSON, which this API's GeoJSON endpoints use.
- **An alert with no position has `"location": {}`.** An alert at exactly 0°
  latitude or longitude also loses its location. An alert for an area can
  list it in `eventValues.points` instead, as the alert source sent the
  points; their order is not converted.
- **Several fields are never filled today:** `endTime`, `severity`,
  `certainty`, `urgency`, `distance` and `direction` are always left out, and
  `updateTime` is the same as `startTime`.
- **`sites` lists every site the alert affects;** `site` is the main one.
- **`aps.alert` falls back to the customer ID** when the customer's name
  cannot be read.
