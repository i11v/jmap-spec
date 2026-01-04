# Email/parse

Parse email blobs without creating Email objects. Useful for previewing attachments that are emails or analyzing uploaded messages.

## Basic parse

Parse an email from a blob:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bblob123"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/parse", {
      "accountId": "A123",
      "parsed": {
        "Bblob123": {
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "subject": "Hello World",
          "sentAt": "2024-01-15T10:30:00Z",
          "preview": "This is a preview of the email...",
          "textBody": [
            { "partId": "1", "type": "text/plain" }
          ],
          "htmlBody": [],
          "attachments": [],
          "hasAttachment": false
        }
      },
      "notParseable": []
    }, "0"]
  ]
}
```

## Parse with specific properties

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bblob123"],
      "properties": ["from", "to", "subject", "bodyStructure", "bodyValues"],
      "bodyProperties": ["partId", "type", "size", "name"],
      "fetchAllBodyValues": true
    }, "0"]
  ]
}
```

## Parse multiple blobs

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bblob123", "Bblob456", "Bblob789"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/parse", {
      "accountId": "A123",
      "parsed": {
        "Bblob123": { "subject": "First email", ... },
        "Bblob456": { "subject": "Second email", ... }
      },
      "notParseable": ["Bblob789"]
    }, "0"]
  ]
}
```

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| accountId | `Id` | Yes | Account context (for blob access) |
| blobIds | `Id[]` | Yes | Blobs to parse |
| properties | `String[]\|null` | No | Email properties to return |
| bodyProperties | `String[]\|null` | No | Body part properties to return |
| fetchTextBodyValues | `Boolean` | No | Include text body values |
| fetchHTMLBodyValues | `Boolean` | No | Include HTML body values |
| fetchAllBodyValues | `Boolean` | No | Include all body values |
| maxBodyValueBytes | `UnsignedInt` | No | Max body value size |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account used |
| parsed | `Id[Email]\|null` | Successfully parsed emails |
| notParseable | `Id[]` | Blobs that couldn't be parsed |

## Parsing attached emails

When an email has an attached .eml file:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "A123",
      "ids": ["M999"],
      "properties": ["attachments"]
    }, "0"],
    ["Email/parse", {
      "accountId": "A123",
      "#blobIds": {
        "resultOf": "0",
        "name": "Email/get",
        "path": "/list/0/attachments/*/blobId"
      },
      "properties": ["from", "to", "subject", "sentAt", "preview"]
    }, "1"]
  ]
}
```

## Parse uploaded message

Preview before importing:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bupload999"],
      "properties": ["from", "to", "cc", "subject", "sentAt", "bodyStructure", "attachments"],
      "bodyProperties": ["partId", "type", "size", "name", "disposition"]
    }, "0"]
  ]
}
```

## Full body structure

Get complete MIME structure:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/parse", {
      "accountId": "A123",
      "blobIds": ["Bblob123"],
      "properties": ["bodyStructure", "bodyValues"],
      "bodyProperties": ["partId", "blobId", "type", "charset", "size", "name", "disposition", "cid", "language", "location"],
      "fetchAllBodyValues": true,
      "maxBodyValueBytes": 10000
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/parse", {
      "accountId": "A123",
      "parsed": {
        "Bblob123": {
          "bodyStructure": {
            "type": "multipart/mixed",
            "subParts": [
              {
                "partId": "1",
                "type": "multipart/alternative",
                "subParts": [
                  { "partId": "1.1", "type": "text/plain", "size": 500 },
                  { "partId": "1.2", "type": "text/html", "size": 1500 }
                ]
              },
              {
                "partId": "2",
                "type": "application/pdf",
                "size": 50000,
                "name": "document.pdf",
                "disposition": "attachment"
              }
            ]
          },
          "bodyValues": {
            "1.1": { "value": "Plain text content...", "isTruncated": false },
            "1.2": { "value": "<html><body>...", "isTruncated": false }
          }
        }
      },
      "notParseable": []
    }, "0"]
  ]
}
```

## Properties available

All Email properties can be requested except:
- `id` - not assigned
- `blobId` - use the input blobId
- `threadId` - not assigned
- `mailboxIds` - not assigned
- `keywords` - not assigned
- `receivedAt` - not assigned (use sentAt from headers)

## Error handling

Blobs that can't be parsed as RFC 5322 appear in `notParseable`:

```json
{
  "methodResponses": [
    ["Email/parse", {
      "accountId": "A123",
      "parsed": {},
      "notParseable": ["Binvalid123"]
    }, "0"]
  ]
}
```

## Use cases

1. **Preview attached emails** - Parse .eml attachments
2. **Migration preview** - Check emails before importing
3. **Email analysis** - Extract headers and structure
4. **Validation** - Verify message format before sending
