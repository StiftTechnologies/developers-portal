# Bookings

Bookings is a module designed to allow for students and teachers to connect through in-person or virtual calls on Stift's app.

## Availability Blocks

Availability blocks are used to block time slots during which a teacher is not available and thus cannot be booked.

### Get Availability Blocks

To get a list of all the availability blocks, send this `GET` request:

```http
GET /bookings/availability-blocks
```

#### Request

##### Parameters

No parameters are needed for this request.

#### Response

##### Status

| Code | Description                                        |
| ---- | -------------------------------------------------- |
| 200  | The availability blocks were fetched successfully. |
| 401  | Unauthorized.                                      |

#### Example

```json
[
  {
    "id": "85f23241-78c3-4b1c-8238-1804a82c0ee7",
    "start_date": "2025-12-13T11:00:00.000Z",
    "end_date": "2025-12-13T23:00:00.000Z",
    "reason": "Reason given by the teacher/block creator",
    "created_at": "2024-12-12T19:46:08.623Z",
    "teacher": {
      "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
      "first_name": "Teacher's first name",
      "last_name": "Teacher's last name",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png"
    },
    "created_by": {
      "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
      "first_name": "Block creator's first name",
      "last_name": "Block creator's last name",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png"
    }
  }
]
```
