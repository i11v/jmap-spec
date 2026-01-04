# Email/queryChanges

Standard "/queryChanges" method for getting changes to an Email query result.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "filter": {
        "inMailbox": "MBinbox"
      },
      "sort": [
        { "property": "receivedAt", "isAscending": false }
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
    ["Email/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12400",
      "total": 1250,
      "removed": ["M100", "M101"],
      "added": [
        { "id": "M999", "index": 0 },
        { "id": "M998", "index": 1 }
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
| collapseThreads | `Boolean` | No | Must match original query |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| oldQueryState | `String` | State from request |
| newQueryState | `String` | Current query state |
| total | `UnsignedInt\|null` | New total (if requested) |
| removed | `Id[]` | Ids removed from results |
| added | `AddedItem[]` | Items added with positions |

## AddedItem

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Email id |
| index | `UnsignedInt` | Position in results |

## With collapseThreads

When using thread collapsing, changes reflect the collapsed view:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "filter": { "inMailbox": "MBinbox" },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "sinceQueryState": "q12345",
      "collapseThreads": true
    }, "0"]
  ]
}
```

## Example: New email arrives

New email at top of inbox:

```json
{
  "methodResponses": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 1251,
      "removed": [],
      "added": [
        { "id": "M999", "index": 0 }
      ]
    }, "0"]
  ]
}
```

## Example: Email moved out of mailbox

```json
{
  "methodResponses": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 1249,
      "removed": ["M500"],
      "added": []
    }, "0"]
  ]
}
```

## Example: Email changes position

Email marked as flagged moves in flagged-first sort:

```json
{
  "methodResponses": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "oldQueryState": "q12345",
      "newQueryState": "q12346",
      "total": 1250,
      "removed": ["M500"],
      "added": [
        { "id": "M500", "index": 0 }
      ]
    }, "0"]
  ]
}
```

## Using upToId

Only get changes affecting the visible portion of the list:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "filter": { "inMailbox": "MBinbox" },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "sinceQueryState": "q12345",
      "upToId": "M900"
    }, "0"]
  ]
}
```

Changes beyond `upToId` are not reported.

## Sync workflow

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/queryChanges", {
      "accountId": "A123",
      "filter": { "inMailbox": "MBinbox" },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "sinceQueryState": "q12345",
      "calculateTotal": true
    }, "0"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "0",
        "name": "Email/queryChanges",
        "path": "/added/*/id"
      },
      "properties": ["id", "threadId", "mailboxIds", "keywords", "from", "subject", "receivedAt", "preview"]
    }, "1"]
  ]
}
```

## Error: cannotCalculateChanges

```json
{
  "methodResponses": [
    ["error", {
      "type": "cannotCalculateChanges",
      "description": "Cannot calculate changes for this query"
    }, "0"]
  ]
}
```

Causes:
- Query state is too old
- Filter/sort combination doesn't support changes
- `canCalculateChanges` was false on original query
- Too many changes to report efficiently

Recovery: Re-run the full `Email/query`.

## Applying changes

```javascript
function applyQueryChanges(cachedIds, removed, added) {
  // Remove items
  let result = cachedIds.filter(id => !removed.includes(id));

  // Add items at positions (sort descending by index)
  const sortedAdded = [...added].sort((a, b) => b.index - a.index);
  for (const item of sortedAdded) {
    result.splice(item.index, 0, item.id);
  }

  return result;
}
```

## Performance considerations

For large mailboxes:
- Use `maxChanges` to limit response size
- Use `upToId` to limit to visible items
- Consider using `Email/changes` for full sync
- Use collapsed threads for overview lists
