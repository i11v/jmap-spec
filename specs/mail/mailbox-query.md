# Mailbox/query

Standard "/query" method for searching and filtering Mailbox objects.

## Basic query

Get all mailboxes sorted by name:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "sort": [
        { "property": "name", "isAscending": true }
      ]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/query", {
      "accountId": "A123",
      "queryState": "q12345",
      "canCalculateChanges": true,
      "position": 0,
      "total": 15,
      "ids": ["MBinbox", "MBarchive", "MBdrafts", "MBsent", "MBtrash"]
    }, "0"]
  ]
}
```

## Filter by parent

Top-level mailboxes only:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "parentId": null
      }
    }, "0"]
  ]
}
```

Children of a specific mailbox:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "parentId": "MB123"
      }
    }, "0"]
  ]
}
```

## Filter by name

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "name": "project"
      }
    }, "0"]
  ]
}
```

## Filter by role

Find inbox:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "role": "inbox"
      }
    }, "0"]
  ]
}
```

Find all system mailboxes (with any role):

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "hasAnyRole": true
      }
    }, "0"]
  ]
}
```

Find user-created mailboxes (no role):

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "hasAnyRole": false
      }
    }, "0"]
  ]
}
```

## Filter by subscription

Subscribed mailboxes only:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "isSubscribed": true
      }
    }, "0"]
  ]
}
```

## Sorting

### By sortOrder (custom order)

```json
{
  "sort": [
    { "property": "sortOrder", "isAscending": true }
  ]
}
```

### By name

```json
{
  "sort": [
    { "property": "name", "isAscending": true }
  ]
}
```

### Combined sort

```json
{
  "sort": [
    { "property": "sortOrder", "isAscending": true },
    { "property": "name", "isAscending": true }
  ]
}
```

## Tree sorting (sortAsTree)

Sort as a tree, keeping children under parents:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "sortAsTree": true,
      "sort": [
        { "property": "name", "isAscending": true }
      ]
    }, "0"]
  ]
}
```

With `sortAsTree: true`:
- Parents always come before their children
- Siblings are sorted according to sort comparators
- Results can be displayed as an indented tree

## Tree filtering (filterAsTree)

Only include mailboxes where ancestors also match:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": {
        "name": "project"
      },
      "filterAsTree": true
    }, "0"]
  ]
}
```

With `filterAsTree: true`:
- A mailbox only matches if all its ancestors also match the filter
- Useful for showing subtrees that match criteria

## FilterCondition properties

| Property | Type | Description |
|----------|------|-------------|
| parentId | `Id\|null` | Match exact parentId |
| name | `String` | Name contains this string |
| role | `String\|null` | Match exact role |
| hasAnyRole | `Boolean` | Has any role (true) or no role (false) |
| isSubscribed | `Boolean` | Match isSubscribed value |

## Request arguments

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| accountId | `Id` | Required | Account to query |
| filter | `FilterOperator\|FilterCondition\|null` | null | Filter criteria |
| sort | `Comparator[]\|null` | null | Sort order |
| sortAsTree | `Boolean` | false | Sort preserving tree structure |
| filterAsTree | `Boolean` | false | Only include if ancestors match |
| position | `Int` | 0 | Start position |
| anchor | `Id\|null` | null | Position relative to id |
| anchorOffset | `Int` | 0 | Offset from anchor |
| limit | `UnsignedInt\|null` | null | Max results |
| calculateTotal | `Boolean` | false | Include total count |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| queryState | `String` | State for queryChanges |
| canCalculateChanges | `Boolean` | queryChanges supported |
| position | `UnsignedInt` | Position of first result |
| total | `UnsignedInt\|null` | Total matching (if requested) |
| ids | `Id[]` | Matching mailbox ids |

## Combined with Mailbox/get

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/query", {
      "accountId": "A123",
      "filter": { "isSubscribed": true },
      "sortAsTree": true,
      "sort": [{ "property": "sortOrder", "isAscending": true }]
    }, "0"],
    ["Mailbox/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Mailbox/query",
        "path": "/ids"
      },
      "properties": ["id", "name", "parentId", "role", "totalEmails", "unreadEmails"]
    }, "1"]
  ]
}
```
