# VacationResponse

The VacationResponse object controls automatic out-of-office replies (vacation responder).

## VacationResponse/get

Get the current vacation response settings:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/get", {
      "accountId": "A123",
      "ids": null
    }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["VacationResponse/get", {
      "accountId": "A123",
      "state": "v100",
      "list": [
        {
          "id": "singleton",
          "isEnabled": true,
          "fromDate": "2024-01-20T00:00:00Z",
          "toDate": "2024-01-27T23:59:59Z",
          "subject": "Out of Office",
          "textBody": "I am currently out of the office with limited access to email. I will respond to your message when I return on January 28th.\n\nFor urgent matters, please contact support@example.com.",
          "htmlBody": "<p>I am currently out of the office with limited access to email. I will respond to your message when I return on January 28th.</p><p>For urgent matters, please contact <a href=\"mailto:support@example.com\">support@example.com</a>.</p>"
        }
      ],
      "notFound": []
    }, "0"]
  ]
}
```

## VacationResponse properties

| Property | Type | Description |
|----------|------|-------------|
| id | `Id` | Always "singleton" - only one exists per account |
| isEnabled | `Boolean` | Whether auto-replies are active |
| fromDate | `UTCDate\|null` | Start sending replies after this time |
| toDate | `UTCDate\|null` | Stop sending replies after this time |
| subject | `String\|null` | Subject line for auto-reply |
| textBody | `String\|null` | Plain text body of auto-reply |
| htmlBody | `String\|null` | HTML body of auto-reply |

## VacationResponse/set

### Enable vacation response

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "update": {
        "singleton": {
          "isEnabled": true,
          "fromDate": "2024-01-20T00:00:00Z",
          "toDate": "2024-01-27T23:59:59Z",
          "subject": "Out of Office",
          "textBody": "I am currently out of the office...",
          "htmlBody": "<p>I am currently out of the office...</p>"
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
    ["VacationResponse/set", {
      "accountId": "A123",
      "oldState": "v100",
      "newState": "v101",
      "updated": {
        "singleton": null
      }
    }, "0"]
  ]
}
```

### Disable vacation response

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "update": {
        "singleton": {
          "isEnabled": false
        }
      }
    }, "0"]
  ]
}
```

### Update message only

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "update": {
        "singleton": {
          "subject": "Updated: Out of Office",
          "textBody": "I am away until February 1st...",
          "htmlBody": "<p>I am away until February 1st...</p>"
        }
      }
    }, "0"]
  ]
}
```

### Clear dates (always active when enabled)

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "update": {
        "singleton": {
          "isEnabled": true,
          "fromDate": null,
          "toDate": null
        }
      }
    }, "0"]
  ]
}
```

## Singleton behavior

VacationResponse is a singleton type:
- Only one VacationResponse exists per account
- The id is always "singleton"
- Cannot create new VacationResponse objects
- Cannot destroy the VacationResponse
- Can only update it

### Error: singleton

Attempting to create:

```json
{
  "methodResponses": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "notCreated": {
        "new-vr": {
          "type": "singleton",
          "description": "Only one VacationResponse may exist"
        }
      }
    }, "0"]
  ]
}
```

Attempting to destroy:

```json
{
  "methodResponses": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "notDestroyed": {
        "singleton": {
          "type": "singleton",
          "description": "VacationResponse cannot be destroyed"
        }
      }
    }, "0"]
  ]
}
```

## Date range behavior

| fromDate | toDate | Behavior |
|----------|--------|----------|
| null | null | Active whenever isEnabled=true |
| set | null | Active from fromDate onward |
| null | set | Active until toDate |
| set | set | Active during the date range |

## Reply behavior

When active, the server automatically replies:
- Once per sender (typically per 7 days)
- Only to emails addressed directly to the user
- Not to mailing lists, spam, or automated messages
- Using the configured subject and body

## textBody vs htmlBody

| Field | Usage |
|-------|-------|
| textBody | Used for plain text emails |
| htmlBody | Used for HTML-capable clients |

Best practice: Set both for maximum compatibility.

## Checking status

Quick check if vacation is active:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/get", {
      "accountId": "A123",
      "ids": ["singleton"],
      "properties": ["isEnabled", "fromDate", "toDate"]
    }, "0"]
  ]
}
```

## Capability

The `urn:ietf:params:jmap:vacationresponse` capability must be present in both:
- Session capabilities
- Account's accountCapabilities

Check before using:

```javascript
function hasVacationResponse(session, accountId) {
  return session.capabilities['urn:ietf:params:jmap:vacationresponse'] &&
         session.accounts[accountId]?.accountCapabilities['urn:ietf:params:jmap:vacationresponse'];
}
```

## Example: Full workflow

Set up vacation for a trip:

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:vacationresponse"],
  "methodCalls": [
    ["VacationResponse/set", {
      "accountId": "A123",
      "update": {
        "singleton": {
          "isEnabled": true,
          "fromDate": "2024-02-15T00:00:00Z",
          "toDate": "2024-02-25T23:59:59Z",
          "subject": "Away on vacation",
          "textBody": "Hello,\n\nThank you for your email. I am currently on vacation from February 15-25 and will have limited access to email.\n\nI will respond to your message when I return. For urgent matters, please contact my colleague at colleague@example.com.\n\nBest regards,\nAlice",
          "htmlBody": "<p>Hello,</p><p>Thank you for your email. I am currently on vacation from February 15-25 and will have limited access to email.</p><p>I will respond to your message when I return. For urgent matters, please contact my colleague at <a href=\"mailto:colleague@example.com\">colleague@example.com</a>.</p><p>Best regards,<br>Alice</p>"
        }
      }
    }, "0"]
  ]
}
```
