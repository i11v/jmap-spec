# Mailbox/queryChanges

Standard "/queryChanges" method for getting changes to a Mailbox query result.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "filter": {
        "isSubscribed": true
      },
      "sort": [
        { "property": "name", "isAscending": true }
      ],
      "sinceQueryState": "q12345"
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12350",
      "total": 18,
      "removed": ["MB789"],
      "added": [
        { "id": "MB999", "index": 5 }
      ]
    }, "0"]
  ]
}
```

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| accountId | `Id` | Yes | Account to query |
| filter | `FilterOperator\|FilterCondition\|null` | No | Same filter as original query |
| sort | `Comparator[]\|null` | No | Same sort as original query |
| sinceQueryState | `String` | Yes | Query state from previous query |
| maxChanges | `UnsignedInt\|null` | No | Maximum changes to return |
| upToId | `Id\|null` | No | Only return changes up to this id |
| calculateTotal | `Boolean` | No | Include new total count |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldQueryState | `String` | State from request |
| newQueryState | `String` | Current query state |
| total | `UnsignedInt\|null` | New total (if calculateTotal=true) |
| removed | `Id[]` | Ids removed from results |
| added | `AddedItem[]` | Items added with their positions |

## AddedItem

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Mailbox id |
| index | `UnsignedInt` | New position in results |

## Example: Mailbox renamed

Mailbox renamed from "Alpha" to "Zebra" (moves in sorted list):

```json
{
  "methodResponses": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 15,
      "removed": ["MB456"],
      "added": [
        { "id": "MB456", "index": 14 }
      ]
    }, "0"]
  ]
}
```

The same id appears in both `removed` and `added` because its position changed.

## Example: New mailbox created

```json
{
  "methodResponses": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 16,
      "removed": [],
      "added": [
        { "id": "MB999", "index": 8 }
      ]
    }, "0"]
  ]
}
```

## Example: Mailbox deleted

```json
{
  "methodResponses": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 14,
      "removed": ["MB789"],
      "added": []
    }, "0"]
  ]
}
```

## With maxChanges

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "filter": { "isSubscribed": true },
      "sort": [{ "property": "name", "isAscending": true }],
      "sinceQueryState": "q12345",
      "maxChanges": 10
    }, "0"]
  ]
}
```

If there are more changes than `maxChanges`, client must re-run the full query.

## Error: cannotCalculateChanges

```json
{
  "methodResponses": [
    ["error", {
      "type": "cannotCalculateChanges",
      "description": "Query state too old or filter/sort changed"
    }, "0"]
  ]
}
```

Causes:
- Query state is too old
- Server cannot efficiently calculate changes for this filter/sort
- `canCalculateChanges` was false on the original query

Client must re-run the full `Mailbox/query`.

## Applying changes to cached results

1. Remove all ids in `removed` from cached list
2. Insert items from `added` at their specified index
3. Update `total` if provided

```javascript
// Pseudo-code for applying changes
function applyChanges(cachedIds, removed, added) {
  // Remove items
  cachedIds = cachedIds.filter(id => !removed.includes(id));

  // Add items at their positions (sorted by index descending to maintain positions)
  added.sort((a, b) => b.index - a.index);
  for (const item of added) {
    cachedIds.splice(item.index, 0, item.id);
  }

  return cachedIds;
}
```

## Sync workflow with queryChanges

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Mailbox/queryChanges", {
      "accountId": "A123",
      "filter": { "isSubscribed": true },
      "sort": [{ "property": "name", "isAscending": true }],
      "sinceQueryState": "q12345",
      "calculateTotal": true
    }, "0"],
    ["Mailbox/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Mailbox/queryChanges",
        "path": "/added/*/id"
      }
    }, "1"]
  ]
}
```
