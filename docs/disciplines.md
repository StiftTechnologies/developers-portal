# Disciplines

Disciplines are a key part of the application. They are used to organize questions. Groups can have multiple disciplines, and questions can only be sent to groups that have the same disciplines as the question.

Disciplines also have `subjects`, which are used to organize questions inside a discipline.

## Get all disciplines

If you want to get the list of all disciplines, send this `GET` request:

```http
GET /disciplines
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
    "id": 1,
    "name": "Matemática",
    "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_mathematics.png",
    "header_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/headers/header_mathematics.jpg"
  },
  {
    "id": 2,
    "name": "Física",
    "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_physics.png",
    "header_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/headers/header_physics.jpg"
  },
  {
    "id": 3,
    "name": "Química",
    "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_chemistry.png",
    "header_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/headers/header_chemistry.jpg"
  }
]
```

## Get a discipline

If you want to get a specific discipline, send this `GET` request:

```http

GET /disciplines/:id
```

### Request

#### Parameters

| Parameter | Location      | Type | Description     | Required |
| --------- | ------------- | ---- | --------------- | -------- |
| **id**    | URL Parameter | int  | Discipline's id | Yes      |

#### Example

```http
GET /disciplines/1
``` 

### Response

#### Status

| Code | Description                               |
| ---- | ----------------------------------------- |
| 200  | The discipline was retrieved succesfully. |
| 401  | Unauthorized.                             |

#### Example

```json
{
  "id": 1,
  "name": "Matemática",
  "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_mathematics.png",
  "header_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/headers/header_mathematics.jpg",
  "knowledge_areas": ["MATHEMATICS"],
  "subjects": [
    {
      "id": 1,
      "name": "Álgebra",
      "icon_url": null
    },
    {
      "id": 2,
      "name": "Análise combinatória",
      "icon_url": null
    },
    {
      "id": 4,
      "name": "Análise de gráficos",
      "icon_url": null
    }
  ]
}
```

## Create a discipline

If you want to create a new discipline, send this `POST` request:

```http
POST /disciplines
```

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **name** | Body | string | The discipline's display name. | Yes |
| **internal_name** | Body | string | An internal identifier for the discipline, shown only to admins. | No |
| **icon** | Body | string | Filename for the discipline icon (e.g., `icon.png`). If provided, returns a presigned upload URL. | No |
| **header** | Body | string | Filename for the discipline header image (e.g., `header.jpg`). If provided, returns a presigned upload URL. | No |
| **subjects** | Body | array | List of subjects to create within the discipline. | No |
| **subjects[].name** | Body | string | The subject's name. | Yes (if subjects provided) |
| **subjects[].enabled** | Body | boolean | Whether the subject is enabled. Defaults to `true`. | No |
| **allow_only_features** | Body | array | List of features allowed for this discipline. If set, restricts which features students can use when asking questions. If not set, all features are allowed. | No |
| **owner_ids** | Body | array | List of school IDs that own this discipline. | Yes |

##### Allowed Features

The `allow_only_features` array can contain the following values:

- **CAPTURE_IMAGE**: Allow students to capture images from their camera.
- **CAPTURE_VIDEO**: Allow students to capture videos from their camera.
- **CAPTURE_AUDIO**: Allow students to capture audio.
- **PICK_IMAGE**: Allow students to pick images from their gallery.
- **PICK_VIDEOS**: Allow students to pick videos from their gallery.
- **PICK_PDFS**: Allow students to attach PDF files.
- **USE_DESCRIPTION**: Allow students to add a text description to questions.
- **ANSWER_PREFERENCES**: Allow students to set answer preferences.
- **ANONYMOUS_MODE**: Allow students to ask questions anonymously.
- **SEND_REPLY**: Allow students to send replies to answers.

#### Example

```json
{
  "name": "Matemática",
  "internal_name": "math_101",
  "icon": "icon.png",
  "header": "header.jpg",
  "subjects": [
    { "name": "Álgebra", "enabled": true },
    { "name": "Geometria", "enabled": true }
  ],
  "allow_only_features": ["CAPTURE_IMAGE", "PICK_IMAGE", "USE_DESCRIPTION"],
  "owner_ids": ["550e8400-e29b-41d4-a716-446655440000"]
}
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 201 | The discipline was created successfully. |
| 401 | Unauthorized. |
| 500 | Internal server error. |

#### Example

```json
{
  "id": 10,
  "name": "Matemática",
  "internal_name": "math_101",
  "icon_url": "",
  "header_url": "",
  "allow_only_features": ["CAPTURE_IMAGE", "PICK_IMAGE", "USE_DESCRIPTION"],
  "owners": [{ "id": "550e8400-e29b-41d4-a716-446655440000" }],
  "icon": {
    "upload": {
      "url": "https://s3.amazonaws.com/...",
      "fields": { ... }
    }
  },
  "header": {
    "upload": {
      "url": "https://s3.amazonaws.com/...",
      "fields": { ... }
    }
  },
  "subjects": [
    { "id": 1, "name": "Álgebra", "enabled": true },
    { "id": 2, "name": "Geometria", "enabled": true }
  ]
}
```

**Note:** If `icon` or `header` filenames are provided, the response includes presigned upload URLs. Use these URLs to upload the actual image files to S3. Once uploaded, the `icon_url` and `header_url` will be updated automatically.

## Update a discipline

If you want to update an existing discipline, send this `PATCH` request:

```http
PATCH /disciplines/:disciplineId
```

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **disciplineId** | URL Parameter | int | The discipline's ID. | Yes |
| **name** | Body | string | The discipline's display name. | No |
| **internal_name** | Body | string | An internal identifier for the discipline. | No |
| **icon** | Body | string | Filename for the discipline icon (e.g., `icon.png`). If provided, returns a presigned upload URL. | No |
| **header** | Body | string | Filename for the discipline header image (e.g., `header.jpg`). If provided, returns a presigned upload URL. | No |
| **allow_only_features** | Body | array | List of features allowed for this discipline. See [Allowed Features](#allowed-features) for available values. | No |
| **owner_ids** | Body | array | List of school IDs that own this discipline. | No |

#### Example

```http
PATCH /disciplines/10
```

```json
{
  "name": "Matemática Avançada",
  "allow_only_features": ["CAPTURE_IMAGE", "PICK_IMAGE", "PICK_PDFS", "USE_DESCRIPTION"]
}
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | The discipline was updated successfully. |
| 401 | Unauthorized. |
| 403 | Forbidden. You don't have permission to edit this discipline. |
| 404 | Discipline not found. |
| 500 | Internal server error. |

#### Example

```json
{
  "id": 10,
  "name": "Matemática Avançada",
  "internal_name": "math_101",
  "icon_url": "https://sa-east-1-public-cdn.stift.com.br/disciplines/10/icons/icon.png",
  "header_url": "https://sa-east-1-public-cdn.stift.com.br/disciplines/10/headers/header.jpg",
  "allow_only_features": ["CAPTURE_IMAGE", "PICK_IMAGE", "PICK_PDFS", "USE_DESCRIPTION"],
  "owners": [{ "id": "550e8400-e29b-41d4-a716-446655440000" }]
}
```
