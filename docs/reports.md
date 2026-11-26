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

## Bookings Report

If you want to generate a report about the bookings that happened or are scheduled to happen, send this `POST` request:

```http
POST /reports/bookings
```

### Request

#### Parameters

| Parameter      | Location | Type                     | Description                                                                                                                                                                          | Required |
| -------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| **start_date** | Body     | Milliseconds since epoch | The date to start the report data in milliseconds since UNIX Epoch. For scheduled meetings, considers the date the booking was scheduled at.                                         | Yes      |
| **end_date**   | Body     | Milliseconds since epoch | The date to end the report data in milliseconds since UNIX Epoch. For scheduled meetings, considers the date the booking was scheduled at.                                           | Yes      |
| **school_ids** | Body     | string[]                 | List of the school ids you want to generate the report about. If not specified, it will bring data about all schools you have access to.                                             | No       |
| **Accept**     | Header   | Mime type                | The format you want the report to be in. Supported values are `application/json` and `text/csv`. It defaults to `application/json`. In CSV mode, column names will be in Portuguese. | No       |

### Response

#### Status

| Code | Description                           |
| ---- | ------------------------------------- |
| 200  | The report was generated succesfully. |
| 401  | Unauthorized.                         |

#### Booking Status (Status field)

| Status                  | Description                                                                                      |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| PRESENT                 | Meeting exists and has at least started.                                                         |
| NO_SHOW                 | Scheduled meeting has passed, no meeting exists.                                                 |
| OUTSIDE_SCHEDULE_CANCEL | The student cancelled the booking later than the minimum hours to cancel without losing credits. |
| INSIDE_SCHEDULE_CANCEL  | The student cancelled the booking within the minimum hours to cancel without losing credits.     |
| CANCELLED               | The booking was cancelled by someone else (teacher, group admin).                                |
| PENDING                 | The booking is scheduled to happen.                                                              |
| QUEUE                   | The booking is in the queue.                                                                     |
| LEFT_QUEUE              | The student left the queue without joining a meeting.                                            |
| NOT_REALIZED            | The student joined the queue and the meeting was not held, or the user is still in the queue.    |

#### Example

```json
[
  {
    "id": "a300c187-5883-41dd-a7dc-03d4ebdd5a4d",
    "school_name": "School",
    "created_at": "15/10/2024 15:41",
    "scheduled_for": "17/10/2024 17:00",
    "start_time": "",
    "end_time": "",
    "duration": "",
    "student_external_id": "123456",
    "student_name": "Student Name",
    "student_email": "student@stift.com.br",
    "teacher_name": "Teacher Name",
    "teacher_email": "teacher@stift.com.br",
    "group_name": "Group Name",
    "discipline_name": "Discipline Name",
    "booking_room_name": "Booking Room Name",
    "booking_room_modal": "Online",
    "rating_by_student": 4.9,
    "rating_by_teacher": "",
    "call_quality_rating_by_student": "",
    "call_quality_rating_by_teacher": 5.0,
    "status": "Pendente",
    "cancelled_at": "",
    "cancelled_by_name": ""
  }
]
```

## Bookings Workload

If you want to generate a report about the weekly workload of the teachers, send this `POST` request:

```http
POST /reports/bookings-workload
```

### Request

#### Parameters

