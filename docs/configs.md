# Configs

The configs endpoint returns bootstrap data used across the platform: disciplines and subjects, localized statuses and answer preferences, optional user-data forms, sticker packs, and the current server time.

As a **GROUP_ADMIN**, disciplines are scoped to your groups and schools (via group relations, doubts in your groups, and school relations). Sticker packs and user-data sessions are scoped to your groups.

Localized strings (`statuses`, `answer_preferences`) use the request locale from the **`X-Locale`** header (e.g. `pt_BR`, `en_US`, `es_CL`). When omitted or unsupported, **`pt_BR`** is used.

For a flat list of all disciplines without other config data, see [Disciplines](disciplines.md).

## Get configs

If you want to load application configs, send this `GET` request:

```http
GET /configs
```

### Request

#### Parameters

| Parameter | Location | Type | Description | Required |
| ----- | ----- | --- | ----- | ----- |
| **X-Locale** | Header | string | Locale for localized labels (`pt_BR`, `en_US`, `es_CL`). Defaults to `pt_BR`. | No |

#### Example

```http
GET /configs
X-Locale: pt_BR
```

### Response

#### Status

| Code | Description |
| ---- | ---- |
| 200 | Configs were returned successfully. |
| 401 | Unauthorized. |

#### Example

```json
{
  "disciplines": [
    {
      "id": 1,
      "name": "Matemática",
      "internal_name": "mathematics",
      "icon_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/icons/ic_mathematics.png",
      "header_url": "https://sa-east-1-public-cdn.stift.com.br/configs/disciplines/headers/header_mathematics.jpg",
      "subjects": [
        {
          "id": 10,
          "name": "Álgebra",
          "icon_url": "https://..."
        }
      ],
      "enabled": true,
      "remaining_questions": 1
    }
  ],
  "subjects": [
    {
      "id": 10,
      "name": "Álgebra",
      "icon_url": "https://..."
    }
  ],
  "statuses": [
    {
      "id": 3,
      "name_id": 3,
      "icon_url": "https://...",
      "description_id": 3,
      "message_student": true,
      "message_school": false,
      "contains_text": false,
      "name": "Aguardando resposta",
      "description": "Aguardando resposta"
    }
  ],
  "answer_preferences": [
    {
      "id": 6,
      "name": "Passo a passo"
    }
  ],
  "user_data_sessions": [
    {
      "id": 1,
      "title": "Perfil do estudante",
      "user_data_fields": [
        {
          "id": 1,
          "name": "Série",
          "short_name": "grade",
          "selection_limit": 1,
          "required": true,
          "options": [
            {
              "id": 1,
              "name": "1º ano"
            }
          ]
        }
      ]
    }
  ],
  "sticker_packs": [
    {
      "id": 1,
      "name": "Default",
      "stickers": [
        {
          "id": 1,
          "name": "thumbs_up",
          "url": "https://..."
        }
      ]
    }
  ],
  "current_time": 1749902400000
}
```

#### Response fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| **disciplines** | array | Disciplines available to the authenticated user. Each item includes nested **subjects**, **enabled** (`true` for GROUP_ADMIN), and **remaining_questions** (placeholder `1` in this response). |
| **subjects** | array | Flat list of all subjects from the returned disciplines. |
| **statuses** | array | Question/doubt status catalog with localized **name** and **description**. Use **id** when filtering (e.g. `status_id` on [List questions](questions.md#list-questions)). |
| **answer_preferences** | array | Enabled answer preference options with localized **name**. Use **id** when creating a [doubt](doubts.md#create-a-doubt). |
| **user_data_sessions** | array | User-data collection forms scoped to your groups and user type. Empty when you have no groups. |
| **sticker_packs** | array | Sticker packs linked to your groups. Empty when you have no groups. |
| **current_time** | int | Server time in milliseconds since the UNIX epoch. |
