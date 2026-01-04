# EmailSubmission

The EmailSubmission object represents the submission of an email for delivery.
This is used to send emails through the JMAP server.

## Submit an email

Basic email submission:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "create": {
        "send-1": {
          "emailId": "Mdraft123",
          "identityId": "I12345"
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
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "oldState": "100",
      "newState": "101",
      "created": {
        "send-1": {
          "id": "ES789",
          "sendAt": "2024-01-15T10:30:00Z"
        }
      }
    }, "0"]
  ]
}
```

## Create email and send

Create an email and send it in one request:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "create": {
        "draft-1": {
          "mailboxIds": { "MBdrafts123": true },
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "Hello",
          "bodyStructure": {
            "type": "text/plain",
            "partId": "1"
          },
          "bodyValues": {
            "1": { "value": "Hello Bob!" }
          }
        }
      }
    }, "0"],
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "create": {
        "send-1": {
          "emailId": "#draft-1",
          "identityId": "I12345"
        }
      },
      "onSuccessUpdateEmail": {
        "#send-1": {
          "mailboxIds/MBdrafts123": null,
          "mailboxIds/MBsent456": true,
          "keywords/$draft": null
        }
      }
    }, "1"]
  ]
}
```

## Send with envelope

Override the envelope (recipients):

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "create": {
        "send-1": {
          "emailId": "Mdraft123",
          "identityId": "I12345",
          "envelope": {
            "mailFrom": {
              "email": "alice@example.com",
              "parameters": null
            },
            "rcptTo": [
              { "email": "bob@example.com", "parameters": null },
              { "email": "carol@example.com", "parameters": null }
            ]
          }
        }
      }
    }, "0"]
  ]
}
```

## Delayed send

Schedule email for later delivery:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "create": {
        "send-1": {
          "emailId": "Mdraft123",
          "identityId": "I12345",
          "sendAt": "2024-01-20T09:00:00Z"
        }
      }
    }, "0"]
  ]
}
```

## Cancel delayed send

Cancel a scheduled email:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["EmailSubmission/set", {
      "accountId": "u33084183",
      "update": {
        "ES789": {
          "undoStatus": "canceled"
        }
      }
    }, "0"]
  ]
}
```

## Get submission status

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["EmailSubmission/get", {
      "accountId": "u33084183",
      "ids": ["ES789"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/get", {
      "accountId": "u33084183",
      "state": "101",
      "list": [
        {
          "id": "ES789",
          "identityId": "I12345",
          "emailId": "Mdraft123",
          "threadId": "T456",
          "envelope": {
            "mailFrom": { "email": "alice@example.com", "parameters": null },
            "rcptTo": [{ "email": "bob@example.com", "parameters": null }]
          },
          "sendAt": "2024-01-15T10:30:00Z",
          "undoStatus": "final",
          "deliveryStatus": {
            "bob@example.com": {
              "smtpReply": "250 2.0.0 OK",
              "delivered": "yes",
              "displayed": "unknown"
            }
          },
          "dsnBlobIds": [],
          "mdnBlobIds": []
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## onSuccessUpdateEmail

Move sent email to Sent folder and remove draft flag:

```json
{
  "onSuccessUpdateEmail": {
    "#send-1": {
      "mailboxIds/MBdrafts123": null,
      "mailboxIds/MBsent456": true,
      "keywords/$draft": null,
      "keywords/$seen": true
    }
  }
}
```

## onSuccessDestroyEmail

Delete the draft after sending:

```json
{
  "onSuccessDestroyEmail": ["#send-1"]
}
```

## EmailSubmission properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Immutable, server-set |
| identityId | `Id` | Required. Identity to send from |
| emailId | `Id` | Required. Email to send |
| threadId | `Id` | Immutable, server-set |
| envelope | `Envelope\|null` | Override envelope, or derive from headers |
| sendAt | `UTCDate` | When to send (immediate if omitted) |
| undoStatus | `String` | "pending", "final", or "canceled" |
| deliveryStatus | `String[DeliveryStatus]\|null` | Per-recipient status |
| dsnBlobIds | `Id[]` | Delivery Status Notification blobs |
| mdnBlobIds | `Id[]` | Message Disposition Notification blobs |

## Envelope object

| Property | Type | Description |
|----------|------|-------------|
| mailFrom | `Address` | Sender address |
| rcptTo | `Address[]` | Recipient addresses |

## Address object

| Property | Type | Description |
|----------|------|-------------|
| email | `String` | Email address |
| parameters | `Object\|null` | SMTP parameters |

## DeliveryStatus object

| Property | Type | Description |
|----------|------|-------------|
| smtpReply | `String` | SMTP reply string |
| delivered | `String` | "queued", "yes", "no", "unknown" |
| displayed | `String` | "yes", "unknown" |

## UndoStatus values

| Status | Description |
|--------|-------------|
| pending | Submission is queued, can be canceled |
| final | Submission complete, cannot be undone |
| canceled | Submission was canceled before sending |

## Error cases

### Invalid identity

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/set", {
      "notCreated": {
        "send-1": {
          "type": "invalidProperties",
          "properties": ["identityId"],
          "description": "Identity not found"
        }
      }
    }, "0"]
  ]
}
```

### Invalid email

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/set", {
      "notCreated": {
        "send-1": {
          "type": "invalidProperties",
          "properties": ["emailId"],
          "description": "Email not found"
        }
      }
    }, "0"]
  ]
}
```

### No recipients

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/set", {
      "notCreated": {
        "send-1": {
          "type": "invalidEmail",
          "description": "Email has no recipients"
        }
      }
    }, "0"]
  ]
}
```

### Forbidden sender

```json
// response
{
  "methodResponses": [
    ["EmailSubmission/set", {
      "notCreated": {
        "send-1": {
          "type": "forbiddenFrom",
          "description": "Cannot send from this address"
        }
      }
    }, "0"]
  ]
}
```

## Identity/get

Get available sending identities:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:submission"
  ],
  "methodCalls": [
    ["Identity/get", {
      "accountId": "u33084183",
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
      "accountId": "u33084183",
      "state": "50",
      "list": [
        {
          "id": "I12345",
          "name": "Alice Smith",
          "email": "alice@example.com",
          "replyTo": null,
          "bcc": null,
          "textSignature": "Best regards,\nAlice",
          "htmlSignature": "<p>Best regards,<br>Alice</p>",
          "mayDelete": false
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```
