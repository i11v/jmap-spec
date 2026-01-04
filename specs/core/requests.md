# Requests and Responses

JMAP uses a simple request/response model over HTTP POST. Multiple method calls can be batched in a single request for efficiency.

## Request object

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail"
  ],
  "methodCalls": [
    ["Email/query", {
      "accountId": "A123",
      "filter": { "inMailbox": "MB1" },
      "limit": 10
    }, "a"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "a",
        "name": "Email/query",
        "path": "/ids"
      },
      "properties": ["id", "subject", "from"]
    }, "b"]
  ],
  "createdIds": {
    "temp-id-1": "real-id-1"
  }
}
```

## Request properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| using | `String[]` | Yes | Capability URIs to enable |
| methodCalls | `Invocation[]` | Yes | Ordered list of method calls |
| createdIds | `Id[Id]` | No | Map of client ids to server ids |

## Method call format

Each method call is an array: `[methodName, arguments, callId]`

| Index | Type | Description |
|-------|------|-------------|
| 0 | `String` | Method name (e.g., "Email/get") |
| 1 | `Object` | Method arguments |
| 2 | `String` | Client-specified call identifier |

## Response object

```json
{
  "methodResponses": [
    ["Email/query", {
      "accountId": "A123",
      "queryState": "q12345",
      "canCalculateChanges": true,
      "position": 0,
      "total": 100,
      "ids": ["M1", "M2", "M3"]
    }, "a"],
    ["Email/get", {
      "accountId": "A123",
      "state": "s67890",
      "list": [
        {"id": "M1", "subject": "Hello", "from": [{"email": "bob@example.com"}]},
        {"id": "M2", "subject": "Re: Hello", "from": [{"email": "alice@example.com"}]}
      ],
      "notFound": []
    }, "b"]
  ],
  "sessionState": "abc123"
}
```

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| methodResponses | `Invocation[]` | Ordered list of responses |
| sessionState | `String` | Current session state |
| createdIds | `Id[Id]` | Map of created ids (if any) |

## Back-references

Use `#` prefix and ResultReference to reference results from previous calls:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "A123",
      "filter": { "inMailbox": "MB1" },
      "limit": 10
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/query",
        "path": "/ids"
      }
    }, "1"]
  ]
}
```

## ResultReference object

| Property | Type | Description |
|----------|------|-------------|
| resultOf | `String` | Call id to reference |
| name | `String` | Method name that produced the result |
| path | `String` | JSON Pointer to value in response |

## Common path expressions

| Path | Description |
|------|-------------|
| `/ids` | Array of ids from query |
| `/list/*/id` | All id properties from list |
| `/list/*/threadId` | All threadId properties |
| `/created/*/id` | All created object ids |
| `/updated` | Array of updated ids |

## Creation references

Reference objects being created in the same request using `#` prefix:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "A123",
      "create": {
        "parent": {
          "name": "Parent",
          "parentId": null
        },
        "child": {
          "name": "Child",
          "parentId": "#parent"
        }
      }
    }, "0"]
  ]
}
```

## HTTP request

```http
POST /api/ HTTP/1.1
Host: jmap.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [...]
}
```

## HTTP response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "methodResponses": [...],
  "sessionState": "abc123"
}
```

## Required capabilities

The `using` array must include all capabilities needed for the methods being called:

| Capability | Required for |
|------------|--------------|
| urn:ietf:params:jmap:core | Always required |
| urn:ietf:params:jmap:mail | Mailbox, Thread, Email, SearchSnippet |
| urn:ietf:params:jmap:submission | Identity, EmailSubmission |
| urn:ietf:params:jmap:vacationresponse | VacationResponse |

## Core/echo method

Test method that echoes back its arguments:

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["Core/echo", {
      "hello": "world",
      "numbers": [1, 2, 3]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Core/echo", {
      "hello": "world",
      "numbers": [1, 2, 3]
    }, "0"]
  ],
  "sessionState": "abc123"
}
```

## Request limits

Requests must respect session limits:

| Limit | Session property |
|-------|------------------|
| Request size | maxSizeRequest |
| Method calls | maxCallsInRequest |
| Objects in get | maxObjectsInGet |
| Objects in set | maxObjectsInSet |

## Concurrent requests

Multiple requests can be sent in parallel up to `maxConcurrentRequests`. Each request is processed independently.
