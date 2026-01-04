# Standard Methods

JMAP defines standard method patterns for all data types. Each type implements some or all of these methods.

## Foo/get

Retrieve objects by id.

### Request

```json
["Foo/get", {
  "accountId": "A123",
  "ids": ["F1", "F2", "F3"],
  "properties": ["id", "name", "value"]
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Required. Account to query. |
| ids | `Id[]\|null` | Object ids to fetch, or null for all. |
| properties | `String[]\|null` | Properties to return, or null for all. |

### Response

```json
["Foo/get", {
  "accountId": "A123",
  "state": "s12345",
  "list": [
    { "id": "F1", "name": "First", "value": 100 },
    { "id": "F2", "name": "Second", "value": 200 }
  ],
  "notFound": ["F3"]
}, "0"]
```

### Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| state | `String` | Current state string |
| list | `Foo[]` | Array of found objects |
| notFound | `Id[]` | Ids that were not found |

## Foo/changes

Get changes since a previous state.

### Request

```json
["Foo/changes", {
  "accountId": "A123",
  "sinceState": "s12345",
  "maxChanges": 100
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Required. Account to query. |
| sinceState | `String` | Required. Previous state string. |
| maxChanges | `UnsignedInt\|null` | Max changes to return. |

### Response

```json
["Foo/changes", {
  "accountId": "A123",
  "oldState": "s12345",
  "newState": "s12350",
  "hasMoreChanges": false,
  "created": ["F4", "F5"],
  "updated": ["F1"],
  "destroyed": ["F2"]
}, "0"]
```

### Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldState | `String` | State from request |
| newState | `String` | Current state |
| hasMoreChanges | `Boolean` | More changes available |
| created | `Id[]` | Newly created object ids |
| updated | `Id[]` | Updated object ids |
| destroyed | `Id[]` | Destroyed object ids |

## Foo/set

Create, update, and destroy objects.

### Request

```json
["Foo/set", {
  "accountId": "A123",
  "ifInState": "s12345",
  "create": {
    "new-1": { "name": "New Object", "value": 50 }
  },
  "update": {
    "F1": { "name": "Updated Name" }
  },
  "destroy": ["F2"]
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Required. Account to modify. |
| ifInState | `String\|null` | Expected state (optimistic locking). |
| create | `Id[Foo]\|null` | Map of creation id to object. |
| update | `Id[PatchObject]\|null` | Map of id to patches. |
| destroy | `Id[]\|null` | Ids to destroy. |

### Response

```json
["Foo/set", {
  "accountId": "A123",
  "oldState": "s12345",
  "newState": "s12346",
  "created": {
    "new-1": {
      "id": "F6",
      "name": "New Object",
      "value": 50
    }
  },
  "updated": {
    "F1": null
  },
  "destroyed": ["F2"],
  "notCreated": null,
  "notUpdated": null,
  "notDestroyed": null
}, "0"]
```

### Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account modified |
| oldState | `String` | State before changes |
| newState | `String` | State after changes |
| created | `Id[Foo]\|null` | Created objects with server-set values |
| updated | `Id[Foo\|null]\|null` | Updated ids (value has changed props) |
| destroyed | `Id[]\|null` | Destroyed ids |
| notCreated | `Id[SetError]\|null` | Creation errors |
| notUpdated | `Id[SetError]\|null` | Update errors |
| notDestroyed | `Id[SetError]\|null` | Destroy errors |

## Foo/copy

Copy objects between accounts.

### Request

```json
["Foo/copy", {
  "fromAccountId": "A123",
  "ifFromInState": "s12345",
  "accountId": "A456",
  "ifInState": "s67890",
  "create": {
    "copy-1": "F1",
    "copy-2": "F2"
  },
  "onSuccessDestroyOriginal": false
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| fromAccountId | `Id` | Source account |
| ifFromInState | `String\|null` | Expected source state |
| accountId | `Id` | Destination account |
| ifInState | `String\|null` | Expected destination state |
| create | `Id[Id]` | Map of new id to source id |
| onSuccessDestroyOriginal | `Boolean` | Delete source after copy |

### Response

```json
["Foo/copy", {
  "fromAccountId": "A123",
  "accountId": "A456",
  "oldState": "s67890",
  "newState": "s67891",
  "created": {
    "copy-1": { "id": "F100", ... }
  },
  "notCreated": {
    "copy-2": { "type": "notFound" }
  }
}, "0"]
```

## Foo/query

Search and filter objects.

### Request

```json
["Foo/query", {
  "accountId": "A123",
  "filter": {
    "name": "test"
  },
  "sort": [
    { "property": "name", "isAscending": true }
  ],
  "position": 0,
  "limit": 10,
  "calculateTotal": true
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Required. Account to query. |
| filter | `FilterOperator\|FilterCondition\|null` | Filter criteria |
| sort | `Comparator[]\|null` | Sort order |
| position | `Int` | Start index (default: 0) |
| anchor | `Id\|null` | Id to position relative to |
| anchorOffset | `Int` | Offset from anchor (default: 0) |
| limit | `UnsignedInt\|null` | Max results |
| calculateTotal | `Boolean` | Include total count (default: false) |

### Response

```json
["Foo/query", {
  "accountId": "A123",
  "queryState": "q12345",
  "canCalculateChanges": true,
  "position": 0,
  "total": 42,
  "ids": ["F1", "F3", "F5"]
}, "0"]
```

### Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| queryState | `String` | State for queryChanges |
| canCalculateChanges | `Boolean` | queryChanges supported |
| position | `UnsignedInt` | Index of first result |
| total | `UnsignedInt\|null` | Total matching (if requested) |
| ids | `Id[]` | Matching object ids |

## Foo/queryChanges

Get changes to a query result.

### Request

```json
["Foo/queryChanges", {
  "accountId": "A123",
  "filter": { "name": "test" },
  "sort": [{ "property": "name", "isAscending": true }],
  "sinceQueryState": "q12345",
  "maxChanges": 100,
  "upToId": null,
  "calculateTotal": false
}, "0"]
```

### Arguments

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Required. Account to query. |
| filter | `FilterOperator\|FilterCondition\|null` | Same filter as query |
| sort | `Comparator[]\|null` | Same sort as query |
| sinceQueryState | `String` | Previous queryState |
| maxChanges | `UnsignedInt\|null` | Max changes |
| upToId | `Id\|null` | Limit changes to before this id |
| calculateTotal | `Boolean` | Include total |

### Response

```json
["Foo/queryChanges", {
  "accountId": "A123",
  "oldQueryState": "q12345",
  "newQueryState": "q12350",
  "total": 45,
  "removed": ["F3"],
  "added": [
    { "id": "F10", "index": 2 }
  ]
}, "0"]
```

### Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldQueryState | `String` | State from request |
| newQueryState | `String` | Current query state |
| total | `UnsignedInt\|null` | New total (if requested) |
| removed | `Id[]` | Ids removed from results |
| added | `AddedItem[]` | Items added with positions |

### AddedItem

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Object id |
| index | `UnsignedInt` | Position in results |

## Filter operators

Combine filters with AND, OR, NOT:

```json
{
  "operator": "AND",
  "conditions": [
    { "name": "test" },
    { "value": 100 }
  ]
}
```

```json
{
  "operator": "OR",
  "conditions": [
    { "name": "foo" },
    { "name": "bar" }
  ]
}
```

```json
{
  "operator": "NOT",
  "conditions": [
    { "isArchived": true }
  ]
}
```

## Comparator

```json
{
  "property": "name",
  "isAscending": true,
  "collation": "i;unicode-casemap"
}
```

| Property | Type | Description |
|----------|------|-------------|
| property | `String` | Property to sort by |
| isAscending | `Boolean` | Ascending order (default: true) |
| collation | `String\|null` | Collation algorithm |
