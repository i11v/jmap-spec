# Email/query

Standard "/query" method for searching and filtering Email objects as described in RFC 8620 Section 5.5.
Returns a list of Email ids matching the given filter, sorted according to the specified comparators.

## Basic query

Query all emails in a mailbox:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "u33084183",
      "filter": {
        "inMailbox": "MB23cfa8094c0f41e6"
      },
      "sort": [
        { "property": "receivedAt", "isAscending": false }
      ],
      "position": 0,
      "limit": 10
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/query", {
      "accountId": "u33084183",
      "queryState": "1234567890",
      "canCalculateChanges": true,
      "position": 0,
      "total": 16307,
      "ids": [
        "Mf123u456",
        "Mf123u457",
        "Mf123u458"
      ]
    }, "0"]
  ]
}
```

## Filter conditions

### By mailbox

```json
{
  "filter": {
    "inMailbox": "MB23cfa8094c0f41e6"
  }
}
```

### Exclude mailbox

```json
{
  "filter": {
    "inMailboxOtherThan": ["MBtrash123", "MBspam456"]
  }
}
```

### Unread emails

```json
{
  "filter": {
    "inMailbox": "MB23cfa8094c0f41e6",
    "hasKeyword": "$seen",
    "notKeyword": "$seen"
  }
}
```

Filter for unread emails (without $seen keyword):

```json
{
  "filter": {
    "inMailbox": "MB23cfa8094c0f41e6",
    "notKeyword": "$seen"
  }
}
```

### By sender

```json
{
  "filter": {
    "from": "alice@example.com"
  }
}
```

### By recipient

```json
{
  "filter": {
    "to": "bob@example.com"
  }
}
```

### By subject

```json
{
  "filter": {
    "subject": "meeting notes"
  }
}
```

### Full text search

```json
{
  "filter": {
    "text": "quarterly report"
  }
}
```

### By date range

```json
{
  "filter": {
    "after": "2024-01-01T00:00:00Z",
    "before": "2024-02-01T00:00:00Z"
  }
}
```

### With attachments

```json
{
  "filter": {
    "hasAttachment": true
  }
}
```

### By size

```json
{
  "filter": {
    "minSize": 1048576
  }
}
```

## Compound filters

### AND filter (all conditions must match)

```json
{
  "filter": {
    "operator": "AND",
    "conditions": [
      { "inMailbox": "MB23cfa8094c0f41e6" },
      { "from": "boss@company.com" },
      { "notKeyword": "$seen" }
    ]
  }
}
```

### OR filter (any condition must match)

```json
{
  "filter": {
    "operator": "OR",
    "conditions": [
      { "from": "alice@example.com" },
      { "from": "bob@example.com" }
    ]
  }
}
```

### NOT filter

```json
{
  "filter": {
    "operator": "NOT",
    "conditions": [
      { "hasKeyword": "$draft" }
    ]
  }
}
```

### Nested filters

```json
{
  "filter": {
    "operator": "AND",
    "conditions": [
      { "inMailbox": "MB23cfa8094c0f41e6" },
      {
        "operator": "OR",
        "conditions": [
          { "from": "alice@example.com" },
          { "from": "bob@example.com" }
        ]
      }
    ]
  }
}
```

## Sorting

### By received date (newest first)

```json
{
  "sort": [
    { "property": "receivedAt", "isAscending": false }
  ]
}
```

### By sent date

```json
{
  "sort": [
    { "property": "sentAt", "isAscending": false }
  ]
}
```

### By subject

```json
{
  "sort": [
    { "property": "subject", "isAscending": true }
  ]
}
```

### By sender

```json
{
  "sort": [
    { "property": "from", "isAscending": true }
  ]
}
```

### Multiple sort criteria

```json
{
  "sort": [
    { "property": "hasKeyword", "keyword": "$flagged", "isAscending": false },
    { "property": "receivedAt", "isAscending": false }
  ]
}
```

## Pagination

### Using position and limit

```json
{
  "position": 0,
  "limit": 50
}
```

### Fetch next page

```json
{
  "position": 50,
  "limit": 50
}
```

### Using anchor

```json
{
  "anchor": "Mf123u456",
  "anchorOffset": -10,
  "limit": 50
}
```

## Collapsing threads

Collapse results to one email per thread:

```json
{
  "collapseThreads": true
}
```

## FilterCondition properties

| Property | Type | Description |
|----------|------|-------------|
| inMailbox | `Id` | Email must be in this mailbox |
| inMailboxOtherThan | `Id[]` | Email must NOT be in any of these mailboxes |
| before | `UTCDate` | receivedAt must be before this date |
| after | `UTCDate` | receivedAt must be after this date |
| minSize | `UnsignedInt` | Email size must be >= this value (bytes) |
| maxSize | `UnsignedInt` | Email size must be <= this value (bytes) |
| allInThreadHaveKeyword | `String` | All emails in thread must have this keyword |
| someInThreadHaveKeyword | `String` | At least one email in thread has this keyword |
| noneInThreadHaveKeyword | `String` | No email in thread has this keyword |
| hasKeyword | `String` | Email must have this keyword |
| notKeyword | `String` | Email must NOT have this keyword |
| hasAttachment | `Boolean` | Email must have/not have attachments |
| text | `String` | Full text search in headers and body |
| from | `String` | Match in From header |
| to | `String` | Match in To header |
| cc | `String` | Match in Cc header |
| bcc | `String` | Match in Bcc header |
| subject | `String` | Match in Subject header |
| body | `String` | Full text search in body only |
| header | `String[]` | Match specific header [name, value] |

## Error cases

### Invalid filter property

```json
{
  "filter": {
    "invalidProperty": "value"
  }
}
```

```json
// error response
{
  "methodResponses": [
    ["error", {
      "type": "invalidArguments",
      "description": "Unknown filter property: invalidProperty"
    }, "0"]
  ]
}
```

### Unsupported sort property

```json
{
  "sort": [
    { "property": "unsupportedField", "isAscending": true }
  ]
}
```

```json
// error response
{
  "methodResponses": [
    ["error", {
      "type": "unsupportedSort",
      "description": "Sort by 'unsupportedField' is not supported"
    }, "0"]
  ]
}
```
