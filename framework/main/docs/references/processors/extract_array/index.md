---
title: "extract_array"
date: 0001-01-01
summary: "extract_array #     Category Scope     Transform record    The &ldquo;extract_array&rdquo; pipeline processor: pull one element (or flatten all elements as indexed fields) out of an array field (Vector&rsquo;s parse arrays / beats extract_array equivalent).
Configuration #     Field Type Default Description     field string  Source field to read from.   index int 0 Destination index name."
---


# extract_array

| Category | Scope |
|----------|-------|
| Transform | record |

The "extract_array" pipeline processor: pull one element (or flatten all elements as indexed fields) out of an array field (Vector's parse arrays / beats extract_array equivalent).

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `field` | string |  | Source field to read from. |
| `index` | int | 0 | Destination index name. |
| `target` | string |  | Destination prefix or field to write the result to. |

## Example

```yaml
processor:
  - extract_array:
      field: "message"
      index: 10
      target: "parsed"
```

