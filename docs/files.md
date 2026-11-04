# Files

Each site and customer has file storage: report outputs, uploads from the
Grower Portal, and files Agrology shares with the customer, such as photos
and PDFs. The API does not stream files itself. It hands you a presigned URL,
valid for 15 minutes, that you then download from or upload to directly.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/fileserver/site/{siteID}` | Get a download URL for a site file ([`getFileBySite`](../openapi.yml)) |
| `GET` | `/fileserver/customer/{customerID}` | Get a download URL for a customer file ([`getFileByCustomer`](../openapi.yml)) |
| `PUT` | `/fileserver/customer/{customerID}` | Get an upload URL for a customer file ([`putFileByCustomer`](../openapi.yml)) |
| `GET` | `/fileserver/customer/{customerID}/shared` | List files shared with a customer ([`listFilesSharedWithCustomer`](../openapi.yml)) |

Downloads and the shared list need Viewer access to the customer (for a site
file, the customer that owns the site); uploads need Editor access.

## Download a file

Pass the file's path in storage as `file`, URL-encoded:

```bash
curl -G -H "Authorization: Bearer $ACCESS_TOKEN" \
  --data-urlencode "file=reports/0b6d1f3e/summary.pdf" \
  https://api.agrology.ag/v2/fileserver/customer/acme-farms
```

<!-- schema: PreSignedURLResponse -->
```json
{
  "url": "https://example-bucket.s3.amazonaws.com/reports/0b6d1f3e/summary.pdf?X-Amz-Expires=900&X-Amz-Signature=...",
  "file": "summary.pdf"
}
```

Then fetch `url` within 15 minutes, with no Agrology credential. Add
`redirect=true` to get a 302 straight to the file instead.

## Download a report run's files

A report run lists its output files by name in `files`. Each is stored at
`reports/<runID>/<name>` in the customer's storage, so pass that path as
`file` to `GET /fileserver/customer/{customerID}`. See
[Reports](reports.md).

## Upload a file

Ask for an upload URL, then `PUT` the file to `uploadUrl`. Send no body with
the first request. `file` must start with `uploads/customer/portal/`:

```bash
curl -X PUT -G -H "Authorization: Bearer $ACCESS_TOKEN" \
  --data-urlencode "file=uploads/customer/portal/soil-samples.csv" \
  https://api.agrology.ag/v2/fileserver/customer/acme-farms
```

<!-- schema: PreSignedURLResponse -->
```json
{
  "uploadUrl": "https://example-bucket.s3.amazonaws.com/uploads/customer/portal/1755205200/soil-samples.csv?X-Amz-Expires=900&X-Amz-Signature=...",
  "url": "https://example-bucket.s3.amazonaws.com/uploads/customer/portal/1755205200/soil-samples.csv",
  "file": "soil-samples.csv"
}
```

```bash
curl -X PUT --upload-file soil-samples.csv "$UPLOAD_URL"
```

The file is stored under a timestamp folder, as `url` shows. `url` is the
stored location, not a download link: download it with
`GET /fileserver/customer/{customerID}` and the path after the bucket name.

## List files shared with a customer

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/fileserver/customer/acme-farms/shared
```

<!-- schema: CustomerFiles -->
```json
[
  {
    "customerID": "acme-farms",
    "id": "3a7e5c1d-0f42-4b8e-a6d9-5c2b1e8f7a03",
    "description": "Station install, north block",
    "filename": "install-north.jpg",
    "folder": "photos",
    "fileType": "image",
    "status": "success",
    "siteID": "north-vineyard",
    "s3url": "s3://example-bucket/photos/install-north.jpg",
    "imageHeight": 3024,
    "imageWidth": 4032,
    "visibleToCustomer": 1,
    "createdAt": 1755205200,
    "createdBy": "field.tech@example.com",
    "lastModified": 1755205200,
    "lastModifiedBy": "field.tech@example.com"
  }
]
```

## Gotchas

- **A URL is issued for any path.** The file is not checked, so a path that
  does not exist still gets a URL, which then fails when you fetch it.
- **Upload names are decoded twice.** Encode a `%` or `+` in an upload name
  twice, or a `+` becomes a space.
- **An upload path outside `uploads/customer/portal/`, or one ending in `/`,
  is a 400.**
- **`disableRedirects` does not apply here.** With `redirect=true` the 302
  body repeats the URL as `{"code":302,"message":"redirect","location":...}`,
  with no Content-Type.
- **The shared list returns only files whose processing succeeded** and that
  are marked visible to the customer.
