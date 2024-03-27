# Groups

Groups are the way to organize users, as in classes. Once users are inside a group, they can only see the data that is related to that group. A user can be in multiple groups at a time. Groups are part of a school.

### Business Model

The business model is the way the school will use the app. It defines which modules are available to the school and how the questions are answered.

- **B2B_SAAS**: The group's questions are answered by the school's teachers.
- **B2B_OUTSOURCING**: The group's questions are answered by the Stift's teachers.
- **EXAMS_ONLY**: Only the exams module is available.

## Create a group

If you want to create a new group, send this `POST` request:

```http
POST /groups
```

### Request

#### Parameters

| Parameter                   | Location | Type                           | Description                                                       | Required |
| --------------------------- | -------- | ------------------------------ | ----------------------------------------------------------------- | -------- |
| **name**                    | Body     | string                         | Group's name                                                      | Yes      |
| **discipline_restrictions** | Body     | [Discipline[]](disciplines.md) | Disciplines the group has access to                               | Yes      |
| **business_model**          | Body     | string                         | Group's [business model](./#business-model)                       | Yes      |
| **copy**                    | Body     | [Copy](#copy)                  | Copy object if you want to base the new group on an existing one. | No       |

##### Copy

If you want to create a new group based on an existing one, you can use the `copy` parameter. It must have the `id` of the group you want to copy. You must also specify what you want to copy from the group. The copied group's parameters will override other specified parameters. The available options are:

- **discipline_restrictions (boolean)**: Copy the disciplines the source group has.
- **teachers (boolean)**: Copy the teachers the source group has.
- **cost_preference (boolean)**: Copy the cost preference the source group has.

###### Copy example

```json
{
  "id": "7033d947-e140-4a83-86b3-cd8ffcb87913",
  "discipline_restrictions": true,
  "teachers": true,
  "cost_preference": true
}
```

#### Example

```json
{
  "name": "Extensivo 2023.2",
  "business_model": "B2B_OUTSOURCING",
  "discipline_restrictions": [
    {
      "id": 1
    },
    {
      "id": 2
    },
    {
      "id": 3
    }
  ],
  "copy": {
    "id": "7033d947-e140-4a83-86b3-cd8ffcb87913",
    "discipline_restrictions": true,
    "teachers": true,
    "cost_preference": true
  }
}
```

### Response

#### Status

| Code | Description                                                               |
| ---- | ------------------------------------------------------------------------- |
| 201  | The group was successfully created.                                       |
| 400  | Some parameter must be malformed or missing. Check the response for more. |
| 401  | Unauthorized.                                                             |

#### Example

```json
{
  "id": "7033d947-e140-4a83-86b3-cd8ffcb87913",
  "name": "Extensivo 2023.2",
  "school": {
    "id": "a6312d26-42c6-45dd-8eeb-5a6f85a1834f"
  },
  "business_model": "B2B_OUTSOURCING",
  "discipline_restrictions": [
    {
      "id": 1
    },
    {
      "id": 2
    },
    {
      "id": 3
    }
  ],
  "created_at": "2023-08-10T19:21:09.026Z"
}
```

## List groups

If you want to list all groups, send this `GET` request:

```http
GET /groups
```

### Request

#### Parameters

None.

### Response

#### Status

| Code | Description                         |
| ---- | ----------------------------------- |
| 200  | The list was retrieved succesfully. |
| 401  | Unauthorized.                       |

#### Example

```json
[
  {
    "id": "7033d947-e140-4a83-86b3-cd8ffcb87913",
    "name": "Extensivo 2023.2",
    "school": {
      "id": "a6312d26-42c6-45dd-8eeb-5a6f85a1834f",
      "name": "Demo School"
    },
    "supergroup": {
      "name": "Turmas"
    },
    "students_count": 0,
    "teachers_count": 0,
    "total_questions_count": 0,
    "answered_questions_count": 0,
    "error_questions_count": 0
  },
  {
    "id": "6f3a7df1-5f30-4122-8df2-2a54f2a32c51",
    "name": "Intensivo 2023.2",
    "school": {
      "id": "a6312d26-42c6-45dd-8eeb-5a6f85a1834f",
      "name": "Demo School"
    },
    "supergroup": {
      "name": "Turmas"
    },
    "students_count": 0,
    "teachers_count": 0,
    "total_questions_count": 0,
    "answered_questions_count": 0,
    "error_questions_count": 0
  }
]
```

## Get group

If you want to get a specific group, send this `GET` request:

```http
GET /groups/:id
```

### Request

#### Parameters

| Parameter | Location      | Type   | Description | Required |
| --------- | ------------- | ------ | ----------- | -------- |
| **id**    | URL Parameter | string | Group's id  | Yes      |

#### Example

```http
GET /groups/7033d947-e140-4a83-86b3-cd8ffcb87913
```

### Response

#### Status

| Code | Description                          |
| ---- | ------------------------------------ |
| 200  | The group was retrieved succesfully. |
| 401  | Unauthorized.                        |

#### Example

```json
{
  "id": "7033d947-e140-4a83-86b3-cd8ffcb87913",
  "name": "Extensivo 2023.2",
  "students": [],
  "teachers": []
}
```
