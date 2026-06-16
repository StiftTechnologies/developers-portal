# Questions

Questions are student submissions inside a doubt. As a **GROUP_ADMIN**, you can list and inspect questions from the groups you manage, and act on behalf of students using the `on_behalf_of` field (same shape as [Create a doubt](doubts.md#create-a-doubt)).

## List questions

If you want to search and filter questions from your groups, send this `GET` request:

```http
GET /questions
```

Results are paginated. The total number of matching questions is returned in the **`X-Total-Count`** response header (exposed via `Access-Control-Expose-Headers`).

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **query** | Query | string | Full-text search on the question content. | No |
| **group** | Query | string (UUID) | Filter by a single group id. Must be one of your groups. | No |
| **group_ids** | Query | string | Comma-separated list of group ids. Only ids from your groups are applied. If omitted, all your groups are included. | No |
| **school_ids** | Query | string | Comma-separated list of school ids. | No |
| **discipline_ids** | Query | string | Comma-separated list of discipline ids. | No |
| **start_date** | Query | int | Filter questions created on or after this timestamp (milliseconds since UNIX epoch). | No |
| **end_date** | Query | int | Filter questions created on or before this timestamp (milliseconds since UNIX epoch). | No |
| **rating_range** | Query | string | Comma-separated min and max answer rating (e.g. `3,5`). Both values between 0 and 5. | No |
| **status_id** | Query | int | Filter by question status id. | No |
| **has_reply** | Query | boolean | `true` — only questions with a student reply. `false` — only questions without a reply. | No |
| **user_ids** | Query | string | Comma-separated list of user UUIDs. Returns questions where **any** of these users is the student who asked **or** the teacher who answered. Useful to list questions from specific students or answered by specific teachers. | No |
| **cost_center_ids** | Query | string | Comma-separated list of cost center ids. | No |
| **sort** | Query | string | Sort order. One or more pairs separated by commas: `field direction`. Fields: `created_at`, `rating`. Directions: `ASC`, `DESC`. Example: `created_at DESC` or `rating ASC,created_at DESC`. Defaults to `created_at DESC`. | No |
| **limit** | Query | int | Maximum number of questions to return. Defaults to `100`. | No |
| **offset** | Query | int | Number of questions to skip for pagination. Defaults to `0`. | No |

##### `user_ids` filter

Send user ids as a comma-separated string:

```http
GET /questions?user_ids=fc2779a8-5229-4656-86d5-4e742b61c1c5,330fa799-efbb-45c0-8755-34d928b8a3c4
```

A question matches when **either**:

- the **student** who created the question is in `user_ids`, or
- the **teacher** who answered the question is in `user_ids`.

Combine with other filters (e.g. `group_ids`, `start_date`) to narrow results.

#### Example

```http
GET /questions?group_ids=bee1b51e-1843-443b-8bd2-9c46c86373c5&user_ids=fc2779a8-5229-4656-86d5-4e742b61c1c5&start_date=1710000000000&limit=50&sort=created_at DESC
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The search was successful. Returns a list of questions. |
| 400 | Some parameter must be malformed or missing. Check the response for more. |
| 401 | Unauthorized. |
| 403 | Forbidden. |

#### Headers

| Header | Description |
| ------ | ----------- |
| **X-Total-Count** | Total number of questions matching the filters (not only the current page). |

#### Example

```json
[
  {
    "id": "706cb23b-6e65-4863-afec-4d5fb2af17a6",
    "type": "QUESTION",
    "created_at": "2026-06-14T14:04:02.057Z",
    "updated_at": "2026-06-14T18:18:43.599Z",
    "has_reply": false,
    "status": {
      "id": 3,
      "name": "Available",
      "icon_url": "https://sa-east-1-public-cdn.stift.com.br/..."
    },
    "subject": {
      "name": "Algebra"
    },
    "discipline": {
      "id": 1,
      "name": "Mathematics",
      "icon_url": "https://sa-east-1-public-cdn.stift.com.br/..."
    },
    "created_by": {
      "id": "fc2779a8-5229-4656-86d5-4e742b61c1c5",
      "first_name": "John",
      "last_name": "Doe",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/..."
    },
    "answer": {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "created_at": "2026-06-14T18:18:43.599Z",
      "response_time_in_seconds": 14400,
      "seen": true,
      "answerRating": {
        "understood": true,
        "rating": 4.5
      },
      "created_by": {
        "id": "330fa799-efbb-45c0-8755-34d928b8a3c4",
        "first_name": "Jane",
        "last_name": "Smith",
        "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/..."
      }
    },
    "group": {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5",
      "name": "Class 1"
    }
  }
]
```

## Get a question

If you want the full details of a single question (attachments, AI hint/solution, parent and child questions, answer), send this `GET` request:

```http
GET /question/:questionId
```

As a **GROUP_ADMIN**, you can only access questions whose destination group is one of your groups.

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **questionId** | Path | string (UUID) | Question id | Yes |

#### Example

```http
GET /question/706cb23b-6e65-4863-afec-4d5fb2af17a6
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The question was returned successfully. |
| 401 | Unauthorized. |
| 404 | Question not found or not in one of your groups. |

#### Example

```json
{
  "id": "706cb23b-6e65-4863-afec-4d5fb2af17a6",
  "type": "QUESTION",
  "description": "How do I factor this expression?",
  "created_at": "2026-06-14T14:04:02.057Z",
  "available_at": "2026-06-14T14:04:05.000Z",
  "subjects": ["Algebra"],
  "status": {
    "id": 3,
    "name": "Available",
    "icon_url": "https://sa-east-1-public-cdn.stift.com.br/...",
    "message_student": "Your question is available to teachers."
  },
  "subject": {
    "id": 12,
    "name": "Algebra"
  },
  "attachments": [
    {
      "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
      "type": "image/jpeg",
      "url": "https://..."
    }
  ],
  "doubt": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "anonymous": false,
    "discipline": {
      "id": 1,
      "name": "Mathematics"
    },
    "created_by": {
      "id": "fc2779a8-5229-4656-86d5-4e742b61c1c5",
      "first_name": "John",
      "last_name": "Doe",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/..."
    },
    "destination_group": {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5",
      "name": "Class 1"
    },
    "answer_preferences": []
  },
  "answer": {
    "id": "f9e8d7c6-b5a4-3210-fedc-ba9876543210",
    "description": "Here is the step-by-step solution...",
    "created_at": "2026-06-14T18:18:43.599Z",
    "rating": 4.5,
    "rated_at": "2026-06-15T10:00:00.000Z",
    "answerRating": {
      "understood": true,
      "rating": 4.5,
      "tags": [],
      "comment": null,
      "created_at": "2026-06-15T10:00:00.000Z"
    },
    "created_by": {
      "id": "330fa799-efbb-45c0-8755-34d928b8a3c4",
      "first_name": "Jane",
      "last_name": "Smith",
      "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/..."
    },
    "attachments": []
  },
  "hint": {
    "status": "GENERATED",
    "intro": "...",
    "hint": "...",
    "created_at": "2026-06-14T14:05:00.000Z"
  },
  "solution": {
    "status": "GENERATED",
    "solution": "...",
    "audio_url": "https://...",
    "generated_image_url": "https://...",
    "created_at": "2026-06-14T14:06:00.000Z"
  },
  "child_question": null,
  "parent_question": null
}
```

Attachment, hint, solution, and answer fields may be `null` depending on question state. Presigned URLs on attachments and on `solution.audio_url` / `solution.generated_image_url` are time-limited.

## Get a question solution

If you want the AI-generated solution for a question (text, audio, image, follow-ups, and any existing rating), send this `GET` request:

```http
GET /question/:questionId/solution
```

This endpoint is for **STUDENT** users. The solution is only returned when the authenticated student is the one who created the question.

As a **GROUP_ADMIN**, use [Get a question](#get-a-question) instead — the response includes the `solution` object with presigned `audio_url` and `generated_image_url` when available.

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **questionId** | Path | string (UUID) | Question id | Yes |

#### Example

```http
GET /question/706cb23b-6e65-4863-afec-4d5fb2af17a6/solution
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The solution was returned successfully. |
| 401 | Unauthorized. |
| 404 | Solution not found, or the question does not belong to the authenticated student. |

#### Example

```json
{
  "status": "GENERATED",
  "solution": "To factor this expression, first identify the common factors...",
  "generated_at": "2026-06-14T14:06:00.000Z",
  "audio_url": "https://...",
  "audio_alignment": [],
  "generated_image_url": "https://...",
  "follow_ups": [
    {
      "id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
      "text": "Can you explain step 2?",
      "response": null,
      "status": "PENDING"
    }
  ],
  "rating": {
    "was_question_answered": null,
    "teacher_answer_needed": true,
    "feedback": null
  }
}
```

`audio_url`, `generated_image_url`, `audio_alignment`, `follow_ups`, and `rating` may be omitted or empty depending on question state. Presigned URLs are time-limited.

## Rate a question solution

If you want to rate an AI-generated solution on behalf of a student, send this `PUT` request. **You must send `on_behalf_of` with the same shape as [Create a doubt](doubts.md#create-a-doubt).**

```http
PUT /question/:questionId/solution/rating
```

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **questionId** | Path | string | Question UUID | Yes |
| **teacher_answer_needed** | Body | boolean | Whether the student still needs a teacher to answer the question | No |

#### Example

```json
{
  "on_behalf_of": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@stift.com.br",
    "group": {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
    }
  },
  "teacher_answer_needed": true
}
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The solution rating was saved successfully. |
| 400 | Some parameter must be malformed or missing. Check the response for more. |
| 401 | Unauthorized. |
| 403 | Forbidden. The group in `on_behalf_of.group.id` is not in your GROUP_ADMIN groups. |
| 404 | Solution not found for this question. |

#### Example

```json
{
  "teacher_answer_needed": true,
  "feedback": null,
  "was_question_answered": null,
  "created_at": "2026-06-14T18:30:00.000Z",
  "updated_at": "2026-06-14T18:30:00.000Z"
}
```
