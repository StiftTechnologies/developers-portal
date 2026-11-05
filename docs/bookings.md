# Bookings

Bookings is a module that connects students and teachers through in-person or virtual calls on Stift's app.

This page shows how a group admin uses an API key to book a meeting for a student. The API key must belong to a GROUP_ADMIN user. You create and configure booking rooms in the Portal. This page does not cover that configuration.

## Concepts

- Booking room: a space where students meet teachers. Each room has a discipline, a session length (`minutes_per_session`) and a list of teachers with weekly availability.
- Room type: a room is `SCHEDULE` or `QUEUE`. Students book a time in a `SCHEDULE` room. Students wait in line in a `QUEUE` room. You can book meetings only in `SCHEDULE` rooms.
- Scheduled meeting: one booking of one student with one teacher at a fixed time. This page also calls it a meeting.
- Credit: the right to book one meeting. Each booking uses one credit of the student. Credits come from the room configuration of the student's group, or from single credits that you give to a student in the Portal.
- Physical room: a real room that the school assigns to in-person meetings. If the booking room has physical rooms turned on, each in-person meeting gets a physical room.

### Access rules

The API checks these rules on each request:

- The booking room must be shared with at least one group of the API key's user.
- The student must be in one of those groups, and the student must not be blocked.
- The teacher must be in one of those groups, and the teacher must have availability in the room.

If the room is not shared with your groups, or none of its teachers is in your groups, the API returns 404 `Booking room not found`.

### Meeting status

The `status` field of a meeting has one of these values:

| Status          | Description                                                       |
| --------------- | ----------------------------------------------------------------- |
| **PENDING**     | The meeting is in the future.                                     |
| **IN_PROGRESS** | The meeting started and did not end.                              |
| **PRESENT**     | The meeting started and ended.                                    |
| **ABSENT**      | The meeting time passed and the meeting did not start.            |
| **CANCELLED**   | Someone cancelled the meeting.                                    |

## Book a meeting for a student

Do these steps to book a meeting:

