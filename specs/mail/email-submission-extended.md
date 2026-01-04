# EmailSubmission (Extended)

Additional EmailSubmission methods: changes, query, and queryChanges.

## EmailSubmission/get

See email-submission.md for basic get usage.

## EmailSubmission/changes

Get changes to submissions since a previous state:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/changes", {
      "accountId": "A123",
      "sinceState": "es100"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/changes", {
      "accountId": "A123",
      "oldState": "es100",
      "newState": "es110",
      "hasMoreChanges": false,
      "created": ["ES200", "ES201"],
      "updated": ["ES150"],
      "destroyed": ["ES100"]
    }, "0"]
  ]
}
```

### What triggers changes

- **created**: New submission created
- **updated**: Delivery status changed, undoStatus changed
- **destroyed**: Submission expired or cleaned up

## EmailSubmission/query

Query submissions with filters and sorting:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/query", {
      "accountId": "A123",
      "filter": {
        "undoStatus": "pending"
      },
      "sort": [
        { "property": "sendAt", "isAscending": false }
      ],
      "limit": 50
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/query", {
      "accountId": "A123",
      "queryState": "qes100",
      "canCalculateChanges": true,
      "position": 0,
      "total": 3,
      "ids": ["ES200", "ES201", "ES202"]
    }, "0"]
  ]
}
```

### FilterCondition properties

| Property | Type | Description |
|----------|------|-------------|
| identityIds | `Id[]` | Submissions from these identities |
| emailIds | `Id[]` | Submissions of these emails |
| threadIds | `Id[]` | Submissions of emails in these threads |
| undoStatus | `String` | "pending", "final", or "canceled" |
| before | `UTCDate` | sendAt before this date |
| after | `UTCDate` | sendAt after this date |

### Sort properties

| Property | Description |
|----------|-------------|
| emailId | Sort by email id |
| threadId | Sort by thread id |
| sendAt | Sort by send time |

## EmailSubmission/queryChanges

Get changes to a query result:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/queryChanges", {
      "accountId": "A123",
      "filter": {
        "undoStatus": "pending"
      },
      "sort": [
        { "property": "sendAt", "isAscending": false }
      ],
      "sinceQueryState": "qes100"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "qes100",
      "newQueryState": "qes105",
      "total": 2,
      "removed": ["ES200"],
      "added": [
        { "id": "ES203", "index": 0 }
      ]
    }, "0"]
  ]
}
```

## Monitoring scheduled emails

Query pending submissions to show scheduled emails:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/query", {
      "accountId": "A123",
      "filter": {
        "undoStatus": "pending"
      },
      "sort": [{ "property": "sendAt", "isAscending": true }]
    }, "0"],
    ["EmailSubmission/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "EmailSubmission/query",
        "path": "/ids"
      }
    }, "1"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "1",
        "name": "EmailSubmission/get",
        "path": "/list/*/emailId"
      },
      "properties": ["id", "subject", "to", "sentAt"]
    }, "2"]
  ]
}
```

## Sync workflow

Track submission status changes:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/changes", {
      "accountId": "A123",
      "sinceState": "es100"
    }, "0"],
    ["EmailSubmission/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "EmailSubmission/changes",
        "path": "/updated"
      }
    }, "1"]
  ]
}
```

## Delivery status tracking

Monitor delivery of sent emails:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/get", {
      "accountId": "A123",
      "ids": ["ES150"],
      "properties": ["id", "emailId", "undoStatus", "deliveryStatus", "dsnBlobIds"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/get", {
      "accountId": "A123",
      "state": "es110",
      "list": [
        {
          "id": "ES150",
          "emailId": "M500",
          "undoStatus": "final",
          "deliveryStatus": {
            "bob@example.com": {
              "smtpReply": "250 2.0.0 OK",
              "delivered": "yes",
              "displayed": "unknown"
            },
            "carol@example.com": {
              "smtpReply": "550 5.1.1 User unknown",
              "delivered": "no",
              "displayed": "unknown"
            }
          },
          "dsnBlobIds": ["Bdsn123"]
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## DeliveryStatus values

### delivered

| Value | Description |
|-------|-------------|
| queued | Message queued for delivery |
| yes | Successfully delivered |
| no | Delivery failed |
| unknown | Delivery status unknown |

### displayed

| Value | Description |
|-------|-------------|
| yes | Recipient opened/displayed message (MDN received) |
| unknown | No MDN received |

## DSN and MDN

- `dsnBlobIds`: Delivery Status Notification messages (bounces)
- `mdnBlobIds`: Message Disposition Notifications (read receipts)

Fetch DSN content:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bdsn123"],
      "properties": ["from", "subject", "textBody", "bodyValues"],
      "fetchTextBodyValues": true
    }, "0"]
  ]
}
```

## Cleanup

Old submissions are typically cleaned up by the server. Query to check what's retained:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/query", {
      "accountId": "A123",
      "filter": {
        "before": "2024-01-01T00:00:00Z"
      },
      "calculateTotal": true
    }, "0"]
  ]
}
```
