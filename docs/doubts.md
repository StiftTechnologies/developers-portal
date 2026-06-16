# Doubts

Doubts represent a student's question submission to a group. As a **GROUP_ADMIN**, you can create doubts on behalf of students using the `on_behalf_of` field.


To list, inspect, and rate questions from your groups, see [Questions](questions.md).

## Create a doubt

If you want to create a doubt on behalf of a student, send this `POST` request:

```http
POST /doubts
```

**To allow unlimited questions for the student in a group, set `remaining_questions` to -1 in the student's [GroupData](users.md#groupdata) before creating the doubt, or ensure the user is linked to the group with the default unlimited quota.**

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **discipline** | Body | int | Discipline id | Yes |
| **on_behalf_of** | Body | object | Student to act on behalf of. Required for GROUP_ADMIN and ADMIN. | Yes (GROUP_ADMIN/ADMIN) |
| **on_behalf_of.first_name** | Body | string | Student's first name | Yes (when `on_behalf_of` is sent) |
| **on_behalf_of.last_name** | Body | string | Student's last name | Yes (when `on_behalf_of` is sent) |
| **on_behalf_of.email** | Body | string | Student's email. **If a user with the same email already exists, that user is reused; otherwise a new STUDENT is created.** | Yes (when `on_behalf_of` is sent) |
| **on_behalf_of.group.id** | Body | string | Group id. Must belong to your GROUP_ADMIN groups. The student is linked to this group if not already. | Yes (when `on_behalf_of` is sent) |
| **questions** | Body | array | List of question objects | Yes |
| **questions[].subject** | Body | int | Subject id | Yes |
| **questions[].type** | Body | string | Question type | Yes |
| **questions[].description** | Body | string | Question text | No |
| **questions[].attachments** | Body | array | List of file names (with extension) to upload after creation | No |
| **answer_preferences** | Body | array | Answer preference ids (max 3) | No |
| **anonymous** | Body | boolean | Whether the doubt is anonymous. Defaults to `false`. | No |
| **group_id** | Body | string | Destination group id. When omitted, defaults to `on_behalf_of.group.id`. | No |
| **send_directly_to_teachers** | Body | boolean | Skip AI and send directly to teachers. Defaults to `false`. | No |

#### Example

```json
{
  "discipline": 1,
  "on_behalf_of": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@stift.com.br",
    "group": {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
    }
  },
  "anonymous": false,
  "questions": [
    {
      "type": "QUESTION",
      "subject": 12,
      "description": "How do I solve this equation?",
      "attachments": ["photo.jpg"]
    }
  ]
}
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The doubt was successfully created. |
| 400 | Some parameter must be malformed or missing. Check the response for more. |
| 401 | Unauthorized. |
| 403 | Forbidden. The group in `on_behalf_of.group.id` is not in your GROUP_ADMIN groups. |

#### Example

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "discipline": {
    "id": 1,
    "name": "Mathematics"
  },
  "destination_group": {
    "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
  },
  "status": {
    "id": 2,
    "name": "Pending upload"
  },
  "questions": [
    {
      "id": "f9e8d7c6-b5a4-3210-fedc-ba9876543210",
      "description": "How do I solve this equation?",
      "attachments": [
        {
          "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
          "status": "PENDING_UPLOAD",
          "file_name": "c3d4e5f6-a7b8-9012-cdef-345678901234.jpg",
          "type": "image/jpeg",
          "upload": {
            "id": "d4e5f6a7-b8c9-0123-def0-456789012345",
            "url": "https://s3.amazonaws.com/...",
            "expires_in": 900,
            "headers": {
              "Content-Type": "image/jpeg",
              "x-amz-acl": "private"
            },
            "notify_upload_url": "https://api.stift.com.br/v1/upload/d4e5f6a7-b8c9-0123-def0-456789012345/notify",
            "refresh_token": {
              "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
              "expires_in": 259200
            }
          }
        }
      ]
    }
  ],
  "created_at": "2026-06-14T18:00:00.000Z"
}
```

### Uploading attachments

When a question includes `attachments`, send file **names** (with extension) in the create request. The response includes an `upload` object per attachment. You can upload the file using **either** method below.

#### Upload object

```json title="Upload object in attachment response"
{
  "id": "d4e5f6a7-b8c9-0123-def0-456789012345",
  "url": "https://s3.amazonaws.com/...",
  "expires_in": 900,
  "headers": {
    "Content-Type": "image/jpeg",
    "x-amz-acl": "private"
  },
  "notify_upload_url": "https://api.stift.com.br/v1/upload/d4e5f6a7-b8c9-0123-def0-456789012345/notify",
  "refresh_token": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 259200
  },
  "tus_upload": {
    "url": "https://uploader.stift.com.br/files",
    "metadata": {
      "upload_id": "d4e5f6a7-b8c9-0123-def0-456789012345",
      "content_type": "image/jpeg"
    }
  }
}
```

The question only becomes available after the upload is completed and finalized.

#### Option A — Binary upload (signed S3 URL)

Best for small files and single-shot uploads. Not resumable.

1. Upload the file with a `PUT` request to `upload.url`, using the headers from `upload.headers` and the file binary as the body:

```http
PUT {upload.url}
Content-Type: image/jpeg
x-amz-acl: private

<file binary>
```

2. Notify completion:

```http
POST {upload.notify_upload_url}
```

3. If `upload.url` expires before you finish (`upload.expires_in` seconds), request a new signed URL:

```http
POST /upload/refresh
```

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Option B — Resumable upload (tus)

Best for large files or unstable connections. Supports pause and resume.

1. Use a [tus](https://tus.io) client pointed at `https://uploader.stift.com.br/files`.
2. Send the header `Authorization: Bearer <your GROUP_ADMIN API key>`.
3. Send tus metadata:
   - `upload_id` — value of `upload.id`
   - `content_type` — value of the attachment `type` (e.g. `image/jpeg`)
4. Finalization is **automatic** on the server after upload completes. You do **not** need to call `notify_upload_url`.

The tus endpoint accepts your **GROUP_ADMIN API key** for authentication. The upload is linked to the attachment via `upload_id`; the underlying `Upload` record is already associated with the student (the user in `on_behalf_of`).
