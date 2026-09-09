---
title: "context_enrich"
date: 0001-01-01
summary: "context_enrich #     Category Scope     Transform record    A processor that promotes collection context — the collecting agent&rsquo;s identity and the envelope&rsquo;s stable resource attributes — into the record&rsquo;s Fields.
Configuration #     Field Type Default Description     fields liststring  Source fields to read from.   ignore_missing bool true Do not fail when the source field is missing."
---


# context_enrich

| Category | Scope |
|----------|-------|
| Transform | record |

A processor that promotes collection context — the collecting agent's identity and the envelope's stable resource attributes — into the record's Fields.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fields` | liststring |  | Source fields to read from. |
| `ignore_missing` | bool | true | Do not fail when the source field is missing. |

## Example

```yaml
processor:
  - context_enrich:
      fields: ["message", "tags"]
      ignore_missing: true
```