1. Find the booking room with [Get booking rooms](#get-booking-rooms).
2. Make sure that the student has credits with [Get student credits](#get-student-credits).
3. Get the free times with [Get available schedules](#get-available-schedules).
4. Pick a time and a teacher with the status `AVAILABLE` from that response.
5. Book the meeting with [Create a booking](#create-a-booking).

Use a time and a teacher from step 3. The create request does not compare your time with the teacher's weekly availability. It checks only availability blocks and other meetings of the teacher.

## Get booking rooms

To get the booking rooms of your schools, send this `GET` request:

```http
GET /bookings
```

This list contains every room that belongs to your schools. If a room in this list is not shared with your groups, you cannot book in it. See [Access rules](#access-rules).

### Request

#### Parameters

| Parameter         | Location | Type    | Description                                                                 | Required |
| ----------------- | -------- | ------- | --------------------------------------------------------------------------- | -------- |
| **show_teachers** | Query    | boolean | If `true`, each room includes a `teachers` list. The default is `false`.     | No       |

### Response

#### Status

| Code | Description                              |
| ---- | ---------------------------------------- |
| 200  | The booking rooms were fetched.          |
| 401  | Unauthorized.                            |
| 403  | The user type of the API key cannot use this request. |

#### Example

```json
[
  {
    "id": "6f1c2d3e-4b5a-4c6d-8e7f-9a0b1c2d3e4f",
    "title": "Mathematics tutoring",
    "minutes_per_session": 30,
    "type": "SCHEDULE",
    "discipline": {
      "id": 1,
      "name": "Mathematics",
      "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_math.png"
    },
    "teachers": [
      {
        "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
        "first_name": "Teacher's first name",
        "last_name": "Teacher's last name",
        "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png"
      }
    ]
  }
]
```

Rooms of type `QUEUE` also include `queue_length` and `estimated_wait_time` (in minutes).

## Get student credits

To get the credits of a student in a booking room, send this `GET` request:

```http
GET /bookings/{booking_room_id}/credits
```

### Request

#### Parameters

| Parameter           | Location | Type   | Description                     | Required |
| ------------------- | -------- | ------ | ------------------------------- | -------- |
| **booking_room_id** | Path     | string | The id of the booking room.     | Yes      |
| **student_id**      | Query    | string | The id of the student.          | Yes      |

### Response

| Field         | Type    | Description                                                         |
| ------------- | ------- | ------------------------------------------------------------------- |
| **used**      | integer | The number of credits that the student used.                        |
| **available** | integer | The number of credits that the student can use now. `-1` means no limit. |

#### Status

| Code | Description                                                   |
| ---- | ------------------------------------------------------------- |
| 200  | The credits were fetched.                                     |
| 401  | Unauthorized.                                                 |
| 403  | The user type of the API key cannot use this request. |

#### Example

```json
{
  "used": 2,
  "available": 2
}
```

If `available` is `0`, the create request returns 403 `INSUFFICIENT_CREDITS`. To give the student more credits, change the room configuration of the student's group in the Portal, or give the student a single credit.

## Get available schedules

To get the times that a student can book in a room, send this `GET` request:

```http
GET /bookings/{booking_room_id}/schedules
```

The response covers the room's booking window, for example the next seven days. For group admins, the room's minimum scheduling notice does not apply. The response can contain times that start soon.

### Request

#### Parameters

| Parameter               | Location | Type   | Description                                                                                       | Required |
| ----------------------- | -------- | ------ | ------------------------------------------------------------------------------------------------- | -------- |
| **booking_room_id**     | Path     | string | The id of the booking room.                                                                       | Yes      |
| **studentId**           | Query    | string | The id of the student. If the student has a meeting in another room at a time, the response hides that time. | Yes      |
| **rescheduleMeetingId** | Query    | string | The id of a meeting that you want to move. The time of this meeting shows as free.               | No       |

The names of these query parameters are in camelCase. Send them exactly as written.

### Response

The response is a list of times, from the earliest to the latest. Each item has these fields:

| Field                  | Type             | Description                                                                                         |
| ---------------------- | ---------------- | --------------------------------------------------------------------------------------------------- |
| **date**               | string           | The start of the time, in ISO 8601 format and UTC. Send this value as `schedule_datetime`.          |
| **booked**             | boolean          | `true` means that no teacher is free at this time.                                                          |
| **scheduled_by_user**  | boolean          | `true` means that the student already has a meeting at this time in this room.                              |
| **teachers**           | array            | The teachers of the room at this time. Each item has `id`, `name` and `status`.                     |
| **physical_rooms_ids** | array of strings | The free physical rooms at this time. If the room does not use physical rooms, this field is not present.   |

The teacher `status` is one of these values:

- `AVAILABLE`: the teacher is free. You can book this teacher.
- `BOOKED_BY_USER`: the teacher has a meeting with this student at this time.
- `UNAVAILABLE`: the teacher is not free.

#### Status

| Code | Description                                                   |
| ---- | ------------------------------------------------------------- |
| 200  | The available schedules were fetched.                         |
| 401  | Unauthorized.                                                 |
| 403  | The user type of the API key cannot use this request. |
| 404  | `Booking room not found`. See [Access rules](#access-rules).  |

#### Example

```json
[
  {
    "date": "2026-11-05T13:00:00.000Z",
    "scheduled_by_user": false,
    "teachers": [
      {
        "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
        "status": "AVAILABLE",
        "name": "Teacher's first name Teacher's last name"
      }
    ],
    "booked": false
  }
]
```

## Create a booking

To book a meeting for a student, send this `POST` request:

```http
POST /bookings/{booking_room_id}/schedule
```

The request uses one credit of the student. The meeting shows in the student's and the teacher's app.

### Request

#### Parameters

| Parameter             | Location | Type   | Description                                                                                                                                                                 | Required |
| --------------------- | -------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **booking_room_id**   | Path     | string | The id of the booking room.                                                                                                                                                 | Yes      |
| **action**            | Body     | string | Use `SCHEDULE`.                                                                                                                                                             | Yes      |
| **schedule_datetime** | Body     | string | The start of the meeting. Use a `date` value from [Get available schedules](#get-available-schedules) without changes.                                                      | Yes      |
| **student_id**        | Body     | string | The id of the student.                                                                                                                                                      | Yes      |
| **teacher_id**        | Body     | string | The id of a teacher with the status `AVAILABLE` at that time.                                                                                                               | Yes      |
| **description**       | Body     | string | A note about the meeting. The teacher can read it.                                                                                                                          | No       |
| **physical_room_id**  | Body     | string | The id of a physical room from `physical_rooms_ids`. If the room does not use physical rooms, the API ignores this field. If you do not send it, the API picks a free physical room.                 | No       |

```json title="Request body"
{
  "action": "SCHEDULE",
  "schedule_datetime": "2026-11-05T13:00:00.000Z",
  "student_id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
  "teacher_id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
  "description": "Review of chapter 4 exercises"
}
```

### Response

The response contains the new meeting. Keep its `id`. You use the `id` to get, move or cancel the meeting. The response contains more fields than the example. Do not depend on fields that this page does not describe. To get the full meeting, use [Get a booking](#get-a-booking).

#### Status

| Code | Description                                                                                                       |
| ---- | ----------------------------------------------------------------------------------------------------------------- |
| 200  | The meeting was booked.                                                                                           |
| 400  | A parameter is missing or has a wrong format.                                                                     |
| 401  | Unauthorized.                                                                                                     |
| 403  | `UNAVAILABLE_DATE`: the time is in the past, the teacher is not free, or no physical room is free.                |
| 403  | `INSUFFICIENT_CREDITS`: the student has no credits in this room.                                                  |
| 403  | `ACTIVE_BOOKING_BLOCK`: the student cannot book in this room now, for example after missed meetings.              |
| 404  | `Booking room not found`. See [Access rules](#access-rules).                                                      |

#### Example

```json
{
  "id": "0b8d3a51-2c4e-4f6a-9b1d-7e3f5a9c2d10",
  "scheduled_for": "2026-11-05T13:00:00.000Z",
  "scheduled_duration_in_minutes": 30,
  "description": "Review of chapter 4 exercises",
  "booking_room": {
    "id": "6f1c2d3e-4b5a-4c6d-8e7f-9a0b1c2d3e4f",
    "minutes_per_session": 30
  },
  "created_by": {
    "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0"
  },
  "teacher": {
    "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823"
  },
  "created_at": "2026-11-04T18:22:41.512Z",
  "updated_at": "2026-11-04T18:22:41.512Z"
}
```

`created_by` is the student. The API records the meeting as the student's booking.

The `ACTIVE_BOOKING_BLOCK` error lists the blocks in `errors`:

```json title="Response Code: 403 Forbidden"
{
  "error": {
    "code": 403,
    "message": "ACTIVE_BOOKING_BLOCK",
    "errors": [
      {
        "no_show_window_hours": 720,
        "no_show_count_to_block": 3,
        "no_show_block_duration_hours": 168,
        "reason": null,
        "end_date": "2026-11-10T12:00:00.000Z"
      }
    ]
  }
}
```

If the room sends an e-mail for each new booking, the student gets that e-mail and Stift sends the `SCHEDULED_MEETING_CREATED` [webhook](webhooks.md#booking-events). Otherwise, Stift does not send this webhook.

## Get bookings

To get the meetings of all the rooms that are shared with your groups, send this `GET` request:

```http
GET /bookings/schedule
```

To get the meetings of one room, send this `GET` request:

```http
GET /bookings/{booking_room_id}/schedule
```

The list contains cancelled meetings unless you hide them. The newest meetings come first.

### Request

#### Parameters

| Parameter                     | Location | Type    | Description                                                                                         | Required |
| ----------------------------- | -------- | ------- | --------------------------------------------------------------------------------------------------- | -------- |
| **booking_room_id**           | Path     | string  | The id of the booking room. Only for the second request.                                            | No       |
| **show_only_future_meetings** | Query    | boolean | If `true`, the list contains only meetings that did not end. The earliest meetings come first. Only for the first request. | No       |
| **show_only_past_meetings**   | Query    | boolean | If `true`, the list contains only meetings that ended. Only for the first request.                  | No       |
| **hide_cancelled_meetings**   | Query    | boolean | If `true`, the list does not contain cancelled meetings.                                            | No       |

### Response

Each item has these fields:

| Field             | Type   | Description                                                                                 |
| ----------------- | ------ | ------------------------------------------------------------------------------------------- |
| **id**            | string | The id of the meeting.                                                                      |
| **user**          | object | The student (`id`, `first_name`, `last_name`, `profile_photo_url`, `online_at`).            |
| **scheduled_for** | string | The booked start time.                                                                      |
| **start_date**    | string | If the meeting started, the real start time. Otherwise, the booked start time.               |
| **end_date**      | string | If the meeting ended, the real end time. Otherwise, the booked end time.                     |
| **status**        | string | See [Meeting status](#meeting-status).                                                      |
| **booking_room**  | object | The room (`id`, `title`, `description`, `type`, `minutes_per_session`, `discipline`). Only in the first request. |
| **physical_room** | object | The physical room (`id`, `name`, `location`). If the meeting has no physical room, this field is not present.                  |

To get the teacher of a meeting, use [Get a booking](#get-a-booking).

#### Status

| Code | Description                                   |
| ---- | --------------------------------------------- |
| 200  | The meetings were fetched.                    |
| 401  | Unauthorized.                                 |
| 403  | The user type of the API key cannot use this request. |

#### Example

```json
[
  {
    "id": "0b8d3a51-2c4e-4f6a-9b1d-7e3f5a9c2d10",
    "user": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student's first name",
      "last_name": "Student's last name",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png",
      "online_at": "2026-11-04T18:20:00.000Z"
    },
    "scheduled_for": "2026-11-05T13:00:00.000Z",
    "start_date": "2026-11-05T13:00:00.000Z",
    "end_date": "2026-11-05T13:30:00.000Z",
    "booking_room": {
      "id": "6f1c2d3e-4b5a-4c6d-8e7f-9a0b1c2d3e4f",
      "title": "Mathematics tutoring",
      "description": null,
      "type": "SCHEDULE",
      "minutes_per_session": 30,
      "discipline": {
        "id": 1,
        "name": "Mathematics"
      }
    },
    "status": "PENDING"
  }
]
```

## Get a booking

To get one meeting, send this `GET` request:

```http
GET /bookings/{booking_room_id}/schedule/{scheduled_meeting_id}
```

### Request

#### Parameters

| Parameter                | Location | Type   | Description                  | Required |
| ------------------------ | -------- | ------ | ---------------------------- | -------- |
| **booking_room_id**      | Path     | string | The id of the booking room.  | Yes      |
| **scheduled_meeting_id** | Path     | string | The id of the meeting.       | Yes      |

### Response

The response contains these fields:

- `id`, `scheduled_for`, `scheduled_duration_in_minutes`, `description`, `created_at` and `status`.
- `user`: the student.
- `teacher`: the teacher (`id`, `first_name`, `last_name`, `profile_photo_url`, `online_at`).
- `booking_room`: the room, with its `discipline`.
- `attachments`: the files that the student sent with the booking.
- `physical_room`: the physical room. If the meeting has no physical room, this field is not present.
- `meeting`: the data of the call, only after the call ended. It contains `started_at`, `ended_at`, `duration` (in seconds), `ratings`, `teacher_notes`, `student_feedback` and `teacher_feedback`.
- `cancelled_by`, `cancellation_reason` and `cancelled_at`. If the status is not `CANCELLED`, these fields are not present.

#### Status

| Code | Description                                   |
| ---- | --------------------------------------------- |
| 200  | The meeting was fetched.                      |
| 401  | Unauthorized.                                 |
| 403  | The user type of the API key cannot use this request. |
| 404  | `Scheduled meeting not found`.                |

## Move a booking

To change the time or the teacher of a meeting, send this `POST` request:

```http
POST /bookings/{booking_room_id}/schedule/{scheduled_meeting_id}
```

You can move only a meeting with the status `PENDING`. The student does not lose or get back a credit. The API sends a notification to the student and to the teachers.

To find a new time, send [Get available schedules](#get-available-schedules) with `studentId` and `rescheduleMeetingId`. The new time must be in that response, with the teacher in the status `AVAILABLE`.

### Request

#### Parameters

| Parameter                | Location | Type   | Description                                                                                         | Required |
| ------------------------ | -------- | ------ | --------------------------------------------------------------------------------------------------- | -------- |
| **booking_room_id**      | Path     | string | The id of the booking room.                                                                         | Yes      |
| **scheduled_meeting_id** | Path     | string | The id of the meeting.                                                                              | Yes      |
| **schedule_datetime**    | Body     | string | The new start time. Use a `date` value from the available schedules without changes.                | Yes      |
| **teacher_id**           | Body     | string | The teacher of the meeting. Send the current teacher to change only the time.                       | Yes      |
| **physical_room_id**     | Body     | string | The id of a physical room from `physical_rooms_ids`. If you do not send it, the API picks a free physical room. | No       |

```json title="Request body"
{
  "schedule_datetime": "2026-11-06T14:00:00.000Z",
  "teacher_id": "f73c6e53-cdca-4f01-b64d-04b9346c9823"
}
```

### Response

The response body is an empty object (`{}`).

#### Status

| Code | Description                                                         |
| ---- | ------------------------------------------------------------------- |
| 200  | The meeting was moved.                                              |
| 400  | `Scheduled meeting status must be PENDING`, or a parameter is wrong. |
| 401  | Unauthorized.                                                       |
| 403  | `UNAVAILABLE_DATE`: the new time or the teacher is not free.        |
| 404  | `Scheduled meeting not found` or `Booking room not found`.          |

## Cancel a booking

To cancel a meeting, send this `DELETE` request:

```http
DELETE /bookings/{booking_room_id}/schedule/{scheduled_meeting_id}
```

You can cancel a meeting until its booked end time. When a group admin cancels a meeting, the student always gets the credit back, and the room's cancellation deadline does not apply. The API sends a notification to the student and to the teacher.

### Request

#### Parameters

| Parameter                | Location | Type   | Description                                                   | Required |
| ------------------------ | -------- | ------ | ------------------------------------------------------------- | -------- |
| **booking_room_id**      | Path     | string | The id of the booking room.                                   | Yes      |
| **scheduled_meeting_id** | Path     | string | The id of the meeting.                                        | Yes      |
| **cancellation_reason**  | Body     | string | The reason for the cancellation. The meeting detail shows it. | No       |

```json title="Request body"
{
  "cancellation_reason": "The student has an exam at this time."
}
```

### Response

#### Status

| Code | Description                                                                         |
| ---- | ----------------------------------------------------------------------------------- |
| 200  | The meeting was cancelled.                                                          |
| 401  | Unauthorized.                                                                       |
| 403  | The user type of the API key cannot use this request. |
| 404  | `Meeting not found`: the meeting does not exist, ended, or is already cancelled.    |

#### Example

```json
{
  "canceled": true
}
```

## Availability Blocks

Availability blocks are used to block time slots during which a teacher is not available and thus cannot be booked.

### Get Availability Blocks

To get a list of all the availability blocks, send this `GET` request:

```http
GET /bookings/availability-blocks
```

#### Request

##### Parameters

| Parameter      | Location | Type                     | Description                                                                                                                                                                          | Required |
| -------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| **start_date** | Body     | Milliseconds since epoch | The date to start showing availability blocks in milliseconds since UNIX Epoch. If not specified, it defaults to the current year's start.                                         | No      |
| **end_date**   | Body     | Milliseconds since epoch | The date to end showing availability blocks in milliseconds since UNIX Epoch. If not specified, it defaults to the current year's end.                                           | No      |

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
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png",
      "disciplines": [
        {"name": "Mathematics"},
        {"name": "Physics"}
      ]
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
