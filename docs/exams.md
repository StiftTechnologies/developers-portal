# Exams

Exams are assessments that can be created and assigned to groups of students. The exam system includes question management, where questions can be created in a centralized database and then assigned to specific exams and groups.

### Question Types

Questions can be of different types to support various assessment methods:

- **MULTIPLE_CHOICE**: Question with predefined answer options
- **DISCURSIVE**: Open-ended question requiring written response
- **CONTENT**: Informational content without answers

### Question Difficulties

Questions can be assigned difficulty levels to help with proper assessment distribution:

- **VERY_EASY**: Beginner level
- **EASY**: Basic level
- **MEDIUM**: Intermediate level
- **HARD**: Advanced level
- **VERY_HARD**: Expert level

### Group Access Control

Users can only create and update questions and exams for groups they have access to:

## Create Question

If you want to create a new quiz question in the question database, send this `POST` request:

```http
POST /exams/questions-db
```

### Request

#### Parameters

| Parameter                                 | Location | Type             | Description                                                             | Required |
| ----------------------------------------- | -------- | ---------------- | ----------------------------------------------------------------------- | -------- |
| **description**                           | Body     | string           | The question description/text in markdown format                        | Yes      |
| **discipline_id**                         | Body     | number           | ID of the discipline this question belongs to                           | Yes      |
| **subject**                               | Body     | number \| string | Subject ID (existing) or subject name (new)                             | Yes      |
| **type**                                  | Body     | string           | Question type: `MULTIPLE_CHOICE`, `DISCURSIVE`, or `CONTENT`            | Yes      |
| **other_subjects**                        | Body     | array            | Additional subjects related to this question                            | No       |
| **difficulty**                            | Body     | string           | Question difficulty: `VERY_EASY`, `EASY`, `MEDIUM`, `HARD`, `VERY_HARD` | No       |
| **source**                                | Body     | string           | Source of the question (e.g., textbook, exam)                           | Yes      |
| **source_edition**                        | Body     | string           | Edition of the source material                                          | No       |
| **source_edition_observation**            | Body     | string           | Additional observations about the source edition                        | No       |
| **source_question_number**                | Body     | number           | Question number in the source material                                  | No       |
| **solution**                              | Body     | string           | Detailed solution explanation in markdown format                        | No       |
| **study_orientation**                     | Body     | string           | Study guidance for the question in markdown format                      | No       |
| **hint**                                  | Body     | string           | Hint to help solve the question in markdown format                      | No       |
| **answers**                               | Body     | array            | Array of answer options                                                 | Yes      |
| **answers[].description**                 | Body     | string           | Answer option text in markdown format                                   | Yes      |
| **answers[].is_correct_answer**           | Body     | boolean          | Whether this answer is correct                                          | Yes      |
| **question_to_groups**                    | Body     | array            | Group assignments for the question                                      | No       |
| **question_to_groups[].group_ids**        | Body     | array            | Array of group UUIDs                                                    | Yes      |
| **question_to_groups[].available_after**  | Body     | number           | Unix timestamp when question becomes available                          | Yes      |
| **question_to_groups[].available_before** | Body     | number           | Unix timestamp when question expires                                    | Yes      |
| **exams**                                 | Body     | array            | Exams this question should be added to                                  | No       |
| **exams[].id**                            | Body     | string           | UUID of the exam                                                        | Yes      |

**Note**: All markdown fields (`description`, `solution`, `study_orientation`, `hint`, and `answers[].description`) support standard markdown formatting including images. You can include images, videos, PDFs, or audio files as links - they will be automatically rendered if the file extension is detected.

#### Example

