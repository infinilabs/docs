---
title: "echo"
date: 0001-01-01
summary: "echo #     Category Scope     Framework pipeline    Configuration #     Field Type Default Description     message string  Message text to log or process.    Example #  pipeline: - name: demo processor: - echo: message: &#34;hello world&#34; "
---


# echo

| Category | Scope |
|----------|-------|
| Framework | pipeline |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `message` | string |  | Message text to log or process. |

## Example

```yaml
pipeline:
  - name: demo
    processor:
      - echo:
          message: "hello world"
```

