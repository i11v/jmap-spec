# Email/copy

Copy emails between accounts.

## Basic copy

Copy emails from one account to another:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "create": {
        "copy1": {
          "id": "M999",
          "mailboxIds": { "MBinbox456": true }
        }
      }
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "oldState": "s100",
      "newState": "s101",
      "created": {
        "copy1": {
          "id": "M1000",
          "blobId": "B2000",
          "threadId": "T500",
          "size": 12345
        }
      },
      "notCreated": null
    }, "0"]
  ]
}
```

## Copy with new keywords

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "create": {
        "copy1": {
          "id": "M999",
          "mailboxIds": { "MBinbox456": true },
          "keywords": { "$seen": true }
        }
      }
    }, "0"]
  ]
}
```

## Copy multiple emails

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "create": {
        "copy1": {
          "id": "M999",
          "mailboxIds": { "MBinbox456": true }
        },
        "copy2": {
          "id": "M1000",
          "mailboxIds": { "MBinbox456": true }
        },
        "copy3": {
          "id": "M1001",
          "mailboxIds": { "MBarchive456": true }
        }
      }
    }, "0"]
  ]
}
```

## Move (copy and delete)

Copy then destroy original:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "create": {
        "move1": {
          "id": "M999",
          "mailboxIds": { "MBinbox456": true }
        }
      },
      "onSuccessDestroyOriginal": true
    }, "0"]
  ]
}
```

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| fromAccountId | `Id` | Yes | Source account |
| ifFromInState | `String\|null` | No | Expected source state |
| accountId | `Id` | Yes | Destination account |
| ifInState | `String\|null` | No | Expected destination state |
| create | `Id[EmailCopy]` | Yes | Map of creation id to copy spec |
| onSuccessDestroyOriginal | `Boolean` | No | Delete source after copy |

## EmailCopy object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| id | `Id` | Yes | Source email id |
| mailboxIds | `Id[Boolean]` | Yes | Destination mailboxes |
| keywords | `String[Boolean]` | No | Keywords for copy (default: same as original) |
| receivedAt | `UTCDate` | No | Received date (default: same as original) |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| fromAccountId | `Id` | Source account |
| accountId | `Id` | Destination account |
| oldState | `String` | Destination state before |
| newState | `String` | Destination state after |
| created | `Id[Email]\|null` | Successfully copied emails |
| notCreated | `Id[SetError]\|null` | Copy errors |

## Created email properties

Server-set properties returned for each copy:

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | New email id in destination |
| blobId | `Id` | Blob id (may be same or new) |
| threadId | `Id` | Thread id in destination |
| size | `UnsignedInt` | Email size |

## Error cases

### Source not found

```json
{
  "notCreated": {
    "copy1": {
      "type": "notFound",
      "description": "Email not found in source account"
    }
  }
}
```

### Invalid mailbox

```json
{
  "notCreated": {
    "copy1": {
      "type": "invalidProperties",
      "properties": ["mailboxIds"],
      "description": "Mailbox not found in destination account"
    }
  }
}
```

### Forbidden

```json
{
  "notCreated": {
    "copy1": {
      "type": "forbidden",
      "description": "Cannot copy to this account"
    }
  }
}
```

### Over quota

```json
{
  "notCreated": {
    "copy1": {
      "type": "overQuota",
      "description": "Destination account storage quota exceeded"
    }
  }
}
```

### Too large

```json
{
  "notCreated": {
    "copy1": {
      "type": "tooLarge",
      "description": "Email exceeds destination account limits"
    }
  }
}
```

## State checking

Use `ifFromInState` and `ifInState` for optimistic locking:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/copy", {
      "fromAccountId": "A123",
      "ifFromInState": "s500",
      "accountId": "A456",
      "ifInState": "s100",
      "create": {
        "copy1": {
          "id": "M999",
          "mailboxIds": { "MBinbox456": true }
        }
      }
    }, "0"]
  ]
}
```

If states don't match:

```json
{
  "methodResponses": [
    ["error", {
      "type": "stateMismatch",
      "description": "State has changed"
    }, "0"]
  ]
}
```

## Thread handling

- Copied emails join existing threads in destination based on Message-ID/References
- A new thread is created if no matching thread exists
- Thread assignment is server-determined