```json
{
  "description": "O Atomiuim, representado na imagem é umdos principais pontos turísticos de Bruxelas. Ele foi construído em 1958 para a primeira grande exposição mundial depois da Segunda Guerra Mundial, a Feira Mundial de Bruxelas. Trata-se de uma estrutura metálica construída no formato de um cubo. Essa estrutura está apoiada por um dos vértices sobre uma base paralela ao plano do solo, e a diagonal do cubo, contendo esse vértice, é ortogonal ao plano da base. Centradas nos vértices desse cubo, foram construídas oito esferas metálicas, e uma outra esfera foi construída centrada no ponto de interseção das diagonais do cubo. As oito esferas sobre os vértices são interligadas segundo suas arestas, e a esfera central se conecta a elas pelas diagonais do cubo. Todas essas interligações são feitas por tubos cilíndricos que possuem escadas em seu interior, permitindo o deslocamento de pessoas pela parte interna da estrutura. Na diagonal ortogonal à base, o deslocamento é feito por um elevador, que permite o deslocamento entre as esferas da base e a esfera do ponto mais alto, passando pela esfera central. Considere um visitante que se deslocou pelo interior do Atomium sempre em linha reta e seguindo o menor trajeto entre dois vértices, passando por todas as arestas e todas as diagonais do cubo.\n![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/1442427b-0cdd-4a10-a046-f4a4591acb6f.png)\nDisponível em: http://trupedatrip.com. Acesso em: 25 out. 2019. A projeção ortogonal sobre o plano do solo do trajeto percorrido por esse visitante é representada por",
  "discipline_id": 1,
  "subject": "Projeção Ortogonal",
  "type": "MULTIPLE_CHOICE",
  "difficulty": "MEDIUM",
  "source": "ENEM",
  "source_edition": "2021",
  "source_edition_observation": "Prova Azul - Dia 2",
  "source_question_number": 137,
  "hint": "Procure uma foto do Atomium no Google! A projeção ortogonal será muito mais facilmente visualizada.",
  "solution": "a) Todos os segmentos encontram-se no centro do quadrado e não como mostrado na figura.\nb) Dos 8 vértices do cubo, apenas 2 pares estão alinhados, ou seja, apenas 2 pares possuem a mesma projeção ortogonal. Assim, há a projeção de 6 pontos referentes aos vértices do cubo e não 4, como mostrado na figura.\nc) Não se trata de um retângulo.\nd) Não se trata de um retângulo.",
  "study_orientation": "https://humorquantico.com/courses/enem-2021/lessons/matematica/topic/o-atomium-representado-na-imagem/",
  "answers": [
    {
      "description": "A) ![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/51ec0352-d1a3-476c-a508-fdba7c494d16.png)",
      "is_correct_answer": false
    },
    {
      "description": "B) ![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/cd234955-e77f-4df5-93c8-11733ca767d9.png)",
      "is_correct_answer": false
    },
    {
      "description": "C) ![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/12d67c37-019d-48fe-9d4c-48f0eb2163fe.png)",
      "is_correct_answer": false
    },
    {
      "description": "D) ![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/59e04784-e1ff-4dbe-bccc-164cccbdce67.png)",
      "is_correct_answer": false
    },
    {
      "description": "E) ![](https://us-east-1-ugc-cdn.stift.com.br/quiz-questions/attachments/d2c48fe2-5f9e-4d02-a304-8afbcad91d5f.png)",
      "is_correct_answer": true
    }
  ],
  "question_to_groups": [
    {
      "group_ids": ["550e8400-e29b-41d4-a716-446655440000"],
      "available_after": 1640995200000,
      "available_before": 1672531200000
    }
  ],
  "exams": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001"
    }
  ]
}
```

### Response

#### Status

| Code | Description                               |
| ---- | ----------------------------------------- |
| 201  | The question was created successfully.    |
| 400  | Bad request - validation failed.          |
| 401  | Unauthorized.                             |
| 403  | Forbidden - insufficient permissions.     |
| 422  | Validation error - check required fields. |

## Update Question

If you want to update an existing quiz question in the question database, send this `PATCH` request:

```http
PATCH /exams/questions-db/:questionId
```

### Request

#### Parameters

| Parameter      | Location | Type          | Description                    | Required |
| -------------- | -------- | ------------- | ------------------------------ | -------- |
| **questionId** | URL      | string (UUID) | UUID of the question to update | Yes      |

