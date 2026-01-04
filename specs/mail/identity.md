# Identity

An Identity represents a sender identity for submitting emails. It contains information about the "From" address and other metadata used when sending.

## Identity/get

Retrieve identities:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["Identity/get", {
      "accountId": "A123",
      "ids": null
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Identity/get", {
      "accountId": "A123",
      "state": "i100",
      "list": [
        {
          "id": "I001",
          "name": "Alice Smith",
          "email": "alice@example.com",
          "replyTo": null,
          "bcc": null,
          "textSignature": "Best regards,\nAlice",
          "htmlSignature": "<p>Best regards,<br>Alice</p>",
          "mayDelete": false
        },
        {
          "id": "I002",
          "name": "Alice (Work)",
          "email": "alice.smith@company.com",
          "replyTo": [{ "email": "support@company.com" }],
          "bcc": [{ "email": "archive@company.com" }],
          "textSignature": "Alice Smith\nSenior Engineer",
          "htmlSignature": "<p><b>Alice Smith</b><br>Senior Engineer</p>",
          "mayDelete": true
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Identity properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Immutable, server-set |
| name | `String` | Display name for From header |
| email | `String` | Email address for From header |
| replyTo | `EmailAddress[]\|null` | Default Reply-To addresses |
| bcc | `EmailAddress[]\|null` | Auto-added BCC addresses |
| textSignature | `String` | Plain text signature |
| htmlSignature | `String` | HTML signature |
| mayDelete | `Boolean` | Server-set. Can this identity be deleted? |

## Identity/changes

Get changes since a previous state:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["Identity/changes", {
      "accountId": "A123",
      "sinceState": "i100"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Identity/changes", {
      "accountId": "A123",
      "oldState": "i100",
      "newState": "i105",
      "hasMoreChanges": false,
      "created": ["I003"],
      "updated": ["I001"],
      "destroyed": []
    }, "0"]
  ]
}
```

## Identity/set

### Create identity

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["Identity/set", {
      "accountId": "A123",
      "create": {
        "new-id": {
          "name": "Alice (Personal)",
          "email": "alice.personal@example.com",
          "textSignature": "- Alice",
          "htmlSignature": "<p>- Alice</p>"
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
    ["Identity/set", {
      "accountId": "A123",
      "oldState": "i100",
      "newState": "i101",
      "created": {
        "new-id": {
          "id": "I003",
          "mayDelete": true
        }
      },
      "updated": null,
      "destroyed": null,
      "notCreated": null,
      "notUpdated": null,
      "notDestroyed": null
    }, "0"]
  ]
}
```

### Update identity

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["Identity/set", {
      "accountId": "A123",
      "update": {
        "I001": {
          "name": "Alice S.",
          "textSignature": "Cheers,\nAlice"
        }
      }
    }, "0"]
  ]
}
```

### Destroy identity

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["Identity/set", {
      "accountId": "A123",
      "destroy": ["I002"]
    }, "0"]
  ]
}
```

## Creatable/Updatable properties

| Property | Create | Update |
|----------|--------|--------|
| name | Yes | Yes |
| email | Yes | Server-dependent |
| replyTo | Yes | Yes |
| bcc | Yes | Yes |
| textSignature | Yes | Yes |
| htmlSignature | Yes | Yes |

## Error cases

### Forbidden email

```json
{
  "notCreated": {
    "new-id": {
      "type": "forbiddenFrom",
      "description": "Cannot create identity with this email address"
    }
  }
}
```

### Cannot delete

```json
{
  "notDestroyed": {
    "I001": {
      "type": "forbidden",
      "description": "This identity cannot be deleted"
    }
  }
}
```

### Too many identities

```json
{
  "notCreated": {
    "new-id": {
      "type": "overQuota",
      "description": "Maximum number of identities reached"
    }
  }
}
```

## Using identity in EmailSubmission

Reference identity when sending:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail", "urn:ietf:params:jmap:submission"],
  "methodCalls": [
    ["EmailSubmission/set", {
      "accountId": "A123",
      "create": {
        "send1": {
          "emailId": "Mdraft123",
          "identityId": "I001"
        }
      }
    }, "0"]
  ]
}
```

## Signatures

Signatures are stored but NOT automatically added to emails. Clients should:

1. Fetch the identity
2. Append signature when composing
3. Use textSignature for plain text emails
4. Use htmlSignature for HTML emails

```javascript
function createEmailWithSignature(identity, bodyText) {
  const fullBody = bodyText + '\n\n' + identity.textSignature;
  return {
    from: [{ name: identity.name, email: identity.email }],
    bodyStructure: { type: 'text/plain', partId: '1' },
    bodyValues: { '1': { value: fullBody } }
  };
}
```

## Auto BCC

When an identity has `bcc` set, clients SHOULD automatically add those addresses to emails sent with that identity. The server does NOT automatically add them.

## Default identity

The first identity returned (or one marked in a vendor extension) is typically the default. Use it when composing new messages.
