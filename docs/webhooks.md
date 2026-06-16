# Webhooks

In order to allow for a seamless integration with your system, we provide a way to notify you when events happen on Stift, such as when a question is answered, a user is created, or a booking room session starts.

When a subscribed event happens, we send an HTTP `POST` request to the URL you configured for your school. Webhooks are only delivered for events that you have explicitly enabled.

## Request Format

Every webhook is delivered as a `POST` request with a JSON body that follows the same envelope, regardless of the event:

```http
POST https://your-system.com/your-webhook-endpoint
```

### Headers

| Header | Description |
| ----- | ----- |
| **Content-Type** | Always `application/json`. |
| **Stift-Webhook-Id** | The unique ID of the webhook configuration that triggered the request. Use it to identify which configuration sent the event. |
| **Stift-Event-Id** | The unique ID of the event. Use it to guarantee idempotency (see [Idempotency](#idempotency)). |

### Body

| Parameter | Type | Description |
| ----- | --- | ----- |
| **event** | string | The event type that was triggered (e.g. `QUESTION_ANSWERED`). |
| **created_at** | string | ISO 8601 timestamp of when the event was generated. |
| **data** | object | The event payload. Its shape depends on the event category (see below). |

```json
{
  "event": "QUESTION_ANSWERED",
  "created_at": "2024-12-12T13:20:08.348Z",
  "data": {}
}
```

## Question Events

Question events are sent whenever something happens to a question or a reply. They all share the same `data` payload shape. Fields related to the answer, hint, solution, or ratings are only populated once they exist on the question; otherwise they are `null`.

### Available Events

| Event | Description |
| ---- | ---- |
| **QUESTION_CREATED** | A question was created. |
| **QUESTION_AVAILABLE** | A question was created and is now available for processing. |
| **QUESTION_AVAILABLE_TO_TEACHERS** | A question became available for teachers to answer. |
| **QUESTION_UNAVAILABLE_TO_TEACHERS** | A question is no longer available for teachers to answer. |
| **QUESTION_UPDATED** | A question changed state (e.g. it was routed to teachers after a solution evaluation). |
| **QUESTION_ANSWERED** | A question was answered by a teacher. |
| **QUESTION_SOLUTION_CREATED** | An AI solution was generated for a question. |
| **QUESTION_SOLUTION_RATED** | A question's AI solution was rated by the student. |
| **ANSWER_AVAILABLE_TO_STUDENT** | An answer became available to the student. |
| **ANSWER_UPDATED** | An existing answer was edited by the teacher. |
| **ANSWER_RATED** | An answer was rated. |
| **REPLY_CREATED** | A reply (follow-up question) was created. |
| **REPLY_AVAILABLE** | A reply became available for processing. |
| **REPLY_ANSWERED** | A reply (follow-up question) was answered by a teacher. |

### Payload

| Parameter | Type | Description |
| ----- | --- | ----- |
| **id** | string | The question ID. |
| **created_at** | string | When the question was created. |
| **student** | object | The student who created the question (`id`, `first_name`, `last_name`, `email`). |
| **answer** | object | The answer data, or `null` if not answered yet. Contains `id`, `created_at`, and the `teacher` who answered. |
| **group** | object | The destination group (`id`, `name`, `business_model`). |
| **school** | object | The school the group belongs to (`id`, `name`). |
| **discipline** | object | The question's discipline (`id`, `name`). |
| **subject** | object | The question's subject (`id`, `name`). |
| **status** | object | The current question status (`id`). |
| **hint** | object | The AI-generated hint, or `null`. |
| **hint_rating** | object | The student's rating of the hint, or `null`. |
| **solution** | object | The AI-generated solution, or `null`. |
| **solution_rating** | object | The student's rating of the solution, or `null`. |
| **url** | string | A deep link to the question in the Stift app. |

#### Example

```json title="Event: QUESTION_ANSWERED"
{
  "event": "QUESTION_ANSWERED",
  "created_at": "2024-12-12T13:20:08.348Z",
  "data": {
    "id": "c64a24e4-ebb5-40ef-8828-9a7c006b5062",
    "created_at": "2024-12-12T13:17:08.348Z",
    "student": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student",
      "last_name": "Test",
      "email": "teststudent@stift.com.br"
    },
    "answer": {
      "id": "b043caa2-af8d-464d-9e74-40da4ead9ffd",
      "created_at": "2024-12-12T13:20:08.348Z",
      "teacher": {
        "id": "09c25b20-266e-49d8-89d2-e7622c84a6d9",
        "first_name": "Teacher",
        "last_name": "Test",
        "email": "testteacher@stift.com.br"
      }
    },
    "group": {
      "id": "e5f8abe2-53f2-48bc-b7a4-d6167b9e900f",
      "name": "Test Group",
      "business_model": "B2B_SAAS"
    },
    "school": {
      "id": "8f03cc77-bf49-4a9f-9b7c-e7dba5f068d7",
      "name": "Test School"
    },
    "discipline": {
      "id": "4c2b6d1e-9f3a-4b2c-8d1e-2f3a4b5c6d7e",
      "name": "Math"
    },
    "subject": {
      "id": "1a2b3c4d-5e6f-7a8b-9c0d-1e2f3a4b5c6d",
      "name": "Algebra"
    },
    "status": {
      "id": 5
    },
    "hint": null,
    "hint_rating": null,
    "solution": null,
    "solution_rating": null,
    "url": "https://app.stift.com.br/?question=c64a24e4-ebb5-40ef-8828-9a7c006b5062"
  }
}
```

The `REPLY_ANSWERED` event uses the exact same payload shape; it is sent when a reply (a follow-up question) is answered instead of an original question.

#### Solution and Rating Example

When the event is related to a solution (e.g. `QUESTION_SOLUTION_RATED`), the `solution` and `solution_rating` fields are populated:

```json title="solution and solution_rating fields"
{
  "solution": {
    "status": "OK",
    "solution": {
      "type": 1,
      "structure": { "intro": true, "cards": 2, "outro": true },
      "intro": { "title": "Overview", "content": "We will solve step by step." },
      "cards": [
        { "id": "card-1", "collapsed": false, "collapsible": true, "title": "Step 1" }
      ],
      "cards_content": [
        { "id": "card-1", "content": "Move all terms to one side." }
      ],
      "outro": { "title": "Result", "content": "x = 4" }
    },
    "created_at": "2024-12-12T13:18:00.000Z",
    "updated_at": "2024-12-12T13:18:00.000Z"
  },
  "solution_rating": {
    "teacher_answer_needed": false,
    "feedback": "Clear explanation.",
    "was_question_answered": "YES",
    "created_at": "2024-12-12T13:19:00.000Z",
    "updated_at": "2024-12-12T13:19:00.000Z"
  }
}
```

## User Events

User events are sent when something happens to a user account.

### Available Events

| Event | Description |
| ---- | ---- |
| **STUDENT_CREATED** | A student account was created. |
| **FIRST_LOGIN** | A student logged in for the first time. |

### Payload

| Parameter | Type | Description |
| ----- | --- | ----- |
| **id** | string | The user ID. |
| **created_at** | string | When the user was created. |
| **student** | object | The user's data (`id`, `first_name`, `last_name`, `email`, `type`). |
| **url** | string | A deep link to the user in the Stift app. |

The `type` field inside `student` can be `STUDENT` or `TEACHER`.

#### Example

```json title="Event: STUDENT_CREATED"
{
  "event": "STUDENT_CREATED",
  "created_at": "2024-12-12T13:17:08.348Z",
  "data": {
    "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
    "created_at": "2024-12-12T13:17:08.348Z",
    "student": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student",
      "last_name": "Test",
      "email": "teststudent@stift.com.br",
      "type": "STUDENT"
    },
    "url": "https://app.stift.com.br/?user=3591f1c3-cf26-4f36-b4d2-d7078359c0d0"
  }
}
```

## Booking Events

Booking events are sent when something happens inside a booking room, such as a student joining a queue, a meeting being created, or a session starting. They all share the same `data` payload shape; the `meeting`, `session`, and `queue_users` fields are only populated when relevant to the event.

### Available Events

| Event | Description |
| ---- | ---- |
| **STUDENT_JOINED_BOOKING_ROOM_QUEUE** | A student joined the booking room queue. |
| **STUDENT_LEFT_BOOKING_ROOM_QUEUE** | A student left the booking room queue. |
| **STUDENT_MISSED_BOOKING_ROOM_QUEUE_JOINING_WINDOW** | A student missed the window to join after being invited. |
| **BOOKING_ROOM_MEETING_CREATED** | A meeting between a student and a teacher was created. |
| **BOOKING_ROOM_MEETING_ENDED** | A meeting ended. |
| **BOOKING_ROOM_SESSION_STARTED** | A teacher started a session in the booking room. |
| **BOOKING_ROOM_SESSION_ENDED** | A teacher's session ended. |
| **TEACHER_FEEDBACK_PROVIDED** | A teacher provided feedback for a meeting. |
| **SCHEDULED_MEETING_CREATED** | A meeting was scheduled. |

### Payload

| Parameter | Type | Description |
| ----- | --- | ----- |
| **booking_room** | object | The booking room (`id`, `title`, `type`, `minutes_per_session`, `is_virtual`, `location`, and `belongs_to.id` for the school). |
| **user** | object | The user that triggered the event (`id`, `first_name`, `type`, and `current_booking_room_session.id`). |
| **meeting** | object | The meeting data, when applicable. Contains `id`, `started_at`, `ended_at`, `session.id`, `student.id`, `teacher.id`. |
| **session** | object | The session data, when applicable. Contains `id`, `status`, `active_meeting.id`, `users`, `owner.id`. |
| **queue_users** | array | The current queue, ordered by position. Only present for queue-type booking rooms. Each entry has `id`, `joined_at`, `user.id`. |

The booking room `type` can be `SCHEDULE` or `QUEUE`. The session `status` can be `ACTIVE` or `INACTIVE`.

#### Example

```json title="Event: BOOKING_ROOM_MEETING_CREATED"
{
  "event": "BOOKING_ROOM_MEETING_CREATED",
  "created_at": "2024-12-12T13:20:08.348Z",
  "data": {
    "booking_room": {
      "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
      "title": "Math Tutoring",
      "type": "QUEUE",
      "minutes_per_session": 30,
      "is_virtual": true,
      "location": null,
      "belongs_to": {
        "id": "8f03cc77-bf49-4a9f-9b7c-e7dba5f068d7"
      }
    },
    "user": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student",
      "type": "STUDENT",
      "current_booking_room_session": {
        "id": "7c6b5a4d-3e2f-1a0b-9c8d-7e6f5a4b3c2d"
      }
    },
    "meeting": {
      "id": "b043caa2-af8d-464d-9e74-40da4ead9ffd",
      "started_at": "2024-12-12T13:20:08.348Z",
      "ended_at": null,
      "session": {
        "id": "7c6b5a4d-3e2f-1a0b-9c8d-7e6f5a4b3c2d"
      },
      "student": {
        "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0"
      },
      "teacher": {
        "id": "09c25b20-266e-49d8-89d2-e7622c84a6d9"
      }
    },
    "session": null,
    "queue_users": [
      {
        "id": "d4c3b2a1-f6e5-8b7a-0d9c-2f1e4b3a6c5d",
        "joined_at": "2024-12-12T13:15:08.348Z",
        "user": {
          "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0"
        }
      }
    ]
  }
}
```

## Security

In order to provide you with a way to validate the events, we attribute a unique ID to each webhook configuration. You can use this ID to validate the authenticity of the event. It is sent in the webhook's request headers, as `Stift-Webhook-Id`.

## Retry Policy

Our webhooks system is designed to retry sending the event in case of failure. We will retry sending the event up to 3 times, with an exponential backoff strategy of 1, 2, and 4 seconds. A delivery is considered successful when your endpoint responds with a `2xx` status code.

## Idempotency

To prevent the same event from being processed multiple times in case of retries, we provide a way for you to ensure that the event is processed only once. You can use the `Stift-Event-Id` header to store the event's ID and check if it has already been processed.
