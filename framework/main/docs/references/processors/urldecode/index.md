---
title: "urldecode"
date: 0001-01-01
summary: "urldecode #     Category Scope     Parsing record    The &ldquo;urldecode&rdquo; pipeline processor: URL-decode string fields (query-string style unescaping).
Configuration #     Field Type Default Description     fields listfromTo  Source fields to read from.   ignore_missing bool false Do not fail when the source field is missing.   fail_on_error bool true Fail the record when the field cannot be decoded."
---


# urldecode

| Category | Scope |
|----------|-------|
| Parsing | record |

The "urldecode" pipeline processor: URL-decode string fields (query-string style unescaping).

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fields` | listfromTo |  | Source fields to read from. |
| `ignore_missing` | bool | false | Do not fail when the source field is missing. |
| `fail_on_error` | bool | true | Fail the record when the field cannot be decoded. |

## Example

```yaml
processor:
  - urldecode:
      fields: ["message", "tags"]
      ignore_missing: true
      fail_on_error: true
```

