# Getting started

## Basics

Stift's API is RESTful and uses standard HTTP response codes, authentication, and verbs.

You can use your favorite HTTP/REST library that is available for your programming language to make HTTP calls.

You must send JSON payloads in your requests and expect to get JSON responses. Don't forget to add these headers to each and every request.

```http
Content-Type: application/json
Accept: application/json
```

Use the following base URL for all API endpoints:

```http
https://api.stift.com.br/v1/
```

## Authentication

### API Keys

API keys are a quick way to implement machine-to-machine authentication without any direct inputs from a human beyond initial setup.

You can get an API key by contacting us through our official channels.

Once the key is generated, please copy and store it immediately and don't publish it anywhere public. Treat it as you would a password, as it gives access to all resources inside your account.

Once you have your API key, provide it in an Authorization header together with your request payload, where XXX is your token:

```http
Authorization: Bearer XXX
```

If you fail to provide a valid token, you'll get this response on all requests.

```json title="Response Code: 401 Unauthorized"
{
    "error": {
        "code": 401,
        "message": "Unauthorized."
    }
}
```

### Errors

You might encounter validation errors when sending requests to API endpoints. They will come in the following format:

```json title="Response Code: 400 Bad Request"
{
    "error": {
        "code": 400,
        "message": "Invalid school id."
    }
}
```