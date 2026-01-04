# Thread/changes

Standard "/changes" method for getting Thread changes since a previous state.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/changes", {
      "accountId": "A123",
      "sinceState": "s12345"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Thread/changes", {
      "accountId": "A123",
      "oldState": "s12345",
      "newState": "s12400",
      "hasMoreChanges": false,
      "created": ["T999", "T1000"],
      "updated": ["T500", "T501"],
      "destroyed": ["T100"]
    }, "0"]
  ]
}
```

## With max changes limit

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/changes", {
      "accountId": "A123",
      "sinceState": "s12345",
      "maxChanges": 100
    }, "0"]
  ]
}
```

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
| created | `Id[]` | Ids of newly created threads |
| updated | `Id[]` | Ids of updated threads |
| destroyed | `Id[]` | Ids of destroyed threads |

## What triggers changes

### created

- New email arrives that doesn't match existing thread
- Email imported/created with new Message-ID

### updated

- New email added to existing thread
- Email removed from thread
- Thread merged with another thread

### destroyed

- All emails in thread destroyed
- Thread merged into another thread

## Sync workflow

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Thread/changes", {
      "accountId": "A123",
      "sinceState": "s12345"
    }, "0"],
    ["Thread/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Thread/changes",
        "path": "/created"
      }
    }, "1"],
    ["Thread/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Thread/changes",
        "path": "/updated"
      }
    }, "2"]
  ]
}
```

## Combined with Email sync

Typical pattern for syncing both:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/changes", {
      "accountId": "A123",
      "sinceState": "e12345"
    }, "0"],
    ["Thread/changes", {
      "accountId": "A123",
      "sinceState": "t12345"
    }, "1"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/changes",
        "path": "/created"
      },
      "properties": ["id", "threadId", "from", "subject", "receivedAt", "preview"]
    }, "2"],
    ["Thread/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "1",
        "name": "Thread/changes",
        "path": "/updated"
      }
    }, "3"]
  ]
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

Recovery: Refetch all threads for the desired mailbox using Email/query with collapseThreads.

## Looping for all changes

```javascript
async function syncAllThreadChanges(accountId, sinceState) {
  let state = sinceState;
  let allCreated = [];
  let allUpdated = [];
  let allDestroyed = [];

  while (true) {
    const response = await jmap.call('Thread/changes', {
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

## Thread state vs Email state

Thread state and Email state may change independently:
- Email marked as read → Email state changes, Thread state may not
- New reply arrives → Both states change
- Email deleted → Both states change
- Thread merged → Thread state changes

Always track both states separately for accurate sync.
