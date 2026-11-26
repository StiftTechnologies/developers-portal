# Extracurricular Activities

Extracurricular activities are work that teachers do outside of answering questions, for example a study session or a science fair. A teacher or a coordinator records each activity with a type, a start time and an end time. Schools use these records to see and pay for this work.

This page shows how to manage categories and types, and how to record, list, change and delete activities. To export activities to a file, see the [Extracurricular Activities Report](reports.md#extracurricular-activities-report).

All paths on this page start with `/extracurricular-activities`.

## Concepts

- Category: a group of activity types, for example "Plantão". A category belongs to one or more schools. Only teachers of these schools can record activities in the category.
- Type: one kind of activity in a category, for example "Plantão de dúvidas". Each activity has one type. If a type has `requires_description` set to `true`, each activity of this type must have a description.
- Activity: one period of work of one teacher in one school. It has a type, a `start_date`, an `end_date` and an optional `description`.
- Disabled: a category or type that you turn off with `disabled: true`. You cannot record new activities with it. You can turn it on again.
- Deleted: a category or type that you delete. You cannot use it again. The activities that use it stay, and keep its name.

### Module

The `EXTRACURRICULAR_ACTIVITIES` module must be on for the groups of the API key's user. Contact us to turn it on. If the user has their own list of modules, that list must also contain `EXTRACURRICULAR_ACTIVITIES`. Without the module, all requests on this page return 403.

### Who can call each request

Your [API key](index.md#api-keys) acts as the user that it belongs to. The type of that user sets what you can do:

| Request                                         | ADMIN | GROUP_ADMIN | TEACHER              |
| ----------------------------------------------- | ----- | ----------- | -------------------- |
| Get categories                                  | Yes   | Yes         | Yes                  |
| Create, change or delete a category or a type   | Yes   | Yes         | No                   |
| Get activities, get an activity                 | Yes   | Yes         | Yes, only their own  |
| Create an activity                              | Yes   | Yes         | Yes, only for themselves |
| Change or delete an activity                    | Yes   | Yes         | No                   |

Other user types get 403.

### Access rules

The API checks these rules on each request:

- A GROUP_ADMIN user sees only the schools that they manage and the schools of their groups. A TEACHER user sees only the schools of their groups that have the module.
- A GROUP_ADMIN user sees only the activities of teachers that are in one of their groups or schools. Activities of teachers with the `OUTSOURCING` business model are not visible to GROUP_ADMIN users.
- A TEACHER user sees only their own activities.
- If an activity, category, type, school or teacher is not visible to you, the API returns 404. It does not return 403.

A category can belong to schools that you do not manage. You can see the category, but its `schools` list contains only your schools and `can_manage` is `false`. To change or delete a category, or to add a type to it, you must manage all of its schools. Otherwise, the API returns 403.

### Teachers who can get activities

To get a new activity in a school, a teacher must meet all of these conditions:

- The school is one of the schools of the category.
- The teacher is in a group of that school, and the module is on for that group. If the teacher has their own list of modules, that list contains the module.
- The teacher is not blocked on the platform, and has no active block in that group or school.

If the teacher has more than one such school, the activity goes to the school of the teacher's oldest group, unless you send `school_id`. A blocked teacher cannot get new activities, but you can still change and delete their old activities.

## Get categories

To get the categories of your schools, send this `GET` request:

```http
GET /extracurricular-activities/categories
```

The categories and their types are in alphabetical order. Deleted categories and types are not in the list.

The response has two views:

- Manager view: for ADMIN and GROUP_ADMIN users that do not send `teacher_id`. It contains all categories of your schools.
- Teacher view: for TEACHER users, and for managers that send `teacher_id`. It contains only categories that the teacher can use now: the category and at least one of its types are not disabled, and the teacher can get activities in one of its schools. Disabled types are not in the list. The API ignores `include_disabled`. Each category has `eligible_school_ids`.

Use the teacher view to show the choices before you [create an activity](#create-an-activity) for a teacher.

### Request

#### Parameters

| Parameter            | Location | Type     | Description                                                                                                                              | Required |
| -------------------- | -------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **include_types**    | Query    | boolean  | If `true`, each category includes a `types` list. The default is `false`.                                                                | No       |
| **include_disabled** | Query    | boolean  | If `true`, the list also contains disabled categories and types. The default is `false`. Only for the manager view.                      | No       |
| **teacher_id**       | Query    | string   | The id of a teacher. If you send it, you get the teacher view for this teacher. A TEACHER user can send only their own id.               | No       |
| **school_ids**       | Query    | string[] | Only categories of these schools. Send the ids separated by commas, or repeat the parameter. Up to 500 ids.                              | No       |

### Response

Each item has these fields:

| Field                   | Type             | Description                                                                                                            |
| ----------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **id**                  | string           | The id of the category.                                                                                                |
| **name**                | string           | The name of the category.                                                                                              |
| **disabled**            | boolean          | `true` means that the category is disabled.                                                                            |
| **can_manage**          | boolean          | `true` means that you can change and delete the category and add types to it.                                          |
| **schools**             | array            | The schools of the category that you can see (`id`, `name`).                                                           |
| **types**               | array            | The types of the category (`id`, `name`, `disabled`, `requires_description`). Only with `include_types=true`.           |
| **eligible_school_ids** | array of strings | The schools of the category where the teacher can get activities. The oldest group comes first. Only in the teacher view. |

#### Status

| Code | Description                                                                                   |
| ---- | --------------------------------------------------------------------------------------------- |
| 200  | The categories were fetched.                                                                  |
| 400  | A parameter has a wrong format.                                                               |
| 401  | Unauthorized.                                                                                 |
| 403  | The module is off, the user type cannot use this request, or a TEACHER sent another teacher's id. |
| 404  | `Teacher not found`: the teacher of `teacher_id` is not visible to you.                       |

#### Example

```http
GET /extracurricular-activities/categories?include_types=true
```

```json
[
  {
    "id": "8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b",
    "name": "Plantão",
    "disabled": false,
    "can_manage": true,
    "schools": [
      {
        "id": "d4e5f6a7-b8c9-4d0e-8f1a-2b3c4d5e6f70",
        "name": "School Name"
      }
    ],
    "types": [
      {
        "id": "2c3d4e5f-6a7b-4c8d-9e0f-1a2b3c4d5e6f",
        "name": "Outros",
        "disabled": false,
        "requires_description": true
      },
      {
        "id": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
        "name": "Plantão de dúvidas",
        "disabled": false,
        "requires_description": false
      }
    ]
  }
]
```

## Create a category

To create a category, send this `POST` request:

```http
POST /extracurricular-activities/categories
```

The name of a category must be unique in each of its schools. For this check, the API ignores case and accents. Thus "plantao" and "Plantão" are the same name. Disabled categories count. Deleted categories do not count.

### Request

#### Parameters

| Parameter      | Location | Type     | Description                                                                   | Required |
| -------------- | -------- | -------- | ----------------------------------------------------------------------------- | -------- |
| **name**       | Body     | string   | The name of the category, from 1 to 100 characters. The API removes the spaces at the start and at the end. | Yes      |
| **school_ids** | Body     | string[] | The schools of the category. From 1 to 500 ids. You must manage all of them.  | Yes      |

```json title="Request body"
{
  "name": "Plantão",
  "school_ids": ["d4e5f6a7-b8c9-4d0e-8f1a-2b3c4d5e6f70"]
}
```

### Response

The response contains the new category, with the same fields as in [Get categories](#get-categories). `types` is an empty list.

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 201  | The category was created.                                                                           |
| 400  | A parameter is missing or has a wrong format.                                                       |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, or the user type cannot use this request.                                        |
| 404  | `School not found`: a school does not exist or is not visible to you.                               |
| 409  | `An activity category with this name already exists in one of these schools`.                      |

#### Example

```json
{
  "id": "8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b",
  "name": "Plantão",
  "disabled": false,
  "can_manage": true,
  "schools": [
    {
      "id": "d4e5f6a7-b8c9-4d0e-8f1a-2b3c4d5e6f70",
      "name": "School Name"
    }
  ],
  "types": []
}
```

## Change a category

To rename, disable or enable a category, or to change its schools, send this `PATCH` request:

```http
PATCH /extracurricular-activities/categories/{category_id}
```

Send only the fields that you want to change. If you remove a school from the category, the old activities in that school do not change.

### Request

#### Parameters

| Parameter       | Location | Type     | Description                                                                                       | Required |
| --------------- | -------- | -------- | ------------------------------------------------------------------------------------------------- | -------- |
| **category_id** | Path     | string   | The id of the category.                                                                           | Yes      |
| **name**        | Body     | string   | The new name, from 1 to 100 characters.                                                           | No       |
| **school_ids**  | Body     | string[] | The new full list of schools. From 1 to 500 ids. You must manage all of them.                     | No       |
| **disabled**    | Body     | boolean  | `true` disables the category. `false` enables it.                                                 | No       |

```json title="Request body"
{
  "disabled": true
}
```

### Response

The response contains the category with its `types`, also the disabled types.

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The category was changed.                                                                           |
| 400  | A parameter has a wrong format.                                                                     |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, the user type cannot use this request, or you do not manage all schools of the category. |
| 404  | `Activity category not found`, or `School not found`.                                               |
| 409  | `An activity category with this name already exists in one of these schools`.                      |

## Delete a category

To delete a category and all of its types, send this `DELETE` request:

```http
DELETE /extracurricular-activities/categories/{category_id}
```

The activities of the category stay. They keep the names of the category and the type, and show `"deleted": true`. You cannot undo this request. After the delete, you can create a new category with the same name.

### Request

#### Parameters

| Parameter       | Location | Type   | Description             | Required |
| --------------- | -------- | ------ | ----------------------- | -------- |
| **category_id** | Path     | string | The id of the category. | Yes      |

### Response

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The category was deleted.                                                                           |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, the user type cannot use this request, or you do not manage all schools of the category. |
| 404  | `Activity category not found`.                                                                      |

#### Example

```json
{
  "id": "8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b"
}
```

## Create a type

To add a type to a category, send this `POST` request:

```http
POST /extracurricular-activities/categories/{category_id}/types
```

The name of a type must be unique in its category. For this check, the API ignores case and accents. Disabled types count. Deleted types do not count.

To get the types of a category, use [Get categories](#get-categories) with `include_types=true`.

### Request

#### Parameters

| Parameter                | Location | Type    | Description                                                                              | Required |
| ------------------------ | -------- | ------- | ---------------------------------------------------------------------------------------- | -------- |
| **category_id**          | Path     | string  | The id of the category.                                                                  | Yes      |
| **name**                 | Body     | string  | The name of the type, from 1 to 100 characters.                                          | Yes      |
| **requires_description** | Body     | boolean | If `true`, each activity of this type must have a description. The default is `false`.  | No       |

```json title="Request body"
{
  "name": "Outros",
  "requires_description": true
}
```

### Response

| Field                    | Type    | Description                                         |
| ------------------------ | ------- | --------------------------------------------------- |
| **id**                   | string  | The id of the type.                                 |
| **name**                 | string  | The name of the type.                               |
| **disabled**             | boolean | `true` means that the type is disabled.             |
| **requires_description** | boolean | `true` means that activities of this type need a description. |
| **category_id**          | string  | The id of the category.                             |

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 201  | The type was created.                                                                               |
| 400  | A parameter is missing or has a wrong format.                                                       |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, the user type cannot use this request, or you do not manage all schools of the category. |
| 404  | `Activity category not found`.                                                                      |
| 409  | `An activity type with this name already exists in this category`.                                  |

#### Example

```json
{
  "id": "2c3d4e5f-6a7b-4c8d-9e0f-1a2b3c4d5e6f",
  "name": "Outros",
  "disabled": false,
  "requires_description": true,
  "category_id": "8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b"
}
```

## Change a type

To rename, disable or enable a type, or to change `requires_description`, send this `PATCH` request:

```http
PATCH /extracurricular-activities/types/{type_id}
```

Send only the fields that you want to change. A change to `requires_description` does not change old activities. When you change the type or the description of an old activity, the API checks the new value.

### Request

#### Parameters

| Parameter                | Location | Type    | Description                                              | Required |
| ------------------------ | -------- | ------- | -------------------------------------------------------- | -------- |
| **type_id**              | Path     | string  | The id of the type.                                      | Yes      |
| **name**                 | Body     | string  | The new name, from 1 to 100 characters.                  | No       |
| **disabled**             | Body     | boolean | `true` disables the type. `false` enables it.            | No       |
| **requires_description** | Body     | boolean | If `true`, activities of this type need a description.   | No       |

### Response

The response contains the type, with the same fields as in [Create a type](#create-a-type).

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The type was changed.                                                                               |
| 400  | A parameter has a wrong format.                                                                     |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, the user type cannot use this request, or you do not manage all schools of the category. |
| 404  | `Activity type not found`: the type or its category is deleted, or is not visible to you.           |
| 409  | `An activity type with this name already exists in this category`.                                  |

## Delete a type

To delete a type, send this `DELETE` request:

```http
DELETE /extracurricular-activities/types/{type_id}
```

The activities of the type stay. They keep its name and show `"deleted": true` in `type`. You cannot undo this request.

### Request

#### Parameters

| Parameter   | Location | Type   | Description         | Required |
| ----------- | -------- | ------ | ------------------- | -------- |
| **type_id** | Path     | string | The id of the type. | Yes      |

### Response

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The type was deleted.                                                                               |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, the user type cannot use this request, or you do not manage all schools of the category. |
| 404  | `Activity type not found`.                                                                          |

#### Example

```json
{
  "id": "2c3d4e5f-6a7b-4c8d-9e0f-1a2b3c4d5e6f"
}
```

## The activity object

The requests below return activities with these fields:

| Field           | Type           | Description                                                                                                         |
| --------------- | -------------- | ------------------------------------------------------------------------------------------------------------------- |
| **id**          | string         | The id of the activity.                                                                                             |
| **description** | string \| null | The description of the activity.                                                                                    |
| **start_date**  | string         | The start of the activity, in ISO 8601 format and UTC.                                                              |
| **end_date**    | string         | The end of the activity, in ISO 8601 format and UTC.                                                                |
| **created_at**  | string         | When the activity was recorded.                                                                                     |
| **updated_at**  | string         | When the activity was last changed.                                                                                 |
| **school**      | object         | The school of the activity (`id`, `name`).                                                                          |
| **category**    | object         | The category (`id`, `name`, `disabled`, `deleted`).                                                                 |
| **type**        | object         | The type (`id`, `name`, `disabled`, `deleted`, `requires_description`).                                             |
| **teacher**     | object         | The teacher that did the activity (`id`, `first_name`, `last_name`, `email`).                                       |
| **created_by**  | object         | The user that recorded the activity (`id`, `first_name`, `last_name`). This can be the teacher or a coordinator.    |

`category` and `type` show the values of today, not the values at the time of the activity. If someone disabled or deleted them later, `disabled` or `deleted` is `true`.

```json title="Activity"
{
  "id": "5e6f7a8b-9c0d-4e1f-8a2b-3c4d5e6f7a8b",
  "description": "Review of chapter 4 exercises",
  "start_date": "2026-11-18T13:00:00.000Z",
  "end_date": "2026-11-18T15:00:00.000Z",
  "created_at": "2026-11-18T15:10:42.318Z",
  "updated_at": "2026-11-18T15:10:42.318Z",
  "school": {
    "id": "d4e5f6a7-b8c9-4d0e-8f1a-2b3c4d5e6f70",
    "name": "School Name"
  },
  "category": {
    "id": "8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b",
    "name": "Plantão",
    "disabled": false,
    "deleted": false
  },
  "type": {
    "id": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
    "name": "Plantão de dúvidas",
    "disabled": false,
    "deleted": false,
    "requires_description": false
  },
  "teacher": {
    "id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
    "first_name": "Teacher's first name",
    "last_name": "Teacher's last name",
    "email": "teacher@stift.com.br"
  },
  "created_by": {
    "id": "3b4c5d6e-7f8a-4b9c-8d0e-1f2a3b4c5d6e",
    "first_name": "Coordinator's first name",
    "last_name": "Coordinator's last name"
  }
}
```

## Get activities

To get the activities that are visible to you, send this `GET` request:

```http
GET /extracurricular-activities
```

The newest activities come first (by `start_date`). The response is paginated. The `X-Total-Count` response header contains the number of activities that match the filters, not only the number on this page.

### Request

#### Parameters

| Parameter        | Location | Type                     | Description                                                                                                | Required |
| ---------------- | -------- | ------------------------ | ---------------------------------------------------------------------------------------------------------- | -------- |
| **start_date**   | Query    | Milliseconds since epoch | Only activities that end after this time.                                                                  | No       |
| **end_date**     | Query    | Milliseconds since epoch | Only activities that start before this time.                                                               | No       |
| **teacher_ids**  | Query    | string[]                 | Only activities of these teachers.                                                                         | No       |
| **school_ids**   | Query    | string[]                 | Only activities in these schools.                                                                          | No       |
| **category_ids** | Query    | string[]                 | Only activities of these categories.                                                                       | No       |
| **type_ids**     | Query    | string[]                 | Only activities of these types.                                                                            | No       |
| **group_ids**    | Query    | string[]                 | Only activities of teachers that are in one of these groups now.                                           | No       |
| **limit**        | Query    | integer                  | The maximum number of activities to return, from 1 to 1000. The default is `100`.                          | No       |
| **offset**       | Query    | integer                  | The number of activities to skip. The default is `0`.                                                      | No       |

With `start_date` and `end_date`, you get all activities that overlap this period. An activity that starts before `start_date` and ends after it is in the list.

For the id lists, send the ids separated by commas, or repeat the parameter. Each list can have up to 500 ids. Each id must be a UUID.

If a TEACHER user filters by other teachers, the list is empty.

```http
GET /extracurricular-activities?start_date=1793502000000&end_date=1796094000000&category_ids=8a2f4c1e-5b3d-4e6f-9a7b-1c2d3e4f5a6b&limit=50
```

### Response

The response is a list of [activity objects](#the-activity-object).

#### Headers

| Header            | Description                                                         |
| ----------------- | ------------------------------------------------------------------- |
| **X-Total-Count** | The number of activities that match the filters, on all pages.      |

#### Status

| Code | Description                                                     |
| ---- | --------------------------------------------------------------- |
| 200  | The activities were fetched.                                    |
| 400  | A parameter has a wrong format, or `limit` is more than 1000.   |
| 401  | Unauthorized.                                                   |
| 403  | The module is off, or the user type cannot use this request.    |

## Get an activity

To get one activity, send this `GET` request:

```http
GET /extracurricular-activities/{activity_id}
```

### Request

#### Parameters

| Parameter       | Location | Type   | Description             | Required |
| --------------- | -------- | ------ | ----------------------- | -------- |
| **activity_id** | Path     | string | The id of the activity. | Yes      |

### Response

The response is an [activity object](#the-activity-object).

#### Status

| Code | Description                                                                           |
| ---- | ------------------------------------------------------------------------------------- |
| 200  | The activity was fetched.                                                             |
| 400  | `activity_id` is not a UUID.                                                          |
| 401  | Unauthorized.                                                                         |
| 403  | The module is off, or the user type cannot use this request.                          |
| 404  | `Extracurricular activity not found`: the activity is deleted or not visible to you.  |

## Create an activity

To record an activity, send this `POST` request:

```http
POST /extracurricular-activities
```

A TEACHER user records activities only for themselves. An ADMIN or GROUP_ADMIN user must send `teacher_id`. The teacher must be able to get activities in a school of the category. See [Teachers who can get activities](#teachers-who-can-get-activities).

### Request

#### Parameters

| Parameter       | Location | Type                     | Description                                                                                                                                          | Required |
| --------------- | -------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **type_id**     | Body     | string                   | The id of the type. The type and its category must not be disabled.                                                                                  | Yes      |
| **start_date**  | Body     | Milliseconds since epoch | The start of the activity.                                                                                                                           | Yes      |
| **end_date**    | Body     | Milliseconds since epoch | The end of the activity. It must be after `start_date`.                                                                                              | Yes      |
| **teacher_id**  | Body     | string                   | The id of the teacher. Required for ADMIN and GROUP_ADMIN users. A TEACHER user can leave it out or send their own id.                               | Yes\*    |
| **school_id**   | Body     | string                   | The school of the activity. Use an id from `eligible_school_ids` in [Get categories](#get-categories). If you do not send it, the API uses the school of the teacher's oldest group. | No       |
| **description** | Body     | string                   | A description of the activity, up to 5000 characters. For a type with `requires_description` set to `true`, this field is required. A text with only spaces counts as no description. | No       |

```json title="Request body"
{
  "type_id": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
  "teacher_id": "f73c6e53-cdca-4f01-b64d-04b9346c9823",
  "start_date": 1795006800000,
  "end_date": 1795014000000,
  "description": "Review of chapter 4 exercises"
}
```

### Response

The response is the new [activity object](#the-activity-object).

#### Status

| Code | Description                                                                                                                      |
| ---- | -------------------------------------------------------------------------------------------------------------------------------- |
| 201  | The activity was recorded.                                                                                                       |
| 400  | A parameter is missing or has a wrong format, or the description is missing for a type that requires it.                         |
| 400  | `teacher_id is required`.                                                                                                        |
| 400  | `end_date` is not after `start_date`. `errors[0].msg` is `end_date must be after start_date`.                                    |
| 400  | `This activity type or its category is disabled`.                                                                                |
| 400  | `This activity category is not available to the teacher`: the teacher cannot get activities in any school of the category, for example because the teacher is blocked. |
| 400  | `school_id is not a school of this category where the teacher has the module`.                                                  |
| 401  | Unauthorized.                                                                                                                    |
| 403  | The module is off, the user type cannot use this request, or a TEACHER sent another teacher's id.                                |
| 404  | `Activity type not found`: the type is deleted, or its category is not visible to you.                                           |
| 404  | `Teacher not found`: the teacher is not visible to you.                                                                          |

If the description is missing for a type that requires it, `errors` names the `description` parameter:

```json title="Response Code: 400 Bad Request"
{
  "error": {
    "code": 400,
    "message": "Missing or malformed required parameters:\n- description is required for this activity type: null",
    "errors": [
      {
        "value": null,
        "msg": "description is required for this activity type",
        "param": "description",
        "location": "body"
      }
    ]
  }
}
```

## Change an activity

To change an activity, send this `PATCH` request:

```http
PATCH /extracurricular-activities/{activity_id}
```

Only ADMIN and GROUP_ADMIN users can change activities. You cannot change the teacher. Send only the fields that you want to change.

These rules apply:

- You can keep the current type. This is also true for a type that is now disabled or deleted. To move the activity to another type, that type must not be disabled.
- If you change the type and do not send `school_id`, the activity keeps its school when that school is in the new category. Otherwise, it goes to the school of the teacher's oldest group in the new category. If the teacher has no school in the new category, the API returns 400.
- You can change activities of blocked teachers.
- If the type requires a description, you cannot remove the description.

### Request

#### Parameters

| Parameter       | Location | Type                     | Description                                                                                       | Required |
| --------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------- | -------- |
| **activity_id** | Path     | string                   | The id of the activity.                                                                           | Yes      |
| **type_id**     | Body     | string                   | The id of the new type.                                                                           | No       |
| **school_id**   | Body     | string                   | The id of the new school. It must be a school of the category where the teacher has the module.   | No       |
| **description** | Body     | string \| null           | The new description, up to 5000 characters. Send `null` or `""` to remove it.                     | No       |
| **start_date**  | Body     | Milliseconds since epoch | The new start.                                                                                    | No       |
| **end_date**    | Body     | Milliseconds since epoch | The new end. It must be after the start.                                                          | No       |

```json title="Request body"
{
  "end_date": 1795014000000,
  "description": "Review of chapter 4 and 5 exercises"
}
```

### Response

The response is the changed [activity object](#the-activity-object).

#### Status

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The activity was changed.                                                                           |
| 400  | A parameter has a wrong format, or the description is missing for a type that requires it.         |
| 400  | The end is not after the start. If you send both dates, `errors[0].msg` is `end_date must be after start_date`. If you send one date, the `message` is `end_date must be after start_date`. |
| 400  | `This activity type or its category is disabled`: the new type or its category is disabled.         |
| 400  | `This activity category is not available to the teacher`: the teacher has no school in the category of the new type. |
| 400  | `school_id is not a school of this category where the teacher has the module`.                      |
| 401  | Unauthorized.                                                                                       |
| 403  | The module is off, or the user type cannot use this request.                                        |
| 404  | `Extracurricular activity not found`.                                                               |
| 404  | `Activity type not found`: the new type is deleted or not visible to you.                           |

## Delete an activity

To delete an activity, send this `DELETE` request:

```http
DELETE /extracurricular-activities/{activity_id}
```

Only ADMIN and GROUP_ADMIN users can delete activities. A deleted activity is not in the list or in the report. You cannot undo this request.

### Request

#### Parameters

| Parameter       | Location | Type   | Description             | Required |
| --------------- | -------- | ------ | ----------------------- | -------- |
| **activity_id** | Path     | string | The id of the activity. | Yes      |

### Response

#### Status

| Code | Description                                                   |
| ---- | ------------------------------------------------------------- |
| 200  | The activity was deleted.                                     |
| 401  | Unauthorized.                                                 |
| 403  | The module is off, or the user type cannot use this request.  |
| 404  | `Extracurricular activity not found`.                         |

#### Example

```json
{
  "id": "5e6f7a8b-9c0d-4e1f-8a2b-3c4d5e6f7a8b"
}
```
