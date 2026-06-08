# Bug Reports

Bug reports are submitted by users from within the apps and contain a message along with diagnostic logs. You can list the bug reports created by users that share groups with you.

## List bug reports

If you want to list the bug reports, send this `GET` request:

```http
GET /analytics/bug-reports
```

Group admins only receive bug reports created by users who belong to the same groups they have access to. Admins receive every bug report.

### Request

#### Parameters

| Parameter      | Location | Type                     | Description                                                                                                | Required |
| -------------- | -------- | ------------------------ | ---------------------------------------------------------------------------------------------------------- | -------- |
| **start_date** | Query    | Milliseconds since epoch | Only return bug reports created at or after this date, in milliseconds since UNIX Epoch.                   | No       |
| **end_date**   | Query    | Milliseconds since epoch | Only return bug reports created at or before this date, in milliseconds since UNIX Epoch.                  | No       |

### Response

#### Status

| Code | Description                          |
| ---- | ------------------------------------ |
| 200  | The bug reports were returned.       |
| 401  | Unauthorized.                        |
| 403  | You don't have access to this resource. |

#### Example

```json
[
  {
    "id": "706cb23b-6e65-4863-afec-4d5fb2af17a6",
    "message": "The app crashes when I open my schedule.",
    "created_at": "2024-12-12T13:17:08.348Z",
    "created_by": {
      "id": "fc2779a8-5229-4656-86d5-4e742b61c1c5",
      "first_name": "Student",
      "last_name": "Name",
      "email": "student@stift.com.br",
      "type": "STUDENT"
    }
  }
]
```
