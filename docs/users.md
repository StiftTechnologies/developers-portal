# Users

Users are the central part of the application. They can be created, edited and deleted. They can also be assigned to groups and roles.

### Roles

Roles determine what a user can do in the application. They are defined by a name and a set of permissions.

- **STUDENT**: Can only access the student's app and view their own data.
- **TEACHER**: Can only access the teacher's app, answer questions from the groups they are part of and view their own data.
- **GROUP_ADMIN**: Can only access the group admin's app and view their all data from the groups they are part of.

### Teacher access

Two fields control what a teacher can do. You can send them in the [create](#create-a-user) and [patch](#patch-a-user) requests of a TEACHER user.

- `portal_access`: If `true`, the teacher can also sign in to the Portal with read-only access. The default is `false`.
- `can_answer`: If `false`, the teacher stays in their groups but cannot answer questions. The default is `true`.

These rules apply to both fields:

- The fields are only for TEACHER users. If you send `portal_access: true` or `can_answer: false` for another type, the API returns 400.
- Only an API key that belongs to a group admin can change `portal_access`. With another API key, `portal_access: true` returns 400 and `portal_access: false` has no effect.
- If you do not send a field, its current value does not change.

To sign a teacher in to the Portal, generate a login code with the `PORTAL` target. See [Auth Integration](auth_integration.md#generate-a-login-code).

### Groups

Groups are the way to organize users, as in classes. Once users are inside a group, they can only see the data that is related to that group. A user can be in multiple groups at a time.

<!-- See more about groups [here](groups.md). -->

#### GroupData

The GroupData object is used to save information about the relation of a user with a group. It's primary use is the amount of remaining_questions a user has in a group.

The value of remaining_questions is the amount of questions a user can send to a group.

**To allow a user to send unlimited questions in a group, set `remaining_questions` to -1.**
If the GroupData for a group is not specified, the user will have unlimited questions in that group.

```json title="GroupData"
{
  "group": {
    "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
  },
  "remaining_questions": -1
}
```

## Search users

If you want to search for users, send this `GET` request. This endpoint implements search functionality and pagination:

```http
GET /users
```

### Request

#### Parameters

| Parameter         | Location | Type    | Description                                                                                                                                                               | Required |
| ----------------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **query**         | Query    | string  | Search query to filter users by. Searches using vectors determined by the user's first_name, last_name, and email, so any of these are valid queries.                     | No       |
| **limit**         | Query    | string  | The maximum limit of users to return. Defaults to 100.                                                                                                                    | No       |
| **offset**        | Query    | string  | The numbers of users to skip. The order is always determined by the user's creation date, and by the closest match to the query, if there is any.                         | No       |
| **type**          | Query    | string  | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values.                                                                                                                   | No       |
| **discipline_id** | Query    | int     | Search teachers by a [discipline id](disciplines.md).                                                                                                                     | No       |
| **group_ids**     | Query    | string  | Search users by the groups they are part of, with a comma separated list of group ids. Example: bee1b51e-1843-443b-8bd2-9c46c86373c5,d7e595ef-cdfa-406c-81af-227fde165309 | No       |
| **blocked**       | Query    | boolean | Set this parameter to true to search only for blocked users.                                                                                                              | No       |

### Response

#### Status

| Code | Description                                                               |
| ---- | ------------------------------------------------------------------------- |
| 200  | The search was successful. Returns a list of matched users.               |
| 400  | Some parameter must be malformed or missing. Check the response for more. |
| 401  | Unauthorized.                                                             |

#### Example

```json
[
  {
    "id": "d7e595ef-cdfa-406c-81af-227fde165309",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@stift.com.br",
    "type": "STUDENT",
    "profile_photo_url": "https://sa-east-1-public-cdn.stift.com.br/resources/images/placeholders/profile_photo_placeholder.png",
    "created_at": "2024-12-05T08:37:41.811Z"
  }
]
```

## Create a user

If you want to create a new user, send this `POST` request:

```http
POST /users
```

### Request

#### Parameters

| Parameter           | Location | Type    | Description                                                                                                                                                     | Required |
| ------------------- | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **first_name**      | Body     | string  | User's first name (John)                                                                                                                                        | Yes      |
| **last_name**       | Body     | string  | User's last name (Doe)                                                                                                                                          | Yes      |
| **email**           | Body     | string  | User's email. **If a user with the same email already exists, this endpoint will act as a PATCH request, and the user will be unblocked if they were blocked.** | Yes      |
| **type**            | Body     | string  | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values                                                                                                          | Yes      |
| **group_ids**       | Body     | array   | List of the group ids the user should be part of. Optional for GROUP_ADMIN users                                                                                | Yes\*    |
| **groups_data**     | Body     | array   | List of [GroupData](#groupdata) objects, with at least one item. If a group of the user is not in the list, the user can send unlimited questions in that group | Yes\*    |
| **discipline_ids**  | Body     | array   | List of the [discipline ids](disciplines.md) that the teacher answers                                                                                           | Yes\*    |
| **school_ids**      | Body     | array   | List of the school ids that the group admin manages. Only for GROUP_ADMIN users. Your API key's user must also manage each school                               | No       |
| **teacher_details** | Body     | object  | Only for TEACHER users. Accepts `has_undergraduate_degree` (boolean) and `admission_date` (2024-02-01)                                                          | No       |
| **portal_access**   | Body     | boolean | Only for TEACHER users. If `true`, the teacher can also use the Portal. The default is `false`. See [Teacher access](#teacher-access)                           | No       |
| **can_answer**      | Body     | boolean | Only for TEACHER users. If `false`, the teacher cannot answer questions. The default is `true`. See [Teacher access](#teacher-access)                           | No       |
| **gender**          | Body     | string  | Use MASCULINE, FEMININE or OTHER as accepted values                                                                                                             | No       |
| **birth_date**      | Body     | string  | User's birth date (2002-11-28)                                                                                                                                  | No       |
| **phone**           | Body     | string  | User's phone number (+5548991234567)                                                                                                                            | No       |
| **location**        | Body     | string  | The closest known location to the user (Brazil)                                                                                                                 | No       |
| **password**        | Body     | string  | The user's password. Minimum length is 6                                                                                                                        | No       |

\* `group_ids` is required for STUDENT and TEACHER users. `groups_data` is required for STUDENT users. `discipline_ids` is required for TEACHER users.

#### Example

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@stift.com.br",
  "location": "Santa Catarina, Brasil",
  "gender": "MASCULINE",
  "type": "STUDENT",
  "group_ids": ["bee1b51e-1843-443b-8bd2-9c46c86373c5"],
  "groups_data": [
    {
      "group": {
        "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
      },
      "remaining_questions": -1
    }
  ],
  "password": "12345678"
}
```

This example creates a teacher who can also use the Portal:

```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@stift.com.br",
  "type": "TEACHER",
  "group_ids": ["bee1b51e-1843-443b-8bd2-9c46c86373c5"],
  "discipline_ids": [1, 4],
  "teacher_details": {
    "has_undergraduate_degree": true,
    "admission_date": "2024-02-01"
  },
  "portal_access": true,
  "can_answer": true
}
```

### Response

#### Status

| Code | Description                                                               |
| ---- | ------------------------------------------------------------------------- |
| 201  | The user was successfully created.                                        |
| 400  | Some parameter must be malformed or missing. Check the response for more. |
| 401  | Unauthorized.                                                             |

#### Example

```json
{
  "id": "d7e595ef-cdfa-406c-81af-227fde165309",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@stift.com.br",
  "phone": null,
  "location": "Santa Catarina, Brasil",
  "gender": "MASCULINE",
  "birth_date": null,
  "type": "STUDENT",
  "groups": [
    {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5",
      "name": "Class 1",
      "school": {
        "id": "5207bb23-27df-45d8-9dc9-767c8a65640b"
      }
    }
  ],
  "groups_data": [
    {
      "group": {
        "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
      },
      "remaining_questions": -1
    }
  ],
  "portal_access": false,
  "password": "12345678",
  "created_at": "2024-12-05T08:37:41.811Z"
}
```

## Patch a user

If you want to update an existing user, send this `PATCH` request. You only need to send the parameters you want to update:

```http
PATCH /users/:id
```

### Request

#### Parameters

| Parameter          | Location | Type    | Description                                                                                                                                                                                                          | Required |
| ------------------ | -------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **first_name**     | Body     | string  | User's first name (John)                                                                                                                                                                                             | No       |
| **last_name**      | Body     | string  | User's last name (Doe)                                                                                                                                                                                               | No       |
| **email**          | Body     | string  | User's email                                                                                                                                                                                                         | No       |
| **type**           | Body     | string  | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values                                                                                                                                                               | No       |
| **blocked**        | Body     | boolean | Determines if the user should be blocked from accessing the app or not. If blocked, they will not be able to perform any actions, but their data will be stored for a defined period of time until they are deleted. | No       |
| **group_ids**      | Body     | array   | List of the group ids the user should be part of                                                                                                                                                                     | No       |
| **groups_data**    | Body     | array   | List of [GroupData](#groupdata) objects. If not specified, the user will have unlimited questions                                                                                                                    | No       |
| **discipline_ids** | Body     | array   | List of the [discipline ids](disciplines.md) that the teacher answers. Replaces the current list                                                                                                                     | No       |
| **school_ids**     | Body     | array   | List of the school ids that the group admin manages. Only for GROUP_ADMIN users. Your API key's user must also manage each school                                                                                    | No       |
| **portal_access**  | Body     | boolean | Only for TEACHER users. If `true`, the teacher can also use the Portal. See [Teacher access](#teacher-access)                                                                                                        | No       |
| **can_answer**     | Body     | boolean | Only for TEACHER users. If `false`, the teacher cannot answer questions. See [Teacher access](#teacher-access)                                                                                                       | No       |
| **gender**         | Body     | string  | Use MASCULINE, FEMININE or OTHER as accepted values                                                                                                                                                                  | No       |
| **birth_date**     | Body     | string  | User's birth date (2002-11-28)                                                                                                                                                                                       | No       |
| **phone**          | Body     | string  | User's phone number (+5548991234567)                                                                                                                                                                                 | No       |
| **location**       | Body     | string  | The closest known location to the user (Brazil)                                                                                                                                                                      | No       |
| **new_password**   | Body     | string  | The user's updated password. Minimum length is 6. The password can only be updated if the user is 100% owned by your API Key, which means, your API key has access to all the user's groups.                         | No       |

#### Example

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@stift.com.br",
  "blocked": true,
  "location": "Santa Catarina, Brasil",
  "gender": "MASCULINE",
  "type": "STUDENT",
  "group_ids": ["bee1b51e-1843-443b-8bd2-9c46c86373c5"],
  "groups_data": [
    {
      "group": {
        "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
      },
      "remaining_questions": -1
    }
  ],
  "new_password": "12345678"
}
```

### Response

#### Status

| Code | Description                                                               |
| ---- | ------------------------------------------------------------------------- |
| 200  | The user was successfully patched.                                        |
| 400  | Some parameter must be malformed or missing. Check the response for more. |
| 401  | Unauthorized.                                                             |

#### Example

```json
{
  "id": "d7e595ef-cdfa-406c-81af-227fde165309",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@stift.com.br",
  "phone": null,
  "location": "Santa Catarina, Brasil",
  "gender": "MASCULINE",
  "birth_date": null,
  "type": "STUDENT",
  "groups": [
    {
      "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5",
      "name": "Class 1",
      "school": {
        "id": "5207bb23-27df-45d8-9dc9-767c8a65640b"
      }
    }
  ],
  "groups_data": [
    {
      "group": {
        "id": "bee1b51e-1843-443b-8bd2-9c46c86373c5"
      },
      "remaining_questions": -1
    }
  ],
  "blocked": true,
  "portal_access": false,
  "created_at": "2024-12-05T08:37:41.811Z"
}
```
