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
| **start_date** | Body     | Milliseconds since epoch | The date to start the report data in milliseconds since UNIX Epoch. For scheduled meetings, considers the date the booking is scheduled for.                                         | Yes      |
| **end_date**   | Body     | Milliseconds since epoch | The date to end the report data in milliseconds since UNIX Epoch. For scheduled meetings, considers the date the booking is scheduled for.                                           | Yes      |
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
