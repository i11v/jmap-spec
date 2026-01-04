# Mailbox/set

Standard "/set" method for creating, updating, and destroying Mailbox objects as described in RFC 8620 Section 5.3.

## Create a mailbox

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "create": {
        "new-mailbox-1": {
          "name": "Project Alpha",
          "parentId": null
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
    ["Mailbox/set", {
      "accountId": "u33084183",
      "oldState": "78540",
      "newState": "78541",
      "created": {
        "new-mailbox-1": {
          "id": "MB789abc123",
          "sortOrder": 0,
          "totalEmails": 0,
          "unreadEmails": 0,
          "totalThreads": 0,
          "unreadThreads": 0,
          "myRights": {
            "mayReadItems": true,
            "mayAddItems": true,
            "mayRemoveItems": true,
            "maySetSeen": true,
            "maySetKeywords": true,
            "mayCreateChild": true,
            "mayRename": true,
            "mayDelete": true,
            "maySubmit": true
          },
          "isSubscribed": true
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

## Create nested mailbox

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "create": {
        "parent": {
          "name": "Projects",
          "parentId": null
        },
        "child": {
          "name": "Project Alpha",
          "parentId": "#parent"
        }
      }
    }, "0"]
  ]
}
```

## Create with all properties

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "create": {
        "new-mailbox-1": {
          "name": "Important",
          "parentId": null,
          "sortOrder": 5,
          "isSubscribed": true
        }
      }
    }, "0"]
  ]
}
```

## Update a mailbox

### Rename

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "update": {
        "MB789abc123": {
          "name": "Project Beta"
        }
      }
    }, "0"]
  ]
}
```

### Move to different parent

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "update": {
        "MB789abc123": {
          "parentId": "MB456def789"
        }
      }
    }, "0"]
  ]
}
```

### Move to top level

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "update": {
        "MB789abc123": {
          "parentId": null
        }
      }
    }, "0"]
  ]
}
```

### Update sort order

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "update": {
        "MB789abc123": {
          "sortOrder": 10
        }
      }
    }, "0"]
  ]
}
```

### Update subscription

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "update": {
        "MB789abc123": {
          "isSubscribed": false
        }
      }
    }, "0"]
  ]
}
```

## Destroy a mailbox

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "destroy": ["MB789abc123"]
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "oldState": "78541",
      "newState": "78542",
      "created": null,
      "updated": null,
      "destroyed": ["MB789abc123"],
      "notCreated": null,
      "notUpdated": null,
      "notDestroyed": null
    }, "0"]
  ]
}
```

## Destroy with emails

Remove mailbox and all its emails:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "destroy": ["MB789abc123"],
      "onDestroyRemoveEmails": true
    }, "0"]
  ]
}
```

## Multiple operations

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "create": {
        "new-1": { "name": "New Folder", "parentId": null }
      },
      "update": {
        "MB123": { "name": "Renamed Folder" }
      },
      "destroy": ["MB456"]
    }, "0"]
  ]
}
```

## Error cases

### Mailbox has child

```json
// request to destroy parent with children
{
  "destroy": ["MBparent123"]
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "oldState": "78541",
      "newState": "78541",
      "notDestroyed": {
        "MBparent123": {
          "type": "mailboxHasChild",
          "description": "Mailbox still has child mailboxes"
        }
      }
    }, "0"]
  ]
}
```

### Mailbox has emails

```json
// request without onDestroyRemoveEmails
{
  "destroy": ["MB123"]
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "oldState": "78541",
      "newState": "78541",
      "notDestroyed": {
        "MB123": {
          "type": "mailboxHasEmail",
          "description": "Mailbox has emails and onDestroyRemoveEmails is false"
        }
      }
    }, "0"]
  ]
}
```

### Duplicate name

```json
// request with existing name
{
  "create": {
    "new-1": { "name": "Inbox", "parentId": null }
  }
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "notCreated": {
        "new-1": {
          "type": "invalidProperties",
          "properties": ["name"],
          "description": "A mailbox with this name already exists at this level"
        }
      }
    }, "0"]
  ]
}
```

### Invalid parent

```json
// request with non-existent parent
{
  "create": {
    "new-1": { "name": "Child", "parentId": "nonexistent" }
  }
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "notCreated": {
        "new-1": {
          "type": "invalidProperties",
          "properties": ["parentId"],
          "description": "Parent mailbox not found"
        }
      }
    }, "0"]
  ]
}
```

### Cannot modify role

```json
// request to change role
{
  "update": {
    "MBinbox": { "role": "drafts" }
  }
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "notUpdated": {
        "MBinbox": {
          "type": "invalidProperties",
          "properties": ["role"],
          "description": "Cannot modify the role property"
        }
      }
    }, "0"]
  ]
}
```

### Insufficient permissions

```json
// request without permission
{
  "destroy": ["MBinbox"]
}
```

```json
// error response
{
  "methodResponses": [
    ["Mailbox/set", {
      "accountId": "u33084183",
      "notDestroyed": {
        "MBinbox": {
          "type": "forbidden",
          "description": "You do not have permission to delete this mailbox"
        }
      }
    }, "0"]
  ]
}
```

## SetError types

| Type | Description |
|------|-------------|
| forbidden | No permission for this operation |
| notFound | Mailbox id does not exist |
| invalidProperties | Invalid property values |
| mailboxHasChild | Cannot destroy: has child mailboxes |
| mailboxHasEmail | Cannot destroy: has emails (and onDestroyRemoveEmails=false) |
| tooManyMailboxes | Account mailbox limit reached |
| invalidPatch | Invalid JSON patch |

## Creatable/updatable properties

| Property | Create | Update |
|----------|--------|--------|
| name | Required | Yes |
| parentId | Optional (null) | Yes |
| role | No | No (immutable after server-set) |
| sortOrder | Optional (0) | Yes |
| isSubscribed | Optional (true) | Yes |
