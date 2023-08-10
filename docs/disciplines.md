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
