# Session

The JMAP Session resource provides information about the server's capabilities and the accounts available to the authenticated user. A client must first fetch the Session to discover API endpoints and account information.

## Fetching the session

```http
GET /.well-known/jmap HTTP/1.1
Host: jmap.example.com
Authorization: Bearer <token>
```

```json
// response
{
  "capabilities": {
    "urn:ietf:params:jmap:core": {
      "maxSizeUpload": 50000000,
      "maxConcurrentUpload": 8,
      "maxSizeRequest": 10000000,
      "maxConcurrentRequests": 8,
      "maxCallsInRequest": 32,
      "maxObjectsInGet": 256,
      "maxObjectsInSet": 128,
      "collationAlgorithms": [
        "i;ascii-casemap",
        "i;ascii-numeric",
        "i;unicode-casemap"
      ]
    },
    "urn:ietf:params:jmap:mail": {},
    "urn:ietf:params:jmap:submission": {}
  },
  "accounts": {
    "u33084183": {
      "name": "alice@example.com",
      "isPersonal": true,
      "isReadOnly": false,
      "accountCapabilities": {
        "urn:ietf:params:jmap:mail": {
          "maxMailboxesPerEmail": null,
          "maxMailboxDepth": 10,
          "maxSizeMailboxName": 490,
          "maxSizeAttachmentsPerEmail": 50000000,
          "emailQuerySortOptions": [
            "receivedAt",
            "sentAt",
            "size",
            "from",
            "to",
            "subject"
          ],
          "mayCreateTopLevelMailbox": true
        },
        "urn:ietf:params:jmap:submission": {
          "maxDelayedSend": 86400,
          "submissionExtensions": {}
        }
      }
    },
    "u12345678": {
      "name": "Shared Mailbox",
      "isPersonal": false,
      "isReadOnly": true,
      "accountCapabilities": {
        "urn:ietf:params:jmap:mail": {
          "maxMailboxesPerEmail": null,
          "maxMailboxDepth": 10,
          "maxSizeMailboxName": 490,
          "maxSizeAttachmentsPerEmail": 50000000,
          "emailQuerySortOptions": ["receivedAt", "sentAt"],
          "mayCreateTopLevelMailbox": false
        }
      }
    }
  },
  "primaryAccounts": {
    "urn:ietf:params:jmap:mail": "u33084183",
    "urn:ietf:params:jmap:submission": "u33084183"
  },
  "username": "alice@example.com",
  "apiUrl": "https://jmap.example.com/api/",
  "downloadUrl": "https://jmap.example.com/download/{accountId}/{blobId}/{name}?type={type}",
  "uploadUrl": "https://jmap.example.com/upload/{accountId}/",
  "eventSourceUrl": "https://jmap.example.com/eventsource/?types={types}&closeafter={closeafter}&ping={ping}",
  "state": "abc123"
}
```

## Session properties

| Property | Type | Description |
|----------|------|-------------|
| capabilities | `String[Object]` | Server capabilities map |
| accounts | `Id[Account]` | Accounts the user can access |
| primaryAccounts | `String[Id]` | Default account for each capability |
| username | `String` | Authenticated username |
| apiUrl | `String` | URL for API requests |
| downloadUrl | `String` | URL template for downloads |
| uploadUrl | `String` | URL template for uploads |
| eventSourceUrl | `String` | URL template for push events |
| state | `String` | Session state string |

## Core capability (urn:ietf:params:jmap:core)

| Property | Type | Description |
|----------|------|-------------|
| maxSizeUpload | `UnsignedInt` | Max file upload size in bytes |
| maxConcurrentUpload | `UnsignedInt` | Max concurrent uploads |
| maxSizeRequest | `UnsignedInt` | Max API request size in bytes |
| maxConcurrentRequests | `UnsignedInt` | Max concurrent API requests |
| maxCallsInRequest | `UnsignedInt` | Max method calls per request |
| maxObjectsInGet | `UnsignedInt` | Max objects in /get call |
| maxObjectsInSet | `UnsignedInt` | Max objects in /set call |
| collationAlgorithms | `String[]` | Supported sort collations |

## Account object

| Property | Type | Description |
|----------|------|-------------|
| name | `String` | User-friendly account name |
| isPersonal | `Boolean` | True if user's own account |
| isReadOnly | `Boolean` | True if entire account is read-only |
| accountCapabilities | `String[Object]` | Capabilities for this account |

## Mail capability (urn:ietf:params:jmap:mail)

| Property | Type | Description |
|----------|------|-------------|
| maxMailboxesPerEmail | `UnsignedInt\|null` | Max mailboxes per email |
| maxMailboxDepth | `UnsignedInt\|null` | Max mailbox hierarchy depth |
| maxSizeMailboxName | `UnsignedInt` | Max mailbox name length (bytes) |
| maxSizeAttachmentsPerEmail | `UnsignedInt` | Max attachment size per email |
| emailQuerySortOptions | `String[]` | Supported Email/query sort properties |
| mayCreateTopLevelMailbox | `Boolean` | Can create root mailboxes |

## Submission capability (urn:ietf:params:jmap:submission)

| Property | Type | Description |
|----------|------|-------------|
| maxDelayedSend | `UnsignedInt` | Max send delay in seconds (0 = not supported) |
| submissionExtensions | `String[String[]]` | SMTP extensions (ehlo-name to ehlo-args) |

## Service autodiscovery

Clients can discover the JMAP Session URL from a domain using:

```
https://<domain>/.well-known/jmap
```

Or via DNS SRV record:

```
_jmap._tcp.<domain>
```

## Session state changes

The `state` property changes when:
- An account is added or removed
- Account capabilities change
- Primary account assignments change
- Server capabilities change

Clients can detect this via the `sessionState` in API responses and refetch the Session when needed.

## Example with multiple capabilities

```json
{
  "capabilities": {
    "urn:ietf:params:jmap:core": {
      "maxSizeUpload": 50000000,
      "maxConcurrentUpload": 4,
      "maxSizeRequest": 10000000,
      "maxConcurrentRequests": 4,
      "maxCallsInRequest": 16,
      "maxObjectsInGet": 500,
      "maxObjectsInSet": 500,
      "collationAlgorithms": ["i;ascii-casemap", "i;unicode-casemap"]
    },
    "urn:ietf:params:jmap:mail": {},
    "urn:ietf:params:jmap:submission": {},
    "urn:ietf:params:jmap:vacationresponse": {},
    "https://vendor.example/custom-extension": {
      "customProperty": "value"
    }
  },
  "accounts": {
    "A123": {
      "name": "user@example.com",
      "isPersonal": true,
      "isReadOnly": false,
      "accountCapabilities": {
        "urn:ietf:params:jmap:mail": {},
        "urn:ietf:params:jmap:submission": {},
        "urn:ietf:params:jmap:vacationresponse": {}
      }
    }
  },
  "primaryAccounts": {
    "urn:ietf:params:jmap:mail": "A123",
    "urn:ietf:params:jmap:submission": "A123",
    "urn:ietf:params:jmap:vacationresponse": "A123"
  },
  "username": "user@example.com",
  "apiUrl": "https://api.example.com/jmap/",
  "downloadUrl": "https://api.example.com/download/{accountId}/{blobId}/{name}?type={type}",
  "uploadUrl": "https://api.example.com/upload/{accountId}/",
  "eventSourceUrl": "https://api.example.com/events?types={types}&closeafter={closeafter}&ping={ping}",
  "state": "s1234567890"
}
```
