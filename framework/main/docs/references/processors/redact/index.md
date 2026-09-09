---
title: "redact"
date: 0001-01-01
summary: "redact #     Category Scope     Governance record    The &ldquo;redact&rdquo; pipeline processor: mask sensitive substrings (phone numbers, national ids, cards, emails or arbitrary regexes) in the configured string fields before the record is shipped or stored.
Configuration #     Field Type Default Description     fields liststring  Source fields to read from.   patterns liststring  Extraction patterns tried in order; the first match wins."
---


# redact

| Category | Scope |
|----------|-------|
| Governance | record |

The "redact" pipeline processor: mask sensitive substrings (phone numbers, national ids, cards, emails or arbitrary regexes) in the configured string fields before the record is shipped or stored.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fields` | liststring |  | Source fields to read from. |
| `patterns` | liststring |  | Extraction patterns tried in order; the first match wins. |
| `replacement` | string | "***" | Replacement text used by the substitution. |

## Example

```yaml
processor:
  - redact:
      fields: ["message", "tags"]
      patterns: ["%{IP:client} %{WORD:verb} %{URIPATHPARAM:request}"]
      replacement: "replacement"
```

