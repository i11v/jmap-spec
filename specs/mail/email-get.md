# Email/get

Standard "/get" method for retrieving Email objects as described in RFC 8620 Section 5.1.
Supports additional arguments for controlling body part fetching.

## Basic email retrieval

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456", "Mf123u457"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/get", {
      "accountId": "u33084183",
      "state": "456789",
      "list": [
        {
          "id": "Mf123u456",
          "blobId": "B1234567890",
          "threadId": "T1234",
          "mailboxIds": { "MB23cfa8094c0f41e6": true },
          "keywords": { "$seen": true, "$flagged": true },
          "size": 12345,
          "receivedAt": "2024-01-15T10:30:00Z",
          "messageId": ["msg123@example.com"],
          "inReplyTo": null,
          "references": null,
          "sender": null,
          "from": [{ "name": "Alice", "email": "alice@example.com" }],
          "to": [{ "name": "Bob", "email": "bob@example.com" }],
          "cc": null,
          "bcc": null,
          "replyTo": null,
          "subject": "Hello World",
          "sentAt": "2024-01-15T10:29:00Z",
          "hasAttachment": false,
          "preview": "This is a preview of the email content..."
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Fetch specific properties

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "threadId", "from", "subject", "receivedAt"]
    }, "0"]
  ]
}
```

## Fetch with body values

### Text body only

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "subject", "textBody", "bodyValues"],
      "fetchTextBodyValues": true
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/get", {
      "accountId": "u33084183",
      "state": "456789",
      "list": [
        {
          "id": "Mf123u456",
          "subject": "Hello World",
          "textBody": [
            {
              "partId": "1",
              "blobId": "B1234567890-1",
              "size": 256,
              "type": "text/plain"
            }
          ],
          "bodyValues": {
            "1": {
              "value": "This is the plain text content of the email.",
              "isEncodingProblem": false,
              "isTruncated": false
            }
          }
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

### HTML body

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "subject", "htmlBody", "bodyValues"],
      "fetchHTMLBodyValues": true
    }, "0"]
  ]
}
```

### Truncated body values

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "subject", "textBody", "bodyValues"],
      "fetchTextBodyValues": true,
      "maxBodyValueBytes": 256
    }, "0"]
  ]
}
```

```json
// response with truncation
{
  "methodResponses": [
    ["Email/get", {
      "accountId": "u33084183",
      "state": "456789",
      "list": [
        {
          "id": "Mf123u456",
          "subject": "Long Email",
          "textBody": [{ "partId": "1", "blobId": "B123", "size": 10000, "type": "text/plain" }],
          "bodyValues": {
            "1": {
              "value": "This is the beginning of a very long email...",
              "isEncodingProblem": false,
              "isTruncated": true
            }
          }
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Fetch body structure

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "bodyStructure"],
      "bodyProperties": ["partId", "blobId", "size", "name", "type", "charset", "disposition", "cid"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/get", {
      "accountId": "u33084183",
      "state": "456789",
      "list": [
        {
          "id": "Mf123u456",
          "bodyStructure": {
            "partId": null,
            "blobId": null,
            "size": 12345,
            "type": "multipart/mixed",
            "subParts": [
              {
                "partId": "1",
                "blobId": "B123-1",
                "size": 256,
                "type": "text/plain",
                "charset": "utf-8",
                "disposition": null,
                "cid": null
              },
              {
                "partId": "2",
                "blobId": "B123-2",
                "size": 10240,
                "name": "report.pdf",
                "type": "application/pdf",
                "disposition": "attachment",
                "cid": null
              }
            ]
          }
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Fetch attachments list

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": ["id", "subject", "attachments", "hasAttachment"]
    }, "0"]
  ]
}
```

## Fetch custom headers

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/get", {
      "accountId": "u33084183",
      "ids": ["Mf123u456"],
      "properties": [
        "id",
        "header:List-Unsubscribe",
        "header:List-Unsubscribe:asURLs",
        "header:X-Priority"
      ]
    }, "0"]
  ]
}
```

## Email object properties

### Metadata properties (fast to fetch)

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Immutable, server-set |
| blobId | `Id` | Immutable, server-set. Blob id of raw RFC5322 message |
| threadId | `Id` | Immutable, server-set |
| mailboxIds | `Id[Boolean]` | Map of mailbox ids this email belongs to |
| keywords | `String[Boolean]` | Map of keywords ($seen, $flagged, $draft, etc.) |
| size | `UnsignedInt` | Immutable, server-set. Size in bytes |
| receivedAt | `UTCDate` | Immutable, server-set. When received by server |

### Header properties (fast to fetch)

| Property | Type | Description |
|----------|------|-------------|
| messageId | `String[]|null` | Message-ID header values |
| inReplyTo | `String[]|null` | In-Reply-To header values |
| references | `String[]|null` | References header values |
| sender | `EmailAddress[]|null` | Sender header |
| from | `EmailAddress[]|null` | From header |
| to | `EmailAddress[]|null` | To header |
| cc | `EmailAddress[]|null` | Cc header |
| bcc | `EmailAddress[]|null` | Bcc header |
| replyTo | `EmailAddress[]|null` | Reply-To header |
| subject | `String|null` | Subject header |
| sentAt | `Date|null` | Date header |

### Body properties (may be slower)

| Property | Type | Description |
|----------|------|-------------|
| bodyStructure | `EmailBodyPart` | Full MIME structure |
| bodyValues | `String[EmailBodyValue]` | Map of partId to body content |
| textBody | `EmailBodyPart[]` | Text parts for display |
| htmlBody | `EmailBodyPart[]` | HTML parts for display |
| attachments | `EmailBodyPart[]` | Attachment parts |
| hasAttachment | `Boolean` | Has downloadable attachments |
| preview | `String` | Plaintext preview (max 256 chars) |

## Default properties

If `properties` is omitted, these are returned:

```json
["id", "blobId", "threadId", "mailboxIds", "keywords", "size",
 "receivedAt", "messageId", "inReplyTo", "references", "sender",
 "from", "to", "cc", "bcc", "replyTo", "subject", "sentAt",
 "hasAttachment", "preview", "bodyValues", "textBody", "htmlBody",
 "attachments"]
```

## Header parsing forms

Custom headers can be requested in different parsed forms:

| Suffix | Description | Example |
|--------|-------------|---------|
| (none) | Raw form | `header:Subject` |
| :asText | Decoded text | `header:Subject:asText` |
| :asAddresses | Parsed addresses | `header:From:asAddresses` |
| :asMessageIds | Parsed message IDs | `header:References:asMessageIds` |
| :asDate | Parsed date | `header:Date:asDate` |
| :asURLs | Parsed URLs | `header:List-Unsubscribe:asURLs` |
| :all | All instances (array) | `header:Received:all` |
