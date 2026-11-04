# Access management

Customer admins can see and change who has access to their customer. Access
comes at three levels: `viewer` reads data, `editor` also changes it (for
example dashboards, reports and experiments), and `admin` also manages
access. These endpoints manage customer-level access, which covers all of
the customer's sites.

## Endpoints

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/access/manage/{customerID}` | List who has access to a customer ([`listCustomerAccess`](../openapi.yml)) |
| `POST` | `/access/manage/{customerID}/{accessUser}/{accessLevel}` | Grant or change a user's access ([`createOrUpdateCustomerAccess`](../openapi.yml)) |
| `DELETE` | `/access/manage/{customerID}/{accessUser}` | Remove a user's access ([`deleteCustomerAccess`](../openapi.yml)) |

All three need a customer-level Admin grant: Admin of one of the customer's
sites is not enough.

## List who has access

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/access/manage/acme-farms
```

<!-- schema: CustomerAccessList -->
```json
[
  {
    "email": "grower@example.com",
    "customer": "acme-farms",
    "role": "editor",
    "addedBy": "owner@example.com",
    "lastModifiedBy": "owner@example.com",
    "creationDate": 1748736000,
    "lastModified": 1755205200
  }
]
```

## Grant or change access

`accessUser` is the user's email address, URL-encoded; `accessLevel` is
`viewer`, `editor` or `admin`. Send no body.

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/access/manage/acme-farms/grower%40example.com/editor
```

A `201` with an empty body means the access was granted or changed, replacing
any level the user had.

## Remove access

```bash
curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://api.agrology.ag/v2/access/manage/acme-farms/grower%40example.com
```

A `204` means it is gone, including when the user had no access to remove.

## Gotchas

- **The list shows customer-level access only.** Users with access to single
  sites are not listed, and neither are Agrology staff (`@agrology.ag`
  addresses).
- **Removing access leaves site-level access and the user's login in
  place.**
- **A new user gets a login but no invitation email.** Tell them yourself.
- **A 400 from a grant does not mean nothing changed.** Creating the login
  runs alongside the grant, so if it fails the access may still be saved:
  list the access to check.
- **Emails are stored in lower case.** Send them in lower case.
