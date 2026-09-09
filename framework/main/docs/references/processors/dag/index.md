---
title: "dag"
date: 0001-01-01
summary: "dag #     Category Scope     Framework pipeline    Configuration #     Field Type Default Description     message string  Message text to log or process.    Example #  processor: - dag: message: &#34;message&#34; "
---


# dag

| Category | Scope |
|----------|-------|
| Framework | pipeline |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `message` | string |  | Message text to log or process. |

## Example

```yaml
processor:
  - dag:
      message: "message"
```

