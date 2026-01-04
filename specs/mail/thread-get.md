# Thread/get

Standard "/get" method for retrieving Thread objects as described in RFC 8620 Section 5.1.
A Thread represents a conversation of related emails.

## Basic thread retrieval

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/get", {
      "accountId": "u33084183",
      "ids": ["T1234567890"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Thread/get", {
      "accountId": "u33084183",
      "state": "789012",
      "list": [
        {
          "id": "T1234567890",
          "emailIds": [
            "Mf123u456",
            "Mf123u457",
            "Mf123u458"
          ]
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Multiple threads

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/get", {
      "accountId": "u33084183",
      "ids": ["T1234567890", "T0987654321"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Thread/get", {
      "accountId": "u33084183",
      "state": "789012",
      "list": [
        {
          "id": "T1234567890",
          "emailIds": ["Mf123u456", "Mf123u457", "Mf123u458"]
        },
        {
          "id": "T0987654321",
          "emailIds": ["Mf789a123"]
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Thread not found

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/get", {
      "accountId": "u33084183",
      "ids": ["T1234567890", "nonexistent"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Thread/get", {
      "accountId": "u33084183",
      "state": "789012",
      "list": [
        {
          "id": "T1234567890",
          "emailIds": ["Mf123u456", "Mf123u457"]
        }
      ],
      "notFound": ["nonexistent"]
    }, "0"]
  ]
}
```

## Combined with Email/query

Typical pattern: Query emails with collapseThreads, then fetch threads:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "u33084183",
      "filter": { "inMailbox": "MBinbox123" },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "collapseThreads": true,
      "limit": 20
    }, "0"],
    ["Email/get", {
      "accountId": "u33084183",
      "#ids": {
        "resultOf": "0",
        "name": "Email/query",
        "path": "/ids"
      },
      "properties": ["threadId", "from", "subject", "receivedAt", "preview"]
    }, "1"],
    ["Thread/get", {
      "accountId": "u33084183",
      "#ids": {
        "resultOf": "1",
        "name": "Email/get",
        "path": "/list/*/threadId"
      }
    }, "2"]
  ]
}
```

## Thread properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Immutable, server-set. The thread identifier. |
| emailIds | `Id[]` | Immutable, server-set. List of email ids in this thread, sorted by receivedAt (oldest first). |

## Thread behavior

### Email ordering

Emails in a thread are ordered by their `receivedAt` timestamp, oldest first:

```json
{
  "id": "T123",
  "emailIds": [
    "M001",  // oldest (receivedAt: 2024-01-01T10:00:00Z)
    "M002",  // middle (receivedAt: 2024-01-01T11:00:00Z)
    "M003"   // newest (receivedAt: 2024-01-01T12:00:00Z)
  ]
}
```

### Thread linking

Emails are linked into threads based on:
- Message-ID header
- In-Reply-To header
- References header

### Single email thread

An email not linked to any others forms its own thread:

```json
{
  "id": "T456",
  "emailIds": ["M789"]
}
```

### Thread merging

When a new email arrives that links previously separate threads, the threads are merged and one id becomes canonical.

### Thread splitting

Threads may be split if an email is deleted that was the only link between parts of the conversation.

## Empty threads

A Thread object will never be returned with an empty `emailIds` array. If all emails in a thread are destroyed, the thread is automatically destroyed.

## Thread changes

Use `Thread/changes` to detect when threads are modified:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/changes", {
      "accountId": "u33084183",
      "sinceState": "789010"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Thread/changes", {
      "accountId": "u33084183",
      "oldState": "789010",
      "newState": "789012",
      "hasMoreChanges": false,
      "created": ["T999"],
      "updated": ["T123"],
      "destroyed": ["T456"]
    }, "0"]
  ]
}
```
