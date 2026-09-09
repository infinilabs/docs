---
title: "xml"
date: 0001-01-01
summary: "xml #     Category Scope     Parsing record    The &ldquo;xml&rdquo; pipeline processor: XML → map conversion (Graylog&rsquo;s xml_to_json). Attributes are preserved under &ldquo;_&quot;-prefixed keys; repeated elements collapse to slices.
Configuration #     Field Type Default Description     source string  Source field to read from.   target_field string  Destination field to write the result to."
---


# xml

| Category | Scope |
|----------|-------|
| Parsing | record |

The "xml" pipeline processor: XML → map conversion (Graylog's xml_to_json). Attributes are preserved under "_"-prefixed keys; repeated elements collapse to slices.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | string |  | Source field to read from. |
| `target_field` | string |  | Destination field to write the result to. |
| `ignore_failure` | bool |  | Do not fail the record when processing errors; the record passes through unchanged. |

## Example

```yaml
processor:
  - xml:
      source: "message"
      target_field: "parsed"
      ignore_failure: true
```

