# Reports

Reports are used to get information about the platform's usage.

## Questions Report

If you want to generate a report about the questions, send this `POST` request:

```http
POST /reports/questions
```

### Request

#### Parameters

| Parameter      | Location | Type                     | Description                                                                                                                            | Required |
| -------------- | -------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **start_date** | Body     | Milliseconds since epoch | The date to start the report data in milliseconds since UNIX Epoch.                                                                    | Yes      |
| **end_date**   | Body     | Milliseconds since epoch | The date to end the report data in milliseconds since UNIX Epoch.                                                                      | Yes      |
| **group_ids**  | Body     | string[]                 | List of the group ids you want to generate the report about. If not specified, it will bring data about all groups you have access to. | No       |
| **Accept**     | Header   | Mime type                | The format you want the report to be in. Supported values are `application/json` and `text/csv`. It defaults to `application/json`.    | No       |

### Response

#### Status

| Code | Description                           |
| ---- | ------------------------------------- |
| 200  | The report was generated succesfully. |
| 401  | Unauthorized.                         |

#### Example

```json
[
  {
    "question_id": "706cb23b-6e65-4863-afec-4d5fb2af17a6",
    "school": "School",
    "group": "Group",
    "student_id": "fc2779a8-5229-4656-86d5-4e742b61c1c5",
    "student_name": "Student Name",
    "student_email": "student@stift.com.br",
    "teacher_id": "330fa799-efbb-45c0-8755-34d928b8a3c4",
    "teacher_name": "Teacher Name",
    "teacher_email": "teacher@stift.com.br",
    "question_discipline": "Discipline",
    "question_subject": "Subject",
    "question_created_at": "2024-02-19T14:04:02.057Z",
    "question_answered_at": "2024-02-19T18:18:43.599Z",
    "question_status": "Respondida",
    "question_anonymous": false,
    "question_description": "Description",
    "answer_description": "",
    "reply_description": "",
    "reply_answer_description": "",
    "answer_rating": "",
    "answer_understood": "",
    "answer_comment": "",
    "has_seen_answer": false,
    "link": "https://portal.stift.com.br/#/questions/706cb23b-6e65-4863-afec-4d5fb2af17a6/"
  }
]
```
