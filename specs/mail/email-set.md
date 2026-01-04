# Email/set

Standard "/set" method for creating, updating, and destroying Email objects as described in RFC 8620 Section 5.3.

## Create an email (draft)

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "create": {
        "draft-1": {
          "mailboxIds": { "MBdrafts123": true },
          "keywords": { "$draft": true },
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "Hello World",
          "bodyStructure": {
            "type": "text/plain",
            "partId": "1"
          },
          "bodyValues": {
            "1": {
              "value": "This is the email body."
            }
          }
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
    ["Email/set", {
      "accountId": "u33084183",
      "oldState": "123456",
      "newState": "123457",
      "created": {
        "draft-1": {
          "id": "Mdraft789",
          "blobId": "Bdraft789",
          "threadId": "Tdraft789",
          "size": 256
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

## Create HTML email

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "create": {
        "html-email": {
          "mailboxIds": { "MBdrafts123": true },
          "keywords": { "$draft": true },
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "HTML Email",
          "bodyStructure": {
            "type": "text/html",
            "partId": "1"
          },
          "bodyValues": {
            "1": {
              "value": "<html><body><h1>Hello</h1><p>This is <b>HTML</b> content.</p></body></html>"
            }
          }
        }
      }
    }, "0"]
  ]
}
```

## Create multipart email

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "create": {
        "multipart-email": {
          "mailboxIds": { "MBdrafts123": true },
          "keywords": { "$draft": true },
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "Multipart Email",
          "bodyStructure": {
            "type": "multipart/alternative",
            "subParts": [
              {
                "type": "text/plain",
                "partId": "text"
              },
              {
                "type": "text/html",
                "partId": "html"
              }
            ]
          },
          "bodyValues": {
            "text": { "value": "Plain text version" },
            "html": { "value": "<html><body><p>HTML version</p></body></html>" }
          }
        }
      }
    }, "0"]
  ]
}
```

## Create email with attachment

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "create": {
        "email-with-attachment": {
          "mailboxIds": { "MBdrafts123": true },
          "keywords": { "$draft": true },
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "Email with Attachment",
          "bodyStructure": {
            "type": "multipart/mixed",
            "subParts": [
              {
                "type": "text/plain",
                "partId": "body"
              },
              {
                "type": "application/pdf",
                "blobId": "Buploadedfile123",
                "name": "document.pdf",
                "disposition": "attachment"
              }
            ]
          },
          "bodyValues": {
            "body": { "value": "Please find the document attached." }
          }
        }
      }
    }, "0"]
  ]
}
```

## Update email keywords

### Mark as read

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "keywords/$seen": true
        }
      }
    }, "0"]
  ]
}
```

### Mark as unread

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "keywords/$seen": null
        }
      }
    }, "0"]
  ]
}
```

### Flag email

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "keywords/$flagged": true
        }
      }
    }, "0"]
  ]
}
```

### Replace all keywords

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "keywords": {
            "$seen": true,
            "$flagged": true,
            "$answered": true
          }
        }
      }
    }, "0"]
  ]
}
```

## Move email to mailbox

### Move to single mailbox

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "mailboxIds": { "MBarchive123": true }
        }
      }
    }, "0"]
  ]
}
```

### Add to mailbox (label)

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "mailboxIds/MBimportant789": true
        }
      }
    }, "0"]
  ]
}
```

### Remove from mailbox

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "mailboxIds/MBinbox123": null
        }
      }
    }, "0"]
  ]
}
```

### Move to trash

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": {
          "mailboxIds": { "MBtrash123": true }
        }
      }
    }, "0"]
  ]
}
```

## Destroy emails

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "destroy": ["Mf123u456", "Mf123u457"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/set", {
      "accountId": "u33084183",
      "oldState": "123456",
      "newState": "123458",
      "destroyed": ["Mf123u456", "Mf123u457"]
    }, "0"]
  ]
}
```

## Bulk operations

### Mark multiple as read

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": { "keywords/$seen": true },
        "Mf123u457": { "keywords/$seen": true },
        "Mf123u458": { "keywords/$seen": true }
      }
    }, "0"]
  ]
}
```

### Move multiple emails

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "u33084183",
      "update": {
        "Mf123u456": { "mailboxIds": { "MBarchive123": true } },
        "Mf123u457": { "mailboxIds": { "MBarchive123": true } }
      }
    }, "0"]
  ]
}
```

## Error cases

### Email not found

```json
// response
{
  "methodResponses": [
    ["Email/set", {
      "accountId": "u33084183",
      "notUpdated": {
        "nonexistent-id": {
          "type": "notFound",
          "description": "Email not found"
        }
      }
    }, "0"]
  ]
}
```

### Invalid mailbox

```json
// response
{
  "methodResponses": [
    ["Email/set", {
      "accountId": "u33084183",
      "notUpdated": {
        "Mf123u456": {
          "type": "invalidProperties",
          "properties": ["mailboxIds"],
          "description": "Mailbox id not found: MBinvalid123"
        }
      }
    }, "0"]
  ]
}
```

### No mailbox assigned

```json
// response
{
  "methodResponses": [
    ["Email/set", {
      "accountId": "u33084183",
      "notUpdated": {
        "Mf123u456": {
          "type": "invalidProperties",
          "properties": ["mailboxIds"],
          "description": "Email must belong to at least one mailbox"
        }
      }
    }, "0"]
  ]
}
```

### Forbidden operation

```json
// response
{
  "methodResponses": [
    ["Email/set", {
      "accountId": "u33084183",
      "notUpdated": {
        "Mf123u456": {
          "type": "forbidden",
          "description": "Cannot modify email in this mailbox"
        }
      }
    }, "0"]
  ]
}
```

## Standard keywords

| Keyword | Description |
|---------|-------------|
| $seen | Message has been read |
| $draft | Message is a draft |
| $flagged | Message is flagged/starred |
| $answered | Message has been replied to |
| $forwarded | Message has been forwarded |
| $phishing | Message is suspected phishing |
| $junk | Message is spam |
| $notjunk | Message is not spam |

## SetError types

| Type | Description |
|------|-------------|
| forbidden | No permission for this operation |
| notFound | Email id does not exist |
| invalidProperties | Invalid property values |
| tooLarge | Email is too large |
| tooManyKeywords | Too many keywords |
| invalidEmail | Email structure is invalid |
| invalidPatch | Invalid JSON patch |

## Creatable properties

When creating an email:

| Property | Required | Description |
|----------|----------|-------------|
| mailboxIds | Yes | Must be in at least one mailbox |
| keywords | No | Default: empty |
| receivedAt | No | Default: current time |
| messageId | No | Server may generate if omitted |
| from | No | Sender addresses |
| to | No | Recipient addresses |
| cc | No | CC addresses |
| bcc | No | BCC addresses |
| replyTo | No | Reply-To addresses |
| subject | No | Email subject |
| sentAt | No | Date header |
| bodyStructure | Yes* | MIME structure (*or use textBody/htmlBody) |
| bodyValues | Yes* | Body content by partId |
| textBody | Yes* | Simplified text body |
| htmlBody | Yes* | Simplified HTML body |
| attachments | No | Attachment references |

## Updatable properties

| Property | Updatable | Notes |
|----------|-----------|-------|
| mailboxIds | Yes | Can move/copy between mailboxes |
| keywords | Yes | Can add/remove keywords |
| All others | No | Immutable after creation |
