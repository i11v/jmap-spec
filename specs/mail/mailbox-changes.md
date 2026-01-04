# Mailbox/changes

Standard "/changes" method for getting Mailbox changes since a previous state.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "sinceState": "s78540"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "oldState": "s78540",
      "newState": "s78545",
      "hasMoreChanges": false,
      "created": ["MB999"],
      "updated": ["MB123", "MB456"],
      "destroyed": ["MB789"],
      "updatedProperties": null
    }, "0"]
  ]
}
```

## With max changes limit

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "sinceState": "s78540",
      "maxChanges": 50
    }, "0"]
  ]
}
```

```json
// response with more changes
{
  "methodResponses": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "oldState": "s78540",
      "newState": "s78542",
      "hasMoreChanges": true,
      "created": ["MB999", "MB1000"],
      "updated": ["MB123"],
      "destroyed": [],
      "updatedProperties": null
    }, "0"]
  ]
}
```

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldState | `String` | State from request |
| newState | `String` | Current state |
| hasMoreChanges | `Boolean` | More changes available beyond maxChanges |
| created | `Id[]` | Ids of newly created mailboxes |
| updated | `Id[]` | Ids of updated mailboxes |
| destroyed | `Id[]` | Ids of destroyed mailboxes |
| updatedProperties | `String[]\|null` | Properties that changed (special) |

## updatedProperties optimization

If only count properties changed, `updatedProperties` lists them:

```json
{
  "methodResponses": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "oldState": "s78540",
      "newState": "s78541",
      "hasMoreChanges": false,
      "created": [],
      "updated": ["MB123", "MB456"],
      "destroyed": [],
      "updatedProperties": ["totalEmails", "unreadEmails", "totalThreads", "unreadThreads"]
    }, "0"]
  ]
}
```

This allows efficient fetching of only changed properties:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "sinceState": "s78540"
    }, "0"],
    ["Mailbox/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Mailbox/changes",
        "path": "/updated"
      },
      "#properties": {
        "resultOf": "0",
        "name": "Mailbox/changes",
        "path": "/updatedProperties"
      }
    }, "1"]
  ]
}
```

## Count-only properties

Properties that may be listed in `updatedProperties`:
- `totalEmails`
- `unreadEmails`
- `totalThreads`
- `unreadThreads`

If any other property changed, `updatedProperties` is `null`.

## Sync workflow

Typical sync pattern:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/changes", {
      "accountId": "A123",
      "sinceState": "s78540"
    }, "0"],
    ["Mailbox/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Mailbox/changes",
        "path": "/created"
      }
    }, "1"],
    ["Mailbox/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Mailbox/changes",
        "path": "/updated"
      }
    }, "2"]
  ]
}
```

## Error: cannotCalculateChanges

When the state is too old:

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

Client must do a full Mailbox/get with `ids: null` to resync.

## Error: invalidArguments

Invalid sinceState:

```json
{
  "methodResponses": [
    ["error", {
      "type": "invalidArguments",
      "description": "Invalid sinceState value"
    }, "0"]
  ]
}
```