| Parameter      | Location | Type      | Description                                                                                                                                                                          | Required |
| -------------- | -------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| **school_ids** | Body     | string[]  | List of the school ids you want to generate the report about. If not specified, it will bring data about all schools you have access to.                                             | No       |
| **Accept**     | Header   | Mime type | The format you want the report to be in. Supported values are `application/json` and `text/csv`. It defaults to `application/json`. In CSV mode, column names will be in Portuguese. | No       |

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
    "teacher_name": "Teacher Name",
    "teacher_email": "teacher@stift.com.br",
    "weekly_minutes": 60,
    "discipline_name": "Discipline Name"
  }
]
```

## Groups Usage Report

To generate a Groups Usage report, send the following `POST` request:

```http
POST /reports/groups-usage
```

### Request

#### Parameters

| Parameter            | Location | Type                     | Description                                                                                                                                                                                                                   | Required |
| -------------------- | -------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **start_date**       | Body     | Milliseconds since epoch | The date to start the report data in milliseconds since UNIX Epoch.                                                                                                                                                           | Yes      |
| **end_date**         | Body     | Milliseconds since epoch | The date to end the report data in milliseconds since UNIX Epoch.                                                                                                                                                             | Yes      |
| **group_ids**        | Body     | string[]                 | List of the group ids you want to generate the report about.                                                                                                                                                                  | Yes      |
| **name**             | Body     | string                   | The name of the report being generated.                                                                                                                                                                                       | No       |
| **show_disciplines** | Body     | boolean                  | Whether to show the disciplines data in the report. It defaults to `false`.                                                                                                                                                   | No       |
| **show_teachers**    | Body     | boolean                  | Whether to show the teachers data in the report. It defaults to `false`.                                                                                                                                                      | No       |
| **Accept**           | Header   | Mime type                | The format you want the report to be in. Supported values are `application/json` (json), `text/csv` (csv), and `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (xlsx). It defaults to `application/json`. | No       |

### Response

#### Status

| Code | Description                            |
| ---- | -------------------------------------- |
| 200  | The report was generated successfully. |
| 401  | Unauthorized.                          |

#### Example Response

```json
{
  "start_date": "01/01/2024",
  "end_date": "04/12/2024",
  "total_questions": 41,
  "average_adjusted_response_time_in_minutes": 43615.19,
  "questions_count_by_discipline": {
    "Matemática": 41
  },
  "active_users_percentage": 1,
  "average_answer_rating": null,
  "average_nps": null,
  "average_response_time_nps": null,
  "nps_answers": [],
  "answer_ratings": [],
  "students": [
    {
      "name": "Aluno Teste",
      "groups": ["Turma de Testes"],
      "email": "student@stift.com.br",
      "active": true,
      "created_at": "2023-04-26",
      "blocked_at": null,
      "deleted_at": null,
      "questions_count": 41,
      "questions": [
        {
          "discipline": {
            "id": 1,
            "name": "Matemática"
          },
          "subject": {
            "id": 1,
            "name": "Álgebra"
          }
        },
        {
          "discipline": {
            "id": 1,
            "name": "Matemática"
          },
          "subject": {
            "id": 1,
            "name": "Álgebra"
          }
        }
      ]
    }
  ],
  "teachers": [
    {
      "first_name": "Professor",
      "last_name": "Teste",
      "email": "teacher@stift.com.br",
      "groups": ["Turma de Testes"],
      "disciplines": ["Matemática"],
      "active": true,
      "answered_questions_count": 3,
      "average_response_time_in_minutes": 43615.19,
      "created_at": "2023-04-26",
      "blocked_at": null,
      "deleted_at": null
    },
    {
      "first_name": "Prof Luccas",
      "last_name": "Santos",
      "email": "teacher2@stift.com.br",
      "groups": ["Turma de Testes"],
      "disciplines": [],
      "active": false,
      "answered_questions_count": 0,
      "average_response_time_in_minutes": null,
      "created_at": "2024-06-03",
      "blocked_at": null,
      "deleted_at": null
    }
  ],
  "disciplines": [
    {
      "name": "Matemática",
      "top_subjects": {
        "Álgebra": 38,
        "Aritmética": 2,
        "Análise combinatória": 1
      },
      "total_questions": 41,
      "average_response_time_in_minutes": 3191.36
    }
  ]
}
```

## Extracurricular Activities Report

To get the [extracurricular activities](extracurricular-activities.md) of a period in a file, send this `POST` request:

```http
POST /reports/extracurricular-activities
```

