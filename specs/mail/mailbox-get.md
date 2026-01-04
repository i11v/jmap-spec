# Mailbox/get

Standard "/get" method for retrieving Mailbox objects as described in RFC 8620 Section 5.1.
The `ids` argument may be `null` to fetch all mailboxes at once.

## Basic mailbox retrieval

Fetch all mailboxes in an account:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/get", {
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
    ["Mailbox/get", {
      "accountId": "u33084183",
      "state": "78540",
      "list": [
        {
          "id": "MB23cfa8094c0f41e6",
          "name": "Inbox",
          "parentId": null,
          "role": "inbox",
          "sortOrder": 10,
          "totalEmails": 16307,
          "unreadEmails": 13905,
          "totalThreads": 5833,
          "unreadThreads": 5128,
          "myRights": {
            "mayAddItems": true,
            "mayRename": false,
            "maySubmit": true,
            "mayDelete": false,
            "maySetKeywords": true,
            "mayRemoveItems": true,
            "mayCreateChild": true,
            "maySetSeen": true,
            "mayReadItems": true
          },
          "isSubscribed": true
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Fetch specific mailboxes by ID

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/get", {
      "accountId": "u33084183",
      "ids": ["MB23cfa8094c0f41e6", "MB674cc24095db49ce"]
    }, "0"]
  ]
}
```

## Fetch specific properties only

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/get", {
      "accountId": "u33084183",
      "ids": null,
      "properties": ["id", "name", "role", "totalEmails", "unreadEmails"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/get", {
      "accountId": "u33084183",
      "state": "78540",
      "list": [
        {
          "id": "MB23cfa8094c0f41e6",
          "name": "Inbox",
          "role": "inbox",
          "totalEmails": 16307,
          "unreadEmails": 13905
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## Not found IDs

When requesting mailboxes that don't exist, they appear in `notFound`:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/get", {
      "accountId": "u33084183",
      "ids": ["MB23cfa8094c0f41e6", "nonexistent-id"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/get", {
      "accountId": "u33084183",
      "state": "78540",
      "list": [
        {
          "id": "MB23cfa8094c0f41e6",
          "name": "Inbox"
        }
      ],
      "notFound": ["nonexistent-id"]
    }, "0"]
  ]
}
```

## Mailbox properties

A Mailbox object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Immutable, server-set. The id of the Mailbox. |
| name | `String` | User-visible name. Must be Net-Unicode, at least 1 char. |
| parentId | `Id\|null` | Parent mailbox id, or null for top-level. |
| role | `String\|null` | Special role (inbox, drafts, sent, trash, etc.) |
| sortOrder | `UnsignedInt` | Display order (0 to 2^31-1). Lower = displayed first. |
| totalEmails | `UnsignedInt` | Server-set. Number of emails in this mailbox. |
| unreadEmails | `UnsignedInt` | Server-set. Emails without $seen or $draft keywords. |
| totalThreads | `UnsignedInt` | Server-set. Threads with at least one email here. |
| unreadThreads | `UnsignedInt` | Server-set. Unread threads in this mailbox. |
| myRights | `MailboxRights` | Server-set. ACL permissions for this mailbox. |
| isSubscribed | `Boolean` | User's subscription preference. |

## MailboxRights properties

| Property | Type | Description |
|----------|------|-------------|
| mayReadItems | `Boolean` | Can use mailbox in Email/query filter. |
| mayAddItems | `Boolean` | Can add mail to this mailbox. |
| mayRemoveItems | `Boolean` | Can remove mail from this mailbox. |
| maySetSeen | `Boolean` | Can add/remove $seen keyword. |
| maySetKeywords | `Boolean` | Can add/remove keywords (except $seen). |
| mayCreateChild | `Boolean` | Can create child mailboxes. |
| mayRename | `Boolean` | Can rename or move this mailbox. |
| mayDelete | `Boolean` | Can delete this mailbox. |
| maySubmit | `Boolean` | Can submit messages to this mailbox. |

## Role values

Standard mailbox roles (from IMAP SPECIAL-USE):

- `inbox` - The inbox
- `drafts` - Drafts folder
- `sent` - Sent messages
- `trash` - Deleted messages
- `archive` - Archived messages
- `junk` - Spam/junk folder
- `important` - Important messages
- `all` - Virtual mailbox containing all messages
