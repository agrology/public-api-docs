# Customer inputs

Customer inputs record what happened in the field: plantings, harvests,
applications, bud break, wildfire events and so on. Each input has a
category, and the category decides which keys its `data` may hold.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/customer-inputs/categories` | List the input categories ([`listCustomerInputCategories`](../openapi.yml)) |
| `GET` | `/customer-inputs/customer/{customerID}` | List a customer's inputs ([`listCustomerInputsByCustomerID`](../openapi.yml)) |
| `GET` | `/customer-inputs/customer/{customerID}/category/{category}` | List a customer's inputs in a category ([`listCustomerInputsByCustomerIDAndCategory`](../openapi.yml)) |
| `POST` | `/customer-inputs/customer/{customerID}` | Record a customer input ([`createCustomerInput`](../openapi.yml)) |
| `PUT` | `/customer-inputs/{customerID}/{id}` | Update a customer input ([`updateCustomerInput`](../openapi.yml)) |
| `DELETE` | `/customer-inputs/{customerID}/{id}` | Delete a customer input ([`deleteCustomerInput`](../openapi.yml)) |

Any logged-in caller can read the categories. Listing inputs needs Viewer
access to the customer; recording, changing and deleting need Editor access.

## Find a category's keys

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/customer-inputs/categories
```

<!-- schema: CustomerInputCategories -->
```json
[
  {
    "name": "Harvest",
    "requiredValues": [
      {"key": "date", "description": "Date", "type": "date"},
      {"key": "location", "description": "Location", "type": "string"}
    ],
    "optionalValues": [
      {"key": "crop", "description": "Crop", "type": "string"},
      {"key": "yield", "description": "Yield", "type": "string"},
      {"key": "quality", "description": "Quality", "type": "string"}
    ]
  }
]
```

Each value's `type` says how to send it, always as a string or number:

| `type` | Send |
| --- | --- |
| `string` | any text |
| `number` | a number |
| `date` | `"YYYY-MM-DD"` |
| `dateRange` | `"YYYY-MM-DD to YYYY-MM-DD"` |
| `gps` | `"latitude, longitude"` |
| `enum.string:A,B,C` | one of the listed values |
| `s3file` | the HTTPS URL of a file you uploaded under `uploads/customer/portal/` (see [Files](files.md)) |

## Record an input

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @input.json \
  https://api.agrology.ag/v2/customer-inputs/customer/acme-farms
```

<!-- schema: CustomerInputSubmission -->
```json
{
  "category": "Harvest",
  "data": {
    "date": "2026-09-18",
    "location": "North Vineyard, block 3",
    "crop": "Cabernet Sauvignon",
    "yield": "4.2 t/acre"
  }
}
```

<!-- schema: Created -->
```json
{"status": "ok", "id": "5e2a9c7b-3d41-4f86-a0b2-7c1e9d5f3a18"}
```

## List inputs

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/customer-inputs/customer/acme-farms
```

<!-- schema: CustomerInputs -->
```json
[
  {
    "customerID": "acme-farms",
    "id": "5e2a9c7b-3d41-4f86-a0b2-7c1e9d5f3a18",
    "category": "Harvest",
    "data": {
      "date": "2026-09-18",
      "location": "North Vineyard, block 3",
      "crop": "Cabernet Sauvignon",
      "yield": "4.2 t/acre"
    },
    "createdAt": 1789743600,
    "createdBy": "grower@example.com",
    "lastModified": 1789743600,
    "lastModifiedBy": "grower@example.com"
  }
]
```

## Change or delete an input

`PUT` takes the same body as create. Send `category` even if it is
unchanged, and send the whole `data` object: it replaces the stored one. To
remove `data` entirely, send `"unset": ["data"]` without `data`. Both `PUT`
and `DELETE` answer `204` with no body. A delete is permanent.

## Gotchas

- **The category path does not filter.**
  `/customer-inputs/customer/{customerID}/category/{category}` returns every
  input for the customer today, so filter on `category` yourself.
- **Required keys are not enforced.** A key outside the category's
  `requiredValues` and `optionalValues` is a 400, but leaving out a required
  key is accepted.
- **Only `s3file` values are checked.** Dates, numbers and enum values are
  stored as sent.
- **Keep values to strings and numbers.** An array or object is accepted,
  but a later update to that key can fail with a 502.
- **An ID that does not exist is a 400** with `item does not exist`, not a
  404.
