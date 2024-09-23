# Webhooks

In order to allow for a seamless integration with your system, we provide a way to notify you when events happen on Stift, such as when a question or reply is created or answered.

## Events

### QUESTION_ANSWERED

This event is triggered when a question is answered. Its payload brings data about the question that was answered:

```json
{
  "event": "QUESTION_ANSWERED",
  "data": {
    "id": "c64a24e4-ebb5-40ef-8828-9a7c006b5062",
    "created_at": "2024-12-12T13:17:08.348Z",
    "student": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student",
      "last_name": "Test",
      "email": "teststudent@stift.com.br"
    },
    "answer": {
      "id": "b043caa2-af8d-464d-9e74-40da4ead9ffd",
      "created_at": "2024-12-12T13:20:08.348Z",
      "teacher": {
        "id": "09c25b20-266e-49d8-89d2-e7622c84a6d9",
        "first_name": "Teacher",
        "last_name": "Tect",
        "email": "testteacher@stift.com.br"
      }
    },
    "group": {
      "id": "e5f8abe2-53f2-48bc-b7a4-d6167b9e900f",
      "name": "Test Group",
      "business_model": "B2B_SAAS"
    },
    "school": {
      "id": "8f03cc77-bf49-4a9f-9b7c-e7dba5f068d7",
      "name": "Test School"
    },
    "discipline": { "name": "Math" },
    "subject": { "name": "Algebra" }
  }
}
```

### REPLY_ANSWERED

This event is triggered when a reply is answered. Its payload bring data about the reply that was answered:

```json
{
  "event": "REPLY_ANSWERED",
  "data": {
    "id": "c64a24e4-ebb5-40ef-8828-9a7c006b5062",
    "created_at": "2024-12-12T13:17:08.348Z",
    "student": {
      "id": "3591f1c3-cf26-4f36-b4d2-d7078359c0d0",
      "first_name": "Student",
      "last_name": "Test",
      "email": "teststudent@stift.com.br"
    },
    "answer": {
      "id": "b043caa2-af8d-464d-9e74-40da4ead9ffd",
      "created_at": "2024-12-12T13:20:08.348Z",
      "teacher": {
        "id": "09c25b20-266e-49d8-89d2-e7622c84a6d9",
        "first_name": "Teacher",
        "last_name": "Tect",
        "email": "testteacher@stift.com.br"
      }
    },
    "group": {
      "id": "e5f8abe2-53f2-48bc-b7a4-d6167b9e900f",
      "name": "Test Group",
      "business_model": "B2B_OUTSOURCING"
    },
    "school": {
      "id": "8f03cc77-bf49-4a9f-9b7c-e7dba5f068d7",
      "name": "Test School"
    },
    "discipline": { "name": "Math" },
    "subject": { "name": "Algebra" },
    "url": "https://app.stift.com.br/?question=c64a24e4-ebb5-40ef-8828-9a7c006b5062"
  }
}
```

## Security

In order to provide you with a way to validate the events, we attribute a unique ID to each webhook. You can use this ID to validate the authenticity of the event. It is sent in the webhook's request headers, as `Stift-Webhook-ID`.

## Retry Policy

Our webhooks system is designed to retry sending the event in case of failure. We will retry sending the event up to 3 times, with an exponential backoff strategy of 1, 2, and 4 seconds.

## Idempotency

To prevent the same event from being processed multiple times in case of retries, we provide a way for you to ensure that the event is processed only once. You can use the `Stift-Event-ID` to store the event's ID and check if it has already been processed.
