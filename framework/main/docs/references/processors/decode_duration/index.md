---
title: "decode_duration"
date: 0001-01-01
summary: "decode_duration #     Category Scope     Transform record    The &ldquo;decode_duration&rdquo; pipeline processor: convert a Go duration string like &ldquo;5s&rdquo; or &ldquo;1m30s&rdquo; into a numeric duration (milliseconds by default).
Configuration #     Field Type Default Description     field string  Source field to read from.   format string  Preset format selector (e.g. log format).    Example #  processor: - decode_duration: field: &#34;message&#34; format: &#34;format&#34; "
---


# decode_duration

| Category | Scope |
|----------|-------|
| Transform | record |

The "decode_duration" pipeline processor: convert a Go duration string like "5s" or "1m30s" into a numeric duration (milliseconds by default).

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `field` | string |  | Source field to read from. |
| `format` | string |  | Preset format selector (e.g. log format). |

## Example

```yaml
processor:
  - decode_duration:
      field: "message"
      format: "format"
```

