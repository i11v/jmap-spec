# Push Notifications

JMAP supports push notifications to alert clients when data changes, avoiding the need for polling.

## Push mechanisms

JMAP provides two push mechanisms:

1. **EventSource** - Server-sent events over HTTP
2. **PushSubscription** - Web Push via callback URL

## EventSource

Connect to the eventSourceUrl from the Session using Server-Sent Events:

```
https://jmap.example.com/events?types={types}&closeafter={closeafter}&ping={ping}
```

### URL parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| types | `String` | Comma-separated type names (e.g., "Email,Mailbox") or "*" for all |
| closeafter | `String` | "state" to close after StateChange, or "no" |
| ping | `UnsignedInt` | Seconds between keep-alive pings (0 to disable) |

### Connection example

```javascript
const url = `https://jmap.example.com/events?types=Email,Mailbox&closeafter=no&ping=30`;
const eventSource = new EventSource(url, {
  headers: { 'Authorization': 'Bearer <token>' }
});

eventSource.addEventListener('state', (event) => {
  const change = JSON.parse(event.data);
  console.log('State changed:', change);
});

eventSource.addEventListener('ping', (event) => {
  console.log('Ping received');
});
```

### StateChange event

```json
{
  "type": "StateChange",
  "changed": {
    "A123": {
      "Email": "s456",
      "Mailbox": "s789",
      "Thread": "s012"
    }
  }
}
```

### StateChange properties

| Property | Type | Description |
|----------|------|-------------|
| type | `String` | Always "StateChange" |
| changed | `Id[TypeState]` | Map of accountId to changed types |

### TypeState

Map of type name to new state string:

```json
{
  "Email": "s456",
  "Mailbox": "s789"
}
```

## PushSubscription

Register a callback URL to receive push notifications.

### Create subscription

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/set", {
      "create": {
        "push1": {
          "deviceClientId": "device123",
          "url": "https://push.example.com/callback",
          "types": ["Email", "Mailbox"],
          "expires": "2024-02-01T00:00:00Z"
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
    ["PushSubscription/set", {
      "created": {
        "push1": {
          "id": "PS789",
          "keys": {
            "p256dh": "BNcRd...",
            "auth": "tBHI..."
          },
          "expires": "2024-02-01T00:00:00Z"
        }
      }
    }, "0"]
  ]
}
```

### PushSubscription properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Subscription identifier |
| deviceClientId | `String` | Client-provided device identifier |
| url | `String` | Callback URL for push notifications |
| keys | `Object\|null` | Encryption keys if provided |
| types | `String[]\|null` | Types to notify, or null for all |
| expires | `UTCDate\|null` | Expiration time |
| verificationCode | `String` | Code sent to verify URL |

### Verify subscription

After creating, verify ownership of the callback URL:

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/set", {
      "update": {
        "PS789": {
          "verificationCode": "abc123"
        }
      }
    }, "0"]
  ]
}
```

### Push notification format

Notifications are sent as HTTP POST to the callback URL:

```http
POST /callback HTTP/1.1
Host: push.example.com
Content-Type: application/json
TTL: 86400

{
  "@type": "PushVerification",
  "pushSubscriptionId": "PS789",
  "verificationCode": "abc123"
}
```

Or for state changes:

```http
POST /callback HTTP/1.1
Host: push.example.com
Content-Type: application/json
TTL: 86400

{
  "@type": "StateChange",
  "changed": {
    "A123": {
      "Email": "s456"
    }
  }
}
```

### Web Push encryption

For encrypted Web Push, provide keys:

```json
{
  "create": {
    "push1": {
      "deviceClientId": "device123",
      "url": "https://fcm.googleapis.com/...",
      "keys": {
        "p256dh": "BNcRd...",
        "auth": "tBHI..."
      },
      "types": null
    }
  }
}
```

## PushSubscription/get

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/get", {
      "ids": null
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["PushSubscription/get", {
      "list": [
        {
          "id": "PS789",
          "deviceClientId": "device123",
          "url": "https://push.example.com/callback",
          "types": ["Email", "Mailbox"],
          "expires": "2024-02-01T00:00:00Z"
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## PushSubscription/set

### Update expiration

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/set", {
      "update": {
        "PS789": {
          "expires": "2024-03-01T00:00:00Z"
        }
      }
    }, "0"]
  ]
}
```

### Update types

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/set", {
      "update": {
        "PS789": {
          "types": ["Email"]
        }
      }
    }, "0"]
  ]
}
```

### Delete subscription

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["PushSubscription/set", {
      "destroy": ["PS789"]
    }, "0"]
  ]
}
```

## Handling push notifications

1. Receive StateChange notification
2. Compare type states with local state
3. Call /changes for each changed type
4. Fetch updated objects

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/changes", {
      "accountId": "A123",
      "sinceState": "s123"
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/changes",
        "path": "/created"
      }
    }, "1"]
  ]
}
```

## Special types

### EmailDelivery

For mail, servers must support "EmailDelivery" type:
- Changes only when new Email arrives
- Does NOT change for reads, deletes, moves
- Useful for battery-efficient new mail notifications

```json
{
  "create": {
    "push1": {
      "deviceClientId": "device123",
      "url": "https://push.example.com/callback",
      "types": ["EmailDelivery"]
    }
  }
}
```
