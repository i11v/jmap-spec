# Binary Data

JMAP provides separate endpoints for uploading and downloading binary data (blobs).

## Uploading binary data

Upload files using HTTP POST to the upload URL from the Session:

```http
POST /upload/{accountId}/ HTTP/1.1
Host: jmap.example.com
Authorization: Bearer <token>
Content-Type: image/png
Content-Length: 12345

<binary data>
```

```json
// response
{
  "accountId": "A123",
  "blobId": "B789xyz",
  "type": "image/png",
  "size": 12345
}
```

## Upload response

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account the blob was uploaded to |
| blobId | `Id` | Identifier for the uploaded blob |
| type | `String` | Media type of the uploaded data |
| size | `UnsignedInt` | Size in bytes |

## Upload limits

Uploads are limited by session capabilities:

| Limit | Property |
|-------|----------|
| Max file size | maxSizeUpload |
| Max concurrent uploads | maxConcurrentUpload |

## Upload error responses

### 400 Bad Request

Invalid request format.

### 401 Unauthorized

Authentication required.

### 403 Forbidden

Not authorized to upload to this account.

### 404 Not Found

Account not found.

### 413 Payload Too Large

File exceeds maxSizeUpload:

```http
HTTP/1.1 413 Payload Too Large

{
  "type": "urn:ietf:params:jmap:error:limit",
  "limit": "maxSizeUpload",
  "detail": "File size exceeds maximum of 50MB"
}
```

## Downloading binary data

Download files using HTTP GET with the download URL template:

```
https://jmap.example.com/download/{accountId}/{blobId}/{name}?type={type}
```

### URL template variables

| Variable | Description |
|----------|-------------|
| accountId | Account id |
| blobId | Blob identifier |
| name | Suggested filename for download |
| type | Media type for Content-Type header |

### Example download

```http
GET /download/A123/B789xyz/attachment.pdf?type=application/pdf HTTP/1.1
Host: jmap.example.com
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="attachment.pdf"
Content-Length: 12345

<binary data>
```

## Download headers

Response headers:

| Header | Value |
|--------|-------|
| Content-Type | From `type` parameter or stored type |
| Content-Disposition | `attachment; filename="<name>"` |
| Content-Length | Size in bytes |
| Cache-Control | May be cached |

## Download error responses

### 401 Unauthorized

Authentication required.

### 403 Forbidden

Not authorized to access this blob.

### 404 Not Found

Blob or account not found.

## Blob lifecycle

Blobs are reference-counted:
- Created when uploaded or as part of an Email
- Remain available while referenced by any object
- May be deleted when no longer referenced

## Blob references in Email

When creating emails, reference uploaded blobs for attachments:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/set", {
      "accountId": "A123",
      "create": {
        "draft": {
          "mailboxIds": { "MBdrafts": true },
          "from": [{ "email": "me@example.com" }],
          "to": [{ "email": "you@example.com" }],
          "subject": "File attached",
          "bodyStructure": {
            "type": "multipart/mixed",
            "subParts": [
              {
                "type": "text/plain",
                "partId": "body"
              },
              {
                "type": "application/pdf",
                "blobId": "B789xyz",
                "name": "document.pdf",
                "disposition": "attachment"
              }
            ]
          },
          "bodyValues": {
            "body": { "value": "Please see attached." }
          }
        }
      }
    }, "0"]
  ]
}
```

## Copying blobs between accounts

Use the Blob/copy method:

```json
{
  "using": ["urn:ietf:params:jmap:core"],
  "methodCalls": [
    ["Blob/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "blobIds": ["B789xyz", "B123abc"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Blob/copy", {
      "fromAccountId": "A123",
      "accountId": "A456",
      "copied": {
        "B789xyz": "B999new",
        "B123abc": "B888new"
      },
      "notCopied": null
    }, "0"]
  ]
}
```

## Blob/copy response

| Property | Type | Description |
|----------|------|-------------|
| fromAccountId | `Id` | Source account |
| accountId | `Id` | Destination account |
| copied | `Id[Id]\|null` | Map of source to new blob ids |
| notCopied | `Id[SetError]\|null` | Errors for failed copies |

## Blob/copy errors

| Error | Description |
|-------|-------------|
| notFound | Blob not found in source account |
| forbidden | Cannot access source or destination |
| overQuota | Destination account quota exceeded |
| tooLarge | Blob exceeds destination limits |
