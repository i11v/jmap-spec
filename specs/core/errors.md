# Errors

JMAP defines errors at two levels: request-level errors (HTTP) and method-level errors (in responses).

## Request-level errors

HTTP status codes indicate request-level problems:

### 400 Bad Request

The request is not valid JSON or is not a valid Request object:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "urn:ietf:params:jmap:error:notJSON",
  "status": 400,
  "detail": "The request body is not valid JSON"
}
```

### 400 Not Request

Valid JSON but not a valid JMAP Request:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "urn:ietf:params:jmap:error:notRequest",
  "status": 400,
  "detail": "The request is not a valid JMAP Request object"
}
```

### 401 Unauthorized

Authentication credentials are missing or invalid:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="jmap"
```

### 403 Forbidden

Authenticated but not authorized for this resource.

### 404 Not Found

The endpoint does not exist.

### 429 Too Many Requests

Rate limit exceeded:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{
  "type": "urn:ietf:params:jmap:error:limit",
  "status": 429,
  "detail": "Too many requests",
  "limit": "maxConcurrentRequests"
}
```

### 503 Service Unavailable

Server temporarily unavailable.

## Problem details format

Request errors use RFC 7807 Problem Details:

| Property | Type | Description |
|----------|------|-------------|
| type | `String` | Error type URI |
| status | `Number` | HTTP status code |
| title | `String` | Short human-readable summary |
| detail | `String` | Detailed explanation |
| limit | `String` | (optional) Which limit was exceeded |

## Request error types

| Type URI | Description |
|----------|-------------|
| urn:ietf:params:jmap:error:notJSON | Body is not valid JSON |
| urn:ietf:params:jmap:error:notRequest | Not a valid Request object |
| urn:ietf:params:jmap:error:limit | Server limit exceeded |
| urn:ietf:params:jmap:error:unknownCapability | Unknown capability in `using` |

## Method-level errors

When a method call fails, an error response replaces the normal response:

```json
{
  "methodResponses": [
    ["error", {
      "type": "invalidArguments",
      "description": "Unknown filter property: foo"
    }, "0"]
  ]
}
```

## Error response format

| Property | Type | Description |
|----------|------|-------------|
| type | `String` | Error type identifier |
| description | `String\|null` | Human-readable explanation |

## Standard error types

### unknownMethod

Method name not recognized:

```json
["error", {
  "type": "unknownMethod",
  "description": "Unknown method: Foo/bar"
}, "0"]
```

### invalidArguments

Invalid method arguments:

```json
["error", {
  "type": "invalidArguments",
  "description": "Property 'ids' must be an array or null"
}, "0"]
```

### invalidResultReference

Back-reference is invalid:

```json
["error", {
  "type": "invalidResultReference",
  "description": "Could not resolve reference: resultOf 'a' not found"
}, "0"]
```

### forbidden

Action not permitted:

```json
["error", {
  "type": "forbidden",
  "description": "You do not have permission to access this account"
}, "0"]
```

### accountNotFound

Account id doesn't exist:

```json
["error", {
  "type": "accountNotFound",
  "description": "Account 'X123' not found"
}, "0"]
```

### accountNotSupportedByMethod

Account doesn't support this method:

```json
["error", {
  "type": "accountNotSupportedByMethod",
  "description": "Account does not support urn:ietf:params:jmap:mail"
}, "0"]
```

### accountReadOnly

Account is read-only:

```json
["error", {
  "type": "accountReadOnly",
  "description": "This account is read-only"
}, "0"]
```

### anchorNotFound

Query anchor not found:

```json
["error", {
  "type": "anchorNotFound",
  "description": "Anchor id 'M123' not found in results"
}, "0"]
```

### cannotCalculateChanges

Cannot calculate changes for queryChanges:

```json
["error", {
  "type": "cannotCalculateChanges",
  "description": "State too old, full sync required"
}, "0"]
```

### stateMismatch

Expected state doesn't match:

```json
["error", {
  "type": "stateMismatch",
  "description": "Expected state 's123' but current state is 's456'"
}, "0"]
```

### requestTooLarge

Too many objects requested:

```json
["error", {
  "type": "requestTooLarge",
  "description": "Requested 1000 objects but limit is 500"
}, "0"]
```

### serverFail

Internal server error:

```json
["error", {
  "type": "serverFail",
  "description": "Internal server error"
}, "0"]
```

### serverPartialFail

Some but not all operations failed:

```json
["error", {
  "type": "serverPartialFail",
  "description": "Partial failure processing request"
}, "0"]
```

### serverUnavailable

Server temporarily unavailable:

```json
["error", {
  "type": "serverUnavailable",
  "description": "Service temporarily unavailable"
}, "0"]
```

### unknownCapability

Capability not in `using` array:

```json
["error", {
  "type": "unknownCapability",
  "description": "Method requires urn:ietf:params:jmap:mail capability"
}, "0"]
```

## SetError types

Errors in /set methods are reported per-object in notCreated/notUpdated/notDestroyed:

### forbidden

```json
{
  "notCreated": {
    "new-1": {
      "type": "forbidden",
      "description": "Cannot create objects in this account"
    }
  }
}
```

### notFound

```json
{
  "notUpdated": {
    "M123": {
      "type": "notFound",
      "description": "Object not found"
    }
  }
}
```

### invalidProperties

```json
{
  "notCreated": {
    "new-1": {
      "type": "invalidProperties",
      "properties": ["name", "parentId"],
      "description": "Invalid property values"
    }
  }
}
```

### singleton

Object already exists (for singleton types):

```json
{
  "notCreated": {
    "new-1": {
      "type": "singleton",
      "description": "Only one VacationResponse may exist"
    }
  }
}
```

### overQuota

Quota exceeded:

```json
{
  "notCreated": {
    "new-1": {
      "type": "overQuota",
      "description": "Storage quota exceeded"
    }
  }
}
```

### tooLarge

Object too large:

```json
{
  "notCreated": {
    "new-1": {
      "type": "tooLarge",
      "description": "Email exceeds maximum size"
    }
  }
}
```

### rateLimit

Rate limit exceeded:

```json
{
  "notCreated": {
    "new-1": {
      "type": "rateLimit",
      "description": "Too many objects created recently"
    }
  }
}
```

### invalidPatch

Invalid JSON Patch in update:

```json
{
  "notUpdated": {
    "M123": {
      "type": "invalidPatch",
      "description": "Invalid patch operation"
    }
  }
}
```

### willDestroy

Object will be destroyed in same request:

```json
{
  "notUpdated": {
    "M123": {
      "type": "willDestroy",
      "description": "Object is being destroyed in this request"
    }
  }
}
```