All request body parameters are the same as [Create Question](#create-question). All fields are required even when updating.

#### Example

```json
{
  "description": "What is the capital and largest city of France?",
  "discipline_id": 1,
  "subject": 5,
  "type": "MULTIPLE_CHOICE",
  "difficulty": "MEDIUM",
  "source": "Geography Textbook 2024",
  "source_edition": "5th Edition",
  "source_question_number": 42,
  "hint": "Think about the most famous city in France, known for the Eiffel Tower",
  "solution": "Paris is the capital and largest city of France, known for landmarks like the Eiffel Tower and Louvre Museum.",
  "study_orientation": "Review European capitals and major cities",
  "answers": [
    {
      "description": "Paris",
      "is_correct_answer": true
    },
    {
      "description": "Lyon",
      "is_correct_answer": false
    },
    {
      "description": "Marseille",
      "is_correct_answer": false
    },
    {
      "description": "Toulouse",
      "is_correct_answer": false
    }
  ],
  "question_to_groups": [
    {
      "group_ids": ["550e8400-e29b-41d4-a716-446655440000"],
      "available_after": 1640995200000,
      "available_before": 1672531200000
    }
  ],
  "exams": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001"
    }
  ]
}
```

### Response

#### Status

| Code | Description                               |
| ---- | ----------------------------------------- |
| 200  | The question was updated successfully.    |
| 400  | Bad request - validation failed.          |
| 401  | Unauthorized.                             |
| 403  | Forbidden - insufficient permissions.     |
| 404  | Question not found.                       |
| 422  | Validation error - check required fields. |

## Create Exam

If you want to create a new exam with associated questions and group assignments, send this `POST` request:

```http
POST /exams
```

### Request

#### Parameters

| Parameter                                    | Location | Type    | Description                                           | Required |
| -------------------------------------------- | -------- | ------- | ----------------------------------------------------- | -------- |
| **name**                                     | Body     | string  | Name of the exam                                      | Yes      |
| **description**                              | Body     | string  | Description of the exam                               | Yes      |
| **exam_to_groups**                           | Body     | array   | Group assignments for the exam (minimum 1)            | Yes      |
| **exam_to_groups[].group_ids**               | Body     | array   | Array of group UUIDs (minimum 1)                      | Yes      |
| **exam_to_groups[].available_after**         | Body     | number  | Unix timestamp when exam becomes available            | Yes      |
| **exam_to_groups[].available_before**        | Body     | number  | Unix timestamp when exam expires                      | Yes      |
| **exam_to_groups[].instant_answer_feedback** | Body     | boolean | Whether to show immediate feedback after answering    | Yes      |
| **exam_to_groups[].allow_multiple_answers**  | Body     | boolean | Whether students can change their answers             | Yes      |
| **exam_to_groups[].is_readonly**             | Body     | boolean | Whether the exam is in read-only mode                 | Yes      |
| **question_ids**                             | Body     | array   | Global array of question UUIDs for the exam           | No       |

#### Example

```json
{
  "name": "Midterm Geography Exam",
  "description": "Comprehensive midterm examination covering European geography and capitals",
  "exam_to_groups": [
    {
      "group_ids": ["550e8400-e29b-41d4-a716-446655440000"],
      "available_after": 1640995200000,
      "available_before": 1641081600000,
      "instant_answer_feedback": false,
      "allow_multiple_answers": true,
      "is_readonly": false
    },
    {
      "group_ids": ["660e8400-e29b-41d4-a716-446655440001"],
      "available_after": 1641081600000,
      "available_before": 1641168000000,
      "instant_answer_feedback": true,
      "allow_multiple_answers": false,
      "is_readonly": false
    }
  ],
  "question_ids": [
    "123e4567-e89b-12d3-a456-426614174000",
    "234e5678-e89b-12d3-a456-426614174001"
  ]
}
```

### Response

#### Status

| Code | Description                               |
| ---- | ----------------------------------------- |
| 201  | The exam was created successfully.        |
| 400  | Bad request - validation failed.          |
| 401  | Unauthorized.                             |
| 403  | Forbidden - insufficient permissions.     |
| 422  | Validation error - check required fields. |


## Update Exam

If you want to update an existing exam, including its group assignments and question associations, send this `PATCH` request:

```http
PATCH /exams/:examId
```

### Request

#### Parameters

| Parameter  | Location | Type          | Description                | Required |
| ---------- | -------- | ------------- | -------------------------- | -------- |
| **examId** | URL      | string (UUID) | UUID of the exam to update | Yes      |

All request body parameters are the same as [Create Exam](#create-exam). All fields are required even when updating.

#### Example

```json
{
  "name": "Updated Midterm Geography Exam",
  "description": "Comprehensive midterm examination covering European and Asian geography",
  "exam_to_groups": [
    {
      "group_ids": ["550e8400-e29b-41d4-a716-446655440000"],
      "available_after": 1640995200000,
      "available_before": 1641254400000,
      "instant_answer_feedback": true,
      "allow_multiple_answers": true,
      "is_readonly": false
    }
  ],
  "question_ids": [
    "123e4567-e89b-12d3-a456-426614174000",
    "234e5678-e89b-12d3-a456-426614174001"
  ]
}
```

### Response

#### Status

| Code | Description                               |
| ---- | ----------------------------------------- |
| 200  | The exam was updated successfully.        |
| 400  | Bad request - validation failed.          |
| 401  | Unauthorized.                             |
| 403  | Forbidden - insufficient permissions.     |
| 404  | Exam not found.                           |
| 422  | Validation error - check required fields. |

### Subject Handling

When creating or updating questions, subjects can be handled in two ways:

- **By ID**: Reference an existing subject using its numeric ID
- **By Name**: Create a new subject by providing a string name - new subjects automatically inherit the discipline of the parent question. Note that if you provide a subject name, a new subject entity will be created regardless of wheather a subject with the same name already exists.

### Timestamp Handling

All date fields accept Unix timestamps (milliseconds since epoch) and are returned as ISO 8601 strings. Available dates control when exams and questions become accessible to students.

### Question and Exam Associations

- Questions can be associated with multiple exams
- When updating questions, only exams that the user has access to can be modified
- Existing exam associations for inaccessible exams are preserved
