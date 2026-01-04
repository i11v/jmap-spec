# Email/changes

Standard "/changes" method for getting Email changes since a previous state.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/changes", {
      "accountId": "A123",
      "sinceState": "s123456"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/changes", {
      "accountId": "A123",
      "oldState": "s123456",
      "newState": "s123500",
      "hasMoreChanges": false,
      "created": ["M999", "M1000", "M1001"],
      "updated": ["M500", "M501"],
      "destroyed": ["M100", "M101"]
    }, "0"]
  ]
}
```

## With max changes limit

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/changes", {
      "accountId": "A123",
      "sinceState": "s123456",
      "maxChanges": 100
    }, "0"]
  ]
}
```

```json
// response with more changes
{
  "methodResponses": [
    ["Email/changes", {
      "accountId": "A123",
      "oldState": "s123456",
      "newState": "s123480",
      "hasMoreChanges": true,
      "created": ["M999", "M1000"],
      "updated": ["M500"],
      "destroyed": ["M100"]
    }, "0"]
  ]
}
```

When `hasMoreChanges` is true, call again with `sinceState: newState` to get more.

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| accountId | `Id` | Yes | Account to query |
| sinceState | `String` | Yes | Previous state string |
| maxChanges | `UnsignedInt\|null` | No | Max changes to return |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldState | `String` | State from request |
| newState | `String` | Current state |
| hasMoreChanges | `Boolean` | More changes available |
| created | `Id[]` | Ids of newly created emails |
| updated | `Id[]` | Ids of updated emails |
| destroyed | `Id[]` | Ids of destroyed emails |

## What triggers "updated"

An email appears in `updated` when:
- Keywords changed ($seen, $flagged, etc.)
- Mailbox membership changed (moved, labeled)

## What does NOT trigger "updated"

Email content is immutable, so body/headers never cause updates.

## Sync workflow

Typical sync pattern:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/changes", {
      "accountId": "A123",
      "sinceState": "s123456"
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/changes",
        "path": "/created"
      },
      "properties": ["id", "threadId", "mailboxIds", "keywords", "from", "subject", "receivedAt", "preview"]
    }, "1"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/changes",
        "path": "/updated"
      },
      "properties": ["id", "mailboxIds", "keywords"]
    }, "2"]
  ]
}
```

## Looping for all changes

```javascript
async function syncAllChanges(accountId, sinceState) {
  let state = sinceState;
  let allCreated = [];
  let allUpdated = [];
  let allDestroyed = [];

  while (true) {
    const response = await jmap.call('Email/changes', {
      accountId,
      sinceState: state,
      maxChanges: 500
    });

    allCreated.push(...response.created);
    allUpdated.push(...response.updated);
    allDestroyed.push(...response.destroyed);

    state = response.newState;

    if (!response.hasMoreChanges) {
      break;
    }
  }

  return { state, allCreated, allUpdated, allDestroyed };
}
```

## Error: cannotCalculateChanges

```json
{
  "methodResponses": [
    ["error", {
      "type": "cannotCalculateChanges",
      "description": "State is too old, full sync required"
    }, "0"]
  ]
}
```

When this occurs:
1. The sinceState is too old (server no longer has change log)
2. Client must perform full sync

Full sync approach:
```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "A123",
      "filter": null,
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "limit": 1000
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/query",
        "path": "/ids"
      },
      "properties": ["id", "threadId", "mailboxIds", "keywords", "from", "subject", "receivedAt", "preview"]
    }, "1"]
  ]
}
```

## Efficient delta sync

For UI with a list of emails, combine with queryChanges:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "filter": { "inMailbox": "MBinbox" },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "sinceQueryState": "q999",
      "calculateTotal": true
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/queryChanges",
        "path": "/added/*/id"
      },
      "properties": ["id", "threadId", "from", "subject", "receivedAt", "preview", "keywords"]
    }, "1"]
  ]
}
```