Only ADMIN and GROUP_ADMIN users can use this request. The `EXTRACURRICULAR_ACTIVITIES` module must be on for the user. The report contains the same activities that [Get activities](extracurricular-activities.md#get-activities) returns to the same user. It also contains activities whose category or type was disabled or deleted later. Deleted activities are not in the report.

The period can be at most 366 days. For a longer period, use [Report Requests](#report-requests).

### Request

#### Parameters

| Parameter        | Location | Type                     | Description                                                                                                                                                                                                                                       | Required |
| ---------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **start_date**   | Body     | Milliseconds since epoch | The start of the period. The report contains activities that end after this time.                                                                                                                                                                 | Yes      |
| **end_date**     | Body     | Milliseconds since epoch | The end of the period. The report contains activities that start before this time.                                                                                                                                                                | Yes      |
| **school_ids**   | Body     | string[]                 | Only activities in these schools. From 1 to 500 ids. If you do not send it, the report contains all schools that you can see.                                                                                                                     | No       |
| **teacher_ids**  | Body     | string[]                 | Only activities of these teachers. From 1 to 500 ids.                                                                                                                                                                                             | No       |
| **category_ids** | Body     | string[]                 | Only activities of these categories. From 1 to 500 ids.                                                                                                                                                                                           | No       |
| **Accept**       | Header   | Mime type                | The format of the report: `application/json` (json), `text/csv` (csv) or `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (xlsx). `*/*` also returns json. **If you do not send this header, the report is in CSV.** Send one value only. | No       |

The report includes each activity that overlaps the period, also when it starts before `start_date` or ends after `end_date`. The report has no filter for types or groups.

```json title="Request body"
{
  "start_date": 1793502000000,
  "end_date": 1796093999000,
  "category_ids": ["8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b"]
}
```

### Response

The oldest activities come first. Each row has these columns:

| Field (json)        | Column (csv, xlsx) | Description                                                        |
| ------------------- | ------------------ | ------------------------------------------------------------------ |
| **school_name**     | Escola             | The school of the activity.                                        |
| **teacher_name**    | Professor          | The full name of the teacher.                                      |
| **teacher_email**   | E-mail do professor | The e-mail of the teacher.                                        |
| **category_name**   | Categoria          | The name of the category.                                          |
| **type_name**       | Tipo               | The name of the type.                                              |
| **description**     | Descrição          | The description. In json, `null` if the activity has none.         |
| **start_date**      | Início             | The start of the activity.                                         |
| **end_date**        | Fim                | The end of the activity.                                           |
| **duration_hours**  | Duração (horas)    | The length of the activity in hours, with up to 2 decimals.        |
| **created_by_name** | Registrado por     | The full name of the user that recorded the activity.              |
| **created_at**      | Registrado em      | When the activity was recorded.                                    |
| **updated_at**      | Atualizado em      | When the activity was last changed.                                |

The format changes the values:

- json: the field names are in English. Dates are in ISO 8601 format and UTC. `duration_hours` is a number, for example `1.5`.
- csv and xlsx: the column names are in Portuguese. Dates are in the `dd/MM/yyyy HH:mm` format, in the São Paulo time zone (`America/Sao_Paulo`).
- csv: the file starts with a UTF-8 byte order mark, lines end with CRLF, and all values are in double quotes. `duration_hours` uses a comma, for example `1,5`.
- csv: if a text value starts with `=`, `+`, `-`, `@`, a tab or a carriage return, the value gets a `'` in front of it, for example `'=SUM(A1)`. This stops spreadsheet programs from running the value as a formula. The json and xlsx values do not change.

For csv and xlsx, the `Content-disposition` header gives a file name such as `atividades-extracurriculares_2026-11-01_a_2026-11-30.csv`.

#### Status

| Code | Description                                                                                                                         |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 200  | The report was generated successfully.                                                                                              |
| 400  | A parameter is missing or has a wrong format, a list has more than 500 ids, or the `Accept` header has a value that is not supported. |
| 401  | Unauthorized.                                                                                                                       |
| 403  | The module is off, or the user type cannot use this request.                                                                        |
| 404  | `No schools found.`: no school of `school_ids` is visible to you.                                                                   |
| 422  | The period is longer than 366 days.                                                                                                 |

```json title="Response Code: 422 Unprocessable Entity"
{
  "error": {
    "code": 422,
    "message": "The date range must be at most 366 days. Use POST /v1/reports/requests for longer ranges."
  }
}
```

#### Example Response

```json
[
  {
    "school_name": "School Name",
    "teacher_name": "Teacher Name",
    "teacher_email": "teacher@stift.com.br",
    "category_name": "Plantão",
    "type_name": "Plantão de dúvidas",
    "description": "Review of chapter 4 exercises",
    "start_date": "2026-11-18T13:00:00.000Z",
    "end_date": "2026-11-18T15:00:00.000Z",
    "duration_hours": 2,
    "created_by_name": "Coordinator Name",
    "created_at": "2026-11-18T15:10:42.318Z",
    "updated_at": "2026-11-18T15:10:42.318Z"
  }
]
```

The same row in CSV:

```text
"Escola","Professor","E-mail do professor","Categoria","Tipo","Descrição","Início","Fim","Duração (horas)","Registrado por","Registrado em","Atualizado em"
"School Name","Teacher Name","teacher@stift.com.br","Plantão","Plantão de dúvidas","Review of chapter 4 exercises","18/11/2026 10:00","18/11/2026 12:00","2","Coordinator Name","18/11/2026 12:10","18/11/2026 12:10"
```

## Report Requests

A report request makes a report file in the background. Use it for large reports, for example an extracurricular activities report for more than 366 days. Do these steps:

1. Create the request with [Create a report request](#create-a-report-request). Keep the `id`.
2. Get the request with [Get a report request](#get-a-report-request) until `status` is `DONE` or `FAILED`.
3. Download the file from `download_url`.

Only ADMIN and GROUP_ADMIN users can use these requests. You can see only the report requests that your API key's user created.

When you create the request, the API saves your access: your schools and your groups. If your access changes later, the request keeps the saved access.

### Report request status

| Status         | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| **PENDING**    | The request is in the queue.                                  |
| **PROCESSING** | Stift is making the file.                                     |
| **DONE**       | The file is ready. `download_url` has the link.               |
| **FAILED**     | Stift could not make the file. `error` has the reason.        |

### Create a report request

To create a report request, send this `POST` request:

```http
POST /reports/requests
```

#### Parameters

`report_type` is one of `QUESTIONS`, `BOOKINGS`, `GROUPS_USAGE`, `EXAM`, `EXAM_ANSWERS` and `EXTRACURRICULAR_ACTIVITIES`. This page shows the parameters for `EXTRACURRICULAR_ACTIVITIES`:

| Parameter        | Location | Type                     | Description                                                                                                  | Required |
| ---------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------ | -------- |
| **report_type**  | Body     | string                   | Use `EXTRACURRICULAR_ACTIVITIES`.                                                                            | Yes      |
| **format**       | Body     | string                   | The format of the file: `csv` or `xlsx`.                                                                     | Yes      |
| **start_date**   | Body     | Milliseconds since epoch | The start of the period. There is no limit on the length of the period.                                      | Yes      |
| **end_date**     | Body     | Milliseconds since epoch | The end of the period.                                                                                       | Yes      |
| **school_ids**   | Body     | string[]                 | Only activities in these schools. From 1 to 500 ids.                                                         | No       |
| **teacher_ids**  | Body     | string[]                 | Only activities of these teachers. From 1 to 500 ids.                                                        | No       |
| **category_ids** | Body     | string[]                 | Only activities of these categories. From 1 to 500 ids.                                                      | No       |

The file has the same rows and columns as the csv or xlsx of the [Extracurricular Activities Report](#extracurricular-activities-report).

```json title="Request body"
{
  "report_type": "EXTRACURRICULAR_ACTIVITIES",
  "format": "xlsx",
  "start_date": 1769914800000,
  "end_date": 1796093999000
}
```

#### Status

| Code | Description                                                                         |
| ---- | ----------------------------------------------------------------------------------- |
| 201  | The request was created.                                                            |
| 400  | A parameter is missing or has a wrong format, or a list has more than 500 ids.      |
| 401  | Unauthorized.                                                                       |
| 403  | The module is off, or the user type cannot use this request.                        |
| 404  | `No schools found.`: no school of `school_ids` is visible to you.                   |

#### Example Response

```json
{
  "id": "7c8d9e0f-1a2b-4c3d-8e4f-5a6b7c8d9e0f",
  "report_type": "EXTRACURRICULAR_ACTIVITIES",
  "format": "xlsx",
  "status": "PENDING",
  "created_at": "2026-11-26T14:02:11.904Z"
}
```

### Get a report request

To get the status of a report request, send this `GET` request:

```http
GET /reports/requests/{report_request_id}
```

#### Parameters

| Parameter             | Location | Type   | Description                   | Required |
| --------------------- | -------- | ------ | ----------------------------- | -------- |
| **report_request_id** | Path     | string | The id of the report request. | Yes      |

#### Response

| Field            | Type           | Description                                                                                       |
| ---------------- | -------------- | ------------------------------------------------------------------------------------------------- |
| **id**           | string         | The id of the report request.                                                                     |
| **report_type**  | string         | The report type.                                                                                  |
| **format**       | string         | `csv` or `xlsx`.                                                                                  |
| **status**       | string         | See [Report request status](#report-request-status).                                              |
| **params**       | object         | The parameters that the API saved for the report. Do not depend on its fields.                    |
| **file_name**    | string \| null | The name of the file, when `status` is `DONE`.                                                    |
| **error**        | string \| null | The reason, when `status` is `FAILED`.                                                            |
| **created_at**   | string         | When the request was created.                                                                     |
| **finished_at**  | string \| null | When the file was ready or the request failed.                                                    |
| **download_url** | string         | A link to the file. Only when `status` is `DONE`. The link expires after one hour. To get a new link, send this request again. |

#### Status

| Code | Description                                                                                |
| ---- | ------------------------------------------------------------------------------------------ |
| 200  | The report request was fetched.                                                            |
| 400  | `report_request_id` is not a UUID.                                                         |
| 401  | Unauthorized.                                                                              |
| 403  | The user type cannot use this request.                                                     |
| 404  | `Report request not found.`: the request does not exist, or another user created it.      |

#### Example Response

```json
{
  "id": "7c8d9e0f-1a2b-4c3d-8e4f-5a6b7c8d9e0f",
  "report_type": "EXTRACURRICULAR_ACTIVITIES",
  "format": "xlsx",
  "status": "DONE",
  "params": {},
  "file_name": "atividades-extracurriculares_2026-02-01_a_2026-11-30.xlsx",
  "error": null,
  "created_at": "2026-11-26T14:02:11.904Z",
  "finished_at": "2026-11-26T14:02:19.377Z",
  "download_url": "https://files.example.com/reports/7c8d9e0f-1a2b-4c3d-8e4f-5a6b7c8d9e0f/atividades-extracurriculares_2026-02-01_a_2026-11-30.xlsx?signature=..."
}
```

The example does not show the fields of `params`.

### Get report requests

To get the report requests that you created, send this `GET` request:

```http
GET /reports/requests
```

The newest requests come first. The items have the same fields as in [Get a report request](#get-a-report-request), but no `download_url`.

#### Parameters

| Parameter  | Location | Type    | Description                                                                    | Required |
| ---------- | -------- | ------- | ------------------------------------------------------------------------------ | -------- |
| **limit**  | Query    | integer | The maximum number of requests to return, from 1 to 100. The default is `20`.  | No       |
| **offset** | Query    | integer | The number of requests to skip. The default is `0`.                            | No       |

#### Status

| Code | Description                                 |
| ---- | ------------------------------------------- |
| 200  | The report requests were fetched.           |
| 400  | A parameter has a wrong format.             |
| 401  | Unauthorized.                               |
| 403  | The user type cannot use this request.      |
