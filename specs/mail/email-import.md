# Email/import

Import emails from uploaded blobs (RFC 5322 messages).

## Basic import

Import a single email from an uploaded blob:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/import", {
      "accountId": "A123",
      "emails": {
        "import1": {
          "blobId": "Bupload123",
          "mailboxIds": { "MBinbox": true },
          "keywords": { "$seen": true }
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
    ["Email/import", {
      "accountId": "A123",
      "oldState": "s500",
      "newState": "s501",
      "created": {
        "import1": {
          "id": "M999",
          "blobId": "B999",
          "threadId": "T100",
          "size": 12345
        }
      },
      "notCreated": null
    }, "0"]
  ]
}
```

## Import workflow

1. Upload the raw RFC 5322 message
2. Import the uploaded blob as an Email

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/import", {
      "accountId": "A123",
      "emails": {
        "import1": {
          "blobId": "Bupload123",
          "mailboxIds": { "MBinbox": true },
          "keywords": {}
        }
      }
    }, "0"]
  ]
}
```

## Import with specific date

Override the received date:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/import", {
      "accountId": "A123",
      "emails": {
        "import1": {
          "blobId": "Bupload123",
          "mailboxIds": { "MBarchive": true },
          "keywords": { "$seen": true },
          "receivedAt": "2020-01-15T10:30:00Z"
        }
      }
    }, "0"]
  ]
}
```

## Import multiple emails

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/import", {
      "accountId": "A123",
      "emails": {
        "import1": {
          "blobId": "Bupload123",
          "mailboxIds": { "MBinbox": true }
        },
        "import2": {
          "blobId": "Bupload124",
          "mailboxIds": { "MBinbox": true }
        },
        "import3": {
          "blobId": "Bupload125",
          "mailboxIds": { "MBarchive": true },
          "keywords": { "$seen": true }
        }
      }
    }, "0"]
  ]
}
```

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| accountId | `Id` | Yes | Account to import into |
| ifInState | `String\|null` | No | Expected state |
| emails | `Id[EmailImport]` | Yes | Map of creation id to import spec |

## EmailImport object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| blobId | `Id` | Yes | Blob containing RFC 5322 message |
| mailboxIds | `Id[Boolean]` | Yes | Mailboxes to add email to |
| keywords | `String[Boolean]` | No | Keywords (default: empty) |
| receivedAt | `UTCDate` | No | Received date (default: current time) |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account imported to |
| oldState | `String` | State before import |
| newState | `String` | State after import |
| created | `Id[Email]\|null` | Successfully imported emails |
| notCreated | `Id[SetError]\|null` | Import errors |

## Created email properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | New email id |
| blobId | `Id` | Email blob id |
| threadId | `Id` | Thread id |
| size | `UnsignedInt` | Email size in bytes |

## Error cases

### Blob not found

```json
{
  "notCreated": {
    "import1": {
      "type": "notFound",
      "description": "Blob not found"
    }
  }
}
```

### Invalid email

```json
{
  "notCreated": {
    "import1": {
      "type": "invalidEmail",
      "properties": [],
      "description": "Blob is not a valid RFC 5322 message"
    }
  }
}
```

### Invalid mailbox

```json
{
  "notCreated": {
    "import1": {
      "type": "invalidProperties",
      "properties": ["mailboxIds"],
      "description": "Mailbox not found"
    }
  }
}
```

### Too large

```json
{
  "notCreated": {
    "import1": {
      "type": "tooLarge",
      "description": "Email exceeds maximum size"
    }
  }
}
```

### Over quota

```json
{
  "notCreated": {
    "import1": {
      "type": "overQuota",
      "description": "Storage quota exceeded"
    }
  }
}
```

### Forbidden

```json
{
  "notCreated": {
    "import1": {
      "type": "forbidden",
      "description": "Cannot import to this mailbox"
    }
  }
}
```

## Migration use case

Importing emails during migration from another system:

```javascript
async function migrateEmails(jmapClient, emails, destMailboxId) {
  // Upload all messages first
  const uploads = await Promise.all(
    emails.map(email => jmapClient.upload(email.rawMessage))
  );

  // Import in batches
  const batchSize = 50;
  for (let i = 0; i < uploads.length; i += batchSize) {
    const batch = uploads.slice(i, i + batchSize);
    const importSpec = {};

    batch.forEach((upload, j) => {
      importSpec[`import${i + j}`] = {
        blobId: upload.blobId,
        mailboxIds: { [destMailboxId]: true },
        keywords: emails[i + j].keywords || {},
        receivedAt: emails[i + j].receivedAt
      };
    });

    await jmapClient.call('Email/import', {
      accountId: accountId,
      emails: importSpec
    });
  }
}
```

## Thread assignment

- Server assigns threadId based on Message-ID, In-Reply-To, References headers
- Imported emails join existing threads when possible
- A new thread is created if no matching thread exists

## Preserving dates

When migrating, preserve the original received date:

```json
{
  "import1": {
    "blobId": "Bupload123",
    "mailboxIds": { "MBarchive": true },
    "keywords": { "$seen": true },
    "receivedAt": "2015-06-20T14:30:00Z"
  }
}
```

If not specified, `receivedAt` defaults to the current server time.
