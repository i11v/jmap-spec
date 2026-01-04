# SearchSnippet/get

Get highlighted search snippets for emails matching a query filter.

## Basic usage

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "A123",
      "filter": {
        "text": "quarterly report"
      },
      "limit": 10
    }, "0"],
    ["SearchSnippet/get", {
      "accountId": "A123",
      "filter": {
        "text": "quarterly report"
      },
      "#emailIds": {
        "resultOf": "0",
        "name": "Email/query",
        "path": "/ids"
      }
    }, "1"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Email/query", {
      "accountId": "A123",
      "queryState": "q12345",
      "ids": ["M100", "M200", "M300"]
    }, "0"],
    ["SearchSnippet/get", {
      "accountId": "A123",
      "list": [
        {
          "emailId": "M100",
          "subject": "Q4 <mark>Quarterly</mark> <mark>Report</mark>",
          "preview": "Please find attached the <mark>quarterly</mark> <mark>report</mark> for Q4..."
        },
        {
          "emailId": "M200",
          "subject": null,
          "preview": "The <mark>quarterly</mark> financial <mark>report</mark> shows..."
        },
        {
          "emailId": "M300",
          "subject": "<mark>Quarterly</mark> Review Meeting",
          "preview": "Let's discuss the <mark>report</mark> in our meeting..."
        }
      ],
      "notFound": []
    }, "1"]
  ]
}
```

## Request arguments

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| accountId | `Id` | Yes | Account to query |
| filter | `FilterOperator\|FilterCondition` | Yes | Same filter used in Email/query |
| emailIds | `Id[]` | Yes | Email ids to get snippets for |

## Response properties

| Property | Type | Description |
|----------|------|-------------|
| accountId | `Id` | Account queried |
| list | `SearchSnippet[]` | Snippets for found emails |
| notFound | `Id[]\|null` | Email ids not found |

## SearchSnippet object

| Property | Type | Description |
|----------|------|-------------|
| emailId | `Id` | Email this snippet is for |
| subject | `String\|null` | Highlighted subject, or null if no match |
| preview | `String\|null` | Highlighted body preview, or null if no match |

## Highlighting format

Matching terms are wrapped in `<mark>` tags:

- Input filter: `"text": "quarterly report"`
- Subject: `"Q4 <mark>Quarterly</mark> <mark>Report</mark>"`
- Preview: `"The <mark>quarterly</mark> financial <mark>report</mark> shows growth..."`

## Combined workflow

Typical search workflow:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Email/query", {
      "accountId": "A123",
      "filter": {
        "text": "project alpha"
      },
      "sort": [{ "property": "receivedAt", "isAscending": false }],
      "limit": 20,
      "calculateTotal": true
    }, "q"],
    ["Email/get", {
      "accountId": "A123",
      "#ids": {
        "resultOf": "q",
        "name": "Email/query",
        "path": "/ids"
      },
      "properties": ["id", "threadId", "from", "to", "subject", "receivedAt", "hasAttachment", "keywords"]
    }, "g"],
    ["SearchSnippet/get", {
      "accountId": "A123",
      "filter": {
        "text": "project alpha"
      },
      "#emailIds": {
        "resultOf": "q",
        "name": "Email/query",
        "path": "/ids"
      }
    }, "s"]
  ]
}
```

## Filter must match

The filter in SearchSnippet/get must match the filter used in Email/query for accurate highlighting:

```json
{
  "filter": {
    "operator": "AND",
    "conditions": [
      { "inMailbox": "MBinbox" },
      { "text": "important meeting" }
    ]
  }
}
```

## Multiple search terms

```json
{
  "filter": {
    "text": "budget review 2024"
  }
}
```

Each matching term is highlighted separately:
```json
{
  "preview": "The <mark>2024</mark> <mark>budget</mark> <mark>review</mark> process..."
}
```

## No match in subject

When the search term isn't in the subject:

```json
{
  "emailId": "M100",
  "subject": null,
  "preview": "Found the <mark>keyword</mark> in the body..."
}
```

## No match in body

When the search term is only in the subject:

```json
{
  "emailId": "M100",
  "subject": "Meeting about <mark>Project</mark> <mark>X</mark>",
  "preview": null
}
```

## Full-text vs header search

SearchSnippet works best with `text` filter (full-text search).

For header-specific searches, snippets may be limited:

```json
{
  "filter": {
    "from": "alice@example.com"
  }
}
```

The server may not highlight "alice@example.com" in the preview since it searched headers only.

## Performance considerations

- Request snippets only for visible search results
- Use pagination (limit) to avoid fetching too many
- Cache snippets on client for same query

## HTML escaping

The snippet text is HTML-safe except for `<mark>` tags:
- Special characters (`<`, `>`, `&`) are escaped
- Only `<mark>` and `</mark>` tags are present
- Safe to insert directly into HTML UI

## Server variations

Snippet behavior varies by server:
- Context length around matches
- Number of matches shown
- Stemming/synonym highlighting
- HTML vs plain text source

The spec doesn't mandate specific behavior, only the format.
