# Users

Users are the central part of the application. They can be created, edited and deleted. They can also be assigned to groups and roles.

### Roles

Roles determine what a user can do in the application. They are defined by a name and a set of permissions.

- **STUDENT**: Can only access the student's app and view their own data.
- **TEACHER**: Can only access the teacher's app, answer questions from the groups they are part of and view their own data.
- **GROUP_ADMIN**: Can only access the group admin's app and view their all data from the groups they are part of.

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

| Parameter  | Location | Type   | Description                                                                                                                                           | Required |
| ---------- | -------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **query**  | Query    | string | Search query to filter users by. Searches using vectors determined by the user's first_name, last_name, and email, so any of these are valid queries. | No       |
| **limit**  | Query    | string | The maximum limit of users to return. Defaults to 100.                                                                                                | No       |
| **offset** | Query    | string | The numbers of users to skip. The order is always determined by the user's creation date, and by the closest match to the query, if there is any.     | No       |
| **type**   | Query    | string | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values.                                                                                               | No       |

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

| Parameter       | Location | Type   | Description                                                                                                                                                     | Required |
| --------------- | -------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **first_name**  | Body     | string | User's first name (John)                                                                                                                                        | Yes      |
| **last_name**   | Body     | string | User's last name (Doe)                                                                                                                                          | Yes      |
| **email**       | Body     | string | User's email. **If a user with the same email already exists, this endpoint will act as a PATCH request, and the user will be unblocked if they were blocked.** | Yes      |
| **type**        | Body     | string | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values                                                                                                          | Yes      |
| **group_ids**   | Body     | array  | List of the group ids the user should be part of                                                                                                                | Yes      |
| **groups_data** | Body     | array  | List of [GroupData](./#groupdata) objects. If not specified, the user will have unlimited questions                                                             | No       |
| **gender**      | Body     | string | Use MASCULINE, FEMININE or OTHER as accepted values                                                                                                             | No       |
| **birth_date**  | Body     | string | User's birth date (2002-11-28)                                                                                                                                  | No       |
| **phone**       | Body     | string | User's phone number (+5548991234567)                                                                                                                            | No       |
| **location**    | Body     | string | The closest known location to the user (Brazil)                                                                                                                 | No       |
| **password**    | Body     | string | The user's password. Minimum length is 8                                                                                                                        | No       |

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

| Parameter        | Location | Type    | Description                                                                                                                                                                                                          | Required |
| ---------------- | -------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| **first_name**   | Body     | string  | User's first name (John)                                                                                                                                                                                             | No       |
| **last_name**    | Body     | string  | User's last name (Doe)                                                                                                                                                                                               | No       |
| **email**        | Body     | string  | User's email                                                                                                                                                                                                         | No       |
| **type**         | Body     | string  | Use STUDENT, TEACHER or GROUP_ADMIN as accepted values                                                                                                                                                               | No       |
| **blocked**      | Body     | boolean | Determines if the user should be blocked from accessing the app or not. If blocked, they will not be able to perform any actions, but their data will be stored for a defined period of time until they are deleted. | No       |
| **group_ids**    | Body     | array   | List of the group ids the user should be part of                                                                                                                                                                     | No       |
| **groups_data**  | Body     | array   | List of [GroupData](./#groupdata) objects. If not specified, the user will have unlimited questions                                                                                                                  | No       |
| **gender**       | Body     | string  | Use MASCULINE, FEMININE or OTHER as accepted values                                                                                                                                                                  | No       |
| **birth_date**   | Body     | string  | User's birth date (2002-11-28)                                                                                                                                                                                       | No       |
| **phone**        | Body     | string  | User's phone number (+5548991234567)                                                                                                                                                                                 | No       |
| **location**     | Body     | string  | The closest known location to the user (Brazil)                                                                                                                                                                      | No       |
| **new_password** | Body     | string  | The user's updated password. Minimum length is 8. The password can only be updated if the user is 100% owned by your API Key, which means, your API key has access to all the user's groups.                          | No       |

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
  "created_at": "2024-12-05T08:37:41.811Z"
}
```
