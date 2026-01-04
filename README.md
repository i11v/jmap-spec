# JMAP Protocol Test Specifications

Complete markdown-based test specifications for JMAP (JSON Meta Application Protocol), covering RFC 8620 (Core) and RFC 8621 (Mail). Format inspired by [ty's mdtest](https://github.com/astral-sh/ruff).

## Structure

```
jmap-spec/
├── specs/
│   ├── core/                        # RFC 8620 - JMAP Core
│   │   ├── session.md               # Session resource
│   │   ├── requests.md              # Request/response format
│   │   ├── errors.md                # Error handling
│   │   ├── binary.md                # Upload/download
│   │   ├── push.md                  # Push notifications
│   │   └── standard-methods.md      # Standard method patterns
│   │
│   └── mail/                        # RFC 8621 - JMAP Mail
│       ├── mailbox-get.md           # Mailbox/get
│       ├── mailbox-set.md           # Mailbox/set
│       ├── mailbox-query.md         # Mailbox/query
│       ├── mailbox-changes.md       # Mailbox/changes
│       ├── mailbox-query-changes.md # Mailbox/queryChanges
│       │
│       ├── email-get.md             # Email/get
│       ├── email-set.md             # Email/set
│       ├── email-query.md           # Email/query
│       ├── email-changes.md         # Email/changes
│       ├── email-query-changes.md   # Email/queryChanges
│       ├── email-copy.md            # Email/copy
│       ├── email-import.md          # Email/import
│       ├── email-parse.md           # Email/parse
│       │
│       ├── thread-get.md            # Thread/get
│       ├── thread-changes.md        # Thread/changes
│       │
│       ├── search-snippet.md        # SearchSnippet/get
│       │
│       ├── identity.md              # Identity/get, /set, /changes
│       ├── email-submission.md      # EmailSubmission/set, /get
│       ├── email-submission-extended.md # EmailSubmission/query, /changes
│       │
│       └── vacation-response.md     # VacationResponse/get, /set
```

## Specification Format

Each spec file uses markdown with JSON code blocks:

````markdown
# Method/name

Description of the method.

## Test case name

Description of test case.

```json
{
  "using": ["urn:ietf:params:jmap:core", "urn:ietf:params:jmap:mail"],
  "methodCalls": [
    ["Method/name", { ... }, "0"]
  ]
}
```

```json
// response
{
  "methodResponses": [
    ["Method/name", { ... }, "0"]
  ]
}
```
````

## Coverage

### RFC 8620 - JMAP Core

| Topic | Spec File |
|-------|-----------|
| Session resource | `core/session.md` |
| Request/response | `core/requests.md` |
| Error handling | `core/errors.md` |
| Binary upload/download | `core/binary.md` |
| Push notifications | `core/push.md` |
| Standard methods | `core/standard-methods.md` |

### RFC 8621 - JMAP Mail

#### Mailbox
| Method | Spec File |
|--------|-----------|
| Mailbox/get | `mail/mailbox-get.md` |
| Mailbox/set | `mail/mailbox-set.md` |
| Mailbox/query | `mail/mailbox-query.md` |
| Mailbox/changes | `mail/mailbox-changes.md` |
| Mailbox/queryChanges | `mail/mailbox-query-changes.md` |

#### Email
| Method | Spec File |
|--------|-----------|
| Email/get | `mail/email-get.md` |
| Email/set | `mail/email-set.md` |
| Email/query | `mail/email-query.md` |
| Email/changes | `mail/email-changes.md` |
| Email/queryChanges | `mail/email-query-changes.md` |
| Email/copy | `mail/email-copy.md` |
| Email/import | `mail/email-import.md` |
| Email/parse | `mail/email-parse.md` |

#### Thread
| Method | Spec File |
|--------|-----------|
| Thread/get | `mail/thread-get.md` |
| Thread/changes | `mail/thread-changes.md` |

#### SearchSnippet
| Method | Spec File |
|--------|-----------|
| SearchSnippet/get | `mail/search-snippet.md` |

#### Identity
| Method | Spec File |
|--------|-----------|
| Identity/get | `mail/identity.md` |
| Identity/set | `mail/identity.md` |
| Identity/changes | `mail/identity.md` |

#### EmailSubmission
| Method | Spec File |
|--------|-----------|
| EmailSubmission/get | `mail/email-submission.md` |
| EmailSubmission/set | `mail/email-submission.md` |
| EmailSubmission/query | `mail/email-submission-extended.md` |
| EmailSubmission/changes | `mail/email-submission-extended.md` |
| EmailSubmission/queryChanges | `mail/email-submission-extended.md` |

#### VacationResponse
| Method | Spec File |
|--------|-----------|
| VacationResponse/get | `mail/vacation-response.md` |
| VacationResponse/set | `mail/vacation-response.md` |

## Capabilities

| Capability URI | Description |
|----------------|-------------|
| `urn:ietf:params:jmap:core` | Core JMAP (always required) |
| `urn:ietf:params:jmap:mail` | Mailbox, Thread, Email, SearchSnippet |
| `urn:ietf:params:jmap:submission` | Identity, EmailSubmission |
| `urn:ietf:params:jmap:vacationresponse` | VacationResponse |

## References

- [RFC 8620](https://datatracker.ietf.org/doc/html/rfc8620) - JMAP Core
- [RFC 8621](https://datatracker.ietf.org/doc/html/rfc8621) - JMAP for Mail
- [JMAP Spec Source](https://github.com/jmapio/jmap) - Official JMAP specification repository
- [ty's mdtest](https://github.com/astral-sh/ruff/tree/main/crates/ty_python_semantic/resources/mdtest) - Test format inspiration

## Usage with TypeScript/Vite

These specifications are designed to be parsed and used to generate tests for TypeScript JMAP client libraries.

### Typical workflow

1. Parse spec markdown files
2. Extract JSON request/response pairs
3. Generate Vitest test cases
4. Run against a JMAP server

### Example test generation

```typescript
import { parseSpec } from './spec-parser';
import { describe, it, expect } from 'vitest';

const specs = parseSpec('specs/mailbox-get.md');

describe('Mailbox/get', () => {
  for (const testCase of specs.testCases) {
    it(testCase.name, async () => {
      const response = await jmapClient.request(testCase.request);
      expect(response).toMatchObject(testCase.expectedResponse);
    });
  }
});
```

### Spec parser implementation

A parser should:
1. Parse markdown headings for test names
2. Extract JSON code blocks
3. Identify request blocks (no `// response` comment)
4. Identify response blocks (`// response` comment)
5. Build test cases from paired request/response blocks
